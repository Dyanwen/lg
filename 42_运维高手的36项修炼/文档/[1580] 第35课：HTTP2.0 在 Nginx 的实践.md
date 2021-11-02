<p data-nodeid="1084" class="">本课时讲解的内容是关于 HTTP2.0 在 Nginx 中的实践。我们会介绍 HTTP2.0 的协议，同时讲解 Nginx 是如何支持 HTTP2.0 特性作配置。</p>


<h3 data-nodeid="493">关于 HTTP 协议</h3>
<p data-nodeid="1538">在当前这个互联网时代，越来越多的应用场景使用 HTTP 协议（超文本传输协议），它是一个十分重要的协议。HTTP 当前主要使用的版本是 HTTP1.1，HTTP2.0 和 HTTP3.0，在这张图里我们先来整体了解下。</p>
<p data-nodeid="1539" class=""><img src="https://s0.lgstatic.com/i/image/M00/3A/5C/CgqCHl8hWe-ABsyqAABPxZNtCbI636.png" alt="image.png" data-nodeid="1543"></p>


<p data-nodeid="496">HTTP1.1 算是早期的版本了，如图所示它包含几大部分：HTTP 协议报文，底层 TCP 内容，还有 IP 及 MAC 这些内容。</p>
<p data-nodeid="497">在 HTTP1.1 协议里，最大的问题就是安全问题，因为它没法做到数据加密，为了解决这种数据安全性的问题，就引入了 HTTPS 协议。</p>
<p data-nodeid="498">HTTPS 对比 HTTP1.1 就是支持 SSL 数据加密并传送，这就解决了数据传输被劫持的问题。通过 HTTPS 做数据的加密和解密虽然解决了安全的问题，但造成性能损耗及增加多次客户端和服务端建连，所以性能造成了一定损耗，如何同时保障安全和性能更优呢？HTTP 协议本身的内容就有必要进行升级，也就接下来我们要讲的 HTTP2.0。</p>
<p data-nodeid="499">HTTP2.0 提出重要概念 Stream（流），将一个 TCP 连接分为若干个流（Stream），每个流中可以传输若干消息（Message），它是以流的方式进行数据包传输，通过建立 I/O 复用的机制来保障客户端连接和服务端请求响应的效率。</p>
<p data-nodeid="500">另外就是在压缩上的变化，它采用了 Hpark 机制进行头部压缩。而 HTTP1.1 协议是不做 HTTP 头信息的压缩的，HTTP2.0 则通过 Hpark 的方式进行了头部压缩，这样就减少了客户端和服务端传送的流量，并且 Hpark 在服务端建立起了一个字典，只以增量的方式传输投入的报文，所以整个这套机制改善了头部报文传送效率问题。</p>
<p data-nodeid="501">这就是 HTTP2.0 的协议最大的两个特性，而 HTTP3.0 的诞生解决了 HTTP2.0 一些缺陷，这些缺陷我们将在课程后面的内容为你介绍。</p>
<h3 data-nodeid="502">HTTP2.0 协议</h3>
<p data-nodeid="503">接下来我们要重点介绍的就是 HTTP2.0 协议。说首先来具体讲一讲 HTTP2.0 协议解决了 HTTP1.1 协议的哪些问题，也就是 HTTP1.1 协议里存在的劣势。</p>
<h4 data-nodeid="504">HTTP1.1 的劣势</h4>
<p data-nodeid="505">首先就是HTTP1.1性能问题。</p>
<p data-nodeid="506">第一表现是 TCP 连接存在请求阻塞的问题，因为一个连接同时只能处理一个请求，而浏览器又只能以先进先出的按照优先级的方式进行请求处理。所以如果一个响应没有及时返回时，后面的请求就都会阻塞。</p>
<p data-nodeid="507">为了优化请求阻塞的问题，很多技术人员开发前端架构会设计成多个域名，这样就可以让浏览器和服务端同时建立起多个连接。但是浏览器对于用 HTTP1.1 协议同时创建的连接个数是有限制的（不同浏览器限制个数不一样）。所以使用这种方式提高页面数据加载效率有限。</p>
<p data-nodeid="508">第二个问题就是内容的明文传输，我们刚刚讲过，HTTP1.1 协议对于数据内容是以明文的方式传输的，没有任何加密，所以容易出现内容被劫持。</p>
<h4 data-nodeid="509">HTTP2.0 的优势</h4>
<p data-nodeid="2224">HTTP1.1 这两点劣势在 HTTP2.0 都得到了很好的解决，接下来我们就介绍 HTTP2.0。</p>


