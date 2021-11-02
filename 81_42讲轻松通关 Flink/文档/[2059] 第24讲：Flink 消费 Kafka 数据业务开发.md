<p data-nodeid="3">在上一课时中我们提过在实时计算的场景下，绝大多数的数据源都是消息系统，而 Kafka 从众多的消息中间件中脱颖而出，主要是因为<strong data-nodeid="70">高吞吐</strong>、<strong data-nodeid="71">低延迟</strong>的特点；同时也讲了 Flink 作为生产者像 Kafka 写入数据的方式和代码实现。这一课时我们将从以下几个方面介绍 Flink 消费 Kafka 中的数据方式和源码实现。</p>
<h3 data-nodeid="4">Flink 如何消费 Kafka</h3>
<p data-nodeid="5">Flink 在和 Kafka 对接的过程中，跟 Kafka 的版本是强相关的。上一课时也提到了，我们在使用 Kafka 连接器时需要引用相对应的 Jar 包依赖，对于某些连接器比如 Kafka 是有版本要求的，一定要去<a href="https://ci.apache.org/projects/flink/flink-docs-stable/dev/connectors/kafka.html" data-nodeid="76">官方网站</a>找到对应的依赖版本。</p>
<p data-nodeid="6">我们本地的 Kafka 版本是 2.1.0，所以需要对应的类是 FlinkKafkaConsumer。首先需要在 pom.xml 中引入 jar 包依赖：</p>
<pre class="lang-xml" data-nodeid="7"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-connector-kafka_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="8">下面将对 Flink 消费 Kafka 数据的方式进行分类讲解。</p>
<h4 data-nodeid="9">消费单个 Topic</h4>
<p data-nodeid="10">上一课时我们在本地搭建了 Kafka 环境，并且手动创建了名为 test 的 Topic，然后向名为 test 的 Topic 中写入了数据。</p>
<p data-nodeid="11">那么现在我们要消费这个 Topic 中的数据，该怎么做呢？</p>
<pre class="lang-java" data-nodeid="12"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
    env.enableCheckpointing(<span class="hljs-number">5000</span>);
    Properties properties = <span class="hljs-keyword">new</span> Properties();
    properties.setProperty(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"127.0.0.1:9092"</span>);
    <span class="hljs-comment">// 如果你是0.8版本的Kafka，需要配置</span>
    <span class="hljs-comment">//properties.setProperty("zookeeper.connect", "localhost:2181");</span>
    <span class="hljs-comment">//设置消费组</span>
    properties.setProperty(<span class="hljs-string">"group.id"</span>, <span class="hljs-string">"group_test"</span>);
    FlinkKafkaConsumer&lt;String&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(<span class="hljs-string">"test"</span>, <span class="hljs-keyword">new</span> SimpleStringSchema(), properties);
    <span class="hljs-comment">//设置从最早的ffset消费</span>
    consumer.setStartFromEarliest();
    <span class="hljs-comment">//还可以手动指定相应的 topic, partition，offset,然后从指定好的位置开始消费</span>
    <span class="hljs-comment">//HashMap&lt;KafkaTopicPartition, Long&gt; map = new HashMap&lt;&gt;();</span>
    <span class="hljs-comment">//map.put(new KafkaTopicPartition("test", 1), 10240L);</span>
    <span class="hljs-comment">//假如partition有多个，可以指定每个partition的消费位置</span>
    <span class="hljs-comment">//map.put(new KafkaTopicPartition("test", 2), 10560L);</span>
    <span class="hljs-comment">//然后各个partition从指定位置消费</span>
    <span class="hljs-comment">//consumer.setStartFromSpecificOffsets(map);</span>
    env.addSource(consumer).flatMap(<span class="hljs-keyword">new</span> FlatMapFunction&lt;String, String&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flatMap</span><span class="hljs-params">(String value, Collector&lt;String&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            System.out.println(value);
        }
    });
    env.execute(<span class="hljs-string">"start consumer..."</span>);
}
</code></pre>
<p data-nodeid="13">在设置消费 Kafka 中的数据时，可以显示地指定从某个 Topic 的每一个 Partition 中进行消费。</p>
<h4 data-nodeid="14">消费多个 Topic</h4>
<p data-nodeid="15">我们的业务中会有这样的情况，同样的数据根据类型不同发送到了不同的 Topic 中，比如线上的订单数据根据来源不同分别发往移动端和 PC 端两个 Topic 中。但是我们不想把同样的代码复制一份，需重新指定一个 Topic 进行消费，这时候应该怎么办呢？</p>
<pre class="lang-java" data-nodeid="16"><code data-language="java">Properties properties = <span class="hljs-keyword">new</span> Properties();
properties.setProperty(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"127.0.0.1:9092"</span>);
<span class="hljs-comment">// 如果你是0.8版本的Kafka，需要配置</span>
<span class="hljs-comment">//properties.setProperty("zookeeper.connect", "localhost:2181");</span>
<span class="hljs-comment">//设置消费组</span>
properties.setProperty(<span class="hljs-string">"group.id"</span>, <span class="hljs-string">"group_test"</span>);
FlinkKafkaConsumer&lt;String&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(<span class="hljs-string">"test"</span>, <span class="hljs-keyword">new</span> SimpleStringSchema(), properties);
ArrayList&lt;String&gt; topics = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
        topics.add(<span class="hljs-string">"test_A"</span>);
        topics.add(<span class="hljs-string">"test_B"</span>);
       <span class="hljs-comment">// 传入一个 list，完美解决了这个问题</span>
        FlinkKafkaConsumer&lt;Tuple2&lt;String, String&gt;&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(topics, <span class="hljs-keyword">new</span> SimpleStringSchema(), properties);
