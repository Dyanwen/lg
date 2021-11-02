<p data-nodeid="8633" class="">React 团队面向开发者给出了两条 React-Hooks 的使用原则，原则的内容如下：</p>
<ol data-nodeid="8634">
<li data-nodeid="8635">
<p data-nodeid="8636">只在 React 函数中调用 Hook；</p>
</li>
<li data-nodeid="8637">
<p data-nodeid="8638">不要在循环、条件或嵌套函数中调用 Hook。</p>
</li>
</ol>
<p data-nodeid="8639">原则 1 无须多言，React-Hooks 本身就是 React 组件的“钩子”，在普通函数里引入意义不大。我相信更多的人在原则 2 上栽过跟头，或者说至今仍然对它半信半疑。其实，原则 2 中强调的所有“<strong data-nodeid="8722">不要</strong>”，都是在指向同一个目的，那就是<strong data-nodeid="8723">要确保 Hooks 在每次渲染时都保持同样的执行顺序</strong>。</p>
<p data-nodeid="8640">为什么顺序如此重要？这就要从 Hooks 的实现机制说起了。这里我就以 useState 为例，带你从现象入手，深度探索一番 React-Hooks 的工作原理。</p>
<p data-nodeid="8641">注：本讲 Demo 基于 React 16.8.x 版本进行演示。</p>
<h3 data-nodeid="8642">从现象看问题：若不保证 Hooks 执行顺序，会带来什么麻烦？</h3>
<p data-nodeid="8643">先来看一个小 Demo：</p>
<pre class="lang-javascript" data-nodeid="8644"><code data-language="javascript"><span class="hljs-keyword">import</span> React, { useState } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;

<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">PersonalInfoComponent</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-comment">// 集中定义变量</span>
  <span class="hljs-keyword">let</span> name, age, career, setName, setCareer;

  <span class="hljs-comment">// 获取姓名状态</span>
  [name, setName] = useState(<span class="hljs-string">"修言"</span>);

  <span class="hljs-comment">// 获取年龄状态</span>
  [age] = useState(<span class="hljs-string">"99"</span>);

  <span class="hljs-comment">// 获取职业状态</span>
  [career, setCareer] = useState(<span class="hljs-string">"我是一个前端，爱吃小熊饼干"</span>);

  <span class="hljs-comment">// 输出职业信息</span>
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"career"</span>, career);

  <span class="hljs-comment">// 编写 UI 逻辑</span>
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"personalInfo"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>姓名：{name}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>年龄：{age}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>职业：{career}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span>
        <span class="hljs-attr">onClick</span>=<span class="hljs-string">{()</span> =&gt;</span> {
          setName("秀妍");
        }}
      &gt;
        修改姓名
      <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}

<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> PersonalInfoComponent;
</code></pre>
<p data-nodeid="8645">这个 PersonalInfoComponent 组件渲染出来的界面长这样：</p>
<p data-nodeid="8646"><img src="https://s0.lgstatic.com/i/image/M00/89/5F/Ciqc1F_YT0uAT1kZAACw9EfbQe8557.png" alt="1.png" data-nodeid="8731"></p>
<p data-nodeid="10684" class="te-preview-highlight">PersonalInfoComponent 用于对个人信息进行展示，这里展示的内容包括姓名、年龄、职业。出于测试效果需要，PersonalInfoComponent 还允许你点击“修改姓名”按钮修改姓名信息。点击一次后，“修言”会被修改为“秀妍”，如下图所示：</p>





