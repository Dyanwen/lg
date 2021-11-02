<p data-nodeid="1920" class="">上一讲，我们引入了 Spring 框架中专门用于构建响应式 Web 服务的 WebFlux 框架，同时我也给出了两种创建 RESTful 风格 HTTP 端点实现方法中的一种，即注解编程模型。今天，我将介绍另一种实现方法——如何使用函数式编程模型创建响应式 RESTful 服务，这种编程模型与传统的基于 Spring MVC 构建 RESTful 服务的方法有较大差别。</p>
<h3 data-nodeid="1921">WebFlux 函数式编程模型</h3>
<p data-nodeid="2315">在引入函数式编程模型之前，让我先带你回顾一下 Spring WebFlux 的系统架构图，如下所示。</p>
<p data-nodeid="2316" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/32/E0/Cgp9HWBuf--Aek0BAAT9zBvy7qI451.png" alt="图片2.png" data-nodeid="2321"></p>
<div data-nodeid="2317"><p style="text-align:center">Spring WebFlux 架构图（来自 Spring 官网）</p></div>




<p data-nodeid="1925">在上图的后半部分，你可以看到在 Spring WebFlux 中，函数式编程模型的核心概念是 Router Functions，对标 Spring MVC 中的 @Controller、@RequestMapping 等标准注解。而 Router Functions 则提供一套函数式风格的 API，其中最重要的就是 Router 和 Handler 接口。我们可以简单把 Router 对应成 RequestMapping，把 Controller 对应为 Handler。</p>
<p data-nodeid="1926">当我发起一个远程调用，传入的 HTTP 请求由 HandlerFunction 处理， HandlerFunction 本质上是一个接收 ServerRequest 并返回 Mono 的函数。ServerRequest 和 ServerResponse 是一对不可变接口，用来提供对底层 HTTP 消息的友好访问。在介绍具体的实现案例之前，让我先从这两个接口开始讲起。</p>
<h4 data-nodeid="1927">ServerRequest</h4>
<p data-nodeid="1928">ServerRequest 代表请求对象，可以访问各种 HTTP 请求元素，包括请求方法、URI 和参数，以及通过单独的 ServerRequest.Headers 获取 HTTP 请求头信息。ServerRequest 通过一系列 bodyToMono() 和 bodyToFlux() 方法提供对请求消息体进行访问的途径。例如，如果我们希望将请求消息体提取为 Mono 类型的对象，可以使用如下方法。<br>
<br></p>
<pre class="lang-java" data-nodeid="1929"><code data-language="java">Mono&lt;String&gt; string = request.bodyToMono(String.class);
</code></pre>
<p data-nodeid="1930">而如果我们希望将请求消息体提取为 Flux 类型的对象，可以使用如下方法，其中 Order 是可以从请求消息体反序列化的实体类。<br>
<br></p>
<pre class="lang-java" data-nodeid="1931"><code data-language="java">Flux&lt;Order&gt; order = request.bodyToFlux(Order.class);
</code></pre>
<p data-nodeid="1932">上述的 bodyToMono() 和 bodyToFlux() 两个方法实际上是通用的 ServerRequest.body(BodyExtractor) 工具方法的快捷方式，该方法如下所示。</p>
<pre class="lang-java" data-nodeid="1933"><code data-language="java">&lt;T&gt; <span class="hljs-function">T <span class="hljs-title">body</span><span class="hljs-params">(BodyExtractor&lt;T, ? <span class="hljs-keyword">super</span> ServerHttpRequest&gt; extractor)</span></span>;
</code></pre>
<p data-nodeid="1934">BodyExtractor 是一种请求消息体的提取器，允许我们编写自己的提取逻辑。请注意 BodyExtractor 提取的对象是一个 ServerHttpRequest 类型的实例，而这个 ServerHttpRequest 是非阻塞的，与之对应的还有一个 ServerHttpResponse 对象。响应式 Web 操作的正是这组非阻塞的 ServerHttpRequest 和 ServerHttpResponse 对象，而不再是 Spring MVC 里的传统 HttpServletRequest 和 HttpServletResponse 对象。</p>
<p data-nodeid="1935">当然，如果我们不需要实现定制化的提取逻辑，就可以使用框架为我们提供的常用的 BodyExtractors 实例。通过 BodyExtractors，上面的例子可以替换为以下形式。</p>
<pre class="lang-java" data-nodeid="1936"><code data-language="java">Mono&lt;String&gt; string = 
	request.body(BodyExtractors.toMono(String.class);
	&nbsp;
Flux&lt;Person&gt; Order= 
	request.body(BodyExtractors.toFlux(Order.class);
</code></pre>
<h4 data-nodeid="1937">ServerResponse</h4>
<p data-nodeid="1938">与ServerRequest 对应，ServerResponse 提供对 HTTP 响应的访问。由于它是不可变的，因此我们可以使用构建器创建一个新的 ServerResponse。构建器允许设置响应状态、添加响应标题并提供响应的具体内容。例如，下面的示例演示了如何通过 ok() 方法创建代表 200 状态码的响应，其中我将响应体的类型设置为 JSON 格式，而响应的具体内容是一个 Mono 对象。<br>
<br></p>
<pre class="lang-java" data-nodeid="1939"><code data-language="java">Mono&lt;Order&gt; order = …;
ServerResponse.ok().contentType(MediaType.APPLICATION_JSON)
&nbsp;&nbsp;&nbsp;&nbsp; .body(order);
</code></pre>
<p data-nodeid="1940">通过 body() 方法来加载响应内容是构建 ServerResponse 最常见的方法，这里我们将 Order 对象作为返回值。如果想要返回各种类型的对象，我们也可以使用 BodyInserters 工具类所提供的构建方法，如常见的 fromObject() 和 fromPublisher() 方法等。以下示例代码中，我们通过 fromObject() 方法直接返回一个 "Hello World"。</p>
<pre class="lang-java" data-nodeid="1941"><code data-language="java">ServerResponse.ok().body(BodyInserters.fromObject(<span class="hljs-string">"Hello World"</span>));
</code></pre>
<p data-nodeid="1942">上述方法的背后实际上是利用 BodyBuilder 接口中的一组 body() 方法，来构建一个 ServerResponse 对象，典型的 body() 方法如下所示。</p>
<pre class="lang-java" data-nodeid="1943"><code data-language="java"><span class="hljs-function">Mono&lt;ServerResponse&gt; <span class="hljs-title">body</span><span class="hljs-params">(BodyInserter&lt;?, ? <span class="hljs-keyword">super</span> ServerHttpResponse&gt; inserter)</span></span>;
</code></pre>
<p data-nodeid="1944">这里我们同样看到了非阻塞式的 ServerHttpResponse 对象。这种 body() 方法比较常见的用法是返回新增和更新操作的结果，你在本讲后续的内容中将会看到这种使用方法。</p>
<h4 data-nodeid="1945">HandlerFunction</h4>
<p data-nodeid="1946">将 ServerRequest 和 ServerResponse 组合在一起就可以创建 HandlerFunction。HandlerFunction 也是一个接口，定义如下。</p>
<pre class="lang-java" data-nodeid="1947"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">HandlerFunction</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ServerResponse</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Mono&lt;T&gt; <span class="hljs-title">handle</span><span class="hljs-params">(ServerRequest request)</span></span>;
}
</code></pre>
<p data-nodeid="1948">我们可以通过实现 HandlerFunction 接口中的 handle() 方法来创建定制化的请求响应处理机制。例如，以下所示的是一个简单的“Hello World”处理函数代码示例。</p>
<pre class="lang-java" data-nodeid="1949"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloWorldHandlerFunction</span> <span class="hljs-keyword">implements</span> 
	<span class="hljs-title">HandlerFunction</span>&lt;<span class="hljs-title">ServerResponse</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;ServerResponse&gt; <span class="hljs-title">handle</span><span class="hljs-params">(ServerRequest request)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ServerResponse.ok().body(
	BodyInserters.fromObject(<span class="hljs-string">"Hello World"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
};
</code></pre>
<p data-nodeid="1950">可以看到，这里使用了前面介绍的 ServerResponse 所提供的 body() 方法返回一个 String 类型的消息体。</p>
<p data-nodeid="1951">通常，针对某个领域实体都存在 CRUD 等常见的操作，所以需要编写多个类似的处理函数，比较烦琐。这时候就推荐将多个处理函数分组到一个专门的 Handler 类中。在本讲的后面我同样会演示这种实现方法。</p>
<h4 data-nodeid="1952">RouterFunction</h4>
<p data-nodeid="1953">现在，我们已经可以通过 HandlerFunction 创建请求的处理逻辑，接下来需要把具体请求与这种处理逻辑关联起来，RouterFunction 可以帮助我们实现这一目标。RouterFunction 与传统 SpringMVC 中的 @RequestMapping 注解功能类似。</p>
<p data-nodeid="1954">创建 RouterFunction 的最常见做法是使用如下所示的 route 方法，该方法通过使用请求谓词和处理函数创建一个 ServerResponse 对象。</p>
<pre class="lang-java" data-nodeid="1955"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> &lt;T extends ServerResponse&gt; <span class="hljs-function">RouterFunction&lt;T&gt; <span class="hljs-title">route</span><span class="hljs-params">(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RequestPredicate predicate, HandlerFunction&lt;T&gt; handlerFunction)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultRouterFunction&lt;&gt;(predicate, handlerFunction);
}
</code></pre>
<p data-nodeid="1956">RouterFunction 的核心逻辑位于这里的 DefaultRouterFunction 类中，该类的 route() 方法如下所示。</p>
<pre class="lang-java" data-nodeid="1957"><code data-language="java"><span class="hljs-keyword">public</span> Mono&lt;HandlerFunction&lt;T&gt;&gt; route(ServerRequest request) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.predicate.test(request)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (logger.isTraceEnabled()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String logPrefix = request.exchange().getLogPrefix();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.trace(logPrefix + String.format(<span class="hljs-string">"Matched %s"</span>, <span class="hljs-keyword">this</span>.predicate));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(<span class="hljs-keyword">this</span>.handlerFunction);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.empty();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="1958">可以看到，该方法将传入的 ServerRequest 路由到具体的处理函数 HandlerFunction。如果请求与特定路由匹配，则返回处理函数的结果，否则就返回一个空的 Mono。</p>
<p data-nodeid="1959">RequestPredicates 工具类提供了常用的谓词，能够实现包括基于路径、HTTP 方法、内容类型等条件的自动匹配。一个简单的 RouterFunction 示例如下，我们用它来实现对 "/hello-world"请求路径的自动路由，这里用到了前面创建的 HelloWorldHandlerFunction。</p>
<pre class="lang-java" data-nodeid="1960"><code data-language="java">RouterFunction&lt;ServerResponse&gt; helloWorldRoute =&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RouterFunctions.route(RequestPredicates.path(<span class="hljs-string">"/hello-world"</span>),
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> HelloWorldHandlerFunction());
</code></pre>
<p data-nodeid="1961">类似的，我们应该把 RouterFunction 和各种 HandlerFunction 按照需求结合起来一起使用，常见的做法也是根据领域对象来设计对应的 RouterFunction。</p>
<p data-nodeid="1962">路由机制的优势在于它的组合型。两个路由功能可以组合成一个新的路由功能，并通过一定的评估方法路由到其中任何一个处理函数。如果第一个路由的谓词不匹配，则第二个谓词会被评估。请注意组合的路由器功能会按照顺序进行评估，因此在通用功能之前放置一些特定功能是一项最佳实践。在 RouterFunction 中，同样提供了对应的组合方法来实现这一目标，请看下面的代码。</p>
<pre class="lang-java" data-nodeid="1963"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">default</span> RouterFunction&lt;T&gt; <span class="hljs-title">and</span><span class="hljs-params">(RouterFunction&lt;T&gt; other)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> RouterFunctions.SameComposedRouterFunction&lt;&gt;(<span class="hljs-keyword">this</span>, other);
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">default</span> RouterFunction&lt;T&gt; <span class="hljs-title">andRoute</span><span class="hljs-params">(RequestPredicate predicate, HandlerFunction&lt;T&gt; handlerFunction)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> and(RouterFunctions.route(predicate, handlerFunction));
}
</code></pre>
<p data-nodeid="1964">我们可以通过调用上述两个方法中的任意一个来组合两个路由功能，其中后者相当于 RouterFunction.and() 方法与 RouterFunctions.route() 方法的集成。以下代码演示了 RouterFunctions 的组合特性。</p>
<pre class="lang-java" data-nodeid="1965"><code data-language="java">RouterFunction&lt;ServerResponse&gt; personRoute =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; route(GET(<span class="hljs-string">"/orders/{id}"</span>).and(accept(APPLICATION_JSON)), personHandler::getOrderById)
.andRoute(GET(<span class="hljs-string">"/orders"</span>).and(accept(APPLICATION_JSON)), personHandler::getOrders)
.andRoute(POST(<span class="hljs-string">"/orders"</span>).and(contentType(APPLICATION_JSON)), personHandler::createOrder);
</code></pre>
<p data-nodeid="1966">RequestPredicates 工具类所提供的大多数谓词也具备组合特性。例如， RequestPredicates.GET(String) 方法的实现如下所示。</p>
<pre class="lang-java" data-nodeid="1967"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> RequestPredicate <span class="hljs-title">GET</span><span class="hljs-params">(String pattern)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> method(HttpMethod.GET).and(path(pattern));
}
</code></pre>
<p data-nodeid="1968">可以看到，该方法是 RequestPredicates.method(HttpMethod.GET) 和 RequestPredicates.path(String) 的组合。我们可以通过调用 RequestPredicate.and(RequestPredicate) 方法或 RequestPredicate.or(RequestPredicate) 方法来构建复杂的请求谓词。</p>
<h3 data-nodeid="1969">案例集成：ReactiveSpringCSS 中的 Web 服务</h3>
<p data-nodeid="1970">现在回到 ReactiveSpringCSS 案例，我们已经明确了在案例系统中，customer-service 分别需要访问 account-service 和 order-service 服务中的 Web 服务。在上一讲的内容中，我们已经基于注解编程模型实现了 account-service 中的 AccountController。今天我将继续给你演示 order-service 中 Web 服务的实现过程。</p>
<p data-nodeid="1971">基于函数式编程模型，在 order-service 中，我们编写一个 OrderHandler 专门实现根据 OrderNumber 获取 Order 领域实体的处理函数，如下所示。</p>
<pre class="lang-java" data-nodeid="1972"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderHandler</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> OrderService orderService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;ServerResponse&gt; <span class="hljs-title">getOrderByOrderNumber</span><span class="hljs-params">(ServerRequest request)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String orderNumber = request.pathVariable(<span class="hljs-string">"orderNumber"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ServerResponse.ok().body(<span class="hljs-keyword">this</span>.orderService.getOrderByOrderNumber(orderNumber), Order.class);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="1973">在上述代码示例中，我们创建了一个 OrderHandler 类，然后注入 OrderService 并实现了一个 getOrderByOrderNumber() 处理函数。</p>
<p data-nodeid="1974">现在我们已经具备了 OrderHandler，就可以创建对应的 OrderRouter 了，示例代码如下。</p>
<pre class="lang-java" data-nodeid="1975"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderRouter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> RouterFunction&lt;ServerResponse&gt; <span class="hljs-title">routeOrder</span><span class="hljs-params">(OrderHandler orderHandler)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> RouterFunctions.route(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RequestPredicates.GET(<span class="hljs-string">"/orders/{orderNumber}"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and(RequestPredicates.accept(MediaType.APPLICATION_JSON)),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orderHandler::getOrderByOrderNumber);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="1976">在这个示例中，我们通过访问“/orders/{orderNumber}”端点就会自动触发 orderHandler 中的 getOrderByOrderNumber() 方法并返回相应的 ServerResponse。</p>
<p data-nodeid="1977">接下来，假设我们已经分别通过远程调用获取了目标 Account 对象和 Order 对象，那么 generateCustomerTicket 方法的执行流程就可以明确了。基于响应式编程的实现方法，我们可以得到如下所示的示例代码。</p>
<pre class="lang-java" data-nodeid="1978"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;CustomerTicket&gt; <span class="hljs-title">generateCustomerTicket</span><span class="hljs-params">(String accountId, String orderNumber)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 创建 CustomerTicket 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CustomerTicket customerTicket = <span class="hljs-keyword">new</span> CustomerTicket();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setId(<span class="hljs-string">"C_"</span> + UUID.randomUUID().toString());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 account-service 获取 Account 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;AccountMapper&gt; accountMapper = getRemoteAccountByAccountId(accountId);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 order-service 中获取 Order 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;OrderMapper&gt; orderMapper = getRemoteOrderByOrderNumber(orderNumber);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;CustomerTicket&gt; monoCustomerTicket = 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono.zip(accountMapper, orderMapper).flatMap(tuple -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountMapper account = tuple.getT1();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; OrderMapper order = tuple.getT2();

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(account == <span class="hljs-keyword">null</span> || order == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(customerTicket);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 设置 CustomerTicket 对象属性</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setAccountId(account.getId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setOrderNumber(order.getOrderNumber());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setCreateTime(<span class="hljs-keyword">new</span> Date());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setDescription(<span class="hljs-string">"TestCustomerTicket"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(customerTicket);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 保存 CustomerTicket 对象并返回</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> monoCustomerTicket.flatMap(customerTicketRepository::save);
}
</code></pre>
<p data-nodeid="1979">显然，这里的 getRemoteAccountById 和 getRemoteOrderByOrderNumber 方法都涉及了非阻塞式的远程 Web 服务的调用，这一过程我们将放在下一讲中详细介绍。</p>
<p data-nodeid="1980">请注意，到这里时使用了 Reactor 框架中的 zip 操作符，将 accountMapper 流中的元素与 orderMapper 流中的元素按照一对一的方式进行合并，合并之后得到一个 Tuple2 对象。然后，我们再分别从这个 Tuple2 对象中获取 AccountMapper 和 OrderMapper 对象，并将它们的属性填充到所生成的 CustomerTicket 对象中。最后，我们通过 flatMap 操作符调用了 customerTicketRepository 的 save 方法完成了数据的持久化。这是 zip 和 flatMap 这两个操作符非常经典的一种应用场景，你需要熟练掌握。</p>
<h3 data-nodeid="1981">小结与预告</h3>
<p data-nodeid="1982">好了，那么本讲内容就说到这。延续上一讲，我们接着讨论了 Spring WebFlux 的使用方法，并给出了基于函数式编程模型的 RESTful 端点创建方法。在这种开发模型中，开发人员需要重点把握 ServerRequest、ServerResponse、HandlerFunction 以及 RouterFunction 这四个核心对象的使用方法。</p>
<p data-nodeid="1983">这里给你留一道思考题：你知道在 WebFlux 函数式编程模型中包含哪些核心编程对象吗？</p>
<p data-nodeid="1984">现在，我们已经通过 WebFlux 构建了响应式 Web 服务，下一步就是如何来消费它们了。Spring 也专门提供了一个非阻塞式的 WebClient 工具类来完成这一目标，下一讲我就来和你系统地讨论这个工具类的使用方法，到时见。</p>
<blockquote data-nodeid="1985">
<p data-nodeid="1986" class="">点击链接，获取课程相关代码↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2053">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### *晨：
> 老师你好，在项目中，背压怎么设置，设置到哪块

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 就是使用一组onBackpressureXXX操作符，后面案例中会有示例

