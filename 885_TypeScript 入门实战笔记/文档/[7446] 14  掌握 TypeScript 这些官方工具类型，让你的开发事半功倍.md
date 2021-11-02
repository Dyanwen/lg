<p data-nodeid="11916">在 13 讲中我们学习了如何增强 TypeScript 类型系统，这一讲将继续深入了解 TypeScript 官方提供的全局工具类型。</p>


<p data-nodeid="11267">在 TypeScript 中提供了许多自带的工具类型，因为这些类型都是全局可用的，所以无须导入即可直接使用。了解了基础的工具类型后，我们不仅知道 TypeScript 如何利用前几讲介绍的基础类型知识实现这些工具类型，还知道如何更好地利用这些基础类型，以免重复造轮子，并能通过这些工具类型实现更复杂的类型。</p>
<p data-nodeid="11268">根据使用范围，我们可以将工具类型划分为操作接口类型、联合类型、函数类型、字符串类型这 4 个方向，下面一一介绍。</p>
<h3 data-nodeid="11269">操作接口类型</h3>
<h4 data-nodeid="11270">Partial</h4>
<p data-nodeid="11271">Partial 工具类型可以将一个类型的所有属性变为可选的，且该工具类型返回的类型是给定类型的所有子集，下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11272"><code data-language="typescript"><span class="hljs-keyword">type</span> Partial&lt;T&gt; = {
  [P <span class="hljs-keyword">in</span> keyof T]?: T[P];
};
<span class="hljs-keyword">interface</span> Person {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
  weight?: <span class="hljs-built_in">number</span>;
}
<span class="hljs-keyword">type</span> PartialPerson = Partial&lt;Person&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">interface</span> PartialPerson {
  name?: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
  weight?: <span class="hljs-built_in">number</span>;
}
</code></pre>
<p data-nodeid="11273">在上述示例中，我们使用映射类型取出了传入类型的所有键值，并将其值设定为可选的。</p>
<h4 data-nodeid="11274">Required</h4>
<p data-nodeid="11275">与 Partial 工具类型相反，Required 工具类型可以将给定类型的所有属性变为必填的，下面我们看一个具体示例。</p>
<pre class="lang-typescript" data-nodeid="11276"><code data-language="typescript"><span class="hljs-keyword">type</span> Required&lt;T&gt; = {
  [P <span class="hljs-keyword">in</span> keyof T]-?: T[P];
};
<span class="hljs-keyword">type</span> RequiredPerson = Required&lt;Person&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">interface</span> RequiredPerson {
  name: <span class="hljs-built_in">string</span>;
  age: <span class="hljs-built_in">number</span>;
  weight: <span class="hljs-built_in">number</span>;
}
</code></pre>
<p data-nodeid="11277">在上述示例中，映射类型在键值的后面使用了一个 - 符号，- 与 ? 组合起来表示去除类型的可选属性，因此给定类型的所有属性都变为了必填。</p>
<h4 data-nodeid="11278">Readonly</h4>
<p data-nodeid="11279">Readonly 工具类型可以将给定类型的所有属性设为只读，这意味着给定类型的属性不可以被重新赋值，下面我们看一个具体的示例。</p>
<pre class="lang-typescript" data-nodeid="11280"><code data-language="typescript"><span class="hljs-keyword">type</span> Readonly&lt;T&gt; = {
  readonly [P <span class="hljs-keyword">in</span> keyof T]: T[P];
};
<span class="hljs-keyword">type</span> ReadonlyPerson = Readonly&lt;Person&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">interface</span> ReadonlyPerson {
  readonly name: <span class="hljs-built_in">string</span>;
  readonly age?: <span class="hljs-built_in">number</span>;
  readonly weight?: <span class="hljs-built_in">number</span>;
}
</code></pre>
<p data-nodeid="11281">在上述示例中，经过 Readonly 处理后，ReadonlyPerson 的 name、age、weight 等属性都变成了 readonly 只读。</p>
<h4 data-nodeid="11282">Pick</h4>
<p data-nodeid="11283">Pick 工具类型可以从给定的类型中选取出指定的键值，然后组成一个新的类型，下面我们看一个具体的示例。</p>
<pre class="lang-typescript" data-nodeid="11284"><code data-language="typescript"><span class="hljs-keyword">type</span> Pick&lt;T, K <span class="hljs-keyword">extends</span> keyof T&gt; = {
  [P <span class="hljs-keyword">in</span> K]: T[P];
};
<span class="hljs-keyword">type</span> NewPerson = Pick&lt;Person, <span class="hljs-string">'name'</span> | <span class="hljs-string">'age'</span>&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">interface</span> NewPerson {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
}
</code></pre>
<p data-nodeid="11285">在上述示例中，Pick工具类型接收了两个泛型参数，第一个 T 为给定的参数类型，而第二个参数为需要提取的键值 key。有了参数类型和需要提取的键值 key，我们就可以通过映射类型很容易地实现 Pick 工具类型的功能。</p>
<h4 data-nodeid="11286">Omit</h4>
<p data-nodeid="11287">与 Pick 类型相反，Omit 工具类型的功能是返回去除指定的键值之后返回的新类型，下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11288"><code data-language="typescript"><span class="hljs-keyword">type</span> Omit&lt;T, K <span class="hljs-keyword">extends</span> keyof <span class="hljs-built_in">any</span>&gt; = Pick&lt;T, Exclude&lt;keyof T, K&gt;&gt;;
<span class="hljs-keyword">type</span> NewPerson = Omit&lt;Person, <span class="hljs-string">'weight'</span>&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">interface</span> NewPerson {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
}
</code></pre>
<p data-nodeid="11289">在上述示例中，Omit 类型的实现使用了前面介绍的 Pick 类型。我们知道 Pick 类型的作用是选取给定类型的指定属性，那么这里的 Omit 的作用应该是选取除了指定属性之外的属性，而 Exclude 工具类型的作用就是从入参 T 属性的联合类型中排除入参 K 指定的若干属性。</p>
<blockquote data-nodeid="11290">
<p data-nodeid="11291"><strong data-nodeid="11394">Tips</strong>：操作接口类型这一小节所介绍的工具类型都使用了映射类型。通过映射类型，我们可以对原类型的属性进行重新映射，从而组成想要的类型。</p>
</blockquote>
<h3 data-nodeid="11292">联合类型</h3>
<h4 data-nodeid="11293">Exclude</h4>
<p data-nodeid="11294">在介绍 Omit 类型的实现中，我们使用了 Exclude 类型。通过使用 Exclude 类型，我们从接口的所有属性中去除了指定属性，因此，Exclude 的作用就是从联合类型中去除指定的类型。</p>
<p data-nodeid="11295">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11296"><code data-language="typescript"><span class="hljs-keyword">type</span> Exclude&lt;T, U&gt; = T <span class="hljs-keyword">extends</span> U ? never : T;
<span class="hljs-keyword">type</span> T = Exclude&lt;<span class="hljs-string">'a'</span> | <span class="hljs-string">'b'</span> | <span class="hljs-string">'c'</span>, <span class="hljs-string">'a'</span>&gt;; <span class="hljs-comment">// =&gt; 'b' | 'c'</span>
<span class="hljs-keyword">type</span> NewPerson = Omit&lt;Person, <span class="hljs-string">'weight'</span>&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">type</span> NewPerson = Pick&lt;Person, Exclude&lt;keyof Person, <span class="hljs-string">'weight'</span>&gt;&gt;;
<span class="hljs-comment">// 其中</span>
<span class="hljs-keyword">type</span> ExcludeKeys = Exclude&lt;keyof Person, <span class="hljs-string">'weight'</span>&gt;; <span class="hljs-comment">// =&gt; 'name' | 'age'</span>
</code></pre>
<p data-nodeid="11297">在上述示例中，Exclude 的实现使用了条件类型。如果类型 T 可被分配给类型 U ，则不返回类型 T，否则返回此类型 T ，这样我们就从联合类型中去除了指定的类型。</p>
<p data-nodeid="11298">再回看之前的 NewPerson 类型的例子，我们也就很明白了。在 ExcludeKeys 中，如果 Person 类型的属性是我们要去除的属性，则不返回该属性，否则返回其类型。</p>
<h4 data-nodeid="11299">Extract</h4>
<p data-nodeid="11300">Extract 类型的作用与 Exclude 正好相反，Extract 主要用来从联合类型中提取指定的类型，类似于操作接口类型中的 Pick 类型。</p>
<p data-nodeid="11301">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11302"><code data-language="typescript"><span class="hljs-keyword">type</span> Extract&lt;T, U&gt; = T <span class="hljs-keyword">extends</span> U ? T : never;
<span class="hljs-keyword">type</span> T = Extract&lt;<span class="hljs-string">'a'</span> | <span class="hljs-string">'b'</span> | <span class="hljs-string">'c'</span>, <span class="hljs-string">'a'</span>&gt;; <span class="hljs-comment">// =&gt; 'a'</span>
</code></pre>
<p data-nodeid="11303">通过上述示例，我们发现 Extract 类型相当于取出两个联合类型的交集。</p>
<p data-nodeid="11304">此外，我们还可以基于 Extract 实现一个获取接口类型交集的工具类型，如下示例：</p>
<pre class="lang-typescript" data-nodeid="11305"><code data-language="typescript"><span class="hljs-keyword">type</span> Intersect&lt;T, U&gt; = {
  [K <span class="hljs-keyword">in</span> Extract&lt;keyof T, keyof U&gt;]: T[K];
};
<span class="hljs-keyword">interface</span> Person {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
  weight?: <span class="hljs-built_in">number</span>;
}
<span class="hljs-keyword">interface</span> NewPerson {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
}
<span class="hljs-keyword">type</span> T = Intersect&lt;Person, NewPerson&gt;;
<span class="hljs-comment">// 相当于</span>
<span class="hljs-keyword">type</span> T = {
  name: <span class="hljs-built_in">string</span>;
  age?: <span class="hljs-built_in">number</span>;
};
</code></pre>
<p data-nodeid="11306">在上述的例子中，我们使用了 Extract 类型来提取两个接口类型属性的交集，并使用映射类型生成了一个新的类型。</p>
<h4 data-nodeid="11307">NonNullable</h4>
<p data-nodeid="11308">NonNullable 的作用是从联合类型中去除 null 或者 undefined 的类型。如果你对条件类型已经很熟悉了，那么应该知道如何实现 NonNullable 类型了。</p>
<p data-nodeid="11309">下面看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11310"><code data-language="typescript"><span class="hljs-keyword">type</span> NonNullable&lt;T&gt; = T <span class="hljs-keyword">extends</span> <span class="hljs-literal">null</span> | <span class="hljs-literal">undefined</span> ? never : T;
<span class="hljs-comment">// 等同于使用 Exclude</span>
<span class="hljs-keyword">type</span> NonNullable&lt;T&gt; = Exclude&lt;T, <span class="hljs-literal">null</span> | <span class="hljs-literal">undefined</span>&gt;;
<span class="hljs-keyword">type</span> T = NonNullable&lt;<span class="hljs-built_in">string</span> | <span class="hljs-built_in">number</span> | <span class="hljs-literal">undefined</span> | <span class="hljs-literal">null</span>&gt;; <span class="hljs-comment">// =&gt; string | number</span>
</code></pre>
<p data-nodeid="11311">在上述示例中，如果 NonNullable 传入的类型可以被分配给 null 或是 undefined ，则不返回该类型，否则返回其具体类型。</p>
<h4 data-nodeid="11312">Record</h4>
<p data-nodeid="11313">Record 的作用是生成接口类型，然后我们使用传入的泛型参数分别作为接口类型的属性和值。</p>
<p data-nodeid="11314">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11315"><code data-language="typescript"><span class="hljs-keyword">type</span> Record&lt;K <span class="hljs-keyword">extends</span> keyof <span class="hljs-built_in">any</span>, T&gt; = {
  [P <span class="hljs-keyword">in</span> K]: T;
};
<span class="hljs-keyword">type</span> MenuKey = <span class="hljs-string">'home'</span> | <span class="hljs-string">'about'</span> | <span class="hljs-string">'more'</span>;
<span class="hljs-keyword">interface</span> Menu {
  label: <span class="hljs-built_in">string</span>;
  hidden?: <span class="hljs-built_in">boolean</span>;
}
<span class="hljs-keyword">const</span> menus: Record&lt;MenuKey, Menu&gt; = {
  about: { label: <span class="hljs-string">'关于'</span> },
  home: { label: <span class="hljs-string">'主页'</span> },
  more: { label: <span class="hljs-string">'更多'</span>, hidden: <span class="hljs-literal">true</span> },
};
</code></pre>
<p data-nodeid="11316">在上述示例中，Record 类型接收了两个泛型参数：第一个参数作为接口类型的属性，第二个参数作为接口类型的属性值。</p>
<blockquote data-nodeid="11317">
<p data-nodeid="11318"><strong data-nodeid="11420">需要注意：这里的实现限定了第一个泛型参数继承自</strong><code data-backticks="1" data-nodeid="11418">keyof any</code>。</p>
</blockquote>
<p data-nodeid="11319">在 TypeScript 中，keyof any 指代可以作为对象键的属性，如下示例：</p>
<pre class="lang-typescript" data-nodeid="11320"><code data-language="typescript"><span class="hljs-keyword">type</span> T = keyof <span class="hljs-built_in">any</span>; <span class="hljs-comment">// =&gt; string | number | symbol</span>
</code></pre>
<blockquote data-nodeid="11321">
<p data-nodeid="11322"><strong data-nodeid="11432">说明</strong>：目前，JavaScript 仅支持<code data-backticks="1" data-nodeid="11426">string</code>、<code data-backticks="1" data-nodeid="11428">number</code>、<code data-backticks="1" data-nodeid="11430">symbol</code>作为对象的键值。</p>
</blockquote>
<h3 data-nodeid="11323">函数类型</h3>
<h4 data-nodeid="11324">ConstructorParameters</h4>
<p data-nodeid="11325">ConstructorParameters 可以用来获取构造函数的构造参数，而 ConstructorParameters 类型的实现则需要使用 infer 关键字推断构造参数的类型。</p>
<p data-nodeid="11326">关于 infer 关键字，我们可以把它当成简单的模式匹配来看待。如果真实的参数类型和 infer 匹配的一致，那么就返回匹配到的这个类型。</p>
<p data-nodeid="11327">下面看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11328"><code data-language="typescript"><span class="hljs-keyword">type</span> ConstructorParameters&lt;T <span class="hljs-keyword">extends</span> <span class="hljs-keyword">new</span> (...args: <span class="hljs-built_in">any</span>) =&gt; <span class="hljs-built_in">any</span>&gt; = T <span class="hljs-keyword">extends</span> <span class="hljs-keyword">new</span> (
  ...args: infer P
) =&gt; <span class="hljs-built_in">any</span>
  ? P
  : never;
