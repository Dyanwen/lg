<p data-nodeid="88703" class="">今天为你介绍数据分析师最常用的数据处理工具 Hive 的一些使用技巧。这些技巧我们在工作中使用得比较频繁，如果运用得当，将为我们省去不少时间精力。</p>
<p data-nodeid="88704">那么首先，我们先来了解下 Hive。Hive 是 Facebook 开源的一款基于 Hadoop 的数据仓库工具，它能完美支持 SQL 查询功能，将 SQL 查询转变为 MapReduce 任务执行。这使得大数据统计得以实现。Hive 是最早的也是目前应用最广泛的大数据处理解决方案。</p>
<p data-nodeid="88705">Hive 的重要性不必多言。 数据分析师在工作中使用 Hive SQL 来处理大数据本是家常便饭。</p>
<p data-nodeid="88706">本课时将重点介绍 Hive 在工作中常用的数据处理技巧。</p>
<h3 data-nodeid="88707">基本操作</h3>
<p data-nodeid="88708">这里简单介绍 Hive 最基础的操作，<strong data-nodeid="88876">创建表</strong>和<strong data-nodeid="88877">加载数据</strong>。重点说明 Hive 创建内部表和外部表的应用场景。</p>
<h4 data-nodeid="88709">创建表</h4>
<p data-nodeid="88710">创建表的语法如下：</p>
<pre class="lang-java" data-nodeid="88711"><code data-language="java">CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
  [(col_name data_type [COMMENT col_comment], ...)]
  [COMMENT table_comment]
  [<span class="hljs-function">PARTITIONED <span class="hljs-title">BY</span> <span class="hljs-params">(col_name data_type [COMMENT col_comment], ...)</span>]
  [CLUSTERED <span class="hljs-title">BY</span> <span class="hljs-params">(col_name, col_name, ...)</span> [SORTED <span class="hljs-title">BY</span> <span class="hljs-params">(col_name [ASC|DESC], ...)</span>] INTO num_buckets BUCKETS]
  [
   [ROW FORMAT row_format] [STORED AS file_format]
   | STORED BY 'storage.handler.class.name' [ WITH <span class="hljs-title">SERDEPROPERTIES</span> <span class="hljs-params">(...)</span> ]
  ]
  [LOCATION hdfs_path]
</span></code></pre>
<p data-nodeid="88712">上面的代码看起来不是很容易理解，这里通过两个例子重点讲下如何按照上面语法进行 Hive 表创建，并且重点说下创建<strong data-nodeid="88889">内部表</strong>和<strong data-nodeid="88890">外部表</strong>的区别。</p>
<ul data-nodeid="88713">
<li data-nodeid="88714">
<p data-nodeid="88715"><strong data-nodeid="88894">内部表</strong></p>
</li>
</ul>
<p data-nodeid="88716">第一步，创建一个内部表，按天分区，字段直接以'\t'分割：</p>
<pre class="lang-java" data-nodeid="88717"><code data-language="java">create table table_test_1  -- 这里没有标注external创建的表, 就是内部表
	uid string comment <span class="hljs-string">'用户id'</span>,
	age string comment <span class="hljs-string">'用户年龄'</span>,
	gender string comment <span class="hljs-string">'用户性别'</span>
<span class="hljs-function">PARTITIONED <span class="hljs-title">BY</span><span class="hljs-params">(dt STRING)</span>
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY '\t'
STORED AS Textfile
</span></code></pre>
<p data-nodeid="88718">第二步，向刚创建的表加载数据，有本机数据加载和 HDFS 路径数据加载两种方式。本机本地数据导入刚创建的表方式如下：</p>
<pre class="lang-java" data-nodeid="88719"><code data-language="java">LOAD DATA LOCAL INPATH <span class="hljs-string">'本机文件路径'</span> <span class="hljs-function">OVERWRITE INTO TABLE table_test_1 <span class="hljs-title">PARTITION</span><span class="hljs-params">(dt=<span class="hljs-string">'20200822'</span>)</span></span>;
</code></pre>
<p data-nodeid="89932" class="">HDFS 路径加载数据方式如下。</p>


