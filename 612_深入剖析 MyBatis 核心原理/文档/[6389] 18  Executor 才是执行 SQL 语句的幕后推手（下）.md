<p data-nodeid="3296">在上一讲中，我们首先介绍了模板方法模式的相关知识，然后介绍了 Executor 接口的核心方法，最后分析了 BaseExecutor 抽象类是如何利用模板方法模式为其他 Executor 抽象了一级缓存和事务管理的能力。这一讲，我们再来介绍剩余的四个重点 Executor 实现。</p>
<p data-nodeid="3297" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/26/66/CioPOWBa7q-Aa_fiAAF6eDPI3C0273.png" alt="图片13.png" data-nodeid="3302"></p>
<div data-nodeid="3298"><p style="text-align:center">Executor 接口继承关系图</p></div>




<h3 data-nodeid="1208">SimpleExecutor</h3>
<p data-nodeid="1209">我们来看 BaseExecutor 的第一个子类—— SimpleExecutor，同时<strong data-nodeid="1314">它也是 Executor 接口最简单的实现</strong>。</p>
<p data-nodeid="1210">正如上一讲中分析的那样，BaseExecutor 通过模板方法模式实现了读写一级缓存、事务管理等不随场景变化的基础方法，在 SimpleExecutor、ReuseExecutor、BatchExecutor 等实现类中，不再处理这些不变的逻辑，而只要关注 4 个 do*() 方法的实现即可。</p>
<p data-nodeid="1211">这里我们重点来看 SimpleExecutor 中 doQuery() 方法的实现逻辑。</p>
<ol data-nodeid="1212">
<li data-nodeid="1213">
<p data-nodeid="1214">通过 newStatementHandler() 方法创建 StatementHandler 对象，其中会根据 MappedStatement.statementType 配置创建相应的 StatementHandler 实现对象，并添加 RoutingStatementHandler 装饰器。</p>
</li>
<li data-nodeid="1215">
<p data-nodeid="1216">通过 prepareStatement() 方法初始化 Statement 对象，其中还依赖 ParameterHandler 填充 SQL 语句中的占位符。</p>
</li>
<li data-nodeid="1217">
<p data-nodeid="1218">通过 StatementHandler.query() 方法执行 SQL 语句，并通过我们前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6385&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1324">14</a>和<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6386&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1328">15</a>讲介绍的 DefaultResultSetHandler 将 ResultSet 映射成结果对象并返回。</p>
</li>
</ol>
<p data-nodeid="1219">doQuery() 方法的核心代码实现如下所示：</p>
<pre class="lang-java" data-nodeid="1220"><code data-language="java"><span class="hljs-keyword">public</span> &lt;E&gt; <span class="hljs-function">List&lt;E&gt; <span class="hljs-title">doQuery</span><span class="hljs-params">(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; Statement stmt = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Configuration configuration = ms.getConfiguration();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建StatementHandler对象，实际返回的是RoutingStatementHandler对象（我们在第16讲介绍过）</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 其中根据MappedStatement.statementType选择具体的StatementHandler实现</span>
&nbsp; &nbsp; &nbsp; &nbsp; StatementHandler handler = configuration.newStatementHandler(wrapper, ms, parameter, rowBounds, resultHandler, boundSql);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 完成StatementHandler的创建和初始化，该方法会调用StatementHandler.prepare()方法创建</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Statement对象，然后调用StatementHandler.parameterize()方法处理占位符</span>
&nbsp; &nbsp; &nbsp; &nbsp; stmt = prepareStatement(handler, ms.getStatementLog());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用StatementHandler.query()方法，执行SQL语句，并通过ResultSetHandler完成结果集的映射</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> handler.query(stmt, resultHandler);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; closeStatement(stmt);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1221">SimpleExecutor 中的 doQueryCursor()、update() 等方法实现与 doQuery() 方法的实现基本类似，这里不再展开介绍，你若感兴趣的话可以参考<a href="https://github.com/xxxlxy2008/mybatis?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1334">源码</a>进行分析。</p>
<h3 data-nodeid="1222">ReuseExecutor</h3>
<p data-nodeid="1223">你如果有过 JDBC 优化经验的话，可能会知道重用 Statement 对象是一种常见的优化手段，主要目的是减少 SQL 预编译开销，同时还会降低 Statement 对象的创建和销毁频率，这在一定程度上可以提升系统性能。</p>
<p data-nodeid="1224">ReuseExecutor 这个 BaseExecutor 实现就<strong data-nodeid="1345">实现了重用 Statement 的优化</strong>，ReuseExecutor 维护了一个 statementMap 字段（HashMap&lt;String, Statement&gt;类型）来缓存已有的 Statement 对象，该缓存的 Key 是 SQL 模板，Value 是 SQL 模板对应的 Statement 对象。这样在执行相同 SQL 模板时，我们就可以复用 Statement 对象了。</p>
<p data-nodeid="1225">ReuseExecutor 中的 do*() 方法实现与前面介绍的 SimpleExecutor 实现完全一样，两者唯一的<strong data-nodeid="1353">区别在于其中依赖的 prepareStatement() 方法</strong>：SimpleExecutor 每次都会创建全新的 Statement 对象，ReuseExecutor 则是先尝试查询 statementMap 缓存，如果缓存命中，则会重用其中的 Statement 对象。</p>
<p data-nodeid="1226">另外，在事务提交/回滚以及 Executor 关闭的时候，需要同时关闭 statementMap 集合中缓存的全部 Statement 对象，这部分逻辑是在 doFlushStatements() 方法中实现的，核心代码如下：</p>
<pre class="lang-java" data-nodeid="1227"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;BatchResult&gt; <span class="hljs-title">doFlushStatements</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> isRollback)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 关闭statementMap集合中缓存的全部Statement对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Statement stmt : statementMap.values()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; closeStatement(stmt);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 清空statementMap集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; statementMap.clear();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Collections.emptyList();
&nbsp; &nbsp; }
</code></pre>
<h3 data-nodeid="1228">BatchExecutor</h3>
<p data-nodeid="1229">批处理是 JDBC 编程中的另一种优化手段。</p>
<p data-nodeid="1230">JDBC 在执行 SQL 语句时，会将 SQL 语句以及实参通过网络请求的方式发送到数据库，一次执行一条 SQL 语句，一方面会减小请求包的有效负载，另一个方面会增加耗费在网络通信上的时间。通过批处理的方式，我们就可以在 JDBC 客户端缓存多条 SQL 语句，然后在 flush 或缓存满的时候，将多条 SQL 语句打包发送到数据库执行，这样就可以有效地降低上述两方面的损耗，从而提高系统性能。</p>
<p data-nodeid="1231">不过，有一点需要特别注意：每次向数据库发送的 SQL 语句的条数是有上限的，如果批量执行的时候超过这个上限值，数据库就会抛出异常，拒绝执行这一批 SQL 语句，所以我们<strong data-nodeid="1363">需要控制批量发送 SQL 语句的条数和频率</strong>。</p>
<p data-nodeid="1232"><strong data-nodeid="1370">BatchExecutor 是用于实现批处理的 Executor 实现</strong>，其中维护了一个 List<code data-backticks="1" data-nodeid="1368">&lt;Statement&gt;</code> 集合（statementList 字段）用来缓存一批 SQL，每个 Statement 可以写入多条 SQL。</p>
<p data-nodeid="1233">我们知道 JDBC 的批处理操作只支持 insert、update、delete 等修改操作，也就是说 BatchExecutor 对批处理的实现集中在 doUpdate() 方法中。在 doUpdate() 方法中追加一条待执行的 SQL 语句时，BatchExecutor 会先将该条 SQL 语句与最近一次追加的 SQL 语句进行比较，如果相同，则追加到最近一次使用的 Statement 对象中；如果不同，则追加到一个全新的 Statement 对象，同时会将新建的 Statement 对象放入 statementList 缓存中。</p>
<p data-nodeid="1234">下面是 BatchExecutor.doUpdate() 方法的核心逻辑：</p>
<pre class="lang-java" data-nodeid="1235"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">doUpdate</span><span class="hljs-params">(MappedStatement ms, Object parameterObject)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; <span class="hljs-keyword">final</span> Configuration configuration = ms.getConfiguration();
&nbsp; <span class="hljs-comment">// 创建StatementHandler对象</span>
&nbsp; <span class="hljs-keyword">final</span> StatementHandler handler = configuration.newStatementHandler(<span class="hljs-keyword">this</span>, ms, parameterObject, RowBounds.DEFAULT, <span class="hljs-keyword">null</span>, <span class="hljs-keyword">null</span>);
&nbsp; <span class="hljs-keyword">final</span> BoundSql boundSql = handler.getBoundSql();
&nbsp; &nbsp; <span class="hljs-comment">// 获取此次追加的SQL模板</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> String sql = boundSql.getSql();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Statement stmt;
&nbsp; &nbsp; <span class="hljs-comment">// 比较此次追加的SQL模板与最近一次追加的SQL模板，以及两个MappedStatement对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (sql.equals(currentSql) &amp;&amp; ms.equals(currentStatement)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 两者相同，则获取statementList集合中最后一个Statement对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> last = statementList.size() - <span class="hljs-number">1</span>;
&nbsp; &nbsp; &nbsp; &nbsp; stmt = statementList.get(last);
&nbsp; &nbsp; &nbsp; &nbsp; applyTransactionTimeout(stmt);
&nbsp; &nbsp; &nbsp; &nbsp; handler.parameterize(stmt); <span class="hljs-comment">// 设置实参</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 查找该Statement对象对应的BatchResult对象，并记录用户传入的实参</span>
&nbsp; &nbsp; &nbsp; &nbsp; BatchResult batchResult = batchResultList.get(last);
&nbsp; &nbsp; &nbsp; &nbsp; batchResult.addParameterObject(parameterObject);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Connection connection = getConnection(ms.getStatementLog());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建新的Statement对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; stmt = handler.prepare(connection, transaction.getTimeout());
&nbsp; &nbsp; &nbsp; &nbsp; handler.parameterize(stmt);<span class="hljs-comment">// 设置实参</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 更新currentSql和currentStatement</span>
&nbsp; &nbsp; &nbsp; &nbsp; currentSql = sql;
&nbsp; &nbsp; &nbsp; &nbsp; currentStatement = ms;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将新创建的Statement对象添加到statementList集合中</span>
&nbsp; &nbsp; &nbsp; &nbsp; statementList.add(stmt);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 为新Statement对象添加新的BatchResult对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; batchResultList.add(<span class="hljs-keyword">new</span> BatchResult(ms, sql, parameterObject));
&nbsp; &nbsp; }
&nbsp; &nbsp; handler.batch(stmt);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> BATCH_UPDATE_RETURN_VALUE;
}
</code></pre>
<p data-nodeid="1236">这里使用到的 BatchResult 用于记录批处理的结果，一个 BatchResult 对象与一个 Statement 对象对应，BatchResult 中维护了一个 updateCounts 字段（int[] 数组类型）来记录关联 Statement 对象执行批处理的结果。</p>
<p data-nodeid="1237">添加完待执行的 SQL 语句之后，我们再来看一下 doFlushStatements() 方法，其中会通过 Statement.executeBatch() 方法批量执行 SQL，然后 SQL 语句影响行数以及数据库生成的主键填充到相应的 BatchResult 对象中返回。下面是其核心实现：</p>
<pre class="lang-java" data-nodeid="1238"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;BatchResult&gt; <span class="hljs-title">doFlushStatements</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> isRollback)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 用于储存批处理的结果</span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;BatchResult&gt; results = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果明确指定了要回滚事务，则直接返回空集合，忽略statementList集合中记录的SQL语句</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isRollback) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Collections.emptyList();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>, n = statementList.size(); i &lt; n; i++) { <span class="hljs-comment">// 遍历statementList集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Statement stmt = statementList.get(i);<span class="hljs-comment">// 获取Statement对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; applyTransactionTimeout(stmt);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; BatchResult batchResult = batchResultList.get(i); <span class="hljs-comment">// 获取对应BatchResult对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用Statement.executeBatch()方法批量执行其中记录的SQL语句，并使用返回的int数组</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 更新BatchResult.updateCounts字段，其中每一个元素都表示一条SQL语句影响的记录条数</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; batchResult.setUpdateCounts(stmt.executeBatch());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MappedStatement ms = batchResult.getMappedStatement();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; List&lt;Object&gt; parameterObjects = batchResult.getParameterObjects();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取配置的KeyGenerator对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; KeyGenerator keyGenerator = ms.getKeyGenerator();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (Jdbc3KeyGenerator.class.equals(keyGenerator.getClass())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 获取数据库生成的主键，并记录到实参中对应的字段
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Jdbc3KeyGenerator jdbc3KeyGenerator = (Jdbc3KeyGenerator) keyGenerator;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; jdbc3KeyGenerator.processBatch(ms, stmt, parameterObjects);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!NoKeyGenerator.class.equals(keyGenerator.getClass())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 其他类型的KeyGenerator，会调用其processAfter()方法
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Object parameter : parameterObjects) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; keyGenerator.processAfter(<span class="hljs-keyword">this</span>, ms, stmt, parameter);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; closeStatement(stmt);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (BatchUpdateException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 异常处理逻辑
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 添加BatchResult到results集合
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; results.add(batchResult);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> results;
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 释放资源</span>
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="1239">CachingExecutor</h3>
<p data-nodeid="1240">CachingExecutor 是我们最后一个要介绍的 Executor 接口实现类，它是<strong data-nodeid="1384">一个 Executor 装饰器实现，会在其他 Executor 的基础之上添加二级缓存的相关功能</strong>。在上一讲中，我们已经介绍过了一级缓存，下面就接着讲解二级缓存相关的内容。</p>
<h4 data-nodeid="1241">1. 二级缓存</h4>
<p data-nodeid="1242">我们知道一级缓存的生命周期默认与 SqlSession 相同，而这里介绍的 MyBatis 中的二级缓存则与应用程序的生命周期相同。与二级缓存相关的配置主要有下面三项。</p>
<p data-nodeid="1243"><strong data-nodeid="1393">第一项，二级缓存全局开关</strong>。这个全局开关是 mybatis-config.xml 配置文件中的 cacheEnabled 配置项。当 cacheEnabled 被设置为 true 时，才会开启二级缓存功能，开启二级缓存功能之后，下面两项的配置才会控制二级缓存的行为。</p>
<p data-nodeid="1244"><strong data-nodeid="1402">第二项，命名空间级别开关</strong>。在 Mapper 配置文件中，可以通过配置 <code data-backticks="1" data-nodeid="1398">&lt;cache&gt;</code> 标签或 <code data-backticks="1" data-nodeid="1400">&lt;cache-ref&gt;</code> 标签开启二级缓存功能。</p>
<ul data-nodeid="1245">
<li data-nodeid="1246">
<p data-nodeid="1247">在解析到 <code data-backticks="1" data-nodeid="1404">&lt;cache&gt;</code> 标签时，MyBatis 会为当前 Mapper.xml 文件对应的命名空间创建一个关联的 Cache 对象（默认为 PerpetualCache 类型的对象），作为其二级缓存的实现。此外，<code data-backticks="1" data-nodeid="1406">&lt;cache&gt;</code> 标签中还提供了一个 type 属性，我们可以通过该属性使用自定义的 Cache 类型。</p>
</li>
<li data-nodeid="1248">
<p data-nodeid="1249">在解析到 <code data-backticks="1" data-nodeid="1409">&lt;cache-ref&gt;</code> 标签时，MyBatis 并不会创建新的 Cache 对象，而是根据 <code data-backticks="1" data-nodeid="1411">&lt;cache-ref&gt;</code> 标签的 namespace 属性查找指定命名空间对应的 Cache 对象，然后让当前命名空间与指定命名空间共享同一个 Cache 对象。</p>
</li>
</ul>
<p data-nodeid="1250"><strong data-nodeid="1419">第三项，语句级别开关</strong>。我们可以通过 <code data-backticks="1" data-nodeid="1417">&lt;select&gt;</code> 标签中的 useCache 属性，控制该 select 语句查询到的结果对象是否保存到二级缓存中，useCache 属性默认值为 true。</p>
<h4 data-nodeid="1251">2. TransactionalCache</h4>
<p data-nodeid="1252">了解了二级缓存的生命周期、基本概念以及相关配置之后，我们开始介绍 CachingExecutor 依赖的底层组件。</p>
<p data-nodeid="1253">CachingExecutor 底层除了依赖 PerpetualCache 实现来缓存数据之外，还会<strong data-nodeid="1429">依赖 TransactionalCache 和 TransactionalCacheManager 两个组件</strong>，下面我们就一一详细介绍下。</p>
<p data-nodeid="1254">TransactionalCache 是 Cache 接口众多实现之一，它也是一个装饰器，用来记录一个事务中添加到二级缓存中的缓存。</p>
<p data-nodeid="1255">TransactionalCache 中的 entriesToAddOnCommit 字段（Map<code data-backticks="1" data-nodeid="1432">&lt;Object, Object&gt;</code> 类型）用来暂存当前事务中添加到二级缓存中的数据，这些数据在事务提交时才会真正添加到底层的 Cache 对象（也就是二级缓存）中。这一点我们可以从 TransactionalCache 的 putObject() 方法以及 flushPendingEntries() 方法（commit() 方法会调用该方法）中看到相关代码实现：</p>
<pre class="lang-java" data-nodeid="1256"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">putObject</span><span class="hljs-params">(Object key, Object object)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 将数据暂存到entriesToAddOnCommit集合</span>
&nbsp; &nbsp; entriesToAddOnCommit.put(key, object);
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flushPendingEntries</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;Object, Object&gt; entry : entriesToAddOnCommit.entrySet()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将entriesToAddOnCommit集合中的数据添加到二级缓存</span>
&nbsp; &nbsp; &nbsp; &nbsp; delegate.putObject(entry.getKey(), entry.getValue());
&nbsp; &nbsp; }
&nbsp; &nbsp; ... <span class="hljs-comment">// 其他逻辑</span>
}
</code></pre>
<p data-nodeid="1257">那为什么要在事务提交时才将 entriesToAddOnCommit 集合中的缓存数据写入底层真正的二级缓存中，而不是像操作一级缓存那样，每次查询都直接写入缓存呢？其实这是<strong data-nodeid="1439">为了防止出现“脏读”</strong>。</p>
<p data-nodeid="1258">我们假设当前数据库的隔离级别是“不可重复读”，如下图所示，两个业务线程分别开启了 T1、T2 两个事务：</p>
<ul data-nodeid="2095">
<li data-nodeid="2096">
<p data-nodeid="2097">在事务 T1 中添加了记录 A，之后查询记录 A；</p>
</li>
<li data-nodeid="2098">
<p data-nodeid="2099">事务 T2 会查询记录 A。</p>
</li>
</ul>
<p data-nodeid="2100" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/66/CioPOWBa7oCAaZuSAAF5PEDTpm8320.png" alt="图片12.png" data-nodeid="2106"></p>
<div data-nodeid="2101"><p style="text-align:center">两事务并发操作的示意图</p></div>




