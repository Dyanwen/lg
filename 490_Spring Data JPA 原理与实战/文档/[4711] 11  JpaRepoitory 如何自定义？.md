<p data-nodeid="1525" class="">通过前面课时的内容，相信你已经掌握了很多 Repository 的高级用法，但是在实际工作场景中也难免会出现自定义 Repsitory 实现类的场景，这一课时我们就来看一下如何定义自己的 Repository 实现类。要知道 JPA 的操作核心是 EntityManager，那么我们先看看 Entitymanager 究竟为何物。</p>
<h3 data-nodeid="1526">EntityManager 介绍</h3>
<p data-nodeid="1527">Java Persistence API 规定，操作数据库实体必须要通过 EntityManager 进行，而我们前面看到了所有的 Repository 在 JPA 里面的实现类是 SimpleJpaRepository，它在真正操作实体的时候都是调用 EntityManager 里面的方法。</p>
<p data-nodeid="1528">我们在 SimpleJpaRepository 里面设置一个断点，这样可以很容易看得出来 EntityManger 是 JPA 的接口协议，而其现类是 Hibernate 里面的 SessionImpl，如下图所示：</p>
<p data-nodeid="1529"><img src="https://s0.lgstatic.com/i/image/M00/60/71/CgqCHl-NSiyAayfKAAd_nCX2604232.png" alt="Drawing 0.png" data-nodeid="1676"></p>
<p data-nodeid="1530">那么我们看看 EntityManager 给我们提供了哪些方法。</p>
<h4 data-nodeid="1531">EntityManager 方法有哪些？</h4>
<p data-nodeid="1532">下面介绍几个重要的、比较常用的方法，不常用的我将一笔带过，如果你有兴趣可以自行查看。</p>
<pre class="lang-java" data-nodeid="1533"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">EntityManager</span> </span>{
  <span class="hljs-comment">//用于将新创建的Entity纳入EntityManager的管理。该方法执行后，传入persist()方法的 Entity 对象转换成持久化状态。</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">persist</span><span class="hljs-params">(Object entity)</span></span>;
  <span class="hljs-comment">//将游离态的实体merge到当前的persistence context里面，一般用于更新。</span>
  <span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">merge</span><span class="hljs-params">(T entity)</span></span>;
  <span class="hljs-comment">//将实体对象删除，物理删除</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">remove</span><span class="hljs-params">(Object entity)</span></span>;
  <span class="hljs-comment">//将当前的persistence context中的实体，同步到数据库里面，只有执行了这个方法，上面的EntityManager的操作才会DB生效；</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flush</span><span class="hljs-params">()</span></span>;
  <span class="hljs-comment">//根据实体类型和主键查询一个实体对象；</span>
  <span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">find</span><span class="hljs-params">(Class&lt;T&gt; entityClass, Object primaryKey)</span></span>;
  <span class="hljs-comment">//根据JPQL创建一个Query对象</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> Query <span class="hljs-title">createQuery</span><span class="hljs-params">(String qlString)</span></span>;
  <span class="hljs-comment">//利用CriteriaUpdate创建更新查询</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> Query <span class="hljs-title">createQuery</span><span class="hljs-params">(CriteriaUpdate updateQuery)</span></span>;
  <span class="hljs-comment">//利用原生的sql语句创建查询，可以是查询、更新、删除等sql</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> Query <span class="hljs-title">createNativeQuery</span><span class="hljs-params">(String sqlString)</span></span>;
  ...<span class="hljs-comment">//其他方法我就不一一列举了，用法很简单，我们只要参看SimpleJpaRepository里面怎么用的，我们怎么用就可以了；</span>
}
</code></pre>
<p data-nodeid="1534">这一课时我们先知道 EntityManager 的语法和用法就好，在之后的第 21 课时介绍 Persistence Context 的时候，再详细讲一下其对实体状态的影响，以及每种状态代表什么意思。</p>
<p data-nodeid="1535">那么现在你知道了这些语法，该怎么使用呢？</p>
<h4 data-nodeid="1536">EntityManager 如何使用？</h4>
<p data-nodeid="1537">它的使用方法很简单，我们在任何地方只要能获得 EntityManager，就可以进行里面的操作。</p>
<p data-nodeid="1538"><strong data-nodeid="1687">获得 EntityManager 的方式：通过 @PersistenceContext 注解。</strong></p>
<p data-nodeid="1539">将 @PersistenceContext 注解标注在 EntityManager 类型的字段上，这样得到的 EntityManager 就是容器管理的 EntityManager。由于是容器管理的，所以我们不需要、也不应该显式关闭注入的 EntityManager 实例。</p>
<p data-nodeid="1540">下面是关于这种方式的例子，我们想要在测试类中获得 @PersistenceContext 里面的 EntityManager，看看代码应该怎么写。</p>
<pre class="lang-java" data-nodeid="1541"><code data-language="java"><span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-meta">@TestInstance(TestInstance.Lifecycle.PER_CLASS)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRepositoryTest</span> </span>{
    <span class="hljs-comment">//利用该方式获得entityManager</span>
    <span class="hljs-meta">@PersistenceContext</span>
    <span class="hljs-keyword">private</span> EntityManager entityManager;
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository userRepository;
    <span class="hljs-comment">/**
     * 测试entityManager用法
     *
     * <span class="hljs-doctag">@throws</span> JsonProcessingException
     */</span>
    <span class="hljs-meta">@Test</span>
    <span class="hljs-meta">@Rollback(false)</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testEntityManager</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> JsonProcessingException </span>{
        <span class="hljs-comment">//测试找到一个User对象</span>
        User user = entityManager.find(User.class,<span class="hljs-number">2L</span>);
        Assertions.assertEquals(user.getAddresses(),<span class="hljs-string">"shanghai"</span>);

        <span class="hljs-comment">//我们改变一下user的删除状态</span>
        user.setDeleted(<span class="hljs-keyword">true</span>);
        <span class="hljs-comment">//merger方法</span>
        entityManager.merge(user);
        <span class="hljs-comment">//更新到数据库里面</span>
        entityManager.flush();

        <span class="hljs-comment">//再通过createQuery创建一个JPQL，进行查询</span>
        List&lt;User&gt; users =  entityManager.createQuery(<span class="hljs-string">"select u From User u where u.name=?1"</span>)
                .setParameter(<span class="hljs-number">1</span>,<span class="hljs-string">"jack"</span>)
                .getResultList();
        Assertions.assertTrue(users.get(<span class="hljs-number">0</span>).getDeleted());
    }
}
</code></pre>
<p data-nodeid="1542">我们通过这个测试用例，可以知道 EntityManager 使用起来还是比较容易的。不过在实际工作中，我不建议直接操作 EntityManager，因为如果你操作不熟练的话，会出现一些事务异常。因此我还是建议你通过 Spring Data JPA 给我们提供的 Repositories 方式进行操作。<br>
提示一下，你在写框架的时候可以直接操作 EntityManager，<strong data-nodeid="1697">切记不要在任何业务代码里面都用到 EntityManager，否则自己的代码到最后就会很难维护</strong>。</p>
<p data-nodeid="1543">EntityManager 我们了解完了，那么我们再看下 @EnableJpaRepositories 对自定义 Repository 起了什么作用。</p>
<h3 data-nodeid="1544">@EnableJpaRepositories 详解</h3>
<p data-nodeid="1545">下面分别从 @EnableJpaRepositories 的语法，以及其默认加载方式来详细介绍一下。</p>
<h4 data-nodeid="1546">@EnableJpaRepositories 语法</h4>
<p data-nodeid="1547">我们还是直接看代码，如下所示：</p>
<pre class="lang-java" data-nodeid="1548"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> EnableJpaRepositories {
   String[] value() <span class="hljs-keyword">default</span> {};
   String[] basePackages() <span class="hljs-keyword">default</span> {};
   Class&lt;?&gt;[] basePackageClasses() <span class="hljs-keyword">default</span> {};
   Filter[] includeFilters() <span class="hljs-keyword">default</span> {};
   Filter[] excludeFilters() <span class="hljs-keyword">default</span> {};
   <span class="hljs-function">String <span class="hljs-title">repositoryImplementationPostfix</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> "Impl"</span>;
   <span class="hljs-function">String <span class="hljs-title">namedQueriesLocation</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-function">Key <span class="hljs-title">queryLookupStrategy</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> Key.CREATE_IF_NOT_FOUND</span>;
   Class&lt;?&gt; repositoryFactoryBeanClass() <span class="hljs-keyword">default</span> JpaRepositoryFactoryBean.class;
   Class&lt;?&gt; repositoryBaseClass() <span class="hljs-keyword">default</span> DefaultRepositoryBaseClass.class;
   <span class="hljs-function">String <span class="hljs-title">entityManagerFactoryRef</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> "entityManagerFactory"</span>;
   <span class="hljs-function">String <span class="hljs-title">transactionManagerRef</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> "transactionManager"</span>;
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">considerNestedRepositories</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">enableDefaultTransactions</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>
<p data-nodeid="1549">下面我对里面的 10 个方法进行一下具体说明：</p>
<p data-nodeid="1550"><strong data-nodeid="1707">1）value 等于 basePackage</strong></p>
<p data-nodeid="1551">用于配置扫描 Repositories 所在的 package 及子 package。</p>
<p data-nodeid="1552">可以配置为单个字符串。</p>
<pre class="lang-java" data-nodeid="1553"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(basePackages = "com.example")</span>
</code></pre>
<p data-nodeid="1554">也可以配置为字符串数组形式，即多个情况。</p>
<pre class="lang-java" data-nodeid="1555"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(basePackages = {"com.sample.repository1", 	"com.sample.repository2"})</span>
</code></pre>
<p data-nodeid="1556">默认 @SpringBootApplication 注解出现目录及其子目录。</p>
<p data-nodeid="1557"><strong data-nodeid="1715">2）basePackageClasses</strong></p>
<p data-nodeid="1558">指定 Repository 类所在包，可以替换 basePackage 的使用。</p>
<p data-nodeid="1559">一样可以单个字符，下面例子表示 BookRepository.class 所在 Package 下面的所有 Repositories 都会被扫描注册。</p>
<pre class="lang-java" data-nodeid="1560"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(basePackageClasses = BookRepository.class)</span>
</code></pre>
<p data-nodeid="1561">也可以多个字符，下面的例子代表 ShopRepository.class, OrganizationRepository.class 所在的 package下面的所有 Repositories 都会被扫描。</p>
<pre class="lang-java" data-nodeid="1562"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(basePackageClasses = {ShopRepository.class, OrganizationRepository.class})</span>
</code></pre>
<p data-nodeid="1563"><strong data-nodeid="1722">3）includeFilters</strong></p>
<p data-nodeid="1564">指定包含的过滤器，该过滤器采用 ComponentScan 的过滤器，可以指定过滤器类型。</p>
<p data-nodeid="1565">下面的例子表示只扫描带 Repository 注解的类。</p>
<pre class="lang-java" data-nodeid="1566"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories( includeFilters={@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Repository.class)})</span>
</code></pre>
<p data-nodeid="1567"><strong data-nodeid="1728">4）excludeFilters</strong></p>
<p data-nodeid="1568">指定不包含过滤器，该过滤器也是采用 ComponentScan 的过滤器里面的类。</p>
<p data-nodeid="1569">下面的例子表示，带 @Service 和 @Controller 注解的类，不用扫描进去，当我们的项目变大了之后可以加快应用的启动速度。</p>
<pre class="lang-java" data-nodeid="1570"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(excludeFilters={@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Service.class),@ComponentScan.Filter(type=FilterType.ANNOTATION, value=Controller.class)})</span>
</code></pre>
<p data-nodeid="1571"><strong data-nodeid="1734">5）repositoryImplementationPostfix</strong></p>
<p data-nodeid="1572">当我们自定义 Repository 的时候，约定的接口 Repository 的实现类的后缀是什么，默认是 Impl。例子我在下面详细讲解。</p>
<p data-nodeid="1573"><strong data-nodeid="1739">6）namedQueriesLocation</strong></p>
<p data-nodeid="1574">named SQL 存放的位置，默认为 META-INF/jpa-named-queries.properties</p>
<p data-nodeid="1575">例子如下：</p>
<pre class="lang-java" data-nodeid="1576"><code data-language="java">Todo.findBySearchTermNamedFile=<span class="hljs-function">SELECT t FROM Table t WHERE <span class="hljs-title">LOWER</span><span class="hljs-params">(t.description)</span> LIKE <span class="hljs-title">LOWER</span><span class="hljs-params">(CONCAT(<span class="hljs-string">'%'</span>, :searchTerm, <span class="hljs-string">'%'</span>)</span>) ORDER BY t.title ASC
</span></code></pre>
<p data-nodeid="1577">这个你知道就行了，我建议不要用，因为它虽然功能很强大，但是，当我们使用了这么复杂的方法时，你需要想一想是否有更简单的方法。</p>
<p data-nodeid="1578"><strong data-nodeid="1746">7）queryLookupStrategy</strong></p>
<p data-nodeid="1579">构建条件查询的查找策略，包含三种方式：CREATE、USE_DECLARED_QUERY、CREATE_IF_NOT_FOUND。</p>
<p data-nodeid="1580">正如我们前几课时介绍的：</p>
<ul data-nodeid="1581">
<li data-nodeid="1582">
<p data-nodeid="1583">CREATE：按照接口名称自动构建查询方法，即我们前面说的 Defining Query Methods；</p>
</li>
<li data-nodeid="1584">
<p data-nodeid="1585">USE_DECLARED_QUERY：用 @Query 这种方式查询；</p>
</li>
<li data-nodeid="1586">
<p data-nodeid="1587">CREATE_IF_NOT_FOUND：如果有 @Query 注解，先以这个为准；如果不起作用，再用 Defining Query Methods；这个是默认的，基本不需要修改，我们知道就行了。</p>
</li>
</ul>
<p data-nodeid="1588"><strong data-nodeid="1775">8）repositoryFactoryBeanClass</strong></p>
<p data-nodeid="1589">指定生产 Repository 的工厂类，默认 JpaRepositoryFactoryBean。JpaRepositoryFactoryBean 的主要作用是以动态代理的方式，帮我们所有 Repository 的接口生成实现类。例如当我们通过断点，看到 UserRepository 的实现类是 SimpleJpaRepository 代理对象的时候，就是这个工厂类干的，一般我们很少会去改变这个生成代理的机制。</p>
<p data-nodeid="1590"><strong data-nodeid="1780">9）entityManagerFactoryRef</strong></p>
<p data-nodeid="1591">用来指定创建和生产 EntityManager 的工厂类是哪个，默认是 name=“entityManagerFactory” 的 Bean。一般用于多数据配置。</p>
<p data-nodeid="1592"><strong data-nodeid="1787">10）Class&lt;?&gt; repositoryBaseClass()</strong></p>
<p data-nodeid="1593">用来指定我们自定义的 Repository 的实现类是什么。默认是 DefaultRepositoryBaseClass，即表示没有指定的 Repository 的实现基类。</p>
<p data-nodeid="1594"><strong data-nodeid="1795">11）String transactionManagerRef() default "transactionManager"</strong></p>
<p data-nodeid="1595">用来指定默认的事务处理是哪个类，默认是 transactionManager，一般用于多数据源。</p>
<p data-nodeid="1596">以上就是 @EnableJpaRepositories 的基本语法了，涉及的方法比较多，你可以慢慢探索。下面再看看默认是怎么加载的。</p>
<h4 data-nodeid="1597">@EnableJpaRepositories 默认加载方式</h4>
<p data-nodeid="1598">默认情况下是 spring boot 的自动加载机制，通过 spring.factories 的文件加载 JpaRepositoriesAutoConfiguration，如下图：</p>
<p data-nodeid="1599"><img src="https://s0.lgstatic.com/i/image/M00/60/71/CgqCHl-NSnqAe4i7AAHNYXt2Rbo960.png" alt="Drawing 1.png" data-nodeid="1802"></p>
<p data-nodeid="1600">JpaRepositoriesAutoConfiguration 里面再进行 @Import(JpaRepositoriesRegistrar.class) 操作，显示如下：</p>
<p data-nodeid="1601"><img src="https://s0.lgstatic.com/i/image/M00/60/66/Ciqc1F-NSoOAZLC9AAFKgEB_ZbM671.png" alt="Drawing 2.png" data-nodeid="1806"></p>
<p data-nodeid="1602">而 JpaRepositoriesRegistrar.class 里面配置了 @EnableJpaRepositories，从而使默认值产生了如下效果：</p>
<p data-nodeid="1603"><img src="https://s0.lgstatic.com/i/image/M00/60/66/Ciqc1F-NSoqAC5SBAAGV8mrWK7o741.png" alt="Drawing 3.png" data-nodeid="1810"></p>
<p data-nodeid="1604">这样关于 @EnableJpaRepositories 的语法以及默认加载方式就介绍完了，你就可以知道通过 @EnableJpaRepositories 可以完成很多我们自定义的需求。那么到底如何定义自己的 Repository 的实现类呢？我们接着看。</p>
<h3 data-nodeid="1605">自定义 Repository 的 impl 的方法</h3>
<p data-nodeid="1606">定义自己的 Repository 的实现，有以下两种方法。</p>
<h4 data-nodeid="1607">第一种方法：定义独立的 Repository 的 Impl 实现类</h4>
<p data-nodeid="1608">我们通过一个实例说明一下，假设我们要实现一个逻辑删除的功能，看看应该怎么做？</p>
<p data-nodeid="1609"><strong data-nodeid="1819">第一步：定义一个 CustomizedUserRepository 接口。</strong></p>
<p data-nodeid="1610">此接口会自动被 @EnableJpaRepositories 开启之后扫描到，代码如下：</p>
<pre class="lang-java" data-nodeid="1611"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.customized;
<span class="hljs-keyword">import</span> com.example.jpa.example1.User;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">CustomizedUserRepository</span> </span>{
    <span class="hljs-function">User <span class="hljs-title">logicallyDelete</span><span class="hljs-params">(User user)</span></span>;
}
</code></pre>
<p data-nodeid="1612"><strong data-nodeid="1824">第二步：创建一个 CustomizedUserRepositoryImpl 实现类。</strong></p>
<p data-nodeid="1613">并且实现类用我们上面说过的 Impl 结尾，如下所示：</p>
<pre class="lang-java" data-nodeid="1614"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.customized;
<span class="hljs-keyword">import</span> com.example.jpa.example1.User;
<span class="hljs-keyword">import</span> javax.persistence.EntityManager;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomizedUserRepositoryImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">CustomizedUserRepository</span> </span>{
    <span class="hljs-keyword">private</span> EntityManager entityManager;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CustomizedUserRepositoryImpl</span><span class="hljs-params">(EntityManager entityManager)</span> </span>{
        <span class="hljs-keyword">this</span>.entityManager = entityManager;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">logicallyDelete</span><span class="hljs-params">(User user)</span> </span>{
        user.setDeleted(<span class="hljs-keyword">true</span>);
        <span class="hljs-keyword">return</span> entityManager.merge(user);
    }
}
</code></pre>
<p data-nodeid="1615">其中我们也发现了 EntityManager 的第二种注入方式，即直接放在构造方法里面，通过 Spring 自动注入。</p>
<p data-nodeid="1616"><strong data-nodeid="1830">第三步：当用到 UserRepository 的时候，直接继承我们自定义的 CustomizedUserRepository 接口即可。</strong></p>
<pre class="lang-java" data-nodeid="1617"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt;, <span class="hljs-title">JpaSpecificationExecutor</span>&lt;<span class="hljs-title">User</span>&gt;, <span class="hljs-title">CustomizedUserRepository</span> </span>{
}
</code></pre>
<p data-nodeid="1618"><strong data-nodeid="1834">第四步：写一个测试用例测试一下。</strong></p>
<pre class="lang-java" data-nodeid="1619"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testCustomizedUserRepository</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">//查出来一个User对象</span>
    User user = userRepository.findById(<span class="hljs-number">2L</span>).get();
    <span class="hljs-comment">//调用我们的逻辑删除方法进行删除</span>
    userRepository.logicallyDelete(user);
    <span class="hljs-comment">//我们再重新查出来，看看值变了没有</span>
    List&lt;User&gt; users = userRepository.findAll();
    Assertions.assertEquals(users.get(<span class="hljs-number">0</span>).getDeleted(),Boolean.TRUE);
}
</code></pre>
<p data-nodeid="1620">最后调用刚才我们自定义的逻辑删除方法 logicallyDelete，跑一下测试用例，结果完全通过。那么此种方法的实现原理是什么呢？我们通过 debug 分析一下。</p>
<h4 data-nodeid="1621">原理分析</h4>
<p data-nodeid="1622">我们在上面讲过 Class&lt;?&gt; repositoryFactoryBeanClass() default JpaRepositoryFactoryBean.class，repository 的动态代理创建工厂是： JpaRepositoryFactoryBean，它会帮我们生产 repository 的实现类，那么我们直接看一下JpaRepositoryFactoryBean 的源码，分析其原理。</p>
<p data-nodeid="1623"><img src="https://s0.lgstatic.com/i/image/M00/60/72/CgqCHl-NSq6AFAjuAAD93443waY861.png" alt="Drawing 4.png" data-nodeid="1842"></p>
<p data-nodeid="1624">设置一个断点，就会发现，每个 Repository 都会构建一个 JpaRepositoryFactory，当 JpaRepositoryFactory 加载完之后会执行 afterPropertiesSet() 方法，找到 UserRepository 的 Fragment（即我们自定义的 CustomizedUserRepositoryImpl），如下所示：</p>
<p data-nodeid="1625"><img src="https://s0.lgstatic.com/i/image/M00/60/72/CgqCHl-NSrSAZj08AALzEQG8Sws504.png" alt="Drawing 5.png" data-nodeid="1846"></p>
<p data-nodeid="1626">我们再看 RepositoryFactory 里面的所有方法，如下图，一看就是动态代理生成 Repository 的实现类，我们进到这个方法里面设置个断点继续观察。</p>
<p data-nodeid="1627"><img src="https://s0.lgstatic.com/i/image/M00/60/72/CgqCHl-NSrmANrzIAALtqlmUVEE696.png" alt="Drawing 6.png" data-nodeid="1850"></p>
<p data-nodeid="1628">然后我们通过断点可以看到，fragments 放到了 composition 里面，最后又放到了 advice 里面，最后才生成了我们的 repository 的代理类。这时我们再打开 repository 详细地看看里面的值。</p>
<p data-nodeid="1629"><img src="https://s0.lgstatic.com/i/image/M00/60/72/CgqCHl-NSr-AfcXSAAQrh_gENO8150.png" alt="Drawing 7.png" data-nodeid="1854"></p>
<p data-nodeid="1630">可以看到 repository 里面的 interfaces，就是我们刚才测试 userRepository 里面的接口定义的。</p>
<p data-nodeid="1631"><img src="https://s0.lgstatic.com/i/image/M00/60/66/Ciqc1F-NSsaAeUC3AALLvvFPRrM408.png" alt="Drawing 8.png" data-nodeid="1858"></p>
<p data-nodeid="1632">我们可以看到 advisors 里面第六个就是我们自定义的接口的实现类，从这里可以得出结论：spring 通过扫描所有 repository 的接口和实现类，并且通过 aop 的切面和动态代理的方式，可以知道我们自定义的接口的实现类是什么。</p>
<p data-nodeid="1633">针对不同的 repository 自定义的接口和实现类，需要我们手动去 extends，这种比较适合不同的业务场景有各自的 repository 的实现情况。还有一种方法是我们直接改变动态代理的实现类，我们接着看。</p>
<h4 data-nodeid="1634">第二种方法：通过 @EnableJpaRepositories 定义默认的 Repository 的实现类</h4>
<p data-nodeid="1635">当面对复杂业务的时候，难免会自定义一些公用的方法，或者覆盖一些默认实现的情况。举个例子：很多时候线上的数据是不允许删除的，所以这个时候需要我们覆盖 SimpleJpaRepository 里面的删除方法，换成更新，进行逻辑删除，而不是物理删除。那么接下来我们看看应该怎么做？</p>
<p data-nodeid="1636"><strong data-nodeid="1867">第一步：正如上面我们讲的利用 @EnableJpaRepositories 指定 repositoryBaseClass</strong>，代码如下：</p>
<pre class="lang-java" data-nodeid="1637"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableWebMvc</span>
<span class="hljs-meta">@EnableJpaRepositories(repositoryImplementationPostfix = "Impl",repositoryBaseClass = CustomerBaseRepository.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JpaApplication</span> </span>{
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
      SpringApplication.run(JpaApplication.class, args);
   }
}
</code></pre>
<p data-nodeid="1638">可以看出，在启动项目的时候，通过 @EnableJpaRepositories 指定我们 repositoryBaseClass 的基类是 CustomerBaseRepository。</p>
<p data-nodeid="1639"><strong data-nodeid="1872">第二步：创建 CustomerBaseRepository 继承 SimpleJpaRepository 即可。</strong></p>
<p data-nodeid="1640">继承 SimpleJpaRepository 之后，我们直接覆盖 delete 方法即可，代码如下：</p>
<pre class="lang-java" data-nodeid="1641"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.customized;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.support.JpaEntityInformation;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.support.SimpleJpaRepository;
<span class="hljs-keyword">import</span> org.springframework.transaction.annotation.Transactional;
<span class="hljs-keyword">import</span> javax.persistence.EntityManager;
<span class="hljs-meta">@Transactional(readOnly = true)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomerBaseRepository</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span>,<span class="hljs-title">ID</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">SimpleJpaRepository</span>&lt;<span class="hljs-title">T</span>,<span class="hljs-title">ID</span>&gt;  </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> JpaEntityInformation&lt;T, ?&gt; entityInformation;
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> EntityManager em;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CustomerBaseRepository</span><span class="hljs-params">(JpaEntityInformation&lt;T, ?&gt; entityInformation, EntityManager entityManager)</span> </span>{
        <span class="hljs-keyword">super</span>(entityInformation, entityManager);
        <span class="hljs-keyword">this</span>.entityInformation = entityInformation;
        <span class="hljs-keyword">this</span>.em = entityManager;
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CustomerBaseRepository</span><span class="hljs-params">(Class&lt;T&gt; domainClass, EntityManager em)</span> </span>{
        <span class="hljs-keyword">super</span>(domainClass, em);
        entityInformation = <span class="hljs-keyword">null</span>;
        <span class="hljs-keyword">this</span>.em = em;
    }
    <span class="hljs-comment">//覆盖删除方法，实现逻辑删除，换成更新方法</span>
    <span class="hljs-meta">@Transactional</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">(T entity)</span> </span>{
        entity.setDeleted(Boolean.TRUE);
        em.merge(entity);
    }
}
</code></pre>
<p data-nodeid="1642">需要注意的是，这里需要覆盖父类的构造方法，接收 EntityManager，并赋值给自己类里面的私有变量。</p>
<p data-nodeid="1643"><strong data-nodeid="1878">第三步：写一个测试用例测试一下。</strong></p>
<pre class="lang-java" data-nodeid="1644"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testCustomizedBaseRepository</span><span class="hljs-params">()</span> </span>{
    User user = userRepository.findById(<span class="hljs-number">2L</span>).get();
    userRepository.logicallyDelete(user);
    userRepository.delete(user);
    List&lt;User&gt; users = userRepository.findAll();
    Assertions.assertEquals(users.get(<span class="hljs-number">0</span>).getDeleted(),Boolean.TRUE);
}
</code></pre>
<p data-nodeid="1645">你可以发现，我们执行完“删除”之后，数据库里面的 User 还在，只不过 deleted，变成了已删除状态。那么这是为什么呢？我们分析一下原理。</p>
<h4 data-nodeid="1646">原理分析</h4>
<p data-nodeid="1647">还是打开 RepositoryFactory 里面的父类方法，它会根据 @EnableJpaRepositories 里面我们配置的 repositoryBaseClass，加载我们自定义的实现类，关键方法如下：</p>
<p data-nodeid="1648"><img src="https://s0.lgstatic.com/i/image/M00/60/67/Ciqc1F-NS0KACTP-AAHMT_HqmJA240.png" alt="Drawing 9.png" data-nodeid="1884"></p>
<p data-nodeid="1649">我们还看刚才的方法的断点，如下：</p>
<p data-nodeid="1650"><img src="https://s0.lgstatic.com/i/image/M00/60/67/Ciqc1F-NS0iAQ1eDAAFzL2qkapU450.png" alt="Drawing 10.png" data-nodeid="1888"></p>
<p data-nodeid="1651">可以看到 information 已经变成了我们扩展的基类了，而最终生成的 repository 的实现类也换成了 CustomerBaseRepository。</p>
<p data-nodeid="1652">自定义的方法，我们讲完了，那么它都会在哪些实际场景用到呢？接着看一下。</p>
<h3 data-nodeid="1653">实际应用场景是什么？</h3>
<p data-nodeid="1654">在实际工作中，有哪些场景会用到自定义 Repository 呢？</p>
<ol data-nodeid="1655">
<li data-nodeid="1656">
<p data-nodeid="1657">首先肯定是我们做框架的时候、解决一些通用问题的时候，如逻辑删除，正如我们上面的实例所示的样子。</p>
</li>
<li data-nodeid="1658">
<p data-nodeid="1659">在实际生产中经常会有这样的场景：对外暴露的是 UUID 查询方法，而对内暴露的是 Long 类型的 ID，这时候我们就可以自定义一个 FindByIdOrUUID 的底层实现方法，可以选择在自定义的 Respository 接口里面实现。</p>
</li>
<li data-nodeid="1660">
<p data-nodeid="1661">Defining Query Methods 和 @Query 满足不了我们的查询，但是我们又想用它的方法语义的时候，就可以考虑实现不同的 Respository 的实现类，来满足我们不同业务场景的复杂查询。我见过有团队这样用过，不过个人感觉一般用不到，如果你用到了说明你的代码肯定有优化空间，代码不应该过于复杂。</p>
</li>
</ol>
<p data-nodeid="1662">上面我们讲到了逻辑删除，还有一个是利用 @SQLDelete 也可以做到，用法如下：</p>
<pre class="lang-java" data-nodeid="1663"><code data-language="java"><span class="hljs-meta">@SQLDelete(sql = "UPDATE user SET deleted = true where deleted =false and id = ?")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
....
}
</code></pre>
<p data-nodeid="1664">这个时候不需要我们自定义 Respository 也可做到，这个方法的优点就是灵活，而缺点就是需要我们一个一个配置在实体上面。你可以根据实际场景自由选择方式。</p>
<h3 data-nodeid="1665">总结</h3>
<p data-nodeid="3434" class="te-preview-highlight">到这里，本课时的内容也介绍完了。我们通过介绍 EntityManager 和 @EnableJpaRepositories，实现了我们自定义 Repository 的两种方法，你可以学习一下我的分析问题思路，进而应用到自身，学会举一反三。</p>



<p data-nodeid="1667">也希望你踊跃留言，我们一起讨论，一起进步。下一课时我要讲讲实战过程中，我们的基类应该如何设计。</p>
<blockquote data-nodeid="1668">
<p data-nodeid="1669" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="1905">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **龙：
> 老师，你好，如果用自定义repository对BaseEntity实体做逻辑删除，但对非BaseEntity做物理删除，该怎么实现，在delete(T entity)里面判断类型吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以实现两个基类呀。BaseRemoveEntity(物理删除)和BaseDeleteEntity(逻辑删除)

##### *中：
> 平时逻辑删除采用第二种配合使用@SQLDelete@SqlWhere没想到可以自定义repository，so easy。有个简单问题@Table(name="user")public class UserEntity{}name可不可以通过el表达式去取className,增删前后缀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当然可以呀，具体写法可以研究一下，欢迎给老师留言

