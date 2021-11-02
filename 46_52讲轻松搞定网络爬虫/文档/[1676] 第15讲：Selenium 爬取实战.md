<p data-nodeid="123809" class="">在上一课时我们学习了 Selenium 的基本用法，本课时我们就来结合一个实际的案例来体会一下 Selenium 的适用场景以及使用方法。</p>
<h3 data-nodeid="123810">准备工作</h3>
<p data-nodeid="123811">在本课时开始之前，请确保已经做好了如下准备工作：</p>
<ul data-nodeid="123812">
<li data-nodeid="123813">
<p data-nodeid="123814">安装好 Chrome 浏览器并正确配置了 ChromeDriver。</p>
</li>
<li data-nodeid="123815">
<p data-nodeid="123816">安装好 Python （至少为 3.6 版本）并能成功运行 Python 程序。</p>
</li>
<li data-nodeid="123817">
<p data-nodeid="123818">安装好了 Selenium 相关的包并能成功用 Selenium 打开 Chrome 浏览器。</p>
</li>
</ul>
<h3 data-nodeid="123819">适用场景</h3>
<p data-nodeid="123820">在前面的实战案例中，有的网页我们可以直接用 requests 来爬取，有的可以直接通过分析 Ajax 来爬取，不同的网站类型有其适用的爬取方法。</p>
<p data-nodeid="123821">Selenium 同样也有其适用场景。对于那些带有 JavaScript 渲染的网页，我们多数情况下是无法直接用 requests 爬取网页源码的，不过在有些情况下我们可以直接用 requests 来模拟 Ajax 请求来直接得到数据。</p>
<p data-nodeid="123822">然而在有些情况下 Ajax 的一些请求接口可能带有一些加密参数，如 token、sign 等等，如果不分析清楚这些参数是怎么生成的话，我们就难以模拟和构造这些参数。怎么办呢？这时候我们可以直接选择使用 Selenium 驱动浏览器渲染的方式来另辟蹊径，实现所见即所得的爬取，这样我们就无需关心在这个网页背后发生了什么请求、得到什么数据以及怎么渲染页面这些过程，我们看到的页面就是最终浏览器帮我们模拟了 Ajax 请求和 JavaScript 渲染得到的最终结果，而 Selenium 正好也能拿到这个最终结果，相当于绕过了 Ajax 请求分析和模拟的阶段，直达目标。</p>
<p data-nodeid="123823">然而 Selenium 当然也有其局限性，它的爬取效率较低，有些爬取需要模拟浏览器的操作，实现相对烦琐。不过在某些场景下也不失为一种有效的爬取手段。</p>
<h3 data-nodeid="123824">爬取目标</h3>
<p data-nodeid="126447" class="">本课时我们就拿一个适用 Selenium 的站点来做案例，其链接为：<a href="https://dynamic2.scrape.center/" data-nodeid="126451">https://dynamic2.scrape.center/</a>，还是和之前一样的电影网站，页面如图所示。</p>


