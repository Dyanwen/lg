<p data-nodeid="187894">在上一讲中，我已经带你在 ReactiveSpringCSS 案例系统中通过 WebFlux 创建了响应式 Web 服务，并给你留下了一道思考题：如何实现非阻塞式的跨服务调用？</p>
<p data-nodeid="187895">我们知道在 Spring 中存在一个功能强大的工具类 RestTemplate，专门用来实现基于 HTTP 协议的远程请求和响应处理。RestTemplate 的主要问题在于不支持响应式流规范，也就无法提供非阻塞式的流式操作。Spring 5 全面引入响应式编程模型，同时也提供了 RestTemplate 的响应式版本，这就是 WebClient 工具类。</p>
<p data-nodeid="187896">这一讲我们就针对 WebClient 展开详细的探讨。首先我会带你创建和配置 WebClient对象；然后使用 WebClient 来访问远程 Web 服务，并介绍该组件的一些使用技巧；最后我依然会结合 ReactiveSpringCSS 案例来给出与现有服务之间的集成过程。</p>
<h3 data-nodeid="187897">创建和配置 WebClient</h3>
<p data-nodeid="187898">WebClient 类位于 org.springframework.web.reactive.function.client 包中，要想在项目中集成 WebClient 类，只需要引入 WebFlux 依赖即可。</p>
<h4 data-nodeid="187899">创建 WebClient</h4>
<p data-nodeid="187900">创建 WebClient 有两种方法，一种是通过它所提供的 create() 工厂方法，另一种则是使用 WebClient Builder 构造器工具类。</p>
<p data-nodeid="187901">我们可以直接使用 create() 工厂方法来创建 WebClient 的实例，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187902"><code data-language="java">WebClient webClient = WebClient.create();
</code></pre>
<p data-nodeid="187903">如果我们创建 WebClient 的目的是针对某一个特定服务进行操作，那么就可以使用该服务的地址作为 baseUrl 来初始化 WebClient，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187904"><code data-language="java">WebClient webClient = 
	WebClient.create(<span class="hljs-string">"https://localhost:8081/accounts"</span>);
</code></pre>
<p data-nodeid="187905">WebClient 还附带了一个构造器类 Builder，使用方法也很简单，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187906"><code data-language="java">WebClient webClient = WebClient.builder().build();
</code></pre>
<h4 data-nodeid="187907">配置 WebClient</h4>
<p data-nodeid="187908">创建完 WebClient 实例之后，我们还可以在 WebClient.builder() 方法中添加相关的配置项，来对 WebClient 的行为做一些控制，通常用来设置消息头信息等，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187909"><code data-language="java">WebClient webClient = WebClient.builder()
	.baseUrl(<span class="hljs-string">"https://localhost:8081/accounts"</span>)
    .defaultHeader(HttpHeaders.CONTENT_TYPE, <span class="hljs-string">"application/json"</span>)
	.defaultHeader(HttpHeaders.USER_AGENT, <span class="hljs-string">"Reactive WebClient"</span>)
	.build();
