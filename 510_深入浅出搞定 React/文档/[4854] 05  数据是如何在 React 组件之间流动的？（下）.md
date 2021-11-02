<p data-nodeid="1337" class="">在上个课时，我们掌握了 React 数据流方案中风格相对“朴素”的 Props 单向数据流方案，以及通用性较强的“发布-订阅”模式。在本课时，我们将一起认识 React 天然具备的全局通信方式“Context API”，并对 Redux 的设计思想和编码形态进行初步的探索。</p>
<h3 data-nodeid="1338">使用 Context API 维护全局状态</h3>
<p data-nodeid="1339">Context API 是 React 官方提供的一种组件树全局通信的方式。</p>
<p data-nodeid="1340">在 React 16.3 之前，Context API 由于存在种种局限性，并不被 React 官方提倡使用，开发者更多的是把它作为一个概念来探讨。而从 v 16.3.0 开始，React 对 Context API 进行了改进，新的 Context API 具备更强的可用性。这里我们首先针对 React 16 下 Context API 的形态进行介绍。</p>
<h4 data-nodeid="1341">图解 Context API 工作流</h4>
<p data-nodeid="1342">Context API 有 3 个关键的要素：React.createContext、Provider、Consumer。</p>
<p data-nodeid="1343">我们通过调用 React.createContext，可以创建出一组 Provider。Provider 作为数据的提供方，可以将数据下发给自身组件树中任意层级的 Consumer，这三者之间的关系用一张图来表示：</p>
<p data-nodeid="1344"><img src="https://s0.lgstatic.com/i/image/M00/62/97/CgqCHl-Sm7iAQ6ZRAAEW2Me7WVg371.png" alt="图片3.png" data-nodeid="1459"></p>
<p data-nodeid="1345">注意：Cosumer 不仅能够读取到 Provider 下发的数据，<strong data-nodeid="1465">还能读取到这些数据后续的更新</strong>。这意味着数据在生产者和消费者之间能够及时同步，这对 Context 这种模式来说至关重要。</p>
<h4 data-nodeid="1346">从编码的角度认识“三要素”</h4>
<ul data-nodeid="1347">
<li data-nodeid="1348">
<p data-nodeid="1349"><strong data-nodeid="1471">React.createContext</strong>，作用是创建一个 context 对象。下面是一个简单的用法示范：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1350"><code data-language="java"><span class="hljs-keyword">const</span> AppContext = React.createContext()
</code></pre>
<p data-nodeid="1351">注意，在创建的过程中，我们可以选择性地传入一个 defaultValue：</p>
<pre class="lang-java" data-nodeid="1352"><code data-language="java"><span class="hljs-keyword">const</span> AppContext = React.createContext(defaultValue)
</code></pre>
<p data-nodeid="1353">从创建出的 context 对象中，我们可以读取到 Provider 和 Consumer：</p>
<pre class="lang-java" data-nodeid="1354"><code data-language="java"><span class="hljs-keyword">const</span> { Provider, Consumer } = AppContext
</code></pre>
<ul data-nodeid="1355">
<li data-nodeid="1356">
<p data-nodeid="1357"><strong data-nodeid="1478">Provider</strong>，可以理解为“数据的 Provider（提供者）”。</p>
</li>
</ul>
<p data-nodeid="1358">我们使用 Provider 对组件树中的根组件进行包裹，然后传入名为“value”的属性，这个 value 就是后续在组件树中流动的“数据”，它可以被 Consumer 消费。使用示例如下：</p>
<pre class="lang-java" data-nodeid="1359"><code data-language="java">&lt;Provider value={title: this.state.title, content: this.state.content}&gt;
  &lt;Title /&gt;
  &lt;Content /&gt;
 &lt;/Provider&gt;
</code></pre>
<ul data-nodeid="1360">
<li data-nodeid="1361">
<p data-nodeid="1362"><strong data-nodeid="1484">Consumer</strong>，顾名思义就是“数据的消费者”，它可以读取 Provider 下发下来的数据。</p>
</li>
</ul>
<p data-nodeid="1363">其特点是需要接收一个函数作为子元素，这个函数需要返回一个组件。像这样：</p>
<pre class="lang-js" data-nodeid="1364"><code data-language="js">&lt;Consumer&gt;
  {value =&gt; <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>{value.title}<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>}
