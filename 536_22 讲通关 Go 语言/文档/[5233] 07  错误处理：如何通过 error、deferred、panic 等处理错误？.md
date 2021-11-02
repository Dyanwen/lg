<p data-nodeid="1081">上节课我为你讲解了结构体和接口，并留了一个小作业，让你自己练习实现有两个方法的接口。现在我就以“人既会走也会跑”为例进行讲解。</p>
<p data-nodeid="1082">首先定义一个接口 WalkRun，它有两个方法 Walk 和 Run，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1083"><code data-language="go"><span class="hljs-keyword">type</span> WalkRun <span class="hljs-keyword">interface</span> {
   Walk()
   Run()
}
</code></pre>
<p data-nodeid="1084">现在就可以让结构体 person 实现这个接口了，如下所示：</p>
<pre class="lang-go" data-nodeid="1085"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(p *person)</span> <span class="hljs-title">Walk</span><span class="hljs-params">()</span></span>{
   fmt.Printf(<span class="hljs-string">"%s能走\n"</span>,p.name)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(p *person)</span> <span class="hljs-title">Run</span><span class="hljs-params">()</span></span>{
   fmt.Printf(<span class="hljs-string">"%s能跑\n"</span>,p.name)
}
</code></pre>
<p data-nodeid="1086">关键点在于，让接口的每个方法都实现，也就实现了这个接口。</p>
<blockquote data-nodeid="1087">
<p data-nodeid="1088">提示：%s 是占位符，和 p.name 对应，也就是 p.name 的值，具体可以参考 fmt.Printf 函数的文档。</p>
</blockquote>
<p data-nodeid="1089">下面进行本节课的讲解。这节课我会带你学习 Go 语言的错误和异常，在我们编写程序的时候，可能会遇到一些问题，该怎么处理它们呢？</p>
<h3 data-nodeid="1090">错误</h3>
<p data-nodeid="1091">在 Go 语言中，错误是可以预期的，并且不是非常严重，不会影响程序的运行。对于这类问题，可以用返回错误给调用者的方法，让调用者自己决定如何处理。</p>
<h4 data-nodeid="1092">error 接口</h4>
<p data-nodeid="1093">在 Go 语言中，错误是通过内置的 error 接口表示的。它非常简单，只有一个 Error 方法用来返回具体的错误信息，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1094"><code data-language="go"><span class="hljs-keyword">type</span> error <span class="hljs-keyword">interface</span> {
   Error() <span class="hljs-keyword">string</span>
}
</code></pre>
<p data-nodeid="1095">在下面的代码中，我演示了一个字符串转整数的例子：</p>
<p data-nodeid="1096"><em data-nodeid="1213"><strong data-nodeid="1212">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1097"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   i,err:=strconv.Atoi(<span class="hljs-string">"a"</span>)
   <span class="hljs-keyword">if</span> err!=<span class="hljs-literal">nil</span> {
      fmt.Println(err)
   }<span class="hljs-keyword">else</span> {
      fmt.Println(i)
   }
}
</code></pre>
<p data-nodeid="1098">这里我故意使用了字符串 "a"，尝试把它转为整数。我们知道 "a" 是无法转为数字的，所以运行这段程序，会打印出如下错误信息：</p>
<pre class="lang-java" data-nodeid="1099"><code data-language="java">strconv.Atoi: parsing <span class="hljs-string">"a"</span>: invalid syntax
</code></pre>
<p data-nodeid="1100">这个错误信息就是通过接口 error 返回的。我们来看关于函数 strconv.Atoi 的定义，如下所示：</p>
<pre class="lang-go" data-nodeid="1101"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Atoi</span><span class="hljs-params">(s <span class="hljs-keyword">string</span>)</span> <span class="hljs-params">(<span class="hljs-keyword">int</span>, error)</span></span>
</code></pre>
<p data-nodeid="1102">一般而言，error 接口用于当方法或者函数执行遇到错误时进行返回，而且是第二个返回值。通过这种方式，可以让调用者自己根据错误信息决定如何进行下一步处理。</p>
<blockquote data-nodeid="1103">
<p data-nodeid="1104">小提示：因为方法和函数基本上差不多，区别只在于有无接收者，所以以后当我称方法或函数，表达的是一个意思，不会把这两个名字都写出来。</p>
</blockquote>
<h4 data-nodeid="1105">error 工厂函数</h4>
<p data-nodeid="1106">除了可以使用其他函数，自己定义的函数也可以返回错误信息给调用者，如下面的代码所示：</p>
<p data-nodeid="1107"><em data-nodeid="1232"><strong data-nodeid="1231">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1108"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">add</span><span class="hljs-params">(a,b <span class="hljs-keyword">int</span>)</span> <span class="hljs-params">(<span class="hljs-keyword">int</span>,error)</span></span>{
   <span class="hljs-keyword">if</span> a&lt;<span class="hljs-number">0</span> || b&lt;<span class="hljs-number">0</span> {
      <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>,errors.New(<span class="hljs-string">"a或者b不能为负数"</span>)
   }<span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span> a+b,<span class="hljs-literal">nil</span>
   }
}
</code></pre>
<p data-nodeid="1109">add 函数会在 a 或者 b 任何一个为负数的情况下，返回一个错误信息，如果 a、b 都不为负数，错误信息部分会返回 nil，这也是常见的做法。所以调用者可以通过错误信息是否为 nil 进行判断。</p>
<p data-nodeid="1110">下面的 add 函数示例，是使用 errors.New 这个工厂函数生成的错误信息，它接收一个字符串参数，返回一个 error 接口，这些在上节课的结构体和接口部分有过详细介绍，不再赘述。</p>
<p data-nodeid="1111"><em data-nodeid="1239"><strong data-nodeid="1238">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1112"><code data-language="go">sum,err:=add(<span class="hljs-number">-1</span>,<span class="hljs-number">2</span>)
<span class="hljs-keyword">if</span> err!=<span class="hljs-literal">nil</span> {
   fmt.Println(err)
}<span class="hljs-keyword">else</span> {
   fmt.Println(sum)
}
</code></pre>
<h4 data-nodeid="1113">自定义 error</h4>
<p data-nodeid="1114">你可能会想，上面采用工厂返回错误信息的方式只能传递一个字符串，也就是携带的信息只有字符串，如果想要携带更多信息（比如错误码信息）该怎么办呢？这个时候就需要自定义 error 。</p>
<p data-nodeid="1115">自定义 error 其实就是先自定义一个新类型，比如结构体，然后让这个类型实现 error 接口，如下面的代码所示：</p>
<p data-nodeid="1116"><em data-nodeid="1247"><strong data-nodeid="1246">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1117"><code data-language="go"><span class="hljs-keyword">type</span> commonError <span class="hljs-keyword">struct</span> {
   errorCode <span class="hljs-keyword">int</span> <span class="hljs-comment">//错误码</span>
   errorMsg <span class="hljs-keyword">string</span> <span class="hljs-comment">//错误信息</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(ce *commonError)</span> <span class="hljs-title">Error</span><span class="hljs-params">()</span> <span class="hljs-title">string</span></span>{
   <span class="hljs-keyword">return</span> ce.errorMsg
}
</code></pre>
<p data-nodeid="1118">有了自定义的 error，就可以使用它携带更多的信息，现在我改造上面的例子，返回刚刚自定义的 commonError，如下所示：</p>
<p data-nodeid="1119"><em data-nodeid="1253"><strong data-nodeid="1252">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1120"><code data-language="go"><span class="hljs-keyword">return</span> <span class="hljs-number">0</span>, &amp;commonError{
   errorCode: <span class="hljs-number">1</span>,
   errorMsg:  <span class="hljs-string">"a或者b不能为负数"</span>}