<p data-nodeid="8648"><img src="https://s0.lgstatic.com/i/image/M00/89/6A/CgqCHl_YT1qAUSuVAAC-xZcsk54138.png" alt="2.png" data-nodeid="8735"></p>
<p data-nodeid="8649">到目前为止，组件的行为都是符合我们的预期的，一切看上去都是那么的和谐。但倘若我对代码做一丝小小的改变，把一部分的 useState 操作放进 if 语句里，事情就会变得大不一样。改动后的代码如下：</p>
<pre class="lang-js" data-nodeid="8650"><code data-language="js"><span class="hljs-keyword">import</span> React, { useState } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-comment">// isMounted 用于记录是否已挂载（是否是首次渲染）</span>
<span class="hljs-keyword">let</span> isMounted = <span class="hljs-literal">false</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">PersonalInfoComponent</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-comment">// 定义变量的逻辑不变</span>
  <span class="hljs-keyword">let</span> name, age, career, setName, setCareer;

  <span class="hljs-comment">// 这里追加对 isMounted 的输出，这是一个 debug 性质的操作</span>
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"isMounted is"</span>, isMounted);
  <span class="hljs-comment">// 这里追加 if 逻辑：只有在首次渲染（组件还未挂载）时，才获取 name、age 两个状态</span>
  <span class="hljs-keyword">if</span> (!isMounted) {
    <span class="hljs-comment">// eslint-disable-next-line</span>
    [name, setName] = useState(<span class="hljs-string">"修言"</span>);
    <span class="hljs-comment">// eslint-disable-next-line</span>
    [age] = useState(<span class="hljs-string">"99"</span>);

    <span class="hljs-comment">// if 内部的逻辑执行一次后，就将 isMounted 置为 true（说明已挂载，后续都不再是首次渲染了）</span>
    isMounted = <span class="hljs-literal">true</span>;
  }

  <span class="hljs-comment">// 对职业信息的获取逻辑不变</span>
  [career, setCareer] = useState(<span class="hljs-string">"我是一个前端，爱吃小熊饼干"</span>);
  <span class="hljs-comment">// 这里追加对 career 的输出，这也是一个 debug 性质的操作</span>
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"career"</span>, career);
  <span class="hljs-comment">// UI 逻辑的改动在于，name和age成了可选的展示项，若值为空，则不展示</span>
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"personalInfo"</span>&gt;</span>
      {name ? <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>姓名：{name}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span> : null}
      {age ? <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>年龄：{age}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span> : null}
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>职业：{career}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span>
        <span class="hljs-attr">onClick</span>=<span class="hljs-string">{()</span> =&gt;</span> {
          setName("秀妍");
        }}
      &gt;
        修改姓名
      <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> PersonalInfoComponent;
</code></pre>
<p data-nodeid="8651">修改后的组件在初始渲染的时候，界面与上个版本无异：</p>
<p data-nodeid="8652"><img src="https://s0.lgstatic.com/i/image/M00/67/64/CgqCHl-hJDaAC6-qAACIdJOIg3E041.png" alt="Drawing 5.png" data-nodeid="8740"></p>
<p data-nodeid="8653">注意，你在自己电脑上模仿这段代码的时候，千万不要漏掉 if 语句里面<code data-backticks="1" data-nodeid="8742">// eslint-disable-next-line</code>这个注释——因为目前大部分的 React 项目都在内部预置了对 React-Hooks-Rule（React-Hooks 使用规则）的强校验，而示例代码中把 Hooks 放进 if 语句的操作作为一种不合规操作，会被直接识别为 Error 级别的错误，进而导致程序报错。这里我们只有将相关代码的 eslint 校验给禁用掉，才能够避免校验性质的报错，从而更直观地看到错误的效果到底是什么样的，进而理解错误的原因。</p>
<p data-nodeid="8654">修改后的组件在初始挂载的时候，实际执行的逻辑内容和上个版本是没有区别的，都涉及对 name、age、career 三个状态的获取和渲染。理论上来说，<strong data-nodeid="8749">变化应该发生在我单击“修改姓名”之后触发的二次渲染里</strong>：二次渲染时，isMounted 已经被置为 true，if 内部的逻辑会被直接跳过。此时按照代码注释中给出的设计意图，这里我希望在二次渲染时，只获取并展示 career 这一个状态。那么事情是否会如我所愿呢？我们一起来看看单击“修改姓名”按钮后会发生什么：</p>
<p data-nodeid="8655"><img src="https://s0.lgstatic.com/i/image/M00/67/64/CgqCHl-hJEOAMfdIAAJ8aDhIGdA549.png" alt="Drawing 7.png" data-nodeid="8752"></p>
<p data-nodeid="8656">组件不仅没有像预期中一样发生界面变化，甚至直接报错了。报错信息提醒我们，这是因为“<strong data-nodeid="8758">组件渲染的 Hooks 比期望中更少</strong>”。</p>
<p data-nodeid="8657">确实，按照现有的逻辑，初始渲染调用了三次 useState，而二次渲染时只会调用一次。但仅仅因为这个，就要报错吗？</p>
<p data-nodeid="8658">按道理来说，二次渲染的时候，只要我获取到的 career 值没有问题，那么渲染就应该是没有问题的（因为二次渲染实际只会渲染 career 这一个状态），React 就没有理由阻止我的渲染动作。啊这……难道是 career 出问题了吗？还好我们预先留了一手 Debug 逻辑，每次渲染的时候都会尝试去输出一次 isMounted 和 career 这两个变量的值。现在我们就赶紧来看看，这两个变量到底是什么情况。</p>
<p data-nodeid="8659">首先我将界面重置回初次挂载的状态，观察控制台的输出，如下图所示：</p>
<p data-nodeid="8660"><img src="https://s0.lgstatic.com/i/image/M00/67/64/CgqCHl-hJHSAL8SuAAHP-0rTPKY784.png" alt="Drawing 9.png" data-nodeid="8764"></p>
<p data-nodeid="8661">这里我把关键的 isMounted 和 career 两个变量用红色框框圈了出来：isMounted 值为 false，说明是初次渲染；career 值为“我是一个前端，爱吃小熊饼干”，这也是没有问题的。</p>
<p data-nodeid="8662">接下来单击“修改姓名”按钮后，我们再来看一眼两个变量的内容，如下图所示：</p>
<p data-nodeid="8663"><img src="https://s0.lgstatic.com/i/image/M00/67/64/CgqCHl-hJRiAP2doAAKt-ZhwxQ0744.png" alt="图片11.png" data-nodeid="8769"></p>
<p data-nodeid="8664">二次渲染时，isMounted 为 true，这个没毛病。但是 career 竟然被修改为了“秀妍”，这也太诡异了？代码里面可不是这么写的。赶紧回头确认一下按钮单击事件的回调内容，代码如下所示：</p>
<pre class="lang-js" data-nodeid="8665"><code data-language="js"> &lt;button
   onClick={() =&gt; {
    setName(<span class="hljs-string">"秀妍"</span>);
  }}
   &gt;
  修改姓名
