<p data-nodeid="1037" class="">在上一讲中，我们着重介绍了 Spark 的概念与架构，在这一讲，我将为你介绍一个新的计算框架——Flink，以及它与 Spark 有着怎样的联系与不同。</p>
<p data-nodeid="1038">在介绍 Flink 之前，我想先介绍两个概念：批处理与流处理。</p>
<h3 data-nodeid="1039">批处理与流处理</h3>
<p data-nodeid="1040"><strong data-nodeid="1115">批处理</strong></p>
<p data-nodeid="1041">所谓的批处理，从字面意思理解，就是把一整块数据切分成一小块一小块，每一个小块称为一批。把一个小块数据分配给一个计算节点进行运算，这种情况称为批处理。</p>
<p data-nodeid="1042">所以说，批处理针对的数据是一个<strong data-nodeid="1126">有限集合</strong>，也就是<strong data-nodeid="1127">有界数据</strong>，这些数据在处理之前就已经存储在我们的源数据地址，当我们要进行处理的时候直接从这个数据集进行读取就可以了。</p>
<p data-nodeid="1043"><strong data-nodeid="1131">流处理</strong></p>
<p data-nodeid="1044">与批处理相对的，流处理的数据是<strong data-nodeid="1137">无界的</strong>，数据就像一条河里的水源源不断地从上游流到计算框架中，我们不知道数据的总量是多少，也不知道什么时候结束。更发散来考虑，当雨雪天气，水流可能激增，而干旱时节水流可能枯竭，这就是流处理所应对的情况。</p>
<p data-nodeid="1045">我们最开始讲过的 Hadoop&nbsp;MapReduce 就是一个批处理计算框架，而 Spark 和 Flink 是混合计算框架，既可以进行批处理计算，也可以进行流处理计算。带着这两个概念，我们来看一下什么是 Flink。</p>
<h3 data-nodeid="2326" class="">什么是 Flink</h3>



<p data-nodeid="1047">我们在 Flink 的官网可以看到对 Flink 的精准描述：</p>
<blockquote data-nodeid="1048">
<p data-nodeid="1049">Apache Flink 是一个框架和分布式处理引擎，用于在无边界和有边界数据流上进行有状态的计算，Flink 能在所有常见集群环境中运行，并能以内存速度和任意规模进行计算。</p>
</blockquote>
<p data-nodeid="1050">其实 Flink 的产生时间跟 Spark 差不多，最早出现在 2008 年由柏林理工大学发起的研究项目中，并在 2014 年把代码捐赠给了 Apache 基金会。</p>
<p data-nodeid="1051">同时，Flink 的功能与 Spark 也基本一致，都属于<strong data-nodeid="1148">大数据计算框架</strong>，那么我们来看一下 Flink 有什么特色。</p>
<h3 data-nodeid="1052">Flink 的特色</h3>
<p data-nodeid="6512" class="">1.<strong data-nodeid="6517">数据皆流</strong></p>


<p data-nodeid="1054">在 Flink 的构建思想上，<strong data-nodeid="1164">把所有的数据都看作是流式数据</strong>，<strong data-nodeid="1165">所有的处理方式都是流处理</strong>。对于事实上的批数据，只不过当成一种特殊的数据流，我们称之为有界流，也就是说这个流数据有开始有结束，我们可以等着这个流获取完全后统一进行计算。</p>
<p data-nodeid="1055">对于在我们公司中真正的流数据，我们将其称为无界流，这个数据只有开始，没有结束。只要我们的业务还在运转，用户还在浏览我们的 App，查看、下单、支付数据就会源源不断地传送过来。处理这种数据，不光是汇总起来就完事了，很多时候还需要注意<strong data-nodeid="1171">数据的顺序</strong>，比如说正常情况下肯定是先点击，再下单，最后支付。如果数据的顺序搞错了，已经有了支付，但是没有点击和下单，那这数据计算起来就乱了套了。</p>
<p data-nodeid="1056">我们说 Flink 就是为了流处理而生，自然而然，Flink 的创建者也为流处理做了很多优化。在数据接收方面，Flink 与 Kafka 有异曲同工之妙，都是基于<strong data-nodeid="1177">事件驱动</strong>的，也就是数据随来随处理，而 Spark 实现的流处理实际上是微批处理，只是把数据块划分的更小。同时，Flink 还有精确的时间控制和状态以保障一致性。</p>
<p data-nodeid="7564" class="">2.<strong data-nodeid="7569">多平台支持</strong></p>


<p data-nodeid="1058">同 Spark 一样，Flink 可以作为单独的服务进行部署运行，也可以与 Hadoop、Mesos、Kubernetes 集成部署。</p>
<p data-nodeid="8616" class="">3.<strong data-nodeid="8621">高速</strong></p>


