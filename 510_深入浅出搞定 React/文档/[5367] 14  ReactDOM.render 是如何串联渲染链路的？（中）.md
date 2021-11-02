<p data-nodeid="2389" class="">上一讲我们对 ReactDOM.render 的调用链路、包括其对应的初始化阶段的工作内容都有了学习和掌握。这一讲我们在此基础上，学习后续的 render 阶段和 commit 阶段。这其中，render 阶段可以认为是整个渲染链路中最为核心的一环，因为我们反复强调“找不同”的过程，恰恰就是在这个阶段发生的。</p>
<p data-nodeid="2390">render 阶段做的事情有很多，这一讲我们将以 beginWork 为线索，着重探讨 Fiber 树的构建过程。</p>
<h3 data-nodeid="2391">拆解 ReactDOM.render 调用栈——render 阶段</h3>
<p data-nodeid="2392">首先，我们复习一下 render 阶段在整个渲染链路中的定位，如下图所示。</p>
<p data-nodeid="2393"><img src="https://s0.lgstatic.com/i/image/M00/71/0B/CgqCHl-8xCmAcvVyAADtTCzN0RM929.png" alt="Drawing 0.png" data-nodeid="2561"></p>
<p data-nodeid="2394">图中，performSyncWorkOnRoot 标志着 render 阶段的开始，finishSyncRender 标志着 render 阶段的结束。这中间包含了大量的 beginWork、completeWork 调用栈，正是 render 的工作内容。</p>
<blockquote data-nodeid="2395">
<p data-nodeid="2396">beginWork、completeWork 这两个方法需要注意，它们串联起的是一个“模拟递归”的过程。</p>
</blockquote>
<p data-nodeid="2397">在第 10 讲“栈调和”中强调过，React 15 下的调和过程<strong data-nodeid="2577">是一个递归的过程</strong>。而 Fiber 架构下的调和过程，虽然并不是依赖递归来实现的，<strong data-nodeid="2578">但在 ReactDOM.render 触发的同步模式下，它仍然是一个深度优先搜索的过程</strong>。在这个过程中，<strong data-nodeid="2579">beginWork 将创建新的 Fiber 节点，而 completeWork 则负责将 Fiber 节点映射为 DOM 节点</strong>。</p>
<p data-nodeid="2398">那么问题就来了：截止到上一讲，我们的 Fiber 树都还长这个样子：</p>
<p data-nodeid="2399"><img src="https://s0.lgstatic.com/i/image/M00/71/0A/CgqCHl-8w7qAc91bAABOxKDmLgA173.png" alt="Drawing 1.png" data-nodeid="2583"></p>
<p data-nodeid="2400">就这么个样子，你遍历它，能遍历出来什么？到底怎么个遍历法？接下来我们就深入到源码里去一探究竟！</p>
<h3 data-nodeid="2401">workInProgress 节点的创建</h3>
<p data-nodeid="2402">上一讲曾经提到，performSyncWorkOnRoot &nbsp;是 render 阶段的起点，而这个函数最关键的地方在于它调用了 renderRootSync。下面我们放大 Performance 调用栈，来看看 renderRootSync 被调用后，紧接着发生了什么：</p>
<p data-nodeid="2403"><img src="https://s0.lgstatic.com/i/image/M00/70/FF/Ciqc1F-8xByAOzCeAAAoruuugdE734.png" alt="Drawing 2.png" data-nodeid="2589"></p>
<p data-nodeid="2404">紧随其后的是 prepareFreshStack，这里不卖关子，prepareFreshStack 的作用是重置一个新的堆栈环境，其中最需要我们关注的步骤，就是对<strong data-nodeid="2595">createWorkInProgress</strong> 的调用。以下我对 createWorkInProgress 的主要逻辑进行了提取（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="2405"><code data-language="java"><span class="hljs-comment">// 这里入参中的 current 传入的是现有树结构中的 rootFiber 对象</span>
<span class="hljs-function">function <span class="hljs-title">createWorkInProgress</span><span class="hljs-params">(current, pendingProps)</span> </span>{
  <span class="hljs-keyword">var</span> workInProgress = current.alternate;
  <span class="hljs-comment">// ReactDOM.render 触发的首屏渲染将进入这个逻辑</span>
  <span class="hljs-keyword">if</span> (workInProgress === <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 这是需要你关注的第一个点，workInProgress 是 createFiber 方法的返回值</span>
    workInProgress = createFiber(current.tag, pendingProps, current.key, current.mode);
    workInProgress.elementType = current.elementType;
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;
    <span class="hljs-comment">// 这是需要你关注的第二个点，workInProgress 的 alternate 将指向 current</span>
    workInProgress.alternate = current;
    <span class="hljs-comment">// 这是需要你关注的第三个点，current 的 alternate 将反过来指向 workInProgress</span>
    current.alternate = workInProgress;
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// else 的逻辑此处先不用关注</span>
  }

  <span class="hljs-comment">// 以下省略大量 workInProgress 对象的属性处理逻辑</span>
  <span class="hljs-comment">// 返回 workInProgress 节点</span>
  <span class="hljs-keyword">return</span> workInProgress;
}
</code></pre>
<p data-nodeid="2406">首先要声明的是，该函数中的 current 入参指的是现有树结构中的 rootFiber 对象，如下图所示：</p>
<p data-nodeid="2407"><img src="https://s0.lgstatic.com/i/image/M00/70/FF/Ciqc1F-8xDeAR3RMAAClHPw_BEk265.png" alt="Drawing 3.png" data-nodeid="2599"></p>
<p data-nodeid="2408">源码太长（其实经过处理已经不长了）不看版的重点如下：</p>
<ul data-nodeid="2409">
<li data-nodeid="2410">
<p data-nodeid="2411">createWorkInProgress 将<strong data-nodeid="2610">调用 createFiber</strong>，workInProgress<strong data-nodeid="2611">是 createFiber 方法的返回值</strong>；</p>
</li>
<li data-nodeid="2412">
<p data-nodeid="2413">workInProgress 的 <strong data-nodeid="2617">alternate 将指向 current</strong>；</p>
</li>
<li data-nodeid="2414">
<p data-nodeid="2415"><strong data-nodeid="2622">current 的 alternate 将反过来指向 workInProgress</strong>。</p>
</li>
</ul>
<p data-nodeid="2416">理解了这三点，你就会自然而然地想知道 workInProgress 的本体到底是什么样的，也就是<strong data-nodeid="2628">createFiber 到底会返回什么</strong>。下面我们就看看 createFiber 的逻辑：</p>
<pre class="lang-java" data-nodeid="2417"><code data-language="java"><span class="hljs-keyword">var</span> createFiber = function (tag, pendingProps, key, mode) {

  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> FiberNode(tag, pendingProps, key, mode);
};
</code></pre>
<p data-nodeid="2418"><strong data-nodeid="2637">代码出奇的简单，但信息却给得很到位 —— createFiber 将创建一个 FiberNode 实例</strong>，而 FiberNode，上一讲已经讲过，它正是 Fiber 节点的类型。<strong data-nodeid="2638">因此 workInProgress 就是一个 Fiber 节点</strong>。不仅如此，细心的你可能还会发现 workInProgress 的创建入参其实来源于 current，如下面代码所示：</p>
<pre class="lang-java" data-nodeid="2419"><code data-language="java"> workInProgress = createFiber(current.tag, pendingProps, current.key, current.mode);
</code></pre>
<p data-nodeid="2420"><strong data-nodeid="2643">实锤了，workInProgress 节点其实就是 current 节点（即 rootFiber）的副本</strong>。</p>
<p data-nodeid="2421">再结合 &nbsp;current 指向 rootFiber 对象（同样是 FiberNode 实例），以及 current 和 workInProgress 通过 alternate 互相连接这些信息，我们可以分析出这波操作执行完之后，整棵树的结构应该如下图所示：</p>
<p data-nodeid="2422"><img src="https://s0.lgstatic.com/i/image/M00/71/49/CgqCHl-91EqAJlftAAB6KmeoTMw529.png" alt="1.png" data-nodeid="2647"></p>
<p data-nodeid="2423">完成了这个任务之后，就会进入 workLoopSync 的逻辑。这个 workLoopSync 函数也是个“人狠话不多”的主，它的逻辑同样是简洁明了的，如下所示（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="2424"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">workLoopSync</span><span class="hljs-params">()</span> </span>{
  <span class="hljs-comment">// 若 workInProgress 不为空</span>
  <span class="hljs-keyword">while</span> (workInProgress !== <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 针对它执行 performUnitOfWork 方法</span>
    performUnitOfWork(workInProgress);
  }
}
</code></pre>
<p data-nodeid="2425">workLoopSync 做的事情就是<strong data-nodeid="2654">通过 while 循环反复判断 workInProgress 是否为空，并在不为空的情况下针对它执行 performUnitOfWork 函数</strong>。</p>
<p data-nodeid="2426">而 performUnitOfWork 函数将<strong data-nodeid="2664">触发对 beginWork 的调用，进而实现对新 Fiber 节点的创建</strong>。若 beginWork 所创建的 Fiber 节点不为空，则 performUniOfWork 会用这个新的 Fiber 节点来更新 workInProgress 的值，<strong data-nodeid="2665">为下一次循环做准备</strong>。</p>
<p data-nodeid="2427"><strong data-nodeid="2670">通过循环调用 performUnitOfWork 来触发 beginWork，新的 Fiber 节点就会被不断地创建</strong>。当 workInProgress 终于为空时，说明没有新的节点可以创建了，也就意味着已经完成对整棵 Fiber 树的构建。</p>
<p data-nodeid="2428">在这个过程中，<strong data-nodeid="2680">每一个被创建出来的新 Fiber 节点，都会一个一个挂载为最初那个 workInProgress 节点（如下图高亮处）的后代节点</strong>。而上述过程中构建出的这棵 Fiber 树，也正是大名鼎鼎的 <strong data-nodeid="2681">workInProgress 树</strong>。</p>
<p data-nodeid="2429"><img src="https://s0.lgstatic.com/i/image/M00/71/49/CgqCHl-91HeADxF2AACYnkvx4lM165.png" alt="2.png" data-nodeid="2684"></p>
<p data-nodeid="2430">相应地，图中 current 指针所指向的根节点所在的那棵树，我们叫它“<strong data-nodeid="2690">current 树</strong>”。</p>
<p data-nodeid="2431">这时候，相信一些同学心里已经开始犯嘀咕了：一棵 current 树，一棵 workInProgress 树，这名堂也太多了吧！况且这两棵 Fiber 树至少在现在看来，是完全没区别的（毕竟都还只有一个根节点，哈哈）。React 这样设计的目的何在？或者换个问法——到底是什么样的事情一棵树做不到，非得搞两棵“一样”的树出来？</p>
<p data-nodeid="2432">如果你想知道答案，就请好好把握住接下来的两讲内容吧！在一步一步理解 Fiber 树的构建和更新过程之后，我将带你去认识“两棵 Fiber 树”这一现象背后的动机。</p>
<p data-nodeid="2433">接下来我们就深入到 beginWork 和 completeWork 的逻辑里去，一起看看 Fiber 树的构建过程及最终形态。</p>
<h3 data-nodeid="2434">beginWork 开启 Fiber 节点创建过程</h3>
<p data-nodeid="2435">有一说一，beginWork 的源码实在是长到不科学。这里我们本着抓主要矛盾的原则，针对与树构建过程强相关的动作进行逻辑提取，代码如下（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="2436"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">beginWork</span><span class="hljs-params">(current, workInProgress, renderLanes)</span> </span>{
  ......

  <span class="hljs-comment">//  current 节点不为空的情况下，会加一道辨识，看看是否有更新逻辑要处理</span>
  <span class="hljs-keyword">if</span> (current !== <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 获取新旧 props</span>
    <span class="hljs-keyword">var</span> oldProps = current.memoizedProps;
    <span class="hljs-keyword">var</span> newProps = workInProgress.pendingProps;

    <span class="hljs-comment">// 若 props 更新或者上下文改变，则认为需要"接受更新"</span>
    <span class="hljs-keyword">if</span> (oldProps !== newProps || hasContextChanged() || (
     workInProgress.type !== current.type )) {
      <span class="hljs-comment">// 打个更新标</span>
      didReceiveUpdate = <span class="hljs-keyword">true</span>;
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (xxx) {
      <span class="hljs-comment">// 不需要更新的情况 A</span>
      <span class="hljs-keyword">return</span> A
    } <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">if</span> (需要更新的情况 B) {
        didReceiveUpdate = <span class="hljs-keyword">true</span>;
      } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 不需要更新的其他情况，这里我们的首次渲染就将执行到这一行的逻辑</span>
        didReceiveUpdate = <span class="hljs-keyword">false</span>;
      }
    }
  } <span class="hljs-keyword">else</span> {
    didReceiveUpdate = <span class="hljs-keyword">false</span>;
  } 
  ......
  <span class="hljs-comment">// 这坨 switch 是 beginWork 中的核心逻辑，原有的代码量相当大</span>
  <span class="hljs-keyword">switch</span> (workInProgress.tag) {
    ......
    <span class="hljs-comment">// 这里省略掉大量形如"case: xxx"的逻辑</span>
    <span class="hljs-comment">// 根节点将进入这个逻辑</span>
    <span class="hljs-keyword">case</span> HostRoot:
      <span class="hljs-keyword">return</span> updateHostRoot(current, workInProgress, renderLanes)
    <span class="hljs-comment">// dom 标签对应的节点将进入这个逻辑</span>
    <span class="hljs-keyword">case</span> HostComponent:
      <span class="hljs-keyword">return</span> updateHostComponent(current, workInProgress, renderLanes)

    <span class="hljs-comment">// 文本节点将进入这个逻辑</span>
    <span class="hljs-keyword">case</span> HostText:
      <span class="hljs-keyword">return</span> updateHostText(current, workInProgress)
    ...... 
    <span class="hljs-comment">// 这里省略掉大量形如"case: xxx"的逻辑</span>
  }
  <span class="hljs-comment">// 这里是错误兜底，处理 switch 匹配不上的情况</span>
  {
    {
      <span class="hljs-keyword">throw</span> Error(
        <span class="hljs-string">"Unknown unit of work tag ("</span> +
          workInProgress.tag +
          <span class="hljs-string">"). This error is likely caused by a bug in React. Please file an issue."</span>
      )
    }
  }
}
</code></pre>
<p data-nodeid="2437">beginWork 源码太长不看版的重点总结：</p>
<ol data-nodeid="2438">
<li data-nodeid="2439">
<p data-nodeid="2440">beginWork 的入参是<strong data-nodeid="2702">一对用 alternate 连接起来的 workInProgress 和 current 节点</strong>；</p>
</li>
<li data-nodeid="2441" class="">
<p data-nodeid="2442" class=""><strong data-nodeid="2711">beginWork 的核心逻辑是根据 fiber 节点（workInProgress</strong>）<strong data-nodeid="2712">的 tag 属性的不同，调用不同的节点创建函数</strong>。</p>
</li>
</ol>
<p data-nodeid="6574" class="">当前的 current 节点是 rootFiber，而 workInProgress 则是 current 的副本，它们的 tag 都是 3，如下图所示：</p>




