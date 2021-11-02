<p data-nodeid="27241">JavaScript 在运行过程中与其他语言有所不一样，如果你不理解 JavaScript 的词法环境、执行上下文等内容，很容易会在开发过程中埋下“莫名奇妙”的 Bug，比如<code data-backticks="1" data-nodeid="27243">this</code>指向和预期不一致、某个变量不知道为什么被改了，等等。所以今天我就跟大家聊一聊 JavaScript 代码的运行过程。</p>



<p data-nodeid="25732">大家都知道，JavaScript 代码是需要在 JavaScript 引擎中运行的。我们在说到 JavaScript 运行的时候，常常会提到执行环境、词法环境、作用域、执行上下文、闭包等内容。这些概念看起来都差不多，却好像又不大容易区分清楚，它们分别都在描述什么呢？</p>
<p data-nodeid="25733">这些词语都是与 JavaScript 引擎执行代码的过程有关，为了搞清楚这些概念之间的区别，我们可以回顾下 JavaScript 代码运行过程中的各个阶段。</p>
<h3 data-nodeid="28245" class="">JavaScript 代码运行的各个阶段</h3>

<p data-nodeid="25735">JavaScript 是弱类型语言，在运行时才能确定变量类型。即使是如今流行的 TypeScript，也只是增加了编译时（编译成 JavaScript）的类型检测（对于编译器相信大家都有所了解，代码编译过程中编译器会进行词法分析、语法分析、语义分析、生成 AST 等处理）。</p>
<p data-nodeid="25736">同样，JavaScript 引擎在执行 JavaScript 代码时，也会从上到下进行词法分析、语法分析、语义分析等处理，并在代码解析完成后生成 AST（抽象语法树），最终根据 AST 生成 CPU 可以执行的机器码并执行。</p>
<p data-nodeid="25737">这个过程，我们后面统一描述为语法分析阶段。除了语法分析阶段，JavaScript 引擎在执行代码时还会进行其他的处理。以 V8 引擎为例，在 V8 引擎中 JavaScript 代码的运行过程主要分成三个阶段。</p>
<ol data-nodeid="31294">
<li data-nodeid="31295">
<p data-nodeid="31296"><strong data-nodeid="31305">语法分析阶段。</strong> 该阶段会对代码进行语法分析，检查是否有语法错误（SyntaxError），如果发现语法错误，会在控制台抛出异常并终止执行。</p>
</li>
<li data-nodeid="31297">
<p data-nodeid="31298"><strong data-nodeid="31310">编译阶段。</strong> 该阶段会进行执行上下文（Execution Context）的创建，包括创建变量对象、建立作用域链、确定 this 的指向等。每进入一个不同的运行环境时，V8 引擎都会创建一个新的执行上下文。</p>
</li>
<li data-nodeid="31299">
<p data-nodeid="31300" class=""><strong data-nodeid="31315">执行阶段。</strong> 将编译阶段中创建的执行上下文压入调用栈，并成为正在运行的执行上下文，代码执行结束后，将其弹出调用栈。</p>
</li>
</ol>



<p data-nodeid="25745">其中，语法分析阶段属于编译器通用内容，就不再赘述。前面提到的执行环境、词法环境、作用域、执行上下文等内容都是在编译和执行阶段中产生的概念。</p>
<blockquote data-nodeid="25746">
<p data-nodeid="25747">关于调用栈的内容我们会在下一讲详细讲解，目前我们只需要知道 JavaScript 在运行过程中会产生一个调用栈，调用栈遵循 LIFO（先进后出，后进先出）原则即可。</p>
</blockquote>
<p data-nodeid="25748">今天，我们重点介绍编译阶段，而编译阶段的核心便是执行上下文的创建。</p>
<h3 data-nodeid="32322" class="">执行上下文的创建</h3>

<p data-nodeid="25750">执行上下文的创建离不开 JavaScript 的运行环境，JavaScript 运行环境包括全局环境、函数环境和<code data-backticks="1" data-nodeid="25933">eval</code>，其中全局环境和函数环境的创建过程如下：</p>
<ol data-nodeid="25751">
<li data-nodeid="25752">
<p data-nodeid="25753">第一次载入 JavaScript 代码时，首先会创建一个全局环境。全局环境位于最外层，直到应用程序退出后（例如关闭浏览器和网页）才会被销毁。</p>
</li>
<li data-nodeid="25754">
<p data-nodeid="25755">每个函数都有自己的运行环境，当函数被调用时，则会进入该函数的运行环境。当该环境中的代码被全部执行完毕后，该环境会被销毁。不同的函数运行环境不一样，即使是同一个函数，在被多次调用时也会创建多个不同的函数环境。</p>
</li>
</ol>
<p data-nodeid="25756">在不同的运行环境中，变量和函数可访问的其他数据范围不同，环境的行为（比如创建和销毁）也有所区别。而每进入一个不同的运行环境时，JavaScript 都会创建一个新的执行上下文，该过程包括：</p>
<ul data-nodeid="25757">
<li data-nodeid="25758">
<p data-nodeid="25759">建立作用域链（Scope Chain）；</p>
</li>
<li data-nodeid="25760">
<p data-nodeid="25761">创建变量对象（Variable Object，简称 VO）；</p>
</li>
<li data-nodeid="25762">
<p data-nodeid="25763">确定 this 的指向。</p>
</li>
</ul>
<p data-nodeid="25764">由于建立作用域链过程中会涉及变量对象的概念，因此我们先来看看变量对象的创建，再看建立作用域链和确定 this 的指向。</p>
<h4 data-nodeid="33330" class="">创建变量对象</h4>

