<p data-nodeid="32454">作为一名前端开发工程师，平时开发页面少不了要写一些过渡动画，通常可以用 CSS 脚本来实现，当然一些时候也会使用 JavaScript 操作 DOM 来实现动画。那么，如果我们使用 Vue.js 技术栈，有没有好的实现动画的方式呢？</p>



<p data-nodeid="31923">答案是肯定的——有，Vue.js 提供了内置的 Transition 组件，它可以让我们轻松实现动画过渡效果。</p>
<h3 data-nodeid="31924">Transition 组件的用法</h3>
<blockquote data-nodeid="33515">
<p data-nodeid="33516" class="">如果你还不太熟悉 Transition 组件的使用，我建议你先去看它的<a href="https://v3.vuejs.org/guide/transitions-enterleave.html" data-nodeid="33520">官网文档</a>。</p>
</blockquote>



<p data-nodeid="31927">Transition 组件通常有三类用法：CSS 过渡，CSS 动画和 JavaScript 钩子。我们分别用几个示例来说明，这里我希望你可以敲代码运行感受一下。</p>
<p data-nodeid="34218">首先来看 CSS 过渡：</p>


<pre class="lang-js" data-nodeid="34393"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"show = !show"</span>&gt;</span>
      Toggle render
    <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">transition</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"fade"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>hello<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
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
<p data-nodeid="34394">CSS 过渡主要定义了一些过渡的 CSS 样式，当我们点击按钮切换文本显隐的时候，就会应用这些 CSS 样式，实现过渡效果。</p>
<p data-nodeid="35237">接着来看 CSS 动画：</p>