<p data-nodeid="1060">同样的，Flink 也使用了内存作为计算的中间缓存。在前面我们已经知道，批处理由于是已经累积下来的数据，所以需要大吞吐量，而流处理是来即处理，需要的是低延迟。在 Flink 中，使用了一种缓存块机制来保障两种计算的速度。当缓存块的超时时间设置为 0，那么只要有数据就立即处理，适合处理无界数据，而当缓存块超时时间设置为无限大，那么就要等着数据结束才处理，这样更适合有界数据。</p>
<p data-nodeid="1061">当然，在具体的工作中，开发人员可以<strong data-nodeid="1199">根据业务场景对超时时间进行配置</strong>，<strong data-nodeid="1200">以获取最佳状态</strong>。而这些中间存储，首先是会存在内存中，如果内容不够用再开启磁盘存储。</p>
<h3 data-nodeid="1062">Flink 的组件架构</h3>
<p data-nodeid="1063"><img src="https://s0.lgstatic.com/i/image6/M00/18/E1/Cgp9HWBJhG2AedBYAAIOse0WBJU239.png" alt="图片1.png" data-nodeid="1204"></p>
<div data-nodeid="1064"><p style="text-align:center">Flink 的组件架构图</p></div>
<p data-nodeid="1065">与 Spark 类似，Flink 也实现了用一整套组件来支持整个体系的运转，如上图所示。</p>
<ul data-nodeid="1066">
<li data-nodeid="1067">
<p data-nodeid="1068">最底层是部署相关的组件，包括了支持本地单机部署、集群部署，以及云上部署的组件。</p>
</li>
<li data-nodeid="1069">
<p data-nodeid="1070">Core 核心层，是 Flink 实现的最关键组件，包括支持分布式的流处理运算，各种分配和调度系统都在这一部分实现，为更上层的 API 提供基础服务，这也为用户的方便使用奠定了基础。</p>
</li>
<li data-nodeid="1071">
<p data-nodeid="1072">API 和 Lib 层，提供了流处理和批处理计算的各种 API，以及针对特定的计算支持库，比如 FlinkML 就是和 SparkMlib 类似的机器学习库，而 Gelly 是和 GraphX 类似的图处理计算库。</p>
</li>
</ul>
<p data-nodeid="1073">看起来，Flink 与 Spark 似乎是没有什么太大的区别，那么下面我们来总结对比下 Spark 和 Flink。</p>
<h3 data-nodeid="1074">Spark&nbsp;VS&nbsp;Flink</h3>
<p data-nodeid="3364" class="">1.<strong data-nodeid="3369">核心实现</strong></p>


<p data-nodeid="1076">在核心实现方面：</p>
<ul data-nodeid="1077">
<li data-nodeid="1078">
<p data-nodeid="1079">Spark 主要使用 Scala 语言编写而成；</p>
</li>
<li data-nodeid="1080">
<p data-nodeid="1081">而 Flink 早期是使用 Java 进行编写的，但是后期的很多更新也使用了 Scala 语言。</p>
</li>
</ul>
<p data-nodeid="1082">Scala 语言对大数据处理更加友好，但是做程序员的同学都知道，使用什么样的语言来编写程序其实对整个体系架构的影响并不是很大。</p>
<p data-nodeid="4410" class="">2.<strong data-nodeid="4415">编程接口</strong></p>


<p data-nodeid="1084">在编程接口方面，Spark 和 Flink 就更加相似了。二者都提供了对各种编程语言的支持，包括 Java、Python、Scala 等，都可以用来编写 Spark 或者 Flink 程序。</p>
<p data-nodeid="5460" class="">3.<strong data-nodeid="5465">计算模型</strong></p>


