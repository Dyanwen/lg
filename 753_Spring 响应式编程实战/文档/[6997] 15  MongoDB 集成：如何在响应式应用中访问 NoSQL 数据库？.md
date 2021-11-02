<p data-nodeid="448" class="">上一讲开始，我们进入了响应式数据访问这一模块的学习，并且引出了 Spring 家族中专门用于实现数据访问的 Spring Data 框架及其响应式版本。我们知道 Spring Data 支持多种响应式 Repository 用来构建全栈响应式编程模型，而 MongoDB 就是其中具有代表性的一种数据存储库。今天，我就将结合案例来给出 Reactive MongoDB 的使用方式。</p>
<h3 data-nodeid="449">Spring Data MongoDB Reactive 技术栈</h3>
<p data-nodeid="450">在介绍 Spring Data MongoDB Reactive 的使用方式之前，我们先来简要分析它的基本组成结构和所使用的技术栈。</p>
<p data-nodeid="451">显然，ReactiveMongoRepository 是开发人员所需要面对的第一个核心组件，你已经在上一讲中看到过它的定义，如下所示。</p>
<pre class="lang-java" data-nodeid="452"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ReactiveMongoRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; <span class="hljs-keyword">extends</span> 
<span class="hljs-title">ReactiveSortingRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt;, <span class="hljs-title">ReactiveQueryByExampleExecutor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Mono&lt;S&gt; <span class="hljs-title">insert</span><span class="hljs-params">(S entity)</span></span>;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Flux&lt;S&gt; <span class="hljs-title">insert</span><span class="hljs-params">(Iterable&lt;S&gt; entities)</span></span>;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Flux&lt;S&gt; <span class="hljs-title">insert</span><span class="hljs-params">(Publisher&lt;S&gt; entities)</span></span>;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Flux&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example)</span></span>;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Flux&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example, Sort sort)</span></span>;
}
</code></pre>
<p data-nodeid="453">Spring Data MongoDB Reactive 模块只有一个 ReactiveMongoRepository 接口的实现类，即 SimpleReactiveMongoRepository 类。它为 ReactiveMongoRepository 的所有方法提供实现，并使用 ReactiveMongoOperations 接口处理针对 MongoDB 的数据访问操作。例如在 SimpleReactiveMongoRepository 类中有一个 findAllById 方法，如下所示。</p>
<pre class="lang-java" data-nodeid="454"><code data-language="java">&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;T&gt; <span class="hljs-title">findAllById</span><span class="hljs-params">(Publisher&lt;ID&gt; ids)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(ids, <span class="hljs-string">"The given Publisher of Id's must not be null!"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Flux.from(ids).buffer().flatMap(<span class="hljs-keyword">this</span>::findAllById);
&nbsp;&nbsp;&nbsp; }
</code></pre>
<p data-nodeid="455">可以看到这个方法使用 buffer 操作符收集所有 ids，然后使用 findAllById(Iterable <code data-backticks="1" data-nodeid="520">&lt;ID&gt;</code> ids) 重载方法创建一个请求。该方法反过来构建 Query 对象并调用 findAll(Query query) 方法，这时候触发 ReactiveMongoOperations 实例的 mongoOperations.find (query,...) 方法。</p>
<p data-nodeid="456">而 ReactiveMongoOperations 接口的实现类就是 ReactiveMongoTemplate 类，在这个模板工具类中，基于 MongoDB 提供的 Java 驱动程序完成对数据库的访问。我们在 ReactiveMongoTemplate 类所引用的包结构中可以看到这些驱动程序的客户端组件，如下所示。</p>
<pre class="lang-java" data-nodeid="457"><code data-language="java"><span class="hljs-keyword">import</span> com.mongodb.client.model.UpdateOptions;
<span class="hljs-keyword">import</span> com.mongodb.client.result.UpdateResult;
…
&nbsp;
<span class="hljs-keyword">import</span> com.mongodb.reactivestreams.client.MongoClient;
<span class="hljs-keyword">import</span> com.mongodb.reactivestreams.client.MongoCollection;
…
</code></pre>
<p data-nodeid="879">Spring Data 中的响应式 MongoDB 连接基于 MongoDB 响应式流 Java 驱动程序（mongo-java-driver-reactivestreams）构建。该驱动程序提供具有非阻塞背压的异步流处理。另一方面，响应式驱动程序构建在 MongoDB 异步 Java 驱动程序（mongo-java-driver-async）之上。这个异步驱动程序是低级别的，并且具有基于回调的 API，因此它不像更高级别的响应式流驱动程序那样易于使用。下图展示了 Spring Data 中整个响应式 MongoDB 的技术栈。</p>
<p data-nodeid="880" class=""><img src="https://s0.lgstatic.com/i/image6/M01/38/5F/Cgp9HWB5NlyAUZ8vAAEVEX4GDg4227.png" alt="图片1.png" data-nodeid="885"></p>
<div data-nodeid="881"><p style="text-align:center">Spring Data MongoDB Reactive 技术栈</p></div>