<pre class="lang-javascript" data-nodeid="34397"><code data-language="javascript">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"show = !show"</span>&gt;</span>Toggle show<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">transition</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"bounce"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>Vue is an awesome front-end MVVM framework. We can use it to build multiple apps.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
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
  <span class="hljs-selector-class">.bounce-enter-active</span> {
    <span class="hljs-attribute">animation</span>: bounce-in <span class="hljs-number">0.5s</span>;
  }
  <span class="hljs-selector-class">.bounce-leave-active</span> {
    <span class="hljs-attribute">animation</span>: bounce-in <span class="hljs-number">0.5s</span> reverse;
  }
  <span class="hljs-keyword">@keyframes</span> bounce-in {
    0% {
      <span class="hljs-attribute">transform</span>: <span class="hljs-built_in">scale</span>(<span class="hljs-number">0</span>);
    }
    50% {
      <span class="hljs-attribute">transform</span>: <span class="hljs-built_in">scale</span>(<span class="hljs-number">1.5</span>);
    }
    100% {
      <span class="hljs-attribute">transform</span>: <span class="hljs-built_in">scale</span>(<span class="hljs-number">1</span>);
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span></span>
</code></pre>
<p data-nodeid="34398">和 CSS 过渡类似，CSS 动画主要定义了一些动画的 CSS 样式，当我们去点击按钮切换文本显隐的时候，就会应用这些 CSS 样式，实现动画效果。</p>
<p data-nodeid="34399">最后，是 JavaScript 钩子：</p>
<pre class="lang-javascript" data-nodeid="35410"><code data-language="javascript">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"show = !show"</span>&gt;</span>
      Toggle render
    <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">transition</span>
      @<span class="hljs-attr">before-enter</span>=<span class="hljs-string">"beforeEnter"</span>
      @<span class="hljs-attr">enter</span>=<span class="hljs-string">"enter"</span>
      @<span class="hljs-attr">before-leave</span>=<span class="hljs-string">"beforeLeave"</span>
      @<span class="hljs-attr">leave</span>=<span class="hljs-string">"leave"</span>
      <span class="hljs-attr">css</span>=<span class="hljs-string">"false"</span>
    &gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>hello<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">transition</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">show</span>: <span class="hljs-literal">true</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      beforeEnter(el) {
        el.style.opacity = <span class="hljs-number">0</span>
        el.style.transition = <span class="hljs-string">'opacity 0.5s ease'</span>
      },
      enter(el) {
        <span class="hljs-built_in">this</span>.$el.offsetHeight
        el.style.opacity = <span class="hljs-number">1</span>
      },
      beforeLeave(el) {
        el.style.opacity = <span class="hljs-number">1</span>
      },
      leave(el) {
        el.style.transition = <span class="hljs-string">'opacity 0.5s ease'</span>
        el.style.opacity = <span class="hljs-number">0</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="35411">Transition 组件也允许在一个过渡组件中定义它过渡生命周期的 JavaScript 钩子函数，我们可以在这些钩子函数中编写 JavaScript 操作 DOM 来实现过渡动画效果。</p>
<h3 data-nodeid="35412">Transition 组件的核心思想</h3>
<p data-nodeid="35413">通过前面三个示例，我们不难发现都是在点击按钮时，通过修改 v-if 的条件值来触发过渡动画的。</p>
<p data-nodeid="35414">其实 Transition 组件过渡动画的触发条件有以下四点：</p>
<ul data-nodeid="35415">
<li data-nodeid="35416">
<p data-nodeid="35417">条件渲染 (使用 v-if)；</p>
</li>
<li data-nodeid="35418">
<p data-nodeid="35419">条件展示 (使用 v-show)；</p>
</li>
<li data-nodeid="35420">
<p data-nodeid="35421">动态组件；</p>
</li>
<li data-nodeid="35422">
<p data-nodeid="35423">组件根节点。</p>
</li>
</ul>
<p data-nodeid="35424">所以你只能在上述四种情况中使用 Transition 组件，在进入/离开过渡的时候会有 6 个 class 切换。</p>
<ol data-nodeid="36220">
<li data-nodeid="36221">
<p data-nodeid="36222"><strong data-nodeid="36238">v-enter-from</strong>：定义进入过渡的开始状态。在元素被插入之前生效，在元素被插入之后的下一帧移除。</p>
</li>
<li data-nodeid="36223">
<p data-nodeid="36224"><strong data-nodeid="36243">v-enter-active</strong>：定义进入过渡生效时的状态。在整个进入过渡的阶段中应用，在元素被插入之前生效，在过渡动画完成之后移除。这个类可以被用来定义进入过渡的过程时间，延迟和曲线函数。</p>
</li>
<li data-nodeid="36225">
<p data-nodeid="36226"><strong data-nodeid="36248">v-enter-to</strong>：定义进入过渡的结束状态。在元素被插入之后下一帧生效 (与此同时 v-enter-from 被移除)，在过渡动画完成之后移除。</p>
</li>
<li data-nodeid="36227">
<p data-nodeid="36228"><strong data-nodeid="36253">v-leave-from</strong>：定义离开过渡的开始状态。在离开过渡被触发时立刻生效，下一帧被移除。</p>
</li>
<li data-nodeid="36229">
<p data-nodeid="36230"><strong data-nodeid="36258">v-leave-active</strong>：定义离开过渡生效时的状态。在整个离开过渡的阶段中应用，在离开过渡被触发时立刻生效，在过渡动画完成之后移除。这个类可以被用来定义离开过渡的过程时间，延迟和曲线函数。</p>
</li>
<li data-nodeid="36231">
<p data-nodeid="36232"><strong data-nodeid="36263">v-leave-to</strong>：定义离开过渡的结束状态。在离开过渡被触发之后下一帧生效 (与此同时 v-leave-from 被删除)，在过渡动画完成之后移除。</p>
</li>
</ol>
<p data-nodeid="36233" class=""><img src="https://s0.lgstatic.com/i/image/M00/55/F3/CgqCHl9q7XSAZVLbAAIHrhK4PT8658.png" alt="transitions.png" data-nodeid="36266"></p>


<p data-nodeid="35439">其实说白了 Transition 组件的核心思想就是，<strong data-nodeid="35518">Transition 包裹的元素插入删除时</strong>，<strong data-nodeid="35519">在适当的时机插入这些 CSS 样式</strong>，而这些 CSS 的实现则决定了元素的过渡动画。</p>
<p data-nodeid="35440">大致了解了 Transition 组件的用法和核心思想后，接下来我们就来探究 Transition 组件的实现原理。</p>
<h3 data-nodeid="35441">Transition 组件的实现原理</h3>
<p data-nodeid="35442">为了方便你的理解，我们还是结合示例来分析：</p>
<pre class="lang-js" data-nodeid="36605"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"app"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"show = !show"</span>&gt;</span>
      Toggle render
    <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">transition</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"fade"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">v-if</span>=<span class="hljs-string">"show"</span>&gt;</span>hello<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">transition</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
</code></pre>
<p data-nodeid="36606">先来看模板编译后生成的 render 函数：</p>
<pre class="lang-js" data-nodeid="36996"><code data-language="js"><span class="hljs-keyword">import</span> { createVNode <span class="hljs-keyword">as</span> _createVNode, openBlock <span class="hljs-keyword">as</span> _openBlock, createBlock <span class="hljs-keyword">as</span> _createBlock, createCommentVNode <span class="hljs-keyword">as</span> _createCommentVNode, Transition <span class="hljs-keyword">as</span> _Transition, withCtx <span class="hljs-keyword">as</span> _withCtx } <span class="hljs-keyword">from</span> <span class="hljs-string">"vue"</span>
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">render</span>(<span class="hljs-params">_ctx, _cache, $props, $setup, $data, $options</span>) </span>{
  <span class="hljs-keyword">return</span> (_openBlock(), _createBlock(<span class="hljs-string">"template"</span>, <span class="hljs-literal">null</span>, [
    _createVNode(<span class="hljs-string">"div"</span>, { <span class="hljs-attr">class</span>: <span class="hljs-string">"app"</span> }, [
      _createVNode(<span class="hljs-string">"button"</span>, {
        <span class="hljs-attr">onClick</span>: $event =&gt; (_ctx.show = !_ctx.show)
      }, <span class="hljs-string">" Toggle render "</span>, <span class="hljs-number">8</span> <span class="hljs-comment">/* PROPS */</span>, [<span class="hljs-string">"onClick"</span>]),
      _createVNode(_Transition, { <span class="hljs-attr">name</span>: <span class="hljs-string">"fade"</span> }, {
        <span class="hljs-attr">default</span>: _withCtx(<span class="hljs-function">() =&gt;</span> [
          (_ctx.show)
            ? (_openBlock(), _createBlock(<span class="hljs-string">"p"</span>, { <span class="hljs-attr">key</span>: <span class="hljs-number">0</span> }, <span class="hljs-string">"hello"</span>))
            : _createCommentVNode(<span class="hljs-string">"v-if"</span>, <span class="hljs-literal">true</span>)
        ]),
        <span class="hljs-attr">_</span>: <span class="hljs-number">1</span>
      })
    ])
  ]))
}
</code></pre>
<p data-nodeid="36997">对于 Transition 组件部分，生成的 render 函数主要创建了Transition 组件 vnode，并且有一个默认插槽。</p>
<p data-nodeid="36998">我们接着来看 Transition 组件的定义：</p>
<pre class="lang-java" data-nodeid="36999"><code data-language="java"><span class="hljs-keyword">const</span> Transition = (props, { slots }) =&gt; h(BaseTransition, resolveTransitionProps(props), slots)
<span class="hljs-keyword">const</span> BaseTransition = {
  name: `BaseTransition`,
  props: {
    mode: String,
    appear: Boolean,
    persisted: Boolean,
    <span class="hljs-comment">// enter</span>
    onBeforeEnter: TransitionHookValidator,
    onEnter: TransitionHookValidator,
    onAfterEnter: TransitionHookValidator,
    onEnterCancelled: TransitionHookValidator,
    <span class="hljs-comment">// leave</span>
    onBeforeLeave: TransitionHookValidator,
    onLeave: TransitionHookValidator,
    onAfterLeave: TransitionHookValidator,
    onLeaveCancelled: TransitionHookValidator,
    <span class="hljs-comment">// appear</span>
    onBeforeAppear: TransitionHookValidator,
    onAppear: TransitionHookValidator,
    onAfterAppear: TransitionHookValidator,
    onAppearCancelled: TransitionHookValidator
  },
  setup(props, { slots }) {
    <span class="hljs-keyword">const</span> instance = getCurrentInstance()
    <span class="hljs-keyword">const</span> state = useTransitionState()
    <span class="hljs-function">let prevTransitionKey
    <span class="hljs-title">return</span> <span class="hljs-params">()</span> </span>=&gt; {
      <span class="hljs-keyword">const</span> children = slots.<span class="hljs-keyword">default</span> &amp;&amp; getTransitionRawChildren(slots.<span class="hljs-keyword">default</span>(), <span class="hljs-keyword">true</span>)
      <span class="hljs-keyword">if</span> (!children || !children.length) {
        <span class="hljs-keyword">return</span>
      }
      <span class="hljs-comment">// Transition 组件只允许一个子元素节点，多个报警告，提示使用 TransitionGroup 组件</span>
      <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>) &amp;&amp; children.length &gt; <span class="hljs-number">1</span>) {
        warn(<span class="hljs-string">'&lt;transition&gt; can only be used on a single element or component. Use '</span> +
          <span class="hljs-string">'&lt;transition-group&gt; for lists.'</span>)
      }
      <span class="hljs-comment">// 不需要追踪响应式，所以改成原始值，提升性能</span>
      <span class="hljs-keyword">const</span> rawProps = toRaw(props)
      <span class="hljs-keyword">const</span> { mode } = rawProps
      <span class="hljs-comment">// 检查 mode 是否合法</span>
      <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>) &amp;&amp; mode &amp;&amp; ![<span class="hljs-string">'in-out'</span>, <span class="hljs-string">'out-in'</span>, <span class="hljs-string">'default'</span>].includes(mode)) {
        warn(`invalid &lt;transition&gt; mode: ${mode}`)
      }
      <span class="hljs-comment">// 获取第一个子元素节点</span>
      <span class="hljs-keyword">const</span> child = children[<span class="hljs-number">0</span>]
      <span class="hljs-keyword">if</span> (state.isLeaving) {
        <span class="hljs-keyword">return</span> emptyPlaceholder(child)
      }
      <span class="hljs-comment">// 处理 &lt;transition&gt;&lt;keep-alive/&gt;&lt;/transition&gt; 的情况</span>
      <span class="hljs-keyword">const</span> innerChild = getKeepAliveChild(child)
      <span class="hljs-keyword">if</span> (!innerChild) {
        <span class="hljs-keyword">return</span> emptyPlaceholder(child)
      }
      <span class="hljs-keyword">const</span> enterHooks = resolveTransitionHooks(innerChild, rawProps, state, instance)
        setTransitionHooks(innerChild, enterHooks)
      <span class="hljs-keyword">const</span> oldChild = instance.subTree
      <span class="hljs-keyword">const</span> oldInnerChild = oldChild &amp;&amp; getKeepAliveChild(oldChild)
      let transitionKeyChanged = <span class="hljs-keyword">false</span>
      <span class="hljs-keyword">const</span> { getTransitionKey } = innerChild.<span class="hljs-function">type
      <span class="hljs-title">if</span> <span class="hljs-params">(getTransitionKey)</span> </span>{
        <span class="hljs-keyword">const</span> key = getTransitionKey()
        <span class="hljs-keyword">if</span> (prevTransitionKey === undefined) {
          prevTransitionKey = key
        }
        <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (key !== prevTransitionKey) {
          prevTransitionKey = key
          transitionKeyChanged = <span class="hljs-keyword">true</span>
        }
      }
      <span class="hljs-keyword">if</span> (oldInnerChild &amp;&amp;
        oldInnerChild.type !== Comment &amp;&amp;
        (!isSameVNodeType(innerChild, oldInnerChild) || transitionKeyChanged)) {
        <span class="hljs-keyword">const</span> leavingHooks = resolveTransitionHooks(oldInnerChild, rawProps, state, instance)
        <span class="hljs-comment">// 更新旧树的钩子函数</span>
        setTransitionHooks(oldInnerChild, leavingHooks)
        <span class="hljs-comment">// 在两个视图之间切换</span>
        <span class="hljs-keyword">if</span> (mode === <span class="hljs-string">'out-in'</span>) {
          state.isLeaving = <span class="hljs-keyword">true</span>
          <span class="hljs-comment">// 返回空的占位符节点，当离开过渡结束后，重新渲染组件</span>
          leavingHooks.afterLeave = () =&gt; {
            state.isLeaving = <span class="hljs-keyword">false</span>
            instance.update()
          }
          <span class="hljs-keyword">return</span> emptyPlaceholder(child)
        }
        <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (mode === <span class="hljs-string">'in-out'</span>) {
          leavingHooks.delayLeave = (el, earlyRemove, delayedLeave) =&gt; {
            <span class="hljs-keyword">const</span> leavingVNodesCache = getLeavingNodesForType(state, oldInnerChild)
            leavingVNodesCache[String(oldInnerChild.key)] = oldInnerChild
            <span class="hljs-comment">// early removal callback</span>
            el._leaveCb = () =&gt; {
              earlyRemove()
              el._leaveCb = undefined
              delete enterHooks.delayedLeave
            }
            enterHooks.delayedLeave = delayedLeave
          }
        }
      }
      <span class="hljs-keyword">return</span> child
    }
  }
}
</code></pre>
<p data-nodeid="37000">可以看到，Transition 组件是在 BaseTransition 的基础上封装的高阶函数式组件。由于整个 Transition 的实现代码较多，我就挑重点，为你讲清楚整体的实现思路。</p>
<p data-nodeid="37001">我把 Transition 组件的实现分成组件的渲染、钩子函数的执行、模式的应用三个部分去详细说明。</p>
<h4 data-nodeid="37002">组件的渲染</h4>
<p data-nodeid="37003">先来看 Transition 组件是如何渲染的。我们重点看 setup 函数部分的逻辑。</p>
<p data-nodeid="37004">Transition 组件和前面学习的 KeepAlive 组件一样，是一个抽象组件，组件本身不渲染任何实体节点，只渲染第一个子元素节点。</p>
<blockquote data-nodeid="37005">
<p data-nodeid="37006">注意，Transition 组件内部只能嵌套一个子元素节点，如果有多个节点需要用 TransitionGroup 组件。</p>
</blockquote>
<p data-nodeid="37007">如果 Transition 组件内部嵌套的是 KeepAlive 组件，那么它会继续查找 KeepAlive 组件嵌套的第一个子元素节点，来作为渲染的元素节点。</p>
<p data-nodeid="37008">如果 Transition 组件内部没有嵌套任何子节点，那么它会渲染空的注释节点。</p>
<p data-nodeid="37009">在渲染的过程中，Transition 组件还会通过 resolveTransitionHooks 去定义组件创建和删除阶段的钩子函数对象，然后再通过 setTransitionHooks函数去把这个钩子函数对象设置到 vnode.transition 上。</p>
<p data-nodeid="37010">渲染过程中，还会判断这是否是一次更新渲染，如果是会对不同的模式执行不同的处理逻辑，我会在后续介绍模式的应用时详细说明。</p>
<p data-nodeid="37011">以上就是 Transition 组件渲染做的事情，你需要记住的是<strong data-nodeid="37035">Transition 渲染的是组件嵌套的第一个子元素节点</strong>。</p>
<p data-nodeid="37012">但是 Transition 是如何在节点的创建和删除过程中设置那些与过渡动画相关的 CSS 的呢？这些都与钩子函数相关，我们先来看 setTransitionHooks 的实现，看看它定义的钩子函数对象是怎样的：</p>
<pre class="lang-java" data-nodeid="37013"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">resolveTransitionHooks</span><span class="hljs-params">(vnode, props, state, instance)</span> </span>{
  <span class="hljs-keyword">const</span> { appear, mode, persisted = <span class="hljs-keyword">false</span>, onBeforeEnter, onEnter, onAfterEnter, onEnterCancelled, onBeforeLeave, onLeave, onAfterLeave, onLeaveCancelled, onBeforeAppear, onAppear, onAfterAppear, onAppearCancelled } = props
  <span class="hljs-keyword">const</span> key = String(vnode.key)
  <span class="hljs-keyword">const</span> leavingVNodesCache = getLeavingNodesForType(state, vnode)
  <span class="hljs-keyword">const</span> callHook = (hook, args) =&gt; {
    hook &amp;&amp;
    callWithAsyncErrorHandling(hook, instance, <span class="hljs-number">9</span> <span class="hljs-comment">/* TRANSITION_HOOK */</span>, args)
  }
  <span class="hljs-keyword">const</span> hooks = {
    mode,
    persisted,
    beforeEnter(el) {
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
    },
    enter(el) {
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
    },
    leave(el, remove) {
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
    },
    clone(vnode) {
      <span class="hljs-keyword">return</span> resolveTransitionHooks(vnode, props, state, instance)
    }
  }
  <span class="hljs-keyword">return</span> hooks
}
</code></pre>
<p data-nodeid="37014">钩子函数对象定义了 4 个钩子函数，分别是 beforeEnter，enter，leave 和 clone，它们的执行时机是什么，又是怎么处理 我们给 Transition 组件传递的一些 Prop 的？其中，beforeEnter、enter 和 leave 发生在元素的插入和删除阶段，接下来我们就来分析这几个钩子函数的执行过程。</p>
<p data-nodeid="37015">好的，今天我们就先讲到这里，下节课继续分析钩子函数的执行。</p>
<blockquote data-nodeid="37016">
<p data-nodeid="37017">本节课的相关代码在源代码中的位置如下：<br>
packages/runtime-core/src/components/BasetTransition.ts<br>
packages/runtime-core/src/renderer.ts<br>
packages/runtime-dom/src/components/Transition.ts</p>
</blockquote>

---

### 精选评论


