<p data-nodeid="1189" class="">React-Router 是 React 场景下的路由解决方案，本讲我们将学习 React-Router 的实现机制，并基于此提取和探讨通用的前端路由解决方案。</p>
<blockquote data-nodeid="1190">
<p data-nodeid="1191">注：没有使用过 React-Router 的同学，可以点击<a href="https://reactrouter.com/web/guides/quick-start" data-nodeid="1306">这里</a>完成快速上手。</p>
</blockquote>
<h3 data-nodeid="1192">认识 React-Router</h3>
<p data-nodeid="1193">本着尽快进入主题的原则，这里我用一个尽可能简单的 Demo 作为引子，帮助你快速地把握 React-Router 的核心功能。请看下面代码（解析在注释里）：</p>
<pre class="lang-js" data-nodeid="1194"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-comment">// 引入 React-Router 中的相关组件</span>
<span class="hljs-keyword">import</span> { BrowserRouter <span class="hljs-keyword">as</span> Router, Route, Link } <span class="hljs-keyword">from</span> <span class="hljs-string">"react-router-dom"</span>;
<span class="hljs-comment">// 导出目标组件</span>
<span class="hljs-keyword">const</span> BasicExample = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> (
  <span class="hljs-comment">// 组件最外层用 Router 包裹</span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">Router</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
          // 具体的标签用 Link 包裹
          <span class="hljs-tag">&lt;<span class="hljs-name">Link</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"/"</span>&gt;</span>Home<span class="hljs-tag">&lt;/<span class="hljs-name">Link</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
          // 具体的标签用 Link 包裹
          <span class="hljs-tag">&lt;<span class="hljs-name">Link</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"/about"</span>&gt;</span>About<span class="hljs-tag">&lt;/<span class="hljs-name">Link</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
          // 具体的标签用 Link 包裹
          <span class="hljs-tag">&lt;<span class="hljs-name">Link</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"/dashboard"</span>&gt;</span>Dashboard<span class="hljs-tag">&lt;/<span class="hljs-name">Link</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">hr</span> /&gt;</span>

      // Route 是用于声明路由映射到应用程序的组件层
      <span class="hljs-tag">&lt;<span class="hljs-name">Route</span> <span class="hljs-attr">exact</span> <span class="hljs-attr">path</span>=<span class="hljs-string">"/"</span> <span class="hljs-attr">component</span>=<span class="hljs-string">{Home}</span> /&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">Route</span> <span class="hljs-attr">path</span>=<span class="hljs-string">"/about"</span> <span class="hljs-attr">component</span>=<span class="hljs-string">{About}</span> /&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">Route</span> <span class="hljs-attr">path</span>=<span class="hljs-string">"/dashboard"</span> <span class="hljs-attr">component</span>=<span class="hljs-string">{Dashboard}</span> /&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">Router</span>&gt;</span></span>
);
<span class="hljs-comment">// Home 组件的定义</span>
<span class="hljs-keyword">const</span> Home = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> (
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>Home<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
);
<span class="hljs-comment">// About 组件的定义</span>
<span class="hljs-keyword">const</span> About = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> (
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>About<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
);
<span class="hljs-comment">// Dashboard 的定义</span>
<span class="hljs-keyword">const</span> Dashboard = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> (
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>Dashboard<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
);