<p data-nodeid="511">首先是 HTTP2.0 多路复用机制，所谓多路复用，就是在一个 TCP 连接里面，客户端可以通过 HTTP2.0 协议同时发送多个请求流，实现单连接上多请求 - 响应并行，这就解决了连接阻塞的问题，减少了 TCP 连接的数量和 TCP 连接慢的情况。</p>
<p data-nodeid="512">第二就是 HTTP2.0 用到了分帧二进制的方式进行传输，也就是在一个请求流中以数据帧作为传输的最小单位，实现二进制传输，并且基于 HTTPS 协议，解决明文传输安全性问题。</p>
<p data-nodeid="2337">最后值得一提的就是服务端的主动推送机制，浏览器发送一个请求的时候，服务端会主动向浏览器推送与请求相关的资源，这个时候浏览器就不用再发起后续的请求了。</p>
<p data-nodeid="2338" class=""><img src="https://s0.lgstatic.com/i/image/M00/3A/5C/CgqCHl8hWhOANgePAAIKBW6JBzk016.png" alt="image1.png" data-nodeid="2342"></p>




<p data-nodeid="515">在上面这张图中，左边是服务端，右边是客户端，客户端在请求流 stream1（也就是 page.html）的时候，服务端就会主动向客户端响应，除了 page.html 响应的内容以外，还同时响应了 stream2、stream4(script.js、style.css)，服务端主动响应给了客户端，这样机制减少多次连接，从而提高对应页面的响应性能和用户体验。</p>
<h3 data-nodeid="516">HTTP2.0 在 Nginx 中的配置实践</h3>
<p data-nodeid="517">了解了 HTTP2.0 的优势以后，我们就 Nginx 是如何支持 HTTP2.0 协议的为你进行介绍。</p>
<p data-nodeid="518">我们首先讲到的是对 HTTP2.0 整个协议的支持，在 Nginx 的版本里，支持 HTTP2.0 的协议对其版本是有要求的，Nginx 的软件版本需要大于 Nginx1.10 的版本，才能支持到 HTTP2.0 协议。同时，由于 openssl 版本做了升级，所以对操作系统或 Linux 系统上面 openssl 库版本也有要求，openssl 需要大于 1.0.2 版本，在同时满足了这样的前提下，我们就可以来配置 Nginx 配置文件以使得 Nginx 支持 HTTP2.0 协议。</p>
<pre class="lang-java" data-nodeid="3266"><code data-language="java">server { 
&nbsp; &nbsp; listen&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">443</span> ssl http2; 
&nbsp; &nbsp; charset&nbsp; &nbsp; &nbsp; utf-<span class="hljs-number">8</span>; 
&nbsp; &nbsp; server_name&nbsp; www.imoocc.com imoocc.com; 
&nbsp; &nbsp; access_log&nbsp; /opt/app/jeson/logs/https_access.log&nbsp; main; 
&nbsp; &nbsp; error_log&nbsp; /opt/app/jeson/logs/https_error.log; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; 
&nbsp; &nbsp; ssl_certificate /jeson/key/www.imoocc.com.pem; 
&nbsp; &nbsp; ssl_certificate_key /jeson/key/www.imoocc.com.key; 
&nbsp; &nbsp; ssl_session_timeout <span class="hljs-number">5</span>m; 
&nbsp; &nbsp; ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; 
&nbsp; &nbsp; ssl_protocols TLSv1 TLSv1.<span class="hljs-number">1</span> TLSv1.<span class="hljs-number">2</span>; 
&nbsp; &nbsp; ssl_prefer_server_ciphers on; 
    … 
} 
</code></pre>




<p data-nodeid="3620">配置如上，对于 http2.0 协议的支持 Nginx 配置文件中在监听的端口后面加入了一个 http2（listen&nbsp;443 ssl http2），这就配置使得 Nginx 里面支持 HTTP2.0，配置完毕可以重启 Nginx 服务。</p>
<p data-nodeid="4080">接下来，测试对比 Nginx 分别在 HTTP1.1 和 HTTP2.0+HTTPS 两个场景中性能上的差异，这个测试我们在前端打开浏览器-开发者工具，示例这里先访问的是我的 HTTP1.1协议博客地址（<a href="http://www.jesonc.com" data-nodeid="4085">http://www.imoocc.com</a>）。</p>
<p data-nodeid="4081" class=""><img src="https://s0.lgstatic.com/i/image/M00/3A/51/Ciqc1F8hWmOAPL8qAAIV_f1lXdE332.png" alt="image (1).png" data-nodeid="4093"></p>





