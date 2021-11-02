<p data-nodeid="1113" class="">时下虽然接入 JSX 语法的框架越来越多，但与之缘分最深的毫无疑问仍然是 React。2013 年，当 React 带着 JSX 横空出世时，社区曾对 JSX 有过不少的争议，但如今，越来越多的人面对 JSX 都要说上一句“真香”！本课时我们就来一起认识下这个“真香”的 JSX，聊一聊“JSX 代码是如何‘摇身一变’成为 DOM 的”。</p>
<h3 data-nodeid="1114">关于 JSX 的 3 个“大问题”</h3>
<p data-nodeid="1115">在日常的 React 开发工作中，我们已经习惯了使用 JSX 来描述 React 的组件内容。关于 JSX 语法本身，相信每位 React 开发者都不陌生。这里我用一个简单的 React 组件，来帮你迅速地唤醒自己脑海中与 JSX 相关的记忆。下面这个组件中的 render 方法返回值，就是用 JSX 代码来填充的：</p>
<pre class="lang-js" data-nodeid="1116"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> ReactDOM <span class="hljs-keyword">from</span> <span class="hljs-string">"react-dom"</span>;

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">App</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-keyword">return</span> (
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">h1</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"title"</span>&gt;</span>I am the title<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"content"</span>&gt;</span>I am the content<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    );
  }
}

