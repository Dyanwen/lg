<p data-nodeid="965" class="">在本节课开始之前，我们先一起回忆上节课的思考题：是否可以有多个 defer，如果可以的话，它们的执行顺序是怎么样的？</p>
<p data-nodeid="966">对于这道题，可以直接采用写代码测试的方式，如下所示：</p>
<pre class="lang-go" data-nodeid="967"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">moreDefer</span><span class="hljs-params">()</span></span>{
   <span class="hljs-keyword">defer</span>  fmt.Println(<span class="hljs-string">"First defer"</span>)
   <span class="hljs-keyword">defer</span>  fmt.Println(<span class="hljs-string">"Second defer"</span>)
   <span class="hljs-keyword">defer</span>  fmt.Println(<span class="hljs-string">"Three defer"</span>)
   fmt.Println(<span class="hljs-string">"函数自身代码"</span>)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span>{
  moreDefer()
}
</code></pre>
<p data-nodeid="968">我定义了 moreDefer 函数，函数里有三个 defer 语句，然后在 main 函数里调用它。运行这段程序可以看到如下内容输出：</p>
<pre class="lang-java" data-nodeid="969"><code data-language="java">函数自身代码
Three defer
Second defer
First defer
</code></pre>
<p data-nodeid="970">通过以上示例可以证明：</p>
<ol data-nodeid="971">
<li data-nodeid="972">
<p data-nodeid="973">在一个方法或者函数中，可以有多个 defer 语句；</p>
</li>
<li data-nodeid="974">
<p data-nodeid="975">多个 defer 语句的执行顺序依照后进先出的原则。</p>
</li>
</ol>
<p data-nodeid="976">defer 有一个调用栈，越早定义越靠近栈的底部，越晚定义越靠近栈的顶部，在执行这些 defer 语句的时候，会先从栈顶弹出一个 defer 然后执行它，也就是我们示例中的结果。</p>
<p data-nodeid="977">下面我们开始本节课的学习。本节课是 Go 语言的重点——协程和通道，它们是 Go 语言并发的基础，我会从这两个基础概念开始，带你逐步深入 Go 语言的并发。</p>
<h3 data-nodeid="978">什么是并发</h3>
<p data-nodeid="979">前面的课程中，我所写的代码都按照顺序执行，也就是上一句代码执行完，才会执行下一句，这样的代码逻辑简单，也符合我们的阅读习惯。</p>
<p data-nodeid="980">但这样是不够的，因为计算机很强大，如果只让它干完一件事情再干另外一件事情就太浪费了。比如一款音乐软件，使用它听音乐的时候还想让它下载歌曲，同一时刻做了两件事，在编程中，这就是并发，并发可以让你编写的程序在同一时刻做多几件事情。</p>
<h3 data-nodeid="981">进程和线程</h3>
<p data-nodeid="982">讲并发就绕不开线程，不过在介绍线程之前，我先为你介绍什么是进程。</p>
<h4 data-nodeid="983">进程</h4>
<p data-nodeid="984">在操作系统中，进程是一个非常重要的概念。当你启动一个软件（比如浏览器）的时候，操作系统会为这个软件创建一个进程，这个进程是该软件的工作空间，它包含了软件运行所需的所有资源，比如内存空间、文件句柄，还有下面要讲的线程等。下面的图片就是我的电脑上运行的进程：</p>
<p data-nodeid="985"><img src="https://s0.lgstatic.com/i/image/M00/70/AE/CgqCHl-7fwyAdSu_AADl16erQwg589.png" alt="Drawing 0.png" data-nodeid="1095"></p>
<div data-nodeid="986"><p style="text-align:center">（电脑运行的进程）</p></div>
<p data-nodeid="987">那么线程是什么呢？</p>
<h4 data-nodeid="988">线程</h4>
<p data-nodeid="989">线程是进程的执行空间，一个进程可以有多个线程，线程被操作系统调度执行，比如下载一个文件，发送一个消息等。这种多个线程被操作系统同时调度执行的情况，就是多线程的并发。</p>
<p data-nodeid="990">一个程序启动，就会有对应的进程被创建，同时进程也会启动一个线程，这个线程叫作主线程。如果主线程结束，那么整个程序就退出了。有了主线程，就可以从主线里启动很多其他线程，也就有了多线程的并发。</p>
<h3 data-nodeid="991">协程（Goroutine）</h3>
<p data-nodeid="992">Go 语言中没有线程的概念，只有协程，也称为 goroutine。相比线程来说，协程更加轻量，一个程序可以随意启动成千上万个 goroutine。</p>
<p data-nodeid="993">goroutine 被 Go runtime 所调度，这一点和线程不一样。也就是说，Go 语言的并发是由 Go 自己所调度的，自己决定同时执行多少个 goroutine，什么时候执行哪几个。这些对于我们开发者来说完全透明，只需要在编码的时候告诉 Go 语言要启动几个 goroutine，至于如何调度执行，我们不用关心。</p>
<p data-nodeid="994">要启动一个 goroutine 非常简单，Go 语言为我们提供了 go 关键字，相比其他编程语言简化了很多，如下面的代码所示：</p>
<p data-nodeid="995"><em data-nodeid="1108"><strong data-nodeid="1107">ch08/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="996"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">go</span> fmt.Println(<span class="hljs-string">"飞雪无情"</span>)
   fmt.Println(<span class="hljs-string">"我是 main goroutine"</span>)
   time.Sleep(time.Second)
}
</code></pre>
<p data-nodeid="997">这样就启动了一个 goroutine，用来调用 fmt.Println 函数，打印“飞雪无情”。所以这段代码里有两个 goroutine，一个是 main 函数启动的 main goroutine，一个是我自己通过 go 关键字启动的 goroutine。</p>
<p data-nodeid="998">从示例中可以总结出 go 关键字的语法，如下所示：</p>
<pre class="lang-go" data-nodeid="999"><code data-language="go"><span class="hljs-keyword">go</span> function()
</code></pre>
<p data-nodeid="1000">go 关键字后跟一个方法或者函数的调用，就可以启动一个 goroutine，让方法在这个新启动的 goroutine 中运行。运行以上示例，可以看到如下输出：</p>
<pre class="lang-java" data-nodeid="1001"><code data-language="java">我是 main goroutine
飞雪无情
</code></pre>
<p data-nodeid="1002">从输出结果也可以看出，程序是并发的，go 关键字启动的 goroutine 并不阻塞 main goroutine 的执行，所以我们才会看到如上打印结果。</p>
<blockquote data-nodeid="1003">
<p data-nodeid="1004">小提示：示例中的 time.Sleep(time.Second) 表示等待一秒，这里是让 main goroutine 等一秒，不然 main goroutine 执行完毕程序就退出了，也就看不到启动的新 goroutine 中“飞雪无情”的打印结果了。</p>
</blockquote>
<h3 data-nodeid="1005">Channel</h3>
<p data-nodeid="1006">那么如果启动了多个 goroutine，它们之间该如何通信呢？这就是 Go 语言提供的 channel（通道）要解决的问题。</p>
<h4 data-nodeid="1007">声明一个 channel</h4>
<p data-nodeid="1008">在 Go 语言中，声明一个 channel 非常简单，使用内置的 make 函数即可，如下所示：</p>
<pre class="lang-go" data-nodeid="1009"><code data-language="go">ch:=<span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">string</span>)
</code></pre>
<p data-nodeid="1010">其中 chan 是一个关键字，表示是 channel 类型。后面的 string 表示 channel 里的数据是 string 类型。通过 channel 的声明也可以看到，chan 是一个集合类型。</p>
<p data-nodeid="1011">定义好 chan 后就可以使用了，一个 chan 的操作只有两种：发送和接收。</p>
<ol data-nodeid="1012">
<li data-nodeid="1013">
<p data-nodeid="1014">接收：获取 chan 中的值，操作符为 &lt;- chan。</p>
</li>
<li data-nodeid="1015">
<p data-nodeid="1016">发送：向 chan 发送值，把值放在 chan 中，操作符为 chan &lt;-。</p>
</li>
</ol>
<blockquote data-nodeid="1017">
<p data-nodeid="1018">小技巧：这里注意发送和接收的操作符，都是 &lt;- ，只不过位置不同。接收的 &lt;- 操作符在 chan 的左侧，发送的 &lt;- 操作符在 chan 的右侧。</p>
</blockquote>
<p data-nodeid="1019">现在我把上个示例改造下，使用 chan 来代替 time.Sleep 函数的等待工作，如下面的代码所示：</p>
<p data-nodeid="1020"><em data-nodeid="1138"><strong data-nodeid="1137">ch08/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1021"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   ch:=<span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">string</span>)

   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      fmt.Println(<span class="hljs-string">"飞雪无情"</span>)
      ch &lt;- <span class="hljs-string">"goroutine 完成"</span>
   }()

   fmt.Println(<span class="hljs-string">"我是 main goroutine"</span>)

   v:=&lt;-ch
   fmt.Println(<span class="hljs-string">"接收到的chan中的值为："</span>,v)
}
</code></pre>
<p data-nodeid="1022">运行这个示例，可以发现程序并没有退出，可以看到"飞雪无情"的输出结果，达到了 time.Sleep 函数的效果，如下所示：</p>
<pre class="lang-java" data-nodeid="1023"><code data-language="java">我是 main goroutine
飞雪无情
接收到的chan中的值为： goroutine 完成
</code></pre>
<p data-nodeid="1024">可以这样理解：在上面的示例中，我们在新启动的 goroutine 中向 chan 类型的变量 ch 发送值；在 main goroutine 中，从变量 ch 接收值；如果 ch 中没有值，则阻塞等待到 ch 中有值可以接收为止。</p>
<p data-nodeid="1025">相信你应该明白为什么程序不会在新的 goroutine 完成之前退出了，因为通过 make 创建的 chan 中没有值，而 main goroutine 又想从 chan 中获取值，获取不到就一直等待，等到另一个 goroutine 向 chan 发送值为止。</p>
<p data-nodeid="1026">channel 有点像在两个 goroutine 之间架设的管道，一个 goroutine 可以往这个管道里发送数据，另外一个可以从这个管道里取数据，有点类似于我们说的队列。</p>
<h4 data-nodeid="1027">无缓冲 channel</h4>
<p data-nodeid="1028">上面的示例中，使用 make 创建的 chan 就是一个无缓冲 channel，它的容量是 0，不能存储任何数据。所以无缓冲 channel 只起到传输数据的作用，数据并不会在 channel 中做任何停留。这也意味着，无缓冲 channel 的发送和接收操作是同时进行的，它也可以称为同步 channel。</p>
<h4 data-nodeid="1029">有缓冲 channel</h4>
<p data-nodeid="1030">有缓冲 channel 类似一个可阻塞的队列，内部的元素先进先出。通过 make 函数的第二个参数可以指定 channel 容量的大小，进而创建一个有缓冲 channel，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1031"><code data-language="go">cacheCh:=<span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">int</span>,<span class="hljs-number">5</span>)
</code></pre>
<p data-nodeid="1032">我创建了一个容量为 5 的 channel，内部的元素类型是 int，也就是说这个 channel 内部最多可以存放 5 个类型为 int 的元素，如下图所示：</p>
<p data-nodeid="1033"><img src="https://s0.lgstatic.com/i/image/M00/70/AE/CgqCHl-7fzmAVLu0AACSjW-neAE188.png" alt="Drawing 2.png" data-nodeid="1154"></p>
<div data-nodeid="1034"><p style="text-align:center">（有缓冲 channel）</p></div>
<p data-nodeid="1035">一个有缓冲 channel 具备以下特点：</p>
<ol data-nodeid="1036">
<li data-nodeid="1037">
<p data-nodeid="1038">有缓冲 channel 的内部有一个缓冲队列；</p>
</li>
<li data-nodeid="1039">
<p data-nodeid="1040">发送操作是向队列的尾部插入元素，如果队列已满，则阻塞等待，直到另一个 goroutine 执行，接收操作释放队列的空间；</p>
</li>
<li data-nodeid="1041">
<p data-nodeid="1042">接收操作是从队列的头部获取元素并把它从队列中删除，如果队列为空，则阻塞等待，直到另一个 goroutine 执行，发送操作插入新的元素。</p>
</li>
</ol>
<p data-nodeid="1043">因为有缓冲 channel 类似一个队列，可以获取它的容量和里面元素的个数。如下面的代码所示：</p>
<p data-nodeid="1044"><em data-nodeid="1164"><strong data-nodeid="1163">ch08/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1045"><code data-language="go">cacheCh:=<span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">int</span>,<span class="hljs-number">5</span>)
cacheCh &lt;- <span class="hljs-number">2</span>
cacheCh &lt;- <span class="hljs-number">3</span>
fmt.Println(<span class="hljs-string">"cacheCh容量为:"</span>,<span class="hljs-built_in">cap</span>(cacheCh),<span class="hljs-string">",元素个数为："</span>,<span class="hljs-built_in">len</span>(cacheCh))
</code></pre>
<p data-nodeid="3622" class="te-preview-highlight">其中，通过内置函数 cap 可以获取 channel 的容量，也就是最大能存放多少个元素，通过内置函数 len 可以获取 channel 中元素的个数。</p>






