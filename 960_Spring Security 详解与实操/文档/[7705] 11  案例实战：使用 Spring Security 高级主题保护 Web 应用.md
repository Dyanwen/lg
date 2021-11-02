<p data-nodeid="11691" class="">前面几讲我们系统介绍了 Spring Security 提供的一些高级功能，包括过滤器、CSRF 保护、CORS 以及全局方法，这些都是非常实用的功能特性。作为阶段性的总结，今天的内容将利用这些功能特性构建在安全领域中的一种典型的认证机制，即<strong data-nodeid="11772">多因素认证</strong>（Multi-Factor Authentication，MFA）机制。</p>
<h3 data-nodeid="11692">案例设计和初始化</h3>
<p data-nodeid="11693">在今天的案例中，我们构建多因素认证的思路并不是采用第三方成熟的解决方案，而是基于 Spring Security 的功能特性来自己设计并实现一个简单而完整的认证机制。</p>
<h4 data-nodeid="11694">多因素认证设计</h4>
<p data-nodeid="11695">多因素认证是一种安全访问控制的方法，基本的设计理念在于<strong data-nodeid="11781">用户想要访问最终的资源，至少需要通过两种以上的认证机制</strong>。</p>
<p data-nodeid="11696">那么，我们如何实现多种认证机制呢？一种常见的做法是分成两个步骤，第一步通过用户名和密码获取一个认证码（Authentication Code），第二步基于用户名和这个认证码进行安全访问。基于这种多因素认证的基本执行流程如下图所示：</p>
<p data-nodeid="12781" class=""><img src="https://s0.lgstatic.com/i/image6/M00/49/AD/Cgp9HWDcHxyASKleAAC5AmJ3Hwc285.png" alt="Drawing 0.png" data-nodeid="12784"></p>


<div data-nodeid="13219" class=""><p style="text-align:center">多因素认证的实现方式示意图</p></div>

<h4 data-nodeid="11700">系统初始化</h4>
<p data-nodeid="11701">为了实现多因素认证，我们需要构建一个独立的认证服务 Auth-Service，该服务<strong data-nodeid="11792">同时提供了基于用户名+密码以及用户名+认证码的认证形式</strong>。当然，实现认证的前提是构建用户体系，因此我们需要提供如下所示的 User 实体类：</p>
<pre class="lang-java" data-nodeid="11702"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GeneratedValue(strategy = GenerationType.IDENTITY)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Integer id;
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String username;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String password;
}
</code></pre>
<p data-nodeid="11703">可以看到，User 对象中包含了用户名 Username 和密码 Password 的定义。同样的，在如下所示的代表认证码的 AuthCode 对象中包含了用户名 Username 和具体的认证码 Code 字段的定义：</p>
<pre class="lang-java" data-nodeid="11704"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthCode</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GeneratedValue(strategy = GenerationType.IDENTITY)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Integer id;
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String username;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String code;&nbsp;&nbsp; 
}
</code></pre>
<p data-nodeid="11705">基于 User 和 AuthCode 实体对象，我们也给出创建数据库表的对应 SQL 定义，如下所示：</p>
<pre class="lang-xml" data-nodeid="11706"><code data-language="xml">CREATE TABLE IF NOT EXISTS `spring_security_demo`.`user` (
&nbsp;&nbsp;&nbsp; `id` INT NOT NULL AUTO_INCREMENT,
&nbsp;&nbsp;&nbsp; `username` VARCHAR(45) NULL,
&nbsp;&nbsp;&nbsp; `password` TEXT NULL,
&nbsp;&nbsp;&nbsp; PRIMARY KEY (`id`));
&nbsp;
CREATE TABLE IF NOT EXISTS `spring_security_demo`.`auth_code` (
&nbsp;&nbsp;&nbsp; `id` INT NOT NULL AUTO_INCREMENT,
&nbsp;&nbsp;&nbsp; `username` VARCHAR(45) NOT NULL,
&nbsp;&nbsp;&nbsp; `code` VARCHAR(45) NULL,
&nbsp;&nbsp;&nbsp; PRIMARY KEY (`id`));
</code></pre>
<p data-nodeid="11707">有了认证服务，接下来我们需要构建一个业务服务 Business-Service，该业务服务通过集成认证服务，完成具体的认证操作，并返回访问令牌（Token）给到客户端系统。因此，从依赖关系上讲，Business-Service 会调用 Auth-Service，如下图所示：</p>
<p data-nodeid="14083" class=""><img src="https://s0.lgstatic.com/i/image6/M00/49/AD/Cgp9HWDcHyuAT3auAACddvz56Cc067.png" alt="Drawing 2.png" data-nodeid="14087"></p>
<div data-nodeid="14084"><p style="text-align:center">Business-Service 调用 Auth-Service 关系图</p></div>




