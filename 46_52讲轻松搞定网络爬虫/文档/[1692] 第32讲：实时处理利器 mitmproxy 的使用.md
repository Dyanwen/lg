<p>在上一节课我们讲解了 Charles 的使用，它可以帮助我们抓取 HTTP 和 HTTPS 的数据包，抓到请求之后，我们如果能够分析出接口请求的一些规律，就能轻松通过 Python 脚本来进行改写。可是当请求里面包含一些无规律的参数的时候，可能就束手无策了。本节课我们介绍一个叫作 mitmproxy 的工具，它可以对抓包的结果通过脚本进行实时处理和保存，接下来我们来一起了解下吧。</p>
<h3>介绍</h3>
<p>mitmproxy 是一个支持 HTTP 和 HTTPS 的抓包程序，有类似 Fiddler、Charles 的功能，只不过它是一个控制台的形式操作。</p>
<p>mitmproxy 还有两个关联组件。一个是 mitmdump，它是 mitmproxy 的命令行接口，利用它我们可以对接 Python 脚本，用 Python 实现实时监听后的处理。另一个是 mitmweb，它是一个 Web 程序，通过它我们可以清楚观察 mitmproxy 捕获的请求。</p>
<p>下面我们来了解它们的用法。</p>
<h3>准备工作</h3>
<p>请确保已经正确安装好了 mitmproxy，并且手机和 PC 处于同一个局域网下，同时配置好了 mitmproxy 的 CA 证书，具体的配置可以参考 <a href="https://cuiqingcai.com/5391.html">https://cuiqingcai.com/5391.html</a>。</p>
<h3>mitmproxy 的功能</h3>
<p>mitmproxy 有如下几项功能。</p>
<ul>
<li>拦截 HTTP 和 HTTPS 请求和响应；</li>
<li>保存 HTTP 会话并进行分析；</li>
<li>模拟客户端发起请求，模拟服务端返回响应；</li>
<li>利用反向代理将流量转发给指定的服务器；</li>
<li>支持 Mac 和 Linux 上的透明代理；</li>
<li>利用 Python 对 HTTP 请求和响应进行实时处理。</li>
</ul>
<h3>抓包原理</h3>
<p>和 Charles 一样，mitmproxy 运行于自己的 PC 上，mitmproxy 会在 PC 的 8080 端口运行，然后开启一个代理服务，这个服务实际上是一个 HTTP/HTTPS 的代理。</p>
<p>手机和 PC 在同一个局域网内，设置代理为 mitmproxy 的代理地址，这样手机在访问互联网的时候流量数据包就会流经 mitmproxy，mitmproxy 再去转发这些数据包到真实的服务器，服务器返回数据包时再由 mitmproxy 转发回手机，这样 mitmproxy 就相当于起了中间人的作用，抓取到所有 Request 和 Response，另外这个过程还可以对接 mitmdump，抓取到的 Request 和 Response 的具体内容都可以直接用 Python 来处理，比如得到 Response 之后我们可以直接进行解析，然后存入数据库，这样就完成了数据的解析和存储过程。</p>
<h3>设置代理</h3>
<p>首先，我们需要运行 mitmproxy，mitmproxy 启动命令如下所示：</p>
<pre><code data-language="java" class="lang-java">mitmproxy
</code></pre>
<p>运行之后会在 8080 端口上运行一个代理服务，如图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpNuAD-tAAAFocJlgTJw723.png" alt="image001.png"></p>
<p>右下角会出现当前正在监听的端口。</p>
<p>或者启动 mitmdump，它也会监听 8080 端口，命令如下所示：</p>
<pre><code data-language="java" class="lang-java">mitmdump
</code></pre>
<p>运行结果如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpOWAYBu_AAGjckfB2P0577.png" alt="image003.png"></p>
<p>将手机和 PC 连接在同一局域网下，设置代理为当前代理。首先看看 PC 的当前局域网 IP。</p>
<p>Windows 上的命令如下所示：</p>
<pre><code data-language="java" class="lang-java">ipconfig
</code></pre>
<p>Linux 和 Mac 上的命令如下所示：</p>
<pre><code data-language="java" class="lang-java">ifconfig
</code></pre>
<p>输出结果如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpPmATzjpAAzMMArMk2k217.png" alt="image005.png"></p>
<p>一般类似 10.<em>.</em>.* 或 172.16.<em>.</em> 或 192.168.1.* 这样的 IP 就是当前 PC 的局域网 IP，例如此图中 PC 的 IP 为 192.168.1.28，手机代理设置类似如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpQGADWl1AAQNcZZwAJk640.png" alt="image007.png"></p>
<p>这样我们就配置好了 mitmproxy 的代理。</p>
<h3>mitmproxy 的使用</h3>
<p>确保 mitmproxy 正常运行，并且手机和 PC 处于同一个局域网内，设置了 mitmproxy 的代理。</p>
<p>运行 mitmproxy，命令如下所示：</p>
<pre><code data-language="java" class="lang-java">mitmproxy
</code></pre>
<p>设置成功之后，我们只需要在手机浏览器上访问任意的网页或浏览任意的 App 即可。例如在手机上打开百度，mitmproxy 页面便会呈现出手机上的所有请求，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpQqAO882AAygXA7Arr0930.png" alt="image009.png"></p>
<p>这就相当于之前我们在浏览器开发者工具监听到的浏览器请求，在这里我们借助于 mitmproxy 完成。Charles 完全也可以做到。</p>
<p>这里是刚才手机打开百度页面时的所有请求列表，左下角显示的 2/38 代表一共发生了 38 个请求，当前箭头所指的是第二个请求。</p>
<p>每个请求开头都有一个 GET 或 POST，这是各个请求的请求方式。紧接的是请求的 URL。第二行开头的数字就是请求对应的响应状态码，后面是响应内容的类型，如 text/html 代表网页文档、image/gif 代表图片。再往后是响应体的大小和响应的时间。</p>
<p>当前呈现了所有请求和响应的概览，我们可以通过这个页面观察到所有的请求。</p>
<p>如果想查看某个请求的详情，我们可以敲击回车，进入请求的详情页面，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/43/Ciqc1F7DpRGAed2RAAKUzw9sA88228.png" alt="image011.png"></p>
<p>可以看到 Headers 的详细信息，如 Host、Cookies、User-Agent 等。</p>
<p>最上方是一个 Request、Response、Detail 的列表，当前处在 Request 这个选项上。这时我们再点击 Tab 键，即可查看这个请求对应的响应详情，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/43/Ciqc1F7DpRiAchgEAAt5NRISBNA804.png" alt="image013.png"></p>
<p>最上面是响应头的信息，下拉之后我们可以看到响应体的信息。针对当前请求，响应体就是网页的源代码。</p>
<p>这时再敲击 Tab 键，切换到最后一个选项卡 Detail，即可看到当前请求的详细信息，如服务器的 IP 和端口、HTTP 协议版本、客户端的 IP 和端口等，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/43/Ciqc1F7DpSGAdR2pAA7QQksOayg698.png" alt="image015.png"></p>
<p>mitmproxy 还提供了命令行式的编辑功能，我们可以在此页面中重新编辑请求。敲击 e 键即可进入编辑功能，这时它会询问你要编辑哪部分内容，如 Cookies、Query、URL 等，每个选项的第一个字母会高亮显示。敲击要编辑内容名称的首字母即可进入该内容的编辑页面，如敲击 m 即可编辑请求的方式，敲击 q 即可修改 GET 请求参数 Query。</p>
<p>这时我们敲击 q，进入到编辑 Query 的页面。由于没有任何参数，我们可以敲击 a 来增加一行，然后就可以输入参数对应的 Key 和 Value，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/43/Ciqc1F7DpSqAbCwfAAImV4JXWVc342.png" alt="image017.png"></p>
<p>这里我们输入 Key 为 wd，Value 为 NBA。</p>
<p>然后再敲击 Esc 键和 q 键，返回之前的页面，再敲击 e 和 p 键修改 Path。和上面一样，敲击 a 增加 Path 的内容，这时我们将 Path 修改为 s，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/44/Ciqc1F7DpUeATCP3AAJHi1jBRU0650.png" alt="image019.png"></p>
<p>再敲击 esc 和 q 键返回，这时我们可以看到最上面的请求链接变成了 <a href="https://www.baidu.com/s?wd=NBA">https://www.baidu.com/s?wd=NBA</a>，访问这个页面，可以看到百度搜索 NBA 关键词的搜索结果，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/44/Ciqc1F7DpU-AEg_gAAu2td4jTBo266.png" alt="image021.png"></p>
<p>敲击 a 保存修改，敲击 r 重新发起修改后的请求，即可看到上方请求方式前面多了一个回旋箭头，这说明重新执行了修改后的请求。这时我们再观察响应体内容，即可看到搜索 NBA 的页面结果的源代码，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpVeADS_BAAlkuNQ5gr4764.png" alt="image023.png"></p>
<p>以上内容便是 mitmproxy 的简单用法。利用 mitmproxy，我们可以观察到手机上的所有请求，还可以对请求进行修改并重新发起。</p>
<p>Fiddler、Charles 也有这个功能，而且它们的图形界面操作更加方便。那么 mitmproxy 的优势何在？</p>
<p>mitmproxy 的强大之处体现在它的另一个工具 mitmdump，有了它我们可以直接对接 Python 对请求进行处理。下面我们来看看 mitmdump 的用法。</p>
<h3>mitmdump 的使用</h3>
<p>mitmdump 是 mitmproxy 的命令行接口，同时还可以对接 Python 对请求进行处理，这是相比 Fiddler、Charles 等工具更加方便的地方。有了它我们可以不用手动截获和分析 HTTP 请求和响应，只需写好请求和响应的处理逻辑即可。它还可以实现数据的解析、存储等工作，这些过程都可以通过 Python 实现。</p>
<h4>实例引入</h4>
<p>我们可以使用命令启动 mitmproxy，并把截获的数据保存到文件中，命令如下所示：</p>
<pre><code data-language="java" class="lang-java">mitmdump -w outfile
</code></pre>
<p>其中 outfile 的名称任意，截获的数据都会被保存到此文件中。<br>
还可以指定一个脚本来处理截获的数据，使用 - s 参数即可：</p>
<pre><code data-language="js" class="lang-js">mitmdump -s script.py
</code></pre>
<p>这里指定了当前处理脚本为 script.py，它需要放置在当前命令执行的目录下。<br>
我们可以在脚本里写入如下的代码：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">def <span class="hljs-title">request</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;flow.request.headers['User-Agent'] </span>= <span class="hljs-string">'MitmProxy'</span>
&nbsp; &nbsp;print(flow.request.headers)
</code></pre>
<p>我们定义了一个 request 方法，参数为 flow，它其实是一个 HTTPFlow 对象，通过 request 属性即可获取到当前请求对象。然后打印输出了请求的请求头，将请求头的 User-Agent 修改成了 MitmProxy。<br>
运行之后我们在手机端访问 <a href="http://httpbin.org/get">http://httpbin.org/get</a>，就可以看到有如下情况发生。</p>
<p>手机端的页面显示如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpWWAEXLDAAPpVyfYPeU461.png" alt="image025.png"></p>
<p>PC 端控制台输出如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/4F/CgqCHl7DpW2AQ0xRAAGcdBbVLGE992.png" alt="image027.png"></p>
<p>手机端返回结果的 Headers 实际上就是请求的 Headers，User-Agent 被修改成了 mitmproxy。PC 端控制台输出了修改后的 Headers 内容，其 User-Agent 的内容正是 mitmproxy。</p>
<p>所以，通过这三行代码我们就可以完成对请求的改写。print 方法输出结果可以呈现在 PC 端控制台上，可以方便地进行调试。</p>
<h4>日志输出</h4>
<p>mitmdump 提供了专门的日志输出功能，可以设定不同级别以不同颜色输出结果。我们把脚本修改成如下内容：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">from mitmproxy <span class="hljs-keyword">import</span> ctx
def <span class="hljs-title">request</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;flow.request.headers['User-Agent'] </span>= <span class="hljs-string">'MitmProxy'</span>
&nbsp; &nbsp;ctx.log.info(str(flow.request.headers))
&nbsp; &nbsp;ctx.log.warn(str(flow.request.headers))
&nbsp; &nbsp;ctx.log.error(str(flow.request.headers))
</code></pre>
<p>这里调用了 ctx 模块，它有一个 log 功能，调用不同的输出方法就可以输出不同颜色的结果，以方便我们做调试。例如，info 方法输出的内容是白色的，warn 方法输出的内容是黄色的，error 方法输出的内容是红色的。运行结果如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpXeARkg1AAKHytbwpaA616.png" alt="image029.png"></p>
<p>不同的颜色对应不同级别的输出，我们可以将不同的结果合理划分级别输出，以更直观方便地查看调试信息。</p>
<h4>Request</h4>
<p>最开始我们实现了 request 方法并且对 Headers 进行了修改。下面我们来看看 Request 还有哪些常用的功能。我们先用一个实例来感受一下。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">from mitmproxy <span class="hljs-keyword">import</span> ctx
def <span class="hljs-title">request</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;request </span>= flow.request
&nbsp; &nbsp;info = ctx.log.<span class="hljs-function">info
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(request.url)</span>
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(request.headers)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(request.cookies)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(request.host)</span>
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(request.method)</span>
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(request.port)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(request.scheme)</span>
</span></code></pre>
<p>我们修改脚本，然后在手机上打开百度，即可看到 PC 端控制台输出了一系列的请求，在这里我们找到第一个请求。控制台打印输出了 Request 的一些常见属性，如 URL、Headers、Cookies、Host、Method、Scheme 等。输出结果如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpYCAMUGXAARnPtp_bW0145.png" alt="image031.png"></p>
<p>结果中分别输出了请求链接、请求头、请求 Cookies、请求 Host、请求方法、请求端口、请求协议这些内容。</p>
<p>同时我们还可以对任意属性进行修改，就像最初修改 Headers 一样，直接赋值即可。例如，这里将请求的 URL 修改一下，脚本修改如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">def <span class="hljs-title">request</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;url </span>= <span class="hljs-string">'https://httpbin.org/get'</span>
&nbsp; &nbsp;flow.request.url = url
</code></pre>
<p>手机端得到如下结果，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/44/Ciqc1F7DpYiAIDjmAAD_0LaTzwM375.gif" alt="image033.gif"></p>
<p>比较有意思的是，浏览器最上方还是呈现百度的 URL，但是页面已经变成了 httpbin.org 的页面了。另外，Cookies 明显还是百度的 Cookies。我们只是用简单的脚本就成功把请求修改为其他的站点。通过这种方式修改和伪造请求就变得轻而易举。</p>
<p>通过这个实例我们知道，有时候 URL 虽然是正确的，但是内容并非正确。我们需要进一步提高自己的安全防范意识。</p>
<p>Request 还有很多属性，在此不再一一列举。更多属性可以参考：<a href="http://docs.mitmproxy.org/en/latest/scripting/api.html">http://docs.mitmproxy.org/en/latest/scripting/api.html</a>。</p>
<p>只要我们了解了基本用法，会很容易地获取和修改 Reqeust 的任意内容，比如可以用修改 Cookies、添加代理等方式来规避反爬。</p>
<h4>Response</h4>
<p>对于爬虫来说，我们更加关心的其实是响应的内容，因为 Response Body 才是爬取的结果。对于响应来说，mitmdump 也提供了对应的处理接口，就是 response 方法。下面我们用一个实例感受一下。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">from mitmproxy <span class="hljs-keyword">import</span> ctx
def <span class="hljs-title">response</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;response </span>= flow.response
&nbsp; &nbsp;info = ctx.log.<span class="hljs-function">info
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.status_code)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.headers)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.cookies)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.text)</span>)
</span></code></pre>
<p>将脚本修改为如上内容，然后手机访问：<a href="http://httpbin.org/get%E3%80%82">http://httpbin.org/get。</a></p>
<p>这里打印输出了响应的 status_code、headers、cookies、text 这几个属性，其中最主要的 text 属性就是网页的源代码。</p>
<p>PC 端控制台输出如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpZWAGDXGAACrECqUeiI666.gif" alt="image035.gif"></p>
<p>控制台输出了响应的状态码、响应头、Cookies、响应体这几部分内容。</p>
<p>我们可以通过 response 方法获取每个请求的响应内容。接下来再进行响应的信息提取和存储，我们就可以成功完成爬取了。</p>
<h3>示例操作</h3>
<p>下面我们来介绍一个 App 的爬取实现，示例 App 可以参见附件。</p>
<p>通过 Charles 抓包之后我们可以发现，其接口的 URL 中包含了一个 token 参数，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/44/Ciqc1F7DpZyAQHt2AABslDvyVeM718.gif" alt="image037.gif"></p>
<p>而且这个 token 每次请求都是变化的，我们目前也观察不出它到底是怎么构造的。</p>
<p>那么我们如果想把抓包的这些结果保存下来，应该怎么办呢？显然就不好直接用程序来构造这些请求了，因为 token 的生成逻辑我们无从得知。这时候我们可以采取 mitmdump 实时处理的操作，只要能抓到包，那就可以把抓包的结果实时处理并保存下来。</p>
<p>首先我们写一个 mitmdump 脚本，还是和原来一样，保存为 spider.py 内容如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">from mitmproxy <span class="hljs-keyword">import</span> ctx
def <span class="hljs-title">response</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;response </span>= flow.response
&nbsp; &nbsp;info = ctx.log.<span class="hljs-function">info
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.status_code)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.headers)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.cookies)</span>)
&nbsp; &nbsp;<span class="hljs-title">info</span><span class="hljs-params">(str(response.text)</span>)
</span></code></pre>
<p>然后启动一下脚本，命令如下：</p>
<pre><code data-language="java" class="lang-java">mitmdump -s spider.py
</code></pre>
<p>设置好手机代理为 mitmdump 的地址。<br>
这时候打开手机 App，会看到如下的界面，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpaWAfq73AABteXWJjR0664.gif" alt="image039.gif"></p>
<p>这时候我们再返回控制台，查看下 mitmdump 的输出，就会变成如下内容，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpayAeXJZAAM_VpzRqDs531.gif" alt="image041.gif"></p>
<p>我们可以看到这里就输出来了抓包后的结果，最后的这个内容实际上就是接口的返回结果，我们直接将其输出到控制台上了。</p>
<p>经过一些分析我们可以很轻松得知一次请求就会获取 10 条电影的数据，我们可以对脚本进行下改写，对这个结果进行分析处理。比如我们可以遍历每一条电影数据，然后将其保存到本地，成为 JSON 文件。</p>
<p>脚本就可以改写如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">from mitmproxy <span class="hljs-keyword">import</span> ctx
<span class="hljs-keyword">import</span> json
def <span class="hljs-title">response</span><span class="hljs-params">(flow)</span>:
&nbsp; &nbsp;response </span>= flow.response
&nbsp; &nbsp;<span class="hljs-keyword">if</span> response.status_code != <span class="hljs-number">200</span>:
&nbsp; <span class="hljs-keyword">return</span>
&nbsp; &nbsp;data = json.loads(str(response.text))
&nbsp; &nbsp;<span class="hljs-keyword">for</span> item in data.get(<span class="hljs-string">'results'</span>):
&nbsp; name = item.get(<span class="hljs-string">'name'</span>)
&nbsp; <span class="hljs-function">with <span class="hljs-title">open</span><span class="hljs-params">(f<span class="hljs-string">'{name}.json'</span>, <span class="hljs-string">'w'</span>, encoding=<span class="hljs-string">'utf-8'</span>)</span> as f:
&nbsp; f.<span class="hljs-title">write</span><span class="hljs-params">(json.dumps(item, indent=<span class="hljs-number">2</span>, ensure_ascii=False)</span>)
</span></code></pre>
<p>这里我们首先对 response 的内容进行了解析，然后遍历了 results 字段的每个内容，然后获取其 name 字段当作文件输出的名称，最后输出保存成一个 JSON 文件。</p>
<p>这时候，我们再重新运行 mitmdump 和 App，就可以发现 mitmdump 的运行目录下就出现了好多 JSON 文件，这些 JSON 文件的内容就是抓包的电影数据的结果，如图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/50/CgqCHl7DpbOAYi38AAA7xUxQYRM576.jpg" alt="image043.jpg"></p>
<p>我们打开其中一个结果，内容如下：</p>
<pre><code data-language="java" class="lang-java">{
&nbsp;<span class="hljs-string">"id"</span>: <span class="hljs-number">3</span>,
&nbsp;<span class="hljs-string">"name"</span>: <span class="hljs-string">"肖申克的救赎"</span>,
&nbsp;<span class="hljs-string">"alias"</span>: <span class="hljs-string">"The Shawshank Redemption"</span>,
&nbsp;<span class="hljs-string">"cover"</span>: <span class="hljs-string">"https://p0.meituan.net/movie/283292171619cdfd5b240c8fd093f1eb255670.jpg@464w_644h_1e_1c"</span>,
&nbsp;<span class="hljs-string">"categories"</span>: [
&nbsp; &nbsp;<span class="hljs-string">"剧情"</span>,
&nbsp; &nbsp;<span class="hljs-string">"犯罪"</span>
],
&nbsp;<span class="hljs-string">"published_at"</span>: <span class="hljs-string">"1994-09-10"</span>,
&nbsp;<span class="hljs-string">"minute"</span>: <span class="hljs-number">142</span>,
&nbsp;<span class="hljs-string">"score"</span>: <span class="hljs-number">9.5</span>,
&nbsp;<span class="hljs-string">"regions"</span>: [
&nbsp; &nbsp;<span class="hljs-string">"美国"</span>
]
}
</code></pre>
<p>可以看到这就是我们在 Charles 中看到的返回结果中的一条电影数据的内容，我们通过 mitmdump 对接实时处理脚本实现了分析和保存。</p>
<p>以上便是 mitmproxy 和 mitmdump 的基本用法。</p>

---

### 精选评论

##### Derek：
> 赞！<div><br></div>

