<p data-nodeid="45625" class="">对于前端开发者来说，不管是对初学者还是已独当一面的资深前端开发者，HTML 都是最基础的内容。</p>
<p data-nodeid="45626">今天，我主要介绍 HTML 和网页有什么关系，以及与 DOM 有什么不同。通过本讲内容，你将掌握浏览器是怎么处理 HTML 内容的，以及在这个过程中我们可以进行怎样的处理来提升网页的性能，从而提升用户的体验。</p>
<h3 data-nodeid="45627">浏览器页面加载过程</h3>
<p data-nodeid="45628">不知你是否有过这样的体验：当打开某个浏览器的时候，发现一直在转圈，或者等了好长时间才打开页面……</p>
<p data-nodeid="45629">此时的你，会选择关掉页面还是耐心等待呢？</p>
<p data-nodeid="45630">这一现象，除了网络不稳定、网速过慢等原因，大多数都是由于页面设计不合理导致加载时间过长导致的。</p>
<p data-nodeid="45631">我们都知道，页面是用 HTML/CSS/JavaScript 来编写的。</p>
<blockquote data-nodeid="45632">
<p data-nodeid="45633">其中，HTML 的职责在于告知浏览器如何组织页面，以及搭建页面的基本结构；<br>
CSS 用来装饰 HTML，让我们的页面更好看；<br>
JavaScript 则可以丰富页面功能，使静态页面动起来。</p>
</blockquote>
<p data-nodeid="45634">HTML由一系列的元素组成，通常称为HTML元素。HTML 元素通常被用来定义一个网页结构，基本上所有网页都是这样的 HTML 结构：</p>
<pre class="lang-plain" data-nodeid="45635"><code data-language="plain">&lt;html&gt;
  &lt;head&gt;&lt;/head&gt;
  &lt;body&gt;&lt;/body&gt;