<span class="hljs-keyword">class</span> Person {
  <span class="hljs-keyword">constructor</span>(<span class="hljs-params">name: <span class="hljs-built_in">string</span>, age?: <span class="hljs-built_in">number</span></span>) {}
}
<span class="hljs-keyword">type</span> T = ConstructorParameters&lt;<span class="hljs-keyword">typeof</span> Person&gt;; <span class="hljs-comment">// [name: string, age?: number]</span>
</code></pre>
<p data-nodeid="11329">在上述示例中，ConstructorParameters 泛型接收了一个参数，并且限制了这个参数需要实现构造函数。于是，我们通过 infer 关键字匹配了构造函数内的构造参数，并返回了这些参数。因此，可以看到第 11 行匹配了 Person 构造函数的两个参数，并返回了一个元组类型 [string, number] 给类型别名 T。</p>
<h4 data-nodeid="11330">Parameters</h4>
<p data-nodeid="11331">Parameters 的作用与 ConstructorParameters 类似，Parameters 可以用来获取函数的参数并返回序对，如下示例：</p>
<pre class="lang-typescript" data-nodeid="11332"><code data-language="typescript"><span class="hljs-keyword">type</span> Parameters&lt;T <span class="hljs-keyword">extends</span> (...args: <span class="hljs-built_in">any</span>) =&gt; <span class="hljs-built_in">any</span>&gt; = T <span class="hljs-keyword">extends</span> (...args: infer P) =&gt; <span class="hljs-built_in">any</span> ? P : never;
<span class="hljs-keyword">type</span> T0 = Parameters&lt;<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-built_in">void</span>&gt;; <span class="hljs-comment">// []</span>
<span class="hljs-keyword">type</span> T1 = Parameters&lt;<span class="hljs-function">(<span class="hljs-params">x: <span class="hljs-built_in">number</span>, y?: <span class="hljs-built_in">string</span></span>) =&gt;</span> <span class="hljs-built_in">void</span>&gt;; <span class="hljs-comment">// [x: number, y?: string]</span>
</code></pre>
<p data-nodeid="11333">在上述示例中，Parameters 的泛型参数限制了传入的类型需要满足函数类型。</p>
<h4 data-nodeid="11334">ReturnType</h4>
<p data-nodeid="11335">ReturnType 的作用是用来获取函数的返回类型，下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11336"><code data-language="typescript"><span class="hljs-keyword">type</span> ReturnType&lt;T <span class="hljs-keyword">extends</span> (...args: <span class="hljs-built_in">any</span>) =&gt; <span class="hljs-built_in">any</span>&gt; = T <span class="hljs-keyword">extends</span> (...args: <span class="hljs-built_in">any</span>) =&gt; infer R ? R : <span class="hljs-built_in">any</span>;
<span class="hljs-keyword">type</span> T0 = ReturnType&lt;<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-built_in">void</span>&gt;; <span class="hljs-comment">// =&gt; void</span>
<span class="hljs-keyword">type</span> T1 = ReturnType&lt;<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-built_in">string</span>&gt;; <span class="hljs-comment">// =&gt; string</span>
</code></pre>
<p data-nodeid="11337">在上述示例中，<code data-backticks="1" data-nodeid="11449">ReturnType</code>的泛型参数限制了传入的类型需要满足函数类型。</p>
<h4 data-nodeid="11338">ThisParameterType</h4>
<p data-nodeid="11339">ThisParameterType 可以用来获取函数的 this 参数类型。</p>
<p data-nodeid="11340">关于函数的 this 参数，我们在 05 讲函数类型中介绍过，下面看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11341"><code data-language="typescript"><span class="hljs-keyword">type</span> ThisParameterType&lt;T&gt; = T <span class="hljs-keyword">extends</span> (<span class="hljs-keyword">this</span>: infer U, ...args: <span class="hljs-built_in">any</span>[]) =&gt; <span class="hljs-built_in">any</span> ? U : unknown;
<span class="hljs-keyword">type</span> T = ThisParameterType&lt;<span class="hljs-function">(<span class="hljs-params"><span class="hljs-keyword">this</span>: <span class="hljs-built_in">Number</span>, x: <span class="hljs-built_in">number</span></span>) =&gt;</span> <span class="hljs-built_in">void</span>&gt;; <span class="hljs-comment">// Number</span>
</code></pre>
<p data-nodeid="11342">在上述示例的第 1 行中，因为函数类型的第一个参数声明的是 this 参数类型，所以我们可以直接使用 infer 关键字进行匹配并获取 this 参数类型。在示例的第 3 行，类型别名 T 得到的类型就是 Number。</p>
<h4 data-nodeid="11343">ThisType</h4>
<p data-nodeid="11344">ThisType 的作用是可以在对象字面量中指定 this 的类型。ThisType 不返回转换后的类型，而是通过 ThisType 的泛型参数指定 this 的类型，下面看一个具体的示例：</p>
<blockquote data-nodeid="11345">
<p data-nodeid="11346">注意：如果你想使用这个工具类型，那么需要开启<code data-backticks="1" data-nodeid="11458">noImplicitThis</code>的 TypeScript 配置。</p>
</blockquote>
<pre class="lang-typescript" data-nodeid="11347"><code data-language="typescript"><span class="hljs-keyword">type</span> ObjectDescriptor&lt;D, M&gt; = {
  data?: D;
  methods?: M &amp; ThisType&lt;D &amp; M&gt;; <span class="hljs-comment">// methods 中 this 的类型是 D &amp; M</span>
};
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">makeObject</span>&lt;<span class="hljs-title">D</span>, <span class="hljs-title">M</span>&gt;(<span class="hljs-params">desc: ObjectDescriptor&lt;D, M&gt;</span>): <span class="hljs-title">D</span> &amp; <span class="hljs-title">M</span> </span>{
  <span class="hljs-keyword">let</span> data: object = desc.data || {};
  <span class="hljs-keyword">let</span> methods: object = desc.methods || {};
  <span class="hljs-keyword">return</span> { ...data, ...methods } <span class="hljs-keyword">as</span> D &amp; M;
}
<span class="hljs-keyword">const</span> obj = makeObject({
  data: { x: <span class="hljs-number">0</span>, y: <span class="hljs-number">0</span> },
  methods: {
    moveBy(dx: <span class="hljs-built_in">number</span>, dy: <span class="hljs-built_in">number</span>) {
      <span class="hljs-keyword">this</span>.x += dx; <span class="hljs-comment">// this =&gt; D &amp; M</span>
      <span class="hljs-keyword">this</span>.y += dy; <span class="hljs-comment">// this =&gt; D &amp; M</span>
    },
  },
});
obj.x = <span class="hljs-number">10</span>;
obj.y = <span class="hljs-number">20</span>;
obj.moveBy(<span class="hljs-number">5</span>, <span class="hljs-number">5</span>);
</code></pre>
<p data-nodeid="11348">在上述示例子中，methods 属性的 this 类型为 D &amp; M，在上下文中指代 { x: number, y: number } &amp; { moveBy(dx: number, dy: number): void }。</p>
<p data-nodeid="11349">ThisType 工具类型只是提供了一个空的泛型接口，仅可以在对象字面量上下文中被 TypeScript 识别，如下所示：</p>
<pre class="lang-typescript" data-nodeid="11350"><code data-language="typescript"><span class="hljs-keyword">interface</span> ThisType&lt;T&gt; {}
</code></pre>
<p data-nodeid="11351">也就是说该类型的作用相当于任意空接口。</p>
<h4 data-nodeid="11352">OmitThisParameter</h4>
<p data-nodeid="11353">OmitThisParameter 工具类型主要用来去除函数类型中的 this 类型。如果传入的函数类型没有显式声明 this 类型，那么返回的仍是原来的函数类型。</p>
<p data-nodeid="11354">下面看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="11355"><code data-language="typescript"><span class="hljs-keyword">type</span> OmitThisParameter&lt;T&gt; = unknown <span class="hljs-keyword">extends</span> ThisParameterType&lt;T&gt;
  ? T
  : T <span class="hljs-keyword">extends</span> (...args: infer A) =&gt; infer R
  ? <span class="hljs-function">(<span class="hljs-params">...args: A</span>) =&gt;</span> R
  : T;