<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> BasicExample;
</code></pre>
<p data-nodeid="1195">这个 Demo 渲染出的页面效果如下图所示：</p>
<p data-nodeid="1196"><img src="https://s0.lgstatic.com/i/image/M00/8B/20/CgqCHl_bOA2AfagHAAAwif17aiI096.png" alt="Drawing 0.png" data-nodeid="1313"></p>
<p data-nodeid="1197">当我点击不同的链接时，ul 元素内部就会展示不同的组件内容。比如当我点击“About”链接时，就会展示 About 组件的内容，效果如下图所示：</p>
<p data-nodeid="1198"><img src="https://s0.lgstatic.com/i/image2/M01/02/F3/CgpVE1_bOBOAQTs_AAAuMzu6RbY087.png" alt="Drawing 1.png" data-nodeid="1317"></p>
<p data-nodeid="1199">注意，点击 About 后，界面中发生变化的地方有两处（见下图标红处），除了 ul 元素的内容改变了之外，路由信息也改变了。</p>
<p data-nodeid="1200"><img src="https://s0.lgstatic.com/i/image2/M01/02/F2/Cip5yF_bOBmAfCOxAAA2a1pswj4899.png" alt="Drawing 2.png" data-nodeid="1321"></p>
<p data-nodeid="1201">在 React-Router 中，各种细碎的功能点有不少，但作为 React 框架的前端路由解决方案，它最基本也是最核心的能力，其实正是你刚刚所见到的这一幕——<strong data-nodeid="1327">路由的跳转</strong>。这也是我们接下来讨论的重点。</p>
<p data-nodeid="1202">接下来我们就结合 React-Router 的源码，一起来看看“跳转”这个动作是如何实现的。</p>
<h3 data-nodeid="1203">React-Router 是如何实现路由跳转的？</h3>
<p data-nodeid="1204">首先需要回顾下 Demo 中的第一行代码：</p>
<pre class="lang-java" data-nodeid="1205"><code data-language="java"><span class="hljs-keyword">import</span> { BrowserRouter as Router, Route, Link } from <span class="hljs-string">"react-router-dom"</span>;
</code></pre>
<p data-nodeid="1206">这行代码告诉我们，为了实现一个简单的路由跳转效果，一共从 React-Router 中引入了以下 3 个组件：</p>
<ul data-nodeid="1207">
<li data-nodeid="1208">
<p data-nodeid="1209">BrowserRouter</p>
</li>
<li data-nodeid="1210">
<p data-nodeid="1211">Route</p>
</li>
<li data-nodeid="1212">
<p data-nodeid="1213">Link</p>
</li>
</ul>
<p data-nodeid="1214">这 3 个组件也就代表了 React-Router 中的 3 个核心角色：</p>
<ul data-nodeid="1215">
<li data-nodeid="1216">
<p data-nodeid="1217"><strong data-nodeid="1340">路由器</strong>，比如 BrowserRouter 和 HashRouter</p>
</li>
<li data-nodeid="1218">
<p data-nodeid="1219"><strong data-nodeid="1345">路由</strong>，比如 Route 和 Switch</p>
</li>
<li data-nodeid="1220">
<p data-nodeid="1221"><strong data-nodeid="1350">导航</strong>，比如 Link、NavLink、Redirect</p>
</li>
</ul>
<p data-nodeid="1222">路由（以 Route 为代表）负责定义路径与组件之间的映射关系，而导航（以 Link 为代表）负责触发路径的改变，路由器（包括 BrowserRouter 和 HashRouter）则会根据 Route 定义出来的映射关系，为新的路径匹配它对应的逻辑。</p>
<p data-nodeid="1223">以上便是 3 个角色“打配合”的过程。这其中，最需要你注意的是路由器这个角色，React Router 曾在说明文档中官宣它是“React Router 应用程序的核心”。因此学习 React Router，最要紧的是搞明白路由器的工作机制。</p>
<h4 data-nodeid="1224">路由器：BrowserRouter 和 HashRouter</h4>
<p data-nodeid="1225"><strong data-nodeid="1358">路由器负责感知路由的变化并作出反应，它是整个路由系统中最为重要的一环</strong>。React-Router 支持我们使用 hash（对应 HashRouter）和 browser（对应 BrowserRouter） 两种路由规则，这里我们把两种规则都讲一下。</p>
<p data-nodeid="1226">HashRouter、BrowserRouter，这俩人名字这么像，该不会底层逻辑区别也不大吧？别说，还真是如此。我们首先来瞟一眼 HashRouter 的源码：</p>
<p data-nodeid="1227"><img src="https://s0.lgstatic.com/i/image2/M01/02/F2/Cip5yF_bOCSAErIlAAEU7gTEf-c538.png" alt="Drawing 3.png" data-nodeid="1362"></p>
<p data-nodeid="1228">再瞟一眼 BrowserRouter 的源码：</p>
<p data-nodeid="1229"><img src="https://s0.lgstatic.com/i/image/M00/8B/15/Ciqc1F_bOCyAFB9oAADQnB3x2AY718.png" alt="Drawing 4.png" data-nodeid="1366"></p>
<p data-nodeid="1230">我们会发现这两个文件惊人的相似，而最关键的区别我也已经在图中分别标出，即它们调用的 history 实例化方法不同：HashRouter 调用了 <a href="https://github.com/ReactTraining/history/blob/v4.7.2/modules/createHashHistory.js" data-nodeid="1370">createHashHistory</a>，BrowserRouter 调用了 <a href="https://github.com/ReactTraining/history/blob/v4.7.2/modules/createBrowserHistory.js" data-nodeid="1374">createBrowserHistory</a>。</p>
<p data-nodeid="1231">这两个 history 的实例化方法均来源于 <a href="https://github.com/ReactTraining/history" data-nodeid="1379">history</a> 这个独立的代码库，关于它的实现细节，你倒不必纠结。对于 <a href="https://github.com/ReactTraining/history/blob/v4.7.2/modules/createHashHistory.js" data-nodeid="1383">createHashHistory</a> 和 <a href="https://github.com/ReactTraining/history/blob/v4.7.2/modules/createBrowserHistory.js" data-nodeid="1387">createBrowserHistory</a> 这两个 API，我们最要紧的是掌握它们各自的特征。</p>
<ul data-nodeid="2070">
<li data-nodeid="2071">
<p data-nodeid="2072"><code data-backticks="1" data-nodeid="2074">createBrowserHistory</code>：它将在浏览器中使用 <a href="https://developer.mozilla.org/zh-CN/docs/Web/API/History" data-nodeid="2078">HTML5 history API</a> 来处理 URL（见下图标红处的说明），它能够处理形如这样的 URL，example.com/some/path。由此可得，<strong data-nodeid="2083">BrowserRouter 是使用 HTML 5 的 history API 来控制路由跳转的。</strong></p>
</li>
</ul>
<p data-nodeid="2073" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/78/Cip5yF_cKqmATWf8AAN3hb7gX3k493.png" alt="图片1.png" data-nodeid="2086"></p>


