<p data-nodeid="2417" class="">我们在第 02 课时中使用 Flink Table &amp; SQL 的 API 实现了最简单的 WordCount 程序。在这一课时中，将分别从 Flink Table &amp; SQL 的背景和编程模型、常见的 API、算子和内置函数等对 Flink Table &amp; SQL 做一个详细的讲解和概括，最后模拟了一个实际业务场景使用 Flink Table &amp; SQL 开发。</p>
<h3 data-nodeid="2418">Flink Table &amp; SQL 概述</h3>
<h4 data-nodeid="2419">背景</h4>
<p data-nodeid="2420">我们在前面的课时中讲过 Flink 的分层模型，Flink 自身提供了不同级别的抽象来支持我们开发流式或者批量处理程序，下图描述了 Flink 支持的 4 种不同级别的抽象。</p>
<p data-nodeid="2421"><img src="https://s0.lgstatic.com/i/image3/M01/16/F1/Ciqah16mnASAOZj0AACsqOUWhS0726.png" alt="image.png" data-nodeid="2542"></p>
<p data-nodeid="2422"><strong data-nodeid="2551">Table API</strong> 和 <strong data-nodeid="2552">SQL</strong> 处于最顶端，是 Flink 提供的高级 API 操作。Flink SQL 是 Flink 实时计算为简化计算模型，降低用户使用实时计算门槛而设计的一套符合标准 SQL 语义的开发语言。</p>
<p data-nodeid="2423">我们在第 04 课时中提到过，Flink 在编程模型上提供了 DataStream 和 DataSet 两套 API，并没有做到事实上的批流统一，因为用户和开发者还是开发了两套代码。正是因为 Flink Table &amp; SQL 的加入，可以说 Flink 在某种程度上做到了事实上的批流一体。</p>
<h4 data-nodeid="2424">原理</h4>
<p data-nodeid="2425">你之前可能都了解过 Hive，在离线计算场景下 Hive 几乎扛起了离线数据处理的半壁江山。它的底层对 SQL 的解析用到了 Apache Calcite，Flink 同样把 SQL 的解析、优化和执行教给了 Calcite。</p>
<p data-nodeid="2426">下图是一张经典的 Flink Table &amp; SQL 实现原理图，可以看到 Calcite 在整个架构中处于绝对核心地位。</p>
<p data-nodeid="2427"><img src="https://s0.lgstatic.com/i/image3/M01/16/F1/Ciqah16mnGWASOD-AAJ7jTakc-4812.png" alt="image (1).png" data-nodeid="2563"><br>
从图中可以看到无论是批查询 SQL 还是流式查询 SQL，都会经过对应的转换器 Parser 转换成为节点树 SQLNode tree，然后生成逻辑执行计划 Logical Plan，逻辑执行计划在经过优化后生成真正可以执行的物理执行计划，交给 DataSet 或者 DataStream 的 API 去执行。</p>
<p data-nodeid="2428">在这里我们不对 Calcite 的原理过度展开，有兴趣的可以直接在官网上学习。</p>
<p data-nodeid="2429">一个完整的 Flink Table &amp; SQL Job 也是由 Source、Transformation、Sink 构成：</p>
<p data-nodeid="2430"><img src="https://s0.lgstatic.com/i/image3/M01/16/F1/Ciqah16mnHiAa99JAAFeAnFYzIE146.png" alt="image (2).png" data-nodeid="2572"></p>
<ul data-nodeid="2431">
<li data-nodeid="2432">
<p data-nodeid="2433"><strong data-nodeid="2577">Source 部分</strong>来源于外部数据源，我们经常用的有 Kafka、MySQL 等；</p>
</li>
<li data-nodeid="2434">
<p data-nodeid="2435"><strong data-nodeid="2584">Transformation 部分</strong>则是 Flink Table &amp; SQL 支持的常用 SQL 算子，比如简单的 Select、Groupby 等，当然在这里也有更为复杂的多流 Join、流与维表的 Join 等；</p>
</li>
<li data-nodeid="2436">
<p data-nodeid="2437"><strong data-nodeid="2589">Sink 部分</strong>是指的结果存储比如 MySQL、HBase 或 Kakfa 等。</p>
</li>
</ul>
<h4 data-nodeid="2438">动态表</h4>
<p data-nodeid="2439">与传统的表 SQL 查询相比，Flink Table &amp; SQL 在处理流数据时会时时刻刻处于动态的数据变化中，所以便有了一个<strong data-nodeid="2598">动态表</strong>的概念。</p>
<p data-nodeid="2440">动态表的查询与静态表一样，但是，在查询动态表的时候，SQL 会做连续查询，不会终止。</p>
<p data-nodeid="2441">我们举个简单的例子，Flink 程序接受一个 Kafka 流作为输入，Kafka 中为用户的购买记录：</p>
<p data-nodeid="2442"><img src="https://s0.lgstatic.com/i/image3/M01/16/F2/Ciqah16mnKaASUgqAAHL5tHLarA558.png" alt="image (3).png" data-nodeid="2603"></p>
<p data-nodeid="2443">首先，Kafka 的消息会被源源不断的解析成一张不断增长的动态表，我们在动态表上执行的 SQL 会不断生成新的动态表作为结果表。</p>
<h3 data-nodeid="2444">Flink Table &amp; SQL 算子和内置函数</h3>
<p data-nodeid="2445">我们在讲解 Flink Table &amp; SQL 所支持的常用算子前，需要说明一点，Flink 自从 0.9 版本开始支持 Table &amp; SQL 功能一直处于完善开发中，且在不断进行迭代。</p>
<p data-nodeid="2446">我们在官网中也可以看到这样的提示：</p>
<blockquote data-nodeid="2447">
<p data-nodeid="2448">Please note that the Table API and SQL are not yet feature complete and are being actively developed. Not all operations are supported by every combination of [Table API, SQL] and [stream, batch] input.</p>
</blockquote>
<p data-nodeid="2449">Flink Table &amp; SQL 的开发一直在进行中，并没有支持所有场景下的计算逻辑。从我个人实践角度来讲，在使用原生的 Flink Table &amp; SQL 时，务必查询官网当前版本对 Table &amp; SQL 的支持程度，尽量选择场景明确，逻辑不是极其复杂的场景。</p>
<h4 data-nodeid="2450">常用算子</h4>
<p data-nodeid="2451">目前 Flink SQL 支持的语法主要如下：</p>
<pre class="lang-SQL" data-nodeid="2452"><code data-language="SQL">query:
  <span class="hljs-keyword">values</span>
  | {
      <span class="hljs-keyword">select</span>
      | selectWithoutFrom
      | <span class="hljs-keyword">query</span> <span class="hljs-keyword">UNION</span> [ <span class="hljs-keyword">ALL</span> ] <span class="hljs-keyword">query</span>
      | <span class="hljs-keyword">query</span> <span class="hljs-keyword">EXCEPT</span> <span class="hljs-keyword">query</span>
      | <span class="hljs-keyword">query</span> <span class="hljs-keyword">INTERSECT</span> <span class="hljs-keyword">query</span>
    }
    [ <span class="hljs-keyword">ORDER</span> <span class="hljs-keyword">BY</span> orderItem [, orderItem ]* ]
    [ <span class="hljs-keyword">LIMIT</span> { <span class="hljs-keyword">count</span> | <span class="hljs-keyword">ALL</span> } ]
    [ <span class="hljs-keyword">OFFSET</span> <span class="hljs-keyword">start</span> { <span class="hljs-keyword">ROW</span> | <span class="hljs-keyword">ROWS</span> } ]
    [ <span class="hljs-keyword">FETCH</span> { <span class="hljs-keyword">FIRST</span> | <span class="hljs-keyword">NEXT</span> } [ <span class="hljs-keyword">count</span> ] { <span class="hljs-keyword">ROW</span> | <span class="hljs-keyword">ROWS</span> } <span class="hljs-keyword">ONLY</span>]

