<h3>功能需求</h3>
<p>这一课时我们来写一个 CSS 预处理器，它的功能可以理解为精简版的 <a href="https://stylus.bootcss.com/">stylus</a>，主要实现的功能有：</p>
<ul>
<li>用空格和换行符替代花括号、冒号和分号；</li>
<li>支持选择器的嵌套组合；</li>
<li>支持以“$”符号开头的变量定义和使用。</li>
</ul>
<p>如果你对这种风格不是很熟悉也没关系，通过下面这个例子你就能很快明白。</p>
<p><strong>目标 CSS 代码</strong>，为 5 条样式规则。第 1 条和第 5 条样式规则是最简单的，使用 1 个选择器，定义了 1 条样式属性；第 2 条规则多用了一个标签选择器，样式属性值为多个字符串组成；第 3 条规则使用了类选择器；第 4 条规则增加了属性选择器，并且样式属性增加为 2 条。</p>
<pre><code data-language="css" class="lang-css"><span class="hljs-selector-tag">div</span> {<span class="hljs-attribute">color</span>:darkkhaki;}
<span class="hljs-selector-tag">div</span> <span class="hljs-selector-tag">p</span> {<span class="hljs-attribute">border</span>:<span class="hljs-number">1px</span> solid lightgreen;}
<span class="hljs-selector-tag">div</span> <span class="hljs-selector-class">.a-b</span> {<span class="hljs-attribute">background-color</span>:lightyellow;}
<span class="hljs-selector-tag">div</span> <span class="hljs-selector-class">.a-b</span> <span class="hljs-selector-attr">[data]</span> {<span class="hljs-attribute">padding</span>:<span class="hljs-number">15px</span>;<span class="hljs-attribute">font-size</span>:<span class="hljs-number">12px</span>;}
<span class="hljs-selector-class">.d-ib</span> {<span class="hljs-attribute">display</span>:inline-block;}
</code></pre>
<p>再来看看“源代码”，首先声明了两个变量，然后通过换行缩进定义了上述样式规则中的<strong>选择器和样式</strong>：</p>
<pre><code data-language="java" class="lang-java">$ib inline-block
$borderColor lightgreen
div
&nbsp; p
&nbsp; &nbsp; border <span class="hljs-number">1</span>px solid $borderColor
&nbsp; color darkkhaki
&nbsp; .a-b
&nbsp; &nbsp; background-color lightyellow
&nbsp; &nbsp; [data]
&nbsp; &nbsp; &nbsp; padding <span class="hljs-number">15</span>px
&nbsp; &nbsp; &nbsp; font-size <span class="hljs-number">12</span>px
.d-ib
&nbsp; display $ib
</code></pre>
<p>像上面这种强制缩进换行的风格应用非常广泛，比如编程语言 Python、HTML 模板 pug、预处理器 Sass（以“.sass”为后缀的文件）。</p>
<p>这种风格可能有些工程师并不适应，因为缩进空格数不一致就会导致程序解析失败或执行出错。但它也有一些优点，比如格式整齐，省去了花括号等冗余字符，减少了代码量。推荐大家在项目中使用。</p>
<h3>编译器</h3>
<p>对预处理器这种能将一种语言（法）转换成另一种语言（法）的程序一般称之为“<strong>编译器</strong>”。我们平常所知的高级语言都离不开编译器，比如 C++、Java、JavaScript。</p>
<p>不同语言的编译器的工作流程有些差异，但大体上可以分成三个步骤：解析（Parsing）、转换（Transformation）及代码生成（Code Generation）。</p>
<h4>解析</h4>
<p>解析步骤一般分为两个阶段：<strong>词法分析</strong>和<strong>语法分析</strong>。</p>
<p>词法分析就是将接收到的源代码转换成令牌（Token），完成这个过程的函数或工具被称之为<strong>词法分析器</strong>（Tokenizer 或 Lexer）。</p>
<p>令牌由一些代码语句的碎片生成，它们可以是数字、标签、标点符号、运算符，或者其他任何东西。</p>
<p>将代码令牌化之后会进入语法分析，这个过程会将之前生成的令牌转换成一种带有令牌关系描述的抽象表示，这种抽象的表示称之为<strong>抽象语法树</strong>（Abstract Syntax Tree，AST）。完成这个过程的函数或工具被称为<strong>语法分析器</strong>（Parser）。</p>
<p>抽象语法树通常是一个深度嵌套的对象，这种数据结构不仅更贴合代码逻辑，在后面的操作效率方面相对于令牌数组也更有优势。</p>
<p>可以回想一下，我们在第 06 讲中提到的解析 HTML 流程也包括了这两个步骤。</p>
<h4>转换</h4>
<p>解析完成之后的下一步就是<strong>转换</strong>，即把 AST 拿过来然后做一些修改，完成这个过程的函数或工具被称之为<strong>转换器</strong>（Transformer）。</p>
<p>在这个过程中，AST 中的节点可以被修改和删除，也可以新增节点。根本目的就是为了代码生成的时候更加方便。</p>
<h4>代码生成</h4>
<p>编译器的最后一步就是根据转换后的 AST 来生成目标代码，这个阶段做的事情有时候会和转换重叠，但是代码生成最主要的部分还是根据转换后的 AST 来输出代码。完成这个过程的函数或工具被称之为<strong>生成器</strong>（Generator）。</p>
<p>代码生成有几种不同的工作方式，有些编译器将会重用之前生成的令牌，有些会创建独立代码</p>
<p>表示，以便于线性地输出代码。但是接下来我们还是着重于使用之前生成好的 AST。</p>
<p>代码生成器必须知道如何“打印”转换后的 AST 中所有类型的节点，然后递归地调用自身，直到所有代码都被打印到一个很长的字符串中。</p>
<h3>代码实现</h3>
<p>学习了编译器相关知识之后，我们再来按照上述步骤编写代码。</p>
<h4>词法分析</h4>
<p>在进行词法分析之前，首先要考虑字符串可以被拆分成多少种类型的令牌，然后再确定令牌的判断条件及解析方式。</p>
<p>通过分析源代码，可以将字符串分为变量、变量值、选择器、属性、属性值 5 种类型。但其中属性值和变量可以合并成一类进行处理，为了方便后面语法分析，变量可以拆分成变量定义和变量引用。</p>
<p>由于缩进会对语法分析产生影响（样式规则缩进空格数决定了属于哪个选择器），所以也要加入令牌对象。</p>
<p>因此一个令牌对象结构如下，type 属性表示令牌类型，value 属性存储令牌字符内容，indent 属性记录缩进空格数：</p>
<pre><code data-language="typescript" class="lang-typescript">{
&nbsp; <span class="hljs-keyword">type</span>: <span class="hljs-string">"variableDef"</span> | <span class="hljs-string">"variableRef"</span> | <span class="hljs-string">"selector"</span> | <span class="hljs-string">"property"</span> | <span class="hljs-string">"value"</span>, <span class="hljs-comment">//枚举值，分别对应变量定义、变量引用、选择器、属性、值</span>
&nbsp; value: <span class="hljs-built_in">string</span>, <span class="hljs-comment">// token字符值，即被分解的字符串</span>
&nbsp; indent: <span class="hljs-built_in">number</span> <span class="hljs-comment">// 缩进空格数，需要根据它判断从属关系</span>
}
</code></pre>
<p>然后确定各种类型令牌的判断条件：</p>
<ul>
<li><strong>variableDef</strong>，以“$”符号开头，该行前面无其他非空字符串；</li>
<li><strong>variableRef</strong>，以“$”符号开头，该行前面有非空字符串；</li>
<li><strong>selector</strong>，独占一行，该行无其他非空字符串；</li>
<li><strong>property</strong>，以字母开头，该行前面无其他非空字符串；</li>
<li><strong>value</strong>，非该行第一个字符串，且该行第一个字符串为 property 或 variableDef 类型。</li>
</ul>
<p>最后再来确定令牌解析方式。</p>
<p>一般进行词法解析的时候，可以逐个字符进行解析判断，但考虑到源代码语法的特殊性——换行符和空格缩进会影响语法解析，所以可以考虑逐行逐个单词进行解析。</p>
<p>词法分析代码如下所示：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">tokenize</span>(<span class="hljs-params">text</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;text.trim().split(<span class="hljs-regexp">/\n|\r\n/</span>).reduce(<span class="hljs-function">(<span class="hljs-params">tokens,&nbsp;line,&nbsp;idx</span>)&nbsp;=&gt;</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;spaces&nbsp;=&nbsp;line.match(<span class="hljs-regexp">/^\s+/</span>)&nbsp;||&nbsp;[<span class="hljs-string">''</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;indent&nbsp;=&nbsp;spaces[<span class="hljs-number">0</span>].length
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;input&nbsp;=&nbsp;line.trim()
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;words&nbsp;=&nbsp;input.split(<span class="hljs-regexp">/\s/</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;value&nbsp;=&nbsp;words.shift()
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(words.length&nbsp;===&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tokens.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-string">'selector'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;indent
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;type&nbsp;=&nbsp;<span class="hljs-string">''</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(<span class="hljs-regexp">/^\$/</span>.test(value))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type&nbsp;=&nbsp;<span class="hljs-string">'variableDef'</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;<span class="hljs-keyword">if</span>&nbsp;(<span class="hljs-regexp">/^[a-zA-Z-]+$/</span>.test(value))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type&nbsp;=&nbsp;<span class="hljs-string">'property'</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span>&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-built_in">Error</span>(<span class="hljs-string">`Tokenize&nbsp;error:Line&nbsp;<span class="hljs-subst">${idx}</span>&nbsp;"<span class="hljs-subst">${value}</span>"&nbsp;is&nbsp;not&nbsp;a&nbsp;vairable&nbsp;or&nbsp;property!`</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tokens.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;indent
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(value&nbsp;=&nbsp;words.shift())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tokens.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-regexp">/^\$/</span>.test(value)&nbsp;?&nbsp;<span class="hljs-string">'variableRef'</span>&nbsp;:&nbsp;<span class="hljs-string">'value'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">indent</span>:&nbsp;<span class="hljs-number">0</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;tokens;
&nbsp;&nbsp;},&nbsp;[])
}
</code></pre>
<h4>语法分析</h4>
<p>现在我们来分析如何将上一步生成的令牌数组转化成抽象语法树，树结构相对于数组而言，最大的特点是具有层级关系，哪些令牌具有层级关系呢？</p>
<p>从缩进中不难看出，选择器与选择器、选择器与属性都存在层级关系，那么我们可以分别通过 <strong>children 属性和 rules 属性</strong>来描述这两类层级关系。</p>
<p>要判断层级关系需要借助缩进空格数，所以节点需要增加一个属性 indent。</p>
<p>考虑到构建树时可能会产生回溯，那么可以设置一个数组来记录当前构建路径。当遇到非父子关系的节点时，沿着当前路径往上找到其父节点。</p>
<p>最后为了简化树结构，这一步也可以将变量值进行替换，从而减少变量节点。</p>
<p>所以抽象语法树可以写成如下结构。首先定义一个根节点，在其 children 属性中添加选择器节点，选择器节点相对令牌而言增加了 2 个属性：</p>
<ul>
<li><strong>rules</strong>，存储当前选择器的样式属性和值组成的对象，其中值以字符串数组的形式存储；</li>
<li><strong>children</strong>，子选择器节点。</li>
</ul>
<pre><code data-language="javascript" class="lang-javascript">{
&nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-string">'root'</span>,
&nbsp;&nbsp;<span class="hljs-attr">children</span>:&nbsp;[{
  &nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-string">'selector'</span>,
    <span class="hljs-attr">value</span>: string
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">rules</span>:&nbsp;[{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">property</span>:&nbsp;string,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">value</span>:&nbsp;string[],
&nbsp;&nbsp;&nbsp;&nbsp;}],
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">indent</span>:&nbsp;number,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">children</span>:&nbsp;[]
&nbsp;&nbsp;}]
}
</code></pre>
<p>由于考虑到一个属性的值可能会由多个令牌组成，比如 border 属性的值由“1px” “solid” “$borderColor” 3 个令牌组成，所以将 value 属性设置为字符串数组。</p>
<p>语法分析代码如下所示。首先定义一个根节点，然后按照先进先出的方式遍历令牌数组，遇到变量定义时，将变量名和对应的值存入到缓存对象中；当遇到属性时，插入到当前选择器节点的 rules 属性中，遇到值和变量引用时都将插入到当前选择器节点 rules 属性数组最后一个对象的 value 数组中，但是变量引用在插入之前需要借助缓存对象的变量值进行替换。当遇到选择器节点时，则需要往对应的父选择器节点 children 属性中插入，并将指针指向被插入的节点，同时记得将被插入的节点添加到用于存储遍历路径的数组中：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">parse</span>(<span class="hljs-params">tokens</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;ast&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-string">'root'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">children</span>:&nbsp;[],
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">indent</span>:&nbsp;<span class="hljs-number">-1</span>
&nbsp;&nbsp;};
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;path&nbsp;=&nbsp;[ast]
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;preNode&nbsp;=&nbsp;ast
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;node
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;vDict&nbsp;=&nbsp;{}
&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(node&nbsp;=&nbsp;tokens.shift())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'variableDef'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(tokens[<span class="hljs-number">0</span>]&nbsp;&amp;&amp;&nbsp;tokens[<span class="hljs-number">0</span>].type&nbsp;===&nbsp;<span class="hljs-string">'value'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;vNode&nbsp;=&nbsp;tokens.shift()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vDict[node.value]&nbsp;=&nbsp;vNode.value
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode.rules[preNode.rules.length&nbsp;-&nbsp;<span class="hljs-number">1</span>].value&nbsp;=&nbsp;vDict[node.value]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">continue</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'property'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.indent&nbsp;&gt;&nbsp;preNode.indent)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode.rules.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">property</span>:&nbsp;node.value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">value</span>:&nbsp;[]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;parent&nbsp;=&nbsp;path.pop()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(node.indent&nbsp;&lt;=&nbsp;parent.indent)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parent&nbsp;=&nbsp;path.pop()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parent.rules.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">property</span>:&nbsp;node.value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">value</span>:&nbsp;[]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode&nbsp;=&nbsp;parent
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;path.push(parent)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">continue</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'value'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode.rules[preNode.rules.length&nbsp;-&nbsp;<span class="hljs-number">1</span>].value.push(node.value);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">console</span>.error(preNode)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">continue</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'variableRef'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode.rules[preNode.rules.length&nbsp;-&nbsp;<span class="hljs-number">1</span>].value.push(vDict[node.value]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">continue</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'selector'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;item&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">type</span>:&nbsp;<span class="hljs-string">'selector'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">value</span>:&nbsp;node.value,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">indent</span>:&nbsp;node.indent,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">rules</span>:&nbsp;[],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">children</span>:&nbsp;[]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.indent&nbsp;&gt;&nbsp;preNode.indent)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;path[path.length&nbsp;-&nbsp;<span class="hljs-number">1</span>].indent&nbsp;===&nbsp;node.indent&nbsp;&amp;&amp;&nbsp;path.pop()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;path.push(item)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode.children.push(item);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;preNode&nbsp;=&nbsp;item;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;parent&nbsp;=&nbsp;path.pop()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(node.indent&nbsp;&lt;=&nbsp;parent.indent)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parent&nbsp;=&nbsp;path.pop()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parent.children.push(item)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;path.push(item)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;ast;
}
</code></pre>
<h4>转换</h4>
<p>在转换之前我们先来看看要生成的目标代码结构，其更像是一个由一条条样式规则组成的数组，所以我们考虑将抽象语法树转换成“抽象语法数组”。</p>
<p>在遍历树节点时，需要记录当前遍历路径，以方便选择器的拼接；同时可以考虑将“值”类型的节点拼接在一起。最后形成下面的数组结构，数组中每个元素对象包括两个属性，selector 属性值为当前规则的选择器，rules 属性为数组，数组中每个元素对象包含 property 和 value 属性：</p>
<pre><code data-language="css" class="lang-css">{
 &nbsp;<span class="hljs-attribute">selector</span>: string,
 &nbsp;rules: {
 &nbsp; &nbsp;property: string,
 &nbsp; &nbsp;value: string
 &nbsp;}<span class="hljs-selector-attr">[]</span>
}<span class="hljs-selector-attr">[]</span>
</code></pre>
<p>具体代码实现如下，递归遍历抽象语法树，遍历的时候完成选择器拼接以及属性值的拼接，最终返回一个与 CSS 样式规则相对应的数组：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">transform</span>(<span class="hljs-params">ast</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;newAst&nbsp;=&nbsp;[];
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">traverse</span>(<span class="hljs-params">node,&nbsp;result,&nbsp;prefix</span>)&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;selector&nbsp;=&nbsp;<span class="hljs-string">''</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(node.type&nbsp;===&nbsp;<span class="hljs-string">'selector'</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;selector&nbsp;=&nbsp;[...prefix,&nbsp;node.value];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">selector</span>:&nbsp;selector.join(<span class="hljs-string">'&nbsp;'</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">rules</span>:&nbsp;node.rules.reduce(<span class="hljs-function">(<span class="hljs-params">acc,&nbsp;rule</span>)&nbsp;=&gt;</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;acc.push({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">property</span>:&nbsp;rule.property,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">value</span>:&nbsp;rule.value.join(<span class="hljs-string">'&nbsp;'</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;acc;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},&nbsp;[])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">let</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;node.children.length;&nbsp;i++)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;traverse(node.children[i],&nbsp;result,&nbsp;selector)
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;&nbsp;traverse(ast,&nbsp;newAst,&nbsp;[])
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;newAst;
}
</code></pre>
<p>实现方式比较简单，通过函数递归遍历树，然后重新拼接选择器和属性的值，最终返回数组结构。</p>
<h4>代码生成</h4>
<p>有了新的“抽象语法数组”，生成目标代码就只需要通过 map 操作对数组进行遍历，然后将选择器、属性、值拼接成字符串返回即可。</p>
<p>具体代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-function"><span class="hljs-keyword">function</span>&nbsp;<span class="hljs-title">generate</span>(<span class="hljs-params">nodes</span>)&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;nodes.map(<span class="hljs-function"><span class="hljs-params">n</span>&nbsp;=&gt;</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;rules&nbsp;=&nbsp;n.rules.reduce(<span class="hljs-function">(<span class="hljs-params">acc,&nbsp;item</span>)&nbsp;=&gt;</span>&nbsp;acc&nbsp;+=&nbsp;<span class="hljs-string">`<span class="hljs-subst">${item.property}</span>:<span class="hljs-subst">${item.value}</span>;`</span>,&nbsp;<span class="hljs-string">''</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-string">`<span class="hljs-subst">${n.selector}</span>&nbsp;{<span class="hljs-subst">${rules}</span>}`</span>
&nbsp;&nbsp;}).join(<span class="hljs-string">'\n'</span>)
}
</code></pre>
<h3>总结</h3>
<p>这一课时动手实践了一个简单的 CSS 预处理器，希望你能更好地掌握 CSS 工具预处理器的基本原理，同时也希望通过这个实现过程带你跨入编译器的大门。编译器属于大家日用而不知的重要工具，像 webpack、Babel这些著名工具以及 JavaScript 引擎都用到了它。</p>
<p><a href="https://github.com/yalishizhude/course/blob/master/plus1/pre.js">完整代码地址</a></p>
<p>最后布置一道思考题：你能否为预处理器添加一些其他功能呢（比如局部变量）？</p>

---

### 精选评论

##### *月：
> 你说这个棒不棒😉😉😉😉