<p data-nodeid="1086">计算模型，或者我们也可以叫作<strong data-nodeid="1233">设计理念</strong>，这是引发两者前进方向的最大区别点。</p>
<p data-nodeid="1087">前面我们已经介绍过了，Flink 是把所有数据都看作流来进行处理，所以它本身对<strong data-nodeid="1239">流式数据</strong>有着非常优秀的计算性能，在流计算方面做了大量的优化。</p>
<p data-nodeid="1088">而 Spark 虽然也是混合计算框架，但是 Spark 的设计理念是<strong data-nodeid="1249">批处理</strong>，也就是所有数据都是批数据。在处理流数据的时候使用了模拟的办法，把数据分割成更小的批来进行处理，从而模拟流式处理，所以在 Spark 中的流处理，我们也可以称为<strong data-nodeid="1250">微批处理</strong>。所以，Spark 的迭代和优化只能无限接近真流处理，而无法称为真流处理。但是对于 Flink 则没有这样的情况。</p>
<p data-nodeid="1089">有一种说法很妙——“我们可以认为 Flink 选择了‘batch on streaming’的架构，不同于 Spark 选择的‘streaming on batch’架构”。</p>
<p data-nodeid="1090">当然，早期的 Flink 由于针对流处理进行的优化，也使得它在批处理方面仍然没有 Spark 性能良好，这也就引出了我们下面要介绍的流批一体技术。</p>
<h3 data-nodeid="1091">流批一体</h3>
<p data-nodeid="1092"><img src="https://s0.lgstatic.com/i/image6/M01/18/DD/CioPOWBJhH6AQM7TAAFlUA0Gdi0482.png" alt="图片2.png" data-nodeid="1256"></p>
<div data-nodeid="1093"><p style="text-align:center">Flink 架构变化图</p></div>
<p data-nodeid="1094">在 Flink 早期的时候，虽然说在思想上是把批处理当作流处理的一种特殊形式，但是实际上在处理的时候仍然是分开实现的，不管是 API 层还是 Core 层的 Runtime 都没能够实现完全统一。用户在进行流处理和批处理的时候要分别进行程序的编写，非常麻烦。</p>
<p data-nodeid="1095">但是在 Flink 的 1.9 版本，Flink 开始完善流批一体，Flink SQL 率先实现了流批一体语义，使得用户只需学习、使用一套 SQL 就可以进行流批一体的开发，大幅节省开发成本。</p>
<p data-nodeid="1096">但是只调整 SQL&nbsp;API 并不能解决用户的所有需求。一些定制化程度较高，比如需要精细化的操纵状态存储的作业还是需要继续使用 DataStream API。在常见的业务场景中，用户写了一份<strong data-nodeid="1268">流计算作业</strong>后，一般还会再准备一个<strong data-nodeid="1269">离线作业</strong>进行历史数据的批量回刷。但是 DataStream 虽然能很好地解决流计算场景的各种需求，但却缺乏对批处理的高效支持。</p>
<p data-nodeid="1097">所以，在最近的两年内，Flink 主打流批一体的升级，从上面的架构图变化我们可以看出来，在最新的 1.11 版本，不管是 SQL 还是 DataStream&nbsp;API，都已经可以使用同一套编写规范，而只需要进行简单的选择就可以进行批处理或者流处理。</p>
<p data-nodeid="1098">阿里巴巴在 2019 年初收购了 Flink 创始公司和团队 Ververica，开始投入更多资源在 Flink 生态和社区上。到了 2020 年，国内外主流科技公司几乎都已经选择了 Flink 作为其实时计算解决方案，我们看到 Flink 已经成为大数据业界实时计算的事实标准，在阿里巴巴公布的迭代计划中，Flink 会支持更加智能的流批融合，甚至是自动切换。</p>
<p data-nodeid="1099">随着流批一体技术的实现，使用 Flink 的公司不再需要维护两套架构，部署两套代码，维护成本会进一步降低，我觉得 Flink 会变得更加普及，甚至是取代 Spark 成为新一代主流计算框架。</p>
<h3 data-nodeid="1100">总结</h3>
<p data-nodeid="1101">在这一讲中，我们介绍了一个与 Spark 极为类似的计算框架 Flink，并将 Flink 与 Spark 进行了对比。你可以看到，Flink 与 Spark 出现时间差不多，所实现的功能也极为类似，但是，由于 Flink 与 Spark 的核心理念的差别，也就是对批处理与流处理的思考不同，使得 Flink 更加符合当前大型互联网公司的需求，尤其是随着流批一体技术的落地，未来谁将获得更多关注，让我们尽情期待吧。</p>
<p data-nodeid="1102"><img src="https://s0.lgstatic.com/i/image6/M00/18/E1/Cgp9HWBJhJuAQeCtAAV4evNdojo805.png" alt="（大数据13金句.png" data-nodeid="1277"></p>
<p data-nodeid="1103">那关于Flink 与 Spark 你还有什么其他的观点吗？欢迎在留言区与我交流。</p>
<p data-nodeid="1104">下一模块我们将进入大数据的挖掘和分析，先从下一讲的标准化数据挖掘全流程说起，到时见。</p>
<hr data-nodeid="1105">
<p data-nodeid="1106"><a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1284"><img src="https://s0.lgstatic.com/i/image6/M00/00/6D/Cgp9HWAaHaOAI85HAAUCrlmIuEw966.png" alt="Drawing 2.png" data-nodeid="1283"></a></p>
<p data-nodeid="1107"><strong data-nodeid="1288">《大数据开发高薪训练营》</strong></p>
<p data-nodeid="1108" class="">PB 级企业大数据项目实战 + 拉勾硬核内推，5 个月全面掌握大数据核心技能。<a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1292">点击链接</a>，全面赋能！</p>

---

### 精选评论


