<p data-nodeid="48841" class="">在 React 的面试中会对 Hooks API 进行一个区分度考察，重点往往会落在 useEffect 与 useLayoutEffect 上。很有意思，光从名字来看，它们就很相像，所以被点名的概率就很高。这一讲我们来重点讲解下这部分内容。</p>
<h3 data-nodeid="48842">审题</h3>
<p data-nodeid="48843">在第 04 讲<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=566&amp;sid=20-h5Url-0#/detail/pc?id=5794" data-nodeid="48938">“类组件与函数组件有什么区别呢？”</a>中讲过该类型的题目，其中提到<strong data-nodeid="48944">描述区别，就是求同存异的过程。</strong> 那我们可以直接用同样的思路来思考下这道题。</p>
<p data-nodeid="48844">先挖掘 useEffect 与 useLayoutEffect 的共性，它们被用于解决什么问题，其次发掘独特的个性、各自适用的场景、设计原理以及未来趋势等。</p>
<h3 data-nodeid="48845">承题</h3>
<p data-nodeid="48846">根据以上的分析，该讲所讲解题目的答题思路就有了。</p>
<p data-nodeid="48847">首先是论述<strong data-nodeid="48953">共同点</strong>：</p>
<ul data-nodeid="48848">
<li data-nodeid="48849">
<p data-nodeid="48850"><strong data-nodeid="48958">使用方式</strong>，也就是列举使用方式上有什么相似处，共同用于解决什么问题；</p>
</li>
<li data-nodeid="48851">
<p data-nodeid="48852"><strong data-nodeid="48963">运用效果</strong>，使用后的执行效果上有什么相似处。</p>
</li>
</ul>
<p data-nodeid="48853">其次是<strong data-nodeid="48969">不同点</strong>：</p>
<ul data-nodeid="48854">
<li data-nodeid="48855">
<p data-nodeid="48856"><strong data-nodeid="48974">使用场景</strong>，两者在使用场景的区分点在哪里，为什么可以或者不可以混用；</p>
</li>
<li data-nodeid="48857">
<p data-nodeid="48858"><strong data-nodeid="48979">独有能力</strong>，什么能力是其独有的，而另外一方没有的；</p>
</li>
<li data-nodeid="48859">
<p data-nodeid="48860"><strong data-nodeid="48984">设计原理</strong>，即从内部设计挖掘本质的原因；</p>
</li>
<li data-nodeid="48861">
<p data-nodeid="48862"><strong data-nodeid="48989">未来趋势</strong>，两者在未来的发展趋势上有什么区别，是否存在一方可能在使用频率上胜出或者淡出的情况。</p>
</li>
</ul>
<p data-nodeid="48863"><img src="https://s0.lgstatic.com/i/image/M00/90/3F/Ciqc1GAKhGuAeJVzAABnKbg5gv0029.png" alt="Drawing 1.png" data-nodeid="48992"></p>
<h3 data-nodeid="48864">破题</h3>
<h4 data-nodeid="48865">共同点</h4>
<p data-nodeid="48866"><strong data-nodeid="48998">使用方式</strong></p>
<p data-nodeid="48867">如果你读过 React Hooks 的官方文档，你可能会发现这么一段描述：useLayoutEffect 的函数签名与 useEffect 相同。</p>
<p data-nodeid="48868">那什么是<strong data-nodeid="49005">函数签名呢</strong>？函数签名就像我们在银行账号上签写的个人签名一样，独一无二，具有法律效应。以下面这段代码为例：</p>
<pre class="lang-java" data-nodeid="48869"><code data-language="java">MyObject.prototype.myFunction(value)
</code></pre>
<p data-nodeid="48870">应用 MDN 的描述，在 JavaScript 中的签名通常包括这样几个部分：</p>
<ul data-nodeid="48871">
<li data-nodeid="48872">
<p data-nodeid="48873">该函数是安装在一个名为&nbsp;MyObject&nbsp;的<a href="https://developer.mozilla.org/zh-CN/docs/Glossary/Object" data-nodeid="49010">对象</a>上；</p>
</li>
<li data-nodeid="48874">
<p data-nodeid="48875">该函数安装在&nbsp;MyObject&nbsp;的原型上（因此它是一个<a href="https://developer.mozilla.org/en-US/docs/Glossary/instance_method" data-nodeid="49015">实例方法</a>，而不是一个<a href="https://developer.mozilla.org/en-US/docs/Glossary/static_method" data-nodeid="49019">静态方法/类方法</a>）；</p>
</li>
<li data-nodeid="48876">
<p data-nodeid="48877">该函数的名称是&nbsp;myFunction；</p>
</li>
<li data-nodeid="48878">
<p data-nodeid="48879">该函数接收一个叫&nbsp;value&nbsp;的参数，且没有进一步定义。</p>
</li>
</ul>
<p data-nodeid="48880">那为什么说 useEffect 与 useLayoutEffect 函数签名相同呢？它们俩的名字完全不同啊。这是因为在源码中，它们调用的是同一个函数。下面的这段代码是 React useEffect 与 useLayoutEffect 在 ReactFiberHooks.js 源码中的样子。</p>
<pre class="lang-java" data-nodeid="48881"><code data-language="java"><span class="hljs-comment">// useEffect</span>
useEffect(
   create: () =&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
   deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
 ): <span class="hljs-keyword">void</span> {
   currentHookNameInDev = <span class="hljs-string">'useEffect'</span>;
   mountHookTypesDev();
   checkDepsAreArrayDev(deps);
   <span class="hljs-keyword">return</span> mountEffect(create, deps);
 },
 
