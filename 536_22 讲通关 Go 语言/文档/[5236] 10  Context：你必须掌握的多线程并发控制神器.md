<p data-nodeid="18663">在上一节课中我留了一个作业，也就是让你自己练习使用 sync.Map，相信你已经做出来了。现在我为你讲解 sync.Map 的方法。</p>


<ol data-nodeid="17741">
<li data-nodeid="17742">
<p data-nodeid="17743"><strong data-nodeid="17871">Store</strong>：存储一对 key-value 值。</p>
</li>
<li data-nodeid="17744">
<p data-nodeid="17745"><strong data-nodeid="17876">Load</strong>：根据 key 获取对应的 value 值，并且可以判断 key 是否存在。</p>
</li>
<li data-nodeid="17746">
<p data-nodeid="17747"><strong data-nodeid="17881">LoadOrStore</strong>：如果 key 对应的 value 存在，则返回该 value；如果不存在，存储相应的 value。</p>
</li>
<li data-nodeid="17748">
<p data-nodeid="17749"><strong data-nodeid="17886">Delete</strong>：删除一个 key-value 键值对。</p>
</li>
<li data-nodeid="17750">
<p data-nodeid="17751"><strong data-nodeid="17891">Range</strong>：循环迭代 sync.Map，效果与 for range 一样。</p>
</li>
</ol>
<p data-nodeid="17752">相信有了这些方法的介绍，你对 sync.Map 会有更深入的理解。下面开始今天的课程：如何通过 Context 更好地控制并发。</p>
<h3 data-nodeid="19277" class="">协程如何退出</h3>

<p data-nodeid="17754">一个协程启动后，大部分情况需要等待里面的代码执行完毕，然后协程会自行退出。但是如果有一种情景，需要让协程提前退出怎么办呢？在下面的代码中，我做了一个监控狗用来监控程序：</p>
<p data-nodeid="17755"><em data-nodeid="17899"><strong data-nodeid="17898">ch10/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="17756"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">var</span> wg sync.WaitGroup
   wg.Add(<span class="hljs-number">1</span>)
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">defer</span> wg.Done()
      watchDog(<span class="hljs-string">"【监控狗1】"</span>)
   }()
   wg.Wait()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">watchDog</span><span class="hljs-params">(name <span class="hljs-keyword">string</span>)</span></span>{
   <span class="hljs-comment">//开启for select循环，一直后台监控</span>
   <span class="hljs-keyword">for</span>{
      <span class="hljs-keyword">select</span> {
      <span class="hljs-keyword">default</span>:
         fmt.Println(name,<span class="hljs-string">"正在监控……"</span>)
      }
      time.Sleep(<span class="hljs-number">1</span>*time.Second)
   }
}
</code></pre>
<p data-nodeid="17757">我通过 watchDog 函数实现了一个监控狗，它会一直在后台运行，每隔一秒就会打印"监控狗正在监控……"的文字。</p>
<p data-nodeid="17758">如果需要让监控狗停止监控、退出程序，一个办法是定义一个全局变量，其他地方可以通过修改这个变量发出停止监控狗的通知。然后在协程中先检查这个变量，如果发现被通知关闭就停止监控，退出当前协程。</p>
<p data-nodeid="17759">但是这种方法需要通过加锁来保证多协程下并发的安全，基于这个思路，有个升级版的方案：用 select+channel 做检测，如下面的代码所示：</p>
<p data-nodeid="17760"><em data-nodeid="17911"><strong data-nodeid="17910">ch10/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="17761"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">var</span> wg sync.WaitGroup
   wg.Add(<span class="hljs-number">1</span>)
   stopCh := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">bool</span>) <span class="hljs-comment">//用来停止监控狗</span>
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">defer</span> wg.Done()
      watchDog(stopCh,<span class="hljs-string">"【监控狗1】"</span>)
   }()
   time.Sleep(<span class="hljs-number">5</span> * time.Second) <span class="hljs-comment">//先让监控狗监控5秒</span>
   stopCh &lt;- <span class="hljs-literal">true</span> <span class="hljs-comment">//发停止指令</span>
   wg.Wait()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">watchDog</span><span class="hljs-params">(stopCh <span class="hljs-keyword">chan</span> <span class="hljs-keyword">bool</span>,name <span class="hljs-keyword">string</span>)</span></span>{
   <span class="hljs-comment">//开启for select循环，一直后台监控</span>
   <span class="hljs-keyword">for</span>{
      <span class="hljs-keyword">select</span> {
      <span class="hljs-keyword">case</span> &lt;-stopCh:
         fmt.Println(name,<span class="hljs-string">"停止指令已收到，马上停止"</span>)
         <span class="hljs-keyword">return</span>
      <span class="hljs-keyword">default</span>:
         fmt.Println(name,<span class="hljs-string">"正在监控……"</span>)
      }
      time.Sleep(<span class="hljs-number">1</span>*time.Second)
   }
}
</code></pre>
<p data-nodeid="17762">这个示例是使用 select+channel 的方式改造的 watchDog 函数，实现了通过 channel 发送指令让监控狗停止，进而达到协程退出的目的。以上示例主要有两处修改，具体如下：</p>
<ol data-nodeid="17763">
<li data-nodeid="17764">
<p data-nodeid="17765">为 watchDog 函数增加 stopCh 参数，用于接收停止指令；</p>
</li>
<li data-nodeid="17766">
<p data-nodeid="17767">在 main 函数中，声明用于停止的 stopCh，传递给 watchDog 函数，然后通过 stopCh&lt;-true 发送停止指令让协程退出。</p>
</li>
</ol>
<h3 data-nodeid="19891" class="">初识 Context</h3>

