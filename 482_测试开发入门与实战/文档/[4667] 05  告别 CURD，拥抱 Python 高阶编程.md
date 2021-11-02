<p data-nodeid="1141" class="">上节课我们一起学习了 Python 的基础编程知识，本课时我将带你继续进阶，向你讲解 Python 的高阶编程知识。</p>
<p data-nodeid="1142">我想，你在自主开发测试框架的过程中，经常会碰见这样的困惑：</p>
<ul data-nodeid="1143">
<li data-nodeid="1144">
<p data-nodeid="1145">我仅仅想运行带着某些特定标签的测试用例，但是我不知道具体哪些用例带着这些标签，我该怎么做？</p>
</li>
<li data-nodeid="1146">
<p data-nodeid="1147">我想给我的每一个函数都增加个打印功能，但是我又不想改动函数本身，该怎么做？</p>
</li>
<li data-nodeid="1148">
<p data-nodeid="1149">我想让测试框架根据用户输入，做出不同的处理反应，但是我的输入不是一成不变的，我输入的参数多一些或者少一些，框架就报错了，该怎么办？</p>
</li>
</ul>
<p data-nodeid="1150">这些问题看起来是一个个不同的业务需求，但它们的背后，其实对应着 Python 语言中的一个个高阶编程技巧。</p>
<p data-nodeid="1151">这些技巧，就好比是绝世武功中的内功心法和武功秘籍， 所谓“万丈高楼平地起”，掌握这些高阶技巧，有助你开发出更优秀的测试框架。下面我们就一起来看一看，Python 中的这些内功心法有哪些？</p>
<h3 data-nodeid="1152">列表表达式（List Comprehension）</h3>
<p data-nodeid="1153">俗话说“人生苦短，我用 Python”，Python 为了简化程序的代码行数做了很多努力，其中最经典的就是列表表达式。</p>
<p data-nodeid="1154">比如我有如下函数，用来输出一个单词中的所有字符：</p>
<pre class="lang-yaml" data-nodeid="1155"><code data-language="yaml"><span class="hljs-string">def</span> <span class="hljs-string">output_letter(letter):</span>
    <span class="hljs-string">l</span> <span class="hljs-string">=</span> <span class="hljs-string">[]</span>
    <span class="hljs-attr">for item in letter:</span>
        <span class="hljs-string">l.append(item)</span>
    <span class="hljs-string">return</span> <span class="hljs-string">l</span>
<span class="hljs-string">if</span> <span class="hljs-string">__name__</span> <span class="hljs-string">==</span> <span class="hljs-attr">"__main__":</span>
    <span class="hljs-string">print(output_letter('kevin'))</span>
