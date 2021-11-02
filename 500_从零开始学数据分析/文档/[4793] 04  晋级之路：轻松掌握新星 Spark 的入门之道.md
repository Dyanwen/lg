<p data-nodeid="1761" class="">上一课时我们介绍了 Hive 以及常用技能，今天我们来学习下 Spark。因为 Spark 拥有强大的数据处理能力，近几年越来越多的互联网公司开始使用 Spark，掌握 Spark 基础技能，可以大大扩大你匹配数据分析和处理相关工作岗位的机会。</p>
<h3 data-nodeid="1762">初识 Spark</h3>
<p data-nodeid="1763">首先，我们来简单看下 Spark 的背景，它于 2009 年诞生于伯克利大学的 AMPLab，是一款基于内存计算的大数据并行计算框架，被设定为可用于构建一些大型的、低延迟的数据分析的应用程序。</p>
<p data-nodeid="1764">2013 年，Spark 在加入 Apache 孵化器项目后，凭借东风，开始迅速地发展。如今，Spark 已成为 Apache 最重要的三大分布式计算系统开源项目之一（三大分布计算系统为 Hadoop、Spark、Storm）。另一方面，自 2015 年开始， 很多公司部署以 Spark 代替 Hive、MapReduce 的战略。其中 Spark 开发语言 Scala，也因为 Spark 的走火，成为受大厂欢迎的技能。</p>
<h3 data-nodeid="1765">Spark 的优势所在</h3>
<p data-nodeid="1766">通过上节课我们了解到，Hive 任务和许多任务一样，也是被翻译成 MapReduce 后进行大数据处理的。Spark 相比 MapReduce，<strong data-nodeid="1921">最大的优势就是：快！</strong> 天下武功，唯快不破。Spark 占了“快”这个优点，因此获得了互联网大厂的青睐。</p>
<p data-nodeid="1767">Spark 的快表现在以下两点：</p>
<p data-nodeid="1768"><strong data-nodeid="1927">第一，执行速度快。</strong> 主要原因是 MR 作业对中间处理的数据需要落盘，下次再读取时。磁盘 I/O 成本高。相对而言，基于内存计算的 Spark 就不存在这个问题。对 Spark 而言，中间数据优先放到集群内存，只有数据量大到内存无法保存时，才会保存到磁盘。另外 MR 作业有多个 job， 每个 job 运行完后，下个 job 都需要重新申请计算资源。而 Spark 作业则会一直持续到数据处理结束，中间不需要再向集群申请计算资源。</p>
<p data-nodeid="1769">下面是 Spark 和 MR 性能比较，我们可以看出，对比 MR，Spark 仅使用了十分之一的计算资源，便获得了比 Hadoop 快 3 倍的速度。</p>
<p data-nodeid="1770"><img src="https://s0.lgstatic.com/i/image/M00/59/E7/CgqCHl9y00uACBmVAAMW2KU9DWQ697.png" alt="image (1).png" data-nodeid="1931"></p>
<p data-nodeid="1771"><strong data-nodeid="1936">第二，MR 程序编写的复杂度高。</strong> 比如一个基础统计需要几十行程序，Spark 则只需要几行程序，不用像 MR 那样配置各个隐藏在各个函数的操作，简洁高效。它不仅节省人力，而且专注数据逻辑处理。</p>
<p data-nodeid="1772">举个简单的例子。</p>
<ul data-nodeid="1773">
<li data-nodeid="1774">
<p data-nodeid="1775"><a href="https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html" data-nodeid="1940">Java 编写 MR</a></p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1776"><code data-language="java"><span class="hljs-keyword">import</span> java.io.IOException;
<span class="hljs-keyword">import</span> java.util.StringTokenizer;
<span class="hljs-keyword">import</span> org.apache.hadoop.conf.Configuration;
<span class="hljs-keyword">import</span> org.apache.hadoop.fs.Path;
<span class="hljs-keyword">import</span> org.apache.hadoop.io.IntWritable;
<span class="hljs-keyword">import</span> org.apache.hadoop.io.Text;
<span class="hljs-keyword">import</span> org.apache.hadoop.mapreduce.Job;
<span class="hljs-keyword">import</span> org.apache.hadoop.mapreduce.Mapper;
<span class="hljs-keyword">import</span> org.apache.hadoop.mapreduce.Reducer;
<span class="hljs-keyword">import</span> org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
<span class="hljs-keyword">import</span> org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WordCount</span> </span>{
  <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TokenizerMapper</span>
       <span class="hljs-keyword">extends</span> <span class="hljs-title">Mapper</span>&lt;<span class="hljs-title">Object</span>, <span class="hljs-title">Text</span>, <span class="hljs-title">Text</span>, <span class="hljs-title">IntWritable</span>&gt;</span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">static</span> IntWritable one = <span class="hljs-keyword">new</span> IntWritable(<span class="hljs-number">1</span>);
    <span class="hljs-keyword">private</span> Text word = <span class="hljs-keyword">new</span> Text();
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">map</span><span class="hljs-params">(Object key, Text value, Context context
                    )</span> <span class="hljs-keyword">throws</span> IOException, InterruptedException </span>{
      StringTokenizer itr = <span class="hljs-keyword">new</span> StringTokenizer(value.toString());
      <span class="hljs-keyword">while</span> (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
  <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">IntSumReducer</span>
       <span class="hljs-keyword">extends</span> <span class="hljs-title">Reducer</span>&lt;<span class="hljs-title">Text</span>,<span class="hljs-title">IntWritable</span>,<span class="hljs-title">Text</span>,<span class="hljs-title">IntWritable</span>&gt; </span>{
    <span class="hljs-keyword">private</span> IntWritable result = <span class="hljs-keyword">new</span> IntWritable();
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">reduce</span><span class="hljs-params">(Text key, Iterable&lt;IntWritable&gt; values,
                       Context context
                       )</span> <span class="hljs-keyword">throws</span> IOException, InterruptedException </span>{
      <span class="hljs-keyword">int</span> sum = <span class="hljs-number">0</span>;
      <span class="hljs-keyword">for</span> (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
    Configuration conf = <span class="hljs-keyword">new</span> Configuration();
    Job job = Job.getInstance(conf, <span class="hljs-string">"word count"</span>);
    job.setJarByClass(WordCount.class);
    job.setMapperClass(TokenizerMapper.class);
    job.setCombinerClass(IntSumReducer.class);
    job.setReducerClass(IntSumReducer.class);
    job.setOutputKeyClass(Text.class);
    job.setOutputValueClass(IntWritable.class);
    FileInputFormat.addInputPath(job, <span class="hljs-keyword">new</span> Path(args[<span class="hljs-number">0</span>]));
    FileOutputFormat.setOutputPath(job, <span class="hljs-keyword">new</span> Path(args[<span class="hljs-number">1</span>]));
    System.exit(job.waitForCompletion(<span class="hljs-keyword">true</span>) ? <span class="hljs-number">0</span> : <span class="hljs-number">1</span>);
  }
}
</code></pre>
<ul data-nodeid="1777">
<li data-nodeid="1778">
<p data-nodeid="1779">Spark 进行 WordCount 统计</p>
</li>
</ul>
<pre class="lang-scala" data-nodeid="1780"><code data-language="scala"><span class="hljs-keyword">val</span> lines = sc.textFile(<span class="hljs-string">"hdfs路径或本地文件路径"</span>)
<span class="hljs-keyword">val</span> res = lines.flatMap(line=&gt; line.split(<span class="hljs-string">" "</span>)) \
             .map(word=&gt; (word, <span class="hljs-number">1</span>)) \
             .reduceByKey(_ + _)
res.saveAsTextFile(<span class="hljs-string">"hdfs://...或本地文件路径"</span>)
</code></pre>
<p data-nodeid="1781">通过代码对比，我们明显能看出，Spark 比 Java 简洁不止一两行代码。</p>
<p data-nodeid="1782">MR 只有 map 和 reduce 两种操作，稍复杂的统计逻辑，就需要通过两种操作组合成多个 job 才能完成，另外还需要复杂的参数配置，最后使程序变得十分臃肿，开发成本很高。 Spark 提供多种转化操作方法，除了 map 和 reduce，还包括 flatMap、filter、union、join、reduceByKey、 sample、zip 等，这使得数据处理代码变得极其简洁。</p>
<p data-nodeid="1783">在互联网公司，时间就是成本，速度在某种程度上限制业务增长的瓶颈，这是老板最不想看到的。Spark 在大数据处理上的优势，不仅能够让数据流动得更快，而且因其语言上的简洁，更加提升了工程师开发效率，让数据挖掘的成本变得更低。大厂积极部署 Spark，反哺 Spark 开源社区，也使得 Spark 开源社区极其活跃，Spark 数据生态也越来越完善。</p>
<h3 data-nodeid="1784">Spark 数据生态完善</h3>
<p data-nodeid="1785">除了数据处理性能提升，Spark 还提供完善的大数据处理库套装，包括：Spark SQL、Spark Streaming（流式计算）、MLlib（机器学习）、GraphX （图计算）等， 这些功能也让 Spark 相比 Hadoop 优势更大了。</p>
<ul data-nodeid="1786">
<li data-nodeid="1787">
<p data-nodeid="1788"><strong data-nodeid="1951">Spark Core</strong>：Spark Core 包含 Spark 的基本功能，如内存计算、任务调度、部署模式、故障恢复、存储管理等。</p>
</li>
<li data-nodeid="1789">
<p data-nodeid="1790"><strong data-nodeid="1956">Spark SQL</strong>：Spark SQL 允许开发人员直接处理 RDD，同时也可查询 Hive、 HBase 等外部数据源。 Spark SQL 的一个重要特点是其能够统一处理关系表和 RDD，使得开发人员可以轻松地使用 SQL 命令进行查询，并进行更复杂的数据分析。</p>
</li>
<li data-nodeid="1791">
<p data-nodeid="1792"><strong data-nodeid="1961">Spark Streaming</strong>：Spark Streaming 支持高吞吐量、可容错处理的实时流数据处理，其核心思路是将流式计算分解成一系列短小的批处理作业。</p>
</li>
<li data-nodeid="1793">
<p data-nodeid="1794"><strong data-nodeid="1966">MLlib（机器学习）</strong>：MLlib 提供了常用机器学习算法的实现，包括聚类、分类、回归、协同过滤等。从而降低了机器学习的门槛，开发人员只要具备一定的理论知识就能进行机器学习的工作。</p>
</li>
<li data-nodeid="1795">
<p data-nodeid="1796"><strong data-nodeid="1971">GraphX（图计算）</strong>：GraphX 是 Spark 中用于图计算的 API ，可认为是 Pregel 在 Spark 上的重写及优化，Graphx 性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。</p>
</li>
</ul>
<p data-nodeid="1797">以上完善了 Spark 数据生态，让开发者通过 Spark 能挖掘更多数据价值，这也使 Spark 在同类产品中脱颖而出。</p>
<h3 data-nodeid="1798">Spark 应用场景</h3>
<p data-nodeid="1799">Spark 是一款基于内存的迭代计算框架，适用于需要多次操作特定数据集的应用场合。需要反复操作的次数越多，所需读取的数据量越大，受益越大，数据量小但是计算密集度较大的场合，受益就相对较小。</p>
<p data-nodeid="1800">Spark 两类应用场景：<strong data-nodeid="1984">离线统计</strong>和<strong data-nodeid="1985">实时计算</strong>。</p>
<ul data-nodeid="1801">
<li data-nodeid="1802">
<p data-nodeid="1803"><strong data-nodeid="1989">离线统计</strong></p>
</li>
</ul>
<p data-nodeid="1804">Spark 会应用在一些更底层的更大数据量离线任务中，在相同计算资源条件下，Spark 数据处理性能可以保证报表数据准时生产。</p>
<p data-nodeid="1805">举个例子，亿级日活用户的 App，每个用户使用 App 的产生的数据，包括浏览的内容、转评赞行为等都会形成日志上传。这部分用户的行为数据，需要经过许多层的数据任务处理才能最后呈现到数据分析师、产品、运营等业务方眼前。这时候，保证昨天的数据能被今天业务方及时看到，并推动各自业务制定决策。就显得尤为重要了。传统 Hive 遇到数据量较大的情况，处理效率会变低，这样就使得重要的数据指标延迟很久。昨天的数据有可能要今天下午、晚上甚至第 3 天才看到，严重影响业务决策。</p>
<p data-nodeid="1806">这时候 Spark 的优势就体现出来了。首先，使用 Spark 处理最底层日志的前几次统计聚合。如果聚合后数据量相对较小，再融合 Hive 数据任务进行处理。这样就能大幅提升数据产出效率，让数据能够及时为业务方所使用了。</p>
<ul data-nodeid="1807">
<li data-nodeid="1808">
<p data-nodeid="1809"><strong data-nodeid="1996">实时计算</strong></p>
</li>
</ul>
<p data-nodeid="1810">除了数据处理和统计，Spark 主要应用在互联网计算广告、准实时报表、机器学习等业务上。我们来看一下 Spark 在<strong data-nodeid="2006">广告业务</strong>和<strong data-nodeid="2007">准实时报表</strong>方面的运用。</p>
<p data-nodeid="1811"><strong data-nodeid="2012">广告业务</strong>：广告业务作为最依赖算法模型的业务，需要大量的互联网模型训练和预估，中间经过无数数据迭代，一份数据会被切分成很多部分，经历过很多轮训练。这时候 Spark 以最快速度完成模型训练的优势就能体现出来了。</p>
<p data-nodeid="1812"><strong data-nodeid="2017">准实时报表</strong>: 业务方希望实时掌控最近每分钟，App 各个功能模块被使用的次数，即沿着时间轴，按每分钟被切分成一段段，希望了解每分钟产生的各维度指标的统计。这个 Hive 完全无法完成。Spark Streaming 是专门为此场景设计的流式计算解决方案之一。</p>
<h3 data-nodeid="1813">Spark 专用编程语言 Scala</h3>
<p data-nodeid="1814">Spark 使用 Scala 语言进行实现。这里简单介绍下 Scala 语言。Scala 来源于 Java，但 Scala使用函数式编程思维来开发程序。 另外 Scala 程序可以编译成 .class 文件在 JVM 上运行。这点与 Java 相同，这两种语言大部分 API 是可以相互调用的。</p>
<p data-nodeid="1815">Scala 学习门槛相比 Java 来说要高一些。这里主要介绍与 Scala 的数据处理相关的函数和方法，目的在于带你了解 Scala 基本用法，从而方便学习 Spark 数据处理部分。对于 Scala 高阶函数使用方法感兴趣的同学，可以参考本文最后推荐的网站进行学习。</p>
<h4 data-nodeid="1816">表达式</h4>
<p data-nodeid="1817">我们几乎可以说 Scala 一切都是表达式。</p>
<pre class="lang-java" data-nodeid="1818"><code data-language="java">scala&gt; <span class="hljs-number">1</span>+<span class="hljs-number">2</span>
res1: Int = <span class="hljs-number">3</span>
</code></pre>
<p data-nodeid="1819">res1 是解释器自动创建的变量名称，用来指代表达式的计算结果。它是 Int 类型，值为3。</p>
<h4 data-nodeid="1820">变量类型</h4>
<p data-nodeid="1821">Scala 的变量类型只有两种类型：<strong data-nodeid="2034">val</strong>和<strong data-nodeid="2035">var</strong>。val 为不可变变量，我们来看下面的例子：</p>
<pre class="lang-scala" data-nodeid="1822"><code data-language="scala"><span class="hljs-comment">// 声明变量a, 可以将任何数据或者表达式赋值给a</span>
scala&gt; <span class="hljs-keyword">val</span> a = <span class="hljs-number">100</span>
a: <span class="hljs-type">Int</span> = <span class="hljs-number">100</span>
scala&gt; <span class="hljs-keyword">val</span> a = <span class="hljs-string">"hello scala"</span>
a: <span class="hljs-type">String</span> = hello scala
scala&gt; <span class="hljs-keyword">val</span> a = <span class="hljs-number">1</span> + <span class="hljs-number">2</span>
a: <span class="hljs-type">Int</span> = <span class="hljs-number">3</span>
<span class="hljs-comment">// 但不能对a再进行修改</span>
scala&gt; a = <span class="hljs-number">1</span>
&lt;console&gt;:<span class="hljs-number">25</span>: error: reassignment to <span class="hljs-keyword">val</span>
       a = <span class="hljs-number">1</span>
         ^
</code></pre>
<p data-nodeid="1823">var 为可变变量，即声明后可以再次修改的变量， 举例：</p>
<pre class="lang-scala" data-nodeid="1824"><code data-language="scala"><span class="hljs-comment">// 声明a为可变变量</span>
scala&gt; <span class="hljs-keyword">var</span> a = <span class="hljs-number">100</span>
a: <span class="hljs-type">Int</span> = <span class="hljs-number">100</span>
<span class="hljs-comment">// a的值, 可以被修改</span>
scala&gt; a = <span class="hljs-number">99</span>
a: <span class="hljs-type">Int</span> = <span class="hljs-number">99</span>
<span class="hljs-comment">// 但注意, a的类型, 不能被修改</span>
scala&gt; a = <span class="hljs-string">"hello"</span>
&lt;console&gt;:<span class="hljs-number">25</span>: error: <span class="hljs-class"><span class="hljs-keyword">type</span> <span class="hljs-title">mismatch</span></span>;
 found   : <span class="hljs-type">String</span>(<span class="hljs-string">"hello"</span>)
 required: <span class="hljs-type">Int</span>
       a = <span class="hljs-string">"hello"</span>
           ^
</code></pre>
<p data-nodeid="1825">注意：在数据处理过程中，能使用 val 的时候，尽量不使用 var。这是为了让多线程中的并发访问保持读写一致性。可变变量的值可以被其他逻辑修改，常量则不能被修改。因此整体上更为稳定一些。</p>
<h4 data-nodeid="1826">数据类型</h4>
<p data-nodeid="1827">Scala 的基础数据类型，如表格所示，主要包括 Byte、Short、Int、Long、Float、Double、Char、String、Boolean、Unit、Nul、Nothing 等。</p>
<p data-nodeid="1828"><img src="https://s0.lgstatic.com/i/image/M00/5B/E0/CgqCHl-AFEqAeB85AAETKXlAa00248.png" alt="Lark20201009-154106.png" data-nodeid="2042"></p>
<h4 data-nodeid="1829">基础数据结构</h4>
<p data-nodeid="1830">Scala 基础数据结构还包括：数组、列表、元组、集合，和其他高级语言类似。简单举个例子：</p>
<pre class="lang-scala" data-nodeid="1831"><code data-language="scala"><span class="hljs-comment">// 数组</span>
<span class="hljs-keyword">val</span> arr = <span class="hljs-keyword">new</span> <span class="hljs-type">Array</span>[<span class="hljs-type">Int</span>](<span class="hljs-number">3</span>)
scala&gt; <span class="hljs-keyword">val</span> arr = <span class="hljs-keyword">new</span> <span class="hljs-type">Array</span>[<span class="hljs-type">Int</span>](<span class="hljs-number">3</span>)
arr: <span class="hljs-type">Array</span>[<span class="hljs-type">Int</span>] = <span class="hljs-type">Array</span>(<span class="hljs-number">0</span>, <span class="hljs-number">0</span>, <span class="hljs-number">0</span>)
<span class="hljs-comment">// 给数组每个元素, 分别赋值(默认为0)</span>
scala&gt; arr(<span class="hljs-number">0</span>) = <span class="hljs-number">10</span>
scala&gt; arr(<span class="hljs-number">1</span>) = <span class="hljs-number">20</span>
scala&gt; arr(<span class="hljs-number">2</span>) = <span class="hljs-number">30</span>
scala&gt; arr
res5: <span class="hljs-type">Array</span>[<span class="hljs-type">Int</span>] = <span class="hljs-type">Array</span>(<span class="hljs-number">10</span>, <span class="hljs-number">20</span>, <span class="hljs-number">30</span>)
<span class="hljs-comment">// 列表</span>
scala&gt; <span class="hljs-keyword">val</span> list = <span class="hljs-type">List</span>(<span class="hljs-number">10</span>,<span class="hljs-number">20</span>,<span class="hljs-number">30</span>)
list: <span class="hljs-type">List</span>[<span class="hljs-type">Int</span>] = <span class="hljs-type">List</span>(<span class="hljs-number">10</span>, <span class="hljs-number">20</span>, <span class="hljs-number">30</span>)
scala&gt; list.sum
res10: <span class="hljs-type">Int</span> = <span class="hljs-number">60</span>
<span class="hljs-comment">// 元组</span>
scala&gt; <span class="hljs-keyword">val</span> tuple = ('a', <span class="hljs-number">100</span>, 'b', <span class="hljs-number">99</span>)
tuple: (<span class="hljs-type">Char</span>, <span class="hljs-type">Int</span>, <span class="hljs-type">Char</span>, <span class="hljs-type">Int</span>) = (a,<span class="hljs-number">100</span>,b,<span class="hljs-number">99</span>)
<span class="hljs-comment">// 集合</span>
scala&gt; <span class="hljs-keyword">var</span> set = <span class="hljs-type">Set</span>(<span class="hljs-string">"wang"</span>, <span class="hljs-string">"li"</span>, <span class="hljs-string">"zhang"</span>)
set: scala.collection.immutable.<span class="hljs-type">Set</span>[<span class="hljs-type">String</span>] = <span class="hljs-type">Set</span>(wang, li, zhang)
<span class="hljs-comment">// 增加重复的元素"li"， 结果不变</span>
scala&gt; set+=<span class="hljs-string">"li"</span>
scala&gt; set
res7: scala.collection.immutable.<span class="hljs-type">Set</span>[<span class="hljs-type">String</span>] = <span class="hljs-type">Set</span>(wang, li, zhang)
<span class="hljs-comment">// 增加不同元素, 会体现出来</span>
scala&gt; set+=<span class="hljs-string">"sun"</span>
scala&gt; set
res9: scala.collection.immutable.<span class="hljs-type">Set</span>[<span class="hljs-type">String</span>] = <span class="hljs-type">Set</span>(wang, li, zhang, sun)
</code></pre>
<p data-nodeid="1832">以上介绍了 Scala 的基础变量、数据类型和数据结构。其实函数方法调用对于每种数据结构都集成了，这里就不细说了。你可以在交互式命令行进行手敲学习，多实践几次就熟练了。<br>
有了对 Scala 语言基础认知，我们下面再开始介绍 Spark 如何进行数据处理的。</p>
<h3 data-nodeid="1833">Spark 数据处理实例</h3>
<p data-nodeid="1834">Spark 的核心是建立在统一的抽象 RDD (Resilient Distributed Dataset) 之上，RDD 全称为<strong data-nodeid="2054">弹性分布式数据集。</strong> RDD 是一个不可变、可分区、里面元素可以并行计算的集合。它使 Spark 的各个组件可以无缝进行集成（现在 Spark 3.0 之后开始倡导 DataSet ，但理解 RDD 能够帮助我们更好地理解 Spark）。</p>
<p data-nodeid="1835">Spark 数据处理的基本流程：第一步，创建 RDD；第二步，对 RDD 进行数据处理，并最终生成想要的数据结果。</p>
<h4 data-nodeid="1836">创建 RDD</h4>
<p data-nodeid="1837">我们可以从本机或者集群获取数据创建 RDD 。</p>
<p data-nodeid="1838">第一，从本机创建 RDD。参考下面代码：</p>
<pre class="lang-java" data-nodeid="1839"><code data-language="java"><span class="hljs-comment">// sc 是Spark交互界面自带的 sparkContext, 它有所有spark内部函数可调用</span>
scala&gt; val lines = sc.textFile(<span class="hljs-string">"file:///usr/local/.../word.txt"</span>)
</code></pre>
<p data-nodeid="1840">第二，从集群创建 RDD。以下代码进行了说明。</p>
<pre class="lang-java" data-nodeid="1841"><code data-language="java">scala&gt; val lines = sc.textFile(<span class="hljs-string">"hdfs:///usr/xxx/.../word.txt"</span>)
</code></pre>
<p data-nodeid="1842">第三，通过数组、集合、列表等创建 RDD。可以参考一下代码。</p>
<pre class="lang-java" data-nodeid="1843"><code data-language="java"><span class="hljs-comment">// 声明一个列表变量</span>
scala&gt; val list = List(<span class="hljs-number">10</span>, <span class="hljs-number">20</span>, <span class="hljs-number">30</span>)
list: List[Int] = List(<span class="hljs-number">10</span>, <span class="hljs-number">20</span>, <span class="hljs-number">30</span>)
<span class="hljs-comment">// 通过函数parallelize将列表转化成RDD</span>
scala&gt; val rdd = sc.parallelize(list)
rdd: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[<span class="hljs-number">18</span>] at parallelize at &lt;console&gt;:<span class="hljs-number">26</span>
</code></pre>
<h4 data-nodeid="1844">RDD 转换 ( transformations ) 和行动 ( actions )</h4>
<p data-nodeid="1845">详细可参考：<a href="https://spark.apache.org/docs/latest/rdd-programming-guide.html" data-nodeid="2065">spark 官网：rdd-programming-guide</a></p>
<p data-nodeid="1846">transformations 常用的 API 有以下形式：</p>
<ul data-nodeid="1847">
<li data-nodeid="1848">
<p data-nodeid="1849"><strong data-nodeid="2071">map(func)</strong>：通过 func 函数，对 RDD 数据转换生成新 RDD，可以简单理解为 Python 中 lambda 函数。</p>
</li>
<li data-nodeid="1850">
<p data-nodeid="1851"><strong data-nodeid="2076">filter(func)</strong>：对于原有 RDD 中，满足经过 func 处理后返回 True 的数据保留下来，生成新数据集合。</p>
</li>
<li data-nodeid="1852">
<p data-nodeid="1853"><strong data-nodeid="2081">flatMap(func)</strong>：将原有数据打平，可以简单理解为行转列。</p>
</li>
<li data-nodeid="1854">
<p data-nodeid="1855"><strong data-nodeid="2088">sample(withReplacement,&nbsp;fraction,&nbsp;seed)</strong>：对原有 RDD 抽样，生成新 RDD<br>
union(otherDataset)： 将两个 RDD 合并后生成新 RDD。</p>
</li>
<li data-nodeid="1856">
<p data-nodeid="1857"><strong data-nodeid="2097">reduceByKey(func, [numPartitions])</strong>：针对( K, V )样式 RDD，将 K 作为被聚合的 key，计算 V 的值。</p>
</li>
<li data-nodeid="1858">
<p data-nodeid="1859"><strong data-nodeid="2102">repartition(numPartitions)</strong>：将原有 RDD 重新组合到不同的分区中，比如原来 RDD 在 1000 个分区上，小文件过多，但实际只有几 MB 数据，这时一般会 reshuffle 到一个分区即可。</p>
</li>
</ul>
<p data-nodeid="1860">actions 常用 API 主要有以下形式：</p>
<ul data-nodeid="1861">
<li data-nodeid="1862">
<p data-nodeid="1863"><strong data-nodeid="2108">reduce(func)</strong>：使用函数来聚合 RDD 相关的元素。</p>
</li>
<li data-nodeid="1864">
<p data-nodeid="1865"><strong data-nodeid="2113">collect()</strong>：将数据集的所有元素作为数组返回。</p>
</li>
<li data-nodeid="1866">
<p data-nodeid="1867">null<br>
<strong data-nodeid="2120">count()</strong>：返回 RDD 元素数量。</p>
</li>
<li data-nodeid="1868">
<p data-nodeid="1869"><strong data-nodeid="2125">first()</strong>：取 RDD 第一个元素。</p>
</li>
<li data-nodeid="1870">
<p data-nodeid="1871"><strong data-nodeid="2130">take(n)</strong>：取 RDD 第 N 个元素。</p>
</li>
<li data-nodeid="1872">
<p data-nodeid="1873"><strong data-nodeid="2135">saveAsTextFile(path)</strong>：将处理后的 RDD 保存到 HDFS 路径。</p>
</li>
<li data-nodeid="1874">
<p data-nodeid="1875"><strong data-nodeid="2140">foreach(println)</strong>：对 RDD 中每个元素，使用 func 进行处理，比如打印每个元素。</p>
</li>
</ul>
<p data-nodeid="1876">RDD 转换操作，即 transformations 操作，是指将 RDD 由一种数据结构样式转换成另外一种样式，每次转换会生成新的 RDD 供下次 transformations 使用。需要注意的是，<strong data-nodeid="2146">通过transformations 生成的 RDD 是惰性求值的</strong>，可以简单理解为一个 RDD 经过各种transformations ，只是记录了数据转换的过程，不会触发计算。只有最终遇到 actions 操作，才会发送真正的计算。</p>
<p data-nodeid="1877">接下来我们通过实际案例，了解下 transformations 和 actions 函数的使用。</p>
<h4 data-nodeid="1878">数据源介绍</h4>
<pre class="lang-scala" data-nodeid="1879"><code data-language="scala"><span class="hljs-comment">// </span>
scala&gt; <span class="hljs-keyword">val</span> rdd = sc.textFile(<span class="hljs-string">"/Users/leihao1/data/qujinger/专栏课程/example-city"</span>)
<span class="hljs-comment">// take, foreach用法举例：打印前3行 </span>
scala&gt; rdd.take(<span class="hljs-number">3</span>).foreach(println)
<span class="hljs-number">2019</span>/<span class="hljs-number">8</span>/<span class="hljs-number">14</span>	浙江省	<span class="hljs-number">861225</span>
<span class="hljs-number">2019</span>/<span class="hljs-number">10</span>/<span class="hljs-number">24</span>	广东省	<span class="hljs-number">767495</span>
<span class="hljs-number">2020</span>/<span class="hljs-number">3</span>/<span class="hljs-number">18</span>	河北省	<span class="hljs-number">767495</span>
</code></pre>
<p data-nodeid="1880">数据源中共三列，分别是：日期、省份、用户访问次数。下面通过 Spark 来完成下面几个统计任务：</p>
<ol data-nodeid="1881">
<li data-nodeid="1882">
<p data-nodeid="1883">数据总行数。</p>
</li>
<li data-nodeid="1884">
<p data-nodeid="1885">各省份的访问次数总和。</p>
</li>
<li data-nodeid="1886">
<p data-nodeid="1887">过滤出大于 50 万次访问次数的省份。</p>
</li>
</ol>
<p data-nodeid="1888">这些都是简单而普遍的统计场景，我们看看 Spark 是如何完成的。</p>
<p data-nodeid="1889"><strong data-nodeid="2159">第一步：读取数据。</strong><br>
Spark 在进行数据处理前，也需要先加载数据，可以从本机读取数据，也可以从分布时集群获取数据。</p>
<p data-nodeid="1890">下面演示的是读取本地的练习数据。如果希望读 HDFS 数据，只需要将本地路径替换成 HDFS 路径即可。</p>
<pre class="lang-scala" data-nodeid="1891"><code data-language="scala"><span class="hljs-comment">// map用法举例： 通过map将原来rdd转化为另外一个rdd，只取后两列数据</span>
scala&gt; <span class="hljs-keyword">val</span> rdd1 = data.map(line =&gt; line.split(<span class="hljs-string">"\t"</span>)).map(arr =&gt; (arr(<span class="hljs-number">1</span>), arr(<span class="hljs-number">2</span>)))
rdd1: org.apache.spark.rdd.<span class="hljs-type">RDD</span>[(<span class="hljs-type">String</span>, <span class="hljs-type">String</span>)] = <span class="hljs-type">MapPartitionsRDD</span>[<span class="hljs-number">26</span>] at map at &lt;console&gt;:<span class="hljs-number">27</span>
scala&gt; rdd1.take(<span class="hljs-number">3</span>).foreach(println)
(浙江省,<span class="hljs-number">861225</span>)
(广东省,<span class="hljs-number">767495</span>)
(河北省,<span class="hljs-number">7674</span>)
</code></pre>
<p data-nodeid="1892"><strong data-nodeid="2167">第二步：行数统计</strong>。<br>
数据分析师最常用聚合函数完成数据统计。比如，查询一个表的总函数，计算某个字段数据之和。Spark 也有类似的基础功能。下面，我们通过 Spark 来统计数据源的总行数，以及 pv 字段的加和，也就是看总的浏览次数。</p>
<pre class="lang-scala" data-nodeid="1893"><code data-language="scala"><span class="hljs-comment">// count用法举例：计算行数</span>
scala&gt; rdd1.count
res33: <span class="hljs-type">Long</span> = <span class="hljs-number">20</span>
<span class="hljs-comment">// reduce用法举例：计算总pv数, 即将最后一列数字求和</span>
scala&gt; rdd1.map{<span class="hljs-keyword">case</span>(city, pv) =&gt; pv.toLong}.reduce(_+_)
res37: <span class="hljs-type">Long</span> = <span class="hljs-number">7806</span>
</code></pre>
<p data-nodeid="1894"><strong data-nodeid="2173">第三步：分组聚合。</strong><br>
Spark 分组聚合函数是用 reduceByKey 实现的，类似 SQL 语言里的 group by 用法，但又远比 SQL 功能强大。下面我们通过 reduceByKey 来分组汇总下各个省份的总流量。</p>
<pre class="lang-scala te-preview-highlight" data-nodeid="2201"><code data-language="scala"><span class="hljs-comment">// reduceByKey用法举例：统计各省份的总行数, 总pv数</span>
scala&gt; rdd1.map{<span class="hljs-keyword">case</span>(province, pv) =&gt;
     | (province, (<span class="hljs-number">1</span>, pv.toLong))
     | }.reduceByKey{<span class="hljs-keyword">case</span>((n1, pv1), (n2, pv2)) =&gt;
     | (n1+n2, pv1+pv2)
     | }.foreach(println)
(湖北省,(<span class="hljs-number">1</span>,<span class="hljs-number">541376</span>))
(江西省,(<span class="hljs-number">1</span>,<span class="hljs-number">371881</span>))
(浙江省,(<span class="hljs-number">1</span>,<span class="hljs-number">861225</span>))
(湖南省,(<span class="hljs-number">1</span>,<span class="hljs-number">79023</span>))
(江苏省,(<span class="hljs-number">1</span>,<span class="hljs-number">371881</span>))
(华盛顿,(<span class="hljs-number">1</span>,<span class="hljs-number">371881</span>))
(重庆,(<span class="hljs-number">1</span>,<span class="hljs-number">371881</span>))
(山东省,(<span class="hljs-number">1</span>,<span class="hljs-number">541376</span>))
(河北省,(<span class="hljs-number">1</span>,<span class="hljs-number">767495</span>))
(广东省,(<span class="hljs-number">4</span>,<span class="hljs-number">1759775</span>))
(上海,(<span class="hljs-number">6</span>,<span class="hljs-number">1396543</span>))
(河南省,(<span class="hljs-number">1</span>,<span class="hljs-number">3718</span>
</code></pre>

<p data-nodeid="1896"><strong data-nodeid="2180">第四步：完成过滤</strong>。<br>
数据分析师在查询数据的时候，都会用到条件过滤来获取满足自己查询目标，即 SQL 语言中的 where 条件语句。在 Spark 中的过滤方式有很多种， 其中最常用的是 filter 函数。下面我们，通过 filter 函数来过滤出满足 50 万浏览量的数据行：</p>
<pre class="lang-scala" data-nodeid="1897"><code data-language="scala"><span class="hljs-comment">// filter用法举例：过滤函数</span>
scala&gt; rdd1.filter{<span class="hljs-keyword">case</span>(_, pv) =&gt; pv.toLong&gt;=<span class="hljs-number">500000</span>}.foreach(println)
(浙江省,<span class="hljs-number">861225</span>)
(广东省,<span class="hljs-number">767495</span>)
(河北省,<span class="hljs-number">767495</span>)
(山东省,<span class="hljs-number">541376</span>)
(广东省,<span class="hljs-number">541376</span>)
(上海,<span class="hljs-number">541376</span>)
(湖北省,<span class="hljs-number">541376</span>)
</code></pre>
<p data-nodeid="1898">以上，通过 Spark 来完成数据分析师熟悉的几个统计 case ，相信你会对 Spark 数据处理有更直观的认识。在实际工作中，Spark 处理的数据量和复杂度可能会更高。在提交 Spark 任务时，肯定会涉及基础资源配置、内存调优、数据倾斜处理等问题，这里不展开讲了，我们只需要了解，Spark 基本上就是按照上面方式，再进行组合和优化，完成基础数据处理的。</p>
<h3 data-nodeid="1899">总结</h3>
<p data-nodeid="1900">本课时先介绍到这里，希望通过学习，能让你了解到 Spark 的优势，使用场景，以及如何使用 Spark 进行大数据处理。非常感谢你的学习，如果有关于 Spark 数据处理的一些问题，欢迎在留言区提问。</p>
<p data-nodeid="1901">最后给你推荐一些学习网站。Spark 生态很强大，对于数据分析师来讲，可以重点了解 Spark 数据处理的部分。对机器学习、流失计算感兴趣的同学，则可以进一步学习 MLlib、Spark Streaming。而官网是最好最全面的学习文档，感兴趣的同学可以点击以下链接，通过网站进行进一步学习。</p>
<ul data-nodeid="1902">
<li data-nodeid="1903">
<p data-nodeid="1904"><a href="http://spark.apache.org/examples.html#" data-nodeid="2187">Spark 官网：Examples</a>。</p>
</li>
<li data-nodeid="1905">
<p data-nodeid="1906"><a href="https://spark-reference-doc-cn.readthedocs.io/zh_CN/latest/programming-guide/quick-start.html" data-nodeid="2191">Spark 入门中文文档</a>。</p>
</li>
<li data-nodeid="1907">
<p data-nodeid="1908"><a href="https://docs.scala-lang.org/overviews/scala-book/introduction.html" data-nodeid="2195">Scala 官网</a>。</p>
</li>
<li data-nodeid="1909">
<p data-nodeid="1910" class=""><a href="http://twitter.github.io/scala_school/zh_cn/" data-nodeid="2199">twitter scala 课堂</a>。</p>
</li>
</ul>

---

### 精选评论

##### *潜：
> 数据分析师代码能力要这么强啊…

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 别吓到你，其实不用，但最起码SQL能力一定要有。python，shell要了解一点

##### **杰：
> 老师，请问下仅使用python的工具包pyspark进行相应的大数据分析，找工作会有局限吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对于spark有要求的岗位来说，不会的。

##### **铭：
> 好强啊🙏🙏🙏

##### **伟：
> to B和 to C的数据分析有何侧重点？能对比讲一下么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; toC: 更关注用户个体行为，拉新留存转化
toB: 更多以企业整体考虑，企业是要考虑ROI，盈利等商业化问题，视角不一样，另外toB很多业务细节是在线下，且不同企业客户需求差异比较大，可能会定制化

