<p data-nodeid="1105" class="">在 06 讲“权限管理：如何剖析 Spring Security 的授权原理？”中，我们介绍 Spring Security 授权流程时提到了过滤器的概念。今天，我们就直面这个主题，详细分析 Spring Security 中的过滤器架构，进一步学习实现自定义过滤器的系统方法。</p>
<h3 data-nodeid="1106">Spring Security 过滤器架构</h3>
<p data-nodeid="1107">过滤器是一种通用机制，在处理 Web 请求的过程中发挥了重要作用。可以说，目前市面上所有的 Web 开发框架都或多或少使用了过滤器完成对请求的处理，Spring Security 也不例外。Spring Security 中的过滤器架构是<strong data-nodeid="1233">基于 Servlet</strong>构建的，所以我们先从 Servlet 中的过滤器开始说起。</p>
<h4 data-nodeid="1108">Servlet 与管道-过滤器模式</h4>
<p data-nodeid="1109">和业界大多数处理 Web 请求的框架一样，Servlet 中采用的最基本的架构就是管道-过滤器（Pipe-Filter）架构模式。管道-过滤器架构模式的示意图如下所示：</p>
<p data-nodeid="1110"><img src="https://s0.lgstatic.com/i/image6/M01/46/46/Cgp9HWDJ2q6ADHioAABYcUjpFY4513.png" alt="Drawing 0.png" data-nodeid="1238"></p>
<div data-nodeid="1111"><p style="text-align:center">管道-过滤器架构模式示意图</p></div>
<p data-nodeid="1112">结合上图我们可以看到，处理业务逻辑的组件被称为过滤器，而处理结果通过相邻过滤器之间的管道进行传输，这样就构成了一个过滤器链。</p>
<p data-nodeid="1113">在 Servlet 中，代表过滤器的 Filter 接口定义如下：</p>
<pre class="lang-java" data-nodeid="1114"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Filter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(FilterConfig filterConfig)</span> <span class="hljs-keyword">throws</span> ServletException</span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilter</span><span class="hljs-params">(ServletRequest request, ServletResponse response, FilterChain chain)</span> <span class="hljs-keyword">throws</span> IOException, ServletException</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">destroy</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="1115">当应用程序启动时，Servlet 容器就会调用 init() 方法。<strong data-nodeid="1246">这个方法只会在容器启动时调用一次</strong>，因此包含了初始化过滤器的相关代码。对应的，destroy() 方法用于释放该过滤器占有的资源。</p>
<p data-nodeid="1116">一个过滤器组件所包含的业务逻辑应该位于 doFilter() 方法中，该方法带有三个参数，分别是<strong data-nodeid="1252">ServletRequest、ServletResponse 和 FilterChain</strong>。这三个参数都很重要，我们一一说明。</p>
<ul data-nodeid="1117">
<li data-nodeid="1118">
<p data-nodeid="1119">ServletRequest：表示 HTTP 请求，我们使用该对象获取有关请求的详细信息。</p>
</li>
<li data-nodeid="1120">
<p data-nodeid="1121">ServletResponse：表示 HTTP 响应，我们使用该对象构建响应结果，然后将其发送回客户端或沿着过滤器链向后传递。</p>
</li>
<li data-nodeid="1122">
<p data-nodeid="1123">FilterChain：表示过滤器链，我们使用该对象将请求转发到链中的下一个过滤器。</p>
</li>
</ul>
<p data-nodeid="1124">请注意，<strong data-nodeid="1261">过滤器链中的过滤器是有顺序的</strong>，这点非常重要，我们在本讲后续内容中会针对这点展开讲解。</p>
<h4 data-nodeid="1125">Spring Security 中的过滤器链</h4>
<p data-nodeid="1126">在 Spring Security 中，其核心流程的执行也是依赖于一组过滤器，这些过滤器在框架启动后会自动进行初始化，如图所示：</p>
<p data-nodeid="1127"><img src="https://s0.lgstatic.com/i/image6/M01/46/46/Cgp9HWDJ2rmAfrXzAABUC_CslH8843.png" alt="Drawing 1.png" data-nodeid="1266"></p>
<div data-nodeid="1128"><p style="text-align:center">Spring Security 中的过滤器链示意图</p></div>
<p data-nodeid="1129">在上图中，我们看到了几个常见的 Filter，比如 BasicAuthenticationFilter、UsernamePasswordAuthenticationFilter 等，这些类都直接或间接实现了 Servlet 中的 Filter 接口，并完成某一项具体的认证机制。例如，上图中的 BasicAuthenticationFilter 用来验证用户的身份凭证；而 UsernamePasswordAuthenticationFilter 会检查输入的用户名和密码，并根据认证结果决定是否将这一结果传递给下一个过滤器。</p>
<p data-nodeid="1130">请注意，<strong data-nodeid="1277">整个 Spring Security 过滤器链的末端是一个 FilterSecurityInterceptor，它本质上也是一个 Filter</strong>。但与其他用于完成认证操作的 Filter 不同，它的核心功能是<strong data-nodeid="1278">实现权限控制</strong>，也就是用来判定该请求是否能够访问目标 HTTP 端点。FilterSecurityInterceptor 对于权限控制的粒度可以到方法级别，能够满足前面提到的精细化访问控制。我们在 06 讲“权限管理：如何剖析 Spring Security 的授权原理？”中已经对这个拦截器做了详细的介绍，这里就不再展开了。</p>
<p data-nodeid="1131">通过上述分析，我们明确了在 Spring Security 中，认证和授权这两个安全性需求是通过一系列的过滤器来实现的。而过滤器的真正价值不仅在于实现了认证和授权，更为开发人员提供了一个扩展 Spring Security 框架的有效手段。</p>
<h3 data-nodeid="1132">实现自定义过滤器</h3>
<p data-nodeid="1133">在 Spring Security 中创建一个新的过滤器并不复杂，只需要<strong data-nodeid="1286">遵循 Servlet 所提供的 Filter 接口约定</strong>即可。</p>
<h4 data-nodeid="1134">开发过滤器</h4>
<p data-nodeid="1135">讲到开发自定义的过滤器，最经典的应用场景就是记录 HTTP 请求的访问日志。如下所示就是一种常见的实现方式：</p>
<pre class="lang-java" data-nodeid="1136"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LoggingFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Logger logger =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Logger.getLogger(AuthenticationLoggingFilter.class.getName());
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilter</span><span class="hljs-params">(ServletRequest request, ServletResponse response, FilterChain filterChain)</span> <span class="hljs-keyword">throws</span> IOException, ServletException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpServletRequest httpRequest = (HttpServletRequest) request;
&nbsp;
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;  <span class="hljs-comment">//从 ServletRequest 获取请求数据并记录</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String uniqueRequestId = httpRequest.getHeader(<span class="hljs-string">"UniqueRequestId"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.info(<span class="hljs-string">"成功对请求进行了认证： "</span> +&nbsp; uniqueRequestId);
&nbsp;
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; <span class="hljs-comment">//将请求继续在过滤器链上进行传递</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filterChain.doFilter(request, response);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="1137">这里我们定义了一个 LoggingFilter，用来记录已经通过用户认证的请求中包含的一个特定的消息头“UniqueRequestId”，通过这个唯一的请求 Id，我们可以对请求进行跟踪、监控和分析。在实现一个自定义的过滤器组件时，我们通常会从 ServletRequest 中获取请求数据，并在 ServletResponse 中设置响应数据，然后通过 filterChain 的 doFilter() 方法将请求继续在过滤器链上进行传递。</p>
<p data-nodeid="1138">接下来，我们想象这样一种场景，业务上我们需要根据客户端请求头中是否包含某一个特定的标志位，来决定请求是否有效。如图所示：</p>
<p data-nodeid="1139"><img src="https://s0.lgstatic.com/i/image6/M00/46/4E/CioPOWDJ2seAeDCHAABb0EbFFhw357.png" alt="Drawing 2.png" data-nodeid="1293"></p>
<div data-nodeid="1140"><p style="text-align:center">根据标志位设计过滤器示意图</p></div>
<p data-nodeid="1141">这在现实开发过程中也是一种常见的应用场景，可以实现定制化的安全性控制。针对这种应用场景，我们可以实现如下所示的 RequestValidationFilter 过滤器：</p>
<pre class="lang-java" data-nodeid="1142"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RequestValidationFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilter</span><span class="hljs-params">(ServletRequest request, ServletResponse response, FilterChain filterChain)</span> <span class="hljs-keyword">throws</span> IOException, ServletException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpServletRequest httpRequest = (HttpServletRequest) request;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpServletResponse httpResponse = (HttpServletResponse) response;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String requestId = httpRequest.getHeader(<span class="hljs-string">"SecurityFlag"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (requestId == <span class="hljs-keyword">null</span> || requestId.isBlank()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; httpResponse.setStatus(HttpServletResponse.SC_BAD_REQUEST);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filterChain.doFilter(request, response);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="1143">这里我们从 HttpServletRequest 对象的请求头中获取了“SecurityFlag”标志位，否则将直接抛出一个 400 Bad Request 响应结果。根据需要，我们也可以实现各种自定义的异常处理逻辑。</p>
<h4 data-nodeid="1144">配置过滤器</h4>
<p data-nodeid="1145">现在，我们已经实现了几个有价值的过滤器了，下一步就是将这些过滤器整合到 Spring Security 的整个过滤器链中。这里，我想特别强调一点，和 Servlet 中的过滤器一样，<strong data-nodeid="1302">Spring Security 中的过滤器也是有顺序的</strong>。也就是说，将过滤器放置在过滤器链的具体位置需要符合每个过滤器本身的功能特性，不能将这些过滤器随意排列组合。</p>
<p data-nodeid="1146">我们来举例说明合理设置过滤器顺序的重要性。在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7696&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1306">“用户认证：如何使用 Spring Security 构建用户认证体系？”</a>一讲中我们提到了 HTTP 基础认证机制，而在 Spring Security 中，实现这一认证机制的就是 BasicAuthenticationFilter。</p>
<p data-nodeid="1147">如果我们想要实现定制化的安全性控制策略，就可以实现类似前面介绍的 RequestValidationFilter 这样的过滤器，并放置在 BasicAuthenticationFilter 前。这样，在执行用户认证之前，我们就可以排除掉一批无效请求，效果如下所示：</p>
<p data-nodeid="1148"><img src="https://s0.lgstatic.com/i/image6/M00/46/4E/CioPOWDJ2tWAOEqvAABaXzH63gI541.png" alt="Drawing 3.png" data-nodeid="1311"></p>
<div data-nodeid="1149"><p style="text-align:center">RequestValidationFilter 的位置示意图</p></div>
<p data-nodeid="1150">上图中的 RequestValidationFilter 确保那些没有携带有效请求头信息的请求不会执行不必要的用户认证。基于这种场景，把 RequestValidationFilter 放在 BasicAuthenticationFilter 之后就不是很合适了，因为用户已经完成了认证操作。</p>
<p data-nodeid="1151">同样，针对前面已经构建的 LoggingFilter，原则上我们可以把它放在过滤器链的任何位置，因为它只记录了日志。但有没有更合适的位置呢？结合 RequestValidationFilter 来看，同样对于一个无效的请求而言，记录日志是没有什么意义的。所以 LoggingFilter 应该放置在 RequestValidationFilter 之后。另一方面，对于日志操作而言，通常只需要记录那些已经通过认证的请求，所以也推荐将 LoggingFilter 放在 BasicAuthenticationFilter 之后。最终，这三个过滤器之间的关系如下图所示：</p>
<p data-nodeid="1152"><img src="https://s0.lgstatic.com/i/image6/M00/46/4E/CioPOWDJ2t-AMX8kAAAyTVuPA_s000.png" alt="Drawing 4.png" data-nodeid="1316"></p>
<div data-nodeid="1153"><p style="text-align:center">三个过滤器的位置示意图</p></div>
<p data-nodeid="1154">在 Spring Security 中，提供了一组可以往过滤器链中添加过滤器的工具方法，包括 addFilterBefore()、addFilterAfter()、addFilterAt() 以及 addFilter() 等，它们都定义在 HttpSecurity 类中。这些方法的含义都很明确，使用起来也很简单，例如，想要实现如上图所示的效果，我们可以编写这样的代码：</p>
<pre class="lang-java" data-nodeid="1155"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.addFilterBefore(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> RequestValidationFilter(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BasicAuthenticationFilter.class)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .addFilterAfter(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> LoggingFilter(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BasicAuthenticationFilter.class)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizeRequests()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .anyRequest()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .permitAll();
}
</code></pre>
<p data-nodeid="1156">这里，我们使用了 addFilterBefore() 和 addFilterAfter() 方法在 BasicAuthenticationFilter 之前和之后分别添加了 RequestValidationFilter 和 LoggingFilter。</p>
<h3 data-nodeid="1157">Spring Security 中的过滤器</h3>
<p data-nodeid="1158">下表列举了 Spring Security 中常用的过滤器名称、功能以及它们的顺序关系：</p>
<p data-nodeid="1374" class=""><img src="https://s0.lgstatic.com/i/image6/M00/47/4E/CioPOWDP_VCAc51eAADyJ6yBpaY941.png" alt="image.png" data-nodeid="1377"></p>

<div data-nodeid="1211"><p style="text-align:center">Spring Security 中的常见过滤器一览表</p></div>
<p data-nodeid="1212">这里以最基础的 UsernamePasswordAuthenticationFilter 为例，该类的定义及核心方法 attemptAuthentication 如下所示：</p>
<pre class="lang-java" data-nodeid="1213"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UsernamePasswordAuthenticationFilter</span> <span class="hljs-keyword">extends</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-title">AbstractAuthenticationProcessingFilter</span> </span>{
&nbsp;
    <span class="hljs-function"><span class="hljs-keyword">public</span> Authentication <span class="hljs-title">attemptAuthentication</span><span class="hljs-params">(HttpServletRequest request,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpServletResponse response)</span> <span class="hljs-keyword">throws</span> AuthenticationException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (postOnly &amp;&amp; !request.getMethod().equals(<span class="hljs-string">"POST"</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> AuthenticationServiceException(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"Authentication method not supported: "</span> + request.getMethod());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = obtainUsername(request);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = obtainPassword(request);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (username == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; username = <span class="hljs-string">""</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (password == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; password = <span class="hljs-string">""</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; username = username.trim();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UsernamePasswordAuthenticationToken authRequest = <span class="hljs-keyword">new</span> UsernamePasswordAuthenticationToken(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; username, password);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; setDetails(request, authRequest);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.getAuthenticationManager().authenticate(authRequest);
    }
    …
}
</code></pre>
<p data-nodeid="1214">围绕上述方法，我们结合前面已经介绍的认证和授权相关实现原理，可以引出该框架中一系列核心类并梳理它们之间的交互结构，如下图所示：</p>
<p data-nodeid="1754" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/47/4E/CioPOWDP_VuAIAgyAAFOMaUuRl4162.png" alt="image (1).png" data-nodeid="1762"></p>
<div data-nodeid="1755"><p style="text-align:center">UsernamePasswordAuthenticationFilter 相关核心类图</p></div>


<p data-nodeid="1217">上图中的很多类，我们通过名称就能明白它的含义和作用。以位于左下角的 SecurityContextHolder 为例，它是一个典型的 Holder 类，存储了应用的安全上下文对象 SecurityContext，而这个上下文对象中就包含了用户的认证信息。</p>
<p data-nodeid="1218">我们也可以大胆猜想，它的内部应该使用 ThreadLocal 确保线程访问的安全性。更具体的，我们已经在“权限管理：如何剖析 Spring Security 的授权原理？”中讲解过 SecurityContext 的使用方法。</p>
<p data-nodeid="1219">正如 UsernamePasswordAuthenticationFilter 中的代码所示，一个 HTTP 请求到达之后，会通过一系列的 Filter 完成用户认证，而具体的工作交由 AuthenticationManager 完成，这个过程又会涉及 AuthenticationProvider 以及 UserDetailsService 等多个核心组件之间的交互。关于 Spring Security 中认证流程的详细描述，你可以参考<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7697&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1364">“认证体系：如何深入理解 Spring Security 的用户认证机制？”</a>做一些回顾。</p>
<h3 data-nodeid="1220">小结与预告</h3>
<p data-nodeid="1221">这一讲我们关注于 Spring Security 中的一个核心组件——过滤器。在请求-响应式处理框架中，过滤器发挥着重要的作用，它用来实现对请求的拦截，并定义认证和授权逻辑。同时，我们也可以根据需要实现各种自定义的过滤器组件，从而实现对 Spring Security 的动态扩展。本讲对 Spring Security 中的过滤器架构和开发方式都做了详细的介绍，你可以反复学习。</p>
<p data-nodeid="1222">本讲内容总结如下：</p>
<p data-nodeid="1223"><img src="https://s0.lgstatic.com/i/image6/M00/46/4E/CioPOWDJ2viAN_UiAAB6xt7iJjo286.png" alt="Drawing 6.png" data-nodeid="1371"></p>
<p data-nodeid="1224">最后，给你留一道思考题：在 Spring Security 中，你能简单描述使用过滤器实现用户认证的操作过程吗？欢迎你在留言区和我分享自己的观点。</p>
<p data-nodeid="1225" class="">介绍完过滤器机制，下一讲，让我们把目光转向攻击应对部分，讨论 Spring Security 框架如何应对跨站请求伪造攻击，以及如何实现跨域资源共享。</p>

---

### 精选评论

##### *西：
> 用于认证的过滤器会把登陆请求拦截下来，然后提取请求中的登陆凭证，比如用户名跟密码，然后去数据库查询用户信息，比对密码，正确的话就登陆成功，然后设置到holder类，标志已认证成功标识为true，然后请求继续调用后面的过滤器。

##### JackLi：
> 期待下次更新！

