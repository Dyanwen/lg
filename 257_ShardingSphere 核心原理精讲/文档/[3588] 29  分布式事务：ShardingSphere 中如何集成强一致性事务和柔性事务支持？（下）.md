<p data-nodeid="6776" class="">在上一课时中，我们针对 ShardingSphere 中支持强一致性事务的 XAShardingTransactionManager 的部分内容进行了详细的展开，今天我们继续讲解该类的剩余内容，同时也会介绍支持柔性事务的 SeataATShardingTransactionManager。</p>
<h3 data-nodeid="6777">XAShardingTransactionManager</h3>
<p data-nodeid="6778">关于 XAShardingTransactionManager，上一讲中我们介绍了 XADataSource、XAConnection 和 XATransactionDataSource 等核心类。</p>
<p data-nodeid="6779">接下来，我们在上一讲的基础上给出 XATransactionManager 和 ShardingConnection 类的实现过程。</p>
<h4 data-nodeid="6780">1.XATransactionManager</h4>
<p data-nodeid="6781">让我们先回到 XAShardingTransactionManager。我们已经在前面介绍了 XAShardingTransactionManager 中的变量，接下来看一下它所实现的方法，首先是如下所示的 init 方法：</p>
<pre class="lang-java" data-nodeid="6782"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> Collection&lt;ResourceDataSource&gt; resourceDataSources)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (ResourceDataSource each : resourceDataSources) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建XATransactionDataSource并进行缓存</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cachedDataSources.put(each.getOriginalName(), <span class="hljs-keyword">new</span> XATransactionDataSource(databaseType, each.getUniqueResourceName(), each.getDataSource(), xaTransactionManager));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//初始化XATransactionManager</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.init();
}
</code></pre>
<p data-nodeid="6783">上述方法根据传入的 ResourceDataSource 构建了 XATransactionDataSource 并放入缓存中，同时对通过 SPI 机制创建的 XATransactionManager 也执行了它的 init 方法进行初始化。</p>
<p data-nodeid="6784">XAShardingTransactionManager 的 getTransactionType、isInTransaction 和 getConnection 方法都比较简单，如下所示：</p>
<pre class="lang-java" data-nodeid="6785"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> TransactionType <span class="hljs-title">getTransactionType</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> TransactionType.XA;
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isInTransaction</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Status.STATUS_NO_TRANSACTION != xaTransactionManager.getTransactionManager().getStatus();
}
	&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Connection <span class="hljs-title">getConnection</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> cachedDataSources.get(dataSourceName).getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> SystemException | RollbackException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> SQLException(ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6786">而与事务操作相关的 begin、commit 和 rollback 方法的实现同样比较简单，都是直接委托保存在 XATransactionManager 中的 TransactionManager 进行完成，如下所示：</p>
<pre class="lang-java" data-nodeid="6787"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">begin</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.getTransactionManager().begin();
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.getTransactionManager().commit();
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaTransactionManager.getTransactionManager().rollback();
}
</code></pre>
<p data-nodeid="6788">至此，sharding-transaction-xa-core 工程中的所有内容都已经介绍完毕。让我们转到 sharding-transaction-xa-atomikos-manager 工程，看看 AtomikosTransactionManager 的实现，这也是 ShardingSphere 中关于 TransactionManager 的默认实现。</p>
<p data-nodeid="6789">而在此之前，让我们先来看一下代表资源的 AtomikosXARecoverableResource 的实现，如下所示：</p>
<pre class="lang-java" data-nodeid="6790"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AtomikosXARecoverableResource</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JdbcTransactionalResource</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String resourceName;

