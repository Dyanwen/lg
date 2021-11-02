<p data-nodeid="685" class="">我们知道在微服务架构中，可以根据需要在服务提供者和消费者之间架设 API 网关，来满足需要服务路由和控制的各种场景。在前面两个课时中，我们分析了 Zuul 网关的使用方式和实现原理。今天我们继续介绍另一款 API 网关，即 Spring Cloud Gateway。</p>
<h3 data-nodeid="686">Spring Cloud Gateway 简介</h3>
<p data-nodeid="687">Spring Cloud Gateway 是 Spring 官方自己开发的一款 API 网关。在具体展开介绍 Spring Cloud Gateway 之前，我们有必要对它和 Netflix Zuul 做一个对比。通过上一课时的分析，我们知道 Zuul 的实现原理是对 Servlet 的一层封装，通信模式上采用的是阻塞式 I/O。而在技术体系上，Spring Cloud Gateway 基于最新的 Spring 5 和 Spring Boot 2，以及用于响应式编程的 Project Reactor 框架，提供的是响应式、非阻塞式 I/O 模型。所以较之 Netflix Zuul，<strong data-nodeid="739">性能上</strong>Spring Cloud Gateway 显然要更胜一筹。</p>
<p data-nodeid="688">另一方面，<strong data-nodeid="745">从功能上</strong>，Spring Cloud Gateway 也比 Zuul 更为丰富。除了通用的服务路由机制之外，Spring Cloud Gateway 还支持请求限流等面向服务容错方面的功能，同样也能与 Hystrix 等框架进行良好的集成。</p>
<p data-nodeid="689">讲到这里，你可能会觉得既然有了性能和功能都更高一筹的 Spring Cloud Gateway，那我们为什么还要介绍 Zuul 呢？这是因为 Spring Cloud Gateway 的源码非常复杂，出现问题不容易排查和解决。而 Zuul 的编程模型和底层原理都非常简单，开发调试上也容易把握。</p>
<p data-nodeid="690">要想在微服务架构中引入 Spring Cloud Gateway，我们同样需要构建一个独立的 Spring Boot 应用程序，并在 Maven 中添加如下依赖项：</p>
<pre class="lang-xml" data-nodeid="691"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-gateway<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="692">按照约定，我们把这个独立的微服务命名为 gateway-server，然后在作为 Bootstrap 类的 GatewayApplication 上添加 @EnableDiscoveryClient 注解即可，如下所示：</p>
<pre class="lang-java" data-nodeid="693"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableDiscoveryClient</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">GatewayApplication</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(GatewayApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<h3 data-nodeid="694">Spring Cloud Gateway 与服务路由</h3>
<p data-nodeid="695">在引入 Spring Cloud Gateway 之后，我们先重点讨论一下它作为 API 网关的核心功能，即服务路由。但在此之前，我们同样要先对 Spring Cloud Gateway 的基本架构进行一个初步的了解。</p>
<h4 data-nodeid="696">Spring Cloud Gateway 基本架构</h4>
<p data-nodeid="697">Spring Cloud Gateway 中的核心概念有两个，一个是过<strong data-nodeid="761">滤器（Filter）</strong>，一个是<strong data-nodeid="762">谓词（Predicate）</strong>。Spring Cloud Gateway 的整体架构图如下图所示：</p>
<p data-nodeid="698"><img src="https://s0.lgstatic.com/i/image/M00/64/30/Ciqc1F-X3WqATvJfAAGAafgQhSE126.png" alt="Lark20201027-164157.png" data-nodeid="765"></p>
<div data-nodeid="699"><p style="text-align:center">Spring Cloud Gateway 基本架构图</p></div>
<p data-nodeid="700">Spring Cloud Gateway 中的过滤器和 Zuul 中的过滤器是同一个概念。它们都可以用于在处理 HTTP 请求之前或之后修改请求本身，及对应响应结果。区别在于两者的类型和实现方式不同。Spring Cloud Gateway 的种类非常丰富，我们在今天的后续内容中会有专门主题对其进行详细的展开。</p>
<p data-nodeid="701">而所谓谓词，本质上是一种判断条件，用于将 HTTP 请求与路由进行匹配。Spring Cloud Gateway 内置了大量的谓词组件，可以分别对 HTTP 请求的消息头、请求路径等常见的路由媒介进行自动匹配以便决定路由结果。这里，我们无意对所有谓词一一展开，你可以参考官方文档做进一步学习。</p>
<p data-nodeid="702">事实上，除了指定服务的名称和目标服务地址之外，使用 Spring Cloud Gateway 最主要的开发工作就是配置谓词和过滤器规则，让我们来试一下吧。</p>
<h4 data-nodeid="703">使用 Spring Cloud Gateway 实现路由</h4>
<p data-nodeid="704">与 Zuul 一样，我们同样通过配置项来设置 Spring Cloud Gateway 对 HTTP 请求的路由行为。但与 Zuul 不同，默认情况下，Spring Cloud Gateway 并不支持与服务发现机制之间的自动集成。所以，为了启用该功能，我们需要在配置文件中添加如下配置项：</p>
<pre class="lang-xml" data-nodeid="705"><code data-language="xml">spring:
  cloud:
	gateway:
