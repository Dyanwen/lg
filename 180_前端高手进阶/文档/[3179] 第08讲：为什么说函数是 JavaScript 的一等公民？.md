<p data-nodeid="1361" class=""><strong data-nodeid="1526">数据类型与函数是很多高级语言中最重要的两个概念</strong>，前者用来存储数据，后者用来存储代码。JavaScript 中的函数相对于数据类型而言更加复杂，它可以有属性，也可以被赋值给一个变量，还可以作为参数被传递......正是这些强大特性让它成了 JavaScript 的“一等公民”。</p>
<p data-nodeid="1362">下面我们就来详细了解函数的重要特性。</p>
<h3 data-nodeid="1363">this 关键字</h3>
<p data-nodeid="1364">什么是 this？this 是 JavaScript 的一个关键字，一般指向<strong data-nodeid="1534">调用它的对象</strong>。</p>
<p data-nodeid="1365">这句话其实有两层意思，首先 this 指向的应该是一个对象，更具体地说是函数执行的“上下文对象”。其次这个对象指向的是“调用它”的对象，如果调用它的不是对象或对象不存在，则会指向全局对象（严格模式下为 undefined）。</p>
<p data-nodeid="1366">下面举几个例子来进行说明。</p>
<ul data-nodeid="1367">
<li data-nodeid="1368">
<p data-nodeid="1369">当代码 1 执行 fn() 函数时，实际上就是通过对象 o 来调用的，所以 this 指向对象 o。</p>
</li>
<li data-nodeid="1370">
<p data-nodeid="1371">代码 2 也是同样的道理，通过实例 a 来调用，this 指向类实例 a。</p>
</li>
<li data-nodeid="1372">
<p data-nodeid="1373">代码 3 则可以看成是通过全局对象来调用，this 会指向全局对象（需要注意的是，严格模式下会是 undefined）。</p>
</li>
</ul>
<pre class="lang-javascript" data-nodeid="1374"><code data-language="javascript"><span class="hljs-comment">// 代码 1</span>
<span class="hljs-keyword">var</span> o = {
  fn() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
  }
}
o.fn() <span class="hljs-comment">// o</span>
<span class="hljs-comment">// 代码 2</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">A</span> </span>{
  fn() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
  }
}
<span class="hljs-keyword">var</span> a = <span class="hljs-keyword">new</span> A() 
a.fn()<span class="hljs-comment">// a</span>
<span class="hljs-comment">// 代码 3</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
}
fn() <span class="hljs-comment">// 浏览器：Window；Node.js：global</span>
</code></pre>
<p data-nodeid="1375">是不是觉得 this 的用法很简单？别着急，我们再来看看其他例子以加深理解。</p>
<p data-nodeid="1376">（1）如果在函数 fn2() 中调用函数 fn()，那么当调用函数 fn2() 的时候，函数 fn() 的 this 指向哪里呢？</p>
<pre class="lang-javascript" data-nodeid="1377"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>) </span>{<span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn2</span>(<span class="hljs-params"></span>) </span>{fn()}
fn2() <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1378"><strong data-nodeid="1546">由于没有找到调用 fn 的对象，所以 this 会指向全局对象</strong>，答案就是 window（Node.js 下是 global）。</p>
<p data-nodeid="1379">（2）再把这段代码稍稍改变一下，让函数 fn2() 作为对象 obj 的属性，通过 obj 属性来调用 fn2，此时函数 fn() 的 this 指向哪里呢？</p>
<pre class="lang-javascript" data-nodeid="1380"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>) </span>{<span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn2</span>(<span class="hljs-params"></span>) </span>{fn()}
<span class="hljs-keyword">var</span> obj = {fn2}
obj.fn2() <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1381">这里需要注意，调用函数 fn() 的是函数 fn2() 而不是 obj。虽然 fn2() 作为 obj 的属性调用，但 <strong data-nodeid="1555">fn2()中的 this 指向并不会传递给函数 fn()，</strong> 所以答案也是 window（Node.js 下是 global）。<br>
（3）对象 dx 拥有数组属性 arr，在属性 arr 的 forEach 回调函数中输出 this，指向的是什么呢？</p>
<pre class="lang-javascript" data-nodeid="1382"><code data-language="javascript"><span class="hljs-keyword">var</span> dx = {
  <span class="hljs-attr">arr</span>: [<span class="hljs-number">1</span>]
}
dx.arr.forEach(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{<span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)}) <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1383">按照之前的说法，很多同学可能会觉得输出的应该是对象 dx 的属性 arr 数组。但其实仍然是全局对象。</p>
<p data-nodeid="1384">如果你看过 forEach 的说明文档便会知道，它有两个参数，第一个是回调函数，第二个是 this 指向的对象，这里只传入了回调函数，第二个参数没有传入，默认为 undefined，所以正确答案应该是输出全局对象。</p>
<p data-nodeid="1385">类似的，需要传入 this 指向的函数还有：every()、find()、findIndex()、map()、some()，在使用的时候需要特别注意。</p>
<p data-nodeid="1386">（4）前面提到通过类实例来调用函数时，this 会指向实例。那么如果像下面的代码，创建一个 fun 变量来引用实例 b 的 fn() 函数，当调用 fun() 的时候 this 会指向什么呢？</p>
<pre class="lang-javascript" data-nodeid="1387"><code data-language="javascript"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">B</span> </span>{
  fn() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
  }
}
<span class="hljs-keyword">var</span> b = <span class="hljs-keyword">new</span> B()
<span class="hljs-keyword">var</span> fun = b.fn
fun() <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1388">这道题你可能会很容易回答出来：fun 是在全局下调用的，所以 this 应该指向的是全局对象。这个思路没有没问题，但是这里有个隐藏的知识点。那就是 ES6 下的 class 内部默认采用的是严格模式，实际上面代码的类定义部分可以理解为下面的形式。</p>
<pre class="lang-javascript" data-nodeid="1389"><code data-language="javascript"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">B</span> </span>{
<span class="hljs-meta">  'use strict'</span>;
  fn() {
    <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
  }
}
</code></pre>
<p data-nodeid="1390">而严格模式下不会指定全局对象为默认调用对象，所以答案是 undefined。</p>
<p data-nodeid="1391">（5）ES6 新加入的箭头函数不会创建自己的 this，它只会从自己的作用域链的上一层继承 this。可以简单地理解为<strong data-nodeid="1567">箭头函数的 this 继承自上层的 this</strong>，但在全局环境下定义仍会指向全局对象。</p>
<pre class="lang-javascript" data-nodeid="1392"><code data-language="javascript"><span class="hljs-keyword">var</span> arrow = {<span class="hljs-attr">fn</span>: <span class="hljs-function">() =&gt;</span> {
  <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
}}
arrow.fn() <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1393">所以虽然通过对象 arrow 来调用箭头函数 fn()，那么 this 指向不是 arrow 对象，而是全局对象。如果要让 fn() 箭头函数指向 arrow 对象，我们还需要再加一层函数，让箭头函数的上层 this 指向 arrow 对象。</p>
<pre class="lang-javascript" data-nodeid="1394"><code data-language="javascript"><span class="hljs-keyword">var</span> arrow = {
  fn() {
  &nbsp; <span class="hljs-keyword">const</span> a = <span class="hljs-function">() =&gt;</span> <span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)
&nbsp;   a()
  }
}
arrow.fn()  <span class="hljs-comment">// arrow</span>
</code></pre>
<p data-nodeid="1395">（6）前面提到 this 指向的要么是调用它的对象，要么是 undefined，那么如果将 this 指向一个基础类型的数据会发生什么呢？</p>
<p data-nodeid="1396">比如下面的代码将 this 指向数字 0，打印出的 this 是什么呢？</p>
<pre class="lang-javascript" data-nodeid="1397"><code data-language="javascript">[<span class="hljs-number">0</span>].forEach(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{<span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>)}, <span class="hljs-number">0</span>) <span class="hljs-comment">// ?</span>
</code></pre>
<p data-nodeid="1398">结合上一讲关于数据类型的知识，我们知道基础类型也可以转换成对应的引用对象。所以这里 this 指向的是一个值为 0 的 Number 类型对象。</p>
<p data-nodeid="1399">（7）改变 this 指向的常见 3 种方式有 bind、call 和 apply。call 和 apply 用法功能基本类似，都是通过传入 this 指向的对象以及参数来调用函数。区别在于传参方式，前者为逐个参数传递，后者将参数放入一个数组，以数组的形式传递。bind 有些特殊，它不但可以绑定 this 指向也可以绑定函数参数并返回一个新的函数，当 c 调用新的函数时，绑定之后的 this 或参数将无法再被改变。</p>
<pre class="lang-javascript" data-nodeid="1400"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getName</span>(<span class="hljs-params"></span>) </span>{<span class="hljs-built_in">console</span>.log(<span class="hljs-built_in">this</span>.name)}
<span class="hljs-keyword">var</span> b = getName.bind({<span class="hljs-attr">name</span>: <span class="hljs-string">'bind'</span>})
b()
getName.call({<span class="hljs-attr">name</span>: <span class="hljs-string">'call'</span>})
getName.apply({<span class="hljs-attr">name</span>: <span class="hljs-string">'apply'</span>})
</code></pre>
<p data-nodeid="1401">由于 this 指向的不确定性，所以很容易在调用时发生意想不到的情况。在编写代码时，应尽量避免使用 this，比如可以写成纯函数的形式，也可以通过参数来传递上下文对象。实在要使用 this 的话，可以考虑使用 bind 等方式将其绑定。</p>
<h3 data-nodeid="1402">补充 1：箭头函数</h3>
<p data-nodeid="1403">箭头函数和普通函数相比，有以下几个区别，在开发中应特别注意：</p>
<ul data-nodeid="1404">
<li data-nodeid="1405">
<p data-nodeid="1406">不绑定 arguments 对象，也就是说在箭头函数内访问 arguments 对象会报错；</p>
</li>
<li data-nodeid="1407">
<p data-nodeid="1408">不能用作构造器，也就是说不能通过关键字 new 来创建实例；</p>
</li>
<li data-nodeid="1409">
<p data-nodeid="1410">默认不会创建 prototype 原型属性；</p>
</li>
<li data-nodeid="1411">
<p data-nodeid="1412">不能用作 Generator() 函数，不能使用 yeild 关键字。</p>
</li>
</ul>
<h3 data-nodeid="1413">函数的转换</h3>
<p data-nodeid="1414">在讲函数转化之前，先来看一道题：编写一个 add() 函数，支持对多个参数求和以及多次调用求和。示例如下：</p>
<pre class="lang-javascript" data-nodeid="1415"><code data-language="javascript">add(<span class="hljs-number">1</span>) <span class="hljs-comment">// 1</span>
add(<span class="hljs-number">1</span>)(<span class="hljs-number">2</span>)<span class="hljs-comment">// 3</span>
add(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>)(<span class="hljs-number">3</span>, <span class="hljs-number">4</span>, <span class="hljs-number">5</span>)(<span class="hljs-number">6</span>) <span class="hljs-comment">// 21</span>
</code></pre>
<p data-nodeid="1416">对于不定参数的求和处理比较简单，很容易想到通过 arguments 或者扩展符的方式获取数组形式的参数，然后通过 reduce 累加求和。但如果直接返回结果那么后面的调用肯定会报错，所以每次返回的必须是函数，才能保证可以连续调用。也就是说 add 返回值既是一个可调用的函数又是求和的数值结果。</p>
<p data-nodeid="1417">要实现这个要求，我们必须知道函数相关的两个隐式转换函数 toString() 和 valueOf()。toString() 函数会在打印函数的时候调用，比如 console.log、valueOf 会在获取函数原始值时调用，比如加法操作。</p>
<p data-nodeid="1418">具体代码实现如下，在 add() 函数内部定义一个 fn() 函数并返回。fn() 函数的主要职能就是拼接参数并返回自身，当调用 toString() 和 valueOf() 函数时对拼接好的参数进行累加求和并返回。</p>
<pre class="lang-javascript" data-nodeid="3063"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">add</span>(<span class="hljs-params">...args</span>) </span>{
&nbsp; <span class="hljs-keyword">let</span> arr = args
&nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params">...newArgs</span>) </span>{
&nbsp; &nbsp; arr = [...arr, ...newArgs]
&nbsp; &nbsp; <span class="hljs-keyword">return</span> fn;
&nbsp; }
&nbsp; fn.toString = fn.valueOf = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> arr.reduce(<span class="hljs-function">(<span class="hljs-params">acc, cur</span>) =&gt;</span> acc + <span class="hljs-built_in">parseInt</span>(cur))
&nbsp; }
&nbsp; <span class="hljs-keyword">return</span> fn
}
</code></pre>



