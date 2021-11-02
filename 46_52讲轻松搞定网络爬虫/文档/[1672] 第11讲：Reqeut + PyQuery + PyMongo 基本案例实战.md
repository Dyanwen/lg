<p data-nodeid="55363" class="te-preview-highlight">在前面我们已经学习了多进程、requests、正则表达式、pyquery、PyMongo 等的基本用法，但我们还没有完整地实现一个爬取案例。本课时，我们就来实现一个完整的网站爬虫案例，把前面学习的知识点串联起来，同时加深对这些知识点的理解。</p>
<h3 data-nodeid="55364">准备工作</h3>
<p data-nodeid="55365">在本节课开始之前，我们需要做好如下的准备工作：</p>
<ul data-nodeid="55366">
<li data-nodeid="55367">
<p data-nodeid="55368">安装好 Python3（最低为 3.6 版本），并能成功运行 Python3 程序。</p>
</li>
<li data-nodeid="55369">
<p data-nodeid="55370">了解 Python 多进程的基本原理。</p>
</li>
<li data-nodeid="55371">
<p data-nodeid="55372">了解 Python HTTP 请求库 requests 的基本用法。</p>
</li>
<li data-nodeid="55373">
<p data-nodeid="55374">了解正则表达式的用法和 Python 中正则表达式库 re 的基本用法。</p>
</li>
<li data-nodeid="55375">
<p data-nodeid="55376">了解 Python HTML 解析库 pyquery 的基本用法。</p>
</li>
<li data-nodeid="55377">
<p data-nodeid="55378">了解 MongoDB 并安装和启动 MongoDB 服务。</p>
</li>
<li data-nodeid="55379">
<p data-nodeid="55380">了解 Python 的 MongoDB 操作库 PyMongo 的基本用法。</p>
</li>
</ul>
<p data-nodeid="55381">以上内容在前面的课时中均有讲解，如果你还没有准备好，那么我建议你可以再复习一下这些内容。</p>
<h3 data-nodeid="55382">爬取目标</h3>
<p data-nodeid="56617" class="">这节课我们以一个基本的静态网站作为案例进行爬取，需要爬取的链接为：<a href="https://static1.scrape.center/" data-nodeid="56621">https://static1.scrape.center/</a>，这个网站里面包含了一些电影信息，界面如下：</p>


<p data-nodeid="55384"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/CgpOIF5zioaAJ2pmAAVK9IN6YAk404.png" alt="" data-nodeid="55553"></p>
<p data-nodeid="55385">首页是一个影片列表，每栏里都包含了这部电影的封面、名称、分类、上映时间、评分等内容，同时列表页还支持翻页，点击相应的页码我们就能进入到对应的新列表页。</p>
<p data-nodeid="55386">如果我们点开其中一部电影，会进入电影的详情页面，比如我们点开第一部《霸王别姬》，会得到如下页面：</p>
<p data-nodeid="55387"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/Cgq2xl5zioeATkf_AAZdwA77BpU974.png" alt="" data-nodeid="55557"></p>
<p data-nodeid="55388">这里显示的内容更加丰富、包括剧情简介、导演、演员等信息。</p>
<p data-nodeid="55389">我们这节课要完成的目标是：</p>
<ul data-nodeid="55390">
<li data-nodeid="55391">
<p data-nodeid="55392">用 requests 爬取这个站点每一页的电影列表，顺着列表再爬取每个电影的详情页。</p>
</li>
<li data-nodeid="55393">
<p data-nodeid="55394">用 pyquery 和正则表达式提取每部电影的名称、封面、类别、上映时间、评分、剧情简介等内容。</p>
</li>
<li data-nodeid="55395">
<p data-nodeid="55396">把以上爬取的内容存入 MongoDB 数据库。</p>
</li>
<li data-nodeid="55397">
<p data-nodeid="55398">使用多进程实现爬取的加速。</p>
</li>
</ul>
<p data-nodeid="55399">那么我们现在就开始吧。</p>
<h3 data-nodeid="55400">爬取列表页</h3>
<p data-nodeid="58293" class="">爬取的第一步肯定要从列表页入手，我们首先观察一下列表页的结构和翻页规则。在浏览器中访问&nbsp;<a href="https://static1.scrape.center/" data-nodeid="58297">https://static1.scrape.center/</a>，然后打开浏览器开发者工具，观察每一个电影信息区块对应的 HTML，以及进入到详情页的 URL 是怎样的，如图所示：</p>


<p data-nodeid="55402"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/CgpOIF5zioeAQINcAAPHrIebZSk802.png" alt="" data-nodeid="55572"></p>
<p data-nodeid="55403">可以看到每部电影对应的区块都是一个 div 节点，它的 class 属性都有 el-card 这个值。每个列表页有 10 个这样的 div 节点，也就对应着 10 部电影的信息。</p>
<p data-nodeid="55404">我们再分析下从列表页是怎么进入到详情页的，我们选中电影的名称，看下结果：</p>
<p data-nodeid="55405"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/Cgq2xl5zioiAOdUKAAPy8ODV7Vw336.png" alt="" data-nodeid="55576"></p>
<p data-nodeid="61657" class="">可以看到这个名称实际上是一个 h2 节点，其内部的文字就是电影的标题。h2 节点的外面包含了一个 a 节点，这个 a 节点带有 href 属性，这就是一个超链接，其中 href 的值为 /detail/1，这是一个相对网站的根 URL&nbsp;<a href="https://static1.scrape.center/" data-nodeid="61661">https://static1.scrape.center/</a>&nbsp;路径，加上网站的根 URL 就构成了&nbsp;<a href="https://static1.scrape.center/detail/1" data-nodeid="61665">https://static1.scrape.center/detail/1</a>，也就是这部电影详情页的 URL。这样我们只需要提取这个 href 属性就能构造出详情页的 URL 并接着爬取了。</p>




