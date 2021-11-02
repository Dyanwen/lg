<p data-nodeid="45916">你好，欢迎来到第 20 讲。前面我们已经学习完了两个模块：基础知识以及高阶用法与实战的内容，不知道你掌握得如何，有疑问的地方一定要留言提问，或者和大家一起讨论，请记住学习的路上你不是一个人在战斗。</p>


<p data-nodeid="45267">那么从这一讲开始，我们进入“模块三：原理与问题排查”知识的学习。这一模块，我将带你了解Hibernate 的加载过程、Session 和事务之间的关系，帮助你知道在遇到 LazyException 以及经典的 N+1 SQL 问题时该如何解决，希望你在工作中可以灵活运用所学知识。</p>
<p data-nodeid="45268">这一讲，我们来分析一下在 Spring Data JPA 的项目下面 Hibernate 的配置参数有哪些，先从 Hibernate 的整体架构进行分析。</p>
<h3 data-nodeid="45269">Hibernate 架构分析</h3>
<p data-nodeid="46770">首先看一下 Hibernate 5.2 版本中，官方提供的架构图。</p>
<p data-nodeid="46771" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08OiAT8tUAAAtx02IC70594.png" alt="Drawing 0.png" data-nodeid="46775"></p>


<p data-nodeid="45272">从架构图上，我们可以知道 Hiberante 实现的 ORM 的接口有两种，一种是 Hiberante 自己的 API 接口；一种是 Java Persistence API 的接口实现。</p>
<p data-nodeid="45273">因为 Hibernate 其实是比 Java Persistence API 早几年发展的，后来才有了 Java 的持久化协议。以我个人的观点来看，随着时间的推移，Hiberante 的实现逻辑可能会逐渐被弱化，由 Java Persistence API 统一对外提供服务。</p>
<p data-nodeid="47628">那么有了这个基础，我们研究 Hibernate 在 Spring Data JPA 里面的作用，得出的结论就是：Hibernate 5.2 是 Spring Data JPA 持久化操作的核心。我们再从类上面具体看一下，关键类的图如下所示：</p>
<p data-nodeid="47629" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08PGACMAWAABkBYWN3EQ292.png" alt="Drawing 1.png" data-nodeid="47633"></p>


<p data-nodeid="45276">结合类的关系图来看，Session 接口和 SessionFactory 接口都是 Hibernate 的概念，而 EntityManger 和 EntityManagerFactory 都是 Java Persistence API 协议规定的接口。</p>
<p data-nodeid="45277">不过 HibernateEntityManger 从 Hibernate 5.2 之后就开始不推荐使用了，而是建议直接使用 EntityManager 接口即可。那么我们看看 Hibernate 在 Spring BOOT 里面是如何被加载进去的。</p>
<h3 data-nodeid="45278">Hibernate 5 在 Spring Boot 2 里面的加载过程</h3>
<p data-nodeid="48486">不同的 Spring Boot 版本，可能加载类的实现逻辑是不一样的，但是分析过程都是相同的。我们先打开 spring.factories 文件，如下图所示，其中可以自动加载 Hibernate 的只有一个类，那就是 HibernateJpaAutoConfiguration。</p>
<p data-nodeid="48487" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/3A/Ciqc1F-08PqAOGq1AAPvvo_ZD7w314.png" alt="Drawing 2.png" data-nodeid="48491"></p>


<p data-nodeid="45281">HibernateJpaAutoConfiguration 就是 Spring Boot 加载 Hibernate 的主要入口，所以我们可以直接打开这个类看一下。</p>
<pre class="lang-java" data-nodeid="45282"><code data-language="java"><span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
<span class="hljs-meta">@ConditionalOnClass({ LocalContainerEntityManagerFactoryBean.class, EntityManager.class, SessionImplementor.class })</span>
<span class="hljs-meta">@EnableConfigurationProperties(JpaProperties.class)</span><span class="hljs-comment">//JPAProperties的配置</span>
<span class="hljs-meta">@AutoConfigureAfter({ DataSourceAutoConfiguration.class })</span>
<span class="hljs-meta">@Import(HibernateJpaConfiguration.class)</span> <span class="hljs-comment">//hibernate加载的关键类</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HibernateJpaAutoConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="45283">其中，我们第一个需要关注的就是 JpaProperties 类，因为通过这个类我们可以间接知道，application.properties 里面可以配置的 spring.jpa 的属性有哪些。</p>
<h4 data-nodeid="45284">JpaProperties 属性</h4>
<p data-nodeid="49344">我们打开 JpaProperties 类看一下，如下图所示。</p>
<p data-nodeid="49345" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/3A/Ciqc1F-08Q6AR-PoAAGybbuAJ7g272.png" alt="Drawing 3.png" data-nodeid="49349"></p>


