<p data-nodeid="547" class="">上一节课我们梳理了组件渲染的过程，本质上就是把各种类型的 vnode 渲染成真实 DOM。我们也知道了组件是由模板、组件描述对象和数据构成的，数据的变化会影响组件的变化。组件的渲染过程中创建了一个带副作用的渲染函数，当数据变化的时候就会执行这个渲染函数来触发组件的更新。那么接下来，我们就具体分析一下组件的更新过程。</p>
<h3 data-nodeid="548">副作用渲染函数更新组件的过程</h3>
<p data-nodeid="549">我们先来回顾一下带副作用渲染函数 setupRenderEffect 的实现，但是这次我们要重点关注更新组件部分的逻辑：</p>
<pre class="lang-java" data-nodeid="550"><code data-language="java"><span class="hljs-keyword">const</span> setupRenderEffect = (instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized) =&gt; {
  <span class="hljs-comment">// 创建响应式的副作用渲染函数</span>
  instance.update = effect(<span class="hljs-function">function <span class="hljs-title">componentEffect</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">if</span> (!instance.isMounted) {
      <span class="hljs-comment">// 渲染组件</span>
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 更新组件</span>
      let { next, vnode } = instance
      <span class="hljs-comment">// next 表示新的组件 vnode</span>
      <span class="hljs-keyword">if</span> (next) {
        <span class="hljs-comment">// 更新组件 vnode 节点信息</span>
        updateComponentPreRender(instance, next, optimized)
      }
      <span class="hljs-keyword">else</span> {
        next = vnode
      }
      <span class="hljs-comment">// 渲染新的子树 vnode</span>
      <span class="hljs-keyword">const</span> nextTree = renderComponentRoot(instance)
      <span class="hljs-comment">// 缓存旧的子树 vnode</span>
      <span class="hljs-keyword">const</span> prevTree = instance.subTree
      <span class="hljs-comment">// 更新子树 vnode</span>
      instance.subTree = nextTree
      <span class="hljs-comment">// 组件更新核心逻辑，根据新旧子树 vnode 做 patch</span>
      patch(prevTree, nextTree,
        <span class="hljs-comment">// 如果在 teleport 组件中父节点可能已经改变，所以容器直接找旧树 DOM 元素的父节点</span>
        hostParentNode(prevTree.el),
        <span class="hljs-comment">// 参考节点在 fragment 的情况可能改变，所以直接找旧树 DOM 元素的下一个节点</span>
        getNextHostNode(prevTree),
        instance,
        parentSuspense,
        isSVG)
      <span class="hljs-comment">// 缓存更新后的 DOM 节点</span>
      next.el = nextTree.el
    }
  }, prodEffectOptions)
}
</code></pre>
<p data-nodeid="551">可以看到，更新组件主要做三件事情：<strong data-nodeid="629">更新组件 vnode 节点、渲染新的子树 vnode、根据新旧子树 vnode 执行 patch 逻辑</strong>。</p>
<p data-nodeid="552">首先是更新组件 vnode 节点，这里会有一个条件判断，判断组件实例中是否有新的组件 vnode（用 next 表示），有则更新组件 vnode，没有 next 指向之前的组件 vnode。为什么需要判断，这其实涉及一个组件更新策略的逻辑，我们稍后会讲。</p>
<p data-nodeid="553">接着是渲染新的子树 vnode，因为数据发生了变化，模板又和数据相关，所以渲染生成的子树 vnode 也会发生相应的变化。</p>
<p data-nodeid="554">最后就是<strong data-nodeid="637">核心的 patch 逻辑</strong>，用来找出新旧子树 vnode 的不同，并找到一种合适的方式更新 DOM，接下来我们就来分析这个过程。</p>
<h4 data-nodeid="555">核心逻辑：patch 流程</h4>
<p data-nodeid="556">我们先来看 patch 流程的实现代码：</p>
<pre class="lang-java" data-nodeid="557"><code data-language="java"><span class="hljs-keyword">const</span> patch = (n1, n2, container, anchor = <span class="hljs-keyword">null</span>, parentComponent = <span class="hljs-keyword">null</span>, parentSuspense = <span class="hljs-keyword">null</span>, isSVG = <span class="hljs-keyword">false</span>, optimized = <span class="hljs-keyword">false</span>) =&gt; {
  <span class="hljs-comment">// 如果存在新旧节点, 且新旧节点类型不同，则销毁旧节点</span>
  <span class="hljs-keyword">if</span> (n1 &amp;&amp; !isSameVNodeType(n1, n2)) {
    anchor = getNextHostNode(n1)
    unmount(n1, parentComponent, parentSuspense, <span class="hljs-keyword">true</span>)
    <span class="hljs-comment">// n1 设置为 null 保证后续都走 mount 逻辑</span>
    n1 = <span class="hljs-keyword">null</span>
  }
  <span class="hljs-keyword">const</span> { type, shapeFlag } = <span class="hljs-function">n2
  <span class="hljs-title">switch</span> <span class="hljs-params">(type)</span> </span>{
    <span class="hljs-keyword">case</span> Text:
      <span class="hljs-comment">// 处理文本节点</span>
      <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">case</span> Comment:
      <span class="hljs-comment">// 处理注释节点</span>
      <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">case</span> Static:
      <span class="hljs-comment">// 处理静态节点</span>
      <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">case</span> Fragment:
      <span class="hljs-comment">// 处理 Fragment 元素</span>
      <span class="hljs-keyword">break</span>
    <span class="hljs-keyword">default</span>:
      <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">1</span> <span class="hljs-comment">/* ELEMENT */</span>) {
        <span class="hljs-comment">// 处理普通 DOM 元素</span>
        processElement(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">6</span> <span class="hljs-comment">/* COMPONENT */</span>) {
        <span class="hljs-comment">// 处理组件</span>
        processComponent(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">64</span> <span class="hljs-comment">/* TELEPORT */</span>) {
        <span class="hljs-comment">// 处理 TELEPORT</span>
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">128</span> <span class="hljs-comment">/* SUSPENSE */</span>) {
        <span class="hljs-comment">// 处理 SUSPENSE</span>
      }
  }
}
<span class="hljs-function">function <span class="hljs-title">isSameVNodeType</span> <span class="hljs-params">(n1, n2)</span> </span>{
  <span class="hljs-comment">// n1 和 n2 节点的 type 和 key 都相同，才是相同节点</span>
  <span class="hljs-keyword">return</span> n1.type === n2.type &amp;&amp; n1.key === n2.key
}
</code></pre>
<p data-nodeid="558">在这个过程中，首先判断新旧节点是否是相同的 vnode 类型，如果不同，比如一个 div 更新成一个 ul，那么最简单的操作就是删除旧的 div 节点，再去挂载新的 ul 节点。</p>
<p data-nodeid="559">如果是相同的 vnode 类型，就需要走 diff 更新流程了，接着会根据不同的 vnode 类型执行不同的处理逻辑，这里我们仍然只分析普通元素类型和组件类型的处理过程。</p>
<h5 data-nodeid="560">1. 处理组件</h5>
<p data-nodeid="561">如何<strong data-nodeid="650">处理组件</strong>的呢？举个例子，我们在父组件 App 中里引入了 Hello 组件：</p>
<pre class="lang-js" data-nodeid="729"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>This is an app.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">hello</span> <span class="hljs-attr">:msg</span>=<span class="hljs-string">"msg"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">hello</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"toggle"</span>&gt;</span>Toggle msg<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">msg</span>: <span class="hljs-string">'Vue'</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      toggle() {
        <span class="hljs-built_in">this</span>.msg = <span class="hljs-built_in">this</span>.msg ==== <span class="hljs-string">'Vue'</span>? <span class="hljs-string">'World'</span>: <span class="hljs-string">'Vue'</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>

<p data-nodeid="563">Hello 组件中是 <code data-backticks="1" data-nodeid="652">&lt;div&gt;</code> 包裹着一个 <code data-backticks="1" data-nodeid="654">&lt;p&gt;</code> 标签， 如下所示：</p>
<pre class="lang-js" data-nodeid="564"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"hello"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Hello, {{msg}}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">props</span>: {
      <span class="hljs-attr">msg</span>: <span class="hljs-built_in">String</span>
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="565">点击 App 组件中的按钮执行 toggle 函数，就会修改 data 中的 msg，并且会触发App 组件的重新渲染。</p>
<p data-nodeid="566">结合前面对渲染函数的流程分析，这里 App 组件的根节点是 div 标签，重新渲染的子树 vnode 节点是一个普通元素的 vnode，应该先走 processElement 逻辑。组件的更新最终还是要转换成内部真实 DOM 的更新，而实际上普通元素的处理流程才是真正做 DOM 的更新，由于稍后我们会详细分析普通元素的处理流程，所以我们先跳过这里，继续往下看。</p>
<p data-nodeid="567">和渲染过程类似，更新过程也是一个树的深度优先遍历过程，更新完当前节点后，就会遍历更新它的子节点，因此在遍历的过程中会遇到 hello 这个组件 vnode 节点，就会执行到 processComponent 处理逻辑中，我们再来看一下它的实现，我们重点关注一下组件更新的相关逻辑：</p>
<pre class="lang-java" data-nodeid="568"><code data-language="java"><span class="hljs-keyword">const</span> processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized) =&gt; {
  <span class="hljs-keyword">if</span> (n1 == <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 挂载组件</span>
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 更新子组件</span>
    updateComponent(n1, n2, parentComponent, optimized)
  }
}
<span class="hljs-keyword">const</span> updateComponent = (n1, n2, parentComponent, optimized) =&gt; {
  <span class="hljs-keyword">const</span> instance = (n2.component = n1.component)
  <span class="hljs-comment">// 根据新旧子组件 vnode 判断是否需要更新子组件</span>
  <span class="hljs-keyword">if</span> (shouldUpdateComponent(n1, n2, parentComponent, optimized)) {
    <span class="hljs-comment">// 新的子组件 vnode 赋值给 instance.next</span>
    instance.next = n2
    <span class="hljs-comment">// 子组件也可能因为数据变化被添加到更新队列里了，移除它们防止对一个子组件重复更新</span>
    invalidateJob(instance.update)
    <span class="hljs-comment">// 执行子组件的副作用渲染函数</span>
    instance.update()
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 不需要更新，只复制属性</span>
    n2.component = n1.component
    n2.el = n1.el
  }
}
</code></pre>
<p data-nodeid="569">可以看到，processComponent 主要通过执行 updateComponent 函数来更新子组件，updateComponent 函数在更新子组件的时候，会先执行 shouldUpdateComponent 函数，根据新旧子组件 vnode 来判断是否需要更新子组件。这里你只需要知道，在 shouldUpdateComponent 函数的内部，主要是通过检测和对比组件 vnode 中的 props、chidren、dirs、transiton 等属性，来决定子组件是否需要更新。</p>
<p data-nodeid="570">这是很好理解的，因为在一个组件的子组件是否需要更新，我们主要依据子组件 vnode 是否存在一些会影响组件更新的属性变化进行判断，如果存在就会更新子组件。</p>
<p data-nodeid="571">虽然 Vue.js 的更新粒度是组件级别的，组件的数据变化只会影响当前组件的更新，但是在组件更新的过程中，也会对子组件做一定的检查，判断子组件是否也要更新，并通过某种机制避免子组件重复更新。</p>
<p data-nodeid="572">我们接着看 updateComponent 函数，如果 shouldUpdateComponent 返回 true ，那么在它的最后，先执行 invalidateJob（instance.update）避免子组件由于自身数据变化导致的重复更新，然后又执行了子组件的副作用渲染函数 instance.update 来主动触发子组件的更新。</p>
<p data-nodeid="573">再回到副作用渲染函数中，有了前面的讲解，我们再看组件更新的这部分代码，就能很好地理解它的逻辑了：</p>
<pre class="lang-java" data-nodeid="574"><code data-language="java"><span class="hljs-comment">// 更新组件</span>
let { next, vnode } = instance
<span class="hljs-comment">// next 表示新的组件 vnode</span>
<span class="hljs-keyword">if</span> (next) {
  <span class="hljs-comment">// 更新组件 vnode 节点信息</span>
  updateComponentPreRender(instance, next, optimized)
}
<span class="hljs-keyword">else</span> {
  next = vnode
}
<span class="hljs-keyword">const</span> updateComponentPreRender = (instance, nextVNode, optimized) =&gt; {
  <span class="hljs-comment">// 新组件 vnode 的 component 属性指向组件实例</span>
  nextVNode.component = instance
  <span class="hljs-comment">// 旧组件 vnode 的 props 属性</span>
  <span class="hljs-keyword">const</span> prevProps = instance.vnode.props
  <span class="hljs-comment">// 组件实例的 vnode 属性指向新的组件 vnode</span>
  instance.vnode = nextVNode
  <span class="hljs-comment">// 清空 next 属性，为了下一次重新渲染准备</span>
  instance.next = <span class="hljs-keyword">null</span>
  <span class="hljs-comment">// 更新 props</span>
  updateProps(instance, nextVNode.props, prevProps, optimized)
  <span class="hljs-comment">// 更新 插槽</span>
  updateSlots(instance, nextVNode.children)
}
</code></pre>
<p data-nodeid="575">结合上面的代码，我们在更新组件的 DOM 前，需要先更新组件 vnode 节点信息，包括更改组件实例的 vnode 指针、更新 props 和更新插槽等一系列操作，因为组件在稍后执行 renderComponentRoot 时会重新渲染新的子树 vnode ，它依赖了更新后的组件 vnode 中的 props 和 slots 等数据。</p>
<p data-nodeid="576">所以我们现在知道了一个组件重新渲染可能会有两种场景，一种是组件本身的数据变化，这种情况下 next 是 null；另一种是父组件在更新的过程中，遇到子组件节点，先判断子组件是否需要更新，如果需要则主动执行子组件的重新渲染方法，这种情况下 next 就是新的子组件 vnode。</p>
<p data-nodeid="577">你可能还会有疑问，这个子组件对应的新的组件 vnode 是什么时候创建的呢？答案很简单，它是在父组件重新渲染的过程中，通过 renderComponentRoot 渲染子树 vnode 的时候生成，因为子树 vnode 是个树形结构，通过遍历它的子节点就可以访问到其对应的组件 vnode。再拿我们前面举的例子说，当 App 组件重新渲染的时候，在执行 renderComponentRoot 生成子树 vnode 的过程中，也生成了 hello 组件对应的新的组件 vnode。</p>
<p data-nodeid="578">所以 processComponent 处理组件 vnode，本质上就是去判断子组件是否需要更新，如果需要则递归执行子组件的副作用渲染函数来更新，否则仅仅更新一些 vnode 的属性，并让子组件实例保留对组件 vnode 的引用，用于子组件自身数据变化引起组件重新渲染的时候，在渲染函数内部可以拿到新的组件 vnode。</p>
<p data-nodeid="579">前面也说过，组件是抽象的，组件的更新最终还是会落到对普通 DOM 元素的更新。所以接下来我们详细分析一下组件更新中<strong data-nodeid="673">对普通元素</strong>的处理流程。</p>
<h5 data-nodeid="580">2. 处理普通元素</h5>
<p data-nodeid="581">我们再来看如何处理普通元素，我把之前的示例稍加修改，将其中的 Hello 组件删掉，如下所示：</p>
<pre class="lang-js" data-nodeid="582"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>This is {{msg}}.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"toggle"</span>&gt;</span>Toggle msg<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">msg</span>: <span class="hljs-string">'Vue'</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      toggle() {
        <span class="hljs-built_in">this</span>.msg = <span class="hljs-string">'Vue'</span>? <span class="hljs-string">'World'</span>: <span class="hljs-string">'Vue'</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="583">当我们点击 App 组件中的按钮会执行 toggle 函数，然后修改 data 中的 msg，这就触发了 App 组件的重新渲染。</p>
