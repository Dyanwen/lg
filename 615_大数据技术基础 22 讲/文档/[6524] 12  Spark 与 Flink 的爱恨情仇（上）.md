<p data-nodeid="1457" class="">前面我们相对完整地介绍了 Hadoop 架构，并且在上一讲讲解了 Hadoop 中的计算框架 MapReduce 的基本思想。我们已经知道，MapReduce 的主要功能就是<strong data-nodeid="1558">并行计算</strong>，但是它也不是十全十美的，MapReduce 高成本的硬伤使得它已经不能很好地解决新时代的问题。</p>
<p data-nodeid="1458"><img src="https://s0.lgstatic.com/i/image6/M00/18/36/Cgp9HWBIlC-ASvztAAV5KV3DBYA200.png" alt="（大数据12金句.png" data-nodeid="1561"></p>
<p data-nodeid="1459">这一讲我们所要介绍的 Spark 就是由 MapReduce 思想衍生出来的新的分布式计算框架，但是它又不仅仅是一个 MapReduce 的升级版。接下来就让我们看一下，Spark 到底是一个什么样的工具。</p>
<h3 data-nodeid="1460">什么是 Spark</h3>
<p data-nodeid="1461">打开 Spark 的官网，我们看到的第一句话就是对 Spark 的定义：<strong data-nodeid="1577">Spark 是用于大规模数据处理的通用分析引擎</strong>。当然，原文是英文的，这句是我翻译过来的。这句话非常简洁明了地讲解了 Spark 的功能，一个是<strong data-nodeid="1578">针对大规模数据</strong>，一个是<strong data-nodeid="1579">通用分析引擎</strong>。</p>
<p data-nodeid="1462">让我们简单回顾一下 Spark 的发展历程。</p>
<p data-nodeid="1463">在 2009 年，加州大学伯克利分校的 RAD 实验室（AMP 实验室前身）推出了 Spark 框架，并表明 Spark 是一个类似 MapReduce 的通用并行框架，<strong data-nodeid="1586">可用来构建大型的、低延迟的数据分析应用程序</strong>。实验室的研究人员在使用 MapReduce 时发现它在迭代计算和交互计算上效率低下，于是开始设计 Spark。很快，Spark 在一些任务上的效率就已经比 MapReduce 提升了 10 倍以上。</p>
<p data-nodeid="1464">这里需要提到 Spark 的一个特点，那就是<strong data-nodeid="1592">低延迟</strong>。从一开始 Spark 就瞄准了低延迟这样一个目标，这也为它后来能够进入工业界并广泛地应用于生产奠定了基础。</p>
<p data-nodeid="1465">在 2013 年的时候，Spark 项目加入了 Apache，并在 2014 年的 Gray&nbsp;sort&nbsp;Benchmark 比赛上全面超越了 Hadoop 拿下了当年的冠军。这个比赛是对 100TB 的数据进行排序操作：</p>
<ul data-nodeid="1466">
<li data-nodeid="1467">
<p data-nodeid="1468">Spark 使用了 206 台 EC2 机器在 23 分钟内完成了操作；</p>
</li>
<li data-nodeid="1469">
<p data-nodeid="1470">而 Hadoop 的 MapReduce 则使用了 2100 个机器并耗费了 72 分钟。</p>
</li>
</ul>
<p data-nodeid="1471">在后来的一段时间里，Spark 在互联网公司的大数据架构中逐渐成为<strong data-nodeid="1601">主流计算框架</strong>。</p>
<h3 data-nodeid="1472"><strong data-nodeid="1605">Spark 的特色</strong></h3>
<p data-nodeid="1473">在 Spark 的官网上我们也可以看到，Spark 的特色有五个方面。</p>
<p data-nodeid="1474">（1）<strong data-nodeid="1611">高速</strong></p>
<p data-nodeid="1475">Spark 使用了最新的 DAG 调度方案，查询、优化和物理执行引擎，在批处理和“流”处理上都表现优异。这里，我给“流”处理加了引号，在后面我们会解释为什么这里会加一个引号。在官方给出的示例中，在 Spark 框架中执行逻辑回归运算要比 MapReduce 快一百多倍。</p>
<p data-nodeid="1476">（2）<strong data-nodeid="1617">易用</strong></p>
<p data-nodeid="1477">Spark 提供了 80 多种<strong data-nodeid="1623">高阶操作</strong>，构建应用更加方便。Spark 还提供了多种语言支持，使用 Java、Scala、Python、R 甚至是 SQL 都可以非常容易地编写 Spark 程序。</p>
<p data-nodeid="1478">（3）<strong data-nodeid="1628">通用</strong></p>
<p data-nodeid="1479">如下图所示，Spark 除了提供计算框架以外还整合了 SQL、流式处理，以及机器学习算法库和图操作算法库。其中 Spark&nbsp;SQL 支持使用 HiveQL 和 Spark 进行交互。</p>
<p data-nodeid="1480"><img src="https://s0.lgstatic.com/i/image6/M01/16/E2/CioPOWBHD7-AREhMAAGqR6vGaeg851.png" alt="图片1.png" data-nodeid="1632"></p>
<p data-nodeid="1481">（4）<strong data-nodeid="1637">多平台支持</strong></p>
<p data-nodeid="1482">Spark 本身是可以独立运行的，当然，它也可以运行在 Hadoop、Mesos、Kubernetes，甚至是云平台上。它还支持访问各种不同的数据源，比如 HDFS、HBase、Hive、Cassandra 都是可以的。</p>
<p data-nodeid="1483">（5）<strong data-nodeid="1643">内存化</strong></p>
<p data-nodeid="1484">这点在官网上没有，但是很重要。在运算流程上，Spark 和 MapReduce 基本上是一致的，但是有一个很大的不同就是中间结果的存储：</p>
<ul data-nodeid="1485">
<li data-nodeid="1486">
<p data-nodeid="1487"><strong data-nodeid="1649">MapReduce 所有的中间结果都是保存在磁盘上</strong>；</p>
</li>
<li data-nodeid="1488">
<p data-nodeid="1489"><strong data-nodeid="1654">而 Spark 的中间结果是保存在内存中的</strong>。</p>
</li>
</ul>
<p data-nodeid="1490">这你也就明白了为什么 Spark 的运算速度要快那么多，内存起到了很重要的作用，同样的，由于使用内存存储， Spark 处理的数据规模相对来说就会小一些，但是幸运的是现在内存也不是很贵，公司也都愿意多花点钱从而节约点时间。</p>
<h3 data-nodeid="1491">Spark 基础</h3>
<h4 data-nodeid="1492">RDD</h4>
<p data-nodeid="1493">RDD 是 Resillient Distributed Dataset（弹性分布式数据集）的简称，它是 Spark 的一个基本数据结构，也是<strong data-nodeid="1663">Spark 最核心的数据结构</strong>。</p>
<p data-nodeid="1494">Spark 在运行时，会将 RDD 分割成不同的 Partition（分区），并把这些分区投放到不同的计算节点下面，进行并行运算。实际上，Spark 程序的整个生命流程，就是对这些数据的处理，创建 RDD、转换 RDD、调用 RDD 操作计算各种结果等。所以，理解了 RDD 的各种操作基本上就了解了 Spark 的核心，对这部分感兴趣的同学可以寻找更多的资料来进行学习。</p>
<h4 data-nodeid="1495">两种操作</h4>
<p data-nodeid="1496">在我们的运算中，通常包含两种操作，一个是<strong data-nodeid="1687">Transformation</strong>（<strong data-nodeid="1688">转换操作</strong>），一个是<strong data-nodeid="1689">Action</strong>（<strong data-nodeid="1690">行动操作</strong>）。这两个操作最大的区别是 Spark 对它们的<strong data-nodeid="1691">处理逻辑</strong>是不一样的。</p>
<p data-nodeid="1497">对于转换操作，比如说 Filter（过滤），这些操作是对 RDD 本身的一些操作，其结果是数据的规模发生变化，而数据的类型或者说内容不会发生变化。</p>
<p data-nodeid="1498">对于这种操作 Spark 是进行<strong data-nodeid="1702">惰性运算</strong>，也就是说 Spark 只是记录下这个操作，而不会真的去执行它。只有当 Spark 遇到了行动操作，才会把之前的转换操作进行统一的处理，我们可以认为行动操作是一些产生结果的操作，它的输出是<strong data-nodeid="1703">非 RDD 对象</strong>。这么做的好处是减少了不必要的存储开销，毕竟 Spark 是使用内存来进行中间结果的存储。</p>
<p data-nodeid="1499">这么说可能有点迷茫了，我们写两行代码再来解释一下。</p>
<blockquote data-nodeid="1500">
<p data-nodeid="1501">val RDD1 = sc.textFile("data.txt")<br>
val RDD2 = RDD1.map(x =&gt; x+1)<br>
val RDD3 = RDD2.filter(lambda x: "Python" in x)<br>
val RDD4 = RDD3.reduceByKey((x,y) =&gt; x + y)</p>
<blockquote data-nodeid="1502">
<p data-nodeid="1503">val RDD5 = RDD2.filter(lambda x: "Java" in x)<br>
RDD5.count</p>
</blockquote>
</blockquote>
<p data-nodeid="1504">根据这几行代码，我们可以得出如下图的一个执行逻辑。</p>
<p data-nodeid="1505"><img src="https://s0.lgstatic.com/i/image6/M01/16/E5/Cgp9HWBHD8uAdDXAAAEIkWLgGpg729.png" alt="图片2.png" data-nodeid="1730"></p>
<ul data-nodeid="1821">
<li data-nodeid="1822">
<p data-nodeid="1823" class="te-preview-highlight">首先，我们从文件中读取数据，这时候形成 RDD1。</p>
</li>
<li data-nodeid="1824">
<p data-nodeid="1825">后面两步执行了 map、filter 和 reduceByKey 操作，map 操作给所有值加了 1，而 filter 操作过滤出带有 Python 的数据，reduceByKey 对 x 和 y 进行相加。这几个操作都属于 Transformation 操作，输出的结果都仍然是 RDD。</p>
</li>
<li data-nodeid="1826">
<p data-nodeid="1827">而最后一行执行了 count 操作，count 就是一个 Action 操作，它返回的是 RDD5 中的数据元素个数。</p>
</li>
</ul>

