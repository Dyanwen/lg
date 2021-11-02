<p data-nodeid="10255">在上一课时我们了解了 Pyppeteer 的基本用法，确实我们可以发现其相比 Selenium 有很多方便之处。</p>
<p data-nodeid="10256">本课时我们就来使用 Pyppeteer 针对之前的 Selenium 案例做一次改写，来体会一下二者的不同之处，同时也加强一下对 Pyppeteer 的理解和掌握情况。</p>
<h3 data-nodeid="10257">爬取目标</h3>
<p data-nodeid="10258">本课时我们要爬取的目标和之前是一样的，还是 Selenium 的那个案例，地址为：<a href="https://dynamic2.scrape.center/" data-nodeid="10354">https://dynamic2.scrape.center/</a>，如下图所示。<br>
<img src="https://s0.lgstatic.com/i/image3/M01/85/2E/Cgq2xl6NsnyAfz5HAAJisxSlYKw076.png" alt="" data-nodeid="10358"><br>
这个网站的每个详情页的 URL 都是带有加密参数的，同时 Ajax 接口也都有加密参数和时效性。具体的介绍可以看下 Selenium 课时。</p>
<h3 data-nodeid="10259">本节目标</h3>
<p data-nodeid="10260">爬取目标和那一节也是一样的：</p>
<ul data-nodeid="10261">
<li data-nodeid="10262">
<p data-nodeid="10263">遍历每一页列表页，然后获取每部电影详情页的 URL。</p>
</li>
<li data-nodeid="10264">
<p data-nodeid="10265">爬取每部电影的详情页，然后提取其名称、评分、类别、封面、简介等信息。</p>
</li>
<li data-nodeid="10266">
<p data-nodeid="10267">爬取到的数据存为 JSON 文件。</p>
</li>
</ul>
<p data-nodeid="10268">要求和之前也是一样的，只不过我们这里的实现就全用 Pyppeteer 来做了。</p>
<h3 data-nodeid="10269">准备工作</h3>
<p data-nodeid="10270">在本课时开始之前，我们需要做好如下准备工作：</p>
<ul data-nodeid="10271">
<li data-nodeid="10272">
<p data-nodeid="10273">安装好 Python （最低为 Python 3.6）版本，并能成功运行 Python 程序。</p>
</li>
<li data-nodeid="10274">
<p data-nodeid="10275">安装好 Pyppeteer 并能成功运行示例。</p>
</li>
</ul>
<p data-nodeid="10276">其他的浏览器、驱动配置就不需要了，这也是相比 Selenium 更加方便的地方。</p>
<p data-nodeid="10277">页面分析在这里就不多介绍了，还是列表页 + 详情页的结构，具体可以参考 Selenium 那一课时的内容。</p>
<h3 data-nodeid="10278">爬取列表页</h3>
<p data-nodeid="10279">首先我们先做一些准备工作，定义一些基础的配置，包括日志定义、变量等等并引入一些必要的包，代码如下：</p>
<pre class="lang-python" data-nodeid="10280"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;logging
logging.basicConfig(level=logging.INFO,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format=<span class="hljs-string">'%(asctime)s&nbsp;-&nbsp;%(levelname)s:&nbsp;%(message)s'</span>)
INDEX_URL&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/page/{page}'</span>
TIMEOUT&nbsp;=&nbsp;<span class="hljs-number">10</span>
TOTAL_PAGE&nbsp;=&nbsp;<span class="hljs-number">10</span>
WINDOW_WIDTH,&nbsp;WINDOW_HEIGHT&nbsp;=&nbsp;<span class="hljs-number">1366</span>,&nbsp;<span class="hljs-number">768</span>
HEADLESS&nbsp;=&nbsp;<span class="hljs-literal">False</span>
</code></pre>
<p data-nodeid="10281">这里大多数的配置和之前是一样的，不过这里我们额外定义了窗口的宽高信息，这里定义为 1366 x 768，你也可以随意指定适合自己屏幕的宽高信息。另外这里定义了一个变量 HEADLESS，用来指定是否启用 Pyppeteer 的无头模式，如果为 False，那么启动 Pyppeteer 的时候就会弹出一个 Chromium 浏览器窗口。</p>
<p data-nodeid="10282">接着我们再定义一个初始化 Pyppeteer 的方法，包括启动 Pyppeteer，新建一个页面选项卡，设置窗口大小等操作，代码实现如下：</p>
<pre class="lang-python" data-nodeid="10283"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;pyppeteer&nbsp;<span class="hljs-keyword">import</span>&nbsp;launch
browser,&nbsp;tab&nbsp;=&nbsp;<span class="hljs-literal">None</span>,&nbsp;<span class="hljs-literal">None</span>
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">init</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">global</span>&nbsp;browser,&nbsp;tab
&nbsp;&nbsp;&nbsp;browser&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;launch(headless=HEADLESS,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;args=[<span class="hljs-string">'--disable-infobars'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">f'--window-size=<span class="hljs-subst">{WINDOW_WIDTH}</span>,<span class="hljs-subst">{WINDOW_HEIGHT}</span>'</span>])
&nbsp;&nbsp;&nbsp;tab&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;browser.newPage()
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.setViewport({<span class="hljs-string">'width'</span>:&nbsp;WINDOW_WIDTH,&nbsp;<span class="hljs-string">'height'</span>:&nbsp;WINDOW_HEIGHT})
</code></pre>
<p data-nodeid="10284">在这里我们先声明了一个 browser 对象，代表 Pyppeteer 所用的浏览器对象，tab 代表新建的页面选项卡，这里把两项设置为全局变量，方便其他的方法调用。</p>
<p data-nodeid="10285">另外定义了一个 init 方法，调用了 Pyppeteer 的 launch 方法，传入了 headless 为 HEADLESS，将其设置为非无头模式，另外还通过 args 指定了隐藏提示条并设定了窗口的宽高。</p>
<p data-nodeid="10286">接下来我们像之前一样，定义一个通用的爬取方法，代码如下：</p>
<pre class="lang-python" data-nodeid="10287"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;pyppeteer.errors&nbsp;<span class="hljs-keyword">import</span>&nbsp;TimeoutError
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_page</span>(<span class="hljs-params">url,&nbsp;selector</span>):</span>
&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'scraping&nbsp;%s'</span>,&nbsp;url)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.goto(url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.waitForSelector(selector,&nbsp;options={
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'timeout'</span>:&nbsp;TIMEOUT&nbsp;*&nbsp;<span class="hljs-number">1000</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">except</span>&nbsp;TimeoutError:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.error(<span class="hljs-string">'error&nbsp;occurred&nbsp;while&nbsp;scraping&nbsp;%s'</span>,&nbsp;url,&nbsp;exc_info=<span class="hljs-literal">True</span>)
</code></pre>
<p data-nodeid="10288">这里我们定义了一个 scrape_page 方法，它接收两个参数，一个是 url，代表要爬取的链接，使用 goto 方法调用即可；另外一个是 selector，即要等待渲染出的节点对应的 CSS 选择器，这里我们使用 waitForSelector 方法并传入了 selector，并通过 options 指定了最长等待时间。</p>
<p data-nodeid="10289">这样的话在运行时页面会首先访问这个 URL，然后等待某个符合 selector 的节点加载出来，最长等待 10 秒，如果 10 秒内加载出来了，那就接着往下执行，否则抛出异常，捕获 TimeoutError 并输出错误日志。</p>
<p data-nodeid="10290">接下来，我们就实现一下爬取列表页的方法，代码实现如下：</p>
<pre class="lang-python" data-nodeid="10291"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_index</span>(<span class="hljs-params">page</span>):</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;INDEX_URL.format(page=page)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_page(url,&nbsp;<span class="hljs-string">'.item&nbsp;.name'</span>)
</code></pre>
<p data-nodeid="10292">这里我们定义了 scrape_index 方法来爬取页面，其接受一个参数 page，代表要爬取的页码，这里我们首先通过 INDEX_URL 构造了列表页的 URL，然后调用 scrape_page 方法传入了 url 和要等待加载的选择器。</p>
<p data-nodeid="10293">这里的选择器我们使用的是 .item .name，这就是列表页中每部电影的名称，如果这个加载出来了，那么就代表页面加载成功了，如图所示。<br>
<img src="https://s0.lgstatic.com/i/image3/M01/0C/18/Ciqah16NsnyAKbHhAAPy_Jk6bGI885.png" alt="" data-nodeid="10395"></p>
<p data-nodeid="10294">好，接下来我们可以再定义一个解析列表页的方法，提取出每部电影的详情页 URL，定义如下：</p>
<pre class="lang-python" data-nodeid="10295"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">parse_index</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorAllEval(<span class="hljs-string">'.item&nbsp;.name'</span>,&nbsp;<span class="hljs-string">'nodes&nbsp;=&gt;&nbsp;nodes.map(node&nbsp;=&gt;&nbsp;node.href)'</span>)
</code></pre>
<p data-nodeid="10296">这里我们调用了 querySelectorAllEval 方法，它接收两个参数，第一个参数是 selector，代表要选择的节点对应的 CSS 选择器；第二个参数是 pageFunction，代表的是要执行的 JavaScript 方法，这里需要传入的是一段 JavaScript 字符串，整个方法的作用是选择 selector 对应的节点，然后对这些节点通过 pageFunction 定义的逻辑抽取出对应的结果并返回。</p>
<p data-nodeid="10297">所以这里第一个参数 selector 就传入电影名称对应的节点，其实是超链接 a 节点。由于提取结果有多个，所以这里 JavaScript 对应的 pageFunction 输入参数就是 nodes，输出结果是调用了 map 方法得到每个 node，然后调用 node 的 href 属性即可。这样返回结果就是当前列表页的所有电影的详情页 URL 组成的列表了。</p>
<p data-nodeid="10298">好，接下来我们来串联调用一下看看，代码实现如下：</p>
<pre class="lang-python" data-nodeid="10299"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;asyncio
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;init()
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;page&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">1</span>,&nbsp;TOTAL_PAGE&nbsp;+&nbsp;<span class="hljs-number">1</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_index(page)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_urls&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;parse_index()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'detail_urls&nbsp;%s'</span>,&nbsp;detail_urls)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">finally</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;browser.close()
<span class="hljs-keyword">if</span>&nbsp;__name__&nbsp;==&nbsp;<span class="hljs-string">'__main__'</span>:
&nbsp;&nbsp;&nbsp;asyncio.get_event_loop().run_until_complete(main())
</code></pre>
<p data-nodeid="10300">这里我们定义了一个 mian 方法，将前面定义的几个方法串联调用了一下。首先调用了 init 方法，然后循环遍历页码，调用了 scrape_index 方法爬取了每一页列表页，接着我们调用了 parse_index 方法，从列表页中提取出详情页的每个 URL，然后输出结果。</p>
<p data-nodeid="10301">运行结果如下：</p>
<pre class="lang-python" data-nodeid="10302"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">13</span>:<span class="hljs-number">54</span>:<span class="hljs-number">28</span>,<span class="hljs-number">879</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">13</span>:<span class="hljs-number">54</span>:<span class="hljs-number">31</span>,<span class="hljs-number">411</span>&nbsp;-&nbsp;INFO:&nbsp;detail_urls&nbsp;[<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx'</span>,&nbsp;...,
<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWI5'</span>,&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIxMA=='</span>]
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">13</span>:<span class="hljs-number">54</span>:<span class="hljs-number">31</span>,<span class="hljs-number">411</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">2</span>
</code></pre>
<p data-nodeid="10303">由于内容较多，这里省略了部分内容。</p>
<p data-nodeid="10304">在这里可以看到，每一次的返回结果都会是当前列表页提取出来的所有详情页 URL 组成的列表，我们下一步就可以用这些 URL 来接着爬取了。</p>
<h3 data-nodeid="10305">爬取详情页</h3>
<p data-nodeid="10306">拿到详情页的 URL 之后，下一步就是爬取每一个详情页然后提取信息了，首先我们定义一个爬取详情页的方法，代码如下：</p>
<pre class="lang-python" data-nodeid="10307"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_detail</span>(<span class="hljs-params">url</span>):</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_page(url,&nbsp;<span class="hljs-string">'h2'</span>)
</code></pre>
<p data-nodeid="10308">代码非常简单，就是直接调用了 scrape_page 方法，然后传入了要等待加载的节点的选择器，这里我们就直接用了 h2 了，对应的就是详情页的电影名称，如图所示。<br>
<img src="https://s0.lgstatic.com/i/image3/M01/85/2E/Cgq2xl6Nsn2ADDDkAAVLMGBzKmQ220.png" alt="" data-nodeid="10415"><br>
如果顺利运行，那么当前 Pyppeteer 就已经成功加载出详情页了，下一步就是提取里面的信息了。</p>
<p data-nodeid="10309">接下来我们再定义一个提取详情信息的方法，代码如下：</p>
<pre class="lang-python" data-nodeid="10310"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">parse_detail</span>():</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;tab.url
&nbsp;&nbsp;&nbsp;name&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorEval(<span class="hljs-string">'h2'</span>,&nbsp;<span class="hljs-string">'node&nbsp;=&gt;&nbsp;node.innerText'</span>)
&nbsp;&nbsp;&nbsp;categories&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorAllEval(<span class="hljs-string">'.categories&nbsp;button&nbsp;span'</span>,&nbsp;<span class="hljs-string">'nodes&nbsp;=&gt;&nbsp;nodes.map(node&nbsp;=&gt;&nbsp;node.innerText)'</span>)
&nbsp;&nbsp;&nbsp;cover&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorEval(<span class="hljs-string">'.cover'</span>,&nbsp;<span class="hljs-string">'node&nbsp;=&gt;&nbsp;node.src'</span>)
&nbsp;&nbsp;&nbsp;score&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorEval(<span class="hljs-string">'.score'</span>,&nbsp;<span class="hljs-string">'node&nbsp;=&gt;&nbsp;node.innerText'</span>)
&nbsp;&nbsp;&nbsp;drama&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;tab.querySelectorEval(<span class="hljs-string">'.drama&nbsp;p'</span>,&nbsp;<span class="hljs-string">'node&nbsp;=&gt;&nbsp;node.innerText'</span>)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'url'</span>:&nbsp;url,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'name'</span>:&nbsp;name,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'categories'</span>:&nbsp;categories,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;cover,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'score'</span>:&nbsp;score,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'drama'</span>:&nbsp;drama
&nbsp;&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="10311">这里我们定义了一个 parse_detail 方法，提取了 URL、名称、类别、封面、分数、简介等内容，提取方式如下：</p>
<ul data-nodeid="10312">
<li data-nodeid="10313">
<p data-nodeid="10314">URL：直接调用 tab 对象的 url 属性即可获取当前页面的 URL。</p>
</li>
<li data-nodeid="10315">
<p data-nodeid="10316">名称：由于名称只有一个节点，所以这里我们调用了 querySelectorEval 方法来提取，而不是querySelectorAllEval，第一个参数传入 h2，提取到了名称对应的节点，然后第二个参数传入提取的 pageFunction，调用了 node 的 innerText 属性提取了文本值，即电影名称。</p>
</li>
<li data-nodeid="10317">
<p data-nodeid="10318">类别：类别有多个，所以我们这里调用了 querySelectorAllEval 方法来提取，其对应的 CSS 选择器为 .categories button span，可以选中多个类别节点。接下来还是像之前提取详情页 URL 一样，pageFunction 使用 nodes 参数，然后调用 map 方法提取 node 的 innerText 就得到所有类别结果了。</p>
</li>
<li data-nodeid="10319">
<p data-nodeid="10320">封面：同样地，可以使用 CSS 选择器 .cover 直接获取封面对应的节点，但是由于其封面的 URL 对应的是 src 这个属性，所以这里提取的是 src 属性。</p>
</li>
<li data-nodeid="10321">
<p data-nodeid="10322">分数：分数对应的 CSS 选择器为 .score ，类似的原理，提取 node 的 innerText 即可。</p>
</li>
<li data-nodeid="10323">
<p data-nodeid="10324">简介：同样可以使用 CSS 选择器 .drama p 直接获取简介对应的节点，然后调用 innerText 属性提取文本即可。</p>
</li>
</ul>
<p data-nodeid="10325">最后我们将提取结果汇总成一个字典然后返回即可。</p>
<p data-nodeid="10326">接下来 main 方法里面，我们增加 scrape_detail 和 parse_detail 方法的调用，main 方法改写如下：</p>
<pre class="lang-python" data-nodeid="10327"><code data-language="python"><span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;init()
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;page&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">1</span>,&nbsp;TOTAL_PAGE&nbsp;+&nbsp;<span class="hljs-number">1</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_index(page)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_urls&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;parse_index()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;detail_url&nbsp;<span class="hljs-keyword">in</span>&nbsp;detail_urls:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;scrape_detail(detail_url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_data&nbsp;=&nbsp;<span class="hljs-keyword">await</span>&nbsp;parse_detail()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'data&nbsp;%s'</span>,&nbsp;detail_data)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">finally</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;browser.close()
</code></pre>
<p data-nodeid="10328">重新看下运行结果，运行结果如下：</p>
<pre class="lang-python" data-nodeid="10329"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">14</span>:<span class="hljs-number">12</span>:<span class="hljs-number">39</span>,<span class="hljs-number">564</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">14</span>:<span class="hljs-number">12</span>:<span class="hljs-number">42</span>,<span class="hljs-number">935</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">14</span>:<span class="hljs-number">12</span>:<span class="hljs-number">45</span>,<span class="hljs-number">781</span>&nbsp;-&nbsp;INFO:&nbsp;data&nbsp;{<span class="hljs-string">'url'</span>:&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx'</span>,&nbsp;<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'霸王别姬&nbsp;-&nbsp;Farewell&nbsp;My&nbsp;Concubine'</span>,&nbsp;<span class="hljs-string">'categories'</span>:&nbsp;[<span class="hljs-string">'剧情'</span>,&nbsp;<span class="hljs-string">'爱情'</span>],&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;<span class="hljs-string">'https://p0.meituan.net/movie/ce4da3e03e655b5b88ed31b5cd7896cf62472.jpg@464w_644h_1e_1c'</span>,&nbsp;<span class="hljs-string">'score'</span>:&nbsp;<span class="hljs-string">'9.5'</span>,&nbsp;<span class="hljs-string">'drama'</span>:&nbsp;<span class="hljs-string">'影片借一出《霸王别姬》的京戏，牵扯出三个人之间一段随时代风云变幻的爱恨情仇。段小楼（张丰毅&nbsp;饰）与程蝶衣（张国荣&nbsp;饰）是一对打小一起长大的师兄弟，两人一个演生，一个饰旦，一向配合天衣无缝，尤其一出《霸王别姬》，更是誉满京城，为此，两人约定合演一辈子《霸王别姬》。但两人对戏剧与人生关系的理解有本质不同，段小楼深知戏非人生，程蝶衣则是人戏不分。段小楼在认为该成家立业之时迎娶了名妓菊仙（巩俐&nbsp;饰），致使程蝶衣认定菊仙是可耻的第三者，使段小楼做了叛徒，自此，三人围绕一出《霸王别姬》生出的爱恨情仇战开始随着时代风云的变迁不断升级，终酿成悲剧。'</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-04</span><span class="hljs-number">-08</span>&nbsp;<span class="hljs-number">14</span>:<span class="hljs-number">12</span>:<span class="hljs-number">45</span>,<span class="hljs-number">782</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIy
</code></pre>
<p data-nodeid="10330">这里可以看到，首先先爬取了列表页，然后提取出了详情页之后接着开始爬详情页，然后提取出我们想要的电影信息之后，再接着去爬下一个详情页。</p>
<p data-nodeid="10331">这样，所有的详情页都会被我们爬取下来啦。</p>
<h3 data-nodeid="10332">数据存储</h3>
<p data-nodeid="10333">最后，我们再像之前一样添加一个数据存储的方法，为了方便，这里还是保存为 JSON 文本文件，实现如下：</p>
<pre class="lang-python" data-nodeid="10334"><code data-language="python"><span class="hljs-keyword">import</span>&nbsp;json
<span class="hljs-keyword">from</span>&nbsp;os&nbsp;<span class="hljs-keyword">import</span>&nbsp;makedirs
<span class="hljs-keyword">from</span>&nbsp;os.path&nbsp;<span class="hljs-keyword">import</span>&nbsp;exists
RESULTS_DIR&nbsp;=&nbsp;<span class="hljs-string">'results'</span>
exists(RESULTS_DIR)&nbsp;<span class="hljs-keyword">or</span>&nbsp;makedirs(RESULTS_DIR)
<span class="hljs-keyword">async</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">save_data</span>(<span class="hljs-params">data</span>):</span>
&nbsp;&nbsp;&nbsp;name&nbsp;=&nbsp;data.get(<span class="hljs-string">'name'</span>)
&nbsp;&nbsp;&nbsp;data_path&nbsp;=&nbsp;<span class="hljs-string">f'<span class="hljs-subst">{RESULTS_DIR}</span>/<span class="hljs-subst">{name}</span>.json'</span>
&nbsp;&nbsp;&nbsp;json.dump(data,&nbsp;open(data_path,&nbsp;<span class="hljs-string">'w'</span>,&nbsp;encoding=<span class="hljs-string">'utf-8'</span>),&nbsp;ensure_ascii=<span class="hljs-literal">False</span>,&nbsp;indent=<span class="hljs-number">2</span>)
</code></pre>
<p data-nodeid="10335">这里原理和之前是完全相同的，但是由于这里我们使用的是 Pyppeteer，是异步调用，所以 save_data 方法前面需要加 async。</p>
<p data-nodeid="10336">最后添加上 save_data 的调用，完整看下运行效果。</p>
<h3 data-nodeid="10337">问题排查</h3>
<p data-nodeid="10338">在运行过程中，由于 Pyppeteer 本身实现的原因，可能连续运行 20 秒之后控制台就会出现如下错误：</p>
<pre class="lang-python" data-nodeid="10339"><code data-language="python">pyppeteer.errors.NetworkError:&nbsp;Protocol&nbsp;Error&nbsp;(Runtime.evaluate):&nbsp;Session&nbsp;closed.&nbsp;Most&nbsp;likely&nbsp;the&nbsp;page&nbsp;has&nbsp;been&nbsp;closed.
</code></pre>
<p data-nodeid="10340">其原因是 Pyppeteer 内部使用了 Websocket，在 Websocket 客户端发送 ping 信号 20 秒之后仍未收到 pong 应答，就会中断连接。</p>
<p data-nodeid="10341">问题的解决方法和详情描述见 <a href="https://github.com/miyakogi/pyppeteer/issues/178" data-nodeid="10451">https://github.com/miyakogi/pyppeteer/issues/178</a>，此时我们可以通过修改 Pyppeteer 源代码来解决这个问题，对应的代码修改见：<a href="https://github.com/miyakogi/pyppeteer/pull/160/files" data-nodeid="10455">https://github.com/miyakogi/pyppeteer/pull/160/files</a>，即把 connect 方法添加 ping_interval=None, ping_timeout=None 两个参数即可。</p>
<p data-nodeid="10342">另外也可以复写一下 Connection 的实现，其解决方案同样可以在&nbsp;<a href="https://github.com/miyakogi/pyppeteer/pull/160" data-nodeid="10464">https://github.com/miyakogi/pyppeteer/pull/160</a>&nbsp;找到，如 patch_pyppeteer 的定义。</p>
<h3 data-nodeid="10343">无头模式</h3>
<p data-nodeid="10344">最后如果代码能稳定运行了，我们可以将其改为无头模式，将 HEADLESS 修改为 True 即可，这样在运行的时候就不会弹出浏览器窗口了。</p>
<h3 data-nodeid="10345">总结</h3>
<p data-nodeid="10346">本课时我们通过实例来讲解了 Pyppeteer 爬取一个完整网站的过程，从而对 Pyppeteer 的使用有进一步的掌握。</p>
<p data-nodeid="10347" class="te-preview-highlight">本节代码：<a href="https://github.com/Python3WebSpider/ScrapeDynamic2" data-nodeid="10475">https://github.com/Python3WebSpider/ScrapeDynamic2</a>。</p>

---

### 精选评论

##### *飞：
> 这个类选择器还是用不惯，而且不知道为什么我本地用xpath就可以，直接用你的代码就不行，不过这都是细枝末节了，期待后续的逆向部分，加油！！！