<p data-nodeid="45287">通过这个类，我们可以在 application.properties 里面得到如下配置项。</p>
<pre class="lang-yaml" data-nodeid="45288"><code data-language="yaml"><span class="hljs-comment"># 可以配置JPA的实现者的原始属性的配置，如：这里我们用的JPA的实现者是hibernate</span>
<span class="hljs-comment"># 那么hibernate里面的一些属性设置就可以通过如下方式实现，具体properties里面有哪些，本讲会详细介绍，我们先知道这里可以设置即可</span>
<span class="hljs-string">spring.jpa.properties.hibernate.hbm2ddl.auto=none</span>
<span class="hljs-comment">#hibernate的persistence.xml文件有哪些，目前已经不推荐使用</span>
<span class="hljs-comment">#spring.jpa.mapping-resources=</span>
<span class="hljs-comment"># 指定数据源的类型，如果不指定，Spring Boot加载Datasource的时候会根据URL的协议自己判断</span>
<span class="hljs-comment"># 如：spring.datasource.url=jdbc:mysql://localhost:3306/test 上面可以明确知道是mysql数据源，所以这个可以不需要指定；</span>
<span class="hljs-comment"># 应用场景，当我们通过代理的方式，可能通过datasource.url没办法判断数据源类型的时候，可以通过如下方式指定，可选的值有：DB2,H2,HSQL,INFORMIX,MYSQL,ORACLE,POSTGRESQL,SQL_SERVER,SYBASE)</span>
<span class="hljs-string">spring.jpa.database=mysql</span>
<span class="hljs-comment"># 是否在启动阶段根据实体初始化数据库的schema，默认false，当我们用内存数据库做测试的时候可以打开，很有用</span>
<span class="hljs-string">spring.jpa.generate-ddl=false</span>
<span class="hljs-comment"># 和spring.jpa.database用法差不多，指定数据库的平台，默认会自己发现；一般不需要指定，database-platform指定的必须是org.hibernate.dialect.Dialect的子类，如mysql默认是用下面的platform</span>
<span class="hljs-string">spring.jpa.database-platform=org.hibernate.dialect.MySQLInnoDBDialect</span>
<span class="hljs-comment"># 是否在view层打开session，默认是true，其实大部分场景不需要打开，我们可以设置成false，</span>
<span class="hljs-comment"># 22课时我们再详细讲解</span>
<span class="hljs-string">spring.jpa.open-in-view=false</span>
<span class="hljs-comment"># 是否显示sql，当执行JPA的数据库操作的时候，默认是false，在本地开发的时候我们可以把这个打开，有助于分析sql是不是我们预期的</span>
<span class="hljs-comment"># 在生产环境的时候建议给这个设置成false，改由logging.level.org.hibernate.SQL=DEBUG代替，这样的话日志默认是基于logback输出的</span>
<span class="hljs-comment"># 而不是直接打印到控制台的，有利于增加traceid和线程ID等信息，便于分析</span>
<span class="hljs-string">spring.jpa.show-sql=true</span>
</code></pre>
<p data-nodeid="45289">其中，spring.jpa.show-sql=true 输出的 sql 效果如下所示。</p>
<pre class="lang-sql" data-nodeid="45290"><code data-language="sql">Hibernate: <span class="hljs-keyword">insert</span> <span class="hljs-keyword">into</span> user_info (create_time, create_user_id, last_modified_time, last_modified_user_id, <span class="hljs-keyword">version</span>, ages, email_address, last_name, telephone, <span class="hljs-keyword">id</span>) <span class="hljs-keyword">values</span> (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)
</code></pre>
<p data-nodeid="45291">上面是孤立无援的 System.out.println 的效果，如果是在线上环境，多线程的情况下就不知道是哪个线程输出来的，而 logging.level.org.hibernate.SQL=DEBUG 输出的 sql 效果如下所示。</p>
<pre class="lang-sql" data-nodeid="45292"><code data-language="sql">2020-11-08 16:54:22.275 DEBUG 6589 <span class="hljs-comment">--- [nio-8087-exec-1] org.hibernate.SQL                        : insert into user_info (create_time, create_user_id, last_modified_time, last_modified_user_id, version, ages, email_address, last_name, telephone, id) values (?, ?, ?, ?, ?, ?, ?, ?, ?, ?)</span>
</code></pre>
<p data-nodeid="45293">这样我们可以轻易知道线程 ID 和执行时间，甚至可以有 tranceID 和 spanID 进行日志跟踪，方便分析是哪个线程打印的。</p>
<p data-nodeid="45294">我们了解完了 JpaProperties，下面再看另外一个关键类 HibernateJpaConfiguration，它也是 HibernateJpaAutoConfiguration 导入进来加载的。</p>
<h4 data-nodeid="45295">HibernateJpaConfiguration 分析</h4>
<p data-nodeid="45296">我们通过上述 HibernateJpaAutoConfiguration 里面的 @Import(HibernateJpaConfiguration.class)，打开 HibernateJpaConfiguration.class 看看是什么情况。</p>
<pre class="lang-java" data-nodeid="45297"><code data-language="java"><span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
<span class="hljs-meta">@EnableConfigurationProperties(HibernateProperties.class)</span>
<span class="hljs-meta">@ConditionalOnSingleCandidate(DataSource.class)</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HibernateJpaConfiguration</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaBaseConfiguration</span> </span>{
.......<span class="hljs-comment">//其他我们暂不关心的代码我们可以先省略}</span>
</code></pre>
<p data-nodeid="50638">通过源码我们可以得到 Hibernate 在 JPA 中配置的三个重要线索，下面详细说明。</p>
<p data-nodeid="50639"><strong data-nodeid="50644">第一个线索：HibernatePropertes 这个配置类对应的是 spring.jpa.hibernate 的配置。</strong></p>

<p data-nodeid="50202">我们通过源码可以看得出来，@EnableConfigurationProperties(HibernateProperties.class) 启用了 HibernatePropertes 的配置类，如下图所示。</p>
<p data-nodeid="50203" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08RuALMNSAAKTACKebbE349.png" alt="Drawing 4.png" data-nodeid="50207"></p>


<p data-nodeid="45301">其中可以看到 application.properties 的配置项，如下所示。</p>
<pre class="lang-yaml" data-nodeid="45302"><code data-language="yaml"><span class="hljs-comment"># 正如我们之前课时讲到的nameing的物理策略值有：org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy(默认)和org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl</span>
<span class="hljs-string">spring.jpa.hibernate.naming.physical-strategy=</span>
<span class="hljs-comment"># ddl的生成策略，默认none；如果我们没有指定任何数据源的url，采用的是spring的集成数据源，也就是内存数据源H2的时候，默认值是create-drop；</span>
<span class="hljs-comment"># 所以你会发现当我们每次用H2的时候什么都没做，它就会自动帮我们创建表等，内存数据库和写测试用的时候，create-drop就非常方便了；不过，当我们生产数据库的时候一定要设置成none;</span>
<span class="hljs-string">spring.jpa.hibernate.ddl-auto=none</span>
<span class="hljs-comment"># 当我们的@Id配置成@GeneratedValue(strategy= GenerationType.AUTO)的时候是否采用hibernate的Id-generator-mappings(即会默认帮我们创建一张表hibernate_sequence来存储和生成ID)，默认是true</span>
<span class="hljs-string">spring.jpa.hibernate.use-new-id-generator-mappings=true</span>
</code></pre>
<p data-nodeid="45303"><strong data-nodeid="45417">第二个线索：通过源码我们还可以看得出来，HibernateJpaConfiguration 的父类 JpaBaseConfiguration 也会优先加载，此类就是 Spring Boot 加载 JPA 的核心逻辑。</strong></p>
<p data-nodeid="45304">那么我们打开 JpaBaseConfiguration 类看一下源码。</p>
<pre class="lang-java" data-nodeid="45305"><code data-language="java"><span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
<span class="hljs-meta">@EnableConfigurationProperties(JpaProperties.class)</span>
<span class="hljs-comment">//DataSourceInitializedPublisher用来进行数据源的初始化操作</span>
<span class="hljs-meta">@Import(DataSourceInitializedPublisher.Registrar.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JpaBaseConfiguration</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">BeanFactoryAware</span> </span>{
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">JpaBaseConfiguration</span><span class="hljs-params">(DataSource dataSource, JpaProperties properties,
      ObjectProvider&lt;JtaTransactionManager&gt; jtaTransactionManager)</span> </span>{
   <span class="hljs-keyword">this</span>.dataSource = dataSource;
   <span class="hljs-keyword">this</span>.properties = properties;
   <span class="hljs-comment">//jtaTransactionManager赋值，正常情况下我们用不到，一般用来解决分布式事务的场景才会用到。</span>
   <span class="hljs-keyword">this</span>.jtaTransactionManager = jtaTransactionManager.getIfAvailable(); 
}
<span class="hljs-comment">//加载JPA的实现方式</span>
<span class="hljs-meta">@Bean</span>
<span class="hljs-meta">@ConditionalOnMissingBean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> JpaVendorAdapter <span class="hljs-title">jpaVendorAdapter</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-comment">//createJpaVendorAdapter是由子类HibernateJpaConfiguration实现的，创建JPA的实现类</span>
   AbstractJpaVendorAdapter adapter = createJpaVendorAdapter();
   adapter.setShowSql(<span class="hljs-keyword">this</span>.properties.isShowSql());
   <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.properties.getDatabase() != <span class="hljs-keyword">null</span>) {
      adapter.setDatabase(<span class="hljs-keyword">this</span>.properties.getDatabase());
   }
   <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.properties.getDatabasePlatform() != <span class="hljs-keyword">null</span>) {
   adapter.setDatabasePlatform(<span class="hljs-keyword">this</span>.properties.getDatabasePlatform());
   }
   adapter.setGenerateDdl(<span class="hljs-keyword">this</span>.properties.isGenerateDdl());
   <span class="hljs-keyword">return</span> adapter;
}
.......其他我们暂时不关心的代码先省略}
</code></pre>
<p data-nodeid="45306">我们从上面的源码中可以看到，@Import(DataSourceInitializedPublisher.Registrar.class) 是用来初始化数据的；从构造函数中我们也可以看到其是否有用到 jtaTransactionManager（这个是分布式事务才会用到）；而 createJpaVendorAdapter() 是在 HibernateJpaConfiguration 里面实现的，这个要重点说一下，关键代码如下。</p>
<pre class="lang-java" data-nodeid="45307"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HibernateJpaConfiguration</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaBaseConfiguration</span> </span>{
<span class="hljs-comment">//这里是hibernate和Jpa的结合，可以看到使用的HibernateJpaVendorAdapter作为JPA的实现者，感兴趣的话你可以打开HibernateJpaVendorAdapter里面设置一些断点，就会知道Spring boot是如何一步一步加载Hibernate的了；</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> AbstractJpaVendorAdapter <span class="hljs-title">createJpaVendorAdapter</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> HibernateJpaVendorAdapter();
}
.......}
</code></pre>
<p data-nodeid="45308">现在我们知道了 HibernateJpaVendorAdapter 的加载逻辑，而 HibernateJpaVendorAdapter 里面实现了 Hibernate 的初始化逻辑，我在这里不多说了，你过后可以仔细 debug 看一下，基本上就是 Hibernate 5.2 官方的加载逻辑。那么 Hibernate Jpa 对应的原始配置有哪些呢？</p>
<p data-nodeid="45309"><strong data-nodeid="45424">第三个线索：spring.jpa.properties 配置项有哪些？</strong></p>
<p data-nodeid="51497">我们如果接着在 HibernateJpaConfiguration 类里面 debug 查看关键代码的话，可以找到如下代码。</p>
<p data-nodeid="52154"><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08SmAc9J8AARC3m5NP2o468.png" alt="Drawing 5.png" data-nodeid="52158"></p>
<p data-nodeid="52155">上图中的代码显示，JpaProperties 类里面的 properties 属性，也就是 spring.jpa.properties 的配置加载到了 vendorProperties（即 Hibernate 5.2）里面。而 properties 里面是 HashMap 结构，那么它都可以支持哪些配置呢？</p>