&lt;/button&gt;
</code></pre>
<p data-nodeid="8666">确实，代码是没错的，我们调用的是 setName，那么它修改的状态也应该是 name，而不是 career。</p>
<p data-nodeid="8667">那为什么最后发生变化的竟然是 career 呢？年轻人，不如我们一起来看一看 Hooks 的实现机制吧！</p>
<h3 data-nodeid="8668">从源码调用流程看原理：Hooks 的正常运作，在底层依赖于顺序链表</h3>
<p data-nodeid="8669">这里强调“源码流程”而非“源码”，主要有两方面的考虑：</p>
<ol data-nodeid="8670">
<li data-nodeid="8671">
<p data-nodeid="8672">React-Hooks 在源码层面和 Fiber 关联十分密切，我们目前仍然处于基础夯实阶段，对 Fiber 机制相关的底层实现暂时没有讨论，盲目啃源码在这个阶段来说没有意义；</p>
</li>
<li data-nodeid="8673">
<p data-nodeid="8674">原理 !== 源码，阅读源码只是掌握原理的一种手段，在某些场景下，阅读源码确实能够迅速帮我们定位到问题的本质（比如 React.createElement 的源码就可以快速帮我们理解 JSX 转换出来的到底是什么东西）；而 React-Hooks 的源码链路相对来说比较长，涉及的关键函数 renderWithHooks 中“脏逻辑”也比较多，整体来说，学习成本比较高，学习效果也难以保证。</p>
</li>
</ol>
<p data-nodeid="8675">综上所述，这里我不会精细地贴出每一行具体的源码，而是针对关键方法做重点分析。同时我也<strong data-nodeid="8788">不建议你在对 Fiber 底层实现没有认知的前提下去和 Hooks 源码死磕</strong>。对于搞清楚“Hooks 的执行顺序为什么必须一样”这个问题来说，重要的并不是去细抠每一行代码到底都做了什么，而是要搞清楚整个<strong data-nodeid="8789">调用链路</strong>是什么样的。如果我们能够理解 Hooks 在每个关键环节都做了哪些事情，同时也能理解这些关键环节是如何对最终的渲染结果产生影响的，那么理解 Hooks 的工作机制对于你来说就不在话下了。</p>
<h4 data-nodeid="8676">以 useState 为例，分析 React-Hooks 的调用链路</h4>
<p data-nodeid="8677">首先要说明的是，React-Hooks 的调用链路在首次渲染和更新阶段是不同的，这里我将两个阶段的链路各总结进了两张大图里，我们依次来看。首先是首次渲染的过程，请看下图：</p>
<p data-nodeid="8678"><img src="https://s0.lgstatic.com/i/image/M00/67/59/Ciqc1F-hJYCAWVjCAAEtNT9pGHA170.png" alt="图片12.png" data-nodeid="8794"></p>
<p data-nodeid="8679">在这个流程中，useState 触发的一系列操作最后会落到 mountState 里面去，所以我们重点需要关注的就是 mountState 做了什么事情。以下我为你提取了 mountState 的源码：</p>
<pre class="lang-java" data-nodeid="8680"><code data-language="java"><span class="hljs-comment">// 进入 mounState 逻辑</span>
<span class="hljs-function">function <span class="hljs-title">mountState</span><span class="hljs-params">(initialState)</span> </span>{

  <span class="hljs-comment">// 将新的 hook 对象追加进链表尾部</span>
  <span class="hljs-keyword">var</span> hook = mountWorkInProgressHook();

  <span class="hljs-comment">// initialState 可以是一个回调，若是回调，则取回调执行后的值</span>
  <span class="hljs-keyword">if</span> (typeof initialState === <span class="hljs-string">'function'</span>) {
    <span class="hljs-comment">// $FlowFixMe: Flow doesn't like mixed types</span>
    initialState = initialState();
  }

  <span class="hljs-comment">// 创建当前 hook 对象的更新队列，这一步主要是为了能够依序保留 dispatch</span>
  <span class="hljs-keyword">const</span> queue = hook.queue = {
    last: <span class="hljs-keyword">null</span>,
    dispatch: <span class="hljs-keyword">null</span>,
    lastRenderedReducer: basicStateReducer,
    lastRenderedState: (initialState: any),
  };

  <span class="hljs-comment">// 将 initialState 作为一个“记忆值”存下来</span>
  hook.memoizedState = hook.baseState = initialState;

  <span class="hljs-comment">// dispatch 是由上下文中一个叫 dispatchAction 的方法创建的，这里不必纠结这个方法具体做了什么</span>
  <span class="hljs-keyword">var</span> dispatch = queue.dispatch = dispatchAction.bind(<span class="hljs-keyword">null</span>, currentlyRenderingFiber$<span class="hljs-number">1</span>, queue);
  <span class="hljs-comment">// 返回目标数组，dispatch 其实就是示例中常常见到的 setXXX 这个函数，想不到吧？哈哈</span>
  <span class="hljs-keyword">return</span> [hook.memoizedState, dispatch];
}
</code></pre>
<p data-nodeid="8681">从这段源码中我们可以看出，<strong data-nodeid="8801">mounState 的主要工作是初始化 Hooks</strong>。在整段源码中，最需要关注的是 mountWorkInProgressHook 方法，它为我们道出了 Hooks 背后的数据结构组织形式。以下是 mountWorkInProgressHook 方法的源码：</p>
<pre class="lang-java" data-nodeid="8682"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">mountWorkInProgressHook</span><span class="hljs-params">()</span> </span>{
  <span class="hljs-comment">// 注意，单个 hook 是以对象的形式存在的</span>
  <span class="hljs-keyword">var</span> hook = {
    memoizedState: <span class="hljs-keyword">null</span>,
    baseState: <span class="hljs-keyword">null</span>,
    baseQueue: <span class="hljs-keyword">null</span>,
    queue: <span class="hljs-keyword">null</span>,
    next: <span class="hljs-keyword">null</span>
  };
  <span class="hljs-keyword">if</span> (workInProgressHook === <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 这行代码每个 React 版本不太一样，但做的都是同一件事：将 hook 作为链表的头节点处理</span>
    firstWorkInProgressHook = workInProgressHook = hook;
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 若链表不为空，则将 hook 追加到链表尾部</span>
    workInProgressHook = workInProgressHook.next = hook;
  }
  <span class="hljs-comment">// 返回当前的 hook</span>
  <span class="hljs-keyword">return</span> workInProgressHook;
}
</code></pre>
<p data-nodeid="8683">到这里可以看出，<strong data-nodeid="8807">hook 相关的所有信息收敛在一个 hook 对象里，而 hook 对象之间以单向链表的形式相互串联</strong>。</p>
<p data-nodeid="8684">接下来我们再看更新过程的大图：</p>
<p data-nodeid="8685"><img src="https://s0.lgstatic.com/i/image/M00/67/59/Ciqc1F-hJTGANs5yAAD4e6ACv8Q643.png" alt="图片13.png" data-nodeid="8811"></p>
<p data-nodeid="8686">根据图中高亮部分的提示不难看出，首次渲染和更新渲染的区别，在于调用的是 mountState，还是 updateState。mountState 做了什么，你已经非常清楚了；而 updateState 之后的操作链路，虽然涉及的代码有很多，但其实做的事情很容易理解：<strong data-nodeid="8817">按顺序去遍历之前构建好的链表，取出对应的数据信息进行渲染</strong>。</p>
<p data-nodeid="8687">我们把 mountState 和 updateState 做的事情放在一起来看：mountState（首次渲染）构建链表并渲染；updateState 依次遍历链表并渲染。</p>
<p data-nodeid="8688">看到这里，你是不是已经大概知道怎么回事儿了？没错，<strong data-nodeid="8824">hooks 的渲染是通过“依次遍历”来定位每个 hooks 内容的。如果前后两次读到的链表在顺序上出现差异，那么渲染的结果自然是不可控的</strong>。</p>
<p data-nodeid="8689">这个现象有点像我们构建了一个长度确定的数组，数组中的每个坑位都对应着一块确切的信息，后续每次从数组里取值的时候，只能够通过索引（也就是位置）来定位数据。也正因为如此，在许多文章里，都会直截了当地下这样的定义：Hooks 的本质就是数组。但读完这一课时的内容你就会知道，<strong data-nodeid="8830">Hooks 的本质其实是链表</strong>。</p>
<p data-nodeid="8690">接下来我们把这个已知的结论还原到 PersonalInfoComponent 里去，看看实际项目中，变量到底是怎么发生变化的。</p>
<h3 data-nodeid="8691">站在底层视角，重现 PersonalInfoComponent 组件的执行过程</h3>
<p data-nodeid="8692">我们先来复习一下修改过后的 PersonalInfoComponent 组件代码：</p>
<pre class="lang-js" data-nodeid="8693"><code data-language="js"><span class="hljs-keyword">import</span> React, { useState } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-comment">// isMounted 用于记录是否已挂载（是否是首次渲染）</span>
<span class="hljs-keyword">let</span> isMounted = <span class="hljs-literal">false</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">PersonalInfoComponent</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-comment">// 定义变量的逻辑不变</span>
  <span class="hljs-keyword">let</span> name, age, career, setName, setCareer;

  <span class="hljs-comment">// 这里追加对 isMounted 的输出，这是一个 debug 性质的操作</span>
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"isMounted is"</span>, isMounted);
  <span class="hljs-comment">// 这里追加 if 逻辑：只有在首次渲染（组件还未挂载）时，才获取 name、age 两个状态</span>
  <span class="hljs-keyword">if</span> (!isMounted) {
    <span class="hljs-comment">// eslint-disable-next-line</span>
    [name, setName] = useState(<span class="hljs-string">"修言"</span>);
    <span class="hljs-comment">// eslint-disable-next-line</span>
    [age] = useState(<span class="hljs-string">"99"</span>);

    <span class="hljs-comment">// if 内部的逻辑执行一次后，就将 isMounted 置为 true（说明已挂载，后续都不再是首次渲染了）</span>
    isMounted = <span class="hljs-literal">true</span>;
  }

  <span class="hljs-comment">// 对职业信息的获取逻辑不变</span>
  [career, setCareer] = useState(<span class="hljs-string">"我是一个前端，爱吃小熊饼干"</span>);
  <span class="hljs-comment">// 这里追加对 career 的输出，这也是一个 debug 性质的操作</span>
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"career"</span>, career);
  <span class="hljs-comment">// UI 逻辑的改动在于，name 和 age 成了可选的展示项，若值为空，则不展示</span>
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"personalInfo"</span>&gt;</span>
      {name ? <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>姓名：{name}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span> : null}
      {age ? <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>年龄：{age}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span> : null}
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>职业：{career}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span>
        <span class="hljs-attr">onClick</span>=<span class="hljs-string">{()</span> =&gt;</span> {
          setName("秀妍");
        }}
      &gt;
        修改姓名
      <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> PersonalInfoComponent;
