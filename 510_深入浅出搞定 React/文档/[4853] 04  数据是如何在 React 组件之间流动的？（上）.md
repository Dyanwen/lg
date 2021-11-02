<p data-nodeid="12983" class="">通过前面 3 个课时的学习，相信你已经对 React 生命周期相关的“Why”“What”和“How”有了系统的理解和掌握。当我们谈论生命周期时，其实谈论的是组件的“内心世界”。但组件和人是一样的，它不仅需要拥有丰富的内心世界，还应该建立健全的“人际关系”，要学会<strong data-nodeid="13143">沟通和表达</strong>。</p>
<p data-nodeid="12984">从本课时开始，我们将一起探索蕴含在 React 组件中的“沟通与表达”的艺术。我们知道，React 的核心特征是“<strong data-nodeid="13149">数据驱动视图</strong>”，这个特征在业内有一个非常有名的函数式来表达：</p>
<p data-nodeid="12985"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-OmrSAZkEwAAA2ThydXNs410.png" alt="Drawing 1.png" data-nodeid="13152"></p>
<p data-nodeid="12986">这个表达式有很多的版本，一些版本会把入参里的 data 替换成 state，但它们本质上都指向同一个含义，那就是<strong data-nodeid="13158">React 的视图会随着数据的变化而变化</strong>。数据这个角色在 React 中的地位可见一斑。</p>
<p data-nodeid="12987">在 React 中，如果说两个组件之间希望能够产生“耦合”（即 A 组件希望能够通过某种方式影响到 B 组件），那么毫无疑问，这两个组件必须先建立数据上的连接，以实现所谓的“组件间通信”。</p>
<p data-nodeid="12988">“组件间通信”的背后是一套环环相扣的 React 数据流解决方案。虽然这套解决方案在业内已经有了比较成熟和稳定的结论，但<strong data-nodeid="13165">许多人仍然会因为知识的系统性和整体性不强而吃亏</strong>。</p>
<p data-nodeid="12989"><strong data-nodeid="13170">在前面三个课时中，我们的学习思路是往纵深处去寻觅：铺陈大量的前置知识，然后一步一步地去询问生命周期背后的“Why”，最终揪出 Fiber 架构这个大 boss</strong>（不过学到这里，这个“纵深”我们才只挖到一半，专栏第二模块还有一大波 Fiber 原理等待我们继续寻觅）。</p>
<p data-nodeid="12990">在接下来的第 04 和 05 课时中，我们要做的事情则更倾向于横向的“聚合”：我将用简单易懂的语言，帮你理解当下实践中 React 数据通信的四个大方向，并针对每个方向给出具体的场景和用例。<strong data-nodeid="13176">这些知识本身并不难，但摊子却可以铺得非常大，相关的问题在面试中也始终具备较高的区分度</strong>。要想扎扎实实掌握，必须耐下心、沉住气，在学习过程中主动地去串联自己的知识链路。</p>
<h3 data-nodeid="12991">基于 props 的单向数据流</h3>
<p data-nodeid="12992"><img src="https://s0.lgstatic.com/i/image/M00/61/02/CgqCHl-OmsuAF_FSAAB4ormSPI8355.png" alt="Drawing 2.png" data-nodeid="13180"></p>
<p data-nodeid="12993">既然 props 是组件的入参，那么组件之间通过修改对方的入参来完成数据通信就是天经地义的事情了。不过，这个“修改”也是有原则的——你必须确保所有操作都在“<strong data-nodeid="13186">单向数据流</strong>”这个前提下。</p>
<p data-nodeid="12994">所谓单向数据流，指的就是当前组件的 state 以 props 的形式流动时，<strong data-nodeid="13192">只能流向组件树中比自己层级更低的组件。</strong> 比如在父-子组件这种嵌套关系中，只能由父组件传 props 给子组件，而不能反过来。</p>
<p data-nodeid="12995">听上去虽然限制重重，但用起来却是相当的灵活。基于 props 传参这种形式，我们可以轻松实现父-子通信、子-父通信和兄弟组件通信。</p>
<h4 data-nodeid="12996">父-子组件通信</h4>
<p data-nodeid="12997"><strong data-nodeid="13198">原理讲解</strong></p>
<p data-nodeid="12998">这是最常见、也是最好解决的一个通信场景。React 的数据流是单向的，父组件可以直接将 this.props 传入子组件，实现父-子间的通信。这里我给出一个示例。</p>
<p data-nodeid="12999"><strong data-nodeid="13203">编码实现</strong></p>
<ul data-nodeid="13000">
<li data-nodeid="13001">
<p data-nodeid="13002">子组件编码内容：</p>
</li>
</ul>
<pre class="lang-js" data-nodeid="13003"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">Child</span>(<span class="hljs-params">props</span>) </span>{
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"child"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{`子组件所接收到的来自父组件的文本内容是：[${props.fatherText}]`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>
<ul data-nodeid="13004">
<li data-nodeid="13005">
<p data-nodeid="13006">父组件编码内容：</p>
</li>
</ul>
<pre class="lang-js" data-nodeid="13007"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Father</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 初始化父组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"初始化的父组件的文本"</span>
  };
  <span class="hljs-comment">// 按钮的监听函数，用于更新 text 值</span>
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: <span class="hljs-string">"改变后的父组件文本"</span>
    });
  };
  <span class="hljs-comment">// 渲染父组件</span>
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"father"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span>&gt;</span>
          点击修改父组件传入子组件的文本
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        {/* 引入子组件，并通过 props 下发具体的状态值实现父-子通信 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">Child</span> <span class="hljs-attr">fatherText</span>=<span class="hljs-string">{this.state.text}</span> /&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="13008"><strong data-nodeid="13209">视图层验证</strong></p>
<p data-nodeid="13009">我们直接对父组件进行渲染，可以看到大致如下图所示的界面：</p>
<p data-nodeid="13010"><img src="https://s0.lgstatic.com/i/image/M00/61/02/CgqCHl-OmuWAaqeoAABhmFu-VMo782.png" alt="Drawing 3.png" data-nodeid="13213"></p>
<p data-nodeid="13011">通过子组件顺利读取到父组件的 this.props.text，从这一点可以看出，父-子之间的通信是没有问题的。此时假如我们点击父组件中的按钮，父组件的 this.state.text 会发生变化，同时子组件读取到的 props.text 也会跟着发生变化（如下图所示），也就是说，父子组件的数据始终保持一致。</p>
<p data-nodeid="13012"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-Omu-AcVKJAABe2pgKMlQ354.png" alt="Drawing 4.png" data-nodeid="13217"></p>
<p data-nodeid="13013">由此我们便充分验证了父-子组件基于 props 实现通信的可行性。</p>
<h4 data-nodeid="13014">子-父组件通信</h4>
<p data-nodeid="13015"><strong data-nodeid="13223">原理讲解</strong></p>
<p data-nodeid="13016">考虑到 props 是单向的，子组件并不能直接将自己的数据塞给父组件，但 props 的形式也可以是多样的。假如父组件传递给子组件的是一个<strong data-nodeid="13233">绑定了自身上下文的函数</strong>，那么子组件在调用该函数时，就可以<strong data-nodeid="13234">将想要交给父组件的数据以函数入参的形式给出去</strong>，以此来间接地实现数据从子组件到父组件的流动。</p>
<p data-nodeid="13017"><strong data-nodeid="13238">编码实现</strong></p>
<p data-nodeid="13018">这里我们只需对父-子通信中的示例稍做修改，就可以完成子-父组件通信的可行性验证。</p>
<p data-nodeid="13019">首先是对子组件的修改。在 Child 中，我们需要增加对状态的维护，以及对 Father 组件传入的函数形式入参的调用。子组件编码内容如下，修改点我已在代码中以注释的形式标出：</p>
<pre class="lang-js" data-nodeid="13020"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Child</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 初始化子组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">'子组件的文本'</span>
  }

  <span class="hljs-comment">// 子组件的按钮监听函数</span>
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-comment">// changeText 中，调用了父组件传入的 changeFatherText 方法</span>
    <span class="hljs-keyword">this</span>.props.changeFatherText(<span class="hljs-keyword">this</span>.state.text)
  }
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"child"</span>&gt;</span>
        {/* 注意这里把修改父组件文本的动作放在了 Child 里 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span>&gt;</span>
          点击更新父组件的文本
        <span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="13021">在父组件中，我们只需要在 changeText 函数上开一个传参的口子，作为数据通信的入口，然后把 changeText 放在 props 里交给子组件即可。父组件的编码内容如下：</p>
<pre class="lang-js" data-nodeid="13022"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Father</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 初始化父组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"初始化的父组件的文本"</span>
  };
  <span class="hljs-comment">// 这个方法会作为 props 传给子组件，用于更新父组件 text 值。newText 正是开放给子组件的数据通信入口</span>
  changeText = <span class="hljs-function">(<span class="hljs-params">newText</span>) =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: newText
    });
  };
  <span class="hljs-comment">// 渲染父组件</span>
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"father"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{`父组件的文本内容是：[${this.state.text}]`}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        {/* 引入子组件，并通过 props 中下发可传参的函数 实现子-父通信 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">Child</span>
          <span class="hljs-attr">changeFatherText</span>=<span class="hljs-string">{this.changeText}</span>
        /&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
</code></pre>
<p data-nodeid="13023"><strong data-nodeid="13245">视图层验证</strong></p>
<p data-nodeid="13024">新的示例渲染后的界面大致如下图所示：</p>
<p data-nodeid="13025"><img src="https://s0.lgstatic.com/i/image/M00/61/03/CgqCHl-Om1qAWYoYAABEbXaJOH4748.png" alt="Drawing 5.png" data-nodeid="13249"></p>
<p data-nodeid="13026">注意，在这个 case 中，我们将具有更新数据能力的按钮转移到了子组件中。</p>
<p data-nodeid="13027">当点击子组件中的按钮时，会调用已经绑定了父组件上下文的 this.props.changeFatherText 方法，同时将子组件的 this.state.text 以函数入参的形式传入，由此便能够间接地用子组件的 state 去更新父组件的 state。</p>
<p data-nodeid="13028">点击按钮后，父组件的文本会按照我们的预期被子组件更新掉，如下图所示：</p>
<p data-nodeid="13029"><img src="https://s0.lgstatic.com/i/image/M00/61/03/CgqCHl-Om1KAR7b2AABAwOe1KdQ729.png" alt="Drawing 6.png" data-nodeid="13255"></p>
<h4 data-nodeid="13030">兄弟组件通信</h4>
<p data-nodeid="13031"><strong data-nodeid="13260">原理讲解</strong></p>
<p data-nodeid="13032">兄弟组件之间共享了同一个父组件，如下图所示，这是一个非常重要的先决条件。</p>
<p data-nodeid="13033"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-Om2qAJmdoAADBknkoDh4735.png" alt="Drawing 8.png" data-nodeid="13264"></p>
<p data-nodeid="13034">这个先决条件使得我们可以继续利用父子组件这一层关系，将“兄弟 1 → 兄弟 2”之间的通信，转化为“兄弟 1 → 父组件”（子-父通信）、“父组件 → 兄弟 2”（父-子）通信两个步骤，如下图所示，这样一来就能够巧妙地把“兄弟”之间的新问题化解为“父子”之间的旧问题。</p>
<p data-nodeid="13035"><img src="https://s0.lgstatic.com/i/image/M00/61/03/CgqCHl-Om3KAMCvhAADUh2BcieU209.png" alt="Drawing 10.png" data-nodeid="13268"></p>
<p data-nodeid="13036"><strong data-nodeid="13272">编码实现</strong></p>
<p data-nodeid="13037">接下来我们仍然从编码的角度进行验证。首先新增一个 NewChild 组件作为与 Child 组件同层级的兄弟组件。NewChild 将作为数据的发送方，将数据发送给 Child。在 NewChild 中，我们需要处理 NewChild 和 Father 之间的关系。NewChild 组件编码如下：</p>
<pre class="lang-js" data-nodeid="13038"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">NewChild</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"来自 newChild 的文本"</span>
  };
  <span class="hljs-comment">// NewChild 组件的按钮监听函数</span>
  changeText = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-comment">// changeText 中，调用了父组件传入的 changeFatherText 方法</span>
    <span class="hljs-keyword">this</span>.props.changeFatherText(<span class="hljs-keyword">this</span>.state.text);
  };
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"child"</span>&gt;</span>
        {/* 注意这里把修改父组件文本（同时也是 Child 组件的文本）的动作放在了 NewChild 里 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.changeText}</span>&gt;</span>点击更新 Child 组件的文本<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="13039">接下来看看 Father 组件。在 Father 组件中，我们通过 text 属性连接 Father 和 Child，通过 changeText 函数来连接 Father 和 NewChild。由此便把 text 属性的渲染工作交给了 Child，把 text 属性的更新工作交给 NewÇhild，以此来实现数据从 NewChild 到 Child 的流动。Father 组件编码如下：</p>
<pre class="lang-js" data-nodeid="13040"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Father</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 初始化父组件的 state</span>
  state = {
    <span class="hljs-attr">text</span>: <span class="hljs-string">"初始化的父组件的文本"</span>
  };
  <span class="hljs-comment">// 传给 NewChild 组件按钮的监听函数，用于更新父组件 text 值（这个 text 值同时也是 Child 的 props）</span>
  changeText = <span class="hljs-function">(<span class="hljs-params">newText</span>) =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">text</span>: newText
    });
  };
  <span class="hljs-comment">// 渲染父组件</span>
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"father"</span>&gt;</span>
        {/* 引入 Child 组件，并通过 props 中下发具体的状态值 实现父-子通信 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">Child</span> <span class="hljs-attr">fatherText</span>=<span class="hljs-string">{this.state.text}</span> /&gt;</span>
        {/* 引入 NewChild 组件，并通过 props 中下发可传参的函数 实现子-父通信 */}
        <span class="hljs-tag">&lt;<span class="hljs-name">NewChild</span> <span class="hljs-attr">changeFatherText</span>=<span class="hljs-string">{this.changeText}</span> /&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="13041"><strong data-nodeid="13278">视图层验证</strong></p>
