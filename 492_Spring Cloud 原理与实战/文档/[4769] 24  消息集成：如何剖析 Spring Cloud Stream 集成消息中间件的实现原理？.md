<p data-nodeid="529" class="">Spring Cloud Stream 中的内容比较多，今天我们重点关注的是如何实现 Spring Cloud Stream 与其他消息中间件的整合过程，因此只介绍消息发送和接收的主流程。我们将分别从<strong data-nodeid="606">Spring Cloud Stream</strong>以及<strong data-nodeid="607">消息中间件</strong>的角度出发，分析如何基于这一主流程，完成两者之间的无缝集成。</p>
<h3 data-nodeid="530">Spring Cloud Stream 中的 Binder</h3>
<p data-nodeid="531">通过前面几个课时的介绍，我们明确了 Binder 组件是 Spring Cloud Stream 与各种消息中间件进行集成的核心组件，而 Binder 组件的实现过程涉及一批核心类之间的相互协作。接下来，我们就对 Binder 相关的核心类做源码级的展开。</p>
<h4 data-nodeid="532">BindableProxyFactory</h4>
<p data-nodeid="533">我们知道在发送和接收消息时，需要使用 @EnableBinding 注解，该注解的作用就是告诉 Spring Cloud Stream 将该应用程序绑定到消息中间件，从而实现两者之间的连接。我们来到 org.springframework.cloud.stream.binding 包下的 BindableProxyFactory 类。根据该类上的注释，BindableProxyFactory 是用于初始化由 @EnableBinding 注解所提供接口的工厂类，该类的定义如下所示：</p>
<pre class="lang-java" data-nodeid="534"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BindableProxyFactory</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">MethodInterceptor</span>, <span class="hljs-title">FactoryBean</span>&lt;<span class="hljs-title">Object</span>&gt;, <span class="hljs-title">Bindable</span>, <span class="hljs-title">InitializingBean</span>
</span></code></pre>
<p data-nodeid="535">注意到 BindableProxyFactory 同时实现了 MethodInterceptor 接口和 Bindable 接口。其中前者是 AOP 中的方法拦截器，而后者是一个标明能够绑定 Input 和 Output 的接口。我们先来看 MethodInterceptor 中用于拦截的 invoke 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="536"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> Object <span class="hljs-title">invoke</span><span class="hljs-params">(MethodInvocation invocation)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Method method = invocation.getMethod();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object boundTarget = targetCache.get(method);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (boundTarget != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> boundTarget;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Input input = AnnotationUtils.findAnnotation(method, Input.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (input != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String name = BindingBeanDefinitionRegistryUtils.getBindingTargetName(input, method);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; boundTarget = <span class="hljs-keyword">this</span>.inputHolders.get(name).getBoundTarget();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; targetCache.put(method, boundTarget);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> boundTarget;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Output output = AnnotationUtils.findAnnotation(method, Output.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (output != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String name = BindingBeanDefinitionRegistryUtils.getBindingTargetName(output, method);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; boundTarget = <span class="hljs-keyword">this</span>.outputHolders.get(name).getBoundTarget();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; targetCache.put(method, boundTarget);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> boundTarget;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
}
</code></pre>
<p data-nodeid="537">这里的逻辑比较简单，可以看到 BindableProxyFactory 保存了一个缓存对象 targetCache。如果所调用方法已经存在于缓存中，则直接返回目标对象。反之，会根据 @Input 和 @Output 注解从 inputHolders 和 outputHolders 中获取对应的目标对象并放入缓存中。这里使用缓存的作用仅仅是为了加快每次方法调用的速度，而系统在初始化时通过重写 afterPropertiesSet 方法，已经将所有的目标对象都放置在 inputHolders 和 outputHolders 这两个集合中。至于这里提到的这个目标对象，暂时可以把它理解为就是一种 MessageChannel 对象，后面会对其进行展开。</p>
<p data-nodeid="538">然后我们来看 Bindable 接口的定义，如下所示：</p>
<pre class="lang-java" data-nodeid="539"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Bindable</span> </span>{
&nbsp; <span class="hljs-keyword">default</span> Collection&lt;Binding&lt;Object&gt;&gt; createAndBindInputs(BindingService adapter) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.&lt;Binding&lt;Object&gt;&gt;emptyList();
&nbsp; }
&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">void</span> <span class="hljs-title">bindOutputs</span><span class="hljs-params">(BindingService adapter)</span> </span>{}
&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">void</span> <span class="hljs-title">unbindInputs</span><span class="hljs-params">(BindingService adapter)</span> </span>{}
&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">void</span> <span class="hljs-title">unbindOutputs</span><span class="hljs-params">(BindingService adapter)</span> </span>{}
&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> Set&lt;String&gt; <span class="hljs-title">getInputs</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.emptySet();
&nbsp; }

&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> Set&lt;String&gt; <span class="hljs-title">getOutputs</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.emptySet();
&nbsp; }
}
</code></pre>
<p data-nodeid="540">显然，这个接口提供了对 Input 和 Output 的绑定和解绑操作。在 BindableProxyFactory 中，对以上几个方法的实现过程基本都类似，我们随机挑选一个 bindOutputs 方法进行展开，如下所示：</p>
<pre class="lang-java" data-nodeid="541"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">bindOutputs</span><span class="hljs-params">(BindingService bindingService)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;String, BoundTargetHolder&gt; boundTargetHolderEntry : <span class="hljs-keyword">this</span>.outputHolders.entrySet()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BoundTargetHolder boundTargetHolder = boundTargetHolderEntry.getValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String outputTargetName = boundTargetHolderEntry.getKey();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (boundTargetHolderEntry.getValue().isBindable()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (log.isDebugEnabled()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; log.debug(String.format(<span class="hljs-string">"Binding %s:%s:%s"</span>, <span class="hljs-keyword">this</span>.namespace, <span class="hljs-keyword">this</span>.type, outputTargetName));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bindingService.bindProducer(boundTargetHolder.getBoundTarget(), outputTargetName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="542">这里需要引入另一个重要的工具类 BindingService，该类提供了对 Input 和 Output 目标对象进行绑定的能力。但事实上，通过类上的注释可以看到，这也是一个外观类，它将底层的绑定动作委托给了 Binder。我们以绑定生产者的 bindProducer 方法为例展开讨论，该方法如下所示：</p>
<pre class="lang-java" data-nodeid="543"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">Binding&lt;T&gt; <span class="hljs-title">bindProducer</span><span class="hljs-params">(T output, String outputName)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String bindingTarget = <span class="hljs-keyword">this</span>.bindingServiceProperties
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getBindingDestination(outputName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Binder&lt;T, ?, ProducerProperties&gt; binder = (Binder&lt;T, ?, ProducerProperties&gt;) getBinder(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; outputName, output.getClass());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ProducerProperties producerProperties = <span class="hljs-keyword">this</span>.bindingServiceProperties
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getProducerProperties(outputName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (binder <span class="hljs-keyword">instanceof</span> ExtendedPropertiesBinder) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object extension = ((ExtendedPropertiesBinder) binder)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getExtendedProducerProperties(outputName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ExtendedProducerProperties extendedProducerProperties = <span class="hljs-keyword">new</span> ExtendedProducerProperties&lt;&gt;(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; extension);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; BeanUtils.copyProperties(producerProperties, extendedProducerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; producerProperties = extendedProducerProperties;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; validate(producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Binding&lt;T&gt; binding = doBindProducer(output, bindingTarget, binder, producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.producerBindings.put(outputName, binding);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> binding;
}
</code></pre>
<p data-nodeid="544">显然，这里的 doBindProducer 方法完成了真正的绑定操作，如下所示：</p>
<pre class="lang-java" data-nodeid="545"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">Binding&lt;T&gt; <span class="hljs-title">doBindProducer</span><span class="hljs-params">(T output, String bindingTarget, Binder&lt;T, ?, ProducerProperties&gt; binder,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ProducerProperties producerProperties)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.taskScheduler == <span class="hljs-keyword">null</span> || <span class="hljs-keyword">this</span>.bindingServiceProperties.getBindingRetryInterval() &lt;= <span class="hljs-number">0</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> binder.bindProducer(bindingTarget, output, producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> binder.bindProducer(bindingTarget, output, producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (RuntimeException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LateBinding&lt;T&gt; late = <span class="hljs-keyword">new</span> LateBinding&lt;T&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rescheduleProducerBinding(output, bindingTarget, binder, producerProperties, late, e);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> late;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="546">从这个方法中，我们终于看到了 Spring Cloud Stream 中最核心的概念 Binder，通过 Binder 的 bindProducer 方法完成了目标对象的绑定。</p>
<h4 data-nodeid="547">Binder</h4>
<p data-nodeid="548">Binder 是一个接口，分别提供了绑定生产者和消费者的方法，如下所示：</p>
<pre class="lang-java" data-nodeid="549"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Binder</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">C</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ConsumerProperties</span>, <span class="hljs-title">P</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ProducerProperties</span>&gt; </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Binding&lt;T&gt; <span class="hljs-title">bindConsumer</span><span class="hljs-params">(String name, String group, T inboundBindTarget, C consumerProperties)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Binding&lt;T&gt; <span class="hljs-title">bindProducer</span><span class="hljs-params">(String name, T outboundBindTarget, P producerProperties)</span></span>;
}
</code></pre>
<p data-nodeid="550">在介绍 Binder 接口的具体实现类之前，我们先来看一下如何获取一个 Binder，getBinder 方法如下所示。</p>
<pre class="lang-java" data-nodeid="551"><code data-language="java"><span class="hljs-keyword">protected</span> &lt;T&gt; Binder&lt;T, ?, ?&gt; getBinder(String channelName, Class&lt;T&gt; bindableType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String binderConfigurationName = <span class="hljs-keyword">this</span>.bindingServiceProperties.getBinder(channelName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> binderFactory.getBinder(binderConfigurationName, bindableType);
</code></pre>
<p data-nodeid="552">显然，这里用到了个工厂模式。工厂类 BinderFactory 的定义如下所示：</p>
<pre class="lang-java" data-nodeid="553"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">BinderFactory</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; &lt;T&gt; Binder&lt;T, ? extends ConsumerProperties, ? extends ProducerProperties&gt; getBinder(String configurationName,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Class&lt;? extends T&gt; bindableType);
}
</code></pre>
<p data-nodeid="554">BinderFactory 只有一个方法，根据给定的配置名称 configurationName 和绑定类型 bindableType 获取 Binder 实例。而 BinderFactory 的实现类也只有一个，即 DefaultBinderFactory。在该实现类的 getBinder 方法中对配置信息进行了校验，并通过 getBinderInstance 获取真正的 Binder 实例。在 getBinderInstance 方法中，我们通过一系列基于 Spring 容器的步骤构建了一个上下文对象 ConfigurableApplicationContext，并通过该上下文对象获取实现了 Binder 接口的 Java bean，核心代码就是下面这句：</p>
<pre class="lang-java" data-nodeid="555"><code data-language="java">Binder&lt;T, ?, ?&gt; binder = binderProducingContext.getBean(Binder.class);
</code></pre>
<p data-nodeid="556">当然，对于 BinderFactory 而言，缓存也是需要的。在 DefaultBinderFactory 中存在一个 binderInstanceCache 变量，使用了一个 Map 来保存配置名称所对应的 Binder 对象。</p>
<h4 data-nodeid="557">AbstractMessageChannelBinder</h4>
<p data-nodeid="558">既然我们已经能够获取 Binder 实例，接下去就来讨论 Binder 实例中对 bindConsumer 和 bindProducer 方法的实现过程。在 Spring Cloud Stream 中，Binder 接口的类层关系如下所示，注意到这里还展示了 spring-cloud-stream-binder-rabbit 代码工程中的 RabbitMessageChannelBinder 类，这个类在本课时讲到 Spring Cloud Stream 与 RabbitMQ 进行集成时会具体展开：</p>
<p data-nodeid="5805" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/81/B6/Ciqc1F_RmKqAehbuAAUIZHxCJ9g730.png" alt="图片1.png" data-nodeid="5809"></p>
<div data-nodeid="5806"><p style="text-align:center">Binder 接口类层结构图</p></div>




<p data-nodeid="561">Spring Cloud Stream 首先提供了一个 AbstractBinder，这是一个抽象类，提供的 bindConsumer 和 bindProducer 方法实现如下所示：</p>
<pre class="lang-java" data-nodeid="562"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Binding&lt;T&gt; <span class="hljs-title">bindConsumer</span><span class="hljs-params">(String name, String group, T target, C properties)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (StringUtils.isEmpty(group)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.isTrue(!properties.isPartitioned(), <span class="hljs-string">"A consumer group is required for a partitioned subscription"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> doBindConsumer(name, group, target, properties);
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> Binding&lt;T&gt; <span class="hljs-title">doBindConsumer</span><span class="hljs-params">(String name, String group, T inputTarget, C properties)</span></span>;
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Binding&lt;T&gt; <span class="hljs-title">bindProducer</span><span class="hljs-params">(String name, T outboundBindTarget, P properties)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> doBindProducer(name, outboundBindTarget, properties);
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> Binding&lt;T&gt; <span class="hljs-title">doBindProducer</span><span class="hljs-params">(String name, T outboundBindTarget, P properties)</span></span>;
</code></pre>
<p data-nodeid="563">可以看到，它对 Binder 接口中相关方法只是提供了空实现，并把具体实现过程通过 doBindConsumer 和 doBindProducer 抽象方法交由子类进行完成。显然，从设计模式上讲，AbstractBinder 应用了很典型的模板方法模式。</p>
<p data-nodeid="564">AbstractBinder 的子类是 AbstractMessageChannelBinder，它同样也是一个抽象类。我们来看它的 doBindProducer 方法，并对该方法中的核心语句进行提取和整理：</p>
<pre class="lang-java" data-nodeid="565"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Binding&lt;MessageChannel&gt; <span class="hljs-title">doBindProducer</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String destination, MessageChannel outputChannel,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> P producerProperties)</span> <span class="hljs-keyword">throws</span> BinderException </span>{
	 
	…
<span class="hljs-keyword">final</span> MessageHandler producerMessageHandler;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> ProducerDestination producerDestination;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; producerDestination = <span class="hljs-keyword">this</span>.provisioningProvider.provisionProducerDestination(destination,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SubscribableChannel errorChannel = producerProperties.isErrorChannelEnabled()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ? registerErrorInfrastructure(producerDestination) : <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; producerMessageHandler = createProducerMessageHandler(producerDestination, producerProperties,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; errorChannel);

	…
postProcessOutputChannel(outputChannel, producerProperties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ((SubscribableChannel) outputChannel).subscribe(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> SendingHandler(producerMessageHandler, HeaderMode.embeddedHeaders
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .equals(producerProperties.getHeaderMode()), <span class="hljs-keyword">this</span>.headersToEmbed,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; producerProperties.isUseNativeEncoding()));
&nbsp;
Binding&lt;MessageChannel&gt; binding = <span class="hljs-keyword">new</span> DefaultBinding&lt;MessageChannel&gt;(destination, <span class="hljs-keyword">null</span>, outputChannel,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; producerMessageHandler <span class="hljs-keyword">instanceof</span> Lifecycle ? (Lifecycle) producerMessageHandler : <span class="hljs-keyword">null</span>) {
	…
};
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; doPublishEvent(<span class="hljs-keyword">new</span> BindingCreatedEvent(binding));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> binding;
}
</code></pre>
<p data-nodeid="566">上述代码的核心逻辑在于，Source 里的 output 发送消息到 outputChannel 通道之后会被 SendingHandler 这个 MessageHandler 进行处理。从设计模式上讲，SendingHandler 是一个静态代理类，因此它又将这个处理过程委托给了由 createProducerMessageHandler 方法所创建的 producerMessageHandler，这点从 SendingHandler 的定义中可以得到验证，如下所示的 delegate 就是传入的 producerMessageHandler：</p>
<pre class="lang-java" data-nodeid="567"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SendingHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractMessageHandler</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Lifecycle</span> </span>{
&nbsp;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> MessageHandler delegate;
&nbsp;
        <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleMessageInternal</span><span class="hljs-params">(Message&lt;?&gt; message)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Message&lt;?&gt; messageToSend = (<span class="hljs-keyword">this</span>.useNativeEncoding) ? message
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; : serializeAndEmbedHeadersIfApplicable(message);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.delegate.handleMessage(messageToSend);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">// 省略其他方法</span>
}
</code></pre>
<p data-nodeid="568">请注意，同样作为一个模板方法类，AbstractMessageChannelBinder 具有三个抽象方法，即 createProducerMessageHandler、postProcessOutputChannel 和 afterUnbindProducer，这三个方法都需要由它的子类进行实现。也就是说，SendingHandler 所使用的 producerMessageHandler 需要由 AbstractMessageChannelBinder 子类负责进行创建。</p>
<p data-nodeid="569">需要注意的是，作为统一的数据模型，SendingHandler 以及 producerMessageHandler 中使用的都是 Spring Messaging 组件中的 Message 消息对象，而 createProducerMessageHandler 内部会把这个 Message 消息对象转换成对应中间件的消息数据格式并进行发送。</p>
<p data-nodeid="570">下面转到消息消费的场景，我们来看 AbstractMessageChannelBinder 的 doBindConsumer 方法。该方法的核心语句是创建一个消费者端点 ConsumerEndpoint，如下所示：</p>
<pre class="lang-java" data-nodeid="571"><code data-language="java">MessageProducer consumerEndpoint = createConsumerEndpoint(destination, group, properties);
consumerEndpoint.setOutputChannel(inputChannel);
</code></pre>
<p data-nodeid="572">这两行代码有两个注意点。首先，createConsumerEndpoint 是一个抽象方法，需要 AbstractMessageChannelBinder 的子类进行实现。与 createProducerMessageHandler 一样，createConsumerEndpoint 需要把中间件对应的消息数据结构转换成 Spring Messaging 中统一的 Message 消息对象。</p>
<p data-nodeid="573">然后，我们注意到这里的 consumerEndpoint 类型是 MessageProducer。MessageProducer 在 Spring Integration 中代表的是消息的生产者，它会把从第三方消息中间件中收到的消息转发到 inputChannel 所指定的通道中。基于 @StreamListener 注解，在 Spring Cloud Stream 中存在一个 StreamListenerMessageHandler 类，用于订阅 inputChannel 消息通道中传入的消息并进行消费。</p>
<p data-nodeid="574">作为总结，我们可以用如下所示的流程图来概括整个消息发送和消费流程：</p>
<p data-nodeid="5266" class=""><img src="https://s0.lgstatic.com/i/image/M00/81/C1/CgqCHl_RmIOAbYcHAAHhiR5WQIE310.png" alt="图片2.png" data-nodeid="5270"></p>
<div data-nodeid="5267"><p style="text-align:center">消息发送和消费整体流程图</p></div>




































<h3 data-nodeid="577">Spring Cloud Stream 集成 RabbitMQ</h3>
<p data-nodeid="578">到目前为止，Spring Cloud Stream 提供了对 RabbitMQ 和 Kafka 这两款主流消息中间件的集成。今天，我们选择使用 RabbitMQ 作为示例，讲解如何通过 Spring Cloud Stream 所提供的 Binder 完成与具体消息中间件的整合。</p>
<p data-nodeid="579">Spring Cloud Stream 团队提供了 spring-cloud-stream-binder-rabbit 作为与 RabbitMQ 集成的代码工程。这个工程只有四个类，我们需要重点关注的就是实现了 AbstractMessageChannelBinder 中几个抽象方法的 RabbitMessageChannelBinder 类。</p>
<h4 data-nodeid="580">集成消息发送</h4>
<p data-nodeid="581">首先找到 RabbitMessageChannelBinder中 的 createProducerMessageHandler 方法，我们知道该方法用于完成消息的发送。我们在 createProducerMessageHandler 中找到了以下核心代码：</p>
<pre class="lang-java" data-nodeid="582"><code data-language="java"><span class="hljs-keyword">final</span> AmqpOutboundEndpoint endpoint = <span class="hljs-keyword">new</span> AmqpOutboundEndpoint(&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; buildRabbitTemplate(producerProperties.getExtension(), errorChannel != <span class="hljs-keyword">null</span>));
endpoint.setExchangeName(producerDestination.getName());
</code></pre>
<p data-nodeid="583">首先，在 buildRabbitTemplate 方法中，我们看到了 RabbitTemplate 的构建过程。RabbitTemplate 是 Spring Amqp 组件中提供的专门用于封装与 RabbitMQ 底层交互 API 的模板类。在构建 RabbitTemplate 的整个过程中，涉及设置与 RabbitMQ 相关的 ConnectionFactory 等众多参数。</p>
<p data-nodeid="584">然后，我们发现 RabbitMessageChannelBinder 也是直接集成了 Spring Integration 中用于整合 AQMP 协议的 AmqpOutboundEndpoint。AmqpOutboundEndpoint 提供了如下所示的 send 方法进行消息的发送：</p>
<pre class="lang-java" data-nodeid="585"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(String exchangeName, String routingKey,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> Message&lt;?&gt; requestMessage, CorrelationData correlationData)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.amqpTemplate <span class="hljs-keyword">instanceof</span> RabbitTemplate) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MessageConverter converter = ((RabbitTemplate) <span class="hljs-keyword">this</span>.amqpTemplate).getMessageConverter();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; org.springframework.amqp.core.Message amqpMessage = MappingUtils.mapMessage(requestMessage, converter,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getHeaderMapper(), getDefaultDeliveryMode(), isHeadersMappedLast());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; addDelayProperty(requestMessage, amqpMessage);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ((RabbitTemplate) <span class="hljs-keyword">this</span>.amqpTemplate).send(exchangeName, routingKey, amqpMessage, correlationData);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.amqpTemplate.convertAndSend(exchangeName, routingKey, requestMessage.getPayload(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; message -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getHeaderMapper().fromHeadersToRequest(requestMessage.getHeaders(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; message.getMessageProperties());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> message;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="586">可以看到这里依赖于 Spring Amqp 提供的 AmqpTemplate 接口进行消息的发送，而 RabbitTemplate 是 AmqpTemplate 的一个实现类。同时，通过 Spring Integration 组件中提供的 MessageConverter 工具类完成了从 org.springframework.messaging.Message 到 org.springframework.amqp.core.Message 这两个消息数据结构之间的转换。</p>
<h4 data-nodeid="587">集成消息消费</h4>
<p data-nodeid="588">RabbitMessageChannelBinder 中与消息消费相关的是 createConsumerEndpoint 方法。这个方法中大量使用了 Spring Amqp 和 Spring Integration 中的工具类。该方法最终返回的是一个 AmqpInboundChannelAdapter 对象。在 Spring Integration 中，AmqpInboundChannelAdapter 是一种 InboundChannelAdapter，代表面向输入的通道适配器，提供了消息监听功能，如下所示</p>
<pre class="lang-java" data-nodeid="589"><code data-language="java"><span class="hljs-keyword">protected</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Listener</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ChannelAwareMessageListener</span>, <span class="hljs-title">RetryListener</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onMessage</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Message message, <span class="hljs-keyword">final</span> Channel channel)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略相关实现</span>
	 }
}
</code></pre>
<p data-nodeid="590">在这个 onMessage 方法中，调用了 createAndSend 方法完成消息的创建和发送，如下所示：</p>
<pre class="lang-java" data-nodeid="591"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createAndSend</span><span class="hljs-params">(Message message, Channel channel)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; org.springframework.messaging.Message&lt;Object&gt; messagingMessage = createMessage(message, channel);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; setAttributesIfNecessary(message, messagingMessage);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sendMessage(messagingMessage);
}
	&nbsp;
	&nbsp;
<span class="hljs-keyword">private</span> org.springframework.messaging.<span class="hljs-function">Message&lt;Object&gt; <span class="hljs-title">createMessage</span><span class="hljs-params">(Message message, Channel channel)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object payload = AmqpInboundChannelAdapter.<span class="hljs-keyword">this</span>.messageConverter.fromMessage(message);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; headers = AmqpInboundChannelAdapter.<span class="hljs-keyword">this</span>.headerMapper
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .toHeadersFromRequest(message.getMessageProperties());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (AmqpInboundChannelAdapter.<span class="hljs-keyword">this</span>.messageListenerContainer.getAcknowledgeMode()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; == AcknowledgeMode.MANUAL) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; headers.put(AmqpHeaders.DELIVERY_TAG, message.getMessageProperties().getDeliveryTag());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; headers.put(AmqpHeaders.CHANNEL, channel);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (AmqpInboundChannelAdapter.<span class="hljs-keyword">this</span>.retryTemplate != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; headers.put(IntegrationMessageHeaderAccessor.DELIVERY_ATTEMPT, <span class="hljs-keyword">new</span> AtomicInteger());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> org.springframework.messaging.Message&lt;Object&gt; messagingMessage = getMessageBuilderFactory()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withPayload(payload)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .copyHeaders(headers)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> messagingMessage;
}
</code></pre>
<p data-nodeid="592">显然，在上述 createMessage 方法中，我们完成了消息数据格式从 org.springframework.amqp.core.Message 到 org.springframework.messaging.Message 的反向转换。</p>
<h3 data-nodeid="593">小结与预告</h3>
<p data-nodeid="594">Binder 是 Spring Cloud Stream 的核心组件，通过这个组件，Spring Cloud Stream 完成了与第三方消息中间件的集成。在本课时中，我们花了较大篇幅系统分析了 Binder 组件相关的核心类。然后，基于这些核心类以及 RabbitMQ，我们给出了 Spring Cloud Stream 集成RabbitMQ 的实现原理。</p>
<p data-nodeid="595">这里给你留一道思考题：在 Spring Cloud Stream 中，Binder 组件对于消息发送和消费做了哪些抽象？</p>
<p data-nodeid="596" class="">介绍完 Spring Cloud Stream 之后，我们又将启动一个新的话题，即安全性。在微服务架构中，安全性的重要性往往被忽略，值得我们系统的进行分析和实现。下一课时，我们首先关注如何理解微服务访问的安全需求和实现方案。</p>

---

### 精选评论

##### **栋：
> 每一个服务示例要批量消费模式，有多个主题的话要自定义吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的

