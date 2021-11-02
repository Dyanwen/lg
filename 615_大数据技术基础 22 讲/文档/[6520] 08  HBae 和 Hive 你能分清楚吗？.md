<p data-nodeid="1413" class="">在上一讲中，我带你了解了 HDFS 的基本框架，并且动手安装了 Hadoop 系统。我们都知道 HDFS 是 Hadoop 中用来管理文件的系统，是 Hadoop 的核心之一。在实际的生产工作中，仅仅有一套文件管理系统还不能很好地支撑我们业务的需求，我们还希望对数据进行更加便捷的操作，这一讲，我就带你了解下，在日常工作中经常用的而且与 HDFS 有紧密联系的两个工具 Hive 和 HBase。</p>
<h3 data-nodeid="1414">Hive</h3>
<p data-nodeid="1415">我在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=615#/detail/pc?id=6517" data-nodeid="1492">《05 | 大数据开发必备工具——Hadoop》</a>中做过一些简单的介绍，在 Hadoop 系统中，负责计算的部分是 MapReduce，也就是说我们要处理 HDFS 中存储的数据，进行各种统计分析以及运算的话需要去开发一个 MapReduce 程序。</p>
<p data-nodeid="1416">虽然说 MapReduce 已经对分布式计算进行了很好的封装（这部分我们会在&lt;11 | MapReduce 处理大数据的基本思想有哪些&gt;中讲到），但是使用其 API 进行开发对于很多人来说仍然是一件很困难的事情。比如说很多的数据产品经理或者运营人员只是想统计一下数字，却要去学习如何写代码开发的一套程序，这个难度可想而知，因此 Hive 便应运而生。Hive 会解析 SQL 语句然后转化成 MapReduce 的程序运行，只是学习 SQL 语句对于一个产品经理来说就要简单得多了。</p>
<p data-nodeid="1417">下面我们就来介绍一下 Hive。简单来说，Hive 就是一个<strong data-nodeid="1506">数据仓库</strong>，仓库中的数据都是在 HDFS中管理的数据文件，同时 Hive 支持类似 SQL 语句的功能，你可以通过这些语句来进行数据分析，Hive 会把这些语句转换成可执行的 MapReduce 代码，然后进行计算。这里的计算，仅限于查找和分析。</p>
<p data-nodeid="1418">Hive 所处理的是已经存储起来的数据，这种计算也就是我们所说的<strong data-nodeid="1512">离线计算</strong>（区别于实时计算）。通过 Hive 的操作，可以让你在操作数据时感觉是在使用 MySQL，从而减少你的学习成本。</p>
<p data-nodeid="1419">接下来，我们来看一下 Hive 的基本体系架构。如下图所示：</p>
<p data-nodeid="3198" class=""><img src="https://s0.lgstatic.com/i/image6/M00/09/21/CioPOWA1tRiAQ376AAGwuar_vIs772.png" alt="图片1.png" data-nodeid="3201"></p>



