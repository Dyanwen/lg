<p data-nodeid="56901">React 应用也是前端应用，如果之前你知道一些前端项目普适的性能优化手段，比如资源加载过程中的优化、减少重绘与回流、服务端渲染、启用 CDN 等，那么这些手段对于 React 来说也是同样奏效的。</p>
<p data-nodeid="56902">不过对于 React 项目来说，它有一个区别于传统前端项目的重要特点，就是<strong data-nodeid="57014">以 React 组件的形式来组织逻辑</strong>：组件允许我们将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。因此，除了前面所提到的普适的前端性能优化手段之外，React 还有一些充满了自身特色的性能优化思路，这些思路基本都围绕“组件性能优化”这个中心思想展开。本讲我将带你认识其中最关键的 3 个思路：</p>
<ol data-nodeid="56903">
<li data-nodeid="56904">
<p data-nodeid="56905"><strong data-nodeid="57018">使用 shouldComponentUpdate 规避冗余的更新逻辑</strong></p>
</li>
<li data-nodeid="56906">
<p data-nodeid="56907"><strong data-nodeid="57022">PureComponent + Immutable.js</strong></p>
</li>
<li data-nodeid="56908">
<p data-nodeid="56909"><strong data-nodeid="57026">React.memo 与 useMemo</strong></p>
</li>
</ol>
<blockquote data-nodeid="56910">
<p data-nodeid="56911">注：<strong data-nodeid="57032">这 3 个思路同时也是 React 面试中“性能优化”这一环的核心所在</strong>。大家在回答类似题目的时候，不管其他的细枝末节的优化策略能不能想起来，以上三点一定要尽量答全。</p>
</blockquote>
<h3 data-nodeid="56912">朴素思路：善用 shouldComponentUpdate</h3>
<p data-nodeid="56913">shouldComponentUpdate 是 React 类组件的一个生命周期。关于 shouldComponentUpdate 是什么，我们已经在第 02 讲有过介绍，这里先简单复习一下。</p>
<p data-nodeid="56914">shouldComponentUpdate 的调用形式如下：</p>
<pre class="lang-js" data-nodeid="60415"><code data-language="js">shouldComponentUpdate(nextProps, nextState)
</code></pre>

<p data-nodeid="56916">render 方法由于伴随着对虚拟 DOM 的构建和对比，过程可以说相当耗时。而在 React 当中，很多时候我们会不经意间就频繁地调用了 render。为了避免不必要的 render 操作带来的性能开销，React 提供了 shouldComponentUpdate 这个口子。<strong data-nodeid="57041">React 组件会根据 shouldComponentUpdate 的返回值，来决定是否执行该方法之后的生命周期，进而决定是否对组件进行 re-render（重渲染）</strong>。</p>
<p data-nodeid="58253" class="">shouldComponentUpdate 的默认值为 true，也就是说 <strong data-nodeid="58259">“无条件 re-render”</strong>。在实际的开发中，我们往往通过手动往 shouldComponentUpdate 中填充判定逻辑，来实现“有条件的 re-render”。</p>

<p data-nodeid="56918">接下来我们通过一个 Demo，来感受一下 shouldComponentUpdate 到底是如何解决问题的。在这个 Demo 中会涉及 3 个组件：子组件 ChildA、ChildB 及父组件 App 组件。</p>
<p data-nodeid="56919">首先我们来看两个子组件的代码，这里为了尽量简化与数据变更无关的逻辑，ChildA 和 ChildB 都只负责从父组件处读取数据并渲染，它们的编码分别如下所示。</p>
<p data-nodeid="56920">ChildA.js：</p>
<pre class="lang-js" data-nodeid="58798"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ChildA</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ChildA 的render方法执行了"</span>);
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"childA"</span>&gt;</span>
        子组件A的内容：
        {this.props.text}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>

