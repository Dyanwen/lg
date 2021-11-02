<p data-nodeid="1389" class="">React 生命周期已经是一个老生常谈的话题了，几乎没有哪一门 React 入门教材会省略对组件生命周期的介绍。然而，入门教材在设计上往往追求的是“简单省事、迅速上手”，这就导致许多同学对于生命周期知识的刻板印象为“背就完了、别想太多”。</p>
<p data-nodeid="1390">“背就完了”这样简单粗暴的学习方式，或许可以帮助你理解“What to do”，到达“How to do”，但却不能帮助你去思考和认知“Why to do”。作为一个专业的 React 开发者，我们必须要求自己在知其然的基础上，知其所以然。</p>
<p data-nodeid="1391">在本课时和下一个课时，我将抱着帮你做到“知其所以然”的目的，以 React 的基本原理为引子，<strong data-nodeid="1503">对 React 15、React 16 两个版本的生命周期进行探讨、比对和总结，通过搞清楚一个又一个的“Why”，来帮你建立系统而完善的生命周期知识体系</strong>。</p>
<h3 data-nodeid="1392">生命周期背后的设计思想：把握 React 中的“大方向”</h3>
<p data-nodeid="1393">在介绍具体的生命周期之前，我想先带你初步理解 React 框架中的一些关键的设计思想，以便为你后续的学习提供不可或缺的“加速度”。</p>
<p data-nodeid="1394">如果你经常翻阅 React 官网或者 React 官方的一些文章，你会发现“<strong data-nodeid="1515">组件</strong>”和“<strong data-nodeid="1516">虚拟 DOM</strong>”这两个词的出镜率是非常高的，它们是 React 基本原理中极为关键的两个概念，也是我们这个小节的学习切入点。</p>
<h4 data-nodeid="1395">虚拟 DOM：核心算法的基石</h4>
<p data-nodeid="1396">通过 01 课时的学习，你已经知晓了虚拟 DOM 节点的基本形态，现在我们需要简单了解下虚拟 DOM 在整个 React 工作流中的作用。</p>
<p data-nodeid="1397">组件在初始化时，会通过调用生命周期中的 render 方法，<strong data-nodeid="1524">生成虚拟 DOM</strong>，然后再通过调用 ReactDOM.render 方法，实现虚拟 DOM 到真实 DOM 的转换。</p>
<p data-nodeid="1398">当组件更新时，会再次通过调用 render 方法<strong data-nodeid="1534">生成新的虚拟 DOM</strong>，然后借助 diff（这是一个非常关键的算法，我将在“模块二：核心原理”重点讲解）<strong data-nodeid="1535">定位出两次虚拟 DOM 的差异</strong>，从而针对发生变化的真实 DOM 作定向更新。</p>
<p data-nodeid="1399">以上就是 React 框架核心算法的大致流程。对于这套关键的工作流来说，“虚拟 DOM”是所有操作的大前提，是核心算法的基石。</p>
<h4 data-nodeid="1400">组件化：工程化思想在框架中的落地</h4>
<p data-nodeid="1401">组件化是一种优秀的软件设计思想，也是 React 团队在研发效能方面所做的一个重要的努力。</p>
<p data-nodeid="1402">在一个 React 项目中，几乎所有的可见/不可见的内容都可以被抽离为各种各样的组件，每个组件既是“封闭”的，也是“开放”的。</p>
<p data-nodeid="1403">所谓“封闭”，主要是针对“渲染工作流”（指从<strong data-nodeid="1549">组件数据改变</strong>到<strong data-nodeid="1550">组件实际更新发生的</strong>过程）来说的。在组件自身的渲染工作流中，每个组件都只处理它内部的渲染逻辑。在没有数据流交互的情况下，组件与组件之间可以做到“各自为政”。</p>
<p data-nodeid="1404">而所谓“开放”，则是针对组件间通信来说的。React 允许开发者基于“单向数据流”的原则完成组件间的通信。而组件之间的通信又将改变通信双方/某一方内部的数据，进而对渲染结果构成影响。所以说在数据这个“红娘”的牵线搭桥之下，组件之间又是彼此开放的，是可以相互影响的。</p>
<p data-nodeid="1405">这一“开放”与“封闭”兼具的特性，使得 React 组件<strong data-nodeid="1557">既专注又灵活</strong>，具备高度的可重用性和可维护性。</p>
<h4 data-nodeid="1406">生命周期方法的本质：组件的“灵魂”与“躯干”</h4>
<p data-nodeid="1407">之前我曾经在社区读过一篇文章，文中将 render 方法形容为 React 组件的“灵魂”。当时我对这句话产生了非常强烈的共鸣，这里我就想以这个曾经打动过我的比喻为引子，帮助你从宏观上建立对 React 生命周期的感性认知。</p>
<p data-nodeid="1408">注意，这里提到的 render 方法，和我们 01 课时所说的 ReactDOM.render 可不是一个东西，它指的是 React 组件内部的这个生命周期方法：</p>
<pre class="lang-js" data-nodeid="1409"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LifeCycle</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{

  render() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"render方法执行"</span>);
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"container"</span>&gt;</span>
        this is content
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="1410">前面咱们介绍了虚拟 DOM、组件化，倘若把这两块知识整合一下，你就会发现这两个概念似乎都在围着 render 这个生命周期打转：虚拟 DOM 自然不必多说，它的生成都要仰仗 render；而组件化概念中所提及的“渲染工作流”，这里指的是从<strong data-nodeid="1570">组件数据改变</strong>到<strong data-nodeid="1571">组件实际更新发生的</strong>过程，这个过程的实现同样离不开 render。</p>
<p data-nodeid="1411">由此看来，render 方法在整个组件生命周期中确实举足轻重，它担得起“灵魂”这个有分量的比喻。那么如果将 render 方法比作组件的“<strong data-nodeid="1581">灵魂</strong>”，render 之外的生命周期方法就完全可以理解为是组件的“<strong data-nodeid="1582">躯干</strong>”。</p>
<p data-nodeid="1412">“躯干”未必总是会做具体的事情（比如说我们可以选择性地省略对 render 之外的任何生命周期方法内容的编写），而“灵魂”却总是充实的（render 函数却坚决不能省略）；倘若“躯干”做了点什么，往往都会直接或间接地影响到“灵魂”（因为即便是 render 之外的生命周期逻辑，也大部分是在为 render 层面的效果服务）；“躯干”和“灵魂”一起，共同构成了 React 组件完整而不可分割的“生命时间轴”。</p>
<h3 data-nodeid="1413">拆解 React 生命周期：从 React 15 说起</h3>
<p data-nodeid="1414">我发现时下许多资料在讲解 React 生命周期时，喜欢直接拿 React 16 开刀。这样做虽然省事儿，却也模糊掉了新老生命周期变化背后的“Why”（关于两者的差异，我们会在“03 课时”中详细讲解）。这里为了把这个“Why”拎出来，我将首先带你认识 React 15 的生命周期流程。</p>
<p data-nodeid="1415">在 React 15 中，大家需要关注以下几个生命周期方法：</p>
<pre class="lang-java" data-nodeid="1416"><code data-language="java">constructor()
componentWillReceiveProps()
shouldComponentUpdate()
componentWillMount()
componentWillUpdate()
componentDidUpdate()
componentDidMount()
render()
componentWillUnmount()
</code></pre>
<blockquote data-nodeid="1417">
<p data-nodeid="1418">如果你接触 React 足够早，或许会记得还有 getDefaultProps 和 getInitState 这两个方法，它们都是 React.createClass() 模式下初始化数据的方法。由于这种写法在 ES6 普及后已经不常见，这里不再详细展开。</p>
</blockquote>
<p data-nodeid="1419">这些生命周期方法是如何彼此串联、相互依存的呢？这里我为你总结了一张大图：</p>
<p data-nodeid="1420"><img src="https://s0.lgstatic.com/i/image/M00/5E/31/Ciqc1F-GZbGAGNcBAAE775qohj8453.png" alt="1.png" data-nodeid="1591"></p>
<p data-nodeid="1421">接下来，我就围绕这张大图，分阶段讨论组件生命周期的运作规律。在学习的过程中，下面这个 Demo 可以帮助你具体地验证每个阶段的工作流程：</p>
<pre class="lang-js te-preview-highlight" data-nodeid="7991"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> ReactDOM <span class="hljs-keyword">from</span> <span class="hljs-string">"react-dom"</span>;
<span class="hljs-comment">// 定义子组件</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LifeCycle</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-keyword">constructor</span>(props) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"进入constructor"</span>);
    <span class="hljs-keyword">super</span>(props);
    <span class="hljs-comment">// state 可以在 constructor 里初始化</span>
    <span class="hljs-keyword">this</span>.state = { <span class="hljs-attr">text</span>: <span class="hljs-string">"子组件的文本"</span> };
  }
  <span class="hljs-comment">// 初始化渲染时调用</span>
  componentWillMount() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"componentWillMount方法执行"</span>);
  }
  <span class="hljs-comment">// 初始化渲染时调用</span>
  componentDidMount() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"componentDidMount方法执行"</span>);
  }
  <span class="hljs-comment">// 父组件修改组件的props时会调用</span>
  componentWillReceiveProps(nextProps) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"componentWillReceiveProps方法执行"</span>);
  }
  <span class="hljs-comment">// 组件更新时调用</span>
  shouldComponentUpdate(nextProps, nextState) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"shouldComponentUpdate方法执行"</span>);
    <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
  }

  <span class="hljs-comment">// 组件更新时调用</span>
  componentWillUpdate(nextProps, nextState) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"componentWillUpdate方法执行"</span>);
  }
  <span class="hljs-comment">// 组件更新后调用</span>
  componentDidUpdate(preProps, preState) {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"componentDidUpdate方法执行"</span>);
  }
  <span class="hljs-comment">// 组件卸载时调用</span>
  componentWillUnmount() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"子组件的componentWillUnmount方法执行"</span>);
  }
  <span class="hljs-comment">// 点击按钮，修改子组件文本内容的方法</span>
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: <span class="hljs-string">"修改后的子组件文本"</span>
    });
  };
  render() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"render方法执行"</span>);
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"container"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"changeText"</span>&gt;</span>
          修改子组件文本内容
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"textContent"</span>&gt;</span>{this.state.text}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"fatherContent"</span>&gt;</span>{this.props.text}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
<span class="hljs-comment">// 定义 LifeCycle 组件的父组件</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LifeCycleContainer</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{

  <span class="hljs-comment">// state 也可以像这样用属性声明的形式初始化</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"父组件的文本"</span>,
    <span class="hljs-attr">hideChild</span>: <span class="hljs-literal">false</span>
  };
  <span class="hljs-comment">// 点击按钮，修改父组件文本的方法</span>
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: <span class="hljs-string">"修改后的父组件文本"</span>
    });
  };
  <span class="hljs-comment">// 点击按钮，隐藏（卸载）LifeCycle 组件的方法</span>
  hideChild = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">hideChild</span>: <span class="hljs-literal">true</span>
    });
  };
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"fatherContainer"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"changeText"</span>&gt;</span>
          修改父组件文本内容
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.hideChild}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"hideChild"</span>&gt;</span>
          隐藏子组件
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        {this.state.hideChild ? null : <span class="hljs-tag">&lt;<span class="hljs-name">LifeCycle</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{this.state.text}</span> /&gt;</span>}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
ReactDOM.render(<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">LifeCycleContainer</span> /&gt;</span></span>, <span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">"root"</span>));
</code></pre>