<p data-nodeid="1266">如果事务 T1 查询记录 A 时，就将 A 对应的结果对象写入二级缓存，那在事务 T2 查询记录 A 时，会从二级缓存中直接拿到结果对象。此时的事务 T1 仍然未提交，也就出现了“脏读”。</p>
<p data-nodeid="1267">我们按照 TransactionalCache 的实现再来分析下，事务 T1 查询 A 数据的时候，未命中二级缓存，就会击穿到数据库，因为写入和读取 A 都是在事务 T1 中，所以能够查询成功，同时更新 entriesToAddOnCommit 集合。事务 T2 查询记录 A&nbsp;时，同样也会击穿二级缓存，访问数据库，因为此时写入和读取 A 是不同的事务，且数据库的事务隔离级别为“不可重复读”，这就导致事务 T2 无法查询到记录 A，也就避免了“脏读”。</p>
<p data-nodeid="1268">如上图所示，事务 T1 在提交时，会将 entriesToAddOnCommit 中的数据添加到二级缓存中，所以事务 T2 第二次查询记录 A 时，会命中二级缓存，也就出现了同一事务中多次读取的结果不同的现象，也就是我们说的“不可重复读”。</p>
<p data-nodeid="1269">TransactionalCache 中的另一个核心字段是 entriesMissedInCache，它用来记录未命中的 CacheKey 对象。在 getObject() 方法中，我们可以看到写入 entriesMissedInCache 集合的相关代码片段：</p>
<pre class="lang-java" data-nodeid="1270"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">getObject</span><span class="hljs-params">(Object key)</span> </span>{
&nbsp; &nbsp; Object object = delegate.getObject(key);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (object == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; entriesMissedInCache.add(key);
&nbsp; &nbsp; }
    ... <span class="hljs-comment">// 其他逻辑</span>
}
</code></pre>
<p data-nodeid="1271">在事务提交的时候，会将 entriesMissedInCache 集合中的 CacheKey 写入底层的二级缓存（写入时的 Value 为 null）。在事务回滚时，会调用底层二级缓存的 removeObject() 方法，删除 entriesMissedInCache 集合中 CacheKey。</p>
<p data-nodeid="1272">你可能会问，为什么要用 entriesMissedInCache 集合记录未命中缓存的 CacheKey 呢？为什么还要在缓存结束时处理这些 CacheKey 呢？这主要是与<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6380&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1454">第 9 讲</a>介绍的 BlockingCache 装饰器相关。在前面介绍 Cache 时我们提到过，CacheBuilder 默认会添加 BlockingCache 这个装饰器，而 BlockingCache 的 getObject() 方法会有给 CacheKey 加锁的逻辑，需要在 putObject() 方法或 removeObject() 方法中解锁，<strong data-nodeid="1460">否则这个 CacheKey 会被一直锁住，无法使用</strong>。</p>
<p data-nodeid="1273">看完 TransactionalCache 的核心实现之后，我们再来看 TransactionalCache 的管理者—— TransactionalCacheManager，其中定义了一个 transactionalCaches 字段（HashMap&lt;Cache, TransactionalCache&gt;类型）维护当前 CachingExecutor 使用到的二级缓存，该集合的 Key 是二级缓存对象，Value 是装饰二级缓存的 TransactionalCache 对象。</p>
<p data-nodeid="1274">TransactionalCacheManager 中的方法实现都比较简单，都是基于 transactionalCaches 集合以及 TransactionalCache 的同名方法实现的，这里不再展开介绍，你若感兴趣的话可以参考<a href="https://github.com/xxxlxy2008/mybatis?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1467">源码</a>进行分析。</p>
<h4 data-nodeid="1275">3. 核心实现</h4>
<p data-nodeid="1276">了解了二级缓存基本概念以及 TransactionalCache 核心实现之后，我们再来看 CachingExecutor 的核心实现。</p>
<p data-nodeid="1277">CachingExecutor 作为一个装饰器，其中自然会维护一个 Executor 类型字段指向被装饰的 Executor 对象，同时它还创建了一个 TransactionalCacheManager 对象来管理使用到的二级缓存。</p>
<p data-nodeid="1278"><strong data-nodeid="1478">CachingExecutor 的核心在于 query() 方法</strong>，其核心操作大致可总结为如下。</p>
<ul data-nodeid="1279">
<li data-nodeid="1280">
<p data-nodeid="1281">获取 BoundSql 对象，创建查询语句对应的 CacheKey 对象。</p>
</li>
<li data-nodeid="1282">
<p data-nodeid="1283">尝试获取当前命名空间使用的二级缓存，如果没有指定二级缓存，则表示未开启二级缓存功能。如果未开启二级缓存功能，则直接使用被装饰的 Executor 对象进行数据库查询操作。如果开启了二级缓存功能，则继续后面的步骤。</p>
</li>
<li data-nodeid="1284">
<p data-nodeid="1285">查询二级缓存，这里使用到 TransactionalCacheManager.getObject() 方法，如果二级缓存命中，则直接将该结果对象返回。</p>
</li>
<li data-nodeid="1286">
<p data-nodeid="1287">如果二级缓存未命中，则通过被装饰的 Executor 对象进行查询。正如前面介绍的那样，BaseExecutor 会先查询一级缓存，如果一级缓存未命中时，才会真正查询数据库。最后，会将查询到的结果对象放入 TransactionalCache.entriesToAddOnCommit 集合中暂存，等待事务提交时再写入二级缓存。</p>
</li>
</ul>
<p data-nodeid="1288">下面是 CachingExecutor.query() 方法的核心代码片段：</p>
<pre class="lang-java" data-nodeid="1289"><code data-language="java"><span class="hljs-keyword">public</span> &lt;E&gt; <span class="hljs-function">List&lt;E&gt; <span class="hljs-title">query</span><span class="hljs-params">(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 获取BoundSql对象</span>
&nbsp; &nbsp; BoundSql boundSql = ms.getBoundSql(parameterObject);
    <span class="hljs-comment">// 创建相应的CacheKey</span>
&nbsp; &nbsp; CacheKey key = createCacheKey(ms, parameterObject, rowBounds, boundSql);
    <span class="hljs-comment">// 调用下面的query()方法重载</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
<span class="hljs-keyword">public</span> &lt;E&gt; <span class="hljs-function">List&lt;E&gt; <span class="hljs-title">query</span><span class="hljs-params">(MappedStatement ms, Object parameterObject, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; Cache cache = ms.getCache(); <span class="hljs-comment">// 获取该命名空间使用的二级缓存</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cache != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 是否开启了二级缓存功能</span>
&nbsp; &nbsp; &nbsp; &nbsp; flushCacheIfRequired(ms); <span class="hljs-comment">// 根据&lt;select&gt;标签配置决定是否需要清空二级缓存</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 检测useCache配置以及是否使用了resultHandler配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ms.isUseCache() &amp;&amp; resultHandler == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ensureNoOutParams(ms, boundSql); <span class="hljs-comment">// 是否包含输出参数</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 查询二级缓存</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; List&lt;E&gt; list = (List&lt;E&gt;) tcm.getObject(cache, key);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (list == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 二级缓存未命中，通过被装饰的Executor对象查询结果对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; list = delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将查询结果放入TransactionalCache.entriesToAddOnCommit集合中暂存</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; tcm.putObject(cache, key, list);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> list;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 如果未开启二级缓存，直接通过被装饰的Executor对象查询结果对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> delegate.query(ms, parameterObject, rowBounds, resultHandler, key, boundSql);
}
</code></pre>
<h3 data-nodeid="1290">总结</h3>
<p data-nodeid="1291">紧接上一讲的内容，我们详细分析了 Executor 接口的核心实现类。</p>
<ul data-nodeid="1292">
<li data-nodeid="1293">
<p data-nodeid="1294">首先介绍了最常用、也是最简单的 Executor 实现类—— SimpleExecutor 实现，它底层完全依赖 StatementHandler、DefaultResultSetHandler 和 JDBC API 完成数据库查询和结果集映射。</p>
</li>
<li data-nodeid="1295">
<p data-nodeid="1296">接下来讲解了 ReuseExecutor 和 BatchExecutor 实现，其中 ReuseExecutor 实现了 Statement 对象的重用，而 BatchExecutor 实现了批处理的相关逻辑。</p>
</li>
<li data-nodeid="1297">
<p data-nodeid="1298">最后讲解了 CachingExecutor 实现，其中重点介绍了二级缓存的内容以及 CachingExecutor 底层的 TransactionalCache、TransactionalCacheManager 等核心组件。</p>
</li>
</ul>
<p data-nodeid="1299">从下一讲开始，我们就进入本课程的第四个模块，重点将放在 MyBatis 的接口层、插件体系以及衍生框架上面，记得按时来听课。</p>
<hr data-nodeid="1300">
<p data-nodeid="1301"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1494"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="1493"></a></p>
<p data-nodeid="1302"><strong data-nodeid="1498">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="1303" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1502">点击链接，提升自己</a>！</p>

---

### 精选评论

##### *成：
> 为什么commit和rollback之后需要清空statementmap呢？

##### **用户8801：
> 个人感觉二级缓存对一般的应用用处不大，只能用于不需要实时数据的情况，在多项目部署的情况下，容易导致读取不到db的真实数据。

