<p data-nodeid="1" class="">作为 Spark Streaming 的技术继任者，Structured Streaming 出现得其实有点晚，但是得益于 Spark 庞大的用户群体和社区，Structured Streaming 仍然是实时处理领域一种非常有竞争力和前景的技术。</p>
<p data-nodeid="2">从前面的 Spark Streaming 可以看到，Spark Streaming 仍然采用 RDD 与算子的组合进行编程。这其实是 Spark 官方不推荐的，它的缺点也显而易见，而 Structured Streaming 的改进是由内而外的，<strong data-nodeid="129">它不仅参考了 MillWheel 系统与 Dataflow 模型，并且还引入 DataFrame 与 SQL 作为自己的编程接口。</strong></p>
<p data-nodeid="3">本课时的主要内容有：</p>
<ul data-nodeid="4">
<li data-nodeid="5">
<p data-nodeid="6">Structured Streaming 的关键抽象与架构</p>
</li>
<li data-nodeid="7">
<p data-nodeid="8">操作</p>
</li>
<li data-nodeid="9">
<p data-nodeid="10">输入与输出</p>
</li>
</ul>
<h3 data-nodeid="11">Structured Streaming 抽象与架构</h3>
<p data-nodeid="1669">前面提过，Spark Streaming 的本质是将数据流抽象成流，以微批的方式进行处理，而 Structured Streaming 则不同，它采用的是 Google Dataflow 的思想，将数据流抽象成无边界表（unbounded table），如下图所示：</p>
<p data-nodeid="1670" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/65/Ciqc1F714b2AWMr1AACvyq3aDHQ628.png" alt="image (42).png" data-nodeid="1678"></p>



<p data-nodeid="2795"><strong data-nodeid="2801">这其实是将流抽象成了批</strong>，在这种抽象中，实时的数据流会不停追加到下图这张表中。我们通过对输入表查询的方式得到流处理的结果，并生成结果表。在每个触发间隔，新的数据将追加到输入表中，最终触发计算将会更新结果表，如下图所示：</p>
<p data-nodeid="2796" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/71/CgqCHl714caAWsdCAACXtBrLfeA468.png" alt="image (43).png" data-nodeid="2808"></p>



<p data-nodeid="18">第二行的方块表示结果表（Result Table），第三行的方块表示输出的外部存储，结果表更新到外部存储的方式，也就是输出模式，有 3 种，分别是完全模式、追加模式与更新模式。</p>
<ul data-nodeid="19">
<li data-nodeid="20">
<p data-nodeid="21">完全模式：整个更新的结果表将被写入外部存储，由存储连接器来决定如何写入整张表。上图即为完全模式下的输出结果，一般用来调试。</p>
</li>
<li data-nodeid="22">
<p data-nodeid="23">追加模式：只将上次触发后追加到结果表的数据写到外部存储。这种模式适用于不希望修改已经存在于结果表的数据场景。这是默认的输出模式。在这种模式下，窗口内定时触发生成的中间结果会保存到中间状态，最后一次触发生成的结果才会写到结果表。</p>
</li>
<li data-nodeid="24">
<p data-nodeid="25">更新模式：只将上次触发后更新到结果表的数据写到外部存储。这与完全模式不同，它只输出上次触发后变更的结果记录。如果窗口中不包含聚合逻辑，则意味着不需要修改以前结果表中的数据，其实与追加模式无异。</p>
</li>
</ul>
<p data-nodeid="26"><strong data-nodeid="156">这些其实就是对应的Dataflow的触发模式。</strong></p>
<p data-nodeid="27">作为 MillWheel 和 Dataflow 的实现者，实现端到端恰好一次的消息送达保证是必不可少的，为了实现这一目标，Structured Streaming 引入了 Source、Sink 和 StreamExecution 等组件，可以准确地追踪处理进度，以便从重启或重新处理中处理任何类型的故障。每个流式数据源（Source）都具有偏移量（类似于 Kafka 的偏移量）追踪读取进度，以便在故障发生后进行数据回放。在每次触发计算时，执行引擎（StreamExecution）使用检查点和预写日志机制记录来记录数据源偏移量。数据输出（Sink）允许自定义，如设计成幂等效果或者采取两段式提交。结合可重放的数据源与可靠的数据输出，Structured Streaming 可以在任何时候实现端到端的恰好一次消息送达保证。</p>
<p data-nodeid="28">Structured Streaming 不会保存整个无边界表的数据。它会从数据源读取数据，以增量处理的方式来更新结果，然后丢弃这些源数据，只会保留更新结果所需的最小化中间状态数据（如中间计数）。Structured Streaming 支持基于事件时间处理数据，对于晚到和乱序数据的处理，也同样引入了水位机制，后面会详细介绍。</p>
<p data-nodeid="3933">Structured Streaming 也遵循 Driver 和 Executor 的主从架构，其功能与前面介绍的无异：Driver 负责调度，Executor 负责执行，如下图所示。其中 Driver 和前面介绍的 Driver 差别不大，SQLExecution 变成了 StreamExecution，Executor 多了 StateStore，但是取消了和 Receiver 相关的组件，这是一个比较大的变化，代表接收数据的方式变了。下面我将就这两个组件展开讲解。</p>
<p data-nodeid="3934" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/66/Ciqc1F714dSAQf1tAAGAQh0SRqQ386.png" alt="x1.png" data-nodeid="3938"></p>



