<p data-nodeid="3349" class="">今天我们开始讨论 Spring Cloud 中的另一个核心技术组件，<strong data-nodeid="3415">API 网关</strong>。</p>
<p data-nodeid="3350">我们先来简单介绍 API 网关的基本结构，然后给出 Spring Cloud 中关于 API 网关的解决方案。今天的内容重点是介绍如何使用 Zuul 这一特定工具来构建 API 网关的实现过程。</p>
<h3 data-nodeid="3351">什么是 API 网关？</h3>
<p data-nodeid="3352">在微服务架构中，API 网关（也叫服务网关）的出现有其必然性。通常，单个微服务提供的 API 粒度与客户端请求的粒度不一定完全匹配。多个服务之间通过对细粒度 API 的聚合才能满足客户端的要求。更为重要的是，网关能够起到客户端与微服务之间的隔离作用。随着业务需求的变化和时间的演进，网关背后的各个微服务的划分和实现可能需要做相应的调整和升级。这种调整和升级应该实现对客户端透明，如下所示：</p>
<p data-nodeid="3353"><img src="https://s0.lgstatic.com/i/image/M00/61/17/CgqCHl-Os0qAGZryAABHtvSGSm8333.png" alt="Lark20201020-175149.png" data-nodeid="3421"></p>
<div data-nodeid="3354"><p style="text-align:center">API 网关的聚合和隔离作用示意图</p></div>
<p data-nodeid="3355">当然，如果我们在服务调用过程中添加了一个网关层，那么所有的客户端都通过这个统一的网关接入微服务。这样一些非业务功能性需求就可以在网关层进行集中处理。这些需求中，比较常见的包括<strong data-nodeid="3447">请求监控</strong>、<strong data-nodeid="3448">安全管理</strong>、<strong data-nodeid="3449">路由规则</strong>、<strong data-nodeid="3450">日志记录</strong>、<strong data-nodeid="3451">访问控制</strong>、<strong data-nodeid="3452">服务适配</strong>等功能：</p>
<p data-nodeid="3356"><img src="https://s0.lgstatic.com/i/image/M00/61/0C/Ciqc1F-Os1SAHbuIAABCdN3hS7M839.png" alt="Lark20201020-175156.png" data-nodeid="3455"></p>
<div data-nodeid="3357"><p style="text-align:center">服务网关的处理机制</p></div>
<p data-nodeid="3358">在 Spring Cloud 中，针对 API 网关的实现提供了两种解决方案。一种是集成 Netflix 中的 Zuul 网关，一种是自研 Spring Cloud Gateway。接下来，我们先来讨论如何使用 Zuul 构建 API 网关。</p>
<h3 data-nodeid="3359">如何使用 Zuul 构建 API 网关？</h3>
<p data-nodeid="3360">我们在《案例驱动：如何通过实战案例来学习 Spring Cloud 框架？》中提到，与其他微服务一样，服务网关本身同样是一种服务，也是一个标准的 Spring Boot 应用程序。为了构建 Zuul 服务器，我们创建一下新的 Maven 工程 zuul-server，并引入 spring-cloud-starter-netflix-zuul 依赖，如下所示。</p>
<pre class="lang-xml" data-nodeid="3361"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-netflix-zuul<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="3362">然后我们创建 Bootstrap 类，代码如下：</p>
<pre class="lang-java" data-nodeid="3363"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableZuulProxy</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ZuulServerApplication</span> </span>{
	&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(ZuulServerApplication.class, args);
&nbsp;&nbsp;&nbsp; &nbsp;}
}
</code></pre>
<p data-nodeid="3364">这里引入了一个新的注解 @EnableZuulProxy，嵌入该注解的 Bootstrap 类将自动成为 Zuul 服务器的入口。@EnableZuulProxy 注解非常强大，我们基于该注解就可以使用 Zuul 中的各种内置过滤器实现复杂的服务路由，关于 Zuul 提供的过滤器以及实现原理我们将在下一课时中进行详细介绍。</p>
<p data-nodeid="3365">API 网关的定位和功能势必涉及服务的发现和调用，所以 API 网关与注册中心关系密切。以Eureka 为代表的注册中心为服务路由提供了服务定义信息，这是能够实现服务路由的基础。为了与 Eureka 进行交互，在 zuul-server 中，我们需要向 application.yml 配置文件中添加对 Eureka 的集成。配置内容如下所示，关于各个配置项的内容我们已经在介绍 Eureka 时做了详细介绍，这里不再赘述。</p>
<pre class="lang-java" data-nodeid="3366"><code data-language="java">server:
  port: <span class="hljs-number">5555</span>
	&nbsp;