<ul data-nodeid="1236">
<li data-nodeid="1237">
<p data-nodeid="1238" class="te-preview-highlight"><code data-backticks="1" data-nodeid="1402">createHashHistory</code>：它是使用 hash tag (#) 处理 URL 的方法，能够处理形如这样的 URL，example.com/#/some/path。我们可以看到<a href="https://github.com/ReactTraining/history/blob/v4.7.2/modules/createHashHistory.js" data-nodeid="1406">它的源码中</a>对各种方法的定义基本都围绕 hash 展开（如下图所示），由此可得，<strong data-nodeid="1412">HashRouter 是通过 URL 的 hash 属性来控制路由跳转的</strong>。</p>
</li>
</ul>
<p data-nodeid="1239"><img src="https://s0.lgstatic.com/i/image/M00/8B/15/Ciqc1F_bOFaAWDI8AAD_FnBQsTc850.png" alt="Drawing 6.png" data-nodeid="1415"></p>
<blockquote data-nodeid="1240">
<p data-nodeid="1241">注：关于 hash 和 history 这两种模式，我们在下文中还会持续探讨。</p>
</blockquote>
<p data-nodeid="1242">现在，见识了表面现象，了解了背后机制。我们不妨回到故事的原点，再多问自己一个问题：为什么我们需要 React-Router？</p>
<p data-nodeid="1243">或者把这个问题稍微拔高一点：<strong data-nodeid="1423">为什么我们需要前端路由</strong>？</p>
<p data-nodeid="1244">这一切的一切，都要从很久以前说起。</p>
<h3 data-nodeid="1245">理解前端路由——是什么？解决什么问题？</h3>
<h4 data-nodeid="1246">背景——问题的产生</h4>
<p data-nodeid="1247">在前端技术早期，一个 URL 对应一个页面，如果你要从 A 页面切换到 B 页面，那么必然伴随着页面的刷新。这个体验并不好，不过在最初也是无奈之举——毕竟用户只有在刷新页面的情况下，才可以重新去请求数据。</p>
<p data-nodeid="1248">后来，改变发生了——Ajax 出现了，它允许人们在不刷新页面的情况下发起请求；与之共生的，还有“不刷新页面即可更新页面内容”这种需求。在这样的背景下，出现了<strong data-nodeid="1433">SPA（单页面应用</strong>）。</p>
<p data-nodeid="1249">SPA 极大地提升了用户体验，它允许页面在不刷新的情况下更新页面内容，使内容的切换更加流畅。但是在 SPA 诞生之初，人们并没有考虑到“定位”这个问题——在内容切换前后，页面的 URL 都是一样的，这就带来了两个问题：</p>
<ul data-nodeid="1250">
<li data-nodeid="1251">
<p data-nodeid="1252">SPA 其实并不知道当前的页面“进展到了哪一步”，可能你在一个站点下经过了反复的“前进”才终于唤出了某一块内容，但是此时只要刷新一下页面，一切就会被清零，你必须重复之前的操作才可以重新对内容进行定位——SPA 并不会“记住”你的操作；</p>
</li>
<li data-nodeid="1253">
<p data-nodeid="1254">由于有且仅有一个 URL 给页面做映射，这对 SEO 也不够友好，搜索引擎无法收集全面的信息。</p>
</li>
</ul>
<p data-nodeid="1255">为了解决这个问题，前端路由出现了。</p>
<h4 data-nodeid="1256">前端路由——SPA“定位”解决方案</h4>
<p data-nodeid="1257">前端路由可以帮助我们在仅有一个页面的情况下，“记住”用户当前走到了哪一步——为 SPA 中的各个视图匹配一个唯一标识。这意味着用户前进、后退触发的新内容，都会映射到不同的 URL 上去。此时即便他刷新页面，因为当前的 URL 可以标识出他所处的位置，因此内容也不会丢失。</p>
<p data-nodeid="1258">那么如何实现这个目的呢？首先我们要解决以下两个问题。</p>
<ul data-nodeid="1259">
<li data-nodeid="1260">
<p data-nodeid="1261">当用户刷新页面时，浏览器会默认根据当前 URL 对资源进行重新定位（发送请求）。这个动作对 SPA 是不必要的，因为 SPA 作为单页面，无论如何也只会有一个资源与之对应。此时若走正常的请求-刷新流程，反而会使用户的前进后退操作无法被记录。</p>
</li>
<li data-nodeid="1262">
<p data-nodeid="1263">单页面应用对服务端来说，就是一个 URL、一套资源，那么如何做到用“不同的 URL”来映射不同的视图内容呢？</p>
</li>
</ul>
<p data-nodeid="1264">从这两个问题来看，服务端已经救不了 SPA 这个场景了。所以要靠咱们前端自力更生，不然怎么叫“前端路由”呢？作为前端，我们可以提供以下这样的解决思路。</p>
<ul data-nodeid="1265">
<li data-nodeid="1266">
<p data-nodeid="1267"><strong data-nodeid="1448">拦截用户的刷新操作，避免服务端盲目响应、返回不符合预期的资源内容</strong>，把刷新这个动作完全放到前端逻辑里消化掉；</p>
</li>
<li data-nodeid="1268">
<p data-nodeid="1269"><strong data-nodeid="1453">感知 URL 的变化</strong>。这里不是说要改造 URL、凭空制造出 N 个 URL 来。而是说 URL 还是那个 URL，只不过我们可以给它做一些微小的处理，这些处理并不会影响 URL 本身的性质，不会影响服务器对它的识别，只有我们前端能感知到。一旦我们感知到了，我们就根据这些变化、用 JS 去给它生成不同的内容。</p>
</li>
</ul>
<h3 data-nodeid="1270">实践思路——hash 与 history</h3>
<p data-nodeid="1271">接下来重点就来了，现在前端界对前端路由有哪些实现思路？这里需要掌握的两个实践就是 hash 与 history。</p>
<h4 data-nodeid="1272">hash 模式</h4>
<p data-nodeid="1273">hash 模式是指通过改变 URL 后面以“#”分隔的字符串（这货其实就是 URL 上的哈希值），从而让页面感知到路由变化的一种实现方式。举个例子，比如这样的一个 URL：</p>
<pre class="lang-java" data-nodeid="1274"><code data-language="java">https:<span class="hljs-comment">//www.imooc.com/</span>
</code></pre>
<p data-nodeid="1275">我就可以通过增加和改变哈希值，来让这个 URL 变得有那么一点点不一样：</p>
<pre class="lang-java" data-nodeid="1276"><code data-language="java"><span class="hljs-comment">// 主页</span>
https:<span class="hljs-comment">//www.imooc.com/#index</span>
<span class="hljs-comment">// 活动页</span>
https:<span class="hljs-comment">//www.imooc.com/#activePage</span>
</code></pre>
<p data-nodeid="1277">这个“不一样”是前端完全可感知的——JS 可以帮我们捕获到哈希值的内容。在 hash 模式下，我们实现路由的思路可以概括如下：</p>
<p data-nodeid="1278">（1）hash 的改变：我们可以通过 location 暴露出来的属性，直接去修改当前 URL 的 hash 值：</p>
<pre class="lang-java" data-nodeid="1279"><code data-language="java">window.location.hash = <span class="hljs-string">'index'</span>;
</code></pre>
<p data-nodeid="1280">（2）hash 的感知：通过监听 “hashchange”事件，可以用 JS 来捕捉 hash 值的变化，进而决定我们页面内容是否需要更新：</p>
<pre class="lang-java" data-nodeid="1281"><code data-language="java"><span class="hljs-comment">// 监听hash变化，点击浏览器的前进后退会触发</span>
window.addEventListener(<span class="hljs-string">'hashchange'</span>, function(event){ 
    <span class="hljs-comment">// 根据 hash 的变化更新内容</span>
},<span class="hljs-keyword">false</span>)
</code></pre>
<h4 data-nodeid="1282">history 模式</h4>
<p data-nodeid="1283">大家知道，在我们浏览器的左上角，往往有这样的操作点：</p>
<p data-nodeid="1284"><img src="https://s0.lgstatic.com/i/image/M00/8B/20/CgqCHl_bOGeAQW2AAACAaTKsTyM327.png" alt="Drawing 7.png" data-nodeid="1466"></p>
<p data-nodeid="1285">通过点击前进后退箭头，就可以实现页面间的跳转。这样的行为，其实是可以通过 API 来实现的。</p>
<p data-nodeid="1286">浏览器的 history API 赋予了我们这样的能力，在 HTML 4 时，就可以通过下面的接口来操作浏览历史、实现跳转动作：</p>
<pre class="lang-java" data-nodeid="1287"><code data-language="java">window.history.forward()  <span class="hljs-comment">// 前进到下一页</span>
</code></pre>
<pre class="lang-java" data-nodeid="1288"><code data-language="java">window.history.back() <span class="hljs-comment">// 后退到上一页</span>
</code></pre>
<pre class="lang-java" data-nodeid="1289"><code data-language="java">window.history.go(<span class="hljs-number">2</span>) <span class="hljs-comment">// 前进两页</span>
</code></pre>
<pre class="lang-java" data-nodeid="1290"><code data-language="java">window.history.go(-<span class="hljs-number">2</span>) <span class="hljs-comment">// 后退两页</span>
</code></pre>
<p data-nodeid="1291">很有趣吧？遗憾的是，在这个阶段，我们能做的只是“切换”，而不能“改变”。好在从 HTML 5 开始，浏览器支持了 pushState 和 replaceState 两个 API，允许我们对浏览历史进行修改和新增：</p>
<pre class="lang-java" data-nodeid="1292"><code data-language="java">history.pushState(data[,title][,url]); <span class="hljs-comment">// 向浏览历史中追加一条记录</span>
</code></pre>
<pre class="lang-java" data-nodeid="1293"><code data-language="java">history.replaceState(data[,title][,url]); <span class="hljs-comment">// 修改（替换）当前页在浏览历史中的信息</span>
</code></pre>
<p data-nodeid="1294">这样一来，修改动作就齐活了。</p>
<p data-nodeid="1295">有修改，就要有对修改的感知能力。在 history 模式下，我们可以通过监听 popstate 事件来达到我们的目的：</p>
<pre class="lang-java" data-nodeid="1296"><code data-language="java">window.addEventListener(<span class="hljs-string">'popstate'</span>, function(e) {
  console.log(e)
});
</code></pre>
<p data-nodeid="1297">每当浏览历史发生变化，popstate 事件都会被触发。</p>
<p data-nodeid="1298"><strong data-nodeid="1481">注</strong>：go、forward 和 back 等方法的调用确实会触发 popstate，但是<strong data-nodeid="1482">pushState 和 replaceState 不会</strong>。不过这一点问题不大，我们可以通过自定义事件和全局事件总线来手动触发事件。</p>
<h3 data-nodeid="1299">总结</h3>
<p data-nodeid="1300">本讲我们以 React-Router 为切入点，结合源码剖析了 React-Router 中“跳转”这一动作的实现原理，由此牵出了针对“前端路由方案”这个知识点相对系统的探讨。行文至此，React 周边生态所涉及的重难点知识，相信已经深深地烙印在了你的脑海里。</p>
<p data-nodeid="1301" class="">下一讲开始，我们将围绕“React 设计模式与最佳实践”以及“React 性能优化”两条主线展开学习。彼时，站在“生产实践”这个全新的视角去认识 React 后，相信各位对它的理解定会更上一层楼。大家加油！</p>

---

### 精选评论

##### **威：
> 快看完了，意犹未尽😀

##### console_man：
> 比心修言