</code></pre>
<p data-nodeid="187910">上述代码展示了 defaultHeader 的使用方法，WebClient.builder() 还包含 defaultCookie、defaultRequest 等多个配置项可供使用。</p>
<p data-nodeid="187911">现在，我们已经成功创建了 WebClient 对象，接下来就可以使用该对象来访问远程服务了。</p>
<h3 data-nodeid="187912">使用 WebClient 访问服务</h3>
<p data-nodeid="187913">在远程服务访问上，WebClient 有几种常见的使用方式，包括非常实用的 retrieve() 和 exchange() 方法、用于封装请求数据的 RequestBody，以及表单和文件的提交。接下来我就对这些使用方式进行详细介绍并给出相关示例。</p>
<h4 data-nodeid="187914">构造 URL</h4>
<p data-nodeid="187915">Web 请求中通过请求路径可以携带参数，在使用 WebClient 时也可以在它提供的 uri() 方法中添加路径变量和参数值。如果我们定义一个包含路径变量名为 id 的 URL，然后将 id 值设置为 100，那么就可以使用如下示例代码。</p>
<pre class="lang-java" data-nodeid="187916"><code data-language="java">webClient.get().uri(<span class="hljs-string">"http://localhost:8081/accounts/{id}"</span>, <span class="hljs-number">100</span>);
</code></pre>
<p data-nodeid="187917">当然，URL 中也可以使用多个路径变量以及多个参数值。如下所示的代码中就定义了 URL 中拥有路径变量 param1 和 param2，实际访问的时候将被替换为 value1 和 value2。如果有很多的参数，只要按照需求对请求地址进行拼装即可。</p>
<pre class="lang-java" data-nodeid="187918"><code data-language="java">webClient.get().uri(<span class="hljs-string">"http://localhost:8081/account/{param1}/{ param2}"</span>, <span class="hljs-string">"value1"</span>, <span class="hljs-string">"value12"</span>);
</code></pre>
<p data-nodeid="187919">同时，我们也可以事先把这些路径变量和参数值拼装成一个 Map 对象，然后赋值给当前 URL。如下所示的代码就定义了 Key 为 param1 和 param2 的 HashMap，实际访问时会从这个 HashMap 中获取参数值进行替换，从而得到最终的请求路径为<a href="http://localhost:8081/accounts/value1/value2?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="188002">http://localhost:8081/accounts/value1/value2</a>，如下所示。</p>
<pre class="lang-java" data-nodeid="187920"><code data-language="java">Map&lt;String, Object&gt; uriVariables = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
uriVariables.put(<span class="hljs-string">"param1"</span>, <span class="hljs-string">"value1"</span>);
uriVariables.put(<span class="hljs-string">"param2"</span>, <span class="hljs-string">"value2"</span>);
webClient.get().uri(<span class="hljs-string">"http://localhost:8081/accounts/{param1}/{param2}"</span>, variables);
</code></pre>
<p data-nodeid="187921">我们还可以通过使用 URIBuilder 来获取对请求信息的完全控制，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187922"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;Account&gt; <span class="hljs-title">getAccounts</span><span class="hljs-params">(String username, String token)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> webClient.get()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(uriBuilder -&gt; uriBuilder.path(<span class="hljs-string">"/accounts"</span>).build())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .header(<span class="hljs-string">"Authorization"</span>, <span class="hljs-string">"Basic "</span> + Base64Utils
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .encodeToString((username + <span class="hljs-string">":"</span> + token).getBytes(UTF_8)))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToFlux(Account.class);
}
</code></pre>
<p data-nodeid="187923">这里我们为每次请求添加了包含授权信息的“Authorization”消息头，用来传递用户名和访问令牌。一旦我们准备好请求信息，就可以使用 WebClient 提供的一系列工具方法完成远程服务的访问，例如上面示例中的 retrieve() 方法。</p>
<h4 data-nodeid="187924">retrieve() 方法</h4>
<p data-nodeid="187925">retrieve() 方法是获取响应主体并对其进行解码的最简单方法，我们再看一个示例，如下所示。</p>
<pre class="lang-java" data-nodeid="187926"><code data-language="java">WebClient webClient = WebClient.create(<span class="hljs-string">"http://localhost:8081"</span>);
&nbsp;
Mono&lt;Account&gt; result = webClient.get()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"/accounts/{id}"</span>, id)
	    .accept(MediaType.APPLICATION_JSON)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToMono(Account.class);
</code></pre>
<p data-nodeid="187927">上述代码使用 JSON 作为序列化方式，我们也可以根据需要设置其他方式，例如采用 MediaType.TEXT_EVENT_STREAM 以实现基于流的处理，示例如下。</p>
<pre class="lang-java" data-nodeid="187928"><code data-language="java">Flux&lt;Order&gt; result = webClient.get()
  .uri(<span class="hljs-string">"/accounts"</span>).accept(MediaType.TEXT_EVENT_STREAM)
  .retrieve()
  .bodyToFlux(Account.class);
</code></pre>
<h4 data-nodeid="187929">exchange() 方法</h4>
<p data-nodeid="187930">如果希望对响应拥有更多的控制权，retrieve() 方法就显得无能为力，这时候我们可以使用 exchange() 方法来访问整个响应结果，该响应结果是一个 ClientResponse 对象，包含了响应的状态码、Cookie 等信息，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187931"><code data-language="java">Mono&lt;Account&gt; result = webClient.get()
 .uri(<span class="hljs-string">"/accounts/{id}"</span>, id)
 .accept(MediaType.APPLICATION_JSON)
 .exchange() 
 .flatMap(response -&gt; response.bodyToMono(Account.class));
