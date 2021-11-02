<p data-nodeid="1253" class="">从本课时开始，我们将逐步进入 React-Hooks 的世界。</p>
<p data-nodeid="1254">在动笔写 React-Hooks 之前，我发现许多人对这块的知识非常不自信，至少在面试场景下，几乎没有几个人在聊到 React-Hooks 的时候，能像聊 Diff 算法、Fiber 架构一样滔滔不绝、言之有物。后来我仔细反思了一下，认为问题应该出在学习姿势上。</p>
<p data-nodeid="1255">提起 React-Hooks，可能很多人的第一反应，都会是 useState、useEffect、useContext 这些琐碎且繁多的 API。似乎 React-Hooks 就是一坨没有感情的工具性代码，压根没有啥玄妙的东西在里面，那些大厂面试官天天让咱聊 React-Hooks，到底是想听啥呢？</p>
<h3 data-nodeid="1256">掌握 React-Hooks 的正确姿势</h3>
<p data-nodeid="1257">前面我和你聊到过，当我们由浅入深地认知一样新事物的时候，往往需要遵循“Why→What→How”这样的一个认知过程。</p>
<p data-nodeid="1258">在我的读者中，不少人在“What”和“How”这两个环节做得都不错，但是却疏于钻研背后的“Why”。其实这三者是相辅相成、缺一不可的：当我们了解了具体的“What”和“How”之后，往往能够更加具象地回答理论层面“Why”的问题；而我们对“Why”的探索和认知，也必然会反哺到对“What”的理解和对“How”的实践。</p>
<p data-nodeid="1259">这其中，我们尤其不能忽略对“Why”的把控。</p>
<p data-nodeid="1260"><img src="https://s0.lgstatic.com/i/image/M00/65/48/CgqCHl-aaC-AP5wdAACO3S8xt5c566.png" alt="1.png" data-nodeid="1349"></p>
<p data-nodeid="1261">React-Hooks 自 React 16.8 以来才真正被推而广之，对我们每一个老 React 开发来说，它都是一个新事物。如果在认知它的过程当中，我们能够遵循“Why→What→How”这样的一个学习法则，并且以此为线索，梳理出属于自己的完整知识链路。那么我相信，面对再刁钻的面试官，你都可以做到心中有数、对答如流。</p>
<p data-nodeid="1262">接下来两个课时，我们就遵循这个学习法则，向 React-Hooks 发起挑战，真正理解它背后的设计动机与工作模式。</p>
<h3 data-nodeid="1263">React-Hooks 设计动机初探</h3>
<p data-nodeid="1264">开篇我们先来聊“Why”。React-Hooks 这个东西比较特别，它是 React 团队在真刀真枪的 React 组件开发实践中，逐渐认知到的一个改进点，这背后其实涉及对<strong data-nodeid="1362">类组件</strong>和<strong data-nodeid="1363">函数组件</strong>两种组件形式的思考和侧重。因此，你首先得知道，什么是类组件、什么是函数组件，并完成对这两种组件形式的辨析。</p>
<h4 data-nodeid="1265">何谓类组件（Class Component）</h4>
<p data-nodeid="1266">所谓类组件，就是基于 ES6 Class 这种写法，通过继承 React.Component 得来的 React 组件。以下是一个典型的类组件：</p>
<pre class="lang-js" data-nodeid="1267"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DemoClass</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
 
  <span class="hljs-comment">// 初始化类组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">""</span>
  };
  <span class="hljs-comment">// 编写生命周期方法 didMount</span>
  componentDidMount() {
    <span class="hljs-comment">// 省略业务逻辑</span>
  }
  <span class="hljs-comment">// 编写自定义的实例方法</span>
  changeText = <span class="hljs-function">(<span class="hljs-params">newText</span>) =&gt;</span> {
    <span class="hljs-comment">// 更新 state</span>
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: newText
    });
  };
  <span class="hljs-comment">// 编写生命周期方法 render</span>
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"demoClass"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{this.state.text}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span>&gt;</span>点我修改<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<h4 data-nodeid="1268">何谓函数组件/无状态组件（Function Component/Stateless Component）</h4>
<p data-nodeid="1269">函数组件顾名思义，就是<strong data-nodeid="1372">以函数的形态</strong>存在的 React 组件。早期并没有 React-Hooks 的加持，函数组件内部无法定义和维护 state，因此它还有一个别名叫“无状态组件”。以下是一个典型的函数组件：</p>
<pre class="lang-js" data-nodeid="1270"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">DemoFunction</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-keyword">const</span> { text } = props
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"demoFunction"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{`function 组件所接收到的来自外界的文本内容是：[${text}]`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>
<h4 data-nodeid="1271">函数组件与类组件的对比：无关“优劣”，只谈“不同”</h4>
<p data-nodeid="1272">我们先基于上面的两个 Demo，从形态上对两种组件做区分。它们之间肉眼可见的区别就包括但不限于：</p>
<ul data-nodeid="1273">
<li data-nodeid="1274">
<p data-nodeid="1275">类组件需要继承 class，函数组件不需要；</p>
</li>
<li data-nodeid="1276">
<p data-nodeid="1277">类组件可以访问生命周期方法，函数组件不能；</p>
</li>
<li data-nodeid="1278">
<p data-nodeid="1279">类组件中可以获取到实例化后的 this，并基于这个 this 做各种各样的事情，而函数组件不可以；</p>
</li>
<li data-nodeid="1280">
<p data-nodeid="1281">类组件中可以定义并维护 state（状态），而函数组件不可以；</p>
</li>
<li data-nodeid="1282">
<p data-nodeid="1283">......</p>
</li>
</ul>
<p data-nodeid="1284">单就我们列出的这几点里面，频繁出现了“类组件可以 xxx，函数组件不可以 xxx”，这是否就意味着类组件比函数组件更好呢？</p>
<p data-nodeid="1285">答案当然是否定的。你可以说，在 React-Hooks 出现之前的世界里，<strong data-nodeid="1386">类组件的能力边界明显强于函数组件</strong>，但要进一步推导“类组件强于函数组件”，未免显得有些牵强。同理，一些文章中一味鼓吹函数组件轻量优雅上手迅速，不久的将来一定会把类组件干没（类组件：我做错了什么？）之类的，更是不可偏听偏信。</p>
<p data-nodeid="1286">当我们讨论这两种组件形式时，<strong data-nodeid="1392">不应怀揣“孰优孰劣”这样的成见，而应该更多地去关注两者的不同，进而把不同的特性与不同的场景做连接</strong>，这样才能求得一个全面的、辩证的认知。</p>
<h3 data-nodeid="1287">重新理解类组件：包裹在面向对象思想下的“重装战舰”</h3>
<p data-nodeid="1288">类组件是面向对象编程思想的一种表征。面向对象是一个老生常谈的概念了，当我们应用面向对象的时候，总是会有意或无意地做这样两件事情。</p>
<ol data-nodeid="1289">
<li data-nodeid="1290">
<p data-nodeid="1291">封装：将一类属性和方法，“聚拢”到一个 Class 里去。</p>
</li>
<li data-nodeid="1292">
<p data-nodeid="1293">继承：新的 Class 可以通过继承现有 Class，实现对某一类属性和方法的复用。</p>
</li>
</ol>
<p data-nodeid="1294">React 类组件也不例外。我们再次审视一下这个典型的类组件 Case：</p>
<pre class="lang-js" data-nodeid="1295"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DemoClass</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
 
  <span class="hljs-comment">// 初始化类组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">""</span>
  };
  <span class="hljs-comment">// 编写生命周期方法 didMount</span>
  componentDidMount() {
    <span class="hljs-comment">// 省略业务逻辑</span>
  }
  <span class="hljs-comment">// 编写自定义的实例方法</span>
  changeText = <span class="hljs-function">(<span class="hljs-params">newText</span>) =&gt;</span> {
    <span class="hljs-comment">// 更新 state</span>
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: newText
    });
  };
  <span class="hljs-comment">// 编写生命周期方法 render</span>
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"demoClass"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{this.state.text}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span>&gt;</span>点我修改<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="1296">不难看出，React 类组件内部预置了相当多的“现成的东西”等着你去调度/定制，state 和生命周期就是这些“现成东西”中的典型。要想得到这些东西，难度也不大，你只需要轻轻地<strong data-nodeid="1403">继承</strong>一个 React.Component 即可。</p>
<p data-nodeid="1297">这种感觉就好像是你不费吹灰之力，就拥有了一辆“重装战舰”，该有的枪炮导弹早已配备整齐，就等你操纵控制台上的一堆开关了。</p>
<p data-nodeid="1298">毋庸置疑，类组件给到开发者的东西是足够多的，但“多”就是“好”吗？其实未必。</p>
<p data-nodeid="1299">把一个人塞进重装战舰里，他就一定能操纵这台战舰吗？如果他没有经过严格的训练，不清楚每一个操作点的内涵，那他极有可能会把炮弹打到友军的营地里去。</p>
<p data-nodeid="1300">React 类组件，也有同样的问题——它提供了多少东西，你就需要学多少东西。假如背不住生命周期，你的组件逻辑顺序大概率会变成一团糟。<strong data-nodeid="1412">“大而全”的背后，是不可忽视的学习成本</strong>。</p>
<p data-nodeid="1301">再想这样一个场景：假如我现在只是需要打死一只蚊子，而不是打掉一个军队。这时候继续开动重装战舰，是不是正应了那句老话——“可以，但没有必要”。这也是类组件的一个不便，<strong data-nodeid="1418">它太重了</strong>，对于解决许多问题来说，编写一个类组件实在是一个过于复杂的姿势。复杂的姿势必然带来高昂的理解成本，这也是我们所不想看到的。</p>
<p data-nodeid="1302">更要命的是，由于开发者编写的逻辑在<strong data-nodeid="1430">封装</strong>后是和组件粘在一起的，这就使得类**组件内部的逻辑难以实现拆分和复用。**如果你想要打破这个僵局，则需要进一步学习更加复杂的设计模式（比如高阶组件、Render Props 等），用更高的学习成本来交换一点点编码的灵活度。</p>
<p data-nodeid="1303">这一切的一切，光是想想就让人头秃。所以说，<strong data-nodeid="1436">类组件固然强大， 但它绝非万能</strong>。</p>
<h3 data-nodeid="1304">深入理解函数组件：呼应 React 设计思想的“轻巧快艇”</h3>
<p data-nodeid="1305">我们再来看这个函数组件的 case：</p>
<pre class="lang-js" data-nodeid="1306"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">DemoFunction</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-keyword">const</span> { text } = props
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"demoFunction"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{`function 组件所接收到的来自外界的文本内容是：[${text}]`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>
<p data-nodeid="1307">当然啦，要是你以为函数组件的简单是因为它只能承担渲染这一种任务，那可就太小瞧它了。它同样能够承接相对复杂的交互逻辑，像这样：</p>
<pre class="lang-js" data-nodeid="1308"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">DemoFunction</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-keyword">const</span> { text } = props 

  <span class="hljs-keyword">const</span> showAlert = <span class="hljs-function"><span class="hljs-params">()</span>=&gt;</span> {
    alert(<span class="hljs-string">`我接收到的文本是<span class="hljs-subst">${text}</span>`</span>)
  } 

  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"demoFunction"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{`function 组件所接收到的来自外界的文本内容是：[${text}]`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{showAlert}</span>&gt;</span>点击弹窗<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>
<p data-nodeid="1309">相比于类组件，函数组件肉眼可见的特质自然包括轻量、灵活、易于组织和维护、较低的学习成本等。这些要素毫无疑问是重要的，它们也确实驱动着 React 团队做出改变。但是除此之外，还有一个非常容易被大家忽视、也极少有人能真正理解到的知识点，我在这里要着重讲一下。这个知识点缘起于 React 作者 Dan 早期特意为类组件和函数组件写过的<a href="https://overreacted.io/how-are-function-components-different-from-classes/" data-nodeid="1443">一篇非常棒的对比文章</a>，这篇文章很长，但是通篇都在论证这一句话：</p>
<blockquote data-nodeid="1310">
<p data-nodeid="1311"><strong data-nodeid="1448">函数组件会捕获 render 内部的状态，这是两类组件最大的不同。</strong></p>
</blockquote>
<p data-nodeid="1312">初读这篇文章时，我像文中的作者一样，感慨 JS 闭包机制竟能给到我们这么重要的解决问题的灵感。但在反复思考过后的现在，我更希望引导我的读者们去认知到这样一件事情——<strong data-nodeid="1454">类组件和函数组件之间，纵有千差万别，但最不能够被我们忽视掉的，是心智模式层面的差异</strong>，是面向对象和函数式编程这两套不同的设计思想之间的差异。</p>
<p data-nodeid="1313">说得更具体一点，<strong data-nodeid="1460">函数组件更加契合 React 框架的设计理念</strong>。何出此言？不要忘了这个赫赫有名的 React 公式：</p>
<p data-nodeid="1314"><img src="https://s0.lgstatic.com/i/image/M00/65/49/CgqCHl-aaQSAFRMAAAA9kdaM6Jw090.png" alt="2.png" data-nodeid="1463"></p>
<p data-nodeid="1315">不夸张地说，<strong data-nodeid="1481">React 组件本身的定位就是函数，一个吃进数据、吐出 UI 的函数</strong>。作为开发者，我们编写的是声明式的代码，而 React 框架的主要工作，就是及时地<strong data-nodeid="1482">把声明式的代码转换为命令式的 DOM 操作，把数据层面的描述映射到用户可见的 UI 变化中去</strong>。这就意味着从原则上来讲，<strong data-nodeid="1483">React 的数据应该总是紧紧地和渲染绑定在一起的</strong>，<strong data-nodeid="1484">而类组件做不到这一点</strong>。</p>
<p data-nodeid="1316">为什么类组件做不到？这里我摘出上述<a href="https://overreacted.io/how-are-function-components-different-from-classes/" data-nodeid="1488">文章</a>中的 Demo，站在一个新的视角来解读一下“**函数组件会捕获 render 内部的状态，这是两类组件最大的不同”**这个结论。首先我们来看这样一个类组件：</p>
<pre class="lang-js" data-nodeid="1317"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ProfilePage</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  showMessage = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    alert(<span class="hljs-string">'Followed '</span> + <span class="hljs-keyword">this</span>.props.user);
  };
  handleClick = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    setTimeout(<span class="hljs-keyword">this</span>.showMessage, <span class="hljs-number">3000</span>);
  };
  render() {
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.handleClick}</span>&gt;</span>Follow<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>;
  }
}
</code></pre>
<p data-nodeid="1318">这个组件返回的是一个按钮，交互内容也很简单：点击按钮后，过 3s，界面上会弹出“Followed xxx”的文案。类似于我们在微博上点击“关注某人”之后弹出的“已关注”这样的提醒。</p>
<p data-nodeid="1319">看起来好像没啥毛病，但是如果你在这个<a href="https://codesandbox.io/s/pjqnl16lm7" data-nodeid="1500">在线 Demo</a>中尝试点击基于类组件形式编写的 ProfilePage 按钮后 3s 内把用户切换为 Sophie，你就会看到如下图所示的效果：</p>
<p data-nodeid="1320"><img src="https://s0.lgstatic.com/i/image/M00/65/3D/Ciqc1F-aaNGAQ0imAAETlLd2DpM458.png" alt="3.png" data-nodeid="1504"></p>
<p data-nodeid="1321">图源：<a href="https://overreacted.io/how-are-function-components-different-from-classes/" data-nodeid="1508">https://overreacted.io/how-are-function-components-different-from-classes/</a></p>
<p data-nodeid="1322">明明我们是在 Dan 的主页点击的关注，结果却提示了“Followed Sophie”！</p>
<p data-nodeid="1323">这个现象必然让许多人感到困惑：user 的内容是通过 props 下发的，props 作为不可变值，为什么会从 Dan 变成 Sophie 呢？</p>
<p data-nodeid="1324">因为<strong data-nodeid="1516">虽然 props 本身是不可变的，但 this 却是可变的，this 上的数据是可以被修改的</strong>，this.props 的调用每次都会获取最新的 props，而这正是 React 确保数据实时性的一个重要手段。</p>
<p data-nodeid="1325">多数情况下，在 React 生命周期对执行顺序的调控下，this.props 和 this.state 的变化都能够和预期中的渲染动作保持一致。但在这个案例中，<strong data-nodeid="1522">我们通过 setTimeout 将预期中的渲染推迟了 3s，打破了 this.props 和渲染动作之间的这种时机上的关联</strong>，进而导致渲染时捕获到的是一个错误的、修改后的 this.props。这就是问题的所在。</p>
<p data-nodeid="1326">但如果我们把 ProfilePage 改造为一个像这样的函数组件：</p>
<pre class="lang-js" data-nodeid="1327"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ProfilePage</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-keyword">const</span> showMessage = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    alert(<span class="hljs-string">'Followed '</span> + props.user);
  };
  <span class="hljs-keyword">const</span> handleClick = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    setTimeout(showMessage, <span class="hljs-number">3000</span>);
  };
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{handleClick}</span>&gt;</span>Follow<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
  );
}
</code></pre>
<p data-nodeid="1328">事情就会大不一样。</p>
<p data-nodeid="1329">props 会在 ProfilePage 函数执行的一瞬间就被捕获，而 props 本身又是一个不可变值，因此<strong data-nodeid="1530">我们可以充分确保从现在开始，在任何时机下读取到的 props，都是最初捕获到的那个 props</strong>。当父组件传入新的 props 来尝试重新渲染 ProfilePage 时，本质上是基于新的 props 入参发起了一次全新的函数调用，并不会影响上一次调用对上一个 props 的捕获。这样一来，我们便确保了渲染结果确实能够符合预期。</p>
<p data-nodeid="5982" class="te-preview-highlight">如果你认真阅读了我前面说过的那些话，相信你现在一定也不仅仅能够充分理解 Dan 所想要表达的“<strong data-nodeid="5992">函数组件会捕获 render 内部的状态</strong>”这个结论，而是能够更进一步地意识到这样一件事情：<strong data-nodeid="5993">函数组件真正地把数据和渲染绑定到了一起</strong>。</p>








