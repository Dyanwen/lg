<p data-nodeid="3029">本课时我们来学习如何处理结构化数据：DataFrame、Dataset 和 Spark SQL。由于本课时是专栏的第 3 模块：Spark 高级编程的第 1 课，在开始今天的课程之前，首先对上一个模块进行一个总结。</p>
<h3 data-nodeid="3030">模块回顾</h3>
<p data-nodeid="3031">在第 2 模块里，我们学习了 Spark 核心数据结构 RDD 和算子，以及 Spark 相关的一些底层原理。可以看到 RDD 将大数据集抽象为集合，这掩盖了分布式数据集的复杂性，而函数式编程风格的算子也能满足不同的数据处理逻辑。但是，RDD + 算子的组合，对于普通分析师来说还是不太友好，他们习惯于“表”的概念而非“集合”，而使用基于集合完成数据处理的逻辑更像是程序员们的思维方式。对于数据处理逻辑，分析师们更习惯用 SQL 而非算子来表达。所以，Spark 借鉴了 Python 数据分析库 pandas 中 DataFrame 的概念，推出了 DataFrame、Dataset 与 Spark SQL。</p>
<p data-nodeid="3032">在数据科学领域中，DataFrame 抽象了矩阵，如 R、pandas 中的 DataFrame；在数据工程领域，如 Spark SQL 中，DataFrame 更多地代表了关系型数据库中的表，这样就可以利用简单易学的 SQL 来进行数据分析；在 Spark 中，我们既可以用 Spark SQL + DataFrame 的组合实现海量数据分析，也可用 DataFrame + MLlib（Spark 机器学习库）的组合实现海量数据挖掘。</p>
<p data-nodeid="3033">在计算机领域中，高级往往意味着简单、封装程度高，而与之对应的通常是复杂、底层。对于 Spark 编程来说，RDD + 算子的组合无疑是比较底层的，而 <strong data-nodeid="3254">DataFrame + Spark SQL 的组合无论从学习成本，还是从性能开销上来说，都显著优于前者组合，所以无论是分析师还是程序员，这种方式才是使用 Spark 的首选。</strong> 此外，对于分析师来说，DataFrame 对他们来说并不陌生，熟悉的概念也能让他们快速上手。</p>
<p data-nodeid="3034">本课时的主要内容有 4点：</p>
<ul data-nodeid="3035">
<li data-nodeid="3036">
<p data-nodeid="3037">DataFrame、Dataset 的起源与演变</p>
</li>
<li data-nodeid="3038">
<p data-nodeid="3039">DataFrame API</p>
</li>
<li data-nodeid="3040">
<p data-nodeid="3041">Dataset API</p>
</li>
<li data-nodeid="3042">
<p data-nodeid="3043">Spark SQL</p>
</li>
</ul>
<p data-nodeid="3044">这里特别说明的是，由于 DataFrame API 的 Scala 版本与 Python 版本大同小异，差异极小，所以本课时的代码以 Scala 版本为主。</p>
<h3 data-nodeid="3045">DataFrame、Dataset 的起源与演变</h3>
<p data-nodeid="3046">DataFrame 在 Spark 1.3 被引入，它的出现取代了 SchemaRDD，Dataset 最开始在 Spark 1.6 被引入，当时还属于实验性质，在 2.0 版本时正式成为 Spark 的一部分，并且在 Spark 2.0 中，DataFrame API 与 Dataset API 在形式上得到了统一。</p>
<p data-nodeid="3047">Dataset API 提供了类型安全的面向对象编程接口。Dataset 可以通过将表达式和数据字段暴露给查询计划程序和 Tungsten 的快速内存编码，从而利用 Catalyst 优化器。但是，现在 DataFrame 和 Dataset 都作为 Apache Spark 2.0 的一部分，<strong data-nodeid="3268">其实 DataFrame 现在是 Dataset Untyped API 的特殊情况</strong>。更具体地说：</p>
<pre class="lang-java" data-nodeid="3048"><code data-language="java">DataFrame = Dataset[Row]
</code></pre>
<p data-nodeid="3049">下面这张图比较清楚地表示了 DataFrame 与 Dataset 的变迁与关系。</p>
<p data-nodeid="3050"><img alt="2.png" src="https://s0.lgstatic.com/i/image/M00/0E/F1/Ciqc1F7GfyGAfOt8AAEFYyYXSjM987.png" data-nodeid="3272"></p>
<p data-nodeid="3051"><strong data-nodeid="3276">由于 Python 不是类型安全的语言，所以 Spark Python API 没有 Dataset API，而只提供了 DataFrame API。当然，Java 和 Scala 就没有这种问题。</strong></p>
<h3 data-nodeid="3052">DataFrame API</h3>
<p data-nodeid="3053">DataFrame 与 Dataset API 提供了简单的、统一的并且更富表达力的 API ，简言之，与 RDD 与算子的组合相比，DataFrame 与 Dataset API 更加高级，所以这也是为什么我将这个模块命名为 Spark 高级编程。</p>
<p data-nodeid="3054">DataFrame 不仅可以使用 SQL 进行查询，其自身也具有灵活的 API 可以对数据进行查询，与 RDD API 相比，DataFrame API 包含了更多的应用语义，<strong data-nodeid="3284">所谓应用语义，就是能让计算框架知道你的目标的信息</strong>，这样计算框架就能更有针对性地对作业进行优化，本课时主要介绍如何创建DataFrame 以及如何利用 DataFrame 进行查询。</p>
<h4 data-nodeid="3055">1、创建DataFrame</h4>
<p data-nodeid="3056">DataFrame 目前支持多种数据源、文件格式，如 Json、CSV 等，也支持由外部数据库直接读取数据生成，此外还支持由 RDD 通过类型反射生成，甚至还可以通过流式数据源生成，这在下个模块会详细介绍。DataFrame API 非常标准，创建 DataFrame 都通过 read 读取器进行读取。下面列举了如何读取几种常见格式的文件。</p>
<ul data-nodeid="3057">
<li data-nodeid="3058">
<p data-nodeid="3059">读取 Json 文件。Json 文件如下：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3060"><code data-language="scala">{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>}
{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"age"</span>:<span class="hljs-number">30</span>}
{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"age"</span>:<span class="hljs-number">19</span>}
......
<span class="hljs-keyword">val</span> df = spark.read.json(<span class="hljs-string">"examples/src/main/resources/people.json"</span>)
</code></pre>
<p data-nodeid="3061">我们可以利用初始化好的 SparkSession（spark）读取 Json 格式文件。</p>
<ul data-nodeid="3062">
<li data-nodeid="3063">
<p data-nodeid="3064">读取 CSV 文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3065"><code data-language="scala"><span class="hljs-keyword">val</span> df = spark.read.csv(<span class="hljs-string">"examples/src/main/resources/people.csv"</span>)
</code></pre>
<ul data-nodeid="3066">
<li data-nodeid="3067">
<p data-nodeid="3068">从 Parquet 格式文件中生成：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3069"><code data-language="scala"><span class="hljs-keyword">val</span> df = spark.read.parquet(<span class="hljs-string">"examples/src/main/resources/people.csv"</span>)
</code></pre>
<ul data-nodeid="3070">
<li data-nodeid="3071">
<p data-nodeid="3072">从 ORC 格式文件中生成：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3073"><code data-language="scala"><span class="hljs-keyword">val</span> df = spark.read.orc(<span class="hljs-string">"examples/src/main/resources/people.csv"</span>)
关于 <span class="hljs-type">ORC</span> 与 <span class="hljs-type">Parquet</span> 文件格式会在后面详细介绍。
</code></pre>
<ul data-nodeid="3074">
<li data-nodeid="3075">
<p data-nodeid="3076">从文本中生成：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3077"><code data-language="scala"><span class="hljs-keyword">val</span> df = spark.read.text(<span class="hljs-string">"examples/src/main/resources/people.csv"</span>)
</code></pre>
<ul data-nodeid="3078">
<li data-nodeid="3079">
<p data-nodeid="3080">通过 JDBC 连接外部数据库读取数据生成</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3081"><code data-language="scala"><span class="hljs-keyword">val</span> df = spark.read
.format(<span class="hljs-string">"jdbc"</span>)
.option(<span class="hljs-string">"url"</span>, <span class="hljs-string">"jdbc:postgresql:dbserver"</span>)
.option(<span class="hljs-string">"dbtable"</span>, <span class="hljs-string">"schema.tablename"</span>)
.option(<span class="hljs-string">"user"</span>, <span class="hljs-string">"username"</span>)
.option(<span class="hljs-string">"password"</span>, <span class="hljs-string">"password"</span>)
.load()
</code></pre>
<p data-nodeid="3082">上面的代码表示通过 JDBC 相关配置，读取数据。</p>
<ul data-nodeid="3083">
<li data-nodeid="3084">
<p data-nodeid="3085">通过 RDD 反射生成。此种方法是字符串反射为 DataFrame 的 Schema，再和已经存在的 RDD 一起生成 DataFrame，代码如下所示：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3086"><code data-language="scala"><span class="hljs-keyword">import</span> spark.implicits._
<span class="hljs-keyword">val</span> schemaString = <span class="hljs-string">"id f1 f2 f3 f4"</span>
<span class="hljs-comment">// 通过字符串转换和类型反射生成schema</span>
<span class="hljs-keyword">val</span> fields = schemaString.split(<span class="hljs-string">" "</span>).map(fieldName =&gt; <span class="hljs-type">StructField</span>(fieldName, <span class="hljs-type">StringType</span>, nullable = <span class="hljs-literal">true</span>))
<span class="hljs-keyword">val</span> schema = <span class="hljs-type">StructType</span>(fields)
<span class="hljs-comment">// 需要将RDD转化为RDD[Row]类型</span>
<span class="hljs-keyword">val</span> rowRDD = spark.sparkContext.textFile(textFilePath).map(_.split(<span class="hljs-string">","</span>)).map(attributes =&gt; 
<span class="hljs-type">Row</span>(attributes(<span class="hljs-number">0</span>), 
attributes(<span class="hljs-number">1</span>),
attributes(<span class="hljs-number">2</span>),
attributes(<span class="hljs-number">3</span>),
attributes(<span class="hljs-number">4</span>).trim)
)
<span class="hljs-comment">// 生成DataFrame</span>
<span class="hljs-keyword">val</span> df = spark.createDataFrame(rowRDD, schema)
</code></pre>
<p data-nodeid="3087">注意这种方式需要隐式转换，需在转换前写上第一行：</p>
<pre class="lang-scala" data-nodeid="3088"><code data-language="scala"><span class="hljs-keyword">import</span> spark.implicits._
</code></pre>
<p data-nodeid="3089">DataFrame 初始化完成后，可以通过 show 方法来查看数据，Json、Parquet、ORC 等数据源是自带 Schema 的，而那些无 Schema 的数据源，DataFrame 会自己生成 Schema。</p>
<p data-nodeid="3090">Json 文件生成的 DataFrame 如下：</p>
<p data-nodeid="3091"><img alt="image (5).png" src="https://s0.lgstatic.com/i/image/M00/0E/E5/CgqCHl7GWLqAKKlmAAAetVmsk_s361.png" data-nodeid="3301"></p>
<p data-nodeid="3092">CSV 文件生成的 DataFrame 如下：</p>
<p data-nodeid="3093"><img alt="image (6).png" src="https://s0.lgstatic.com/i/image/M00/0E/D9/Ciqc1F7GWMmANNN-AAAqx4SoyJI563.png" data-nodeid="3305"></p>
<h4 data-nodeid="3094">2、查询</h4>
<p data-nodeid="3095">完成初始化的工作之后就可以使用 DataFrame API 进行查询了，DataFrame API 主要分为两种风格，一种依然是 RDD 算子风格，如 reduce、groupByKey、map、flatMap 等，另外一种则是 SQL 风格，如 select、where 等。</p>
<p data-nodeid="3096"><strong data-nodeid="3311">2.1 算子风格</strong></p>
<p data-nodeid="3097">我们简单选取几个有代表性的 RDD 算子风格的 API，具体如下：</p>
<pre class="lang-scala" data-nodeid="3098"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">groupByKey</span></span>[<span class="hljs-type">K</span>: <span class="hljs-type">Encoder</span>](func: <span class="hljs-type">T</span> =&gt; <span class="hljs-type">K</span>): <span class="hljs-type">KeyValueGroupedDataset</span>[<span class="hljs-type">K</span>, <span class="hljs-type">T</span>]
</code></pre>
<p data-nodeid="3099">与 RDD 算子版作用相同，返回类型为 Dataset。</p>
<pre class="lang-scala" data-nodeid="3100"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">map</span></span>[<span class="hljs-type">U</span> : <span class="hljs-type">Encoder</span>](func: <span class="hljs-type">T</span> =&gt; <span class="hljs-type">U</span>): <span class="hljs-type">Dataset</span>[<span class="hljs-type">U</span>]
</code></pre>
<p data-nodeid="3101">与 RDD 算子版作用相同，返回类型为 Dataset。</p>
<pre class="lang-scala" data-nodeid="3102"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">flatMap</span></span>[<span class="hljs-type">U</span> : <span class="hljs-type">Encoder</span>](func: <span class="hljs-type">T</span> =&gt; <span class="hljs-type">TraversableOnce</span>[<span class="hljs-type">U</span>]): <span class="hljs-type">Dataset</span>[<span class="hljs-type">U</span>]
</code></pre>
<p data-nodeid="3103">与 RDD 算子版作用相同，返回类型为 Dataset。<br>
这些算子用法大同小异，但都需要传入 Encoder 参数，这可以通过隐式转换解决，在调用算子前，需加上：</p>
<pre class="lang-scala" data-nodeid="3104"><code data-language="scala"><span class="hljs-keyword">import</span> spark.implicits._
</code></pre>
<p data-nodeid="3105"><strong data-nodeid="3321">2.2 SQL风格</strong></p>
<p data-nodeid="3106">这类 API 的共同之处就是支持将部分 SQL 语法的字符串作为参数直接传入。</p>
<ul data-nodeid="3107">
<li data-nodeid="3108">
<p data-nodeid="3109">select 和 where</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3110"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">select</span></span>(cols: <span class="hljs-type">Column</span>*): <span class="hljs-type">DataFrame</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">where</span></span>(conditionExpr: <span class="hljs-type">String</span>): <span class="hljs-type">Dataset</span>[<span class="hljs-type">T</span>]
</code></pre>
<p data-nodeid="3111">条件查询，例如：</p>
<pre class="lang-scala" data-nodeid="3112"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>).where(<span class="hljs-string">"name is not null and age &gt; 10"</span>).foreach(println(_))
</code></pre>
<ul data-nodeid="3113">
<li data-nodeid="3114">
<p data-nodeid="3115">groupBy</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3116"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">groupBy</span></span>(col1: <span class="hljs-type">String</span>, cols: <span class="hljs-type">String</span>*): <span class="hljs-type">RelationalGroupedDataset</span>
</code></pre>
<p data-nodeid="3117">分组统计。例如：</p>
<pre class="lang-scala" data-nodeid="3118"><code data-language="scala">df.select(<span class="hljs-string">"name"</span>,<span class="hljs-string">"age"</span>).groupBy(<span class="hljs-string">"age"</span>).count().foreach(println(_))
</code></pre>
<p data-nodeid="3119">此外，某些 RDD 算子风格的 API 也可以传入部分 SQL 语法的字符串，如 filter。例如：</p>
<pre class="lang-scala" data-nodeid="3120"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>).foreach(println(_))
</code></pre>
<ul data-nodeid="3121">
<li data-nodeid="3122">
<p data-nodeid="3123">join</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="3124"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">join</span></span>(right: <span class="hljs-type">Dataset</span>[_], usingColumns: <span class="hljs-type">Seq</span>[<span class="hljs-type">String</span>], joinType: <span class="hljs-type">String</span>): <span class="hljs-type">DataFrame</span>
</code></pre>
<p data-nodeid="3125">DataFrame API 还支持最普遍的连接操作，代码如下：</p>
<pre class="lang-scala" data-nodeid="3126"><code data-language="scala"><span class="hljs-keyword">val</span> leftDF = ...
<span class="hljs-keyword">val</span> rightDF = ...
leftDF.join(rightDF, leftDF(<span class="hljs-string">"pid"</span>) === rightDF(<span class="hljs-string">"fid"</span>), <span class="hljs-string">"left_outer"</span>).foreach(println(_))
</code></pre>
<p data-nodeid="3127">其中 joinType 参数支持常用的连接类型，选项有 inner、cross、outer、full、full_outer、left、left_outer、right、right_outer、left_semi 和 left_anti，其中 cross 表示笛卡儿积，这在实际使用中比较少见；left_semi 是左半连接，是 Spark 对标准 SQL 中的 in 关键字的变通实现；left_anti 是 Spark 对标准 SQL 中的 not in 关键字的变通实现。<br>
除了 groupBy 这种分组方式，DataFrame 还支持一些特别的分组方式如 pivot、rollup、cube 等，以及常用的分析函数，先来看一个数据集：</p>
<pre class="lang-java" data-nodeid="3128"><code data-language="java">{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">92</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}
{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">75</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">62</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">96</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">98</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">78</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2017"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">90</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">76</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">74</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">68</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">95</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">81</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2016"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">95</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">91</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">85</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">77</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">63</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">99</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">79</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">85</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>, <span class="hljs-string">"year"</span>:<span class="hljs-string">"2015"</span>}
</code></pre>
<p data-nodeid="3129">以上是某班学生 3 年的成绩单，一共有 3 个维度，即 name、subject 和 year，度量为 grade，也就是成绩，因此，这个 DataFrame 可以看成三维数据立方体，如下图所示。</p>
<p data-nodeid="3130"><img alt="1.png" src="https://s0.lgstatic.com/i/image/M00/0E/FD/CgqCHl7Gfy6AJXolAAAbv92AgtQ320.png" data-nodeid="3350"></p>
<p data-nodeid="3131">现在需要统计每个学生各科目 3 年的平均成绩，该操作可以通过下面的方式实现：</p>
<pre class="lang-scala" data-nodeid="15985"><code data-language="scala">dfSG.groupBy(<span class="hljs-string">"name"</span>,<span class="hljs-string">"subject"</span>).avg(<span class="hljs-string">"grade"</span>)
</code></pre>
<p data-nodeid="15986">但是，这种形式使结果数据集只有两列——name 和 subject，不利于进一步分析，而利用 DataFrame 的数据透视 pivot 功能无疑更加方便。 pivot 功能在 pandas、Excel 等分析工具已得到了广泛应用，用户想使用透视功能，需要指定分组规则、需要透视的列以及聚合的维度列。所谓“透视”比较形象，即在分组结果上，对每一组进行“透视”，透视的结果会导致每一组基于透视列展开，最后再根据聚合操作进行聚合，统计每个学生每科 3 年平均成绩实现如下：</p>
<pre class="lang-scala" data-nodeid="15987"><code data-language="scala">dfSG.groupBy(<span class="hljs-string">"name"</span>).pivot(<span class="hljs-string">"subject"</span>).avg(<span class="hljs-string">"grade"</span>).show()
</code></pre>
<p data-nodeid="15988">结果如下：</p>
<p data-nodeid="15989"><img alt="image (8).png" src="https://s0.lgstatic.com/i/image/M00/0E/E5/CgqCHl7GWV2AGLYaAABeumgs0CI148.png" data-nodeid="16102"></p>
<p data-nodeid="15990">除了 groupBy 之外，DataFrame 还提供 rollup 和 cube 的方式进行分组聚合，如下：</p>
<pre class="lang-scala" data-nodeid="15991"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">rollup</span></span>(col1: <span class="hljs-type">String</span>, cols: <span class="hljs-type">String</span>*): <span class="hljs-type">RelationalGroupedDataset</span>
</code></pre>
<p data-nodeid="15992">rollup 也是用来进行分组统计，只不过分组逻辑有所不同，假设 rollup(A,B,C)，其中 A、B、C 分别为 3 列，那么会先对 A、B、C 进行分组，然后依次对 A、B 进行分组、对 A 进行分组、对全表进行分组，执行：</p>
<pre class="lang-scala" data-nodeid="15993"><code data-language="scala">dfSG.rollup(<span class="hljs-string">"name"</span>, <span class="hljs-string">"subject"</span>).avg(<span class="hljs-string">"grade"</span>).show()
</code></pre>
<p data-nodeid="15994">结果如下：</p>
<p data-nodeid="15995"><img alt="image (9).png" src="https://s0.lgstatic.com/i/image/M00/0E/E6/CgqCHl7GWkqAAgoOAACqSY_hmRI132.png" data-nodeid="16108"></p>
<p data-nodeid="15996">可以看到，除了按照 name + subject 的组合键进行分组，还分别对每个人进行了分组，如 Michael,null，此外还将全表分为了一组，如 null,null。</p>
<pre class="lang-scala" data-nodeid="15997"><code data-language="scala"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">cube</span></span>(col1: <span class="hljs-type">String</span>, cols: <span class="hljs-type">String</span>*): <span class="hljs-type">RelationalGroupedDataset</span>
</code></pre>
<p data-nodeid="15998">cube 与 rollup 类似，分组依据有所不同，仍以 cube(A,B,C) 为例，分组依据分别是 (A,B,C)、(A,B)、(A,C)、(B,C)、(A)、(B)、(C)、全表，执行：</p>
<pre class="lang-scala" data-nodeid="15999"><code data-language="scala">dfSG.cube(<span class="hljs-string">"name"</span>, <span class="hljs-string">"subject"</span>).avg(<span class="hljs-string">"grade"</span>).show()
</code></pre>
<p data-nodeid="16000">结果如下：</p>
<p data-nodeid="16001"><img alt="image (10).png" src="https://s0.lgstatic.com/i/image/M00/0E/E6/CgqCHl7GWleAMtybAADZqfH4AY0141.png" data-nodeid="16114"></p>
<p data-nodeid="16002">可以看到与 rollup 不同，这里还分别对每个科目进行分组，如 null、math。</p>
<p data-nodeid="16003">在实际使用中，你应该尽量选用并习惯于用 SQL 风格的算子完成开发任务，SQL 风格的查询 API 不光表现力强，另外也非常易读。</p>
<h4 data-nodeid="16004">3、写出</h4>
<p data-nodeid="16005">与创建 DataFrame 的 read 读取器相对应，写出为 write 输出器 API。下面列举了如何输出几种常见格式的文件。</p>
<ul data-nodeid="16006">
<li data-nodeid="16007">
<p data-nodeid="16008">写出为 Json 文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16009"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>).write.json(<span class="hljs-string">"/your/output/path"</span>)
</code></pre>
<ul data-nodeid="16010">
<li data-nodeid="16011">
<p data-nodeid="16012">写出为 Parquet 文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16013"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>).write.parquet(<span class="hljs-string">"/your/output/path"</span>)
</code></pre>
<ul data-nodeid="16014">
<li data-nodeid="16015">
<p data-nodeid="16016">写出为 ORC 文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16017"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>).write.orc(<span class="hljs-string">"/your/output/path"</span>)
</code></pre>
<ul data-nodeid="16018">
<li data-nodeid="16019">
<p data-nodeid="16020">写出为文本文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16021"><code data-language="scala">df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>).write.text(<span class="hljs-string">"/your/output/path"</span>)
</code></pre>
<ul data-nodeid="16022">
<li data-nodeid="16023">
<p data-nodeid="16024">写出为 CSV 文件：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16025"><code data-language="scala"><span class="hljs-keyword">val</span> saveOptions = <span class="hljs-type">Map</span>(<span class="hljs-string">"header"</span> -&gt; <span class="hljs-string">"true"</span>, <span class="hljs-string">"path"</span> -&gt; <span class="hljs-string">"csvout"</span>)
df.select(<span class="hljs-string">"age"</span>, <span class="hljs-string">"name"</span>).filter(<span class="hljs-string">"age &gt; 10"</span>)
.write
.format(<span class="hljs-string">"com.databricks.spark.csv"</span>)
.mode(<span class="hljs-type">SaveMode</span>.<span class="hljs-type">Overwrite</span>)
.options(saveOptions)
.save()
</code></pre>
<p data-nodeid="16026">我们还可以在保存时对格式已经输出的方式进行设定，例如本例中是保留表头，并且输出方式是 Overwrite，输出方式有 Append、ErrorIfExist、Ignore、Overwrite，分别代表追加到已有输出路径中、如果输出路径存在则报错、存在则停止、存在则覆盖。</p>
<ul data-nodeid="16027">
<li data-nodeid="16028">
<p data-nodeid="16029">写出到关系型数据库：</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="16030"><code data-language="scala"><span class="hljs-keyword">val</span> prop = <span class="hljs-keyword">new</span> java.util.<span class="hljs-type">Properties</span>
prop.setProperty(<span class="hljs-string">"user"</span>,<span class="hljs-string">"spark"</span>)
prop.setProperty(<span class="hljs-string">"password"</span>,<span class="hljs-string">"123"</span>)
df.write.mode(<span class="hljs-type">SaveMode</span>.<span class="hljs-type">Append</span>).jdbc(<span class="hljs-string">"jdbc:mysql://localhost:3306/test"</span>,<span class="hljs-string">"tablename"</span>,prop)
</code></pre>
<p data-nodeid="16031">写出到关系型数据库同样基于 JDBC ，用此种方式写入关系型数据库，表名可以不存在。</p>
<h3 data-nodeid="16032">Dataset API</h3>
<p data-nodeid="16033">从本质上来说，DataFrame 只是 Dataset 的一种特殊情况，在 Spark 2.x 中已经得到了统一：</p>
<pre class="lang-plain" data-nodeid="16034"><code data-language="plain">DataFrame = Dataset[Row]
</code></pre>
<p data-nodeid="16035">因此，在使用 DataFrame API 的过程中，很容易就会自动转换为 Dataset[String]、Dataset[Int] 等类型。除此之外，用户还可以自定义类型。下面来看看 DataFrame 转成 Dataset 的例子，下面是一个 Json 文件，记录了学生的单科成绩：</p>
<pre class="lang-scala" data-nodeid="16036"><code data-language="scala">{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">92</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>}
{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">75</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">62</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"Chinese"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Michael"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">96</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Andy"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">98</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Justin"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">78</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>}

{<span class="hljs-string">"name"</span>:<span class="hljs-string">"Berta"</span>, <span class="hljs-string">"grade"</span>:<span class="hljs-number">87</span>, <span class="hljs-string">"subject"</span>:<span class="hljs-string">"math"</span>}
</code></pre>
<p data-nodeid="16037">代码如下：</p>
<pre class="lang-scala" data-nodeid="16038"><code data-language="scala"><span class="hljs-comment">// 首先定义StudentGrade类</span>
<span class="hljs-keyword">case</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StudentGrade</span>(<span class="hljs-params">name: <span class="hljs-type">String</span>, subject: <span class="hljs-type">String</span>, grade: <span class="hljs-type">Long</span></span>)</span>
<span class="hljs-comment">// 生成DataFrame</span>
<span class="hljs-keyword">val</span> dfSG = spark.read.json(<span class="hljs-string">"data/examples/target/scala-2.11/classes/student_grade.json"</span>)
<span class="hljs-comment">// 方法1:通过map函数手动转换为Dataset[StudentGrade]类型</span>
<span class="hljs-keyword">val</span> dsSG: <span class="hljs-type">Dataset</span>[<span class="hljs-type">StudentGrade</span>] = dfSG.map(a =&gt; <span class="hljs-type">StudentGrade</span>(a.getAs[<span class="hljs-type">String</span>](<span class="hljs-number">0</span>),a.getAs[<span class="hljs-type">String</span>](<span class="hljs-number">1</span>),a.getAs[<span class="hljs-type">Long</span>](<span class="hljs-number">2</span>)))
<span class="hljs-comment">// 方法2:使用DataFrame的as函数进行转换</span>
<span class="hljs-keyword">val</span> dsSG2: <span class="hljs-type">Dataset</span>[<span class="hljs-type">StudentGrade</span>] = dfSG.as[<span class="hljs-type">StudentGrade</span>]
<span class="hljs-comment">// 方法3：通过RDD转换而成（基于同样内容的CSV文件）</span>
<span class="hljs-keyword">val</span> dsSG3 = spark.sparkContext.
textFile(<span class="hljs-string">"data/examples/target/scala-2.11/classes/student_grade.csv"</span>).
map[<span class="hljs-type">StudentGrade</span>](row =&gt; {
     <span class="hljs-keyword">val</span> fields = row.split(<span class="hljs-string">","</span>)
	&nbsp;<span class="hljs-type">StudentGrade</span>(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fields(<span class="hljs-number">0</span>).toString(),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fields(<span class="hljs-number">1</span>).toString(),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fields(<span class="hljs-number">2</span>).toLong
	&nbsp;&nbsp;&nbsp; )
}).toDS
	&nbsp;
<span class="hljs-comment">// 求每科的平均分</span>
dsSG3.groupBy(<span class="hljs-string">"subject"</span>).mean(<span class="hljs-string">"grade"</span>).foreach(println(_))
以上 <span class="hljs-number">3</span> 种方法都可以将 <span class="hljs-type">DataFrame</span> 转换为 <span class="hljs-type">Dataset</span> 。转换完成后，就可以使用其 <span class="hljs-type">API</span> 对数据进行分析，使用方式与 <span class="hljs-type">DataFrame</span> 并无不同。
</code></pre>
<h3 data-nodeid="16039">Spark SQL</h3>
<p data-nodeid="16040">在实际工作中，使用频率最高的当属 Spark SQL，通常一个大数据处理项目中，70% 的数据处理任务都是由 Spark SQL 完成，它贯穿于数据预处理、数据转换和最后的数据分析。由于 SQL 的学习成本低、用户基数大、函数丰富，Spark SQL 也通常是使用 Spark 最方便的方式。此外，由于 SQL 包含了丰富的应用语义，所以 Catalyst 优化器带来的性能巨大提升也使 Spark SQL 成为编写 Spark 作业的最佳方式。接下来我将为你介绍 Spark SQL 的使用。</p>
<p data-nodeid="16041">从使用层面上来讲，要想用好 Spark SQL，只需要编写 SQL 就行了，本课时的最后简单介绍了下 SQL 的常用语法，方便没有接触过 SQL 的同学快速入门。</p>
<h4 data-nodeid="16042">1、创建临时视图</h4>
<p data-nodeid="16043">想使用 Spark SQL，可以先创建临时视图，相当于数据库中的表，这可以通过已经存在的 DataFrame、Dataset 直接生成；也可以直接从 Hive 元数据库中获取元数据信息直接进行查询。先来看看创建临时视图：</p>
<pre class="lang-scala" data-nodeid="16044"><code data-language="scala"><span class="hljs-keyword">case</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StudentGrade</span>(<span class="hljs-params">name: <span class="hljs-type">String</span>, subject: <span class="hljs-type">String</span>, grade: <span class="hljs-type">Long</span></span>)</span>
<span class="hljs-comment">// 生成DataFrame</span>
<span class="hljs-keyword">val</span> dfSG = spark.read.json(<span class="hljs-string">"data/examples/target/scala-2.11/classes/student_grade.json"</span>)
<span class="hljs-comment">// 生成Dataset</span>
<span class="hljs-keyword">val</span> dsSG = dfSG.map(
	&nbsp;&nbsp;a =&gt; <span class="hljs-type">StudentGrade</span>( 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a.getAs[<span class="hljs-type">String</span>](<span class="hljs-string">"name"</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a.getAs[<span class="hljs-type">String</span>](<span class="hljs-string">"subject"</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;a.getAs[<span class="hljs-type">Long</span>](<span class="hljs-string">"grade"</span>)
	&nbsp;&nbsp;)
)
	&nbsp;
<span class="hljs-comment">// 创建临时视图</span>
dfSG.createOrReplaceTempView(<span class="hljs-string">"student_grade_df"</span>)
dsSG.createOrReplaceTempView(<span class="hljs-string">"student_grade_ds"</span>)
<span class="hljs-comment">// 计算每科的平均分</span>
spark.sql(<span class="hljs-string">"SELECT subject, AVG(grade) FROM student_grade_df GROUP BY subject"</span>).show()
spark.sql(<span class="hljs-string">"SELECT subject, AVG(grade) FROM student_grade_ds GROUP BY subject"</span>).show()
</code></pre>
<p data-nodeid="16045">对于 Dataset 来说，对象类型的数据结构会作为临时视图的元数据，在 SQL 中可以直接使用。</p>
<h4 data-nodeid="16046">2、使用Hive元数据</h4>
<p data-nodeid="16047">随着 Spark 越来越流行，有很多情况，需要将 Hive 作业改写成 Spark SQL 作业，Spark SQL 可以通过 hive-site.xml 文件的配置，直接读取 Hive 元数据。这样，改写的工作量就小了很多，代码如下：</p>
<pre class="lang-scala" data-nodeid="16048"><code data-language="scala"><span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
.builder()
.master(<span class="hljs-string">"local[*]"</span>)
.appName(<span class="hljs-string">"Hive on Spark"</span>)
.enableHiveSupport()
.getOrCreate()
<span class="hljs-comment">// 直接查询</span>
spark.sql(…………)
</code></pre>
<p data-nodeid="16049">代码中通过 enableHiveSupport 方法开启对 Hive 的支持，但需要将 Hive 配置文件 hive-site.xml 复制到 Spark 的配置文件夹下。</p>
<h3 data-nodeid="16050">3、查询语句</h3>
<p data-nodeid="16051">Spark 的 SQL 语法源于 Presto （一种支持 SQL 的大规模并行处理技术，适合 OLAP），在源码中我们可以看见，Spark 的 SQL 解析引擎直接采用了 Presto 的 SQL 语法文件。查询是 Spark SQL 的核心功能，Spark SQL 的查询语句模式如下：</p>
<pre class="lang-SQL" data-nodeid="16052"><code data-language="SQL">[ <span class="hljs-keyword">WITH</span> with_query [, ...] ]
<span class="hljs-keyword">SELECT</span> [ <span class="hljs-keyword">ALL</span> | <span class="hljs-keyword">DISTINCT</span> ] select_expr [, ...]
[ <span class="hljs-keyword">FROM</span> from_item [, ...] ]
[ <span class="hljs-keyword">WHERE</span> condition ]
[ <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> expression [, ...] ]
[ <span class="hljs-keyword">HAVING</span> condition]
[ <span class="hljs-keyword">UNION</span> [ <span class="hljs-keyword">ALL</span> | <span class="hljs-keyword">DISTINCT</span> ] <span class="hljs-keyword">select</span> ]
[ <span class="hljs-keyword">ORDER</span> <span class="hljs-keyword">BY</span> expression [ <span class="hljs-keyword">ASC</span> | <span class="hljs-keyword">DESC</span> ] [, ...] ]
[ <span class="hljs-keyword">LIMIT</span> <span class="hljs-keyword">count</span> ]
</code></pre>
<p data-nodeid="16053">其中 from_item 为以下之一：</p>
<pre class="lang-scala" data-nodeid="16054"><code data-language="scala">table_name [ [ <span class="hljs-type">AS</span> ] alias [ ( column_alias [, ...] ) ] ]
from_item join_type from_item [ <span class="hljs-type">ON</span> join_condition | <span class="hljs-type">USING</span> ( join_column [, ...] ) ]
</code></pre>
<p data-nodeid="16055">该模式基本涵盖了 Spark SQL 中查询语句的各种写法。</p>
<p data-nodeid="16056"><strong data-nodeid="16157">3.1 SELECT 与 FROM 子句</strong></p>
<p data-nodeid="16057">SELECT 与 FROM 是构成查询语句的最小单元，SELECT 后面跟列名表示要查询的列，或者用 * 表示所有列，FROM 后面跟表名，示例如下：</p>
<pre class="lang-sql" data-nodeid="16058"><code data-language="sql"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>, grade <span class="hljs-keyword">FROM</span> student_grade t;
</code></pre>
<p data-nodeid="16059">在使用过程中，对列名和表名都可以赋予别名，这里对 student_grade 赋予别名 t，此外我们还可以对某一列用关键字 DISTINCT 进行去重，默认为 ALL，表示不去重：</p>
<pre class="lang-sql" data-nodeid="16060"><code data-language="sql"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">COUNT</span>( <span class="hljs-keyword">DISTINCT</span> <span class="hljs-keyword">name</span>) <span class="hljs-keyword">FROM</span> student_grade;
</code></pre>
<p data-nodeid="16061">上面这条 SQL 代表统计有多少学生参加了考试。</p>
<p data-nodeid="16062"><strong data-nodeid="16168">3.2 WHERE 子句</strong></p>
<p data-nodeid="16063">WHERE 子句经常和 SELECT 配合使用，用来过滤参与查询的数据集，WHERE 后面一般会由运算符组合成谓词表达式（返回值为 True 或者 False ），例如：</p>
<pre class="lang-sql" data-nodeid="16064"><code data-language="sql"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">WHERE</span> grade &gt; <span class="hljs-number">90</span>;
<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">IS</span> <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>;
<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">LIKE</span> <span class="hljs-string">"*ndy"</span>;
</code></pre>
<p data-nodeid="16065">常见的运算符还有 !=、&lt;&gt; 等，此外还可以用逻辑运算符：AND、OR 组合谓词表达式进行查询，例如：</p>
<pre class="lang-sql" data-nodeid="16066"><code data-language="sql"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">WHERE</span> grade &gt; <span class="hljs-number">90</span> <span class="hljs-keyword">AND</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">IS</span> <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>
</code></pre>
<p data-nodeid="16067"><strong data-nodeid="16178">3.3 GROUP BY 子句</strong></p>
<p data-nodeid="16068">GROUP&nbsp;BY&nbsp;子句用于对&nbsp;SELECT&nbsp;语句的输出进行分组，分组中是匹配值的数据行。GROUP&nbsp;BY&nbsp;子句支持指定列名或列序号（从 1 开始）表达式。以下查询是等价的，都会对&nbsp;subject 列进行分组，第一个查询使用列序号，第二个查询使用列名：</p>
<pre class="lang-sql" data-nodeid="16069"><code data-language="sql"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">avg</span>(grade), subject <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> <span class="hljs-number">2</span>;
<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">avg</span>(grade), subject <span class="hljs-keyword">FROM</span> student_grade <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> subject;
</code></pre>
<p data-nodeid="16070"><strong data-nodeid="16183">使用 GROUP BY 子句时需注意，出现在 SELECT 后面的列，要么同时出现在 GROUP BY 后面，要么就在聚合函数中。</strong></p>
<p data-nodeid="16071"><strong data-nodeid="16187">3.4 HAVING 子句</strong></p>
<p data-nodeid="16072">HAVING 子句与聚合函数以及 GROUP&nbsp;BY 子句配合使用，用来过滤分组统计的结果。HAVING 子句去掉不满足条件的分组。在分组和聚合计算完成后，HAVING 对分组进行过滤。例如以下查询会过滤掉平均分大于 90 分的科目：</p>
<pre class="lang-sql" data-nodeid="16073"><code data-language="sql"><span class="hljs-keyword">SELECT</span> subject,<span class="hljs-keyword">AVG</span>(grade) 
<span class="hljs-keyword">FROM</span> student_grade 
<span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> subject 
<span class="hljs-keyword">HAVING</span> <span class="hljs-keyword">AVG</span>(grade) &lt; <span class="hljs-number">90</span>;
</code></pre>
<p data-nodeid="16074"><strong data-nodeid="16192">3.5 UNION 子句</strong></p>
<p data-nodeid="16075">UNION 子句用于将多个查询语句的结果合并为一个结果集：</p>
<pre class="lang-plain" data-nodeid="16076"><code data-language="plain">query UNION [ALL | DISTINCT] query
</code></pre>
<p data-nodeid="16077">参数&nbsp;ALL&nbsp;或&nbsp;DISTINCT&nbsp;可以控制最终结果集包含哪些行。如果指定参数 ALL，则包含全部行，即使行完全相同；如果指定参数&nbsp;DISTINCT ，则合并结果集，结果集只有唯一不重复的行；如果不指定参数，执行时默认使用&nbsp;DISTINCT。下面这句 SQL 是将两个班级的成绩进行合并：</p>
<pre class="lang-sql" data-nodeid="16078"><code data-language="sql"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade_class1
<span class="hljs-keyword">UNION</span> <span class="hljs-keyword">ALL</span> 
<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade_class2;
</code></pre>
<p data-nodeid="16079">多个 UNION 子句会从左向右执行，除非用括号明确指定顺序。</p>
<h4 data-nodeid="16080">3.6 ORDER BY 子句</h4>
<p data-nodeid="16081">ORDER&nbsp;BY&nbsp;子句按照一个或多个输出表达式对结果集排序：</p>
<pre class="lang-plain" data-nodeid="16082"><code data-language="plain">ORDER BY expression [ ASC | DESC ] [ NULLS { FIRST | LAST } ] [, ...]
</code></pre>
<p data-nodeid="16083">每个表达式由列名或列序号（从 1 开始）组成。ORDER&nbsp;BY&nbsp;子句作为查询的最后一步，在&nbsp;GROUP&nbsp;BY&nbsp;和&nbsp;HAVING&nbsp;子句之后。ASC 为默认升序，DESC 为降序。下面这句 SQL 会对结果进行过滤，并按照平均分进行排序，<strong data-nodeid="16203">注意这里使用了列别名</strong>：</p>
<pre class="lang-sql" data-nodeid="16084"><code data-language="sql"><span class="hljs-keyword">SELECT</span> subject,<span class="hljs-keyword">AVG</span>(grade) <span class="hljs-keyword">avg</span> 
<span class="hljs-keyword">FROM</span> student_grade 
<span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> subject 
<span class="hljs-keyword">HAVING</span> <span class="hljs-keyword">AVG</span>(grade) &lt; <span class="hljs-number">90</span> 
<span class="hljs-keyword">ORDER</span> <span class="hljs-keyword">BY</span> <span class="hljs-keyword">avg</span> <span class="hljs-keyword">DESC</span>;
</code></pre>
<p data-nodeid="16085"><strong data-nodeid="16207">3.7 LIMIT 子句</strong></p>
<p data-nodeid="16086">LIMIT&nbsp;子句限制结果集的行数，这在查询大表时很有用。以下示例为对单科成绩进行排序并只返回前 3 名的记录：</p>
<pre class="lang-sql" data-nodeid="16087"><code data-language="sql"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> student_grade 
<span class="hljs-keyword">WHERE</span> subject = <span class="hljs-string">'math'</span> 
<span class="hljs-keyword">ORDER</span> <span class="hljs-keyword">BY</span> grade <span class="hljs-keyword">DESC</span> 
<span class="hljs-keyword">LIMIT</span> <span class="hljs-number">3</span>;
</code></pre>
<p data-nodeid="16088"><strong data-nodeid="16212">3.8 JOIN 子句</strong></p>
<p data-nodeid="16089">JOIN 操作可以将多个有关联的表进行关联查询，下面这句 SQL 是查询数学成绩在 90 分以上的学生的院系，其中院系信息可以从学生基础信息表内，通过姓名连接得到：</p>
<pre class="lang-sql" data-nodeid="16090"><code data-language="sql"><span class="hljs-keyword">SELECT</span> g.*, a.department 
<span class="hljs-keyword">FROM</span> student_grade g 
<span class="hljs-keyword">JOIN</span> student_basic b 
<span class="hljs-keyword">ON</span> g.name = b.name 
<span class="hljs-keyword">WHERE</span> g.subject = <span class="hljs-string">'math'</span> <span class="hljs-keyword">and</span> grade &gt; <span class="hljs-number">90</span>
</code></pre>
<p data-nodeid="16091">在这句 SQL 中，表 g 被称为驱动表或是左表，表 b 被称为右表。如前所述，Spark 支持多种连接类型。</p>
<h3 data-nodeid="16092">小结</h3>
<p data-nodeid="16093">本课时主要介绍了 DataFrame、Dataset 与 Spark SQL，相对于 RDD 与算子，这种数据处理方式无疑对分析师来说更为友好，如果前面完成了习题，那么你的体会要更加深刻，而且这也是 Spark 官方推荐的 Spark API，无论是从性能还是从开发效率来说都是全方位领先于 RDD 与算子的组合，这也是很好理解的，举个例子，学习了 Python 后，想用 Python 直接进行数据分析无疑是很不方便的，Python 的 pandas 库则很好地解决了这个问题。</p>
<p data-nodeid="16094"><strong data-nodeid="16220">由于 Spark 对于 SQL 支持得非常好，而 pandas 在这方面没那么强大，所以，在某些场景，你可以选择 Spark SQL 来代替 pandas，这有时对于分析师来说非常好用。</strong></p>
<p data-nodeid="16095">最后给你出一个思考题，表中存有某一年某只股票的历史价格，表结构如下：</p>
<p data-nodeid="16096">股票 id，时间戳，成交价格</p>
<p data-nodeid="16097">问题是：请用一句 SQL 计算这只股票最长连续价格上涨天数，这里解释一个概念，如果某天的开盘价小于当天的收盘价，就认为这只股票当天是上涨的。这个需求看起来简单，但是用 SQL 写出来还是比较复杂的，需要用到子查询等内容，如果你能够完成这个思考题，我相信你能很好地使用 Spark SQL。</p>

---

### 精选评论

##### **超：
> 思考题挺有意思，写出来跟大家交流<div>表stock：</div><div><div>+-------+-------------------+------+</div><div>|company|date&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;|price |</div><div>+-------+-------------------+------+</div></div><div>sql:</div><div><div>with stock_rank as (</div><div>&nbsp; select company, from_unixtime(unix_timestamp(date,'yyyy/MM/dd HH:mm:ss'),'yyyy-MM-dd') as date, price,</div><div>&nbsp; --标记当天开盘价</div><div>&nbsp; row_number() over (partition by company,from_unixtime(unix_timestamp(date,'yyyy/MM/dd HH:mm:ss'),'yyyy-MM-dd') order by date) as rank_asc,</div><div>&nbsp; --标记当天收盘价</div><div>&nbsp; row_number() over (partition by company,from_unixtime(unix_timestamp(date,'yyyy/MM/dd HH:mm:ss'),'yyyy-MM-dd') order by date desc) as rank_desc</div><div>&nbsp; from stock</div><div>&nbsp; order by company, date, rank_asc</div><div>),</div><div>rise_date as (--哪一天上涨了</div><div>&nbsp; select a.company, a.date,&nbsp;</div><div>&nbsp; date_sub(a.date, dense_rank() over (partition by a.company order by a.date) - 1) as begin_date</div><div>&nbsp; from (select company, date, price from stock_rank where rank_asc=1) a</div><div>&nbsp; join (select company, date, price from stock_rank where rank_desc=1) b</div><div>&nbsp; on a.company=b.company and a.date=b.date</div><div>&nbsp; where a.price &lt; b.price</div><div>),</div><div>rise_day as (--连续上涨天数</div><div>select company, min(date) as start_date, count(*) as rise_days</div><div>from rise_date</div><div>group by company,begin_date</div><div>)</div><div>--最长连续价格上涨天数</div><div>select company, max(rise_days) as longest_rise</div><div>from rise_day</div><div>group by company</div></div>

##### *艺：
> 接上一条 使用价格下降的日期来做筛选： 日期间隔即为价格上涨 求取价格下降的日期间隔最大值即可！！！

##### *艺：
> 可将数据信息按照天数分组 合并最小时间和最大时间的价格（*-1） 若数值大于0 则说明开盘价格大于收盘价格 ，若数值小于0 则说明收盘价格大于开盘价格 今日上涨。至此可筛选出所有价格上涨的日期，使用lead函数向下取一行，求出两个日期之间差值，差值为1即连续。目前思路只到这里 后续来补充

##### jay：
> 棒

