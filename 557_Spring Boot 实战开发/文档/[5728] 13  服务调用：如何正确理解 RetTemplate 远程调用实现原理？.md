<p data-nodeid="32794" class="">在 12 讲中，我们详细描述了如何使用 RestTemplate 访问 HTTP 端点的使用方法，它涉及 RestTemplate 初始化、发起请求及获取响应结果等核心环节。今天，我们将基于上一课时中的这些环节，从源码出发让你真正理解 RestTemplate 实现远程调用的底层原理。</p>
<h3 data-nodeid="32795">初始化 RestTemplate 实例</h3>
<p data-nodeid="32796">12 讲中我们提到可以通过 RestTemplate 提供的几个构造函数对 RestTemplate 进行初始化。在分析这些构造函数之前，我们有必要先看一下 RestTemplate 类的定义，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32797"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RestTemplate</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">InterceptingHttpAccessor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RestOperations</span>
</span></code></pre>
<p data-nodeid="32798">从上述代码中，我们可以看到 RestTemplate 扩展了 InterceptingHttpAccessor 抽象类，并实现了 RestOperations 接口。接下来我们围绕 RestTemplate 的方法定义进行设计思路的梳理。</p>
<p data-nodeid="32799">首先，我们来看看 RestOperations 接口的定义，这里截取了部分核心方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32800"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">RestOperations</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">getForObject</span><span class="hljs-params">(String url, Class&lt;T&gt; responseType, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
	&lt;T&gt; <span class="hljs-function">ResponseEntity&lt;T&gt; <span class="hljs-title">getForEntity</span><span class="hljs-params">(String url, Class&lt;T&gt; responseType, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
	&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">postForObject</span><span class="hljs-params">(String url, <span class="hljs-meta">@Nullable</span> Object request, Class&lt;T&gt; responseType,Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">put</span><span class="hljs-params">(String url, <span class="hljs-meta">@Nullable</span> Object request, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
	&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">(String url, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
	&lt;T&gt; <span class="hljs-function">ResponseEntity&lt;T&gt; <span class="hljs-title">exchange</span><span class="hljs-params">(String url, HttpMethod method, <span class="hljs-meta">@Nullable</span> HttpEntity&lt;?&gt; requestEntity,
	&nbsp;
&nbsp;&nbsp;&nbsp; Class&lt;T&gt; responseType, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException</span>;
	…
}
</code></pre>
<p data-nodeid="32801">显然，RestOperations 接口定义了 12 讲中介绍到的 get/post/put/delete/exhange 等所有远程调用方法组，这些方法都遵循 RESTful 架构风格而设计。RestTemplate 为这些接口提供了实现机制，这是它的一条代码支线。</p>
<p data-nodeid="32802">然后我们再看 InterceptingHttpAccessor，它是一个抽象类，包含的核心变量如下代码所示：</p>
<pre class="lang-java" data-nodeid="32803"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InterceptingHttpAccessor</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">HttpAccessor</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;ClientHttpRequestInterceptor&gt; interceptors = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">volatile</span> ClientHttpRequestFactory interceptingRequestFactory;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
}
</code></pre>
<p data-nodeid="32804">通过变量定义，我们明确了 InterceptingHttpAccessor 包含两部分处理功能，一部分负责设置和管理请求拦截器 ClientHttpRequestInterceptor，另一部分负责获取用于创建客户端 HTTP 请求的工厂类 ClientHttpRequestFactory。</p>
<p data-nodeid="32805">同时，我们注意到 InterceptingHttpAccessor 同样存在一个父类 HttpAccessor，这个父类值真正实现了 ClientHttpRequestFactory 的创建及如何通过 ClientHttpRequestFactory 获取代表客户端请求的 ClientHttpRequest 对象。HttpAccessor 的核心变量如下代码所示：</p>
<pre class="lang-java" data-nodeid="32806"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HttpAccessor</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> ClientHttpRequestFactory requestFactory = <span class="hljs-keyword">new</span> SimpleClientHttpRequestFactory();
&nbsp;&nbsp;&nbsp; …
}
</code></pre>
<p data-nodeid="32807">从以上代码我们可以看到，HttpAccessor 中创建了 SimpleClientHttpRequestFactory 作为系统默认的 ClientHttpRequestFactory。关于 ClientHttpRequestFactory，我们会在本课时的后续内容中进行详细的讨论。</p>
<p data-nodeid="32808">最后，针对这部分内容我们再来梳理下 RestTemplate 的类层结构，如下图所示：</p>
<p data-nodeid="33231" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/04/34/Cip5yF_q_YOAHp35AAB_-PnTOp8699.png" alt="图片3.png" data-nodeid="33235"></p>
<div data-nodeid="33232"><p style="text-align:center">RestTemplate 的类层结构</p></div>


