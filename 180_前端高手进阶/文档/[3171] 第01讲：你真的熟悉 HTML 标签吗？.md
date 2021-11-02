<p data-nodeid="1089" class="">本课时主要讲解那些“看不见”的 HTML 标签。</p>
<p data-nodeid="1090">提到 HTML 标签，前端工程师会非常熟悉，因为在开发页面时经常使用。但往往关注更多的是页面渲染效果及交互逻辑，也就是对用户可见可操作的部分，比如表单、菜单栏、列表、图文。</p>
<p data-nodeid="1091">其实还有一些非常重要却容易被忽视的标签，这些标签大多数用在页面头部 head 标签内，虽然对用户不可见，但如果在某些场景下，比如交互实现、性能优化、搜索优化，合理利用它们就可以达到事半功倍的效果。</p>
<p data-nodeid="1092">这一课时就和你来聊聊那些“看不见”的 HTML 标签及其使用场景。</p>
<h3 data-nodeid="1093">交互实现</h3>
<p data-nodeid="1094">我经常会向我的团队成员提倡一个编码原则：<strong data-nodeid="1198">Less code, less bug</strong>。</p>
<p data-nodeid="1095">在实现一个功能的时候，我们编写的代码越多，不仅开发成本越高，而且代码的健壮性也越差。<br>
它和 KISS（Keep it simple, stupid）原则及奥卡姆剃刀原则（如无必要，勿增实体）有相同的意思，都是提倡<strong data-nodeid="1206">编码简约</strong>。</p>
<p data-nodeid="1096">下面介绍几个标签，来看看如何帮助我们更简单地实现一些页面交互效果。</p>
<h4 data-nodeid="1097">meta 标签：自动刷新/跳转</h4>
<p data-nodeid="1098">假设要实现一个类似 PPT 自动播放的效果，你很可能会想到使用 JavaScript 定时器控制页面跳转来实现。但其实有更加简洁的实现方法，比如通过 meta 标签来实现：</p>
<pre class="lang-html" data-nodeid="1099"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">http-equiv</span>=<span class="hljs-string">"Refresh"</span> <span class="hljs-attr">content</span>=<span class="hljs-string">"5; URL=page2.html"</span>&gt;</span>
</code></pre>
<p data-nodeid="1100">上面的代码会在 5s 之后自动跳转到同域下的 page2.html 页面。我们要实现 PPT 自动播放的功能，只需要在每个页面的 meta 标签内设置好下一个页面的地址即可。</p>
<p data-nodeid="1101">另一种场景，比如每隔一分钟就需要刷新页面的大屏幕监控，也可以通过 meta 标签来实现，只需去掉后面的 URL 即可：</p>
<pre class="lang-html" data-nodeid="1102"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">http-equiv</span>=<span class="hljs-string">"Refresh"</span> <span class="hljs-attr">content</span>=<span class="hljs-string">"60"</span>&gt;</span>
</code></pre>
<p data-nodeid="1103">细心的你可能会好奇，既然这样做又方便又快捷，为什么这种用法比较少见呢？</p>
<p data-nodeid="1104">一方面是因为不少前端工程师对 meta 标签用法缺乏深入了解，另一方面也是因为在使用它的时候，刷新和跳转操作是不可取消的，所以对刷新时间间隔或者需要手动取消的，还是推荐使用 JavaScript 定时器来实现。但是，如果你只是想实现页面的定时刷新或跳转（比如某些页面缺乏访问权限，在 x 秒后跳回首页这样的场景）建议你可以实践下 meta 标签的用法。</p>
<h4 data-nodeid="1105">title 标签与 Hack 手段：消息提醒</h4>
<p data-nodeid="1106">作为前端工程师的你对 B/S 架构肯定不陌生，它有很多的优点，比如版本更新方便、跨平台、跨终端，但在处理某些场景，比如即时通信场景时，就会变得比较麻烦。</p>
<p data-nodeid="1107">因为前后端通信深度依赖 HTTP 协议，而 HTTP 协议采用“请求-响应”模式，这就决定了服务端也只能被动地发送数据。一种低效的解决方案是客户端通过轮询机制获取最新消息（HTML5 下可使用 WebSocket 协议）。</p>
<p data-nodeid="1108">消息提醒功能实现则比较困难，HTML5 标准发布之前，浏览器没有开放图标闪烁、弹出系统消息之类的接口，只能借助一些 Hack 的手段，比如修改 title 标签来达到类似的效果（HTML5 下可使用 Web Notifications API 弹出系统消息）。</p>
<p data-nodeid="1109">下面这段代码中，通过定时修改 title 标签内容，模拟了类似消息提醒的闪烁效果：</p>
<pre class="lang-js" data-nodeid="1110"><code data-language="js"><span class="hljs-keyword">let</span> msgNum = <span class="hljs-number">1</span> <span class="hljs-comment">// 消息条数</span>
<span class="hljs-keyword">let</span> cnt = <span class="hljs-number">0</span> <span class="hljs-comment">// 计数器</span>
<span class="hljs-keyword">const</span> inerval = <span class="hljs-built_in">setInterval</span>(<span class="hljs-function">() =&gt;</span> {
  cnt = (cnt + <span class="hljs-number">1</span>) % <span class="hljs-number">2</span>
  <span class="hljs-keyword">if</span>(msgNum===<span class="hljs-number">0</span>) {
    <span class="hljs-comment">// 通过DOM修改title</span>
    <span class="hljs-built_in">document</span>.title += <span class="hljs-string">`聊天页面`</span>
    <span class="hljs-built_in">clearInterval</span>(interval)
    <span class="hljs-keyword">return</span>
  }
  <span class="hljs-keyword">const</span> prefix = cnt % <span class="hljs-number">2</span> ? <span class="hljs-string">`新消息(<span class="hljs-subst">${msgNum}</span>)`</span> : <span class="hljs-string">''</span>
  <span class="hljs-built_in">document</span>.title = <span class="hljs-string">`<span class="hljs-subst">${prefix}</span>聊天页面`</span>
}, <span class="hljs-number">1000</span>)
</code></pre>
<p data-nodeid="1111">实现效果如下图所示，可以看到标签名称上有提示文字在闪烁。</p>
<p data-nodeid="1112"><img src="https://s0.lgstatic.com/i/image/M00/07/67/CgqCHl65LJGAR25PAAAXBLXRFXg133.gif" alt="categories.gif" data-nodeid="1222"></p>
<p data-nodeid="1113">通过模拟消息闪烁，可以让用户在浏览其他页面的时候，及时得知服务端返回的消息。</p>
<p data-nodeid="1114">定时修改 title 标签内容，除了用来实现闪烁效果之外，还可以制作其他动画效果，比如文字滚动，但需要注意浏览器会对 title 标签文本进行去空格操作。</p>
<p data-nodeid="1115">动态修改 title 标签的用途不仅在于消息提醒，你还可以将一些关键信息显示到标签上（比如下载时的进度、当前操作步骤），从而提升用户体验。</p>
<h3 data-nodeid="1116">性能优化</h3>
<p data-nodeid="1117">性能优化是前端开发中避不开的问题，性能问题无外乎两方面原因：渲染速度慢、请求时间长。性能优化虽然涉及很多复杂的原因和解决方案，但其实只要通过合理地使用标签，就可以在一定程度上提升渲染速度以及减少请求时间。</p>
<h4 data-nodeid="1118">script 标签：调整加载顺序提升渲染速度</h4>
<p data-nodeid="1119">由于浏览器的底层运行机制，渲染引擎在解析 HTML 时，若遇到 script 标签引用文件，则会暂停解析过程，同时通知网络线程加载文件，文件加载后会切换至 JavaScript 引擎来执行对应代码，代码执行完成之后切换至渲染引擎继续渲染页面。</p>
<p data-nodeid="1120">在这一过程中可以看到，页面渲染过程中包含了请求文件以及执行文件的时间，但页面的首次渲染可能并不依赖这些文件，这些请求和执行文件的动作反而延长了用户看到页面的时间，从而降低了用户体验。</p>
<p data-nodeid="1121">为了减少这些时间损耗，可以借助 script 标签的 3 个属性来实现。</p>
<ul data-nodeid="1122">
<li data-nodeid="1123">
<p data-nodeid="1124"><strong data-nodeid="1236">async 属性</strong>。立即请求文件，但不阻塞渲染引擎，而是文件加载完毕后阻塞渲染引擎并立即执行文件内容。</p>
</li>
<li data-nodeid="1125">
<p data-nodeid="1126"><strong data-nodeid="1241">defer 属性</strong>。立即请求文件，但不阻塞渲染引擎，等到解析完 HTML 之后再执行文件内容。</p>
</li>
<li data-nodeid="1127">
<p data-nodeid="1128"><strong data-nodeid="1246">HTML5 标准 type 属性</strong>，对应值为“module”。让浏览器按照 ECMA Script 6 标准将文件当作模块进行解析，默认阻塞效果同 defer，也可以配合 async 在请求完成后立即执行。</p>
</li>
</ul>
<p data-nodeid="1129">具体效果可以参看下图：</p>
<p data-nodeid="1130"><img src="https://s0.lgstatic.com/i/image/M00/07/0E/Ciqc1F647iiAZx3cAAB1ewBzlh0431.png" alt="1583465393011-810652f489ca6136.png" data-nodeid="1250"></p>
<p data-nodeid="1131">其中，绿色的线表示执行解析 HTML ，蓝色的线表示请求文件，红色的线表示执行文件。</p>
<p data-nodeid="1132">从图中可以得知，采用 3 种属性都能减少请求文件引起的阻塞时间，只有 defer 属性以及 type="module" 情况下能保证渲染引擎的优先执行，从而减少执行文件内容消耗的时间，让用户更快地看见页面（即使这些页面内容可能并没有完全地显示）。</p>
<p data-nodeid="1133">除此之外还应当注意，当渲染引擎解析 HTML 遇到 script 标签引入文件时，会立即进行一次渲染。所以这也就是为什么构建工具会把编译好的引用 JavaScript 代码的 script 标签放入到 body 标签底部，因为当渲染引擎执行到 body 底部时会先将已解析的内容渲染出来，然后再去请求相应的 JavaScript 文件。如果是内联脚本（即不通过 src 属性引用外部脚本文件直接在 HTML 编写 JavaScript 代码的形式），渲染引擎则不会渲染。</p>
<h4 data-nodeid="1134">link 标签：通过预处理提升渲染速度</h4>
<p data-nodeid="1135">在我们对大型单页应用进行性能优化时，也许会用到按需懒加载的方式，来加载对应的模块，但如果能合理利用 link 标签的 rel 属性值来进行预加载，就能进一步提升渲染速度。</p>
<ul data-nodeid="1136">
<li data-nodeid="1137">
<p data-nodeid="1138"><strong data-nodeid="1264">dns-prefetch</strong>。当 link 标签的 rel 属性值为“dns-prefetch”时，浏览器会对某个域名预先进行 DNS 解析并缓存。这样，当浏览器在请求同域名资源的时候，能省去从域名查询 IP 的过程，从而减少时间损耗。下图是淘宝网设置的 DNS 预解析。</p>
</li>
</ul>
<p data-nodeid="1139"><img src="https://s0.lgstatic.com/i/image/M00/07/0E/Ciqc1F647jWAHmc_AAAiNGoHmY8154.png" alt="1583466667742-993b502f80fa3567.png" data-nodeid="1267"></p>
<ul data-nodeid="1140">
<li data-nodeid="1141">
<p data-nodeid="1142"><strong data-nodeid="1272">preconnect</strong>。让浏览器在一个 HTTP 请求正式发给服务器前预先执行一些操作，这包括 DNS 解析、TLS 协商、TCP 握手，通过消除往返延迟来为用户节省时间。</p>
</li>
<li data-nodeid="1143">
<p data-nodeid="1144"><strong data-nodeid="1277">prefetch/preload</strong>。两个值都是让浏览器预先下载并缓存某个资源，但不同的是，prefetch 可能会在浏览器忙时被忽略，而 preload 则是一定会被预先下载。</p>
</li>
<li data-nodeid="1145">
<p data-nodeid="1146"><strong data-nodeid="1282">prerender</strong>。浏览器不仅会加载资源，还会解析执行页面，进行预渲染。</p>
</li>
</ul>
<p data-nodeid="1147">这几个属性值恰好反映了浏览器获取资源文件的过程，在这里我绘制了一个流程简图，方便你记忆。</p>
<p data-nodeid="1148"><img src="https://s0.lgstatic.com/i/image/M00/07/0E/Ciqc1F647j-AFiBtAABWh7ld3uA965.png" alt="1583470467405-1d2eb8baf7568d31.png" data-nodeid="1286"></p>
<p data-nodeid="1149">浏览器获取资源文件的流程</p>
<h3 data-nodeid="1150">搜索优化</h3>
<p data-nodeid="1151">你所写的前端代码，除了要让浏览器更好执行，有时候也要考虑更方便其他程序（如搜索引擎）理解。合理地使用 meta 标签和 link 标签，恰好能让搜索引擎更好地理解和收录我们的页面。</p>
<h4 data-nodeid="1152">meta 标签：提取关键信息</h4>
<p data-nodeid="1153">通过 meta 标签可以设置页面的描述信息，从而让搜索引擎更好地展示搜索结果。</p>
<p data-nodeid="1154">例如，在百度中搜索“拉勾”，就会发现网站的描述信息，这些描述信息就是通过 meta 标签专门为搜索引擎设置的，目的是方便用户预览搜索到的结果。</p>
<p data-nodeid="1155"><img src="https://s0.lgstatic.com/i/image/M00/07/0F/Ciqc1F647kmAMJF6AABXM1K7WdY483.png" alt="1583481178981-737e6b76d555f457.png" data-nodeid="1295"></p>
<p data-nodeid="1156">为了让搜索引擎更好地识别页面，除了描述信息之外还可以使用关键字，这样即使页面其他地方没有包含搜索内容，也可以被搜索到（当然搜索引擎有自己的权重和算法，如果滥用关键字是会被降权的，比如 Google 引擎就会对堆砌大量相同关键词的网页进行惩罚，降低它被搜索到的权重）。</p>
<p data-nodeid="1157">当我们搜索关键字“垂直互联网招聘”的时候搜索结果会显示拉勾网的信息，虽然显示的搜索内容上并没有看到“垂直互联网招聘”字样，这就是因为拉勾网页面中设置了这个关键字。</p>
<p data-nodeid="1158"><img src="https://s0.lgstatic.com/i/image/M00/07/0F/Ciqc1F647lSAGbePAAEeMKqCVgw178.png" alt="1583481543840-ce4f715602d1b084.png" data-nodeid="1300"></p>
<p data-nodeid="1159">对应代码如下：</p>
<pre class="lang-html" data-nodeid="1160"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">content</span>=<span class="hljs-string">"拉勾,拉勾网,拉勾招聘,拉钩, 拉钩网 ,互联网招聘,拉勾互联网招聘, 移动互联网招聘, 垂直互联网招聘, 微信招聘, 微博招聘, 拉勾官网, 拉勾百科,跳槽, 高薪职位, 互联网圈子, IT招聘, 职场招聘, 猎头招聘,O2O招聘, LBS招聘, 社交招聘, 校园招聘, 校招,社会招聘,社招"</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"keywords"</span>&gt;</span>
</code></pre>
<p data-nodeid="1161">在实际工作中，推荐使用一些关键字工具来挑选，比如 <a href="https://trends.google.com/trends" data-nodeid="1305">Google Trends</a>、<a href="https://data.chinaz.com/keyword/" data-nodeid="1309">站长工具</a>。下图是我使用站长工具搜索“招聘”关键字得到的结果，可以看到得到了相当关键的一些信息，比如全网搜索指数、关键词特点。</p>
<p data-nodeid="1162"><img src="https://s0.lgstatic.com/i/image/M00/07/0F/CgqCHl647l2Abd9XAAEL0O2drYw681.png" alt="image.png" data-nodeid="1313"></p>
<h4 data-nodeid="1163">link 标签：减少重复</h4>
<p data-nodeid="1164">有时候为了用户访问方便或者出于历史原因，对于同一个页面会有多个网址，又或者存在某些重定向页面，比如：</p>
<ul data-nodeid="9731">
<li data-nodeid="9732">
<p data-nodeid="9733"><a href="https://edu.lagou.com" data-nodeid="9738">https://lagou.com/a.html</a></p>
</li>
<li data-nodeid="9734">
<p data-nodeid="9735" class=""><a href="https://edu.lagou.com" data-nodeid="9741">https://lagou.com/detail?id=</a>"abcd"</p>
</li>
</ul>
