<p data-nodeid="2444"><img src="https://s0.lgstatic.com/i/image/M00/71/0B/CgqCHl-8xHmAV2FMAABmLqBlHD0379.png" alt="Drawing 6.png" data-nodeid="2716"></p>
<p data-nodeid="2445">而 3 正是 HostRoot 所对应的值，因此第一个 beginWork 将进入 updateHostRoot 的逻辑。</p>
<p data-nodeid="2446">这里你先不必急于关注 updateHostRoot 的逻辑细节。事实上，在整段 switch 逻辑里，包含的形如“update+类型名”这样的函数是非常多的。在专栏示例的 Demo 中，就涉及了对 updateHostRoot、updateHostComponent 等的调用，十来种 updateXXX，我们不可能一个一个去扣每一个函数的逻辑。</p>
<p data-nodeid="2447">幸运的是，这些函数之间不仅命名形式一致，工作内容也相似。就 render 链路来说，它们共同的特性，就是都会<strong data-nodeid="2724">通过调用 reconcileChildren 方法，生成当前节点的子节点</strong>。</p>
<p data-nodeid="2448">reconcileChildren 的源码如下：</p>
<pre class="lang-java" data-nodeid="2449"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">reconcileChildren</span><span class="hljs-params">(current, workInProgress, nextChildren, renderLanes)</span> </span>{
  <span class="hljs-comment">// 判断 current 是否为 null</span>
  <span class="hljs-keyword">if</span> (current === <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 若 current 为 null，则进入 mountChildFibers 的逻辑</span>
    workInProgress.child = mountChildFibers(workInProgress, <span class="hljs-keyword">null</span>, nextChildren, renderLanes);
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 若 current 不为 null，则进入 reconcileChildFibers 的逻辑</span>
    workInProgress.child = reconcileChildFibers(workInProgress, current.child, nextChildren, renderLanes);
  }
}
</code></pre>
<p data-nodeid="2450">从源码来看，reconcileChildren 也只是做逻辑的分发，具体的工作还要到 <strong data-nodeid="2735">mountChildFibers</strong> 和 <strong data-nodeid="2736">reconcileChildFibers</strong> 里去看。</p>
<h3 data-nodeid="2451">ChildReconciler，处理 Fiber 节点的幕后“操盘手”</h3>
<p data-nodeid="2452">那么这两个函数又是何方神圣呢？在源码中，我们可以觅得这样两个赋值语句：</p>
<pre class="lang-java" data-nodeid="2453"><code data-language="java"><span class="hljs-keyword">var</span> reconcileChildFibers = ChildReconciler(<span class="hljs-keyword">true</span>);
<span class="hljs-keyword">var</span> mountChildFibers = ChildReconciler(<span class="hljs-keyword">false</span>);
</code></pre>
<p data-nodeid="2454">原来 reconcileChildFibers 和 mountChildFibers 不仅名字相似，出处也一致。<strong data-nodeid="2744">它们都是 ChildReconciler 这个函数的返回值，仅仅存在入参上的区别</strong>。而 ChildReconciler，则是一个实打实的“庞然大物”，其内部的逻辑量堪比 N 个 beginWork。这里我将关键要素提取如下（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="2455"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">ChildReconciler</span><span class="hljs-params">(shouldTrackSideEffects)</span> </span>{
  <span class="hljs-comment">// 删除节点的逻辑</span>
  <span class="hljs-function">function <span class="hljs-title">deleteChild</span><span class="hljs-params">(returnFiber, childToDelete)</span> </span>{
    <span class="hljs-keyword">if</span> (!shouldTrackSideEffects) {
      <span class="hljs-comment">// Noop.</span>
      <span class="hljs-keyword">return</span>;
    } 
    <span class="hljs-comment">// 以下执行删除逻辑</span>
  }
 
  ......


  <span class="hljs-comment">// 单个节点的插入逻辑</span>
  <span class="hljs-function">function <span class="hljs-title">placeSingleChild</span><span class="hljs-params">(newFiber)</span> </span>{
    <span class="hljs-keyword">if</span> (shouldTrackSideEffects &amp;&amp; newFiber.alternate === <span class="hljs-keyword">null</span>) {
      newFiber.flags = Placement;
    }
    <span class="hljs-keyword">return</span> newFiber;
  }

  <span class="hljs-comment">// 插入节点的逻辑</span>
  <span class="hljs-function">function <span class="hljs-title">placeChild</span><span class="hljs-params">(newFiber, lastPlacedIndex, newIndex)</span> </span>{
    newFiber.index = newIndex;
    <span class="hljs-keyword">if</span> (!shouldTrackSideEffects) {
      <span class="hljs-comment">// Noop.</span>
      <span class="hljs-keyword">return</span> lastPlacedIndex;
    }
    <span class="hljs-comment">// 以下执行插入逻辑</span>
  }
  ......
  <span class="hljs-comment">// 此处省略一系列 updateXXX 的函数，它们用于处理 Fiber 节点的更新</span>

  <span class="hljs-comment">// 处理不止一个子节点的情况</span>
  <span class="hljs-function">function <span class="hljs-title">reconcileChildrenArray</span><span class="hljs-params">(returnFiber, currentFirstChild, newChildren, lanes)</span> </span>{
    ......
  }
  <span class="hljs-comment">// 此处省略一堆 reconcileXXXXX 形式的函数，它们负责处理具体的 reconcile 逻辑</span>
  <span class="hljs-function">function <span class="hljs-title">reconcileChildFibers</span><span class="hljs-params">(returnFiber, currentFirstChild, newChild, lanes)</span> </span>{
    <span class="hljs-comment">// 这是一个逻辑分发器，它读取入参后，会经过一系列的条件判断，调用上方所定义的负责具体节点操作的函数</span>
  }

  <span class="hljs-comment">// 将总的 reconcileChildFibers 函数返回</span>
  <span class="hljs-keyword">return</span> reconcileChildFibers;
}
</code></pre>
<p data-nodeid="2456">由于原本的代码量着实巨大，感兴趣的同学可以点开<a href="https://github.com/facebook/react/blob/56e9feead0f91075ba0a4f725c9e4e343bca1c67/packages/react-reconciler/src/ReactChildFiber.old.js#L253" data-nodeid="2748">这个文件</a>查看细节，此处我仅针对与主流程强相关的逻辑为你总结以下要点：</p>
<ol data-nodeid="2457">
<li data-nodeid="2458">
<p data-nodeid="2459">关键的入参 shouldTrackSideEffects，意为“是否需要追踪副作用”，<strong data-nodeid="2755">因此 reconcileChildFibers 和 mountChildFibers 的不同，在于对副作用的处理不同</strong>；</p>
</li>
<li data-nodeid="2460">
<p data-nodeid="2461">ChildReconciler 中定义了大量如 placeXXX、deleteXXX、updateXXX、reconcileXXX 等这样的函数，这些函数覆盖了对 Fiber 节点的创建、增加、删除、修改等动作，将直接或间接地被 reconcileChildFibers 所调用；</p>
</li>
<li data-nodeid="2462">
<p data-nodeid="2463">ChildReconciler 的返回值是一个名为 reconcileChildFibers 的函数，这个函数是一个逻辑分发器，<strong data-nodeid="2762">它将根据入参的不同，执行不同的 Fiber 节点操作，最终返回不同的目标 Fiber 节点</strong>。</p>
</li>
</ol>
<p data-nodeid="2464">对于第 1 点，这里展开说说。对副作用的处理不同，到底是哪里不同？以 placeSingleChild 为例，以下是 placeSingleChild 的源码：</p>
<pre class="lang-java" data-nodeid="2465"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">placeSingleChild</span><span class="hljs-params">(newFiber)</span> </span>{
  <span class="hljs-keyword">if</span> (shouldTrackSideEffects &amp;&amp; newFiber.alternate === <span class="hljs-keyword">null</span>) {
    newFiber.flags = Placement;
  }
  <span class="hljs-keyword">return</span> newFiber;
}
</code></pre>
<p data-nodeid="2466">可以看出，一旦判断 shouldTrackSideEffects 为 false，那么下面所有的逻辑都不执行了，直接返回。那如果执行下去会发生什么呢？简而言之就是给 Fiber 节点打上一个叫“flags”的标记，像这样：</p>
<pre class="lang-java" data-nodeid="2467"><code data-language="java">newFiber.flags = Placement;
</code></pre>
<p data-nodeid="2468">这个名为 flags 的标记有何作用呢？</p>
<h4 data-nodeid="2469">小科普：flags 是什么</h4>
<p data-nodeid="2470">由于这里我引用的是 <a href="https://github.com/facebook/react/blob/56e9feead0f91075ba0a4f725c9e4e343bca1c67/packages/react-reconciler/src/ReactChildFiber.old.js#L253" data-nodeid="2770">v17.0.0 版本的源码</a>，属性名已经变更为 flags，但在更早一些的版本中，这个属性名叫“effectTag”。在时下的社区讨论中，effectTag 这个命名更常见，也更语义化，因此下文我将以 “effectTag”代指“flags”。</p>
<p data-nodeid="2471">Placement 这个 effectTag 的意义，是在渲染器执行时，也就是真实 DOM 渲染时，告诉渲染器：<strong data-nodeid="2789">我这里需要新增 DOM 节点</strong>。 effectTag 记录的是<strong data-nodeid="2790">副作用的类型</strong>，而<strong data-nodeid="2791">所谓“副作用”</strong>，React 给出的定义是“<strong data-nodeid="2792">数据获取、订阅或者修改 DOM</strong>”等动作。在这里，Placement 对应的显然是 DOM 相关的副作用操作。</p>
<p data-nodeid="2472">像 Placement 这样的副作用标识，还有很多，它们均以二进制常量的形式存在，下图我为你截取了局部（你可以在<a href="https://github.com/facebook/react/blob/1fb18e22ae66fdb1dc127347e169e73948778e5a/packages/react-reconciler/src/ReactSideEffectTags.js" data-nodeid="2796">这个文件</a>里查看 effectTag 的类型）：</p>
<p data-nodeid="2473"><img src="https://s0.lgstatic.com/i/image/M00/71/0B/CgqCHl-8xIyAZ3VoAADupBJcrgo966.png" alt="Drawing 7.png" data-nodeid="2800"></p>
<p data-nodeid="2474">回到我们的调用链路里来，由于 current 是 rootFiber，它不为 null，因此它将走入的是下图所高亮的这行逻辑。也就是说在 mountChildFibers 和 reconcileChildFibers 之间，它选择的是 <strong data-nodeid="2806">reconcileChildFibers</strong>：</p>
<p data-nodeid="2475"><img src="https://s0.lgstatic.com/i/image/M00/71/07/Ciqc1F-80U-AfncYAAEt69YE2-g951.png" alt="Drawing 8.png" data-nodeid="2809"></p>
<p data-nodeid="2476">结合前面的分析可知，reconcileChildFibers 是<code data-backticks="1" data-nodeid="2811">ChildReconciler(true)</code>的返回值。入参为 true，意味着其内部逻辑是允许追踪副作用的，因此“打 effectTag”这个动作将会生效。</p>
<p data-nodeid="2477">接下来进入 reconcileChildFibers 的逻辑，在 reconcileChildFibers 这个逻辑分发器中，会把 rootFiber 子节点的创建工作分发给 reconcileXXX 函数家族的一员——reconcileSingleElement 来处理，具体的调用形式如下图高亮处所示：</p>
<p data-nodeid="2478"><img src="https://s0.lgstatic.com/i/image/M00/71/07/Ciqc1F-80VaABnJCAACe4hcSiBM598.png" alt="Drawing 9.png" data-nodeid="2816"></p>
<p data-nodeid="2479">reconcileSingleElement 将基于 rootFiber 子节点的 ReactElement 对象信息，创建其对应的 FiberNode。这个过程中涉及的函数调用如下图高亮处所示：</p>
<p data-nodeid="2480"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80VyAC2P6AAJfHF2gzfs579.png" alt="Drawing 10.png" data-nodeid="2820"></p>
<p data-nodeid="2481">这里需要说明的一点是：<strong data-nodeid="2830">rootFiber 作为 Fiber 树的根节点</strong>，它并没有一个确切的 ReactElement 与之映射。结合 JSX 结构来看，<strong data-nodeid="2831">我们可以将其理解为是 JSX 中根组件的父节点</strong>。课时所给出的 Demo 中，组件编码如下：</p>
<pre class="lang-java" data-nodeid="2482"><code data-language="java">import React from "react";
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
<p data-nodeid="2483">可以看出，根组件是一个类型为 App 的函数组件，因此 <strong data-nodeid="2837">rootFiber 就是 App 的父节点</strong>。</p>
<p data-nodeid="2484">结合这个分析来看，图中的 _created4 是根据 rootFiber 的第一个子节点对应的 ReactElement 来创建的 Fiber 节点，那么它就是 <strong data-nodeid="2847">App 所对应的 Fiber 节点</strong>。现在我为你打印出运行时的 _created4 值，会发现确实如此：</p>
<p data-nodeid="2485"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80WaAXLPeAAD-OcP7y4o323.png" alt="Drawing 11.png" data-nodeid="2850"></p>
<p data-nodeid="2486">App 所对应的 Fiber 节点，将被 placeSingleChild 打上“Placement”（新增）的副作用标记，而后作为 reconcileChildFibers 函数的返回值，返回给下图中的 workInProgress.child：</p>
<p data-nodeid="2487"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80WyARnfDAAGNRsiaht8973.png" alt="Drawing 12.png" data-nodeid="2854"></p>
<p data-nodeid="2488">reconcileChildren 函数上下文里的 workInProgress 就是 rootFiber 节点。那么此时，我们就将新创建的 App Fiber 节点和 rootFiber 关联了起来，整个 Fiber 树如下图所示：</p>
<p data-nodeid="2489"><img src="https://s0.lgstatic.com/i/image/M00/71/3E/Ciqc1F-91MmARvQRAADFJC1K20o629.png" alt="3.png" data-nodeid="2858"></p>
<h3 data-nodeid="2490">Fiber 节点的创建过程梳理</h3>
<p data-nodeid="2491">分析完 App FiberNode 的创建过程，我们先不必急于继续往下走这个渲染链路。因为其实最关键的东西已经讲完了，剩余节点的创建只不过是对 performUnitOfWork、 beginWork 和 ChildReconciler 等相关逻辑的重复。</p>
<p data-nodeid="2492">刚刚这一通分析所涉及的调用栈很长，相信不少人如果是初读的话，过程中肯定不可避免地要反复回看，确认自己现在到底在调用栈的哪一环。这里为了方便你把握逻辑脉络，我将本讲讲解的 beginWork 所触发的调用流程总结进一张大图：</p>
<p data-nodeid="2493"><img src="https://s0.lgstatic.com/i/image/M00/71/47/Ciqc1F-97fSAYLUIAAGBjhvNylg581.png" alt="7.png" data-nodeid="2864"></p>
<h3 data-nodeid="2494">Fiber 树的构建过程</h3>
<p data-nodeid="2495">理解了 Fiber 节点的创建过程，就不难理解 Fiber 树的构建过程了。</p>
<p data-nodeid="2496">前面我们已经锲而不舍地研究了各路关键函数的源码逻辑，此时相信你已经能够将函数名与函数的工作内容做到对号入座。这里我们不必再纠结与源码的实现细节，可以直接从工作流程的角度来看后续节点的创建。</p>
<h4 data-nodeid="2497">循环创建新的 Fiber 节点</h4>
<p data-nodeid="2498">研究节点创建的工作流，我们的切入点是<code data-backticks="1" data-nodeid="2870">workLoopSync</code>这个函数。</p>
<p data-nodeid="2499">为什么选它？这里来复习一遍<code data-backticks="1" data-nodeid="2873">workLoopSync</code>会做什么：</p>
<pre class="lang-java" data-nodeid="2500"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">workLoopSync</span><span class="hljs-params">()</span> </span>{
  <span class="hljs-comment">// 若 workInProgress 不为空</span>
  <span class="hljs-keyword">while</span> (workInProgress !== <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 针对它执行 performUnitOfWork 方法</span>
    performUnitOfWork(workInProgress);
  }
}
</code></pre>
<p data-nodeid="2501"><strong data-nodeid="2883">它会循环地调用 performUnitOfWork</strong>，而 performUnitOfWork，开篇我们已经点到过它，其主要工作是“通过调用 beginWork，来实现新 Fiber 节点的创建”；它还有一个次要工作，<strong data-nodeid="2884">就是把新创建的这个 Fiber 节点的值更新到 workInProgress 变量里去</strong>。源码中的相关逻辑提取如下：</p>
<pre class="lang-java" data-nodeid="2502"><code data-language="java"><span class="hljs-comment">// 新建 Fiber 节点</span>
next = beginWork$<span class="hljs-number">1</span>(current, unitOfWork, subtreeRenderLanes);
<span class="hljs-comment">// 将新的 Fiber 节点赋值给 workInProgress</span>
<span class="hljs-keyword">if</span> (next === <span class="hljs-keyword">null</span>) {
  <span class="hljs-comment">// If this doesn't spawn new work, complete the current work.</span>
  completeUnitOfWork(unitOfWork);
} <span class="hljs-keyword">else</span> {
  workInProgress = next;
}
</code></pre>
<p data-nodeid="2503">如此便能够确保每次 performUnitOfWork 执行完毕后，当前的 <strong data-nodeid="2890">workInProgress 都存储着下一个需要被处理的节点，从而为下一次的 workLoopSync 循环做好准备</strong>。</p>
<p data-nodeid="2504">现在我在 workLoopSync 内部打个断点，尝试输出每一次获取到的 workInProgress 的值，workInProgress 值的变化过程如下图所示：</p>
<p data-nodeid="2505"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80ZuAA1HAAAEBle-yZFM332.png" alt="Drawing 15.png" data-nodeid="2894"></p>
<p data-nodeid="2506">共有 7 个节点，若你点击展开查看每个节点的内容，就会发现这 7 个节点其实分别是：</p>
<ul data-nodeid="2507">
<li data-nodeid="2508">
<p data-nodeid="2509">rootFiber（当前 Fiber 树的根节点）</p>
</li>
<li data-nodeid="2510">
<p data-nodeid="2511">App FiberNode（App 函数组件对应的节点）</p>
</li>
<li data-nodeid="2512">
<p data-nodeid="2513">class 为 App 的 DOM 元素对应的节点，其内容如下图所示</p>
</li>
</ul>
<p data-nodeid="2514"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80aSAF7MKAAEHjyZ0Xwk039.png" alt="Drawing 16.png" data-nodeid="2901"></p>
<ul data-nodeid="2515">
<li data-nodeid="2516">
<p data-nodeid="2517">class 为 container 的 DOM 元素对应的节点，其内容如下图所示</p>
</li>
</ul>
<p data-nodeid="2518"><img src="https://s0.lgstatic.com/i/image/M00/71/07/Ciqc1F-80aqAJId4AACkvKHjlTM377.png" alt="Drawing 17.png" data-nodeid="2905"></p>
<ul data-nodeid="2519">
<li data-nodeid="2520">
<p data-nodeid="2521">h1 标签对应的节点</p>
</li>
<li data-nodeid="2522">
<p data-nodeid="2523">第 1 个 p 标签对应的 FiberNode，内容为“我是第一段话”，如下图所示</p>
</li>
</ul>
<p data-nodeid="2524"><img src="https://s0.lgstatic.com/i/image/M00/71/07/Ciqc1F-80bGAGFKTAADArDpX9j4096.png" alt="Drawing 18.png" data-nodeid="2910"></p>
<ul data-nodeid="2525">
<li data-nodeid="2526">
<p data-nodeid="2527">第 2 个 p 标签对应的 FiberNode，内容为“我是第二段话”，如下图所示</p>
</li>
</ul>
<p data-nodeid="2528"><img src="https://s0.lgstatic.com/i/image/M00/71/12/CgqCHl-80biASe4KAAEZMaZTIY8632.png" alt="Drawing 19.png" data-nodeid="2914"></p>
<p data-nodeid="2529">结合这 7 个 FiberNode，再对照对照我们的 Demo：</p>
<pre class="lang-java" data-nodeid="2530"><code data-language="java">function App() {
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
</code></pre>
<p data-nodeid="2531"><strong data-nodeid="2920">你会发现组件自上而下，每一个非文本类型的 ReactElement 都有了它对应的 Fiber 节点</strong>。</p>
<blockquote data-nodeid="2532">
<p data-nodeid="2533">注：React 并不会为所有的文本类型 ReactElement 创建对应的 FiberNode，这是一种优化策略。是否需要创建 FiberNode，在源码中是通过<a href="https://github.com/facebook/react/blob/765e89b908206fe62feb10240604db224f38de7d/packages/react-reconciler/src/ReactFiberBeginWork.new.js#L1068" data-nodeid="2924">isDirectTextChild</a>这个变量来区分的。</p>
</blockquote>
<p data-nodeid="2534">这样一来，我们构建的这棵树里，就多出了不少 FiberNode，如下图所示：</p>
<p data-nodeid="2535"><img src="https://s0.lgstatic.com/i/image/M00/71/49/CgqCHl-91PKANLSRAACt8c-uYAk378.png" alt="4.png" data-nodeid="2929"></p>
<p data-nodeid="2536">Fiber 节点有是有了，但这些 Fiber 节点之间又是如何相互连接的呢？</p>
<h4 data-nodeid="2537">Fiber 节点间是如何连接的呢</h4>
<p data-nodeid="2538"><strong data-nodeid="2940">不同的 Fiber 节点之间，将通过 child、return、sibling 这 3 个属性建立关系</strong>，<strong data-nodeid="2941">其中 child、return 记录的是父子节点关系，而 sibling 记录的则是兄弟节点关系</strong>。</p>
<p data-nodeid="2539">这里我以 h1 这个元素对应的 Fiber 节点为例，给你展示下它是如何与其他节点相连接的。展开这个 Fiber 节点，对它的 child、 return、sibling 3 个属性作截取，如下图所示：</p>
<p data-nodeid="2540">child 属性为 null，说明 h1 节点没有子 Fiber 节点：</p>
<p data-nodeid="2541"><img src="https://s0.lgstatic.com/i/image/M00/71/13/CgqCHl-80d2AV6r7AABCQ4zzis4597.png" alt="Drawing 21.png" data-nodeid="2946"></p>
<p data-nodeid="2542">return 属性局部截图：</p>
<p data-nodeid="2543"><img src="https://s0.lgstatic.com/i/image/M00/71/07/Ciqc1F-80eOAMhlKAACxayioeh4810.png" alt="Drawing 22.png" data-nodeid="2950"></p>
<p data-nodeid="2544">sibling 属性局部截图：</p>
<p data-nodeid="2545"><img src="https://s0.lgstatic.com/i/image/M00/71/13/CgqCHl-80eiAJ6doAAClFZDD7jE642.png" alt="Drawing 23.png" data-nodeid="2954"></p>
<p data-nodeid="2546">可以看到，return 属性指向的是 class 为 container 的 div 节点，而 sibling 属性指向的是第 1 个 p 节点。结合 JSX 中的嵌套关系我们不难得知 ——<strong data-nodeid="2960">FiberNode 实例中，return 指向的是当前 Fiber 节点的父节点，而 sibling 指向的是当前节点的第 1 个兄弟节点</strong>。</p>
<p data-nodeid="2547">结合这 3 个属性所记录的节点间关系信息，我们可以轻松地将上面梳理出来的新 FiberNode 连接起来：</p>
<p data-nodeid="2548"><img src="https://s0.lgstatic.com/i/image/M00/71/3E/Ciqc1F-91RGAAygAAAEYVWI-PXg439.png" alt="5.png" data-nodeid="2964"></p>
<p data-nodeid="11373" class="">以上便是 workInProgress Fiber 树的最终形态了。从图中可以看出，虽然人们习惯上仍然将眼前的这个产物称为“Fiber 树”，但<strong data-nodeid="11379">它的数据结构本质其实已经从树变成了链表</strong>。</p>




<p data-nodeid="2550" class="te-preview-highlight">注意，在分析 Fiber 树的构建过程时，我们选取了 <strong data-nodeid="2980">beginWork</strong> 作为切入点，但整个 Fiber 树的构建过程中，并不是只有 beginWork 在工作。这其中，还穿插着 <strong data-nodeid="2981">completeWork</strong> 的工作。只有将 completeWork 和 beginWork 放在一起来看，你才能够真正理解，Fiber 架构下的“深度优先遍历”到底是怎么一回事。</p>
<h3 data-nodeid="2551">总结</h3>
<p data-nodeid="2552">通过本讲的学习，你掌握了 beginWork 的实现原理、理清了 Fiber 节点的创建链路，最终串联起了 Fiber 树的宏观构建过程。至此，你已经揽获了 render 阶段大半的知识，这一路道阻且难，胜在收获满满。</p>
<p data-nodeid="2553">下一讲，我们一方面将乘胜追击，继续探索 completeWork 的工作内容，将整个 render 阶段讲透；另一方面，我会带你快速地过一遍 commit 阶段的工作流，并基于此去串联由初始化、render、commit 所组成的完整渲染工作流，力求对整个 ReactDOM.render 所触发的渲染链路形成一个系统、通透的理解。</p>
<p data-nodeid="2554" class="">此外，在本讲的开头，我还给你留下了一个悬念，也就是“为什么需要两棵 Fiber 树”的问题。这个问题的答案，也将会随着我们对 Fiber 探索的深入，逐渐浮出水面。</p>

---

### 精选评论

##### **8542：
> react15是通过递归来diff的元素的。而react16的reactdom. render是通过同深度优先搜索来diff。组件内部的render可以用react. createelement生成虚拟dom，在最顶部将根虚拟dom和真实dom容器，传给reactdom. render()在初次渲染的时候虚拟dom会被处理为current树，他是一个fiber树，然后渲染即可。在更新的时候，worhloopsync使用while循环，不断的执行performunitofwork，用beginwork生成一个新的fiber节点，组装到workinprocess中，此时内存里面就有2棵fiber树，一个是current，一个是workinprocess树。他们通过相互替换比较的办法完成后续的diff

##### console_man：
> 老师，你好。目前有这样一个疑问。在一个页面中根据条件渲染组件A和B，初始化时默认显示A组件，点击按钮后显示B组件。请问这时是通过什么方式把B组件中的JSX转换为ReactElement的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; JSX转换为ReactElement不是运行时处理的，是编译时处理的。转换的方式都是通过编译来实现。

##### **5127：
> 赞，这块我还得跟着代码来一遍

##### **蓉：
> 举步维艰

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油哦

##### **强：
> 太棒了，结合源码和视频走一遍

##### **8542：
> 初始化的时候根据render的两个参数生成了一个对象fiberroot，他的current指向rootfiber他是后续fiber的根节点。并且用当前的current得出一个workinprogress来。进入render阶段：此时这个根workinprogress不为空，进入beginwork将他的子节点转成fiber节点，然后存在workinprogress中为下一次workloopsync的循环做准备，总之beginwork的作用就是将所有的子元素reactelement全部转成fiber节点就好了。这些fiber节点又通过他们各自属性return. children.sibli ng三个属性，相互关联，形成一颗fiber树。这个fiber树从本质来说，他就是一个链表，fiber链表

##### **6943：
> 太牛了，老师的讲解太通俗易懂了

##### **逸：
> 看了2遍，beginWork和workLoopSync的主要逻辑是构建fiber节点和fiber树，还没有真正映射到真实DOM上。

##### **康：
> 开始有点吃力了，给自己打个气

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油~要相信自己！

