<p data-nodeid="46020">在上节课中我留了一个思考题，在一个字符串中查找另外一个字符串是否存在，这个其实是字符串查找的功能，假如我需要在“飞雪无情”这个字符串中查找“飞雪”，可以这么做：</p>
<pre class="lang-go" data-nodeid="46021"><code data-language="go">i:=strings.Index(<span class="hljs-string">"飞雪无情"</span>,<span class="hljs-string">"飞雪"</span>)
</code></pre>
<p data-nodeid="46022">这就是 Go 语言标准库为我们提供的常用函数，以供我们使用，减少开发。</p>
<p data-nodeid="46023">这节课我们继续讲解 Go 语言，今天的内容是：Go 语言代码逻辑的控制。</p>
<p data-nodeid="46024">流程控制语句用于控制程序的执行顺序，这样你的程序就具备了逻辑结构。一般流程控制语句需要和各种条件结合使用，比如用于条件判断的 if，用于选择的 switch，用于循环的 for 等。这一节课，我会为你详细介绍，通过示例演示它们的使用方式。</p>
<h3 data-nodeid="46025">if 条件语句</h3>
<p data-nodeid="46026">if 语句是条件语句，它根据布尔值的表达式来决定选择哪个分支执行：如果表达式的值为 true，则 if 分支被执行；如果表达式的值为 false，则 else 分支被执行。下面，我们来看一个 if 条件语句示例：</p>
<p data-nodeid="46027"><em data-nodeid="46114"><strong data-nodeid="46113">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46028"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
    i:=<span class="hljs-number">10</span>
    <span class="hljs-keyword">if</span> i &gt;<span class="hljs-number">10</span> {
        fmt.Println(<span class="hljs-string">"i&gt;10"</span>)
    } <span class="hljs-keyword">else</span> {
        fmt.Println(<span class="hljs-string">"i&lt;=10"</span>)
    }
}
</code></pre>
<p data-nodeid="46029">这是一个非常简单的 if……else 条件语句，当 i&gt;10 为 true 的时候，if 分支被执行，否则就执行 else 分支，你自己可以运行这段代码，验证打印结果。</p>
<p data-nodeid="46030">关于 if 条件语句的使用有一些规则：</p>
<ol data-nodeid="46031">
<li data-nodeid="46032">
<p data-nodeid="46033">if 后面的条件表达式不需要使用 ()，这和有些编程语言不一样，也更体现 Go 语言的简洁；</p>
</li>
<li data-nodeid="46034">
<p data-nodeid="46035">每个条件分支（if 或者 else）中的大括号是必须的，哪怕大括号里只有一行代码（如示例）；</p>
</li>
<li data-nodeid="46036">
<p data-nodeid="46037">if 紧跟的大括号 { 不能独占一行，else 前的大括号 } 也不能独占一行，否则会编译不通过；</p>
</li>
<li data-nodeid="46038">
<p data-nodeid="46039">在 if……else 条件语句中还可以增加多个 else if，增加更多的条件分支。</p>
</li>
</ol>
<p data-nodeid="46040">通过 go run ch03/main.go 运行下面的这段代码，会看到输出了 5&lt;i&lt;=10 ，这说明代码中的 else if i&gt;5 &amp;&amp; i&lt;=10 成立，该分支被执行。</p>
<p data-nodeid="46041"><em data-nodeid="46135"><strong data-nodeid="46134">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46042"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
    i:=<span class="hljs-number">6</span>
    <span class="hljs-keyword">if</span> i &gt;<span class="hljs-number">10</span> {
        fmt.Println(<span class="hljs-string">"i&gt;10"</span>)
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>  i&gt;<span class="hljs-number">5</span> &amp;&amp; i&lt;=<span class="hljs-number">10</span> {
        fmt.Println(<span class="hljs-string">"5&lt;i&lt;=10"</span>)
    } <span class="hljs-keyword">else</span> {
        fmt.Println(<span class="hljs-string">"i&lt;=5"</span>)
    }
}
</code></pre>
<p data-nodeid="46043">你可以通过修改 i 的初始值，来验证其他分支的执行情况。</p>
<p data-nodeid="46044">你还可以增加更多的 else if，以增加更多的条件分支，不过这种方式不被推荐，因为代码可读性差，多个条件分支可以使用我后面讲到的 switch 代替，使代码更简洁。</p>
<p data-nodeid="46045">和其他编程语言不同，在 Go 语言的 if 语句中，可以有一个简单的表达式语句，并将该语句和条件语句使用分号 ; 分开。同样是以上的示例，我使用这种方式对其改造，如下面代码所示：</p>
<p data-nodeid="46046"><em data-nodeid="46143"><strong data-nodeid="46142">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46047"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
    <span class="hljs-keyword">if</span> i:=<span class="hljs-number">6</span>; i &gt;<span class="hljs-number">10</span> {
        fmt.Println(<span class="hljs-string">"i&gt;10"</span>)
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>  i&gt;<span class="hljs-number">5</span> &amp;&amp; i&lt;=<span class="hljs-number">10</span> {
        fmt.Println(<span class="hljs-string">"5&lt;i&lt;=10"</span>)
    } <span class="hljs-keyword">else</span> {
        fmt.Println(<span class="hljs-string">"i&lt;=5"</span>)
    }
}
</code></pre>
<p data-nodeid="46048">在 if 关键字之后，i&gt;10 条件语句之前，通过分号 ; 分隔被初始化的 i:=6。这个简单语句主要用来在 if 条件判断之前做一些初始化工作，可以发现输出结果是一样的。</p>
<p data-nodeid="46049">通过 if 简单语句声明的变量，只能在整个 if……else if……else 条件语句中使用，比如以上示例中的变量 i。</p>
<h3 data-nodeid="46050">switch 选择语句</h3>
<p data-nodeid="46051">if 条件语句比较适合分支较少的情况，如果有很多分支的话，选择 switch 会更方便，比如以上示例，使用 switch 改造后的代码如下：</p>
<p data-nodeid="46052"><em data-nodeid="46152"><strong data-nodeid="46151">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46053"><code data-language="go"><span class="hljs-keyword">switch</span> i:=<span class="hljs-number">6</span>;{
<span class="hljs-keyword">case</span> i&gt;<span class="hljs-number">10</span>:
    fmt.Println(<span class="hljs-string">"i&gt;10"</span>)
