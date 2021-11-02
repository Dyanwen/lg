<p data-nodeid="38113" class="">上节课，我们已经知道了，Vue.js 提供了内置的 Transition 组件帮我们实现动画过渡效果。在之前的分析中我把 Transition 组件的实现分成了三个部分：组件的渲染、钩子函数的执行、模式的应用。这节课我们从钩子函数的执行继续探究 Transition 组件的实现原理。</p>




<h3 data-nodeid="37387">钩子函数的执行</h3>
<p data-nodeid="37388">这个部分我们先来看 beforeEnter 钩子函数。</p>
<p data-nodeid="37389">在 patch 阶段的 mountElement 函数中，在插入元素节点前且存在过渡的条件下会执行 vnode.transition 中的 beforeEnter 函数，我们来看它的定义：</p>
<pre class="lang-java" data-nodeid="37390"><code data-language="java">beforeEnter(el) {
  let hook = <span class="hljs-function">onBeforeEnter
  <span class="hljs-title">if</span> <span class="hljs-params">(!state.isMounted)</span> </span>{
    <span class="hljs-keyword">if</span> (appear) {
      hook = onBeforeAppear || onBeforeEnter
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span>
    }
  }
  <span class="hljs-keyword">if</span> (el._leaveCb) {
    el._leaveCb(<span class="hljs-keyword">true</span> <span class="hljs-comment">/* cancelled */</span>)
  }
  <span class="hljs-keyword">const</span> leavingVNode = leavingVNodesCache[key]
  <span class="hljs-keyword">if</span> (leavingVNode &amp;&amp;
    isSameVNodeType(vnode, leavingVNode) &amp;&amp;
    leavingVNode.el._leaveCb) {
    leavingVNode.el._leaveCb()
  }
  callHook(hook, [el])
}
</code></pre>
<p data-nodeid="37391">beforeEnter 钩子函数主要做的事情就是根据 appear 的值和 DOM 是否挂载，来执行 onBeforeEnter 函数或者是 onBeforeAppear 函数，其他的逻辑我们暂时先不看。</p>
<p data-nodeid="37392">appear、onBeforeEnter、onBeforeAppear 这些变量都是从 props 中获取的，那么这些 props 是怎么初始化的呢？回到 Transition 组件的定义：</p>
<pre class="lang-java" data-nodeid="37393"><code data-language="java"><span class="hljs-keyword">const</span> Transition = (props, { slots }) =&gt; h(BaseTransition, resolveTransitionProps(props), slots)
</code></pre>
<p data-nodeid="37394">可以看到，传递的 props 经过了 resolveTransitionProps 函数的封装，我们来看它的定义：</p>
<pre class="lang-java" data-nodeid="37395"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">resolveTransitionProps</span><span class="hljs-params">(rawProps)</span> </span>{
  let { name = <span class="hljs-string">'v'</span>, type, css = <span class="hljs-keyword">true</span>, duration, enterFromClass = `${name}-enter-from`, enterActiveClass = `${name}-enter-active`, enterToClass = `${name}-enter-to`, appearFromClass = enterFromClass, appearActiveClass = enterActiveClass, appearToClass = enterToClass, leaveFromClass = `${name}-leave-from`, leaveActiveClass = `${name}-leave-active`, leaveToClass = `${name}-leave-to` } = rawProps
  <span class="hljs-keyword">const</span> baseProps = {}
  <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key in rawProps) {
    <span class="hljs-keyword">if</span> (!(key in DOMTransitionPropsValidators)) {
      baseProps[key] = rawProps[key]
    }
  }
  <span class="hljs-keyword">if</span> (!css) {
    <span class="hljs-keyword">return</span> baseProps
  }
  <span class="hljs-keyword">const</span> durations = normalizeDuration(duration)
  <span class="hljs-keyword">const</span> enterDuration = durations &amp;&amp; durations[<span class="hljs-number">0</span>]
  <span class="hljs-keyword">const</span> leaveDuration = durations &amp;&amp; durations[<span class="hljs-number">1</span>]
  <span class="hljs-keyword">const</span> { onBeforeEnter, onEnter, onEnterCancelled, onLeave, onLeaveCancelled, onBeforeAppear = onBeforeEnter, onAppear = onEnter, onAppearCancelled = onEnterCancelled } = baseProps
  <span class="hljs-keyword">const</span> finishEnter = (el, isAppear, done) =&gt; {
    removeTransitionClass(el, isAppear ? appearToClass : enterToClass)
    removeTransitionClass(el, isAppear ? appearActiveClass : enterActiveClass)
    done &amp;&amp; done()
  }
  <span class="hljs-keyword">const</span> finishLeave = (el, done) =&gt; {
    removeTransitionClass(el, leaveToClass)
    removeTransitionClass(el, leaveActiveClass)
    done &amp;&amp; done()
  }
  <span class="hljs-keyword">const</span> makeEnterHook = (isAppear) =&gt; {
    <span class="hljs-keyword">return</span> (el, done) =&gt; {
      <span class="hljs-keyword">const</span> hook = isAppear ? onAppear : onEnter
      <span class="hljs-keyword">const</span> resolve = () =&gt; finishEnter(el, isAppear, done)
      hook &amp;&amp; hook(el, resolve)
      nextFrame(() =&gt; {
        removeTransitionClass(el, isAppear ? appearFromClass : enterFromClass)
        addTransitionClass(el, isAppear ? appearToClass : enterToClass)
        <span class="hljs-keyword">if</span> (!(hook &amp;&amp; hook.length &gt; <span class="hljs-number">1</span>)) {
          <span class="hljs-keyword">if</span> (enterDuration) {
            setTimeout(resolve, enterDuration)
          }
          <span class="hljs-keyword">else</span> {
            whenTransitionEnds(el, type, resolve)
          }
        }
      })
    }
  }
  <span class="hljs-keyword">return</span> extend(baseProps, {
    onBeforeEnter(el) {
      onBeforeEnter &amp;&amp; onBeforeEnter(el)
      addTransitionClass(el, enterActiveClass)
      addTransitionClass(el, enterFromClass)
    },
    onBeforeAppear(el) {
      onBeforeAppear &amp;&amp; onBeforeAppear(el)
      addTransitionClass(el, appearActiveClass)
      addTransitionClass(el, appearFromClass)
    },
    onEnter: makeEnterHook(<span class="hljs-keyword">false</span>),
    onAppear: makeEnterHook(<span class="hljs-keyword">true</span>),
    onLeave(el, done) {
      <span class="hljs-keyword">const</span> resolve = () =&gt; finishLeave(el, done)
      addTransitionClass(el, leaveActiveClass)
      addTransitionClass(el, leaveFromClass)
      nextFrame(() =&gt; {
        removeTransitionClass(el, leaveFromClass)
        addTransitionClass(el, leaveToClass)
        <span class="hljs-keyword">if</span> (!(onLeave &amp;&amp; onLeave.length &gt; <span class="hljs-number">1</span>)) {
          <span class="hljs-keyword">if</span> (leaveDuration) {
            setTimeout(resolve, leaveDuration)
          }
          <span class="hljs-keyword">else</span> {
            whenTransitionEnds(el, type, resolve)
          }
        }
      })
      onLeave &amp;&amp; onLeave(el, resolve)
    },
    onEnterCancelled(el) {
      finishEnter(el, <span class="hljs-keyword">false</span>)
      onEnterCancelled &amp;&amp; onEnterCancelled(el)
    },
    onAppearCancelled(el) {
      finishEnter(el, <span class="hljs-keyword">true</span>)
      onAppearCancelled &amp;&amp; onAppearCancelled(el)
    },
    onLeaveCancelled(el) {
      finishLeave(el)
      onLeaveCancelled &amp;&amp; onLeaveCancelled(el)
    }
  })
}
</code></pre>
<p data-nodeid="37396">resolveTransitionProps 函数主要作用是，在我们给 Transition 传递的 Props 基础上做一层封装，然后返回一个新的 Props 对象，由于它包含了所有的 Props 处理，你不需要一下子了解所有的实现，按需分析即可。</p>
<p data-nodeid="37397">我们来看 onBeforeEnter 函数，它的内部执行了基础 props 传入的 onBeforeEnter 钩子函数，并且给 DOM 元素 el 添加了 enterActiveClass 和 enterFromClass 样式。</p>
<p data-nodeid="37398">其中，props 传入的 onBeforeEnter 函数就是我们写 Transition 组件时添加的 beforeEnter 钩子函数。enterActiveClass 默认值是 v-enter-active，enterFromClass 默认值是 v-enter-from，如果给 Transition 组件传入了 name 的 prop，比如 fade，那么 enterActiveClass 的值就是 fade-enter-active，enterFromClass 的值就是 fade-enter-from。</p>
<p data-nodeid="37399">原来这就是在 DOM 元素对象在创建后，插入到页面前做的事情：<strong data-nodeid="37477">执行 beforeEnter 钩子函数</strong>，<strong data-nodeid="37478">以及给元素添加相应的 CSS 样式</strong>。</p>
<p data-nodeid="37400">onBeforeAppear 和 onBeforeEnter 的逻辑类似，就不赘述了，它是在我们给 Transition 组件传入 appear 的 Prop，且首次挂载的时候执行的。</p>
<p data-nodeid="37401">执行完 beforeEnter 钩子函数，接着插入元素到页面，然后会执行 vnode.transition 中的enter 钩子函数，我们来看它的定义：</p>
<pre class="lang-java" data-nodeid="37402"><code data-language="java">enter(el) {
  let hook = onEnter
  let afterHook = onAfterEnter
  let cancelHook = <span class="hljs-function">onEnterCancelled
  <span class="hljs-title">if</span> <span class="hljs-params">(!state.isMounted)</span> </span>{
    <span class="hljs-keyword">if</span> (appear) {
      hook = onAppear || onEnter
      afterHook = onAfterAppear || onAfterEnter
      cancelHook = onAppearCancelled || onEnterCancelled
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span>
    }
  }
  let called = <span class="hljs-keyword">false</span>
  <span class="hljs-keyword">const</span> done = (el._enterCb = (cancelled) =&gt; {
    <span class="hljs-keyword">if</span> (called)
      <span class="hljs-keyword">return</span>
    called = <span class="hljs-function"><span class="hljs-keyword">true</span>
    <span class="hljs-title">if</span> <span class="hljs-params">(cancelled)</span> </span>{
      callHook(cancelHook, [el])
    }
    <span class="hljs-keyword">else</span> {
      callHook(afterHook, [el])
    }
    <span class="hljs-keyword">if</span> (hooks.delayedLeave) {
      hooks.delayedLeave()
    }
    el._enterCb = undefined
  })
  <span class="hljs-keyword">if</span> (hook) {
    hook(el, done)
    <span class="hljs-keyword">if</span> (hook.length &lt;= <span class="hljs-number">1</span>) {
      done()
    }
  }
  <span class="hljs-keyword">else</span> {
    done()
  }
}
</code></pre>
<p data-nodeid="37403">enter 钩子函数主要做的事情就是根据 appear 的值和 DOM 是否挂载，执行 onEnter 函数或者是 onAppear 函数，并且这个函数的第二个参数是一个 done 函数，表示过渡动画完成后执行的回调函数，它是异步执行的。</p>
<blockquote data-nodeid="37404">
<p data-nodeid="37405">注意，当 onEnter 或者 onAppear 函数的参数长度小于等于 1 的时候，done 函数在执行完 hook 函数后同步执行。</p>
</blockquote>
<p data-nodeid="37406">在 done 函数的内部，我们会执行 onAfterEnter 函数或者是 onEnterCancelled 函数，其它的逻辑我们也暂时先不看。</p>
<p data-nodeid="37407">同理，onEnter、onAppear、onAfterEnter 和 onEnterCancelled 函数也是从 Props 传入的，我们重点看 onEnter 的实现，它是 makeEnterHook(false) 函数执行后的返回值，如下：</p>
<pre class="lang-java" data-nodeid="37408"><code data-language="java"><span class="hljs-keyword">const</span> makeEnterHook = (isAppear) =&gt; {
  <span class="hljs-keyword">return</span> (el, done) =&gt; {
    <span class="hljs-keyword">const</span> hook = isAppear ? onAppear : onEnter
    <span class="hljs-keyword">const</span> resolve = () =&gt; finishEnter(el, isAppear, done)
    hook &amp;&amp; hook(el, resolve)
    nextFrame(() =&gt; {
      removeTransitionClass(el, isAppear ? appearFromClass : enterFromClass)
      addTransitionClass(el, isAppear ? appearToClass : enterToClass)
      <span class="hljs-keyword">if</span> (!(hook &amp;&amp; hook.length &gt; <span class="hljs-number">1</span>)) {
        <span class="hljs-keyword">if</span> (enterDuration) {
          setTimeout(resolve, enterDuration)
        }
        <span class="hljs-keyword">else</span> {
          whenTransitionEnds(el, type, resolve)
        }
      }
    })
  }
}
</code></pre>
<p data-nodeid="37409">在函数内部，首先执行基础 props 传入的 onEnter 钩子函数，然后在下一帧给 DOM 元素 el 移除了 enterFromClass，同时添加了 enterToClass 样式。</p>
<p data-nodeid="37410">其中，props 传入的 onEnter 函数就是我们写 Transition 组件时添加的 enter 钩子函数，enterFromClass 是我们在 beforeEnter 阶段添加的，会在当前阶段移除，新增的 enterToClass 值默认是 v-enter-to，如果给 Transition 组件传入了 name 的 prop，比如 fade，那么 enterToClass 的值就是 fade-enter-to。</p>
<p data-nodeid="37411">注意，当我们添加了 enterToClass 后，这个时候浏览器就开始根据我们编写的 CSS 进入过渡动画了，那么动画何时结束呢？</p>
<p data-nodeid="37412">Transition 组件允许我们传入 enterDuration 这个 prop，它会指定进入过渡的动画时长，当然如果你不指定，Vue.js 内部会监听动画结束事件，然后在动画结束后，执行 finishEnter 函数，来看它的实现：</p>
<pre class="lang-js" data-nodeid="38401"><code data-language="js"><span class="hljs-keyword">const</span> finishEnter = <span class="hljs-function">(<span class="hljs-params">el, isAppear, done</span>) =&gt;</span> {
  removeTransitionClass(el, isAppear ? appearToClass : enterToClass)
  removeTransitionClass(el, isAppear ? appearActiveClass : enterActiveClass)
  done &amp;&amp; done()
}
</code></pre>
<p data-nodeid="38402">其实就是给 DOM 元素移除 enterToClass 以及 enterActiveClass，同时执行 done 函数，进而执行 onAfterEnter 钩子函数。</p>
<p data-nodeid="38403">至此，元素进入的过渡动画逻辑就分析完了，接下来我们来分析元素离开的过渡动画逻辑。</p>
<p data-nodeid="38404">当元素被删除的时候，会执行 remove 方法，在真正从 DOM 移除元素前且存在过渡的情况下，会执行 vnode.transition 中的 leave 钩子函数，并且把移动 DOM 的方法作为第二个参数传入，我们来看它的定义：</p>
<pre class="lang-java" data-nodeid="38405"><code data-language="java">leave(el, remove) {
  <span class="hljs-keyword">const</span> key = String(vnode.key)
  <span class="hljs-keyword">if</span> (el._enterCb) {
    el._enterCb(<span class="hljs-keyword">true</span> <span class="hljs-comment">/* cancelled */</span>)
  }
  <span class="hljs-keyword">if</span> (state.isUnmounting) {
    <span class="hljs-keyword">return</span> remove()
  }
  callHook(onBeforeLeave, [el])
  let called = <span class="hljs-keyword">false</span>
  <span class="hljs-keyword">const</span> done = (el._leaveCb = (cancelled) =&gt; {
    <span class="hljs-keyword">if</span> (called)
      <span class="hljs-keyword">return</span>
    called = <span class="hljs-function"><span class="hljs-keyword">true</span>
    <span class="hljs-title">remove</span><span class="hljs-params">()</span>
    <span class="hljs-title">if</span> <span class="hljs-params">(cancelled)</span> </span>{
      callHook(onLeaveCancelled, [el])
    }
    <span class="hljs-keyword">else</span> {
      callHook(onAfterLeave, [el])
    }
    el._leaveCb = <span class="hljs-function">undefined
    <span class="hljs-title">if</span> <span class="hljs-params">(leavingVNodesCache[key] === vnode)</span> </span>{
      delete leavingVNodesCache[key]
    }
  })
  leavingVNodesCache[key] = <span class="hljs-function">vnode
  <span class="hljs-title">if</span> <span class="hljs-params">(onLeave)</span> </span>{
    onLeave(el, done)
    <span class="hljs-keyword">if</span> (onLeave.length &lt;= <span class="hljs-number">1</span>) {
      done()
    }
  }
  <span class="hljs-keyword">else</span> {
    done()
  }
}
</code></pre>
<p data-nodeid="38406">leave 钩子函数主要做的事情就是执行 props 传入的 onBeforeLeave 钩子函数和 onLeave 函数，onLeave 函数的第二个参数是一个 done 函数，它表示离开过渡动画结束后执行的回调函数。</p>
<p data-nodeid="38407">done 函数内部主要做的事情就是执行 remove 方法移除 DOM，然后执行 onAfterLeave 钩子函数或者是 onLeaveCancelled 函数，其它的逻辑我们也先不看。</p>
<p data-nodeid="38408">接下来，我们重点看一下 onLeave 函数的实现，看看离开过渡动画是如何执行的。</p>
<pre class="lang-java" data-nodeid="38409"><code data-language="java">onLeave(el, done) {
  <span class="hljs-keyword">const</span> resolve = () =&gt; finishLeave(el, done)
  addTransitionClass(el, leaveActiveClass)
  addTransitionClass(el, leaveFromClass)
  nextFrame(() =&gt; {
    removeTransitionClass(el, leaveFromClass)
    addTransitionClass(el, leaveToClass)
    <span class="hljs-keyword">if</span> (!(onLeave &amp;&amp; onLeave.length &gt; <span class="hljs-number">1</span>)) {
      <span class="hljs-keyword">if</span> (leaveDuration) {
        setTimeout(resolve, leaveDuration)
      }
      <span class="hljs-keyword">else</span> {
        whenTransitionEnds(el, type, resolve)
      }
    }
  })
  onLeave &amp;&amp; onLeave(el, resolve)
}
</code></pre>
<p data-nodeid="38410">onLeave 函数首先给 DOM 元素添加 leaveActiveClass 和 leaveFromClass，并执行基础 props 传入的 onLeave 钩子函数，然后在下一帧移除 leaveFromClass，并添加 leaveToClass。</p>
<p data-nodeid="38411">其中，leaveActiveClass 的默认值是 v-leave-active，leaveFromClass 的默认值是 v-leave-from，leaveToClass 的默认值是 v-leave-to。如果给 Transition 组件传入了 name 的 prop，比如 fade，那么 leaveActiveClass 的值就是 fade-leave-active，leaveFromClass 的值就是 fade-leave-from，leaveToClass 的值就是 fade-leave-to。</p>
<p data-nodeid="38412">注意，当我们添加 leaveToClass 时，浏览器就开始根据我们编写的 CSS 执行离开过渡动画了，那么动画何时结束呢？</p>
<p data-nodeid="38413">和进入动画类似，Transition 组件允许我们传入 leaveDuration 这个 prop，指定过渡的动画时长，当然如果你不指定，Vue.js 内部会监听动画结束事件，然后在动画结束后，执行 resolve 函数，它是执行 finishLeave 函数的返回值，来看它的实现：</p>
<pre class="lang-java" data-nodeid="38414"><code data-language="java"><span class="hljs-keyword">const</span> finishLeave = (el, done) =&gt; {
  removeTransitionClass(el, leaveToClass)
  removeTransitionClass(el, leaveActiveClass)
  done &amp;&amp; done()
}
</code></pre>
<p data-nodeid="38415">其实就是给 DOM 元素移除 leaveToClass 以及 leaveActiveClass，同时执行 done 函数，进而执行 onAfterLeave 钩子函数。</p>
<p data-nodeid="38416">至此，元素离开的过渡动画逻辑就分析完了，可以看出离开过渡动画和进入过渡动画是的思路差不多，本质上都是在添加和移除一些 CSS 去执行动画，并且在过程中执行用户传入的钩子函数。</p>
<h3 data-nodeid="38417">模式的应用</h3>
<p data-nodeid="38418">前面我们在介绍 Transition 的渲染过程中提到过模式的应用，模式有什么用呢，我们还是通过示例说明，把前面的例子稍加修改：</p>
<pre class="lang-js" data-nodeid="38772"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"show = !show"</span>&gt;</span>
      Toggle render
    <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">transition</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"fade"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>hello<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-else</span>&gt;</span>hi<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">transition</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">show</span>: <span class="hljs-literal">true</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">style</span>&gt;</span><span class="css">
  <span class="hljs-selector-class">.fade-enter-active</span>,
  <span class="hljs-selector-class">.fade-leave-active</span> {
    <span class="hljs-attribute">transition</span>: opacity <span class="hljs-number">0.5s</span> ease;
  }
  <span class="hljs-selector-class">.fade-enter-from</span>,
  <span class="hljs-selector-class">.fade-leave-to</span> {
    <span class="hljs-attribute">opacity</span>: <span class="hljs-number">0</span>;
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span></span>
</code></pre>
<p data-nodeid="38773">我们在 show 条件为 false 的情况下，显示字符串 hi，你可以运行这个示例，然后会发现这个过渡效果有点生硬，并不理想。</p>
<p data-nodeid="38774">然后，我们给这个 Transition 组件加一个 out-in 的 mode：</p>
<pre class="lang-js" data-nodeid="39111"><code data-language="js">&lt;transition mode=<span class="hljs-string">"out-in"</span> name=<span class="hljs-string">"fade"</span>&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>hello<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-else</span>&gt;</span>hi<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
&lt;/transition&gt;
</code></pre>
<p data-nodeid="39112">我们会发现这个过渡效果好多了，hello 文本先完成离开的过渡后，hi 文本开始进入过渡动画。</p>
<p data-nodeid="39113">模式非常适合这种两个元素切换的场景，Vue.js 给 Transition 组件提供了两种模式， in-out 和 out-in ，它们有什么区别呢？</p>
<ul data-nodeid="39114">
<li data-nodeid="39115">
<p data-nodeid="39116">在 in-out 模式下，新元素先进行过渡，完成之后当前元素过渡离开。</p>
</li>
<li data-nodeid="39117">
<p data-nodeid="39118">在 out-in 模式下，当前元素先进行过渡，完成之后新元素过渡进入。</p>
</li>
</ul>
<p data-nodeid="39119">在实际工作中，你大部分情况都是在使用 out-in 模式，而 in-out 模式很少用到，所以接下来我们就来分析 out-in 模式的实现原理。</p>
<p data-nodeid="39120">我们先不妨思考一下，为什么在不加模式的情况下，会出现示例那样的过渡效果。</p>
<p data-nodeid="39121">当我们点击按钮，show 变量由 true 变成 false，会触发当前元素 hello 文本的离开动画，也会同时触发新元素 hi 文本的进入动画。由于动画是同时进行的，而且在离开动画结束之前，当前元素 hello 是没有被移除 DOM 的，所以它还会占位，就把新元素 hi 文本挤到下面去了。当 hello 文本的离开动画执行完毕从 DOM 中删除后，hi 文本才能回到之前的位置。</p>
<p data-nodeid="39122">那么，我们怎么做才能做到当前元素过渡动画执行完毕后，再执行新元素的过渡呢？</p>
<p data-nodeid="39123">我们来看一下 out-in 模式的实现：</p>
<pre class="lang-java" data-nodeid="39124"><code data-language="java"><span class="hljs-keyword">const</span> leavingHooks = resolveTransitionHooks(oldInnerChild, rawProps, state, instance)
setTransitionHooks(oldInnerChild, leavingHooks)
<span class="hljs-keyword">if</span> (mode === <span class="hljs-string">'out-in'</span>) {
  state.isLeaving = <span class="hljs-keyword">true</span>
  leavingHooks.afterLeave = () =&gt; {
    state.isLeaving = <span class="hljs-keyword">false</span>
    instance.update()
  }
  <span class="hljs-keyword">return</span> emptyPlaceholder(child)
}
</code></pre>
<p data-nodeid="39125">当模式为 out-in 的时候，会标记 state.isLeaving 为 true，然后返回一个空的注释节点，同时更新当前元素的钩子函数中的 afterLeave 函数，内部执行 instance.update 重新渲染组件。</p>
<p data-nodeid="39126">这样做就保证了在当前元素执行离开过渡的时候，新元素只渲染成一个注释节点，这样页面上看上去还是只执行当前元素的离开过渡动画。</p>
<p data-nodeid="39127">然后当离开动画执行完毕后，触发了 Transition 组件的重新渲染，这个时候就可以如期渲染新元素并执行进入过渡动画了，是不是很巧妙呢？</p>
<h3 data-nodeid="39128">总结</h3>
<p data-nodeid="39129">好的，到这里我们这一节的学习就结束啦，通过这节课的学习，你应该了解了 Transition 组件是如何渲染的，如何执行过渡动画和相应的钩子函数的，以及当两个视图切换时，模式的工作原理是怎样的。</p>
<p data-nodeid="39130">最后，给你留一道思考题，Transition 组件在 beforeEnter 钩子函数里会判断 el._leaveCb 是否存在，存在则执行，在 leave 钩子函数里会判断 el._enterCb 是否存在，存在则执行，这么做的原因是什么？欢迎你在留言区与我分享。</p>
<blockquote data-nodeid="39131">
<p data-nodeid="39132">本节课的相关代码在源代码中的位置如下：<br>
packages/runtime-core/src/components/BasetTransition.ts<br>
packages/runtime-core/src/renderer.ts<br>
packages/runtime-dom/src/components/Transition.ts</p>
</blockquote>

---

### 精选评论