<span class="hljs-comment">#此方法的输出为：</span>
<span class="hljs-string">['k',</span> <span class="hljs-string">'e'</span><span class="hljs-string">,</span> <span class="hljs-string">'v'</span><span class="hljs-string">,</span> <span class="hljs-string">'i'</span><span class="hljs-string">,</span> <span class="hljs-string">'n]
</span></code></pre>
<p data-nodeid="1156">Python 觉得这样写代码行数太多了，不优雅，于是有了如下的写法：</p>
<pre class="lang-java" data-nodeid="1157"><code data-language="java">[expression <span class="hljs-keyword">for</span> item in list]
</code></pre>
<p data-nodeid="1158">对应于我们的函数就变成了：</p>
<pre class="lang-yaml" data-nodeid="1159"><code data-language="yaml"><span class="hljs-string">def</span> <span class="hljs-string">output_letter(letter):</span>
   <span class="hljs-string">return</span> <span class="hljs-string">[l</span> <span class="hljs-string">for</span> <span class="hljs-string">l</span> <span class="hljs-string">in</span> <span class="hljs-string">letter]</span>

<span class="hljs-string">if</span> <span class="hljs-string">__name__</span> <span class="hljs-string">==</span> <span class="hljs-attr">"__main__":</span>
    <span class="hljs-string">print(output_letter('kevin'))</span>

<span class="hljs-comment">#此方法的输出为：</span>
<span class="hljs-string">['k',</span> <span class="hljs-string">'e'</span><span class="hljs-string">,</span> <span class="hljs-string">'v'</span><span class="hljs-string">,</span> <span class="hljs-string">'i'</span><span class="hljs-string">,</span> <span class="hljs-string">'n'</span><span class="hljs-string">]</span>
</code></pre>
<p data-nodeid="1160">是不是瞬间少了很多代码，逻辑也更清晰？不仅如此，Python 还允许我们在列表表达式中进行判断。</p>
<pre class="lang-java" data-nodeid="1161"><code data-language="java">[expression <span class="hljs-keyword">for</span> item in list <span class="hljs-keyword">if</span> xxx <span class="hljs-keyword">else</span> yyy]
</code></pre>
<p data-nodeid="1162">例如我有一个列表，里面包括多个字符，我希望返回那些包含字母 k 的字符。</p>
<pre class="lang-python" data-nodeid="1163"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">output_letter</span>(<span class="hljs-params">letter</span>):</span>
   <span class="hljs-keyword">return</span> [l <span class="hljs-keyword">for</span> l <span class="hljs-keyword">in</span> letter <span class="hljs-keyword">if</span> <span class="hljs-string">'k'</span> <span class="hljs-keyword">in</span> l]
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    print(output_letter([<span class="hljs-string">'kevin'</span>, <span class="hljs-string">'did'</span>, <span class="hljs-string">'automation'</span>, <span class="hljs-string">'well'</span>]))
</code></pre>
<p data-nodeid="1164">列表表达式可以使我们的函数非常简洁易懂，并且减少代码量。</p>
<h3 data-nodeid="1165">匿名函数（lambda）</h3>
<p data-nodeid="1166">除了列表表达式可以减少代码量以外，Python 中还提供了匿名函数，当你的函数逻辑非常少时，你无须再定义一个函数，可采用匿名函数来减少代码量。匿名函数的语法如下：</p>
<pre class="lang-java" data-nodeid="1167"><code data-language="java">lambda arguments : expression
</code></pre>
<p data-nodeid="1168">举例来说，我们有一个函数，用来得出列表中的每一个元素的平方，正常的写法是这样的：</p>
<pre class="lang-java" data-nodeid="1169"><code data-language="java"><span class="hljs-function">def <span class="hljs-title">square</span><span class="hljs-params">(l)</span>:
    square_list </span>= []
    <span class="hljs-keyword">for</span> ele in l:
        square_list.append(ele * ele)
    <span class="hljs-keyword">return</span> square_list

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    print(square([<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>]))
</code></pre>
<p data-nodeid="1170">用了 lambda 后，是这样的：</p>
<pre class="lang-java" data-nodeid="1171"><code data-language="java">a = lambda l: [item * item <span class="hljs-keyword">for</span> item in l]

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    print(a([<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>]))
</code></pre>
<p data-nodeid="1172">匿名函数大大地减少了代码工作量，但是也会让代码的可读性降低，所以通常逻辑不复杂的函数，可以考虑使用匿名函数。</p>
<h3 data-nodeid="1173">自省/反射（Reflection）</h3>
<p data-nodeid="1174">在编程中，自省是一种在运行时查找有关对象的信息的能力；而反射则更进一步，它使对象能够在运行时进行修改。</p>
<p data-nodeid="1175">自省和反射是 Python 中非常重要的概念，我们可以通过自省和反射来实现很多高级功能，例如动态查找待运行测试用例。</p>
<p data-nodeid="1176">自省最经典的用法就是查看对象类型。</p>
<h4 data-nodeid="1177">1.type</h4>
<pre class="lang-python" data-nodeid="1178"><code data-language="python"><span class="hljs-comment">#返回对象类型</span>
type（obj）
</code></pre>
<p data-nodeid="1179">比如：</p>
<pre class="lang-java" data-nodeid="1180"><code data-language="java">&gt;&gt;&gt; type(<span class="hljs-number">7</span>)
&lt;<span class="hljs-class"><span class="hljs-keyword">class</span> '<span class="hljs-title">int</span>'&gt;
&gt;&gt;&gt; <span class="hljs-title">type</span>(2.0)
&lt;<span class="hljs-title">class</span> '<span class="hljs-title">float</span>'&gt;
&gt;&gt;&gt; <span class="hljs-title">type</span>(<span class="hljs-title">int</span>)
&lt;<span class="hljs-title">class</span> '<span class="hljs-title">type</span>'&gt;
</span></code></pre>
<p data-nodeid="1181">type() 函数的返回值，我们称为类型对象，类型对象告诉我们参数属于哪种类对象实例。如上文所示，解释器便在告诉我们整数 7 属于 int 类，2.0 属于 float 类，而 int 属于类类型。</p>
<pre class="lang-python" data-nodeid="1182"><code data-language="python">type() 常常跟函数isinstance() 配合使用，用来检测一个变量是否是我们需要的类型：
<span class="hljs-comment">#下述例子判断给定的数字是否整型(int类)</span>
x = <span class="hljs-number">6</span>
<span class="hljs-keyword">if</span> isinstance(x, int):
    print(<span class="hljs-string">'I am int'</span>)
    <span class="hljs-comment">#你的逻辑</span>
</code></pre>
<p data-nodeid="1183">自省还有以下其他几种用法。</p>
<h4 data-nodeid="1184">2.dir</h4>
<p data-nodeid="1185">dir() 可以用来获取当前模块的属性列表，也可以获取指定一个属性。</p>
<pre class="lang-python" data-nodeid="1186"><code data-language="python"><span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    my_list = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>]
    print(dir(my_list))
    print(dir(my_list).__class__)