<p data-nodeid="25766">什么是变量对象呢？每个执行上下文都会有一个关联的变量对象，该对象上会保存这个上下文中定义的所有变量和函数。</p>
<p data-nodeid="25767">而在浏览器中，全局环境的变量对象是<code data-backticks="1" data-nodeid="25945">window</code>对象，因此所有的全局变量和函数都是作为<code data-backticks="1" data-nodeid="25947">window</code>对象的属性和方法创建的。相应的，在 Node 中全局环境的变量对象则是<code data-backticks="1" data-nodeid="25949">global</code>对象。</p>
<p data-nodeid="25768">了解了什么是变量对象之后，我们来看下创建变量对象的过程。创建变量对象将会创建<code data-backticks="1" data-nodeid="25952">arguments</code>对象（仅函数环境下），同时会检查当前上下文的函数声明和变量声明。</p>
<ul data-nodeid="25769">
<li data-nodeid="25770">
<p data-nodeid="25771">对于变量声明：此时会给变量分配内存，并将其初始化为<code data-backticks="1" data-nodeid="25955">undefined</code>（该过程只进行定义声明，执行阶段才执行赋值语句）。</p>
</li>
<li data-nodeid="25772">
<p data-nodeid="25773">对于函数声明：此时会在内存里创建函数对象，并且直接初始化为该函数对象。</p>
</li>
</ul>
<p data-nodeid="25774">上述变量声明和函数声明的处理过程，便是我们常说的变量提升和函数提升，其中函数声明提升会优先于变量声明提升。因为变量提升容易带来变量在预期外被覆盖掉的问题，同时还可能导致本应该被销毁的变量没有被销毁等情况。因此 ES6 中引入了<code data-backticks="1" data-nodeid="25959">let</code>和<code data-backticks="1" data-nodeid="25961">const</code>关键字，从而使 JavaScript 也拥有了块级作用域。</p>
<p data-nodeid="25775">或许你会感到疑惑，JavaScript 是怎么支持块级作用域的呢？这就涉及作用域的概念。</p>
<p data-nodeid="25776">在各类编程语言中，作用域分为静态作用域和动态作用域。JavaScript 采用的是词法作用域（Lexical Scoping），也就是静态作用域。词法作用域中的变量，在编译过程中会产生一个确定的作用域。</p>
<p data-nodeid="25777">到这里，或许你对会词法作用域、作用域、执行上下文、词法环境之间的关系依然感到混乱，没关系，我这就来给你梳理下。</p>
<p data-nodeid="25778">刚刚说到，词法作用域中的变量，在编译过程中会产生一个确定的作用域，这个作用域即当前的执行上下文，在 ES5 后我们使用词法环境（Lexical Environment）替代作用域来描述该执行上下文。因此，词法环境可理解为我们常说的作用域，同样也指当前的执行上下文（注意，是当前的执行上下文）。</p>
<p data-nodeid="25779">在 JavaScript 中，词法环境又分为词法环境（Lexical Environment）和变量环境（Variable Environment）两种，其中：</p>
<ul data-nodeid="25780">
<li data-nodeid="25781">
<p data-nodeid="25782">变量环境用来记录<code data-backticks="1" data-nodeid="25969">var</code>/<code data-backticks="1" data-nodeid="25971">function</code>等变量声明；</p>
</li>
<li data-nodeid="25783">
<p data-nodeid="25784">词法环境是用来记录<code data-backticks="1" data-nodeid="25974">let</code>/<code data-backticks="1" data-nodeid="25976">const</code>/<code data-backticks="1" data-nodeid="25978">class</code>等变量声明。</p>
</li>
</ul>
<p data-nodeid="25785">也就是说，创建变量过程中会进行函数提升和变量提升，JavaScript 会通过词法环境来记录函数和变量声明。通过使用两个词法环境（而不是一个）分别记录不同的变量声明内容，JavaScript 实现了支持块级作用域的同时，不影响原有的变量声明和函数声明。</p>
<p data-nodeid="25786">这就是创建变量的过程，它属于执行上下文创建中的一环。创建变量的过程会产生作用域，作用域也被称为词法环境，那词法环境是由什么组成的呢？下面我结合作用域链的建立过程一起来进行分析。</p>
<h4 data-nodeid="34338" class="">建立作用域链</h4>