<p data-nodeid="55407">接下来我们来分析下翻页的逻辑，我们拉到页面的最下方，可以看到分页页码，如图所示：</p>
<p data-nodeid="55408"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/CgpOIF5zioiAc8WPAARcToG65Dw195.png" alt="" data-nodeid="55588"></p>
<p data-nodeid="55409">页面显示一共有 100 条数据，10 页的内容，因此页码最多是 10。接着我们点击第 2 页，如图所示：</p>
<p data-nodeid="55410"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/Cgq2xl5ziomAckUSAAQX_VVjG7U401.png" alt="" data-nodeid="55591"></p>
<p data-nodeid="63337" class="">可以看到网页的 URL 变成了&nbsp;<a href="http://%20https://static1.center/page/2" data-nodeid="63341">https://static1.scrape.center/page/2</a>，相比根 URL 多了 &nbsp;/page/2 &nbsp;这部分内容。网页的结构还是和原来一模一样，所以我们可以和第 1 页一样处理。</p>


<p data-nodeid="55412">接着我们查看第 3 页、第 4 页等内容，可以发现有这么一个规律，每一页的 URL 最后分别变成了 /page/3、/page/4。所以，/page 后面跟的就是列表页的页码，当然第 1 页也是一样，我们在根 URL 后面加上 /page/1 也是能访问的，只不过网站做了一下处理，默认的页码是 1，所以显示第 1 页的内容。</p>
<p data-nodeid="55413">好，分析到这里，逻辑基本就清晰了。</p>
<p data-nodeid="55414">如果我们要完成列表页的爬取，可以这么实现：</p>
<ul data-nodeid="55415">
<li data-nodeid="55416">
<p data-nodeid="55417">遍历页码构造 10 页的索引页 URL。</p>
</li>
<li data-nodeid="55418">
<p data-nodeid="55419">从每个索引页分析提取出每个电影的详情页 URL。</p>
</li>
</ul>
<p data-nodeid="55420">现在我们写代码来实现一下吧。</p>
<p data-nodeid="55421">首先，我们需要先定义一些基础的变量，并引入一些必要的库，写法如下：</p>
<pre class="lang-python" data-nodeid="64175"><code data-language="python"><span class="hljs-keyword">import</span> requests
<span class="hljs-keyword">import</span> logging
<span class="hljs-keyword">import</span> re
<span class="hljs-keyword">import</span> pymongo
<span class="hljs-keyword">from</span> pyquery <span class="hljs-keyword">import</span> PyQuery <span class="hljs-keyword">as</span> pq
<span class="hljs-keyword">from</span> urllib.parse <span class="hljs-keyword">import</span> urljoin

logging.basicConfig(level=logging.INFO,
                    format=<span class="hljs-string">'%(asctime)s - %(levelname)s: %(message)s'</span>)

BASE_URL = <span class="hljs-string">'https://static1.scrape.center'</span>
TOTAL_PAGE = <span class="hljs-number">10</span>
</code></pre>

<p data-nodeid="55423">这里我们引入了 requests 用来爬取页面，logging 用来输出信息，re 用来实现正则表达式解析，pyquery 用来直接解析网页，pymongo 用来实现 MongoDB 存储，urljoin 用来做 URL 的拼接。</p>
<p data-nodeid="55424">接着我们定义日志输出级别和输出格式，完成之后再定义 BASE_URL 为当前站点的根 URL，TOTAL_PAGE 为需要爬取的总页码数量。</p>
<p data-nodeid="55425">定义好了之后，我们来实现一个页面爬取的方法吧，实现如下：</p>
<pre class="lang-python" data-nodeid="55426"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">scrape_page</span>(<span class="hljs-params">url</span>):</span>
    logging.info(<span class="hljs-string">'scraping %s...'</span>, url)
    <span class="hljs-keyword">try</span>:
        response = requests.get(url)
        <span class="hljs-keyword">if</span> response.status_code == <span class="hljs-number">200</span>:
            <span class="hljs-keyword">return</span> response.text
        logging.error(<span class="hljs-string">'get invalid status code %s while scraping %s'</span>, response.status_code, url)
    <span class="hljs-keyword">except</span> requests.RequestException:
        logging.error(<span class="hljs-string">'error occurred while scraping %s'</span>, url, exc_info=<span class="hljs-literal">True</span>)
</code></pre>
<p data-nodeid="55427">考虑到我们不仅要爬取列表页，还要爬取详情页，所以在这里我们定义一个较通用的爬取页面的方法，叫作 scrape_page，它接收一个 url 参数，返回页面的 html 代码。</p>
<p data-nodeid="55428">这里我们首先判断状态码是不是 200，如果是，则直接返回页面的 HTML 代码，如果不是，则会输出错误日志信息。另外，这里实现了 requests 的异常处理，如果出现了爬取异常，则会输出对应的错误日志信息。这时我们将 logging 的 error 方法的 exc_info 参数设置为 True 则可以打印出 Traceback 错误堆栈信息。</p>
<p data-nodeid="55429">好了，有了 scrape_page 方法之后，我们给这个方法传入一个 url，正常情况下它就可以返回页面的 HTML 代码了。</p>
<p data-nodeid="55430">在这个基础上，我们来定义列表页的爬取方法吧，实现如下：</p>
<pre class="lang-python" data-nodeid="55431"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">scrape_index</span>(<span class="hljs-params">page</span>):</span>
    index_url = <span class="hljs-string">f'<span class="hljs-subst">{BASE_URL}</span>/page/<span class="hljs-subst">{page}</span>'</span>
    <span class="hljs-keyword">return</span> scrape_page(index_url)
