<p style="line-height: 1.75em; text-align: justify;"></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">内容</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">主要分为</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"> 4 </span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">个</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">模块</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：</span><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 1 个模块中会讲解什么是长连接和短连接。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 2 个模块讲解长连接和短连接应用场景。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 3 个模块结合 Timewait 和 TCP 连接的关系来讲解它们是如何产生的。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 4 个模块会分析 Timewait 对系统性能产生的一些影响，以及应该如何去进行优化。 </span></p></li>
</ul>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">什么是长连接和短连接</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们进入第 1 个模块的学习，什么是长连接和短连接？这里我画了一张图：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/10/79/Ciqah16X-TaACWjHAARWKvt67Qs928.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">左边是短连接，右边是长连接。我们会看到短连接为每一次的数据传输准备了一个传输通道，比如客户端向服务端要传送数据的时候，它会先建立连接，然后传递数据，最后关闭连接。当要传递第 2 份数据的时候，又要重复这个过程。所以短连接就是在每一次传输数据前，建立一次连接的通道。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">长连接则是建立了一条可以连接通道，并一直保持，每一次传输数据时会复用同一条连接通道。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">建立长连接的前提</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">了解了长连接&amp;短连接各自的原理后，我们再来讲一讲，如果客户端要跟服务端建立长连接，它的前提是什么？ </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">主要有两个前提，</span><strong style="font-size: 12pt;">第 1 </strong><strong style="font-size: 12pt;">个</strong><strong style="font-size: 12pt;">是客户端</strong><strong style="font-size: 12pt;">使用长连接方式</strong><strong style="font-size: 12pt;">请求。第 2 </strong><strong style="font-size: 12pt;">个是服</strong><strong style="font-size: 12pt;">务端需要支持长连接</strong><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">。</span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">客户端请求做长连接是指，客户端不能主动去断开一条连接，而是要在复用连接数据传输。服务端需要支持长连接是指在长连接的时间周期内不能主动断开连接，并能支持客户端的请求模式。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">我们可以来看一下服务端，拿服务端常用的代理服务 Nginx 为例，在 Nginx 里有一个 keepalive 的配置，如果我们把它设置为 0，则表示把长连接关闭，服务端只能支持短连接，如果我们把 keepalive 设置一个较长的时间周期（比如写成 </span>keepalive_timeout 120s），<span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">它表示支持两分钟的长连接。 </span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/89/8F/Cgq2xl6X-TaAeLlvAAB2ENLEb-Q379.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">keepalive_timeout 控制着长连接的时间周期。如果我们想要观察是否是客户端和服务端建立起来了 HTTP 长连接，可以在 Chrome 浏览器里打开开发者工具，请求一个网站指定的地址，然后就可以在"network"标签选项里看到请求头和请求返回头，Request Headers 里面有一个 Connection，如果数值是 keep-alive 则表示是以长连接的方式去寻求服务端。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/03/4A/CgoCgV6X-TeAAaR6AAB5Rzix4gA237.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">要判断服务端返回的内容是否支持长连接，我们也可以通过同样方式，查看 Response Headers 里面的 Connection，如果也是 keep-alive，则表示用双向建立长连接的方式。 </span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">HTTP 协议与长连接的关系</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">刚讲到了长连接和短连接原理，那么接下来我们再围绕 HTTP 协议讲一讲它和长连接的关系。我们知道 HTTP 协议是一个常见的互联网协议标准，我们访问网站都使用了 HTTP 协议。HTTP 协议版本分为： 1.0 版本、 1.1 版本及正逐步广泛使用的 HTTP 2.0 版本。 </span></p>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/89/8F/Cgq2xl6X-TeAKMNgAAEzmy2gzLQ928.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">HTTP 1.0 是非常早的版本了，它对长连接是不支持的，如果老的浏览器只支持 1.0 版本，就无法建立 HTTP 协议的长连接。HTTP 1.1 版本的话,目前绝大部分的浏览器都是可以支持的， 通过 HTTP1.1 协议可以实现长连接但无法做到复用数据通道做传输，而 HTTP 2.0 是近几年来推出的新协议，不仅支持长连接，更可以支持连接复用数据通道来做传输。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">什么是连接复用传输模式呢？这里我画了两张图，下面这张图是基于 HTTP1.1 的长连接方式。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/10/79/Ciqah16X-TeAKr5mAAMZsMci8Zs140.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们会看到建立起一次 TCP 连接以后，传输样例是这样的：客户端向服务端请求一个 index 。html 页面，服务端会返回对应的内容，后面请求 aa.css、bb.js，服务端只能串行逐一返回，每一个请求和对应响应都在一条通道内完整完成后才能请求后一个数据。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">而 HTTP2.0 复用数据的方式是这样的，建立起 TCP 连接通道以后，客户端向服务端请求的是 index html，接下来请求 aa.css 、 bb.js，可以做到同时发起两个请求到服务端，而服务端可以同时响应并行传送数据到客户端，这样就做到了</span><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">多路“HTTP数据传送”</span><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63); font-size: 12pt;">。 </span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/03/4A/CgoCgV6X-TiABS7ZAAMqlyk1mEg118.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">刚刚讲到了长连接和短连接各自的一些原理及内容，那么接下来我将为你讲解长连接和短连接各自的优势是什么？ </span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">长连接和短连接的优势与劣势</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们要从 4 个方面来分析它们的优劣：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 1 个就是 TCP 的建连资源开销，我们会看到短连接和长连接最大的开销就是 TCP 的建连，这一块的开销也是因为它们的差异性而引起的，由于短连接在每一次数据请求时都要建立连接，所以说它的 TCP 建连会比长连接的开销更大。长连接只用到了一次建连，所以相比短连接来说建连的开销更小。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 2 个为是否存在探活资源开销。我们知道，长连接要保持一条连接通道的话，那么服务端和客户端要实时保持探活的状态。探活的资源开销对于长连接来说是存在的，而对于短连接来说是不存在的。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 3 个我们要对比在承受同样的并发请求情况下服务端的支撑能力。可以这么理解，假设短连接和长连接，同样是 1000 个并发请求到服务端，在同样请求个数的前提下，长连接的服务端整体开销会更大一些，而短连接因为它建立起来就断掉了，所以它的时间周期更小，对服务端的影响略小一些。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 4 个区别就是服务端是否支持主动连接客户端。长连接是可以轻易做到的，在很多聊天室的场景里面用长连接是一个不错的选择，而短连接往往是客户端去请求服务端连接就关闭了，无法做到在 TCP 连接基础上实现双向主动通信。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这就是长连接和短连接的区别。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">长连接和短连接的使用场景</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那么我来为你总结一下长连接和短连接的使用场景，在大部分 Web 服务或是 API 网站服务里，基本会使用短连接。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">长连接的应用场景有三个：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 1 种是运用在后端数据库的连接里，比如我们会看到 Java 有一个连接池，连接池就是配置的固定连接数据库的连接通道，连接后端数据库的场景里，长连接会更加常用。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 2 种方式就是刚才讲到的聊天室场景，因为服务端要向客户端主动推送消息。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 3 种是服务端向客户端主动推送消息，也就是消息推送，我们看到手机端的一些消息推送，也属于长连接常用到的一些场景。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来我们来分析连接和常见连接状态 Timewait 的关系。当我们做运维的时候，常会看到系统上报 Time wait 个数超过系统允许 bucket 队列的个数限制，那么这些警告对我们来说有多大的影响？接下来进行讲解。 </span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Timewait 和 TCP 连接的关系</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先我们需要了解 Timewait 和连接的关系。Timewait 的连接状态是由 TCP 的 4 次挥手过程产生的。4 次挥手的过程如图所示。我们可以看到，左边是主动关闭的一端，右边就是被动关闭的一端。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/89/8F/Cgq2xl6X-TiAfMBxAASAApv99no139.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">主动关闭的一端向服务端发送第 1 次挥手的请求，被动方返回了一个 ACK 的包，可以看到被动关闭端再一次发送完 FIN 和 ACK 包以后，主动关闭端就会会先发送一个 ACK 响应回去，然后进入 TIME_WAIT 状态。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们看到在它进入 TIME_WAIT 状态之前，理论上 4 次挥手已经完成了，为什么 TIME_WAIT 还需要保留一段时间？ </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这是因为 TCP的协议标准，需要保证4 次挥手过程中最后一次连接发送的稳定性，如果ACK包发送不成功，就需要再次发送 ACK 包。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">大量的 Timewait 产生会造成文件句柄、内存和端口的占用，由于系统会把过多的 time-wait socket 删除、回收，在网络条件不好的情况下，就可能会导致数据包重复的进行发送。 </span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如何对 TimeWait 进行优化</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">了解了 Timewait 的影响（结论：通常不会直接造成服务连接的影响，但是会造成一些资源上及新建连接风险），为了避免过多的 Timewait 产生，我们需要考虑去进行一些优化。在单机系统上做性能优化的话，我们需要考虑两点：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 1 点就是考虑把Timewait 队列加大。在操作系统资源、硬件资源能满足的情况下，我们可以把 tcp_max_tw_buckets 的值数调高，它的缓冲值也就越大。这个数字是我们可以进行操作系统内核优化的。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 2 点即我们需要尝试修改操作系统内核优化的内容，也就是调整 TIME_WAIT 超出时间，如果它的操作时间能够更快地让操作系统进行回收，那么它就可以更快地释放资源。当然这个时间也不能调的太小，要不然 Timewait 的作用的意义就很小了。所以我建议操作系统内核的参数 tcp_fin_timeout 这个值调到 30，这就是一个比较合理的优化空间。 </span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">以上就是 Timewait 的优化方式。</span></p>

---

### 精选评论