<h4 data-nodeid="1421">Hive 的体系架构</h4>
<p data-nodeid="1422" class="">上面这张图来自早期的 Hive 架构文档，分成两大部分，左侧是 Hive 的主体，右侧是Hadoop 系统，右上是 MapReduce，右下是 HDFS，中间有几条线连接，说明了 Hive 与 Hadoop 两大核心的关系。</p>
<p data-nodeid="1423">（1）<strong data-nodeid="1523">UI</strong></p>
<p data-nodeid="1424"><strong data-nodeid="1528">用户界面</strong>，我们也可以认为是一个客户端，这里主要负责与使用者的交互，我们通过 UI 向系统提交查询和其他操作。当然，在 Hive 中还封装了 ThriftServer，我们可以在开发中使用 Java、 Python 或者 C++ 等语言来访问 Server，从而调用 Hive。</p>
<p data-nodeid="1425">（2）<strong data-nodeid="1533">驱动器（Driver）</strong></p>
<p data-nodeid="1426">驱动器在接收 HiveQL 语句之后，创建会话来启动语句的执行，并监控执行的生命周期和进度。在图中可以看到，驱动器既负责与<strong data-nodeid="1539">编译器</strong>的交互，又负责与执行引擎的交互。</p>
<p data-nodeid="1427">（3）<strong data-nodeid="1544">编译器（Compiler）</strong></p>
<p data-nodeid="1428">编译器接收驱动器传来的 HiveQL，并从元数据仓中获取所需要的元数据，然后对 HiveQL 语句进行编译，将其转化为可执行的计划，按照不同的执行步骤拆分成 MapReduce 和 HDFS 的各个阶段的操作并发送给驱动器。</p>
<p data-nodeid="1429">（4）<strong data-nodeid="1550">执行引擎（Execution Engine）</strong></p>
<p data-nodeid="1430">在编译和优化之后，执行器将执行任务。它对 Hadoop 的作业进行<strong data-nodeid="1556">跟踪和交互</strong>，调度需要运行的任务。</p>
<p data-nodeid="1431">（5）<strong data-nodeid="1561">元数据仓（Metastore）</strong></p>
<p data-nodeid="1432">元数据指的是我们构建的 Hive 表的表名、表字段、表结构、分区、类型、存储路径等等，元数据通常存储在传统的关系型数据库中，比如 MySQL。</p>
<h4 data-nodeid="1433">Hive 的优缺点</h4>
<p data-nodeid="1434">Hive 的优点有很多，我主要从以下几个方面为你讲解。</p>
<p data-nodeid="1435">（1）<strong data-nodeid="1569">简单易上手</strong></p>
<p data-nodeid="1436">只需要了解 SQL 语言就可以使用 Hive，降低了使用 MapReduce 进行数据分析的难度，很多互联网公司都会使用 Hive 进行日志分析，比如说淘宝、美团等等，使用 Hive 统计网站的 PV、UV 等信息，简单便捷。</p>
<p data-nodeid="1437">（2）<strong data-nodeid="1575">Hive 提供统一的元数据管理</strong></p>
<p data-nodeid="1438">通过元数据管理可以实现描述信息的格式化，使得数据可以共享给 Presto、Impala、SparkSQL 等 SQL 查询引擎。</p>
<p data-nodeid="1439">（3）<strong data-nodeid="1581">可扩展性好</strong></p>
<p data-nodeid="1440">跟 Hadoop 的其他组件一样，Hive 也具备良好的可扩展性，只需要添加机器就可以部署分布式的 Hive 集群。</p>
<p data-nodeid="1441">（4）<strong data-nodeid="1587">支持自定义函数（UDF）</strong></p>
<p data-nodeid="1442">SQL 的功能虽然非常多，足够支撑我们平时常用的统计方案，但是对于一些个性化的定制方案，使用 SQL 明显要麻烦很多，Hive 支持使用自定义函数的方式来加入自己编写的功能，方便了开发人员。</p>
<p data-nodeid="1443">当然 Hive 也是有缺点的。</p>
<p data-nodeid="1444">（1）<strong data-nodeid="1594">速度较慢</strong></p>
<p data-nodeid="1445">由于 Hive 的底层数据仍然是存储在 HDFS 上的，所以速度比较慢，只适合离线查询。在写程序时一般也是使用 Hive 来一次性加载数据，不适合在代码中反复访问。</p>
<p data-nodeid="1446">（2）<strong data-nodeid="1600">不支持单条数据操作</strong></p>
<p data-nodeid="1447">这个仍然是跟 HDFS 的存储相关，我们不能任意修改 HDFS 里的数据，所以 Hive 也不行，要想修改数据只能<strong data-nodeid="1606">整个文件进行替换</strong>。</p>
<h3 data-nodeid="1448">HBase</h3>
<p data-nodeid="1449">跟 Hive 不同，HBase 是一个在 Hadoop 大数据体系中应用很多的<strong data-nodeid="1613">NoSQL 数据库</strong>，HBase 源于谷歌发表的论文：Bigtable。HBase 同样利用 HDFS 作为底层存储，但是并不是简单地使用原本的数据，只是使用 HDFS 作为它的存储系统。也就是说，HBase 只是利用 Hadoop 的 HDFS 帮助其管理数据的持久化文件。HBase 提供实时处理数据的能力，弥补了早期 Hadoop只能离线处理数据的不足。</p>
<h4 data-nodeid="1450">HBase 的表结构</h4>
<p data-nodeid="1451">下图是 HBase 的表结构：</p>
<p data-nodeid="5340" class=""><img src="https://s0.lgstatic.com/i/image6/M00/09/21/CioPOWA1tTSAGS8IAAHeE1sSO4k668.png" alt="图片2.png" data-nodeid="5343"></p>



