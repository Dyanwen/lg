<p data-nodeid="340713" class="">在前面几节课我们介绍了 Netty 编解码的基础知识，想必你已经掌握了 Netty 实现编解码逻辑的技巧。那么接下来我们如何将编解码后的结果发送出去呢？在 Netty 中实现数据发送非常简单，只需要调用 writeAndFlush 方法即可，这么简单的一行代码究竟 Netty 帮我们完成了哪些事情呢？一起进入我们今天这节课要探讨的主题吧！</p>
<h3 data-nodeid="340714">Pipeline 事件传播回顾</h3>
<p data-nodeid="340715">在介绍 writeAndFlush 的工作原理之前，我们首先回顾下 Pipeline 的事件传播机制，因为他们是息息相关的。根据网络数据的流向，ChannelPipeline 分为入站 ChannelInboundHandler 和出站 ChannelOutboundHandler 两种处理器，如下图所示。</p>
<p data-nodeid="342334"><img src="https://s0.lgstatic.com/i/image/M00/6D/A3/Ciqc1F-uZy-Ae5ElAANEp703VGM268.png" alt="图片11.png" data-nodeid="342337"></p>





<p data-nodeid="340717">当我们从客户端向服务端发送请求，或者服务端向客户端响应请求结果都属于出站处理器 ChannelOutboundHandler 的行为，所以当我们调用 writeAndFlush 时，数据一定会在 Pipeline 中进行传播。</p>
<p data-nodeid="340718">在这里我首先抛出几个问题，学完本节课后可以用于检验下自己是否真的理解了 writeAndFlush 的原理。</p>
<ul data-nodeid="340719">
<li data-nodeid="340720">
<p data-nodeid="340721">writeAndFlush 是如何触发事件传播的？数据是怎样写到 Socket 底层的？</p>
</li>
<li data-nodeid="340722">
<p data-nodeid="340723">为什么会有 write 和 flush 两个动作？执行 flush 之前数据是如何存储的？</p>
</li>
<li data-nodeid="340724">
<p data-nodeid="340725">writeAndFlush 是同步还是异步？它是线程安全的吗？</p>
</li>
</ul>
<h3 data-nodeid="340726">writeAndFlush 事件传播分析</h3>
<p data-nodeid="340727">为了便于我们分析 writeAndFlush 的事件传播流程，首先我们通过代码模拟一个最简单的数据出站场景，服务端在接收到客户端的请求后，将响应结果编码后写回客户端。</p>
<p data-nodeid="340728">以下是服务端的启动类，分别注册了三个 ChannelHandler：<strong data-nodeid="340824">固定长度解码器 FixedLengthFrameDecoder</strong>、<strong data-nodeid="340825">响应结果编码器 ResponseSampleEncoder</strong>、<strong data-nodeid="340826">业务逻辑处理器 RequestSampleHandler</strong>。</p>
<pre class="lang-java" data-nodeid="340729"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">startEchoServer</span><span class="hljs-params">(<span class="hljs-keyword">int</span> port)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; EventLoopGroup bossGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();
&nbsp; &nbsp; &nbsp; &nbsp; EventLoopGroup workerGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ServerBootstrap b = <span class="hljs-keyword">new</span> ServerBootstrap();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; b.group(bossGroup, workerGroup)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .channel(NioServerSocketChannel.class)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> FixedLengthFrameDecoder(<span class="hljs-number">10</span>));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> ResponseSampleEncoder());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> RequestSampleHandler());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ChannelFuture f = b.bind(port).sync();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; f.channel().closeFuture().sync();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; bossGroup.shutdownGracefully();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; workerGroup.shutdownGracefully();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> EchoServer().startEchoServer(<span class="hljs-number">8088</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340730">其中固定长度解码器 FixedLengthFrameDecoder 是 Netty 自带的解码器，在这里就不做赘述了。下面我们分别看下另外两个 ChannelHandler 的具体实现。</p>
<p data-nodeid="340731">响应结果编码器 ResponseSampleEncoder 用于将服务端的处理结果进行编码，具体的实现逻辑如下：</p>
<pre class="lang-java" data-nodeid="340732"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ResponseSampleEncoder</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">MessageToByteEncoder</span>&lt;<span class="hljs-title">ResponseSample</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">encode</span><span class="hljs-params">(ChannelHandlerContext ctx, ResponseSample msg, ByteBuf out)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (msg != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.writeBytes(msg.getCode().getBytes());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.writeBytes(msg.getData().getBytes());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.writeLong(msg.getTimestamp());
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340733">RequestSampleHandler 主要负责客户端的数据处理，并通过调用 ctx.channel().writeAndFlush 向客户端返回 ResponseSample 对象，其中包含返回码、响应数据以及时间戳。</p>
<pre class="lang-java" data-nodeid="340734"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RequestSampleHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelInboundHandlerAdapter</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; String data = ((ByteBuf) msg).toString(CharsetUtil.UTF_8);
&nbsp; &nbsp; &nbsp; &nbsp; ResponseSample response = <span class="hljs-keyword">new</span> ResponseSample(<span class="hljs-string">"OK"</span>, data, System.currentTimeMillis());
&nbsp; &nbsp; &nbsp; &nbsp; ctx.channel().writeAndFlush(response);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340735">通过以上的代码示例我们可以描绘出 Pipeline 的链表结构，如下图所示。</p>
<p data-nodeid="343348"><img src="https://s0.lgstatic.com/i/image/M00/6D/AE/CgqCHl-uZz2AD58hAAHvDJLyzyU332.png" alt="图片12.png" data-nodeid="343351"></p>



<p data-nodeid="340737">那么当 RequestSampleHandler 调用 writeAndFlush 时，数据是如何在 Pipeline 中传播、处理并向客户端发送的呢？下面我们结合该场景对 writeAndFlush 的处理流程做深入的分析。</p>
<p data-nodeid="340738">既然 writeAndFlush 是特有的出站操作，那么我们猜测它是从 Pipeline 的 Tail 节点开始传播的，然后一直向前传播到 Head 节点。我们跟进去 ctx.channel().writeAndFlush 的源码，如下所示，发现 DefaultChannelPipeline 类中果然是调用的 Tail 节点 writeAndFlush 方法。</p>
<pre class="lang-java" data-nodeid="340739"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelFuture <span class="hljs-title">writeAndFlush</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> tail.writeAndFlush(msg);
}
</code></pre>
<p data-nodeid="340740">继续跟进 tail.writeAndFlush 的源码，最终会定位到 AbstractChannelHandlerContext 中的 write 方法。该方法是 writeAndFlush 的<strong data-nodeid="340841">核心逻辑</strong>，具体见以下源码。</p>
<pre class="lang-java" data-nodeid="340741"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(Object msg, <span class="hljs-keyword">boolean</span> flush, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// ...... 省略部分非核心代码 ......</span>

&nbsp; &nbsp; <span class="hljs-comment">// 找到 Pipeline 链表中下一个 Outbound 类型的 ChannelHandler 节点</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> AbstractChannelHandlerContext next = findContextOutbound(flush ?
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; (MASK_WRITE | MASK_FLUSH) : MASK_WRITE);
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Object m = pipeline.touch(msg, next);
&nbsp; &nbsp; EventExecutor executor = next.executor();
&nbsp; &nbsp; <span class="hljs-comment">// 判断当前线程是否是 NioEventLoop 中的线程</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor.inEventLoop()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (flush) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 因为 flush == true，所以流程走到这里</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; next.invokeWriteAndFlush(m, promise);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; next.invokeWrite(m, promise);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> AbstractWriteTask task;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (flush) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; task = WriteAndFlushTask.newInstance(next, m, promise);
&nbsp; &nbsp; &nbsp; &nbsp; }&nbsp; <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; task = WriteTask.newInstance(next, m, promise);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!safeExecute(executor, task, promise, m)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; task.cancel();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340742">首先我们确认下方法的入参，因为我们需要执行 flush 动作，所以 flush == true；write 方法还需要 ChannelPromise 参数，可见写操作是个异步的过程。AbstractChannelHandlerContext 会默认初始化一个 ChannelPromise 完成该异步操作，ChannelPromise 内部持有当前的 Channel 和 EventLoop，此外你可以向 ChannelPromise 中注册回调监听 listener 来获得异步操作的结果。</p>
<p data-nodeid="340743">write 方法的核心逻辑主要分为三个重要步骤，我已经以注释的形式在源码中标注出来。下面我们将结合上文中的 EchoServer 代码示例详细分析 write 方法的执行机制。</p>
<p data-nodeid="340744">第一步，调用 findContextOutbound 方法找到 Pipeline 链表中下一个 Outbound 类型的 ChannelHandler。在我们模拟的场景中下一个 Outbound 节点是 ResponseSampleEncoder。</p>
<p data-nodeid="340745">第二步，通过 inEventLoop 方法判断当前线程的身份标识，如果当前线程和 EventLoop 分配给当前 Channel 的线程是同一个线程的话，那么所提交的任务将被立即执行。否则当前的操作将被封装成一个 Task 放入到 EventLoop 的任务队列，稍后执行。所以 writeAndFlush 是否是线程安全的呢，你心里有答案了吗？</p>
<p data-nodeid="340746">第三步，因为 flush== true，将会直接执行 next.invokeWriteAndFlush(m, promise) 这行代码，我们跟进去源码。发现最终会它会执行下一个 ChannelHandler 节点的 write 方法，那么流程又回到了 到 AbstractChannelHandlerContext 中重复执行 write 方法，继续寻找下一个 Outbound 节点。</p>
<pre class="lang-java" data-nodeid="340747"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeWriteAndFlush</span><span class="hljs-params">(Object msg, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokeHandler()) {
&nbsp; &nbsp; &nbsp; &nbsp; invokeWrite0(msg, promise);
&nbsp; &nbsp; &nbsp; &nbsp; invokeFlush0();
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; writeAndFlush(msg, promise);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeWrite0</span><span class="hljs-params">(Object msg, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ((ChannelOutboundHandler) handler()).write(<span class="hljs-keyword">this</span>, msg, promise);
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; notifyOutboundHandlerException(t, promise);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340748">为什么 ResponseSampleEncoder 中重写的是 encode 方法，而不是 write 方法？encode 方法又是什么时机被执行的呢？这就回到了《Netty 如何实现自定义通信协议》课程中所介绍的 MessageToByteEncoder 源码。因为我们在实现编码器的时候都会继承 MessageToByteEncoder 抽象类，MessageToByteEncoder 重写了 ChanneOutboundHandler 的 write 方法，其中会调用子类实现的 encode 方法完成数据编码，在这里我们不再赘述。</p>
<p data-nodeid="340749">到目前为止，writeAndFlush 的事件传播流程已经分析完毕，可以看出 Netty 的 Pipeline 设计非常精妙，调用 writeAndFlush 时数据是在 Outbound 类型的 ChannelHandler 节点之间进行传播，那么最终数据是如何写到 Socket 底层的呢？我们一起继续向下分析吧。</p>
<h3 data-nodeid="340750">写 Buffer 队列</h3>
<p data-nodeid="340751">通过上述场景示例分析，我们知道数据将会在 Pipeline 中一直寻找 Outbound 节点并向前传播，直到 Head 节点结束，由 Head 节点完成最后的数据发送。所以 Pipeline 中的 Head 节点在完成 writeAndFlush 过程中扮演着重要的角色。我们直接看下 Head 节点的 write 方法源码：</p>
<pre class="lang-java" data-nodeid="340752"><code data-language="java"><span class="hljs-comment">// HeadContext # write</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; unsafe.write(msg, promise);
}
<span class="hljs-comment">// AbstractChannel # AbstractUnsafe # write</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(Object msg, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; assertEventLoop();
&nbsp; &nbsp; ChannelOutboundBuffer outboundBuffer = <span class="hljs-keyword">this</span>.outboundBuffer;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (outboundBuffer == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; safeSetFailure(promise, newClosedChannelException(initialCloseCause));
&nbsp; &nbsp; &nbsp; &nbsp; ReferenceCountUtil.release(msg);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">int</span> size;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; msg = filterOutboundMessage(msg); <span class="hljs-comment">// 过滤消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; size = pipeline.estimatorHandle().size(msg);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (size &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; size = <span class="hljs-number">0</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; safeSetFailure(promise, t);
&nbsp; &nbsp; &nbsp; &nbsp; ReferenceCountUtil.release(msg);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; outboundBuffer.addMessage(msg, size, promise); <span class="hljs-comment">// 向 Buffer 中添加数据</span>
}
</code></pre>
<p data-nodeid="340753">可以看出 Head 节点是通过调用 unsafe 对象完成数据写入的，unsafe 对应的是 NioSocketChannelUnsafe 对象实例，最终调用到 AbstractChannel 中的 write 方法，该方法有两个重要的点需要指出：</p>
<ol data-nodeid="340754">
<li data-nodeid="340755">
<p data-nodeid="340756">filterOutboundMessage 方法会对待写入的 msg 进行过滤，如果 msg 使用的不是 DirectByteBuf，那么它会将 msg 转换成 DirectByteBuf。</p>
</li>
<li data-nodeid="340757">
<p data-nodeid="340758">ChannelOutboundBuffer 可以理解为一个缓存结构，从源码最后一行 outboundBuffer.addMessage 可以看出是在向这个缓存中添加数据，所以 ChannelOutboundBuffer 才是理解数据发送的关键。</p>
</li>
</ol>
<p data-nodeid="340759">writeAndFlush 主要分为两个步骤，write 和 flush。通过上面的分析可以看出只调用 write 方法，数据并不会被真正发送出去，而是存储在 ChannelOutboundBuffer 的缓存内。下面我们重点分析一下 ChannelOutboundBuffer 的内部构造，跟进一下 addMessage 的源码：</p>
<pre class="lang-java" data-nodeid="340760"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addMessage</span><span class="hljs-params">(Object msg, <span class="hljs-keyword">int</span> size, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; Entry entry = Entry.newInstance(msg, size, total(msg), promise);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (tailEntry == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; flushedEntry = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Entry tail = tailEntry;
&nbsp; &nbsp; &nbsp; &nbsp; tail.next = entry;
&nbsp; &nbsp; }
&nbsp; &nbsp; tailEntry = entry;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (unflushedEntry == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; unflushedEntry = entry;
&nbsp; &nbsp; }
&nbsp; &nbsp; incrementPendingOutboundBytes(entry.pendingSize, <span class="hljs-keyword">false</span>);
}
</code></pre>
<p data-nodeid="340761">ChannelOutboundBuffer 缓存是一个链表结构，每次传入的数据都会被封装成一个 Entry 对象添加到链表中。ChannelOutboundBuffer 包含<strong data-nodeid="340871">三个非常重要的指针</strong>：第一个被写到缓冲区的<strong data-nodeid="340872">节点 flushedEntry</strong>、第一个未被写到缓冲区的<strong data-nodeid="340873">节点 unflushedEntry</strong>和最后一个<strong data-nodeid="340874">节点 tailEntry。</strong></p>
<p data-nodeid="340762">在初始状态下这三个指针都指向 NULL，当我们每次调用 write 方法是，都会调用 addMessage 方法改变这三个指针的指向，可以参考下图理解指针的移动过程会更加形象。</p>
<p data-nodeid="344362"><img src="https://s0.lgstatic.com/i/image/M00/6D/AE/CgqCHl-uZ1GADbu0AAMyHCydEjU371.png" alt="图片13.png" data-nodeid="344365"></p>



<p data-nodeid="340764">第一次调用 write，因为链表里只有一个数据，所以 unflushedEntry 和 tailEntry 指针都指向第一个添加的数据 msg1。flushedEntry 指针在没有触发 flush 动作时会一直指向 NULL。</p>
<p data-nodeid="340765">第二次调用 write，tailEntry 指针会指向新加入的 msg2，unflushedEntry 保持不变。</p>
<p data-nodeid="340766">第 N 次调用 write，tailEntry 指针会不断指向新加入的 msgN，unflushedEntry 依然保持不变，unflushedEntry 和 tailEntry 指针之间的数据都是未写入 Socket 缓冲区的。</p>
<p data-nodeid="340767">以上便是写 Buffer 队列写入数据的实现原理，但是我们不可能一直向缓存中写入数据，所以 addMessage 方法中每次写入数据后都会调用 incrementPendingOutboundBytes 方法判断缓存的水位线，具体源码如下。</p>
<pre class="lang-java" data-nodeid="340768"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> DEFAULT_LOW_WATER_MARK = <span class="hljs-number">32</span> * <span class="hljs-number">1024</span>;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> DEFAULT_HIGH_WATER_MARK = <span class="hljs-number">64</span> * <span class="hljs-number">1024</span>;
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">incrementPendingOutboundBytes</span><span class="hljs-params">(<span class="hljs-keyword">long</span> size, <span class="hljs-keyword">boolean</span> invokeLater)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (size == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-keyword">long</span> newWriteBufferSize = TOTAL_PENDING_SIZE_UPDATER.addAndGet(<span class="hljs-keyword">this</span>, size);
&nbsp; &nbsp; <span class="hljs-comment">// 判断缓存大小是否超过高水位线</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (newWriteBufferSize &gt; channel.config().getWriteBufferHighWaterMark()) {
&nbsp; &nbsp; &nbsp; &nbsp; setUnwritable(invokeLater);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340769">incrementPendingOutboundBytes 的逻辑非常简单，每次添加数据时都会累加数据的字节数，然后判断缓存大小是否超过所设置的高水位线 64KB，如果超过了高水位，那么 Channel 会被设置为不可写状态。直到缓存的数据大小低于低水位线 32KB 以后，Channel 才恢复成可写状态。</p>
<p data-nodeid="340770">有关写数据的逻辑已经分析完了，那么执行 flush 动作缓存又会是什么变化呢？我们接下来一起看下 flush 的工作原理吧。</p>
<h3 data-nodeid="340771">刷新 Buffer 队列</h3>
<p data-nodeid="340772">当执行完 write 写操作之后，invokeFlush0 会触发 flush 动作，与 write 方法类似，flush 方法同样会从 Tail 节点开始传播到 Head 节点，同样我们跟进下 HeadContext 的 flush 源码：</p>
<pre class="lang-java" data-nodeid="340773"><code data-language="java"><span class="hljs-comment">// HeadContext # flush</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flush</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> </span>{
&nbsp; &nbsp; unsafe.flush();
}
<span class="hljs-comment">// AbstractChannel # flush</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flush</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; assertEventLoop();
&nbsp; &nbsp; ChannelOutboundBuffer outboundBuffer = <span class="hljs-keyword">this</span>.outboundBuffer;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (outboundBuffer == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; outboundBuffer.addFlush();
&nbsp; &nbsp; flush0();
}
</code></pre>
<p data-nodeid="340774">可以看出 flush 的核心逻辑主要分为两个步骤：addFlush 和 flush0，下面我们逐一对它们进行分析。</p>
<p data-nodeid="340775">首先看下 addFlush 方法的源码：</p>
<pre class="lang-java" data-nodeid="340776"><code data-language="java"><span class="hljs-comment">// ChannelOutboundBuffer # addFlush</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addFlush</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; Entry entry = unflushedEntry;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (entry != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (flushedEntry == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; flushedEntry = entry;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; flushed ++;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!entry.promise.setUncancellable()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> pending = entry.cancel();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 减去待发送的数据，如果总字节数低于低水位，那么 Channel 将变为可写状态</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; decrementPendingOutboundBytes(pending, <span class="hljs-keyword">false</span>, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; entry = entry.next;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">while</span> (entry != <span class="hljs-keyword">null</span>);
&nbsp; &nbsp; &nbsp; &nbsp; unflushedEntry = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="340777">addFlush 方法同样也会操作 ChannelOutboundBuffer 缓存数据。在执行 addFlush 方法时，缓存中的指针变化又是如何呢？如下图所示，我们在写入流程的基础上继续进行分析。</p>
<p data-nodeid="345376"><img src="https://s0.lgstatic.com/i/image/M00/6D/AE/CgqCHl-uZ2CAFvXuAAJkYjAgb8A346.png" alt="图片14.png" data-nodeid="345379"></p>