<h4 data-nodeid="32">1、 StreamExecution</h4>
<p data-nodeid="33">StreamExecution 是 Structured Streaming 的执行引擎，是 StreamingQuery 接口的实现，在用户的代码中被初始化，start() 方法是这个过程的入口，如下面的代码所示：</p>
<pre class="lang-scala" data-nodeid="34"><code data-language="scala">...
<span class="hljs-keyword">val</span>&nbsp;lines&nbsp;=&nbsp;spark
.readStream
.format(<span class="hljs-string">"socket"</span>)
.option(<span class="hljs-string">"host"</span>,&nbsp;<span class="hljs-string">"localhost"</span>)
.option(<span class="hljs-string">"port"</span>,&nbsp;<span class="hljs-number">9999</span>)
.load()
...
<span class="hljs-keyword">val</span>&nbsp;query:&nbsp;&nbsp;<span class="hljs-type">QueryExecution</span>&nbsp;=&nbsp;wordCounts.map(…).writeStream
.trigger(<span class="hljs-type">ProcessingTime</span>(<span class="hljs-number">2.</span>seconds))
.outputMode(<span class="hljs-string">"complete"</span>)
.format(<span class="hljs-string">"console"</span>)
.start()
query.awaitTermination()
</code></pre>
<p data-nodeid="35">StreamExecution 有几个比较重要的成员变量：Trigger、LogicalPlan、Sink 和 Source。其中 Trigger 表示用户设定的触发间隔，它决定了一次处理的数据大小，LogicalPlan 代表执行计划，从用户代码的 start 方法开始，就进入了 StreamExecution。首先，可以看见这里设置触发器为每 2 分钟（处理时间）处理一次，用户从 Source 中取得相应的数据；接着根据用户定义的运算逻辑和 Trigger 得到优化过后的 IncrementalExecution，最后再交给 Sink 的 addBatch 方法触发整个过程执行；执行完成后，会通知 Source 修改相应的偏移量。<strong data-nodeid="171">其中 IncrementalExecution 是 Structured Streaming 独有的，它体现了 Structured Streaming 的执行方式是增量微批，主要针对前面 3 种输出模式进行优化。</strong> Source、Sink 和 StreamExecution 抽象了整个流处理过程。从上面的这个过程可以看出，StreamExecution 存在的目的还是要将流式数据转化为 DataFrame 的执行计划并执行，其实最后还是由 SQLExecution 负责执行，也就是与批处理同样的执行引擎。这样的好处就在于无缝对接了 DataFrame 与 Tungsten 巨大的性能优化，并且统一了流处理和批处理的计算引擎。</p>
<p data-nodeid="36">再来看看 query 初始化的代码，与上一课时中 Dataflow 的样例代码非常相似，在这段代码中我们定义了 Trigger、输出模式、处理逻辑。在处理逻辑中，还可以定义窗口和水位，完全是一种 Dataflow 思想的体现。其中，Trigger 组件的选项有以下几个。</p>
<ul data-nodeid="37">
<li data-nodeid="38">
<p data-nodeid="39">未指定（默认）。如果没有指定触发器，默认的触发逻辑是微批处理，一旦前一个微批完成处理，将立即生成下一个微批进行处理。</p>
</li>
<li data-nodeid="40">
<p data-nodeid="41">固定间隔的微批触发器。将以固定间隔进行触发，固定间隔长度由用户设置（默认的触发器实际上固定间隔长度为 0，即不固定）。如果先前的微批在当前间隔完成，则等到该间隔结束再开始下一个微批处理；如果前一个微批处理需要的时间超过了间隔长度，那么下一个微批将在前一个微批完成后立即执行，而不是等待下一个间隔边界；如果没有数据，则不会触发新的微批处理。固定间隔的微批触发器设定如下：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="42"><code data-language="scala">.trigger(<span class="hljs-type">Trigger</span>.<span class="hljs-type">ProcessingTime</span>(<span class="hljs-number">2000</span>))
</code></pre>
<ul data-nodeid="43">
<li data-nodeid="44">
<p data-nodeid="45">一次性微批触发器。查询只会触发一次针对所有可用数据的微批处理，然后自行停止。**这在希望定期启动集群来处理上一个时间段里累积的所有数据，<b><strong data-nodeid="188">然后</strong></b>停止集群的场景里非常有用。**在某些情况下，可以显著减少性能开销。</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="46"><code data-language="scala">.trigger(<span class="hljs-type">Trigger</span>.<span class="hljs-type">Once</span>())
</code></pre>
<ul data-nodeid="47">
<li data-nodeid="48">
<p data-nodeid="49">固定检查点间隔的连续触发器。固定检查点间隔的连续触发器是 Spark 新加入的流执行模式——连续处理的体现。连续处理是 Spark 2.3 中引入的实验性质的流执行模式，可以实现约 1ms 的端到端延迟，并且实现了“至少一次”消息送达保证，而默认的微批处理引擎虽然实现了“恰好一次”的消息送达保证，但是延迟最少也要 100ms 左右。对于某些类型的查询，我们可以通过只修改触发器而不修改应用逻辑来启用该特性。如下：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="50"><code data-language="scala">.trigger(<span class="hljs-type">Trigger</span>.<span class="hljs-type">Continuous</span>(<span class="hljs-string">"1&nbsp;second"</span>))&nbsp;<span class="hljs-comment">//&nbsp;只需修改这一行</span>
</code></pre>
<p data-nodeid="51">参数 1 秒指的是检查点间隔，意味着执行引擎每秒会记录一次执行进度，得到的检查点文件将采用和微批执行引擎兼容的格式，所以我们可以在不同触发器之间任意切换，例如以微批触发模式启动的查询，可以以连续触发模式重新启动。无论何时切换，用户都可以获得“至少一次”的消息传递保证。目前连续触发器只支持特定的操作、数据源和输出。</p>
<p data-nodeid="52">从触发器的多样性来看，Dataflow 中的触发器功能无疑更加强大，既提供了固定周期触发器（AtPeriod），也可以基于水位（AtWatermark）等，但按照 Structured Streaming 的发展速度，相信很快能够实现更多类型的触发器。</p>
<h4 data-nodeid="53">2、 StateStore</h4>
<p data-nodeid="54">StateStore 的作用是作为数据流转的状态存储（持久化）。它本质是一个分布式、高可用、分版本的键值存储，提供 get、put、remove 等增删改查操作。它的分片逻辑是算子编号（operatorId）加分区编号（partitionId）。</p>
<h3 data-nodeid="55">操作</h3>
<p data-nodeid="56">既然使用了 DataFrame 与 SQL，因此从使用上来说，无论是普通的转换 API 还是 Spark SQL，都是完全一样的，甚至一些封装过的批处理的代码都可以直接复用。那么按照 Dataflow的理论，这就是 what 部分，我们在本节还要解决：</p>
<ul data-nodeid="57">
<li data-nodeid="58">
<p data-nodeid="59">根据事件时间，哪些数据会参与计算（where）；</p>
</li>
<li data-nodeid="60">
<p data-nodeid="61">什么时候触发计算（when）；</p>
</li>
<li data-nodeid="62">
<p data-nodeid="63">早期的计算结果如何被修正（how）。</p>
</li>
</ul>
<p data-nodeid="64">前面其实已经介绍过相应的内容，<strong data-nodeid="203">where 由窗口来指定，when 由 Trigger 组件来指定，how 由 watermark 机制来解决。</strong></p>
<p data-nodeid="65">下面这个例子包含了 what、where、when、how 的声明：</p>
<pre class="lang-scala" data-nodeid="66"><code data-language="scala">...
<span class="hljs-keyword">val</span>&nbsp;windowDuration&nbsp;=&nbsp;<span class="hljs-string">"3000&nbsp;seconds"</span>
<span class="hljs-keyword">val</span>&nbsp;slideDuration&nbsp;=&nbsp;<span class="hljs-string">"1000&nbsp;seconds"</span>
&nbsp;
<span class="hljs-keyword">val</span>&nbsp;lines&nbsp;=&nbsp;spark.readStream
.format(<span class="hljs-string">"socket"</span>)
.option(<span class="hljs-string">"host"</span>,&nbsp;<span class="hljs-string">"localhost"</span>)
.option(<span class="hljs-string">"port"</span>,&nbsp;<span class="hljs-number">9999</span>)
.option(<span class="hljs-string">"includeTimestamp"</span>,&nbsp;<span class="hljs-literal">true</span>)
.load()