<span class="hljs-keyword">type</span> T = OmitThisParameter&lt;<span class="hljs-function">(<span class="hljs-params"><span class="hljs-keyword">this</span>: <span class="hljs-built_in">Number</span>, x: <span class="hljs-built_in">number</span></span>) =&gt;</span> <span class="hljs-built_in">string</span>&gt;; <span class="hljs-comment">// (x: number) =&gt; string</span>
</code></pre>
<p data-nodeid="11356">在上述示例中， ThisParameterType 类型的实现如果传入的泛型参数无法推断 this 的类型，则会返回 unknown 类型。在OmitThisParameter 的实现中，第一个条件语句如果传入的函数参数没有 this 类型，则返回原类型；否则通过 infer 分别获取函数参数和返回值的类型构造一个新的没有 this 的函数类型，并返回这个函数类型。</p>
<h3 data-nodeid="11357">字符串类型</h3>
<h4 data-nodeid="11358">模板字符串</h4>
<p data-nodeid="11359">TypeScript 自 4.1版本起开始支持模板字符串字面量类型。为此，TypeScript 也提供了 Uppercase、Lowercase、Capitalize、Uncapitalize这 4 种内置的操作字符串的类型，如下示例：</p>
<pre class="lang-typescript" data-nodeid="11360"><code data-language="typescript"><span class="hljs-comment">// 转换字符串字面量到大写字母</span>
<span class="hljs-keyword">type</span> Uppercase&lt;S <span class="hljs-keyword">extends</span> <span class="hljs-built_in">string</span>&gt; = intrinsic;
<span class="hljs-comment">// 转换字符串字面量到小写字母</span>
<span class="hljs-keyword">type</span> Lowercase&lt;S <span class="hljs-keyword">extends</span> <span class="hljs-built_in">string</span>&gt; = intrinsic;
<span class="hljs-comment">// 转换字符串字面量的第一个字母为大写字母</span>
<span class="hljs-keyword">type</span> Capitalize&lt;S <span class="hljs-keyword">extends</span> <span class="hljs-built_in">string</span>&gt; = intrinsic;
<span class="hljs-comment">// 转换字符串字面量的第一个字母为小写字母</span>
<span class="hljs-keyword">type</span> Uncapitalize&lt;S <span class="hljs-keyword">extends</span> <span class="hljs-built_in">string</span>&gt; = intrinsic;
<span class="hljs-keyword">type</span> T0 = Uppercase&lt;<span class="hljs-string">'Hello'</span>&gt;; <span class="hljs-comment">// =&gt; 'HELLO'</span>
<span class="hljs-keyword">type</span> T1 = Lowercase&lt;T0&gt;; <span class="hljs-comment">// =&gt; 'hello'</span>
<span class="hljs-keyword">type</span> T2 = Capitalize&lt;T1&gt;; <span class="hljs-comment">// =&gt; 'Hello'</span>
<span class="hljs-keyword">type</span> T3 = Uncapitalize&lt;T2&gt;; <span class="hljs-comment">// =&gt; 'hello'</span>
</code></pre>
<p data-nodeid="11361">在上述示例中，这 4 种操作字符串字面量工具类型的实现都是使用 JavaScript 运行时的字符串操作函数计算出来的，且不支持语言区域设置。以下代码是这 4 种字符串工具类型的实际实现。</p>
<pre class="lang-typescript" data-nodeid="11362"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">applyStringMapping</span>(<span class="hljs-params">symbol: Symbol, str: <span class="hljs-built_in">string</span></span>) </span>{
  <span class="hljs-keyword">switch</span> (intrinsicTypeKinds.get(symbol.escapedName <span class="hljs-keyword">as</span> <span class="hljs-built_in">string</span>)) {
    <span class="hljs-keyword">case</span> IntrinsicTypeKind.Uppercase:
      <span class="hljs-keyword">return</span> str.toUpperCase();
    <span class="hljs-keyword">case</span> IntrinsicTypeKind.Lowercase:
      <span class="hljs-keyword">return</span> str.toLowerCase();
    <span class="hljs-keyword">case</span> IntrinsicTypeKind.Capitalize:
      <span class="hljs-keyword">return</span> str.charAt(<span class="hljs-number">0</span>).toUpperCase() + str.slice(<span class="hljs-number">1</span>);
    <span class="hljs-keyword">case</span> IntrinsicTypeKind.Uncapitalize:
      <span class="hljs-keyword">return</span> str.charAt(<span class="hljs-number">0</span>).toLowerCase() + str.slice(<span class="hljs-number">1</span>);
  }
  <span class="hljs-keyword">return</span> str;
}
</code></pre>
<p data-nodeid="11363">在上述代码中可以看到，字符串的转换使用了 JavaScript 中字符串的 toUpperCase 和 toLowerCase 方法，而不是 toLocaleUpperCase 和 toLocaleLowerCase。其中 toUpperCase 和 toLowerCase 采用的是 Unicode 编码默认的大小写转换规则。</p>
<h3 data-nodeid="11364">小结与预告</h3>
<p data-nodeid="11365">这一讲我们学习了操作接口类型、联合类型、函数、字符串的工具类。</p>
<p data-nodeid="11366">在学习这些工具类型的实现时，我们发现它们都是基于映射类型、条件类型、infer 推断实现的。可以说掌握了这 3 种类型操作的技巧，我们就可以自由地组合更多的工具类型了。</p>
<p data-nodeid="11367">插播一道思考题：基于 Exclude 工具类型的代码实现，请你分析一下为什么它可以从联合类型中排除掉指定成员？欢迎你在留言区与我互动/交流。</p>
<p data-nodeid="11368">当然，这道题涉及的知识点大概率超纲了，在 15 讲编写自定义工具类型中我们将更详细地分析。请你保持好奇心，敬请期待吧！</p>
<p data-nodeid="11369">如果你觉得本专栏有价值，欢迎分享给更多好友！</p>

---

### 精选评论

##### *杰：
> 到联合类型结束，都能看懂 工作中也经常用，然而函数类型的工具方法就。。。。。😫

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 多品读几遍！你可以把这些工具类型当成类型里的函数就比较好记忆和理解了！

