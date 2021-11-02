<p data-nodeid="3046" class="">本课时主要讲解 Kafka 的一些核心概念，以及模拟消息并发送。</p>


<h3 data-nodeid="1627">大数据消息中间件的王者——Kafka</h3>
<p data-nodeid="1628">在上一课时中提过在实时计算的场景下，我们绝大多数的数据源都是消息系统。所以，一个强大的消息中间件来支撑高达几十万的 QPS，以及海量数据存储就显得极其重要。</p>
<p data-nodeid="1629">Kafka 从众多的消息中间件中脱颖而出，主要是因为<strong data-nodeid="1747">高吞吐</strong>、<strong data-nodeid="1748">低延迟</strong>的特点；另外基于 Kafka 的生态越来越完善，各个实时处理框架包括 Flink 在消息处理上都会优先进行支持。在第 14 课时“Flink Exactly-once 实现原理解析”中提到 Flink 和 Kafka 结合实现端到端精确一次语义的原理。</p>
<p data-nodeid="1630">Kafka 从众多的消息中间件中脱颖而出，已经成为大数据生态系统中必不可少的一员，主要的特性包括：</p>
<ul data-nodeid="1631">
<li data-nodeid="1632">
<p data-nodeid="1633">高吞吐</p>
</li>
<li data-nodeid="1634">
<p data-nodeid="1635">低延迟</p>
</li>
<li data-nodeid="1636">
<p data-nodeid="1637">高容错</p>
</li>
<li data-nodeid="1638">
<p data-nodeid="1639">可靠性</p>
</li>
<li data-nodeid="1640">
<p data-nodeid="1641">生态丰富</p>
</li>
</ul>
<p data-nodeid="1642">为了接下来更好地理解和使用 Kafka，我们首先来看一下 Kafka 中的核心概念和基本入门。</p>
<h3 data-nodeid="1643">Kafka 核心概念</h3>
<p data-nodeid="1644">Kafka 是一个消息队列，<strong data-nodeid="1766">生产者</strong>向消息队列中写入数据，<strong data-nodeid="1767">消费者</strong>从队列中获取数据并进行消费。作为一个企业级的消息中间件，Kafka 会支持庞大的业务，不同的业务会有多个队列，我们用 Topic 来给队列命名，在使用 Kafka 时必须指定 Topic。</p>
<p data-nodeid="1645">我们可以认为一个 Topic 就是一个队列，每个 Topic 又会被分成多个 Partition，这样做是为了横向扩展，<strong data-nodeid="1772">提高吞吐量。</strong></p>
<p data-nodeid="3608" class="">Kafka 中每个 Partition 都对应一个 <strong data-nodeid="3614">Broker</strong>，一个 Broker 可以管理多个 Partition。举个例子，假如 Kafka 的某个 Topic 有 10 个 Partition、2 个 Broker，那么每个 Broker 就会管理 5 个 Partition。我们可以把 Partition 简单理解为一个文件，在接收生产者的数据时，需要将数据动态追加到 Partition 上。</p>

<p data-nodeid="8178">生产者会决定将数据写入哪个 Partition，消费者自己维护消费数据的位置，我们称为 <strong data-nodeid="8185">Offset</strong>。</p>
<p data-nodeid="8179" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/23/Ciqc1F79tIaAFzrEAAEE3vibcoU312.png" alt="1.png" data-nodeid="8188"></p>

<p data-nodeid="7897">同时，Kafka 提供了时间策略对过期的消息进行处理。</p>











