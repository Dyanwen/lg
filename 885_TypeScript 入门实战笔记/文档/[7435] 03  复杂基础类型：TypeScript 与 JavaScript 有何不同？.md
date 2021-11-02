<p data-nodeid="1409" class="">学习完原始类型等知识点，你可能已经对 TypeScript 有了基本的认知。在接下来这一讲中，我将带你接触稍微复杂一点的类型结构（比如数组、any 等比较难理解的特殊类型）及其使用场景。</p>
<blockquote data-nodeid="1410">
<p data-nodeid="1411">学习建议：请使用 VS Code 新建一个 03.Basic.2.ts 文件，然后尝试课程中的所有示例。</p>
</blockquote>
<h3 data-nodeid="1412">数组</h3>
<p data-nodeid="1413">因为 TypeScript 的数组和元组转译为 JavaScript 后都是数组，所以这里我们把数组和元组这两个类型整合到一起介绍，也方便你更好地对比学习。</p>
<h4 data-nodeid="1414">数组类型（Array）</h4>
<p data-nodeid="1415">在 TypeScript 中，我们也可以像 JavaScript 一样定义数组类型，并且指定数组元素的类型。</p>
<p data-nodeid="1416">首先，我们可以直接使用 [] 的形式定义数组类型，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1417"><code data-language="typescript"><span class="hljs-comment">/** 子元素是数字类型的数组 */</span>
<span class="hljs-keyword">let</span> arrayOfNumber: <span class="hljs-built_in">number</span>[] = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>];
<span class="hljs-comment">/** 子元素是字符串类型的数组 */</span>
<span class="hljs-keyword">let</span> arrayOfString: <span class="hljs-built_in">string</span>[] = [<span class="hljs-string">'x'</span>, <span class="hljs-string">'y'</span>, <span class="hljs-string">'z'</span>];
</code></pre>
<p data-nodeid="1418">同样，我们也可以使用 Array 泛型（在第 10 讲会详细介绍泛型）定义数组类型，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1419"><code data-language="typescript"><span class="hljs-comment">/** 子元素是数字类型的数组 */</span>
<span class="hljs-keyword">let</span> arrayOfNumber: <span class="hljs-built_in">Array</span>&lt;<span class="hljs-built_in">number</span>&gt; = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>];
<span class="hljs-comment">/** 子元素是字符串类型的数组 */</span>
<span class="hljs-keyword">let</span> arrayOfString: <span class="hljs-built_in">Array</span>&lt;<span class="hljs-built_in">string</span>&gt; = [<span class="hljs-string">'x'</span>, <span class="hljs-string">'y'</span>, <span class="hljs-string">'z'</span>];
</code></pre>
<p data-nodeid="1420">以上两种定义数组类型的方式虽然本质上没有任何区别，但是我更推荐使用 [] 这种形式来定义。<strong data-nodeid="1558">一方面可以避免与 JSX 的语法冲突，另一方面可以减少不少代码量</strong>。</p>
<p data-nodeid="1421">如果我们明确指定了数组元素的类型，以下所有操作都将因为不符合类型约定而提示错误。</p>
<pre class="lang-typescript" data-nodeid="1422"><code data-language="typescript"><span class="hljs-keyword">let</span> arrayOfNumber: <span class="hljs-built_in">number</span>[] = [<span class="hljs-string">'x'</span>, <span class="hljs-string">'y'</span>, <span class="hljs-string">'z'</span>]; <span class="hljs-comment">// 提示 ts(2322)</span>
arrayOfNumber[<span class="hljs-number">3</span>] = <span class="hljs-string">'a'</span>; <span class="hljs-comment">// 提示 ts(2322)</span>
arrayOfNumber.push(<span class="hljs-string">'b'</span>); <span class="hljs-comment">// 提示 ts(2345)</span>
<span class="hljs-keyword">let</span> arrayOfString: <span class="hljs-built_in">string</span>[] = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>]; <span class="hljs-comment">// 提示 ts(2322)</span>
arrayOfString[<span class="hljs-number">3</span>] = <span class="hljs-number">1</span>; <span class="hljs-comment">// 提示 ts(2322)</span>
arrayOfString.push(<span class="hljs-number">2</span>); <span class="hljs-comment">// 提示 ts(2345)</span>
</code></pre>
<h4 data-nodeid="1423">元组类型（Tuple）</h4>
<p data-nodeid="1424">元组最重要的特性是可以限制数组元素的个数和类型，它特别适合用来实现多值返回。</p>
<p data-nodeid="1425">我们熟知的一个使用元组的场景是 React Hooks（关于 React Hooks 的简介<a href="https://reactjs.org/docs/hooks-intro.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1565">请点击这里查看</a>），例如 useState 示例：</p>
<pre class="lang-typescript" data-nodeid="1426"><code data-language="typescript"><span class="hljs-keyword">import</span> { useState } <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">useCount</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> [count, setCount] = useState(<span class="hljs-number">0</span>);
  <span class="hljs-keyword">return</span> ....;
}
</code></pre>
<p data-nodeid="1427">在 JavaScript 中并没有元组的概念，作为一门动态类型语言，它的优势是<strong data-nodeid="1572">天然支持多类型元素数组</strong>。</p>
<p data-nodeid="1428">我们假设以下两个数组的元素类型如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1429"><code data-language="typescript">[state, setState]
[setState, state]
</code></pre>
<p data-nodeid="1430">从上面可以看出，state 是一个类型为 State 的对象，而 setState 是一个类型为 SetState 的函数。</p>
<p data-nodeid="1431"><strong data-nodeid="1578">注意：这里我们用全小写表示值，首字母大写表示（TypeScript）类型。</strong></p>
<p data-nodeid="1432">对于 JavaScript 而言，上面的数组其实长的都一样，并没有一个有效的途径可以区分彼此。</p>
<p data-nodeid="1433">不过，出于较好的扩展性、可读性和稳定性考虑，我们往往会更偏向于<strong data-nodeid="1585">把不同类型的值通过键值对的形式塞到一个对象中，再返回这个对象</strong>（尽管这样会增加代码量），而不是使用没有任何限制的数组。比如我们可能会使用如下的对象结构来替换数组：</p>
<pre class="lang-typescript" data-nodeid="1434"><code data-language="typescript">{
  state,
  setState
}
</code></pre>
<p data-nodeid="1435">而 TypeScript 的元组类型正好弥补了这个不足，使得定义包含固定个数元素、每个元素类型未必相同的数组成为可能。（需要注意的是，毕竟 TypeScript 会转译成 JavaScript，所以 TypeScript 的元组无法在运行时约束所谓的“元组”像真正的元组一样，保证元素类型、长度不可变更）。</p>
<p data-nodeid="1436">对于 TypeScript 而言，如下所示的两个元组类型其实并不相同：</p>
<pre class="lang-typescript" data-nodeid="1437"><code data-language="typescript">[State, SetState]
[SetState, State]
</code></pre>
<p data-nodeid="1438">所以添加了不同元组类型注解的数组后，在 TypeScript 静态类型检测层面就变成了两个不相同的元组，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1439"><code data-language="typescript"><span class="hljs-keyword">const</span> x: [State, SetState] = [state, setState];
<span class="hljs-keyword">const</span> y: [SetState, State] = [setState, state];
</code></pre>
<p data-nodeid="1440">下面我们还是使用所熟知的 React Hooks 来介绍 TypeScript 元组的应用场景。</p>
<p data-nodeid="1441">比如 useState 的返回值类型是一个元组类型，如下代码所示（以下仅是简单的例子，事实上 useState 的类型定义更为复杂）：</p>
<pre class="lang-typescript" data-nodeid="1442"><code data-language="typescript">(state: State) =&gt; [State, SetState]
</code></pre>
<p data-nodeid="1443">元组相较对象而言，不仅为我们实现解构赋值提供了极大便利，还减少了不少代码量，这可能也是 React 官方如此设计核心 Hooks 的重要原因之一。</p>
<p data-nodeid="1444">但事实上，许多第三方的 Hooks 往往会出于扩展性、稳定性等考虑，尤其是需要返回的值的个数超过 2 个时，会更偏向于使用对象作为返回值。</p>
<blockquote data-nodeid="1445">
<p data-nodeid="1446">这里需要注意：数组类型的值只有显示添加了元组类型注解后（或者使用 as const，声明为只读元组），TypeScript 才会把它当作元组，否则推荐出来的类型就是普通的数组类型（第 4 讲会介绍类型推断）。</p>
</blockquote>
<p data-nodeid="1447">相对于以上熟悉的 JavaScript 一般味道的数组类型，接下来我们将介绍几种不一样且需要费点心力理解的类型——特殊类型（这是并不是 TypeScript 官方的定义，这么划分是为了更好地组织知识点）。</p>
<h3 data-nodeid="1448">特殊类型</h3>
<h4 data-nodeid="1449">1. any</h4>
<p data-nodeid="1450">any 指的是一个任意类型，它是官方提供的一个选择性绕过静态类型检测的作弊方式。</p>
<p data-nodeid="1451">我们可以对被注解为 any 类型的变量进行任何操作，包括获取事实上并不存在的属性、方法，并且 TypeScript 还无法检测其属性是否存在、类型是否正确。</p>
<p data-nodeid="1452">比如我们可以把任何类型的值赋值给 any 类型的变量，也可以把 any 类型的值赋值给任意类型（除 never 以外）的变量，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1453"><code data-language="typescript"><span class="hljs-keyword">let</span> anything: <span class="hljs-built_in">any</span> = {};
anything.doAnything(); <span class="hljs-comment">// 不会提示错误</span>
anything = <span class="hljs-number">1</span>; <span class="hljs-comment">// 不会提示错误</span>
anything = <span class="hljs-string">'x'</span>; <span class="hljs-comment">// 不会提示错误</span>
<span class="hljs-keyword">let</span> num: <span class="hljs-built_in">number</span> = anything;&nbsp;<span class="hljs-comment">// 不会提示错误</span>
<span class="hljs-keyword">let</span> str: <span class="hljs-built_in">string</span> = anything;&nbsp;<span class="hljs-comment">// 不会提示错误</span>
</code></pre>
<p data-nodeid="1454">如果我们不想花费过高的成本为复杂的数据添加类型注解，或者已经引入了缺少类型注解的第三方组件库，这时就可以把这些值全部注解为 any 类型，并告诉 TypeScript 选择性地忽略静态类型检测。</p>
<p data-nodeid="1455">尤其是在将一个基于 JavaScript 的应用改造成 TypeScript 的过程中，我们不得不借助 any 来选择性添加和忽略对某些 JavaScript 模块的静态类型检测，直至逐步替换掉所有的 JavaScript。</p>
<p data-nodeid="1456">any 类型会在对象的调用链中进行传导，即所有 any 类型的任意属性的类型都是 any，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1457"><code data-language="typescript"><span class="hljs-keyword">let</span> anything: <span class="hljs-built_in">any</span> = {};
<span class="hljs-keyword">let</span> z = anything.x.y.z; <span class="hljs-comment">// z 类型是 any，不会提示错误</span>
z(); <span class="hljs-comment">// 不会提示错误</span>
</code></pre>
<p data-nodeid="1458">这里我们需要明白且记住：<strong data-nodeid="1610">Any is Hell（Any 是地狱）</strong>。</p>
<p data-nodeid="1459">从长远来看，使用 any 绝对是一个坏习惯。如果一个 TypeScript 应用中充满了 any，此时静态类型检测基本起不到任何作用，也就是说与直接使用 JavaScript 没有任何区别。<strong data-nodeid="1615">因此，除非有充足的理由，否则我们应该尽量避免使用 any ，并且开启禁用隐式 any 的设置。</strong></p>
<h4 data-nodeid="1460">2. unknown</h4>
<p data-nodeid="1461">unknown 是 TypeScript 3.0 中添加的一个类型，它主要用来描述类型并不确定的变量。</p>
<p data-nodeid="1462">比如在多个 if else 条件分支场景下，它可以用来接收不同条件下类型各异的返回值的临时变量，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1463"><code data-language="typescript"><span class="hljs-keyword">let</span> result: unknown;
<span class="hljs-keyword">if</span> (x) {
  result = x();
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (y) {
  result = y();
} ...
</code></pre>
<p data-nodeid="1464">在 3.0 以前的版本中，只有使用 any 才能满足这种动态类型场景。</p>
<p data-nodeid="1465">与 any 不同的是，unknown 在类型上更安全。比如我们可以将任意类型的值赋值给 unknown，但 unknown 类型的值只能赋值给 unknown 或 any，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1466"><code data-language="typescript"><span class="hljs-keyword">let</span> result: unknown;
<span class="hljs-keyword">let</span> num: <span class="hljs-built_in">number</span> = result; <span class="hljs-comment">// 提示 ts(2322)</span>
<span class="hljs-keyword">let</span> anything: <span class="hljs-built_in">any</span> = result; <span class="hljs-comment">// 不会提示错误</span>
</code></pre>
<p data-nodeid="1467">使用 unknown 后，TypeScript 会对它做类型检测。但是，如果不缩小类型（Type Narrowing），我们对 unknown 执行的任何操作都会出现如下所示错误：</p>
<pre class="lang-typescript" data-nodeid="1468"><code data-language="typescript"><span class="hljs-keyword">let</span> result: unknown;
result.toFixed(); <span class="hljs-comment">// 提示 ts(2571)</span>
</code></pre>
<p data-nodeid="1469"><strong data-nodeid="1628">而所有的类型缩小手段对 unknown 都有效</strong>，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1470"><code data-language="typescript"><span class="hljs-keyword">let</span> result: unknown;
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">typeof</span> result === <span class="hljs-string">'number'</span>) {
  result.toFixed(); <span class="hljs-comment">// 此处 hover result 提示类型是 number，不会提示错误</span>
}
</code></pre>
<h4 data-nodeid="1471">3. void、undefined、null</h4>
<p data-nodeid="1472">考虑再三，我们还是决定把 void、undefined 和 null “三废柴”特殊类型整合到一起介绍。</p>
<p data-nodeid="1473">依照官方的说法，它们实际上并没有太大的用处，尤其是在本专栏中强烈推荐并要求的 strict 模式下，它们是名副其实的“废柴”。</p>
<p data-nodeid="1474">首先我们来说一下 void 类型，它仅适用于表示没有返回值的函数。即如果该函数没有返回值，那它的类型就是 void。</p>
<p data-nodeid="1475">在 strict 模式下，声明一个 void 类型的变量几乎没有任何实际用处，因为我们不能把 void 类型的变量值再赋值给除了 any 和 unkown 之外的任何类型变量。</p>
<p data-nodeid="1476">然后我们说说 undefined 类型 和 null 类型，它们是 TypeScript 值与类型关键字同名的唯二例外。但这并不影响它们被称为“废柴”，因为单纯声明 undefined 或者 null 类型的变量也是无比鸡肋，示例如下所示：</p>
<pre class="lang-typescript" data-nodeid="1477"><code data-language="typescript"><span class="hljs-keyword">let</span> undeclared: <span class="hljs-literal">undefined</span> = <span class="hljs-literal">undefined</span>; <span class="hljs-comment">// 鸡肋</span>
<span class="hljs-keyword">let</span> nullable: <span class="hljs-literal">null</span> = <span class="hljs-literal">null</span>; <span class="hljs-comment">// 鸡肋</span>
</code></pre>
<p data-nodeid="1478">undefined 的最大价值主要体现在接口类型（第 7 讲会涉及）上，它表示一个可缺省、未定义的属性。</p>
<p data-nodeid="1479">这里分享一个稍微有点费解的设计：<strong data-nodeid="1642">我们可以把 undefined 值或类型是 undefined 的变量赋值给 void 类型变量，反过来，类型是 void 但值是 undefined 的变量不能赋值给 undefined 类型。</strong></p>
<pre class="lang-typescript" data-nodeid="1480"><code data-language="typescript"><span class="hljs-keyword">const</span> userInfo: {
  id?: <span class="hljs-built_in">number</span>;
} = {};
<span class="hljs-keyword">let</span> undeclared: <span class="hljs-literal">undefined</span> = <span class="hljs-literal">undefined</span>;
<span class="hljs-keyword">let</span> unusable: <span class="hljs-built_in">void</span> = <span class="hljs-literal">undefined</span>;
unusable = undeclared; <span class="hljs-comment">// ok</span>
undeclared = unusable; <span class="hljs-comment">// ts(2322)</span>
</code></pre>
<p data-nodeid="1481">而 null 的价值我认为主要体现在接口制定上，它表明对象或属性可能是空值。尤其是在前后端交互的接口，比如 Java Restful、Graphql，任何涉及查询的属性、对象都可能是 null 空对象，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1482"><code data-language="typescript"><span class="hljs-keyword">const</span> userInfo: {
  name: <span class="hljs-literal">null</span> | <span class="hljs-built_in">string</span>
} = { name: <span class="hljs-literal">null</span> };
</code></pre>
<p data-nodeid="1483">除此之外，undefined 和 null 类型还具备警示意义，它们可以提醒我们针对可能操作这两种（类型）值的情况做容错处理。</p>
<p data-nodeid="1484">我们需要类型守卫（Type Guard，<strong data-nodeid="1650">第 11 讲会专门讲解</strong>）在操作之前判断值的类型是否支持当前的操作。类型守卫既能通过类型缩小影响 TypeScript 的类型检测，也能保障 JavaScript 运行时的安全性，如下代码所示：</p>
<pre class="lang-typescript te-preview-highlight" data-nodeid="5286"><code data-language="typescript"><span class="hljs-keyword">const</span> userInfo: {
  id?: <span class="hljs-built_in">number</span>;
  name?: <span class="hljs-literal">null</span> | <span class="hljs-built_in">string</span>
} = { id: <span class="hljs-number">1</span>, name: <span class="hljs-string">'Captain'</span> };
<span class="hljs-keyword">if</span> (userInfo.id !== <span class="hljs-literal">undefined</span>) { <span class="hljs-comment">// Type Guard</span>
  userInfo.id.toFixed(); <span class="hljs-comment">// id 的类型缩小成 number</span>
}
</code></pre>






