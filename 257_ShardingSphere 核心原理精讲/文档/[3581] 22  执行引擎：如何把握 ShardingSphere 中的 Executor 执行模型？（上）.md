<p data-nodeid="8887" class="">在上一课时中，我们对 ShardingGroupExecuteCallback 和 SQLExecuteTemplate 做了介绍。从设计上讲，前者充当 ShardingExecuteEngine 的回调入口；而后者则是一个模板类，完成对 ShardingExecuteEngine 的封装并提供了对外的统一入口，这些类都位于底层的 sharding-core-execute 工程中。</p>
<p data-nodeid="8888"><img src="https://s0.lgstatic.com/i/image/M00/47/4A/CgqCHl9HalOAccqPAACp0Ky_Tl8886.png" alt="image.png" data-nodeid="8998"></p>
<p data-nodeid="8889">从今天开始，我们将进入到 sharding-jdbc-core 工程，来看看 ShardingSphere 中执行引擎上层设计中的几个核心类。</p>
<h3 data-nodeid="8890">AbstractStatementExecutor</h3>
<p data-nodeid="8891">如上图所示，根据上一课时中的执行引擎整体结构图，可以看到<strong data-nodeid="9010">SQLExecuteTemplate</strong>的直接使用者是<strong data-nodeid="9011">AbstractStatementExecutor 类</strong>，今天我们就从这个类开始展开讨论，该类的变量比较多，我们先来看一下：</p>
<pre class="lang-java" data-nodeid="8892"><code data-language="java"><span class="hljs-comment">//数据库类型</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> DatabaseType databaseType;
<span class="hljs-comment">//JDBC中用于指定结果处理方式的 resultSetType</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetType;
<span class="hljs-comment">//JDBC中用于指定是否可对结果集进行修改的 resultSetConcurrency</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetConcurrency; 
<span class="hljs-comment">//JDBC中用于指定事务提交或回滚后结果集是否仍然可用的 resultSetConcurrency</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetHoldability;
<span class="hljs-comment">//分片 Connection</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingConnection connection;
<span class="hljs-comment">//用于数据准备的模板类</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SQLExecutePrepareTemplate sqlExecutePrepareTemplate;
<span class="hljs-comment">//SQL 执行模板类</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SQLExecuteTemplate sqlExecuteTemplate;
<span class="hljs-comment">//JDBC的Connection列表</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;Connection&gt; connections = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
<span class="hljs-comment">//SQLStatement 上下文</span>
<span class="hljs-keyword">private</span> SQLStatementContext sqlStatementContext;
<span class="hljs-comment">//参数集</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;List&lt;Object&gt;&gt; parameterSets = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;(); 
<span class="hljs-comment">//JDBC的Statement 列表</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;Statement&gt; statements = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;(); 
<span class="hljs-comment">//JDBC的ResultSet 列表</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;ResultSet&gt; resultSets = <span class="hljs-keyword">new</span> CopyOnWriteArrayList&lt;&gt;();
<span class="hljs-comment">//ShardingExecuteGroup 列表</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Collection&lt;ShardingExecuteGroup&lt;StatementExecuteUnit&gt;&gt; executeGroups = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
</code></pre>
<p data-nodeid="8893">从这个类开始，我们会慢慢接触 JDBC 规范相关的对象，因为 ShardingSphere 的设计目标是，重写一套与目前的 JDBC 规范完全兼容的体系。这里，我们看到的 Connection、Statement 和 ResultSet 等对象，以及 resultSetType、resultSetConcurrency、resultSetHoldability 等参数，都是属于 JDBC 规范中的内容，我们在注释上做了特别的说明，你对此也都比较熟悉。</p>
<p data-nodeid="8894">而像 ShardingSphere 自己封装的 ShardingConnection 对象也很重要，我们已经在《03 | 规范兼容：JDBC 规范与 ShardingSphere 是什么关系？》中对这个类的实现方式，以及如何兼容 JDBC 规范的详细过程做了介绍。</p>
<p data-nodeid="8895">在 AbstractStatementExecutor 中，这些变量的展开，会涉及很多 sharding-jdbc-core 代码工程，关于数据库访问相关的类的介绍，包括我们以前已经接触过的 ShardingStatement 和 ShardingPreparedStatement 等类，所以我们在展开 AbstractStatementExecutor 类的具体实现方法之前，需要对这些类有一定的了解。</p>
<p data-nodeid="8896">在 AbstractStatementExecutor 构造函数中，我们发现了上一课时中介绍的执行引擎 ShardingExecuteEngine 的创建过程，并通过它创建了 SQLExecuteTemplate 模板类，相关代码如下所示：</p>
<pre class="lang-java" data-nodeid="8897"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AbstractStatementExecutor</span><span class="hljs-params">(<span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetType, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetConcurrency, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetHoldability, <span class="hljs-keyword">final</span> ShardingConnection shardingConnection)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;ShardingExecuteEngine executeEngine = connection.getRuntimeContext().getExecuteEngine();
&nbsp;&nbsp;&nbsp;&nbsp;sqlExecuteTemplate = <span class="hljs-keyword">new</span> SQLExecuteTemplate(executeEngine, connection.isHoldTransaction());
}
</code></pre>
<p data-nodeid="8898">同时，AbstractStatementExecutor 中如下所示的 cacheStatements 方法也很有特色，该方法会根据持有的 ShardingExecuteGroup 类分别填充 statements 和 parameterSets 这两个对象，以供 AbstractStatementExecutor 的子类进行使用：<br>
<br></p>
<pre class="lang-java" data-nodeid="8899"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cacheStatements</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> (ShardingExecuteGroup&lt;StatementExecuteUnit&gt; each : executeGroups) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;statements.addAll(Lists.transform(each.getInputs(), <span class="hljs-keyword">new</span> Function&lt;StatementExecuteUnit, Statement&gt;() {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> Statement <span class="hljs-title">apply</span><span class="hljs-params">(<span class="hljs-keyword">final</span> StatementExecuteUnit input)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> input.getStatement();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameterSets.addAll(Lists.transform(each.getInputs(), <span class="hljs-keyword">new</span> Function&lt;StatementExecuteUnit, List&lt;Object&gt;&gt;() {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Object&gt; <span class="hljs-title">apply</span><span class="hljs-params">(<span class="hljs-keyword">final</span> StatementExecuteUnit input)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> input.getRouteUnit().getSqlUnit().getParameters();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}));
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="8900">注意：这里在实现方式上使用了 Google 提供的 Guava 框架中的 Lists.transform 方法，从而完成了不同对象之间的转换过程，这种实现方式在 ShardingSphere 中应用广泛，非常值得你学习。</p>
<p data-nodeid="8901">然后我们来看 AbstractStatementExecutor 中最核心的方法，即执行回调的 executeCallback 方法：</p>
<pre class="lang-java" data-nodeid="8902"><code data-language="java"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> &lt;T&gt; <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">executeCallback</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SQLExecuteCallback&lt;T&gt; executeCallback)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;T&gt; result = sqlExecuteTemplate.executeGroup((Collection) executeGroups, executeCallback);
&nbsp;&nbsp;&nbsp;&nbsp;refreshMetaDataIfNeeded(connection.getRuntimeContext(), sqlStatementContext);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="8903">显然，在这里应该使用 SQLExecuteTemplate 模板类来完成具体回调的执行过程。同时，我可以看到这里还有一个 refreshMetaDataIfNeeded 辅助方法用来刷选元数据。</p>
<p data-nodeid="8904">AbstractStatementExecutor 有两个实现类：一个是普通的 StatementExecutor，一个是 PreparedStatementExecutor，接下来我将分别进行讲解。</p>
<p data-nodeid="8905"><img src="https://s0.lgstatic.com/i/image/M00/47/3F/Ciqc1F9HamWACCzmAABPdP2Sna8714.png" alt="image (1).png" data-nodeid="9027"></p>
<h3 data-nodeid="8906">StatementExecutor</h3>
<p data-nodeid="8907">我们来到 StatementExecutor，先看它的用于执行初始化操作的 init 方法：</p>
<pre class="lang-java" data-nodeid="8908"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SQLRouteResult routeResult)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;setSqlStatementContext(routeResult.getSqlStatementContext());
&nbsp;&nbsp;&nbsp;&nbsp;getExecuteGroups().addAll(obtainExecuteGroups(routeResult.getRouteUnits()));
&nbsp;&nbsp;&nbsp;&nbsp;cacheStatements();
}
</code></pre>
<p data-nodeid="8909">这里的 cacheStatements 方法前面已经介绍过，而 obtainExecuteGroups 方法用于获取所需的 ShardingExecuteGroup 集合。要实现这个方法，就需要引入 SQLExecutePrepareTemplate 和对应的回调 SQLExecutePrepareCallback。</p>
<h4 data-nodeid="8910">1.SQLExecutePrepareCallback</h4>
<p data-nodeid="8911">从命名上看，让人感觉 SQLExecutePrepareTemplate 和 SQLExecuteTemplate 应该是一对，尤其是名称中有一个“Prepare”，让人联想到 PreparedStatement。</p>
<p data-nodeid="8912"><strong data-nodeid="9041">但事实上，SQLExecutePrepareTemplate 与 SQLExecuteTemplate 没有什么关联</strong>，它也不是像 SQLExecuteTemplate 一样提供了 ShardingExecuteEngine 的封装，而是主要关注于 ShardingExecuteGroup 数据的收集和拼装，换句话说是<strong data-nodeid="9042">为了准备（Prepare）数据</strong>。</p>
<p data-nodeid="8913">在 SQLExecutePrepareTemplate 中，核心的功能就是下面这个方法，该方法传入了一个 SQLExecutePrepareCallback 对象，并返回 ShardingExecuteGroup 的一个集合：</p>
<pre class="lang-java" data-nodeid="8914"><code data-language="java"><span class="hljs-keyword">public</span> Collection&lt;ShardingExecuteGroup&lt;StatementExecuteUnit&gt;&gt; getExecuteUnitGroups(<span class="hljs-keyword">final</span> Collection&lt;RouteUnit&gt; routeUnits, <span class="hljs-keyword">final</span> SQLExecutePrepareCallback callback) <span class="hljs-keyword">throws</span> SQLException {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> getSynchronizedExecuteUnitGroups(routeUnits, callback);
}
</code></pre>
<p data-nodeid="8915">为了构建这个集合，SQLExecutePrepareTemplate 实现了很多辅助方法，同时它还引入了一个 SQLExecutePrepareCallback 回调，来完成 ShardingExecuteGroup 数据结构中部分数据的填充。SQLExecutePrepareCallback 接口定义如下，可以看到 Connection 和 StatementExecuteUnit 这两个对象是通过回调来创建的：</p>
<pre class="lang-java" data-nodeid="8916"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SQLExecutePrepareCallback</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 Connection 列表</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">List&lt;Connection&gt; <span class="hljs-title">getConnections</span><span class="hljs-params">(ConnectionMode connectionMode, String dataSourceName, <span class="hljs-keyword">int</span> connectionSize)</span> <span class="hljs-keyword">throws</span> SQLException</span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 Statement 执行单元</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">StatementExecuteUnit <span class="hljs-title">createStatementExecuteUnit</span><span class="hljs-params">(Connection connection, RouteUnit routeUnit, ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
}
</code></pre>
<p data-nodeid="8917">当我们获取了想要的 ShardingExecuteGroup 之后，相当于完成了 StatementExecutor 的初始化工作。该类中剩下的就是一系列以“execute”开头的 SQL 执行方法，包括 executeQuery、executeUpdate，以及它们的各种重载方法。我们先来看用于查询的 executeQuery 方法：</p>
<pre class="lang-java" data-nodeid="8918"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;QueryResult&gt; <span class="hljs-title">executeQuery</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> isExceptionThrown = ExecutorExceptionHandler.isExceptionThrown();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建 SQLExecuteCallback 并执行查询</span>
&nbsp;&nbsp;&nbsp;&nbsp;SQLExecuteCallback&lt;QueryResult&gt; executeCallback = <span class="hljs-keyword">new</span> SQLExecuteCallback&lt;QueryResult&gt;(getDatabaseType(), isExceptionThrown) {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">protected</span> QueryResult <span class="hljs-title">executeSQL</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> getQueryResult(sql, statement, connectionMode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//执行 SQLExecuteCallback 并返回结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> executeCallback(executeCallback);
}
</code></pre>
<p data-nodeid="8919">我们已经在上一课时中介绍过这个方法，我们知道 SQLExecuteCallback 实现了 ShardingGroupExecuteCallback 接口并提供了 executeSQL 模板方法。而在上述 executeQuery 方法中，executeSQL 模板方法的实现过程，就是调用如下所示的 getQueryResult 方法：</p>
<pre class="lang-java" data-nodeid="8920"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> QueryResult <span class="hljs-title">getQueryResult</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//通过 Statement 执行 SQL 并获取结果</span>
&nbsp;ResultSet resultSet = statement.executeQuery(sql);
&nbsp;&nbsp;&nbsp;&nbsp;getResultSets().add(resultSet);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//根据连接模式来确认构建结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> ConnectionMode.MEMORY_STRICTLY == connectionMode ? <span class="hljs-keyword">new</span> StreamQueryResult(resultSet) : <span class="hljs-keyword">new</span> MemoryQueryResult(resultSet);
}
</code></pre>
<h4 data-nodeid="8921">2.ConnectionMode</h4>
<p data-nodeid="8922">getQueryResult 方法中完全基于 JDBC 中的 Statement 和 ResultSet 对象来执行查询并返回结果。</p>
<p data-nodeid="8923">但是，这里也引入了 ShardingSphere 执行引擎中非常重要的一个概念，即<strong data-nodeid="9054">ConnectionMode（连接模式）</strong>，它是一个枚举：</p>
<pre class="lang-java" data-nodeid="8924"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">enum</span> ConnectionMode {
&nbsp;&nbsp;&nbsp; MEMORY_STRICTLY, CONNECTION_STRICTLY
}
</code></pre>
<p data-nodeid="8925">可以看到有两种具体的连接模式：MEMORY_STRICTLY 和 CONNECTION_STRICTLY。</p>
<ul data-nodeid="8926">
<li data-nodeid="8927">
<p data-nodeid="8928">MEMORY_STRICTLY 代表<strong data-nodeid="9067">内存限制模式</strong>，</p>
</li>
<li data-nodeid="8929">
<p data-nodeid="8930">CONNECTION_STRICTLY 代表<strong data-nodeid="9075">连接限制模式</strong>。</p>
</li>
</ul>
<p data-nodeid="8931"><strong data-nodeid="9080">ConnectionMode（连接模式）</strong> 是 ShardingSphere 所提出的一个特有概念，背后体现的是一种设计上的平衡思想。从数据库访问资源的角度来看，一方面是对数据库连接资源的控制保护，另一方面是采用更优的归并模式达到对中间件内存资源的节省，如何处理好两者之间的关系，是 ShardingSphere 执行引擎需求解决的问题。</p>
<p data-nodeid="8932">为此，ShardingSphere 提出了连接模式的概念，简单举例说明：</p>
<ul data-nodeid="8933">
<li data-nodeid="8934">
<p data-nodeid="8935">当采用<strong data-nodeid="9091">内存限制模式</strong>时，对于同一数据源，如果有 10 张分表，那么执行时会获取 10 个连接并进行<strong data-nodeid="9092">并行执行</strong>；</p>
</li>
<li data-nodeid="8936">
<p data-nodeid="8937">而当采用<strong data-nodeid="9102">连接限制模式</strong>时，执行过程中只会获取 1 个连接而进行<strong data-nodeid="9103">串行执行</strong>。</p>
</li>
</ul>
<p data-nodeid="8938"><strong data-nodeid="9107">那么这个 ConnectionMode 是怎么得出来的呢？</strong></p>
<p data-nodeid="8939">实际上这部分代码位于 SQLExecutePrepareTemplate 中，我们根据 maxConnectionsSizePerQuery 这个配置项，以及与每个数据库所需要执行的 SQL 数量进行比较，然后得出具体的 ConnectionMode：</p>
<pre class="lang-java" data-nodeid="8940"><code data-language="java">ConnectionMode connectionMode = maxConnectionsSizePerQuery &lt; sqlUnits.size() ? ConnectionMode.CONNECTION_STRICTLY : ConnectionMode.MEMORY_STRICTLY;
</code></pre>
<p data-nodeid="8941">关于这个判断条件，我们可以使用一张简单的示意图来进行说明，如下所示：</p>
<p data-nodeid="9854" class=""><img src="https://s0.lgstatic.com/i/image/M00/47/3F/Ciqc1F9HaoaAYskMAACJIb5G6C8859.png" alt="image (2).png" data-nodeid="9857"></p>