orderItem:
  expression [ <span class="hljs-keyword">ASC</span> | <span class="hljs-keyword">DESC</span> ]

<span class="hljs-keyword">select</span>:
  <span class="hljs-keyword">SELECT</span> [ <span class="hljs-keyword">ALL</span> | <span class="hljs-keyword">DISTINCT</span> ]
  { * | projectItem [, projectItem ]* }
  <span class="hljs-keyword">FROM</span> tableExpression
  [ <span class="hljs-keyword">WHERE</span> booleanExpression ]
  [ <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> { groupItem [, groupItem ]* } ]
  [ <span class="hljs-keyword">HAVING</span> booleanExpression ]
  [ <span class="hljs-keyword">WINDOW</span> windowName <span class="hljs-keyword">AS</span> windowSpec [, windowName <span class="hljs-keyword">AS</span> windowSpec ]* ]

selectWithoutFrom:
  <span class="hljs-keyword">SELECT</span> [ <span class="hljs-keyword">ALL</span> | <span class="hljs-keyword">DISTINCT</span> ]
  { * | projectItem [, projectItem ]* }

projectItem:
  expression [ [ <span class="hljs-keyword">AS</span> ] columnAlias ]
  | tableAlias . *

tableExpression:
  tableReference [, tableReference ]*
  | tableExpression [ <span class="hljs-keyword">NATURAL</span> ] [ <span class="hljs-keyword">LEFT</span> | <span class="hljs-keyword">RIGHT</span> | <span class="hljs-keyword">FULL</span> ] <span class="hljs-keyword">JOIN</span> tableExpression [ joinCondition ]

