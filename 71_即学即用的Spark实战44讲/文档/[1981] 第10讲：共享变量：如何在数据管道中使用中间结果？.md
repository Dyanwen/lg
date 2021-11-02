<p>今天我将为你讲解：如何用共享变量在数据管道中使用中间结果。共享变量是 Spark 中进阶特性之一，一共有两种：</p>
<ul>
<li>广播变量；</li>
<li>累加器。</li>
</ul>
<p>这两种变量可以认为是在用算子定义的数据管道外的两个全局变量，供所有计算任务使用。在 Spark 作业中，用户编写的高阶函数会在集群中的 Executor 里执行，这些 Executor 可能会用到相同的变量，这些变量被复制到每个 Executor 中，而 Executor 对变量的更新不会传回 Driver。</p>
<p>在计算任务中支持通用的可读写变量一般是低效的，即便如此，Spark 还是提供了两类共享变量：广播变量（broadcast variable）与累加器（accumulator）。当然，对于分布式变量，如果不加限制会出现一致性的问题，所以共享变量是两种非常特殊的变量。</p>
<ul>
<li>广播变量：只读；</li>
<li>累加器：只能增加。</li>
</ul>
<h3>广播变量</h3>
<p>广播变量类似于 MapReduce 中的 DistributeFile，通常来说是一份不大的数据集，一旦广播变量在 Driver 中被创建，整个数据集就会在集群中进行广播，能让所有正在运行的计算任务以只读方式访问。广播变量支持一些简单的数据类型，如整型、集合类型等，也支持很多复杂数据类型，如一些自定义的数据类型。</p>
<p>广播变量为了保证数据被广播到所有节点，使用了很多办法。这其实是一个很重要的问题，我们不能期望 100 个或者 1000 个 Executor 去连接 Driver，并拉取数据，这会让 Driver 不堪重负。Executor 采用的是通过 HTTP 连接去拉取数据，类似于 BitTorrent 点对点传输。这样的方式更具扩展性，避免了所有 Executor 都去向 Driver 请求数据而造成 Driver 故障。</p>
<p>Spark 广播机制运作方式是这样的：Driver 将已序列化的数据切分成小块，然后将其存储在自己的块管理器 BlockManager 中，当 Executor 开始运行时，每个 Executor 首先从自己的内部块管理器中试图获取广播变量，如果以前广播过，那么直接使用；如果没有，Executor 就会从 Driver 或者其他可用的 Executor 去拉取数据块。一旦拿到数据块，就会放到自己的块管理器中。供自己和其他需要拉取的 Executor 使用。这就很好地防止了 Driver 单点的性能瓶颈，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/09/F1/CgqCHl69BwmAP2B7AAGvkqEmVkk327.png" alt="图片1.png"></p>
<p>下面来看看如何在 Spark 作业中创建、使用广播变量。代码如下：</p>
<pre><code data-language="js" class="lang-js">scala&gt; val rdd_one = sc.parallelize(Seq(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>))
<span class="hljs-attr">rdd_one</span>: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[<span class="hljs-number">101</span>] at
parallelize at &lt;<span class="hljs-built_in">console</span>&gt;:<span class="hljs-number">25</span>
    scala&gt; val i = <span class="hljs-number">5</span>
    <span class="hljs-attr">i</span>: Int = <span class="hljs-number">5</span>
scala&gt; val bi = sc.broadcast(i)
<span class="hljs-attr">bi</span>: org.apache.spark.broadcast.Broadcast[Int] = Broadcast(<span class="hljs-number">147</span>)
scala&gt; bi.value
<span class="hljs-attr">res166</span>: Int = <span class="hljs-number">5</span>
scala&gt; rdd_one.take(<span class="hljs-number">5</span>)
<span class="hljs-attr">res164</span>: <span class="hljs-built_in">Array</span>[Int] = <span class="hljs-built_in">Array</span>(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>)
scala&gt; rdd_one.map(<span class="hljs-function"><span class="hljs-params">j</span> =&gt;</span> j + bi.value).take(<span class="hljs-number">5</span>)
<span class="hljs-attr">res165</span>: <span class="hljs-built_in">Array</span>[Int] = <span class="hljs-built_in">Array</span>(<span class="hljs-number">6</span>, <span class="hljs-number">7</span>, <span class="hljs-number">8</span>)
</code></pre>
<p>在用户定义的高阶函数中，可以直接使用广播变量的引用。下面看一个集合类型的广播变量：</p>
<pre><code>scala&gt; val rdd_one = sc.parallelize(Seq(1,2,3))
    rdd_one: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[109] at
