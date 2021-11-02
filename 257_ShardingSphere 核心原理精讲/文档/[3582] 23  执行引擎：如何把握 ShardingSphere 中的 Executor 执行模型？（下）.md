<p data-nodeid="3770" class="">在上一课时，我们已经对 ShardingSphere 执行引擎中关于底层的 SQLExecuteTemplate，以及上层的 StatementExecutor 和 PreparedStatementExecutor 对象进行了全面介绍。</p>
<p data-nodeid="3771">今天，我们在此基础上更上一层，重点关注 ShardingStatement 和 ShardingPreparedStatement 对象，这两个对象分别是 StatementExecutor 和 PreparedStatementExecutor 的使用者。</p>
<h3 data-nodeid="3772">ShardingStatement</h3>
<p data-nodeid="3773">我们先来看 ShardingStatement 类，该类中的变量在前面的内容中都已经有过介绍：</p>
<pre class="lang-java" data-nodeid="3774"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingConnection connection;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> StatementExecutor statementExecutor;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> returnGeneratedKeys;
<span class="hljs-keyword">private</span> SQLRouteResult sqlRouteResult;
<span class="hljs-keyword">private</span> ResultSet currentResultSet;
</code></pre>
<p data-nodeid="3775">ShardingStatement 类的构造函数同样不是很复杂，我们发现 StatementExecutor 就是在这个构造函数中完成了其创建过程：</p>
<pre class="lang-java" data-nodeid="3776"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ShardingStatement</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingConnection connection, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetType, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetConcurrency, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetHoldability)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(Statement.class);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.connection = connection;
&nbsp;&nbsp;&nbsp; //创建 StatementExecutor
&nbsp;&nbsp;&nbsp; statementExecutor = <span class="hljs-keyword">new</span> StatementExecutor(resultSetType, resultSetConcurrency, resultSetHoldability, connection);
}
</code></pre>
<p data-nodeid="3777">在继续介绍 ShardingStatement 之前，我们先梳理一下与它相关的类层结构。我们在 <strong data-nodeid="3874">“06 | 规范兼容：JDBC 规范与 ShardingSphere 是什么关系？”</strong> 中的 ShardingConnection 提到，ShardingSphere 通过适配器模式包装了自己的实现类，除了已经介绍的 ShardingConnection 类之外，还包含今天要介绍的 ShardingStatement 和 ShardingPreparedStament。</p>
<p data-nodeid="3778">根据这一点，我们可以想象 ShardingStatement 应该具备与 ShardingConnection 类似的类层结构：</p>
<p data-nodeid="3779"><img src="https://s0.lgstatic.com/i/image/M00/48/9D/CgqCHl9MzLGAdeNfAACM0dnojxQ073.png" alt="Drawing 0.png" data-nodeid="3878"></p>
<p data-nodeid="3780">然后我们来到上图中 AbstractStatementAdapter 类，这里的很多方法的风格都与 ShardingConnection 的父类 AbstractConnectionAdapter 一致，例如如下所示的 setPoolable 方法：</p>
<pre class="lang-java" data-nodeid="3781"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setPoolable</span><span class="hljs-params">(<span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> poolable)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.poolable = poolable;
&nbsp;&nbsp;&nbsp; recordMethodInvocation(targetClass, <span class="hljs-string">"setPoolable"</span>, <span class="hljs-keyword">new</span> Class[] {<span class="hljs-keyword">boolean</span>.class}, <span class="hljs-keyword">new</span> Object[] {poolable});
&nbsp;&nbsp;&nbsp; forceExecuteTemplate.execute((Collection) getRoutedStatements(), <span class="hljs-keyword">new</span> ForceExecuteCallback&lt;Statement&gt;() {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">execute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Statement statement)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement.setPoolable(poolable);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; });
</code></pre>
<blockquote data-nodeid="7099">
<p data-nodeid="7100" class="">这里涉及的 recordMethodInvocation 方法、ForceExecuteTemplate，以及 ForceExecuteCallback 我们都已经在“03 | 规范兼容：JDBC 规范与 ShardingSphere 是什么关系？”中进行了介绍，这里不再展开。</p>
</blockquote>








<p data-nodeid="3784">同样，AbstractStatementAdapter 的父类 AbstractUnsupportedOperationStatement 的作用也与 AbstractUnsupportedOperationConnection 的作用完全一致。</p>
<p data-nodeid="3785">了解了 ShardingStatement 的类层结构之后，我们来看它的核心方法，首当其冲的还是它的 executeQuery 方法：</p>
<pre class="lang-java" data-nodeid="3786"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ResultSet <span class="hljs-title">executeQuery</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (Strings.isNullOrEmpty(sql)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> SQLException(SQLExceptionConstant.SQL_STRING_NULL_OR_EMPTY);
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; ResultSet result;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//清除 StatementExecutor 中的相关变量</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clearPrevious();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行路由引擎，获取路由结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shard(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//初始化 StatementExecutor</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; initStatementExecutor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//调用归并引擎</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MergeEngine mergeEngine = MergeEngineFactory.newInstance(connection.getRuntimeContext().getDatabaseType(), connection.getRuntimeContext().getRule(), sqlRouteResult, connection.getRuntimeContext().getMetaData().getRelationMetas(), statementExecutor.executeQuery());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取归并结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result = getResultSet(mergeEngine);
&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; currentResultSet = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; currentResultSet = result;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="3787">这个方法中有几个子方法值得具体展开一下，首先是 shard 方法：</p>
<pre class="lang-java" data-nodeid="3788"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">shard</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> </span>{
    <span class="hljs-comment">//从 Connection 中获取 ShardingRuntimeContext 上下文</span>
&nbsp;&nbsp;&nbsp; ShardingRuntimeContext runtimeContext = connection.getRuntimeContext();
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建 SimpleQueryShardingEngine</span>
&nbsp;&nbsp;&nbsp; SimpleQueryShardingEngine shardingEngine = <span class="hljs-keyword">new</span> SimpleQueryShardingEngine(runtimeContext.getRule(), runtimeContext.getProps(), runtimeContext.getMetaData(), runtimeContext.getParseEngine());
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行分片路由并获取路由结果</span>
&nbsp;&nbsp;&nbsp; sqlRouteResult = shardingEngine.shard(sql, Collections.emptyList());
}
</code></pre>
<p data-nodeid="3789">这段代码就是路由引擎的入口，我们创建了 SimpleQueryShardingEngine，并调用它的 shard 方法获取路由结果对象 SQLRouteResult。</p>
<p data-nodeid="3790">然后我们来看 initStatementExecutor 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="3791"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initStatementExecutor</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; statementExecutor.init(sqlRouteResult);
&nbsp;&nbsp;&nbsp; replayMethodForStatements();
}
</code></pre>
<p data-nodeid="3792">这里通过路由结果对象 SQLRouteResult 对 statementExecutor 进行了初始化，然后执行了一个 replayMethodForStatements 方法：</p>
<pre class="lang-java" data-nodeid="3793"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">replayMethodForStatements</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Statement each : statementExecutor.getStatements()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; replayMethodsInvocation(each);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3794">该方法实际上就是调用了基于反射的 replayMethodsInvocation 方法，然后这个replayMethodsInvocation 方法会针对 statementExecutor 中所有 Statement的 SQL 操作执行目标方法。</p>
<p data-nodeid="3795">最后，我们通过执行 statementExecutor.executeQuery() 方法获取 SQL 执行的结果，并用这个结果来创建<strong data-nodeid="3901">归并引擎 MergeEngine</strong>，并通过归并引擎 MergeEngine 获取最终的执行结果。</p>
<blockquote data-nodeid="3796">
<p data-nodeid="3797"><strong data-nodeid="3906">归并引擎</strong>是 ShardingSphere 中与 SQL 解析引擎、路由引擎以及执行引擎并列的一个引擎，我们在下一课时中就会开始介绍这块内容，这里先不做具体展开。</p>
</blockquote>
<p data-nodeid="3798">以 ShardingStatement 中的其中一个 executeUpdate 方法为例，可以看到它的执行流程也与前面的 executeQuery 方法非常类似：</p>
<pre class="lang-java" data-nodeid="3799"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">executeUpdate</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-comment">//清除 StatementExecutor 中的相关变量</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clearPrevious();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行路由引擎，获取路由结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shard(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//初始化 StatementExecutor</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; initStatementExecutor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> statementExecutor.executeUpdate();
&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; currentResultSet = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3800">当然，对于 Update 操作而言，不需要通过归并引擎做结果的归并。</p>
<h3 data-nodeid="3801">ShardingPreparedStatement</h3>
<p data-nodeid="3802">我们接着来看 ShardingPreparedStatement 类，这个类的变量也基本都是前面介绍过的对象：</p>
<pre class="lang-java" data-nodeid="3803"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ShardingConnection connection;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String sql;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> PreparedQueryShardingEngine shardingEngine;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> PreparedStatementExecutor preparedStatementExecutor;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> BatchPreparedStatementExecutor batchPreparedStatementExecutor;
<span class="hljs-keyword">private</span> SQLRouteResult sqlRouteResult;
<span class="hljs-keyword">private</span> ResultSet currentResultSet;
</code></pre>
<p data-nodeid="3804">这里的 ShardingEngine、PreparedStatementExecutor 和 BatchPreparedStatementExecutor 对象的创建过程都发生在 ShardingPreparedStatement 的构造函数中。</p>
<p data-nodeid="3805">然后我们来看它的代表性方法 ExecuteQuery，如下所示：</p>
<pre class="lang-java" data-nodeid="3806"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ResultSet <span class="hljs-title">executeQuery</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; ResultSet result;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clearPrevious();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shard();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; initPreparedStatementExecutor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MergeEngine mergeEngine = MergeEngineFactory.newInstance(connection.getRuntimeContext().getDatabaseType(), connection.getRuntimeContext().getRule(), sqlRouteResult, connection.getRuntimeContext().getMetaData().getRelationMetas(), preparedStatementExecutor.executeQuery());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result = getResultSet(mergeEngine);
&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clearBatch();
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; currentResultSet = result;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="3807">这里我们没加注释，但也应该理解这一方法的执行流程，因为该方法的风格与 ShardingStatement 中的同名方法非常一致。</p>
<p data-nodeid="3808">关于 ShardingPreparedStatement 就没有太多可以介绍的内容了，我们接着来看它的父类<strong data-nodeid="3919">AbstractShardingPreparedStatementAdapter 类</strong>，看到该类持有一个 SetParameterMethodInvocation 的列表，以及一个参数列表：</p>
<pre class="lang-java" data-nodeid="3809"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;SetParameterMethodInvocation&gt; setParameterMethodInvocations = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
</code></pre>
<p data-nodeid="3810">这里的 SetParameterMethodInvocation 类直接集成了介绍 ShardingConnection 时提到的 JdbcMethodInvocation 类：</p>
<pre class="lang-java" data-nodeid="3811"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SetParameterMethodInvocation</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JdbcMethodInvocation</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Getter</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> index;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Getter</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Object value;

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SetParameterMethodInvocation</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Method method, <span class="hljs-keyword">final</span> Object[] arguments, <span class="hljs-keyword">final</span> Object value)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(method, arguments);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.index = (<span class="hljs-keyword">int</span>) arguments[<span class="hljs-number">0</span>];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.value = value;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">changeValueArgument</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Object value)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getArguments()[<span class="hljs-number">1</span>] = value;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3812">对于 ShardingPreparedStatement 而言，这个类的作用是在 JdbcMethodInvocation 中所保存的方法和参数的基础上，添加了 SQL 执行过程中所需要的参数信息。</p>
<p data-nodeid="3813">所以它的 replaySetParameter 方法就变成了如下的风格：</p>
<pre class="lang-java" data-nodeid="3814"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">replaySetParameter</span><span class="hljs-params">(<span class="hljs-keyword">final</span> PreparedStatement preparedStatement, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters)</span> </span>{
&nbsp;&nbsp;&nbsp; setParameterMethodInvocations.clear();
&nbsp;&nbsp; <span class="hljs-comment">//添加参数信息</span>
&nbsp;&nbsp;&nbsp; addParameters(parameters);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (SetParameterMethodInvocation each : setParameterMethodInvocations) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; each.invoke(preparedStatement);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3815">关于 AbstractShardingPreparedStatementAdapter 还需要注意的是它的<strong data-nodeid="3928">类层结构</strong>，如下图所示，可以看到 AbstractShardingPreparedStatementAdapter 继承了 AbstractUnsupportedOperationPreparedStatement 类；而 AbstractUnsupportedOperationPreparedStatement 却又继承了 AbstractStatementAdapter 类并实现了 PreparedStatement：</p>
<p data-nodeid="3816"><img src="https://s0.lgstatic.com/i/image/M00/48/92/Ciqc1F9MzNeACiagAACzQd-8eig186.png" alt="Drawing 2.png" data-nodeid="3931"></p>
<p data-nodeid="3817">形成这种类层结构的原因在于，PreparedStatement 本来就是在 Statement 的基础上添加了各种参数设置功能，换句话说，Statement 的功能 PreparedStatement 都应该有。</p>
<p data-nodeid="3818">所以一方面 AbstractStatementAdapter 提供了所有 Statement 的功能；另一方面，AbstractShardingPreparedStatementAdapter 首先把 AbstractStatementAdapter 所有的功能继承过来，但它自身可能有一些无法实现的关于 PreparedStatement 的功能，所以同样提供了 AbstractUnsupportedOperationPreparedStatement 类，并被最终的 AbstractShardingPreparedStatementAdapter 适配器类所继承。</p>
<p data-nodeid="3819">这样就形成了如上图所示的复杂类层结构。</p>
<h3 data-nodeid="3820">ShardingConnection</h3>
<p data-nodeid="3821">介绍完 ShardingStatement 和 ShardingPreparedStatement 之后，我们来关注使用它们的具体应用场景，这也是 ShardingSphere 执行引擎的最后一部分内容。</p>
<p data-nodeid="3822">通过查看调用关系，我们发现创建这两个类的入口都在 ShardingConnection 类中，该类包含了用于创建 ShardingStatement 的 createStatement 方法和用于创建 ShardingPreparedStatement 的 prepareStatement 方法，以及它们的各种重载方法：</p>
<pre class="lang-java" data-nodeid="3823"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Statement <span class="hljs-title">createStatement</span><span class="hljs-params">(<span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetType, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetConcurrency, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetHoldability)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingStatement(<span class="hljs-keyword">this</span>, resultSetType, resultSetConcurrency, resultSetHoldability);
 }
	&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> PreparedStatement <span class="hljs-title">prepareStatement</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetType, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetConcurrency, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> resultSetHoldability)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingPreparedStatement(<span class="hljs-keyword">this</span>, sql, resultSetType, resultSetConcurrency, resultSetHoldability);
}
</code></pre>
<p data-nodeid="3824">同时，ShardingConnection 中包含了用于管理分布式事务的 ShardingTransactionManager。关于分布式事务的讨论不是今天的重点，我们后面会有专题来做详细展开。</p>
<p data-nodeid="3825">但我们可以先看一下 commit 和 rollback 方法：</p>
<pre class="lang-java" data-nodeid="3826"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (TransactionType.LOCAL == transactionType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.commit();
&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingTransactionManager.commit();
&nbsp;&nbsp;&nbsp; }
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (TransactionType.LOCAL == transactionType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.rollback();
&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingTransactionManager.rollback();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3827">可以看到这两个方法的逻辑还是比较清晰的，即当事务类型为本地事务时直接调用 ShardingConnection 父类 AbstractConnectionAdapter 中的 commit 和 rollback 方法，这两个方法会调用真正的 connection 的相关方法。</p>
<p data-nodeid="3828">以 commit 方法为例，我们可以看到 AbstractConnectionAdapter 中基于这一设计思想的实现过程：</p>
<pre class="lang-java" data-nodeid="3829"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; forceExecuteTemplate.execute(cachedConnections.values(), <span class="hljs-keyword">new</span> ForceExecuteCallback&lt;Connection&gt;() {

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">execute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Connection connection)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connection.commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; });
}
</code></pre>
<h3 data-nodeid="3830">ShardingDataSource</h3>
<p data-nodeid="3831">我们知道在 JDBC 规范中，可以通过 DataSource 获取 Connection 对象。ShardingSphere 完全兼容 JDBC 规范，所以 ShardingConnection 的创建过程应该也是在对应的 DataSource 中，这个 DataSource 就是<strong data-nodeid="3948">ShardingDataSource</strong>。</p>
<p data-nodeid="3832">ShardingDataSource 类比较简单，其构造函数如下所示：</p>
<pre class="lang-java" data-nodeid="3833"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ShardingDataSource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Map&lt;String, DataSource&gt; dataSourceMap, <span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> Properties props)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(dataSourceMap);
&nbsp;&nbsp;&nbsp; checkDataSourceType(dataSourceMap);
&nbsp;&nbsp;&nbsp; runtimeContext = <span class="hljs-keyword">new</span> ShardingRuntimeContext(dataSourceMap, shardingRule, props, getDatabaseType());
}
</code></pre>
<p data-nodeid="3834">可以看到，ShardingRuntimeContext 这个上下文对象是在 ShardingDataSource 的构造函数中被创建的，而创建 ShardingConnection 的过程也很直接：</p>
<pre class="lang-java" data-nodeid="3835"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ShardingConnection <span class="hljs-title">getConnection</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingConnection(getDataSourceMap(), runtimeContext, TransactionTypeHolder.get());
}
</code></pre>
<p data-nodeid="3836">在 ShardingDataSource 的实现上，也同样采用的是装饰器模式，所以它的类层结构也与 ShardingConnection 的类似。在 ShardingDataSource 的父类 AbstractDataSourceAdapter 中，主要的工作是完成 DatabaseType 的创建，核心方法 createDatabaseType 如下所示：</p>
<pre class="lang-java" data-nodeid="3837"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> DatabaseType <span class="hljs-title">createDatabaseType</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DataSource dataSource)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (dataSource <span class="hljs-keyword">instanceof</span> AbstractDataSourceAdapter) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ((AbstractDataSourceAdapter) dataSource).databaseType;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> (Connection connection = dataSource.getConnection()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> DatabaseTypes.getDatabaseTypeByURL(connection.getMetaData().getURL());
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3838">可以看到这里使用到了 DatabaseTypes 类，该类负责 DatabaseType 实例的动态管理。而在 ShardingSphere 中，DatabaseType 接口代表数据库类型：</p>
<pre class="lang-java" data-nodeid="3839"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DatabaseType</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取数据库名称</span>
	<span class="hljs-function">String <span class="hljs-title">getName</span><span class="hljs-params">()</span></span>;
	<span class="hljs-comment">//获取 JDBC URL 的前缀</span>
	<span class="hljs-function">Collection&lt;String&gt; <span class="hljs-title">getJdbcUrlPrefixAlias</span><span class="hljs-params">()</span></span>;
	<span class="hljs-comment">//获取数据源元数据</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">DataSourceMetaData <span class="hljs-title">getDataSourceMetaData</span><span class="hljs-params">(String url, String username)</span></span>;
}
</code></pre>
<p data-nodeid="3840">可以想象 ShardingSphere 中针对各种数据库提供了 DatabaseType 接口的实现类，其中以 MySQLDatabaseType 为例：</p>
<pre class="lang-java" data-nodeid="3841"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MySQLDatabaseType</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">DatabaseType</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"MySQL"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Collection&lt;String&gt; <span class="hljs-title">getJdbcUrlPrefixAlias</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.singletonList(<span class="hljs-string">"jdbc:mysqlx:"</span>);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> MySQLDataSourceMetaData <span class="hljs-title">getDataSourceMetaData</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String url, <span class="hljs-keyword">final</span> String username)</span> </span>{
&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MySQLDataSourceMetaData(url);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3842">上述代码中的 MySQLDataSourceMetaData 实现了 DataSourceMetaData 接口，并提供如下所示的对输入 url 的解析过程：</p>
<pre class="lang-java" data-nodeid="3843"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">MySQLDataSourceMetaData</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String url)</span> </span>{
&nbsp;&nbsp;&nbsp; Matcher matcher = pattern.matcher(url);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!matcher.find()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UnrecognizedDatabaseURLException(url, pattern.pattern());
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; hostName = matcher.group(<span class="hljs-number">4</span>);
&nbsp;&nbsp;&nbsp; port = Strings.isNullOrEmpty(matcher.group(<span class="hljs-number">5</span>)) ? DEFAULT_PORT : Integer.valueOf(matcher.group(<span class="hljs-number">5</span>));
&nbsp;&nbsp;&nbsp; catalog = matcher.group(<span class="hljs-number">6</span>);
&nbsp;&nbsp;&nbsp; schema = <span class="hljs-keyword">null</span>;
}
</code></pre>
<p data-nodeid="3844">显然，DatabaseType 用于保存与特定数据库元数据相关的信息，ShardingSphere 还基于 SPI 机制实现对各种 DatabaseType 实例的动态管理。</p>
<p data-nodeid="3845">最后，我们来到 ShardingDataSourceFactory 工厂类，该类负责 ShardingDataSource 的创建：</p>
<pre class="lang-java" data-nodeid="3846"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingDataSourceFactory</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> DataSource <span class="hljs-title">createDataSource</span><span class="hljs-params">(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> Map&lt;String, DataSource&gt; dataSourceMap, <span class="hljs-keyword">final</span> ShardingRuleConfiguration shardingRuleConfig, <span class="hljs-keyword">final</span> Properties props)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ShardingDataSource(dataSourceMap, <span class="hljs-keyword">new</span> ShardingRule(shardingRuleConfig, dataSourceMap.keySet()), props);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3847">我们在这里创建了 ShardingDataSource，同时发现 ShardingRule 的创建过程实际上也是在这里，通过传入的 ShardingRuleConfiguration 来构建一个新的 ShardingRule 对象。</p>
<p data-nodeid="3848">一旦创建了 DataSource，我们就可以使用与 JDBC 规范完全兼容的 API，通过该 DataSource 完成各种 SQL 的执行。我们可以回顾 ShardingDataSourceFactory 的使用过程来加深对他的理解：</p>
<pre class="lang-java" data-nodeid="3849"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">dataSource</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//创建分片规则配置类</span>
&nbsp;&nbsp;&nbsp;&nbsp;ShardingRuleConfiguration shardingRuleConfig = <span class="hljs-keyword">new</span> ShardingRuleConfiguration();

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建分表规则配置类</span>
&nbsp;&nbsp;&nbsp;&nbsp;TableRuleConfiguration tableRuleConfig = <span class="hljs-keyword">new</span> TableRuleConfiguration(<span class="hljs-string">"user"</span>, <span class="hljs-string">"ds${0..1}.user${0..1}"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//创建分布式主键生成配置类</span>
&nbsp;&nbsp;&nbsp;&nbsp;Properties properties = <span class="hljs-keyword">new</span> Properties();
&nbsp;&nbsp;&nbsp;&nbsp;result.setProperty(<span class="hljs-string">"worker.id"</span>, <span class="hljs-string">"33"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;KeyGeneratorConfiguration keyGeneratorConfig = <span class="hljs-keyword">new</span> KeyGeneratorConfiguration(<span class="hljs-string">"SNOWFLAKE"</span>, <span class="hljs-string">"id"</span>, properties);
&nbsp;&nbsp;&nbsp;&nbsp;result.setKeyGeneratorConfig(keyGeneratorConfig);
&nbsp;&nbsp;&nbsp;&nbsp;shardingRuleConfig.getTableRuleConfigs().add(tableRuleConfig);

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//根据年龄分库，一共分为 2 个库</span>
&nbsp;&nbsp;&nbsp;shardingRuleConfig.setDefaultDatabaseShardingStrategyConfig(<span class="hljs-keyword">new</span> InlineShardingStrategyConfiguration(<span class="hljs-string">"sex"</span>, <span class="hljs-string">"ds${sex % 2}"</span>));

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//根据用户 id 分表，一共分为 2 张表</span>
&nbsp;&nbsp;&nbsp;&nbsp;shardingRuleConfig.setDefaultTableShardingStrategyConfig(<span class="hljs-keyword">new</span> StandardShardingStrategyConfiguration(<span class="hljs-string">"id"</span>, <span class="hljs-string">"user${id % 2}"</span>));

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//通过工厂类创建具体的 DataSource</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> ShardingDataSourceFactory.createDataSource(createDataSourceMap(), shardingRuleConfig, <span class="hljs-keyword">new</span> Properties());
}
</code></pre>
<p data-nodeid="3850">一旦获取了目标 DataSource 之后，我们就可以使用 JDBC 中的核心接口来执行传入的 SQL 语句：</p>
<pre class="lang-java" data-nodeid="3851"><code data-language="java"><span class="hljs-function">List&lt;User&gt; <span class="hljs-title">getUsers</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;User&gt; result = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span> (Connection connection = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PreparedStatement preparedStatement = connection.prepareStatement(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ResultSet resultSet = preparedStatement.executeQuery()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span> (resultSet.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;User user= <span class="hljs-keyword">new</span> User();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//省略设置 User 对象的赋值语句</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result.add(user);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="3852">ShardingSphere 通过在准备阶段获取的连接模式，在执行阶段生成<strong data-nodeid="3973">内存归并结果集</strong>或<strong data-nodeid="3974">流式归并结果集</strong>，并将其传递至<strong data-nodeid="3975">结果归并引擎</strong>，以进行下一步工作。</p>
<h3 data-nodeid="3853">从源码解析到日常开发</h3>
<p data-nodeid="3854">基于<strong data-nodeid="3982">适配器模式</strong>完成对 JDBC 规范的重写，是我们学习 ShardingSphere 框架非常重要的一个切入点，同样也是我们将这种模式应用到日常开发工作中的一个切入点。</p>
<p data-nodeid="3855">适配器模式是作为两个不兼容的接口之间的桥梁。在业务系统中，我们经常会碰到需要与外部系统进行对接和集成的场景，这个时候为了保证内部系统的功能演进，能够独立于外部系统进行发展，一般都需要采用适配器模式完成两者之间的隔离。</p>
<p data-nodeid="3856">当我们设计这种系统时，可以参考 JDBC 规范中的接口定义方式，以及 ShardingSphere 中基于这种接口定义方式，而完成适配的具体做法。</p>
<h3 data-nodeid="3857">小结与预告</h3>
<p data-nodeid="3858">这是 ShardingSphere 执行引擎的最后一个课时，我们围绕执行引擎的上层组件，给出了以“ Sharding”作为前缀的各种 JDBC 规范中的核心接口实现类。</p>
<p data-nodeid="3859">其中 ShardingStatement 和 ShardingPreparedStatement 直接依赖于上一课时介绍的 StatementExecutor 和 PreparedStatementExecutor；而 ShardingConnection 和 ShardingDataSource 则为我们使用执行引擎提供了入口。</p>
<p data-nodeid="3860">这里给你留一道思考题：ShardingSphere 中，AbstractShardingPreparedStatementAdapter 的类层结构为什么会比 AbstractStatementAdapter 复杂很多？欢迎你在留言区与大家讨论，我将逐一点评解答。</p>
<p data-nodeid="3861" class="">现在，我们已经通过执行引擎获取了来自不同数据源的结果数据，对于查询语句而言，我们通常都需要对这些结果数据进行归并才能返回给客户端。在接下来的内容中，就让我们来分析一下 ShardingSphere 的归并引擎。</p>

---

### 精选评论


