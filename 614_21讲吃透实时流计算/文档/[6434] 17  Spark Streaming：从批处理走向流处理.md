<p data-nodeid="4111">今天，我们还是从<strong data-nodeid="4203">系统架构、流的描述、流的处理、流的状态、消息处理可靠性</strong>这五个方面对 Spark Streaming 进行分析和讲解。</p>
<h3 data-nodeid="4112">Spark Streaming</h3>
<p data-nodeid="4113">说到 Spark Streaming，还得从 Spark 谈起。如今在大数据的世界里，Spark 早已是众所周知的大数据处理和分析框架。Spark 在其诞生之初，由于采用内存计算和 DAG 简化处理流程的原因，使得大数据处理性能得到显著提升，一下子就将传统大数据批处理框架 Hadoop MapReduce 比了下去，取而代之成为大数据领域最耀眼的明星框架。</p>
<p data-nodeid="4114">后来随着流计算技术的兴起，Spark 在批处理领域取得巨大成功之后，也开始将其触角延伸到流计算领域，于是诞生了 Spark Streaming。Spark Streaming 是一种建立在 Spark 批处理技术上的流计算框架，它提供了可扩展、高吞吐和错误容忍的流数据处理功能。</p>
<p data-nodeid="4115">我们从 Spark Streaming 的系统架构中，就能看到 Spark Streaming 流计算技术和 Spark 批处理技术，这两者之间一脉相承的关系。</p>
<h3 data-nodeid="4116">系统架构</h3>
<p data-nodeid="4117">下面的图 1 描述了 Spark Streaming 的工作原理。</p>
<p data-nodeid="5371" class=""><img src="https://s0.lgstatic.com/i/image6/M01/20/52/CioPOWBS_7eAXRqXAAGiz_WNEU8237.png" alt="Drawing 1.png" data-nodeid="5374"></p>



<p data-nodeid="4121">从上面的图 1 可以看出，当 Spark Streaming 接收到流数据时，先是将其切分成一个个的 RDD（Resilient Distributed Datasets，弹性分布式数据集），每个 RDD 实际是一个小的块数据。然后，这些 RDD 块数据再由 Spark 引擎进行各种处理。最后，处理完的结果同样是以一个个的 RDD 块数据依次输出。</p>
<p data-nodeid="4122">所以，<strong data-nodeid="4223">Spark Streaming 本质上是将流数据分成一段段块数据后，进行连续不断地批处理</strong>。</p>
<h3 data-nodeid="4123">流的描述</h3>
<p data-nodeid="4124">接下来，我们就来看看在 Spark Streaming 中如何描述一个流计算过程。</p>
<p data-nodeid="4125">由于 Spark Streaming 是构建在 Spark 之上，而 Spark 的核心是一个针对 RDD 块数据做批处理的执行引擎。所以 Spark Streaming 在描述流时，采用了“模版”的概念。具体如下图 2 所示。</p>
<p data-nodeid="5863" class=""><img src="https://s0.lgstatic.com/i/image6/M00/20/52/CioPOWBS_-6Aa4RJAAJLOFofOxM909.png" alt="Drawing 3.png" data-nodeid="5866"></p>