<h3 data-nodeid="1420">原型</h3>
<p data-nodeid="1421">原型是 JavaScript 的重要特性之一，可以让对象从其他对象继承功能特性，所以 JavaScript 也被称为“<strong data-nodeid="1591">基于原型的语言</strong>”。</p>
<p data-nodeid="1422">严格地说，原型应该是对象的特性，但函数其实也是一种特殊的对象。例如，我们对自定义的函数进行 instanceof Object 操作时，其结果是 true。</p>
<pre class="lang-javascript" data-nodeid="1423"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>)</span>{} 
fn <span class="hljs-keyword">instanceof</span> <span class="hljs-built_in">Object</span> <span class="hljs-comment">// true</span>
</code></pre>
<p data-nodeid="1424">而且我们为了实现类的特性，更多的是在函数中使用它，所以在函数这一课时中来深入讲解原型。</p>
<h4 data-nodeid="1425">什么是原型和原型链？</h4>
<p data-nodeid="1426">简单地理解，原型就是对象的属性，包括<strong data-nodeid="1605">被称为隐式原型的 <strong data-nodeid="1604">proto</strong> 属性和被称为显式原型的 prototype 属性</strong>。</p>
<p data-nodeid="1427">隐式原型通常在创建实例的时候就会自动指向构造函数的显式原型。例如，在下面的示例代码中，当创建对象 a 时，a 的隐式原型会指向构造函数 Object() 的显式原型。</p>
<pre class="lang-javascript" data-nodeid="1428"><code data-language="javascript"><span class="hljs-keyword">var</span> a = {}
a.__proto__ === <span class="hljs-built_in">Object</span>.prototype <span class="hljs-comment">// true</span>
<span class="hljs-keyword">var</span> b= <span class="hljs-keyword">new</span> <span class="hljs-built_in">Object</span>()
b.__proto__ === a.__proto__ <span class="hljs-comment">// true</span>
</code></pre>
<p data-nodeid="1429">显式原型是内置函数（比如 Date() 函数）的默认属性，在自定义函数时（箭头函数除外）也会默认生成，生成的显式原型对象只有一个属性 constructor ，该属性指向函数自身。通常配合 new 关键字一起使用，当通过 new 关键字创建函数实例时，会将实例的隐式原型指向构造函数的显式原型。</p>
<pre class="lang-javascript" data-nodeid="1430"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>) </span>{} 
fn.prototype.constructor === fn <span class="hljs-comment">// true</span>
</code></pre>
<p data-nodeid="1431">看到这里，不少同学可能会产生一种错觉，那就是隐式原型必须和显式原型配合使用，这种想法是错误的。</p>
<p data-nodeid="1432">下面的代码声明了 parent 和 child 两个对象，其中对象 child 定义了属性 name 和隐式原型 <strong data-nodeid="1614">proto</strong>，隐式原型指向对象 parent，对象 parent 定义了 code 和 name 两个属性。</p>
<p data-nodeid="1433">当打印 child.name 的时候会输出对象 child 的 name 属性值，当打印 child.code 时由于对象 child 没有属性 code，所以会找到原型对象 parent 的属性 code，将 parent.code 的值打印出来。同时可以通过打印结果看到，对象 parent 并没有显式原型属性。如果要区分对象 child 的属性是否继承自原型对象，可以通过 hasOwnProperty() 函数来判断。</p>
<pre class="lang-javascript" data-nodeid="1434"><code data-language="javascript"><span class="hljs-keyword">var</span> parent = {<span class="hljs-attr">code</span>:<span class="hljs-string">'p'</span>,<span class="hljs-attr">name</span>:<span class="hljs-string">'parent'</span>}
<span class="hljs-keyword">var</span> child = {<span class="hljs-attr">__proto__</span>: parent, <span class="hljs-attr">name</span>: <span class="hljs-string">'child'</span>}
<span class="hljs-built_in">console</span>.log(parent.prototype) <span class="hljs-comment">// undefined</span>
<span class="hljs-built_in">console</span>.log(child.name) <span class="hljs-comment">// "child"</span>
<span class="hljs-built_in">console</span>.log(child.code) <span class="hljs-comment">// "p"</span>
child.hasOwnProperty(<span class="hljs-string">'name'</span>) <span class="hljs-comment">// true</span>
child.hasOwnProperty(<span class="hljs-string">'code'</span>) <span class="hljs-comment">// false</span>
</code></pre>
<p data-nodeid="1435">在这个例子中，如果对象 parent 也没有属性 code，那么会继续在对象 parent 的原型对象中寻找属性 code，以此类推，逐个原型对象依次进行查找，直到找到属性 code 或原型对象没有指向时停止。</p>
<p data-nodeid="1436">这种类似递归的链式查找机制被称作“原型链”。</p>
<h4 data-nodeid="1437">new 操作符实现了什么？</h4>
<p data-nodeid="1438">前面提到显式原型对象在使用 new 关键字的时候会被自动创建。现在再来具体分析通过 new 关键字创建函数实例时到底发生了什么。</p>
<p data-nodeid="1439">下面的代码通过 new 关键字创建了一个函数 F() 的实例。</p>
<pre class="lang-javascript" data-nodeid="1440"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">F</span>(<span class="hljs-params">init</span>) </span>{}
<span class="hljs-keyword">var</span> f = <span class="hljs-keyword">new</span> F(args)
</code></pre>
<p data-nodeid="1441">其中主要包含了 3 个步骤：</p>
<ol data-nodeid="1442">
<li data-nodeid="1443">
<p data-nodeid="1444">创建一个临时的空对象，为了表述方便，我们命名为 fn，让对象 fn 的隐式原型指向函数 F 的显式原型；</p>
</li>
<li data-nodeid="1445">
<p data-nodeid="1446">执行函数 F()，将 this 指向对象 fn，并传入参数 args，得到执行结果 result；</p>
</li>
<li data-nodeid="1447">
<p data-nodeid="1448">判断上一步的执行结果 result，如果 result 为非空对象，则返回 result，否则返回 fn。</p>
</li>
</ol>
<p data-nodeid="1449">具体可以表述为下面的代码：</p>
<pre class="lang-javascript" data-nodeid="1450"><code data-language="javascript"><span class="hljs-keyword">var</span> fn = <span class="hljs-built_in">Object</span>.create(F.prototype)
<span class="hljs-keyword">var</span> obj = F.apply(fn, args)
<span class="hljs-keyword">var</span> f = obj &amp;&amp; <span class="hljs-keyword">typeof</span> obj === <span class="hljs-string">'object'</span> ? obj : fn;
</code></pre>
<h4 data-nodeid="1451">怎么通过原型链实现多层继承？</h4>
<p data-nodeid="1452">结合原型链和 new 操作符的相关知识，就可以实现多层继承特性了。下面通过一个简单的例子进行说明。</p>
<p data-nodeid="1453">假设构造函数 B() 需要继承构造函数 A()，就可以通过将函数 B() 的显式原型指向一个函数 A() 的实例，然后再对 B 的显式原型进行扩展。那么通过函数 B() 创建的实例，既能访问用函数 B() 的属性 b，也能访问函数 A() 的属性 a，从而实现了多层继承。</p>
<pre class="lang-javascript" data-nodeid="1454"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">A</span>(<span class="hljs-params"></span>) </span>{
}
A.prototype.a = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">'a'</span>;
}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">B</span>(<span class="hljs-params"></span>) </span>{
}
B.prototype = <span class="hljs-keyword">new</span> A()
B.prototype.b = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">'b'</span>;
}
<span class="hljs-keyword">var</span> c = <span class="hljs-keyword">new</span> B()
c.b() <span class="hljs-comment">// 'b'</span>
c.a() <span class="hljs-comment">// 'a'</span>
</code></pre>
<h3 data-nodeid="1455">补充 2：typeof 和 instanceof</h3>
<p data-nodeid="1456"><strong data-nodeid="1633">typeof</strong></p>
<p data-nodeid="1457">用来获取一个值的类型，可能的结果有下面几种：</p>
<table data-nodeid="1459">
<thead data-nodeid="1460">
<tr data-nodeid="1461">
<th data-org-content="类型" data-nodeid="1463">类型</th>
<th data-org-content="结果" data-nodeid="1464">结果</th>
</tr>
</thead>
<tbody data-nodeid="1467">
<tr data-nodeid="1468">
<td data-org-content="Undefined" data-nodeid="1469">Undefined</td>
<td data-org-content="&quot;undefined&quot;" data-nodeid="1470">"undefined"</td>
</tr>
<tr data-nodeid="1471">
<td data-org-content="Boolean" data-nodeid="1472">Boolean</td>
<td data-org-content="&quot;boolean&quot;" data-nodeid="1473">"boolean"</td>
</tr>
<tr data-nodeid="1474">
<td data-org-content="Number" data-nodeid="1475">Number</td>
<td data-org-content="&quot;number&quot;" data-nodeid="1476">"number"</td>
</tr>
<tr data-nodeid="1477">
<td data-org-content="BigInt" data-nodeid="1478">BigInt</td>
<td data-org-content="&quot;bigint&quot;" data-nodeid="1479">"bigint"</td>
</tr>
<tr data-nodeid="1480">
<td data-org-content="String" data-nodeid="1481">String</td>
<td data-org-content="&quot;string&quot;" data-nodeid="1482">"string"</td>
</tr>
<tr data-nodeid="1483">
<td data-org-content="Symbol" data-nodeid="1484">Symbol</td>
<td data-org-content="&quot;symbol&quot;" data-nodeid="1485">"symbol"</td>
</tr>
<tr data-nodeid="1486">
<td data-org-content="函数对象" data-nodeid="1487">函数对象</td>
<td data-org-content="&quot;function&quot;" data-nodeid="1488">"function"</td>
</tr>
<tr data-nodeid="1489">
<td data-org-content="其他对象及 null" data-nodeid="1490">其他对象及 null</td>
<td data-org-content="&quot;object&quot;" data-nodeid="1491">"object"</td>
</tr>
</tbody>
</table>
<p data-nodeid="1492"><strong data-nodeid="1672">instanceof</strong></p>
<p data-nodeid="1493">用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。例如，在表达式 left instanceof right 中，会沿着 left 的原型链查找，看看是否存在 right 的 prototype 对象。</p>
<pre class="lang-javascript" data-nodeid="1494"><code data-language="javascript">left.__proto__.__proto__... =?= right.prototype
</code></pre>
<h3 data-nodeid="1495">作用域</h3>
<p data-nodeid="1496">作用域是指赋值、取值操作的执行范围，通过作用域机制可以有效地防止变量、函数的重复定义，以及控制它们的可访问性。</p>
<p data-nodeid="1497">虽然在浏览器端和 Node.js 端作用域的处理有所不同，比如对于全局作用域，浏览器会自动将未主动声明的变量提升到全局作用域，而 Node.js 则需要显式的挂载到 global 对象上。又比如在 ES6 之前，浏览器不提供模块级别的作用域，而 Node.js 的 CommonJS 模块机制就提供了模块级别的作用域。但在类型上，可以分为全局作用域（window/global）、块级作用域（let、const、try/catch）、模块作用域（ES6 Module、CommonJS）及本课时重点讨论的函数作用域。</p>
<h4 data-nodeid="1498">命名提升</h4>
<p data-nodeid="1499">对于使用 var 关键字声明的变量以及创建命名函数的时候，JavaScript 在解释执行的时候都会将其声明内容提升到作用域顶部，这种机制称为“<strong data-nodeid="1683">命名提升</strong>”。</p>
<p data-nodeid="1500">变量的命名提升允许我们在同（子）级作用域中，在变量声明之前进行引用，但要注意，得到的是未赋值的变量。而且仅限 var 关键字声明的变量，对于 let 和 const 在定义之前引用会报错。</p>
<pre class="lang-javascript" data-nodeid="1501"><code data-language="javascript"><span class="hljs-built_in">console</span>.log(a) <span class="hljs-comment">// undefined</span>
<span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>
<span class="hljs-built_in">console</span>.log(b) <span class="hljs-comment">// 报错</span>
<span class="hljs-keyword">let</span> b = <span class="hljs-number">2</span>
</code></pre>
<p data-nodeid="1502">函数的命名提升则意味着可以在同级作用域或者子级作用域里，在函数定义之前进行调用。</p>
<pre class="lang-javascript" data-nodeid="1503"><code data-language="javascript">fn() <span class="hljs-comment">// 2</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">return</span> <span class="hljs-number">2</span>
}
</code></pre>
<p data-nodeid="1504">结合以上两点我们再来看看下面两种函数定义的区别，方式 1 将函数赋值给变量 f；方式 2 定义了一个函数 f()。</p>
<pre class="lang-javascript" data-nodeid="1505"><code data-language="javascript"><span class="hljs-comment">// 方式1</span>
<span class="hljs-keyword">var</span> f = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{...}
<span class="hljs-comment">// 方式2</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">f</span>(<span class="hljs-params"></span>) </span>{...}
</code></pre>
<p data-nodeid="1506">两种方式对于调用函数方式以及返回结果而言是没有区别的，但根据命名提升的规则，我们可以得知方式 1 创建了一个匿名函数，让变量 f 指向它，这里会发生变量的命名提升；如果我们在定义函数之前调用会报错，而方式 2 则不会。</p>
<h4 data-nodeid="1507">闭包</h4>
<p data-nodeid="1508">在函数内部访问外部函数作用域时就会产生闭包。闭包很有用，因为它允许将函数与其所操作的某些数据（环境）关联起来。这种关联不只是跨作用域引用，也可以实现数据与函数的隔离。</p>
<p data-nodeid="1509">比如下面的代码就通过闭包来实现单例模式。</p>
<pre class="lang-javascript" data-nodeid="1510"><code data-language="javascript"><span class="hljs-keyword">var</span> SingleStudent = (<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{&nbsp;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">Student</span>(<span class="hljs-params"></span>) </span>{}
&nbsp; &nbsp; <span class="hljs-keyword">var</span> _student;&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (_student) <span class="hljs-keyword">return</span> _student;
&nbsp; &nbsp; &nbsp; &nbsp; _student = <span class="hljs-keyword">new</span> Student()
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> _student;
&nbsp; &nbsp; }
}())
<span class="hljs-keyword">var</span> s = <span class="hljs-keyword">new</span> SingleStudent()
<span class="hljs-keyword">var</span> s2 = <span class="hljs-keyword">new</span> SingleStudent()
s === s2 <span class="hljs-comment">// true</span>
</code></pre>
<p data-nodeid="1511">函数 SingleStudent 内部通过闭包创建了一个私有变量 _student，这个变量只能通过返回的匿名函数来访问，匿名函数在返回变量时对其进行判断，如果存在则直接返回，不存在则在创建保存后返回。</p>
<h3 data-nodeid="1512">补充 3：经典笔试题</h3>
<pre class="lang-javascript" data-nodeid="1513"><code data-language="javascript"><span class="hljs-keyword">for</span>( <span class="hljs-keyword">var</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">5</span>; i++ ) {
	<span class="hljs-built_in">setTimeout</span>(<span class="hljs-function">() =&gt;</span> {
		<span class="hljs-built_in">console</span>.log( i );
	}, <span class="hljs-number">1000</span> * i)
}
</code></pre>
<p data-nodeid="1514">这是一道作用域相关的经典笔试题，需要实现的功能是每隔 1 秒控制台打印数字 0 到 4。但实际执行效果是每隔一秒打印的数字都是 5，为什么会这样呢？</p>
<p data-nodeid="1515">如果把这段代码转换一下，手动对变量 i 进行命名提升，你就会发现 for 循环和打印函数共享了同一个变量 i，这就是问题所在。</p>
<pre class="lang-javascript" data-nodeid="1516"><code data-language="javascript"><span class="hljs-keyword">var</span> i;
<span class="hljs-keyword">for</span>(i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">5</span>; i++ ) {
	<span class="hljs-built_in">setTimeout</span>(<span class="hljs-function">() =&gt;</span> {
		<span class="hljs-built_in">console</span>.log(i);
	}, <span class="hljs-number">1000</span> * i)
}
</code></pre>
<p data-nodeid="1517">要修复这段代码方法也有很多，比如将 var 关键字替换成 let，从而创建块级作用域。</p>
<pre class="lang-javascript" data-nodeid="1518"><code data-language="javascript"><span class="hljs-keyword">for</span>(<span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">5</span>; i++ ) {
	<span class="hljs-built_in">setTimeout</span>(<span class="hljs-function">() =&gt;</span> {
		<span class="hljs-built_in">console</span>.log(i);
	}, <span class="hljs-number">1000</span> * i)
}
<span class="hljs-comment">/**
等价于
for(var i = 0; i &lt; 5; i++ ) {
    let _i = i
	setTimeout(() =&gt; {
		console.log(_i);
	}, 1000 * i)
}
 */</span>
