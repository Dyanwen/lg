<p data-nodeid="725" class="">在上一课时中，我们学习了 Spark Streaming 的抽象、架构以及数据处理方式，但是流处理的输入是动态的数据源，假设在出错是常态的情况下，如何在动态的数据流中仍然兼顾恰好一次的消息送达保证（结果正确性），是生产环境中必须考虑的问题。</p>
<p data-nodeid="726">本课时的主要内容有：</p>
<ul data-nodeid="727">
<li data-nodeid="728">
<p data-nodeid="729">输入和输出</p>
</li>
<li data-nodeid="730">
<p data-nodeid="731">容错与结果正确性</p>
</li>
</ul>
<h3 data-nodeid="732">输入和输出</h3>
<p data-nodeid="906" class="te-preview-highlight">Spark Streaming 作为一个流处理系统，对接了很多输入源，除了一些实验性质的输入源，如ConstantInputDStream（每批次都为常数集合）、socketTextStream（监听套接字作为输入）、textFileStream（本地文件作为输入，常常用来监控文件夹），在生产环境中用得最多的还是 Kafka 这类消息队列。在本课时中，我们选用 Kafka 0.8 版本，介绍 Spark Streaming 与 Kafka 集成，这是在生产环境中最常见的一种情况。</p>

<p data-nodeid="734">在 Spark 中，为连接 Kafka 提供了两种方式，即基于 Receiver 的方式和 Kafka Direct API。这两种方式在使用上大同小异，但原理却截然不同，先来看看基于 Receiver 的方式：</p>
<pre class="lang-scala" data-nodeid="735"><code data-language="scala">...
<span class="hljs-keyword">val</span>&nbsp;kafkaParams&nbsp;=&nbsp;<span class="hljs-type">Map</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">Object</span>](
&nbsp;&nbsp;<span class="hljs-string">"bootstrap.servers"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"localhost:9092,anotherhost:9092"</span>,
&nbsp;&nbsp;<span class="hljs-string">"key.deserializer"</span>&nbsp;-&gt;&nbsp;classOf[<span class="hljs-type">StringDeserializer</span>],
&nbsp;&nbsp;<span class="hljs-string">"value.deserializer"</span>&nbsp;-&gt;&nbsp;classOf[<span class="hljs-type">StringDeserializer</span>],
&nbsp;&nbsp;<span class="hljs-string">"group.id"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"groupId"</span>,
&nbsp;&nbsp;<span class="hljs-string">"auto.offset.reset"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"latest"</span>,
&nbsp;&nbsp;<span class="hljs-string">"enable.auto.commit"</span>&nbsp;-&gt;&nbsp;(<span class="hljs-literal">true</span>:&nbsp;java.lang.<span class="hljs-type">Boolean</span>)
)
&nbsp;
<span class="hljs-keyword">val</span>&nbsp;topics&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-string">"topicA"</span>,&nbsp;<span class="hljs-string">"topicB"</span>)
<span class="hljs-keyword">val</span>&nbsp;messages&nbsp;=&nbsp;<span class="hljs-type">KafkaUtils</span>.createDirectStream[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](
&nbsp;&nbsp;ssc,
&nbsp;&nbsp;<span class="hljs-type">PreferConsistent</span>,
&nbsp;&nbsp;<span class="hljs-type">Subscribe</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](topics,&nbsp;kafkaParams)
)
&nbsp;
messages.map(record&nbsp;=&gt;&nbsp;(record.key,&nbsp;record.value))
</code></pre>
<p data-nodeid="736">用这种方式来与 Kafka 集成，配置中设置了 enable.auto.commit 为 true，表明自己不需要维护 offset，而是由 Kafka 自己来维护（在 Kafka 0.10 后，默认的 offset 存储位置改为了 Kafka，实际上就是 Kafka 的一个 topic），Kafka 消费者会周期性地（默认为 5s）去修改偏移量。这种方式接收的数据都保存在 Receiver 中，一旦出现意外，数据就有可能丢失，要想避免丢失的情况，就必须采用 WAL（Write Ahead Log，预写日志）机制，在数据写入内存前先进行持久化。</p>
<p data-nodeid="737">现在我们来试想一种情况，数据从 Kafka 取出后，进行了 WAL，在这个时候，Driver 与 Executor 因为某种原因宕机，这时最新偏移量还没来得及提交，那么在 Driver 恢复后，会从记录的偏移量继续消费数据并处理 WAL 的数据，这样一来，被 WAL 持久化的数据就会被重复计算一次。因此，开启了 WAL 后，这样的容错机制最多只能实现“至少一次”的消息送达语义。而且开启 WAL 后，增加了 I/O 开销，降低了 Spark Streaming 的吞吐量，还会产生冗余存储。这个过程如下图所示。</p>
<p data-nodeid="738"><img src="https://s0.lgstatic.com/i/image/M00/22/47/Ciqc1F7sH4OAQpACAAQQ-fsjH8c019.png" alt="11.png" data-nodeid="812"></p>
<p data-nodeid="739">如果业务场景对“恰好一次”的消息送达语义有着强烈的需求，那么基于 Receiver 的方式是无法满足的，基于 Spark Streaming 提供的 Direct API 形式，克服了这一缺点。Direct API 是 Spark 1.3 后增加的新特性，相比基于 Receiver 方法的“间接”，这种方式更加“直接”。</p>
<p data-nodeid="740">在这种方式中，消费者会定期轮询 Kafka，得到在每个 topic 中每个分区的最新偏移量，根据这个偏移量来确定每个批次的范围，这个信息会记录在 Checkpoint 中，当作业启动时，会根据这个范围用消费者 API 直接获取数据。这样的话，就相当于把 Kafka 变成了一个文件系统，而 offset 的范围就是文件地址，Direct API 用这种方式将流式数据源变成了静态数据源，再利用 Spark 本身的 DAG 容错机制，使所有计算失败的数据均可溯源，从而实现了“恰好一次”的消息送达语义。**请注意，Direct API 不需要采用WAL预写日志机制，因为所有数据都相当于在 Kafka 中被持久化了，作业恢复后直接从 Kafka 读取即可，**如下图所示：</p>
<p data-nodeid="741"><img src="https://s0.lgstatic.com/i/image/M00/22/53/CgqCHl7sH5WAX7rOAAMNV673x94978.png" alt="12.png" data-nodeid="823"></p>
<p data-nodeid="742">这种方式带来的优点显而易见，不仅克服了 WAL 带来的效率缺陷，还简化了并行性，使用 Direct API，Spark Streaming 会为每个 Kafka 的分区创建对应的 RDD 分区，这样就不需要使用 ssc.union() 方法来进行合并了，这也便于理解和调优。另外，这样的架构还保证了<strong data-nodeid="829">输入-处理阶段</strong>的“恰好一次”的消息送达语义，这就类似于消息的“回放”，虽然目前 Kafka 本身不支持消息回放，但用这种方式间接地实现了消息回放的功能。下面我们来看一个使用 Direct API 的完整例子：</p>
<pre class="lang-scala" data-nodeid="743"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.<span class="hljs-type">SparkConf</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.<span class="hljs-type">StreamingContext</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.<span class="hljs-type">Seconds</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.kafka010.<span class="hljs-type">LocationStrategies</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.kafka010.<span class="hljs-type">KafkaUtils</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.kafka010.<span class="hljs-type">ConsumerStrategies</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.kafka010.<span class="hljs-type">HasOffsetRanges</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.streaming.kafka010.<span class="hljs-type">CanCommitOffsets</span>
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;SparkStreamingKafkaDirexct&nbsp;</span></span>{

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>])&nbsp;{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span>
&nbsp;&nbsp;&nbsp;&nbsp;.builder
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"SparkStreamingKafkaDirexct"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate()

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;sc&nbsp;=&nbsp;spark.sparkContext

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;ssc&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StreamingContext</span>(sc,&nbsp;batchDuration&nbsp;=&nbsp;<span class="hljs-type">Seconds</span>(<span class="hljs-number">2</span>))

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Kafka的topic</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;topics&nbsp;=&nbsp;args(<span class="hljs-number">2</span>)

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;topicsSet:&nbsp;<span class="hljs-type">Set</span>[<span class="hljs-type">String</span>]&nbsp;=&nbsp;topics.split(<span class="hljs-string">","</span>).toSet

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Kafka配置参数</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;kafkaParams:&nbsp;<span class="hljs-type">Map</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">Object</span>]&nbsp;=&nbsp;<span class="hljs-type">Map</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"metadata.broker.list"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"kafka01:9092,kafka02:9092,kafka03:9092"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"group.id"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"apple_sample"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"serializer.class"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"kafka.serializer.StringEncoder"</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;自动将偏移重置为最新的偏移，如果是第一次启动程序，应该为smallest，从头开始读</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"auto.offset.reset"</span>&nbsp;-&gt;&nbsp;<span class="hljs-string">"latest"</span>
&nbsp;&nbsp;&nbsp;&nbsp;)&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;用Kafka&nbsp;Direct&nbsp;API直接读数据</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;messages&nbsp;=&nbsp;<span class="hljs-type">KafkaUtils</span>.createDirectStream[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ssc,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">LocationStrategies</span>.<span class="hljs-type">PreferConsistent</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">ConsumerStrategies</span>.<span class="hljs-type">Subscribe</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](topicsSet,&nbsp;kafkaParams)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;在该批次数据处理完之后，将该offset提交给Kafka，“...”代表用户自己定义的处理逻辑</span>
&nbsp;&nbsp;&nbsp;&nbsp;messages.map(...).foreachRDD(mess&nbsp;=&gt;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取offset集合</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;offsetsList&nbsp;=&nbsp;mess.asInstanceOf[<span class="hljs-type">HasOffsetRanges</span>].offsetRanges
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asInstanceOf[<span class="hljs-type">CanCommitOffsets</span>].commitAsync(offsetsList)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;)

