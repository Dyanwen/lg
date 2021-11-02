<p data-nodeid="96207">你好，欢迎学习第 21 讲。上一讲我们介绍了 Hibernate 和 JPA 在 Spring Boot 里面的配置项相关内容，那么这一讲其实是对前一讲内容的延续，我们再介绍一下 Hibernate 和 JPA 的一个核心概念 Persistence Context。</p>


<p data-nodeid="95222">这个概念是 JPA 入门者，或者初中级开发人员最容易用错的一部分内容，今天我们就来弄清楚它的来龙去脉，分析原理及用法，帮你更好地掌握，以便熟练运用。我们先从了解核心概念入手。</p>
<h3 data-nodeid="95223">Persistence Context 相关的核心概念有哪些？</h3>
<h4 data-nodeid="95224">EntityManagerFactory 和 Persistence Unit 是什么？</h4>
<p data-nodeid="95225">按照 JPA 协议里面的定义：persistence unit 是一些持久化配置的集合，里面包含了数据源的配置、EntityManagerFactory 的配置，spring 3.1 之前主要是通过 persistence.xml 的方式来配置一个 persistence unit。</p>
<p data-nodeid="95226">而 spring 3.1 之后已经不再推荐这种方式了，但是还保留了 persistence unit 的概念，我们只需要在配置 LocalContainerEntityManagerFactory 的时候，指定 persistence unit 的名字即可，正如我在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=490#/detail/pc?id=4718" data-nodeid="95406">18 | 生产环境多数据源的处理方法有哪些？</a>”中讲解多数据的时候一样。</p>
<p data-nodeid="95227">请看下面代码，我们直接指定 persistenceUnit 的 name 即可。</p>
<pre class="lang-java" data-nodeid="95228"><code data-language="java">    <span class="hljs-meta">@Bean(name = "db2EntityManagerFactory")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> LocalContainerEntityManagerFactoryBean <span class="hljs-title">entityManagerFactory</span><span class="hljs-params">(EntityManagerFactoryBuilder builder, <span class="hljs-meta">@Qualifier("db2DataSource")</span> DataSource db2DataSource)</span> </span>{
        <span class="hljs-keyword">return</span> builder.dataSource(db2DataSource)
            .packages(<span class="hljs-string">"com.example.jpa.example1.db2"</span>) <span class="hljs-comment">//数据2的实体所在的路径</span>
            .persistenceUnit(<span class="hljs-string">"db2"</span>)<span class="hljs-comment">// persistenceUnit的名字采用db2</span>
            .build();
    }
</code></pre>
<p data-nodeid="95229">EntityManagerFactory 的用途就比较明显了，即根据不同的数据源，来管理 Entity 和创建 EntityManger，在整个 application 的生命周期中是单例状态。所以在 spring 的 application 里面获得 EntityManagerFactory 有两种方式。</p>
<p data-nodeid="95230"><strong data-nodeid="95413">第一种：通过 Spring 的 Bean 的方式注入。</strong></p>
<pre class="lang-java" data-nodeid="95231"><code data-language="java"><span class="hljs-meta">@Autowired</span>
<span class="hljs-meta">@Qualifier(value="db2EntityManagerFactory")</span> 
<span class="hljs-keyword">private</span> EntityManagerFactory entityManagerFactory;
</code></pre>
<p data-nodeid="96863">这种方式是我比较推荐的，它利用了 Spring 自身的 Bean 的管理机制。</p>
<p data-nodeid="96864"><strong data-nodeid="96869">第二种：利用 java.persistence.PersistenceUnit 注解的方式获取。</strong></p>

