<p data-nodeid="46159">在上个课时里，我们已经成功构建了数据立方体，在结果呈现之前只剩下最后两步：多维分析和结果可视化。在一个商业智能系统中，最后两步是呈现给客户的关键，如果是自己开发，需要大量的工作。在这个课时里，我们将介绍两种开源技术 Superset 和 Presto，以满足实时分析和可视化的需求。其中 Presto 可以用来完成我们对数据立方体进行多维分析的需求，而 Superset 则可以用来将 Presto 返回的结果进行可视化呈现。两者之间本身也能很好地集成，非常适合我们的项目。</p>
<h3 data-nodeid="46160">Presto</h3>
<p data-nodeid="46161">Presto 是一种用于大数据的高性能分布式SQL查询引擎，其架构允许用户查询各种数据源，如 Hadoop、AWS S3、Alluxio、MySQL、Cassandra、Kafka 和 MongoDB。你甚至可以在单个查询中查询来自多个数据源的数据。</p>
<p data-nodeid="46162">Presto 是Apache 许可证下发布的社区驱动的开源软件。它是 Dremel 的实现，与 ORC 配合通常会有非常好的效果。从前面的课时内容可以得知，得益于 Dremel 架构，Presto 是非常好的 OLAP 与 Ad-hoc 查询工具，并且支持 JDBC。</p>
<p data-nodeid="47571" class="">它最初是Facebook为数据分析师设计和开发的，用于在 Apache Hadoop 中的大型数据仓库上运行交互式查询。这意味着，它可以直接对 HDFS 上的数据进行分析，这是一个非常有用的特性。联想到数据立方体，可以发现这是一个逻辑的概念。在大数据概念出现以前，数据立方体通常通过事实表 + 维度表，也就是星型模型的方式呈现（如“第 38 课时 | 数据仓库与商业智能系统架构剖析”中的表格所示）。这种方式的数据冗余性很低，但是每次进行分析都需要连接操作，可以说是用时间换空间。</p>


<p data-nodeid="46164">但是有了列式存储以后，这个问题迎刃而解，<strong data-nodeid="46228">虽然数据冗余性很高，但实际占用空间并不大</strong>，对于后续的多维分析来说又非常友好，无疑比星型模型更优。另外，<strong data-nodeid="46229">Presto 可以直接操作 HDFS 上的数据</strong>，也就无需将数据立方体再次导出，少了很多工作量。而传统的数据集市往往基于数据库，这样就需要将数据再次导出到数据库中。</p>
<p data-nodeid="46165">此外，还有一些需要简要了解的内容。</p>
<p data-nodeid="46166">Presto 的架构与使用集群计算（MPP）的传统数据库管理系统非常类似，它可以视为一个协调器节点，与多个工作节点同步工作。客户端提交已解析和计划的 SQL 语句，然后将并行任务安排给工作节点。工作节点一同处理来自数据源的行，并生成返回给客户端的结果。同在每个查询上使用 Hadoop 的MapReduce机制的原始 Apache Hive 执行模型相比，Presto 不会将中间结果写入磁盘，从而明显提高速度。</p>
<p data-nodeid="46167">Presto 是用Java 语言编写的，单个 Presto 查询可以组合来自多个源的数据。Presto 提供数据源的连接器，包括 Alluxio、Hadoop 分布式文件系统、Amazon S3 中的文件、MySQL、PostgreSQL、Microsoft SQL Server、Amazon Redshift、Apache Kudu、Apache Phoenix、Apache Kafka、Apache Cassandra、Apache Accumulo、MongoDB 和 Redis。与其他只支持 Hadoop 特定发行版的工具（如 Cloudera Impala）不同，Presto 可以使用任何风格的 Hadoop，也可以不用 Hadoop。它支持计算和存储的分离，可以在本地和云中部署。</p>
<p data-nodeid="46168">由于 Presto 支持 JDBC，所以在使用上与任何支持 JDBC 的 SQL 工具来说并没有任何不同，这点我们在后面可以看到。</p>
<h3 data-nodeid="46169">Superset</h3>
<p data-nodeid="46170">Superset 是由 Airbnb 开源出来的企业级商业智能工具，目前属于 Apache 孵化器，但是其关注度已经超越了很多 Apache 顶级项目。简单来说，Superset 主要提供以下几方面的功能：</p>
<ul data-nodeid="46171">
<li data-nodeid="46172">
<p data-nodeid="46173">与数据源和 OLAP 工具进行集成。</p>
</li>
<li data-nodeid="46174">
<p data-nodeid="46175">配置数据立方体。</p>
</li>
<li data-nodeid="46176">
<p data-nodeid="46177">Ad-hoc 查询。</p>
</li>
<li data-nodeid="46178">
<p data-nodeid="46179">以仪表盘和卡片的形式提供的可视化解决方案。</p>
</li>
<li data-nodeid="46180">
<p data-nodeid="46181">权限管理。</p>
</li>
</ul>
<p data-nodeid="46182">下面结合本项目对 Superset 进行介绍。</p>
<p data-nodeid="48181">首先登录 Superset 应用：</p>
<p data-nodeid="48182" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/20/CgqCHl9QpNGAQf6kAAAuk0cKAiA356.png" alt="Drawing 0.png" data-nodeid="48186"></p>


<p data-nodeid="48795">登录后进入仪表盘页面，如下图：</p>
<p data-nodeid="48796" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/20/CgqCHl9QpNiAHMXGAABZ4RVwLHo758.png" alt="Drawing 1.png" data-nodeid="48800"></p>


