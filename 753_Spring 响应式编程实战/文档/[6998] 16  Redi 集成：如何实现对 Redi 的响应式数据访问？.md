<p data-nodeid="418" class="">上一讲，我们介绍了 Spring Data MongoDB Reactive 组件，它是 Spring Data MongoDB 的响应式版本。今天我们要讨论的是 Spring Data Redis Reactive 组件，它专门针对 Redis 这款 NoSQL 数据库提供了响应式编程能力。</p>
<p data-nodeid="419">使用该组件的步骤与 MongoDB 类似，我们同样围绕开发环境的初始化、Repository 的创建以及与 Service 层之间的集成这些步骤展开讨论，并结合 ReactiveSpringCSS 案例来集成这款主流的缓存中间件。</p>
<h3 data-nodeid="420">Spring Data Redis Reactive 技术栈</h3>
<p data-nodeid="421">我们可以通过导入 spring-boot-starter-data-redis-reactive 依赖来集成 Spring Data Reactive Redis 模块。与上一讲介绍的 MongoDB 不同，Redis 不提供响应式存储库，也就没有 ReactiveMongoRepository 这样的工具类可以直接使用。因此，ReactiveRedisTemplate 类成为响应式 Redis 数据访问的核心工具类。与 ReactiveMongoTemplate 和 ReactiveMongoOperations 之间的关系类似，ReactiveRedisTemplate 模板类实现了 ReactiveRedisOperations 接口定义的 API。</p>
<p data-nodeid="422">ReactiveRedisOperations 中定义了一批针对 Redis 各种数据结构的操作方法，如下所示。</p>
<pre class="lang-java" data-nodeid="423"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ReactiveRedisOperations</span>&lt;<span class="hljs-title">K</span>, <span class="hljs-title">V</span>&gt; </span>{
&nbsp;
    &lt;HK, HV&gt; <span class="hljs-function">ReactiveHashOperations&lt;K, HK, HV&gt; <span class="hljs-title">opsForHash</span><span class="hljs-params">()</span></span>;
	&lt;K, HK, HV&gt; <span class="hljs-function">ReactiveHashOperations&lt;K, HK, HV&gt; <span class="hljs-title">opsForHash</span><span class="hljs-params">(RedisSerializationContext&lt;K, ?&gt; serializationContext)</span></span>;
&nbsp;
    <span class="hljs-function">ReactiveListOperations&lt;K, V&gt; <span class="hljs-title">opsForList</span><span class="hljs-params">()</span></span>;
	&lt;K, V&gt; <span class="hljs-function">ReactiveListOperations&lt;K, V&gt; <span class="hljs-title">opsForList</span><span class="hljs-params">(RedisSerializationContext&lt;K, V&gt; serializationContext)</span></span>;
&nbsp;
    <span class="hljs-function">ReactiveSetOperations&lt;K, V&gt; <span class="hljs-title">opsForSet</span><span class="hljs-params">()</span></span>;
	&lt;K, V&gt; <span class="hljs-function">ReactiveSetOperations&lt;K, V&gt; <span class="hljs-title">opsForSet</span><span class="hljs-params">(RedisSerializationContext&lt;K, V&gt; serializationContext)</span></span>;
&nbsp;
    <span class="hljs-function">ReactiveValueOperations&lt;K, V&gt; <span class="hljs-title">opsForValue</span><span class="hljs-params">()</span></span>;
	&lt;K, V&gt; <span class="hljs-function">ReactiveValueOperations&lt;K, V&gt; <span class="hljs-title">opsForValue</span><span class="hljs-params">(RedisSerializationContext&lt;K, V&gt; serializationContext)</span></span>;
&nbsp;
    <span class="hljs-function">ReactiveZSetOperations&lt;K, V&gt; <span class="hljs-title">opsForZSet</span><span class="hljs-params">()</span></span>;
	&lt;K, V&gt; <span class="hljs-function">ReactiveZSetOperations&lt;K, V&gt; <span class="hljs-title">opsForZSet</span><span class="hljs-params">(RedisSerializationContext&lt;K, V&gt; serializationContext)</span></span>;
&nbsp;
…
}
</code></pre>
<p data-nodeid="424">上述方法分别对应了 Redis 中 String、List、Set、ZSet 和 Hash 这五种常见的数据结构。同时，我们还看到了一个用于序列化管理的上下文对象 RedisSerializationContext。ReactiveRedisTemplate 提供了所有必需的序列化/反序列化过程，所支持的序列化方式包括常见的 Jackson2JsonRedisSerializer、JdkSerializationRedisSerializer、StringRedisSerializer 等。</p>
<p data-nodeid="425">除了序列化管理，ReactiveRedisTemplate 的另一个核心功能是完成自动化的连接过程管理。应用程序想要访问 Redis，就需要获取 RedisConnection，而获取 RedisConnection 的途径是通过注入 RedisConnectionFactory 到 ReactiveRedisTemplate 中，该 ReactiveRedisTemplate 就能获取 RedisConnection。</p>
<p data-nodeid="426">在 Redis 中，常见的 ConnectionFactory 有两种，一种是传统的 JedisConnectionFactory，而另一种就是新型的 LettuceConnectionFactory。LettuceConnectionFactory 基于 Netty 创建连接实例，可以在多个线程间实现线程安全，满足多线程环境下的并发访问要求。更为重要的是，LettuceConnectionFactory 同时支持响应式的数据访问用法，它是 ReactiveRedisConnectionFactory 接口的一种实现。Lettuce 也是目前 Redis 唯一的响应式 Java 连接器。Lettuce 4.x 版本使用 RxJava 作为底层响应式流实现方案。但是，该库的 5.x 分支切换到了 Project Reactor。</p>
<h3 data-nodeid="427">应用 Reactive Redis</h3>
<p data-nodeid="428">在你对这一组件有一定的解了之后，接下来，我将要为你介绍使用 Spring Data Redis Reactive 进行系统开发的具体过程，首先要说的同样是初始化运行环境。</p>
<h4 data-nodeid="429">初始化 Reactive Redis 运行环境</h4>
<p data-nodeid="430">首先我们在 pom 文件中添加 spring-boot-starter-data-redis-reactive 依赖，如下所示。</p>
<pre class="lang-java" data-nodeid="431"><code data-language="java">&lt;dependency&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp;  &lt;artifactId&gt;spring-boot-starter-data-redis-reactive&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="1370">然后我们通过 Maven 可以得到如下图所示的组件依赖图，可以看到 spring-boot-starter-data-redis-reactive 组件同时依赖于 spring-data-redis 和 luttuce-core 组件。</p>
<p data-nodeid="1371" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/3A/1D/Cgp9HWB-UKuAHIxRAA0aJtyWMgI410.png" alt="图片2.png" data-nodeid="1376"></p>
<div data-nodeid="1372"><p style="text-align:center">spring-boot-starter-data-redis-reactive 组件依赖图</p></div>