<pre class="lang-java" data-nodeid="95233"><code data-language="java"><span class="hljs-meta">@PersistenceUnit("db2")</span>
<span class="hljs-keyword">private</span> EntityManagerFactory entityManagerFactory;
</code></pre>
<h4 data-nodeid="95234">EntityManager 和 PersistenceContext 是什么？</h4>
<p data-nodeid="95235">按照 JPA 协议的规范，我们先理解一下 PersistenceContext，它是用来管理会话里面的 Entity 状态的一个上下文环境，使 Entity 的实例有了不同的状态，也就是我们所说的实体实例的生命周期。</p>
<p data-nodeid="95236">而这些实体在 PersistenceContext 中的不同状态都是通过 EntityManager 提供的一些方法进行管理的，也就是说：</p>
<ol data-nodeid="95237">
<li data-nodeid="95238">
<p data-nodeid="95239">PersistenceContext 是持久化上下文，是 JPA 协议定义的，而 Hibernate 的实现是通过 Session 创建和销毁的，也就是说一个 Session 有且仅有一个 PersistenceContext；</p>
</li>
<li data-nodeid="95240">
<p data-nodeid="95241">PersistenceContext 既然是持久化上下文，里面管理的是 Entity 的状态；</p>
</li>
<li data-nodeid="95242">
<p data-nodeid="95243">EntityManager 是通过 PersistenceContext 创建的，用来管理 PersistenceContext 中 Entity 状态的方法，离开 PersistenceContext 持久化上下文，EntityManager 没有意义；</p>
</li>
<li data-nodeid="95244">
<p data-nodeid="95245">EntityManger 是操作对象的唯一入口，一个请求里面可能会有多个 EntityManger 对象。</p>
</li>
</ol>
<p data-nodeid="95246">下面我们看一下 PersistenceContext 是怎么创建的。直接打开 SessionImpl 的构造方法，就可以知道 PersistenceContext 是和 Session 的生命周期绑定的，关键代码如下：</p>
<pre class="lang-java" data-nodeid="95247"><code data-language="java"><span class="hljs-comment">//session实例初始化的入口</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SessionImpl</span><span class="hljs-params">(SessionFactoryImpl factory, SessionCreationOptions options)</span> </span>{
   <span class="hljs-keyword">super</span>( factory, options );
   <span class="hljs-comment">//Session里面创建了persistenceContext，每次session都是新对象</span>
   <span class="hljs-keyword">this</span>.persistenceContext = createPersistenceContext();
