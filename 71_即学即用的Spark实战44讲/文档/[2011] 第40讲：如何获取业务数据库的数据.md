<p data-nodeid="56392">在前面的内容中，我们特意说过在现实情况中，数据并不会被轻易获取，通常数据都在业务数据库中，需要将其抽取，进行下一步的转换操作。这一步在真实环境中会花费大量时间，尤其是数据量特别大的情况，因为通常会涉及巨量的读写性能消耗。</p>
<p data-nodeid="56393">在不同的业务场景下，业务数据库通常会有不同的选择，主要分为两类：关系型数据库和NoSQL 数据库。</p>
<ul data-nodeid="56394">
<li data-nodeid="56395">
<p data-nodeid="56396"><strong data-nodeid="56463">关系数据库</strong>：是创建在关系模型基础上的数据库，借助集合代数等数学概念和方法来处理数据库中的数据。现实世界中的各种实体以及实体之间的各种联系均用关系模型来表示。关系型数据库主要使用 SQL 作为自己的查询语言。</p>
</li>
<li data-nodeid="56397">
<p data-nodeid="56398"><strong data-nodeid="56468">NOSQL（Not Only SQL）</strong>：是对不同于传统关系数据库的数据库管理系统的统称。该系统允许部分数据使用 SQL 系统存储，允许其他数据使用 NOSQL 系统存储。其数据存储可以不需要固定的表格模式以及元数据，也经常会避免使用 SQL 的连接操作，一般有分布式、高可用、高可扩展的特征。</p>
</li>
</ul>
<p data-nodeid="56399">本课时的主要内容有：</p>
<ul data-nodeid="56400">
<li data-nodeid="56401">
<p data-nodeid="56402">项目架构</p>
</li>
<li data-nodeid="56403">
<p data-nodeid="56404">关系型数据库的数据导出</p>
</li>
<li data-nodeid="56405">
<p data-nodeid="56406">NoSQL 数据库的数据导出</p>
</li>
</ul>
<h3 data-nodeid="56407">项目架构</h3>
<p data-nodeid="56408">在开始本课时的主要内容前，我们先来看看整个项目的架构，也让你可以更好地了解这个项目的设计与规划。</p>
<p data-nodeid="57971">下图从下至上是系统的分层设计，最下面是数据源，也就是我们的业务数据库。它通过数据导入层，导入到我们的大数据平台中，这个大数据平台通常由 Hadoop 生态构建，通过数据转换层进行转换与处理，处理后生成的数据集合可以看成数据集市，最后由 BI 应用访问数据集市层进行报表生成。可以看到数据从下往上地单向流动。</p>
<p data-nodeid="57972" class=""><img src="https://s0.lgstatic.com/i/image/M00/47/57/Ciqc1F9HkmiACjkbAABW85xL6RU920.png" alt="1.png" data-nodeid="57976"></p>