<p data-nodeid="17769">以上示例是 select+channel 比较经典的使用场景，这里也顺便复习了 select 的知识。</p>
<p data-nodeid="17770">通过 select+channel 让协程退出的方式比较优雅，但是如果我们希望做到同时取消很多个协程呢？如果是定时取消协程又该怎么办？这时候 select+channel 的局限性就凸现出来了，即使定义了多个 channel 解决问题，代码逻辑也会非常复杂、难以维护。</p>
<p data-nodeid="17771">要解决这种复杂的协程问题，必须有一种可以跟踪协程的方案，只有跟踪到每个协程，才能更好地控制它们，这种方案就是 Go 语言标准库为我们提供的 Context，也是这节课的主角。</p>
<p data-nodeid="17772">现在我通过 Context 重写上面的示例，实现让监控狗停止的功能，如下所示：</p>
<p data-nodeid="17773"><em data-nodeid="17926"><strong data-nodeid="17925">ch10/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="17774"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">var</span> wg sync.WaitGroup
   wg.Add(<span class="hljs-number">1</span>)
   ctx,stop:=context.WithCancel(context.Background())
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">defer</span> wg.Done()
      watchDog(ctx,<span class="hljs-string">"【监控狗1】"</span>)
   }()
   time.Sleep(<span class="hljs-number">5</span> * time.Second) <span class="hljs-comment">//先让监控狗监控5秒</span>
   stop() <span class="hljs-comment">//发停止指令</span>
   wg.Wait()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">watchDog</span><span class="hljs-params">(ctx context.Context,name <span class="hljs-keyword">string</span>)</span></span> {
   <span class="hljs-comment">//开启for select循环，一直后台监控</span>
   <span class="hljs-keyword">for</span> {
      <span class="hljs-keyword">select</span> {
      <span class="hljs-keyword">case</span> &lt;-ctx.Done():
         fmt.Println(name,<span class="hljs-string">"停止指令已收到，马上停止"</span>)
         <span class="hljs-keyword">return</span>
      <span class="hljs-keyword">default</span>:
         fmt.Println(name,<span class="hljs-string">"正在监控……"</span>)
      }
      time.Sleep(<span class="hljs-number">1</span> * time.Second)
   }
}
</code></pre>
<p data-nodeid="17775">相比 select+channel 的方案，Context 方案主要有 4 个改动点。</p>
<ol data-nodeid="17776">
<li data-nodeid="17777">
<p data-nodeid="17778">watchDog 的 stopCh 参数换成了 ctx，类型为 context.Context。</p>
</li>
<li data-nodeid="17779">
<p data-nodeid="17780">原来的 case &lt;-stopCh 改为 case &lt;-ctx.Done()，用于判断是否停止。</p>
</li>
<li data-nodeid="17781">
<p data-nodeid="17782">使用 context.WithCancel(context.Background()) 函数生成一个可以取消的 Context，用于发送停止指令。这里的 context.Background() 用于生成一个空 Context，一般作为整个 Context 树的根节点。</p>
</li>
<li data-nodeid="17783">
<p data-nodeid="17784">原来的 stopCh &lt;- true 停止指令，改为 context.WithCancel 函数返回的取消函数 stop()。</p>
</li>
</ol>
<p data-nodeid="17785">可以看到，这和修改前的整体代码结构一样，只不过从 channel 换成了 Context。以上示例只是 Context 的一种使用场景，它的能力不止于此，现在我来介绍什么是 Context。</p>
<h2 data-nodeid="17786">什么是 Context</h2>
<p data-nodeid="17787">一个任务会有很多个协程协作完成，一次 HTTP 请求也会触发很多个协程的启动，而这些协程有可能会启动更多的子协程，并且无法预知有多少层协程、每一层有多少个协程。</p>
<p data-nodeid="17788">如果因为某些原因导致任务终止了，HTTP 请求取消了，那么它们启动的协程怎么办？该如何取消呢？因为取消这些协程可以节约内存，提升性能，同时避免不可预料的 Bug。</p>
<p data-nodeid="17789">Context 就是用来简化解决这些问题的，并且是并发安全的。Context 是一个接口，它具备手动、定时、超时发出取消信号、传值等功能，主要用于控制多个协程之间的协作，尤其是取消操作。一旦取消指令下达，那么被 Context 跟踪的这些协程都会收到取消信号，就可以做清理和退出操作。</p>
<p data-nodeid="17790">Context 接口只有四个方法，下面进行详细介绍，在开发中你会经常使用它们，你可以结合下面的代码来看。</p>
<pre class="lang-go" data-nodeid="17791"><code data-language="go"><span class="hljs-keyword">type</span> Context <span class="hljs-keyword">interface</span> {
   Deadline() (deadline time.Time, ok <span class="hljs-keyword">bool</span>)
   Done() &lt;-<span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
   Err() error
   Value(key <span class="hljs-keyword">interface</span>{}) <span class="hljs-keyword">interface</span>{}
}
</code></pre>
<ol data-nodeid="17792">
<li data-nodeid="17793">
<p data-nodeid="17794">Deadline 方法可以获取设置的截止时间，第一个返回值 deadline 是截止时间，到了这个时间点，Context 会自动发起取消请求，第二个返回值 ok 代表是否设置了截止时间。</p>
</li>
<li data-nodeid="17795">
<p data-nodeid="17796">Done 方法返回一个只读的 channel，类型为 struct{}。在协程中，如果该方法返回的 chan 可以读取，则意味着 Context 已经发起了取消信号。通过 Done 方法收到这个信号后，就可以做清理操作，然后退出协程，释放资源。</p>
</li>
<li data-nodeid="17797">
<p data-nodeid="17798">Err 方法返回取消的错误原因，即因为什么原因 Context 被取消。</p>
</li>
<li data-nodeid="17799">
<p data-nodeid="17800">Value 方法获取该 Context 上绑定的值，是一个键值对，所以要通过一个 key 才可以获取对应的值。</p>
</li>
</ol>
<p data-nodeid="17801">Context 接口的四个方法中最常用的就是 Done 方法，它返回一个只读的 channel，用于接收取消信号。当 Context 取消的时候，会关闭这个只读 channel，也就等于发出了取消信号。</p>
<h3 data-nodeid="20505" class="">Context 树</h3>


