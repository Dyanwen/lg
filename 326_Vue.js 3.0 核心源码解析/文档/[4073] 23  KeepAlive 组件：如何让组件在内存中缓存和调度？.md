<p data-nodeid="29118">通过前面的学习，我们了解到多个平行组件条件渲染，当满足条件的时候会触发某个组件的挂载，而已渲染的组件当条件不满足的时候会触发组件的卸载，举个例子：</p>



<pre class="lang-js" data-nodeid="29452"><code data-language="js">&lt;comp-a v-<span class="hljs-keyword">if</span>=<span class="hljs-string">"flag"</span>&gt;&lt;/comp-a&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">comp-b</span> <span class="hljs-attr">v-else</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">comp-b</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"flag=!flag"</span>&gt;</span>toggle<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
</code></pre>
<p data-nodeid="29453">这里，当 flag 为 true 的时候，就会触发组件 A 的渲染，然后我们点击按钮把 flag 修改为 false，又会触发组件 A 的卸载，及组件 B 的渲染。</p>
<p data-nodeid="29454">根据我们前面的学习，我们也知道组件的挂载和卸载都是一个递归过程，会有一定的性能损耗，对于这种可能会频繁切换的组件，我们有没有办法减少这其中的性能损耗呢？</p>
<p data-nodeid="29455">答案是有的，Vue.js 提供了内置组件 KeepAlive，我们可以这么使用它：</p>
<pre class="lang-js" data-nodeid="29948"><code data-language="js">&lt;keep-alive&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">comp-a</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"flag"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">comp-a</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">comp-b</span> <span class="hljs-attr">v-else</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">comp-b</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"flag=!flag"</span>&gt;</span>toggle<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
&lt;/keep-alive&gt;
</code></pre>
<p data-nodeid="29949">我们可以用模板导出工具看一下它编译后的 render 函数：</p>
<pre class="lang-java" data-nodeid="29950"><code data-language="java"><span class="hljs-keyword">import</span> { resolveComponent as _resolveComponent, createVNode as _createVNode, createCommentVNode as _createCommentVNode, KeepAlive as _KeepAlive, openBlock as _openBlock, createBlock as _createBlock } from <span class="hljs-string">"vue"</span>
<span class="hljs-function">export function <span class="hljs-title">render</span><span class="hljs-params">(_ctx, _cache, $props, $setup, $data, $options)</span> </span>{
  <span class="hljs-keyword">const</span> _component_comp_a = _resolveComponent(<span class="hljs-string">"comp-a"</span>)
  <span class="hljs-keyword">const</span> _component_comp_b = _resolveComponent(<span class="hljs-string">"comp-b"</span>)
  <span class="hljs-keyword">return</span> (_openBlock(), _createBlock(_KeepAlive, <span class="hljs-keyword">null</span>, [
    (_ctx.flag)
      ? _createVNode(_component_comp_a, { key: <span class="hljs-number">0</span> })
      : _createVNode(_component_comp_b, { key: <span class="hljs-number">1</span> }),
    _createVNode(<span class="hljs-string">"button"</span>, {
      onClick: $event =&gt; (_ctx.flag=!_ctx.flag)
    }, <span class="hljs-string">"toggle"</span>, <span class="hljs-number">8</span> <span class="hljs-comment">/* PROPS */</span>, [<span class="hljs-string">"onClick"</span>])
  ], <span class="hljs-number">1024</span> <span class="hljs-comment">/* DYNAMIC_SLOTS */</span>))
}
</code></pre>
<p data-nodeid="29951">我们使用了 KeepAlive 组件对这两个组件做了一层封装，KeepAlive 是一个抽象组件，它并不会渲染成一个真实的 DOM，只会渲染内部包裹的子节点，并且让内部的子组件在切换的时候，不会走一整套递归卸载和挂载 DOM的流程，从而优化了性能。</p>
<p data-nodeid="29952">那么它具体是怎么做的呢？我们再来看 KeepAlive 组件的定义：</p>
<pre class="lang-java" data-nodeid="29953"><code data-language="java"><span class="hljs-keyword">const</span> KeepAliveImpl = {
  name: `KeepAlive`,
  __isKeepAlive: <span class="hljs-keyword">true</span>,
  inheritRef: <span class="hljs-keyword">true</span>,
  props: {
    include: [String, RegExp, Array],
    exclude: [String, RegExp, Array],
    max: [String, Number]
  },
  setup(props, { slots }) {
    <span class="hljs-keyword">const</span> cache = <span class="hljs-keyword">new</span> Map()
    <span class="hljs-keyword">const</span> keys = <span class="hljs-keyword">new</span> Set()
    let current = <span class="hljs-keyword">null</span>
    <span class="hljs-keyword">const</span> instance = getCurrentInstance()
    <span class="hljs-keyword">const</span> parentSuspense = instance.suspense
    <span class="hljs-keyword">const</span> sharedContext = instance.ctx
    <span class="hljs-keyword">const</span> { renderer: { p: patch, m: move, um: _unmount, o: { createElement } } } = sharedContext
    <span class="hljs-keyword">const</span> storageContainer = createElement(<span class="hljs-string">'div'</span>)
    sharedContext.activate = (vnode, container, anchor, isSVG, optimized) =&gt; {
      <span class="hljs-keyword">const</span> instance = vnode.<span class="hljs-function">component
      <span class="hljs-title">move</span><span class="hljs-params">(vnode, container, anchor, <span class="hljs-number">0</span> <span class="hljs-comment">/* ENTER */</span>, parentSuspense)</span>
      <span class="hljs-title">patch</span><span class="hljs-params">(instance.vnode, vnode, container, anchor, instance, parentSuspense, isSVG, optimized)</span>
      <span class="hljs-title">queuePostRenderEffect</span><span class="hljs-params">(()</span> </span>=&gt; {
        instance.isDeactivated = <span class="hljs-function"><span class="hljs-keyword">false</span>
        <span class="hljs-title">if</span> <span class="hljs-params">(instance.a)</span> </span>{
          invokeArrayFns(instance.a)
        }
        <span class="hljs-keyword">const</span> vnodeHook = vnode.props &amp;&amp; vnode.props.<span class="hljs-function">onVnodeMounted
        <span class="hljs-title">if</span> <span class="hljs-params">(vnodeHook)</span> </span>{
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
      }, parentSuspense)
    }
    sharedContext.deactivate = (vnode) =&gt; {
      <span class="hljs-keyword">const</span> instance = vnode.<span class="hljs-function">component
      <span class="hljs-title">move</span><span class="hljs-params">(vnode, storageContainer, <span class="hljs-keyword">null</span>, <span class="hljs-number">1</span> <span class="hljs-comment">/* LEAVE */</span>, parentSuspense)</span>
      <span class="hljs-title">queuePostRenderEffect</span><span class="hljs-params">(()</span> </span>=&gt; {
        <span class="hljs-keyword">if</span> (instance.da) {
          invokeArrayFns(instance.da)
        }
        <span class="hljs-keyword">const</span> vnodeHook = vnode.props &amp;&amp; vnode.props.<span class="hljs-function">onVnodeUnmounted
        <span class="hljs-title">if</span> <span class="hljs-params">(vnodeHook)</span> </span>{
          invokeVNodeHook(vnodeHook, instance.parent, vnode)
        }
        instance.isDeactivated = <span class="hljs-keyword">true</span>
      }, parentSuspense)
    }
    <span class="hljs-function">function <span class="hljs-title">unmount</span><span class="hljs-params">(vnode)</span> </span>{
      resetShapeFlag(vnode)
      _unmount(vnode, instance, parentSuspense)
    }
    <span class="hljs-function">function <span class="hljs-title">pruneCache</span><span class="hljs-params">(filter)</span> </span>{
      cache.forEach((vnode, key) =&gt; {
        <span class="hljs-keyword">const</span> name = getName(vnode.type)
        <span class="hljs-keyword">if</span> (name &amp;&amp; (!filter || !filter(name))) {
          pruneCacheEntry(key)
        }
      })
    }
    <span class="hljs-function">function <span class="hljs-title">pruneCacheEntry</span><span class="hljs-params">(key)</span> </span>{
      <span class="hljs-keyword">const</span> cached = cache.get(key)
      <span class="hljs-keyword">if</span> (!current || cached.type !== current.type) {
        unmount(cached)
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (current) {
        resetShapeFlag(current)
      }
      cache.delete(key)
      keys.delete(key)
    }
    watch(() =&gt; [props.include, props.exclude], ([include, exclude]) =&gt; {
      include &amp;&amp; pruneCache(name =&gt; matches(include, name))
      exclude &amp;&amp; !pruneCache(name =&gt; matches(exclude, name))
    })
    let pendingCacheKey = <span class="hljs-keyword">null</span>
    <span class="hljs-keyword">const</span> cacheSubtree = () =&gt; {
      <span class="hljs-keyword">if</span> (pendingCacheKey != <span class="hljs-keyword">null</span>) {
        cache.set(pendingCacheKey, instance.subTree)
      }
    }
    onBeforeMount(cacheSubtree)
    onBeforeUpdate(cacheSubtree)
    onBeforeUnmount(() =&gt; {
      cache.forEach(cached =&gt; {
        <span class="hljs-keyword">const</span> { subTree, suspense } = <span class="hljs-function">instance
        <span class="hljs-title">if</span> <span class="hljs-params">(cached.type === subTree.type)</span> </span>{
          resetShapeFlag(subTree)
          <span class="hljs-keyword">const</span> da = subTree.component.da
          da &amp;&amp; queuePostRenderEffect(da, suspense)
          <span class="hljs-keyword">return</span>
        }
        unmount(cached)
      })
    })
    <span class="hljs-keyword">return</span> () =&gt; {
      pendingCacheKey = <span class="hljs-function"><span class="hljs-keyword">null</span>
      <span class="hljs-title">if</span> <span class="hljs-params">(!slots.<span class="hljs-keyword">default</span>)</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>
      }
      <span class="hljs-keyword">const</span> children = slots.<span class="hljs-keyword">default</span>()
      let vnode = children[<span class="hljs-number">0</span>]
      <span class="hljs-keyword">if</span> (children.length &gt; <span class="hljs-number">1</span>) {
        <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
          warn(`KeepAlive should contain exactly one component child.`)
        }
        current = <span class="hljs-keyword">null</span>
        <span class="hljs-keyword">return</span> children
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!isVNode(vnode) ||
        !(vnode.shapeFlag &amp; <span class="hljs-number">4</span> <span class="hljs-comment">/* STATEFUL_COMPONENT */</span>)) {
        current = <span class="hljs-keyword">null</span>
        <span class="hljs-keyword">return</span> vnode
      }
      <span class="hljs-keyword">const</span> comp = vnode.type
      <span class="hljs-keyword">const</span> name = getName(comp)
      <span class="hljs-keyword">const</span> { include, exclude, max } = <span class="hljs-function">props
      <span class="hljs-title">if</span> <span class="hljs-params">((include &amp;&amp; (!name || !matches(include, name)</span>)) ||
        <span class="hljs-params">(exclude &amp;&amp; name &amp;&amp; matches(exclude, name)</span>)) </span>{
        <span class="hljs-keyword">return</span> (current = vnode)
      }
      <span class="hljs-keyword">const</span> key = vnode.key == <span class="hljs-keyword">null</span> ? comp : vnode.key
      <span class="hljs-keyword">const</span> cachedVNode = cache.get(key)
      <span class="hljs-keyword">if</span> (vnode.el) {
        vnode = cloneVNode(vnode)
      }
      pendingCacheKey = <span class="hljs-function">key
      <span class="hljs-title">if</span> <span class="hljs-params">(cachedVNode)</span> </span>{
        vnode.el = cachedVNode.el
        vnode.component = cachedVNode.component
        vnode.shapeFlag |= <span class="hljs-number">512</span> <span class="hljs-comment">/* COMPONENT_KEPT_ALIVE */</span>
        keys.delete(key)
        keys.add(key)
      }
      <span class="hljs-keyword">else</span> {
        keys.add(key)
        <span class="hljs-keyword">if</span> (max &amp;&amp; keys.size &gt; parseInt(max, <span class="hljs-number">10</span>)) {
          pruneCacheEntry(keys.values().next().value)
        }
      }
      vnode.shapeFlag |= <span class="hljs-number">256</span> <span class="hljs-comment">/* COMPONENT_SHOULD_KEEP_ALIVE */</span>
      current = vnode
      <span class="hljs-keyword">return</span> vnode
    }
  }
}
</code></pre>
<p data-nodeid="29954">我把 KeepAlive 的实现拆成四个部分：<strong data-nodeid="30044">组件的渲染</strong>、<strong data-nodeid="30045">缓存的设计</strong>、<strong data-nodeid="30046">Props 设计</strong>和<strong data-nodeid="30047">组件的卸载</strong>。接下来，我们就来依次分析它们的实现。分析的过程中，我会结合前面的示例讲解，希望你也能够运行这个示例，并加入一些断点调试。</p>
<h3 data-nodeid="29955">组件的渲染</h3>
<p data-nodeid="29956">首先，我们来看组件的渲染部分，可以看到 KeepAlive 组件使用了 Composition API 的方式去实现，我们已经学习过了，当 setup 函数返回的是一个函数，那么这个函数就是组件的渲染函数，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="29957"><code data-language="java"><span class="hljs-keyword">return</span> () =&gt; {
  pendingCacheKey = <span class="hljs-function"><span class="hljs-keyword">null</span>
  <span class="hljs-title">if</span> <span class="hljs-params">(!slots.<span class="hljs-keyword">default</span>)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>
  }
  <span class="hljs-keyword">const</span> children = slots.<span class="hljs-keyword">default</span>()
  let vnode = children[<span class="hljs-number">0</span>]
  <span class="hljs-keyword">if</span> (children.length &gt; <span class="hljs-number">1</span>) {
    <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
      warn(`KeepAlive should contain exactly one component child.`)
    }
    current = <span class="hljs-keyword">null</span>
    <span class="hljs-keyword">return</span> children
  }
  <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!isVNode(vnode) ||
    !(vnode.shapeFlag &amp; <span class="hljs-number">4</span> <span class="hljs-comment">/* STATEFUL_COMPONENT */</span>)) {
    current = <span class="hljs-keyword">null</span>
    <span class="hljs-keyword">return</span> vnode
  }
  <span class="hljs-keyword">const</span> comp = vnode.type
  <span class="hljs-keyword">const</span> name = getName(comp)
  <span class="hljs-keyword">const</span> { include, exclude, max } = <span class="hljs-function">props
  <span class="hljs-title">if</span> <span class="hljs-params">((include &amp;&amp; (!name || !matches(include, name)</span>)) ||
    <span class="hljs-params">(exclude &amp;&amp; name &amp;&amp; matches(exclude, name)</span>)) </span>{
    <span class="hljs-keyword">return</span> (current = vnode)
  }
  <span class="hljs-keyword">const</span> key = vnode.key == <span class="hljs-keyword">null</span> ? comp : vnode.key
  <span class="hljs-keyword">const</span> cachedVNode = cache.get(key)
  <span class="hljs-keyword">if</span> (vnode.el) {
    vnode = cloneVNode(vnode)
  }
  pendingCacheKey = <span class="hljs-function">key
  <span class="hljs-title">if</span> <span class="hljs-params">(cachedVNode)</span> </span>{
    vnode.el = cachedVNode.el
    vnode.component = cachedVNode.component
    <span class="hljs-comment">// 避免 vnode 节点作为新节点被挂载</span>
    vnode.shapeFlag |= <span class="hljs-number">512</span> <span class="hljs-comment">/* COMPONENT_KEPT_ALIVE */</span>
    <span class="hljs-comment">// 让这个 key 始终新鲜</span>
    keys.delete(key)
    keys.add(key)
  }
  <span class="hljs-keyword">else</span> {
    keys.add(key)
    <span class="hljs-comment">// 删除最久不用的 key，符合 LRU 思想</span>
    <span class="hljs-keyword">if</span> (max &amp;&amp; keys.size &gt; parseInt(max, <span class="hljs-number">10</span>)) {
      pruneCacheEntry(keys.values().next().value)
    }
  }
  <span class="hljs-comment">// 避免 vnode 被卸载</span>
  vnode.shapeFlag |= <span class="hljs-number">256</span> <span class="hljs-comment">/* COMPONENT_SHOULD_KEEP_ALIVE */</span>
  current = vnode
  <span class="hljs-keyword">return</span> vnode
}
</code></pre>
<p data-nodeid="29958">函数先从 slots.default() 拿到子节点 children，它就是 KeepAlive 组件包裹的子组件，由于 KeepAlive 只能渲染单个子节点，所以当 children 长度大于 1 的时候会报警告。</p>
<p data-nodeid="29959">我们先不考虑缓存部分，KeepAlive 渲染的 vnode 就是子节点 children 的第一个元素，它是函数的返回值。</p>
<p data-nodeid="29960">因此我们说 KeepAlive 是抽象组件，它本身不渲染成实体节点，而是渲染它的第一个子节点。</p>
<p data-nodeid="29961">当然，没有缓存的 KeepAlive 组件是没有灵魂的，这种抽象的封装也是没有任何意义的，所以接下来我们重点来看它的缓存是如何设计的。</p>
<h3 data-nodeid="29962">缓存的设计</h3>
<p data-nodeid="29963">我们先来思考一件事情，我们需要缓存什么？</p>
<p data-nodeid="29964">组件的递归 patch 过程，主要就是为了渲染 DOM，显然这个递归过程是有一定的性能耗时的，既然目标是为了渲染 DOM，那么我们是不是可以把 DOM 缓存了，这样下一次渲染我们就可以直接从缓存里获取 DOM 并渲染，就不需要每次都重新递归渲染了。</p>
<p data-nodeid="29965">实际上 KeepAlive 组件就是这么做的，它注入了两个钩子函数，onBeforeMount 和 onBeforeUpdate，在这两个钩子函数内部都执行了 cacheSubtree 函数来做缓存：</p>
<pre class="lang-java" data-nodeid="29966"><code data-language="java"><span class="hljs-keyword">const</span> cacheSubtree = () =&gt; {
  <span class="hljs-keyword">if</span> (pendingCacheKey != <span class="hljs-keyword">null</span>) {
    cache.set(pendingCacheKey, instance.subTree)
  }
}
</code></pre>
<p data-nodeid="29967">由于 pendingCacheKey 是在 KeepAlive 的 render 函数中才会被赋值，所以 KeepAlive 首次进入 onBeforeMount 钩子函数的时候是不会缓存的。</p>
<p data-nodeid="29968">然后 KeepAlive 执行 render 的时候，pendingCacheKey 会被赋值为 vnode.key，我们回过头看一下示例渲染后的模板：</p>
<pre class="lang-java" data-nodeid="29969"><code data-language="java"><span class="hljs-keyword">import</span> { resolveComponent as _resolveComponent, createVNode as _createVNode, createCommentVNode as _createCommentVNode, KeepAlive as _KeepAlive, openBlock as _openBlock, createBlock as _createBlock } from <span class="hljs-string">"vue"</span>
<span class="hljs-function">export function <span class="hljs-title">render</span><span class="hljs-params">(_ctx, _cache, $props, $setup, $data, $options)</span> </span>{
  <span class="hljs-keyword">const</span> _component_comp_a = _resolveComponent(<span class="hljs-string">"comp-a"</span>)
  <span class="hljs-keyword">const</span> _component_comp_b = _resolveComponent(<span class="hljs-string">"comp-b"</span>)
  <span class="hljs-keyword">return</span> (_openBlock(), _createBlock(_KeepAlive, <span class="hljs-keyword">null</span>, [
    (_ctx.flag)
      ? _createVNode(_component_comp_a, { key: <span class="hljs-number">0</span> })
      : _createVNode(_component_comp_b, { key: <span class="hljs-number">1</span> }),
    _createVNode(<span class="hljs-string">"button"</span>, {
      onClick: $event =&gt; (_ctx.flag=!_ctx.flag)
    }, <span class="hljs-string">"toggle"</span>, <span class="hljs-number">8</span> <span class="hljs-comment">/* PROPS */</span>, [<span class="hljs-string">"onClick"</span>])
  ], <span class="hljs-number">1024</span> <span class="hljs-comment">/* DYNAMIC_SLOTS */</span>))
}
</code></pre>
<p data-nodeid="29970">我们注意到 KeepAlive 的子节点创建的时候都添加了一个 key 的 prop，它就是专门为 KeepAlive 的缓存设计的，这样每一个子节点都能有一个唯一的 key。</p>
<p data-nodeid="29971">页面首先渲染 A 组件，接着当我们点击按钮的时候，修改了 flag 的值，会触发当前组件的重新渲染，进而也触发了 KeepAlvie 组件的重新渲染，在组件重新渲染前，会执行 onBeforeUpdate 对应的钩子函数，也就再次执行到 cacheSubtree 函数中。</p>
<p data-nodeid="29972">这个时候 pendingCacheKey 对应的是 A 组件 vnode 的 key，instance.subTree 对应的也是 A 组件的渲染子树，所以 KeepAlive 每次在更新前，会缓存前一个组件的渲染子树。</p>
<blockquote data-nodeid="29973">
<p data-nodeid="29974">经过前面的分析，我认为 onBeforeMount 的钩子函数注入似乎并没有必要，我在源码中删除后再跑 Vue.js 3.0 的单测也能通过，如果你有不同意见，欢迎在留言区与我分享。</p>
</blockquote>
<p data-nodeid="29975">这个时候渲染了 B 组件，当我们再次点击按钮，修改 flag 值的时候，会再次触发KeepAlvie 组件的重新渲染，当然此时执行 onBeforeUpdate 钩子函数缓存的就是 B 组件的渲染子树了。</p>
<p data-nodeid="29976">接着再次执行 KeepAlive 组件的 render 函数，此时就可以从缓存中根据 A 组件的 key 拿到对应的渲染子树 cachedVNode 的了，然后执行如下逻辑：</p>
<pre class="lang-java" data-nodeid="29977"><code data-language="java"><span class="hljs-keyword">if</span> (cachedVNode) {
  vnode.el = cachedVNode.el
  vnode.component = cachedVNode.component
  <span class="hljs-comment">// 避免 vnode 节点作为新节点被挂载</span>
  vnode.shapeFlag |= <span class="hljs-number">512</span> <span class="hljs-comment">/* COMPONENT_KEPT_ALIVE */</span>
  <span class="hljs-comment">// 让这个 key 始终新鲜</span>
  keys.delete(key)
  keys.add(key)
}
<span class="hljs-keyword">else</span> {
  keys.add(key)
  <span class="hljs-comment">// 删除最久不用的 key，符合 LRU 思想</span>
  <span class="hljs-keyword">if</span> (max &amp;&amp; keys.size &gt; parseInt(max, <span class="hljs-number">10</span>)) {
    pruneCacheEntry(keys.values().next().value)
  }
}
</code></pre>
<p data-nodeid="29978">有了缓存的渲染子树后，我们就可以直接拿到它对应的 DOM 以及组件实例 component，赋值给 KeepAlive 的 vnode，并更新 vnode.shapeFlag，以便后续 patch 阶段使用。</p>
<blockquote data-nodeid="29979">
<p data-nodeid="29980">注意，这里有一个额外的缓存管理的逻辑，我们稍后讲 Props 设计的时候会详细说。</p>
</blockquote>
<p data-nodeid="29981">那么，对于 KeepAlive 组件的渲染来说，有缓存和没缓存在 patch 阶段有何区别呢，由于 KeepAlive 缓存的都是有状态的组件 vnode，我们再来回顾一下 patchComponent 函数的实现：</p>
<pre class="lang-java" data-nodeid="29982"><code data-language="java"><span class="hljs-keyword">const</span> processComponent = (n1, n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized) =&gt; {
  <span class="hljs-keyword">if</span> (n1 == <span class="hljs-keyword">null</span>) {
    <span class="hljs-comment">// 处理 KeepAlive 组件</span>
    <span class="hljs-keyword">if</span> (n2.shapeFlag &amp; <span class="hljs-number">512</span> <span class="hljs-comment">/* COMPONENT_KEPT_ALIVE */</span>) {
      parentComponent.ctx.activate(n2, container, anchor, isSVG, optimized)
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 挂载组件</span>
      mountComponent(n2, container, anchor, parentComponent, parentSuspense, isSVG, optimized)
    }
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 更新组件</span>
  }
}
</code></pre>
<p data-nodeid="29983">KeepAlive 首次渲染某一个子节点时，和正常的组件节点渲染没有区别，但是有缓存后，由于标记了 shapeFlag，所以在执行processComponent函数时会走到处理 KeepAlive 组件的逻辑中，执行 KeepAlive 组件实例上下文中的 activate 函数，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="29984"><code data-language="java">sharedContext.activate = (vnode, container, anchor, isSVG, optimized) =&gt; {
  <span class="hljs-keyword">const</span> instance = vnode.<span class="hljs-function">component
  <span class="hljs-title">move</span><span class="hljs-params">(vnode, container, anchor, <span class="hljs-number">0</span> <span class="hljs-comment">/* ENTER */</span>, parentSuspense)</span>
  <span class="hljs-title">patch</span><span class="hljs-params">(instance.vnode, vnode, container, anchor, instance, parentSuspense, isSVG, optimized)</span>
  <span class="hljs-title">queuePostRenderEffect</span><span class="hljs-params">(()</span> </span>=&gt; {
    instance.isDeactivated = <span class="hljs-function"><span class="hljs-keyword">false</span>
    <span class="hljs-title">if</span> <span class="hljs-params">(instance.a)</span> </span>{
      invokeArrayFns(instance.a)
    }
    <span class="hljs-keyword">const</span> vnodeHook = vnode.props &amp;&amp; vnode.props.<span class="hljs-function">onVnodeMounted
    <span class="hljs-title">if</span> <span class="hljs-params">(vnodeHook)</span> </span>{
      invokeVNodeHook(vnodeHook, instance.parent, vnode)
    }
  }, parentSuspense)
}
</code></pre>
<p data-nodeid="29985">可以看到，由于此时已经能从 vnode.el 中拿到缓存的 DOM 了，所以可以直接调用 move 方法挂载节点，然后执行 patch 方法更新组件，以防止 props 发生变化的情况。</p>
<p data-nodeid="29986">接下来，就是通过 queuePostRenderEffect 的方式，在组件渲染完毕后，执行子节点组件定义的 activated 钩子函数。</p>
<p data-nodeid="29987">至此，我们就了解了 KeepAlive 的缓存设计，KeepAlive 包裹的子组件在其渲染后，下一次 KeepAlive 组件更新前会被缓存，缓存后的子组件在下一次渲染的时候直接从缓存中拿到子树 vnode 以及对应的 DOM 元素，直接渲染即可。</p>
<p data-nodeid="29988">当然，光有缓存还不够灵活，有些时候我们想针对某些子组件缓存，某些子组件不缓存，另外，我们还想限制 KeepAlive 组件的最大缓存个数，怎么办呢？KeepAlive 设计了几个 Props，允许我们可以对上述需求做配置。</p>
<h3 data-nodeid="29989">Props 设计</h3>
<p data-nodeid="29990">KeepAlive 一共支持了三个 Props，分别是 include、exclude 和 max。</p>
<pre class="lang-js" data-nodeid="30437"><code data-language="js">props: {
  <span class="hljs-attr">include</span>: [<span class="hljs-built_in">String</span>, <span class="hljs-built_in">RegExp</span>, <span class="hljs-built_in">Array</span>],
  <span class="hljs-attr">exclude</span>: [<span class="hljs-built_in">String</span>, <span class="hljs-built_in">RegExp</span>, <span class="hljs-built_in">Array</span>],
  <span class="hljs-attr">max</span>: [<span class="hljs-built_in">String</span>, <span class="hljs-built_in">Number</span>]
}
</code></pre>
<p data-nodeid="30438">include 和 exclude 对应的实现逻辑如下：</p>
<pre class="lang-java" data-nodeid="30439"><code data-language="java"><span class="hljs-keyword">const</span> { include, exclude, max } = <span class="hljs-function">props
<span class="hljs-title">if</span> <span class="hljs-params">((include &amp;&amp; (!name || !matches(include, name)</span>)) ||
  <span class="hljs-params">(exclude &amp;&amp; name &amp;&amp; matches(exclude, name)</span>)) </span>{
  <span class="hljs-keyword">return</span> (current = vnode)
}
</code></pre>
<p data-nodeid="30440">很好理解，如果子组件名称不匹配 include 的 vnode ，以及子组件名称匹配 exclude 的 vnode 都不应该被缓存，而应该直接返回。</p>
<p data-nodeid="30441">当然，由于 props 是响应式的，在 include 和 exclude props 发生变化的时候也应该有相关的处理逻辑，如下：</p>
<pre class="lang-js" data-nodeid="30831"><code data-language="js">watch(<span class="hljs-function">() =&gt;</span> [props.include, props.exclude], <span class="hljs-function">(<span class="hljs-params">[include, exclude]</span>) =&gt;</span> {
  include &amp;&amp; pruneCache(<span class="hljs-function"><span class="hljs-params">name</span> =&gt;</span> matches(include, name))
  exclude &amp;&amp; !pruneCache(<span class="hljs-function"><span class="hljs-params">name</span> =&gt;</span> matches(exclude, name))
})
</code></pre>
<p data-nodeid="30832">监听的逻辑也很简单，当 include 发生变化的时候，从缓存中删除那些 name 不匹配 include 的 vnode 节点；当 exclude 发生变化的时候，从缓存中删除那些 name 匹配 exclude 的 vnode 节点。</p>
<p data-nodeid="30833">除了 include 和 exclude 之外，KeepAlive 组件还支持了 max prop 来控制缓存的最大个数。</p>
<p data-nodeid="30834">由于缓存本身就是占用了内存，所以有些场景我们希望限制 KeepAlive 缓存的个数，这时我们可以通过 max 属性来控制，当缓存新的 vnode 的时候，会做一定程度的缓存管理，如下：</p>
<pre class="lang-java" data-nodeid="30835"><code data-language="java">keys.add(key)
<span class="hljs-comment">// 删除最久不用的 key，符合 LRU 思想</span>
<span class="hljs-keyword">if</span> (max &amp;&amp; keys.size &gt; parseInt(max, <span class="hljs-number">10</span>)) {
pruneCacheEntry(keys.values().next().value)
}
</code></pre>
<p data-nodeid="30836">由于新的缓存 key 都是在 keys 的结尾添加的，所以当缓存的个数超过 max 的时候，就从最前面开始删除，符合 LRU 最近最少使用的算法思想。</p>
<h3 data-nodeid="30837">组件的卸载</h3>
<p data-nodeid="30838">了解完 KeepAlive 组件的渲染、缓存和 Props 设计后，我们接着来看 KeepAlive 组件的卸载过程。</p>
<p data-nodeid="30839">我们先来分析 KeepAlive 内部包裹的子组件的卸载过程，前面我们提到 KeepAlive 渲染的过程实际上是渲染它的第一个子组件节点，并且会给渲染的 vnode 打上如下标记：</p>
<pre class="lang-java" data-nodeid="30840"><code data-language="java">vnode.shapeFlag |= <span class="hljs-number">256</span> <span class="hljs-comment">/* COMPONENT_SHOULD_KEEP_ALIVE */</span>
</code></pre>
<p data-nodeid="30841">加上这个 shapeFlag 有什么用呢，我们结合前面的示例来分析。</p>
<pre class="lang-js" data-nodeid="31217"><code data-language="js">&lt;keep-alive&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">comp-a</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"flag"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">comp-a</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">comp-b</span> <span class="hljs-attr">v-else</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">comp-b</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"flag=!flag"</span>&gt;</span>toggle<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
&lt;/keep-alive&gt;
</code></pre>
<p data-nodeid="31584">当 flag 为 true 的时候，渲染 A 组件，然后我们点击按钮修改 flag 的值，会触发 KeepAlive 组件的重新渲染，会先执行 BeforeUpdate 钩子函数缓存 A 组件对应的渲染子树 vnode，然后再执行 patch 更新子组件。</p>
<p data-nodeid="31585">这个时候会执行 B 组件的渲染，以及 A 组件的卸载，我们知道组件的卸载会执行 unmount 方法，其中有一个关于 KeepAlive 组件的逻辑，如下：</p>

