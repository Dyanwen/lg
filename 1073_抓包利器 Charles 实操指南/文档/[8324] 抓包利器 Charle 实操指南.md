<h4 data-nodeid="1833" class=""><span style="color:#f4882d"><span style="color:#406bf6">本课目录</span></span><span style="color:#f6af40"><strong data-nodeid="1985">（思路导航，学习不迷茫）</strong></span></h4>
<ol data-nodeid="1834">
<li data-nodeid="1835">
<p data-nodeid="1836">Charles 工作原理</p>
</li>
<li data-nodeid="1837">
<p data-nodeid="1838">搭建基础代理环境（Web 端与 App 端）</p>
</li>
<li data-nodeid="1839">
<p data-nodeid="1840">接口过滤、拦截和修改<br>
（过滤接口请求与动态修改请求数据）</p>
</li>
<li data-nodeid="1841">
<p data-nodeid="1842">远程映射</p>
</li>
<li data-nodeid="1843">
<p data-nodeid="1844">慢网络模拟</p>
</li>
<li data-nodeid="1845">
<p data-nodeid="1846">微服务分支测试</p>
</li>
<li data-nodeid="1847">
<p data-nodeid="1848">简单接口并发测试</p>
</li>
<li data-nodeid="1849">
<p data-nodeid="1850">Charles 应用常见问题</p>
</li>
<li data-nodeid="1851">
<p data-nodeid="1852">小结</p>
</li>
</ol>
<h4 data-nodeid="1853"><span style="color:#f4882d"><span style="color:#406bf6">本课核心图</span></span><span style="color:#f6af40"><strong data-nodeid="2007">（脑图启示，知识结构化）</strong></span></h4>
<p data-nodeid="1854"><img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0DqAIU_4AAEWQezKFMI897.png" alt="image.png" data-nodeid="2010"></p>
<p data-nodeid="1855">你好，我是蔡超，今天我来分享一下手工测试中如何玩转抓包神器 Charles。</p>
<blockquote data-nodeid="1856">
<p data-nodeid="1857">下载地址：<a href="https://www.charlesproxy.com" data-nodeid="2015">https://www.charlesproxy.com</a></p>
</blockquote>
<p data-nodeid="1858">在日常手工测试过程中，特别是 Web 端测试或者 App 端测试中。我们除了通过页面操作验证功能是否正确外，还需要验证接口的返回是否正确。有时候为了测试某些特殊的测试场景，还不得不去更改服务端的返回值，这部分操作就必须用到测试工具。</p>
<p data-nodeid="1859">加上当前大部分软件架构都是前后端分离的，测试人员在测试时发现问题后，常常面临要把这个 Bug 指派给谁的问题。如果不经分析，随便指派人，也会招致开发的不满。</p>
<p data-nodeid="1860">所以，掌握网络抓包工具，对发现的功能问题进行简单的归因判断，就变得比较重要。</p>
<p data-nodeid="1861"><strong data-nodeid="2022">【为什么要学习本课？】</strong></p>
<p data-nodeid="1862">虽然很多同学都在日常工作中有 Charles 的使用经验，网络上关于 Charles 的介绍也汗牛充栋。但大都是零零散散的知识点，把这些知识点一个个串起来就要花费不少时间；如果遇见已经失效的总结，更是会白白浪费时间。</p>
<p data-nodeid="1863">故今天我们就来系统地重温下 Charles 的重点功能使用方法，本篇教程基本覆盖了 Charles 日常使用的方方面面，大概需要 15 分钟左右，你就能掌握 Charles 的重点用法。</p>
<p data-nodeid="1864"><strong data-nodeid="2028">对于刚入门软件测试的新人、或者刚接触后端测试的新人来说，希望你能通过短短的课程，掌握 Charles 的日常使用；而对于已经熟悉 Charles 使用的老手，也可将本篇文章作为培训新人使用 Charles 的指南文档。</strong></p>
<h3 data-nodeid="1865">Charles 工作原理</h3>
<p data-nodeid="1866">在正式讲解之前，我们有必要知道 Charles 是如何工作的，下图是 Charles 官方网站上的一个 Charles 工作原理示意图。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4E/3E/CioPOWDy0FeAW7xPAADTPg4Kkjg538.png" alt="2.png" data-nodeid="2034"><br>
由图可见，任何通过 App 端（当然也报告 Web 端）发送给后端的请求，都将被 Charles 截获并转发给后端。同样，任何的后端返回值，也会经由 Charles 转发给客户端，这里的客户端即包括 App 端也包括 Web 端。</p>
<p data-nodeid="1867">由此可见，Charles 的工作原理类似于中间人代理，即任何客户端和服务端的通信都会经过 Charles，于是 Charles 便可以拦截来自客户端和服务端的任何请求，并进行修改。</p>
<p data-nodeid="1868">Charles 在使用前必须进行安装和配置，不同端的配置各有不同。我以 Mac 系统为例，大致说下注意事项。</p>
<h3 data-nodeid="1869">搭建基础代理环境</h3>
<h4 data-nodeid="1870">1.Web 端</h4>
<p data-nodeid="1871">安装我就不介绍了，直接去官方网站下载。安装好 Charles 后，首先去安装根证书。根证书的位置在 Help--&gt; SSL Proxying --&gt; Install Charles Root Certificate。</p>
<p data-nodeid="1872">证书安装后，点击 Charles 的菜单， 选择 Proxy--&gt;macOS Proxy.<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4E/3E/CioPOWDy0HiAQIbfAAFtIRZrgcQ567.png" alt="image-1.png" data-nodeid="2046"></p>
<p data-nodeid="1873">此时，当你在浏览器里访问一个网页的时候，就可以在 Charles 里看到网络请求了。</p>
<p data-nodeid="1874">当前绝大多数的软件都是 HTTPS 访问了，如果想抓包 HTTPS 的请求，则需要通过 Charles 的菜单 Proxy--&gt;SSL Proxying Settings 来进行设置。</p>
<p data-nodeid="1875"><img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0JWAHKMLAAZEDAIEqA4773.png" alt="image-2.png" data-nodeid="2051"></p>
<p data-nodeid="1876">在弹出的对话框中，我们可以进行如下操作：<br>
勾选 Enable SSL Proxying，并点击 ADD 按钮，再编辑如下。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0MSAUDe_AAD0Z8uuUQU404.png" alt="image-3.png" data-nodeid="2058"></p>
<p data-nodeid="1877">Host 填*，Port 填写 443，这样就可以抓取 https 的请求了。</p>
<h4 data-nodeid="1878">2.App 端</h4>
<p data-nodeid="1879">但如果你想在手机上使用 Charles，就需要进行多步配置。</p>
<p data-nodeid="1880">首先，你需要在 Charles 所在的机器上进行如下配置，找到菜单 Proxy--&gt;Proxy Settings 进行以下设置。</p>
<p data-nodeid="1881">填入代理端口 8888（也可以自定义填其他端口）；<br>
并且勾上Enable transparent HTTP proxying；<br>
勾选 Support HTTP/2。</p>
<p data-nodeid="1882"><img src="https://s0.lgstatic.com/i/image6/M00/4E/3E/CioPOWDy0NOAWSrqAAB9LZxs-u8817.png" alt="image-4.png" data-nodeid="2072"></p>
<p data-nodeid="1883">其次，确定手机端代理地址，还是在 Charles 安装的那台机器上。</p>
<p data-nodeid="1884">通过点击 Charles 的菜单help–&gt;SSL Proxying–&gt; Install Charles Root Ceriticate On a Mobile Device or Remote Browser：，然后切换到手机端，根据以下图片进行操作，具体步骤为：</p>
<p data-nodeid="1885"><img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy13SAYEc1AABinGRGZBc309.png" alt="137ionzd0QedwDfu-1.png" data-nodeid="2077"></p>
<p data-nodeid="1886">在 App 端，通过浏览器访问网页：chls.pro/ssl，并下载安装证书。</p>
<p data-nodeid="1887">在 App 端，设置网络代理的值跟图中一致。</p>
<p data-nodeid="1888">这样就可以开始抓包了。</p>
<p data-nodeid="1889">需要注意的是，有些设备特别是 Android 设备无法直接安装下载的证书，需要进入到设置-&gt;安全-&gt;凭据存储 -&gt;从存储设备安装证书。</p>
<h3 data-nodeid="1890">接口过滤、拦截和修改</h3>
<p data-nodeid="1891">当你配置好后，你就可以在 Charles 面板中看到所有的网络请求。</p>
<h4 data-nodeid="1892">1.过滤接口请求</h4>
<p data-nodeid="1893">因为所有的网络请求都会被抓取，那么信息太多也会造成干扰，所以可以过滤接口的请求。过滤接口的请求方式有 2 种。<br>
<strong data-nodeid="2092">1）直接过滤</strong><br>
直接过滤很简单，直接在 Charles 的 Filter 这栏中填写你要过滤的关键字即可。</p>
<p data-nodeid="1894"><img src="https://s0.lgstatic.com/i/image6/M00/4E/3E/CioPOWDy0OSAeiBnAACuEEz6S90184.png" alt="image-5.png" data-nodeid="2095"></p>
<p data-nodeid="1895"><strong data-nodeid="2101">2）通过菜单 Recording Settings 设置</strong><br>
首先，通过 Charles 菜单选择 Proxy--&gt;Recording Settings。</p>
<p data-nodeid="1896"><img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0SeACKIaAAFYOFhGdCg861.png" alt="image-6.png" data-nodeid="2104"></p>
<p data-nodeid="1897">接着在弹出的对话框中，选择 Include， 然后点击 Add 按钮，并按照你的需求设置 Filter。注意：如果你什么都不填写，就表示全部符合条件。</p>
<p data-nodeid="1898"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0TiAYoNqAADsg3jn5AA810.png" alt="image-7.png" data-nodeid="2108"></p>
<p data-nodeid="1899">通过上述设置，只有网站域名为 m.baidu.com 的数据才会被抓取，你也可以根据自己需要更改筛选条件。</p>
<h4 data-nodeid="1900">2.动态修改请求数据</h4>
<p data-nodeid="1901">当发送给服务端的数据需要进行修改时，最简单的方式就是找到这个请求，然后鼠标右键，并选择 Breakpoints。</p>
<p data-nodeid="1902"><img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0UmAVrhrAAQguX3aq1w536.png" alt="image-8.png" data-nodeid="2114"></p>
<p data-nodeid="1903">这样，当你这请求再次被触发时，它就会被拦截。</p>
<p data-nodeid="1904">就拿上图这个例子来说，原本我是在输入框中输入了“iTesting”，然后我右键选择了它为 Breakpoint。这样，当我再次在输入框中输入了“iTesting”时，这个请求就会被拦截，并且供我更改。</p>
<p data-nodeid="1905"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0ViANEaMAAGOz3OQKSw877.png" alt="image-9.png" data-nodeid="2119"></p>
<p data-nodeid="1906">此时，点击 EditRequest，你将会看到这个请求的所有参数，同时你可以直接进行更改后点击 Execute 让它执行。</p>
<p data-nodeid="1907"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3F/CioPOWDy2EOAD-oIAADNt-NhnAg708.png" alt="Goith8efddvk95O1.png" data-nodeid="2123"></p>
<p data-nodeid="1908">点击 Execute 后，请求就会被发送。这时，你还可以对服务端返回给你的数据进行修改。<br>
点击 Edit Response，选择 JSON Text 标签，即可对请求的各项返回进行更改。</p>
<p data-nodeid="1909">如下所示，更改完毕后点击 Execute，服务端返回的内容，就变成你更改后的内容了。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0WaAXb3WAAC0r1pFNL8531.png" alt="image-10.png" data-nodeid="2131"></p>
<p data-nodeid="1910">打断点更改网络请求是 Charles 最常用的一个方法，在实践中，当测试版本升级，模拟接口返回错误时就会用到打断点。</p>
<p data-nodeid="1911">但一个接口、一个接口的打断点毕竟太浪费实践，除此之外，还可以自定义 BreakPoint Settings， 方法如下：菜单栏选择 Proxy--&gt;BreakPoint Settings。</p>
<p data-nodeid="1912"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0XeAOwKXAAFaP6Dq-bs005.png" alt="image-11.png" data-nodeid="2136"></p>
<p data-nodeid="1913">然后在 Breaking Setting 中，勾选“Enable BreakPoints”，然后点击 Add 按钮，添加对某个请求的断点设置如下。</p>
<p data-nodeid="1914"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0ZyASbjnAADN4xxmSh8713.png" alt="image-12.png" data-nodeid="2140"></p>
<p data-nodeid="1915">你可以根据需求，自己定义要抓取的请求地址；并且可以通过勾选 Request、Reponse 的方式来决定是只更改发往服务端的请求数据（勾选 Request 即可）。</p>
<p data-nodeid="1916">讲过了接口过滤、拦截和修改。我们就知道，这部分的应用场景就是普通的接口请求的查看、手工更改以及返回。</p>
<p data-nodeid="1917">关于这部分，有一个典型应用场景，也是我面试时考察 Charles 使用的必考题目。</p>
<p data-nodeid="1918"><strong data-nodeid="2147">那就是：假设我有这么一个需求，客户端判断服务端返回的 SDK 版本号，如果大于 2.0，就会弹出一个对话框提升你更新 app。但是当前我们 SDK 最高才到 1.0，问你该怎么办？</strong></p>
<p data-nodeid="1919">仅仅这一个问题，即考察了接口的过滤、拦截、修改这 3 个部分，如果测试工程师在面试时答不出这道题，那么他大概率就不会通过面试。</p>
<h3 data-nodeid="1920">远程映射</h3>
<p data-nodeid="1921">通过 Breakpoint 的方式进行修改请求数据虽然有效，但是修改请求这个操作需要人工干预，需要花费时间。再者，在测试时有可能接口返回，你已经提前知道了，那么就不需要先请求再更改数据，可以准备好返回数据直接进行模拟。</p>
<p data-nodeid="1922"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0aqAc_rUAAQLyj04DN4725.png" alt="image-13.png" data-nodeid="2153"></p>
<p data-nodeid="1923">方式为点击 Tools --&gt; Map Local，然后在弹出的对话框中，勾选“Enable Map Local”，接着设置你要覆盖的请求。</p>
<p data-nodeid="1924">注意：Map From 是你的原始请求。Map To 是你期望的结果，这个结果你可以放在本地文件中，以 Text 或者 Json 的格式保持都可以。当你使用 Map Local 后，当有目标请求发生时，Charles 直接将你提供的文件内容当做返回值返回。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/3E/CioPOWDy0b2AIiAQAAD8MHnIaLw377.png" alt="image-14.png" data-nodeid="2159"></p>
<p data-nodeid="1925">远程映射常常用于客户端或者服务端，对返回的时间有超时判断，或者需要更改太多服务端返回值时使用。 毕竟一个个更改接口返回值需要花费时间，而由于超时时间非常短，等你改完再去执行，接口就直接超时了，这个时候就需要用到远程映射来直接更改。</p>
<h3 data-nodeid="1926">慢网络模拟</h3>
<p data-nodeid="1927">测试时模拟各自不同网络速度的情况也比较常见，Charles 可以轻松做到慢网络模拟。</p>
<p data-nodeid="1928"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3F/CioPOWDy2SCAcfsqAAXpPRN4AJs592.png" alt="j8vNw3ghmdm42RcG.png" data-nodeid="2165"></p>
<p data-nodeid="1929">从菜单 Porxy--》Throttle Settings 进入，先通过 Add 添加你要进行网络速度限制的站点，然后勾选“Enable Throttling”，选择“Throttle preset”，这个时候你会看到不同的网络速度情况，可以根据需要选择，即可完成网络速度限制。</p>
<p data-nodeid="1930"><img src="https://s0.lgstatic.com/i/image6/M01/4E/3F/CioPOWDy2VaAXWmUAAG9NEmkJS8007.png" alt="image.png" data-nodeid="2169"></p>
<h3 data-nodeid="1931">微服务分支测试</h3>
<p data-nodeid="1932">在采用微服务架构后，我们就需要对不同版本的微服务分支进行测试，那么这就有必要进行分支测试。在当下，分支测试通常都是根据 Header 进行区分。既然如此，我们就可以给指定的请求添加 Header，以 Charles 也可以进行微服务分支测试，方式如下：<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy2YqAa-hfAADSXs6uK5g035.png" alt="image-1.png" data-nodeid="2175"></p>
<p data-nodeid="1933">菜单 Tools -&gt; Rewrite，然后勾选“Enable Rewrite”“Debug in Error Log”，点击 Add，并在右上的 Location 这个栏目下，点击 Add。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/37/Cgp9HWDy1giAU1hXAABs8WFqp_o829.png" alt="image-17.png" data-nodeid="2180"></p>
<p data-nodeid="1934">在弹出的对话框中， Host 输入*，点击 OK。<br>
输入* 代表任何地址都将被 Rewrite。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/37/Cgp9HWDy1mmAObd0AAA0VBTypNI080.png" alt="fKox6hTdhwLkDNs4.png" data-nodeid="2191"></p>
<p data-nodeid="1935">然后继续点击下图中的 Add 按钮。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/37/Cgp9HWDy1qSATVfhAABk5nW2APg309.png" alt="wHmdob7WIpbvedo1.png" data-nodeid="2196"></p>
<p data-nodeid="1936">在下面的图中配置 Rewrite 的规则。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy1eSAD-lrAABwpiyp1M4696.png" alt="70ddwRrrK0MfefFm-1.png" data-nodeid="2201"></p>
<p data-nodeid="1937">其中，Type 的值是个下拉列表， 选择 add Header，where 勾选 Request。</p>
<p data-nodeid="1938">然后在下方 Replace 栏目，输入你想增加的 header 的 key 和 value，保存即可。<br>
<img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy1FyAeVytAAB98mT_kKs947.png" alt="image-18.png" data-nodeid="2207"><br>
通过这种方式，可以对微服务的不同分支进行测试。</p>
<h3 data-nodeid="1939">简单接口并发测试</h3>
<p data-nodeid="1940">Charles 还可以做接口并发。针对某个接口，假设我们想测试其基本的性能，可以直接选择那个接口，然后右键选择 Repeat Advanced。</p>
<p data-nodeid="1941"><img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy1DyAMnQWAAOEVhedncI860.png" alt="image-19.png" data-nodeid="2214"></p>
<p data-nodeid="1942">此时，会弹出设置的框让你选择，其中：</p>
<ul data-nodeid="1943">
<li data-nodeid="1944">
<p data-nodeid="1945">Iterations：是并发轮次数，进行多少轮次的测试。</p>
</li>
<li data-nodeid="1946">
<p data-nodeid="1947">Concurrency：是并发线程数，每轮测试几个请求同时发。</p>
</li>
<li data-nodeid="1948">
<p data-nodeid="1949">Repeat delay：可设置轮次之间的间隔，以毫秒计算。</p>
</li>
</ul>
<p data-nodeid="1950"><img src="https://s0.lgstatic.com/i/image6/M00/4E/36/Cgp9HWDy1TuAQo4YAAA3PAqFYE8958.png" alt="6NwKQzfJZvMKVm5m.png" data-nodeid="2221"></p>
<p data-nodeid="1951">点击确定。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy1CqAa1L8AADK1w24_t4053.png" alt="image-20.png" data-nodeid="2226"></p>
<p data-nodeid="1952">你会看到，所有的请求已经发出来了。通过这种方式，压力就产生了，我们可以观察服务端的响应时间来判断起基本的性能是否达标。</p>
<p data-nodeid="1953">以上就是使用 Charles 进行网络抓包的常用方法。在实际工作中，掌握 Charles 的抓包是每个测试工程师必备的基本技能，如果你对上面的介绍的几种 Charles 用法还不熟悉，那么你可以赶快按照上文介绍，掌握其使用；否则，很难适应当前微服务架构下的复杂测试要求。</p>
<p data-nodeid="1954">Charles 的应用过程中，不少同学会有不少问题。我准备了一些常见问题并给予解答。</p>
<h3 data-nodeid="1955">Charles 应用常见问题</h3>
<p data-nodeid="1956"><strong data-nodeid="2238">【Question 1 】</strong><br>
问：配置好 Charles 后，浏览网页提示“您的连接不是私密连接”<br>
答：打开 SSL Proxying Setting，在弹出的对话框中，做如下操作。</p>
<ol data-nodeid="1957">
<li data-nodeid="1958">
<p data-nodeid="1959">勾选 Enable SSL Proxying；</p>
</li>
<li data-nodeid="1960">
<p data-nodeid="1961">点击 ADD 按钮，编辑如下。<br>
Host 填*，Port 填写 443，这样就可以抓取 https 的请求了。</p>
</li>
</ol>
<p data-nodeid="1962"><img src="https://s0.lgstatic.com/i/image6/M00/4E/3F/CioPOWDy05mAHaFUAAD0Z8uuUQU067.png" alt="image-21.png" data-nodeid="2247"></p>
<p data-nodeid="12466" class="te-preview-highlight"><strong data-nodeid="12474">【Question 2 】</strong><br>
问：Charles 关闭，电脑端就无法上网了，为什么？<br>
答：查看电脑的网络设置-&gt;高级设置-&gt;代理，检查并取消选中 HTTP 和 HTTPS 代理选项。</p>












