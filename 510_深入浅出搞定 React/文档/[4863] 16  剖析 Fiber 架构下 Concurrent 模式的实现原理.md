<p data-nodeid="4694" class="">你好，欢迎来到第 16 讲，关于 Fiber 架构的实现原理和编码形态，其实我们已经洋洋洒洒地分析了 3 讲了。</p>
<p data-nodeid="4695">在过去的 3 讲里，通过对整个 ReactDOM.render 所触发的渲染链路进行了分析和串联，我们已经把 Fiber 架构在实现层面的大部分要点都过了一遍。刚讲过的这部分知识，一方面相对来说复杂度比较高，需要一些耐心反复地理解和消化；另一方面，本讲接下来要讲解的内容，也和它存在着较强的依赖关系，因此对这些前置知识的把握就显得尤为重要。</p>
<p data-nodeid="4696">下面我说几个函数，帮你检验一下自己的学习效果：</p>
<ol data-nodeid="4697">
<li data-nodeid="4698">
<p data-nodeid="4699">performSyncWorkOnRoot</p>
</li>
<li data-nodeid="4700">
<p data-nodeid="4701">workLoopSync</p>
</li>
<li data-nodeid="4702">
<p data-nodeid="4703">performUnitOfWork</p>
</li>
<li data-nodeid="4704">
<p data-nodeid="4705">beginWork</p>
</li>
<li data-nodeid="4706">
<p data-nodeid="4707">completeWork</p>
</li>
<li data-nodeid="4708">
<p data-nodeid="4709">completeUnitOfWork</p>
</li>
<li data-nodeid="4710">
<p data-nodeid="4711">reconcileChildFibers</p>
</li>
</ol>
<p data-nodeid="4712">如果你对这些函数的执行时机和工作内容仍然感到不那么熟悉，那么不妨回到前 3 个课时里，结合案例和源码，重新捋顺一遍自己的思路，再回来续上你的知识链路。在接下来的讲解中，若对以上方法及其相关逻辑有所涉及，我将不再重复赘述。</p>
<p data-nodeid="4713">本讲我将带你去认识 Fiber 架构最迷人的那一面——Concurrent 模式（异步渲染）下的“<strong data-nodeid="4879">时间切片</strong>”和“<strong data-nodeid="4880">优先级</strong>”实现。</p>
<p data-nodeid="4714">在切入正题之前，我首先会回答上一讲遗留下来的“两棵树”问题。“两棵树”之间的合作模式足以将挂载过程和更新过程联系起来，对于本讲来说，是一个不错的学习切入点。</p>
<h3 data-nodeid="4715">current 树 与 workInProgress 树：“双缓冲”模式在 Fiber 架构下的实现</h3>
<h4 data-nodeid="4716">什么是“双缓冲”模式</h4>
<p data-nodeid="77176" class="">“双缓冲”模式其实是一种在游戏领域由来已久的经典设计模式。为了帮助你快速理解它，这里我先举一个生活中的例子：假如你去看一场总时长只有 1 个小时的话剧，这场话剧中场不休息，需要不间断地演出。</p>






































<p data-nodeid="81094" class="">按照剧情的需求，半个小时处需要一次转场。所谓转场，就是说话剧舞台的灯光、布景、氛围等全部要切换到另一种风格里去。在不中断演出的情况下，想要实现转场，怎么办呢？场务工作做得再快，也要十几二十分钟，这对一场时长 1 小时的话剧来说，实在太漫长了。观众也无法接受这样的剧情“卡顿”体验。</p>





















<p data-nodeid="4719">有一种解法，那就是<strong data-nodeid="4891">准备两个舞台</strong>来做这场戏，当第一个舞台处于使用中时，第二个舞台的布局已经完成。这样当第一个舞台的表演结束时，只需要把第一个舞台的灯光灭掉，第二个舞台的灯光亮起，就可以做到剧情的无缝衔接了。</p>
<p data-nodeid="83706" class="te-preview-highlight">事实上，在真实的话剧中，我们也确实常常看到这样的画面——演员从舞台的左侧走到了右侧，灯光一切换，就从卧室（左侧舞台）走到了公园（右侧舞台）；又从公园（右侧舞台）走到了办公室（左侧舞台）。左侧舞台的布景从卧室变成了办公室，这个过程正是在演员利用右侧舞台表演时完成的。</p>


<p data-nodeid="4721">在这个过程中，我们可以认为，<strong data-nodeid="4898">左侧舞台和右侧舞台分别是两套缓冲数据，而呈现在观众眼前的连贯画面，就是不同的缓冲数据交替被读取后的结果</strong>。</p>
<p data-nodeid="4722">在计算机图形领域，通过让图形硬件交替读取两套缓冲数据，可以实现画面的无缝切换，减少视觉效果上的抖动甚至卡顿。而在 React 中，双缓冲模式的主要利好，则是<strong data-nodeid="4904">能够帮我们较大限度地实现 Fiber 节点的复用</strong>，从而减少性能方面的开销。</p>
<h4 data-nodeid="4723">current 树与 workInProgress 树之间是如何“相互利用”的</h4>
<p data-nodeid="4724">在 React 中，current 树与 workInProgress 树，两棵树可以对标“双缓冲”模式下的两套缓冲数据：当 current 树呈现在用户眼前时，所有的更新都会由 workInProgress 树来承接。workInProgress 树将会在用户看不到的地方（内存里）悄悄地完成所有改变，直到“灯光”打到它身上，也就是 current 指针指向它的时候，此时就意味着 commit 阶段已经执行完毕，workInProgress 树变成了那棵呈现在界面上的 current 树。</p>
<p data-nodeid="4725">接下来我将用一个 Demo，带你切身感受一把 workInProgress 树和 current 树“相互利用”的过程。代码如下：</p>
<pre class="lang-js" data-nodeid="4726"><code data-language="js"><span class="hljs-keyword">import</span> { useState } <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">App</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> [state, setState] = useState(<span class="hljs-number">0</span>)
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{()</span> =&gt;</span> { setState(state + 1) }} className="container"&gt;
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">style</span>=<span class="hljs-string">{{</span> <span class="hljs-attr">width:</span> <span class="hljs-attr">128</span>, <span class="hljs-attr">textAlign:</span> '<span class="hljs-attr">center</span>' }}&gt;</span>
          {state}
        <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}