<pre class="lang-java" data-nodeid="88721"><code data-language="java">LOAD DATA INPATH <span class="hljs-string">'hdfs文件路径'</span> <span class="hljs-function">OVERWRITE INTO TABLE table_test_1 <span class="hljs-title">PARTITION</span><span class="hljs-params">(dt=<span class="hljs-string">'20200822'</span>)</span></span>;
</code></pre>
<ul data-nodeid="88722">
<li data-nodeid="88723">
<p data-nodeid="88724"><strong data-nodeid="88906">外部表</strong></p>
</li>
</ul>
<p data-nodeid="88725"><strong data-nodeid="88911">外部表</strong>创建，即为某路径下的文件指定为一个 Hive 表结构，这种外部表的创建，不涉及数据的移动。</p>
<p data-nodeid="88726">外部表创建有两种方式，第一种，就是在建表的时候就指定外部表的数据依赖路径。下面的代码即做了这方面的展示：</p>
<pre class="lang-java" data-nodeid="88727"><code data-language="java">-- 第一步, 创建外部表：
create external table table_test_2  -- 这里标注external创建的表, 就是外部表
	uid string comment <span class="hljs-string">'用户id'</span>,
	age string comment <span class="hljs-string">'用户年龄'</span>,
	gender string comment <span class="hljs-string">'用户性别'</span>
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY <span class="hljs-string">'\t'</span>
LOCATION <span class="hljs-string">'HDFS路径'</span>   -- 这里指定集群中某个路径的文件为该外部表数据源
</code></pre>
<p data-nodeid="88728">第二种，是先创建一个外部表结构，然后再指定路径。</p>
<pre class="lang-java" data-nodeid="88729"><code data-language="java">-- 第一步, 创建外部表:
create external table table_test_2  -- 这里标注external创建的表, 就是外部表
	uid string comment <span class="hljs-string">'用户id'</span>,
	age string comment <span class="hljs-string">'用户年龄'</span>,
	gender string comment <span class="hljs-string">'用户性别'</span>
ROW FORMAT DELIMITED
  FIELDS TERMINATED BY <span class="hljs-string">'\t'</span>
