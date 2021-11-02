<p data-nodeid="6154" class="">通过前面课程的学习，我们已经掌握了构建一个 Spring Boot 应用程序的数据访问层组件实现方法。接下来的几讲，我们将讨论另一层组件，即 Web 服务层的构建方式。</p>
















<p data-nodeid="7814" class="">服务与服务之间的交互是系统设计和发展的必然需求，其涉及 Web 服务的发布及消费，今天我们先讨论如何在 Spring Boot 应用程序中发布 Web 服务。</p>





<h3 data-nodeid="679">SpringCSS 系统中的服务交互</h3>
<p data-nodeid="680">在具体的技术体系介绍之前，我们先来梳理 SpringCSS 案例中服务交互之间的应用场景。</p>
<p data-nodeid="8146" class="">对于客服系统而言，其核心业务流程是生成客服工单，而工单的生成通常需要使用用户账户信息和所关联的订单信息。</p>

<p data-nodeid="19820" class="">在 SpringCSS 案例中，前面几讲我们已经构建了一个用于管理订单的 order-service，接下来我们将分别构建管理用户账户的 account-service 及核心的客服服务 customer-service。</p>



































<p data-nodeid="24480" class="">关于三个服务之间的交互方式，我们先通过一张图了解下，如下图所示：</p>














<p data-nodeid="684"><img src="https://s0.lgstatic.com/i/image2/M01/03/C1/CgpVE1_hv2uAGbTPAABvOdgwGhs113.png" alt="图片6 (1).png" data-nodeid="769"></p>
<div data-nodeid="685"><p style="text-align:center">SpringCSS 案例系统中三个服务的交互方式图</p></div>
<p data-nodeid="29140" class="">实际上，通过上图我们已经可以梳理工单生成 generateCustomerTicket 核心方法的执行流程，这里我们先给出代码的框架，如下代码所示：</p>














<pre class="lang-java" data-nodeid="687"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> CustomerTicket <span class="hljs-title">generateCustomerTicket</span><span class="hljs-params">(Long accountId, String orderNumber)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 创建客服工单对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CustomerTicket customerTicket = <span class="hljs-keyword">new</span> CustomerTicket();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 account-service 中获取 Account 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Account account = getRemoteAccountById(accountId);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 order-service 中获取 Order 读写</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = getRemoteOrderByOrderNumber(orderNumber);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 设置 CustomerTicket 对象并保存</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setAccountId(accountId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setOrderNumber(order.getOrderNumber());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicketRepository.save(customerTicket);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> customerTicket;
}
</code></pre>
<p data-nodeid="31132" class="">因 getRemoteAccountById 与 getRemoteOrderByOrderNumber 方法都涉及远程 Web 服务的调用，因此首先我们需要创建 Web 服务。</p>






<p data-nodeid="32128" class="">而 Spring Boot 为我们创建 Web 服务提供了非常强大的组件化支持，简单而方便，我们一起来看一下。</p>



<h3 data-nodeid="690">创建 RESTful 服务</h3>
<p data-nodeid="691">在当下的分布式系统及微服务架构中，RESTful 风格是一种主流的 Web 服务表现方式。</p>
<p data-nodeid="692">在接下来的内容中，我们将演示如何使用 Spring Boot 创建 RESTful 服务。但在此之前，我们先来理解什么是 RESTful 服务。</p>
<h4 data-nodeid="693">理解 RESTful 架构风格</h4>
<p data-nodeid="694">你可能听说过 REST 这个名称，但并不清楚它的含义。</p>
<p data-nodeid="34802" class="">REST（Representational State Transfer，表述性状态转移）本质上只是一种架构风格而不是一种规范，这种架构风格把位于服务器端的访问入口看作一个资源，每个资源都使用 URI（Universal Resource Identifier，统一资源标识符） 得到一个唯一的地址，且在传输协议上使用标准的 HTTP 方法，比如最常见的  GET、PUT、POST 和 DELETE。</p>








<p data-nodeid="696" class="">下表展示了 RESTful 风格的一些具体示例：</p>
<p data-nodeid="697"><img src="https://s0.lgstatic.com/i/image2/M01/03/C0/Cip5yF_hv3eAbVtLAAGhaTfDT5s664.png" alt="图片1 (2).png" data-nodeid="782"></p>
<div data-nodeid="698"><p style="text-align:center">RESTful 风格示例</p></div>
<p data-nodeid="699">另一方面，客户端与服务器端的数据交互涉及序列化问题。关于序列化完成业务对象在网络环境上的传输的实现方式有很多，常见的有文本和二进制两大类。</p>
<p data-nodeid="700">目前 JSON 是一种被广泛采用的序列化方式，本课程中所有的代码实例我们都将 JSON 作为默认的序列化方式。</p>
<h4 data-nodeid="701">使用基础注解</h4>
<p data-nodeid="35134" class="">在原有 Spring Boot 应用程序的基础上，我们可以通过构建一系列的 Controller 类暴露 RESTful 风格的 HTTP 端点。这里的 Controller 与 Spring MVC 中的 Controller 概念上一致，最简单的 Controller 类如下代码所示：</p>