parallelize at &lt;console&gt;:25
scala&gt; val m = scala.collection.mutable.HashMap(1 -&gt; 2, 2 -&gt; 3, 3 -&gt; 4)
    m: scala.collection.mutable.HashMap[Int,Int] = Map(2 -&gt; 3, 1 -&gt; 2, 3 -&gt; 4)
scala&gt; val bm = sc.broadcast(m)
bm:
org.apache.spark.broadcast.Broadcast[scala.collection.mutable.HashMap[Int,I
nt]] = Broadcast(178)
scala&gt; rdd_one.map(j =&gt; j * bm.value(j)).take(5)
res191: Array[Int] = Array(2, 6, 12)
</code></pre>
<p>该例中，元素乘以元素对应值得到最后结果。广播变量会持续占用内存，当我们不需要的时候，可以用 unpersist 算子将其移除，这时，如果计算任务又用到广播变量，那么就会重新拉取数据，如下：</p>
<pre><code data-language="java" class="lang-java">    ...
scala&gt; val rdd_one = sc.parallelize(Seq(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>))
rdd_one: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[<span class="hljs-number">101</span>] at
parallelize at &lt;console&gt;:<span class="hljs-number">25</span>
scala&gt; val k = <span class="hljs-number">5</span>
k: Int = <span class="hljs-number">5</span>
scala&gt; val bk = sc.broadcast(k)
bk: org.apache.spark.broadcast.Broadcast[Int] = Broadcast(<span class="hljs-number">163</span>)
scala&gt; rdd_one.map(j =&gt; j + bk.value).take(<span class="hljs-number">5</span>)
res184: Array[Int] = Array(<span class="hljs-number">6</span>, <span class="hljs-number">7</span>, <span class="hljs-number">8</span>)
scala&gt; bk.unpersist
scala&gt; rdd_one.map(j =&gt; j + bk.value).take(<span class="hljs-number">5</span>)
res186: Array[Int] = Array(<span class="hljs-number">6</span>, <span class="hljs-number">7</span>, <span class="hljs-number">8</span>)
</code></pre>
<p>你还可以使用 destroy 方法彻底销毁广播变量，调用该方法后，如果计算任务中又用到广播变量，则会抛出异常：</p>
<pre><code data-language="java" class="lang-java">scala&gt; val rdd_one = sc.parallelize(Seq(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>))
rdd_one: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[<span class="hljs-number">101</span>] at
parallelize at &lt;console&gt;:<span class="hljs-number">25</span>
scala&gt; val k = <span class="hljs-number">5</span>
k: Int = <span class="hljs-number">5</span>
scala&gt; val bk = sc.broadcast(k)
bk: org.apache.spark.broadcast.Broadcast[Int] = Broadcast(<span class="hljs-number">163</span>)
scala&gt; rdd_one.map(j =&gt; j + bk.value).take(<span class="hljs-number">5</span>)
res184: Array[Int] = Array(<span class="hljs-number">6</span>, <span class="hljs-number">7</span>, <span class="hljs-number">8</span>)
scala&gt; bk.destroy
scala&gt; rdd_one.map(j =&gt; j + bk.value).take(<span class="hljs-number">5</span>)
<span class="hljs-number">17</span>/<span class="hljs-number">05</span>/<span class="hljs-number">27</span> <span class="hljs-number">14</span>:<span class="hljs-number">07</span>:<span class="hljs-number">28</span> ERROR Utils: Exception encountered
org.apache.spark.SparkException: <span class="hljs-function">Attempted to use <span class="hljs-title">Broadcast</span><span class="hljs-params">(<span class="hljs-number">163</span>)</span> after it
was <span class="hljs-title">destroyed</span> <span class="hljs-params">(destroy at &lt;console&gt;:<span class="hljs-number">30</span>)</span>
at org.apache.spark.broadcast.Broadcast.<span class="hljs-title">assertValid</span><span class="hljs-params">(Broadcast.scala:<span class="hljs-number">144</span>)</span>
at
org.apache.spark.broadcast.TorrentBroadcast$$anonfun$writeObject$1.apply$mc
V$<span class="hljs-title">sp</span><span class="hljs-params">(TorrentBroadcast.scala:<span class="hljs-number">202</span>)</span>
at org.apache.spark.broadcast.TorrentBroadcast$$anonfun$wri
</span></code></pre>
<p><strong>广播变量在一定数据量范围内可以有效地使作业避免 Shuffle，使计算尽可能本地运行，Spark 的 Map 端连接操作就是用广播变量实现的。</strong></p>
<p>为了让你更好地理解上面那句话的意思，我再举一个比较典型的场景，我们希望对海量的日志进行校验，日志可以简单认为是如下的格式：<br>
表 A：校验码，内容</p>
<p>也就是说，我们需要根据校验码的不同，对内容采取不同规则的校验，而检验码与校验规则的映射则存储在另外一个数据库：<br>
表 B：校验码，规则</p>
<p>这样，情况就比较清楚了，如果不考虑广播变量，我们有这么两种做法：</p>
<ol>
<li>直接使用 map 算子，在 map 算子中的自定义函数中去查询数据库，那么有多少行，就要查询多少次数据库，这样性能非常差。</li>
<li>先将表 B 查出来转化为 RDD，使用 join 算子进行连接操作后，再使用 map 算子进行处理，这样做性能会比前一种方式好很多，但是会引起大量的 Shuffle 操作，对资源消耗不小。</li>
</ol>
<p>当考虑广播变量后，我们有了这样一种做法（Python 风格伪代码）：</p>
<pre><code data-language="SQL" class="lang-SQL"><span class="hljs-comment">###表A</span>
tableA = spark.sparkcontext.textFrom('/path')
<span class="hljs-comment">###广播表B</span>
validateTable = spark.sparkcontext.broadcast(queryTable())
<span class="hljs-comment">###验证函数，在验证函数中会取得对应的校验规则进行校验</span>
def validate(validateNo,validateTable ):
......
<span class="hljs-comment">##统计校验结果</span>
validateResult = tableA.map(validate).reduceByKey((lambda x , y: x + y))
....
</code></pre>
<p>这样，相当于先将小表进行广播，广播到每个 Executor 的内存中，供 map 函数使用，这就避免了 Shuffle，虽然语义上还是 join（小表放内存），但无论是资源消耗还是执行时间，都要远优于前面两种方式。</p>
<h3>累加器</h3>
<p>与广播变量只读不同，累加器是一种只能进行增加操作的共享变量。如果你想知道记录中有多少错误数据，一种方法是针对这种错误数据编写额外逻辑，另一种方式是使用累加器。用法如下：</p>
<pre><code data-language="java" class="lang-java">    ...
scala&gt; val acc1 = sc.longAccumulator(<span class="hljs-string">"acc1"</span>)
acc1: org.apache.spark.util.LongAccumulator = LongAccumulator(id: <span class="hljs-number">10355</span>,
name: Some(acc1), value: <span class="hljs-number">0</span>)
scala&gt; val someRDD = tableRDD.map(x =&gt; {acc1.add(<span class="hljs-number">1</span>); x})
someRDD: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[<span class="hljs-number">99</span>] at map at
&lt;console&gt;:<span class="hljs-number">29</span>
scala&gt; acc1.value
res156: Long = <span class="hljs-number">0</span> <span class="hljs-comment">/*there has been no action on the RDD so accumulator did
not get incremented*/</span>
scala&gt; someRDD.count
res157: Long = <span class="hljs-number">351</span>
scala&gt; acc1.value
res158: Long = <span class="hljs-number">351</span>
scala&gt; acc1
res145: org.apache.spark.util.LongAccumulator = LongAccumulator(id: <span class="hljs-number">10355</span>,
name: Some(acc1), value: <span class="hljs-number">351</span>)
</code></pre>
<p>上面这个例子用 SparkContext 初始化了一个长整型的累加器。LongAccumulator 方法会将累加器变量置为 0。行动算子 count 触发计算后，累加器在 map 函数中被调用，其值会一直增加，最后定格为 351。Spark 内置的累加器有如下几种。</p>
<ul>
<li>LongAccumulator：长整型累加器，用于求和、计数、求均值的 64 位整数。</li>
<li>DoubleAccumulator：双精度型累加器，用于求和、计数、求均值的双精度浮点数。</li>
<li>CollectionAccumulator[T]：集合型累加器，可以用来收集所需信息的集合。</li>
</ul>
<p>所有这些累加器都是继承自 AccumulatorV2，如果这些累加器还是不能满足用户的需求，Spark 允许自定义累加器。如果需要某两列进行汇总，无疑自定义累加器比直接编写逻辑要方便很多，例如：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/09/F1/CgqCHl69BxaAR5emAAAg3H3pKGc444.png" alt="图片2.png"></p>
<p>这个表只有两列，需要统计 A 列与 B 列的汇总值。下面来看看根据上面的逻辑如何实现一个自定义累加器。代码如下：</p>
<pre><code data-language="java" class="lang-java">	<span class="hljs-keyword">import</span> org.apache.spark.util.AccumulatorV2
	<span class="hljs-keyword">import</span> org.apache.spark.SparkConf
	<span class="hljs-keyword">import</span> org.apache.spark.SparkContext
	<span class="hljs-keyword">import</span> org.apache.spark.SparkConf
	 
	<span class="hljs-comment">// 构造一个保存累加结果的类</span>
	<span class="hljs-function"><span class="hljs-keyword">case</span> class <span class="hljs-title">SumAandB</span><span class="hljs-params">(A: Long, B: Long)</span>
	 
	class FieldAccumulator extends AccumulatorV2[SumAandB,SumAandB] </span>{
	
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> A:Long = <span class="hljs-number">0L</span>
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">var</span> B:Long = <span class="hljs-number">0L</span>
	    <span class="hljs-comment">// 如果A和B同时为0，则累加器值为0</span>
	    override def isZero: Boolean = A == <span class="hljs-number">0</span> &amp;&amp; B == <span class="hljs-number">0L</span>
	    <span class="hljs-comment">// 复制一个累加器</span>
	    <span class="hljs-function">override def <span class="hljs-title">copy</span><span class="hljs-params">()</span>: FieldAccumulator </span>= {
	        val newAcc = <span class="hljs-keyword">new</span> FieldAccumulator
	        newAcc.A = <span class="hljs-keyword">this</span>.A
	        newAcc.B = <span class="hljs-keyword">this</span>.B
	        newAcc
	    }
	    <span class="hljs-comment">// 重置累加器为0</span>
	    <span class="hljs-function">override def <span class="hljs-title">reset</span><span class="hljs-params">()</span>: Unit </span>= { A = <span class="hljs-number">0</span> ; B = <span class="hljs-number">0L</span> }
	    <span class="hljs-comment">// 用累加器记录汇总结果</span>
	    <span class="hljs-function">override def <span class="hljs-title">add</span><span class="hljs-params">(v: SumAandB)</span>: Unit </span>= {
	        A += v.A
	        B += v.B
	    }
	    <span class="hljs-comment">// 合并两个累加器</span>
	    <span class="hljs-function">override def <span class="hljs-title">merge</span><span class="hljs-params">(other: AccumulatorV2[SumAandB, SumAandB])</span>: Unit </span>= {
	        other match {
	        <span class="hljs-keyword">case</span> o: FieldAccumulator =&gt; {
	            A += o.A
	            B += o.B}
	        <span class="hljs-keyword">case</span> _ =&gt;
	        }
	    }
	    <span class="hljs-comment">// 当Spark调用时返回结果</span>
	    override def value: SumAandB = SumAandB(A,B)
	}
