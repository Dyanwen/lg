<p data-nodeid="141144">第 08 课时我们提过窗口和时间的概念，Flink 框架支持事件时间、摄入时间和处理时间三种。Watermark（水印）的出现是用于处理数据从 Source 产生，再到转换和输出，在这个过程中由于网络和反压的原因导致了消息乱序问题。</p>



<p data-nodeid="141396">那么在实际的开发过程中，如何正确地使用 Watermark 呢？</p>
<h3 data-nodeid="141397">使用 Watermark 必知必会</h3>


<h4 data-nodeid="140493">Watermark 和事件时间</h4>
<p data-nodeid="140494">事件时间（Event Time）是数据产生的时间，这个时间一般在数据中自带，由消息的生产者生成。例如，我们的上游是 Kafka 消息，那么每个生成的消息中自带一个时间戳代表该条数据的产生时间，这个时间是固定的，从数据的诞生开始就一直携带。所以，我们在处理消息乱序的情况时，会用 EventTime 和 Watermark 进行配合使用。</p>
<p data-nodeid="140495">我们只需要一行代码，就可以在代码中指定 Flink 系统使用的时间类型为 EventTime：</p>
<pre class="lang-java" data-nodeid="146043"><code data-language="java">env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
</code></pre>



















<p data-nodeid="140497">那么为什么不用处理时间（Processing Time）和摄入时间（Ingestion Time）呢？</p>
<p data-nodeid="140498">处理时间（Processing Time）指的是数据被 Flink 框架处理时机器的系统时间，这个时间本身存在不确定性，比如因为网络延迟等原因。</p>
<p data-nodeid="140499">摄入时间（Ingestion Time）理论上处于事件时间（Event Time）和处理时间（Processing Time）之间，可以用来防止 Flink 内部处理数据发生乱序的情况，但是无法解决数据进入 Flink 之前的乱序行为。</p>
<p data-nodeid="140500">所以我们一般都会用 EventTime、WaterMark 和窗口配合使用来解决消息的乱序和延迟问题。</p>
<h4 data-nodeid="140501">水印的本质是时间戳</h4>
<p data-nodeid="140502">水印的本质是一个一个的时间戳，这个时间戳存在 DataStream 的数据流中，Watermark 的生成方式有两种：</p>
<ul data-nodeid="140503">
<li data-nodeid="140504">
<p data-nodeid="140505">AssignerWithPeriodicWatermarks 生成周期水印，周期默认的时间是 200ms；</p>
</li>
<li data-nodeid="140506">
<p data-nodeid="140507">AssignerWithPunctuatedWatermarks 按需生成水印。</p>
</li>
</ul>
<p data-nodeid="140508">当 Flink 系统中出现了一个 Watermark T，那么就意味着 EventTime &lt;= T 的数据都已经到达。当 Wartermark T 通过窗口后，后续到来的迟到数据就会被丢弃。</p>
<h4 data-nodeid="140509">窗口触发和乱序时间</h4>
<p data-nodeid="140510">Flink 在用时间 + 窗口 + 水印来解决实际生产中的数据乱序问题，有如下的触发条件：</p>
<ul data-nodeid="140511">
<li data-nodeid="140512">
<p data-nodeid="140513">watermark 时间 &gt;= window_end_time；</p>
</li>
<li data-nodeid="140514">
<p data-nodeid="140515">在 [window_start_time,window_end_time) 中有数据存在，这个窗口是左闭右开的。</p>
</li>
</ul>
<p data-nodeid="140516">但是有些业务场景需要我们等待一段时间，也就是接受一定范围的迟到数据，此时 allowedLateness 的设置就显得尤为重要。简单地说，allowedLateness 的设置就是对于那些水印通过窗口的结束时间后，还允许等待一段时间。</p>
<p data-nodeid="140517">如果业务中的实际数据因为网络原因，乱序现象非常严重，allowedLateness 允许迟到的时间如果设置太小，则会导致很多次极少量数据触发窗口计算，严重影响数据的正确性。</p>
<h3 data-nodeid="140518">Flink 消费 Kafka 保证消息有序</h3>
<p data-nodeid="140519">我们在第 23 课时“Mock Kafka 消息并发送”中提过，可以认为 Kafka 中的一个 Topic 就是一个队列，每个 Topic 又会被分成多个 Partition，每个 Partition 中的消息是有序的。但是有的业务场景需要我们保障所有 Partition 中的消息有序，一般情况下需要把 Partition 的个数设置为一个，但这种情况是不能接受的，会严重影响数据的吞吐量。</p>
<p data-nodeid="140520">但是，Flink 消费 Kafka 时可以做到数据的全局有序，也可以多个 Partition 并发消费，这就是 Flink 中的 Kafka-partition-aware 特性。</p>
<p data-nodeid="146536">我们在使用这种特性生成水印时，水印会在 Flink 消费 Kafka 的消费端生成，并且每个分区的时间戳严格升序。当数据进行 Shuffle 时，水印的合并机制会产生全局有序的水印。</p>
<p data-nodeid="146537" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/2F/45/Ciqc1F8GvWiAV75CAAD0qegtgIs264.png" alt="image (2).png" data-nodeid="146545"></p>