<p data-nodeid="25788">作用域链，顾名思义，就是将各个作用域通过某种方式连接在一起。</p>
<p data-nodeid="25789">前面说过，作用域就是词法环境，而词法环境由两个成员组成。</p>
<ul data-nodeid="25790">
<li data-nodeid="25791">
<p data-nodeid="25792">环境记录（Environment Record）：用于记录自身词法环境中的变量对象。</p>
</li>
<li data-nodeid="25793">
<p data-nodeid="25794">外部词法环境引用（Outer Lexical Environment）：记录外层词法环境的引用。</p>
</li>
</ul>
<p data-nodeid="25795">通过外部词法环境的引用，作用域可以层层拓展，建立起从里到外延伸的一条作用域链。当某个变量无法在自身词法环境记录中找到时，可以根据外部词法环境引用向外层进行寻找，直到最外层的词法环境中外部词法环境引用为<code data-backticks="1" data-nodeid="25988">null</code>，这便是作用域链的变量查询。</p>
<p data-nodeid="25796">那么，这个外部词法环境引用又是怎样指向外层呢？我们来看看 JavaScript 中是如何通过外部词法环境引用来创建作用域的。</p>
<p data-nodeid="25797">为了方便描述，我们将 JavaScript 代码运行过程分为定义期和执行期，前面提到的编译阶段则属于定义期。</p>
<p data-nodeid="25798">来看一个例子，我们定义了全局函数<code data-backticks="1" data-nodeid="25993">foo</code>，并在该函数中定义了函数<code data-backticks="1" data-nodeid="25995">bar</code>：</p>
<pre class="lang-java" data-nodeid="25799"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">foo</span><span class="hljs-params">()</span> </span>{
 &nbsp;console.dir(bar);
 &nbsp;<span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>;
 &nbsp;<span class="hljs-function">function <span class="hljs-title">bar</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp;a = <span class="hljs-number">2</span>;
  }
}
console.dir(foo);
foo();
</code></pre>
<p data-nodeid="25800">前面我们说到，JavaScript 使用的是静态作用域，因此函数的作用域在定义期已经决定了。在上面的例子中，全局函数<code data-backticks="1" data-nodeid="25998">foo</code>创建了一个<code data-backticks="1" data-nodeid="26000">foo</code>的<code data-backticks="1" data-nodeid="26002">[[scope]]</code>属性，包含了全局<code data-backticks="1" data-nodeid="26004">[[scope]]</code>：</p>
<pre class="lang-java" data-nodeid="25801"><code data-language="java">foo[[scope]] = [globalContext];
</code></pre>
<p data-nodeid="25802">而当我们执行<code data-backticks="1" data-nodeid="26007">foo()</code>时，也会分别进入<code data-backticks="1" data-nodeid="26009">foo</code>函数的定义期和执行期。</p>
<p data-nodeid="25803">在<code data-backticks="1" data-nodeid="26012">foo</code>函数的定义期时，函数<code data-backticks="1" data-nodeid="26014">bar</code>的<code data-backticks="1" data-nodeid="26016">[[scope]]</code>将会包含全局<code data-backticks="1" data-nodeid="26018">[[scope]]</code>和<code data-backticks="1" data-nodeid="26020">foo</code>的<code data-backticks="1" data-nodeid="26022">[[scope]]</code>：</p>
<pre class="lang-java" data-nodeid="25804"><code data-language="java">bar[[scope]] = [fooContext, globalContext];
</code></pre>
<p data-nodeid="25805">运行上述代码，我们可以在控制台看到符合预期的输出：</p>
<p data-nodeid="35346" class=""><img src="https://s0.lgstatic.com/i/image6/M01/37/18/Cgp9HWB1uyGAAaZIAAK9qHI3wvE362.png" alt="图片2.png" data-nodeid="35349"></p>