<p data-nodeid="1513">如果 Spark 对每一步都认真执行，那么第一步读取文件后，就已经把全量的数据存储在了内存中，但实际这并没有什么用，当 Spark 根据上面的执行逻辑分析了运算流程之后，会对 Transformation 操作进行整合，直到遇到 Action 操作才真正去进行运算。不知道这样解释，你有没有明白一些。</p>
<h4 data-nodeid="1514">两种依赖关系</h4>
<p data-nodeid="1515">由于在 Spark 中流转的数据通常都是<strong data-nodeid="1741">RDD 格式</strong>，从我们的代码可以看出，每一步生成的 RDD 都会依赖于上一步的 RDD，所以，具备上下联系的 RDD 产生的关系被称为依赖关系。</p>
<p data-nodeid="1516">根据不同情况又分成<strong data-nodeid="1751">窄依赖关系</strong>和<strong data-nodeid="1752">宽依赖关系</strong>两种：</p>
<ul data-nodeid="1517">
<li data-nodeid="1518">
<p data-nodeid="1519">窄依赖关系指的是生成下级 RDD 不会引起数据在不同的分区（Partition）之间进行迁移（Shuffle）；</p>
</li>
<li data-nodeid="1520">
<p data-nodeid="1521">宽依赖指的是要生成的 RDD 依赖于多个分区的数据，很明显这会导致处理速度的下降。</p>
</li>
</ul>
<p data-nodeid="1522">接下来，我们再来看一下 Spark 整体的运行架构。</p>
<p data-nodeid="1523"><img src="https://s0.lgstatic.com/i/image6/M01/16/E6/Cgp9HWBHD9WAWOAiAAHtErtXIXA457.png" alt="图片3.png" data-nodeid="1758"></p>
<p data-nodeid="1524">如上图，这是一个不太完整的运行架构，但是我们可以借此大概了解一下其中的概念。</p>
<p data-nodeid="1525">当我们编写了一个 Spark 程序，也就是一个 Application 时：</p>
<ul data-nodeid="1526">
<li data-nodeid="1527">
<p data-nodeid="1528">这个 Application 以 Action 操作为界限，被划分成多个 Job；</p>
</li>
<li data-nodeid="1529">
<p data-nodeid="1530">一个 Job 以 Shuffle 为界限被划分成多个 Stage；</p>
</li>
<li data-nodeid="1531">
<p data-nodeid="1532">而一个 Stage 又会分成多个 Task，Stage 的划分和调度是由 DAGScheduler 来负责的，而 Task 由 TASKScheduler 来进行调度。</p>
</li>
</ul>
<p data-nodeid="1533">这些调度的终极目标就是<strong data-nodeid="1769">根据依赖的关系找出开销最小的调度方法</strong>。最终，这些任务被分派到不同的 Worker 上进行运算。</p>
<p data-nodeid="1534"><img src="https://s0.lgstatic.com/i/image6/M01/16/E2/CioPOWBHD92ASZUGAAHQghdzsQg687.png" alt="图片4.png" data-nodeid="1772"></p>
<p data-nodeid="1535">我们再把视野放得稍微高一点，来看看运行架构。如上图所示，这里的 Driver 也就是我们在代码中常说的 main 函数，当它被执行时会创建一个 SparkContext 对象，这个对象会与服务器中的 Cluster&nbsp;Manager 进行通信，从而申请任务所需要的资源。当成功获得资源之后，它的 Task 会被分配到 Worker 的执行器中，从而处理对应的运算。</p>
<h3 data-nodeid="1536"><strong data-nodeid="1777">开发时需要注意的问题</strong></h3>
<p data-nodeid="1537">正是由于 Spark 所具备的这些特性，在开发的时候也会引起一些问题。</p>
<p data-nodeid="1538">（1）<strong data-nodeid="1783">数据倾斜</strong></p>
<p data-nodeid="1539">由于 RDD 数据会被分配到不同的分区，存在常见的问题就是<strong data-nodeid="1789">数据倾斜</strong>，某些分区分配了过多的数据，而另外的一些分区则分配了很少的数据，这会导致一些节点的运算过于缓慢。</p>
<p data-nodeid="1540">（2）<strong data-nodeid="1794">过多地使用 Action 操作</strong></p>
<p data-nodeid="1541">由于 Action 操作会引发真正的运算，并对数据进行缓存，所以，如果 Action 操作太多会造成内存的大量消耗，从而占用过多的资源。</p>
<p data-nodeid="1542">（3）<strong data-nodeid="1800">宽依赖过多</strong></p>
<p data-nodeid="1543">前面我们讲到，宽依赖会引发 Shuffle，将不同 Partition 的数据迁移到同一个 Partition 中去，所以这个操作也会消耗很多时间。</p>
<p data-nodeid="1544">这几个问题是我们在开发一个 Spark 程序时需要注意的问题。</p>
<h3 data-nodeid="1545">总结</h3>
<p data-nodeid="1546">在这一讲中，我对 Spark 的基本概念和发展历程做了简单的介绍，然后对其中比较重要的一些概念进行了说明。你是否对 Spark 框架已经有了一个基本了解，我想至少应该了解了 Spark 是什么。</p>
<p data-nodeid="1547">至于说如何使用 Spark 并运用自如，那可能需要你在课下不断地去练习，甚至需要有一个落地的场景，在应用中去学习。有任何心得或者问题，都欢迎和我交流。</p>
<p data-nodeid="1548">下一讲，我们将介绍一个新的计算框架 Flink，一起来看一看，跟 Spark 比起来，Flink 又有什么样的不同呢？</p>
<hr data-nodeid="1549">
<p data-nodeid="1550"><a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1811"><img src="https://s0.lgstatic.com/i/image6/M00/00/6D/Cgp9HWAaHaOAI85HAAUCrlmIuEw966.png" alt="Drawing 2.png" data-nodeid="1810"></a></p>
<p data-nodeid="1551"><strong data-nodeid="1815">《大数据开发高薪训练营》</strong></p>
<p data-nodeid="1552" class="">PB 级企业大数据项目实战 + 拉勾硬核内推，5 个月全面掌握大数据核心技能。<a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1819">点击链接</a>，全面赋能！</p>

---

### 精选评论