<p data-nodeid="123826"><img src="https://s0.lgstatic.com/i/image3/M01/7E/EC/Cgq2xl6Bvw-AStH9AASuzBi8U3Y686.png" alt="" data-nodeid="123927"></p>
<p data-nodeid="123827">初看之下页面和之前也没有什么区别，但仔细观察可以发现其 Ajax 请求接口和每部电影的 URL 都包含了加密参数。</p>
<p data-nodeid="123828">比如我们点击任意一部电影，观察一下 URL 的变化，如图所示。</p>
<p data-nodeid="123829"><img src="https://s0.lgstatic.com/i/image3/M01/05/D6/Ciqah16Bvw-AZFzUAAWYkjYkT8Y467.png" alt="" data-nodeid="123931"></p>
<p data-nodeid="123830">这里我们可以看到详情页的 URL 和之前就不一样了，在之前的案例中，URL 的 detail 后面本来直接跟的是 id，如 1、2、3 等数字，但是这里直接变成了一个长字符串，看似是一个 Base64 编码的内容，所以这里我们无法直接根据规律构造详情页的 URL 了。</p>
<p data-nodeid="123831">好，那么接下来我们直接看看 Ajax 的请求，我们从列表页的第 1 页到第 10 页依次点一下，观察一下 Ajax 请求是怎样的，如图所示。</p>
<p data-nodeid="123832"><img src="https://s0.lgstatic.com/i/image3/M01/7E/ED/Cgq2xl6Bvw-AI4IOAAO62yR--O0446.png" alt="" data-nodeid="123935"></p>
<p data-nodeid="123833">可以看到这里接口的参数比之前多了一个 token，而且每次请求的 token 都是不同的，这个 token 同样看似是一个 Base64 编码的字符串。更困难的是，这个接口还是有时效性的，如果我们把 Ajax 接口 URL 直接复制下来，短期内是可以访问的，但是过段时间之后就无法访问了，会直接返回 401 状态码。</p>
<p data-nodeid="123834">那现在怎么办呢？之前我们可以直接用 requests 来构造 Ajax 请求，但现在 Ajax 请求接口带了这个 token，而且还是可变的，现在我们也不知道 token 的生成逻辑，那就没法直接通过构造 Ajax 请求的方式来爬取了。这时候我们可以把 token 的生成逻辑分析出来再模拟 Ajax 请求，但这种方式相对较难。所以这里我们可以直接用 Selenium 来绕过这个阶段，直接获取最终 JavaScript 渲染完成的页面源码，再提取数据就好了。</p>
<p data-nodeid="123835">所以本课时我们要完成的目标有：</p>
<ul data-nodeid="123836">
<li data-nodeid="123837">
<p data-nodeid="123838">通过 Selenium 遍历列表页，获取每部电影的详情页 URL。</p>
</li>
<li data-nodeid="123839">
<p data-nodeid="123840">通过 Selenium 根据上一步获取的详情页 URL 爬取每部电影的详情页。</p>
</li>
<li data-nodeid="123841">
<p data-nodeid="123842">提取每部电影的名称、类别、分数、简介、封面等内容。</p>
</li>
</ul>
<h3 data-nodeid="123843">爬取列表页</h3>
<p data-nodeid="123844">首先要我们要做如下初始化的工作，代码如下：</p>
<pre class="lang-python" data-nodeid="124100"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;webdriver
<span class="hljs-keyword">from</span>&nbsp;selenium.common.exceptions&nbsp;<span class="hljs-keyword">import</span>&nbsp;TimeoutException
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.common.by&nbsp;<span class="hljs-keyword">import</span>&nbsp;By
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support&nbsp;<span class="hljs-keyword">import</span>&nbsp;expected_conditions&nbsp;<span class="hljs-keyword">as</span>&nbsp;EC
<span class="hljs-keyword">from</span>&nbsp;selenium.webdriver.support.wait&nbsp;<span class="hljs-keyword">import</span>&nbsp;WebDriverWait
<span class="hljs-keyword">import</span>&nbsp;logging
logging.basicConfig(level=logging.INFO,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format=<span class="hljs-string">'%(asctime)s&nbsp;-&nbsp;%(levelname)s:&nbsp;%(message)s'</span>)
INDEX_URL&nbsp;=&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/page/{page}'</span>
TIME_OUT&nbsp;=&nbsp;<span class="hljs-number">10</span>
TOTAL_PAGE&nbsp;=&nbsp;<span class="hljs-number">10</span>
browser&nbsp;=&nbsp;webdriver.Chrome()
wait&nbsp;=&nbsp;WebDriverWait(browser,&nbsp;TIME_OUT)
</code></pre>