<p data-nodeid="1964"><strong data-nodeid="2263">【Question 3 】</strong><br>
问： iOS 上证书安装了，但是无法抓包，为什么？<br>
答： 两个检查。</p>
<ol data-nodeid="1965">
<li data-nodeid="1966">
<p data-nodeid="1967">检查是否信任该证书。 在手机设置-&gt;通用-&gt;关于本机-&gt;证书信任设置中信任。</p>
</li>
<li data-nodeid="1968">
<p data-nodeid="1969">通用-&gt;描述文件与设备管理，选中安装的配置文件，并验证。</p>
</li>
</ol>
<p data-nodeid="1970"><strong data-nodeid="2273">【Question 4 】</strong><br>
问：App 端开启代理，为什么 Charles 没有出现“允许”提示？<br>
答： App 端和电脑端必须连接同一个网络。</p>
<p data-nodeid="1971"><strong data-nodeid="2281">【Question 5 】</strong><br>
问：某些安卓机型证书下载后无法直接安装。<br>
答：通常见于安卓机，例如小米手机。证书下载后，从“更多设置-&gt;系统安全-&gt;从存储设备安装”这个路径进行安装。</p>
<h3 data-nodeid="1972">小结</h3>
<p data-nodeid="1973">同学们，这就是 Charles 最常用的功能了。Charles 作为网络抓包第一利器，熟练掌握其用法，可以提升我们的手工测试效率，把我们从繁琐的重复性劳动中解放出来。</p>
<p data-nodeid="1974" class="">对于今天的学习，你有哪些收获呢？可以通过下图进行回顾消化哦。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/4E/36/Cgp9HWDy0DqAIU_4AAEWQezKFMI897.png" alt="image.png" data-nodeid="2288"><br>
今天我们讲的内容就到这里了，我是蔡超，我们下次再见。</p>

---

### 精选评论

##### **阳光：
> windows版的都是乱码啊

##### **栓：
> 非常实用！感谢