<p data-nodeid="340779">此时 flushedEntry 指针有所改变，变更为 unflushedEntry 指针所指向的数据，然后 unflushedEntry 指针指向 NULL，flushedEntry 指针指向的数据才会被真正发送到 Socket 缓冲区。</p>
<p data-nodeid="340780">在 addFlush 源码中 decrementPendingOutboundBytes 与之前 addMessage 源码中的 incrementPendingOutboundBytes 是相对应的。decrementPendingOutboundBytes 主要作用是减去待发送的数据字节，如果缓存的大小已经小于低水位，那么 Channel 会恢复为可写状态。</p>
<p data-nodeid="340781">addFlush 的大体流程我们已经介绍完毕，接下来便是第二步负责发送数据的 flush0 方法。同样我们跟进 flush0 的源码，定位出 flush0 的核心调用链路：</p>
<pre class="lang-java" data-nodeid="340782"><code data-language="java"><span class="hljs-comment">// AbstractNioUnsafe # flush0</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flush0</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!isFlushPending()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>.flush0();
&nbsp; &nbsp; }
}
<span class="hljs-comment">// AbstractNioByteChannel # doWrite</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doWrite</span><span class="hljs-params">(ChannelOutboundBuffer in)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> writeSpinCount = config().getWriteSpinCount();
&nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Object msg = in.current();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (msg == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; clearOpWrite();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; writeSpinCount -= doWriteInternal(in, msg);
&nbsp; &nbsp; } <span class="hljs-keyword">while</span> (writeSpinCount &gt; <span class="hljs-number">0</span>);
&nbsp; &nbsp; incompleteWrite(writeSpinCount &lt; <span class="hljs-number">0</span>);
}
</code></pre>
<p data-nodeid="340783">实际 flush0 的调用层次很深，但其实核心的逻辑在于 AbstractNioByteChannel 的 doWrite 方法，该方法负责将数据真正写入到 Socket 缓冲区。doWrite 方法的处理流程主要分为三步：</p>
<p data-nodeid="340784">第一，根据配置获取自旋锁的次数 writeSpinCount。那么你的疑问就来了，这个自旋锁的次数主要是用来干什么的呢？当我们向 Socket 底层写数据的时候，如果每次要写入的数据量很大，是不可能一次将数据写完的，所以只能分批写入。Netty 在不断调用执行写入逻辑的时候，EventLoop 线程可能一直在等待，这样有可能会阻塞其他事件处理。所以这里自旋锁的次数相当于控制一次写入数据的最大的循环执行次数，如果超过所设置的自旋锁次数，那么写操作将会被暂时中断。</p>
<p data-nodeid="340785">第二，根据自旋锁次数重复调用 doWriteInternal 方法发送数据，每成功发送一次数据，自旋锁的次数 writeSpinCount 减 1，当 writeSpinCount 耗尽，那么 doWrite 操作将会被暂时中断。doWriteInternal 的源码涉及 JDK NIO 底层，在这里我们不再深入展开，它的主要作用在于删除缓存中的链表节点以及调用底层 API 发送数据，有兴趣的同学可以自行研究。</p>
<p data-nodeid="340786">第三，调用 incompleteWrite 方法确保数据能够全部发送出去，因为自旋锁次数的限制，可能数据并没有写完，所以需要继续 OP_WRITE 事件；如果数据已经写完，清除 OP_WRITE 事件即可。</p>
<p data-nodeid="346176">至此，整个 writeAndFlush 的工作原理已经全部分析完了，整个过程的调用层次比较深，我整理了 writeAndFlush 的时序图，如下所示，帮助大家梳理 writeAndFlush 的调用流程，加深对上述知识点的理解。</p>
<p data-nodeid="346177" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/6D/A3/Ciqc1F-uZ4iAYZDxAAROuJN6ruk510.png" alt="图片15.png" data-nodeid="346181"></p>



