<p data-nodeid="4249">在上一课时我们介绍了异步爬虫的基本原理和 asyncio 的基本用法，另外在最后简单提及了 aiohttp 实现网页爬取的过程，这一可是我们来介绍一下 aiohttp 的常见用法，以及通过一个实战案例来介绍下使用 aiohttp 完成网页异步爬取的过程。</p>
<h3 data-nodeid="4250">aiohttp</h3>
<p data-nodeid="4251">前面介绍的 asyncio 模块内部实现了对 TCP、UDP、SSL 协议的异步操作，但是对于 HTTP 请求的异步操作来说，我们就需要用到 aiohttp 来实现了。</p>
<p data-nodeid="4252">aiohttp 是一个基于 asyncio 的异步 HTTP 网络模块，它既提供了服务端，又提供了客户端。其中我们用服务端可以搭建一个支持异步处理的服务器，用于处理请求并返回响应，类似于 Django、Flask、Tornado 等一些 Web 服务器。而客户端我们就可以用来发起请求，就类似于 requests 来发起一个 HTTP 请求然后获得响应，但 requests 发起的是同步的网络请求，而 aiohttp 则发起的是异步的。</p>
<p data-nodeid="4253">本课时我们就主要来了解一下 aiohttp 客户端部分的使用。</p>
<h3 data-nodeid="4254">基本使用</h3>
<h4 data-nodeid="4255">基本实例</h4>
<p data-nodeid="4256">首先我们来看一个基本的 aiohttp 请求案例，代码如下：</p>
<pre class="lang-python" data-nodeid="4257"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">fetch</span>(<span class="hljs-params">session,&nbsp;url</span>):</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.get(url)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.text(),&nbsp;response.status
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession()&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;html,&nbsp;status&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;fetch(session,&nbsp;<span class="hljs-string">'https://cuiqingcai.com'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">f'html:&nbsp;<span class="hljs-subst">{html[:<span class="hljs-number">100</span>]}</span>...'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">f'status:&nbsp;<span class="hljs-subst">{status}</span>'</span>)
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;loop&nbsp;=&nbsp;asyncio.get_event_loop()
&nbsp;&nbsp;&nbsp;loop.run_until_complete(main())
</code></pre>
<p data-nodeid="4258">在这里我们使用 aiohttp 来爬取了我的个人博客，获得了源码和响应状态码并输出，运行结果如下：</p>
<pre class="lang-python" data-nodeid="4259"><code data-language="python">html:&nbsp;&lt;!DOCTYPE&nbsp;HTML&gt;
&lt;html&gt;
&lt;head&gt;
&lt;meta&nbsp;charset=<span class="hljs-string">"UTF-8"</span>&gt;
&lt;meta&nbsp;name=<span class="hljs-string">"baidu-tc-verification"</span>&nbsp;content=...
status:&nbsp;<span class="hljs-number">200</span>
</code></pre>
<p data-nodeid="4260">这里网页源码过长，只截取输出了一部分，可以看到我们成功获取了网页的源代码及响应状态码 200，也就完成了一次基本的 HTTP 请求，即我们成功使用 aiohttp 通过异步的方式进行了网页的爬取，当然这个操作用之前我们所讲的 requests 同样也可以做到。</p>
<p data-nodeid="4261">我们可以看到其请求方法的定义和之前有了明显的区别，主要有如下几点：</p>
<ul data-nodeid="4262">
<li data-nodeid="4263">
<p data-nodeid="4264">首先在导入库的时候，我们除了必须要引入 aiohttp 这个库之外，还必须要引入 asyncio 这个库，因为要实现异步爬取需要启动协程，而协程则需要借助于 asyncio 里面的事件循环来执行。除了事件循环，asyncio 里面也提供了很多基础的异步操作。</p>
</li>
<li data-nodeid="4265">
<p data-nodeid="4266">异步爬取的方法的定义和之前有所不同，在每个异步方法前面统一要加 async 来修饰。</p>
</li>
<li data-nodeid="4267">
<p data-nodeid="4268">with as 语句前面同样需要加 async 来修饰，在 Python 中，with as 语句用于声明一个上下文管理器，能够帮我们自动分配和释放资源，而在异步方法中，with as 前面加上 async 代表声明一个支持异步的上下文管理器。</p>
</li>
<li data-nodeid="4269">
<p data-nodeid="4270">对于一些返回 coroutine 的操作，前面需要加 await 来修饰，如 response 调用 text 方法，查询 API 可以发现其返回的是 coroutine 对象，那么前面就要加 await；而对于状态码来说，其返回值就是一个数值类型，那么前面就不需要加 await。所以，这里可以按照实际情况处理，参考官方文档说明，看看其对应的返回值是怎样的类型，然后决定加不加 await 就可以了。</p>
</li>
<li data-nodeid="4271">
<p data-nodeid="4272">最后，定义完爬取方法之后，实际上是 main 方法调用了 fetch 方法。要运行的话，必须要启用事件循环，事件循环就需要使用 asyncio 库，然后使用 run_until_complete 方法来运行。</p>
</li>
</ul>
<blockquote data-nodeid="4273">
<p data-nodeid="4274">注意在 Python 3.7 及以后的版本中，我们可以使用 asyncio.run(main()) 来代替最后的启动操作，不需要显式声明事件循环，run 方法内部会自动启动一个事件循环。但这里为了兼容更多的 Python 版本，依然还是显式声明了事件循环。</p>
</blockquote>
<h4 data-nodeid="4275">URL 参数设置</h4>
<p data-nodeid="4276">对于 URL 参数的设置，我们可以借助于 params 参数，传入一个字典即可，示例如下：</p>
<pre class="lang-python" data-nodeid="4277"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;params&nbsp;=&nbsp;{<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'germey'</span>,&nbsp;<span class="hljs-string">'age'</span>:&nbsp;<span class="hljs-number">25</span>}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession()&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.get(<span class="hljs-string">'https://httpbin.org/get'</span>,&nbsp;params=params)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-keyword">await</span>&nbsp;response.text())
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4278">运行结果如下：</p>
<pre class="lang-python" data-nodeid="4279"><code data-language="python">{
&nbsp;<span class="hljs-string">"args"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"age"</span>:&nbsp;<span class="hljs-string">"25"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"name"</span>:&nbsp;<span class="hljs-string">"germey"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"headers"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept"</span>:&nbsp;<span class="hljs-string">"*/*"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept-Encoding"</span>:&nbsp;<span class="hljs-string">"gzip,&nbsp;deflate"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Host"</span>:&nbsp;<span class="hljs-string">"httpbin.org"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"User-Agent"</span>:&nbsp;<span class="hljs-string">"Python/3.7&nbsp;aiohttp/3.6.2"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"X-Amzn-Trace-Id"</span>:&nbsp;<span class="hljs-string">"Root=1-5e85eed2-d240ac90f4dddf40b4723ef0"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"origin"</span>:&nbsp;<span class="hljs-string">"17.20.255.122"</span>,
&nbsp;<span class="hljs-string">"url"</span>:&nbsp;<span class="hljs-string">"https://httpbin.org/get?name=germey&amp;age=25"</span>
}
</code></pre>
<p data-nodeid="4280">这里可以看到，其实际请求的 URL 为&nbsp;<a href="https://httpbin.org/get?name=germey&amp;age=25" data-nodeid="4427">https://httpbin.org/get?name=germey&amp;age=25</a>，其 URL 请求参数就对应了 params 的内容。</p>
<h4 data-nodeid="4281">其他请求类型</h4>
<p data-nodeid="4282">另外 aiohttp 还支持其他的请求类型，如 POST、PUT、DELETE 等等，这个和 requests 的使用方式有点类似，示例如下：</p>
<pre class="lang-python" data-nodeid="4283"><code data-language="python">session.post(<span class="hljs-string">'http://httpbin.org/post'</span>,&nbsp;data=<span class="hljs-string">b'data'</span>)
session.put(<span class="hljs-string">'http://httpbin.org/put'</span>,&nbsp;data=<span class="hljs-string">b'data'</span>)
session.delete(<span class="hljs-string">'http://httpbin.org/delete'</span>)
session.head(<span class="hljs-string">'http://httpbin.org/get'</span>)
session.options(<span class="hljs-string">'http://httpbin.org/get'</span>)
session.patch(<span class="hljs-string">'http://httpbin.org/patch'</span>,&nbsp;data=<span class="hljs-string">b'data'</span>)
</code></pre>
<h4 data-nodeid="4284">POST 数据</h4>
<p data-nodeid="4285">对于 POST 表单提交，其对应的请求头的 Content-type 为 application/x-www-form-urlencoded，我们可以用如下方式来实现，代码示例如下：</p>
<pre class="lang-python" data-nodeid="4286"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;{<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'germey'</span>,&nbsp;<span class="hljs-string">'age'</span>:&nbsp;<span class="hljs-number">25</span>}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession()&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.post(<span class="hljs-string">'https://httpbin.org/post'</span>,&nbsp;data=data)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-keyword">await</span>&nbsp;response.text())
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4287">运行结果如下：</p>
<pre class="lang-python" data-nodeid="4288"><code data-language="python">{
&nbsp;<span class="hljs-string">"args"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"data"</span>:&nbsp;<span class="hljs-string">""</span>,
&nbsp;<span class="hljs-string">"files"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"form"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"age"</span>:&nbsp;<span class="hljs-string">"25"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"name"</span>:&nbsp;<span class="hljs-string">"germey"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"headers"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept"</span>:&nbsp;<span class="hljs-string">"*/*"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept-Encoding"</span>:&nbsp;<span class="hljs-string">"gzip,&nbsp;deflate"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Length"</span>:&nbsp;<span class="hljs-string">"18"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Type"</span>:&nbsp;<span class="hljs-string">"application/x-www-form-urlencoded"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Host"</span>:&nbsp;<span class="hljs-string">"httpbin.org"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"User-Agent"</span>:&nbsp;<span class="hljs-string">"Python/3.7&nbsp;aiohttp/3.6.2"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"X-Amzn-Trace-Id"</span>:&nbsp;<span class="hljs-string">"Root=1-5e85f0b2-9017ea603a68dc285e0552d0"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"json"</span>:&nbsp;null,
&nbsp;<span class="hljs-string">"origin"</span>:&nbsp;<span class="hljs-string">"17.20.255.58"</span>,
&nbsp;<span class="hljs-string">"url"</span>:&nbsp;<span class="hljs-string">"https://httpbin.org/post"</span>
}
</code></pre>
<p data-nodeid="4289">对于 POST JSON 数据提交，其对应的请求头的 Content-type 为 application/json，我们只需要将 post 方法的 data 参数改成 json 即可，代码示例如下：</p>
<pre class="lang-python" data-nodeid="4290"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;{<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'germey'</span>,&nbsp;<span class="hljs-string">'age'</span>:&nbsp;<span class="hljs-number">25</span>}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession()&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.post(<span class="hljs-string">'https://httpbin.org/post'</span>,&nbsp;json=data)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-keyword">await</span>&nbsp;response.text())
</code></pre>
<p data-nodeid="4291">运行结果如下：</p>
<pre class="lang-python" data-nodeid="4292"><code data-language="python">{
&nbsp;<span class="hljs-string">"args"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"data"</span>:&nbsp;<span class="hljs-string">"{\"name\":&nbsp;\"germey\",&nbsp;\"age\":&nbsp;25}"</span>,
&nbsp;<span class="hljs-string">"files"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"form"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"headers"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept"</span>:&nbsp;<span class="hljs-string">"*/*"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept-Encoding"</span>:&nbsp;<span class="hljs-string">"gzip,&nbsp;deflate"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Length"</span>:&nbsp;<span class="hljs-string">"29"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Type"</span>:&nbsp;<span class="hljs-string">"application/json"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Host"</span>:&nbsp;<span class="hljs-string">"httpbin.org"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"User-Agent"</span>:&nbsp;<span class="hljs-string">"Python/3.7&nbsp;aiohttp/3.6.2"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"X-Amzn-Trace-Id"</span>:&nbsp;<span class="hljs-string">"Root=1-5e85f03e-c91c9a20c79b9780dbed7540"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"json"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"age"</span>:&nbsp;<span class="hljs-number">25</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"name"</span>:&nbsp;<span class="hljs-string">"germey"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"origin"</span>:&nbsp;<span class="hljs-string">"17.20.255.58"</span>,
&nbsp;<span class="hljs-string">"url"</span>:&nbsp;<span class="hljs-string">"https://httpbin.org/post"</span>
}
</code></pre>
<h4 data-nodeid="4293">响应字段</h4>
<p data-nodeid="4294">对于响应来说，我们可以用如下的方法分别获取响应的状态码、响应头、响应体、响应体二进制内容、响应体 JSON 结果，代码示例如下：</p>
<pre class="lang-python" data-nodeid="4295"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;{<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'germey'</span>,&nbsp;<span class="hljs-string">'age'</span>:&nbsp;<span class="hljs-number">25</span>}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession()&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.post(<span class="hljs-string">'https://httpbin.org/post'</span>,&nbsp;data=data)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'status:'</span>,&nbsp;response.status)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'headers:'</span>,&nbsp;response.headers)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'body:'</span>,&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.text())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'bytes:'</span>,&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.read())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'json:'</span>,&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.json())
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4296">运行结果如下：</p>
<pre class="lang-python" data-nodeid="4297"><code data-language="python">status:&nbsp;<span class="hljs-number">200</span>
headers:&nbsp;&lt;CIMultiDictProxy(<span class="hljs-string">'Date'</span>:&nbsp;<span class="hljs-string">'Thu,&nbsp;02&nbsp;Apr&nbsp;2020&nbsp;14:13:05&nbsp;GMT'</span>,&nbsp;<span class="hljs-string">'Content-Type'</span>:&nbsp;<span class="hljs-string">'application/json'</span>,&nbsp;<span class="hljs-string">'Content-Length'</span>:&nbsp;<span class="hljs-string">'503'</span>,&nbsp;<span class="hljs-string">'Connection'</span>:&nbsp;<span class="hljs-string">'keep-alive'</span>,&nbsp;<span class="hljs-string">'Server'</span>:&nbsp;<span class="hljs-string">'gunicorn/19.9.0'</span>,&nbsp;<span class="hljs-string">'Access-Control-Allow-Origin'</span>:&nbsp;<span class="hljs-string">'*'</span>,&nbsp;<span class="hljs-string">'Access-Control-Allow-Credentials'</span>:&nbsp;<span class="hljs-string">'true'</span>)&gt;
body:&nbsp;{
&nbsp;<span class="hljs-string">"args"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"data"</span>:&nbsp;<span class="hljs-string">""</span>,
&nbsp;<span class="hljs-string">"files"</span>:&nbsp;{},
&nbsp;<span class="hljs-string">"form"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"age"</span>:&nbsp;<span class="hljs-string">"25"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"name"</span>:&nbsp;<span class="hljs-string">"germey"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"headers"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept"</span>:&nbsp;<span class="hljs-string">"*/*"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Accept-Encoding"</span>:&nbsp;<span class="hljs-string">"gzip,&nbsp;deflate"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Length"</span>:&nbsp;<span class="hljs-string">"18"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Content-Type"</span>:&nbsp;<span class="hljs-string">"application/x-www-form-urlencoded"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Host"</span>:&nbsp;<span class="hljs-string">"httpbin.org"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"User-Agent"</span>:&nbsp;<span class="hljs-string">"Python/3.7&nbsp;aiohttp/3.6.2"</span>,
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"X-Amzn-Trace-Id"</span>:&nbsp;<span class="hljs-string">"Root=1-5e85f2f1-f55326ff5800b15886c8e029"</span>
&nbsp;},
&nbsp;<span class="hljs-string">"json"</span>:&nbsp;null,
&nbsp;<span class="hljs-string">"origin"</span>:&nbsp;<span class="hljs-string">"17.20.255.58"</span>,
&nbsp;<span class="hljs-string">"url"</span>:&nbsp;<span class="hljs-string">"https://httpbin.org/post"</span>
}
bytes:&nbsp;<span class="hljs-string">b'{\n&nbsp;&nbsp;"args":&nbsp;{},&nbsp;\n&nbsp;&nbsp;"data":&nbsp;"",&nbsp;\n&nbsp;&nbsp;"files":&nbsp;{},&nbsp;\n&nbsp;&nbsp;"form":&nbsp;{\n&nbsp;&nbsp;&nbsp;&nbsp;"age":&nbsp;"25",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"name":&nbsp;"germey"\n&nbsp;&nbsp;},&nbsp;\n&nbsp;&nbsp;"headers":&nbsp;{\n&nbsp;&nbsp;&nbsp;&nbsp;"Accept":&nbsp;"*/*",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"Accept-Encoding":&nbsp;"gzip,&nbsp;deflate",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"Content-Length":&nbsp;"18",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"Content-Type":&nbsp;"application/x-www-form-urlencoded",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"Host":&nbsp;"httpbin.org",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"User-Agent":&nbsp;"Python/3.7&nbsp;aiohttp/3.6.2",&nbsp;\n&nbsp;&nbsp;&nbsp;&nbsp;"X-Amzn-Trace-Id":&nbsp;"Root=1-5e85f2f1-f55326ff5800b15886c8e029"\n&nbsp;&nbsp;},&nbsp;\n&nbsp;&nbsp;"json":&nbsp;null,&nbsp;\n&nbsp;&nbsp;"origin":&nbsp;"17.20.255.58",&nbsp;\n&nbsp;&nbsp;"url":&nbsp;"https://httpbin.org/post"\n}\n'</span>
json:&nbsp;{<span class="hljs-string">'args'</span>:&nbsp;{},&nbsp;<span class="hljs-string">'data'</span>:&nbsp;<span class="hljs-string">''</span>,&nbsp;<span class="hljs-string">'files'</span>:&nbsp;{},&nbsp;<span class="hljs-string">'form'</span>:&nbsp;{<span class="hljs-string">'age'</span>:&nbsp;<span class="hljs-string">'25'</span>,&nbsp;<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'germey'</span>},&nbsp;<span class="hljs-string">'headers'</span>:&nbsp;{<span class="hljs-string">'Accept'</span>:&nbsp;<span class="hljs-string">'*/*'</span>,&nbsp;<span class="hljs-string">'Accept-Encoding'</span>:&nbsp;<span class="hljs-string">'gzip,&nbsp;deflate'</span>,&nbsp;<span class="hljs-string">'Content-Length'</span>:&nbsp;<span class="hljs-string">'18'</span>,&nbsp;<span class="hljs-string">'Content-Type'</span>:&nbsp;<span class="hljs-string">'application/x-www-form-urlencoded'</span>,&nbsp;<span class="hljs-string">'Host'</span>:&nbsp;<span class="hljs-string">'httpbin.org'</span>,&nbsp;<span class="hljs-string">'User-Agent'</span>:&nbsp;<span class="hljs-string">'Python/3.7&nbsp;aiohttp/3.6.2'</span>,&nbsp;<span class="hljs-string">'X-Amzn-Trace-Id'</span>:&nbsp;<span class="hljs-string">'Root=1-5e85f2f1-f55326ff5800b15886c8e029'</span>},&nbsp;<span class="hljs-string">'json'</span>:&nbsp;<span class="hljs-literal">None</span>,&nbsp;<span class="hljs-string">'origin'</span>:&nbsp;<span class="hljs-string">'17.20.255.58'</span>,&nbsp;<span class="hljs-string">'url'</span>:&nbsp;<span class="hljs-string">'https://httpbin.org/post'</span>}
</code></pre>
<p data-nodeid="4298">这里我们可以看到有些字段前面需要加 await，有的则不需要。其原则是，如果其返回的是一个 coroutine 对象（如 async 修饰的方法），那么前面就要加 await，具体可以看 aiohttp 的 API，其链接为：<a href="https://docs.aiohttp.org/en/stable/client_reference.html" data-nodeid="4444">https://docs.aiohttp.org/en/stable/client_reference.html</a>。</p>
<h4 data-nodeid="4299">超时设置</h4>
<p data-nodeid="4300">对于超时的设置，我们可以借助于 ClientTimeout 对象，比如这里我要设置 1 秒的超时，可以这么来实现：</p>
<pre class="lang-python" data-nodeid="4301"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;timeout&nbsp;=&nbsp;aiohttp.ClientTimeout(total=<span class="hljs-number">1</span>)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;aiohttp.ClientSession(timeout=timeout)&nbsp;<span class="hljs-keyword">as</span>&nbsp;session:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.get(<span class="hljs-string">'https://httpbin.org/get'</span>)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'status:'</span>,&nbsp;response.status)
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4302">如果在 1 秒之内成功获取响应的话，运行结果如下：</p>
<pre class="lang-python" data-nodeid="4303"><code data-language="python"><span class="hljs-number">200</span>
</code></pre>
<p data-nodeid="4304">如果超时的话，会抛出 TimeoutError 异常，其类型为 asyncio.TimeoutError，我们再进行异常捕获即可。</p>
<p data-nodeid="4305">另外 ClientTimeout 对象声明时还有其他参数，如 connect、socket_connect 等，详细说明可以参考官方文档：<a href="https://docs.aiohttp.org/en/stable/client_quickstart.html#timeouts" data-nodeid="4457">https://docs.aiohttp.org/en/stable/client_quickstart.html#timeouts</a>。</p>
<h4 data-nodeid="4306">并发限制</h4>
<p data-nodeid="4307">由于 aiohttp 可以支持非常大的并发，比如上万、十万、百万都是能做到的，但这么大的并发量，目标网站是很可能在短时间内无法响应的，而且很可能瞬时间将目标网站爬挂掉。所以我们需要控制一下爬取的并发量。</p>
<p data-nodeid="4308">在一般情况下，我们可以借助于 asyncio 的 Semaphore 来控制并发量，代码示例如下：</p>
<pre class="lang-python" data-nodeid="4309"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">import</span>&nbsp;aiohttp
CONCURRENCY&nbsp;=&nbsp;<span class="hljs-number">5</span>
URL&nbsp;=&nbsp;<span class="hljs-string">'https://www.baidu.com'</span>
semaphore&nbsp;=&nbsp;asyncio.Semaphore(CONCURRENCY)
session&nbsp;=&nbsp;<span class="hljs-literal">None</span>
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_api</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;semaphore:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(<span class="hljs-string">'scraping'</span>,&nbsp;URL)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.get(URL)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;asyncio.sleep(<span class="hljs-number">1</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.text()
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">global</span>&nbsp;session
&nbsp;&nbsp;&nbsp;session&nbsp;=&nbsp;aiohttp.ClientSession()
&nbsp;&nbsp;&nbsp;scrape_index_tasks&nbsp;=&nbsp;[asyncio.ensure_future(scrape_api())&nbsp;<span class="hljs-keyword">for</span>&nbsp;_&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">10000</span>)]
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;asyncio.gather(*scrape_index_tasks)
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4310">在这里我们声明了 CONCURRENCY 代表爬取的最大并发量为 5，同时声明爬取的目标 URL 为百度。接着我们借助于 Semaphore 创建了一个信号量对象，赋值为 semaphore，这样我们就可以用它来控制最大并发量了。怎么使用呢？我们这里把它直接放置在对应的爬取方法里面，使用 async with 语句将 semaphore 作为上下文对象即可。这样的话，信号量可以控制进入爬取的最大协程数量，最大数量就是我们声明的 CONCURRENCY 的值。</p>
<p data-nodeid="4311">在 main 方法里面，我们声明了 10000 个 task，传递给 gather 方法运行。倘若不加以限制，这 10000 个 task 会被同时执行，并发数量太大。但有了信号量的控制之后，同时运行的 task 的数量最大会被控制在 5 个，这样就能给 aiohttp 限制速度了。</p>
<p data-nodeid="4312">在这里，aiohttp 的基本使用就介绍这么多，更详细的内容还是推荐你到官方文档查阅，链接：<a href="https://docs.aiohttp.org/" data-nodeid="4467">https://docs.aiohttp.org/</a>。</p>
<h3 data-nodeid="4313">爬取实战</h3>
<p data-nodeid="4314">上面我们介绍了 aiohttp 的基本用法之后，下面我们来根据一个实例实现异步爬虫的实战演练吧。</p>
<p data-nodeid="4315">本次我们要爬取的网站是：<a href="https://dynamic5.scrape.center/" data-nodeid="4474">https://dynamic5.scrape.center/</a>，页面如图所示。<br>
<img src="https://s0.lgstatic.com/i/image3/M01/82/0F/Cgq2xl6G-LCALW48AAcm_HQNyJ0576.png" alt="" data-nodeid="4478"></p>
<p data-nodeid="4316">这是一个书籍网站，整个网站包含了数千本书籍信息，网站是 JavaScript 渲染的，数据可以通过 Ajax 接口获取到，并且接口没有设置任何反爬措施和加密参数，另外由于这个网站比之前的电影案例网站数据量大一些，所以更加适合做异步爬取。</p>
<p data-nodeid="4317">本课时我们要完成的目标有：</p>
<ul data-nodeid="4318">
<li data-nodeid="4319">
<p data-nodeid="4320">使用 aiohttp 完成全站的书籍数据爬取。</p>
</li>
<li data-nodeid="4321">
<p data-nodeid="4322">将数据通过异步的方式保存到 MongoDB 中。</p>
</li>
</ul>
<p data-nodeid="4323">在本课时开始之前，请确保你已经做好了如下准备工作：</p>
<ul data-nodeid="4324">
<li data-nodeid="4325">
<p data-nodeid="4326">安装好了 Python（最低为 Python 3.6 版本，最好为 3.7 版本或以上），并能成功运行 Python 程序。</p>
</li>
<li data-nodeid="4327">
<p data-nodeid="4328">了解了 Ajax 爬取的一些基本原理和模拟方法。</p>
</li>
<li data-nodeid="4329">
<p data-nodeid="4330">了解了异步爬虫的基本原理和 asyncio 库的基本用法。</p>
</li>
<li data-nodeid="4331">
<p data-nodeid="4332">了解了 aiohttp 库的基本用法。</p>
</li>
<li data-nodeid="4333">
<p data-nodeid="4334">安装并成功运行了 MongoDB 数据库，并安装了异步存储库 motor。</p>
</li>
</ul>
<blockquote data-nodeid="4335">
<p data-nodeid="4336">注：这里要实现 MongoDB 异步存储，需要异步 MongoDB 存储库，叫作 motor，安装命令为：<code data-backticks="1" data-nodeid="4490">pip3&nbsp;install&nbsp;motor</code></p>
</blockquote>
<h3 data-nodeid="4337">页面分析</h3>
<p data-nodeid="4338">在之前我们讲解了 Ajax 的基本分析方法，本课时的站点结构和之前 Ajax 分析的站点结构类似，都是列表页加详情页的结构，加载方式都是 Ajax，所以我们能轻松分析到如下信息：</p>
<ul data-nodeid="4339">
<li data-nodeid="4340">
<p data-nodeid="4341">列表页的 Ajax 请求接口格式为：<a href="https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=" data-nodeid="4498">https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=</a>{offset}，limit 的值即为每一页的书的个数，offset 的值为每一页的偏移量，其计算公式为 offset = limit * (page - 1) ，如第 1 页 offset 的值为 0，第 2 页 offset 的值为 18，以此类推。</p>
</li>
<li data-nodeid="4342">
<p data-nodeid="4343">列表页 Ajax 接口返回的数据里 results 字段包含当前页 18 本书的信息，其中每本书的数据里面包含一个字段 id，这个 id 就是书本身的 ID，可以用来进一步请求详情页。</p>
</li>
<li data-nodeid="4344">
<p data-nodeid="4345">详情页的 Ajax 请求接口格式为：<a href="https://dynamic5.scrape.center/api/book/" data-nodeid="4506">https://dynamic5.scrape.center/api/book/</a>{id}，id 即为书的 ID，可以从列表页的返回结果中获取。</p>
</li>
</ul>
<p data-nodeid="4346">如果你掌握了 Ajax 爬取实战一课时的内容话，上面的内容应该很容易分析出来。如有难度，可以复习下之前的知识。</p>
<h3 data-nodeid="4347">实现思路</h3>
<p data-nodeid="4348">其实一个完善的异步爬虫应该能够充分利用资源进行全速爬取，其思路是维护一个动态变化的爬取队列，每产生一个新的 task 就会将其放入队列中，有专门的爬虫消费者从队列中获取 task 并执行，能做到在最大并发量的前提下充分利用等待时间进行额外的爬取处理。</p>
<p data-nodeid="4349">但上面的实现思路整体较为烦琐，需要设计爬取队列、回调函数、消费者等机制，需要实现的功能较多。由于我们刚刚接触 aiohttp 的基本用法，本课时也主要是了解 aiohttp 的实战应用，所以这里我们将爬取案例的实现稍微简化一下。</p>
<p data-nodeid="4350">在这里我们将爬取的逻辑拆分成两部分，第一部分为爬取列表页，第二部分为爬取详情页。由于异步爬虫的关键点在于并发执行，所以我们可以将爬取拆分为两个阶段：</p>
<ul data-nodeid="4351">
<li data-nodeid="4352">
<p data-nodeid="4353">第一阶段为所有列表页的异步爬取，我们可以将所有的列表页的爬取任务集合起来，声明为 task 组成的列表，进行异步爬取。</p>
</li>
<li data-nodeid="4354">
<p data-nodeid="4355">第二阶段则是拿到上一步列表页的所有内容并解析，拿到所有书的 id 信息，组合为所有详情页的爬取任务集合，声明为 task 组成的列表，进行异步爬取，同时爬取的结果也以异步的方式存储到 MongoDB 里面。</p>
</li>
</ul>
<p data-nodeid="4356">因为两个阶段的拆分之后需要串行执行，所以可能不能达到协程的最佳调度方式和资源利用情况，但也差不了很多。但这个实现思路比较简单清晰，代码实现也比较简单，能够帮我们快速了解 aiohttp 的基本使用。</p>
<h3 data-nodeid="4357">基本配置</h3>
<p data-nodeid="4358">首先我们先配置一些基本的变量并引入一些必需的库，代码如下：</p>
<pre class="lang-python" data-nodeid="4359"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">import</span>&nbsp;aiohttp
<span class="hljs-keyword">import</span>&nbsp;logging
logging.basicConfig(level=logging.INFO,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format=<span class="hljs-string">'%(asctime)s&nbsp;-&nbsp;%(levelname)s:&nbsp;%(message)s'</span>)
INDEX_URL&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic5.scrape.center/api/book/?limit=18&amp;offset={offset}'</span>
DETAIL_URL&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic5.scrape.center/api/book/{id}'</span>
PAGE_SIZE&nbsp;=&nbsp;<span class="hljs-number">18</span>
PAGE_NUMBER&nbsp;=&nbsp;<span class="hljs-number">100</span>
CONCURRENCY&nbsp;=&nbsp;<span class="hljs-number">5</span>
</code></pre>
<p data-nodeid="4360">在这里我们导入了 asyncio、aiohttp、logging 这三个库，然后定义了 logging 的基本配置。接着定义了 URL、爬取页码数量 PAGE_NUMBER、并发量 CONCURRENCY 等信息。</p>
<h3 data-nodeid="4361">爬取列表页</h3>
<p data-nodeid="4362">首先，第一阶段我们就来爬取列表页，还是和之前一样，我们先定义一个通用的爬取方法，代码如下：</p>
<pre class="lang-python" data-nodeid="4363"><code data-language="python">semaphore&nbsp;=&nbsp;asyncio.Semaphore(CONCURRENCY)
session&nbsp;=&nbsp;<span class="hljs-literal">None</span>
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_api</span>(<span class="hljs-params">url</span>):</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;semaphore:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'scraping&nbsp;%s'</span>,&nbsp;url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-keyword">with</span>&nbsp;session.get(url)&nbsp;<span class="hljs-keyword">as</span>&nbsp;response:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;response.json()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">except</span>&nbsp;aiohttp.ClientError:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.error(<span class="hljs-string">'error&nbsp;occurred&nbsp;while&nbsp;scraping&nbsp;%s'</span>,&nbsp;url,&nbsp;exc_info=<span class="hljs-literal">True</span>)
</code></pre>
<p data-nodeid="4364">在这里我们声明了一个信号量，用来控制最大并发数量。</p>
<p data-nodeid="4365">接着我们定义了 scrape_api 方法，该方法接收一个参数 url。首先使用 async with 引入信号量作为上下文，接着调用了 session 的 get 方法请求这个 url，然后返回响应的 JSON 格式的结果。另外这里还进行了异常处理，捕获了 ClientError，如果出现错误，会输出异常信息。</p>
<p data-nodeid="4366">接着，对于列表页的爬取，实现如下：</p>
<pre class="lang-python" data-nodeid="4367"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_index</span>(<span class="hljs-params">page</span>):</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;INDEX_URL.format(offset=PAGE_SIZE&nbsp;*&nbsp;(page&nbsp;-&nbsp;<span class="hljs-number">1</span>))
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_api(url)
</code></pre>
<p data-nodeid="4368">这里定义了一个 scrape_index 方法用于爬取列表页，它接收一个参数为 page，然后构造了列表页的 URL，将其传给 scrape_api 方法即可。这里注意方法同样需要用 async 修饰，调用的 scrape_api 方法前面需要加 await，因为 scrape_api 调用之后本身会返回一个 coroutine。另外由于 scrape_api 返回结果就是 JSON 格式，因此 scrape_index 的返回结果就是我们想要爬取的信息，不需要再额外解析了。</p>
<p data-nodeid="4369">好，接着我们定义一个 main 方法，将上面的方法串联起来调用一下，实现如下：</p>
<pre class="lang-python" data-nodeid="4370"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;json
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">global</span>&nbsp;session
&nbsp;&nbsp;&nbsp;session&nbsp;=&nbsp;aiohttp.ClientSession()
&nbsp;&nbsp;&nbsp;scrape_index_tasks&nbsp;=&nbsp;[asyncio.ensure_future(scrape_index(page))&nbsp;<span class="hljs-keyword">for</span>&nbsp;page&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">1</span>,&nbsp;PAGE_NUMBER&nbsp;+&nbsp;<span class="hljs-number">1</span>)]
&nbsp;&nbsp;&nbsp;results&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;asyncio.gather(*scrape_index_tasks)
&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'results&nbsp;%s'</span>,&nbsp;json.dumps(results,&nbsp;ensure_ascii=<span class="hljs-literal">False</span>,&nbsp;indent=<span class="hljs-number">2</span>))

<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="4371">这里我们首先声明了 session 对象，即最初声明的全局变量，将 session 作为全局变量的话我们就不需要每次在各个方法里面传递了，实现比较简单。</p>
<p data-nodeid="4372">接着我们定义了 scrape_index_tasks，它就是爬取列表页的所有 task，接着我们调用 asyncio 的 gather 方法并传入 task 列表，将结果赋值为 results，它是所有 task 返回结果组成的列表。</p>
<p data-nodeid="4373">最后我们调用 main 方法，使用事件循环启动该 main 方法对应的协程即可。</p>
<p data-nodeid="4374">运行结果如下：</p>
<pre class="lang-python" data-nodeid="4375"><code data-language="python">2020-04-03&nbsp;03:45:54,692&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=0
2020-04-03&nbsp;03:45:54,707&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=18
2020-04-03&nbsp;03:45:54,707&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=36
2020-04-03&nbsp;03:45:54,708&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=54
2020-04-03&nbsp;03:45:54,708&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=72
2020-04-03&nbsp;03:45:56,431&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=90
2020-04-03&nbsp;03:45:56,435&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/?limit=18&amp;offset=108
</code></pre>
<p data-nodeid="4376">可以看到这里就开始异步爬取了，并发量是由我们控制的，目前为 5，当然也可以进一步调高并发量，在网站能承受的情况下，爬取速度会进一步加快。</p>
<p data-nodeid="4377">最后 results 就是所有列表页得到的结果，我们将其赋值为 results 对象，接着我们就可以用它来进行第二阶段的爬取了。</p>
<h3 data-nodeid="4378">爬取详情页</h3>
<p data-nodeid="4379">第二阶段就是爬取详情页并保存数据了，由于每个详情页对应一本书，每本书需要一个 ID，而这个 ID 又正好存在 results 里面，所以下面我们就需要将所有详情页的 ID 获取出来。</p>
<p data-nodeid="4380">在 main 方法里增加 results 的解析代码，实现如下：</p>
<pre class="lang-python" data-nodeid="4381"><code data-language="python">ids&nbsp;=&nbsp;[]
<span class="hljs-keyword">for</span>&nbsp;index_data&nbsp;<span class="hljs-keyword">in</span>&nbsp;results:
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;<span class="hljs-keyword">not</span>&nbsp;index_data:&nbsp;<span class="hljs-keyword">continue</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;item&nbsp;<span class="hljs-keyword">in</span>&nbsp;index_data.get(<span class="hljs-string">'results'</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ids.append(item.get(<span class="hljs-string">'id'</span>))
</code></pre>
<p data-nodeid="4382">这样 ids 就是所有书的 id 了，然后我们用所有的 id 来构造所有详情页对应的 task，来进行异步爬取即可。</p>
<p data-nodeid="4383">那么这里再定义一个爬取详情页和保存数据的方法，实现如下：</p>
<pre class="lang-python" data-nodeid="4384"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;motor.motor_asyncio&nbsp;<span class="hljs-keyword">import</span>&nbsp;AsyncIOMotorClient
MONGO_CONNECTION_STRING&nbsp;=&nbsp;<span class="hljs-string">'mongodb://localhost:27017'</span>
MONGO_DB_NAME&nbsp;=&nbsp;<span class="hljs-string">'books'</span>
MONGO_COLLECTION_NAME&nbsp;=&nbsp;<span class="hljs-string">'books'</span>
client&nbsp;=&nbsp;AsyncIOMotorClient(MONGO_CONNECTION_STRING)
db&nbsp;=&nbsp;client[MONGO_DB_NAME]
collection&nbsp;=&nbsp;db[MONGO_COLLECTION_NAME]
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">save_data</span>(<span class="hljs-params">data</span>):</span>
&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'saving&nbsp;data&nbsp;%s'</span>,&nbsp;data)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;data:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;collection.update_one({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'id'</span>:&nbsp;data.get(<span class="hljs-string">'id'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'$set'</span>:&nbsp;data
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},&nbsp;upsert=<span class="hljs-literal">True</span>)
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_detail</span>(<span class="hljs-params">id</span>):</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;DETAIL_URL.format(id=id)
&nbsp;&nbsp;&nbsp;data&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_api(url)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;save_data(data)
</code></pre>
<p data-nodeid="4385">这里我们定义了 scrape_detail 方法用于爬取详情页数据并调用 save_data 方法保存数据，save_data 方法用于将数据库保存到 MongoDB 里面。</p>
<p data-nodeid="4386">在这里我们用到了支持异步的 MongoDB 存储库 motor，MongoDB 的连接声明和 pymongo 是类似的，保存数据的调用方法也是基本一致，不过整个都换成了异步方法。</p>
<p data-nodeid="4387">好，接着我们就在 main 方法里面增加 scrape_detail 方法的调用即可，实现如下：</p>
<pre class="lang-python" data-nodeid="4388"><code data-language="python">scrape_detail_tasks&nbsp;=&nbsp;[asyncio.ensure_future(scrape_detail(id))&nbsp;<span class="hljs-keyword">for</span>&nbsp;id&nbsp;<span class="hljs-keyword">in</span>&nbsp;ids]
<span class="hljs-keyword">await</span>&nbsp;asyncio.wait(scrape_detail_tasks)
<span class="hljs-keyword">await</span>&nbsp;session.close()
</code></pre>
<p data-nodeid="4389">在这里我们先声明了 scrape_detail_tasks，即所有详情页的爬取 task 组成的列表，接着调用了 asyncio 的 wait 方法调用执行即可，当然这里也可以用 gather 方法，效果是一样的，只不过返回结果略有差异。最后全部执行完毕关闭 session 即可。</p>
<p data-nodeid="4390">一些详情页的爬取过程运行如下：</p>
<pre class="lang-python" data-nodeid="4391"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">32</span>,<span class="hljs-number">576</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">2301475</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">32</span>,<span class="hljs-number">576</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">2351866</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">32</span>,<span class="hljs-number">577</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">2828384</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">32</span>,<span class="hljs-number">577</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">3040352</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">32</span>,<span class="hljs-number">578</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">3074810</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">44</span>,<span class="hljs-number">858</span>&nbsp;-&nbsp;INFO:&nbsp;saving&nbsp;data&nbsp;{<span class="hljs-string">'id'</span>:&nbsp;<span class="hljs-string">'3040352'</span>,&nbsp;<span class="hljs-string">'comments'</span>:&nbsp;[{<span class="hljs-string">'id'</span>:&nbsp;<span class="hljs-string">'387952888'</span>,&nbsp;<span class="hljs-string">'content'</span>:&nbsp;<span class="hljs-string">'温馨文，青梅竹马神马的很有爱~'</span>},&nbsp;...,&nbsp;{<span class="hljs-string">'id'</span>:&nbsp;<span class="hljs-string">'2005314253'</span>,&nbsp;<span class="hljs-string">'content'</span>:&nbsp;<span class="hljs-string">'沈晋&amp;秦央，文比较短，平平淡淡，贴近生活，短文的缺点不细腻'</span>}],&nbsp;<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'那些风花雪月'</span>,&nbsp;<span class="hljs-string">'authors'</span>:&nbsp;[<span class="hljs-string">'\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;公子欢喜'</span>],&nbsp;<span class="hljs-string">'translators'</span>:&nbsp;[],&nbsp;<span class="hljs-string">'publisher'</span>:&nbsp;<span class="hljs-string">'龍馬出版社'</span>,&nbsp;<span class="hljs-string">'tags'</span>:&nbsp;[<span class="hljs-string">'公子欢喜'</span>,&nbsp;<span class="hljs-string">'耽美'</span>,&nbsp;<span class="hljs-string">'BL'</span>,&nbsp;<span class="hljs-string">'小说'</span>,&nbsp;<span class="hljs-string">'现代'</span>,&nbsp;<span class="hljs-string">'校园'</span>,&nbsp;<span class="hljs-string">'耽美小说'</span>,&nbsp;<span class="hljs-string">'那些风花雪月'</span>],&nbsp;<span class="hljs-string">'url'</span>:&nbsp;<span class="hljs-string">'https://book.douban.com/subject/3040352/'</span>,&nbsp;<span class="hljs-string">'isbn'</span>:&nbsp;<span class="hljs-string">'9789866685156'</span>,&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;<span class="hljs-string">'https://img9.doubanio.com/view/subject/l/public/s3029724.jpg'</span>,&nbsp;<span class="hljs-string">'page_number'</span>:&nbsp;<span class="hljs-literal">None</span>,&nbsp;<span class="hljs-string">'price'</span>:&nbsp;<span class="hljs-literal">None</span>,&nbsp;<span class="hljs-string">'score'</span>:&nbsp;<span class="hljs-string">'8.1'</span>,&nbsp;<span class="hljs-string">'introduction'</span>:&nbsp;<span class="hljs-string">''</span>,&nbsp;<span class="hljs-string">'catalog'</span>:&nbsp;<span class="hljs-literal">None</span>,&nbsp;<span class="hljs-string">'published_at'</span>:&nbsp;<span class="hljs-string">'2008-03-26T16:00:00Z'</span>,&nbsp;<span class="hljs-string">'updated_at'</span>:&nbsp;<span class="hljs-string">'2020-03-21T16:59:39.584722Z'</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-03</span>&nbsp;<span class="hljs-number">04</span>:<span class="hljs-number">00</span>:<span class="hljs-number">44</span>,<span class="hljs-number">859</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic5.scrape.center/api/book/<span class="hljs-number">2994915</span>
...
</code></pre>
<p data-nodeid="4392">最后我们观察下，爬取到的数据也都保存到 MongoDB 数据库里面了，如图所示：</p>
<p data-nodeid="4393"><img src="https://s0.lgstatic.com/i/image3/M01/08/F9/Ciqah16G-LCABdDSAAStnDYrg58898.png" alt="" data-nodeid="4576"></p>
<p data-nodeid="4394">至此，我们就使用 aiohttp 完成了书籍网站的异步爬取。</p>
<h3 data-nodeid="4395">总结</h3>
<p data-nodeid="4396">本课时的内容较多，我们了解了 aiohttp 的基本用法，然后通过一个实例讲解了 aiohttp 异步爬虫的具体实现。学习过程我们可以发现，相比普通的单线程爬虫来说，使用异步可以大大提高爬取效率，后面我们也可以多多使用。</p>
<p data-nodeid="4397" class="te-preview-highlight">本课时代码：<a href="https://github.com/Germey/ScrapeDynamic5" data-nodeid="4583">https://github.com/Germey/ScrapeDynamic5</a>。</p>

---

### 精选评论

##### bing wu：
> 老师，就比如最近很火的网剧，龙岭迷窟，有的小网站就可以免费观看，他是不是也是爬虫爬到的呀？那么这个视频链接他们是怎么得到的呢，不应该视频链接加密了么🙋🙋

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个很可能是他有 vip 账号，然后他用 vip 账号登录了之后把它能看到的内容如视频等爬下来了，虽然有加密，但是也可以通过 JavaScript 逆向，或者 Selenium 等模拟的方式爬取到的

##### **9328：
> 老师我前面用了mysql存储了（我只会关系型数据库），到这里应该怎么异步存储呀，我这是自己给自己设置难度呀（手动狗头）

##### **林：
> 有学习群么，爬虫队列的目的是不是为了控制爬虫的速度和爬取的优先级。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 添加微信：lagoujuzi，回复“爬虫”，拉你进群～

##### **6968：
> 协程异步和多线程并发效果一样的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不一样，前者效率会更高

##### **耿爱生活：
> 不行，已经吃不消了😂😂小编快鼓励鼓励我😫😫😫

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 发现问题是好事哦，可以结合网络搜索补充自己的知识储备，课后可以利用文字版反复理解，学习效果会更好。

##### **8255：
> 在安装motor的时候出现错误，是和pymongo的版本对不上吗<br><br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不太清楚你是什么错误，如果你 pymongo 版本比较低的话，可以试着降低下 motor 版本。