<p data-nodeid="17804">我们不需要自己实现 Context 接口，Go 语言提供了函数可以帮助我们生成不同的 Context，通过这些函数可以生成一颗 Context 树，这样 Context 才可以关联起来，父 Context 发出取消信号的时候，子 Context 也会发出，这样就可以控制不同层级的协程退出。</p>
<p data-nodeid="17805">从使用功能上分，有四种实现好的 Context。</p>
<ol data-nodeid="17806">
<li data-nodeid="17807">
<p data-nodeid="17808"><strong data-nodeid="17956">空 Context</strong>：不可取消，没有截止时间，主要用于 Context 树的根节点。</p>
</li>
<li data-nodeid="17809">
<p data-nodeid="17810"><strong data-nodeid="17961">可取消的 Context</strong>：用于发出取消信号，当取消的时候，它的子 Context 也会取消。</p>
</li>
<li data-nodeid="17811">
<p data-nodeid="17812"><strong data-nodeid="17966">可定时取消的 Context</strong>：多了一个定时的功能。</p>
</li>
<li data-nodeid="17813">
<p data-nodeid="17814"><strong data-nodeid="17971">值 Context</strong>：用于存储一个 key-value 键值对。</p>
</li>
</ol>
<p data-nodeid="21711">从下图 Context 的衍生树可以看到，最顶部的是空 Context，它作为整棵 Context 树的根节点，在 Go 语言中，可以通过 context.Background() 获取一个根节点 Context。</p>
<p data-nodeid="21712" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/D3/CgqCHl_EyHOARbBqAAKzKmhclWo807.png" alt="Drawing 1.png" data-nodeid="21716"></p>



