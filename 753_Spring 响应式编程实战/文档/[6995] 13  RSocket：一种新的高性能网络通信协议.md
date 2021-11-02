<p data-nodeid="484" class="">前面几讲我们讨论了如何使用 WebFlux 构建响应式 Web 服务的实现方案。WebFlux 和 WebMVC 一样，都是基于 HTTP 协议实现请求-响应式的交互方式。这种交互方案很简单，但不够灵活，也无法应对所有的响应式应用场景。那么，有没有在网络协议层上提供更加丰富的交互方式呢？答案是肯定的，那就是我们今天要讨论的 RSocket 协议。</p>
<p data-nodeid="485">这一讲，我将从 RSocket 协议的特性、交互模式以及与主流开发框架之间的集成等几个方面来和你讨论，相信学完之后，你就会发现 <strong data-nodeid="562">RSocket 是一款全新的协议，它基于响应式数据流，为我们提供了高性能的网络通信机制</strong>。</p>
<h3 data-nodeid="486">RSocket 协议</h3>
<p data-nodeid="487">在引入 RSocket 协议之前，我们先来讨论为什么需要这样一个协议。关于它的背景，让我从传统的请求-响应模式所存在的问题开始说起。</p>
<h4 data-nodeid="488">请求-响应模式的问题</h4>
<p data-nodeid="489">我们知道常用的 HTTP 协议的优势在于其广泛的适用性，有非常多的服务器和客户端实现工具的支持，但 HTTP 协议本身比较简单，只支持请求-响应模式。而这种模式对于很多应用场景来说是不合适的。</p>
<p data-nodeid="490">典型的例子就是消息推送，以 HTTP 协议为例，如果客户端需要获取最新的推送消息，就必须使用轮询。客户端不停地发送请求到服务器来检查更新，这无疑造成了大量的资源浪费。请求-响应模式的另外一个问题是，如果某个请求的响应时间过长，会阻塞之后的其他请求的处理，正如“09 | 框架升级：WebFlux 比 Web MVC 到底好在哪里”中所分析的那样。</p>
<p data-nodeid="491">虽然服务器发送事件（Server-Sent Events，SSE）可以用来推送消息，不过，SSE 是一个简单的文本协议，仅提供有限的功能。此外，WebSocket 可以进行双向数据传输，但长连接会造成服务之间的紧密耦合，WebSocket 的使用就不符合响应式系统要求，因为协议不提供控制背压的可能性，而背压是回弹性系统的重要组成部分。</p>
<p data-nodeid="492">事实上，响应式编程的实施目前主要有两个障碍，一个是关系型数据访问，我们将在“17 | R2DBC：关系型数据库能具备响应式数据访问特性吗”中专门讨论这个话题；而另一个就是网络协议。幸运的是，响应式流规范背后的开发团队理解了跨网络、异步、低延迟通信的必要性。2015 年，RSocket 协议就在这样的背景下诞生了。</p>
<h4 data-nodeid="493">RSocket 协议与交互模式</h4>
<p data-nodeid="494">RSocket 是一种新的第 7 层语言无关的应用网络协议，用来解决单一的请求-响应模式以及现有网络传输协议所存在的问题，提供 Java、JavaScript、C++ 和 Kotlin 等多种语言的实现版本。</p>
<p data-nodeid="495">RSocket 是一个二进制的协议，以异步消息的方式提供 4 种交互模式，除了请求-响应（request/response）模式之外，还包括请求-响应流（request/stream）、即发-即忘（fire-and-forget）和通道（channel）这三种新的交互模式。这些模式的基本特性如下所示。</p>
<ul data-nodeid="496">
<li data-nodeid="497">
<p data-nodeid="498">请求-响应模式：这是最典型也最常见的模式。发送方在发送消息给接收方之后，等待与之对应的响应消息。</p>
</li>
<li data-nodeid="499">
<p data-nodeid="500">请求-响应流模式：发送方的每个请求消息，都对应于接收方的一个消息流作为响应。</p>
</li>
<li data-nodeid="501">
<p data-nodeid="502">即发-即忘模式：发送方的请求消息没有与之对应的响应。</p>
</li>
<li data-nodeid="503">
<p data-nodeid="504">通道模式：在发送方和接收方之间建立一个双向传输的通道。</p>
</li>
</ul>
<p data-nodeid="505">RSocket 专门设计用来与响应式风格应用程序进行配合使用，在使用 RSocket 协议时，背压和流量控制仍然有效。</p>
<p data-nodeid="506">为了更好地理解 RSocket 协议，让我们将它与 HTTP 协议做一些对比。在第 9 讲中，我已经提到过 Servlet 是基于 HTTP 协议之上的一套 Java API 规范，将 HTTP 请求转化为一个 ServletRequest 对象，并将处理结果封装成一个 ServletResponse 对象进行返回。HTTP 协议为了兼容各种应用方式，本身有一定的复杂性，性能一般。而 RSocket 采用的是自定义二进制协议，其本身的定位就是高性能通信协议，性能上比 HTTP 高出一个数量级。</p>
<p data-nodeid="957">在交互模式上，与 HTTP 的请求-响应这种单向的交互模式不同，RSocket 倡导的是对等通信，不再使用传统的客户端-服务器端单向通信模式，而是在两端之间可以自由地相互发送和处理请求。RSocket 协议在交互方式上可以参考下图。</p>
<p data-nodeid="958" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/37/06/CioPOWB1XhiAZv3EAACVQCujHBw318.png" alt="图片1.png" data-nodeid="963"></p>
<div data-nodeid="959"><p style="text-align:center">RSocket 协议的交互方式</p></div>




