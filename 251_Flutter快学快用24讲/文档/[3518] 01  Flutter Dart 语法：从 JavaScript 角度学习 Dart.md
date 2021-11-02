<p data-nodeid="7214" class="">本课时我主要从 JavaScript 角度来讲解如何学习 Dart。</p>
<p data-nodeid="7215">在学习本课时之前，你需要有一定的 JavaScript 基础，比如基础数据类型、函数、基础运算符、类、异步原理和文件库引入等，这也是 JavaScript 的核心知识点。接下来将通过对比与 JavaScript 的差异点来学习 Dart 语言。</p>
<h3 data-nodeid="7216">基础数据类型</h3>
<p data-nodeid="7217">与 JavaScript 相比较，我们整体上看一下图 1 两种语言的对比情况，相似的部分这里就不介绍了，比如 Number 和 String，其使用方式基本一致。下面主要基于两者的差异点逐一讲解，避免混淆或错误使用。</p>
<p data-nodeid="7218"><img src="https://s0.lgstatic.com/i/image/M00/1C/5F/CgqCHl7gWVGAdOndAACK0gUBRW0309.png" alt="image (4).png" data-nodeid="7298"></p>
<p data-nodeid="7219">图 1 Dart 与 JavaScript 基础数据类型对比</p>
<h4 data-nodeid="7220">Symbol 的区别</h4>
<p data-nodeid="7221">在 JavaScript 中，Symbol 是将基础数据类型转换为唯一标识符，核心应用是可以将复杂引用数据类型转换为对象数据类型的键名。</p>
<p data-nodeid="7222">在 Dart 中，Symbol 是不透明的动态字符串名称，用于反映库中的元数据。用 Symbol 可以获得或引用类的一个镜像，概念比较复杂，但其实和 JavaScript 的用法基本上是一致的。例如，下面代码首先 new 了一个 test 为 Map 数据类型，设置一个属性 #t（Symbol 类型），然后分别打印 test、test 的 #t、test 的 Symbol("t") 和 #t。</p>
<pre class="lang-dart" data-nodeid="7223"><code data-language="dart"><span class="hljs-keyword">void</span> main() {
  <span class="hljs-built_in">Map</span> test = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Map</span>();
  test[#t] = <span class="hljs-string">'symbol test'</span>;
  <span class="hljs-built_in">print</span>(test);
  <span class="hljs-built_in">print</span>(test[#t]);
  <span class="hljs-built_in">print</span>(test[<span class="hljs-built_in">Symbol</span>(<span class="hljs-string">'t'</span>)]);
  <span class="hljs-built_in">print</span>(#t);
}
</code></pre>
<p data-nodeid="7224">运行代码结果如下：</p>
<pre class="lang-dart" data-nodeid="7225"><code data-language="dart">flutter: {<span class="hljs-built_in">Symbol</span>(<span class="hljs-string">"t"</span>): symbol test}
flutter: symbol test
flutter: symbol test
flutter: <span class="hljs-built_in">Symbol</span>(<span class="hljs-string">"t"</span>)
</code></pre>
<p data-nodeid="7226">其中，test 包含了一个有 Symbol 为对象的 Key，value 为 symbol test 字符串的对象。test 的 #t 与 Symbol("t") 打印结果一致，#t 则与 Symbol("t') 是同一形式。</p>
<p data-nodeid="7227">在上面的代码示例中，两者的核心在使用上基本是一致的，只是在理解方面相对不一样。<strong data-nodeid="7321">Symbol 在 Dart 中是一种反射概念，而在 JavaScript 中则是创建唯一标识的概念。</strong></p>
<h4 data-nodeid="7228">Undefined 和 Null</h4>
<p data-nodeid="7229">由于 Dart 是静态脚本语言，因此在 Dart 中如果没有定义一个变量是无法通过编译的；而 JavaScript 是动态脚本语言，因此存在脚本在运行期间未定义的情况。所以这一点的不同决定了 Dart 在 Undefined 类型上与 JavaScript 的差异。</p>
<p data-nodeid="7230">null 在 Dart 中是的确存在的，官网上是这样解释的，null 是弱类型 object 的子类型，并非基础数据类型。所有数据类型，如果被初始化后没有赋值的话都将会被赋值 null 类型。</p>
<p data-nodeid="7231">下面的代码，首先定义了一个弱类型 number，其次定义了 int 类型的 num2，number 类型的 num1 以及 double 类型的 num3 ，最后我们打印出这些只定义了未被赋值的值。</p>
<pre class="lang-dart" data-nodeid="7232"><code data-language="dart"><span class="hljs-keyword">var</span> number;
<span class="hljs-built_in">int</span> num2;
<span class="hljs-built_in">num</span> num1;
<span class="hljs-built_in">double</span> num3;
<span class="hljs-built_in">print</span>(<span class="hljs-string">'number is var:<span class="hljs-subst">$number</span>,num2 is int:<span class="hljs-subst">$num2</span>,num2 is num:<span class="hljs-subst">$num1</span>,num3 is double:<span class="hljs-subst">$num3</span>'</span>);
</code></pre>
<p data-nodeid="7233">可以看到运行结果如下：</p>
<pre class="lang-dart" data-nodeid="7234"><code data-language="dart">flutter: number <span class="hljs-keyword">is</span> <span class="hljs-keyword">var</span>:<span class="hljs-keyword">null</span>,num2 <span class="hljs-keyword">is</span> <span class="hljs-built_in">int</span>:<span class="hljs-keyword">null</span>,num2 <span class="hljs-keyword">is</span> <span class="hljs-built_in">num</span>:<span class="hljs-keyword">null</span>,num3 <span class="hljs-keyword">is</span> <span class="hljs-built_in">double</span>:<span class="hljs-keyword">null</span>
</code></pre>
<p data-nodeid="7235">从运行结果我们可以看到，代码中声明了变量，但未赋值的变量在运行时都会被赋值为 null，这就是 Dart 中 null 类型存在的目的。</p>
<h4 data-nodeid="7236">Map 和 List</h4>
<p data-nodeid="7237">Map 和 List 与 JavaScript 中的 Array 和 Map 基本一致，但在 JavaScript 中不是基本数据类型，都属于引用数据类型。因此也就是分类不同，但在用法和类型上基本没有太大差异。</p>
<h4 data-nodeid="7238">弱类型（var、object 和 dynamic）</h4>
<p data-nodeid="7239">相对 JavaScript 而言，Dart 也存在弱类型（可以使用 var、object 和 dynamic 来声明），不过在这方面为了避免弱类型导致的客户端（App）Crash 的异常，Dart 还是对弱类型加强了校验。</p>
<p data-nodeid="7240">var 数据类型声明，第一次赋值时，将其数据类型绑定。下面代码使用 var 声明了一个弱类型 t，并赋值 String 类型 123，而接下来又对 t 进行其他类型的赋值。</p>
<pre class="lang-dart" data-nodeid="7241"><code data-language="dart"><span class="hljs-keyword">var</span> t = <span class="hljs-string">'123'</span>;
t = <span class="hljs-number">123</span>;
</code></pre>
<p data-nodeid="7242">这样的代码在 Dart 编译前就会报错，因为 t 在一次 var 赋值时就已经被绑定为 String 类型了，再进行赋值 Number 类型时就会报错。</p>
<pre class="lang-dart" data-nodeid="7243"><code data-language="dart">Assign value to <span class="hljs-keyword">new</span> local variable
</code></pre>
<p data-nodeid="7244">object 可以进行任何赋值，没有约束，这一点类似 JavaScript 中的 var 关键词赋值。在编译期，object 会对数据调用做一定的判断，并且报错。例如，声明时为 String 类型，但是在调用 length 时，编译期就会报错。如果数据来自接口层，则很容易导致运行时报错。因此这个要尽量减少使用，避免运行时报错导致客户端（App）Crash 的异常。</p>
<p data-nodeid="7245">dynamic 也是动态的数据类型，但如果数据类型调用异常，则只会在运行时报错，这点是非常危险的，因此在使用 dynamic 时要非常慎重。</p>
<h3 data-nodeid="7246">基础运算符</h3>
<p data-nodeid="7247">两种语言的基础运算符基本都一致。由于 Dart 是强数据类型，因此在 Dart 中没有 “=== ”的运算符。在 Dart 中有一些类型测试运算符，与 JavaScript 中的类型转换和 typeof 有点相似。</p>
<p data-nodeid="7248">这里也介绍一些 Dart 中比较简洁的写法：</p>
<ul data-nodeid="7249">
<li data-nodeid="7250">
<p data-nodeid="7251">?? 运算符，比如，t??'test' 是 t!= null ? t : 'test' 的缩写；</p>
</li>
<li data-nodeid="7252">
<p data-nodeid="7253">级联操作，允许对同一对象或者同一函数进行一系列操作，例如下面代码的 testObj 对象中有三个方法 add()、delete() 和 show()，应用级联操作可以依次进行调用。</p>
</li>
</ul>
<pre class="lang-dart" data-nodeid="7254"><code data-language="dart">testObj.add(<span class="hljs-string">'t'</span>)
..delete(<span class="hljs-string">'d'</span>)
..<span class="hljs-keyword">show</span>()
</code></pre>
<h3 data-nodeid="7255">函数</h3>
<p data-nodeid="7256">从我的理解来说，两者区别不大。箭头函数、函数闭包、匿名函数、高阶函数、参数可选等基本上都一样。在 Dart 中由于是强类型，因此在声明函数的时候可以增加一个返回类型，这点在 TypeScript 中的用法是一致的，对于前端开发人员来说，没有太多的差异点。</p>
<h3 data-nodeid="7257">类</h3>
<p data-nodeid="7258">类的概念在各种语言上大部分都是一致的，但在用法上可能存在差异，这里着重介绍一下 Dart 比较特殊的一些用法。</p>
<h4 data-nodeid="7259">命名构造函数</h4>
<p data-nodeid="7260">Dart 支持一个函数有多个构造函数，并且在实例化的时候可以选择不同的构造函数。</p>
<p data-nodeid="7261">下面的代码声明了一个 Dog 类，类中有一个 color 变量属性和两个构造函数。red 构造函数设置 Dog 类的 color 属性为 red，black 构造函数设置 Dog 类的 color 属性为 black。最后在 main 函数中分别用两个构造函数创建两个实例，并分别打印实例的 color 属性。</p>
<pre class="lang-dart" data-nodeid="7262"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Dog</span> </span>{
  <span class="hljs-built_in">String</span> color;
  Dog.red(){
    <span class="hljs-keyword">this</span>.color = <span class="hljs-string">'red'</span>;
  }

  Dog.black(){
    <span class="hljs-keyword">this</span>.color = <span class="hljs-string">'black'</span>;
  }
}
<span class="hljs-keyword">void</span> main(<span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; args) {
  Dog redDog = <span class="hljs-keyword">new</span> Dog.red();
  <span class="hljs-built_in">print</span>(redDog.color);

  Dog blackDog = <span class="hljs-keyword">new</span> Dog.black();
  <span class="hljs-built_in">print</span>(blackDog.color);
}
</code></pre>
<p data-nodeid="7263">运行代码后输出了两种颜色，即 red 和 black。就代码而言，我们可以应用同一个类不同的构造函数实现类不同场景下的实例化。</p>
<h4 data-nodeid="7264">访问控制</h4>
<p data-nodeid="7265">默认情况下都是 public，如果需要设置为私有属性，则在方法或者属性前使用 “_”。</p>
<h4 data-nodeid="7266">抽象类和泛型类</h4>
<p data-nodeid="7267">抽象类和其他语言的抽象类概念一样，这里在 JavaScript 中没有这种概念，因此这里稍微提及一下，主要是实现一个类被用于其他子类继承，抽象类是无法实例化的。</p>
<p data-nodeid="7268">下面的代码使用关键词 abstract 声明了一个有攻击性的武器抽象类，包含一个攻击函数和一个伤害力获取函数，Gun 和 BowAndArrow 都是继承抽象类，并需要实现抽象类中的方法。</p>
<pre class="lang-dart" data-nodeid="7269"><code data-language="dart"><span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AggressiveArms</span> </span>{
  attack();
  hurt()；
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Gun</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AggressiveArms</span> </span>{
  attack() {
    <span class="hljs-built_in">print</span>(<span class="hljs-string">"造成100点伤害"</span>);
  }
  hurt() {
    <span class="hljs-built_in">print</span>(<span class="hljs-string">"可以造成100点伤害"</span>);
  }
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BowAndArrow</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AggressiveArms</span> </span>{
  attack() {
    <span class="hljs-built_in">print</span>(<span class="hljs-string">"造成20点伤害"</span>);
  }
  hurt() {
    <span class="hljs-built_in">print</span>(<span class="hljs-string">"可以造成20点伤害"</span>);
  }
}
</code></pre>
<p data-nodeid="7270">泛型类，主要在不确定返回数据结构时使用，这点与 TypeScript 中的泛型概念一样。</p>
<p data-nodeid="9127">在下面的代码中，我们不确定数组中存储的类型是 int 还是 string，又或者是 bool，这时候可以使用泛型 来表示。在使用泛型类的时候可以将设定为自己需要的类型，比如下面的 string 调用和 int 调用。</p>
<pre class="lang-dart" data-nodeid="9128"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Array</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
  <span class="hljs-built_in">List</span> _list = <span class="hljs-keyword">new</span> <span class="hljs-built_in">List</span>&lt;T&gt;();
  Array();
  <span class="hljs-keyword">void</span> add&lt;T&gt;(T value) {
    <span class="hljs-keyword">this</span>._list.add(value);
  }
  <span class="hljs-keyword">get</span> value{
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>._list;
  }
}
<span class="hljs-keyword">void</span> main(<span class="hljs-built_in">List</span>&lt;<span class="hljs-built_in">String</span>&gt; args) {
  Array arr = <span class="hljs-keyword">new</span> Array&lt;<span class="hljs-built_in">String</span>&gt;();
  arr.add(<span class="hljs-string">'aa'</span>);
  arr.add(<span class="hljs-string">'bb'</span>);
  <span class="hljs-built_in">print</span>(arr.value);


  Array arr2 = <span class="hljs-keyword">new</span> Array&lt;<span class="hljs-built_in">int</span>&gt;();
  arr2.add(<span class="hljs-number">1</span>);
  arr2.add(<span class="hljs-number">2</span>);
  <span class="hljs-built_in">print</span>(arr2.value);
}
</code></pre>








<h3 data-nodeid="7273">库与调用</h3>
<h4 data-nodeid="7274">Dart 库管理</h4>
<p data-nodeid="7275">Dart 和 JavaScript 一样，有一个库管理资源（<a href="http://pub.dev" data-nodeid="7375">pub.dev</a>）。你可以在这里搜索找到你想要的一些库，接下来只要在 Dart 的配置文件 pubspec.yaml 中增加该库即可。这点类似于在 JavaScript 的 package.json 中增加声明一样，同样也有 dependencies 和 dev_dependencies。</p>
<p data-nodeid="7276">增加类似的数据配置，如下代码：</p>
<pre class="lang-yaml" data-nodeid="7277"><code data-language="yaml"><span class="hljs-attr">dependencies:</span>
  <span class="hljs-attr">cupertino_icons:</span> <span class="hljs-string">^0.1.2</span>
  <span class="hljs-attr">dio:</span> <span class="hljs-string">^3.0.4</span>
  <span class="hljs-attr">image_test_utils:</span> <span class="hljs-string">^1.0.0</span>
<span class="hljs-attr">dev_dependencies:</span>
  <span class="hljs-attr">flutter_test:</span>
    <span class="hljs-attr">sdk:</span> <span class="hljs-string">flutter</span>
</code></pre>
<h4 data-nodeid="7278">开发 Dart 库</h4>
<p data-nodeid="7279">Dart 也支持开发者自己开发一些库，并且发布到 pub.dev 上，这点基本上和 npm 管理一致，这里我只介绍 pub.dev 库的基本格式。</p>
<pre class="lang-yaml" data-nodeid="7280"><code data-language="yaml"><span class="hljs-string">dart_string_manip</span>
<span class="hljs-string">├──</span> <span class="hljs-string">example</span>
<span class="hljs-string">|</span>  <span class="hljs-string">└──</span> <span class="hljs-string">main.dart</span>
<span class="hljs-string">├──</span> <span class="hljs-string">lib</span>
<span class="hljs-string">|</span>  <span class="hljs-string">├──</span> <span class="hljs-string">dart_string_manip.dart</span>
<span class="hljs-string">|</span>  <span class="hljs-string">└──</span> <span class="hljs-string">src</span>
<span class="hljs-string">|</span>     <span class="hljs-string">├──</span> <span class="hljs-string">classes.dart</span>
<span class="hljs-string">|</span>     <span class="hljs-string">└──</span> <span class="hljs-string">functions.dart</span>
<span class="hljs-string">├──</span> <span class="hljs-string">.gitignore</span>
<span class="hljs-string">├──</span> <span class="hljs-string">.packages</span>
<span class="hljs-string">├──</span> <span class="hljs-string">LICENSE</span>
<span class="hljs-string">├──</span> <span class="hljs-string">README.md</span>
<span class="hljs-string">├──</span> <span class="hljs-string">pubspec.lock</span>
<span class="hljs-string">└──</span> <span class="hljs-string">pubspec.yaml</span>
</code></pre>
<p data-nodeid="7281">对于前端开发人员来说，这个结构和我们所看到的 npm 模块很相似，pubspec 和 package 很相似，核心是 lib 中的库名对应的库文件 .dart，该文件是一个 dart 类。类的概念上面已经介绍过了，将私有方法使用 "_" 保护，其他就可以被引用该库的模块调用，如果是自身库的一些实现逻辑，可以放在 src 中。</p>
<p data-nodeid="7282">开发完成该库以后，如果需要发布到 pub.dev，则可以参照<a href="https://flutter.dev/docs/development/packages-and-plugins/developing-packages" data-nodeid="7390">官网的说明</a>，按步骤进行即可。</p>
<h4 data-nodeid="7283">Dart 调用库</h4>
<p data-nodeid="7284">这里引入库的方式也与 ES6 的 import 语法很相似。先看看下面的一个例子，其目的是引入 pages 下的 homepage.dart 模块。</p>
<pre class="lang-dart" data-nodeid="7285"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'package:startup_namer/pages/homepage.dart'</span>;
</code></pre>
<p data-nodeid="7286">在上面的例子中，import 为关键词，package 为协议，可以使用 http 的方式，不过最好使用本地 package 方式，避免性能受影响。接下来的 startup_namer 为库名或者说是该项目名，pages 为 lib 下的一个文件夹，homepage.dart 则为具体需要引入的库文件名。</p>
<p data-nodeid="7287">当然这里也可以使用相对路径的方式，不过建议使用 package 的方式，以保持整个项目代码的一致性，因为对于第三方模块则必须使用 package 的方式。</p>
<h3 data-nodeid="7288">总结</h3>
<p data-nodeid="7289">本课时首先介绍了 Dart 基础数据类型、基础运算符、类以及库与调用。然后通过对比 JavaScript 的一些特殊差异性，来加深前端开发人员对 Dart 语言编程的理解。相信你通过本课时的学习，可以掌握 Dart 的编程，并且能够写一些 Dart 的第三方库。</p>
<p data-nodeid="7290">下一课时，我将介绍 Dart 的事件循环机制，掌握了其核心运行机制原理，才能编写出更高效、更有质量的代码。</p>
<p data-nodeid="7291">点击这里下载本课时源码，Flutter 专栏，源码地址：<a href="https://github.com/love-flutter/flutter-column" data-nodeid="7404">https://github.com/love-flutter/flutter-column</a></p>

---

### 精选评论

##### **涛：
> 我是学习iOS的没有JavaScript的基础,请问大神们&nbsp;<span style="font-size: 0.427rem;">class Array&lt;T&gt; {</span><div>&nbsp; List _list = new List&lt;T&gt;();</div><div>&nbsp; Array();</div><div>&nbsp; void add&lt;T&gt;(T value) {</div><div>&nbsp; &nbsp; this._list.add(value);</div><div>&nbsp; }</div><div>&nbsp; get value{</div><div>&nbsp; &nbsp; return this._list;</div><div>&nbsp; }</div><div>}</div><div>这段代码中&nbsp;Array(); &nbsp;什么含义? &nbsp;</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你是值括号里面的Array()的话，代表的是构造函数，我们定义了一个Array类，里面包含了一个构造函数。如果我们希望Array类有一些初始化的参数，可以在构造函数中设置初始值。例如这段代码：class Array<T> {
 List _list = new List<T>();
 Array(this._list);
 void add<T>(T value) {
  this._list.add(value);
 }
 get value{
  return this._list;
 }
}
void main() {
Array arr = new Array([1, 2]);
arr.add(1);
print(arr.value);
}

##### *瑞：
> 打卡1

##### **柳：
> 老师讲的很好，👍

##### **0390：
> 用var声明的变量不能进行隐式转换类型，为什么还说它是弱类型？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; var 类型可赋予不同类型的值，例如我们可以 var fl = ‘a’; 也可以 var flu = 10; 也可以 var flut = []; 你所看到的 var 并不能准确的表明该变量的数据类型。只是说 var 定义初始化了类型以后是不可更改的。

##### **8755：
> 开始咯😇

##### **的阿科：
> 本课时首先介绍了 Dart 基础数据类型、基础运算符、类以及库与调用。然后通过对比 JavaScript 的一些特殊差异性，来加深前端开发人员对 Dart 语言编程的理解。相信你通过本课时的学习，可以掌握 Dart 的编程，并且能够写一些 Dart 的第三方库。

##### **博：
> 老师，flutter为啥不选javascript作为编译语言？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 从历史原因讲，Dart 是 google 推出的，而 Flutter 也是 google自家产品。从区别上讲，在某些方面Javascript和Dart还是有区别，比如Javascript是动态脚本语言，而Dart可以支持AOT和JIT两种模式，从而在性能上有一定的优化提升，其次Javascript是弱类型语言，而Dart是强类型，等等，所以还是存在一些区别。

##### *晗：
> 这里的所有new 关键字是不是也可以省略了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 很棒，涉及到new的这部分都可以省略，一般情况下用了analyzer都会提示去掉new。

##### **玲：
> 泛型中加个Array()是啥意思？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个已经有同学问过，Array()是构造函数，具体可以看下上面评论回复中，我举的例子。

##### **涛：
> Dart中不是没有Array吗,而是用List代替,为什么在泛型那个例子中有class Array&lt;T&gt; {<div>}这样的代码</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里的是意思是定义一个Array类，可以用List来定义一个Array类。

##### *斌：
> 打卡打卡

##### **0508：
> <span style="font-size: 16.0125px;">Flutter和Dart是什么关系啊？</span>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 类似于React和Javascript的关系，flutter是技术框架，而dart是flutter中的编程语言

##### *铎：
> 打卡

##### *颖：
> Dart中symbol有哪些使用场景呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 通俗易懂一点说就是，symbol创建了一个唯一索引的内存地址，你可以通过内存地址访问到相应的。目前这个在开发应用过程基本不会使用到这个类型。在实际应用过程中，大家可以参考Javascript中的用法，核心还是在属性名的使用，就是使用Symbol类型作为属性，在需要重名属性名的情况下。其次在反射方面，在dart:mirrors必须传入Symbol类型，可以看下这个例子，https://www.tutorialspoint.com/dart_programming/dart_programming_symbol.htm。

##### **2833：
> 打卡

##### **民：
> 打卡

##### *涛：
> 从头学

##### **泽：
> <span style="font-size: 16.7811px;">老师 ，这个不透明类型是指什么？ 像oc也有好多opaque type ，</span><div><span style="color: rgb(51, 51, 51); font-family: &quot;SF Pro Display&quot;, &quot;SF Pro Icons&quot;, &quot;Helvetica Neue&quot;, Helvetica, Arial, sans-serif; font-size: 19px; letter-spacing: 0.342px;">An opaque type that represents a method in a class definition</span><br><div><span style="font-size: 16.7811px;">不明白什么</span></div></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不透明可以理解为，在计算机处理中是有一个内存地址，而如何将这个内存地址使用可见字符来描述，这就是Symbol做的一个桥梁。