<p data-nodeid="13042">编码完成之后，界面大致的结构如下图所示：</p>
<p data-nodeid="13043"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-Om4CAaYYmAABgp6tGilY796.png" alt="Drawing 11.png" data-nodeid="13282"></p>
<p data-nodeid="13044">由于整体结构稍微复杂了一些，这里我把 Father、Child 和 NewChild 在图中的大致范围标一下：</p>
<p data-nodeid="13045"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-Om4eAFvb7AACD10q8ddE601.png" alt="Drawing 12.png" data-nodeid="13286"></p>
<ul data-nodeid="13046">
<li data-nodeid="13047">
<p data-nodeid="13048">红色所圈范围为 Father 组件，它包括了 Child 和 NewChild；</p>
</li>
<li data-nodeid="13049">
<p data-nodeid="13050">灰色圈住的按钮是 NewChild 组件的渲染结果，它可以触发数据的改变；</p>
</li>
<li data-nodeid="13051">
<p data-nodeid="13052">蓝色圈住的文本是 Child 组件的渲染结果，它负责感知和渲染数据。</p>
</li>
</ul>
<p data-nodeid="13053">现在我点击位于 NewChild 组件中的“点击更新 Child 组件的文本”按钮，就可以看到 Child 会跟着发生变化，如下图所示，进而验证方案的可行性。</p>
<p data-nodeid="13054"><img src="https://s0.lgstatic.com/i/image/M00/60/F7/Ciqc1F-Om5GAagXzAAIiBlsRdIM293.png" alt="Drawing 14.png" data-nodeid="13293"></p>
<h4 data-nodeid="13055">为什么不推荐用 props 解决其他场景的需求</h4>
<p data-nodeid="13056">至此，我们给出了 props 传参这种形式比较适合处理的三种场景。尽管这并不意味着其他场景不能用 props 处理，但如果你试图用简单的 props 传递完成更加复杂的通信需求，往往会得不偿失。这里我给你举一个比较极端的例子：</p>
<p data-nodeid="13057"><img src="https://s0.lgstatic.com/i/image/M00/60/F8/Ciqc1F-Om5iAAUUhAABLimeJTao712.png" alt="Drawing 16.png" data-nodeid="13298"></p>
<p data-nodeid="13058">如上图所示，可以看到这是一个典型的多层嵌套组件结构。A 组件倘若想要和层层相隔的 E 组件实现通信，就必须把 props 经过 B、C、D 一层一层地传递下去。在这个过程中，反反复复的 props 传递不仅会带来庞大的工作量和代码量，还会污染中间无辜的 B、C、D 组件的属性结构。</p>
<p data-nodeid="13059">层层传递的优点是非常简单，用已有知识就能解决，但问题是会浪费很多代码，非常烦琐，中间作为桥梁的组件会引入很多不属于自己的属性。短期来看，写代码的人会很痛苦；长期来看，整个项目的维护成本都会变得非常高昂。因此，<strong data-nodeid="13305">层层传递 props 要不得</strong>。</p>
<p data-nodeid="13060">那有没有更加灵活的解决方案，能够帮我们处理“任意组件”之间的通信需求呢？答案是不仅有，而且姿势还很多。我先从最朴素的“发布-订阅”模式讲起。</p>
<h3 data-nodeid="13061">利用“发布-订阅”模式驱动数据流</h3>
<p data-nodeid="13062">“发布-订阅”模式可谓是解决通信类问题的“万金油”，在前端世界的应用非常广泛，比如：</p>
<ul data-nodeid="13063">
<li data-nodeid="13064">
<p data-nodeid="13065">前两年爆火的 socket.io 模块，它就是一个典型的跨端发布-订阅模式的实现；</p>
</li>
<li data-nodeid="13066">
<p data-nodeid="13067">在 Node.js 中，许多原生模块也是以 EventEmitter 为基类实现的；</p>
</li>
<li data-nodeid="13068">
<p data-nodeid="13069">不过大家最为熟知的，应该还是 Vue.js 中作为常规操作被推而广之的“全局事件总线” EventBus。</p>
</li>
</ul>
<p data-nodeid="13070">这些应用之间虽然名字各不相同，但内核是一致的，也就是我们下面要讲到的“发布-订阅”模型。</p>
<h4 data-nodeid="13071">理解事件的发布-订阅机制</h4>
<p data-nodeid="13072">发布-订阅机制早期最广泛的应用，应该是在浏览器的 DOM 事件中。 &nbsp;相信有过原生 JavaScript 开发经验的同学，对下面这样的用法都不会陌生：</p>
<pre class="lang-js" data-nodeid="13073"><code data-language="js">target.addEventListener(type, listener, useCapture);
</code></pre>
<p data-nodeid="13074">通过调用 addEventListener 方法，我们可以创建一个事件监听器，这个动作就是“订阅”。比如我可以监听 click（点击）事件：</p>
<pre class="lang-java" data-nodeid="13075"><code data-language="java">el.addEventListener(<span class="hljs-string">"click"</span>, func, <span class="hljs-keyword">false</span>);
</code></pre>
<p data-nodeid="13076">这样一来，当 click 事件被触发时，事件会被“发布”出去，进而触发监听这个事件的 func 函数。这就是一个最简单的发布-订阅案例。</p>
<p data-nodeid="13077">使用发布-订阅模式的优点在于，<strong data-nodeid="13322">监听事件的位置和触发事件的位置是不受限的</strong>，就算相隔十万八千里，只要它们在同一个上下文里，就能够彼此感知。这个特性，太适合用来应对“任意组件通信”这种场景了。</p>
<h4 data-nodeid="13078">发布-订阅模型 API 设计思路</h4>
<p data-nodeid="13079">通过前面的讲解，不难看出发布-订阅模式中有两个关键的动作：<strong data-nodeid="13329">事件的监听（订阅）和事件的触发（发布）</strong>，这两个动作自然而然地对应着两个基本的 API 方法。</p>
<ul data-nodeid="13080">
<li data-nodeid="13081">
<p data-nodeid="13082">on()：负责注册事件的监听器，指定事件触发时的回调函数。</p>
</li>
<li data-nodeid="13083">
<p data-nodeid="13084">emit()：负责触发事件，可以通过传参使其在触发的时候携带数据 。</p>
</li>
</ul>
<p data-nodeid="13085">最后，只进不出总是不太合理的，我们还要考虑一个 off() 方法，必要的时候用它来删除用不到的监听器：</p>
<ul data-nodeid="13086">
<li data-nodeid="13087">
<p data-nodeid="13088">off()：负责监听器的删除。</p>
</li>
</ul>
<h4 data-nodeid="13089">发布-订阅模型编码实现</h4>
<p data-nodeid="13090">“发布-订阅”模式不仅在应用层面十分受欢迎，它更是面试官的心头好。在涉及设计模式的面试中，如果只允许出一道题，那么我相信大多数的面试官都会和我一样，会毫不犹豫地选择考察“发布-订阅模式的实现”。 接下来我就手把手带你来做这道题，写出一个同时拥有 on、emit 和 off 的 EventEmitter。</p>
<p data-nodeid="13091">在写代码之前，先要捋清楚思路。这里我把“实现 EventEmitter”这个大问题，拆解为 3 个具体的小问题，下面我们逐个来解决。</p>
<ul data-nodeid="13092">
<li data-nodeid="13093">
<p data-nodeid="13094"><strong data-nodeid="13340">问题一：事件和监听函数的对应关系如何处理？</strong></p>
</li>
</ul>
<p data-nodeid="13095">提到“对应关系”，应该联想到的是“映射”。在 JavaScript 中，处理“映射”我们大部分情况下都是用对象来做的。所以说在全局我们需要设置一个对象，来存储事件和监听函数之间的关系：</p>
<pre class="lang-java" data-nodeid="13096"><code data-language="java">constructor() {
  <span class="hljs-comment">// eventMap 用来存储事件和监听函数之间的关系</span>
  <span class="hljs-keyword">this</span>.eventMap= {}
}
</code></pre>
<ul data-nodeid="13097">
<li data-nodeid="13098">
<p data-nodeid="13099"><strong data-nodeid="13345">问题二：如何实现订阅？</strong></p>
</li>
</ul>
<p data-nodeid="16627" class="">所谓“订阅”，也就是注册事件监听函数的过程。这是一个“写”操作，具体来说就是把事件和对应的监听函数写入到 eventMap 里面去：</p>





