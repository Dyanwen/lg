<p data-nodeid="689" class="">在 01 讲中，我们提到 Spring 家族具备很多款开源框架，开发人员可以基于这些开发框架实现各种 Spring 应用程序。在 02 讲中，我们无意对所有这些 Spring 应用程序的类型和开发方式过多展开，而是主要集中在基于 Spring Boot 开发面向 Web 场景的服务，这也是互联网应用程序最常见的表现形式。在介绍基于 Spring Boot 的开发模式之前，让我们先将它与传统的 Spring MVC 进行简单对比。</p>
<h3 data-nodeid="690">Spring MVC VS Spring Boot</h3>
<p data-nodeid="691">在典型的 Web 应用程序中，前后端通常采用基于 HTTP 协议完成请求和响应，开发过程中需要完成 URL 地址的映射、HTTP 请求的构建、数据的序列化和反序列化以及实现各个服务自身内部的业务逻辑，如下图所示：</p>
<p data-nodeid="692"><img src="https://s0.lgstatic.com/i/image/M00/70/96/CgqCHl-7WIiAQimVAABHwO_CPqU821.png" alt="Drawing 0.png" data-nodeid="773"></p>
<div data-nodeid="693"><p style="text-align:center">HTTP 请求响应过程</p></div>
<p data-nodeid="694">我们先来看基于 Spring MVC 完成上述开发流程所需要的开发步骤，如下图所示：</p>
<p data-nodeid="695"><img src="https://s0.lgstatic.com/i/image/M00/70/96/CgqCHl-7WI2AaDcrAABBSHVyAdE329.png" alt="Drawing 1.png" data-nodeid="777"></p>
<div data-nodeid="696"><p style="text-align:center">基于 Spring MVC 的 Web 应用程序开发流程</p></div>
<p data-nodeid="697">上图中包括使用 web.xml 定义 Spring 的 DispatcherServlet、完成启动 Spring MVC 的配置文件、编写响应 HTTP 请求的 Controller 以及将服务部署到 Tomcat Web 服务器等步骤。事实上，基于传统的 Spring MVC 框架开发 Web 应用逐渐暴露出一些问题，比较典型的就是配置工作过于复杂和繁重，以及缺少必要的应用程序管理和监控机制。</p>
<p data-nodeid="698">如果想优化这一套开发过程，有几个点值得我们去挖掘，比方说减少不必要的配置工作、启动依赖项的自动管理、简化部署并提供应用监控等。而这些优化点恰巧推动了以 Spring Boot 为代表的新一代开发框架的诞生，基于 Spring Boot 的开发流程见下图：</p>
<p data-nodeid="699"><img src="https://s0.lgstatic.com/i/image/M00/70/8A/Ciqc1F-7WJeAHD-nAABRAXax5k4419.png" alt="Drawing 2.png" data-nodeid="782"></p>
<div data-nodeid="700"><p style="text-align:center">基于 Spring Boot 的 Web 应用程序开发流程</p></div>
<p data-nodeid="701">从上图中可以看到，它与基于 Spring MVC 的开发流程在配置信息的管理、服务部署和监控等方面有明显不同。作为 Spring 家族新的一员，Spring Boot 提供了令人兴奋的特性，这些特性的核心价值在于确保了开发过程的简单性，具体体现在编码、配置、部署、监控等多个方面。</p>
<p data-nodeid="702">首先，Spring Boot 使编码更简单。我们只需要在 Maven 中添加一项依赖并实现一个方法就可以提供微服务架构中所推崇的 RESTful 风格接口。</p>
<p data-nodeid="703">其次，Spring Boot 使配置更简单。它把 Spring 中基于 XML 的功能配置方式转换为 Java Config，同时提供了 .yml 文件来优化原有基于 .properties 和 .xml 文件的配置方案，.yml 文件对配置信息的组织更为直观方便，语义也更为强大。同时，基于 Spring Boot 的自动配置特性，对常见的各种工具和框架均提供了默认的 starter 组件来简化配置。</p>
<p data-nodeid="704">最后，在部署方案上，Spring Boot 也创造了一键启动的新模式。Spring Boot 部署包结构参考下图：</p>
<p data-nodeid="705"><img src="https://s0.lgstatic.com/i/image/M00/70/8A/Ciqc1F-7WKCALlK_AAAvL2X5nlU081.png" alt="Drawing 3.png" data-nodeid="789"></p>
<div data-nodeid="706"><p style="text-align:center">Spring Boot部署包结构</p></div>
<p data-nodeid="707">从图中我们可以看到，相较于传统模式下的 war 包，Spring Boot 部署包既包含了业务代码和各种第三方类库，同时也内嵌了 HTTP 容器。这种包结构支持 java –jar application.jar 方式的一键启动，不需要部署独立的应用服务器，通过默认内嵌 Tomcat 就可以运行整个应用程序。</p>
<p data-nodeid="708">最后，基于 Spring Boot 新提供的 Actuator 组件，开发和运维人员可以通过 RESTful 接口获取应用程序的当前运行时状态并对这些状态背后的度量指标进行监控和报警。例如可以通过“/env/{name}”端点获取系统环境变量、通过“/mapping”端点获取所有 RESTful 服务、通过“/dump”端点获取线程工作状态以及通过“/metrics/{name}”端点获取 JVM 性能指标等。</p>
<h3 data-nodeid="709">剖析一个 Spring Web 应用程序</h3>
<p data-nodeid="710">针对一个基于 Spring Boot 开发的 Web 应用程序，其代码组织方式需要遵循一定的项目结构。在 02 讲中，如果不做特殊说明，我们都将使用 Maven 来管理项目工程中的结构和包依赖。一个典型的 Web 应用程序的项目结构如下图所示：</p>
<p data-nodeid="711"><img src="https://s0.lgstatic.com/i/image/M00/70/96/CgqCHl-7WKuAE6hIAABP4_ORBpU588.png" alt="Drawing 4.png" data-nodeid="796"></p>
<div data-nodeid="712"><p style="text-align:center">Spring Boot Web 项目结构图</p></div>
<p data-nodeid="713">在上图中，有几个地方需要特别注意，我也在图中做了专门的标注，分别是包依赖、启动类、控制器类以及配置，让我们讲此部分内容分别做一些展开。</p>
<h4 data-nodeid="714">包依赖</h4>
<p data-nodeid="715">Spring Boot 提供了一系列 starter 工程来简化各种组件之间的依赖关系。以开发 Web 服务为例，我们需要引入 spring-boot-starter-web 这个工程，而这个工程中并没有具体的代码，只是包含了一些 pom 依赖，如下所示：</p>
<ul data-nodeid="716">
<li data-nodeid="717">
<p data-nodeid="718">org.springframework.boot:spring-boot-starter</p>
</li>
<li data-nodeid="719">
<p data-nodeid="720">org.springframework.boot:spring-boot-starter-tomcat</p>
</li>
<li data-nodeid="721">
<p data-nodeid="722">org.springframework.boot:spring-boot-starter-validation</p>
</li>
<li data-nodeid="723">
<p data-nodeid="724">com.fasterxml.jackson.core:jackson-databind</p>
</li>
<li data-nodeid="725">
<p data-nodeid="726">org.springframework:spring-web</p>
</li>
<li data-nodeid="727">
<p data-nodeid="728">org.springframework:spring-webmvc</p>
</li>
</ul>
<p data-nodeid="729">可以看到，这里包括了传统 Spring MVC 应用程序中会使用到的 spring-web 和 spring-webmvc 组件，因此 Spring Boot 在底层实现上还是基于这两个组件完成对 Web 请求响应流程的构建。</p>
<p data-nodeid="730">如果我们使用 Spring Boot 2.2.4 版本，你会发现它所依赖的 Spring 组件都升级到了 5.X 版本，如下图所示：</p>
<p data-nodeid="731"><img src="https://s0.lgstatic.com/i/image/M00/70/8A/Ciqc1F-7WLOAHBZ_AABP1erBa_k988.png" alt="Drawing 5.png" data-nodeid="810"></p>
<div data-nodeid="732"><p style="text-align:center">Spring Boot 2.2.4 版本的包依赖示意图</p></div>
<p data-nodeid="733">在应用程序中引入 spring-boot-starter-web 组件就像引入一个普通的 Maven 依赖一样，如下所示。</p>
<pre class="lang-xml" data-nodeid="734"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="735">一旦 spring-boot-starter-web 组件引入完毕，我们就可以充分利用 Spring Boot 提供的自动配置机制开发 Web 应用程序。</p>
<h4 data-nodeid="736">启动类</h4>
<p data-nodeid="737">使用 Spring Boot 的最重要的一个步骤是创建一个 Bootstrap 启动类。Bootstrap 类结构简单且比较固化，如下所示：</p>
<pre class="lang-java" data-nodeid="738"><code data-language="java"><span class="hljs-keyword">import</span> org.springframework.boot.SpringApplication;
<span class="hljs-keyword">import</span> org.springframework.boot.autoconfigure.SpringBootApplication;
&nbsp;
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloApplication</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(HelloApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="739">显然，这里引入了一个全新的注解 @SpringBootApplication。在 Spring Boot 中，添加了该注解的类就是整个应用程序的入口，一方面会启动整个 Spring 容器，另一方面也会自动扫描代码包结构下的 @Component、@Service、@Repository、@Controller 等注解并把这些注解对应的类转化为 Bean 对象全部加载到 Spring 容器中。</p>
<h4 data-nodeid="740">控制器类</h4>
<p data-nodeid="741">Bootstrap 类为我们提供了 Spring Boot 应用程序的入口，相当于应用程序已经有了最基本的骨架。接下来我们就可以添加 HTTP 请求的访问入口，表现在 Spring Boot 中也就是一系列的 Controller 类。这里的 Controller 与 Spring MVC 中的 Controller 在概念上是一致的，一个典型的 Controller 类如下所示：</p>
<pre class="lang-java" data-nodeid="742"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value = "accounts")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountController</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AccountService accountService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping(value = "/{accountId}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Account <span class="hljs-title">getAccountById</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("accountId")</span> Long accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Account account = accountService.getAccountById(accountId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> account;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="743">请注意，以上代码中包含了 @RestController、@RequestMapping 和 @GetMapping 这三个注解。其中，@RequestMapping 用于指定请求地址的映射关系，@GetMapping 的作用等同于指定了 GET 请求的 @RequestMapping 注解，而 @RestController 注解是传统 Spring MVC 中所提供的 @Controller 注解的升级版，相当于就是 @Controller 和 @ResponseEntity 注解的结合体，会自动使用 JSON 实现序列化/反序列化操作。</p>
<h4 data-nodeid="744">配置文件</h4>
<p data-nodeid="745">我们注意到，在 src/main/resources 目录下存在一个 application.yml 文件，这就是 Spring Boot 中的主配置文件。例如，我们可以将如下所示的端口、服务名称以及数据库访问等配置信息添加到这个配置文件中：</p>
<pre class="lang-xml te-preview-highlight" data-nodeid="2178"><code data-language="xml">server:
&nbsp; port: 8081
&nbsp;
spring:
&nbsp; application:
&nbsp;&nbsp;&nbsp; name: orderservice 
&nbsp; datasource:
&nbsp;&nbsp;&nbsp; driver-class-name: com.mysql.cj.jdbc.Driver
&nbsp;&nbsp;&nbsp; url: jdbc:mysql://127.0.0.1:3306/appointment
&nbsp;&nbsp;&nbsp; username: root
    password: root
</code></pre>





<p data-nodeid="747">事实上，Spring Boot 提供了强大的自动配置机制，如果没有特殊的配置需求，开发人员完全可以基于 Spring Boot 内置的配置体系完成诸如数据库访问相关配置信息的自动集成。</p>
<h3 data-nodeid="748">案例驱动：SpringCSS</h3>
<p data-nodeid="749">介绍完了基于 Spring Boot 创建一个 Web 应用的基本流程之后，我们将引出本课程的案例系统 SpringCSS，这里的 CSS 是对客户服务系统 Customer Service System 的简称。客服服务是电商、健康类业务场景中非常常见的一种业务场景，我们将通过构建一个精简但又完整的系统来展示 Spring Boot 相关设计理念和各项技术组件。</p>
<p data-nodeid="750">现实场景下的客户服务业务逻辑一般都非常复杂，而案例系统的目的在于演示技术实现过程，不在于介绍具体业务逻辑。所以，我们对案例的业务流程做了高度的简化，但涉及的各项技术都可以直接应用到日常开发过程中。</p>
<h4 data-nodeid="751">SpringCSS 整体架构</h4>
<p data-nodeid="752">在 SpringCSS 中，存在一个 customer-service，这是一个 Spring Boot 应用程序，也是整个案例系统中的主体服务。在该服务中，我们可以将采用经典的分层架构，即将服务分成 Web 层、Service 层和 Repository 层。</p>
<p data-nodeid="753">在客服系统中，我们知道其核心业务是生成客户工单。为此，customer-service 一般会与用户服务 account-service 进行交互，但因为用户账户信息的更新属于低频事件，所以我们设计的实现方式是 account-service 通过消息中间件的方式将用户账户变更信息主动推送给 customer–service，从而完成用户信息的获取操作。而针对 order-service，其定位是订单系统，customer-service 也需要从该服务中查询订单信息。SpringCSS 的整个系统交互过程如下图所示：</p>
<p data-nodeid="754"><img src="https://s0.lgstatic.com/i/image/M00/70/96/CgqCHl-7WMKAR1x_AACL4oIyVgU534.png" alt="Drawing 6.png" data-nodeid="830"></p>
<div data-nodeid="755"><p style="text-align:center">SpringCSS 系统的整体架构图</p></div>
<p data-nodeid="756">在上图中，引出了构建 SpringCSS 的多项技术组件，在后续课程中我们会对这些技术组件做专题介绍。</p>
<h4 data-nodeid="757">从案例实战到原理剖析</h4>
<p data-nodeid="758">更进一步，通过案例帮你完成基于 Spring Boot 框架构建 Web 应用程序是 02 讲的一大目标，但也不是唯一目标。作为扩展，希望我们通过对优秀开源框架的学习，掌握各个核心组件背后的运行机制，进而深入理解架构的实现原理。</p>
<p data-nodeid="759">在本专栏中，我们将通过熟悉源码，剖析 Spring Boot 中核心组件的工作原理，典型的场景包括 Spring Boot 的自动配置实现原理、数据库访问实现原理、HTTP 远程调用实现原理等。</p>
<p data-nodeid="760">通过源码级的深入剖析学习上述核心组件的实现原理时，你可以掌握系统架构设计和实现过程中的方法和技巧，并指导日常的开发工作。</p>
<h3 data-nodeid="761">小结与预告</h3>
<p data-nodeid="762">案例分析是掌握一个框架应用方式的最好方法。本课程是一款以案例驱动的 Spring Boot 应用程序开发课程，今天我们主要针对一个典型的 Spring Boot Web 应用程序的组织结构和开发方式进行了详细介绍，并引出了贯穿整个课程体系的 SpringCSS 案例系统。</p>
<p data-nodeid="763">这里给你留一道思考题：在实现基于 Spring Boot 的 Web 应用程序时，开发人员的开发工作涉及哪几个方面？</p>
<p data-nodeid="764">从 03 讲开始，我们就将基于 SpringCSS 案例来解析 Spring Boot 框架，首当其冲的是它的配置体系。</p>
<p data-nodeid="765"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="844"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="843"></a></p>
<p data-nodeid="766"><strong data-nodeid="848">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="767" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="852">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **波：
> application.yml和bootstrap.yml有什么区别？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这两个都是配置文件，本质上是一样的，其中 bootstrap 比 application 更先加载，可以理解为 bootstrap 是系统级别的配置文件，application 是应用级别的配置文件。

##### **波：
> 学完

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 太棒了，学习效率好高，给你大大的赞~

##### **9137：
> @ResponseEntity和@ResponseBody是一样的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; @ResponseEntity注解比@ResponseBody注解多添加了http状态码相关的内容，相当于@ResponseStatus与@ResponseBody=
ResponseEntity

##### *杰：
> 很接地气

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 给你笔芯，你一定要将这门接地气的课程坚持学完哦，加油~

##### *瑶：
> 这讲很好

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢小可爱，小编很开心，请不要停！

##### **林：
> SpringBoot大行其道，必须的掌握！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油~你可以的

##### **璐：
> 领域对象设计以及实现、接口设计及实现(包括业务逻辑)、数据库表结构、配置文件，想到的就这些了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的。

##### **6781：
> 彩

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 彩~彩~

##### *里：
> 添加不上抽奖助手了，是不是被封号了？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 请您点此链接http://to1.top/aqEvuqai 添加小助手（建议用手机打开）。