</code></pre>
<p data-nodeid="8694">从代码里面，我们可以提取出来的 useState 调用有三个：</p>
<pre class="lang-java" data-nodeid="8695"><code data-language="java">[name, setName] = useState(<span class="hljs-string">"修言"</span>);
[age] = useState(<span class="hljs-string">"99"</span>);
[career, setCareer] = useState(<span class="hljs-string">"我是一个前端，爱吃小熊饼干"</span>);
</code></pre>
<p data-nodeid="8696">这三个调用在首次渲染的时候都会发生，伴随而来的链表结构如图所示：</p>
<p data-nodeid="8697"><img src="https://s0.lgstatic.com/i/image/M00/67/59/Ciqc1F-hJUWAe27kAAC_6mxli_Q918.png" alt="图片14.png" data-nodeid="8838"></p>
<p data-nodeid="8698">当首次渲染结束，进行二次渲染的时候，实际发生的 useState 调用只有一个：</p>
<pre class="lang-java" data-nodeid="8699"><code data-language="java">useState(<span class="hljs-string">"我是一个前端，爱吃小熊饼干"</span>)
</code></pre>
<p data-nodeid="8700">而此时的链表情况如下图所示：</p>
<p data-nodeid="8701"><img src="https://s0.lgstatic.com/i/image/M00/67/65/CgqCHl-hJeCAY_aoAAF7Tt5bK8k880.png" alt="图片15.png" data-nodeid="8843"></p>
<p data-nodeid="8702">我们再复习一遍更新（二次渲染）的时候会发生什么事情：updateState 会依次遍历链表、读取数据并渲染。注意这个过程就像从数组中依次取值一样，是完全按照顺序（或者说索引）来的。因此 React 不会看你命名的变量名是 career 还是别的什么，它只认你这一次 useState 调用，于是它难免会认为：<strong data-nodeid="8849">喔，原来你想要的是第一个位置的 hook 啊</strong>。</p>
<p data-nodeid="8703">然后就会有下面这样的效果：</p>
<p data-nodeid="8704"><img src="https://s0.lgstatic.com/i/image/M00/67/65/CgqCHl-hJe2ATIhGAAHpze3gFHg893.png" alt="图片16.png" data-nodeid="8853"></p>
<p data-nodeid="8705">如此一来，career 就自然而然地取到了链表头节点 hook 对象中的“秀妍”这个值。</p>
<h3 data-nodeid="8706">总结</h3>
<p data-nodeid="8707">三个课时学完了，到这里，我们对 React-Hooks 的学习，才终于算是告一段落。</p>
<p data-nodeid="8708">在过去的三个课时里，我们摸排了“动机”，认知了“工作模式”，最后更是结合源码、深挖了一把 React-Hooks 的底层原理。我们所做的这所有的努力，都是为了能够真正吃透 React-Hooks，不仅要确保实践中不出错，还要做到面试时有底气。</p>
<p data-nodeid="8709" class="">接下来，我们就将进入整个专栏真正的“深水区”，逐步切入“虚拟 DOM → Diff 算法 → Fiber 架构”这个知识链路里来。在后续的学习中，我们将延续并且强化这种“刨根问底”的风格，紧贴源码、原理和面试题来向 React 最为核心的部分发起挑战。真正的战斗，才刚刚开始，大家加油~</p>