<p data-nodeid="56922">ChildB.js：</p>
<pre class="lang-js" data-nodeid="59337"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ChildB</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ChildB 的render方法执行了"</span>);
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"childB"</span>&gt;</span>
        子组件B的内容：
        {this.props.text}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>

<p data-nodeid="56924">在共同的父组件 App.js 中，会将 ChildA 和 ChildB 组合起来，并分别向其中注入数据：</p>
<pre class="lang-js" data-nodeid="59876"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> ChildA <span class="hljs-keyword">from</span> <span class="hljs-string">'./ChildA'</span>
<span class="hljs-keyword">import</span> ChildB <span class="hljs-keyword">from</span> <span class="hljs-string">'./ChildB'</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">App</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  state = {
    <span class="hljs-attr">textA</span>: <span class="hljs-string">'我是A的文本'</span>,
    <span class="hljs-attr">textB</span>: <span class="hljs-string">'我是B的文本'</span>
  }
  changeA = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">textA</span>: <span class="hljs-string">'A的文本被修改了'</span>
    })
  }
  changeB = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">textB</span>: <span class="hljs-string">'B的文本被修改了'</span>
    })
  }
  render() {
    <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"container"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeA}</span>&gt;</span>点击修改A处的文本<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeB}</span>&gt;</span>点击修改B处的文本<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">ChildA</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{this.state.textA}/</span>&gt;</span>
          <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">ChildB</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{this.state.textB}/</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
  }
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> App;
</code></pre>

<p data-nodeid="56926">App 组件最终渲染到界面上的效果如下图所示，两个子组件在图中分别被不同颜色的标注圈出：</p>
<p data-nodeid="60954" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/D3/CgqCHl_ga3-ADPKZAACHTPJhWNw299.png" alt="Drawing 0.png" data-nodeid="60957"></p>

<p data-nodeid="56928">通过点击左右两个按钮，我们可以分别对 ChildA 和 ChildB 中的文案进行修改。</p>
<p data-nodeid="56929">由于初次渲染时，两个组件的 render 函数都必然会被触发，因此控制台在挂载完成后的输出内容如下图所示：</p>
<p data-nodeid="61496" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/AA/CgpVE1_ga4qAdOlsAAAzU_bU8eQ279.png" alt="Drawing 1.png" data-nodeid="61499"></p>

<p data-nodeid="56931">接下来我点击左侧的按钮，尝试对 A 处的文本进行修改。我们可以看到界面上只有 A 处的渲染效果发生了改变，如下图箭头处所示：</p>
<p data-nodeid="62038" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/A9/Cip5yF_ga5KALnO1AABLrDrgDGM452.png" alt="Drawing 2.png" data-nodeid="62041"></p>

<p data-nodeid="56933">但是如果我们打开控制台，会发现输出的内容如下图所示：</p>
<p data-nodeid="62580" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/A9/Cip5yF_ga5qABE7ZAABs-adr_7k107.png" alt="Drawing 3.png" data-nodeid="62583"></p>

<p data-nodeid="56935">这样的输出结果告诉我们，在刚刚的点击动作后，不仅 ChildA 的 re-render 被触发了，ChildB 的 re-render 也被触发了。</p>
<p data-nodeid="56936">在 React 中，<strong data-nodeid="57075">只要父组件发生了更新，那么所有的子组件都会被无条件更新</strong>。这就导致了 ChildB 的 props 尽管没有发生任何变化，它本身也没有任何需要被更新的点，却还是会走一遍更新流程。</p>
<blockquote data-nodeid="56937">
<p data-nodeid="56938">注：同样的情况也适用于组件自身的更新：当组件自身调用了 setState 后，那么不管 setState 前后的状态内容是否真正发生了变化，它都会去走一遍更新流程。</p>
</blockquote>
<p data-nodeid="56939">而在刚刚这个更新流程中，shouldComponentUpdate 函数没有被手动定义，因此它将返回“true”这个默认值。“true”则意味着对更新流程不作任何制止，也即所谓的“无条件 re-render”。在这种情况下，我们就可以考虑使用 shouldComponentUpdate 来对更新过程进行管控，避免没有意义的 re-render 发生。</p>
<p data-nodeid="56940">现在我们就可以为 ChildB 加装这样一段 shouldComponentUpdate 逻辑：</p>
<pre class="lang-js" data-nodeid="63122"><code data-language="js">shouldComponentUpdate(nextProps, nextState) {
  <span class="hljs-comment">// 判断 text 属性在父组件更新前后有没有发生变化，若没有发生变化，则返回 false</span>
  <span class="hljs-keyword">if</span>(nextProps.text === <span class="hljs-keyword">this</span>.props.text) {
    <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>
  }
  <span class="hljs-comment">// 只有在 text 属性值确实发生变化时，才允许更新进行下去</span>
  <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>
}
</code></pre>