<p data-nodeid="8944">如上图所示，我们可以看到如果每个数据库连接所指向的 SQL 数多于一条时，走的是内存限制模式，反之走的是连接限制模式。</p>
<h4 data-nodeid="8945">3.StreamQueryResult VS MemoryQueryResult</h4>
<p data-nodeid="8946">在了解了 ConnectionMode（连接模式） 的设计理念后，我们再来看 StatementExecutor 的 executeQuery 方法返回的是一个 QueryResult。</p>
<p data-nodeid="8947">在 ShardingSphere 中，<strong data-nodeid="9121">QueryResult 是一个代表查询结果的接口</strong>，可以看到该接口封装了很多面向底层数据获取的方法：</p>
<pre class="lang-java" data-nodeid="8948"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">QueryResult</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">next</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">getValue</span><span class="hljs-params">(<span class="hljs-keyword">int</span> columnIndex, Class&lt;?&gt; type)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">getCalendarValue</span><span class="hljs-params">(<span class="hljs-keyword">int</span> columnIndex, Class&lt;?&gt; type, Calendar calendar)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">InputStream <span class="hljs-title">getInputStream</span><span class="hljs-params">(<span class="hljs-keyword">int</span> columnIndex, String type)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">wasNull</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">getColumnCount</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getColumnLabel</span><span class="hljs-params">(<span class="hljs-keyword">int</span> columnIndex)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isCaseSensitive</span><span class="hljs-params">(<span class="hljs-keyword">int</span> columnIndex)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
}
</code></pre>
<p data-nodeid="8949">在 ShardingSphere中，<strong data-nodeid="9127">QueryResult 接口存在于 StreamQueryResult（代表流式归并结果）和 MemoryQueryResult （代表内存归并结果）这两个实现类</strong>。</p>
<blockquote data-nodeid="8950">
<p data-nodeid="8951">ShardingSphere 采用这样的设计实际上跟前面介绍的 ConnectionMode 有直接关系。</p>
</blockquote>
<ul data-nodeid="8952">
<li data-nodeid="8953">
<p data-nodeid="8954">我们知道，在<strong data-nodeid="9138">内存限制</strong>模式中，ShardingSphere 对一次操作所耗费的数据库连接数量<strong data-nodeid="9139">不做限制</strong>；</p>
</li>
<li data-nodeid="8955">
<p data-nodeid="8956">而当采用<strong data-nodeid="9149">连接限制</strong>模式时，ShardingSphere<strong data-nodeid="9150">严格控制</strong>对一次操作所耗费的数据库连接数量。</p>
</li>
</ul>
<p data-nodeid="8957">基于这样的设计原理，如上面的 ConnectionMode 的计算示意图所示：在 maxConnectionSizePerQuery 允许的范围内，当一个连接需要执行的请求数量大于 1 时，意味着当前的数据库连接无法持有相应的数据结果集，则必须采用<strong data-nodeid="9160">内存归并</strong>；反之，则可以采用<strong data-nodeid="9161">流式归并</strong>。</p>
<ul data-nodeid="8958">
<li data-nodeid="8959">
<p data-nodeid="8960"><strong data-nodeid="9165">StreamQueryResult</strong></p>
</li>
</ul>
<p data-nodeid="8961">我们通过对比 StreamQueryResult 和 MemoryQueryResult 的实现过程，对上述原理做进一步分析，在 StreamQueryResult 中，它的 next 方法非常简单：</p>
<pre class="lang-java" data-nodeid="8962"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">next</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> resultSet.next();
}
</code></pre>
<p data-nodeid="8963">显然这是一种<strong data-nodeid="9172">流式处理</strong>的方式，从 ResultSet 中获取下一个数据行。</p>
<ul data-nodeid="8964">
<li data-nodeid="8965">
<p data-nodeid="8966"><strong data-nodeid="9176">MemoryQueryResult</strong></p>
</li>
</ul>
<p data-nodeid="8967">我们再来看 MemoryQueryResult，在它的构造函数中，通过 getRows 方法把 ResultSet 中的全部数据行，先进行获取并存储在内存变量 rows 中：</p>
<pre class="lang-java" data-nodeid="8968"><code data-language="java"><span class="hljs-keyword">private</span> Iterator&lt;List&lt;Object&gt;&gt; getRows(<span class="hljs-keyword">final</span> ResultSet resultSet) <span class="hljs-keyword">throws</span> SQLException {
&nbsp;&nbsp;&nbsp; Collection&lt;List&lt;Object&gt;&gt; result = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (resultSet.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Object&gt; rowData = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(resultSet.getMetaData().getColumnCount());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> columnIndex = <span class="hljs-number">1</span>; columnIndex &lt;= resultSet.getMetaData().getColumnCount(); columnIndex++) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取每一个 Row 的数据</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object rowValue = getRowValue(resultSet, columnIndex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//存放在内存中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rowData.add(resultSet.wasNull() ? <span class="hljs-keyword">null</span> : rowValue);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.add(rowData);
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result.iterator();
}
</code></pre>
<p data-nodeid="8969">基于以上方法，MemoryQueryResult 的 next 方法应该是，从这个 rows 变量中获取下一个数据行，如下所示：</p>
<pre class="lang-java" data-nodeid="8970"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">next</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (rows.hasNext()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; currentRow = rows.next();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; currentRow = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
</code></pre>
<p data-nodeid="8971"><strong data-nodeid="9182">通过这种方式，我们就将传统的流式处理方式转变成了内存处理方式。</strong></p>
<p data-nodeid="8972">关于 ConnectionMode 和两种 QueryResult 的讨论就到这里，让我们回到 StatementExecutor。理解了 StatementExecutor 的 executeQuery 方法之后，我们再来看它更为通用的 execute 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="8973"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">execute</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
     <span class="hljs-keyword">return</span> execute(<span class="hljs-keyword">new</span> Executor() {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">execute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> statement.execute(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
}
</code></pre>
<p data-nodeid="8974">注意到上述 execute 方法并没有使用 SQLExecuteCallback 回调，而是使用了一个 Executor 接口，该接口定义如下：</p>
<pre class="lang-java" data-nodeid="8975"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Executor</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行 SQL</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">execute</span><span class="hljs-params">(Statement statement, String sql)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
}
</code></pre>
<p data-nodeid="8976">然后我们再继续往下看，发现在改方法实际的执行过程中，还是用到了 SQLExecuteCallback 回调：</p>
<pre class="lang-java" data-nodeid="8977"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">execute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Executor executor)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> isExceptionThrown = ExecutorExceptionHandler.isExceptionThrown();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建 SQLExecuteCallback 并执行</span>
&nbsp;&nbsp;&nbsp;&nbsp;SQLExecuteCallback&lt;Boolean&gt; executeCallback = <span class="hljs-keyword">new</span> SQLExecuteCallback&lt;Boolean&gt;(getDatabaseType(), isExceptionThrown) {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> Boolean <span class="hljs-title">executeSQL</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//使用 Executor 进行执行</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> executor.execute(statement, sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; };
&nbsp;&nbsp;&nbsp; List&lt;Boolean&gt; result = executeCallback(executeCallback);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">null</span> == result || result.isEmpty() || <span class="hljs-keyword">null</span> == result.get(<span class="hljs-number">0</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result.get(<span class="hljs-number">0</span>);
}
</code></pre>
<p data-nodeid="8978">这里多嵌套一层的目的是，更好地分离代码的职责，并对执行结果进行处理，同样的处理技巧在 StatementExecutor 的 executeUpdate 方法中也有体现。</p>
<h3 data-nodeid="8979">PreparedStatementExecutor</h3>
<p data-nodeid="8980">讲完 StatementExecutor 之后，我们来看 PreparedStatementExecutor。PreparedStatementExecutor 包含了与 StatementExecutor 一样的用于初始化的 init 方法。然后，我们同样来看它如下所示的 executeQuery 方法，可以看到这里的处理方式与在 StatementExecutor 的一致：</p>
<pre class="lang-java" data-nodeid="8981"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;QueryResult&gt; <span class="hljs-title">executeQuery</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> isExceptionThrown = ExecutorExceptionHandler.isExceptionThrown();
&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建 SQLExecuteCallback 并执行</span>
&nbsp;&nbsp;&nbsp;&nbsp;SQLExecuteCallback&lt;QueryResult&gt; executeCallback = <span class="hljs-keyword">new</span> SQLExecuteCallback&lt;QueryResult&gt;(getDatabaseType(), isExceptionThrown) {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">protected</span> QueryResult <span class="hljs-title">executeSQL</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> getQueryResult(statement, connectionMode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> executeCallback(executeCallback);
}
</code></pre>
<p data-nodeid="8982">然后，我们再来看它的 execute 方法，就会发现有不同点：</p>
<pre class="lang-java" data-nodeid="8983"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">execute</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">boolean</span> isExceptionThrown = ExecutorExceptionHandler.isExceptionThrown();
&nbsp;&nbsp;&nbsp;&nbsp;SQLExecuteCallback&lt;Boolean&gt; executeCallback = SQLExecuteCallbackFactory.getPreparedSQLExecuteCallback(getDatabaseType(), isExceptionThrown);
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;Boolean&gt; result = executeCallback(executeCallback);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span> (<span class="hljs-keyword">null</span> == result || result.isEmpty() || <span class="hljs-keyword">null</span> == result.get(<span class="hljs-number">0</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> result.get(<span class="hljs-number">0</span>);
}
</code></pre>
<p data-nodeid="8984">与 StatementExecutor 不同，PreparedStatementExecutor 在实现 execute 方法时没有设计类似 Executor 这样的接口，而是直接提供了一个工厂类 SQLExecuteCallbackFactory：</p>
<pre class="lang-java" data-nodeid="8985"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SQLExecuteCallbackFactory</span> </span>{
	…
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> SQLExecuteCallback&lt;Boolean&gt; <span class="hljs-title">getPreparedSQLExecuteCallback</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> isExceptionThrown)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SQLExecuteCallback&lt;Boolean&gt;(databaseType, isExceptionThrown) {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> Boolean <span class="hljs-title">executeSQL</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> Statement statement, <span class="hljs-keyword">final</span> ConnectionMode connectionMode)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ((PreparedStatement) statement).execute();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };
	}
}
</code></pre>
<p data-nodeid="8986">注意到这里的静态方法 getPreparedSQLExecuteCallback 也就是返回了一个 SQLExecuteCallback 回调的实现，而在这个实现中使用了 JDBC 底层的 PreparedStatement 完成具体 SQL 的执行过程。</p>
<p data-nodeid="8987">至此，我们对 ShardingSphere 中两个主要执行器 StatementExecutor 和 PreparedStatementExecutor 都进行了详细介绍。</p>
<h3 data-nodeid="8988">从源码解析到日常开发</h3>
<p data-nodeid="8989">本课时关于两种 QueryResult 的设计思想，同样可以应用到日常开发中。当我们面对如何处理来自数据库或外部数据源的数据时，可以根据需要设计<strong data-nodeid="9203">流式访问方式</strong>和<strong data-nodeid="9204">内存访问方式</strong>，这两种访问方式在数据访问过程中都具有一定的代表性。</p>
<p data-nodeid="8990">通常，我们会首先想到将所有访问到的数据存放在内存中，再进行二次处理，但这种处理方式会面临性能问题，流式访问方式性能更高，但需要我们挖掘适合的应用场景。</p>
<h3 data-nodeid="8991">小结与预告</h3>
<p data-nodeid="8992">今天介绍了 ShardingSphere 执行引擎主题的第二个课时，我们重点围绕执行引擎中的执行器展开讨论，给出了 StatementExecutor 和 PreparedStatementExecutor 这两种执行器的实现方式，也给出了 ShardingSphere 中关于连接模式的详细讨论。</p>
<p data-nodeid="8993">这里给大家留一道思考题：ShardingSphere 中连接模式的概念和作用是什么？欢迎你在留言区与大家讨论，我将逐一点评解答。</p>
<p data-nodeid="8994" class="">从类层结构而言，StatementExecutor 和 PreparedStatementExecutor 都属于底层组件，在下一课时，我们会介绍包括 ShardingStatement 和 PreparedShardingStatement 在内的位于更加上层的执行引擎组件。</p>

---

### 精选评论