&lt;/Consumer&gt;
</code></pre>
<p data-nodeid="1365">注意，当 Consumer 没有对应的 Provider 时，value 参数会直接取创建 context 时传递给 createContext 的 defaultValue。</p>
<h4 data-nodeid="1366">新的 Context API 解决了什么问题</h4>
<p data-nodeid="1367">想要知道新的 Context API 解决了什么问题，先要知道过时的 Context API 存在什么问题。</p>
<p data-nodeid="1368"><strong data-nodeid="1492">我们先从编码角度认识“过时的”Context API</strong></p>
<p data-nodeid="1369">“过时的”是 React 官方对旧的 &nbsp;Context API 的描述，由于个人和团队在实际项目中都并不会考虑去使用旧 Context API 来解决问题，这里我直接引用<a href="https://zh-hans.reactjs.org/docs/legacy-context.html" data-nodeid="1496">过时的文档</a>中的 Context API 使用示例：</p>
<pre class="lang-js" data-nodeid="1370"><code data-language="js"><span class="hljs-keyword">import</span> PropTypes <span class="hljs-keyword">from</span> <span class="hljs-string">'prop-types'</span>;
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Button</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">style</span>=<span class="hljs-string">{{background:</span> <span class="hljs-attr">this.context.color</span>}}&gt;</span>
        {this.props.children}
      <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
    );
  }
}
Button.contextTypes = {
  <span class="hljs-attr">color</span>: PropTypes.string
};
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Message</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
        {this.props.text} <span class="hljs-tag">&lt;<span class="hljs-name">Button</span>&gt;</span>Delete<span class="hljs-tag">&lt;/<span class="hljs-name">Button</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MessageList</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  getChildContext() {
    <span class="hljs-keyword">return</span> {<span class="hljs-attr">color</span>: <span class="hljs-string">"purple"</span>};
  }
  render() {
    <span class="hljs-keyword">const</span> children = <span class="hljs-keyword">this</span>.props.messages.map(<span class="hljs-function">(<span class="hljs-params">message</span>) =&gt;</span>
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">Message</span> <span class="hljs-attr">text</span>=<span class="hljs-string">{message.text}</span> /&gt;</span></span>
    );
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>{children}<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>;
  }
}
MessageList.childContextTypes = {
  <span class="hljs-attr">color</span>: PropTypes.string
};
</code></pre>
<p data-nodeid="1371">为了方便你理解，我将上述代码对应的组织结构梳理到一张图里，如下图所示：</p>
<p data-nodeid="1372"><img src="https://s0.lgstatic.com/i/image/M00/62/8C/Ciqc1F-Sm8qAMOB0AAEcBeEv_vs533.png" alt="图片4.png" data-nodeid="1501"></p>
<p data-nodeid="1373">借着这张图，我们来理解旧的 Context API 的工作过程：</p>
<ul data-nodeid="1374">
<li data-nodeid="1375">
<p data-nodeid="1376">首先，通过给 MessageList 设置 childContextTypes 和 getChildContext，可以使其承担起 context 的生产者的角色；</p>
</li>
<li data-nodeid="1377">
<p data-nodeid="1378">然后，MessageList 的组件树内部所有层级的组件都可以通过定义 contextTypes 来成为数据的消费者，进而通过 this.context 访问到 MessageList 提供的数据。</p>
</li>
</ul>
<p data-nodeid="1379">现在回过头来，我们再从编码角度审视一遍“过时的” Context API 的用法。</p>
<p data-nodeid="1380">首先映入眼帘的第一个问题是<strong data-nodeid="1511">代码不够优雅</strong>：一眼望去，你很难迅速辨别出谁是 Provider、谁是 Consumer。同时这琐碎的属性设置和 API 编写过程，也足够我们写代码的时候“喝一壶了”。总而言之，从编码形态的角度来说，“过时的” Context API 和新 Context API 相去甚远。</p>
<p data-nodeid="1381">不过，这还不是最要命的，最要命的弊端我们从编码层面暂时感知不出来，但是一旦你感知到它，麻烦就大了——前面我们特别提到过，“Cosumer 不仅能够读取到 Provider 下发的数据，<strong data-nodeid="1517">还能够读取到这些数据后续的更新”</strong>。数据在生产者和消费者之间的及时同步，这一点对于 Context 这种模式来说是至关重要的，但旧的 Conext API 无法保证这一点：</p>
<blockquote data-nodeid="1382">
<p data-nodeid="1383">如果组件提供的一个Context发生了变化，而中间父组件的 shouldComponentUpdate 返回 false，<strong data-nodeid="1527">那么使用到该值的后代组件不会进行更新</strong>。使用了 Context 的组件则完全失控，所以基本上没有办法能够可靠的更新 Context。<a href="https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076" data-nodeid="1525">这篇博客文章</a>很好地解释了为何会出现此类问题，以及你该如何规避它。 &nbsp;——React 官方</p>
</blockquote>
<p data-nodeid="1384">新的 Context API 改进了这一点：<strong data-nodeid="1537">即便组件的 shouldComponentUpdate 返回 false，它仍然可以“穿透”组件继续向后代组件进行传播</strong>，<strong data-nodeid="1538">进而确保了数据生产者和数据消费者之间数据的一致性</strong>。再加上更加“好看”的语义化的声明式写法，新版 Context API 终于顺利地摘掉了“试验性 API”的帽子，成了一种确实可行的 React 组件间通信解决方案。</p>
<p data-nodeid="1385">理解了 Context API 的前世今生，接下来我们继续来串联 React 组件间通信的解决方案。</p>
<h3 data-nodeid="1386">第三方数据流框架“课代表”：初探 Redux</h3>
<p data-nodeid="1387">对于简单的跨层级组件通信，我们可以使用发布-订阅模式或者 Context API 来搞定。但是随着应用的复杂度不断提升，需要维护的状态越来越多，组件间的关系也越来越难以处理的时候，我们就需要请出 Redux 来帮忙了。</p>
<h4 data-nodeid="1388">什么是 Redux</h4>
<p data-nodeid="1389">我们先来看一下官方对 Redux 的描述：</p>
<blockquote data-nodeid="1390">
<p data-nodeid="1391">Redux 是 JavaScript 状态容器，它提供可预测的状态管理。</p>
</blockquote>
<p data-nodeid="1392">我们一起品品这句话背后的深意：</p>
<ul data-nodeid="1393">
<li data-nodeid="1394">
<p data-nodeid="1395">Redux 是为<strong data-nodeid="1551">JavaScript</strong>应用而生的，也就是说它不是 React 的专利，React 可以用，Vue 可以用，原生 JavaScript 也可以用；</p>
</li>
<li data-nodeid="1396">
<p data-nodeid="1397">Redux 是一个<strong data-nodeid="1557">状态容器</strong>，什么是状态容器？这里我举个例子。</p>
</li>
</ul>
<p data-nodeid="1398">假如把一个 React 项目里面的所有组件拉进一个钉钉群，那么 Redux 就充当了这个群里的“群文件”角色，所有的组件都可以把需要在组件树里流动的数据存储在群文件里。当某个数据改变的时候，其他组件都能够通过下载最新的群文件来获取到数据的最新值。这就是“状态容器”的含义——存放公共数据的仓库。</p>
<p data-nodeid="1399">读懂了这个比喻之后，你对 Redux、数据和 React 组件的关系想必已经形成了一个初步的认知。这里我帮你把这层关系总结进一张图里：</p>
<p data-nodeid="1400"><img src="https://s0.lgstatic.com/i/image/M00/62/97/CgqCHl-Sm9qASdHXAAEjhh30y4s113.png" alt="图片5.png" data-nodeid="1562"></p>
<h4 data-nodeid="1401">Redux 是如何帮助 React 管理数据的</h4>
<p data-nodeid="1402">Redux 主要由三部分组成：store、reducer 和 action。我们先来看看它们各自代表什么：</p>
<ul data-nodeid="1403">
<li data-nodeid="1404">
<p data-nodeid="1405">store 就好比组件群里的“群文件”，它是一个<strong data-nodeid="1570">单一的数据源</strong>，而且是只读的；</p>
</li>
<li data-nodeid="1406">
<p data-nodeid="1407">action 人如其名，是“动作”的意思，它是<strong data-nodeid="1576">对变化的描述</strong>。</p>
</li>
</ul>
<p data-nodeid="1408">举个例子，下面这个对象就是一个 action：</p>
<pre class="lang-java" data-nodeid="1409"><code data-language="java"><span class="hljs-keyword">const</span> action = {
  type: <span class="hljs-string">"ADD_ITEM"</span>,
  payload: <span class="hljs-string">'&lt;li&gt;text&lt;/li&gt;'</span>
}
</code></pre>
<ul data-nodeid="1410">
<li data-nodeid="1411">
<p data-nodeid="1412">reducer 是一个函数，它负责<strong data-nodeid="1583">对变化进行分发和处理，</strong> 最终将新的数据返回给 store。</p>
</li>
</ul>
<p data-nodeid="1413">store、action 和 reducer 三者紧密配合，便形成了 Redux 独树一帜的工作流：</p>
<p data-nodeid="1414"><img src="https://s0.lgstatic.com/i/image/M00/62/97/CgqCHl-Sm-yADE6PAACSEywFSaA197.png" alt="图片6.png" data-nodeid="1587"></p>
<p data-nodeid="1415">从上图中，我们首先读出的是数据的流向规律：<strong data-nodeid="1593">在 Redux 的整个工作过程中，数据流是严格单向的</strong>。这一点一定一定要背下来，面试的时候也一定一定要记得说——不管面试官问的是 Redux 的设计思想还是工作流还是别的什么概念性的知识，开局先放这句话，准没错。</p>
<p data-nodeid="1416">接下来仍然是围绕上图，我们来一起看看 Redux 是如何帮助 React 管理数据流的。对于一个 React 应用来说，视图（View）层面的所有数据（state）都来自 store（再一次诠释了单一数据源的原则）。</p>
<p data-nodeid="1417">如果你想对数据进行修改，只有一种途径：派发 action。action 会被 reducer 读取，进而根据 action 内容的不同对数据进行修改、生成新的 state（状态），这个新的 state 会更新到 store 对象里，进而驱动视图层面做出对应的改变。</p>
<p data-nodeid="1418">对于组件来说，任何组件都可以通过约定的方式从 store 读取到全局的状态，任何组件也都可以通过合理地派发 action 来修改全局的状态。<strong data-nodeid="1601">Redux 通过提供一个统一的状态容器，使得数据能够自由而有序地在任意组件之间穿梭</strong>，这就是 Redux 实现组件间通信的思路。</p>
<h4 data-nodeid="1419">从编码的角度理解 Redux 工作流</h4>
<p data-nodeid="1420">到这里，你已经了解了 Redux 的设计思想和要素关系。接下来我们将站在编码的角度，探讨 Redux 的工作流，将上文中所提及的各个要素和流程具象化。</p>
<p data-nodeid="1421"><strong data-nodeid="1609">1. 使用 createStore 来完成 store 对象的创建</strong></p>
<pre class="lang-java" data-nodeid="1422"><code data-language="java"><span class="hljs-comment">// 引入 redux</span>
<span class="hljs-keyword">import</span> { createStore } from <span class="hljs-string">'redux'</span>
<span class="hljs-comment">// 创建 store</span>
<span class="hljs-keyword">const</span> store = createStore(
    reducer,
    initial_state,
    applyMiddleware(middleware1, middleware2, ...)
);
</code></pre>
<p data-nodeid="1423">createStore 方法是一切的开始，它接收三个入参：</p>
<ul data-nodeid="1424">
<li data-nodeid="1425">
<p data-nodeid="1426">reducer；</p>
</li>
<li data-nodeid="1427">
<p data-nodeid="1428">初始状态内容；</p>
</li>
<li data-nodeid="1429">
<p data-nodeid="1430">指定中间件（这个你先不用管）。</p>
</li>
</ul>
<p data-nodeid="1431">这其中一般来说，只有 reducer 是你不得不传的。下面我们就看看 reducer 的编码形态是什么样的。</p>
<p data-nodeid="1432"><strong data-nodeid="1620">2. reducer 的作用是将新的 state 返回给 store</strong></p>
<p data-nodeid="1433">一个 reducer 一定是一个纯函数，它可以有各种各样的内在逻辑，但它最终一定要返回一个 state：</p>
<pre class="lang-java" data-nodeid="1434"><code data-language="java"><span class="hljs-keyword">const</span> reducer = (state, action) =&gt; {
    <span class="hljs-comment">// 此处是各种样的 state处理逻辑</span>
    <span class="hljs-keyword">return</span> new_state
}
</code></pre>
<p data-nodeid="1435">当我们基于某个 reducer 去创建 store 的时候，其实就是给这个 store 指定了一套更新规则：</p>
<pre class="lang-java" data-nodeid="1436"><code data-language="java"><span class="hljs-comment">// 更新规则全都写在 reducer 里 </span>
<span class="hljs-keyword">const</span> store = createStore(reducer)
</code></pre>
<p data-nodeid="1437"><strong data-nodeid="1628">3. action 的作用是通知 reducer “让改变发生”</strong></p>
<p data-nodeid="1438">如何在浩如烟海的 store 状态库中，准确地命中某个我们希望它发生改变的 state 呢？reducer 内部的逻辑虽然不尽相同，但其本质工作都是“将 action 与和它对应的更新动作对应起来，并处理这个更新”。所以说<strong data-nodeid="1634">要想让 state 发生改变，就必须用正确的 action 来驱动这个改变</strong>。</p>
<p data-nodeid="1439">前面我们已经介绍过 action 的形态，这里再提点一下。首先，action 是一个大致长这样的对象：</p>
<pre class="lang-java" data-nodeid="1440"><code data-language="java"><span class="hljs-keyword">const</span> action = {
  type: <span class="hljs-string">"ADD_ITEM"</span>,
  payload: <span class="hljs-string">'&lt;li&gt;text&lt;/li&gt;'</span>
}
</code></pre>
<p data-nodeid="1441">action 对象中允许传入的属性有多个，<strong data-nodeid="1641">但只有 type 是必传的</strong>。type 是 action 的唯一标识，reducer 正是通过不同的 type 来识别出需要更新的不同的 state，由此才能够实现精准的“定向更新”。</p>
<p data-nodeid="1442"><strong data-nodeid="1647">4. 派发 action，靠的是 dispatch</strong></p>
<p data-nodeid="1443">action 本身只是一个对象，要想让 reducer 感知到 action，还需要“派发 action”这个动作，这个动作是由 store.dispatch 完成的。这里我简单地示范一下：</p>
<pre class="lang-java" data-nodeid="1444"><code data-language="java"><span class="hljs-keyword">import</span> { createStore } from <span class="hljs-string">'redux'</span>
<span class="hljs-comment">// 创建 reducer</span>
<span class="hljs-keyword">const</span> reducer = (state, action) =&gt; {
    <span class="hljs-comment">// 此处是各种样的 state处理逻辑</span>
    <span class="hljs-keyword">return</span> new_state
}
<span class="hljs-comment">// 基于 reducer 创建 state</span>
<span class="hljs-keyword">const</span> store = createStore(reducer)
<span class="hljs-comment">// 创建一个 action，这个 action 用 “ADD_ITEM” 来标识 </span>
<span class="hljs-keyword">const</span> action = {
  type: <span class="hljs-string">"ADD_ITEM"</span>,
  payload: <span class="hljs-string">'&lt;li&gt;text&lt;/li&gt;'</span>
}
<span class="hljs-comment">// 使用 dispatch 派发 action，action 会进入到 reducer 里触发对应的更新</span>
store.dispatch(action)
</code></pre>
<p data-nodeid="2338">以上这段代码，是从编码角度对 Redux 主要工作流的概括，这里我同样为你总结了一张对应的流程图：</p>
<p data-nodeid="3019"><img src="https://s0.lgstatic.com/i/image/M00/81/9F/CgqCHl_Rii2AVvUbAADn4s_6rB8369.png" alt="图片7.png" data-nodeid="3023"></p>
<p data-nodeid="3020">注意：先别急着死磕。本课时并不要求你掌握 Redux 中涉及的所有概念和原理，只需要你跟着我的思路走，大致理解 Redux 中几个关键角色之间的关系，进而明白 Redux 是如何驱动数据在 React 组件间流动、如何帮助我们实现<strong data-nodeid="3029">灵活的组件间通信</strong>的，这就够了。关于更多 Redux 的技术细节，我将在专栏的第三个大模块慢慢推敲。</p>