<p data-nodeid="25808">可以看到：</p>
<ul data-nodeid="25809">
<li data-nodeid="25810">
<p data-nodeid="25811"><code data-backticks="1" data-nodeid="26032">foo</code>的<code data-backticks="1" data-nodeid="26034">[[scope]]</code>属性包含了全局<code data-backticks="1" data-nodeid="26036">[[scope]]</code></p>
</li>
<li data-nodeid="25812">
<p data-nodeid="25813"><code data-backticks="1" data-nodeid="26037">bar</code>的<code data-backticks="1" data-nodeid="26039">[[scope]]</code>将会包含全局<code data-backticks="1" data-nodeid="26041">[[scope]]</code>和<code data-backticks="1" data-nodeid="26043">foo</code>的<code data-backticks="1" data-nodeid="26045">[[scope]]</code></p>
</li>
</ul>
<p data-nodeid="25814">也就是说，<strong data-nodeid="26051">JavaScript 会通过外部词法环境引用来创建变量对象的一个作用域链，从而保证对执行环境有权访问的变量和函数的有序访问</strong>。除了创建作用域链之外，在这个过程中还会对创建的变量对象做一些处理。</p>
<p data-nodeid="25815">前面我们说过，编译阶段会进行变量对象（VO）的创建，该过程会进行函数声明和变量声明，这时候变量的值被初始化为 undefined。在代码进入执行阶段之后，JavaScript 会对变量进行赋值，此时变量对象会转为活动对象（Active Object，简称 AO），转换后的活动对象才可被访问，这就是 VO -&gt; AO 的过程。</p>
<p data-nodeid="25816">为了更好地理解这个过程，我们来看个例子，我们在<code data-backticks="1" data-nodeid="26054">foo</code>函数中定义了变量<code data-backticks="1" data-nodeid="26056">b</code>、函数<code data-backticks="1" data-nodeid="26058">c</code>和函数表达式变量<code data-backticks="1" data-nodeid="26060">d</code>：</p>
<pre class="lang-java" data-nodeid="25817"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">foo</span><span class="hljs-params">(a)</span> </span>{
 &nbsp;<span class="hljs-keyword">var</span> b = <span class="hljs-number">2</span>;
 &nbsp;<span class="hljs-function">function <span class="hljs-title">c</span><span class="hljs-params">()</span> </span>{}
 &nbsp;<span class="hljs-keyword">var</span> d = function() {};
}
​
foo(<span class="hljs-number">1</span>);
</code></pre>
<p data-nodeid="25818">在执行<code data-backticks="1" data-nodeid="26063">foo(1)</code>时，首先进入定义期，此时：</p>
<ul data-nodeid="25819">
<li data-nodeid="25820">
<p data-nodeid="25821">参数变量<code data-backticks="1" data-nodeid="26066">a</code>的值为<code data-backticks="1" data-nodeid="26068">1</code></p>
</li>
<li data-nodeid="25822">
<p data-nodeid="25823">变量<code data-backticks="1" data-nodeid="26070">b</code>和<code data-backticks="1" data-nodeid="26072">d</code>初始化为<code data-backticks="1" data-nodeid="26074">undefined</code></p>
</li>
<li data-nodeid="25824">
<p data-nodeid="25825">函数<code data-backticks="1" data-nodeid="26076">c</code>创建函数并初始化</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="25826"><code data-language="java">AO = {
 &nbsp;arguments: {
 &nbsp; &nbsp;<span class="hljs-number">0</span>: <span class="hljs-number">1</span>,
 &nbsp; &nbsp;length: <span class="hljs-number">1</span>
  },
 &nbsp;a: <span class="hljs-number">1</span>,
 &nbsp;b: undefined,
 &nbsp;c: <span class="hljs-function">reference to function <span class="hljs-title">c</span><span class="hljs-params">()</span></span>{},
 &nbsp;d: undefined
}
</code></pre>
<p data-nodeid="25827">前面我们也有提到，进入执行期之后，会执行赋值语句进行赋值，此时变量<code data-backticks="1" data-nodeid="26079">b</code>和<code data-backticks="1" data-nodeid="26081">d</code>会被赋值为 2 和函数表达式：</p>
<pre class="lang-java" data-nodeid="25828"><code data-language="java">AO = {
  &nbsp;arguments: {
 &nbsp; &nbsp;<span class="hljs-number">0</span>: <span class="hljs-number">1</span>,
 &nbsp; &nbsp;length: <span class="hljs-number">1</span>
  },
 &nbsp;a: <span class="hljs-number">1</span>,
 &nbsp;b: <span class="hljs-number">2</span>,
 &nbsp;c: <span class="hljs-function">reference to function <span class="hljs-title">c</span><span class="hljs-params">()</span></span>{},
 &nbsp;d: reference to FunctionExpression <span class="hljs-string">"d"</span>
}
</code></pre>
<p data-nodeid="25829">这就是 VO -&gt; AO 过程。</p>
<ul data-nodeid="25830">
<li data-nodeid="25831">
<p data-nodeid="25832">在定义期（编译阶段）：该对象值仍为<code data-backticks="1" data-nodeid="26085">undefined</code>，且处于不可访问的状态。</p>
</li>
<li data-nodeid="25833">
<p data-nodeid="25834">进入执行期（执行阶段）：VO 被激活，其中变量属性会进行赋值。</p>
</li>
</ul>
<p data-nodeid="25835">实际上在执行的时候，除了 VO 被激活，活动对象还会添加函数执行时传入的参数和<code data-backticks="1" data-nodeid="26089">arguments</code>这个特殊对象，因此 AO 和 VO 的关系可以用以下关系来表达：</p>
<pre class="lang-java" data-nodeid="25836"><code data-language="java">AO = VO + function parameters + arguments
</code></pre>
<p data-nodeid="25837">现在，我们知道作用域链是在进入代码的执行阶段时，通过外部词法环境引用来创建的。总结如下：</p>
<ul data-nodeid="25838">
<li data-nodeid="25839">
<p data-nodeid="25840">在编译阶段，JavaScript 在创建执行上下文的时候会先创建变量对象（VO）；</p>
</li>
<li data-nodeid="25841">
<p data-nodeid="25842">在执行阶段，变量对象（VO）被激活为活动对象（ AO），函数内部的变量对象通过外部词法环境的引用创建作用域链。</p>
</li>
</ul>
<p data-nodeid="25843">虽然 JavaScript 代码的运行过程可以分为语法分析阶段、编译阶段和执行阶段，但由于在 JavaScript 引擎中是通过调用栈的方式来执行 JavaScript 代码的（下一讲会介绍），因此并不存在“整个 JavaScript 运行过程只会在某个阶段中”这一说法，比如上面例子中<code data-backticks="1" data-nodeid="26095">bar</code>函数的编译阶段，其实是在<code data-backticks="1" data-nodeid="26097">foo</code>函数的执行阶段中。</p>
<p data-nodeid="25844">一般来说，当函数执行结束之后，执行期上下文将被销毁（作用域链和活动对象均被销毁）。但有时候我们想要保留其中一些变量对象，不想被销毁，此时就会使用到闭包。</p>
<p data-nodeid="25845">我们已经知道，通过作用域链，我们可以在函数内部可以直接读取外部以及全局变量，但外部环境是无法访问内部函数里的变量。比如下面的例子中，<code data-backticks="1" data-nodeid="26101">foo</code>函数中定义了变量<code data-backticks="1" data-nodeid="26103">a</code>：</p>
<pre class="lang-java" data-nodeid="25846"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">foo</span><span class="hljs-params">()</span> </span>{
 &nbsp;<span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>;
}
foo();
console.log(a); <span class="hljs-comment">// undefined</span>
</code></pre>
<p data-nodeid="25847">我们在全局环境下无法访问函数<code data-backticks="1" data-nodeid="26106">foo</code>中的变量<code data-backticks="1" data-nodeid="26108">a</code>，这是因为全局函数的作用域链里，不含有函数<code data-backticks="1" data-nodeid="26110">foo</code>内的作用域。</p>
<p data-nodeid="25848">如果我们想要访问内部函数的变量，可以通过函数<code data-backticks="1" data-nodeid="26113">foo</code>中的函数<code data-backticks="1" data-nodeid="26115">bar</code>返回变量<code data-backticks="1" data-nodeid="26117">a</code>，并将函数<code data-backticks="1" data-nodeid="26119">bar</code>返回，这样我们在全局环境中也可以通过调用函数<code data-backticks="1" data-nodeid="26121">foo</code>返回的函数<code data-backticks="1" data-nodeid="26123">bar</code>，来访问变量<code data-backticks="1" data-nodeid="26125">a</code>：</p>
<pre class="lang-java" data-nodeid="25849"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">foo</span><span class="hljs-params">()</span> </span>{
  <span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>;
  <span class="hljs-function">function <span class="hljs-title">bar</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> a;
  }
  <span class="hljs-keyword">return</span> bar;
}
<span class="hljs-keyword">var</span> b = foo();
console.log(b()); <span class="hljs-comment">// 1</span>
</code></pre>
<p data-nodeid="25850">前面我们说到，当函数执行结束之后，执行期上下文将被销毁，其中包括作用域链和激活对象。那么，在这个例子中，当<code data-backticks="1" data-nodeid="26128">b()</code>执行时，<code data-backticks="1" data-nodeid="26130">foo</code>函数上下文包括作用域都已经被销毁了，为什么<code data-backticks="1" data-nodeid="26132">foo</code>作用域下的<code data-backticks="1" data-nodeid="26134">a</code>依然可以被访问到呢？</p>
<p data-nodeid="25851">这是因为<code data-backticks="1" data-nodeid="26137">bar</code>函数引用了<code data-backticks="1" data-nodeid="26139">foo</code>函数变量对象中的值，此时即使创建<code data-backticks="1" data-nodeid="26141">bar</code>函数的<code data-backticks="1" data-nodeid="26143">foo</code>函数执行上下文被销毁了，但它的变量对象依然会保留在 JavaScript 内存中，<code data-backticks="1" data-nodeid="26145">bar</code>函数依然可以通过<code data-backticks="1" data-nodeid="26147">bar</code>函数的作用域链找到它，并进行访问。这便是我们常说的闭包，即使创建它的上下文已经销毁，它仍然被保留在内存中。</p>
<p data-nodeid="25852">闭包使得我们可以从外部读取局部变量，在大多数项目中都会被使用到，常见的用途包括：</p>
<ul data-nodeid="25853">
<li data-nodeid="25854">
<p data-nodeid="25855">用于从外部读取其他函数内部变量的函数；</p>
</li>
<li data-nodeid="25856">
<p data-nodeid="25857">可以使用闭包来模拟私有方法；</p>
</li>
<li data-nodeid="25858">
<p data-nodeid="25859">让这些变量的值始终保持在内存中。</p>
</li>
</ul>
<p data-nodeid="25860">需要注意的是，我们在使用闭包的时候，需要及时清理不再使用到的变量，否则可能导致内存泄漏问题。</p>
<p data-nodeid="25861">相信大家现在已经掌握了作用域链的建立过程，那么作用域链的用途想必大家也已经了解，比如在函数执行过程中变量的解析：</p>
<ul data-nodeid="25862">
<li data-nodeid="25863">
<p data-nodeid="25864">从当前词法环境开始，沿着作用域链逐级向外层寻找环境记录，直到找到同名变量为止；</p>
</li>
<li data-nodeid="25865">
<p data-nodeid="25866">找到后不再继续遍历，找不到就报错。</p>
</li>
</ul>
<p data-nodeid="25867">下面我们继续来看，执行上下文的创建过程中还会做的一件事：确定<code data-backticks="1" data-nodeid="26158">this</code>的指向。</p>
<h4 data-nodeid="38350" class="te-preview-highlight">确定 this 的指向</h4>

