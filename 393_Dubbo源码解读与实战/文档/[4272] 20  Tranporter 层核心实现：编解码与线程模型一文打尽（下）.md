<p data-nodeid="178722" class="">在上一课时中，我们深入分析了 Transporter 层中 Server 相关的核心抽象类以及基于 Netty 4 的实现类。本课时我们继续分析 Transporter 层中剩余的核心接口实现，主要涉及 Client 接口、Channel 接口、ChannelHandler 接口，以及相关的关键组件。</p>
<h3 data-nodeid="178723">Client 继承路线分析</h3>
<p data-nodeid="178724">在上一课时分析 AbstractEndpoint 的时候可以看到，除了 AbstractServer 这一条继承线之外，还有 AbstractClient 这条继承线，它是对客户端的抽象。AbstractClient 中的核心字段有如下几个。</p>
<ul data-nodeid="178725">
<li data-nodeid="178726">
<p data-nodeid="178727">connectLock（Lock 类型）：在 Client 底层进行连接、断开、重连等操作时，需要获取该锁进行同步。</p>
</li>
<li data-nodeid="178728">
<p data-nodeid="178729">needReconnect（Boolean 类型）：在发送数据之前，会检查 Client 底层的连接是否断开，如果断开了，则会根据 needReconnect 字段，决定是否重连。</p>
</li>
<li data-nodeid="178730">
<p data-nodeid="178731">executor（ExecutorService 类型）：当前 Client 关联的线程池，线程池的具体内容在上一课时已经详细介绍过了，这里不再赘述。</p>
</li>
</ul>
<p data-nodeid="178732">在 AbstractClient 的构造方法中，会解析 URL 初始化 needReconnect 字段和 executor字段，如下示例代码：</p>
<pre class="lang-java" data-nodeid="178733"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AbstractClient</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(url, handler); <span class="hljs-comment">// 调用父类的构造方法</span>
&nbsp; &nbsp; <span class="hljs-comment">// 解析URL，初始化needReconnect值</span>
&nbsp; &nbsp; needReconnect = url.getParameter(<span class="hljs-string">"send.reconnect"</span>, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; initExecutor(url); &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 解析URL，初始化executor</span>
&nbsp; &nbsp; doOpen(); &nbsp; &nbsp;<span class="hljs-comment">// 初始化底层的NIO库的相关组件</span>
&nbsp; &nbsp; <span class="hljs-comment">// 创建底层连接</span>
&nbsp; &nbsp; connect(); <span class="hljs-comment">// 省略异常处理的逻辑</span>
}
</code></pre>
<p data-nodeid="178734">与 AbstractServer 类似，AbstractClient 定义了 doOpen()、doClose()、doConnect()和doDisConnect() 四个抽象方法给子类实现。</p>
<p data-nodeid="178735">下面来看基于 Netty 4 实现的 NettyClient，它继承了 AbstractClient 抽象类，实现了上述四个 do*() 抽象方法，我们这里重点关注 doOpen() 方法和 doConnect() 方法。在 NettyClient 的 doOpen() 方法中会通过 Bootstrap 构建客户端，其中会完成连接超时时间、keepalive 等参数的设置，以及 ChannelHandler 的创建和注册，具体实现如下所示：</p>
<pre class="lang-java" data-nodeid="178736"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doOpen</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 创建NettyClientHandler</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> NettyClientHandler nettyClientHandler = <span class="hljs-keyword">new</span> NettyClientHandler(getUrl(), <span class="hljs-keyword">this</span>);
&nbsp; &nbsp; bootstrap = <span class="hljs-keyword">new</span> Bootstrap(); <span class="hljs-comment">// 创建Bootstrap</span>
&nbsp; &nbsp; bootstrap.group(NIO_EVENT_LOOP_GROUP)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .option(ChannelOption.SO_KEEPALIVE, <span class="hljs-keyword">true</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .option(ChannelOption.TCP_NODELAY, <span class="hljs-keyword">true</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .option(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .channel(socketChannelClass());
&nbsp; &nbsp; <span class="hljs-comment">// 设置连接超时时间，这里使用到AbstractEndpoint中的connectTimeout字段</span>
&nbsp; &nbsp; bootstrap.option(ChannelOption.CONNECT_TIMEOUT_MILLIS, Math.max(<span class="hljs-number">3000</span>, getConnectTimeout()));
&nbsp; &nbsp; bootstrap.handler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 心跳请求的时间间隔</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> heartbeatInterval = UrlUtils.getHeartbeat(getUrl());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 通过NettyCodecAdapter创建Netty中的编解码器，这里不再重复介绍</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NettyCodecAdapter adapter = <span class="hljs-keyword">new</span> NettyCodecAdapter(getCodec(), getUrl(), NettyClient.<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 注册ChannelHandler</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-string">"decoder"</span>, adapter.getDecoder())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"encoder"</span>, adapter.getEncoder())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"client-idle-handler"</span>, <span class="hljs-keyword">new</span> IdleStateHandler(heartbeatInterval, <span class="hljs-number">0</span>, <span class="hljs-number">0</span>, MILLISECONDS))
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"handler"</span>, nettyClientHandler);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果需要Socks5Proxy，需要添加Socks5ProxyHandler(略)</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; });
}
</code></pre>
<p data-nodeid="178737">得到的 NettyClient 结构如下图所示：</p>
<p data-nodeid="180294"><img src="https://s0.lgstatic.com/i/image/M00/5A/2F/Ciqc1F90P1yAYThvAADLV6SJeac973.png" alt="Lark20200930-161759.png" data-nodeid="180297"></p>