<p data-nodeid="140523">我们从上图中可以看出，每个生成的水印是如何在多个分区的数据中进行传递的。</p>
<p data-nodeid="140524">代码实现如下：</p>
<pre class="lang-java" data-nodeid="140525"><code data-language="java">FlinkKafkaConsumer09&lt;MyType&gt; kafkaSource = <span class="hljs-keyword">new</span> FlinkKafkaConsumer09&lt;&gt;(<span class="hljs-string">"topic"</span>, schema, props);
kafkaSource.assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> AscendingTimestampExtractor&lt;MyType&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractAscendingTimestamp</span><span class="hljs-params">(MyType element)</span> </span>{
        <span class="hljs-keyword">return</span> element.eventTimestamp();
    }
});
DataStream&lt;MyType&gt; stream = env.addSource(kafkaSource);
</code></pre>
<h3 data-nodeid="140526">Flink 预定义的时间戳提取器和水印发射器</h3>
<p data-nodeid="140527">Flink 本身提供了两个预定义实现类去生成水印：</p>
<ul data-nodeid="140528">
<li data-nodeid="140529">
<p data-nodeid="140530">AscendingTimestampExtractor&nbsp;时间戳递增</p>
</li>
<li data-nodeid="140531">
<p data-nodeid="140532">BoundedOutOfOrdernessTimestampExtractor&nbsp;处理乱序消息和延迟时间</p>
</li>
</ul>
<h4 data-nodeid="140533">AscendingTimestampExtractor 递增时间戳提取器</h4>
<p data-nodeid="140534">AscendingTimestampExtractor 是周期性生成水印的一个简单实现，这种方式会产生严格递增的水印。它的实现如下：</p>
<pre class="lang-java" data-nodeid="140535"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AscendingTimestampExtractor</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">AssignerWithPeriodicWatermarks</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
...
   <span class="hljs-function"><span class="hljs-keyword">public</span> AscendingTimestampExtractor&lt;T&gt; <span class="hljs-title">withViolationHandler</span><span class="hljs-params">(MonotonyViolationHandler handler)</span> </span>{
      <span class="hljs-keyword">this</span>.violationHandler = requireNonNull(handler);
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
   }
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(T element, <span class="hljs-keyword">long</span> elementPrevTimestamp)</span> </span>{
      <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> newTimestamp = extractAscendingTimestamp(element);
      <span class="hljs-keyword">if</span> (newTimestamp &gt;= <span class="hljs-keyword">this</span>.currentTimestamp) {
         <span class="hljs-keyword">this</span>.currentTimestamp = newTimestamp;
         <span class="hljs-keyword">return</span> newTimestamp;
      } <span class="hljs-keyword">else</span> {
         violationHandler.handleViolation(newTimestamp, <span class="hljs-keyword">this</span>.currentTimestamp);
         <span class="hljs-keyword">return</span> newTimestamp;
      }
   }
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Watermark <span class="hljs-title">getCurrentWatermark</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Watermark(currentTimestamp == Long.MIN_VALUE ? Long.MIN_VALUE : currentTimestamp - <span class="hljs-number">1</span>);
   }