<p data-nodeid="1650">Kafka 的每个消费者都有一个消费组来进行标识，同一个消费组的不同实例分布在多个进程或者多个机器上。</p>
<p data-nodeid="1651">Kafka 的源数据存储在 ZooKeeper 中，其中包含 Broker、Topic、Partition 等信息。在 0.8 版本之前，Kafka 还会将消费的 Offset 存储在 ZooKeeper 中。</p>
<p data-nodeid="1652">此外，ZooKeeper 还负责集群的 Broker 选举，以及所有 Topic 的 Partition 副本信息等。</p>
<h3 data-nodeid="1653">Kafka 连接 Flink</h3>
<p data-nodeid="1654">我们在第 12 课时“Flink 常用的 Source 和 Connector”中提过，Flink 中支持了比较丰富的用来连接第三方的连接器，Kafka Connector 是 Flink 支持的各种各样的连接器中比较完善的之一。</p>
<blockquote data-nodeid="1655">
<p data-nodeid="1656">Flink 提供了专门的 Kafka 连接器，向 Kafka Topic 中读取或者写入数据。Flink Kafka Consumer 集成了 Flink 的 Checkpoint 机制，可提供 exactly-once 的处理语义。为此，Flink 并不完全依赖于跟踪 Kafka 消费组的偏移量，而是在内部跟踪和检查偏移量。</p>
</blockquote>
<p data-nodeid="1657">同时也提过，我们在使用 Kafka 连接器时需要引用相对应的 Jar 包依赖。对于某些连接器比如 Kafka 是有版本要求的，一定要去<a href="https://ci.apache.org/projects/flink/flink-docs-stable/dev/connectors/kafka.html" data-nodeid="1799">官方网站</a>找到对应的依赖版本。</p>
<p data-nodeid="9385">我在下表中给出了不同版本的 Kafka，以及对应的 Connector 关系：</p>
<p data-nodeid="9386"><img src="https://s0.lgstatic.com/i/image/M00/2B/23/Ciqc1F79tJmAVPOaAAJmKKhvawg414.png" alt="2.png" data-nodeid="9390"></p>





<h4 data-nodeid="1705">Kafka 本地环境搭建</h4>
<p data-nodeid="1706">我们在本地环境搭建一个 Kafka_2.11-2.1.0 版本的 Kafka 单机环境，然后模拟一些数据写入到队列中。</p>
<p data-nodeid="1707">我们可以在<a href="http://kafka.apache.org/downloads" data-nodeid="1885">这里</a>下载对应版本的 Kafka，把压缩包进行解压，然后使用下面的命令启动单机版本的 Kafka。</p>
<p data-nodeid="1708">解压：</p>
<pre class="lang-powershell" data-nodeid="1709"><code data-language="powershell">&gt; tar <span class="hljs-literal">-xzf</span> kafka_2.<span class="hljs-number">11</span><span class="hljs-literal">-2</span>.<span class="hljs-number">1.0</span>.tgz
&gt; <span class="hljs-built_in">cd</span> kafka_2.<span class="hljs-number">11</span><span class="hljs-literal">-2</span>.<span class="hljs-number">1.0</span>
</code></pre>
<p data-nodeid="1710">启动 ZooKeeper 和 Kafka Server：</p>
<pre class="lang-powershell" data-nodeid="1711"><code data-language="powershell">启动ZK：nohup bin/zookeeper<span class="hljs-literal">-server</span><span class="hljs-literal">-start</span>.sh config/zookeeper.properties  &amp;
启动Server: 
nohup bin/kafka<span class="hljs-literal">-server</span><span class="hljs-literal">-start</span>.sh config/server.properties &amp;
</code></pre>
<p data-nodeid="1712">创建一个名为 test 的 Topic：</p>
<pre class="lang-powershell" data-nodeid="1713"><code data-language="powershell">bin/kafka<span class="hljs-literal">-topics</span>.sh -<span class="hljs-literal">-create</span> -<span class="hljs-literal">-zookeeper</span> localhost:<span class="hljs-number">2181</span> -<span class="hljs-literal">-replication</span><span class="hljs-literal">-factor</span> <span class="hljs-number">1</span> -<span class="hljs-literal">-partitions</span> <span class="hljs-number">1</span> -<span class="hljs-literal">-topic</span> test
</code></pre>
<h4 data-nodeid="1714">Kafka Producer</h4>
<p data-nodeid="1715">首先我们需要新增一个依赖，然后向名为 test 的 Topic 中写入数据。</p>
<p data-nodeid="1716">新增 Maven 依赖：</p>
<pre class="lang-html" data-nodeid="9714"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-connector-kafka_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>


