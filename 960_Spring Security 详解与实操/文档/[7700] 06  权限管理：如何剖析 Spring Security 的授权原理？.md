<p data-nodeid="146819">上一讲，我们分析了 Spring Security 中提供的授权功能。你可以发现使用这一功能的方法很简单，只需要基于 HttpSecurity 对象提供的一组工具方法就能实现复杂场景下的访问控制。但是，易于使用的功能往往内部实现都没有表面看起来那么简单，今天我就来和你一起深入分析授权功能背后的实现机制。针对授权功能，Spring Security 在实现过程中采用了很多优秀的设计理念和实现技巧，值得我们深入学习。</p>
<h3 data-nodeid="146820">Spring Security 授权整体流程</h3>
<p data-nodeid="146821">我们先来简单回顾一下上一讲的内容。我们知道在 Spring Security 中，实现对所有请求权限控制的配置方法只需要如下所示的一行代码：</p>
<pre class="lang-java" data-nodeid="146822"><code data-language="java">http.authorizeRequests();
</code></pre>
<p data-nodeid="146823">我们可以结合 HTTP 请求的响应流程来理解这行代码的执行效果。当一个 HTTP 请求来到 Servlet 容器时，会被容器拦截，并添加一些附加的处理逻辑。在 Servlet 中，这种处理逻辑就是通过过滤器（Filter）来实现的，多个过滤器按照一定的顺序组合在一起就构成了一个过滤器链。关于过滤器的详细讨论我们会在 08 讲“管道过滤：如何基于 Spring Security 过滤器扩展安全性？”中展开，在本讲中，我们只需要知道 Spring Security 同样也<strong data-nodeid="146907">基于过滤器拦截请求</strong>，从而实现对访问权限的限制即可。</p>
<p data-nodeid="146824">在 Spring Security 中，存在一个叫 FilterSecurityInterceptor 的拦截器，它位于整个过滤器链的末端，核心功能是<strong data-nodeid="146917">对权限控制过程进行拦截</strong>，以此判定该请求是否能够访问目标 HTTP 端点。FilterSecurityInterceptor 是整个权限控制的第一个环节，我们把它称为<strong data-nodeid="146918">拦截请求</strong>。</p>
<p data-nodeid="146825">我们对请求进行拦截之后，下一步就要获取该请求的访问资源，以及访问这些资源需要的权限信息。我们把这一步骤称为<strong data-nodeid="146924">获取权限配置</strong>。在 Spring Security 中，存在一个 SecurityMetadataSource 接口，该接口保存着一系列安全元数据的数据源，代表权限配置的抽象。我们在上一讲中已经通过配置方法设置了很多权限信息，例如：</p>
<pre class="lang-java" data-nodeid="146826"><code data-language="java">http.authorizeRequests().anyRequest().hasAuthority(<span class="hljs-string">"CREATE"</span>);&nbsp; 
</code></pre>
<p data-nodeid="146827">请注意，http.authorizeRequests() 方法的返回值是一个 ExpressionInterceptUrlRegistry，anyRequest() 方法返回值是一个 AuthorizedUrl，而 hasAuthority() 方法返回的又是一个 ExpressionInterceptUrlRegistry。这些对象在今天的内容中都会介绍到。</p>
<p data-nodeid="146828">SecurityMetadataSource 接口定义了一组方法来操作这些权限配置，具体权限配置的表现形式是<strong data-nodeid="146931">ConfigAttribute 接口</strong>。通过 ExpressionInterceptUrlRegistry 和 AuthorizedUrl，我们能够把配置信息转变为具体的 ConfigAttribute。</p>
<p data-nodeid="146829">当我们获取了权限配置信息后，就可以根据这些配置决定 HTTP 请求是否具有访问权限，也就是执行授权决策。Spring Security 专门提供了一个 AccessDecisionManager 接口完成该操作。而在 AccessDecisionManager 接口中，又把具体的决策过程委托给了 AccessDecisionVoter 接口。<strong data-nodeid="146937">AccessDecisionVoter 可以被认为是一种投票器，负责对授权决策进行表决</strong>。</p>
<p data-nodeid="146830">以上三个步骤构成了 Spring Security 的授权整体工作流程，可以用如下所示的时序图表示：</p>
<p data-nodeid="148104" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/35/CioPOWC99xWAASrVAABnJyNceVM498.png" alt="Drawing 0.png" data-nodeid="148108"></p>
<div data-nodeid="148105"><p style="text-align:center">Spring Security 的授权整体工作流程</p></div>