</code></pre>
<p data-nodeid="1187">比如我们运行上述代码，则会有如下结果。</p>
<pre class="lang-dart" data-nodeid="1188"><code data-language="dart">#第一个<span class="hljs-built_in">print</span>返回
[<span class="hljs-string">'__add__'</span>, <span class="hljs-string">'__class__'</span>, <span class="hljs-string">'__contains__'</span>, <span class="hljs-string">'__delattr__'</span>, <span class="hljs-string">'__delitem__'</span>, <span class="hljs-string">'__dir__'</span>, <span class="hljs-string">'__doc__'</span>, <span class="hljs-string">'__eq__'</span>, <span class="hljs-string">'__format__'</span>, <span class="hljs-string">'__ge__'</span>, <span class="hljs-string">'__getattribute__'</span>, <span class="hljs-string">'__getitem__'</span>, <span class="hljs-string">'__gt__'</span>, <span class="hljs-string">'__hash__'</span>, <span class="hljs-string">'__iadd__'</span>, <span class="hljs-string">'__imul__'</span>, <span class="hljs-string">'__init__'</span>, <span class="hljs-string">'__init_subclass__'</span>, <span class="hljs-string">'__iter__'</span>, <span class="hljs-string">'__le__'</span>, <span class="hljs-string">'__len__'</span>, <span class="hljs-string">'__lt__'</span>, <span class="hljs-string">'__mul__'</span>, <span class="hljs-string">'__ne__'</span>, <span class="hljs-string">'__new__'</span>, <span class="hljs-string">'__reduce__'</span>, <span class="hljs-string">'__reduce_ex__'</span>, <span class="hljs-string">'__repr__'</span>, <span class="hljs-string">'__reversed__'</span>, <span class="hljs-string">'__rmul__'</span>, <span class="hljs-string">'__setattr__'</span>, <span class="hljs-string">'__setitem__'</span>, <span class="hljs-string">'__sizeof__'</span>, <span class="hljs-string">'__str__'</span>, <span class="hljs-string">'__subclasshook__'</span>, <span class="hljs-string">'append'</span>, <span class="hljs-string">'clear'</span>, <span class="hljs-string">'copy'</span>, <span class="hljs-string">'count'</span>, <span class="hljs-string">'extend'</span>, <span class="hljs-string">'index'</span>, <span class="hljs-string">'insert'</span>, <span class="hljs-string">'pop'</span>, <span class="hljs-string">'remove'</span>, <span class="hljs-string">'reverse'</span>, <span class="hljs-string">'sort'</span>]
#第二个<span class="hljs-built_in">print</span>返回
&lt;<span class="hljs-class"><span class="hljs-keyword">class</span> '<span class="hljs-title">list</span>'&gt;
</span></code></pre>
<h4 data-nodeid="1189">3. id</h4>
<p data-nodeid="1190"><strong data-nodeid="1318">id()</strong> 函数返回对象的唯一标识符。</p>
<pre class="lang-python" data-nodeid="1191"><code data-language="python"><span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    name = <span class="hljs-string">"kevin"</span>
    print(id(name))
