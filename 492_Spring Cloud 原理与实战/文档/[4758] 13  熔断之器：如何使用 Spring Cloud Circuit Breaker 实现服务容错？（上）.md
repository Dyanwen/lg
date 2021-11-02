<p data-nodeid="641" class="">在上一课时中，我们全面梳理了在微服务架构中实现服务容错的设计思想和实现方案，也引出了 Spring Cloud 中专门用于实现服务容错的 Spring Cloud Circuit Breaker 框架。我们知道 Spring Cloud Circuit Breaker 是一个集成性的框架，内部整合了 Netflix Hystrix、Resilience4j、Sentinel 和 Spring Retry 这四款独立的熔断器组件。由于课时有限，我们无意对这四款组件都进行详细的展开，而是更多关注于 Netflix 旗下的 Hystrix，以及受 Hystrix 启发而诞生的 Resilience4j。在今天的课时中，我们先来讨论 Netflix Hystrix。</p>
<h3 data-nodeid="642">引入 Hystrix</h3>
<p data-nodeid="643">要想在微服务中添加对 Netflix Hystrix 的支持，我们首先需要在 Maven 中添加对 spring-cloud-starter-netflix-hystrix 的依赖。过程如下所示：</p>
<pre class="lang-xml" data-nodeid="644"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-netflix-hystrix<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="645">另一方面，通过上一课时的介绍，我们也明确了在微服务架构中，服务调用之间势必需要<strong data-nodeid="711">引入熔断器机制</strong>，<strong data-nodeid="712">确保服务容错</strong>。所以，Spring Cloud 推出了一个全新的注解，@SpringCloudApplication 注解。该注解用来集成服务治理和服务熔断方面的核心功能。@SpringCloudApplication 注解定义如下所示。</p>
<pre class="lang-java" data-nodeid="646"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Inherited</span>
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableDiscoveryClient</span>
<span class="hljs-meta">@EnableCircuitBreaker</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> SpringCloudApplication {
}
</code></pre>
<p data-nodeid="647">可以看到 @SpringCloudApplication 是一个组合注解，整合了 @SpringBootApplication、@EnableDiscoveryClient 和 @EnableCircuitBreaker 这三个微服务所需的核心注解。我们可以直接使用该注解来简化代码。因此，从今天开始，在所有的业务服务中，我们都将使用这个新的 @SpringCloudApplication 注解。以 intervention-service 为例，现在的 Bootstrap 类的定义如下所示：</p>
<pre class="lang-java" data-nodeid="648"><code data-language="java"><span class="hljs-meta">@SpringCloudApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InterventionApplication</span>
</span></code></pre>
<p data-nodeid="649">在 Hystrix 中，最核心的莫过于<strong data-nodeid="719">HystrixCommand 类</strong>。HystrixCommand 是一个抽象类，只包含了一个抽象方法，即如下所示的 run 方法：</p>
<pre class="lang-java" data-nodeid="650"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> R <span class="hljs-title">run</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception</span>;
</code></pre>
<p data-nodeid="651">显然，这个方法是让开发人员实现服务容错所需要处理的业务逻辑。在微服务架构中，我们通常在这个 run 方法中添加对远程服务的访问代码。</p>
<p data-nodeid="652">同时我们在 HystrixCommand 类中还发现了另一个很有用的方法 getFallback。这个方法用于在 HystrixCommand 子类中设置服务回退函数的具体实现，如下所示：</p>
<pre class="lang-java" data-nodeid="653"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> R <span class="hljs-title">getFallback</span><span class="hljs-params">()</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UnsupportedOperationException(<span class="hljs-string">"No fallback available."</span>);
}
</code></pre>
<p data-nodeid="654">Hystrix 是一个非常经典而完善的服务容错开发框架，同时支持了上一课时中所提到的服务隔离、服务熔断和服务回退机制。下面的内容，就让我们逐一了解一下这些功能吧。</p>
<h3 data-nodeid="655">使用 Hystrix 实现服务隔离</h3>
<p data-nodeid="656">基于前面对 HystrixCommand 抽象类的理解，我们就可以提供一个该类的子类来实现服务隔离。针对服务隔离，Hystrix 组件在提供了线程池隔离机制的同时，还实现了<strong data-nodeid="729">信号量隔离</strong>。这里，我们基于最常用的线程池隔离来进行介绍。典型的 HystrixCommand 子类代码风格如下所示：</p>
<pre class="lang-java" data-nodeid="657"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">GetUserCommand</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">HystrixCommand</span>&lt;<span class="hljs-title">UserMapper</span>&gt; </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//远程调用 user-service 的客户端工具类</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserServiceClient userServiceClient;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">GetUserCommand</span><span class="hljs-params">(String name)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(Setter.withGroupKey(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置命令组</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCommandGroupKey.Factory.asKey(<span class="hljs-string">"springHealthGroup"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置命令键</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .andCommandKey(HystrixCommandKey.Factory.asKey(<span class="hljs-string">"interventionKey"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置线程池键</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .andThreadPoolKey(HystrixThreadPoolKey.Factory.asKey(name))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置命令属性</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .andCommandPropertiesDefaults(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCommandProperties.Setter()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withExecutionTimeoutInMilliseconds(<span class="hljs-number">5000</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置线程池属性</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .andThreadPoolPropertiesDefaults(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixThreadPoolProperties.Setter()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withMaxQueueSize(<span class="hljs-number">10</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withCoreSize(<span class="hljs-number">2</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; );
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> UserMapper <span class="hljs-title">run</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> userServiceClient.getUserByUserName(<span class="hljs-string">"springhealth_user1"</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> UserMapper <span class="hljs-title">getFallback</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UserMapper(<span class="hljs-number">1L</span>,<span class="hljs-string">"user1"</span>,<span class="hljs-string">"springhealth_user1"</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="658">上述代码中使用了 Hystrix 中的很多常见的配置项，这些配置项大多数也涉及线程池隔离的相关概念。在 Hystrix 中，从控制粒度上讲，开发人员可以从服务分组和服务本身这两个维度出发，对线程隔离机制进行配置。也就是说我们既可以把一批服务都划分到一个线程池中，也可以把单个服务划分到一个线程池中。上述代码中的 HystrixCommandGroupKey 和 HystrixCommandKey 分别用来配置<strong data-nodeid="743">服务分组名称</strong>和<strong data-nodeid="744">服务名称</strong>，然后 HystrixThreadPoolKey 用来配置<strong data-nodeid="745">线程池</strong>的名称。</p>
<p data-nodeid="659">当我们根据需要设置<del data-nodeid="751">了</del>分组、服务以及线程池名称后，接下来就需要指定与线程池相关的各个属性。这些属性都包含在 HystrixThreadPoolProperties 中。例如，在上述代码中，我们使用 maxQueueSize 配置线程池队列的最大值，使用 coreSize 配置核心线程池的最大值等。同时，我们也注意到可以使用 withExecutionTimeoutInMilliseconds 配置项来指定请求的超时时间。</p>
<p data-nodeid="660">虽然，上述代码有助于我们更好的理解 Hystrix 中线程池隔离实现机制，但在日常开发过程中，一般不建议你通过创建一个 HystrixCommand 子类的方式来实现服务隔离，而是推荐你使用更为简单的 @HystrixCommand 注解。@HystrixCommand 是 Hystrix 为简化开发过程而专门提供的一个注解，定义如下：</p>
<pre class="lang-java" data-nodeid="661"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> HystrixCommand {

&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">groupKey</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">commandKey</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">threadPoolKey</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">fallbackMethod</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; HystrixProperty[] commandProperties() <span class="hljs-keyword">default</span> {};
&nbsp;&nbsp;&nbsp; HystrixProperty[] threadPoolProperties() <span class="hljs-keyword">default</span> {};
&nbsp;&nbsp;&nbsp; Class&lt;? extends Throwable&gt;[] ignoreExceptions() <span class="hljs-keyword">default</span> {};
&nbsp;&nbsp;&nbsp; <span class="hljs-function">ObservableExecutionMode <span class="hljs-title">observableExecutionMode</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ObservableExecutionMode.EAGER</span>;
&nbsp;&nbsp;&nbsp; HystrixException[] raiseHystrixExceptions() <span class="hljs-keyword">default</span> {};
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">defaultFallback</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
}
</code></pre>
<p data-nodeid="662">在上述定义中，我们看到了用于设置分组、服务与线程池名称相关的 groupKey、commandKey 和 threadPoolKey方法，以及与线程池属性相关的 threadPoolProperties 对象。让我们回到案例，并使用 @HystrixCommand 注解进行重构，效果如下：</p>
<pre class="lang-java" data-nodeid="663"><code data-language="java"><span class="hljs-meta">@HystrixCommand</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> UserMapper <span class="hljs-title">getUser</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> userClient.getUserByUserName(userName);
}
</code></pre>
<p data-nodeid="664">可以看到这里只使用了 @HystrixCommand 这个注解就完成 HystrixCommand 的创建。当然，我们也可以进一步使用 @HystrixProperty 注解来设置所需的各项属性，如下所示：</p>
<pre class="lang-java" data-nodeid="665"><code data-language="java"><span class="hljs-meta">@HystrixCommand(threadPoolKey = "springHealthGroup",
&nbsp;&nbsp;&nbsp; threadPoolProperties =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;@HystrixProperty(name="coreSize",value="2"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;@HystrixProperty(name="maxQueueSize",value="10")
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
)</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> UserMapper <span class="hljs-title">getUser</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> userClient.getUserByUserName(userName);
}
</code></pre>
<p data-nodeid="666">同样可以看到，我们为该 HystrixCommand 设置了 threadPoolKey，也提供了 threadPoolProperties 来设置 coreSize 和 maxQueueSize。</p>
<h3 data-nodeid="667">使用 Hystrix 实现服务熔断</h3>
<p data-nodeid="668">在上一课时中，我们知道熔断器有三个状态，其中<strong data-nodeid="770">打开</strong>和<strong data-nodeid="771">半打开状态</strong>会导致<strong data-nodeid="772">触发熔断机制</strong>。针对服务熔断，我们同样来考虑案例系统中一个服务依赖调用的具体场景，这个场景是对前面介绍的服务隔离的衍生。在 SpringHealth 案例中，我们知道 intervention-service 需要调用 user-service 和 device-service 来生成健康干预记录，该操作的代码流程如下所示：</p>
<pre class="lang-java" data-nodeid="669"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Intervention <span class="hljs-title">generateIntervention</span><span class="hljs-params">(String userName, String deviceCode)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Generate intervention record with user: {} from device: {}"</span>, userName, deviceCode);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Intervention intervention = <span class="hljs-keyword">new</span> Intervention();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取远程 User 信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserMapper user = getUser(userName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (user == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> intervention;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Get remote user: {} is successful"</span>, userName);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取远程 Device 信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DeviceMapper device = getDevice(deviceCode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (device == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> intervention;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Get remote device: {} is successful"</span>, deviceCode);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建并保存 Intervention 信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; intervention.setUserId(user.getId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; intervention.setDeviceId(device.getId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; intervention.setHealthData(device.getHealthData());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; intervention.setIntervention(<span class="hljs-string">"InterventionForDemo"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; intervention.setCreateTime(<span class="hljs-keyword">new</span> Date());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; interventionRepository.save(intervention);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> intervention;
}
</code></pre>
<p data-nodeid="670">显然，上述代码中 getUser() 方法和 getDevice() 方法都会涉及微服务之间的相互依赖和调用，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="671"><code data-language="java"><span class="hljs-meta">@Autowired</span>
<span class="hljs-keyword">private</span> UserServiceClient userClient;
&nbsp;
<span class="hljs-meta">@Autowired</span>
<span class="hljs-keyword">private</span> DeviceServiceClient deviceClient;
&nbsp;
<span class="hljs-meta">@HystrixCommand</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> UserMapper <span class="hljs-title">getUser</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> userClient.getUserByUserName(userName);
}
&nbsp;
<span class="hljs-meta">@HystrixCommand</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> DeviceMapper <span class="hljs-title">getDevice</span><span class="hljs-params">(String deviceCode)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> deviceClient.getDevice(deviceCode);
}
</code></pre>
<p data-nodeid="672">这里通过注入 UserServiceClient 和 DeviceServiceClient 两个工具类来实现远程调用，这两个工具类都使用了 Ribbon 和 RestTemplate 来实现调用过程的客户端负载均衡，关于客户端负载均衡相关内容我们在《负载均衡：如何使用 Ribbon 实现客户端负载均衡？》中已经进行了介绍。</p>
<p data-nodeid="673">在微服务环境下，使用 UserServiceClient 和 DeviceServiceClient 的调用过程可能会出现响应超时等问题，这个时候 intervention-service 作为服务消费者需要做到服务容错。要嵌入 Hystrix 提供的熔断机制，我们只需要在这两个方法上添加 @HystrixCommand 注解即可。在前面的代码我已经做了相应的示例。</p>
<p data-nodeid="674">现在我们来模拟一下远程调用超时的场景，调整 getDevice() 方法的代码，通过 Thread.sleep(2000) 来模拟响应时间过长的场景，如下所示。</p>
<pre class="lang-java" data-nodeid="675"><code data-language="java"><span class="hljs-meta">@HystrixCommand</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> DeviceMapper <span class="hljs-title">getDevice</span><span class="hljs-params">(String deviceCode)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Thread.sleep(<span class="hljs-number">2000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (InterruptedException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> deviceClient.getDevice(deviceCode);
}
</code></pre>
<p data-nodeid="676">现在我们创建一个端点，如下所示：</p>
<pre class="lang-java" data-nodeid="677"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value="interventions")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InterventionController</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> InterventionService interventionService;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@RequestMapping(value = "/{userName}/{deviceCode}", method = RequestMethod.POST)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Intervention <span class="hljs-title">generateIntervention</span><span class="hljs-params">( <span class="hljs-meta">@PathVariable("userName")</span> String userName,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PathVariable("deviceCode")</span> String deviceCode)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Intervention intervention = interventionService.generateIntervention(userName, deviceCode);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> intervention;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="678">显然，这个端点是用来访问 InterventionService 并生成 Intervention 记录的。现在，让我们访问这个端点：</p>
<pre class="lang-xml" data-nodeid="679"><code data-language="xml">http://localhost:8083/interventionss/springhealth_user1/device_blood
</code></pre>
<p data-nodeid="680">首先，我们在 intervention-service 的控制台中会看到“java.lang.InterruptedException: sleep interrupted”异常地抛出，而抛出该异常的来源正是 Hystrix。</p>
<p data-nodeid="681">然后，我们来查看端点调用的返回值，如下所示：</p>
<pre class="lang-xml" data-nodeid="682"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"timestamp":"1601881721343",
 &nbsp;&nbsp;&nbsp;&nbsp;"status":500,
 &nbsp;&nbsp;&nbsp;&nbsp;"error":"Internal&nbsp;Server&nbsp;Error",
 &nbsp;&nbsp;&nbsp;&nbsp;"exception":"com.netflix.hystrix.exception.HystrixRuntimeException",
 &nbsp;&nbsp;&nbsp;&nbsp;"message":"generate Intervention&nbsp;time-out&nbsp;and&nbsp;fallback&nbsp;failed.",
 &nbsp;&nbsp;&nbsp;&nbsp;"path":"/interventions/springhealth_user1/device_blood"
 }