<span class="hljs-keyword">case</span> i&gt;<span class="hljs-number">5</span> &amp;&amp; i&lt;=<span class="hljs-number">10</span>:
    fmt.Println(<span class="hljs-string">"5&lt;i&lt;=10"</span>)
<span class="hljs-keyword">default</span>:
    fmt.Println(<span class="hljs-string">"i&lt;=5"</span>)
}
</code></pre>
<p data-nodeid="46054">switch 语句同样也可以用一个简单的语句来做初始化，同样也是用分号 ; 分隔。每一个 case 就是一个分支，分支条件为 true 该分支才会执行，而且 case 分支后的条件表达式也不用小括号 () 包裹。</p>
<p data-nodeid="46055">在 Go 语言中，switch 的 case 从上到下逐一进行判断，一旦满足条件，立即执行对应的分支并返回，其余分支不再做判断。也就是说 Go 语言的 switch 在默认情况下，case 最后自带 break。这和其他编程语言不一样，比如 C 语言在 case 分支里必须要有明确的 break 才能退出一个 case。Go 语言的这种设计就是为了防止忘记写 break 时，下一个 case 被执行。</p>
<p data-nodeid="46056">那么如果你真的有需要，的确需要执行下一个紧跟的 case 怎么办呢？Go 语言也考虑到了，提供了 fallthrough 关键字。现在看个例子，如下面的代码所示：</p>
<p data-nodeid="46057"><em data-nodeid="46160"><strong data-nodeid="46159">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46058"><code data-language="go"><span class="hljs-keyword">switch</span> j:=<span class="hljs-number">1</span>;j {
<span class="hljs-keyword">case</span> <span class="hljs-number">1</span>:
    <span class="hljs-keyword">fallthrough</span>
<span class="hljs-keyword">case</span> <span class="hljs-number">2</span>:
    fmt.Println(<span class="hljs-string">"1"</span>)
<span class="hljs-keyword">default</span>:
    fmt.Println(<span class="hljs-string">"没有匹配"</span>)
}
</code></pre>
<p data-nodeid="46059">以上示例运行会输出 1，如果省略 case 1: 后面的 fallthrough，则不会有任何输出。</p>
<p data-nodeid="46060">不知道你是否可以发现，和上一个例子对比，这个例子的 switch 后面是有表达式的，也就是输入了 ;j，而上一个例子的 switch 后只有一个用于初始化的简单语句。</p>
<p data-nodeid="46061">当 switch 之后有表达式时，case 后的值就要和这个表达式的结果类型相同，比如这里的 j 是 int 类型，那么 case 后就只能使用 int 类型，如示例中的 case 1、case 2。如果是其他类型，比如使用 case "a" ，会提示类型不匹配，无法编译通过。</p>
<p data-nodeid="46062">而对于 switch 后省略表达式的情况，整个 switch 结构就和 if……else 条件语句等同了。</p>
<p data-nodeid="46063">switch 后的表达式也没有太多限制，是一个合法的表达式即可，也不用一定要求是常量或者整数。你甚至可以像如下代码一样，直接把比较表达式放在 switch 之后：</p>
<p data-nodeid="46064"><em data-nodeid="46174"><strong data-nodeid="46173">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46065"><code data-language="go"><span class="hljs-keyword">switch</span> <span class="hljs-number">2</span>&gt;<span class="hljs-number">1</span> {
<span class="hljs-keyword">case</span> <span class="hljs-literal">true</span>:
    fmt.Println(<span class="hljs-string">"2&gt;1"</span>)
