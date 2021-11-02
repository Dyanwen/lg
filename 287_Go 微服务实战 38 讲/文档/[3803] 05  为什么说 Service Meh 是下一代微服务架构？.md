<p data-nodeid="67805">在前面第 2 课时我们介绍过，Service Mesh（服务网格） 是云原生的代表技术之一，并且在后面的组件案例实践中，Service Mesh 也是其中的“主角”，因此我们非常有必要重点介绍下 Service Mesh 的诞生背景、相关特性以及三种常见的开源组件。</p>
<h3 data-nodeid="67806">Service Mesh 背后的诉求</h3>
<p data-nodeid="67807">一种技术的出现必然是有各种推动的因素，Service Mesh 也一样，它的出现就得益于微服务架构的发展。那 Service Mesh 出现时，其背后的诉求是什么呢？</p>
<h4 data-nodeid="68961" class="">1. 微服务架构的复杂性</h4>

<p data-nodeid="69841">在微服务架构中，应用系统往往被拆分成很多个微服务（可以多达成百上千），数量庞大的微服务实例使得服务治理具有一定的挑战，比如说常见的服务注册、服务发现、服务实例的负载均衡，以及为了保护服务器实现熔断、重试等基础功能。除此之外，应用程序中还加上了大量的非功能性代码。</p>
<p data-nodeid="69842" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/9C/CgqCHl8OniqAaOpTAABLWy0eR68344.png" alt="Drawing 0.png" data-nodeid="69846"><br>
微服务架构的基础功能</p>




<p data-nodeid="70738" class="">归根结底，在微服务架构中，<strong data-nodeid="70744">微服务组件复杂、上手门槛比较高</strong>成了痛点问题。业务开发团队需要一定的学习周期才能上手微服务架构的开发，而人力资源的昂贵以及人员的流动性使得开发成本变高。业务开发团队更加擅长的是某一具体领域的业务，而不是技术的深度。应用系统的核心价值在于实现相应的业务，所以对于业务开发人员来说，微服务仅仅是手段，不是最终的目标。我们需要对业务开发人员“屏蔽”微服务的基础组件，使得微服务之间的通信对于业务开发人员透明。</p>


<p data-nodeid="67815">为应对这个问题，有一些实践是利用 API 网关接收请求，网关作为代理处理外部服务的请求，并提供服务注册与发现、负载均衡、日志监控、容错等功能。然而，这种方案也存在不足，比如网关的单点故障、系统架构变得异常庞大；从功能来看，API 网关主要是面向用户，也就是说它可以解决从用户到各个后端服务的流量问题，至于其他问题，它可能就无能为力了。而我们需要的是一个完整的贯穿整个请求周期的方案，或者至少是一些能够与 API 网关互补的方案和工具。</p>
<h4 data-nodeid="71617" class="">2. 微服务本身的挑战</h4>


<p data-nodeid="67819">微服务还有其<strong data-nodeid="67911">自身引入的复杂度</strong>，有比学习微服务框架更艰巨的挑战，如微服务的划分、设计良好的声明式 API、单体旧应用的迁移，还涉及跨多个服务的数据一致性，这都会令大部分团队疲于应付。</p>
<p data-nodeid="67820">除此之外，<strong data-nodeid="67917">版本兼容性</strong>也是一个挑战。微服务框架很难一开始就完美无缺，在现实的软件工程中一般不存在这样完美无缺的框架，功能会分为多个里程碑迭代，发布之后就会有补丁修复……没有任何问题，这只是一种理想状态。业务服务中引入微服务的基础组件，这样业务服务的代码和微服务的 SDK 强耦合在一起，导致业务升级和微服务 SDK 的升级强绑定在了一起。如果客户端 SDK 和服务器端版本不一致，那就得谨慎对待客户端与服务端的兼容性问题。版本兼容性的处理非常复杂，特别是在服务端和客户端数量庞大的情况下，每对客户端和服务端的版本都有可能不同，这对于兼容性测试也会造成很大的压力。同时，对于异构的系统，还需要开发多语言的 SDK，维护成本很高。</p>
<h4 data-nodeid="72483" class="">3. 本质诉求</h4>