<span class="hljs-keyword">const</span> rootElement = <span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">"root"</span>);
ReactDOM.render(<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">App</span> /&gt;</span></span>, rootElement);
</code></pre>
<p data-nodeid="1117">由于本专栏的整体目标是帮助你在 React 这个领域完成从“小工”到“行家”的进阶，此处我无意再去带你反复咀嚼 JSX 的基础语法，而是希望能够引导你去探寻 JSX 背后的故事。针对这“背后的故事”，我总结了 3 个最具代表性和区分度的问题。</p>
<p data-nodeid="1118">在开始正式讲解之前，我希望你能在自己心中尝试回答这 3 个问题：</p>
<ul data-nodeid="1119">
<li data-nodeid="1120">
<p data-nodeid="1121">JSX 的本质是什么，它和 JS 之间到底是什么关系？</p>
</li>
<li data-nodeid="1122">
<p data-nodeid="1123">为什么要用 JSX？不用会有什么后果？</p>
</li>
<li data-nodeid="1124">
<p data-nodeid="1125">JSX 背后的功能模块是什么，这个功能模块都做了哪些事情？</p>
</li>
</ul>
<p data-nodeid="1126">面对以上问题，如果你无法形成清晰且系统的思路，那么很可能是你把 JSX 想得过于简单了。大多数人只是简单地把它理解为模板语法的一种，但事实上，JSX 作为 React 框架的一大特色，它与 React 本身的运作机制之间存在着千丝万缕的联系。</p>
<p data-nodeid="1127">上述 3 个问题的答案，就恰恰隐藏在这层“联系”中，在面试场景下，候选人对这层“联系”吃得透不透，是我们评价其在 React 方面是否“资深”的一个重要依据。</p>
<p data-nodeid="1128">接下来，我就将带你由表及里地起底 JSX 相关的底层原理，帮助你吃透这层“联系”，建立起强大的理论自信。你可以将“能够用自己的话回答上面 3 个问题”来作为本课时的学习目标，待课时结束后，记得回来检验自己的学习成果^_^。</p>
<h3 data-nodeid="1129">JSX 的本质：JavaScript 的语法扩展</h3>
<p data-nodeid="1130">JSX 到底是什么，我们先来看看 <a href="https://reactjs.org/docs/glossary.html#jsx" data-nodeid="1217">React 官网</a>给出的一段定义：</p>
<blockquote data-nodeid="1131">
<p data-nodeid="1132">JSX 是 JavaScript 的一种语法扩展，它和模板语言很接近，但是它充分具备 JavaScript 的能力。</p>
</blockquote>
<p data-nodeid="1133">“语法扩展”这一点在理解上几乎不会产生歧义，不过“它充分具备 JavaScript 的能力”这句，却总让人摸不着头脑，JSX 和 JS 怎么看也不像是一路人啊？这就引出了“<strong data-nodeid="1225">JSX 语法是如何在 JavaScript 中生效的</strong>”这个问题。</p>
<h4 data-nodeid="1134">JSX 语法是如何在 JavaScript 中生效的：认识 Babel</h4>
<p data-nodeid="1135">Facebook 公司给 JSX 的定位是 JavaScript 的“扩展”，而非 JavaScript 的“某个版本”，这就直接决定了浏览器并不会像天然支持 JavaScript 一样地支持 JSX。那么，JSX 的语法是如何在 JavaScript 中生效的呢？<a href="https://reactjs.org/docs/glossary.html#jsx" data-nodeid="1230">React 官网</a>其实早已给过我们线索：</p>
<blockquote data-nodeid="1136">
<p data-nodeid="1137">JSX 会被编译为 React.createElement()，&nbsp;React.createElement() 将返回一个叫作“React Element”的 JS 对象。</p>
</blockquote>
<p data-nodeid="1138">这里提到，JSX 在被<strong data-nodeid="1238">编译</strong>后，会变成一个针对 React.createElement 的调用，此时你大可不必急于关注 React.createElement 这个 API 到底做了什么（下文会单独讲解）。咱们先来说说这个“编译”是怎么回事：“编译”这个动作，是由 Babel 来完成的。</p>
<p data-nodeid="1139"><strong data-nodeid="1242">什么是 Babel 呢？</strong></p>
<blockquote data-nodeid="1140">
<p data-nodeid="1141">Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。<br>
—— Babel 官网</p>
</blockquote>
<p data-nodeid="1142">比如说，ES2015+ 版本推出了一种名为“模板字符串”的新语法，这种语法在一些低版本的浏览器里并不兼容。下面是一段模板字符串的示例代码：</p>
<pre class="lang-java" data-nodeid="1143"><code data-language="java"><span class="hljs-keyword">var</span> name = <span class="hljs-string">"Guy Fieri"</span>;
<span class="hljs-keyword">var</span> place = <span class="hljs-string">"Flavortown"</span>;
`Hello ${name}, ready <span class="hljs-keyword">for</span> ${place}?`;
</code></pre>
<p data-nodeid="1144">Babel 就可以帮我们把这段代码转换为大部分低版本浏览器也能够识别的 ES5 代码：</p>
<pre class="lang-java" data-nodeid="1145"><code data-language="java"><span class="hljs-keyword">var</span> name = <span class="hljs-string">"Guy Fieri"</span>;
<span class="hljs-keyword">var</span> place = <span class="hljs-string">"Flavortown"</span>;
<span class="hljs-string">"Hello "</span>.concat(name, <span class="hljs-string">", ready for "</span>).concat(place, <span class="hljs-string">"?"</span>);
</code></pre>
<p data-nodeid="1146">类似的，<strong data-nodeid="1255">Babel 也具备将 JSX 语法转换为 JavaScript 代码的能力</strong>。<br>
那么 Babel 具体会将 JSX 处理成什么样子呢？我们不如直接打开 Babel 的 playground 来看一看。这里我仍然键入文章开头示例代码中的JSX 部分：</p>
<p data-nodeid="1147"><img src="https://s0.lgstatic.com/i/image/M00/5C/73/CgqCHl-BegWAbxNEAAH9HxafvWE988.png" alt="Drawing 0.png" data-nodeid="1258"></p>
<p data-nodeid="1148">可以看到，所有的 JSX 标签都被转化成了 React.createElement 调用，这也就意味着，我们写的 JSX 其实写的就是 React.createElement，虽然它看起来有点像 HTML，但也只是“看起来像”而已。<strong data-nodeid="1272">JSX 的本质是</strong>React.createElement<strong data-nodeid="1273">这个 JavaScript 调用的语法糖</strong>，这也就完美地呼应上了 React 官方给出的“<strong data-nodeid="1274">JSX 充分具备 JavaScript 的能力</strong>”这句话。</p>
<h4 data-nodeid="1149">React 选用 JSX 语法的动机</h4>
<p data-nodeid="1150">换个角度想想，既然 JSX 等价于一次 React.createElement 调用，那么 React 官方为什么不直接引导我们用 React.createElement 来创建元素呢？</p>
<p data-nodeid="1151">原因非常简单，我们来看一个相对复杂一些的组件的 JSX 代码和 React.createElement 调用之间的对比。它们各自的形态如下图所示，图中左侧是 JSX 代码，右侧是 React.createElement 调用：</p>
<p data-nodeid="1152"><img src="https://s0.lgstatic.com/i/image/M00/5C/73/CgqCHl-Beg-AXBihAA4t3S7nxKc532.png" alt="Drawing 1.png" data-nodeid="1280"></p>
<p data-nodeid="1153">你会发现，在实际功能效果一致的前提下，JSX 代码层次分明、嵌套关系清晰；而 React.createElement 代码则给人一种非常混乱的“杂糅感”，这样的代码不仅读起来不友好，写起来也费劲。</p>
<p data-nodeid="1154"><strong data-nodeid="1285">JSX 语法糖允许前端开发者使用我们最为熟悉的类 HTML 标签语法来创建虚拟 DOM，在降低学习成本的同时，也提升了研发效率与研发体验。</strong></p>
<p data-nodeid="1155">读到这里，相信你已经充分理解了“<strong data-nodeid="1291">JSX 是 JavaScript 的一种语法扩展，它和模板语言很接近，但是它充分具备 JavaScript 的能力</strong>。&nbsp;”这一定义背后的深意。那么我们文中反复提及的 React.createElement 又是何方神圣呢？下面我们就深入相关源码来一窥究竟。</p>
<h3 data-nodeid="1156">JSX 是如何映射为 DOM 的：起底 createElement 源码</h3>
<p data-nodeid="1157">在分析开始之前，你可以先尝试阅读我追加进源码中的逐行代码解析，大致理解 createElement 中每一行代码的作用：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="2505"><code data-language="java"><span class="hljs-comment">/**
 101. React的创建元素方法
 */</span>
<span class="hljs-function">export function <span class="hljs-title">createElement</span><span class="hljs-params">(type, config, children)</span> </span>{
  <span class="hljs-comment">// propName 变量用于储存后面需要用到的元素属性</span>
  let propName; 
  <span class="hljs-comment">// props 变量用于储存元素属性的键值对集合</span>
  <span class="hljs-keyword">const</span> props = {}; 
  <span class="hljs-comment">// key、ref、self、source 均为 React 元素的属性，此处不必深究</span>
  let key = <span class="hljs-keyword">null</span>;
  let ref = <span class="hljs-keyword">null</span>; 
  let self = <span class="hljs-keyword">null</span>; 
  let source = <span class="hljs-keyword">null</span>; 

  <span class="hljs-comment">// config 对象中存储的是元素的属性</span>
  <span class="hljs-keyword">if</span> (config != <span class="hljs-keyword">null</span>) { 
    <span class="hljs-comment">// 进来之后做的第一件事，是依次对 ref、key、self 和 source 属性赋值</span>
    <span class="hljs-keyword">if</span> (hasValidRef(config)) {
      ref = config.ref;
    }
    <span class="hljs-comment">// 此处将 key 值字符串化</span>
    <span class="hljs-keyword">if</span> (hasValidKey(config)) {
      key = <span class="hljs-string">''</span> + config.key; 
    }
    self = config.__self === undefined ? <span class="hljs-keyword">null</span> : config.__self;
    source = config.__source === undefined ? <span class="hljs-keyword">null</span> : config.__source;
    <span class="hljs-comment">// 接着就是要把 config 里面的属性都一个一个挪到 props 这个之前声明好的对象里面</span>
    <span class="hljs-keyword">for</span> (propName in config) {
      <span class="hljs-keyword">if</span> (
        <span class="hljs-comment">// 筛选出可以提进 props 对象里的属性</span>
        hasOwnProperty.call(config, propName) &amp;&amp;
        !RESERVED_PROPS.hasOwnProperty(propName) 
      ) {
        props[propName] = config[propName]; 
      }
    }
  }
  <span class="hljs-comment">// childrenLength 指的是当前元素的子元素的个数，减去的 2 是 type 和 config 两个参数占用的长度</span>
  <span class="hljs-keyword">const</span> childrenLength = arguments.length - <span class="hljs-number">2</span>; 
  <span class="hljs-comment">// 如果抛去type和config，就只剩下一个参数，一般意味着文本节点出现了</span>
  <span class="hljs-keyword">if</span> (childrenLength === <span class="hljs-number">1</span>) { 
    <span class="hljs-comment">// 直接把这个参数的值赋给props.children</span>
    props.children = children; 
    <span class="hljs-comment">// 处理嵌套多个子元素的情况</span>
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (childrenLength &gt; <span class="hljs-number">1</span>) { 
    <span class="hljs-comment">// 声明一个子元素数组</span>
    <span class="hljs-keyword">const</span> childArray = Array(childrenLength); 
    <span class="hljs-comment">// 把子元素推进数组里</span>
    <span class="hljs-keyword">for</span> (let i = <span class="hljs-number">0</span>; i &lt; childrenLength; i++) { 
      childArray[i] = arguments[i + <span class="hljs-number">2</span>];
    }
    <span class="hljs-comment">// 最后把这个数组赋值给props.children</span>
    props.children = childArray; 
  } 

  <span class="hljs-comment">// 处理 defaultProps</span>
  <span class="hljs-keyword">if</span> (type &amp;&amp; type.defaultProps) {
    <span class="hljs-keyword">const</span> defaultProps = type.defaultProps;
    <span class="hljs-keyword">for</span> (propName in defaultProps) { 
      <span class="hljs-keyword">if</span> (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  <span class="hljs-comment">// 最后返回一个调用ReactElement执行方法，并传入刚才处理过的参数</span>
  <span class="hljs-keyword">return</span> ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}
</code></pre>



<p data-nodeid="1159">上面是对源码细节的初步展示，接下来我会带你逐步提取源码中的关键知识点和核心思想。</p>
<h4 data-nodeid="1160">入参解读：创造一个元素需要知道哪些信息</h4>
<p data-nodeid="1161">我们先来看看方法的入参：</p>
<pre class="lang-java" data-nodeid="1162"><code data-language="java"><span class="hljs-function">export function <span class="hljs-title">createElement</span><span class="hljs-params">(type, config, children)</span>
</span></code></pre>
<p data-nodeid="1163">createElement 有 3 个入参，这 3 个入参囊括了 React 创建一个元素所需要知道的全部信息。</p>
<ul data-nodeid="1164">
<li data-nodeid="1165">
<p data-nodeid="1166">type：用于标识节点的类型。它可以是类似“h1”“div”这样的标准 HTML 标签字符串，也可以是 React 组件类型或 React fragment 类型。</p>
</li>
<li data-nodeid="1167">
<p data-nodeid="1168">config：以对象形式传入，组件所有的属性都会以键值对的形式存储在 config 对象中。</p>
</li>
<li data-nodeid="1169">
<p data-nodeid="1170">children：以对象形式传入，它记录的是组件标签之间嵌套的内容，也就是所谓的“子节点”“子元素”。</p>
</li>
</ul>
<p data-nodeid="1171">如果文字描述使你觉得抽象，下面这个调用示例可以帮你增进对概念的理解：</p>
<pre class="lang-java" data-nodeid="1172"><code data-language="java">React.createElement(<span class="hljs-string">"ul"</span>, {
  <span class="hljs-comment">// 传入属性键值对</span>
  className: <span class="hljs-string">"list"</span>
   <span class="hljs-comment">// 从第三个入参开始往后，传入的参数都是 children</span>
}, React.createElement(<span class="hljs-string">"li"</span>, {
  key: <span class="hljs-string">"1"</span>
}, <span class="hljs-string">"1"</span>), React.createElement(<span class="hljs-string">"li"</span>, {
  key: <span class="hljs-string">"2"</span>
}, <span class="hljs-string">"2"</span>));
</code></pre>
<p data-nodeid="1173">这个调用对应的 DOM 结构如下：</p>
<pre class="lang-js" data-nodeid="1174"><code data-language="js">&lt;ul className=<span class="hljs-string">"list"</span>&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"1"</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"2"</span>&gt;</span>2<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1175">对入参的形式和内容有了大致的把握之后，下面我们继续来讲解 createElement 的函数逻辑。</p>
<h4 data-nodeid="1176">createElement 函数体拆解</h4>
<p data-nodeid="1177" class="">前面你已经阅读过 createElement 源码细化到每一行的解读，这里我想和你探讨的是 createElement<strong data-nodeid="1310">在逻辑层面的任务流转</strong>。针对这个过程，我为你总结了下面这张流程图：</p>
<p data-nodeid="1178"><img src="https://s0.lgstatic.com/i/image/M00/5C/69/Ciqc1F-BeuGAepNsAACqreYXrj0410.png" alt="Drawing 3.png" data-nodeid="1313"></p>
<p data-nodeid="1179">这个流程图，或许会打破不少同学对 createElement 的幻想。<strong data-nodeid="1323">在实际的面试场景下，许多候选人由于缺乏对源码的了解，谈及 createElement 时总会倾向于去夸大它的“工作量”</strong>。但其实，相信你也已经发现了，createElement 中并没有十分复杂的涉及算法或真实 DOM 的逻辑，它的<strong data-nodeid="1324">每一个步骤几乎都是在格式化数据</strong>。</p>
<p data-nodeid="1180">说得更直白点，createElement 就像是开发者和 ReactElement 调用之间的一个“<strong data-nodeid="1334">转换器</strong>”、一个<strong data-nodeid="1335">数据处理层</strong>。它可以从开发者处接受相对简单的参数，然后将这些参数按照 ReactElement 的预期做一层格式化，最终通过调用 ReactElement 来实现元素的创建。整个过程如下图所示：</p>
<p data-nodeid="1181"><img src="https://s0.lgstatic.com/i/image/M00/5C/69/Ciqc1F-BevGANuu4AACN5mBDMlg569.png" alt="Drawing 5.png" data-nodeid="1338"></p>
<p data-nodeid="1182">现在看来，createElement 原来只是个“参数中介”。此时我们的注意力自然而然地就聚焦在了 ReactElement 上，接下来我们就乘胜追击，一起去挖一挖 ReactElement 的源码吧！</p>
<h4 data-nodeid="1183">出参解读：初识虚拟 DOM</h4>
<p data-nodeid="1184">上面已经分析过，createElement 执行到最后会 return 一个针对 ReactElement 的调用。这里关于 ReactElement，我依然先给出源码 + 注释形式的解析：</p>
<pre class="lang-java" data-nodeid="1185"><code data-language="java"><span class="hljs-keyword">const</span> ReactElement = function(type, key, ref, self, source, owner, props) {
  <span class="hljs-keyword">const</span> element = {
    <span class="hljs-comment">// REACT_ELEMENT_TYPE是一个常量，用来标识该对象是一个ReactElement</span>
    $$typeof: REACT_ELEMENT_TYPE,

    <span class="hljs-comment">// 内置属性赋值</span>
    type: type,
    key: key,
    ref: ref,
    props: props,

    <span class="hljs-comment">// 记录创造该元素的组件</span>
    _owner: owner,
  };

  <span class="hljs-comment">// </span>
  <span class="hljs-keyword">if</span> (__DEV__) {
    <span class="hljs-comment">// 这里是一些针对 __DEV__ 环境下的处理，对于大家理解主要逻辑意义不大，此处我直接省略掉，以免混淆视听</span>
  }

  <span class="hljs-keyword">return</span> element;
};
</code></pre>
<p data-nodeid="1186">ReactElement 的代码出乎意料的简短，从逻辑上我们可以看出，ReactElement 其实只做了一件事情，那就是“<strong data-nodeid="1351">创建</strong>”，说得更精确一点，是“<strong data-nodeid="1352">组装</strong>”：ReactElement 把传入的参数按照一定的规范，“组装”进了 element 对象里，并把它返回给了 React.createElement，最终 React.createElement 又把它交回到了开发者手中。整个过程如下图所示：</p>
<p data-nodeid="1187"><img src="https://s0.lgstatic.com/i/image/M00/5C/74/CgqCHl-Bex6AM5rhAACJMrix5bk913.png" alt="Drawing 7.png" data-nodeid="1355"></p>
<p data-nodeid="1188">如果你想要验证这一点，可以尝试输出我们示例中 App 组件的 JSX 部分：</p>
<pre class="lang-js" data-nodeid="1189"><code data-language="js"><span class="hljs-keyword">const</span> AppJSX = (<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"App"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"title"</span>&gt;</span>I am the title<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">className</span>=<span class="hljs-string">"content"</span>&gt;</span>I am the content<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>)

<span class="hljs-built_in">console</span>.log(AppJSX)
</code></pre>
<p data-nodeid="1190">你会发现它确实是一个标准的 ReactElement 对象实例，如下图（生产环境下的输出结果）所示：</p>
<p data-nodeid="1191"><img src="https://s0.lgstatic.com/i/image/M00/5C/69/Ciqc1F-BezKAW4rXAAIUYQW6Lk0911.png" alt="Drawing 8.png" data-nodeid="1360"></p>
<p data-nodeid="1192">这个 ReactElement 对象实例，本质上是<strong data-nodeid="1370">以 JavaScript 对象形式存在的对 DOM 的描述</strong>，也就是老生常谈的“虚拟 DOM”（<strong data-nodeid="1371">准确地说，是虚拟 DOM 中的一个节点</strong>。关于虚拟 DOM， 我们将在专栏的“模块二：核心原理”中花大量的篇幅来研究它，此处你只需要能够结合源码，形成初步认知即可）。</p>
<p data-nodeid="1193">既然是“虚拟 DOM”，那就意味着和渲染到页面上的真实 DOM 之间还有一些距离，这个“距离”，就是由大家喜闻乐见的<strong data-nodeid="1377">ReactDOM.render</strong>方法来填补的。</p>
<p data-nodeid="1194">在每一个 React 项目的入口文件中，都少不了对 React.render 函数的调用。下面我简单介绍下 ReactDOM.render 方法的入参规则：</p>
<pre class="lang-java" data-nodeid="1195"><code data-language="java">ReactDOM.render(
    <span class="hljs-comment">// 需要渲染的元素（ReactElement）</span>
    element, 
    <span class="hljs-comment">// 元素挂载的目标容器（一个真实DOM）</span>
    container,
    <span class="hljs-comment">// 回调函数，可选参数，可以用来处理渲染结束后的逻辑</span>
    [callback]
)
</code></pre>
<p data-nodeid="1196">ReactDOM.render 方法可以接收 3 个参数，其中<strong data-nodeid="1388">第二个参数就是一个真实的 DOM 节点</strong>，<strong data-nodeid="1389">这个真实的 DOM 节点充当“容器”的角色</strong>，React 元素最终会被渲染到这个“容器”里面去。比如，示例中的 App 组件，它对应的 render 调用是这样的：</p>
<pre class="lang-java" data-nodeid="1197"><code data-language="java"><span class="hljs-keyword">const</span> rootElement = document.getElementById(<span class="hljs-string">"root"</span>);
ReactDOM.render(&lt;App /&gt;, rootElement);
</code></pre>
<p data-nodeid="1198">注意，这个真实 DOM 一定是确实存在的。比如，在 App 组件对应的 index.html 中，已经提前预置 了 id 为 root 的根节点：</p>
<pre class="lang-js" data-nodeid="1199"><code data-language="js">&lt;body&gt;
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"root"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/body&gt;
</code></pre>

---

### 精选评论

##### **雨：
> 原来自己一直没弄明白JSX ，老师讲的很透彻，期待后面的课程

##### *忠：
> 有一处不太理解，希望能够指教下：React不能作为单独一个库使用吗，比如我在html里面直接引入react.min.js，然后写JSX使用，是必须要结合babel一起使用吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你可以在 html 里面直接引入 react.min.js，react 以什么样的形式引入不会对 JSX 构成什么影响；JSX 的问题在于浏览器没法直接识别它，所以我们需要一个编译器或者说具备编译能力的东东（一般是 Babel，或许有人也会用 Typescript 或者别的东西，这个选择是不定的，只能说大多数人更倾向于使用 Babel）来做这个转译的工作。

##### **侯：
> 秀妍写得太好啦，从掘金追过来的迷弟，忍不住想要催更

##### **5949：
> 请问构建出最终的虚拟DOM，应该需要递归的过程吧，能否麻烦补充完善一下递归的流程呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 专栏会涵盖这部分内容，可以关注下大纲和整体课程设计，虚拟DOM树的构建在第二模块。本课时归属于基础篇，知识讲解的侧重点和第二模块是不同的。

##### **汐文：
> 1.JSX即是React.createElement()的语法糖。2.ReactElement即是虚拟DOM，是将createElement出来的数据结构化。3.render即是把所有ReactElement挂载到一个真实的DOM容器上。

##### **伟：
> 大佬分享的太好了，深入浅出🙌

##### **6400：
> 1.React.createElement对JSX进行数据处理、清洗2.ReactElement是符合虚拟DOM规范的JS对象3.React.render将虚拟节点变成真实节点挂载在HTML上；

##### **9332：
> 播音主持专业的吗？太厉害了

##### *平：
> 所以其实不能脱离babel以及ReactDOM.render来看JSX，因为没有这两个它没法发挥作用

##### console_man：
> 秀妍大佬出品，必属精品

##### *攀：
> react17将createElement 改为了jsx函数，由编译器（bable）或者ts引入调用。

##### *梅：
> 期待老师后面的更新

##### *阳：
> 写的不错，之前阅读过源码、调试，老哥讲的算是一次深入学习了。

##### Tusi：
> 文笔不错，写作思路清晰！刚看到津津有味发现还没更，哈哈！

##### **楠：
> 自己写写，看看源码，收获会更大哦

##### *浩：
> 学习了！

##### **7107：
> 秀妍秀妍，我是德善😆😆😆

##### **用户5615：
> 很清晰，比很多一来就晒很多代码的好很多.

##### **东：
> ReactDOM.render方法的第二个参数作为真实DOM的容器可以是body吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题可以通过自己本地跑一个demo得到答案，也可以通过把专栏中的代码直接copy下来修改React.render方法的第二个参数得到答案。这两种途径都比口头提问对自己的帮助更大，不信你试试看。

##### *松：
> 老师太牛了，深入浅出，看了一遍理清了createElement的主流程，厉害厉害

##### *磊：
> 条例清晰，过程平滑，语言平实，很不错！

##### **清：
> 完全没学过react的直接学习会不会难度颇高？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 试着学习一下，要相信自己，有问题也可以在留言中提问哦

##### **宝：
> 清晰明确

##### **里的火：
> 厉害👍👍👍👍👍👍👍👍👍👍👍

##### EagleClark：
> 老师讲得很透彻啊，以前都没想过这些问题。

##### **森：
> 请问下老师， ReactDOM.render()中的第一个参数，是整个应用对应的完整的虚拟dom吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以这样理解。

##### **云：
> React.render将虚拟节点变成真实节点挂载在HTML上，这个地方是如何实现的呢？一直有疑问

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 专栏的第13-15讲详细地结合源码分析了这个问题，可以去看看。

##### **7679：
> 大佬讲解的非常好通俗易懂

##### *巍：
> 确实讲的很清晰，循序渐进，很有收获吗

##### **华：
> 文章的思路以及讲解比较透彻且容易理解， 很赞但是这里有个疑问： 文中提到JSX映射到DOM时， 只做了父子元素之间的初始化， 那么对于多层元素嵌套是怎么处理的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; JSX映射到DOM的详细过程在13-16讲有系统深入的分析，可以去看看。

##### **0064：
> 之前用的vue，没看过react，但是确实写的很清楚，看完这个对jsx到虚拟dom到挂在真实dom都很清晰了

##### **津：
> 讲的太好了 期待学习后面的课程

##### *莉：
> 重新认识一遍，有收获

##### **慧：
> 还得写，才能把学到的变成自己的~😋

##### **华：
> 大佬说的确实很清晰😀

##### **昊：
> 读了第一个课时，感觉非常好，让我有一个成体系的概念，期待下一个课时！

##### *飞：
> 老师讲的很好，受益匪浅

##### Loktar：
> 一直很喜欢React，老师的这种源码+注释+解析的方式真的非常的用心，对理解源码很有帮助，期待老师的后续课程更新，希望所有Reacter与React一起越走越远

##### *帅：
> 不错

##### *一：
> 就喜欢这种梳理性的文章

##### **涛：
> 比Vue多了一层Jsx😈

##### **的天空：
> 能请教下，上面得画图软件，是使用得什么？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; PPT哦

