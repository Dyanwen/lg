<p data-nodeid="57629" class="">随着微服务架构的盛行，远程调用成了开发微服务必不可少的能力，RPC 框架作为微服务体系的底层支撑，也成了日常开发的必备工具。当下，RPC 框架已经不仅是进行远程调用的基础工具，还需要提供路由、服务发现、负载均衡、容错等能力。那么今天，我们就以“<strong data-nodeid="57692">怎样实现 RPC 框架</strong>”为引，从设计者角度看看如何设计一个 RPC 框架。</p>
<h3 data-nodeid="57630" class="">基础能力设计</h3>
<p data-nodeid="57631" class="">RPC（Remote Procedure Call）远程过程调用，顾名思义最基本的能力当然是远程调用一个过程。放到今天的面向对象的语言中，其实就是调用一个远程的方法。在远程我们必须先定义这个方法，然后才可以通过 RPC 框架调用该方法，远程调用不仅可以传参数、获取到返回值，还可以捕捉调用过程中的异常。RPC 让远程调用就像本地调用一样。</p>
<p data-nodeid="57632" class="">假设我们实现了一个<code data-backticks="1" data-nodeid="57696">rpc</code>对象，其中的<code data-backticks="1" data-nodeid="57698">invoke</code>方法可以实现远程调用。下面这段伪代码在调用远程的<code data-backticks="1" data-nodeid="57700">greetings</code>方法（RPC 调用），并向远程方法传递参数<code data-backticks="1" data-nodeid="57702">arg1``arg2</code>，然后再接收到远程的返回值。</p>
<pre class="lang-java" data-nodeid="57633"><code data-language="java"><span class="hljs-keyword">var</span> result = rpc.invoke(<span class="hljs-string">"greetings"</span>, arg1, arg2, ...)
</code></pre>
<p data-nodeid="58876">这段程序将本地看作 一个 RPC 的客户端，将远程看作一个 RPC 的服务端。如下图所示：</p>
<p data-nodeid="58877" class=""><img src="https://s0.lgstatic.com/i/image6/M00/40/28/Cgp9HWCjgYmAH8UHAAFonM-fpzM453.png" alt="Drawing 1.png" data-nodeid="58881"></p>



<p data-nodeid="57637" class="">服务 A 发起远程方法调用，RPC 客户端通过某种协议将请求发送给服务 B，服务 B 解析请求，进行本地方法的调用，将结果返回到服务 B 的 RPC 服务端，最终返回到服务 A。对服务 A 来说，调用的是一个函数，从接口到返回值的设计，和调用本地函数并没有太大的差别。当然，程序员不能完全忽略这是一次远程方法调用，因为远程调用的开销较大。如果程序员没有意识到调用远程方法有网络开销，就可能会写出下面这段程序：</p>
<pre class="lang-java" data-nodeid="57638"><code data-language="java"><span class="hljs-keyword">for</span>(<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">1000000</span>; i++) {
  rpc.invoke(...)
}
</code></pre>
<p data-nodeid="57639" class="">之所以写出上面的程序，是因为你（程序员）没有意识到 rpc.invoke 是一次远程调用。在实际的操作过程中，<code data-backticks="1" data-nodeid="57713">rpc.invoke</code>可能被封装到了某个业务方法中，程序员调用的时候便容易忽视这是一次远程操作。所以 RPC 调用时就要求你（程序员）对性能有清晰的认识。</p>
<h3 data-nodeid="57640">多路复用的优化</h3>
<p data-nodeid="57641" class=""><strong data-nodeid="57724">RPC 提供的是远程方法的调用，但本质上是数据的传递，传递数据有一个最基本的问题要处理，就是提升吞吐量（单位时间传递的数据量）</strong>。如果为每个远程调用（请求）建立一个连接，就会造成资源的浪费，因此通常我们会考虑多个请求复用一个连接，叫作<strong data-nodeid="57725">多路复用</strong>。</p>
<p data-nodeid="59566" class="">在具体实现多路复用的时候，也会有不同的策略。假设我们要发送数据 A、B、C、D，那么一种方式是建立一个连接，依次将 A、B、C、D 发过去，就像下图这样：</p>
<p data-nodeid="59567" class=""><img src="https://s0.lgstatic.com/i/image6/M00/40/31/CioPOWCjgaOAUlOMAABy4il5v8s683.png" alt="Drawing 3.png" data-nodeid="59571"></p>