</code></pre>
<p data-nodeid="55432">方法名称叫作 scrape_index，这个方法会接收一个 page 参数，即列表页的页码，我们在方法里面实现列表页的 URL 拼接，然后调用 scrape_page 方法爬取即可得到列表页的 HTML 代码了。</p>
<p data-nodeid="55433">获取了 HTML 代码后，下一步就是解析列表页，并得到每部电影的详情页的 URL 了，实现如下：</p>
<pre class="lang-python" data-nodeid="55434"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">parse_index</span>(<span class="hljs-params">html</span>):</span>
    doc = pq(html)
    links = doc(<span class="hljs-string">'.el-card .name'</span>)
    <span class="hljs-keyword">for</span> link <span class="hljs-keyword">in</span> links.items():
        href = link.attr(<span class="hljs-string">'href'</span>)
        detail_url = urljoin(BASE_URL, href)
        logging.info(<span class="hljs-string">'get detail url %s'</span>, detail_url)
        <span class="hljs-keyword">yield</span> detail_url
</code></pre>
<p data-nodeid="65850" class="">在这里我们定义了 parse_index 方法，它接收一个 html 参数，即列表页的 HTML 代码。接着我们用 pyquery 新建一个 PyQuery 对象，完成之后再用 .el-card .name 选择器选出来每个电影名称对应的超链接节点。我们遍历这些节点，通过调用 attr 方法并传入 href 获得详情页的 URL 路径，得到的 href 就是我们在上文所说的类似 &nbsp;/detail/1 &nbsp;这样的结果。由于这并不是一个完整的 URL，所以我们需要借助 urljoin 方法把 BASE_URL 和 href 拼接起来，获得详情页的完整 URL，得到的结果就是类似&nbsp;<a href="https://static1.scrape.center/detail/1" data-nodeid="65858">https://static1.scrape.center/detail/1</a>&nbsp;这样完整的 URL 了，最后 yield 返回即可。</p>


<p data-nodeid="55436">这样我们通过调用 parse_index 方法传入列表页的 HTML 代码就可以获得该列表页所有电影的详情页 URL 了。</p>
<p data-nodeid="55437">好，接下来我们把上面的方法串联调用一下，实现如下：</p>
<pre class="lang-python" data-nodeid="55438"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span>():</span>
    <span class="hljs-keyword">for</span> page <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>, TOTAL_PAGE + <span class="hljs-number">1</span>):
        index_html = scrape_index(page)
        detail_urls = parse_index(index_html)
        logging.info(<span class="hljs-string">'detail urls %s'</span>, list(detail_urls))

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
    main()
</code></pre>
<p data-nodeid="55439">这里我们定义了 main 方法来完成上面所有方法的调用，首先使用 range 方法遍历一下页码，得到的 page 是 1~10，接着把 page 变量传给 scrape_index 方法，得到列表页的 HTML，赋值为 index_html 变量。接下来再将 index_html 变量传给 parse_index 方法，得到列表页所有电影的详情页 URL，赋值为 detail_urls，结果是一个生成器，我们调用 list 方法就可以将其输出出来。</p>
<p data-nodeid="55440">好，我们运行一下上面的代码，结果如下：</p>
<pre class="lang-python" data-nodeid="85851"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">50</span>,<span class="hljs-number">505</span> - INFO: scraping https://static1.scrape.center/page/<span class="hljs-number">1.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">949</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">2</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">3</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">4</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">5</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">6</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">7</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">8</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">9</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">950</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">10</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">951</span> - INFO: detail urls [<span class="hljs-string">'https://static1.scrape.center/detail/1'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/2'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/3'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/4'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/5'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/6'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/7'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/8'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/9'</span>, <span class="hljs-string">'https://static1.scrape.center/detail/10'</span>]
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">51</span>,<span class="hljs-number">951</span> - INFO: scraping https://static1.scrape.center/page/<span class="hljs-number">2.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">52</span>,<span class="hljs-number">842</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">11</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">22</span>:<span class="hljs-number">39</span>:<span class="hljs-number">52</span>,<span class="hljs-number">842</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">12</span>
...
</code></pre>
























