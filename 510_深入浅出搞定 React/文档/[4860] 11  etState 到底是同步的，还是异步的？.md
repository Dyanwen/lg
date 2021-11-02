<p data-nodeid="853" class="">setState 对于许多的 React 开发者来说，像是一个“最熟悉的陌生人”：</p>
<ul data-nodeid="854">
<li data-nodeid="855">
<p data-nodeid="856">当你入门 React 的时候，接触的第一波 API 里一定有 setState——数据驱动视图，没它就没法创造变化；</p>
</li>
<li data-nodeid="857">
<p data-nodeid="858">当你项目的数据流乱作一团的时候，层层排查到最后，始作俑者也往往是 setState——工作机制太复杂，文档又不说清楚，只能先“摸着石头过河”。</p>
</li>
</ul>
<p data-nodeid="859">久而久之，setState 的工作机制渐渐与 React 调和算法并驾齐驱，成了 React 核心原理中区分度最高的知识模块之一。本讲我们就紧贴 React 源码和时下最高频的面试题目，帮你从根儿上理解 setState 工作流。</p>
<h3 data-nodeid="860">从一道面试题说起</h3>
<p data-nodeid="861">这是一道变体繁多的面试题，在 BAT 等一线大厂的面试中考察频率非常高。首先题目会给出一个这样的 App 组件，在它的内部会有如下代码所示的几个不同的 setState 操作：</p>
<pre class="lang-js" data-nodeid="862"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">"./styles.css"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">App</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span></span>{
  state = {
    <span class="hljs-attr">count</span>: <span class="hljs-number">0</span>
  }
  increment = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'increment setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">count</span>: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
    });
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'increment setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  }
  triple = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'triple setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">count</span>: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
    });
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">count</span>: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
    });
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">count</span>: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
    });
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'triple setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  }
  reduce = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    setTimeout(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
      <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'reduce setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
      <span class="hljs-keyword">this</span>.setState({
        <span class="hljs-attr">count</span>: <span class="hljs-keyword">this</span>.state.count - <span class="hljs-number">1</span>
      });
      <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'reduce setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
    },<span class="hljs-number">0</span>);
  }
  render(){
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.increment}</span>&gt;</span>点我增加<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.triple}</span>&gt;</span>点我增加三倍<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.reduce}</span>&gt;</span>点我减少<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  }
}
</code></pre>
<p data-nodeid="863">接着我把组件挂载到 DOM 上：</p>
<pre class="lang-js" data-nodeid="864"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> ReactDOM <span class="hljs-keyword">from</span> <span class="hljs-string">"react-dom"</span>;
<span class="hljs-keyword">import</span> App <span class="hljs-keyword">from</span> <span class="hljs-string">"./App"</span>;
<span class="hljs-keyword">const</span> rootElement = <span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">"root"</span>);
ReactDOM.render(
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">React.StrictMode</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">App</span> /&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">React.StrictMode</span>&gt;</span></span>,
  rootElement
);
</code></pre>
<p data-nodeid="865">此时浏览器里渲染出来的是如下图所示的三个按钮：</p>
<p data-nodeid="866"><img src="https://s0.lgstatic.com/i/image/M00/6C/16/Ciqc1F-qYzOAEHeBAAAouh3EFik606.png" alt="Drawing 0.png" data-nodeid="955"></p>
<p data-nodeid="867">此时有个问题，若从左到右依次点击每个按钮，控制台的输出会是什么样的？读到这里，建议你先暂停 1 分钟在脑子里跑一下代码，看看和下图实际运行出来的结果是否有出入。</p>
<p data-nodeid="868"><img src="https://s0.lgstatic.com/i/image/M00/6D/8A/Ciqc1F-uMdqAVUoFAAIqtDlymxs173.png" alt="图片4.png" data-nodeid="959"></p>
<p data-nodeid="869">如果你是一个熟手 React 开发，那么 increment 这个方法的输出结果想必难不倒你——正如许许多多的 React 入门教学所声称的那样，“setState 是一个异步的方法”，这意味着当我们执行完 setState 后，state 本身并不会立刻发生改变。 因此紧跟在 setState 后面输出的 state 值，仍然会维持在它的初始状态（0）。在同步代码执行完毕后的某个“神奇时刻”，state 才会“恰恰好”地增加到 1。</p>
<p data-nodeid="870">但这个“神奇时刻”到底何时发生，所谓的“恰恰好”又如何界定呢？如果你对这个问题搞不太清楚，那么 triple 方法的输出对你来说就会有一定的迷惑性——setState 一次不好使， setState 三次也没用，state 到底是在哪个环节发生了变化呢？</p>
<p data-nodeid="871">带着这样的困惑，你决定先抛开一切去看看 reduce 方法里是什么光景，结果更令人大跌眼镜，reduce 方法里的 setState 竟然是同步更新的！这......到底是我们初学 React 时拿到了错误的基础教程，还是电脑坏了？</p>
<p data-nodeid="872">要想理解眼前发生的这魔幻的一切，我们还得从 setState 的工作机制里去找线索。</p>
<h3 data-nodeid="873">异步的动机和原理——批量更新的艺术</h3>
<p data-nodeid="874">我们首先要认知的一个问题：在 setState 调用之后，都发生了哪些事情？基于截止到现在的专栏知识储备，你可能会更倾向于站在生命周期的角度去思考这个问题，得出一个如下图所示的结论：</p>
<p data-nodeid="875"><img src="https://s0.lgstatic.com/i/image/M00/6D/8A/Ciqc1F-uMeSAYK6FAABN0Vwnq5M814.png" alt="图片3.png" data-nodeid="968"></p>
<p data-nodeid="876">从图上我们可以看出，一个完整的更新流程，涉及了包括 re-render（重渲染） 在内的多个步骤。re-render 本身涉及对 DOM 的操作，它会带来较大的性能开销。假如说“一次 setState 就触发一个完整的更新流程”这个结论成立，那么每一次 setState 的调用都会触发一次 re-render，我们的视图很可能没刷新几次就卡死了。这个过程如我们下面代码中的箭头流程图所示：</p>
<pre class="lang-java" data-nodeid="877"><code data-language="java"><span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    shouldComponentUpdate-&gt;componentWillUpdate-&gt;render-&gt;componentDidUpdate
});
<span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    shouldComponentUpdate-&gt;componentWillUpdate-&gt;render-&gt;componentDidUpdate
});
<span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    shouldComponentUpdate-&gt;componentWillUpdate-&gt;render-&gt;componentDidUpdate
});
</code></pre>
<p data-nodeid="878">事实上，这正是 setState 异步的一个重要的动机——<strong data-nodeid="975">避免频繁的 re-render</strong>。</p>
<p data-nodeid="879">在实际的 React 运行时中，setState 异步的实现方式有点类似于 Vue 的 $nextTick 和浏览器里的 Event-Loop：<strong data-nodeid="981">每来一个 setState，就把它塞进一个队列里“攒起来”。等时机成熟，再把“攒起来”的 state 结果做合并，最后只针对最新的 state 值走一次更新流程。这个过程，叫作“批量更新”</strong>，批量更新的过程正如下面代码中的箭头流程图所示：</p>
<pre class="lang-java" data-nodeid="880"><code data-language="java"><span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    入队，[count+<span class="hljs-number">1</span>的任务]
});
<span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    入队，[count+<span class="hljs-number">1</span>的任务，count+<span class="hljs-number">1</span>的任务]
});
<span class="hljs-keyword">this</span>.setState({
  count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>    ===&gt;    入队, [count+<span class="hljs-number">1</span>的任务，count+<span class="hljs-number">1</span>的任务, count+<span class="hljs-number">1</span>的任务]
});
                                          ↓
                                         合并 state，[count+<span class="hljs-number">1</span>的任务]
                                          ↓
                                         执行 count+<span class="hljs-number">1</span>的任务
