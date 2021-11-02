<p data-nodeid="850" class="">在本模块前面的课时中，我们已经学习了 Spark Streaming 的架构、原理、用法以及生产环境中需要考虑的问题。对于 Spark Streaming 的学习，我们已经告一段落了。在学习 Spark 最新的流处理套件 Structured Streaming 之前，你有必要来看看一种新的计算模型或者范式：Dataflow，它也是 Structured Streaming、Flink、Apex 等最新技术的理论基础，从这种新的计算模型中，我们能发现不少有趣且非常重要的内容。</p>


<p data-nodeid="3">本课时的主要内容有：</p>
<ul data-nodeid="4">
<li data-nodeid="5">
<p data-nodeid="6">Google MillWheel 系统</p>
</li>
<li data-nodeid="7">
<p data-nodeid="8">Dataflow 模型</p>
</li>
</ul>
<h3 data-nodeid="9">Google MillWheel 系统</h3>
<p data-nodeid="1414" class="">Google MillWheel（水磨轮转）系统来源于谷歌公司在 2013 年发表的一篇论文：“MillWheel: Fault-Tolerant Stream Processing atInternet Scale”，它致力于构建一种低延迟、大规模的流处理系统，<strong data-nodeid="1420">用户只需定义计算拓扑和应用代码，系统会自动管理持久化状态以及连续的数据流，所有的这一切都在框架的容错保证之下</strong>。换句话说，Google MillWheel 系统是一种高吞吐、低延迟、数据不重不丢且具有容错性保证的分布式流处理框架，并且还提供了晚到和乱序数据的解决方案，可以说是下一代流处理系统的雏形。Spark 2.2 正式发布的 Structured Streaming 和 Flink 中都可以看到 Google MillWheel 的影子。Google MillWheel 的最大特色是对乱序数据与晚到数据的处理方法。</p>

<p data-nodeid="11">乱序和晚到数据处理方式的提出，体现了业界对于实时数据处理结果正确性不断提升的要求。在解释晚到数据之前，需要先了解两个与时间相关的概念。</p>
<ul data-nodeid="12">
<li data-nodeid="13">
<p data-nodeid="14"><strong data-nodeid="113">事件时间</strong>（event time），事件时间指的是事件发生的时间，在消息诞生时就被系统记录在消息中。</p>
</li>
<li data-nodeid="15">
<p data-nodeid="16"><strong data-nodeid="118">处理时间</strong>（processing time），处理时间指的是在数据管道中处理数据时，该消息被数据处理系统观察到的时间，是数据处理系统的时间，这里并没有假设分布式系统中时钟是同步的。</p>
</li>
</ul>
<p data-nodeid="2521">在现实情况中，由于网络存在延迟、处理本身需要时间，以及数据管道内部的性能消耗等原因，会导致同一条数据的这两个时间存在差异，如下图所示：</p>
<p data-nodeid="2522" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/3D/CgqCHl7xsSeANW58AAAiN_q9GEU479.png" alt="Drawing 0.png" data-nodeid="2526"></p>


<p data-nodeid="19">斜线表示的是理想情况，处理时间与事件时间完全相等，曲线表示的是实际情况，通常我们将其称之为时间域倾斜。曲线所代表的数据就是晚到数据和乱序数据，晚到数据和乱序数据一定是用户希望按照数据的事件时间顺序来处理数据才有的概念，如果只是按照处理时间来处理，晚到和乱序就无从说起了。</p>
<p data-nodeid="3908">基于此，Google MillWheel 提出了一种低水位（low watermark）机制，作为一种解决方案。我们先来看看低水位的定义，如下图所示。</p>
<p data-nodeid="3909" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/3D/CgqCHl7xsTSAb3ZpAACGLqdJa50524.png" alt="s1.png" data-nodeid="3913"></p>

<p data-nodeid="3632">A 和 C 是流处理计算拓扑中的两个计算单元（可以简单理解为 Spark DAG 中的两个 Executor ），C 是 A 的上游，会将数据源源不断地发送给 A，A 会维护一个时间戳，这个时间戳就是上文提到的低水位，它本质是一个边界，代表不会有晚于这个时间的数据发送给 A，如上图所示。换句话说，低水位后到达的数据很有可能已经丢失了，也没必要参与计算，流处理系统可以略过这些数据或是由应用自行处理，据谷歌表示，这部分丢掉的数据占整体数据的 0.001% 左右，考虑到晚到和乱序已经成为数据流的常态，系统也不可能无休止地等下去，这个误差还是可以接受的。</p>





