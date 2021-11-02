<p data-nodeid="29600" class="">diff 算法通常被认为 React 的核心点，在面试中很受重视。之前，关于 diff 算法的问法仅仅停留在“React 的 diff 算法的流程是什么”。但近几年有所变化，问题开始变得多样了，一个比较常见的场景是对比其他框架进行阐述。这一讲我们来一起探讨下这个问题应该如何回答。</p>
<h3 data-nodeid="29601">破题</h3>
<p data-nodeid="29602">这个题目虽然有对比，但本质上仍然是一道原理题。根据我们之前学习的方法论，原理题需要按照“<strong data-nodeid="29740">讲概念</strong>，<strong data-nodeid="29741">说用途</strong>，<strong data-nodeid="29742">理思路</strong>，<strong data-nodeid="29743">优缺点</strong>，<strong data-nodeid="29744">列一遍</strong>”的思路来答题。</p>
<p data-nodeid="29603">针对 React 而言，diff 算法是对<strong data-nodeid="29761">知识深度的考核</strong>。面试官考察的不仅仅是你会用就可以，更重要的是你在使用中有没有思考，对 diff 算法有没有透彻的理解，这是本题的第一道关卡。对于前端工程师而言，<strong data-nodeid="29762">这是一道能够快速划分</strong>“<strong data-nodeid="29763">内功</strong>”<strong data-nodeid="29764">等级的常见题目。</strong></p>
<p data-nodeid="29604">而题目中的“其他框架”，则是在考核你知识面的广度。所以你在回答的时候，务必采取“先分类，后讲述”的方式。切忌语无伦次，没有条理、没有区分度、一股脑地表达。而且，你的分类方式还向面试官透露了对知识点的理解度。</p>
<p data-nodeid="29605">那讲到 React，不得不说的其他框架有两个。</p>
<ul data-nodeid="29606">
<li data-nodeid="29607">
<p data-nodeid="29608">Vue，因为 React 与 Vue 是国内前端中的主流框架。</p>
</li>
<li data-nodeid="29609">
<p data-nodeid="29610">类 React 框架，又被称为 React-like 框架，通常是指 Preact、 inferno 等兼容 React API 的框架，它们与 React 设计相似、 使用相似。</p>
</li>
</ul>
<p data-nodeid="29611">所以该讲我们就拿 Vue 和 Preact 与 React 的 diff 算法进行比较。</p>
<h3 data-nodeid="29612">承题</h3>
<p data-nodeid="29613">到这里，我们就清楚了本讲考察的两个重点，可以梳理出这样的一个答题框架：</p>
<ol data-nodeid="29614">
<li data-nodeid="29615">
<p data-nodeid="29616">就 React diff 算法完成“讲概念，说用途，理思路，优缺点，列一遍”的组合拳；</p>
</li>
<li data-nodeid="29617">
<p data-nodeid="29618">横向对比 React、React-like 框架及 Vue 的 diff 算法。</p>
</li>
</ol>
<p data-nodeid="29619"><img src="https://s0.lgstatic.com/i/image/M00/8C/55/CgqCHl_qyoCARaC-AABHz3sJYwo329.png" alt="Drawing 1.png" data-nodeid="29776"></p>
<h3 data-nodeid="29620">入手</h3>
<h4 data-nodeid="29621">Diff 算法</h4>
<p data-nodeid="29622">首先主角当然是“diff 算法”，但讨论 diff 算法一定是建立在虚拟 DOM 的基础上的。第 09 讲“Virtual DOM 的工作原理是什么？”讲过，使用虚拟 DOM 而非直接操作真实 DOM 是现代前端框架的一个基本认知。</p>
<p data-nodeid="29623">而 diff 算法探讨的就是虚拟 DOM 树发生变化后，生成 DOM 树更新补丁的方式。它通过对比新旧两株虚拟 DOM 树的变更差异，将更新补丁作用于真实 DOM，以最小成本完成视图更新。</p>
<p data-nodeid="29624"><img src="https://s0.lgstatic.com/i/image/M00/8C/55/CgqCHl_qyouAAkb9AAB_cmWuZhc920.png" alt="Drawing 3.png" data-nodeid="29783"></p>
<p data-nodeid="29625">具体的流程是这样的：</p>
<ul data-nodeid="29626">
<li data-nodeid="29627">
<p data-nodeid="29628">真实 DOM 与虚拟 DOM 之间存在一个映射关系。这个映射关系依靠初始化时的 JSX 建立完成；</p>
</li>
<li data-nodeid="29629">
<p data-nodeid="29630">当虚拟 DOM 发生变化后，就会根据差距计算生成 patch，这个 patch 是一个结构化的数据，内容包含了增加、更新、移除等；</p>
</li>
<li data-nodeid="29631">
<p data-nodeid="29632">最后再根据 patch 去更新真实的 DOM，反馈到用户的界面上。</p>
</li>
</ul>
<p data-nodeid="29633"><img src="https://s0.lgstatic.com/i/image/M00/8C/55/CgqCHl_qypGAZPuGAADYrK9nkJY878.png" alt="Drawing 4.png" data-nodeid="29790"></p>
<p data-nodeid="29634">举一个简单易懂的例子：</p>
<pre class="lang-java" data-nodeid="29635"><code data-language="java">import React from 'react'