<p data-nodeid="55442">由于输出内容比较多，这里只贴了一部分。</p>
<p data-nodeid="55443">可以看到，在这个过程中程序首先爬取了第 1 页列表页，然后得到了对应详情页的每个 URL，接着再接着爬第 2 页、第 3 页，一直到第 10 页，依次输出了每一页的详情页 URL。这样，我们就成功获取到所有电影详情页 URL 啦。</p>
<h3 data-nodeid="55444">爬取详情页</h3>
<p data-nodeid="55445">现在我们已经成功获取所有详情页 URL 了，那么下一步当然就是解析详情页并提取出我们想要的信息了。</p>
<p data-nodeid="55446">我们首先观察一下详情页的 HTML 代码吧，如图所示：</p>
<p data-nodeid="55447"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/Cgq2xl5zioqAFeI4AAXOH43947I062.png" alt="" data-nodeid="55660"></p>
<p data-nodeid="55448">经过分析，我们想要提取的内容和对应的节点信息如下：</p>
<ul data-nodeid="55449">
<li data-nodeid="55450">
<p data-nodeid="55451">封面：是一个 img 节点，其 class 属性为 cover。</p>
</li>
<li data-nodeid="55452">
<p data-nodeid="55453">名称：是一个 h2 节点，其内容便是名称。</p>
</li>
<li data-nodeid="55454">
<p data-nodeid="55455">类别：是 span 节点，其内容便是类别内容，其外侧是 button 节点，再外侧则是 class 为 categories 的 div 节点。</p>
</li>
<li data-nodeid="55456">
<p data-nodeid="55457">上映时间：是 span 节点，其内容包含了上映时间，其外侧是包含了 class 为 info 的 div 节点。但注意这个 div 前面还有一个 class 为 info 的 div 节点，我们可以使用其内容来区分，也可以使用 nth-child 或 nth-of-type 这样的选择器来区分。另外提取结果中还多了「上映」二字，我们可以用正则表达式把日期提取出来。</p>
</li>
<li data-nodeid="55458">
<p data-nodeid="55459">评分：是一个 p 节点，其内容便是评分，p 节点的 class 属性为 score。</p>
</li>
<li data-nodeid="55460">
<p data-nodeid="55461">剧情简介：是一个 p 节点，其内容便是剧情简介，其外侧是 class 为 drama 的 div 节点。</p>
</li>
</ul>
<p data-nodeid="55462">看上去有点复杂，但是不用担心，有了 pyquery 和正则表达式，我们可以轻松搞定。</p>
<p data-nodeid="55463">接着我们来实现一下代码吧。</p>
<p data-nodeid="55464">刚才我们已经成功获取了详情页的 URL，接下来我们要定义一个详情页的爬取方法，实现如下：</p>
<pre class="lang-python" data-nodeid="55465"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">scrape_detail</span>(<span class="hljs-params">url</span>):</span>
    <span class="hljs-keyword">return</span> scrape_page(url)
</code></pre>
<p data-nodeid="55466">这里定义了一个 scrape_detail 方法，它接收一个 url 参数，并通过调用 scrape_page 方法获得网页源代码。由于我们刚才已经实现了 scrape_page 方法，所以在这里我们不用再写一遍页面爬取的逻辑了，直接调用即可，这就做到了代码复用。</p>
<p data-nodeid="55467">另外你可能会问，这个 scrape_detail 方法里面只调用了 scrape_page 方法，没有别的功能，那爬取详情页直接用 scrape_page 方法不就好了，还有必要再单独定义 scrape_detail 方法吗？</p>
<p data-nodeid="55468">答案是有必要，单独定义一个 scrape_detail 方法在逻辑上会显得更清晰，而且以后如果我们想要对 scrape_detail 方法进行改动，比如添加日志输出或是增加预处理，都可以在 scrape_detail 里面实现，而不用改动 scrape_page 方法，灵活性会更好。</p>
<p data-nodeid="55469">好了，详情页的爬取方法已经实现了，接着就是详情页的解析了，实现如下：</p>
<pre class="lang-python" data-nodeid="55470"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">parse_detail</span>(<span class="hljs-params">html</span>):</span>
    doc = pq(html)
    cover = doc(<span class="hljs-string">'img.cover'</span>).attr(<span class="hljs-string">'src'</span>)
    name = doc(<span class="hljs-string">'a &gt; h2'</span>).text()
    categories = [item.text() <span class="hljs-keyword">for</span> item <span class="hljs-keyword">in</span> doc(<span class="hljs-string">'.categories button span'</span>).items()]
    published_at = doc(<span class="hljs-string">'.info:contains(上映)'</span>).text()
    published_at = re.search(<span class="hljs-string">'(\d{4}-\d{2}-\d{2})'</span>, published_at).group(<span class="hljs-number">1</span>) \
        <span class="hljs-keyword">if</span> published_at <span class="hljs-keyword">and</span> re.search(<span class="hljs-string">'\d{4}-\d{2}-\d{2}'</span>, published_at) <span class="hljs-keyword">else</span> <span class="hljs-literal">None</span>
    drama = doc(<span class="hljs-string">'.drama p'</span>).text()
    score = doc(<span class="hljs-string">'p.score'</span>).text()
    score = float(score) <span class="hljs-keyword">if</span> score <span class="hljs-keyword">else</span> <span class="hljs-literal">None</span>
    <span class="hljs-keyword">return</span> {
        <span class="hljs-string">'cover'</span>: cover,
        <span class="hljs-string">'name'</span>: name,
        <span class="hljs-string">'categories'</span>: categories,
        <span class="hljs-string">'published_at'</span>: published_at,
        <span class="hljs-string">'drama'</span>: drama,
        <span class="hljs-string">'score'</span>: score
    }