<p data-nodeid="25869">在 JavaScript 中，<code data-backticks="1" data-nodeid="26162">this</code>指向执行当前代码对象的所有者，可简单理解为<code data-backticks="1" data-nodeid="26164">this</code>指向最后调用当前代码的那个对象。相信大家都很熟悉<code data-backticks="1" data-nodeid="26166">this</code>，因此这里我就进行结论性的简单总结。</p>
<p data-nodeid="25870">根据 JavaScript 中函数的调用方式不同，<code data-backticks="1" data-nodeid="26169">this</code>的指向分为以下情况。</p>
<p data-nodeid="36348" class=""><img src="https://s0.lgstatic.com/i/image6/M01/37/18/Cgp9HWB1uzSAQvuHAAJh7k1PAh8263.png" alt="this 指向.png" data-nodeid="36351"></p>

<ul data-nodeid="25872">
<li data-nodeid="25873">
<p data-nodeid="25874">在全局环境中，<code data-backticks="1" data-nodeid="26175">this</code>指向全局对象（在浏览器中为<code data-backticks="1" data-nodeid="26177">window</code>）</p>
</li>
<li data-nodeid="25875">
<p data-nodeid="25876">在函数内部，<code data-backticks="1" data-nodeid="26180">this</code>的值取决于函数被调用的方式</p>
<ul data-nodeid="25877">
<li data-nodeid="25878">
<p data-nodeid="25879">函数作为对象的方法被调用，<code data-backticks="1" data-nodeid="26183">this</code>指向调用这个方法的对象</p>
</li>
<li data-nodeid="25880">
<p data-nodeid="25881">函数用作构造函数时（使用<code data-backticks="1" data-nodeid="26186">new</code>关键字），它的<code data-backticks="1" data-nodeid="26188">this</code>被绑定到正在构造的新对象</p>
</li>
<li data-nodeid="25882">
<p data-nodeid="25883">在类的构造函数中，<code data-backticks="1" data-nodeid="26191">this</code>是一个常规对象，类中所有非静态的方法都会被添加到<code data-backticks="1" data-nodeid="26193">this</code>的原型中</p>
</li>
</ul>
</li>
<li data-nodeid="25884">
<p data-nodeid="25885">在箭头函数中，<code data-backticks="1" data-nodeid="26196">this</code>指向它被创建时的环境</p>
</li>
<li data-nodeid="25886">
<p data-nodeid="25887">使用<code data-backticks="1" data-nodeid="26199">apply</code>、<code data-backticks="1" data-nodeid="26201">call</code>、<code data-backticks="1" data-nodeid="26203">bind</code>等方式调用：根据 API 不同，可切换函数执行的上下文环境，即<code data-backticks="1" data-nodeid="26205">this</code>绑定的对象</p>
</li>
</ul>
<p data-nodeid="25888">可以看到，<code data-backticks="1" data-nodeid="26208">this</code>在不同的情况下会有不同的指向，在 ES6 箭头函数还没出现之前，为了能正确获取某个运行环境下<code data-backticks="1" data-nodeid="26210">this</code>对象，我们常常会使用<code data-backticks="1" data-nodeid="26212">var that = this;</code>、<code data-backticks="1" data-nodeid="26214">var self = this;</code>这样的代码将变量分配给<code data-backticks="1" data-nodeid="26216">this</code>，便于使用。这种方式降低了代码可读性，因此如今这种做法不再被提倡，通过正确使用箭头函数，我们可以更好地管理作用域。</p>
<p data-nodeid="25889">到这里，围绕 JavaScript 的编译阶段和执行阶段中执行上下文创建相关的内容已经介绍完毕。</p>
<h3 data-nodeid="37350" class="">小结</h3>

