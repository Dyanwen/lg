<p data-nodeid="13710">今天，我们来看第四种开源流计算框架 Apache Flink。我们继续从<strong data-nodeid="13716">系统架构、流的描述、流的处理、流的状态、消息处理可靠性</strong>这五个方面来进行分析和讲解。</p>



<h3 data-nodeid="12123">Apache Flink</h3>
<p data-nodeid="12124">在流计算技术不断发展的今天，有关流计算的理论和模型已经构建得比较清晰和完善。Apache Flink（后简称 Flink） 就是这些流计算领域最新理论和模型的优秀实践。相比 Spark 在批处理领域的火爆流行，Flink 可以说是目前流计算领域最耀眼的明星框架了。</p>
<p data-nodeid="12125">Flink 和 Spark 都是“流”“批”一体的分布式计算框架，但不同于<strong data-nodeid="12339">Spark 的核心是“批处理”引擎，Flink 的核心是“流计算”引擎</strong>。</p>
<h3 data-nodeid="12126">系统架构</h3>
<p data-nodeid="12127">我们先来看 Flink 的系统架构。</p>
<p data-nodeid="12128">Flink 是一个主从（Master/Worker）架构的分布式系统。主节点负责调度流计算作业，管理和监控任务执行。当主节点从客户端接收到作业相关的 JAR 包和资源后，进行分析和优化，生成执行计划，也就是需要执行的任务，然后将相关的任务分配给各个 Worker，由 Worker 负责任务的具体执行。</p>
<p data-nodeid="12129">图 1 展示了 Flink 的系统架构。</p>
<p data-nodeid="14769" class=""><img src="https://s0.lgstatic.com/i/image6/M01/27/A9/CioPOWBdgJuAFv9cAAMbscARMZA482.png" alt="Drawing 1.png" data-nodeid="14772"></p>



<p data-nodeid="12133">可以看出，Flink 可以部署在诸如 Yarn、Mesos 和 K8S 等分布式资源管理器上。其整体架构与 Storm 和 Spark Streaming 等分布式流计算框架类似，但与这些流计算框架不同的是，<strong data-nodeid="12356">Flink 明确地把状态管理（尤其是流信息状态管理）纳入到了其系统架构中</strong>。</p>
<p data-nodeid="12134">在 Flink 计算节点执行任务的过程中，可以将状态保存到本地。通过 checkpoint 机制，再配合诸如 HDFS、S3 和 NFS 这样的分布式文件系统，Flink 在不降低性能的同时，实现了状态的分布式管理。</p>
<h3 data-nodeid="12135">流的描述</h3>
<p data-nodeid="12136">接下来，我们再来看看在 Flink 中如何描述一个流计算过程。下面的图 2 展示了 Flink 用于描述流计算过程的 DAG 组成。</p>
<p data-nodeid="15813" class=""><img src="https://s0.lgstatic.com/i/image6/M01/27/A9/CioPOWBdgKiAVR1QAANcMDm_ZVg804.png" alt="Drawing 3.png" data-nodeid="15816"></p>



<p data-nodeid="12140">从图 2 可以看出，Flink 使用了 DataStream 来描述数据流。Flink 的数据输入、数据处理和数据输出均与 DataStream 有关，具体来说是这样的。</p>
<ul data-nodeid="17899">
<li data-nodeid="17900">
<p data-nodeid="17901">首先是<strong data-nodeid="17911">Source。</strong> 它用于描述 Flink 流数据的输入源。当流数据输入 Flink 系统后，就被表示为 DataStream。Flink 的 Source 可以是消息中间件、数据库、文件系统或其他各种数据源。</p>
</li>
<li data-nodeid="17902">
<p data-nodeid="17903" class="">然后是<strong data-nodeid="17917">Transformation。</strong> Transformation 将一个或多个 DataStream 转化为一个新的DataStream，是 Flink 对流数据进行处理的地方。目前，Flink 提供 map、flatMap、filter、keyBy、reduce、fold、aggregations、window、union、join、split、select 和iterate 等类型的 Transformation。</p>
</li>
<li data-nodeid="17904">
<p data-nodeid="17905">最后是<strong data-nodeid="17923">Sink</strong>。Sink 是 Flink 将 DataStream 输出到外部系统的地方，比如写入控制台、数据库、文件系统或消息中间件等。</p>
</li>
</ul>