<h3 data-nodeid="510">使用 RSocket 实现远程交互</h3>
<p data-nodeid="511">想要在应用程序中使用 RSocket 协议，我们需要引入如下依赖。</p>
<pre class="lang-java" data-nodeid="512"><code data-language="java">&lt;dependency&gt;
&nbsp;&nbsp;&nbsp; &lt;groupId&gt;io.rsocket&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp; &lt;artifactId&gt;rsocket-core&lt;/artifactId&gt;
&lt;/dependency&gt;
&lt;dependency&gt;
&nbsp;&nbsp;&nbsp; &lt;groupId&gt;io.rsocket&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp; &lt;artifactId&gt;rsocket-transport-netty&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="513">可以看到，这里使用了 rsocket-transport-netty 包，该包的底层实现就是 Reactor Netty 组件，支持 TCP 和 WebSocket 协议。如果你想使用 UDP 协议，那么可以引入 rsocket-transport-aeron 包。在引入这些包之后，就可以使用该协议中最核心的 RSocket 接口了，我们一起来看一下。</p>
<h4 data-nodeid="514">RSocket 接口</h4>
<p data-nodeid="515">RSocket 接口的定义如下所示。</p>
<pre class="lang-java" data-nodeid="516"><code data-language="java"><span class="hljs-keyword">import</span> org.reactivestreams.Publisher;
<span class="hljs-keyword">import</span> reactor.core.publisher.Flux;
<span class="hljs-keyword">import</span> reactor.core.publisher.Mono;
&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">RSocket</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Availability</span>, <span class="hljs-title">Closeable</span> </span>{

	<span class="hljs-comment">//推送元信息，数据可以自定义</span>
	<span class="hljs-function">Mono&lt;Void&gt; <span class="hljs-title">metadataPush</span><span class="hljs-params">(Payload payload)</span></span>;
	&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//请求-响应模式，发送一个请求并接收一个响应。该协议也比 HTTP 更具优势，因为它是异步且多路复用的</span>
	<span class="hljs-function">Mono&lt;Payload&gt; <span class="hljs-title">requestResponse</span><span class="hljs-params">(Payload payload)</span></span>;
	&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//即发-即忘模式，请求-响应的优化，在不需要响应时非常有用</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Mono&lt;Void&gt; <span class="hljs-title">fireAndForget</span><span class="hljs-params">(Payload payload)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//请求-响应流模式，类似返回集合的请求/响应，集合将以流的方式返回，而不是等到查询完成</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Flux&lt;Payload&gt; <span class="hljs-title">requestStream</span><span class="hljs-params">(Payload payload)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//通道模式，允许任意交互模型的双向消息流</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Flux&lt;Payload&gt; <span class="hljs-title">requestChannel</span><span class="hljs-params">(Publisher&lt;Payload&gt; payloads)</span></span>;
}
</code></pre>
<p data-nodeid="517">显然，RSocket 接口通过四个方法分别实现了它所提供的四种交互模式，其中 requestResponse 方法返回的是一个 Mono<code data-backticks="1" data-nodeid="595">&lt;Payload&gt;</code> 对象，这里的 Payload 代表的就是一种消息对象，它由两部分组成：元信息 metadata 和数据 data，类似常见的消息通信中的消息头和消息体的概念。</p>
<p data-nodeid="518">然后，我们发现 fireAndForget 方法返回的是一个 Mono<code data-backticks="1" data-nodeid="598">&lt;Void&gt;</code> 流，符合即发-即忘模式的语义。而 requestStream 作为请求-响应流模式的实现，与 requestResponse 的区别在于它的返回值是一个 Flux 流，而不是一个 Mono 对象。</p>
<p data-nodeid="519">最后，我们注意到这几个方法的输入都是一个 Payload 消息对象，而不是一个响应式流对象。但 requestChannel 方法就不一样了，它的输入同样是一个代表响应式流的 Publisher 对象，这意味着此种模式下的输入输出都是响应式流，也就是说可以进行客户端和服务器端之间的双向交互。</p>
<p data-nodeid="520">rsocket-core 包针对 RSocket 接口提供了一个抽象的实现类 AbstractRSocket，对上述方法做了简单的实现封装。在使用过程中，我们可以基于这个 AbstractRSocket 类来提供某一个交互模式的具体实现逻辑，而不需要完全实现 RSocket 接口中的所有方法。</p>
<h4 data-nodeid="521">使用 RSocket 的交互模式</h4>
<p data-nodeid="522">介绍完 RSocket 接口之后，我们来看看具体如何使用它所提供的四种交互模式。这里以最常见的请求-响应交互模式为例，给出使用 RSocket 协议的使用方法。与使用 HTTP 协议一样，这个过程需要构建服务器端和客户端，并通过客户端发起请求。</p>
<p data-nodeid="523">我们先来看如何构建 RSocket 服务器端，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="524"><code data-language="java">RSocketFactory.receive()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .acceptor(((setup, sendingSocket) -&gt; Mono.just(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> AbstractRSocket() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Payload&gt; <span class="hljs-title">requestResponse</span><span class="hljs-params">(Payload payload)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(DefaultPayload.create(<span class="hljs-string">"Hello: "</span> + payload.getDataUtf8()));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .transport(TcpServerTransport.create(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">7000</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .start()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .subscribe();
</code></pre>
<p data-nodeid="525">这里的 RSocketFactory.receive() 方法返回用来创建服务器的 ServerRSocketFactory 类的对象。ServerRSocketFactory 的 acceptor() 方法的输入参数是 SocketAcceptor 接口。</p>
<p data-nodeid="526">上述代码中，我们用到了前面介绍的 RSocket 抽象实现类 AbstractRSocket，重写了其中的 requestResponse() 方法，对输入的参数前面添加一个 "Hello: " 前缀并返回；接下来的 transport() 方法指定 ServerTransport 接口的实现类 TcpServerTransport 作为 RSocket 底层的传输层实现，通过该方法，服务器端就启动了本地 7000 端口并监听来自客户端的请求；最后，我们通过 start().subscribe() 来触发整个启动过程。</p>
<p data-nodeid="527">构建完服务器端，我们来构建客户端组件，如下所示。</p>
<pre class="lang-java" data-nodeid="528"><code data-language="java">RSocket socket = RSocketFactory.connect()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .transport(TcpClientTransport.create(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">7000</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .start()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .block();
</code></pre>
<p data-nodeid="529">RSocketFactory.connect() 方法用来创建 RSocket 客户端，返回 ClientRSocketFactory 类的实例对象；接下来的 transport() 方法指定传输层 ClientTransport 实现；和服务器端组件 TcpServerTransport 对应，这里使用的是 TcpClientTransport 来连接本地服务器上的 7000 端口；最后调用 start().block() 方法等待客户端启动并返回 RSocket 对象。</p>
<p data-nodeid="530">现在，我们就可以使用 RSocket 的 requestResponse() 方法来发送请求并获取响应了，如下所示。</p>
<pre class="lang-java" data-nodeid="531"><code data-language="java">socket.requestResponse(DefaultPayload.create(<span class="hljs-string">"World"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(Payload::getDataUtf8)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .doOnNext(System.out::println)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .doFinally(signalType -&gt; socket.dispose())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .then()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .block();
</code></pre>
<p data-nodeid="532">我们可以使用 DefaultPayload.create() 方法来简单地创建 Payload 对象，然后通过 RSocket 类的 dispose() 方法用来销毁客户端对象。这样，整个调用过程就结束了。执行这次请求，我们会在控制台上获取“Hello: World”。</p>
<h3 data-nodeid="533">RSocket 与框架集成</h3>
<p data-nodeid="534">通常，我们不会直接使用 RSocket 原生开发库进行应用程序的开发，而是借助特定的开发框架。在 Java 领域中，Spring Boot、Spring Cloud 以及 Dubbo 等主流开发框架都集成了 RSocket 协议。下面我就分别为你说明。</p>
<h4 data-nodeid="535">集成 RSocket 与 Spring 框架</h4>
<p data-nodeid="536">想要在 Spring Boot 中使用 RSocket 协议，我们需要引入如下依赖。</p>
<pre class="lang-java" data-nodeid="537"><code data-language="java">&lt;dependency&gt;
&nbsp;&nbsp;&nbsp; &lt;groupId&gt;org.springframework.boot&lt;/groupId&gt;
&nbsp;&nbsp;&nbsp; &lt;artifactId&gt;spring-boot-starter-rsocket&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="538">然后，我们同样先来构建一个请求-响应式交互方式。基于“10 | WebFlux（上）：如何使用注解编程模式构建异步非阻塞服务”中的内容，我们可以构建如下所示一个简单 Controller。</p>
<pre class="lang-java" data-nodeid="539"><code data-language="java"><span class="hljs-meta">@Controller</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloController</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@MessageMapping("hello")</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;String&gt; <span class="hljs-title">hello</span><span class="hljs-params">(String input)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Mono.just(<span class="hljs-string">"Hello: "</span> + input);
&nbsp; }
}
</code></pre>
<p data-nodeid="540">你可以注意到，这里我引入了一个新的注解 @MessageMapping。跟 @RequestMapping 注解类似，@MessageMapping 是 Spring 提供的一个注解，用来指定 WebSocket、RSocket 等协议中消息处理的目的地。然后，我们输入了一个 String 类型的参数并返回一个 Mono 对象，符合请求-响应交互模式的定义。</p>
<p data-nodeid="541">为了访问这个 RSocket 端点，我们需要构建一个 RSocketRequester 对象，构建方式如下所示。</p>
<pre class="lang-java" data-nodeid="542"><code data-language="java"><span class="hljs-meta">@Autowired</span>
RSocketRequester.Builder builder;
&nbsp;
RSocketRequester requester = builder.dataMimeType(MimeTypeUtils.TEXT_PLAIN)
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .connect(TcpClientTransport.create(<span class="hljs-number">7000</span>)).block();
</code></pre>
<p data-nodeid="543">基于这个 RSocketRequester 对象，我们就可以通过它的 route 方法路由到前面通过 @MessageMapping 注解构建的 "hello" 端点，如下所示。</p>
<pre class="lang-java" data-nodeid="544"><code data-language="java">Mono&lt;String&gt; response = requester.route(<span class="hljs-string">"hello"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .data(<span class="hljs-string">"World"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .retrieveMono(String.class);
</code></pre>
<p data-nodeid="545">我们再来看一个请求-响应流的示例，如下所示。</p>
<pre class="lang-java" data-nodeid="546"><code data-language="java"><span class="hljs-meta">@MessageMapping("stream")</span>
<span class="hljs-function">Flux&lt;Message&gt; <span class="hljs-title">stream</span><span class="hljs-params">(Message request)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Flux
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .interval(Duration.ofSeconds(<span class="hljs-number">1</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(index -&gt; <span class="hljs-keyword">new</span> Message(request.getParam, index));
}
</code></pre>
<p data-nodeid="547">这里根据输入的 Message 对象，返回一个 Flux 流，每一秒发送一个添加了 Index 的新 Message 对象。</p>
<h4 data-nodeid="548">集成 RSocket 与其他框架</h4>
<p data-nodeid="549">针对其他开发框架，Dubbo 在 3.0.0-SNAPSHOT 版本里基于 RSocket 对响应式编程提供了支持，开发人员可以非常方便地使用 RSocket 的 API。而随着 Spring 框架的持续升级，5.2 版本中也把 RSocket 作为缺省的通信协议。</p>
<h3 data-nodeid="550">小结与预告</h3>
<p data-nodeid="551">作为构建响应式 Web 服务的最后一环，本讲讨论了 RSocket 这款新的高性能网络通信协议。与 HTTP 协议相比，RSocket 提供了四种不同的交互模式来提供多样化的网络通信体验。同时，RSocket 也无缝集成了响应式流。我们可以通过 Spring Boot 框架来使用这款异步、非阻塞式通信协议。</p>
<p data-nodeid="552">这里给你留一道思考题：你知道 RSocket 提供了哪四种交互模式，各自与响应式流是怎么整合的？</p>
<p data-nodeid="553">下一讲就要开始讨论响应式数据访问层的构建过程，Spring 家族专门提供了 Spring Boot 框架来实现这一目标，我们下一讲再见。</p>
<blockquote data-nodeid="554">
<p data-nodeid="555" class="">点击链接，获取课程相关代码 ↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="643">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### *瑾：
> 老师，我最近一直在研究netty，project reactor core，web flux，flink一些基于异步流式处理框架。想请教下老师有推荐的后端响应式编程的中文书籍吗？考虑到最终使用到真实项目中落地，最好能有进阶实战类的书籍推荐下。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 《Spring响应式编程(图灵出品) 》，可以看看这本

##### **6881：
> 页面上怎么使用rsocket呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 页面使用RSocket的方式跟使用SSE的差不多，就是两端建立一个连接，然后服务端就可以向页面推送数据流

##### *瑞：
> RSocket在boot中使用会和mvc冲突吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的，两个处理模型

##### **伟：
> 1

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 1