&nbsp;<span class="hljs-keyword">val</span>&nbsp;words&nbsp;=&nbsp;lines.as[(<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">TimestampType</span>)].flatMap(line&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;line._1.split(<span class="hljs-string">"&nbsp;"</span>).map(word&nbsp;=&gt;&nbsp;(word,&nbsp;line._2))
&nbsp;).toDF(<span class="hljs-string">"word"</span>,&nbsp;<span class="hljs-string">"timestamp"</span>)
&nbsp;
<span class="hljs-keyword">val</span>&nbsp;windowedCounts&nbsp;=&nbsp;words.groupBy(
window($<span class="hljs-string">"timestamp"</span>,&nbsp;windowDuration,&nbsp;slideDuration),&nbsp;
$<span class="hljs-string">"word"</span>)
.count().orderBy(<span class="hljs-string">"window"</span>)
...
</code></pre>
<p data-nodeid="5057">上面这段代码的计算逻辑如下图所示：</p>
<p data-nodeid="5058" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/71/CgqCHl714eeAar29AAFaO4mDKsc740.png" alt="image (44).png" data-nodeid="5066"></p>



<p data-nodeid="6191">上图中，下方的结果表显示了 5 分钟滑动步长以及 10 分钟大小的窗口聚合结果。窗口跨度也是结果表的一个字段，该字段名为 window，从图中虚线箭头可以看出，触发器（Trigger）采用的是固定间隔触发，每 5 分钟触发一次。聚合的时间窗口是以数据的事件时间为准的，那么这样一定会存在晚到和乱序数据的问题。与 Google Dataflow 模型类似，Structured Streaming 也基于水位机制对晚到和乱序数据提供了相应处理机制，如下图所示：</p>
<p data-nodeid="6192" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/71/CgqCHl714fCACLm3AAFcA6HjgFA959.png" alt="image (45).png" data-nodeid="6200"></p>



<p data-nodeid="7333">当 12:04 的晚到数据在 12:11 到达时，它实际应该落在 12:00-12:10 的窗口里。在下一次触发时间（12:15）到来时，会更新结果表中的结果。Structured Streaming 会将这种类似聚合需求长时间维护在一个中间状态以便处理晚到数据，但是，<strong data-nodeid="7340">如果对于长期运行的应用，对晚到数据无限期等待的代价或许会很大，这时系统应该有一种方法知道什么时候应该丢弃那些过时的中间结果，这样应用也不用再去处理这种晚到数据了。</strong> 和 Google Dataflow 一样，Structured Streaming 也采用了水位机制，旨在让计算引擎追踪数据中当前的事件时间，并尝试清除过时的中间状态。在 Structured Streaming 中，水位定义为当前最大的事件时间减去晚到时间最大容忍值。对一个 T 时刻的特定窗口，计算单元会维护其结果状态，当计算单元接收到的晚到数据，满足观察到的最大事件时间减去晚到时间最大容忍值，大于 T 的条件时，都允许晚到数据修改结果状态。换句话说，在水位线前面的数据会被处理，后面的数据则会被丢弃，如下图所示。</p>
<p data-nodeid="7334" class=""><img src="https://s0.lgstatic.com/i/image/M00/27/66/Ciqc1F714fiAW20gAAK8f9_2AvA958.png" alt="image (46).png" data-nodeid="7347"></p>



<p data-nodeid="75">坐标轴中的虚线表示了至今为止计算单元观察到的最大事件时间；浅色实心圆点代表正常处理的数据；深色实心圆点代表了晚到数据，但是在可以容忍的晚到范围之内，而空心圆点数据表示晚到且在不能容忍的时间范围之内。判断是否可以容忍的依据是阶梯形实线（当前最大事件时间减去最大容忍晚到时间），也就是上面说的水位，在每个触发间隔前，会重新计算水位。晚到数据以这种机制被计算、被丢弃，最后更新结果表。</p>
<p data-nodeid="76">上图是更新输出模式，所以在每个触发点都会输出结果表，晚到数据会在后面的触发点再更新到结果表。</p>
<p data-nodeid="77">开启 Structured Streaming 的水位机制很简单，只需在代码中加上 withWatermark：</p>
<pre class="lang-scala" data-nodeid="78"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;windowedCounts&nbsp;=&nbsp;words
<span class="hljs-comment">//&nbsp;最大容忍时间是5分钟，指定水位基于的列名timestamp</span>
.withWatermark(<span class="hljs-string">"timestamp"</span>,&nbsp;<span class="hljs-string">"5&nbsp;minmutes"</span>)
.groupBy(
window(
$<span class="hljs-string">"timestamp"</span>,&nbsp;
windowDuration,&nbsp;
slideDuration),&nbsp;
$<span class="hljs-string">"word"</span>)
.count()
</code></pre>
<p data-nodeid="79">想要使用水位机制需要满足一定条件，具体如下。</p>
<ul data-nodeid="8488">
<li data-nodeid="8489">
<p data-nodeid="8490">输出模式必须是更新模式和追加模式。在这两种模式中，会导致结果输出有所差别，在更新模式中，晚到的数据会被处理后再修改结果表，如图f所示，但是在追加模式中，由于不能修改结果表的内容，因此只有在水位超过了窗口结束时间后的下一个触发点才会触发聚合操作，也就是说，在追加模式中，<strong data-nodeid="8503">是等所有晚到时间小于最大容忍时间的数据都接收到后，再进行聚合操作，并一次性输出到结果表</strong>，如下图所示，这两种模式的差别主要考虑的是接收器的差别，有些接收器不能进行修改，如文件系统。</p>
</li>
<li data-nodeid="8491">
<p data-nodeid="8492">聚合操作必须有事件时间列，或者有一个基于事件时间列的窗口。</p>
</li>
<li data-nodeid="8493">
<p data-nodeid="8494">水位机制与聚合操作中使用的时间列必须相同。</p>
</li>
<li data-nodeid="8495">
<p data-nodeid="8496">水位机制必须在聚合操作之前被调用。</p>
</li>
</ul>
<p data-nodeid="8497" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/27/71/CgqCHl714giAKuj4AAJeh4dq0zU144.png" alt="image (47).png" data-nodeid="8513"></p>



<h3 data-nodeid="91">输入与输出</h3>
<p data-nodeid="92">在 Structured Streaming 中，输入被统一抽象为 Source，与 Spark Streaming 中各种不同的输入源不同，输出也被统一抽象为 Sink，这在 Spark Streaming 并没有进行抽象，这么做的好处不言而喻，有了统一的输出抽象，要实现端到端级别的“恰好一次”消息送达语义的实现就不再那么困难。也就是说，Structured Streaming 将整个流处理过程抽象为“Source + StreamExecution + Sink”，对应我们前面讲的“输入-处理-输出”过程，这样 Structured Streaming 就有能力为用户提供端到端级别“恰好一次”消息送达语义的实现。</p>
<p data-nodeid="93">下面分别来看看 Source 和 Sink 组件。</p>
<h4 data-nodeid="94">1. Source</h4>
<p data-nodeid="95">Source 抽象了流式数据从接收到处理之间的过程，它是一个接口，其定义非常简单，如下面的代码所示：</p>
<pre class="lang-scala" data-nodeid="96"><code data-language="scala"><span class="hljs-class"><span class="hljs-keyword">trait</span><span class="hljs-title">&nbsp;Source&nbsp;</span></span>{
<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;schema</span></span>:&nbsp;<span class="hljs-type">StructType</span>
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;getOffset</span></span>:&nbsp;<span class="hljs-type">Option</span>[<span class="hljs-type">Offset</span>]
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;getBatch</span></span>(start:&nbsp;<span class="hljs-type">Option</span>[<span class="hljs-type">Offset</span>],&nbsp;end:&nbsp;<span class="hljs-type">Offset</span>):&nbsp;<span class="hljs-type">DataFrame</span>
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;commit</span></span>(end:&nbsp;<span class="hljs-type">Offset</span>)&nbsp;:&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{}
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;stop</span></span>():&nbsp;<span class="hljs-type">Unit</span>
}
</code></pre>
<p data-nodeid="97">从这个接口可以看出，无论是哪种数据源，都必须实现 getOffset 方法，这意味着在 Structured Streaming 中，所有数据源都可以向 Spark Streaming 中 Kafka Direct API 那样通过维护偏移量的方式进行容错（回放）。所有的数据源皆是如此，也就意味着，Source 完全抛弃了基于 Receiver 的数据接收方式，省事省心。<br>
目前 Source 的实现类有 KafkaSource 和 FileStreamSource，分别代表了 Kafka 数据源和 HDFS 数据源，还提供了用于测试的控制台数据源、Socket 数据源和 Rate 数据源（Rate 数据源能以指定速率生成数据，生成的每条数据都带有时间戳和消息 ID）的实现类。KafkaSource 很好理解，它本身的数据结构中就自带 offset，可以很好地和接口中 getOffset 相匹配，而 HDFS 的 offset 概念稍微复杂一点，每当调用 getOffset 方法时，HDFS 都会扫描文件夹下的文件，将最新的一些文件进行聚合成批，再赋予一个递增数字，FileStreamSource 对象中会维护一个名为 seenFiles 的 HashMap，用来保存已经扫描过的文件，判断依据是文件年龄和文件名。不管是哪种偏移量（除了 TextSocketSource），最后都会被 StreamExecution 预写日志 WAL 以便进行回放。这与 Spark Streaming 中介绍的思路是完全一样的，只不过 Structured Streaming 已经在这个环节中做到了对用户完全透明并泛化到所有数据源。</p>
<p data-nodeid="98">下面来看一个从 KafkaSource 中读取数据的例子，在这个例子中，会根据Kafka地址读取消息并将其转换为结构化数据：</p>
<pre class="lang-scala" data-nodeid="99"><code data-language="scala">......
<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span>
.builder
.master(<span class="hljs-string">"local[2]"</span>)
.appName(<span class="hljs-string">"StructuredNetworkWordCount"</span>)
.getOrCreate()
&nbsp;
<span class="hljs-keyword">import</span>&nbsp;spark.implicits._
&nbsp;
<span class="hljs-keyword">val</span>&nbsp;df&nbsp;=&nbsp;spark
.readStream
.format(<span class="hljs-string">"kafka"</span>)
.option(<span class="hljs-string">"kafka.bootstrap.servers"</span>,&nbsp;<span class="hljs-string">"host1:port1,host2:port2"</span>)
.option(<span class="hljs-string">"subscribe"</span>,&nbsp;<span class="hljs-string">"topicA"</span>)
.load()
&nbsp;
df.selectExpr(<span class="hljs-string">"CAST(key&nbsp;AS&nbsp;STRING)"</span>,&nbsp;<span class="hljs-string">"CAST(value&nbsp;AS&nbsp;STRING)"</span>).as[(<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">String</span>)]
.....
</code></pre>
<h4 data-nodeid="100">2. Sink</h4>
<p data-nodeid="101">Sink 是 Structured Streaming 对输出过程的抽象，目的也是实现对用户透明的容错处理。与Source 对应，Sink 也有 FileStreamSink、KafkaSink、ForeachSink 等，分别对应的输出为 HDFS、Kafka 和自定义 Sink，此外还提供了用于测试的控制台输出和内存输出的实现类。Sink 接口非常简单，如下面的代码所示：</p>
<pre class="lang-scala" data-nodeid="102"><code data-language="scala"><span class="hljs-class"><span class="hljs-keyword">trait</span><span class="hljs-title">&nbsp;Sink&nbsp;</span></span>{
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;addBatch</span></span>(batchId:&nbsp;<span class="hljs-type">Long</span>,&nbsp;data:&nbsp;<span class="hljs-type">DataFrame</span>):&nbsp;<span class="hljs-type">Unit</span>
}
</code></pre>
<p data-nodeid="103">其中只定义了一个方法 addBatch，整个计算过程都是由这个方法触发执行的，现在来深入了解下这个方法，以 KafkaSink 为例：</p>
<pre class="lang-scala" data-nodeid="104"><code data-language="scala"><span class="hljs-keyword">override</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;addBatch</span></span>(batchId:&nbsp;<span class="hljs-type">Long</span>,&nbsp;data:&nbsp;<span class="hljs-type">DataFrame</span>):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(batchId&nbsp;&lt;=&nbsp;latestBatchId)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logInfo(<span class="hljs-string">s"Skipping&nbsp;already&nbsp;committed&nbsp;batch&nbsp;<span class="hljs-subst">$batchId</span>"</span>)
&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">KafkaWriter</span>.write(sqlContext.sparkSession,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;data.queryExecution,&nbsp;executorKafkaParams,&nbsp;topic)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;latestBatchId&nbsp;=&nbsp;batchId
&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="105">在这个方法中，先判断了是否是重复提交的数据，然后由 KafkaWriter 的 write() 方法进行写入，写入完成后，更新最新的 batchId。接下来我们来看看 write() 方法：</p>
<pre class="lang-scala" data-nodeid="106"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;write</span></span>(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sparkSession:&nbsp;<span class="hljs-type">SparkSession</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;queryExecution:&nbsp;<span class="hljs-type">QueryExecution</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kafkaParameters:&nbsp;ju.<span class="hljs-type">Map</span>[<span class="hljs-type">String</span>,&nbsp;<span class="hljs-type">Object</span>],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;topic:&nbsp;<span class="hljs-type">Option</span>[<span class="hljs-type">String</span>]&nbsp;=&nbsp;<span class="hljs-type">None</span>):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;schema&nbsp;=&nbsp;queryExecution.analyzed.output
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;validateQuery(queryExecution,&nbsp;kafkaParameters,&nbsp;topic)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;queryExecution.toRdd.foreachPartition&nbsp;{&nbsp;iter&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;writeTask&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">KafkaWriteTask</span>(kafkaParameters,&nbsp;schema,&nbsp;topic)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">Utils</span>.tryWithSafeFinally(block&nbsp;=&nbsp;writeTask.execute(iter))(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;finallyBlock&nbsp;=&nbsp;writeTask.close())
}
</code></pre>
<p data-nodeid="107">在这个方法中，由 queryExecution 的 toRDD 方法触发计算开始，在最后 foreachPartition 操作中完成写入 Kafka 的操作。最后的写入操作是在 KafkaWriteTask 的 execute 方法完成的，如下面的代码所示：</p>
<pre class="lang-scala" data-nodeid="108"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;execute</span></span>(iterator:&nbsp;<span class="hljs-type">Iterator</span>[<span class="hljs-type">InternalRow</span>]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;producer&nbsp;=&nbsp;<span class="hljs-type">CachedKafkaProducer</span>.getOrCreate(producerConfiguration)
&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(iterator.hasNext&nbsp;&amp;&amp;&nbsp;failedWrite&nbsp;==&nbsp;<span class="hljs-literal">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;currentRow&nbsp;=&nbsp;iterator.next()
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;projectedRow&nbsp;=&nbsp;projection(currentRow)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;topic&nbsp;=&nbsp;projectedRow.getUTF8String(<span class="hljs-number">0</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;key&nbsp;=&nbsp;projectedRow.getBinary(<span class="hljs-number">1</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;value&nbsp;=&nbsp;projectedRow.getBinary(<span class="hljs-number">2</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(topic&nbsp;==&nbsp;<span class="hljs-literal">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span>&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">NullPointerException</span>(<span class="hljs-string">s"null&nbsp;topic&nbsp;present&nbsp;in&nbsp;the&nbsp;data.&nbsp;Use&nbsp;the&nbsp;"</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">s"<span class="hljs-subst">${KafkaSourceProvider.TOPIC_OPTION_KEY}</span>&nbsp;option&nbsp;for&nbsp;setting&nbsp;a&nbsp;default&nbsp;topic."</span>)
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;由上面构建好的topic、key和value生成最后待插入的消息</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;record&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">ProducerRecord</span>[<span class="hljs-type">Array</span>[<span class="hljs-type">Byte</span>],&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">Byte</span>]](topic.toString,&nbsp;key,&nbsp;value)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;callback&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Callback</span>()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">override</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;onCompletion</span></span>(recordMetadata:&nbsp;<span class="hljs-type">RecordMetadata</span>,&nbsp;e:&nbsp;<span class="hljs-type">Exception</span>):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(failedWrite&nbsp;==&nbsp;<span class="hljs-literal">null</span>&nbsp;&amp;&amp;&nbsp;e&nbsp;!=&nbsp;<span class="hljs-literal">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;failedWrite&nbsp;=&nbsp;e
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;producer.send(record,&nbsp;callback)
&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="109">插入完成后，按照前面课时的思路，还要更新 Source 的偏移量，这由 StreamExecution 中的 batchCommitLog.add(currentBatchId) 完成。整个过程与 Spark Streaming 中实现的过程大同小异，不同的是这个过程更加透明，用户不用直接维护偏移量，比起 Spark Streaming 的各种 API 调用，更加优雅。但是目前的版本，Spark Streaming 中的问题仍然存在，还是<strong data-nodeid="266">无法在事务层面保证处理-输出的这个环节做到完美的“恰好一次”，只能做到“至少一次”</strong>，需要通过使输出幂等来实现，或者使用 Structured Streaming 提供的去重操作：</p>
<pre class="lang-scala" data-nodeid="110"><code data-language="scala"><span class="hljs-comment">//&nbsp;参数为去重列名</span>
streamingDf.dropDuplicates(<span class="hljs-string">"uniquecolumn"</span>)
</code></pre>
<p data-nodeid="111">目前 FileStreamSink 提供了幂等保证，用户不用自己实现幂等逻辑，而 ForeachSink 则提供了自定义的 Writer，因此是否幂等取决于用户自己是否实现，目前 ForeachWriter 只有 Scala 和 Java 接口。ForeachWriter 代码如下：</p>
<pre class="lang-scala" data-nodeid="112"><code data-language="scala"><span class="hljs-keyword">abstract</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span><span class="hljs-title">&nbsp;ForeachWriter</span>[<span class="hljs-type">T</span>]<span class="hljs-title">&nbsp;extends&nbsp;Serializable&nbsp;</span></span>{
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;open</span></span>(partitionId:&nbsp;<span class="hljs-type">Long</span>,version:&nbsp;<span class="hljs-type">Long</span>):&nbsp;<span class="hljs-type">Boolean</span>
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;process</span></span>(value:&nbsp;<span class="hljs-type">T</span>):&nbsp;<span class="hljs-type">Unit</span>
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;close</span></span>(errorOrNull:&nbsp;<span class="hljs-type">Throwable</span>):&nbsp;<span class="hljs-type">Unit</span>
}
</code></pre>
<p data-nodeid="113">ForeachWriter 是一个抽象类，任何继承它的类必须重写 open、process 和 close 这 3 个方法，这 3 个方法会在 Executor 上依次被调用：</p>
<ul data-nodeid="114">
<li data-nodeid="115">
<p data-nodeid="116">open 方法的参数 partitionId 代表了输出分区ID；</p>
</li>
<li data-nodeid="117">
<p data-nodeid="118">version 是一个单调递增的 ID，随着每次触发而增加，我们可以通过两个参数判断这一批数据是否继续输出。如果返回值为 true，就调用 process 方法；如果返回值为 false，则不会调用 process 方法。</p>
</li>
</ul>
<p data-nodeid="119">每当调用 open 方法时，close 方法也会被调用（除非 JVM 因为某些错误而退出），因此我们可以在 open 方法中打开外部存储连接，并在 close 方法中关闭外部存储连接。</p>
<h3 data-nodeid="120">小结</h3>
<p data-nodeid="121">本课时介绍了 Structured Streaming 的相关内容，可以看到 Structured Streaming 忠实地复刻了 Dataflow 思路与实现，但是在声明计算逻辑环节复用了 Spark的DataFrame + Spark SQL，使得流处理编程变得十分简单，相信你也有相同的体会。另外，这样一来的话，<strong data-nodeid="278">Spark编程的批处理与流处理在形式上就得到了完美的统一</strong>，相信你在完成了本模块的实践课时后，体会会更深。</p>
<p data-nodeid="122">最后给你留一个思考题：</p>
<p data-nodeid="123">在流处理中，连接操作涉及两个数据源的操作，会相对比较复杂，其中每个数据源都有可能是静态的数据，或者是动态的数据流，那么 Structured Streaming 是如何处理连接操作的呢，它支持哪些类型的连接，又有什么限制呢？</p>

---

### 精选评论


