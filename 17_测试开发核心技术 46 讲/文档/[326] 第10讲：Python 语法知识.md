<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时我们就主要学习 Python 的基础语法知识，因为本课时的内容非常简单，这里就不再带你演示具体的代码了。</span><br></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">if 条件判断</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先，我们学习 if 条件判断语句，以具体的场景为例，如代码所示：</span></p>
<pre>x&nbsp;=&nbsp;input("你好，请输入你编写自动化测试用例常用的测试框架：")
if&nbsp;x&nbsp;==&nbsp;"pytest":
print("高级测试工程师")
elif&nbsp;x&nbsp;==&nbsp;"unittest":
print("中级测试工程师")
else:
print("不符合要求")</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们提出了一个问题“请输入你编写自动化测试用例的测试框架：”并将问题的答案赋值给变量 x，接下来通过 if 判断变量 x 是否与指定的条件匹配，比如回答是否为 pytest、unittest 等，如果回答的是 pytest，就打印高级测试工程师，以此类推。通过 if 条件判断语句可以根据变量值来匹配不同的操作。</span></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">for 遍历循环</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二个是 for 遍历循环，for遍历循环会根据数据本身的特征遍历其中的每一项数据，上一课时我们学习的集合、元祖、列表等内容都是序列型的数据，针对这些序列型的数据我们就可以对其进行相应的遍历，如代码所示：</span><br></p>
<pre>interviewss&nbsp;=&nbsp;[26,28,30,32,26,35,40]
for&nbsp;age&nbsp;in&nbsp;interviewss
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;30&nbsp;&lt;&nbsp;age&nbsp;&lt;&nbsp;35:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(f"{age}岁可能是一个高级工程师")
&nbsp;&nbsp;&nbsp;&nbsp;elif&nbsp;age&nbsp;&gt;=&nbsp;35:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;(f"{age}岁可能是一个测试专家或者测试管理")
&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;(f"{age}岁可能是一个初中级工程师")</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们首先定义了一个列表并往里面进行赋值，然后可以使用 for in 结构的语句来进行循环遍历，在 interviewee 中判断 age 的范围，然后根据 age 的范围进行一个初始化的判断并输出不同的结果，for 遍历循环的特点是 for in 结构使用非常简单，可遍历的数据结构都可以使用 for in 结构来遍历数据。</span></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">for 计数循环</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二个 for 循环叫作 for 计数循环，for 计数循环严格按照计数计算并进行循环遍历，举个例子，如代码所示：</span></p>
<pre>interviewss&nbsp;=&nbsp;[26,28,30,32,26,35,40]
for&nbsp;i&nbsp;in&nbsp;range(5):
if&nbsp;30&nbsp;&lt;&nbsp;interviewss[i]&nbsp;&nbsp;&lt;&nbsp;35:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print(f"第{i}名面试者：{interviewss[i]}岁可能是一个高级工程师")
&nbsp;&nbsp;&nbsp;&nbsp;elif&nbsp;age&nbsp;&gt;=&nbsp;35:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;(f"第{i}名面试者：{interviewss[i]}岁可能是一个测试专家或者测试管理")
&nbsp;&nbsp;&nbsp;&nbsp;else:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;print&nbsp;(f"第{i}名面试者：{interviewss[i]}岁可能是一个初中级工程师")</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">仍使用上面定义的列表，这次我们通过 for i in rang(5) 获取前 5 个数据进行判断，range(5)会生成一个标记位从 0~4 的列表，然后在这个子列表中进行循环遍历，这便是一个典型的计数循环，可以根据下标获取相应的数据再通过 if 语句判断面试者对应的职称。</span></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">while 循环</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来是 while 循环，while 循环是条件轮询的循环结构，它会不断地轮询一个条件，如代码所示：</span></p>
<pre>i&nbsp;=&nbsp;1
while&nbsp;i&nbsp;&lt;&nbsp;6：
&nbsp;&nbsp;&nbsp;&nbsp;print(i)
&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;+=1</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 while 后设定了一个判断条件，只有当判断条件为 true 时，才可以继续往下执行，如果不为 true 就跳出循环，可以看到 while 循环也非常简单。</span></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">函数与参数</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">前面我们学习了条件判断语句和循环语句，接下来我们学习函数和类是如何定义的。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">首先是函数，如代码所示：</span></p>
<pre>def&nbsp;x(a,b=1,*c,**d):
&nbsp;&nbsp;&nbsp;&nbsp;print(f"a={a}")
&nbsp;&nbsp;&nbsp;&nbsp;print(f"b={b}")
&nbsp;&nbsp;&nbsp;&nbsp;print(f"c={c}")
&nbsp;&nbsp;&nbsp;&nbsp;print(f"d={d}")

