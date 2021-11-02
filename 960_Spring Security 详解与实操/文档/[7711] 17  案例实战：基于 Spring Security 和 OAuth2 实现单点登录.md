<p data-nodeid="179177" class="">单点登录（Single Sign-On，SSO）是我们设计和实现 Web 系统时经常需要面临的一个问题，允许用户使用一组凭据来登录多个相互独立但又需要保持统一登录状态的 Web 应用程序。单点登录的实现需要特定的技术和框架，而 Spring Security 也提供了它的解决方案。本课时将基于 OAuth2 协议来构建 SSO。</p>









<h3 data-nodeid="3238">什么是单点登录？</h3>
<p data-nodeid="18598" class="">与其说 SSO 是一种技术体系，不如说它是一种应用场景。因此，我们有必要先来看看 SSO 与本专栏前面所介绍的各种技术体系之间的关联关系。</p>




<h4 data-nodeid="3240">单点登录与OAuth2协议</h4>
<p data-nodeid="23622" class="">假设存在 A 和 B 两个独立的系统，但它们相互信任，并通过单点登录系统进行了统一的管理和维护。那么无论访问系统 A 还是系统 B，当用户在身份认证服务器上登录一次以后，即可获得访问另一个系统的权限。同时这个过程是完全自动化的，SSO 通过实现集中式登录系统来达到这一目标，该系统处理用户的身份认证并与其他应用程序共享该认证信息。</p>








<p data-nodeid="26150" class="">说到这里，你可能会问为什么我们需要实施 SSO 呢？原因很简单，因为它提供了很多优势。下面我们具体分析一下。</p>


<ul data-nodeid="29009">
<li data-nodeid="29010">
<p data-nodeid="29011">首先，借助 SSO 可以确保系统更加安全，我们只需要一台集中式服务器来管理用户身份，而不需要将用户凭证扩展到各个服务，因此能够减少被攻击的维度。</p>
</li>
<li data-nodeid="29012">
<p data-nodeid="29013">其次，可以想象持续输入用户名和密码来访问不同的服务，是一件让用户感到很困扰的事情。而 SSO 将不同的服务组合在一起，以便用户可以在服务之间进行无缝导航，从而提高用户体验。</p>
</li>
<li data-nodeid="29014" class="">
<p data-nodeid="29015">同时，SSO 也能帮助我们更好地了解客户，因为我们拥有对客户信息的单一视图，能够更好地构建用户画像。</p>
</li>
</ul>







<p data-nodeid="39562" class="">那么，如何构建 SSO 呢？各个公司可能有不同的做法，而采用 Spring Security 和 OAuth2 协议是一个不错的选择，因为实现过程非常简单。虽然 OAuth2 一开始是用来允许用户授权第三方应用访问其资源的一种协议，也就是说其目标不是专门用来实现 SSO，但是我们可以利用它的功能特性来变相地实现单点登录，这就需要用到 OAuth2 四种授权模式中的授权码模式。<strong data-nodeid="39616">关于 OAuth2 协议和授权码模式</strong>，你可以参考<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7706" data-nodeid="39570">《12</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7706" data-nodeid="39573"> | </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7706" data-nodeid="39576">开放协议：OAuth2</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7706" data-nodeid="39579"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7706" data-nodeid="39582">协议解决的是什么问题？》</a>做一些回顾。同时，在使用 OAuth2 协议实现SSO时，我们也会使用 JWT 来生成和管理 Token，<strong data-nodeid="39617">关于 JWT</strong>，你也可以回顾<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39590">《15</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39593"> | </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39596">令牌扩展：如何使用</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39599"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39602">JWT</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39605"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39608">实现定制化</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39611"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="39614">Token？》</a>课时中的内容。</p>
















<h4 data-nodeid="3251">单点登录的工作流程</h4>
<p data-nodeid="46524" class="">在具体介绍实现方案之前，我们先对 SSO 的工作流程做一下展开，从而了解典型 SSO 系统背后的设计思想。下图描述了 SSO 流程，可以看到我们有两个应用程序 App1 和 App2，以及一个集中式 SSO 服务器。</p>











<p data-nodeid="4150" class=""><img src="https://s0.lgstatic.com/i/image6/M01/4E/DD/CioPOWD2gzCAVFtFAADBTaeFEM8683.png" alt="17讲.png" data-nodeid="4154"></p>