<h3 data-nodeid="56412">关系型数据库的数据导出</h3>
<p data-nodeid="56413">关系型数据库的导出过程比较简单，Spark 也有现成的方案，在之前的课程中，我们学习过 read 读取器 API，可以很方便地从支持 JDBC 的数据库中拉取数据，如下面的代码所示：</p>
<pre class="lang-scala" data-nodeid="56414"><code data-language="scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">BICubing</span> </span>{
&nbsp; &nbsp;&nbsp;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
&nbsp; &nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>.builder().appName(<span class="hljs-string">"BI-CUBING"</span>)
&nbsp; &nbsp; .master(<span class="hljs-string">"local"</span>)
&nbsp; &nbsp; .getOrCreate()&nbsp;&nbsp;
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">//busniess表</span>
&nbsp; &nbsp; <span class="hljs-keyword">val</span> busniess = spark.read.format(<span class="hljs-string">"jdbc"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"driver"</span>,<span class="hljs-string">"com.mysql.jdbc.Driver"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:mysql://master:3306/ttable"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"busniess"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"root"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"123456"</span>)
&nbsp; &nbsp; .load()
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">//user表</span>
&nbsp; &nbsp; <span class="hljs-keyword">val</span> user = spark.read.format(<span class="hljs-string">"jdbc"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"driver"</span>,<span class="hljs-string">"com.mysql.jdbc.Driver"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:mysql://master:3306/ttable"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"user"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"root"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"123456"</span>)
&nbsp; &nbsp; .load()
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">//checkin表</span>
&nbsp; &nbsp; <span class="hljs-keyword">val</span> checkin = spark.read.format(<span class="hljs-string">"jdbc"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"driver"</span>,<span class="hljs-string">"com.mysql.jdbc.Driver"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:mysql://master:3306/ttable"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"chenckin"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"root"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"123456"</span>)
&nbsp; &nbsp; .load()
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">//tip表</span>
&nbsp; &nbsp; <span class="hljs-keyword">val</span> tip = spark.read.format(<span class="hljs-string">"jdbc"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"driver"</span>,<span class="hljs-string">"com.mysql.jdbc.Driver"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:mysql://master:3306/ttable"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"tip"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"root"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"123456"</span>)
&nbsp; &nbsp; .load()
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">//review表</span>
&nbsp; &nbsp; <span class="hljs-keyword">val</span> review = spark.read.format(<span class="hljs-string">"jdbc"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"driver"</span>,<span class="hljs-string">"com.mysql.jdbc.Driver"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:mysql://master:3306/ttable"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"review"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"root"</span>)
&nbsp; &nbsp; .option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"123456"</span>)
&nbsp; &nbsp; .load()
    
    busniess.createOrReplaceTempView(<span class="hljs-string">"busniess"</span>)
&nbsp; &nbsp; user.createOrReplaceTempView(<span class="hljs-string">"user"</span>)
&nbsp; &nbsp; tip.createOrReplaceTempView(<span class="hljs-string">"tip"</span>)
&nbsp; &nbsp; checkin.createOrReplaceTempView(<span class="hljs-string">"checkin"</span>)
&nbsp; &nbsp; review.createOrReplaceTempView(<span class="hljs-string">"review"</span>)&nbsp; &nbsp;&nbsp;
&nbsp; }
&nbsp;&nbsp;
}
</code></pre>
<p data-nodeid="56415">从代码中可以看到，读取出来的数据可直接进行转换与处理，这个过程需要比较长的时间，主要取决于数据量的大小与数据库的读性能。由于数据库不是分布式的，它的读性能至关重要。</p>
<h3 data-nodeid="56416">NoSQL 数据库的数据导出</h3>
<p data-nodeid="56417">NoSQL 数据库主要以开源软件为主，产品丰富，使用广泛。在很多场景下，它已经取代了关系型数据库，其中比较有代表性的就是 MongoDB。</p>
<p data-nodeid="56418">MongoDB 是一个文档数据库，它将数据存储在类似 JSON 的文档中。这是认识数据的最自然方法，比传统的行/列模型更具表现力和功能。它的主要特点有：</p>
<p data-nodeid="56419"><strong data-nodeid="56491">丰富的 JSON 文档</strong></p>
<ul data-nodeid="56420">
<li data-nodeid="56421">
<p data-nodeid="56422">是处理数据最自然、有效的方式。</p>
</li>
<li data-nodeid="56423">
<p data-nodeid="56424">支持数组和嵌套对象作为值。</p>
</li>
<li data-nodeid="56425">
<p data-nodeid="56426">允许灵活和动态的模式。</p>
</li>
</ul>
<p data-nodeid="56427"><strong data-nodeid="56498">强大的查询语言</strong></p>
<ul data-nodeid="56428">
<li data-nodeid="56429">
<p data-nodeid="56430">丰富且富有表现力的查询语言，无论你的文档中有多少个嵌套，都可以按任何字段进行过滤和排序。</p>
</li>
<li data-nodeid="56431">
<p data-nodeid="56432">支持聚合和其他现代用例，例如基于地理的搜索、图形搜索和文本搜索。</p>
</li>
<li data-nodeid="56433">
<p data-nodeid="56434">查询本身就是 JSON，因此很容易组合，不再需要连接字符串来动态生成 SQL 查询。</p>
</li>
</ul>
<p data-nodeid="56435"><strong data-nodeid="56505">类似关系型数据库的一些特性</strong></p>
<ul data-nodeid="56436">
<li data-nodeid="56437">
<p data-nodeid="56438">具有快照隔离的分布式多文档 ACID 事务。</p>
</li>
<li data-nodeid="56439">
<p data-nodeid="56440">支持查询连接。</p>
</li>
</ul>
<p data-nodeid="56441">从上面可以看出，MongoDB 的底层数据结构和 JSON 非常类似，所以 MongoDB 也提供直接将数据导出为 JSON 格式的工具，我们可以使用 mongoexport 命令将文档集合导出为 json 文件。导出命令如下面的代码所示：</p>
<pre class="lang-yaml" data-nodeid="62365"><code data-language="yaml"><span class="hljs-string">mongoexport</span> <span class="hljs-string">-d</span> <span class="hljs-string">dbtable</span> <span class="hljs-string">-c</span> <span class="hljs-string">busniess</span> <span class="hljs-string">--json</span> <span class="hljs-string">-o</span> <span class="hljs-string">/yourpath/busniess.json</span>
<span class="hljs-string">mongoexport</span> <span class="hljs-string">-d</span> <span class="hljs-string">dbtable</span> <span class="hljs-string">-c</span> <span class="hljs-string">review</span> <span class="hljs-string">--json</span> <span class="hljs-string">-o</span> <span class="hljs-string">/yourpath/review.json</span> 
<span class="hljs-string">mongoexport</span> <span class="hljs-string">-d</span> <span class="hljs-string">dbtable</span> <span class="hljs-string">-c</span> <span class="hljs-string">tip</span> <span class="hljs-string">--json</span> <span class="hljs-string">-o</span> <span class="hljs-string">/yourpath/tip.json</span>
<span class="hljs-string">mongoexport</span> <span class="hljs-string">-d</span> <span class="hljs-string">dbtable</span> <span class="hljs-string">-c</span> <span class="hljs-string">user</span> <span class="hljs-string">--json</span> <span class="hljs-string">-o</span> <span class="hljs-string">/yourpath/user.json</span>
<span class="hljs-string">mongoexport</span> <span class="hljs-string">-d</span> <span class="hljs-string">dbtable</span> <span class="hljs-string">-c</span> <span class="hljs-string">checkin</span> <span class="hljs-string">--json</span> <span class="hljs-string">-o</span> <span class="hljs-string">/yourpath/checkin.json</span>
</code></pre>



















<p data-nodeid="56443">导出完成后，大多数情况下，还需要将其上传到 Hadoop 的文件系统 HDFS 中，HDFS 提供了 put 命令，非常方便，命令如以下代码所示：</p>
<pre class="lang-yaml" data-nodeid="65137"><code data-language="yaml"><span class="hljs-string">hadoop</span> <span class="hljs-string">dfs</span> <span class="hljs-string">-put</span> <span class="hljs-string">/yourpath/busniess.json</span> <span class="hljs-string">/dw/bi/busniess.json</span>
<span class="hljs-string">hadoop</span> <span class="hljs-string">dfs</span> <span class="hljs-string">-put</span> <span class="hljs-string">/yourpath/review.json</span>&nbsp;<span class="hljs-string">/dw/bi</span>
<span class="hljs-string">hadoop</span> <span class="hljs-string">dfs</span> <span class="hljs-string">-put</span> <span class="hljs-string">/yourpath/tip.json</span>&nbsp;<span class="hljs-string">/dw/bi/</span>
<span class="hljs-string">hadoop</span> <span class="hljs-string">dfs</span> <span class="hljs-string">-put</span> <span class="hljs-string">/yourpath/user.json</span> <span class="hljs-string">/dw/bi/</span>
<span class="hljs-string">hadoop</span> <span class="hljs-string">dfs</span> <span class="hljs-string">-put</span> <span class="hljs-string">/yourpath/checkin.json</span> <span class="hljs-string">/dw/bi/</span>
</code></pre>












<p data-nodeid="56445">第一个路径地址是本地文件地址，也就是 MongoDB 导出的地址，第二个路径地址为上传到 HDFS 的文件夹地址。</p>
<p data-nodeid="56446">上传完成后，还需要用 Spark 读取器 API 进行读取，以便后续的转换与处理，代码如下：</p>
<pre class="lang-scala" data-nodeid="56447"><code data-language="scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">BICubing</span> </span>{
&nbsp;&nbsp;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>.builder().appName(<span class="hljs-string">"BI-CUBING"</span>)
&nbsp; &nbsp; .master(<span class="hljs-string">"local"</span>)
&nbsp; &nbsp; .getOrCreate()&nbsp;&nbsp;
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">val</span> busniess = spark.read.format(<span class="hljs-string">"json"</span>).load(<span class="hljs-string">"/dw/bi/busniess.json"</span>)
&nbsp; &nbsp; <span class="hljs-keyword">val</span> user = spark.read.format(<span class="hljs-string">"json"</span>).load(<span class="hljs-string">"/dw/bi/user.json"</span>)
&nbsp; &nbsp; <span class="hljs-keyword">val</span> checkin = spark.read.format(<span class="hljs-string">"json"</span>).load(<span class="hljs-string">"/dw/bi/checkin.json"</span>)
&nbsp; &nbsp; <span class="hljs-keyword">val</span> review = spark.read.format(<span class="hljs-string">"json"</span>).load(<span class="hljs-string">"/dw/bi/review.json"</span>)
&nbsp; &nbsp; <span class="hljs-keyword">val</span> tip = spark.read.format(<span class="hljs-string">"json"</span>).load(<span class="hljs-string">"/dw/bi/tip.json"</span>)
&nbsp; &nbsp; busniess.createOrReplaceTempView(<span class="hljs-string">"busniess"</span>)
&nbsp; &nbsp; user.createOrReplaceTempView(<span class="hljs-string">"user"</span>)
&nbsp; &nbsp; tip.createOrReplaceTempView(<span class="hljs-string">"tip"</span>)
&nbsp; &nbsp; checkin.createOrReplaceTempView(<span class="hljs-string">"checkin"</span>)
&nbsp; &nbsp; review.createOrReplaceTempView(<span class="hljs-string">"review"</span>)
&nbsp; &nbsp; &nbsp;&nbsp;
&nbsp; }
&nbsp;&nbsp;
}
</code></pre>
<h3 data-nodeid="57025">总结</h3>


<p data-nodeid="56450">数据导出过程非常耗时且重要，是 ETL 的重要组成部分，也是数据分析的基础。通常来说，在生产环境中，你的业务数据库不止有一种，所以这个过程有可能非常复杂。本课时完成的是分层设计中的数据导入层，使用的是数据源层的数据。</p>
<p data-nodeid="56451">另外，大家可以看到，在 NoSQL 数据库的数据导出的过程中，既有命令行，也有 Spark 代码，两者交替进行，所以你还需要将这些过程整合到一个过程中去，这部分我们将在下个课时介绍。</p>
<p data-nodeid="56452">本节课的内容看似简单，但要运用到实际情况中还需反复练习，所以我们暂不介绍更多，希望你在课后努力消化，有问题欢迎留言。</p>

---

### 精选评论


