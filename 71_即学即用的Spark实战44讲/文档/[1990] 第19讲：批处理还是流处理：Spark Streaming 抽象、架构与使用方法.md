<p data-nodeid="2325" class="">Spark Streaming 是 Spark 0.7 推出的流处理库，代表 Spark 正式进入流处理领域，距今已有快 6 年的时间。在这段时间中，随着 Spark 不断完善，Spark Streaming 在业界已得到广泛应用，应该算是目前最主要的流处理解决方案之一。随着 Spark 2.2 的 Structured Streaming 正式推出，Spark 下一代流处理技术已经呼之欲出。但是由于目前的客观情况，Structured Streaming 的成熟度还不太高，大量的流处理应用不可能也没有必要马上迁移至 Structured Streaming，所以 Spark Streaming 在今后一段时间还将继续活跃。另外，它的架构和它的抽象也值得我们学习和深思。在本节我们将会学习 Spark Streaming，主要内容有：</p>
<ul data-nodeid="2326">
<li data-nodeid="2327">
<p data-nodeid="2328">Spark Streaming 关键抽象与架构；</p>
</li>
<li data-nodeid="2329">
<p data-nodeid="2330">转换算子。</p>
</li>
</ul>
<h3 data-nodeid="2331">关键抽象与架构</h3>
<p data-nodeid="3741">要想深入理解 Spark Streaming，首先还是要了解 Spark Streaming 的关键抽象 DStream，DStream 意指 Discretized Stream（离散化流），<strong data-nodeid="3748">它大体上来说是一个 RDD 流（序列），其元素（RDD）可以理解为从输入流生成的批</strong>。在流处理的过程中，用户其实就是在对 DStream 进行各种变换，最后再输出。如下图所示，可以看出 Spark Streaming 的输入是连续的，经过 Spark Streaming 接收后，会变成一个 RDD 序列，之后的处理逻辑是基于 DStream 来操作的。</p>
<p data-nodeid="3742" class=""><img src="https://s0.lgstatic.com/i/image/M00/20/79/Ciqc1F7ohKSAGECBAADMWzmUmoQ981.png" alt="image" data-nodeid="3751"></p>



<p data-nodeid="5188">DStream 的生成依据是按照时间间隔切分，该时间间隔的数据会生成一个微批（mini-batch），即一个 RDD，所以该间隔也被称为批次间隔，DStream 里面持有对所有产生的 RDD 的引用，虽然 RDD 和 DStream 非常像，在种类上基本都是一一对应的，如 UnionDStream 与 UnionRDD，但是 DStream 还是和 RDD 有本质不同，如下图所示。</p>


<p data-nodeid="4701" class=""><img src="https://s0.lgstatic.com/i/image/M00/20/84/CgqCHl7ohLWAZQ9hAACfMLRTSwM614.png" alt="image" data-nodeid="4707"></p>


<p data-nodeid="2338">具体表现为：</p>
<pre class="lang-java" data-nodeid="2339"><code data-language="java">RDD = DStream @ batch T
DStream = <span class="hljs-function">RDD <span class="hljs-title">range</span> <span class="hljs-params">(tn<span class="hljs-number">-1</span>,tn)</span>
</span></code></pre>
<p data-nodeid="2340">RDD 是 DStream 中某个批次的数据，而 DStream 代表了一段时间所产生的 RDD。所以通过这种方式，Spark Streaming 把对连续流的处理，变成了对批序列 DStream 的处理。我们会在 StreamingContext 设置批次间隔大小，一般大于 200ms。</p>
<p data-nodeid="2341">如果我们仔细思考的话，会发现，这种抽象将连续的流看成了某一时刻静止的批，所以在提到 Spark Streaming 中的 RDD 时，一定要注意它还有个时间维度，最准确的说法是某段时间内的 RDD。</p>
<p data-nodeid="2342"><strong data-nodeid="2438">最后，可以看到 Spark Streaming 对流的抽象本质上还是流，只是处理是基于批来处理的。这与 Structured Streaming 来说是不同的，我们在后面会讲到。</strong></p>
<p data-nodeid="6881">Spark Streaming 在架构上与 Spark 离线计算架构非常相似，主要分为 Driver 与 Executor，同样可以运行在 Yarn、Mesos 上，也能够以 standalone 和 local 模式运行。它们之间的关系仍旧是 Driver 负责调度，Executor 执行任务。如下图所示。</p>
<p data-nodeid="6882" class=""><img src="https://s0.lgstatic.com/i/image/M00/20/84/CgqCHl7ohNmAEpWIAAHuZ2LI2To588.png" alt="image" data-nodeid="6886"></p>





