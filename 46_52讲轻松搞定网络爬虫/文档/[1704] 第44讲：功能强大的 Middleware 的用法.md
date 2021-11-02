<p data-nodeid="328225" class="">我们在 Scrapy 架构中，可以看到有一个叫作 Middleware 的概念，中文翻译过来就叫作中间件，在 Scrapy 中有两种 Middleware，一种是 Spider Middleware，另一种是 Downloader Middleware，本节课我们分别来介绍下。</p>
<h3 data-nodeid="328226">Spider Middleware 的用法</h3>
<p data-nodeid="328227">Spider Middleware 是介入 Scrapy 的 Spider 处理机制的钩子框架。</p>
<p data-nodeid="328228">当 Downloader 生成 Response 之后，Response 会被发送给 Spider，在发送给 Spider 之前，Response 会首先经过 Spider Middleware 处理，当 Spider 处理生成 Item 和 Request 之后，Item 和 Request 还会经过 Spider Middleware 的处理。</p>
<p data-nodeid="328229">Spider Middleware 有如下三个作用。</p>
<ul data-nodeid="328230">
<li data-nodeid="328231">
<p data-nodeid="328232">我们可以在 Downloader 生成的 Response 发送给 Spider 之前，也就是在 Response 发送给 Spider 之前对 Response 进行处理。</p>
</li>
<li data-nodeid="328233">
<p data-nodeid="328234">我们可以在 Spider 生成的 Request 发送给 Scheduler 之前，也就是在 Request 发送给 Scheduler 之前对 Request 进行处理。</p>
</li>
<li data-nodeid="328235">
<p data-nodeid="328236">我们可以在 Spider 生成的 Item 发送给 Item Pipeline 之前，也就是在 Item 发送给 Item Pipeline 之前对 Item 进行处理。</p>
</li>
</ul>
<h4 data-nodeid="328237">使用说明</h4>
<p data-nodeid="328238">需要说明的是，Scrapy 其实已经提供了许多 Spider Middleware，它们被 SPIDER_MIDDLEWARES_BASE 这个变量所定义。</p>
<p data-nodeid="328239">SPIDER_MIDDLEWARES_BASE 变量的内容如下：</p>
<pre class="lang-java" data-nodeid="328240"><code data-language="java">{
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware'</span>: <span class="hljs-number">50</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.spidermiddlewares.offsite.OffsiteMiddleware'</span>: <span class="hljs-number">500</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.spidermiddlewares.referer.RefererMiddleware'</span>: <span class="hljs-number">700</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware'</span>: <span class="hljs-number">800</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.spidermiddlewares.depth.DepthMiddleware'</span>: <span class="hljs-number">900</span>,
}
</code></pre>
<p data-nodeid="328241">和 Downloader Middleware 一样，Spider Middleware 首先加入 SPIDER_MIDDLEWARES 的设置中，该设置会和 Scrapy 中 SPIDER_MIDDLEWARES_BASE 定义的 Spider Middleware 合并。然后根据键值的数字优先级排序，得到一个有序列表。第一个 Middleware 是最靠近引擎的，最后一个 Middleware 是最靠近 Spider 的。</p>
<h4 data-nodeid="328242">核心方法</h4>
<p data-nodeid="328243">Scrapy 内置的 Spider Middleware 为 Scrapy 提供了基础的功能。如果我们想要扩展其功能，只需要实现某几个方法即可。</p>
<p data-nodeid="328244">每个 Spider Middleware 都定义了以下一个或多个方法的类，核心方法有如下 4 个。</p>
<ul data-nodeid="328245">
<li data-nodeid="328246">
<p data-nodeid="328247">process_spider_input(response, spider)</p>
</li>
<li data-nodeid="328248">
<p data-nodeid="328249">process_spider_output(response, result, spider)</p>
</li>
<li data-nodeid="328250">
<p data-nodeid="328251">process_spider_exception(response, exception, spider)</p>
</li>
<li data-nodeid="328252">
<p data-nodeid="328253">process_start_requests(start_requests, spider)</p>
</li>
</ul>
<p data-nodeid="328254">只需要实现其中一个方法就可以定义一个 Spider Middleware。下面我们来看看这 4 个方法的详细用法。</p>
<h5 data-nodeid="328255">process_spider_input(response, spider)</h5>
<p data-nodeid="328256">当 Response 通过 Spider Middleware 时，该方法被调用，处理该 Response。</p>
<p data-nodeid="328257">方法的参数有两个：</p>
<ul data-nodeid="328258">
<li data-nodeid="328259">
<p data-nodeid="328260">response，即 Response 对象，即被处理的 Response；</p>
</li>
<li data-nodeid="328261">
<p data-nodeid="328262">spider，即 Spider 对象，即该 response 对应的 Spider。</p>
</li>
</ul>
<p data-nodeid="328263">process_spider_input() 应该返回 None 或者抛出一个异常。</p>
<ul data-nodeid="328264">
<li data-nodeid="328265">
<p data-nodeid="328266">如果其返回 None，Scrapy 将会继续处理该 Response，调用所有其他的 Spider Middleware 直到 Spider 处理该 Response。</p>
</li>
<li data-nodeid="328267">
<p data-nodeid="328268">如果其抛出一个异常，Scrapy 将不会调用任何其他 Spider Middleware 的 process_spider_input() 方法，并调用 Request 的 errback() 方法。 errback 的输出将会以另一个方向被重新输入到中间件中，使用 process_spider_output() 方法来处理，当其抛出异常时则调用 process_spider_exception() 来处理。</p>
</li>
</ul>
<h5 data-nodeid="328269">process_spider_output(response, result, spider)</h5>
<p data-nodeid="328270">当 Spider 处理 Response 返回结果时，该方法被调用。</p>
<p data-nodeid="328271">方法的参数有三个：</p>
<ul data-nodeid="328272">
<li data-nodeid="328273">
<p data-nodeid="328274">response，即 Response 对象，即生成该输出的 Response；</p>
</li>
<li data-nodeid="328275">
<p data-nodeid="328276">result，包含 Request 或 Item 对象的可迭代对象，即 Spider 返回的结果；</p>
</li>
<li data-nodeid="328277">
<p data-nodeid="328278">spider，即 Spider 对象，即其结果对应的 Spider。</p>
</li>
</ul>
<p data-nodeid="328279">process_spider_output() 必须返回包含 Request 或 Item 对象的可迭代对象。</p>
<h5 data-nodeid="328280">process_spider_exception(response, exception, spider)</h5>
<p data-nodeid="328281">当 Spider 或 Spider Middleware 的 process_spider_input() 方法抛出异常时， 该方法被调用。</p>
<p data-nodeid="328282">方法的参数有三个：</p>
<ul data-nodeid="328283">
<li data-nodeid="328284">
<p data-nodeid="328285">response，即 Response 对象，即异常被抛出时被处理的 Response；</p>
</li>
<li data-nodeid="328286">
<p data-nodeid="328287">exception，即 Exception 对象，被抛出的异常；</p>
</li>
<li data-nodeid="328288">
<p data-nodeid="328289">spider，即 Spider 对象，即抛出该异常的 Spider。</p>
</li>
</ul>
<p data-nodeid="328290">process_spider_exception() 必须返回结果，要么返回 None ， 要么返回一个包含 Response 或 Item 对象的可迭代对象。</p>
<ul data-nodeid="328291">
<li data-nodeid="328292">
<p data-nodeid="328293">如果其返回 None ，Scrapy 将继续处理该异常，调用其他 Spider Middleware 中的 process_spider_exception() 方法，直到所有 Spider Middleware 都被调用。</p>
</li>
<li data-nodeid="328294">
<p data-nodeid="328295">如果其返回一个可迭代对象，则其他 Spider Middleware 的 process_spider_output() 方法被调用， 其他的 process_spider_exception() 将不会被调用。</p>
</li>
</ul>
<h5 data-nodeid="328296">process_start_requests(start_requests, spider)</h5>
<p data-nodeid="328297">该方法以 Spider 启动的 Request 为参数被调用，执行的过程类似于 process_spider_output() ，只不过其没有相关联的 Response 并且必须返回 Request。</p>
<p data-nodeid="328298">方法的参数有两个：</p>
<ul data-nodeid="328299">
<li data-nodeid="328300">
<p data-nodeid="328301">start_requests，即包含 Request 的可迭代对象，即 Start Requests；</p>
</li>
<li data-nodeid="328302">
<p data-nodeid="328303">spider，即 Spider 对象，即 Start Requests 所属的 Spider。</p>
</li>
</ul>
<p data-nodeid="328304">其必须返回另一个包含 Request 对象的可迭代对象。</p>
<h3 data-nodeid="328305">Downloader Middleware 的用法</h3>
<p data-nodeid="328306">Downloader Middleware 即下载中间件，它是处于 Scrapy 的 Request 和 Response 之间的处理模块。</p>
<p data-nodeid="328307">Scheduler 从队列中拿出一个 Request 发送给 Downloader 执行下载，这个过程会经过 Downloader Middleware 的处理。另外，当 Downloader 将 Request 下载完成得到 Response 返回给 Spider 时会再次经过 Downloader Middleware 处理。</p>
<p data-nodeid="328308">也就是说，Downloader Middleware 在整个架构中起作用的位置是以下两个。</p>
<ul data-nodeid="328309">
<li data-nodeid="328310">
<p data-nodeid="328311">在 Scheduler 调度出队列的 Request 发送给 Downloader 下载之前，也就是我们可以在 Request 执行下载之前对其进行修改。</p>
</li>
<li data-nodeid="328312">
<p data-nodeid="328313">在下载后生成的 Response 发送给 Spider 之前，也就是我们可以在生成 Resposne 被 Spider 解析之前对其进行修改。</p>
</li>
</ul>
<p data-nodeid="328314">Downloader Middleware 的功能十分强大，修改 User-Agent、处理重定向、设置代理、失败重试、设置 Cookies 等功能都需要借助它来实现。下面我们来了解一下 Downloader Middleware 的详细用法。</p>
<h4 data-nodeid="328315">使用说明</h4>
<p data-nodeid="328316">需要说明的是，Scrapy 其实已经提供了许多 Downloader Middleware，比如负责失败重试、自动重定向等功能的 Middleware，它们被 DOWNLOADER_MIDDLEWARES_BASE 变量所定义。</p>
<p data-nodeid="328317">DOWNLOADER_MIDDLEWARES_BASE 变量的内容如下所示：</p>
<pre class="lang-java" data-nodeid="328318"><code data-language="java">{
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware'</span>: <span class="hljs-number">100</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware'</span>: <span class="hljs-number">300</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware'</span>: <span class="hljs-number">350</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware'</span>: <span class="hljs-number">400</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware'</span>: <span class="hljs-number">500</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.retry.RetryMiddleware'</span>: <span class="hljs-number">550</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware'</span>: <span class="hljs-number">560</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware'</span>: <span class="hljs-number">580</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware'</span>: <span class="hljs-number">590</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.redirect.RedirectMiddleware'</span>: <span class="hljs-number">600</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.cookies.CookiesMiddleware'</span>: <span class="hljs-number">700</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware'</span>: <span class="hljs-number">750</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.stats.DownloaderStats'</span>: <span class="hljs-number">850</span>,
 &nbsp; &nbsp;<span class="hljs-string">'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware'</span>: <span class="hljs-number">900</span>,
}
</code></pre>
<p data-nodeid="328800">这是一个字典格式，字典的键名是 Scrapy 内置的 Downloader Middleware 的名称，键值代表了调用的优先级，优先级是一个数字，数字越小代表越靠近 Scrapy 引擎，数字越大代表越靠近 Downloader。每个 Downloader Middleware 都可以定义 process_request() 和 request_response() 方法来分别处理请求和响应，对于 process_request() 方法来说，优先级数字越小越先被调用，对于 process_response() 方法来说，优先级数字越大越先被调用。</p>
<p data-nodeid="328801">如果自己定义的 Downloader Middleware 要添加到项目里，DOWNLOADER_MIDDLEWARES_BASE 变量不能直接修改。Scrapy 提供了另外一个设置变量 DOWNLOADER_MIDDLEWARES，我们直接修改这个变量就可以添加自己定义的 Downloader Middleware，以及禁用 DOWNLOADER_MIDDLEWARES_BASE 里面定义的 Downloader Middleware。下面我们具体来看看 Downloader Middleware 的使用方法。</p>

