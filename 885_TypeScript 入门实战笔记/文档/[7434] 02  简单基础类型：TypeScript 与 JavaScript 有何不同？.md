<p data-nodeid="38120">这一讲，我们将从最基本的语法、原始类型层面的知识点正式开启 TypeScript 学习之旅。“不积跬步，无以至千里”，只有融会贯通、夯实基础，我们才能在后续的学习中厚积薄发。</p>
<blockquote data-nodeid="39530">
<p data-nodeid="39531" class=""><strong data-nodeid="39537">学习建议：</strong> 为了更直观地学习这一讲的内容，请你使用配置好的 VS Code IDE（可以回顾一下“01 | 如何快速搭建 TypeScript 开发环境？”的内容） 亲自尝试编写以下涉及的所有示例，比如新建一个“02.basic.1.ts”。</p>
</blockquote>
<h3 data-nodeid="39532">TypeScript 简介</h3>


<p data-nodeid="38124">TypeScript 其实就是类型化的 JavaScript，它不仅支持 JavaScript 的所有特性，还在 JavaScript 的基础上添加了静态类型注解扩展。</p>
<p data-nodeid="38125">这里我们举个例子来说明一下，比如 JavaScript 中虽然提供了原始数据类型 string、number，但是它无法检测我们是不是按照约定的类型对变量赋值，而 TypeScript 会对赋值及其他所有操作默认做静态类型检测。</p>
<p data-nodeid="38126">因此，从某种意义上来说，<strong data-nodeid="38239">TypeScript 其实就是 JavaScript 的超集</strong>，如下图所示：</p>
<p data-nodeid="41229"><img src="https://s0.lgstatic.com/i/image6/M00/3D/B5/CioPOWCV_xuAZSI_AAdZCdHFgM8072.png" alt="Drawing 1.png" data-nodeid="41233"></p>
<div data-nodeid="41230" class=""><p style="text-align:center">TypeScript 是 JavaScript 的超集示意图</p></div>




<p data-nodeid="38130">在 TypeScript 中，我们不仅可以轻易复用 JavaScript 的代码、最新特性，还能使用可选的静态类型进行检查报错，使得编写的代码更健壮、更易于维护。比如在开发阶段，我们通过 TypeScript 代码转译器就能快速消除很多低级错误（如 typo、类型等）。</p>
<p data-nodeid="38131">接下来我们一起看看 TypeScript 的基本语法。</p>
<h3 data-nodeid="38132">基本语法</h3>
<p data-nodeid="38133">在语法层面，缺省类型注解的 TypeScript 与 JavaScript 完全一致。因此，我们可以把 TypeScript 代码的编写看作是为 JavaScript 代码添加类型注解。</p>
<p data-nodeid="38134">在 TypeScript 语法中，类型的标注主要通过类型后置语法来实现，下面我们通过一个具体示例进行说明。</p>
<pre class="lang-typescript" data-nodeid="38135"><code data-language="typescript"><span class="hljs-keyword">let</span> num = <span class="hljs-number">1</span>;
</code></pre>
<p data-nodeid="38136">示例中的语法同时符合 JavaScript 语法和 TypeScript 语法。</p>
<p data-nodeid="38137">而 TypeScript 语法与 JavaScript 语法的区别在于，我们可以在 TypeScript 中显式声明变量<code data-backticks="1" data-nodeid="38254">num</code>仅仅是数字类型，也就是说只需在变量<code data-backticks="1" data-nodeid="38256">num</code>后添加<code data-backticks="1" data-nodeid="38258">: number</code>类型注解即可，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="38138"><code data-language="typescript"><span class="hljs-keyword">let</span> num: <span class="hljs-built_in">number</span> = <span class="hljs-number">1</span>;
</code></pre>
<p data-nodeid="38139"><strong data-nodeid="38270">特殊说明：</strong><code data-backticks="1" data-nodeid="38263">number</code>表示数字类型，<code data-backticks="1" data-nodeid="38265">:</code>用<strong data-nodeid="38271">来分割变量和类型的分隔符。</strong></p>
<p data-nodeid="38140">同理，我们也可以把<code data-backticks="1" data-nodeid="38273">:</code>后的<code data-backticks="1" data-nodeid="38275">number</code>换成其他的类型（比如 JavaScript 原始类型：number、string、boolean、null、undefined、symbol 等），此时，num 变量也就拥有了 TypeScript 同名的原始类型定义。</p>
<p data-nodeid="38141">关于 JavaScript 原始数据类型到 TypeScript 类型的映射关系如下表所示：</p>
<p data-nodeid="41788" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3D/B5/CioPOWCV_y2AfRkCAAJ0QW8Nr1k253.png" alt="Drawing 2.png" data-nodeid="41791"></p>