<p data-nodeid="1331">经过岁月的洗礼，React 团队显然也认识到了，<strong data-nodeid="1548">函数组件是一个更加匹配其设计理念、也更有利于逻辑拆分与重用的组件表达形式</strong>，接下来便开始“用脚投票”，用实际行动支持开发者编写函数式组件。于是，React-Hooks 便应运而生。</p>
<h3 data-nodeid="1332">Hooks 的本质：一套能够使函数组件更强大、更灵活的“钩子”</h3>
<p data-nodeid="1333">React-Hooks 是什么？它是一套能够使函数组件更强大、更灵活的“钩子”。</p>
<p data-nodeid="1334">前面我们已经说过，函数组件比起类组件“少”了很多东西，比如生命周期、对 state 的管理等。这就给函数组件的使用带来了非常多的局限性，导致我们并不能使用函数这种形式，写出一个真正的全功能的组件。</p>
<p data-nodeid="1335">React-Hooks 的出现，就是为了帮助函数组件补齐这些（相对于类组件来说）缺失的能力。</p>
<p data-nodeid="1336"><strong data-nodeid="1561">如果说函数组件是一台轻巧的快艇，那么 React-Hooks 就是一个内容丰富的零部件箱</strong>。“重装战舰”所预置的那些设备，这个箱子里基本全都有，同时它还不强制你全都要，而是<strong data-nodeid="1562">允许你自由地选择和使用你需要的那些能力</strong>，然后将这些能力以 Hook（钩子）的形式“钩”进你的组件里，从而定制出一个最适合你的“专属战舰”。</p>
<h3 data-nodeid="1337">总结</h3>
<p data-nodeid="1338">行文至此，关于“Why”的研究已经基本到位，对于“What”的认知也已经初见眉目。虽然本课时并没有贴上哪怕一行 React-Hooks 相关的代码，但我相信，你对 React-Hooks 本质的把握已经超越了非常多的 React 开发者。</p>
<p data-nodeid="1339" class="">在下个课时，我们将会和 React-Hooks 面对面交锋，从编码层面上认知“What”，从实践角度理解“How”。相信在课时的最后，你会对本文所讲解的“Why”有更深刻的理解和感悟。</p>