<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> App;
</code></pre>
<p data-nodeid="4727">这个组件挂载后呈现出的界面很简单，就是一个数字 0，如下图所示：</p>
<p data-nodeid="4728"><img src="https://s0.lgstatic.com/i/image/M00/73/AB/CgqCHl_GIlaAHn_FAAAchawzt4s009.png" alt="Drawing 1.png" data-nodeid="4911"></p>
<p data-nodeid="4729">每点击数字 0 一下，它的值就会 +1，这就是我们的更新动作。</p>
<h5 data-nodeid="4730">挂载后的 Fiber &nbsp;树</h5>
<p data-nodeid="4731">关于 Fiber 树的构建过程，前面已经详细讲解过，这里不再重复。下面我直接为你展示挂载时的 render 阶段结束后，commit 执行前，两棵 Fiber 树的形态，如下图所示：</p>
<p data-nodeid="4732"><img src="https://s0.lgstatic.com/i/image/M00/73/AD/Ciqc1F_GK5KAPsNRAADXbk-KIQg893.png" alt="图片14.png" data-nodeid="4917"></p>
<p data-nodeid="4733">待 commit 阶段完成后，右侧的 workInProgress 树对应的 DOM 树就被真正渲染到了页面上，此时 current 指针会指向 workInProgress 树：</p>
<p data-nodeid="4734"><img src="https://s0.lgstatic.com/i/image/M00/73/AB/CgqCHl_GImqABNjLAACxddF2jhY942.png" alt="Drawing 5.png" data-nodeid="4921"></p>
<p data-nodeid="4735">由于挂载是一个从无到有的过程，在这个过程中我们是在不断地创建新节点，因此还谈不上什么“节点复用”。节点复用要到更新过程中去看。</p>
<h5 data-nodeid="4736">第一次更新</h5>
<p data-nodeid="4737">现在我点击数字 0，触发一次更新。这次更新中，下图高亮的 rootFiber 节点就会被复用：</p>
<p data-nodeid="4738"><img src="https://s0.lgstatic.com/i/image/M00/73/AB/CgqCHl_GInGAAhaiAADrvJFiOdA159.png" alt="Drawing 7.png" data-nodeid="4927"></p>
<p data-nodeid="4739">这段复用的逻辑在 beginWork 调用链路中的 createWorkInProgress 方法里。这里我为你截取了 createWorkInProgress 方法里面一段非常关键的逻辑，请看下图：</p>
<p data-nodeid="4740"><img src="https://s0.lgstatic.com/i/image/M00/73/AB/CgqCHl_GIneAPBZcAAKQF1HrIv0329.png" alt="Drawing 8.png" data-nodeid="4931"></p>
<p data-nodeid="4741">在 createWorkInProgress 方法中，会先取当前节点的 alternate 属性，将其记为 workInProgress 节点。对于 rootFiber 节点来说，它的 alternate 属性，其实就是上一棵 current 树的 rootFiber，如下图高亮部分所示：</p>
<p data-nodeid="4742"><img src="https://s0.lgstatic.com/i/image/M00/73/A0/Ciqc1F_GIn2AK_7QAADrvJFiOdA218.png" alt="Drawing 10.png" data-nodeid="4935"></p>
<p data-nodeid="4743"><strong data-nodeid="4940">当检查到上一棵 current 树的 rootFiber 存在时，React 会直接复用这个节点，让它作为下一棵 workInProgress 的节点存在下去</strong>，也就是说会走进 createWorkInProgress 的 else 逻辑里去。如果它和目标的 workInProgress 节点之间存在差异，直接在该节点上修改属性、使其与目标节点一致即可，而不必再创建新的 Fiber 节点。</p>
<p data-nodeid="4744">至于剩下的 App、div、p 等节点，由于没有对应的 alternate 节点存在，因此它们的 createWorkInProgress 调用会走进下图高亮处的逻辑中：</p>
<p data-nodeid="4745"><img src="https://s0.lgstatic.com/i/image/M00/73/A0/Ciqc1F_GIoSAIIE5AAJG_0ANgrg415.png" alt="Drawing 11.png" data-nodeid="4944"></p>
<p data-nodeid="4746">在这段逻辑里，将调用 createFiber 来新建一个 FiberNode。</p>
<p data-nodeid="4747">第一次更新结束后，我们会得到一棵新的 workInProgress Fiber 树，current 指针最后将会指向这棵新的 workInProgress Fiber 树，如下图所示：</p>
<p data-nodeid="4748"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIouAVjKDAAEwJVqKwHY353.png" alt="Drawing 13.png" data-nodeid="4949"></p>
<h5 data-nodeid="4749">第二次更新</h5>
<p data-nodeid="4750">接下来我们再次点击数字 1，触发 state 的第二次更新。</p>
<p data-nodeid="4751">在这次更新中，current 树中的每一个 alternate 属性都不为空（如上图所示）。因此每次通过 beginWork 触发 createWorkInProgress 调用时，都会一致地走入 else 里面的逻辑，也就是直接复用现成的节点。</p>
<p data-nodeid="4752">以上便是 current 树和 work 树相互“打配合”，实现节点复用的过程。</p>
<h3 data-nodeid="4753">更新链路要素拆解</h3>
<p data-nodeid="4754">在上一讲，我们已经学习了挂载阶段的渲染链路。同步模式下的更新链路与挂载链路的 render 阶段基本是一致的，都是通过 performSyncWorkOnRoot 来触发包括 beginWork、completeWork 在内的深度优先搜索过程。这里我为你展示一个更新过程的调用栈，请看下图：</p>
<p data-nodeid="4755"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIpSAfb9qAACwnR8L9Po005.png" alt="Drawing 14.png" data-nodeid="4958"></p>
<p data-nodeid="4756">你会发现还是熟悉的配方，还是原来的味道。<strong data-nodeid="4968">其实，挂载可以理解为一种特殊的更新，ReactDOM.render 和 setState 一样，也是一种触发更新的姿势</strong>。在 React 中，ReactDOM.render、setState、useState 等方法都是可以触发更新的，这些方法发起的调用链路很相似，是因为它们最后“殊途同归”，<strong data-nodeid="4969">都会通过创建 update 对象来进入同一套更新工作流</strong>。</p>
<h4 data-nodeid="4757">update 的创建</h4>
<p data-nodeid="4758">接下来我继续以开篇的 Demo 为例，为你拆解更新链路中的要素。在点击数字后，点击相关的回调被执行，它首先触发的是 dispatchAction 这个方法，如下图所示：</p>
<p data-nodeid="4759"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIpqAFVQWAAE4Rdg6o0g228.png" alt="Drawing 15.png" data-nodeid="4974"></p>
<p data-nodeid="4760">请你关注图中两处标红的函数调用，你会看到 dispatchAction 方法在 performSyncWorkOnRoot 的左边。也就是说整体的更新链路应该是这样的：</p>
<p data-nodeid="4761"><img src="https://s0.lgstatic.com/i/image/M00/73/A1/Ciqc1F_GIqKAP_3fAABp3EtlwDk160.png" alt="Drawing 16.png" data-nodeid="4978"></p>
<p data-nodeid="4762">dispatchAction 中，会完成 update 对象的创建，如下图标红处所示：</p>
<p data-nodeid="4763"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIqiAOf4aAAQVWAWjjt0722.png" alt="Drawing 17.png" data-nodeid="4982"></p>
<h4 data-nodeid="4764">从 update 对象到 scheduleUpdateOnFiber</h4>
<p data-nodeid="4765">等等，这段逻辑你是否觉得似曾相识？如果你对 ReactDOM.render 系列的第一课时还有印象的话，我希望你能回忆起 updateContainer 这个方法。在 updateContainer 中，React 曾经有过性质一模一样的行为，这里我为你截取了 updateContainer 函数中的相关逻辑：</p>
<p data-nodeid="4766"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIrCAJiUaAACZL_H8ts8659.png" alt="Drawing 18.png" data-nodeid="4987"></p>
<p data-nodeid="4767">图中这一段代码的逻辑是非常清晰的，以 enqueueUpdate 为界，它一共做了以下三件事。</p>
<ol data-nodeid="4768">
<li data-nodeid="4769">
<p data-nodeid="4770">enqueueUpdate 之前：<strong data-nodeid="4994">创建 update</strong>。</p>
</li>
<li data-nodeid="4771">
<p data-nodeid="4772">enqueueUpdate 调用：<strong data-nodeid="5004">将 update 入队</strong>。这里简单说下，每一个 Fiber 节点都会有一个属于它自己的 updateQueue，用于存储多个更新，这个 updateQueue 是以链表的形式存在的。在 render 阶段，<strong data-nodeid="5005">updateQueue 的内容会成为 render 阶段计算 Fiber 节点的新 state 的依据</strong>。</p>
</li>
<li data-nodeid="4773">
<p data-nodeid="4774">scheduleUpdateOnFiber：<strong data-nodeid="5011">调度 update</strong>。如果你对之前学过的知识还有印象，会记得同步挂载链路中，这个方法后面紧跟的就是 performSyncWorkOnRoot 所触发的 render 阶段，如下图所示：</p>
</li>
</ol>
<p data-nodeid="4775"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIrmAO5UJAABDaDLvIyM528.png" alt="Drawing 19.png" data-nodeid="5014"></p>
<p data-nodeid="4776">现在我们再回过头来看 dispatchAction 的逻辑，你会发现 dispatchAction 里面同样有对这三个动作的处理。上面我对 dispatchAction 的局部截图，包含了对 update 对象的创建和入队处理。dispatchAction 的更新调度动作，在函数的末尾，如下图所示：</p>
<p data-nodeid="4777"><img src="https://s0.lgstatic.com/i/image/M00/73/AC/CgqCHl_GIsGADUaBAAAYE3Ps56g927.png" alt="Drawing 20.png" data-nodeid="5018"></p>
<p data-nodeid="4778">这里有一个点需要提示一下：dispatchAction 中，<strong data-nodeid="5028">调度的是当前触发更新的节点</strong>，这一点和挂载过程需要区分开来。在挂载过程中，updateContainer 会直接调度根节点。其实，对于更新这种场景来说，<strong data-nodeid="5029">大部分的更新动作确实都不是由根节点触发的</strong>，而 render 阶段的起点则是根节点。因此在 scheduleUpdateOnFiber 中，有这样一个方法，见下图标红处：</p>
<p data-nodeid="4779"><img src="https://s0.lgstatic.com/i/image/M00/73/A1/Ciqc1F_GIseASLuzAAVy6vkOrOA579.png" alt="Drawing 21.png" data-nodeid="5032"></p>
<p data-nodeid="4780">markUpdateLaneFromFiberToRoot 将会从当前 Fiber 节点开始，向上遍历直至根节点，并将根节点返回。</p>
<h4 data-nodeid="4781">scheduleUpdateOnFiber 如何区分同步还是异步？</h4>
<p data-nodeid="4782">如果你对之前学过的同步渲染链路分析还有印象，相信你对下面这段逻辑不会陌生：</p>
<p data-nodeid="4783"><img src="https://s0.lgstatic.com/i/image/M00/73/A1/Ciqc1F_GItCAerpyAAQXnhAd3k4018.png" alt="Drawing 22.png" data-nodeid="5038"></p>
<p data-nodeid="4784">这是 scheduleUpdateOnFiber 中的一段逻辑。在同步的渲染链路中，lane === SyncLane 这个条件是成立的，因此会直接进入 performSyncWorkOnRoot 的逻辑，开启同步的 render 流程；而在异步渲染模式下，则将进入 else 的逻辑。</p>
<p data-nodeid="4785">在 else 中，需要引起你注意的是 ensureRootIsScheduled 这个方法，该方法很关键，它将决定如何开启当前更新所对应的 render 阶段。在 ensureRootIsScheduled 中，有<a href="https://github.com/facebook/react/blob/b6df4417c79c11cfb44f965fab55b573882b1d54/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L602" data-nodeid="5043">这样一段核心逻</a>辑（解析在注释里）：</p>
<pre class="lang-java" data-nodeid="4786"><code data-language="java"><span class="hljs-keyword">if</span> (newCallbackPriority === SyncLanePriority) {
    <span class="hljs-comment">// 同步更新的 render 入口</span>
    newCallbackNode = scheduleSyncCallback(performSyncWorkOnRoot.bind(<span class="hljs-keyword">null</span>, root));
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 将当前任务的 lane 优先级转换为 scheduler 可理解的优先级</span>
    <span class="hljs-keyword">var</span> schedulerPriorityLevel = lanePriorityToSchedulerPriority(newCallbackPriority);
    <span class="hljs-comment">// 异步更新的 render 入口</span>
    newCallbackNode = scheduleCallback(schedulerPriorityLevel, performConcurrentWorkOnRoot.bind(<span class="hljs-keyword">null</span>, root));
  }