<h4 data-nodeid="328320">核心方法</h4>
<p data-nodeid="328321">Scrapy 内置的 Downloader Middleware 为 Scrapy 提供了基础的功能，但在项目实战中我们往往需要单独定义 Downloader Middleware。不用担心，这个过程非常简单，我们只需要实现某几个方法即可。</p>
<p data-nodeid="328322">每个 Downloader Middleware 都定义了一个或多个方法的类，核心的方法有如下三个。</p>
<ul data-nodeid="328323">
<li data-nodeid="328324">
<p data-nodeid="328325">process_request(request, spider)</p>
</li>
<li data-nodeid="328326">
<p data-nodeid="328327">process_response(request, response, spider)</p>
</li>
<li data-nodeid="328328">
<p data-nodeid="328329">process_exception(request, exception, spider)</p>
</li>
</ul>
<p data-nodeid="328330">我们只需要实现至少一个方法，就可以定义一个 Downloader Middleware。下面我们来看看这三个方法的详细用法。</p>
<h5 data-nodeid="328331">process_request(request, spider)</h5>
<p data-nodeid="328332">Request 被 Scrapy 引擎调度给 Downloader 之前，process_request() 方法就会被调用，也就是在 Request 从队列里调度出来到 Downloader 下载执行之前，我们都可以用 process_request() 方法对 Request 进行处理。方法的返回值必须为 None、Response 对象、Request 对象之一，或者抛出 IgnoreRequest 异常。</p>
<p data-nodeid="328333">process_request() 方法的参数有如下两个。</p>
<ul data-nodeid="328334">
<li data-nodeid="328335">
<p data-nodeid="328336">request，即 Request 对象，即被处理的 Request；</p>
</li>
<li data-nodeid="328337">
<p data-nodeid="328338">spider，即 Spider 对象，即此 Request 对应的 Spider。</p>
</li>
</ul>
<p data-nodeid="328339">返回类型不同，产生的效果也不同。下面归纳一下不同的返回情况。</p>
<ul data-nodeid="328340">
<li data-nodeid="328341">
<p data-nodeid="328342">当返回为 None 时，Scrapy 将继续处理该 Request，接着执行其他 Downloader Middleware 的 process_request() 方法，直到 Downloader 把 Request 执行后得到 Response 才结束。这个过程其实就是修改 Request 的过程，不同的 Downloader Middleware 按照设置的优先级顺序依次对 Request 进行修改，最后推送至 Downloader 执行。</p>
</li>
<li data-nodeid="328343">
<p data-nodeid="328344">当返回为 Response 对象时，更低优先级的 Downloader Middleware 的 process_request() 和 process_exception() 方法就不会被继续调用，每个 Downloader Middleware 的 process_response() 方法转而被依次调用。调用完毕之后，直接将 Response 对象发送给 Spider 来处理。</p>
</li>
<li data-nodeid="328345">
<p data-nodeid="328346">当返回为 Request 对象时，更低优先级的 Downloader Middleware 的 process_request() 方法会停止执行。这个 Request 会重新放到调度队列里，其实它就是一个全新的 Request，等待被调度。如果被 Scheduler 调度了，那么所有的 Downloader Middleware 的 process_request() 方法会被重新按照顺序执行。</p>
</li>
<li data-nodeid="328347">
<p data-nodeid="328348">如果 IgnoreRequest 异常抛出，则所有的 Downloader Middleware 的 process_exception() 方法会依次执行。如果没有一个方法处理这个异常，那么 Request 的 errorback() 方法就会回调。如果该异常还没有被处理，那么它便会被忽略。</p>
</li>
</ul>
<h5 data-nodeid="328349">process_response(request, response, spider)</h5>
<p data-nodeid="328350">Downloader 执行 Request 下载之后，会得到对应的 Response。Scrapy 引擎便会将 Response 发送给 Spider 进行解析。在发送之前，我们都可以用 process_response() 方法来对 Response 进行处理。方法的返回值必须为 Request 对象、Response 对象之一，或者抛出 IgnoreRequest 异常。</p>
<p data-nodeid="328351">process_response() 方法的参数有如下三个。</p>
<ul data-nodeid="328352">
<li data-nodeid="328353">
<p data-nodeid="328354">request，是 Request 对象，即此 Response 对应的 Request。</p>
</li>
<li data-nodeid="328355">
<p data-nodeid="328356">response，是 Response 对象，即此被处理的 Response。</p>
</li>
<li data-nodeid="328357">
<p data-nodeid="328358">spider，是 Spider 对象，即此 Response 对应的 Spider。</p>
</li>
</ul>
<p data-nodeid="328359">下面对不同的返回情况做一下归纳：</p>
<ul data-nodeid="328360">
<li data-nodeid="328361">
<p data-nodeid="328362">当返回为 Request 对象时，更低优先级的 Downloader Middleware 的 process_response() 方法不会继续调用。该 Request 对象会重新放到调度队列里等待被调度，它相当于一个全新的 Request。然后，该 Request 会被 process_request() 方法顺次处理。</p>
</li>
<li data-nodeid="328363">
<p data-nodeid="328364">当返回为 Response 对象时，更低优先级的 Downloader Middleware 的 process_response() 方法会继续调用，继续对该 Response 对象进行处理。</p>
</li>
<li data-nodeid="328365">
<p data-nodeid="328366">如果 IgnoreRequest 异常抛出，则 Request 的 errorback() 方法会回调。如果该异常还没有被处理，那么它便会被忽略。</p>
</li>
</ul>
<h5 data-nodeid="328367">process_exception(request, exception, spider)</h5>
<p data-nodeid="328368">当 Downloader 或 process_request() 方法抛出异常时，例如抛出 IgnoreRequest 异常，process_exception() 方法就会被调用。方法的返回值必须为 None、Response 对象、Request 对象之一。</p>
<p data-nodeid="328369">process_exception() 方法的参数有如下三个。</p>
<ul data-nodeid="328370">
<li data-nodeid="328371">
<p data-nodeid="328372">request，即 Request 对象，即产生异常的 Request。</p>
</li>
<li data-nodeid="328373">
<p data-nodeid="328374">exception，即 Exception 对象，即抛出的异常。</p>
</li>
<li data-nodeid="328375">
<p data-nodeid="328376">spdier，即 Spider 对象，即 Request 对应的 Spider。</p>
</li>
</ul>
<p data-nodeid="328377">下面归纳一下不同的返回值。</p>
<ul data-nodeid="328378">
<li data-nodeid="328379">
<p data-nodeid="328380">当返回为 None 时，更低优先级的 Downloader Middleware 的 process_exception() 会被继续顺次调用，直到所有的方法都被调度完毕。</p>
</li>
<li data-nodeid="328381">
<p data-nodeid="328382">当返回为 Response 对象时，更低优先级的 Downloader Middleware 的 process_exception() 方法不再被继续调用，每个 Downloader Middleware 的 process_response() 方法转而被依次调用。</p>
</li>
<li data-nodeid="328383">
<p data-nodeid="328384">当返回为 Request 对象时，更低优先级的 Downloader Middleware 的 process_exception() 也不再被继续调用，该 Request 对象会重新放到调度队列里面等待被调度，它相当于一个全新的 Request。然后，该 Request 又会被 process_request() 方法顺次处理。</p>
</li>
</ul>
<p data-nodeid="328385">以上内容便是这三个方法的详细使用逻辑。在使用它们之前，请先对这三个方法的返回值的处理情况有一个清晰的认识。在自定义 Downloader Middleware 的时候，也一定要注意每个方法的返回类型。</p>
<p data-nodeid="328386">下面我们用一个案例实战来加深一下对 Downloader Middleware 用法的理解。</p>
<h4 data-nodeid="328387">项目实战</h4>
<p data-nodeid="328388">新建一个项目，命令如下所示：</p>
<pre class="lang-java" data-nodeid="328389"><code data-language="java">scrapy startproject scrapydownloadertest
</code></pre>
<p data-nodeid="328390">新建了一个 Scrapy 项目，名为 scrapydownloadertest。进入项目，新建一个 Spider，命令如下所示：</p>
<pre class="lang-java" data-nodeid="328391"><code data-language="java">scrapy genspider httpbin httpbin.org
</code></pre>
<p data-nodeid="328392">新建了一个 Spider，名为 httpbin，源代码如下所示：</p>
<pre class="lang-java" data-nodeid="328393"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">import</span> scrapy
class <span class="hljs-title">HttpbinSpider</span><span class="hljs-params">(scrapy.Spider)</span>:
 &nbsp; &nbsp;name </span>= <span class="hljs-string">'httpbin'</span>
 &nbsp; &nbsp;allowed_domains = [<span class="hljs-string">'httpbin.org'</span>]
 &nbsp; &nbsp;start_urls = [<span class="hljs-string">'http://httpbin.org/'</span>]

 &nbsp; &nbsp;<span class="hljs-function">def <span class="hljs-title">parse</span><span class="hljs-params">(self, response)</span>:
 &nbsp; &nbsp; &nbsp; &nbsp;pass