...
}
</code></pre>
<p data-nodeid="140536">该种水印的生成方式适用于那些数据本身的时间戳在每个并行的任务中是单调递增的，例如，我们上面使用 AscendingTimestampExtractor 处理 Kafka 多个 Partition 的情况。</p>
<p data-nodeid="140537">一个简单的案例如下所示：</p>
<pre class="lang-java" data-nodeid="140538"><code data-language="java">DataStream&lt;MyEvent&gt; stream = ...
DataStream&lt;MyEvent&gt; withTimestampsAndWatermarks =
    stream.assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> AscendingTimestampExtractor&lt;MyEvent&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractAscendingTimestamp</span><span class="hljs-params">(MyEvent element)</span> </span>{
            <span class="hljs-keyword">return</span> element.getCreationTime();
        }
});
</code></pre>
<h4 data-nodeid="140539">BoundedOutOfOrdernessTimestampExtractor 允许特定数量延迟的提取器</h4>
<p data-nodeid="140540">我们在上面提过有些业务场景需要等待一段时间，也就是接受一定范围的迟到数据，此时 allowedLateness 的设置就显得尤为重要。这种提取器也是周期性生成水印的实现，接受 allowedLateness 作为参数。</p>
<p data-nodeid="140541">它的实现如下：</p>
<pre class="lang-java" data-nodeid="140542"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BoundedOutOfOrdernessTimestampExtractor</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">AssignerWithPeriodicWatermarks</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
...
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> maxOutOfOrderness;
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">BoundedOutOfOrdernessTimestampExtractor</span><span class="hljs-params">(Time maxOutOfOrderness)</span> </span>{
      <span class="hljs-keyword">if</span> (maxOutOfOrderness.toMilliseconds() &lt; <span class="hljs-number">0</span>) {
         <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RuntimeException(<span class="hljs-string">"Tried to set the maximum allowed "</span> +
            <span class="hljs-string">"lateness to "</span> + maxOutOfOrderness + <span class="hljs-string">". This parameter cannot be negative."</span>);
      }
      <span class="hljs-keyword">this</span>.maxOutOfOrderness = maxOutOfOrderness.toMilliseconds();
      <span class="hljs-keyword">this</span>.currentMaxTimestamp = Long.MIN_VALUE + <span class="hljs-keyword">this</span>.maxOutOfOrderness;
   }
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">getMaxOutOfOrdernessInMillis</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> maxOutOfOrderness;
   }
