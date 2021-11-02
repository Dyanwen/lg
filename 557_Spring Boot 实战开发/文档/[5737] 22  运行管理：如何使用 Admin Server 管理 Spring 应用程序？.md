<p data-nodeid="889" class="">前面 2 讲通过引入 Actuator 组件，我们为 Spring Boot 应用程序添加了系统监控功能。基于 Actuator 暴露的各种 HTTP 端点，开发人员可以获取系统的运行时状态。而端点是一种底层的监控技术，这就要求我们对 HTTP 协议和 Spring Boot 应用程序的构建方式有一定的了解。</p>
<p data-nodeid="890">那么，有没有更简单的、基于可视化的方式获取这些端点背后的信息呢？答案是肯定的。因此，这一讲我们将要介绍 Spring Boot Admin 组件。</p>
<h3 data-nodeid="891">引入 Spring Boot Admin 组件</h3>
<p data-nodeid="1509">Spring Boot Admin 是一个用于监控 Spring Boot 的应用程序，它的基本原理是通过统计、集成 Spring Boot Actuator 中提供的各种 HTTP 端点，从而提供简洁的可视化 WEB UI，如下图所示：</p>
<p data-nodeid="1510" class=""><img src="https://s0.lgstatic.com/i/image/M00/93/0C/Ciqc1GATrxuABqR3AAC7e5_Dyo4605.png" alt="图片1.png" data-nodeid="1515"></p>
<div data-nodeid="1511"><p style="text-align:center">Spring Boot Admin 基本原理图</p></div>