</code></pre>
<p data-nodeid="4787">请你关注<strong data-nodeid="5054">performSyncWorkOnRoot 和 performConcurrentWorkOnRoot</strong> 这两个方法：<strong data-nodeid="5055">前者是同步更新模式下的 render 阶段入口；而后者是异步模式下的 render 阶段入口</strong>。</p>
<p data-nodeid="4788">从这段逻辑中我们可以看出，React 会以当前更新任务的优先级类型为依据，决定接下来是调度 performSyncWorkOnRoot 还是 performConcurrentWorkOnRoot。这里调度任务用到的函数分别是 scheduleSyncCallback 和 scheduleCallback，<strong data-nodeid="5065">这两个函数在内部都是通过调用 unstable_scheduleCallback 方法来执行任务调度的</strong>。而 unstable_scheduleCallback 正是 Scheduler（调度器）中导出的一个核心方法，也是本讲的一个重点。</p>
<p data-nodeid="4789">在解读 unstable_scheduleCallback 的工作原理之前，我们先来一起认识一下 Scheduler。</p>
<h3 data-nodeid="4790">Scheduler——“时间切片”与“优先级”的幕后推手</h3>
<p data-nodeid="4791">Scheduler 从架构上来看，是 Fiber 架构分层中的“调度层”；从实现上来看，它并非一段内嵌的逻辑，而是一个与 react-dom 同级的文件夹，如下图所示，其中收敛了所有相对通用的调度逻辑：</p>
<p data-nodeid="4792"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIt-AVQdjAAEHWIrq_Po945.png" alt="Drawing 23.png" data-nodeid="5073"></p>
<p data-nodeid="4793">通过前面的学习，我们已经知道 Fiber 架构下的异步渲染（即 Concurrent 模式）的核心特征分别是“<strong data-nodeid="5083">时间切片</strong>”与“<strong data-nodeid="5084">优先级调度</strong>”。而这两点，也正是 Scheduler 的核心能力。接下来，我们就以这两个特征为线索，解锁 Scheduler 的工作原理。</p>
<h4 data-nodeid="4794">结合 React 调用栈，理解时间切片现象</h4>
<p data-nodeid="4795">在理解时间切片的实现原理之前，我们首先要搞清楚时间切片是一种什么样的现象。</p>
<p data-nodeid="4796">在 ReactDOM.render 相关的课时中，我曾经强调过，同步渲染模式下的 render 阶段，是一个同步的、深度优先搜索的过程。同步的过程会带来什么样的麻烦呢？在第 12 讲中，大家已经从理论层面初步认识过这个问题。现在，我们直接通过调用栈来理解它，下面是一个渲染工作量相对比较大的 React Demo，代码如下：</p>
<pre class="lang-js" data-nodeid="4797"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">App</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> arr = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Array</span>(<span class="hljs-number">1000</span>).fill(<span class="hljs-number">0</span>)
  <span class="hljs-keyword">const</span> renderContent = arr.map(
    <span class="hljs-function">(<span class="hljs-params">i, index</span>) =&gt;</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">style</span>=<span class="hljs-string">{{</span> <span class="hljs-attr">width:</span> <span class="hljs-attr">128</span>, <span class="hljs-attr">textAlign:</span> '<span class="hljs-attr">center</span>' }}&gt;</span>{`测试文本第${index}行`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  )
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"container"</span>&gt;</span>
        {
          renderContent
        }
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> App;
</code></pre>
<p data-nodeid="4798">这个 App 组件会在界面上渲染出 1000 行文本，局部效果如下图所示：</p>
<p data-nodeid="4799"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIuyABtYCAABNHJ0zT3I546.png" alt="Drawing 24.png" data-nodeid="5091"></p>
<p data-nodeid="4800">当我使用 ReactDOM.render 来渲染这个长列表时，它的调用栈如下图所示：</p>
<p data-nodeid="4801"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIvOALbOqAAOn8pi1lpw155.png" alt="Drawing 25.png" data-nodeid="5095"></p>
<p data-nodeid="4802">在这张图中，你就不必再重复去关注 beginWork、completeWork 之流了，请把目光放在调用栈的上层，也就是图中标红的地方——一个不间断的灰色“Task”长条，对浏览器来说就意味着是一个<strong data-nodeid="5101">不可中断</strong>的任务。</p>
<p data-nodeid="4803">在我的浏览器上，这个 Task 的执行时长在 130ms 以上（将鼠标悬浮在 Task 长条上就可以查看执行时长）。而<strong data-nodeid="5115">浏览器的刷新频率为 60Hz，也就是说每 16.6ms 就会刷新一次</strong>。在这 16.6ms 里，除了 JS 线程外，渲染线程也是有工作要处理的，<strong data-nodeid="5116">但超长的 Task 显然会挤占渲染线程的工作时间</strong>，引起“掉帧”，进而带来卡顿的风险，这也正是第 12 讲中所提到的“<strong data-nodeid="5117">JS 对主线程的超时占用</strong>”问题。</p>
<p data-nodeid="4804">若将 ReactDOM.render 调用改为 createRoot 调用（即开启 Concurrent 模式），调用栈就会变成下面这样：</p>
<p data-nodeid="4805"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIvuATEYcAALRQfAuBFI173.png" alt="Drawing 26.png" data-nodeid="5121"></p>
<p data-nodeid="4806">请继续将你的注意力放在顶层的 Task 长条上。</p>
<p data-nodeid="4807">你会发现那一个不间断的 Task 长条（大任务），如今像是被“切”过了一样，已经变成了多个断断续续的 Task “短条”（小任务），单个短 Task 的执行时长在我的浏览器中是 5ms 左右。这些短 Task 的工作量加起来，和之前长 Task 工作量是一样的。但短 Task 之间留出的时间缝隙，却给了浏览器喘息的机会，这就是所谓的“时间切片”效果。</p>
<h4 data-nodeid="4808">时间切片是如何实现的？</h4>
<p data-nodeid="4809">在同步渲染中，循环创建 Fiber 节点、构建 Fiber 树的过程是由 <strong data-nodeid="5130">workLoopSync</strong> 函数来触发的。这里我们来复习一下 workLoopSync 的源码，请看下图：</p>
<p data-nodeid="4810"><img src="https://s0.lgstatic.com/i/image/M00/73/AD/CgqCHl_GIwKAHTjDAACgXrFd5c0290.png" alt="Drawing 27.png" data-nodeid="5133"></p>
<p data-nodeid="4811">在 workLoopSync 中，只要 workInProgress 不为空，while 循环就不会结束，它所触发的是一个同步的 performUnitOfWork 循环调用过程。</p>
<p data-nodeid="4812">而在异步渲染模式下，这个循环是由 <strong data-nodeid="5140">workLoopConcurrent</strong> 来开启的。workLoopConcurrent 的工作内容和 workLoopSync 非常相似，仅仅在循环判断上有一处不同，请注意下图源码中标红部分：</p>
<p data-nodeid="4813"><img src="https://s0.lgstatic.com/i/image/M00/73/AD/CgqCHl_GIwiAAsOJAADS76o4FHc058.png" alt="Drawing 28.png" data-nodeid="5143"></p>
<p data-nodeid="4814">shouldYield 直译过来的话是“需要让出”。顾名思义，<strong data-nodeid="5149">当 shouldYield() 调用返回为 true 时，就说明当前需要对主线程进行让出了，此时 whille 循环的判断条件整体为 false，while 循环将不再继续</strong>。</p>
<p data-nodeid="4815">那么这个 shouldYield 又是何方神圣呢？在源码中，我们可以寻得这样两行赋值语句：</p>
<pre class="lang-java" data-nodeid="4816"><code data-language="java"><span class="hljs-keyword">var</span> Scheduler_shouldYield = Scheduler.unstable_shouldYield,
......
<span class="hljs-keyword">var</span> shouldYield = Scheduler_shouldYield;
</code></pre>
<p data-nodeid="4817">从这两行代码中我们可以看出，shouldYield 的本体其实是 <strong data-nodeid="5160">Scheduler.unstable_shouldYield</strong>，也就是 Scheduler 包中导出的 unstable_shouldYield 方法，该方法本身比较简单。其源码如下图标红处所示：</p>
<p data-nodeid="4818"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIxeAJas8AAEiY1Hfbsw871.png" alt="Drawing 29.png" data-nodeid="5163"></p>
<p data-nodeid="4819">其中 unstable_now 这里实际取的就是 performance.now() 的值，即“<strong data-nodeid="5175">当前时间</strong>”。那么 deadline 又是什么呢？它可以被理解为<strong data-nodeid="5176">当前时间切片的到期时间</strong>，它的计算过程在 Scheduler 包中的 performWorkUntilDeadline 方法里可以找到，也就是下图的标红部分：</p>
<p data-nodeid="4820"><img src="https://s0.lgstatic.com/i/image/M00/73/A2/Ciqc1F_GIx-AL94QAAHs2HJQXfA804.png" alt="Drawing 30.png" data-nodeid="5179"></p>
<p data-nodeid="4821">在这行算式里，currentTime 是当前时间，yieldInterval 是<strong data-nodeid="5185">时间切片的长度</strong>。注意，时间切片的长度并不是一个常量，它是由 React 根据浏览器的帧率大小计算所得出来的，与浏览器的性能有关。</p>
<p data-nodeid="4822">现在我们来总结一下时间切片的实现原理：React 会根据浏览器的帧率，计算出时间切片的大小，并结合当前时间计算出每一个切片的到期时间。在 workLoopConcurrent 中，while 循环每次执行前，会调用 shouldYield 函数来询问当前时间切片是否到期，若已到期，则结束循环、出让主线程的控制权。</p>
<h4 data-nodeid="4823">优先级调度是如何实现的</h4>
<p data-nodeid="4824">在“更新链路要素拆解”这一小节的末尾，我们已经知道，无论是 scheduleSyncCallback 还是 scheduleCallback，最终都是通过调用 <strong data-nodeid="5201">unstable_scheduleCallback</strong> 来发起调度的。unstable_scheduleCallback 是 Scheduler 导出的一个核心方法，它将<strong data-nodeid="5202">结合任务的优先级信息为其执行不同的调度逻辑</strong>。</p>
<p data-nodeid="4825">接下来我们就结合源码，一起看看这个过程是如何实现的（解析在注释里）。</p>
<pre class="lang-java" data-nodeid="4826"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">unstable_scheduleCallback</span><span class="hljs-params">(priorityLevel, callback, options)</span> </span>{
  <span class="hljs-comment">// 获取当前时间</span>
  <span class="hljs-keyword">var</span> currentTime = <span class="hljs-keyword">exports</span>.unstable_now();
  <span class="hljs-comment">// 声明 startTime，startTime 是任务的预期开始时间</span>
  <span class="hljs-keyword">var</span> startTime;
  <span class="hljs-comment">// 以下是对 options 入参的处理</span>
  <span class="hljs-keyword">if</span> (typeof options === <span class="hljs-string">'object'</span> &amp;&amp; options !== <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">var</span> delay = options.delay;

    <span class="hljs-comment">// 若入参规定了延迟时间，则累加延迟时间</span>
    <span class="hljs-keyword">if</span> (typeof delay === <span class="hljs-string">'number'</span> &amp;&amp; delay &gt; <span class="hljs-number">0</span>) {
      startTime = currentTime + delay;
    } <span class="hljs-keyword">else</span> {
      startTime = currentTime;
    }
  } <span class="hljs-keyword">else</span> {
    startTime = currentTime;
  }
  <span class="hljs-comment">// timeout 是 expirationTime 的计算依据</span>
  <span class="hljs-keyword">var</span> timeout;
  <span class="hljs-comment">// 根据 priorityLevel，确定 timeout 的值</span>
  <span class="hljs-keyword">switch</span> (priorityLevel) {
    <span class="hljs-keyword">case</span> ImmediatePriority:
      timeout = IMMEDIATE_PRIORITY_TIMEOUT;
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">case</span> UserBlockingPriority:
      timeout = USER_BLOCKING_PRIORITY_TIMEOUT;
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">case</span> IdlePriority:
      timeout = IDLE_PRIORITY_TIMEOUT;
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">case</span> LowPriority:
      timeout = LOW_PRIORITY_TIMEOUT;
      <span class="hljs-keyword">break</span>;
    <span class="hljs-keyword">case</span> NormalPriority:
    <span class="hljs-keyword">default</span>:
      timeout = NORMAL_PRIORITY_TIMEOUT;
      <span class="hljs-keyword">break</span>;
  }
  <span class="hljs-comment">// 优先级越高，timout 越小，expirationTime 越小</span>
  <span class="hljs-keyword">var</span> expirationTime = startTime + timeout;

  <span class="hljs-comment">// 创建 task 对象</span>
  <span class="hljs-keyword">var</span> newTask = {
    id: taskIdCounter++,
    callback: callback,
    priorityLevel: priorityLevel,
    startTime: startTime,
    expirationTime: expirationTime,
    sortIndex: -<span class="hljs-number">1</span>
  };

  {
    newTask.isQueued = <span class="hljs-keyword">false</span>;
  }
  <span class="hljs-comment">// 若当前时间小于开始时间，说明该任务可延时执行(未过期）</span>
  <span class="hljs-keyword">if</span> (startTime &gt; currentTime) {
    <span class="hljs-comment">// 将未过期任务推入 "timerQueue"</span>
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);

    <span class="hljs-comment">// 若 taskQueue 中没有可执行的任务，而当前任务又是 timerQueue 中的第一个任务</span>
    <span class="hljs-keyword">if</span> (peek(taskQueue) === <span class="hljs-keyword">null</span> &amp;&amp; newTask === peek(timerQueue)) {
      ......
          <span class="hljs-comment">// 那么就派发一个延时任务，这个延时任务用于检查当前任务是否过期</span>
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// else 里处理的是当前时间大于 startTime 的情况，说明这个任务已过期</span>
    newTask.sortIndex = expirationTime;
    <span class="hljs-comment">// 过期的任务会被推入 taskQueue</span>
    push(taskQueue, newTask);
    ......

    <span class="hljs-comment">// 执行 taskQueue 中的任务</span>
    requestHostCallback(flushWork);
  }
  <span class="hljs-keyword">return</span> newTask;
}
</code></pre>
<p data-nodeid="4827">从源码中我们可以看出，unstable_scheduleCallback 的主要工作是针对当前任务创建一个 task，然后结合 startTime 信息将这个 task 推入 <strong data-nodeid="5215">timerQueue</strong> 或 <strong data-nodeid="5216">taskQueue</strong>，最后根据 timerQueue 和 taskQueue 的情况，执行延时任务或即时任务。</p>
<p data-nodeid="4828">要想理解这个过程，首先要搞清楚以下几个概念。</p>
<ul data-nodeid="4829">
<li data-nodeid="4830">
<p data-nodeid="4831"><strong data-nodeid="5222">startTime</strong>：任务的开始时间。</p>
</li>
<li data-nodeid="4832">
<p data-nodeid="4833"><strong data-nodeid="5227">expirationTime</strong>：这是一个和优先级相关的值，expirationTime 越小，任务的优先级就越高。</p>
</li>
<li data-nodeid="4834">
<p data-nodeid="4835"><strong data-nodeid="5236">timerQueue</strong>：一个以 startTime 为排序依据的<strong data-nodeid="5237">小顶堆</strong>，它存储的是 startTime 大于当前时间（也就是待执行）的任务。</p>
</li>
<li data-nodeid="4836">
<p data-nodeid="4837"><strong data-nodeid="5246">taskQueue</strong>：一个以 expirationTime 为排序依据的<strong data-nodeid="5247">小顶堆</strong>，它存储的是 startTime 小于当前时间（也就是已过期）的任务。</p>
</li>
</ul>
<p data-nodeid="4838">这里的“小顶堆”概念可能会触及一部分同学的知识盲区，我简单解释下：堆是一种特殊的<a href="https://baike.baidu.com/item/%E5%AE%8C%E5%85%A8%E4%BA%8C%E5%8F%89%E6%A0%91" data-nodeid="5251">完全二叉树</a>。如果对一棵完全二叉树来说，它每个结点的结点值都不大于其左右孩子的结点值，这样的完全二叉树就叫“<a href="https://baike.baidu.com/item/%E6%9C%80%E5%B0%8F%E5%A0%86?fromtitle=%E5%B0%8F%E9%A1%B6%E5%A0%86&amp;fromid=4634212" data-nodeid="5255">小顶堆</a>”。小顶堆自身特有的插入和删除逻辑，<strong data-nodeid="5265">决定了无论我们怎么增删小顶堆的元素，其根节点一定是所有元素中值最小的一个节点</strong>。这样的性质，使得小顶堆经常被用于实现<a href="https://baike.baidu.com/item/%E4%BC%98%E5%85%88%E9%98%9F%E5%88%97" data-nodeid="5263">优先队列</a>。</p>
<p data-nodeid="4839">结合小顶堆的特性，我们再来看源码中涉及 timerQueue 和 taskQueue 的操作，这段代码同时也是整个 unstable_scheduleCallback 方法中的核心逻辑：</p>
<pre class="lang-java" data-nodeid="4840"><code data-language="java"><span class="hljs-comment">// 若当前时间小于开始时间，说明该任务可延时执行(未过期）</span>
  <span class="hljs-keyword">if</span> (startTime &gt; currentTime) {
    <span class="hljs-comment">// 将未过期任务推入 "timerQueue"</span>
    newTask.sortIndex = startTime;
    push(timerQueue, newTask);

    <span class="hljs-comment">// 若 taskQueue 中没有可执行的任务，而当前任务又是 timerQueue 中的第一个任务</span>
    <span class="hljs-keyword">if</span> (peek(taskQueue) === <span class="hljs-keyword">null</span> &amp;&amp; newTask === peek(timerQueue)) {
      ......
          <span class="hljs-comment">// 那么就派发一个延时任务，这个延时任务用于将过期的 task 加入 taskQueue 队列</span>
      requestHostTimeout(handleTimeout, startTime - currentTime);
    }
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// else 里处理的是当前时间大于 startTime 的情况，说明这个任务已过期</span>
    newTask.sortIndex = expirationTime;
    <span class="hljs-comment">// 过期的任务会被推入 taskQueue</span>
    push(taskQueue, newTask);
    ......

    <span class="hljs-comment">// 执行 taskQueue 中的任务</span>
    requestHostCallback(flushWork);
  }