<p data-nodeid="4129">上面的图 2 说明了 Spark Streaming 是如何描述流计算过程的，具体如下：</p>
<ul data-nodeid="4130">
<li data-nodeid="4131">
<p data-nodeid="4132">首先是 RDD。它是 Spark 引擎的核心概念，代表了一个数据集合，是 Spark 进行数据处理的计算单元。</p>
</li>
<li data-nodeid="4133">
<p data-nodeid="4134">然后是 DStream。它是 Spark Streaming 对流的抽象，代表了连续数据流。<strong data-nodeid="4241">在 Spark Streaming 系统内部，DStream 是同一类 RDD 的模板</strong>，因而我们也可以将 DStream 视为由同一类 RDD 组成的序列，每个 RDD 代表了一段间隔内的流数据。</p>
</li>
<li data-nodeid="4135">
<p data-nodeid="4136">然后是 Transformation。它代表了 Spark Streaming 对 DStream（也就是同一类 RDD） 的处理逻辑。目前， DStream 提供了很多 Transformation 相关 API，包括 map、flatMap、filter、reduce、union、join、transform 和 updateStateByKey 等。通过这些 API，可以对 DStream 做各种转换，从而将一个数据流变为另一个数据流。</p>
</li>
<li data-nodeid="4137">
<p data-nodeid="4138">接着是 DStreamGraph。它是 Spark Streaming 对流计算过程的描述，也就是 DAG。<strong data-nodeid="4248">在 Spark Streaming 系统内部，DStreamGraph 代表了 RDD 处理过程 DAG 的模板</strong>。DStream 通过 Transformation 可以连接成 DStreamGraph，这就相当于用“点”和“线”画成 DAG 的过程。</p>
</li>
<li data-nodeid="4139">
<p data-nodeid="4140">最后是 Output Operations。它是 Spark Streaming 将 DStream 输出到控制台、数据库或文件系统等外部系统的操作。目前，DStream 支持的 Output Operations 包括 print、saveAsTextFiles、saveAsObjectFiles、saveAsHadoopFiles 和 foreachRDD。由于这些操作会触发外部系统访问，所以 DStream 各种转化的执行实际上是由这些操作触发的。</p>
</li>
</ul>
<p data-nodeid="4141">从上面 Spark Streaming 的概念可以看出，对应到我们在模块二中讨论过的流计算框架组成，DStreamGraph 对应了 DAG，DStream 相当于 DAG 中的“队列”， 而 Transformation 和 Output Operations 则相当于执行任务的“节点”。</p>
<h3 data-nodeid="4142">流的处理</h3>
<p data-nodeid="4143">接下来，我们再来看 Spark Streaming 中的流是怎么被处理的。与 Storm 类似，我们从<strong data-nodeid="4257">流的输入、流的处理、流的输出和反向压力四个方面</strong>来讨论。</p>
<p data-nodeid="4144">首先是<strong data-nodeid="4263">流的输入</strong>。Spark Streaming 提供了三种创建输入数据流的方式。</p>
<ul data-nodeid="4145">
<li data-nodeid="4146">
<p data-nodeid="4147">一是基础数据源，通过 StreamingContext 的相关 API，直接构建输入数据流。这类 API 通常是从 socket、文件或内存中构建输入数据流，比如 socketTextStream、textFileStream、queueStream 等。</p>
</li>
<li data-nodeid="4148">
<p data-nodeid="4149">二是高级数据源，通过外部工具类从 Kafka、Flume、Kinesis 等消息中间件或消息源构建输入数据流。</p>
</li>
<li data-nodeid="4150">
<p data-nodeid="4151">三是自定义数据源，当用户实现了 org.apache.spark.streaming.receiver 抽象类时，就可以实现一个自定义的数据源了。</p>
</li>
</ul>
<p data-nodeid="4152">由于 Spark Streaming 是用 DStream 来表示数据流，所以输入数据流也表示为 DStream。下面的代码（<a href="https://github.com/alain898/realtime_stream_computing_course/blob/main/course17/src/main/java/com/alain898/course/realtimestreaming/course17/sparkstreaming/WordCountExample.java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="4270">本课时完整代码请参考这里</a>）演示了从 TCP 连接中构建文本数据输入流的过程。</p>
<pre class="lang-java" data-nodeid="4153"><code data-language="java">SparkConf conf = <span class="hljs-keyword">new</span> SparkConf().setMaster(<span class="hljs-string">"local[2]"</span>).setAppName(<span class="hljs-string">"WordCountExample"</span>);
JavaStreamingContext jssc = <span class="hljs-keyword">new</span> JavaStreamingContext(conf, Durations.seconds(<span class="hljs-number">1</span>));
JavaReceiverInputDStream&lt;String&gt; lines = jssc.socketTextStream(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">9999</span>);
</code></pre>
<p data-nodeid="4154">在上面的代码中，我们用 socketTextStream 创建了一个从本地 9999 端口接收文本数据的输入流。</p>
<p data-nodeid="6828" class="">然后是<strong data-nodeid="6834">流的处理</strong>。Spark Streaming 对流的处理，是通过 DStream 的各种转化操作 API 完成。DStream 的转换操作大体上包含了三类。</p>


<ul data-nodeid="4156">
<li data-nodeid="4157">
<p data-nodeid="4158"><strong data-nodeid="4282">第一类是常用的流式处理操作</strong>，比如 map、filter、reduce、count、transform 等。</p>
</li>
<li data-nodeid="4159">
<p data-nodeid="4160"><strong data-nodeid="4287">第二类是流数据状态相关的操作</strong>，比如 union、join、cogroup、window 等。</p>
</li>
<li data-nodeid="4161">
<p data-nodeid="4162"><strong data-nodeid="4292">第三类则是流信息状态相关的操作</strong>，目前有 updateStateByKey 和 mapWithState。</p>
</li>
</ul>
<p data-nodeid="4163">下面是一个对 DStream 进行转化处理的例子。</p>
<pre class="lang-java" data-nodeid="4164"><code data-language="java"><span class="hljs-comment">// 将每一行分割成单词，然后统计单词出现次数</span>
JavaDStream&lt;String&gt; words = lines.flatMap(x -&gt; Arrays.asList(x.split(<span class="hljs-string">" "</span>)).iterator());
JavaPairDStream&lt;String, Integer&gt; pairs = words.mapToPair(s -&gt; <span class="hljs-keyword">new</span> Tuple2&lt;&gt;(s, <span class="hljs-number">1</span>));
JavaPairDStream&lt;String, Integer&gt; wordCounts = pairs.reduceByKey((i1, i2) -&gt; i1 + i2);
</code></pre>
<p data-nodeid="7313">在上面的代码中，先将从 socket 中读出的文本流 lines ，对每行文本分词后，用 flatMap 转化为单词流 words。然后用 mapToPair 将单词流 words 转化为计数元组流 pairs。最后，以单词为分组进行数量统计，通过 reduceByKey 转化为单词计数流 wordCounts。</p>
<p data-nodeid="7314">接下来是<strong data-nodeid="7321">流的输出</strong>。Spark Streaming 允许 DStream 输出到外部系统，这是通过 DStream 的各种输出操作完成的。DStream 的输出操作可以将数据输出到控制台、文件系统或数据库等。</p>

<p data-nodeid="4166">目前 DStream 的输出操作有 print、saveAsTextFiles、saveAsHadoopFiles 和 foreachRDD 等。其中 foreachRDD 是一个通用的 DStream 输出接口，用户可以通过 foreachRDD 自定义各种 Spark Streaming 输出方式。下面的例子演示了将单词计数流打印到控制台。</p>
<pre class="lang-java" data-nodeid="4167"><code data-language="java">wordCounts.print();
</code></pre>
<p data-nodeid="4168">最后是<strong data-nodeid="4308">反向压力</strong>。早期版本 Spark 不支持反向压力，但从 Spark 1.5 版本开始，Spark Streaming 也引入了反向压力功能，这是不是正说明了反向压力功能对流计算系统的必要性！默认情况下 Spark Streaming 的反向压力功能是关闭的。当要使用反向压力功能时，需要将 spark.streaming.backpressure.enabled 设置为 true。</p>
<p data-nodeid="4169">整体而言，Spark 的反向压力借鉴了工业控制中 PID 控制器的思路，其工作原理如下。</p>
<ul data-nodeid="4170">
<li data-nodeid="4171">
<p data-nodeid="4172">首先，当 Spark 在处理完每批数据时，统计每批数据的处理结束时间、处理时延、等待时延、处理消息数等信息。</p>
</li>
<li data-nodeid="4173">
<p data-nodeid="4174">然后，根据统计信息估计处理速度，并将这个估计值通知给数据生产者。</p>
</li>
<li data-nodeid="4175">
<p data-nodeid="4176">最后，数据生产者根据估计出的处理速度，动态调整生产速度，最终使得生产速度与处理速度相匹配。</p>
</li>
</ul>
<h3 data-nodeid="4177">流的状态</h3>
<p data-nodeid="4178">接下来，我们再来看 Spark Streaming 中流的状态问题。Spark Streaming 关于流的状态管理，也是在 DStream 提供的转化操作中实现的。</p>
<p data-nodeid="4179">我们先来看<strong data-nodeid="4320">流数据状态</strong>。由于 DStream 本身就是将数据流分成 RDD 做批处理，所以 Spark Streaming 天然就需要对数据进行缓存和状态管理。换言之，组成 DStream 的一个个 RDD，就是一种流数据状态。DStream 提供了一些 window 相关的 API，实现了对流数据的窗口管理，并基于窗口实现了 count 和 reduce 这两类聚合功能。另外，DStream 还提供了union、join 和 cogroup 三种在多个流之间进行关联操作的 API。以上窗口和关联操作相关的 API，也属于 Spark 对流数据状态的支持。</p>
<p data-nodeid="4180">说完流数据状态，我们再来看<strong data-nodeid="4326">流信息状态</strong>。DStream 的 updateStateByKey 和 mapWithState 操作提供了流信息状态管理的方法。updateStateByKey 和 mapWithState 都可以基于 key 来记录历史信息，并在新的数据到来时，对这些信息进行更新。</p>
<p data-nodeid="4181">不同的是，updateStateByKey 会返回记录的所有历史信息，而 mapWithState 只会返回处理当前一批数据时更新的信息。就好像，前者是在返回一个完整的直方图，而后者则只返回直方图中发生变化的柱条。</p>
<p data-nodeid="4182">由此可见， mapWithState 比 updateStateByKey 的性能会优越很多。而且从功能上讲，如果不是用于报表生成的场景，大多数实时流计算应用中，使用 mapWithState 也会更合适。</p>
<h3 data-nodeid="4183">消息处理可靠性</h3>
<p data-nodeid="4184">最后，我们来看下 Spark Streaming 中消息处理可靠性的问题。Spark Streaming 对消息可靠性的保证，是由数据接收、数据处理和数据输出共同决定的。从 1.2 版本开始，Spark 引入 WAL（write ahead logs）机制，可以将接收的数据先保存到错误容忍的存储上。当打开 WAL 机制后，再配合可靠的数据接收器（比如 Kafka），Spark Streaming 能够提供“至少一次”的消息接收。从 1.3 版本开始，Spark 又引入了 Kafka Direct API，进而可以实现“精确一次”的消息接收。</p>
<p data-nodeid="4185">由于 Spark Streaming 对数据的处理是基于 RDD 完成，而 RDD 提供了“精确一次”的消息处理。所以在数据处理部分，Spark Streaming 天然具备“精确一次”的消息可靠性保证。</p>
<p data-nodeid="4186">但是，Spark Streaming 的数据输出部分目前只具备“至少一次”的可靠性保证。也就是说，经过处理后的数据，可能会被多次输出到外部系统。</p>
<p data-nodeid="4187">在一些场景下，这个不会有什么问题。比如输出数据是保存到文件系统，重复发送的结果只是覆盖掉之前写过一遍的数据。</p>
<p data-nodeid="4188">但是在另一些场景下，比如需要根据输出增量更新数据库，那就需要做一些额外的去重处理了。一种可行的方法是，在各个 RDD 中新增一个唯一标志符来表示这批数据，然后在写入数据库时，使用这个唯一标志符来检查数据之前是否写入过。当然，这时写入数据库的动作需要使用事务来保证完整性了。</p>
<h3 data-nodeid="4189">小结</h3>
<p data-nodeid="7800" class="">总的来说，Spark Streaming 作为一种<strong data-nodeid="7818">从批处理框架发展而来的流计算框架</strong>，代表了大数据领域的<strong data-nodeid="7819">两种趋势</strong>。一是，<strong data-nodeid="7820">部分原本用批处理完成的任务，正逐渐转向由流处理来完成</strong>。二是，这种 <strong data-nodeid="7821">“流”“批”一体的框架越来越成为主流</strong>，毕竟用一个框架完成两种不同类型的计算任务，会极大地简化系统的复杂程度。</p>

<p data-nodeid="4191">但是，Spark Streaming 这种将流数据按批处理的方式，还是存在一定的问题。问题主要是，Spark Streaming 将数据从流数据划分为 RDD 块数据时，时间间隔不能设置得太短，目前比较典型的最小时间间隔是 100 毫秒。这就意味着，Spark Streaming 处理流数据的时间延迟，最少会是 100 毫秒。</p>
<p data-nodeid="4192">这对于实时要求更加严苛的场景是不适用的，同时这也意味着 Spark Streaming 并不能按逐条的方式，对流数据进行处理。这点需要你在实际开发过程中尤其注意，看你的业务场景是否可以接受 100 毫秒以上的延迟，以及是否不需要逐事件处理。</p>
<p data-nodeid="4193">最后留一个小作业，如何分别使用 Spark Streaming 的流数据状态 API 和流信息状态 API 实现“过去 24 小时同一种商品的交易总金额”这种计算呢？可以将你的想法和问题在留言区写下来，我看到后会进行分析和解答。</p>
<p data-nodeid="4194">下面是本课时的脑图，以帮助你理解。</p>
<p data-nodeid="8302" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/20/52/CioPOWBTABeAf-AEABKJEPt9vJE024.png" alt="Drawing 4.png" data-nodeid="8305"></p>

---

### 精选评论

##### **9160：
> 冒昧和楼上的哥们探讨下，我们和你业务差不多，甚至比你的还复杂，我们四五张核心表才能算一个指标都是简单的了。我们目前是kafka+spark streaming+kudu，维表在hbase，目前我们的做法是分几个spark streaming去做整个链路
1、第一个主要是消费kafka里面的采集到的核心库的增删改的增量数据，然后分别去更新对应的dw层的表，然后把本批次有可能影响到的主键信息去重回流到kafka，
2、第二个spark streaming消费这个topic，回查dw层的数据，我们把这个动作成为放大读操作，这样就拿到了几个主键的小全量数据，然后做指标计算，轻度汇总。
3、再把第二个流程涉及到的人的信息回流到kafka，启动一个spark streaming去消费，再按照第二个的思路，拿到人的小全量去做计算。
整个链路基本控制在30秒左右，关于高度汇总层还是跑分钟的批，同时也请教下老师，还有优化的点么？感觉领导还是不太满意，但逻辑实在太复杂了。麻烦老师有时间回复下，领导还想着flink去做，我想哭😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈哈，看完你描述的过程，我觉得从整个处理流程而言，你已经做得很漂亮了哈，因为你的整个处理链路都将“增量”计算的优势发挥出来了，避免了很多重复和全量的计算。并且你这种“轻度汇总”和“高度汇总”结合的方式也是符合Lambda架构的。至于要不要用Flink，我和你们领导的意见是一致的，我也推荐你尝试下。因为Flink在流处理这方面，API易用性和丰富程度，以及性能，都比Spark Streaming更有优势。比如，我能够想到的就是，在Flink里，你在步骤1->2，以及2->3时产生的主键信息，不必再回流到Kafka里，而是直接在Flink里通过DataStream或KeyedStream做进一步处理。这样，整个1->2->3都是可以放在一个Flink应用中实现的，避免了不必要的Kafka传输，应该也可以降低你整个程序流程的复杂度。然后对于“轻度汇总”和“高度汇总”结合的方式，使用Flink的话，可能可以改造成Kappa架构，这样对于“轻度汇总”和“高度汇总”，就只需要写一套Flink流计算的代码了。当然，这个还是需要看你的“高度汇总”的逻辑是不是和“轻度汇总”是一致的，以及是否容易改成Flink流处理过程。最后，不管你是Java工程师还是大数据工程师，我都觉得Flink是值得深入研究的工具，你真的不妨尝试下。再说工作嘛，你不做这件事，还是有其他事情要做的。（手动滑稽脸）

##### *里：
> 老师好，看到前边您的回答，也希望您先为我解惑一下。我们项目目前的技术选型 为kafka+sparkstreaming+hbase,也会涉及历史数据的更新和删除 ；对于以下业务案例：select a.a1,a.a2 ,b.b1,b.b2 ,c.c1from A ainner join B b on b.key=a.keyleft C c on c.key=b.keywhere ....对于 A B C 的任意一张表的变更，都需要触发该业务场景 ，因此 使用了 hbase 的 get/scan 来查询数据，生成结果数据 写入hbase,这一过程涉及 旧数据删除和新数据更新，对于贴源层数据更新， 需要依次变更 明细层 和 轻度汇总层问题1：当A:B 是1：N 的关系时，这个 N 正常情况下 N 1 万 ，在这种情况下就会内存溢出，这个情况 除了 增加内存，是否有更好的办法？问题2：在明细层可以根据数据的版本号保证数据的最终一致性，但是在轻度汇总层，程序异常终止或者被 用 yarn 命令杀死，轻度汇总层就无法保证正确性，现在我们的办法是额外启动一个跑批程序来修正这个问题，请问有没有更好的办法 ；问题3：这个sparkstreaming程序直接使用foreachpartition 处理数据，但是我的想法是使用一些算子 对数据做一个 分组 去重 来 减少 hbase 的操作 ，但是被领导否决了，他认为打乱了 kafka和程序的partition的对应关系，不好管理offset,导致丢数据 ;望解惑

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第1个问题：如果一个查询比较复杂，内存放不下的话，你可以将一个SQL分成多步来实现，并且将where条件提前到join之前，这样可以减少很多数据。再就是将中间计算结果持久化到HDFS上，这样也会减少内存使用。

第2个问题：我认为你们现在的方式就是比较好的方案了。从你的描述来看，你们的轻度汇总层就有点像Lambda架构中的快速处理层和服务层的功能。以Lambda架构的实践来看，数据最重要的是不变性和可重复计算性。每次批处理层计算完成时，就会用新的计算结果覆盖掉之前旧的快速处理层和批处理层合并结果。你们现在的方案，实际上就是保证了数据的不变性和可重复计算性，用重跑任务的方式修正之前错误的结果，我认为这就是很好的方案了。可能更好的改进是，将这种任务做成定期运行的任务，这就有点像checkpoint的思路了。

第3个问题：我认为你的思路是非常正确的。可以先将流数据读取出来，读取出的流数据被描述为DStream/RDD，之后DStream/RDD在处理过程中，本身就可以达到exactly-once可靠性级别。Spark从1.3版本引入的Kafka Direct API，是能够保证从Kafka读取消息时是exactly-once级别可靠性的。所以，我认为从Kafka读取消息后，再用算子对数据做分组进行去重的思路是正确的，这并不会影响丢不丢数据。总的来说，你只需要按照这么一种思路来看待问题，就是保证消息输入和处理是exactly-once级别的可靠性，输出部分是“幂等”的效果，这样整个流程就都是exactly-once的，不存在重复或丢数据的问题。因为整个过程，从RDD的依赖关系来看，就是一次次的“快照”而已，可能Flink在这方面比Spark Streaming做得更直观些。