</code></pre>
<p data-nodeid="683">在这里，我们发现 HTTP 响应状态为 500，而抛出的异常为 HystrixRuntimeException，从异常信息上可以看出引起该异常的原因是超时。事实上，默认情况下，添加了 @HystrixCommand 注解的方法调用超过了 1000 毫秒就会触发超时异常，显然上例中设置的 2000 毫秒满足触发条件。</p>
<p data-nodeid="684">和设置线程池属性一样，在 HystrixCommand 中我们也可以对熔断的超时时间、失败率等各项阈值进行设置。例如我们可以在 getDevice() 方法上添加如下配置项以改变 Hystrix 的默认行为：</p>
<pre class="lang-java" data-nodeid="685"><code data-language="java"><span class="hljs-meta">@HystrixCommand(commandProperties = {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "3000")
})</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> DeviceMapper <span class="hljs-title">getDevice</span><span class="hljs-params">(String deviceCode)</span>
</span></code></pre>
<p data-nodeid="686">上面示例中的 execution.isolation.thread.timeoutInMilliseconds 配置项就是用来设置 Hystrix 的超时时间，现在我们把它设置成 3000 毫秒。这时，我们再次访问 <a href="http://localhost:8083/interventionsorders/springhealth_user1/device_blood" data-nodeid="790">http://localhost:8083/interventions/springhealth_user1/device_blood</a>端点，就会发现请求会正常返回。当然，Hystrix 还提供了一系列的配置项来细化对熔断器的控制。常见的配置项如下所示：</p>
<pre class="lang-java" data-nodeid="687"><code data-language="java"><span class="hljs-meta">@HystrixCommand(commandProperties = {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "12000"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //一个滑动窗口内最小的请求数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "circuitBreaker.requestVolumeThreshold", value = "10"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //错误比率阈值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "circuitBreaker.errorThresholdPercentage", value = "75"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //触发熔断的时间值
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "7000"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //一个滑动窗口的时间长度
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "metrics.rollingStats.timeInMilliseconds", value = "15000"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //一个滑动窗口被划分的数量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name = "metrics.rollingStats.numBuckets", value = "5") })</span>
</code></pre>
<p data-nodeid="688">我们在后续介绍 Hystrix 熔断器实现原理和滑动窗口机制时会对这些配置项的作用做进一步展开。</p>
<h3 data-nodeid="689">使用 Hystrix 实现服务回退</h3>
<p data-nodeid="690">Hystrix 在服务调用失败时都可以执行服务回退逻辑。在开发过程上，我们只需要提供一个 Fallback 方法实现并进行配置即可。例如，在 SpringHealth 案例系统中，对于 intervention-service 中访问 user-service 和 device-service 这两个远程调用场景，我们都可以实现 Fallback 方法。回退方法的实现也非常方便，唯一需要注意的就是 Fallback 方法的参数和返回值必须与真实的方法完全一致。如下所示的就是 Fallback 方法的一个示例：</p>
<pre class="lang-java" data-nodeid="691"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> UserMapper <span class="hljs-title">getUserFallback</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserMapper fallbackUser = <span class="hljs-keyword">new</span> UserMapper(<span class="hljs-number">0L</span>,<span class="hljs-string">"no_user"</span>,<span class="hljs-string">"not_existed_user"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> fallbackUser;
}
</code></pre>
<p data-nodeid="692">我们通过构建一个不存在的 User 信息来返回 Fallback 结果。有了这个 Fallback 方法，剩下来要做的就是在 @HystrixCommand 注解中设置“fallbackMethod”配置项。重构后的 getUser 方法如下所示：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="801"><code data-language="java"><span class="hljs-meta">@HystrixCommand(threadPoolKey = "springHealthGroup",
&nbsp;&nbsp;&nbsp; threadPoolProperties =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name="coreSize",value="2"),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; @HystrixProperty(name="maxQueueSize",value="10")
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fallbackMethod = "getUserFallback"
)</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> UserMapper <span class="hljs-title">getUser</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> userClient.getUserByUserName(userName);
}
</code></pre>