</code></pre>
<p data-nodeid="1121">我通过字面量的方式创建一个 *commonError 返回，其中 errorCode 值为 1，errorMsg 值为 “a 或者 b 不能为负数”。</p>
<h4 data-nodeid="1122">error 断言</h4>
<p data-nodeid="1123">有了自定义的 error，并且携带了更多的错误信息后，就可以使用这些信息了。你需要先把返回的 error 接口转换为自定义的错误类型，用到的知识是上节课的类型断言。</p>
<p data-nodeid="1124">下面代码中的 err.(*commonError) 就是类型断言在 error 接口上的应用，也可以称为 error 断言。</p>
<p data-nodeid="1125"><em data-nodeid="1266"><strong data-nodeid="1265">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1126"><code data-language="go">sum, err := add(<span class="hljs-number">-1</span>, <span class="hljs-number">2</span>)
<span class="hljs-keyword">if</span> cm,ok:=err.(*commonError);ok{
   fmt.Println(<span class="hljs-string">"错误代码为:"</span>,cm.errorCode,<span class="hljs-string">"，错误信息为："</span>,cm.errorMsg)
} <span class="hljs-keyword">else</span> {
   fmt.Println(sum)
}
</code></pre>
<p data-nodeid="1127">如果返回的 ok 为 true，说明 error 断言成功，正确返回了 *commonError 类型的变量 cm，所以就可以像示例中一样使用变量 cm 的 errorCode 和 errorMsg 字段信息了。</p>
<h3 data-nodeid="1128">错误嵌套</h3>
<h4 data-nodeid="1129">Error Wrapping</h4>
<p data-nodeid="1130">error 接口虽然比较简洁，但是功能也比较弱。想象一下，假如我们有这样的需求：基于一个存在的 error 再生成一个 error，需要怎么做呢？这就是错误嵌套。</p>
<p data-nodeid="1131">这种需求是存在的，比如调用一个函数，返回了一个错误信息 error，在不想丢失这个 error 的情况下，又想添加一些额外信息返回新的 error。这时候，我们首先想到的应该是自定义一个 struct，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1132"><code data-language="go"><span class="hljs-keyword">type</span>&nbsp;MyError&nbsp;<span class="hljs-keyword">struct</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;err&nbsp;error
&nbsp;&nbsp;&nbsp;&nbsp;msg&nbsp;<span class="hljs-keyword">string</span>
}
</code></pre>
<p data-nodeid="1133">这个结构体有两个字段，其中 error 类型的 err 字段用于存放已存在的 error，string 类型的 msg 字段用于存放新的错误信息，<strong data-nodeid="1279">这种方式就是 error 的嵌套</strong>。</p>
<p data-nodeid="1134">现在让 MyError 这个 struct 实现 error 接口，然后在初始化 MyError 的时候传递存在的 error 和新的错误信息，如下面的代码所示：</p>
<pre class="lang-go" data-nodeid="1135"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span>&nbsp;<span class="hljs-params">(e&nbsp;*MyError)</span>&nbsp;<span class="hljs-title">Error</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-title">string</span></span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;e.err.Error()&nbsp;+&nbsp;e.msg
}
<span class="hljs-function"><span class="hljs-keyword">func</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">()</span></span>&nbsp;{
    <span class="hljs-comment">//err是一个存在的错误，可以从另外一个函数返回</span>
