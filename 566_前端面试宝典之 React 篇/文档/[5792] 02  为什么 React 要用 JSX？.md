<p data-nodeid="61284" class="">本讲我们一起来聊聊在面试中“为什么&nbsp;React 要用 JSX?”该如何回答。</p>
<h3 data-nodeid="61285">破题</h3>
<p data-nodeid="61286">初读一遍题目“为什么 React 要用 JSX？”，你可能会感觉有点怪怪的。这时你可以换个角度想一下，就好像有人在问你，“为什么你喜欢吃甜豆腐脑？”</p>
<p data-nodeid="61287">你是不是想迫不及待地写一首诗，赞美甜豆腐脑到底有多好吃呢？这你可就没答到点子上了。其实面试官的潜台词是“咸豆腐脑为什么不可以呢？”，对于这个问题来说是这样的。</p>
<p data-nodeid="61288">这便是我想着重告诉你的一个思路——通过比较论证的方式证明甜豆腐脑更胜一筹。</p>
<p data-nodeid="61289">当然，如果你是一位豆腐脑大师，甚至一名颇有威望的豆腐脑专家。那么，摆在大家面前对比的方案将会更多，甚至有酸辣豆腐脑、巧克力豆腐脑、韩式豆腐脑等等。所以，这里问“为什么用 JSX”，其引申含义是“为什么不用 A、B、C？”</p>
<p data-nodeid="61290">无论是面试还是晋升，“为什么采用该技术方案”这一类问题是主考官最爱提的。这类问题其实在考察你的两个方面：</p>
<ul data-nodeid="61291">
<li data-nodeid="61292">
<p data-nodeid="61293"><strong data-nodeid="61380">技术广度，深挖知识面涉猎广度，对流行框架的模板方案是否知悉了解；</strong></p>
</li>
<li data-nodeid="61294">
<p data-nodeid="61295"><strong data-nodeid="61384">技术方案调研能力。</strong></p>
</li>
</ul>
<p data-nodeid="61296">大多数时候，我们选取技术方案主要依靠直觉和习惯。这样既缺乏技术方案调研比对的过程，又缺乏个人的深度思考。所以这道题，如果<strong data-nodeid="61390">你的回答是“JSX 更简单易用，React 官方推荐”，当然不行</strong>！你要说服主考官，你就得拿出更多口味的“豆腐脑”进行比对才行。</p>
<h3 data-nodeid="61297">承题</h3>
<p data-nodeid="61298">通过以上的分析，我们可以使用**“三步走技巧”，即 “一句话解释，核心概念，方案对比”**的解题思路，来回答面试中“为什么 React 使用 JSX？”这类问题。</p>
<ol data-nodeid="61299">
<li data-nodeid="61300">
<p data-nodeid="61301">一句话解释 JSX。首先能一句话说清楚 JSX 到底是什么。</p>
</li>
<li data-nodeid="61302">
<p data-nodeid="61303">核心概念。JSX 用于解决什么问题？如何使用？</p>
</li>
<li data-nodeid="61304">
<p data-nodeid="61305">方案对比。与其他的方案对比，说明 React 选用 JSX 的必要性。</p>
</li>
</ol>
<p data-nodeid="61306"><img src="https://s0.lgstatic.com/i/image/M00/73/A4/Ciqc1F_GJQ-AK9FZAAC_MeElm70712.png" alt="图片1.png" data-nodeid="61404"></p>
<h3 data-nodeid="61307">入手</h3>
<h4 data-nodeid="61308">一句话解释</h4>
<p data-nodeid="61309">按照 React 官方的解释，<strong data-nodeid="61412">JSX 是一个 JavaScript 的语法扩展，或者说是一个类似于 XML 的 ECMAScript 语法扩展</strong>。它本身没有太多的语法定义，也不期望引入更多的标准。</p>
<p data-nodeid="61310">实际上，在 16 年的时候，JSX 公布过 2.0 的建设计划与小部分新特性，但很快被 Facebook 放弃掉了。整个计划在公布不到两个月的时间里便停掉了。其中一个原因是 JSX 的设计初衷，即并不希望引入太多的标准，也不期望 JSX 加入浏览器或者 ECMAScript 标准。</p>
<p data-nodeid="61311">那这是为什么呢？这就涉及了 JSX 的核心概念。</p>
<h4 data-nodeid="61312">核心概念</h4>
<p data-nodeid="61313">其实 React 本身并不强制使用 JSX。在没有 JSX 的时候，React 实现一个组件依赖于使用 React.createElement 函数。代码如下：</p>
<pre class="lang-java" data-nodeid="61314"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Hello</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-keyword">return</span> React.createElement(
        <span class="hljs-string">'div'</span>,
        <span class="hljs-keyword">null</span>, 
        `Hello ${<span class="hljs-keyword">this</span>.props.toWhat}`
      );
  }
}
ReactDOM.render(
  React.createElement(Hello, {toWhat: <span class="hljs-string">'World'</span>}, <span class="hljs-keyword">null</span>),
  document.getElementById(<span class="hljs-string">'root'</span>)
);
</code></pre>
<p data-nodeid="61315">而 JSX 更像是一种<strong data-nodeid="61422">语法糖</strong>，通过类似 XML 的描述方式，描写函数对象。在采用 JSX 之后，这段代码会这样写：</p>
<pre class="lang-js" data-nodeid="61316"><code data-language="js"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Hello</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">React</span>.<span class="hljs-title">Component</span> </span>{
  render() {
    <span class="hljs-keyword">return</span> <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>Hello {this.props.toWhat}<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>;
  }
}
ReactDOM.render(
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">Hello</span> <span class="hljs-attr">toWhat</span>=<span class="hljs-string">"World"</span> /&gt;</span></span>,
  <span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'root'</span>)
);
</code></pre>
<p data-nodeid="61317">通过这样的对比，你可以清晰地发现，<strong data-nodeid="61427">代码变得更为简洁，而且代码结构层次更为清晰。</strong></p>
<p data-nodeid="61318">因为 React 需要将组件转化为虚拟 DOM 树，所以我们在编写代码时，实际上是在手写一棵结构树。而<strong data-nodeid="61433">XML 在树结构的描述上天生具有可读性强的优势</strong>。</p>
<p data-nodeid="61319">但这样可读性强的代码仅仅是给写程序的同学看的，实际上在运行的时候，会使用 Babel 插件将 JSX 语法的代码还原为 React.createElement 的代码。</p>
<p data-nodeid="61320">那既然可以用插件帮我们编译转换代码，那为什么不直接使用模板呢？通过下一部分的方案对比可以解决你的问题。</p>
<h4 data-nodeid="61321">方案对比</h4>
<p data-nodeid="61322"><strong data-nodeid="61440">设计初衷</strong></p>
<p data-nodeid="61323">谈论其他方案之前，就需要谈到 React 的设计初衷，也是计算机科学里面一个非常重要的概念，叫作关注点分离（Separation of concerns）。</p>
<blockquote data-nodeid="61324">
<p data-nodeid="61325">关注点分离在计算机科学中，是将代码分隔为不同部分的设计原则，是面向对象的程序设计的核心概念。其中每一部分会有各自的关注焦点。</p>
<p data-nodeid="61326">关注点分离的价值在于简化程序的开发和维护。当关注点分开时，各部分可以重复使用，以及独立开发和更新。具有特殊价值的是能够稍后改进或修改一段代码，而无须知道其他部分的细节必须对这些部分进行相应的更改。</p>
</blockquote>
<p data-nodeid="61327">在 React 中，关注点的基本单位是组件。在接触一段时间 React 开发后，你会发现 React 单个组件是高内聚的，组件之间耦合度很低。</p>
<p data-nodeid="61328">那模板不能做到吗？</p>
<p data-nodeid="61329"><strong data-nodeid="61449">模板</strong></p>
<p data-nodeid="61330"><strong data-nodeid="61454">React 团队认为引入模板是一种不佳的实现。</strong> 因为模板分离了技术栈，而非关注点的模板同时又引入了更多的概念。比如新的模板语法、模板指令等，以 AngularJS 为例，我们可以看一下有多少新概念的引入。</p>
<pre class="lang-js" data-nodeid="61331"><code data-language="js">&lt;!doctype html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span> <span class="hljs-attr">ng-app</span>=<span class="hljs-string">"docsBindExample"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"http://code.angularjs.org/1.2.25/angular.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"script.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">ng-controller</span>=<span class="hljs-string">"Ctrl1"</span>&gt;</span>
      Hello <span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">ng-model</span>=<span class="hljs-string">'name'</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">hr</span>/&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">ng-bind</span>=<span class="hljs-string">"name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">br</span>/&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">ng:bind</span>=<span class="hljs-string">"name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">br</span>/&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">ng_bind</span>=<span class="hljs-string">"name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">br</span>/&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">data-ng-bind</span>=<span class="hljs-string">"name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">br</span>/&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">x-ng-bind</span>=<span class="hljs-string">"name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span> <span class="hljs-tag">&lt;<span class="hljs-name">br</span>/&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