...
</code></pre>
<p data-nodeid="17">我们可以传入一个 list 来解决消费多个 Topic 的问题，如果用户需要区分两个 Topic 中的数据，那么需要在发往 Kafka 中数据新增一个字段，用来区分来源。</p>
<h4 data-nodeid="18">消息序列化</h4>
<p data-nodeid="19">我们在上述消费 Kafka 消息时，都默认指定了消息的序列化方式，即 SimpleStringSchema。这里需要注意的是，在我们使用 SimpleStringSchema 的时候，返回的结果中只有原数据，没有 topic、parition 等信息，这时候可以自定义序列化的方式来实现自定义返回数据的结构。</p>
<pre class="lang-java" data-nodeid="20"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomDeSerializationSchema</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">KafkaDeserializationSchema</span>&lt;<span class="hljs-title">ConsumerRecord</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>&gt;&gt; </span>{
    <span class="hljs-comment">//是否表示流的最后一条元素,设置为false，表示数据会源源不断地到来</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isEndOfStream</span><span class="hljs-params">(ConsumerRecord&lt;String, String&gt; nextElement)</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }
    <span class="hljs-comment">//这里返回一个ConsumerRecord&lt;String,String&gt;类型的数据，除了原数据还包括topic，offset，partition等信息</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> ConsumerRecord&lt;String, String&gt; <span class="hljs-title">deserialize</span><span class="hljs-params">(ConsumerRecord&lt;<span class="hljs-keyword">byte</span>[], <span class="hljs-keyword">byte</span>[]&gt; record)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ConsumerRecord&lt;String, String&gt;(
                record.topic(),
                record.partition(),
                record.offset(),
                <span class="hljs-keyword">new</span> String(record.key()),
                <span class="hljs-keyword">new</span> String(record.value())
        );
    }
    <span class="hljs-comment">//指定数据的输入类型</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> TypeInformation&lt;ConsumerRecord&lt;String, String&gt;&gt; getProducedType() {
        <span class="hljs-keyword">return</span> TypeInformation.of(<span class="hljs-keyword">new</span> TypeHint&lt;ConsumerRecord&lt;String, String&gt;&gt;(){});
    }
}
</code></pre>
<p data-nodeid="21">这里自定义了 CustomDeSerializationSchema 信息，就可以直接使用了。</p>
<h4 data-nodeid="22">Parition 和 Topic 动态发现</h4>
<p data-nodeid="23">在很多场景下，随着业务的扩展，我们需要对 Kafka 的分区进行扩展，为了防止新增的分区没有被及时发现导致数据丢失，消费者必须要感知 Partition 的动态变化，可以使用 FlinkKafkaConsumer 的动态分区发现实现。</p>
<p data-nodeid="24">我们只需要指定下面的配置，即可打开动态分区发现功能：每隔 10ms 会动态获取 Topic 的元数据，对于新增的 Partition 会自动从最早的位点开始消费数据。</p>
<pre class="lang-java" data-nodeid="25"><code data-language="java">properties.setProperty(FlinkKafkaConsumerBase.KEY_PARTITION_DISCOVERY_INTERVAL_MILLIS, <span class="hljs-string">"10"</span>);
</code></pre>
<p data-nodeid="26">如果业务场景需要我们动态地发现 Topic，可以指定 Topic 的正则表达式：</p>
<pre class="lang-java" data-nodeid="27"><code data-language="java">FlinkKafkaConsumer&lt;String&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(Pattern.compile(<span class="hljs-string">"^test_([A-Za-z0-9]*)$"</span>), <span class="hljs-keyword">new</span> SimpleStringSchema(), properties);
</code></pre>
<h4 data-nodeid="28">Flink 消费 Kafka 设置 offset 的方法</h4>
<p data-nodeid="29">Flink 消费 Kafka 需要指定消费的 offset，也就是<strong data-nodeid="100">偏移量</strong>。Flink 读取 Kafka 的消息有五种消费方式：</p>
<ul data-nodeid="30">
<li data-nodeid="31">
<p data-nodeid="32">指定 Topic 和 Partition</p>
</li>
<li data-nodeid="33">
<p data-nodeid="34">从最早位点开始消费</p>
</li>
<li data-nodeid="35">
<p data-nodeid="36">从指定时间点开始消费</p>
</li>
<li data-nodeid="37">
<p data-nodeid="38">从最新的数据开始消费</p>
</li>
<li data-nodeid="39">
<p data-nodeid="40">从上次消费位点开始消费</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="41"><code data-language="java"><span class="hljs-comment">/**
* Flink从指定的topic和parition中指定的offset开始
*/</span>
Map&lt;KafkaTopicPartition, Long&gt; offsets = <span class="hljs-keyword">new</span> HashedMap();
offsets.put(<span class="hljs-keyword">new</span> KafkaTopicPartition(<span class="hljs-string">"test"</span>, <span class="hljs-number">0</span>), <span class="hljs-number">10000L</span>);
offsets.put(<span class="hljs-keyword">new</span> KafkaTopicPartition(<span class="hljs-string">"test"</span>, <span class="hljs-number">1</span>), <span class="hljs-number">20000L</span>);
offsets.put(<span class="hljs-keyword">new</span> KafkaTopicPartition(<span class="hljs-string">"test"</span>, <span class="hljs-number">2</span>), <span class="hljs-number">30000L</span>);
consumer.setStartFromSpecificOffsets(offsets);
<span class="hljs-comment">/**
* Flink从topic中最早的offset消费
*/</span>
consumer.setStartFromEarliest();
<span class="hljs-comment">/**
* Flink从topic中指定的时间点开始消费
*/</span>
consumer.setStartFromTimestamp(<span class="hljs-number">1559801580000l</span>);
<span class="hljs-comment">/**
* Flink从topic中最新的数据开始消费
*/</span>
consumer.setStartFromLatest();
<span class="hljs-comment">/**
* Flink从topic中指定的group上次消费的位置开始消费，所以必须配置group.id参数
*/</span>
consumer.setStartFromGroupOffsets();
</code></pre>
<h3 data-nodeid="1077">源码解析</h3>
<p data-nodeid="1078" class=""><img src="https://s0.lgstatic.com/i/image/M00/2D/33/CgqCHl8C4CuABMKNAAFGiOmCWHA338.png" alt="Drawing 0.png" data-nodeid="1082"></p>