<span class="hljs-function">function <span class="hljs-title">mountEffect</span><span class="hljs-params">(
	  create: ()</span> </span>=&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
	  deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
	): <span class="hljs-keyword">void</span> {
	  <span class="hljs-keyword">if</span> (__DEV__) {
	    <span class="hljs-comment">// $FlowExpectedError - jest isn't a global, and isn't recognized outside of tests</span>
	    <span class="hljs-keyword">if</span> (<span class="hljs-string">'undefined'</span> !== typeof jest) {
	      warnIfNotCurrentlyActingEffectsInDEV(currentlyRenderingFiber);
	    }
	  }
	  <span class="hljs-keyword">return</span> mountEffectImpl(
	    UpdateEffect | PassiveEffect | PassiveStaticEffect,
	    HookPassive,
	    create,
	    deps,
	  );
	}
 
<span class="hljs-comment">// useLayoutEffect</span>
useLayoutEffect(
   create: () =&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
   deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
 ): <span class="hljs-keyword">void</span> {
   currentHookNameInDev = <span class="hljs-string">'useLayoutEffect'</span>;
   mountHookTypesDev();
   checkDepsAreArrayDev(deps);
   <span class="hljs-keyword">return</span> mountLayoutEffect(create, deps);
 },
 
<span class="hljs-function">function <span class="hljs-title">mountLayoutEffect</span><span class="hljs-params">(
	  create: ()</span> </span>=&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
	  deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
	): <span class="hljs-keyword">void</span> {
	  <span class="hljs-keyword">return</span> mountEffectImpl(UpdateEffect, HookLayout, create, deps);
}
</code></pre>
<p data-nodeid="48882">可以看出：</p>
<ul data-nodeid="48883">
<li data-nodeid="48884">
<p data-nodeid="48885">useEffect 先调用 mountEffect，再调用 mountEffectImpl；</p>
</li>
<li data-nodeid="48886">
<p data-nodeid="48887">useLayoutEffect 会先调用 mountLayoutEffect，再调用 mountEffectImpl。</p>
</li>
</ul>
<p data-nodeid="48888">那么你会发现最终调用的都是同一个名为 mountEffectImpl 的函数，入参一致，返回值也一致，所以函数签名是相同的。</p>
<p data-nodeid="48889">从代码角度而言，虽然是两个函数，但使用方式是完全一致的，甚至一定程度上可以相互替换。</p>
<p data-nodeid="48890"><strong data-nodeid="49032">运用效果</strong></p>
<p data-nodeid="48891">从运用效果上而言，useEffect 与 useLayoutEffect 两者都是<strong data-nodeid="49038">用于处理副作用</strong>，这些副作用包括改变 DOM、设置订阅、操作定时器等。在函数组件内部操作副作用是不被允许的，所以需要使用这两个函数去处理。</p>
<p data-nodeid="48892">虽然看起来很像，但在执行效果上仍然有些许差异。React 官方团队甚至直言，如果不能掌握 useLayoutEffect，不妨直接使用 useEffect。在使用 useEffect 时遇到了问题，再尝试使用 useLayoutEffect。</p>
<h4 data-nodeid="48893">不同点</h4>
<p data-nodeid="48894"><strong data-nodeid="49044">使用场景</strong></p>
<p data-nodeid="48895">虽然官方团队给出了一个看似友好的建议，但我们并不能将这样模糊的结果作为正式答案回复给面试官。所以两者的差异在哪里？我们不如用代码来说明。下面通过一个案例来讲解两者的区别。</p>
<p data-nodeid="48896">先使用 useEffect 编写一个组件，在这个组件里面包含了两部分：</p>
<ul data-nodeid="48897">
<li data-nodeid="48898">
<p data-nodeid="48899">组件展示内容，也就是 className 为 square 的部分会展示一个圆圈；</p>
</li>
<li data-nodeid="48900">
<p data-nodeid="48901">在 useEffect 中操作修改 square 的样式，将它的样式重置到页面正中间。</p>
</li>
</ul>
<pre class="lang-js" data-nodeid="48902"><code data-language="js"><span class="hljs-keyword">import</span> React, { useEffect } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">"./styles.css"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> () =&gt; {
  useEffect(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">const</span> greenSquare = <span class="hljs-built_in">document</span>.querySelector(<span class="hljs-string">".square"</span>);
    greenSquare.style.transform = <span class="hljs-string">"translate(-50%, -50%)"</span>;
    greenSquare.style.left = <span class="hljs-string">"50%"</span>;
    greenSquare.style.top = <span class="hljs-string">"50%"</span>;
  });
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"square"</span> /&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
};
</code></pre>
<p data-nodeid="48903">下面再补充一下样式文件，其中 App 样式中规中矩设置间距，square 样式主要设置圈儿的颜色与大小，接下来就可以看看它的效果了。</p>
<pre class="lang-java" data-nodeid="48904"><code data-language="java">.App {
  text-align: center;
  margin: <span class="hljs-number">0</span>;
  padding: <span class="hljs-number">0</span>;
}
.square {
  width: <span class="hljs-number">100</span>px;
  height: <span class="hljs-number">100</span>px;
  position: absolute;
  top: <span class="hljs-number">50</span>px;
  left: <span class="hljs-number">0</span>;
  background: red;
  border-radius: <span class="hljs-number">50</span>%;
}
</code></pre>
<p data-nodeid="48905">然后执行代码， 你就会发现红圈在渲染后出现了肉眼可见的瞬移，一下飘到了中间。</p>
<p data-nodeid="48906"><img src="https://s0.lgstatic.com/i/image2/M01/08/32/Cip5yGAKhLOADIYUAAIoKyWYNqU863.gif" alt="GIF1.gif" data-nodeid="49053"></p>
<p data-nodeid="48907">那如果使用 useLayoutEffect 又会怎么样呢？其他代码都不需要修改，只需要像下面这样，将 useEffect 替换为 useLayoutEffect：</p>
<pre class="lang-java" data-nodeid="48908"><code data-language="java">import React, { useLayoutEffect } from "react";
import "./styles.css";
export default () =&gt; {
  useLayoutEffect(() =&gt; {
    const greenSquare = document.querySelector(".square");
    greenSquare.style.transform = "translate(-50%, -50%)";
    greenSquare.style.left = "50%";
    greenSquare.style.top = "50%";
  });
  return (
    &lt;div className="App"&gt;
      &lt;div className="square" /&gt;
    &lt;/div&gt;
  );
};
</code></pre>
<p data-nodeid="48909">接下来再看执行的效果，你会发现红圈是静止在页面中央，仿佛并没有使用代码强制调整样式的过程。</p>
<p data-nodeid="48910"><img src="https://s0.lgstatic.com/i/image2/M01/08/34/CgpVE2AKhLyANGs2AAIwo7CyA_E780.gif" alt="GIF2.gif" data-nodeid="49058"></p>
<p data-nodeid="48911">虽然在实际的项目中，我们并不会这么粗暴地去调整组件样式，但这个案例足以说明两者的区别与使用场景。在 React 社区中最佳的实践是这样推荐的，大多数场景下可以直接使用<strong data-nodeid="49068">useEffect</strong>，但是如果你的代码引起了页面闪烁，也就是引起了组件突然改变位置、颜色及其他效果等的情况下，就推荐使用<strong data-nodeid="49069">useLayoutEffect</strong>来处理。那么总结起来就是如果有直接操作 DOM 样式或者引起 DOM 样式更新的场景更推荐使用 useLayoutEffect。</p>
<p data-nodeid="48912">那既然内部都是调用同一个函数，为什么会有这样的区别呢？在探讨这个问题时就需要从 Hooks 的设计原理说起了。</p>
<p data-nodeid="48913"><strong data-nodeid="49074">设计原理</strong></p>
<p data-nodeid="48914">首先可以看下这个图：</p>
<p data-nodeid="48915"><img src="https://s0.lgstatic.com/i/image2/M01/08/34/CgpVE2AKhP6AFNRnAAB9M55aj8I408.png" alt="Drawing 4.png" data-nodeid="49078"></p>
<p data-nodeid="48916">这个图表达了什么意思呢？首先所有的 Hooks，也就是 useState、useEffect、useLayoutEffect 等，都是导入到了 Dispatcher 对象中。在调用 Hook 时，会通过 Dispatcher 调用对应的 Hook 函数。所有的 Hooks 会按顺序存入对应 Fiber 的状态队列中，这样 React 就能知道当前的 Hook 属于哪个 Fiber，这里就是<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=566&amp;sid=20-h5Url-0#/detail/pc?id=5806" data-nodeid="49082">上一讲</a>所提到的<strong data-nodeid="49088">Hooks 链表</strong>。但 Effect Hooks 会有些不同，它涉及了一些额外的处理逻辑。每个 Fiber 的 Hooks 队列中保存了 effect 节点，而每个 effect 的类型都有可能不同，需要在合适的阶段去执行。</p>
<p data-nodeid="48917">那么 LayoutEffect 与普通的 Effect 都是 effect，但标记并不一样，所以在调用时，就会有些许不同。回到前面的底层代码，你会发现只有第一个参数和第二个参数是不一样的，其中 UpdateEffect、PassiveEffect、PassiveStaticEffect 就是 Fiber 的标记；HookPassive 和 HookLayout 就是当前 Effect 的标记。如下代码所示：</p>
<pre class="lang-java" data-nodeid="48918"><code data-language="java"><span class="hljs-comment">// useEffect 调用的底层函数</span>
<span class="hljs-function">function <span class="hljs-title">mountEffect</span><span class="hljs-params">(
	  create: ()</span> </span>=&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
	  deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
	): <span class="hljs-keyword">void</span> {
	  <span class="hljs-keyword">if</span> (__DEV__) {
	    <span class="hljs-comment">// $FlowExpectedError - jest isn't a global, and isn't recognized outside of tests</span>
	    <span class="hljs-keyword">if</span> (<span class="hljs-string">'undefined'</span> !== typeof jest) {
	      warnIfNotCurrentlyActingEffectsInDEV(currentlyRenderingFiber);
	    }
	  }
	  <span class="hljs-keyword">return</span> mountEffectImpl(
	    UpdateEffect | PassiveEffect | PassiveStaticEffect,
	    HookPassive,
	    create,
	    deps,
	  );
	}