<div data-nodeid="48724" class=""><p style="text-align:center">SSO 工作流程图</p></div>

<p data-nodeid="48096" class="">结合上图，我们先来看针对 App1 的工作流程。</p>






<ul data-nodeid="69160">
<li data-nodeid="69161">
<p data-nodeid="69162">用户第一次访问 App1。由于用户未登录，所以将用户重定向到 SSO 服务器。</p>
</li>
<li data-nodeid="69163">
<p data-nodeid="69164">用户在 SSO 服务器提供的登录页面上输入用户凭据。SSO 服务器验证凭据并生成 SSO Token，然后 SSO 服务器在 Cookie 中保存这个 Token，以供用户进行后续登录。</p>
</li>
<li data-nodeid="69165">
<p data-nodeid="69166" class="">SSO 服务器将用户重定向到 App1。在重定向 URL 中，就会附上这个 SSO Token 作为查询参数。</p>
</li>
<li data-nodeid="69167">
<p data-nodeid="69168">App1 将 Token 保存在其 Cookie 中，并将当前的交互方式更改为已登录的用户。App1 可以通过查询 SSO 服务器或 Token 来获取与用户相关的信息。我们知道 JWT 是可以自定义扩展的，所以这时候就可以利用 JWT 来传递用户信息。</p>
</li>
</ul>
































<p data-nodeid="70427" class="">现在，我们再来看一下同一用户尝试访问 App2 的工作流程。</p>


<ul data-nodeid="80595">
<li data-nodeid="80596">
<p data-nodeid="80597">由于应用程序只能访问相同来源的 Cookie，它不知道用户已登录到 App2。因此，同样会将用户重定向到 SSO 服务器。</p>
</li>
<li data-nodeid="80598">
<p data-nodeid="80599" class="">SSO 服务器发现该用户已经设置了 Cookie，因此它会立即将用户重定向到 App2，并在 URL 中附加 SSO Token 作为查询参数。</p>
</li>
<li data-nodeid="80600">
<p data-nodeid="80601">App2 同样将 Token 存储在 Cookie 中，并将其交互方式更改为已登录用户。</p>
</li>
</ul>
















<p data-nodeid="84999" class="">整个流程结束之后，用户浏览器中将设置三个 Cookie，每个 Cookie 分别针对 App1、App2 和 SSO Server 域。</p>







<p data-nodeid="86255" class="">关于上述流程，业界存在各种各样的实现方案和工具，包括 Facebook Connect、Open Id Connect、CAS、Kerbos、SAML 等。我们无意对这些具体的工具做详细展开，而是围绕到目前为止已经掌握的技术来从零构建SSO服务器端和客户端组件。</p>


<h3 data-nodeid="87511" class="">实现 SSO 服务器端</h3>


<p data-nodeid="90651" class="">基于 Spring Security实现 SSO 服务端的核心工作，还是使用一系列我们已经很熟悉的配置体系，来配置基础的认证授权信息，以及与 OAuth2 协议之间的整合过程。</p>





<h4 data-nodeid="3276">配置基础认证和授权信息</h4>
<p data-nodeid="91907" class="">我们同样通过继承 WebSecurityConfigurerAdapter 类来实现自定义的认证和授权信息配置，这个过程比较简单，完整代码如下所示：</p>


