<p data-nodeid="5008" class="">前面的几节课，我们了解了数据从哪里来，以及数据的存储。但是在这个过程中，有一个非常重要的问题还没有介绍，也就是数据需要借助什么工具从前端进入我们的存储环节？如果说数据就像我们的石油，在石油被开采之后，需要通过管道运输到储油罐中。这一讲，我们就来介绍一下大数据体系下承担着输油管道角色的工具——Kafka 和 Flume。</p>





<h3 data-nodeid="1542">什么是 Kafka</h3>
<p data-nodeid="1543">首先，我们来看一下 Kafka 是什么。在大数据的架构中，<strong data-nodeid="1641">数据的采集和传输是一个非常重要的环节</strong>，如何保障如此大批量的数据能够不重不漏的传输，当发生故障时如何保障数据的有效性，当网络堵塞时如何进行缓存，这需要相应的基础设施对其提供支持。</p>
<p data-nodeid="1544">我们这一讲说的是消息系统，那么你自然也知道 Kafka 也是一个消息系统。所谓的消息系统其实很简单，就是把数据从一个地方发往另外一个地方的工具。当然，只说是消息系统还不能够体现出 Kafka 的优越性，Kafka 也有其自己的特色，它是一个高吞吐量、分布式的发布 / 订阅消息系统，然而最核心的是“削峰填谷”（后续会具体讲解）。</p>
<p data-nodeid="1545">Kafka 支持多种开发语言，比如 Java、C/C++、Python、Go、Erlang、Node.js 等。现在很多主流的分布式处理系统都支持与 Kafka 的集成，比如 Spark、Flink 等。同时，Kafka 也是基于 ZooKeeper 协调管理的系统，说到这里，你可以再返回<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=615#/detail/pc?id=6517" data-nodeid="1648">《05 | 大数据开发必备工具——Hadoop》</a>中，看一下 Kafka 在大数据架构中所处的位置。</p>
<h3 data-nodeid="1546">Kafka 的结构与概念</h3>
<p data-nodeid="1547">经过了简短的介绍，我们大致可以了解 Kafka 是什么了，在 Kafka 的基本结构中，有两个重要的参与者：</p>
<ul data-nodeid="1548">
<li data-nodeid="1549">
<p data-nodeid="1550">消息的生产者（Producer）；</p>
</li>
<li data-nodeid="1551">
<p data-nodeid="1552">消息的消费者（Consumer）。</p>
</li>
</ul>
<p data-nodeid="1553">如下图所示，<strong data-nodeid="1663">Kafka 集群在消息的生产者和消费者之间建立起了联系的机制</strong>，<strong data-nodeid="1664">来保障消息的运输</strong>。生产者负责生产消息，将消息写入 Kafka 集群，而消费者从 Kafka 集群中拉取消息，也可以称为消费消息。Kafka 所要解决的就是如何来存储这些消息，如何进行集群调度实现负载均衡，如何保障通信等等问题。</p>
<p data-nodeid="1554"><img src="https://s0.lgstatic.com/i/image6/M01/10/EC/CioPOWA_MLmAGkXXAAHKYnvRFIE039.png" alt="图片1.png" data-nodeid="1667"></p>
<div data-nodeid="1555"><p style="text-align:center">Kafka 集群的联系机制</p></div>
<p data-nodeid="1556">接下来我们介绍几个 Kafka 中重要的概念。</p>
<h4 data-nodeid="1557">1.生产者与消费者</h4>
<p data-nodeid="1558">生产者负责将消息发送给 Kafka，它可以是 App，可以是服务，也可以是各种 SDK。</p>
<p data-nodeid="1559">而消费者则使用拉取的方式从 Kafka 中获取数据。</p>
<p data-nodeid="1560">每一个消费者都属于一个特定的小组（Group），同一个主题（Topic）的一条消息只能被同一个消费组下某一个消费者消费，但不同消费组的消费者可同时消费该消息。依赖这个<strong data-nodeid="1685">消费小组</strong>的理念，可以对 Kafka 中的消息进行控制，<strong data-nodeid="1686">如果需要对消息进行多个消费者重复消费</strong>，<strong data-nodeid="1687">那么就配置成多个消费组</strong>，而如果希望多个消费者共同处理一个消息源，那么就把这些消费者配置在一个消费组就可以了。</p>
<h4 data-nodeid="1561">2.消息</h4>
<p data-nodeid="1562">消息就是 Kafka 中传输的<strong data-nodeid="1694">最基本的单位</strong>，除去我们需要传输的数据，Kafka 还会给每条消息增加一个头部信息以对每一条消息进行标记，方便在 Kafka 中的处理。</p>
<h4 data-nodeid="1563">3.主题（Topic）</h4>
<p data-nodeid="1564">上面我们说到了主题，在 Kafka 中，<strong data-nodeid="1701">一个主题其实就是一组消息</strong>。在配置的时候一旦确定主题名称，生产者就可以把消息发送到某个主题中，消费者订阅这个主题，一旦主题中有数据就可以进行消费。</p>
<h4 data-nodeid="1565">4.分区（Partition）和副本</h4>
<p data-nodeid="1566">在 Kafka 中，<strong data-nodeid="1720">每个主题会被分成若干个分区</strong>。每个分区是一个有序的队列，是在物理上进行存储的一组消息，而一个分区会有若干个副本<strong data-nodeid="1721">保障数据的可用性</strong>。从理论上来说，分区数越多系统的吞吐量越高，但是这需要根据集群的实际情况进行配置，看服务器是否能够支撑。与此同时，Kafka 对消息的缓存也<strong data-nodeid="1722">受到分区和副本数量的限制</strong>。在 Kafka 的缓存策略上，一般是<strong data-nodeid="1723">按时长进行缓存</strong>，比如说存储一个星期的数据，或者按分区的大小进行缓存。</p>
<h4 data-nodeid="1567">5.偏移量</h4>
<p data-nodeid="1568">关于偏移量的概念，我们可以理解成<strong data-nodeid="1734">一种消息的索引</strong>。因为 Kafka 是一个消息队列的服务，我们不能对数据进行随机读写，而是要<strong data-nodeid="1735">按照顺序进行</strong>，所以需要给每条消息都分配一个按顺序递增的偏移量。这样消费者在消费数据的时候就可以通过制定偏移量来选择开始读取数据的位置。</p>
<p data-nodeid="1569">在我们平时的数据开发工作中，更多的往往是<strong data-nodeid="1741">作为生产者或者消费者</strong>来使用 Kafka，而不需要关注 Kafka 系统的部署和维护，感知比较明确的就是上面说到的这些概念，基本都是我们在代码中需要进行配置的。当然，在 Kafka 中还有一些基本组成部分，比如代理、ISR 以及对 ZooKeeper 的使用等，如果你对 Kafka 有更加深入的学习需求，可以进一步查看这些内容。在我们加入了上面这些组成元素之后，我们再来看一下 Kafka 的结构图。</p>
<p data-nodeid="1570"><img src="https://s0.lgstatic.com/i/image6/M01/10/EF/Cgp9HWA_MMyAN36KAAIqabLcxUQ692.png" alt="图片2.png" data-nodeid="1744"></p>
<div data-nodeid="1571"><p style="text-align:center">Kafka 的结构图</p></div>
<p data-nodeid="1572">大致的流程为，生产者生产数据，然后把数据推送到 Kafka 集群中，并确定<strong data-nodeid="1750">数据流的主题</strong>。Kafka 集群配合 ZooKeeper 集群完成调度、负载均衡、缓存等等功能，等待消费者消费数据。</p>
<h3 data-nodeid="1573">Kafka 的特点</h3>
<p data-nodeid="13530" class="">经过不断发展，Kafka 已经成为<strong data-nodeid="13536">主流的消息队列工具</strong>，类似的工具还有 RabbitMQ、Redis 消息队列、ZeroMQ、RocketMQ 等等。</p>