<pre class="lang-java te-preview-highlight" data-nodeid="38471"><code data-language="java"><span class="hljs-comment">// type 这里就代表事件的名称</span>
on(type, handler) {
  <span class="hljs-comment">// hanlder 必须是一个函数，如果不是直接报错</span>
  <span class="hljs-keyword">if</span>(!(handler <span class="hljs-keyword">instanceof</span> Function)) {
    <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> Error(<span class="hljs-string">"哥 你错了 请传一个函数"</span>)
  }
  <span class="hljs-comment">// 判断 type 事件对应的队列是否存在</span>
  <span class="hljs-keyword">if</span>(!<span class="hljs-keyword">this</span>.eventMap[type]) {
   <span class="hljs-comment">// 若不存在，新建该队列</span>
    <span class="hljs-keyword">this</span>.eventMap[type] = []
  }
  <span class="hljs-comment">// 若存在，直接往队列里推入 handler</span>
  <span class="hljs-keyword">this</span>.eventMap[type].push(handler)
}
</code></pre>



























<ul data-nodeid="13102">
<li data-nodeid="13103">
<p data-nodeid="13104"><strong data-nodeid="13350">问题三：如何实现发布？</strong></p>
</li>
</ul>
<p data-nodeid="13105">订阅操作是一个“写”操作，相应的，发布操作就是一个“读”操作。发布的本质是触发安装在某个事件上的监听函数，我们需要做的就是找到这个事件对应的监听函数队列，将队列中的 handler 依次执行出队：</p>
<pre class="lang-java" data-nodeid="13106"><code data-language="java"><span class="hljs-comment">// 别忘了我们前面说过触发时是可以携带数据的，params 就是数据的载体</span>
emit(type, params) {
  <span class="hljs-comment">// 假设该事件是有订阅的（对应的事件队列存在）</span>
  <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>.eventMap[type]) {
    <span class="hljs-comment">// 将事件队列里的 handler 依次执行出队</span>
    <span class="hljs-keyword">this</span>.eventMap[type].forEach((handler, index)=&gt; {
      <span class="hljs-comment">// 注意别忘了读取 params</span>
      handler(params)
    })
  }
}
</code></pre>
<p data-nodeid="13107">到这里，最最关键的 on 方法和 emit 方法就实现完毕了。最后我们补充一个 off 方法：</p>
<pre class="lang-java" data-nodeid="13108"><code data-language="java">off(type, handler) {
  <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>.eventMap[type]) {
    <span class="hljs-keyword">this</span>.eventMap[type].splice(<span class="hljs-keyword">this</span>.eventMap[type].indexOf(handler)&gt;&gt;&gt;<span class="hljs-number">0</span>,<span class="hljs-number">1</span>)
  }
}
</code></pre>
<p data-nodeid="13109">接着把这些代码片段拼接进一个 class 里面，一个核心功能完备的 EventEmitter 就完成啦：</p>
<pre class="lang-java" data-nodeid="13110"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">myEventEmitter</span> </span>{
  constructor() {
    <span class="hljs-comment">// eventMap 用来存储事件和监听函数之间的关系</span>
    <span class="hljs-keyword">this</span>.eventMap = {};
  }
  <span class="hljs-comment">// type 这里就代表事件的名称</span>
  on(type, handler) {
    <span class="hljs-comment">// hanlder 必须是一个函数，如果不是直接报错</span>
    <span class="hljs-keyword">if</span> (!(handler <span class="hljs-keyword">instanceof</span> Function)) {
      <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> Error(<span class="hljs-string">"哥 你错了 请传一个函数"</span>);
    }
    <span class="hljs-comment">// 判断 type 事件对应的队列是否存在</span>
    <span class="hljs-keyword">if</span> (!<span class="hljs-keyword">this</span>.eventMap[type]) {
      <span class="hljs-comment">// 若不存在，新建该队列</span>
      <span class="hljs-keyword">this</span>.eventMap[type] = [];
    }
    <span class="hljs-comment">// 若存在，直接往队列里推入 handler</span>
    <span class="hljs-keyword">this</span>.eventMap[type].push(handler);
  }
  <span class="hljs-comment">// 别忘了我们前面说过触发时是可以携带数据的，params 就是数据的载体</span>
  emit(type, params) {
    <span class="hljs-comment">// 假设该事件是有订阅的（对应的事件队列存在）</span>
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.eventMap[type]) {
      <span class="hljs-comment">// 将事件队列里的 handler 依次执行出队</span>
      <span class="hljs-keyword">this</span>.eventMap[type].forEach((handler, index) =&gt; {
        <span class="hljs-comment">// 注意别忘了读取 params</span>
        handler(params);
      });
    }
  }
  off(type, handler) {
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.eventMap[type]) {
      <span class="hljs-keyword">this</span>.eventMap[type].splice(<span class="hljs-keyword">this</span>.eventMap[type].indexOf(handler) &gt;&gt;&gt; <span class="hljs-number">0</span>, <span class="hljs-number">1</span>);
    }
  }
}
</code></pre>
<p data-nodeid="13111">下面我们对 myEventEmitter 进行一个简单的测试，创建一个 myEvent 对象作为 myEventEmitter 的实例，然后针对名为 “test” 的事件进行监听和触发：</p>
<pre class="lang-java" data-nodeid="13112"><code data-language="java"><span class="hljs-comment">// 实例化 myEventEmitter</span>
<span class="hljs-keyword">const</span> myEvent = <span class="hljs-keyword">new</span> myEventEmitter();
<span class="hljs-comment">// 编写一个简单的 handler</span>
<span class="hljs-keyword">const</span> testHandler = function (params) {
  console.log(`test事件被触发了，testHandler 接收到的入参是${params}`);
};
<span class="hljs-comment">// 监听 test 事件</span>
myEvent.on(<span class="hljs-string">"test"</span>, testHandler);
<span class="hljs-comment">// 在触发 test 事件的同时，传入希望 testHandler 感知的参数</span>
myEvent.emit(<span class="hljs-string">"test"</span>, <span class="hljs-string">"newState"</span>);
</code></pre>
<p data-nodeid="13113">以上代码会输出下面红色矩形框住的部分作为运行结果：</p>
<p data-nodeid="13114"><img src="https://s0.lgstatic.com/i/image/M00/60/F8/Ciqc1F-Om7eAC75dAAMfTZMkn3A636.png" alt="Drawing 17.png" data-nodeid="13358"></p>
<p data-nodeid="13115">由此可以看出，EventEmitter 的实例已经具备发布-订阅的能力，执行结果符合预期。</p>
<p data-nodeid="13116">现在你可以试想一下，对于任意的两个组件 A 和 B，假如我希望实现双方之间的通信，借助 EventEmitter 来做就很简单了，以数据从 A 流向 B 为例。</p>
<p data-nodeid="13117">我们可以在 B 中编写一个handler（记得将这个 handler 的 this 绑到 B 身上），在这个 handler 中进行以 B 为上下文的 this.setState 操作，然后将这个 handler 作为监听器与某个事件关联起来。比如这样：</p>
<pre class="lang-js" data-nodeid="13118"><code data-language="js"><span class="hljs-comment">// 注意这个 myEvent 是提前实例化并挂载到全局的，此处不再重复示范实例化过程</span>
<span class="hljs-keyword">const</span> globalEvent = <span class="hljs-built_in">window</span>.myEvent
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">B</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 这里省略掉其他业务逻辑</span>
  state = {
    <span class="hljs-attr">newParams</span>: <span class="hljs-string">""</span>
  };
  handler = <span class="hljs-function">(<span class="hljs-params">params</span>) =&gt;</span> {
    <span class="hljs-keyword">this</span>.setState({
      <span class="hljs-attr">newParams</span>: params
    });
  };
  bindHandler = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    globalEvent.on(<span class="hljs-string">"someEvent"</span>, <span class="hljs-keyword">this</span>.handler);
  };
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.bindHandler}</span>&gt;</span>点我监听A的动作<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>A传入的内容是[{this.state.newParams}]<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}
</code></pre>
<p data-nodeid="13119">接下来在 A 组件中，只需要直接触发对应的事件，然后将希望携带给 B 的数据作为入参传递给 emit 方法即可。代码如下：</p>
<pre class="lang-js" data-nodeid="13120"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">A</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  <span class="hljs-comment">// 这里省略掉其他业务逻辑</span>
  state = {
    <span class="hljs-attr">infoToB</span>: <span class="hljs-string">"哈哈哈哈我来自A"</span>
  };
  reportToB = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
    <span class="hljs-comment">// 这里的 infoToB 表示 A 自身状态中需要让 B 感知的那部分数据</span>
    globalEvent.emit(<span class="hljs-string">"someEvent"</span>, <span class="hljs-keyword">this</span>.state.infoToB);
  };
  render() {
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{this.reportToB}</span>&gt;</span>点我把state传递给B<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>;
  }
}
</code></pre>
<p data-nodeid="13121">如此一来，便能够实现 A 到 B 的通信了。这里我将 A 与 B 编排为兄弟组件，代码如下：</p>
<pre class="lang-js" data-nodeid="13122"><code data-language="js"><span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">App</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">B</span> /&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">A</span> /&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
}
</code></pre>
<p data-nodeid="13123">你也可以在自己的 Demo 里将 A 和 B 定义为更加复杂的嵌套关系，这里我给出的这个 Demo 运行起来会渲染出这样的界面，如下图所示：</p>
<p data-nodeid="13124"><img src="https://s0.lgstatic.com/i/image/M00/61/03/CgqCHl-Om8uAMxXIAAA9zJmLTSg441.png" alt="Drawing 18.png" data-nodeid="13367"></p>
<p data-nodeid="13125" class="">依次点击顶部和底部的按钮，就可以实现对 someEvent 这个事件的监听和触发，进而观察到中间这行文本的改变，如下图所示：</p>
<p data-nodeid="13126"><img src="https://s0.lgstatic.com/i/image/M00/60/F8/Ciqc1F-Om9CAUv1uAABH1-iBy-U054.png" alt="Drawing 19.png" data-nodeid="13371"></p>
<p data-nodeid="13127">由此我们便可以验证到发布-订阅模式驱动 React 数据流的可行性。为了强化你对过程的理解，我将 A 与 B 的通信过程梳理进了一张图里，供你参考：</p>
<p data-nodeid="13128"><img src="https://s0.lgstatic.com/i/image/M00/6E/B0/Ciqc1F-zbFSAa6tMAAC2rDdKPpI299.png" alt="Lark20201117-141619.png" data-nodeid="13375"></p>
<h3 data-nodeid="13129">总结</h3>
<p data-nodeid="13130">本课时，我们对 React 数据流管理方案中的前两个大方向进行了学习：</p>
<ul data-nodeid="13131">
<li data-nodeid="13132">
<p data-nodeid="13133">使用基于 Props 的单向数据流串联父子、兄弟组件；</p>
</li>
<li data-nodeid="13134">
<p data-nodeid="13135">利用“发布-订阅”模式驱动 React 数据在任意组件间流动。</p>
</li>
</ul>
<p data-nodeid="13136">这两个方向下的解决方案，单纯从理解上来看，难度都不高。<strong data-nodeid="13385">你需要把重点放在对编码的实现和理解上，尤其是基于“发布-订阅”模式实现的 EventEmitter，多年来一直是面试的大热点，务必要好好把握</strong>。</p>
<p data-nodeid="13137" class="">这一课时就讲到这里了，下个课时，我们将继续站在“数据在 React 组件中的流动”这个视角，对 React 中的 Context API，以及第三方数据流管理框架中的“佼佼者” Redux 进行分析，相信一定能够为你带来不一样的理解和收获。</p>

