<p data-nodeid="122690" class="">之前的课程我分享了微服务在生产实践中的一些经验教训，但我也强调微服务架构是一种风格，需要具体的技术来支撑，而在 Java 领域微服务是如何实现的？本讲我将通过下面三个方面来阐述：</p>
<ul data-nodeid="122691">
<li data-nodeid="122692">
<p data-nodeid="122693">通用的微服务架构应包含哪些组件；</p>
</li>
<li data-nodeid="122694">
<p data-nodeid="122695">Spring Cloud 是如何支撑微服务架构的；</p>
</li>
<li data-nodeid="122696">
<p data-nodeid="122697">我为什么强烈推荐 Spring Cloud Alibaba？</p>
</li>
</ul>
<h3 data-nodeid="122698">通用的微服务架构应包含哪些组件</h3>
<p data-nodeid="122699">相对于单体式架构的简单粗暴，微服务架构将应用打散，形成多个微服务进行独立开发、测试、部署与运维。虽然从管理与逻辑上更符合业务需要，但微服务架构也带来了诸多急需解决的核心问题：</p>
<ol data-nodeid="122700">
<li data-nodeid="122701">
<p data-nodeid="122702">如何发现新服务节点以及检查服务节点的状态？</p>
</li>
<li data-nodeid="122703">
<p data-nodeid="122704">如何发现服务及负载均衡如何实现？</p>
</li>
<li data-nodeid="122705">
<p data-nodeid="122706">服务间如何进行消息通信？</p>
</li>
<li data-nodeid="122707">
<p data-nodeid="122708">如何对使用者暴露服务 API？</p>
</li>
<li data-nodeid="122709">
<p data-nodeid="122710">如何集中管理众多服务节点的配置文件？</p>
</li>
<li data-nodeid="122711">
<p data-nodeid="122712">如何收集服务节点的日志并统一管理？</p>
</li>
<li data-nodeid="122713">
<p data-nodeid="122714">如何实现服务间调用链路追踪？</p>
</li>
<li data-nodeid="122715">
<p data-nodeid="122716">如何对系统进行链路保护，避免微服务雪崩？</p>
</li>
</ol>
<p data-nodeid="122717">可以发现，上述这些问题并不是针对某种语言或某种技术的，任何软件厂商要构建微服务架构就必须面对这些问题，要么独立开发要么将已有多种技术整合形成整体解决方案。好在经过多年沉淀，业内已经有了标准答案，下图清晰的说明微服务架构需要的标准组件。</p>
<p data-nodeid="123127" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB7dmAMXFxAAFTzOxiKrU853.png" alt="图片1.png" data-nodeid="123131"></p>
<div data-nodeid="123128"><p style="text-align:center">微服务架构标准组件</p></div>


<p data-nodeid="122720">下面我来介绍每种组件的职责。</p>
<ul data-nodeid="122721">
<li data-nodeid="122722" class="">
<p data-nodeid="122723"><strong data-nodeid="122941">注册中心（Service Registry）</strong>：注册中心是微服务架构最核心的组件。它起到新服务节点的注册与状态维护的作用，通过注册中心解决了上述问题 1。微服务节点在启动时会将自身的服务名称、IP、端口等信息在注册中心中进行登记，注册中心会定时检查该节点的运行状态。注册中心通常会采用心跳机制最大程度保证其持有的服务节点列表都是可用的。</p>
</li>
</ul>
<p data-nodeid="124006" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3C/CioPOWBB7eqAPUlxAAEh8rtRqRA639.png" alt="图片2.png" data-nodeid="124010"></p>
<div data-nodeid="124007"><p style="text-align:center">服务注册流程</p></div>


<ul data-nodeid="122726">
<li data-nodeid="122727">
<p data-nodeid="122728"><strong data-nodeid="122949">负载均衡（Load Balance）</strong>：负载均衡器解决了问题 2。通常在微服务彼此调用时并不是直接通过IP、端口直接访问，而是首先通过服务名在注册中心查询该服务拥有哪些可用节点，然后注册中心将可用节点列表返回给服务调用者，这个过程称为“服务发现”。因服务高可用的要求，服务调用者会接收到多个可用节点，必须要从中进行选择，因此在服务调用者一端必须内置负载均衡器，通过负载均衡策略选择适合的节点发起实质的通信请求。</p>
</li>
</ul>
<p data-nodeid="125764" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB7fyAA0CzAAEc4ADEq64041.png" alt="图片3.png" data-nodeid="125768"></p>
<div data-nodeid="125765"><p style="text-align:center">服务发现与负载均衡</p></div>