<h3 data-nodeid="340788">总结</h3>
<p data-nodeid="340789">本节课我们深入分析了 writeAndFlush 的处理流程，可以总结以下三点：</p>
<ul data-nodeid="340790">
<li data-nodeid="340791">
<p data-nodeid="340792" class="">writeAndFlush 属于出站操作，它是从 Pipeline 的 Tail 节点开始进行事件传播，一直向前传播到 Head 节点。不管在 write 还是 flush 过程，Head 节点都中扮演着重要的角色。</p>
</li>
<li data-nodeid="340793">
<p data-nodeid="340794">write 方法并没有将数据写入 Socket 缓冲区，只是将数据写入到 ChannelOutboundBuffer 缓存中，ChannelOutboundBuffer 缓存内部是由单向链表实现的。</p>
</li>
<li data-nodeid="340795">
<p data-nodeid="340796">flush 方法才最终将数据写入到 Socket 缓冲区。</p>
</li>
</ul>
<p data-nodeid="340797" class="">最后，留一个小的思考题，Channel 和 ChannelHandlerContext 都有 writeAndFlush 方法，它们之间有什么区别呢？</p>

---

### 精选评论

##### Q：
> 思考题之前已经介绍过了，channel是从tail开始处理直到head，而handlercontext则是从当前包装的outbound开始处理直到head，相比channel可以少执行一些outbound的操作