export default class ExampleComponent extends React.Component {
  render() {
    if(this.props.isVisible) {
       return &lt;div className="visible"&gt;visbile&lt;/div&gt;;
    }
     return &lt;div className="hidden"&gt;hidden&lt;/div&gt;;
  }
}
</code></pre>
<p data-nodeid="29636">这里，首先我们假定 ExampleComponent 可见，然后再改变它的状态，让它不可见 。映射为真实的 DOM 操作是这样的，React 会创建一个 div 节点。</p>
<pre class="lang-java" data-nodeid="29637"><code data-language="java"> &lt;div class="visible"&gt;visbile&lt;/div&gt;
</code></pre>
<p data-nodeid="29638">当把 visbile 的值变为 false 时，就会替换 class 属性为 hidden，并重写内部的 innerText 为 hidden。这样一个<strong data-nodeid="29802">生成补丁</strong>、<strong data-nodeid="29803">更新差异</strong>的过程统称为 diff 算法。</p>
<p data-nodeid="29639">在整个过程中你需要注意 3 点：<strong data-nodeid="29809">更新时机、遍历算法、优化策略</strong>，这些也是面试官最爱考察的。</p>
<p data-nodeid="29640"><strong data-nodeid="29813">更新时机</strong></p>
<p data-nodeid="29641">更新时机就是触发更新、进行差异对比的时机。根据前面的章节内容可以知道，更新发生在setState、Hooks 调用等操作以后。此时，树的结点发生变化，开始进行比对。那这里涉及一个问题，即两株树如何对比差异?</p>
<p data-nodeid="29642">这里就需要使用遍历算法。</p>
<p data-nodeid="29643"><strong data-nodeid="29819">遍历算法</strong></p>
<p data-nodeid="29644">遍历算法是指沿着某条搜索路线，依次对树的每个节点做访问。通常分为两种：深度优先遍历和广度优先遍历。</p>
<ul data-nodeid="29645">
<li data-nodeid="29646">
<p data-nodeid="29647">深度优先遍历，是从根节点出发，沿着左子树方向进行纵向遍历，直到找到叶子节点为止。然后回溯到前一个节点，进行右子树节点的遍历，直到遍历完所有可达节点。</p>
</li>
<li data-nodeid="29648">
<p data-nodeid="29649">广度优先遍历，则是从根节点出发，在横向遍历二叉树层段节点的基础上，纵向遍历二叉树的层次。</p>
</li>
</ul>
<p data-nodeid="29650">React 选择了哪一种遍历方式呢？它的 diff 算法采用了深度优先遍历算法。因为广度优先遍历可能会导致组件的生命周期时序错乱，而深度优先遍历算法就可以解决这个问题。</p>
<p data-nodeid="29651"><strong data-nodeid="29827">优化策略</strong></p>
<p data-nodeid="29652">优化策略是指 React 对 diff 算法做的优化手段。</p>
<p data-nodeid="29653">虽然深度优先遍历保证了组件的生命周期时序不错乱，但传统的 diff 算法也带来了一个严重的性能瓶颈，复杂程度为 O(n^3)(后期备注上角标)，其中 n 表示树的节点总数。正如计算机科学中常见的优化方案一样，React 用了一个非常经典的手法将复杂度降低为 O(n)，也就是<a href="https://leetcode-cn.com/tag/divide-and-conquer/" data-nodeid="29832">分治</a>，即通过“分而治之”这一巧妙的思想分解问题。</p>
<p data-nodeid="29654">具体而言， React 分别从<strong data-nodeid="29847">树</strong>、<strong data-nodeid="29848">组件</strong>及<strong data-nodeid="29849">元素</strong>三个层面进行复杂度的优化，并诞生了与之对应的策略。</p>
<p data-nodeid="29655"><strong data-nodeid="29858">策略一</strong>：<strong data-nodeid="29859">忽略节点跨层级操作场景，提升比对效率</strong>。</p>
<p data-nodeid="29656">这一策略需要进行<strong data-nodeid="29865">树比对</strong>，即对树进行分层比较。树比对的处理手法是非常“暴力”的，即两棵树只对同一层次的节点进行比较，如果发现节点已经不存在了，则该节点及其子节点会被完全删除掉，不会用于进一步的比较，这就提升了比对效率。</p>
<p data-nodeid="29657"><strong data-nodeid="29882">策略二</strong>：<strong data-nodeid="29883">如果组件的 class 一致</strong>，<strong data-nodeid="29884">则默认为相似的树结构</strong>，<strong data-nodeid="29885">否则默认为不同的树结构</strong>。</p>
<p data-nodeid="29658">在组件比对的过程中：</p>
<ul data-nodeid="29659">
<li data-nodeid="29660">
<p data-nodeid="29661">如果组件是同一类型则进行树比对；</p>
</li>
<li data-nodeid="29662">
<p data-nodeid="29663">如果不是则直接放入补丁中。</p>
</li>
</ul>
<p data-nodeid="29664">只要父组件类型不同，就会被重新渲染。这也就是为什么</p>
<p data-nodeid="29665">shouldComponentUpdate、PureComponent 及 React.memo 可以提高性能的原因。</p>
<p data-nodeid="29666"><strong data-nodeid="29899">策略三：同一层级的子节点</strong>，<strong data-nodeid="29900">可以通过标记 key 的方式进行列表对比</strong>。</p>
<p data-nodeid="29667">元素比对主要发生在同层级中，通过<strong data-nodeid="29906">标记节点操作</strong>生成补丁。节点操作包含了插入、移动、删除等。其中节点重新排序同时涉及插入、移动、删除三个操作，所以效率消耗最大，此时策略三起到了至关重要的作用。</p>
<p data-nodeid="29668">通过标记 key 的方式，React 可以直接移动 DOM 节点，降低内耗。操作代码如下：</p>
<pre class="lang-java" data-nodeid="29669"><code data-language="java">&lt;ul&gt;
  &lt;li key="a"&gt;a&lt;/li&gt;
  &lt;li key="b"&gt;b&lt;/li&gt;
  &lt;li key="c"&gt;c&lt;/li&gt;
  &lt;li key="d"&gt;d&lt;/li&gt;