<p data-nodeid="2346">在 Driver 中，有几个关键模块，SparkStreamingContext 、DStreamGraph、JobScheduler、Checkpoint、ReceiverTracker，下面我将就这几个模块分别介绍。</p>
<ul data-nodeid="2347">
<li data-nodeid="2348">
<p data-nodeid="2349">SparkStreamingContext：SparkStreamingContext是一开始在用户代码中初始化完成的。它主要的工作是对作业进行一些配置，例如 DStream 切分的批次间隔（Duration），以及与其他模块进行交互，如 DStreamGraph 和 JobScheduler 等。</p>
</li>
<li data-nodeid="2350">
<p data-nodeid="2351">DStreamGraph：既然 Spark Streaming 最后还是对批的处理，那么批处理中根据计算逻辑生成的 RDD DAG 也是存在的，它由 DStreamGraph 生成。DStreamGraph 维护了输入 DStream 与输出 DStream 的实例，还会通过 generateJobs 方法生成一个作业集合（RDD DAG），它会由 JobScheduler 调度启动执行任务。</p>
</li>
<li data-nodeid="2352">
<p data-nodeid="2353">JobScheduler：JobScheduler 顾名思义是 Spark Streaming 的作业调度器，在创建 SparkStreamingContext 的同时，JobScheduler 也会作为它的一部分被创建，所有的任务都是最后由它调度 Executor 来执行。</p>
</li>
<li data-nodeid="2354">
<p data-nodeid="2355">Checkpoint：在 Spark 中，任何一个 RDD 丢失，都可以通过依赖关系重新计算得到， Checkpoint 是 Spark Streaming 容错机制的核心，会定时对已算好的中间结果以及其他中间状态进行存储，避免了依赖链过长的问题。这样就算某个 DStream 丢失了，也不用从头开始计算，只需从最近的依赖关系开始计算即可。</p>
</li>
<li data-nodeid="2356">
<p data-nodeid="2357">ReceiverTracker：ReceiverTracker 通过 Executor 上的 ReceiverSupvisor 来管理所有的 Receiver。主要功能是把需要计算的数据发送给 Executor。当 Executor 接收完毕后，也会将数据块的元数据上报给 ReceiverTracker。</p>
</li>
</ul>
<p data-nodeid="2358">从上面这几个组件的关系上来说，SparkStreamingContext 负责与其他组件交互，DStreamGraph 与 JobScheduler 负责调度，Checkpoint 负责容错，ReceiverTracker 负责与 Executor 进行数据交互。</p>
<p data-nodeid="2359">Executor 是具体的任务执行者，其中重要的组件有 Receiver、ReceiverSupvisor、ReceiveredBlockHandler，ReceiverTracker 会和 Executor 通信，启动 ReceiverSupvisor 实例，ReceiverSupvisor 会马上启动 Receiver 开始接收数据。Receiver 接收到数据后，用 ReceiverdBlockHandler 以块的方式写到 Executor 的磁盘或者内存，对应的实现是 BlockManagerBasedBlockHandler 和 WriteAheadLogBasedBlockHandler，前者是根据 Executor 的 StorageLevel 写到相应的存储层，后者会先进行预写日志（Write Ahead Log），其中，后者能对流式数据源提供更好的容错性。数据接收完毕后，会根据调度开始计算任务。</p>
<p data-nodeid="2360">Spark Streaming 的作业初始化与提交和 Spark SQL 作业有些不同，我们还是通过初始化 SparkSession 的方式得到 StreamingContext 的引用，再对其设置一个关键参数：批次间隔后，就可以进行数据接收和数据处理的动作。</p>
<h3 data-nodeid="2361">无状态的转换算子</h3>
<p data-nodeid="7823">基于上面的抽象，对流进行处理与批处理就没什么不同了，我们只着眼于此刻正在处理的这个时间范围内的 RDD，所以数据处理方式与批处理并没有什么不同，算子也与批处理没多大区别，算子作用与数据流中的每个 RDD，这类算子我们称之为无状态算子，如下图所示：</p>
<p data-nodeid="7824" class=""><img src="https://s0.lgstatic.com/i/image/M00/20/79/Ciqc1F7ohOaAN6pwAADlQFH86M8061.png" alt="image" data-nodeid="7828"></p>