</code></pre>
<p data-nodeid="881">值得注意的是，只要我们的同步代码还在执行，“攒起来”这个动作就不会停止。（注：这里之所以多次 +1 最终只有一次生效，是因为在同一个方法中多次 setState 的合并动作不是单纯地将更新累加。比如这里对于相同属性的设置，React 只会为其保留最后一次的更新）。因此就算我们在 React 中写了这样一个 100 次的 setState 循环：</p>
<pre class="lang-java" data-nodeid="882"><code data-language="java">test = () =&gt; {
  console.log(<span class="hljs-string">'循环100次 setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  <span class="hljs-keyword">for</span>(let i=<span class="hljs-number">0</span>;i&lt;<span class="hljs-number">100</span>;i++) {
    <span class="hljs-keyword">this</span>.setState({
      count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
    })
  }
  console.log(<span class="hljs-string">'循环100次 setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
}
</code></pre>
<p data-nodeid="883">也只是会增加 state 任务入队的次数，并不会带来频繁的 re-render。当 100 次调用结束后，仅仅是 state 的任务队列内容发生了变化， state 本身并不会立刻改变：</p>
<p data-nodeid="884"><img src="https://s0.lgstatic.com/i/image/M00/6D/8B/Ciqc1F-uMfKALHLXAAEBeCrt5lE676.png" alt="图片5.png" data-nodeid="986"></p>
<h3 data-nodeid="885">“同步现象”背后的故事：从源码角度看 setState 工作流</h3>
<p data-nodeid="886">读到这里，相信你对异步这回事多少有些眉目了。接下来我们就要重点理解刚刚代码里最诡异的一部分——setState 的同步现象：</p>
<pre class="lang-java" data-nodeid="887"><code data-language="java">reduce = () =&gt; {
  setTimeout(() =&gt; {
    console.log(<span class="hljs-string">'reduce setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
    <span class="hljs-keyword">this</span>.setState({
      count: <span class="hljs-keyword">this</span>.state.count - <span class="hljs-number">1</span>
    });
    console.log(<span class="hljs-string">'reduce setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  },<span class="hljs-number">0</span>);
}
</code></pre>
<p data-nodeid="888">从题目上看，setState 似乎是在 setTimeout 函数的“保护”之下，才有了同步这一“特异功能”。事实也的确如此，假如我们把 setTimeout 摘掉，setState 前后的 console 表现将会与 increment 方法中无异：</p>
<pre class="lang-java" data-nodeid="889"><code data-language="java">reduce = () =&gt; {
  <span class="hljs-comment">// setTimeout(() =&gt; {</span>
  console.log(<span class="hljs-string">'reduce setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  <span class="hljs-keyword">this</span>.setState({
    count: <span class="hljs-keyword">this</span>.state.count - <span class="hljs-number">1</span>
  });
  console.log(<span class="hljs-string">'reduce setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  <span class="hljs-comment">// },0);</span>
}
</code></pre>
<p data-nodeid="890">点击后的输出结果如下图所示：</p>
<p data-nodeid="891"><img src="https://s0.lgstatic.com/i/image/M00/6D/96/CgqCHl-uMguADJiMAAEld6KAKBI013.png" alt="图片6.png" data-nodeid="993"></p>
<p data-nodeid="892">现在问题就变得清晰多了：为什么 setTimeout 可以将 setState 的执行顺序从异步变为同步？</p>
<p data-nodeid="893">这里我先给出一个结论：<strong data-nodeid="1000">并不是 setTimeout 改变了 setState，而是 setTimeout 帮助 setState “逃脱”了 React 对它的管控。只要是在 React 管控下的 setState，一定是异步的</strong>。</p>
<p data-nodeid="894">接下来我们就从 React 源码里，去寻求佐证这个结论的线索。</p>
<blockquote data-nodeid="895">
<p data-nodeid="896">tips：时下虽然市场里的 React 16、React 17 十分火热，但就 setState 这块知识来说，React 15 仍然是最佳的学习素材。因此下文所有涉及源码的分析，都会围绕 React 15 展开。关于 React 16 之后 Fiber 机制给 setState 带来的改变，我们会有专门一讲来分析，不在本讲的讨论范围内。</p>
</blockquote>
<h4 data-nodeid="897">解读 setState 工作流</h4>
<p data-nodeid="898">我们阅读任何框架的源码，都应该带着问题、带着目的去读。React 中对于功能的拆分是比较细致的，setState 这部分涉及了多个方法。为了方便你理解，我这里先把主流程提取为一张大图：</p>
<p data-nodeid="899"><img src="https://s0.lgstatic.com/i/image2/M01/04/81/Cip5yF_yswuAWzDfAAEc1lISh-Q211.png" alt="3.png" data-nodeid="1007"></p>
<p data-nodeid="900">接下来我们就沿着这个流程，逐个在源码中对号入座。首先是 setState 入口函数：</p>
<pre class="lang-java" data-nodeid="901"><code data-language="java">ReactComponent.prototype.setState = function (partialState, callback) {
  <span class="hljs-keyword">this</span>.updater.enqueueSetState(<span class="hljs-keyword">this</span>, partialState);
  <span class="hljs-keyword">if</span> (callback) {
    <span class="hljs-keyword">this</span>.updater.enqueueCallback(<span class="hljs-keyword">this</span>, callback, <span class="hljs-string">'setState'</span>);
  }
};
</code></pre>
<p data-nodeid="902">入口函数在这里就是充当一个分发器的角色，根据入参的不同，将其分发到不同的功能函数中去。这里我们以对象形式的入参为例，可以看到它直接调用了 this.updater.enqueueSetState 这个方法：</p>
<pre class="lang-java" data-nodeid="903"><code data-language="java">enqueueSetState: function (publicInstance, partialState) {
  <span class="hljs-comment">// 根据 this 拿到对应的组件实例</span>
  <span class="hljs-keyword">var</span> internalInstance = getInternalInstanceReadyForUpdate(publicInstance, <span class="hljs-string">'setState'</span>);
  <span class="hljs-comment">// 这个 queue 对应的就是一个组件实例的 state 数组</span>
  <span class="hljs-keyword">var</span> queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = []);
  queue.push(partialState);
  <span class="hljs-comment">//  enqueueUpdate 用来处理当前的组件实例</span>
  enqueueUpdate(internalInstance);
}
</code></pre>
<p data-nodeid="904">这里我总结一下，enqueueSetState 做了两件事：</p>
<ul data-nodeid="905">
<li data-nodeid="906">
<p data-nodeid="907">将新的 state 放进组件的状态队列里；</p>
</li>
<li data-nodeid="908">
<p data-nodeid="909">用 enqueueUpdate 来处理将要更新的实例对象。</p>
</li>
</ul>
<p data-nodeid="910">继续往下走，看看 enqueueUpdate 做了什么：</p>
<pre class="lang-java" data-nodeid="911"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">enqueueUpdate</span><span class="hljs-params">(component)</span> </span>{
  ensureInjected();
  <span class="hljs-comment">// 注意这一句是问题的关键，isBatchingUpdates标识着当前是否处于批量创建/更新组件的阶段</span>
  <span class="hljs-keyword">if</span> (!batchingStrategy.isBatchingUpdates) {
    <span class="hljs-comment">// 若当前没有处于批量创建/更新组件的阶段，则立即更新组件</span>
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    <span class="hljs-keyword">return</span>;
  }
  <span class="hljs-comment">// 否则，先把组件塞入 dirtyComponents 队列里，让它“再等等”</span>
  dirtyComponents.push(component);
  <span class="hljs-keyword">if</span> (component._updateBatchNumber == <span class="hljs-keyword">null</span>) {
    component._updateBatchNumber = updateBatchNumber + <span class="hljs-number">1</span>;
  }
}
</code></pre>
<p data-nodeid="912">这个 enqueueUpdate 非常有嚼头，它引出了一个关键的对象——batchingStrategy，该对象所具备的isBatchingUpdates属性直接决定了当下是要走更新流程，还是应该排队等待；其中的batchedUpdates 方法更是能够直接发起更新流程。由此我们可以大胆推测，<strong data-nodeid="1021">batchingStrategy 或许正是 React 内部专门用于管控批量更新的对象</strong>。<br>
接下来，我们就一起来研究研究这个 batchingStrategy。</p>
<pre class="lang-java" data-nodeid="913"><code data-language="java"><span class="hljs-comment">/**
 * batchingStrategy源码
**/</span>
 
<span class="hljs-keyword">var</span> ReactDefaultBatchingStrategy = {
  <span class="hljs-comment">// 全局唯一的锁标识</span>
  isBatchingUpdates: <span class="hljs-keyword">false</span>,
 
  <span class="hljs-comment">// 发起更新动作的方法</span>
  batchedUpdates: function(callback, a, b, c, d, e) {
    <span class="hljs-comment">// 缓存锁变量</span>
    <span class="hljs-keyword">var</span> alreadyBatchingStrategy = ReactDefaultBatchingStrategy. isBatchingUpdates
    <span class="hljs-comment">// 把锁“锁上”</span>
    ReactDefaultBatchingStrategy. isBatchingUpdates = <span class="hljs-function"><span class="hljs-keyword">true</span>

    <span class="hljs-title">if</span> <span class="hljs-params">(alreadyBatchingStrategy)</span> </span>{
      callback(a, b, c, d, e)
    } <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 启动事务，将 callback 放进事务里执行</span>
      transaction.perform(callback, <span class="hljs-keyword">null</span>, a, b, c, d, e)
    }
  }
}
</code></pre>
<p data-nodeid="914">batchingStrategy 对象并不复杂，你可以理解为它是一个“锁管理器”。</p>
<p data-nodeid="915">这里的“锁”，是指 React 全局唯一的 isBatchingUpdates 变量，isBatchingUpdates 的初始值是 false，意味着“当前并未进行任何批量更新操作”。每当 React 调用 batchedUpdate 去执行更新动作时，会先把这个锁给“锁上”（置为 true），表明“现在正处于批量更新过程中”。当锁被“锁上”的时候，任何需要更新的组件都只能暂时进入 dirtyComponents 里排队等候下一次的批量更新，而不能随意“插队”。此处体现的“任务锁”的思想，是 React 面对大量状态仍然能够实现有序分批处理的基石。</p>
<p data-nodeid="916">理解了批量更新整体的管理机制，还需要注意 batchedUpdates 中，有一个引人注目的调用：</p>
<pre class="lang-java" data-nodeid="917"><code data-language="java">transaction.perform(callback, <span class="hljs-keyword">null</span>, a, b, c, d, e)
</code></pre>
<p data-nodeid="918">这行代码为我们引出了一个更为硬核的概念——React 中的 Transaction（事务）机制。</p>
<h4 data-nodeid="919">理解 React 中的 Transaction（事务） 机制</h4>
<p data-nodeid="920">Transaction 在 React 源码中的分布可以说非常广泛。如果你在 Debug React 项目的过程中，发现函数调用栈中出现了 initialize、perform、close、closeAll 或者 notifyAll 这样的方法名，那么很可能你当前就处于一个 Trasaction 中。</p>
<p data-nodeid="921">Transaction 在 React 源码中表现为一个核心类，React 官方曾经这样描述它：<strong data-nodeid="1033">Transaction 是创建一个黑盒</strong>，该黑盒能够封装任何的方法。因此，那些需要在函数运行前、后运行的方法可以通过此方法封装（即使函数运行中有异常抛出，这些固定的方法仍可运行），实例化 Transaction 时只需提供相关的方法即可。</p>
<p data-nodeid="922">这段话初读有点拗口，这里我推荐你结合 React 源码中的一段针对 Transaction 的注释来理解它：</p>
<pre class="lang-java" data-nodeid="923"><code data-language="java">* &lt;pre&gt;
 *                       wrappers (injected at creation time)
 *                                      +        +
 *                                      |        |
 *                    +-----------------|--------|--------------+
 *                    |                 v        |              |
 *                    |      +---------------+   |              |
 *                    |   +--|    wrapper1   |---|----+         |
 *                    |   |  +---------------+   v    |         |
 *                    |   |          +-------------+  |         |
 *                    |   |     +----|   wrapper2  |--------+   |
 *                    |   |     |    +-------------+  |     |   |
 *                    |   |     |                     |     |   |
 *                    |   v     v                     v     v   | wrapper
 *                    | +---+ +---+   +---------+   +---+ +---+ | invariants
 * perform(anyMethod) | |   | |   |   |         |   |   | |   | | maintained
 * +-----------------&gt;|-|---|-|---|--&gt;|anyMethod|---|---|-|---|-|--------&gt;
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | |   | |   |   |         |   |   | |   | |
 *                    | +---+ +---+   +---------+   +---+ +---+ |
 *                    |  initialize                    close    |
 *                    +-----------------------------------------+
 * &lt;/pre&gt;
</code></pre>
<p data-nodeid="924">说白了，Transaction 就像是一个“壳子”，它首先会将目标函数用 wrapper（一组 initialize 及 close 方法称为一个 wrapper） 封装起来，同时需要使用 Transaction 类暴露的 perform 方法去执行它。如上面的注释所示，在 anyMethod 执行之前，perform 会先执行所有 wrapper 的 initialize 方法，执行完后，再执行所有 wrapper 的 close 方法。这就是 React 中的事务机制。</p>
<h4 data-nodeid="925">“同步现象”的本质</h4>
<p data-nodeid="926">下面结合对事务机制的理解，我们继续来看在 ReactDefaultBatchingStrategy 这个对象。ReactDefaultBatchingStrategy 其实就是一个批量更新策略事务，它的 wrapper 有两个：FLUSH_BATCHED_UPDATES 和 RESET_BATCHED_UPDATES。</p>
<pre class="lang-java" data-nodeid="927"><code data-language="java"><span class="hljs-keyword">var</span> RESET_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: function () {
    ReactDefaultBatchingStrategy.isBatchingUpdates = <span class="hljs-keyword">false</span>;
  }
};
<span class="hljs-keyword">var</span> FLUSH_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: ReactUpdates.flushBatchedUpdates.bind(ReactUpdates)
};
<span class="hljs-keyword">var</span> TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];
</code></pre>
<p data-nodeid="928">我们把这两个 wrapper 套进 Transaction 的执行机制里，不难得出一个这样的流程：</p>
<p data-nodeid="929"><img src="https://s0.lgstatic.com/i/image/M00/6E/2E/Ciqc1F-x-tyAbioYAACikzik89A130.png" alt="图片5.png" data-nodeid="1049"></p>
<p data-nodeid="6630" class="te-preview-highlight">到这里，相信你对 isBatchingUpdates 管控下的批量更新机制已经了然于胸。但是 setState 为何会表现同步这个问题，似乎还是没有从当前展示出来的源码里得到根本上的回答。这是因为 batchedUpdates 这个方法，不仅仅会在 setState 之后才被调用。若我们在 React 源码中全局搜索 batchedUpdates，会发现调用它的地方很多，但与更新流有关的只有这两个地方：</p>