<p data-nodeid="123846">首先我们导入了一些必要的 Selenium 模块，包括 webdriver、WebDriverWait 等等，后面我们会用到它们来实现页面的爬取和延迟等待等设置。然后接着定义了一些变量和日志配置，和之前几课时的内容是类似的。接着我们使用 Chrome 类生成了一个 webdriver 对象，赋值为 browser，这里我们可以通过 browser 调用 Selenium 的一些 API 来完成一些浏览器的操作，如截图、点击、下拉等等。最后我们又声明了一个 WebDriverWait 对象，利用它我们可以配置页面加载的最长等待时间。</p>
<p data-nodeid="125271" class="">好，接下来我们就观察下列表页，实现列表页的爬取吧。这里可以观察到列表页的 URL 还是有一定规律的，比如第一页为&nbsp;<a href="https://dynamic2.scrape.center/page/1" data-nodeid="125275">https://dynamic2.scrape.center/page/1</a>，页码就是 URL 最后的数字，所以这里我们可以直接来构造每一页的 URL。</p>


<p data-nodeid="123848">那么每个列表页要怎么判断是否加载成功了呢？很简单，当页面出现了我们想要的内容就代表加载成功了。在这里我们就可以用 Selenium 的隐式判断条件来判定，比如每部电影的信息区块的 CSS 选择器为 #index .item，如图所示。</p>
<p data-nodeid="123849"><img src="https://s0.lgstatic.com/i/image3/M01/05/D6/Ciqah16Bvw-AdZyFAAQm5r5A8I0442.png" alt="" data-nodeid="123952"></p>
<p data-nodeid="123850">所以这里我们直接使用 visibility_of_all_elements_located 判断条件加上 CSS 选择器的内容即可判定页面有没有加载出来，配合 WebDriverWait 的超时配置，我们就可以实现 10 秒的页面的加载监听。如果 10 秒之内，我们所配置的条件符合，则代表页面加载成功，否则则会抛出 TimeoutException 异常。</p>
<p data-nodeid="123851">代码实现如下：</p>
<pre class="lang-python" data-nodeid="123852"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_page</span>(<span class="hljs-params">url,&nbsp;condition,&nbsp;locator</span>):</span>
&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'scraping&nbsp;%s'</span>,&nbsp;url)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;browser.get(url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wait.until(condition(locator))
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">except</span>&nbsp;TimeoutException:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.error(<span class="hljs-string">'error&nbsp;occurred&nbsp;while&nbsp;scraping&nbsp;%s'</span>,&nbsp;url,&nbsp;exc_info=<span class="hljs-literal">True</span>)
<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_index</span>(<span class="hljs-params">page</span>):</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;INDEX_URL.format(page=page)
&nbsp;&nbsp;&nbsp;scrape_page(url,&nbsp;condition=EC.visibility_of_all_elements_located,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;locator=(By.CSS_SELECTOR,&nbsp;<span class="hljs-string">'#index&nbsp;.item'</span>))
</code></pre>
<p data-nodeid="123853">这里我们定义了两个方法。</p>
<p data-nodeid="123854">第一个方法 scrape_page 依然是一个通用的爬取方法，它可以实现任意 URL 的爬取和状态监听以及异常处理，它接收 url、condition、locator 三个参数，其中 url 参数就是要爬取的页面 URL；condition 就是页面加载的判定条件，它可以是 expected_conditions 的其中某一项判定条件，如 visibility_of_all_elements_located、visibility_of_element_located 等等；locator 代表定位器，是一个元组，它可以通过配置查询条件和参数来获取一个或多个节点，如 (By.CSS_SELECTOR, '#index .item') 则代表通过 CSS 选择器查找 #index .item 来获取列表页所有电影信息节点。另外爬取的过程添加了 TimeoutException 检测，如果在规定时间（这里为 10 秒）没有加载出来对应的节点，那就抛出 TimeoutException 异常并输出错误日志。</p>
<p data-nodeid="123855">第二个方法 scrape_index 则是爬取列表页的方法，它接收一个参数 page，通过调用 scrape_page 方法并传入 condition 和 locator 对象，完成页面的爬取。这里 condition 我们用的是 visibility_of_all_elements_located，代表所有的节点都加载出来才算成功。</p>
<p data-nodeid="123856">注意，这里爬取页面我们不需要返回任何结果，因为执行完 scrape_index 后，页面正好处在对应的页面加载完成的状态，我们利用 browser 对象可以进一步进行信息的提取。</p>
<p data-nodeid="123857">好，现在我们已经可以加载出来列表页了，下一步当然就是进行列表页的解析，提取出详情页 URL ，我们定义一个如下的解析列表页的方法：</p>
<pre class="lang-python" data-nodeid="123858"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;urllib.parse&nbsp;<span class="hljs-keyword">import</span>&nbsp;urljoin
<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">parse_index</span>():</span>
&nbsp;&nbsp;&nbsp;elements&nbsp;=&nbsp;browser.find_elements_by_css_selector(<span class="hljs-string">'#index&nbsp;.item&nbsp;.name'</span>)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;element&nbsp;<span class="hljs-keyword">in</span>&nbsp;elements:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;href&nbsp;=&nbsp;element.get_attribute(<span class="hljs-string">'href'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">yield</span>&nbsp;urljoin(INDEX_URL,&nbsp;href)
</code></pre>
<p data-nodeid="123859">这里我们通过 find_elements_by_css_selector 方法直接提取了所有电影的名称，接着遍历结果，通过 get_attribute 方法提取了详情页的 href，再用 urljoin 方法合并成一个完整的 URL。</p>
<p data-nodeid="123860">最后，我们再用一个 main 方法把上面的方法串联起来，实现如下：</p>
<pre class="lang-python" data-nodeid="123861"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;page&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">1</span>,&nbsp;TOTAL_PAGE&nbsp;+&nbsp;<span class="hljs-number">1</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;scrape_index(page)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_urls&nbsp;=&nbsp;parse_index()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'details&nbsp;urls&nbsp;%s'</span>,&nbsp;list(detail_urls))
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">finally</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;browser.close()
</code></pre>
<p data-nodeid="123862">这里我们就是遍历了所有页码，依次爬取了每一页的列表页并提取出来了详情页的 URL。</p>
<p data-nodeid="123863">运行结果如下：</p>
<pre class="lang-python" data-nodeid="129367"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">03</span>:<span class="hljs-number">09</span>,<span class="hljs-number">896</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">03</span>:<span class="hljs-number">13</span>,<span class="hljs-number">724</span>&nbsp;-&nbsp;INFO:&nbsp;details&nbsp;urls&nbsp;[<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx'</span>,
...
<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWI5'</span>,&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIxMA=='</span>]
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">03</span>:<span class="hljs-number">13</span>,<span class="hljs-number">724</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">2</span>
...
</code></pre>





<p data-nodeid="123865">由于输出内容较多，这里省略了部分内容。</p>
<p data-nodeid="123866">观察结果我们可以发现，详情页那一个个不规则的 URL 就成功被我们提取到了！</p>
<h3 data-nodeid="123867">爬取详情页</h3>
<p data-nodeid="123868">好了，既然现在我们已经可以成功拿到详情页的 URL 了，接下来我们就进一步完成详情页的爬取并提取对应的信息吧。</p>
<p data-nodeid="123869">同样的逻辑，详情页我们也可以加一个判定条件，如判断电影名称加载出来了就代表详情页加载成功，同样调用 scrape_page 方法即可，代码实现如下：</p>
<pre class="lang-python" data-nodeid="123870"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">scrape_detail</span>(<span class="hljs-params">url</span>):</span>
&nbsp;&nbsp;&nbsp;scrape_page(url,&nbsp;condition=EC.visibility_of_element_located,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;locator=(By.TAG_NAME,&nbsp;<span class="hljs-string">'h2'</span>))
</code></pre>
<p data-nodeid="123871">这里的判定条件 condition 我们使用的是 visibility_of_element_located，即判断单个元素出现即可，locator 我们传入的是 (By.TAG_NAME, 'h2')，即 h2 这个节点，也就是电影的名称对应的节点，如图所示。</p>
<p data-nodeid="123872"><img src="https://s0.lgstatic.com/i/image3/M01/7E/ED/Cgq2xl6Bvw-AdrCfAAV8yEmeyb4309.png" alt="" data-nodeid="124041"></p>
<p data-nodeid="123873">如果执行了 scrape_detail 方法，没有出现 TimeoutException 的话，页面就加载成功了，接着我们再定义一个解析详情页的方法，来提取出我们想要的信息就可以了，实现如下：</p>
<pre class="lang-python" data-nodeid="123874"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">parse_detail</span>():</span>
&nbsp;&nbsp;&nbsp;url&nbsp;=&nbsp;browser.current_url
&nbsp;&nbsp;&nbsp;name&nbsp;=&nbsp;browser.find_element_by_tag_name(<span class="hljs-string">'h2'</span>).text
&nbsp;&nbsp;&nbsp;categories&nbsp;=&nbsp;[element.text&nbsp;<span class="hljs-keyword">for</span>&nbsp;element&nbsp;<span class="hljs-keyword">in</span>&nbsp;browser.find_elements_by_css_selector(<span class="hljs-string">'.categories&nbsp;button&nbsp;span'</span>)]
&nbsp;&nbsp;&nbsp;cover&nbsp;=&nbsp;browser.find_element_by_css_selector(<span class="hljs-string">'.cover'</span>).get_attribute(<span class="hljs-string">'src'</span>)
&nbsp;&nbsp;&nbsp;score&nbsp;=&nbsp;browser.find_element_by_class_name(<span class="hljs-string">'score'</span>).text
&nbsp;&nbsp;&nbsp;drama&nbsp;=&nbsp;browser.find_element_by_css_selector(<span class="hljs-string">'.drama&nbsp;p'</span>).text
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'url'</span>:&nbsp;url,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'name'</span>:&nbsp;name,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'categories'</span>:&nbsp;categories,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;cover,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'score'</span>:&nbsp;score,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">'drama'</span>:&nbsp;drama
&nbsp;&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="123875">这里我们定义了一个 parse_detail 方法，提取了 URL、名称、类别、封面、分数、简介等内容，提取方式如下：</p>
<ul data-nodeid="123876">
<li data-nodeid="123877">
<p data-nodeid="123878">URL：直接调用 browser 对象的 current_url 属性即可获取当前页面的 URL。</p>
</li>
<li data-nodeid="123879">
<p data-nodeid="123880">名称：通过提取 h2 节点内部的文本即可获取，这里使用了 find_element_by_tag_name 方法并传入 h2，提取到了名称的节点，然后调用 text 属性即提取了节点内部的文本，即电影名称。</p>
</li>
<li data-nodeid="123881">
<p data-nodeid="123882">类别：为了方便，类别我们可以通过 CSS 选择器来提取，其对应的 CSS 选择器为 .categories button span，可以选中多个类别节点，这里我们通过 find_elements_by_css_selector 即可提取 CSS 选择器对应的多个类别节点，然后依次遍历这个结果，调用它的 text 属性获取节点内部文本即可。</p>
</li>
<li data-nodeid="123883">
<p data-nodeid="123884">封面：同样可以使用 CSS 选择器 .cover 直接获取封面对应的节点，但是由于其封面的 URL 对应的是 src 这个属性，所以这里用 get_attribute 方法并传入 src 来提取。</p>
</li>
<li data-nodeid="123885">
<p data-nodeid="123886">分数：分数对应的 CSS 选择器为 .score ，我们可以用上面同样的方式来提取，但是这里我们换了一个方法，叫作 find_element_by_class_name，它可以使用 class 的名称来提取节点，能达到同样的效果，不过这里传入的参数就是 class 的名称 score 而不是 .score 了。提取节点之后，我们再调用 text 属性提取节点文本即可。</p>
</li>
<li data-nodeid="123887">
<p data-nodeid="123888">简介：同样可以使用 CSS 选择器 .drama p 直接获取简介对应的节点，然后调用 text 属性提取文本即可。</p>
</li>
</ul>
<p data-nodeid="123889">最后，我们把结果构造成一个字典返回即可。</p>
<p data-nodeid="123890">接下来，我们在 main 方法中再添加这两个方法的调用，实现如下：</p>
<pre class="lang-python" data-nodeid="123891"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">main</span>():</span>
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;page&nbsp;<span class="hljs-keyword">in</span>&nbsp;range(<span class="hljs-number">1</span>,&nbsp;TOTAL_PAGE&nbsp;+&nbsp;<span class="hljs-number">1</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;scrape_index(page)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_urls&nbsp;=&nbsp;parse_index()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;detail_url&nbsp;<span class="hljs-keyword">in</span>&nbsp;list(detail_urls):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'get&nbsp;detail&nbsp;url&nbsp;%s'</span>,&nbsp;detail_url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;scrape_detail(detail_url)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;detail_data&nbsp;=&nbsp;parse_detail()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;logging.info(<span class="hljs-string">'detail&nbsp;data&nbsp;%s'</span>,&nbsp;detail_data)
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">finally</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;browser.close()
</code></pre>
<p data-nodeid="123892">这样，爬取完列表页之后，我们就可以依次爬取详情页，来提取每部电影的具体信息了。</p>
<pre class="lang-python te-preview-highlight" data-nodeid="133448"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">10</span>,<span class="hljs-number">723</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/page/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">16</span>,<span class="hljs-number">997</span>&nbsp;-&nbsp;INFO:&nbsp;get&nbsp;detail&nbsp;url&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">16</span>,<span class="hljs-number">997</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span>,<span class="hljs-number">289</span>&nbsp;-&nbsp;INFO:&nbsp;detail&nbsp;data&nbsp;{<span class="hljs-string">'url'</span>:&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx'</span>,&nbsp;<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'霸王别姬&nbsp;-&nbsp;Farewell&nbsp;My&nbsp;Concubine'</span>,&nbsp;<span class="hljs-string">'categories'</span>:&nbsp;[<span class="hljs-string">'剧情'</span>,&nbsp;<span class="hljs-string">'爱情'</span>],&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;<span class="hljs-string">'https://p0.meituan.net/movie/ce4da3e03e655b5b88ed31b5cd7896cf62472.jpg@464w_644h_1e_1c'</span>,&nbsp;<span class="hljs-string">'score'</span>:&nbsp;<span class="hljs-string">'9.5'</span>,&nbsp;<span class="hljs-string">'drama'</span>:&nbsp;<span class="hljs-string">'影片借一出《霸王别姬》的京戏，牵扯出三个人之间一段随时代风云变幻的爱恨情仇。段小楼（张丰毅&nbsp;饰）与程蝶衣（张国荣&nbsp;饰）是一对打小一起长大的师兄弟，两人一个演生，一个饰旦，一向配合天衣无缝，尤其一出《霸王别姬》，更是誉满京城，为此，两人约定合演一辈子《霸王别姬》。但两人对戏剧与人生关系的理解有本质不同，段小楼深知戏非人生，程蝶衣则是人戏不分。段小楼在认为该成家立业之时迎娶了名妓菊仙（巩俐&nbsp;饰），致使程蝶衣认定菊仙是可耻的第三者，使段小楼做了叛徒，自此，三人围绕一出《霸王别姬》生出的爱恨情仇战开始随着时代风云的变迁不断升级，终酿成悲剧。'</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span>,<span class="hljs-number">291</span>&nbsp;-&nbsp;INFO:&nbsp;get&nbsp;detail&nbsp;url&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIy
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">19</span>,<span class="hljs-number">291</span>&nbsp;-&nbsp;INFO:&nbsp;scraping&nbsp;https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIy
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-29</span>&nbsp;<span class="hljs-number">12</span>:<span class="hljs-number">24</span>:<span class="hljs-number">21</span>,<span class="hljs-number">524</span>&nbsp;-&nbsp;INFO:&nbsp;detail&nbsp;data&nbsp;{<span class="hljs-string">'url'</span>:&nbsp;<span class="hljs-string">'https://dynamic2.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIy'</span>,&nbsp;<span class="hljs-string">'name'</span>:&nbsp;<span class="hljs-string">'这个杀手不太冷&nbsp;-&nbsp;Léon'</span>,&nbsp;<span class="hljs-string">'categories'</span>:&nbsp;[<span class="hljs-string">'剧情'</span>,&nbsp;<span class="hljs-string">'动作'</span>,&nbsp;<span class="hljs-string">'犯罪'</span>],&nbsp;<span class="hljs-string">'cover'</span>:&nbsp;<span class="hljs-string">'https://p1.meituan.net/movie/6bea9af4524dfbd0b668eaa7e187c3df767253.jpg@464w_644h_1e_1c'</span>,&nbsp;<span class="hljs-string">'score'</span>:&nbsp;<span class="hljs-string">'9.5'</span>,&nbsp;<span class="hljs-string">'drama'</span>:&nbsp;<span class="hljs-string">'里昂（让·雷诺&nbsp;饰）是名孤独的职业杀手，受人雇佣。一天，邻居家小姑娘马蒂尔德（纳塔丽·波特曼&nbsp;饰）敲开他的房门，要求在他那里暂避杀身之祸。原来邻居家的主人是警方缉毒组的眼线，只因贪污了一小包毒品而遭恶警（加里·奥德曼&nbsp;饰）杀害全家的惩罚。马蒂尔德&nbsp;得到里昂的留救，幸免于难，并留在里昂那里。里昂教小女孩使枪，她教里昂法文，两人关系日趋亲密，相处融洽。&nbsp;女孩想着去报仇，反倒被抓，里昂及时赶到，将女孩救回。混杂着哀怨情仇的正邪之战渐次升级，更大的冲突在所难免……'</span>}
...
</code></pre>







<p data-nodeid="123894">这样详情页数据我们也可以提取到了。</p>
<h3 data-nodeid="123895">数据存储</h3>
<p data-nodeid="123896">最后，我们再像之前一样添加一个数据存储的方法，为了方便，这里还是保存为 JSON 文本文件，实现如下：</p>
<pre class="lang-python" data-nodeid="123897"><code data-language="python"><span class="hljs-keyword">from</span>&nbsp;os&nbsp;<span class="hljs-keyword">import</span>&nbsp;makedirs
<span class="hljs-keyword">from</span>&nbsp;os.path&nbsp;<span class="hljs-keyword">import</span>&nbsp;exists
RESULTS_DIR&nbsp;=&nbsp;<span class="hljs-string">'results'</span>
exists(RESULTS_DIR)&nbsp;<span class="hljs-keyword">or</span>&nbsp;makedirs(RESULTS_DIR)
<span class="hljs-function"><span class="hljs-keyword">def</span>&nbsp;<span class="hljs-title">save_data</span>(<span class="hljs-params">data</span>):</span>
&nbsp;&nbsp;&nbsp;name&nbsp;=&nbsp;data.get(<span class="hljs-string">'name'</span>)
&nbsp;&nbsp;&nbsp;data_path&nbsp;=&nbsp;<span class="hljs-string">f'<span class="hljs-subst">{RESULTS_DIR}</span>/<span class="hljs-subst">{name}</span>.json'</span>
&nbsp;&nbsp;&nbsp;json.dump(data,&nbsp;open(data_path,&nbsp;<span class="hljs-string">'w'</span>,&nbsp;encoding=<span class="hljs-string">'utf-8'</span>),&nbsp;ensure_ascii=<span class="hljs-literal">False</span>,&nbsp;indent=<span class="hljs-number">2</span>)
</code></pre>
<p data-nodeid="123898">这里原理和实现方式与 Ajax 爬取实战课时是完全相同的，不再赘述。</p>
<p data-nodeid="123899">最后添加上 save_data 的调用，完整看下运行效果。</p>
<h3 data-nodeid="123900">Headless</h3>
<p data-nodeid="123901">如果觉得爬取过程中弹出浏览器有所干扰，我们可以开启 Chrome 的 Headless 模式，这样爬取过程中便不会再弹出浏览器了，同时爬取速度还有进一步的提升。</p>
<p data-nodeid="123902">只需要做如下修改即可：</p>
<pre class="lang-python" data-nodeid="123903"><code data-language="python">options&nbsp;=&nbsp;webdriver.ChromeOptions()
options.add_argument(<span class="hljs-string">'--headless'</span>)
browser&nbsp;=&nbsp;webdriver.Chrome(options=options)
</code></pre>
<p data-nodeid="123904">这里通过 ChromeOptions 添加了 --headless 参数，然后用 ChromeOptions 来进行 Chrome 的初始化即可。</p>
<p data-nodeid="123905">修改后再重新运行代码，Chrome 浏览器就不会弹出来了，爬取结果是完全一样的。</p>
<h3 data-nodeid="123906">总结</h3>
<p data-nodeid="123907">本课时我们通过一个案例了解了 Selenium 的适用场景，并结合案例使用 Selenium 实现了页面的爬取，从而对 Selenium 的使用有进一步的掌握。</p>
<p data-nodeid="123908" class="">以后我们就知道什么时候可以用 Selenium 以及怎样使用 Selenium 来完成页面的爬取啦。</p>

---

### 精选评论

##### rocketeerli：
> Selenium 实战部分，最后保存文件时，在 windows10 系统中不能直接使用 name 变量创建文件，文件名不合法会导致部分文件无法创建进而不能存取数据。因为部分名字中包含冒号，需要对 name 变量进行冒号替换才行 name.replace(":", "")<br>

##### *鸟：
> 跟着老师写下来了，但是运行后没有结果，不知道哪里出现了问题？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以关注 拉勾教育公众号加入学习群，和老师直接沟通～

##### **林：
> <span style="caret-color: rgba(0, 0, 0, 0.847); color: rgba(0, 0, 0, 0.847); font-family: -apple-system-font; font-size: 12px; text-size-adjust: auto;">老师，有个问题请教下，main方法中的for循环如果把生成器类型的detail_urls又转成列表----list(detail_urls),那为什么不直接让parse_index返回个list呢(现在使用yield），反正最终for循环要的都是个list</span><span style="caret-color: rgba(0, 0, 0, 0.847); font-family: -apple-system-font; font-size: 12px; text-size-adjust: auto;">？</span><br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当然是可以的，确实是可以在那个方法里面把 list 就返回，但这样你得先声明一个 list，然后 append，感觉略麻烦所以我就生成器这么写了。另外我习惯是在生成器的调用的时候外面套用一层 list。能达到同样效果

##### *骥：
> 怎么知道后面那一串的构造方法啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 说的是 wait until 那块吗？可以查下对应的官方文档。

##### **泽：
> selenium实战，firefox浏览器用生成器的写法去获取详情页url就会报element is stale错误，不用生成器可以规避

##### **0552：
> 怎么加学习群

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注拉勾教育公众号 咨询小助手加入学习群

##### *懂：
> 1 可以根据尾部标签判断是否加载好页面<div>2 处理超时的请求</div><div>3 储存爬下来需要的数据</div><div>另外可以开启无浏览器节目模式</div>