<blockquote data-nodeid="1047">
<p data-nodeid="1048">小提示：无缓冲 channel 其实就是一个容量大小为 0 的 channel。比如 make(chan int,0)。</p>
</blockquote>
<h4 data-nodeid="1049">关闭 channel</h4>
<p data-nodeid="1050">channel 还可以使用内置函数 close 关闭，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1051"><code data-language="go"><span class="hljs-built_in">close</span>(cacheCh)
</code></pre>
<p data-nodeid="1052">如果一个 channel 被关闭了，就不能向里面发送数据了，如果发送的话，会引起 painc 异常。但是还可以接收 channel 里的数据，如果 channel 里没有数据的话，接收的数据是元素类型的零值。</p>
<h4 data-nodeid="1053">单向 channel</h4>
<p data-nodeid="1054">有时候，我们有一些特殊的业务需求，比如限制一个 channel 只可以接收但是不能发送，或者限制一个 channel 只能发送但不能接收，这种 channel 称为单向 channel。</p>
<p data-nodeid="1055">单向 channel 的声明也很简单，只需要在声明的时候带上 &lt;- 操作符即可，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1056"><code data-language="go">onlySend := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span>&lt;- <span class="hljs-keyword">int</span>)
onlyReceive:=<span class="hljs-built_in">make</span>(&lt;-<span class="hljs-keyword">chan</span> <span class="hljs-keyword">int</span>)
</code></pre>
<p data-nodeid="1057">注意，声明单向 channel &lt;- 操作符的位置和上面讲到的发送和接收操作是一样的。</p>
<p data-nodeid="1058">在函数或者方法的参数中，使用单向 channel 的较多，这样可以防止一些操作影响了 channel。</p>
<p data-nodeid="1059">下面示例中的 counter 函数，它的参数 out 是一个只能发送的 channel，所以在 counter 函数体内使用参数 out 时，只能对其进行发送操作，如果执行接收操作，则程序不能编译通过。</p>
<pre class="lang-go" data-nodeid="1060"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">counter</span><span class="hljs-params">(out <span class="hljs-keyword">chan</span>&lt;- <span class="hljs-keyword">int</span>)</span></span> {
  <span class="hljs-comment">//函数内容使用变量out，只能进行发送操作</span>
}
</code></pre>
<h3 data-nodeid="1061">select+channel 示例</h3>
<p data-nodeid="1062">假设要从网上下载一个文件，我启动了 3 个 goroutine 进行下载，并把结果发送到 3 个 channel 中。其中，哪个先下载好，就会使用哪个 channel 的结果。</p>
<p data-nodeid="1063">在这种情况下，如果我们尝试获取第一个 channel 的结果，程序就会被阻塞，无法获取剩下两个 channel 的结果，也无法判断哪个先下载好。这个时候就需要用到多路复用操作了，在 Go 语言中，通过 select 语句可以实现多路复用，其语句格式如下：</p>
<pre class="lang-go" data-nodeid="1064"><code data-language="go"><span class="hljs-keyword">select</span> {
<span class="hljs-keyword">case</span> i1 = &lt;-c1:
     <span class="hljs-comment">//todo</span>
<span class="hljs-keyword">case</span> c2 &lt;- i2:
	<span class="hljs-comment">//todo</span>
<span class="hljs-keyword">default</span>:
	<span class="hljs-comment">// default todo</span>
}
</code></pre>
<p data-nodeid="1065">整体结构和 switch 非常像，都有 case 和 default，只不过 select 的 case 是一个个可以操作的 channel。</p>
<blockquote data-nodeid="1066">
<p data-nodeid="1067">小提示：多路复用可以简单地理解为，N 个 channel 中，任意一个 channel 有数据产生，select 都可以监听到，然后执行相应的分支，接收数据并处理。</p>
</blockquote>
<p data-nodeid="1068">有了 select 语句，就可以实现下载的例子了。如下面的代码所示：</p>
<p data-nodeid="1069"><em data-nodeid="1190"><strong data-nodeid="1189">ch08/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1070"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-comment">//声明三个存放结果的channel</span>
   firstCh := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">string</span>)
   secondCh := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">string</span>)
   threeCh := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">string</span>)
   <span class="hljs-comment">//同时开启3个goroutine下载</span>
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      firstCh &lt;- downloadFile(<span class="hljs-string">"firstCh"</span>)
   }()
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      secondCh &lt;- downloadFile(<span class="hljs-string">"secondCh"</span>)
   }()
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      threeCh &lt;- downloadFile(<span class="hljs-string">"threeCh"</span>)
   }()
   <span class="hljs-comment">//开始select多路复用，哪个channel能获取到值，</span>
   <span class="hljs-comment">//就说明哪个最先下载好，就用哪个。</span>
   <span class="hljs-keyword">select</span> {
   <span class="hljs-keyword">case</span> filePath := &lt;-firstCh:
      fmt.Println(filePath)
   <span class="hljs-keyword">case</span> filePath := &lt;-secondCh:
      fmt.Println(filePath)
   <span class="hljs-keyword">case</span> filePath := &lt;-threeCh:
      fmt.Println(filePath)
   }
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">downloadFile</span><span class="hljs-params">(chanName <span class="hljs-keyword">string</span>)</span> <span class="hljs-title">string</span></span> {
   <span class="hljs-comment">//模拟下载文件,可以自己随机time.Sleep点时间试试</span>
   time.Sleep(time.Second)
   <span class="hljs-keyword">return</span> chanName+<span class="hljs-string">":filePath"</span>
}
</code></pre>
<p data-nodeid="1071">如果这些 case 中有一个可以执行，select 语句会选择该 case 执行，如果同时有多个 case 可以被执行，则随机选择一个，这样每个 case 都有平等的被执行的机会。如果一个 select 没有任何 case，那么它会一直等待下去。</p>
<h3 data-nodeid="1072">总结</h3>
<p data-nodeid="1073">在这节课中，我为你介绍了如何通过 go 关键字启动一个 goroutine，以及如何通过 channel 实现 goroutine 间的数据传递，这些都是 Go 语言并发的基础，理解它们可以更好地掌握并发。</p>
<p data-nodeid="1074">在 Go 语言中，提倡通过通信来共享内存，而不是通过共享内存来通信，其实就是提倡通过 channel 发送接收消息的方式进行数据传递，而不是通过修改同一个变量。所以在<strong data-nodeid="1198">数据流动、传递的场景中要优先使用 channel，它是并发安全的，性能也不错。</strong></p>
<p data-nodeid="1075"><img src="https://s0.lgstatic.com/i/image/M00/70/AF/CgqCHl-7f1eAOkA5AAUt2ZtY7Ec582.png" alt="Drawing 3.png" data-nodeid="1201"></p>
<p data-nodeid="1076">到这里就要结束今天的课程了，本节课留个思考题，猜一猜 channel 是怎么做到并发安全的？</p>
<p data-nodeid="1077" class="">下节课我们要学习第 9 讲“同步原语：sync 包让你对并发控制得心应手”，记得来听课！</p>