joinCondition:
  <span class="hljs-keyword">ON</span> booleanExpression
  | <span class="hljs-keyword">USING</span> <span class="hljs-string">'('</span> <span class="hljs-keyword">column</span> [, <span class="hljs-keyword">column</span> ]* <span class="hljs-string">')'</span>

tableReference:
  tablePrimary
  [ matchRecognize ]
  [ [ <span class="hljs-keyword">AS</span> ] <span class="hljs-keyword">alias</span> [ <span class="hljs-string">'('</span> columnAlias [, columnAlias ]* <span class="hljs-string">')'</span> ] ]

tablePrimary:
  [ <span class="hljs-keyword">TABLE</span> ] [ [ catalogName . ] schemaName . ] tableName
  | <span class="hljs-keyword">LATERAL</span> <span class="hljs-keyword">TABLE</span> <span class="hljs-string">'('</span> functionName <span class="hljs-string">'('</span> expression [, expression ]* <span class="hljs-string">')'</span> <span class="hljs-string">')'</span>
  | <span class="hljs-keyword">UNNEST</span> <span class="hljs-string">'('</span> expression <span class="hljs-string">')'</span>

<span class="hljs-keyword">values</span>:
  <span class="hljs-keyword">VALUES</span> expression [, expression ]*

groupItem:
  expression
  | <span class="hljs-string">'('</span> <span class="hljs-string">')'</span>
  | <span class="hljs-string">'('</span> expression [, expression ]* <span class="hljs-string">')'</span>
  | <span class="hljs-keyword">CUBE</span> <span class="hljs-string">'('</span> expression [, expression ]* <span class="hljs-string">')'</span>
  | <span class="hljs-keyword">ROLLUP</span> <span class="hljs-string">'('</span> expression [, expression ]* <span class="hljs-string">')'</span>
  | <span class="hljs-keyword">GROUPING</span> <span class="hljs-keyword">SETS</span> <span class="hljs-string">'('</span> groupItem [, groupItem ]* <span class="hljs-string">')'</span>

windowRef:
    windowName
  | windowSpec

windowSpec:
    [ windowName ]
    <span class="hljs-string">'('</span>
    [ <span class="hljs-keyword">ORDER</span> <span class="hljs-keyword">BY</span> orderItem [, orderItem ]* ]
    [ <span class="hljs-keyword">PARTITION</span> <span class="hljs-keyword">BY</span> expression [, expression ]* ]
    [
        <span class="hljs-keyword">RANGE</span> numericOrIntervalExpression {<span class="hljs-keyword">PRECEDING</span>}
      | <span class="hljs-keyword">ROWS</span> numericExpression {<span class="hljs-keyword">PRECEDING</span>}
    ]
    <span class="hljs-string">')'</span>