<p data-nodeid="1575">我们在前面说过，Kafka 最大的特色就是“<strong data-nodeid="1767">削峰填谷</strong>”，这是它在应用上的特点，这里的谷和峰指的是<strong data-nodeid="1768">数据流量的谷和峰</strong>，削峰填谷的含义即在数据生产方 A 和数据消费方 B 对数据流量的处理能力不同的时候，我们就可以使用 Kafka 作为中间传输的管道。那么在具体的设计上，Kafka 有什么特点呢，接下来让我们看一下。</p>
<h4 data-nodeid="1576">1.消息持久化</h4>
<p data-nodeid="1577"><strong data-nodeid="1774">Kafka 选择以文件系统来存储数据</strong>。很多消息系统为了提升大规模数据处理和快速传输的能力，并不会对数据进行持久化存储，或者只是缓存非常少量的数据，而 Kafka 会把数据存在磁盘上，一方面是磁盘的存储容量大，另外一方面是经过持久化的数据可以支持更多的应用，不管是实时的还是离线的，都可以进行支持。</p>
<h4 data-nodeid="1578">2.处理速度快</h4>
<p data-nodeid="1579">为了获得高吞吐量，Kafka 可是下了不少功夫。我们知道硬盘是使用<strong data-nodeid="1785">物理磁头</strong>来进行数据读写的，通常磁盘的速度都以<strong data-nodeid="1786">转数</strong>来描述，比如 5400 转、7200 转。因为随机寻址的话，需要通过转动移动到下一个地址。但是由于 Kafka 是队列的形式，创造性地对磁盘顺序读写，大大增加了磁盘的使用效率，既获得了大存储量，又提高了速度。同时 Kafka 中还加入了很多其他方面的优化，比如通过数据压缩来增加吞吐量、支持每秒数百万级别的消息。</p>
<h4 data-nodeid="1580">3.扩展性</h4>
<p data-nodeid="10438" class="">与大数据体系中的其他组件一样，Kafka 也同样支持<strong data-nodeid="10444">使用多台廉价服务器</strong>来组建一个大规模的消息系统，通过 ZooKeeper 的关联，Kafka 也非常易于进行水平扩展。</p>







