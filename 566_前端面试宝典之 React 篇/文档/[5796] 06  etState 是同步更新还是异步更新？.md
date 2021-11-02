<p data-nodeid="893" class="">本讲我们一起来探讨“setState 是同步更新还是异步更新”，这个问题在面试中应该如何回答。</p>
<h3 data-nodeid="894">破题</h3>
<p data-nodeid="895">“是 A 还是 B ”是一个在面试中经常会被问到的问题类型，这类问题有相当强的迷惑性，因为在不同的场景中会有不同的选择：</p>
<ul data-nodeid="896">
<li data-nodeid="897">
<p data-nodeid="898">可能是 A；</p>
</li>
<li data-nodeid="899">
<p data-nodeid="900">也可能是 B；</p>
</li>
<li data-nodeid="901">
<p data-nodeid="902">甚至 A 和 B 同时存在的可能性也是有的。</p>
</li>
</ul>
<p data-nodeid="903">所以就需要把问题放在具体的场景中探讨，才能有更加全面准确的回答。在面对类似的问题时，要先把场景理清楚，再去思考如何回答，一定不要让自己犯“想当然”的错误。这是回答类似问题第一个需要注意的点。</p>
<p data-nodeid="904">回到 setState 本身上来，setState 用于变更状态，触发组件重新渲染，更新视图 UI。有很多应聘者，并不清楚 state 在什么时候会被更新，所以难以解释到底是同步的还是异步的，也不清楚这个问题具体涉及哪些概念？</p>
<p data-nodeid="905">本题也是大厂面试中的一道高频题，常被用作检验应聘者的资深程度。</p>
<p data-nodeid="906">以上就是这个问题的“碎碎念”了，接下来是整理答题思路。</p>
<h3 data-nodeid="907">承题</h3>
<p data-nodeid="908">回到问题本身上来，其实思路很简单，只要能说清楚什么是同步场景，什么是异步场景，那问题自然而然就解决了。</p>
<p data-nodeid="909"><img src="https://s0.lgstatic.com/i/image2/M01/01/3D/Cip5yF_YUpOAALIlAABNx0PyF94306.png" alt="Drawing 1.png" data-nodeid="1010"></p>
<h3 data-nodeid="910">入手</h3>
<p data-nodeid="911">在分析场景之前，需要先补充一个很重要的知识点，即合成事件，同样它也是 React 面试中很容易被考察的点。合成事件与 setState 的触发更新有千丝万缕的关系，也只有在了解合成事件后，我们才能继续聊 setState。</p>
<h4 data-nodeid="912">合成事件</h4>
<p data-nodeid="913">在没有合成事件前，大家是如何处理事件的呢？由于很多同学都是直接从 React 和 Vue 开始入门的，所以很可能不太清楚这样一个在过去非常常见的场景。</p>
<p data-nodeid="914">假设一个列表的 ul 标签下面有 10000 个 li 标签。现在需要添加点击事件，通过点击获取当前 li 标签中的文本。那该如何操作？如果按照现在 React 的编写方式，就是为每一个 li 标签添加 onclick 事件。有 10000 个 li 标签，则会添加 10000 个事件。这是一种非常不友好的方式，会对页面的性能产生影响。</p>
<pre class="lang-java" data-nodeid="915"><code data-language="java">&lt;ul&gt;
  &lt;li onclick="geText(this)"&gt;1&lt;/li&gt;
  &lt;li onclick="geText(this)"&gt;2&lt;/li&gt;
  &lt;li onclick="geText(this)"&gt;3&lt;/li&gt;
  &lt;li onclick="geText(this)"&gt;4&lt;/li&gt;
  &lt;li onclick="geText(this)"&gt;5&lt;/li&gt;
  &nbsp;...
  &lt;li onclick="geText(this)"&gt;10000&lt;/li&gt;
&lt;/ul&gt;
</code></pre>
<p data-nodeid="916">那该怎么优化呢？最恰当的处理方式是采用<strong data-nodeid="1021">事件委托</strong>。通过将事件绑定在 ul 标签上这样的方式来解决。当 li 标签被点击时，由事件冒泡到父级的 ul 标签去触发，并在 ul 标签的 onclick 事件中，确认是哪一个 li 标签触发的点击事件。</p>
<pre class="lang-java" data-nodeid="917"><code data-language="java">&lt;ul id="test"&gt;
  &lt;li&gt;1&lt;/li&gt;
  &lt;li&gt;2&lt;/li&gt;
  &lt;li&gt;3&lt;/li&gt;
  &lt;li&gt;4&lt;/li&gt;
  &lt;li&gt;5&lt;/li&gt;
  &lt;li&gt;10000&lt;/li&gt;