<p data-nodeid="146833">接下来，我们基于这张类图分别对拦截请求、获取权限配置、执行授权决策三个步骤逐一展开讲解。</p>
<h3 data-nodeid="146834">拦截请求</h3>
<p data-nodeid="146835">作为一种拦截器，FilterSecurityInterceptor 实现了对请求的拦截。我们先来看它的定义，如下所示：</p>
<pre class="lang-java" data-nodeid="146836"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FilterSecurityInterceptor</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractSecurityInterceptor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span>
</span></code></pre>
<p data-nodeid="146837">FilterSecurityInterceptor 实现了 Servlet 的 Filter 接口，所以本质上也是一种过滤器，并实现了 Filter 接口的 invoke 方法。在它的 invoke 方法中，FilterSecurityInterceptor 自身并没有执行任何特殊的操作，只是获取了 HTTP 请求并调用了基类 AbstractSecurityInterceptor 中的 beforeInvocation() 方法对请求进行拦截：</p>
<pre class="lang-java" data-nodeid="146838"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(FilterInvocation fi)</span> <span class="hljs-keyword">throws</span> IOException, ServletException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;InterceptorStatusToken token = <span class="hljs-keyword">super</span>.beforeInvocation(fi);
&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">super</span>.afterInvocation(token, <span class="hljs-keyword">null</span>);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
}
</code></pre>
<p data-nodeid="146839">AbstractSecurityInterceptor 中的 beforeInvocation() 方法非常长，我们把它裁剪之后，可以得到如下所示的主流程代码：</p>
<pre class="lang-java" data-nodeid="146840"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> InterceptorStatusToken <span class="hljs-title">beforeInvocation</span><span class="hljs-params">(Object object)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	    …
	    <span class="hljs-comment">//获取 ConfigAttribute 集合</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt; ConfigAttribute &gt; attributes = <span class="hljs-keyword">this</span>.obtainSecurityMetadataSource()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getAttributes(object);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取认证信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Authentication authenticated = authenticateIfRequired();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行授权</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.accessDecisionManager.decide(authenticated, object, attributes);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (AccessDeniedException accessDeniedException) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
}
</code></pre>
<p data-nodeid="146841">可以看到，上述操作从配置好的&nbsp;SecurityMetadataSource&nbsp;中获取当前请求所对应的&nbsp;ConfigAttribute，即权限信息。那么，这个 SecurityMetadataSource 又是怎么来的呢？</p>
<h3 data-nodeid="146842">获取访问策略</h3>
<p data-nodeid="146843">我们注意到在 FilterSecurityInterceptor 中定义了一个 FilterInvocationSecurityMetadataSource 变量，并通过一个 setSecurityMetadataSource() 方法进行注入，显然，这个变量就是一种 SecurityMetadataSource。</p>
<h4 data-nodeid="146844">MetadataSource</h4>
<p data-nodeid="146845">通过翻阅 FilterSecurityInterceptor 的调用关系，我们发现初始化该类的地方是在 AbstractInterceptUrlConfigurer 类中，如下所示：</p>
<pre class="lang-java" data-nodeid="146846"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> FilterSecurityInterceptor <span class="hljs-title">createFilterSecurityInterceptor</span><span class="hljs-params">(H http,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FilterInvocationSecurityMetadataSource metadataSource,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthenticationManager authenticationManager)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FilterSecurityInterceptor securityInterceptor = <span class="hljs-keyword">new</span> FilterSecurityInterceptor();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; securityInterceptor.setSecurityMetadataSource(metadataSource);
&nbsp;&nbsp;&nbsp;  securityInterceptor.setAccessDecisionManager(getAccessDecisionManager(http));
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; securityInterceptor.setAuthenticationManager(authenticationManager);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; securityInterceptor.afterPropertiesSet();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> securityInterceptor;
}
</code></pre>
<p data-nodeid="146847">而 FilterInvocationSecurityMetadataSource 对象的创建则是基于 AbstractInterceptUrlConfigurer 中提供的抽象方法：</p>
<pre class="lang-java" data-nodeid="146848"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">abstract</span> FilterInvocationSecurityMetadataSource <span class="hljs-title">createMetadataSource</span><span class="hljs-params">(H http)</span></span>;
</code></pre>
<p data-nodeid="146849">这个方法的实现过程由 AbstractInterceptUrlConfigurer 的子类 ExpressionUrlAuthorizationConfigurer 提供，如下所示：</p>
<pre class="lang-java" data-nodeid="146850"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function">ExpressionBasedFilterInvocationSecurityMetadataSource <span class="hljs-title">createMetadataSource</span><span class="hljs-params">(H http)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LinkedHashMap&lt;RequestMatcher, Collection&lt;ConfigAttribute&gt;&gt; requestMap = REGISTRY.createRequestMap();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ExpressionBasedFilterInvocationSecurityMetadataSource(requestMap,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getExpressionHandler(http));
}
</code></pre>
<p data-nodeid="146851">请你注意：这里有个<strong data-nodeid="146958">REGISTRY 对象</strong>，它的类型是 ExpressionInterceptUrlRegistry。这和前面介绍的内容相对应，我们在前面已经提到 http.authorizeRequests() 方法的返回值类型就是这个 ExpressionInterceptUrlRegistry。</p>
<h4 data-nodeid="146852">ExpressionInterceptUrlRegistry</h4>
<p data-nodeid="146853">我们继续看 ExpressionInterceptUrlRegistry 中 createRequestMap() 的实现过程，如下所示：</p>
<pre class="lang-java" data-nodeid="146854"><code data-language="java"><span class="hljs-keyword">final</span> LinkedHashMap&lt;RequestMatcher, Collection&lt;ConfigAttribute&gt;&gt; createRequestMap() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LinkedHashMap&lt;RequestMatcher, Collection&lt;ConfigAttribute&gt;&gt; requestMap = <span class="hljs-keyword">new</span> LinkedHashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (UrlMapping mapping : getUrlMappings()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RequestMatcher matcher = mapping.getRequestMatcher();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt;ConfigAttribute&gt; configAttrs = mapping.getConfigAttrs();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; requestMap.put(matcher, configAttrs);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> requestMap;
}
</code></pre>
<p data-nodeid="148477" class="">这段代码把配置的 http.authorizeRequests() 转化为 UrlMappings，然后进一步转换为  RequestMatcher 与 Collection<code data-backticks="1" data-nodeid="148479">&lt;ConfigAttribute&gt;</code> 之间的映射关系。那么，创建这些 UrlMappings 的入口又在哪里呢？同样也是在 ExpressionUrlAuthorizationConfigurer 中的 interceptUrl 方法，如下所示：</p>

