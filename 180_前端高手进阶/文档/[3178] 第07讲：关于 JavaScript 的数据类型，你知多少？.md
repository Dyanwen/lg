<p>数据类型通常是一门编程语言的基础知识，JavaScript 的数据类型可以分为 7 种：空（Null）、未定义（Undefined）、数字（Number）、字符串（String）、布尔值（Boolean）、符号（Symbol）、对象（Object）。</p>
<p>其中前 6 种类型为<strong>基础类型</strong>，最后 1 种为<strong>引用类型</strong>。这两者的区别在于，基础类型的数据在被引用或拷贝时，是值传递，也就是说会创建一个完全相等的变量；而引用类型只是创建一个指针指向原有的变量，实际上两个变量是“共享”这个数据的，并没有重新创建一个新的数据。</p>
<p>下面我们就来分别介绍这 7 种数据类型的重要概念及常见操作。</p>
<h3>Undefined</h3>
<p>Undefined 是一个很特殊的数据类型，它只有一个值，也就是 undefined。可以通过下面几种方式来得到 undefined：</p>
<ul>
<li>引用已声明但未初始化的变量；</li>
<li>引用未定义的对象属性；</li>
<li>执行无返回值函数；</li>
<li>执行 void 表达式；</li>
<li>全局常量 window.undefined 或 undefined。</li>
</ul>
<p>对应代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">var</span> a; <span class="hljs-comment">// undefined</span>
<span class="hljs-keyword">var</span> o = {}
o.b <span class="hljs-comment">// undefined</span>
(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {})() <span class="hljs-comment">// undefined</span>
<span class="hljs-keyword">void</span> <span class="hljs-number">0</span> <span class="hljs-comment">// undefined</span>
<span class="hljs-built_in">window</span>.undefined <span class="hljs-comment">// undefined</span>
</code></pre>
<p>其中比较推荐通过 void 表达式来得到 undefined 值，因为这种方式既简便（window.undefined 或 undefined 常量的字符长度都大于 "void 0" 表达式）又不需要引用额外的变量和属性；同时它作为表达式还可以配合三目运算符使用，代表不执行任何操作。</p>
<p>如下面的代码就表示满足条件 x 大于 0 且小于 5 的时候执行函数 fn，否则不进行任何操作：</p>
<pre><code data-language="javascript" class="lang-javascript">x&gt;<span class="hljs-number">0</span> &amp;&amp; x&lt;<span class="hljs-number">5</span> ? fn() : <span class="hljs-keyword">void</span> <span class="hljs-number">0</span>;
</code></pre>
<p>如何判断一个变量的值是否为 undefined 呢？<br>
下面的代码给出了 3 种方式来判断变量 x 是否为 undefined，你可以先思考一下哪一种可行。</p>
<p>方式 1 直接通过<strong>逻辑取非</strong>操作来将变量 x 强制转换为布尔值进行判断；方式 2 通过 3 个等号将变量 x 与 undefined 做<strong>真值比较</strong>；方式 3 通过 typeof 关键字获取变量 x 的类型，然后与 'undefined' 字符串做<strong>真值比较：</strong></p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// 方式1</span>
<span class="hljs-keyword">if</span>(!x) {
  ...
}
<span class="hljs-comment">// 方式2</span>
<span class="hljs-keyword">if</span>(x===<span class="hljs-literal">undefined</span>) {
  ...
}
<span class="hljs-comment">// 方式2</span>
<span class="hljs-keyword">if</span>(<span class="hljs-keyword">typeof</span> x === <span class="hljs-string">'undefined'</span>) {
  ...
}
</code></pre>
<p>现在来揭晓答案，方式 1 不可行，因为只要变量 x 的值为 undefined、空字符串、数值 0、null 时都会判断为真。方式 2 也存在一些问题，虽然通过 “===” 和 undefined 值做比较是可行的，但如果 x 未定义则会抛出错误 “ReferenceError: x is not defined” 导致程序执行终止，这对于代码的健壮性显然是不利的。方式 3 则解决了这一问题。</p>
<h3>Null</h3>
<p>Null 数据类型和 Undefined 类似，只有唯一的一个值 null，都可以表示空值，甚至我们通过 “==” 来比较它们是否相等的时候得到的结果都是 true，但 null 是 JavaScript 保留关键字，而 undefined 只是一个常量。</p>
<p>也就是说我们可以声明名称为 undefined 的变量（虽然只能在老版本的 IE 浏览器中给它重新赋值），但将 null 作为变量使用时则会报错。</p>
<h3>Boolean</h3>
<p>Boolean 数据类型只有两个值：true 和 false，分别代表真和假，理解和使用起来并不复杂。但是我们常常会将各种表达式和变量转换成 Boolean 数据类型来当作判断条件，这时候就要注意了。</p>
<p>下面是一个简单地将星期数转换成中文的函数，比如输入数字 1，函数就会返回“星期一”，输入数字 2 会返回“星期二”，以此类推，如果未输入数字则返回 undefined。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getWeek</span>(<span class="hljs-params">week</span>) </span>{
  <span class="hljs-keyword">const</span> dict = [<span class="hljs-string">'日'</span>, <span class="hljs-string">'一'</span>, <span class="hljs-string">'二'</span>, <span class="hljs-string">'三'</span>, <span class="hljs-string">'四'</span>, <span class="hljs-string">'五'</span>, <span class="hljs-string">'六'</span>];
  <span class="hljs-keyword">if</span>(week) <span class="hljs-keyword">return</span> <span class="hljs-string">`星期<span class="hljs-subst">${dict[week]}</span>`</span>;
}
</code></pre>
<p>这里在 if 语句中就进行了类型转换，将 week 变量转换成 Boolean 数据类型，而 0、空字符串、null、undefined 在转换时都会返回 false。所以这段代码在输入 0 的时候不会返回“星期日”，而返回 undefined。<br>
我们在做强制类型转换的时候一定要考虑这个问题。</p>
<h3>Number</h3>
<h4>两个重要值</h4>
<p>Number 是数值类型，有 2 个特殊数值得注意一下，即 NaN 和 Infinity。</p>
<ul>
<li>NaN（Not a Number）通常在计算失败的时候会得到该值。要判断一个变量是否为 NaN，则可以通过 Number.isNaN 函数进行判断。</li>
<li>Infinity 是无穷大，加上负号 “-” 会变成无穷小，在某些场景下比较有用，比如通过数值来表示权重或者优先级，Infinity 可以表示最高优先级或最大权重。</li>
</ul>
<h4>进制转换</h4>
<p>当我们需要将其他进制的整数转换成十进制显示的时候可以使用 parseInt 函数，该函数第一个参数为数值或字符串，第二个参数为进制数，默认为 10，当进制数转换失败时会返回 NaN。所以，如果在数组的 map 函数的回调函数中直接调用 parseInt，那么会将数组元素和索引值都作为参数传入。</p>
<pre><code data-language="javascript" class="lang-javascript">[<span class="hljs-string">'0'</span>, <span class="hljs-string">'1'</span>, <span class="hljs-string">'2'</span>].map(<span class="hljs-built_in">parseInt</span>) <span class="hljs-comment">// [0, NaN, NaN]</span>
</code></pre>
<p>而将十进制转换成其他进制时，可以通过 toString 函数来实现。</p>
<pre><code data-language="javascript" class="lang-javascript">(<span class="hljs-number">10</span>).toString(<span class="hljs-number">2</span>) <span class="hljs-comment">// "1010"</span>
</code></pre>
<h4>精度问题</h4>
<p>对于数值类型的数据，还有一个比较值得注意的问题，那就是<strong>精度问题</strong>，在进行浮点数运算时很容易碰到。比如我们执行简单的运算 0.1 + 0.2，得到的结果是 0.30000000000000004，如果直接和 0.3 作相等判断时就会得到 false。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-number">0.1</span> + <span class="hljs-number">0.2</span> <span class="hljs-comment">// 0.30000000000000004</span>
</code></pre>
<p>出现这种情况的原因在于计算的时候，JavaScript 引擎会先将十进制数转换为二进制，然后进行加法运算，再将所得结果转换为十进制。在进制转换过程中如果小数位是无限的，就会出现误差。同样的，对于下面的表达式，将数字 5 开方后再平方得到的结果也和数字 5 不相等。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-built_in">Math</span>.pow(<span class="hljs-built_in">Math</span>.pow(<span class="hljs-number">5</span>, <span class="hljs-number">1</span>/<span class="hljs-number">2</span>), <span class="hljs-number">2</span>) <span class="hljs-comment">// 5.000000000000001</span>
</code></pre>
<p>对于这个问题的解决方法也很简单，那就是消除无限小数位。</p>
<ul>
<li>一种方式是先转换成整数进行计算，然后再转换回小数，这种方式适合在小数位不是很多的时候。比如一些程序的支付功能 API 以“分”为单位，从而避免使用小数进行计算。</li>
<li>还有另一种方法就是舍弃末尾的小数位。比如对上面的加法就可以先调用 toPrecision 截取 12 位，然后调用 parseFloat 函数转换回浮点数。</li>
</ul>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-built_in">parseFloat</span>((<span class="hljs-number">0.1</span> + <span class="hljs-number">0.2</span>).toPrecision(<span class="hljs-number">12</span>)) <span class="hljs-comment">// 0.3</span>
</code></pre>
<h3>String</h3>
<p>String 类型是最常用的数据类型了，关于它的基础 API 函数大家应该比较熟悉了，这里我就不多介绍了。下面通过一道笔试题来重点介绍它的使用场景。</p>
<p><strong>千位分隔符</strong>是指为了方便识别较大数字，每隔三位数会加入 1 个逗号，该逗号就是千位分隔符。如果要编写一个函数来为输入值的数字添加千分位分隔符，该怎么实现呢？</p>
<p>一种很容易想到的方法就是从右往左遍历数值每一位，每隔 3 位添加分隔符。为了操作方便，我们可以将数值转换成字符数组，而要实现从右往左遍历，一种实现方式是通过 for 循环的索引值找到对应的字符；而另一种方式是通过数组反转，从而变成从左到右操作。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">sep</span>(<span class="hljs-params">n</span>) </span>{
&nbsp; <span class="hljs-keyword">let</span> [i, c] = n.toString().split(<span class="hljs-regexp">/(\.\d+)/</span>)
&nbsp; <span class="hljs-keyword">return</span> i.split(<span class="hljs-string">''</span>).reverse().map(<span class="hljs-function">(<span class="hljs-params">c, idx</span>) =&gt;</span> (idx+<span class="hljs-number">1</span>) % <span class="hljs-number">3</span> === <span class="hljs-number">0</span> ? <span class="hljs-string">','</span> + c: c).reverse().join(<span class="hljs-string">''</span>).replace(<span class="hljs-regexp">/^,/</span>, <span class="hljs-string">''</span>) + c
}
</code></pre>
<p>这种方式就是将字符串数据转化成引用类型数据，即用数组来实现。</p>
<p>第二种方式则是通过引用类型，即用正则表达式对字符进行替换来实现。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">sep2</span>(<span class="hljs-params">n</span>)</span>{
&nbsp; <span class="hljs-keyword">let</span> str = n.toString()
&nbsp; str.indexOf(<span class="hljs-string">'.'</span>) &lt; <span class="hljs-number">0</span> ? str+= <span class="hljs-string">'.'</span> : <span class="hljs-keyword">void</span> <span class="hljs-number">0</span>
&nbsp; <span class="hljs-keyword">return</span> str.replace(<span class="hljs-regexp">/(\d)(?=(\d{3})+\.)/g</span>, <span class="hljs-string">'$1,'</span>).replace(<span class="hljs-regexp">/\.$/</span>, <span class="hljs-string">''</span>)
}
</code></pre>
<h3>Symbol</h3>
<p>Symbol 是 ES6 中引入的新数据类型，它表示一个唯一的常量，通过 Symbol 函数来创建对应的数据类型，创建时可以添加变量描述，该变量描述在传入时会被强行转换成字符串进行存储。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">var</span> a = <span class="hljs-built_in">Symbol</span>(<span class="hljs-string">'1'</span>)
<span class="hljs-keyword">var</span> b = <span class="hljs-built_in">Symbol</span>(<span class="hljs-number">1</span>)
a.description === b.description <span class="hljs-comment">// true</span>
<span class="hljs-keyword">var</span> c = <span class="hljs-built_in">Symbol</span>({<span class="hljs-attr">id</span>: <span class="hljs-number">1</span>})
c.description <span class="hljs-comment">// [object Object]</span>
<span class="hljs-keyword">var</span> _a = <span class="hljs-built_in">Symbol</span>(<span class="hljs-string">'1'</span>)
_a == a <span class="hljs-comment">// false</span>
</code></pre>
<p>基于上面的特性，Symbol 属性类型比较适合用于两类场景中：<strong>常量值和对象属性</strong>。</p>
<h4>避免常量值重复</h4>
<p>假设有个 getValue 函数，根据传入的字符串参数 key 执行对应代码逻辑。代码如下所示：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getValue</span>(<span class="hljs-params">key</span>) </span>{
&nbsp; <span class="hljs-keyword">switch</span>(key){
&nbsp; &nbsp; <span class="hljs-keyword">case</span> <span class="hljs-string">'A'</span>:
&nbsp; &nbsp; &nbsp; ...
&nbsp; &nbsp; ...
&nbsp; &nbsp; case <span class="hljs-string">'B'</span>:
      ...
&nbsp; }
}
getValue(<span class="hljs-string">'B'</span>);
</code></pre>
<p>这段代码对调用者而言非常不友好，因为代码中使用了魔术字符串（魔术字符串是指在代码之中多次出现、与代码形成强耦合的某一个具体的字符串或者数值），导致调用 getValue 函数时需要查看函数源码才能找到参数 key 的可选值。所以可以将参数 key 的值以常量的方式声明出来。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">const</span> KEY = {
&nbsp; <span class="hljs-attr">alibaba</span>: <span class="hljs-string">'A'</span>,
&nbsp; <span class="hljs-attr">baidu</span>: <span class="hljs-string">'B'</span>,
&nbsp; ...
}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getValue</span>(<span class="hljs-params">key</span>) </span>{
&nbsp; <span class="hljs-keyword">switch</span>(key){
&nbsp; &nbsp; <span class="hljs-keyword">case</span> KEY.alibaba:
&nbsp; &nbsp; &nbsp; ...
&nbsp; &nbsp; ...
&nbsp; &nbsp; case KEY.baidu:
&nbsp; &nbsp; &nbsp; ...
&nbsp; }
}
getValue(KEY.baidu);
</code></pre>
<p>但这样也并非完美，假设现在我们要在 KEY 常量中加入一个 key，根据对应的规则，很有可能会出现值重复的情况：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">const</span> KEY = {
  <span class="hljs-attr">alibaba</span>: <span class="hljs-string">'A'</span>,
  <span class="hljs-attr">baidu</span>: <span class="hljs-string">'B'</span>,
  ...
  bytedance: <span class="hljs-string">'B'</span>
}
</code></pre>
<p>这显然会出现问题：</p>
<pre><code data-language="javascript" class="lang-javascript">getValue(KEY.baidu) <span class="hljs-comment">// 等同于 getValue(KEY.bytedance)</span>
</code></pre>
<p>所以在这种场景下更适合使用 Symbol，我们不关心值本身，只关心值的唯一性。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">const</span> KEY = {
&nbsp; <span class="hljs-attr">alibaba</span>: <span class="hljs-built_in">Symbol</span>(),
&nbsp; <span class="hljs-attr">baidu</span>: <span class="hljs-built_in">Symbol</span>(),
&nbsp; ...
&nbsp; bytedance: <span class="hljs-built_in">Symbol</span>()
}
</code></pre>
<h4>避免对象属性覆盖</h4>
<p>假设有这样一个函数 fn，需要对传入的对象参数添加一个临时属性 user，但可能该对象参数中已经有这个属性了，如果直接赋值就会覆盖之前的值。此时就可以使用 Symbol 来避免这个问题。</p>
<p>创建一个 Symbol 数据类型的变量，然后将该变量作为对象参数的属性进行赋值和读取，这样就能避免覆盖的情况，示例代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fn</span>(<span class="hljs-params">o</span>) </span>{ <span class="hljs-comment">// {user: {id: xx, name: yy}}</span>
&nbsp; <span class="hljs-keyword">const</span> s = <span class="hljs-built_in">Symbol</span>()
&nbsp; o[s] = <span class="hljs-string">'zzz'</span>
&nbsp; ...
}
</code></pre>
<h3>补充：类型转换</h3>
<h4>什么是类型转换？</h4>
<p>JavaScript 这种弱类型的语言，相对于其他高级语言有一个特点，那就是在处理不同数据类型运算或逻辑操作时会强制转换成同一数据类型。如果我们不理解这个特点，就很容易在编写代码时产生 bug。</p>
<p>通常强制转换的目标数据类型为 String、Number、Boolean 这三种。下面的表格中显示了 6 种基础数据类型转换关系。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/17/C1/CgqCHl7XaNOAOR-5AAC7iyHcEyQ034.png" alt="前端07.png"></p>
<p>除了不同类型的转换之外，操作同种数据类型也会发生转换。把基本类型的数据换成对应的对象过程称之为“<strong>装箱转换</strong>”，反过来，把数据对象转换为基本类型的过程称之为“<strong>拆箱转换</strong>”。</p>
<p>对于装箱和拆箱转换操作，我们既可以显示地手动实现，比如将 Number 数据类型转换成 Number 对象；也可以通过一些操作触发浏览器显式地自动转换，比如将对 Number 对象进行加法运算。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">var</span> n = <span class="hljs-number">1</span>
<span class="hljs-keyword">var</span> o = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Number</span>(n) <span class="hljs-comment">// 显式装箱</span>
o.valueOf() <span class="hljs-comment">// 显式拆箱</span>
n.toPrecision(<span class="hljs-number">3</span>) <span class="hljs-comment">// 隐式装箱, 实际操作：var tmp = new Number(n);tmp.toPrecision(3);tmp = null;</span>
o + <span class="hljs-number">2</span> <span class="hljs-comment">// 隐式拆箱，实际操作:var tmp = o.valueOf();tmp + 2;tmp = null;</span>
</code></pre>
<h4>什么时候会触发类型转换？</h4>
<p>下面这些常见的操作会触发隐式地类型转换，我们在编写代码的时候一定要注意。</p>
<ul>
<li><strong>运算相关的操作符</strong>包括 +、-、+=、++、* 、/、%、&lt;&lt;、&amp; 等。</li>
<li><strong>数据比较相关的操作符</strong>包括 &gt;、&lt;、== 、&lt;=、&gt;=、===。</li>
<li><strong>逻辑判断相关的操作符</strong>包括 &amp;&amp;、!、||、三目运算符。</li>
</ul>
<h3>Object</h3>
<p>相对于基础类型，引用类型 Object 则复杂很多。简单地说，Object 类型数据就是键值对的集合，键是一个字符串（或者 Symbol） ，值可以是任意类型的值； 复杂地说，Object 又包括很多子类型，比如 Date、Array、Set、RegExp。</p>
<p>对于 Object 类型，我们重点理解一种常见的操作，即深拷贝。</p>
<ul>
<li>由于引用类型在赋值时只传递指针，这种拷贝方式称为<strong>浅拷贝</strong>。</li>
<li>而创建一个新的与之相同的引用类型数据的过程称之为<strong>深拷贝</strong>。</li>
</ul>
<p>现在我们来实现一个拷贝函数，支持上面 7 种类型的数据拷贝。</p>
<p>对于 6 种基础类型，我们只需简单的赋值即可，而 Object 类型变量需要特殊操作。因为通过等号“=”赋值只是<strong>浅拷贝</strong>，要实现真正的拷贝操作则需要通过遍历键来赋值对应的值，这个过程中如果遇到 Object 类型还需要再次进行遍历。</p>
<p>为了准确判断每种数据类型，我们可以先通过 typeof 来查看每种数据类型的描述：</p>
<pre><code data-language="javascript" class="lang-javascript">[<span class="hljs-literal">undefined</span>,&nbsp;<span class="hljs-literal">null</span>,&nbsp;<span class="hljs-literal">true</span>,&nbsp;<span class="hljs-string">''</span>,&nbsp;<span class="hljs-number">0</span>,&nbsp;<span class="hljs-built_in">Symbol</span>(),&nbsp;{}].map(<span class="hljs-function"><span class="hljs-params">it</span>&nbsp;=&gt;</span>&nbsp;<span class="hljs-keyword">typeof</span>&nbsp;it)<span class="hljs-comment">//&nbsp;["undefined",&nbsp;"object",&nbsp;"boolean",&nbsp;"string",&nbsp;"number",&nbsp;"symbol",&nbsp;"object"]</span>
</code></pre>
<p>发现 null 有些特殊，返回结果和 Object 类型一样都为"object"，所以需要再次进行判断。按照上面分析的结论，我们可以写出下面的函数：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">clone</span>(<span class="hljs-params">data</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;result&nbsp;=&nbsp;{}
&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;keys&nbsp;=&nbsp;[...Object.getOwnPropertyNames(data),&nbsp;...Object.getOwnPropertySymbols(data)]
&nbsp;&nbsp;<span class="hljs-keyword">if</span>(!keys.length)&nbsp;<span class="hljs-keyword">return</span>&nbsp;data
&nbsp;&nbsp;keys.forEach(<span class="hljs-function"><span class="hljs-params">key</span>&nbsp;=&gt;</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;item&nbsp;=&nbsp;data[key]
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(<span class="hljs-keyword">typeof</span>&nbsp;item&nbsp;===&nbsp;<span class="hljs-string">'object'</span>&nbsp;&amp;&amp;&nbsp;item)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result[key]&nbsp;=&nbsp;clone(item)
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result[key]&nbsp;=&nbsp;item
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;})
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;result
}
</code></pre>
<p>在遍历 Object 类型数据时，我们需要把 Symbol 数据类型也考虑进来，所以不能通过 Object.keys 获取键名或 for...in 方式遍历，而是通过 getOwnPropertyNames 和 getOwnPropertySymbols 函数将键名组合成数组，然后进行遍历。对于键数组长度为 0 的非 Object 类型的数据可直接返回，然后再遍历递归，最终实现拷贝。</p>
<p>我们在编写递归函数的时候需要特别注意的是，递归调用的终止条件，避免无限递归。那在这个 clone 函数中有没有可能出现无限递归调用呢？</p>
<p>答案是有的。那就是当对象数据嵌套的时候，比如像下面这种情况，对象 a 的键 b 指向对象 b，对象 b 的键 a 指向对象 a，那么执行 clone 函数就会出现死循环，从而耗尽内存。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">var</span> a = {
<span class="hljs-keyword">var</span> b = {}
a.b = b
b.a = a
</code></pre>
<p>怎么避免这种情况呢？一种简单的方式就是把已添加的对象记录下来，这样下次碰到相同的对象引用时，直接指向记录中的对象即可。要实现这个记录功能，我们可以借助 ES6 推出的 WeakMap 对象，该对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。</p>
<p>我们对 clone 函数改造一下，添加一个 WeakMap 来记录已经拷贝过的对象，如果当前对象已经被拷贝过，那么直接从 WeakMap 中取出，否则重新创建一个对象并加入 WeakMap 中。具体代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">clone</span>(<span class="hljs-params">obj</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;map&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-built_in">WeakMap</span>()
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">deep</span>(<span class="hljs-params">data</span>)&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;result&nbsp;=&nbsp;{}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;keys&nbsp;=&nbsp;[...Object.getOwnPropertyNames(data),&nbsp;...Object.getOwnPropertySymbols(data)]
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>(!keys.length)&nbsp;<span class="hljs-keyword">return</span>&nbsp;data
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;exist&nbsp;=&nbsp;map.get(data)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(exist)&nbsp;<span class="hljs-keyword">return</span>&nbsp;exist
&nbsp;&nbsp;&nbsp;&nbsp;map.set(data,&nbsp;result)
&nbsp;&nbsp;&nbsp;&nbsp;keys.forEach(<span class="hljs-function"><span class="hljs-params">key</span>&nbsp;=&gt;</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;item&nbsp;=&nbsp;data[key]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(<span class="hljs-keyword">typeof</span>&nbsp;item&nbsp;===&nbsp;<span class="hljs-string">'object'</span>&nbsp;&amp;&amp;&nbsp;item)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result[key]&nbsp;=&nbsp;deep(item)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result[key]&nbsp;=&nbsp;item
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;result
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;deep(obj)
}
</code></pre>
<h3>总结</h3>
<p>这一课时通过实例与原理相结合，带你深入理解了 JavaScript 的 6 种基础数据类型和 1 种引用数据类型。对于 6 种基础数据类型，我们要熟知它们之间的转换关系，而引用类型则比较复杂，重点讲了如何深拷贝一个对象。其实引用对象的子类型比较多，由于篇幅所限没有进行一一讲解，需要大家在平常工作中继续留心积累。</p>
<p>最后布置一道思考题：你能否写出一个函数来判断两个变量是否相等？</p>