<pre class="lang-java" data-nodeid="931"><code data-language="java"><span class="hljs-comment">// ReactMount.js</span>
_renderNewRootComponent: function( nextElement, container, shouldReuseMarkup, context ) {
  <span class="hljs-comment">// 实例化组件</span>
  <span class="hljs-keyword">var</span> componentInstance = instantiateReactComponent(nextElement);
  <span class="hljs-comment">// 初始渲染直接调用 batchedUpdates 进行同步渲染</span>
  ReactUpdates.batchedUpdates(
    batchedMountComponentIntoNode,
    componentInstance,
    container,
    shouldReuseMarkup,
    context
  );
  ...
}
</code></pre>
<p data-nodeid="932">这段代码是在首次渲染组件时会执行的一个方法，我们看到它内部调用了一次 batchedUpdates，这是因为在组件的渲染过程中，会按照顺序调用各个生命周期函数。开发者很有可能在声明周期函数中调用 setState。因此，我们需要通过开启 batch 来确保所有的更新都能够进入 dirtyComponents 里去，进而确保初始渲染流程中所有的 setState 都是生效的。</p>
<p data-nodeid="933">下面代码是 React 事件系统的一部分。当我们在组件上绑定了事件之后，事件中也有可能会触发 setState。为了确保每一次 setState 都有效，React 同样会在此处手动开启批量更新。</p>
<pre class="lang-java" data-nodeid="934"><code data-language="java"><span class="hljs-comment">// ReactEventListener.js</span>
dispatchEvent: function (topLevelType, nativeEvent) {
  ...
  <span class="hljs-keyword">try</span> {
    <span class="hljs-comment">// 处理事件</span>
    ReactUpdates.batchedUpdates(handleTopLevelImpl, bookKeeping);
  } <span class="hljs-keyword">finally</span> {
    TopLevelCallbackBookKeeping.release(bookKeeping);
  }
}
</code></pre>
<p data-nodeid="935">话说到这里，一切都变得明朗了起来：isBatchingUpdates 这个变量，在 React 的生命周期函数以及合成事件执行前，已经被 React 悄悄修改为了 true，这时我们所做的 setState 操作自然不会立即生效。当函数执行完毕后，事务的 close 方法会再把 isBatchingUpdates 改为 false。</p>
<p data-nodeid="936">以开头示例中的 increment 方法为例，整个过程像是这样：</p>
<pre class="lang-java" data-nodeid="937"><code data-language="java">increment = () =&gt; {
  <span class="hljs-comment">// 进来先锁上</span>
  isBatchingUpdates = <span class="hljs-keyword">true</span>
  console.log(<span class="hljs-string">'increment setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  <span class="hljs-keyword">this</span>.setState({
    count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
  });
  console.log(<span class="hljs-string">'increment setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  <span class="hljs-comment">// 执行完函数再放开</span>
  isBatchingUpdates = <span class="hljs-keyword">false</span>
}
</code></pre>
<p data-nodeid="938">很明显，在 isBatchingUpdates 的约束下，setState 只能是异步的。而当 setTimeout 从中作祟时，事情就会发生一点点变化：</p>
<pre class="lang-java" data-nodeid="939"><code data-language="java">reduce = () =&gt; {
  <span class="hljs-comment">// 进来先锁上</span>
  isBatchingUpdates = <span class="hljs-function"><span class="hljs-keyword">true</span>
  <span class="hljs-title">setTimeout</span><span class="hljs-params">(()</span> </span>=&gt; {
    console.log(<span class="hljs-string">'reduce setState前的count'</span>, <span class="hljs-keyword">this</span>.state.count)
    <span class="hljs-keyword">this</span>.setState({
      count: <span class="hljs-keyword">this</span>.state.count - <span class="hljs-number">1</span>
    });
    console.log(<span class="hljs-string">'reduce setState后的count'</span>, <span class="hljs-keyword">this</span>.state.count)
  },<span class="hljs-number">0</span>);
  <span class="hljs-comment">// 执行完函数再放开</span>
  isBatchingUpdates = <span class="hljs-keyword">false</span>
}
</code></pre>
<p data-nodeid="940">会发现，咱们开头锁上的那个 isBatchingUpdates，对 setTimeout 内部的执行逻辑完全没有约束力。因为 isBatchingUpdates 是在同步代码中变化的，而 setTimeout 的逻辑是异步执行的。当 this.setState 调用真正发生的时候，isBatchingUpdates 早已经被重置为了 false，这就使得当前场景下的 setState 具备了立刻发起同步更新的能力。所以咱们前面说的没错——<strong data-nodeid="1061">setState 并不是具备同步这种特性，只是在特定的情境下，它会从 React 的异步管控中“逃脱”掉</strong>。</p>
<h3 data-nodeid="941">总结</h3>
<p data-nodeid="942">道理很简单，原理却很复杂。最后，我们再一次面对面回答一下标题提出的问题，对整个 setState 工作流做一个总结。</p>
<p data-nodeid="943">setState 并不是单纯同步/异步的，它的表现会因调用场景的不同而不同：在 React 钩子函数及合成事件中，它表现为异步；而在 setTimeout、setInterval 等函数中，包括在 DOM 原生事件中，它都表现为同步。这种差异，本质上是由 React 事务机制和批量更新机制的工作方式来决定的。</p>
<p data-nodeid="944" class="">行文至此，相信你已经对 setState 有了知根知底的理解。我们整篇文章的讨论，目前都建立在 React 15 的基础上。React 16 以来，整个 React 核心算法被重写，setState 也不可避免地被“Fiber化”。那么到底什么是“Fiber”，它到底怎样改变着包括 setState 在内的 React 的各个核心技术模块，这就是我们下面两讲要重点讨论的问题了。</p>