<p data-nodeid="45313">我们打开 org.hibernate.cfg.AvailableSettings 可以看到 Hibernate 支持的配置项大概有 100 多个配置信息，如下所示。</p>
<pre class="lang-java" data-nodeid="45314"><code data-language="java">String JPA_PERSISTENCE_PROVIDER = <span class="hljs-string">"javax.persistence.provider"</span>;
String JPA_TRANSACTION_TYPE = <span class="hljs-string">"javax.persistence.transactionType"</span>;
String JPA_JTA_DATASOURCE = <span class="hljs-string">"javax.persistence.jtaDataSource"</span>;
String JPA_NON_JTA_DATASOURCE = <span class="hljs-string">"javax.persistence.nonJtaDataSource"</span>;
String JPA_JDBC_DRIVER = <span class="hljs-string">"javax.persistence.jdbc.driver"</span>;
String JPA_JDBC_URL = <span class="hljs-string">"javax.persistence.jdbc.url"</span>;
String JPA_JDBC_USER = <span class="hljs-string">"javax.persistence.jdbc.user"</span>;
String JPA_JDBC_PASSWORD = <span class="hljs-string">"javax.persistence.jdbc.password"</span>;
String JPA_SHARED_CACHE_MODE = <span class="hljs-string">"javax.persistence.sharedCache.mode"</span>;
String JPA_SHARED_CACHE_RETRIEVE_MODE =<span class="hljs-string">"javax.persistence.cache.retrieveMode"</span>;
String JPA_SHARED_CACHE_STORE_MODE =<span class="hljs-string">"javax.persistence.cache.storeMode"</span>;
String JPA_VALIDATION_MODE = <span class="hljs-string">"javax.persistence.validation.mode"</span>;
String JPA_VALIDATION_FACTORY = <span class="hljs-string">"javax.persistence.validation.factory"</span>;
String JPA_PERSIST_VALIDATION_GROUP = <span class="hljs-string">"javax.persistence.validation.group.pre-persist"</span>;
String JPA_UPDATE_VALIDATION_GROUP = <span class="hljs-string">"javax.persistence.validation.group.pre-update"</span>;
String JPA_REMOVE_VALIDATION_GROUP = <span class="hljs-string">"javax.persistence.validation.group.pre-remove"</span>;
String JPA_LOCK_SCOPE = <span class="hljs-string">"javax.persistence.lock.scope"</span>;
String JPA_LOCK_TIMEOUT = <span class="hljs-string">"javax.persistence.lock.timeout"</span>;
String CDI_BEAN_MANAGER = <span class="hljs-string">"javax.persistence.bean.manager"</span>;
String CLASSLOADERS = <span class="hljs-string">"hibernate.classLoaders"</span>;
String TC_CLASSLOADER = <span class="hljs-string">"hibernate.classLoader.tccl_lookup_precedence"</span>;
String APP_CLASSLOADER = <span class="hljs-string">"hibernate.classLoader.application"</span>;
String RESOURCES_CLASSLOADER = <span class="hljs-string">"hibernate.classLoader.resources"</span>;
String HIBERNATE_CLASSLOADER = <span class="hljs-string">"hibernate.classLoader.hibernate"</span>;
String ENVIRONMENT_CLASSLOADER = <span class="hljs-string">"hibernate.classLoader.environment"</span>;
String JPA_METAMODEL_GENERATION = <span class="hljs-string">"hibernate.ejb.metamodel.generation"</span>;
String JPA_METAMODEL_POPULATION = <span class="hljs-string">"hibernate.ejb.metamodel.population"</span>;
String STATIC_METAMODEL_POPULATION = <span class="hljs-string">"hibernate.jpa.static_metamodel.population"</span>;
String CONNECTION_PROVIDER =<span class="hljs-string">"hibernate.connection.provider_class"</span>;
String DRIVER =<span class="hljs-string">"hibernate.connection.driver_class"</span>;
String URL =<span class="hljs-string">"hibernate.connection.url"</span>;
String USER =<span class="hljs-string">"hibernate.connection.username"</span>;
String PASS =<span class="hljs-string">"hibernate.connection.password"</span>;
String ISOLATION =<span class="hljs-string">"hibernate.connection.isolation"</span>;
String AUTOCOMMIT = <span class="hljs-string">"hibernate.connection.autocommit"</span>;
String POOL_SIZE =<span class="hljs-string">"hibernate.connection.pool_size"</span>;
String DATASOURCE =<span class="hljs-string">"hibernate.connection.datasource"</span>;
String CONNECTION_PROVIDER_DISABLES_AUTOCOMMIT= <span class="hljs-string">"hibernate.connection.provider_disables_autocommit"</span>;
String CONNECTION_PREFIX = <span class="hljs-string">"hibernate.connection"</span>;
String JNDI_CLASS =<span class="hljs-string">"hibernate.jndi.class"</span>;
String JNDI_URL =<span class="hljs-string">"hibernate.jndi.url"</span>;
String JNDI_PREFIX = <span class="hljs-string">"hibernate.jndi"</span>;
String DIALECT =<span class="hljs-string">"hibernate.dialect"</span>;
String DIALECT_RESOLVERS = <span class="hljs-string">"hibernate.dialect_resolvers"</span>;
String STORAGE_ENGINE = <span class="hljs-string">"hibernate.dialect.storage_engine"</span>;
String SCHEMA_MANAGEMENT_TOOL = <span class="hljs-string">"hibernate.schema_management_tool"</span>;
String TRANSACTION_COORDINATOR_STRATEGY = <span class="hljs-string">"hibernate.transaction.coordinator_class"</span>;
String JTA_PLATFORM = <span class="hljs-string">"hibernate.transaction.jta.platform"</span>;
String PREFER_USER_TRANSACTION = <span class="hljs-string">"hibernate.jta.prefer_user_transaction"</span>;
String JTA_PLATFORM_RESOLVER = <span class="hljs-string">"hibernate.transaction.jta.platform_resolver"</span>;
String JTA_CACHE_TM = <span class="hljs-string">"hibernate.jta.cacheTransactionManager"</span>;
String JTA_CACHE_UT = <span class="hljs-string">"hibernate.jta.cacheUserTransaction"</span>;
String JDBC_TYLE_PARAMS_ZERO_BASE = <span class="hljs-string">"hibernate.query.sql.jdbc_style_params_base"</span>;
String DEFAULT_CATALOG = <span class="hljs-string">"hibernate.default_catalog"</span>;
String DEFAULT_SCHEMA = <span class="hljs-string">"hibernate.default_schema"</span>;
String DEFAULT_CACHE_CONCURRENCY_STRATEGY = <span class="hljs-string">"hibernate.cache.default_cache_concurrency_strategy"</span>;
String USE_NEW_ID_GENERATOR_MAPPINGS = <span class="hljs-string">"hibernate.id.new_generator_mappings"</span>;
String FORCE_DISCRIMINATOR_IN_SELECTS_BY_DEFAULT = <span class="hljs-string">"hibernate.discriminator.force_in_select"</span>;
String IMPLICIT_DISCRIMINATOR_COLUMNS_FOR_JOINED_SUBCLASS = <span class="hljs-string">"hibernate.discriminator.implicit_for_joined"</span>;
String IGNORE_EXPLICIT_DISCRIMINATOR_COLUMNS_FOR_JOINED_SUBCLASS = <span class="hljs-string">"hibernate.discriminator.ignore_explicit_for_joined"</span>;
String USE_NATIONALIZED_CHARACTER_DATA = <span class="hljs-string">"hibernate.use_nationalized_character_data"</span>;
String SCANNER_DEPRECATED = <span class="hljs-string">"hibernate.ejb.resource_scanner"</span>;
String SCANNER = <span class="hljs-string">"hibernate.archive.scanner"</span>;
String SCANNER_ARCHIVE_INTERPRETER = <span class="hljs-string">"hibernate.archive.interpreter"</span>;
String SCANNER_DISCOVERY = <span class="hljs-string">"hibernate.archive.autodetection"</span>;
String IMPLICIT_NAMING_STRATEGY = <span class="hljs-string">"hibernate.implicit_naming_strategy"</span>;
String PHYSICAL_NAMING_STRATEGY = <span class="hljs-string">"hibernate.physical_naming_strategy"</span>;
String ARTIFACT_PROCESSING_ORDER = <span class="hljs-string">"hibernate.mapping.precedence"</span>;
String KEYWORD_AUTO_QUOTING_ENABLED = <span class="hljs-string">"hibernate.auto_quote_keyword"</span>;
String XML_MAPPING_ENABLED = <span class="hljs-string">"hibernate.xml_mapping_enabled"</span>;
String SESSION_FACTORY_NAME = <span class="hljs-string">"hibernate.session_factory_name"</span>;
String SESSION_FACTORY_NAME_IS_JNDI = <span class="hljs-string">"hibernate.session_factory_name_is_jndi"</span>;
String SHOW_SQL =<span class="hljs-string">"hibernate.show_sql"</span>;
String FORMAT_SQL =<span class="hljs-string">"hibernate.format_sql"</span>;
String USE_SQL_COMMENTS =<span class="hljs-string">"hibernate.use_sql_comments"</span>;
String MAX_FETCH_DEPTH = <span class="hljs-string">"hibernate.max_fetch_depth"</span>;
String DEFAULT_BATCH_FETCH_SIZE = <span class="hljs-string">"hibernate.default_batch_fetch_size"</span>;
String USE_STREAMS_FOR_BINARY = <span class="hljs-string">"hibernate.jdbc.use_streams_for_binary"</span>;
String USE_SCROLLABLE_RESULTSET = <span class="hljs-string">"hibernate.jdbc.use_scrollable_resultset"</span>;
String USE_GET_GENERATED_KEYS = <span class="hljs-string">"hibernate.jdbc.use_get_generated_keys"</span>;
String STATEMENT_FETCH_SIZE = <span class="hljs-string">"hibernate.jdbc.fetch_size"</span>;
String STATEMENT_BATCH_SIZE = <span class="hljs-string">"hibernate.jdbc.batch_size"</span>;
String BATCH_STRATEGY = <span class="hljs-string">"hibernate.jdbc.factory_class"</span>;
String BATCH_VERSIONED_DATA = <span class="hljs-string">"hibernate.jdbc.batch_versioned_data"</span>;
String JDBC_TIME_ZONE = <span class="hljs-string">"hibernate.jdbc.time_zone"</span>;
String AUTO_CLOSE_SESSION = <span class="hljs-string">"hibernate.transaction.auto_close_session"</span>;
String FLUSH_BEFORE_COMPLETION = <span class="hljs-string">"hibernate.transaction.flush_before_completion"</span>;
String ACQUIRE_CONNECTIONS = <span class="hljs-string">"hibernate.connection.acquisition_mode"</span>;
String RELEASE_CONNECTIONS = <span class="hljs-string">"hibernate.connection.release_mode"</span>;
String CONNECTION_HANDLING = <span class="hljs-string">"hibernate.connection.handling_mode"</span>;
String CURRENT_SESSION_CONTEXT_CLASS = <span class="hljs-string">"hibernate.current_session_context_class"</span>;
String USE_IDENTIFIER_ROLLBACK = <span class="hljs-string">"hibernate.use_identifier_rollback"</span>;
String USE_REFLECTION_OPTIMIZER = <span class="hljs-string">"hibernate.bytecode.use_reflection_optimizer"</span>;
String ENFORCE_LEGACY_PROXY_CLASSNAMES = <span class="hljs-string">"hibernate.bytecode.enforce_legacy_proxy_classnames"</span>;
String ALLOW_ENHANCEMENT_AS_PROXY = <span class="hljs-string">"hibernate.bytecode.allow_enhancement_as_proxy"</span>;
String QUERY_TRANSLATOR = <span class="hljs-string">"hibernate.query.factory_class"</span>;
String QUERY_SUBSTITUTIONS = <span class="hljs-string">"hibernate.query.substitutions"</span>;
String QUERY_STARTUP_CHECKING = <span class="hljs-string">"hibernate.query.startup_check"</span>;
String CONVENTIONAL_JAVA_CONSTANTS = <span class="hljs-string">"hibernate.query.conventional_java_constants"</span>;
String SQL_EXCEPTION_CONVERTER = <span class="hljs-string">"hibernate.jdbc.sql_exception_converter"</span>;
String WRAP_RESULT_SETS = <span class="hljs-string">"hibernate.jdbc.wrap_result_sets"</span>;
String NATIVE_EXCEPTION_HANDLING_51_COMPLIANCE = <span class="hljs-string">"hibernate.native_exception_handling_51_compliance"</span>;
String ORDER_UPDATES = <span class="hljs-string">"hibernate.order_updates"</span>;
String ORDER_INSERTS = <span class="hljs-string">"hibernate.order_inserts"</span>;
String JPA_CALLBACKS_ENABLED = <span class="hljs-string">"hibernate.jpa_callbacks.enabled"</span>;
String DEFAULT_NULL_ORDERING = <span class="hljs-string">"hibernate.order_by.default_null_ordering"</span>;
String LOG_JDBC_WARNINGS =  <span class="hljs-string">"hibernate.jdbc.log.warnings"</span>;
String BEAN_CONTAINER = <span class="hljs-string">"hibernate.resource.beans.container"</span>;
String C3P0_CONFIG_PREFIX = <span class="hljs-string">"hibernate.c3p0"</span>;
String C3P0_MAX_SIZE = <span class="hljs-string">"hibernate.c3p0.max_size"</span>;
String C3P0_MIN_SIZE = <span class="hljs-string">"hibernate.c3p0.min_size"</span>;
String C3P0_TIMEOUT = <span class="hljs-string">"hibernate.c3p0.timeout"</span>;
String C3P0_MAX_STATEMENTS = <span class="hljs-string">"hibernate.c3p0.max_statements"</span>;
String C3P0_ACQUIRE_INCREMENT = <span class="hljs-string">"hibernate.c3p0.acquire_increment"</span>;
String C3P0_IDLE_TEST_PERIOD = <span class="hljs-string">"hibernate.c3p0.idle_test_period"</span>;
String PROXOOL_CONFIG_PREFIX = <span class="hljs-string">"hibernate.proxool"</span>;
String PROXOOL_PREFIX = PROXOOL_CONFIG_PREFIX;
String PROXOOL_XML = <span class="hljs-string">"hibernate.proxool.xml"</span>;
String PROXOOL_PROPERTIES = <span class="hljs-string">"hibernate.proxool.properties"</span>;
String PROXOOL_EXISTING_POOL = <span class="hljs-string">"hibernate.proxool.existing_pool"</span>;
String PROXOOL_POOL_ALIAS = <span class="hljs-string">"hibernate.proxool.pool_alias"</span>;
String CACHE_REGION_FACTORY = <span class="hljs-string">"hibernate.cache.region.factory_class"</span>;
String CACHE_KEYS_FACTORY = <span class="hljs-string">"hibernate.cache.keys_factory"</span>;
String CACHE_PROVIDER_CONFIG = <span class="hljs-string">"hibernate.cache.provider_configuration_file_resource_path"</span>;
String USE_SECOND_LEVEL_CACHE = <span class="hljs-string">"hibernate.cache.use_second_level_cache"</span>;
String USE_QUERY_CACHE = <span class="hljs-string">"hibernate.cache.use_query_cache"</span>;
String QUERY_CACHE_FACTORY = <span class="hljs-string">"hibernate.cache.query_cache_factory"</span>;
String CACHE_REGION_PREFIX = <span class="hljs-string">"hibernate.cache.region_prefix"</span>;
String USE_MINIMAL_PUTS = <span class="hljs-string">"hibernate.cache.use_minimal_puts"</span>;
String USE_STRUCTURED_CACHE = <span class="hljs-string">"hibernate.cache.use_structured_entries"</span>;
String AUTO_EVICT_COLLECTION_CACHE = <span class="hljs-string">"hibernate.cache.auto_evict_collection_cache"</span>;
String USE_DIRECT_REFERENCE_CACHE_ENTRIES = <span class="hljs-string">"hibernate.cache.use_reference_entries"</span>;
String DEFAULT_ENTITY_MODE = <span class="hljs-string">"hibernate.default_entity_mode"</span>;
String GLOBALLY_QUOTED_IDENTIFIERS = <span class="hljs-string">"hibernate.globally_quoted_identifiers"</span>;
String GLOBALLY_QUOTED_IDENTIFIERS_SKIP_COLUMN_DEFINITIONS = <span class="hljs-string">"hibernate.globally_quoted_identifiers_skip_column_definitions"</span>;
String CHECK_NULLABILITY = <span class="hljs-string">"hibernate.check_nullability"</span>;
String BYTECODE_PROVIDER = <span class="hljs-string">"hibernate.bytecode.provider"</span>;
String JPAQL_STRICT_COMPLIANCE= <span class="hljs-string">"hibernate.query.jpaql_strict_compliance"</span>;
String PREFER_POOLED_VALUES_LO = <span class="hljs-string">"hibernate.id.optimizer.pooled.prefer_lo"</span>;
String PREFERRED_POOLED_OPTIMIZER = <span class="hljs-string">"hibernate.id.optimizer.pooled.preferred"</span>;
String QUERY_PLAN_CACHE_MAX_STRONG_REFERENCES = <span class="hljs-string">"hibernate.query.plan_cache_max_strong_references"</span>;
String QUERY_PLAN_CACHE_MAX_SOFT_REFERENCES = <span class="hljs-string">"hibernate.query.plan_cache_max_soft_references"</span>;
String QUERY_PLAN_CACHE_MAX_SIZE = <span class="hljs-string">"hibernate.query.plan_cache_max_size"</span>;
String QUERY_PLAN_CACHE_PARAMETER_METADATA_MAX_SIZE = <span class="hljs-string">"hibernate.query.plan_parameter_metadata_max_size"</span>;
String NON_CONTEXTUAL_LOB_CREATION = <span class="hljs-string">"hibernate.jdbc.lob.non_contextual_creation"</span>;
String HBM2DDL_AUTO = <span class="hljs-string">"hibernate.hbm2ddl.auto"</span>;
String HBM2DDL_DATABASE_ACTION = <span class="hljs-string">"javax.persistence.schema-generation.database.action"</span>;
String HBM2DDL_SCRIPTS_ACTION = <span class="hljs-string">"javax.persistence.schema-generation.scripts.action"</span>;
String HBM2DDL_CONNECTION = <span class="hljs-string">"javax.persistence.schema-generation-connection"</span>;
String HBM2DDL_DB_NAME = <span class="hljs-string">"javax.persistence.database-product-name"</span>;
String HBM2DDL_DB_MAJOR_VERSION = <span class="hljs-string">"javax.persistence.database-major-version"</span>;
String HBM2DDL_DB_MINOR_VERSION = <span class="hljs-string">"javax.persistence.database-minor-version"</span>;
String HBM2DDL_CREATE_SOURCE = <span class="hljs-string">"javax.persistence.schema-generation.create-source"</span>;
String HBM2DDL_DROP_SOURCE = <span class="hljs-string">"javax.persistence.schema-generation.drop-source"</span>;
String HBM2DDL_CREATE_SCRIPT_SOURCE = <span class="hljs-string">"javax.persistence.schema-generation.create-script-source"</span>;
String HBM2DDL_DROP_SCRIPT_SOURCE = <span class="hljs-string">"javax.persistence.schema-generation.drop-script-source"</span>;
String HBM2DDL_SCRIPTS_CREATE_TARGET = <span class="hljs-string">"javax.persistence.schema-generation.scripts.create-target"</span>;
String HBM2DDL_SCRIPTS_DROP_TARGET = <span class="hljs-string">"javax.persistence.schema-generation.scripts.drop-target"</span>;
String HBM2DDL_IMPORT_FILES = <span class="hljs-string">"hibernate.hbm2ddl.import_files"</span>;
String HBM2DDL_LOAD_SCRIPT_SOURCE = <span class="hljs-string">"javax.persistence.sql-load-script-source"</span>;
String HBM2DDL_IMPORT_FILES_SQL_EXTRACTOR = <span class="hljs-string">"hibernate.hbm2ddl.import_files_sql_extractor"</span>;
String HBM2DDL_CREATE_NAMESPACES = <span class="hljs-string">"hibernate.hbm2ddl.create_namespaces"</span>;
String HBM2DLL_CREATE_NAMESPACES = <span class="hljs-string">"hibernate.hbm2dll.create_namespaces"</span>;
String HBM2DDL_CREATE_SCHEMAS = <span class="hljs-string">"javax.persistence.create-database-schemas"</span>;
String HBM2DLL_CREATE_SCHEMAS = HBM2DDL_CREATE_SCHEMAS;
String HBM2DDL_FILTER_PROVIDER = <span class="hljs-string">"hibernate.hbm2ddl.schema_filter_provider"</span>;
String HBM2DDL_JDBC_METADATA_EXTRACTOR_STRATEGY = <span class="hljs-string">"hibernate.hbm2ddl.jdbc_metadata_extraction_strategy"</span>;
String HBM2DDL_DELIMITER = <span class="hljs-string">"hibernate.hbm2ddl.delimiter"</span>;
String HBM2DDL_CHARSET_NAME = <span class="hljs-string">"hibernate.hbm2ddl.charset_name"</span>;
String HBM2DDL_HALT_ON_ERROR = <span class="hljs-string">"hibernate.hbm2ddl.halt_on_error"</span>;
String JMX_ENABLED = <span class="hljs-string">"hibernate.jmx.enabled"</span>;
String JMX_PLATFORM_SERVER = <span class="hljs-string">"hibernate.jmx.usePlatformServer"</span>;
String JMX_AGENT_ID = <span class="hljs-string">"hibernate.jmx.agentId"</span>;
String JMX_DOMAIN_NAME = <span class="hljs-string">"hibernate.jmx.defaultDomain"</span>;
String JMX_SF_NAME = <span class="hljs-string">"hibernate.jmx.sessionFactoryName"</span>;
String JMX_DEFAULT_OBJ_NAME_DOMAIN = <span class="hljs-string">"org.hibernate.core"</span>;
String CUSTOM_ENTITY_DIRTINESS_STRATEGY = <span class="hljs-string">"hibernate.entity_dirtiness_strategy"</span>;
String USE_ENTITY_WHERE_CLAUSE_FOR_COLLECTIONS = <span class="hljs-string">"hibernate.use_entity_where_clause_for_collections"</span>;
String MULTI_TENANT = <span class="hljs-string">"hibernate.multiTenancy"</span>;
String MULTI_TENANT_CONNECTION_PROVIDER = <span class="hljs-string">"hibernate.multi_tenant_connection_provider"</span>;
String MULTI_TENANT_IDENTIFIER_RESOLVER = <span class="hljs-string">"hibernate.tenant_identifier_resolver"</span>;
String INTERCEPTOR = <span class="hljs-string">"hibernate.session_factory.interceptor"</span>;
String SESSION_SCOPED_INTERCEPTOR = <span class="hljs-string">"hibernate.session_factory.session_scoped_interceptor"</span>;
String STATEMENT_INSPECTOR = <span class="hljs-string">"hibernate.session_factory.statement_inspector"</span>;
String ENABLE_LAZY_LOAD_NO_TRANS = <span class="hljs-string">"hibernate.enable_lazy_load_no_trans"</span>;
String HQL_BULK_ID_STRATEGY = <span class="hljs-string">"hibernate.hql.bulk_id_strategy"</span>;
String BATCH_FETCH_STYLE = <span class="hljs-string">"hibernate.batch_fetch_style"</span>;
String DELAY_ENTITY_LOADER_CREATIONS = <span class="hljs-string">"hibernate.loader.delay_entity_loader_creations"</span>;
String JTA_TRACK_BY_THREAD = <span class="hljs-string">"hibernate.jta.track_by_thread"</span>;
String JACC_CONTEXT_ID = <span class="hljs-string">"hibernate.jacc_context_id"</span>;
String JACC_PREFIX = <span class="hljs-string">"hibernate.jacc"</span>;
String JACC_ENABLED = <span class="hljs-string">"hibernate.jacc.enabled"</span>;
String ENABLE_SYNONYMS = <span class="hljs-string">"hibernate.synonyms"</span>;
String EXTRA_PHYSICAL_TABLE_TYPES = <span class="hljs-string">"hibernate.hbm2ddl.extra_physical_table_types"</span>;
String DEPRECATED_EXTRA_PHYSICAL_TABLE_TYPES = <span class="hljs-string">"hibernate.hbm2dll.extra_physical_table_types"</span>;
String UNIQUE_CONSTRAINT_SCHEMA_UPDATE_STRATEGY = <span class="hljs-string">"hibernate.schema_update.unique_constraint_strategy"</span>;
String GENERATE_STATISTICS = <span class="hljs-string">"hibernate.generate_statistics"</span>;
String LOG_SESSION_METRICS = <span class="hljs-string">"hibernate.session.events.log"</span>;
String LOG_SLOW_QUERY = <span class="hljs-string">"hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS"</span>;
String AUTO_SESSION_EVENTS_LISTENER = <span class="hljs-string">"hibernate.session.events.auto"</span>;
String PROCEDURE_NULL_PARAM_PASSING = <span class="hljs-string">"hibernate.proc.param_null_passing"</span>;
String CREATE_EMPTY_COMPOSITES_ENABLED = <span class="hljs-string">"hibernate.create_empty_composites.enabled"</span>;
String ALLOW_JTA_TRANSACTION_ACCESS = <span class="hljs-string">"hibernate.jta.allowTransactionAccess"</span>;
String ALLOW_UPDATE_OUTSIDE_TRANSACTION = <span class="hljs-string">"hibernate.allow_update_outside_transaction"</span>;
String COLLECTION_JOIN_SUBQUERY = <span class="hljs-string">"hibernate.collection_join_subquery"</span>;
String ALLOW_REFRESH_DETACHED_ENTITY = <span class="hljs-string">"hibernate.allow_refresh_detached_entity"</span>;
String MERGE_ENTITY_COPY_OBSERVER = <span class="hljs-string">"hibernate.event.merge.entity_copy_observer"</span>;
String USE_LEGACY_LIMIT_HANDLERS = <span class="hljs-string">"hibernate.legacy_limit_handler"</span>;
String VALIDATE_QUERY_PARAMETERS = <span class="hljs-string">"hibernate.query.validate_parameters"</span>;
String CRITERIA_LITERAL_HANDLING_MODE = <span class="hljs-string">"hibernate.criteria.literal_handling_mode"</span>;
String PREFER_GENERATOR_NAME_AS_DEFAULT_SEQUENCE_NAME = <span class="hljs-string">"hibernate.model.generator_name_as_sequence_name"</span>;
String JPA_TRANSACTION_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.transaction"</span>;
String JPA_QUERY_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.query"</span>;
String JPA_LIST_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.list"</span>;
String JPA_CLOSED_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.closed"</span>;
String JPA_PROXY_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.proxy"</span>;
String JPA_CACHING_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.caching"</span>;
String JPA_ID_GENERATOR_GLOBAL_SCOPE_COMPLIANCE = <span class="hljs-string">"hibernate.jpa.compliance.global_id_generators"</span>;
String TABLE_GENERATOR_STORE_LAST_USED = <span class="hljs-string">"hibernate.id.generator.stored_last_used"</span>;
String FAIL_ON_PAGINATION_OVER_COLLECTION_FETCH = <span class="hljs-string">"hibernate.query.fail_on_pagination_over_collection_fetch"</span>;
String IMMUTABLE_ENTITY_UPDATE_QUERY_HANDLING_MODE = <span class="hljs-string">"hibernate.query.immutable_entity_update_query_handling_mode"</span>;
String IN_CLAUSE_PARAMETER_PADDING = <span class="hljs-string">"hibernate.query.in_clause_parameter_padding"</span>;
String QUERY_STATISTICS_MAX_SIZE = <span class="hljs-string">"hibernate.statistics.query_max_size"</span>;
String SEQUENCE_INCREMENT_SIZE_MISMATCH_STRATEGY = <span class="hljs-string">"hibernate.id.sequence.increment_size_mismatch_strategy"</span>;
String OMIT_JOIN_OF_SUPERCLASS_TABLES = <span class="hljs-string">"hibernate.query.omit_join_of_superclass_tables"</span>;
</code></pre>
<p data-nodeid="45315">我担心有些同学懒得去看源码，所以就都贴到这里来了，你可以大概了解一下，做到心中有数。</p>
<p data-nodeid="45316">那么接下来我们看看该怎么使用 AvailableSettings 里面的配置呢？</p>
<p data-nodeid="45317"><strong data-nodeid="45436">AvailableSettings 里面的配置项的用法</strong></p>
<p data-nodeid="45318">我们只需要将 AvailableSettings 变量的值放到 spring.jpa.properties 里面即可，如下这些是我们常用的。</p>
<pre class="lang-yaml" data-nodeid="45319"><code data-language="yaml"><span class="hljs-comment">##开启hibernate statistics的信息，如session、连接等日志：</span>
<span class="hljs-string">spring.jpa.properties.hibernate.generate_statistics=true</span>
<span class="hljs-comment"># 格式化 SQL</span>
<span class="hljs-attr">spring.jpa.properties.hibernate.format_sql:</span> <span class="hljs-literal">true</span>
<span class="hljs-comment"># 显示 SQL</span>
<span class="hljs-attr">spring.jpa.properties.hibernate.show_sql:</span> <span class="hljs-literal">true</span>
<span class="hljs-comment"># 添加 HQL 相关的注释信息</span>
<span class="hljs-attr">spring.jpa.properties.hibernate.use_sql_comments:</span> <span class="hljs-literal">true</span>
<span class="hljs-comment"># hbm2ddl的策略 validate, update, create, create-drop, none，建议配置成validate，</span>
<span class="hljs-comment"># 这样在我们启动项目的时候就知道生产数据库的表结构是否正确的了，而不用等到运行期间才发现问题。</span>
<span class="hljs-string">spring.jpa.properties.hibernate.hbm2ddl.auto=validate</span>
<span class="hljs-comment"># 关联关系的时候取数据的深度，默认是3层，我们可以设置成2级，防止其他开发乱用，提高sql性能</span>
<span class="hljs-string">spring.jpa.properties.hibernate.max_fetch_depth=2</span>
<span class="hljs-comment"># 批量fetch大小默认 -1</span>
<span class="hljs-string">spring.jpa.properties.hibernate.default_batch_fetch_size=</span> <span class="hljs-number">100</span>
<span class="hljs-comment"># 事务完成之前是否进行flush操作，即同步到db里面去，默认是true</span>
<span class="hljs-string">spring.jpa.properties.hibernate.transaction.flush_before_completion=true</span>
<span class="hljs-comment"># 事务结束之后是否关闭session，默认false</span>
<span class="hljs-string">spring.jpa.properties.hibernate.transaction.auto_close_session=false</span>
<span class="hljs-comment"># 有的时候不只要批量查询，也会批量更新，默认batch size是15，我们可以根据实际情况自由调整，可以提高批量更新的效率；</span>
<span class="hljs-string">spring.jpa.properties.hibernate.jdbc.batch_size=100</span>
</code></pre>
<p data-nodeid="45320">其他的配置不经常用，我们就不需要关心了，你只知道在哪里看就好，实际用到时，发现哪些是我没举例的，你直接看源码会非常好理解的。</p>
<p data-nodeid="45321">这里我为什么要特别强调这个 Hibernate 的配置类呢？因为有的时候我们遇到问题会去网上搜索解决方案，发现别人给的配置可能不对，那么你就可以想到从这个源码中进行查看，并找到解决办法。</p>
<p data-nodeid="45322">本讲我们只关心了 JpaVendorAdapter 和 properties 的创建逻辑，我们前面在讲数据源的时候也说过这个类，里面有我们关心的 PlatformTransactionManager transactionManager 和 LocalContainerEntityManagerFactoryBean entityManagerFactory 的创建逻辑，而 JpaBaseConfiguration 这个类实现的逻辑还有很多，我在第 22 讲介绍 Session 的配置 open-in-view 的时候还会再详细介绍这个类。</p>
<p data-nodeid="45323">那么说了这么多加载的类，它们之间是什么关系呢？我们通过一个图来知晓一下。</p>
<h4 data-nodeid="52797">自动加载过程类之间的关系图</h4>
<p data-nodeid="52798" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08UuABFufAAFail5ZuqU603.png" alt="Drawing 6.png" data-nodeid="52802"></p>


