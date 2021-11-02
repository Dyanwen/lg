<p data-nodeid="18439" class="">今天我们来聊聊二级缓存相关的话题。</p>
<p data-nodeid="18440">我们在使用 Mybatis 的时候，基本不用关心什么是二级缓存。而如果你是 Hibernate 的使用者，一定经常听说和使用过 Hibernate 的二级缓存，那么我们应该怎么看待它呢？这一讲一起来揭晓 Cache 的相关概念以及在生产环境中的最佳实践。</p>
<h3 data-nodeid="18441">二级缓存的概念</h3>
<p data-nodeid="18442">上一讲我们介绍了一级缓存相关的内容，一级缓存的实体的生命周期和 PersistenceContext 是相同的，即载体为同一个 Session 才有效；而 Hibernate 提出了二级缓存的概念，也就是可以在不同的 Session 之间共享实体实例，说白了就是在单个应用内的整个 application 生命周期之内共享实体，减少数据库查询。</p>
<p data-nodeid="18443">由于 JPA 协议本身并没有规定二级缓存的概念，所以这是 Hiberante 独有的特性。所以在 Hibernate 中，从数据库里面查询实体的过程就变成了：第一步先看看一级缓存里面有没有实体，如果没有再看看二级缓存里面有没有，如果还是没有再从数据库里面查询。那么在 Hibernate 的环境下如何开启二级缓存呢？</p>
<h4 data-nodeid="18444">Hibernate 中二级缓存的配置方法</h4>
<p data-nodeid="18445">Hibernate 中，默认情况下二级缓存是关闭的，如果想开启二级缓存需要通过如下三个步骤。</p>
<p data-nodeid="18446"><strong data-nodeid="18547">第一步：引入第三方二级缓存的实现的 jar</strong>。</p>
<p data-nodeid="18447">因为 Hibernate 本身并没有实现缓存的功能，而是主要依赖第三方，如 Ehcache、jcache、redis 等第三方库。下面我们以 EhCache 为例，利用 gradle 引入 hibernate-ehcace 的依赖。代码如下所示。</p>
<pre class="lang-java" data-nodeid="18448"><code data-language="java">implementation <span class="hljs-string">'org.hibernate:hibernate-ehcache:5.2.2.Final'</span>
</code></pre>
<p data-nodeid="18449">如果我们想用 jcache，可以通过如下方式。</p>
<pre class="lang-java" data-nodeid="18450"><code data-language="java">compile <span class="hljs-string">'org.hibernate:hibernate-jcache:5.2.2.Final'</span>
</code></pre>
<p data-nodeid="18451"><strong data-nodeid="18554">第二步：在配置文件里面开启二级缓存</strong>。</p>
<p data-nodeid="18452">二级缓存默认是关闭的，所以需要我们用如下方式开启二级缓存，并且配置 cache.region.factory_class 为不同的缓存实现类。</p>
<pre class="lang-java" data-nodeid="18453"><code data-language="java">hibernate.cache.use_second_level_cache=<span class="hljs-keyword">true</span>
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory
</code></pre>
<p data-nodeid="18454"><strong data-nodeid="18562">第三步：在用到二级缓存的地方配置 @Cacheable 和 @Cache 的策略</strong>。</p>
<pre class="lang-java" data-nodeid="18455"><code data-language="java"><span class="hljs-keyword">import</span> javax.persistence.Cacheable;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Cacheable</span>
<span class="hljs-meta">@org</span>.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{......}
</code></pre>
<p data-nodeid="18456">通过以上三步就可以轻松实现二级缓存了，但是这时请你思考一下，这真的能应用到我们实际生产环境中吗？会不会有副作用？</p>
<h4 data-nodeid="18457">二级缓存的思考</h4>
<p data-nodeid="18458">二级缓存主要解决的是单应用场景下跨 Session 生命周期的实体共享问题，可是我们一定要通过 Hibernate 来做吗？答案并不是，其实我们可以通过各种 Cache 的手段来做，因为 Hibernate 里面一级缓存的复杂度相对较高，并且使用的话实体的生命周期会有变化，查询问题的过程较为麻烦。</p>
<p data-nodeid="18459">同时，随着现在逐渐微服务化、分布式化，如今的应用都不是单机应用，那么缓存之间如何共享呢？分布式缓存又该如何解决？比如一个机器变了，另一个机器没变，应该如何处理？似乎 Hiberante 并没有考虑到这些问题。</p>
<p data-nodeid="18460">此外，还有什么时间数据会变更、变化了之后如何清除缓存，等等，这些都是我们要思考的，所以 Hibernate 的二级缓存听起来“高大上”，但是使用起来绝对没有那么简单。</p>
<p data-nodeid="18461">那么经过这一连串的疑问，如果我们不用 Hibernate 的二级缓存，还有没有更好的解决方案呢？</p>
<h3 data-nodeid="18462">利用 Redis 进行缓存</h3>
<p data-nodeid="18463">在我们实际工作中经常需要 cache 的就是 Redis，那么我们通过一个例子，来看下 Spring Cache 结合 Redis 是怎么使用的。</p>
<h4 data-nodeid="18464">Spring Cache 和 Redis 结合</h4>
<p data-nodeid="18465">第一步：在 gradle 中引入 cache 和 redis 的依赖，代码如下所示。</p>
<pre class="lang-java" data-nodeid="18466"><code data-language="java"><span class="hljs-comment">//原来我们只用到了JPA</span>
implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-data-jpa'</span>
<span class="hljs-comment">//为了引入cache和redis机制需要引入如下两个jar包</span>
implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-data-redis'</span> <span class="hljs-comment">//redis的依赖</span>
implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-cache'</span> <span class="hljs-comment">//cache 的依赖</span>
</code></pre>
<p data-nodeid="18467">第二步：在 application.properties 里面增加 redis 的相关配置，代码如下。</p>
<pre class="lang-java" data-nodeid="18468"><code data-language="java">spring.redis.host=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>
spring.redis.port=<span class="hljs-number">6379</span>
spring.redis.password=sySj6vmYke
spring.redis.timeout=<span class="hljs-number">6000</span>
spring.redis.pool.max-active=<span class="hljs-number">8</span>
spring.redis.pool.max-idle=<span class="hljs-number">8</span>
spring.redis.pool.max-wait=-<span class="hljs-number">1</span>
spring.redis.pool.min-idle=<span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="18469">第三步：通过 @EnableCaching 开启缓存，增加 configuration 配置类，代码如下所示。</p>
<pre class="lang-java" data-nodeid="18470"><code data-language="java"><span class="hljs-meta">@EnableCaching</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CacheConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="18471">第四步：在我们需要缓存的地方添加 @Cacheable 注解即可。为了方便演示，我把 @Cacheable 注解配置在了 controller 方法上，代码如下。</p>
<pre class="lang-java" data-nodeid="18472"><code data-language="java"><span class="hljs-meta">@GetMapping("/user/info/{id}")</span>
<span class="hljs-meta">@Cacheable(value = "userInfo", key = "{#root.methodName, #id}", unless = "#result == null")</span> <span class="hljs-comment">//利用默认key值生成规则value加key生成一个redis的key值，result==null的时候不进行缓存</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfo</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> Long id)</span> </span>{
   <span class="hljs-comment">//第二次就不会再执行这里了</span>
   <span class="hljs-keyword">return</span> userInfoRepository.findById(id).get();
}
</code></pre>
<p data-nodeid="18473">第五步：启动项目，请求一下这个 API 会发现，第一次请求过后，redis 里面就有一条记录了，如下图所示。</p>
<p data-nodeid="18678" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/A1/CgpVE1_gGfGAAYRDAAGxaGz4d8A668.png" alt="Drawing 0.png" data-nodeid="18681"></p>

