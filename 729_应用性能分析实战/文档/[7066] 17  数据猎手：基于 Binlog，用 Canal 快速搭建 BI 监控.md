<p data-nodeid="3621" class="">这一讲我将带领你学习如何基于 Mysql 数据的 Binlog，通过阿里开源的数据同步神器 Canal 来构建无侵入高扩展的业务 BI 监控架构。</p>


<p data-nodeid="2540">业务 BI 监控对战略决策非常重要，它不仅要满足日常企业内部对业务 BI 需求，如交易系统的交易订单数据分析报表、实时的交易图表等；而且还需要通过业务 BI 监控出的数据，提前对系统做出扩缩的部署指导，实现多方面的降本提效。所以业务 BI 监控是每个企业的必要基建。</p>
<p data-nodeid="2541">在项目初期，我们可以在业务系统中通过埋点等手段进行业务 BI 监控的实现。但当业务形成规模后，若还在业务服务内部完成业务 BI 监控需求的编程，不仅会影响到业务需求的正常迭代，还会影响数据分析的性能。毕竟 BI 监控里面杂糅着很多与大数据处理相关的技术。</p>
<p data-nodeid="2542">所以本课程会以建设通用的业务 BI 监控为实践，介绍如何通过 Binlog 配制出业务 BI 数据源，以及如何通过 Canal 构建出准实时的 BI 监控方案。</p>
<h3 data-nodeid="2543">Binlog</h3>
<p data-nodeid="2544">Binlog 是 Mysql The Binary Log（二进制日志文件）的缩写，其官方介绍是：Binary Log 是 Mysql 记录其自身数据在发生变更时，Mysql 服务层以二进制形式进行存储的系统日志文件。由于并非底层实现引擎模块所记录的日志，所以具备了底层无关性和前后的兼容性。</p>
<blockquote data-nodeid="2545">
<p data-nodeid="2546">你可以点击<a href="https://dev.mysql.com/doc/refman/8.0/en/binary-log.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2664">Mysql 官方文档</a>，了解 Binlog 的更多详情。</p>
</blockquote>
<h4 data-nodeid="2547">1.Binlog 三种用途</h4>
<ul data-nodeid="2548">
<li data-nodeid="2549">
<p data-nodeid="2550">数据赋能上，有了 Binlog 的内容，通过反解析就可以反映出自身数据变更。</p>
</li>
<li data-nodeid="2551">
<p data-nodeid="2552">技术拓展上，有了 Binlog，Mysql 实现了主从复制模式，应用服务在不需要事务的需求场景下，通过使用读库可以极大地释放主库压力，进而提高集群的 TPS（事务处理的吞吐量）。</p>
</li>
<li data-nodeid="2553">
<p data-nodeid="2554">容灾能力上，误删的数据或硬件故障有了 Binlog 就有了“后悔药”，可以根据问题的时间点快速进行数据恢复。</p>
</li>
</ul>
<h4 data-nodeid="2555">2.Binlog 三种模式</h4>
<p data-nodeid="2556">常见业务 BI 监控，就是利用了 Binlog 内容可以反映业务数据变更这一能力而实现监控的。</p>
<p data-nodeid="2557">那有个问题：Mysql 中的 Binlog 一共有以下三种模式，那么应该配置哪种 Binlog 模式来实现业务 BI 监控呢？</p>
<ul data-nodeid="2558">
<li data-nodeid="2559">
<p data-nodeid="2560">Statement Level 模式</p>
</li>
<li data-nodeid="2561">
<p data-nodeid="2562">Row Level 模式</p>
</li>
<li data-nodeid="2563">
<p data-nodeid="2564">Mix Level模式</p>
</li>
</ul>
<p data-nodeid="2565">学习前，我们先思考下：应用服务在执行一条 SQL 语句时，与 MYSQL 是怎么交互的？在交互过程中，两者都有什么变化？</p>
<p data-nodeid="2566">在应用服务上设计开发需求，我们很熟悉，但对 Mysql 却用得较少（虽然或多或少有些知道）。为避免讲述得过于抽象，下面以下面订单表为例来讲解。表里面有订单号、订单状态两个字段，里面存储着两行订单数据。</p>
<table data-nodeid="4056">
<thead data-nodeid="4057">
<tr data-nodeid="4058">
<th align="center" data-nodeid="4060">订单号</th>
<th data-nodeid="4061">订单状态</th>
</tr>
</thead>
<tbody data-nodeid="4064">
<tr data-nodeid="4065">
<td align="center" data-nodeid="4066">NO1</td>
<td data-nodeid="4067">草稿</td>
</tr>
<tr data-nodeid="4068">
<td align="center" data-nodeid="4069">NO2</td>
<td data-nodeid="4070">等待支付</td>
</tr>
</tbody>
</table>