<p data-nodeid="1453">（1）<strong data-nodeid="1624">行键</strong>（Row&nbsp;Key）</p>
<p data-nodeid="1454" class="te-preview-highlight">这是我们<strong data-nodeid="1634">一行数据的唯一标识</strong>，比如说我们平时的数据都会有一个唯一 ID，就可以用来作为 Row&nbsp;Key。但是需要注意的是，HBase 在存储 Row&nbsp;Key 的时候是按照<strong data-nodeid="1635">字典顺序</strong>存放的，所以如果你的 Row&nbsp;Key 不是以分布均匀的数字或字母开头的，很可能造成存储集中在某一台机器上，这会大大降低查询效率，所以这种时候需要设计存储的 Row Key，比如在每个 ID 的前面都加一个 HASH 值来提升查询性能。</p>
<p data-nodeid="1455">（2）<strong data-nodeid="1641">列簇</strong>（Column&nbsp;Family）</p>
<p data-nodeid="1456">可以看作是<strong data-nodeid="1647">一组列</strong>，实际上一个列簇的作用也是用来管理若干个列，优化查询速度。所以列簇的名字要尽量短，同时对于经常需要一起查询的列放在一个列簇下面。比如说对于用户信息，一个用户的静态属性（姓名、年龄、电话、地址等）可以放在一个列簇下面，动态属性（点赞、收藏、评论等）可以放在一个列簇下面。HBase表中的列簇需要预先定义，而列不需要，如果要新增列簇就要先停用这个表。</p>
<p data-nodeid="1457">（3）<strong data-nodeid="1653">单元</strong>（Cell）</p>
<p data-nodeid="1458">指的是一个确定的<strong data-nodeid="1659">存储单元</strong>。通过 Row&nbsp;Key、列簇 、列名以及版本号来确定。</p>
<p data-nodeid="1459">（4）<strong data-nodeid="1665">时间戳</strong>（Timestamp）</p>
<p data-nodeid="1460">用来标记前面说的一份数据的不同版本。</p>
<p data-nodeid="1461">（5）<strong data-nodeid="1672">区域</strong>（Region）</p>
<p data-nodeid="1462">一个 Region 可以看作是多行数据的集合。当一个表的数据逐渐增多，需要进行分布式存储，那么这个表就会自动分裂成多个 Region，然后分配到不同的 RegionServer 上面去。</p>
<h4 data-nodeid="1463">HBase 的优缺点</h4>
<p data-nodeid="1464">HBase 的优势在于<strong data-nodeid="1688">实时计算</strong>，所有实时数据都直接存入 HBase 中，客户端通过 API 直接访问 HBase，实现实时计算。由于它使用的是<strong data-nodeid="1690">NoSQL，<strong data-nodeid="1689">或者说是</strong>列式结构</strong>，从而提高了查找性能，使其能运用于大数据场景，这是它跟 MapReduce 的区别。</p>
<p data-nodeid="1465">除此之外，它还有其他优点。</p>
<ul data-nodeid="1466">
<li data-nodeid="1467">
<p data-nodeid="1468"><strong data-nodeid="1700">容量大性能高</strong>。一张 HBase 表可以支持百亿行、数千列的存储，而查询效率不会有明显的变化。同时 HBase 还可以支持<strong data-nodeid="1701">高并发的读写操作</strong>。</p>
</li>
<li data-nodeid="1469">
<p data-nodeid="1470"><strong data-nodeid="1718">列存储</strong>，<strong data-nodeid="1719">无须设定表结构</strong>。对于传统数据库，比如 MySQL 是按行来存储的，检索主要依赖于事前建立的索引，在数据量很大的时候<strong data-nodeid="1720">增加列</strong>或者<strong data-nodeid="1721">更新索引</strong>都是非常缓慢的，而 HBase 每一列都是单独存储的，每一行每一列的那一个单元都是独立的存储，也就是数据本身即是索引。也因为如此，列可以在写入数据的时候动态地进行添加，而不需要在创建表的时候就设定。在具体存储时，每一行可能有不同的列。</p>
</li>
</ul>
<p data-nodeid="1471">而 HBase 不能支持条件查询，也不能用 SQL 语句进行查询。在使用的时候，一般只能使用Row&nbsp;Key 来进行查询。</p>
<h3 data-nodeid="1472">HBase 与 Hive 的使用</h3>
<p data-nodeid="1473">在实际的工作中，HBase 与 Hive 在我们的大数据架构中实际上处于不同的位置，通常搭配来进行使用。</p>
<p data-nodeid="1474">由于 HBase 支持<strong data-nodeid="1734">实时随机查询</strong>的特性，主要使用 HBase 进行大量的明细数据的<strong data-nodeid="1735">随机实时查询</strong>。比如说以用户 ID 为 Key 的用户信息，以 Itemid 为 Key 的商品信息、各种交易明细等等。在数据收集上来之后通过解析实时流然后存储到 HBase 中，以备查询。而在查询 HBase 的时候一般也是对整条数据进行查询。</p>
<p data-nodeid="1475">就我们前面已经介绍的，Hive 本身并不解决存储的问题，它只是把 HDFS 中的<strong data-nodeid="1745">结构化数据</strong>进行了展示，而最核心的功能是实现了对这些结构化文件的查询。在日常的工作中，通常使用 Hive分区表来记录一个时间段内的数据，并进行<strong data-nodeid="1746">离线的批量数据计算</strong>，比如统计分析各种数据指标。</p>
<p data-nodeid="1476">由于同为 Hadoop 体系的重要工具，Hive 与 HBase 也提供了一些访问机制。有时候我们希望能够在 Hive 中查询 HBase 的数据，可以通过关联外表的形式，在 Hive 上创建一个指向对应Hbase 表的外部表。</p>
<h3 data-nodeid="1477">总结</h3>
<p data-nodeid="1478">在这一讲中，我比较简要地介绍了 Hadoop 体系里使用比较广泛的两个工具：Hive 与 HBase。表面上，它们都与 HDFS 存储有很强的关系，但是也有非常多的不同之处。它们所实现的功能和解决的问题也有很大的区别。在这一讲中，我主要是讲了它们的基本结构、优缺点和适用情况，没有涉及具体的使用。</p>
<p data-nodeid="1479">为了更好地让你在课后进行一些使用上的探索，这里布置一个小作业：在 Hive 中，我们通常会使用分区表来按时间段对数据进行存储，那么在创建一个按天分区的 Hive 表时该如何编写创建语句呢？希望你能认真地思考下，探索中有任何问题或者心得，欢迎在评论区与我交流。</p>
<p data-nodeid="1480">下一讲让我们一起探讨为什么大公司都在做云服务，到时候见。</p>
<hr data-nodeid="1481">
<p data-nodeid="1482"><a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1756"><img src="https://s0.lgstatic.com/i/image6/M00/00/6D/Cgp9HWAaHaOAI85HAAUCrlmIuEw966.png" alt="Drawing 2.png" data-nodeid="1755"></a></p>
<p data-nodeid="1483"><strong data-nodeid="1760">《大数据开发高薪训练营》</strong></p>
<p data-nodeid="1484" class="">PB 级企业大数据项目实战 + 拉勾硬核内推，5 个月全面掌握大数据核心技能。<a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1764">点击链接</a>，全面赋能！</p>

