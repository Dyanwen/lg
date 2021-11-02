<p data-nodeid="25901" class="">上个课时我们介绍了网页防护技术，包括接口加密和 JavaScript 压缩、加密和混淆。这就引出了一个问题，如果我们碰到了这样的网站，那该怎么去分析和爬取呢？</p>
<p data-nodeid="25902">本课时我们就通过一个案例来介绍一下这种网站的爬取思路，本课时介绍的这个案例网站不仅在 API 接口层有加密，而且前端 JavaScript 也带有压缩和混淆，其前端压缩打包工具使用了现在流行的 Webpack，混淆工具是使用了 javascript-obfuscator，这二者结合起来，前端的代码会变得难以阅读和分析。</p>
<p data-nodeid="25903">如果我们不使用 Selenium 或 Pyppeteer 等工具来模拟浏览器的形式爬取的话，要想直接从接口层面上获取数据，基本上需要一点点调试分析 JavaScript 的调用逻辑、堆栈调用关系来弄清楚整个网站加密的实现方法，我们可以称这个过程叫 JavaScript 逆向。这些接口的加密参数往往都是一些加密算法或编码的组合，完全搞明白其中的逻辑之后，我们就能把这个算法用 Python 模拟出来，从而实现接口的请求了。</p>
<h3 data-nodeid="25904">案例介绍</h3>
<p data-nodeid="27858" class="">案例的地址为：<a href="https://dynamic6.scrape.center/" data-nodeid="27862">https://dynamic6.scrape.center/</a>，页面如图所示。</p>


<p data-nodeid="25906"><img src="https://s0.lgstatic.com/i/image/M00/00/DC/Ciqc1F6qkZqANciBAA8meafvsHM491.png" alt="image.png" data-nodeid="26054"></p>
<p data-nodeid="25907">初看之下并没有什么特殊的，但仔细观察可以发现其 Ajax 请求接口和每部电影的 URL 都包含了加密参数。</p>
<p data-nodeid="25908">比如我们点击任意一部电影，观察一下 URL 的变化，如图所示。</p>
<p data-nodeid="25909"><img src="https://s0.lgstatic.com/i/image/M00/00/DC/CgqCHl6qkaOAH4oXABChHxWKtLo031.png" alt="image (1).png" data-nodeid="26059"></p>
<p data-nodeid="25910">这里我们可以看到详情页的 URL 包含了一个长字符串，看似是一个 Base64 编码的内容。</p>
<p data-nodeid="25911">那么接下来直接看看 Ajax 的请求，我们从列表页的第 1 页到第 10 页依次点一下，观察一下 Ajax 请求是怎样的，如图所示。</p>
<p data-nodeid="25912"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkbiASEZbAAoPAmLuiLs518.png" alt="image (2).png" data-nodeid="26064"></p>
<p data-nodeid="25913">可以看到 Ajax 接口的 URL 里面多了一个 token，而且不同的页码 token 是不一样的，这个 token 同样看似是一个 Base64 编码的字符串。</p>
<p data-nodeid="25914">另外更困难的是，这个接口还是有时效性的，如果我们把 Ajax 接口 URL 直接复制下来，短期内是可以访问的，但是过段时间之后就无法访问了，会直接返回 401 状态码。</p>
<p data-nodeid="25915">接下来我们再看下列表页的返回结果，比如我们打开第一个请求，看看第一部电影数据的返回结果，如图所示。</p>
<p data-nodeid="25916"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkcOAQupqAAMYHAP-Uvk656.png" alt="image (3).png" data-nodeid="26070"></p>
<p data-nodeid="29434" class="">这里我们把看似是第一部电影的返回结果全展开了，但是刚才我们观察到第一部电影的 URL 的链接却为 <a href="https://dynamic6.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx" data-nodeid="29438">https://dynamic6.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx</a>，看起来是 Base64 编码，我们解码一下，结果为 ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs*-!i-0-mb1，但是看起来似乎还是毫无规律，这个解码后的结果又是怎么来的呢？返回结果里面也并不包含这个字符串，那这又是怎么构造的呢？</p>