</code></pre>
<h3 data-nodeid="1519">总结</h3>
<p data-nodeid="1520">本课时介绍了函数相关的重要内容，包括 this 关键字的指向、原型与原型链的使用、函数的隐式转换、函数和作用域的关系，希望大家能理解并记忆。</p>
<p data-nodeid="1521" class="">最后布置一道思考题：结合本课时的内容，思考一下修改函数的 this 指向，到底有多少种方式呢？</p>

---

### 精选评论

##### Warn：
> this指向、原型链这些真的是概念一看都懂，看完又啥都不懂

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 哈哈哈哈哈哈，加油

##### *忠：
> 不存在隐式原型，显式原型吧，__proto__ 本身就不是ECMA的标准，是浏览器自己实现的。

##### *道：
> 根据 MDN 文档">someObject.Prototype ，property __proto__ 是非标准而被浏览器支持的属性。

##### *道：
> 这一节看得稍辛苦，补了一下基础。

##### *聪：
> 鄙人写的关于原型链相关的一篇博文：https://chen-cong.blog.csdn.net/article/details/81211729，相信你看了一定会有收获

##### **飞：
> 不太理解，显示原型，和隐式原型这两个概念😔😇

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以简单地理解为显式原型是隐式原型的别名。

##### *默：
> 绑定this，有bind，apply，call。还有箭头函数