...
</code></pre>
<p data-nodeid="2453">可以看到 Flink SQL 和传统的 SQL 一样，支持了包含查询、连接、聚合等场景，另外还支持了包括窗口、排序等场景。下面我就以最常用的算子来做详细的讲解。</p>
<p data-nodeid="2454"><strong data-nodeid="2636">SELECT/AS/WHERE</strong></p>
<p data-nodeid="2455">SELECT、WHERE 和传统 SQL 用法一样，用于筛选和过滤数据，同时适用于 DataStream 和 DataSet。</p>
<pre class="lang-SQL" data-nodeid="2456"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span>;
<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>，age <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span>;
</code></pre>
<p data-nodeid="2457">当然我们也可以在 WHERE 条件中使用 =、&lt;、&gt;、&lt;&gt;、&gt;=、&lt;=，以及 AND、OR 等表达式的组合：</p>
<pre class="lang-SQL" data-nodeid="2458"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>，age <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span> <span class="hljs-keyword">where</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">LIKE</span> <span class="hljs-string">'%小明%'</span>;
<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span> <span class="hljs-keyword">WHERE</span> age = <span class="hljs-number">20</span>;
<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>, age
<span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span>
<span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">IN</span> (<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">FROM</span> Table2)
</code></pre>
<p data-nodeid="2459"><strong data-nodeid="2648">GROUP BY / DISTINCT/HAVING</strong></p>
<p data-nodeid="2460">GROUP BY 用于进行分组操作，DISTINCT 用于结果去重。<br>
HAVING 和传统 SQL 一样，可以用来在聚合函数之后进行筛选。</p>
<pre class="lang-SQL" data-nodeid="2461"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">DISTINCT</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span>;
<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>, <span class="hljs-keyword">SUM</span>(score) <span class="hljs-keyword">as</span> TotalScore <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span> <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> <span class="hljs-keyword">name</span>;
<span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">name</span>, <span class="hljs-keyword">SUM</span>(score) <span class="hljs-keyword">as</span> TotalScore <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">Table</span> <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> <span class="hljs-keyword">name</span> <span class="hljs-keyword">HAVING</span>
<span class="hljs-keyword">SUM</span>(score) &gt; <span class="hljs-number">300</span>;
</code></pre>
<p data-nodeid="2462"><strong data-nodeid="2655">JOIN</strong></p>
<p data-nodeid="2463">JOIN 可以用于把来自两个表的数据联合起来形成结果表，目前 Flink 的 Join 只支持等值连接。Flink 支持的 JOIN 类型包括：</p>
<pre class="lang-SQL" data-nodeid="2464"><code data-language="SQL">JOIN - INNER JOIN
LEFT JOIN - LEFT OUTER JOIN
RIGHT JOIN - RIGHT OUTER JOIN
FULL JOIN - FULL OUTER JOIN
</code></pre>
<p data-nodeid="2465">例如，用用户表和商品表进行关联：</p>
<pre class="lang-SQL" data-nodeid="2466"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> *
<span class="hljs-keyword">FROM</span> <span class="hljs-keyword">User</span> <span class="hljs-keyword">LEFT</span> <span class="hljs-keyword">JOIN</span> Product <span class="hljs-keyword">ON</span> User.name = Product.buyer

<span class="hljs-keyword">SELECT</span> *
<span class="hljs-keyword">FROM</span> <span class="hljs-keyword">User</span> <span class="hljs-keyword">RIGHT</span> <span class="hljs-keyword">JOIN</span> Product <span class="hljs-keyword">ON</span> User.name = Product.buyer

<span class="hljs-keyword">SELECT</span> *
<span class="hljs-keyword">FROM</span> <span class="hljs-keyword">User</span> <span class="hljs-keyword">FULL</span> <span class="hljs-keyword">OUTER</span> <span class="hljs-keyword">JOIN</span> Product <span class="hljs-keyword">ON</span> User.name = Product.buyer
</code></pre>
<p data-nodeid="2467">LEFT JOIN、RIGHT JOIN 、FULL JOIN 相与我们传统 SQL 中含义一样。</p>
<p data-nodeid="2468"><strong data-nodeid="2662">WINDOW</strong></p>
<p data-nodeid="2469">根据窗口数据划分的不同，目前 Apache Flink 有如下 3 种：</p>
<ul data-nodeid="2470">
<li data-nodeid="2471">
<p data-nodeid="2472"><strong data-nodeid="2668">滚动窗口</strong>，窗口数据有固定的大小，窗口中的数据不会叠加；</p>
</li>
<li data-nodeid="2473">
<p data-nodeid="2474"><strong data-nodeid="2673">滑动窗口</strong>，窗口数据有固定大小，并且有生成间隔；</p>
</li>
<li data-nodeid="2475">
<p data-nodeid="2476"><strong data-nodeid="2678">会话窗口</strong>，窗口数据没有固定的大小，根据用户传入的参数进行划分，窗口数据无叠加；</p>
</li>
</ul>
<p data-nodeid="2477"><strong data-nodeid="2682">滚动窗口</strong></p>
<p data-nodeid="2478">滚动窗口的特点是：有固定大小、窗口中的数据不会重叠，如下图所示：<br>
<img src="https://s0.lgstatic.com/i/image3/M01/16/F3/Ciqah16mnZSAP4G5AAEyag8FwPM908.png" alt="image (4).png" data-nodeid="2687"></p>
<p data-nodeid="2479">滚动窗口的语法：</p>
<pre class="lang-SQL" data-nodeid="2480"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> 
    [gk],
    [TUMBLE_START(timeCol, <span class="hljs-keyword">size</span>)], 
    [TUMBLE_END(timeCol, <span class="hljs-keyword">size</span>)], 
    agg1(col1), 
    ... 
    aggn(colN)