<p data-nodeid="25918">再然后，这仅仅是某一个详情页页面的 URL，其真实数据是通过 Ajax 加载的，那么 Ajax 请求又是怎样的呢，我们再观察下，如图所示。</p>
<p data-nodeid="25919"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkeKAfDyaAAk629L2zsE019.png" alt="image (4).png" data-nodeid="26083"></p>
<p data-nodeid="25920">好，这里我们发现其 Ajax 接口除了包含刚才所说的 URL 中携带的字符串，又多了一个 token，同样也是类似 Base64 编码的内容。</p>
<p data-nodeid="25921">那么总结下来这个网站就有如下特点：</p>
<ul data-nodeid="25922">
<li data-nodeid="25923">
<p data-nodeid="25924">列表页的 Ajax 接口参数带有加密的 token；</p>
</li>
<li data-nodeid="25925">
<p data-nodeid="25926">详情页的 URL 带有加密 id；</p>
</li>
<li data-nodeid="25927">
<p data-nodeid="25928">详情页的 Ajax 接口参数带有加密 id 和加密 token。</p>
</li>
</ul>
<p data-nodeid="25929">那如果我们要想通过接口的形式来爬取，必须要把这些加密 id 和 token 构造出来才行，而且必须要一步步来，首先我们要构造出列表页 Ajax 接口的 token 参数，然后才能获取每部电影的数据信息，然后根据数据信息构造出加密 id 和 token。</p>
<p data-nodeid="25930">OK，到现在为止我们就知道了这个网站接口的加密情况了，我们下一步就是去找这个加密实现逻辑了。</p>
<p data-nodeid="25931">由于是网页，所以其加密逻辑一定藏在前端代码中，但前面我们也说了，前端为了保护其接口加密逻辑不被轻易分析出来，会采取压缩、混淆的方式来加大分析的难度。</p>
<p data-nodeid="25932">接下来，我们就来看看这个网站的源代码和 JavaScript 文件是怎样的吧。</p>
<p data-nodeid="25933">首先看看网站源代码，我们在网站上点击右键，弹出选项菜单，然后点击“查看源代码”，可以看到结果如图所示。</p>
<p data-nodeid="25934"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkgGANjrqAASTjV0jn7Q608.png" alt="image (5).png" data-nodeid="26096"></p>
<p data-nodeid="25935">内容如下：</p>
<pre class="lang-html" data-nodeid="25936"><code data-language="html"><span class="hljs-meta">&lt;!DOCTYPE <span class="hljs-meta-keyword">html</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">html</span> <span class="hljs-attr">lang</span>=<span class="hljs-string">en</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">charset</span>=<span class="hljs-string">utf-8</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">http-equiv</span>=<span class="hljs-string">X-UA-Compatible</span> <span class="hljs-attr">content</span>=<span class="hljs-string">"IE=edge"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">name</span>=<span class="hljs-string">viewport</span> <span class="hljs-attr">content</span>=<span class="hljs-string">"width=device-width,initial-scale=1"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">icon</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/favicon.ico</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Scrape | Movie<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/css/chunk-19c920f8.2a6496e0.css</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">prefetch</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/css/chunk-2f73b8f3.5b462e16.css</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">prefetch</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/js/chunk-19c920f8.c3a1129d.js</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">prefetch</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/js/chunk-2f73b8f3.8f2fc3cd.js</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">prefetch</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/js/chunk-4dec7ef0.e4c2b130.js</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">prefetch</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/css/app.ea9d802a.css</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">preload</span> <span class="hljs-attr">as</span>=<span class="hljs-string">style</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/js/app.5ef0d454.js</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">preload</span> <span class="hljs-attr">as</span>=<span class="hljs-string">script</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/js/chunk-vendors.77daf991.js</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">preload</span> <span class="hljs-attr">as</span>=<span class="hljs-string">script</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">/css/app.ea9d802a.css</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">stylesheet</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">noscript</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">strong</span>&gt;</span>We're sorry but portal doesn't work properly without JavaScript enabled. Please enable it to continue.<span class="hljs-tag">&lt;/<span class="hljs-name">strong</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">noscript</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">app</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">/js/chunk-vendors.77daf991.js</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">/js/app.5ef0d454.js</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</code></pre>
<p data-nodeid="25937">这是一个典型的 SPA（单页 Web 应用）的页面， 其 JavaScript 文件名带有编码字符、chunk、vendors 等关键字，整体就是经过 Webpack 打包压缩后的源代码，目前主流的前端开发，如 Vue.js、React.js 的输出结果都是类似这样的结果。</p>
<p data-nodeid="25938">好，那么我们再看下其 JavaScript 代码是什么样子的，我们在开发者工具中打开 Sources 选项卡下的 Page 选项卡，然后打开 js 文件夹，这里我们就能看到 JavaScript 的源代码，如图所示。</p>
<p data-nodeid="25939"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/Ciqc1F6qkjGAGyKRAAYIIGMFEZw194.png" alt="image (6).png" data-nodeid="26102"></p>
<p data-nodeid="25940">我们随便复制一些出来，看看是什么样子的，结果如下：</p>
<pre class="lang-js" data-nodeid="25941"><code data-language="js">\(<span class="hljs-built_in">window</span>\[<span class="hljs-string">'webpackJsonp'</span>\]=<span class="hljs-built_in">window</span>\[<span class="hljs-string">'webpackJsonp'</span>\]\|\|\[\]\)\[<span class="hljs-string">'push'</span>\]\(\[\[<span class="hljs-string">'chunk\-19c920f8'</span>\]\,\{<span class="hljs-string">'5a19'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x3cb7c3\,\_0x5cb6ab\,\_0x5f5010\</span>)\</span>{\}\,<span class="hljs-string">'c6bf'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x1846fe\,\_0x459c04\,\_0x1ff8e3\</span>)\</span>{\}\,<span class="hljs-string">'ca9c'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x195201\,\_0xc41ead\,\_0x1b389c\</span>)\</span>{<span class="hljs-string">'use strict'</span>;<span class="hljs-keyword">var</span> \_0x468b4e=\_0x1b389c\(<span class="hljs-string">'5a19'</span>\)\,\_0x232454=\_0x1b389c[<span class="hljs-string">'n'</span>](_0x468b4e);\_0x232454[<span class="hljs-string">'a'</span>];},<span class="hljs-string">'d504'</span>:...,[\_0xd670a1[<span class="hljs-string">'\_v'</span>](_0xd670a1%<span class="hljs-number">5</span>B<span class="hljs-string">'_s'</span>%<span class="hljs-number">5</span>D(_0x2227b6)+<span class="hljs-string">'%5Cx0a%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20%5Cx20'</span>)]);}),<span class="hljs-number">0x1</span>),\_0x4ef533(<span class="hljs-string">'div'</span>,{<span class="hljs-string">'staticClass'</span>:<span class="hljs-string">'m-v-sm\x20info'</span>},[\_0x4ef533(<span class="hljs-string">'span'</span>,[\_0xd670a1[<span class="hljs-string">'\_v'</span>](_0xd670a1%<span class="hljs-number">5</span>B<span class="hljs-string">'_s'</span>%<span class="hljs-number">5</span>D(_0x1cc7eb%<span class="hljs-number">5</span>B<span class="hljs-string">'regions'</span>%<span class="hljs-number">5</span>D%<span class="hljs-number">5</span>B<span class="hljs-string">'join'</span>%<span class="hljs-number">5</span>D(<span class="hljs-string">'%E3%80%81'</span>)))]),\_0x4ef533(<span class="hljs-string">'span'</span>,[\_0xd670a1[<span class="hljs-string">'\_v'</span>](<span class="hljs-string">'%5Cx20/%5Cx20'</span>)]),\_0x4ef533(<span class="hljs-string">'span'</span>,[\_0xd670a1[<span class="hljs-string">'\_v'</span>](_0xd670a1%<span class="hljs-number">5</span>B<span class="hljs-string">'_s'</span>%<span class="hljs-number">5</span>D(_0x1cc7eb%<span class="hljs-number">5</span>B<span class="hljs-string">'minute'</span>%<span class="hljs-number">5</span>D)+<span class="hljs-string">'%5Cx20%E5%88%86%E9%92%9F'</span>)])]),\_0x4ef533(<span class="hljs-string">'div'</span>,...,\_0x4ef533(<span class="hljs-string">'el-col'</span>,{<span class="hljs-string">'attrs'</span>:{<span class="hljs-string">'xs'</span>:<span class="hljs-number">0x5</span>,<span class="hljs-string">'sm'</span>:<span class="hljs-number">0x5</span>,<span class="hljs-string">'md'</span>:<span class="hljs-number">0x4</span>}},[\_0x4ef533(<span class="hljs-string">'p'</span>,{<span class="hljs-string">'staticClass'</span>:<span class="hljs-string">'score\x20m-t-md\x20m-b-n-sm'</span>},[\_0xd670a1[<span class="hljs-string">'\_v'</span>](_0xd670a1%<span class="hljs-number">5</span>B<span class="hljs-string">'_s'</span>%<span class="hljs-number">5</span>D(_0x1cc7eb%<span class="hljs-number">5</span>B<span class="hljs-string">'score'</span>%<span class="hljs-number">5</span>D%<span class="hljs-number">5</span>B<span class="hljs-string">'toFixed'</span>%<span class="hljs-number">5</span>D(<span class="hljs-number">0x1</span>)))\]\)\,\_0x4ef533\(<span class="hljs-string">'p'</span>\,\[\_0x4ef533\(<span class="hljs-string">'el\-rate'</span>\,\{<span class="hljs-string">'attrs'</span>:\{<span class="hljs-string">'value'</span>:\_0x1cc7eb\[<span class="hljs-string">'score'</span>\]/<span class="hljs-number">0x2</span>\,<span class="hljs-string">'disabled'</span>:<span class="hljs-string">''</span>\,<span class="hljs-string">'max'</span>:<span class="hljs-number">0x5</span>\,<span class="hljs-string">'text\-color'</span>:<span class="hljs-string">'\#ff9900'</span>\}\}\)\]\,<span class="hljs-number">0x1</span>\)\]\)\]\,<span class="hljs-number">0x1</span>\)\]\,<span class="hljs-number">0x1</span>\);\}\)\,<span class="hljs-number">0x1</span>\)\]\,<span class="hljs-number">0x1</span>\)\,\_0x4ef533\(<span class="hljs-string">'el\-row'</span>\,\[\_0x4ef533\(<span class="hljs-string">'el\-col'</span>\,\{<span class="hljs-string">'attrs'</span>:\{<span class="hljs-string">'span'</span>:<span class="hljs-number">0xa</span>\,<span class="hljs-string">'offset'</span>:<span class="hljs-number">0xb</span>\}\}\,\[\_0x4ef533\(<span class="hljs-string">'div'</span>\,\{<span class="hljs-string">'staticClass'</span>:<span class="hljs-string">'pagination\x20m\-v\-lg'</span>\}\,\[\_0x4ef533\(<span class="hljs-string">'el\-pagination'</span>\,\.\.\.:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x347c29\</span>)\</span>{\_0xd670a1\[<span class="hljs-string">'page'</span>\]=\_0x347c29;\}\,<span class="hljs-string">'update:current\-page'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x79754e\</span>)\</span>{\_0xd670a1\[<span class="hljs-string">'page'</span>\]=\_0x79754e;\}\}\}\)\]\,<span class="hljs-number">0x1</span>\)\]\)\]\,<span class="hljs-number">0x1</span>\)\]\,<span class="hljs-number">0x1</span>\);\}\,\_0x357ebc=\[\]\,\_0x18b11a=\_0x1a3e60\(<span class="hljs-string">'7d92'</span>\)\,\_0x4369=\_0x1a3e60\(<span class="hljs-string">'3e22'</span>\)\,\.\.\.;<span class="hljs-keyword">var</span> \_0x498df8=\.\.\.\[<span class="hljs-string">'then'</span>\]\(<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x59d600\</span>)\</span>{<span class="hljs-keyword">var</span> \_0x1249bc=\_0x59d600\[<span class="hljs-string">'data'</span>\]\,\_0x10e324=\_0x1249bc\[<span class="hljs-string">'results'</span>\]\,\_0x47d41b=\_0x1249bc\[<span class="hljs-string">'count'</span>\];\_0x531b38\[<span class="hljs-string">'loading'</span>\]=\!<span class="hljs-number">0x1</span>\,\_0x531b38\[<span class="hljs-string">'movies'</span>\]=\_0x10e324\,\_0x531b38\[<span class="hljs-string">'total'</span>\]=\_0x47d41b;\}\);\}\}\}\,\_0x28192a=\_0x5f39bd\,\_0x5f5978=\(\_0x1a3e60\(<span class="hljs-string">'ca9c'</span>\)\,\_0x1a3e60\(<span class="hljs-string">'eb45'</span>\)\,\_0x1a3e60\(<span class="hljs-string">'2877'</span>\)\)\,\_0x3fae81=<span class="hljs-built_in">Object</span>\(\_0x5f5978\[<span class="hljs-string">'a'</span>\]\)\(\_0x28192a\,\_0x443d6e\,\_0x357ebc\,\!<span class="hljs-number">0x1</span>\,<span class="hljs-literal">null</span>\,<span class="hljs-string">'724ecf3b'</span>\,<span class="hljs-literal">null</span>\);\_0x6f764c\[<span class="hljs-string">'default'</span>\]=\_0x3fae81\[<span class="hljs-string">'exports'</span>\];\}\,<span class="hljs-string">'eb45'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>\(<span class="hljs-params">\_0x1d3c3c\,\_0x52e11c\,\_0x3f1276\</span>)\</span>{<span class="hljs-string">'use strict'</span>;<span class="hljs-keyword">var</span> \_0x79046c=\_0x3f1276\(<span class="hljs-string">'c6bf'</span>\)\,\_0x219366=\_0x3f1276[<span class="hljs-string">'n'</span>](_0x79046c);\_0x219366[<span class="hljs-string">'a'</span>];}}]);
</code></pre>
<p data-nodeid="25942">就是这种感觉，可以看到一些变量都是一些十六进制字符串，而且代码全被压缩了。</p>
<p data-nodeid="25943">没错，我们就是要从这里面找出 token 和 id 的构造逻辑，看起来是不是很崩溃？</p>
<p data-nodeid="25944">要完全分析出整个网站的加密逻辑还是有一定难度的，不过不用担心，我们本课时会一步步地讲解逆向的思路、方法和技巧，如果你能跟着这个过程学习完，相信还是能学会一定的 JavaScript 逆向技巧的。</p>
<blockquote data-nodeid="25945">
<p data-nodeid="25946">为了适当降低难度，本课时案例的 JavaScript 混淆其实并没有设置的特别复杂，并没有开启字符串编码、控制流扁平化等混淆方式。</p>
</blockquote>
<h3 data-nodeid="25947">列表页 Ajax 入口寻找</h3>
<p data-nodeid="25948">接下来，我们就开始第一步入口的寻找吧，这里简单介绍两种寻找入口的方式：</p>
<ul data-nodeid="25949">
<li data-nodeid="25950">
<p data-nodeid="25951">全局搜索标志字符串；</p>
</li>
<li data-nodeid="25952">
<p data-nodeid="25953">设置 Ajax 断点。</p>
</li>
</ul>
<h4 data-nodeid="25954">全局搜索标志字符串</h4>
<p data-nodeid="25955">一些关键的字符串通常会作为找寻 JavaScript 混淆入口的依据，我们可以通过全局搜索的方式来查找，然后根据搜索到的结果大体观察是否是我们想找的入口。</p>
<p data-nodeid="25956">然后，我们重新打开列表页的 Ajax 接口，看下请求的 Ajax 接口，如图所示。</p>
<p data-nodeid="25957" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qknSAaJi9AAZGLdfg3f4342.png" alt="image (7).png" data-nodeid="26117"></p>
<p data-nodeid="31014" class="">这里的 Ajax 接口的 URL 为 <a href="https://dynamic6.scrape.center/api/movie/?limit=10&amp;offset=0&amp;token=NTRhYWJhNzAyYTZiMTc0ZThkZTExNzBiNTMyMDJkN2UxZWYyMmNiZCwxNTg4MTc4NTYz" data-nodeid="31022">https://dynamic6.scrape.center/api/movie/?limit=10&amp;offset=0&amp;token=NTRhYWJhNzAyYTZiMTc0ZThkZTExNzBiNTMyMDJkN2UxZWYyMmNiZCwxNTg4MTc4NTYz</a>，可以看到带有 offset、limit、token 三个参数，入口寻找关键就是找 token，我们全局搜索下 token 是否存在，可以点击开发者工具右上角的下拉选项卡，然后点击 Search，如图所示。</p>