<pre class="lang-java" data-nodeid="146856"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">interceptUrl</span><span class="hljs-params">(Iterable&lt;? extends RequestMatcher&gt; requestMatchers,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt;ConfigAttribute&gt; configAttributes)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (RequestMatcher requestMatcher : requestMatchers) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; REGISTRY.addMapping(<span class="hljs-keyword">new</span> AbstractConfigAttributeRequestMatcherRegistry.UrlMapping(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; requestMatcher, configAttributes));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<h4 data-nodeid="146857">AuthorizedUrl</h4>
<p data-nodeid="146858">我们进一步跟踪代码的运行流程，发现上述 interceptUrl() 方法的调用入口是在如下所示的 access() 方法中：</p>
<pre class="lang-java" data-nodeid="146859"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ExpressionInterceptUrlRegistry <span class="hljs-title">access</span><span class="hljs-params">(String attribute)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (not) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; attribute = <span class="hljs-string">"!"</span> + attribute;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; interceptUrl(requestMatchers, SecurityConfig.createList(attribute));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ExpressionUrlAuthorizationConfigurer.<span class="hljs-keyword">this</span>.REGISTRY;
}
</code></pre>
<p data-nodeid="146860">结合上一讲的内容，我们不难理解这个 access() 方法的作用。请注意，这个方法位于 AuthorizedUrl 类中，而我们执行 http.authorizeRequests().anyRequest() 方法的返回值就是这个 AuthorizedUrl。在该类中定义了一批我们已经熟悉的配置方法，例如 hasRole、hasAuthority 等，而这些方法在内部都是调用了上面这个 access() 方法：</p>
<pre class="lang-java" data-nodeid="146861"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ExpressionInterceptUrlRegistry <span class="hljs-title">hasRole</span><span class="hljs-params">(String role)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> access(ExpressionUrlAuthorizationConfigurer.hasRole(role));
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> ExpressionInterceptUrlRegistry <span class="hljs-title">hasAuthority</span><span class="hljs-params">(String authority)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> access(ExpressionUrlAuthorizationConfigurer.hasAuthority(authority));
}
</code></pre>
<p data-nodeid="146862">讲到这里，获取访问策略的流程就基本完成了，我们得到了一组代表权限的 ConfigAttribute 对象。</p>
<h3 data-nodeid="146863">执行授权决策</h3>
<p data-nodeid="146864">执行授权决策的前提是<strong data-nodeid="146974">获取认证信息</strong>，因此，我们在 FilterSecurityInterceptor 的拦截流程中发现了如下一行执行认证操作的代码：</p>
<pre class="lang-java" data-nodeid="146865"><code data-language="java">Authentication authenticated = authenticateIfRequired();
</code></pre>
<p data-nodeid="146866">这里的 authenticateIfRequired() 方法执行认证操作，该方法实现如下：</p>
<pre class="lang-java" data-nodeid="146867"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Authentication <span class="hljs-title">authenticateIfRequired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; …
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; authentication = authenticationManager.authenticate(authentication);
	    …
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; SecurityContextHolder.getContext().setAuthentication(authentication);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> authentication;
}
</code></pre>
<p data-nodeid="146868">可以看到认证逻辑并不复杂，首先根据上下文对象中是否存在 Authentication 对象来判断当前用户是否已通过认证。如果尚未通过身份认证，则调用 AuthenticationManager 进行认证，并把 Authentication 存储到上下文对象中。关于用户认证流程的详细介绍你可以回顾“认证体系：如何深入理解 Spring Security 的认证机制？”中的内容。</p>
<h4 data-nodeid="146869">AccessDecisionManager</h4>
<p data-nodeid="146870">AccessDecisionManager 是用来进行授权决策的入口，其最核心的方法就是如下所示的 decide() 方法，前面我们已经看到了这个方法的执行过程：</p>
<pre class="lang-java" data-nodeid="146871"><code data-language="java"><span class="hljs-keyword">this</span>.accessDecisionManager.decide(authenticated, object, attributes);
</code></pre>
<p data-nodeid="146872">而在前面介绍 AbstractInterceptUrlConfigurer&nbsp;类时，我们同样发现了获取和创建 AccessDecisionManager 的对应方法：</p>
<pre class="lang-java" data-nodeid="146873"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> AccessDecisionManager <span class="hljs-title">getAccessDecisionManager</span><span class="hljs-params">(H http)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (accessDecisionManager == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accessDecisionManager = createDefaultAccessDecisionManager(http);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> accessDecisionManager;
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">private</span> AccessDecisionManager <span class="hljs-title">createDefaultAccessDecisionManager</span><span class="hljs-params">(H http)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AffirmativeBased result = <span class="hljs-keyword">new</span> AffirmativeBased(getDecisionVoters(http));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> postProcess(result);
}
</code></pre>
<p data-nodeid="146874">显然，如果没有设置自定义的 AccessDecisionManager，默认会创建一个&nbsp;AffirmativeBased&nbsp;实例。AffirmativeBased&nbsp;的 decide() 方法如下所示：</p>
<pre class="lang-java" data-nodeid="146875"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">decide</span><span class="hljs-params">(Authentication authentication, Object object, Collection&lt;ConfigAttribute&gt; configAttributes)</span> <span class="hljs-keyword">throws</span> AccessDeniedException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">int</span> deny = <span class="hljs-number">0</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (AccessDecisionVoter voter : getDecisionVoters()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">int</span> result = voter.vote(authentication, object, configAttributes);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">switch</span> (result) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> AccessDecisionVoter.ACCESS_GRANTED:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> AccessDecisionVoter.ACCESS_DENIED:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; deny++;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">break</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">default</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (deny &gt; <span class="hljs-number">0</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> AccessDeniedException(messages.getMessage(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"AbstractAccessDecisionManager.accessDenied"</span>, <span class="hljs-string">"Access is denied"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; checkAllowIfAllAbstainDecisions();
}
</code></pre>
<p data-nodeid="146876">可以看到，这里把真正计算是否具有访问权限的过程委托给了一组 AccessDecisionVoter 对象，只要其中有任意一个的结果是拒绝，就会抛出一个 AccessDeniedException。</p>
<h4 data-nodeid="146877">AccessDecisionVoter</h4>
<p data-nodeid="146878">AccessDecisionVoter 同样是一个接口，提供了如下所示的 vote() 方法：</p>
<pre class="lang-java" data-nodeid="146879"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">vote</span><span class="hljs-params">(Authentication authentication, S object,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collection&lt;ConfigAttribute&gt; attributes)</span></span>;
</code></pre>
<p data-nodeid="146880">我们再次在 AbstractInterceptUrlConfigurer 类中找到了获取 AccessDecisionVoter 的 getDecisionVoters() 抽象方法定义，如下所示：</p>
<pre class="lang-java" data-nodeid="146881"><code data-language="java"><span class="hljs-keyword">abstract</span> List&lt;AccessDecisionVoter&lt;?&gt;&gt; getDecisionVoters(H http);
</code></pre>
<p data-nodeid="146882">同样是在它的子类 ExpressionUrlAuthorizationConfigurer&nbsp;中，我们找到了这个抽象方法的具体实现：</p>
<pre class="lang-java" data-nodeid="146883"><code data-language="java"><span class="hljs-meta">@Override</span>
List&lt;AccessDecisionVoter&lt;?&gt;&gt; getDecisionVoters(H http) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;AccessDecisionVoter&lt;?&gt;&gt; decisionVoters = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; WebExpressionVoter expressionVoter = <span class="hljs-keyword">new</span> WebExpressionVoter();
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp; expressionVoter.setExpressionHandler(getExpressionHandler(http));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; decisionVoters.add(expressionVoter);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> decisionVoters;
}
</code></pre>
<p data-nodeid="146884">可以看到，这里创建的 AccessDecisionVoter 实际上都是 WebExpressionVoter，它的 vote() 方法如下所示：</p>
<pre class="lang-java" data-nodeid="146885"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">vote</span><span class="hljs-params">(Authentication authentication, FilterInvocation fi, Collection&lt;ConfigAttribute&gt; attributes)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; WebExpressionConfigAttribute weca = findConfigAttribute(attributes);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; EvaluationContext ctx = expressionHandler.createEvaluationContext(authentication, fi);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx = weca.postProcess(ctx, fi);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ExpressionUtils.evaluateAsBoolean(weca.getAuthorizeExpression(), ctx) ? ACCESS_GRANTED: ACCESS_DENIED;
}
</code></pre>
<p data-nodeid="146886">这里出现了一个 SecurityExpressionHandler，看类名就可以发现与 Spring 中的表达式语言相关，它会构建一个用于评估的上下文对象 EvaluationContext。而 ExpressionUtils.evaluateAsBoolean() 方法就是根据从 WebExpressionConfigAttribute 获取的授权表达式，以及这个 EvaluationContext 上下文对象完成最终结果的评估：</p>
<pre class="lang-java" data-nodeid="146887"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">evaluateAsBoolean</span><span class="hljs-params">(Expression expr, EvaluationContext ctx)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> expr.getValue(ctx, Boolean.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (EvaluationException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="146888">显然，最终的评估过程只是简单使用了 Spring 所提供的 SpEL 表达式语言。</p>
<p data-nodeid="146889">作为总结，我们把这一流程中涉及的核心组件以类图的形式进行了梳理，如下图所示：</p>
<p data-nodeid="149216" class=""><img src="https://s0.lgstatic.com/i/image6/M00/44/2D/Cgp9HWC99zqAGUOYAACtn7EjrO0914.png" alt="Drawing 1.png" data-nodeid="149220"></p>
<div data-nodeid="149217"><p style="text-align:center">Spring Security 授权相关核心类图</p></div>



<h3 data-nodeid="146892">小结与预告</h3>
<p data-nodeid="146893">这一讲我们关注的是 Spring Security 授权机制的实现原理，我们把整个授权过程拆分成<strong data-nodeid="146998">拦截请求、获取访问策略和执行授权决策</strong>这三大步骤。针对每一个步骤，都涉及了一组核心类及其它们之间的交互关系。针对这些核心类的讲解思路是围绕着上一讲介绍的基本配置方法展开讨论的，确保实际应用能与源码分析衔接在一起。</p>
<p data-nodeid="146894">本讲内容总结如下：</p>
<p data-nodeid="149591" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/44/35/CioPOWC990GAfLaXAACI8-bD1xs493.png" alt="Drawing 2.png" data-nodeid="149594"></p>

<p data-nodeid="146896">最后给你留一道思考题：在 Spring Security 中，你能简要描述整个授权机制的执行过程吗？</p>
<p data-nodeid="146897">介绍完授权机制的实现原理，我们关于 Spring Security 的基础功能讲解就接近尾声了。下一讲我们来设计并实现一个案例系统，基于目前已经讲解的内容来保护 Web 应用程序。</p>

---

### 精选评论