<span class="hljs-comment">#输出</span>
<span class="hljs-number">140245720259120</span>
</code></pre>
<h4 data-nodeid="1192">4.inspect</h4>
<p data-nodeid="1193">inspect 模块提供了一些有用的函数帮助获取对象的信息，例如模块、类、方法、函数、回溯、帧对象，以及代码对象。</p>
<p data-nodeid="1194">例如它可以帮助你检查类的内容，获取某个方法的源代码，取得并格式化某个函数的参数列表，或者获取你需要显示的回溯的详细信息。</p>
<p data-nodeid="1195">inspect 有很多函数，我以一个实际例子为依托，介绍常用的几种。假设现在我们有个项目，它的文件结构是这样的：</p>
<pre class="lang-yaml" data-nodeid="1196"><code data-language="yaml"><span class="hljs-string">testProject</span>
  <span class="hljs-string">--|tests</span>
      <span class="hljs-string">--|__init__.py</span>
      <span class="hljs-string">--|test1.py</span>
      <span class="hljs-string">--|test2.py</span>
</code></pre>
<p data-nodeid="1197">其中，test1.py 的内容如下：</p>
<pre class="lang-python" data-nodeid="1198"><code data-language="python"><span class="hljs-keyword">import</span> inspect
<span class="hljs-keyword">from</span> tests.test2 <span class="hljs-keyword">import</span> hello
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KevinTest</span>():</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-params">self</span>):</span>
        print(<span class="hljs-string">'i am kevin cai'</span>)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">say_hello</span>(<span class="hljs-params">self, name</span>):</span>
        hello()
        <span class="hljs-keyword">return</span> <span class="hljs-string">'Hello {name}'</span>.format(name=name)
</code></pre>
<p data-nodeid="1199">test2.py 内容如下：</p>
<pre class="lang-python" data-nodeid="1200"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">hello</span>():</span>
    print(<span class="hljs-string">'hello from test2'</span>)
</code></pre>
<ul data-nodeid="1201">
<li data-nodeid="1202">
<p data-nodeid="1203"><strong data-nodeid="1328">inspect.getmodulename</strong></p>
</li>
</ul>
<p data-nodeid="1204">inspect.getmodulename(path) 用来获取指定路径下的 module 名。</p>
<pre class="lang-python" data-nodeid="1205"><code data-language="python"><span class="hljs-comment"># 在test1.py中新增如下代码。</span>
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment">#此打印语句输出test1。 即当前模块名是test1</span>
    print(inspect.getmodulename(__file__))
</code></pre>
<ul data-nodeid="1206">
<li data-nodeid="1207">
<p data-nodeid="1208"><strong data-nodeid="1333">inspect.getmodule</strong></p>
</li>
</ul>
<p data-nodeid="1209">inspect.getmodule(object) 用来返回 object 定义在哪个 module 中。</p>
<pre class="lang-python" data-nodeid="1210"><code data-language="python"><span class="hljs-comment"># 在test1.py中新增如下代码。</span>
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment">#此语句输出&lt;module 'tests.test2' from '/Users/kevin/automation/testProjectPython/tests/test2.py'&gt;</span>
    print(inspect.getmodule(hello))
</code></pre>
<ul data-nodeid="1211">
<li data-nodeid="1212">
<p data-nodeid="1213"><strong data-nodeid="1338">inspect.getfile</strong></p>
</li>
</ul>
<p data-nodeid="1214">inspect.getfile(object) 用来返回 object 定义在哪个 file 中。</p>
<pre class="lang-python" data-nodeid="1215"><code data-language="python"><span class="hljs-comment"># 在test1.py中新增如下代码。</span>
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>
    test = KevinTest()
    <span class="hljs-comment">#此语句输出/Users/kevin/automation/testProjectPython/tests/test1.py</span>
    print(inspect.getfile(test.say_hello))
</code></pre>
<ul data-nodeid="1216">
<li data-nodeid="1217">
<p data-nodeid="1218"><strong data-nodeid="1343">inspect.getmembers</strong></p>
</li>
</ul>
<p data-nodeid="1219">inspect.getmembers(object) 用来返回 object 的所有成员列表（为 (name, value) 的形式）。</p>
<pre class="lang-python" data-nodeid="1220"><code data-language="python"><span class="hljs-comment"># 在test1.py中新增如下代码。</span>
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    test = KevinTest()
    <span class="hljs-comment">#此语句输出test里的所有是方法的成员变量。输出是一个列表</span>
    <span class="hljs-comment">#[('__init__', &lt;bound method KevinTest.__init__ of &lt;__main__.KevinTest object at 0x10911ef28&gt;&gt;), ('say_hello', &lt;bound method KevinTest.say_hello of &lt;__main__.KevinTest object at 0x10911ef28&gt;&gt;)]</span>
    print(inspect.getmembers(test, inspect.ismethod))