angular.module('docsBindExample', [])
  .controller('Ctrl1', function Ctrl1($scope) {
    $scope.name = 'Max Karl Ernst Ludwig Planck (April 23, 1858 –        October 4, 1947)';
  });
</span></code></pre>
<p data-nodeid="61332">这段代码有很强的疏离感，引入了非常多 Angular 独有的概念。但 JSX 并不会引入太多新的概念，它仍然是 JavaScript，就连条件表达式和循环都仍然是 JavaScript 的方式。如下代码所示：</p>
<pre class="lang-js" data-nodeid="61333"><code data-language="js"><span class="hljs-keyword">const</span> App = <span class="hljs-function">(<span class="hljs-params">props</span>) =&gt;</span> {
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
       {props.isShow? <span class="hljs-tag">&lt;<span class="hljs-name">a</span>&gt;</span>show<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span> : <span class="hljs-tag">&lt;<span class="hljs-name">a</span>&gt;</span>hidden<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>}
       {props.names.map(name =&gt; <span class="hljs-tag">&lt;<span class="hljs-name">a</span>&gt;</span>{name}<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>)}
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  )
}
</code></pre>
<p data-nodeid="61334">即便是粗略的比较代码，也可以看出 React 代码更简洁，更具有可读性，更贴近 HTML。</p>
<p data-nodeid="61335">那模板字符串也可以套用 HTML，所以用模板字符串不行吗？</p>
<p data-nodeid="61336"><strong data-nodeid="61461">模板字符串</strong></p>
<p data-nodeid="61337">我们来看下面的例子：</p>
<pre class="lang-js" data-nodeid="61338"><code data-language="js"><span class="hljs-keyword">var</span> box = jsx<span class="hljs-string">`
  &lt;<span class="hljs-subst">${Box}</span>&gt;
    <span class="hljs-subst">${
      shouldShowAnswer(user) ?
      jsx<span class="hljs-string">`&lt;<span class="hljs-subst">${Answer}</span> value=<span class="hljs-subst">${<span class="hljs-literal">false</span>}</span>&gt;no&lt;/<span class="hljs-subst">${Answer}</span>&gt;`</span> :
      jsx<span class="hljs-string">`
        &lt;<span class="hljs-subst">${Box.Comment}</span>&gt;
         Text Content
        &lt;/<span class="hljs-subst">${Box.Comment}</span>&gt;
      `</span>
    }</span>
  &lt;/<span class="hljs-subst">${Box}</span>&gt;
`</span>;
</code></pre>
<p data-nodeid="61339">这显然不是一个容易的方案，代码结构变得更复杂了，而且开发工具的代码提示也会变得很困难。</p>
<p data-nodeid="61340"><strong data-nodeid="61467">JXON</strong></p>
<p data-nodeid="61341">JXON 非常类似于当下的 JSX，它的结构是这样的：</p>
<pre class="lang-js" data-nodeid="61342"><code data-language="js">&lt;catalog&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">product</span> <span class="hljs-attr">description</span>=<span class="hljs-string">"Cardigan Sweater"</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">catalog_item</span> <span class="hljs-attr">gender</span>=<span class="hljs-string">"Men's"</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">item_number</span>&gt;</span>QWZ5671<span class="hljs-tag">&lt;/<span class="hljs-name">item_number</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">price</span>&gt;</span>39.95<span class="hljs-tag">&lt;/<span class="hljs-name">price</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">size</span> <span class="hljs-attr">description</span>=<span class="hljs-string">"Medium"</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">color_swatch</span> <span class="hljs-attr">image</span>=<span class="hljs-string">"red_cardigan.jpg"</span>&gt;</span>Red<span class="hljs-tag">&lt;/<span class="hljs-name">color_swatch</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">color_swatch</span> <span class="hljs-attr">image</span>=<span class="hljs-string">"burgundy_cardigan.jpg"</span>&gt;</span>Burgundy<span class="hljs-tag">&lt;/<span class="hljs-name">color_swatch</span>&gt;</span>
     <span class="hljs-tag">&lt;/<span class="hljs-name">size</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">size</span> <span class="hljs-attr">description</span>=<span class="hljs-string">"Large"</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">color_swatch</span> <span class="hljs-attr">image</span>=<span class="hljs-string">"red_cardigan.jpg"</span>&gt;</span>Red<span class="hljs-tag">&lt;/<span class="hljs-name">color_swatch</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">color_swatch</span> <span class="hljs-attr">image</span>=<span class="hljs-string">"burgundy_cardigan.jpg"</span>&gt;</span>Burgundy<span class="hljs-tag">&lt;/<span class="hljs-name">color_swatch</span>&gt;</span>
     <span class="hljs-tag">&lt;/<span class="hljs-name">size</span>&gt;</span>
   <span class="hljs-tag">&lt;/<span class="hljs-name">catalog_item</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">catalog_item</span> <span class="hljs-attr">gender</span>=<span class="hljs-string">"Women's"</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">item_number</span>&gt;</span>RRX9856<span class="hljs-tag">&lt;/<span class="hljs-name">item_number</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">discount_until</span>&gt;</span>Dec 25, 1995<span class="hljs-tag">&lt;/<span class="hljs-name">discount_until</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">price</span>&gt;</span>42.50<span class="hljs-tag">&lt;/<span class="hljs-name">price</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">size</span> <span class="hljs-attr">description</span>=<span class="hljs-string">"Medium"</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">color_swatch</span> <span class="hljs-attr">image</span>=<span class="hljs-string">"black_cardigan.jpg"</span>&gt;</span>Black<span class="hljs-tag">&lt;/<span class="hljs-name">color_swatch</span>&gt;</span>
     <span class="hljs-tag">&lt;/<span class="hljs-name">size</span>&gt;</span>
   <span class="hljs-tag">&lt;/<span class="hljs-name">catalog_item</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">product</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"text/javascript"</span>&gt;</span><span class="handlebars"><span class="xml">&lt;![CDATA[function matchwo(a,b) {
    if (a &lt; b &amp;&amp; a &lt; 0) { return 1; }
    else { return 0; }
}]]&gt;</span></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
&lt;/catalog&gt;
</code></pre>
<p data-nodeid="61343">但最终放弃 JXON 这一方案的原因是，大括号不能为元素在树中开始和结束的位置，提供很好的语法提示。</p>
<h3 data-nodeid="61344">答题</h3>
<p data-nodeid="61345">经过以上的梳理，我们可以尝试答题了。</p>
<blockquote data-nodeid="61346">
<p data-nodeid="61347">在回答问题之前，我首先解释下什么是 JSX 吧。JSX 是一个 JavaScript 的语法扩展，结构类似 XML。</p>
<blockquote data-nodeid="61348">
<p data-nodeid="61349">JSX 主要用于声明 React 元素，但 React 中并不强制使用 JSX。即使使用了 JSX，也会在构建过程中，通过 Babel 插件编译为 React.createElement。所以 JSX 更像是 React.createElement 的一种语法糖。</p>
</blockquote>
<p data-nodeid="61350">所以从这里可以看出，React 团队并不想引入 JavaScript 本身以外的开发体系。而是希望通过合理的关注点分离保持组件开发的纯粹性。</p>
<p data-nodeid="61351">接下来与 JSX 以外的三种技术方案进行对比。</p>
<p data-nodeid="61352">首先是模板，React 团队认为模板不应该是开发过程中的关注点，因为引入了模板语法、模板指令等概念，是一种不佳的实现方案。</p>
<p data-nodeid="61353">其次是模板字符串，模板字符串编写的结构会造成多次内部嵌套，使整个结构变得复杂，并且优化代码提示也会变得困难重重。</p>
<p data-nodeid="61354">最后是 JXON，同样因为代码提示困难的原因而被放弃。</p>
<p data-nodeid="61355">所以 React 最后选用了 JSX，因为 JSX 与其设计思想贴合，不需要引入过多新的概念，对编辑器的代码提示也极为友好。</p>
</blockquote>
<p data-nodeid="61356">大家在学完这讲内容后，就可以对照以下知识导图，检验自己的学习成果了。</p>
<p data-nodeid="61357"><img src="https://s0.lgstatic.com/i/image/M00/73/A4/Ciqc1F_GJSSAU6odAAFLeX8UyTo307.png" alt="图片2.png" data-nodeid="61483"></p>
<h3 data-nodeid="61358">进阶</h3>
<p data-nodeid="61359"><strong data-nodeid="61489">Babel 插件如何实现 JSX 到 JS 的编译？</strong> 在 React 面试中，这个问题很容易被追问，也经常被要求手写。</p>
<p data-nodeid="61360">它的实现原理是这样的。Babel 读取代码并解析，生成 AST，再将 AST 传入插件层进行转换，在转换时就可以将 JSX 的结构转换为 React.createElement 的函数。如下代码所示：</p>
<pre class="lang-js" data-nodeid="61361"><code data-language="js"><span class="hljs-built_in">module</span>.exports = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">babel</span>) </span>{
&nbsp; <span class="hljs-keyword">var</span> t = babel.types;
&nbsp; <span class="hljs-keyword">return</span> {
&nbsp; &nbsp; <span class="hljs-attr">name</span>: <span class="hljs-string">"custom-jsx-plugin"</span>,
&nbsp; &nbsp; <span class="hljs-attr">visitor</span>: {
&nbsp; &nbsp; &nbsp; JSXElement(path) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> openingElement = path.node.openingElement;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> tagName = openingElement.name.name;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> args = [];&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; args.push(t.stringLiteral(tagName));&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> attribs = t.nullLiteral();&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; args.push(attribs);&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> reactIdentifier = t.identifier(<span class="hljs-string">"React"</span>); <span class="hljs-comment">//object</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> createElementIdentifier = t.identifier(<span class="hljs-string">"createElement"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> callee = t.memberExpression(reactIdentifier, createElementIdentifier)
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> callExpression = t.callExpression(callee, args);
&nbsp; &nbsp; &nbsp; &nbsp; callExpression.arguments = callExpression.arguments.concat(path.node.children);
&nbsp; &nbsp; &nbsp; &nbsp; path.replaceWith(callExpression, path.node);&nbsp;
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; },
&nbsp; };
};
</code></pre>
<p data-nodeid="61362">这里布置个小作业给大家：弄清楚这段代码是如何运行起来的。</p>
<h3 data-nodeid="61363">总结</h3>
<p data-nodeid="61364">本讲主要讲解了 React 选用 JSX 的原因，通过这一讲你可以掌握 JSX 的核心思想与替代方案。但似乎离组件的主题还有点儿距离，毕竟 JSX 只是组件的一种描述形式，真正到组件上还有诸如生命周期一类的东西。下一讲我就带你到组件的生命周期中去一探究竟。</p>
<p data-nodeid="61365">在进阶部分，我给你留了个小作业，你可以通过查询 Babel 的<a href="https://www.babeljs.cn/docs/plugins" data-nodeid="61497">开发文档</a>来解决这个问题。</p>
<p data-nodeid="61366">无论是在学习还是完成小作业的过程中遇到任何问题，都可以随时在留言区留言，我将与你共同探讨。</p>
<p data-nodeid="61367"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="61504"><img src="https://s0.lgstatic.com/i/image/M00/72/94/Ciqc1F_EZ0eANc6tAASyC72ZqWw643.png" alt="Drawing 2.png" data-nodeid="61503"></a></p>
<p data-nodeid="61368">《大前端高薪训练营》</p>
<p data-nodeid="61369" class="te-preview-highlight">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="61509">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **用户8654：
> 老师讲得实在是太好了

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 对呢，老师讲得思路很清晰，不仅可以学到知识点，还可以学会答题方法呢。

##### *浩：
> 老师回答面试题的方法确实有一套😀

##### **鑫：
> 思路清晰

##### **3970：
> ```javascriptmodule.exports">function">babel){">var">t">babel.types;">return{">name:">"custom-jsx-plugin",">/**转换是一个递归的搜索过程*/">visitor:{">JSXElement(path){">//获得当前节点的起始标签">var">openingElement">path.node.openingElement;">//名称">var">tagName">openingElement.name.name;">/**React.createElement的参数:childrens)*/">var">args">//tagName">args.push(t.stringLiteral(tagName));">//null">var">attribs">t.nullLiteral();">args.push(attribs);">//及其调用参数">var">reactIdentifier">t.identifier("React");">//object">var">createElementIdentifier">t.identifier("createElement");">var">callee">t.memberExpression(reactIdentifier,">createElementIdentifier)">var">callExpression">t.callExpression(callee,">args);">//中">callExpression.arguments">callExpression.arguments.concat(path.node.children);">//代码。">path.replaceWith(callExpression,">path.node);},},```