##### *镇：
> 老师 ，自旋锁那块的逻辑是不是可以这样理解：加入调用了一次doWrite()方法，要发送一个大数据，假设要分8次才能写入完成，而此时获取的自旋锁次数是7，在发送时，每写入成功一批数据NioEventLoop线程的自旋锁次数都减一，那么就会剩余一批数据没有写入成功，这时是要交给inCompleteWrite()方法去处理，是这样的吗

##### peng：
> 用channel写的话，数据会从Tail开始往Head传播(当然是在outbount上)，如果是用ChannelHandlerContext的话，会从当前节点往Head传播（不会通过Tail了）

##### **丁：
> 老师如果超过水位了。这个channel不能写了，会影响挂在这个EventLoop的其他channel吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会影响其他的 Channel

##### *涛：
> 很细，对netty数据处理流程有了更深的理解

##### **5968：
> 有没有讲解Netty怎么实现一个高并发的连接池的呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 专栏里没有哦，学完整个专栏有兴趣可以自己实现哈。

##### **嵩：
> 提个小建议，老是可以代码放到github吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实战课代码我会放到github上

##### **杰：
> 很赞

##### **鹏：
> 彩！

##### 无：
> 2020-11-17 打卡

##### **年：
> 写的很好，很好啊

##### **升：
> 打卡

