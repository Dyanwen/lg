<p>今天带给你的内容是“函数式编程：你用什么声明？你在声明什么？”，在上个课时我们学习了 Spark 编程，Spark 编程风格是典型的函数式，很多开发人员在接触到这种编程风格时会有些陌生，<strong>本课时的主要目的就对这种编程风格进行一种解构，破除这种风格的神秘感，从理论上降低这种编程风格的难度</strong>。当然了，由于函数式编程包含了丰富的理论与实践，所以我们无法面面俱到的进行讲解，而是以标题的两个问题作为切入点来达到本课时的目的，本课时的主要内容有：</p>
<ul>
<li>函数式编程与声明式编程。</li>
<li>你用什么声明？</li>
<li>你在声明什么？</li>
<li>函数式编程语言的特点。</li>
</ul>
<h3>函数式编程与命令式编程</h3>
<p>在 Spark 诞生之初，就有人诟病为什么 AMP 实验室选了一个如此小众的语言：Scala，很多人还将原因归结为这是由于学院派的高冷，但事实证明，选择 Scala 是非常正确的，Scala 很多特性与 Spark 本身理念非常契合，可以说天生一对。Scala 背后所代表的函数式编程思想也变得越来越为人所知。函数式编程思想早在 50 多年前就被提出，但当时的硬件性能太弱，并不能发挥出这种思想的优势。目前多核 CPU 大行其道，函数式编程在并发方面的优势也逐渐显示出了威力。这就好像在 Java 在被发明之初，总是有人说消耗内存太多、运行速度太慢，但是随着硬件性能的翻倍，Java 无疑是一种非常好的选择。</p>
<p><strong>函数式编程属于声明式编程的一种，与声明式编程相对的是命令式编程，命令式编程是按照“程序是一系列改变状态的命令”来建模的一种建模风格，而函数式编程思想是“程序是表达式和变换，以数学方程的形式建立模型，并且尽可能避免可变状态”</strong>。函数式编程会有一些类别的操作（算子），如映射、过滤或者归约，每一种都有不同的函数作为代表，如 filter、map、reduce。这些函数实现的是低阶变换（这里就是前面讲的算子），而用户定义的函数将作为这些函数的参数（这里可以理解为高阶变换）来实现整个方程。</p>
<p><strong>命令式编程将计算机程序看成动作的序列，程序运行的过程就是求解的过程</strong>，这就好比，阅读一段命令式编程风格的代码，如果不阅读到最后一行，一般来说无法确定程序的目的，这和题目求解过程有异曲同工之妙，在命令式编程中，解题过程由状态的转换来完成，而状态就是我们经常说的变量。而函数式编程则是从结果入手，用户通过函数定义了从最初输入到最终输出的映射关系，从这个角度上来说，用户编写代码描述了用户的最终结果（我想要什么），而并不关心（或者说不需要关心）求解过程，所以函数式编程绝对不会去操作某个具体的值，也就无所谓变量了。</p>
<p>举一个声明式编程的例子，这是用户编写的代码：</p>
<pre><code data-language="SQL" class="lang-SQL"><span class="hljs-keyword">SELECT</span> class_no, <span class="hljs-keyword">COUNT</span>(*) <span class="hljs-keyword">FROM</span> student_info <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> class_no
</code></pre>
<p>由于 SQL 是很典型的声明式编程，用户只需要告诉 SQL 引擎统计每个班的人数，至于底层是怎么执行的，用户不需要关心。在《数据库系统概论》（第五版）提到：数据库会把用户提交的 SQL 查询转化为等价的扩展关系代数表达式，用户用函数式编程的思想进行编码，其实就是直接描述这个关系代数表达式。</p>
<p>说了这么多函数式与声明式，让我们来看个例子，有一个数据清洗的任务，需要将姓氏集合中的单字符姓名（脏数据）去掉，并将首字母大写，最后再拼成一个逗号分隔的字符串，先来看看命令式的实现（Python 版）：</p>
<pre><code data-language="java" class="lang-java">family_names = [<span class="hljs-string">"ann"</span>,<span class="hljs-string">"bob"</span>,<span class="hljs-string">"c"</span>,<span class="hljs-string">"david"</span>]
clean_family_names = []
<span class="hljs-function"><span class="hljs-keyword">for</span> i in <span class="hljs-title">range</span><span class="hljs-params">(len(family_names)</span>):
    family_name </span>= family_names[i]
    <span class="hljs-keyword">if</span> (len(family_name) &gt; <span class="hljs-number">1</span>):
        clean_family_names.append(family_name.capitalize())
