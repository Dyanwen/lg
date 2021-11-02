<p data-nodeid="50488" class="">今天我们将在上一课时的基础上，详细展开 ShardingSphere 中分布式事务的具体实现过程。首先，我们将介绍支持强一致性事务的 XAShardingTransactionManager。</p>
<h3 data-nodeid="50489">XAShardingTransactionManager</h3>
<p data-nodeid="50490">让我们回到 ShardingSphere，来到 sharding-transaction-xa-core 工程的 XAShardingTransactionManager 类，该类是分布式事务的 XA 实现类。</p>
<p data-nodeid="50491">我们先来看 XAShardingTransactionManager 类的定义和所包含的变量：</p>
<pre class="lang-java" data-nodeid="50492"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">XAShardingTransactionManager</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ShardingTransactionManager</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Map&lt;String, XATransactionDataSource&gt; cachedDataSources = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();

	<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> XATransactionManager xaTransactionManager = XATransactionManagerLoader.getInstance().getTransactionManager();
	&nbsp;
}
</code></pre>
<p data-nodeid="50493">可以看到 XAShardingTransactionManager 实现了上一课时中介绍的 ShardingTransactionManager 接口，并保存着一组 XATransactionDataSource。同时，XATransactionManager 实例的加载仍然是采用了 JDK 中的 ServiceLoader 类，如下所示：</p>
<pre class="lang-java" data-nodeid="50494"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> XATransactionManager <span class="hljs-title">load</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Iterator&lt;XATransactionManager&gt; xaTransactionManagers = ServiceLoader.load(XATransactionManager.class).iterator();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!xaTransactionManagers.hasNext()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> AtomikosTransactionManager();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; XATransactionManager result = xaTransactionManagers.next();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (xaTransactionManagers.hasNext()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; log.warn(<span class="hljs-string">"There are more than one transaction mangers existing, chosen first one by default."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="50495">XATransactionManager 就是对各种第三方 XA 事务管理器的一种抽象，通过上述代码，可以看到在找不到合适的 XATransactionManager 的情况下，系统默认会创建一个 AtomikosTransactionManager。而这个 XATransactionManager 的定义实际上是位于单独的一个代码工程中，即 sharding-transaction-xa-spi 工程，该接口定义如下所示：</p>
<pre class="lang-java" data-nodeid="50496"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">XATransactionManager</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AutoCloseable</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//初始化 XA 事务管理器</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//注册事务恢复资源</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">registerRecoveryResource</span><span class="hljs-params">(String dataSourceName, XADataSource xaDataSource)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//移除事务恢复资源</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">removeRecoveryResource</span><span class="hljs-params">(String dataSourceName, XADataSource xaDataSource)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//嵌入一个 SingleXAResource 资源</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">enlistResource</span><span class="hljs-params">(SingleXAResource singleXAResource)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//返回 TransactionManager</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">TransactionManager <span class="hljs-title">getTransactionManager</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="50497">这些接口方法从命名上基本可以理解其含义，但详细的用法我们还是要结合具体的 XATransactionManager 实现类进行理解。这里我们还发现了一个 SingleXAResource，这个类同样位于 sharding-transaction-xa-spi 工程中，从名称上看，应该是对 JTA 中 XAResource 接口的一种实现，我们来看一下：</p>
<pre class="lang-java" data-nodeid="50498"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingleXAResource</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">XAResource</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String resourceName;

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> XAResource delegate;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Xid xid, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> i)</span> <span class="hljs-keyword">throws</span> XAException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; delegate.start(xid, i);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Xid xid, <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> b)</span> <span class="hljs-keyword">throws</span> XAException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; delegate.commit(xid, b);
&nbsp;&nbsp;&nbsp; }

	<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Xid xid)</span> <span class="hljs-keyword">throws</span> XAException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; delegate.rollback(xid);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isSameRM</span><span class="hljs-params">(<span class="hljs-keyword">final</span> XAResource xaResource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SingleXAResource singleXAResource = (SingleXAResource) xaResource;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> resourceName.equals(singleXAResource.getResourceName());
	}
	…
}
</code></pre>
<p data-nodeid="50499">可以看到 SingleXAResource 虽然实现了 JTA 的 XAResource 接口，但更像是一个代理类，具体的操作方法还是委托给了内部的 XAResource 进行实现。</p>
<p data-nodeid="50500">接下来，我们将围绕 XA 分布式事务中的几个核心类展开讨论。</p>
<h4 data-nodeid="50501">1.XADataSource</h4>
<p data-nodeid="50502">XADataSource 属于 JDBC 规范中的内容，我们在“03 | 规范兼容：JDBC 规范与 ShardingSphere 是什么关系？”中已经提到过这个接口，该接口的作用就是获取 XAConnection。</p>
<p data-nodeid="50503">那么 XADataSource 是如何构建出来的呢？我们首先找到了一个 XADataSourceFactory 工厂类，显然该类负责生成具体的 XADataSource，如下所示的就是完成这一工作的 build 方法：</p>
<pre class="lang-java" data-nodeid="50504"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> XADataSource <span class="hljs-title">build</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> DataSource dataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; XADataSourceDefinition xaDataSourceDefinition = XADataSourceDefinitionFactory.getXADataSourceDefinition(databaseType);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; XADataSource result = createXADataSource(xaDataSourceDefinition);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Properties xaProperties = xaDataSourceDefinition.getXAProperties(SWAPPER.swap(dataSource));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PropertyUtils.setProperties(result, xaProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="50505">这里首先用到了一个 XADataSourceDefinition 接口，从命名上看应该是关于 XADataSource 的一种定义，如下所示：</p>
<pre class="lang-java" data-nodeid="50506"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">XADataSourceDefinition</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">DatabaseTypeAwareSPI</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 XA 驱动类名</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Collection&lt;String&gt; <span class="hljs-title">getXADriverClassName</span><span class="hljs-params">()</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 XA 属性</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Properties <span class="hljs-title">getXAProperties</span><span class="hljs-params">(DatabaseAccessConfiguration databaseAccessConfiguration)</span></span>;
}
</code></pre>
<p data-nodeid="50507">可以看到这个接口继承了 DatabaseTypeAwareSPI，从命名上看这也是一个 SPI 接口，其定义如下所示：</p>
<pre class="lang-java" data-nodeid="50508"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DatabaseTypeAwareSPI</span> </span>{ 
&nbsp;&nbsp; &nbsp;<span class="hljs-comment">//获取数据库类型</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getDatabaseType</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="50509">在 ShardingSphere 中，继承 DatabaseTypeAwareSPI 接口的就只有 XADataSourceDefinition 接口，而后者存在一批实现类，整体的类层结构如下所示：</p>
<p data-nodeid="50510"><img src="https://s0.lgstatic.com/i/image/M00/50/92/Ciqc1F9jCmiAI4cLAAE2ATnYWp4900.png" alt="Drawing 0.png" data-nodeid="50597"></p>
<div data-nodeid="50511"><p style="text-align:center">XADataSourceDefinition 的实现类</p></div>
<p data-nodeid="50512">这里以 MySQLXADataSourceDefinition 为例展开讨论，该类分别实现了 DatabaseTypeAwareSPI 和 XADataSourceDefinition 这两个接口中所定义的三个方法：</p>
<pre class="lang-java" data-nodeid="50513"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MySQLXADataSourceDefinition</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">XADataSourceDefinition</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getDatabaseType</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"MySQL"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Collection&lt;String&gt; <span class="hljs-title">getXADriverClassName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Arrays.asList(<span class="hljs-string">"com.mysql.jdbc.jdbc2.optional.MysqlXADataSource"</span>, <span class="hljs-string">"com.mysql.cj.jdbc.MysqlXADataSource"</span>);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Properties <span class="hljs-title">getXAProperties</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseAccessConfiguration databaseAccessConfiguration)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Properties result = <span class="hljs-keyword">new</span> Properties();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setProperty(<span class="hljs-string">"user"</span>, databaseAccessConfiguration.getUsername());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setProperty(<span class="hljs-string">"password"</span>, Optional.fromNullable(databaseAccessConfiguration.getPassword()).or(<span class="hljs-string">""</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setProperty(<span class="hljs-string">"URL"</span>, databaseAccessConfiguration.getUrl());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;…
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50514">我们从这里得知，作为数据库供应商，MySQL 提供了两个 XADataSource 的驱动程序。而在 getXAProperties 中，我们发现 URL、Username 和 Password 等信息是通过 DatabaseAccessConfiguration 对象进行获取的，该对象在本文后面会介绍到。</p>
<p data-nodeid="50515">另一方面，因为 DatabaseTypeAwareSPI 接口命名中带有 SPI，所以我们不难想象各种 XADataSourceDefinition 实际上也是基于 SPI 机制进行加载的，这在用于获取 XADataSourceDefinition 的工厂类 XADataSourceDefinitionFactory 中可以得到确认：</p>
<pre class="lang-java" data-nodeid="50516"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">XADataSourceDefinitionFactory</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Map&lt;DatabaseType, XADataSourceDefinition&gt; XA_DATA_SOURCE_DEFINITIONS = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();

	<span class="hljs-keyword">static</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过 ServiceLoader 加载 XADataSourceDefinition</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (XADataSourceDefinition each : ServiceLoader.load(XADataSourceDefinition.class)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; XA_DATA_SOURCE_DEFINITIONS.put(DatabaseTypes.getActualDatabaseType(each.getDatabaseType()), each);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> XADataSourceDefinition <span class="hljs-title">getXADataSourceDefinition</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> XA_DATA_SOURCE_DEFINITIONS.get(databaseType);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50517">同样，在 sharding-transaction-xa-core 工程中，我们也发现了如下所示的 SPI 配置信息：</p>
<p data-nodeid="50518"><img src="https://s0.lgstatic.com/i/image/M00/50/93/Ciqc1F9jCoWAOFRpAACUXKjEF6o633.png" alt="Drawing 1.png" data-nodeid="50604"></p>
<div data-nodeid="50519"><p style="text-align:center">sharding-transaction-xa-core 工程中的 SPI 配置</p></div>
<p data-nodeid="50520">当根据数据库类型获取了对应的 XADataSourceDefinition 之后，我们就可以根据 XADriverClassName 来创建具体的 XADataSource：</p>
<pre class="lang-java" data-nodeid="50521"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> XADataSource <span class="hljs-title">loadXADataSource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String xaDataSourceClassName)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Class xaDataSourceClass;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//加载 XADataSource 实现类</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaDataSourceClass = Thread.currentThread().getContextClassLoader().loadClass(xaDataSourceClassName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> ClassNotFoundException ignored) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaDataSourceClass = Class.forName(xaDataSourceClassName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> ClassNotFoundException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"Failed to load [%s]"</span>, xaDataSourceClassName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (XADataSource) xaDataSourceClass.newInstance();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> InstantiationException | IllegalAccessException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"Failed to instance [%s]"</span>, xaDataSourceClassName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50522">这里会先从当前线程的 ContextClassLoader 中加载目标驱动的实现类，如果加载不到，就直接通过反射进行创建，最后返回 XADataSource 的实例对象。</p>
<p data-nodeid="50523">当获取了 XADataSource 的实例对象之后，我们需要设置它的属性，这部分工作是由 DataSourceSwapper 类来完成的。在这里，ShardingSphere 针对不同类型的数据库连接池工具还专门做了一层封装，提取了 DataSourcePropertyProvider 接口用于对 DataSource的URL 、Username 和 Password 等基础信息进行抽象。</p>
<p data-nodeid="50524">DataSourcePropertyProvider 接口的定义如下所示：</p>
<pre class="lang-java" data-nodeid="50525"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DataSourcePropertyProvider</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getDataSourceClassName</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getURLPropertyName</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getUsernamePropertyName</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getPasswordPropertyName</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="50526">DataSourcePropertyProvider 的实现类有两个，一个是 DefaultDataSourcePropertyProvider，另一个是 HikariCPPropertyProvider。ShardingSphere 默认使用的是 HikariCPPropertyProvider，这点可以从如下所示的 SPI 配置文件中得到确认：</p>
<p data-nodeid="50527"><img src="https://s0.lgstatic.com/i/image/M00/50/93/Ciqc1F9jCpSAGChUAAB8-cv8fCU688.png" alt="Drawing 2.png" data-nodeid="50612"></p>
<div data-nodeid="50528"><p style="text-align:center">DataSourcePropertyProvider 的 SPI 配置</p></div>
<p data-nodeid="50529">HikariCPPropertyProvider 实现了 DataSourcePropertyProvider 接口，并包含了对这些基础信息的定义：</p>
<pre class="lang-java" data-nodeid="50530"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HikariCPPropertyProvider</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">DataSourcePropertyProvider</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getDataSourceClassName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"com.zaxxer.hikari.HikariDataSource"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getURLPropertyName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"jdbcUrl"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUsernamePropertyName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"username"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getPasswordPropertyName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"password"</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50531">然后在 DataSourceSwapper 的 swap 方法中，实际上就是通过反射来构建 findGetterMethod 工具方法，并以此获取 URL、Username 和 Password 等基础信息，并返回一个 DatabaseAccessConfiguration 对象供具体的 XADataSourceDefinition 进行使用。</p>
<p data-nodeid="50532">swap 方法的实现如下所示：</p>
<pre class="lang-java" data-nodeid="50533"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DatabaseAccessConfiguration <span class="hljs-title">swap</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DataSource dataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataSourcePropertyProvider provider = DataSourcePropertyProviderLoader.getProvider(dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String url = (String) findGetterMethod(dataSource, provider.getURLPropertyName()).invoke(dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = (String) findGetterMethod(dataSource, provider.getUsernamePropertyName()).invoke(dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = (String) findGetterMethod(dataSource, provider.getPasswordPropertyName()).invoke(dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DatabaseAccessConfiguration(url, username, password);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;} <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> ReflectiveOperationException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"Cannot swap data source type: `%s`, please provide an implementation from SPI `%s`"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSource.getClass().getName(), DataSourcePropertyProvider.class.getName());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50534">至此，我们对 XADataSource 的构建过程描述完毕。这个过程不算复杂，但涉及的类比较多，值得我们以 XADataSourceFactory 为中心画一张类图作为总结：</p>
<p data-nodeid="50535"><img src="https://s0.lgstatic.com/i/image/M00/50/93/Ciqc1F9jCqGAYmlZAACYlVXsQ44048.png" alt="image.png" data-nodeid="50619"></p>
<h4 data-nodeid="50536">2.XAConnection</h4>
<p data-nodeid="50537">讲完 XADataSource，我们接着来讲 XAConnection，XAConnection 同样是 JDBC 规范中的接口。</p>
<p data-nodeid="50538">负责创建 XAConnection 的工厂类 XAConnectionFactory 如下所示：</p>
<pre class="lang-java" data-nodeid="50539"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">XAConnectionFactory</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//基于普通 Connection 创建 XAConnection </span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> XAConnection <span class="hljs-title">createXAConnection</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> XADataSource xaDataSource, <span class="hljs-keyword">final</span> Connection connection)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">switch</span> (databaseType.getName()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> <span class="hljs-string">"MySQL"</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MySQLXAConnectionWrapper().wrap(xaDataSource, connection);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> <span class="hljs-string">"MariaDB"</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MariaDBXAConnectionWrapper().wrap(xaDataSource, connection);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> <span class="hljs-string">"PostgreSQL"</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> PostgreSQLXAConnectionWrapper().wrap(xaDataSource, connection);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> <span class="hljs-string">"H2"</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> H2XAConnectionWrapper().wrap(xaDataSource, connection);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">default</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UnsupportedOperationException(String.format(<span class="hljs-string">"Cannot support database type: `%s`"</span>, databaseType));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50540">可以看到，相较 XADataSource，创建 XAConnection 的过程就显得直接明了。这里通过一个 switch 语句根据数据库类型分别构建了对应的 ConnectionWrapper，然后再调用 wrap 方法返回 XAConnection。</p>
<p data-nodeid="50541">我们还是以 MySQLXAConnectionWrapper 为例来分析具体的实现过程。</p>
<p data-nodeid="50542">MySQLXAConnectionWrapper 实现了 XAConnectionWrapper 接口，所以我们先来看 XAConnectionWrapper 接口的定义：</p>
<pre class="lang-java" data-nodeid="50543"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">XAConnectionWrapper</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//基于 XADataSource 把 Connection 包装成 XAConnection</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">XAConnection <span class="hljs-title">wrap</span><span class="hljs-params">(XADataSource xaDataSource, Connection connection)</span></span>;
}
</code></pre>
<p data-nodeid="50544">XAConnectionWrapper 接口只有一个方法，即根据传入的 XADataSource 和一个普通 Connection 对象创建出一个新的 XAConnection 对象。XAConnectionWrapper 接口的类层结构如下所示：</p>
<p data-nodeid="50545"><img src="https://s0.lgstatic.com/i/image/M00/50/93/Ciqc1F9jCrCAXTkWAAD4zJLBg8I622.png" alt="Drawing 4.png" data-nodeid="50629"></p>
<div data-nodeid="50546"><p style="text-align:center">XAConnectionWrapper 接口的实现类</p></div>
<p data-nodeid="50547">MySQLXAConnectionWrapper 中的 warp 方法如下所示：</p>
<pre class="lang-java" data-nodeid="50548"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> XAConnection <span class="hljs-title">wrap</span><span class="hljs-params">(<span class="hljs-keyword">final</span> XADataSource xaDataSource, <span class="hljs-keyword">final</span> Connection connection)</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取真实 Connection 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection physicalConnection = unwrapPhysicalConnection(xaDataSource.getClass().getName(), connection);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Method method = xaDataSource.getClass().getDeclaredMethod(<span class="hljs-string">"wrapConnection"</span>, Connection.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; method.setAccessible(<span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //通过反射包装 Connection 对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (XAConnection) method.invoke(xaDataSource, physicalConnection);
}
</code></pre>
<p data-nodeid="50549">上述方法的流程为先通过 unwrapPhysicalConnection 将传入的 Connection 转变为一个真实的连接对象，然后再基于 XADataSource 的 wrapConnection 方法通过反射对这个物理连接进行包装，从而形成一个 XAConnection 对象。</p>
<p data-nodeid="50550">对于 Mysql 而言，我们在前面的内容中已经知道它有两种 XADataSource 驱动类。而在 MySQLXAConnectionWrapper 我们同样找到了如下所示的这两种驱动类的类名定义：</p>
<pre class="lang-java" data-nodeid="50551"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String MYSQL_XA_DATASOURCE_5 = <span class="hljs-string">"com.mysql.jdbc.jdbc2.optional.MysqlXADataSource"</span>;

<span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String MYSQL_XA_DATASOURCE_8 = <span class="hljs-string">"com.mysql.cj.jdbc.MysqlXADataSource"</span>;
</code></pre>
<p data-nodeid="50552">显然，根据数据库版本的不同，这两个驱动类的行为也有所不同。因此，unwrapPhysicalConnection 的处理过程如下所示：</p>
<pre class="lang-java" data-nodeid="50553"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Connection <span class="hljs-title">unwrapPhysicalConnection</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String xaDataSourceClassName, <span class="hljs-keyword">final</span> Connection connection)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">switch</span> (xaDataSourceClassName) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> MYSQL_XA_DATASOURCE_5:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (Connection) connection.unwrap(Class.forName(<span class="hljs-string">"com.mysql.jdbc.Connection"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> MYSQL_XA_DATASOURCE_8:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (Connection) connection.unwrap(Class.forName(<span class="hljs-string">"com.mysql.cj.jdbc.JdbcConnection"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">default</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UnsupportedOperationException(String.format(<span class="hljs-string">"Cannot support xa datasource: `%s`"</span>, xaDataSourceClassName));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50554">作为对比，我们再来看 PostgreSQLXAConnectionWrapper，它的 wrap 方法则比较简单，如下所示。显然，这部分内容的理解需要对不同的数据库驱动有一定的了解。</p>
<pre class="lang-java" data-nodeid="50555"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> XAConnection <span class="hljs-title">wrap</span><span class="hljs-params">(<span class="hljs-keyword">final</span> XADataSource xaDataSource, <span class="hljs-keyword">final</span> Connection connection)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BaseConnection physicalConnection = (BaseConnection) connection.unwrap(Class.forName(<span class="hljs-string">"org.postgresql.core.BaseConnection"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> PGXAConnection(physicalConnection);
}
</code></pre>
<h4 data-nodeid="50556">3.XATransactionDataSource</h4>
<p data-nodeid="50557">介绍完了 XADataSource 和 XAConnection 的创建过程之后，让我们回到 XAShardingTransactionManager，我们发现这里用到的 DataSource 并不是 JDBC 中原生的 XADataSource，而是一种 XATransactionDataSource。</p>
<p data-nodeid="50558">我们来到这个 XATransactionDataSource 类，该类的变量和构造函数如下所示：</p>
<pre class="lang-java" data-nodeid="50559"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> DatabaseType databaseType;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String resourceName;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> DataSource dataSource;
<span class="hljs-keyword">private</span> XADataSource xaDataSource;
<span class="hljs-keyword">private</span> XATransactionManager xaTransactionManager; 
	&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">XATransactionDataSource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> String resourceName, <span class="hljs-keyword">final</span> DataSource dataSource, <span class="hljs-keyword">final</span> XATransactionManager xaTransactionManager)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.databaseType = databaseType;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.resourceName = resourceName;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.dataSource = dataSource;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!CONTAINER_DATASOURCE_NAMES.contains(dataSource.getClass().getSimpleName())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.xaDataSource = XADataSourceFactory.build(databaseType, dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.xaTransactionManager = xaTransactionManager;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.registerRecoveryResource(resourceName, xaDataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50560">上述变量我们都认识，而在构造函数中，调用了 XATransactionManager 类中的 registerRecoveryResource 方法将构建的 XADataSource 作为一种资源进行注册。</p>
<p data-nodeid="50561">然后，我们来看 XATransactionDataSource 中的核心方法 getConnection，如下所示：</p>
<pre class="lang-java" data-nodeid="50562"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Connection <span class="hljs-title">getConnection</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException, SystemException, RollbackException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (CONTAINER_DATASOURCE_NAMES.contains(dataSource.getClass().getSimpleName())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//从DataSource中 构建一个 Connection</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection result = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过 XAConnectionFactory 创建一个 XAConnection</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; XAConnection xaConnection = XAConnectionFactory.createXAConnection(databaseType, xaDataSource, result);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//从 XATransactionManager 中获取 Transaction 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> Transaction transaction = xaTransactionManager.getTransactionManager().getTransaction();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判当前线程中是否存在这个 Transaction</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!enlistedTransactions.get().contains(transaction)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//将 XAConnection 中的 XAResource 与目标 Transaction 对象关联起来</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; transaction.enlistResource(<span class="hljs-keyword">new</span> SingleXAResource(resourceName, xaConnection.getXAResource()));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//Transaction 中注册一个 Synchronization 接口</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; transaction.registerSynchronization(<span class="hljs-keyword">new</span> Synchronization() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">beforeCompletion</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enlistedTransactions.get().remove(transaction);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">afterCompletion</span><span class="hljs-params">(<span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> status)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enlistedTransactions.get().clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//将该 Transaction 对象放入到当前线程中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enlistedTransactions.get().add(transaction);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="50563">这里先从 DataSource 中构建一个 Connection，然后传入到 XAConnectionFactory 中创建一个 XAConnection，接着从 XATransactionManager 中获取 Transaction 对象。请注意在 XATransactionDataSource 中存在一个 ThreadLocal 变量 enlistedTransactions，用于保存当前线程所涉及的 Transaction 对象列表：</p>
<pre class="lang-java" data-nodeid="50564"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;Set&lt;Transaction&gt;&gt; enlistedTransactions = <span class="hljs-keyword">new</span> ThreadLocal&lt;Set&lt;Transaction&gt;&gt;() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Set&lt;Transaction&gt; <span class="hljs-title">initialValue</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> HashSet&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
};
</code></pre>
<p data-nodeid="50565">在上述方法中，当从 XATransactionManager 中获取 Transaction 对象之后，会先判断 enlistedTransactions中 是否存在该 Transaction 对象，如果没有，则将 XAConnection 中的 XAResource 与目标 Transaction 对象关联起来。</p>
<p data-nodeid="50566">然后我们再来看 Transaction 对象的 registerSynchronization 方法的使用方法，该方法注册了一个 Synchronization 接口，该接口包含了 beforeCompletion 和 afterCompletion 这两个方法。</p>
<p data-nodeid="50567">在二阶段提交之前，TransctionManager 会调用 Synchronization 接口的 beforeCompletion 方法；而当事务结束时，TransctionManager 会调用 Synchronization 接口的 afterCompletion方法。我们在 getConnection 方法中看到这两个方法的应用。最终，我们把该 Transaction 对象放入到线程安全的 enlistedTransactions 中。</p>
<p data-nodeid="50568">最后，我们来看一下 XATransactionDataSource 的 close 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="50569"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!CONTAINER_DATASOURCE_NAMES.contains(dataSource.getClass().getSimpleName())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.removeRecoveryResource(resourceName, xaDataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; close(dataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="50570">可以看到，这里调用了 XATransactionManager 的 removeRecoveryResource 方法将资源进行移出。</p>
<p data-nodeid="50571">至此，基于 XATransactionDataSource 获取 Connection 的过程也介绍完毕。关于 XATransactionManager的 具体内容我们放在下一课时中继续进行探讨。</p>
<h3 data-nodeid="50572">从源码解析到日常开发</h3>
<p data-nodeid="50573">ShardingSphere 作为一款完全兼容 JDBC 规范的分布式数据库中间件，同样完成了针对分布式事务中的相关对象的兼容。今天的课程中，进一步强化了我们对 JDBC 规范的理解和如何扩展JDBC 规范中核心接口的方法。同时，在 MySQLXAConnectionWrapper 这个 Wrapper 类中，我们也再次看到使用反射技术创建 XAConnection 对象的实现方法。这些开发技巧都值得我们进行学习和应用。</p>
<h3 data-nodeid="50574">小结与预告</h3>
<p data-nodeid="50575">分布式事务是一个相对复杂的概念，ShardingSphere 中提供了强一致性和最终一致性两种实现方案。今天的内容我们围绕基于 XA 协议的分片事务管理器 XAShardingTransactionManager 展开了讨论，在理解 XAShardingTransactionManager 中 XADataSource、XAConnection 等核心对象时，重点还是需要站在 JDBC 规范的基础上，掌握与分布式事务集成和兼容的整个过程，本课时对这一过程进行了详细的介绍。</p>
<p data-nodeid="50576">这里给你留一道思考题：ShardingSphere 中对分布式环境下的强一致性事务做了哪些维度的抽象？欢迎你在留言区与大家讨论，我将逐一点评解答。</p>
<p data-nodeid="51152">XAShardingTransactionManager 的内容很多，下一课时，我们将在今天课时的基础上，继续探讨 XAShardingTransactionManager 的剩余部分内容，以及 ShardingSphere 中另一个分片事务管理器 SeataATShardingTransactionManager。</p>
<p data-nodeid="73099" class=""><a href="https://wj.qq.com/s2/7238084/d702/" data-nodeid="73103">课程评价入口，挑选 5 名小伙伴赠送小礼品~</a></p>

---

### 精选评论