<p data-nodeid="38173">接下来，我们详细地了解一下原始类型。</p>
<h3 data-nodeid="38174">原始类型</h3>
<p data-nodeid="42246" class="">在 JavaScript 中，原始类型指的是非对象且没有方法的数据类型，它包括 string、number、bigint、boolean、undefined 和 symbol 这六种 <strong data-nodeid="42252">（null 是一个伪原始类型，它在 JavaScript 中实际上是一个对象，且所有的结构化类型都是通过 null 原型链派生而来）</strong>。</p>

<p data-nodeid="38176">在 JavaScript 语言中，原始类型值是最底层的实现，对应到 TypeScript 中同样也是最底层的类型。</p>
<p data-nodeid="38177">为了实现更合理的逻辑边界，本专栏我们把以上原始类型拆分为基础类型和特殊类型这两部分进行讲解。02 讲主要讲解字符串、数字（包括 number 和 bigint）、布尔值、Symbol 这 4 种基础类型，03 讲主要讲解 null 和 undefined 等特殊字符。</p>
<h4 data-nodeid="38178">字符串</h4>
<p data-nodeid="38179">在 JavaScript 中，我们可以使用<code data-backticks="1" data-nodeid="38312">string</code>表示 JavaScript 中任意的字符串（包括模板字符串），具体示例如下所示：</p>
<pre class="lang-typescript" data-nodeid="38180"><code data-language="typescript"><span class="hljs-keyword">let</span> firstname: <span class="hljs-built_in">string</span> = <span class="hljs-string">'Captain'</span>; <span class="hljs-comment">// 字符串字面量</span>
<span class="hljs-keyword">let</span> familyname: <span class="hljs-built_in">string</span> = <span class="hljs-built_in">String</span>(<span class="hljs-string">'S'</span>); <span class="hljs-comment">// 显式类型转换</span>
<span class="hljs-keyword">let</span> fullname: <span class="hljs-built_in">string</span> = <span class="hljs-string">`my name is <span class="hljs-subst">${firstname}</span>.<span class="hljs-subst">${familyname}</span>`</span>;&nbsp;<span class="hljs-comment">// 模板字符串</span>
</code></pre>
<blockquote data-nodeid="38181">
<p data-nodeid="38182"><strong data-nodeid="38317">说明：所有 JavaScript 支持的定义字符串的方法，我们都可以直接在 TypeScript 中使用。</strong></p>
</blockquote>
<h4 data-nodeid="38183">数字</h4>
<p data-nodeid="38184">同样，我们可以使用<code data-backticks="1" data-nodeid="38320">number</code>类型表示 JavaScript 已经支持或者即将支持的十进制整数、浮点数，以及二进制数、八进制数、十六进制数，具体的示例如下所示：</p>
<pre class="lang-typescript" data-nodeid="38185"><code data-language="typescript"><span class="hljs-comment">/** 十进制整数 */</span>
<span class="hljs-keyword">let</span> integer: <span class="hljs-built_in">number</span> = <span class="hljs-number">6</span>;
<span class="hljs-comment">/** 十进制整数 */</span>
<span class="hljs-keyword">let</span> integer2: <span class="hljs-built_in">number</span> = <span class="hljs-built_in">Number</span>(<span class="hljs-number">42</span>);
<span class="hljs-comment">/** 十进制浮点数 */</span>
<span class="hljs-keyword">let</span> decimal: <span class="hljs-built_in">number</span> = <span class="hljs-number">3.14</span>;
<span class="hljs-comment">/** 二进制整数 */</span>
<span class="hljs-keyword">let</span> binary: <span class="hljs-built_in">number</span> = <span class="hljs-number">0b1010</span>;
<span class="hljs-comment">/** 八进制整数 */</span>
<span class="hljs-keyword">let</span> octal: <span class="hljs-built_in">number</span> = <span class="hljs-number">0o744</span>;
<span class="hljs-comment">/** 十六进制整数 */</span>
<span class="hljs-keyword">let</span> hex: <span class="hljs-built_in">number</span> = <span class="hljs-number">0xf00d</span>;
</code></pre>
<p data-nodeid="38186">如果使用较少的大整数，那么我们可以使用<code data-backticks="1" data-nodeid="38323">bigint</code>类型来表示，如下代码所示。</p>
<pre class="lang-typescript" data-nodeid="38187"><code data-language="typescript"><span class="hljs-keyword">let</span> big: bigint =  <span class="hljs-number">100n</span>;
</code></pre>
<p data-nodeid="38188"><strong data-nodeid="38332">请注意：虽然</strong><code data-backticks="1" data-nodeid="38328">number</code>和<code data-backticks="1" data-nodeid="38330">bigint</code>都表示数字，但是这两个类型不兼容。</p>
<p data-nodeid="38189">因此，如果我们在 VS Code IDE 中输入如下示例，问题面板中将会抛出一个类型不兼容的  ts(2322)  错误。</p>
<pre class="lang-typescript" data-nodeid="38190"><code data-language="typescript">big = integer;
integer = big;
</code></pre>
<h4 data-nodeid="38191">布尔值</h4>
<p data-nodeid="38192">同样，我们可以使用<code data-backticks="1" data-nodeid="38336">boolean</code>表示 True 或者 False，如下代码所示。</p>
<pre class="lang-typescript" data-nodeid="38193"><code data-language="typescript"><span class="hljs-comment">/** TypeScript 真香 为 真 */</span>
<span class="hljs-keyword">let</span> TypeScriptIsGreat: <span class="hljs-built_in">boolean</span> = <span class="hljs-literal">true</span>;
 <span class="hljs-comment">/** TypeScript 太糟糕了 为 否 */</span>