---

### 精选评论

##### **4577：
> Hive是通过HQL对结构化数据进行的查询，请问Hbase 是通过什么，用什么样的方式对数据进行查询的呢，以及Hbase 对应的随机实时查询是理解为对实时数据的查询吗，是统计数据吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你好，Hive和HBase的实现方法其实是有本质的区别。Hive可以理解为简化了的MapReduce工具，其中实现了对已有的结构化数据查询的功能，它并不影响底层存储。而HBase可以理解为是一种数据库，它是通过对数据的存储方式的改变来实现快速查询，对于有多列的数据，HBase实际上会存储多个Key-Value数据，也就是说它是针对存储结构的改进，其中所说的实时查询是查询的速度，不是说对适时数据的查询。

##### **林：
> 想请教下大数据组件安装问题，hadoop是每个节点都需要安装来实现分布式计算和存储，hbase和hive是不是也要每个节点安装才能实现分布式存储和计算？还有flume,sqoop,kafka等组件呢？我看到教学视频上有些组件是只装在一个节点，有些组件又是每个节点都要复制一份，所以一直很疑惑

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是每个组件都需要在每个机器上安装

1、hadoop中的hdfs做存储的，要靠很多台机器一起才能存储海量数据，所以在多个节点安装
2、hadoop中的mapreduce做计算的，要靠很多台机器一起才能有效、高速的进行海量数据的计算，所以在多个节点安装
3、hbase是基于hadoop中hdfs的一个分布式海量列式非关系型数据库，需要多个节点安装
4、hive，在一个节点安装即可，可以理解为hive就是一个客户端工具，把我们写的类SQL语句翻译成MapReduce任务，提交到MapReduce集群运行，所以既然是这样一个客户端，不需要每个节点都安装，安装一个即可
5、flume 是采集日志数据的，一般，如果你的日志数据分布在多个机器上，那就在每个机器上都安装一个flume采集这个机器上的日志，如果只分布在一台机器上，那就在这个机器上安装一个就行
6、sqoop 做数据迁移的，比如mysql和hdfs之间的迁移，它可以连接某一个mysql和hdfs平台，基于一个sqoop软件写多个脚本执行多次即可，所以不用每个节点都搞一个
7、kafka 集群模式需要多个节点安装，可以理解为它就是做分布式存储的，靠多台机器存储消息数据，所以集群模式是多节点安装

##### *伟：
> 老师，想请教一下。cdh版本的hbase和Apache的hbase有什么差异呢？如果java端链接Phoenix驱动有强版本限制吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你好，CDH版本主要是解决了大数据各种工具的版本兼容问题，并做了一些优化，功能方面没有本质的区别，可以说CDH版本降低了部署的难度。第二个问题版本的适配问题可能需要查询对应版本和Java版本的兼容情况。