&lt;/ui&gt;
</code></pre>
<p data-nodeid="29670">以上是 React Diff 算法最基本的内容，除此以外，由于 React 16 引入<strong data-nodeid="29913">Fiber 设计</strong>，所以我们还需要了解 Fiber 给 diff 算法带来的影响。</p>
<p data-nodeid="29671">Fiber 机制下节点与树分别采用 FiberNode 与 FiberTree 进行重构。FiberNode 使用了双链表的结构，可以直接找到兄弟节点与子节点，使得整个更新过程可以随时暂停恢复。FiberTree 则是通过 FiberNode 构成的树。</p>
<p data-nodeid="29672">Fiber 机制下，整个更新过程由 current 与 workInProgress 两株树双缓冲完成。当 workInProgress 更新完成后，通过修改 current 相关指针指向的节点，直接抛弃老树，虽然非常简单粗暴，却非常合理。</p>
<p data-nodeid="29673">这些就是 React 中 diff 算法的回答要点，我们再来看看其他框架需要掌握什么。</p>
<h4 data-nodeid="29674">其他框架</h4>
<p data-nodeid="29675"><strong data-nodeid="29921">Preact</strong></p>
<p data-nodeid="29676">在众多的 React-like 框架中，<strong data-nodeid="29927">Preact 适用范围最广，生命力最强</strong>。它以仅 3kb 的小巧特点应用于对体积追求非常极致的场景。也正因为体积受限，Preact 在 diff 算法上做了裁剪。</p>
<p data-nodeid="29677">以下 Preact 的 diff 算法的图示，可以看到它将 diff 分为了三个类型：Fragment、Component 及 DOM Node。</p>
<p data-nodeid="29678"><img src="https://s0.lgstatic.com/i/image/M00/8C/4A/Ciqc1F_qyqeAWvpYAAEVYxYrQis686.png" alt="Drawing 6.png" data-nodeid="29931"></p>
<ul data-nodeid="29679">
<li data-nodeid="29680">
<p data-nodeid="29681">Fragment 对应 React 的树比较；</p>
</li>
<li data-nodeid="29682">
<p data-nodeid="29683">Component 对应组件比较，它们在原理上是相通的，所以这里我们不再赘述；</p>
</li>
<li data-nodeid="29684">
<p data-nodeid="29685">最大的不同在于 DOM Node 这一层，Preact 并没有 Patch 的过程，而是直接更新 DOM 节点属性。</p>
</li>
</ul>
<p data-nodeid="29686"><strong data-nodeid="29938">Vue</strong></p>
<p data-nodeid="29687">Vue 2.0 因为使用了 <a href="https://github.com/snabbdom/snabbdom/tree/8079ba78685b0f0e0e67891782c3e8fb9d54d5b8" data-nodeid="29942">snabbdom</a>，所以整体思路与 React 相同。但在元素对比时，如果新旧两个元素是同一个元素，且没有设置 key 时，snabbdom 在 diff 子元素中会一次性对比<strong data-nodeid="29960">旧节点</strong>、<strong data-nodeid="29961">新节点</strong>及它们的<strong data-nodeid="29962">首尾元素</strong>四个节点，以及<strong data-nodeid="29963">验证列表</strong>是否有变化。Vue 3.0 整体变化不大，依然没有引入 Fiber 等设计，也没有时间切片等功能。</p>
<h3 data-nodeid="29688">答题</h3>
<blockquote data-nodeid="29689">
<p data-nodeid="29690">在回答有何不同之前，首先需要说明下什么是 diff 算法。</p>
<p data-nodeid="29691">diff 算法是指生成更新补丁的方式，主要应用于虚拟 DOM 树变化后，更新真实 DOM。所以 diff 算法一定存在这样一个过程：触发更新 → 生成补丁 → 应用补丁。</p>
<p data-nodeid="29692">React 的 diff 算法，触发更新的时机主要在 state 变化与 hooks 调用之后。此时触发虚拟 DOM 树变更遍历，采用了深度优先遍历算法。但传统的遍历方式，效率较低。为了优化效率，使用了分治的方式。将单一节点比对转化为了 3 种类型节点的比对，分别是树、组件及元素，以此提升效率。</p>
<p data-nodeid="29693">树比对：由于网页视图中较少有跨层级节点移动，两株虚拟 DOM 树只对同一层次的节点进行比较。</p>
<p data-nodeid="29694">组件比对：如果组件是同一类型，则进行树比对，如果不是，则直接放入到补丁中。</p>
<p data-nodeid="29695">元素比对：主要发生在同层级中，通过标记节点操作生成补丁，节点操作对应真实的 DOM 剪裁操作。</p>
<p data-nodeid="29696">以上是经典的 React diff 算法内容。自 React 16 起，引入了 Fiber 架构。为了使整个更新过程可随时暂停恢复，节点与树分别采用了 FiberNode 与 FiberTree 进行重构。fiberNode 使用了双链表的结构，可以直接找到兄弟节点与子节点。</p>
<p data-nodeid="29697">整个更新过程由 current 与 workInProgress 两株树双缓冲完成。workInProgress 更新完成后，再通过修改 current 相关指针指向新节点。</p>
<p data-nodeid="29698">然后拿 Vue 和 Preact 与 React 的 diff 算法进行对比。</p>
<p data-nodeid="29699">Preact 的 Diff 算法相较于 React，整体设计思路相似，但最底层的元素采用了真实 DOM 对比操作，也没有采用 Fiber 设计。Vue 的 Diff 算法整体也与 React 相似，同样未实现 Fiber 设计。</p>
<p data-nodeid="29700">然后进行横向比较，React 拥有完整的 Diff 算法策略，且拥有随时中断更新的时间切片能力，在大批量节点更新的极端情况下，拥有更友好的交互体验。</p>
<p data-nodeid="29701">Preact 可以在一些对性能要求不高，仅需要渲染框架的简单场景下应用。</p>
<p data-nodeid="29702">Vue 的整体 diff 策略与 React 对齐，虽然缺乏时间切片能力，但这并不意味着 Vue 的性能更差，因为在 Vue 3 初期引入过，后期因为收益不高移除掉了。除了高帧率动画，在 Vue 中其他的场景几乎都可以使用防抖和节流去提高响应性能。</p>
</blockquote>
<p data-nodeid="29703"><img src="https://s0.lgstatic.com/i/image2/M01/04/31/CgpVE1_q2zGAe9UzAACKAZViwbM237.png" alt="Diff 算法1.png" data-nodeid="29980"><br>
这里需要注意的是：对比过程中切忌踩一捧一，容易引发面试官反感。</p>
<h3 data-nodeid="29704">进阶</h3>
<p data-nodeid="29705"><strong data-nodeid="29988">学习原理的目的就是应用。那如何根据 React diff 算法原理优化代码呢？</strong> 这个问题其实按优化方式逆向回答即可。</p>
<blockquote data-nodeid="29706">
<p data-nodeid="29707">根据 diff 算法的设计原则，应尽量避免跨层级节点移动。</p>
<p data-nodeid="29708">通过设置唯一 key 进行优化，尽量减少组件层级深度。因为过深的层级会加深遍历深度，带来性能问题。</p>
<p data-nodeid="29709">设置 shouldComponentUpdate 或者 React.pureComponet 减少 diff 次数。</p>
</blockquote>
<h3 data-nodeid="29710">总结</h3>
<p data-nodeid="29711">diff 算法一直在 React 中处于核心的位置，所以本讲讲到的内容，你一定要掌握。如果你有什么问题或者想法，欢迎留言与我互动。</p>
<p data-nodeid="29712">在下一讲，我会为你讲述 React 的渲染流程，下节课见！</p>
<hr data-nodeid="29713">
<p data-nodeid="29714"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="29999"><img src="https://s0.lgstatic.com/i/image/M00/72/94/Ciqc1F_EZ0eANc6tAASyC72ZqWw643.png" alt="Drawing 2.png" data-nodeid="29998"></a></p>
<p data-nodeid="29715">《大前端高薪训练营》</p>
<p data-nodeid="29716" class="te-preview-highlight">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="30004">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **贵：
> 这么好的文章，为啥评论交流的小伙伴这么少😂