</code></pre>
<p data-nodeid="88730">上面我们通过代码展示了 Hive 表的创建，下面来讲解下 Hive 复合类型数据。</p>
<h4 data-nodeid="88731">复合类型数据</h4>
<p data-nodeid="88732">对于 Hive 的基础数据结果，比如 string、bigint 等比较简单，这里直接跳过。下面经哥讲下 Hive 的复合类型数据用法。即 map、array、json，这三个复合数据类型要相对有一点点难度，可在工作中我们会经常使用。</p>
<p data-nodeid="88733"><strong data-nodeid="88920">map</strong></p>
<p data-nodeid="88734">首先是 map。map 的数据结构是 key-value 格式。数据语法为 map(k1,v1,k2,v2,…)。使用给定的 key-value 对，构造一个 map 数据结构。</p>
<p data-nodeid="88735">下面这个例子表现的就是 map 构建。</p>
<pre class="lang-sql" data-nodeid="88736"><code data-language="sql">hive&gt; select map('k1','v1','k2','v2'), map('k1,'v1','k2','v2')['k1'] -- 获取k1对应值, map'k1',v1','k2','v2')['k2'] -- 获取k2对应值from tble_tet;OK{"k2""v","k1:"1"} v1 v2
</code></pre>
<p data-nodeid="88737">工作中，map 样式数据被建表的同学声明为 string。这时候如果要使用 map 类型的特性，需要先将 string 转化为 map 类型。那么首先，我们要将下面字符串 'zhang:101&amp;wang:102&amp;li:103' 转化为 map，就需要使用 str_to_map 这个函数：</p>
<pre class="lang-sql" data-nodeid="88738"><code data-language="sql">- str_to_map(your_string, delimiter1, delimiter2)
<span class="hljs-comment">-- your_string: 是准备要转化的字符串</span>
<span class="hljs-comment">-- delimiter1: 将字符串切分成KV格式</span>
<span class="hljs-comment">-- 每个KV的分隔符</span>
</code></pre>
<p data-nodeid="88739">举个例子，我们可以看一下这个代码：</p>
<pre class="lang-java" data-nodeid="88740"><code data-language="java"><span class="hljs-function">select <span class="hljs-title">str_to_map</span><span class="hljs-params">(<span class="hljs-string">'zhang:101&amp;wang:102&amp;li:103'</span>, <span class="hljs-string">'&amp;'</span>, <span class="hljs-string">':'</span>)</span></span>;  -- 这里注意第<span class="hljs-number">2</span>、<span class="hljs-number">3</span>个参数
&gt;&gt;&gt; 输出如下: 
{<span class="hljs-string">"zhang"</span>:<span class="hljs-string">"101"</span>,<span class="hljs-string">"wang"</span>:<span class="hljs-string">"102"</span>,<span class="hljs-string">"li"</span>:<span class="hljs-string">"103"</span>}
</code></pre>
<p data-nodeid="88741">以上为 map 构建过程。<br>
<strong data-nodeid="88942">array</strong></p>
<p data-nodeid="88742">了解过 map 后，我们再来了解一下 array。array 的语法为 array(val1,val2,val3,…)，操作类型为 array。我们可以使用给定的表达式，构造一个 array 数据结构。举例：</p>
<pre class="lang-java" data-nodeid="88743"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">array</span><span class="hljs-params">(<span class="hljs-number">100</span>,<span class="hljs-number">200</span>,<span class="hljs-number">300</span>)</span>
, <span class="hljs-title">array</span><span class="hljs-params">(<span class="hljs-number">100</span>,<span class="hljs-number">200</span>,<span class="hljs-number">300</span>)</span>[0]
, <span class="hljs-title">array</span><span class="hljs-params">(<span class="hljs-number">100</span>,<span class="hljs-number">200</span>,<span class="hljs-number">300</span>)</span>[1]
from table_test</span>;
OK
[<span class="hljs-number">100</span>,<span class="hljs-number">200</span>,<span class="hljs-number">300</span>] <span class="hljs-number">100</span> <span class="hljs-number">200</span>
</code></pre>
<p data-nodeid="88744">刚才我们说到实际工作的表，几乎都是默认使用 string 存储，很少在创建表的时候使用 array，往往都是基于 string 生成 array 进行使用的。那么，什么时候会用到 array 类型呢？下面我们举两个使用 array 类型的实例：</p>
<ul data-nodeid="88745">
<li data-nodeid="88746">
<p data-nodeid="88747"><strong data-nodeid="88948">split 切分字符串返回 array 类型。</strong></p>
</li>
</ul>
<p data-nodeid="88748">第一个例子为 split 切分字符串返回 array 类型。如下代码所示。</p>
<pre class="lang-java" data-nodeid="88749"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">split</span><span class="hljs-params">(<span class="hljs-string">'zhang:wang:li'</span>, <span class="hljs-string">':'</span>)</span>    -- 即：将字符串切分为array
, <span class="hljs-title">split</span><span class="hljs-params">(<span class="hljs-string">'zhang:wang:li'</span>, <span class="hljs-string">':'</span>)</span>[0]            -- 即：将字符串切分为array, 并获取array第一个元素
, <span class="hljs-title">split</span><span class="hljs-params">(<span class="hljs-string">'zhang:wang:li'</span>, <span class="hljs-string">':'</span>)</span>[1]            -- 即：将字符串切分为array, 并获取array第二个元素
from table_test</span>;
OK
[<span class="hljs-string">"zhang"</span>,<span class="hljs-string">"wang"</span>,<span class="hljs-string">"li"</span>] zhang wang
</code></pre>
<ul data-nodeid="88750">
<li data-nodeid="88751">
<p data-nodeid="88752"><strong data-nodeid="88955">collect_list 聚合某字段返回 array 类型。</strong></p>
</li>
</ul>
<p data-nodeid="88753">我们假设一张记录用户登录使用手机型号的表，如下所示：</p>
<pre class="lang-java" data-nodeid="88754"><code data-language="java">hive&gt; select userId, phone from table_test;
OK
<span class="hljs-number">10001</span> MI_1
<span class="hljs-number">10001</span> MI_2
<span class="hljs-number">10001</span> MI_3
<span class="hljs-number">10001</span> MI_4
<span class="hljs-number">10002</span> MIX_2
<span class="hljs-number">10002</span> IPHONE11
<span class="hljs-number">10002</span> HUAWEI
</code></pre>
<p data-nodeid="88755">随后，我们对不同用户的不同终端类型以及终端数进行统计。</p>
<pre class="lang-js" data-nodeid="88756"><code data-language="js">select userId
, sort_array(collect_set(phone)) <span class="hljs-keyword">as</span> phone_array   -- 这里将同一个用户使用的终端类型情况聚合到一个数组内并去除重复的手机型号, 最后再排序
, size(collect_set(phone)) <span class="hljs-keyword">as</span> phone_size   --  将用一个用户使用手机类型去重后求数组大小, 即为用户使用不同终端数量
<span class="hljs-keyword">from</span> table_test
group by userId
</code></pre>
<p data-nodeid="88757">需要注意的是<strong data-nodeid="88971">collect_set、collect_list</strong>。这里生成 array、sort_array 和 size 函数，是为了对生成的 array 进行处理。<br>
在工作中，这样的应用场景很多。我们可以把手机终端替换成 IP 地址、省份、兴趣标签等，得出不同的业务场景。比如，分析一个用户在 1 天或者 1 分钟内切换 IP 地址的数量，用来作为一个公司反作弊的数据指标；分析下单用户身上的兴趣标签，用来为业务方提供优质的决策。</p>
<p data-nodeid="88758"><strong data-nodeid="88975">json</strong></p>
<p data-nodeid="88759">再来看下 json，json 同样是我们使用较多的数据。因为 json 具有良好的可读性和便于快速编写的特点，可以在不同平台之间进行数据交换。所以 json 数据类型在开发接口的时候应用最广。</p>
<p data-nodeid="88760">语法使用 get_json_object(string json_string, string path)。返回值使用 string。json 字符串可被 Hive 存储，解析可以直接使用 get_json_object。需要说明的是，解析 json 的字符串 json_string，则返回 path 指定的内容。如果输入的 json 字符串无效，则返回 NULL。</p>
<p data-nodeid="88761">下面两个例子示范 json 字符串是如何解析使用的。</p>
<p data-nodeid="88762">第一个例子相对简单，只解析 json 第一层：</p>
<pre class="lang-java" data-nodeid="88763"><code data-language="java">hive&gt; select get_json_object('{"data":
{"teachers":\[{"gender":1,"age":"30"},{"gender":0,"age":"35"}\],
"students":\[{"gender":1,"age":"10"},{"gender":0,"age":"11"}\]
},"city_name":"beijing"', '$.city_name');
&gt;&gt;&gt; 输出：
beijing
</code></pre>
<p data-nodeid="88764">这个例子目的在于让你了解 get_json_object 的基本用法。</p>
<p data-nodeid="88765">第二个例子相对复杂。这是一个嵌套 json array 的解析过程。需要你花时间学习、理解。以后再遇到 json 嵌套问题，都可以用这种方法解决。</p>
<p data-nodeid="88766">这个例子中的数据如下：</p>
<pre class="lang-java" data-nodeid="88767"><code data-language="java">hive&gt; select json_sample from table_test;
OK
-- 返回json字符串，记录信息为用户在某个时间访问了不同城市旅游产品信息，为方便阅读，已经过json格式化处理：
{
<span class="hljs-string">"time"</span>: <span class="hljs-number">1597920700558</span>,
<span class="hljs-string">"from"</span>: <span class="hljs-string">"104"</span>,
<span class="hljs-string">"uid"</span>: <span class="hljs-string">"124789"</span>,
<span class="hljs-string">"ip"</span>: <span class="hljs-string">"198.168.0.52"</span>,
<span class="hljs-string">"data"</span>: [{
<span class="hljs-string">"post_id"</span>: <span class="hljs-string">"{\"city_id\":\"10001\",\"time\":1597920700558\"}"</span>,
<span class="hljs-string">"type"</span>: <span class="hljs-number">3</span>
}, {
<span class="hljs-string">"post_id"</span>: <span class="hljs-string">"{\"city_id\":\"10002\",\"time\":1597920700558\"}"</span>,
<span class="hljs-string">"type"</span>: <span class="hljs-number">8</span>
}]
}
</code></pre>
<p data-nodeid="88768">这里的目的是将用户访问过的城市 id 提取出来，并进行<strong data-nodeid="89004">行转列</strong>操作。</p>
<pre class="lang-js" data-nodeid="88769"><code data-language="js">hive&gt; select uid, split(get_json_object(get_json_object(json_sample, <span class="hljs-string">'$.post_id'</span>),<span class="hljs-string">'$.city_id'</span>),<span class="hljs-string">'_'</span>)[<span class="hljs-number">0</span>] <span class="hljs-keyword">as</span> city_id
<span class="hljs-keyword">from</span> table_test
lateral view explode(split(
regexp_replace(
regexp_replace(
get_json_object(data, <span class="hljs-string">'$.data'</span>),
<span class="hljs-string">'\\[|\\]'</span>,<span class="hljs-string">''</span>),  --将 json 数组两边的中括号去掉。 这里需要特别注意反斜杠\在hive中的用法。

<span class="hljs-string">'\\}\\,\\{'</span>    --将 json 数组元素之间的逗号换成分号， 因为data内部是多个 json 串，不同 json 串也是用逗号分隔，这个容易和下一层分隔符混淆
,<span class="hljs-string">'\\}\;\\{'</span>),

<span class="hljs-string">'\\;'</span>)) tb <span class="hljs-keyword">as</span> json_sample  -- 将 json 数组切分成一个个 json 结构的字符串，以使用 get_json_object 进行解析
-- 最后输出结果如下：
<span class="hljs-number">124789</span>  <span class="hljs-number">10001</span>
<span class="hljs-number">124789</span>  <span class="hljs-number">10002</span>
</code></pre>
<p data-nodeid="88770">可以看出，上面 SQL 解析出 json 中 uid、city_id， 并将一行转为两行输出。这个案例分享的是对于嵌套 json 的解析过程。如果可以理解并掌握，那么以后 Hive 中涉及 json 数据解析都不会是问题。</p>
<h3 data-nodeid="88771">特殊数据处理场景</h3>
<p data-nodeid="88772">除去上述场景外，还有一些特殊的数据处理场景。下面我们介绍一个较为典型的<strong data-nodeid="89014">时间函数</strong>。</p>
<h4 data-nodeid="88773">时间函数</h4>
<p data-nodeid="88774">时间类型，应用场景非常广泛，处理也很有技巧性。</p>
<p data-nodeid="88775">针对工作中经常使用的日期转化和运算，我分享一个例子，相信对你很有帮助。</p>
<p data-nodeid="88776">首先，我讲下日期格式转化：</p>
<ul data-nodeid="88777">
<li data-nodeid="88778">
<p data-nodeid="88779">时间戳转化日期：from_unixtime(bigint unixtime, string format)。</p>
</li>
</ul>
<p data-nodeid="88780">这里我们需要两个参数。</p>
<p data-nodeid="88781">参数1：unixtime 为时间戳，即从 19700101 开始至今累计的秒数。</p>
<p data-nodeid="88782">参数2：是希望转行日期的格式，比如年月日可表示为"yyyy-MM-dd"。</p>
<pre class="lang-java" data-nodeid="88783"><code data-language="java">-- 注意下面：将字符串可以按照第二个参数定制的样式输出
hive&gt; <span class="hljs-function">select <span class="hljs-title">from_unixtime</span><span class="hljs-params">(<span class="hljs-number">1598360886</span>, <span class="hljs-string">'yyyy-MM-dd HH:mm:ss'</span>)</span>
, <span class="hljs-title">from_unixtime</span><span class="hljs-params">(<span class="hljs-number">1598360886</span>, <span class="hljs-string">'yyyy-MM-dd HH'</span>)</span>
, <span class="hljs-title">from_unixtime</span><span class="hljs-params">(<span class="hljs-number">1598360886</span>, <span class="hljs-string">'yyyy-MM-dd'</span>)</span>
, <span class="hljs-title">from_unixtime</span><span class="hljs-params">(<span class="hljs-number">1598360886</span>, <span class="hljs-string">'yyyyMMdd'</span>)</span>
OK
</span></code></pre>
<ul data-nodeid="88784">
<li data-nodeid="88785">
<p data-nodeid="88786">日期转化时间戳：unix_timestamp(string date, string format)。</p>
</li>
</ul>
<p data-nodeid="88787">这里同样需要两个参数。</p>
<p data-nodeid="88788">参数1：date为日期字符串，如"2020-01-01"。</p>
<p data-nodeid="88789">参数2：是对应参数 1 的格式，格式必须保持一致，即前面是年月日，这里也要对应年月日。</p>
<pre class="lang-java" data-nodeid="88790"><code data-language="java">-- 根据第一个参数格式, 第二个参数格式也需要一致才可以运行
hive&gt; <span class="hljs-function">select <span class="hljs-title">unix_timestamp</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23'</span>, <span class="hljs-string">'yyyy-MM-dd'</span>)</span>
, <span class="hljs-title">unix_timestamp</span><span class="hljs-params">(<span class="hljs-string">'20200823'</span>, <span class="hljs-string">'yyyyMMdd'</span>)</span>
, <span class="hljs-title">unix_timestamp</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23 21'</span>, <span class="hljs-string">'yyyy-MM-dd'</span>)</span>
, <span class="hljs-title">unix_timestamp</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23 21:30:30'</span>, <span class="hljs-string">'yyyy-MM-dd HH:mm:ss'</span>)</span>
, <span class="hljs-title">unix_timestamp</span><span class="hljs-params">(<span class="hljs-string">'20200823 21:30:30'</span>, <span class="hljs-string">'yyyyMMdd HH:mm:ss'</span>)</span>   --  注意与上一行格式上区别, 以及两个参数对应的调整
OK
1598112000  1598112000  1598112000  1598189430
</span></code></pre>
<ul data-nodeid="88791">
<li data-nodeid="88792">
<p data-nodeid="88793">yyyyMMdd 转化为 yyyy-MM-dd。</p>
</li>
</ul>
<p data-nodeid="88794">先说下这种转化的应用场景，yyyyMMdd 格式往往会作为 Hive 日期分区常见样式，但是导出的数据在 Excel 表会被认为是 8 位数字，而不是日期格式。这个时候，我们可以在数据导入 Excel 后再手动转化为 yyyy-MM-dd。还有一种方式是 Hive 输出日期格式就是 yyyy-MM-dd， 这也就是这个日期转化的应用场景。</p>
<p data-nodeid="88795">针对这种日期格式转化，我分享下面两种方法给你。</p>
<p data-nodeid="88796">第一种方法：将 from_unixtime 和 unix_timestamp 结合起来。</p>
<pre class="lang-java" data-nodeid="88797"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">from_unixtime</span><span class="hljs-params">(unix_timestamp(<span class="hljs-string">'20200823'</span>,<span class="hljs-string">'yyyyMMdd'</span>)</span>, 'yyyy-MM-dd')
OK
2020-08-23
</span></code></pre>
<p data-nodeid="88798">第二种方法：字符串函数 substr。</p>
<pre class="lang-java" data-nodeid="88799"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">concat</span><span class="hljs-params">(substr(<span class="hljs-string">'20200823'</span>,<span class="hljs-number">1</span>,<span class="hljs-number">4</span>)</span>,"-",<span class="hljs-title">substr</span><span class="hljs-params">(<span class="hljs-string">'20200823'</span>,<span class="hljs-number">5</span>,<span class="hljs-number">2</span>)</span>,"-",<span class="hljs-title">substr</span><span class="hljs-params">(<span class="hljs-string">'20200823'</span>,<span class="hljs-number">7</span>,<span class="hljs-number">2</span>)</span>) as dt</span>;
OK
<span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">23</span>
</code></pre>
<ul data-nodeid="88800">
<li data-nodeid="88801">
<p data-nodeid="88802">月份转化季度。即给出 1~12 整数，划分到不同季度，提取月份转化为整数。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="88803"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">floor</span><span class="hljs-params">((month(<span class="hljs-string">'2020-08-23'</span>)</span>-1)/3)+1                  -- 季度 date</span>=yyyy-MM-dd
, floor((cast(substr(<span class="hljs-string">'20200823'</span>,<span class="hljs-number">5</span>,<span class="hljs-number">2</span>) as <span class="hljs-keyword">int</span>)-<span class="hljs-number">1</span>)/<span class="hljs-number">3</span>)+<span class="hljs-number">1</span>             -- 季度 date=yyyyMMd
, pmod(datediff(<span class="hljs-string">'2020-08-23'</span>,<span class="hljs-string">'2012-01-01'</span>),<span class="hljs-number">7</span>)         -- 星期几, <span class="hljs-number">0</span>-<span class="hljs-number">6</span>, <span class="hljs-number">0</span>表示周日, 表示周六
, weekofyear(<span class="hljs-string">'2020-08-23'</span>)                            -- 一年中第几周
, year(<span class="hljs-string">'2020-08-23'</span>)   -- 取出年的数字部分
, month(<span class="hljs-string">'2020-08-23'</span>)  -- 取出月的数字部分
, day(<span class="hljs-string">'2020-08-23'</span>)  -- 取出日的数字部分
, hour(<span class="hljs-string">'2020-08-23 21:30:35'</span>)  -- 取出日的数字部分
, minute(<span class="hljs-string">'2020-08-23 21:30:35'</span>)  -- 取出日的数字部分
, second(<span class="hljs-string">'2020-08-23 21:30:35'</span>)  -- 取出日的数字部分
OK
<span class="hljs-number">3</span> <span class="hljs-number">3</span> <span class="hljs-number">0</span> <span class="hljs-number">34</span>  <span class="hljs-number">2020</span>  <span class="hljs-number">8</span> <span class="hljs-number">23</span>  <span class="hljs-number">21</span>  <span class="hljs-number">30</span>  <span class="hljs-number">35</span>
</code></pre>
<p data-nodeid="88804">日期运算很重要，相对容易理解。下面直接列举函数再附上实例，大家会一目了然。</p>
<ul data-nodeid="88805">
<li data-nodeid="88806">
<p data-nodeid="88807">datediff(string enddate, string startdate)函数，用来计算两个日期天数差。</p>
</li>
</ul>
<p data-nodeid="88808">即，返回计算两个参数 enddate-startdate 间隔天数。</p>
<p data-nodeid="88809">例子如下：</p>
<pre class="lang-java" data-nodeid="88810"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">datediff</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23'</span>,<span class="hljs-string">'2020-08-20'</span>)</span>
OK
3
</span></code></pre>
<ul data-nodeid="88811">
<li data-nodeid="88812">
<p data-nodeid="88813">date_add(string startdate, int days)的函数例子如下：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="88814"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">date_add</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23'</span>,<span class="hljs-number">3</span>)</span>   -- 注意这里也可以填写负数， 即返回前几天日期
OK
2020-08-26
</span></code></pre>
<ul data-nodeid="88815">
<li data-nodeid="88816">
<p data-nodeid="88817">date_sub(string startdate, int days) -- 这个用法和 date_add 相反，只是返回当前日期，并减去几天的日期。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="88818"><code data-language="java">hive&gt; <span class="hljs-function">select <span class="hljs-title">date_sub</span><span class="hljs-params">(<span class="hljs-string">'2020-08-23'</span>,<span class="hljs-number">3</span>)</span>  -- 这里也可以为负数，即返回后几天日期
OK
2020-08-20
</span></code></pre>
<p data-nodeid="88819">除了以上处理技巧，我们在第一节数据处理十大技能专门讲了 Hive 两个重要技能：行列转化和 row_number 函数。如果你感兴趣，可以去“入行必备：数据处理的十大技巧”进行学习。</p>
<h3 data-nodeid="88820">查询性能优化</h3>
<p data-nodeid="88821">Hive 查询优化功能表现在很多方面，这里主要讲解日常工作中数据分析师应该注意的 case。</p>
<h4 data-nodeid="88822">合并小文件</h4>
<p data-nodeid="88823">Hive SQL 被解析为 MapReduce 执行，一个文件会对应一个 mapper 来处理。如果数据源是大量的小文件，则会启动大量的 mapper 任务。这就好比一辆卡车一次只运送一个西瓜，浪费了大量资源。这时候，我们需要把西瓜聚合起来，让一辆卡车一次至少搬运几百斤西瓜。</p>
<p data-nodeid="88824">对于 Hive 来说，就是将输入的小文件进行合并，从而减少 mapper 任务数量。</p>
<p data-nodeid="88825">例如，当查询数据时，如果已知对应数据源有很多小文件，可以先进行如下设置：</p>
<pre class="lang-js" data-nodeid="88826"><code data-language="js">set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat; -- <span class="hljs-built_in">Map</span>端输入、合并文件之后按照block的大小分割（默认）
set hive.input.format=org.apache.hadoop.hive.ql.io.HiveInputFormat; -- <span class="hljs-built_in">Map</span>端输入，不合并
</code></pre>
<p data-nodeid="88827">当自己在建立中间表时，输出数据为了避免小文件过多，可以进行如下设置：</p>
<pre class="lang-java" data-nodeid="88828"><code data-language="java">set hive.merge.mapredfiles = <span class="hljs-keyword">true</span>;
set hive.merge.size.per.task=<span class="hljs-number">256000000</span>;  -- <span class="hljs-number">256</span>MB
set hive.merge.smallfiles.avgsize=<span class="hljs-number">256000000</span>;  -- <span class="hljs-number">256</span>MB
</code></pre>
<h4 data-nodeid="88829">数据倾斜</h4>
<p data-nodeid="88830">数据倾斜是什么？我同样举一个卡车运西瓜的例子说明。假设现在有 1 万个西瓜，有 100 辆卡车，前 99 辆车每辆车一次只运送 1 个西瓜，剩余 9901 个西瓜都让最后一辆卡车拉。这就是“西瓜倾斜”。同样，在大数据处理过程中，一个数据处理任务的数据量可能有几百 GB 或者几十 TB，对应会有几百甚至几千台机器一起处理。如果某一台机器被分配的数据量比其他机器多几十倍甚至几百倍，那就是我们说的数据倾斜了。</p>
<p data-nodeid="88831">这样的情况造成的结果是，数据处理任务结束时间因为其中一台机器而延后，处理效率被严重降低。避免数据倾斜非常重要。前几年数据分析师面试的时候，面试官经常考应聘者数据倾斜的处理问题。现在解决数据倾斜的方案更加成熟，社区考虑 Hive 使用过程中会出现这种情况，进而提供了比较好的解决方案。现在，仅仅需要你配置如下参数分两部分进行。</p>
<p data-nodeid="88832">第一部分：有些聚合可以先在 Map 端进行，然后进入 Reduce 端，这时数据量会小很多。我们再得出最终结果即可。</p>
<pre class="lang-java" data-nodeid="88833"><code data-language="java">set hive.map.aggr=<span class="hljs-keyword">true</span>; -- 开启Map端聚合参数设置
set hive.grouby.mapaggr.checkinterval=<span class="hljs-number">100000</span>; -- 在Map端进行聚合操作的条目数目
</code></pre>
<p data-nodeid="88834">第二部分：如下参数将会添加一个 MapReduce 任务，目的就是让数据平均分配到各个 reduce 上，这样基本能解决数据倾斜问题。</p>
<pre class="lang-java" data-nodeid="88835"><code data-language="java">set hive.groupby.skewindata = <span class="hljs-keyword">true</span>; -- 有数据倾斜的时候进行负载均衡（默认是<span class="hljs-keyword">false</span>）
</code></pre>
<p data-nodeid="88836">上面通过实例解释了 Hive 进行数据统计中，为什么会遇到数据倾斜，以及如何解决数据倾斜。最后我们来分享下 Hive 如何合理控制 map 和 reduce 数量来优化数据处理效率。</p>
<h3 data-nodeid="88837">控制 map/reduce 任务数量</h3>
<h4 data-nodeid="88838">控制 map 数量</h4>
<p data-nodeid="88839">我们先来了解下什么情况要设置 map 数量。一般来讲，map 数量默认，不需要我们设置。Hadoop 运维老师们会监控和设置一个相对通用的值。一般情况下，默认值就能满足需求了。</p>
<p data-nodeid="88840">但是，当我们明确知道表的数据量不大，而 Hive 运行启动了几千个 map 的时候，就有必要减小 map 的数量了。好比 1000 个西瓜没必要安排 100 辆车去拉，安排 2 辆车就可以搞定了。</p>
<p data-nodeid="88841">另一方面，当我们发现 map 数量不多，但 map 运行速度极慢的时候。这时可以 check 下数据表，看看实际需求是不是很大？如果 Hive 启动的 map 数据比较少，就如同用 2 辆车去拉 10000 个西瓜，明显是不够的。</p>
<p data-nodeid="88842">假设如果真遇到上面情况，那么如何调整 map 数量？我们通常会采用以下两种方式解决。</p>
<ul data-nodeid="88843">
<li data-nodeid="88844">
<p data-nodeid="88845">第一种解决办法是<strong data-nodeid="89090">增加 mapper 个数</strong>。可以设置 set mapred.map.tasks= 一个很大的数值， 需要比系统默认的 map 数量大。</p>
</li>
<li data-nodeid="88846">
<p data-nodeid="88847">第二种解决办法是<strong data-nodeid="89096">减少 mapper 个数</strong>。set maperd.min.split.size= 一个数字，该数值单位是字节，比如设置 1GB，即为 1024000000，因为默认一个 mapper 是 64MB，这样设置就可以让一个 mapper 处理 1GB 数据，自然 mapper 的数量也就减少了。</p>
</li>
</ul>
<p data-nodeid="88848">上面介绍 Hive 任务在什么情况下需要调整 map 数量， 以及如何设置 map 数量，下面将对 reducer 数量的控制。</p>
<h4 data-nodeid="88849">控制 reducer 数量</h4>
<p data-nodeid="88850">相比调整 mapper 数量，调整 reducer 数量的场景在工作中相对要多一些。当然，一般情况下我们也是不需要调整的。有些同学认为设置更多 reducer 会加快计算任务，但其实结果却不尽人意。</p>
<p data-nodeid="88851">比如：当数据经过 map 操作后输出的数据结果只有 1KB 大小，却要启动 100 个 reducer 去处理。这时候启动和消耗 reducer 的时间就会很多， 最后会形成 100 个非常小的文件落盘，这对以后的任务计算来讲都是负担。相反，如果 mapper 输出的数据结果很大。有 100 GB，却只有 2 个 reducer 去处理，那必然也会浪费很长时间。这个时候，就需要调节 reducer 数量了。</p>
<p data-nodeid="88852">我们来看如何调整 reducer 数量，这里主要有两种调整方式。</p>
<p data-nodeid="88853">方式一：通过设置每个 reducer 处理的数量大小，最多 reducer 数量来间接控制 reducer 数据。</p>
<pre class="lang-sql" data-nodeid="88854"><code data-language="sql"><span class="hljs-keyword">set</span> hive.exec.reducers.bytes.per.reducer=一个数字(默认<span class="hljs-number">1</span>GB, 即<span class="hljs-number">1024000000</span>)
<span class="hljs-keyword">set</span> hive.exec.reducers.max=一个数字(默认为<span class="hljs-number">999</span>)
</code></pre>
<p data-nodeid="88855">方式二：直接设置 reducer 数量。比如下面设置 reducer 为 500，无论数据大小，都会强制启动 500 个 reducer 处理数据。</p>
<pre data-nodeid="88856"><code>set mapred.reduce.tasks=500
</code></pre>
<p data-nodeid="88857">上面分享的对 reducer 数量的控制，可能工作中会遇到。尤其在多个大表进行 join 操作的时候，希望你可以掌握使用的场景和方法。</p>
<h3 data-nodeid="88858">总结</h3>
<p data-nodeid="88859">关于 Hive 使用的技巧有很多。这节课给出的技巧主要来自我工作 5 年多的使用经验，挑选出的一些比较有价值的核心处理技巧。我把这些技巧分享给你，这里非常感谢你的学习，之后 Hive 相关问题，可以留言交流学习。</p>
<hr data-nodeid="88860">
<p data-nodeid="88861" class="">Hive 是数据分析师经常用到的工具。取经儿老师在课程中介绍了不少 Hive 知识，但 Hive 的使用技巧无法在一篇文章内讲完。 想要了解更多 Hive 的知识，也可以点击“<a href="https://shenceyun.lagou.com/t/Jka" data-nodeid="89110">数据分析实战训练营</a>”进行学习哦。</p>

---

### 精选评论

##### 123：
> 数据分析的课程，为什么要讲十大数据呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课程数据分析分两部分讲解，一部分讲数据处理，一部分讲业务。这个10大数据处理属于技能部分。

##### *凯：
> 不先讲一下怎么安装使用hive吗？之前使用sql的时候都是windows系统，hive应该要Linux系统吧，很多人应该都完全没接触过，如果他已经对hive有一定了解会安装使用了解语法了，还有什么必要看这个呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实际工作还真不是这样的，很多工具的安装都是公司运维团队搞定，比如，hive的工作环境一般公司会提供，一般是数仓、运维来搞定环境，这样大部分数分不用来搞定环境的问题，只需要专心写sql进行业务分析。

##### **2952：
> 是否有时间函数从 提取yyyymmddhh或yyyymmdd,且保持长度一致，比如hour()提取出08而不是8？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个建议直接使用substr函数，比如: substr('20200801 09:30:00', 10, 2) 即，输出'09'