&lt;/html&gt;
</code></pre>
<p data-nodeid="45636">其中：</p>
<ul data-nodeid="45637">
<li data-nodeid="45638">
<p data-nodeid="45639"><code data-backticks="1" data-nodeid="45747">&lt;html&gt;</code>元素是页面的根元素，它描述完整的网页；</p>
</li>
<li data-nodeid="45640">
<p data-nodeid="45641"><code data-backticks="1" data-nodeid="45749">&lt;head&gt;</code>元素包含了我们想包含在 HTML 页面中，但不希望显示在网页里的内容；</p>
</li>
<li data-nodeid="45642">
<p data-nodeid="45643"><code data-backticks="1" data-nodeid="45751">&lt;body&gt;</code>元素包含了我们访问页面时所有显示在页面上的内容，是用户最终能看到的内容。</p>
</li>
</ul>
<p data-nodeid="45644">HTML 中的元素特别多，其中还包括可用于 Web Components 的自定义元素。</p>
<p data-nodeid="45645">前面我们提到<strong data-nodeid="45762">页面 HTML 结构不合理可能会导致页面响应慢，这个过程很多时候体现在</strong><code data-backticks="1" data-nodeid="45758">&lt;script&gt;</code>和<code data-backticks="1" data-nodeid="45760">&lt;style&gt;</code>元素的设计上，它们会影响页面加载过程中对 Javascript 和 CSS 代码的处理。</p>
<p data-nodeid="45646">因此，如果想要提升页面的加载速度，就需要了解浏览器页面的加载过程是怎样的，从根本上来解决问题。</p>
<p data-nodeid="45647">浏览器在加载页面的时候会用到 GUI 渲染线程和 JavaScript 引擎线程（更详细的浏览器加载和渲染机制将在第 7 讲中介绍）。其中，GUI 渲染线程负责渲染浏览器界面 HTML 元素，JavaScript 引擎线程主要负责处理 JavaScript 脚本程序。</p>
<p data-nodeid="45648">由于 JavaScript 在执行过程中还可能会改动界面结构和样式，因此它们之间被设计为互斥的关系。也就是说，当 JavaScript 引擎执行时，GUI 线程会被挂起。</p>
<p data-nodeid="45649">以<a href="https://kaiwu.lagou.com?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="45769">拉勾官网</a>为例，我们来看看网页加载流程。</p>
<p data-nodeid="45650">（1）当我们打开<a href="https://kaiwu.lagou.com?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="45774">拉勾官网</a>的时候，浏览器会从服务器中获取到 HTML 内容。</p>
<p data-nodeid="45651">（2）浏览器获取到 HTML 内容后，就开始从上到下解析 HTML 的元素。</p>
<p data-nodeid="45652"><img src="https://s0.lgstatic.com/i/image6/M01/33/EB/Cgp9HWBv-yOAFR4QAAHJHfvWnpQ926.png" alt="Drawing 0.png" data-nodeid="45779"></p>
<div data-nodeid="45653"><p style="text-align:center">从上到下解析 HTML 元素图</p></div>
<p data-nodeid="45654">（3）<code data-backticks="1" data-nodeid="45781">&lt;head&gt;</code>元素内容会先被解析，此时浏览器还没开始渲染页面。</p>
<blockquote data-nodeid="45655">
<p data-nodeid="45656">我们看到<code data-backticks="1" data-nodeid="45784">&lt;head&gt;</code>元素里有用于描述页面元数据的<code data-backticks="1" data-nodeid="45786">&lt;meta&gt;</code>元素，还有一些<code data-backticks="1" data-nodeid="45788">&lt;link&gt;</code>元素涉及外部资源（如图片、CSS 样式等），此时浏览器会去获取这些外部资源。<br>
除此之外，我们还能看到<code data-backticks="1" data-nodeid="45792">&lt;head&gt;</code>元素中还包含着不少的<code data-backticks="1" data-nodeid="45794">&lt;script&gt;</code>元素，这些<code data-backticks="1" data-nodeid="45796">&lt;script&gt;</code>元素通过<code data-backticks="1" data-nodeid="45798">src</code>属性指向外部资源。</p>
</blockquote>
<p data-nodeid="45657">（4）当浏览器解析到这里时（步骤 3），会暂停解析并下载 JavaScript 脚本。</p>
<p data-nodeid="45658">（5）当 JavaScript 脚本下载完成后，浏览器的控制权转交给 JavaScript 引擎。当脚本执行完成后，控制权会交回给渲染引擎，渲染引擎继续往下解析 HTML 页面。</p>
<p data-nodeid="45659">（6）此时<code data-backticks="1" data-nodeid="45803">&lt;body&gt;</code>元素内容开始被解析，浏览器开始渲染页面。</p>
<p data-nodeid="45660">在这个过程中，我们看到<code data-backticks="1" data-nodeid="45806">&lt;head&gt;</code>中放置的<code data-backticks="1" data-nodeid="45808">&lt;script&gt;</code>元素会阻塞页面的渲染过程：把 JavaScript 放在<code data-backticks="1" data-nodeid="45810">&lt;head&gt;</code>里，意味着必须把所有 JavaScript 代码都下载、解析和解释完成后，才能开始渲染页面。</p>
<p data-nodeid="45661">到这里，我们就明白了：<strong data-nodeid="45817">如果外部脚本加载时间很长（比如一直无法完成下载），就会造成网页长时间失去响应，浏览器就会呈现“假死”状态，用户体验会变得很糟糕</strong>。</p>
<p data-nodeid="45662">因此，对于对性能要求较高、需要快速将内容呈现给用户的网页，常常会将 JavaScript 脚本放在<code data-backticks="1" data-nodeid="45819">&lt;body&gt;</code>的最后面。这样可以避免资源阻塞，页面得以迅速展示。我们还可以使用<code data-backticks="1" data-nodeid="45821">defer</code>/<code data-backticks="1" data-nodeid="45823">async</code>/<code data-backticks="1" data-nodeid="45825">preload</code>等属性来标记<code data-backticks="1" data-nodeid="45827">&lt;script&gt;</code>标签，来控制 JavaScript 的加载顺序。</p>
<p data-nodeid="45663">我们再来看看百度首页。</p>
<p data-nodeid="45664"><img src="https://s0.lgstatic.com/i/image6/M01/33/EB/Cgp9HWBv-y6AXTsoAAFUrHlJt_M137.png" alt="Drawing 1.png" data-nodeid="45832"></p>
<div data-nodeid="45665"><p style="text-align:center">百度首页 HTML 元素图</p></div>
<p data-nodeid="45666">可以看到，虽然百度首页的<code data-backticks="1" data-nodeid="45834">&lt;head&gt;</code>元素里也包括了一些<code data-backticks="1" data-nodeid="45836">&lt;script&gt;</code>元素，但大多数都加上了<code data-backticks="1" data-nodeid="45838">async</code>属性。<code data-backticks="1" data-nodeid="45840">async</code>属性会让这些脚本并行进行请求获取资源，同时当资源获取完成后尽快解析和执行，这个过程是异步的，不会阻塞 HTML 的解析和渲染。</p>
<p data-nodeid="45667">对于百度这样的搜索引擎来说，必须要在最短的时间内提供到可用的服务给用户，其中就包括搜索框的显示及可交互，除此之外的内容优先级会相对较低。</p>
<p data-nodeid="45668">浏览器在渲染页面的过程需要解析 HTML、CSS 以得到 DOM 树和 CSS 规则树，它们结合后才生成最终的渲染树并渲染。因此，我们还常常将 CSS 放在<code data-backticks="1" data-nodeid="45844">&lt;head&gt;</code>里，可用来避免浏览器渲染的重复计算。</p>
<h3 data-nodeid="45669">HTML 与 DOM 有什么不同</h3>
<p data-nodeid="45670">我们知道<code data-backticks="1" data-nodeid="45848">&lt;p&gt;</code>是 HTML 元素，但又常常将<code data-backticks="1" data-nodeid="45850">&lt;p&gt;</code>这样一个元素称为 DOM 节点，那么 HTML 和 DOM 到底有什么不一样呢？</p>
<p data-nodeid="45671">根据 <a href="https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="45855">MDN</a> 官方描述：文档对象模型（DOM）是 HTML 和 XML 文档的编程接口。</p>
<p data-nodeid="45672">也就是说，DOM 是用来操作和描述 HTML 文档的接口。<strong data-nodeid="45862">如果说浏览器用 HTML 来描述网页的结构并渲染，那么使用 DOM 则可以获取网页的结构并进行操作</strong>。一般来说，我们使用 JavaScript 来操作 DOM 接口，从而实现页面的动态变化，以及用户的交互操作。</p>
<p data-nodeid="45673">在开发过程中，常常用对象的方式来描述某一类事物，用特定的结构集合来描述某些事物的集合。DOM 也一样，它将 HTML 文档解析成一个由 DOM 节点以及包含属性和方法的相关对象组成的结构集合。</p>
<p data-nodeid="45674">比如这里，我们在拉勾官网中检查滚动控制面板的元素，如下图所示：</p>
<p data-nodeid="45675"><img src="https://s0.lgstatic.com/i/image6/M00/34/25/CioPOWBwLXSAEWhbAAPaCqoG0Vk876.png" alt="图片3.png" data-nodeid="45867"></p>
<div data-nodeid="45676"><p style="text-align:center">控制台元素检查示意图</p></div>
<p data-nodeid="45677">可以在控制台中获取到该滚动控制面板对应的 DOM 节点，通过右键保存到临时变量后，便可以在 console 面板中通过 DOM 接口获取该节点的信息，或者进行一些修改节点的操作，如下图所示：</p>
<p data-nodeid="45678"><img src="https://s0.lgstatic.com/i/image6/M00/34/25/CioPOWBwLXuABaDNAAJTwBwHDRw721.png" alt="图片4.png" data-nodeid="45871"></p>
<div data-nodeid="45679"><p style="text-align:center">控制台 DOM 对象操作示意图</p></div>
<p data-nodeid="45680">我们来看看，浏览器中的 HTML 是怎样被解析成 DOM 的。</p>
<h4 data-nodeid="45681">DOM 解析</h4>
<p data-nodeid="45682">我们常见的 HTML 元素，在浏览器中会被解析成节点。比如下面这样的 HTML 内容：</p>
<pre class="lang-xml" data-nodeid="45683"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>文档标题<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"xx.com/xx"</span>&gt;</span>我的链接<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>我的标题<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</code></pre>
<p data-nodeid="45684">打开控制台 Elements 面板，可以看到这样的 HTML 结构，如下图所示：</p>
<p data-nodeid="45685"><img src="https://s0.lgstatic.com/i/image6/M01/33/EB/Cgp9HWBv-1GAHK67AAAuHUpDrAg091.png" alt="Drawing 4.png" data-nodeid="45878"></p>
<div data-nodeid="45686"><p style="text-align:center">控制台查看 HTML 元素图</p></div>
<p data-nodeid="45687">在浏览器中，上面的 HTML 会被解析成这样的 DOM 树，如下图所示：</p>
<p data-nodeid="45688"><img src="https://s0.lgstatic.com/i/image6/M00/34/1D/Cgp9HWBwLYmAP2m9AAB9VgDwVDs760.png" alt="图片6.png" data-nodeid="45882"></p>
<div data-nodeid="45689"><p style="text-align:center">DOM 树示意图</p></div>
<p data-nodeid="45690">我们都知道，对于树状结构来说，常常使用<code data-backticks="1" data-nodeid="45884">parent</code>/<code data-backticks="1" data-nodeid="45886">child</code>/<code data-backticks="1" data-nodeid="45888">sibling</code>等方式来描述各个节点之间的关系，对于 DOM 树也不例外。或许对于很多前端开发者来说，“DOM 是树状结构”已经是个过于基础的认识，因此我们也常常忽略掉开发过程中对它的依赖程度。</p>
<p data-nodeid="45691">举个例子，我们常常会对页面功能进行抽象，并封装成组件。但不管怎么进行整理，页面最终依然是基于 DOM 的树状结构，因此组件也是呈树状结构，组件间的关系也同样可以使用<code data-backticks="1" data-nodeid="45891">parent</code>/<code data-backticks="1" data-nodeid="45893">child</code>/<code data-backticks="1" data-nodeid="45895">sibling</code>这样的方式来描述。</p>
<p data-nodeid="45692">同时，现在大多数应用程序同样以<code data-backticks="1" data-nodeid="45898">root</code>为根节点展开，我们进行状态管理、数据管理也常常会呈现出树状结构，这在 Angular.js 升级到 Angular 的过程中也有所体现。Angular 增加了树状结构的模块化设计，不管是脏检查机制，还是依赖注入的管理，都由于这样的调整提升了性能、降低了模块间的耦合程度。</p>
<h4 data-nodeid="45693">操作 DOM</h4>
<p data-nodeid="45694">除了获取 DOM 结构以外，通过 HTML DOM 相关接口，我们还可以使用 JavaScript 来访问 DOM 树中的节点，也可以创建或删除节点。比如我们想在上面的滚动控制面板中删除一个播放子列，可以这么操作：</p>
<pre class="lang-javascript" data-nodeid="45695"><code data-language="javascript"><span class="hljs-comment">// 获取到 class 为 swiper-control 的第一个节点，这里得到我们的滚动控制面板</span>
<span class="hljs-keyword">const</span> controlPanel = <span class="hljs-built_in">document</span>.getElementsByClassName(<span class="hljs-string">"swiper-control"</span>)[<span class="hljs-number">0</span>];
<span class="hljs-comment">// 获取滚动控制面板的第一个子节点</span>
<span class="hljs-comment">// 这里是“就业率口碑训练营限时抄底”文本所在的子列</span>
<span class="hljs-keyword">const</span> firstChild = controlPanel.firstElementChild;
<span class="hljs-comment">// 删除滚动控制面板的子节点</span>
controlPanel.removeChild(firstChild);
</code></pre>
<p data-nodeid="45696">操作之后，我们能看到节点被顺利删除，如下图所示：</p>
<p data-nodeid="45697"><img src="https://s0.lgstatic.com/i/image6/M00/34/1D/Cgp9HWBwLZGAHBmCAAQlChG17Pw065.png" alt="图片7.png" data-nodeid="45905"></p>
<div data-nodeid="45698"><p style="text-align:center">DOM 节点删除后示意图</p></div>
<p data-nodeid="45699">随着应用程序越来越复杂，DOM 操作越来越频繁，需要监听事件和在事件回调更新页面的 DOM 操作也越来越多，频繁的 DOM 操作会导致页面频繁地进行计算和渲染，导致不小的性能开销。于是虚拟 DOM 的想法便被人提出，并在许多框架中都有实现。</p>
<p data-nodeid="45700">虚拟 DOM 其实是用来模拟真实 DOM 的中间产物，它的设计大致可分成 3 个过程：</p>
<ol data-nodeid="45701">
<li data-nodeid="45702">
<p data-nodeid="45703">用 JavaScript 对象模拟 DOM 树，得到一棵虚拟 DOM 树；</p>
</li>
<li data-nodeid="45704">
<p data-nodeid="45705">当页面数据变更时，生成新的虚拟 DOM 树，比较新旧两棵虚拟 DOM 树的差异；</p>
</li>
<li data-nodeid="45706">
<p data-nodeid="45707">把差异应用到真正的 DOM 树上。</p>
</li>
</ol>
<p data-nodeid="45708">后面我在介绍前端框架时，会更详细地介绍虚拟 DOM 部分的内容。</p>
<h4 data-nodeid="45709">事件委托</h4>
<p data-nodeid="45710">我们知道，浏览器中各个元素从页面中接收事件的顺序包括事件捕获阶段、目标阶段、事件冒泡阶段。其中，基于事件冒泡机制，我们可以实现将子元素的事件委托给父级元素来进行处理，这便是事件委托。</p>
<p data-nodeid="45711">在拉勾官网上，我们需要监听滚动控制面板中的几个文本被点击，从而控制广告面板的展示内容，如下图所示：</p>
<p data-nodeid="45712"><img src="https://s0.lgstatic.com/i/image6/M00/34/25/CioPOWBwLZmAbHu7AAD6mAT107Y372.png" alt="图片8.png" data-nodeid="45917"></p>
<div data-nodeid="45713"><p style="text-align:center">滚动控制面板 DOM 结构示意图</p></div>
<p data-nodeid="45714">如果我们在每个元素上都进行监听的话，则需要绑定三个事件。</p>
<pre class="lang-javascript" data-nodeid="45715"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">clickEventFunction</span>(<span class="hljs-params">e</span>) </span>{
  <span class="hljs-built_in">console</span>.log(e.target === <span class="hljs-keyword">this</span>); <span class="hljs-comment">// logs `true`</span>
  <span class="hljs-comment">// 这里可以用 this 获取当前元素</span>
  <span class="hljs-comment">// 此处控制广告面板的展示内容</span>
}
<span class="hljs-comment">// 元素2、5、8绑定</span>
element2.addEventListener(<span class="hljs-string">"click"</span>, clickEventFunction, <span class="hljs-literal">false</span>);
element5.addEventListener(<span class="hljs-string">"click"</span>, clickEventFunction, <span class="hljs-literal">false</span>);
element8.addEventListener(<span class="hljs-string">"click"</span>, clickEventFunction, <span class="hljs-literal">false</span>);
</code></pre>
<p data-nodeid="45716">使用事件委托，可以通过将事件添加到它们的父节点，而将事件委托给父节点来触发处理函数：</p>
<pre class="lang-javascript" data-nodeid="45717"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">clickEventFunction</span>(<span class="hljs-params">event</span>) </span>{
  <span class="hljs-built_in">console</span>.log(e.target === <span class="hljs-keyword">this</span>); <span class="hljs-comment">// logs `false`</span>
  <span class="hljs-comment">// 获取被点击的元素</span>
  <span class="hljs-keyword">const</span> eventTarget = event.target;
  <span class="hljs-comment">// 检查源元素`event.target`是否符合预期</span>
  <span class="hljs-comment">// 此处控制广告面板的展示内容</span>
}
<span class="hljs-comment">// 元素1绑定</span>
element1.addEventListener(<span class="hljs-string">"click"</span>, clickEventFunction, <span class="hljs-literal">false</span>);
</code></pre>
<p data-nodeid="45718">这样能解决什么问题呢？</p>
<ul data-nodeid="45719">
<li data-nodeid="45720">
<p data-nodeid="45721">绑定子元素会绑定很多次的事件，而绑定父元素只需要一次绑定。</p>
</li>
<li data-nodeid="45722">
<p data-nodeid="45723">将事件委托给父节点，这样我们对子元素的增加和删除、移动等，都不需要重新进行事件绑定。</p>
</li>
</ul>
<p data-nodeid="45724">常见的使用方式主要是上述这种列表结构，每个选项都可以进行编辑、删除、添加标签等功能，而把事件委托给父元素，不管我们新增、删除、更新选项，都不需要手动去绑定和移除事件。</p>
<p data-nodeid="45725">如果在列表数量内容较大的时候，对成千上万节点进行事件监听，也是不小的性能消耗。<strong data-nodeid="45929">使用事件委托的方式，我们可以大量减少浏览器对元素的监听，也是在前端性能优化中比较简单和基础的一个做法</strong>。</p>
<p data-nodeid="45726">需要注意的是，如果我们直接在<code data-backticks="1" data-nodeid="45931">document.body</code>上进行事件委托，可能会带来额外的问题。由于浏览器在进行页面渲染的时候会有合成的步骤，合成的过程会先将页面分成不同的合成层，而用户与浏览器进行交互的时候需要接收事件。此时，浏览器会将页面上具有事件处理程序的区域进行标记，被标记的区域会与主线程进行通信。</p>
<p data-nodeid="45727">如果我们<code data-backticks="1" data-nodeid="45934">document.body</code>上被绑定了事件，这时候整个页面都会被标记。即使我们的页面不关心某些部分的用户交互，合成器线程也必须与主线程进行通信，并在每次事件发生时进行等待。这种情况，我们可以使用<code data-backticks="1" data-nodeid="45936">passive: true</code>选项来解决。</p>
<h3 data-nodeid="45728">小结</h3>
<p data-nodeid="45729">关于 HTML，我今天侧重讲了 HTML 的作用，以及它是如何影响浏览器中页面的加载过程的，同时还介绍了使用 DOM 接口来控制 HTML 的展示和功能逻辑。</p>
<p data-nodeid="45730">很多时候，我们对一些基础内容也都需要不定期地进行复习。古人云“温故而知新”，一些原本认为已经固化的认知，在重新学习的过程中，或许你可以得到新的理解。比如，虚拟 DOM 的设计其实参考了网页中 DOM 设计的很多地方（树状结构、DOM 属性），却又通过简化、新旧对比的方式巧妙地避开了容易出现性能瓶颈的地方，从而提升了页面渲染的性能。</p>
<p data-nodeid="45731">再比如，很多前端框架在监测数据变更的时候采用了树状结构（Angular 2.0+、Vue 3.0+），也是因为即使我们对应用进行了模块化、组件化，最终它在浏览器页面中的呈现和组织方式也依然是树状的，而树状的方式也很好地避免了循环依赖的问题。</p>
<p data-nodeid="45732" class="te-preview-highlight">那么你呢，你在重识 HTML 过程中，学到了新的知识吗？欢迎在留言区分享你的发现。</p>