---

### 精选评论

##### Loktar：
> 老哥这个  右移符号用的妙啊😀

##### **峰：
> 我来解释一下 off 方法中无符号右移的作用吧，这里是为了处理传入一个事件队列中不存在的函数时，不会意外的移除掉，我们知道 splice 的第一个参数是负数时，会从数组的最后往前找。试想一下，如果传入一个不存在的函数给 off 方法，indexOf 找不到会返回 -1 ，再调用 splice 就会将队列中最后一个函数删除掉了。而使用无符号右移，-1 无符号右移的结果为 4294967295，这个数足够大，不会对原队列造成影响，就很秒 )：

##### **用户7412：
> 这个无符号右移，之前都没用过，二进制的转码看了一下也是一知半解，不过这地方之所以能用是因为 -1  0 一定等于4294967295。而这个数够大，已经相当于没有对原数组做任何处理了，秒啊

##### WEL：
> 本人小白一枚，对于代码中的这个：const globalEvent = window.myEvent 如何定义，我在代码中定义，一直报错，网上也没给出相应的答案，老师可否指点迷津😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; window.myEvent = new myEventEmitter();

##### **光：
> 修言老师有没有 Vue 的教程呀，准不准备出出，老师的 React 课程实在是太棒了！很期待能有 Vue 的课程

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以期待一下哦