<p data-nodeid="67824">接下来我们探讨下业务服务最关心的是什么，比如写一个商品服务，对商品做增删改查的操作，你会发现基础设施、跨语言、兼容性和商品服务本身并没关系，而服务间的通讯才是最需要解决的问题。</p>
<p data-nodeid="67825">比如，为了保证将客户端发出的业务请求发去一个正确的地方，需要用什么样的负载均衡？要不要做灰度？最终这些解决方案，都是让请求去访问正确的后端服务。整个过程当中，这个请求是从来不发生更改的。</p>
<p data-nodeid="67826">既然在开发微服务的时候不用特别关心服务的通讯层，那是不是可以把微服务的技术栈向下移呢？</p>
<p data-nodeid="67827">微服务的早期先驱，如 Netflix、Twitter 等大型互联网公司，它们通过建立内部库的方式处理这些问题，然后提供给所有服务使用。但这种方法的问题在于这些库相对来说是比较“脆弱”的，很难保证它们可以适应所有的技术堆栈选择，且很难把库扩展到成百上千个微服务中。</p>
<p data-nodeid="67828">为了应对上述的问题，Service Mesh 出现了，&nbsp;Service Mesh 通过<strong data-nodeid="67928">独立进程</strong>的方式隔离微服务基础组件，对这个独立进程升级、运维要比传统的微服务方式简单得多。</p>
<h3 data-nodeid="67829">什么是 Service Mesh</h3>
<p data-nodeid="75500" class=""><strong data-nodeid="75509">Service Mesh（服务网格）</strong>，最早在 2016 年 9 月，由开发 Linkerd 的 Buoyant 公司提出。2017 年，Linkerd 加入 CNCF，由 CNCF 托管孵化，Linkerd 是第一个加入 CNCF 的 Service Mesh 项目。Service Mesh 开始变得流行起来，特别是在技术社区，有人指出 <strong data-nodeid="75510">Service Mesh 会是下一代的微服务架构基础</strong>。</p>







<p data-nodeid="67831">关于 Service Mesh 的定义，目前比较认同的是 Buoyant 的 CEO William Morgan 在博客中给出的定义：</p>
<blockquote data-nodeid="67832">
<p data-nodeid="67833">Service Mesh 是用于处理服务到服务通信的专用基础架构层。云原生有着复杂的服务拓扑，它负责可靠的传递请求。实际上，Service Mesh 通常是作为一组轻量级网络代理实现，这些代理与应用程序代码部署在一起，应用程序无感知。</p>
</blockquote>
<p data-nodeid="67834">Service Mesh 模式的核心在于<strong data-nodeid="67961">将客户端 SDK 剥离，以 Proxy 独立进程运行</strong>，目标是将原来存在于 SDK 中的各种能力下沉，为应用减负，以帮助应用云原生化。</p>
<p data-nodeid="67835">Service Mesh 的第一代产品，如 Linkerd 1 和 Envoy，天然支持虚拟机。随着云原生的崛起，到了 Istio 和 Linkerd 2 ，不支持虚拟机。相比虚拟机，Kubernetes 提供了太多便利。 绝大部分Service Mesh 的实现都支持 Kubernetes，有些实现甚至只支持 Kubernetes。就这样，Service Mesh 逐步发展为一个独立的基础设施层。</p>
<p data-nodeid="78304" class="">在云原生架构下，应用系统可能由数百个微服务组成，微服务一般又是多实例部署，并且每一个实例都可能处于不断变化的状态，因为它们是由 Kubernetes 之类的资源调度系统动态调度。 Kubernetes 中的 Service Mesh 实现模式被命名为 <strong data-nodeid="78310">Sidecar</strong>（边车模式，因为类似连接到摩托车的边车）。</p>

<p data-nodeid="77892" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/92/Ciqc1F8OnrqAWCmyAAWNZlJGJJQ859.png" alt="Drawing 1.png" data-nodeid="77901"><br>
边车</p>