<span class="hljs-keyword">let</span> TypeScriptIsBad: <span class="hljs-built_in">boolean</span> = <span class="hljs-literal">false</span>;
</code></pre>
<h4 data-nodeid="38194">Symbol</h4>
<p data-nodeid="38195">自 ECMAScript 6 起，TypeScript 开始支持新的<code data-backticks="1" data-nodeid="38340">Symbol</code>原始类型， 即我们可以通过<code data-backticks="1" data-nodeid="38342">Symbol</code>构造函数，创建一个独一无二的标记；同时，还可以使用<code data-backticks="1" data-nodeid="38344">symbol</code>表示如下代码所示的类型。</p>
<pre class="lang-typescript" data-nodeid="38196"><code data-language="typescript"><span class="hljs-keyword">let</span> sym1: symbol = Symbol();
<span class="hljs-keyword">let</span> sym2: symbol = Symbol(<span class="hljs-string">'42'</span>);
</code></pre>
<p data-nodeid="38197"><strong data-nodeid="38349">当然，TypeScript 还包含 Number、String、Boolean、Symbol 等类型（注意区分大小写）。</strong></p>
<blockquote data-nodeid="38198">
<p data-nodeid="38199"><strong data-nodeid="38354">特殊说明：请你千万别将它们和小写格式对应的 number、string、boolean、symbol 进行等价</strong>。不信的话，你可以思考并验证如下所示的示例。</p>
</blockquote>
<pre class="lang-typescript" data-nodeid="38200"><code data-language="typescript"><span class="hljs-keyword">let</span> sym: symbol = Symbol(<span class="hljs-string">'a'</span>);
<span class="hljs-keyword">let</span> sym2: Symbol = Symbol(<span class="hljs-string">'b'</span>);
sym = sym2 <span class="hljs-comment">// ok or fail?</span>
sym2 = sym <span class="hljs-comment">// ok or fail?</span>
<span class="hljs-keyword">let</span> str: <span class="hljs-built_in">String</span> = <span class="hljs-keyword">new</span> <span class="hljs-built_in">String</span>(<span class="hljs-string">'a'</span>);
<span class="hljs-keyword">let</span> str2: <span class="hljs-built_in">string</span> = <span class="hljs-string">'a'</span>;
str = str2; <span class="hljs-comment">// ok or fail?</span>
str2 = str; <span class="hljs-comment">// ok or fail?</span>
</code></pre>
<p data-nodeid="38201">实际上，我们压根使用不到 Number、String、Boolean、Symbol 类型，因为它们并没有什么特殊的用途。这就像我们不必使用 JavaScript Number、String、Boolean 等构造函数 new 一个相应的实例一样。</p>
<p data-nodeid="38202">介绍完这几种原始类型后，你可能会心生疑问：缺省类型注解的有无似乎没有什么明显的作用，就像如下所示的示例一样：</p>
<pre class="lang-typescript" data-nodeid="38203"><code data-language="typescript">{
  <span class="hljs-keyword">let</span> mustBeNum = <span class="hljs-number">1</span>;
}
{
  <span class="hljs-keyword">let</span> mustBeNum: <span class="hljs-built_in">number</span> = <span class="hljs-number">1</span>;
}
</code></pre>
<p data-nodeid="38204">其实，以上这两种写法在 TypeScript 中是等价的，这得益于基于上下文的类型推导（这部分内容我们将在 04 讲中详细说明）。</p>
<p data-nodeid="38205">下面，我们对上面的示例稍做一下修改，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="38206"><code data-language="typescript">{
  <span class="hljs-keyword">let</span> mustBeNum = <span class="hljs-string">'badString'</span>;
}
{
  <span class="hljs-keyword">let</span> mustBeNum: <span class="hljs-built_in">number</span> = <span class="hljs-string">'badString'</span>;
}
</code></pre>
<p data-nodeid="38207">此时，我们可以看到 VS Code 的内容和问题面板区域提示了错误（其他 IDE 也会出现类似提示），如下图所示：</p>
<p data-nodeid="43164" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3D/AC/Cgp9HWCV_0GAYFH7AATQLps4G-c499.png" alt="Drawing 4.png" data-nodeid="43168"></p>
<div data-nodeid="43165"><p style="text-align:center">错误提示效果图</p></div>