<span class="hljs-keyword">FROM</span> Tab1
<span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> [gk], TUMBLE(timeCol, <span class="hljs-keyword">size</span>)
</code></pre>
<p data-nodeid="2481">举例说明，我们需要计算每个用户每天的订单数量：</p>
<pre class="lang-SQL" data-nodeid="2482"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">user</span>, TUMBLE_START(timeLine, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">DAY</span>) <span class="hljs-keyword">as</span> winStart, <span class="hljs-keyword">SUM</span>(amount) <span class="hljs-keyword">FROM</span> Orders <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> TUMBLE(timeLine, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">DAY</span>), <span class="hljs-keyword">user</span>;
</code></pre>
<p data-nodeid="2483">其中，TUMBLE_START 和 TUMBLE_END 代表窗口的开始时间和窗口的结束时间，TUMBLE (timeLine, INTERVAL '1' DAY) 中的 timeLine 代表时间字段所在的列，INTERVAL '1' DAY 表示时间间隔为一天。</p>
<p data-nodeid="2484"><strong data-nodeid="2706">滑动窗口</strong></p>
<p data-nodeid="2485">滑动窗口有固定的大小，与滚动窗口不同的是滑动窗口可以通过 slide 参数控制滑动窗口的创建频率。需要注意的是，多个滑动窗口可能会发生数据重叠，具体语义如下：</p>
<p data-nodeid="2486"><img src="https://s0.lgstatic.com/i/image3/M01/09/C4/CgoCgV6mnceAEzZNAAEs6POgpPQ859.png" alt="image (5).png" data-nodeid="2710"></p>
<p data-nodeid="2487">滑动窗口的语法与滚动窗口相比，只多了一个 slide 参数：</p>
<pre class="lang-SQL" data-nodeid="2488"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> 
    [gk], 
    [HOP_START(timeCol, slide, <span class="hljs-keyword">size</span>)] ,
    [HOP_END(timeCol, slide, <span class="hljs-keyword">size</span>)],
    agg1(col1), 
    ... 
    aggN(colN) 
<span class="hljs-keyword">FROM</span> Tab1
<span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> [gk], HOP(timeCol, slide, <span class="hljs-keyword">size</span>)
</code></pre>
<p data-nodeid="2489">例如，我们要每间隔一小时计算一次过去 24 小时内每个商品的销量：</p>
<pre class="lang-SQL" data-nodeid="2490"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> product, <span class="hljs-keyword">SUM</span>(amount) <span class="hljs-keyword">FROM</span> Orders <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> HOP(rowtime, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">HOUR</span>, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">DAY</span>), product
</code></pre>
<p data-nodeid="2491">上述案例中的 INTERVAL '1' HOUR 代表滑动窗口生成的时间间隔。</p>
<p data-nodeid="2492"><strong data-nodeid="2721">会话窗口</strong></p>
<p data-nodeid="2493">会话窗口定义了一个非活动时间，假如在指定的时间间隔内没有出现事件或消息，则会话窗口关闭。</p>
<p data-nodeid="2494"><img src="https://s0.lgstatic.com/i/image3/M01/09/C4/CgoCgV6mngCAVToGAAEyMz6pyiE117.png" alt="image (6).png" data-nodeid="2725"><br>
会话窗口的语法如下：</p>
<pre class="lang-SQL" data-nodeid="2495"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> 
    [gk], 
    SESSION_START(timeCol, gap) <span class="hljs-keyword">AS</span> winStart,
    SESSION_END(timeCol, gap) <span class="hljs-keyword">AS</span> winEnd,
    agg1(col1),
     ... 
    aggn(colN)
<span class="hljs-keyword">FROM</span> Tab1
<span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> [gk], <span class="hljs-keyword">SESSION</span>(timeCol, gap)
</code></pre>
<p data-nodeid="2496">举例，我们需要计算每个用户过去 1 小时内的订单量：</p>
<pre class="lang-SQL" data-nodeid="2497"><code data-language="SQL"><span class="hljs-keyword">SELECT</span> <span class="hljs-keyword">user</span>, SESSION_START(rowtime, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">HOUR</span>) <span class="hljs-keyword">AS</span> sStart, SESSION_ROWTIME(rowtime, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">HOUR</span>) <span class="hljs-keyword">AS</span> sEnd, <span class="hljs-keyword">SUM</span>(amount) <span class="hljs-keyword">FROM</span> Orders <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> <span class="hljs-keyword">SESSION</span>(rowtime, <span class="hljs-built_in">INTERVAL</span> <span class="hljs-string">'1'</span> <span class="hljs-keyword">HOUR</span>), <span class="hljs-keyword">user</span>
</code></pre>
<h4 data-nodeid="2498">内置函数</h4>
<p data-nodeid="2499">Flink 中还有大量的内置函数，我们可以直接使用，将内置函数分类如下：</p>
<ul data-nodeid="2500">
<li data-nodeid="2501">
<p data-nodeid="2502">比较函数</p>
</li>
<li data-nodeid="2503">
<p data-nodeid="2504">逻辑函数</p>
</li>
<li data-nodeid="2505">
<p data-nodeid="2506">算术函数</p>
</li>
<li data-nodeid="2507">
<p data-nodeid="2508">字符串处理函数</p>
</li>
<li data-nodeid="2509">
<p data-nodeid="2510">时间函数</p>
</li>
</ul>
<p data-nodeid="2511"><strong data-nodeid="2743">比较函数</strong><br>
<img src="https://s0.lgstatic.com/i/image3/M01/09/D0/CgoCgV6mqKWAfZRoAAEspJ_3kgs250.png" alt="比较函数.png" data-nodeid="2742"></p>
<p data-nodeid="2512"><strong data-nodeid="2751">逻辑函数</strong><br>
<img src="https://s0.lgstatic.com/i/image3/M01/09/D0/CgoCgV6mqNCAODW7AAB_4lfsI5k037.png" alt="逻辑函数.png" data-nodeid="2750"></p>
<p data-nodeid="2513"><strong data-nodeid="2759">算术函数</strong><br>
<img src="https://s0.lgstatic.com/i/image3/M01/09/D1/CgoCgV6mqNeALwSaAABekYvTZkw048.png" alt="算术函数.png" data-nodeid="2758"></p>
<p data-nodeid="2514"><strong data-nodeid="2767">字符串处理函数</strong><br>
<img src="https://s0.lgstatic.com/i/image3/M01/09/D1/CgoCgV6mqN2AEnGrAABlm2eH_OY974.png" alt="字符串处理函数.png" data-nodeid="2766"></p>
<p data-nodeid="2515"><strong data-nodeid="2775">时间函数</strong><br>
<img src="https://s0.lgstatic.com/i/image3/M01/09/D1/CgoCgV6mqOOAcYBFAACiFeqRUIM306.png" alt="时间函数.png" data-nodeid="2774"></p>
<h3 data-nodeid="2516">Flink Table &amp; SQL 案例</h3>
<p data-nodeid="2517">上面分别介绍了 Flink Table &amp; SQL 的原理和支持的算子，我们模拟一个实时的数据流，然后讲解 SQL JOIN 的用法。</p>
<p data-nodeid="2518">在上一课时中，我们利用 Flink 提供的自定义 Source 功能来实现一个自定义的实时数据源，具体实现如下：</p>
<pre class="lang-JAVA" data-nodeid="2519"><code data-language="JAVA"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyStreamingSource</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SourceFunction</span>&lt;<span class="hljs-title">Item</span>&gt; </span>{

    <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isRunning = <span class="hljs-keyword">true</span>;

    <span class="hljs-comment">/**
     * 重写run方法产生一个源源不断的数据发送源
     * <span class="hljs-doctag">@param</span> ctx
     * <span class="hljs-doctag">@throws</span> Exception
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(SourceContext&lt;Item&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">while</span>(isRunning){
            Item item = generateItem();
            ctx.collect(item);

            <span class="hljs-comment">//每秒产生一条数据</span>
            Thread.sleep(<span class="hljs-number">1000</span>);
        }
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cancel</span><span class="hljs-params">()</span> </span>{
        isRunning = <span class="hljs-keyword">false</span>;
    }

    <span class="hljs-comment">//随机产生一条商品数据</span>
    <span class="hljs-function"><span class="hljs-keyword">private</span> Item <span class="hljs-title">generateItem</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">int</span> i = <span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">100</span>);
        ArrayList&lt;String&gt; list = <span class="hljs-keyword">new</span> ArrayList();
        list.add(<span class="hljs-string">"HAT"</span>);
        list.add(<span class="hljs-string">"TIE"</span>);
        list.add(<span class="hljs-string">"SHOE"</span>);
        Item item = <span class="hljs-keyword">new</span> Item();
        item.setName(list.get(<span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">3</span>)));
        item.setId(i);
        <span class="hljs-keyword">return</span> item;
    }
}
</code></pre>
<p data-nodeid="2520">我们把实时的商品数据流进行分流，分成 even 和 odd 两个流进行 JOIN，条件是名称相同，最后，把两个流的 JOIN 结果输出。</p>
<pre class="lang-JAVA" data-nodeid="2521"><code data-language="JAVA"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StreamingDemo</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        EnvironmentSettings bsSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();
        StreamExecutionEnvironment bsEnv = StreamExecutionEnvironment.getExecutionEnvironment();
        StreamTableEnvironment bsTableEnv = StreamTableEnvironment.create(bsEnv, bsSettings);

        SingleOutputStreamOperator&lt;Item&gt; source = bsEnv.addSource(<span class="hljs-keyword">new</span> MyStreamingSource()).map(<span class="hljs-keyword">new</span> MapFunction&lt;Item, Item&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Item <span class="hljs-title">map</span><span class="hljs-params">(Item item)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                <span class="hljs-keyword">return</span> item;
            }
        });

        DataStream&lt;Item&gt; evenSelect = source.split(<span class="hljs-keyword">new</span> OutputSelector&lt;Item&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Iterable&lt;String&gt; <span class="hljs-title">select</span><span class="hljs-params">(Item value)</span> </span>{
                List&lt;String&gt; output = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
                <span class="hljs-keyword">if</span> (value.getId() % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>) {
                    output.add(<span class="hljs-string">"even"</span>);
                } <span class="hljs-keyword">else</span> {
                    output.add(<span class="hljs-string">"odd"</span>);
                }
                <span class="hljs-keyword">return</span> output;
            }
        }).select(<span class="hljs-string">"even"</span>);

        DataStream&lt;Item&gt; oddSelect = source.split(<span class="hljs-keyword">new</span> OutputSelector&lt;Item&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Iterable&lt;String&gt; <span class="hljs-title">select</span><span class="hljs-params">(Item value)</span> </span>{
                List&lt;String&gt; output = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
                <span class="hljs-keyword">if</span> (value.getId() % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>) {
                    output.add(<span class="hljs-string">"even"</span>);
                } <span class="hljs-keyword">else</span> {
                    output.add(<span class="hljs-string">"odd"</span>);
                }
                <span class="hljs-keyword">return</span> output;
            }
        }).select(<span class="hljs-string">"odd"</span>);


        bsTableEnv.createTemporaryView(<span class="hljs-string">"evenTable"</span>, evenSelect, <span class="hljs-string">"name,id"</span>);
        bsTableEnv.createTemporaryView(<span class="hljs-string">"oddTable"</span>, oddSelect, <span class="hljs-string">"name,id"</span>);

        Table queryTable = bsTableEnv.sqlQuery(<span class="hljs-string">"select a.id,a.name,b.id,b.name from evenTable as a join oddTable as b on a.name = b.name"</span>);

        queryTable.printSchema();

        bsTableEnv.toRetractStream(queryTable, TypeInformation.of(<span class="hljs-keyword">new</span> TypeHint&lt;Tuple4&lt;Integer,String,Integer,String&gt;&gt;(){})).print();

        bsEnv.execute(<span class="hljs-string">"streaming sql job"</span>);
    }

}
</code></pre>
<p data-nodeid="2522">直接右键运行，在控制台可以看到输出：</p>
<p data-nodeid="4691"><img src="https://s0.lgstatic.com/i/image3/M01/09/C6/CgoCgV6mnuqAa8d3AAINXpfEq-8636.png" alt="image (7).png" data-nodeid="4695"></p>
<p data-nodeid="9294" class="te-preview-highlight"><a href="https://github.com/wangzhiwubigdata/quickstart" data-nodeid="9297">点击这里下载本课程源码</a></p>







<h3 data-nodeid="4309">总结</h3>




<p data-nodeid="2525">我们在这一课时中讲解了 Flink Table &amp; SQL 的背景和原理，并且讲解了动态表的概念；同时对 Flink 支持的常用 SQL 和内置函数进行了讲解；最后用一个案例，讲解了整个 Flink Table &amp; SQL 的使用。</p>

---

### 精选评论

##### *磊：
> 看了好多个拉勾专栏了，flink这个专栏算是最良心的。从循序渐进的课程安排，到细致易懂的内容，到老师热心回复 都很好！感谢这个老师，希望拉勾别的专栏向这个看齐。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 也祝你学习愉快

##### *铁：
> 我本地搭建的环境是flink-1.11，我看文档createTemporaryView这个方法在1.11更新了，所以就改了一下示例bsTableEnv.createTemporaryView("evenTable", evenSelect, $("id"), $("name")); 不过运行到这里就报错就报错：Too many fields referenced from an atomic type. 求指教！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你使用了Tuple类，记得用flink的tuple。另外你需要使用TypeInformation来指定列的类型。

##### 123：
> 很赞

##### *熙：
> <div>您好，读完文章，有以下两个问题，还望解答下，感谢~~~</div><div><span style="font-size: 16.0125px;">1. 意思是sql 和 table api都会经过转换器转换成dataset或datastream api是么？</span></div><div><span style="font-size: 16.0125px;">2. 如果1成立的话，那是不是只要掌握dataset或者datastream原理的话，其实就是掌握了sql，table api是么？</span></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1.是的 2.sql的开发效率远大于原生api

##### **-4年-大数据开发：
> 讲师回复： 实际生产中双流join并不提倡，会遇到很多状态问题。一般会把其中一个流放入Hbase这样的存储中再去关联。文中的odd和even两个流是为了模拟join所需的输入。我现在生产有一张用户表，包含用户的基本信息和状态；大概4亿数据存储在hbase；如果把hbase当成维表关联性能怎么样；

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 事实上是可以的，HBase 的查询 qps 非常高，如果是维表的话，其实效率是没有问题的。我们在实际生产中 HBase 中的数据大概存半年，超过几十T的数据。

##### **生：
> 这个SQL，落地在哪里，如果我要做展示，我应该搞？比如说，我想用展示工具grafana显示结果。应该落地在哪里呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; grafna支持的数据源都可以的，例如MySql

##### *宁：
> 写的很用心，继续学习ing

##### Johngo：
> 这个使用的数据Flink哪个版本，我用1.10.0尝试实现sql相关，总是没办法跑通😔

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 开源的Flink的sql是非常不稳定的功能，如果你用了阿里云的Blink就会好些。

##### **蜗牛：
> 我有一句不理解：“Flink 自从 0.9 版本开始支持 Table &amp; SQL 功能一直处于完善开发中，且在不断进行迭代。”，是 1.9 还是 0.9 版本？？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 0.9就支持了，但是一直在更新

##### **姐：
> bsTableEnv.createTemporaryView("evenTable", evenSelect, "name,id");这行报Too many fields referenced from an atomic type.的错误，Item已经有无参构造器、getter和setter了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好好检查一下你的类，需要满足：
1.类必须是公有的。
2.它必须有一个公有的无参构造器（默认构造器）。
3.所有的字段要么是公有的要么必须可以通过 getter 和 setter 函数访问。例如一个名为 foo 的字段，它的 getter 和 setter 方法必须命名为 getFoo() 和 setFoo()。
4.字段的类型必须被已注册的序列化程序所支持。

##### *航：
> count window有类似time window这种SQL里面的TUMBLE写法吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 目前只支持 TimeWindow，相应的表达语句 TUMBLE、HOP、SESSION，CountWindow 不支持。

##### **9524：
> 临时表有100万行 10个字段 占用多少内存

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个跟每个字段的长度、类型都有关系吧。保守估计应该是几百M。

##### **发：
> 在">bsTableEnv.createTemporaryView("evenTable", evenSelect, "id,name"); 处报错：Exception in thread "main" org.apache.flink.table.api.ValidationException: Too many fields referenced from an atomic type.开始是Tuple4 类引入错了，后面纠正为Flink的了也不行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要给你的Java对象一个无参构造方法，最好加上getter和setter

##### **用户4300：
> Kafka 的消息会被源源不断的解析成一张不断增长的动态表，我想问一下老师，这动态表的大小数量的限制吗？如果源源不断的数据，是怎么存储这些状态的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 会根据配置，存储在内存或者第三方的数据库中

##### *少：
> 老师，我对最后一个例子的执行结果不是很理解。常规sql进行join是一个集合，观察了例子的执行结果是没有重复数据的，这是even和odd两个流增量元素的join结果吗？最后这个例子套用到具体业务上是一个什么样的场景呢？请老师讲解一下。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实际生产中双流join并不提倡，会遇到很多状态问题。一般会把其中一个流放入Hbase这样的存储中再去关联。文中的odd和even两个流是为了模拟join所需的输入。

##### **5317：
> 会话窗口那个例子，实际的含义是在1小时没有用户下单的时候，就计算每个用户的订单量吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是当前时间的过去一小时

##### *军：
> createTemporaryView() 方法的使用编译不通过，换成了<span style="background-color: transparent;"><font color="#008080" face="Menlo, Lucida Console, monospace"><span style="white-space: pre-wrap;">registerDataStream()可以使用，但是示例运行报错 'Exception in thread "main" java.lang.NoSuchMethodError: org.apache.flink.table.catalog.FunctionCatalog.&lt;init&gt;(Lorg/apache/flink/table/catalog/CatalogManager;)V'</span></font></span><div><span style="background-color: transparent;"><font color="#008080" face="Menlo, Lucida Console, monospace"><span style="white-space: pre-wrap;"><br></span></font></span></div><div><font color="#008080" face="Menlo, Lucida Console, monospace"><span style="white-space: pre-wrap;">是为啥</span></font></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 重新提交了POM依赖，再试试

##### **7651：
> 会话窗口案例，过去一小时的订单量，这个不太能理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 就是在一个窗口内，计算过去一个小时有多少订单

##### *曙：
> logic plan之后是StreamGraph ,不是生成物理执行计划吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

##### *帅：
> 最后一个例子输出用了toRetractStream，请问这里用toAppendStream/toUpsertStream是不是也可以？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

