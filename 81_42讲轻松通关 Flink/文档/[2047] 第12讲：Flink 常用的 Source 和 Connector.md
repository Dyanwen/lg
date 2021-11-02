<p data-nodeid="19066" class="">本课时我们主要介绍 Flink 中支持的 Source 和常用的 Connector。</p>
<p data-nodeid="19067">Flink 作为实时计算领域强大的计算能力，以及与其他系统进行对接的能力都非常强大。Flink 自身实现了多种 Source 和 Connector 方法，并且还提供了多种与第三方系统进行对接的 Connector。</p>
<p data-nodeid="19068">我们可以把这些 Source、Connector 分成以下几个大类。</p>
<h3 data-nodeid="19069">预定义和自定义 Source</h3>
<p data-nodeid="19070">在前面的第 04 课时“Flink 常用的 DataSet 和 DataStream API”中提到过几种 Flink 已经实现的新建 DataStream 方法。</p>
<h4 data-nodeid="19071">基于文件</h4>
<p data-nodeid="19072">我们在本地环境进行测试时可以方便地从本地文件读取数据：</p>
<pre class="lang-java" data-nodeid="19073"><code data-language="java">readTextFile(path)
readFile(fileInputFormat, path)
...
</code></pre>
<p data-nodeid="19074">可以直接在 ExecutionEnvironment 和 StreamExecutionEnvironment 类中找到 Flink 支持的读取本地文件的方法，如下图所示：</p>
<p data-nodeid="19075"><img src="https://s0.lgstatic.com/i/image/M00/10/F0/CgqCHl7LZOmARiTtAAUjtOQOdFM469.png" alt="image.png" data-nodeid="19163"></p>
<p data-nodeid="19076"><img src="https://s0.lgstatic.com/i/image/M00/10/E5/Ciqc1F7LZPCATHI4AAWm1YuLPzc592.png" alt="image (1).png" data-nodeid="19166"></p>
<pre class="lang-js" data-nodeid="19077"><code data-language="js">ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
<span class="hljs-comment">// read text file from local files system</span>
DataSet&lt;<span class="hljs-built_in">String</span>&gt; localLines = env.readTextFile(<span class="hljs-string">"file:///path/to/my/textfile"</span>);
<span class="hljs-comment">// read text file from an HDFS running at nnHost:nnPort</span>
DataSet&lt;<span class="hljs-built_in">String</span>&gt; hdfsLines = env.readTextFile(<span class="hljs-string">"hdfs://nnHost:nnPort/path/to/my/textfile"</span>);
<span class="hljs-comment">// read a CSV file with three fields</span>
DataSet&lt;Tuple3&lt;Integer, <span class="hljs-built_in">String</span>, Double&gt;&gt; csvInput = env.readCsvFile(<span class="hljs-string">"hdfs:///the/CSV/file"</span>)
	                       .types(Integer.class, <span class="hljs-built_in">String</span>.class, Double.class);
<span class="hljs-comment">// read a CSV file with five fields, taking only two of them</span>
DataSet&lt;Tuple2&lt;<span class="hljs-built_in">String</span>, Double&gt;&gt; csvInput = env.readCsvFile(<span class="hljs-string">"hdfs:///the/CSV/file"</span>)
                               .includeFields(<span class="hljs-string">"10010"</span>)  <span class="hljs-comment">// take the first and the fourth field</span>
	                       .types(<span class="hljs-built_in">String</span>.class, Double.class);
<span class="hljs-comment">// read a CSV file with three fields into a POJO (Person.class) with corresponding fields</span>
DataSet&lt;Person&gt;&gt; csvInput = env.readCsvFile(<span class="hljs-string">"hdfs:///the/CSV/file"</span>)
                         .pojoType(Person.class, <span class="hljs-string">"name"</span>, <span class="hljs-string">"age"</span>, <span class="hljs-string">"zipcode"</span>);