<p data-nodeid="43623">以上就是类型注解作用的直观体现。</p>


<p data-nodeid="38212">如果变量所处的上下文环境特别复杂，在开发阶段就能检测出低级类型错误的能力将显得尤为重要，而这种能力主要来源于 TypeScript 实现的静态类型检测。</p>
<h3 data-nodeid="38213">静态类型检测</h3>
<p data-nodeid="38214">在编译时期，静态类型的编程语言即可准确地发现类型错误，这就是静态类型检测的优势。</p>
<p data-nodeid="38215">在编译（转译）时期，TypeScript 编译器将通过对比检测变量接收值的类型与我们显示注解的类型，从而检测类型是否存在错误。如果两个类型完全一致，显示检测通过；如果两个类型不一致，它就会抛出一个编译期错误，告知我们编码错误，具体示例如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="38216"><code data-language="typescript"><span class="hljs-keyword">const</span> trueNum: <span class="hljs-built_in">number</span> = <span class="hljs-number">42</span>;
<span class="hljs-keyword">const</span> fakeNum: <span class="hljs-built_in">number</span> = <span class="hljs-string">"42"</span>; <span class="hljs-comment">// ts(2322) Type 'string' is not assignable to type 'number'.</span>
</code></pre>
<p data-nodeid="38217">在以上示例中，首先我们声明了一个数字类型的变量<code data-backticks="1" data-nodeid="38373">trueNum</code>，通过编译器检测后，发现接收值是 42，且它的类型是<code data-backticks="1" data-nodeid="38375">number</code>，可见两者类型完全一致。此时，TypeScript 编译器就会显示检测通过。</p>
<p data-nodeid="38218">而如果我们声明了一个<code data-backticks="1" data-nodeid="38378">string</code>类型的变量<code data-backticks="1" data-nodeid="38380">fakeNum</code>，通过编译器检测后，发现接收值为 "42"，且它的类型是<code data-backticks="1" data-nodeid="38386">number</code>，可见两者类型不一致 。此时，TypeScript 编译器就会抛出一个字符串值不能为数字类型变量赋值的ts(2322)  错误，也就是说检测不通过。</p>
<p data-nodeid="38219">实际上，正如 01 讲提到，TypeScript 的语言服务可以和 VS Code 完美集成。<strong data-nodeid="38392">因此，在编写代码的同时，我们可以同步进行静态类型检测（无须等到编译后再做检测），极大地提升了开发体验和效率。</strong></p>
<p data-nodeid="38220">以上就是 TypeScript 中基本语法和原始类型的介绍。</p>
<h3 data-nodeid="38221">小结</h3>
<p data-nodeid="38222" class="">这一讲通过与 JavaScript 的基础类型进行对比，我们得知：TypeScript 其实就是添加了类型注解的 JavaScript，它并没有任何颠覆性的变动。因此，学习并掌握 TypeScript 一定会是一件极其容易的事情。</p>
<p data-nodeid="38223"><strong data-nodeid="38399">插播一个思考题：请举例说明 ts(2322)  是一个什么错误？什么时候会抛出这个错误？欢迎你在留言区进行互动、交流。</strong></p>
<p data-nodeid="38224">另外，如果你觉得本专栏有价值，欢迎分享给更多好友哦~</p>