x(2)
x(2,3)
x(2,3,4,5)
x(2,3,4,5,x=1,y=2)</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Python 中可以通过两种方式定义一个函数。</span></p>
<ul style=" white-space: normal;">
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">第一种方式通过 def 语句定义函数，def 后定义一个函数名，函数名后括号里的是传入函数的参数，函数体内只简单打印相应的参数，通过这种方式创建的函数通常叫作命名函数。</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">第二种方式是通过 lambda 表达式定义函数，lambda 表达式允许我们创建一个特殊的方法来执行相应的操作，与第一种方式相对应这种方式是匿名函数。</span></p></li>
</ul>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">函数的另一个重要知识点是参数，Python 中函数的参数主要分为：</span></p>
<ul style=" white-space: normal;">
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">默认参数；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">命名参数；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">变长参数；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">词典参数。</span></p></li>
</ul>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">比如参数 a 和 b=1 都是命名参数，又因为参数 b 设置了默认的初始值，同时它也是默认参数，参数 *c 则表示变长参数，也就是参数个数是可变的；**d 表示词典参数，里面传入的是类似于上一课时我们学习过的词典数据结构里面的 k/v 结构数据。接下来我们将代码中的赋值带入参数，来对比它们有何异同。</span></p>
<ul style=" white-space: normal;">
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">当执行 x(2) 时，a=2，b=1；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">当执行 x(2,3) 时，a=2，b=3；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">当执行 x(2,3,4,5) 时，a=2，b=3，c=4,5；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">当执行 x(2,3,4,5,x=1,y=2)，a=2，b=3，c=4,5，d= x=1,y=2。</span></p></li>
</ul>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">这就是函数和参数的相关知识点，希望你在课后多加练习。</span></p>
<h2 style="white-space: normal;"><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">类与方法</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">最后，我们看下类和方法，在 Python 中定义一个类也非常简单，只需要在 class 后面加上对应的类名即可。我们举个简单的例子，如代码所示：</span></p>
<pre>class&nbsp;MyClass
&nbsp;&nbsp;&nbsp;&nbsp;i&nbsp;=&nbsp;12345

&nbsp;&nbsp;&nbsp;&nbsp;def&nbsp;f(self)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;"hello&nbsp;world"

x&nbsp;=&nbsp;MyClass()
x.f()</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">&nbsp;</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在类里面涉及几个关键的知识点：</span></p>
<ul style=" white-space: normal;">
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">第一个是类的定义，通过 class + 类名 的方式即可定义一个类；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><span style="font-size: 12pt;">然后是初始化方法，初始化方法使用</span><span style="background-color: rgb(255, 255, 255); color: rgb(77, 77, 77); font-size: 12pt; font-family: &quot;Microsoft YaHei&quot;, sans-serif;">&nbsp;__init__ 即可；</span></span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="background-color: rgb(255, 255, 255); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">第三个是类变量，定义在类里的变量即为类变量，比如 i = 12345；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">k可以使用 @ class 的方式将方法升级为类方法；</span></p></li>
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="background-color: rgb(255, 255, 255); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">当调用方法时传入的参数是 self 就表示该方法为实例方法。</span></p></li>
</ul>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">到这里本课时的内容就结束了，通过本课时的学习你可以掌握 Python 常见的控制语句，以及函数与类的相关知识，希望你在课后针对这方面的内容勤加练习，如果你想要更深入地研究还可以查看 Python 的使用文档。</span></p>

---

### 精选评论

##### **怕升职记：
> <div>学习了，第一次知道，可以这样输出字符串（f在前面）</div><div>print(f"你好，a的值是：{a}")</div>

