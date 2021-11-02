<p style="text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">你好，我是你的数据结构课老师蔡元楠，欢迎进入第 10 课时的内容“树在 Amazon 中的应用”。</span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这一讲的内容以 Amazon 云服务为例，来分享分布式 SQL 数据库如何使用树这样的数据结构优化 SQL 执行的。虽然我迫不及待地想和你直接深入底层技术原理，但是在讲解非常酷的技术细节之前，我们还是先回过头看一下什么是 SQL 执行优化？为什么要优化 SQL 执行？</span></p>
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>什么是 SQL</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">SQL（Structured Query Language）在硅谷科技公司一般读作 sequel，其实在 1970 年代 IBM 发明 SQL 的时候它本来被称作 SEQUEL（Structured English Query Language），但后来因为 SEQUEL 作为商标已经被注册了，只能改名为 SQL。不熟悉这段历史的人有时候会读作 S-Q-L（分开三个字母读），虽然也听得懂的，但没有那么地道了！</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">SQL 被用来管理和查询关系型数据库。比如：</span></p>
<pre>SELECT&nbsp;name&nbsp;FROM&nbsp;table;</pre>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这样一行 SQL 可以选取下表中所有的 name 这一列：</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/61/DA/CgpOIF4f_qCAFrrXAABVuB8FTZk278.png">执行这行 SQL 指令的输出一般就是 Alex、David、Patrick。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">SQL 在 20 世纪 80 年代成为了 ANSI/ISO 国际标准。但即使是国际标准，不同的关系型数据库的实现都有很多不同，比如 MySQL、PostgreSQL 对于 SQL 标准的实现就不同。更不用说在专门优化的大型分布式数据库就更不一样了，比如本文要介绍的 Amazon 云服务 AWS 里所使用的 SQL 执行引擎，就拥有很多独门秘籍的实现。</span></p>
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>为什么要优化 SQL 的执行</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们已经了解了 SQL 最常用的用途就是查询数据。然而关系型数据库在执行 SQL 语句时必须要考虑效率，即：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">需要多长时间完成这条 SQL 查询？</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">需要多少计算资源，包括 CPU、磁盘、电能去执行这条 SQL 语句？</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">关系型数据库具体怎样执行 SQL 语句我们称之为执行计划（Execution Plan），很不幸的是同样一条 SQL 语句的可选执行计划非常多。我们来看一个例子：</span></p>
<pre>SELECT&nbsp;*
FROM&nbsp;customers&nbsp;c,&nbsp;orders&nbsp;o
WHERE&nbsp;c.id&nbsp;=&nbsp;o.cust_id&nbsp;AND&nbsp;c.creation_timestamp&nbsp;&lt;&nbsp;1579069516;</pre>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这个例子的 SQL 从语义上理解的话，是想要在 customers 和 orders 这两种表中，找出所有 customers 的 creation_timestamp 早于某个时间的所有数据，看起来似乎很好理解啊。其实背后的 SQL 引擎在执行它的时候是有很多纠结的，纠结什么呢？就是选择执行计划。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">由于篇幅有限，在这里就不再罗列所有可能的执行计划了。但是为了阐述我的观点，以下列举了几个执行计划的例子：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Creation_timestamp 小于这样一个过滤条件应该在 JOIN 语句前执行呢，还是在 JOIN 语句之后执行？</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">对于 JOIN 语句我们应该使用 merge join，还是 hash join，还是用建立的 index 使用 nested loop join 呢？</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果是 hash join 或者 nested loop join 的话，我们应该是先遍历 cutomers 表然后查找对应的 orders，还是先遍历 orders 表再查找对应的 customers 呢？</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果我们对于 creation_timestamp 建立有二级索引（Secondary Index）的话，我们是应该用二级索引查找 creation_timestamp 还是用主索引查找 ID 比较快？</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">作为数据库 SQL 引擎，这些问题的答案都是相互依赖的，也都是需要交叉考虑的。比如说如果选择 nested loop join，可能搭配使用二级索引比较好；但是如果使用 merge join 的话，可能还是使用主索引比较好。最优的执行计划还取决于我们的表有多少行，各种物理操作的相对速度，还有不同数据的存储位置和数据值的分布等，可以说要考虑的参数是非常非常的多！</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">所以在 Amazon AWS 这样大型的分布式 SQL 数据库中，对 SQL 的执行计划进行了大量的优化。一个简单的 SQL 执行引擎模型是这样子的：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先是 SQL 解析器（Parser），它负责把用户输入的 SQL 解析成 SQL 语法树（AST），对的，就是我们讲的树！</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">后面的 SQL 优化器（Optimizer）接受前面解析的原生语法树，对它进行优化重写语法树和执行计划。一般优化器不仅仅会看语法树，还需要结合特定的用户数据库配置，数据实际分布进行优化。</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">最后面的 SQL 执行器（Executor）才会去真正的执行优化重写的 SQL 语法树。</span></p></li>
</ul>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/61/DA/CgpOIF4f_r2AfVt5AAAtd-U383M254.png"></span></p>
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>SQL 语法树是个什么树</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">AST（Abstract Syntax Tree）即语法树，在计算机科学中是用树来表达源代码的一种方式。我们理解的编译器很大一部分工作就是把源码表示成语法树。不过在这里不展开语法树的讲解，因为这实在是一个巨大深奥的计算机科学主题，这一讲会专注在理解 SQL 这个语言的 AST 表达上。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们来看一个很简单的例子：</span></p>
<pre>SELECT&nbsp;name&nbsp;FROM&nbsp;table;</pre>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">还是在我们刚才表中选取 name 这一列。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><img src="https://s0.lgstatic.com/i/image3/M01/61/DB/Cgq2xl4f_uSAGzwqAABVuB8FTZk049.png"></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果用语法树表达的话会是下面图中的样子，这个树的根是 SELECT 节点，在下面左子树是 name，右子树是 FROM 节点为根的子树。FROM 节点下面是 table 叶子节点。</span><br></p>
<p style="text-align:center;line-height: 2;line-height: 115%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/61/DB/Cgq2xl4f_vCAelA_AADz55RQ5Gk214.png"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在用 AST 表达 SQL 语句时，SQL 操作符永远是子树的根，而树的叶子则是比如这里的 name 或者 table。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">用 AST 解析 SQL 语句之后，我们对于 SQL 语句的分析和优化就变得更为直接了。你可以很快找出这个 SQL 语句的操作就是 SELECT 和 FROM，他们都是子树的根节点。每一个 SQL 语句中的 token 在语法书中都拥有了语义上的含义，比如 from 和 name 不仅仅是单词不一样，他们在语义上是不同的含义，from 是操作，而 name 是一张表中的列名称。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们可以根据解析后的语法树对这棵树做进一步的修改，比如在 FROM 下面的子节点我们知道是一个表的名字，可以把表的完整路径解析出来；再比如我们知道 SELECT 下面的左子树是一个表的列名称，可以把完整的表名称解析出来。在做完这些名字的解析之后，这个 SQL 语法树就变成了如下图所示的样子。</span></p>
<p style="text-align:center;line-height: 2;line-height: 115%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/61/DA/CgpOIF4f_vuAS2ZLAAFUDkBSqfc827.png"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们可以看到，原来的 name 节点变成了 schema.table.name，而原来的 table 节点变成了 schema.table，这些名称的解析只需要通过操作树的节点就可以了，是不是很方便啊！</span></p>
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>树的序列化和 S-expression</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们已经知道了在大型分布式树的数据库中，SQL 的语法树解析是整个 SQL 引擎的第一步。那么在后续的优化器和执行器中我们是怎样传递这棵树的呢？要知道，在 Amazon 分布式数据库中，众多系统不仅仅是几个简单的函数而已，往往是好几个不同的服务器组合而成，在不同的服务器 RPC 之间传递树，我们需要把树序列化成可以在网络中传输解析的格式。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这就是 S-expression 了，S-expression 有时候也简称为 sexp，最初是由 Lisp 语言发明并被人们广泛使用。现在的 S-expression 已经有很多变种，常见的 S-expression 被定义为：</span></p>
<ul>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">一个不可分的元素（atom）</span></p></li>
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">或者是 “(x y)”，左括号，x，y，右括号的形式，其中 x 和 y 都是 S-expression</span></p></li>
</ul>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">其实从这个定义来看的话，也和我们树的定义非常像，都是递归式定义。我们来看看刚才 SQL 语法树用 S-expression 怎样表示？还记得我们 SQL 语法树的根是一个 SELECT 节点嘛，下面还有一个以 FROM 为根节点的子树。你也许想到了我们可能需要一个嵌套的 S-expression，的确如此，正是像下面这样：</span><br></p>
<pre>(SELECT&nbsp;schema.table.name&nbsp;(FROM&nbsp;schema.table))</pre>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们需要有一个嵌套的括号，在这里，括号的第一个 token 被定义成这个括号表达的子树的根，每一个嵌套的括号就是一个子树。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">S-expression 的反序列化也很简单，只需要把每一个括号展开成一个子树就可以了！这样你就可以在系统中的不同服务之间用 RPC 传递这样的 S-expression。把一些中间计算的树传递给之后的系统服务。</span></p>
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>总结</strong></span></p></h3>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这一讲我们学习了树在 Amazon 这样的超大型分布式数据库系统中是如何应用的。首先了解了为什么超大型分布式数据库需要对 SQL 引擎进行优化；然后分析了怎么解析 SQL 语法树，怎样用树表达一个 SQL 语句，为什么树的表达能给我们处理 SQL 语句带来便利；最后学习了 S-expression 怎样序列化表达通用的树。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OK，这节课就讲到这里啦，下一课时我将分享“平衡树的性能优化”，记得按时来听课哈。</span></p>

---

### 精选评论