<div data-nodeid="179658"><p style="text-align:center">NettyClient 结构图</p></div>




<p data-nodeid="178740">NettyClientHandler 的实现方法与上一课时介绍的 NettyServerHandler 类似，同样是实现了 Netty 中的 ChannelDuplexHandler，其中会将所有方法委托给 NettyClient 关联的 ChannelHandler 对象进行处理。两者在 userEventTriggered() 方法的实现上有所不同，NettyServerHandler 在收到 IdleStateEvent 事件时会断开连接，而 NettyClientHandler 则会发送心跳消息，具体实现如下：</p>
<pre class="lang-java" data-nodeid="178741"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">userEventTriggered</span><span class="hljs-params">(ChannelHandlerContext ctx, Object evt)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (evt <span class="hljs-keyword">instanceof</span> IdleStateEvent) {
&nbsp; &nbsp; &nbsp; &nbsp; NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
&nbsp; &nbsp; &nbsp; &nbsp; Request req = <span class="hljs-keyword">new</span> Request();&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; req.setVersion(Version.getProtocolVersion());
&nbsp; &nbsp; &nbsp; &nbsp; req.setTwoWay(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; req.setEvent(HEARTBEAT_EVENT); <span class="hljs-comment">// 发送心跳请求</span>
&nbsp; &nbsp; &nbsp; &nbsp; channel.send(req);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>.userEventTriggered(ctx, evt);
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="178742">Channel 继承线分析</h3>
<p data-nodeid="178743">除了上一课时介绍的 AbstractEndpoint 之外，AbstractChannel 也继承了 AbstractPeer 这个抽象类，同时还继承了 Channel 接口。AbstractChannel 实现非常简单，只是在 send() 方法中检测了底层连接的状态，没有实现具体的发送消息的逻辑。</p>
<p data-nodeid="178744">这里我们依然以基于 Netty 4 的实现—— NettyChannel 为例，分析它对 AbstractChannel 的实现。NettyChannel 中的核心字段有如下几个。</p>
<ul data-nodeid="178745">
<li data-nodeid="178746">
<p data-nodeid="178747">channel（Channel类型）：Netty 框架中的 Channel，与当前的 Dubbo Channel 对象一一对应。</p>
</li>
<li data-nodeid="178748">
<p data-nodeid="178749">attributes（Map&lt;String, Object&gt;类型）：当前 Channel 中附加属性，都会记录到该 Map 中。NettyChannel 中提供的 getAttribute()、hasAttribute()、setAttribute() 等方法，都是操作该集合。</p>
</li>
<li data-nodeid="178750">
<p data-nodeid="178751">active（AtomicBoolean）：用于标识当前 Channel 是否可用。</p>
</li>
</ul>
<p data-nodeid="178752">另外，在 NettyChannel 中还有一个静态的 Map 集合（CHANNEL_MAP 字段），用来缓存当前 JVM 中 Netty 框架 Channel 与 Dubbo Channel 之间的映射关系。从下图的调用关系中可以看到，NettyChannel 提供了读写 CHANNEL_MAP 集合的方法：</p>
<p data-nodeid="178753"><img src="https://s0.lgstatic.com/i/image/M00/58/FF/CgqCHl9wcRiAZFTaAADTxIPND7k175.png" alt="Drawing 1.png" data-nodeid="178891"></p>
<p data-nodeid="178754">NettyChannel 中还有一个要介绍的是 send() 方法，它会通过底层关联的 Netty 框架 Channel，将数据发送到对端。其中，可以通过第二个参数指定是否等待发送操作结束，具体实现如下：</p>
<pre class="lang-java" data-nodeid="178755"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(Object message, <span class="hljs-keyword">boolean</span> sent)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
    <span class="hljs-comment">// 调用AbstractChannel的send()方法检测连接是否可用</span>
&nbsp; &nbsp; <span class="hljs-keyword">super</span>.send(message, sent); 
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> success = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; <span class="hljs-keyword">int</span> timeout = <span class="hljs-number">0</span>;
    <span class="hljs-comment">//&nbsp;依赖Netty框架的Channel发送数据</span>
&nbsp; &nbsp; ChannelFuture future = channel.writeAndFlush(message);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (sent) { <span class="hljs-comment">// 等待发送结束，有超时时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; timeout = getUrl().getPositiveParameter(TIMEOUT_KEY, DEFAULT_TIMEOUT);
&nbsp; &nbsp; &nbsp; &nbsp; success = future.await(timeout);
&nbsp; &nbsp; }
&nbsp; &nbsp; Throwable cause = future.cause();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cause != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> cause;
&nbsp; &nbsp; }
    <span class="hljs-comment">// 出现异常会调用removeChannelIfDisconnected()方法，在底层连接断开时，</span>
    <span class="hljs-comment">//&nbsp;会清理CHANNEL_MAP缓存(略)</span>
}
</code></pre>
<h3 data-nodeid="178756">ChannelHandler 继承线分析</h3>
<p data-nodeid="178757">前文介绍的 AbstractServer、AbstractClient 以及 Channel 实现，都是通过 AbstractPeer 实现了 ChannelHandler 接口，但只是做了一层简单的委托（也可以说成是装饰器），将全部方法委托给了其底层关联的 ChannelHandler 对象。</p>
<p data-nodeid="178758">这里我们就深入分析 ChannelHandler 的其他实现类，涉及的实现类如下所示：</p>
<p data-nodeid="178759"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wcSGAXo7JAANZ2BjquOE739.png" alt="Drawing 2.png" data-nodeid="178898"></p>
<div data-nodeid="178760"><p style="text-align:center">ChannelHandler 继承关系图</p></div>
<p data-nodeid="178761">其中<strong data-nodeid="178911">ChannelHandlerDispatcher</strong>在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=393#/detail/pc?id=4269" data-nodeid="178906">第 17</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=393#/detail/pc?id=4269" data-nodeid="178909">课时</a>已经介绍过了，它负责将多个 ChannelHandler 对象聚合成一个 ChannelHandler 对象。</p>
<p data-nodeid="178762"><strong data-nodeid="178916">ChannelHandlerAdapter</strong>是 ChannelHandler 的一个空实现，TelnetHandlerAdapter 继承了它并实现了 TelnetHandler 接口。至于Dubbo 对 Telnet 的支持，我们会在后面的课时中单独介绍，这里就先不展开分析了。</p>
<p data-nodeid="178763">从名字上看，<strong data-nodeid="178922">ChannelHandlerDelegate</strong>接口是对另一个 ChannelHandler 对象的封装，它的两个实现类 AbstractChannelHandlerDelegate 和 WrappedChannelHandler 中也仅仅是封装了另一个 ChannelHandler 对象。</p>
<p data-nodeid="178764">其中，<strong data-nodeid="178928">AbstractChannelHandlerDelegate</strong>有三个实现类，都比较简单，我们来逐个讲解。</p>
<ul data-nodeid="178765">
<li data-nodeid="178766">
<p data-nodeid="178767">MultiMessageHandler：专门处理 MultiMessage 的 ChannelHandler 实现。MultiMessage 是 Exchange 层的一种消息类型，它其中封装了多个消息。在 MultiMessageHandler 收到 MultiMessage 消息的时候，received() 方法会遍历其中的所有消息，并交给底层的 ChannelHandler 对象进行处理。</p>
</li>
<li data-nodeid="178768">
<p data-nodeid="178769">DecodeHandler：专门处理 Decodeable 的 ChannelHandler 实现。实现了 Decodeable 接口的类都会提供了一个 decode() 方法实现对自身的解码，DecodeHandler.received() 方法就是通过该方法得到解码后的消息，然后传递给底层的 ChannelHandler 对象继续处理。</p>
</li>
<li data-nodeid="178770">
<p data-nodeid="178771">HeartbeatHandler：专门处理心跳消息的 ChannelHandler 实现。在 HeartbeatHandler.received() 方法接收心跳请求的时候，会生成相应的心跳响应并返回；在收到心跳响应的时候，会打印相应的日志；在收到其他类型的消息时，会传递给底层的 ChannelHandler 对象进行处理。下面是其核心实现：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="178772"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">received</span><span class="hljs-params">(Channel channel, Object message)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; setReadTimestamp(channel); <span class="hljs-comment">// 记录最近的读写事件时间戳</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isHeartbeatRequest(message)) { <span class="hljs-comment">// 收到心跳请求</span>
&nbsp; &nbsp; &nbsp; &nbsp; Request req = (Request) message;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (req.isTwoWay()) { <span class="hljs-comment">// 返回心跳响应，注意，携带请求的ID</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Response res = <span class="hljs-keyword">new</span> Response(req.getId(), req.getVersion());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res.setEvent(HEARTBEAT_EVENT);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; channel.send(res);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isHeartbeatResponse(message)) { <span class="hljs-comment">// 收到心跳响应</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 打印日志(略)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; handler.received(channel, message);
}
</code></pre>
<p data-nodeid="178773">另外，我们可以看到，在 received() 和 send() 方法中，HeartbeatHandler 会将最近一次的读写时间作为附加属性记录到 Channel 中。</p>
<p data-nodeid="178774">通过上述介绍，我们发现 AbstractChannelHandlerDelegate 下的三个实现，其实都是在原有 ChannelHandler 的基础上添加了一些增强功能，这是典型的装饰器模式的应用。</p>
<h4 data-nodeid="178775">Dispatcher 与 ChannelHandler</h4>
<p data-nodeid="178776">接下来，我们介绍 ChannelHandlerDelegate 接口的另一条继承线——<strong data-nodeid="178940">WrappedChannelHandler</strong>，其子类主要是决定了 Dubbo 以何种线程模型处理收到的事件和消息，就是所谓的“消息派发机制”，与前面介绍的 ThreadPool 有紧密的联系。</p>
<p data-nodeid="178777"><img src="https://s0.lgstatic.com/i/image/M00/58/FF/CgqCHl9wcTGAdInYAAJOSSxusf4539.png" alt="Drawing 3.png" data-nodeid="178943"></p>
<div data-nodeid="178778"><p style="text-align:center">WrappedChannelHandler 继承关系图</p></div>
<p data-nodeid="178779">从上图中我们可以看到，每个 WrappedChannelHandler 实现类的对象都由一个相应的 Dispatcher 实现类创建，下面是 Dispatcher 接口的定义：</p>
<pre class="lang-java" data-nodeid="178780"><code data-language="java"><span class="hljs-meta">@SPI(AllDispatcher.NAME)</span> <span class="hljs-comment">// 默认扩展名是all</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Dispatcher</span> </span>{
    <span class="hljs-comment">// 通过URL中的参数可以指定扩展名，覆盖默认扩展名</span>
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({"dispatcher", "dispather", "channel.handler"})</span>
&nbsp; &nbsp; <span class="hljs-function">ChannelHandler <span class="hljs-title">dispatch</span><span class="hljs-params">(ChannelHandler handler, URL url)</span></span>;
}
</code></pre>
<p data-nodeid="178781"><strong data-nodeid="178949">AllDispatcher 创建的是 AllChannelHandler 对象</strong>，它会将所有网络事件以及消息交给关联的线程池进行处理。AllChannelHandler覆盖了 WrappedChannelHandler 中除了 sent() 方法之外的其他网络事件处理方法，将调用其底层的 ChannelHandler 的逻辑放到关联的线程池中执行。</p>
<p data-nodeid="178782">我们先来看 connect() 方法，其中会将CONNECTED 事件的处理封装成ChannelEventRunnable提交到线程池中执行，具体实现如下：</p>
<pre class="lang-java" data-nodeid="178783"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">connected</span><span class="hljs-params">(Channel channel)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; ExecutorService executor = getExecutorService(); <span class="hljs-comment">// 获取公共线程池</span>
    <span class="hljs-comment">// 将CONNECTED事件的处理封装成ChannelEventRunnable提交到线程池中执行</span>
   &nbsp;executor.execute(<span class="hljs-keyword">new</span> ChannelEventRunnable(channel, handler, ChannelState.CONNECTED));
    <span class="hljs-comment">// 省略异常处理的逻辑</span>
}
</code></pre>
<p data-nodeid="178784">这里的 getExecutorService() 方法会按照当前端点（Server/Client）的 URL 从 ExecutorRepository 中获取相应的公共线程池。</p>
<p data-nodeid="178785">disconnected()方法处理连接断开事件，caught() 方法处理异常事件，它们也是按照上述方式实现的，这里不再展开赘述。</p>
<p data-nodeid="178786">received() 方法会在当前端点收到数据的时候被调用，具体执行流程是先由 IO 线程（也就是 Netty 中的 EventLoopGroup）从二进制流中解码出请求，然后调用 AllChannelHandler 的 received() 方法，其中会将请求提交给线程池执行，执行完后调用 sent()方法，向对端写回响应结果。received() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="178787"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">received</span><span class="hljs-params">(Channel channel, Object message)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 获取线程池</span>
&nbsp; &nbsp; ExecutorService executor = getPreferredExecutorService(message);
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将消息封装成ChannelEventRunnable任务，提交到线程池中执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; executor.execute(<span class="hljs-keyword">new</span> ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果线程池满了，请求会被拒绝，这里会根据请求配置决定是否返回一个说明性的响应</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span>(message <span class="hljs-keyword">instanceof</span> Request &amp;&amp; t <span class="hljs-keyword">instanceof</span> RejectedExecutionException){
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sendFeedback(channel, (Request) message, t);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ExecutionException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="178788">getPreferredExecutorService() 方法对响应做了特殊处理：如果请求在发送的时候指定了关联的线程池，在收到对应的响应消息的时候，会优先根据请求的 ID 查找请求关联的线程池处理响应。</p>
<pre class="lang-java" data-nodeid="178789"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ExecutorService <span class="hljs-title">getPreferredExecutorService</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (msg <span class="hljs-keyword">instanceof</span> Response) {
&nbsp; &nbsp; &nbsp; &nbsp; Response response = (Response) msg;
&nbsp; &nbsp; &nbsp; &nbsp; DefaultFuture responseFuture = DefaultFuture.getFuture(response.getId()); <span class="hljs-comment">// 获取请求关联的DefaultFuture</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (responseFuture == <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getSharedExecutorService();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 如果请求关联了线程池，则会获取相关的线程来处理响应</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ExecutorService executor = responseFuture.getExecutor();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor == <span class="hljs-keyword">null</span> || executor.isShutdown()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; executor = getSharedExecutorService();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> executor;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 如果是请求消息，则直接使用公共的线程池处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getSharedExecutorService();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="178790">这里涉及了 Request 和 Response 的概念，是 Exchange 层的概念，在后面会展开介绍，这里你只需要知道它们是不同的消息类型即可。</p>
<p data-nodeid="178791">注意，AllChannelHandler 并没有覆盖父类的 sent() 方法，也就是说，发送消息是直接在当前线程调用 sent() 方法完成的。</p>
<p data-nodeid="178792">下面我们来看剩余的 WrappedChannelHandler 的实现。ExecutionChannelHandler（由 ExecutionDispatcher 创建）只会将请求消息派发到线程池进行处理，也就是只重写了 received() 方法。对于响应消息以及其他网络事件（例如，连接建立事件、连接断开事件、心跳消息等），ExecutionChannelHandler 会直接在 IO 线程中进行处理。</p>
<p data-nodeid="178793">DirectChannelHandler 实现（由 DirectDispatcher 创建）会在 IO 线程中处理所有的消息和网络事件。</p>
<p data-nodeid="178794">MessageOnlyChannelHandler 实现（由 MessageOnlyDispatcher 创建）会将所有收到的消息提交到线程池处理，其他网络事件则是由 IO 线程直接处理。</p>
<p data-nodeid="178795">ConnectionOrderedChannelHandler 实现（由 ConnectionOrderedDispatcher 创建）会将收到的消息交给线程池进行处理，对于连接建立以及断开事件，会提交到一个独立的线程池并排队进行处理。在 ConnectionOrderedChannelHandler 的构造方法中，会初始化一个线程池，该线程池的队列长度是固定的：</p>
<pre class="lang-java" data-nodeid="178796"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ConnectionOrderedChannelHandler</span><span class="hljs-params">(ChannelHandler handler, URL url)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(handler, url);
&nbsp; &nbsp; String threadName = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
    <span class="hljs-comment">// 注意，该线程池只有一个线程，队列的长度也是固定的，</span>
    <span class="hljs-comment">// 由URL中的connect.queue.capacity参数指定</span>
&nbsp; &nbsp; connectionExecutor = <span class="hljs-keyword">new</span> ThreadPoolExecutor(<span class="hljs-number">1</span>, <span class="hljs-number">1</span>, 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0L</span>, TimeUnit.MILLISECONDS,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> LinkedBlockingQueue&lt;Runnable&gt;(url.getPositiveParameter(CONNECT_QUEUE_CAPACITY, Integer.MAX_VALUE)),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> NamedThreadFactory(threadName, <span class="hljs-keyword">true</span>),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> AbortPolicyWithReport(threadName, url)
&nbsp; &nbsp; );
&nbsp; &nbsp; queuewarninglimit = url.getParameter(CONNECT_QUEUE_WARNING_SIZE, DEFAULT_CONNECT_QUEUE_WARNING_SIZE);
}
</code></pre>
<p data-nodeid="178797">在 ConnectionOrderedChannelHandler 的 connected() 方法和 disconnected() 方法实现中，会将连接建立和断开事件交给上述 connectionExecutor 线程池排队处理。</p>
<p data-nodeid="178798">在上面介绍 WrappedChannelHandler 各个实现的时候，我们会看到其中有针对 ThreadlessExecutor 这种线程池类型的特殊处理，例如，ExecutionChannelHandler.received() 方法中就有如下的分支逻辑：</p>
<pre class="lang-java" data-nodeid="178799"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">received</span><span class="hljs-params">(Channel channel, Object message)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 获取线程池（请求绑定的线程池或是公共线程池）</span>
&nbsp; &nbsp; ExecutorService executor = getPreferredExecutorService(message);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (message <span class="hljs-keyword">instanceof</span> Request) { <span class="hljs-comment">// 请求消息直接提交给线程池处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; executor.execute(<span class="hljs-keyword">new</span> ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (executor <span class="hljs-keyword">instanceof</span> ThreadlessExecutor) {
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 针对ThreadlessExecutor这种线程池类型的特殊处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; executor.execute(<span class="hljs-keyword">new</span> ChannelEventRunnable(channel, handler, ChannelState.RECEIVED, message));
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; handler.received(channel, message);
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="178800">ThreadlessExecutor 优化</h4>
<p data-nodeid="178801">ThreadlessExecutor 是一种特殊类型的线程池，与其他正常的线程池最主要的区别是：<strong data-nodeid="178969">ThreadlessExecutor 内部不管理任何线程</strong>。</p>
<p data-nodeid="178802">我们可以调用 ThreadlessExecutor 的execute() 方法，将任务提交给这个线程池，但是这些提交的任务不会被调度到任何线程执行，而是存储在阻塞队列中，只有当其他线程调用 ThreadlessExecutor.waitAndDrain() 方法时才会真正执行。也说就是，执行任务的与调用 waitAndDrain() 方法的是同一个线程。</p>
<p data-nodeid="178803">**那为什么会有 ThreadlessExecutor 这个实现呢？**这主要是因为在 Dubbo 2.7.5 版本之前，在 WrappedChannelHandler 中会为每个连接启动一个线程池。</p>
<blockquote data-nodeid="178804">
<p data-nodeid="178805">老版本中没有 ExecutorRepository 的概念，不会根据 URL 复用同一个线程池，而是通过 SPI 找到 ThreadPool 实现创建新线程池。</p>
</blockquote>
<p data-nodeid="178806">此时，Dubbo Consumer 同步请求的线程模型如下图所示：</p>
<p data-nodeid="178807"><img src="https://s0.lgstatic.com/i/image/M00/58/FF/CgqCHl9wcUWAY3b0AAFKI4e5Oa0017.png" alt="Drawing 4.png" data-nodeid="178981"></p>
<div data-nodeid="178808"><p style="text-align:center">Dubbo Consumer 同步请求线程模型</p></div>
<p data-nodeid="178809">从图中我们可以看到下面的请求-响应流程：</p>
<ol data-nodeid="178810">
<li data-nodeid="178811">
<p data-nodeid="178812">业务线程发出请求之后，拿到一个 Future 实例。</p>
</li>
<li data-nodeid="178813">
<p data-nodeid="178814">业务线程紧接着调用 Future.get() 阻塞等待请求结果返回。</p>
</li>
<li data-nodeid="178815">
<p data-nodeid="178816">当响应返回之后，交由连接关联的独立线程池进行反序列化等解析处理。</p>
</li>
<li data-nodeid="178817">
<p data-nodeid="178818">待处理完成之后，将业务结果通过 Future.set() 方法返回给业务线程。</p>
</li>
</ol>
<p data-nodeid="178819">在这个设计里面，Consumer 端会维护一个线程池，而且线程池是按照连接隔离的，即每个连接独享一个线程池。这样，当面临需要消费大量服务且并发数比较大的场景时，例如，典型网关类场景，可能会导致 Consumer 端线程个数不断增加，导致线程调度消耗过多 CPU ，也可能因为线程创建过多而导致 OOM。</p>
<p data-nodeid="178820">为了解决上述问题，Dubbo 在 2.7.5 版本之后，<strong data-nodeid="178993">引入了 ThreadlessExecutor</strong>，将线程模型修改成了下图的样子：</p>
<p data-nodeid="178821"><img src="https://s0.lgstatic.com/i/image/M00/58/FF/CgqCHl9wcVCAQdJjAAFE8eFivcY750.png" alt="Drawing 5.png" data-nodeid="178996"></p>
<div data-nodeid="178822"><p style="text-align:center">引入 ThreadlessExecutor 后的结构图</p></div>
<ol data-nodeid="178823">
<li data-nodeid="178824">
<p data-nodeid="178825">业务线程发出请求之后，拿到一个 Future 对象。</p>
</li>
<li data-nodeid="178826">
<p data-nodeid="178827">业务线程会调用 ThreadlessExecutor.waitAndDrain() 方法，waitAndDrain() 方法会在阻塞队列上等待。</p>
</li>
<li data-nodeid="178828">
<p data-nodeid="178829">当收到响应时，IO 线程会生成一个任务，填充到 ThreadlessExecutor 队列中，</p>
</li>
<li data-nodeid="178830">
<p data-nodeid="178831">业务线程会将上面添加的任务取出，并在本线程中执行。得到业务结果之后，调用 Future.set() 方法进行设置，此时 waitAndDrain() 方法返回。</p>
</li>
<li data-nodeid="178832">
<p data-nodeid="178833">业务线程从 Future 中拿到结果值。</p>
</li>
</ol>
<p data-nodeid="178834">了解了 ThreadlessExecutor 出现的缘由之后，接下来我们再深入了解一下 ThreadlessExecutor 的核心实现。首先是 ThreadlessExecutor 的核心字段，有如下几个。</p>
<ul data-nodeid="178835">
<li data-nodeid="178836">
<p data-nodeid="178837">queue（LinkedBlockingQueue类型）：阻塞队列，用来在 IO 线程和业务线程之间传递任务。</p>
</li>
<li data-nodeid="178838">
<p data-nodeid="178839">waiting、finished（Boolean类型）：ThreadlessExecutor 中的 waitAndDrain() 方法一般与一次 RPC 调用绑定，只会执行一次。当后续再次调用 waitAndDrain() 方法时，会检查 finished 字段，若为true，则此次调用直接返回。当后续再次调用 execute() 方法提交任务时，会根据 waiting 字段决定任务是放入 queue 队列等待业务线程执行，还是直接由 sharedExecutor 线程池执行。</p>
</li>
<li data-nodeid="178840">
<p data-nodeid="178841">sharedExecutor（ExecutorService类型）：ThreadlessExecutor 底层关联的共享线程池，当业务线程已经不再等待响应时，会由该共享线程执行提交的任务。</p>
</li>
<li data-nodeid="178842">
<p data-nodeid="178843">waitingFuture（CompletableFuture类型）：指向请求对应的 DefaultFuture 对象，其具体实现我们会在后面的课时详细展开介绍。</p>
</li>
</ul>
<p data-nodeid="178844"><strong data-nodeid="179016">ThreadlessExecutor 的核心逻辑在 execute() 方法和 waitAndDrain() 方法</strong>**。**execute() 方法相对简单，它会根据 waiting 状态决定任务提交到哪里，相关示例代码如下：</p>
<pre class="lang-java" data-nodeid="178845"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">execute</span><span class="hljs-params">(Runnable runnable)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (lock) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!waiting) { <span class="hljs-comment">// 判断业务线程是否还在等待响应结果</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 不等待，则直接交给共享线程池处理任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sharedExecutor.execute(runnable); 
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {<span class="hljs-comment">// 业务线程还在等待，则将任务写入队列，然后由业务线程自己执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; queue.add(runnable);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="178846">waitAndDrain() 方法中首先会检测 finished 字段值，然后获取阻塞队列中的全部任务并执行，执行完成之后会修改finished和 waiting 字段，标识当前 ThreadlessExecutor 已使用完毕，无业务线程等待。</p>
<pre class="lang-java" data-nodeid="178847"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">waitAndDrain</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (finished) { <span class="hljs-comment">// 检测当前ThreadlessExecutor状态</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 获取阻塞队列中获取任务</span>
&nbsp; &nbsp; Runnable runnable = queue.take();
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (lock) {
&nbsp; &nbsp; &nbsp; &nbsp; waiting = <span class="hljs-keyword">false</span>; <span class="hljs-comment">// 修改waiting状态</span>
&nbsp; &nbsp; &nbsp; &nbsp; runnable.run(); <span class="hljs-comment">// 执行任务</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; runnable = queue.poll(); <span class="hljs-comment">// 如果阻塞队列中还有其他任务，也需要一并执行</span>
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (runnable != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; runnable.run(); <span class="hljs-comment">// 省略异常处理逻辑</span>
&nbsp; &nbsp; &nbsp; &nbsp; runnable = queue.poll();
&nbsp; &nbsp; }
&nbsp; &nbsp; finished = <span class="hljs-keyword">true</span>; <span class="hljs-comment">// 修改finished状态</span>
}
</code></pre>
<p data-nodeid="178848">到此为止，Transporter 层对 ChannelHandler 的实现就介绍完了，其中涉及了多个 ChannelHandler 的装饰器，为了帮助你更好地理解，这里我们回到 NettyServer 中，看看它是如何对上层 ChannelHandler 进行封装的。</p>
<p data-nodeid="178849">在 NettyServer 的构造方法中会调用 ChannelHandlers.wrap() 方法对传入的 ChannelHandler 对象进行修饰：</p>
<pre class="lang-java" data-nodeid="178850"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> ChannelHandler <span class="hljs-title">wrapInternal</span><span class="hljs-params">(ChannelHandler handler, URL url)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MultiMessageHandler(<span class="hljs-keyword">new</span> HeartbeatHandler(ExtensionLoader.getExtensionLoader(Dispatcher.class)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .getAdaptiveExtension().dispatch(handler, url)));
}
</code></pre>
<p data-nodeid="178851">结合前面的分析，我们可以得到下面这张图：</p>
<p data-nodeid="178852"><img src="https://s0.lgstatic.com/i/image/M00/58/F4/Ciqc1F9wcV-AFAcTAADpElrp-Wc888.png" alt="Drawing 6.png" data-nodeid="179023"></p>
<div data-nodeid="178853"><p style="text-align:center">Server 端 ChannelHandler 结构图</p></div>
<p data-nodeid="178854">我们可以在创建 NettyServerHandler 的地方添加断点 Debug 得到下图，也印证了上图的内容：</p>
<p data-nodeid="178855"><img src="https://s0.lgstatic.com/i/image/M00/58/FF/CgqCHl9wcWaAJVA3AACBSF4eCzg786.png" alt="Drawing 7.png" data-nodeid="179027"></p>
<h3 data-nodeid="178856">总结</h3>
<p data-nodeid="178857">本课时我们重点介绍了 Dubbo Transporter 层中 Client、 Channel、ChannelHandler 相关的实现以及优化。</p>
<p data-nodeid="178858">首先我们介绍了 AbstractClient 抽象接口以及基于 Netty 4 的 NettyClient 实现。接下来，介绍了 AbstractChannel 抽象类以及 NettyChannel 实现。最后，我们深入分析了 ChannelHandler 接口实现，其中详细分析 WrappedChannelHandler 等关键 ChannelHandler 实现，以及 ThreadlessExecutor 优化。</p>
<p data-nodeid="178859" class="">关于Dubbo 的 Transporter 层，你若还有什么疑问或想法，欢迎你留言跟我分享。</p>

---

### 精选评论

##### **伟：
> 看了三遍了，感觉还需要再啃好多遍。。。😇

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 加油

##### *泽：
> 打卡

##### **飞：
> 请问线程池什么时候会关闭或销毁呢？

##### **同学：
> 这一块儿 有点费脑子

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 多看几遍，加油哇~