<p data-nodeid="435">在上图中，我们同时看到 luttuce-core 组件中使用了 Project Reactor 框架中的 reactor-core 组件，这点与前面介绍的技术栈是完全一致的。</p>
<p data-nodeid="436">为了获取连接，我们需要初始化 LettuceConnectionFactory。LettuceConnectionFactory 类的最简单使用方法如下所示。</p>
<pre class="lang-java" data-nodeid="437"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ReactiveRedisConnectionFactory <span class="hljs-title">lettuceConnectionFactory</span><span class="hljs-params">()</span> </span>{
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LettuceConnectionFactory();
}
	&nbsp;
<span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ReactiveRedisConnectionFactory <span class="hljs-title">lettuceConnectionFactory</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> LettuceConnectionFactory(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">6379</span>);
}
</code></pre>
<p data-nodeid="438">当然，LettuceConnectionFactory 也提供了一系列配置项供我们在初始化时进行设置，示例代码如下所示，我们可以对连接的安全性、超时时间等参数进行设置。</p>
<pre class="lang-java" data-nodeid="439"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ReactiveRedisConnectionFactory <span class="hljs-title">lettuceConnectionFactory</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RedisStandaloneConfiguration redisStandaloneConfiguration = <span class="hljs-keyword">new</span> RedisStandaloneConfiguration();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; redisStandaloneConfiguration.setDatabase(database);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; redisStandaloneConfiguration.setHostName(host);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; redisStandaloneConfiguration.setPort(port);
&nbsp;&nbsp;&nbsp;  redisStandaloneConfiguration.setPassword(RedisPassword.of(password));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LettuceClientConfiguration.LettuceClientConfigurationBuilder lettuceClientConfigurationBuilder = LettuceClientConfiguration
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .builder();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LettuceConnectionFactory factory = <span class="hljs-keyword">new</span> LettuceConnectionFactory(redisStandaloneConfiguration,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lettuceClientConfigurationBuilder.build());
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> factory;
}
</code></pre>
<p data-nodeid="440">有了 LettuceConnectionFactory，就可以用它来进一步初始化 ReactiveRedisTemplate。ReactiveRedisTemplate 的创建方式如下所示。与传统 RedisTemplate 创建方式的主要区别在于，ReactiveRedisTemplate 依赖于 ReactiveRedisConnectionFactory 来获取 ReactiveRedisConnection。</p>
<pre class="lang-java" data-nodeid="441"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-function">ReactiveRedisTemplate&lt;String, String&gt; 
<span class="hljs-title">reactiveRedisTemplate</span><span class="hljs-params">(ReactiveRedisConnectionFactory factory)</span> </span>{
<span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ReactiveRedisTemplate&lt;&gt;(factory, 
	RedisSerializationContext.string());
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">ReactiveRedisTemplate&lt;String, Account&gt; <span class="hljs-title">redisOperations</span><span class="hljs-params">(ReactiveRedisConnectionFactory factory)</span> </span>{
	Jackson2JsonRedisSerializer&lt;Account&gt; serializer = <span class="hljs-keyword">new</span> 
	Jackson2JsonRedisSerializer&lt;&gt;(Account.class);
&nbsp;
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; RedisSerializationContext.RedisSerializationContextBuilder
	&lt;String, Account&gt; builder = RedisSerializationContext
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .newSerializationContext(<span class="hljs-keyword">new</span> StringRedisSerializer());
&nbsp;
&nbsp;&nbsp;&nbsp; RedisSerializationContext&lt;String, Account&gt; context = 
	builder.value(serializer).build();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ReactiveRedisTemplate&lt;&gt;(factory, context);
}
</code></pre>
<p data-nodeid="442">以上代码中，我们使用了 Jackson2 作为数据存储的序列化方式，并对 Account 对象映射的规则做了初始化。</p>
<h4 data-nodeid="443">创建 Reactive Redis Repository</h4>
<p data-nodeid="444">因为没有直接可用的 Reactive Repository，所以在创建针对 Redis 的响应式 Repository 时，我们将采用“14 | 响应式全栈：响应式编程能为数据访问过程带来什么样的变化”中介绍的第三种方法，即自定义数据访问层接口。这里创建了 AccountRedisRepository 接口，代码如下。</p>
<pre class="lang-java" data-nodeid="445"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountRedisRepository</span> </span>{
&nbsp;   <span class="hljs-function">Mono&lt;Boolean&gt; <span class="hljs-title">saveAccount</span><span class="hljs-params">(Account account)</span></span>;

&nbsp;   <span class="hljs-function">Mono&lt;Boolean&gt; <span class="hljs-title">updateAccount</span><span class="hljs-params">(Account account)</span></span>;

&nbsp;   <span class="hljs-function">Mono&lt;Boolean&gt; <span class="hljs-title">deleteAccount</span><span class="hljs-params">(String accountId)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Mono&lt;Account&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String accountId)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Flux&lt;Account&gt; <span class="hljs-title">findAllAccounts</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="446">然后我们创建了 AccountRedisRepositoryImpl 类来实现 AccountRedisRepository 接口中定义的方法，这里就会用到前面初始化的 ReactiveRedisTemplate。AccountRedisRepositoryImpl 类代码如下所示。</p>
<pre class="lang-java" data-nodeid="447"><code data-language="java">Repository
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountRedisRepositoryImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AccountRedisRepository</span></span>{
&nbsp;
&nbsp; <span class="hljs-meta">@Autowired</span>
  <span class="hljs-keyword">private</span> ReactiveRedisTemplate&lt;String, Account&gt; reactiveRedisTemplate;
&nbsp;
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String HASH_NAME = <span class="hljs-string">"Account:"</span>;
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Boolean&gt; <span class="hljs-title">saveAccount</span><span class="hljs-params">(Account account)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> reactiveRedisTemplate.opsForValue().set(HASH_NAME + account.getId(), account);
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Boolean&gt; <span class="hljs-title">updateAccount</span><span class="hljs-params">(Account account)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> reactiveRedisTemplate.opsForValue().set(HASH_NAME + account.getId(), account);
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Boolean&gt; <span class="hljs-title">deleteAccount</span><span class="hljs-params">(String accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> reactiveRedisTemplate.opsForValue().delete(HASH_NAME + accountId);
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Account&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> reactiveRedisTemplate.opsForValue().get(HASH_NAME + accountId);
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;Account&gt; <span class="hljs-title">findAllAccounts</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> reactiveRedisTemplate.keys(HASH_NAME + <span class="hljs-string">"*"</span>).flatMap((String key) -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;Account&gt; mono = reactiveRedisTemplate.opsForValue().get(key);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> mono;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp; }
}
</code></pre>
<p data-nodeid="448">上述代码中的 reactiveRedisTemplate.opsForValue() 方法将使用 ReactiveValueOperations 来实现对 Redis 中数据的具体操作。与传统的 RedisOperations 工具类一样，响应式 Redis 也提供了 ReactiveHashOperations、ReactiveListOperations、ReactiveSetOperations、ReactiveValueOperations 和 ReactiveZSetOperations 类来分别处理不同的数据类型。</p>
<p data-nodeid="449">同时，在最后的 findAllAccounts() 方法中，我也演示了如何使用“07 | Reactor 操作符（上）：如何快速转换响应式流”中介绍的 flatMap 操作符，来根据 Redis 中的 Key 获取具体实体对象的处理方式，这种处理方式在响应式编程过程中应用非常广泛。</p>
<h4 data-nodeid="450">在 Service 层中调用 Reactive Repository</h4>
<p data-nodeid="451">在 Service 层中调用 AccountRedisRepository 的过程比较简单，我们创建了对应的 AccountService 类，代码如下所示。</p>
<pre class="lang-java" data-nodeid="452"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountService</span> </span>{
    <span class="hljs-meta">@Autowired</span>
	<span class="hljs-keyword">private</span> AccountRedisRepository 
	accountRepository;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Boolean&gt; <span class="hljs-title">save</span><span class="hljs-params">(Account account)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository.saveAccount(account);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Boolean&gt; <span class="hljs-title">delete</span><span class="hljs-params">(String id)</span> </span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository.deleteAccount(id);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Account&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String id)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository
	.findAccountById(id).log(<span class="hljs-string">"findOneAccount"</span>);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;Account&gt; <span class="hljs-title">findAllAccounts</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;      <span class="hljs-keyword">return</span> accountRepository
	.findAllAccounts().log(<span class="hljs-string">"findAllAccounts"</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="453">可以看到，这些方法都只是对 AccountRedisRepository 中对应方法的简单调用。在每次调用中，我们也通过 log 操作符进行了日志的记录。</p>
<h3 data-nodeid="454">案例集成：在 ReactiveSpringCSS 案例中整合 Redis</h3>
<p data-nodeid="455">在 ReactiveSpringCSS 中，Redis 的作用是实现缓存，这里就需要考虑它的具体应用场景。我们知道，在整体架构上，customer-service 一般会与用户服务 account-service 进行交互，但因为用户账户信息的更新属于低频事件，所以我们设计的实现方式是 account-service 通过消息中间件的方式将用户账户变更信息主动推送给 customer–service。而在这个时候，customer–service 就可以把接收到的用户账户信息保存在 Redis 中，两者之间的交互过程如下图所示。</p>
<p data-nodeid="456"><img src="https://s0.lgstatic.com/i/image6/M00/39/FD/CioPOWB9VM-AZWmYAADG17Qg_-g409.png" alt="Drawing 3.png" data-nodeid="522"></p>
<div data-nodeid="457"><p style="text-align:center">customer-service 服务与 account-service 服务交互图</p></div>
<p data-nodeid="458">在“10 | WebFlux（上）：如何使用注解编程模式构建异步非阻塞服务”中，我们已经梳理了 customer-service 中用于生成客户工单的 generateCustomerTicket 方法的整体流程，我带你回顾一下其伪代码。</p>
<pre class="lang-java" data-nodeid="459"><code data-language="java">generateCustomerTicket {
&nbsp;
&nbsp;&nbsp;&nbsp; 创建 CustomerTicket 对象
&nbsp;
	从远程 account-service 中获取 Account 对象
&nbsp;
	从远程 order-service 中获取Order对象
&nbsp;
	设置 CustomerTicket 对象属性
&nbsp;
	保存 CustomerTicket 对象并返回
}
</code></pre>
<p data-nodeid="460">现在，customer-service 已经可以从 Redis 缓存中获取变更之后的用户账户信息了。但如果用户信息没有变更，那么 Redis 中就不存在相关数据，我们还是需要访问远程服务。因此，整体流程需要做相应的调整，如下所示。</p>
<pre class="lang-java" data-nodeid="461"><code data-language="java">generateCustomerTicket {
&nbsp;
&nbsp;&nbsp;&nbsp; 创建 CustomerTicket 对象
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(Redis 中已存在目标 Account 对象) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 从 Redis 中获取 Account 对象
	} <span class="hljs-keyword">else</span> {
	    从远程 account-service 中获取 Account 对象
	}
&nbsp;
	从远程 order-service 中获取 Order 对象
&nbsp;
	设置 CustomerTicket 对象属性
&nbsp;
	保存 CustomerTicket 对象并返回
}
</code></pre>
<p data-nodeid="462">让我们把上述流程的调整反映在案例代码上。针对上述流程，我先基于第 10 讲中的实现方法给出原始 ReactiveAccountClient 类的实现过程，如下所示。</p>
<pre class="lang-java" data-nodeid="463"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveAccountClient</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; account = WebClient.create()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .get()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"http://127.0.0.1:8082/accounts/{accountId}"</span>, accountId) 
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .bodyToMono(AccountMapper.class).log("getAccountFromRemote");
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> account;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="464">可以看到这里直接使用 WebClient 类完成了对 account-service 的远程调用。现在，流程做了调整，我们需要在进行远程调用之前，先访问 Redis。因此，我们先在 ReactiveAccountClient 类中添加针对访问 Redis 的相关代码。</p>
<pre class="lang-java" data-nodeid="465"><code data-language="java"><span class="hljs-meta">@Autowired</span>
<span class="hljs-keyword">private</span> AccountRedisRepository accountRedisRepository;