<p data-nodeid="1486">我们不建议随意使用非空断言（下面要讲的“类型断言”中会详细介绍非空断言）来排除值可能为 null 或 undefined 的情况，因为这样很不安全。</p>
<pre class="lang-typescript" data-nodeid="1487"><code data-language="typescript">userInfo.id!.toFixed(); <span class="hljs-comment">// ok，但不建议</span>
userInfo.name!.toLowerCase() <span class="hljs-comment">// ok，但不建议</span>
</code></pre>
<p data-nodeid="1488">而比非空断言更安全、类型守卫更方便的做法是使用单问号（Optional Chain）、双问号（空值合并），我们可以使用它们来保障代码的安全性，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1489"><code data-language="typescript">userInfo.id?.toFixed(); <span class="hljs-comment">// Optional Chain</span>
<span class="hljs-keyword">const</span> myName = userInfo.name?? <span class="hljs-string">`my name is <span class="hljs-subst">${info.name}</span>`</span>; <span class="hljs-comment">// 空值合并</span>
</code></pre>
<h4 data-nodeid="1490">4. never</h4>
<p data-nodeid="1491">never 表示永远不会发生值的类型，这里我们举一个实际的场景进行说明。</p>
<p data-nodeid="1492">首先，我们定义一个统一抛出错误的函数，代码示例如下（圆括号后 : + 类型注解 表示函数返回值的类型，关于函数类型我们会在后续 <strong data-nodeid="1662">“第 5 讲：函数类型”详细讲解</strong>）：</p>
<pre class="lang-typescript" data-nodeid="1493"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ThrowError</span>(<span class="hljs-params">msg: <span class="hljs-built_in">string</span></span>): <span class="hljs-title">never</span> </span>{
  <span class="hljs-keyword">throw</span> <span class="hljs-built_in">Error</span>(msg);
}
</code></pre>
<p data-nodeid="1494">以上函数因为永远不会有返回值，所以它的返回值类型就是 never。</p>
<p data-nodeid="1495">同样，如果函数代码中是一个死循环，那么这个函数的返回值类型也是 never，如下代码所示。</p>
<pre class="lang-typescript" data-nodeid="1496"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">InfiniteLoop</span>(<span class="hljs-params"></span>): <span class="hljs-title">never</span> </span>{
  <span class="hljs-keyword">while</span> (<span class="hljs-literal">true</span>) {}
}
</code></pre>
<p data-nodeid="1497">never 是所有类型的子类型，它可以给所有类型赋值，如下代码所示。</p>
<pre class="lang-typescript" data-nodeid="1498"><code data-language="typescript"><span class="hljs-keyword">let</span> Unreachable: never = <span class="hljs-number">1</span>; <span class="hljs-comment">// ts(2322)</span>
Unreachable = <span class="hljs-string">'string'</span>; <span class="hljs-comment">// ts(2322)</span>
Unreachable = <span class="hljs-literal">true</span>; <span class="hljs-comment">// ts(2322)</span>
<span class="hljs-keyword">let</span> num: <span class="hljs-built_in">number</span> = Unreachable; <span class="hljs-comment">// ok</span>
<span class="hljs-keyword">let</span> str: <span class="hljs-built_in">string</span> = Unreachable;&nbsp;<span class="hljs-comment">// ok</span>
<span class="hljs-keyword">let</span> bool: <span class="hljs-built_in">boolean</span> = Unreachable;&nbsp;<span class="hljs-comment">// ok</span>
</code></pre>
<p data-nodeid="1499">但是反过来，除了 never 自身以外，其他类型（包括 any 在内的类型）都不能为 never 类型赋值。</p>
<p data-nodeid="1500">在恒为 false 的类型守卫条件判断下，变量的类型将缩小为 never（never 是所有其他类型的子类型，所以是类型缩小为 never，而不是变成 never）。因此，条件判断中的相关操作始终会报无法更正的错误（我们可以把这理解为一种基于静态类型检测的 Dead Code 检测机制），如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1501"><code data-language="typescript"><span class="hljs-keyword">const</span> str: <span class="hljs-built_in">string</span> = <span class="hljs-string">'string'</span>;
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">typeof</span> str === <span class="hljs-string">'number'</span>) {
  str.toLowerCase(); <span class="hljs-comment">// Property 'toLowerCase' does not exist on type 'never'.ts(2339)</span>
}
</code></pre>
<p data-nodeid="1502">基于 never 的特性，我们还可以使用 never 实现一些有意思的功能。比如我们可以把 never 作为接口类型下的属性类型，用来禁止写接口下特定的属性，示例代码如下：</p>
<pre class="lang-typescript" data-nodeid="1503"><code data-language="typescript"><span class="hljs-keyword">const</span> props: {
  id: <span class="hljs-built_in">number</span>,
  name?: never
} = {
  id: <span class="hljs-number">1</span>
}
props.name = <span class="hljs-literal">null</span>; <span class="hljs-comment">// ts(2322))</span>
props.name = <span class="hljs-string">'str'</span>; <span class="hljs-comment">// ts(2322)</span>
props.name = <span class="hljs-number">1</span>; <span class="hljs-comment">// ts(2322)</span>
</code></pre>
<p data-nodeid="1504">此时，无论我们给 props.name 赋什么类型的值，它都会提示类型错误，实际效果等同于 name 只读 。</p>
<h4 data-nodeid="1505">5. object</h4>
<p data-nodeid="1506">object 类型表示非原始类型的类型，即非&nbsp;number、string、boolean、bigint、symbol、null、undefined 的类型。然而，它也是个没有什么用武之地的类型，如下所示的一个应用场景是用来表示 Object.create 的类型。</p>
<pre class="lang-typescript" data-nodeid="1507"><code data-language="typescript"><span class="hljs-keyword">declare</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">create</span>(<span class="hljs-params">o: object | <span class="hljs-literal">null</span></span>): <span class="hljs-title">any</span></span>;
create({}); <span class="hljs-comment">// ok</span>
create(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">null</span>); <span class="hljs-comment">// ok</span>
create(<span class="hljs-number">2</span>); <span class="hljs-comment">// ts(2345)</span>
create(<span class="hljs-string">'string'</span>); <span class="hljs-comment">// ts(2345)</span>
</code></pre>
<h3 data-nodeid="1508">类型断言（Type Assertion）</h3>
<p data-nodeid="1509">TypeScript 类型检测无法做到绝对智能，毕竟程序不能像人一样思考。有时会碰到我们比 TypeScript 更清楚实际类型的情况，比如下面的例子：</p>
<pre class="lang-typescript" data-nodeid="1510"><code data-language="typescript"><span class="hljs-keyword">const</span> arrayNumber: <span class="hljs-built_in">number</span>[] = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>];
<span class="hljs-keyword">const</span> greaterThan2: <span class="hljs-built_in">number</span> = arrayNumber.find(<span class="hljs-function"><span class="hljs-params">num</span> =&gt;</span> num &gt; <span class="hljs-number">2</span>); <span class="hljs-comment">// 提示 ts(2322)</span>
</code></pre>
<p data-nodeid="1511">其中，greaterThan2 一定是一个数字（确切地讲是 3），因为 arrayNumber 中明显有大于 2 的成员，但静态类型对运行时的逻辑无能为力。</p>
<p data-nodeid="1512">在 TypeScript 看来，greaterThan2 的类型既可能是数字，也可能是 undefined，所以上面的示例中提示了一个 ts(2322) 错误，此时我们不能把类型 undefined 分配给类型 number。</p>
<p data-nodeid="1513">不过，我们可以使用一种笃定的方式——<strong data-nodeid="1683">类型断言</strong>（类似仅作用在类型层面的强制类型转换）告诉 TypeScript 按照我们的方式做类型检查。</p>
<p data-nodeid="1514">比如，我们可以使用 as 语法做类型断言，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1515"><code data-language="typescript"><span class="hljs-keyword">const</span> arrayNumber: <span class="hljs-built_in">number</span>[] = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>];
<span class="hljs-keyword">const</span> greaterThan2: <span class="hljs-built_in">number</span> = arrayNumber.find(<span class="hljs-function"><span class="hljs-params">num</span> =&gt;</span> num &gt; <span class="hljs-number">2</span>) <span class="hljs-keyword">as</span> <span class="hljs-built_in">number</span>;
</code></pre>
<p data-nodeid="1516">又或者是使用尖括号 + 类型的格式做类型断言，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="1517"><code data-language="typescript"><span class="hljs-keyword">const</span> arrayNumber: <span class="hljs-built_in">number</span>[] = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>];
<span class="hljs-keyword">const</span> greaterThan2: <span class="hljs-built_in">number</span> = &lt;<span class="hljs-built_in">number</span>&gt;arrayNumber.find(<span class="hljs-function"><span class="hljs-params">num</span> =&gt;</span> num &gt; <span class="hljs-number">2</span>);
</code></pre>
<p data-nodeid="1518">以上两种方式虽然没有任何区别，但是尖括号格式会与 JSX 产生语法冲突，因此我们更推荐使用 as 语法。</p>
<blockquote data-nodeid="1519">
<p data-nodeid="1520">注意：类型断言的操作对象必须满足某些约束关系，否则我们将得到一个 ts(2352) 错误，即从类型“源类型”到类型“目标类型”的转换是错误的，因为这两种类型不能充分重叠。</p>
</blockquote>
<p data-nodeid="1521">我一度喜欢用“指鹿为马”来形容类型断言，但其实也不够准确。</p>
<p data-nodeid="1522">从物种类型上看，鹿和马肯定不能转换，虽然它们都是动物（继承自同一个父类），但是鹿有“角属性”，马有“鬃毛属性”，所以两者不能充分重叠。</p>
<p data-nodeid="1523"><strong data-nodeid="1693">如果我们把它换成“指白马为马”“指马为白马”，就可以很贴切地体现类型断言的约束条件：父子、子父类型之间可以使用类型断言进行转换。</strong></p>
<blockquote data-nodeid="1524">
<p data-nodeid="1525"><strong data-nodeid="1722">注意</strong>：这个结论完全适用于复杂类型，但是对于 number、string、boolean 原始类型来说，不仅父子类型可以相互断言，父类型相同的类型也可以相互断言，比如 1 as 2、'a' as 'b'、true as false（这里的 2、'b'、false 被称之为字面量类型，在第 4 讲里会详细介绍），反过来 2 as 1、'b' as 'a'、false as true 也是被允许的（这里的 1、'a'、true 是字面量类型），尽管这样的断言没有任何意义。</p>
</blockquote>
<p data-nodeid="1526">另外，any 和 unknown 这两个特殊类型属于万金油，因为它们既可以被断言成任何类型，反过来任何类型也都可以被断言成 any 或 unknown。因此，如果我们想强行“指鹿为马”，就可以先把“鹿”断言为 any 或 unknown，然后再把 any 和 unknown 断言为“马”，比如鹿 as any as 马。</p>
<p data-nodeid="1527">我们除了可以把特定类型断言成符合约束添加的其他类型之外，还可以使用“字面量值 + as const”语法结构进行常量断言，具体示例如下所示：</p>
<pre class="lang-typescript" data-nodeid="1528"><code data-language="typescript"><span class="hljs-comment">/** str 类型是 '"str"' */</span>
<span class="hljs-keyword">let</span> str = <span class="hljs-string">'str'</span> <span class="hljs-keyword">as</span> <span class="hljs-keyword">const</span>;
<span class="hljs-comment">/** readOnlyArr 类型是 'readonly [0, 1]' */</span>
<span class="hljs-keyword">const</span> readOnlyArr = [<span class="hljs-number">0</span>, <span class="hljs-number">1</span>] <span class="hljs-keyword">as</span> <span class="hljs-keyword">const</span>;
</code></pre>
<p data-nodeid="1529">常量断言所涉及的字面量（字面量即代码中，比如 '"str"'、'1'、'true'、'{}'）与字面量类型相关的知识点将在 <strong data-nodeid="1748">“第 03 讲：字面量类型”</strong> 中详细讲解，这里我们就不对实例代码做原理解析了。你可以保持着好奇心，期待后续内容。</p>
<p data-nodeid="1530">此外还有一种特殊非空断言，即在值（变量、属性）的后边添加 '!' 断言操作符，它可以用来排除值为 null、undefined 的情况，具体示例如下：</p>
<pre class="lang-typescript" data-nodeid="1531"><code data-language="typescript"><span class="hljs-keyword">let</span> mayNullOrUndefinedOrString: <span class="hljs-literal">null</span> | <span class="hljs-literal">undefined</span> | <span class="hljs-built_in">string</span>;
mayNullOrUndefinedOrString!.toString(); <span class="hljs-comment">// ok</span>
mayNullOrUndefinedOrString.toString(); <span class="hljs-comment">// ts(2531)</span>
</code></pre>
<p data-nodeid="1532">对于非空断言来说，我们同样应该把它视作和 any 一样危险的选择。</p>
<p data-nodeid="1533">在复杂应用场景中，如果我们使用非空断言，就无法保证之前一定非空的值，比如页面中一定存在 id 为 feedback 的元素，数组中一定有满足 &gt; 2 条件的数字，这些都不会被其他人改变。而一旦保证被改变，错误只会在运行环境中抛出，而静态类型检测是发现不了这些错误的。</p>
<p data-nodeid="1534">所以，我们建议使用类型守卫（更多讲解，见“第 11 讲：类型守卫”）来代替非空断言，比如如下所示的条件判断：</p>
<pre class="lang-typescript" data-nodeid="1535"><code data-language="typescript"><span class="hljs-keyword">let</span> mayNullOrUndefinedOrString: <span class="hljs-literal">null</span> | <span class="hljs-literal">undefined</span> | <span class="hljs-built_in">string</span>;
<span class="hljs-keyword">if</span> (<span class="hljs-keyword">typeof</span> mayNullOrUndefinedOrString === <span class="hljs-string">'string'</span>) {
  mayNullOrUndefinedOrString.toString(); <span class="hljs-comment">// ok</span>
}
</code></pre>
<h3 data-nodeid="1536">小结与预告</h3>
<p data-nodeid="1537">到这里，TypeScript 所有的基础类型就交代完了，你需要反复消化，夯实基础，为 04讲将要接触的稍微复杂的类型和应用场景做好准备。</p>
<p data-nodeid="1538" class="">这里插播一个思考题：类型断言需要满足什么约束条件？欢迎你在留言区与我进行互动、交流。另外，如果你觉得本专栏有价值，欢迎分享给更多的好友哦~</p>

