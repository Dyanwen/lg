<p data-nodeid="426871" class="">从这一课时我们就进入“集群”模块了，今天我们分享的是一篇加餐文章，主题是：深潜 Directory 实现，探秘服务目录玄机。</p>
<p data-nodeid="426872">在生产环境中，为了保证服务的可靠性、吞吐量以及容错能力，我们通常会在多个服务器上运行相同的服务端程序，然后以<strong data-nodeid="426993">集群</strong>的形式对外提供服务。根据各项性能指标的要求不同，各个服务端集群中服务实例的个数也不尽相同，从几个实例到几百个实例不等。</p>
<p data-nodeid="426873">对于客户端程序来说，就会出现几个问题：</p>
<ul data-nodeid="426874">
<li data-nodeid="426875">
<p data-nodeid="426876">客户端程序是否要感知每个服务端地址？</p>
</li>
<li data-nodeid="426877">
<p data-nodeid="426878">客户端程序的一次请求，到底调用哪个服务端程序呢？</p>
</li>
<li data-nodeid="426879">
<p data-nodeid="426880">请求失败之后的处理是重试，还会是抛出异常？</p>
</li>
<li data-nodeid="426881">
<p data-nodeid="426882">如果是重试，是再次请求该服务实例，还是尝试请求其他服务实例？</p>
</li>
<li data-nodeid="426883">
<p data-nodeid="426884">服务端集群如何做到负载均衡，负载均衡的标准是什么呢？</p>
</li>
<li data-nodeid="426885">
<p data-nodeid="426886">……</p>
</li>
</ul>
<p data-nodeid="426887">为了解决上述问题，<strong data-nodeid="427006">Dubbo 独立出了一个实现集群功能的模块—— dubbo-cluster</strong>。</p>
<p data-nodeid="426888"><img src="https://s0.lgstatic.com/i/image/M00/6B/E0/Ciqc1F-qN92ADHx8AACiY_cvusQ921.png" alt="Drawing 0.png" data-nodeid="427009"></p>
<div data-nodeid="426889"><p style="text-align:center">dubbo-cluster 结构图</p></div>
<p data-nodeid="426890">作为 dubbo-cluster 模块分析的第一课时，我们就首先来了解一下 dubbo-cluster 模块的架构以及最核心的 Cluster 接口。</p>
<h3 data-nodeid="426891">Cluster 架构</h3>
<p data-nodeid="427785">dubbo-cluster 模块的主要功能是将多个 Provider 伪装成一个 Provider 供 Consumer 调用，其中涉及集群的容错处理、路由规则的处理以及负载均衡。下图展示了 dubbo-cluster 的核心组件：</p>
<p data-nodeid="427786" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/6C/18/Ciqc1F-qY-CAQ08VAAFAZaC5kyU044.png" alt="Lark20201110-175555.png" data-nodeid="427791"></p>
<div data-nodeid="427787"><p style="text-align:center">Cluster 核心接口图</p></div>




<p data-nodeid="426895">由图我们可以看出，dubbo-cluster 主要包括以下四个核心接口：</p>
<ul data-nodeid="426896">
<li data-nodeid="426897">
<p data-nodeid="426898"><strong data-nodeid="427021">Cluster 接口</strong>，是集群容错的接口，主要是在某些 Provider 节点发生故障时，让 Consumer 的调用请求能够发送到正常的 Provider 节点，从而保证整个系统的可用性。</p>
</li>
<li data-nodeid="426899">
<p data-nodeid="426900"><strong data-nodeid="427026">Directory 接口</strong>，表示多个 Invoker 的集合，是后续路由规则、负载均衡策略以及集群容错的基础。</p>
</li>
<li data-nodeid="426901">
<p data-nodeid="426902"><strong data-nodeid="427031">Router 接口</strong>，抽象的是路由器，请求经过 Router 的时候，会按照用户指定的规则匹配出符合条件的 Provider。</p>
</li>
<li data-nodeid="426903">
<p data-nodeid="426904"><strong data-nodeid="427036">LoadBalance 接口</strong>，是负载均衡接口，Consumer 会按照指定的负载均衡策略，从 Provider 集合中选出一个最合适的 Provider 节点来处理请求。</p>
</li>
</ul>
<p data-nodeid="426905">Cluster 层的核心流程是这样的：当调用进入 Cluster 的时候，Cluster 会创建一个 AbstractClusterInvoker 对象，在这个 AbstractClusterInvoker 中，首先会从 Directory 中获取当前 Invoker 集合；然后按照 Router 集合进行路由，得到符合条件的 Invoker 集合；接下来按照 LoadBalance 指定的负载均衡策略得到最终要调用的 Invoker 对象。</p>
<p data-nodeid="426906">了解了 dubbo-cluster 模块的核心架构和基础组件之后，我们后续将会按照上面架构图的顺序介绍每个接口的定义以及相关实现。</p>
<h3 data-nodeid="426907">Directory 接口详解</h3>
<p data-nodeid="426908">Directory 接口表示的是一个集合，该集合由多个 Invoker 构成，后续的路由处理、负载均衡、集群容错等一系列操作都是在 Directory 基础上实现的。</p>
<p data-nodeid="426909">下面我们深入分析一下 Directory 的相关内容，首先是 Directory 接口中定义的方法：</p>
<pre class="lang-java" data-nodeid="426910"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Directory</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">Node</span> </span>{
    <span class="hljs-comment">// 服务接口类型</span>