---

### 精选评论

##### **洲：
> HTML是web开发的基石，用于告诉浏览器如何组织页面的方式，浏览器会根据实际HTML的内容生成一棵树，就是DOM树，可以通过JavaScript访问这颗树来对页面进行更多额外的操作

##### **萍：
> 请问: document.body添加事件委托，每次触发事件时，会产生生等待，为什么会产生等待？passive: true的作用是什么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 产生等待是因为合成器线程于主线程进行通信。passive 设置为 true 时，表示 listener 永远不会调用 preventDefault。根据规范，passive 选项的默认值始终为 false，这引入了处理某些触摸事件（以及其他）的事件监听器在尝试处理滚动时阻止浏览器的主线程的可能性，从而导致滚动处理期间性能可能大大降低。

##### *聪：
> 老师，CSS会阻塞渲染吗？是CSSOM树构建完成之后，页面才开始渲染的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 页面渲染会解析HTML和CSS，生成 DOM Tree 和 CSS Rule Tree，两者结合生成渲染树。最终渲染在页面中的便是渲染树，所以为了避免页面重新渲染，CSS应该放在 header 里哦~更详细的我们会在第 8 讲中进行介绍~

##### *聪：
> 事件委托的第二个例子：【console.log(e.target === this);】应该为【console.log(event.target === this);】

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; get