<p data-nodeid="56942">在这段逻辑中，我们对 ChildB 中的可变数据，也就是 this.props.text 这个属性进行了判断。</p>
<p data-nodeid="56943">这样，当父组件 App 组件发生更新、进而试图触发 ChildB 的更新流程时，shouldComponentUpdate 就会充当一个“守门员”的角色：它会检查新下发的 props.text 是否和之前的值一致，如果一致，那么就没有更新的必要，直接返回“false”将整个 ChildB 的更新生命周期中断掉即可。只有当 props.text 确实发生变化时，它才会“准许” re-render 的发生。</p>
<p data-nodeid="56944">在 shouldComponentUpdate 的加持下，当我们再次点击左侧按钮，试图修改 ChildA 的渲染内容时，控制台的输出就会变成下图这样：</p>
<p data-nodeid="63661" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/AA/CgpVE1_ga6yAVvq5AABmBay34YA804.png" alt="Drawing 4.png" data-nodeid="63664"></p>

<p data-nodeid="56946">我们看到，控制台中现在只有 ChildA 的 re-render 提示。ChildB “稳如泰山”，成功躲开了一次多余的渲染。</p>
<p data-nodeid="56947">使用 shouldComponentUpdate 来调停不必要的更新，避免无意义的 re-render 发生，这是 React 组件中最基本的性能优化手段，也是最重要的手段。许多看似高级的玩法，都是基于 shouldComponentUpdate 衍生出来的。我们接下来要讲的 PureComponent，就是这类玩法中的典型。</p>
<h3 data-nodeid="56948">进阶玩法：PureComponent &nbsp;+ Immutable.js</h3>
<h4 data-nodeid="56949">PureComponent：提前帮你安排好更新判定逻辑</h4>
<p data-nodeid="56950">shouldComponentUpdate 虽然一定程度上帮我们解决了性能方面的问题，但每次避免 re-render，都要手动实现一次 shouldComponentUpdate，未免太累了。作为一个不喜欢重复劳动的前端开发者来说，在写了不计其数个 shouldComponentUpdate 逻辑之后，难免会怀疑人生，进而发出由衷的感叹——“这玩意儿要是能内置到组件里该多好啊！”。</p>
<p data-nodeid="65835" class="">哪里有需求，哪里就有产品。React 15.3 很明显听到了开发者的声音，它新增了一个叫 <a href="https://zh-hans.reactjs.org/docs/react-api.html#reactpurecomponent" data-nodeid="65839">PureComponent</a> 的类，恰到好处地解决了“程序员写 shouldComponentUpdate 写出腱鞘炎”这个问题。</p>