<span class="hljs-keyword">case</span> <span class="hljs-literal">false</span>:
    fmt.Println(<span class="hljs-string">"2&lt;=1"</span>)
}
</code></pre>
<p class="te-preview-highlight" data-nodeid="58469">可见 Go 语言的 switch 语句非常强大且灵活。</p>





























<h3 data-nodeid="46067">for 循环语句</h3>
<p data-nodeid="46068">当需要计算 1 到 100 的数字之和时，如果用代码将一个个数字加起来，会非常复杂，可读性也不好，这就体现出循环语句的存在价值了。</p>
<p data-nodeid="46069">下面是一个经典的 for 循环示例，从这个示例中，我们可以分析出 for 循环由三部分组成，其中，需要使用两个 ; 分隔，如下所示：</p>
<p data-nodeid="46070"><em data-nodeid="46183"><strong data-nodeid="46182">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46071"><code data-language="go">sum:=<span class="hljs-number">0</span>
<span class="hljs-keyword">for</span> i:=<span class="hljs-number">1</span>;i&lt;=<span class="hljs-number">100</span>;i++ {
    sum+=i
}
fmt.Println(<span class="hljs-string">"the sum is"</span>,sum)
</code></pre>
<p data-nodeid="46072">其中：</p>
<ol data-nodeid="46073">
<li data-nodeid="46074">
<p data-nodeid="46075">第一部分是一个简单语句，一般用于 for 循环的初始化，比如这里声明了一个变量，并对 i:=1 初始化；</p>
</li>
<li data-nodeid="46076">
<p data-nodeid="46077">第二部分是 for 循环的条件，也就是说，它表示 for 循环什么时候结束。这里的条件是 i&lt;=100；</p>
</li>
<li data-nodeid="46078">
<p data-nodeid="46079">第三部分是更新语句，一般用于更新循环的变量，比如这里 i++，这样才能达到递增循环的目的。</p>
</li>
</ol>
<p data-nodeid="46080">需要特别留意的是，Go 语言里的 for 循环非常强大，以上介绍的三部分组成都不是必须的，可以被省略，下面我就来为你演示，省略以上三部分后的效果。</p>
<p data-nodeid="46081">如果你以前学过其他编程语言，可能会见到 while 这样的循环语句，在 Go 语言中没有 while 循环，但是可以通过 for 达到 while 的效果，如以下代码所示：</p>
<p data-nodeid="46082"><em data-nodeid="46196"><strong data-nodeid="46195">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46083"><code data-language="go">sum:=<span class="hljs-number">0</span>
i:=<span class="hljs-number">1</span>
<span class="hljs-keyword">for</span> i&lt;=<span class="hljs-number">100</span> {
    sum+=i
    i++
}
fmt.Println(<span class="hljs-string">"the sum is"</span>,sum)
</code></pre>
<p data-nodeid="46084">这个示例和上面的 for 示例的效果是一样的，但是这里的 for 后只有 i&lt;=100 这一个条件语句，也就是说，它达到了 while 的效果。</p>
<p data-nodeid="46085">在 Go 语言中，同样支持使用 continue、break 控制 for 循环：</p>
<ol data-nodeid="46086">
<li data-nodeid="46087">
<p data-nodeid="46088">continue 可以跳出本次循环，继续执行下一个循环。</p>
</li>
<li data-nodeid="46089">
<p data-nodeid="46090">break 可以跳出整个 for 循环，哪怕 for 循环没有执行完，也会强制终止。</p>
</li>
</ol>
<p data-nodeid="46091">现在我对上面计算 100 以内整数和的示例再进行修改，演示 break 的用法，如以下代码：</p>
<p data-nodeid="46092"><em data-nodeid="46208"><strong data-nodeid="46207">ch03/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="46093"><code data-language="go">sum:=<span class="hljs-number">0</span>
i:=<span class="hljs-number">1</span>
<span class="hljs-keyword">for</span> {
    sum+=i
    i++
    <span class="hljs-keyword">if</span> i&gt;<span class="hljs-number">100</span> {
        <span class="hljs-keyword">break</span>
    }
}
fmt.Println(<span class="hljs-string">"the sum is"</span>,sum)
</code></pre>
<p data-nodeid="46094">这个示例使用的是没有任何条件的 for 循环，也称为 for 无限循环。此外，使用 break 退出无限循环，条件是 i&gt;100。</p>
<h3 data-nodeid="46095">总结</h3>
<p data-nodeid="46096">这节课主要讲解 if、for 和 switch 这样的控制语句的基本用法，使用它们，你可以更好地控制程序的逻辑结构，达到业务需求的目的。</p>
<p data-nodeid="46097">这节课的思考题是：任意举个例子，练习 for 循环 continue 的使用。</p>
<p data-nodeid="46098">Go 语言提供的控制语句非常强大，本节课我并没有全部介绍，比如 switch 选择语句中的类型选择，for 循环语句中的 for range 等高级能力。这些高级能力我会在后面的课程中逐一介绍，接下来要讲的集合类型，就会详细地为你演示如何使用 for range 遍历集合，记得来听课！</p>
<hr data-nodeid="46099">
<p data-nodeid="46100"><strong data-nodeid="46224">《Java <b><strong data-nodeid="46223">工程师高薪训练营</strong></b>》</strong></p>
<p data-nodeid="46101">拉勾背书内推+硬核实战技术干货，帮助每位 Java 工程师达到阿里 P7 技术能力。<a href="https://kaiwu.lagou.com/java_architect.html?utm_source=lagouedu&amp;utm_medium=zhuanlan&amp;utm_campaign=Java%E5%B7%A5%E7%A8%8B%E5%B8%88%E9%AB%98%E8%96%AA%E8%AE%AD%E7%BB%83%E8%90%A5" data-nodeid="46228">点击链接，快来领取！</a></p>
<p data-nodeid="46102"><strong data-nodeid="46232">《Java 就业集训营》</strong></p>
<p data-nodeid="46103">零基础 180 天高薪就业，<a href="https://kaiwu.lagou.com/java_basic.html?utm_source=zhuanlan%20article&amp;utm_medium=bottom&amp;utm_campaign=Go%E8%AF%AD%E8%A8%80%E4%B8%93%E6%A0%8F#/index" data-nodeid="46236">点击链接，快来领取！</a></p>