##### **飞：
> B组件注册 A组件触发

##### *飞：
> 非常赞

##### *阳：
> 非常好，和直前的设计模式小册 对上了

##### **原：
> 循序渐进，娓娓道来，听得太舒服啦

##### *赢：
> 写的太好了！大佬的小册我也都买过！一如既往的好！讲的太明白了！

##### **8542：
> 父子，子父，兄弟基于props的通信，单向数据流，向下流动。任意组件发布订阅模式。重点考核发布订阅模式。

##### *鹤：
> 这是位运算

##### **涛：
> 点赞

##### *浩：
> 右移符号涨姿势了

##### *攀：
> 给修大点个赞

##### **逸：
> const globalEvent = window.myEvent，是指赋值给全局window对象，方便调用

##### **安：
> 继续打卡

##### **克：
> 无符号右移骚操作啊

##### *平：
> 哈哈，这个无符号右移运算符，一开始我确实没看懂。看了下面各位大佬的解释，以及这篇博文https://www.jianshu.com/p/6c518e7b4690 之后，我算明白了精髓所在。首先负数以其正值的补码形式表达，而无符号运算符( 0)，这里没有移动一位数，但是会把这个补码转化为一个正值，一个足够大的正值4294967295。正常情况下没人会往数据塞这么多数据，所以这个做法可以认为实际上可行，虽然理论上有上限。