<p data-nodeid="56952">PureComponent 与 Component 的区别点，就在于它内置了对 shouldComponentUpdate 的实现：PureComponent 将会在 shouldComponentUpdate 中对组件更新前后的 props 和 state 进行<strong data-nodeid="57100">浅比较</strong>，并根据浅比较的结果，决定是否需要继续更新流程。</p>
<p data-nodeid="56953">“浅比较”将针对值类型数据对比其值是否相等，而针对数组、对象等引用类型的数据则对比其引用是否相等。</p>
<p data-nodeid="56954">在我们开篇的 Demo 中，若把 ChildB 的父类从 Component 替换为 PureComponent（修改后的代码如下所示），那么无须手动编写 shouldComponentUpdate，也可以达到同样避免 re-render 的目的。</p>
<pre class="lang-js" data-nodeid="66379"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ChildB</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">PureComponent</span> </span>{
  render() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ChildB 的render方法执行了"</span>);
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"childB"</span>&gt;</span>
        子组件B的内容：
        {this.props.text}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>

<p data-nodeid="56956">此时再去修改 ChildA 中的文本，我们会发现 ChildB 同样不受影响。点击左侧按钮后，控制台对应的输出内容如下图高亮处所示：</p>
<p data-nodeid="66918" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/A9/Cip5yF_ga8qADhf9AACUfTqE0ag890.png" alt="Drawing 5.png" data-nodeid="66921"></p>