<p data-nodeid="694">现在你可以模拟远程方法调用的各种异常情况，并观察这个 Fallback 是否已经生效了。</p>
<h3 data-nodeid="695">小结与预告</h3>
<p data-nodeid="696">本课时对 Hystrix 这款服务容错实现框架进行了详细了讨论，并结合 SpringHealth 案例系统给出了使用该框架的示例代码。Hystrix 是服务容错领域的代表性框架，包含了服务隔离、服务容错和服务回退功能，值得你进行深入的理解并掌握运用。为了帮助你更好的学习 Hystrix 框架，我们在后续课时中还会专门从源码级别分析他的实现原理。</p>
<p data-nodeid="697">这里给你留一道思考题：你能说出 Hystrix 中有哪些常见的配置项吗？</p>
<p data-nodeid="698" class="">讲完 Hystrix，我们将进一步讲解 Spring Cloud Circuit Breaker 框架。我们知道 Spring Cloud Circuit Breaker 中集成了多种服务容错框架，其中包括 Hystrix 却也不仅包括 Hystrix。下一课时，我们将首先探讨 Spring Cloud Circuit Breaker 中对服务容错的抽象机制，并完成对 Hystrix 使用方式的重构，以及介绍另一款主流的服务熔断工具 Resilience4j。</p>

---

### 精选评论

##### **7540：
> fallbackMethod的方法名，和你创建的不一样，这可以吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，只要跟你定义的业务方法一致就好