&lt;/ul&gt;
&lt;script&gt;
  function getEventTarget(e) {
      e = e || window.event;
      return e.target || e.srcElement; 
  }
  var ul = document.getElementById('test');
  ul.onclick = function(event) {
      var target = getEventTarget(event);
      alert(target.innerHTML);
  };
&lt;/script&gt;
</code></pre>
<p data-nodeid="918">同样，出于性能考虑，合成事件也是如此：</p>
<ul data-nodeid="919">
<li data-nodeid="920">
<p data-nodeid="921">React 给 document 挂上事件监听；</p>
</li>
<li data-nodeid="922">
<p data-nodeid="923">DOM 事件触发后冒泡到 document；</p>
</li>
<li data-nodeid="924">
<p data-nodeid="925">React 找到对应的组件，造出一个合成事件出来；</p>
</li>
<li data-nodeid="926">
<p data-nodeid="927">并按组件树模拟一遍事件冒泡。</p>
</li>
</ul>
<p data-nodeid="928"><img src="https://s0.lgstatic.com/i/image2/M01/01/3E/CgpVE1_YUqKAA-jWAACt3Mh2xk8536.png" alt="Drawing 2.png" data-nodeid="1029"></p>
<div data-nodeid="929"><p style="text-align:center">React 17 之前的事件冒泡流程图</p></div>
<p data-nodeid="930">所以这就造成了，在一个页面中，只能有一个版本的 React。如果有多个版本，事件就乱套了。值得一提的是，这个问题在 React 17 中得到了解决，事件委托不再挂在 document 上，而是挂在 DOM 容器上，也就是 ReactDom.Render 所调用的节点上。</p>
<p data-nodeid="931"><img src="https://s0.lgstatic.com/i/image2/M01/01/3E/Cip5yF_YUzCAWTyoAAB1ljK7rSM539.png" alt="Drawing 3.png" data-nodeid="1033"></p>
<div data-nodeid="932"><p style="text-align:center">React 17 后的事件冒泡流程图</p></div>
<p data-nodeid="933">那到底哪些事件会被捕获生成合成事件呢？可以从 React 的源码测试文件中一探究竟。下面的测试快照中罗列了大量的事件名，也只有在这份快照中的事件，才会被捕获生成合成事件。</p>
<pre class="lang-java" data-nodeid="934"><code data-language="java"><span class="hljs-comment">// react/packages/react-dom/src/__tests__/__snapshots__/ReactTestUtils-test.js.snap</span>
Array [
	  <span class="hljs-string">"abort"</span>,
	  <span class="hljs-string">"animationEnd"</span>,
	  <span class="hljs-string">"animationIteration"</span>,
	  <span class="hljs-string">"animationStart"</span>,
	  <span class="hljs-string">"auxClick"</span>,
	  <span class="hljs-string">"beforeInput"</span>,
	  <span class="hljs-string">"blur"</span>,
	  <span class="hljs-string">"canPlay"</span>,
	  <span class="hljs-string">"canPlayThrough"</span>,
	  <span class="hljs-string">"cancel"</span>,
	  <span class="hljs-string">"change"</span>,
	  <span class="hljs-string">"click"</span>,
	  <span class="hljs-string">"close"</span>,
	  <span class="hljs-string">"compositionEnd"</span>,
	  <span class="hljs-string">"compositionStart"</span>,
	  <span class="hljs-string">"compositionUpdate"</span>,
	  <span class="hljs-string">"contextMenu"</span>,
	  <span class="hljs-string">"copy"</span>,
	  <span class="hljs-string">"cut"</span>,
	  <span class="hljs-string">"doubleClick"</span>,
	  <span class="hljs-string">"drag"</span>,
	  <span class="hljs-string">"dragEnd"</span>,
	  <span class="hljs-string">"dragEnter"</span>,
	  <span class="hljs-string">"dragExit"</span>,
	  <span class="hljs-string">"dragLeave"</span>,
	  <span class="hljs-string">"dragOver"</span>,
	  <span class="hljs-string">"dragStart"</span>,
	  <span class="hljs-string">"drop"</span>,
	  <span class="hljs-string">"durationChange"</span>,
	  <span class="hljs-string">"emptied"</span>,
	  <span class="hljs-string">"encrypted"</span>,
	  <span class="hljs-string">"ended"</span>,
	  <span class="hljs-string">"error"</span>,
	  <span class="hljs-string">"focus"</span>,
	  <span class="hljs-string">"gotPointerCapture"</span>,
	  <span class="hljs-string">"input"</span>,
	  <span class="hljs-string">"invalid"</span>,
	  <span class="hljs-string">"keyDown"</span>,
	  <span class="hljs-string">"keyPress"</span>,
	  <span class="hljs-string">"keyUp"</span>,
	  <span class="hljs-string">"load"</span>,
	  <span class="hljs-string">"loadStart"</span>,
	  <span class="hljs-string">"loadedData"</span>,
	  <span class="hljs-string">"loadedMetadata"</span>,
	  <span class="hljs-string">"lostPointerCapture"</span>,
	  <span class="hljs-string">"mouseDown"</span>,
	  <span class="hljs-string">"mouseEnter"</span>,
	  <span class="hljs-string">"mouseLeave"</span>,
	  <span class="hljs-string">"mouseMove"</span>,
	  <span class="hljs-string">"mouseOut"</span>,
	  <span class="hljs-string">"mouseOver"</span>,
	  <span class="hljs-string">"mouseUp"</span>,
	  <span class="hljs-string">"paste"</span>,
	  <span class="hljs-string">"pause"</span>,
	  <span class="hljs-string">"play"</span>,
	  <span class="hljs-string">"playing"</span>,
	  <span class="hljs-string">"pointerCancel"</span>,
	  <span class="hljs-string">"pointerDown"</span>,
	  <span class="hljs-string">"pointerEnter"</span>,
	  <span class="hljs-string">"pointerLeave"</span>,
	  <span class="hljs-string">"pointerMove"</span>,
	  <span class="hljs-string">"pointerOut"</span>,
	  <span class="hljs-string">"pointerOver"</span>,
	  <span class="hljs-string">"pointerUp"</span>,
	  <span class="hljs-string">"progress"</span>,
	  <span class="hljs-string">"rateChange"</span>,
	  <span class="hljs-string">"reset"</span>,
	  <span class="hljs-string">"scroll"</span>,
	  <span class="hljs-string">"seeked"</span>,
	  <span class="hljs-string">"seeking"</span>,
	  <span class="hljs-string">"select"</span>,
	  <span class="hljs-string">"stalled"</span>,
	  <span class="hljs-string">"submit"</span>,
	  <span class="hljs-string">"suspend"</span>,
	  <span class="hljs-string">"timeUpdate"</span>,
	  <span class="hljs-string">"toggle"</span>,
	  <span class="hljs-string">"touchCancel"</span>,
	  <span class="hljs-string">"touchEnd"</span>,
	  <span class="hljs-string">"touchMove"</span>,
	  <span class="hljs-string">"touchStart"</span>,
	  <span class="hljs-string">"transitionEnd"</span>,
	  <span class="hljs-string">"volumeChange"</span>,
	  <span class="hljs-string">"waiting"</span>,
	  <span class="hljs-string">"wheel"</span>,
	]