<pre class="lang-java" data-nodeid="3278"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebSecurityConfiguration</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.userDetailsService(userDetailsServiceBean()).passwordEncoder(passwordEncoder());
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(WebSecurity web)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; web.ignoring().antMatchers(<span class="hljs-string">"/assets/**"</span>, <span class="hljs-string">"/css/**"</span>, <span class="hljs-string">"/images/**"</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.formLogin()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .loginPage(<span class="hljs-string">"/login"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizeRequests()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .antMatchers(<span class="hljs-string">"/login"</span>).permitAll()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .anyRequest()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authenticated()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and().csrf().disable().cors();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> UserDetailsService <span class="hljs-title">userDetailsServiceBean</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt;UserDetails&gt; users = buildUsers();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> InMemoryUserDetailsManager(users);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Collection&lt;UserDetails&gt; <span class="hljs-title">buildUsers</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = passwordEncoder().encode(<span class="hljs-string">"12345"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;UserDetails&gt; users = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserDetails user_admin = User.withUsername(<span class="hljs-string">"admin"</span>).password(password).authorities(<span class="hljs-string">"ADMIN"</span>, <span class="hljs-string">"USER"</span>).build();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; users.add(user_admin);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> users;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> PasswordEncoder <span class="hljs-title">passwordEncoder</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> BCryptPasswordEncoder();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> AuthenticationManager <span class="hljs-title">authenticationManagerBean</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.authenticationManagerBean();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="95047" class="">在上述代码中，我们综合使用了已经掌握的 Spring Security 中与认证、授权、密码管理、CSRF、CORS 相关的多项功能特性，通过loginPage()方法指定了 SSO 服务器上的登录界面地址，并初始化了一个“admin”用户用来执行登录操作。</p>





<h4 data-nodeid="96303" class="">配置 OAuth2 授权服务器</h4>


<p data-nodeid="99443" class="">然后，我们创建一个 AuthorizationServerConfiguration 类来继承 AuthorizationServerConfigurerAdapter，请注意，在这个类上需要添加 @EnableAuthorizationServer 注解，如下所示：</p>





<pre class="lang-java" data-nodeid="3282"><code data-language="java"><span class="hljs-meta">@EnableAuthorizationServer</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorizationServerConfiguration</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AuthorizationServerConfigurerAdapter</span> </span>{
</code></pre>
<p data-nodeid="107221" class="">配置 OAuth2 授权服务器的重点工作是指定需要参与 SSO 的客户端。在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107225">《13</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107228"> | </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107231">授权体系：如何在微服务架构中集成</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107234"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107237">OAuth2</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107240"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="107243">协议？》</a>课时中，我们给出了 Spring Security 中描述客户端详情的 ClientDetails 接口，以及用于管理 ClientDetails 的 ClientDetailsService。基于 ClientDetailsService，我们就可以定制化对ClientDetails的创建过程，示例代码如下所示：</p>












<pre class="lang-java" data-nodeid="3284"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ClientDetailsService <span class="hljs-title">inMemoryClientDetailsService</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> InMemoryClientDetailsServiceBuilder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;<span class="hljs-comment">//创建 app1 客户端</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withClient(<span class="hljs-string">"app1"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .secret(passwordEncoder.encode(<span class="hljs-string">"app1_secret"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .scopes(<span class="hljs-string">"all"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizedGrantTypes(<span class="hljs-string">"authorization_code"</span>, <span class="hljs-string">"refresh_token"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .redirectUris(<span class="hljs-string">"http://localhost:8080/app1/login"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .accessTokenValiditySeconds(<span class="hljs-number">7200</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .autoApprove(<span class="hljs-keyword">true</span>)
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 创建 app2 客户端</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .withClient(<span class="hljs-string">"app2"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .secret(passwordEncoder.encode(<span class="hljs-string">"app2_secret"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .scopes(<span class="hljs-string">"all"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizedGrantTypes(<span class="hljs-string">"authorization_code"</span>, <span class="hljs-string">"refresh_token"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .redirectUris(<span class="hljs-string">"http://localhost:8090/app2/login"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .accessTokenValiditySeconds(<span class="hljs-number">7200</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .autoApprove(<span class="hljs-keyword">true</span>)
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .build();
}
</code></pre>
<p data-nodeid="179805" class="">这里我们通过 InMemoryClientDetailsServiceBuilder 构建了一个基于内存的 ClientDetailsService，然后通过这个 ClientDetailsService 创建了两个 ClientDetails，分别对应 app1 和 app2。请注意，这里指定的 authorizedGrantTypes为代表授权码模式的 “authorization_code”。</p>











<p data-nodeid="116055" class="">同时，我们还需要在 AuthorizationServerConfiguration 类中添加对 JWT 的相关设置：</p>




<pre class="lang-java" data-nodeid="3287"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthorizationServerEndpointsConfigurer endpoints)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; endpoints.accessTokenConverter(jwtAccessTokenConverter())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .tokenStore(jwtTokenStore());
}
&nbsp;
<span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> JwtTokenStore <span class="hljs-title">jwtTokenStore</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> JwtTokenStore(jwtAccessTokenConverter());
}
&nbsp;
<span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> JwtAccessTokenConverter <span class="hljs-title">jwtAccessTokenConverter</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JwtAccessTokenConverter jwtAccessTokenConverter = <span class="hljs-keyword">new</span> JwtAccessTokenConverter();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jwtAccessTokenConverter.setSigningKey(<span class="hljs-string">"123456"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> jwtAccessTokenConverter;
}
</code></pre>
<p data-nodeid="3288">这里使用的设置方法我们在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3430">《15</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3433"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3436">|</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3439"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3442">令牌扩展：如何使用</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3445"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3448">JWT</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3451"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3454">实现定制化</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3457"> </a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7709" data-nodeid="3460">Token？》</a>课时中都已经介绍过了，这里不再详细展开。</p>
<h3 data-nodeid="117311" class="">实现 SSO 客户端</h3>


<p data-nodeid="120451" class="">介绍完 SSO 服务器端配置，接下来，我们来讨论客户端的实现过程。在客户端中，我们同样创建一个继承了 WebSecurityConfigurerAdapter 的 WebSecurityConfiguration，用来设置认证和授权机制，如下所示：</p>





<pre class="lang-java" data-nodeid="3291"><code data-language="java"><span class="hljs-meta">@EnableOAuth2Sso</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebSecurityConfiguration</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(WebSecurity web)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.configure(web);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.logout()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizeRequests()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .anyRequest().authenticated()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .csrf().disable();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="121707" class="">这里唯一需要强调的就是 @EnableOAuth2Sso 注解，这是单点登录相关自动化配置的入口，定义如下：</p>


<pre class="lang-java" data-nodeid="3293"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@EnableOAuth2Client</span>
<span class="hljs-meta">@EnableConfigurationProperties(OAuth2SsoProperties.class)</span>
<span class="hljs-meta">@Import({ OAuth2SsoDefaultConfiguration.class, OAuth2SsoCustomConfiguration.class,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResourceServerTokenServicesConfiguration.class })</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> EnableOAuth2Sso {
&nbsp;
}
</code></pre>
<p data-nodeid="122963" class="">在 @EnableOAuth2Sso 注解上，我们找到了 @EnableOAuth2Client 注解，代表启用了 OAuth2Client 客户端。同时，OAuth2SsoDefaultConfiguration 和 OAuth2SsoCustomConfiguration 用来配置基于 OAuth2 的 SSO 行为，而在 ResourceServerTokenServicesConfiguration 中则配置了基于 JWT 来处理 Token 的相关操作。</p>


<p data-nodeid="125475" class="">接着，我们在 app1 客户端的 application.yml 配置文件中，添加如下配置项：</p>




<pre class="lang-xml" data-nodeid="3296"><code data-language="xml">server:
&nbsp; port: 8080
&nbsp; servlet:
&nbsp;&nbsp;&nbsp; context-path: /app1
</code></pre>
<p data-nodeid="128011" class="">这里用到了 server.servlet.context-path 配置项，用来设置应用的上下文路径，相当于为完整的URL地址添加了一个前缀。这样，原本访问“<a href="http://localhost:8080/login" data-nodeid="128015">http://localhost:8080/login</a>”的地址就会变成<a href="http://localhost:8080/app1/login" data-nodeid="128019">http://localhost:8080/app1/login</a>，这是使用 SSO 时的一个常见的技巧。</p>




<p data-nodeid="3298">然后，我们再在配置文件中添加如下配置项：</p>
<pre class="lang-xml" data-nodeid="3299"><code data-language="xml">security:
&nbsp; oauth2:
&nbsp;&nbsp;&nbsp; client:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; client-id: app1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; client-secret: app1_secret
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; access-token-uri: http://localhost:8888/oauth/token
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; user-authorization-uri: http://localhost:8888/oauth/authorize
&nbsp;&nbsp;&nbsp; resource:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jwt:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; key-uri: http://localhost:8888/oauth/token_key
</code></pre>
<p data-nodeid="131787" class="">这些配置项是针对 OAuth2 协议的专用配置项，我们看到了用于设置客户端信息的“client”配置段，除了客户端Id和密码之外，还指定了用于获取 token 的“access-token-uri”地址以及执行授权的“user-authorization-uri”地址，这些都应该指向前面已经创建的 SSO 服务器地址。</p>






<p data-nodeid="133043" class="">另一方面，一旦在配置文件中添加了“security.oauth2.resource.jwt”配置项，对Token的校验就会使用 JwtTokenStore了，这样就能跟SSO服务器端所创建的 JwtTokenStore 进行对应。</p>


<p data-nodeid="136283" class="">到目前为止，我们已经创建了一个 SSO 客户端应用 app1，而创建 app2 的过程是完全一样的，这里不再展开。完整的代码你可以参考<a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136287">https://github.com/lagou</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136290">E</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136293">d</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136296">Anna</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136299">/SpringSecurity-jianxiang</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136302">/</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136305">t</a><a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSsoDemo" data-nodeid="136308">ree/main/SpringSsoDemo</a>。</p>





<h3 data-nodeid="3303">案例演示</h3>
<p data-nodeid="181067" class="">最后，让我们演示一下整个单点登录过程。依次启动 SSO 服务器以及 app1 和 app2，然后在浏览器中访问 app1 地址<a href="http://localhost:8080/app1/system/profile" data-nodeid="181071">http://localhost:8080/app1/system/profile</a>，这时候浏览器就会重定向到 SSO 服务器登录页面。</p>









<p data-nodeid="143268" class="">请注意，如果我们在访问上述地址时打开了浏览器的“网络”标签并查看其访问路径，就可以看到确实是先跳转到了app1的登录页面（<a href="http://localhost:8080/app1/login" data-nodeid="143272">http://localhost:8080/app1/login</a>），然后又重定向到 SSO 服务器。由于用户处于未登录状态，所以最后又重定向到 SSO 服务器的登录界面（<a href="http://localhost:8888/login" data-nodeid="143276">http://localhost:8888/login</a>），整个请求的跳转过程如下图所示：</p>




<p data-nodeid="8529"><img src="https://s0.lgstatic.com/i/image6/M00/4E/D6/Cgp9HWD2g5SAV9ZCAACNhBC8lAY290.png" alt="17-2.png" data-nodeid="8533"></p>
<div data-nodeid="144531" class=""><p style="text-align:center">未登录状态访问 app1 时的网络请求跳转流程图</p></div>









<p data-nodeid="145786" class="">我们在 SSO 服务器的登录界面输入正确的用户名和密码之后就可以认证成功了，这时候我们再看网络请求的过程，如下所示：</p>


<p data-nodeid="11364"><img src="https://s0.lgstatic.com/i/image6/M00/4E/D6/Cgp9HWD2g4OAY_pLAACZ0h8qrkQ895.png" alt="17-3png.png" data-nodeid="11368"></p>


<div data-nodeid="147041" class=""><p style="text-align:center">登录 app1 过程的网络请求跳转流程图</p></div>









<p data-nodeid="182334" class="te-preview-highlight">可以看到，在成功登录之后，授权系统重定向到 app1 中配置的回调地址（<a href="http://localhost:8080/app1/login" data-nodeid="182338">http://localhost:8080/app1/logi</a><a href="http://localhost:8080/app1/login" data-nodeid="182341">n</a>）。与此同时，我们在请求地址中还发现了两个新的参数 code 和 state。app1 客户端就会根据这个 code 来访问 SSO 服务器的/oauth/token 接口来申请 token。申请成功后，重定向到 app1 配置的回调地址。</p>















<p data-nodeid="169732" class="">现在，如果你访问 app2，与第一次访问 app1 相同，浏览器先重定向到 app2 的登录页面，然后又重定向到 SSO 服务器的授权链接，最后直接就重新重定向到 app2 的登录页面。不同之处在于，此次访问并不需要再次重定向到 SSO 服务器进行登录，而是成功访问 SSO 服务器的授权接口，并携带着 code 重定向到 app2 的回调路径。然后 app2 根据 code 再次访问 /oauth/token 接口拿到 token，这样就可以正常访问受保护的资源了。</p>
























<h3 data-nodeid="3311">小结与预告</h3>
<p data-nodeid="172892" class="">本课时是相对独立的一部分内容，针对日常开发过程中经常碰到的<strong data-nodeid="172898">单点登录场景</strong>做了案例的设计和实现。我们可以把各个独立的系统看成一个个客户端，然后基于 OAuth2 协议来实现 SSO。此外，本课时还对如何构建 SSO 服务器端和客户端组件，以及两者之间的交互过程进行了详细的介绍。</p>





<p data-nodeid="175409" class="">这里给你留一道思考题：你能结合 OAuth2 协议描述 SSO 的整体工作流程吗？</p>




<p data-nodeid="177921" class="">介绍完 OAuth2 协议以及应用场景之后，我们将引入一个全新的话题，既响应式编程，这是一种技术趋势。下一课时我们将讨论如何为 Spring Security 添加响应式编程特性。</p>

---

### 精选评论