print clean_family_names
</code></pre>
<p>再来看看函数式（Scala 版）的实现：</p>
<pre><code data-language="java" class="lang-java">val familyNames = List(<span class="hljs-string">"ann"</span>,<span class="hljs-string">"bob"</span>,<span class="hljs-string">"c"</span>,<span class="hljs-string">"david"</span>)
println(
        familyNames.filter(p =&gt; p.length() &gt; <span class="hljs-number">1</span>).
        map(f =&gt; f.capitalize).
        reduce((a,b) =&gt; a + <span class="hljs-string">","</span> + b).toString()
)
</code></pre>
<p>从这个例子中我们可以看出，在命令式编程的版本中，只执行了一次循环，在函数式编程的版本里，循环执行了 3 次（filter、map、reduce），每一次只完成一种逻辑（用户编写的匿名函数），从性能上来说，当然前者更为优秀，这说明了在硬件性能羸弱时，函数式的缺点会被放大，但我们也看到了，在函数式编程的版本中不用维护外部状态 i，这种方式对于并行计算场景非常友好。</p>
<h3>你用什么声明</h3>
<p>在开始学习编程时，我相信当你看到这个代码时，内心某种程度上是抗拒的：</p>
<pre><code data-language="js" class="lang-js">x = x + <span class="hljs-number">1</span>
</code></pre>
<p>原因很简单，这个等式在数学上是不成立的。随着我们对某种编程语言理解程度加深，这种感觉很快就消失了。当提到函数式编程，我们还是习惯于将函数式编程中的“函数式”与某种编程语言的函数进行对应理解，但很遗憾，这么做对于理解函数式没什么帮助。而我们最初的感觉才是对的，也就是说，这里的函数式是指我们早已选择刻意遗忘的数学含义上的函数 f(x)。</p>
<p>其实，第 1 个问题的答案已经呼之欲出了。<strong>你用什么声明？很显然，函数式就是用函数进行声明的一种编程风格。这里的函数指的是数学含义上的函数。</strong></p>
<p>我们回过头来，看看 08 课时中的算子组成的管道。前面讲过，可以这样来理解：算子实现的是低阶变换，而用户定义的函数将作为这些算子的参数，也就是高阶函数（比如map 算子的函数参数）。这么理解当然没错，但是还是不够直接，我们可以试着从我们初中数学就学过的<strong>复合函数</strong>的角度来理解。</p>
<p>在严格的函数式编程中，所有函数都遵循数学函数的定义，必须有自变量（入参），必须有因变量（返回值）。用户定义的逻辑以高阶函数（high order）的形式体现，即用户可以将自定义函数以参数形式传入其他低阶函数中。其实从数学的角度上来说，这是很自然的，如下是一个数学表达式：</p>
<pre><code data-language="java" class="lang-java">y = sqrt (x + b)
</code></pre>
<p>括号中的函数 f1：x + b 作为参数传给函数 f2 = sqrt(x) ，这是初中的复合函数的用法。相对于高阶函数，函数式语言一般会提供一些低阶函数用于构建整个流程，这些低阶函数都是无副作用的，非常适合并行计算。高阶函数可以让用户专注于业务逻辑，而不需要去费心构建整个数据流。</p>
<p>到了这一步，我相信你能够重新看待你编写的 Spark 代码，你其实就是在用f(x)来声明你计算逻辑，只是这个f(x)可以看成是一个比较复杂的复合函数而已。你编写的代码就是在描述这个f(x),所以说，严格意义上，一个作业的所有代码，都可以用一行代码写完，也就是一个等式来表示。</p>
<h3>你在声明什么</h3>
<p>这个问题很有意思，我先不正面回答，先来和你聊聊一个小学奥数的话题：<strong>定义新运算</strong>，定义新运算是小学奥赛的一个考点，也就是说，同学们，只要你小学上过奥数，那么大概率应该是学习过这个内容的，我们来看看<a href="https://baike.baidu.com/item/%E5%AE%9A%E4%B9%89%E6%96%B0%E8%BF%90%E7%AE%97/1083945?fr=aladdin">百度百科</a>对定义新运算的定义：</p>
<blockquote>
<p>定义新运算是指用一个符号和已知运算表达式表示一种新的运算。定义新运算是一种特别设计的计算形式，它使用一些特殊的运算符号，这是与四则运算中的加减乘除符号是不一样的。新定义的算式中有括号的，要先算括号里的，但它在没有转化前，是不适合于各种运算的。</p>
<p>小学六年级奥数中体现的问题，解题方法较简单。解答定义新运算，关键是要正确地理解新定义运算的算式含义。然后严格按照新定义运算的计算程序，将数值代入，转化为常规的四则运算算式进行计算。</p>
<p>定义新运算是一种特殊设计的运算形式，它使用的是一些特殊的运算符号，如：*、Δ 等，这是与四则运算中的加减乘除不同的。 这里有一个问题：当 a≥b=b 时 ab=bxb 当 a&lt;b=a 时 ab=a 当 x=2 时，求 (1x)-(3x) 的值。</p>
<p>解题方法如下：<br>
3△2=3+2+6=11<br>
5△5=5+5+25=35<br>
设 a<em>b=﹙a+b﹚÷3<br>
则 6</em>﹙5*4﹚=3</p>
</blockquote>
<p>相信你能够完全理解上面所说的内容。这里注意，前面一直说的算子，它的英文是 operator，大多数资料翻译为算子，看起来信达雅，但其实掩盖了 operator 的本质，operator 如果直译应该被翻译为运算符。你编写的 Spark 代码与新运算的定义不谋而合，也是用一些已知的<strong>运算符</strong>（Spark 算子）在定义一种新运算。</p>
<p>现在，结合用运算符和定义新运算的概念，我想你应该能够回答这个问题：你在声明什么？答案很简单：<strong>用运算符定义一种新运算</strong>。</p>
<h3>函数式编程语言的一些特点</h3>
<p>作为函数式编程思想的实现，函数式编程语言都有一些有趣的共性，另外，<strong>如果你把 Spark 看成一门函数式编程语言的话，你会发现，这些共性仍然存在</strong>。</p>
<h4>低阶函数与核心数据结构</h4>
<p>如果使用低阶函数与高阶函数来完成我们的程序，这时其实就是将程序控制权让位于语言，而我们专注于业务逻辑。这样做的好处还在于，有利于程序优化，享受免费的性能提升午餐。比如语言开发者专注于优化低阶函数，而应用开发者则专注于优化高阶函数。低阶函数是复用的，因此当低阶函数性能提升时，程序不需要改一行代码就能免费获得性能提升。此外，函数式编程语言通常只提供几种核心数据结构，供开发者选择，它希望开发者能基于这些简单的数据结构组合出复杂的数据结构，这与低阶函数的思想是一致的，很多函数式编程语言的特性会着重优化低阶函数与核心数据结构。但这与面向对象的命令式编程是不一样的，在 OOP 中，面向对象编程的语言鼓励开发者针对具体问题建立专门的数据结构。</p>
<p>通过前几个课时的内容可以看到，Spark 的核心数据结构只有一个，就是 RDD，而其他函数式编程语言，如 Scala，核心数据结构也非常少。</p>
<h4>惰性求值</h4>
<p>惰性求值（lazy evaluation）是函数式编程语言常见的一种特性，通常指尽量延后求解表达式的值，这样可以对开销大的计算按需计算，利用惰性求值的特性可以构建无限大的集合。惰性求值可以用闭包来实现。Spark 也是采用了惰性求值来触发计算。</p>
<h4>函数记忆</h4>
<p>由于在函数式编程中，函数本身是无状态的，因此给定入参，一定能得到一定的结果。基于此，函数式语言会对函数进行记忆或者缓存，以斐波那契数列举例，首先用尾递归来实现对斐波那契数列求和，Python 代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function">def <span class="hljs-title">Fibonacci</span> <span class="hljs-params">( n )</span>:
   <span class="hljs-keyword">if</span> n </span>== <span class="hljs-number">0</span> :
      res = <span class="hljs-number">0</span>
   elif num == <span class="hljs-number">1</span>:
      res = <span class="hljs-number">1</span>
   <span class="hljs-keyword">else</span>:
      res = Fibonacci ( n - <span class="hljs-number">1</span> ) + Fibonacci ( n - <span class="hljs-number">2</span> )
