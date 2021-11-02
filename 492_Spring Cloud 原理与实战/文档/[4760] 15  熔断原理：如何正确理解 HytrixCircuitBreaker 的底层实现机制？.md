<p data-nodeid="429354" class="">介绍完 Spring Cloud 中针对服务容错机制的 Spring Cloud Circuit Breaker 组件之后，今天我们还是探讨其中的 Hystrix 组件。Hystrix 作为最为经典的服务容错实现框架，我们有必须要对它的底层实现机制有一定的了解。</p>
<h3 data-nodeid="429355">HystrixCircuitBreaker</h3>
<p data-nodeid="429356">通过前面两个课时的介绍，我们知道在日常开发中，使用 Hystrix 熔断器的最简单方法就是在 Spring Boot 应用程序中添加 @EnableCircuitBreaker 注解。让我们先来分析一下这个注解背后的实现原理。</p>
<h4 data-nodeid="429357">@EnableCircuitBreaker 注解</h4>
<p data-nodeid="429358">@EnableCircuitBreaker 注解的作用就是告诉 Spring Cloud 在该服务中启用 Hystrix，其效果就相当于在应用程序中自动注入了熔断器。与介绍负载均衡时提到的 LoadBalancerClient 接口一样，Spring Cloud 也把 @EnableCircuitBreaker 注解作为一种公共组件放在 spring-cloud-commons 工程中，其定义如下所示：</p>
<pre class="lang-java" data-nodeid="429359"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Inherited</span>
<span class="hljs-meta">@Import(EnableCircuitBreakerImportSelector.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> EnableCircuitBreaker { 
}
</code></pre>
<p data-nodeid="429360">可以看到，这里通过 @Import 注解引入了 EnableCircuitBreakerImportSelector 类，该类定义如下：</p>
<pre class="lang-java" data-nodeid="429361"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EnableCircuitBreakerImportSelector</span> <span class="hljs-keyword">extends</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-title">SpringFactoryImportSelector</span>&lt;<span class="hljs-title">EnableCircuitBreaker</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> getEnvironment().getProperty(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"spring.cloud.circuit.breaker.enabled"</span>, Boolean.class, Boolean.TRUE);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
}
</code></pre>
<p data-nodeid="429362">关于 Spring Boot 中各种 ImportSelector 类的工作原理，其作用类似于 JDK 中的 SPI，它会在代码工程中的 META-INF/spring.factories 文件夹下寻找相应的配置项。上述的 EnableCircuitBreakerImportSelector 类会加载 spring.factories 中标明为 EnableCircuitBreaker 的配置类，如下所示：</p>
<pre class="lang-xml" data-nodeid="429363"><code data-language="xml">org.springframework.cloud.client.circuitbreaker.EnableCircuitBreaker=\
	org.springframework.cloud.netflix.hystrix.HystrixCircuitBreakerConfiguration