---

### 精选评论

##### *飞：
> 抛一个疑问哈，react用链表来严格保证hooks的顺序，这样设计的理念是什么呢？为什么不用其他方法来取state，取消固定顺序岂不是更加灵活吗？

##### **琳：
> 很认真的在学习 突然看都秀妍 有点出戏到1988 “要叫秀妍不然考不上大学”

##### **逸：
> update*/set*这些函数是在挂载的时候就返回的dispatch函数，是与每个变量绑定的dispatch，所以在执行更新变量操作的时候回更新到对应的值，但是在更新渲染时（即第二次开始的渲染），useState获取值是从queue中按照顺序获取值的，所以处在if判断条件中没有执行的语句不会执行，所以获取不到，自然就给到了career这个变量

##### **蓉：
> 那按照React 团队给出的两条 React-Hooks 的使用原则原话来看，除了usState, 那其他的钩子也不能在循环、条件或嵌套函数中调用，比方说，useContext、useRef、useEffect、我想问下，是其他的钩子也是和useState一样用的链表吗

##### Kevin：
> 17 提示setName is not a function

##### **杰：
> 有个疑问，必须按顺序调用和使用链表结构这两者有必然关联吗？其他数据结构也支持顺序遍历的吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 必须按照顺序调用从根本上来说是因为 useState 这个钩子在设计层面并没有“状态命名”这个动作，也就是说你每生成一个新的状态，React 并不知道这个状态名字叫啥，所以需要通过顺序来索引到对应的状态值。文中并没有提及这两者之间存在必然关联，只是结合源码对现象作分析，而我们看到的客观事实是，React 用了链表。