##### **9390：
> 请问redux是不是就直接解决了多层component数据流传输不方便的问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

##### **舟：
> 最后这个-1＞＞＞0怎么理解啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; >>> 是无符号按位右移运算符。考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

##### **奎：
> 发布-订阅 讲的真好，大佬大佬，无符号右移看了一下评论区 哈哈 又学会一个骚操作

##### **琪：
> 研究了下位运算符">正数有符号右移与无符号右移结果是一样的，而负数无符号右移，会用0填充空位，同时把负数作为正数处理，结果会非常大。-14294967295

##### *忠：
> 右移运算符 这个精妙在哪里呢？indexOf()  0

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

##### **刚：
> 对发布-订阅 又有了进一步的认识...

##### **环：
> 太喜欢老师的课了，通俗易懂！感谢老师！

##### **燕：
> 写的太好了，有收获

##### **4992：
> 这个 右移 运算符精妙，捣鼓了一会才弄明白，恍然大悟的感觉，哈哈哈

##### *琴：
> const globalEvent = window.myEvent，这里需要怎么挂载到全局呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; window.myEvent = new myEventEmitter();

##### **阳：
> 内容逻辑清晰，也很好理解，讲的特别好

##### **龙：
> 大佬，我有个不太明白的地方，off函数里边的那个右移符号的作用？可以去掉或者用其他方式替代吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