<p data-nodeid="25891">今天我主要介绍了 JavaScript 代码的运行过程，该过程分为语法分析阶段、编译阶段、执行阶段三个阶段。</p>
<p data-nodeid="25892">在编译阶段，JavaScript会进行执行上下文的创建，包括：</p>
<ul data-nodeid="25893">
<li data-nodeid="25894">
<p data-nodeid="25895">创建变量对象，进行变量声明和函数声明，此时会产生变量提升和函数提升；</p>
</li>
<li data-nodeid="25896">
<p data-nodeid="25897">通过添加对外部词法环境的引用，建立作用域链，通过作用域链可以访问外部的变量对象；</p>
</li>
<li data-nodeid="25898">
<p data-nodeid="25899">确定 this 的指向。</p>
</li>
</ul>
<p data-nodeid="25900">在执行阶段，变量对象（VO）会被激活为活动对象（AO），变量会进行赋值，此时活动对象才可被访问。在执行结束之后，作用域链和活动对象均被销毁，使用闭包可使活动对象依然被保留在内存中。这就是 JavaScript 代码的运行过程。</p>
<p data-nodeid="25901">我们前面也说过，下面这段代码中<code data-backticks="1" data-nodeid="26227">bar</code>函数的编译阶段是在<code data-backticks="1" data-nodeid="26229">foo</code>函数的执行阶段中 ：</p>
<pre class="lang-java" data-nodeid="25902"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">foo</span><span class="hljs-params">()</span> </span>{
 &nbsp;console.dir(bar);
 &nbsp;<span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>;
 &nbsp;<span class="hljs-function">function <span class="hljs-title">bar</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp;a = <span class="hljs-number">2</span>;
  }
}
console.dir(foo);
foo();
</code></pre>
<p data-nodeid="25903">你能说出整段代码的运行过程分别是怎样的，变量对象 AO/VO、作用域链、this 指向在各个阶段中又会怎样表现呢？可以把你的想法写在留言区。</p>
<p data-nodeid="25904">其实，JavaScript 的运行过程和 EventLoop 结合可以有更好的理解，关于 EventLoop 我会在下一讲进行介绍，你也可以在学习之后再来结合本讲内容进行总结。</p>