---

### 精选评论

##### **峰：
> 被程序耽误了的作家？文笔和文风太赞了

##### **明：
> 真的很棒。目前看过关于react最好的课程。

##### QYM：
> 意犹未尽啊～

##### **豪：
> 一元的课真香

##### leoge：
> 视频中提到Class模式下，this发生了变化，导致follow的名字对不上。我的理解this是不会变的（实例并未重新生成），变的是传进来的props。如下，绑定到user就可以了保持一致了。showMessage = (user) = {    alert('Followed ' + user);  };handleClick = () = {    const {user} = this.props;    setTimeout(() = this.showMessage(user), 3000);  };

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这里想要表达的就是 this 上的 props 可以被更改。

##### **坤：
> 老师说：几乎没有几个人在聊到 React-Hooks 的时候，能像聊 Diff 算法、Fiber 架构一样滔滔不绝、言之有物。 我想说， Diff算法、Fiber架构我也不知道聊啥，咋整

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 继续往后看。第10讲以栈调和为例分析了 Diff 的逻辑，13-16讲结合源码分析了 Fiber 架构的实现原理。

##### *翀：
> 杠精一下，针对老师的例子提供另一个思路。Follow按钮这个例子通过箭头函数也能避免例子中出现的问题。代码如下：import React from 'react';class ProfilePage extends React.Component {  showMessage = (name) = {    alert('Followed ' + name);  };  handleClick = (name) = {    setTimeout(()=this.showMessage(name), 3000);  };  render() {    return button onClick={()= this.handleClick(this.props.user)}/button;  }}export default ProfilePage;

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 读完下个课时应该能够化解你的执着，哈哈