<p data-nodeid="584">App 组件的根节点是 div 标签，重新渲染的子树 vnode 节点是一个普通元素的 vnode，所以应该先走 processElement 逻辑，我们来看这个函数的实现：</p>
<pre class="lang-java" data-nodeid="585"><code data-language="java"><span class="hljs-keyword">const</span> processElement = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized) =&gt; {
  isSVG = isSVG || n2.type === <span class="hljs-string">'svg'</span>
  <span class="hljs-keyword">if</span> (n1 == <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 挂载元素</span>
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 更新元素</span>
    patchElement(n1, n2, parentComponent, parentSuspense, isSVG, optimized)
  }
}
<span class="hljs-keyword">const</span> patchElement = (n1, n2, parentComponent, parentSuspense, isSVG, optimized) =&gt; {
  <span class="hljs-keyword">const</span> el = (n2.el = n1.el)
  <span class="hljs-keyword">const</span> oldProps = (n1 &amp;&amp; n1.props) || EMPTY_OBJ
  <span class="hljs-keyword">const</span> newProps = n2.props || EMPTY_OBJ
  <span class="hljs-comment">// 更新 props</span>
  patchProps(el, n2, oldProps, newProps, parentComponent, parentSuspense, isSVG)
  <span class="hljs-keyword">const</span> areChildrenSVG = isSVG &amp;&amp; n2.type !== <span class="hljs-string">'foreignObject'</span>
  <span class="hljs-comment">// 更新子节点</span>
  patchChildren(n1, n2, el, <span class="hljs-keyword">null</span>, parentComponent, parentSuspense, areChildrenSVG)
}
</code></pre>
<p data-nodeid="586">可以看到，更新元素的过程主要做两件事情：更新 props 和更新子节点。其实这是很好理解的，因为一个 DOM 节点元素就是由它自身的一些属性和子节点构成的。</p>
<p data-nodeid="587">首先是更新 props，这里的 patchProps 函数就是在更新 DOM 节点的 class、style、event 以及其它的一些 DOM 属性，这个过程我不再深入分析了，感兴趣的同学可以自己看这部分代码。</p>
<p data-nodeid="588">其次是更新子节点，我们来看一下这里的 patchChildren 函数的实现：</p>
<pre class="lang-java" data-nodeid="589"><code data-language="java"><span class="hljs-keyword">const</span> patchChildren = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized = <span class="hljs-keyword">false</span>) =&gt; {
  <span class="hljs-keyword">const</span> c1 = n1 &amp;&amp; n1.children
  <span class="hljs-keyword">const</span> prevShapeFlag = n1 ? n1.shapeFlag : <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> c2 = n2.children
  <span class="hljs-keyword">const</span> { shapeFlag } = n2
  <span class="hljs-comment">// 子节点有 3 种可能情况：文本、数组、空</span>
  <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">8</span> <span class="hljs-comment">/* TEXT_CHILDREN */</span>) {
    <span class="hljs-keyword">if</span> (prevShapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
      <span class="hljs-comment">// 数组 -&gt; 文本，则删除之前的子节点</span>
      unmountChildren(c1, parentComponent, parentSuspense)
    }
    <span class="hljs-keyword">if</span> (c2 !== c1) {
      <span class="hljs-comment">// 文本对比不同，则替换为新文本</span>
      hostSetElementText(container, c2)
    }
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-keyword">if</span> (prevShapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
      <span class="hljs-comment">// 之前的子节点是数组</span>
      <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
        <span class="hljs-comment">// 新的子节点仍然是数组，则做完整地 diff</span>
        patchKeyedChildren(c1, c2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
      <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 数组 -&gt; 空，则仅仅删除之前的子节点</span>
        unmountChildren(c1, parentComponent, parentSuspense, <span class="hljs-keyword">true</span>)
      }
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 之前的子节点是文本节点或者为空</span>
      <span class="hljs-comment">// 新的子节点是数组或者为空</span>
      <span class="hljs-keyword">if</span> (prevShapeFlag &amp; <span class="hljs-number">8</span> <span class="hljs-comment">/* TEXT_CHILDREN */</span>) {
        <span class="hljs-comment">// 如果之前子节点是文本，则把它清空</span>
        hostSetElementText(container, <span class="hljs-string">''</span>)
      }
      <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
        <span class="hljs-comment">// 如果新的子节点是数组，则挂载新子节点</span>
        mountChildren(c2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
    }
  }
}
</code></pre>
<p data-nodeid="590">对于一个元素的子节点 vnode 可能会有三种情况：纯文本、vnode 数组和空。那么根据排列组合对于新旧子节点来说就有九种情况，我们可以通过三张图来表示。</p>
<p data-nodeid="591">首先来看一下<strong data-nodeid="689">旧子节点是纯文本</strong>的情况：</p>
<ul data-nodeid="592">
<li data-nodeid="593">
<p data-nodeid="594">如果新子节点也是纯文本，那么做简单地文本替换即可；</p>
</li>
<li data-nodeid="595">
<p data-nodeid="596">如果新子节点是空，那么删除旧子节点即可；</p>
</li>
<li data-nodeid="597">
<p data-nodeid="598">如果新子节点是 vnode 数组，那么先把旧子节点的文本清空，再去旧子节点的父容器下添加多个新子节点。</p>
</li>
</ul>
<p data-nodeid="599"><img src="https://s0.lgstatic.com/i/image/M00/31/18/Ciqc1F8MBDWAfUAXAADe59XvjHY701.png" alt="2.png" data-nodeid="695"></p>
<p data-nodeid="600">接下来看一下<strong data-nodeid="701">旧子节点是空</strong>的情况：</p>
<ul data-nodeid="601">
<li data-nodeid="602">
<p data-nodeid="603">如果新子节点是纯文本，那么在旧子节点的父容器下添加新文本节点即可；</p>
</li>
<li data-nodeid="604">
<p data-nodeid="605">如果新子节点也是空，那么什么都不需要做；</p>
</li>
<li data-nodeid="606">
<p data-nodeid="607">如果新子节点是 vnode 数组，那么直接去旧子节点的父容器下添加多个新子节点即可。</p>
</li>
</ul>
<p data-nodeid="608"><img src="https://s0.lgstatic.com/i/image/M00/31/23/CgqCHl8MBEOANnFmAADYr-_R5mM894.png" alt="3.png" data-nodeid="707"></p>
<p data-nodeid="609">最后来看一下<strong data-nodeid="713">旧子节点是 vnode 数组</strong>的情况：</p>
<ul data-nodeid="610">
<li data-nodeid="611">
<p data-nodeid="612">如果新子节点是纯文本，那么先删除旧子节点，再去旧子节点的父容器下添加新文本节点；</p>
</li>
<li data-nodeid="613">
<p data-nodeid="614">如果新子节点是空，那么删除旧子节点即可；</p>
</li>
<li data-nodeid="615">
<p data-nodeid="616">如果新子节点也是 vnode 数组，那么就需要做完整的 diff 新旧子节点了，这是最复杂的情况，内部运用了核心 diff 算法。</p>
</li>
</ul>
<p data-nodeid="617"><img src="https://s0.lgstatic.com/i/image/M00/31/23/CgqCHl8MBCuAUZksAADplAU2718113.png" alt="1.png" data-nodeid="719"></p>
<p data-nodeid="618">下节课我们就来深入探究一下这个复杂的 diff 算法。</p>
<blockquote data-nodeid="619">
<p data-nodeid="620" class=""><strong data-nodeid="728">本节课的相关代码在源代码中的位置如下：</strong><br>
packages/runtime-core/src/renderer.ts<br>
packages/runtime-core/src/componentRenderUtils.ts</p>
</blockquote>

---

### 精选评论

##### *璕：
> 在3.0.3的版本中，文中的demo走不到patchChildren函数里面了，在patchElement函数内会直接判断newVnode是否达到有dynamicChildren，如果有dynamicChildren,直接进入patchBlockChildren函数,进行vnode的更新

##### **棋：
> 更新节点-vnode与oldvnode是否是静态节点(-YES-，退出patch)-NO-vnode是文本节点(-YES-，与oldvnode是文本节点且文本是否相同。YES：跳过，NO：替换)-NO-vnode与oldvnode都存在子节点(-YES-，对比每个子节点都走patch流程)-NO-只有vnode存在子节点(-YES-，清空oldvnode，插入vnode。oldvnode可能是文本节点所以清空)-NO-vnode是空节点(清空oldvnode)这是vue2.x的patch节点对比流程

##### **宇：
> 每天进步一点点~

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油哦~~