<h3 data-nodeid="1448">总结</h3>
<p data-nodeid="1449" class="">在 04 和 05 课时，我讲解的知识点覆盖面广、跨度大。面试场景下，考察此类问题的目的也主要是对候选人的知识广度进行检验。因此对于这两节的内容，你应抱着梳理“<strong data-nodeid="1669">知识地图</strong>”的目的去学习，以<strong data-nodeid="1670">构建知识体系</strong>为第一要务。完成第一要务后，再带着一个完整的上下文，去攻克某个具体的薄弱点。</p>

---

### 精选评论

##### **宇：
> 说个想法，可能会让作者大大不高兴。看到现在，对我而言暂时还没有太多干货，大部分都是有经验的开发本就应该掌握的知识，可以当做巩固复习。希望后续的fiber部分，react源码部分能得到不一样的收获。最后还是要点个赞：把react的一些行为职责 设计思想讲的很清楚，产生共鸣的感觉还是很棒的！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，我不高兴了。目前专栏更新进度仍然在基础篇，站在基础篇强求深度无异于缘木求鱼。在开篇词中有提到，读者的学习层次跨度是非常大的，内容必须有向下兼容的能力，不会只面向某个能力层次的群体做局部的知识讲解，那就不是知识体系了，更谈不上深入浅出。是否“干货”，还请读完整个专栏再来定义。

