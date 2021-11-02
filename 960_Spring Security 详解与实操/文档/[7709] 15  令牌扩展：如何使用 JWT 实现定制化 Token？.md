<p data-nodeid="5313" class="">上一讲我们详细介绍了在微服务架构中如何使用 Token 对微服务的访问过程进行权限控制，这里的 Token 是类似“b7c2c7e0-0223-40e2-911d-eff82d125b80”的一种字符串结构。显然，这种格式的 Token 包含的内容应该是很有限的，那么是否有办法实现更为丰富的 Token 呢？答案是肯定的。</p>
<p data-nodeid="5314">事实上，在 OAuth2 协议中并没有明确规定 Token 具体的组成结构，而在现实应用中，我也不太建议你使用上一讲中我们用到的 Token 格式，而是更倾向于采用 JWT。今天我们就基于 JWT 讨论如何实现定制化 Token 这一话题。</p>
<h3 data-nodeid="5315">什么是 JWT？</h3>
<p data-nodeid="5316">JWT 的全称是 JSON Web Token，所以它本质上是一种基于 JSON 表示的 Token。<strong data-nodeid="5385">JWT 的设计目标就是为 OAuth2 协议中使用的 Token 提供一种标准结构，所以它经常与 OAuth2 协议集成在一起使用</strong>。</p>
<h4 data-nodeid="5317">JWT 的基本结构</h4>
<p data-nodeid="5318">从结构上讲，JWT 本身由三段信息构成：第一段为头部（Header），第二段为有效负载（Payload），第三段为签名（Signature）。如下所示：</p>
<pre class="lang-xml" data-nodeid="5319"><code data-language="xml">header. payload. signature
</code></pre>
<p data-nodeid="5320">从数据格式上来看，以上三个部分的内容都是一个 JSON 对象。在 JWT 中，每一段 JSON 对象都用 Base64 进行编码，编码后的内容用“.”号连接一起，所以 JWT 本质上就是一个字符串。如下所示即为一个 JWT 字符串的示例：</p>
<pre class="lang-xml" data-nodeid="5321"><code data-language="xml">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL3NwcmluZy5leGFtcGxlLmNvbSIsInN1YiI6Im1haWx0bzpzcHJpbmdAZXhhbXBsZS5jb20iLCJuYmYiOjE2MTU4MTg2NDYsImV4cCI6MTYxNTgyMjI0NiwiaWF0IjoxNjE1ODE4NjQ2LCJqdGkiOiJpZDEyMzQ1NiIsInR5cCI6Imh0dHBzOi8vc3ByaW5nLmV4YW1wbGUuY29tL3JlZ2lzdGVyIn0.Nweh3OPKl-p0PrSNDUQZ9LkJVWxjAP76uQscYJFQr9w
</code></pre>
<p data-nodeid="5322">显然，我们无法从这个经过 Base64 编码的字符串中获取任何有用的信息。业界也存在一些在线生成和解析 JWT 的工具，针对上面这个 JWT 字符串，我们可以通过这些工具获取其包含的原始 JSON 数据，如下所示：</p>
<pre class="lang-xml" data-nodeid="5323"><code data-language="xml">{
&nbsp;alg: "HS256",
&nbsp;typ: "JWT"
}.
{
&nbsp;iss: "https://spring.example.com",
&nbsp;sub: "mailto:spring@example.com",
&nbsp;nbf: 1615818646,
&nbsp;exp: 1615822246,
&nbsp;iat: 1615818646,
&nbsp;jti: "id123456",
&nbsp;typ: "https://spring.example.com/register"
}.
[signature]
</code></pre>
<p data-nodeid="5324">我们可以清晰地看到一个 JWT 中包含的 Header 部分和 Payload 部分的数据，出于安全考虑，JWT 解析工具通常都不会展示 Signature 部分数据。</p>
<h4 data-nodeid="5325">JWT 的优势</h4>
<p data-nodeid="5326">JWT 具有很多优秀的功能特性，它的数据表示方式采用语言无关的 JSON 格式，可以与各个异构系统进行集成。同时，JWT 是一种表示数据的标准，所有人都可以遵循这种标准来传递数据。</p>
<p data-nodeid="5327">在安全领域，我们通常用它<strong data-nodeid="5398">传递被认证的用户身份信息</strong>，以便从资源服务器获取资源。同时，JWT 在结构上也提供了良好的扩展性，开发人员可以根据需求增加一些额外信息用于处理复杂的业务逻辑。因为 JWT 中的数据都是被加密的，所以它除了可以直接用于认证，也可以处理加密需求。</p>
<h3 data-nodeid="5328">如何集成 OAuth2 与 JWT？</h3>
<p data-nodeid="5329">看到这里，可能你已经认识到了JWT 和 OAuth2 面向的是不同的应用场景，本身并没有任何关联。但在很多情况下，我们讨论 OAuth2 的实现时，会把 JWT 作为一种认证机制进行使用。</p>
<p data-nodeid="5330">Spring Security 为 JWT 的生成和验证提供了开箱即用的支持。当然，想要发送和消费 JWT，OAuth2 授权服务和各个受保护的微服务必须以不同的方式进行配置。整个开发流程与上一讲介绍的生成普通 Token 是一致的，不同之处在于配置的内容和方式。接下来，我们来看如何在 OAuth2 授权服务器中配置 JWT。</p>
<p data-nodeid="5331">对于所有需要用到 JWT 的独立服务来说，首先我们需要<strong data-nodeid="5407">在 Maven 的 pom 文件中添加对应的依赖包</strong>，如下所示：</p>
<pre class="lang-xml" data-nodeid="5332"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.security<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-security-jwt<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="5333">下一步就是提供一个配置类用于完成 JWT 的生成和转换。事实上，在 OAuth2 协议中专门提供了一个接口用于管理 Token 的存储，这个接口就是 TokenStore，而该接口的实现类 JwtTokenStore 则专门用来存储 JWT Token。对应的，我们也将创建一个用于配置 JwtTokenStore 的配置类 JWTTokenStoreConfig，如下所示：</p>
<pre class="lang-xml" data-nodeid="5334"><code data-language="xml">@Configuration
public class JWTTokenStoreConfig {

@Bean
public TokenStore tokenStore() {
return new JwtTokenStore(jwtAccessTokenConverter());
}

@Bean
public JwtAccessTokenConverter jwtAccessTokenConverter() {
JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
converter.setSigningKey("123456");
return converter;
}

@Bean
public DefaultTokenServices tokenServices() {
DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
defaultTokenServices.setTokenStore(tokenStore());
defaultTokenServices.setSupportRefreshToken(true);
return defaultTokenServices;
}
}
</code></pre>
<p data-nodeid="5335">可以看到，这里构建了 JwtTokenStore 对象，而在它的构造函数中传入了一个 JwtAccessTokenConverter。JwtAccessTokenConverters 是一个用来转换 JWT 的转换器，转换的过程需要签名键。创建 JwtTokenStore 后，我们通过 tokenServices 方法返回了已经设置 JwtTokenStore 对象的 DefaultTokenServices。</p>
<p data-nodeid="5336">上述 JWTTokenStoreConfig 的作用就是创建了一系列对象以供 Spring 容器使用，我们什么时候会用到这些对象呢？答案就是在将 JWT 集成到 OAuth2 授权服务的过程中，而这个过程似曾相似。基于 13 讲“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7707" data-nodeid="5413">授权体系：如何构建 OAuth2 授权服务器？</a>”中的讨论，我们可以构建一个配置类来覆写 AuthorizationServerConfigurerAdapter 中的 configure 方法，回想原先的这个configure 方法实现如下：</p>
<pre class="lang-java" data-nodeid="5337"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthorizationServerEndpointsConfigurer endpoints)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; endpoints.authenticationManager(authenticationManager)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .userDetailsService(userDetailsService);
}
</code></pre>
<p data-nodeid="5338">集成 JWT 之后，该方法的实现过程则需要调整，如下所示：</p>
<pre class="lang-java" data-nodeid="5339"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthorizationServerEndpointsConfigurer endpoints)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TokenEnhancerChain tokenEnhancerChain = <span class="hljs-keyword">new</span> TokenEnhancerChain();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;       tokenEnhancerChain.setTokenEnhancers(Arrays.asList(jwtTokenEnhancer, jwtAccessTokenConverter));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; endpoints.tokenStore(tokenStore).accessTokenConverter(jwtAccessTokenConverter).tokenEnhancer(tokenEnhancerChain)
        .authenticationManager(authenticationManager)
        .userDetailsService(userDetailsService);
}
</code></pre>
<p data-nodeid="5340">可以看到，这里构建了一个针对 Token 的增强链 TokenEnhancerChain，并用到了在 JWTTokenStoreConfig 中创建的 tokenStore、jwtAccessTokenConverter 对象。至此，我们在 OAuth2 协议中集成 JWT 的过程就介绍完了，也就是说现在我们访问 OAuth2 授权服务器时获取的 Token 应该就是 JWT Token。</p>
<p data-nodeid="5341">我们来尝试一下，通过 Postman，我们发起了请求并得到了相应的 Token：</p>
<pre class="lang-xml" data-nodeid="5342"><code data-language="xml">{
&nbsp;&nbsp;&nbsp; "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzeXN0ZW0iOiJTcHJpbmcgU3lzdGVtIiwidXNlcl9uYW1lIjoic3ByaW5nX3VzZXIiLCJzY29wZSI6WyJ3ZWJjbGllbnQiXSwiZXhwIjoxNjE3NTYwODU0LCJhdXRob3JpdGllcyI6WyJST0xFX1VTRVIiXSwianRpIjoiY2UyYTgzZmYtMjMzMC00YmQ1LTk4MzUtOWIyYzE0N2Y2MTcyIiwiY2xpZW50X2lkIjoic3ByaW5nIn0.Cd_x3r-Fi9hudA2W80amLEga0utPiOJCgBxxLI4Lsb8",
&nbsp;&nbsp;&nbsp; "token_type": "bearer",
&nbsp;&nbsp;&nbsp; "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzeXN0ZW0iOiJTcHJpbmcgU3lzdGVtIiwidXNlcl9uYW1lIjoic3ByaW5nX3VzZXIiLCJzY29wZSI6WyJ3ZWJjbGllbnQiXSwiYXRpIjoiY2UyYTgzZmYtMjMzMC00YmQ1LTk4MzUtOWIyYzE0N2Y2MTcyIiwiZXhwIjoxNjIwMTA5NjU0LCJhdXRob3JpdGllcyI6WyJST0xFX1VTRVIiXSwianRpIjoiMDA0NjIxY2MtMmRmZi00ZDJiLWE0YWUtNTU5MzM5YzkyYmFhIiwiY2xpZW50X2lkIjoic3ByaW5nIn0.xDhGwhNTq7Iun9yLENaCvh8mrVHkabu3J8sP0NXENq0",
&nbsp;&nbsp;&nbsp; "expires_in": 43199,
&nbsp;&nbsp;&nbsp; "scope": "webclient",
&nbsp;&nbsp;&nbsp; "system": "Spring System",
&nbsp;&nbsp;&nbsp; "jti": "ce2a83ff-2330-4bd5-9835-9b2c147f6172"
}
</code></pre>
<p data-nodeid="5343">显然，这里的 access_token 和 refresh_token 已经是经过 Base64 编码的字符串。同样，我们可以通过在线工具来解析这个 JSON 数据格式的内容，如下所示的就是 access_token 的原始内容：</p>
<pre class="lang-xml" data-nodeid="5344"><code data-language="xml">{
&nbsp;alg: "HS256",
&nbsp;typ: "JWT"
}.
{
&nbsp;system: "Spring System",
&nbsp;user_name: "spring_user",
&nbsp;scope: [
&nbsp; "webclient"
&nbsp;],
&nbsp;exp: 1617560854,
&nbsp;authorities: [
&nbsp; "ROLE_USER"
&nbsp;],
&nbsp;jti: "ce2a83ff-2330-4bd5-9835-9b2c147f6172",
&nbsp;client_id: "spring"
}.
[signature]
</code></pre>
<h3 data-nodeid="5345">如何在微服务中使用 JWT？</h3>
<p data-nodeid="5346">在微服务中使用 JWT 的第一步也是配置工作。我们需要在各个微服务中添加一个 WTTokenStoreConfig 配置类，这个配置类的内容就是创建一个 JwtTokenStore 并构建 tokenServices，具体代码在前面已经做了介绍，这里不再展开。</p>
<p data-nodeid="5347">配置工作完成后，剩下的问题就是在服务调用链中传播 JWT。在上一讲中，我们给出了 OAuth2RestTemplate 这个工具类，该类可以传播普通的 Token。可惜的是，它并不能传播基于 JWT 的 Token。从实现原理上，OAuth2RestTemplate 也是在 RestTemplate 的基础上做了一层封装，所以我们的思路也是尝试<strong data-nodeid="5432">在 RestTemplate 请求中添加对 JWT 的支持</strong>。</p>
<ul data-nodeid="5348">
<li data-nodeid="5349">
<p data-nodeid="5350">我们知道， HTTP 请求是通过在 Header 部分中添加一个“Authorization”消息头来完成对 Token 的传递，所以第一步需要能够从 HTTP 请求中获取这个 JWT Token。</p>
</li>
<li data-nodeid="5351">
<p data-nodeid="5352">然后第二步我们需要将这个 Token 存储在一个线程安全的地方，以便在后续的服务链中进行使用。</p>
</li>
<li data-nodeid="5353">
<p data-nodeid="5354">第三步，也是最关键的一步，就是在通过 RestTemplate 发起请求时，能够把这个 Token 自动嵌入到所发起的每一个 HTTP 请求中。</p>
</li>
</ul>
<p data-nodeid="5355">整个实现思路如下图所示：</p>
<p data-nodeid="5796" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/4D/B7/Cgp9HWDwBeGACDQUAAFfi6HuFQU869.jpg" alt="1.png" data-nodeid="5799"><br>
在服务调用链中传播 JWT Token 的三个实现步骤</p>



