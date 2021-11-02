<p data-nodeid="337" class="">上一课时我们学习了 Flink 消费 Kafka 数据计算 PV 和 UV 的水印和窗口设计，并且定义了窗口计算的触发器，完成了计算 PV 和 UV 前的所有准备工作。</p>
<p data-nodeid="338">接下来就需要计算 PV 和 UV 了。在当前业务场景下，根据 userId 进行统计，PV 需要对 userId 进行统计，而 UV 则需要对 userId 进行去重统计。</p>
<p data-nodeid="339">下面我们使用不同的方法来统计 PV 和 UV。</p>
<h3 data-nodeid="340">单窗口内存统计</h3>
<p data-nodeid="341">这种方法需要把一天内所有的数据进行缓存，然后在内存中遍历接收的数据，进行 PV 和 UV 的叠加统计。</p>
<pre class="lang-java" data-nodeid="342"><code data-language="java">StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment(); 
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime); 
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
SingleOutputStreamOperator&lt;UserClick&gt; userClickSingleOutputStreamOperator = dataStream.assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> BoundedOutOfOrdernessTimestampExtractor&lt;UserClick&gt;(Time.seconds(<span class="hljs-number">30</span>)) { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(UserClick element)</span> </span>{ 
        <span class="hljs-keyword">return</span> element.getTimestamp(); 
    } 
}); 
userClickSingleOutputStreamOperator 
        .windowAll(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
        .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
...
</code></pre>
<p data-nodeid="343">在上一课时中我们已经定义了全局的窗口，并且自定义触发器，每 20 秒触发一次计算输出中间结果。</p>
<p data-nodeid="344">我们在后面可以继续调用 process 方法，自定义 ProcessFunction 如下：</p>
<pre class="lang-java" data-nodeid="345"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyProcessFunction</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ProcessAllWindowFunction</span>&lt;<span class="hljs-title">UserClick</span>,<span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>,<span class="hljs-title">Integer</span>&gt;,<span class="hljs-title">TimeWindow</span>&gt; </span>{ 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">process</span><span class="hljs-params">(Context context, Iterable&lt;UserClick&gt; elements, Collector&lt;Tuple2&lt;String, Integer&gt;&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        HashSet&lt;String&gt; uv = Sets.newHashSet(); 
        Integer pv = <span class="hljs-number">0</span>; 
        Iterator&lt;UserClick&gt; iterator = elements.iterator(); 
        <span class="hljs-keyword">while</span> (iterator.hasNext()){ 
            String userId = iterator.next().getUserId(); 
            uv.add(userId); 
            pv = pv + <span class="hljs-number">1</span>; 
        } 
        out.collect(Tuple2.of(<span class="hljs-string">"pv"</span>,pv)); 
        out.collect(Tuple2.of(<span class="hljs-string">"uv"</span>,uv.size())); 
    } 
}
</code></pre>
<p data-nodeid="346">我们自定义的 ProcessFunction 继承了 ProcessAllWindowFunction 并且覆写了 process 方法。在 process 方法中，新建了一个 HashSet，利用 HashSet 对 userId 去重，即是我们需要的 UV。PV 初始化为 0 并且每来一条数据则加1，最后得到的就是 PV。</p>
<p data-nodeid="347">TumblingProcessingTimeWindows.of(Time.days(1), Time.hours(-8)) 方法每天从 0 点开始计算并且每天都会清空数据。</p>
<p data-nodeid="348">这种方法代码简单清晰，但是有很严重的内存占用问题。如果我们的数据量很大，那么所定义的 TumblingProcessingTimeWindows 窗口会缓存一整天的数据，内存消耗非常大。</p>
<h3 data-nodeid="349">分组窗口 + 过期数据剔除</h3>
<p data-nodeid="421" class="">为了减少窗口内缓存的数据量，我们可以根据用户的访问时间戳所在天进行分组，然后将数据分散在各个窗口内进行计算，接着在 State 中进行汇总。</p>

<p data-nodeid="2817" class="">首先，我们把 DataStream 按照用户的访问时间所在天进行分组：</p>