##### *浩：
> ‘career 就自然而然地取到了链表头节点 hook 对象中的“秀妍”这个值？这个地方有点不明白，为什么更新的时候顺序调用链表，是career 的值被修改了？’说下个人观点。首次渲染的时候注册了三个state，但是二次渲染的时候，只剩下setCareer这一个state，更新的时候不回去匹配你的name，会直接吧“秀妍”赋给career。

##### **超：
> 我的理解是第二次渲染更新fiber中存的hook链表时，react会按照组件中声明的hook顺序去返回对应位置的hook的返回内容，因此如果hook在判断中调用，将导致hook返回的state和setState被替换，与预期返回值不符，从而出现不可预测的问题。是这样吗😮

##### *飞：
> career 就自然而然地取到了链表头节点 hook 对象中的“秀妍”这个值？这个地方有点不明白，为什么更新的时候顺序调用链表，是career 的值被修改了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 答案在原文中找哈：“updateState 会依次遍历链表、读取数据并渲染。注意这个过程就像从数组中依次取值一样，是完全按照顺序（或者说索引）来的。因此 React 不会看你命名的变量名是 career 还是别的什么，它只认你这一次 useState 调用，于是它难免会认为：喔，原来你想要的是第一个位置的 hook 啊。”把握住按照顺序取值这个点。