<p data-nodeid="56958">在值类型数据这种场景下，PureComponent 可以说是战无不胜。但是如果数据类型为引用类型，那么这种基于浅比较的判断逻辑就会带来这样两个风险：</p>
<ol data-nodeid="56959">
<li data-nodeid="56960">
<p data-nodeid="56961">若数据内容没变，但是引用变了，那么浅比较仍然会认为“数据发生了变化”，进而触发一次不必要的更新，导致过度渲染；</p>
</li>
<li data-nodeid="56962">
<p data-nodeid="56963">若数据内容变了，但是引用没变，那么浅比较则会认为“数据没有发生变化”，进而阻断一次更新，导致不渲染。</p>
</li>
</ol>
<p data-nodeid="56964">怎么办呢？Immutable.js 来帮忙！</p>
<h4 data-nodeid="56965">Immutable：“不可变值”让“变化”无处遁形</h4>
<p data-nodeid="56966">PureComponent 浅比较带来的问题，本质上是对“变化”的判断不够精准导致的。那有没有一种办法，能够让引用的变化和内容的变化之间，建立一种必然的联系呢？</p>
<p data-nodeid="56967">这就是 Immutable.js 所做的事情。</p>
<p data-nodeid="56968">Immutable 直译过来是“不可变的”，顾名思义，Immutable.js 是对“不可变值”这一思想的贯彻实践。它在 2014 年被 Facebook 团队推出，Facebook 给它的定位是“实现持久性数据结构的库”。<strong data-nodeid="57119">所谓“持久性数据”，指的是这个数据只要被创建出来了，就不能被更改。我们对当前数据的任何修改动作，都会导致一个新的对象的返回</strong>。这就将数据内容的变化和数据的引用严格地关联了起来，使得“变化”无处遁形。</p>
<p data-nodeid="56969">这里我用一个简单的例子，来演示一下 Immutable.js 的效果。请看下面代码：</p>
<pre class="lang-java" data-nodeid="56970"><code data-language="java"><span class="hljs-comment">// 引入 immutable 库里的 Map 对象，它用于创建对象</span>
<span class="hljs-keyword">import</span> { Map } from <span class="hljs-string">'immutable'</span>
<span class="hljs-comment">// 初始化一个对象 baseMap</span>
<span class="hljs-keyword">const</span> baseMap = Map({
  name: <span class="hljs-string">'修言'</span>,
  career: <span class="hljs-string">'前端'</span>,
  age: <span class="hljs-number">99</span>
})
<span class="hljs-comment">// 使用 immutable 暴露的 Api 来修改 baseMap 的内容</span>
<span class="hljs-keyword">const</span> changedMap = baseMap.set({
  age: <span class="hljs-number">100</span>
})
<span class="hljs-comment">// 我们会发现修改 baseMap 后将会返回一个新的对象，这个对象的引用和 baseMap 是不同的</span>
console.log(<span class="hljs-string">'baseMap === changedMap'</span>, baseMap === changedMap)
</code></pre>
<p data-nodeid="56971">由此可见，PureComonent 和 Immutable.js 真是一对好基友！在实际的开发中，我们也确实经常左手 PureComonent，右手 Immutable.js，研发质量大大地提升呀！</p>
<blockquote data-nodeid="56972">
<p data-nodeid="56973">值得注意的是，由于 Immutable.js 存在一定的学习成本，并不是所有场景下都可以作为最优解被团队采纳。因此，一些团队也会基于 PureComonent 和 Immutable.js 去打造将两者结合的公共类，通过改写 setState 来提升研发体验，这也是不错的思路。</p>
</blockquote>
<h3 data-nodeid="56974">函数组件的性能优化：React.memo 和 useMemo</h3>
<p data-nodeid="56975">以上咱们讨论的都是类组件的优化思路。那么在函数组件中，有没有什么通用的手段可以阻止“过度 re-render”的发生呢？接下来我们就一起认识一下“函数版”的 shouldComponentUpdate/Purecomponent —— React.memo。</p>
<h4 data-nodeid="56976">React.memo：“函数版”shouldComponentUpdate/PureComponent</h4>
<p data-nodeid="56977">React.memo 是 React 导出的一个顶层函数，它本质上是一个高阶组件，负责对函数组件进行包装。基本的调用姿势如下面代码所示：</p>
<pre class="lang-java" data-nodeid="56978"><code data-language="java"><span class="hljs-keyword">import</span> React from <span class="hljs-string">"react"</span>;
<span class="hljs-comment">// 定义一个函数组件</span>
<span class="hljs-function">function <span class="hljs-title">FunctionDemo</span><span class="hljs-params">(props)</span> </span>{
  <span class="hljs-keyword">return</span> xxx
}
<span class="hljs-comment">// areEqual 函数是 memo 的第二个入参，我们之前放在 shouldComponentUpdate 里面的逻辑就可以转移至此处</span>
<span class="hljs-function">function <span class="hljs-title">areEqual</span><span class="hljs-params">(prevProps, nextProps)</span> </span>{
  <span class="hljs-comment">/*
  return true if passing nextProps to render would return
  the same result as passing prevProps to render,
  otherwise return false
  */</span>
}
<span class="hljs-comment">// 使用 React.memo 来包装函数组件</span>
export <span class="hljs-keyword">default</span> React.memo(FunctionDemo, areEqual);
</code></pre>
<p data-nodeid="56979"><strong data-nodeid="57131">React.memo 会帮我们“记住”函数组件的渲染结果，在组件前后两次 props 对比结果一致的情况下，它会直接复用最近一次渲染的结果</strong>。如果我们的组件在相同的 props 下会渲染相同的结果，那么使用 React.memo 来包装它将是个不错的选择。</p>
<p data-nodeid="56980">从示例中我们可以看出，React.memo 接收两个参数，第一个参数是我们需要渲染的目标组件，第二个参数 areEqual 则用来承接 props 的对比逻辑。<strong data-nodeid="57137">之前我们在 shouldComponentUpdate 里面做的事情，现在就可以放在 areEqual 里来做</strong>。</p>
<p data-nodeid="56981">比如开篇 Demo 中的 ChildB 组件，就完全可以用 Function Component + React.memo 来改造。改造后的 ChildB 代码如下：</p>
<pre class="lang-js" data-nodeid="67460"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-comment">// 将 ChildB 改写为 function 组件</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ChildB</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ChildB 的render 逻辑执行了"</span>);
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"childB"</span>&gt;</span>
      子组件B的内容：
      {props.text}
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
<span class="hljs-comment">// areEqual 用于对比 props 的变化</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">areEqual</span>(<span class="hljs-params">prevProps, nextProps</span>) </span>{
  <span class="hljs-keyword">if</span>(prevProps.text === nextProps.text) {
    <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>
  }
  <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>
}
<span class="hljs-comment">// 使用 React.memo 来包装 ChildB</span>
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> React.memo(ChildB, areEqual);
</code></pre>