<ul data-nodeid="122731">
<li data-nodeid="122732">
<p data-nodeid="122733"><strong data-nodeid="122957">服务通信（Communication）</strong>：服务通信组件解决了问题 3。在微服务定义中阐述服务间通信采用轻量级协议，通常是 HTTP RESTful 风格。但因 RESTful 风格过于灵活，必须加以约束，通常在应用时对其进行上层封装，例如在 Spring Cloud 中就提供了 Feign 和 RestTemplate 两种技术屏蔽底层实现 RESTful 通信细节，所有开发者是基于封装后统一的SDK进行开发，这有利于团队间协作。</p>
</li>
<li data-nodeid="122734">
<p data-nodeid="122735"><strong data-nodeid="122962">API 服务网关（API Gateway）</strong>：服务网关解决问题 4。对于最终用户来说，微服务的通信与各种实现细节应该是透明的，用户只需关注他要使用的 API 接口即可。因此微服务架构引入服务网关控制用户的访问权限。服务网关是外部环境访问内部微服务的唯一途径，在这个基础上还可以扩展出其他功能，例如：用户认证与授权、容错限流、动态路由、A/B测试、灰度发布等。</p>
</li>
</ul>
<p data-nodeid="126643" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB7gWAY_a8AAD5jgY2hwc651.png" alt="图片4.png" data-nodeid="126647"></p>
<div data-nodeid="126644"><p style="text-align:center">API 服务网关</p></div>


<ul data-nodeid="122738">
<li data-nodeid="122739">
<p data-nodeid="122740"><strong data-nodeid="122970">配置中心（Config Management）</strong>：配置中心解决问题 5。微服务架构下，微服务节点都包含自己的各种配置文件，如JDBC地址、自定义配置、环境配置等。要知道互联网公司微服务节点可能是成千上万个，如果这些配置信息分散存储在节点上，如发生配置变更就必须逐个调整，这必将给运维人员带来巨大的工作量。配置中心便因此而生，通过部署配置中心服务器，将原本分散的配置文件从应用中剥离，集中转存到配置中心。一般配置中心会提供 UI 界面，可以方便快捷的实现大规模集群的配置调整。</p>
</li>
</ul>
<p data-nodeid="127522" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB7gyAbCHMAAHJx5a01gM755.png" alt="图片5.png" data-nodeid="127526"></p>
<div data-nodeid="127523" class=""><p style="text-align:center">庞杂而重复的配置文件</p></div>


<p data-nodeid="128401" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB7haAI07QAAE5Ncx4NU0920.png" alt="图片6.png" data-nodeid="128405"></p>
<div data-nodeid="128402"><p style="text-align:center">引入配置中心集中管理配置</p></div>


<ul data-nodeid="122745">
<li data-nodeid="122746">
<p data-nodeid="122747"><strong data-nodeid="122981">集中式日志管理（Centralized Logging）</strong>：集中式日志解决问题 6。因为微服务架构默认将应用日志分散保存在每一个微服务节点上，当系统进行用户行为分析、数据统计时必须收集所有节点日志数据。那如何有效收集所有节点的运行日志，并对其进行分析汇总呢？业内常见的方案有 ELK、EFK，通过搭建独立的日志收集系统，定时抓取增量日志形成有效的统计报表，为决策提供数据支撑。</p>
</li>
</ul>
<p data-nodeid="129280" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB7iCAfs2OAANoxddK8vE764.png" alt="图片7.png" data-nodeid="129284"></p>
<div data-nodeid="129281"><p style="text-align:center">ELK 日志手机系统</p></div>


<ul data-nodeid="122750">
<li data-nodeid="122751">
<p data-nodeid="122752"><strong data-nodeid="122989">分布式链路追踪（Distributed Tracing）</strong>：分布式追踪解决问题 7。一个复杂的业务流程可能需要连续调用多个微服务，我们需要记录一个完整业务逻辑涉及的每一个微服务的运行状态，再通过可视化链路图展现，帮助软件工程师在系统出错时分析解决问题。</p>
</li>
</ul>
<p data-nodeid="130159" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB7iiALA62AAEAs99EyBw000.png" alt="图片8.png" data-nodeid="130163"></p>
<div data-nodeid="130160"><p style="text-align:center">微服务链路追踪</p></div>