<p data-nodeid="1463">而下图展示了基于 Maven 依赖所找到的对应的组件库。</p>
<p data-nodeid="1464" class=""><img src="https://s0.lgstatic.com/i/image6/M01/38/68/CioPOWB5NmeAMPT_AAa3JDFhfRw228.png" alt="图片2.png" data-nodeid="1469"></p>
<div data-nodeid="1465"><p style="text-align:center">Spring Data MongoDB Reactive 中的组件库</p></div>




<h3 data-nodeid="464">应用 Reactive MongoDB</h3>
<p data-nodeid="465">接下来，我要介绍使用 Spring Data MongoDB Reactive 进行系统开发的具体过程，将从开发环境的准备、Repository 的创建、数据的初始化以及与 Service 层之间的集成等几个步骤为你介绍。</p>
<p data-nodeid="466">首先要说的是需要初始化运行环境。</p>
<h4 data-nodeid="467">初始化 Reactive MongoDB 运行环境</h4>
<p data-nodeid="468">我们需要在 pom 文件中添加 spring-boot-starter-data-mongodb-reactive 依赖，如下所示。</p>
<pre class="lang-java" data-nodeid="469"><code data-language="java">&lt;dependency&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;artifactId&gt;spring-boot-starter-data-mongodb-reactive&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="2047">然后我们通过 Maven 来查看组件依赖关系可以得到如下所示的组件依赖图。</p>
<p data-nodeid="2048" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/38/68/CioPOWB5NnOAFoIqAAhDv7J-DwA202.png" alt="图片3.png" data-nodeid="2053"></p>
<div data-nodeid="2049"><p style="text-align:center">spring-boot-starter-data-mongodb-reactive 组件依赖图</p></div>