<pre class="lang-java" data-nodeid="2993"><code data-language="java">userClickSingleOutputStreamOperator 
        .keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;UserClick, String&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKey</span><span class="hljs-params">(UserClick value)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                <span class="hljs-keyword">return</span> DateUtil.timeStampToDate(value.getTimestamp()); 
            } 
        }) 
        .window(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
        .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
        .evictor(TimeEvictor.of(Time.seconds(<span class="hljs-number">0</span>), <span class="hljs-keyword">true</span>)) 
        ...
</code></pre>

<p data-nodeid="3168" class="">然后根据用户的访问时间所在天进行分组并且调用了 evictor 来剔除已经计算过的数据。</p>

<p data-nodeid="1753">其中的 DateUtil 是获取时间戳的年月日：</p>
<pre data-nodeid="2642" class=""><code>public class DateUtil {
    public static String timeStampToDate(Long timestamp){
        ThreadLocal&lt;SimpleDateFormat&gt; threadLocal
                = ThreadLocal.withInitial(() -&gt; new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        String format = threadLocal.get().format(new Date(timestamp));
        return format.substring(0,10);
    }
}
</code></pre>















<p data-nodeid="680">接下来看看 Flink 中的剔除器原理和使用方法。</p>




<h4 data-nodeid="355">剔除器</h4>
<p data-nodeid="356">Flink 中的剔除器可以在 Window Function 执行前或后使用，用来从 Window 中剔除元素。目前 Flink 支持了三种类型的剔除器，具体如下。</p>
<ul data-nodeid="357">
<li data-nodeid="358">
<p data-nodeid="359">CountEvictor：数量剔除器。在 Window 中保留指定数量的元素，并从窗口头部开始丢弃其余元素。</p>
</li>
<li data-nodeid="360">
<p data-nodeid="361">DeltaEvictor：阈值剔除器。计算 Window 中最后一个元素与其余每个元素之间的增量，丢弃增量大于或等于阈值的元素。</p>
</li>
<li data-nodeid="362">
<p data-nodeid="363">TimeEvictor：时间剔除器。保留 Window 中最近一段时间内的元素，并丢弃其余元素。</p>
</li>
</ul>
<h4 data-nodeid="364">剔除器源码</h4>
<p data-nodeid="365">我们在这里使用了 TimeEvictor，来看看该剔除器的源码实现：</p>
<pre class="lang-java" data-nodeid="366"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">evict</span><span class="hljs-params">(Iterable&lt;TimestampedValue&lt;Object&gt;&gt; elements, <span class="hljs-keyword">int</span> size, EvictorContext ctx)</span> </span>{ 
   <span class="hljs-keyword">if</span> (!hasTimestamp(elements)) { 
      <span class="hljs-keyword">return</span>; 
   } 
   <span class="hljs-keyword">long</span> currentTime = getMaxTimestamp(elements); 
   <span class="hljs-keyword">long</span> evictCutoff = currentTime - windowSize; 
   <span class="hljs-keyword">for</span> (Iterator&lt;TimestampedValue&lt;Object&gt;&gt; iterator = elements.iterator(); iterator.hasNext(); ) { 
      TimestampedValue&lt;Object&gt; record = iterator.next(); 
      <span class="hljs-keyword">if</span> (record.getTimestamp() &lt;= evictCutoff) { 
         iterator.remove(); 
      } 
   } 
} 
...
</code></pre>
<p data-nodeid="367">可以看到，该剔除器首先会找到窗口中元素的最大时间戳，并且找到当前窗口的<strong data-nodeid="409">截断点</strong>：最大的时间戳减去要保留的时间段；然后遍历窗口中每一个元素，如果当前元素的时间戳小于等于截断点，则剔除该元素。</p>
<p data-nodeid="368">所以，我们定义的 TimeEvictor.of(Time.seconds(0), true) 则会剔除每条计算过的元素。</p>
<p data-nodeid="369">接下来我们实现自己的 ProcessFunction：</p>
<pre class="lang-java" data-nodeid="370"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyProcessWindowFunction</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ProcessWindowFunction</span>&lt;<span class="hljs-title">UserClick</span>,<span class="hljs-title">Tuple3</span>&lt;<span class="hljs-title">String</span>,<span class="hljs-title">String</span>, <span class="hljs-title">Integer</span>&gt;,<span class="hljs-title">String</span>,<span class="hljs-title">TimeWindow</span>&gt;</span>{ 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> MapState&lt;String, String&gt; uvState; 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> ValueState&lt;Integer&gt; pvState; 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.open(parameters); 
        uvState = <span class="hljs-keyword">this</span>.getRuntimeContext().getMapState(<span class="hljs-keyword">new</span> MapStateDescriptor&lt;&gt;(<span class="hljs-string">"uv"</span>, String.class, String.class)); 
        pvState = <span class="hljs-keyword">this</span>.getRuntimeContext().getState(<span class="hljs-keyword">new</span> ValueStateDescriptor&lt;Integer&gt;(<span class="hljs-string">"pv"</span>, Integer.class)); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">process</span><span class="hljs-params">(String s, Context context, Iterable&lt;UserClick&gt; elements, Collector&lt;Tuple3&lt;String, String, Integer&gt;&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        Integer pv = <span class="hljs-number">0</span>; 
        Iterator&lt;UserClick&gt; iterator = elements.iterator(); 
        <span class="hljs-keyword">while</span> (iterator.hasNext()){ 
            pv = pv + <span class="hljs-number">1</span>; 
            String userId = iterator.next().getUserId(); 
            uvState.put(userId,<span class="hljs-keyword">null</span>); 
        } 
        pvState.update(pvState.value() + pv); 
        Integer uv = <span class="hljs-number">0</span>; 
        Iterator&lt;String&gt; uvIterator = uvState.keys().iterator(); 
        <span class="hljs-keyword">while</span> (uvIterator.hasNext()){ 
            String next = uvIterator.next(); 
            uv = uv + <span class="hljs-number">1</span>; 
        } 
        Integer value = pvState.value(); 
        <span class="hljs-keyword">if</span>(<span class="hljs-keyword">null</span> == value){ 
            pvState.update(pv); 
        }<span class="hljs-keyword">else</span> { 
            pvState.update(value + pv); 
        } 
        out.collect(Tuple3.of(s,<span class="hljs-string">"uv"</span>,uv)); 
        out.collect(Tuple3.of(s,<span class="hljs-string">"pv"</span>,pvState.value())); 
    } 
}
</code></pre>
<p data-nodeid="371">在 State 中进行 PV 和 UV 的迭代计算，并且将计算过的数据剔除出窗口。保证窗口中的数据不会占用太多内存。</p>
<p data-nodeid="372">我们在主程序中可以直接使用：</p>
<pre class="lang-java" data-nodeid="373"><code data-language="java">userClickSingleOutputStreamOperator 
        .keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;UserClick, String&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKey</span><span class="hljs-params">(UserClick value)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                <span class="hljs-keyword">return</span> value.getUserId(); 
            } 
        }) 
        .window(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
        .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
        .evictor(TimeEvictor.of(Time.seconds(<span class="hljs-number">0</span>), <span class="hljs-keyword">true</span>)) 
        .process(<span class="hljs-keyword">new</span> MyProcessWindowFunction());
</code></pre>
<h3 data-nodeid="374">使用 BitMap / 布隆过滤器</h3>
<p data-nodeid="375">我们在计算 UV 时需要进行全局去重，在第 20 课时“Flink 高级应用之海量数据高效去重”中介绍过大数量下的去重方法，这里我们就可以使用 BitMap 或者布隆过滤器进行去重。</p>
<p data-nodeid="376">假如用户的 ID 可以转化为 Long 型，可以使用 BitMap 进行去重计算 UV：</p>
<pre class="lang-java" data-nodeid="377"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyProcessWindowFunctionBitMap</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ProcessWindowFunction</span>&lt;<span class="hljs-title">UserClick</span>,<span class="hljs-title">Tuple3</span>&lt;<span class="hljs-title">String</span>,<span class="hljs-title">String</span>, <span class="hljs-title">Integer</span>&gt;,<span class="hljs-title">String</span>,<span class="hljs-title">TimeWindow</span>&gt;</span>{ 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> ValueState&lt;Integer&gt; uvState; 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> ValueState&lt;Integer&gt; pvState; 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> ValueState&lt;Roaring64NavigableMap&gt; bitMapState; 

    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.open(parameters); 
        uvState = <span class="hljs-keyword">this</span>.getRuntimeContext().getState(<span class="hljs-keyword">new</span> ValueStateDescriptor&lt;Integer&gt;(<span class="hljs-string">"uv"</span>, Integer.class)); 
        pvState = <span class="hljs-keyword">this</span>.getRuntimeContext().getState(<span class="hljs-keyword">new</span> ValueStateDescriptor&lt;Integer&gt;(<span class="hljs-string">"pv"</span>, Integer.class)); 
        bitMapState = <span class="hljs-keyword">this</span>.getRuntimeContext().getState(<span class="hljs-keyword">new</span> ValueStateDescriptor(<span class="hljs-string">"bitMap"</span>, TypeInformation.of(<span class="hljs-keyword">new</span> TypeHint&lt;Roaring64NavigableMap&gt;() { 
        }))); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">process</span><span class="hljs-params">(String s, Context context, Iterable&lt;UserClick&gt; elements, Collector&lt;Tuple3&lt;String, String, Integer&gt;&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        Integer uv = uvState.value(); 
        Integer pv = pvState.value(); 
        Roaring64NavigableMap bitMap = bitMapState.value(); 
        <span class="hljs-keyword">if</span>(bitMap == <span class="hljs-keyword">null</span>){ 
            bitMap = <span class="hljs-keyword">new</span> Roaring64NavigableMap(); 
            uv = <span class="hljs-number">0</span>; 
            pv = <span class="hljs-number">0</span>; 
        } 
        Iterator&lt;UserClick&gt; iterator = elements.iterator(); 
        <span class="hljs-keyword">while</span> (iterator.hasNext()){ 
            pv = pv + <span class="hljs-number">1</span>; 
            String userId = iterator.next().getUserId(); 
            <span class="hljs-comment">//如果userId可以转成long </span>
            bitMap.add(Long.valueOf(userId)); 
        } 
        out.collect(Tuple3.of(s,<span class="hljs-string">"uv"</span>,bitMap.getIntCardinality())); 
        out.collect(Tuple3.of(s,<span class="hljs-string">"pv"</span>,pv)); 

    } 
}
</code></pre>
<p data-nodeid="378">上面定义了一个 Roaring64NavigableMap 用来存储用户 ID，最后只需要调用 bitMap.getIntCardinality() 即可求得去重后的 UV 值。</p>
<p data-nodeid="379">当然了，如果业务中的用户 ID 是字符串类型，不能被转化为 Long 型，那么你可以使用布隆过滤器进行去重。</p>
<h3 data-nodeid="380">总结</h3>
<p data-nodeid="381" class="">这一课时我们使用了三种方法进行了 PV 和 UV 的计算，分别是全窗口内存统计、使用分组和过期数据剔除、使用 BitMap / 布隆过滤器，在实际业务中我们可以根据业务场景灵活使用。</p>