<p data-nodeid="45326">从上图中，我们可以看出以下几点内容。</p>
<ol data-nodeid="45327">
<li data-nodeid="45328">
<p data-nodeid="45329">JpaBaseConfiguration 是 Jpa 和 Hibernate 被加载的基石，里面通过 BeanFactoryAware 的接口的 bean 加载生命周期也实现了一些逻辑。</p>
</li>
<li data-nodeid="45330">
<p data-nodeid="45331">HibernateJpaConfiguration 是 JpaBaseConfiguration 的子类，覆盖了一些父类里面的配置相关的特殊逻辑，并且里面引用了 JpaPropeties 和 HibernateProperties 的配置项。</p>
</li>
<li data-nodeid="45332">
<p data-nodeid="45333">HibernateJpaAutoConfiguration 是 Spring Boot 自动加载 HibernateJpaConfiguration 的桥梁，起到了 importHibernateJpaConfiguration 和加载 HibernateJpaConfiguration 的作用。</p>
</li>
<li data-nodeid="45334">
<p data-nodeid="45335">JpaRepositoriesAutoConfiguration 和 HibernateJpaAutoConfiguration、DataSourceAutoConfiguration 分别加载 JpaRepositories 的逻辑和 HibernateJPA、数据源，都是被 spring.factories 自动装配进入到 Spring Boot 里面的，而三者之间有加载的先后顺序。</p>
</li>
<li data-nodeid="45336">
<p data-nodeid="45337">上图的 UML 还展示了几个 Configuration 类的加载顺序和依赖关系，顺序是从上到下进行加载的，其中 DataSourceAutoConfiguration 最先加载、HibernateJpaAutoConfiguration 第二顺序加载、JpaRepositoriesAutoConfiguration 最后加载。</p>
</li>
</ol>
<p data-nodeid="45338">我们了解完了 Hibernate 5 在 Spring Boot 里面的加载过程，那么来看下 JpaRepositoriesAutoConfiguration 的主要作用有哪些。</p>
<h3 data-nodeid="45339">Spring Data JPA Repositories Bootstrap Mode</h3>
<p data-nodeid="45340">我们通过上面分享的整个加载过程可以发现，DataSourceAutoConfiguration 完成了数据源的加载，HibernateJpaAutoConfiguration 完成了 Hibernate 的加载过程，而 JpaRepositoriesAutoConfiguration 要做的就是解决我们之前定义的 Repositories 相关的实体和接口的加载初始化过程，这是 Spring Data JPA 的主要实现逻辑，和 Hiberante、数据源没什么关系了。</p>
<p data-nodeid="53655">我们可以通过 JpaRepositoriesAutoConfiguration 的源码发现其主要职责和实现方式，利用异步线程池初始化 repositories，关键源码如下：</p>
<p data-nodeid="53656" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/45/CgqCHl-08VmASV72AAJavgahY_A852.png" alt="Drawing 7.png" data-nodeid="53660"></p>