<p data-nodeid="44">从上面的类图可以看出，FlinkKafkaConsumer 继承了 FlinkKafkaConsumerBase，而 FlinkKafkaConsumerBase 最终是对 SourceFunction 进行了实现。</p>
<p data-nodeid="45">整体的流程：FlinkKafkaConsumer 首先创建了 KafkaFetcher 对象，然后 KafkaFetcher 创建了 KafkaConsumerThread 和 Handover，KafkaConsumerThread 负责直接从 Kafka 中读取 msg，并交给 Handover，然后 Handover 将 msg 传递给 KafkaFetcher.emitRecord 将消息发出。</p>
<p data-nodeid="46">因为 FlinkKafkaConsumerBase 实现了 RichFunction 接口，所以当程序启动的时候，会首先调用 FlinkKafkaConsumerBase.open 方法：</p>
<pre class="lang-java" data-nodeid="47"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration configuration)</span> <span class="hljs-keyword">throws</span> Exception </span>{
   <span class="hljs-comment">// 指定offset的提交方式</span>
   <span class="hljs-keyword">this</span>.offsetCommitMode = OffsetCommitModes.fromConfiguration(
         getIsAutoCommitEnabled(),
         enableCommitOnCheckpoints,
         ((StreamingRuntimeContext) getRuntimeContext()).isCheckpointingEnabled());
   <span class="hljs-comment">// 创建分区发现器</span>
   <span class="hljs-keyword">this</span>.partitionDiscoverer = createPartitionDiscoverer(
         topicsDescriptor,
         getRuntimeContext().getIndexOfThisSubtask(),
         getRuntimeContext().getNumberOfParallelSubtasks());
   <span class="hljs-keyword">this</span>.partitionDiscoverer.open();
   subscribedPartitionsToStartOffsets = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
   <span class="hljs-keyword">final</span> List&lt;KafkaTopicPartition&gt; allPartitions = partitionDiscoverer.discoverPartitions();
   <span class="hljs-keyword">if</span> (restoredState != <span class="hljs-keyword">null</span>) {
      <span class="hljs-keyword">for</span> (KafkaTopicPartition partition : allPartitions) {
         <span class="hljs-keyword">if</span> (!restoredState.containsKey(partition)) {
            restoredState.put(partition, KafkaTopicPartitionStateSentinel.EARLIEST_OFFSET);
         }
      }
      <span class="hljs-keyword">for</span> (Map.Entry&lt;KafkaTopicPartition, Long&gt; restoredStateEntry : restoredState.entrySet()) {
         <span class="hljs-keyword">if</span> (!restoredFromOldState) {
           
            <span class="hljs-keyword">if</span> (KafkaTopicPartitionAssigner.assign(
               restoredStateEntry.getKey(), getRuntimeContext().getNumberOfParallelSubtasks())
                  == getRuntimeContext().getIndexOfThisSubtask()){
               subscribedPartitionsToStartOffsets.put(restoredStateEntry.getKey(), restoredStateEntry.getValue());
            }
         } <span class="hljs-keyword">else</span> {
           subscribedPartitionsToStartOffsets.put(restoredStateEntry.getKey(), restoredStateEntry.getValue());
         }
      }
      <span class="hljs-keyword">if</span> (filterRestoredPartitionsWithCurrentTopicsDescriptor) {
         subscribedPartitionsToStartOffsets.entrySet().removeIf(entry -&gt; {
            <span class="hljs-keyword">if</span> (!topicsDescriptor.isMatchingTopic(entry.getKey().getTopic())) {
               LOG.warn(
                  <span class="hljs-string">"{} is removed from subscribed partitions since it is no longer associated with topics descriptor of current execution."</span>,
                  entry.getKey());
               <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
            }
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
         });
      }
      LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading {} partitions with offsets in restored state: {}"</span>,
         getRuntimeContext().getIndexOfThisSubtask(), subscribedPartitionsToStartOffsets.size(), subscribedPartitionsToStartOffsets);
   } <span class="hljs-keyword">else</span> {
    
      <span class="hljs-keyword">switch</span> (startupMode) {
         <span class="hljs-keyword">case</span> SPECIFIC_OFFSETS:
            <span class="hljs-keyword">if</span> (specificStartupOffsets == <span class="hljs-keyword">null</span>) {
               <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(
                  <span class="hljs-string">"Startup mode for the consumer set to "</span> + StartupMode.SPECIFIC_OFFSETS +
                     <span class="hljs-string">", but no specific offsets were specified."</span>);
            }
            <span class="hljs-keyword">for</span> (KafkaTopicPartition seedPartition : allPartitions) {
               Long specificOffset = specificStartupOffsets.get(seedPartition);
               <span class="hljs-keyword">if</span> (specificOffset != <span class="hljs-keyword">null</span>) {
                                 subscribedPartitionsToStartOffsets.put(seedPartition, specificOffset - <span class="hljs-number">1</span>);
               } <span class="hljs-keyword">else</span> {
               subscribedPartitionsToStartOffsets.put(seedPartition, KafkaTopicPartitionStateSentinel.GROUP_OFFSET);
               }
            }
            <span class="hljs-keyword">break</span>;
         <span class="hljs-keyword">case</span> TIMESTAMP:
            <span class="hljs-keyword">if</span> (startupOffsetsTimestamp == <span class="hljs-keyword">null</span>) {
               <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(
                  <span class="hljs-string">"Startup mode for the consumer set to "</span> + StartupMode.TIMESTAMP +
                     <span class="hljs-string">", but no startup timestamp was specified."</span>);
            }
            <span class="hljs-keyword">for</span> (Map.Entry&lt;KafkaTopicPartition, Long&gt; partitionToOffset
                  : fetchOffsetsWithTimestamp(allPartitions, startupOffsetsTimestamp).entrySet()) {
               subscribedPartitionsToStartOffsets.put(
                  partitionToOffset.getKey(),
                  (partitionToOffset.getValue() == <span class="hljs-keyword">null</span>)
                      KafkaTopicPartitionStateSentinel.LATEST_OFFSET
                        : partitionToOffset.getValue() - <span class="hljs-number">1</span>);
            }
            <span class="hljs-keyword">break</span>;
         <span class="hljs-keyword">default</span>:
            <span class="hljs-keyword">for</span> (KafkaTopicPartition seedPartition : allPartitions) {
               subscribedPartitionsToStartOffsets.put(seedPartition, startupMode.getStateSentinel());
            }
      }
      <span class="hljs-keyword">if</span> (!subscribedPartitionsToStartOffsets.isEmpty()) {
         <span class="hljs-keyword">switch</span> (startupMode) {
            <span class="hljs-keyword">case</span> EARLIEST:
               LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading the following {} partitions from the earliest offsets: {}"</span>,
                  getRuntimeContext().getIndexOfThisSubtask(),
                  subscribedPartitionsToStartOffsets.size(),
                  subscribedPartitionsToStartOffsets.keySet());
               <span class="hljs-keyword">break</span>;
            <span class="hljs-keyword">case</span> LATEST:
               LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading the following {} partitions from the latest offsets: {}"</span>,
                  getRuntimeContext().getIndexOfThisSubtask(),
                  subscribedPartitionsToStartOffsets.size(),
                  subscribedPartitionsToStartOffsets.keySet());
               <span class="hljs-keyword">break</span>;
            <span class="hljs-keyword">case</span> TIMESTAMP:
               LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading the following {} partitions from timestamp {}: {}"</span>,
                  getRuntimeContext().getIndexOfThisSubtask(),
                  subscribedPartitionsToStartOffsets.size(),
                  startupOffsetsTimestamp,
                  subscribedPartitionsToStartOffsets.keySet());
               <span class="hljs-keyword">break</span>;
            <span class="hljs-keyword">case</span> SPECIFIC_OFFSETS:
               LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading the following {} partitions from the specified startup offsets {}: {}"</span>,
                  getRuntimeContext().getIndexOfThisSubtask(),
                  subscribedPartitionsToStartOffsets.size(),
                  specificStartupOffsets,
                  subscribedPartitionsToStartOffsets.keySet());
               List&lt;KafkaTopicPartition&gt; partitionsDefaultedToGroupOffsets = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(subscribedPartitionsToStartOffsets.size());
               <span class="hljs-keyword">for</span> (Map.Entry&lt;KafkaTopicPartition, Long&gt; subscribedPartition : subscribedPartitionsToStartOffsets.entrySet()) {
                  <span class="hljs-keyword">if</span> (subscribedPartition.getValue() == KafkaTopicPartitionStateSentinel.GROUP_OFFSET) {
                     partitionsDefaultedToGroupOffsets.add(subscribedPartition.getKey());
                  }
               }
               <span class="hljs-keyword">if</span> (partitionsDefaultedToGroupOffsets.size() &gt; <span class="hljs-number">0</span>) {
                  LOG.warn(<span class="hljs-string">"Consumer subtask {} cannot find offsets for the following {} partitions in the specified startup offsets: {}"</span> +
                        <span class="hljs-string">"; their startup offsets will be defaulted to their committed group offsets in Kafka."</span>,
                     getRuntimeContext().getIndexOfThisSubtask(),
                     partitionsDefaultedToGroupOffsets.size(),
                     partitionsDefaultedToGroupOffsets);
               }
               <span class="hljs-keyword">break</span>;
            <span class="hljs-keyword">case</span> GROUP_OFFSETS:
               LOG.info(<span class="hljs-string">"Consumer subtask {} will start reading the following {} partitions from the committed group offsets in Kafka: {}"</span>,
                  getRuntimeContext().getIndexOfThisSubtask(),
                  subscribedPartitionsToStartOffsets.size(),
                  subscribedPartitionsToStartOffsets.keySet());
         }
      } <span class="hljs-keyword">else</span> {
         LOG.info(<span class="hljs-string">"Consumer subtask {} initially has no partitions to read from."</span>,
            getRuntimeContext().getIndexOfThisSubtask());
      }
   }
}
</code></pre>
<p data-nodeid="48">对 Kafka 中的 Topic 和 Partition 的数据进行读取的核心逻辑都在 run 方法中：</p>
<pre class="lang-java" data-nodeid="49"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(SourceContext&lt;T&gt; sourceContext)</span> <span class="hljs-keyword">throws</span> Exception </span>{
   <span class="hljs-keyword">if</span> (subscribedPartitionsToStartOffsets == <span class="hljs-keyword">null</span>) {
      <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> Exception(<span class="hljs-string">"The partitions were not set for the consumer"</span>);
   }
   <span class="hljs-keyword">this</span>.successfulCommits = <span class="hljs-keyword">this</span>.getRuntimeContext().getMetricGroup().counter(COMMITS_SUCCEEDED_METRICS_COUNTER);
   <span class="hljs-keyword">this</span>.failedCommits =  <span class="hljs-keyword">this</span>.getRuntimeContext().getMetricGroup().counter(COMMITS_FAILED_METRICS_COUNTER);
   <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> subtaskIndex = <span class="hljs-keyword">this</span>.getRuntimeContext().getIndexOfThisSubtask();
   <span class="hljs-keyword">this</span>.offsetCommitCallback = <span class="hljs-keyword">new</span> KafkaCommitCallback() {
      <span class="hljs-meta">@Override</span>
      <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onSuccess</span><span class="hljs-params">()</span> </span>{
         successfulCommits.inc();
      }
      <span class="hljs-meta">@Override</span>
      <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onException</span><span class="hljs-params">(Throwable cause)</span> </span>{
         LOG.warn(String.format(<span class="hljs-string">"Consumer subtask %d failed async Kafka commit."</span>, subtaskIndex), cause);
         failedCommits.inc();
      }
   };

   <span class="hljs-keyword">if</span> (subscribedPartitionsToStartOffsets.isEmpty()) {
      sourceContext.markAsTemporarilyIdle();
   }
   LOG.info(<span class="hljs-string">"Consumer subtask {} creating fetcher with offsets {}."</span>,
      getRuntimeContext().getIndexOfThisSubtask(), subscribedPartitionsToStartOffsets);
  
   <span class="hljs-keyword">this</span>.kafkaFetcher = createFetcher(
         sourceContext,
         subscribedPartitionsToStartOffsets,
         periodicWatermarkAssigner,
         punctuatedWatermarkAssigner,
         (StreamingRuntimeContext) getRuntimeContext(),
         offsetCommitMode,
         getRuntimeContext().getMetricGroup().addGroup(KAFKA_CONSUMER_METRICS_GROUP),
         useMetrics);
   <span class="hljs-keyword">if</span> (!running) {
      <span class="hljs-keyword">return</span>;
   }
   <span class="hljs-keyword">if</span> (discoveryIntervalMillis == PARTITION_DISCOVERY_DISABLED) {
      kafkaFetcher.runFetchLoop();
   } <span class="hljs-keyword">else</span> {
      runWithPartitionDiscovery();
   }
}
</code></pre>
<h3 data-nodeid="50">Flink 消费 Kafka 数据代码</h3>
<p data-nodeid="51">上面介绍了 Flink 消费 Kafka 的方式，以及消息序列化的方式，同时介绍了分区和 Topic 的动态发现方法，那么回到我们的项目中来，消费 Kafka 数据的完整代码如下：</p>
<pre class="lang-java" data-nodeid="52"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KafkaConsumer</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.enableCheckpointing(<span class="hljs-number">5000</span>);
        Properties properties = <span class="hljs-keyword">new</span> Properties();
        properties.setProperty(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"127.0.0.1:9092"</span>);
        <span class="hljs-comment">//设置消费组</span>
        properties.setProperty(<span class="hljs-string">"group.id"</span>, <span class="hljs-string">"group_test"</span>);
        properties.setProperty(FlinkKafkaConsumerBase.KEY_PARTITION_DISCOVERY_INTERVAL_MILLIS, <span class="hljs-string">"10"</span>);
        FlinkKafkaConsumer&lt;String&gt; consumer = <span class="hljs-keyword">new</span> FlinkKafkaConsumer&lt;&gt;(<span class="hljs-string">"test"</span>, <span class="hljs-keyword">new</span> SimpleStringSchema(), properties);
        <span class="hljs-comment">//设置从最早的ffset消费</span>
        consumer.setStartFromEarliest();
        env.addSource(consumer).flatMap(<span class="hljs-keyword">new</span> FlatMapFunction&lt;String, String&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flatMap</span><span class="hljs-params">(String value, Collector&lt;String&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                System.out.println(value);
            }
        });
        env.execute(<span class="hljs-string">"start consumer..."</span>);
    }
}
</code></pre>
<p data-nodeid="1662">我们可以直接右键运行代码，在控制台中可以看到数据的正常打印，如下图所示：</p>
<p data-nodeid="1663" class=""><img src="https://s0.lgstatic.com/i/image/M00/2D/28/Ciqc1F8C4FGAe30yAAMlcCNXF_o519.png" alt="Drawing 1.png" data-nodeid="1667"></p>