<pre class="lang-java" data-nodeid="31219"><code data-language="java"><span class="hljs-keyword">const</span> unmount = (vnode, parentComponent, parentSuspense, doRemove = <span class="hljs-keyword">false</span>) =&gt; {
  <span class="hljs-keyword">const</span> { shapeFlag  } = <span class="hljs-function">vnode
  <span class="hljs-title">if</span> <span class="hljs-params">(shapeFlag &amp; <span class="hljs-number">256</span> <span class="hljs-comment">/* COMPONENT_SHOULD_KEEP_ALIVE */</span>)</span> </span>{
    parentComponent.ctx.deactivate(vnode)
    <span class="hljs-keyword">return</span>
  }
  <span class="hljs-comment">// 卸载组件</span>
}
</code></pre>
<p data-nodeid="31220">如果 shapeFlag 满足 KeepAlive 的条件，则执行相应的 deactivate 函数，它的定义如下：</p>
<pre class="lang-java" data-nodeid="31221"><code data-language="java">sharedContext.deactivate = (vnode) =&gt; {
  <span class="hljs-keyword">const</span> instance = vnode.<span class="hljs-function">component
  <span class="hljs-title">move</span><span class="hljs-params">(vnode, storageContainer, <span class="hljs-keyword">null</span>, <span class="hljs-number">1</span> <span class="hljs-comment">/* LEAVE */</span>, parentSuspense)</span>
  <span class="hljs-title">queuePostRenderEffect</span><span class="hljs-params">(()</span> </span>=&gt; {
    <span class="hljs-keyword">if</span> (instance.da) {
      invokeArrayFns(instance.da)
    }
    <span class="hljs-keyword">const</span> vnodeHook = vnode.props &amp;&amp; vnode.props.<span class="hljs-function">onVnodeUnmounted
    <span class="hljs-title">if</span> <span class="hljs-params">(vnodeHook)</span> </span>{
      invokeVNodeHook(vnodeHook, instance.parent, vnode)
    }
    instance.isDeactivated = <span class="hljs-keyword">true</span>
  }, parentSuspense)
}
</code></pre>
<p data-nodeid="31222">函数首先通过 move 方法从 DOM 树中移除该节点，接着通过 queuePostRenderEffect 的方式执行定义的 deactivated 钩子函数。</p>
<p data-nodeid="31223">注意，这里我们只是移除了 DOM，并没有真正意义上的执行子组件的整套卸载流程。</p>
<p data-nodeid="31224">那么除了点击按钮引起子组件的卸载之外，当 KeepAlive 所在的组件卸载时，由于卸载的递归特性，也会触发 KeepAlive 组件的卸载，在卸载的过程中会执行 onBeforeUnmount 钩子函数，如下：</p>
<pre class="lang-java" data-nodeid="31225"><code data-language="java">onBeforeUnmount(() =&gt; {
  cache.forEach(cached =&gt; {
    <span class="hljs-keyword">const</span> { subTree, suspense } = <span class="hljs-function">instance
    <span class="hljs-title">if</span> <span class="hljs-params">(cached.type === subTree.type)</span> </span>{
      resetShapeFlag(subTree)
      <span class="hljs-keyword">const</span> da = subTree.component.da
      da &amp;&amp; queuePostRenderEffect(da, suspense)
      <span class="hljs-keyword">return</span>
    }
    unmount(cached)
  })
})  
</code></pre>
<p data-nodeid="31226">它会遍历所有缓存的 vnode，并且比对缓存的 vnode 是不是当前 KeepAlive 组件渲染的 vnode。</p>
<p data-nodeid="31227">如果是的话，则执行 resetShapeFlag 方法，它的作用是修改 vnode 的 shapeFlag，不让它再被当作一个 KeepAlive 的 vnode 了，这样就可以走正常的卸载逻辑。接着通过 queuePostRenderEffect 的方式执行子组件的 deactivated 钩子函数。</p>
<p data-nodeid="31228">如果不是，则执行 unmount 方法重置 shapeFlag 以及执行缓存 vnode 的整套卸载流程。</p>
<h3 data-nodeid="31229">总结</h3>
<p data-nodeid="31230">好的，到这里我们这一节的学习也要结束啦，通过这节课的学习，你应该明白 KeepAlive 实际上是一个抽象节点，渲染的是它的第一个子节点，并了解它的缓存设计、Props 设计和卸载过程。</p>
<p data-nodeid="31231">最后，给你留一道思考题，我们是如何给组件注册 activated 和 deactivated 钩子函数的，它们的执行和其他钩子函数比，有什么不同？欢迎你在留言区与我分享。</p>
<blockquote data-nodeid="31232">
<p data-nodeid="31233">本节课的相关代码在源代码中的位置如下：<br>
packages/runtime-core/src/components/KeepAlive.ts<br>
packages/runtime-core/src/renderer.ts</p>
</blockquote>

---

### 精选评论