<p data-nodeid="131038" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB7jCAQ71FAAQc7R46oTA480.png" alt="图片9.png" data-nodeid="131042"></p>
<div data-nodeid="131039"><p style="text-align:center">Zipkin 链路图</p></div>


<ul data-nodeid="122757">
<li data-nodeid="122758">
<p data-nodeid="122759"><strong data-nodeid="123000">服务保护（Service Protection）</strong>：服务保护解决问题 8。在服务间通信过程中，如果某个微服务出现响应高延迟可能会导致线程池满载，严重时会引起系统崩溃。这里就需要引入服务保护组件实现高延迟服务的快速降级，避免系统崩溃。</p>
</li>
</ul>
<p data-nodeid="122760">以上是在微服务架构过程中派生的新问题以及对应的解决方案，这些组件是微服务架构设计时必须考虑的事情。在实现层面，Java 是目前对微服务生态支持最好的编程语言，下面我们就来介绍在Java中如何实现微服务架构的。</p>
<h3 data-nodeid="122761">Spring Cloud 是如何支撑微服务架构的</h3>
<p data-nodeid="122762">在 Java 领域可以说 Spring 无人不知无人不晓，我们现代的企业级应用与互联网应用，绝大多数都是构建在 Spring 这个生态体系上。同样的，Spring Cloud 也是在 Spring 基础上生根发芽的。</p>
<p data-nodeid="122763" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3A/Cgp9HWBB6vmAdPriAAGnjWuGXnw750.png" alt="Drawing 9.png" data-nodeid="123006"></p>
<div data-nodeid="122764"><p style="text-align:center">Spring Cloud 微服务介绍</p></div>
<p data-nodeid="122765">下面这张图很好地诠释了 Spring、Spring Boot、Spring Cloud 它们之间的关系。</p>
<p data-nodeid="122766">Spring 是整个微服务开发生态的基石，在此基础上 Spring 经过封装和简化，形成了Spring Boot 敏捷开发框架，而 Spring Cloud 开发微服务则必须掌握 Spring Boot，如果此时你对Spring 以及 Spring Boot 还不了解，可以在拉勾教育找到相关课程补充学习。</p>
<p data-nodeid="122767">Spring Cloud 提供了完整的微服务架构的技术生态，像刚才我们提到的微服务架构标准组件，在 Spring Cloud 中都有着对应的实现。</p>
<p data-nodeid="122768"><img src="https://s0.lgstatic.com/i/image6/M00/13/37/CioPOWBB6wWAOi_lAAD1IRlSCEE059.png" alt="Drawing 10.png" data-nodeid="123012"></p>
<div data-nodeid="122769"><p style="text-align:center">Spring Cloud 内置组件</p></div>
<p data-nodeid="122770">不过这里有一点需要注意，Spring Cloud 并不是完全由 Spring 机构自主开发的，秉持不重复造轮子的理念，Spring 机构整合了多家厂商的优秀开源产品，比如注册中心就选用了 Netflix 的 Eureka，在此基础上进行整合形成了完整的 Java 微服务架构解决方案。刚才我们列举出来的 8 个微服务的标准组件在 Spring Cloud 中都有对应的具体实现，值得注意的是同一类组件 Spring Cloud 可能给出多种不同的开源产品供我们选择，这些产品在 Spring Cloud 下是彼此兼容的。</p>
<p data-nodeid="131917" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB7kWAR6FnAAZL3b1XI2A758.png" alt="图片13.png" data-nodeid="131920"></p>