<p data-nodeid="49401">可以看到这里还没有一个存在的仪表盘，我们先不进行创建，而是进入配置<strong data-nodeid="49408">数据源</strong>的页面，点击 Source 选项卡，选择 Database，并新增一条记录，如下图：</p>
<p data-nodeid="49866" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpOCAY2WAAABANmHLBIo256.png" alt="Drawing 2.png" data-nodeid="49869"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpOWAFJtIAABwTrPWtDQ394.png" alt="Drawing 3.png" data-nodeid="49873"></p>





<p data-nodeid="50482">此时，通过 Presto 的 JDBC 就能非常容易地将 Presto 与 Superset 进行集成，在红色方框内填入 JDBC URL，如下图：</p>
<p data-nodeid="50483" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpO-AG-UNAADnzM9NcRI985.png" alt="Drawing 4.png" data-nodeid="50487"></p>


<p data-nodeid="51096">配置完成数据源后，我们就可以进入到 SQL Lab 对数据进行查询了，如下图：</p>
<p data-nodeid="51097" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpPaAFzZTAABrVVz4RoM970.png" alt="Drawing 5.png" data-nodeid="51101"></p>


<p data-nodeid="51710">在 SQL Lab 中，我们可以对数据源进行探索，也就是前面所说的 Ad-hoc 查询，如下图：</p>
<p data-nodeid="51711" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpP-AEILZAAEXCRAIoA8226.png" alt="Drawing 6.png" data-nodeid="51715"></p>


<p data-nodeid="46196">在对 Superset 有了一个大致了解后，下面学习 Superset 最重要的功能：<strong data-nodeid="46284">仪表盘（Dashboard）<strong data-nodeid="46283">和卡片</strong>（Chart）</strong>。</p>
<h4 data-nodeid="52945" class="">仪表盘和卡片</h4>




<p data-nodeid="53539">仪表盘是商业智能系统面向用户的最终产物，是用户查看一组报表分析结果的地方，如下图：</p>
<p data-nodeid="53540" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpQyAeubkAALHmtTBzLc454.png" alt="Drawing 7.png" data-nodeid="53544"></p>


<p data-nodeid="46200">上图的仪表盘里有 4 个报表，每个报表是一个卡片。卡片与仪表盘是灵活的多对多关系，可以对不同的仪表盘进行复用，非常方便。</p>
<p data-nodeid="54137">我们先创建好一个仪表盘，即来到 Dashboard 页面，新建一个 Dashboard，如下图：</p>
<p data-nodeid="54138" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/20/CgqCHl9QpRWAazcLAABgZu9ngT8954.png" alt="Drawing 8.png" data-nodeid="54142"></p>


<p data-nodeid="55039">接着生成卡片，从 SQL Lab 的 Explore 按钮开始：</p>
<p data-nodeid="55040" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/20/CgqCHl9QpRyAFMghAADXUvm29h4614.png" alt="Drawing 9.png" data-nodeid="55044"></p>



<p data-nodeid="46205">生成过程也就是 Superset 多维分析的实现，所以，这里需要特别注意的是，在点击 Explore 之前，文本框中 SQL 查询的结果（不需要执行）代表数据立方体。也就是说，不管你是否预先构建好了数据立方体，这里都需要用 SQL 进行描述。对于本课程中的项目来说，当然非常简单，只需要一个普通的 SELECT 查询即可，如果没有预先构建立方体，这里可能就需要进行连接操作。</p>
<p data-nodeid="55637">点击 Explore 后，会进入多维分析的配置页面，如下图：</p>
<p data-nodeid="55638" class=""><img src="https://s0.lgstatic.com/i/image/M00/4A/15/Ciqc1F9QpSOAEA26AAFBEWC_S-w029.png" alt="Drawing 10.png" data-nodeid="55642"></p>


<p data-nodeid="46208">在红色方框内，我们可以按照前面介绍的多维分析方法选择要分析的维度、统计的指标，例如求和、求均值、求最值等。配置完成后，系统会自动匹配可视化方案，如果没有时间序列，一般用饼图，如果有时间序列，则用柱状图或者折线图。点击Save 按钮，就可以将其保存为一个卡片，<strong data-nodeid="46313">在保存的时候，可以选择与 MyYelp 仪表盘进行关联</strong>。我们可以按照这种方式，生成卡片并将其以仪表盘的形式呈现给用户，自此，整个商业智能系统就算完成了。</p>
<h3 data-nodeid="46209">总结</h3>
<p data-nodeid="46210">本课时主要介绍了两个开源工具 Presto 与 Superset，这两个工具对整个系统的完成度提升来说，效果很大，但如果自己要实现 Superset 这种 BI 工具，可想而知工作量是巨大的。另外， Superset 除了支持常规的 JDBC 的 SQL 工具外，还支持很多开源的分析工具，如 Kylin、Druid 等。最后要说明的是，本课时选用 Presto，并不是因为其查询性能快，而是因为它可以直接操作 HDFS 上的数据。</p>

---

### 精选评论

##### **佳：
> presto 与spark sql 有何异同？ 听起来presto支持的sql即时查询 ，spark sql 也都可以，并且都是基于内存并行计算。另外当presto连接传统关系数据库时 查询语句是由谁来执行？是数据库执行 而presto仅做一层包装或者扩展实现一些底层数据库不支持的语法？ 如果查询完全又presto执行那么它是如何绕过数据库引擎的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题问得很好，建议阅读 Dremel 这篇论文，相信能够得到满意的答案。