##### **举：
> 很棒，总结的很到位，有个地方: 优化策略策略二中的，如果组件 class 一致，这里您说的 class 是指的虚拟 dom 的 ＄type 吧？我还以为是 CSS 样式类名呢。

##### **生：
> 每次看都有着不同的心得，这才是一个好的专栏，值得回味

##### **辉：
> 问下react 列表 这里diff 是如何判断出变化的？ 是通过兄弟指针么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不需要，列表的话直接按顺序对比即可，用 key 可以优化子节点及比对过程。
如果我解释的不够清楚，可以查看 React 官网文档（https://zh-hans.reactjs.org/docs/reconciliation.html）对子节点进行递归部分。

##### **清：
> O(N的3次）是怎么算

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 树的编辑距离算法，三言两语说不太清楚，可以看下这篇文章的介绍 https://www.zhihu.com/question/66851503/answer/246766239。
如果希望阅读原文的话，可以先看原文《Reconciliation》（https://reactjs.org/docs/reconciliation.html）中的 the state of the art algorithms have a complexity in the order of O(n3) where n is the number of elements in the tree.
这里的O(n3)出自论文《A Survey on Tree Edit Distance and Related
Problems》（https://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf）

##### *瑞：
> 您好，老师。想问一下这句话”策略二：如果组件的 class 一致，则默认为相似的树结构，否则默认为不同的树结构。“中的 class 是什么？虽然后面有过程讲解但是还是不太清楚。其中又提到”类型“，这里的类型是指什么？有哪些？用户申明的还是 React 内部定义的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的 class  是指 classComponent，这里的类型是指是否是同一种 classComponent。如果我有讲解不清的地方，欢迎继续留言，我会尽力回答。

##### **6400：
> 老师，为什么广度优先遍历可能会导致组件的生命周期时序错乱呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 广度优先是先横向遍历，但 React 是一层一层嵌套的，就像剥蒜一样。如果横向优先处理，就会先平行处理兄弟组件，内部的嵌套层级会延后。
生命周期的调度策略将会变得十分复杂，不再是一个递归策略就能解决的。但如果以后续的 fiber 或 React 17.x 的 lane 可能问题就那么大了。 
以上仅仅是我的个人见解，如果还有疑问的话，欢迎继续留言。

##### **杰：
> 很奇怪引入VDOM的思想不也是考虑到大批量节点操作更新的时候性能会比直接操作真实DOM要更好吗，那么基于这个思想为什么Vue3不引入Fiber时间切片更新机制呢，还有一个是现在Vue3使用了静态节点标记的方式，那这种方式是不是会比Fiber性能更加的好?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这一点尤雨溪回答过，他的结论是 Vue 3 的性能就是比 React 好，基于响应式的设计减少了大量复杂的运算，优化在框架层做了完美的解决。
具体到底谁的设计更好，个人感觉前端框架一直缺少一个公允通用的 benchmark 测试，缺少不服跑个分的实践基础。
以上仅是个人看法，在面试的时候，也可以适度表达一下个人的见解，但需要注意，切忌过度贬低某一技术栈。

##### **梁：
> 面试中遇到过这种问题确实

##### *伟：
> 对比的很好，节省了我们自己总结的时间了。

##### **举：
> 老师，那 React 使用策略进行分治法优化之后，为啥复杂度变成了 O(n),这个O(n) 又是如何计算的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在本文中有提到，主要通过三个方式实现：忽略跨层移动、只做同层比较，同层子节点用 key 比对。
这个 O(n) 是因为变成了线性一一比对而得来的。

##### **梁：
> vue3里面增加了静态节点的标记，性能上进一步提升。

##### *盼：
> 老师的思路太赞了