---

### 精选评论

##### **杰：
> 写个冒泡package mainimport "fmt"func main() {   ar := [15]int{6, 8, 9, 4, 3, 1, 1, 5, 2, 6, 11, 15, 22, 33, 55}   fmt.Print("未排序前: ")   fmt.Println(ar)   num := len(ar)   for i := 0; i ">; i++ {      for j := i + 1; j ">; j++ {         if ar[i]  ar[j] {            ar[i], ar[j] = ar[j], ar[i]         }      }   }   fmt.Print("排序后: ")   fmt.Println(ar)}

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 哇！都看完第三讲啦！点赞点赞

##### **涛：
> 写的不错，但对我来说，比较简单！支持！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢支持！一起加油

##### **恩：
> 原来switch可以这样子写，学到了~

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油！

##### *星：
> 能解释下这个【switch j:=1;j】这个么， 为啥 case 2也会输出？ j 的值应该赋值为 1吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为case1 中有fallthrough关键字。

##### **7935：
> 继续学习

##### **程：
> 浅显易懂，不错不错

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢！加油学习！

##### **钜：
> 思考题：练习 for 循环 continue 的使用package mainimport (    "fmt")func main() {    sum := 0    for i := 1; i = 3; i++ {        if i == 2 {            continue        } else {            sum += i        }    }    fmt.Println("the sum is:", sum)}

##### *琦：
> 接触了很多go的学习资料和课程，这个算是不错的😀

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢支持

##### **辉：
> 打卡

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; yeah

##### **生：
> 第一次注意到这个关键字fallthrough

##### *宇：
> 老师帮忙砍一下代码。这个问题怎么解决dandanSanwei := map[string]int{"head":20}	dandan := person{20,"dandan",dandanSanwei}	fmt.Println(dandan)	structModify(dandan)	fmt.Println(dandan)	pointStuctModify(dandan)	fmt.Println(dandan)func structModify(p person)  {p.age = 100p.name = "father"p.sanWei["head"] = 30}func pointStuctModify(p *person)  {p.age = 100p.name = "father"p.sanWei["head"] = 100}type person struct {age intname stringsanWei map[string]int}当将person作为指针传过去的时候（pointStuctModify），程序报错。cannot use dandan (type person) as type *person in argument to pointStuctModify。但是作为值传递structModify方法。就可以

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为pointStuctModify函数需要一个指针，而你给了一个值。传递&dandan就可以了。

##### yhb：
> index使用需要注意一下，它是根据字节建立的索引飞雪无情，飞雪 返回6

##### **升：
> 打卡

##### *文：
> 基础部分提到的 switch ；j 这小部分还真是以前没有留意过，谢谢

##### *宇：
> 期待老师讲并发那里。

##### **龙：
> 语法的确灵活简单

##### **2365：
> 很快就学完了

##### **峰：
> 期待老师的更新

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 期待你的继续学习

##### **军：
> 打卡第三课。😀😀

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油

##### **安：
> 前面的是基础，期待后面的

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 后面更精彩