---

### 精选评论

##### **宇：
> 前面的，你能给函数组件设置state，说明你的版本至少hooks都出来了，早就是fiber架构了，对state的修改全部是异步的，在16ms的刷新周期里有剩余时间才会被执行，setTimeout是试不出来的。而且函数组件里面的useState只会把某次执行时的state赋值给某个变量，是不变的，你在当前上下文只能获取当前状态切片的state，修改后的是在下一次执行上下文里获取的，所以react文档里说依赖了哪些state，就一定要在[]里写上，不然实际开发中可能会遇到“缓存”bug。

##### symboll：
> 终于上硬菜了哈。

##### **8542：
> setState 并非真异步，只是看上去像异步。在源码中，通过 isBatchingUpdates 来判断setState 是先存进 state 队列还是直接更新，如果值为 true 则执行异步操作，为 false 则直接更新。那么什么情况下 isBatchingUpdates 会为 true 呢？在 React 可以控制的地方，就为 true，比如在 React 生命周期事件和合成事件中，都会走合并操作，延迟更新的策略。但在 React 无法控制的地方，比如原生事件，具体就是在 addEventListener 、setTimeout、setInterval 等事件中，就只能同步更新。一般认为，做异步设计是为了性能优化、减少渲染次数，React 团队还补充了两点。保持内部一致性。如果将 state 改为同步更新，那尽管 state 的更新是同步的，但是 props不是。启用并发更新，完成异步渲染。