</code></pre>
<p data-nodeid="429364">可以看到这里指向了 HystrixCircuitBreakerConfiguration 这个配置类，注意该类位于 spring-cloud-netflix-core 工程中。Hystrix 在整个 Spring Cloud Netflix 框架中的地位很高，这点从它所处的工程位置就可见一斑。框架的设计者认为服务容错是最核心的机制，是所有服务都应该具备的基础功能，所以没有单独创建以 Hystrix 命名的工程，而是将其直接放在了 spring-cloud-netflix-core 工程中。实际上，该工程也只包含了 Hystrix 相关的类。</p>
<p data-nodeid="429365">HystrixCircuitBreakerConfiguration 配置类内部构造了一个切面，即 HystrixCommandAspect。从命名上看，HystrixCommandAspect 是一个用于拦截 HystrixCommand 的切面，这个切面对应的pointcut如下：</p>
<pre class="lang-java" data-nodeid="429366"><code data-language="java"><span class="hljs-meta">@Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand)")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hystrixCommandAnnotationPointcut</span><span class="hljs-params">()</span> </span>{
}
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Pointcut("@annotation(com.netflix.hystrix.contrib.javanica.annotation.HystrixCollapser)")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hystrixCollapserAnnotationPointcut</span><span class="hljs-params">()</span> </span>{
}
&nbsp;
<span class="hljs-meta">@Around("hystrixCommandAnnotationPointcut() || hystrixCollapserAnnotationPointcut()")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">methodsAnnotatedWithHystrixCommand</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ProceedingJoinPoint joinPoint)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
}
</code></pre>
<p data-nodeid="429367">可以看到使用 @HystrixCommand 注解和 @HystrixCollapser 注解修饰的方法都会被这个切面进行拦截，该切面的处理流程如下所示，为了演示简单，部分内容我做了裁剪：</p>
<pre class="lang-java" data-nodeid="429368"><code data-language="java"><span class="hljs-comment">//从切点获取所调用方法的相关信息</span>
Method method = getMethodFromTarget(joinPoint);
MetaHolderFactory metaHolderFactory = META_HOLDER_FACTORY_MAP.get(HystrixPointcutType.of(method));
&nbsp;
<span class="hljs-comment">//创建一个用于封装元数据的 MetaHolder，这些元数据包括方法中与 Hystrix 相关的信息</span>
MetaHolder metaHolder = metaHolderFactory.create(joinPoint);
&nbsp;
<span class="hljs-comment">//根据 MetaHolder 创建出一个 ystrixInvokable 接口，而 HystrixCommand 就是 HystrixInvokable 的实现类</span>
HystrixInvokable invokable = HystrixCommandFactory.getInstance().create(metaHolder);
&nbsp;
<span class="hljs-comment">//得到执行类型，包括同步、异步和响应式三种</span>
ExecutionType executionType = metaHolder.isCollapserAnnotationPresent() ?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; metaHolder.getCollapserExecutionType() : metaHolder.getExecutionType();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!metaHolder.isObservable()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result = CommandExecutor.execute(invokable, executionType, metaHolder);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result = executeObservable(invokable, executionType, metaHolder);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (HystrixBadRequestException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> e.getCause();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (HystrixRuntimeException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> hystrixRuntimeExceptionToThrowable(metaHolder, e);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
<span class="hljs-keyword">return</span> result;
</code></pre>
<p data-nodeid="429369">从这个处理流程中引出了 HystrixInvokable 接口，我们发现它实际上只是一个空接口。在 Hystrix中，存在一批以“-able”结尾的接口定义。例如，HystrixExecutable 和 HystrixObservable 接口就继承了 HystrixInvokable 接口，而这些接口最终都由各种以“-Command”结尾的类来负责实现。HystrixInvokable 接口以及相关的 Command 类的类层关系比较复杂，整体类层结构关系如下所示。</p>
<p data-nodeid="430531"><img src="https://s0.lgstatic.com/i/image/M00/6C/2B/CgqCHl-qaVeAPjoDAAGi61XW-lA326.png" alt="图片1.png" data-nodeid="430534"></p>

<div data-nodeid="430053"><p style="text-align:center">HystrixInvokable 接口的类层关系图</p></div>




<h4 data-nodeid="429372">HystrixCircuitBreaker 实现原理</h4>
<p data-nodeid="429373">我们无意对上图中的所有内容做详细展开，本课时的目的还是梳理整个熔断器的处理流程。所以，我们快速浏览这些类中的参数和代码结构，发现在 AbstractCommand 类中存在一个类型为 HystrixCircuitBreaker 接口的变量，同时在构造函数中给出了它的初始化方法，如下所示：</p>
<pre class="lang-java" data-nodeid="429374"><code data-language="java"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> HystrixCircuitBreaker circuitBreaker;

<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> HystrixCircuitBreaker <span class="hljs-title">initCircuitBreaker</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> enabled, HystrixCircuitBreaker fromConstructor,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCommandGroupKey groupKey, HystrixCommandKey commandKey,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCommandProperties properties, HystrixCommandMetrics metrics)</span> </span>{
&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span> (enabled) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (fromConstructor == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过工厂类获取 HystrixCircuitBreaker 的默认实现</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> HystrixCircuitBreaker.Factory.getInstance(commandKey, groupKey, properties, metrics);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> fromConstructor;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> NoOpCircuitBreaker();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="429375">注意：这里的 enabled 标志位就是前面介绍的 @EnableCircuitBreaker 注解中，通过 EnableCircuitBreakerImportSelector 的 isEnabled() 方法获取的配置，相当于是一个控制是否启用熔断器的开关。如果该标志位为 true，则会创建一个 HystrixCircuitBreaker 实例，反之则返回一个什么都不做的 NoOpCircuitBreaker。</p>
<p data-nodeid="429376">让我们来看一下 HystrixCircuitBreaker 接口的定义，如下所示：</p>
<pre class="lang-java" data-nodeid="429377"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">HystrixCircuitBreaker</span> </span>{
	
	<span class="hljs-comment">//请求是否可被执行</span>
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">allowRequest</span><span class="hljs-params">()</span></span>;
	&nbsp;
	<span class="hljs-comment">//返回当前熔断器是否打开</span>
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isOpen</span><span class="hljs-params">()</span></span>;
	&nbsp;
	<span class="hljs-comment">//关闭熔断器</span>
	<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">markSuccess</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="429378">可以看到 HystrixCircuitBreaker 接口只有三个方法，它的实现类为 HystrixCircuitBreakerImpl，该实现类通过一个 Factory 工厂类进行创建。我们先来看这个工厂类如何创建一个 HystrixCircuitBreaker 实例，也算是对工厂模式的回顾。</p>
<pre class="lang-java" data-nodeid="429379"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Factory</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> ConcurrentHashMap&lt;String, HystrixCircuitBreaker&gt; circuitBreakersByCommand = <span class="hljs-keyword">new</span> ConcurrentHashMap&lt;String, HystrixCircuitBreaker&gt;();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> HystrixCircuitBreaker <span class="hljs-title">getInstance</span><span class="hljs-params">(HystrixCommandKey key, HystrixCommandGroupKey group, HystrixCommandProperties properties, HystrixCommandMetrics metrics)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCircuitBreaker previouslyCached = circuitBreakersByCommand.get(key.name());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (previouslyCached != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> previouslyCached;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HystrixCircuitBreaker cbForCommand = circuitBreakersByCommand.putIfAbsent(key.name(), <span class="hljs-keyword">new</span> HystrixCircuitBreakerImpl(key, group, properties, metrics));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (cbForCommand == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> circuitBreakersByCommand.get(key.name());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> cbForCommand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> HystrixCircuitBreaker <span class="hljs-title">getInstance</span><span class="hljs-params">(HystrixCommandKey key)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> circuitBreakersByCommand.get(key.name());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">reset</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; circuitBreakersByCommand.clear();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="429380">这段代码很明显采用了基于 Key-Value 对的缓存设计思想，其中 Key 为 HystrixCommandKey 的 name，Value 为一个 HystrixCircuitBreakerImpl 实例。如果缓存中能够获取已有 Key 对应的 HystrixCircuitBreakerImpl 实例则直接返回，如果没有则创建一个新的实例并放入缓存。现在已经获取了一个 HystrixCircuitBreakerImpl 实例，让我们看看它如何实现 HystrixCircuitBreaker 的三个方法。</p>
<ul data-nodeid="429381">
<li data-nodeid="429382">
<p data-nodeid="429383">allowRequest()</p>
</li>
</ul>
<p data-nodeid="429384">当请求到来时，HystrixCommand 会首先调用 HystrixCircuitBreaker 中的 allowRequest 方法判断服务是否已经熔断了，而该判断取决于服务访问的失败率。allowRequest 方法首先会判断熔断器是否被强制打开或关闭。如果是强制打开，则直接拒绝请求；如果为强制关闭，则会调用下面要介绍的 isOpen() 方法来判断当前熔断器是否需要打开。如下所示：</p>
<pre class="lang-java" data-nodeid="429385"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">allowRequest</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (properties.circuitBreakerForceOpen().get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (properties.circuitBreakerForceClosed().get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; isOpen();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> !isOpen() || allowSingleTest();
}
</code></pre>
<ul data-nodeid="429386">
<li data-nodeid="429387">
<p data-nodeid="429388">isOpen()</p>
</li>
</ul>
<p data-nodeid="429389">该方法返回当前熔断器是否打开的状态。如果熔断器为 Open 状态，则直接返回 true。如果不是，则会从度量指标中获取请求健康信息并根据熔断阈值判断熔断结果，如下所示：</p>
<pre class="lang-java" data-nodeid="429390"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isOpen</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (circuitOpen.get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;HealthCounts health = metrics.getHealthCounts();
&nbsp;
	        <span class="hljs-comment">// 检查是否达到最小请求数,如果未达到的话即使请求全部失败也不会熔断</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (health.getTotalRequests() &lt; properties.circuitBreakerRequestVolumeThreshold().get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
	        <span class="hljs-comment">// 检查错误百分比是否达到设定的阈值，如果未达到的话也不会熔断</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (health.getErrorPercentage() &lt; properties.circuitBreakerErrorThresholdPercentage().get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">// 如果错误率过高，则进行熔断，并记录下熔断时间</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (circuitOpen.compareAndSet(<span class="hljs-keyword">false</span>, <span class="hljs-keyword">true</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; circuitOpenedOrLastTestedTime.set(System.currentTimeMillis());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<ul data-nodeid="429391">
<li data-nodeid="429392">
<p data-nodeid="429393">markSuccess()</p>
</li>
</ul>
<p data-nodeid="429394">HystrixCircuitBreaker 中的最后一个 markSuccess 方法用于关闭熔断器。在 HystrixCmmand 执行成功的情况下，通过调用该方法可以将打开的熔断器关闭，并重置度量指标对象，如下所示：</p>
<pre class="lang-java" data-nodeid="429395"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">markSuccess</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (circuitOpen.get()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (circuitOpen.compareAndSet(<span class="hljs-keyword">true</span>, <span class="hljs-keyword">false</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//重置度量指标对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; metrics.resetStream();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="429396">这三个方法的执行逻辑实际上都不复杂，HystrixCircuitBreaker 通过一个 circuitOpen 状态位控制着整个熔断判断流程，而这个状态位本身的状态值则取决于系统目前的执行数据和健康指标。我们在这三个方法中看到的 HealthCounts 和 HystrixCommandMetrics 都是这些指标的具体体现。而针对这些指标的采集和处理过程，Hystrix 提供了一套值得我们学习和借鉴的设计思想和实现机制，这就是滑动窗口（Rolling Window）机制。</p>
<h3 data-nodeid="429397">Hystrix 滑动窗口机制</h3>
<p data-nodeid="429398">首先，来看一下在 HystrixCircuitBreaker 的 isOpen() 中使用到的 HealthCounts 类，我们关注它所包含的变量以及这些变量的计算方法，如下所示：</p>
<pre class="lang-java" data-nodeid="429399"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HealthCounts</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> totalCount;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> errorCount;
	&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> errorPercentage;
	&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">public</span> HealthCounts <span class="hljs-title">plus</span><span class="hljs-params">(<span class="hljs-keyword">long</span>[] eventTypeCounts)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> updatedTotalCount = totalCount;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> updatedErrorCount = errorCount;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> successCount = eventTypeCounts[HystrixEventType.SUCCESS.ordinal()];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> failureCount = eventTypeCounts[HystrixEventType.FAILURE.ordinal()];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> timeoutCount = eventTypeCounts[HystrixEventType.TIMEOUT.ordinal()];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> threadPoolRejectedCount = eventTypeCounts[HystrixEventType.THREAD_POOL_REJECTED.ordinal()];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> semaphoreRejectedCount = eventTypeCounts[HystrixEventType.SEMAPHORE_REJECTED.ordinal()];
&nbsp;
&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;updatedTotalCount += (successCount + failureCount + timeoutCount + threadPoolRejectedCount + semaphoreRejectedCount);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; updatedErrorCount += (failureCount + timeoutCount + threadPoolRejectedCount + semaphoreRejectedCount);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> HealthCounts(updatedTotalCount, updatedErrorCount);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="429400">可以看到这个 plus() 方法比较复杂，我们需要明确所传入的 eventTypeCounts 数组的数据来源，这就要引出 Hystrix 中非常核心的一个类 HealthCountsStream。这个核心类在设计上采用了一种特定的机制，也就是所谓的滑动窗口机制。而 Hystrix 在实现这一机制时采用了数据流处理和响应式编程方面的技术和框架。作为知识储备，我们首先需要对这些机制和技术的原理做一定介绍，以便大家更好的理解实现一个熔断器所需要的考虑的各个方面。</p>
<h4 data-nodeid="429401">滑动窗口和数据流处理</h4>
<p data-nodeid="429402">通常，我们想要对一些数据做分析和统计时，会首先采集一定的样本，然后进行一定的分组。在采集策略上可以使用<strong data-nodeid="429492">时间维度</strong>或<strong data-nodeid="429493">数量维度</strong>。滑动窗口显然属于时间维度的采集方式，即采集过程基于一定的时间窗口，而这个时间窗口随着时间的演进而逐步向前滑动。</p>
<p data-nodeid="429403">在 Hystrix 中，采用滑动窗口来采集的系统运行时健康数据包括成功请求数量、失败请求数、超时请求数、被拒绝的请求数等。然后每次取最近 10 秒的数据来进行计算，如果这 10 秒中请求的失败率计算下来超过了 50%，就会触发熔断器的熔断机制。这里的 10 秒就是一个滑动窗口，参考其官网的一幅图：</p>
<p data-nodeid="431712"><img src="https://s0.lgstatic.com/i/image/M00/6C/20/Ciqc1F-qaXCACCVhAAFxRYCoxk8884.png" alt="图片2.png" data-nodeid="431715"></p>

<div data-nodeid="431234"><p style="text-align:center">滑动窗口示意图（来自 Hystrix 官网）</p></div>




<p data-nodeid="429406">在上图中，每一个格子就是一个时间间隔，格子中的数据就是这个时间间隔内所处理的请求数量。通常我们把这种格子称为一个<strong data-nodeid="429503">桶（Bucket）</strong>。然后每当收集好一个新桶之后，就会丢弃掉最旧的一个桶，这样时间窗口就能持续向前滑动。</p>
<p data-nodeid="429407">那么如何来实现这个滑动窗口呢？我们转换思路，可以把系统运行时所产生的所有数据都视为一个个的事件，这样滑动窗口中每个桶的数据都来自源源不断的事件，因此滑动窗口非常适合用观察者模式来实现。同时，对于这些生成的事件，我们通常需要对其进行转换以便执行后续的操作。这两点构成了实现滑动窗口的设计目标和方法。</p>
<h4 data-nodeid="429408">响应式编程和 RxJava 基础</h4>
<p data-nodeid="429409">在技术选型上，Hystrix 采用了<strong data-nodeid="429519">响应式编程框架 RxJava</strong>。如果大家想要完全掌握 Hystrix 中的源码实现过程，还是需要对响应式编程和 RxJava 框架有一定的了解。在响应式编程中，我们把源源不断的事件看成是一个<strong data-nodeid="429520">流（Stream）</strong>，业界也存在一个<strong data-nodeid="429521">响应式流（Reactive Stream）规范</strong>，该规范中的流是一个发布和订阅的过程。关于响应式流以及响应式编程的核心概念以及 RxJava、Project Reactor 等框架，可参考《RxJava反应式编程》和《Spring响应式编程》进行学习。在本课时中，我们无意对 RxJava 做全面而详细的介绍，这里仅仅给出理解 Hystrix 滑动窗口实现过程中所需要的必备知识。</p>
<p data-nodeid="429410">与其他响应式编程框架一样，RxJava 同样实现了响应式流规范。使用 RxJava 实现有一大好处是可以通过 RxJava 的一系列操作符来实现滑动窗口，从而可以依赖 RxJava 的线程模型来确保数据写入和聚合的线程安全。RxJava 中用于健康信息采集的操作符包括 window、flatMap 和 reduce 等。</p>
<ul data-nodeid="429411">
<li data-nodeid="429412">
<p data-nodeid="429413"><strong data-nodeid="429526">window 操作符。</strong></p>
</li>
</ul>
<p data-nodeid="433359">window 操作符用于<strong data-nodeid="433366">开窗操作</strong>，也就是把当前流中的元素采集并合并到另外的流中，该操作符示意图如下图所示：</p>
<p data-nodeid="433844"><img src="https://s0.lgstatic.com/i/image/M00/6C/2B/CgqCHl-qaYCAFzIcAAMR9rxJnM0280.png" alt="图片3.png" data-nodeid="433847"></p>




<div data-nodeid="432415"><p style="text-align:center">window 操作符示意图（来自 Reactor 官网）</p></div>




<p data-nodeid="429417">以上图为例，我们看到有 5 个元素从流中输入，然后我们对其进行开窗操作（窗口大小为 3），这样输入流就变成了两个输出流。</p>
<ul data-nodeid="429418">
<li data-nodeid="429419">
<p data-nodeid="429420"><strong data-nodeid="429540">flatMap 操作符。</strong></p>
</li>
</ul>
<p data-nodeid="429421">flatMap，也就是拉平并转化。与常见的 map 不同，flatMap 操作符把输入流中的每个元素转换成另一个流，再把这些转换之后得到的流元素进行合并。flapMap 操作符示意图如下图所示：</p>
<p data-nodeid="435025"><img src="https://s0.lgstatic.com/i/image/M00/6C/20/Ciqc1F-qaY-ADEiHAAIH-Hu23tg037.png" alt="图片4.png" data-nodeid="435028"></p>

<div data-nodeid="434547"><p style="text-align:center">flapMap 操作符示意图（来自 Reactor 官网）</p></div>




<p data-nodeid="429424">例如，我们对 1 和 5 使用 flatMap 操作，转换的逻辑是返回它们的平方值并进行合并，这样通过 flatMap 操作符之后这两个元素就变成了 1 和 25。</p>
<ul data-nodeid="429425">
<li data-nodeid="429426">
<p data-nodeid="429427"><strong data-nodeid="429549">reduce 操作符。</strong></p>
</li>
</ul>
<p data-nodeid="429428">reduce 操作符对流中包含的所有元素进行累积计算，该操作符示意图见下图所示：</p>
<p data-nodeid="436206"><img src="https://s0.lgstatic.com/i/image/M00/6C/20/Ciqc1F-qaZmAavNWAAKEDkEs9hw282.png" alt="图片5.png" data-nodeid="436209"></p>

<div data-nodeid="435728"><p style="text-align:center">reduce 操作符示意图（来自 Reactor 官网）</p></div>




<p data-nodeid="429431">上图中的具体累积操作通常也是通过一个函数来实现。例如，假如这个函数为一个求和函数，那么对 1 到 10 的数字进行求和时，reduce 操作符的运行结果即为 55。</p>
<p data-nodeid="429432">具备了这些基础知识之后，让我们回到 Hystrix 的 HealthCountsStream 类。</p>
<h4 data-nodeid="429433">HealthCountsStream</h4>
<p data-nodeid="429434">我们首先来看一下 HealthCountsStream 类的类层结构，如下图所示：</p>
<p data-nodeid="437387"><img src="https://s0.lgstatic.com/i/image/M00/6C/2B/CgqCHl-qaaeAdcTgAANQkzIcceY943.png" alt="图片6.png" data-nodeid="437390"></p>

<div data-nodeid="436909" class=""><p style="text-align:center">HealthCountsStream 类层结构图</p></div>




<p data-nodeid="429437">显然，从类的命名上不难看出，BucketedCounterStream 类代表一个窗口类，将基础数据汇总成一个个的桶。它的子类 BucketedRollingCounterStream 在它的基础上添加了滑动窗口处理，将桶汇总成滑动窗口。而 HealthCountsStream 最终表现为一个运行时健康数据流。</p>
<p data-nodeid="429438">在 BucketedCounterStream 类中，通过以下代码把事件流汇总成 Bucket：</p>
<pre class="lang-java" data-nodeid="429439"><code data-language="java"><span class="hljs-keyword">this</span>.bucketedStream = Observable.defer(<span class="hljs-keyword">new</span> Func0&lt;Observable&lt;Bucket&gt;&gt;() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Observable&lt;Bucket&gt; <span class="hljs-title">call</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> inputEventStream
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .observe()
	<span class="hljs-comment">// 使用 window 操作符收集一个 Bucket 时间内的数据</span>
.window(bucketSizeInMs, TimeUnit.MILLISECONDS) 
<span class="hljs-comment">// 将每个 window 内聚集起来的事件集合汇总成 Bucket</span>
.flatMap(reduceBucketToSummary)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .startWith(emptyEventCountsToStart);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
});
</code></pre>
<p data-nodeid="429440">可以看到，这里分别使用了前面介绍的 window 和 flatMap 操作符来完成桶的构建。请注意，该方法返回的是一个 Observable<code data-backticks="1" data-nodeid="429564">&lt;Bucket&gt;</code> 对象。在 RxJava 中，Observable 代表的就是一个<strong data-nodeid="429570">无限流对象</strong>。</p>
<p data-nodeid="429441">我们再来看 BucketedRollingCounterStream 类，该类的构造函数中同样存在一个类似的方法，如下所示（为了避免过于复杂，裁剪了部分代码）：</p>
<pre class="lang-java" data-nodeid="429442"><code data-language="java"><span class="hljs-keyword">this</span>.sourceStream = bucketedStream
	 <span class="hljs-comment">//将 N 个 Bucket 进行汇总</span>
	.window(numBuckets, <span class="hljs-number">1</span>)
	<span class="hljs-comment">//汇总成一个窗口</span>
	.flatMap(reduceWindowToSummary) 
&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//添加背压控制</span>
	.onBackpressureDrop();
</code></pre>
<p data-nodeid="429443">上述方法中基于父类 BucketedCounterStream 已经汇总的 bucketedStream 进行开窗处理，从而获取一个 sourceStream，这个 sourceStream 就是滑动窗口的最终形态。最后的 onBackpressureDrop() 语句是 RxJava 中提供的一种背压（Backpressure）机制，代表了一种流量控制策略，当消费者消费速度过慢时就丢弃数据，不进行积压。这里需要展开的是传入到 flatMap 操作符中的 reduceWindowToSummary 对象，该对象实际上是一个方法，用于设置flatMap的具体操作，定义如下：</p>
<pre class="lang-java" data-nodeid="429444"><code data-language="java">Func1&lt;Observable&lt;Bucket&gt;, Observable&lt;Output&gt;&gt; reduceWindowToSummary = <span class="hljs-keyword">new</span> Func1&lt;Observable&lt;Bucket&gt;, Observable&lt;Output&gt;&gt;() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Observable&lt;Output&gt; <span class="hljs-title">call</span><span class="hljs-params">(Observable&lt;Bucket&gt; window)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> window.scan(getEmptyOutputValue(), reduceBucket).skip(numBuckets);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
};
</code></pre>
<p data-nodeid="429445">这里使用了 <strong data-nodeid="429578">scan + skip</strong> 方法组合来完成类似 reduce 操作符的功能。这样做的主要是因为当数据流终结时，位于流末端的最后一个窗口内的数据往往是不完整的。这时候就需要把这些不完整的窗口进行过滤，从而确保数据不缺失。</p>
<p data-nodeid="429446">最后，我们回到 HealthCountsStream。HealthCountsStream 通过调用父类 BucketedRollingCounterStream 的构造函数完成自身流的构建，并在这个过程中使用了 HealthCounts 类的 plus 方法完成健康指标的计算。</p>
<p data-nodeid="429447">作为总结，Hystrix 巧妙地运用了 RxJava 中的 window、flatMap 等操作符来将单位窗口时间内的事件，及一个个窗口大小的桶聚集到一起，形成滑动窗口。并基于滑动窗口，集成指标数据。这个设计思想非常巧妙，值得我们深入研究并对基于流的处理过程中加以尝试和应用。</p>
<h3 data-nodeid="429448">小结与预告</h3>
<p data-nodeid="429449">今天我们继续讨论经典的 Hystrix 框架，但转换了切入点，我们从实现原理出发结合部分源码对该框架的底层实现机制进行了深入的剖析。我们分析了 @EnableCircuitBreaker 和 HystrixCircuitBreaker 的实现原理。更为重要的是，我们介绍了 Hystrix 中所采用的滑动窗口机制，这种机制对于我们理解和掌握流式数据处理有很大的参考价值。</p>
<p data-nodeid="429450">这里给你留一道思考题：你能简要说明 Hystrix 中的滑动窗口机制是如何做到对运行时数据进行采集和计算的吗？</p>
<p data-nodeid="429451" class="">讲完服务容错机制以及 Spring Cloud Circuit Breaker 框架，从下一课时开始，我们将进入一个新的主题的讲解，即配置中心。我们将首先讨论如何设计分布式环境下的配置中心解决方案。</p>

---

### 精选评论