</code></pre>
<p data-nodeid="187932">以上代码演示了如何对结果执行 flatMap() 操作符的实现方式，通过这一操作符调用 ClientResponse 的 bodyToMono() 方法以获取目标 Account 对象。</p>
<h4 data-nodeid="187933">使用 RequestBody</h4>
<p data-nodeid="187934">如果你有一个 Mono 或 Flux 类型的请求体，那么可以使用 WebClient 的 body() 方法来进行编码，使用示例如下所示。</p>
<pre class="lang-java" data-nodeid="187935"><code data-language="java">Mono&lt;Account&gt; accountMono = ... ;
&nbsp;
Mono&lt;Void&gt; result = webClient.post()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"/accounts"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .contentType(MediaType.APPLICATION_JSON)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .body(accountMono, Account.class)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToMono(Void.class);
</code></pre>
<p data-nodeid="187936">如果请求对象是一个普通的 POJO 而不是 Flux/Mono，则可以使用 syncBody() 方法作为一种快捷方式来传递请求，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187937"><code data-language="java">Account account = ... ;
&nbsp;
Mono&lt;Void&gt; result = webClient.post()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"/accounts"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .contentType(MediaType.APPLICATION_JSON)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .syncBody(account)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToMono(Void.class);
</code></pre>
<h4 data-nodeid="187938">表单和文件提交</h4>
<p data-nodeid="187939">当传递的请求体是一个 MultiValueMap 对象时，WebClient 默认发起的是表单提交。例如，针对用户登录场景，我们可以构建一个 MultiValueMap 对象，并传入参数 username 和 password 进行提交，代码如下所示。</p>
<pre class="lang-java" data-nodeid="187940"><code data-language="java">String baseUrl = <span class="hljs-string">"http://localhost:8081"</span>;
WebClient webClient = WebClient.create(baseUrl);
&nbsp;
MultiValueMap&lt;String, String&gt; map = <span class="hljs-keyword">new</span> LinkedMultiValueMap&lt;&gt;();
map.add(<span class="hljs-string">"username"</span>, <span class="hljs-string">"jianxiang"</span>);
map.add(<span class="hljs-string">"password"</span>, <span class="hljs-string">"password"</span>);
&nbsp;
Mono&lt;String&gt; mono = webClient.post()
	.uri(<span class="hljs-string">"/login"</span>)
	.syncBody(map)
	.retrieve()
	.bodyToMono(String.class);