&nbsp;     discovery:
&nbsp;&nbsp;      locator:
&nbsp;&nbsp;&nbsp;&nbsp;      enabled: true
</code></pre>
<p data-nodeid="706">然后，我们来看一下一条完整路由配置的基本结构，如下所示。</p>
<pre class="lang-xml" data-nodeid="707"><code data-language="xml">spring:
  cloud:
	gateway:
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;routes:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - id: testroute
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; uri: lb://testservice
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; predicates:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Path=/test/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - PrefixPath=/prefix
</code></pre>
<p data-nodeid="708">在上述配置中，有几个需要注意的点。首先我们使用 id 配置项指定了这条<strong data-nodeid="808">路由信息的编号</strong>，这个例子中的“testroute”就起了这个作用。而 uri 配置项中的“lb”代表<strong data-nodeid="809">负载均衡</strong>LoadBalance，也就是说在访问 url 指定的服务名称时需要<strong data-nodeid="810">集成负载均衡机制</strong>。请注意“lb”配置项中所指定的服务名称同样需要与保存在 Eureka 中的服务名称<strong data-nodeid="811">完全一致</strong>。然后我们使用了<strong data-nodeid="812">谓词</strong>来对请求路径进行<strong data-nodeid="813">匹配</strong>，这里的“Path=/test/<strong data-nodeid="814">”代表所有以“/test”开头的请求都将被路由到这条路径中。最后我们还定义了一个</strong>过滤器**，这个过滤器的作用是<strong data-nodeid="815">为路径添加前缀</strong>（Prefix），这样当请求“/test/”时，最后转发到目标服务的路径将会变为“/prefix/test/”。</p>
<p data-nodeid="709">让我们回到 SpringHealth 案例系统，Spring Cloud Gateway 网关服务中完整版的配置信息如下所示：</p>
<pre class="lang-xml" data-nodeid="710"><code data-language="xml">server:
&nbsp; port: 5555

eureka:
&nbsp; instance:
&nbsp;&nbsp;&nbsp; preferIpAddress: true
&nbsp; client:
&nbsp;&nbsp;&nbsp; registerWithEureka: true
&nbsp;&nbsp;&nbsp; fetchRegistry: true
&nbsp;&nbsp;&nbsp; serviceUrl:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; defaultZone: http://localhost:8761/eureka/