</code></pre>
<p>凡是有关键字 override 的方法，均是重载实现自己逻辑的方法。累加器调用方式如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">package</span> com.spark.examples.rdd
 
<span class="hljs-keyword">import</span> org.apache.spark.SparkConf
<span class="hljs-keyword">import</span> org.apache.spark.SparkContext
 
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Driver</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">App</span></span>{

  val conf = <span class="hljs-keyword">new</span> SparkConf
  val sc = <span class="hljs-keyword">new</span> SparkContext(conf)
  val filedAcc = <span class="hljs-keyword">new</span> FieldAccumulator
  sc.register(filedAcc, <span class="hljs-string">" filedAcc "</span>)
  <span class="hljs-comment">// 过滤掉表头</span>
  val tableRDD = sc.textFile(<span class="hljs-string">"table.csv"</span>).filter(_.split(<span class="hljs-string">","</span>)(<span class="hljs-number">0</span>) != <span class="hljs-string">"A"</span>)
  tableRDD.map(x =&gt; {
     val fields = x.split(<span class="hljs-string">","</span>)
     val a = fields(<span class="hljs-number">1</span>).toInt
     val b = fields(<span class="hljs-number">2</span>).toLong
     filedAcc.add(SumAandB (a, b))
     x
  }).count
}
</code></pre>
<p>最后计数器的结果为（3100, 31）。</p>
<h3>小结</h3>
<p>本课时主要介绍了 Spark 的两种共享变量，注意体会广播变量最后介绍的 map 端 join 的场景，这在实际使用中非常普遍。另外广播变量的大小，按照我的经验，要根据 Executor 和 Worker 资源来确定，几十兆、一个 G 的广播变量在大多数情况不会有什么问题，如果资源充足，那么1G~10G 以内问题也不大。</p>
<p>最后我要给你留一个思考题，请你对数据集进行空行统计。你可以先用普通算子完成后，再用累加器的方式完成，并比较两者的执行效率 ，如果有条件的，可以在生产环境中用真实数据集比较下两者之间的差异，差异会更明显。</p>

---

### 精选评论