<p data-nodeid="12148">总的来说，DataStream 在 Flink 中扮演的角色犹如 Spark 中的 RDD，它们都是各自框架中的核心概念。通过对 DataStream 进行各种 Transformation 操作，就形成了描述 Flink 流计算过程的 DAG。</p>
<p data-nodeid="12149">另外值得一提的是，Flink 也支持批处理 DataSet 的概念。不过 Flink 的 DataSet 内部同样是由 DataStream 构成。Flink 这种<strong data-nodeid="12398">将批处理视为流处理</strong>特殊情况的做法，与 Spark Streaming 中<strong data-nodeid="12399">将流处理视为连续批处理</strong>的做法<strong data-nodeid="12400">截然相反</strong>。</p>
<h3 data-nodeid="12150">流的处理</h3>
<p data-nodeid="12151">接下来，我们再来看 Flink 中的流是怎么被处理的。我们从流的输入、流的处理、流的输出和反向压力四个方面来讨论 Flink 中流的处理过程。</p>
<h4 data-nodeid="23091" class="">1. 流的输入</h4>





<p data-nodeid="12155">Flink 使用 StreamExecutionEnvironment.addSource 设置流的数据源 Source。为了使用方便，Flink 在 StreamExecutionEnvironment.addSource 的基础上提供了一些内置的数据源实现。</p>
<p data-nodeid="12156">这些数据源主要包含了四类。</p>
<ul data-nodeid="12157">
<li data-nodeid="12158">
<p data-nodeid="12159">一是，基于文件的输入。从文件中读入数据作为流数据源，比如 readTextFile 和 readFile 等。</p>
</li>
<li data-nodeid="12160">
<p data-nodeid="12161">二是，基于套接字的输入。从 TCP 套接字中读入数据作为流数据源，比如 socketTextStream 等。</p>
</li>
<li data-nodeid="12162">
<p data-nodeid="12163">三是，基于集合的输入。用集合作为流数据源，比如 fromCollection、fromElements、fromParallelCollection 和 generateSequence 等。</p>
</li>
<li data-nodeid="12164">
<p data-nodeid="12165">四是，自定义输入。StreamExecutionEnvironment.addSource 是最通用的流数据源生成方法，用户可以其为基础开发自己的流数据源。一些第三方数据源，比如 flink-connector-kafka 中的 FlinkKafkaConsumer08 就是针对 Kafka 消息中间件开发的流数据源。</p>
</li>
</ul>
<p data-nodeid="12166">Flink 将从数据源读出的数据流表示为 DataStream。下面的代码（<a href="https://github.com/alain898/realtime_stream_computing_course/tree/main/course19?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="12416">本课时完整代码请参考这里</a>）演示了从 TCP 连接中构建文本数据输入流的过程。</p>
<pre class="lang-java" data-nodeid="12167"><code data-language="java"><span class="hljs-keyword">final</span> StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream&lt;String&gt; text = env.socketTextStream(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">9999</span>, <span class="hljs-string">"\n"</span>);
</code></pre>
<p data-nodeid="28211">在上面的代码中，我们用 socketTextStream 创建了一个从本地 9999 端口接收文本数据的输入流。</p>
<h4 data-nodeid="29235" class="">2. 流的处理</h4>






<p data-nodeid="12169">Flink 对流的处理，是通过 DataStream 的各种转化操作完成的。相比 Spark 中 DStream 的转化操作混淆了流数据状态管理和流信息状态管理，<strong data-nodeid="12430">Flink 的设计思路更加清晰，明确地将流信息状态从流数据状态的管理中分离出去</strong>。</p>
<p data-nodeid="12170">DataStream 转化操作只包含了两类操作，一类是常规的流式处理操作，比如 map、filter、reduce、count、transform 等。另一类是流数据状态相关的操作，比如 union、join、coGroup、window 等。</p>
<p data-nodeid="12171">这两类操作都是针对流本身的处理和管理。从设计模式中单一职责原则的角度来看，<strong data-nodeid="12437">Flink 关于流的设计显然更胜一筹</strong>。</p>
<p data-nodeid="12172">下面是一个对 DataStream 进行转化处理的例子。</p>
<pre class="lang-java" data-nodeid="12173"><code data-language="java">DataStream&lt;WordWithCount&gt; windowCounts = text
&nbsp; &nbsp; .flatMap(<span class="hljs-keyword">new</span> FlatMapFunction&lt;String, WordWithCount&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flatMap</span><span class="hljs-params">(String value, Collector&lt;WordWithCount&gt; out)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (String word : value.split(<span class="hljs-string">"\\s"</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.collect(<span class="hljs-keyword">new</span> WordWithCount(word, <span class="hljs-number">1L</span>));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; })
&nbsp; &nbsp; .keyBy(<span class="hljs-string">"word"</span>)
&nbsp; &nbsp; .timeWindow(Time.seconds(<span class="hljs-number">5</span>), Time.seconds(<span class="hljs-number">1</span>))
&nbsp; &nbsp; .reduce(<span class="hljs-keyword">new</span> ReduceFunction&lt;WordWithCount&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> WordWithCount <span class="hljs-title">reduce</span><span class="hljs-params">(WordWithCount a, WordWithCount b)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> WordWithCount(a.word, a.count + b.count);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; });
</code></pre>
<p data-nodeid="34309">在上面的代码中，我们先将从 socket 中读出的文本流 text，用 flatMap 函数对每行文本进行分词，从而将文本流转化为了单词流。然后，我们使用 keyBy 函数，将单词流按照单词（也就是 word）进行分组，从而形成了单词分区流。最后，我们使用 timeWindow 函数，在单词分区流上以 5 秒钟为时间窗口，进行具有“求和”功能的 reduce 聚合计算，这样就得到了每五秒钟内各个单词出现的次数。</p>
<h4 data-nodeid="35325" class="">3. 流的输出</h4>






<p data-nodeid="12175">Flink 使用 DataStream.addSink 设置数据流的输出方法。同时 Flink 在 DataStream.addSink 的基础上提供了一些<strong data-nodeid="12451">内置的流数据输出实现</strong>。</p>
<p data-nodeid="12176">这些流数据输出实现主要包含了四类：</p>
<ul data-nodeid="12177">
<li data-nodeid="12178">
<p data-nodeid="12179">一是，输出到文件系统。将流数据输出到文件系统，比如 writeAsText、writeAsCsv 和 writeUsingOutputFormat。</p>
</li>
<li data-nodeid="12180">
<p data-nodeid="12181">二是，输出到控制台。将流数据打印到控制台，比如 print 和 printToErr。</p>
</li>
<li data-nodeid="12182">
<p data-nodeid="12183">三是，输出到套接字。将流数据输出到 TCP 套接字，比如 writeToSocket。</p>
</li>
<li data-nodeid="12184">
<p data-nodeid="12185">四是，自定义输出。DataStream.addSink 是最通用的流数据输出方法，用户可以其为基础开发自己的流数据输出方法。比如 flink-connector-kafka 中的 FlinkKafkaProducer011 就是针对 Kafka 消息中间件开发的流数据输出方法。</p>
</li>
</ul>
<p data-nodeid="12186">下面的示例演示了将 DataStream 表示的流数据打印到控制台。</p>
<pre class="lang-java" data-nodeid="12187"><code data-language="java">windowCounts.print().setParallelism(<span class="hljs-number">1</span>);
</code></pre>
<p data-nodeid="12188">将上面输入、处理和输出三个部分的代码片段整合起来，就能实现一个具备单词计数功能的 Flink 流计算应用了。</p>
<h4 data-nodeid="40364" class="">4. 反向压力</h4>





<p data-nodeid="12192">Flink 对反向压力的支持非常好，不仅实现了反向压力功能，还直接内置了反向压力的监控功能。Flink 采用容量有限的分布式阻塞队列来进行数据传递，当下游任务从队列中读取消息过慢时，就非常自然地减慢了上游任务往队列中写入消息的速度。这种反向压力的实现思路，和我们在前面 03 课时中使用 JDK 自带的 BlockingQueue 实现反向压力的方法基本一致。</p>
<p data-nodeid="12193">另外值得一提的是，与 Storm 和 Spark Streaming 需要明确打开开关才能使用反向压力功能不一样的是，<strong data-nodeid="12469">Flink 的反向压力功能是天然地包含在了其数据传送方案内的，不需特别再去实现，使用时也无须特别打开开关</strong>。</p>
<h3 data-nodeid="12194">流的状态</h3>
<p data-nodeid="12195">接下来，我们再来看 Flink 中流的状态问题。</p>
<p data-nodeid="12196">Flink 是第一个明确地<strong data-nodeid="12477">将流信息状态管理从流数据状态管理中剥离开来</strong>的流计算框架。大多数其他的流计算框架要么没有流信息状态管理，要么实现的流信息状态管理非常有限，要么流信息状态管理混淆在了流数据状态管理中，使用起来并不方便和明晰。</p>
<p data-nodeid="12197">在<strong data-nodeid="12483">流数据状态</strong>方面，Flink 有关流数据状态的管理，都集中在 DataStream 的转化操作上。这是非常合理的，因为流数据状态管理本身是属于流转化和管理的一部分。比如，流按窗口分块处理、多流的合并、事件乱序处理等功能的实现，虽然也涉及数据缓存和状态操作，但这些功能原本就应该由流计算框架来处理。</p>
<p data-nodeid="12198">DataStream 上与<strong data-nodeid="12489">窗口管理</strong>相关的 API 包括 Window 和 WindowAll。其中 Window 是针对KeyedStream，而 WindowAll 是针对非 KeyedStream。DataStream 基于窗口提供了一系列聚合计算相关的方法，比如 reduce、fold、sum、min、max 和 apply 等。另外，DataStream 还提供了一系列有关流与流之间关联计算的操作，比如 union、join、coGroup 和 connect 等。</p>
<p data-nodeid="12199">除了以上这些对流数据状态的支持方法外，Flink 还提供了非常有特色的 KeyedStream。所谓 KeyedStream 是指将流按照指定的键值（key 值），在逻辑上分成多个独立的流。这些逻辑流在计算时，状态彼此独立、互不影响，但是在物理上这些独立的流可能是合并在同一条物理的数据流中。因此在 KeyedStream 具体实现时，Flink 会在处理每个消息前，将当前运行时上下文切换到键值（key 值）所指定流的上下文，就和切换线程上下文一样，这样优雅地避免了不同逻辑流在运算时的相互干扰。</p>
<p data-nodeid="12200">在<strong data-nodeid="12504">流信息状态</strong>方面，Flink 对流信息状态管理的支持，是它相比当前其他流计算框架更显优势的地方。Flink 在 DataStream 之外，提供了独立的状态管理接口。可以说，<strong data-nodeid="12505">实现流信息状态管理，并将其从对流数据本身的管理中分离出来，是 Flink 在洞悉流计算本质后的明智之举</strong>。因为，<strong data-nodeid="12506">如果说 DataStream 是对数据在时间维度的管理，那么状态接口其实是对数据在空间维度的管理</strong>。Flink 之前的流数据框架对这两个概念的区分可以说并不是非常明确，这也导致它们关于状态的设计不是非常完善，甚至根本没有。</p>
<p data-nodeid="12201">在 Flink 中有两种类型的状态接口：Keyed State 和 Operator State。它们既可以用于流信息状态管理，也可以用于流数据状态管理。</p>
<p data-nodeid="12202">我们先来看<strong data-nodeid="12513">Keyed State</strong>。Keyed State 与 KeyedStream 相关。前面提到，KeyedStream 是对流按照 key 值做出的逻辑划分。每个逻辑流都有自己的上下文，就像每个线程都有自己的线程栈一样。当我们需要在逻辑流中记录一些状态信息时，就可以使用 Keyed State。比如“统计不同 IP 上出现的不同设备数”，就可以将流按照 IP 分成 KeyedStream，这样来自不同 IP 的设备事件，会分发到不同 IP 独有的逻辑流中。然后在逻辑流处理过程中，使用 Keyed State 来记录不同设备数。如此一来，就非常方便地实现了“统计不同 IP 上出现的不同设备数”的功能。</p>
<p data-nodeid="12203">我们再来看<strong data-nodeid="12519">Operator State</strong>。Operator State 与算子有关。其实与 Keyed State 的设计思路非常一致，Keyed State是按 key 值划分状态空间，而 Operator State 则是按照算子的并行度划分状态空间。每个 Operator State 绑定到算子的一个并行实例上，因而这些并行实例在运行时可以维护各自的状态。这有点像线程局部量，每个线程都维护自己的一个状态对象，在运行时互不影响。比如当 Kafka Consumer 在消费同一个 topic 的不同 partition 时，可以用 Operator State 来维护各自消费 partition 的 offset。</p>
<p data-nodeid="12204">另外，从 Flink 1.6 版本开始，Flink 引入了状态生存时间值（State Time-To-Live），这为实际开发中，淘汰过期的状态提供了极大的便利。不过美中不足的是，Flink 虽然针对状态存储，提供了 TTL 机制，但是 TTL 本身实际是一种非常底层的功能。如果 Flink 能够针对状态管理也提供诸如窗口管理这样的功能，会使 Flink 的流信息状态管理更加完善和方便。</p>
<h3 data-nodeid="12205">消息处理可靠性</h3>
<p data-nodeid="12206">最后，我们来看下 Flink 中消息处理可靠性的问题。</p>
<p data-nodeid="12207">Flink 基于 snapshot 和 checkpoint 的故障恢复机制，在 Flink 内部提供了 exactly-once 级别的消息处理可靠性保证。当然，<strong data-nodeid="12528">得到这个保证的前提是，在 Flink 应用中保存状态时必须使用 Flink 内部的状态机制，比如 Keyed State 和 Operator State</strong>。</p>
<p data-nodeid="12208">因为这些 Flink 内部状态的保存和恢复都是包含在 Flink 的故障机制内，由 Flink 系统保证了状态的一致性。如果使用不包含在 Flink 故障恢复机制内的状态存储方案，比如用另外独立的 Redis 记录 PV/UV 统计状态，那么就不能获得 exactly-once 级别的消息处理可靠性保证，而只能得到 at-least-once 级别的可靠性保证了。</p>
<p data-nodeid="12209">要想在 Flink 中，实现从流数据“输入”到“输出”这种“端到端”的 exactly-once 级别的消息处理可靠性保证，还必须有 Flink connectors 配合才行。不同的 Flink connectors 提供了不同级别的可靠性保证。比如在 Source 端，Apache Kafka 提供了 exactly once 保证，Twitter Streaming API 提供了 at most once 保证。在 Sink 端，HDFS rolling sink 提供了 exactly once 保证，Kafka producer 则只提供了 at least once 的保证。</p>
<h3 data-nodeid="12210">小结：如何面对琳琅满目的流计算框架</h3>
<p data-nodeid="12211">今天，我们讨论了 Flink。至此，我们就讨论完了四种主流的开源流计算框架。</p>
<p data-nodeid="12212">下面的表 1 是对模块四所讨论四种流计算框架的比较。</p>

<p data-nodeid="41362" class=""><img src="https://s0.lgstatic.com/i/image6/M01/27/A9/CioPOWBdgRKAGRhrAAEdy_HlR50807.png" alt="Drawing 4.png" data-nodeid="41365"></p>


<p data-nodeid="12315">从上面的表中可以看出，在流计算领域，Flink 在各个方面都更有优势，所以它不愧是当前最好的流计算框架。如果你是刚刚入门流计算领域的话，我最推荐的就是 Flink 了，其次是 Spark Streaming。至于 Storm 和 Samza，你可以了解一下，以帮助理解流计算概念和技术即可。</p>
<p data-nodeid="12316">另外，除了上面四种开源流计算框架外，其实还有一些其他的流计算框架或平台，比如 Akka Streaming、Apache Beam 等，这些流计算框架也各有特色。比如 Akka Streaming 支持丰富灵动的流计算编程 API，可谓是惊艳。再比如 Apache Beam 则是流计算模式的集大成者，可以在底层集成多种不同的流计算框架，试图一统流计算江湖。</p>
<p data-nodeid="12317">既然有这么多的流计算框架，<strong data-nodeid="12626">那我们该如何面对琳琅满目的流计算框架呢</strong>？我认为可以从两个角度来看待这个问题。</p>
<p data-nodeid="12318">从<strong data-nodeid="12632">横向功能特征的角度</strong>来看，其实所有流计算框架的核心概念都是相同的，我们在前面的模块一和模块二中已经详细地讲解了这些概念，比如 DAG、队列、线程、流数据状态、流信息状态、反向压力等。只要我们掌握了这些流计算中的核心概念，把握住了流计算框架中各种问题的关键所在，那么在面对这些流计算框架时，就不会感到眼花缭乱，乱了阵脚。</p>
<p data-nodeid="12319">从<strong data-nodeid="12642">纵向发展历史的角度</strong>来看，以 Flink 为代表的新一代流计算框架，在理论和实践上都已日趋完善和成熟。<strong data-nodeid="12643">当掌握了流计算中的核心概念后，不妨一开始就站在 Flink 这个巨人的肩膀上，开始在流计算领域的探索和实践</strong>。而作为有希望统一流计算领域的 Apache Beam，实际上是构建在各种具体流计算框架上的更高一层统一编程模式，它对流计算中的各种概念和问题做出了总结，是我们追踪流计算领域最新进展的一个好切入点。</p>
<p data-nodeid="12320">到这里为止，我们有关流计算的各种核心概念和对几种开源流计算框架的讲解就完成了。在后面的课程中，我将用 Flink 演示两个应用案例，来帮助你理解如何将流计算技术运用到具体的业务中。</p>
<p data-nodeid="12321">最后留一个小作业，你知道 Flink 的分布式快照是怎么回事吗？它是如何帮助 Flink 实现 exactly-once 语义的呢？可以将你的想法或问题写在留言区。</p>
<p data-nodeid="12322">下面是本课时的脑图，以帮助你理解。</p>
<p data-nodeid="42630" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/27/AC/Cgp9HWBdgSCAbnm5ABGgDbOk8_w551.png" alt="Drawing 5.png" data-nodeid="42633"></p>

---

### 精选评论