##### **生：
> 修言说得对。这是一篇讲述react知识体系的专栏，能把react中平时不会注意到的一些东西通过一些例子言简意赅表达出来。我觉得已经是一篇成功的专栏了（虽然我好像是一块钱白嫖的嘻嘻嘻）😌

##### **用户9471：
> 作为一个没有react开发经验的娃 表示修言大佬的课程对我来说很有用 前面的几节课程反反复复看了好几遍 很有收获 谢谢大佬

##### **豪：
> 感觉Redux这块没用过的人估计会听的很懵逼，个人理解，Redux的Store就像一台机器，这个机器需要原材料（initialstate)、机器指令（action）、指令对应的原材料加工方法（reducer）

##### **5949：
> 您好，针对Context API 部分，希望能深入讲解一下，例如provider与consumer之间是如何交互的，为什么provider的数据更新之后consumer能够跟着进行更新。感觉课程在思想方面讲的很好，但是涉及真正的底层流程感觉比较模糊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你好，05讲在课程设计上属于基础篇，而底层原理则是从08讲开始分析的。一口吃不成个胖子，基础篇的用意就是帮助大家理解设计理念、完善知识体系。不要急，继续往下读，会有更加深入的知识在后面等你。

##### **东：
> 这个你先不用管 这句话爱了 大佬撑腰 学习无负担了

##### **雄：
> 评论怎么这么少，基础思想‘认知’先行，再知识原理‘深入剖析’，最后最佳实践的方式浅出实战。修言的学习方法，大家可以模仿。知识重要，学习方法也重要。优秀人的学习方法是不可多得的成长建议，适合自己的就学为己用，共勉

##### **森：
> 跟上进度了，打卡

##### **生：
> 写的很棒

##### **8542：
> 主要讲了数据传播的两个办法:context，redux

##### Lemon：
> 初学，看了react 官网来的，看了前五节，收获颇多。喜欢作者的讲解风格。

##### **宇：
> 把redux比作钉钉太妙了。。。。我这以后再也忘不掉了。。看见redux就想到了钉钉

##### *凡：
> 非常棒！先从宏观上去讲解设计思想与流程，迅速建立认知。是适合我的“基础篇”。“基础”但“深远”。

##### **刚：
> 类组件和函数组件的区别讲的非常好，有了深刻的认知

##### *冻：
> 收货很多，感谢作者分享

##### **峰：
> 思路真的好清晰呀。谢谢老师！

##### *浩：
> 打卡