<h4 data-nodeid="1582">4.多客户端支持</h4>
<p data-nodeid="1583">前面我们也提到了，Kafka 支持非常多的开发语言，比如 Java、C/C++、Python、Go、Erlang、Node.js 等。</p>
<h4 data-nodeid="1584">5. Kafka Streams</h4>
<p data-nodeid="1585">Kafka 在 0.10 之后版本中引入 Kafka Streams，能够非常好地进行流处理。</p>
<h3 data-nodeid="1586">什么是 Flume</h3>
<p data-nodeid="1587">简单介绍完了 Kafka，我们再来看一下另外一个工具 <strong data-nodeid="1806">Flume</strong>。</p>
<p data-nodeid="1588">我们先回忆一下数据采集的过程：作为前端用户使用的客户端，不管是 App、网页还是小程序，在用户使用的时候，会通过 HTTP 链接把用户的使用数据传输到后端服务器上，服务器上运行的服务把这些回传的数据<strong data-nodeid="1812">通过日志的形式保存在服务器上</strong>，而从日志到我们将数据最终落入 HDFS 或者进入实时计算服务中间还需要一些传输。</p>
<p data-nodeid="16637" class="">当然，这个过程有很多种实现方法，比如说在 Java 开发中，我们可以借助 <strong data-nodeid="16647">kafka-log4j-appender 类库</strong>把 log4j（日志记录类库）记录的日志同步到 Kafka 消息队列，由 Kafka 传输给下游任务。然而这种方式比较简陋，并不太适合大规模集群的处理，因此，这里就有一个<strong data-nodeid="16648">日志采集工具</strong>，那就是 Flume。</p>