<div data-nodeid="22319" class=""><p style="text-align:center">（四种 Context 的衍生树）</p></div>

<p data-nodeid="17819">有了根节点 Context 后，这颗 Context 树要怎么生成呢？需要使用 Go 语言提供的四个函数。</p>
<ol data-nodeid="17820">
<li data-nodeid="17821">
<p data-nodeid="17822"><strong data-nodeid="17985">WithCancel(parent Context)</strong>：生成一个可取消的 Context。</p>
</li>
<li data-nodeid="17823">
<p data-nodeid="17824"><strong data-nodeid="17990">WithDeadline(parent Context, d time.Time)</strong>：生成一个可定时取消的 Context，参数 d 为定时取消的具体时间。</p>
</li>
<li data-nodeid="17825">
<p data-nodeid="17826"><strong data-nodeid="17995">WithTimeout(parent Context, timeout time.Duration)</strong>：生成一个可超时取消的 Context，参数 timeout 用于设置多久后取消</p>
</li>
<li data-nodeid="17827">
<p data-nodeid="17828"><strong data-nodeid="18000">WithValue(parent Context, key, val interface{})</strong>：生成一个可携带 key-value 键值对的 Context。</p>
</li>
</ol>
<p data-nodeid="17829">以上四个生成 Context 的函数中，前三个都属于可取消的 Context，它们是一类函数，最后一个是值 Context，用于存储一个 key-value 键值对。</p>
<h3 data-nodeid="22920" class="">使用 Context 取消多个协程</h3>

<p data-nodeid="17831">取消多个协程也比较简单，把 Context 作为参数传递给协程即可，还是以监控狗为例，如下所示：</p>
<p data-nodeid="17832"><em data-nodeid="18008"><strong data-nodeid="18007">ch10/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="17833"><code data-language="go">wg.Add(<span class="hljs-number">3</span>)
<span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">defer</span> wg.Done()
   watchDog(ctx,<span class="hljs-string">"【监控狗2】"</span>)
}()
<span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
   <span class="hljs-keyword">defer</span> wg.Done()
   watchDog(ctx,<span class="hljs-string">"【监控狗3】"</span>)
}()
</code></pre>
<p data-nodeid="17834">示例中增加了两个监控狗，也就是增加了两个协程，这样一个 Context 就同时控制了三个协程，一旦 Context 发出取消信号，这三个协程都会取消退出。</p>
<p data-nodeid="24704">以上示例中的 Context 没有子 Context，如果一个 Context 有子 Context，在该 Context 取消时会发生什么呢？下面通过一幅图说明：</p>
<p data-nodeid="24705" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/C7/Ciqc1F_EyIyAAO_TAADuPjzGt5U321.png" alt="Drawing 3.png" data-nodeid="24710"></p>
<div data-nodeid="24706"><p style="text-align:center">（Context 取消）</p></div>






<p data-nodeid="17839">可以看到，当节点 Ctx2 取消时，它的子节点 Ctx4、Ctx5 都会被取消，如果还有子节点的子节点，也会被取消。也就是说根节点为 Ctx2 的所有节点都会被取消，其他节点如 Ctx1、Ctx3 和 Ctx6 则不会。</p>
<h3 data-nodeid="25301" class="">Context 传值</h3>

<p data-nodeid="17841">Context 不仅可以取消，还可以传值，通过这个能力，可以把 Context 存储的值供其他协程使用。我通过下面的代码来说明：</p>
<p data-nodeid="17842"><em data-nodeid="18025"><strong data-nodeid="18024">ch10/main.go</strong></em></p>
<pre class="lang-go" data-nodeid="17843"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   wg.Add(<span class="hljs-number">4</span>) <span class="hljs-comment">//记得这里要改为4，原来是3，因为要多启动一个协程</span>
   
  <span class="hljs-comment">//省略其他无关代码</span>
   valCtx:=context.WithValue(ctx,<span class="hljs-string">"userId"</span>,<span class="hljs-number">2</span>)
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">defer</span> wg.Done()
      getUser(valCtx)
   }()
   <span class="hljs-comment">//省略其他无关代码</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getUser</span><span class="hljs-params">(ctx context.Context)</span></span>{
   <span class="hljs-keyword">for</span>  {
      <span class="hljs-keyword">select</span> {
      <span class="hljs-keyword">case</span> &lt;-ctx.Done():
         fmt.Println(<span class="hljs-string">"【获取用户】"</span>,<span class="hljs-string">"协程退出"</span>)
         <span class="hljs-keyword">return</span>
      <span class="hljs-keyword">default</span>:
         userId:=ctx.Value(<span class="hljs-string">"userId"</span>)
         fmt.Println(<span class="hljs-string">"【获取用户】"</span>,<span class="hljs-string">"用户ID为："</span>,userId)
         time.Sleep(<span class="hljs-number">1</span> * time.Second)
      }
   }
}
</code></pre>
<p data-nodeid="17844">这个示例是和上面的示例放在一起运行的，所以我省略了上面实例的重复代码。其中，通过 context.WithValue 函数存储一个 userId 为 2 的键值对，就可以在 getUser 函数中通过 ctx.Value("userId") 方法把对应的值取出来，达到传值的目的。</p>
<h3 data-nodeid="25893" class="">Context 使用原则</h3>