<p data-nodeid="56983">改造后的组件在效果上就等价于 shouldComponentUpdate 加持后的类组件 ChildB。</p>
<p data-nodeid="56984"><strong data-nodeid="57144">这里的 areEqual 函数是一个可选参数，当我们不传入 areEqual 时，React.memo 也可以工作，此时它的作用就类似于 PureComponent——React.memo 会自动为你的组件执行 props 的浅比较逻辑</strong>。</p>
<p data-nodeid="56985">和 shouldComponentUpdate 不同的是，React.memo 只负责对比 props，而不会去感知组件内部状态（state）的变化。</p>
<h4 data-nodeid="56986">useMemo：更加“精细”的 memo</h4>
<p data-nodeid="56987">通过上面的分析我们知道，React.memo 可以实现类似于 shouldComponentUpdate 或者 PureComponent 的效果，对组件级别的 re-render 进行管控。但是有时候，我们希望复用的并不是整个组件，而是组件中的某一个或几个部分。这种更加“精细化”的管控，就需要 useMemo 来帮忙了。</p>
<p data-nodeid="56988"><strong data-nodeid="57152">简而言之，React.memo 控制是否需要重渲染一个组件，而 useMemo 控制的则是是否需要重复执行某一段逻辑</strong>。</p>
<p data-nodeid="56989">useMemo 的使用方式如下面代码所示：</p>
<pre class="lang-java" data-nodeid="56990"><code data-language="java"><span class="hljs-keyword">const</span> memoizedValue = useMemo(() =&gt; computeExpensiveValue(a, b), [a, b]);
</code></pre>
<p data-nodeid="56991">我们可以把目标逻辑作为第一个参数传入，把逻辑的依赖项数组作为第二个参数传入。这样只有当依赖项数组中的某个依赖发生变化时，useMemo 才会重新执行第一个入参中的目标逻辑。</p>
<p data-nodeid="56992" class="">这里我仍然以开篇的示例为例，现在我尝试向 ChildB 中传入两个属性：text 和 count，它们分别是一段文本和一个数字。当我点击右边的按钮时，只有 count 数字会发生变化。改造后的 App 组件代码如下：</p>
<pre class="lang-js" data-nodeid="67999"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">App</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  state = {
    <span class="hljs-attr">textA</span>: <span class="hljs-string">'我是A的文本'</span>,
    <span class="hljs-attr">stateB</span>: {
      <span class="hljs-attr">text</span>: <span class="hljs-string">'我是B的文本'</span>,
      <span class="hljs-attr">count</span>: <span class="hljs-number">10</span>
    }
  }
  changeA = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">textA</span>: <span class="hljs-string">'A的文本被修改了'</span>
    })
  }
  changeB = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">stateB</span>: {
        ...this.state.stateB,
        <span class="hljs-attr">count</span>: <span class="hljs-number">100</span>
      }
    })
  }
  render() {
    <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"container"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeA}</span>&gt;</span>点击修改A处的文本<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeB}</span>&gt;</span>点击修改B处的文本<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">ChildA</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{this.state.textA}/</span>&gt;</span>
          <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
          <span class="hljs-tag">&lt;<span class="hljs-name">ChildB</span> {<span class="hljs-attr">...this.state.stateB</span>}/&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
  }
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> App;
</code></pre>