&nbsp;&nbsp;&nbsp; AtomikosXARecoverableResource(<span class="hljs-keyword">final</span> String serverName, <span class="hljs-keyword">final</span> XADataSource xaDataSource) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(serverName, xaDataSource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resourceName = serverName;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">usesXAResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> XAResource xaResource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> resourceName.equals(((SingleXAResource) xaResource).getResourceName());
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6791">可以看到，这里的 usesXAResource 方法实际上就是通过基于对 SingleXAResource 的 ResourceName 进行比对来确定是否在使用资源，这也是为什么要设计包装了 XAResource 的 SingleXAResource 类的原因。</p>
<p data-nodeid="6792">AtomikosTransactionManager 中使用了 AtomikosXARecoverableResource，其实现过程如下所示：</p>
<pre class="lang-java" data-nodeid="6793"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AtomikosTransactionManager</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">XATransactionManager</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> UserTransactionManager transactionManager = <span class="hljs-keyword">new</span> UserTransactionManager();

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> UserTransactionService userTransactionService = <span class="hljs-keyword">new</span> UserTransactionServiceImp();

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userTransactionService.init();
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerRecoveryResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName, <span class="hljs-keyword">final</span> XADataSource xaDataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userTransactionService.registerResource(<span class="hljs-keyword">new</span> AtomikosXARecoverableResource(dataSourceName, xaDataSource));
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeRecoveryResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName, <span class="hljs-keyword">final</span> XADataSource xaDataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userTransactionService.removeResource(<span class="hljs-keyword">new</span> AtomikosXARecoverableResource(dataSourceName, xaDataSource));
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SneakyThrows</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">enlistResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SingleXAResource xaResource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; transactionManager.getTransaction().enlistResource(xaResource);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> TransactionManager <span class="hljs-title">getTransactionManager</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> transactionManager;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userTransactionService.shutdown(<span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6794">上述方法本质上都是对 Atomikos 的 UserTransactionManager 和 UserTransactionService 的简单调用，注意到 Atomikos 的 UserTransactionManager 实现了 TransactionManager 接口，封装了所有 TransactionManager 需要完成的工作。</p>
<p data-nodeid="6795">看完 sharding-transaction-xa-atomikos-manager 工程之后，我们来到另一个 sharding-transaction-xa-bitronix-manager 工程，该工程提供了基于 bitronix 的 XATransactionManager 实现方案，即 BitronixXATransactionManager 类：</p>
<pre class="lang-java" data-nodeid="6796"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BitronixXATransactionManager</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">XATransactionManager</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> BitronixTransactionManager bitronixTransactionManager = TransactionManagerServices.getTransactionManager();

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SneakyThrows</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerRecoveryResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName, <span class="hljs-keyword">final</span> XADataSource xaDataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResourceRegistrar.register(<span class="hljs-keyword">new</span> BitronixRecoveryResource(dataSourceName, xaDataSource));
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SneakyThrows</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeRecoveryResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName, <span class="hljs-keyword">final</span> XADataSource xaDataSource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResourceRegistrar.unregister(<span class="hljs-keyword">new</span> BitronixRecoveryResource(dataSourceName, xaDataSource));
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SneakyThrows</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">enlistResource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SingleXAResource singleXAResource)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bitronixTransactionManager.getTransaction().enlistResource(singleXAResource);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> TransactionManager <span class="hljs-title">getTransactionManager</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> bitronixTransactionManager;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bitronixTransactionManager.shutdown();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6797">对上述代码的理解也依赖与对 bitronix 框架的熟悉程度，整个封装过程简单明了。我们无意对 bitronix 框架做过多展开，而是更多关注于 ShardingSphere 中对 XATransactionManager 的抽象过程。</p>
<p data-nodeid="6798">作为总结，我们在上一课时的基础上，进一步梳理了 XA 两阶段提交相关的核心类之间的关系，如下图所示：</p>
<p data-nodeid="6799"><img src="https://s0.lgstatic.com/i/image/M00/53/AB/CgqCHl9oXe6AK8JkAAByEfwPBs0489.png" alt="image.png" data-nodeid="6871"></p>
<h4 data-nodeid="6800">2.ShardingConnection</h4>
<p data-nodeid="6801">上图展示了整个流程的源头是在 ShardingConnection 类。我们在 ShardingConnection 的构造函数中发现了创建 ShardingTransactionManager 的过程，如下所示：</p>
<pre class="lang-java" data-nodeid="6802"><code data-language="java">shardingTransactionManager = runtimeContext.getShardingTransactionManagerEngine().getTransactionManager(transactionType);
</code></pre>
<p data-nodeid="6803">在 ShardingConnection 的多处代码中都用到了上面所创建的 shardingTransactionManager 对象。例如，用于获取连接的 createConnection 方法：</p>
<pre class="lang-java" data-nodeid="6804"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> Connection <span class="hljs-title">createConnection</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName, <span class="hljs-keyword">final</span> DataSource dataSource)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> isInShardingTransaction() ? shardingTransactionManager.getConnection(dataSourceName) : dataSource.getConnection();
}
</code></pre>
<p data-nodeid="6805">用于判断是否是在同一个事务中的 isInShardingTransaction 方法：</p>
<pre class="lang-java" data-nodeid="6806"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isInShardingTransaction</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span> != shardingTransactionManager &amp;&amp; shardingTransactionManager.isInTransaction();
}
</code></pre>
<p data-nodeid="6807">以及如下所示的 setAutoCommit 方法完成了对 autoCommit 的处理：</p>
<pre class="lang-java" data-nodeid="6808"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setAutoCommit</span><span class="hljs-params">(<span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> autoCommit)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (TransactionType.LOCAL == transactionType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.setAutoCommit(autoCommit);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (autoCommit &amp;&amp; !shardingTransactionManager.isInTransaction() || !autoCommit &amp;&amp; shardingTransactionManager.isInTransaction()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (autoCommit &amp;&amp; shardingTransactionManager.isInTransaction()) {
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shardingTransactionManager.commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!autoCommit &amp;&amp; !shardingTransactionManager.isInTransaction()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; closeCachedConnections();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingTransactionManager.begin();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6809">在上述方法中，可以看到当事务类型为本地事务时，直接调用 ShardingConnection 的父类 AbstractConnectionAdapter 中的 setAutoCommit 方法完成本地事务的自动提交处理。</p>
<p data-nodeid="6810">而当 autoCommit 为 true 且运行在事务中时，会调用 shardingTransactionManager.commit() 方法完成提交；而当 autoCommit 为 false 且当前不在事务中时，会调用 shardingTransactionManager.begin() 方法启动事务。</p>
<p data-nodeid="6811">最后的 commit 和 rollback 的处理方式与 setAutoCommit 类似，都是根据事务类型来决定是否要进行分布式提交和回滚，如下所示：</p>
<pre class="lang-java" data-nodeid="6812"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (TransactionType.LOCAL == transactionType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingTransactionManager.commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (TransactionType.LOCAL == transactionType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.rollback();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; shardingTransactionManager.rollback();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6813">我们在上一课时中提到，ShardingSphere 在提供了两阶段提交的 XA 协议实现方案的同时，也实现了柔性事务。</p>
<p data-nodeid="6814">在介绍完 XAShardingTransactionManager 之后，我们继续来看基于 Seata 框架的柔性事务 TransactionManager 实现类 SeataATShardingTransactionManager。</p>
<h3 data-nodeid="6815">SeataATShardingTransactionManager</h3>
<p data-nodeid="6816">因为 SeataATShardingTransactionManager 完全采用阿里巴巴的 Seata 框架来提供分布式事务特性，而不是遵循类似 XA 这样的开发规范，所以在代码实现上比 XAShardingTransactionManager 的类层结构要简单很多，把复杂性都屏蔽在了框架的内部。</p>
<p data-nodeid="6817">要想集成 Seata，我们首先需要初始化 TMClient 和 RMClient 这两个客户端对象，在 Seata 内部，这两个客户端之间会基于 RPC 的方式进行通信。</p>
<p data-nodeid="6818">所以，ShardingSphere 在 XAShardingTransactionManager 中的 init 方法中实现了一个 initSeataRPCClient 方法来初始化这两个客户端对象，如下所示：</p>
<pre class="lang-java" data-nodeid="6819"><code data-language="java"><span class="hljs-comment">//根据 seata.conf 配置文件创建配置对象</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> FileConfiguration configuration = <span class="hljs-keyword">new</span> FileConfiguration(<span class="hljs-string">"seata.conf"</span>);
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initSeataRPCClient</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String applicationId = configuration.getConfig(<span class="hljs-string">"client.application.id"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Preconditions.checkNotNull(applicationId, <span class="hljs-string">"please config application id within seata.conf file"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String transactionServiceGroup = configuration.getConfig(<span class="hljs-string">"client.transaction.service.group"</span>, <span class="hljs-string">"default"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TMClient.init(applicationId, transactionServiceGroup);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RMClient.init(applicationId, transactionServiceGroup);
}
</code></pre>
<p data-nodeid="6820">回想我们在“09 | 分布式事务：如何使用强一致事务与柔性事务？”中关于 Seata 使用方式的介绍，不难理解这里通过 seata.conf 配置文件中所配置的 application.id 和 transaction.service.group 这两个配置项来执行初始化操作。</p>
<p data-nodeid="6821">同时，对于 Seata 而言，它也提供了一套构建在 JDBC 规范之上的实现策略，这点和“03 | 规范兼容：JDBC 规范与 ShardingSphere 是什么关系？”中介绍的 ShardingSphere 与 JDBC 规范之间的兼容性类似。</p>
<p data-nodeid="6822">而在命名上，Seata 更为直接明了，使用 DataSourceProxy 和 ConnectionProxy 这种代理对象。以 DataSourceProxy 为例，我们可以梳理它的类层结构如下：</p>
<p data-nodeid="6823"><img src="https://s0.lgstatic.com/i/image/M00/53/AB/CgqCHl9oXgKACi15AAA7sb7XKlo735.png" alt="image (1).png" data-nodeid="6903"></p>
<p data-nodeid="6824">可以看到 DataSourceProxy 实现了自己定义的 Resource 接口，然后继承了抽象类 AbstractDataSourceProxy，而后者则实现了 JDBC 中的 DataSource 接口。</p>
<p data-nodeid="6825">所以，在我们初始化 Seata 框架时，同样需要根据输入的 DataSource 对象来构建 DataSourceProxy，并通过 DataSourceProxy 获取 ConnectionProxy。SeataATShardingTransactionManager 类中的相关代码如下所示：</p>
<pre class="lang-java" data-nodeid="6826"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(<span class="hljs-keyword">final</span> DatabaseType databaseType, <span class="hljs-keyword">final</span> Collection&lt;ResourceDataSource&gt; resourceDataSources)</span> </span>{
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//初始化 Seata 客户端</span>
&nbsp;&nbsp;&nbsp;  initSeataRPCClient();
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//创建 DataSourceProxy 并放入到 Map 中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span> (ResourceDataSource each : resourceDataSources) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSourceMap.put(each.getOriginalName(), <span class="hljs-keyword">new</span> DataSourceProxy(each.getDataSource()));
&nbsp;&nbsp;&nbsp;&nbsp; }
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Connection <span class="hljs-title">getConnection</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据 DataSourceProxy 获取 ConnectionProxy</span>
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> dataSourceMap.get(dataSourceName).getConnection();
}
</code></pre>
<p data-nodeid="6827">介绍完初始化工作之后，我们来看 SeataATShardingTransactionManager 中提供了事务开启和提交相关的入口。在 Seata 中，GlobalTransaction 是一个核心接口，封装了面向用户操作层的分布式事务访问入口，该接口的定义如下所示，可以从方法命名上直接看出对应的操作含义：</p>
<pre class="lang-java" data-nodeid="6828"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">GlobalTransaction</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">begin</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">begin</span><span class="hljs-params">(<span class="hljs-keyword">int</span> timeout)</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">begin</span><span class="hljs-params">(<span class="hljs-keyword">int</span> timeout, String name)</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">GlobalStatus <span class="hljs-title">getStatus</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> TransactionException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getXid</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="6829">ShardingSphere 作为 GlobalTransaction 的用户层，同样基于 GlobalTransaction 接口来完成分布式事务操作。但 ShardingSphere 并没有直接使用这一层，而是设计了一个 SeataTransactionHolder 类，保存着线程安全的 GlobalTransaction 对象。</p>
<p data-nodeid="6830">SeataTransactionHolder 类位于 sharding-transaction-base-seata-at 工程中，定义如下：</p>
<pre class="lang-java" data-nodeid="6831"><code data-language="java"><span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SeataTransactionHolder</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;GlobalTransaction&gt; CONTEXT = <span class="hljs-keyword">new</span> ThreadLocal&lt;&gt;();

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">set</span><span class="hljs-params">(<span class="hljs-keyword">final</span> GlobalTransaction transaction)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CONTEXT.set(transaction);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">static</span> GlobalTransaction <span class="hljs-title">get</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> CONTEXT.get();
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">clear</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CONTEXT.remove();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="6832">可以看到这里使用了 ThreadLocal 工具类来确保对 GlobalTransaction 访问的线程安全性。</p>
<p data-nodeid="6833">接下来的问题是，如何判断当前操作是否处于一个全局事务中呢？</p>
<p data-nodeid="6834">在 Seata 中，存在一个上下文对象 RootContex，该类就是用来保存参与者和发起者之间传播的 Xid。当事务发起者开启全局事务后，会将 Xid 填充到 RootContext 里；然后 Xid 将沿着服务调用链一直传播，进而填充到每个事务参与者进程的 RootContext 里；事务参与者发现 RootContext 中存在 Xid 时，就可以知道自己处于全局事务中。</p>
<p data-nodeid="6835">基于这层原理，我们只需要采用如下所示的判断方法就能得出是否处于全局事务中的结论：</p>
<pre class="lang-java" data-nodeid="6836"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isInTransaction</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span> != RootContext.getXID();
}
</code></pre>
<p data-nodeid="6837">同时，Seata 也提供了一个针对全局事务的上下文类 GlobalTransactionContext，通过这个上下文类，我们可以使用 getCurrent 方法来获取一个 GlobalTransaction对象，或者通过 getCurrentOrCreate 方法在无法获取 GlobalTransaction 对象时新建一个。</p>
<p data-nodeid="6838">讲到这里，我们就不难理解 SeataATShardingTransactionManager 中 begin 方法的实现过程了，如下所示：</p>
<pre class="lang-java" data-nodeid="6839"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-meta">@SneakyThrows</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">begin</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.set(GlobalTransactionContext.getCurrentOrCreate());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.get().begin();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionBroadcaster.collectGlobalTxId();
}
</code></pre>
<p data-nodeid="6840">这里通过 GlobalTransactionContext.getCurrentOrCreate() 方法创建了一个 GlobalTransaction，然后将其保存到了 SeataTransactionHolder 中。接着从 SeataTransactionHolder 中获取一个 GlobalTransaction，并调用 begin 方法启动事务。</p>
<p data-nodeid="6841">注意到这里还有一个 SeataTransactionBroadcaster 类，该类就是用来保存 Seata 全局 Xid 的一个容器类。我们会在事务启动时收集全局 Xid 并进行保存，而在事务提交或回滚时清空这些 Xid。</p>
<p data-nodeid="6842">所以，如下所示的 commit、rollback 和 close 方法的实现过程就都变得容易理解了：</p>
<pre class="lang-java" data-nodeid="6843"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">commit</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.get().commit();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionBroadcaster.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">rollback</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.get().rollback();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionBroadcaster.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSourceMap.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SeataTransactionHolder.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TmRpcClient.getInstance().destroy();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RmRpcClient.getInstance().destroy();
}
</code></pre>
<p data-nodeid="6844">sharding-transaction-base-seata-at 工程中的代码实际上就只有这些内容，这些内容也构成了在 ShardingSphere中 集成 Seata 框架的实现过程。</p>
<h3 data-nodeid="6845">从源码解析到日常开发</h3>
<p data-nodeid="6846">今天的内容给出了在应用程序中如何集成 Seata 分布式事务框架的详细过程，ShardingSphere 为我们提供了一种模版实现。在日常开发过程中，如果我们想要在业务代码中集成 Seata，就可以参考 SeataTransactionHolder、SeataATShardingTransactionManager 等核心类中的代码，而不需要做太多的修改。</p>
<h3 data-nodeid="6847">小结与预告</h3>
<p data-nodeid="6848">本课时是 ShardingSphere 分布式事务的最后一讲，我们介绍完了 XAShardingTransactionManager 的剩余部分内容，以及 SeataATShardingTransactionManager 的完整实现。</p>
<p data-nodeid="6849">回顾上一课时内容，我们发现理解 XAShardingTransactionManager 的难点在于，从 ShardingConnection 到底层 JDBC 规范的整个集成和兼容过程。而对于 XAShardingTransactionManager 而言，我们需要对 Seata 框架本身有一定的了解，才能更好地理解今天的内容。</p>
<p data-nodeid="6850">这里给你留一道思考题：如果让你实现对 Seata 框架的集成，你需要做哪些核心步骤？欢迎你在留言区与大家讨论，我将逐一点评解答。</p>
<p data-nodeid="7230">介绍完分布式事务之后，我们将进入“ShardingSphere 中编排治理方面的源码解析”模块，从下一课时开始，我将要介绍数据脱敏模块的实现原理。</p>
<p data-nodeid="7231" class=""><a href="https://wj.qq.com/s2/7238084/d702/" data-nodeid="7236">课程评价入口，挑选 5 名小伙伴赠送小礼品~</a></p>

---

### 精选评论