</code></pre>
<p data-nodeid="187941">如果想提交 Multipart Data，我们可以使用 MultipartBodyBuilder 工具类来简化请求的构建过程。MultipartBodyBuilder 的使用方法如下所示，最终我们也将得到一个 MultiValueMap 对象。</p>
<pre class="lang-java" data-nodeid="187942"><code data-language="java">MultipartBodyBuilder builder = <span class="hljs-keyword">new</span> MultipartBodyBuilder();
builder.part(<span class="hljs-string">"paramPart"</span>, <span class="hljs-string">"value"</span>);
builder.part(<span class="hljs-string">"filePart"</span>, <span class="hljs-keyword">new</span> FileSystemResource(<span class="hljs-string">"jianxiang.png"</span>));
builder.part(<span class="hljs-string">"accountPart"</span>, <span class="hljs-keyword">new</span> Account(<span class="hljs-string">"jianxiang"</span>));
&nbsp;
MultiValueMap&lt;String, HttpEntity&lt;?&gt;&gt; parts = builder.build();
</code></pre>
<p data-nodeid="187943">一旦 MultiValueMap 构建完成，通过 WebClient 的 syncBody() 方法就可以实现请求提交，我已经在上文中提到过这种实现方法。</p>
<p data-nodeid="187944">以上介绍的都是 WebClient 所提供的基础方法，我们可以使用这些方法来满足面向业务处理的常规请求。但如果你希望对远程调用过程有更多的控制，那么就需要使用 WebClient 所提供的的一些其他使用技巧了。让我们一起来看一下。</p>
<h3 data-nodeid="187945">WebClient 的其他使用技巧</h3>
<p data-nodeid="187946">除了实现常规的 HTTP 请求之外，WebClient 还有一些高级用法，包括请求拦截和异常处理等。</p>
<h4 data-nodeid="187947">请求拦截</h4>
<p data-nodeid="187948" class="">和传统 RestTemplate 一样，WebClient 也支持使用过滤器函数。让我们回顾 WebClient 的构造器 Builder，你会发现 Builder 实际上是一个接口，它内置了一批初始化 Builder 和 WebClient 的方法定义。DefaultWebClientBuilder 就是该接口的默认实现，截取该类的部分核心代码如下。</p>
<pre class="lang-java" data-nodeid="187949"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultWebClientBuilder</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WebClient</span>.<span class="hljs-title">Builder</span> </span>{
	&nbsp;
<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> WebClient.<span class="hljs-function">Builder <span class="hljs-title">filter</span><span class="hljs-params">(ExchangeFilterFunction filter)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(filter, <span class="hljs-string">"ExchangeFilterFunction must not be null"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; initFilters().add(filter);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
	 }
	&nbsp;
	…
	&nbsp;
<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> WebClient <span class="hljs-title">build</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ExchangeFunction exchange = initExchangeFunction();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ExchangeFunction filteredExchange = (<span class="hljs-keyword">this</span>.filters != <span class="hljs-keyword">null</span> ? <span class="hljs-keyword">this</span>.filters.stream()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .reduce(ExchangeFilterFunction::andThen)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(filter -&gt; filter.apply(exchange))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .orElse(exchange) : exchange);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultWebClient(filteredExchange, initUriBuilderFactory(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; unmodifiableCopy(<span class="hljs-keyword">this</span>.defaultHeaders), unmodifiableCopy(<span class="hljs-keyword">this</span>.defaultCookies),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> DefaultWebClientBuilder(<span class="hljs-keyword">this</span>));
	 }
	&nbsp;
	…
}
</code></pre>
<p data-nodeid="187950">可以看到 build() 方法的目的是构建出一个 DefaultWebClient，而 DefaultWebClient 的构造函数中依赖于 ExchangeFunction 接口。我们来看一下 ExchangeFunction 接口的定义，其中的 filter() 方法传入并执行 ExchangeFilterFunction，如下所示。</p>
<pre class="lang-java" data-nodeid="187951"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ExchangeFunction</span> </span>{
	&nbsp;&nbsp;&nbsp; …
	&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> ExchangeFunction <span class="hljs-title">filter</span><span class="hljs-params">(ExchangeFilterFunction filter)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(filter, <span class="hljs-string">"'filter' must not be null"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> filter.apply(<span class="hljs-keyword">this</span>);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="187952">当我们看到 Filter（过滤器）这个单词时，思路上就可以触类旁通了。在 Web 应用程序中，Filter 体现的就是一种拦截器作用，而多个 Filter 组合起来构成一种过滤器链。ExchangeFilterFunction 也是一个接口，其部分核心代码如下所示。</p>
<pre class="lang-java" data-nodeid="187953"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ExchangeFilterFunction</span> </span>{
&nbsp;
<span class="hljs-function">Mono&lt;ClientResponse&gt; <span class="hljs-title">filter</span><span class="hljs-params">(ClientRequest request, ExchangeFunction next)</span></span>;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">default</span> ExchangeFilterFunction <span class="hljs-title">andThen</span><span class="hljs-params">(ExchangeFilterFunction after)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(after, <span class="hljs-string">"'after' must not be null"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (request, next) -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ExchangeFunction nextExchange = exchangeRequest -&gt; after.filter(exchangeRequest, next);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> filter(request, nextExchange);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };
&nbsp;&nbsp;&nbsp; }
	&nbsp;
<span class="hljs-function"><span class="hljs-keyword">default</span> ExchangeFunction <span class="hljs-title">apply</span><span class="hljs-params">(ExchangeFunction exchange)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(exchange, <span class="hljs-string">"'exchange' must not be null"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> request -&gt; <span class="hljs-keyword">this</span>.filter(request, exchange);
	}
	&nbsp;
	…
}
</code></pre>
<p data-nodeid="187954">显然，ExchangeFilterFunction 通过 andThen() 方法将自身添加到过滤器链并实现 filter() 这个函数式方法。我们可以使用过滤器函数以任何方式拦截和修改请求，例如通过修改 ClientRequest 来调用 ExchangeFilterFucntion 过滤器链中的下一个过滤器，或者让 ClientRequest 直接返回以阻止过滤器链的进一步执行。</p>
<p data-nodeid="187955">作为示例，如下代码演示了如何使用过滤器功能添加 HTTP 基础认证机制。</p>
<pre class="lang-java" data-nodeid="187956"><code data-language="java">WebClient client = WebClient.builder()
  .filter(basicAuthentication(<span class="hljs-string">"username"</span>, <span class="hljs-string">"password"</span>))
  .build();
</code></pre>
<p data-nodeid="187957">这样，基于客户端过滤机制，我们不需要在每个请求中添加 Authorization 消息头，过滤器将拦截每个 WebClient 请求并自动添加该消息头。再来看一个例子，我们将编写一个自定义的过滤器函数 logFilter()，代码如下所示。</p>
<pre class="lang-java" data-nodeid="187958"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> ExchangeFilterFunction <span class="hljs-title">logFilter</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (clientRequest, next) -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.info(<span class="hljs-string">"Request: {} {}"</span>, clientRequest.method(), clientRequest.url());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; clientRequest.headers()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .forEach((name, values) -&gt; values.forEach(value -&gt; logger.info(<span class="hljs-string">"{}={}"</span>, name, value)));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> next.exchange(clientRequest);
&nbsp;&nbsp;&nbsp; };
}
</code></pre>
<p data-nodeid="187959">显然，logFilter() 方法的作用是对每个请求做详细的日志记录。我们同样可以通过 filter() 方法把该过滤器添加到请求链路中，代码如下所示。</p>
<pre class="lang-java" data-nodeid="187960"><code data-language="java">WebClient webClient = WebClient.builder()
  .filter(logFilter())
  .build();