<p data-nodeid="45343">而其中加载 repositories 有三种方式，即 spring.data.jpa.repositories.bootstrap-mode 的三个值，分别为 deferred、 lazy、 default，下面详细说明。</p>
<ul data-nodeid="45344">
<li data-nodeid="45345">
<p data-nodeid="45346">deferred：是默认值，表示在启动的时候会进行数据库字段的检查，而 repositories 相关的实例的初始化是 lazy 模式，也就是在第一次用到 repositories 实例的时候再进行初始化。这个比较适合用在测试环境和生产环境中，因为测试不可能覆盖所有场景，万一谁多加个字段或者少一个字段，这样在启动的阶段就可以及时发现问题，不能等进行到生产环境才暴露。</p>
</li>
<li data-nodeid="45347">
<p data-nodeid="45348">lazy：表示启动阶段不会进行数据库字段的检查，也不会初始化 repositories 相关的实例，而是在第一次用到 repositories 实例的时候再进行初始化。这个比较适合用在开发的阶段，可以加快应用的启动速度。如果生产环境中，我们为了提高业务高峰期间水平来扩展应用的启动速度，也可以采用这种模式。</p>
</li>
<li data-nodeid="45349">
<p data-nodeid="45350">default：默认加载方式，但从 Spring Boot 2.0 之后就不是默认值了，表示立即验证、立即初始化 repositories 实例，这种方式启动的速度最慢，但是最保险，运行期间的请求最快，因为避免了第一次请求初始化 repositories 实例的过程。</p>
</li>
</ul>
<p data-nodeid="45351">我们通过在 application.properties 里面修改这一行代码，来测试一下 lazy 的加载方式。</p>
<pre class="lang-java" data-nodeid="45352"><code data-language="java">spring.data.jpa.repositories.bootstrap-mode=lazy
</code></pre>
<p data-nodeid="45353">然后启动我们的项目，就会发现在 tomcat 容器加载完之后，没有用到 UserInfoRepository 之前，这个 UserInfoRepository 是不会进行初始化的。而当我们发一个请求用到了 UserInfoRepository，就进行了初始化。</p>
<p data-nodeid="54513">我们通过日志也可以看到，启动的线程和初始化的线程是不一样的，而初始化的线程是 NIO 线程的名字，表示 request 的 http 线程池里面的线程，具体如下图所示。</p>
<p data-nodeid="56058"><img src="https://s0.lgstatic.com/i/image/M00/6F/3A/Ciqc1F-08WGAaqVkAAR8a19UBFQ188.png" alt="Drawing 8.png" data-nodeid="56061"></p>