&nbsp; &nbsp; <span class="hljs-function">Class&lt;T&gt; <span class="hljs-title">getInterface</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-comment">// list()方法会根据传入的Invocation请求，过滤自身维护的Invoker集合，返回符合条件的Invoker集合</span>
&nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; list(Invocation invocation) <span class="hljs-keyword">throws</span> RpcException;
    <span class="hljs-comment">// getAllInvokers()方法返回当前Directory对象维护的全部Invoker对象</span>
&nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; getAllInvokers();
    <span class="hljs-comment">// Consumer端的URL</span>
&nbsp; &nbsp; <span class="hljs-function">URL <span class="hljs-title">getConsumerUrl</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="426911"><strong data-nodeid="427046">AbstractDirectory 是 Directory 接口的抽象实现</strong>，其中除了维护 Consumer 端的 URL 信息，还维护了一个 RouterChain 对象，用于记录当前使用的 Router 对象集合，也就是后面课时要介绍的路由规则。</p>
<p data-nodeid="426912">AbstractDirectory 对 list() 方法的实现也比较简单，就是直接委托给了 doList() 方法，doList() 是个抽象方法，由 AbstractDirectory 的子类具体实现。</p>
<p data-nodeid="426913"><strong data-nodeid="427052">Directory 接口有 RegistryDirectory 和 StaticDirectory 两个具体实现</strong>，如下图所示：</p>
<p data-nodeid="426914"><img src="https://s0.lgstatic.com/i/image/M00/6B/E0/Ciqc1F-qN_-AMVHmAAA3C6TAxsA315.png" alt="Drawing 2.png" data-nodeid="427055"></p>
<div data-nodeid="426915"><p style="text-align:center">Directory 接口继承关系图</p></div>
<p data-nodeid="426916">其中，<strong data-nodeid="427073">RegistryDirectory 实现</strong>中维护的 Invoker 集合会随着注册中心中维护的注册信息<strong data-nodeid="427074">动态</strong>发生变化，这就依赖了 ZooKeeper 等注册中心的推送能力；<strong data-nodeid="427075">StaticDirectory 实现</strong>中维护的 Invoker 集合则是<strong data-nodeid="427076">静态</strong>的，在 StaticDirectory 对象创建完成之后，不会再发生变化。</p>
<p data-nodeid="426917">下面我们就来分别介绍 Directory 接口的这两个具体实现。</p>
<h4 data-nodeid="426918">1. StaticDirectory</h4>
<p data-nodeid="426919">StaticDirectory 这个 Directory 实现比较简单，在构造方法中，StaticDirectory 会接收一个 Invoker 集合，并赋值到自身的 invokers 字段中，作为底层的 Invoker 集合。在 doList() 方法中，StaticDirectory 会使用 RouterChain 中的 Router 从 invokers 集合中过滤出符合路由规则的 Invoker 对象集合，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426920"><code data-language="java"><span class="hljs-keyword">protected</span> List&lt;Invoker&lt;T&gt;&gt; doList(Invocation invocation) <span class="hljs-keyword">throws</span> RpcException {
&nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; finalInvokers = invokers;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (routerChain != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 通过RouterChain过滤出符合条件的Invoker集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; finalInvokers = routerChain.route(getConsumerUrl(), invocation);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> finalInvokers == <span class="hljs-keyword">null</span> ? Collections.emptyList() : finalInvokers;
}
</code></pre>
<p data-nodeid="426921">在创建 StaticDirectory 对象的时候，如果没有传入 RouterChain 对象，则会根据 URL 构造一个包含内置 Router 的 RouterChain 对象：</p>
<pre class="lang-java" data-nodeid="426922"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">buildRouterChain</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; RouterChain&lt;T&gt; routerChain = RouterChain.buildChain(getUrl()); <span class="hljs-comment">// 创建内置Router集合</span>
    <span class="hljs-comment">// 将invokers与RouterChain关联</span>