</code></pre>
<p data-nodeid="55471">这里我们定义了 parse_detail 方法用于解析详情页，它接收一个 html 参数，解析其中的内容，并以字典的形式返回结果。每个字段的解析情况如下所述：</p>
<ul data-nodeid="55472">
<li data-nodeid="55473">
<p data-nodeid="55474">cover：封面，直接选取 class 为 cover 的 img 节点，并调用 attr 方法获取 src 属性的内容即可。</p>
</li>
<li data-nodeid="55475">
<p data-nodeid="55476">name：名称，直接选取 a 节点的直接子节点 h2 节点，并调用 text 方法提取其文本内容即可得到名称。</p>
</li>
<li data-nodeid="55477">
<p data-nodeid="55478">categories：类别，由于类别是多个，所以这里首先用 .categories button span 选取了 class 为 categories 的节点内部的 span 节点，其结果是多个，所以这里进行了遍历，取出了每个 span 节点的文本内容，得到的便是列表形式的类别。</p>
</li>
<li data-nodeid="55479">
<p data-nodeid="55480">published_at：上映时间，由于 pyquery 支持使用 :contains 直接指定包含的文本内容并进行提取，且每个上映时间信息都包含了「上映」二字，所以我们这里就直接使用 :contains(上映) 提取了 class 为 info 的 div 节点。提取之后，得到的结果类似「1993-07-26 上映」这样，但我们并不想要「上映」这两个字，所以我们又调用了正则表达式把日期单独提取出来了。当然这里也可以直接使用 strip 或 replace 方法把多余的文字去掉，但我们为了练习正则表达式的用法，使用了正则表达式来提取。</p>
</li>
<li data-nodeid="55481">
<p data-nodeid="55482">drama：直接提取 class 为 drama 的节点内部的 p 节点的文本即可。</p>
</li>
<li data-nodeid="55483">
<p data-nodeid="55484">score：直接提取 class 为 score 的 p 节点的文本即可，但由于提取结果是字符串，所以我们需要把它转成浮点数，即 float 类型。</p>
</li>
</ul>
<p data-nodeid="55485">上述字段提取完毕之后，构造一个字典返回即可。</p>
<p data-nodeid="55486">这样，我们就成功完成了详情页的提取和分析了。</p>
<p data-nodeid="55487">最后，我们将 main 方法稍微改写一下，增加这两个方法的调用，改写如下：</p>
<pre class="lang-python" data-nodeid="55488"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span>():</span>
    <span class="hljs-keyword">for</span> page <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>, TOTAL_PAGE + <span class="hljs-number">1</span>):
        index_html = scrape_index(page)
        detail_urls = parse_index(index_html)
        <span class="hljs-keyword">for</span> detail_url <span class="hljs-keyword">in</span> detail_urls:
            detail_html = scrape_detail(detail_url)
            data = parse_detail(detail_html)
            logging.info(<span class="hljs-string">'get detail data %s'</span>, data)