---

### 精选评论

##### **方：
> 如果不设置等待时间或者不等待从channel中取数据， 那goroutine是不是就不会执行，主goroutine执行完就算结束了？但实际开发中有些比如发送邮件/写日志这种IO操作，需要保证执行成功，但主goroutine不需要从IO操作中取值，该怎么实现呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 以API服务为例，它的整个主goroutine就是阻塞的，
因为它要不停的处理HTTP请求，所以你可以不用担心
主goroutine退出。

##### ezra.xu：
> 把发送，接收动作队列化，底层我猜可能用到了cpu原语

##### *涛：
> channel就是安全栈这种数据结构，go把他发挥到了极致，赞！

##### *溶：
> java中的流和有界阻塞队列在这被合并了…

##### **nathan大聖：
> select能实现那三个下载的协程都完成了再往后执行吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是，有一个下载完成就会执行

##### *阳：
> 讲的很好，通俗易懂。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢支持

##### **恩：
> 现在才知道原来有单向channel，这个很实用啊~

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油学习

##### *珣：
> 其实channel应该就是一个线程安全的队列，只不过它在实现线程安全的时候做的比较高明吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 大体是这样的，不过也算不上高明，当然Go语言设计者写的代码，性能肯定是不错的。

##### **繁：
> 只能接收的channel 有啥用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 防止channel的消费者因为误操作往channel里发送了数据