<p data-nodeid="2585">当应用服务通过数据库驱动，对 Mysql 数据库发出以下执行命令后，会收到 Mysql 数据库执行成功的响应，影响数据的条数为两条。</p>
<pre class="lang-java" data-nodeid="2586"><code data-language="java">update order set `订单状态` = <span class="hljs-string">'支付完成'</span>;
</code></pre>
<p data-nodeid="2587">从 Mysql 数据库服务器的视角，看下整体的日志可以记录为以下两部分。</p>
<ul data-nodeid="2588">
<li data-nodeid="2589">
<p data-nodeid="2590"><strong data-nodeid="2690">数据变化日志</strong>：日志的内容就是两条订单记录在语句执行前后的数据的变化情况。比如内容有订单号 NO1 的数据记录，订单状态由“草稿状态”修改为“支付完成状态”，日志内容就是当前时刻 NO1 的数据快照。</p>
</li>
<li data-nodeid="2591">
<p data-nodeid="2592"><strong data-nodeid="2695">执行状况日志</strong>：日志内容就是 Mysql 数据库服务器接收到的 update 执行语句，以及执行情况的记录。</p>
</li>
</ul>
<p data-nodeid="2593">Binlog 的三种模式就是让用户告诉 Mysql 数据库服务器，如何记录数据变化日志和执行情况日志。</p>
<h4 data-nodeid="2594">3.Binlog 三种模式</h4>
<ul data-nodeid="2595">
<li data-nodeid="2596">
<p data-nodeid="2597"><strong data-nodeid="2701">Statement Level 模式</strong></p>
</li>
</ul>
<p data-nodeid="2598">Mysql 默认使用 Statement Level 模式来记录 Binlog 内容。在此模式下，每一条执行数据库的语句都会被记录到 Binlog 中，而此时 Binlog 内容是最小的，所以节约了 I/O 成本，整体性能就是最优的。</p>
<p data-nodeid="2599">当然任何事情都是两面性的，这种模式下主从数据的准确性也是最低的，比如在某些版本使用存储的过程中，会出现插入数据的数据 ID 不一致等问题。</p>
<ul data-nodeid="2600">
<li data-nodeid="2601">
<p data-nodeid="2602"><strong data-nodeid="2707">Row Level 模式</strong></p>
</li>
</ul>
<p data-nodeid="2603">相比 Statement Level 模式，Row Level 模式又走向了另一个极端。在 Row Level 模式下，Mysql 数据库服务器的 Binlog 内容会记录，在执行 SQL 语句前后，详细的记录哪一行修改了，以及修改成什么样子。</p>
<p data-nodeid="2604">由此看来，Row Level 模式下的问题也显而易见。比如，上文中我们执行的订单表执行的更新语句，当订单表的数据量达到海量时，执行更新语句后，Binlog 的数据会瞬间变成海量，会造成下游订阅此 Binglog 的应用服务消费不及时。但是事物是具有两面性的，通用的 BI 监控就是通过严控这种问题的发生而建设的。</p>
<ul data-nodeid="2605">
<li data-nodeid="2606">
<p data-nodeid="2607"><strong data-nodeid="2713">Mix Level 模式</strong></p>
</li>
</ul>
<p data-nodeid="2608">Mix Level 是两种模式的混合模式，并没有常见的使用场景。如果你感兴趣，可以访问官网进行更详细的学习，这里就不过多介绍了。</p>
<p data-nodeid="2609">通过对这三个模式的学习可知，BI 监控需要实时分析和处理业务数据，所以被处理的数据源必须是完整的数据，故此我们需要将 Mysql 的 Binlog 模式配置成 Row Level 模式。</p>
<h3 data-nodeid="2610">Canal</h3>
<p data-nodeid="2611">Mysql 的主从复制就是通过 Binlog 实现，从节点通过读取主库的 binlog，实现主节点和从节点的数据准实时同步。Canal 可以简单理解为将自己伪装成从节点，定时向主节点发送 Dump 请求，拉取和分析 Row Level 模式下的 Binlog 数据。</p>
<h4 data-nodeid="2612">1.主从复制</h4>
<p data-nodeid="2613">主从复制是实现数据准实时同步的通用架构。其原理也非常清晰，它是后世很多数据库中间件的基本原理，如 Canal 基于 Binlog 实现数据同步的设计基石（如下图所示）：</p>
<ul data-nodeid="2614">
<li data-nodeid="2615">
<p data-nodeid="2616">Master 和 Slave 节点组成了 Mysql 数据库集群；</p>
</li>
<li data-nodeid="2617">
<p data-nodeid="2618">Master 节点提供数据更改的能力，而 Slave 节点只提供数据读取的能力；</p>
</li>
<li data-nodeid="2619">
<p data-nodeid="2620">在 Slave 节点拉起时，向 Master 节点拉取带时间戳的数据快照；</p>
</li>
<li data-nodeid="2621">
<p data-nodeid="2622">然后内部存在两个类型的线程，I/O 线程负责拉取 Binlog 文件；</p>
</li>
<li data-nodeid="2623">
<p data-nodeid="2624">SQL 线程负责读取并执行 Relay Log 中的内容。</p>
</li>
</ul>
<p data-nodeid="2625">通过以上几个步骤，实现了主从同步架构。</p>
<p data-nodeid="4505" class=""><img src="https://s0.lgstatic.com/i/image6/M01/40/6F/Cgp9HWCk4g2ARr8vAAUdkIbJM8c153.png" alt="Drawing 0.png" data-nodeid="4508"></p>

