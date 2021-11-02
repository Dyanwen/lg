<p data-nodeid="745" class="">通过前面课程的学习，我们已经知道 Spring Security 可以集成 OAuth2 协议并实现分布式环境下的访问授权。同时，Spring Security 也可以和 Spring Cloud 框架无缝集成，并完成对各个微服务的权限控制。</p>
<p data-nodeid="746">今天我们将设计一个案例系统，从零构建一个完整的微服务系统，除了演示微服务系统构建过程，还将重点展示 OAuth2 协议以及 JWT 在其中所起到的作用。</p>
<h3 data-nodeid="747">案例驱动：SpringAppointment</h3>
<p data-nodeid="748">在本课程中，我们通过构建一个相对精简的完整系统，来展示微服务架构相关的设计理念以及各项技术组件，这个案例系统称为 SpringAppointment。</p>
<p data-nodeid="749">SpringAppointment 包含的业务场景比较简单，可以用来模拟就医过程中的预约处理流程。一般而言，预约流程势必会涉及三个独立的微服务，即就诊卡（Card）服务、预约（Appointment）服务，以及医生（Doctor）服务。</p>
<p data-nodeid="750">我们把以上三个服务统称为<strong data-nodeid="828">业务服务</strong>。纵观整个 SpringAppointment 系统，除了这三个业务微服务之外，还有一批非业务性的基础设施类服务，具体包括：注册中心服务（Eureka）、配置中心服务（Spring Cloud Config），以及 API 网关服务（Zuul）。关于 Spring Cloud 中基础设施类服务的构建过程不是本专栏的重点，你可以参考拉勾上<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=492&amp;sid=20-h5Url-0&amp;lgec_type=website&amp;lgec_sign=86228E00A960E2EB44DCA4027393428B&amp;buyFrom=2&amp;pageId=1pz4#/sale" data-nodeid="826">《Spring Cloud 原理与实战》</a>专栏做详细了解。</p>
<p data-nodeid="751">虽然案例中的各个服务在物理上都是独立的，但就整个系统而言，需要各服务相互协作构成一个完整的微服务系统。也就是说，服务运行时存在一定的依赖性。我们结合系统架构对 SpringAppointment 的运行方式进行梳理，梳理的基本方法就是按照服务列表构建独立服务，并基于注册中心来管理它们之间的依赖关系，如下图所示：</p>
<p data-nodeid="2057" class=""><img src="https://s0.lgstatic.com/i/image6/M00/4D/49/Cgp9HWDuW_OAckmvAAIEbG-aKzA773.png" alt="图片1.png" data-nodeid="2061"></p>
<div data-nodeid="2058"><p style="text-align:center">基于注册中心的服务运行时依赖关系图</p></div>