<p data-nodeid="18475">可以看到，第二次请求之后，取数据就不会再请求数据库了。那么 redis 我们已经熟悉了，那么来看一下 Spring Cache 都做了哪些事情。</p>
<h4 data-nodeid="18476">Spring Cache 介绍</h4>
<p data-nodeid="18477">Spring 3.1 之后引入了基于注释（annotation）的缓存（cache）技术，它本质上不是一个具体的缓存实现方案（例如 EHCache 或者 Redis），而是一个对缓存使用的抽象概念，通过在既有代码中添加少量它定义的各种 annotation，就能够达到缓存方法的返回对象的效果。</p>
<p data-nodeid="18478">Spring 的缓存技术还具备相当的灵活性，不仅能够使用 SpEL（Spring Expression Language）来定义缓存的 key 和各种 condition，还提供开箱即用的缓存临时存储方案，也支持主流的专业缓存，例如 Redis，EHCache 集成。而 Spring Cache 属于 Spring framework 的一部分，在下面图片所示的这个包里面。</p>
<p data-nodeid="19160" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/9F/Cip5yF_gGfmAfvKbAAGFdPiyN9k521.png" alt="Drawing 1.png" data-nodeid="19163"></p>

<p data-nodeid="18480"><strong data-nodeid="18590">Spring cache 里面的主要的注解</strong></p>
<p data-nodeid="18481"><strong data-nodeid="18594">@Cacheable</strong></p>
<p data-nodeid="18482">应用到读取数据的方法上，就是可以缓存的方法，如查找方法：先从缓存中读取，如果没有再调用方法获取数据，然后把数据添加到缓存中。</p>
<pre class="lang-java" data-nodeid="18483"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Cacheable {
   <span class="hljs-meta">@AliasFor("cacheNames")</span>
   String[] value() <span class="hljs-keyword">default</span> {};
<span class="hljs-comment">//cache的名字。可以根据名字设置不同cache处理类。redis里面可以根据cache名字设置不同的失效时间。</span>
   <span class="hljs-meta">@AliasFor("value")</span>
   String[] cacheNames() <span class="hljs-keyword">default</span> {};
<span class="hljs-comment">//缓存的key的名字，支持spel</span>
   <span class="hljs-function">String <span class="hljs-title">key</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
<span class="hljs-comment">//key的生成策略，不指定可以用全局的默认的。</span>
   <span class="hljs-function">String <span class="hljs-title">keyGenerator</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//客户选择不同的CacheManager</span>
   <span class="hljs-function">String <span class="hljs-title">cacheManager</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//配置不同的cache resolver</span>
   <span class="hljs-function">String <span class="hljs-title">cacheResolver</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//满足什么样的条件才能被缓存，支持SpEL，可以去掉方法名、参数</span>
   <span class="hljs-function">String <span class="hljs-title">condition</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
<span class="hljs-comment">//排除哪些返回结果不加入缓存里面去，支持SpEL，实际工作中常见的是result ==null等</span>
   <span class="hljs-function">String <span class="hljs-title">unless</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//是否同步读取缓存、更新缓存</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">sync</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
}
</code></pre>
<p data-nodeid="18484">下面是@Cacheable 相关的例子。</p>
<pre class="lang-java" data-nodeid="18485"><code data-language="java"><span class="hljs-meta">@Cacheable(cacheNames="book", condition="#name.length() &lt; 32", unless="#result.notNeedCache")</span><span class="hljs-comment">//利用SPEL表达式只有当name参数长度小于32的时候再进行缓存，排除notNeedCache的对象</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Book <span class="hljs-title">findBook</span><span class="hljs-params">(String name)</span>
</span></code></pre>
<p data-nodeid="18486"><strong data-nodeid="18600">@CachePut</strong></p>
<p data-nodeid="18487">调用方法时会自动把相应的数据放入缓存，它与 @Cacheable 不同的是所有注解的方法每次都会执行，一般配置在 Update 和 insert 方法上。其源码里面的字段和用法基本与 @Cacheable 相同，只是使用场景不一样，我就不详细介绍了。</p>
<p data-nodeid="18488"><strong data-nodeid="18605">@CacheEvict</strong></p>
<p data-nodeid="18489">删除缓存，一般配置在删除方法上面。代码如下所示。</p>
<pre class="lang-java" data-nodeid="18490"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> CacheEvict {
<span class="hljs-comment">//与@Cacheable相同的部分咱我就不重复叙述了。</span>
......
	<span class="hljs-comment">//是否删除所有的实体对象</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">allEntries</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
   <span class="hljs-comment">//是否方法执行之前执行。默认在方法调用成功之后删除</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">beforeInvocation</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
}
	<span class="hljs-meta">@Caching</span> 所有Cache注解的组合配置方法，源码如下：
	<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Caching {
   Cacheable[] cacheable() <span class="hljs-keyword">default</span> {};
   CachePut[] put() <span class="hljs-keyword">default</span> {};
   CacheEvict[] evict() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="18491">此外，还有 @CacheConfig 表示全局 Cache 配置；@EnableCaching，表示是否开启 SpringCache 的配置。</p>
<p data-nodeid="18492">以上是 SpringCache 中常见的注解，下面我们再来看 Spring Cache Redis 里面主要的类都有哪些。</p>
<p data-nodeid="18493"><strong data-nodeid="18612">Spring Cache Redis 里面主要的类</strong></p>
<p data-nodeid="18494"><strong data-nodeid="18616">org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration</strong></p>
<p data-nodeid="18495">cache 的自动装配类，此类被加载的方式是在 spring boot的spring.factories 文件里面，其关键源码如下所示。</p>
<pre class="lang-java" data-nodeid="18496"><code data-language="java"><span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
<span class="hljs-meta">@ConditionalOnClass(CacheManager.class)</span>
<span class="hljs-meta">@ConditionalOnBean(CacheAspectSupport.class)</span>
<span class="hljs-meta">@ConditionalOnMissingBean(value = CacheManager.class, name = "cacheResolver")</span>
<span class="hljs-meta">@EnableConfigurationProperties(CacheProperties.class)</span>
<span class="hljs-meta">@AutoConfigureAfter({ CouchbaseDataAutoConfiguration.class, HazelcastAutoConfiguration.class,
      HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class })</span>
<span class="hljs-meta">@Import({ CacheConfigurationImportSelector.class, CacheManagerEntityManagerFactoryDependsOnPostProcessor.class })</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CacheAutoConfiguration</span> </span>{
  <span class="hljs-comment">/**
   * {<span class="hljs-doctag">@link</span> ImportSelector} to add {<span class="hljs-doctag">@link</span> CacheType} configuration classes.
   */</span>
  <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CacheConfigurationImportSelector</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ImportSelector</span> </span>{
     <span class="hljs-meta">@Override</span>
     <span class="hljs-keyword">public</span> String[] selectImports(AnnotationMetadata importingClassMetadata) {
        CacheType[] types = CacheType.values();
        String[] imports = <span class="hljs-keyword">new</span> String[types.length];
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; types.length; i++) {
           imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
        }
        <span class="hljs-keyword">return</span> imports;
     }
  }
}
</code></pre>
<p data-nodeid="18497">通过源码可以看到，此类的关键作用是加载 Cache 的依赖配置，以及加载所有 CacheType 的配置文件，而 CacheConfigurations 里面定义了不同的 Cache 实现方式的配置，里面包含了 Ehcache、Redis、Jcache 的各种实现方式，如下图所示。</p>
<p data-nodeid="19642" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/9F/Cip5yF_gGgWAPkIYAAD6KrApZRs929.png" alt="Drawing 2.png" data-nodeid="19645"></p>