##### **8635：
> 虚拟dom和实际dom之间是怎么更新替换的，怎么做到页面不会被重新渲染或者局部渲染的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实我们会在第10讲中有比较详细的介绍，这里给点提示：我们平时操作 DOM 的方式有哪些呢？

##### **4829：
> 最后一段关于document.body进行事件委托的，不是很明白，能解释一下么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 document.body 添加事件委托，每次触发事件时，会产生生等待，产生等待是因为合成器线程于主线程进行通信。passive 设置为 true 时，表示 listener 永远不会调用 preventDefault。根据规范，passive 选项的默认值始终为 false，这引入了处理某些触摸事件（以及其他）的事件监听器在尝试处理滚动时阻止浏览器的主线程的可能性，从而导致滚动处理期间性能可能大大降低。

##### **4344：
> 请问对于事件委托不能绑定在body上，还是有点不在明白？passitive是哪个上面的属性呢？我看评论回答这个问题也没怎么明白，还请老师再回复一下，谢谢😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; passive 是事件绑定的一个参数，可以看看 addEventListener() 这个API~

##### **东：
> React17版本的事件委托就有所修改，从原来的html到React.createElement的根元素上，这个修改的原因和都是上述所说的是不是有相同的原因。还有是个人觉得老师你读讲的语速有点快了，比如有图解的东西可以停一点点吗？，

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 收到~我会努力的

##### **蓉：
> 合成层具体是什么，不是很明白

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 合成又称为 Compositing，在现代浏览器渲染过程中，会将将页面的各个部分分成多个层，分别对其进行栅格化并进行合成。这部分内容我们会在第 8 讲中有介绍哦

##### *聪：
> HTML的规范中指明defer属性的脚本是异步下载的，等到页面解析完成后按顺序执行，但是实际上浏览器并不保证顺序执行，所以页面中多个脚本有依赖关系的不要使用defer，平时最好只设置一个defer脚本

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 棒~