</span></code></pre>
<p data-nodeid="328394">接下来我们修改 start_urls 为：<code data-backticks="1" data-nodeid="328716">['http://httpbin.org/']</code>。随后将 parse() 方法添加一行日志输出，将 response 变量的 text 属性输出，这样我们便可以看到 Scrapy 发送的 Request 信息了。<br>
修改 Spider 内容如下所示：</p>
<pre class="lang-dart" data-nodeid="328395"><code data-language="dart"><span class="hljs-keyword">import</span> scrapy

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HttpbinSpider</span>(<span class="hljs-title">scrapy</span>.<span class="hljs-title">Spider</span>):
 &nbsp; &nbsp;<span class="hljs-title">name</span> = '<span class="hljs-title">httpbin</span>'
 &nbsp; &nbsp;<span class="hljs-title">allowed_domains</span> = ['<span class="hljs-title">httpbin</span>.<span class="hljs-title">org</span>']
 &nbsp; &nbsp;<span class="hljs-title">start_urls</span> = ['<span class="hljs-title">http</span>://<span class="hljs-title">httpbin</span>.<span class="hljs-title">org</span>/<span class="hljs-title">get</span>']

 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">parse</span>(<span class="hljs-title">self</span>, <span class="hljs-title">response</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">logger</span>.<span class="hljs-title">debug</span>(<span class="hljs-title">response</span>.<span class="hljs-title">text</span>)