</code></pre>
<p data-nodeid="935">在有了合成事件的基础后，就更容易理解后续的内容了。</p>
<h4 data-nodeid="936">调用顺序</h4>
<p data-nodeid="937">setState 是不是异步的？我们来从头梳理。</p>
<p data-nodeid="938"><strong data-nodeid="1041">异步场景</strong></p>
<p data-nodeid="939">通常我们认为 setState 是异步的，就像这样一个例子：</p>
<pre class="lang-java" data-nodeid="940"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Component</span> </span>{
&nbsp; &nbsp; state = {
&nbsp; &nbsp; &nbsp; &nbsp; count: <span class="hljs-number">0</span>
&nbsp; &nbsp; }

&nbsp; &nbsp; componentDidMount(){
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.setState({
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;count: <span class="hljs-number">1</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}, () =&gt; {
            console.log(<span class="hljs-keyword">this</span>.state.count) <span class="hljs-comment">//1</span>
         })
&nbsp; &nbsp; &nbsp; &nbsp; console.log(<span class="hljs-keyword">this</span>.state.count) <span class="hljs-comment">// 0</span>
&nbsp; &nbsp; }

&nbsp; &nbsp; render(){
&nbsp; &nbsp; &nbsp; &nbsp; ...
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="941">由于我们接受 setState 是异步的，所以会认为回调函数是异步回调，打出 0 的 console.log 会先执行，打出 1 的会后执行。</p>
<p data-nodeid="942">那接下来这个案例的答案是什么呢？</p>
<pre class="lang-java" data-nodeid="943"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Component</span> </span>{
    state = {
        count: <span class="hljs-number">0</span>
    }

    componentDidMount(){
        <span class="hljs-keyword">this</span>.setState({
           count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
         }, () =&gt; {
            console.log(<span class="hljs-keyword">this</span>.state.count)
         })
         <span class="hljs-keyword">this</span>.setState({
           count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>
         }, () =&gt; {
            console.log(<span class="hljs-keyword">this</span>.state.count)
         })
    }

    render(){
        ...
    }
}
</code></pre>
<p data-nodeid="944">如果你觉得答案是 1,2，那肯定就错了。这种迷惑性极强的考题在面试中非常常见，因为它反直觉。</p>
<p data-nodeid="945">如果重新仔细思考，你会发现当前拿到的 this.state.count 的值并没有变化，都是 0，所以输出结果应该是 1,1。</p>
<p data-nodeid="946">当然，也可以在 setState 函数中获取修改后的 state 值进行修改。</p>
<pre class="lang-java" data-nodeid="947"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Component</span> </span>{
    state = {
        count: <span class="hljs-number">0</span>
    }

    componentDidMount(){
        <span class="hljs-keyword">this</span>.setState(
          preState=&gt; ({
            count:preState.count + <span class="hljs-number">1</span>
        }),()=&gt;{
           console.log(<span class="hljs-keyword">this</span>.state.count)
        })
        <span class="hljs-keyword">this</span>.setState(
          preState=&gt;({
            count:preState.count + <span class="hljs-number">1</span>
        }),()=&gt;{
           console.log(<span class="hljs-keyword">this</span>.state.count)
        })
    }

    render(){
        ...
    }
}
</code></pre>
<p data-nodeid="948">这些通通是异步的回调，如果你以为输出结果是 1,2，那就又错了，实际上是 2,2。</p>
<p data-nodeid="949">为什么会这样呢？当调用 setState 函数时，就会把当前的操作放入队列中。React 根据队列内容，合并 state 数据，完成后再逐一执行回调，根据结果更新虚拟 DOM，触发渲染。所以回调时，state 已经合并计算完成了，输出的结果就是 2,2 了。</p>
<p data-nodeid="950">这非常反直觉，那为什么 React 团队选择了这样一个行为模式，而不是同步进行呢？一种常见的说法是为了优化。通过异步的操作方式，累积更新后，批量合并处理，减少渲染次数，提升性能。但同步就不能批量合并吗？这显然不能完全作为 setState 设计成异步的理由。</p>
<p data-nodeid="951">在 17 年的时候就有人提出这样一个疑问“<a href="https://github.com/facebook/react/issues/11527" data-nodeid="1054">为什么 setState 是异步的</a>”，这个问题得到了官方团队的回复，原因有 2 个。</p>
<ul data-nodeid="952">
<li data-nodeid="953">
<p data-nodeid="954"><strong data-nodeid="1060">保持内部一致性</strong>。如果改为同步更新的方式，尽管 setState 变成了同步，但是 props 不是。</p>
</li>
<li data-nodeid="955">
<p data-nodeid="956"><strong data-nodeid="1065">为后续的架构升级启用并发更新</strong>。为了完成异步渲染，React 会在 setState 时，根据它们的数据来源分配不同的优先级，这些数据来源有：事件回调句柄、动画效果等，再根据优先级并发处理，提升渲染性能。</p>
</li>
</ul>
<p data-nodeid="957">从 React 17 的角度分析，异步的设计无疑是正确的，使异步渲染等最终能在 React 落地。那什么情况下它是同步的呢？</p>
<p data-nodeid="958"><strong data-nodeid="1070">同步场景</strong></p>
<p data-nodeid="959">异步场景中的案例使我们建立了这样一个认知：setState 是异步的，但下面这个案例又会颠覆你的认知。如果我们将 setState 放在 setTimeout 事件中，那情况就完全不同了。</p>
<pre class="lang-java" data-nodeid="960"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Component</span> </span>{
    state = {
        count: <span class="hljs-number">0</span>
    }

    componentDidMount(){
        <span class="hljs-keyword">this</span>.setState({ count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span> });
        console.log(<span class="hljs-keyword">this</span>.state.count);
        setTimeout(() =&gt; {
          <span class="hljs-keyword">this</span>.setState({ count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span> });
          console.log(<span class="hljs-string">"setTimeout: "</span> + <span class="hljs-keyword">this</span>.state.count);
        }, <span class="hljs-number">0</span>);
    }

    render(){
        ...
    }
}
</code></pre>
<p data-nodeid="961">那这时输出的应该是什么呢？如果你认为是 0,0，那么又错了。</p>
<p data-nodeid="962">正确的结果是 0,2。因为 setState 并不是真正的异步函数，它实际上是通过队列延迟执行操作实现的，通过 isBatchingUpdates 来判断 setState 是先存进 state 队列还是直接更新。值为 true 则执行异步操作，false 则直接同步更新。</p>
<p data-nodeid="963"><img src="https://s0.lgstatic.com/i/image2/M01/01/47/Cip5yF_YYfCAXIxiAAEJsQbj_hs785.png" alt="图片1.png" data-nodeid="1076"><br>
在 onClick、onFocus 等事件中，由于合成事件封装了一层，所以可以将 isBatchingUpdates 的状态更新为 true；在 React 的生命周期函数中，同样可以将 isBatchingUpdates 的状态更新为 true。那么在 React 自己的生命周期事件和合成事件中，可以拿到 isBatchingUpdates 的控制权，将状态放进队列，控制执行节奏。而在外部的原生事件中，并没有外层的封装与拦截，无法更新 isBatchingUpdates 的状态为 true。这就造成 isBatchingUpdates 的状态只会为 false，且立即执行。所以在 addEventListener&nbsp;、setTimeout、setInterval 这些原生事件中都会同步更新。</p>
<h3 data-nodeid="964">回答</h3>
<p data-nodeid="965">接下来我们可以答题了。</p>
<blockquote data-nodeid="966">
<p data-nodeid="967">setState 并非真异步，只是看上去像异步。在源码中，通过 isBatchingUpdates 来判断<br>
setState 是先存进 state 队列还是直接更新，如果值为 true 则执行异步操作，为 false 则直接更新。</p>
<p data-nodeid="968">那么什么情况下 isBatchingUpdates 会为 true 呢？在 React 可以控制的地方，就为 true，比如在 React 生命周期事件和合成事件中，都会走合并操作，延迟更新的策略。</p>
<p data-nodeid="969">但在 React 无法控制的地方，比如原生事件，具体就是在 addEventListener&nbsp;、setTimeout、setInterval 等事件中，就只能同步更新。</p>
<p data-nodeid="970">一般认为，做异步设计是为了性能优化、减少渲染次数，React 团队还补充了两点。</p>
<ol data-nodeid="971">
<li data-nodeid="972">
<p data-nodeid="973">保持内部一致性。如果将 state 改为同步更新，那尽管 state 的更新是同步的，但是 props不是。</p>
</li>
<li data-nodeid="974">
<p data-nodeid="975">启用并发更新，完成异步渲染。</p>
</li>
</ol>
</blockquote>
<p data-nodeid="976">综上所述，我们可以整理出下面的知识导图。</p>
<p data-nodeid="977"><img src="https://s0.lgstatic.com/i/image2/M01/01/3E/CgpVE1_YU2KAStLdAAFVKxh7Dyg317.png" alt="Drawing 7.png" data-nodeid="1092"></p>
<h3 data-nodeid="978">进阶</h3>
<p data-nodeid="979">这是一道经常会出现的 React setState 笔试题：下面的代码输出什么呢？</p>
<pre class="lang-java" data-nodeid="980"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  state  = {
      count: <span class="hljs-number">0</span>
  };

    componentDidMount() {
    <span class="hljs-keyword">this</span>.setState({count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>});
    console.log(<span class="hljs-keyword">this</span>.state.count);

    <span class="hljs-keyword">this</span>.setState({count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>});
    console.log(<span class="hljs-keyword">this</span>.state.count);

    setTimeout(() =&gt; {
      <span class="hljs-keyword">this</span>.setState({count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>});
      console.log(<span class="hljs-keyword">this</span>.state.count);

      <span class="hljs-keyword">this</span>.setState({count: <span class="hljs-keyword">this</span>.state.count + <span class="hljs-number">1</span>});
      console.log(<span class="hljs-keyword">this</span>.state.count);
    }, <span class="hljs-number">0</span>);
  }
 
  render() {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
  }
};
</code></pre>
<p data-nodeid="981">我们可以进行如下的分析：</p>
<ul data-nodeid="982">
<li data-nodeid="983">
<p data-nodeid="984">首先第一次和第二次的 console.log，都在 React 的生命周期事件中，所以是异步的处理方式，则输出都为 0；</p>
</li>
<li data-nodeid="985">
<p data-nodeid="986">而在 setTimeout 中的 console.log 处于原生事件中，所以会同步的处理再输出结果，但需要注意，虽然 count 在前面经过了两次的 this.state.count + 1，但是每次获取的 this.state.count 都是初始化时的值，也就是 0；</p>
</li>
<li data-nodeid="987">
<p data-nodeid="988">所以此时 count 是 1，那么后续在 setTimeout 中的输出则是 2 和 3。</p>
</li>
</ul>
<p data-nodeid="989">所以完整答案是 0,0,2,3。</p>
<h3 data-nodeid="990">总结</h3>
<p data-nodeid="991">在本讲中，我们掌握了判断 setState 是同步还是异步的核心关键点：更新队列。不得不再强调一下，看 setState 的输出结果是面试的常考点。所以在面试前，可以再针对性的看一下这部分内容，然后自己执行几次试试。</p>
<p data-nodeid="992">下一讲我将为你介绍另一个常考点，React 的跨组件通信。</p>
<p data-nodeid="993"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1107"><img src="https://s0.lgstatic.com/i/image/M00/72/94/Ciqc1F_EZ0eANc6tAASyC72ZqWw643.png" alt="Drawing 2.png" data-nodeid="1106"></a></p>
<p data-nodeid="994">《大前端高薪训练营》</p>
<p data-nodeid="995" class="te-preview-highlight">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1112">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **3813：
> 有两个问题1. “如果改为同步更新的方式，尽管 setState 变成了同步，但是 props 不是”为什么props不是同步？没有理解。2. 为什么倒数第一个例子是0023，但倒数第二个例子是01而不是02？倒数第二个例子的生命周期里一次setState跟倒数第一个例子里两次setState应该是一样的效果啊，都是把count变为1，然后setTimeout里执行同步，不应该都是2吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 这个句话原文对应的是<RFClarification: why is `setState` asynchronous?>(https://github.com/facebook/react/issues/11527)  中的 “even if state is updated synchronously, props are not. (You can’t know props until you re-render the parent component, and if you do this synchronously, batching goes out of the window.)” 
以展示组件为例，展示组件的 props 通常是由父级组件的 state 驱动的，那 state 更新改为同步了，但你无法控制父级什么时候会去变更子组件的 props， 父级在各种场景都有可能会去发起更新。为了提升性能，更新操作仍然需要做成批处理的形式，但这就很有可能会是的处理超过窗口期。《RFClarification: why is `setState` asynchronous?》原文整篇论述非常有意义，翻译可能存在失真，所以还是建议直接看原文理解。
2.因为在 setTimeout 中的 setState，isbatchingupdates 的标识符并不会被设为 true，
不会将变更放入队列，等待合并更新，所以每次 setState 都会被立即执行，拿到结果。
希望上面的回复能对你有所帮助，如果还有不清楚的地方，也可以看一下这个 demo，https://codesandbox.io/embed/setstate-uxlec?fontsize=14&hidenavigation=1&theme=dark，欢迎继续留言。

##### **涛：
> 其实，在concurrent模式下，打印出来的是另一种结果。就比如最后一个例子，打印出的是 0 0 1 1

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，非常感谢补充了 concurrent 下的结果。
concurrent 模式下，很多就有的认知都会被打碎重塑，但目前 concurrent 还处于 unstable 的状态，所以本讲中探讨的还是同步模式下的结果。

##### **辉：
> isBatchingUpdates 这个变量我在源码里面并没有找到 还是说react 17里面不是这个判断方式了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; React 16.8 可以在 packages/react-reconciler/src/ReactFiberScheduler.js 文件中找到对应代码片段。
但在 React 17 中随着 lane 算法的更新，已经移除了相关的概念，转而通过 lane 优先级来区分。代码对应了 https://github.com/facebook/react/blob/9198a5cec0936a21a5ba194a22fcbac03eba5d1d/packages/react-reconciler/src/ReactFiberWorkLoop.new.js#L726，很有意思的是除了 SyncLanePriority，还新增了  SyncBatchedLanePriority 批量处理同步更新。

##### **6400：
> 老师请问为什么props是异步更新呢，感觉没有关注过props的更新方式

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为子组件的 props 到上层往往是容器组件或者父子局的一个 state 进行控制处理。

##### *磊：
> 一个页面多个react版本是什么样场景？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个场景确实比较少，少到 React 17 才开始支持。主要是页面中部分组件及它们的依赖采用了老版本构建，而其他的部分采用 17 进行构建，需要混合使用。多出现在多个团队在同一页面进行大规模业务交付的情况。很少见，到目前为止，我也就遇到一次的样子。

##### *聪：
> React在什么情况下自己会设置isBatchingUpdates为false呢？如果异步这么好的话，那为何还需要设置isBatchingUpdates这个来控制呢，直接设置所有情况都异步的不行吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我这样来比喻一下，假如现在有个专门处理文件的人叫 React，如果所有文件都从 React 手上通过走流程的话，那么他就能自己安排节奏，控制排期处理。
但如果是有人不走流程，突然找上门，要求处理，怎么办？如果也排期等待的话，不知道后面还会不会有人来要求处理，耽搁时间太长，会不会阻塞别的事，所以只能马上处理了。
不知道例子是否恰当，实际上 React 在自己的生命周期事件和合成事件中会拨开 isBatchingUpdates 的开关，等待合并处理状态，这个节奏是很好控制的。
如果仍然不是很清楚的话，建议阅读下源码（React 16 packages/react-reconciler/src/ReactFiberScheduler.js），没有比代码更好的解释了，不过需要注意，在 React 17 中，用 lanes 替代了 isBatchingUpdates。
如果还有不清楚的地方，欢迎继续留言，我会尽力解答。

##### **晗：
> 最后一个执行结果不应该是0012吗 settimeout第一次拿到初始化是0然后加1输出第一个consolelog再加1输出第二次consolelog 为2 为什么文章里面是0023呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为在 setTimeout 中的 setState，isbatchingupdates 的标识符并不会被设为 true，
不会将变更放入队列，等待合并更新，所以每次 setState 都会被立即执行，拿到结果。
希望上面的回复能对你有所帮助，如果还有不清楚的地方，也可以看一下这个 demo，https://codesandbox.io/embed/setstate-uxlec?fontsize=14&hidenavigation=1&theme=dark，欢迎继续留言。

##### **波：
> 你好，setState传递函数修改count值，那里打印的应该是2，2 不应该是1，2吧，因为是异步，等到执行回调的时候 已经都是2了吧。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为在 setTimeout 中的 setState，isbatchingupdates 的标识符并不会被设为 true，
不会将变更放入队列，等待合并更新，所以每次 setState 都会被立即执行，拿到结果。
希望上面的回复能对你有所帮助，如果还有不清楚的地方，也可以看一下这个 demo，https://codesandbox.io/embed/setstate-uxlec?fontsize=14&hidenavigation=1&theme=dark，欢迎继续留言。

##### **卫：
> 老师，你好，倒数第二个例子，setTimeout外console那里count是0，setTimeout内setState变为同步后，取count值不应该也是0吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的「同步」是指不会累积合并更新处理，并不会改变原有代码执行顺序。
可以看最后一个例子中的例子，在 setTimeout 中的 setState 会快速得到结果，而不会合并处理 state。
如果我还有没讲清的地方，欢迎继续留言。

##### **6400：
> 老师，请问答案为什么不是0034呢，在setTimeout第一个console前有三个setState

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为 setTimeout 外的 setState 会积攒起来一次性批处理，呈现出异步处理的状态。那么两次 this.state.count 拿到的实际上都是 0。

##### *聪：
> 图中那个pendding队列和文中讲的会不会放入“队列”，此队列非彼队列是吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是同一个队列。
如果仍然不是很清楚的话，建议阅读下源码（React 16 packages/react-reconciler/src/ReactFiberScheduler.js），没有比代码更好的解释了，不过需要注意，在 React 17 中，用 lanes 替代了 isBatchingUpdates。
如果还有不清楚的地方，欢迎继续留言，我会尽力解答。