<span class="hljs-function"><span class="hljs-keyword">private</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">getAccountFromCache</span><span class="hljs-params">(String accountId)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRedisRepository.findAccountById(accountId);
}
</code></pre>
<p data-nodeid="466">可以看到这里使用AccountRedisRepository完成数据的查询。然后，我们再基于该方法来重构 ReactiveAccountClient 中的 findAccountById 方法。</p>
<pre class="lang-java" data-nodeid="467"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String accountId)</span></span>{

&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//先从 Redis 中获取目标 Account 对象</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMonoFromCache = getAccountFromCache(accountId); 

&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//如果 Redis 中没有目标 Account 对象，则进行远程获取</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMono = accountMonoFromCache.switchIfEmpty(getAccountFromRemote(accountId));
&nbsp;&nbsp;&nbsp; &nbsp;<span class="hljs-keyword">return</span> accountMono;
}
</code></pre>
<p data-nodeid="468">这里用到了 Reactor 提供的switchIfEmpty 操作符来判断是否能够从 Redis 中获取目标 Account 对象，如果不能则再调用getAccountFromRemote 进行远程获取。getAccountFromRemote 方法如下所示。</p>
<pre class="lang-java" data-nodeid="469"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">getAccountFromRemote</span><span class="hljs-params">(String accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//远程获取 Account 对象</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMonoFromRemote= WebClient.create()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .get()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"http://127.0.0.1:8082/accounts/{accountId}"</span>, accountId) 
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .bodyToMono(AccountMapper.class).log("getAccountFromRemote");
&nbsp;&nbsp;&nbsp;  //将获取到的 Account 对象放入 Redis 中
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> accountMonoFromRemote.flatMap(<span class="hljs-keyword">this</span>::putAccountIntoCache);
}
</code></pre>
<p data-nodeid="470">这里我们先从 account-service 中远程获取 Account 对象，然后再把获取到的对象通过 putAccountIntoCache 方法放入 Redis 中以便下次直接从缓存中进行获取。putAccountIntoCache 方法实现如下。</p>
<pre class="lang-java" data-nodeid="471"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">putAccountIntoCache</span><span class="hljs-params">(AccountMapper account)</span> </span>{
	&nbsp;
	<span class="hljs-keyword">return</span> accountRedisRepository.saveAccount(account).flatMap( saved -&gt; {
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Mono&lt;AccountMapper&gt; savedAccount = accountRedisRepository.findAccountById(account.getId());

&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> savedAccount;
&nbsp;&nbsp;&nbsp;  });
}
</code></pre>
<p data-nodeid="472">上述代码中我们首先通过 accountRedisRepository 的 saveAccount 方法将对象保存到 Redis 中，然后再通过 findAccountById 从 Redis 中获取所保存的数据进行返回。<strong data-nodeid="543">请注意，accountRedisRepository 的 saveAccount 方法的返回值是 Mono<code data-backticks="1" data-nodeid="537">&lt;Boolean&gt;</code>，而我们想要获取的目标对象类型是 Mono<code data-backticks="1" data-nodeid="539">&lt;AccountMapper&gt;</code>，所以这里使用了 flatMap 操作符完成了两者之间的转换</strong>。这也是 flatMap 操作符非常常见的一种用法。</p>
<p data-nodeid="473">最后，让我们确认一下重构之后的 ReactiveAccountClient 的完整代码，如下所示。</p>
<pre class="lang-java" data-nodeid="474"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveAccountClient</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AccountRedisRepository accountRedisRepository;

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">getAccountFromCache</span><span class="hljs-params">(String accountId)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRedisRepository.findAccountById(accountId);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">putAccountIntoCache</span><span class="hljs-params">(AccountMapper account)</span> </span>{

&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> accountRedisRepository.saveAccount(account).flatMap( saved -&gt; {
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; Mono&lt;AccountMapper&gt; savedAccount = accountRedisRepository.findAccountById(account.getId());

&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> savedAccount;
&nbsp;&nbsp;&nbsp;  });
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">findAccountById</span><span class="hljs-params">(String accountId)</span></span>{

&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//先从 Redis 中获取目标 Account 对象</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMonoFromCache= getAccountFromCache(accountId); 

&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//如果 Redis 中没有目标 Account 对象，则进行远程获取</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMono = accountMonoFromCache.switchIfEmpty(getAccountFromRemote(accountId));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountMono;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;AccountMapper&gt; <span class="hljs-title">getAccountFromRemote</span><span class="hljs-params">(String accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//远程获取 Account 对象</span>
&nbsp;&nbsp;&nbsp;  Mono&lt;AccountMapper&gt; accountMonoFromRemote= WebClient.create()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .get()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"http://127.0.0.1:8082/accounts/{accountId}"</span>, accountId) 
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .bodyToMono(AccountMapper.class).log("getAccountFromRemote");