</code></pre>
<p data-nodeid="4841">若判断当前任务是待执行任务，那么该任务会在 sortIndex 属性被赋值为 startTime 后，被<strong data-nodeid="5274">推入 timerQueue</strong>。随后，会进入这样的一段判断逻辑：</p>
<pre class="lang-java" data-nodeid="4842"><code data-language="java"><span class="hljs-comment">// 若 taskQueue 中没有可执行的任务，而当前任务又是 timerQueue 中的第一个任务</span>
<span class="hljs-keyword">if</span> (peek(taskQueue) === <span class="hljs-keyword">null</span> &amp;&amp; newTask === peek(timerQueue)) {
  ......
    <span class="hljs-comment">// 那么就派发一个延时任务，这个延时任务用于将过期的 task 加入 taskQueue 队列</span>
  requestHostTimeout(handleTimeout, startTime - currentTime);
}
</code></pre>
<p data-nodeid="4843">要理解这段逻辑，首先需要理解 peek(xxx) 做了什么：peek() 的入参是一个小顶堆，它将取出这个小顶堆的堆顶元素。</p>
<p data-nodeid="4844">taskQueue 里存储的是已过期的任务，peek(taskQueue) 取出的任务若为空，则说明 taskQueue 为空、当前并没有已过期任务。在没有已过期任务的情况下，会进一步判断 timerQueue，也就是未过期任务队列里的情况。</p>
<p data-nodeid="4845">而通过前面的科普，大家已经知道了小顶堆是一个<strong data-nodeid="5290">相对有序</strong>的数据结构。timerQueue 作为一个小顶堆，它的排序依据其实正是 <strong data-nodeid="5291">sortIndex</strong> 属性的大小。这里的 sortIndex 属性取值为 startTime，<strong data-nodeid="5292">意味着小顶堆的堆顶任务一定是整个 timerQueue 堆结构里 startTime 最小的任务，也就是需要最早被执行的未过期任务</strong>。</p>
<p data-nodeid="4846">若当前任务（newTask）就是 timerQueue 中需要最早被执行的未过期任务，那么 unstable_scheduleCallback 会通过调用 requestHostTimeout，为当前任务发起一个延时调用。</p>
<p data-nodeid="4847">注意，这个延时调用（也就是 handleTimeout）<strong data-nodeid="5305">并不会直接调度执行当前任务</strong>——它的作用是在当前任务到期后，将其从 timerQueue 中取出，加入 taskQueue 中，然后触发对 flushWork 的调用。真正的调度执行过程是在 flushWork 中进行的。<strong data-nodeid="5306">flushWork 中将调用 workLoop，workLoop 会逐一执行 taskQueue 中的任务，直到调度过程被暂停（时间片用尽）或任务全部被清空</strong>。</p>
<p data-nodeid="4848">以上便是针对未过期任务的处理。在这个基础上，我们不难理解 else 中，对过期任务的处理逻辑（也就是下面这段代码）：</p>
<pre class="lang-java" data-nodeid="4849"><code data-language="java">{
  <span class="hljs-comment">// else 里处理的是当前时间大于 startTime 的情况，说明这个任务已过期</span>
  newTask.sortIndex = expirationTime;
  <span class="hljs-comment">// 过期的任务会被推入 taskQueue</span>
  push(taskQueue, newTask);
  ......
  <span class="hljs-comment">// 执行 taskQueue 中的任务</span>
  requestHostCallback(flushWork);
}
</code></pre>
<p data-nodeid="4850">与 timerQueue 不同的是，taskQueue 是一个以 expirationTime 为 sortIndex（排序依据）的小顶堆。对于已过期任务，React 在将其推入 taskQueue 后，会通过 requestHostCallback(flushWork) 发起一个针对 flushWork 的即时任务，而 flushWork 会执行 taskQueue 中过期的任务。</p>
<p data-nodeid="4851">从 React 17.0.0 源码来看，当下 React 发起 Task 调度的姿势有两个：<strong data-nodeid="5322">setTimeout</strong>、<strong data-nodeid="5323">MessageChannel</strong>。在宿主环境不支持 MessageChannel 的情况下，会降级到 setTimeout。但不管是 setTimeout 还是 MessageChannel，它们发起的都是<strong data-nodeid="5324">异步任务</strong>。</p>
<p data-nodeid="4852">因此 &nbsp;requestHostCallback 发起的“即时任务”最早也要等到<strong data-nodeid="5330">下一次事件循环</strong>才能够执行。“即时”仅仅意味它相对于“延时任务”来说，不需要等待指定的时间间隔，并不意味着同步调用。</p>
<p data-nodeid="4853">这里为了方便大家理解，我将 unstable_scheduleCallback 方法的工作流总结进一张大图：</p>
<p data-nodeid="4854"><img src="https://s0.lgstatic.com/i/image/M00/73/AE/CgqCHl_GIzeAHilIAAFT8rmskL8314.png" alt="Drawing 32.png" data-nodeid="5336"></p>
<p data-nodeid="4855" class="">这张大图需要结合楼上的文字解析一起消化，如果你是跳读至此，还请回到文章中细嚼慢咽~^_^</p>
<h3 data-nodeid="4856">总结</h3>
<p data-nodeid="4857">这一讲我们首先认识了“双缓存”模式在 Fiber 架构下的实现，接着对更新链路的种种要素进行了拆解，理解了挂载 / 更新等动作的本质。最后，我们结合源码对 Scheduler（调度器）的核心能力，也就是“时间切片”和“优先级调度”两个方面进行了剖析，最终揭开了 Fiber 架构异步渲染的神秘面纱，理解了 Concurrent 模式背后的实现逻辑。</p>
<p data-nodeid="4858" class="">到这里，关于 Fiber 架构的探讨，就要告一段落了。下一讲将讲解“特别的事件系统：React 事件与 DOM 事件有何不同”，到时见~</p>