<p data-nodeid="55392">我们在分析 Hibernate 的加载方式的时候，会发现日志的重要性，那么都有哪些日志供我们观察呢？如何开启？</p>








<h3 data-nodeid="45357">Debug 时候，日志的配置</h3>
<pre class="lang-yaml" data-nodeid="45358"><code data-language="yaml"><span class="hljs-comment">### 日志级别的灵活运用</span>
<span class="hljs-comment">## hibernate相关</span>
<span class="hljs-comment"># 显示sql的执行日志，如果开了这个,show_sql就可以不用了</span>
<span class="hljs-string">logging.level.org.hibernate.SQL=debug</span>
<span class="hljs-comment"># hibernate id的生成日志</span>
<span class="hljs-string">logging.level.org.hibernate.id=debug</span>
<span class="hljs-comment"># hibernate所有的操作都是PreparedStatement，把sql的执行参数显示出来</span>
<span class="hljs-string">logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE</span>
<span class="hljs-comment"># sql执行完提取的返回值</span>
<span class="hljs-string">logging.level.org.hibernate.type.descriptor.sql=trace</span>
<span class="hljs-comment"># 请求参数</span>
<span class="hljs-string">logging.level.org.hibernate.type=debug</span>
<span class="hljs-comment"># 缓存相关</span>
<span class="hljs-string">logging.level.org.hibernate.cache=debug</span>
<span class="hljs-comment"># 统计hibernate的执行状态</span>
<span class="hljs-string">logging.level.org.hibernate.stat=debug</span>
<span class="hljs-comment"># 查看所有的缓存操作</span>
<span class="hljs-string">logging.level.org.hibernate.event.internal=trace</span>
<span class="hljs-string">logging.level.org.springframework.cache=trace</span>
<span class="hljs-comment"># hibernate 的监控指标日志</span>
<span class="hljs-string">logging.level.org.hibernate.engine.internal.StatisticalLoggingSessionEventListener=DEBUG</span>
<span class="hljs-comment">### 连接池的相关日志</span>
<span class="hljs-comment">## hikari连接池的状态日志，以及连接池是否完好 #连接池的日志效果：HikariCPPool - Pool stats (total=20, active=0, idle=20, waiting=0)</span>
<span class="hljs-string">logging.level.com.zaxxer.hikari=TRACE</span>
<span class="hljs-comment">#开启 debug可以看到 AvailableSettings里面的默认配置的值都有哪些，会输出类似下面的日志格式</span>
<span class="hljs-comment"># org.hibernate.cfg.Settings               : Statistics: enabled</span>
<span class="hljs-comment"># org.hibernate.cfg.Settings               : Default batch fetch size: -1</span>
<span class="hljs-string">logging.level.org.hibernate.cfg=debug</span>
<span class="hljs-comment">#hikari数据的配置项日志</span>
<span class="hljs-string">logging.level.com.zaxxer.hikari.HikariConfig=TRACE</span>
<span class="hljs-comment">### 查看事务相关的日志，事务获取，释放日志</span>
<span class="hljs-string">logging.level.org.springframework.orm.jpa=DEBUG</span>
<span class="hljs-string">logging.level.org.springframework.transaction=TRACE</span>
<span class="hljs-string">logging.level.org.hibernate.engine.transaction.internal.TransactionImpl=DEBUG</span>
<span class="hljs-comment">### 分析connect 以及 orm和 data的处理过程更全的日志</span>
<span class="hljs-string">logging.level.org.springframework.data=trace</span>
<span class="hljs-string">logging.level.org.springframework.orm=trace</span>
</code></pre>
<p data-nodeid="45359">上面是我在分析复杂问题和原理的时候常用的日志配置项目，这里给你提供一个技巧，当我们分析一个问题的时候，如果不知道日志具体在哪个类里面，通过设置&nbsp;logging.level.root=trace 的话，日志又非常多几乎没有办法看，那么我们可以缩小范围，不如说我们分析的是 hikari 包里面相关的问题。</p>
<p data-nodeid="45360">我们可以把整个日志级别 logging.level.root=info 设置成 info，把其他所有的日志都关闭，并把 logging.level.com.zaxxer=trace 设置成最大的，保持日志不受干扰，然后观察日志再逐渐减少查看范围。</p>
<h3 data-nodeid="45361">总结</h3>
<p data-nodeid="45362">这一讲我通过源码分析，帮助你了解了 JpaRepositoriesAutoConfiguration、HibernateJpaAutoConfiguration、DataSourceAutoConfiguration 的主要作用和加载顺序的依赖，还介绍了 Spring Hibernate 的配置项有哪些。</p>
<p data-nodeid="45363">你在工作中可以举一反三，通过 debug 断点一步一步分析出来这一讲没涉及的东西。比如可以自己做一个项目，跟着我的步骤操作，你会对这部分的内容有更深刻的体会。这样当遇到一些问题，并且网上没有合适的资料时，你可以试着采用本讲中我分享给你的思路来解决。</p>
<p data-nodeid="56492" class="te-preview-highlight">下一讲，我会为你介绍一个 Hibernate 实现的 JPA 的概念：Persistence Context。欢迎你提前预习，并结合这一讲内容去思考，有疑问的地方请留言，我会及时给予答复。</p>

<blockquote data-nodeid="45365">
<p data-nodeid="45366">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="45481">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **凡：
> 老师，请问设置了spring.jpa.properties.hibernate. jdbc. batch_size，为什么插入还是不能批量提交？用druid监控看依旧每个插入执行一遍

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; spring.jpa.properties.hibernate.order_inserts=true #开启按需insert sql重新组合