&nbsp; &nbsp; routerChain.setInvokers(invokers);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.setRouterChain(routerChain); <span class="hljs-comment">// 设置routerChain字段</span>
}
</code></pre>
<h4 data-nodeid="426923">2. RegistryDirectory</h4>
<p data-nodeid="426924">RegistryDirectory 是一个动态的 Directory 实现，<strong data-nodeid="427091">实现了 NotifyListener 接口</strong>，当注册中心的服务配置发生变化时，RegistryDirectory 会收到变更通知，然后RegistryDirectory 会根据注册中心推送的通知，动态增删底层 Invoker 集合。</p>
<p data-nodeid="426925">下面我们先来看一下 RegistryDirectory 中的核心字段。</p>
<ul data-nodeid="426926">
<li data-nodeid="426927">
<p data-nodeid="426928">cluster（Cluster 类型）：集群策略适配器，这里通过 Dubbo SPI 方式（即 ExtensionLoader.getAdaptiveExtension() 方法）动态创建适配器实例。</p>
</li>
<li data-nodeid="426929">
<p data-nodeid="426930">routerFactory（RouterFactory 类型）：路由工厂适配器，也是通过 Dubbo SPI 动态创建的适配器实例。routerFactory 字段和 cluster 字段都是静态字段，多个 RegistryDirectory 对象通用。</p>
</li>
<li data-nodeid="426931">
<p data-nodeid="426932">serviceKey（String 类型）：服务对应的 ServiceKey，默认是 {interface}:[group]:[version] 三部分构成。</p>
</li>
<li data-nodeid="426933">
<p data-nodeid="426934">serviceType（Class 类型）：服务接口类型，例如，org.apache.dubbo.demo.DemoService。</p>
</li>
<li data-nodeid="426935">
<p data-nodeid="426936">queryMap（Map&lt;String, String&gt; 类型）：Consumer URL 中 refer 参数解析后得到的全部 KV。</p>
</li>
<li data-nodeid="426937">
<p data-nodeid="426938">directoryUrl（URL 类型）：只保留 Consumer 属性的 URL，也就是由 queryMap 集合重新生成的 URL。</p>
</li>
<li data-nodeid="426939">
<p data-nodeid="426940">multiGroup（boolean类型）：是否引用多个服务组。</p>
</li>
<li data-nodeid="426941">
<p data-nodeid="426942">protocol（Protocol 类型）：使用的 Protocol 实现。</p>
</li>
<li data-nodeid="426943">
<p data-nodeid="426944">registry（Registry 类型）：使用的注册中心实现。</p>
</li>
<li data-nodeid="426945">
<p data-nodeid="426946">invokers（volatile List&lt;Invoker&gt; 类型）：动态更新的 Invoker 集合。</p>
</li>
<li data-nodeid="426947">
<p data-nodeid="426948">urlInvokerMap（volatile Map&lt; String, Invoker&gt; 类型）：Provider URL 与对应 Invoker 之间的映射，该集合会与 invokers 字段同时动态更新。</p>
</li>
<li data-nodeid="426949">
<p data-nodeid="426950">cachedInvokerUrls（volatile Set类型）：当前缓存的所有 Provider 的 URL，该集合会与 invokers 字段同时动态更新。</p>
</li>
<li data-nodeid="426951">
<p data-nodeid="426952">configurators（volatile List&lt; Configurator&gt;类型）：动态更新的配置信息，配置的具体内容在后面的分析中会介绍到。</p>
</li>
</ul>
<p data-nodeid="426953">在 RegistryDirectory 的构造方法中，会<strong data-nodeid="427127">根据传入的注册中心 URL 初始化上述核心字段</strong>，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426954"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">RegistryDirectory</span><span class="hljs-params">(Class&lt;T&gt; serviceType, URL url)</span> </span>{
   <span class="hljs-comment">// 传入的url参数是注册中心的URL，例如，zookeeper://127.0.0.1:2181/org.apache.dubbo.registry.RegistryService?...，其中refer参数包含了Consumer信息，例如，refer=application=dubbo-demo-api-consumer&amp;dubbo=2.0.2&amp;interface=org.apache.dubbo.demo.DemoService&amp;pid=13423&amp;register.ip=192.168.124.3&amp;side=consumer(URLDecode之后的值)</span>
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(url); 
&nbsp; &nbsp; shouldRegister = !ANY_VALUE.equals(url.getServiceInterface()) &amp;&amp; url.getParameter(REGISTER_KEY, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; shouldSimplified = url.getParameter(SIMPLIFIED_KEY, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.serviceType = serviceType;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.serviceKey = url.getServiceKey();
    <span class="hljs-comment">// 解析refer参数值，得到其中Consumer的属性信息</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.queryMap = StringUtils.parseQueryString(url.getParameterAndDecoded(REFER_KEY));
    <span class="hljs-comment">// 将queryMap中的KV作为参数，重新构造URL，其中的protocol和path部分不变</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.overrideDirectoryUrl = <span class="hljs-keyword">this</span>.directoryUrl = turnRegistryUrlToConsumerUrl(url);
&nbsp; &nbsp; String group = directoryUrl.getParameter(GROUP_KEY, <span class="hljs-string">""</span>);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.multiGroup = group != <span class="hljs-keyword">null</span> &amp;&amp; (ANY_VALUE.equals(group) || group.contains(<span class="hljs-string">","</span>));
}
</code></pre>
<p data-nodeid="426955">在完成初始化之后，我们来看 subscribe() 方法，该方法会在 Consumer 进行订阅的时候被调用，其中调用 Registry 的 subscribe() 完成订阅操作，同时还会将当前 RegistryDirectory 对象作为 NotifyListener 监听器添加到 Registry 中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426956"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">subscribe</span><span class="hljs-params">(URL url)</span> </span>{
&nbsp; &nbsp; setConsumerUrl(url);
    <span class="hljs-comment">// 将当前RegistryDirectory对象作为ConfigurationListener记录到CONSUMER_CONFIGURATION_LISTENER中</span>
&nbsp; &nbsp; CONSUMER_CONFIGURATION_LISTENER.addNotifyListener(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; serviceConfigurationListener = <span class="hljs-keyword">new</span> ReferenceConfigurationListener(<span class="hljs-keyword">this</span>, url);
    <span class="hljs-comment">// 完成订阅操作，注册中心的相关操作在前文已经介绍过了，这里不再重复</span>
&nbsp; &nbsp; registry.subscribe(url, <span class="hljs-keyword">this</span>);
}
</code></pre>
<p data-nodeid="426957">我们看到除了作为 NotifyListener 监听器之外，RegistryDirectory 内部还有两个 ConfigurationListener 的内部类（继承关系如下图所示），为了保持连贯，这两个监听器的具体原理我们在后面的课时中会详细介绍，这里先不展开讲述。</p>
<p data-nodeid="426958"><img src="https://s0.lgstatic.com/i/image/M00/6B/EC/CgqCHl-qOBmAbzKkAABZPyC5mIA963.png" alt="Drawing 3.png" data-nodeid="427132"></p>
<div data-nodeid="426959"><p style="text-align:center">RegistryDirectory 内部的 ConfigurationListener 实现</p></div>
<p data-nodeid="426960">通过前面对 Registry 的介绍我们知道，在注册 NotifyListener 的时候，监听的是 providers、configurators 和 routers 三个目录，所以在这三个目录下发生变化的时候，就会触发 RegistryDirectory 的 notify() 方法。</p>
<p data-nodeid="426961">在 RegistryDirectory.notify() 方法中，首先会按照 category 对发生变化的 URL 进行分类，分成 configurators、routers、providers 三类，并分别对不同类型的 URL 进行处理：</p>
<ul data-nodeid="426962">
<li data-nodeid="426963">
<p data-nodeid="426964">将 configurators 类型的 URL 转化为 Configurator，保存到 configurators 字段中；</p>
</li>
<li data-nodeid="426965">
<p data-nodeid="426966">将 router 类型的 URL 转化为 Router，并通过&nbsp;routerChain.addRouters()&nbsp;方法添加 routerChain 中保存；</p>
</li>
<li data-nodeid="426967">
<p data-nodeid="426968">将 provider 类型的 URL 转化为 Invoker 对象，并记录到 invokers 集合和 urlInvokerMap 集合中。</p>
</li>
</ul>
<p data-nodeid="426969">notify() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="426970"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> <span class="hljs-keyword">void</span> <span class="hljs-title">notify</span><span class="hljs-params">(List&lt;URL&gt; urls)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 按照category进行分类，分成configurators、routers、providers三类</span>
&nbsp; &nbsp; Map&lt;String, List&lt;URL&gt;&gt; categoryUrls = urls.stream()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .filter(Objects::nonNull)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .filter(<span class="hljs-keyword">this</span>::isValidCategory)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .filter(<span class="hljs-keyword">this</span>::isNotCompatibleFor26x)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .collect(Collectors.groupingBy(<span class="hljs-keyword">this</span>::judgeCategory));
&nbsp; &nbsp; <span class="hljs-comment">// 获取configurators类型的URL，并转换成Configurator对象</span>
&nbsp; &nbsp; List&lt;URL&gt; configuratorURLs = categoryUrls.getOrDefault(CONFIGURATORS_CATEGORY, Collections.emptyList());
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.configurators = Configurator.toConfigurators(configuratorURLs).orElse(<span class="hljs-keyword">this</span>.configurators);

&nbsp; &nbsp; <span class="hljs-comment">// 获取routers类型的URL，并转成Router对象，添加到RouterChain中</span>
&nbsp; &nbsp; List&lt;URL&gt; routerURLs = categoryUrls.getOrDefault(ROUTERS_CATEGORY, Collections.emptyList());
&nbsp; &nbsp; toRouters(routerURLs).ifPresent(<span class="hljs-keyword">this</span>::addRouters);
&nbsp; &nbsp; <span class="hljs-comment">// 获取providers类型的URL，调用refreshOverrideAndInvoker()方法进行处理</span>
&nbsp; &nbsp; List&lt;URL&gt; providerURLs = categoryUrls.getOrDefault(PROVIDERS_CATEGORY, Collections.emptyList());
&nbsp; &nbsp; ... <span class="hljs-comment">// 在Dubbo3.0中会触发AddressListener监听器，但是现在AddressListener接口还没有实现，所以省略这段代码</span>
&nbsp; &nbsp; refreshOverrideAndInvoker(providerURLs);
}
</code></pre>
<p data-nodeid="426971">我们这里首先来专注<strong data-nodeid="427144">providers 类型 URL 的处理</strong>，具体实现位置在 refreshInvoker() 方法中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426972"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">refreshInvoker</span><span class="hljs-params">(List&lt;URL&gt; invokerUrls)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 如果invokerUrls集合不为空，长度为1，并且协议为empty，则表示该服务的所有Provider都下线了，会销毁当前所有Provider对应的Invoker。</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokerUrls.size() == <span class="hljs-number">1</span> &amp;&amp; invokerUrls.get(<span class="hljs-number">0</span>) != <span class="hljs-keyword">null</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &amp;&amp; EMPTY_PROTOCOL.equals(invokerUrls.get(<span class="hljs-number">0</span>).getProtocol())) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.forbidden = <span class="hljs-keyword">true</span>; <span class="hljs-comment">// forbidden标记设置为true，后续请求将直接抛出异常</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.invokers = Collections.emptyList();
&nbsp; &nbsp; &nbsp; &nbsp; routerChain.setInvokers(<span class="hljs-keyword">this</span>.invokers); <span class="hljs-comment">// 清空RouterChain中的Invoker集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; destroyAllInvokers(); <span class="hljs-comment">// 关闭所有Invoker对象</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.forbidden = <span class="hljs-keyword">false</span>; <span class="hljs-comment">// forbidden标记设置为false，RegistryDirectory可以正常处理后续请求</span>
&nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, Invoker&lt;T&gt;&gt; oldUrlInvokerMap = <span class="hljs-keyword">this</span>.urlInvokerMap; <span class="hljs-comment">// 保存本地引用</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokerUrls == Collections.&lt;URL&gt;emptyList()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invokerUrls = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokerUrls.isEmpty() &amp;&amp; <span class="hljs-keyword">this</span>.cachedInvokerUrls != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果invokerUrls集合为空，并且cachedInvokerUrls不为空，则将使用cachedInvokerUrls缓存的数据，</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 也就是说注册中心中的providers目录未发生变化，invokerUrls则为空，表示cachedInvokerUrls集合中缓存的URL为最新的值</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invokerUrls.addAll(<span class="hljs-keyword">this</span>.cachedInvokerUrls);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果invokerUrls集合不为空，则用invokerUrls集合更新cachedInvokerUrls集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 也就是说，providers发生变化，invokerUrls集合中会包含此时注册中心所有的服务提供者</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.cachedInvokerUrls = <span class="hljs-keyword">new</span> HashSet&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.cachedInvokerUrls.addAll(invokerUrls);<span class="hljs-comment">//Cached invoker urls, convenient for comparison</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokerUrls.isEmpty()) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>; <span class="hljs-comment">// 如果invokerUrls集合为空，即providers目录未发生变更，则无须处理，结束本次更新服务提供者Invoker操作。</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将invokerUrls转换为对应的Invoker映射关系</span>
&nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, Invoker&lt;T&gt;&gt; newUrlInvokerMap = toInvokers(invokerUrls);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (CollectionUtils.isEmptyMap(newUrlInvokerMap)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 更新invokers字段和urlInvokerMap集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; newInvokers = Collections.unmodifiableList(<span class="hljs-keyword">new</span> ArrayList&lt;&gt;(newUrlInvokerMap.values()));
&nbsp; &nbsp; &nbsp; &nbsp; routerChain.setInvokers(newInvokers);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 针对multiGroup的特殊处理，合并多个group的Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.invokers = multiGroup ? toMergeInvokerList(newInvokers) : newInvokers;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.urlInvokerMap = newUrlInvokerMap;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 比较新旧两组Invoker集合，销毁掉已经下线的Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; destroyUnusedInvokers(oldUrlInvokerMap, newUrlInvokerMap);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="426973">通过对 refreshInvoker() 方法的介绍，我们可以看出，其<strong data-nodeid="427150">最核心的逻辑是 Provider URL 转换成 Invoker 对象，也就是 toInvokers() 方法</strong>。下面我们就来深入 toInvokers() 方法内部，看看其具体的转换逻辑：</p>
<pre class="lang-java" data-nodeid="426974"><code data-language="java"><span class="hljs-keyword">private</span> Map&lt;String, Invoker&lt;T&gt;&gt; toInvokers(List&lt;URL&gt; urls) {
&nbsp; &nbsp; ... <span class="hljs-comment">// urls集合为空时，直接返回空Map</span>
&nbsp; &nbsp; Set&lt;String&gt; keys = <span class="hljs-keyword">new</span> HashSet&lt;&gt;();
&nbsp; &nbsp; String queryProtocols = <span class="hljs-keyword">this</span>.queryMap.get(PROTOCOL_KEY); <span class="hljs-comment">// 获取Consumer端支持的协议，即protocol参数指定的协议</span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (URL providerUrl : urls) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (queryProtocols != <span class="hljs-keyword">null</span> &amp;&amp; queryProtocols.length() &gt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> accept = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String[] acceptProtocols = queryProtocols.split(<span class="hljs-string">","</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (String acceptProtocol : acceptProtocols) { <span class="hljs-comment">// 遍历所有Consumer端支持的协议</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (providerUrl.getProtocol().equals(acceptProtocol)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; accept = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!accept) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>; <span class="hljs-comment">// 如果当前URL不支持Consumer端的协议，也就无法执行后续转换成Invoker的逻辑</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (EMPTY_PROTOCOL.equals(providerUrl.getProtocol())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>; <span class="hljs-comment">// 跳过empty协议的URL</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果Consumer端不支持该URL的协议（这里通过SPI方式检测是否有对应的Protocol扩展实现），也会跳过该URL</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!ExtensionLoader.getExtensionLoader(Protocol.class).hasExtension(providerUrl.getProtocol())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.error("...");
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; // 合并URL参数，这个合并过程，在本课时后面展开介绍
&nbsp; &nbsp; &nbsp; &nbsp; URL url = mergeUrl(providerUrl);
&nbsp; &nbsp; &nbsp; &nbsp; // 获取完整URL对应的字符串，也就是在urlInvokerMap集合中的key
&nbsp; &nbsp; &nbsp; &nbsp; String key = url.toFullString();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (keys.contains(key)) { // 跳过重复的URL
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; keys.add(key); // 记录key
&nbsp; &nbsp; &nbsp; &nbsp; // 匹配urlInvokerMap缓存中的Invoker对象，如果命中缓存，直接将Invoker添加到newUrlInvokerMap这个新集合中即可；
&nbsp; &nbsp; &nbsp; &nbsp; // 如果未命中缓存，则创建新的Invoker对象，然后添加到newUrlInvokerMap这个新集合中
&nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, Invoker&lt;T&gt;&gt; localUrlInvokerMap = <span class="hljs-keyword">this</span>.urlInvokerMap;
&nbsp; &nbsp; &nbsp; &nbsp; Invoker&lt;T&gt; invoker = localUrlInvokerMap == <span class="hljs-keyword">null</span> ? <span class="hljs-keyword">null</span> : localUrlInvokerMap.get(key);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invoker == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> enabled = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (url.hasParameter(DISABLED_KEY)) { // 检测URL中的disable和enable参数，决定是否能够创建Invoker对象
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; enabled = !url.getParameter(DISABLED_KEY, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; enabled = url.getParameter(ENABLED_KEY, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (enabled) { <span class="hljs-comment">// 这里通过Protocol.refer()方法创建对应的Invoker对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invoker = <span class="hljs-keyword">new</span> InvokerDelegate&lt;&gt;(protocol.refer(serviceType, url), url, providerUrl);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.error(<span class="hljs-string">"Failed to refer invoker for interface:"</span> + serviceType + <span class="hljs-string">",url:("</span> + url + <span class="hljs-string">")"</span> + t.getMessage(), t);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invoker != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 将key和Invoker对象之间的映射关系记录到newUrlInvokerMap中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; newUrlInvokerMap.put(key, invoker);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {<span class="hljs-comment">// 缓存命中，直接将urlInvokerMap中的Invoker转移到newUrlInvokerMap即可</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; newUrlInvokerMap.put(key, invoker);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; keys.clear();
&nbsp; &nbsp; <span class="hljs-keyword">return</span> newUrlInvokerMap;
}
</code></pre>
<p data-nodeid="426975"><strong data-nodeid="427154">toInvokers() 方法的代码虽然有点长，但核心逻辑就是调用 Protocol.refer() 方法创建 Invoker 对象，其他的逻辑都是在判断是否调用该方法。</strong></p>
<p data-nodeid="426976">在 toInvokers() 方法内部，我们可以看到<strong data-nodeid="427160">调用了 mergeUrl() 方法对 URL 参数进行合并</strong>。在 mergeUrl() 方法中，会将注册中心中 configurators 目录下的 URL（override 协议），以及服务治理控制台动态添加的配置与 Provider URL 进行合并，即覆盖 Provider URL 原有的一些信息，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426977"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> URL <span class="hljs-title">mergeUrl</span><span class="hljs-params">(URL providerUrl)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 首先，移除Provider URL中只在Provider端生效的属性，例如，threadname、threadpool、corethreads、threads、queues等参数。</span>
&nbsp; &nbsp; <span class="hljs-comment">// 然后，用Consumer端的配置覆盖Provider URL的相应配置，其中，version、group、methods、timestamp等参数以Provider端的配置优先</span>
&nbsp; &nbsp; <span class="hljs-comment">// 最后，合并Provider端和Consumer端配置的Filter以及Listener</span>
&nbsp; &nbsp; providerUrl = ClusterUtils.mergeUrl(providerUrl, queryMap); 
&nbsp; &nbsp; <span class="hljs-comment">// 合并configurators类型的URL，configurators类型的URL又分为三类：</span>
&nbsp; &nbsp; <span class="hljs-comment">// 第一类是注册中心Configurators目录下新增的URL(override协议)</span>
&nbsp; &nbsp; <span class="hljs-comment">// 第二类是通过ConsumerConfigurationListener监听器(监听应用级别的配置)得到的动态配置</span>
&nbsp; &nbsp; <span class="hljs-comment">// 第三类是通过ReferenceConfigurationListener监听器(监听服务级别的配置)得到的动态配置</span>
&nbsp; &nbsp; <span class="hljs-comment">// 这里只需要先了解：除了注册中心的configurators目录下有配置信息之外，还有可以在服务治理控制台动态添加配置，</span>
&nbsp; &nbsp; <span class="hljs-comment">// ConsumerConfigurationListener、ReferenceConfigurationListener监听器就是用来监听服务治理控制台的动态配置的</span>
&nbsp; &nbsp; <span class="hljs-comment">// 至于服务治理控制台的具体使用，在后面详细介绍</span>
&nbsp; &nbsp; providerUrl = overrideWithConfigurator(providerUrl);
&nbsp; &nbsp; <span class="hljs-comment">// 增加check=false，即只有在调用时，才检查Provider是否可用</span>
&nbsp; &nbsp; providerUrl = providerUrl.addParameter(Constants.CHECK_KEY, String.valueOf(<span class="hljs-keyword">false</span>));
&nbsp; &nbsp; <span class="hljs-comment">// 重新复制overrideDirectoryUrl，providerUrl在经过第一步参数合并后（包含override协议覆盖后的属性）赋值给overrideDirectoryUrl。</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.overrideDirectoryUrl = <span class="hljs-keyword">this</span>.overrideDirectoryUrl.addParametersIfAbsent(providerUrl.getParameters()); 
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略对Dubbo低版本的兼容处理逻辑</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> providerUrl;
}
</code></pre>
<p data-nodeid="426978">完成 URL 到 Invoker 对象的转换（toInvokers() 方法）之后，其实在 refreshInvoker() 方法的最后，还会<strong data-nodeid="427166">根据 multiGroup 的配置决定是否调用 toMergeInvokerList() 方法将每个 group 中的 Invoker 合并成一个 Invoker</strong>。下面我们一起来看 toMergeInvokerList() 方法的具体实现：</p>
<pre class="lang-java" data-nodeid="426979"><code data-language="java"><span class="hljs-keyword">private</span> List&lt;Invoker&lt;T&gt;&gt; toMergeInvokerList(List&lt;Invoker&lt;T&gt;&gt; invokers) {
&nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; mergedInvokers = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; Map&lt;String, List&lt;Invoker&lt;T&gt;&gt;&gt; groupMap = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Invoker&lt;T&gt; invoker : invokers) { <span class="hljs-comment">// 按照group将Invoker分组</span>
&nbsp; &nbsp; &nbsp; &nbsp; String group = invoker.getUrl().getParameter(GROUP_KEY, <span class="hljs-string">""</span>);
&nbsp; &nbsp; &nbsp; &nbsp; groupMap.computeIfAbsent(group, k -&gt; <span class="hljs-keyword">new</span> ArrayList&lt;&gt;());
&nbsp; &nbsp; &nbsp; &nbsp; groupMap.get(group).add(invoker);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (groupMap.size() == <span class="hljs-number">1</span>) { <span class="hljs-comment">// 如果只有一个group，则直接使用该group分组对应的Invoker集合作为mergedInvokers</span>
&nbsp; &nbsp; &nbsp; &nbsp; mergedInvokers.addAll(groupMap.values().iterator().next());
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (groupMap.size() &gt; <span class="hljs-number">1</span>) { <span class="hljs-comment">// 将每个group对应的Invoker集合合并成一个Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (List&lt;Invoker&lt;T&gt;&gt; groupList : groupMap.values()) {
            <span class="hljs-comment">// 这里使用到StaticDirectory以及Cluster合并每个group中的Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; StaticDirectory&lt;T&gt; staticDirectory = <span class="hljs-keyword">new</span> StaticDirectory&lt;&gt;(groupList);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; staticDirectory.buildRouterChain();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mergedInvokers.add(CLUSTER.join(staticDirectory));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; mergedInvokers = invokers;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> mergedInvokers;
}
</code></pre>
<p data-nodeid="426980">这里使用到了 Cluster 接口的相关功能，我们在后面课时还会继续深入分析 Cluster 接口及其实现，你现在可以将 Cluster 理解为一个黑盒，知道其 join() 方法会将多个 Invoker 对象转换成一个 Invoker 对象即可。</p>
<p data-nodeid="426981">到此为止，RegistryDirectory 处理一次完整的动态 Provider 发现流程就介绍完了。</p>
<p data-nodeid="426982">最后，我们再分析下<strong data-nodeid="427174">RegistryDirectory 中另外一个核心方法—— doList() 方法</strong>，该方法是 AbstractDirectory 留给其子类实现的一个方法，也是通过 Directory 接口获取 Invoker 集合的核心所在，具体实现如下：</p>
<pre class="lang-java" data-nodeid="426983"><code data-language="java"><span class="hljs-keyword">public</span> List&lt;Invoker&lt;T&gt;&gt; doList(Invocation invocation) {
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (forbidden) { <span class="hljs-comment">// 检测forbidden字段，当该字段在refreshInvoker()过程中设置为true时，表示无Provider可用，直接抛出异常</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RpcException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (multiGroup) {
        <span class="hljs-comment">// multiGroup为true时的特殊处理，在refreshInvoker()方法中针对multiGroup为true的场景，已经使用Router进行了筛选，所以这里直接返回接口</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.invokers == <span class="hljs-keyword">null</span> ? Collections.emptyList() : <span class="hljs-keyword">this</span>.invokers;
&nbsp; &nbsp; }
&nbsp; &nbsp; List&lt;Invoker&lt;T&gt;&gt; invokers = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-comment">// 通过RouterChain.route()方法筛选Invoker集合，最终得到符合路由条件的Invoker集合</span>
&nbsp; &nbsp; invokers = routerChain.route(getConsumerUrl(), invocation);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invokers == <span class="hljs-keyword">null</span> ? Collections.emptyList() : invokers;
}
</code></pre>
<h3 data-nodeid="426984">总结</h3>
<p data-nodeid="426985">在本课时，我们首先介绍了 dubbo-cluster 模块的整体架构，简单说明了 Cluster、Directory、Router、LoadBalance 四个核心接口的功能。接下来我们就深入介绍了 Directory 接口的定义以及 StaticDirectory、RegistryDirectory 两个类的核心实现，其中 RegistryDirectory 涉及动态查找 Provider URL 以及处理动态配置的相关逻辑，显得略微复杂了一点，希望你能耐心学习和理解。关于这部分内容，你若有不懂或不理解的地方，也欢迎你留言和我交流。</p>
<p data-nodeid="426986" class="">在下一课时，我们会介绍路由机制相关的内容。</p>

---

### 精选评论

##### *良：
> RegistryDirectory最新版本继承了DynamicDirectory, 将很多属性移到DynamicDirectory中初始化和其他方法实现了