eureka:
&nbsp; instance:
&nbsp;&nbsp;&nbsp; preferIpAddress: <span class="hljs-keyword">true</span>
&nbsp; client:
&nbsp;&nbsp;&nbsp; registerWithEureka: <span class="hljs-keyword">true</span>
&nbsp;&nbsp;&nbsp; fetchRegistry: <span class="hljs-keyword">true</span>
&nbsp;&nbsp;&nbsp; serviceUrl:
	&nbsp;&nbsp; defaultZone: http:<span class="hljs-comment">//localhost:8761/eureka/</span>
</code></pre>
<p data-nodeid="3367">另一方面，API 网关在其服务的消费者和提供者之间提供了一层反向代理，充当着前置负载均衡器的角色。所以，API 网关的定位决定了 Zuul 需要依赖 Ribbon。而 Zuul 可以与 Ribbon 完成无缝集成，我们在后续内容中会看到相关的示例。</p>
<h3 data-nodeid="3368">如何使用 Zuul 实现服务路由？</h3>
<p data-nodeid="3369">对于 API 网关而言，最重要的功能就是服务路由，即通过 Zuul 访问的请求会路由并转发到对应的后端服务中。通过 Zuul 进行服务访问的 URL 通用格式如下所示：</p>
<pre class="lang-xml" data-nodeid="3370"><code data-language="xml">http://zuulservice:5555/service
</code></pre>
<p data-nodeid="3371">其中 zuulservice 代表 Zuul 服务器的地址。而这里的 service 所对应的后端服务的确定就需要依赖位于 Zuul 中的服务路由信息。在 Zuul 中，服务路由信息的设置可以使用以下几种常见做法，让我们一一来展开讨论。</p>
<h4 data-nodeid="3372">基于服务发现映射服务路由</h4>
<p data-nodeid="3373">Zuul 可以基于注册中心的服务发现机制实现自动化服务路由功能。所以使用 Zuul 实现服务路由最常见的、也最推荐的做法就是利用这种自动化的路由映射关系来确定路由信息。</p>
<p data-nodeid="3374">从开发角度讲，系统自动映射也最简单。我们不需要做任何事情，因为在 Eureka 中已经保存了各种服务定义信息。而服务定义信息中包含了各个服务的名称，所以 Zuul 就可以把这些服务的名称与目标服务进行自动匹配。匹配的规则就是直接将目标服务映射到服务名称上。</p>
<p data-nodeid="3375">例如，我们在 user-service 的配置文件中通过以下配置项指定了该服务的名称为 userservice：</p>
<pre class="lang-java" data-nodeid="3376"><code data-language="java">spring:
&nbsp; &nbsp;application:
	&nbsp; &nbsp;name: userservice
</code></pre>
<p data-nodeid="3377" class="">这样我们就可以通过<a href="http://zuulservice:5555/userservice" data-nodeid="3473">http://zuulservice:5555/userservice</a>访问到该服务，注意该 URL 中的目标服务就是 userservice，与服务定义中的名称保持一致。</p>
<p data-nodeid="3378">在 Zuul 启动的过程中，会从 Eureka 中获取当前所有已注册的服务信息，然后自动生成服务名称与目标服务之间的映射关系。现在，让我们来演示一下。我们先后启动 eureka-server、user-service 和 zuul-service，然后访问以下地址：</p>
<pre class="lang-xml" data-nodeid="3896"><code data-language="xml">http://localhost:5555/actuator/routes”
</code></pre>