</code></pre>
<h3 data-nodeid="1221">闭包（closure）</h3>
<p data-nodeid="1222">闭包是一个概念，是指在能够读取其他函数内部变量的函数。这个定义比较抽象，我们来看一段代码：</p>
<pre class="lang-python" data-nodeid="1223"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">outer</span>():</span>
    cheer = <span class="hljs-string">'hello '</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inner</span>(<span class="hljs-params">name</span>):</span>
        <span class="hljs-keyword">return</span> cheer + name
    <span class="hljs-keyword">return</span> inner

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment">#输出hello kevin</span>
    print(outer()(<span class="hljs-string">'kevin'</span>))
</code></pre>
<p data-nodeid="1224">以上代码的意思如下：我定义了一个外部函数 outer 和一个内部函数 inner；在外部函数 outer 内部，我又定义了一个局部变量 cheer（并给定初始值为hello）；然后我在内部函数 inner 里引用了这个局部变量 cheer。最后 outer 函数的返回值是 inner 函数本身。</p>
<p data-nodeid="1225">在本例的调用里，outer 函数接受了两个参数，第一个参数为空，第二个参数为 kevin。那么outer() 的返回值就是 inner。所以 outer()('kevin') 的返回值就是 inner('kevin')。</p>
<p data-nodeid="1226">为了方便你理解，我贴出这个函数的运行过程：</p>
<ul data-nodeid="1227">
<li data-nodeid="1228">
<p data-nodeid="1229">当代码运行时，首先执行的是入口函数，即第 8 行代码，接着是第 10 行代码。</p>
</li>
</ul>
<p data-nodeid="1230"><img src="https://s0.lgstatic.com/i/image/M00/55/2C/CgqCHl9pzJ2AGcCRAAEDr2CYkic136.png" alt="Drawing 0.png" data-nodeid="1361"></p>
<ul data-nodeid="1231">
<li data-nodeid="1232">
<p data-nodeid="1233">继续向后执行，就会进入到第 1 行代码，即 outer() 函数内部；接着第 2 行代码开始执行，变量cheer被定义，并且赋值为“hello”；接着第 3 行代码开始运行，需要注意的是，第 3 行代码执行完，并不会执行第 4 行代码，而是执行第 5 行代码。</p>
</li>
</ul>
<p data-nodeid="1234"><img src="https://s0.lgstatic.com/i/image/M00/55/2C/CgqCHl9pzKSAbpBPAAFuLtfzxWU099.png" alt="Drawing 1.png" data-nodeid="1365"></p>
<ul data-nodeid="1235">
<li data-nodeid="1236">
<p data-nodeid="1237">第 5 行代码执行完毕后，outer() 函数的整个生命周期就已经结束了，继续往后执行：</p>
</li>
</ul>
<p data-nodeid="1238"><img src="https://s0.lgstatic.com/i/image/M00/55/21/Ciqc1F9pzKqAVnrtAAFpN4pwEGg451.png" alt="Drawing 2.png" data-nodeid="1369"></p>
<p data-nodeid="1239">可以看到，代码进入了 inner 函数内部，而且 inner 函数内部可以访问生命周期已经结束的 outer 函数的成员变量 cheer，这个就是闭包的魔力。</p>
<p data-nodeid="1240">最后，inner 函数继续执行，outer 函数里定义的 cheer 被取出，并且连同 name 一起返回。我们就获得到了函数的最终结果“hello kevin”。</p>
<p data-nodeid="1241"><img src="https://s0.lgstatic.com/i/image/M00/55/2C/CgqCHl9pzLGAdIdCAAF2FPx794k879.png" alt="Drawing 3.png" data-nodeid="1374"></p>
<p data-nodeid="1242">了解了闭包如何起作用的，我来总结下闭包的特点。</p>
<p data-nodeid="1243"><strong data-nodeid="1379">闭包的特点：</strong></p>
<ul data-nodeid="1244">
<li data-nodeid="1245">
<p data-nodeid="1246">在一个外部函数里定义一个内部函数，且内部函数里包含对外部函数的访问（即使外部函数生命周期结束后，内部函数仍然可以访问外部函数变量）；</p>
</li>
<li data-nodeid="1247">
<p data-nodeid="1248">外部函数的返回值是内部函数本身。</p>
</li>
</ul>
<p data-nodeid="1249">“闭包”这个概念非常重要，除了在 Python 中，闭包在 JavaScript、Go、PHP 等许多语言中都有广泛的应用。</p>
<p data-nodeid="1250">而闭包在 Python 中的经典应用就是装饰器，而装饰器使 Python 代码能够夹带很多“私货”，下面我们就来看下装饰器的应用。</p>
<h3 data-nodeid="1251">装饰器（decorator）</h3>
<p data-nodeid="1252">装饰器是闭包的一个经典应用。装饰器（decorator）在 python 中用来扩展原函数的功能，目的是在不改变原来函数代码的情况下，给函数增加新的功能。</p>
<h4 data-nodeid="1253">1.实现装饰器</h4>
<p data-nodeid="1254">在我们的测试框架开发中，装饰器非常重要，它可以给函数添加 log 且不影响函数本身。</p>
<p data-nodeid="1255">假设我们有一个函数 sum，作用是用来计算 N 个数字的和：</p>
<pre class="lang-python" data-nodeid="1256"><code data-language="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sum</span>(<span class="hljs-params">*kwargs</span>):</span>
    total = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> ele <span class="hljs-keyword">in</span> kwargs:
        total = total + ele
    <span class="hljs-keyword">return</span> total