---

### 精选评论

##### 淦：
> 老师，真大道若简，话剧例子举的太形象，大赞

##### console_man：
> 秀妍老师，你好。请问下对于这个公式：【startTime = currentTime + delay】，delay 是怎么来的呢。有什么依据吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这不是公式，这是源码里的表达式，这样设计是为了使任务的排列更加灵活。delay是unstable_scheduleCallback这个函数中options入参的一个属性，是一个可以被指定的值。

##### **9918：
> 老师，请问一下，什么情况的时候newTask不是timerQueue的堆顶任务？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果 timerQueue 中有开始时间比 newTask 更早的未过期任务，则newTask不会成为timerQueue的堆顶；若 newTask 为已过期任务，则它不会被推入 timerQueue，而是会被推入 taskQueue。

##### **卜：
> 所以setState是和之前的区别不大吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 最明显的区别是setState严格异步了，所有的更新都会走调度这条路。

##### **玲：
> 这里收获了新知识，谢谢老师

##### **用户3046：
> 还是满多疑惑的：1. update的lane是优先级2. task的priorityLevel也是优先级3. update是每个fiberNode的更新4. task是什么？一个task代表一个render阶段阶段吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; update是一个静态的对象，它是对更新的描述。更新需要能够转换成具体的任务被执行，task就是在调度更新后用于描述任务的。

##### 淦：
> 修言老师，有个疑问望解答。您之前说异步渲染渲染可中断，且一旦中断则完全重新渲染。完全指的不包括已经完成的task吧，只是最近的未完成的那个task吧?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的完全指的是生命周期会从头开始执行，而不是整个渲染任务重头开始执行哈。

##### console_man：
> 老师，你好，请问react-dom的哪个版本可以使用ReactDOM.createRoot方法呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 印象中不久前试用了一下 React17.0.0，也是只能够加前缀才能调用的哈。直接 createRoot 可能在不远的未来~~