<p data-nodeid="1423">该入口文件对应的 index.html 中预置了 id 为 root 的真实 DOM 节点作为根节点，body 标签内容如下：</p>
<pre class="lang-js" data-nodeid="1424"><code data-language="js">&lt;body&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"root"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/body&gt;
</code></pre>
<p data-nodeid="1425">这个 Demo 渲染到浏览器上大概是这样的：</p>
<p data-nodeid="1426"><img src="https://s0.lgstatic.com/i/image/M00/5D/CC/Ciqc1F-FU-yAMLh0AABeqOeqLek815.png" alt="Drawing 1.png" data-nodeid="1597"></p>
<p data-nodeid="1427">此处由于我们强调的是对生命周期执行规律的验证，所以样式上从简，你也可以根据自己的喜好添加 CSS 相关的内容。</p>
<p data-nodeid="1428">接下来我们就结合这个 Demo 和开头的生命周期大图，一起来看看挂载、更新、卸载这 3 个阶段，React 组件都经历了哪些事情。</p>
<h4 data-nodeid="1429">Mounting 阶段：组件的初始化渲染（挂载）</h4>
<p data-nodeid="1430">挂载过程在组件的一生中仅会发生一次，在这个过程中，组件被初始化，然后会被渲染到真实 DOM 里，完成所谓的“首次渲染”。</p>
<p data-nodeid="1431">在挂载阶段，一个 React 组件会按照顺序经历如下图所示的生命周期：</p>
<p data-nodeid="1432"><img src="https://s0.lgstatic.com/i/image/M00/5E/32/Ciqc1F-GZ1OAWETTAAA3Am2CwU0383.png" alt="3.png" data-nodeid="1605"></p>
<p data-nodeid="1433">首先我们来看 constructor 方法，该方法仅仅在挂载的时候被调用一次，我们可以在该方法中对 this.state 进行初始化：</p>
<pre class="lang-java" data-nodeid="1434"><code data-language="java">constructor(props) {
  console.log(<span class="hljs-string">"进入constructor"</span>);
  <span class="hljs-keyword">super</span>(props);
  <span class="hljs-comment">// state 可以在 constructor 里初始化</span>
  <span class="hljs-keyword">this</span>.state = { text: <span class="hljs-string">"子组件的文本"</span> };
}
</code></pre>
<p data-nodeid="1435">componentWillMount、componentDidMount 方法同样只会在挂载阶段被调用一次。其中 componentWillMount 会在执行 render 方法前被触发，一些同学习惯在这个方法里做一些初始化的操作，但这些操作往往会伴随一些风险或者说不必要性（这一点大家先建立认知，具体原因将在“03 课时”展开讲解）。</p>
<p data-nodeid="1436">接下来 render 方法被触发。注意 render 在执行过程中并不会去操作真实 DOM（也就是说不会渲染），它的职能是<strong data-nodeid="1613">把需要渲染的内容返回出来</strong>。真实 DOM 的渲染工作，在挂载阶段是由 ReactDOM.render 来承接的。</p>
<p data-nodeid="1437">componentDidMount 方法在渲染结束后被触发，此时因为真实 DOM 已经挂载到了页面上，我们可以在这个生命周期里执行真实 DOM 相关的操作。此外，类似于异步请求、数据初始化这样的操作也大可以放在这个生命周期来做（侧面印证了 componentWillMount 真的很鸡肋）。</p>
<p data-nodeid="1438">这一整个流程对应的其实就是我们 Demo 页面刚刚打开时，组件完成初始化渲染的过程。下图是 Demo 中的 LifeCycle 组件在挂载过程中控制台的输出，你可以用它来验证挂载过程中生命周期顺序的正确性：</p>
<p data-nodeid="1439"><img src="https://s0.lgstatic.com/i/image/M00/5D/D8/CgqCHl-FU_6AeWUcAAB8X4bjwqE102.png" alt="Drawing 3.png" data-nodeid="1618"></p>
<h4 data-nodeid="1440">Updating 阶段：组件的更新</h4>
<p data-nodeid="1441">组件的更新分为两种：一种是由父组件更新触发的更新；另一种是组件自身调用自己的 setState 触发的更新。这两种更新对应的生命周期流程如下图所示：</p>
<p data-nodeid="1442"><img src="https://s0.lgstatic.com/i/image/M00/5E/3C/CgqCHl-GZf-AUjsLAACmOsiQl3M485.png" alt="2.png" data-nodeid="1623"></p>
<p data-nodeid="1443"><strong data-nodeid="1627">componentWillReceiProps 到底是由什么触发的？</strong></p>
<p data-nodeid="1444">从图中你可以明显看出，父组件触发的更新和组件自身的更新相比，多出了这样一个生命周期方法：</p>
<pre class="lang-java" data-nodeid="1445"><code data-language="java">componentWillReceiveProps(nextProps)
</code></pre>
<p data-nodeid="1446">在这个生命周期方法里，nextProps 表示的是接收到新 props 内容，而现有的 props （相对于 nextProps 的“旧 props”）我们可以通过 this.props 拿到，由此便能够感知到 props 的变化。</p>
<p data-nodeid="1447">写到这里，就不得不在“变化”这个动作上深挖一下了。我在一些社区文章里，包括一些候选人面试时的回答里，都不约而同地见过/听过这样一种说法：<strong data-nodeid="1634">componentWillReceiveProps 是在组件的 props 内容发生了变化时被触发的。</strong></p>
<p data-nodeid="1448"><strong data-nodeid="1639">这种说法不够严谨</strong>。远的不说，就拿咱们上文给出的 Demo 开刀，该界面的控制台输出在初始化完成后是这样的：</p>
<p data-nodeid="1449"><img src="https://s0.lgstatic.com/i/image/M00/5D/CC/Ciqc1F-FVA6AYiD4AADSl2lr-_Q663.png" alt="Drawing 5.png" data-nodeid="1642"></p>
<p data-nodeid="1450">注意，我们代码里面，LifeCycleContainer 这个父组件传递给子组件 LifeCycle 的 props 只有一个 text：</p>
<pre class="lang-java" data-nodeid="1451"><code data-language="java">&lt;LifeCycle text={<span class="hljs-keyword">this</span>.state.text} /&gt;
</code></pre>
<p data-nodeid="1452">假如我点击“修改父组件文本内容”这个按钮，父组件的 this.state.text 会发生改变，进而带动子组件的 this.props.text 发生改变。此时一定会触发 componentWillReceiveProps 这个生命周期，这是毋庸置疑的：</p>
<p data-nodeid="1453"><img src="https://s0.lgstatic.com/i/image/M00/5D/CC/Ciqc1F-FVBWAEqTGAAEdsvX2TAM747.png" alt="Drawing 6.png" data-nodeid="1647"></p>
<p data-nodeid="1454">但如果我现在对父组件的结构进行一个小小的修改，给它一个和子组件完全无关的 state（this.state.ownText），同时相应地给到一个修改这个 state 的方法（this.changeOwnText），并用一个新的 button 按钮来承接这个触发的动作。</p>
<p data-nodeid="1455">改变后的 LifeCycleContainer 如下所示：</p>
<pre class="lang-js" data-nodeid="1456"><code data-language="js"><span class="hljs-comment">// 定义 LifeCycle 组件的父组件</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LifeCycleContainer</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// state 也可以像这样用属性声明的形式初始化</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"父组件的文本"</span>,
    <span class="hljs-comment">// 新增的只与父组件有关的 state</span>
    <span class="hljs-attr">ownText</span>: <span class="hljs-string">"仅仅和父组件有关的文本"</span>,
    <span class="hljs-attr">hideChild</span>: <span class="hljs-literal">false</span>
  };
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: <span class="hljs-string">"修改后的父组件文本"</span>
    });
  };
  <span class="hljs-comment">// 修改 ownText 的方法</span>
  changeOwnText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">ownText</span>: <span class="hljs-string">"修改后的父组件自有文本"</span>
    });
  };
  hideChild = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">hideChild</span>: <span class="hljs-literal">true</span>
    });
  };
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"fatherContainer"</span>&gt;</span>
        {/* 新的button按钮 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeOwnText}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"changeText"</span>&gt;</span>
          修改父组件自有文本内容
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"changeText"</span>&gt;</span>
          修改父组件文本内容
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.hideChild}</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"hideChild"</span>&gt;</span>
          隐藏子组件
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span> {this.state.ownText} <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        {this.state.hideChild ? null : <span class="hljs-tag">&lt;<span class="hljs-name">LifeCycle</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{this.state.text}</span> /&gt;</span>}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="1457">新的界面如下图所示：</p>
<p data-nodeid="1458"><img src="https://s0.lgstatic.com/i/image/M00/5D/CD/Ciqc1F-FVCGAVX_GAAFADHW8-9A107.png" alt="Drawing 7.png" data-nodeid="1653"></p>
<p data-nodeid="1459">可以看到，this.state.ownText 这个状态和子组件完全无关。但是当我点击“修改父组件自有文本内容”这个按钮的时候，componentReceiveProps 仍然被触发了，效果如下图所示：</p>
<p data-nodeid="1460"><img src="https://s0.lgstatic.com/i/image/M00/5D/D8/CgqCHl-FVCqASZNkAAGmF-R62cg649.png" alt="Drawing 8.png" data-nodeid="1657"></p>
<p data-nodeid="1461">耳听为虚，眼见为实。面对这样的运行结果，我不由得要带你复习一下 React 官方文档中的这句话：</p>
<p data-nodeid="1462"><img src="https://s0.lgstatic.com/i/image/M00/5D/E1/Ciqc1F-FaGuADV5vAACZ2YRV6qQ941.png" alt="图片7.png" data-nodeid="1661"></p>
<p data-nodeid="1463"><strong data-nodeid="1666">componentReceiveProps 并不是由 props 的变化触发的，而是由父组件的更新触发的</strong>，这个结论，请你谨记。</p>
<p data-nodeid="1464"><strong data-nodeid="1670">组件自身 setState 触发的更新</strong></p>
<p data-nodeid="1465">this.setState() 调用后导致的更新流程，前面大图中已经有体现，这里我直接沿用上一个 Demo 来做演示。若我们点击上一个 Demo 中的“修改子组件文本内容”这个按钮：</p>
<p data-nodeid="1466"><img src="https://s0.lgstatic.com/i/image/M00/5D/D8/CgqCHl-FVDWABuVmAADVzZuKCO0699.png" alt="Drawing 9.png" data-nodeid="1674"></p>
<p data-nodeid="1467">这个动作将会触发子组件 LifeCycle 自身的更新流程，随之被触发的生命周期函数如下图增加的 console 内容所示：</p>
<p data-nodeid="1468"><img src="https://s0.lgstatic.com/i/image/M00/5D/CD/Ciqc1F-FVDuASw5bAAEhb9melJQ452.png" alt="Drawing 10.png" data-nodeid="1678"></p>
<p data-nodeid="1469">先来说说 componentWillUpdate 和 componentDidUpdate 这一对好基友。</p>
<p data-nodeid="1470">componentWillUpdate 会在 render 前被触发，它和 componentWillMount 类似，允许你在里面做一些不涉及真实 DOM 操作的准备工作；而 componentDidUpdate 则在组件更新完毕后被触发，和 componentDidMount 类似，这个生命周期也经常被用来处理 DOM 操作。此外，我们也常常将 componentDidUpdate 的执行作为子组件更新完毕的标志通知到父组件。</p>
<p data-nodeid="1471"><strong data-nodeid="1684">render 与性能：初识 shouldComponentUpdate</strong></p>
<p data-nodeid="1472">这里需要重点提一下 shouldComponentUpdate 这个生命周期方法，它的调用形式如下所示：</p>
<pre class="lang-js" data-nodeid="1473"><code data-language="js">shouldComponentUpdate(nextProps, nextState)
</code></pre>
<p data-nodeid="1474">render 方法由于伴随着对虚拟 DOM 的构建和对比，过程可以说相当耗时。而在 React 当中，很多时候我们会不经意间就频繁地调用了 render。为了避免不必要的 render 操作带来的性能开销，React 为我们提供了 shouldComponentUpdate 这个口子。</p>
<p data-nodeid="1475">React 组件会根据 shouldComponentUpdate 的返回值，来决定是否执行该方法之后的生命周期，进而决定是否对组件进行<strong data-nodeid="1692">re-render</strong>（重渲染）。shouldComponentUpdate 的默认值为 true，也就是说“无条件 re-render”。在实际的开发中，我们往往通过手动往 shouldComponentUpdate 中填充判定逻辑，或者直接在项目中引入 PureComponent 等最佳实践，来实现“有条件的 re-render”。</p>
<p data-nodeid="1476">关于 shouldComponentUpdate 及 PureComponent 对 React 的优化，我们会在后续的性能小节中详细展开。这里你只需要认识到 shouldComponentUpdate 的基本使用及其<strong data-nodeid="1698">与 React 性能之间的关联关系</strong>即可。</p>
<h4 data-nodeid="1477">Unmounting 阶段：组件的卸载</h4>
<p data-nodeid="1478">组件的销毁阶段本身是比较简单的，只涉及一个生命周期，如下图所示：</p>
<p data-nodeid="1479"><img src="https://s0.lgstatic.com/i/image/M00/5D/EC/CgqCHl-FaHuAVGc_AABE6JqN9E0073.png" alt="图片6.png" data-nodeid="1703"></p>
<p data-nodeid="1480">对应上文的 Demo 来看，我们点击“隐藏子组件”后就可以把 LifeCycle 从父组件中移除掉，进而实现卸载的效果。整个过程如下图所示：</p>
<p data-nodeid="1481"><img src="https://s0.lgstatic.com/i/image/M00/5D/CD/Ciqc1F-FVFeABZvpAAO9lJVFKhs335.png" alt="Drawing 12.png" data-nodeid="1707"></p>
<p data-nodeid="1482">这个生命周期本身不难理解，我们重点说说怎么触发它。组件销毁的常见原因有以下两个。</p>
<ul data-nodeid="1483">
<li data-nodeid="1484">
<p data-nodeid="1485">组件在父组件中被移除了：这种情况相对比较直观，对应的就是我们上图描述的这个过程。</p>
</li>
<li data-nodeid="1486">
<p data-nodeid="1487">组件中设置了 key 属性，父组件在 render 的过程中，发现 key 值和上一次不一致，那么这个组件就会被干掉。</p>
</li>
</ul>
<p data-nodeid="1488">在本课时，只要能够理解到 1 就可以了。对于 2 这种情况，你只需要先记住有这样一种现象，这就够了。至于组件里面为什么要设置 key，为什么 key 改变后组件就必须被干掉？要回答这个问题，需要你先理解 React 的“调和过程”，而“调和过程”也会是我们第二模块中重点讲解的一个内容。这里我先把这个知识点点出来，方便你定位我们整个知识体系里的<strong data-nodeid="1716">重难点</strong>。</p>
<h3 data-nodeid="1489">总结</h3>
<p data-nodeid="1490">在本课时，我们对 React 设计思想中的“虚拟 DOM”和“组件化”这两个关键概念形成了初步的理解，同时也对 React 15 中的生命周期进行了系统的学习和总结。到这里，你已经了解到了 React 生命周期在很长一段“过去”里的形态。</p>
<p data-nodeid="1491">而在 React 16 中，组件的生命周期其实已经发生了一系列的变化。这些变化到底是什么样的，它们背后又蕴含着 React 团队怎样的思量呢？</p>
<p data-nodeid="1492">古人说“以史为镜，可以知兴衰”。在下个课时，我们将一起去“照镜子”，对 React 新旧生命周期进行对比，并探求变化的动机。</p>
<p data-nodeid="1493"><strong data-nodeid="1725">小编有话说</strong>：</p>
<p data-nodeid="1494">作为一名前端开发人员，我相信大家都会有一个明显的感觉：其实前端并没有想象的那么简单。近年来，前端的职责越来越重要，战场越来越多样，应用也越来越复杂。作为现阶段的“入局者”，你是否能够系统地掌握前端的知识体系？你对技术的理解是否触达底层原理？你的能力是否可以受到大厂青睐？</p>
<p data-nodeid="1495" class="">为了帮助前端人实现进阶学习，摆脱高不成低不就的困局。拉勾教育不仅开设了前端领域的专栏课，还研发了<a href="https://kaiwu.lagou.com/fe_enhancement.html?utm_source=lagouedu&amp;utm_medium=zhuanlan&amp;utm_campaign=%E5%A4%A7%E5%89%8D%E7%AB%AF%E9%AB%98%E8%96%AA%E8%AE%AD%E7%BB%83%E8%90%A5" data-nodeid="1730">“大前端高薪训练营”</a>，从知识体系构建、底层基础夯实、实战项目剖析、面试场景模拟到一线大厂内推，一站式解决前端进阶难题，打造你的核心竞争力。<a href="https://kaiwu.lagou.com/fe_enhancement.html?utm_source=lagouedu&amp;utm_medium=zhuanlan&amp;utm_campaign=%E5%A4%A7%E5%89%8D%E7%AB%AF%E9%AB%98%E8%96%AA%E8%AE%AD%E7%BB%83%E8%90%A5" data-nodeid="1734">点击链接</a>，即可了解更多关于前端进阶的内容。</p>

---

### 精选评论

##### **5949：
> 请问，为什么说虚拟DOM生成仰仗render？ 同时render与createElement有什么样的联系呀？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; babel 只是把 JSX 转化成 createElement，这个转化的过程发生在编译阶段，代码还没有运行。执行是在 render 里做的。

##### ezra.xu：
> 加深了对componentWillReceiveProps(nextProps)生命周期函数的理解，即父组件重新渲染时会触发所有子组件的该方法。

##### **6400：
> 代码样例非常赞，请问如何能获取到可运行的代码，老师有github链接吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 虽然没有 Github 链接，但是你可以使用 create-react-app 脚手架来快速完成项目的初始化（传送门 https://zh-hans.reactjs.org/docs/create-a-new-react-app.html），接下来把文中的代码塞进去就可以了。

##### *铮：
> 打卡学习

##### **8621：
> 原文中“componentDidMount 方法在渲染结束后被触发，此时因为真实 DOM 已经挂载到了页面上，我们可以在这个生命周期里执行真实 DOM 相关的操作。此外，类似于异步请求、数据初始化这样的操作也大可以放在这个生命周期来做（侧面印证了 componentWillMount 真的很鸡肋）”，这里有点疑问，componentWillMount中发起异步请求不是更好么，既不影响生命周期流程，也可以在挂载之后更快地初始化数据和重新渲染。如果在didMount中再发起异步请求，就会慢一些。原文中“componentReceiveProps 并不是由 props 的变化触发的，而是由父组件的更新触发的，这个结论，请你谨记。”这里为什么要设计成父组件更新(即使没有更新子组件相关的状态值)也要触发这个子组件生命周期函数呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; componentWillMount 的局限性在第03课时有非常详细的探讨。关于异步请求这一点，“异步请求再怎么快也快不过（React 15 下）同步的生命周期。componentWillMount 结束后，render 会迅速地被触发，所以说首次渲染依然会在数据返回之前执行。这样做不仅没有达到你预想的目的，还会导致服务端渲染场景下的冗余请求等额外问题，得不偿失”。除此之外，在 Fiber 带来的异步渲染机制下，componentWillMount 可能会导致非常严重的 Bug。最后一点，即使你没有开启异步，React 15 下也有不少人能用 componentWillMount 把自己“玩死”.....等等，以上这些在03课时都有很详细的解释说明。

##### **魁：
> 调理思路都很清晰，尤其是对于componentReceiveProps这个方法的讲解，看了很多网上的文章也没看明白，谢谢大佬。

##### *川：
> 调理无比清晰 我很受用

##### **7323：
> 话不多，但是讲的真清晰

##### fanta：
> 打卡 支持 赞！ 这节其实对生命周期的理解更加深刻 对于vDom还是要从另一个剖面切入来讲 期待

##### **侯：
> 终于更新了

##### **用户9471：
> 买过修言大佬两本小册 大佬讲课讲得也好好 很细致

##### **8542：
> 15的生命周期一共有9个，在组件第一次加载的时候执行3个，component willmount里面做一些不涉真dom的操作，但是很多人喜欢将请求数据写在里面。render执行babel给我转换的createelement方法，然后将所有的组件元素转换成reactelement具有特定数据结构的对象，也就是我们说的虚拟dom，之后会执行componentdidmount，在他里面可以操作真是dom，异步请求数据，当父组件发生变化的时候，就会触发组件的更新此时执行的是componentwillreceiveprops，此时能够接受到最新的props。当组件自己的state发生变化的时候，直接触发的是should componentupdate，这是一个用于做性能优化的组件决定这个组是否更新的开关。和pure component，react.memo类似。接下来如果开关打开，组件更新，就进入了conponentwillupdate，用来做和真实dom无关的事情，此时连虚拟dom都没有产生，所以做好别做和dom有关的事情，这个生命周期和componentwillmount相似，只不过一个是第一次渲染的时候执行，一个是组件更新的时候执行，执行的除过执行时机不同，其他都一样。接下来就是render此时就要做经典的diff计算了，通过比较，产生要渲染的组件虚拟dom，之后进入componentdidupdate，他和componentdidmount一样，除过触发时机不同其他都一样，主要是用来操作dom的。如果子组件要从父组件里面移出，或者子组件的key发生了变化，此时组件就要被卸载，触发componentwillunmount，他里面不能操作state，用来解绑事件，删除定时器等。

##### **用户4967：
> 老师问个问题，文中有提到“render 在执行过程中并不会去操作真实 DOM（也就是说不会渲染），它的职能是把需要渲染的内容返回出来。真实 DOM 的渲染工作，在挂载阶段是由 ReactDOM.render 来承接的”。那么更新阶段的 DOM 渲染工作是在哪里完成的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 更新阶段和挂载阶段都会触发React生命周期中的 commit 过程，真实 DOM 的渲染是在这个过程里完成的。细节可以参考13-15节。

##### *辉：
> 打卡，新旧对比非常nice

##### **安：
> 不错，打卡下

##### **宝：
> 学习了总结了

##### **用户9471：
> 菜鸡提问 请问为啥类组件里面才有render方法 函数组件里面没有render方法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; render 是 React.Component 类里面的一个实例方法，函数和类不是一个概念。函数组件本身已经覆盖了 render 方法的职能。

##### WEL：
> 刚看不练是傻把式，打卡学习，github练习demo附上：https://github.com/henni-719/react-demo

##### **豪：
> 思路清晰，干货满满，谢谢！

##### **峰：
> 通俗易懂，赞

##### **2960：
> 粉了秀妍

##### **0376：
> 写的简直太好了，强力推荐

##### **锐：
> 打卡学习

##### **雨：
> 第二篇学完打卡

##### **康：
> 沙发

