<p data-nodeid="1277" class="">由于 ReactDOM.render 的内容比较多，所以这里拆分了上中下三讲来讲解。</p>
<p data-nodeid="1278">在上一讲，我们站在宏观角度对 Fiber 的架构分层和迭代动机有了充分的把握。从本讲开始，我们将以首次渲染为切入点，拆解 Fiber 架构下 ReactDOM.render 所触发的渲染链路，结合源码理解整个链路中所涉及的初始化、render 和 commit 等过程。</p>
<h3 data-nodeid="1279">ReactDOM.render 调用栈的逻辑分层</h3>
<p data-nodeid="1280">开篇先给到你一个简单的 React AppDemo：</p>
<pre class="lang-java" data-nodeid="1281"><code data-language="java">import React from "react";
import ReactDOM from "react-dom";

function App() {
    return (
      &lt;div className="App"&gt;
        &lt;div className="container"&gt;
          &lt;h1&gt;我是标题&lt;/h1&gt;
          &lt;p&gt;我是第一段话&lt;/p&gt;
          &lt;p&gt;我是第二段话&lt;/p&gt;
        &lt;/div&gt;
      &lt;/div&gt;
    );
}

const rootElement = document.getElementById("root");
ReactDOM.render(&lt;App /&gt;, rootElement);
</code></pre>
<p data-nodeid="1282">Demo 启动后，渲染出的界面如下图所示：</p>
<p data-nodeid="1283"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmEOAGbJ5AAAxGM0SPWA261.png" alt="Drawing 0.png" data-nodeid="1379"></p>
<p data-nodeid="1284">现在请你打开 Chrome 的 Performance 面板，点击下图红色圈圈所圈住的这个“记录”按钮：</p>
<p data-nodeid="1285"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmEuALVycAAEENjoXJ6E407.png" alt="Drawing 1.png" data-nodeid="1383"></p>
<p data-nodeid="1286">然后重新访问 Demo 页面对应的本地服务地址，待页面刷新后，终止记录，便能够得到如下图右下角所示的这样一个调用栈大图：</p>
<p data-nodeid="1287"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmFKAFeHBAAQn6ZuFPrI619.png" alt="Drawing 2.png" data-nodeid="1387"></p>
<p data-nodeid="1288">放大该图，定位“src/index.js”这个文件路径，我们就可以找到 ReactDOM.render 方法对应的调用栈，如下图所示：</p>
<p data-nodeid="1289"><img src="https://s0.lgstatic.com/i/image/M00/6E/CE/Ciqc1F-zmFmAXkYlAAI2ONTKc9s081.png" alt="Drawing 3.png" data-nodeid="1391"></p>
<p data-nodeid="1290">从图中你可以看到，ReactDOM.render 方法对应的调用栈非常深，中间涉及的函数量也比较大。如果这张图使你心里发虚，请先不要急于撤退——分析调用栈只是我们理解渲染链路的一个手段，我们的目的是借此提取关键逻辑，而非理解调用栈中的每一个方法。就这张图来说，你首先需要把握的，就是整个调用链路中所包含的三个阶段：</p>
<p data-nodeid="1291"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmGKAFb5NAAItD2ouVoc061.png" alt="Drawing 4.png" data-nodeid="1395"></p>
<p data-nodeid="1292">图中 scheduleUpdateOnFiber 方法的作用是调度更新，在由 ReactDOM.render 发起的首屏渲染这个场景下，它触发的就是 performSyncWorkOnRoot。performSyncWorkOnRoot 开启的正是我们反复强调的 <strong data-nodeid="1405">render 阶段</strong>；而 commitRoot 方法开启的则是真实 DOM 的渲染过程（<strong data-nodeid="1406">commit 阶段</strong>）。因此以scheduleUpdateOnFiber 和 commitRoot 两个方法为界，我们可以大致把 ReactDOM.render 的调用栈划分为三个阶段：</p>
<ol data-nodeid="1293">
<li data-nodeid="1294">
<p data-nodeid="1295">初始化阶段</p>
</li>
<li data-nodeid="1296">
<p data-nodeid="1297">render 阶段</p>
</li>
<li data-nodeid="1298">
<p data-nodeid="1299">commit 阶段</p>
</li>
</ol>
<p data-nodeid="1300">接下来，我们就一起来看看这三个阶段分别做了哪些事情。</p>
<blockquote data-nodeid="1301">
<p data-nodeid="1302">注：渲染链路串讲已被拆分为 3 个课时，本课时讲解的是初始化阶段。</p>
</blockquote>
<h3 data-nodeid="1303">拆解 ReactDOM.render 调用栈——初始化阶段</h3>
<p data-nodeid="1304">首先我们提取出初始化过程中涉及的调用栈大图：</p>
<p data-nodeid="1305"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmGqAU-42AABcbqaOzFc800.png" alt="Drawing 5.png" data-nodeid="1416"></p>
<p data-nodeid="1306">图中的方法虽然看上去又多又杂，但做的事情清清爽爽，那就是<strong data-nodeid="1422">完成 Fiber 树中基本实体的创建</strong>。</p>
<p data-nodeid="1307">什么是基本实体？基本实体有哪些？问题的答案藏在源码里，这里我为你提取了源码中的关键逻辑，首先是 legacyRenderSubtreeIntoContainer 方法。在 ReactDOM.render 函数体中，以下面代码所示的姿势调用了它：</p>
<pre class="lang-java" data-nodeid="1308"><code data-language="java"><span class="hljs-keyword">return</span> legacyRenderSubtreeIntoContainer(<span class="hljs-keyword">null</span>, element, container, <span class="hljs-keyword">false</span>, callback);
</code></pre>
<p data-nodeid="1309">而 legacyRenderSubtreeIntoContainer 的关键逻辑如下（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="1310"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">legacyRenderSubtreeIntoContainer</span><span class="hljs-params">(parentComponent, children, container, forceHydrate, callback)</span> </span>{
  <span class="hljs-comment">// container 对应的是我们传入的真实 DOM 对象</span>
  <span class="hljs-keyword">var</span> root = container._reactRootContainer;
  <span class="hljs-comment">// 初始化 fiberRoot 对象</span>
  <span class="hljs-keyword">var</span> fiberRoot;
  <span class="hljs-comment">// DOM 对象本身不存在 _reactRootContainer 属性，因此 root 为空</span>
  <span class="hljs-keyword">if</span> (!root) {
    <span class="hljs-comment">// 若 root 为空，则初始化 _reactRootContainer，并将其值赋值给 root</span>
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(container, forceHydrate);
    <span class="hljs-comment">// legacyCreateRootFromDOMContainer 创建出的对象会有一个 _internalRoot 属性，将其赋值给 fiberRoot</span>
    fiberRoot = root._internalRoot;

    <span class="hljs-comment">// 这里处理的是 ReactDOM.render 入参中的回调函数，你了解即可</span>
    <span class="hljs-keyword">if</span> (typeof callback === <span class="hljs-string">'function'</span>) {
      <span class="hljs-keyword">var</span> originalCallback = callback;
      callback = function () {
        <span class="hljs-keyword">var</span> instance = getPublicRootInstance(fiberRoot);
        originalCallback.call(instance);
      };
    } <span class="hljs-comment">// Initial mount should not be batched.</span>
    <span class="hljs-comment">// 进入 unbatchedUpdates 方法</span>
    unbatchedUpdates(function () {
      updateContainer(children, fiberRoot, parentComponent, callback);
    });
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// else 逻辑处理的是非首次渲染的情况（即更新），其逻辑除了跳过了初始化工作，与楼上基本一致</span>
    fiberRoot = root._internalRoot;
    <span class="hljs-keyword">if</span> (typeof callback === <span class="hljs-string">'function'</span>) {
      <span class="hljs-keyword">var</span> _originalCallback = callback;
      callback = function () {
        <span class="hljs-keyword">var</span> instance = getPublicRootInstance(fiberRoot);
        _originalCallback.call(instance);
      };
    } <span class="hljs-comment">// Update</span>

    updateContainer(children, fiberRoot, parentComponent, callback);
  }
  <span class="hljs-keyword">return</span> getPublicRootInstance(fiberRoot);
}
</code></pre>
<p data-nodeid="1311">这里我为你总结一下首次渲染过程中 legacyRenderSubtreeIntoContainer 方法的主要逻辑链路：</p>
<p data-nodeid="1312"><img src="https://s0.lgstatic.com/i/image/M00/70/03/CgqCHl-3mfWABLi5AADUzMV7iHA320.png" alt="Lark20201120-182606.png" data-nodeid="1428"></p>
<p data-nodeid="1313">在这个流程中，你需要关注到 fiberRoot 这个对象。fiberRoot 到底是什么呢？这里我将运行时的 root 和 fiberRoot 为你截取出来，其中 root 对象的结构如下图所示：</p>
<p data-nodeid="1314"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmH6AKzPPAADcEbfK6K4199.png" alt="Drawing 6.png" data-nodeid="1432"></p>
<p data-nodeid="1315">可以看出，root 对象（container._reactRootContainer）上有一个 _internalRoot 属性，这个 _internalRoot 也就是 fiberRoot。fiberRoot 的本质是一个 FiberRootNode 对象，其中包含一个 current 属性，该属性同样需要划重点。这里我为你高亮出 current 属性的部分内容：</p>
<p data-nodeid="1316"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmISANlmfAADLqX8jue0154.png" alt="Drawing 7.png" data-nodeid="1442"></p>
<p data-nodeid="1317">或许你会对 current 对象包含的海量属性感到陌生和头大，但这并不妨碍你 Get 到“current 对象是一个 FiberNode 实例”这一点，<strong data-nodeid="1452">而 FiberNode，正是 Fiber 节点对应的对象类型</strong>。current 对象是一个 Fiber 节点，不仅如此，它还是<strong data-nodeid="1453">当前 Fiber 树的头部节点</strong>。</p>
<p data-nodeid="1318">考虑到 current 属性对应的 FiberNode 节点，在调用栈中实际是由 createHostRootFiber 方法创建的，React 源码中也有多处以 rootFiber 代指 current 对象，因此下文中我们将以 rootFiber 指代 current 对象。</p>
<p data-nodeid="1319">读到这里，你脑海中应该不难形成一个这样的指向关系：</p>
<p data-nodeid="1320"><img src="https://s0.lgstatic.com/i/image/M00/6F/F8/Ciqc1F-3mh-AZrlvAABgy8S1u44402.png" alt="Lark20201120-182610.png" data-nodeid="1458"></p>
<p data-nodeid="1321">其中，fiberRoot 的关联对象是真实 DOM 的容器节点；而 rootFiber 则作为虚拟 DOM 的根节点存在。<strong data-nodeid="1464">这两个节点，将是后续整棵 Fiber 树构建的起点</strong>。</p>
<p data-nodeid="1322">接下来，fiberRoot 将和 ReactDOM.render 方法的其他入参一起，被传入 updateContainer 方法，从而形成一个回调。这个回调，正是接下来要调用的 unbatchedUpdates 方法的入参。我们一起看看 unbatchedUpdates 做了什么，下面代码是对 unbatchedUpdates 主体逻辑的提取：</p>
<pre class="lang-java" data-nodeid="1323"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">unbatchedUpdates</span><span class="hljs-params">(fn, a)</span> </span>{
  <span class="hljs-comment">// 这里是对上下文的处理，不必纠结</span>
  <span class="hljs-keyword">var</span> prevExecutionContext = executionContext;
  executionContext &amp;= ~BatchedContext;
  executionContext |= LegacyUnbatchedContext;
  <span class="hljs-keyword">try</span> {
    <span class="hljs-comment">// 重点在这里，直接调用了传入的回调函数 fn，对应当前链路中的 updateContainer 方法</span>
    <span class="hljs-keyword">return</span> fn(a);
  } <span class="hljs-keyword">finally</span> {
    <span class="hljs-comment">// finally 逻辑里是对回调队列的处理，此处不用太关注</span>
    executionContext = prevExecutionContext;
    <span class="hljs-keyword">if</span> (executionContext === NoContext) {
      <span class="hljs-comment">// Flush the immediate callbacks that were scheduled during this batch</span>
      resetRenderTimer();
      flushSyncCallbackQueue();
    }
  }
}
</code></pre>
<p data-nodeid="1324">在 unbatchedUpdates 函数体里，当下你只需要 Get 到一个信息：它直接调用了传入的回调 fn。而在当前链路中，fn 是什么呢？<strong data-nodeid="1471">fn 是一个针对 updateContainer 的调用</strong>：</p>
<pre class="lang-java" data-nodeid="1325"><code data-language="java">unbatchedUpdates(function () {
  updateContainer(children, fiberRoot, parentComponent, callback);
});
</code></pre>
<p data-nodeid="1326">接下来我们很有必要去看看 updateContainer 里面的逻辑。这里我将主体代码提取如下（解析在注释里，如果没有耐心读完可以直接看文字解读）：</p>
<pre class="lang-java" data-nodeid="1327"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">updateContainer</span><span class="hljs-params">(element, container, parentComponent, callback)</span> </span>{
  ......

  <span class="hljs-comment">// 这是一个 event 相关的入参，此处不必关注</span>
  <span class="hljs-keyword">var</span> eventTime = requestEventTime();

  ......

  <span class="hljs-comment">// 这是一个比较关键的入参，lane 表示优先级</span>
  <span class="hljs-keyword">var</span> lane = requestUpdateLane(current$<span class="hljs-number">1</span>);
  <span class="hljs-comment">// 结合 lane（优先级）信息，创建 update 对象，一个 update 对象意味着一个更新</span>
  <span class="hljs-keyword">var</span> update = createUpdate(eventTime, lane); 

  <span class="hljs-comment">// update 的 payload 对应的是一个 React 元素</span>
  update.payload = {
    element: element
  };

  <span class="hljs-comment">// 处理 callback，这个 callback 其实就是我们调用 ReactDOM.render 时传入的 callback</span>
  callback = callback === undefined ? <span class="hljs-keyword">null</span> : callback;
  <span class="hljs-keyword">if</span> (callback !== <span class="hljs-keyword">null</span>) {
    {
      <span class="hljs-keyword">if</span> (typeof callback !== <span class="hljs-string">'function'</span>) {
        error(<span class="hljs-string">'render(...): Expected the last optional `callback` argument to be a '</span> + <span class="hljs-string">'function. Instead received: %s.'</span>, callback);
      }
    }
    update.callback = callback;
  }

  <span class="hljs-comment">// 将 update 入队</span>
  enqueueUpdate(current$<span class="hljs-number">1</span>, update);
  <span class="hljs-comment">// 调度 fiberRoot </span>
  scheduleUpdateOnFiber(current$<span class="hljs-number">1</span>, lane, eventTime);
  <span class="hljs-comment">// 返回当前节点（fiberRoot）的优先级</span>
  <span class="hljs-keyword">return</span> lane;
}
</code></pre>
<p data-nodeid="1328">updateContainer 的逻辑相对来说丰富了点，但大部分逻辑也是在干杂活，它做的最关键的事情可以总结为三件：</p>
<ol data-nodeid="1329">
<li data-nodeid="1330">
<p data-nodeid="1331">请求当前 Fiber 节点的 lane（优先级）；</p>
</li>
<li data-nodeid="1332">
<p data-nodeid="1333">结合 lane（优先级），创建当前 Fiber 节点的 update 对象，并将其入队；</p>
</li>
<li data-nodeid="1334">
<p data-nodeid="1335">调度当前节点（rootFiber）。</p>
</li>
</ol>
<p data-nodeid="1336">函数体中的 scheduleWork 其实就是 scheduleUpdateOnFiber，scheduleUpdateOnFiber 函数的任务是调度当前节点的更新。在这个函数中，会处理一系列与优先级、打断操作相关的逻辑。但是<strong data-nodeid="1482">在 ReactDOM.render 发起的首次渲染链路中，这些意义都不大，因为这个渲染过程其实是同步的</strong>。我们可以尝试在 Source 面板中为该函数打上断点，逐行执行代码，会发现逻辑最终会走到下图的高亮处：</p>
<p data-nodeid="1337"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmJGATpFIAAPP-sFYf70749.png" alt="Drawing 8.png" data-nodeid="1485"></p>
<p data-nodeid="1338">performSyncWorkOnRoot直译过来就是“执行根节点的同步任务”，<strong data-nodeid="1491">这里的“同步”二字需要注意，它明示了接下来即将开启的是一个同步的过程</strong>。这也正是为什么在整个渲染链路中，调度（Schedule）动作没有存在感的原因。</p>
<p data-nodeid="1339">前面我们曾经提到过，performSyncWorkOnRoot 是 render 阶段的起点，render 阶段的任务就是完成 Fiber 树的构建，它是整个渲染链路中最核心的一环。在异步渲染的模式下，render 阶段应该是一个可打断的异步过程（下一讲我们就将针对 render 过程作详细的逻辑拆解）。</p>
<p data-nodeid="1340">而现在，我相信你心里更多的疑惑在于：<strong data-nodeid="1498">都说 Fiber 架构带来的异步渲染是 React 16 的亮点，为什么分析到现在，竟然发现 ReactDOM.render 触发的首次渲染是个同步过程呢</strong>？</p>
<h3 data-nodeid="1341">同步的 ReactDOM.render，异步的 ReactDOM.createRoot</h3>
<p data-nodeid="1342">其实在 React 16，包括近期发布的 React 17 小版本中，React 都有以下 3 种启动方式：</p>
<p data-nodeid="1343"><strong data-nodeid="1516">legacy 模式</strong>：<br>
<code data-backticks="1" data-nodeid="1506">ReactDOM.render(&lt;App /&gt;, rootNode)</code>。这是当前 React App 使用的方式，当前没有计划删除本模式，但是这个模式可能不支持这些新功能。<br>
<strong data-nodeid="1517">blocking 模式</strong>：<br>
<code data-backticks="1" data-nodeid="1514">ReactDOM.createBlockingRoot(rootNode).render(&lt;App /&gt;)</code>。目前正在实验中，作为迁移到 concurrent 模式的第一个步骤。</p>
<p data-nodeid="1344"><strong data-nodeid="1525">concurrent 模式</strong>：<br>
<code data-backticks="1" data-nodeid="1523">ReactDOM.createRoot(rootNode).render(&lt;App /&gt;)</code>。目前在实验中，未来稳定之后，打算作为 React 的默认开发模式，这个模式开启了所有的新功能。</p>
<p data-nodeid="1345">在这 3 种模式中，<strong data-nodeid="1535">我们常用的 ReactDOM.render 对应的是 legacy 模式，它实际触发的仍然是同步的渲染链路</strong>。blocking 模式可以理解为 legacy 和 concurrent 之间的一个过渡形态，之所以会有这个模式，是因为 React 官方希望能够提供<a href="https://zh-hans.reactjs.org/docs/faq-versioning.html#commitment-to-stability" data-nodeid="1533">渐进的迁移策略</a>，帮助我们更加顺滑地过渡到 Concurrent 模式。blocking 在实际应用中是比较低频的一个模式，了解即可。</p>
<p data-nodeid="1346">按照官方的说法，“<strong data-nodeid="1541">长远来看，模式的数量会收敛，不用考虑不同的模式</strong>，但就目前而言，模式是一项重要的迁移策略，让每个人都能决定自己什么时候迁移，并按照自己的速度进行迁移”。由此可以看出，Concurrent 模式确实是 React 的终极目标，也是其创作团队使用 Fiber 架构重写核心算法的动机所在。</p>
<h3 data-nodeid="1347">拓展：关于异步模式下的首次渲染链路</h3>
<p data-nodeid="1348">当下，如果想要开启异步渲染，我们需要调用 <code data-backticks="1" data-nodeid="1544">ReactDOM.createRoot</code>方法来启动应用，那<code data-backticks="1" data-nodeid="1546">ReactDOM.createRoot</code>开启的渲染链路与 ReactDOM.render 有何不同呢？</p>
<p data-nodeid="1349">这里我修改一下调用方式，给你展示一下调用栈。由于本讲的源码取材于 React 17.0.0 版本，在这个版本中，createRoot 仍然是一个 unstable 的方法。因此实际调用的 API 应该是“unstable_createRoot”：</p>
<pre class="lang-java" data-nodeid="1350"><code data-language="java">ReactDOM.unstable_createRoot(rootElement).render(&lt;App /&gt;);
</code></pre>
<p data-nodeid="1351">Concurrent 模式开启后，首次渲染的调用栈变成了如下图所示的样子：</p>
<p data-nodeid="1352"><img src="https://s0.lgstatic.com/i/image/M00/6E/D9/CgqCHl-zmJyAbYZNAAFI67qKm98019.png" alt="Drawing 9.png" data-nodeid="1554"></p>
<p data-nodeid="1353">乍一看，好像和 ReactDOM.render 差别很大，其实不然。图中 createRoot 所触发的逻辑仍然是一些准备性质的初始化工作，此处不必太纠结。关键在于下面我给你框出来的这部分，如下图所示：</p>
<p data-nodeid="1354"><img src="https://s0.lgstatic.com/i/image/M00/6E/CE/Ciqc1F-zmKKAF0ODAADhhdYWzo0441.png" alt="Drawing 10.png" data-nodeid="1558"></p>
<p data-nodeid="1355">我们拉近一点来看，如下图所示：</p>
<p data-nodeid="1356"><img src="https://s0.lgstatic.com/i/image/M00/70/75/CgqCHl-7GiaAUY_zAAxz8mfEvT0309.png" alt="图片1.png" data-nodeid="1562"><br>
你会发现这地方也调用了一个 render。再顺着这个调用往下看，发现有大量的熟悉面孔：updateContainer、requestUpdateLane、createUpdate、scheduleUpdateOnFiber......这些函数在 ReactDOM.render 的调用栈中也出现过。</p>
<p data-nodeid="1357">其实，当前你看到的这个 render 调用链路，和 ReactDOM.render 的调用链路是非常相似的，主要的区别在 scheduleUpdateOnFiber 的这个判断里：</p>
<p data-nodeid="1358"><img src="https://s0.lgstatic.com/i/image/M00/6E/CE/Ciqc1F-zmMKAJFKYAAMfoIVWxeM650.png" alt="image.png" data-nodeid="1568"></p>
<p data-nodeid="1359">在异步渲染模式下，由于请求到的 lane 不再是 SyncLane（同步优先级），故不会再走到 performSyncWorkOnRoot 这个调用，而是会转而执行 else 中调度相关的逻辑。</p>
<p data-nodeid="1360">这里有个点要给你点出来——React 是如何知道当前处于哪个模式的呢？我们可以以 requestUpdateLane 函数为例，下面是它局部的代码：</p>
<pre class="lang-java" data-nodeid="1361"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">requestUpdateLane</span><span class="hljs-params">(fiber)</span> </span>{
  <span class="hljs-comment">// 获取 mode 属性</span>
  <span class="hljs-keyword">var</span> mode = fiber.mode;
  <span class="hljs-comment">// 结合 mode 属性判断当前的</span>
  <span class="hljs-keyword">if</span> ((mode &amp; BlockingMode) === NoMode) {
    <span class="hljs-keyword">return</span> SyncLane;
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> ((mode &amp; ConcurrentMode) === NoMode) {
    <span class="hljs-keyword">return</span> getCurrentPriorityLevel() === ImmediatePriority$<span class="hljs-number">1</span> ? SyncLane : SyncBatchedLane;
  }
  ......
  <span class="hljs-keyword">return</span> lane;
}
</code></pre>
<p data-nodeid="1362">上面代码中需要注意 fiber节点上的 mode 属性：<strong data-nodeid="1576">React 将会通过修改 mode 属性为不同的值，来标识当前处于哪个渲染模式；在执行过程中，也是通过判断这个属性，来区分不同的渲染模式</strong>。</p>
<p data-nodeid="1363">因此不同的渲染模式在挂载阶段的差异，本质上来说并不是工作流的差异（其工作流涉及 初始化 → render → commit 这 3 个步骤），而是 mode 属性的差异。mode 属性决定着这个工作流是一气呵成（同步）的，还是分片执行（异步）的。</p>
<p data-nodeid="1364">关于异步挂载/更新的实现细节，我们将在后续的第 16 讲“Fiber 架构实现原理与编码形态”中详细探讨。</p>
<h3 data-nodeid="1365">Fiber 架构一定是异步渲染吗？</h3>
<p data-nodeid="1366">之前我曾经被读者朋友问到过这样的问题：<strong data-nodeid="1585">React 16 如果没有开启 Concurrent 模式，那它还能叫 Fiber 架构吗</strong>？</p>
<p data-nodeid="1367">这个问题很有意思，从动机上来看，Fiber 架构的设计确实主要是为了 Concurrent 而存在。但经过了本讲紧贴源码的讲解，相信你也能够看出，在 React 16，包括已发布的 React 17 版本中，不管是否是 Concurrent，整个数据结构层面的设计、包括贯穿整个渲染链路的处理逻辑，已经完全用 Fiber 重构了一遍。站在这个角度来看，Fiber 架构在 React 中并不能够和异步渲染画严格的等号，它是一种<strong data-nodeid="1591">同时兼容了同步渲染与异步渲染的设计</strong>。</p>
<h3 data-nodeid="1368">总结</h3>
<p data-nodeid="1369">从本讲开始，我们以 ReactDOM.render 所触发的首次渲染为切入点，试图串联 React Fiber 架构下完整的工作链路，本讲为整个源码链路分析的前半部分。</p>
<p data-nodeid="1370">正所谓“磨刀不误砍柴工”。虽然当前的进度条只推到了初始化这个位置，但在这部分的分析过程中，相信你已经对Fiber 树的初始形态、Fiber 根节点的创建过程建立了感性的认知，同时把握住了 ReactDOM.render 同步渲染的过程特征，理解了 React 当下共存的3种渲染方式。在此基础上，我们再去理解 render 过程，就会轻松得多。</p>
<p data-nodeid="1371" class="te-preview-highlight">整个初始化的工作过程都是在为后续的 render 阶段做准备。现在，我们的 Fiber Tree 还处在只有根节点的起始状态。接下来，我们就要进入到最最关键的 render 阶段里去，一起去看看这棵树是怎么一点点丰满起来的，加油！</p>