##### **阳：
> 这一节讲的很棒啊，期待更新

##### **宇：
> 以前还疑惑为什么它不会每次执行的时候被初始值覆盖掉，现在终于明白了为什么要顺序调用，以及useState如何获取值的。点赞！

##### **森：
> 打卡。组件对hooks的执行顺序有严格的要求，原因可以从mount和update的调用链路分析得出；本质是链表。以后面试更有底气了~~

##### **2533：
> react 17 如果hook用在条件判断里，setName会直接报 is not a function了

##### *帅：
> 请问老师，是否可以理解为不管首次渲染还是更新渲染，都会在函数组件从头到尾执行一遍useState，首次渲染执行时创建链表，更新渲染时按顺序更新链表，如果链表多了或少了就乱套了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 后半部分的理解是ok的。“从头到尾执行一遍useState”这句话不知道你如何理解，不过能够把握住“内部的逻辑在创建和更新时存在区别”这一点就不错的。整体的理解问题不大。

##### **汐文：
> 核心：hooks是链表

##### **男：
> 你们的setName点击之后不会报错吗？？？？😥

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 注意React版本是否与文中一致，以及代码复制粘贴时是否保留了对 eslint 的禁用

##### *聪：
> 看完全文疑问越来越多了。。。第二次调用的是setName而不是useState，是setName也被替换了？在链表中是不是存了setName这个标识然后把useState整个替换了？另这种链表结构有什么优势吗？why 需要选择这种结构？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; useState传入的参数是初始值，非初次渲染不会去找初始值，而是去找更新后的值，也就是去链表中按照顺序查找。链表没有替换 useState，要理解 state 的存储不是以变量名为索引的，而是依赖它在链表中的顺序。

##### *璇：
> 看到小熊饼干，真的好饿呀~

##### **磊：
> 老师的react 版本是多少？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 本讲Demo基于React 16.8x 版本进行演示。

##### **科：
> 如果是：“updateState 会依次遍历链表、读取数据并渲染”。是不是就以为着，我们定义的stateName。只是做为面向我们的标识符。我们我们执行某个操作，去修改state 调用了某个setState。它是完整的走了这个链表吗？

##### *杰：
> 还是有不太理解的地方，有if的情况下，setName找到的是链表第一个元素，没有if，就能找到对应的name，这是为什么？链表不是在第一次已经创建好了吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 链表是一开始就创建好了，但是查找到链表中的哪个元素是代码的执行逻辑决定的。

##### **文：
> 讲的很好

##### **宇：
> 老师能不能讲下diff的实现

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Diff 的逻辑一直是比较稳定的，在React各个大版本中没有原则性的变化，有的只是栈调和和Fiber调和这两种实现姿势。第10讲以栈调和为例对 Diff 进行了分析，可以去看看。

