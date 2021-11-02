<p data-nodeid="473184">随着前端快速发展，JavaScript 语言的设计缺陷在大型项目中逐渐显露。</p>
<p data-nodeid="473185">第 10 课时提到的模块问题就是其中之一，但庆幸的是，ES6 模块在原生层面解决了这个问题，不同环境下的兼容性问题也可以由工具转化代码来解决。</p>
<p data-nodeid="473186">这一课时要提到的<strong data-nodeid="473272">类型问题</strong>，是一个需要依赖第三方规范和工具来解决的缺陷。JavaScript 的类型问题具体表现在下面 3 个方面。</p>
<p data-nodeid="473187"><strong data-nodeid="473280">1.</strong> <strong data-nodeid="473281">类型声明</strong></p>
<p data-nodeid="473188">前面在第 08 课时中已经提过命名的提升特性，如果某个变量命名提升到全局，那么将是危险的。比如下面的代码，函数 fn 内部使用了一个变量 c，由于忘记使用关键字来声明，结果导致覆盖了全局变量 c。</p>
<pre class="lang-javascript" data-nodeid="473189"><code data-language="javascript"><span class="hljs-keyword">var</span> c = <span class="hljs-number">0</span>
...
function fn() {
&nbsp; ...
&nbsp; c = <span class="hljs-number">30</span>;
}
fn();
</code></pre>
<p data-nodeid="473190"><strong data-nodeid="473290">2.</strong> <strong data-nodeid="473291">动态类型</strong></p>
<p data-nodeid="473191">动态类型是指在运行期间才做数据类型检查的语言，即动态类型语言编程时，不用给任何变量指定数据类型。</p>
<p data-nodeid="473192">下面是一个简单的例子，定义了一个函数  printId 来返回某个对象的 id 属性。如果我们在调用函数 printId 时要想了解参数 user 的数据结构和返回值类型，只能通过查看源码，或者运行时调试、打印来获取。当函数结构复杂，参数较多时这个过程就会大大降低代码的可维护性。虽然添加注释能在一定程度上缓解问题，但为函数编写注释并不是强制性约束，能否及时同步注释也可能会成为新的问题。</p>
<p data-nodeid="473193">就函数 printId 本身而言，也无法在编译时校验参数的合法性，只能在运行时添加校验逻辑，这也大大增加了程序出现 bug 的概率。</p>
<pre class="lang-javascript" data-nodeid="473194"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">printId</span>(<span class="hljs-params">user</span>) </span>{
  <span class="hljs-keyword">return</span> user.id
}
</code></pre>
<p data-nodeid="473195"><strong data-nodeid="473302">3.</strong> <strong data-nodeid="473303">弱类型</strong></p>
<p data-nodeid="473196">弱类型是指一个变量可以被赋予不同数据类型的值。这也是一个既灵活又可怕的特性，编写代码的时候非常方便，不用考虑变量的数据类型，但这也很容易出现 bug，调试起来会变得相当困难。</p>
<pre class="lang-javascript" data-nodeid="473197"><code data-language="javascript"><span class="hljs-keyword">var</span> tmp = []
...
tmp = <span class="hljs-literal">null</span>
...
<span class="hljs-comment">// tmp 到底会变成什么？</span>
</code></pre>
<p data-nodeid="473198">为了解决上面 3 个问题，开源社区提供了解决方案——TypeScript。它是基于 JavaScript 的语法糖，也就是说 TypeScript 代码没有单独的运行环境，需要编译成 JavaScript 代码之后才能运行。<br>
从它的名字不难看出，它的核心特性是类型“Type”。具体工作原理就是在代码编译阶段进行类型检测，这样就能在代码部署运行之前及时发现问题。</p>
<h3 data-nodeid="473199">类型与接口</h3>
<p data-nodeid="473200">TypeScript 让 JavaScript 变成了<strong data-nodeid="473324">静态强类型****、变量</strong>需要严格声明的语言，为此定义了两个重要概念：<strong data-nodeid="473326">类型（type）<strong data-nodeid="473325">和</strong>接口（interface）</strong>。</p>
<p data-nodeid="473201">TypeScript 在 JavaScript 原生类型的基础上进行了扩展，但为了和基础类型对象进行区分，采用了小写的形式，比如 Number 类型对应的是 number。类型之间可以互相组合形成新的类型。</p>
<p data-nodeid="473202">一些数据类型在前面第 07 课时中已经提过，这里不再赘述。下面补充一下 TypeScript 扩展的类型。</p>
<p data-nodeid="473203"><strong data-nodeid="473336">1.</strong> <strong data-nodeid="473337">元组</strong></p>
<p data-nodeid="473204">元组可以看成是具有固定长度的数组，其中数组元素类型可以不同。比如下面的代码声明了一个元组变量 x，x 的第一个元素是字符串，第二个是数字；又比如 react hooks 就是用到了元组类型。</p>
<pre class="lang-typescript" data-nodeid="473205"><code data-language="typescript"><span class="hljs-keyword">let</span> x: [<span class="hljs-built_in">string</span>, <span class="hljs-built_in">number</span>];
</code></pre>
<p data-nodeid="473206"><strong data-nodeid="473346">2.</strong> <strong data-nodeid="473347">枚举</strong></p>
<p data-nodeid="473207">枚举指的是带有名字的常量，可以分为<strong data-nodeid="473361">数字枚举</strong>、<strong data-nodeid="473362">字符串枚举</strong>和<strong data-nodeid="473363">异构枚举</strong>（字符串和数字的混合）3 种。比较适用于前后端通用的枚举值，比如通过 AJAX 请求获取的数据状态，对于仅在前端使用的枚举值还是推荐使用 Symbol。</p>
<p data-nodeid="473208">下面是一个异构枚举的例子，定义了数字枚举值 0 和字符串枚举值 "YES"。</p>
<pre class="lang-typescript" data-nodeid="473209"><code data-language="typescript"><span class="hljs-built_in">enum</span> example {
    No = <span class="hljs-number">0</span>,
    Yes = <span class="hljs-string">"YES"</span>,
}
</code></pre>
<p data-nodeid="473210">也可以使用 const 修饰符来定义枚举值，通过这种定义方式，TypeScript 会在编译的时候，直接把枚举引用替换成对应的枚举值而非创建枚举对象。</p>
<pre class="lang-typescript" data-nodeid="473211"><code data-language="typescript"><span class="hljs-built_in">enum</span> example {
&nbsp; &nbsp; No = <span class="hljs-number">0</span>,
&nbsp; &nbsp; Yes = <span class="hljs-string">"YES"</span>,
}
<span class="hljs-built_in">console</span>.log(example.No)
<span class="hljs-comment">// 编译成</span>
<span class="hljs-keyword">var</span> example;
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">example</span>) </span>{
&nbsp; &nbsp; example[example[<span class="hljs-string">"No"</span>] = <span class="hljs-number">0</span>] = <span class="hljs-string">"No"</span>;
&nbsp; &nbsp; example[<span class="hljs-string">"Yes"</span>] = <span class="hljs-string">"YES"</span>;
})(example || (example = {}));
<span class="hljs-built_in">console</span>.log(example.No);
<span class="hljs-comment">////////////</span>
<span class="hljs-keyword">const</span> <span class="hljs-built_in">enum</span> example {
&nbsp; &nbsp; No = <span class="hljs-number">0</span>,
&nbsp; &nbsp; Yes = <span class="hljs-string">"YES"</span>,
}
<span class="hljs-built_in">console</span>.log(example.No)
<span class="hljs-comment">//  编译成</span>
<span class="hljs-built_in">console</span>.log(<span class="hljs-number">0</span> <span class="hljs-comment">/* No */</span>);
</code></pre>
<p data-nodeid="473212"><strong data-nodeid="473377">3.</strong> <strong data-nodeid="473378">any</strong></p>
<p data-nodeid="473213">any 类型代表可以是任何一种类型，所以会跳过类型检查，相当于让变量或返回值又变成弱类型。因此建议尽量减少 any 类型的使用。</p>
<p data-nodeid="473214"><strong data-nodeid="473387">4.</strong> <strong data-nodeid="473388">void</strong></p>
<p data-nodeid="473215">void 表示没有任何类型，常用于描述无返回值的函数。</p>
<p data-nodeid="473216"><strong data-nodeid="473397">5.</strong> <strong data-nodeid="473398">never</strong></p>
<p data-nodeid="473217">never 类型表示的是那些永不存在的值的类型，对于一些特殊的校验场景比较有用，比如代码的完整性检查。下面的示例代码通过穷举判断变量 u 的值来执行对应逻辑，如果此时变量 u 的可选值新增了字符串 "c"，那么这段代码并不会给出提示告诉开发者还有一种  u 等于字符串 "c" 的场景，但如果增加 never 类型赋值的话在编译时就可以给出提示。</p>
<pre class="lang-typescript" data-nodeid="473218"><code data-language="typescript"><span class="hljs-keyword">let</span> u: <span class="hljs-string">'a'</span>|<span class="hljs-string">'b'</span>
<span class="hljs-comment">//...</span>
<span class="hljs-keyword">if</span>(u === <span class="hljs-string">'a'</span>) {
&nbsp; <span class="hljs-comment">//...</span>
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (u === <span class="hljs-string">'b'</span>) {
&nbsp; <span class="hljs-comment">//...</span>
}
</code></pre>
<p data-nodeid="473219">增加了 never 类型变量赋值：</p>
<pre class="lang-typescript" data-nodeid="473220"><code data-language="typescript"><span class="hljs-keyword">let</span> u: <span class="hljs-string">'a'</span>|<span class="hljs-string">'b'</span>|<span class="hljs-string">'c'</span>
<span class="hljs-comment">//...</span>
<span class="hljs-keyword">if</span>(u === <span class="hljs-string">'a'</span>) {
&nbsp; <span class="hljs-comment">//...</span>
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (u === <span class="hljs-string">'b'</span>) {
&nbsp; <span class="hljs-comment">//...</span>
} <span class="hljs-keyword">else</span> {
&nbsp; <span class="hljs-keyword">let</span> trmp: <span class="hljs-built_in">never</span> = u&nbsp;<span class="hljs-comment">// Type '"c"' is not assignable to type 'never'.</span>
}
</code></pre>
<p data-nodeid="473221">接口的作用和类型非常相似，在大多数情况下可以通用，只存在一些细小的区别（比如同名接口可以自动合并，而类型不能；在编译器中将鼠标悬停在接口上显示的是接口名称，悬停在类型上显示的是字面量类型），最明显的区别还是在写法上。</p>
<pre class="lang-typescript" data-nodeid="473222"><code data-language="typescript"><span class="hljs-comment">/* 声明 */</span>
<span class="hljs-keyword">interface</span> IA {
  id: <span class="hljs-built_in">string</span>
}
<span class="hljs-keyword">type</span> TA = {
  id: <span class="hljs-built_in">string</span>
}
<span class="hljs-comment">/* 继承 */</span>
<span class="hljs-keyword">interface</span> IA2 <span class="hljs-keyword">extends</span> IA {
&nbsp; &nbsp; name: <span class="hljs-built_in">string</span>
}
<span class="hljs-keyword">type</span> TA2 = TA &amp; { name: <span class="hljs-built_in">string</span> }
<span class="hljs-comment">/* 实现 */</span>
<span class="hljs-keyword">class</span> A <span class="hljs-keyword">implements</span> IA {
&nbsp; &nbsp; id: <span class="hljs-built_in">string</span> = <span class="hljs-string">''</span>
}
<span class="hljs-keyword">class</span> A2 <span class="hljs-keyword">implements</span> TA {
&nbsp; &nbsp; id: <span class="hljs-built_in">string</span> = <span class="hljs-string">''</span>
}
</code></pre>
<h3 data-nodeid="473223">类型抽象</h3>
<p data-nodeid="473224"><strong data-nodeid="473415">泛型</strong>是对类型的一种抽象，一般用于函数，能让调用者动态地指定部分数据类型。这一点和 any 类型有些像，对于类型的定义具有不确定性，可以指代多种类型，但最大区别在于泛型可以对函数成员或类成员产生约束关系。</p>
<p data-nodeid="473225">下面代码是 react 的钩子函数 useState 的类型定义，就用到了泛型。</p>
<pre class="lang-typescript" data-nodeid="473226"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">useState</span>&lt;<span class="hljs-title">S</span>&gt;(<span class="hljs-params">initialState: S | (() =&gt; S)</span>): [<span class="hljs-title">S</span>, <span class="hljs-title">Dispatch</span>&lt;<span class="hljs-title">SetStateAction</span>&lt;<span class="hljs-title">S</span>&gt;&gt;]</span>;
</code></pre>
<p data-nodeid="473227">这段代码中 S 称为<strong data-nodeid="473424">泛型变量</strong>。从这个定义可看出，useState 可以接收任何类型的参数或回调函数，但返回的元组数据第一个值必定和参数类型或者回调函数返回值类型相同，都为 S。<br>
如果使用 any 类型来取代泛型，那么我们只能知道允许传入任何参数或回调函数，而无法知道返回值与入参的对应关系。</p>
<p data-nodeid="473228">在使用泛型的时候，我们可以通过尖括号来手动指定泛型变量的类型，这个指定操作称之为**类型断言，**也可以不指定，让 TypeScript 自行推断类型。比如下面的代码就通过类型断言，将范型变量指定为 string 类型。</p>
<pre class="lang-typescript" data-nodeid="473229"><code data-language="typescript"><span class="hljs-keyword">const</span> [id, setId] = useState&lt;<span class="hljs-built_in">string</span>&gt;(<span class="hljs-string">''</span>);
</code></pre>
<h3 data-nodeid="473230">类型组合</h3>
<p data-nodeid="473231">类型组合就是把现有的多种类型叠加到一起，组合成一种新的类型，具体有两种方式。</p>
<h4 data-nodeid="473232"><strong data-nodeid="473435">交叉</strong></h4>
<p data-nodeid="473233">交叉就是将多个类型合并为一个类型，操作符为 “&amp;” 。下面的代码定义了一个 Admin 类型，它同时是类型 Student 和类型 Teacher 的交叉类型。 就是说 Admin 类型的对象同时拥有了这 2 种类型的成员。</p>
<pre class="lang-typescript" data-nodeid="473234"><code data-language="typescript"><span class="hljs-keyword">type</span> Admin = Student &amp; Teacher
</code></pre>
<h4 data-nodeid="473235"><strong data-nodeid="473442">联合</strong></h4>
<p data-nodeid="473236">联合就是表示符合多种类型中的任意一个，不同类型通过操作符“|”连接。下面代码定义的类型是 AorB，表示该类型值可以是类型 A，也可以是类型 B。</p>
<pre class="lang-typescript" data-nodeid="473237"><code data-language="typescript"><span class="hljs-keyword">type</span> A = {
  a: <span class="hljs-built_in">string</span>
}
<span class="hljs-keyword">type</span> B = {
  b: <span class="hljs-built_in">number</span>
}
<span class="hljs-keyword">type</span> AorB = A | B
</code></pre>
<p data-nodeid="473238">对于联合类型 AorB，我们能够确定的是它包含了 A 和 B 中共有的成员。如果我们想确切地了解值是否为类型 A，只能通过检查值的方法是否存在来进行判断。例如，下面的变量 v 属于 AorB 类型，在需要确认其具体类型时，先将变量 v 的类型断言为 A，然后再调用其属性 a 进行判断。</p>
<pre class="lang-typescript" data-nodeid="473239"><code data-language="typescript"><span class="hljs-keyword">let</span> v: AorB
<span class="hljs-comment">// ...</span>
<span class="hljs-keyword">if</span> ((&lt;A&gt;v).a) {
&nbsp; <span class="hljs-comment">//...</span>
} <span class="hljs-keyword">else</span> {
&nbsp; (&lt;B&gt;v).b
&nbsp; <span class="hljs-comment">//...</span>
}
</code></pre>
<h3 data-nodeid="473240">类型引用</h3>
<h4 data-nodeid="477306" class="">索引</h4>




<p data-nodeid="473242">索引类型的目的是让 TypeScript 编译器检查出使用了动态属性名的类型，需要通过<strong data-nodeid="473459">索引类型查询</strong>和<strong data-nodeid="473460">索引类型访问</strong>来实现。</p>
<p data-nodeid="473243">下面的示例代码实现了一个简单的函数 getValue ，传入对象和对象属性名获取对应的值。</p>
<pre class="lang-typescript" data-nodeid="473244"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getValue</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">K</span> <span class="hljs-title">extends</span> <span class="hljs-title">keyof</span> <span class="hljs-title">T</span>&gt;(<span class="hljs-params">o: T, name: K</span>): <span class="hljs-title">T</span>[<span class="hljs-title">K</span>] </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> o[name]; <span class="hljs-comment">// o[name] is of type T[K]</span>
}
<span class="hljs-keyword">let</span> com = {
&nbsp; &nbsp; name: <span class="hljs-string">'lagou'</span>,
&nbsp; &nbsp; id: <span class="hljs-number">123</span>
}
<span class="hljs-keyword">let</span> id: <span class="hljs-built_in">number</span> = getValue(com, <span class="hljs-string">'id'</span>)
<span class="hljs-keyword">let</span> no = getValue(com, <span class="hljs-string">'no'</span>) <span class="hljs-comment">//报错：Argument of type '"no"' is not assignable to parameter of type '"id" | "name"'.</span>
</code></pre>
<p data-nodeid="477932">其中，泛型变量 K 继承了泛型变量 T 的属性名联合，这里的 keyof 就是索引类型查询操作符；返回值 T[K] 就是索引访问操作符的使用方式。</p>
<p data-nodeid="477933">前面提到的 Pick 类型就是通过索引类型来实现的。</p>

<h4 data-nodeid="473246">映射</h4>
<p data-nodeid="473247">映射类型是指从已有类型中创建新的类型。TypeScript 预定义了一些类型，比如最常用的 Pick 和 Omit。</p>
<p data-nodeid="473248">下面是 Pick 类型的使用示例及源码，可以看到类型 Pick 从类型 task 中选择属性 "title" 和 "description" 生成了新的类型 simpleTask。</p>
<pre class="lang-typescript" data-nodeid="473249"><code data-language="typescript"><span class="hljs-keyword">type</span> Pick&lt;T, K <span class="hljs-keyword">extends</span> keyof T&gt; = {
&nbsp; [P <span class="hljs-keyword">in</span> K]: T[P];
};
<span class="hljs-keyword">interface</span> task {
&nbsp; title: <span class="hljs-built_in">string</span>;
&nbsp; description: <span class="hljs-built_in">string</span>;
  status: <span class="hljs-built_in">string</span>;
}
<span class="hljs-keyword">type</span> simpleTask = Pick&lt;task, <span class="hljs-string">'title'</span> | <span class="hljs-string">'description'</span>&gt;<span class="hljs-comment">// {title: string;description: string}</span>
</code></pre>
<p data-nodeid="478564">类型 Pick 的实现，先用到了索引类型查询，获取了类型 T 的属性名联合 K，然后通过操作符 in 对其进行遍历，同时又用到了索引类型访问来表示属性值。</p>
<p data-nodeid="478565">由于篇幅所限，更多的预定义类型这里就不一一讲解了，对实现原理感兴趣的同学可以参看其<a href="https://github.com/microsoft/TypeScript/blob/master/lib/lib.es5.d.ts" data-nodeid="478570">源码</a>。</p>

<h3 data-nodeid="473251">实践：编写类型声明</h3>
<p data-nodeid="473252">结合上面所说的内容，再通过一个例子来加深理解。我们以第 03 课时的代码 2 的 debounce 函数为例，为这段代码添加类型声明，转换成 TeypScript 语法。</p>
<p data-nodeid="473253">需要添加类型声明的地方通常是<strong data-nodeid="473494">变量和函数</strong>。</p>
<p data-nodeid="473254">首先给函数 debounce 添加类型，包括参数类型和返回值类型。参数类型使用泛型变量，在调用函数 debounce 的时候手动指定，泛型变量有 3 个：函数 T 、函数 T 的返回值 U 和 函数 T 的参数 V。</p>
<p data-nodeid="473255">然后是变量 timeout ，当定时器存在时它的值为 number，定时器不存在时值为 null。</p>
<p data-nodeid="473256">最后按照之前定义的泛型变量给函数 debounced 和函数 flush 添加类型声明。</p>
<p data-nodeid="473257">具体代码如下：</p>
<pre class="lang-typescript" data-nodeid="473258"><code data-language="typescript"><span class="hljs-keyword">const</span> debounce = &lt;T <span class="hljs-keyword">extends</span> <span class="hljs-built_in">Function</span>, U, V <span class="hljs-keyword">extends</span> <span class="hljs-built_in">any</span>[]&gt;<span class="hljs-function">(<span class="hljs-params">func: T, wait: <span class="hljs-built_in">number</span> = <span class="hljs-number">0</span></span>) =&gt;</span> {
&nbsp; <span class="hljs-keyword">let</span> timeout: <span class="hljs-built_in">number</span> | <span class="hljs-literal">null</span> = <span class="hljs-literal">null</span>
&nbsp; <span class="hljs-keyword">let</span> args: V
&nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">debounced</span>(<span class="hljs-params">...arg: V</span>): <span class="hljs-title">Promise</span>&lt;<span class="hljs-title">U</span>&gt; </span>{
&nbsp; &nbsp; args = arg
&nbsp; &nbsp; <span class="hljs-keyword">if</span>(timeout) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">clearTimeout</span>(timeout)
&nbsp; &nbsp; &nbsp; timeout = <span class="hljs-literal">null</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 以 Promise 的形式返回函数执行结果</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">res, rej</span>) =&gt;</span> {
&nbsp; &nbsp; &nbsp; timeout = <span class="hljs-built_in">setTimeout</span>(<span class="hljs-keyword">async</span> () =&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">const</span> result: U = <span class="hljs-keyword">await</span> func.apply(<span class="hljs-built_in">this</span>, args)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res(result)
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span>(e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; rej(e)
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; }, wait)
&nbsp; &nbsp; })
&nbsp; }
&nbsp; <span class="hljs-comment">// 允许取消</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">cancel</span>(<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; <span class="hljs-built_in">clearTimeout</span>(timeout)
&nbsp; &nbsp; timeout = <span class="hljs-literal">null</span>
&nbsp; }
&nbsp; <span class="hljs-comment">// 允许立即执行</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">flush</span>(<span class="hljs-params"></span>): <span class="hljs-title">U</span> </span>{
&nbsp; &nbsp; cancel()
&nbsp; &nbsp; <span class="hljs-keyword">return</span> func.apply(<span class="hljs-built_in">this</span>, args)
&nbsp; }
&nbsp; debounced.cancel = cancel
&nbsp; debounced.flush = flush
&nbsp; <span class="hljs-keyword">return</span> debounced
}
</code></pre>
<h3 data-nodeid="473259">总结</h3>
<p data-nodeid="473260">这一课时重点讲述了如何通过 TypeScript 来解决 JavaScript 的类型问题，TypeScript 在原有的基础类型上进行了扩展，理解 TypeScript 的基本类型并不难，重点需要掌握如何通过泛型来对类型进行抽象，如何通过组合及引用来对已有的类型创建新的类型。</p>
<p data-nodeid="473261">最后布置一道思考题：TypeScript 能较好地解决编译时类型校验的问题，但无法对运行时的数据（比如通过 AJAX 请求获得的数据）进行校验，你能想到有什么好的方法解决这个问题吗？</p>

---

### 精选评论

##### **磊：
> 有些项目做着做着就变成了anyscript

##### *霄：
> 看到 never，发现看完 never 的解释仍然不清楚，于是看 TS 文档，非常清楚，never 表示用于永远不会发生的值类型，一般用作执行不到 return 的函数返回值类型。never 是任意类型的子类型，却没有任意类型是 never 的子类型。```js// Function returning never must have unreachable end point
function error(message: string): never {
    throw new Error(message);
}

// Inferred return type is never
function fail() {
    return error("Something failed");
}

// Function returning never must have unreachable end point
function infiniteLoop(): never {
    while (true) {
    }
}```

##### *道：
> 思考题：可以利用泛型化请求响应类型来解决

##### **飞：
> 命名提升，还用let可以解决这个问题吗？还是打包转换之后，都会变成var?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; let 可以避免命名提升；如果在打包时转换成ES5语法，是会变成var的。