<p data-nodeid="32811">在 RestTemplate 的类层结构中，我们能快速理解它的设计思想。整个类层结构清晰地分成两条支线，左边支线用于完成与 HTTP 请求相关的实现机制，而右边支线提供了基于 RESTful 风格的操作入口，并使用了面向对象中的接口和抽象类完成这两部分功能的聚合。</p>
<h3 data-nodeid="32812">RestTemplate 核心执行流程</h3>
<p data-nodeid="32813">介绍完 RestTemplate 的实例化过程，接下来我们来分析它的核心执行流程。</p>
<p data-nodeid="32814">作为用于远程调用的模板工具类，我们可以从具备多种请求方式的 exchange 方法入手，该方法的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32815"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">ResponseEntity&lt;T&gt; <span class="hljs-title">exchange</span><span class="hljs-params">(String url, HttpMethod method,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Nullable</span> HttpEntity&lt;?&gt; requestEntity, Class&lt;T&gt; responseType, Object... uriVariables)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> RestClientException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//构建请求回调</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RequestCallback requestCallback = httpEntityCallback(requestEntity, responseType);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//构建响应体抽取器</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseExtractor&lt;ResponseEntity&lt;T&gt;&gt; responseExtractor = responseEntityExtractor(responseType);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行远程调用</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> nonNull(execute(url, method, requestCallback, responseExtractor, uriVariables));
}
</code></pre>
<p data-nodeid="32816">显然，我们应该进一步关注这里的 execute 方法。事实上，无论我们采用 get/put/post/delete 中的哪种方法发起请求，RestTemplate 负责执行远程调用时，使用的都是 execute 方法，该方法定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32817"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-meta">@Nullable</span>
<span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">execute</span><span class="hljs-params">(String url, HttpMethod method, <span class="hljs-meta">@Nullable</span> RequestCallback requestCallback, <span class="hljs-meta">@Nullable</span> ResponseExtractor&lt;T&gt; responseExtractor, Object... uriVariables)</span> <span class="hljs-keyword">throws</span> RestClientException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; URI expanded = getUriTemplateHandler().expand(url, uriVariables);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> doExecute(expanded, method, requestCallback, responseExtractor);
}
</code></pre>
<p data-nodeid="32818">从以上代码中，我们发现 execute 方法首先通过 UriTemplateHandler 构建了一个 URI，然后将请求过程委托给 doExecute 方法进行处理，该方法定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32819"><code data-language="java"><span class="hljs-keyword">protected</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">doExecute</span><span class="hljs-params">(URI url, <span class="hljs-meta">@Nullable</span> HttpMethod method, <span class="hljs-meta">@Nullable</span> RequestCallback requestCallback,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Nullable</span> ResponseExtractor&lt;T&gt; responseExtractor)</span> <span class="hljs-keyword">throws</span> RestClientException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(url, <span class="hljs-string">"URI is required"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(method, <span class="hljs-string">"HttpMethod is required"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClientHttpResponse response = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建请求对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClientHttpRequest request = createRequest(url, method);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (requestCallback != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行对请求的回调</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; requestCallback.doWithRequest(request);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取调用结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response = request.execute();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//处理调用结果</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; handleResponse(url, method, response);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//使用结果提取从结果中提取数据</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (responseExtractor != <span class="hljs-keyword">null</span> ? responseExtractor.extractData(response) : <span class="hljs-keyword">null</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (IOException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String resource = url.toString();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String query = url.getRawQuery();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resource = (query != <span class="hljs-keyword">null</span> ? resource.substring(<span class="hljs-number">0</span>, resource.indexOf(<span class="hljs-string">'?'</span>)) : resource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ResourceAccessException(<span class="hljs-string">"I/O error on "</span> + method.name() +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">" request for \""</span> + resource + <span class="hljs-string">"\": "</span> + ex.getMessage(), ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (response != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.close();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32820">从上述方法中，我们发现使用 RestTemplate 进行远程调用时，主要涉及创建请求对象、执行远程调用及处理响应结果这三大步骤，下面我们分别展开说明下。</p>
<h4 data-nodeid="32821">创建请求对象</h4>
<p data-nodeid="32822">创建请求对象的入口方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="32823"><code data-language="java">ClientHttpRequest request = createRequest(url, method);
</code></pre>
<p data-nodeid="32824">通过跟踪上面的 createRequest 方法，我们发现流程执行到了前面介绍的 HttpAccessor 类，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32825"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HttpAccessor</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> ClientHttpRequestFactory requestFactory = <span class="hljs-keyword">new</span> SimpleClientHttpRequestFactory();
&nbsp;&nbsp;&nbsp; …
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> ClientHttpRequest <span class="hljs-title">createRequest</span><span class="hljs-params">(URI url, HttpMethod method)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClientHttpRequest request = getRequestFactory().createRequest(url, method);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (logger.isDebugEnabled()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Created "</span> + method.name() + <span class="hljs-string">" request for \""</span> + url + <span class="hljs-string">"\""</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> request;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32826">创建 ClientHttpRequest 的过程是一种典型的工厂模式应用场景，这里我们直接创建了一个实现 ClientHttpRequestFactory 接口的 SimpleClientHttpRequestFactory 对象，然后再通过这个对象的 createRequest 方法创建了客户端请求对象 ClientHttpRequest 并返回给上层组件进行使用。ClientHttpRequestFactory 接口的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32827"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ClientHttpRequestFactory</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建客户端请求对象</span>
	<span class="hljs-function">ClientHttpRequest <span class="hljs-title">createRequest</span><span class="hljs-params">(URI uri, HttpMethod httpMethod)</span> <span class="hljs-keyword">throws</span> IOException</span>;
}
</code></pre>
<p data-nodeid="32828">在 Spring 中，存在一批 ClientHttpRequestFactory 接口的实现类，而SimpleClientHttpRequestFactory 是它的默认实现，在实现自定义的 ClientHttpRequestFactory 时，开发人员也可以根据需要自行选择。</p>
<p data-nodeid="32829">为简单起见，我们直接跟踪 SimpleClientHttpRequestFactory 的代码，来看它的 createRequest 方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32830"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> bufferRequestBody = <span class="hljs-keyword">true</span>;
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ClientHttpRequest <span class="hljs-title">createRequest</span><span class="hljs-params">(URI uri, HttpMethod httpMethod)</span> <span class="hljs-keyword">throws</span> IOException </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpURLConnection connection = openConnection(uri.toURL(), <span class="hljs-keyword">this</span>.proxy);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; prepareConnection(connection, httpMethod.name());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.bufferRequestBody) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SimpleBufferingClientHttpRequest(connection, <span class="hljs-keyword">this</span>.outputStreaming);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SimpleStreamingClientHttpRequest(connection, <span class="hljs-keyword">this</span>.chunkSize, <span class="hljs-keyword">this</span>.outputStreaming);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32831">在上述 createRequest 中，首先我们通过传入的 URI 对象构建了一个 HttpURLConnection 对象，然后对该对象进行一些预处理，最后构造并返回一个 ClientHttpRequest 实例。</p>
<p data-nodeid="32832">通过翻阅代码，我们发现上述的 openConnection 方法只是通过 URL 对象的 openConnection 方法返回了一个 UrlConnection，而 prepareConnection 方法也只是完成了对 HttpUrlConnection 超时时间、请求方法等常见属性的设置。</p>
<p data-nodeid="32833">在这里，我们注意到 bufferRequestBody 参数的值为 true，因此通过 createRequest 方法最终返回的结果是一个 SimpleBufferingClientHttpRequest 对象。</p>
<h4 data-nodeid="32834">执行远程调用</h4>
<p data-nodeid="32835">一旦获取了请求对象，我们就可以发起远程调用并获取响应了，RestTemplate 中的入口方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="32836"><code data-language="java">response = request.execute();
</code></pre>
<p data-nodeid="32837">这里的 request 就是前面创建的 SimpleBufferingClientHttpRequest 类，我们可以先来看一下该类的类层结构，如下图所示：</p>
<p data-nodeid="32938" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/36/CgpVE1_q_XeAF3WjAABzAH8vhP8188.png" alt="图片5.png" data-nodeid="32942"></p>
<div data-nodeid="32939"><p style="text-align:center">SimpleBufferingClientHttpRequest 类层结构图</p></div>


<p data-nodeid="32840">在上图的 AbstractClientHttpRequest 中，定义了如下代码所示的 execute 方法。</p>
<pre class="lang-java" data-nodeid="32841"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ClientHttpResponse <span class="hljs-title">execute</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; assertNotExecuted();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClientHttpResponse result = executeInternal(<span class="hljs-keyword">this</span>.headers);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.executed = <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> ClientHttpResponse <span class="hljs-title">executeInternal</span><span class="hljs-params">(HttpHeaders headers)</span> <span class="hljs-keyword">throws</span> IOException</span>;
</code></pre>
<p data-nodeid="32842">AbstractClientHttpRequest 类的作用是防止 HTTP 请求的 Header 和 Body 被多次写入，所以在 execute 方法返回之前，我们设置了一个 executed 标志位。同时，在 execute 方法中，我们最终调用了一个抽象方法 executeInternal，这个方法的实现在 AbstractClientHttpRequest 的子类 AbstractBufferingClientHttpRequest 中，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32843"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> ClientHttpResponse <span class="hljs-title">executeInternal</span><span class="hljs-params">(HttpHeaders headers)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">byte</span>[] bytes = <span class="hljs-keyword">this</span>.bufferedOutput.toByteArray();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (headers.getContentLength() &lt; <span class="hljs-number">0</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; headers.setContentLength(bytes.length);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClientHttpResponse result = executeInternal(headers, bytes);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.bufferedOutput = <span class="hljs-keyword">new</span> ByteArrayOutputStream(<span class="hljs-number">0</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> ClientHttpResponse <span class="hljs-title">executeInternal</span><span class="hljs-params">(HttpHeaders headers, <span class="hljs-keyword">byte</span>[] bufferedOutput)</span>&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> IOException</span>;
</code></pre>
<p data-nodeid="32844">和 AbstractClientHttpRequest 类一样，我们进一步梳理了一个抽象方法 executeInternal，这个抽象方法通过最底层的 SimpleBufferingClientHttpRequest 类实现，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32845"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> ClientHttpResponse <span class="hljs-title">executeInternal</span><span class="hljs-params">(HttpHeaders headers, <span class="hljs-keyword">byte</span>[] bufferedOutput)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; addHeaders(<span class="hljs-keyword">this</span>.connection, headers);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// JDK &lt;1.8 doesn't support getOutputStream with HTTP DELETE</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (getMethod() == HttpMethod.DELETE &amp;&amp; bufferedOutput.length == <span class="hljs-number">0</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.connection.setDoOutput(<span class="hljs-keyword">false</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.connection.getDoOutput() &amp;&amp; <span class="hljs-keyword">this</span>.outputStreaming) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.connection.setFixedLengthStreamingMode(bufferedOutput.length);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.connection.connect();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.connection.getDoOutput()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FileCopyUtils.copy(bufferedOutput, <span class="hljs-keyword">this</span>.connection.getOutputStream());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// Immediately trigger the request in a no-output scenario as well</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.connection.getResponseCode();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SimpleClientHttpResponse(<span class="hljs-keyword">this</span>.connection);
}
</code></pre>
<p data-nodeid="32846">这里通过 FileCopyUtils.copy 工具方法，我们把结果写入输出流上了，executeInternal 方法最终返回的结果是一个包装了 Connection 对象的 SimpleClientHttpResponse。</p>
<h4 data-nodeid="32847">处理响应结果</h4>
<p data-nodeid="32848">一个 HTTP 请求处理的最后一步是从 ClientHttpResponse 中读取输入流，然后格式化为一个响应体并将其转化为业务对象，入口代码如下所示：</p>
<pre class="lang-java" data-nodeid="32849"><code data-language="java"><span class="hljs-comment">//处理调用结果</span>
handleResponse(url, method, response);
<span class="hljs-comment">//使用结果提取从结果中提取数据</span>
<span class="hljs-keyword">return</span> (responseExtractor != <span class="hljs-keyword">null</span> ? responseExtractor.extractData(response) : <span class="hljs-keyword">null</span>);
</code></pre>
<p data-nodeid="32850">我们先来看这里的 handleResponse 方法，定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32851"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleResponse</span><span class="hljs-params">(URI url, HttpMethod method, ClientHttpResponse response)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseErrorHandler errorHandler = getErrorHandler();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">boolean</span> hasError = errorHandler.hasError(response);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (logger.isDebugEnabled()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(method.name() + <span class="hljs-string">" request for \""</span> + url + <span class="hljs-string">"\" resulted in "</span> +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.getRawStatusCode() + <span class="hljs-string">" ("</span> + response.getStatusText() + <span class="hljs-string">")"</span> +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (hasError ? <span class="hljs-string">"; invoking error handler"</span> : <span class="hljs-string">""</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (IOException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// ignore</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (hasError) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; errorHandler.handleError(url, method, response);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32852">以上代码中，通过 getErrorHandler 方法我们获取了一个 ResponseErrorHandler，如果响应的状态码错误，我们可以调用 handleError 来处理错误并抛出异常。在这里，我们发现这段代码实际上并没有真正处理返回的数据，而只是执行了错误处理。</p>
<p data-nodeid="32853">而获取响应数据并完成转化的工作是在 ResponseExtractor 中，该接口定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="32854"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ResponseExtractor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Nullable</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">T <span class="hljs-title">extractData</span><span class="hljs-params">(ClientHttpResponse response)</span> <span class="hljs-keyword">throws</span> IOException</span>;
}
</code></pre>
<p data-nodeid="32855">在 RestTemplate 类中，我们定义了一个 ResponseEntityResponseExtractor 内部类实现了ResponseExtractor 接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32856"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ResponseEntityResponseExtractor</span> &lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">ResponseExtractor</span>&lt;<span class="hljs-title">ResponseEntity</span>&lt;<span class="hljs-title">T</span>&gt;&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Nullable</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> HttpMessageConverterExtractor&lt;T&gt; delegate;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ResponseEntityResponseExtractor</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Type responseType)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (responseType != <span class="hljs-keyword">null</span> &amp;&amp; Void.class != responseType) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.delegate = <span class="hljs-keyword">new</span> HttpMessageConverterExtractor&lt;&gt;(responseType, getMessageConverters(), logger);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.delegate = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> ResponseEntity&lt;T&gt; <span class="hljs-title">extractData</span><span class="hljs-params">(ClientHttpResponse response)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.delegate != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; T body = <span class="hljs-keyword">this</span>.delegate.extractData(response);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ResponseEntity.status(response.getRawStatusCode()).headers(response.getHeaders()).body(body);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ResponseEntity.status(response.getRawStatusCode()).headers(response.getHeaders()).build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32857">在上述代码中，ResponseEntityResponseExtractor 中的 extractData 方法本质上是将数据提取部分的工作委托给了一个代理对象 delegate，而这个 delegate 的类型就是 HttpMessageConverterExtractor。</p>
<p data-nodeid="32858">从命名上看，我们不难看出 HttpMessageConverterExtractor 类的内部使用了 12 讲介绍的 HttpMessageConverter 实现消息的转换，如下代码所示（代码做了裁剪）：</p>
<pre class="lang-java" data-nodeid="32859"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HttpMessageConverterExtractor</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">ResponseExtractor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;HttpMessageConverter&lt;?&gt;&gt; messageConverters;
&nbsp;
<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SuppressWarnings({"unchecked", "rawtypes", "resource"})</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">extractData</span><span class="hljs-params">(ClientHttpResponse response)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MessageBodyClientHttpResponseWrapper responseWrapper = <span class="hljs-keyword">new</span> MessageBodyClientHttpResponseWrapper(response);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!responseWrapper.hasMessageBody() || responseWrapper.hasEmptyMessageBody()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MediaType contentType = getContentType(responseWrapper);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (HttpMessageConverter&lt;?&gt; messageConverter : <span class="hljs-keyword">this</span>.messageConverters) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (messageConverter <span class="hljs-keyword">instanceof</span> GenericHttpMessageConverter) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; GenericHttpMessageConverter&lt;?&gt; genericMessageConverter =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (GenericHttpMessageConverter&lt;?&gt;) messageConverter;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (genericMessageConverter.canRead(<span class="hljs-keyword">this</span>.responseType, <span class="hljs-keyword">null</span>, contentType)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (T) genericMessageConverter.read(<span class="hljs-keyword">this</span>.responseType, <span class="hljs-keyword">null</span>, responseWrapper);
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.responseClass != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (messageConverter.canRead(<span class="hljs-keyword">this</span>.responseClass, contentType)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (T) messageConverter.read((Class) <span class="hljs-keyword">this</span>.responseClass, responseWrapper);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
	…
}
</code></pre>
<p data-nodeid="32860">上述方法看上去有点复杂，但核心逻辑很简单，首先遍历 HttpMessageConveter 列表，然后判断其是否能够读取数据，如果能就调用 read 方法读取数据。</p>
<p data-nodeid="32861">最后，我们讨论下 HttpMessageConveter 中如何实现 read 方法。</p>
<p data-nodeid="32862">先来看 HttpMessageConveter 接口的抽象实现类 AbstractHttpMessageConverter，在它的 read 方法中我们同样定义了一个抽象方法 readInternal，如下代码所示：</p>
<pre class="lang-java" data-nodeid="32863"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> T <span class="hljs-title">read</span><span class="hljs-params">(Class&lt;? extends T&gt; clazz, HttpInputMessage inputMessage)</span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> IOException, HttpMessageNotReadableException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> readInternal(clazz, inputMessage);
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> T <span class="hljs-title">readInternal</span><span class="hljs-params">(Class&lt;? extends T&gt; clazz, HttpInputMessage inputMessage)</span> <span class="hljs-keyword">throws</span> IOException, HttpMessageNotReadableException</span>;
</code></pre>
<p data-nodeid="32864">在 12 讲中，我们提到 Spring 提供了一系列的 HttpMessageConveter 实现消息的转换，而最简单的实现方式是 StringHttpMessageConverter，该类的 read 方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="32865"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> String <span class="hljs-title">readInternal</span><span class="hljs-params">(Class&lt;? extends String&gt; clazz, HttpInputMessage inputMessage)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp; Charset charset = getContentTypeCharset(inputMessage.getHeaders().getContentType());
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> StreamUtils.copyToString(inputMessage.getBody(), charset);
}
</code></pre>
<p data-nodeid="32866">StringHttpMessageConverter 的实现过程：首先从输入消息 HttpInputMessage 中通过 getBody 方法获取消息体，也就是一个 ClientHttpResponse 对象，再通过 copyToString 方法从该对象中读取数据，并返回字符串结果。</p>
<p data-nodeid="32867">至此，通过 RestTemplate 发起、执行及响应整个 HTTP 请求的完整流程就介绍完毕了。</p>
<h3 data-nodeid="32868">从源码解析到日常开发</h3>
<p data-nodeid="32869">本节课涉及了大量关于如果处理 HTTP 请求的实现细节，而这些实现细节对开发人员理解 HTTP 协议、掌握 HTTP 协议及远程调用很大帮助，后期，你可以根据实际需要针对某些细节进一步深入分析。</p>
<p data-nodeid="32870">同时，通过对 RestTemplate 本身及围绕它的多个工具类的设计和实现过程进行梳理，也可以加深我们对抽象类与接口的标准设计理念的理解，并将这些设计理念付诸日常开发过程中。</p>
<h3 data-nodeid="32871">小结与预告</h3>
<p data-nodeid="32872">我们要想深入理解和掌握一个 HTTP 请求的处理过程，剖析 RestTemplate 工具类的实现很有必要。</p>
<p data-nodeid="32873">RestTemplate 中提供了创建请求对象、执行远程调用及处理响应结果这三大步骤的完整实现思路。本节课中我们对这些步骤进行了详细说明，并分析了其中包含的设计理念及实现技巧。</p>
<p data-nodeid="32874">这里给你留一道思考题：在 RestTemplate 中，如何通过 HttpMessageConverter 实现对响应结果的转换处理？</p>
<p data-nodeid="32875" class="">介绍完 Web 服务的构建和消费后，我们需要把目光转到一个应用程序的中间层组件，对于中间层组件而言，其一大应用场景是处理消息通信相关的需求。因此，从 14 讲开始我们将基于 Spring Boot 框架对目前主流的消息中间件及其使用方式进行逐一展开。</p>

---

### 精选评论