<p data-nodeid="3380">这时候可以看到如下所示的键值对信息，这些键值对信息实际上就是服务路由映射信息：</p>
<pre class="lang-xml" data-nodeid="3381"><code data-language="xml">{
	&nbsp; "/userservice/**":"userservice"
}
</code></pre>
<p data-nodeid="4639" class=""><a href="http://localhost:5555/actuator/routes%E2%80%9D" data-nodeid="4642">http://localhost:5555/actuator/routes”</a> 是 Zuul 提供的服务路由端点，展示了目前在 Zuul 中配置的服务路由信息。其中"/userservice/**"中后半部分的“**”代表所有访问该路径以及子路径的请求都将被自动路由到注册在 Eureka 中的名为 userservice 的某一个服务实例中。</p>


<p data-nodeid="3383">当采用这种系统自动映射方式时，如果我们注册一个新服务或下线某个已有服务，那么这份映射列表也会做相应的调整。整个过程对于开发人员完全透明可见。</p>
<h4 data-nodeid="3384">基于动态配置映射服务路由</h4>
<p data-nodeid="3385">基于服务发现机制的系统自动映射非常方便，但也有明显的局限性。在日常开发过程中，我们往往对服务映射关系有更多的定制化需求，比方说不使用默认的服务名称来命名目标服务，或者在各个请求路径之前加一个统一的前缀（Prefix）等。Zuul 充分考虑到了这些需求，开发和运维人员可以通过配置实现服务路由的灵活映射。</p>
<p data-nodeid="3386">首先我们可以在 zuul-server 工程下的 application.yml 配置文件中为 user-service 配置特定的服务名称与请求地址之间的映射关系，如下所示：</p>
<pre class="lang-xml" data-nodeid="3387"><code data-language="xml">zuul:
&nbsp;&nbsp; routes:
&nbsp;&nbsp;&nbsp; &nbsp;userservice: /user/**
</code></pre>
<p data-nodeid="3388">注意，这里我们使用 /user 来为 user-service 指定请求根地址。现在我们访问 <a href="http://zuulservice:5555/user/" data-nodeid="3497">http://zuulservice:5555/user/</a> 时，就相当于将请求发送给了 Eureka 中的 userservice 实例。</p>
<p data-nodeid="5387" class="">现在我们重启 zuul-server 并访问 <a href="http://localhost:5555/actuator/routes%E2%80%9D" data-nodeid="5391">http://localhost:5555/actuator/routes”</a> 端点，得到的服务路由映射关系就变成了如下结果：</p>


<pre class="lang-xml" data-nodeid="3390"><code data-language="xml">{
	 &nbsp; "/user/**":"userservice",
	&nbsp; "/userservice/**":"userservice"
}
</code></pre>
<p data-nodeid="3391">可以看到在原有路由信息的基础上，Zuul 生成了一条新的路由信息，对应配置文件中的配置。现在，相当于将访问某一个服务的入口变成了两个：一个是系统自动映射的路由，一个是通过配置所生成的路由。当我们不希望系统自动映射的路由被外部使用时，我们就可以通过 ignored-services 配置项把它们从服务路由中去掉。再次以 userservice 为例，ignored-services 配置项的使用方法如下所示。</p>
<pre class="lang-xml" data-nodeid="3392"><code data-language="xml">zuul:
&nbsp;&nbsp; routes:
&nbsp;&nbsp;&nbsp;  ignored-services: 'userservice'
&nbsp;&nbsp;&nbsp; &nbsp;userservice: /user/**
</code></pre>
<p data-nodeid="3393">让我们考虑另外一个比较常见的应用场景，在一个大型的微服务架构中，可能会有非常多的微服务。这就需要对这些服务进行全局性的规划，可以通过模块或子系统的方式进行管理。表现在路由信息上，在各个服务请求地址上添加一个前缀用来标识模块和子系统是一项最佳实践。针对这种场景，就可以用到 Zuul 提供的“prefix”配置项，示例如下所示：</p>
<pre class="lang-xml" data-nodeid="3394"><code data-language="xml">zuul:
	prefix:&nbsp; /springhealth
&nbsp;&nbsp; &nbsp;routes:
&nbsp;&nbsp;&nbsp; &nbsp; ignored-services: 'userservice'
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;userservice: /user/**
</code></pre>
<p data-nodeid="6127" class="te-preview-highlight">我们将“prefix”配置项设置为“/springhealth”，代表所有配置的路由请求地址之前都会自动添加“/springhealth”前缀，用来识别这些请求的都属于 springhealth 微服务系统。现在访问 <a href="http://localhost:5555/actuator/routes%E2%80%9D" data-nodeid="6131">http://localhost:5555/actuator/routes”</a> 端点，可以看到所配置的前缀已经生效，如下所示：</p>


<pre class="lang-xml" data-nodeid="3396"><code data-language="xml">{
	 &nbsp;&nbsp; "/springhealth/user/**":"userservice" 
}
</code></pre>
<h4 data-nodeid="3397">基于静态配置映射服务路由</h4>
<p data-nodeid="3398">在绝大多数场景下，通过注册中心映射服务路由是可以支持日常的开发工作的。这也是我们通常推荐的实现方法。但是 Zuul 还提供了不依赖于 Eureka 的服务路由方式，这种方式可以让 Zuul 具备更多的扩展性。这种方式的实现，使我们可以使用自定义的路由规则完成与其他各种第三方系统的集成。</p>
<p data-nodeid="3399">让我们考虑这样一个场景，如果系统中存在一个第三方服务，该服务无法注册到我们的 Eureka 注册中心，还会暴露了一个 HTTP 端点供 SpringHealth 系统进行调用。我们知道 Spring Cloud 中各个服务之间都是通过轻量级的 HTTP 协议进行交互的。而 HTTP 协议具备技术平台无关性，只要能够获取服务的 HTTP 端口地址，原则上我们就可以进行远程访问，而不用关注该服务在实现上采用了何种技术和工具。因此，针对这一场景，我们可以在配置文件中添加如下的静态路由配置，这样访问“/thirdpartyservice/**”时，Zuul 会将该请求转发到外部的第三方服务<a href="http://thirdparty.com/thirdpartyservice" data-nodeid="3519">http://thirdparty.com/thirdpartyservice</a>中。</p>
<pre class="lang-xml" data-nodeid="3400"><code data-language="xml">zuul:
&nbsp; routes:
&nbsp;&nbsp;&nbsp; thirdpartyservice:
&nbsp;&nbsp;&nbsp; &nbsp; path: /thirdpartyservice/**
	 &nbsp;url: http://thirdparty.com/thirdpartyservice
</code></pre>
<p data-nodeid="3401">现在的服务路由信息就会变成如下结果：</p>
<pre class="lang-xml" data-nodeid="3402"><code data-language="xml">{
	 &nbsp;&nbsp; "/springhealth/thirdpartyservice/**":"http://thirdparty.com/thirdpartyservice",
	 &nbsp;&nbsp; "/springhealth/user/**":"userservice"
}
</code></pre>
<p data-nodeid="3403">在上文中介绍 Zuul 的定位时，我们提到 Zuul 能够与 Ribbon 进行整合。而这种整合也来自手工设置静态服务路由的方式，具体实现方式如下所示：</p>
<pre class="lang-xml" data-nodeid="3404"><code data-language="xml">zuul:
&nbsp; routes:
&nbsp;&nbsp;&nbsp; thirdpartyservice:
&nbsp;&nbsp;&nbsp; &nbsp; path: /thirdpartyservice/**
&nbsp;&nbsp;&nbsp; &nbsp; serviceId: thirdpartyservice

ribbon:
&nbsp; eureka:
&nbsp;&nbsp;&nbsp; enabled: false
&nbsp;
thirdpartyservice:
&nbsp; ribbon:
	listOfServers: http://thirdpartyservice1:8080,http://thirdpartyservice2:8080
</code></pre>
<p data-nodeid="3405">这里，我们配置了一个 thirdpartyservice 路由信息，通过“/ thirdpartyservice /**”映射到 serviceId 为“thirdpartyservice”的服务中。然后我们希望通过自定义 Ribbon 的方式来实现客户端负载均衡，这时候就需要关闭 Ribbon 与 Eureka 之间的关联。可以通过“ribbon.eureka.enabled: false”配置项完成这一目标。在不使用 Eureka 的情况下，我们需要手工指定 Ribbon 的服务列表。“users.ribbon.listOfServers”配置项为我们提供了这方面的支持，如在上面的示例中“http://thirdpartyservice1:8080，http://thirdpartyservice2:8080”就为 Ribbon 提供了两个服务定义作为实现负载均衡的服务列表。</p>
<h3 data-nodeid="3406">小结与预告</h3>
<p data-nodeid="3407">本课时讨论 API 网关，我们基于 Netflix Zuul 构建了 API 网关。在微服务架构中，使用 API 网关的核心作用是实现服务访问的路由，而通过 Zuul 实现服务路由的方式有很多，我们分别从基于服务发现的自动映射、基于动态的配置映射以及基于静态的配置映射这三个维度展开了讨论并给出了相关的配置示例。</p>
<p data-nodeid="3408">这里给你留一道思考题：我们通过 Zuul 来实现服务路由，可以采用哪些具体的配置方式？</p>
<p data-nodeid="3409" class="">Zuul 是一款经典的开源API网关，使用上非常简单，而它的内部实现机制也比较容易理解。下一课时，我们将通过源码来深入剖析 Zuul 的实现原理。</p>

---

### 精选评论

##### **星：
> actuator/routes默认配置是看不到结果，需要配置一下 management.endpoints.web.exposure.include: '*'

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，默认情况下应该是不开放的，需要添加配置项，这个算是基本的做法，课程里就不专门介绍了

##### **星：
> ignored-services: 'userservice' 这个配置好像也有点问题，应该是这样配的 ignored-services: '/userservice/**'

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是有效的，用来设置忽略的服务名称

##### *鹏：
> 请教老师：zuul主要用于服务转发和通用功能（Token校验、流量监控、黑白名单等）实现，是一个流量网关；那前面提到的，底层各领域服务提供的接口粒度一般满足不了客户端需求，需要由网关来进行服务调用整合及数据聚合成VO，这里数据聚合写在哪里呢？（1）写在Zuul里面，好像行不通？流量经过zuul的Filter之后，就被转发到其它领域服务中了，没有机会做数据聚合（2）写在各领域服务里面，那领域服务就包含对客户端业务逻辑，不内聚，复用性差了？（3）再加一层业务网关（SpringBoot服务），nginx-各领域服务集合，增加一层，性能就有损耗了？你们是怎么做的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一般在ZUUL之后加一个聚合服务，类似于你说的第三种方案

##### *军：
> 老师，我把您在github上的的springcloud-demo工程下载下来，起来后仍然访问不到zuul端点路由，其他朋友们可以访问到吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里描述上是有问题，地址应该是：http://localhost:5555/actuator/routes，不是http://localhost:5555/routes。重新更新一下代码，然后用http://localhost:5555/actuator/routes试试

##### *军：
> 老师，你查看zuul的路由信息时好像没加actuator,http://localhost:5055/routes而我这样的这样访问直接报404了，我的springboot是2.0.9,springcloud是Finchley,老师您是哪个版本了?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我Spring Cloud用的是Hoxton.SR1，Spring Boot是2.2.4.RELEASE，Zuul跟actuator应该没啥关系，只要Zuul能起来应该就是访问/routes端口的，你再确认一下端口看看