---

### 精选评论

##### **2951：
> 首先将全局执行上下文栈window压入执行栈中，全局执行上下文中， foo是函数声明，被提升，函数声明提升要优先于变量提升，此时全局执行上下文栈中，只包含一个foo函数声明，同时将全局作用域绑定在foo的作用域链上，执行foo函数时，又为foo新创建一个执行上下文栈，并压入执行栈中，此时执行权由全局window，交到了foo函数上，进行编译阶段的工作，将变量a提升，bar函数声明提升，此时foo的执行上下文栈中，a是undefined， bar是一个函数声明，bar的作用域包含全局作用域和foo的作用域，执行阶段， a 赋值为1， 此时a变量对象变为了活动对象，this指向window。foo函数执行完毕，将foo从执行栈中弹出，执行权又交给window。

##### *雨：
> 这一节开始吃力了，有的名词以前没听过

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 哪些名词呢？可以具体说下，小编和讲师也可以针对性的帮你喔！

##### *浩：
> 再学基础知识。

##### **宇：
> vo里面有a b变量，a可以通过闭包被外界访问，b没有，当前作用域执行完后，整个vo都被保留，还是只保留a这块内存？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当函数执行结束之后，执行期上下文将被销毁，其中包括作用域链和激活对象。对于闭包的场景来说，函数执行结束后，执行上下文的作用域链会被销毁，但它的激活对象仍然会被保留在内测中，这里是整个激活对象都会被保存。
因为闭包会保留锁包含函数的作用域，所以会比其他函数更占用内存。