---

### 精选评论

##### **其：
> ts 自动推断可能是空，但是实际可以确保确实存在，比如dom元素或者已知一个列表某项的ID，需要从列表里找到这个数据的时候(分别对应使用querySelector 和find)，可以使用断言

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，如果特别笃定，可以这么做。当然最佳实践，肯定是加一层 if 条件包裹（类型守卫），会更安全。

##### **用户0571：
> 类型断言，用于给告诉TypeScript某个值你非常确定是你断言的类型，而不是TS推测出来的类型。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的！比如在获取一个 DOM 元素时，推断出来的类型是 xxElement | null，但是你非常笃定元素一定存在，这个时候就可以使用类型断言，as xxElement。

##### **生：
> const">arrayNumber:">number[]">1,">2,">3,">4];const">greaterThan2:">number">arrayNumber.find(num">=">num">2);运行这两个代码提示ts(7006),并没有提示ts(2322)呀？
PS:我的ts版本是3.9.9

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是有其他信息干扰了。在严格模式下，因为目标数组 find 方法的返回值是 number | undefined，赋值给 number 类型变量自然会提示 ts(2322) 错误。

##### *一：
> 是不是在给一个变量赋值的时候，最后类型判断之后类型都会是对应的类型和undefined？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果 TS 编译选项设置了 --strictNullChecks true，那么 undefined 也是一个单独的类型，所以最后类型判断都只会是对应的类型。不设置或者设置为 false，就会出现你这种情况

##### **森：
> 单问号（Optional Chain）、双问号（空值合并）不懂这两个用法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这都是为了使代码看起来更加简洁，当值可能为 undefined 和 null 时，这样可以使得代码操作安全，也更加简洁。

1. Optional chaining: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Optional_chaining

2. Nullish coalescing operator: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator

##### **斌：
> 前提已经明确有哪些类型，且后续的操作类型符合已知类型，就可以进行断言

##### **雨：
> 父子，子父之间可以进行类型断言

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这条规则基本可以适用于绝大大部分复杂类型结构的可断言规则。