<p data-nodeid="1590" class="te-preview-highlight"><strong data-nodeid="1832">Flume 是一个高可用</strong>、<strong data-nodeid="1833">分布式的日志收集和传输的系统</strong>。Flume 有源（Source）、通道（Channel）和接收器（Sink）三个主要部分构成。这三个部分组成一个 Agent，每个 Agent都是独立运行的单位，而 Source、Channel、Sink 有各种不同的类型，可以根据需要进行选择：</p>
<ul data-nodeid="1591">
<li data-nodeid="1592">
<p data-nodeid="1593">Channel 可以把数据缓存在内存中，也可以写入磁盘；</p>
</li>
<li data-nodeid="1594">
<p data-nodeid="1595">Sink 可以把数据写入 HBase；</p>
</li>
<li data-nodeid="1596">
<p data-nodeid="1597">HDFS 也可以传输给 Kafka 甚至是另外一个 Agent 的 Source。</p>
</li>
</ul>
<p data-nodeid="1598"><img src="https://s0.lgstatic.com/i/image6/M01/10/EC/CioPOWA_MNmAY-eQAAFmj-Mb_m4932.png" alt="图片3.png" data-nodeid="1839"></p>
<h3 data-nodeid="1599">Flume 中的概念</h3>
<h4 data-nodeid="1600">1.源（Source）</h4>
<p data-nodeid="1601">Source 是负责<strong data-nodeid="1847">接收输入数据</strong>的部分，Source 有两种工作模式：</p>
<ul data-nodeid="1602">
<li data-nodeid="1603">
<p data-nodeid="1604"><strong data-nodeid="1852">主动去拉取数据</strong>；</p>
</li>
<li data-nodeid="1605">
<p data-nodeid="1606"><strong data-nodeid="1857">等待数据传输过来</strong>。</p>
</li>
</ul>
<p data-nodeid="1607">在获取到数据之后，Source 把数据传输给 Channel。</p>
<h4 data-nodeid="1608">2.通道（Channel）</h4>
<p data-nodeid="1609">Channel 是一个<strong data-nodeid="1865">中间环节</strong>，是临时存储数据的部分，Channel 也可以使用不同的配置，比如使用内存、文件甚至是数据库来作为 Channel。</p>
<h4 data-nodeid="1610">3.接收器（Sink）</h4>
<p data-nodeid="1611">Sink 则是封装好的输出部分，选择不同类型的 Sink，将会从 Channel 中获取数据并输出到不同的地方，比如向 HDFS 输出时就使用 HDFS&nbsp;Sink。</p>
<h4 data-nodeid="1612">4.事件（Event）</h4>
<p data-nodeid="1613">Flume 中传递的一个数据单元即称为事件。</p>
<h4 data-nodeid="1614">5.代理（Agent）</h4>
<p data-nodeid="1615">正如我们前面介绍的，一个代理就是一个独立的运行单元，由 Source、Channel 和 Sink 组成，一个 Agent 中可能有多个组件。</p>
<h3 data-nodeid="1616">Kafka 与 Flume 的比较</h3>
<p data-nodeid="1617">可以看到，在<strong data-nodeid="1878">数据传输方面</strong>，Flume 和 Kafka 的实现原理比较相似，但是这两个工具有着各自的侧重点。</p>
<ul data-nodeid="1618">
<li data-nodeid="1619">
<p data-nodeid="1620">Kafka 更侧重于<strong data-nodeid="1884">数据的存储以及流数据的实时处理</strong>，是一个追求高吞吐量、高负载的消息队列。</p>
</li>
<li data-nodeid="1621">
<p data-nodeid="1622">而 Flume 则是侧重于<strong data-nodeid="1890">数据的采集和传输</strong>，提供了很多种接口支持多种数据源的采集，但是 Flume 并不直接提供数据的持久化。</p>
</li>
</ul>
<p data-nodeid="1623">就吞吐量和稳定性来说，Flume 不如 Kafka。所以在使用场景上，如果你需要在两个数据生产和消费速度不同的系统之间传输数据，比如实时产生的数据生产速度会经常发生变化，时段不同会有不同的峰值，如果直接写入 HDFS 可能会发生拥堵，在这种过程中加入 Kafka，就可以先把数据写入 Kafka，再用 Kafka 传输给下游。而对于<strong data-nodeid="1896">Flume 则是提供了更多封装好的组件</strong>，也更加轻量级，最常用于日志的采集，省去了很多自己编写代码的工作。</p>
<p data-nodeid="1624">由于 Kafka 和 Flume 各自的特点，在实际的工作中有很多是把<strong data-nodeid="1902">Kafka 和 Flume 搭配进行使用</strong>，比如线上数据落到日志之后，使用 Flume 进行采集，然后传输给 Kafka，再由 Kafka 传输给计算框架 MapReduce、Spark、Flink 等，或者持久化存储到 HDFS 文件系统中。</p>
<p data-nodeid="1625"><img src="https://s0.lgstatic.com/i/image6/M01/10/EF/Cgp9HWA_MPaAdRChAAV6_PlB9MY663.png" alt="（大数据10.png" data-nodeid="1905"></p>
<h3 data-nodeid="1626">总结</h3>
<p data-nodeid="1627">在工作中，我对 Flume 和 Kafka 最基本的感受是二者都可以用来传输数据，但是实际上各自有各自的功能和特点，在具体使用场景上有比较大的区别。所以在使用的时候，要根据具体的需求来选择使用 Kafka 还是使用 Flume，或者是把两者结合起来使用。Cloudera 甚至开发了一个新的工具——Flafka，融合了两者的特点。</p>
<p data-nodeid="1628">当然，在这一讲中我们只是对这两个工具进行了最基本的介绍，如果你想要深入理解两者的异同，还需要了解更多的细节，甚至自己动手去搭建一个 Kafka 或者 Flume 项目。在学习和实践过程中，有任何问题都可以在留言区与我交流。</p>
<p data-nodeid="1629">完成了数据传输，我们下节课就要进入数据的计算环节，来了解一下 Hadoop 体系两大核心的另外一部分：MapReduce，到时见。</p>
<hr data-nodeid="1630">
<p data-nodeid="1631"><a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1914"><img src="https://s0.lgstatic.com/i/image6/M00/00/6D/Cgp9HWAaHaOAI85HAAUCrlmIuEw966.png" alt="Drawing 2.png" data-nodeid="1913"></a></p>
<p data-nodeid="1632"><strong data-nodeid="1918">《大数据开发高薪训练营》</strong></p>
<p data-nodeid="1633" class="">PB 级企业大数据项目实战 + 拉勾硬核内推，5 个月全面掌握大数据核心技能。<a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="1922">点击链接</a>，全面赋能！</p>

---

### 精选评论

##### *峰：
> 我们一般都是通过cannal把数据传入kafka，然后在对topic里面的数据进行MapReduce

