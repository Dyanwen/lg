<p>这个课时我们将进入：“Spark 核心数据结构：弹性分布式数据集 RDD”的学习，今天的课程内容有两个：RDD 的核心概念以及实践环节：如何创建 RDD。</p>
<h3>RDD 的核心概念</h3>
<p>RDD 是 Spark 最核心的数据结构，RDD（Resilient Distributed Dataset）全称为弹性分布式数据集，是 Spark 对数据的核心抽象，也是最关键的抽象，它实质上是一组分布式的 JVM 不可变对象集合，不可变决定了它是只读的，所以 RDD 在经过变换产生新的 RDD 时，（如下图中 A-B），原有 RDD 不会改变。</p>
<p>弹性主要表现在两个方面：</p>
<ul>
<li>在面对出错情况（例如任意一台节点宕机）时，Spark 能通过 RDD 之间的依赖关系恢复任意出错的 RDD（如 B 和 D 可以算出最后的 RDD），RDD 就像一块海绵一样，无论怎么挤压，都像海绵一样完整；</li>
<li>在经过转换算子处理时，RDD 中的分区数以及分区所在的位置随时都有可能改变。</li>
</ul>
<p><img src="https://s0.lgstatic.com/i/image/M00/00/CD/Ciqc1F6qfhyAEvFNAAIRggB-Gcs425.png" alt="图片1.png"></p>
<p>每个 RDD 都有如下几个成员：</p>
<ul>
<li>分区的集合；</li>
<li>用来基于分区进行计算的函数（算子）；</li>
<li>依赖（与其他 RDD）的集合；</li>
<li>对于键-值型的 RDD 的散列分区器（可选）；</li>
<li>对于用来计算出每个分区的地址集合（可选，如 HDFS 上的块存储的地址）。</li>
</ul>
<p>如下图所示，RDD_0 根据 HDFS 上的块地址生成，块地址集合是 RDD_0 的成员变量，RDD_1由 RDD_0 与转换（transform）函数（算子）转换而成，该算子其实是 RDD_0 内部成员。从这个角度上来说，RDD_1 依赖于 RDD_0，这种依赖关系集合也作为 RDD_1 的成员变量而保存。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/00/CD/CgqCHl6qfjeAZbhpAAEjgjsLvIg341.png" alt="图片2.png"></p>
<p>在 Spark 源码中，RDD 是一个抽象类，根据具体的情况有不同的实现，比如 RDD_0 可以是 MapPartitionRDD，而 RDD_1 由于产生了 Shuffle（数据混洗，后面的课时会讲到），则是 ShuffledRDD。</p>
<p>下面我们来看一下 RDD 的源码，你也可以和前面对着看看：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 表示RDD之间的依赖关系的成员变量</span>
<span class="hljs-meta">@transient</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> deps: Seq[Dependency[_]]
<span class="hljs-comment">// 分区器成员变量</span>
<span class="hljs-meta">@transient</span> val partitioner: Option[Partitioner] = None
<span class="hljs-comment">// 该RDD所引用的分区集合成员变量</span>
<span class="hljs-meta">@transient</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> partitions_ : Array[Partition] = <span class="hljs-keyword">null</span>
<span class="hljs-comment">// 得到该RDD与其他RDD之间的依赖关系</span>
<span class="hljs-keyword">protected</span> def getDependencies: Seq[Dependency[_]] = deps
<span class="hljs-comment">// 得到该RDD所引用的分区</span>
<span class="hljs-keyword">protected</span> def getPartitions: Array[Partition]
<span class="hljs-comment">// 得到每个分区地址</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> def <span class="hljs-title">getPreferredLocations</span><span class="hljs-params">(split: Partition)</span>: Seq[String] </span>= Nil
<span class="hljs-comment">// distinct算子</span>
<span class="hljs-function">def <span class="hljs-title">distinct</span><span class="hljs-params">(numPartitions: Int)</span><span class="hljs-params">(implicit ord: Ordering[T] = <span class="hljs-keyword">null</span>)</span>: RDD[T] </span>= 
withScope  {
    map(x =&gt; (x, <span class="hljs-keyword">null</span>)).reduceByKey((x, y) =&gt; x, numPartitions).map(_._1)
}
</code></pre>
<p>其中，你需要特别注意这一行代码：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@transient</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> partitions_ : Array[Partition] = <span class="hljs-keyword">null</span>
</code></pre>
<p>它说明了一个重要的问题，RDD 是分区的集合，本质上还是一个集合，所以在理解时，你可以用分区之类的概念去理解，但是在使用时，就可以忘记这些，把其当做是一个普通的集合。为了再加深你的印象，我们来理解下模块 1 中 01 课时的 4 行代码：</p>
<pre><code data-language="java" class="lang-java">val list: List[Int] = List(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>,<span class="hljs-number">5</span>)
println(list.map(x =&gt; x + <span class="hljs-number">1</span>).filter { x =&gt; x &gt; <span class="hljs-number">1</span>}.reduce(_ + _))
......
val list: List[Int] = spark.sparkContext.parallelize(List(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>,<span class="hljs-number">5</span>))
println(list.map(x =&gt; x + <span class="hljs-number">1</span>).filter { x =&gt; x &gt; <span class="hljs-number">1</span>}.reduce(_ + _))
</code></pre>
<h3>实践环节：创建 RDD</h3>
<p>我一直强调，Spark 编程是一件不难的工作，而事实也确实如此。在上一课时我们讲解了创建 SparkSession 的代码，现在我们可以通过已有的 SparkSession 直接创建 RDD。在创建 RDD 之前，我们可以将 RDD 的类型分为以下几类：</p>
<ul>
<li>并行集合；</li>
<li>从 HDFS 中读取；</li>
<li>从外部数据源读取；</li>
<li>PairRDD。</li>
</ul>
<p>了解了 RDD 的类型，接下来我们逐个讲解它们的内容：</p>
<h4>并行化集合</h4>
<p>这种 RDD 纯粹是为了学习，将内存中的集合变量转换为 RDD，没太大实际意义。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//val spark: SparkSession = .......</span>
val rdd = spark.sparkcontext.parallelize(Seq(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>))
</code></pre>
<h4>从 HDFS 中读取</h4>
<p>这种生成 RDD 的方式是非常常用的，</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//val spark: SparkSession = .......</span>
val rdd = spark.sparkcontext.textFile(<span class="hljs-string">"hdfs://namenode:8020/user/me/wiki.txt"</span>)
</code></pre>
<h4>从外部数据源读取</h4>
<p>Spark 从 MySQL 中读取数据返回的 RDD 类型是 JdbcRDD，顾名思义，是基于 JDBC 读取数据的，这点与 Sqoop 是相似的，但不同的是 JdbcRDD 必须手动指定数据的上下界，也就是以 MySQL 表某一列的最值作为切分分区的依据。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//val spark: SparkSession = .......</span>
val lowerBound = <span class="hljs-number">1</span>
val upperBound = <span class="hljs-number">1000</span>
val numPartition = <span class="hljs-number">10</span>
val rdd = <span class="hljs-keyword">new</span> JdbcRDD(spark.sparkcontext,() =&gt; {
       Class.forName(<span class="hljs-string">"com.mysql.jdbc.Driver"</span>).newInstance()
       DriverManager.getConnection(<span class="hljs-string">"jdbc:mysql://localhost:3306/db"</span>, <span class="hljs-string">"root"</span>, <span class="hljs-string">"123456"</span>)
   },
   <span class="hljs-string">"SELECT content FROM mysqltable WHERE ID &gt;= ? AND ID &lt;= ?"</span>,
   lowerBound, 
   upperBound, 
   numPartition,
   r =&gt; r.getString(<span class="hljs-number">1</span>)
)
</code></pre>
<p>既然是基于 JDBC 进行读取，那么所有支持 JDBC 的数据库都可以通过这种方式进行读取，也包括支持 JDBC 的分布式数据库，但是你需要注意的是，从代码可以看出，这种方式的原理是利用多个 Executor 同时查询互不交叉的数据范围，从而达到并行抽取的目的。但是这种方式的抽取性能受限于 MySQL 的并发读性能，单纯提高 Executor 的数量到某一阈值后，再提升对性能影响不大。</p>
<p>上面介绍的是通过 JDBC 读取数据库的方式，对于 HBase 这种分布式数据库来说，情况有些不同，HBase 这种分布式数据库，在数据存储时也采用了分区的思想，HBase 的分区名为 Region，那么基于 Region 进行导入这种方式的性能就会比上面那种方式快很多，是真正的并行导入。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//val spark: SparkSession = .......</span>
val sc = spark.sparkcontext
val tablename = <span class="hljs-string">"your_hbasetable"</span>
val conf = HBaseConfiguration.create()
conf.set(<span class="hljs-string">"hbase.zookeeper.quorum"</span>, <span class="hljs-string">"zk1,zk2,zk3"</span>)
conf.set(<span class="hljs-string">"hbase.zookeeper.property.clientPort"</span>, <span class="hljs-string">"2181"</span>)
conf.set(TableInputFormat.INPUT_TABLE, tablename)
val rdd= sc.newAPIHadoopRDD(conf, classOf[TableInputFormat],
classOf[org.apache.hadoop.hbase.io.ImmutableBytesWritable],
classOf[org.apache.hadoop.hbase.client.Result]) 
<span class="hljs-comment">// 利用HBase API解析出行键与列值</span>
rdd_three.foreach{<span class="hljs-keyword">case</span> (_,result) =&gt; {
    val rowkey = Bytes.toString(result.getRow)
    val value1 = Bytes.toString(result.getValue(<span class="hljs-string">"cf"</span>.getBytes,<span class="hljs-string">"c1"</span>.getBytes))
}
</code></pre>
<p>值得一提的是 HBase 有一个第三方组件叫 Phoenix，可以让 HBase 支持 SQL 和 JDBC，在这个组件的配合下，第一种方式也可以用来抽取 HBase 的数据，此外，Spark 也可以读取 HBase 的底层文件 HFile，从而直接绕过 HBase 读取数据。说这么多，无非是想告诉你，读取数据的方法有很多，可以根据自己的需求进行选择。</p>
<p>通过第三方库的支持，Spark 几乎能够读取所有的数据源，例如 Elasticsearch，所以你如果要尝试的话，尽量选用 Maven 来管理依赖。</p>
<h4>PairRDD</h4>
<p>PairRDD 与其他 RDD 并无不同，只不过它的数据类型是 Tuple2[K,V]，表示键值对，因此这种 RDD 也被称为 PairRDD，泛型为 RDD[(K,V)]，而普通 RDD 的数据类型为 Int、String 等。这种数据结构决定了 PairRDD 可以使用某些基于键的算子，如分组、汇总等。PairRDD 可以由普通 RDD 转换得到：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//val spark: SparkSession = .......</span>
val a = spark.sparkcontext.textFile(<span class="hljs-string">"/user/me/wiki"</span>).map(x =&gt; (x,x))
</code></pre>
<h3>小结</h3>
<p>本课时带你学习完了 Spark 最核心的概念 RDD，本质上它可以看成是一个分布式的数据集合，它的目的就是隔离分布式数据集的复杂性，你也自己尝试了几种类型的 RDD。在实际情况中，大家经常会遇到从外部数据源读取成为RDD，如果理解了读取的本质，那么无论是什么数据源都能够轻松应对了。</p>
<p>这里我要给你留个思考题：如何指定你创建的 RDD 的分区数？</p>

---

### 精选评论

##### Max Wang：
> 之前让我们装的pyspark,能不能用python做一遍？ 谢谢！

##### *艺：
> context.parallelize(List(3,5,6,8,9,52,44), 1)

##### *艺：
> val listRDD = sc.makeRDD(1 to 16,4)

##### **庭：
> 可以开一堂课讲一下spark中不同情况下的分区策略嘛，

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢您的反馈，已转告给老师，没准后面课程会有惊喜哦~

##### **6524：
> 老师你好，你提到通过第三方库的支持，Spark 几乎能够读取所有的数据源，我的问题是如果通过Spark 读取JDBC 数据源 做transformation 之后 再写回数据库，可以多个executor 并行处理吗？还是顺序写入

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以并行写入，但受限于数据库的写性能

##### **磊：
> 主要和sc.<span style="color: rgb(77, 77, 77); font-family: &quot;Microsoft YaHei&quot;, &quot;SF Pro Display&quot;, Roboto, Noto, Arial, &quot;PingFang SC&quot;, sans-serif; font-size: 16px; font-variant-ligatures: common-ligatures;">sc.defaultParallelism和sc.</span><span style="color: rgb(77, 77, 77); font-family: &quot;Microsoft YaHei&quot;, &quot;SF Pro Display&quot;, Roboto, Noto, Arial, &quot;PingFang SC&quot;, sans-serif; font-size: 16px; font-variant-ligatures: common-ligatures;">defaultMinPartitions以及HDFS文件的Block数量有关,有的还会默认是1</span>

##### **生：
> 上面那个哥们需要搞pyspark的联系我，我的微信是RUIBIFBI&nbsp;

##### **福：
> 学习