---

### 精选评论

##### **强：
> TypeScript 其实就是添加了类型注解的 JavaScript，它并没有任何颠覆性的变动。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 事实上除了常量枚举（const enums），TypeScript 就是在 JavaScript 语法基础上添加了类型注解，所以大家无需担心在学习 TypeScript 上会有过重的心智负担；

##### **凤：
> ts(2322)是一个静态类型检查的错误，在注解的类型和赋值的类型不同的时候就会抛出这个错误。。。😳

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 手动点赞！

##### **蚪找麻麻：
> 不同点还挺多的，记下来，好好学习

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油

##### **怀：
> 虽然typeof null 的结果是object，但是null并不是对象，这是js语言的一个历史bug啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 赞，大佬！如果要从 JavaScript 底层去看，确实不能说 null 是空对象，而是说空对象指针；从应用层面上讲，说是空对象也是没太大问题（在JavaScript最初的实现中，JavaScript中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于null 代表的是空指针(大多数平台下值为0x00)，因此，null的类型标签是0,  typeof null也因此返回"object"。）。

##### **勿扰：
> 请教老师一个问题type IReactComponent=| React. FC| React. Component class| React. Classic component class这种用法是什么意思，如果这是联合类型，为什么第一个竖线前面没有类型

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 仅仅是为了书写格式好看，没有任何特殊的意义。

##### **俊：
> ts(2322) 类型不兼容，声明的类型和值类型不一致

##### *舒：
> 不知道会不会有 ts 4 版本的内容

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在课程的最后一节会详细讲解 TypeScript 4.* 以后的模板字面量类型、元组相关特性、映射类型键名重映射、迭代器类型变动等重点知识；至于前面章节所涉及的知识点，无论是 3.* 还是 4.*，都是通用且不会过时（因为没有 break changes）。

##### *一：
> 各种类型的声明真的有点麻烦，文档也没有规律遵循

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我们可以善用 TypeScript 类型类型推断（包括基于上下文类型推断）特性，可以推断得出的类型就无需显式编写。

##### **睢阳小天使：
> 啊我原来一直以为TS和JS差不多，大意啦！