......省略一些不重要的代码   
<span class="hljs-function"><span class="hljs-keyword">protected</span> StatefulPersistenceContext <span class="hljs-title">createPersistenceContext</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> StatefulPersistenceContext( <span class="hljs-keyword">this</span> );
}
<span class="hljs-comment">//StatefulPersistenceContext就是PersistenceContext的实现类</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StatefulPersistenceContext</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">PersistenceContext</span> </span>{......}
</code></pre>
<p data-nodeid="95248">我们通过上面的讲述，知道了 PersistenceContext 的创建和销毁机制，那么 EntityManger 如何获得呢？需要通过 @PersistenceContext 的方式进行获取，代码如下：</p>
<pre class="lang-java" data-nodeid="95249"><code data-language="java"><span class="hljs-meta">@PersistenceContext</span>
<span class="hljs-keyword">private</span> EntityManager em;
</code></pre>
<p data-nodeid="95250">而其中 @PersistenceContext 的属性配置有如下这些。</p>
<pre class="lang-java" data-nodeid="95251"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> PersistenceContext {
    <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
    <span class="hljs-comment">//PersistenceContextUnit的名字，多数据源的时候有用</span>
    <span class="hljs-function">String <span class="hljs-title">unitName</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
    <span class="hljs-comment">//是指创建的EntityManager的生命周期是存在事务内还是可以跨事务，默认为生命周期和事务一样；</span>
    <span class="hljs-function">PersistenceContextType <span class="hljs-title">type</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> PersistenceContextType.TRANSACTION</span>;
    <span class="hljs-comment">//同步的类型：只有SYNCHRONIZED和UNSYNCHRONIZED两个值用来表示，但开启事务的时候是否自动加入已开启的事务里面，默认SYNCHRONIZED表示自动加入，不创建新的事务。而UNSYNCHRONIZED表示，不自动加入上下文已经有的事务，自动开启新的事务；这里你使用的时候需要注意看一下事务的日志；</span>
    <span class="hljs-function">SynchronizationType <span class="hljs-title">synchronization</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> SynchronizationType.SYNCHRONIZED</span>;
    <span class="hljs-comment">//持久化的配置属性，这里指我们上一课时讲过的hibernate中AvailableSettings里面的值</span>
    PersistenceProperty[] properties() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="95252">一般情况下保持默认即可，你也可以根据实际情况自由组合，我再举个复杂点的例子。</p>
<pre class="lang-java" data-nodeid="95253"><code data-language="java"><span class="hljs-meta">@PersistenceContext(
      unitName = "db2",//采用数据源2的
      //可以跨事务的EntityManager
      type = PersistenceContextType.EXTENDED,
      properties = {
            //通过properties改变一下自动flush的机制
            @PersistenceProperty(
                  name="org.hibernate.flushMode",
                  value= "MANUAL"//改成手动刷新方式
            )
      }
)</span>
<span class="hljs-keyword">private</span> EntityManager entityManager;
</code></pre>
<p data-nodeid="95254">以上就是 Persistence Context 的相关基础概念。其中，实体的生命周期指的是什么呢？我们来了解一下。</p>
<h3 data-nodeid="95255">实体对象的生命周期</h3>
<p data-nodeid="98172">既然 PersistenceContext 是存储 Entity 的，那么 Entity 在 PersistenceContext 里面肯定有不同的状态。对此，JPA 协议定义了四种状态：new、manager、detached、removed。我们通过一个图来整体认识一下。</p>
<p data-nodeid="98834"><img src="https://s0.lgstatic.com/i/image/M00/70/03/CgqCHl-3m3OAPiQmAAB8FdvAFnE298.png" alt="Drawing 0.png" data-nodeid="98838"></p>
<div data-nodeid="98835" class=""><p style="text-align:center">注：图片来自网络</p></div>





<h4 data-nodeid="95259">第一种：New 状态的对象</h4>
<p data-nodeid="95260">当我们使用关键字 new 的时候创建的实体对象，称为 new 状态的 Entity 对象。它需要同时满足两个条件：new 状态的实体 Id 和 Version 字段都是 null；new 状态的实体没有在 PersistenceContext 中出现过。</p>
<p data-nodeid="95261" class="">那么如果我们要把 new 状态的 Entity 放到 PersistenceContext 里面，有两种方法：执行 entityManager.persist(entity) 方法；通过关联关系的实体关系配置 cascade=PERSIST or cascade=ALL 这种类型，并且关联关系的一方，也执行了 entityManager.persist(entity) 方法。</p>
<p data-nodeid="95262">我们使用一个案例来说明一下。</p>
<pre class="lang-java" data-nodeid="95263"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testPersist</span><span class="hljs-params">()</span> </span>{
    UserInfo userInfo = UserInfo.builder().lastName(<span class="hljs-string">"jack"</span>).build();
    <span class="hljs-comment">//通过contains方法可以验证对象是否在PersistenceContext里面，此时不在</span>
    Assertions.assertFalse(entityManager.contains(userInfo));
    <span class="hljs-comment">//通过persist方法把对象放到PersistenceContext里面</span>
    entityManager.persist(userInfo);
    <span class="hljs-comment">//通过contains方法可以验证对象是否在PersistenceContext里面，此时在</span>
    Assertions.assertTrue(entityManager.contains(userInfo));
    Assertions.assertNotNull(userInfo.getId());
}
</code></pre>
<p data-nodeid="95264">这就是 new 状态的实体对象，我们再来看一下和它类似的 Deteched 状态的对象。</p>
<h4 data-nodeid="95265">第二种：Detached（游离）的实体对象</h4>
<p data-nodeid="95266">Detached 状态的对象表示和 PersistenceContext 脱离关系的 Entity 对象。它和 new 状态的对象的不同点在于：</p>
<ul data-nodeid="95267">
<li data-nodeid="95268">
<p data-nodeid="95269">Detached 是 new 状态的实体对象没有持久化 ID（即没有 ID 和 version）；</p>
</li>
<li data-nodeid="95270">
<p data-nodeid="95271">变成持久化对象需要进行 merger 操作，merger 操作会 copy 一个新的实体对象，然后把新的实体对象变成 Manager 状态。</p>
</li>
</ul>
<p data-nodeid="95272">而 Detached 和 new 状态的对象相同点也有两个方面：</p>
<ul data-nodeid="95273">
<li data-nodeid="95274">
<p data-nodeid="95275">都和 PersistenceContext 脱离了关系；</p>
</li>
<li data-nodeid="95276">
<p data-nodeid="95277">当执行 flush 操作或者 commit 操作的时候，不会进行数据库同步。</p>
</li>
</ul>
<p data-nodeid="95278">如果想让 Manager(persist) 状态的对象从 PersistenceContext 里面游离出来变成 Detached 的状态，可以通过 EntityManager 的 Detach 方法实现，如下面这行代码。</p>
<pre class="lang-java" data-nodeid="95279"><code data-language="java">entityManager.detach(entity);
</code></pre>
<p data-nodeid="95280">当执行完 entityManager.clear()、entityManager.close()，或者事务 commit()、事务 rollback() 之后，所有曾经在 PersistenceContext 里面的实体都会变成 Detached 状态。</p>
<p data-nodeid="95281">而游离状态的对象想回到 PersistenceContext 里面变成 manager 状态的话，只能执行 entityManager 的 merge 方法，也就是下面这行代码。</p>
<pre class="lang-java" data-nodeid="95282"><code data-language="java">entityManager.merge(entity);
</code></pre>
<p data-nodeid="95283">游离状态的实体执行 EntityManager 中 persist 方法的时候就会报异常，我们举个例子：</p>
<pre class="lang-java" data-nodeid="95284"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testMergeException</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">//通过new的方式构建一个游离状态的对象</span>
    UserInfo userInfo = UserInfo.builder().id(<span class="hljs-number">1L</span>).lastName(<span class="hljs-string">"jack"</span>).version(<span class="hljs-number">1</span>).build();
    <span class="hljs-comment">//验证是否存在于persistence context 里面，new的肯定不存在</span>
    Assertions.assertFalse(entityManager.contains(userInfo));
    <span class="hljs-comment">//当执行persist方法的时候就会报异常</span>
    Assertions.assertThrows(PersistentObjectException.class,()-&gt;entityManager.persist(userInfo));
    //detached状态的实体通过merge的方式保存在了persistence context里面
   UserInfo user2 = entityManager.merge(userInfo);
    //验证一下存在于持久化上下文里面
    Assertions.assertTrue(entityManager.contains(user2));
}
</code></pre>
<p data-nodeid="95285">以上就是 new 和 Detached 状态的实体对象，我们再来看第三种——Manager 状态的实体又是什么样的呢？</p>
<h4 data-nodeid="95286">第三种：Manager（persist） 状态的实体</h4>
<p data-nodeid="95287">Manager 状态的实体，顾名思义，是指在 PersistenceContext 里面管理的实体，而此种状态的实体当我们执行事务的 commit()，或者 entityManager 的 flush 方法的时候，就会进行数据库的同步操作。可以说是和数据库的数据有映射关系。</p>
<p data-nodeid="95288">New 状态如果要变成 Manager 的状态，需要执行 persist 方法；而 Detached 状态的实体如果想变成 Manager 的状态，则需要执行 merge 方法。在 session 的生命周期中，任何从数据库里面查询到的 Entity 都会自动成为 Manager 的状态，如 entityManager.findById(id)、entityManager.getReference 等方法。</p>
<p data-nodeid="95289">而 Manager 状态的 Entity 要同步到数据库里面，必须执行 EntityManager 里面的 flush 方法。也就是说我们对 Entity 对象做的任何增删改查，必须通过 entityManager.flush() 执行之后才会变成 SQL 同步到 DB 里面。什么意思呢？我们看个例子。</p>
<pre class="lang-java" data-nodeid="95290"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-meta">@Rollback(value = false)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testManagerException</span><span class="hljs-params">()</span> </span>{
    UserInfo userInfo = UserInfo.builder().lastName(<span class="hljs-string">"jack"</span>).build();
    entityManager.persist(userInfo);
    System.out.println(<span class="hljs-string">"没有执行 flush()方法，产生insert sql"</span>);
    entityManager.flush();
    System.out.println(<span class="hljs-string">"执行了flush()方法，产生了insert sql"</span>);
    Assertions.assertTrue(entityManager.contains(userInfo));
}
</code></pre>
<p data-nodeid="95291">执行完之后，我们可以看到如下输出：</p>
<pre class="lang-java" data-nodeid="95292"><code data-language="java">没有执行 flush()方法，产生insert sql
Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user_info</span> <span class="hljs-params">(create_time, create_user_id, last_modified_time, last_modified_user_id, version, ages, email_address, last_name, telephone, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?, ?, ?, ?, ?, ?)</span>
执行了<span class="hljs-title">flush</span><span class="hljs-params">()</span>方法，产生了insert sql
</span></code></pre>
<p data-nodeid="95293">那么这个时候你可能会问了，并没有看到我们在之前写的 Repository 例子里面手动执行过任何 flush() 操作呀，那么请你带着这个问题继续往下看。了解下实体的第四个状态：Removed。</p>
<h4 data-nodeid="95294">第四种：Removed 的实体状态</h4>
<p data-nodeid="95295">Removed 的状态，顾名思义就是指删除了的实体，但是此实体还在 PersistenceContext 里面，只是在其中表示为 Removed 的状态，它和 Detached 状态的实体最主要的区别就是不在 PersistenceContext 里面，但都有 ID 属性。</p>
<p data-nodeid="95296">而 Removed 状态的实体，当我们执行 entityManager.flush() 方法的时候，就会生成一条 delete 语句到数据库里面。Removed 状态的实体，在执行 flush() 方法之前，执行 entityManger.persist(removedEntity) 方法时候，就会去掉删除的表示，变成 Managed 的状态实例。我们还是看个例子。</p>
<pre class="lang-java" data-nodeid="95297"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testDelete</span><span class="hljs-params">()</span> </span>{
    UserInfo userInfo = UserInfo.builder().lastName(<span class="hljs-string">"jack"</span>).build();
    entityManager.persist(userInfo);
    entityManager.flush();
    System.out.println(<span class="hljs-string">"执行了flush()方法，产生了insert sql"</span>);           
    entityManager.remove(userInfo);
    entityManager.flush();
    System.out.println(<span class="hljs-string">"执行了flush()方法之后，又产生了delete sql"</span>);
}
</code></pre>
<p data-nodeid="95298">执行完之后可以看到如下日志：</p>
<pre class="lang-java" data-nodeid="95299"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user_info</span> <span class="hljs-params">(create_time, create_user_id, last_modified_time, last_modified_user_id, version, ages, email_address, last_name, telephone, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?, ?, ?, ?, ?, ?)</span>
执行了<span class="hljs-title">flush</span><span class="hljs-params">()</span>方法，产生了insert sql
Hibernate: delete from user_info where id</span>=? and version=?
执行了flush()方法之后，又产生了delete sql
</code></pre>
<p data-nodeid="95300">到这里四种实体对象的状态就介绍完了，通过上面的详细解释，你知道了 Entity 的不同状态的时机是什么样的、不同状态直接的转化方式是什么样的，并且知道实体状态的任何变化都是在 Persistence Context 中进行的，和数据一点关系没有。</p>
<p data-nodeid="95301">这仅仅是 JPA 和 Hibernate 为了提高方法执行的性能而设计的缓存实体机制，也是 JPA 和 MyBatis 的主要区别之处。</p>
<p data-nodeid="95302">MyBatis 是对数据库的操作所见即所得的模式；而使用 JPA，你的任何操作都不会产生 DB 的sql。那么什么时间才能进行 DB 的 sql 操作呢？我们看一下 flush 的实现机制。</p>
<h3 data-nodeid="95303">解密 EntityManager 的 flush() 方法</h3>
<p data-nodeid="95304">flush 方法的用法很简单，就是我们在需要 DB 同步 sql 执行的时候，执行 entityManager.flush() 即可，它的作用如下所示。</p>
<h4 data-nodeid="95305">Flush 的作用</h4>
<p data-nodeid="95306">flush 重要的、唯一的作用，就是将 Persistence Context 中变化的实体转化成 sql 语句，同步执行到数据库里面。换句话来说，如果我们不执行 flush() 方法的话，通过 EntityManager 操作的任何 Entity 过程都不会同步到数据库里面。</p>
<p data-nodeid="95307">而 flush() 方法很多时候不需要我们手动操作，这里我直接通过 entityManager 操作 flush() 方法，仅仅是为了向你演示执行过程。实际工作中很少会这样操作，而是会直接利用 JPA 和 Hibernate 底层框架帮我们实现的自动 flush 的机制。</p>
<h4 data-nodeid="95308">Flush 的机制是什么？</h4>
<p data-nodeid="95309">JPA 协议规定了 EntityManager 可以通过如下方法修改 FlushMode。</p>
<pre class="lang-java" data-nodeid="95310"><code data-language="java"><span class="hljs-comment">//entity manager 里面提供的修改FlushMode的方法</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setFlushMode</span><span class="hljs-params">(FlushModeType flushMode)</span></span>;
<span class="hljs-comment">//FlushModeType只有两个值，自动和事务提交之前</span>
<span class="hljs-keyword">public</span> <span class="hljs-keyword">enum</span> FlushModeType {
    <span class="hljs-comment">//事务commit之前</span>
   COMMIT,
    <span class="hljs-comment">//自动规则，默认</span>
   AUTO
}
</code></pre>
<p data-nodeid="95311">而 Hiberbernate 还提供了一种手动触发的机制，可以通过如下代码的方式进行修改。</p>
<pre class="lang-java" data-nodeid="95312"><code data-language="java"><span class="hljs-meta">@PersistenceContext(properties = {@PersistenceProperty(
        name = "org.hibernate.flushMode",
        value = "MANUAL"//手动flush
)})</span>
<span class="hljs-keyword">private</span> EntityManager entityManager;
</code></pre>
<p data-nodeid="95313">手动和 commit 的时候很好理解，就是手动执行 flush 方法，像我们案例中的写法一样；事务就是代码在执行事务 commit 的时候，必须要执行 flush() 方法，否则怎么将 PersistenceContext 中变化了的对象同步到数据库里面呢？下面我重点说一下 flush 的自动机制。</p>
<p data-nodeid="95314"><strong data-nodeid="95480">Flush 的自动机制</strong></p>
<p data-nodeid="95315">默认情况下，JPA 和 Hibernate 都是采用的 AUTO 的 Flush 机制，自动触发的规则如下：</p>
<ol data-nodeid="95316">
<li data-nodeid="95317">
<p data-nodeid="95318">事务 commit 之前，即指执行 transactionManager.commit() 之前都会触发，这个很好理解；</p>
</li>
<li data-nodeid="95319">
<p data-nodeid="95320">执行任何的 JPQL 或者 native SQL（代替直接操作 Entity 的方法）都会触发 flush。这句话怎么理解呢？我们举个例子。</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="95321"><code data-language="java">    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testPersist</span><span class="hljs-params">()</span> </span>{
        UserInfo userInfo = UserInfo.builder().lastName(<span class="hljs-string">"jack"</span>).build();
        <span class="hljs-comment">//通过contains方法可以验证对象是否在PersistenceContext里面，此时不在</span>
        Assertions.assertFalse(entityManager.contains(userInfo));
        <span class="hljs-comment">//通过persist方法把对象放到PersistenceContext里面</span>
        entityManager.persist(userInfo);<span class="hljs-comment">//是直接操作Entity的，不会触发flush操作</span>
        <span class="hljs-comment">//entityManager.remove(userInfo);//是直接操作Entity的，不会触发flush操作</span>
        System.out.println(<span class="hljs-string">"没有执行 flush()方法，产生insert sql"</span>);
        UserInfo userInfo2 = entityManager.find(UserInfo.class,<span class="hljs-number">2L</span>);<span class="hljs-comment">//是直接操作Entity的，这个就不会触发flush操作</span>
<span class="hljs-comment">//        userInfoRepository.queryByFlushTest();//是操作JPQL的，这个就会先触发flush操作；</span>
        System.out.println(<span class="hljs-string">"flush()方法，产生insert sql"</span>);
        <span class="hljs-comment">//通过contains方法可以验证对象是否在PersistenceContext里面，此时在</span>
        Assertions.assertTrue(entityManager.contains(userInfo));
        Assertions.assertNotNull(userInfo.getId());
    }
