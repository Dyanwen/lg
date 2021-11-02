<p data-nodeid="81187">我们都知道，Vue.js 的核心思想之一是组件化，组件就是 DOM 的映射，我们通过嵌套的组件构成了一个组件应用程序的树。</p>



<p data-nodeid="80836">但是，有些时候组件模板的一部分在逻辑上属于该组件，而从技术角度来看，最好将模板的这一部分移动到应用程序之外的其他位置。</p>
<p data-nodeid="80837">一个常见的场景是创建一个包含全屏模式的对话框组件。在大多数情况下，我们希望对话框的逻辑存在于组件中，但是对话框的定位 CSS 是一个很大的问题，它非常容易受到外层父组件的 CSS 影响。</p>
<p data-nodeid="80838">假设我们有这样一个 dialog 组件，用按钮来管理一个 dialog：</p>
<pre class="lang-js" data-nodeid="82753"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">v-show</span>=<span class="hljs-string">"visible"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"dialog"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"dialog-body"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>I'm a dialog!<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"visible=false"</span>&gt;</span>Close<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">visible</span>: <span class="hljs-literal">false</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      show() {
        <span class="hljs-built_in">this</span>.visible = <span class="hljs-literal">true</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">style</span>&gt;</span><span class="css">
  <span class="hljs-selector-class">.dialog</span> {
    <span class="hljs-attribute">position</span>: absolute;
    <span class="hljs-attribute">top</span>: <span class="hljs-number">0</span>; <span class="hljs-attribute">right</span>: <span class="hljs-number">0</span>; <span class="hljs-attribute">bottom</span>: <span class="hljs-number">0</span>; <span class="hljs-attribute">left</span>: <span class="hljs-number">0</span>;
    <span class="hljs-attribute">background-color</span>: <span class="hljs-built_in">rgba</span>(<span class="hljs-number">0</span>,<span class="hljs-number">0</span>,<span class="hljs-number">0</span>,.<span class="hljs-number">5</span>);
    <span class="hljs-attribute">display</span>: flex;
    <span class="hljs-attribute">flex-direction</span>: column;
    <span class="hljs-attribute">align-items</span>: center;
    <span class="hljs-attribute">justify-content</span>: center;
  }
  <span class="hljs-selector-class">.dialog</span> <span class="hljs-selector-class">.dialog-body</span> {
    <span class="hljs-attribute">display</span>: flex;
    <span class="hljs-attribute">flex-direction</span>: column;
    <span class="hljs-attribute">align-items</span>: center;
    <span class="hljs-attribute">justify-content</span>: center;
    <span class="hljs-attribute">background-color</span>: white;
    <span class="hljs-attribute">width</span>: <span class="hljs-number">300px</span>;
    <span class="hljs-attribute">height</span>: <span class="hljs-number">300px</span>;
    <span class="hljs-attribute">padding</span>: <span class="hljs-number">5px</span>;
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span></span>
</code></pre>
<p data-nodeid="82754">然后我们去使用这个组件：</p>
<pre class="lang-js" data-nodeid="83418"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"showDialog"</span>&gt;</span>Show dialog<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">Dialog</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"dialog"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Dialog</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">import</span> Dialog <span class="hljs-keyword">from</span> <span class="hljs-string">'./components/dialog'</span>
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">components</span>: {
      Dialog
    },
    <span class="hljs-attr">methods</span>: {
      showDialog() {
        <span class="hljs-built_in">this</span>.$refs.dialog.show()
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="83419">因为我们的 dialog 组件使用的是 position:absolute 绝对定位的方式，如果它的父级 DOM 有 position 不为 static 的布局方式，那么 dialog 的定位就受到了影响，不能按预期渲染了。</p>
<p data-nodeid="83420">所以一种好的解决方案是把 dialog 组件渲染的这部分 DOM 挂载到 body 下面，这样就不会受到父级样式的影响了。</p>
<p data-nodeid="84225" class="">在 Vue.js 2.x 中，想实现上面的需求，你可以依赖开源插件 <a href="https://github.com/LinusBorg/portal-vue" data-nodeid="84229">portal-vue</a> 或者是 <a href="https://github.com/cube-ui/vue-create-api" data-nodeid="84233">vue-create-api</a>，感兴趣可以自行了解。</p>



<p data-nodeid="83422">而 Vue.js 3.0 把这一能力内置到内核中，提供了一个内置组件 Teleport，它可以轻松帮助我们实现上述需求：</p>
<pre class="lang-js" data-nodeid="84777"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"showDialog"</span>&gt;</span>Show dialog<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">teleport</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"body"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">Dialog</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"dialog"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Dialog</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">teleport</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">import</span> Dialog <span class="hljs-keyword">from</span> <span class="hljs-string">'./components/dialog'</span>
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">components</span>: {
      Dialog
    },
    <span class="hljs-attr">methods</span>: {
      showDialog() {
        <span class="hljs-built_in">this</span>.$refs.dialog.show()
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="84778">Teleport 组件使用起来非常简单，套在想要在别处渲染的组件或者 DOM 节点的外部，然后通过 to 这个 prop 去指定渲染到的位置，to 可以是一个 DOM 选择器字符串，也可以是一个 DOM 节点。</p>
<p data-nodeid="84779">了解了使用方式，接下来，我们就来分析它的实现原理，看看 Teleport 是如何脱离当前组件渲染子组件的。</p>
<h3 data-nodeid="84780">Teleport 实现原理</h3>
<p data-nodeid="84781">对于这类内置组件，Vue.js 从编译阶段就做了特殊处理，我们先来看一下前面示例模板编译后的结果：</p>
<pre class="lang-js" data-nodeid="85396"><code data-language="js"><span class="hljs-keyword">import</span> { createVNode <span class="hljs-keyword">as</span> _createVNode, resolveComponent <span class="hljs-keyword">as</span> _resolveComponent, Teleport <span class="hljs-keyword">as</span> _Teleport, openBlock <span class="hljs-keyword">as</span> _openBlock, createBlock <span class="hljs-keyword">as</span> _createBlock } <span class="hljs-keyword">from</span> <span class="hljs-string">"vue"</span>
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">render</span>(<span class="hljs-params">_ctx, _cache, $props, $setup, $data, $options</span>) </span>{
  <span class="hljs-keyword">const</span> _component_Dialog = _resolveComponent(<span class="hljs-string">"Dialog"</span>)
  <span class="hljs-keyword">return</span> (_openBlock(), _createBlock(<span class="hljs-string">"template"</span>, <span class="hljs-literal">null</span>, [
    _createVNode(<span class="hljs-string">"button"</span>, { <span class="hljs-attr">onClick</span>: _ctx.showDialog }, <span class="hljs-string">"Show dialog"</span>, <span class="hljs-number">8</span> <span class="hljs-comment">/* PROPS */</span>, [<span class="hljs-string">"onClick"</span>]),
    (_openBlock(), _createBlock(_Teleport, { <span class="hljs-attr">to</span>: <span class="hljs-string">"body"</span> }, [
      _createVNode(_component_Dialog, { <span class="hljs-attr">ref</span>: <span class="hljs-string">"dialog"</span> }, <span class="hljs-literal">null</span>, <span class="hljs-number">512</span> <span class="hljs-comment">/* NEED_PATCH */</span>)
    ]))
  ]))
}
</code></pre>
<p data-nodeid="85397">可以看到，对于 teleport 标签，它是直接创建了 Teleport 内置组件，我们接下来来看它的实现：</p>
<pre class="lang-js" data-nodeid="86003"><code data-language="js"><span class="hljs-keyword">const</span> Teleport = {
  <span class="hljs-attr">__isTeleport</span>: <span class="hljs-literal">true</span>,
  process(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized, internals) {
    <span class="hljs-keyword">if</span> (n1 == <span class="hljs-literal">null</span>) {
      <span class="hljs-comment">// 创建逻辑</span>
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 更新逻辑</span>
    }
  },
  remove(vnode, { <span class="hljs-attr">r</span>: remove, <span class="hljs-attr">o</span>: { <span class="hljs-attr">remove</span>: hostRemove } }) {
    <span class="hljs-comment">// 删除逻辑</span>
  },
  <span class="hljs-attr">move</span>: moveTeleport,
  <span class="hljs-attr">hydrate</span>: hydrateTeleport
}
</code></pre>
<p data-nodeid="86004">Teleport 组件的实现就是一个对象，对外提供了几个方法。其中 process 方法负责组件的创建和更新逻辑，remove 方法负责组件的删除逻辑，接下来我们就从这三个方面来分析 Teleport 的实现原理。</p>
<h4 data-nodeid="86005">Teleport 组件创建</h4>
<p data-nodeid="86006">回顾组件创建的过程，会经历 patch 阶段，我们来回顾它的实现：</p>
<pre class="lang-js" data-nodeid="86600"><code data-language="js"><span class="hljs-keyword">const</span> patch = <span class="hljs-function">(<span class="hljs-params">n1, n2, container, anchor = <span class="hljs-literal">null</span>, parentComponent = <span class="hljs-literal">null</span>, parentSuspense = <span class="hljs-literal">null</span>, isSVG = <span class="hljs-literal">false</span>, optimized = <span class="hljs-literal">false</span></span>) =&gt;</span> {
  <span class="hljs-keyword">if</span> (n1 &amp;&amp; !isSameVNodeType(n1, n2)) {
    <span class="hljs-comment">// 如果存在新旧节点, 且新旧节点类型不同，则销毁旧节点</span>
  }
  <span class="hljs-keyword">const</span> { type, shapeFlag } = n2
  <span class="hljs-keyword">switch</span> (type) {
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
    <span class="hljs-attr">default</span>:
      <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">1</span> <span class="hljs-comment">/* ELEMENT */</span>) {
        <span class="hljs-comment">// 处理普通 DOM 元素</span>
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">6</span> <span class="hljs-comment">/* COMPONENT */</span>) {
        <span class="hljs-comment">// 处理组件</span>
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">64</span> <span class="hljs-comment">/* TELEPORT */</span>) {
        <span class="hljs-comment">// 处理 TELEPORT</span>
        type.process(n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized, internals);
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">128</span> <span class="hljs-comment">/* SUSPENSE */</span>) {
        <span class="hljs-comment">// 处理 SUSPENSE</span>
      }
  }
}
</code></pre>
<p data-nodeid="86601">可以看到，在 patch 阶段，会判断如果 type 是一个 Teleport 组件，则会执行它的 process 方法，接下来我们来看 process 方法关于 Teleport 组件创建部分的逻辑：</p>
<pre class="lang-js" data-nodeid="87187"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">process</span>(<span class="hljs-params">n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized, internals</span>) </span>{
  <span class="hljs-keyword">const</span> { <span class="hljs-attr">mc</span>: mountChildren, <span class="hljs-attr">pc</span>: patchChildren, <span class="hljs-attr">pbc</span>: patchBlockChildren, <span class="hljs-attr">o</span>: { insert, querySelector, createText, createComment } } = internals
  <span class="hljs-keyword">const</span> disabled = isTeleportDisabled(n2.props)
  <span class="hljs-keyword">const</span> { shapeFlag, children } = n2
  <span class="hljs-keyword">if</span> (n1 == <span class="hljs-literal">null</span>) {
    <span class="hljs-comment">// 在主视图里插入注释节点或者空白文本节点</span>
    <span class="hljs-keyword">const</span> placeholder = (n2.el = (process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)
      ? createComment(<span class="hljs-string">'teleport start'</span>)
      : createText(<span class="hljs-string">''</span>))
    <span class="hljs-keyword">const</span> mainAnchor = (n2.anchor = (process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)
      ? createComment(<span class="hljs-string">'teleport end'</span>)
      : createText(<span class="hljs-string">''</span>))
    insert(placeholder, container, anchor)
    insert(mainAnchor, container, anchor)
    <span class="hljs-comment">// 获取目标移动的 DOM 节点</span>
    <span class="hljs-keyword">const</span> target = (n2.target = resolveTarget(n2.props, querySelector))
    <span class="hljs-keyword">const</span> targetAnchor = (n2.targetAnchor = createText(<span class="hljs-string">''</span>))
    <span class="hljs-keyword">if</span> (target) {
      insert(targetAnchor, target)
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
      <span class="hljs-comment">// 查找不到 target 则报警告</span>
      warn(<span class="hljs-string">'Invalid Teleport target on mount:'</span>, target, <span class="hljs-string">`(<span class="hljs-subst">${<span class="hljs-keyword">typeof</span> target}</span>)`</span>)
    }
    <span class="hljs-keyword">const</span> mount = <span class="hljs-function">(<span class="hljs-params">container, anchor</span>) =&gt;</span> {
      <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
        <span class="hljs-comment">// 挂载子节点</span>
        mountChildren(children, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
      }
    }
    <span class="hljs-keyword">if</span> (disabled) {
      <span class="hljs-comment">// disabled 情况就在原先的位置挂载</span>
      mount(container, mainAnchor)
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (target) {
      <span class="hljs-comment">// 挂载到 target 的位置</span>
      mount(target, targetAnchor)
    }
  }
}
</code></pre>
<p data-nodeid="87188">Teleport 组件创建部分主要分为三个步骤，<strong data-nodeid="87225">第一步在主视图里插入注释节点或者空白文本节点</strong>，<strong data-nodeid="87226">第二步获取目标元素节点</strong>，<strong data-nodeid="87227">第三步往目标元素插入 Teleport 组件的子节点</strong>。</p>
<p data-nodeid="87189">我们先来看第一步，会在非生产环境往 Teleport 组件原本的位置插入注释节点，在生产环境插入空白文本节点。在开发环境中，组件的 el 对象指向 teleport start 注释节点，组件的 anchor 对象指向teleport end 注释节点。</p>
<p data-nodeid="87190">接着看第二步，会通过 resolveTarget 方法从 props 中的 to 属性以及 DOM 选择器拿到对应要移动到的目标元素 target。</p>
<p data-nodeid="87191">最后看第三步，会判断 disabled 变量的值，它是在 Teleport 组件中通过 prop 传递的，如果 disabled 为 true，那么子节点仍然挂载到 Teleport 原本视图的位置，如果为 false，那么子节点则挂载到 target 目标元素位置。</p>
<p data-nodeid="87192">至此，我们就已经实现了需求，把 Teleport 包裹的子节点脱离了当前组件，渲染到目标位置，是不是很简单呢？</p>
<h4 data-nodeid="87193">Teleport 组件更新</h4>
<p data-nodeid="87194">当然，Teleport 包裹的子节点渲染后并不是一成不变的，当组件发生更新的时候，仍然会执行 patch 逻辑走到 Teleport 的 process 方法，去处理 Teleport 组件的更新，我们来看一下这部分的实现：</p>
<pre class="lang-js" data-nodeid="87741"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">process</span>(<span class="hljs-params">n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized, internals</span>) </span>{
  <span class="hljs-keyword">const</span> { <span class="hljs-attr">mc</span>: mountChildren, <span class="hljs-attr">pc</span>: patchChildren, <span class="hljs-attr">pbc</span>: patchBlockChildren, <span class="hljs-attr">o</span>: { insert, querySelector, createText, createComment } } = internals
  <span class="hljs-keyword">const</span> disabled = isTeleportDisabled(n2.props)
  <span class="hljs-keyword">const</span> { shapeFlag, children } = n2
  <span class="hljs-keyword">if</span> (n1 == <span class="hljs-literal">null</span>) {
    <span class="hljs-comment">// 创建逻辑</span>
  }
  <span class="hljs-keyword">else</span> {
    n2.el = n1.el
    <span class="hljs-keyword">const</span> mainAnchor = (n2.anchor = n1.anchor)
    <span class="hljs-keyword">const</span> target = (n2.target = n1.target)
    <span class="hljs-keyword">const</span> targetAnchor = (n2.targetAnchor = n1.targetAnchor)
    <span class="hljs-comment">// 之前是不是 disabled 状态</span>
    <span class="hljs-keyword">const</span> wasDisabled = isTeleportDisabled(n1.props)
    <span class="hljs-keyword">const</span> currentContainer = wasDisabled ? container : target
    <span class="hljs-keyword">const</span> currentAnchor = wasDisabled ? mainAnchor : targetAnchor
    <span class="hljs-comment">// 更新子节点</span>
    <span class="hljs-keyword">if</span> (n2.dynamicChildren) {
      patchBlockChildren(n1.dynamicChildren, n2.dynamicChildren, currentContainer, parentComponent, parentSuspense, isSVG)
      <span class="hljs-keyword">if</span> (n2.shapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
        <span class="hljs-keyword">const</span> oldChildren = n1.children
        <span class="hljs-keyword">const</span> children = n2.children
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>; i &lt; children.length; i++) {
          <span class="hljs-keyword">if</span> (!children[i].el) {
            children[i].el = oldChildren[i].el
          }
        }
      }
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!optimized) {
      patchChildren(n1, n2, currentContainer, currentAnchor, parentComponent, parentSuspense, isSVG)
    }
    <span class="hljs-keyword">if</span> (disabled) {
      <span class="hljs-keyword">if</span> (!wasDisabled) {
        <span class="hljs-comment">// enabled -&gt; disabled</span>
        <span class="hljs-comment">// 把子节点移动回主容器</span>
        moveTeleport(n2, container, mainAnchor, internals, <span class="hljs-number">1</span> <span class="hljs-comment">/* TOGGLE */</span>)
      }
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">if</span> ((n2.props &amp;&amp; n2.props.to) !== (n1.props &amp;&amp; n1.props.to)) {
        <span class="hljs-comment">// 目标元素改变</span>
        <span class="hljs-keyword">const</span> nextTarget = (n2.target = resolveTarget(n2.props, querySelector))
        <span class="hljs-keyword">if</span> (nextTarget) {
          <span class="hljs-comment">// 移动到新的目标元素</span>
          moveTeleport(n2, nextTarget, <span class="hljs-literal">null</span>, internals, <span class="hljs-number">0</span> <span class="hljs-comment">/* TARGET_CHANGE */</span>)
        }
        <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
          warn(<span class="hljs-string">'Invalid Teleport target on update:'</span>, target, <span class="hljs-string">`(<span class="hljs-subst">${<span class="hljs-keyword">typeof</span> target}</span>)`</span>)
        }
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (wasDisabled) {
        <span class="hljs-comment">// disabled -&gt; enabled</span>
        <span class="hljs-comment">// 移动到目标元素位置</span>
        moveTeleport(n2, target, targetAnchor, internals, <span class="hljs-number">1</span> <span class="hljs-comment">/* TOGGLE */</span>)
      }
    }
  }
}
</code></pre>
<p data-nodeid="87742">Teleport 组件更新无非就是做几件事情：更新子节点，处理 disabled 属性变化的情况，处理 to 属性变化的情况。</p>
<p data-nodeid="87743">首先，是更新 Teleport 组件的子节点，这里更新分为优化更新和普通的全量比对更新两种情况，之前分析过，就不再赘述了。</p>
<p data-nodeid="87744">接着，是判断 Teleport 组件新节点配置 disabled 属性的情况，如果满足新节点 disabled 为 true，且旧节点的 disabled 为 false 的话，说明我们需要把 Teleport 的子节点从目标元素内部移回到主视图内部了。</p>
<p data-nodeid="87745">如果新节点 disabled 为 false，那么先通过 to 属性是否改变来判断目标元素 target 有没有变化，如果有变化，则把 Teleport 的子节点移动到新的 target 内部；如果目标元素没变化，则判断旧节点的 disabled 是否为 true，如果是则把 Teleport 的子节点从主视图内部移动到目标元素内部了。</p>
<h4 data-nodeid="87746">Teleport 组件移除</h4>
<p data-nodeid="87747">前面我们学过，当组件移除的时候会执行 unmount 方法，它的内部会判断如果移除的组件是一个 Teleport 组件，就会执行组件的 remove 方法：</p>
<pre class="lang-js" data-nodeid="88252"><code data-language="js"><span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">64</span> <span class="hljs-comment">/* TELEPORT */</span>) {
  vnode.type.remove(vnode, internals);
}
<span class="hljs-keyword">if</span> (doRemove) {
  remove(vnode);
}
</code></pre>
<p data-nodeid="88253">我们来看一下它的实现：</p>
<pre class="lang-js" data-nodeid="88747"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">remove</span>(<span class="hljs-params">vnode, { r: remove, o: { remove: hostRemove } }</span>) </span>{
  <span class="hljs-keyword">const</span> { shapeFlag, children, anchor } = vnode
  hostRemove(anchor)
  <span class="hljs-keyword">if</span> (shapeFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* ARRAY_CHILDREN */</span>) {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>; i &lt; children.length; i++) {
      remove(children[i])
    }
  }
}
</code></pre>
<p data-nodeid="88748">Teleport 的 remove 方法实现很简单，首先通过 hostRemove 移除主视图渲染的锚点 teleport start 注释节点，然后再去遍历 Teleport 的子节点执行 remove 移除。</p>
<p data-nodeid="88749">执行完 Teleport 的 remove 方法，会继续执行 remove 方法移除 Teleport 主视图的元素 teleport end 注释节点，至此，Teleport 组件完成了移除。</p>
<h3 data-nodeid="88750">总结</h3>
<p data-nodeid="88751">好的，到这里我们这一节的学习也要结束啦，通过这节课的学习，你应该了解了 Teleport 是如何把内部的子元素渲染到目标元素上，并且对 Teleport 组件是如何创建，更新和移除的有所理解。</p>
<p data-nodeid="88752">最后，给你留一道思考题，作为 Vue.js 的内置组件，它需要像用户自定义组件那样先注册后再使用吗？如果不用又是为什么呢？欢迎你在留言区与我分享。</p>
<blockquote data-nodeid="88753">
<p data-nodeid="88754">本节课的相关代码在源代码中的位置如下：<br>
packages/runtime-core/src/components/Teleport.ts<br>
packages/runtime-core/src/renderer.ts</p>
</blockquote>

---

### 精选评论

##### **3770：
> 内置组件，在编译模版函数就已经直接import进来component,不用resolvecomponent去查找