##### **用户5887：
> 已知this指向调用它的对象，函数是特殊的对象，为什么全局函数b中调用函数a，指向的还是全局呢？函数自身为什么不能作为对象，有上下文呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题我想了很久，下面是个人思考，并不一定是正确答案。从js工作原理上来看，如果每次调用函数都去祖先函数去查找上下文，你想想函数调用栈的入栈出栈会有多么频繁，而这种性能损耗是完全可以避免的，因为开发者可以创建一个变量保存状态；从执行结果上来看，也会带来很大麻烦，函数的执行结果就变成不可预期，因为放到不同函数中继承的的上下文不同。

##### **5741：
> 所以看到最后还是没看出来题目“为什么说函数是 JavaScript 的一等公民？”这个问题的官方解答

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; "一等公民"可以理解为有很多权利，对应到语言JavaScript就是说可以实现很多功能，比如课程中提到的模拟类的实现、创建作用域。

##### ty：
> 其实我不太理解隐式原型存在的意义是什么，如果本来就可以通过prototype追溯到原型上去那多增加一个__proto__其实也用不到吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 严格意义上来说，只有一个原型，那就是隐式原型 __proto__，显示原型 prototype 你可以理解为 __proto__ 的语法糖，它的作用就是在创建实例的时候把实例的 __proto_ 指向当前的 prototype。

##### *鸿：
> 修改this指向 bind ，call，apply&nbsp; &nbsp;bind在绑定this函数后不会立即执行，call和apply会，还有闭包不应该是外部访问内部变量吗

##### **用户1028：
> var arrow = {fn: () =&gt; console.log(this)} 箭头函数改成function写法就从this指向window改成arrow对象呀, 没理解包一层函数 var arrow = {fn(){ console.log(this)}}

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 函数 a 继承了函数 fn 的this，指向 arrow