<p data-nodeid="17846">Context 是一种非常好的工具，使用它可以很方便地控制取消多个协程。在 Go 语言标准库中也使用了它们，比如 net/http 中使用 Context 取消网络的请求。</p>
<p data-nodeid="17847">要更好地使用 Context，有一些使用原则需要尽可能地遵守。</p>
<ol data-nodeid="17848">
<li data-nodeid="17849">
<p data-nodeid="17850">Context 不要放在结构体中，要以参数的方式传递。</p>
</li>
<li data-nodeid="17851">
<p data-nodeid="17852">Context 作为函数的参数时，要放在第一位，也就是第一个参数。</p>
</li>
<li data-nodeid="17853">
<p data-nodeid="17854">要使用 context.Background 函数生成根节点的 Context，也就是最顶层的 Context。</p>
</li>
<li data-nodeid="17855">
<p data-nodeid="17856">Context 传值要传递必须的值，而且要尽可能地少，不要什么都传。</p>
</li>
<li data-nodeid="17857">
<p data-nodeid="17858">Context 多协程安全，可以在多个协程中放心使用。</p>
</li>
</ol>
<p data-nodeid="17859">以上原则是规范类的，Go 语言的编译器并不会做这些检查，要靠自己遵守。</p>
<h3 data-nodeid="26485" class="">总结</h3>

<p data-nodeid="27659">Context 通过 With 系列函数生成 Context 树，把相关的 Context 关联起来，这样就可以统一进行控制。一声令下，关联的 Context 都会发出取消信号，使用这些 Context 的协程就可以收到取消信号，然后清理退出。你在定义函数的时候，如果想让外部给你的函数发取消信号，就可以为这个函数增加一个 Context 参数，让外部的调用者可以通过 Context 进行控制，比如下载一个文件超时退出的需求。</p>
<p data-nodeid="27660" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/72/C8/Ciqc1F_EyKiAUdQMAAVK80mD2bY940.png" alt="Drawing 4.png" data-nodeid="27664"></p>


<p data-nodeid="17863">这节课的最后留一个思考题给你：假如一个用户请求访问我们的网站，如何通过 Context 实现日志跟踪？先自己想想，下节课我会揭晓思路。</p>
<p data-nodeid="17864">下节课将学习“并发模式：Go 语言中即学即用的高效并发模式”，记得来听课！</p>

---

### 精选评论

##### **3439：
> 请教老师两个问题1.正常情况下，主协程退出后，子协程无法立即自动退出是吗？但是我理解应该是子协程执行完了它自己就会退出，只是不受主协程的影响，老师说不可预知的事情是否是指子协程可能永不结束，那样长期占用内存造成内存泄露，还有其他危险情况吗？2.使用context，是为了让子协程们受主协程的控制，如果主协程执行完毕，是否必须手动stop才能让各子协程退出？如果主协程执行完退出，是否会自动触发stop，各子协程也相继关闭呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 主线程退出，整个程序就死了。这节的本意是在主线程不退出，也就是程序正常的情况下，使用Context更好的控制子线程。

##### *斌：
> 老师，能不能说一下，context.TODO()和context.Background()在项目中应该怎么使用？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 优先使用 Background，不知道使用什么的话，再选择 TODO

##### *翔：
> 老师，我用context保存认证成功后当前请求登陆的用户信息，这个使用正确吗

##### *凯：
> 对于用户登陆的逻辑来说，是不是我可以在登陆的时候使用带过期时间的context，起一个携程专门监控用户登陆有效期，如果过期触发登出逻辑？另外，老师说的监控用户访问网站行为，是否可以在context 中以键值对形式存储用户访问当前页地址，key为用户id

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不要过渡使用Context，登录有效期的Token可以存放在其他地方，比如Redis。
网站访问的行为也是，可以使用Log打点，存在ELK等一些专门的日志处理软件中