<p data-nodeid="67839">在模式库中，Sidecar 模式的定义是：将应用程序的组件部署到单独的进程或容器中以提供隔离和封装。这种模式还可以使应用程序由异构组件和技术组成。</p>
<p data-nodeid="78905">在 Sidecar 模式中，“边车”与父应用程序（即业务服务）是两个独立的进程，二者生命周期相同，同时被创建和退出。“边车”附加到业务服务，并为应用提供支持功能，如微服务架构中的基本通信。Service Mesh 一般的架构如下图所示：</p>
<p data-nodeid="78906" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/92/Ciqc1F8OnsOAa3MVAABY1memBaA509.png" alt="Drawing 2.png" data-nodeid="78910"><br>
Service Mesh 架构图</p>




<p data-nodeid="67843">从上图可以看到，业务所有的流量都转发到 Service Mesh 的代理服务 Sidecar 中，Sidecar 承担了微服务框架基础的功能，包括服务注册与发现、负载均衡、熔断限流、认证鉴权、日志、监控和缓存加速等。不同的是，Service Mesh 强调的是通过独立进程的代理方式。总体来说，Service Mesh 帮助应用程序在复杂的软件架构和网络中建立稳定的通信机制。</p>
<h3 data-nodeid="67844">Service Mesh 的开源组件</h3>
<p data-nodeid="67845">近几年 Service Mesh 社区比较活跃，其对应的开源组件也很丰富，从最早的 Linkerd 到当前火热的 Istio、Envoy 等组件，下面我们就来重点介绍下这三个开源组件。</p>
<h4 data-nodeid="79313" class="">1. Istio</h4>

<p data-nodeid="67849">Istio 由 Google、IBM 和 Lyft 合作开源，所以 Istio 自诞生之日起就备受瞩目。在 Istio 中，直接使用了 Lyft 公司的 Envoy 作为 Sidecar。2017 年 5 月 Istio 发布了 0.1 版本，现在已经发展到 1.6 版本。Istio 是 Service Mesh 的第二代产品，在刚开始发布时还曾计划提供对非 Kubernetes 的支持，发展到现在基本只支持 Kubernetes 上的使用，实质性取消了对虚拟机的支持。</p>
<p data-nodeid="67850">Istio 功能十分丰富，包括：</p>
<ul data-nodeid="67851">
<li data-nodeid="67852">
<p data-nodeid="67853"><strong data-nodeid="67988">流量管理</strong>：Istio 的基本功能，Istio 的流量路由规则使得你可以轻松控制服务之间的流量和 API 调用。</p>
</li>
<li data-nodeid="67854">
<p data-nodeid="67855"><strong data-nodeid="67993">策略控制</strong>：应用策略并确保其得到执行，并且资源在消费者之间公平分配。</p>
</li>
<li data-nodeid="67856">
<p data-nodeid="67857"><strong data-nodeid="67998">可观测性</strong>：通过自动链路追踪、监控和服务的日志，可以全面了解受监视服务如何与其他服务以及 Istio 组件本身进行交互。</p>
</li>
<li data-nodeid="67858">
<p data-nodeid="67859"><strong data-nodeid="68003">安全认证</strong>：通过托管的身份验证，授权和服务之间通信的加密自动保护服务。Istio Security 提供了全面的安全解决方案来解决这些问题。</p>
</li>
</ul>
<p data-nodeid="80101">Istio 专为可扩展性而设计，可满足多种部署需求。它通过拦截和配置 Mesh 网络流量来做到这一点，架构图如下所示：</p>
<p data-nodeid="80102" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/92/Ciqc1F8OntCAXtNNAARr5zliZpw986.png" alt="Drawing 3.png" data-nodeid="80106"><br>
Istio 架构图</p>