<p data-nodeid="18499"><strong data-nodeid="18625">org.springframework.cache.annotation.CachingConfigurerSupport</strong></p>
<p data-nodeid="18500">通过此类可以自定义 Cache 里面的 CacheManager、CacheResolver、KeyGenerator、CacheErrorHandler，代码如下所示。</p>
<pre class="lang-java" data-nodeid="18501"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CachingConfigurerSupport</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">CachingConfigurer</span> </span>{
  <span class="hljs-comment">// cache的manager，主要是管理不同的cache的实现方式，如redis还是ehcache等</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-meta">@Nullable</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> CacheManager <span class="hljs-title">cacheManager</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   }
   <span class="hljs-comment">// cache的不同实现者的操作方法，CacheResolver解析器，用于根据实际情况来动态解析使用哪个Cache</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-meta">@Nullable</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> CacheResolver <span class="hljs-title">cacheResolver</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   }
   <span class="hljs-comment">//cache的key的生成规则</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-meta">@Nullable</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> KeyGenerator <span class="hljs-title">keyGenerator</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   }
   <span class="hljs-comment">//cache发生异常的回调处理，一般情况下我会打印个warn日志，方便知道发生了什么事情</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-meta">@Nullable</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> CacheErrorHandler <span class="hljs-title">errorHandler</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   }
}
</code></pre>
<p data-nodeid="18502">其中，所有 CacheManager 是 Spring 提供的各种缓存技术抽象接口，通过它来管理，Spring framework 里面默认实现的 CacheManager 有不同的实现类，redis 默认加载的是 RedisCacheManager，如下图所示。</p>
<p data-nodeid="20124" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/9F/Cip5yF_gGgyAJ21FAADMLk_C6ag487.png" alt="Drawing 3.png" data-nodeid="20127"></p>