</code></pre>
<p data-nodeid="1257">现在，我们加了需求，需要记录这个函数开始的时间和结束的时间。<br>
正常情况下，我们的代码是这样的：</p>
<pre class="lang-python" data-nodeid="1258"><code data-language="python"><span class="hljs-keyword">import</span> time
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sum</span>(<span class="hljs-params">*kwargs</span>):</span>
    print(<span class="hljs-string">'function start at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
    total = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> ele <span class="hljs-keyword">in</span> kwargs:
        total = total + ele
    print(<span class="hljs-string">'function end at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
    <span class="hljs-keyword">return</span> total

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    print(sum(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>))
</code></pre>
<p data-nodeid="1259">后来，我们发现这个记录函数开始和结束时间的功能很好用，我们要求把这个功能加到每一个运行函数中去。<br>
那么怎么办呢？难道要每一个函数都去加这样的代码吗？ 这样一点也不符合我们在前几节说的代码规范原则。</p>
<p data-nodeid="1260">所以我们来稍做改变，把计算的函数sum的函数单独抽取出来不变，把时间处理的语句另行定义函数处理。于是上面的函数，就变成了以下的样子：</p>
<pre class="lang-python" data-nodeid="1261"><code data-language="python"><span class="hljs-keyword">import</span> time

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sum</span>(<span class="hljs-params">*kwargs</span>):</span>
    total = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> ele <span class="hljs-keyword">in</span> kwargs:
        total = total + ele
    <span class="hljs-keyword">return</span> total

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">record_time</span>(<span class="hljs-params">*kwargs</span>):</span>
    print(<span class="hljs-string">'function start at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
    total = sum(*kwargs)
    print(<span class="hljs-string">'function end at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
    <span class="hljs-keyword">return</span> total

<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    sum(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>)
</code></pre>
<p data-nodeid="1262">以后我们再给函数加有关时间处理的功能，加到 record_time 里好了，而 sum 函数根本不用变。那这个函数还能更加简化吗？</p>
<p data-nodeid="1263">结合我们刚刚讲过的闭包概念，我们用外函数和内函数来替换下：</p>
<p data-nodeid="1264">record_time就相当于我刚刚讲的outer函数，wrapper函数就是inner函数，只不过我们的inner函数的入参是个函数，这样我们就实现了对函数本身功能的装饰。</p>
<pre class="lang-python" data-nodeid="1265"><code data-language="python"><span class="hljs-keyword">import</span> time

<span class="hljs-comment"># 这个是外函数</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">record_time</span>(<span class="hljs-params">func</span>):</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">wrapper</span>(<span class="hljs-params">*kwargs</span>):</span>
        print(<span class="hljs-string">'function start at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
        total = func(*kwargs)
        print(<span class="hljs-string">'function end at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
        <span class="hljs-keyword">return</span> total
    <span class="hljs-keyword">return</span> wrapper

<span class="hljs-comment"># 这个是我们真正的功能函数</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sum</span>(<span class="hljs-params">*kwargs</span>):</span>
    total = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> ele <span class="hljs-keyword">in</span> kwargs:
        total = total + ele
    time.sleep(<span class="hljs-number">2</span>)
    <span class="hljs-keyword">return</span> total


<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment"># 外函数，内函数，和功能函数一起，实现了不改变功能函数的前提下，给功能函数加功能的操作。</span>
    print(record_time(sum)(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>))
</code></pre>
<p data-nodeid="1266">运行一下，测试结果为：</p>
<pre class="lang-java" data-nodeid="1267"><code data-language="java">function start at <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">14</span> <span class="hljs-number">01</span>:<span class="hljs-number">06</span>:<span class="hljs-number">49</span>
function end at <span class="hljs-number">2020</span>-<span class="hljs-number">08</span>-<span class="hljs-number">14</span> <span class="hljs-number">01</span>:<span class="hljs-number">06</span>:<span class="hljs-number">49</span>
<span class="hljs-number">10</span>
</code></pre>
<p data-nodeid="1426">假设我们的需求又变化啦，我们现在不统计函数的运行开始和结束时间了，改成统计函数的运行时长了，那么我们只需要改 record_time 这个函数就好了，而我们的功能函数 sum 就无须再改了，这样是不是方便了很多？</p>
<p data-nodeid="1427" class="">有了装饰器，我们可以在不改变原有函数代码的前提下，增加、改变原有函数的功能。这种方式也被称作“切面编程”，实际上，装饰器正是切面编程的最佳释例。</p>

<h4 data-nodeid="1269">2.语法糖</h4>
<p data-nodeid="1270">不过你发现没，我们的调用仍然很麻烦，record_time(sum)(1,2,3,4)的调用方式，不容易让使用者理解我们这个函数是在做什么，于是 Python 中为了让大家写起来方便，给了装饰器一个语法糖，其用法如下：</p>
<pre class="lang-java" data-nodeid="1271"><code data-language="java">@decorator
#对应我们的例子就是
@record_time
</code></pre>
<p data-nodeid="11080" class="te-preview-highlight">使用语法糖后，在调用函数时，我们就无须再写这个装饰器函数了，<strong data-nodeid="11087">转而直接写我们的功能函数就可以了</strong>，于是我们的例子就变成了：</p>
<pre class="lang-python" data-nodeid="11081"><code data-language="python"><span class="hljs-keyword">import</span> time


<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">record_time</span>(<span class="hljs-params">func</span>):</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">wrapper</span>(<span class="hljs-params">*kwargs</span>):</span>
        print(<span class="hljs-string">'function start at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
        total = func(*kwargs)
        print(<span class="hljs-string">'function end at {}'</span>.format(time.strftime(<span class="hljs-string">"%Y-%m-%d %H:%M:%S"</span>, time.localtime()) ))
        <span class="hljs-keyword">return</span> total
    <span class="hljs-keyword">return</span> wrapper

<span class="hljs-comment">#注意这一行，我们把record_time这个函数装饰到sum函数上。</span>
<span class="hljs-meta">@record_time</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sum</span>(<span class="hljs-params">*kwargs</span>):</span>
    total = <span class="hljs-number">0</span>
    <span class="hljs-keyword">for</span> ele <span class="hljs-keyword">in</span> kwargs:
        total = total + ele
    time.sleep(<span class="hljs-number">2</span>)
    <span class="hljs-keyword">return</span> total


<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    <span class="hljs-comment">#注意此次无须再写record_time了，这样有利于大家把关注点放在功能函数本身。</span>
    print(sum(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">3</span>,<span class="hljs-number">4</span>))`
</code></pre>


<p data-nodeid="10793" class="">有了装饰器，我们就可以做很多额外的工作，例如插入日志、做事务处理等，在后续的章节中我也会介绍如何利用装饰器给测试用例打标签。</p>


































<h3 data-nodeid="1275">总结</h3>
<p data-nodeid="1276">本小节我向你介绍了 Python 的一些常用高阶技巧，这些技巧有的是单纯地帮助你减少代码量，有的可以使你动态地获取某些对象的属性并进行判断，有些则可以帮助你扩展你原本函数所有的功能。</p>
<p data-nodeid="1277">掌握这些技巧会使你的 Python 编码水平更进一步，更有助于你后续的学习。希望你仔细研读这些技巧并做到熟练应用。</p>
<hr data-nodeid="1278">
<p data-nodeid="1279" class=""><a href="https://shenceyun.lagou.com/t/eka" data-nodeid="1425">“测试开发工程师名企直推营” 入口，免费领取 50G 资料包</a></p>

---

### 精选评论

##### **芝：
> 老师你好，请教个问题。在最后一种写法中，等价于调用record_time(sum)(1,2,3,4)，按照执行顺序，应该是先打印出function start at {}，然后执行 def wrapper(*kwargs)，再执行打印'function end at {}'，最后执行func(*kwargs)。这种真的可以记录sum函数的结束时间么？感觉并没有执行到sum函数就打印了'function end at {}'还请老师答疑解惑，非常感谢~

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢指出，已经重新编辑，并且加了sleep语句，更容易理解。

##### William：
> 语法糖这个例子过渡的好，一下子就明白语法糖@decorator的意思了

##### **0321：
> `print(record_time(sum)(1,2,3,4))`为什么是这样调用的，按照直观理解 我觉得应该是`print(record_time(sum(1,2,3,4)))`😂

##### *珊：
> 老师，您好，这篇文章中没有看到反射的例子貌似，自省的例子是那四个吗？没太理解反射的原理和实现方式。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 其实就是4个方法，hasattr， getattr，setattr，delattr。 其中用的最多的就是getattr这个函数。 你可以搜索了解下。

##### **娟：
> 语法糖中的执行顺序是 打印开始时间，然后调用def wrapper(*kwargs):，然后打印结束时间；但是怎么跑出来的开始时间和结束时间是一样的呢？跟装饰器中代码中跑出的时间不一样呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为执行时间太快了吧。 你理解这段代码的逻辑就可以。在实际工作中，代码逻辑越多，开始和结束时间的差异就越大。你就不会遇见这个问题的。

##### **智：
> return [l for l in letter]   这行代码换成   return [l for item in letter ] 是不是更好

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 要换l就要都换成item，否则会报错。

##### **兴：
> [expression for item in list if xxx else yyy] 是错误的写法，应该为[expression">if xxx else yyyfor item in list]

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 很高兴有人就如何写就更pythonic的代码进行讨论。不过列表表达式的语法python中有公论，是下面的定义：[expression(variable) for variable in input_set [predicate][, …]]。如果还有疑虑，欢迎进一步沟通。也可以直接关注我的公众号iTesting查看更多关于python的基础知识。

##### **阳：
> 老师你好，我们现在做一个把后台log同步到前台的功能，是要改动开源项目的源代码，但是用了websocket之后需要给所有call过的方法加个参数，python有没有什么方便的方法去解决这个问题呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 听起来像是在不改变原来函数的基础上，给函数增加新的功能，这个是每一个编程语言都有的。切面编程。在python里可以直接使用装饰器达到你的目的。其它语言可自行搜索。

##### **婷：
> 感谢啊，python高级这篇看了很久，想请问下，闭包是内部函数还可以调用外部函数的变量，请问下原理是什么，为什么可以在返回内部函数时，可以获得外部函数的变量？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 闭包是个很复杂的概念，如果看完本文还是不容易理解，则可以暂时记住闭包的特点，后面慢慢理解应用。

##### **盼：
> 在最后添加语法糖那段，最后一个打印时间，按照执行顺序，证明不了函数的执行时间吧？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你想获取函数的执行时间， 需要
1. 在在函数运行开始前print语句后加start = time.time()， 运行结束后的print语句前加end = time.time()。
2. duration = end - start的方式获取。
在例子中，为了让大家在把注意力放在装饰器本身上，我没有写duration的计算语句。

##### **潜：
> 装饰器例子里面的打印运行结束时间 应该放在内部函数里面吧，不然 打印开始时间-定义内部函数-打印结束时间，要计算的函数还没执行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯嗯。感谢指出，已经重新编辑，并且加了sleep语句，更容易理解。

##### *林：
> 到位，到位。蔡老大写的教程，瞬间懂了