&nbsp;&nbsp;&nbsp;&nbsp;newErr&nbsp;:=&nbsp;MyError{err,&nbsp;<span class="hljs-string">"数据上传问题"</span>}
}
</code></pre>
<p data-nodeid="1136">这种方式可以满足我们的需求，但是非常烦琐，因为既要定义新的类型还要实现 error 接口。所以从 Go 语言 1.13 版本开始，Go 标准库新增了 Error Wrapping 功能，让我们可以基于一个存在的 error 生成新的 error，并且可以保留原 error 信息，如下面的代码所示：</p>
<p data-nodeid="1137"><em data-nodeid="1286"><strong data-nodeid="1285">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1138"><code data-language="go">e := errors.New(<span class="hljs-string">"原始错误e"</span>)
w := fmt.Errorf(<span class="hljs-string">"Wrap了一个错误:%w"</span>, e)
fmt.Println(w)
</code></pre>
<p data-nodeid="1139">Go 语言没有提供 Wrap 函数，而是扩展了 fmt.Errorf 函数，然后加了一个 %w，通过这种方式，便可以生成 wrapping error。</p>
<h4 data-nodeid="1140">errors.Unwrap 函数</h4>
<p data-nodeid="1141">既然 error 可以包裹嵌套生成一个新的 error，那么也可以被解开，即通过 errors.Unwrap 函数得到被嵌套的 error。</p>
<p data-nodeid="1142">Go 语言提供了 errors.Unwrap 用于获取被嵌套的 error，比如以上例子中的错误变量 w ，就可以对它进行 unwrap，获取被嵌套的原始错误 e。</p>
<p data-nodeid="1143">下面我们运行以下代码：</p>
<pre class="lang-go" data-nodeid="1144"><code data-language="go">fmt.Println(errors.Unwrap(w))
</code></pre>
<p data-nodeid="1145">可以看到这样的信息，即“原始错误 e”。</p>
<pre class="lang-java" data-nodeid="1146"><code data-language="java">原始错误e
</code></pre>
<h4 data-nodeid="1147">errors.Is 函数</h4>
<p data-nodeid="1148">有了 Error Wrapping 后，你会发现原来用的判断两个 error 是不是同一个 error 的方法失效了，比如 Go 语言标准库经常用到的如下代码中的方式：</p>
<pre class="lang-go" data-nodeid="1149"><code data-language="go"><span class="hljs-keyword">if</span>&nbsp;err&nbsp;==&nbsp;os.ErrExist
</code></pre>
<p data-nodeid="1150">为什么会出现这种情况呢？由于 Go 语言的 Error Wrapping 功能，令人不知道返回的 err 是否被嵌套，又嵌套了几层？</p>
<p data-nodeid="1151">于是 Go 语言为我们提供了 errors.Is 函数，用来判断两个 error 是否是同一个，如下所示：</p>
<pre class="lang-go" data-nodeid="1152"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span>&nbsp;<span class="hljs-title">Is</span><span class="hljs-params">(err,&nbsp;target&nbsp;error)</span>&nbsp;<span class="hljs-title">bool</span></span>
</code></pre>
<p data-nodeid="1153">以上就是errors.Is 函数的定义，可以解释为：</p>
<ul data-nodeid="1154">
<li data-nodeid="1155">
<p data-nodeid="1156">如果 err 和 target 是同一个，那么返回 true。</p>
</li>
<li data-nodeid="1157">
<p data-nodeid="1158">如果 err 是一个 wrapping error，target 也包含在这个嵌套 error 链中的话，也返回 true。</p>
</li>
</ul>
<p data-nodeid="1159">可以简单地概括为，两个 error 相等或 err 包含 target 的情况下返回 true，其余返回 false。我们可以用上面的示例判断错误 w 中是否包含错误 e，试试运行下面的代码，来看打印的结果是不是 true。</p>
<pre class="lang-go" data-nodeid="1160"><code data-language="go">fmt.Println(errors.Is(w,e))
</code></pre>
<h4 data-nodeid="1161">errors.As 函数</h4>
<p data-nodeid="1162">同样的原因，有了 error 嵌套后，error 断言也不能用了，因为你不知道一个 error 是否被嵌套，又嵌套了几层。所以 Go 语言为解决这个问题提供了 errors.As 函数，比如前面 error 断言的例子，可以使用 errors.As 函数重写，效果是一样的，如下面的代码所示：</p>
<p data-nodeid="1163"><em data-nodeid="1307"><strong data-nodeid="1306">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1164"><code data-language="go"><span class="hljs-keyword">var</span> cm *commonError
<span class="hljs-keyword">if</span> errors.As(err,&amp;cm){
   fmt.Println(<span class="hljs-string">"错误代码为:"</span>,cm.errorCode,<span class="hljs-string">"，错误信息为："</span>,cm.errorMsg)
} <span class="hljs-keyword">else</span> {
   fmt.Println(sum)
}
</code></pre>
<p data-nodeid="1165">所以在 Go 语言提供的 Error Wrapping 能力下，我们写的代码要尽可能地使用 Is、As 这些函数做判断和转换。</p>
<h3 data-nodeid="1166">Deferred 函数</h3>
<p data-nodeid="1167">在一个自定义函数中，你打开了一个文件，然后需要关闭它以释放资源。不管你的代码执行了多少分支，是否出现了错误，文件是一定要关闭的，这样才能保证资源的释放。</p>
<p data-nodeid="1168">如果这个事情由开发人员来做，随着业务逻辑的复杂会变得非常麻烦，而且还有可能会忘记关闭。基于这种情况，Go 语言为我们提供了 defer 函数，可以保证文件关闭后一定会被执行，不管你自定义的函数出现异常还是错误。</p>
<p data-nodeid="1169">下面的代码是 Go 语言标准包 ioutil 中的 ReadFile 函数，它需要打开一个文件，然后通过 defer 关键字确保在 ReadFile 函数执行结束后，f.Close() 方法被执行，这样文件的资源才一定会释放。</p>
<pre class="lang-go" data-nodeid="1170"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">ReadFile</span><span class="hljs-params">(filename <span class="hljs-keyword">string</span>)</span> <span class="hljs-params">([]<span class="hljs-keyword">byte</span>, error)</span></span> {
   f, err := os.Open(filename)
   <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>, err
   }
   <span class="hljs-keyword">defer</span> f.Close()
   <span class="hljs-comment">//省略无关代码</span>
   <span class="hljs-keyword">return</span> readAll(f, n)
}
</code></pre>
<p data-nodeid="1171">defer 关键字用于修饰一个函数或者方法，使得该函数或者方法在返回前才会执行，也就说被延迟，但又可以保证一定会执行。</p>
<p data-nodeid="1172">以上面的 ReadFile 函数为例，被 defer 修饰的 f.Close 方法延迟执行，也就是说会先执行 readAll(f, n)，然后在整个 ReadFile 函数 return 之前执行 f.Close 方法。</p>
<p data-nodeid="1173">defer 语句常被用于成对的操作，如文件的打开和关闭，加锁和释放锁，连接的建立和断开等。不管多么复杂的操作，都可以保证资源被正确地释放。</p>
<h3 data-nodeid="1174">Panic 异常</h3>
<p data-nodeid="1175">Go 语言是一门静态的强类型语言，很多问题都尽可能地在编译时捕获，但是有一些只能在运行时检查，比如数组越界访问、不相同的类型强制转换等，这类运行时的问题会引起 panic 异常。</p>
<p data-nodeid="1176">除了运行时可以产生 panic 外，我们自己也可以抛出 panic 异常。假设我需要连接 MySQL 数据库，可以写一个连接 MySQL 的函数connectMySQL，如下面的代码所示：</p>
<p data-nodeid="1177"><em data-nodeid="1323"><strong data-nodeid="1322">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1178"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">connectMySQL</span><span class="hljs-params">(ip,username,password <span class="hljs-keyword">string</span>)</span></span>{
   <span class="hljs-keyword">if</span> ip ==<span class="hljs-string">""</span> {
      <span class="hljs-built_in">panic</span>(<span class="hljs-string">"ip不能为空"</span>)
   }
   <span class="hljs-comment">//省略其他代码</span>
}
</code></pre>
<p data-nodeid="1179">在 connectMySQL 函数中，如果 ip 为空会直接抛出 panic 异常。这种逻辑是正确的，因为数据库无法连接成功的话，整个程序运行起来也没有意义，所以就抛出 panic 终止程序的运行。</p>
<p data-nodeid="1180">panic 是 Go 语言内置的函数，可以接受 interface{} 类型的参数，也就是任何类型的值都可以传递给 panic 函数，如下所示：</p>
<pre class="lang-go" data-nodeid="1181"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">panic</span><span class="hljs-params">(v <span class="hljs-keyword">interface</span>{})</span></span>
</code></pre>
<blockquote data-nodeid="1182">
<p data-nodeid="1183">小提示：interface{} 是空接口的意思，在 Go 语言中代表任意类型。</p>
</blockquote>
<p data-nodeid="1907">panic 异常是一种非常严重的情况，会让程序中断运行，使程序崩溃，所以<strong data-nodeid="1913">如果是不影响程序运行的错误，不要使用 panic，使用普通错误 error 即可。</strong></p>
<p data-nodeid="1908"><img alt="pDE7ppQNyfRSIn1Q__thumbnail.png" src="https://s0.lgstatic.com/i/image/M00/6F/79/CgqCHl-15ZSAAsw5AAUnpsfN34w061.png" data-nodeid="1918"></p>

