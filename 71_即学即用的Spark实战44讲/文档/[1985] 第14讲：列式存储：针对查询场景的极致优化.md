<p>在前 2 个课时，我们学习了如何用 DataFrame + SQL 的方式对数据进行分析与处理，需要实践的内容比较多，学习起来未免比较辛苦，那么本课时我们来聊聊列式存储这个比较轻松又实用的话题。在本课时的标题中，提到了查询场景和极致优化，也就是说，如果你的业务场景只是查询（这意味着没有增删改），那么列式存储将带来极其可观的性能提升。</p>
<p>本课时的主要内容有：</p>
<ul>
<li>Google Dremel</li>
<li>列式存储的实现</li>
<li>对比测试</li>
</ul>
<h3>Google Dremel</h3>
<p>Google 在 2004-2006 年期间发表了著名的“三驾马车”论文，开启了大数据时代。在 2010 年，Google 又发表了 3 篇论文，被称为 Google 的“新三驾马车”，可见其分量之重，其中一篇《Dremel: Interactive Analysis of Web-Scale Datasets》，提出了<strong>列式存储与多级执行树</strong>，文中介绍了 Google 运用 Dremel 分析来自互联网的千亿条级别数据的实践。</p>
<p>与论文中提出的列式存储相比，行式存储可以看成是一个行的集合，其中每一行都要求对齐，哪怕某个字段为空（下图中的左半部分），而列式存储则可以看成一个列的集合（下图中的右半部分）。列式存储的优点很明显，主要有以下 4 点：</p>
<ol>
<li>查询时可以只读取涉及的列（选择操作），并且列可以直接作为索引，非常高效，而行式存储则必须读入整行。</li>
<li>列式存储的投影操作非常高效。</li>
<li>在数据稀疏的情况下，压缩率比行式存储高很多，甚至可以考虑将相关的表进行预先连接，来完全避免投影操作。</li>
<li>因为可以直接作用于某一列上，聚合分析非常迅速。</li>
</ol>
<p>行式存储一般擅长的是插入与更新操作，而列式存储一般适用于数据为只读的场景。对于结构化数据，列式存储并不陌生。因此，列式存储技术经常用于传统数据仓库中。下图分别展示了行式存储和列式存储的区别。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/C0/CgqCHl7Ph4GAVimdAACchbaqy4s806.png" alt="1.png"></p>
<p>在文章中，Dremel 在一开始就指出其面对的是只读的嵌套数据，而嵌套数据属于半结构化数据，例如 JSON、XML，所以 Dremel 的创新之处在于提出了一种支持嵌套数据的列式存储，而如今互联网上的数据又正好多是嵌套结构。下图左边是一个嵌套的 Schema，而右边的 r1、r2 为两条样例记录：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/B5/Ciqc1F7Ph42AOmA5AABpX1CjdEQ756.png" alt="2.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/C0/CgqCHl7Ph5KATxn8AAA-f556w0A361.png" alt="3.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/C0/CgqCHl7Ph5iAaJMJAAAbuHEozWo871.png" alt="4.png"></p>
<p>这个 Schema 其实可以转换为一个树形结构，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/C0/CgqCHl7Ph6KAPRaRAAJLU-Fypgc559.png" alt="5.png"></p>
<p>该树结构有 6 个叶子节点，可以看到叶子节点其实就是 Schema 中的基本数据类型，如果将这种嵌套结构的数据展平，那么展平后的表应该有 6 列。如果要应用列式存储来存储这种嵌套结构，还需要解决一个问题，我们看到 r1、r2 的数据结构还是差别非常大，所以需要标识出哪些列的值组成一条完整的记录，但我们不可能为每条记录都维护一个树结构。Google 提出的&nbsp;<a href="https://github.com/Parquet/parquet-mr/wiki/The-striping-and-assembly-algorithms-from-the-Dremel-paper">record shredding and assembly algorithm</a> 算法很好地解决了这个问题，该算法规定，在保存字段值时，还需要额外存储两个数字，分别表示 Repetition level（r）和 Definition level（d）其中，Repetition level 值记录了当前值属于哪一条记录以及它处于该记录的什么位置；另外对于 repeated 和 optional 类型的列，可能一条记录中某一列是没有值的，如果不进行标识就会导致本该属于下一条记录的值被当作当前记录的一部分，对于这种情况就需要用 Definition level 来标识这种情况，通过 Striping &amp; Assembly 算法我们可以将一整条记录还原出来，如下图所示。这样就能用尽可能少的存储空间来表达复杂的嵌套数据格式了。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/B5/Ciqc1F7Ph6uAAXKbAAFlSOVIvZU140.png" alt="6.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/C0/CgqCHl7Ph7KALbsvAAESylp2B9o320.png" alt="7.png"></p>
<p>Dremel 的另外一个组成部分是查询执行树，利用这种架构，Dremel 可以用很低的延迟分析大量数据，这使得 Dremel 非常适合进行交互式分析。Dremel 有很多种开源实现，与 Dremel 一样，它主要分为两部分，一个实现了 Dremel 的嵌套列式存储，如 Apache Parquet、Apache ORC，还有一些实现了 Dremel 的查询执行架构，也就是多级执行树，如 Apache Impala、Aapche Drill 与 Presto。</p>
<p>这里要特别说明的是，多级执行树这种技术与 Spark 这种 MapReduce 类型的计算框架完全不同，它类似于一种大规模并行处理，希望以较低的延迟完成查询，所以并行程度要远远大于 Spark，但是每个执行者的性能要远远弱于 Spark。如果把 Spark 看成是对 CPU 核心的抽象，那么多级执行树可以看成是对线程的抽象。基于此，多级执行树 + 列式存储的组合往往用于 OLAP 的场景。</p>
<p>Parquet 和 ORC 这两种数据格式和 Json 一样都是自描述数据格式，Spark 很早就支持由 Parquet、ORC 格式的数据直接生成 DataFrame。**在课时 12 中曾讲到，我们可以非常方便地通过 read 读取器和 write 写入器读取和生成 Parquet 和 ORC 文件。**列式存储在选择、投影操作的性能优化提升非常明显，此外，Dremel 的高压缩比率也对 Spark 这种 I/O 密集型作业非常友好。在目前 Hadoop、Spark 体系的数据仓库中，已经很少采用 CSV、TEXT 这种格式了。</p>
<h3>列式存储的实现</h3>
<h4>Apache Parquet</h4>
<p>Apache Parquet 是 Dremel 的开源实现，它最先是由 Twitter 与 Cloudera 合作开发并开源，和 Impala 配合使用。Parquet 支持几乎 Hadoop 生态圈的所有项目，与数据处理框架、数据结构以及编程语言无关。</p>
<h4>Apache ORC</h4>
<p>Apache ORC（OptimizedRC file）来源于 RC（RecordColumnar file）格式，但目前已基本取代 RC 格式。ORC 提供 ACID 支持、也提供不同级别的索引，如布隆过滤器、列统计信息（数量、最值等），和 Parquet 一样，它也是自描述的数据格式，但与 Parquet 不同的是，ORC 支持多种复杂数据结构，如集合、映射等。ORC 与 Presto 配合使用，效果非常好。</p>
<h4>Apache CarbonData</h4>
<p>CarbonData 是华为开源的一种列式存储格式，是专门为海量数据分析和处理而生的。CarbonData 于 2016 年开源，目前发展非常迅猛，与 Apache Kylin 并列为由国人主导的两个Apache 顶级项目。它的设计初衷源于，在很多时候，对于同样一份数据，处理方式是不同的，比如以下几种处理方式：</p>
<ul>
<li>
<p>全表扫描，或者选取几列进行过滤；</p>
</li>
<li>
<p>随机访问，如行键值查询，要求低延迟；</p>
</li>
<li>
<p>ad-hoc 交互式分析，如多维聚合分析、上卷、下钻、切片等。</p>
</li>
</ul>
<p>不同的处理方式对于数据格式的需求侧重点是不同的，但 CarbonData 旨在为大数据多样化的分析需求提供一种统一的数据格式。CarbonData 的设计目标为：</p>
<ul>
<li>支持低延迟访问多种数据访问类型；</li>
<li>允许在压缩编码过的数据上进行快速查询；</li>
<li>确保存储空间的高效性；</li>
<li>很好地支持 Hadoop 生态系统；</li>
<li>读最优化的列式存储；</li>
<li>利用多级索引实现低延迟；</li>
<li>支持利用列组来获得基于行的优点；</li>
<li>能够对聚合的延迟解码进行字典编码。</li>
</ul>
<p>如下图所示，这是一个 CarbonData 数据文件，也是 HDFS 上的一个数据块，每个文件由 File Header、File Footer 与若干个 Blocklet 组成，其中 File Header 保存了文件版本号、Schema 以及更新时间戳；File Footer 包含了一些统计信息（每个 Blocklet 的最值）、多维索引等。一个 Blocklet 的默认大小为 64MB，包含多个 Column Page Group，Blocklet 可以看成一个表的水平切片，这个表有多少列，就有多少个 Column Page Group，在一个 Column Page Group 中，一列被分为若干个连续文件，每一个文件被称为 Page，一个 Page 默认为 32000 行，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/13/B5/Ciqc1F7Ph72AcWNZAAP4By_8nu8265.png" alt="8.png"></p>
<p>这里要特别说明的是，CarbonData 在设计理念上没有采取 Dremel 提出的嵌套的列式存储，而是引入了索引和元数据的设计，但仍然属于列式存储格式。</p>
<h3>对比测试</h3>
<p>使用列式存储对 Spark 性能提升的影响是非常巨大的，下面是一份测试结果，包含了对于同样一份数据（368.4G），各种数据格式压缩率的对比，以及一些计算作业耗时的对比：</p>
<table>
<thead>
<tr>
<th align="left"></th>
<th align="left">TEXT</th>
<th align="left">Parquet</th>
<th align="left">ORC</th>
<th align="left">CarbonData</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">压缩后大小</td>
<td align="left">368.4G</td>
<td align="left">298G</td>
<td align="left">148.4G</td>
<td align="left">145.8G</td>
</tr>
<tr>
<td align="left">压缩率</td>
<td align="left">100%</td>
<td align="left">19.11%</td>
<td align="left">59.72%</td>
<td align="left">60.42%</td>
</tr>
</tbody>
</table>
<table>
<thead>
<tr>
<th align="left"></th>
<th align="left">TEXT</th>
<th align="left">Parquet</th>
<th align="left">ORC</th>
<th align="left">CarbonData</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">Count</td>
<td align="left">67s</td>
<td align="left">116s</td>
<td align="left">119s</td>
<td align="left">6s</td>
</tr>
<tr>
<td align="left">Group By</td>
<td align="left">138s</td>
<td align="left">75s</td>
<td align="left">71s</td>
<td align="left">92s</td>
</tr>
<tr>
<td align="left">Join</td>
<td align="left">231s</td>
<td align="left">172s</td>
<td align="left">140s</td>
<td align="left">95s</td>
</tr>
</tbody>
</table>
<p>可以看到 ORC 的压缩率最高，而 CarbonData 在 Spark 批处理这种场景下，性能表现得非常好，是一种非常有前景的技术。</p>
<p>列式存储的压缩率如此之高，从本课时的第一张图也可以看出原因，列式存储作为列的集合，空间几乎没有多余的浪费。如此高的压缩效率也带来了一个优化思路：可以将若干相关的表预先进行连接，连接而成的表可以看成是一张稀疏的宽表，这张宽表对分析来说就非常友好了，但由于采用了列式存储，所以宽表所占的空间并不是指数上涨而是线性增加。<strong>在这种场景下，列式存储使得空间换时间成为可能。</strong></p>
<h3>小结</h3>
<p>目前，列式存储在数据分析领域非常火，比如最近大热的俄罗斯开源列式分析数据库 ClickHouse。Spark 在很早就支持列式存储，而列式存储的使用带来的性能提升是十分巨大的，至于选取哪种列式存储，你可以根据具体的性能表现与存储空间综合进行考虑，不过 ORC 这种格式在绝大多数场景都能胜任。</p>
<p>这里给你留一个思考题，为什么 CarbonData 的 count 性能会远远超过 Dremel 系列的技术，如 ORC 和 Parquet 呢？大家可以从本文中找到答案。</p>

---

### 精选评论

##### *鑫：
> <span style="color: rgb(73, 73, 73); font-family: -apple-system-font, &quot;Helvetica Neue&quot;, sans-serif; font-size: 16px; text-align: justify;">因为CarbonData的 File Footer 包含了一些统计信息（每个 Blocklet 的最值）、多维索引等</span><div style="font-size: 16.0125px;"><span style="color: rgb(73, 73, 73); font-family: -apple-system-font, &quot;Helvetica Neue&quot;, sans-serif; font-size: 16px; text-align: justify;">count的时候也就节省了一些时间&nbsp; 所以快</span></div>

##### *强：
> File Footer 包含了一些统计信息（每个 Blocklet 的最值）、多维索引等

##### **成：
> 因为flie footer里面存放了行数，所以count操作才这么快。