<p data-nodeid="25959"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/Ciqc1F6qkoqANZ5EAAXALwrapv4539.png" alt="image (8).png" data-nodeid="26129"></p>
<p data-nodeid="25960">这样我们就能进入到一个全局搜索模式，我们搜索 token，可以看到的确搜索到了几个结果，如图所示。</p>
<p data-nodeid="25961"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkpWAbJRjAAdFuzcwiio327.png" alt="image (9).png" data-nodeid="26133"></p>
<p data-nodeid="25962">观察一下，下面的两个结果可能是我们想要的，我们点击进入第一个看下，定位到了一个 JavaScript 文件，如图所示。</p>
<p data-nodeid="25963"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/Ciqc1F6qkpyAJI-uAAWL2y8otZc090.png" alt="image (10).png" data-nodeid="26137"></p>
<p data-nodeid="25964">这时候可以看到整个代码都是压缩过的，只有一行，不好看，我们可以点击左下角的 {} 按钮，美化一下 JavaScript 代码，如图所示。</p>
<p data-nodeid="25965"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/CgqCHl6qkqSAUtJ4AAWPHUlveFY575.png" alt="image (11).png" data-nodeid="26141"></p>
<p data-nodeid="25966">美化后的结果就是这样子了，如图所示。</p>
<p data-nodeid="25967"><img src="https://s0.lgstatic.com/i/image/M00/00/DD/Ciqc1F6qkq2AEhDXAAgZeF5_6Xk163.png" alt="image (12).png" data-nodeid="26145"></p>
<p data-nodeid="25968">这时可以看到这里弹出来了一个新的选项卡，其名称是 JavaScript 文件名加上了 :formatted，代表格式化后代码结果，在这里我们再次定位到 token 观察一下。</p>
<p data-nodeid="25969">可以看到这里有 limit、offset、token，然后观察下其他的逻辑，基本上能够确定这就是构造 Ajax 请求的地方了，如果不是的话可以继续搜索其他的文件观察下。</p>
<p data-nodeid="25970">那现在，混淆的入口点我们就成功找到了，这是一个首选的找入口的方法。</p>
<h4 data-nodeid="25971">XHR 断点</h4>
<p data-nodeid="25972">由于这里的 token 字符串并没有被混淆，所以上面的这个方法是奏效的。之前我们也讲过，这种字符串由于非常容易成为找寻入口点的依据，所以这样的字符串也会被混淆成类似 Unicode、Base64、RC4 的一些编码形式，这样我们就没法轻松搜索到了。</p>
<p data-nodeid="25973">那如果遇到这种情况，我们该怎么办呢？这里再介绍一种通过打 XHR 断点的方式来寻找入口。</p>
<p data-nodeid="25974">XHR 断点，顾名思义，就是在发起 XHR 的时候进入断点调试模式，JavaScript 会在发起 Ajax 请求的时候停住，这时候我们可以通过当前的调用栈的逻辑顺着找到入口。怎么设置呢？我们可以在 Sources 选项卡的右侧，XHR/fetch Breakpoints 处添加一个断点选项。</p>
<p data-nodeid="25975">首先点击 + 号，然后输入匹配的 URL 内容，由于 Ajax 接口的形式是 /api/movie/?limit=10... 这样的格式，所这里我们就截取一段填进去就好了，这里填的就是 /api/movie，如图所示。</p>
<p data-nodeid="25976"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/Ciqc1F6qkxqAK4pMAAhc_XTSt_Y367.png" alt="image (13).png" data-nodeid="26156"></p>
<p data-nodeid="25977">添加完毕之后重新刷新页面，可以发现进入了断点模式，如图所示。</p>
<p data-nodeid="25978"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qkyKAVRkUAAUv1oHhpbk473.png" alt="image (14).png" data-nodeid="26160"></p>
<p data-nodeid="25979">好，接下来我们重新点下 {} 格式化代码，看看断点是在哪里，如图所示。</p>
<p data-nodeid="25980"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/Ciqc1F6qkzyAQZcQAAguLqHyraA259.png" alt="image (15).png" data-nodeid="26164"></p>
<p data-nodeid="25981">那这里看到有个 send 的字符，我们可以初步猜测这就是相当于发送 Ajax 请求的一瞬间。</p>
<p data-nodeid="25982">到了这里感觉 Ajax 马上就要发出去了，是不是有点太晚了，我们想找的是构造 Ajax 的时刻来分析 Ajax 参数啊！不用担心，这里我们通过调用栈就可以找回去。我们点击右侧的 Call Stack，这里记录了 JavaScript 的方法逐层调用过程，如图所示。</p>
<p data-nodeid="25983"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/Ciqc1F6qk0qAJ04jAAiOvyuo0wU233.png" alt="image (16).png" data-nodeid="26169"></p>
<p data-nodeid="25984">这里当前指向的是一个名字为 anonymouns，也就是匿名的调用，在它的下方就显示了调用这个 anonymouns 的方法，名字叫作 _0x594ca1，然后再下一层就又显示了调用 _0x594a1 这个方法的方法，依次类推。</p>
<p data-nodeid="25985">这里我们可以逐个往下查找，然后通过一些观察看看有没有 token 这样的信息，就能找到对应的位置了，最后我们就可以找到 onFetchData 这个方法里面实现了这个 token 的构造逻辑，这样我们也成功找到 token 的参数构造的位置了，如图所示。</p>
<p data-nodeid="25986"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk1qASl3iAAfB8DQP-C0830.png" alt="image (17).png" data-nodeid="26178"></p>
<p data-nodeid="25987">好，到现在为止我们就通过两个方法找到入口点了。</p>
<p data-nodeid="25988">其实还有其他的寻找入口的方式，比如 Hook 关键函数的方式，稍后的课程里我们会讲到，这里就暂时不讲了。</p>
<h3 data-nodeid="25989">列表页加密逻辑寻找</h3>
<p data-nodeid="25990">接下来我们已经找到 token 的位置了，可以观察一下这个 token 对应的变量叫作 _0xa70fc9，所以我们的关键就是要找这个变量是哪里来的了。</p>
<p data-nodeid="25991">怎么找呢？我们打个断点看下这个变量是在哪里生成的就好了，我们在对应的行打一个断点，如果打了刚才的 XHR 断点的话可以先取消掉，如图所示。</p>
<p data-nodeid="25992"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk2iAXuw4AAfT4EPsbZI161.png" alt="image (18).png" data-nodeid="26188"></p>
<p data-nodeid="25993">这时候我们就设置了一个新的断点了。由于只有一个断点，可以重新刷新下网页，这时候我们会发现网页停在了新的断点上面。</p>
<p data-nodeid="25994"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/Ciqc1F6qk5KAbR6NAAeJXqFY-QQ969.png" alt="image (19).png" data-nodeid="26192"></p>
<p data-nodeid="25995">这里我们就可以观察下运行的一些变量了，比如我们把鼠标放在各个变量上面去，可以看到变量的一些值和类型，比如我们看 _0x18b11a 这个变量，会有一个浮窗显示，如图所示。</p>
<p data-nodeid="25996"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/Ciqc1F6qk5-AfIb1AAdTDoapgWc715.png" alt="image (20).png" data-nodeid="26198"></p>
<p data-nodeid="25997">另外我们还可以通过在右侧的 Watch 面板添加想要查看的变量名称，如这行代码的内容为：</p>
<pre class="lang-js" data-nodeid="25998"><code data-language="js">, _0xa70fc9 = <span class="hljs-built_in">Object</span>(_0x18b11a[<span class="hljs-string">'a'</span>])(<span class="hljs-keyword">this</span>[<span class="hljs-string">'$store'</span>][<span class="hljs-string">'state'</span>][<span class="hljs-string">'url'</span>][<span class="hljs-string">'index'</span>]);
</code></pre>
<p data-nodeid="25999">我们比较感兴趣的可能就是 _0x18b11a 还有 this 里面的这个值了，我们可以展开 Watch 面板，然后点击 + 号，把想看的变量添加到 Watch 面板里面，如图所示。</p>
<p data-nodeid="26000"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk9eAGFwlAAedDDa3zXI764.png" alt="image (21).png" data-nodeid="26205"></p>
<p data-nodeid="26001">观察下可以发现 _0x18b11a 是一个 Object，它有个 a 属性，其值是一个 function，然后 this['$store']['state']['url']['index'] 的值其实就是 /api/movie，就是 Ajax 请求 URL 的 Path。_0xa70fc9 就是调用了前者这个 function 然后传入了 /api/movie 得到的。</p>
<p data-nodeid="26002">那么下一步就是去寻找这个 function 在哪里了，我们可以把 Watch 面板的 _0x18b11a 展开，这里会显示一个 FunctionLocation，就是这个 function 的代码位置，如图所示。</p>
<p data-nodeid="26003"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk-eAetpKAAfZ8WF5vF8742.png" alt="image (22).png" data-nodeid="26237"></p>
<p data-nodeid="26004">点击进入之后发现其仍然是未格式化的代码，再次点击 {} 格式化代码。</p>
<p data-nodeid="26005">这时候我们就进入了一个新的名字为 _0xc9e475 的方法里面，这个方法里面应该就是 token 的生成逻辑了，我们再打上断点，然后执行面板右上角蓝色箭头状的 Resume 按钮，如图所示。</p>
<p data-nodeid="26006"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk_KAUBCyAAhWMyGe2CQ797.png" alt="image (23).png" data-nodeid="26244"></p>
<p data-nodeid="26007">这时候发现我们已经单步执行到这个位置了。</p>
<p data-nodeid="26008">接下来我们不断进行单步调试，观察这里面的执行逻辑和每一步调试过程中结果都有什么变化，如图所示。</p>
<p data-nodeid="26009"><img src="https://s0.lgstatic.com/i/image/M00/00/DE/CgqCHl6qk_2AIAG8AActuiud1jI582.png" alt="image (24).png" data-nodeid="26249"></p>
<p data-nodeid="26010">在每步的执行过程中，我们可以发现一些运行值会被打到代码的右侧并带有高亮表示，同时在 watch 面板还能看到每步的变量的具体结果。</p>
<p data-nodeid="26011">最后我们总结出这个 token 的构造逻辑如下：</p>
<ul data-nodeid="26012">
<li data-nodeid="26013">
<p data-nodeid="26014">传入的 /api/movie 会构造一个初始化列表，变量命名为 _0x3dde76。</p>
</li>
<li data-nodeid="26015">
<p data-nodeid="26016">获取当前的时间戳，命名为 _0x4c50b4，push 到 _0x3dde76 这个变量里面。</p>
</li>
<li data-nodeid="26017">
<p data-nodeid="26018">将 _0x3dde76 变量用“,”拼接，然后进行 SHA1 编码，命名为 _0x46ba68。</p>
</li>
<li data-nodeid="26019">
<p data-nodeid="26020">将 _0x46ba68 （SHA1 编码的结果）和 _0x4c50b4 （时间戳）用逗号拼接，命名为 _0x495a44。</p>
</li>
<li data-nodeid="26021">
<p data-nodeid="26022">将 _0x495a44 进行 Base64 编码，命名为 _0x2a93f2，得到最后的 token。</p>
</li>
</ul>
<p data-nodeid="26023">以上的一些逻辑经过反复的观察就可以比较轻松地总结出来了，其中有些变量可以实时查看，同时也可以自己输入到控制台上进行反复验证，相信总结出这个结果并不难。</p>
<p data-nodeid="26024">好，那现在加密逻辑我们就分析出来啦，基本的思路就是：</p>
<ul data-nodeid="26025">
<li data-nodeid="26026">
<p data-nodeid="26027">先将 /api/movie 放到一个列表里面；</p>
</li>
<li data-nodeid="26028">
<p data-nodeid="26029">列表中加入当前时间戳；</p>
</li>
<li data-nodeid="26030">
<p data-nodeid="26031">将列表内容用逗号拼接；</p>
</li>
<li data-nodeid="26032">
<p data-nodeid="26033">将拼接的结果进行 SHA1 编码；</p>
</li>
<li data-nodeid="26034">
<p data-nodeid="26035">将编码的结果和时间戳再次拼接；</p>
</li>
<li data-nodeid="26036">
<p data-nodeid="26037">将拼接后的结果进行 Base64 编码。</p>
</li>
</ul>
<p data-nodeid="26038">验证下逻辑没问题的话，我们就可以用 Python 来实现出来啦。</p>
<h3 data-nodeid="26039">Python 实现列表页的爬取</h3>
<p data-nodeid="26040">要用 Python 实现这个逻辑，我们需要借助于两个库，一个是 hashlib，它提供了 sha1 方法；另外一个是 base64 库，它提供了 b64encode 方法对结果进行 Base64 编码。<br>
代码实现如下：</p>
<pre class="lang-js" data-nodeid="26291"><code data-language="js"><span class="hljs-keyword">import</span> hashlib
<span class="hljs-keyword">import</span> time
<span class="hljs-keyword">import</span> base64
<span class="hljs-keyword">from</span> typing <span class="hljs-keyword">import</span> List, Any
<span class="hljs-keyword">import</span> requests