---

### 精选评论

##### **琼：
> 感觉这里之前老师讲的都挺好的，但是从将Fiber架构开始，跨度有点大，感觉有点难以理解（还是我太菜了）

##### console_man：
> 太赞了!讲得很透彻!

##### Sola：
> 条理清晰，娓娓道来，赞

##### **蓉：
> 看第一遍时脑子嗡嗡的，现在看完了上中下再来看第二遍好多了，没有嗡嗡的了

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 温故知新，小伙伴加油哦

##### **森：
> 续上打卡

##### **用户9471：
> 看了几遍 讲得很棒 谢谢大佬

##### **你辣条就跑：
> 太香了！！！给老师疯狂打call！！！！！

##### **豪：
> 看了多少教程视频都是开着倍速看的，这是为数不多的不想开着倍速，一步一步琢磨着看的教程了

##### **8542：
> react根据fibe节点的mode属性判断渲染模式，如果是reactdom.render他的首次渲染是同步的。如改用reactdom.creatroot他的首次渲染就是异步的。虽说fiber代表纤程或者协程，意味着他就是异步。但即使render初次渲染是同步的，但是他是包裹在fiber架构里面一种特殊处理形式。本课主要讲解了渲染的初始化阶段，真实dom和App组件进入reactdom. render以后会被处理成一个新的update对象。这个对象面fiberroot对象(真dom处成的fiber对象)的current属性里面装的是rootfiber对象(app组件生成的fiber对象)此时他会将原来的reactelement处理成fiber对象，也就是在原来的基础上，再添加许多后续fiber架构需要的属性，比如mode等。

##### **8542：
> 当前整个调和过程都用fiber重构了一遍，为了做到渐进式的迁移项目，他包含了三中模式，根据不同的模式进入不同的代码，导致当前react.dom.render的首次渲染依旧是同步的，而reactdom.createroot才是真正的最终的异步的过程。不管同步还是异步，他们都是fiber重构后的结果。

##### *琴：
> 老师讲得太棒了

##### **伟：
> ReactDOM.unstable_createRoot(rootElement).render(好像无法生效

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 检查一下版本看看？实测17.0.1是ok的。关于这个API的更多信息可以看看https://zh-hans.reactjs.org/docs/concurrent-mode-adoption.html

##### *诺：
> 老师，我装了 17.0.1，没找到 createRoot 这个方法哎

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 仔细读一下原文，有提到目前实际调用的api应该是unstable_createRoot