&nbsp;&nbsp;&nbsp;&nbsp;ssc.start()
&nbsp;&nbsp;&nbsp;&nbsp;ssc.awaitTermination()

&nbsp;&nbsp;}

}
</code></pre>
<p data-nodeid="744">从上面这段代码中我们可以发现，首先关闭了自动提交偏移量，改由手动维护。然后再从最新的偏移量开始生成 RDD，经过各种转换算子处理后输出结果，最后用 commitAsync 异步向 Kafka 提交最新的偏移量。一旦使用了 Direct API，用户需要追踪到结果数据输出完成后，再提交偏移量的改动，否则会造成不确定的影响。使用这种方式，无法在事务层面保证<strong data-nodeid="835">处理-输出这个阶段</strong>做到“恰好一次”，因此只能采用输出幂等的方式来达到同样的效果。</p>
<p data-nodeid="745">如果想要在事务的层面，让<strong data-nodeid="841">处理-输出这个阶段</strong>做到“恰好一次”，那么可以将 Kafka 的偏移量与最终结果存储在同一个数据库实例上，这就需要修改代码，一开始，需要从外部数据库上获取最新的偏移量：</p>
<pre class="lang-scala" data-nodeid="746"><code data-language="scala">...
<span class="hljs-comment">//从外部数据库获取偏移量</span>
<span class="hljs-keyword">val</span>&nbsp;fromOffsets:&nbsp;<span class="hljs-type">Map</span>[<span class="hljs-type">TopicPartition</span>,&nbsp;<span class="hljs-type">Long</span>]&nbsp;=&nbsp;setFromOffsets(offsetList)
...
<span class="hljs-comment">//&nbsp;用最新的offset得到初始化RDD</span>
<span class="hljs-keyword">val</span>&nbsp;messages&nbsp;=&nbsp;<span class="hljs-type">KafkaUtils</span>.createDirectStream[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](ssc,
<span class="hljs-type">LocationStrategies</span>.<span class="hljs-type">PreferConsistent</span>,
<span class="hljs-type">ConsumerStrategies</span>.<span class="hljs-type">Subscribe</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>](topicsSet,&nbsp;kafkaParams,&nbsp;fromOffsets))
</code></pre>
<p data-nodeid="747">在最后输出的操作里，由于偏移量与最终数据处理结果要保存到同一个数据库，因此可以利用外部数据库的事务特性，完成最后的工作：</p>
<pre class="lang-scala" data-nodeid="748"><code data-language="scala">messages.map(…).foreachRDD(mess&nbsp;=&gt;&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取offset集合</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;offsetsList&nbsp;=&nbsp;mess.asInstanceOf[<span class="hljs-type">HasOffsetRanges</span>].offsetRanges
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将修改offset与输出最后结果作为一个事务提交</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;transaction{</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yourUpdateTheOffset(offsetsList)</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yourOutputToDatabase(mess)</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;}</span>
}
)
</code></pre>
<p data-nodeid="749">这样一来，Spark Streaming 才算是真正实现了端到端的消息送达保证。</p>
<p data-nodeid="750"><strong data-nodeid="847">在实际开发中，将偏移量和输出结果存储到同一个外部数据库的方式用得并不多，因为这会使业务数据与消息数据耦合在一起，结构不够优雅，反而幂等输出更加流行。</strong></p>
<p data-nodeid="751">最后，来看看 Spark Streaming 的输出操作。Spark Streaming 也是懒加载模式，同样需要类似于 RDD 的行动算子才能真正开始运行，在 Spark Streaming 中，我们称其为输出算子，一共有下面这几种。</p>
<ul data-nodeid="752">
<li data-nodeid="753">
<p data-nodeid="754">print()：打印 DStream 中每个批次的前十个元素。</p>
</li>
<li data-nodeid="755">
<p data-nodeid="756">saveAsTextFiles(prefix, [suffix])：将 DStream 中的内容保存为文本文件。</p>
</li>
<li data-nodeid="757">
<p data-nodeid="758">saveAsObjectFiles(prefix, [suffix])：将 DStream 中的内容保存为 Java 序列化对象的 SequenceFile。</p>
</li>
<li data-nodeid="759">
<p data-nodeid="760">saveAsHadoopFiles(prefix, [suffix])：将 DStream 中的内容保存为 Hadoop 序列化格式（Writable）的文件，可以指定 K、V 类型。</p>
</li>
<li data-nodeid="761">
<p data-nodeid="762">foreachRDD(func)：该算子是 Spark Streaming 独有的，与 transform 算子类似，都是直接可以操作 RDD，我们可以利用该算子来做一些处理工作，例如生成 Parquet 文件写入 HDFS、将数据插入到外部数据库中，如 HBase、Elasticsearch。</p>
</li>
</ul>
<h3 data-nodeid="763">容错与结果正确性</h3>
<p data-nodeid="764">介绍了 Spark Streaming 的架构、用法之后，在本课时中，将会讨论 Spark Streaming 的容错机制，以及结果的正确性保证。要想 Spark Streaming 应用能够全天候无间断地运行，需要利用 Spark 自带的 Checkpoint 容错机制。Checkpoint 会在 Spark Streaming 运行过程中，周期性地保存一些作业相关信息，这样才能让 Spark Streaming 作业从故障（例如系统故障、JVM 崩溃等）中恢复。<strong data-nodeid="871">值得注意的是，作为 Checkpoint 的存储系统，是必须保证高可用的，常见的如 HDFS 就很可靠，更优的选择则是 Alluxio。</strong></p>
<p data-nodeid="765">Checkpoint 主要保存了以下两类信息，其中元数据检查点主要用来恢复 Driver 程序，数据检查点主要用来恢复 Executor 程序。下面我们来分别介绍一下这两类信息：</p>
<ol data-nodeid="766">
<li data-nodeid="767">
<p data-nodeid="768">元数据检查点。元数据主要包括。</p>
</li>
</ol>
<ul data-nodeid="769">
<li data-nodeid="770">
<p data-nodeid="771">配置：创建该 Spark Streaming 应用的配置。</p>
</li>
<li data-nodeid="772">
<p data-nodeid="773">DStream 算子：Spark Streaming 作业中定义的算子。</p>
</li>
<li data-nodeid="774">
<p data-nodeid="775">未完成的批次：那些还在作业队列里未完成的批次。</p>
</li>
</ul>
<p data-nodeid="776">Checkpoint 会周期性地将这些信息保存至外部可靠存储（如 HDFS、Alluxio）。</p>
<ol start="2" data-nodeid="777">
<li data-nodeid="778">
<p data-nodeid="779">数据检查点。</p>
</li>
</ol>
<p data-nodeid="780">将中间生成的 RDD 保存到可靠的外部存储中。我们在上一节中讨论过，如果要使用状态管理的算子，如 updateStateByKey、mapWithState 等，就必须开启 Checkpoint 机制，因为这类算子必须保存中间结果，以供下次计算使用。另外，我们知道 Spark 本身的容错机制是依靠 RDD DAG 的依赖关系通过计算恢复的，但是这也会造成依赖链过长、恢复时间过长的问题，因此我们必须周期性地存储中间结果（状态）至可靠的外部存储来缩短依赖链。我们也可以手动调用 DStream 的 checkpoint 算子进行缓存。如下：</p>
<pre class="lang-scala" data-nodeid="781"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;ds:&nbsp;<span class="hljs-type">DStream</span>[<span class="hljs-type">Int</span>]&nbsp;=&nbsp;...
<span class="hljs-keyword">val</span>&nbsp;cds:&nbsp;<span class="hljs-type">DStream</span>[<span class="hljs-type">Int</span>]&nbsp;=&nbsp;ds.checkpoint(<span class="hljs-type">Seconds</span>(<span class="hljs-number">5</span>))
</code></pre>
<p data-nodeid="782">Checkpoint 机制会按照设置的间隔对 DStream 进行持久化。如果需要启用 Checkpoint 机制，需要对代码做如下改动：</p>
<pre class="lang-scala" data-nodeid="783"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;checkpointDirectory&nbsp;=&nbsp;<span class="hljs-string">"/your_cp_path"</span>&nbsp;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;functionToCreateContext</span></span>():&nbsp;<span class="hljs-type">StreamingContext</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;conf&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">SparkConf</span>().setMaster(<span class="hljs-string">"local[*]"</span>).setAppName(<span class="hljs-string">"Checkpoint"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;ssc&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StreamingContext</span>(conf,<span class="hljs-type">Seconds</span>(<span class="hljs-number">1</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;lines&nbsp;=&nbsp;ssc.socketTextStream(<span class="hljs-string">"localhost"</span>,<span class="hljs-number">9999</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ssc.checkpoint(checkpointDirectory)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ssc
}