<p data-nodeid="2365"><strong data-nodeid="2459">我们在使用无状态算子时，仍然要注意，每次处理的结果都隐含着“这是...时间范围内数据的处理结果”的含义。</strong></p>
<p data-nodeid="2366">这类算子与前面介绍的转换算子没什么不同，如 map、mapPartitions、reduceByKey、reduce、flatmap、glom、filter、repartition、union 等等，这里就不重复描述了。</p>
<h3 data-nodeid="2367">有状态的转换算子</h3>
<p data-nodeid="2368">在实际工作场景中，默认的时间间隔很难满足流处理的业务需要，比如想对 DStream 中的某几个 RDD 进行操作，或者是想保存一些中间结果做增量计算，就需要运用到另一类转换算子：有状态的转换算子。</p>
<p data-nodeid="2369">有状态的转换算子主要分为两种，一种是基于时间窗口，另一种是基于整个时间跨度。本课时将对其进行介绍。</p>
<p data-nodeid="8757">基于时间窗口的概念其实早就深植于 Spark Streaming 中，我们在设置批次间隔时间（如 1 s）时，本质上就是设置了一个时间窗口，在用户代码中的计算逻辑其实是作用在每一个在该批次间隔中形成的 RDD 上，上一个 RDD 和这一个 RDD 的计算结果不会互相影响。当我们需要对若干批次的数据处理结果进行聚合的时候，就需要设置一个更大的时间窗口，如下图所示。</p>
<p data-nodeid="8758" class=""><img src="https://s0.lgstatic.com/i/image/M00/20/84/CgqCHl7ohPCAS2oAAADX2hrAMOk915.png" alt="image" data-nodeid="8762"></p>