<p data-nodeid="23">低水位不是一成不变的，就像真实情况中的水位一样，它会随着 A 处理过的数据的事件时间变化而变化。在 Google MillWheel 模型中，对低水位是这样定义的：</p>
<blockquote data-nodeid="24">
<p data-nodeid="25">对于一个 C→A 的拓扑片段：<br>
A 的低水位&nbsp;=&nbsp;min( A 接收到但还未被处理完毕的最老的数据的事件时间，C 的低水位)；<br>
如果没有输入流，则低水位的值与最大事件时间相等。</p>
</blockquote>
<p data-nodeid="5014">从下图中可以看到，横轴上方是待处理的数据，横轴下方是处理完毕的数据，低水位就是最后一条待处理数据事件后的时间戳，它会随着数据流向前推进。</p>
<p data-nodeid="5015" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/3D/CgqCHl7xsUCAfmWVAAArMHAmXTo698.png" alt="Drawing 2.png" data-nodeid="5019"></p>


<p data-nodeid="6684"><strong data-nodeid="6690">在谷歌 MillWheel 中，可以看到低水位本质上是 A 的一个可变状态。</strong> 得到了低水位的值以后，就可以根据该值来判断是否触发计算，以窗口计算为例，如下图所示。</p>
<p data-nodeid="6685" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/32/Ciqc1F7xsUqAShsvAAAe4vDNCsg590.png" alt="Drawing 3.png" data-nodeid="6693"></p>



<p data-nodeid="30">窗口是天然存在的，时长为 5 min，窗口和窗口之间没有重叠。在 A 中接收到的数据根据其事件时间分布在这 4 个窗口中，随着水位不断上涨，当低水位超过第一个窗口的结束时间（12:05）时，根据低水位的定义，有理由相信属于该窗口的数据已经全部到达，这时就可以触发该窗口进行计算，并且在窗口中，可以根据事件时间进行顺序处理，而在低水位之后的数据，虽然事件时间属于第一个窗口（12:00-12:05），但不会触发任何计算，也就不会体现在结果中。低水位可以认为是谷歌公司提出的一个基准，它给出了一个可以容忍数据晚到的最大极限，在低水位之前到达的数据（乱序数据）会参与计算，低水位后到达的数据（晚到数据）将被丢弃。</p>
<h3 data-nodeid="31">Google Dataflow 模型</h3>
<p data-nodeid="32">谷歌公司的“MillWheel: Fault-Tolerant Stream Processing at Internet Scale”这篇论文着重介绍了 Google MillWheel 系统如何实现，讨论范围只限于流处理的范畴，可以看成是一篇解决某个具体问题的论文。又过了两年，几乎是发表 MillWheel 的原班人马又发表了一篇论文</p>
<p data-nodeid="33">“The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing”，这篇论文与 MillWheel 不同，抽象程度非常高，提出了 Dataflow 模型，在一个很高的层次统一了流处理和批处理的计算模型，把这两类问题变成了一个简单的选择题。<strong data-nodeid="153">Structured Streaming 和 Flink 在设计上很大程度借鉴了 Dataflow 模型的设计思想，从这个层次上来说，这两种技术没有什么不同。</strong></p>
<p data-nodeid="10095" class="">作为和数据打交道的工程师，我们不能把无边界数据集切分成有边界数据集，等待一个批次完整后再做处理。<strong data-nodeid="10104">相反地，我们应该假设自己永远无法知道数据流是否终结，是否有序，数据何时会变完整。</strong> 唯一应该确信的是，新的数据会源源不断，老的数据可能会被撤销或更新，能够让我们应对<strong data-nodeid="10105">这个挑战唯一可行的方法是通过一个通用抽象模型，在数据处理的结果准确性、延迟程度和处理成本（这里的处理成本指的是每条数据的处理成本）之间进行取舍。这也是 Dataflow 模型的最大贡献。</strong></p>