##### *驰：
> 太精彩了吧，学习也有想催更的冲动

##### *权：
> 大大源码关键处解析的太到位，个人觉得机制还是比较复杂，要看透源码任重道远

##### **愿：
> 这是开工的一个福利了哈

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 祝小伙伴开工大吉！

##### **1508：
> 我这里有一个demo：https://codesandbox.io/s/react-dom-batched-updates-forked-05uqd?from-embed 有兴趣可以玩一玩。涵盖了hooks的场景。

##### *杨：
> 这个太牛逼了，终于看到清晰的解释了

##### **峰：
> 跪拜，太通透了

##### **潮：
> 学习了，原先以为是react重写了settimeout等方法使得更新方式为同步，那请问老师，按文中的讲解来理解，似乎只需要在一个异步函数里去执行setstate就会是同步更新？而不仅是定时任务或dom原生事件，比如在promise的then方法中去执行setstate？但是这就涉及到微任务和宏任务对setstate的影响？求老师解惑

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 与其想这么多，不如写个demo试试看吧。你说的这些问题都可以通过本地跑个demo找到答案。

##### *聪：
> 棒

##### **森：
> 赞

##### **扬：
> 经典

##### *影：
> 真的很厉害 厉害厉害

##### **琳：
> 膜拜😄

##### *…：
> “每来一个 setState，就把它塞进一个队列里“攒起来”。等时机成熟，再把“攒起来”的 state 结果做合并，最后只针对最新的 state 值走一次更新流程。这个过程，叫作“批量更新” "---时机成熟是什么时候呢？是怎么判断setState攒起来这个动作已经执行完了，可以进行render了呢？是通过eventloop吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 看一下后半截的源码解析。

##### **光：
> 厉害

##### **航：
> 讲得好啊😀

##### *聪：
> 这部分写的确实不错，期待fiber架构下的解读。

##### **蓉：
> 针对我之前提问的为什么在函数组件的sattimeout里修改state，表现为异步的现象，我已经有答案了，在你之前讲过的课里找到了答案：函数组件会捕获内部的状态。

##### **蓉：
> 我读完本文后明白了在类组件中，在react能监控到的范围如生命周期，react事件回调中，执行setState基本都是异步执行，但如果react监控不到代码遇到如原生事件、settimeout等setState就是同步更改。带着这样的规律我去函数组件中试了下，发现在函数组件中，setTimeout里调用更改对应state时，始终是异步。我很好奇，函数组件中state的更改流程function ProfilePage(props) {  var [a, seta] = useState({ tt: 1 });  const handleClick = () = {    setTimeout(() = {      seta({ tt: 2 });      console.log(a, 98989);    }, 1000);  };  return button onClick={handleClick}Follow----{JSON.stringify(a)}/button;}