<p data-nodeid="2373">时间窗口是由批次间隔组成的有限时间跨度，基于窗口的操作对窗口中所有数据进行处理。此外，窗口是一个逻辑的概念，它可以进行滑动，图中所示的窗口跨度为 3，滑动步长为 2，滑动意味着每隔多少时间，窗口会被触发一次，每批次数据与窗口的对应关系为一对多，意味着某个批次的数据可以存在于多个窗口中。这里注意窗口间隔与滑动步长都必须是 DStream 批次间隔的整数倍。</p>
<p data-nodeid="2374">基于窗口的转换算子主要有 slice、window、countByWindow、reduceByWindow、reduceByKeyAndWindow、countByValueAndWindow 等。</p>
<h4 data-nodeid="2375">slice 算子</h4>
<ul data-nodeid="2376">
<li data-nodeid="2377">
<p data-nodeid="2378">def slice(interval: Interval): Seq[RDD[T]] 和 def slice(fromTime: Time, toTime: Time): Seq[RDD[T]]：slice 算子返回该时间跨度内的 RDD 集合，批次间隔可以用 Interval 进行定义，也可以用起始时间与结束时间来定义，注意，开始时间与结束时间需要是批次间隔的倍数，否则系统会自动进行取整。该算子相当于在整个 DStream 流中截取了一段。</p>
</li>
</ul>
<h4 data-nodeid="2379">window 算子</h4>
<ul data-nodeid="2380">
<li data-nodeid="2381">
<p data-nodeid="2382">window(windowDuration: Duration): DStream[T] 和 window(windowDuration: Duration, slideDuration: Duration): DStream[T]：window 算子定义了窗口的属性，如跨度（windowDuration）和滑动步长（slideDuration），并返回一个新的 DStream，默认的滑动步长为批次间隔。当我们通过 window 算子定义了滑动窗口以后，可以用使用 <strong data-nodeid="2501">join 算子</strong>进行连接操作，例：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="2383"><code data-language="scala"><span class="hljs-keyword">import</span> org.apache.spark.streaming.<span class="hljs-type">StreamingContext</span>
<span class="hljs-keyword">import</span> org.apache.spark.streaming.<span class="hljs-type">Seconds</span>
<span class="hljs-keyword">import</span> org.apache.spark.streaming.dstream.<span class="hljs-type">ConstantInputDStream</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
<span class="hljs-keyword">import</span> org.apache.spark.streaming.dstream.<span class="hljs-type">DStream</span>.toPairDStreamFunctions
	&nbsp;
	&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">SparkStreamingJoin</span> </span>{
	
	&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
	&nbsp;&nbsp;&nbsp;&nbsp; .builder
	&nbsp;&nbsp;&nbsp;&nbsp; .master(<span class="hljs-string">"local[2]"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"SparkStreamingJoin"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; .getOrCreate()
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> sc = spark.sparkContext
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> ssc = <span class="hljs-keyword">new</span> <span class="hljs-type">StreamingContext</span>(sc, batchDuration = <span class="hljs-type">Seconds</span>(<span class="hljs-number">2</span>))
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> leftData = sc.parallelize(<span class="hljs-number">0</span> to <span class="hljs-number">3</span>)
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> leftStream = <span class="hljs-keyword">new</span> <span class="hljs-type">ConstantInputDStream</span>(ssc, leftData)
	&nbsp;
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> rightData = sc.parallelize(<span class="hljs-number">0</span> to <span class="hljs-number">2</span>)
	&nbsp;
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> rightStream = <span class="hljs-keyword">new</span> <span class="hljs-type">ConstantInputDStream</span>(ssc, rightData)
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 连接的DStream窗口需要有相同的Duration或者其中一个DStream的Duration是另一个的整数倍</span>
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> rightWindow = rightStream.map(f =&gt; (f,f)).window(<span class="hljs-type">Seconds</span>(<span class="hljs-number">2</span>),<span class="hljs-type">Seconds</span>(<span class="hljs-number">4</span>))
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> leftWindow = leftStream.map(f =&gt; (f,f)).window(<span class="hljs-type">Seconds</span>(<span class="hljs-number">6</span>),<span class="hljs-type">Seconds</span>(<span class="hljs-number">4</span>))
	
	&nbsp;&nbsp;&nbsp;&nbsp; leftWindow.join(rightWindow).print()
	
	&nbsp;&nbsp;&nbsp;&nbsp; ssc.start()
	
	&nbsp;&nbsp;&nbsp;&nbsp; ssc.awaitTermination()
	
	&nbsp;}
	
}
</code></pre>
<p data-nodeid="2384">这里的 join 要求两个窗口的滑动步长必须一致。</p>
<p data-nodeid="2385"><strong data-nodeid="2506">window 算子很重要的用法是与无状态算子配合使用，使其结果满足需要的时间跨度限制。</strong></p>
<h4 data-nodeid="2386">reduceByWindow 算子</h4>
<p data-nodeid="2387">●&nbsp;&nbsp;&nbsp;&nbsp; def reduceByWindow(reduceFunc: (T, T) =&gt; T,windowDuration: Duration,slideDuration: Duration): DStream[T]和def reduceByWindow(reduceFunc: (T, T) =&gt; T,invReduceFunc: (T, T) =&gt; T,windowDuration: Duration,slideDuration: Duration): DStream[T]：按照 reduceFunc 的逻辑对滑动窗口中的数据进行聚合。</p>
<p data-nodeid="9687">后一个 reduceByWindow 是前一个的重载版本，不同之处在于增加了反函数（invReduceFunc）作为参数。反函数存在的作用在于优化那些增量计算的逻辑，如下图所示，假设 reduceFunc 的作用是对窗口内数据进行累计求和，那么在没有 invReduceFunc 的情况下，计算逻辑是 DStream 每个批次的 RDD 先按照 reduceFunc 的逻辑做一次 reduce，然后在达到窗口触发条件时再做一次同样逻辑的 reduce 操作，但是我们可以发现两个窗口互相重叠的时间区间的数据（此处为 RDD@time3）在之前的窗口已经聚合过了，是没有必要再重新计算的，而反函数版本的 reduceByWindow 则针对此处做了优化，反函数的根本作用在于求重复计算部分的值（此处为 RDD@time3）。</p>
<p data-nodeid="9688" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/20/85/CgqCHl7ohVWARE9kAADX2hrAMOk450.png" alt="image" data-nodeid="9692"></p>