---

### 精选评论

##### **_18500558771：
> 在这一讲中按日期date分组然后process中计算pvuv，但后续代码中改为了keyby用userid分组再process计算pvuv，这有问题吧。如果这样process是在每个key流窗体到期就会调用一次，不再是一天的pvuv，而是一天内某个userid的pvuv 求解 谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是按照date分组，同一天发生的数据发布到一个窗口，在窗口内排序，这样得到每天的排序结果。

##### *亮：
> 你好，请问[分组窗口 + 过期数据剔除]这个按天分组和不分组统计uv都是只有一个窗口，在我看来基本上都是没有优化的啊，有些疑惑

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不按照天分组的，所有数据进入一个窗口并且不带失效。数据量大起来，一个窗口计算那效率就可想而知了。

##### **忠：
> 您好。请问一下这里做的分组窗口不也是当天的数据么，和不做分组有啥区别，感觉数据量并没有减少，都在一个分组去了（过期数据剔除另说）。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的分组是按照天，也就是说每天的数据进入一个组单独计算，不要在一个窗口内计算很多天的数据。

##### *亮：
> 您好，请问同一个算子不同task即使在不同的节点，状态也是共享的是么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是这样的，状态算子也分种类，例如KeyedState是按照某个Key进行分组，那么只有同一组的才能状态共享。搜一下Keyed State和Operator State

##### *亮：
> 有个疑问：【分组窗口 + 过期数据剔除】这部分中，您用了keyBy会把不同的key分到不同的窗口中，而前面讲过状态在子算子中并不是共享的，实现的ProcessWindowFunction并不能统计全局的PV/UV吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在最后的process函数中，窗口中的数据都会去更新mapstate，这里的mapstate是window算子公用的。要注意区分【不同的算子】和【同一个算子的不同task】

##### *灿：
> 老师，最后bitmap去重方法的代码中，pv，uv，bitmap的状态都没有保存回state，这样应该有问题吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 已经保存回state了，我们在open中创建的pvState和uvState两个变量在后面的process方法中都有使用。