</code></pre>
<p data-nodeid="55489">这里我们首先遍历了 detail_urls，获取了每个详情页的 URL，然后依次调用了 scrape_detail 和 parse_detail 方法，最后得到了每个详情页的提取结果，赋值为 data 并输出。</p>
<p data-nodeid="55490">运行结果如下：</p>
<pre class="lang-python" data-nodeid="90849"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">35</span>,<span class="hljs-number">936</span> - INFO: scraping https://static1.scrape.center/page/<span class="hljs-number">1.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">36</span>,<span class="hljs-number">833</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">36</span>,<span class="hljs-number">833</span> - INFO: scraping https://static1.scrape.center/detail/<span class="hljs-number">1.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">39</span>,<span class="hljs-number">985</span> - INFO: get detail data {<span class="hljs-string">'cover'</span>: <span class="hljs-string">'https://p0.meituan.net/movie/ce4da3e03e655b5b88ed31b5cd7896cf62472.jpg@464w_644h_1e_1c'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'霸王别姬 - Farewell My Concubine'</span>, <span class="hljs-string">'categories'</span>: [<span class="hljs-string">'剧情'</span>, <span class="hljs-string">'爱情'</span>], <span class="hljs-string">'published_at'</span>: <span class="hljs-string">'1993-07-26'</span>, <span class="hljs-string">'drama'</span>: <span class="hljs-string">'影片借一出《霸王别姬》的京戏，牵扯出三个人之间一段随时代风云变幻的爱恨情仇。段小楼（张丰毅 饰）与程蝶衣（张国荣 饰）是一对打小一起长大的师兄弟，两人一个演生，一个饰旦，一向配合天衣无缝，尤其一出《霸王别姬》，更是誉满京城，为此，两人约定合演一辈子《霸王别姬》。但两人对戏剧与人生关系的理解有本质不同，段小楼深知戏非人生，程蝶衣则是人戏不分。段小楼在认为该成家立业之时迎娶了名妓菊仙（巩俐 饰），致使程蝶衣认定菊仙是可耻的第三者，使段小楼做了叛徒，自此，三人围绕一出《霸王别姬》生出的爱恨情仇战开始随着时代风云的变迁不断升级，终酿成悲剧。'</span>, <span class="hljs-string">'score'</span>: <span class="hljs-number">9.5</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">39</span>,<span class="hljs-number">985</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">2</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">39</span>,<span class="hljs-number">985</span> - INFO: scraping https://static1.scrape.center/detail/<span class="hljs-number">2.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">41</span>,<span class="hljs-number">061</span> - INFO: get detail data {<span class="hljs-string">'cover'</span>: <span class="hljs-string">'https://p1.meituan.net/movie/6bea9af4524dfbd0b668eaa7e187c3df767253.jpg@464w_644h_1e_1c'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'这个杀手不太冷 - Léon'</span>, <span class="hljs-string">'categories'</span>: [<span class="hljs-string">'剧情'</span>, <span class="hljs-string">'动作'</span>, <span class="hljs-string">'犯罪'</span>], <span class="hljs-string">'published_at'</span>: <span class="hljs-string">'1994-09-14'</span>, <span class="hljs-string">'drama'</span>: <span class="hljs-string">'里昂（让·雷诺 饰）是名孤独的职业杀手，受人雇佣。一天，邻居家小姑娘马蒂尔德（纳塔丽·波特曼 饰）敲开他的房门，要求在他那里暂避杀身之祸。原来邻居家的主人是警方缉毒组的眼线，只因贪污了一小包毒品而遭恶警（加里·奥德曼 饰）杀害全家的惩罚。马蒂尔德 得到里昂的留救，幸免于难，并留在里昂那里。里昂教小女孩使枪，她教里昂法文，两人关系日趋亲密，相处融洽。 女孩想着去报仇，反倒被抓，里昂及时赶到，将女孩救回。混杂着哀怨情仇的正邪之战渐次升级，更大的冲突在所难免……'</span>, <span class="hljs-string">'score'</span>: <span class="hljs-number">9.5</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-08</span> <span class="hljs-number">23</span>:<span class="hljs-number">37</span>:<span class="hljs-number">41</span>,<span class="hljs-number">062</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">3</span>
...
</code></pre>






<p data-nodeid="55492">由于内容较多，这里省略了后续内容。</p>
<p data-nodeid="55493">可以看到，我们已经成功提取出每部电影的基本信息，包括封面、名称、类别，等等。</p>
<h3 data-nodeid="55494">保存到 MongoDB</h3>
<p data-nodeid="55495">成功提取到详情页信息之后，下一步我们就要把数据保存起来了。在上一课时我们学习了 MongoDB 的相关操作，接下来我们就把数据保存到 MongoDB 吧。</p>
<p data-nodeid="55496">在这之前，请确保现在有一个可以正常连接和使用的 MongoDB 数据库。</p>
<p data-nodeid="55497">将数据导入 MongoDB 需要用到 PyMongo 这个库，这个在最开始已经引入过了。那么接下来我们定义一下 MongoDB 的连接配置，实现如下：</p>
<pre class="lang-python" data-nodeid="55498"><code data-language="python">MONGO_CONNECTION_STRING = <span class="hljs-string">'mongodb://localhost:27017'</span>
MONGO_DB_NAME = <span class="hljs-string">'movies'</span>
MONGO_COLLECTION_NAME = <span class="hljs-string">'movies'</span>

client = pymongo.MongoClient(MONGO_CONNECTION_STRING)
db = client[<span class="hljs-string">'movies'</span>]
collection = db[<span class="hljs-string">'movies'</span>]
</code></pre>
<p data-nodeid="55499">在这里我们声明了几个变量，介绍如下：</p>
<ul data-nodeid="55500">
<li data-nodeid="55501">
<p data-nodeid="55502">MONGO_CONNECTION_STRING：MongoDB 的连接字符串，里面定义了 MongoDB 的基本连接信息，如 host、port，还可以定义用户名密码等内容。</p>
</li>
<li data-nodeid="55503">
<p data-nodeid="55504">MONGO_DB_NAME：MongoDB 数据库的名称。</p>
</li>
<li data-nodeid="55505">
<p data-nodeid="55506">MONGO_COLLECTION_NAME：MongoDB 的集合名称。</p>
</li>
</ul>
<p data-nodeid="55507">这里我们用 MongoClient 声明了一个连接对象，然后依次声明了存储的数据库和集合。</p>
<p data-nodeid="55508">接下来，我们再实现一个将数据保存到 MongoDB 的方法，实现如下：</p>
<pre class="lang-python" data-nodeid="55509"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">save_data</span>(<span class="hljs-params">data</span>):</span>
    collection.update_one({
        <span class="hljs-string">'name'</span>: data.get(<span class="hljs-string">'name'</span>)
    }, {
        <span class="hljs-string">'$set'</span>: data
    }, upsert=<span class="hljs-literal">True</span>)
</code></pre>
<p data-nodeid="55510">在这里我们声明了一个 save_data 方法，它接收一个 data 参数，也就是我们刚才提取的电影详情信息。在方法里面，我们调用了 update_one 方法，第 1 个参数是查询条件，即根据 name 进行查询；第 2 个参数是 data 对象本身，也就是所有的数据，这里我们用 $set 操作符表示更新操作；第 3 个参数很关键，这里实际上是 upsert 参数，如果把这个设置为 True，则可以做到存在即更新，不存在即插入的功能，更新会根据第一个参数设置的 name 字段，所以这样可以防止数据库中出现同名的电影数据。</p>
<blockquote data-nodeid="55511">
<p data-nodeid="55512">注：实际上电影可能有同名，但该场景下的爬取数据没有同名情况，当然这里更重要的是实现 MongoDB 的去重操作。</p>
</blockquote>
<p data-nodeid="55513">好的，那么接下来我们将 main 方法稍微改写一下就好了，改写如下：</p>
<pre class="lang-python" data-nodeid="55514"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span>():</span>
    <span class="hljs-keyword">for</span> page <span class="hljs-keyword">in</span> range(<span class="hljs-number">1</span>, TOTAL_PAGE + <span class="hljs-number">1</span>):
        index_html = scrape_index(page)
        detail_urls = parse_index(index_html)
        <span class="hljs-keyword">for</span> detail_url <span class="hljs-keyword">in</span> detail_urls:
            detail_html = scrape_detail(detail_url)
            data = parse_detail(detail_html)
            logging.info(<span class="hljs-string">'get detail data %s'</span>, data)
            logging.info(<span class="hljs-string">'saving data to mongodb'</span>)
            save_data(data)
            logging.info(<span class="hljs-string">'data saved successfully'</span>)
</code></pre>
<p data-nodeid="55515">这里增加了 save_data 方法的调用，并加了一些日志信息。</p>
<p data-nodeid="55516">重新运行，我们看下输出结果：</p>
<pre class="lang-python" data-nodeid="95014"><code data-language="python"><span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">27</span>,<span class="hljs-number">094</span> - INFO: scraping https://static1.scrape.center/page/<span class="hljs-number">1.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">28</span>,<span class="hljs-number">019</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">1</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">28</span>,<span class="hljs-number">019</span> - INFO: scraping https://static1.scrape.center/detail/<span class="hljs-number">1.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">29</span>,<span class="hljs-number">183</span> - INFO: get detail data {<span class="hljs-string">'cover'</span>: <span class="hljs-string">'https://p0.meituan.net/movie/ce4da3e03e655b5b88ed31b5cd7896cf62472.jpg@464w_644h_1e_1c'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'霸王别姬 - Farewell My Concubine'</span>, <span class="hljs-string">'categories'</span>: [<span class="hljs-string">'剧情'</span>, <span class="hljs-string">'爱情'</span>], <span class="hljs-string">'published_at'</span>: <span class="hljs-string">'1993-07-26'</span>, <span class="hljs-string">'drama'</span>: <span class="hljs-string">'影片借一出《霸王别姬》的京戏，牵扯出三个人之间一段随时代风云变幻的爱恨情仇。段小楼（张丰毅 饰）与程蝶衣（张国荣 饰）是一对打小一起长大的师兄弟，两人一个演生，一个饰旦，一向配合天衣无缝，尤其一出《霸王别姬》，更是誉满京城，为此，两人约定合演一辈子《霸王别姬》。但两人对戏剧与人生关系的理解有本质不同，段小楼深知戏非人生，程蝶衣则是人戏不分。段小楼在认为该成家立业之时迎娶了名妓菊仙（巩俐 饰），致使程蝶衣认定菊仙是可耻的第三者，使段小楼做了叛徒，自此，三人围绕一出《霸王别姬》生出的爱恨情仇战开始随着时代风云的变迁不断升级，终酿成悲剧。'</span>, <span class="hljs-string">'score'</span>: <span class="hljs-number">9.5</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">29</span>,<span class="hljs-number">183</span> - INFO: saving data to mongodb
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">29</span>,<span class="hljs-number">288</span> - INFO: data saved successfully
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">29</span>,<span class="hljs-number">288</span> - INFO: get detail url https://static1.scrape.center/detail/<span class="hljs-number">2</span>
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">29</span>,<span class="hljs-number">288</span> - INFO: scraping https://static1.scrape.center/detail/<span class="hljs-number">2.</span>..
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">30</span>,<span class="hljs-number">250</span> - INFO: get detail data {<span class="hljs-string">'cover'</span>: <span class="hljs-string">'https://p1.meituan.net/movie/6bea9af4524dfbd0b668eaa7e187c3df767253.jpg@464w_644h_1e_1c'</span>, <span class="hljs-string">'name'</span>: <span class="hljs-string">'这个杀手不太冷 - Léon'</span>, <span class="hljs-string">'categories'</span>: [<span class="hljs-string">'剧情'</span>, <span class="hljs-string">'动作'</span>, <span class="hljs-string">'犯罪'</span>], <span class="hljs-string">'published_at'</span>: <span class="hljs-string">'1994-09-14'</span>, <span class="hljs-string">'drama'</span>: <span class="hljs-string">'里昂（让·雷诺 饰）是名孤独的职业杀手，受人雇佣。一天，邻居家小姑娘马蒂尔德（纳塔丽·波特曼 饰）敲开他的房门，要求在他那里暂避杀身之祸。原来邻居家的主人是警方缉毒组的眼线，只因贪污了一小包毒品而遭恶警（加里·奥德曼 饰）杀害全家的惩罚。马蒂尔德 得到里昂的留救，幸免于难，并留在里昂那里。里昂教小女孩使枪，她教里昂法文，两人关系日趋亲密，相处融洽。 女孩想着去报仇，反倒被抓，里昂及时赶到，将女孩救回。混杂着哀怨情仇的正邪之战渐次升级，更大的冲突在所难免……'</span>, <span class="hljs-string">'score'</span>: <span class="hljs-number">9.5</span>}
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">30</span>,<span class="hljs-number">250</span> - INFO: saving data to mongodb
<span class="hljs-number">2020</span><span class="hljs-number">-03</span><span class="hljs-number">-09</span> <span class="hljs-number">01</span>:<span class="hljs-number">10</span>:<span class="hljs-number">30</span>,<span class="hljs-number">253</span> - INFO: data saved successfully
...
</code></pre>





<p data-nodeid="55518">在运行结果中我们可以发现，这里输出了存储 MongoDB 成功的信息。</p>
<p data-nodeid="55519">运行完毕之后我们可以使用 MongoDB 客户端工具（例如 Robo 3T ）可视化地查看已经爬取到的数据，结果如下：</p>
<p data-nodeid="55520"><img src="https://s0.lgstatic.com/i/image3/M01/78/4E/CgpOIF5zio2ANIa3AAK44uJ0lis128.png" alt="" data-nodeid="55757"></p>
<p data-nodeid="55521">这样，所有的电影就被我们成功爬取下来啦！不多不少，正好 100 条。</p>
<h3 data-nodeid="55522">多进程加速</h3>
<p data-nodeid="55523">由于整个的爬取是单进程的，而且只能逐条爬取，速度稍微有点慢，有没有方法来对整个爬取过程进行加速呢？</p>
<p data-nodeid="55524">在前面我们讲了多进程的基本原理和使用方法，下面我们就来实践一下多进程的爬取吧。</p>
<p data-nodeid="55525">由于一共有 10 页详情页，并且这 10 页内容是互不干扰的，所以我们可以一页开一个进程来爬取。由于这 10 个列表页页码正好可以提前构造成一个列表，所以我们可以选用多进程里面的进程池 Pool 来实现这个过程。</p>
<p data-nodeid="55526">这里我们需要改写下 main 方法的调用，实现如下：</p>
<pre class="lang-python" data-nodeid="55527"><code data-language="python"><span class="hljs-keyword">import</span> multiprocessing

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span>(<span class="hljs-params">page</span>):</span>
    index_html = scrape_index(page)
    detail_urls = parse_index(index_html)
    <span class="hljs-keyword">for</span> detail_url <span class="hljs-keyword">in</span> detail_urls:
        detail_html = scrape_detail(detail_url)
        data = parse_detail(detail_html)
        logging.info(<span class="hljs-string">'get detail data %s'</span>, data)
        logging.info(<span class="hljs-string">'saving data to mongodb'</span>)
        save_data(data)
        logging.info(<span class="hljs-string">'data saved successfully'</span>)

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">'__main__'</span>:
    pool = multiprocessing.Pool()
    pages = range(<span class="hljs-number">1</span>, TOTAL_PAGE + <span class="hljs-number">1</span>)
    pool.map(main, pages)
    pool.close()
    pool.join()
</code></pre>
<p data-nodeid="55528">这里我们首先给 main 方法添加一个参数 page，用以表示列表页的页码。接着我们声明了一个进程池，并声明 pages 为所有需要遍历的页码，即 1~10。最后调用 map 方法，第 1 个参数就是需要被调用的方法，第 2 个参数就是 pages，即需要遍历的页码。</p>
<p data-nodeid="55529">这样 pages 就会被依次遍历。把 1~10 这 10 个页码分别传递给 main 方法，并把每次的调用变成一个进程，加入到进程池中执行，进程池会根据当前运行环境来决定运行多少进程。比如我的机器的 CPU 有 8 个核，那么进程池的大小会默认设定为 8，这样就会同时有 8 个进程并行执行。</p>
<p data-nodeid="55530">运行输出结果和之前类似，但是可以明显看到加了多进程执行之后，爬取速度快了非常多。我们可以清空一下之前的 MongoDB 数据，可以发现数据依然可以被正常保存到 MongoDB 数据库中。</p>
<h3 data-nodeid="55531">总结</h3>
<p data-nodeid="55532">到现在为止，我们就完成了全站电影数据的爬取并实现了存储和优化。</p>
<p data-nodeid="55533">这节课我们用到的库有 requests、pyquery、PyMongo、multiprocessing、re、logging 等，通过这个案例实战，我们把前面学习到的知识都串联了起来，其中的一些实现方法可以好好思考和体会，也希望这个案例能够让你对爬虫的实现有更实际的了解。</p>
<p data-nodeid="55534" class="">本节代码：<a href="https://github.com/Python3WebSpider/ScrapeStatic1" data-nodeid="55777">https://github.com/Python3WebSpider/ScrapeStatic1</a>。</p>

---

### 精选评论

##### mfm：
> Mac跑完这个代码咋崩溃了

##### *月：
> 跟着敲了一遍，可能是css基础有些差，有些地方还要补充一下，其他讲的都很清楚，整个过程一气呵成，收益良多。

##### **用户3829：
> 打卡，获益良多，感谢催庆才老师，希望能跟上你们的课程进度！

##### **飞：
> 用了多线程和单线程，时间感觉还是单线程用的时间少，计时的话，多线程的话跑差不多是单线程的2倍多的时间，这是怎么回事老师<div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个看看是不是用的方式不对啊？比如你可以把最后的 main 方法作为入口点来并发，或者检查下 join 等是不是用的有问题呢？

##### *栋：
> 这节课价值很大，作为一个不熟悉python，也对java等开发语言不懂，将这个实战消化理解，我&nbsp; 认为顺便依样画葫芦可以去练手爬一爬类似网站。以便于更加理解深入。。

##### harrison sun：
> <pre style="background-color:#2b2b2b;color:#a9b7c6;font-family:'宋体';font-size:11.3pt;"><span style="color:#cc7832;">def </span><span style="color:#ffc66d;">scrape_detail</span>(url):<br>    <span style="color:#cc7832;">return </span>scrape_page(url)</pre><pre style="background-color:#2b2b2b;color:#a9b7c6;font-family:'宋体';font-size:11.3pt;">请问老师，这一行代码的作用是什么，不写直接调用scrape_page(url)效果一样吧</pre>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个文中已经有讲了，降低耦合度，做到代码复用，意义上更清晰，而且方便扩展，具体原因在文中也有讲解。