<p data-nodeid="2390">反函数的参数 (a, b) 所代表的含义为：a 为之前的时间窗口（此处为 window@time3）聚合的结果，b 为之前的时间窗口与当前时间窗口没有重叠部分的聚合结果（此处为 RDD@time1 与 RDD@time2），那么按照反函数的作用，正确反函数应该为 (a,b) =&gt; a - b，Spark Streaming 最后再将反函数的计算结果（RDD@time3）与当前时间窗口剩余的数据（此处为 RDD@time4 与 RDD@time5）进行聚合，得到当前窗口的聚合结果（此处为 window@time5）。不难发现，在时间窗口本身跨度很大，且两个时间窗口互相重叠的部分也很大时，反函数版本的 reduceByWindow在计算时性能会大大优于普通版本的 reduceByWindow。</p>
<h4 data-nodeid="2391">reduceByKeyAndWindow算子</h4>
<ul data-nodeid="2392">
<li data-nodeid="2393">
<p data-nodeid="2394">def reduceByKeyAndWindow(reduceFunc: (V, V) =&gt; V, windowDuration: Duration): DStream[(K, V)]、def reduceByKeyAndWindow(reduceFunc: (V, V) =&gt; V, windowDuration: Duration,slideDuration: Duration,partitioner: Partitioner): DStream[(K, V)]和def reduceByKeyAndWindow(reduceFunc: (V, V) =&gt; V, invReduceFunc: (V, V) =&gt; V,windowDuration: Duration,slideDuration: Duration = self.slideDuration,numPartitions: Int = ssc.sc.defaultParallelism,filterFunc: ((K, V)) =&gt; Boolean = null): DStream[(K, V)]：通过 reduceFunc 的化简逻辑，reduceByKey 的算子会根据 K 对窗口的数据进行分组聚合，返回化简结果。同时，还可以指定 reduce 任务的个数与 Shuffle 的逻辑。另外 reduceByBeyAndWindow 也有反函数的版本。</p>
</li>
</ul>
<h4 data-nodeid="2395">countByWindow 算子</h4>
<ul data-nodeid="2396">
<li data-nodeid="2397">
<p data-nodeid="2398">def countByWindow(windowDuration: Duration, slideDuration: Duration): DStream[Long]：countByWindow 算子计算滑动窗口中数据的数量。</p>
</li>
</ul>
<p data-nodeid="2399">下面我们来看看该算子的实现：</p>
<pre class="lang-scala" data-nodeid="2400"><code data-language="scala">	<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">countByWindow</span></span>(windowDuration: <span class="hljs-type">Duration</span>,slideDuration: <span class="hljs-type">Duration</span>): <span class="hljs-type">DStream</span>[<span class="hljs-type">Long</span>] = ssc.withScope {
	&nbsp;<span class="hljs-keyword">this</span>.map(_ =&gt; <span class="hljs-number">1</span>L).reduceByWindow(_ + _, _ - _, windowDuration, slideDuration)
	}