##### **喜：
> 讲的非常好，有种醍醐灌顶的感觉

##### **华：
> 虽然 props 本身是不可变的 ,这句话应该怎么理解呀老师？为什么说props本身不可变呢

##### **文：
> 讲的太棒了

##### **坚：
> 技术和文笔深的我爱呀

##### *辉：
> 打卡

##### **杰：
> 老师声音挺好听

##### *浩：
> 文中列举的demo虽然实际场景有很大概率不会出现，但问题的提出也倒是让人耳目一新。

##### **威：
> 老师写的真好，一篇文章顶十本书

##### **魁：
> 嗯，后端看了也能明白

##### **贵：
> 打卡

##### **舟：
> 太棒了

##### **文：
> 讲的真好

##### **娇：
> 真香，打卡

##### Loktar：
> 所以函数组件本质就是将类方法render提取出来扩展吗？可以理解JS的闭包机制让函数组件捕获特定的渲染帧，但老想不明白hooks是怎么实现的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 08讲对 hooks 核心原理进行了分析哈，可以去看看。

##### fanta：
> 辛苦了老师😁

##### *菜：
> 老师讲的太好了

##### *林：
> 真香

##### **论：
> 打卡

##### **森：
> 打卡

##### symboll：
> 一元的课真香

##### **无限：
> 讲的很清楚

##### **飞：
> 非常好

##### **松：
> 打卡

##### **卿：
> 打卡