spring:
&nbsp; cloud:
&nbsp;&nbsp;&nbsp; gateway:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; discovery:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; locator:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enabled: true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; routes:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - id: userroute
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; uri: lb://userservice
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; predicates:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Path=/user/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - RewritePath=/user/(?<span class="hljs-tag">&lt;<span class="hljs-name">path</span>&gt;</span>.*), /$\{path}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - id: deviceroute
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; uri: lb://deviceservice
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; predicates:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Path=/device/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - RewritePath=/device/(?<span class="hljs-tag">&lt;<span class="hljs-name">path</span>&gt;</span>.*), /$\{path}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - id: interventionroute
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; uri: lb://interventionservice
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; predicates:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - Path=/intervention/**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; - RewritePath=/intervention/(?<span class="hljs-tag">&lt;<span class="hljs-name">path</span>&gt;</span>.*), /$\{path}
</code></pre>
<p data-nodeid="711">尽管到目前为止，我们还没有完成三个独立的业务服务的完整构建。但从设计和规划上讲，可以先对服务路由进行配置。在上述配置中，我们设置了 Eureka 服务的地址并启用了服务发现机制，然后根据 Eureka 保存的服务名称和地址定义了三条路由规则：userroute、deviceroute 和 interventionroute 分别对应 user-service、device-service 和 intervention-service 这三个微服务。这里，我们也通过在各个服务名称前面加上“lb://”来实现客户端负载均衡。</p>
<p data-nodeid="712">同时，我们同样对请求路径设置了谓词，并添加了一个对请求路径进行重写（Rewrite）的<strong data-nodeid="823">过滤器</strong>。通常，每个微服务自身通过根路径“/”来暴露服务。基于以上配置，通过 Spring Cloud Gateway 暴露它们时，则分别在路径上添加了“/user”“/device”和“/intervention”前缀。这种重写过滤器的效果实际上和前面介绍的前缀过滤器有相同的效果。</p>
<p data-nodeid="713">以上配置项比较常见，我们可以参考并搭建满足自身需求的网关服务。然后，在 Spring Cloud Gateway 的整个功能体系中，还有很多值得我们去挖掘的地方。与 Zuul 一样，Spring Cloud Gateway 的<strong data-nodeid="833">扩展性</strong>也主要体现在<strong data-nodeid="834">过滤器组件</strong>中。</p>
<h3 data-nodeid="714">剖析 Spring Cloud Gateway 中的过滤器</h3>
<p data-nodeid="715">针对过滤器，Spring Cloud Gateway 提供了一个全局过滤器（GlobalFilter）的概念。这个概念的应用对象是路由本身。如果过滤器只针对某一个路由生效，那它就是一个普通的过滤器。而那些对所有路由都生效的过滤器就是全局过滤器。Spring Cloud Gateway 内置了一大批过滤器，我们同样无意对它们一一展开，每个过滤器在官方文档中都有详细的描述。我这里举几个常见的过滤器使用方法。</p>
<p data-nodeid="716">我们首先想到了可以使用全局过滤器来对所有 HTTP 请求进行拦截，具体做法是实现 GlobalFilter 接口，示例代码如下所示。</p>
<pre class="lang-java te-preview-highlight" data-nodeid="855"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JWTAuthFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">GlobalFilter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Void&gt; <span class="hljs-title">filter</span><span class="hljs-params">(ServerWebExchange exchange, GatewayFilterChain chain)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ServerHttpRequest.Builder builder = exchange.getRequest().mutate();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; builder.header(<span class="hljs-string">"Authorization"</span>,<span class="hljs-string">"JWTToken"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> chain.filter(exchange.mutate().request(builder.build()).build());
&nbsp;&nbsp;&nbsp; }
}
</code></pre>

<p data-nodeid="718">以上代码展示了如何利用全局过滤器在所有的请求中添加 Header 的实现方法。在这个示例中，我们给所有经过 API 网关的 HTTP 请求添加了一个消息头，用来设置与 JWT Token 相关的安全认证信息。关于微服务的安全性和 JWT 我们会在《服务认证：如何使用JWT实现定制化Token？》课时进行详细展开。</p>
<p data-nodeid="719">注意到这里的 filter 方法返回了一个 Mono 对象。你可能会问这个 Mono 对象究竟是什么呢？事实上，这是在响应式编程框架 Project Reactor 中代表单个返回值的流式对象。响应式编程是一个复杂的话题，在我们整个 Spring 全家桶课程中会有专题进行介绍，现在你只需要掌握如何使用常见的 API 来构建全局过滤器的方法以及效果。</p>
<p data-nodeid="720">另一方面，我们回想在上一课时中提到的，Zuul 有 pre、route、post 和 error 这四种类型的过滤器，分别对应一个 HTTP 请求的不同生命周期。在这点上，Spring Cloud Gateway 与 Zuul 在设计思想上是一致的，它也提供了可用于 pre 和 post 两种阶段的过滤器。很多时候，我们需要根据场景来构建针对这两个阶段的自定义过滤器。</p>
<p data-nodeid="721">以下代码展示了一个 PostGatewayFilter 的实现方式。我们首先继承一个 AbstractGatewayFilterFactory 类，然后可以通过覆写 apply 方法来提供针对 ServerHttpResponse 对象的任何操作：</p>
<pre class="lang-java" data-nodeid="722"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PostGatewayFilterFactory</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractGatewayFilterFactory</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">PostGatewayFilterFactory</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(Config.class);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> GatewayFilter <span class="hljs-title">apply</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> apply(o -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> GatewayFilter <span class="hljs-title">apply</span><span class="hljs-params">(Config config)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (exchange, chain) -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> chain.filter(exchange).then(Mono.fromRunnable(() -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ServerHttpResponse response = exchange.getResponse();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//针对Response的各种处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Config</span> </span>{
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="723">PreGatewayFilter 的实现方式也类似，只不过处理的目标一般是 ServerHttpRequest 对象。</p>
<p data-nodeid="724">相比 Zuul，<strong data-nodeid="848">请求限流</strong>是 Spring Cloud Gateway 的一项特色功能。为此， Spring Cloud Gateway 中专门存在一个请求限流过滤器 RequestRateLimiter。在今天内容的最后，我们也对这个特殊的过滤器做一些展开。</p>
<p data-nodeid="725">所谓限流，一般的做法是衡量请求处理的速率并对其进行控制。因此，RequestRateLimiter 抽象了两个参数来完成这一目标。其中第一个参数是 replenishRate，该参数用于指定允许用户每秒处理的请求数。而第二个参数是 burstCapacity，它被用来设置一秒钟内允许的最大请求数。如果我们把请求看成是往一个桶里倒水，那么 replenishRate 参数用于控制水流的速度，而 burstCapacity 用于控制桶的大小。请求限流过滤器的完整配置示例如下所示：</p>
<pre class="lang-xml" data-nodeid="726"><code data-language="xml">spring:
  cloud:
    gateway:
	&nbsp;&nbsp;&nbsp;routes:
	&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;- id: requestratelimiterroute
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; uri: lb://interventionservice
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;filters:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;- name: RequestRateLimiter
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;args:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; &nbsp;&nbsp;redis-rate-limiter.replenishRate: 50
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;redis-rate-limiter.burstCapacity: 100
</code></pre>
<p data-nodeid="727">请求限流过滤器在实现上依赖 Redis，所以需要引入 spring-boot-starter-data-redis-reactive 这个支持响应式 Redis 的依赖。然后我们针对访问 ntervention-service 的场景，基于 Redis 分别设置 replenishRate 和 burstCapacity 值为 50 和 100。你可以在日常开发过程中尝试去调整这些参数。</p>
<h3 data-nodeid="728">小结与预告</h3>
<p data-nodeid="729">今天的内容围绕 Spring Cloud Gateway 展开，这是我们课程中介绍的第二个 API 网关实现方案。与 Zuul 相比，Spring Cloud Gateway 在提供高性能的同时也丰富了作为 API 网关的核心功能。在本课时中，我们重点对 Spring Cloud Gateway 中的基本架构、服务路由以及过滤器机制进行了详细的探讨。</p>
<p data-nodeid="730">这里给你留一道思考题：在 Spring Cloud Gateway中，如果想要给每个请求路径添加一个前缀有哪些实现方法？</p>
<p data-nodeid="731" class="">从下一课时开始，我们将进入新主题的探讨，即如何在微服务架构中引入服务容错的设计思想和实现技术，而在具体应用过程中，服务容错与 API 网关也有紧密的关联。</p>

---

### 精选评论

##### **3759：
> 在微服务中服务之间的调用Token是如何传递的，还有当前登录用户的信息如何获取，每个服务中自己去解析token获取当前用户信息还是还有其他更好的方案，还请老师指教。

##### **明：
> 这也太简单了吧……不能顺便讲讲flux嘛，丝毫没有掌握

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课时还是面向大众，通过一些简单的案例介绍完整的技术体系。关于响应式编程方面的内容不在这个课程里，后面拉勾这边会有响应式编程方面的专栏，可以关注一下

##### **潮：
> spring-cloud-starter-netflix-eureka-server 好像和spring-cloud-starter-gateway冲突。用spring-cloud-starter-netflix-eureka-client就没冲突了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; eureka和gateway应该是不冲突的，确认一下版本，可以再试试

##### **文：
> 想问一下网关前为什么要配一个nginx？我有种 网关和nginx功能重合的错觉，期待老师能展开讲讲😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; nginx更多是请求转发，没有业务处理功能的，一般用作前置的负载均衡

##### rainwolf：
> spring cloud gateway前边需要再加个nginx作为网关吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，一般都会前置Nginx