<span class="hljs-keyword">val</span>&nbsp;context&nbsp;=&nbsp;<span class="hljs-type">StreamingContext</span>.getOrCreate(checkpointDirectory,&nbsp;
 functionToCreateContext&nbsp;_)
</code></pre>
<p data-nodeid="784">StreamingContext 需要以 getOrCreate 的方式初始化。这样就能保证，如果从故障中恢复，会获取到上一个检查点的信息。</p>
<p data-nodeid="785">如果数据源是文件，那么上面的方法可以保证完全不丢数据，因为所有的状态都可以根据持久化的数据源复现出来，但如果是流式数据源，想要保证不丢数据是很困难的。因为当 Driver 出故障的时候，有可能接收的数据会丢失，并且不能找回。为了解决这个问题，Spark 1.2 之后引入了预写日志（WAL），Spark Streaming WAL 指的是接收到数据后，在数据处理之前，先对数据进行持久化，完成这个工作的是 BlockManagerBasedBlockHandler 类的实现类WriteAheadLog BasedBlockHandler 。如果开启了 WAL，那么数据会先进行持久化再写到 Executor 的内存中。这样即使内存数据丢失了，在 Driver 恢复后，丢失的数据还是会被处理，这就实现了“至少一次”的消息送达语义。打开 WAL 的方式为：设置spark.streaming.receiver.writeAheadLog.enable 为 true。这种方法其实是对 Receiver 的容错。</p>
<p data-nodeid="786">那么 Checkpoint 就以这种形式完成了 Driver、Executor 和 Receiver 的容错。下面我们来讨论一下 Spark Streaming 计算结果的正确性。</p>
<p data-nodeid="787">先来看看消息送达保证，Spark Streaming 框架本身实现“恰好一次”的消息送达语义比较容易，因为 Spark Streaming 本质上还是进行的批处理，所以它只需在批的层面通过 BatchId 追踪数据处理情况，这和 Spark 是完全一致的，因此它完全能够保证一个批只被处理一次，当一个批没有被成功处理时，肯定就是发生了故障，这时 Checkpoint 机制能够保证从最近持久化的中间结果与待执行的计算任务（DStreamGraph）开始重新计算，保证数据只被处理一次，从而得到正确的结果，该阶段可以认为是<strong data-nodeid="889">处理阶段的消息送达保证</strong>。</p>
<p data-nodeid="788"><strong data-nodeid="893">在流处理场景下，容错问题与结果正确性问题不能孤立地来看待，而是需要考虑在出现故障的情况下如何能够保证结果的正确性。</strong></p>
<h3 data-nodeid="789">小结</h3>
<p data-nodeid="790">在本课时中，将流式处理流程抽象为输入-处理-输出，而基于这个流程，又将流程拆分为三个部分：</p>
<ul data-nodeid="791">
<li data-nodeid="792">
<p data-nodeid="793">输入-处理</p>
</li>
<li data-nodeid="794">
<p data-nodeid="795">处理</p>
</li>
<li data-nodeid="796">
<p data-nodeid="797">处理-输出</p>
</li>
</ul>
<p data-nodeid="798"><strong data-nodeid="903">而这每个部分，都需要假设错误会经常发生的情况下，还要保证“恰好一次”的消息送达保证，才是真正的端到端的消息送达保证，这也是生产环境中必须考虑的问题。</strong> 本课时从三个部分出发，给出了 Spark Streaming 的答案，其中处理-输出这个过程，通常会以幂等的方式解决，这也是在生产环境中非常常用的做法。</p>
<p data-nodeid="799">最后给你留一个思考题：</p>
<p data-nodeid="800" class="">用偏移量与最终数据处理结果保存到同一个数据库，这么做的缺点是什么？</p>

---

### 精选评论

##### **方：
> 引入数据库，要保证数据库的可用性，数据一致性问题，读取数据库会增加计算延时。