<p data-nodeid="473">可以看到 spring-boot-starter-data-mongodb-reactive 组件同时依赖于 spring-data-mongodb、mongodb-driver-reactivestreams 以及 reactor-core 等组件，这点与前面介绍的技术栈是完全一致的。</p>
<p data-nodeid="474">为了集成 Reactive MongoDB，在 Spring Boot 应用程序中，我们可以在它的启动类上添加 @EnableReactiveMongoRepositories 注解，包含该注解的 Spring Boot 启动类如下所示。</p>
<pre class="lang-java" data-nodeid="475"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableReactiveMongoRepositories</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringReactiveMongodbApplication</span> </span>{
&nbsp;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(SpringReactiveMongodbApplication
	.class, args);
&nbsp; }
}
</code></pre>
<p data-nodeid="476">事实上，默认情况下我们一般不需要在 Spring Boot 启动类中手工添加 @EnableReactiveMongoRepositories 注解。因为当添加 spring-boot-starter-data-mongodb-reactive 组件到 classpath 时，MongoReactiveRepositoriesAutoConfiguration 配置类会自动创建与 MongoDB 交互的核心类。MongoReactiveRepositoriesAutoConfiguration 类的定义如下所示。</p>
<pre class="lang-java" data-nodeid="477"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@ConditionalOnClass({ MongoClient.class, ReactiveMongoRepository.class })</span>
<span class="hljs-meta">@ConditionalOnMissingBean({ ReactiveMongoRepositoryFactoryBean.class,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ReactiveMongoRepositoryConfigurationExtension.class })</span>
<span class="hljs-meta">@ConditionalOnProperty(prefix = "spring.data.mongodb.reactive-repositories", name = "enabled", havingValue = "true", matchIfMissing = true)</span>
<span class="hljs-meta">@Import(MongoReactiveRepositoriesAutoConfigureRegistrar.class)</span>
<span class="hljs-meta">@AutoConfigureAfter(MongoReactiveDataAutoConfiguration.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MongoReactiveRepositoriesAutoConfiguration</span> </span>{
&nbsp;
}
</code></pre>
<p data-nodeid="478">可以看到这里引入了 MongoReactiveRepositoriesAutoConfigureRegistrar 类，如果 MongoDB 和 Spring Data 已经在 classpath 上，Spring Boot 会通过 MongoReactiveRepositoriesAutoConfigureRegistrar 类自动帮我们完成配置。MongoReactiveRepositoriesAutoConfigureRegistrar 类的定义如下。</p>
<pre class="lang-java" data-nodeid="479"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MongoReactiveRepositoriesAutoConfigureRegistrar</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractRepositoryConfigurationSourceSupport</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-keyword">protected</span> Class&lt;? extends Annotation&gt; getAnnotation() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> EnableReactiveMongoRepositories.class;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-keyword">protected</span> Class&lt;?&gt; getConfiguration() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> EnableReactiveMongoRepositoriesConfiguration.class;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> RepositoryConfigurationExtension <span class="hljs-title">getRepositoryConfigurationExtension</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ReactiveMongoRepositoryConfigurationExtension();
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@EnableReactiveMongoRepositories</span>
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EnableReactiveMongoRepositoriesConfiguration</span> </span>{
&nbsp;
  }
}
</code></pre>
<p data-nodeid="480">在上述代码中，我们在最后部分看到了熟悉的 @EnableReactiveMongoRepositories 注解。显然，如果我们使用 Spring Boot 的默认配置，就不需要刻意在启动类上添加 @EnableReactiveMongoRepositories 注解。但如果希望修改 MongoDB 的配置行为，这个注解就可以派上用场。以下代码演示了 @EnableReactiveMongoRepositories 注解的使用方法。</p>
<pre class="lang-java" data-nodeid="481"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@EnableReactiveMongoRepositories(basePackageClasses = 
	OrderRepository.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MongoConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractReactiveMongoConfiguration</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> MongoClient <span class="hljs-title">reactiveMongoClient</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> MongoClients.create();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> String <span class="hljs-title">getDatabaseName</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"order_test"</span>;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ReactiveMongoTemplate <span class="hljs-title">mongoTemplate</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ReactiveMongoTemplate(mongoClient(), getDatabaseName());
    }
}
</code></pre>
<p data-nodeid="482">以上代码中，我们通过 @EnableReactiveMongoRepositories 注解指定了“basePackageClasses”为OrderRepository，同时修改所要访问的数据库名为“order_test”，这样就相当于对 Repository 类以及数据库的名称做了人为的指定，而在默认情况下系统会自动扫描 Repository 类，并默认使用领域实体的名称作为数据库名。</p>
<h4 data-nodeid="483">创建 Reactive MongoDB Repository</h4>
<p data-nodeid="484">接下来，我们来创建一个 Reactive MongoDB Repository。在这里，我们定义了一个领域实体 Account，并使用 @Document 和 @Id 等 MongoDB 相关的注解。Account 实体代码如下所示。</p>
<pre class="lang-java" data-nodeid="485"><code data-language="java"><span class="hljs-meta">@Document</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Account</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String id;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String accountCode;
	<span class="hljs-keyword">private</span> String accountName;
	<span class="hljs-comment">//省略 getter/setter</span>
}
</code></pre>
<p data-nodeid="486">我们可以通过上一讲介绍的三种方式中的的任何一种来创建 Reactive MongoDB Repository。我们可以定义继承自 ReactiveMongoRepository 接口的 AccountReactiveMongoRepository 接口，同时该接口还继承了 ReactiveQueryByExampleExecutor 接口。</p>
<p data-nodeid="487">AccountReactiveMongoRepository 接口定义的代码如下所示，可以看到我们完全基于 ReactiveMongoRepository 和 ReactiveQueryByExampleExecutor 接口的默认方法来实现业务功能。</p>
<pre class="lang-java" data-nodeid="488"><code data-language="java"><span class="hljs-meta">@Repository</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountReactiveMongoRepository</span>
<span class="hljs-keyword">extends</span> <span class="hljs-title">ReactiveMongoRepository</span>&lt;<span class="hljs-title">Account</span>, <span class="hljs-title">String</span>&gt;, 
	<span class="hljs-title">ReactiveQueryByExampleExecutor</span>&lt;<span class="hljs-title">Account</span>&gt; </span>{
}
</code></pre>
<h4 data-nodeid="489">使用 CommandLineRunner 初始化 MongoDB 数据</h4>
<p data-nodeid="490">对于 MongoDB 等数据库而言，我们通常需要执行一些数据初始化操作。接下来我将介绍如何通过 Spring Boot 提供的 CommandLineRunner 来实现这一常见场景。</p>
<p data-nodeid="491">很多时候我们希望在系统运行之前执行一些前置操作，为了实现这样的需求，Spring Boot 提供了一个方案，即 CommandLineRunner 接口，定义如下所示。</p>
<pre class="lang-java" data-nodeid="492"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">CommandLineRunner</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(String... args)</span> <span class="hljs-keyword">throws</span> Exception</span>;
}
</code></pre>
<p data-nodeid="493">Spring Boot 应用程序在启动后，会遍历所有已定义的 CommandLineRunner 接口的实例并运行它们的 run() 方法。</p>
<p data-nodeid="494">此外，正如我在前面所介绍的，在 MongoDB 客户端组件中存在一个 MongoOperations 工具类。相对于 Repository 接口而言，MongoOperations 提供了更多方法，也更接近于 MongoDB 的原生态语言。基于 CommandLineRunner 和 MongoOperations，我们就可以对 MongoDB 进行数据初始化，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="495"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InitDatabase</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">CommandLineRunner <span class="hljs-title">init</span><span class="hljs-params">(MongoOperations operations)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> args -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operations.dropCollection(Account.class);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operations.insert(<span class="hljs-keyword">new</span> Account(<span class="hljs-string">"A_"</span> + UUID.randomUUID().toString(),<span class="hljs-string">"account1"</span>, <span class="hljs-string">"jianxiang1"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operations.insert(<span class="hljs-keyword">new</span> Account(<span class="hljs-string">"A_"</span> + UUID.randomUUID().toString(),<span class="hljs-string">"account2"</span>, <span class="hljs-string">"jianxiang"</span>));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operations.findAll(Account.class).forEach(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; account -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(account.getId()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="496">在这个例子中，我们先通过 MongoOperations 的 dropCollection() 方法清除整个 Account 数据库中的数据，然后往该数据库中添加了两条记录，最后我们通过 findAll() 方法执行查询操作，获取新插入的两条数据并打印在控制台上。</p>
<h4 data-nodeid="497">在 Service 层中调用 Reactive Repository</h4>
<p data-nodeid="498">完成 AccountReactiveMongoRepository 并初始化数据之后，我们就可以创建 Service 层组件来调用 AccountReactiveMongoRepository。这里我们创建了 AccountService 类作为 Service 层组件，代码如下所示。</p>
<pre class="lang-java" data-nodeid="499"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountService</span></span>{

    <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> AccountReactiveMongoRepository accountRepository;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Account&gt; <span class="hljs-title">save</span><span class="hljs-params">(Account account)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository.save(account);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Account&gt; <span class="hljs-title">findOne</span><span class="hljs-params">(String id)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository
	.findById(id).log(<span class="hljs-string">"findOneAccount"</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;Account&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository.findAll().log(<span class="hljs-string">"findAllAccounts"</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Void&gt; <span class="hljs-title">delete</span><span class="hljs-params">(String id)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accountRepository
	.deleteById(id).log(<span class="hljs-string">"deleteOneAccount"</span>);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;Account&gt; <span class="hljs-title">getAccountsByAccountName</span><span class="hljs-params">(String accountName)</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp; Account account = <span class="hljs-keyword">new</span> Account();
&nbsp;&nbsp;&nbsp; &nbsp; account.setAccountName(accountName);
&nbsp;
&nbsp;&nbsp;&nbsp; &nbsp; ExampleMatcher matcher = ExampleMatcher.matching()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withIgnoreCase()
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withMatcher(accountName, GenericPropertyMatcher.of(StringMatcher.STARTING))
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withIncludeNullValues();
&nbsp;
&nbsp;&nbsp;&nbsp; &nbsp; Example&lt;Account&gt; example = Example.of(account, matcher);

&nbsp;&nbsp;&nbsp; &nbsp; Flux&lt;Account&gt; accounts = accountRepository.findAll(example).log(<span class="hljs-string">"getAccountsByAccountName"</span>);

&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-keyword">return</span> accounts;
	}
}
</code></pre>
<p data-nodeid="500">AccountService 类中的 save()、findOne()、findAll() 和 delete() 方法都来自 ReactiveMongoRepository 接口，而最后的 findByAccountName() 方法则使用了 ReactiveQueryByExampleExecutor 接口所提供的 QueryByExample 机制。</p>
<p data-nodeid="501">QueryByExample 可以翻译成按示例查询，是一种用户友好的查询技术。它允许动态创建查询，并且不需要编写包含字段名称的查询方法。实际上，QueryByExample 不需要使用特定的数据库查询语言来编写查询语句。从组成结构上讲，QueryByExample 包括 Probe、ExampleMatcher 和 Example 这三个基本组件。其中 Probe 包含对应字段的实例对象；ExampleMatcher 携带有关如何匹配特定字段的详细信息，相当于匹配条件；而 Example 则由 Probe 和 ExampleMatcher 组成，用于构建具体的查询操作。</p>
<p data-nodeid="502">在上述示例代码中，我们首先构建了一个 ExampleMatcher 用于初始化匹配规则，然后通过传入一个 Account 对象实例和 ExampleMatcher 实例构建了一个 Example 对象，最后通过 ReactiveQueryByExampleExecutor 接口中的 findAll() 方法实现了 QueryByExample 机制。</p>
<p data-nodeid="503">同时，你也应该注意到，在 AccountService 的 findOne()、findAll()、delete() 以及 findByAccountName() 这四个方法的最后都调用了 log() 方法，该方法使用了 Reactor 框架中的日志操作符，我们在“08 | Reactor 操作符（下）：如何多样化裁剪响应式流”中有详细介绍。</p>
<p data-nodeid="504">通过添加 log() 方法，在执行这些数据操作时就会获取 Reactor 框架中对数据的详细操作日志信息。在这个示例中，我们启动服务并执行这四个方法，会在控制台中看到对应的日志。其中一部分日志展示了服务启动时通过 CommandLineRunner 插入初始化数据到数据库的过程，另一部分则分别针对各个添加了 log() 方法的操作打印出数据流的执行效果。在 Service 层通过 log() 方法添加日志是一种常见的开发技巧，你可以自己做一些尝试。</p>
<h3 data-nodeid="505">案例集成：构建基于 MongoDB 的数据访问层</h3>
<p data-nodeid="506">在介绍完如何使用 Spring Data MongoDB Reactive 构建基于 MongoDB 的数据访问组件之后，让我们来到 ReactiveSpringCSS 案例中。针对案例中的数据访问场景，本讲所介绍的相关技术都可以直接进行应用。</p>
<p data-nodeid="507">事实上，在前面的介绍中，我们已经构建了 account-service 中的数据访问层，而其他两个服务中的数据访问层也类似，这里就不再细说了，你可以参考放在 Github 上的案例代码进行学习。</p>
<h3 data-nodeid="508">小结与预告</h3>
<p data-nodeid="509">MongoDB 是一款主流的 NoSQL 数据库，其提供了实现响应式流的驱动程序，因此非常适合作为响应式系统中的持久化数据库。而 Spring 家族中的 Spring Data MongoDB Reactive 组件则提供了以响应式流的方法访问 MongoDB 的高效开发模式，本讲结合案例对这一组件的使用方式进行了详细的讨论。</p>
<p data-nodeid="510">这里给你留一道思考题：在使用 Spring Data MongoDB Reactive 时，针对查询操作可以使用哪些高效的实现方法？</p>
<p data-nodeid="511">在今天内容的基础上，下一讲我们将基于 Spring Data 框架中的 Spring Data Redis 组件来访问缓存数据库 Redis，并同样结合 ReactiveSpringCSS 案例完成对现有实现方式的重构。</p>
<blockquote data-nodeid="512">
<p data-nodeid="513" class="">点击链接，获取课程相关代码 ↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="593">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论