<p data-nodeid="1546">通过代码可知，我们之前发往 Kafka 的消息被完整地打印出来了。</p>




<h3 data-nodeid="56">总结</h3>
<p data-nodeid="2021" class="te-preview-highlight">这一课时介绍了 Flink 消费 Kafka 的方式，比如从常用的指定单个或者多个 Topic、消息的序列化、分区的动态发现等，还从源码上介绍了 Flink 消费 Kafka 的原理。通过本课时的学习，相信你可以对 Flink 消费 Kafka 有一个较为全面地了解，根据业务场景可以正确选择消费的方式和配置。</p>

---

### 精选评论

##### **6513：
> 项目中也是用main方法作为入口吗？有集成springboot的案例吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在编写Flink代码时，尽量避免使用spring类的框架，因为没有必要。只要依赖Flink必要的包和一些工具类即可。

##### **冰：
> 老师，这里可以从指定offset消费，怎么在程序终止或停止时保存offset，以便启时使用了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果实际情况需要保存位点，那么一般是自己管理位点，每次停止重启后从自己管理的位点消费，比如你可以存储在mysql中，自己去读取

##### **良：
> 我是用flink消费Kafka从指定的topic和partition中指定的offset处开始消费，实际结果与预期不一致，消费的分区对不上是啥原因呢？示例代码：Mapoffsets.put(new KafkaTopicPartition(topic, 0), 1L);offsets.put(new KafkaTopicPartition(topic, 1), 2L);offsets.put(new KafkaTopicPartition(topic, 2), 3L);consumer.setStartFromSpecificOffsets(offsets);返回结果：ConsumerRecord(topic=new-topic-config-test, partition=0, offset=10105849, key=, value=Python从入门到放弃!ConsumerRecord(topic=new-topic-config-test, partition=3, offset=10107121, key=, value=Python从入门到放弃!ConsumerRecord(topic=new-topic-config-test, partition=2, offset=10102806, key=, value=Java从入门到放弃!

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 从两个原因查询，第一看下并行度的设置要和kafka分区设置保持一致。第二，要保证kafka4个分区都有数据。

##### *轩：
> 请问哪里可以下载项目用的数据，不是源码

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在项目中有数据，也可以自己造一些数据

##### **7324：
> flink如何将key相同的数据写入到kafka的同一个partition呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你可以自定义自己的kafka分区器，可以查一下FlinkKafkaPartitioner的用法，但是一般我们不会这么用，如果你需要自定义写入kafka的分区器，要保证数据尽量均匀，不要引起kafka端的数据倾斜