INDEX\_URL = <span class="hljs-string">'https://dynamic6.scrape.center/api/movie?limit={limit}&amp;offset={offset}&amp;token={token}'</span>
LIMIT = <span class="hljs-number">10</span>
OFFSET = <span class="hljs-number">0</span>

def get\_token(args: List[Any]):
timestamp = str(int(time.time()))
args.append(timestamp)
sign = hashlib.sha1(<span class="hljs-string">','</span>.join(args).encode(<span class="hljs-string">'utf-8'</span>)).hexdigest()
<span class="hljs-keyword">return</span> base64.b64encode(<span class="hljs-string">','</span>.join([sign, timestamp]).encode(<span class="hljs-string">'utf-8'</span>)).decode(<span class="hljs-string">'utf-8'</span>)

args = [<span class="hljs-string">'/api/movie'</span>]
token = get\_token(args=args)
index\_url = INDEX\_URL.format(limit=LIMIT, offset=OFFSET, token=token)
response = requests.get(index\_url)
print(<span class="hljs-string">'response'</span>, response.json())
</code></pre>

<p data-nodeid="26042" class="">这里我们就根据上面的逻辑把加密流程实现出来了，这里我们先模拟爬取了第一页的内容，最后运行一下就可以得到最终的输出结果了。</p>

---

### 精选评论

##### k：
> 这一课真好！

##### **函：
> 没混淆的做多了，突然遇到混淆的一时有点懵😂😂😂

##### **翔：
> 赞

##### *鸟：
> 学到了！