</code></pre>
<h4 data-nodeid="19078">基于 Collections</h4>
<p data-nodeid="19079">我们也可以基于内存中的集合、对象等创建自己的 Source。一般用来进行本地调试或者验证。</p>
<p data-nodeid="19080">例如：</p>
<pre class="lang-javascript" data-nodeid="19081"><code data-language="javascript">fromCollection(Collection)
fromElements(T ...)
</code></pre>
<p data-nodeid="19082">我们也可以在源码中看到 Flink 支持的方法，如下图所示：</p>
<p data-nodeid="19083"><img src="https://s0.lgstatic.com/i/image/M00/10/E5/Ciqc1F7LZQaAcf12AARuVRNchzI825.png" alt="image (2).png" data-nodeid="19173"></p>
<pre class="lang-java" data-nodeid="19084"><code data-language="java">DataSet&lt;String&gt; text = env.fromElements(
      <span class="hljs-string">"Flink Spark Storm"</span>,
      <span class="hljs-string">"Flink Flink Flink"</span>,
      <span class="hljs-string">"Spark Spark Spark"</span>,
      <span class="hljs-string">"Storm Storm Storm"</span>
);
List data = <span class="hljs-keyword">new</span> ArrayList&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt;();
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">0</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">1</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>,<span class="hljs-number">2</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">3</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">5</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">9</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">11</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">13</span>));
DataStreamSource&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; items = env.fromCollection(data);
</code></pre>
<h4 data-nodeid="19085">基于 Socket</h4>
<p data-nodeid="19086">通过监听 Socket 端口，我们可以在本地很方便地模拟一个实时计算环境。</p>
<p data-nodeid="19087">StreamExecutionEnvironment 中提供了 socketTextStream 方法可以通过 host 和 port 从一个 Socket 中以文本的方式读取数据。</p>
<pre class="lang-java" data-nodeid="19088"><code data-language="java">DataStream&lt;String&gt; text = env.socketTextStream(<span class="hljs-string">"127.0.0.1"</span>, <span class="hljs-number">9000</span>, <span class="hljs-string">"\n"</span>);
</code></pre>
<h4 data-nodeid="19089">自定义 Source</h4>
<p data-nodeid="19090">我们可以通过实现 Flink 的SourceFunction 或者 ParallelSourceFunction 来实现单个或者多个并行度的 Source。</p>
<p data-nodeid="19091">例如，我们在之前的课程中用到的：</p>
<pre class="lang-java" data-nodeid="19092"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyStreamingSource</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SourceFunction</span>&lt;<span class="hljs-title">Item</span>&gt; </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isRunning = <span class="hljs-keyword">true</span>;
    <span class="hljs-comment">/**
     * 重写run方法产生一个源源不断的数据发送源
     * <span class="hljs-doctag">@param</span> ctx
     * <span class="hljs-doctag">@throws</span> Exception
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(SourceContext&lt;Item&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">while</span>(isRunning){
            Item item = generateItem();
            ctx.collect(item);
            <span class="hljs-comment">//每秒产生一条数据</span>
            Thread.sleep(<span class="hljs-number">1000</span>);
        }
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cancel</span><span class="hljs-params">()</span> </span>{
        isRunning = <span class="hljs-keyword">false</span>;
    }
    <span class="hljs-comment">//随机产生一条商品数据</span>
    <span class="hljs-function"><span class="hljs-keyword">private</span> Item <span class="hljs-title">generateItem</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">int</span> i = <span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">100</span>);
        ArrayList&lt;String&gt; list = <span class="hljs-keyword">new</span> ArrayList();
        list.add(<span class="hljs-string">"HAT"</span>);
        list.add(<span class="hljs-string">"TIE"</span>);
        list.add(<span class="hljs-string">"SHOE"</span>);
        Item item = <span class="hljs-keyword">new</span> Item();
        item.setName(list.get(<span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">3</span>)));
        item.setId(i);
        <span class="hljs-keyword">return</span> item;
    }
}
</code></pre>
<h3 data-nodeid="19093">自带连接器</h3>
<p data-nodeid="19094">Flink 中支持了比较丰富的用来连接第三方的连接器，可以在官网中找到 Flink 支持的各种各样的连接器：</p>
<ul data-nodeid="19095">
<li data-nodeid="19096">
<p data-nodeid="19097"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/kafka.html" data-nodeid="19184">Apache Kafka</a>&nbsp;(source/sink)</p>
</li>
<li data-nodeid="19098">
<p data-nodeid="19099"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/cassandra.html" data-nodeid="19188">Apache Cassandra</a>&nbsp;(sink)</p>
</li>
<li data-nodeid="19100">
<p data-nodeid="19101"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/kinesis.html" data-nodeid="19192">Amazon Kinesis Streams</a>&nbsp;(source/sink)</p>
</li>
<li data-nodeid="19102">
<p data-nodeid="19103"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/elasticsearch.html" data-nodeid="19196">Elasticsearch</a>&nbsp;(sink)</p>
</li>
<li data-nodeid="19104">
<p data-nodeid="19105"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/filesystem_sink.html" data-nodeid="19200">Hadoop FileSystem</a>&nbsp;(sink)</p>
</li>
<li data-nodeid="19106">
<p data-nodeid="19107"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/rabbitmq.html" data-nodeid="19204">RabbitMQ</a>&nbsp;(source/sink)</p>
</li>
<li data-nodeid="19108">
<p data-nodeid="19109"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/nifi.html" data-nodeid="19208">Apache NiFi</a>&nbsp;(source/sink)</p>
</li>
<li data-nodeid="19110">
<p data-nodeid="19111"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/twitter.html" data-nodeid="19212">Twitter Streaming API</a>&nbsp;(source)</p>
</li>
<li data-nodeid="19112">
<p data-nodeid="19113"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/connectors/pubsub.html" data-nodeid="19216">Google PubSub</a>&nbsp;(source/sink)</p>
</li>
</ul>
<blockquote data-nodeid="19114">
<p data-nodeid="19115">需注意，我们在使用这些连接器时通常需要引用相对应的 Jar 包依赖。而且一定要注意，对于某些连接器比如 Kafka 是有版本要求的，一定要去官方网站找到对应的依赖版本。</p>
</blockquote>
<h3 data-nodeid="19116">基于 Apache Bahir 发布</h3>
<p data-nodeid="19117">Flink 还会基于 Apache Bahir 来发布一些 Connector，比如我们常用的 Redis 等。</p>
<blockquote data-nodeid="19118">
<p data-nodeid="19119">Apache Bahir 的代码最初是从&nbsp;<a href="https://www.oschina.net/p/spark-project" data-nodeid="19224">Apache Spark</a>&nbsp;项目中提取的，后作为一个独立的项目提供。Apache Bahir 通过提供多样化的流连接器（Streaming Connectors）和 SQL 数据源扩展分析平台的覆盖面，最初只是为&nbsp;<a href="https://www.oschina.net/p/spark-project" data-nodeid="19228">Apache Spark</a>&nbsp;提供拓展。目前也为&nbsp;<a href="https://www.oschina.net/p/apache-flink" data-nodeid="19232">Apache Flink</a>&nbsp;提供，后续还可能为&nbsp;<a href="https://www.oschina.net/p/apachebeam" data-nodeid="19236">Apache Beam</a>&nbsp;和更多平台提供拓展服务。</p>
</blockquote>
<p data-nodeid="19120">我们可以在 Bahir 的首页中找到目前支持的 Flink 连接器：</p>
<ul data-nodeid="19121">
<li data-nodeid="19122">
<p data-nodeid="19123">Flink streaming connector for ActiveMQ</p>
</li>
<li data-nodeid="19124">
<p data-nodeid="19125">Flink streaming connector for Akka</p>
</li>
<li data-nodeid="19126">
<p data-nodeid="19127">Flink streaming connector for Flume</p>
</li>
<li data-nodeid="19128">
<p data-nodeid="19129">Flink streaming connector for InfluxDB</p>
</li>
<li data-nodeid="19130">
<p data-nodeid="19131">Flink streaming connector for Kudu</p>
</li>
<li data-nodeid="19132">
<p data-nodeid="19133">Flink streaming connector for Redis</p>
</li>
<li data-nodeid="19134">
<p data-nodeid="19135">Flink streaming connector for Netty</p>
</li>
</ul>
<p data-nodeid="19136">其中就有我们非常熟悉的 Redis，很多同学 Flink 项目中访问 Redis 的方法都是自己进行的实现，推荐使用 Bahir 连接器。</p>
<p data-nodeid="19137">在本地单机情况下：</p>
<pre class="lang-java" data-nodeid="19138"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedisExampleMapper</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RedisMapper</span>&lt;<span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>&gt;&gt;</span>{
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> RedisCommandDescription <span class="hljs-title">getCommandDescription</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> RedisCommandDescription(RedisCommand.HSET, <span class="hljs-string">"HASH_NAME"</span>);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKeyFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f0;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getValueFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f1;
    }
}
FlinkJedisPoolConfig conf = <span class="hljs-keyword">new</span> FlinkJedisPoolConfig.Builder().setHost(<span class="hljs-string">"127.0.0.1"</span>).build();
DataStream&lt;String&gt; stream = ...;
stream.addSink(<span class="hljs-keyword">new</span> RedisSink&lt;Tuple2&lt;String, String&gt;&gt;(conf, <span class="hljs-keyword">new</span> RedisExampleMapper());
</code></pre>
<p data-nodeid="19139">当然我们也可以使用在集群或者哨兵模式下使用 Redis 连接器。</p>
<p data-nodeid="19140">集群模式：</p>
<pre class="lang-java" data-nodeid="19141"><code data-language="java">FlinkJedisPoolConfig conf = <span class="hljs-keyword">new</span> FlinkJedisPoolConfig.Builder()
    .setNodes(<span class="hljs-keyword">new</span> HashSet&lt;InetSocketAddress&gt;(Arrays.asList(<span class="hljs-keyword">new</span> InetSocketAddress(<span class="hljs-number">5601</span>)))).build();
DataStream&lt;String&gt; stream = ...;
stream.addSink(<span class="hljs-keyword">new</span> RedisSink&lt;Tuple2&lt;String, String&gt;&gt;(conf, <span class="hljs-keyword">new</span> RedisExampleMapper());
</code></pre>
<p data-nodeid="19142">哨兵模式：</p>
<pre class="lang-java" data-nodeid="19143"><code data-language="java">FlinkJedisSentinelConfig conf = <span class="hljs-keyword">new</span> FlinkJedisSentinelConfig.Builder()
    .setMasterName(<span class="hljs-string">"master"</span>).setSentinels(...).build();
DataStream&lt;String&gt; stream = ...;
stream.addSink(<span class="hljs-keyword">new</span> RedisSink&lt;Tuple2&lt;String, String&gt;&gt;(conf, <span class="hljs-keyword">new</span> RedisExampleMapper());
</code></pre>
<h3 data-nodeid="19144">基于异步 I/O 和可查询状态</h3>
<p data-nodeid="19145">异步 I/O 和可查询状态都是 Flink 提供的非常底层的与外部系统交互的方式。</p>
<p data-nodeid="19146">其中异步 I/O 是为了解决 Flink 在实时计算中访问外部存储产生的延迟问题，如果我们按照传统的方式使用 MapFunction，那么所有对外部系统的访问都是同步进行的。在很多情况下，计算性能受制于外部系统的响应速度，长时间进行等待，会导致整体吞吐低下。</p>
<p data-nodeid="19147">我们可以通过继承 RichAsyncFunction 来使用异步 I/O：</p>
<pre class="lang-java" data-nodeid="19148"><code data-language="java"><span class="hljs-comment">/**
 * 实现 'AsyncFunction' 用于发送请求和设置回调
 */</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AsyncDatabaseRequest</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichAsyncFunction</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>&gt;&gt; </span>{
    <span class="hljs-comment">/** 能够利用回调函数并发发送请求的数据库客户端 */</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> DatabaseClient client;
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        client = <span class="hljs-keyword">new</span> DatabaseClient(host, post, credentials);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
        client.close();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">asyncInvoke</span><span class="hljs-params">(String key, <span class="hljs-keyword">final</span> ResultFuture&lt;Tuple2&lt;String, String&gt;&gt; resultFuture)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-comment">// 发送异步请求，接收 future 结果</span>
        <span class="hljs-keyword">final</span> Future&lt;String&gt; result = client.query(key);
        <span class="hljs-comment">// 设置客户端完成请求后要执行的回调函数</span>
        <span class="hljs-comment">// 回调函数只是简单地把结果发给 future</span>
        CompletableFuture.supplyAsync(<span class="hljs-keyword">new</span> Supplier&lt;String&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">get</span><span class="hljs-params">()</span> </span>{
                <span class="hljs-keyword">try</span> {
                    <span class="hljs-keyword">return</span> result.get();
                } <span class="hljs-keyword">catch</span> (InterruptedException | ExecutionException e) {
                    <span class="hljs-comment">// 显示地处理异常</span>
                    <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
                }
            }
        }).thenAccept( (String dbResult) -&gt; {
            resultFuture.complete(Collections.singleton(<span class="hljs-keyword">new</span> Tuple2&lt;&gt;(key, dbResult)));
        });
    }
}
<span class="hljs-comment">// 创建初始 DataStream</span>
DataStream&lt;String&gt; stream = ...;
<span class="hljs-comment">// 应用异步 I/O 转换操作</span>
DataStream&lt;Tuple2&lt;String, String&gt;&gt; resultStream =
    AsyncDataStream.unorderedWait(stream, <span class="hljs-keyword">new</span> AsyncDatabaseRequest(), <span class="hljs-number">1000</span>, TimeUnit.MILLISECONDS, <span class="hljs-number">100</span>);
</code></pre>
<p data-nodeid="19149">其中，ResultFuture 的 complete 方法是异步的，不需要等待返回。</p>
<p data-nodeid="19150">我们在之前讲解 Flink State 时，提到过 Flink 提供了 StateDesciptor 方法专门用来访问不同的 state，StateDesciptor 同时还可以通过 setQueryable 使状态变成可以查询状态。可查询状态目前是一个 Beta 功能，暂时不推荐使用。</p>
<h3 data-nodeid="19151">总结</h3>
<p data-nodeid="20228">这一课时讲解了 Flink 主要支持的 Source 和 Connector，这些是我们用 Flink 访问其他系统的桥梁。本节课也为我们寻找合适的连接器指明了方向。其中最重要的 Kafka 连接器我们将会在后面的实战课时中单独讲解。</p>
<p data-nodeid="20229" class="te-preview-highlight"><a href="https://github.com/wangzhiwubigdata/quickstart" data-nodeid="20233">点击这里下载本课程源码</a></p>

---

### 精选评论

##### *涛：
> 请问rocketmq的连接jar有吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; GitHub 有开源的连接 RocketMQ 的项目：https://github.com/apache/rocketmq-externals/tree/master/rocketmq-flink