<pre class="lang-java" data-nodeid="703"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloController</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping("/")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">index</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"Hello World!"</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="704">从以上代码中可以看到，这里包含了 @RestController 和 @GetMapping 这两个注解。</p>
<p data-nodeid="705">其中，@RestController 注解继承自 Spring MVC 中的 @Controller 注解，顾名思义就是一个基于 RESTful 风格的 HTTP 端点，并且会自动使用 JSON 实现 HTTP 请求和响应的序列化/反序列化方式。</p>
<p data-nodeid="706">通过这个特性，在构建 RESTful 服务时，我们可以使用 @RestController 注解代替 @Controller 注解以简化开发。</p>
<p data-nodeid="707">另外一个 @GetMapping 注解也与 Spring MVC 中的 @RequestMapping 注解类似。我们先来看看 @RequestMapping 注解的定义，该注解所提供的属性都比较容易理解，如下代码所示：</p>
<pre class="lang-java" data-nodeid="708"><code data-language="java"><span class="hljs-meta">@Target({ElementType.METHOD, ElementType.TYPE})</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Mapping</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> RequestMapping {
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("path")</span>
&nbsp;&nbsp;&nbsp; String[] value() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("value")</span>
&nbsp;&nbsp;&nbsp; String[] path() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; RequestMethod[] method() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; String[] params() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; String[] headers() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; String[] consumes() <span class="hljs-keyword">default</span> {};
&nbsp;&nbsp;&nbsp; String[] produces() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="709">而 @GetMapping 的注解的定义与 @RequestMapping 非常类似，只是默认使用了 RequestMethod.GET 指定 HTTP 方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="710"><code data-language="java"><span class="hljs-meta">@Target(ElementType.METHOD)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@RequestMapping(method = RequestMethod.GET)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> GetMapping {
</code></pre>
<p data-nodeid="711">Spring Boot 2 中引入的一批新注解中，除了 @GetMapping ，还有 @PutMapping、@PostMapping、@DeleteMapping 等注解，这些注解极大方便了开发人员显式指定 HTTP 的请求方法。当然，你也可以继续使用原先的 @RequestMapping 实现同样的效果。</p>
<p data-nodeid="712">我们再看一个更加具体的示例，以下代码展示了 account-service 中的 AccountController。</p>
<pre class="lang-java" data-nodeid="713"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value = "accounts")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountController</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping(value = "/{accountId}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Account <span class="hljs-title">getAccountById</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("accountId")</span> Long accountId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Account account = <span class="hljs-keyword">new</span> Account();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; account.setId(<span class="hljs-number">1L</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; account.setAccountCode(<span class="hljs-string">"DemoCode"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; account.setAccountName(<span class="hljs-string">"DemoName"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> account;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="714">在该 Controller 中，通过静态的业务代码我们完成了根据账号编号（accountId）获取用户账户信息的业务流程。</p>
<p data-nodeid="715">这里用到了两层 Mapping，第一层的 @RequestMapping 注解在服务层级定义了服务的根路径“/accounts”，第二层的 @GetMapping 注解则在操作级别定义了 HTTP 请求方法的具体路径及参数信息。</p>
<p data-nodeid="716">到这里，一个典型的 RESTful 服务已经开发完成了，现在我们可以通过 java –jar 命令直接运行 Spring Boot 应用程序了。</p>
<p data-nodeid="717">在启动日志中，我们发现了以下输出内容（为了显示效果，部分内容做了调整），可以看到自定义的这个 AccountController 已经成功启动并准备接收响应。</p>
<pre class="lang-xml" data-nodeid="718"><code data-language="xml">RequestMappingHandlerMapping : Mapped "{[/accounts/{accountId}], methods=[GET]}" onto public com.springcss.account.domain.Account com.springcss.account.controller.AccountController.getAccountById (java.lang.Long)
</code></pre>
<p data-nodeid="719">在本课程中，我们将引入 Postman 来演示如何通过 HTTP 协议暴露的端点进行远程服务访问。</p>
<p data-nodeid="720">Postman 为我们完成 HTTP 请求和响应过程提供了可视化界面，你可以尝试编写一个 AccountController，并通过 Postman 访问“<a href="http://localhost:8082/accounts/1" data-nodeid="802">http://localhost:8082/accounts/1</a>”端点以得到响应结果。</p>
<p data-nodeid="721">在前面的 AccountController 中，我们还看到了一个新的注解 @PathVariable，该注解作用于输入的参数，下面我们就来看看如何通过这些注解控制请求的输入。</p>
<h4 data-nodeid="722">控制请求输入和输出</h4>
<p data-nodeid="723">Spring Boot 提供了一系列简单有用的注解来简化对请求输入的控制过程，常用的包括 @PathVariable、@RequestParam 和 @RequestBody。</p>
<p data-nodeid="724" class="">其中 @PathVariable 注解用于获取路径参数，即从类似 url/{id} 这种形式的路径中获取 {id} 参数的值。该注解的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="725"><code data-language="java"><span class="hljs-meta">@Target(ElementType.PARAMETER)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> PathVariable {
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("name")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("value")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">required</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>
<p data-nodeid="726">通常，使用 @PathVariable 注解时，我们只需要指定一个参数的名称即可。我们可以再看一个示例，如下代码所示：</p>
<pre class="lang-java" data-nodeid="727"><code data-language="java"><span class="hljs-meta">@GetMapping(value = "/{accountName}")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Account <span class="hljs-title">getAccountByAccountName</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("accountName")</span> String accountName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; Account account = accountService.getAccountByAccountName(accountName);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> account;
}
</code></pre>
<p data-nodeid="728">@RequestParam 注解的作用与 @PathVariable 注解类似，也是用于获取请求中的参数，但是它面向类似 url?id=XXX 这种路径形式。</p>
<p data-nodeid="729">该注解的定义如下代码所示，相较 @PathVariable 注解，它只是多了一个设置默认值的 defaultValue 属性。</p>
<pre class="lang-java" data-nodeid="730"><code data-language="java"><span class="hljs-meta">@Target(ElementType.PARAMETER)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> RequestParam {
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("name")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor("value")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">required</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">defaultValue</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ValueConstants.DEFAULT_NONE</span>;
}
</code></pre>
<p data-nodeid="731">在 HTTP 协议中，content-type 属性用来指定所传输的内容类型，我们可以通过 @RequestMapping 注解中的 produces 属性来设置这个属性。</p>
<p data-nodeid="732">在设置这个属性时，我们通常会将其设置为“application/json”，如下代码所示：</p>
<pre class="lang-java" data-nodeid="733"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value = "accounts", produces="application/json")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountController</span> </span>{
</code></pre>
<p data-nodeid="734">@RequestBody 注解用来处理 content-type 为 application/json 类型时的编码内容，通过 @RequestBody 注解可以将请求体中的 JSON 字符串绑定到相应的 JavaBean 上。</p>
<p data-nodeid="735">如下代码所示就是一个使用 @RequestBody 注解来控制输入的场景。</p>
<pre class="lang-java" data-nodeid="736"><code data-language="java"><span class="hljs-meta">@PutMapping(value = "/")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">updateAccount</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Account account)</span> </span>{
</code></pre>
<p data-nodeid="737">如果使用 @RequestBody 注解，我们可以在 Postman 中输入一个 JSON 字符串来构建输入对象，如下代码所示：</p>
<p data-nodeid="738"><img src="https://s0.lgstatic.com/i/image2/M01/01/5C/Cip5yF_YgIGACIM7AAAr5-G8i7M764.png" alt="Drawing 1.png" data-nodeid="818"></p>
<div data-nodeid="739"><p style="text-align:center">使用 Postman 输入 JSON 字符串发起 HTTP 请求示例图</p></div>
<p data-nodeid="740">通过以上内容的讲解，我们发现使用注解的操作很简单，接下来我们有必要探讨下控制请求输入的规则。</p>
<p data-nodeid="741"><strong data-nodeid="823">关于控制请求输入的规则，关键在于按照 RESTful 风格的设计原则设计 HTTP 端点，对于这点业界也存在一些约定。</strong></p>
<ul data-nodeid="742">
<li data-nodeid="743">
<p data-nodeid="744">以 Account 这个领域实体为例，如果我们把它视为一种资源，那么 HTTP 端点的根节点命名上通常采用复数形式，即“/accounts”，正如前面的示例代码所示。</p>
</li>
<li data-nodeid="745">
<p data-nodeid="746">在设计 RESTful API 时，我们需要基于 HTTP 语义设计对外暴露的端点的详细路径。针对常见的 CRUD 操作，我们展示了 RESTful API 与非 RESTful API 的一些区别。</p>
</li>
</ul>
<p data-nodeid="747"><img src="https://s0.lgstatic.com/i/image/M00/8B/E8/CgqCHl_hv5yAPnDHAADI8geMxRU064.png" alt="图片3 (1).png" data-nodeid="828"></p>
<div data-nodeid="748"><p style="text-align:center">RESTful 风格对比示例</p></div>
<p data-nodeid="749">基于以上介绍的控制请求输入的实现方法，我们可以给出 account-service 中 AccountController 类的完整实现过程，如下代码所示：</p>
<pre class="lang-java" data-nodeid="750"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value = "accounts", produces="application/json")</span>
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
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping(value = "accountname/{accountName}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Account <span class="hljs-title">getAccountByAccountName</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("accountName")</span> String accountName)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Account account = accountService.getAccountByAccountName(accountName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> account;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PostMapping(value = "/")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addAccount</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Account account)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountService.addAccount(account);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PutMapping(value = "/")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">updateAccount</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Account account)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountService.updateAccount(account);
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@DeleteMapping(value = "/")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAccount</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Account account)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountService.deleteAccount(account);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="751">介绍完对请求输入的控制，我们再来讨论如何控制请求的输出。</p>
<p data-nodeid="752"><strong data-nodeid="835">相较输入控制，输出控制就要简单很多，因为 Spring Boot 所提供的 @RestController 注解已经屏蔽了底层实现的复杂性，我们只需要返回一个普通的业务对象即可。</strong>@RestController 注解相当于是 Spring MVC 中 @Controller 和 @ResponseBody 这两个注解的组合，它们会自动返回 JSON 数据。</p>
<p data-nodeid="753">这里我们也给出了 order-service 中的 OrderController 实现过程，如下代码所示：</p>
<pre class="lang-java" data-nodeid="754"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value="orders/jpa")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JpaOrderController</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; JpaOrderService jpaOrderService;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping(value = "/{orderId}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> JpaOrder <span class="hljs-title">getOrderById</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable</span> Long orderId)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JpaOrder order = jpaOrderService.getOrderById(orderId);
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> order;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping(value = "orderNumber/{orderNumber}")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> JpaOrder <span class="hljs-title">getOrderByOrderNumber</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable</span> String orderNumber)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JpaOrder order = jpaOrderService.getOrderByOrderNumber(orderNumber);
<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JpaOrder order = jpaOrderService.getOrderByOrderNumberByExample(orderNumber);</span>
<span class="hljs-comment">//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JpaOrder order = jpaOrderService.getOrderByOrderNumberBySpecification(orderNumber);</span>
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> order;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@PostMapping(value = "")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> JpaOrder <span class="hljs-title">addOrder</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> JpaOrder order)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JpaOrder result = jpaOrderService.addOrder(order);
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="35798" class="">从上面示例可以看到，我们使用了 09 讲中介绍的 Spring Data JPA 完成实体对象及数据访问功能。</p>