<p data-nodeid="67863">Istio 针对现有的服务网络，提供了一种简单的方式将连接、安全、控制和观测的模块，与应用程序或服务隔离开来，从而使开发人员可以将更多的精力放在核心的业务逻辑上。另外，Istio 直接基于成熟的 Envoy 代理进行构建，控制面组件则都是使用 Go 编写，在不侵入应用程序代码的前提下实现可视性与控制能力。总之，Istio 的设计理念是非常新颖前卫的。</p>
<h4 data-nodeid="80905" class="">2. Linkerd</h4>


<p data-nodeid="67867">2016 年 1 月，前 Twitter 工程师 William Morgan 和 Oliver Gould 组建了一个名为 Buoyant 的公司，同时在 GitHub 上发布了 Linkerd 0.0.7 版本。Linkerd 由 Buoyant 推出，使用 Scala 语言实现，<strong data-nodeid="68016">是业界第一个 Service Mesh</strong>。2017 年 1 月，Linkerd 加入 CNCF； 4 月，发布了 1.0 版本。</p>
<p data-nodeid="67868">Linkerd 的架构由两部分组成：数据平面和控制平面。其中，<strong data-nodeid="68026">数据平面</strong>由轻量级代理组成，它们作为 Sidecar 容器与服务代码的每个实例一起部署；<strong data-nodeid="68027">控制平面</strong>是一组在专用 Kubernetes 命名空间中运行的服务（默认情况下）。这些服务承担聚合遥测数据、提供面向用户的 API、向数据平面代理提供控制数据等功能，它们共同驱动着数据平面的行为。</p>
<p data-nodeid="67869">Linkerd 作为 Service Mesh 的先驱开源组件，在生产环境得到了大规模使用。Linkerd 2 的定位是 Kubernetes 的 Service Mesh，其提供了运行时调试、可观察性、可靠性和安全性，使得运行服务变得更容易、更安全，而无须更改代码。但是随着 Istio 的诞生，前景并不是特别乐观。</p>
<h4 data-nodeid="81695" class="te-preview-highlight">3. Envoy</h4>


<p data-nodeid="67873">2016 年 9 月，Lyft 公司开源 Envoy ，并在 GitHub 上发布了 1.0.0 版本。Envoy 由 C++ 实现，性能和资源消耗上表现优秀。2017 年 9 月，Envoy 加入 CNCF，成为继 Linkerd 之后的第二个 Service Mesh 项目。Envoy 发展平稳，被 Istio 收编之后，Envoy 将自身定义为数据平面，并希望使用者可以通过控制平面来为 Envoy 提供动态配置。Envoy 用于云原生应用，为应用服务提供高性能分布式代理，以及作为大规模微服务架构的 Service Mesh 通信总线和通用数据平面。</p>
<h3 data-nodeid="67874">小结</h3>
<p data-nodeid="67875">数量众多的微服务，使得系统复杂程度和管理难度增加。虽然微服务架构能够解决服务调用、熔断和监控等问题，但这些框架和开源组件基本都具有侵入性，需要在业务服务中引入服务治理组件。</p>
<p data-nodeid="67876">因此，对于微服务规模较大、内部服务存在异构的业务场景，非常推荐你使用 Service Mesh，不需要对服务做特殊的改造，所有业务之外的功能都由 Service Mesh 去做了。Service Mesh 是微服务架构的升级，它使得业务开发者更加关注自身业务服务的实现，微服务的基础设施则由 Service Mesh 来负责，业务开发团队得以回归业务。</p>
<p data-nodeid="67877">但 Service Mesh 不是银弹，那学完本课时，在理解 Service Mesh 架构模式的基础上，你知道 Service Mesh 有哪些局限性吗？欢迎你在留言区分享你的想法。</p>

---

### 精选评论

##### **铖：
> 期待后面的实战，任何理论知识都不如实战写上几行代码

##### **富：
> 为啥我想到了spring的aop

##### **健：
> 期待实战

##### *景：
> 老师 讲的好

##### **2674：
> 老师确实讲的不错，继续学习

##### *杰：
> 我认为局限性一是在service mesh引用后带来的运维负担和心智负担，二是环境中引入额外组件可能带来本地调试和问题溯源的困难。

##### **峰：
> 微服务看完感觉这是一场变革