</code></pre>
<p data-nodeid="95322">而只有执行类似 .queryByFlushTest() 这个方法的时候，才会触发 flush，因为它是用的 JPQL 的机制执行的。</p>
<p data-nodeid="95323">上面的方法触发了 flush 的日志，会输出如下格式，你可以看到这里多了一个 insert 语句。</p>
<pre class="lang-java" data-nodeid="95324"><code data-language="java">没有执行 flush()方法，产生insert sql
Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user_info</span> <span class="hljs-params">(create_time, create_user_id, last_modified_time, last_modified_user_id, version, ages, email_address, last_name, telephone, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?, ?, ?, ?, ?, ?)</span>
Hibernate: select userinfo0_.id as id1_0_, userinfo0_.create_time as create_t2_0_, userinfo0_.create_user_id as create_u3_0_, userinfo0_.last_modified_time as last_mod4_0_, userinfo0_.last_modified_user_id as last_mod5_0_, userinfo0_.version as version6_0_, userinfo0_.ages as ages7_0_, userinfo0_.email_address as email_ad8_0_, userinfo0_.last_name as last_nam9_0_, userinfo0_.telephone as telepho10_0_ from user_info userinfo0_ where userinfo0_.id</span>=<span class="hljs-number">2</span>
flush()方法，产生insert sql
</code></pre>
<p data-nodeid="95325">没有触发 flush 的日志输出的是如下格式，其中没有 insert 语句。</p>
<pre class="lang-java" data-nodeid="95326"><code data-language="java">没有执行 flush()方法，产生insert sql
Hibernate: select userinfo0_.id as id1_0_0_, userinfo0_.create_time as create_t2_0_0_, userinfo0_.create_user_id as create_u3_0_0_, userinfo0_.last_modified_time as last_mod4_0_0_, userinfo0_.last_modified_user_id as last_mod5_0_0_, userinfo0_.version as version6_0_0_, userinfo0_.ages as ages7_0_0_, userinfo0_.email_address as email_ad8_0_0_, userinfo0_.last_name as last_nam9_0_0_, userinfo0_.telephone as telepho10_0_0_ from user_info userinfo0_ where userinfo0_.id=?
flush()方法，产生insert sql
</code></pre>
<p data-nodeid="95327">我们了解完了 flush 的自动触发机制还不够，因为 flush 的自动刷新机制还会改变 update、insert、delete 的执行顺序。</p>
<h4 data-nodeid="95328">Flush 的时候会改变 SQL 的执行顺序</h4>
<p data-nodeid="95329">flush() 方法调用之后，同一个事务内，sql 的执行顺序会变成如下模式：insert 的先执行、delete 的第二个执行、update 的第三个执行。我们举个例子，方法如下：</p>
<pre class="lang-java" data-nodeid="95330"><code data-language="java">entityManager.remove(u3);
UserInfo userInfo = UserInfo.builder().lastName(<span class="hljs-string">"jack"</span>).build();
entityManager.persist(userInfo);
</code></pre>
<p data-nodeid="95331">看一下执行的 sql 会变成如下模样，即先 insert 后 delete。</p>
<pre class="lang-java" data-nodeid="95332"><code data-language="java">Hibernate: insert into user_info 。。。。。。
Hibernate: delete from user_info where id=? and version=?
</code></pre>
<p data-nodeid="95333">这种会改变顺序的现象，主要是由 persistence context 的实体状态机制导致的，所以在 Hibernate 的环境中，顺序会变成如下的 ActionQueue 的模式：</p>
<ol data-nodeid="95334">
<li data-nodeid="95335">
<p data-nodeid="95336"><code data-backticks="1" data-nodeid="95492">OrphanRemovalAction</code></p>
</li>
<li data-nodeid="95337">
<p data-nodeid="95338"><code data-backticks="1" data-nodeid="95493">EntityInsertAction</code>or<code data-backticks="1" data-nodeid="95495">EntityIdentityInsertAction</code></p>
</li>
<li data-nodeid="95339">
<p data-nodeid="95340"><code data-backticks="1" data-nodeid="95496">EntityUpdateAction</code></p>
</li>
<li data-nodeid="95341">
<p data-nodeid="95342"><code data-backticks="1" data-nodeid="95497">CollectionRemoveAction</code></p>
</li>
<li data-nodeid="95343">
<p data-nodeid="95344"><code data-backticks="1" data-nodeid="95498">CollectionUpdateAction</code></p>
</li>
<li data-nodeid="95345">
<p data-nodeid="95346"><code data-backticks="1" data-nodeid="95499">CollectionRecreateAction</code></p>
</li>
<li data-nodeid="95347">
<p data-nodeid="95348"><code data-backticks="1" data-nodeid="95500">EntityDeleteAction</code></p>
</li>
</ol>
<p data-nodeid="95349">flush 的作用你已经知道了，它会把 sql 同步执行到数据库里面。但是需要注意的是，虽然 sql 到数据库里面执行了，那么最终数据是不是持久化，是不是被其他事务看到还会受到控制呢？Flush 与事务 Commit 的关系如何？</p>
<h4 data-nodeid="95350">Flush 与事务 Commit 的关系</h4>
<p data-nodeid="95351">大概有以下几点：</p>
<ol data-nodeid="95352">
<li data-nodeid="95353">
<p data-nodeid="95354">在当前的事务执行 commit 的时候，会触发 flush 方法；</p>
</li>
<li data-nodeid="95355">
<p data-nodeid="95356">在当前的事务执行完 commit 的时候，如果隔离级别是可重复读的话，flush 之后执行的 update、insert、delete 的操作，会被其他的新事务看到最新结果；</p>
</li>
<li data-nodeid="95357">
<p data-nodeid="95358">假设当前的事务是可重复读的，当我们手动执行 flush 方法之后，没有执行事务 commit 方法，那么其他事务是看不到最新值变化的，但是最新值变化对当前没有 commit 的事务是有效的；</p>
</li>
<li data-nodeid="95359">
<p data-nodeid="95360">如果执行了 flush 之后，当前事务发生了 rollback 操作，那么数据将会被回滚（数据库的机制）。</p>
</li>
</ol>
<p data-nodeid="95361">以上介绍的都是 flush 的机制，那么 **Repository 里面的 saveAndFlush 有什么作用呢？</p>
<h4 data-nodeid="95362">saveAndFlush 和 save 的区别</h4>
<p data-nodeid="95363">细心的同学会发现 **Repository 里面有一个 saveAndFlush(entity); 的方法，我们通过查看可以发现如下内容：</p>
<pre class="lang-java" data-nodeid="95364"><code data-language="java"><span class="hljs-meta">@Transactional</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">saveAndFlush</span><span class="hljs-params">(S entity)</span> </span>{
  <span class="hljs-comment">//执行了save方法之后，调用了flush()方法</span>
   S result = save(entity);
   flush();
   <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="95365">而另一个 **Repository 里面的 save 的方法，我们查看其源码如下：</p>