<h4 data-nodeid="2627">2.Canal 工作原理</h4>
<p data-nodeid="2628">Canal 是阿里开源 Java 语言项目，基于数据库 Binlog 日志解析，提供增量数据订阅 &amp; 消费的能力，目前主要支持国内最常用到的关系型数据库 MySQL。</p>
<p data-nodeid="2629">Canal 的工作原理也不复杂（如下图所示）：</p>
<ul data-nodeid="2630">
<li data-nodeid="2631">
<p data-nodeid="2632">Canal 伪装自己成为 Mysql 数据库集群中的 Slave 节点，向 Master 节点发送对 Binlog 的 dump 协议；</p>
</li>
<li data-nodeid="2633">
<p data-nodeid="2634">Mysql Master 节点收到 dump 请求后，将数据推送给 Canal；</p>
</li>
<li data-nodeid="2635" class="">
<p data-nodeid="2636">Canal 解析 Mysql Binlog 数据发送给下游。</p>
</li>
</ul>
<p data-nodeid="4941" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/40/78/CioPOWCk4hOAbZVeAAHP3TFA6b4794.png" alt="Drawing 1.png" data-nodeid="4944"></p>

<p data-nodeid="2638">Canal 对比 Mysql 的从节点最大的不同，就是拉起时不需要预先同步一个时间点的数据快照。Canal 可以对接的下游有很多，最简单直接的方式是通过 Canal 将增量数据转化为 Elasticsearch 搜索引擎的时序索引。然后根据我们前面课程所讲述的，通过 Grafana 或是 Kibana 等可视化套件，将 BI 数据展示出来。</p>
<p data-nodeid="2639">这种架构可以快速构建有关时序需求的增量 BI 监控，且对业务完全无侵入。但这种方式也过于“粗暴”，只能填补增量 BI 监控的空缺。</p>
<p data-nodeid="2640">所以，接下来讲的就是通过对接消息队列下游，实现更好扩展的 BI 监控。</p>
<h4 data-nodeid="2641">3.基于消息队列的 BI 监控</h4>
<p data-nodeid="2642">部分业务的 BI 监控往往二次开发才能完成，所以高扩展性是企业落地 BI 监控的重要指标之一。如上面所讲，通用的业务 BI 监控方案无法满足时，需要有足够的开发性和扩展性，让一线的业务开发有兜底的实现方案。</p>
<p data-nodeid="2643">那 Canal 就是具备这样的架构。通用的方案下，将增量数据准实时的同步到数据库介质中，实现增量数据的业务 BI 监控分析；在通用方案不满足时，Canal 支持将 Binlog 数据投递到消息队列中，如 Kafka 和 RocketMQ 中，一线业务开发人员可以主动对 Binlog 数据进行二次消费，完成业务 BI 监控专项的二次开发来完成需求。</p>
<h3 data-nodeid="2644">小结与思考</h3>
<p data-nodeid="2645">今天的课程，我带你学习了<strong data-nodeid="2748">如何设计通用的业务 BI 监控方案</strong>，通过将关系型数据库 Mysql 的 Binglog 配置 Row Level 模式，将数据库中数据变更内容反映到 Binglog 内容中。</p>
<p data-nodeid="2646">需要注意的是，Row Level 模式下进行一些非常规操作（如全表更新，赠加减字段等）时，就会出现 I/O 瓶颈，也就会出现相关性能负载过高的情况，如磁盘 I/O 过高，主从延迟时间过长等问题。</p>
<p data-nodeid="2647">之后又讲到了 Mysql 主从复制架构，Slave 节点通过 I/O 线程持续向 Master 节点拉取 Binlog 数据，SQL 执行线程持续分析和执行这些 Binlog 数据，最终实现主从复制架构。</p>
<p data-nodeid="2648">那 Canal 的设计也是基于主从复制机构，Canal 将自己伪装成 Slave 节点，也不停向 Master 节点模拟发送 Dump 协议请求，将得到的 Binlog 数据同步到下游消费者。</p>
<ul data-nodeid="2649">
<li data-nodeid="2650">
<p data-nodeid="2651">简单业务 BI 监控场景，可以使用 Canal 直接将增量数据同步到 Elasticsearch，然后通过可视化套件 Grafana 或 Kibana 实现 BI 监控的数据展示。</p>
</li>
<li data-nodeid="2652">
<p data-nodeid="2653">相对复杂的业务 BI 监控场景，可以将增量数据消费到消息队列后，进行专项的业务 BI 监控的二次开发。总之是要实现与业务系统的解耦，让业务系统专注于业务需求的实现，让 BI 监控需求有专门的承接系统。</p>
</li>
</ul>
<p data-nodeid="2654">那么你在工作中是如何实现业务 BI 监控的呢？欢迎在评论区写下你的思考，期待与你的讨论。</p>

---

### 精选评论