<p data-nodeid="11711">接下来，我们分别从这两个服务入手，实现多因素认证机制。</p>
<h3 data-nodeid="11712">实现多因素认证机制</h3>
<p data-nodeid="11713">对于多因素认证机制而言，<strong data-nodeid="11806">实现认证服务是基础</strong>，但难度并不大，我们往下看。</p>
<h4 data-nodeid="11714">实现认证服务</h4>
<p data-nodeid="11715">从表现形式上看，认证服务也是一个 Web 服务，所以内部需要<strong data-nodeid="11813">通过构建 Controller 层组件实现 HTTP 端点的暴露</strong>。为此，我们构建了如下所示的 AuthController：</p>
<pre class="lang-java" data-nodeid="11716"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthController</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserService userService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//添加User</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PostMapping("/user/add")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addUser</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> User user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userService.addUser(user);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过用户名+密码对用户进行首次认证</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PostMapping("/user/auth")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">auth</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> User user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userService.auth(user);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通过用户名+认证码进行二次认证</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PostMapping("/authcode/check")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">check</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> AuthCode authCode, HttpServletResponse response)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (userService.check(authCode)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.setStatus(HttpServletResponse.SC_OK);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.setStatus(HttpServletResponse.SC_FORBIDDEN);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11717">可以看到，这里除了一个添加用户信息的 HTTP 端点之外，我们分别实现了通过用户名+密码对用户进行首次认证的"/user/auth"端点，以及通过用户名+认证码进行二次认证的"/authcode/check"端点。</p>
<p data-nodeid="11718">这两个核心端点背后的实现逻辑都位于 UserService 中，我们先来看其中的 auth() 方法：</p>
<pre class="lang-java" data-nodeid="11719"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">auth</span><span class="hljs-params">(User user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Optional&lt;User&gt; o =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; userRepository.findUserByUsername(user.getUsername());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(o.isPresent()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; User u = o.get();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (passwordEncoder.matches(user.getPassword(), u.getPassword())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//生成或刷新认证码</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; generateOrRenewAutoCode(u);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BadCredentialsException(<span class="hljs-string">"Bad credentials."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BadCredentialsException(<span class="hljs-string">"Bad credentials."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11720">上述代码中的关键流程就是在<strong data-nodeid="11829">完成用户密码匹配之后的刷新认证码流程</strong>，负责实现该流程的 generateOrRenewAutoCode() 方法如下所示：</p>
<pre class="lang-java" data-nodeid="11721"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">generateOrRenewAutoCode</span> <span class="hljs-params">(User u)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String generatedCode = GenerateCodeUtil.generateCode();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Optional&lt;AuthCode&gt; autoCode = autoCodeRepository.findAuthCodeByUsername(u.getUsername());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (autoCode.isPresent()) {<span class="hljs-comment">//如果存在认证码，则刷新该认证码</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthCode code = autoCode.get();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; code.setCode(generatedCode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {<span class="hljs-comment">//如果没有找到认证码，则生成并保存一个新的认证码</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthCode code = <span class="hljs-keyword">new</span> AuthCode();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; code.setUsername(u.getUsername());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; code.setCode(generatedCode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; autoCodeRepository.save(code);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11722">上述方法的流程也很明确，首先通过调用工具类 GenerateCodeUtil 的 generateCode() 方法生成一个认证码，然后根据当前数据库中的状态决定是否对已有的认证码进行刷新，或者直接生成一个新的认证码并保存。因此，<strong data-nodeid="11835">每次调用 UserService 的 auth() 方法就相当于对用户的认证码进行了动态重置</strong>。</p>
<p data-nodeid="11723">一旦用户获取了认证码，并通过该认证码访问系统，认证服务就可以对该认证码进行校验，从而确定其是否有效。对认证码进行验证的方法如下所示：</p>
<pre class="lang-java" data-nodeid="11724"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">check</span><span class="hljs-params">(AuthCode authCodeToValidate)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Optional&lt;AuthCode&gt; authCode = autoCodeRepository.findAuthCodeByUsername(authCodeToValidate.getUsername());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (authCode.isPresent()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AuthCode authCodeInStore = authCode.get();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (authCodeToValidate.getCode().equals(authCodeInStore.getCode())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
</code></pre>
<p data-nodeid="11725">这里的逻辑也很简单，就是<strong data-nodeid="11842">把从数据库中获取的认证码与用户传入的认证码进行比对</strong>。</p>
<p data-nodeid="11726">至此，认证服务的核心功能已经构建完毕，下面我们来看业务服务的实现过程。</p>
<h4 data-nodeid="11727">实现业务服务</h4>
<p data-nodeid="11728">在业务服务中，势必需要<strong data-nodeid="11850">调用认证服务提供的 HTTP 端点</strong>来完成用户认证和认证码认证这两个核心的认证操作。因此，我们需要构建一个认证服务的客户端组件完成远程调用。在案例中，我们参考设计模式中的门面（Facade）模式的设计理念，将这个组件命名为 AuthenticationServerFacade，也就是说它是认证服务的一种门面组件，定义如下：</p>
<pre class="lang-java" data-nodeid="11729"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthenticationServerFacade</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> RestTemplate rest;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Value("${auth.server.base.url}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String baseUrl;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">checkPassword</span><span class="hljs-params">(String username, String password)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String url = baseUrl + <span class="hljs-string">"/user/auth"</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; User body = <span class="hljs-keyword">new</span> User();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; body.setUsername(username);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; body.setPassword(password);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpEntity&lt;User&gt; request = <span class="hljs-keyword">new</span> HttpEntity&lt;User&gt;(body);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rest.postForEntity(url, request, Void.class);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">checkAuthCode</span><span class="hljs-params">(String username, String code)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String url = baseUrl + <span class="hljs-string">"/authcode/check"</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; User body = <span class="hljs-keyword">new</span> User();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; body.setUsername(username);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; body.setCode(code);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpEntity&lt;User&gt; request = <span class="hljs-keyword">new</span> HttpEntity&lt;User&gt;(body);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseEntity&lt;Void&gt; response = rest.postForEntity(url, request, Void.class);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> response.getStatusCode().equals(HttpStatus.OK);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11730">这里的 baseUrl 就是认证服务暴露的服务地址，我们使用 RestTemplate 模板类发起对认证服务的远程调用，并根据返回值来判断认证是否通过。</p>
<p data-nodeid="11731">有了 AuthenticationServerFacade，我们就可以在业务服务中集成认证服务了。我们在每次请求的处理过程中完成这种集成工作，此时需要用到拦截器，而这种集成工作显然需要依赖于认证管理器 AuthenticationManager。因此，我们可以先来设计并实现如下所示的 CustomAuthenticationFilter 代码结构：</p>
<pre class="lang-java" data-nodeid="11732"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomAuthenticationFilter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">OncePerRequestFilter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationManager manager;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilterInternal</span><span class="hljs-params">(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)</span> <span class="hljs-keyword">throws</span> ServletException, IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = request.getHeader(<span class="hljs-string">"username"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = request.getHeader(<span class="hljs-string">"password"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String code = request.getHeader(<span class="hljs-string">"code"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//使用 AuthenticationManager 处理认证过程</span>
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11733">上述代码中第一个需要关注的点是 CustomAuthenticationFilter 所扩展的基类 OncePerRequestFilter。顾名思义，OncePerRequestFilter 能够确保在一次请求中只执行一次过滤器逻辑，不会发生多次重复执行的情况。这里我们分别从 HTTP 请求头中获取了用户名 username、密码 password 以及认证码 code 这三个参数，并尝试使用 AuthenticationManager 完成认证。基于<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7697" data-nodeid="11856">03 讲“认证体系：如何深入理解 Spring Security 的用户认证机制？”</a>中的讨论，我们知道 AuthenticationManager 的背后实际上使用了 AuthenticationProvider 执行具体的认证操作。</p>
<p data-nodeid="11734">再来回想一下认证服务中提供的两种认证操作，一种是基于用户名和密码完成用户认证，一种是基于用户名和认证码完成针对认证码的认证。因此，我们需要针对这两种操作分别实现不同的 AuthenticationProvider。例如，如下所示的 UsernamePasswordAuthenticationProvider 就实现了针对用户名和密码的认证操作：</p>
<pre class="lang-java" data-nodeid="11735"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UsernamePasswordAuthenticationProvider</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationServerFacade authServer;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span> <span class="hljs-keyword">throws</span> AuthenticationException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = authentication.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = String.valueOf(authentication.getCredentials());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//调用认证服务完成认证</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; authServer.checkPassword(username, password);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UsernamePasswordAuthenticationToken(username, password);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; aClass)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> UsernamePasswordAuthentication.class.isAssignableFrom(aClass);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11736">可以看到，这里使用了 AuthenticationServerFacade 门面类来完成对认证服务的远程调用。类似地 ，我们也可以构建针对认证码的 AuthenticationProvider，即如下所示的 AuthCodeAuthenticationProvider：</p>
<pre class="lang-java" data-nodeid="11737"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthCodeAuthenticationProvider</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationServerFacade authServer;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span> <span class="hljs-keyword">throws</span> AuthenticationException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = authentication.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String code = String.valueOf(authentication.getCredentials());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//调用认证服务完成认证</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">boolean</span> result = authServer.checkAuthCode(username, code);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (result) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> AuthCodeAuthentication(username, code);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BadCredentialsException(<span class="hljs-string">"Bad credentials."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; aClass)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> AuthCodeAuthentication.class.isAssignableFrom(aClass);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11738">请注意，无论是 UsernamePasswordAuthenticationProvider 还是 AuthCodeAuthenticationProvider，所返回的 UsernamePasswordAuthentication 和 AuthCodeAuthentication 都是<strong data-nodeid="11865">自定义的认证信息类</strong>，它们都继承了 Spring Security 自带的 UsernamePasswordAuthenticationToken。</p>
<p data-nodeid="11739">现在，让我们回到过滤器组件 CustomAuthenticationFilter，并提供对它的完整实现，如下所示：</p>
<pre class="lang-java" data-nodeid="11740"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomAuthenticationFilter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">OncePerRequestFilter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationManager manager;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doFilterInternal</span><span class="hljs-params">(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)</span> <span class="hljs-keyword">throws</span> ServletException, IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = request.getHeader(<span class="hljs-string">"username"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = request.getHeader(<span class="hljs-string">"password"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String code = request.getHeader(<span class="hljs-string">"code"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//如果认证码为空，说明需要先执行用户名/密码认证</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (code == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Authentication a = <span class="hljs-keyword">new</span> UsernamePasswordAuthentication(username, password);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; manager.authenticate(a);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
	        <span class="hljs-comment">//如果认证码不为空，则执行认证码认证</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Authentication a = <span class="hljs-keyword">new</span> AuthCodeAuthentication(username, code);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; manager.authenticate(a);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//如果认证码认证通过，则通过 UUID 生成一个 Token 并添加在响应的消息头中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String token = UUID.randomUUID().toString();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.setHeader(<span class="hljs-string">"Authorization"</span>, token);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">shouldNotFilter</span><span class="hljs-params">(HttpServletRequest request)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> !request.getServletPath().equals(<span class="hljs-string">"/login"</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11741">CustomAuthenticationFilter 的实现过程比较简单，代码也都是自解释的，唯一需要注意的是<strong data-nodeid="11872">在基于认证码的认证过程通过之后，我们会在响应中添加一个“Authorization”消息头，并使用 UUID 值作为 Token 进行返回</strong>。</p>
<p data-nodeid="11742">针对上述代码，我们可以通过如下所示的类图进行总结：</p>
<p data-nodeid="14947" class=""><img src="https://s0.lgstatic.com/i/image6/M01/49/B5/CioPOWDcH0SAWPjtAAB7UfIfAyk803.png" alt="Drawing 4.png" data-nodeid="14951"></p>
<div data-nodeid="14948"><p style="text-align:center">多因素认证执行核心类图</p></div>



<p data-nodeid="11745">最后，我们需要通过 Spring Security 中的配置体系确保各个类之间的有效协作。为此，我们构建了如下所示的 SecurityConfig 类：</p>
<pre class="lang-java" data-nodeid="11746"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SecurityConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> CustomAuthenticationFilter customAuthenticationFilter;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthCodeAuthenticationProvider authCodeAuthenticationProvider;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UsernamePasswordAuthenticationProvider usernamePasswordAuthenticationProvider;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.authenticationProvider(authCodeAuthenticationProvider)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authenticationProvider(usernamePasswordAuthenticationProvider);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.csrf().disable();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.addFilterAt(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customAuthenticationFilter,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BasicAuthenticationFilter.class);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.authorizeRequests()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .anyRequest().authenticated();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> AuthenticationManager <span class="hljs-title">authenticationManager</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.authenticationManager();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="11747">上述配置内容中，我们看到可以通过 addFilterAt() 方法添加自定义过滤器。关于过滤器使用方式的更多内容，你也可以参考<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7702" data-nodeid="11880">08 讲“管道过滤：如何基于 Spring Security 过滤器扩展安全性？”</a>的内容进行回顾。</p>
<p data-nodeid="11748">关于案例的完整代码你可以在这里进行下载：<a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/MultiFactorAuthenticationDemo" data-nodeid="11885">https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/MultiFactorAuthenticationDemo</a>。</p>
<h3 data-nodeid="11749">案例演示</h3>
<p data-nodeid="11750">现在，让我们分别在本地启动认证服务和业务服务，请注意：认证服务的启动端口是 8080，而业务服务的启动端口是 9090。然后我们打开模拟 HTTP 请求的 Postman 并输入相关参数，如下所示：</p>
<p data-nodeid="15384" class=""><img src="https://s0.lgstatic.com/i/image6/M01/49/B5/CioPOWDcH1CAWsVIAACxvLoOLdE765.png" alt="Drawing 5.png" data-nodeid="15387"></p>


<div data-nodeid="15820" class=""><p style="text-align:center">多因素认证的第一步认证示意图：基于用户名+密码</p></div>

<p data-nodeid="11754">显然，该请求只传入了用户名和密码，所以会基于 UsernamePasswordAuthenticationProvider 执行认证过程，从而为用户“jianxiang”生成认证码。<strong data-nodeid="11897">认证码是动态生成的，所以每次请求对应的结果都是不一样的</strong>，我通过查询数据库，获取该认证码为“9750”，你也可以自己做一些尝试。</p>
<p data-nodeid="11755">有了认证码，相当于完成了多因素认证机制的第一步。接下来，我们再次基于这个认证码构建请求并获取响应结果，如下所示：</p>
<p data-nodeid="16251" class=""><img src="https://s0.lgstatic.com/i/image6/M01/49/AD/Cgp9HWDcH1mACUXeAADWBUPX0po513.png" alt="Drawing 7.png" data-nodeid="16254"></p>


<div data-nodeid="16685" class=""><p style="text-align:center">多因素认证的第二步认证示意图：基于用户名+认证码</p></div>

<p data-nodeid="11759">可以看到，通过传入正确的认证码，我们基于 AuthCodeAuthenticationProvider 完成了多因素认证机制中的第二步认证，并最终在 HTTP 响应中生成了一个“Authorization”消息头。</p>
<h3 data-nodeid="11760">小结与预告</h3>
<p data-nodeid="11761">这一讲我们基于多因素认证机制展示了如何利用 Spring Security 中的一些高级主题保护 Web 应用程序的实现方法。多因素认证机制的实现需要构建多个自定义的 AuthenticationProvider，并通过拦截器完成对请求的统一处理。相信案例中展示的这些开发技巧会给你的日常开发工作带来帮助。</p>
<p data-nodeid="11762">本讲内容总结如下：</p>
<p data-nodeid="17114" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/49/AD/Cgp9HWDcH2KAfoU3AAHK941R6Dk155.png" alt="Drawing 9.png" data-nodeid="17117"></p>

<p data-nodeid="11764">这里给你留一道思考题：在 Spring Security 中，如何利用过滤器实现对用户请求的定制化认证？</p>
<p data-nodeid="11765">介绍完今天的案例，我们将进入 Spring Security 关于 OAuth2 协议的介绍部分，并给出 Spring Security 与 Spring Cloud 进行集成的系统方法。下一讲，我们先从 OAuth2 协议的基本概念开始讲起，帮助你深入理解该协议的应用场景、角色以及多种授权模式。</p>

---

### 精选评论

##### **威：
> 太强了😀