<p data-nodeid="35">这篇论文提出，对于无边界、乱序的数据，可以按照数据本身的特性、事件时间的顺序计算结果；通过以下 4 个维度对数据流进行解构，使用户可以透明地、灵活地组合它们：</p>
<ul data-nodeid="36">
<li data-nodeid="37">
<p data-nodeid="38">计算什么（what）；</p>
</li>
<li data-nodeid="39">
<p data-nodeid="40">根据事件时间，哪些数据会参与计算（where）；</p>
</li>
<li data-nodeid="41">
<p data-nodeid="42">什么时候触发计算（when）；</p>
</li>
<li data-nodeid="43">
<p data-nodeid="44">早期的计算结果如何被修正（how）。</p>
</li>
</ul>
<p data-nodeid="45">此外，计算逻辑不再和处理的数据类型相关，也就是说，无论处理什么数据，用户只需要编写一套代码。具体来说，Dataflow 包含了：</p>
<ul data-nodeid="46">
<li data-nodeid="47">
<p data-nodeid="48">窗口模型，可以支持非对齐窗口，提供创建并使用基于事件时间窗口的一整套 API，<strong data-nodeid="180">对于 Spark Streaming 有状态和无状态的概念都可以用窗口轻松地进行表达</strong>；</p>
</li>
<li data-nodeid="49">
<p data-nodeid="50">触发器模型，可以根据数据流的特征来决定何时输出计算结果的模型，并且提供了一组强有力，且足够灵活的 API 来描述触发语义，比如由低水位触发就是一种触发语义；</p>
</li>
<li data-nodeid="51">
<p data-nodeid="52">增量计算模型，能够将数据变化体现到上述的窗口模型和触发器模型中；</p>
</li>
<li data-nodeid="53">
<p data-nodeid="54">可扩展实现，基于 MillWheel 和 FlumeJava 的可扩展实现；</p>
</li>
<li data-nodeid="55">
<p data-nodeid="56">一系列核心原则，指导 Dataflow 设计的核心原则。</p>
</li>
</ul>
<p data-nodeid="57"><strong data-nodeid="188">这些元素分别解答了上面四个问题：what、where、when、how。</strong></p>
<p data-nodeid="11190">从 Google MillWheel 系统来看，它对无边界的乱序和晚到数据处理提出了一种解决方法，下面通过几个例子来看看如何将这种解决方法泛化，并推广到不同场景中去。假设输入如下图所示。</p>
<p data-nodeid="11191" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/32/Ciqc1F7xsYuAWXMGAAE8Sw7XWkY641.png" alt="Drawing 4.png" data-nodeid="11195"></p>


<p data-nodeid="60">其中，竖轴是处理时间，也就是系统观察到数据的时间，横轴是事件时间，每条数据的值如圆圈数字所示，曲折的曲线是实际的低水位，而直线虚线是理想情况的低水位，可以看到值为 9 的数据落后于水位线，其他数据也存在不同程度的乱序。</p>
<p data-nodeid="12276">下图是我们按照传统批处理来构建求和需求的数据管道。在传统的批处理中，并没有水位线的概念，但是在 Dataflow 的语义中，批处理也引入了水位线的概念。可以看到，在所有数据到来之前，水位线一直不动，直到系统收集到了所有数据，计算发生（下图长方形上沿），水位线开始以平行于事件时间的方向迅速移动，直到无穷远，得到结果 51。这也可以理解为，流处理系统等待所有数据到来后，再开始处理，这样水位线变化和批处理是完全一样的。</p>
<p data-nodeid="19944"><img src="https://s0.lgstatic.com/i/image/M00/26/3E/CgqCHl7xsZeAMGaTAAC4CwyACR8715.png" alt="Drawing 5.png" data-nodeid="19947"></p>

<p data-nodeid="14214">再回到无边界数据中，如果我们采用一个全局窗口，以 MillWheel 那样的方式触发，那么用户永远也等不到结果出现，因为窗口会不断变大，所以必须采用一种新的触发方式，抑或采用别的方式进行开窗操作。</p>