...
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Watermark <span class="hljs-title">getCurrentWatermark</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-comment">// this guarantees that the watermark never goes backwards.</span>
      <span class="hljs-keyword">long</span> potentialWM = currentMaxTimestamp - maxOutOfOrderness;
      <span class="hljs-keyword">if</span> (potentialWM &gt;= lastEmittedWatermark) {
         lastEmittedWatermark = potentialWM;
      }
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Watermark(lastEmittedWatermark);
   }
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(T element, <span class="hljs-keyword">long</span> previousElementTimestamp)</span> </span>{
      <span class="hljs-keyword">long</span> timestamp = extractTimestamp(element);
      <span class="hljs-keyword">if</span> (timestamp &gt; currentMaxTimestamp) {
         currentMaxTimestamp = timestamp;
      }
      <span class="hljs-keyword">return</span> timestamp;
   }
}
</code></pre>
<p data-nodeid="140543">BoundedOutOfOrdernessTimestampExtractor 的构造器接收 maxOutOfOrderness 这个参数，该参数是指定我们接收的消息允许滞后的最大时间。</p>
<h3 data-nodeid="140544">案例</h3>
<p data-nodeid="140545">下面是一个接收 Kafka 消息进行处理，自定义窗口和水印的案例：</p>
<pre class="lang-java" data-nodeid="140546"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WindowWaterMark</span> </span>{

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();
        <span class="hljs-comment">//设置为eventtime事件类型</span>
        env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
        <span class="hljs-comment">//设置水印生成时间间隔100ms</span>
        env.getConfig().setAutoWatermarkInterval(<span class="hljs-number">100</span>);
        DataStream&lt;String&gt; dataStream = env
                .socketTextStream(<span class="hljs-string">"127.0.0.1"</span>, <span class="hljs-number">9000</span>)
                .assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> AssignerWithPeriodicWatermarks&lt;String&gt;() {
                    <span class="hljs-keyword">private</span> Long currentTimeStamp = <span class="hljs-number">0L</span>;
                    <span class="hljs-comment">//设置允许乱序时间</span>
                    <span class="hljs-keyword">private</span> Long maxOutOfOrderness = <span class="hljs-number">5000L</span>;
                    <span class="hljs-meta">@Override</span>
                    <span class="hljs-function"><span class="hljs-keyword">public</span> Watermark <span class="hljs-title">getCurrentWatermark</span><span class="hljs-params">()</span> </span>{
                        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Watermark(currentTimeStamp - maxOutOfOrderness);
                    }
                    <span class="hljs-meta">@Override</span>
                    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(String s, <span class="hljs-keyword">long</span> l)</span> </span>{
                        String[] arr = s.split(<span class="hljs-string">","</span>);
                        <span class="hljs-keyword">long</span> timeStamp = Long.parseLong(arr[<span class="hljs-number">1</span>]);
                        currentTimeStamp = Math.max(timeStamp, currentTimeStamp);
                        System.err.println(s + <span class="hljs-string">",EventTime:"</span> + timeStamp + <span class="hljs-string">",watermark:"</span> + (currentTimeStamp - maxOutOfOrderness));
                        <span class="hljs-keyword">return</span> timeStamp;
                    }
                });
        dataStream.map(<span class="hljs-keyword">new</span> MapFunction&lt;String, Tuple2&lt;String, Long&gt;&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Tuple2&lt;String, Long&gt; <span class="hljs-title">map</span><span class="hljs-params">(String s)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                String[] split = s.split(<span class="hljs-string">","</span>);
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Tuple2&lt;String, Long&gt;(split[<span class="hljs-number">0</span>], Long.parseLong(split[<span class="hljs-number">1</span>]));
            }
        })
                .keyBy(<span class="hljs-number">0</span>)
                .window(TumblingEventTimeWindows.of(Time.seconds(<span class="hljs-number">5</span>)))
                .aggregate(<span class="hljs-keyword">new</span> AggregateFunction&lt;Tuple2&lt;String,Long&gt;, Object, Object&gt;() {
                    ...
                })
                .print();
        env.execute(<span class="hljs-string">"WaterMark Test Demo"</span>);
    }<span class="hljs-comment">//</span>
}
</code></pre>
<p data-nodeid="140547">在这个案例中，我们使用的 AssignerWithPeriodicWatermarks 来自定义水印发射器和时间戳提取器，设置允许乱序时间为 5 秒，并且在一个 5 秒的窗口内进行聚合计算。<br>
在这个案例中，可以看到如何正确使用 Flink 提供的 API 进行水印和时间戳的设置。</p>
<h3 data-nodeid="140548">总结</h3>
<p data-nodeid="140884">这一课时讲解了生产环境中正确使用 Watermark 需要注意的事项，并且介绍了如何保证 Kafka 消息的全局有序，Flink 中自定义的时间戳提取器和水印发射器；最后用一个案例讲解了如何正确使用水印和设置乱序事件。通过这一课时你可以学习到生产中设置水印的正确方法和原理。</p>

---

### 精选评论

##### **峰：
> 王老师，请教一下，消费kakfka，分区与分区之间的有序性是如何保证的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Kafka的单个分区是有序存储的，Flink会顺序消费。但是多个分区之间是不保障有序的。

##### **蜗牛：
> 老师，aggregate 和 apply 方法处理计算有何区别的么？？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第一个是用来做增量计算，第二个做全脸聚合。

##### **飞：
> 我现在四个topic进行leftjoin关联，采用Stream API中的cogroup进行转换操作。stream_one = A.cogroup(b).where(a).equalTo(b).window(slideeventtimewindow(窗口1小时，滑动1分钟)).apply(逻辑计算);然后 stream_two= stream_one.cogroup(c).where(stream_one).equalTo(c).window(slideeventtimewindow(窗口1小时，滑动1分钟)).apply(逻辑计算);然后 stream_three= stream_two.cogroup(d).where(stream_two).equalTo(d).window(slideeventtimewindow(窗口1小时，滑动1分钟)).apply(逻辑计算);最终得到stream_three进行sink输出。不知道这种写法能否优化，感觉每次关联设置窗口，最终输出延迟时间应该是3分钟，这种多流join上有没有更优的方案？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 事实上我们在生产环境大于2个流进行join时，不会用flink的原生join，存在的问题：1. 快慢流导致state存储过大 2.任务一旦失败，state丢失会造成数据不准确，且无法重跑。我们一般是借助外部存储，比如将4个topic全部用flink任务写入hbase，然后在java中进行关联。

