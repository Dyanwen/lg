<p data-nodeid="284070" class="">前面我们花了几个课时对 ShardingSphere 中的 SQL 解析引擎做了介绍，我们明白 SQL 解析的作用就是根据输入的 SQL 语句生成一个 SQLStatement 对象。</p>
<p data-nodeid="284071">从今天开始，我们将进入 <strong data-nodeid="284193">ShardingSphere 的路由（Routing）引擎部分的源码解析</strong>。从流程上讲，<strong data-nodeid="284194">路由引擎</strong>是整个分片引擎执行流程中的第二步，即基于 SQL 解析引擎所生成的 SQLStatement，通过解析执行过程中所携带的上下文信息，来获取匹配数据库和表的分片策略，并生成路由结果。</p>
<h3 data-nodeid="284072">分层：路由引擎整体架构</h3>
<p data-nodeid="284073">与介绍 SQL 解析引擎时一样，我们通过翻阅 ShardingSphere 源码，首先梳理了如下所示的包结构：</p>
<p data-nodeid="285218"><img src="https://s0.lgstatic.com/i/image/M00/6A/D2/CgqCHl-pI5mAXrqWAACIykUr4yg379.png" alt="Lark20201109-190944.png" data-nodeid="285221"></p>



<p data-nodeid="284075">上述包图总结了与路由机制相关的各个核心类，我们可以看到整体呈一种对称结构，即根据是 <strong data-nodeid="284209">PreparedStatement</strong> 还是<strong data-nodeid="284210">普通 Statement</strong> 分成两个分支流程。</p>
<p data-nodeid="284076">同时，我们也可以把这张图中的类按照其所属的包结构<strong data-nodeid="284226">分成两个层次</strong>：位于底层的 sharding-core-route 和位于上层的 sharding-core-entry，这也是 ShardingSphere 中所普遍采用的一种分包原则，即<strong data-nodeid="284227">根据类的所属层级来组织包结构</strong>。关于 ShardingSphere 的分包原则我们在 <a href="https://kaiwu.lagou.com/course/courseInfo.htm?sid=&amp;courseId=257&amp;lagoufrom=noapp" data-nodeid="284224">《12 | 从应用到原理：如何高效阅读 ShardingSphere 源码？》</a>中也已经进行了介绍，接下来我们具体分析这一原则在路由引擎中的应用。</p>
<h4 data-nodeid="284077">1.sharding-core-route 工程</h4>
<p data-nodeid="284078">我们先来看图中的 ShardingRouter 类，该类是整个路由流程的启动点。ShardingRouter 类直接依赖于解析引擎 SQLParseEngine 类完成 SQL 解析并获取 SQLStatement 对象，然后供 PreparedStatementRoutingEngine 和 StatementRoutingEngine 进行使用。注意到这几个类都位于 sharding-core-route 工程中，<strong data-nodeid="284234">处于底层组件</strong>。</p>
<h4 data-nodeid="284079">2.sharding-core-entry 工程</h4>
<p data-nodeid="284080">另一方面，上图中的 PreparedQueryShardingEngine 和 SimpleQueryShardingEngine 则位于 sharding-core-entry 工程中。从包的命名上看，entry 相当于是访问的入口，所以我们可以判断这个工程中所提供的类<strong data-nodeid="284241">属于面向应用层组件</strong>，处于更加上层的位置。PreparedQueryShardingEngine 和 SimpleQueryShardingEngine 的使用者分别是 ShardingPreparedStatement 和 ShardingStatement。这两个类再往上就是 ShardingConnection 以及 ShardingDataSource 这些直接面向应用层的类了。</p>
<h3 data-nodeid="284081">路由核心类：ShardingRouter</h3>
<p data-nodeid="284082">通过以上分析，我们对路由引擎的整体结构有了一个初步的认识。对于采用分层结构的执行流程而言，有两种解析思路，即自上而下或自下而上。今天，我们的思路是<strong data-nodeid="284248">从底层出发逐层往上</strong>分析流程的链路，先来看路由引擎中最底层的对象 ShardingRouter，变量定义如下：</p>
<pre class="lang-java" data-nodeid="284083"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingRule shardingRule; 
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingSphereMetaData metaData; 
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SQLParseEngine parseEngine;
</code></pre>
<p data-nodeid="284084">在 ShardingRouter 中，我们首先看到了熟悉的 SQL 解析引擎 SQLParseEngine 以及它的使用方法：</p>
<pre class="lang-java" data-nodeid="284085"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> SQLStatement <span class="hljs-title">parse</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String logicSQL, <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> useCache)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> parseEngine.parse(logicSQL, useCache); 
}
</code></pre>
<p data-nodeid="284086">上述代码非常简单，即通过 SQLParseEngine 对传入的 SQL 进行解析返回一个 SQLStatement 对象。这里将 SQL 命名为 logicSQL，以便区别在分片和读写分离情况下的真实 SQL。</p>
<p data-nodeid="284087">接下来我们来看一下 ShardingRule，请注意这是一个基础类，代表着分片的各种规则信息。ShardingRule 类位于 sharding-core-common 工程中，主要保存着与分片相关的各种规则信息，以及 ShardingKeyGenerator 等分布式主键的创建过程，各个变量定义以及对应的注释如下所示：</p>
<pre class="lang-java" data-nodeid="284088"><code data-language="java"><span class="hljs-comment">//分片规则配置类，封装各种配置项信息 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingRuleConfiguration ruleConfiguration; 
<span class="hljs-comment">//DataSource 名称列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingDataSourceNames shardingDataSourceNames; 
<span class="hljs-comment">//针对表的规则列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;TableRule&gt; tableRules; 
<span class="hljs-comment">//针对绑定表的规则列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;BindingTableRule&gt; bindingTableRules; 
<span class="hljs-comment">//广播表名称列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;String&gt; broadcastTables; 
<span class="hljs-comment">//默认的数据库分片策略 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingStrategy defaultDatabaseShardingStrategy; 
<span class="hljs-comment">//默认的数据表分片策略 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingStrategy defaultTableShardingStrategy; 
<span class="hljs-comment">//默认的分片键生成器 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingKeyGenerator defaultShardingKeyGenerator; 
<span class="hljs-comment">//针对读写分离的规则列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;MasterSlaveRule&gt; masterSlaveRules; 
<span class="hljs-comment">//加密规则 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> EncryptRule encryptRule;
</code></pre>
<p data-nodeid="284089">ShardingRule 的内容非常丰富，但其定位更多是提供规则信息，而不属于核心流程，因此我们先不对其做详细展开。作为基础规则类，ShardingRule 会贯穿整个分片流程，在后续讲解过程中我们会穿插对它的介绍，这里先对上述变量的名称和含义有简单认识即可。</p>
<p data-nodeid="284090">我们回到 ShardingRouter 类，发现其核心方法只有一个，即 route 方法。这个方法的逻辑比较复杂，我们梳理它的执行步骤，如下图所示：</p>
<p data-nodeid="284091"><img src="https://s0.lgstatic.com/i/image/M00/3F/D0/CgqCHl8xJyqAHmcfAACVSxCxm4s053.png" alt="image (2).png" data-nodeid="284256"></p>
<p data-nodeid="284092">ShardingRouter 是路由引擎的核心类，<strong data-nodeid="284261">在接下来的内容中，我们将对上图中的 6 个步骤分别一 一 详细展开，帮忙你理解一个路由引擎的设计思想和实现机制。</strong></p>
<h4 data-nodeid="284093">1.分片合理性验证</h4>
<p data-nodeid="284094">我们首先来看 ShardingRouter 的第一个步骤，即验证分片信息的合理性，验证方式如下所示：</p>
<pre class="lang-java" data-nodeid="284095"><code data-language="java"><span class="hljs-comment">//使用ShardingStatementValidator对Statement进行验证 </span>
Optional&lt;ShardingStatementValidator&gt; shardingStatementValidator = ShardingStatementValidatorFactory.newInstance(sqlStatement); 
<span class="hljs-keyword">if</span> (shardingStatementValidator.isPresent()) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shardingStatementValidator.get().validate(shardingRule, sqlStatement, parameters); 
}
</code></pre>
<p data-nodeid="284096">这段代码使用 ShardingStatementValidator 对输入的 SQLStatement 进行验证，可以看到这里用到了典型的工厂模式，工厂类 ShardingStatementValidatorFactory 如下所示：</p>
<pre class="lang-java" data-nodeid="284097"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingStatementValidatorFactory</span> </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Optional&lt;ShardingStatementValidator&gt; <span class="hljs-title">newInstance</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SQLStatement sqlStatement)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> InsertStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Optional.&lt;ShardingStatementValidator&gt;of(<span class="hljs-keyword">new</span> ShardingInsertStatementValidator()); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> UpdateStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Optional.&lt;ShardingStatementValidator&gt;of(<span class="hljs-keyword">new</span> ShardingUpdateStatementValidator()); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Optional.absent(); 
&nbsp;&nbsp;&nbsp; } 
}
</code></pre>
<p data-nodeid="284098">注意到 ShardingStatementValidator 要验证的只有 InsertStatement 和 UpdateStatement 这两个 SQLStatement。那么如何进行验证呢？我们来看一下 ShardingStatementValidator 的定义，如下所示：</p>
<pre class="lang-java" data-nodeid="284099"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ShardingStatementValidator</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">SQLStatement</span>&gt; </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//验证分片操作是否支持 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">validate</span><span class="hljs-params">(ShardingRule shardingRule, T sqlStatement, List&lt;Object&gt; parameters)</span></span>; 
}
</code></pre>
<p data-nodeid="284100">对于验证过程而言，核心思想在于根据 SQLStatement 中的 Segment 与 ShardingRule 中的规则来判断它们之间是否有需要特殊处理的判断逻辑。我们以 ShardingInsertStatementValidator 为例来看验证过程，它的 validate 方法如下所示：</p>
<pre class="lang-java" data-nodeid="284101"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingInsertStatementValidator</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ShardingStatementValidator</span>&lt;<span class="hljs-title">InsertStatement</span>&gt; </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">validate</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> InsertStatement sqlStatement, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Optional&lt;OnDuplicateKeyColumnsSegment&gt; onDuplicateKeyColumnsSegment = sqlStatement.findSQLSegment(OnDuplicateKeyColumnsSegment.class); 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //如果是"ON DUPLICATE KEY UPDATE"语句，且如果当前操作的是分片Column时，验证不通过 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (onDuplicateKeyColumnsSegment.isPresent() &amp;&amp; isUpdateShardingKey(shardingRule, onDuplicateKeyColumnsSegment.get(), sqlStatement.getTable().getTableName())) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"INSERT INTO .... ON DUPLICATE KEY UPDATE can not support update for sharding column."</span>); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; … 
}
</code></pre>
<p data-nodeid="284102">可以看到这里的判断逻辑与“ON DUPLICATE KEY UPDATE”这一 Mysql 特有的语法相关，该语法允许我们通过 Update 的方式插入有重复主键的数据行（实际上这个语法也不是常规语法，本身也不大应该被使用）。</p>
<p data-nodeid="284103">ShardingInsertStatementValidator 先判断是否存在 OnDuplicateKeyColumn，然后再判断这个 Column 是否是分片键，如果同时满足这两个条件，则直接抛出一个异常，不允许在分片 Column 上执行“INSERT INTO .... ON DUPLICATE KEY UPDATE”语法。</p>
<h4 data-nodeid="284104">2.获取上下文</h4>
<p data-nodeid="284105">接下来我们来看 ShardingRouter 类中 route 方法的第二段代码，该段代码比较简单，用于获取运行时的 SQLStatement 上下文，如下所示：</p>
<pre class="lang-java" data-nodeid="284106"><code data-language="java"><span class="hljs-comment">//获取 SQLStatementContext </span>
SQLStatementContext sqlStatementContext = SQLStatementContextFactory.newInstance(metaData.getRelationMetas(), logicSQL, parameters, sqlStatement);
</code></pre>
<p data-nodeid="284107">可以看到这里构建了上下文对象 SQLStatementContext，同样用到了工厂模式，工厂类 SQLStatementContextFactory 如下所示：</p>
<pre class="lang-java" data-nodeid="284108"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SQLStatementContextFactory</span> </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> SQLStatementContext <span class="hljs-title">newInstance</span><span class="hljs-params">(<span class="hljs-keyword">final</span> RelationMetas relationMetas, <span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters, <span class="hljs-keyword">final</span> SQLStatement sqlStatement)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> SelectStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SelectSQLStatementContext(relationMetas, sql, parameters, (SelectStatement) sqlStatement); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> InsertStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> InsertSQLStatementContext(relationMetas, parameters, (InsertStatement) sqlStatement); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CommonSQLStatementContext(sqlStatement); 
&nbsp;&nbsp;&nbsp; } 
}
</code></pre>
<p data-nodeid="284109">请注意 SQLStatementContext 只有三种：</p>
<ul data-nodeid="284110">
<li data-nodeid="284111">
<p data-nodeid="284112">SelectSQLStatementContext</p>
</li>
<li data-nodeid="284113">
<p data-nodeid="284114">InsertSQLStatementContext</p>
</li>
<li data-nodeid="284115">
<p data-nodeid="284116">CommonSQLStatementContext</p>
</li>
</ul>
<p data-nodeid="284117">它们都实现了 SQLStatementContext 接口，顾名思义，所谓的 <strong data-nodeid="284281">SQLStatementContext 就是一种上下文对象</strong>，保存着与特定 SQLStatement 相关的上下文信息，用于为后续处理提供数据存储和传递的手段。</p>
<p data-nodeid="284118">我们可以想象在 SQLStatementContext 中势必都持有 SQLStatement 对象以及与表结构信息相关的上下文 TablesContext。</p>
<p data-nodeid="284119">对于 SelectSQLStatement，通常也需要保存与查询相关的分组上下文 GroupByContext、排序上下文 OrderByContext 和分页上下文 PaginationContext；而对于InsertSQLStatementContext 而言，InsertValueContext 则包含了所有与插入操作相关的值对象。</p>
<h4 data-nodeid="284120">3.自动生成主键</h4>
<p data-nodeid="284121">接下来的第三段代码与数据库主键相关，同样只有一句代码，如下所示：</p>
<pre class="lang-java" data-nodeid="284122"><code data-language="java"><span class="hljs-comment">//如果是 InsertStatement 则自动生成主键 </span>
Optional&lt;GeneratedKey&gt; generatedKey = sqlStatement <span class="hljs-keyword">instanceof</span> InsertStatement 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;? GeneratedKey.getGenerateKey(shardingRule, metaData.getTables(), parameters, (InsertStatement) sqlStatement) : Optional.&lt;GeneratedKey&gt;absent();
</code></pre>
<p data-nodeid="284123">这段代码的逻辑比较明确，即如果输入的 SQLStatement 是 InsertStatement，则自动创建一个主键 GeneratedKey，反之就不做处理。</p>
<p data-nodeid="284124">在数据分片的场景下，创建一个分布式主键实际上并没有那么简单，所以在这段代码背后有很多设计的思想和实现的技巧值得我们进行深入分析，关于这个主题，我们已经在 <a href="https://kaiwu.lagou.com/course/courseInfo.htm?sid=&amp;courseId=257&amp;lagoufrom=noapp" data-nodeid="284292">《14 | 分布式主键：ShardingSphere 中有哪些分布式主键实现方式？》</a>中对分布式主键生成机制做了专题分享。</p>
<h4 data-nodeid="284125">4.创建分片条件</h4>
<p data-nodeid="284126">我们来看 ShardingRouter 中 route 方法的第四个步骤，这个步骤的作用是创建分片条件，如下所示：</p>
<pre class="lang-java" data-nodeid="284127"><code data-language="java"><span class="hljs-comment">//创建分片条件 </span>
ShardingConditions shardingConditions = getShardingConditions(parameters, sqlStatementContext, generatedKey.orNull(), metaData.getRelationMetas()); 
<span class="hljs-keyword">boolean</span> needMergeShardingValues = isNeedMergeShardingValues(sqlStatementContext); 
<span class="hljs-keyword">if</span> (sqlStatementContext.getSqlStatement() <span class="hljs-keyword">instanceof</span> DMLStatement &amp;&amp; needMergeShardingValues) { 
&nbsp;&nbsp;&nbsp;&nbsp;checkSubqueryShardingValues(sqlStatementContext, shardingConditions); 
&nbsp;&nbsp;&nbsp;&nbsp;mergeShardingConditions(shardingConditions); 
}
</code></pre>
<p data-nodeid="284128">在 ShardingSphere 中，分片条件对象 ShardingCondition 定义如下所示，包含了一组路由信息和节点信息，其中路由信息包含表名和列名，而节点信息包含数据源名和表名：</p>
<pre class="lang-java" data-nodeid="284129"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingCondition</span> </span>{ 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//路由信息 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;RouteValue&gt; routeValues = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;(); 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//节点信息 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;DataNode&gt; dataNodes = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;(); 
}
</code></pre>
<p data-nodeid="284130">那么如何获取分片条件呢？如下所示的 getShardingConditions 方法给出了具体的实现方式，可以看到这里根据输入的 SQL 类型，分别通过 InsertClauseShardingConditionEngine 和WhereClauseShardingConditionEngine 创建了 ShardingConditions：</p>
<pre class="lang-java" data-nodeid="284131"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> ShardingConditions <span class="hljs-title">getShardingConditions</span><span class="hljs-params">(<span class="hljs-keyword">final</span> List&lt;Object&gt; parameters, <span class="hljs-keyword">final</span> SQLStatementContext sqlStatementContext, <span class="hljs-keyword">final</span> GeneratedKey generatedKey, <span class="hljs-keyword">final</span> RelationMetas relationMetas)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span> (sqlStatementContext.getSqlStatement() <span class="hljs-keyword">instanceof</span> DMLStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//如果是 InsertSQLStatement 上下文 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatementContext <span class="hljs-keyword">instanceof</span> InsertSQLStatementContext) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; InsertSQLStatementContext shardingInsertStatement = (InsertSQLStatementContext) sqlStatementContext; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过 InsertClauseShardingConditionEngine 创建分片条件 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingConditions(<span class="hljs-keyword">new</span> InsertClauseShardingConditionEngine(shardingRule).createShardingConditions(shardingInsertStatement, generatedKey, parameters)); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//否则直接通过 WhereClauseShardingConditionEngine 创建分片条件 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingConditions(<span class="hljs-keyword">new</span> WhereClauseShardingConditionEngine(shardingRule, relationMetas).createShardingConditions(sqlStatementContext.getSqlStatement(), parameters)); 
&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingConditions(Collections.&lt;ShardingCondition&gt;emptyList()); 
}
</code></pre>
<p data-nodeid="284132">对于路由引擎而言，分片条件的主要目的就是提取用于路由的目标数据库、表和列之间的关系，InsertClauseShardingConditionEngine 和 WhereClauseShardingConditionEngine 中的处理逻辑都是为了构建包含这些关系信息的一组 ShardingCondition 对象。</p>
<p data-nodeid="284133">当获取这些 ShardingCondition 之后，我们还看到有一个优化的步骤，即调用mergeShardingConditions，对可以合并的 ShardingCondition 进行合并。</p>
<h4 data-nodeid="284134">5.执行路由</h4>
<p data-nodeid="284135">当我们获取了 SQLStatement 上下文，并创建了分片条件，接下来就是真正执行路由，如下所示：</p>
<pre class="lang-java" data-nodeid="284136"><code data-language="java"><span class="hljs-comment">//获取 RoutingEngine 并执行路由 </span>
RoutingEngine routingEngine = RoutingEngineFactory.newInstance(shardingRule, metaData, sqlStatementContext, shardingConditions); 
RoutingResult routingResult = routingEngine.route();
</code></pre>
<p data-nodeid="284137">这两句代码是 ShardingRouter 类的核心，我们获取了一个 RoutingEngine 实例，然后基于该实例执行路由并返回一个 RoutingResult 对象。RoutingEngine 定义如下，只有一个简单的 route 方法：</p>
<pre class="lang-java" data-nodeid="284138"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">RoutingEngine</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行路由 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">RoutingResult <span class="hljs-title">route</span><span class="hljs-params">()</span></span>; 
}
</code></pre>
<p data-nodeid="284139">在 ShardingSphere 中存在一批 RoutingEngine 的实现类，RoutingEngineFactory 工厂类负责生成这些具体的 RoutingEngine，生成逻辑如下所示：</p>
<pre class="lang-java" data-nodeid="284140"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> RoutingEngine <span class="hljs-title">newInstance</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> ShardingSphereMetaData metaData, <span class="hljs-keyword">final</span> SQLStatementContext sqlStatementContext, <span class="hljs-keyword">final</span> ShardingConditions shardingConditions)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SQLStatement sqlStatement = sqlStatementContext.getSqlStatement(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt;String&gt; tableNames = sqlStatementContext.getTablesContext().getTableNames(); 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;<span class="hljs-comment">//全库路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> TCLStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DatabaseBroadcastRoutingEngine(shardingRule); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//全库表路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> DDLStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TableBroadcastRoutingEngine(shardingRule, metaData.getTables(), sqlStatementContext); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//阻断路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> DALStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> getDALRoutingEngine(shardingRule, sqlStatement, tableNames); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//全实例路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatement <span class="hljs-keyword">instanceof</span> DCLStatement) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> getDCLRoutingEngine(shardingRule, sqlStatementContext, metaData); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//默认库路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (shardingRule.isAllInDefaultDataSource(tableNames)) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultDatabaseRoutingEngine(shardingRule, tableNames); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//全库路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (shardingRule.isAllBroadcastTables(tableNames)) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> sqlStatement <span class="hljs-keyword">instanceof</span> SelectStatement ? <span class="hljs-keyword">new</span> UnicastRoutingEngine(shardingRule, tableNames) : <span class="hljs-keyword">new</span> DatabaseBroadcastRoutingEngine(shardingRule); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//默认库路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatementContext.getSqlStatement() <span class="hljs-keyword">instanceof</span> DMLStatement &amp;&amp; tableNames.isEmpty() &amp;&amp; shardingRule.hasDefaultDataSourceName()) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultDatabaseRoutingEngine(shardingRule, tableNames); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//单播路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (sqlStatementContext.getSqlStatement() <span class="hljs-keyword">instanceof</span> DMLStatement &amp;&amp; shardingConditions.isAlwaysFalse() || tableNames.isEmpty() || !shardingRule.tableRuleExists(tableNames)) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UnicastRoutingEngine(shardingRule, tableNames); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//分片路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> getShardingRoutingEngine(shardingRule, sqlStatementContext, shardingConditions, tableNames); 
}
</code></pre>
<p data-nodeid="284141">这些 RoutingEngine 的具体介绍我们放在下一课时《18 | 路由引擎：如何实现数据访问的分片路由和广播路由？》中进行详细介绍，这里只需要了解 ShardingSphere 在包结构的设计上把具体的 RoutingEngine 分成了六大类：即广播（broadcast）路由、混合（complex）路由、默认数据库（defaultdb）路由、无效（ignore）路由、标准（standard）路由以及单播（unicast）路由，如下所示：</p>
<p data-nodeid="284142"><img src="https://s0.lgstatic.com/i/image/M00/3F/C4/Ciqc1F8xJvuALcqiAAA5dODyQeU720.png" alt="Drawing 3.png" data-nodeid="284309"></p>
<p data-nodeid="284143">不同类型的 RoutingEngine 实现类</p>
<p data-nodeid="284144">RoutingEngine 的执行结果是 RoutingResult，而 RoutingResult 中包含了一个 RoutingUnit集合，RoutingUnit 中的变量定义如下所示，可以看到有两个关于 DataSource 名称的变量以及一个 TableUnit 列表：</p>
<pre class="lang-java" data-nodeid="284145"><code data-language="java"><span class="hljs-comment">//真实数据源名 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String dataSourceName; 
<span class="hljs-comment">//逻辑数据源名 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String masterSlaveLogicDataSourceName; 
<span class="hljs-comment">//表单元列表 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;TableUnit&gt; tableUnits = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
</code></pre>
<p data-nodeid="284146">而 TableUnit 保存着逻辑表名和实际表名，如下所示：</p>
<pre class="lang-java" data-nodeid="284147"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TableUnit</span> </span>{ 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//逻辑表名 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String logicTableName; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//真实表名 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String actualTableName; 
}
</code></pre>
<p data-nodeid="284148">所以 RoutingResult 中保存的实际上就是一组关于数据库与数据表的对应关系，其中库与表都存在逻辑值和真实值。</p>
<h4 data-nodeid="284149">6.构建路由结果</h4>
<p data-nodeid="284150">当通过一系列的路由引擎处理之后，我们获得了 RoutingResult 对象，但并不是直接将其进行返回，而是会构建一个 SQLRouteResult 对象。这就是 ShardingRouter 的 route 方法最后一个步骤，如下所示：</p>
<pre class="lang-java" data-nodeid="284151"><code data-language="java"><span class="hljs-comment">//构建 SQLRouteResult </span>
SQLRouteResult result = <span class="hljs-keyword">new</span> SQLRouteResult(sqlStatementContext, shardingConditions, generatedKey.orNull()); 
result.setRoutingResult(routingResult); 
<span class="hljs-comment">//如果是Insert语句，则设置自动生成的分片键 </span>
<span class="hljs-keyword">if</span> (sqlStatementContext <span class="hljs-keyword">instanceof</span> InsertSQLStatementContext) { 
&nbsp;&nbsp;&nbsp;&nbsp;setGeneratedValues(result); 
} 
<span class="hljs-keyword">return</span> result;
</code></pre>
<p data-nodeid="284152">我们来到 SQLRouteResult 的定义，看看它与 RouteResult 之间有什么不同，SQLRouteResult中 的变量如下所示：</p>
<pre class="lang-java" data-nodeid="284153"><code data-language="java"><span class="hljs-comment">//SQLStatement 上下文 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SQLStatementContext sqlStatementContext; 
<span class="hljs-comment">//分片条件 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingConditions shardingConditions; 
<span class="hljs-comment">//自动生成的分片键 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> GeneratedKey generatedKey; 
<span class="hljs-comment">//一组路由单元 </span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;RouteUnit&gt; routeUnits = <span class="hljs-keyword">new</span> LinkedHashSet&lt;&gt;(); 
<span class="hljs-comment">//由 RoutingEngine 生成的 RoutingResult </span>
<span class="hljs-keyword">private</span> RoutingResult routingResult;
</code></pre>
<p data-nodeid="284154">可以看到 SQLRouteResult 中包含了 RoutingResult。我们可以认为 SQLRouteResult 是整个 SQL 路由返回的路由结果，在后续的流程中还会被 PreparedStatementRoutingEngine 等上层对象所使用，而 RoutingResult 只是 RoutingEngine 返回的路由结果，它的使用者就是位于底层的 ShardingRouter。</p>
<p data-nodeid="284155">同时，我们注意到这里有一个新的 Unit 对象 RouteUnit，包含了数据源名称以及 SQL 单元对象 SQLUnit，如下所示：</p>
<pre class="lang-java" data-nodeid="284156"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RouteUnit</span> </span>{ 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//数据源名 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String dataSourceName; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//SQL 单元 </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SQLUnit sqlUnit; 
}
</code></pre>
<p data-nodeid="284157">这里的 SQLUnit 中就是最终的一条 SQL 语句以及相应参数的组合。因为路由结果对象 SQLRouteResult 会继续传递到分片引擎的后续流程，且内部结构比较复杂，所以这里通过如下所示的类图对其包含的各种变量进行总结，方便你进行理解。</p>
<p data-nodeid="284158"><img src="https://s0.lgstatic.com/i/image/M00/3F/D0/CgqCHl8xJ0eAMp1GAABywd2SYFQ497.png" alt="Drawing 4.png" data-nodeid="284322"></p>
<p data-nodeid="284159">至此，我们把 ShardingRouter 类的核心流程做了介绍。在 ShardingSphere 的路由引擎中，ShardingRouter 可以说是一个承上启下的核心类，向下我们可以挖掘各种 RoutingEngine 的具体实现；向上我们可以延展到读写分离等面向应用的具体场景。</p>
<p data-nodeid="284160">下图展示了 ShardingRouter 的这种定位关系。关于各种 RoutingEngine 的介绍是我们下一课时的内容，今天我们先将基于 ShardingRouter 讨论它的上层结构，从而引出了 ShardingEngine。</p>
<p data-nodeid="284161"><img src="https://s0.lgstatic.com/i/image/M00/3F/D0/CgqCHl8xJ1WAbAmHAAB_-h8F66g956.png" alt="Drawing 6.png" data-nodeid="284327"></p>
<h3 data-nodeid="284162">从底层 ShardingRouter 到上层 ShardingEngine</h3>
<p data-nodeid="284163">我们的思路仍然是从下往上，先来看上图中的 StatementRoutingEngine，其实现如下所示：</p>
<pre class="lang-java" data-nodeid="284164"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StatementRoutingEngine</span> </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingRouter shardingRouter; 

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingMasterSlaveRouter masterSlaveRouter; 

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">StatementRoutingEngine</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> ShardingSphereMetaData metaData, <span class="hljs-keyword">final</span> SQLParseEngine sqlParseEngine)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingRouter = <span class="hljs-keyword">new</span> ShardingRouter(shardingRule, metaData, sqlParseEngine); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; masterSlaveRouter = <span class="hljs-keyword">new</span> ShardingMasterSlaveRouter(shardingRule.getMasterSlaveRules()); 
&nbsp;&nbsp;&nbsp; } 

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> SQLRouteResult <span class="hljs-title">route</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String logicSQL)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SQLStatement sqlStatement = shardingRouter.parse(logicSQL, <span class="hljs-keyword">false</span>); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> masterSlaveRouter.route(shardingRouter.route(logicSQL, Collections.emptyList(), sqlStatement)); 
&nbsp;&nbsp;&nbsp; } 
}
</code></pre>
<p data-nodeid="284165">可以看到在 StatementRoutingEngine 的 route 方法中，通过 ShardingMasterSlaveRouter 对通过 ShardingRouter 所生成的 SQLRouteResult 进行了再一次路由，也就是说在分片路由的基础上添加了主从路由，关于读写分离和主从路由我们会在之后的《26 | 读写分离：普通主从架构和分片主从架构分别是如何实现的？》进行讨论。</p>
<p data-nodeid="284166">现在我们来到 sharding-core-entry 工程，看看更上层的处理流程。整个 sharding-core-entry 工程只有三个类，即作为基类的 BaseShardingEngine 以及两个子类 PreparedQueryShardingEngine 和 SimpleQueryShardingEngine。我们先来看 BaseShardingEngine 类，它本质上是一个模板类，BaseShardingEngine 的 shard 方法如下所示：</p>
<pre class="lang-java" data-nodeid="284167"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> SQLRouteResult <span class="hljs-title">shard</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters)</span> </span>{ 
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//调用模板方法准备参数 </span>
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;Object&gt; clonedParameters = cloneParameters(parameters); 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//执行路由 </span>
&nbsp;&nbsp;&nbsp;&nbsp;SQLRouteResult result = executeRoute(sql, clonedParameters); 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//执行 SQL 转换（Convert）和改写（Rewrite） </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.getRouteUnits().addAll(HintManager.isDatabaseShardingOnly() ? convert(sql, clonedParameters, result) : rewriteAndConvert(sql, clonedParameters, result)); 

	<span class="hljs-comment">//省略日志记录 </span>

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> result; 
}
</code></pre>
<p data-nodeid="284168">在这里我们看到了 SQL 转换（Convert）和改写（Rewrite）的入口，这是路由引擎之外的执行流程，我们今天不做展开。上述代码与路由相关最核心的就是 executeRoute 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="284169"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> SQLRouteResult <span class="hljs-title">executeRoute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> List&lt;Object&gt; clonedParameters)</span> </span>{
&nbsp;routingHook.start(sql); 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span> { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;<span class="hljs-comment">//调用模板方法执行路由并获取结果 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SQLRouteResult result = route(sql, clonedParameters); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;routingHook.finishSuccess(result, metaData.getTables()); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> result; 
&nbsp;&nbsp;&nbsp;&nbsp;} <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> Exception ex) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;routingHook.finishFailure(ex); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span> ex; 
&nbsp;&nbsp;&nbsp;&nbsp;} 
}
</code></pre>
<p data-nodeid="284170">这个方法的处理方式与 SQLParseEngine 的 parse 方法有着类似的代码结构，同样用到了 Hook 机制。</p>
<p data-nodeid="284171">从设计模式上讲，BaseShardingEngine 采用了非常典型的模板方法。当我们需要完成一个过程或一系列步骤时，这些过程或步骤在某一细节层次保持一致，但个别步骤在更详细的层次上的实现可能不同时，可以考虑用模板方法模式来处理。实现模板方法的过程也非常简单，其实就是利用了类的继承机制。作为一个模板类，我们注意到 BaseShardingEngine 提供了两个模板方法供子类进行实现，分别是：</p>
<pre class="lang-java" data-nodeid="284172"><code data-language="java"><span class="hljs-comment">//拷贝参数 </span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> List&lt;Object&gt; <span class="hljs-title">cloneParameters</span><span class="hljs-params">(List&lt;Object&gt; parameters)</span></span>; 
<span class="hljs-comment">//执行路由 </span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> SQLRouteResult <span class="hljs-title">route</span><span class="hljs-params">(String sql, List&lt;Object&gt; parameters)</span></span>;
</code></pre>
<p data-nodeid="284173">显然，对于 SimpleQueryShardingEngine 而言，不需要参数，所以 cloneParameters 直接返回空列表。而 route 方法则直接使用前面介绍的 StatementRoutingEngine 进行路由。SimpleQueryShardingEngine 类的完整实现如下所示：</p>
<pre class="lang-java" data-nodeid="284174"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SimpleQueryShardingEngine</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseShardingEngine</span> </span>{ 

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> StatementRoutingEngine routingEngine; 

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SimpleQueryShardingEngine</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> ShardingProperties shardingProperties, <span class="hljs-keyword">final</span> ShardingSphereMetaData metaData, <span class="hljs-keyword">final</span> SQLParseEngine sqlParseEngine)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(shardingRule, shardingProperties, metaData); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; routingEngine = <span class="hljs-keyword">new</span> StatementRoutingEngine(shardingRule, metaData, sqlParseEngine); 
&nbsp;&nbsp;&nbsp; } 

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> List&lt;Object&gt; <span class="hljs-title">cloneParameters</span><span class="hljs-params">(<span class="hljs-keyword">final</span> List&lt;Object&gt; parameters)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.emptyList(); 
&nbsp;&nbsp;&nbsp; } 

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> SQLRouteResult <span class="hljs-title">route</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> routingEngine.route(sql); 
&nbsp;&nbsp;&nbsp; } 
}
</code></pre>
<p data-nodeid="284175">至此，关于 ShardingSphere 路由引擎部分的内容基本都介绍完毕。对于上层结构而言，我们以 SimpleQueryShardingEngine 为例进行了展开，对于 PreparedQueryShardingEngine 的处理方式也是类似。作为总结，我们通过如下所示的时序图来梳理这些路由的主流程。</p>
<p data-nodeid="284176"><img src="https://s0.lgstatic.com/i/image/M00/3F/D0/CgqCHl8xJ2aAQabtAACUcSURKVc544.png" alt="Drawing 8.png" data-nodeid="284341"></p>
<h3 data-nodeid="284177">从源码解析到日常开发</h3>
<p data-nodeid="284178">分包设计原则可以用来设计和规划开源框架的代码结构。在今天的内容中，我们看到了 ShardingSphere 中非常典型的一种分层和分包实现策略。通过 sharding-core-route 和 sharding-core-entry 这两个工程，我们把路由引擎中位于底层的核心类 ShardingRouter 和位于上层的 PreparedQueryShardingEngine 及 SimpleQueryShardingEngine 类进行了合理的分层管理。ShardingSphere 对于分层和分包策略的应用有很多具体的表现形式，随着课程的不断演进，我们还会看到更多的应用场景。</p>
<h3 data-nodeid="284179">小结与预告</h3>
<p data-nodeid="284180">作为 ShardingSphere 分片引擎的第二个核心组件，路由引擎的目的在于生成 SQLRouteResult目标对象。而整个路由引擎中最核心的就是 ShardingRouter 类。今天，我们对 ShardingRouter 的整体执行流程进行了详细的讨论，同时也引出了路由引擎中的底层对象 RoutingEngine。</p>
<p data-nodeid="284181"><strong data-nodeid="284350">这里给你留一道思考题：ShardingSphere 中，一个完整的路由执行过程需要经历哪些步骤？</strong> 欢迎你在留言区与大家讨论，我将一一点评解答。</p>
<p data-nodeid="284182" class="">在今天的课程中，我们也提到了 ShardingSphere 中存在多种 RoutingEngine。在下一课时的内容中，我们将关注于这些 RoutingEngine 的具体实现过程。</p>

---

### 精选评论

##### **7834：
> 各版本间相差大在正文开头应该就说明的，看到现在也记了很多笔记了，再切到老师的版本也浪费时间

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，版本之间差别比较大，可以选择自己想要学习的版本来看

##### **杰：
> 打卡，整个路有过程很负责

##### **宾：
> 老师，你这源码是哪个版本的？与我看的对不上啊？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用的是4.0.1版本，如果你看的是正在开发的5.X版本分支的话，确实很多会对不起来，4.X版本的话基本都是一致的，5.X和4.X版本的代码变动非常大