<p data-nodeid="15305">如下图所示，采取的是一种基于处理时间定期触发的方式，不断修正之前计算的结果，从触发计算的结果可以看到，每次计算包含原来窗口的计算结果并进行累计求和，这样延迟为 1 分钟，但是用户只能在最后一分钟才能得到完全正确的结果。</p>
<p data-nodeid="19406"><img src="https://s0.lgstatic.com/i/image/M00/26/32/Ciqc1F7xsauANRPmAACNfLEn1Ng858.png" alt="Drawing 6.png" data-nodeid="19409"></p>




<p data-nodeid="16395">下面我们基于事件时间开窗，还是采用批处理的方式，如下图所示：</p>
<p data-nodeid="18858"><img src="https://s0.lgstatic.com/i/image/M00/26/3E/CgqCHl7xsbGAD2ukAADamuClAPM514.png" alt="Drawing 7.png" data-nodeid="18862"></p>
<p data-nodeid="18859">同样，传统的批处理引擎会等待所有数据到来后，再根据事件时间处理，同样也是在水位线到达窗口后触发计算产生结果。现在再来考虑下在基于事件时间的固定窗口下进行微批处理，以一分钟为一个批次。系统每分钟会对窗口中的数据进行处理，而没有像上一个例子中，等待所有数据到来后再处理。每个批次开始时，水位线会从批次开始的时间迅速上升到批次结束的时间，如下图所示。这样每个批次完成后，系统会达到一个新的水位线。我们可以看到在 12:08 的时候，微批处理方式下，3 个窗口已经分别有结果输出，而反观批处理的方式，还没有触发计算，<strong data-nodeid="18867">微批处理选择了低延迟和结果的最终准确性，而批处理则选择了最高的延迟和最好的准确性。</strong></p>





<p data-nodeid="18583"><img src="https://s0.lgstatic.com/i/image/M00/26/32/Ciqc1F7xsbuARoo7AAEHd4SirnA699.png" alt="Drawing 8.png" data-nodeid="18586"></p>




<p data-nodeid="21000">现在我们运用像 MillWheel 这样的流处理引擎来基于事件时间的固定窗口进行实验，如下图所示，该实验类似于 MillWheel 的执行机制。</p>
<p data-nodeid="22062"><img src="https://s0.lgstatic.com/i/image/M00/26/33/Ciqc1F7xsc6AVnbYAADOhp9ZrOI616.png" alt="Drawing 9.png" data-nodeid="22065"></p>




<p data-nodeid="22329" class="">可以看到，当水位线一旦越过固定窗口的结束时间，就会触发计算，但是与 MillWheel 不同的是，虽然值为 9 的数据落后于水位线，但是在这里仍然触发了窗口的计算，<strong data-nodeid="22338">这也是 Dataflow 的设计原则之一：永远不要依赖任何数据完整性标记。</strong> 从上图中可以看到，只有第一个窗口被触发了两次，其余窗口都只被触发了一次，这种计算方式需要等待水位线漫过窗口才会触发，因此整体上的延迟可能比微批处理系统还要差，<strong data-nodeid="22339">这就是单纯依赖水位线可能引起的问题：水位线可能太慢。</strong></p>

<p data-nodeid="23384">那么很自然的，如果我们想降低整体延迟的话，可以考虑将微批的定期触发与 MillWheel 的水位线触发结合起来：系统会周期性地触发，并且在水位线漫过窗口时也会触发。这样整个系统的平均延迟会比微批处理系统更低，因为数据一旦到达就可能会被处理，周期性地触发也会不断进行，它是系统延迟的下限。<strong data-nodeid="23390">如下图所示，这种混合的方式在结果准确性、延迟程度和处理成本之间做出了一个适合大部分需求的取舍。</strong></p>
<p data-nodeid="23385" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/33/Ciqc1F7xseCAOLuvAADwTwRn19U609.png" alt="Drawing 10.png" data-nodeid="23393"></p>



<p data-nodeid="24963" class="">Google Dataflow 精彩的地方在于，<strong data-nodeid="24969">无论我们面对的数据类型和处理方式是什么，最后都转化为对结果准确性、延迟程度和处理成本之间的取舍。</strong> 取舍也意味着三者不可兼得，像这种“不可能三角”，在实际情况中比较普遍，例如分布式系统的 CAP 原则，宏观经济学中的蒙代尔三角，或许这就是自然界中的普遍规律。但是“不可能三角”并不意味着必须取二舍一，更多情况下是在偏重两点的情况下对三者进行权衡，比如上图中混合触发的方式并没有完全放弃结果准确性、延迟程度和处理成本中的任一点。</p>



