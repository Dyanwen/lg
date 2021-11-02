<p>你好，我是范东来，本课时我们要讲解的内容是：用户自定义函数。在上个课时，你了解了 DataFrame、Dataset 和 Spark SQL，如果说 Spark 隐藏了分布式计算的复杂性，那么可以认为 DataFrame、Dataset 和 Spark SQL 比它要更近一步，用统一而简洁的接口隐藏了数据分析的复杂性。</p>
<p>在本课时，我们会主要介绍在上个课时中没有详细讲解的函数与自定义函数。在实际使用中，函数和自定义函数的使用频率非常高，可以说，对于复杂的需求，如果用好了函数，那么事情会简单许多，反之，则会事倍功半。</p>
<p>本课时的主要内容有：</p>
<ul>
<li>窗口函数</li>
<li>函数</li>
<li>用户自定义函数</li>
</ul>
<h3>窗口函数</h3>
<p>首先，我们来看下窗口函数，窗口函数可以使用户针对某个范围的数据进行聚合操作，如：</p>
<ul>
<li>累积和</li>
<li>差值</li>
<li>加权移动平均</li>
</ul>
<p>可以想象一个窗口在全量数据集上进行滑动，用户可以自定义在窗口中的操作，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/13/Ciqc1F7M72uAIFTtAABmsVbo0mQ737.png" alt="1.png"></p>
<p>使用窗口函数，首先需要定义窗口，DataFrame 提供了 API 定义窗口，以及窗口中的计算逻辑，还是以学生成绩为例，现在需要得出每个学生单科最佳成绩以及成绩所在的年份，这个需求就要用到窗口中的 row_number 函数，row_number 函数可以根据窗口中的数据生成行号，首先来定义窗口：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.expressions.<span class="hljs-type">Window</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
&nbsp;
<span class="hljs-keyword">val</span> window = <span class="hljs-type">Window</span>
.partitionBy(<span class="hljs-string">"name"</span>,<span class="hljs-string">"subject"</span>)
.orderBy(desc(<span class="hljs-string">"grade"</span>))
</code></pre>
<p>上面的代码定义了窗口的范围：按照每个人的姓名与科目的组合进行开窗，并控制了数据在窗口中的顺序：按照 grade 降序进行排序，row_number 函数就可以作用在这个窗口上，对每个人每个科目成绩赋予行号，代码如下：</p>
<pre><code data-language="scala" class="lang-scala">dfSG.select(
col(<span class="hljs-string">"name"</span>),
col(<span class="hljs-string">"subject"</span>),
col(<span class="hljs-string">"year"</span>),
col(<span class="hljs-string">"grade"</span>),
row_number().over(window)
.show()
</code></pre>
<p>结果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/13/Ciqc1F7M75KAeDK1AADO_U0qaqQ227.png" alt="image (3).png"></p>
<p>最后只需要从这张表中过滤出 row_number 等于 1 的数据即可。</p>
<p>此外，DataFrame 还提供了 rowsBetween 和 rangeBetween 来进一步定义窗口范围，其中 rowsBetween 是通过物理行号进行控制，rangeBetween 是通过逻辑条件来对窗口进行控制，来看一个简单的例子，一份两个字段的样例数据：</p>
<pre><code data-language="java" class="lang-java">{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">3</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">4</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">6</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">3</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">4</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">6</span>}
</code></pre>
<p>现在通过窗口函数对相同 key 的 num 字段做累加计算。代码如下：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">val</span> windowSlide = <span class="hljs-type">Window</span>
.partitionBy(<span class="hljs-string">"key"</span>)
.orderBy(<span class="hljs-string">"num"</span>)
.rangeBetween(<span class="hljs-type">Window</span>.currentRow + <span class="hljs-number">2</span>,<span class="hljs-type">Window</span>.currentRow + <span class="hljs-number">20</span>)
&nbsp;
<span class="hljs-keyword">val</span> dfWin = spark.read.json(<span class="hljs-string">"json/window.json"</span>)
&nbsp;
dfWin
.select(col(<span class="hljs-string">"key"</span>),sum(<span class="hljs-string">"num"</span>).over(windowSlide))
.sort(<span class="hljs-string">"key"</span>)
.show()
</code></pre>
<p>在 rangeBetween 中，定义的窗口是当前行的 num 值 +2 到当前行的 num 值 +20 这个区间中的数据，如下所示：</p>
<pre><code data-language="java" class="lang-java">{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">4</span>,<span class="hljs-number">22</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">4</span> + <span class="hljs-number">5</span> + <span class="hljs-number">6</span> = <span class="hljs-number">15</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">4</span>,<span class="hljs-number">22</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">4</span> + <span class="hljs-number">5</span> + <span class="hljs-number">6</span> = <span class="hljs-number">15</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">3</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">5</span>,<span class="hljs-number">23</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">5</span> + <span class="hljs-number">6</span> = <span class="hljs-number">11</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">4</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">6</span>,<span class="hljs-number">24</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">6</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">8</span>,<span class="hljs-number">25</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-keyword">null</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">6</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">8</span>,<span class="hljs-number">26</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-keyword">null</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">1</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">3</span>,<span class="hljs-number">21</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">12</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">4</span>,<span class="hljs-number">22</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">12</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">7</span>,<span class="hljs-number">25</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-number">7</span>

{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">7</span>}&nbsp; &nbsp; &nbsp; &nbsp;窗口为[<span class="hljs-number">9</span>,<span class="hljs-number">27</span>]&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;累加和为<span class="hljs-keyword">null</span>
</code></pre>
<p>rangeBetween 通过字段的值定义了参与计算的逻辑窗口大小，也可以使用 rowsBetween 通过行号来指定参与计算的物理窗口，如下所示：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">val</span> windowSlide = <span class="hljs-type">Window</span>
.partitionBy(<span class="hljs-string">"key"</span>)
.orderBy(<span class="hljs-string">"num"</span>)
.rowsBetween(<span class="hljs-type">Window</span>.currentRow - <span class="hljs-number">1</span>,<span class="hljs-type">Window</span>.currentRow + <span class="hljs-number">1</span>)
&nbsp;
dfWin
.select(col(<span class="hljs-string">"key"</span>),sum(<span class="hljs-string">"num"</span>).over(windowSlide))
.sort(<span class="hljs-string">"key"</span>)
.show()
</code></pre>
<p>代码中定义的窗口由当前行、当前行的前一行、当前行的后一行组成，也就是说窗口大小为 3，计算结果如下：</p>
<pre><code data-language="java" class="lang-java">{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">2</span> + <span class="hljs-number">2</span> = <span class="hljs-number">4</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">2</span> + <span class="hljs-number">2</span> + <span class="hljs-number">3</span> = <span class="hljs-number">7</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">3</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">2</span> + <span class="hljs-number">3</span> + <span class="hljs-number">4</span> = <span class="hljs-number">9</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">4</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">3</span> + <span class="hljs-number">4</span> + <span class="hljs-number">5</span> = <span class="hljs-number">12</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">4</span> + <span class="hljs-number">5</span> + <span class="hljs-number">6</span> = <span class="hljs-number">15</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"1"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">6</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">5</span> + <span class="hljs-number">6</span> = <span class="hljs-number">11</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">1</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">1</span> + <span class="hljs-number">2</span> = <span class="hljs-number">3</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">2</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">1</span> + <span class="hljs-number">2</span> + <span class="hljs-number">5</span> = <span class="hljs-number">8</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">5</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">2</span> + <span class="hljs-number">5</span> + <span class="hljs-number">7</span> = <span class="hljs-number">14</span>
{<span class="hljs-string">"key"</span>:<span class="hljs-string">"2"</span>, <span class="hljs-string">"num"</span>:<span class="hljs-number">7</span>}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 累加和为<span class="hljs-number">5</span> + <span class="hljs-number">7</span> = <span class="hljs-number">12</span>

</code></pre>
<h3>函数</h3>
<p>在需要对数据进行分析的时候，我们经常会使用到函数，Spark SQL 提供了丰富的函数供用户选择，基本涵盖了大部分的日常使用。下面介绍一些常用函数：</p>
<h4>1. 转换函数</h4>
<p>cast(value AS type) → type</p>
<p>它显式转换一个值的类型。可以将字符串类型的值转为数字类型，反过来转换也可以，在转换失败的时候，会返回 null。这个函数非常常用。</p>
<h4>2. 数学函数</h4>
<p>log(double base, <a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;a)</p>
<p>求与以 base 为底的 a 的对数。</p>
<p>factorial(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 的阶乘。</p>
<h4>3. 字符串函数</h4>
<p>split(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;str,String&nbsp;pattern)</p>
<p>根据正则表达式 pattern 匹配结果作为依据来切分字符串 str。</p>
<p>substring(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;str,int&nbsp;pos,int&nbsp;len)</p>
<p>返回字符串 str 中，起始位置为 pos，长度为 len 的字符串。</p>
<p>concat(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>...&nbsp;exprs)</p>
<p>连接多个字符串列，形成一个单独的字符串。</p>
<p>translate(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;src,String&nbsp;matchingString,String&nbsp;replaceString)</p>
<p>在字符串 src 中，用 replaceString 替换 mathchingString。</p>
<p>字符串函数也是非常常用的函数类型。</p>
<h4>4. 二进制函数</h4>
<p>bin(Column e)</p>
<p>返回输入内容 e 的二进制值。</p>
<p>base64(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>计算二进制列e的 base64 编码，并以字符串返回。</p>
<h4>5. 日期时间函数</h4>
<p>current_date()</p>
<p>获取当前日期</p>
<p>current_timestamp()</p>
<p>获取当前时间戳</p>
<p>date_format(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;dateExpr,String&nbsp;format)</p>
<p>将日期/时间戳/字符串形式的时间列，按 format 指定的格式表示，并以字符串返回。</p>
<h4>6. 正则表达式函数</h4>
<p>regexp_extract(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e,String&nbsp;exp,int&nbsp;groupIdx)</p>
<p>首先在 e 中匹配正则表达式 exp，按照 groupIdx 的值返回结果，groupIdx 默认值为 1，返回第 1 个匹配成功的内容，0 表示返回全部匹配成功的内容。</p>
<p>regexp_replace(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e,String&nbsp;pattern,String&nbsp;replacement)</p>
<p>用 replacement 替换在 e 中根据 pattern 匹配成功的字符串。</p>
<h4>7. JSON 函数</h4>
<p>get_json_object(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e,String&nbsp;path)</p>
<p>解析 JSON 字符串 e，返回 path 指定的值。</p>
<h4>8. URL 函数</h4>
<p>parse_url(string urlString, string partToExtract [, stringkeyToExtract])</p>
<p>该函数专门用来解析 URL，提取其中的信息，partToExtract 的选项包含 HOST、PATH、QUERY、REF、PROTOCOL、AUTHORITY、USEINFO，函数会根据选项提取出相应的信息。</p>
<h4>9. 聚合函数</h4>
<p>countDistinct(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;expr,<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>...&nbsp;exprs)</p>
<p>返回一列数据或一组数据中不重复项的个数。expr 为返回 column 的表达式。</p>
<p>avg(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 列的平均数。</p>
<p>count(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 列的行数。</p>
<p>max(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 中的最大值</p>
<p>sum(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 中所有数据之和</p>
<p>skewness(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 列的偏度。</p>
<p>stddev_samp(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>stddev(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 的样本标准差。</p>
<p>var_samp(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>variance(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 的样本方差。</p>
<p>var_pop(<a href="http://spark.apache.org/docs/2.0.2/api/java/org/apache/spark/sql/Column.html">Column</a>&nbsp;e)</p>
<p>返回 e 的总体方差。</p>
<p>这类函数顾名思义，作用于很多行，所以往往与统计分析相关。</p>
<h4>10. 窗口函数</h4>
<p>row_number()</p>
<p>对窗口中的数据依次赋予行号。</p>
<p>rank()</p>
<p>与 row_number 函数类似，也是对窗口中的数据依次赋予行号，但是 rank 函数考虑到了 over 子句中排序字段值相同的情况，如下表所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/20/CgqCHl7M8PuAZAR6AABqraAQ7HE674.png" alt="2.png"><br>
dense_rank()</p>
<p>与 row_number 函数类似，也是对窗口中的数据依次赋予行号，但是 dense_rank 函数考虑到over 子句中排序字段值相同的情况，并保证了序号连续。</p>
<p>ntile(n)</p>
<p>将每一个窗口中的数据放入 n 个桶中，用 1-n 的数字加以区分。</p>
<p><strong>在实际开发过程中，大量的需求都可以直接通过函数以及函数的组合完成，一般来说，函数的丰富程度往往超乎你的想象，所以在面临新需求时，建议首先查阅文档，看看有没有函数可以利用，如果实在不行，我们才会使用用户自定义函数（User Defined Function）。</strong></p>
<p><strong>Spark SQL 的函数文档目前我没有发现特别全面的，所以我通常就会直接阅读源码，源码列出了所有的函数，如下：</strong></p>
<p><a href="https://github.com/apache/spark/blob/6646b3e13e46b220a33b5798ef266d8a14f3c85b/sql/core/src/main/scala/org/apache/spark/sql/functions.scala">https://github.com/apache/spark/blob/6646b3e13e46b220a33b5798ef266d8a14f3c85b/sql/core/src/main/scala/org/apache/spark/sql/functions.scala</a></p>
<h3>用户自定义函数</h3>
<p>DataFrame API 支持用户自定义函数，自定义函数有两种：UDF 和 UDAF，前者是类似于 map操作的行处理，一行输入一行输出，后者是聚合处理，多行输入，一行输出，先来看看 UDF，下面的代码会开发一个根据得分显示分数等级的函数 level：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
<span class="hljs-keyword">import</span> scala.reflect.api.materializeTypeTag
&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">MyUDF</span> </span>{
&nbsp; 
&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
&nbsp;&nbsp;&nbsp; .builder
&nbsp;&nbsp;&nbsp; .master(<span class="hljs-string">"local[2]"</span>)
&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"Test"</span>)
&nbsp;&nbsp;&nbsp; .getOrCreate()
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">import</span> spark.implicits._
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> dfSG = spark.read.json(<span class="hljs-string">"examples/target/scala-2.11/classes/student_grade.json"</span>)
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">level</span></span>(grade: <span class="hljs-type">Int</span>): <span class="hljs-type">String</span> = {
	<span class="hljs-keyword">if</span>(grade &gt;= <span class="hljs-number">85</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"A"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(grade &lt; <span class="hljs-number">85</span> &amp; grade &gt;= <span class="hljs-number">75</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"B"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(grade &lt; <span class="hljs-number">75</span> &amp; grade &gt;= <span class="hljs-number">60</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"C"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(grade &lt; <span class="hljs-number">60</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"D"</span>&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"ERROR"</span>
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
	<span class="hljs-keyword">val</span> myUDF = udf(level _)
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; dfSG.select(col(<span class="hljs-string">"name"</span>),myUDF(col(<span class="hljs-string">"grade"</span>))).show()
&nbsp;&nbsp;&nbsp; 
&nbsp; }
&nbsp; 
}
</code></pre>
<p>接下来看看 UDAF，UDAF 是用户自定义聚合函数，分为两种：un-type UDAF 和 safe-type UDAF，前者是与 DataFrame 配合使用，后者只能用于 Dataset，UDAF 需要实现 UserDefinedAggregateFunction 抽象类，本例实现了一个求某列最大值的 UDAF，代码如下：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.expressions._
<span class="hljs-keyword">import</span> org.apache.spark.sql.types._
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Row</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">MyMaxUDAF</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDefinedAggregateFunction</span> </span>{
&nbsp;
&nbsp; <span class="hljs-comment">//指定输入的类型</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inputSchema</span></span>: <span class="hljs-type">StructType</span> 
&nbsp;&nbsp;&nbsp; = <span class="hljs-type">StructType</span>(<span class="hljs-type">Array</span>(<span class="hljs-type">StructField</span>(<span class="hljs-string">"input"</span>, <span class="hljs-type">IntegerType</span>, <span class="hljs-literal">true</span>)))
&nbsp; 
&nbsp; <span class="hljs-comment">//指定中间输出的类型，可指定多个</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">bufferSchema</span></span>: <span class="hljs-type">StructType</span> 
&nbsp;&nbsp;&nbsp; = <span class="hljs-type">StructType</span>(<span class="hljs-type">Array</span>(<span class="hljs-type">StructField</span>(<span class="hljs-string">"max"</span>, <span class="hljs-type">IntegerType</span>, <span class="hljs-literal">true</span>)))
&nbsp;
&nbsp; <span class="hljs-comment">//指定最后输出的类型</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dataType</span></span>: <span class="hljs-type">DataType</span> = <span class="hljs-type">IntegerType</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">deterministic</span></span>: <span class="hljs-type">Boolean</span> = <span class="hljs-literal">true</span>
&nbsp; 
&nbsp; <span class="hljs-comment">//初始化中间结果</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">initialize</span></span>(buffer: <span class="hljs-type">MutableAggregationBuffer</span>): <span class="hljs-type">Unit</span> 
&nbsp;&nbsp;&nbsp; = {buffer(<span class="hljs-number">0</span>) = <span class="hljs-number">0</span>}
&nbsp; 
&nbsp; <span class="hljs-comment">//实现作用在每个分区的结果</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">update</span></span>(buffer: <span class="hljs-type">MutableAggregationBuffer</span>, input: <span class="hljs-type">Row</span>): <span class="hljs-type">Unit</span> = {
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> temp = input.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>)
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> current = buffer.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>)
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(temp &gt; current)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer(<span class="hljs-number">0</span>) = temp
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-comment">//合并多个分区的结果</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">merge</span></span>(buffer1: <span class="hljs-type">MutableAggregationBuffer</span>, buffer2: <span class="hljs-type">Row</span>): <span class="hljs-type">Unit</span> = {
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(buffer1.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>) &lt; buffer2.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer1(<span class="hljs-number">0</span>) = buffer2.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>)&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp; }
&nbsp; 
&nbsp; <span class="hljs-comment">//返回最后的结果</span>
&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">evaluate</span></span>(buffer: <span class="hljs-type">Row</span>): <span class="hljs-type">Any</span> = buffer.getAs[<span class="hljs-type">Int</span>](<span class="hljs-number">0</span>)
}
&nbsp;
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">MyMaxUDAFDriver</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">App</span></span>{
&nbsp; 
&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
&nbsp;&nbsp;&nbsp; .builder
&nbsp;&nbsp;&nbsp; .master(<span class="hljs-string">"local[2]"</span>)
&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"Test"</span>)
&nbsp;&nbsp;&nbsp; .getOrCreate()
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">import</span> spark.implicits._
&nbsp; 
&nbsp; <span class="hljs-keyword">val</span> dfSG = spark.read.json(<span class="hljs-string">"examples/target/scala-2.11/classes/student_grade.json"</span>)
&nbsp;&nbsp;&nbsp; 
&nbsp; dfSG.select(<span class="hljs-type">MyMaxUDAF</span>(col(<span class="hljs-string">"grade"</span>))).show()
}
</code></pre>
<p>可以从代码看到 UDAF 的逻辑，还是类似于 MapReduce 的思想，先通过 update 函数处理每个分区，最后再通过 merge 函数汇总结果。</p>
<p>Dataset 的 UDAF 对应的是 safe-type UDAF，这种类型的 UDAF 只有 Dataset 能够使用，因为 Dataset 是类型安全的。使用方式和 un-type UDAF 类似，也是先要结合自己聚合的逻辑实现 Aggregator 抽象类，最后再通过 Dataset API 调用，此处实现一个求学生成绩平均值的 UDAF，代码如下：</p>
<pre><code data-language="scala" class="lang-scala"><span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Encoders</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Encoder</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.expressions._
<span class="hljs-keyword">import</span> org.apache.spark.sql.types._
<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
<span class="hljs-keyword">import</span> scala.reflect.api.materializeTypeTag
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Dataset</span>
&nbsp;
<span class="hljs-keyword">case</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StudentGrade</span>(<span class="hljs-params">name: <span class="hljs-type">String</span>, subject: <span class="hljs-type">String</span>, grade: <span class="hljs-type">Long</span></span>)</span>
&nbsp;
<span class="hljs-keyword">case</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Average</span>(<span class="hljs-params">var sum: <span class="hljs-type">Long</span>, var count: <span class="hljs-type">Long</span></span>)</span>
&nbsp;
<span class="hljs-comment">//这里定义的三个类型分别是输入类型、中间结果类型、输出类型</span>
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">MyAvgUDAF</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Aggregator</span>[<span class="hljs-type">StudentGrade</span>,<span class="hljs-type">Average</span>,<span class="hljs-type">Double</span>]</span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//初始中间状态</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">zero</span></span>: <span class="hljs-type">Average</span> = <span class="hljs-type">Average</span>(<span class="hljs-number">0</span>L,<span class="hljs-number">0</span>L)
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//更新中间状态</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">reduce</span></span>(buffer: <span class="hljs-type">Average</span>, sg: <span class="hljs-type">StudentGrade</span>): <span class="hljs-type">Average</span> = {
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;buffer.sum += sg.grade
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer.count += <span class="hljs-number">1</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//合并状态</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">merge</span></span>(b1: <span class="hljs-type">Average</span>, b2: <span class="hljs-type">Average</span>): <span class="hljs-type">Average</span> = {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; b1.sum += b2.sum
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; b1.count += b2.count
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; b1
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//得到最后结果</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">finish</span></span>(reduction: <span class="hljs-type">Average</span>): <span class="hljs-type">Double</span> = reduction.sum / reduction.count
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//为中间结果指定编译器</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">bufferEncoder</span></span>: <span class="hljs-type">Encoder</span>[<span class="hljs-type">Average</span>] = <span class="hljs-type">Encoders</span>.product
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//为输出结果指定编译器</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">outputEncoder</span></span>: <span class="hljs-type">Encoder</span>[<span class="hljs-type">Double</span>] = <span class="hljs-type">Encoders</span>.scalaDouble
&nbsp;
}
	通过<span class="hljs-type">Dataset</span> <span class="hljs-type">API</span>调用：
<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">MyAvgUDAFDriver</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">App</span></span>{
&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
&nbsp;&nbsp;&nbsp; .builder
&nbsp;&nbsp;&nbsp; .master(<span class="hljs-string">"local[2]"</span>)
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//.config("spark.reducer.maxSizeInFlight", "128M")</span>
&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"Test"</span>)
&nbsp;&nbsp;&nbsp; .getOrCreate()
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">import</span> spark.implicits._
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//读取数据</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> dfSG = spark.read.json(<span class="hljs-string">"examples/target/scala-2.11/classes/student_grade.json"</span>)
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//生成Dataset</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> dsSG: <span class="hljs-type">Dataset</span>[<span class="hljs-type">StudentGrade</span>] = dfSG.map(a =&gt; <span class="hljs-type">StudentGrade</span>(a.getAs[<span class="hljs-type">String</span>](<span class="hljs-number">0</span>),a.getAs[<span class="hljs-type">String</span>](<span class="hljs-number">1</span>),a.getAs[<span class="hljs-type">Long</span>](<span class="hljs-number">2</span>)))
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//注册UDAF</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> <span class="hljs-type">MyAvg</span> = <span class="hljs-type">MyAvgUDAF</span>.toColumn.name(<span class="hljs-string">"MyAvg"</span>)
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//查询</span>
&nbsp;&nbsp;&nbsp; dsSG.select(<span class="hljs-type">MyAvg</span>).show()
&nbsp; 

}
</code></pre>
<p>自定义函数注册以后，同样可以在 Spark SQL 中使用。</p>
<h3>小结</h3>
<p>最后来做一个小结，这不光针对本课时，而是一个阶段性小结，现在我们学习了 RDD API、DataFrame API 和 Dataset API，对于数据处理来说，它们都能胜任，那么在实际使用中应该如何选择呢。</p>
<p>一般来说，在任何情况下，都不推荐使用 RDD 算子，原因如下：</p>
<ul>
<li>在某种抽象层面上来说，使用 RDD 算子编程相当于直接使用汇编语言或者机器代码进行编程；</li>
<li>RDD算子与 SQL、DataFrame API 和 Dataset API 相比，更偏向于如何做，而非做什么，这样优化的空间很少；</li>
<li>RDD 语言不如 SQL 语法友好。</li>
</ul>
<p>此外，在其他情况，应优先考虑 Dataset，因为静态类型的特点会使计算更加迅速，但用户必须使用静态语言才行，如 Java 与 Scala，像 Python 这种动态语言是没有 Dataset API 的。</p>
<p>下图是用户用不同语言基于 RDD API 和 DataFrame API 开发的应用性能对比，可以看到 Python + RDD API 的组合是远远落后其他组合的，此外，RDD API 开发应用的性能整体要明显落后于 DataFrame API 开发的应用性能。从开发速度和性能上来说，DataFrame + SQL 无疑是最好选择。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/21/CgqCHl7M8VSAOjrdAACafd9hNk4280.png" alt="3.png"></p>
<p>最后，还是留一个思考题，最开始我们举了一个窗口函数的例子，后面在介绍 Spark SQL 中也学习了窗口函数，你能用 SQL 的窗口函数实现同样的逻辑吗？</p>

---

### 精选评论

##### *艺：
> select name,subject,year,grade,row_number()over(partition By name,subject order By grade desc) rnfrom student_table;

##### **9134：
> 范老师，Dataset是类型安全，啥意思？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 类型安全的意思是 如果使用的对象符合它们规定的类型，那么它们是类型安全的。

##### *壮：
> 刚下班老师就更新了，回家路上听完美！😀