<h3 data-nodeid="1630">Recover 捕获 Panic 异常</h3>




<p data-nodeid="1186">通常情况下，我们不对 panic 异常做任何处理，因为既然它是影响程序运行的异常，就让它直接崩溃即可。但是也的确有一些特例，比如在程序崩溃前做一些资源释放的处理，这时候就需要从 panic 异常中恢复，才能完成处理。</p>
<p data-nodeid="1187">在 Go 语言中，可以通过内置的 recover 函数恢复 panic 异常。因为在程序 panic 异常崩溃的时候，只有被 defer 修饰的函数才能被执行，所以 recover 函数要结合 defer 关键字使用才能生效。</p>
<p data-nodeid="1188">下面的示例是通过 defer 关键字 + 匿名函数 + recover 函数从 panic 异常中恢复的方式。</p>
<p data-nodeid="1189"><em data-nodeid="1340"><strong data-nodeid="1339">ch07/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="1190"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">defer</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">if</span> p:=<span class="hljs-built_in">recover</span>();p!=<span class="hljs-literal">nil</span>{
         fmt.Println(p)
      }
   }()
   connectMySQL(<span class="hljs-string">""</span>,<span class="hljs-string">"root"</span>,<span class="hljs-string">"123456"</span>)
}
</code></pre>
<p data-nodeid="1191">运行这个代码，可以看到如下的打印输出，这证明 recover 函数成功捕获了 panic 异常。</p>
<pre class="lang-java" data-nodeid="1192"><code data-language="java">ip 不能为空
</code></pre>
<p data-nodeid="1193">通过这个输出的结果也可以发现，recover 函数返回的值就是通过 panic 函数传递的参数值。</p>
<h3 data-nodeid="1194">总结</h3>
<p data-nodeid="1195">这节课主要讲了 Go 语言的错误处理机制，包括 error、defer、panic 等。在 error、panic 这两种错误机制中，Go 语言更提倡 error 这种轻量错误，而不是 panic。</p>
<p data-nodeid="1196"><strong data-nodeid="1349">本节课的思考题是</strong>：一个函数中可以有多个 defer 语句吗？如果可以的话，它们的执行顺序是什么？可以先思考一下，然后通过写代码的方式验证是否正确。</p>
<p data-nodeid="1197">下节课我们进入本专栏的第二模块：Go 语言的高效并发。我将首先讲解“并发基础：Goroutines 和 Channels 的声明与使用”，记得来听课！</p>