<p data-nodeid="1718">向 test 这个 Topic 中写入数据：</p>
<pre class="lang-java" data-nodeid="1719"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KafkaProducer</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception</span>{
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        env.enableCheckpointing(<span class="hljs-number">5000</span>);
        DataStreamSource&lt;String&gt; text = env.addSource(<span class="hljs-keyword">new</span> MyNoParalleSource()).setParallelism(<span class="hljs-number">1</span>);
        Properties properties = <span class="hljs-keyword">new</span> Properties();
        properties.setProperty(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"127.0.0.1:9092"</span>);
        <span class="hljs-comment">// 2.0 配置 KafkaProducer</span>
        FlinkKafkaProducer&lt;String&gt; producer = <span class="hljs-keyword">new</span> FlinkKafkaProducer&lt;String&gt;(
                <span class="hljs-string">"127.0.0.1:9092"</span>, <span class="hljs-comment">//broker 列表</span>
                <span class="hljs-string">"test"</span>,           <span class="hljs-comment">//topic</span>
                <span class="hljs-keyword">new</span> SimpleStringSchema()); <span class="hljs-comment">// 消息序列化</span>
        
        <span class="hljs-comment">//写入 Kafka 时附加记录的事件时间戳</span>
        producer.setWriteTimestampToKafka(<span class="hljs-keyword">true</span>);
        text.addSink(producer);
        env.execute();
    }
}
</code></pre>
<p data-nodeid="1720">需要注意的是，我们这里使用了一个自定义的 MyNoParalleSource 类，该类使用了 Flink 提供的自定义 Source 方法，该方法会源源不断地产生一些测试数据，代码如下：</p>
<pre class="lang-java" data-nodeid="1721"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyNoParalleSource</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SourceFunction</span>&lt;<span class="hljs-title">String</span>&gt; </span>{
    <span class="hljs-comment">//private long count = 1L;</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isRunning = <span class="hljs-keyword">true</span>;
    <span class="hljs-comment">/**
     * 主要的方法
     * 启动一个source
     * 大部分情况下，都需要在这个run方法中实现一个循环，这样就可以循环产生数据了
     *
     * <span class="hljs-doctag">@param</span> ctx
     * <span class="hljs-doctag">@throws</span> Exception
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(SourceContext&lt;String&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">while</span>(isRunning){
            <span class="hljs-comment">//图书的排行榜</span>
            List&lt;String&gt; books = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
            books.add(<span class="hljs-string">"Pyhton从入门到放弃"</span>);<span class="hljs-comment">//10</span>
            books.add(<span class="hljs-string">"Java从入门到放弃"</span>);<span class="hljs-comment">//8</span>
            books.add(<span class="hljs-string">"Php从入门到放弃"</span>);<span class="hljs-comment">//5</span>
            books.add(<span class="hljs-string">"C++从入门到放弃"</span>);<span class="hljs-comment">//3</span>
            books.add(<span class="hljs-string">"Scala从入门到放弃"</span>);
            <span class="hljs-keyword">int</span> i = <span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">5</span>);
            ctx.collect(books.get(i));
            <span class="hljs-comment">//每2秒产生一条数据</span>
            Thread.sleep(<span class="hljs-number">2000</span>);
        }
    }
    <span class="hljs-comment">//取消一个cancel的时候会调用的方法</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cancel</span><span class="hljs-params">()</span> </span>{
        isRunning = <span class="hljs-keyword">false</span>;
    }
}
</code></pre>
<p data-nodeid="10353">我们在后面会使用这个方法来模拟生产中的订单数据，并进行接续处理；然后通过下面的命令就可以查看本地 Kafka 的 test 这个 Topic 中的数据：</p>
<p data-nodeid="10354" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/2B/2F/CgqCHl79tM2AKuVjAAXrXtM5A00675.png" alt="image (2).png" data-nodeid="10362"></p>


<p data-nodeid="1723">至此，我们就成功地向 Kafka 中写入数据了。</p>
<h3 data-nodeid="1724">源码解析</h3>
<p data-nodeid="1725">FlinkKafkaProducer 的代码十分简洁，首先继承了 TwoPhaseCommitSinkFunction，在第 14 课时“Flink Exactly-once 实现原理解析”中详细讲解过，这个类是 Flink 和 Kafka 结合实现精确一次处理语义的关键。</p>
<p data-nodeid="1726">FlinkProducer 提供了 6 种构造方法，我们可以根据需要选择不同的构造函数：</p>
<pre class="lang-java" data-nodeid="1727"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String brokerList,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SerializationSchema&lt;IN&gt; serializationSchema)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SerializationSchema&lt;IN&gt; serializationSchema,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties producerConfig)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SerializationSchema&lt;IN&gt; serializationSchema,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties producerConfig,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Optional&lt;FlinkKafkaPartitioner&lt;IN&gt;&gt; customPartitioner)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String brokerList,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KeyedSerializationSchema&lt;IN&gt; serializationSchema)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KeyedSerializationSchema&lt;IN&gt; serializationSchema,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties producerConfig)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String topicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KeyedSerializationSchema&lt;IN&gt; serializationSchema,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties producerConfig,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Semantic semantic)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String defaultTopicId,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KeyedSerializationSchema&lt;IN&gt; serializationSchema,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties producerConfig,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Optional&lt;FlinkKafkaPartitioner&lt;IN&gt;&gt; customPartitioner)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FlinkKafkaProducer011</span><span class="hljs-params">(
			String defaultTopicId,
			KeyedSerializationSchema&lt;IN&gt; serializationSchema,
			Properties producerConfig,
			Optional&lt;FlinkKafkaPartitioner&lt;IN&gt;&gt; customPartitioner,
			Semantic semantic,
			<span class="hljs-keyword">int</span> kafkaProducersPoolSize)</span></span>;
</code></pre>
<p data-nodeid="1728">这里有个特别需要注意的属性：FlinkKafkaPartitioner，这个类定义了数据写入 Kafka 的规则，如果用户没有指定，则会默认 FlinkFixedPartitioner，核心处理逻辑如下：</p>
<pre class="lang-java" data-nodeid="1729"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FlinkFixedPartitioner</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">FlinkKafkaPartitioner</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
...
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">partition</span><span class="hljs-params">(T record, <span class="hljs-keyword">byte</span>[] key, <span class="hljs-keyword">byte</span>[] value, String targetTopic, <span class="hljs-keyword">int</span>[] partitions)</span> </span>{
      Preconditions.checkArgument(
         partitions != <span class="hljs-keyword">null</span> &amp;&amp; partitions.length &gt; <span class="hljs-number">0</span>,
         <span class="hljs-string">"Partitions of the target topic is empty."</span>);
      <span class="hljs-keyword">return</span> partitions[parallelInstanceId % partitions.length];
   }
...
}
</code></pre>
<p data-nodeid="1730">此外，FlinkProducer 还封装了 beginTransaction、preCommit、commit、abort 等方法，这几个方法便是实现精确一次处理语义的关键。</p>
<h3 data-nodeid="1731">总结</h3>
<p data-nodeid="2480">本课时我们介绍了 Kafka 的核心概念，可以对其有一个全面的了解，并且还搭建了单机版的 Kafka 环境，使用自定义的数据源向 Kafka 中写入数据，最后从源码层面介绍了 FlinkProducer 的核心实现。通过本课时的学习，你可以对 Kafka 有全面的了解，并且能够使用 Kafka 连接器发送消息。</p>

---

### 精选评论

##### **6496：
> books的初始化应该放到while的外面吧

##### *强：
> 赞~