<p data-nodeid="1170">那么在这些页面中可以这样设置：</p>
<pre class="lang-html te-preview-highlight" data-nodeid="13559"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"https://lagou.com/a.html"</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">"canonical"</span>&gt;</span>
</code></pre>







<p data-nodeid="1172">这样可以让搜索引擎避免花费时间抓取重复网页。不过需要注意的是，它还有个限制条件，那就是指向的网站不允许跨域。</p>
<p data-nodeid="1173">当然，要合并网址还有其他的方式，比如使用站点地图，或者在 HTTP 请求响应头部添加 rel="canonical"。这里，我就不展开介绍了，道理都是相通的，你平时可以多探索和实践。</p>
<h3 data-nodeid="1174">延伸内容：OGP（开放图表协议）</h3>
<p data-nodeid="1175"><img src="https://s0.lgstatic.com/i/image/M00/07/68/Ciqc1F65LLeAZf33AAAvMXPozlk099.png" alt="1583483633365-97ac0eb2d1c6d7d2.png" data-nodeid="1335"></p>
<p data-nodeid="1176">好了，前面我们说了 HTML5 标准的一些标签和属性，下面再延伸说一说基于 meta 标签扩展属性值实现的第三方协议——OGP（Open Graph Protocal，开放图表协议 ）。</p>
<p data-nodeid="1177">OGP 是 Facebook 公司在 2010 年提出的，目的是通过增加文档信息来提升社交网页在被分享时的预览效果。你只需要在一些分享页面中添加一些 meta 标签及属性，支持 OGP 协议的社交网站就会在解析页面时生成丰富的预览信息，比如站点名称、网页作者、预览图片。具体预览效果会因各个网站而有所变化。</p>
<p data-nodeid="1178">下面是微信文章支持 OGP 协议的代码，可以看到通过 meta 标签属性值声明了：网址、预览图片、描述信息、站点名称、网页类型和作者信息。</p>
<pre data-nodeid="1179"><code>            ![1583480543843-477274458e5be00b.png](https://s0.lgstatic.com/i/image/M00/07/0F/CgqCHl647neAc1fJAACYggDXkeE601.png)
</code></pre>
<p data-nodeid="1180">现在百度已经宣布支持，微信文章的不少页面上也添加了相关标签属性，有兴趣的话你可以查看官方网站：<a href="https://ogp.me/" data-nodeid="1342">https://ogp.me/</a>。</p>
<h3 data-nodeid="1181">总结</h3>
<p data-nodeid="1182">本课时，我从交互实现、性能优化、搜索优化场景出发，分别讲解了 meta 标签、title 标签、link 标签，以及 script 标签在这些场景中的重要作用，希望这些内容你都能有效地应用到工作场景中，不再只是了解，而是能够熟练运用。</p>
<p data-nodeid="1183">最后布置一道思考题：说一说你还知道哪些“看不见”的标签及用法？</p>
<h3 data-nodeid="1184">社群福利</h3>
<p data-nodeid="1185">5月20日前，加入社群获得以下福利：</p>
<p data-nodeid="1186">① 讲师交流： 订阅后可加入「 前端进阶交流群 」与讲师直面交流<br>
② 独家资料： 进群后领取 「 前端高手进阶课程PPT」<br>
③ 加餐内容： 社群专属直播 「2020 大厂青睐的前端必备技能」</p>
<p data-nodeid="1187" class="">——&gt; <a href="https://jinshuju.net/f/4nFzjm" data-nodeid="1357">仅限 500 人，戳此加入</a> &lt;——</p>

---

### 精选评论

##### **彬：
> 开头的meta标签的跳转和刷新，我感觉现在用得少的原因还有一个，目前单页面应用是主流，所以也用不上，但是我也确实不知道这个知识点😍

##### *栋：
> 如果我们文档都是类似这样的讲解，很多用法就很清晰明了

##### **生：
> 牛逼，满满的干货！虽然开发了几年了，但是这些知识点确实没有研究过，很重要的基础 ！

##### **鹏：
> 打卡第一天，决定走出自己的舒适圈

##### *琴：
> 除了OPG其他的都有了解过到讲不出来。看不见的标签还有一些为特殊人群设置的，如img的alt属性，input的aria-label属性

##### **上的垂耳兔：
> 打卡第一天学习，感觉又重新认识了这些标签

##### **月：
> 打卡

##### Cobra：
> 打卡😋

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油

##### HopEvan：
> 这里涉及一些seo的知识，可以从中开启对这块学习的兴趣

##### *栋：
> 打卡

##### **生：
> 打卡

##### **喝王老吉：
> 打卡

##### **卓：
> 打卡

##### *召：
> 学习到的前端场景：交互实现、性能优化、搜索优化

##### Telstra4375：
> 打卡

##### **华：
> 满满的干活，收益～

##### **洋：
> 打卡！

##### **逸：
> 打卡0724

##### **贤：
> 打卡

##### *方：
> 很深入探究，告别过去浅尝辄止的学习方式

##### **龙：
> 打卡

##### **4658：
> 打卡

##### **斌：
> 打卡<div><br></div>

##### **飞：
> 不错 很有用

##### **工程师-2年-胡琦：
> 打卡

##### **伟：
> 打卡

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油哦

##### **翀：
> 很棒，学到了，平时很少用

##### **8865：
> 学习了

##### *宁：
> 谢谢