<p data-nodeid="5358">实现这一思路需要你对 HTTP 请求的过程和原理有一定的理解，在代码实现上也需要有一些技巧，下面我一一展开。</p>
<p data-nodeid="5359">首先，在 HTTP 请求过程中，我们可以通过过滤器 Filter 对所有请求进行过滤。Filter 是 Servlet 中的一个核心组件，其基本原理就是构建一个过滤器链并对经过该过滤器链的请求和响应添加定制化的处理机制。Filter 接口的定义如下所示：</p>
<pre class="lang-java" data-nodeid="5360"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Filter</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(FilterConfig filterConfig)</span> <span class="hljs-keyword">throws</span> ServletException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilter</span><span class="hljs-params">(ServletRequest request, ServletResponse response, FilterChain chain)</span> <span class="hljs-keyword">throws</span> IOException, ServletException</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">destroy</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="5361">通常，我们会实现 Filter 接口中的 doFilter 方法。关于过滤器的详细内容，你可以结合 08 讲“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7702" data-nodeid="5446">管道过滤：如何基于 Spring Security 过滤器扩展安全性？</a>”做一下回顾。基于过滤器，我们可以将 ServletRequest 转化为一个 HttpServletRequest 对象，并从该对象中获取“Authorization”消息头，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="5362"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorizationHeaderFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilter</span><span class="hljs-params">(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> IOException, ServletException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthorizationHeaderHolder.getAuthorizationHeader().setAuthorizationHeader(httpServletRequest.getHeader(AuthorizationHeader.AUTHORIZATION_HEADER));

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; filterChain.doFilter(httpServletRequest, servletResponse);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(FilterConfig filterConfig)</span> <span class="hljs-keyword">throws</span> ServletException </span>{}
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">destroy</span><span class="hljs-params">()</span> </span>{}
}
</code></pre>
<p data-nodeid="5363">请注意，这里我们<strong data-nodeid="5453">把从 HTTP 请求中获取的“Authorization”消息头保存到了一个 AuthorizationHeaderHolder 对象中</strong>。从命名上看，AuthorizationHeader 对象代表的就是 HTTP 中“Authorization” 消息头，而 AuthorizationHeaderHolder 是该消息头对象的持有者。这种命名方式在 Spring 等主流开源框架中非常常见。</p>
<p data-nodeid="5364">一般而言，以 -Holder 结尾的多是一种封装类，用于对原有对象添加线程安全等附加特性。这里的 AuthorizationHeaderHolder 就是这样一个封装类，如下所示：</p>
<pre class="lang-java" data-nodeid="5365"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorizationHeaderHolder</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;AuthorizationHeader&gt; authorizationHeaderContext = <span class="hljs-keyword">new</span> ThreadLocal&lt;AuthorizationHeader&gt;();
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> AuthorizationHeader <span class="hljs-title">getAuthorizationHeader</span><span class="hljs-params">()</span></span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthorizationHeader header = authorizationHeaderContext.get();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (header == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  header = <span class="hljs-keyword">new</span> AuthorizationHeader();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; authorizationHeaderContext.set(header);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> authorizationHeaderContext.get();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setAuthorizationHeader</span><span class="hljs-params">(AuthorizationHeader header)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; authorizationHeaderContext.set(header);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="5366">可以看到，这里使用了 ThreadLocal 确保对 AuthorizationHeader 对象访问的线程安全性，AuthorizationHeader 定义如下，用于保存来自 HTTP 请求头的 JWT Token：</p>
<pre class="lang-java" data-nodeid="5367"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorizationHeader</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String AUTHORIZATION_HEADER = <span class="hljs-string">"Authorization"</span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String authorizationHeader = <span class="hljs-keyword">new</span> String();
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getAuthorizationHeader</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> authorizationHeader;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setAuthorizationHeader</span><span class="hljs-params">(String authorizationHeader)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.authorizationHeader = authorizationHeader;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="5368">现在，对于每一个 HTTP 请求，我们都能获取其中的 Token 并将其保存在上下文对象中。剩下的唯一问题就是如何通过 RestTemplate 将这个 Token 继续传递到下一个服务中，以便下一个服务也能从 HTTP 请求中获取 Token 并继续向后传递，从而确保 Token 在整个调用链中持续传播。要想实现这一目标，我们需要对 RestTemplate 进行一些设置，如下所示：</p>
<pre class="lang-java" data-nodeid="5369"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> RestTemplate <span class="hljs-title">getCustomRestTemplate</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RestTemplate template = <span class="hljs-keyword">new</span> RestTemplate();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;ClientHttpRequestInterceptor&gt; interceptors = template.getInterceptors();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (interceptors == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; template.setInterceptors(Collections.singletonList(<span class="hljs-keyword">new</span> AuthorizationHeaderInterceptor()));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; interceptors.add(<span class="hljs-keyword">new</span> AuthorizationHeaderInterceptor());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; template.setInterceptors(interceptors);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> template;
}
</code></pre>
<p data-nodeid="5370">RestTemplate 允许开发人员添加自定义的拦截器 Interceptor，拦截器本质上与过滤器的功能类似，用于对传入的 HTTP 请求进行定制化处理。例如，上述代码中的 AuthorizationHeaderInterceptor 的作用就是在 HTTP 请求的消息头中嵌入保存在 AuthorizationHeaderHolder 中的 JWT Token，如下所示：</p>
<pre class="lang-java" data-nodeid="5371"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorizationHeaderInterceptor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ClientHttpRequestInterceptor</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> ClientHttpResponse <span class="hljs-title">intercept</span><span class="hljs-params">(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpRequest request, <span class="hljs-keyword">byte</span>[] body, ClientHttpRequestExecution execution)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpHeaders headers = request.getHeaders();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; headers.add(AuthorizationHeader.AUTHORIZATION_HEADER, AuthorizationHeaderHolder.getAuthorizationHeader().getAuthorizationHeader());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> execution.execute(request, body);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="5372">至此，在微服务中使用 JWT 的方法已经介绍完毕。关于 JWT 还有一部分内容我们没有介绍，即如何扩展 JWT 中所持有的数据结构，我们会在接下来的案例系统中结合具体的业务场景对这块内容进行补充。</p>
<h3 data-nodeid="5373">小结与预告</h3>
<p data-nodeid="5374">这是介绍微服务安全性知识体系的最后一个课时，关注的是<strong data-nodeid="5465">认证问题而不是授权问题</strong>，为此我们引入了 JWT 机制。JWT 本质上也是一种 Token，只不过提供了标准化的规范定义，可以与 OAuth2 协议进行集成。我们使用 JWT 时，也可以将各种信息添加到这种 Token 中，并在微服务访问链路中进行传播。</p>
<p data-nodeid="5375">这里给你留一道思考题：如果想要确保 JWT 在各个微服务中进行有效传播，需要怎么做？</p>
<p data-nodeid="5376" class="">介绍完 JWT，下一讲又会介绍一个新的案例。我们将基于 Spring Security 和 Spring Cloud 构建一个 SpringOAuth2 案例系统，并给出微服务架构中实现服务安全访问的详细过程。</p>

---

### 精选评论