<p data-nodeid="895">从上图中，我们不难看出，Spring Boot Admin 的整体架构中存在两大角色，即服务器端组件 Admin Server 和客户端组件 Admin Client。其中，Admin Client 实际上是一个普通的 Spring Boot 应用程序，而 Admin Server 则是一个独立服务，需要进行专门构建。</p>
<p data-nodeid="896">接下来，我们先介绍构建 Admin Server 的两种实现方式：一种是简单的基于独立的 Admin 服务；另一种则相对复杂，需要依赖服务注册中心的服务注册和发现机制。</p>
<h4 data-nodeid="897">基于独立服务构建 Admin Server</h4>
<p data-nodeid="898">无论使用哪种方式实现 Admin Server，首先我们都需要创建一个 Spring Boot 应用程序，并在 pom 文件中添加如下所示的依赖项：</p>
<pre class="lang-xml" data-nodeid="899"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>de.codecentric<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-admin-server<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>de.codecentric<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-admin-server-ui<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="900"><strong data-nodeid="1003">请注意： Spring Boot Admin 组件并不是 Spring 家族官方提供的组件，而是来自一个 codecentric AG 团队。</strong></p>
<p data-nodeid="901">如果我们想将普通的 Spring Boot 应用程序转变为 Spring Boot Admin Server，只需要在 Bootstrap 类上添加一个 @EnableAdminServer 注解即可，添加完该注解的 BootStrap 类如下代码所示：</p>
<pre class="lang-java" data-nodeid="902"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableAdminServer</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AdminApplication</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(AdminApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="903">此时，我们会发现使用这种方式构建 Spring Boot Admin Server 就是这么简单。</p>
<p data-nodeid="904">接下来我们启动这个 Spring Boot 应用程序，并打开 Web 界面，就能看到如下所示的效果：</p>
<p data-nodeid="905"><img src="https://s0.lgstatic.com/i/image/M00/90/48/CgqCHmAKf-WAFILtAAAsRr3Jgfg085.png" alt="Drawing 1.png" data-nodeid="1009"></p>
<div data-nodeid="906"><p style="text-align:center">Spring Boot Admin Server 启动效果图</p></div>
<p data-nodeid="907">从图中我们可以看到，目前还没有一个应用程序与 Admin Server 有关联。如果想将应用程序与 Admin Server 进行关联，我们还需要对原有的 Spring Boot 应用程序做一定的改造。</p>
<p data-nodeid="908">首先，我们在 Maven 依赖中引入对 Spring Boot Admin Client 组件的依赖，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="909"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>de.codecentric<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-admin-starter-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="910">然后，我们在配置文件中添加如下配置信息，以便该应用程序能够与 Admin Server 进行关联。</p>
<pre class="lang-xml" data-nodeid="911"><code data-language="xml">spring:
&nbsp; boot:
&nbsp;&nbsp;&nbsp; admin:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; client:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; url: http://localhost:9000
</code></pre>
<p data-nodeid="912"><strong data-nodeid="1016">注意：这里的 9000 就是 Admin Server 的服务器端口。</strong></p>
<p data-nodeid="913">现在我们启动这个应用程序，就会发现 Admin Server 中已经出现了这个应用的名称和地址，如下图所示：</p>
<p data-nodeid="914"><img src="https://s0.lgstatic.com/i/image/M00/90/3D/Ciqc1GAKf--ATR-aAAAxAFlSEVc094.png" alt="Drawing 2.png" data-nodeid="1020"></p>
<div data-nodeid="915"><p style="text-align:center">Spring Boot Admin Server 添加了应用程序之后的效果图</p></div>
<p data-nodeid="916">在图中，我们看到 APPLICATIONS 和 INSTANCES 的数量都是 1，代表 Admin Server 管理着一个应用程序，而该应用程序只有一个运行实例。在界面的下方，我们还能看到这个应用的名称及实例地址。这里你可以尝试使用不同的端口启动应用程序的不同实例，然后观察这个列表的变化。</p>
<h4 data-nodeid="917">基于注册中心构建 Admin Server</h4>
<p data-nodeid="918">虽然基于独立服务构建 Admin Server 和 Admin Client 非常简单，但是需要我们在每个应用程序中添加对 Spring Boot Admin 的 Maven 依赖，并指定 Admin Server 地址。这实际上是一种代码侵入，意味着应用程序与 Admin Server 之间有一种强耦合。</p>
<p data-nodeid="919">那么，有没有更好的办法分离或转嫁这种耦合呢？</p>
<p data-nodeid="920">联想到 Admin Server 和 Admin Client 之间需要建立类似服务注册的关联关系，我们可以认为这是服务注册和发现机制的一种表现形式。</p>
<p data-nodeid="921">在 Spring 家族中，存在一个用于构建微服务架构的 Spring Cloud 框架，而该框架中恰好存在一款专门实现服务注册和发现的组件——服务注册中心 Spring Cloud Netflix Eureka ，且 Spring Boot Admin 内置了与这款注册中心实现工具的无缝集成。</p>
<p data-nodeid="2345">基于注册中心，Admin Server 与各个 Admin Client 之间的交互方式如下图所示：</p>
<p data-nodeid="2346" class=""><img src="https://s0.lgstatic.com/i/image/M00/93/17/CgqCHmATr26AO1VxAACbHS3yQHY687.png" alt="图片13.png" data-nodeid="2351"></p>
<div data-nodeid="2347"><p style="text-align:center">基于 Eureka 的 Admin Server 与 Admin Client 交互图</p></div>




<p data-nodeid="925">使用 Eureka 构建注册中心的过程也很简单，首先我们创建一个独立的 Spring Boot 应用程序，并在 pom 文件中添加如下所示的用于提供 Eureka 服务端功能的 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="926"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-netflix-eureka-server<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="927">引入 Maven 依赖后，我们就可以创建 Spring Boot 的启动类。在示例代码中，我们把该启动类命名为 EurekaServerApplication，如下代码所示：</p>
<pre class="lang-java" data-nodeid="928"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableEurekaServer</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EurekaServerApplication</span> </span>{
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
	&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(EurekaServerApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="929">注意：在上面的代码中，我们在启动类上加了一个@EnableEurekaServer 注解。在 SpringCloud 中，包含 @EnableEurekaServer 注解的服务也就是一个 Eureka 服务器组件。这样，Eureka 服务就构建完毕了。</p>
<p data-nodeid="930">同样，Eureka 服务还为我们提供了一个可视化的 UI 界面，它可以用来观察当前注册到 Eureka 中的应用程序信息，如下图所示：</p>
<p data-nodeid="931"><img src="https://s0.lgstatic.com/i/image2/M01/08/31/Cip5yGAKgDuAcMmLAAB_2n8YYlw199.png" alt="Drawing 4.png" data-nodeid="1037"></p>
<div data-nodeid="932"><p style="text-align:center">Eureka 服务监控页面</p></div>
<p data-nodeid="933">接下来，我们需要 Admin Server 也做相应调整。首先，我们在 pom 文件中添加一个对 spring-cloud-starter-netflix-eureka-client 这个 Eureka 客户端组件的依赖：</p>
<pre class="lang-xml" data-nodeid="934"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-netflix-eureka-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="935">这时 Admin Server 相当于 Eureka 的客户端，因此，我们需要在它的 BootStrap 类上添加 @EnableEurekaClient 注解，以便将 Admin Server 注册到 Eureka 上。</p>
<p data-nodeid="936">重构 Admin Server 的最后一步是调整配置信息，此时我们需要在配置文件中添加如下所示的配置项来指定 Eureka 服务器地址。</p>
<pre class="lang-xml" data-nodeid="937"><code data-language="xml">eureka:
&nbsp; client:
&nbsp;&nbsp;&nbsp; registerWithEureka: true
&nbsp;&nbsp;&nbsp; fetchRegistry: true
&nbsp;&nbsp;&nbsp; serviceUrl:
	&nbsp; defaultZone: http://localhost:8761/eureka/
</code></pre>
<p data-nodeid="938">好了，现在 Admin Server 已经重构完毕，接下来我们一起看看 Admin Client。</p>
<p data-nodeid="939">引入注册中心的目的是降低 Admin Client 与 Admin Server 之间的耦合度，关于这点我们从 Maven 依赖上就可以得到印证。有了注册中心后，Admin Client 就不再依赖 spring-boot-admin-starter-client 组件了，而是直接使用如下所示的 Eureka 客户端组件。</p>
<pre class="lang-xml" data-nodeid="940"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-netflix-eureka-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="941">在配置文件中，我们需要去掉对 Admin Server 地址的引用，直接使用 Eureka 服务端地址即可，且无须对 Admin Client 中的 Bootstrap 类做任何修改。</p>
<p data-nodeid="942">通过以上调整，各个 Admin Client 就能通过 Eureka 注册中心完成与 Admin Server 的关联了。</p>
<h3 data-nodeid="943">使用 Admin Server 监控系统</h3>
<p data-nodeid="944">根据 Spring Boot Admin 官方 Github 上的介绍，Admin Server 监控系统提供了一套完整的可视化方案。基于 Admin Server，健康状态、JVM、内存、Micrometer 的度量、线程、HTTP 跟踪等核心功能都可以通过可视化的 UI 界面进行展示。</p>
<h4 data-nodeid="945">监控系统运行时关键指标</h4>
<p data-nodeid="946">注意到 Admin Server 菜单中有一个“Wallboard”，点击该菜单，我们就可以看到一面应用墙，如下图所示：</p>
<p data-nodeid="947"><img src="https://s0.lgstatic.com/i/image2/M01/08/32/Cip5yGAKgE2AdGRqAABpdUogwxw880.png" alt="Drawing 5.png" data-nodeid="1051"></p>
<div data-nodeid="948"><p style="text-align:center">Admin Server 应用墙</p></div>
<p data-nodeid="3181">点击应用墙中的某个应用，我们就能进入针对该应用的监控信息主界面。在该界面的左侧，包含了监控功能的各级目录，如下图所示：</p>
<p data-nodeid="3182" class=""><img src="https://s0.lgstatic.com/i/image/M00/93/0C/Ciqc1GATr3-ATLpGAAOgyIEu7Sk069.png" alt="图片6.png" data-nodeid="3187"></p>
<div data-nodeid="3183" class=""><p style="text-align:center">Admin Server 监控信息主界面</p></div>




<p data-nodeid="952">在图中，我们看到了最重要的“Health”信息，显然，这一信息来自 Spring Boot Actuator 组件的 Health 端点，这里你可以参考《服务监控：如何使用 Actuator 组件实现系统监控？》的内容进行回顾。</p>
<p data-nodeid="4017">在这个界面上继续往下滑动，我们将看到一些与 JVM 相关的监控信息，比如非常有用的线程、垃圾回收、内存状态等数据，如下图所示：</p>
<p data-nodeid="4018" class=""><img src="https://s0.lgstatic.com/i/image/M00/93/17/CgqCHmATr5KAJhb-AANDUbOuW2I534.png" alt="图片7.png" data-nodeid="4023"></p>
<div data-nodeid="4019"><p style="text-align:center">Admin Server 中的 JVM 监控信息</p></div>




<p data-nodeid="956">这些 JVM 数据都是通过可视化的方式进行展现，并随着运行时状态的变化而实时更新。</p>
<p data-nodeid="957">在 21 讲中，我们详细讨论了 Spring Boot Actuator 中的度量指标。而在 Admin Server 中，同样存在一个“Metrics”菜单，展示效果如下图所示：</p>
<p data-nodeid="958"><img src="https://s0.lgstatic.com/i/image/M00/90/49/CgqCHmAKgGSAaC9sAAA-CBnX4LI723.png" alt="Drawing 8.png" data-nodeid="1065"></p>
<div data-nodeid="959"><p style="text-align:center">Admin Server 中的 Metrics 信息</p></div>
<p data-nodeid="960">在“Metrics”菜单中，开发人员可以通过对各种条件进行筛选，然后添加对应的度量指标。比如上图中，我们针对 HTTP 请求中 /actuator/health 端点进行了过滤，从而得到了度量结果。</p>
<p data-nodeid="961">接着我们一起看看系统环境方面的属性，因为这方面的属性非常之多，所以 Admin Server 也提供了一个过滤器，如下图所示：</p>
<p data-nodeid="962"><img src="https://s0.lgstatic.com/i/image/M00/90/3E/Ciqc1GAKgGuAaRNpAABIghaOFVg132.png" alt="Drawing 9.png" data-nodeid="1070"></p>
<div data-nodeid="963"><p style="text-align:center">Admin Server 中的 Environment 信息</p></div>
<p data-nodeid="964">在上图中，通过输入“spring.”参数，我们就能获取一系列与该参数相关的环境属性。</p>
<p data-nodeid="4853">日志也是我们监控系统的一个重要途径，在 Admin Server 的“Loggers”菜单中，可以看到该应用程序的所有日志信息，如下图所示：</p>
<p data-nodeid="5278" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/93/0C/Ciqc1GATr7OAegcuAAGy8bdkv4k234.png" alt="图片10.png" data-nodeid="5282"></p>
<div data-nodeid="5279"><p style="text-align:center">Admin Server 中的 Loggers 信息</p></div>






<p data-nodeid="968">通过”springcss”关键词对这些日志进行过滤，我们就可以获取 SpringCSS 案例中的日志详细了，图中也显示了每个日志记录器对应的日志级别。</p>
<p data-nodeid="969">最后，我们来看一下 Admin Server 中的“JVM”菜单，该菜单下存在两个子菜单：“Thread Dump”和“Heap Dump”。</p>
<p data-nodeid="970">以“Thread Dump”为例，尽管 Actuator 提供了 /threaddump 端点，但开发人员只能获取触发该端点时的 Dump 信息，而 Admin Server 则提供了一个连续性的可视化监控界面，如下图所示：</p>
<p data-nodeid="971"><img src="https://s0.lgstatic.com/i/image/M00/90/49/CgqCHmAKgHuAcsFSAABDhuAgJBY760.png" alt="Drawing 11.png" data-nodeid="1081"></p>
<div data-nodeid="972"><p style="text-align:center">Admin Server 中的 Thread Dump 信息</p></div>
<p data-nodeid="973">点击图中的色条，我们就可以获取每一个线程的详细信息了，这里你可以尝试做一些分析。</p>
<h4 data-nodeid="974">控制访问安全性</h4>
<p data-nodeid="975">讲到这里，我们会发现 Admin Server 的功能非常强大，而这些功能显然也不应该暴露给所有的开发人员。因此，我们需要控制 Admin Server 的访问安全性。</p>
<p data-nodeid="976">想做到这一点也非常简单，我们只需要集成 Spring Security 即可。</p>
<p data-nodeid="977">结合《用户认证：如何基于 Spring Security 构建用户认证体系？》的内容，我们在 Spring Boot 应用程序中添加一个对 spring-boot-starter-security 的 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="978"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-security<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="979">然后，我们在配置文件中添加如下配置项：</p>
<pre class="lang-xml" data-nodeid="980"><code data-language="xml">spring:
&nbsp; security:
&nbsp;&nbsp;&nbsp; user:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; name: "springcss_admin"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; password: "springcss_password"
</code></pre>
<p data-nodeid="981">重启 Admin Server 后，再次访问 Web 界面时，就需要我们输入用户名和密码了，如下图所示：</p>
<p data-nodeid="982"><img src="https://s0.lgstatic.com/i/image2/M01/08/34/CgpVE2AKgImAOicJAAAiQ2MCOts677.png" alt="Drawing 12.png" data-nodeid="1091"></p>
<div data-nodeid="983"><p style="text-align:center">Admin Server 的安全登录界面</p></div>
<h3 data-nodeid="984">小结与预告</h3>
<p data-nodeid="985">可视化监控一直是开发和运维人员管理应用程序运行时状态的基础诉求，而 Spring Boot Admin 组件正是这样一款可视化的工具。它基于 Spring Boot Actuator 中各个端点所暴露的监控信息，并加以整合和集成。今天的内容首先介绍了构建 Admin Server 以及 Admin Client 的方法，并剖析了 Admin Server 中所具有的一整套的可视化解决方案。</p>
<p data-nodeid="986">这里给你留一道思考题：在使用 Spring Boot Admin 组件时，构建 Admin Server 有哪两种方法？欢迎你在留言区进行互动、交流。</p>
<p data-nodeid="987">介绍完系统监控主题之后，我们将进入到整个课程的最后一个主题，即系统测试。23讲我们将介绍如何对数据访问层组件进行有效测试。</p>
<p data-nodeid="988" class="">另外，如果你觉得本专栏有价值，欢迎分享给好友哦~</p>

---

### 精选评论

##### 代：
> 我只能说不知道是版本不对还是流程就是有问题，按照步骤来，admin 主页根本打不开，场面一度很尴尬

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个组件使用上还是相对简单的，具体系统有什么提示或报错可以贴一下，我们一起看看。