<p data-nodeid="122827">以上是 Spring Cloud 的主要组成部分，这里还有许多没有提到的功能组件，如果你对此感兴趣，可以到 Spring Cloud 官网（<a href="https://spring.io/projects/spring-cloud" data-nodeid="123057">https://spring.io/projects/spring-cloud</a>）了解更详细的内容。</p>
<p data-nodeid="122828">仔细看上面的表格，你会发现其中问题，正是因为 Spring Cloud 是集众家所长，但这也受制于第三方厂商的约束。以 Zuul 为例，Netflix 宣布停止维护，Spring 机构便不得不寻找替代品或者自行研发。与此同时，Spring Cloud 作为国外产品引入国内后也出现了水土不服，例如 Spring Cloud Config 默认将配置文件转存在Github仓库，但国内开发商并不喜欢这种做法，他们更希望将这些配置信息存储到自己的数据库中。因此，阿里巴巴在原有 Spring Cloud 基础上，结合阿里巴巴多年的开源技术沉淀，设计了更符合国情的 Spring Cloud Alibaba。</p>
<h3 data-nodeid="122829">功能更强大的国产微服务生态Spring Cloud Alibaba</h3>
<p data-nodeid="122830">Spring Cloud Alibaba是直接隶属于 Spring Cloud 的子项目。官网是：<a href="https://spring.io/projects/spring-cloud-alibaba#overview" data-nodeid="123064">https://spring.io/projects/spring-cloud-alibaba#overview</a></p>
<p data-nodeid="122831">Spring Cloud Alibaba是国产的微服务开发一站式解决方案，与原有 Spring Cloud 兼容的同时对微服务生态进行扩展，通过添加少量的配置注解，便可实现更符合国情的微服务架构。</p>
<p data-nodeid="122832"><img src="https://s0.lgstatic.com/i/image6/M00/13/37/CioPOWBB6xKAKCJlAAeEtzZ79H0773.png" alt="Drawing 11.png" data-nodeid="123068"></p>
<p data-nodeid="122833">下面是 Spring Cloud 与 Spring Cloud Alibaba 的对比表格。</p>
<p data-nodeid="132611" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB7mGAEJaBAAfPJ9GO8jI984.png" alt="图片14.png" data-nodeid="132614"></p>

<p data-nodeid="122913">我将两者从不同维度进行比对，可以发现相比 Spring Cloud，Spring Cloud Alibaba 对服务注册、配置中心与负载均衡功能都整合进 Nacos，这样简化了微服务架构的复杂度，出问题的概率也会降低。原有的服务保护组件也调整为 Sentinel，相较Hystrix功能更强大，使用也更加友好。在表格最下方也可看到 Spring Cloud Alibaba 基于阿里云强大的能力提供了更多的新特性，很多复杂的应用场景通过 Spring Cloud Alibaba 结合阿里云便可轻松实现。在后面的内容，我们以这个表格作为主线对组件进行逐一讲解，让你轻松掌握各组件用法与设计原理。</p>
<h3 data-nodeid="122914">小结与预告</h3>
<p data-nodeid="122915">本节我们讲解了微服务架构应包含哪些标准组件，同时介绍了 Java 的 Spring Cloud 解决方案，最后我们通过技术栈比对，使你对 Spring Cloud Alibaba 解决方案有了整体的认知。</p>
<p data-nodeid="122916">这里给你留一道思考题：如果公司领导让你带领团队开发三方支付平台，峰值 TPS 过万，请你列出这个系统架构将首要解决的三个最主要的重难点，欢迎你把思考后的结果发布在评论中。</p>
<p data-nodeid="122917" class="">在接下来的课程中，我们将接触到 Spring Cloud Alibaba 体系最重要的核心组件 Nacos，了解 Nacos 的使用方法与设计原理。</p>

---

### 精选评论

##### **峰：
> 事务一致性、服务可靠性、高可用性，不知道对不对，请老师指导

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，作为架构要先保证可靠性，再保证性能。应用可靠、数据可靠是1，性能是后面的0

##### *伟：
> 貌似alibaba生态还是用skywalking做服务链路追踪，然后我不太看好gateway网关

##### **鑫：
> 负载均衡，分布式事务

##### *锋：
> a+p+最终一致性

##### *铁：
> 首先想到是系统可用性，安全性，一致性

##### **秋：
> 服务间通信、事务处理

##### *果：
> 事务，可用性，容错性😅

##### **弗：
> 不能重复支付，分布式事务，高可用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在分布式架构设计时必须时刻考虑幂等性，分布式事务尽量规避。
关于分布式事务话题后面咱们会基于Seata进行详细介绍

##### **9953：
> 逐字斟酌，静候更新