<span class="hljs-keyword">return</span> res
</code></pre>
<p>当 n 等于 4 时，程序执行过程是：</p>
<pre><code data-language="java" class="lang-java">Fibonacci (<span class="hljs-number">4</span>)
Fibonacci (<span class="hljs-number">3</span>)
Fibonacci (<span class="hljs-number">2</span>)
Fibonacci (<span class="hljs-number">1</span>)
Fibonacci (<span class="hljs-number">0</span>)
Fibonacci (<span class="hljs-number">1</span>)
Fibonacci (<span class="hljs-number">2</span>)
Fibonacci (<span class="hljs-number">1</span>)
Fibonacci (<span class="hljs-number">0</span>)
</code></pre>
<p>为了求 Fibonacci (4)，我们执行了 1 次 Fibonacci(3)，2 次 Fibonacci(2)，3 次 Fibonacci(1)，2 次 Fibonacci(0)，一共 8 次计算，在函数式编程语言中，执行过程是这样的：</p>
<pre><code data-language="java" class="lang-java">Fibonacci (<span class="hljs-number">4</span>)
Fibonacci (<span class="hljs-number">3</span>)
Fibonacci (<span class="hljs-number">2</span>)
Fibonacci (<span class="hljs-number">1</span>)
Fibonacci (<span class="hljs-number">0</span>)
</code></pre>
<p>一共只用 4 次计算就可求得 Fibonacci(4)，后面执行的 Fibonacci(0)、Fibonacci(1) 由于函数式编程语言已经缓存了结果，因此不会重复计算。</p>
<p>在这里，你可以与 cache 算子代表的缓存机制联系起来，Spark 允许用户主动缓存。</p>
<h3>小结</h3>
<p>本课时的内容偏理论，但是我相信能够引起你一些深入思考，有些时候，务实之后的务虚会收获更多。这里给你留一个思考题：用函数式风格的代码实现 a - b 的逻辑，逻辑非常简单，难点在于用函数式风格来表达。考虑这道题的特殊性，可能大多数不习惯这么表达，所以我把答案附在下面。</p>
<pre><code data-language="java" class="lang-java">List(<span class="hljs-number">5</span>).zip(List(<span class="hljs-number">4</span>)).map(f =&gt; {f._1 - f._2}).foreach(println(_))
</code></pre>
<p>你可以思考下这么写的优点与缺点，欢迎与我在学习群中互动。</p>

---

### 精选评论

##### *鹏：
> 谁能解释一下这个奥数题？我百度到了文档，越看文档我越觉得扯。。。。

##### **乐：
> 学习群是多少啊

##### **松：
> 我能够理解的是，所谓的命令式编程则是需要告诉程序怎么做，有几个详细的步骤，所以需要很多编写的过程步骤。而函数式编程则需要告诉程序有什么输入和结果，至于期间中的步骤我们并不需要关心。我们只需要通过所谓的低阶函数的组合使用来达到自定义的输入和输出就行了。

##### **学：
> 程序是表达式和转换，我们通过运用低阶运算符来表达一种新的运算

##### *强：
> 也可改为小括号类似这样：res0.map(f =&gt; (f._1-f._2)).foreach(println(_))<br>优点：简洁流畅，逻辑表达清晰<br>缺点：难以调试

##### **强：
> Very good!😆