<p data-nodeid="56994">在 ChildB 中，使用 useMemo 来加持 text 和 count 各自的渲染逻辑。改造后的 ChildB 代码如下所示：</p>
<pre class="lang-js" data-nodeid="68538"><code data-language="js"><span class="hljs-keyword">import</span> React,{ useMemo } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ChildB</span>(<span class="hljs-params">{text, count}</span>) </span>{
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"ChildB 的render 逻辑执行了"</span>);
  <span class="hljs-comment">// text 文本的渲染逻辑</span>
  <span class="hljs-keyword">const</span> renderText = <span class="hljs-function">(<span class="hljs-params">text</span>)=&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'renderText 执行了'</span>)
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>
    子组件B的文本内容：
      {text}
  <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  }
  <span class="hljs-comment">// count 数字的渲染逻辑</span>
  <span class="hljs-keyword">const</span> renderCount = <span class="hljs-function">(<span class="hljs-params">count</span>) =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'renderCount 执行了'</span>)
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>
      子组件B的数字内容：
        {count}
    <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  }
  
  <span class="hljs-comment">// 使用 useMemo 加持两段渲染逻辑</span>
  <span class="hljs-keyword">const</span> textContent = useMemo(<span class="hljs-function"><span class="hljs-params">()</span>=&gt;</span>renderText(text),[text])
  <span class="hljs-keyword">const</span> countContent = useMemo(<span class="hljs-function"><span class="hljs-params">()</span>=&gt;</span>renderCount(count),[count])
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"childB"</span>&gt;</span>
      {textContent}
      {countContent}
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>

<p data-nodeid="56996">渲染 App 组件，我们可以看到初次渲染时，renderText 和 renderCount 都执行了，控制台输出如下图所示：</p>
<p data-nodeid="69077" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/D3/CgqCHl_ga_SAeZvVAACbMQxPKsc444.png" alt="Drawing 6.png" data-nodeid="69080"></p>

<p data-nodeid="56998">点击右边按钮，对 count 进行修改，修改后的界面会发生如下的变化：</p>
<p data-nodeid="69619" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/03/AA/CgpVE1_ga_yAZ5u-AADTkxhPMO8352.png" alt="Drawing 7.png" data-nodeid="69622"></p>

<p data-nodeid="57000">可以看出，由于 count 发生了变化，因此 useMemo 针对 renderCount 的逻辑进行了重计算。而 text 没有发生变化，因此 renderText 的逻辑压根没有执行。</p>
<p data-nodeid="57001">使用 useMemo，我们可以对函数组件的执行逻辑进行更加细粒度的管控（尤其是定向规避掉一些高开销的计算），同时也弥补了 React.memo 无法感知函数内部状态的遗憾，这对我们整体的性能提升是大有裨益的。</p>
<h3 data-nodeid="57002">总结</h3>
<p data-nodeid="57003">本讲，我们学习了 React 组件性能优化中最重要的 3 个思路。</p>
<p data-nodeid="57004">这 3 个思路不仅可以作为大家日常实战的知识储备，更能够帮助你在面试场景下做到言之有物。事实上，在“React 性能优化”这个问题下，许多候选人的回答犹如隔靴搔痒，总在一些无关紧要的细节上使劲儿。若你能把握好本讲的内容，择其中一个或多个方向深入探究，相信你已经超越了大部分的同行。</p>
<p data-nodeid="57005">下一讲，我们将学习 React 组件的设计模式，为打造“高质量应用”做知识储备。</p>

---

### 精选评论

##### **宇：
> memo后的reactdom虽然不会再次通过函数执行了，但它最终还是汇入了父组件的children里面。那么它会跟随变化后的父组件一起参与diff吗？react有没有针对memo的数据做一些特殊标记，不然还是会diff到还是有点浪费性能。

##### **杰：
> 这一节讲的太清楚了，之前一直对这一块的知识云里雾里的，感谢作者！作者有没有技术博客之类的，可以关注下吗

##### **6044：
> 看了这一章，受益颇多😬

##### **辉：
> usecallback也是一种优化手段

##### **5985：
> 打卡

##### **龙：
> 打卡