##### **4391洪育煌：
> 老师介绍的创建作用域链是在执行阶段出现的，那上文中创建执行上下文不是在编译阶段吗？执行上下文包含了创建作用域链不是矛盾了吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 文中有说，虽然 JavaScript 代码的运行过程可以分为语法分析阶段、编译阶段和执行阶段，但由于在 JavaScript 引擎中是通过调用栈的方式来执行 JavaScript 代码的（下一讲会介绍），因此并不存在“整个 JavaScript 运行过程只会在某个阶段中”这一说法，比如上面例子中 bar 函数的编译阶段，其实是在 foo 函数的执行阶段中。

##### **泳：
> "创建变量对象将会创建arguments对象（仅函数环境下）","除了 VO 被激活，活动对象还会添加函数执行时传入的参数和arguments这个特殊对象"。老师您好！arguments对象在编译阶段就创建了，VO=AO也就意味着在执行阶段，为什么在执行阶段又会添加一次arguments对象?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在创建变量的时候，arguments 对象只是形参，进入代码的执行阶段时，真实的参数才会被传进来，最终代码的执行是以真实的传参为准的。

##### *军：
> 问一个问题呀，箭头函数内的执行上下文啥样的，没有arguments对象了吧，另外this应该指向定义箭头函数所在的执行上下文this吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 箭头函数表达式的语法比函数表达式更简洁，并且没有自己的 this，arguments，super 或 new.target。因此，箭头函数的 this 指向它被创建时的环境。

##### *晶：
> 有点点疑问1.编译器分析和v8引擎执行是两部分吗?词法分析、语法分析、语义分析、生成 AST会执行两遍吗？2.变量提升是在编译阶段的创建变量对象过程中吗？不是预解析阶段吗？还是说创建变量对象的过程就是预解析的过程

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 对于 JavaScript 这样的解释型脚本语言来说，都需要支持编译和解析的环境来运行脚本，对于 JavaScript 来说这就是 JavaScript 引擎，而 v8 引擎便是 JavaScript 引擎的一种。
2. 变量提升发生在代码执行之前，可理解为在创建变量对象的过程中或者是创建变量之后，它们都在语法分析/AST后到执行代码之前的这个过程中进行的，这个过程也常常被称作预编译/预解析。

##### *王：
> 老师这里好像写错了:而当我们执行foo()时，也会分别进入foo函数的定义期和执行期。在foo函数的定义期时，函数bar的scope将会包含全局scope和foo的scope.第二个和第三个foo好像要改为bar

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 代码运行会先进入定义期，再进入执行期，这里描述应该是没有问题的

##### **杰：
> js权威指南写的作用域链是在定义的时候创建的，您写的是在执行期间，是否存在冲突？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是在定义期哦

##### **4391洪育煌：
> 变量和函数提升问题：上文中说函数提升优先级更高，如果变量名和函数名重复了，覆盖问题可以详细说一下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 比如 var a = 1; function a(){}
这段代码执行时，定义期中创建函数 a 并初始化；进入执行期之后，会执行赋值语句进行赋值，因此 a 的值为 1。

##### **泳：
> “因为变量提升容易带来变量在预期外被覆盖掉的问题，同时还可能导致本应该被销毁的变量没有被销毁等情况。”请问这一句话怎么理解？我知道变量对象在创建时，当函数名和变量名相同时，函数名优先级大会覆盖变量名。预期外被覆盖和本该被销毁的变量没有被销毁的情况，麻烦您能举个具体的例子吗？想象不出场景

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 变量覆盖比如：
var a = 1;
function b() {
    console.log(a);
    a = 2;
    console.log(a);
}
b();
console.log(a);

本该被销毁的变量没有被销毁比如：
function a(){ 
    for (var i = 0; i < 7; i++) { } 
    console.log(i); 
} 
a()

##### *杰：
> 根据最新的ECMA规范，变量环境只包含var定义的了， 函数声明已经归词法环境了

##### **龙：
> 老师函数名存在哪里？ class词法环境能详细讲一下吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在 ES6 中，环境记录可以分为声明式环境记录、对象环境记录和全局环境记录中，函数环境记录则是声明式环境记录的子类，而 class 也同样存在声明式环境记录中。如果感兴趣可以查阅一下 ES6 文档：https://tc39.es/ecma262/#sec-executable-code-and-execution-contexts

##### *刚：
> 在写VO到AO的例子中，写了两个AO对象，是否第一个应该是VO对象呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，为了方便理解这是同一个对象，因此都使用了 AO 来描述