##### **涛：
> 前排学习小队

##### **成：
> 耐心看完了,有收获😁

##### *旭：
> 打卡

##### fanta：
> this.eventMap[type].splice(this.eventMap[type].indexOf(handler)0,1)修言老师您好 请问这里位运算有什么特殊用意吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 那个不是大于号，>>> 是无符号按位右移运算符。考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

##### jhawn：
> this.eventMap[type].splice(this.eventMap[type].indexOf(handler) 0,1)这里indexOf后面三个大于号如何理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这三个大于号 >>> 是无符号按位右移运算符。考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

##### **3970：
> off(type, handler) { if (this.eventMap[type]) { } }请问这里的移位操作符的作用是什么呢?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; >>> 是无符号按位右移运算符。考虑 indexOf 返回-1 的情况：splice方法喜欢把-1解读为当前数组的最后一个元素，这样子的话，在压根没有对应函数可以删的情况下，不管三七二十一就把最后一个元素给干掉了。而 >>> 符号对正整数没有影响，但对于-1来说它会把-1转换为一个巨大的数（你可以本地运行下试试看，应该是一个32位全是1的二进制数，折算成十进制就是 4294967295）。这个巨大的索引splice是找不到的，找不到就不删，于是一切保持原状，刚好符合我们的预期。