<pre class="lang-java" data-nodeid="95366"><code data-language="java"><span class="hljs-comment">//没有做flush操作，只是，执行了persist或者merge的操作</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span> </span>{
   <span class="hljs-keyword">if</span> (entityInformation.isNew(entity)) {
      em.persist(entity);
      <span class="hljs-keyword">return</span> entity;
   } <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span> em.merge(entity);
   }
}
</code></pre>
<p data-nodeid="95367">所以这个时候我们应该很清楚 Repository 里面提供的 saveAndFlush 和 save 的区别，有如下几点：</p>
<ol data-nodeid="95368">
<li data-nodeid="95369">
<p data-nodeid="95370">saveAndFlush 执行完，再执行 flush，会刷新整个 PersistenceContext 里面的实体并进入到数据库里面，那么当我们频繁调用 saveAndFlush 就失去了 cache 的意义，这个时候就和执行 mybatis 的 saveOrUpdate 是一样的效果；</p>
</li>
<li data-nodeid="95371">
<p data-nodeid="95372">当多次调用相同的 save 方法的时候，最终 flush 执行只会产生一条 sql，在性能上会比 saveAndFlush 高一点；</p>
</li>
<li data-nodeid="95373">
<p data-nodeid="95374">不管是 saveAndFlush 还是 save，都受当前事务控制，事务在没有 commit 之前，都只会影响当前事务的操作；</p>
</li>
</ol>
<p data-nodeid="95375">综上，两种本质的区别就是 flush 执行的时机不一样而已，对数据库中数据的事务一致性没有任何影响。然而有的时候，即使我们调用了 flush 的方法也是一条 sql 都没有，为什么呢？我们再来了解一个概念：Dirty。</p>
<h3 data-nodeid="95376">Entity 的 Dirty 判断逻辑及其作用</h3>
<p data-nodeid="95377">在 PersistenceContext 里面还有一个重要概念，就是当实体不是 Dirty 状态，也就是没有任何变化的时候，是不会进行任何 db 操作的。所以即使我们执行 flush 和 commit，实体没有变化，就没有必要执行，这也能大大减少数据库的压力。</p>
<p data-nodeid="95378">下面通过一个例子，认识一下 Dirty 的效果。</p>
<h4 data-nodeid="95379">Dirty 的效果的例子</h4>
<pre class="lang-java" data-nodeid="95380"><code data-language="java"><span class="hljs-comment">//我们假设数据库里面存在一条id=1的数据，我们不做任何改变执行save或者saveAndFlush，除了select之外，不会产生任何sql语句；</span>
<span class="hljs-meta">@Test</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-meta">@Rollback(value = false)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testDirty</span><span class="hljs-params">()</span> </span>{
    UserInfo userInfo = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
    userInfoRepository.saveAndFlush(userInfo);
    userInfoRepository.save(userInfo);
}
</code></pre>
<p data-nodeid="95381">那么当我们尝试改变一下 userInfo 里面的值，当执行如下方法的时候就会产生 update 的 sql。</p>
<pre class="lang-java" data-nodeid="95382"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-meta">@Rollback(value = false)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testDirty</span><span class="hljs-params">()</span> </span>{
    UserInfo userInfo = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
    userInfo.setLastName(<span class="hljs-string">"jack_test_dirty"</span>);
    userInfoRepository.saveAndFlush(userInfo);
}
</code></pre>
<p data-nodeid="95383">那么实体的 Dirty 判断过程是什么样的呢？我们通过源码来看一下。</p>
<h4 data-nodeid="95384">Entity 判断 Dirty 的过程</h4>
<p data-nodeid="102081">如果我们通过 debug 一步一步分析的话可以找到，DefaultFlushEntityEventListener 的源码里面 isUpdateNecessary 的关键方法如下所示：</p>
<p data-nodeid="102082" class=""><img src="https://s0.lgstatic.com/i/image/M00/70/03/CgqCHl-3m6uAEoT1AAEkZaYyxn8305.png" alt="Drawing 1.png" data-nodeid="102086"></p>