<span class="hljs-comment">// useLayoutEffect 调用的底层函数</span>
<span class="hljs-function">function <span class="hljs-title">mountLayoutEffect</span><span class="hljs-params">(
	  create: ()</span> </span>=&gt; (() =&gt; <span class="hljs-keyword">void</span>) | <span class="hljs-keyword">void</span>,
	  deps: Array&lt;mixed&gt; | <span class="hljs-keyword">void</span> | <span class="hljs-keyword">null</span>,
	): <span class="hljs-keyword">void</span> {
	  <span class="hljs-keyword">return</span> mountEffectImpl(UpdateEffect, HookLayout, create, deps);
	}
</code></pre>
<p data-nodeid="48919">标记为 HookLayout 的 effect 会在所有的 DOM 变更之后同步调用，所以可以使用它来读取 DOM 布局并同步触发重渲染。但既然是同步，就有一个问题，计算量较大的耗时任务必然会造成阻塞，所以这就需要根据实际情况酌情考虑了。如果非必要情况下，使用标准的 useEffect 可以避免阻塞。这段代码在 react/packages/react-reconciler/src/ReactFiberCommitWork.new.js 中，有兴趣的同学可以研读一下。</p>
<h3 data-nodeid="48920">答题</h3>
<p data-nodeid="48921">那么以上就是本讲的全部知识点了，内容并不太多，重点主要在于<strong data-nodeid="49097">分析思路</strong>。那么下面就可以进入答题环节了。</p>
<blockquote data-nodeid="48922">
<p data-nodeid="48923">useEffect 与 useLayoutEffect 的区别在哪里？这个问题可以分为两部分来回答，共同点与不同点。</p>
<p data-nodeid="48924">它们的共同点很简单，底层的函数签名是完全一致的，都是调用的 mountEffectImpl，在使用上也没什么差异，基本可以直接替换，也都是用于处理副作用。</p>
<p data-nodeid="48925">那不同点就很大了，useEffect 在 React 的渲染过程中是被异步调用的，用于绝大多数场景，而 LayoutEffect 会在所有的 DOM 变更之后同步调用，主要用于处理 DOM 操作、调整样式、避免页面闪烁等问题。也正因为是同步处理，所以需要避免在 LayoutEffect 做计算量较大的耗时任务从而造成阻塞。</p>
<p data-nodeid="48926">在未来的趋势上，两个 API 是会长期共存的，暂时没有删减合并的计划，需要开发者根据场景去自行选择。React 团队的建议非常实用，如果实在分不清，先用 useEffect，一般问题不大；如果页面有异常，再直接替换为 useLayoutEffect 即可。</p>
</blockquote>
<p data-nodeid="48927"><img src="https://s0.lgstatic.com/i/image2/M01/08/32/Cip5yGAKhRCAX99HAAD0YKYP40c980.png" alt="Drawing 6.png" data-nodeid="49104"></p>
<h3 data-nodeid="48928">总结</h3>
<p data-nodeid="48929">本题仍然是一个讲区别的点，所以在整体思路上找相同与不同就可以。</p>
<p data-nodeid="48930">这里还有一个很有意思的地方，React 的函数命名追求“望文生义”的效果，这里不是贬义，它在设计上就是希望你从名字猜出真实的作用。比如 componentDidMount、componentDidUpdate 等等虽然名字冗长，但容易理解。从 LayoutEffect 这样一个命名就能看出，它想解决的也就是页面布局的问题。</p>
<p data-nodeid="48931">那么在实际的开发中，还有哪些你觉得不太容易理解的 Hooks？或者容易出错的 Hooks？不妨在留言区留言，我会与你一起交流讨论。</p>
<p data-nodeid="49652">这一讲就到这了，在下一讲中，将主要介绍 React Hooks 的设计模式，到时见。</p>
<hr data-nodeid="49653">
<p data-nodeid="49654"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="49662"><img src="https://s0.lgstatic.com/i/image/M00/72/94/Ciqc1F_EZ0eANc6tAASyC72ZqWw643.png" alt="Drawing 2.png" data-nodeid="49661"></a></p>
<p data-nodeid="49655">《大前端高薪训练营》</p>
<p data-nodeid="49656" class="te-preview-highlight">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="49667">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **兵：
> useContext 有什么最佳实践吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 回答里篇幅有限，可以看一下这篇 《A deeper dive featuring useContext and useReducer》（https://testdriven.io/blog/react-hooks-advanced）。

##### *盼：
> 赞👍