</span></code></pre>
<p data-nodeid="328396">接下来运行此 Spider，执行如下命令：</p>
<pre class="lang-java" data-nodeid="328397"><code data-language="java">scrapy crawl httpbin
</code></pre>
<p data-nodeid="328398">Scrapy 运行结果包含 Scrapy 发送的 Request 信息，内容如下所示：</p>
<pre class="lang-java" data-nodeid="328399"><code data-language="java">{<span class="hljs-string">"args"</span>: {}, 
 &nbsp;<span class="hljs-string">"headers"</span>: {
 &nbsp; &nbsp;<span class="hljs-string">"Accept"</span>: <span class="hljs-string">"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Accept-Encoding"</span>: <span class="hljs-string">"gzip,deflate,br"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Accept-Language"</span>: <span class="hljs-string">"en"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Connection"</span>: <span class="hljs-string">"close"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Host"</span>: <span class="hljs-string">"httpbin.org"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"User-Agent"</span>: <span class="hljs-string">"Scrapy/1.4.0 (+http://scrapy.org)"</span>
  }, 
 &nbsp;<span class="hljs-string">"origin"</span>: <span class="hljs-string">"60.207.237.85"</span>, 
 &nbsp;<span class="hljs-string">"url"</span>: <span class="hljs-string">"http://httpbin.org/get"</span>
}
</code></pre>
<p data-nodeid="328400">我们观察一下 Headers，Scrapy 发送的 Request 使用的 User-Agent 是 Scrapy/1.4.0(+<a href="http://scrapy.org" data-nodeid="328725">http://scrapy.org</a>)，这其实是由 Scrapy 内置的 UserAgentMiddleware 设置的，UserAgentMiddleware 的源码如下所示：</p>
<pre class="lang-dart" data-nodeid="328401"><code data-language="dart">from scrapy <span class="hljs-keyword">import</span> signals

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserAgentMiddleware</span>(<span class="hljs-title">object</span>):
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-title">self</span>, <span class="hljs-title">user_agent</span>='<span class="hljs-title">Scrapy</span>'):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">user_agent</span> = <span class="hljs-title">user_agent</span>

 &nbsp; &nbsp;@<span class="hljs-title">classmethod</span>
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">from_crawler</span>(<span class="hljs-title">cls</span>, <span class="hljs-title">crawler</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">o</span> = <span class="hljs-title">cls</span>(<span class="hljs-title">crawler</span>.<span class="hljs-title">settings</span>['<span class="hljs-title">USER_AGENT</span>'])
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">crawler</span>.<span class="hljs-title">signals</span>.<span class="hljs-title">connect</span>(<span class="hljs-title">o</span>.<span class="hljs-title">spider_opened</span>, <span class="hljs-title">signal</span>=<span class="hljs-title">signals</span>.<span class="hljs-title">spider_opened</span>)
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">o</span>

 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">spider_opened</span>(<span class="hljs-title">self</span>, <span class="hljs-title">spider</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">user_agent</span> = <span class="hljs-title">getattr</span>(<span class="hljs-title">spider</span>, '<span class="hljs-title">user_agent</span>', <span class="hljs-title">self</span>.<span class="hljs-title">user_agent</span>)

 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">process_request</span>(<span class="hljs-title">self</span>, <span class="hljs-title">request</span>, <span class="hljs-title">spider</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">self</span>.<span class="hljs-title">user_agent</span>:
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">request</span>.<span class="hljs-title">headers</span>.<span class="hljs-title">setdefault</span>(<span class="hljs-title">b</span>'<span class="hljs-title">User</span>-<span class="hljs-title">Agent</span>', <span class="hljs-title">self</span>.<span class="hljs-title">user_agent</span>)
</span></code></pre>
<p data-nodeid="328402">在 from_crawler() 方法中，首先尝试获取 settings 里面的 USER_AGENT，然后把 USER_AGENT 传递给 <strong data-nodeid="328756">init</strong>() 方法进行初始化，其参数就是 user_agent。如果没有传递 USER_AGENT 参数就默认设置为 Scrapy 字符串。我们新建的项目没有设置 USER_AGENT，所以这里的 user_agent 变量就是 Scrapy。接下来，在 process_request() 方法中，将 user-agent 变量设置为 headers 变量的一个属性，这样就成功设置了 User-Agent。因此，User-Agent 就是通过此 Downloader Middleware 的 process_request() 方法设置的。<br>
修改请求时的 User-Agent 可以有两种方式：一是修改 settings 里面的 USER_AGENT 变量；二是通过 Downloader Middleware 的 process_request() 方法来修改。</p>
<p data-nodeid="328403">第一种方法非常简单，我们只需要在 setting.py 里面加一行 USER_AGENT 的定义即可：</p>
<pre class="lang-java" data-nodeid="328404"><code data-language="java">USER_AGENT = <span class="hljs-string">'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36'</span>
</code></pre>
<p data-nodeid="328405">一般推荐使用此方法来设置。但是如果想设置得更灵活，比如设置随机的 User-Agent，那就需要借助 Downloader Middleware 了。所以接下来我们用 Downloader Middleware 实现一个随机 User-Agent 的设置。</p>
<p data-nodeid="328406">在 middlewares.py 里面添加一个 RandomUserAgentMiddleware 的类，如下所示：</p>
<pre class="lang-dart" data-nodeid="328407"><code data-language="dart"><span class="hljs-keyword">import</span> random

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RandomUserAgentMiddleware</span>():
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-title">self</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">user_agents</span> = ['<span class="hljs-title">Mozilla</span>/5.0 (<span class="hljs-title">Windows</span>; <span class="hljs-title">U</span>; <span class="hljs-title">MSIE</span> 9.0; <span class="hljs-title">Windows</span> <span class="hljs-title">NT</span> 9.0; <span class="hljs-title">en</span>-<span class="hljs-title">US</span>)',
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'<span class="hljs-title">Mozilla</span>/5.0 (<span class="hljs-title">Windows</span> <span class="hljs-title">NT</span> 6.1) <span class="hljs-title">AppleWebKit</span>/537.2 (<span class="hljs-title">KHTML</span>, <span class="hljs-title">like</span> <span class="hljs-title">Gecko</span>) <span class="hljs-title">Chrome</span>/22.0.1216.0 <span class="hljs-title">Safari</span>/537.2',
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;'<span class="hljs-title">Mozilla</span>/5.0 (<span class="hljs-title">X11</span>; <span class="hljs-title">Ubuntu</span>; <span class="hljs-title">Linux</span> <span class="hljs-title">i686</span>; <span class="hljs-title">rv</span>:15.0) <span class="hljs-title">Gecko</span>/20100101 <span class="hljs-title">Firefox</span>/15.0.1'
 &nbsp; &nbsp; &nbsp;  ]

 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">process_request</span>(<span class="hljs-title">self</span>, <span class="hljs-title">request</span>, <span class="hljs-title">spider</span>):
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">request</span>.<span class="hljs-title">headers</span>['<span class="hljs-title">User</span>-<span class="hljs-title">Agent</span>'] = <span class="hljs-title">random</span>.<span class="hljs-title">choice</span>(<span class="hljs-title">self</span>.<span class="hljs-title">user_agents</span>)
</span></code></pre>
<p data-nodeid="328408">我们首先在类的 __init__() 方法中定义了三个不同的 User-Agent，并用一个列表来表示。接下来实现了 process_request() 方法，它有一个参数 request，我们直接修改 request 的属性即可。在这里我们直接设置了 request 对象的 headers 属性的 User-Agent，设置内容是随机选择的 User-Agent，这样一个 Downloader Middleware 就写好了。</p>
<p data-nodeid="328409">不过，要使之生效我们还需要再去调用这个 Downloader Middleware。在 settings.py 中，将 DOWNLOADER_MIDDLEWARES 取消注释，并设置成如下内容：</p>
<pre class="lang-java" data-nodeid="328410"><code data-language="java">DOWNLOADER_MIDDLEWARES = {<span class="hljs-string">'scrapydownloadertest.middlewares.RandomUserAgentMiddleware'</span>: <span class="hljs-number">543</span>,}
</code></pre>
<p data-nodeid="328411">接下来我们重新运行 Spider，就可以看到 User-Agent 被成功修改为列表中所定义的随机的一个 User-Agent 了：</p>
<pre class="lang-java" data-nodeid="328412"><code data-language="java">{<span class="hljs-string">"args"</span>: {}, 
 &nbsp;<span class="hljs-string">"headers"</span>: {
 &nbsp; &nbsp;<span class="hljs-string">"Accept"</span>: <span class="hljs-string">"text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Accept-Encoding"</span>: <span class="hljs-string">"gzip,deflate,br"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Accept-Language"</span>: <span class="hljs-string">"en"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Connection"</span>: <span class="hljs-string">"close"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"Host"</span>: <span class="hljs-string">"httpbin.org"</span>, 
 &nbsp; &nbsp;<span class="hljs-string">"User-Agent"</span>: <span class="hljs-string">"Mozilla/5.0 (Windows; U; MSIE 9.0; Windows NT 9.0; en-US)"</span>
  }, 
 &nbsp;<span class="hljs-string">"origin"</span>: <span class="hljs-string">"60.207.237.85"</span>, 
 &nbsp;<span class="hljs-string">"url"</span>: <span class="hljs-string">"http://httpbin.org/get"</span>
}
</code></pre>
<p data-nodeid="328413">我们就通过实现 Downloader Middleware 并利用 process_request() 方法成功设置了随机的 User-Agent。</p>
<p data-nodeid="328414">另外，Downloader Middleware 还有 process_response() 方法。Downloader 对 Request 执行下载之后会得到 Response，随后 Scrapy 引擎会将 Response 发送回 Spider 进行处理。但是在 Response 被发送给 Spider 之前，我们同样可以使用 process_response() 方法对 Response 进行处理。比如这里修改一下 Response 的状态码，在 RandomUserAgentMiddleware 添加如下代码：</p>
<pre class="lang-java" data-nodeid="328415"><code data-language="java"><span class="hljs-function">def <span class="hljs-title">process_response</span><span class="hljs-params">(self, request, response, spider)</span>:
 &nbsp; &nbsp;response.status </span>= <span class="hljs-number">201</span>
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> response
</code></pre>
<p data-nodeid="328416">我们将 response 对象的 status 属性修改为 201，随后将 response 返回，这个被修改后的 Response 就会被发送到 Spider。</p>
<p data-nodeid="328417">我们再在 Spider 里面输出修改后的状态码，在 parse() 方法中添加如下的输出语句：</p>
<pre class="lang-java" data-nodeid="328418"><code data-language="java">self.logger.debug(<span class="hljs-string">'Status Code: '</span> + str(response.status))
</code></pre>
<p data-nodeid="328419">重新运行之后，控制台输出了如下内容：</p>
<pre class="lang-java" data-nodeid="328420"><code data-language="java">[httpbin] DEBUG: Status Code: <span class="hljs-number">201</span>
</code></pre>
<p data-nodeid="328421">可以发现，Response 的状态码成功修改了。因此要想对 Response 进行处理，就可以借助于 process_response() 方法。</p>
<p data-nodeid="328422">另外还有一个 process_exception() 方法，它是用来处理异常的方法。如果需要异常处理的话，我们可以调用此方法。不过这个方法的使用频率相对低一些，在此不用实例演示。</p>
<h3 data-nodeid="328423">本节代码</h3>
<p data-nodeid="329972" class="te-preview-highlight">本节源代码为：<br>
<a href="https://github.com/Python3WebSpider/ScrapyDownloaderTest" data-nodeid="329977">https://github.com/Python3WebSpider/ScrapyDownloaderTest</a>。</p>

<h3 data-nodeid="328425">结语</h3>
<p data-nodeid="328426" class="">本节讲解了 Spider Middleware 和 Downloader Middleware 的基本用法。利用它们我们可以方便地实现爬虫逻辑的灵活处理，需要好好掌握。</p>

---

### 精选评论