<p data-nodeid="82">在论文中，Dataflow 也介绍了其简洁而表现力丰富的 API，以上图为例，代码如下：</p>
<pre class="lang-js" data-nodeid="37844"><code data-language="js">PCollection&lt;KV&lt;<span class="hljs-built_in">String</span>,&nbsp;Integer&gt;&gt;&nbsp;output&nbsp;=&nbsp;input
.apply(Window.into(FixedWindows.of(<span class="hljs-number">2</span>,&nbsp;MINUTES))
.trigger(SequenceOf(
RepeatUntil(
AtPeriod(<span class="hljs-number">1</span>,&nbsp;MINUTE),
AtWatermark()),
Repeat(AtWatermark())))
.accumulating())
.apply(Sum.integersPerKey());
</code></pre>

























<p data-nodeid="38359">在这段声明式代码中，可以看到我们定义了窗口的类型（固定窗口）、长度（2 min）和触发的机制（每分钟触发一次；水位线触发；迟到数据触发），另外 accumulating 方法控制了触发的模式，触发模式有 3 种，即累加（Accumulating）、丢弃（Discarding）、累加和撤回（Accumulating &amp; Retracting）。Accumulating 表示窗口一旦触发后，窗口中的数据会被保留，该窗口下一次的触发结果在上一次结果的基础上更新。</p>
<p data-nodeid="38360">你还可以选择 Discarding，该选项表示窗口触发后，窗口的数据会被丢弃，这样窗口每次计算的结果是互相独立的。此外，还可以选择 Accumulating &amp; Retracting，该选项表示窗口的下一次触发会撤回上一次计算的结果重新进行计算，Sum.integersPerKey() 定义了窗口聚合的逻辑。用这种简单的声明方式，我们可以轻易地定义上图中触发计算的逻辑。这套 API 也很好地实现了 <strong data-nodeid="38370">what、where、when、how 这四个问题的答案。</strong></p>

<p data-nodeid="85">我们来看看 Dataflow 在业务场景的应用，如在支付场景中，它们采用的是 Accumulating &amp; Retracting 触发模式加定时触发；还有一些统计场景，在这种场景下，我们希望能够在一个可以接受的时间范围内得到一个相对完整的结果，所以采用了水位线触发，如图 E 所示；而在推荐场景，结果的及时性比基于完备数据的结果有意义得多，所以在这种场景我们采用了处理时间定时触发，如图 D 所示；最后一个是异常检测，异常检测比较适合由数据驱动来进行触发计算，因为一旦异常发生，系统应该马上做出回应，同时这个场景也使用了组合触发器。</p>
<h3 data-nodeid="86">小结</h3>
<p data-nodeid="87">本课时主要介绍了 Dataflow 模型，<strong data-nodeid="278">而本课时的内容可以算是整个课程中最重要的理论</strong>，目前最新的大数据技术 Spark 和 Flink，从设计理念上都是 Dataflow 的实现，从某种程度上来说，这两种技术也没什么不同，即使目前略有差异，最后也会殊途同归。<strong data-nodeid="279">请你一定要花时间将 Dataflow 这部分的内容吃透</strong>，那么再学习 Structured Streaming 和 Flink 就会显得异常的轻松。</p>
<p data-nodeid="88">Spark Streaming 是基于 RDD 与算子的组合进行编程，批处理也一样，那么流处理是否也有 DataFrame + SQL 组合，它们又是如何和本课时的内容结合呢，下个课时我将为你解答这些问题。</p>
<p data-nodeid="89">最后给你留一个思考题：</p>
<p data-nodeid="38885" class="te-preview-highlight">在上一课时，我们将端到端的过程拆分为：输入- 处理 - 输出，那么在处理阶段，Dataflow 是如何实现“恰好一次”的消息送达语义呢？</p>

---

### 精选评论

##### **松：
> 你还可以选择 Discarding，该选项表示窗口触发后，窗口的数据会被丢弃，这样窗口每次计算的结果是互相独立的。此外，还可以选择 Accumulating 这两种方式都可以保证“恰好一次“的语义吧😁