<p data-nodeid="18504"><strong data-nodeid="18634">org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration</strong></p>
<p data-nodeid="18505">它是加载 Cache 的实现者，也是 redis 的实现类，关键源码如下图所示。</p>
<p data-nodeid="20606" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/BE/Ciqc1F_gGhKAQGL2AAIaDnIIDQQ607.png" alt="Drawing 4.png" data-nodeid="20609"></p>

<p data-nodeid="18507">我们可以看得出来，它依赖本身的 Redis 的连接，并且加载了 RedisCacheManager；同时可以看到关于 Cache 和 Redis 的配置有哪些。</p>
<p data-nodeid="18508">通过 CacheProperties 里面 redis 的配置，我们可以设置“key 的统一前缀、默认过期时间、是否缓存 null 值、是否使用前缀”这四个配置。</p>
<p data-nodeid="21088" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8B/C9/CgqCHl_gGhqAf6niAACSWTdYSNc168.png" alt="Drawing 5.png" data-nodeid="21091"></p>

<p data-nodeid="18510">通过这几个主要的类，相信你已经对 Spring Cache 有了简单的了解，下面我们看一下在实际工作中有哪些最佳实践可以提供参考。</p>
<h3 data-nodeid="18511">Spring Cache 结合 Redis 使用的最佳实践</h3>
<p data-nodeid="18512"><strong data-nodeid="18649">不同 cache 的 name 在 redis 里面配置不同的过期时间</strong></p>
<p data-nodeid="18513">默认情况下所有 redis 的 cache 过期时间是一样的，实际工作中一般需要自定义不同 cache 的 name 的过期时间，我们这里 cache 的 name 就是指 @Cacheable 里面 value 属性对应的值。主要步骤如下。</p>
<p data-nodeid="18514">第一步：自定义一个配置文件，用来指定不同的 cacheName 对应的过期时间不一样。代码如下所示。</p>
<pre class="lang-java" data-nodeid="18515"><code data-language="java"><span class="hljs-meta">@Getter</span>
<span class="hljs-meta">@Setter</span>
<span class="hljs-meta">@ConfigurationProperties(prefix = "spring.cache.redis")</span>
<span class="hljs-comment">/**
 * 改善一下cacheName的最佳实践方法，目前主要用不同的cache name不同的过期时间，可以扩展
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyCacheProperties</span> </span>{
    <span class="hljs-keyword">private</span> HashMap&lt;String, Duration&gt; cacheNameConfig;
}
</code></pre>
<p data-nodeid="18516">第二步：通过自定义类 MyRedisCacheManagerBuilderCustomizer 实现 RedisCacheManagerBuilderCustomizer 里面的 customize 方法，用来指定不同的 name 采用不同的 RedisCacheConfiguration，从而达到设置不同的过期时间的效果。代码如下所示。</p>
<pre class="lang-java" data-nodeid="18517"><code data-language="java"><span class="hljs-comment">/**
 * 这个依赖spring boot 2.2 以上版本才有效
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyRedisCacheManagerBuilderCustomizer</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RedisCacheManagerBuilderCustomizer</span> </span>{
    <span class="hljs-keyword">private</span> MyCacheProperties myCacheProperties;
    <span class="hljs-keyword">private</span> RedisCacheConfiguration redisCacheConfiguration;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">MyRedisCacheManagerBuilderCustomizer</span><span class="hljs-params">(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration)</span> </span>{
        <span class="hljs-keyword">this</span>.myCacheProperties = myCacheProperties;
        <span class="hljs-keyword">this</span>.redisCacheConfiguration = redisCacheConfiguration;
    }
    <span class="hljs-comment">/**
     * 利用默认配置的只需要在这里加就可以了
     * spring.cache.cache-names=abc,def,userlist2,user3
     * 下面是不同的cache-name可以配置不同的过期时间，yaml也支持，如果以后还有其他属性扩展可以改这里
     * spring.cache.redis.cache-name-config.user2=2h
     * spring.cache.redis.cache-name-config.def=2m
     * <span class="hljs-doctag">@param</span> builder
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">customize</span><span class="hljs-params">(RedisCacheManager.RedisCacheManagerBuilder builder)</span> </span>{
        <span class="hljs-keyword">if</span> (ObjectUtils.isEmpty(myCacheProperties.getCacheNameConfig())) {
            <span class="hljs-keyword">return</span>;
        }
        Map&lt;String, RedisCacheConfiguration&gt; cacheConfigurations = myCacheProperties.getCacheNameConfig().entrySet().stream()
                .collect(Collectors
                        .toMap(e-&gt;e.getKey(),v-&gt;builder
                                .getCacheConfigurationFor(v.getKey())
                                .orElse(RedisCacheConfiguration.defaultCacheConfig().serializeValuesWith(redisCacheConfiguration.getValueSerializationPair()))
                                .entryTtl(v.getValue())));
        builder.withInitialCacheConfigurations(cacheConfigurations);
    }
}
</code></pre>
<p data-nodeid="18518">第三步：在 CacheConfiguation 里面把我们自定义的 CacheManagerCustomize 加载进去即可，代码如下。</p>
<pre class="lang-java" data-nodeid="18519"><code data-language="java"><span class="hljs-meta">@EnableCaching</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@EnableConfigurationProperties(value = {MyCacheProperties.class,CacheProperties.class})</span>
<span class="hljs-meta">@AutoConfigureAfter({CacheAutoConfiguration.class})</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CacheConfiguration</span> </span>{
    <span class="hljs-comment">/**
     * 支持不同的cache name有不同的缓存时间的配置
     *
     * <span class="hljs-doctag">@param</span> myCacheProperties
     * <span class="hljs-doctag">@param</span> redisCacheConfiguration
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-meta">@ConditionalOnMissingBean(name = "myRedisCacheManagerBuilderCustomizer")</span>
    <span class="hljs-meta">@ConditionalOnClass(RedisCacheManagerBuilderCustomizer.class)</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> MyRedisCacheManagerBuilderCustomizer <span class="hljs-title">myRedisCacheManagerBuilderCustomizer</span><span class="hljs-params">(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration)</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyRedisCacheManagerBuilderCustomizer(myCacheProperties,redisCacheConfiguration);
    }
}
</code></pre>
<p data-nodeid="18520">第四步：使用的时候非常简单，只需要在 application.properties 里面做如下配置即可。</p>
<pre class="lang-java" data-nodeid="18521"><code data-language="java"># 设置默认的过期时间是20分钟
spring.cache.redis.time-to-live=20m
# 设置我们刚才的例子 @Cacheable(value="userInfo")5分钟过期
spring.cache.redis.cache-name-config.userInfo=5m
# 设置 room的cache1小时过期
spring.cache.redis.cache-name-config.room=1h
</code></pre>
<p data-nodeid="18522"><strong data-nodeid="18658">自定义 KeyGenerator 实现，redis 的 key 自定义拼接规则</strong></p>
<p data-nodeid="18523">假如我们不喜欢默认的 cache 生成的 key 的 string 规则，那么可以自定义。我们创建 MyRedisCachingConfigurerSupport 集成 CachingConfigurerSupport 即可，代码如下。</p>
<pre class="lang-java" data-nodeid="18524"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-meta">@Log4j2</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyRedisCachingConfigurerSupport</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CachingConfigurerSupport</span> </span>{
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> KeyGenerator <span class="hljs-title">keyGenerator</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> getKeyGenerator();
    }
    <span class="hljs-comment">/**
     * 覆盖默认的redis key的生成规则，变成"方法名:参数:参数"
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> KeyGenerator <span class="hljs-title">getKeyGenerator</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> (target, method, params) -&gt; {
            StringBuilder key = <span class="hljs-keyword">new</span> StringBuilder();
            key.append(ClassUtils.getQualifiedMethodName(method));
            <span class="hljs-keyword">for</span> (Object obc : params) {
                key.append(<span class="hljs-string">":"</span>).append(obc);
            }
            <span class="hljs-keyword">return</span> key.toString();
        };
    }
}
</code></pre>
<p data-nodeid="18525"><strong data-nodeid="18663">当发生 cache 和 redis 的操作异常时，我们不希望阻碍主流程，打印一个关键日志即可</strong></p>
<p data-nodeid="18526">只需要在 MyRedisCachingConfigurerSupport 里面再实现父类的 errorHandler 即可，代码变成了如下模样。</p>
<pre class="lang-java" data-nodeid="18527"><code data-language="java"><span class="hljs-meta">@Log4j2</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyRedisCachingConfigurerSupport</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CachingConfigurerSupport</span> </span>{
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> KeyGenerator <span class="hljs-title">keyGenerator</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> getKeyGenerator();
    }
    <span class="hljs-comment">/**
     * 覆盖默认的redis key的生成规则，变成"方法名:参数:参数"
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> KeyGenerator <span class="hljs-title">getKeyGenerator</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> (target, method, params) -&gt; {
            StringBuilder key = <span class="hljs-keyword">new</span> StringBuilder();
            key.append(ClassUtils.getQualifiedMethodName(method));
            <span class="hljs-keyword">for</span> (Object obc : params) {
                key.append(<span class="hljs-string">":"</span>).append(obc);
            }
            <span class="hljs-keyword">return</span> key.toString();
        };
    }
    <span class="hljs-comment">/**
     * 覆盖默认异常处理方法，不抛异常，改打印error日志
     *
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> CacheErrorHandler <span class="hljs-title">errorHandler</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CacheErrorHandler() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleCacheGetError</span><span class="hljs-params">(RuntimeException exception, Cache cache, Object key)</span> </span>{
                log.error(String.format(<span class="hljs-string">"Spring cache GET error:cache=%s,key=%s"</span>, cache, key), exception);
            }
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleCachePutError</span><span class="hljs-params">(RuntimeException exception, Cache cache, Object key, Object value)</span> </span>{
                log.error(String.format(<span class="hljs-string">"Spring cache PUT error:cache=%s,key=%s"</span>, cache, key), exception);
            }
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleCacheEvictError</span><span class="hljs-params">(RuntimeException exception, Cache cache, Object key)</span> </span>{
                log.error(String.format(<span class="hljs-string">"Spring cache EVICT error:cache=%s,key=%s"</span>, cache, key), exception);
            }
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleCacheClearError</span><span class="hljs-params">(RuntimeException exception, Cache cache)</span> </span>{
                log.error(String.format(<span class="hljs-string">"Spring cache CLEAR error:cache=%s"</span>, cache), exception);
            }
        };
    }
}
</code></pre>
<p data-nodeid="18528"><strong data-nodeid="18668">改变默认的 cache 里面 redis 的 value 序列化方式</strong></p>
<p data-nodeid="18529">默认有可能是 JDK 序列化方式，所以一般我们看不懂 redis 里面的值，那么就可以把序列化方式改成 JSON 格式，只需要在 CacheConfiguration 里面增加默认的 RedisCacheConfiguration 配置即可，完整的 CacheConfiguration 变成如下代码所示的样子。</p>
<pre class="lang-java" data-nodeid="18530"><code data-language="java"><span class="hljs-meta">@EnableCaching</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@EnableConfigurationProperties(value = {MyCacheProperties.class,CacheProperties.class})</span>
<span class="hljs-meta">@AutoConfigureAfter({CacheAutoConfiguration.class})</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CacheConfiguration</span> </span>{
    <span class="hljs-comment">/**
     * 支持不同的cache name有不同的缓存时间的配置
     *
     * <span class="hljs-doctag">@param</span> myCacheProperties
     * <span class="hljs-doctag">@param</span> redisCacheConfiguration
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-meta">@ConditionalOnMissingBean(name = "myRedisCacheManagerBuilderCustomizer")</span>
    <span class="hljs-meta">@ConditionalOnClass(RedisCacheManagerBuilderCustomizer.class)</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> MyRedisCacheManagerBuilderCustomizer <span class="hljs-title">myRedisCacheManagerBuilderCustomizer</span><span class="hljs-params">(MyCacheProperties myCacheProperties, RedisCacheConfiguration redisCacheConfiguration)</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyRedisCacheManagerBuilderCustomizer(myCacheProperties,redisCacheConfiguration);
    }
    <span class="hljs-comment">/**
     * cache异常不抛异常，只打印error日志
     *
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-meta">@ConditionalOnMissingBean(name = "myRedisCachingConfigurerSupport")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> MyRedisCachingConfigurerSupport <span class="hljs-title">myRedisCachingConfigurerSupport</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MyRedisCachingConfigurerSupport();
    }
    <span class="hljs-comment">/**
     * 依赖默认的ObjectMapper，实现普通的json序列化
     * <span class="hljs-doctag">@param</span> defaultObjectMapper
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Bean(name = "genericJackson2JsonRedisSerializer")</span>
    <span class="hljs-meta">@ConditionalOnMissingBean(name = "genericJackson2JsonRedisSerializer")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> GenericJackson2JsonRedisSerializer <span class="hljs-title">genericJackson2JsonRedisSerializer</span><span class="hljs-params">(ObjectMapper defaultObjectMapper)</span> </span>{
        ObjectMapper objectMapper = defaultObjectMapper.copy();
        objectMapper.registerModule(<span class="hljs-keyword">new</span> Hibernate5Module().enable(REPLACE_PERSISTENT_COLLECTIONS)); <span class="hljs-comment">//支持JPA的实体的json的序列化</span>
        objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY, <span class="hljs-keyword">true</span>);<span class="hljs-comment">//培训</span>
        objectMapper.deactivateDefaultTyping(); <span class="hljs-comment">//关闭 defaultType，不需要关心reids里面是否为对象的类型</span>
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> GenericJackson2JsonRedisSerializer(objectMapper);
    }
    <span class="hljs-comment">/**
     * 覆盖 RedisCacheConfiguration，只是修改serializeValues with jackson
     *
     * <span class="hljs-doctag">@param</span> cacheProperties
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-meta">@ConditionalOnMissingBean(name = "jacksonRedisCacheConfiguration")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> RedisCacheConfiguration <span class="hljs-title">jacksonRedisCacheConfiguration</span><span class="hljs-params">(CacheProperties cacheProperties,
                                                                  GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer)</span> </span>{
        CacheProperties.Redis redisProperties = cacheProperties.getRedis();
        RedisCacheConfiguration config = RedisCacheConfiguration
                .defaultCacheConfig();
        config = config.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(genericJackson2JsonRedisSerializer));<span class="hljs-comment">//修改的关键所在，指定Jackson2JsonRedisSerializer的方式</span>
        <span class="hljs-keyword">if</span> (redisProperties.getTimeToLive() != <span class="hljs-keyword">null</span>) {
            config = config.entryTtl(redisProperties.getTimeToLive());
        }
        <span class="hljs-keyword">if</span> (redisProperties.getKeyPrefix() != <span class="hljs-keyword">null</span>) {
            config = config.prefixCacheNameWith(redisProperties.getKeyPrefix());
        }
        <span class="hljs-keyword">if</span> (!redisProperties.isCacheNullValues()) {
            config = config.disableCachingNullValues();
        }
        <span class="hljs-keyword">if</span> (!redisProperties.isUseKeyPrefix()) {
            config = config.disableKeyPrefix();
        }
        <span class="hljs-keyword">return</span> config;
    }
}
</code></pre>
<h3 data-nodeid="18531">总结</h3>
<p data-nodeid="18532">以上就是本讲的内容了，这一讲的目的是帮助你打开思路，了解 Spring Data 的生态体系。那么由于篇幅有限，我介绍的 Cache、Redis、JPA 只是这三个项目里的冰山一角，你在实际工作中可以根据实际的应用场景，想想它们各自的职责是什么，让它们发挥各自的特长，而不是依赖于 Hibernate 功能的强大，为了用而去用，这样会让代码的可读性和复杂度提高很多，就会遇到各种各样的问题，导致觉得 Hibernate 太难，或者不可控。</p>
<p data-nodeid="18533">其实大多数时候是我们的思路不对，其实万事万物皆有优势和劣势，我们要抛弃其劣势，充分利用各个框架的优势，发挥各自的特长。如果你觉得本专栏对你有帮助，就动动手指分享吧，下一讲我们来聊聊 Spring Data Rest 的相关话题，到时见。</p>
<blockquote data-nodeid="18534">
<p data-nodeid="18535" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="18677">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论