</code></pre>
<p data-nodeid="2401">可以看到 countByWindow 先将每行数据转换成 1，最后再用 reduceByWindow 进行累计求和，它默认就采取了反函数的版本，这也是官方推荐的。</p>
<h4 data-nodeid="2402">countByValueAndWindow 算子</h4>
<ul data-nodeid="2403">
<li data-nodeid="2404">
<p data-nodeid="2405">def countByValueAndWindow(windowDuration: Duration, slideDuration: Duration, numPartitions: Int = ssc.sc.defaultParallelism)(implicit ord: Ordering[T] = null): DStream[(T, Long)]：对每个滑动窗口的数据执行 countByValue 的操作。底层实现也是调用了 reduceByKeyAndWindow 的反函数版本。</p>
</li>
</ul>
<p data-nodeid="2406">介绍了基于窗口的转换算子，我们发现基于窗口的转换操作还是有其局限性，当我们想要对某个键的状态进行整个时间段追踪时，基于窗口就不是那么方便了。</p>
<p data-nodeid="2407">所以我们还需要另外一种有状态的转换操作：mapWithState 与 updateStateByKey，mapWithState 是 Spark 1.6 以后的新特性，官方宣称性能是 updateStateByKey 的十倍，可以认为是 updateStateByKey 的升级版。这两种算子类似于定义一个全局累加器，每个批次的数据处理结果都会将其更新，这样就能得到整个时间段下该 key 的状态值（中间结果）。以 wordcount 为例：</p>
<pre class="lang-scala" data-nodeid="2408"><code data-language="scala"><span class="hljs-keyword">import</span> org.apache.spark.<span class="hljs-type">SparkConf</span>
<span class="hljs-keyword">import</span> org.apache.spark.streaming._
<span class="hljs-keyword">import</span> org.apache.spark.streaming.dstream.<span class="hljs-type">DStream</span>.toPairDStreamFunctions
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
	&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">StatefulNetworkWordCount</span> </span>{
	
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]) {
	
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
	&nbsp;&nbsp;&nbsp;&nbsp; .builder
	&nbsp;&nbsp;&nbsp;&nbsp; .master(<span class="hljs-string">"local[2]"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"StatefulNetworkWordCount"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; .getOrCreate()
	
	&nbsp; &nbsp;&nbsp;<span class="hljs-keyword">val</span> sc = spark.sparkContext
	
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> ssc = <span class="hljs-keyword">new</span> <span class="hljs-type">StreamingContext</span>(sc, <span class="hljs-type">Seconds</span>(<span class="hljs-number">1</span>))
	&nbsp;&nbsp;&nbsp; ssc.checkpoint(<span class="hljs-string">"."</span>)
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> initialRDD = ssc.sparkContext.parallelize(<span class="hljs-type">List</span>((<span class="hljs-string">"hello"</span>, <span class="hljs-number">1</span>), (<span class="hljs-string">"world"</span>, <span class="hljs-number">1</span>)))
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> lines = ssc.socketTextStream(args(<span class="hljs-number">0</span>), args(<span class="hljs-number">1</span>).toInt)
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> words = lines.flatMap(_.split(<span class="hljs-string">" "</span>))
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> wordDstream = words.map(x =&gt; (x, <span class="hljs-number">1</span>))
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 该函数定义了状态更新的逻辑</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> mappingFunc = (word: <span class="hljs-type">String</span>, one: <span class="hljs-type">Option</span>[<span class="hljs-type">Int</span>], state: <span class="hljs-type">State</span>[<span class="hljs-type">Int</span>]) =&gt; {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> sum = one.getOrElse(<span class="hljs-number">0</span>) + state.getOption.getOrElse(<span class="hljs-number">0</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> output = (word, sum)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; state.update(sum)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output
	&nbsp;&nbsp;&nbsp; }
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> stateDstream = wordDstream.mapWithState(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">StateSpec</span>.function(mappingFunc).initialState(initialRDD))
	
	&nbsp;&nbsp;&nbsp; stateDstream.print()
	
	&nbsp;&nbsp;&nbsp; ssc.start()
	&nbsp;&nbsp;&nbsp; ssc.awaitTermination()
	
	&nbsp; }
}
</code></pre>
<p data-nodeid="2409">例子中的核心就是 mappingFunc 的函数，定义了状态更新的逻辑。updateStateByKey 的用法大同小异，也是通过定义状态更新函数来体现状态的变化。用法如下：</p>
<pre class="lang-scala" data-nodeid="2410"><code data-language="scala">	……
	<span class="hljs-keyword">val</span>&nbsp;updateFunc&nbsp;=&nbsp;(values:&nbsp;<span class="hljs-type">Seq</span>[<span class="hljs-type">Int</span>],&nbsp;state:&nbsp;<span class="hljs-type">Option</span>[<span class="hljs-type">Int</span>])&nbsp;=&gt;&nbsp;{
	<span class="hljs-keyword">val</span>&nbsp;currentCount&nbsp;=&nbsp;values.sum
	<span class="hljs-keyword">val</span>&nbsp;previousCount&nbsp;=&nbsp;state.getOrElse(<span class="hljs-number">0</span>)
	<span class="hljs-type">Some</span>(currentCount&nbsp;+&nbsp;previousCount)
	}
	&nbsp;
	<span class="hljs-keyword">val</span>&nbsp;newUpdateFunc&nbsp;=&nbsp;(iterator:&nbsp;<span class="hljs-type">Iterator</span>[(<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">Seq</span>[<span class="hljs-type">Int</span>],&nbsp;<span class="hljs-type">Option</span>[<span class="hljs-type">Int</span>])])&nbsp;=&gt;&nbsp;{
	iterator.flatMap(t&nbsp;=&gt;&nbsp;updateFunc(t._2,&nbsp;t._3).map(s&nbsp;=&gt;&nbsp;(t._1,&nbsp;s)))
	}
	&nbsp;
	<span class="hljs-keyword">val</span>&nbsp;stateDstream&nbsp;=&nbsp;wordDstream.updateStateByKey[<span class="hljs-type">Int</span>](newUpdateFunc,&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">HashPartitioner</span>(ssc.sparkContext.defaultParallelism),&nbsp;<span class="hljs-literal">true</span>,&nbsp;initialRDD)
</code></pre>
<p data-nodeid="2411">这两种实现同样功能的算子在性能上差异巨大的原因在于，updateStateByKey 的原理是将上次计算结果与新批次数据采用 cogroup 操作再进行聚合，而 mapWithState 则是通过维护一个中间状态表，存储上一次计算的结果与当前批次的计算结果，所以直接进行聚合处理即可。</p>
<h3 data-nodeid="2412">小结</h3>
<p data-nodeid="2413">本课时介绍了 Spark Streaming 的关键抽象与架构，Spark Streaming 沿用了 RDD 原有的抽象，将流处理变成了连续的微批处理。如果把原有的数据处理看成是一维的，那么流处理无疑是二维的：它加入了一个很重要的时间维度，并且处理的需求往往与时间维度紧密相关，这就使得虽然 Spark Streaming 仍然使用了 RDD 的抽象，但转换算子分为了有状态和无状态之分。</p>
<p data-nodeid="2414">最后给你留一个思考题：</p>
<p data-nodeid="2415">如何每 2 分钟统计一次最近 5 分钟出现过的每个单词的数量？</p>

---

### 精选评论

##### **强：
> slice 算子虽然有两个版本，但是由于<span style="color: rgb(0, 0, 0); font-family: Menlo; font-size: 9pt;">Interval</span><span style="font-size: 0.427rem;">&nbsp;类是私有的，所以能直接使用的只有两个入参的那个版本</span><br>

##### *艺：
> def countByWindow(windowDuration: Duration, slideDuration: Duration): DStream[Long]：countByWindow 算子计算滑动窗口中数据的数量。">val ssc = new StreamingContext(conf,Seconds(120))val words = lines.flatMap(_.split(""))val pairs = words.map(word=(word,1))val windows = pairs.reduceByKeyAndWindow((a:Int,b:Int)=

##### **强：
> org.apache.spark.sql.SparkSession  哪个版本有SparkSession呢，我试了 2.4.7 都没有

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在啊，可以看链接https://github.com/apache/spark/blob/master/sql/core/src/main/scala/org/apache/spark/sql/SparkSession.scala