<p data-nodeid="522">同时在底端点击鼠标右键，把“协议“这一栏展示出来。在刷新浏览器请求以后，就会看到这样一个页面，这个页里面我们需要重点关注的是三部分，一部分是客户端发起的协议版本类型（Protocol），我们会看到在这一栏里面，向服务端发送的协议版本都是 HTTP1.1。</p>
<p data-nodeid="523">图中右侧是一个连接和请求上的一些关系以及对应的延迟。我们可以在这张图里面关注到，这些页面的元素并没有做到并行的加载。</p>
<p data-nodeid="524">图中下方可以看到整体页面加载完毕后的时长（Finish 8.22s）。</p>
<p data-nodeid="525">我们同样对比通过 HTTP2.0 + HTTPS 访问的这个页面效果，服务端代理 Nginx 中配置完 HTTP2.0 的支持以后，我的浏览器再访问博客站点（地址： <a href="https://www.imoocc.com" data-nodeid="585">https://www.imoocc.com</a>）。</p>
<p data-nodeid="4562">首先是请求协议变化，我们会看到 Chrome 的开发者工具中 Protocol 这一栏是 HTTP2.0 的协议版本。</p>
<p data-nodeid="4563" class=""><img src="https://s0.lgstatic.com/i/image/M00/3A/5C/CgqCHl8hWnGAW0_eAALw9UqakRE997.png" alt="image (2).png" data-nodeid="4571"></p>


<p data-nodeid="528">另外在图的右边元素更多并行的方式加载，最后会看到整个页面的加载时长会略低于前面这张图的时长，因为 HTTPS 本身在整个页面的响应时就会增加，即使这样用HTTP2.0+HTTPS也能明显感觉到页面的加载速度优于 HTTP1.1 协议页面的加载速度。</p>
<h4 data-nodeid="529">Nginx 配置服务端推送</h4>
<p data-nodeid="530">接下来我们再来演示一个案例-服务端的主动推送，这个支持要求 Nginx 版本大于或等于 1.13.9 的版本，Nginx 的配置如下：</p>
<pre class="lang-java" data-nodeid="12969"><code data-language="java">server_name&nbsp; www.imoocc.com; 
&nbsp;root /test;&nbsp; 
&nbsp;index index.html index.htm;&nbsp; 
&nbsp;location = /index.html {&nbsp; &nbsp; 
&nbsp;http2_push /css/style.css;&nbsp; 
&nbsp;http2_push /js/main.js;&nbsp; 
&nbsp;http2_push /img/yule.jpg;&nbsp; 
&nbsp;http2_push /img/avatar.jpg;&nbsp;&nbsp; 
} 
</code></pre>


































<p data-nodeid="13454">如果用户请求 index.html 页面（首页），那么服务通过配置项 http2_push+ 加上对应的资源路径。<br>
这就表示服务端会主动推送这些元素（/css/style.css、/js/main.js、/img/yule.jpg、/img/avatar.jpg）给到客户端，同样通过浏览器的开发者工具在客户端分析，会明显地看到页面加载元素展示上 Push *** 展示方式，这个就表示打开这个页面服务端主动推送过来的页面元素。</p>
<p data-nodeid="13455" class=""><img src="https://s0.lgstatic.com/i/image/M00/3A/51/Ciqc1F8hWpOAD5wgAATOaZqbqn0907.png" alt="image (3).png" data-nodeid="13469"></p>


<p data-nodeid="534">那么我们再来介绍一下 HTTP3.0 优化 HTTP2.0 的哪些内容，HTTP3.0 最大的特点就是支持了 QUIC 协议，QUIC 协议最大的优化方式就是解决了 HTTPS 会话缓存的问题（课时 2 有讲解会话缓存的原理），在客户端缓存 HTTPS 认证回话信息，再一次访问服务端的时候，就可以不用重新建立 HTTPS 会话了，而是在原有认证信息的基础上，直接建立连接。这种方式优化了 HTTPS 这种多次建立连接的问题，由于页面建连速度非常快，所以专业术语称呼为 0RTT 建连**。**</p>
<p data-nodeid="848">另外，虽然 HTTP2.0 虽然解决了 TCP 连接阻塞问题，但是还是存在对列头阻塞的情况，HTTP3.0 中 QUIC 协议就完全不使用 TCP 报文了，而是使用了 UDP 的报文，由于UDP是无连接的，一个连接上的多个 stream 之间没有依赖，数据包阻塞不会影响后面报文正常传送。另外，因为 TCP 是基于 IP 和端口去识别连接的，这种方式在移动端网络等弱网环境造成影响连接的成功率下降，而 QUIC 是通过 ID 的方式去识别一个连接，不管你网络环境如何变化，只要 ID 不变，就能迅速重连上。</p>

---

### 精选评论