---

### 精选评论

##### Amanda：
> 已经有第8种类型了

##### **玥：
> 千位分隔符的第一种方法输入整数会输出整数+undefined

##### **.：
> 自带千分位函数(1234567).toLocaleString()

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯，这是一个高效的方法，但最好指定语言环境，例如 (1234567).toLocaleString('zh-Hans-CN')

##### *客：
> "(10).toString(16)" 这里为什么加了"括号"就可以用 toString 方法，不加的话就报错；加"括号"的话，编译器是做了什么吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 默认地，数字后的点号会被当成小数来进行解析，所以接“toString”会报错。而加了括号之后在进行词法分析的时候括号内的数字 10 就会被单独解析，和后面的属性 toString 以及参数 16 组合成一个表达式。

##### **文：
> 值类型(基本类型)：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。引用数据类型：对象(Object)、数组(Array)、函数(Function)。

##### **用户7763：
> 关于改变this指向的，call，apply，bind…还有吗😂😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; class 中 this 会指向类的实例。

##### **池：
> 这节课很奈斯～😇

##### *聪：
> 还有一种类型：BigInt

##### **珍：
> 关于浅拷贝，我感觉我很困惑。有些说浅拷贝是一层拷贝，有些浅拷贝是等号赋值。所以具体是哪个？我都懵了😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没理解你所说的“一层拷贝“~ 浅拷贝你可以理解为给变量起了个别名，并没有创建新的值。

##### *盼：
> 赞

