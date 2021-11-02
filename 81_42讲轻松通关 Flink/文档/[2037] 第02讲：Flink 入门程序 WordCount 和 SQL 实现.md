<p>本课时我们主要介绍 Flink 的入门程序以及 SQL 形式的实现。</p>
<p>上一课时已经讲解了 Flink 的常用应用场景和架构模型设计，这一课时我们将会从一个最简单的 WordCount 案例作为切入点，并且同时使用 SQL 方式进行实现，为后面的实战课程打好基础。</p>
<p>我们首先会从环境搭建入手，介绍如何搭建本地调试环境的脚手架；然后分别从DataSet（批处理）和 DataStream（流处理）两种方式如何进行单词计数开发；最后介绍 Flink Table 和 SQL 的使用。</p>
<h3>Flink 开发环境</h3>
<p>通常来讲，任何一门大数据框架在实际生产环境中都是以集群的形式运行，而我们调试代码大多数会在本地搭建一个模板工程，Flink 也不例外。</p>
<p>Flink 一个以 Java 及 Scala 作为开发语言的开源大数据项目，通常我们推荐使用 Java 来作为开发语言，Maven 作为编译和包管理工具进行项目构建和编译。对于大多数开发者而言，JDK、Maven 和 Git 这三个开发工具是必不可少的。</p>
<p>关于 JDK、Maven 和 Git 的安装建议如下表所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/89/F7/Cgq2xl6ZFEeAILbpAABJUTt9qUU172.png" alt=""></p>
<h4>工程创建</h4>
<p>一般来说，我们在通过 IDE 创建工程，可以自己新建工程，添加 Maven 依赖，或者直接用 mvn 命令创建应用：</p>
<pre><code data-language="java" class="lang-java">mvn&nbsp;&nbsp;&nbsp;archetype:generate&nbsp;&nbsp;\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-DarchetypeGroupId=org.apache.flink&nbsp;\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-DarchetypeArtifactId=flink-quickstart-java&nbsp;\
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-DarchetypeVersion=<span class="hljs-number">1.10</span><span class="hljs-number">.0</span>
</code></pre>
<p>通过指定 Maven 工程的三要素，即 GroupId、ArtifactId、Version 来创建一个新的工程。同时 Flink 给我提供了更为方便的创建 Flink 工程的方法：</p>
<pre><code data-language="java" class="lang-java">curl&nbsp;https:<span class="hljs-comment">//flink.apache.org/q/quickstart.sh&nbsp;|&nbsp;bash&nbsp;-s&nbsp;1.10.0</span>
</code></pre>
<p>我们在终端直接执行该命令：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/03/70/CgoCgV6YJryAT7kwAAM4SZRfXz0562.png" alt=""></p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/10/9F/Ciqah16YJr2Afd28AAJHVzwSLKg287.png" alt=""></p>
<p>直接出现 Build Success 信息，我们可以在本地目录看到一个已经生成好的名为 <strong>quickstart</strong> &nbsp;的工程。</p>
<p>这里需要的主要的是，自动生成的项目 pom.xml 文件中对于 Flink 的依赖注释掉 scope：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-java<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>${flink.version}<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&lt;scope&gt;provided&lt;/scope&gt;--&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-streaming-java_${scala.binary.version}<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>${flink.version}<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&lt;scope&gt;provided&lt;/scope&gt;--&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<h4>DataSet WordCount</h4>
<p>WordCount 程序是大数据处理框架的入门程序，俗称“<strong>单词计数</strong>”。用来统计一段文字每个单词的出现次数，该程序主要分为两个部分：一部分是将文字拆分成单词；另一部分是单词进行分组计数并打印输出结果。</p>
<p>整体代码实现如下：</p>
<pre><code data-language="java" class="lang-java">&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;创建Flink运行的上下文环境</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;ExecutionEnvironment&nbsp;env&nbsp;=&nbsp;ExecutionEnvironment.getExecutionEnvironment();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;创建DataSet，这里我们的输入是一行一行的文本</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataSet&lt;String&gt;&nbsp;text&nbsp;=&nbsp;env.fromElements(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Flink&nbsp;Spark&nbsp;Storm"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Flink&nbsp;Flink&nbsp;Flink"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Spark&nbsp;Spark&nbsp;Spark"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Storm&nbsp;Storm&nbsp;Storm"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;通过Flink内置的转换函数进行计算</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataSet&lt;Tuple2&lt;String,&nbsp;Integer&gt;&gt;&nbsp;counts&nbsp;=
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;text.flatMap(<span class="hljs-keyword">new</span>&nbsp;LineSplitter())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.groupBy(<span class="hljs-number">0</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.sum(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//结果打印</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counts.printToErr();

&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">LineSplitter</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">FlatMapFunction</span>&lt;<span class="hljs-title">String</span>,&nbsp;<span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>,&nbsp;<span class="hljs-title">Integer</span>&gt;&gt;&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">flatMap</span><span class="hljs-params">(String&nbsp;value,&nbsp;Collector&lt;Tuple2&lt;String,&nbsp;Integer&gt;&gt;&nbsp;out)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将文本分割</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String[]&nbsp;tokens&nbsp;=&nbsp;value.toLowerCase().split(<span class="hljs-string">"\\W+"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(String&nbsp;token&nbsp;:&nbsp;tokens)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(token.length()&nbsp;&gt;&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out.collect(<span class="hljs-keyword">new</span>&nbsp;Tuple2&lt;String,&nbsp;Integer&gt;(token,&nbsp;<span class="hljs-number">1</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
</code></pre>
<p>实现的整个过程中分为以下几个步骤。</p>
<p>首先，我们需要创建 Flink 的上下文运行环境：</p>
<pre><code data-language="java" class="lang-java">ExecutionEnvironment&nbsp;env&nbsp;=&nbsp;ExecutionEnvironment.getExecutionEnvironment();
</code></pre>
<p>然后，使用 fromElements 函数创建一个 DataSet 对象，该对象中包含了我们的输入，使用 FlatMap、GroupBy、SUM 函数进行转换。</p>
<p>最后，直接在控制台打印输出。</p>
<p>我们可以直接右键运行一下 main 方法，在控制台会出现我们打印的计算结果：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/89/B5/Cgq2xl6YJr2ABr1NAAqJItsbg-g629.png" alt=""></p>
<h4>DataStream WordCount</h4>
<p>为了模仿一个流式计算环境，我们选择监听一个本地的 Socket 端口，并且使用 Flink 中的滚动窗口，每 5 秒打印一次计算结果。代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">StreamingJob</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;创建Flink的流式计算环境</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;StreamExecutionEnvironment&nbsp;env&nbsp;=&nbsp;StreamExecutionEnvironment.getExecutionEnvironment();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;监听本地9000端口</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataStream&lt;String&gt;&nbsp;text&nbsp;=&nbsp;env.socketTextStream(<span class="hljs-string">"127.0.0.1"</span>,&nbsp;<span class="hljs-number">9000</span>,&nbsp;<span class="hljs-string">"\n"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将接收的数据进行拆分，分组，窗口计算并且进行聚合输出</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataStream&lt;WordWithCount&gt;&nbsp;windowCounts&nbsp;=&nbsp;text
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.flatMap(<span class="hljs-keyword">new</span>&nbsp;FlatMapFunction&lt;String,&nbsp;WordWithCount&gt;()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">flatMap</span><span class="hljs-params">(String&nbsp;value,&nbsp;Collector&lt;WordWithCount&gt;&nbsp;out)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(String&nbsp;word&nbsp;:&nbsp;value.split(<span class="hljs-string">"\\s"</span>))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;out.collect(<span class="hljs-keyword">new</span>&nbsp;WordWithCount(word,&nbsp;<span class="hljs-number">1L</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.keyBy(<span class="hljs-string">"word"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.timeWindow(Time.seconds(<span class="hljs-number">5</span>),&nbsp;Time.seconds(<span class="hljs-number">1</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.reduce(<span class="hljs-keyword">new</span>&nbsp;ReduceFunction&lt;WordWithCount&gt;()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;WordWithCount&nbsp;<span class="hljs-title">reduce</span><span class="hljs-params">(WordWithCount&nbsp;a,&nbsp;WordWithCount&nbsp;b)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">new</span>&nbsp;WordWithCount(a.word,&nbsp;a.count&nbsp;+&nbsp;b.count);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;打印结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;windowCounts.print().setParallelism(<span class="hljs-number">1</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;env.execute(<span class="hljs-string">"Socket&nbsp;Window&nbsp;WordCount"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Data&nbsp;type&nbsp;for&nbsp;words&nbsp;with&nbsp;count</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">WordWithCount</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;String&nbsp;word;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;count;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">WordWithCount</span><span class="hljs-params">()</span>&nbsp;</span>{}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">WordWithCount</span><span class="hljs-params">(String&nbsp;word,&nbsp;<span class="hljs-keyword">long</span>&nbsp;count)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.word&nbsp;=&nbsp;word;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.count&nbsp;=&nbsp;count;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">toString</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;word&nbsp;+&nbsp;<span class="hljs-string">"&nbsp;:&nbsp;"</span>&nbsp;+&nbsp;count;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>整个<strong>流式计算的过程</strong>分为以下几步。</p>
<p>首先创建一个流式计算环境：</p>
<pre><code data-language="java" class="lang-java">StreamExecutionEnvironment&nbsp;env&nbsp;=&nbsp;StreamExecutionEnvironment.getExecutionEnvironment();
</code></pre>
<p>然后进行监听本地 9000 端口，将接收的数据进行拆分、分组、窗口计算并且进行聚合输出。代码中使用了 Flink 的窗口函数，我们在后面的课程中将详细讲解。</p>
<p>我们在本地使用 <strong>netcat</strong> 命令启动一个端口：</p>
<pre><code data-language="java" class="lang-java">nc&nbsp;-lk&nbsp;<span class="hljs-number">9000</span>
</code></pre>
<p>然后直接运行我们的 main 方法：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/03/70/CgoCgV6YJr2ARVjhAA2Bl2R-xW4872.png" alt=""></p>
<p>可以看到，工程启动后开始监听 127.0.0.1 的 9000 端口。</p>
<p>在 nc 中输入：</p>
<pre><code data-language="java" class="lang-java">$&nbsp;nc&nbsp;-lk&nbsp;<span class="hljs-number">9000</span>
Flink&nbsp;Flink&nbsp;Flink&nbsp;
Flink&nbsp;Spark&nbsp;Storm
</code></pre>
<p>可以在控制台看到：</p>
<pre><code data-language="java" class="lang-java">Flink&nbsp;:&nbsp;<span class="hljs-number">4</span>
Spark&nbsp;:&nbsp;<span class="hljs-number">1</span>
Storm&nbsp;:&nbsp;<span class="hljs-number">1</span>
</code></pre>
<h4>Flink Table &amp; SQL WordCount</h4>
<p>Flink SQL 是 Flink 实时计算为简化计算模型，降低用户使用实时计算门槛而设计的一套符合标准 SQL 语义的开发语言。</p>
<p>一个完整的 Flink SQL 编写的程序包括如下三部分。</p>
<ul>
<li><strong>Source Operator</strong>：是对外部数据源的抽象, 目前 Apache Flink 内置了很多常用的数据源实现，比如 MySQL、Kafka 等。</li>
<li><strong>Transformation Operators</strong>：算子操作主要完成比如查询、聚合操作等，目前 Flink SQL 支持了 Union、Join、Projection、Difference、Intersection 及 window 等大多数传统数据库支持的操作。</li>
<li><strong>Sink Operator</strong>：是对外结果表的抽象，目前 Apache Flink 也内置了很多常用的结果表的抽象，比如 Kafka Sink 等。</li>
</ul>
<p>我们也是通过用一个最经典的 WordCount 程序作为入门，上面已经通过 DataSet/DataStream API 开发，那么实现同样的 WordCount 功能， Flink Table &amp; SQL 核心只需要一行代码：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//省略掉初始化环境等公共代码</span>
SELECT&nbsp;word,&nbsp;COUNT(word)&nbsp;FROM&nbsp;table&nbsp;GROUP&nbsp;BY&nbsp;word;
</code></pre>
<p>首先，整个工程中我们 pom 中的依赖如下图所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-java<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-streaming-java_2.11
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-table-api-java-bridge_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-table-planner-blink_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-table-planner_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-table-api-scala-bridge_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.10.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p>第一步，创建上下文环境：</p>
<pre><code data-language="java" class="lang-java">ExecutionEnvironment&nbsp;fbEnv&nbsp;=&nbsp;ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment&nbsp;fbTableEnv&nbsp;=&nbsp;BatchTableEnvironment.create(fbEnv);
</code></pre>
<p>第二步，读取一行模拟数据作为输入：</p>
<pre><code data-language="java" class="lang-java">String&nbsp;words&nbsp;=&nbsp;<span class="hljs-string">"hello&nbsp;flink&nbsp;hello&nbsp;lagou"</span>;
String[]&nbsp;split&nbsp;=&nbsp;words.split(<span class="hljs-string">"\\W+"</span>);
ArrayList&lt;WC&gt;&nbsp;list&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayList&lt;&gt;();

<span class="hljs-keyword">for</span>(String&nbsp;word&nbsp;:&nbsp;split){
&nbsp;&nbsp;&nbsp;&nbsp;WC&nbsp;wc&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;WC(word,<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;list.add(wc);
}
DataSet&lt;WC&gt;&nbsp;input&nbsp;=&nbsp;fbEnv.fromCollection(list);
</code></pre>
<p>第三步，注册成表，执行 SQL，然后输出：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//DataSet&nbsp;转sql,&nbsp;指定字段名</span>
Table&nbsp;table&nbsp;=&nbsp;fbTableEnv.fromDataSet(input,&nbsp;<span class="hljs-string">"word,frequency"</span>);
table.printSchema();

<span class="hljs-comment">//注册为一个表</span>
fbTableEnv.createTemporaryView(<span class="hljs-string">"WordCount"</span>,&nbsp;table);

Table&nbsp;table02&nbsp;=&nbsp;fbTableEnv.sqlQuery(<span class="hljs-string">"select&nbsp;word&nbsp;as&nbsp;word,&nbsp;sum(frequency)&nbsp;as&nbsp;frequency&nbsp;from&nbsp;WordCount&nbsp;GROUP&nbsp;BY&nbsp;word"</span>);

<span class="hljs-comment">//将表转换DataSet</span>
DataSet&lt;WC&gt;&nbsp;ds3&nbsp;&nbsp;=&nbsp;fbTableEnv.toDataSet(table02,&nbsp;WC<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
ds3.printToErr();
</code></pre>
<p>整体代码结构如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">WordCountSQL</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception</span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//获取运行环境</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ExecutionEnvironment&nbsp;fbEnv&nbsp;=&nbsp;ExecutionEnvironment.getExecutionEnvironment();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建一个tableEnvironment</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BatchTableEnvironment&nbsp;fbTableEnv&nbsp;=&nbsp;BatchTableEnvironment.create(fbEnv);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;words&nbsp;=&nbsp;<span class="hljs-string">"hello&nbsp;flink&nbsp;hello&nbsp;lagou"</span>;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String[]&nbsp;split&nbsp;=&nbsp;words.split(<span class="hljs-string">"\\W+"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayList&lt;WC&gt;&nbsp;list&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayList&lt;&gt;();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>(String&nbsp;word&nbsp;:&nbsp;split){
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WC&nbsp;wc&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;WC(word,<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;list.add(wc);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataSet&lt;WC&gt;&nbsp;input&nbsp;=&nbsp;fbEnv.fromCollection(list);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//DataSet&nbsp;转sql,&nbsp;指定字段名</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table&nbsp;table&nbsp;=&nbsp;fbTableEnv.fromDataSet(input,&nbsp;<span class="hljs-string">"word,frequency"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;table.printSchema();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//注册为一个表</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fbTableEnv.createTemporaryView(<span class="hljs-string">"WordCount"</span>,&nbsp;table);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table&nbsp;table02&nbsp;=&nbsp;fbTableEnv.sqlQuery(<span class="hljs-string">"select&nbsp;word&nbsp;as&nbsp;word,&nbsp;sum(frequency)&nbsp;as&nbsp;frequency&nbsp;from&nbsp;WordCount&nbsp;GROUP&nbsp;BY&nbsp;word"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//将表转换DataSet</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataSet&lt;WC&gt;&nbsp;ds3&nbsp;&nbsp;=&nbsp;fbTableEnv.toDataSet(table02,&nbsp;WC<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ds3.printToErr();
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">WC</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;String&nbsp;word;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;frequency;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">WC</span><span class="hljs-params">()</span>&nbsp;</span>{}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">WC</span><span class="hljs-params">(String&nbsp;word,&nbsp;<span class="hljs-keyword">long</span>&nbsp;frequency)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.word&nbsp;=&nbsp;word;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.frequency&nbsp;=&nbsp;frequency;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">toString</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;&nbsp;word&nbsp;+&nbsp;<span class="hljs-string">",&nbsp;"</span>&nbsp;+&nbsp;frequency;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>我们直接运行该程序，在控制台可以看到输出结果：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/10/9F/Ciqah16YJr6AdfRiAAt-hgEdyno504.png" alt=""></p>
<h3>总结</h3>
<p>本课时介绍了 Flink 的工程创建，如何搭建调试环境的脚手架，同时以 WordCount 单词计数这一最简单最经典的场景用 Flink 进行了实现。第一次体验了 Flink SQL 的强大之处，让你有一个直观的认识，为后续内容打好基础。</p>
<p><a href="https://github.com/wangzhiwubigdata/quickstart">点击这里下载本课程源码</a>。</p>

---

### 精选评论

##### **武：
> 课程案列会上传到Github吗

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 会的哦，Flink 源码地址：https://github.com/wangzhiwubigdata/quickstart

##### **亮：
> 呃，还是scala代码量少啊

##### **1633：
> 感谢，之前看flink的官方文档，虽然有示例。但是由于并没有讲清楚需要依赖的jar包，所以在环境准备上遇到了不少问题。上了这节课，感觉一下子清晰多了！😀

##### *星：
> 请问python可开发大数据嘛？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; python不适合，python是脚本语言。在大数据场景下，多用于写一些简单的脚本。

##### **儒：
> 使用flink更推荐Java而不是Scala吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Flink绝大多数代码是Java开发，Scala是个并不成熟的语言，虽然基于JVM但是性能不稳定，只适合高端玩家。

##### **滨：
> sql的jar包冲突是怎么回事&nbsp;

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用IDEA工具检测一下冲突的原因，可以git pull 一下最新的代码

##### **一：
> 老师，课程不是用scala讲的么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不建议用scala写flink job

##### **欢：
> 紧跟步伐学习，周末在家做一做，看一看源

##### **彬：
> BatchTableEnvironment tEnv = BatchTableEnvironment.create(env);老师这个东西，BatchTableEnvironment我看api源码里是个接口啊，为啥有create方法呢，我在ide里没有create方法，请指教啊。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 注意两点， 第一：Flink版本看看是不是和我的一致，最新的版本中Flink的API发生过变动。第二点，虽然是个接口，但是看方法的签名，static default 接口是可以有默认实现方法的，是Java的语法糖。

##### *靖：
> 运行 可以出结果，但是总是会有异常产生

##### **街大亨：
> curl https://flink.apache.org/q/quickstart.sh | bash -s 1.12.0 ,1.10.0 没用了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 亲测有效，你可以直接访问：https://flink.apache.org/q/quickstart.sh ，找到代码：-DarchetypeVersion=${1:-1.12.0}，这里说的是你指定版本的话就用你指定的版本，不指定的话就用1.12.0版本

##### *艺：
> public static final class LineSplitter implements FlatMapFunction  老师  这个接口总是无法使用，scala 版本2.11 flink版本1.11  这是为什么呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的，你直接把我工程中的 Flink 版本换掉，然后重新导 maven 依赖，这个 FlatMapFunction 是个最基本的函数。

##### **武：
> 本地运行batchJob时 报错：Exception in thread "main" java.lang.RuntimeException: java.util.concurrent.ExecutionException: akka.pattern.AskTimeoutException: Ask timed out on [Actor[akka://flink/user/dispatcher#1330413722]] after [10000 ms]. Message of type [org.apache.flink.runtime.rpc.messages.LocalFencedMessage]. A typical reason for `AskTimeoutException` is that the recipient actor didn't send a reply.请问下是什么原因引起的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://issues.apache.org/jira/browse/FLINK-8485 这个是Flink本身的一个bug，我们自己可以把JDK的版本升级到1.8.1 另外有一个配置可以修改下：akka.ask.timeout: 60 s

##### **用户1554：
> 我拉下来的源码 右键没有办法执行main函数，可能是什么原因呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用maven编译一下，把依赖全部下载完毕。

##### *南：
> 按照课程中的，pom文件中添加依赖后，本地运行，报错：Exception in thread "main" org.apache.flink.table.api.TableException: Create BatchTableEnvironment failed.这个是依赖的版本问题？？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，要注意。Flink在1.10版本做过一些较大更新。建议从1.11开始。

##### **勇：
> fbTableEnv.createTemporaryView("WordCount", table);createTemporaryView 1.9没有这个方法吗?">registerTable

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 把版本更新到1.11吧。1.11又一次较大规模的更新。

##### **吃雪糕：
> flink sql的写法，表的记录数有上限吗？比如上亿的数据数据行性能如何？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; State会变得非常大，性能并不好。我们在SQL或者多个流Join的场景下都会设置State的过期时间。

##### **田：
> 本地模式可以直接运行么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以

##### **强：
> 还不错

##### **博：
> 可以用windows系统吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 最好不要

##### *熙：
> idea本地运行，其实运行时程序本身也会启动了jobmanager，taskmanager的么？？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没错

##### **磊：
> 学习这大数据工具，<div>需要会java web 开发吗</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 最好会

##### **6091：
> 开始学习

##### **6631：
> 老师 帮忙看看一个问题&nbsp;https://github.com/JSQF/flink10_learn.git&nbsp;<div>com.yyb.flink10.stream.WordCount<br></div><div>报错&nbsp;</div><div><div>Error:(64, 10) value build is not a member of ?0</div><div>possible cause: maybe a semicolon is missing before `value build'?</div><div>&nbsp; &nbsp; &nbsp; &nbsp; .build()</div></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; value build之前缺少一个括号

##### **成：
> 有问题啊，最后的flink sql ，pom文件想复制，不能复制，而且需要的jar包无法下载，最后，使用的是Java代码，可以用Scala实现吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以复制的，可以用scala但是不建议

##### lumen：
> 赞一个，作者大大辛苦啦

##### **8527：
> 请教下，sql的window如何使用？用类解析字段的话，未来扩展字段如何解决呢？如果有嵌套的话是不是就更复杂了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; sql高度抽象，如果掌握不好，出现问题很难排查

##### *强：
> 写的很用心，排版很优美，期待更新中😀

##### **6631：
> StreamTable 没有示例啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 二者API类似，把源换成流即可。