<p data-nodeid="57645" class="">在这种结构中，利用一个连接顺序发送 A、B、C、D 将多个请求放入一个连接的方式，节省了多次握手、挥手的时间，但是由于 ABCD 不是真的并行发送，而是顺序发送，当其中某个请求的体积较大时，容易阻塞其他请求。比如下图这种情况：</p>
<p data-nodeid="59914" class=""><img src="https://s0.lgstatic.com/i/image6/M00/40/28/Cgp9HWCjgb-AbjPcAAB4uqNKiqI048.png" alt="Drawing 5.png" data-nodeid="59917"></p>


<p data-nodeid="57648" class="">在 A 较大的时候，B，C，D 就只能等 A 完全传送完成才能发生传送。这样的模型对于 RPC 请求／响应大小不平均的网络不太友好，体积小的请求／响应可能会因为一些大体积的请求／响应而延迟。</p>
<p data-nodeid="60570">因此还有另一种常见的多路复用方案，就是将 Ａ、Ｂ、Ｃ、Ｄ 切片一起传输，如下图所示：</p>
<p data-nodeid="60571" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/40/29/Cgp9HWCjgguARpfEAAENqpybwLU268.png" alt="Drawing 7.png" data-nodeid="60575"></p>



<p data-nodeid="57652">上图中，我们用不同颜色代表不同的传输任务。采用顺序传输方案将 A、B、C、Ｄ 用一个连接传输节省了握手、挥手的成本。切片传输的方案在这之上，将数据切片可以保证大、小任务并行，不会因为大任务阻塞小任务。</p>
<p data-nodeid="57653">另外还有一个需要考虑的点，是单个 TCP 连接的极限传输速度受到窗口大小、缓冲区等因素的制约，不一定可以用满网络资源。如果传输量特别大的时候，有可能需要考虑提供多个连接，每个连接再去考虑多路复用的情况。</p>
<h3 data-nodeid="57654">调用约定和命名</h3>
<p data-nodeid="57655">接下来，我们一起思考下服务的命名。<strong data-nodeid="57756">远程调用一个函数，命名空间＋类名＋方法名是一个比较好的选择，简而言之，每个可以远程调用的方法就是一个字符串</strong>。</p>
<p data-nodeid="57656">比如远程调用一个支付服务对象 PayService 的 pay 方法，命名空间可能是 trade.payment，对象名称是 PayService，方法名称是 pay。组合起来可以是一个完整的字符串，例如用 # 分割：<code data-backticks="1" data-nodeid="57758">trade.payment#PayService#pay</code>。</p>
<p data-nodeid="57657">在进行远程调用的时候，给远程方法命名是调用约定的一部分。我们通过调用命名空间下完整的名称调用远程方法。在面向对象的语言中，还有一种常见的做法是先不具体指定调用的方法，而是先创造一个远程对象的实例。比如上面例子中我们先通过 RPC 框架构造一个 PayService 对象的实例。这里会用到一些特别的编程技巧，比如代理设计模式、动态接口生成等。</p>
<p data-nodeid="57658">不过归根结底，我们调用的本质就是字符串名称。而实现这个调用，你需要知道两件事情：</p>
<ul data-nodeid="57659">
<li data-nodeid="57660">
<p data-nodeid="57661">IP 是多少，也就是方法在哪台机器上调用；</p>
</li>
<li data-nodeid="57662">
<p data-nodeid="57663">端口是多少，也就是哪个服务提供这个调用。</p>
</li>
</ul>
<h3 data-nodeid="57664">注册和发现</h3>
<p data-nodeid="57665">调用的时候，我们需要根据字符串（命名）去获取 IP 和端口（机器和服务）。</p>
<p data-nodeid="57666">机器可以是虚拟机、容器、实体机，也可以是某个拥有虚拟网卡的代理。在网络的世界中，需要的只是网络接口和 IP 地址。而操作系统区分应用需要的是端口。所以，在调用过程中，我们需要的是一个注册表，存储了字符串和 IP + 端口的对应关系。</p>
<p data-nodeid="57667">聪明的同学可能马上会想到，用 Redis 的<code data-backticks="1" data-nodeid="57768">hash</code>对象存储这个对应关系就很不错。当我们上线一个服务的时候，就在 Redis 的某个<code data-backticks="1" data-nodeid="57770">hash</code>对象中存储它和它对应的 IP 地址 + 端口列表。为什么是存一个列表？因为一个服务可能由多个机器提供。</p>
<p data-nodeid="57668">通常我们将写这个<code data-backticks="1" data-nodeid="57773">hash</code>对象的过程，也就是服务被记录的过程称作<strong data-nodeid="57783">注册</strong>。我们远程调用一个 RPC 服务的时候，调用端提供的是 RPC 服务的名称（例如：命名空间+对象+方法），根据名称查找到提供服务的 IP + 端口清单并指定某个 IP + 端口（提供服务）的过程称作<strong data-nodeid="57784">发现</strong>。</p>
<p data-nodeid="57669">当然，我们不能就这样简单理解成：注册就是写一个共享的哈希表，发现就是查哈希表再决定服务的响应者。在实际的设计中，要考虑的因素会更多。</p>
<p data-nodeid="57670">比如基于 Redis 的实现，如果所有 RPC 调用都需要去 Redis 查询，会造成负责发现的中间件压力较大。实际的操作过程中，往往会增加缓存。也就是 RPC 调用者会缓存上一次调用的 IP + 端口。但是这样设计，缓存又可能会和注册表之间产生数据不一致的问题。这个时候，可以考虑由分布式共识服务比如 ZooKeeper 提供订阅，让 RPC 调用者订阅到服务地址的变更，及时更新自己的缓存。</p>
<p data-nodeid="57671"><strong data-nodeid="57791">设计注册和发现两个功能的最大的价值是让客户端不再需要关注服务的部署细节，这样方便在全局动态调整服务的部署策略</strong>。</p>
<h4 data-nodeid="57672">负载均衡的设计</h4>
<p data-nodeid="57673">在设计 RPC 框架的时候，负载均衡器的设计往往需要和 RPC 框架一起考虑。因为 RPC 框架提供了注册、发现的能力，提供发现能力的模块本身就是一个负载均衡器。因此负载均衡可以看作发现模块的一个子组件。请求到达 RPC 的网关（或某个路由程序）后，发现组件会提供服务对应的所有实例（IP + 端口），然后负载均衡算法会指定其中一个响应这个请求。</p>
<h4 data-nodeid="57674">可用性和容灾</h4>
<p data-nodeid="57675">当一个服务实例崩溃的时候（不可用），因为有发现模块的存在，可以及时从注册表中删除这个服务实例。只要服务本身有足够多的实例，比如多个容器而且部署在不同的机器上，那么完全不可能用的风险会大大降低。当然，可用性是不可能 100% 实现的。</p>
<p data-nodeid="57676">另外，注册表和 RPC 调用者之间必然存在不一致现象，而且注册表的更新本身也可能滞后。比如确认一个服务有没有崩溃，可能需要一个心跳程序持续请求这个服务，因此 RPC 的调用者如果调用到一个不存在的服务，或者调用到一个发生崩溃的服务，需要自己重新去发现组件申请新的服务实例（地址 + 端口）。</p>
<p data-nodeid="57677">如果遇到临时访问量剧增，需要扩容的场景。这个时候只需要上线更多的容器，并且去注册即可。当然这要求部署模块和注册模块之间有较高的协同，这块可以用自动化脚本衔接。</p>
<h3 data-nodeid="57678">总结</h3>
<p data-nodeid="57679">总结下，<strong data-nodeid="57804">设计一个 RPC 框架最基础的能力就是实现远程方法的调用</strong>。这里需要一个调用约定，比如怎么描述一个远程的方法，发送端怎么传递参数，接收方如何解析参数？如果发生异常应该如何处理？具体来说，这些事情都不难实现，只是比较烦琐。其实不仅仅在 RPC 调用时有调用约定，编译器在实现函数调用的时候，也会有调用约定。另外，还有一些在 RPC 基础上建立起来的更复杂、更体系化的约定，比如说面向服务架构（SOA）。</p>
<p data-nodeid="57680">在实现了基本调用能力的基础上，接下来就是提供服务的注册、发现能力。有了这两个能力，就可以向客户端完全屏蔽服务的部署细节，并衍生出容灾、负载均衡的设计。</p>
<p data-nodeid="57681">当然，程序员还需要思考底层具体网络的传输问题。如果用 TCP 要思考多路复用以及连接数量的问题；如果是 UDP，需要增加对于可靠性保证的思考。如果使用了消息队列，还需要考虑服务的幂等性设计等。</p>
<h3 data-nodeid="57682">思考题</h3>
<p data-nodeid="57683">最后，再给你出一道需要查资料的思考题目：如何理解 Dubbo 的几个组成部分 Consumer、Provider、Monitor 和 Registry？</p>
<p data-nodeid="57684">这一讲就到这里，发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲是《模块三思考题解答》，希望你自己完成题目后再来看答案和分析。再见！</p>

---

### 精选评论

##### None：
> 写的真好。回忆起以前用过的rpc框架以及相关的配套设施，简直就是他们的设计概要，只是我以前没意识到

##### Joestar：
> “如果调用者没有意识到自己调用的是一个RPC方法，这有可能会造成一场灾难”赞

##### console_man：
> 真是有种醍醐灌顶的感觉

##### **棋：
> 写得真不戳