<h3 data-nodeid="756">小结与预告</h3>
<p data-nodeid="36468">构建 Web 服务是开发 Web 应用程序的基本需求，而设计并实现 RESTful 风格的 Web 服务是开发人员必须具备的开发技能。</p>
<p data-nodeid="38822" class="">基于 Spring Boot 框架，开发人员只需要使用几个注解就能实现复杂的 HTTP 端点，并暴露给其他服务进行使用，工作都变得非常简单。</p>









<p data-nodeid="758">这里给你留一道思考题：在使用 Spring Boot 构建 Web 服务时，可以使用哪些注解实现对输入参数的有效控制？</p>
<p data-nodeid="42182" class="te-preview-highlight">现在，我们已经构建了一个具有 RESTful 风格的 Web 服务。在 12 讲中，我们将介绍 Spring Boot 所提供的另一个模板工具类 RestTemplate 完成对 HTTP 端点的消费。</p>

---

### 精选评论

##### *旺：
> produces是生成的格式吧，consumes是接收的格式？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对，指定生产者所提供的内容格式。

##### **飞：
> 老师，请问为什么注释掉SpringCssUser相关几个类就不报错呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这几个类只是示例代码，是不完整的，除了这几个类之外都是完整的，所以能运行。

##### **飞：
> org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'springCssUserDetailsService': Unsatisfied dependency expressed through field 'repository'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'springCssUserRepository': Invocation of init method failed; nested exception is java.lang.IllegalArgumentException: Not a managed type: class com.springcss.account.security.SpringCssUser老师好，请问运行account-service项目报错是什么问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; SpringCssUser相关几个类都是为了演示用的Demo类，没有实际的应用，直接注释掉不用应该就可以了，你看一下代码整个执行流程应该就明白了。

