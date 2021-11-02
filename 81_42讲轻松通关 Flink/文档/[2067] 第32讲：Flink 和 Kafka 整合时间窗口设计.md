<p data-nodeid="68850">我们在第 31 课时中讲过，在计算 PV 和 UV 等指标前，用 Flink 将原始数据进行了清洗，清洗完毕的数据被发送到另外的 Kafka Topic 中，接下来我们只需要消费指定 Topic 的数据，然后就可以进行指标计算了。</p>



<h3 data-nodeid="68107">Flink 消费 Kafka 数据反序列化</h3>
<p data-nodeid="68108">上一课时定义了用户的行为信息的 Java 对象，我们现在需要消费新的 Kafka Topic 信息，并且把序列化的消息转化为用户的行为对象：</p>
<pre class="lang-java" data-nodeid="68109"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserClick</span> </span>{ 
    <span class="hljs-keyword">private</span> String userId; 
    <span class="hljs-keyword">private</span> Long timestamp; 
    <span class="hljs-keyword">private</span> String action; 
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUserId</span><span class="hljs-params">()</span> </span>{ 
        <span class="hljs-keyword">return</span> userId; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setUserId</span><span class="hljs-params">(String userId)</span> </span>{ 
        <span class="hljs-keyword">this</span>.userId = userId; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> Long <span class="hljs-title">getTimestamp</span><span class="hljs-params">()</span> </span>{ 
        <span class="hljs-keyword">return</span> timestamp; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setTimestamp</span><span class="hljs-params">(Long timestamp)</span> </span>{ 
        <span class="hljs-keyword">this</span>.timestamp = timestamp; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getAction</span><span class="hljs-params">()</span> </span>{ 
        <span class="hljs-keyword">return</span> action; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setAction</span><span class="hljs-params">(String action)</span> </span>{ 
        <span class="hljs-keyword">this</span>.action = action; 
    } 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">UserClick</span><span class="hljs-params">(String userId, Long timestamp, String action)</span> </span>{ 
        <span class="hljs-keyword">this</span>.userId = userId; 
        <span class="hljs-keyword">this</span>.timestamp = timestamp; 
        <span class="hljs-keyword">this</span>.action = action; 
    } 
} 
<span class="hljs-keyword">enum</span> UserAction{ 
    <span class="hljs-comment">//点击 </span>
    CLICK(<span class="hljs-string">"CLICK"</span>), 
    <span class="hljs-comment">//购买 </span>
    PURCHASE(<span class="hljs-string">"PURCHASE"</span>), 
    <span class="hljs-comment">//其他 </span>
    OTHER(<span class="hljs-string">"OTHER"</span>); 
    <span class="hljs-keyword">private</span> String action; 
    UserAction(String action) { 
        <span class="hljs-keyword">this</span>.action = action; 
    } 
} 
</code></pre>
<p data-nodeid="68110">首先，我们需要新建自己的 Kafka Condumer，在第 12 和 24 课时中详细讲解了 Flink 消费 Kafak 消息的原理和实现。在计算 PV 和 UV 的业务场景中，我们选择使用消息中自带的事件时间作为时间特征，代码如下：</p>
<pre class="lang-java" data-nodeid="68111"><code data-language="java">StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment(); 
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime); 
<span class="hljs-comment">// 检查点配置,如果要用到状态后端，那么必须配置 </span>
env.setStateBackend(<span class="hljs-keyword">new</span> MemoryStateBackend(<span class="hljs-keyword">true</span>)); 
Properties properties = <span class="hljs-keyword">new</span> Properties(); 
properties.setProperty(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"127.0.0.1:9092"</span>); 
properties.setProperty(FlinkKafkaConsumerBase.KEY_PARTITION_DISCOVERY_INTERVAL_MILLIS, <span class="hljs-string">"10"</span>); 
FlinkKafkaConsumer&lt;String&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(<span class="hljs-string">"log_user_action"</span>, <span class="hljs-keyword">new</span> SimpleStringSchema(), properties); 
<span class="hljs-comment">//设置从最早的offset消费 </span>
consumer.setStartFromEarliest(); 
DataStream&lt;UserClick&gt; dataStream = env 
        .addSource(consumer) 
        .name(<span class="hljs-string">"log_user_action"</span>) 
        .map(message -&gt; { 
            JSONObject record = JSON.parseObject(message); 
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UserClick( 
                    record.getString(<span class="hljs-string">"user_id"</span>), 
                    record.getLong(<span class="hljs-string">"timestamp"</span>), 
                    record.getString(<span class="hljs-string">"action"</span>) 
            ); 
        }) 
        .returns(TypeInformation.of(UserClick.class)); 
</code></pre>
<p data-nodeid="68112">在上面的代码中，我们消费第 31 课时中写入的新的 Kafka Topic，并且将读取到的数据反序列化为 UserClick 的 DataStream。</p>
<h3 data-nodeid="68113">水印和窗口设计</h3>
<p data-nodeid="68114">我们得到 UserClick 的 DataStream 后，需要进行水印和窗口的设计，可在 DataStream 上调用 assignTimestampsAndWatermarks 方法。我们在第 8 和 25 课时中，详细讲解过 Flink 支持的时间戳提取器和水印发射器：</p>
<ul data-nodeid="68115">
<li data-nodeid="68116">
<p data-nodeid="68117">周期性水印：AssignerWithPeriodicWatermarks</p>
</li>
<li data-nodeid="68118">
<p data-nodeid="68119">特定事件触发水印：PunctuatedWatermark</p>
</li>
</ul>
<p data-nodeid="68120">由于我们的用户访问日志可能存在乱序，所以使用 BoundedOutOfOrdernessTimestampExtractor&nbsp; 来处理乱序消息和延迟时间，我们指定消息的乱序时间 30 秒，具体代码如下：</p>
<pre class="lang-java" data-nodeid="68121"><code data-language="java">SingleOutputStreamOperator&lt;UserClick&gt; userClickSingleOutputStreamOperator = dataStream.assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> BoundedOutOfOrdernessTimestampExtractor&lt;UserClick&gt;(Time.seconds(<span class="hljs-number">30</span>)) { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(UserClick element)</span> </span>{ 
        <span class="hljs-keyword">return</span> element.getTimestamp(); 
    } 
}); 
</code></pre>
<h3 data-nodeid="68122">窗口触发器</h3>
<h4 data-nodeid="68123">窗口设置</h4>
<p data-nodeid="68124">到目前为止，我们已经通过读取 Kafka 中的数据，序列化为用户点击事件的 DataStream，并且完成了水印和时间戳的设计和开发。</p>
<p data-nodeid="68125">接下来，按照业务需要，我们需要开窗并且进行一天内用户点击事件的 PV、UV 计算。在第 8 课时中讲解了 Flink 支持的窗口类型：</p>
<ul data-nodeid="68126">
<li data-nodeid="68127">
<p data-nodeid="68128"><strong data-nodeid="68195">滚动窗口</strong>，窗口数据有固定的大小，窗口中的数据不会叠加；</p>
</li>
<li data-nodeid="68129">
<p data-nodeid="68130"><strong data-nodeid="68200">滑动窗口</strong>，窗口数据有固定的大小，并且有生成间隔；</p>
</li>
<li data-nodeid="68131">
<p data-nodeid="68132"><strong data-nodeid="68205">会话窗口</strong>，窗口数据没有固定的大小，根据用户传入的参数进行划分，窗口数据无叠加。</p>
</li>
</ul>
<p data-nodeid="68133">很明显，我们需要使用滚动窗口：TumblingProcessingTimeWindow，并且按照一般逻辑，需每天 0 点到 24 点进行一次计算和输出。</p>
<p data-nodeid="68134">这里我们使用 Flink 提供的滚动窗口，在构造滚动窗口之前先来看一下滚动窗口的实现：</p>
<pre class="lang-java" data-nodeid="68135"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TumblingEventTimeWindows</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WindowAssigner</span>&lt;<span class="hljs-title">Object</span>, <span class="hljs-title">TimeWindow</span>&gt; </span>{ 
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> serialVersionUID = <span class="hljs-number">1L</span>; 
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> size; 
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> offset; 
   <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">TumblingEventTimeWindows</span><span class="hljs-params">(<span class="hljs-keyword">long</span> size, <span class="hljs-keyword">long</span> offset)</span> </span>{ 
      <span class="hljs-keyword">if</span> (Math.abs(offset) &gt;= size) { 
         <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"TumblingEventTimeWindows parameters must satisfy abs(offset) &lt; size"</span>); 
      } 
      <span class="hljs-keyword">this</span>.size = size; 
      <span class="hljs-keyword">this</span>.offset = offset; 
   } 
   ... 
</code></pre>
<p data-nodeid="68136">这里需要注意的是，很多人会认为，窗口时间的开始时间会是我们的代码启动时间。事实上，根据上面的源码可见，TumblingEventTimeWindows 在构造时，需要指定两个参数：<strong data-nodeid="68217">窗口的长度</strong>和<strong data-nodeid="68218">窗口的 offset</strong>（默认为 0）。</p>
<pre class="lang-java" data-nodeid="68137"><code data-language="java"><span class="hljs-comment">/** 
 * Creates a new {<span class="hljs-doctag">@code</span> TumblingEventTimeWindows} {<span class="hljs-doctag">@link</span> WindowAssigner} that assigns 
 * elements to time windows based on the element timestamp and offset. 
 * 
 * &lt;p&gt;For example, if you want window a stream by hour,but window begins at the 15th minutes 
 * of each hour, you can use {<span class="hljs-doctag">@code</span> of(Time.hours(1),Time.minutes(15))},then you will get 
 * time windows start at 0:15:00,1:15:00,2:15:00,etc. 
 * 
 * &lt;p&gt;Rather than that,if you are living in somewhere which is not using UTC±00:00 time, 
 * such as China which is using UTC+08:00,and you want a time window with size of one day, 
 * and window begins at every 00:00:00 of local time,you may use {<span class="hljs-doctag">@code</span> of(Time.days(1),Time.hours(-8))}. 
 * The parameter of offset is {<span class="hljs-doctag">@code</span> Time.hours(-8))} since UTC+08:00 is 8 hours earlier than UTC time. 
 * 
 * <span class="hljs-doctag">@param</span> size The size of the generated windows. 
 * <span class="hljs-doctag">@param</span> offset The offset which window start would be shifted by. 
 * <span class="hljs-doctag">@return</span> The time policy. 
 */</span> 
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> TumblingEventTimeWindows <span class="hljs-title">of</span><span class="hljs-params">(Time size, Time offset)</span> </span>{ 
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TumblingEventTimeWindows(size.toMilliseconds(), offset.toMilliseconds()); 
} 
</code></pre>
<p data-nodeid="68138">我们如何取得某一天的 0 点这个时间起始点呢？TumblingEventTimeWindows 的 of 方法中已经给了提示：</p>
<blockquote data-nodeid="68139">
<p data-nodeid="68140">window begins at every 00:00:00 of local time,you may use {@code of(Time.days(1),Time.hours(-8))}.</p>
</blockquote>
<p data-nodeid="68141">我们只需要通过 TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8)) 就可以指定在中国的 0&nbsp;点开始创建窗口，然后每天计算一次输出结果即可。</p>
<p data-nodeid="68142">但是，在实际生产环境中，对于大窗口的计算，一般都会设置<strong data-nodeid="68227">触发器</strong>，以一定的频率输出中间结果，而不是等到一天结束时仅仅触发一次。</p>
<p data-nodeid="68143">这里我们就需要设置数据流的 Trigger 属性。</p>
<h4 data-nodeid="68144">触发器设置</h4>
<p data-nodeid="68145">窗口的计算是依赖触发器进行的，每种类型的窗口都有自己的触发器机制，如果用户没有指定，那么会使用默认的触发器。例如 TumblingEventTimeWindows 中自带的触发器如下：</p>
<pre class="lang-java" data-nodeid="68146"><code data-language="java"><span class="hljs-meta">@Override</span> 
<span class="hljs-function"><span class="hljs-keyword">public</span> Trigger&lt;Object, TimeWindow&gt; <span class="hljs-title">getDefaultTrigger</span><span class="hljs-params">(StreamExecutionEnvironment env)</span> </span>{ 
   <span class="hljs-keyword">return</span> EventTimeTrigger.create(); 
} 
</code></pre>
<p data-nodeid="68147">可以看到源码中包含了一个 DefaultTrigger：EventTimeTrigger，而 EventTimeTrigger 的实现如下：</p>
<pre class="lang-java" data-nodeid="68148"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EventTimeTrigger</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Trigger</span>&lt;<span class="hljs-title">Object</span>, <span class="hljs-title">TimeWindow</span>&gt; </span>{ 
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> serialVersionUID = <span class="hljs-number">1L</span>; 
   <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">EventTimeTrigger</span><span class="hljs-params">()</span> </span>{} 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> TriggerResult <span class="hljs-title">onElement</span><span class="hljs-params">(Object element, <span class="hljs-keyword">long</span> timestamp, TimeWindow window, TriggerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
      <span class="hljs-keyword">if</span> (window.maxTimestamp() &lt;= ctx.getCurrentWatermark()) { 
         <span class="hljs-keyword">return</span> TriggerResult.FIRE; 
      } <span class="hljs-keyword">else</span> { 
         ctx.registerEventTimeTimer(window.maxTimestamp()); 
         <span class="hljs-keyword">return</span> TriggerResult.CONTINUE; 
      } 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> TriggerResult <span class="hljs-title">onEventTime</span><span class="hljs-params">(<span class="hljs-keyword">long</span> time, TimeWindow window, TriggerContext ctx)</span> </span>{ 
      <span class="hljs-keyword">return</span> time == window.maxTimestamp() ? 
         TriggerResult.FIRE : 
         TriggerResult.CONTINUE; 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> TriggerResult <span class="hljs-title">onProcessingTime</span><span class="hljs-params">(<span class="hljs-keyword">long</span> time, TimeWindow window, TriggerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
      <span class="hljs-keyword">return</span> TriggerResult.CONTINUE; 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">clear</span><span class="hljs-params">(TimeWindow window, TriggerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
      ctx.deleteEventTimeTimer(window.maxTimestamp()); 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">canMerge</span><span class="hljs-params">()</span> </span>{ 
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>; 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onMerge</span><span class="hljs-params">(TimeWindow window, 
         OnMergeContext ctx)</span> </span>{ 
      <span class="hljs-keyword">long</span> windowMaxTimestamp = window.maxTimestamp(); 
      <span class="hljs-keyword">if</span> (windowMaxTimestamp &gt; ctx.getCurrentWatermark()) { 
         ctx.registerEventTimeTimer(windowMaxTimestamp); 
      } 
   } 
   <span class="hljs-meta">@Override</span> 
   <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{ 
      <span class="hljs-keyword">return</span> <span class="hljs-string">"EventTimeTrigger()"</span>; 
   } 
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> EventTimeTrigger <span class="hljs-title">create</span><span class="hljs-params">()</span> </span>{ 
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> EventTimeTrigger(); 
   } 
} 
</code></pre>
<p data-nodeid="69416">我们可以看到，EventTimeTrigger 触发器的工作原理是 判断当前的水印是否超过了窗口的结束时间，如果超过则触发对窗口内数据的计算，否则不触发计算。<br>
Flink 本身提供了不同种类的触发器供我们使用，如下图所示：</p>
<p data-nodeid="69417" class=""><img src="https://s0.lgstatic.com/i/image/M00/3C/9A/CgqCHl8n4MiAdhQPAACVaOaaD2Y990.png" alt="image.png" data-nodeid="69423"></p>


<p data-nodeid="68151">触发器的类图如上图所示，它们的实际含义如下：</p>
<ul data-nodeid="68152">
<li data-nodeid="68153">
<p data-nodeid="68154">EventTimeTrigger：通过对比 Watermark 和窗口的 Endtime 确定是否触发窗口计算，如果 Watermark 大于 Window EndTime 则触发，否则不触发，窗口将继续等待。</p>
</li>
<li data-nodeid="68155">
<p data-nodeid="68156">ProcessTimeTrigger：通过对比 ProcessTime 和窗口 EndTime 确定是否触发窗口，如果 ProcessTime 大于 EndTime 则触发计算，否则窗口继续等待。</p>
</li>
<li data-nodeid="68157">
<p data-nodeid="68158">ContinuousEventTimeTrigger：根据间隔时间周期性触发窗口或者 Window 的结束时间小于当前 EndTime 触发窗口计算。</p>
</li>
<li data-nodeid="68159">
<p data-nodeid="68160">ContinuousProcessingTimeTrigger：根据间隔时间周期性触发窗口或者 Window 的结束时间小于当前 ProcessTime 触发窗口计算。</p>
</li>
<li data-nodeid="68161">
<p data-nodeid="68162">CountTrigger：根据接入数据量是否超过设定的阈值判断是否触发窗口计算。</p>
</li>
<li data-nodeid="68163">
<p data-nodeid="68164">DeltaTrigger：根据接入数据计算出来的 Delta 指标是否超过指定的 Threshold 去判断是否触发窗口计算。</p>
</li>
<li data-nodeid="68165">
<p data-nodeid="68166">PurgingTrigger：可以将任意触发器作为参数转换为 Purge 类型的触发器，计算完成后数据将被清理。</p>
</li>
</ul>
<p data-nodeid="68167">我们在这里可以选择使用：ContinuousProcessingTimeTrigger 来周期性的触发窗口阶段性计算。</p>
<p data-nodeid="68168">实现代码如下：</p>
<pre class="lang-java" data-nodeid="68169"><code data-language="java">dataStream     .windowAll(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
.trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
</code></pre>
<p data-nodeid="68170">我们使用 ContinuousProcessingTimeTrigger 每隔 20 秒触发计算，输出中间结果。</p>
<p data-nodeid="68171">到目前为止，完成了除计算 PV 和 UV 的所有前置条件的开发，我们实现了：清洗原始数据、写入新的 Kafka Topic、消费新的 Topic 信息、自定义水印和时间戳、自定义窗口和触发器。</p>
<h3 data-nodeid="68172">总结</h3>
<p data-nodeid="68554">这一课时我们学习了 Flink 消费 Kafka 数据计算 PV 和 UV 的水印和窗口设计，并且定义了窗口计算的触发器，完成了计算 PV 和 UV 前的所有准备工作。</p>

---

### 精选评论

##### **星：
> 这些例子跟真实场景像吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，可以根据实际业务场景变通。