&nbsp;&nbsp;&nbsp;  //将获取到的 Account 对象放入 Redis 中
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> accountMonoFromRemote.flatMap(<span class="hljs-keyword">this</span>::putAccountIntoCache);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="475">现在，当我们访问 customer-service 里面 CustomerTicketService 中的 generateCustomerTicket 方法时，就会从 Redis 中获取数据从而提高访问效率。</p>
<h3 data-nodeid="476">小结与预告</h3>
<p data-nodeid="477">Redis 是一款主流的缓存数据库，和 MongoDB 一样提供了实现响应式流的驱动程序。而 Spring 家族中的 Spring Data Redis Reactive 组件则提供了以响应式流的方法访问 Redis 的高效开发模式，本讲结合案例对这一组件的使用方式进行了详细的讨论。</p>
<p data-nodeid="478">最后给你留一道思考题：在使用 Spring Data Redis Reactive 时，如果想要实现一个自定义的响应式 Repository 应该怎么做？</p>
<p data-nodeid="479">在 Spring Data 中，除了 MongoDB 和 Redis 之外，还针对 Cassandra 和 CouchBase 提供了响应式驱动程序，对于关系型数据库而言则没有直接的解决方案。因此，如何在系统中为关系型数据库添加响应式数据访问机制，从而确保全栈式的响应式数据流是一大挑战。下一讲，我们将深入讨论这一话题。</p>
<blockquote data-nodeid="480">
<p data-nodeid="481" class="">点击链接，获取课程相关代码 ↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="554">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### **成：
> 老师，Flux包含的对象获取然后保存到redis中

##### **辉：
> 是否还能够通过注解的方式使用Spring-Cache?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该可以的