<h3 data-nodeid="754">构建 OAuth2 授权服务</h3>
<p data-nodeid="755" class="">在上图中，我们注意到还存在着案例系统中的最后一个基础设施类微服务，即 OAuth2 授权服务，在这里充当着授权中心的作用。关于 OAuth2 授权服务的具体构建步骤已经在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="837">《13</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="840">|</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="843">授权体系：如何在微服务架构中集成</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="846">OAuth2</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="849">协议？》</a>做了详细介绍，这里我们直接创建 WebSecurityConfigurerAdapter 的子类 WebSecurityConfigurer</p>
<p data-nodeid="756">以及 AuthorizationServerConfigurerAdapter 的子类 JWTOAuth2Config，实现代码如下所示：</p>
<pre class="lang-java" data-nodeid="757"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebSecurityConfigurer</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> AuthenticationManager <span class="hljs-title">authenticationManagerBean</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.authenticationManagerBean();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> UserDetailsService <span class="hljs-title">userDetailsServiceBean</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.userDetailsServiceBean();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder builder)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;  builder.inMemoryAuthentication().withUser(<span class="hljs-string">"user"</span>).password(<span class="hljs-string">"{noop}password1"</span>).roles(<span class="hljs-string">"USER"</span>).and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .withUser(<span class="hljs-string">"admin"</span>).password(<span class="hljs-string">"{noop}password2"</span>).roles(<span class="hljs-string">"USER"</span>, <span class="hljs-string">"ADMIN"</span>);
&nbsp;&nbsp;&nbsp; }
}
&nbsp;
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JWTOAuth2Config</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AuthorizationServerConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationManager authenticationManager;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserDetailsService userDetailsService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthorizationServerEndpointsConfigurer endpoints)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;  endpoints.authenticationManager(authenticationManager).userDetailsService(userDetailsService);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(ClientDetailsServiceConfigurer clients)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;  clients.inMemory().withClient(<span class="hljs-string">"appointment_client"</span>).secret(<span class="hljs-string">"{noop}appointment_secret"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizedGrantTypes(<span class="hljs-string">"refresh_token"</span>, <span class="hljs-string">"password"</span>, <span class="hljs-string">"client_credentials"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .scopes(<span class="hljs-string">"webclient"</span>, <span class="hljs-string">"mobileclient"</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<h3 data-nodeid="758">初始化业务服务</h3>
<p data-nodeid="759">在 SpringAppointment 案例系统中，我们需要构建三个业务微服务，即 card-service、appointment-service 和 doctor-service，它们都是独立的 Spring Boot 应用程序。在构建业务服务时，我们首先需要完成它们与基础设施类服务集成。因为 API 网关起到的是服务路由作用，所以对于各个业务服务而言是透明的，而其他的注册中心、配置中心和授权中心都需要每个业务服务完成与它们之间的集成。</p>
<h4 data-nodeid="760">集成注册中心</h4>
<p data-nodeid="761">对于注册中心 Eureka 而言，card-service、appointment-service 和 doctor-service 都是它的客户端，所以需要 spring-cloud-starter-netflix-eureka-client 的依赖，如下所示。</p>
<pre data-nodeid="762"><code>&lt;dependency&gt;
&nbsp;&nbsp;&nbsp; &lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&lt;artifactId&gt;spring-cloud-starter-netflix-eureka-client&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="763">然后，我们以 appointment-service 为例，来看它的 Bootstrap 类，如下所示：</p>
<pre class="lang-java" data-nodeid="764"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableEurekaClient</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AppointmentApplication</span> </span>{
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
	&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(AppointmentApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="765">这里引入了一个新的注解 @EnableEurekaClient，该注解用于表明当前服务就是一个 Eureka 客户端，这样该服务就可以自动注册到 Eureka 服务器。当然，我们也可以直接使用统一的 @SpringCloudApplication 注解来实现 @SpringBootApplication 和 @EnableEurekaClient这两个注解整合在一起的效果。</p>
<p data-nodeid="766">接下来就是最重要的配置工作，appointment-service 中的配置内容如下所示：</p>
<pre class="lang-xml" data-nodeid="767"><code data-language="xml">spring:
&nbsp; application:
	name: appointmentservice
server:
&nbsp; port: 8081
	&nbsp;
eureka:
&nbsp; client:
&nbsp;&nbsp;&nbsp; registerWithEureka: true
&nbsp;&nbsp;&nbsp; fetchRegistry: true
&nbsp;&nbsp;&nbsp; serviceUrl:
	&nbsp; defaultZone: http://localhost:8761/eureka/
</code></pre>
<p data-nodeid="768">显然，这里包含两段配置内容。其中，第一段配置指定了服务的名称和运行时端口。在上面的示例中 appointment-service 的名称通过“spring.application.name=appointmentservice”进行指定，也就是说 appointment-service 在注册中心中的名称为 appointmentservice。在后续的示例中，我们会使用这一名称获取 appointment-service 在 Eureka 中的各种注册信息。</p>
<h4 data-nodeid="769">集成配置中心</h4>
<p data-nodeid="770">要想获取配置服务器中的配置信息，我们首先需要初始化客户端，也就是在将各个业务微服务与 Spring Cloud Config 服务器端进行集成。初始化客户端的第一步是引入 Spring Cloud Config 的客户端组件 spring-cloud-config-client，如下所示。</p>
<pre class="lang-xml" data-nodeid="771"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-config-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="772">然后我们需要在配置文件 application.yml 中配置服务器的访问地址，如下所示：</p>
<pre class="lang-xml" data-nodeid="773"><code data-language="xml">spring: 
&nbsp; cloud:
&nbsp;&nbsp;&nbsp; config:
	&nbsp;&nbsp; enabled: true
	 &nbsp; uri: http://localhost:8888
</code></pre>
<p data-nodeid="774">以上配置信息中，我们指定了配置服务器所在的地址，也就是上面的 uri：<a href="http://localhost:8888" data-nodeid="866">http://localhost:8888</a>。</p>
<p data-nodeid="775">一旦我们引入了 Spring Cloud Config 的客户端组件，相当于在各个微服务中自动集成了访问配置服务器中 HTTP 端点的功能。也就是说，访问配置服务器的过程对于各个微服务而言是透明的，即微服务不需要考虑如何从远程服务器获取配置信息，而只需要考虑如何在 Spring Boot 应用程序中使用这些配置信息。而对于常见的关系型数据访问配置而言，Spring 已经帮助我们内置了整合过程，我们要做的就是引入相关的依赖组件而已。</p>
<p data-nodeid="776">我们以 appointment-service 为例来演示数据库访问功能，案例中使用的是 JPA 和 MySQL，因此需要在服务中引入相关的依赖，如下所示：</p>
<pre class="lang-xml" data-nodeid="777"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
	&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
	&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-data-jpa<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
	&nbsp;
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
	&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>mysql<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
	&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>mysql-connector-java<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="778">现在，我们就可以使用 JPA 提供的数据访问功能来访问 MySQL 数据库了。</p>
<h4 data-nodeid="779">集成授权中心</h4>
<p data-nodeid="780">在业务服务中集成授权中心的实现方法，我们已经在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7708" data-nodeid="875">《14.资源保护：如何使用</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7708" data-nodeid="878">OAuth2</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7708" data-nodeid="881">协议实现对微服务访问进行授权？》</a>中做了详细介绍，这里做一些简单的回顾。首先，我们需要在 Spring Boot 的启动类上添加 @EnableResourceServer 注解：</p>
<pre class="lang-java" data-nodeid="781"><code data-language="java"><span class="hljs-meta">@SpringCloudApplication</span>
<span class="hljs-meta">@EnableResourceServer</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AppointmentApplication</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(AppointmentApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="782">然后，我们需要在配置文件中指定授权中心服务的地址：</p>
<pre class="lang-xml" data-nodeid="783"><code data-language="xml">security:
&nbsp; oauth2:
&nbsp;&nbsp; &nbsp;resource:
	&nbsp; userInfoUri: http://localhost:8080/userinfo
</code></pre>
<p data-nodeid="784">最后，要做的就是在每个业务服务中嵌入访问授权控制。我们可以使用用户层级的权限访问控制、用户+角色层级的权限访问控制，以及用户+角色+操作层级的权限访问控制这三种策略中的任意一种来实现这一目标。</p>
<h3 data-nodeid="785">集成和扩展 JWT</h3>
<p data-nodeid="2814" class="">让我们再次回到 SpringAppointment 案例系统，以用户下单这一业务场景为例，就涉及 appointment-service 同时调用 doctor-service 和 card-service，这三个服务之间的交互方式如下图所示：<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4D/52/CioPOWDuXAyAPXM5AAGY7XqKIyA677.png" alt="图片2.png" data-nodeid="2820"></p>
<div data-nodeid="2815"><p style="text-align:center">SpringAppointment 案例系统中三个业务微服务的交互方式图</p></div>




<p data-nodeid="788">通过这个交互图，实际上我们已经可以梳理出这一场景下的代码结构了，如下所示：</p>
<pre class="lang-java" data-nodeid="789"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Appointment <span class="hljs-title">generateAppointment</span><span class="hljs-params">(String doctorName, String cardCode)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Appointment appointment = <span class="hljs-keyword">new</span> Appointment();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取远程 Card 信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CardMapper card = getCard(cardCode);
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;  …

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取远程 Doctor 信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DoctorMapper doctor = getDoctor(doctorName);
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;  …

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; appointmentRepository.save(appointment);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> appointment;
}
</code></pre>
<p data-nodeid="790">其中 appointment-service 从 card-service 获取 Card 对象，以及从 doctor-service 中获取 Doctor 对象，这两个步骤都会涉及远程 Web 服务的访问。因此，我们首先需要分别在 card-service 和 doctor-service 服务中创建对应的 HTTP 端点。这一过程不是课程的重点，如果你感兴趣，可以参考案例源码自己进行学习：<a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringAppointment" data-nodeid="895">https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringAppointment</a>。</p>
<h4 data-nodeid="791">集成 JWT</h4>
<p data-nodeid="792">在《15 | 令牌扩展：如何使用JWT实现定制化 Token？》中，我们引入了 JWT 并完成了与 OAuth2 协议的集成，从而实现了定制化的 Token。JWT 同样也需要在整个服务调用链路中进行传递。而持有 JWT 的客户端访问 appointment-service 提供的 HTTP 端点进行下单操作，该服务会验证所传入 JWT 的有效性。然后，appointment-service 会再通过网关访问 card-service 和 doctor-service，同样这两个服务也会分别对所传入的 JWT 进行验证，并返回相应的结果。</p>
<p data-nodeid="793">现在，让我们在 appointment-service 中构建一个 CardRestTemplateClient 类，会发现它使用了在《15 | 令牌扩展：如何使用 JWT 实现定制化 Token？》中所创建的 RestTemplate 对象来发起远程调用，代码如下所示：</p>
<pre class="lang-java" data-nodeid="794"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CardRestTemplateClient</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; RestTemplate restTemplate;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> CardMapper <span class="hljs-title">getCardByCardCode</span><span class="hljs-params">(String cardCode)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseEntity&lt;CardMapper&gt; result =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; restTemplate.exchange(<span class="hljs-string">"http://cardservice/cards/{cardCode}"</span>, HttpMethod.GET, <span class="hljs-keyword">null</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CardMapper.class, cardCode);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result.getBody();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="795">我们知道在这个 RestTemplate 中，基于 AuthorizationHeaderInterceptor 对请求进行了拦截，从而完成了 JWT 在各个服务中的正确传播。</p>
<p data-nodeid="796">最后，我们通过 Postman 来验证以上流程的正确性。通过访问 Zuul 中配置的 appointment-service 端点，并传入角色为“ADMIN”的用户对应的 Token 信息，可以看到订单记录已经被成功创建。你可以尝试通过生成不同的 Token 来执行这一流程，并验证授权效果。</p>
<h4 data-nodeid="797">扩展 JWT</h4>
<p data-nodeid="798">在案例的最后，我们来讨论一下如何扩展 JWT。JWT 具有良好的可扩展性，开发人员可以根据需要在 JWT Token 中添加自己想要添加的各种附加信息。</p>
<p data-nodeid="799">针对 JWT 的扩展性场景，Spring Security 专门提供了一个 TokenEnhancer 接口来对 Token 进行增强（Enhance），该接口定义如下：</p>
<pre class="lang-java" data-nodeid="800"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">TokenEnhancer</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function">OAuth2AccessToken <span class="hljs-title">enhance</span><span class="hljs-params">(OAuth2AccessToken accessToken, OAuth2Authentication authentication)</span></span>;
}
</code></pre>
<p data-nodeid="801">可以看到这里处理的是一个 OAuth2AccessToken 接口，而该接口有一个默认的实现类 DefaultOAuth2AccessToken。我们可以通过该实现类的 setAdditionalInformation 方法，以键值对的方式将附加信息添加到 OAuth2AccessToken 中，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="802"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JWTTokenEnhancer</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">TokenEnhancer</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> OAuth2AccessToken <span class="hljs-title">enhance</span><span class="hljs-params">(OAuth2AccessToken accessToken, OAuth2Authentication authentication)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; systemInfo = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; systemInfo.put(<span class="hljs-string">"system"</span>, <span class="hljs-string">"Appointment System"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ((DefaultOAuth2AccessToken) accessToken).setAdditionalInformation(systemInfo);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accessToken;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="803">这里我们以硬编码的方式添加了一个“system”属性，你也可以根据需要进行相应的调整。</p>
<p data-nodeid="804">要想使得上述 JWTTokenEnhancer 类能够生效，我们需要对 JWTOAuth2Config 类中的 configure 方法进行重新配置，并将 JWTTokenEnhancer 嵌入到 TokenEnhancerChain 中，如下所示：</p>
<pre class="lang-java" data-nodeid="805"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthorizationServerEndpointsConfigurer endpoints)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TokenEnhancerChain tokenEnhancerChain = <span class="hljs-keyword">new</span> TokenEnhancerChain();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; tokenEnhancerChain.setTokenEnhancers(Arrays.asList(jwtTokenEnhancer, jwtAccessTokenConverter)); 
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; endpoints.tokenStore(tokenStore)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .accessTokenConverter(jwtAccessTokenConverter)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .tokenEnhancer(tokenEnhancerChain)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authenticationManager(authenticationManager)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .userDetailsService(userDetailsService);
}
</code></pre>
<p data-nodeid="806">请注意，我们在这里通过创建一个 TokenEnhancer 的列表将包括 JWTTokenEnhancer 在内的多个 TokenEnhancer 嵌入到 TokenEnhancerChain 中。</p>
<p data-nodeid="807">现在，我们已经扩展了 JWT Token。那么，如何从这个 JWT Token 中获取所扩展的属性呢？方法也比较简单和固定，如下所示：</p>
<pre class="lang-java" data-nodeid="808"><code data-language="java"><span class="hljs-comment">//获取 JWTToken</span>
RequestContext ctx = RequestContext.getCurrentContext();
String authorizationHeader = ctx.getRequest().getHeader(AUTHORIZATION_HEADER);
String jwtToken = authorizationHeader.replace(<span class="hljs-string">"Bearer "</span>,<span class="hljs-string">""</span>);

<span class="hljs-comment">//解析 JWTToken</span>
String[] split_string = jwtToken.split(<span class="hljs-string">"\\."</span>);
String base64EncodedBody = split_string[<span class="hljs-number">1</span>];
Base64 base64Url = <span class="hljs-keyword">new</span> Base64(<span class="hljs-keyword">true</span>);
String body = <span class="hljs-keyword">new</span> String(base64Url.decode(base64EncodedBody));
JSONObject jsonObj = <span class="hljs-keyword">new</span> JSONObject(body);

<span class="hljs-comment">//获取定制化属性值</span>
String systemName = jsonObj.getString(<span class="hljs-string">"system"</span>);
</code></pre>
<p data-nodeid="809">我们可以把这段代码嵌入到需要使用到自定义“system”属性的任何场景中。</p>
<h3 data-nodeid="810">小结与预告</h3>
<p data-nodeid="811">案例分析是掌握一个框架应用方式的最好方法，对于 OAuth2 协议也是一样。本讲中，我们将 Spring Security 结合 Spring Cloud 构建了一个微服务案例系统 SpringAppointment。然后根据 SpringAppointment 案例中的业务场景划分了各个微服务，并重点介绍了各个业务服务的构建过程。我们一方面展示了业务服务与基础设施服务的集成过程，另一方面也演示了如何集成和扩展 JWT 的实现过程。</p>
<p data-nodeid="812">最后再给你留一道思考题：在业务系统中如何实现对 JWT 进行定制化的扩展呢？欢迎在留言区和我分享你的收获。</p>
<p data-nodeid="813" class="">介绍完 OAuth2 协议以及与 Spring Cloud 框架的集成之后，下一讲我们将介绍 OAuth2 协议的另一个常见的应用场景，这就是单点登录。</p>

---

### 精选评论

##### **林：
> nice