---

### 精选评论

##### **杰：
> 可以有多个defer，倒序执行，和堆栈一样先进后出

##### kopelan：
> 请教下老师，golang中的格式化fmt.Printf("%s：my name is %s","lilei","lilei")，这段代码两个需要替换的字符串都是“lilei”,这有没有简单的方式，比如参数写一个lilei，就可以替换两处？因为在格式化时经常会遇到一段话里面相同的值出现两次或多次，怎么处理才比较好

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有呢，要一一对应。

##### *平：
> errors.Is() 和errors.As() 不是很理解，老师能补充一下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要自己写代码练习下，一定要写出来，再结合文章，就可以很好的理解了。加油！

##### *靖：
> 停不下来了，一天7章，学的似懂非懂，直接干。往后学

##### 张：
> interface{} 是不是相当于java中的泛型和Object?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 和Object差不多，但是最好不要这么对比。

##### xwy：
> sum, err := add(-1, 2)if cm,ok:=err.(*commonError);ok{fmt.Println("错误代码为:",cm.errorCode,"，错误信息为：",cm.errorMsg)} else {fmt.Println(sum)}请问此处的err是接口值还是commentError类型值？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在这里err本质上是一个commentError指针类型，而commentError指针类型又实现了error接口。

##### **峰：
> 1. 关于error，error就是程序执行的错误，是需要代码进行错误处理和规避的。要了解error的处理机制，掌握error封装的方法和API2. 关于defer函数，帮我们延迟处理方法或者代码，可以更方便得进行资源释放等操作3. 关于panic，panic是运行时异常，要警惕程序出现，可以使用recovery来进行异常处理，防止程序崩溃

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 好认真！

##### **升：
> 确实go的异常处理非常灵活啊

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 强大的Go