</code></pre>
<h4 data-nodeid="187961">异常处理</h4>
<p data-nodeid="187962">当发起一个请求所得到的响应状态码为 4XX 或 5XX 时，WebClient 就会抛出一个 WebClientResponseException 异常，我们可以使用 onStatus() 方法来自定义对异常的处理方式，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="187963"><code data-language="java">public Flux&lt;Account&gt; listAccounts() {
&nbsp;&nbsp;&nbsp; return webClient.get()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri("/accounts)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .onStatus(HttpStatus::is4xxClientError, clientResponse -&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono.error(new MyCustomClientException())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .onStatus(HttpStatus::is5xxServerError, clientResponse -&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono.error(new MyCustomServerException())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToFlux(Account.class);
}
</code></pre>
<p data-nodeid="187964">这里我们构建了一个 MyCustomServerException 来返回自定义异常信息。需要注意的是，这种处理方式只适用于使用 retrieve() 方法进行远程请求的场景，exchange() 方法在获取 4XX 或 5XX 响应的情况下不会引发异常。因此，当使用 exchange() 方法时，我们需要自行检查状态码并以合适的方式处理它们。</p>
<h3 data-nodeid="187965">案例集成：基于 WebClient 的非阻塞式跨服务通信</h3>
<p data-nodeid="187966">介绍完 WebClient 类的使用方式之后，让我们回到 ReactiveSpringCSS 案例。在上一讲中，我已经给出了位于 customer-service 的 CustomerService 类中的 generateCustomerTicket 方法的代码结构，该方法用于完成与 account-service 和 order-service 进行集成，我们来回顾一下。</p>
<pre class="lang-java" data-nodeid="187967"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;CustomerTicket&gt; <span class="hljs-title">generateCustomerTicket</span><span class="hljs-params">(String accountId, String orderNumber)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 创建 CustomerTicket 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CustomerTicket customerTicket = <span class="hljs-keyword">new</span> CustomerTicket();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setId(<span class="hljs-string">"C_"</span> + UUID.randomUUID().toString());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 account-service 获取 Account 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;AccountMapper&gt; accountMapper = getRemoteAccountByAccountId(accountId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 从远程 order-service 中获取 Order 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;OrderMapper&gt; orderMapper = getRemoteOrderByOrderNumber(orderNumber);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono&lt;CustomerTicket&gt; monoCustomerTicket = 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Mono.zip(accountMapper, orderMapper).flatMap(tuple -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountMapper account = tuple.getT1();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; OrderMapper order = tuple.getT2();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 设置 CustomerTicket 对象属性</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(customerTicket);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 保存 CustomerTicket 对象并返回</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> monoCustomerTicket.flatMap(customerTicketRepository::save);
}
</code></pre>
<p data-nodeid="187968">另一方面，上一讲中我也给出了在 order-service 中基于函数式编程模型构建 Web 服务的实现过程。同样的，这里也以 getRemoteOrderByOrderNumber 方法为例来和你讨论如何完成与 Web 服务之间的远程调用。getRemoteOrderByOrderNumber 方法定义如下。</p>
<pre class="lang-java" data-nodeid="187969"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Mono&lt;OrderMapper&gt; <span class="hljs-title">getRemoteOrderByOrderNumber</span><span class="hljs-params">(String orderNumber)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderClient.getOrderByOrderNumber(orderNumber);
}
</code></pre>
<p data-nodeid="187970">这里，我们会构建一个 ReactiveOrderClient 类来完成对 order-service 的远程访问，如下所示。</p>
<pre class="lang-java" data-nodeid="187971"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveOrderClient</span> </span>{ 
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;OrderMapper&gt; <span class="hljs-title">getOrderByOrderNumber</span><span class="hljs-params">(String orderNumber)</span> </span>{&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;  Mono&lt;OrderMapper&gt; orderMono = WebClient.create()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .get()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .uri(<span class="hljs-string">"localhost:8081/orders/{orderNumber}"</span>, orderNumber)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieve()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .bodyToMono(OrderMapper.class).log("getOrderFromRemote");
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderMono;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="189025" class="te-preview-highlight">你可以注意到，这里基于 WebClient.create() 工厂方法构建了一个 WebClient 对象，并通过它的 retrieve 方法来完成对远程 order-serivce 的请求过程。同时，我们看到这里的返回对象是一个 Mono<code data-backticks="1" data-nodeid="189027">&lt;OrderMapper&gt;</code>，而不是一个 Mono<code data-backticks="1" data-nodeid="189029">&lt;Order&gt;</code> 对象。事实上，它们的内部字段都是一一对应的，只是位于两个不同的代码工程中，所以故意从命名上做了区分。WebClient 会自动完成 OrderMapper 与 Order 之间的自动映射。</p>


<h3 data-nodeid="187973">小结与预告</h3>
<p data-nodeid="187974">在上一讲的基础上，这一讲我为你引入了 WebClient 模板类来完成对远程 HTTP 端点的响应式访问。WebClient 为开发人员提供了一大批有用的工具方法来实现 HTTP 请求的发送以及响应的获取。同时，该模板类还提供了一批定制化的入口供开发人员嵌入对 HTTP 请求过程进行精细化管理的处理逻辑。</p>
<p data-nodeid="187975">这里给你留一道思考题：在使用 WebClient 时，如何实现对请求异常等控制逻辑的定制化处理？</p>
<p data-nodeid="187976">关于远程调用，业界也存在一些协议，在网络通信层面提供非阻塞式的交互体验。RSocket 就是这一领域的主流实现，下一讲我将针对该协议和你一起展开讨论，到时候见。</p>
<blockquote data-nodeid="187977">
<p data-nodeid="187978">点击链接，获取课程相关代码↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="188054">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### **东：
> 能不能讲讲原理哦，为啥webcclient能够实现非阻塞远程调用，是怎么融入到响应式体系里面去的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这块原理后面有机会再讲，要将的话涉及很多底层技术体系，篇幅会比较大

##### **鹏：
> 请问老师，如果要在线上使用webclient，应该怎么配置它的一些参数，像最大连接数，工作线程该配置多少，是否要使用netty的全局线程这些，项目中如果只有一部分用了mono，后面block拿到结果再去处理别的逻辑，会有问题吗，是不是必须整个链条都是mono的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; WebClient的参数可以参考官网的说明，也取决于项目的需求。
全链路都需要是响应式的，中间有一处是阻塞的，那么整个链路其他响应式部分就是无效的