<p data-nodeid="103383">我们进一步 debug 看 dirtyCheck 的实现，可以看发现如下关键点，从而找出发生变化的 proerties。</p>
<p data-nodeid="103384" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/6F/F8/Ciqc1F-3m7GAUkFiAAEtQI1J900876.png" alt="Drawing 2.png" data-nodeid="103388"></p>


<p data-nodeid="95389">我们再仔细看 persister.findDirty（values, loadedState, entity, session），可以看出来源码里面是通过一个字段一个字段比较的，所以可以知道 PsersistenceContext 中的前后两个 Entity 的哪些字段发生了变化。因此当我们执行完 save 之后，没有产生任何 sql（因为没有变化）。你知道了这个原理之后，就不用再为此“大惊小怪”了。</p>
<p data-nodeid="95390">总结起来就是，在 flush 的时候，Hibernate 会一个个判断实体的前后对象中哪个属性发生变化了，如果没有发生变化，则不产生 update 的 sql 语句；只有变化才会才生 update sql，并且可以做到同一个事务里面的多次 update 合并，从而在一定程度上可以减轻 DB 的压力。</p>
<h3 data-nodeid="95391">总结</h3>
<p data-nodeid="95392">这一讲我为你介绍了 PersistenceContext 的概念、EntityManager 的作用，以及 flush 操作是什么时机进行的，它和事务的关系如何。其中也夹杂了我多年来的经验分享，希望你可以从头到尾学下来。</p>
<p data-nodeid="95393">如果你能完全理解这一讲的内容，那么对于 JPA 和 Hibernate 的核心原理你算是掌握一大半了，没掌握的同学的也不要紧，跟着我的操作，每一篇都仔细探究，你也会逐渐掌握这门技术的，加油吧！</p>
<p data-nodeid="95394">下一课时我会为你介绍 PersistenceContext 的容器 Session 相关的概念，到时见。</p>
<blockquote data-nodeid="95395">
<p data-nodeid="95396">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="95548">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *奇：
> 老师，关于本节中讲到，"PersistenceContext 是持久化上下文，是 JPA 协议定义的，而 Hibernate 的实现是通过 Session 创建和销毁的，也就是说一个 Session 有且仅有一个 PersistenceContext" 这句话，我有一个疑问，这里的Session指的是Web对象生命周期中的Session吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Session是指的Hibernate里面的Session

