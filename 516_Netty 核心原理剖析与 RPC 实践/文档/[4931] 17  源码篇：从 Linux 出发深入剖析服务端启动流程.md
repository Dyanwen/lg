<p data-nodeid="1217" class="">通过前几章课程的学习，我们已经对 Netty 的技术思想和基本原理有了初步的认识，从今天这节课开始我们将正式进入 Netty 核心源码学习的课程。希望能够通过源码解析的方式让你更加深入理解 Netty 的精髓，如 Netty 的设计思想、工程技巧等，为之后继续深入研究 Netty 打下坚实的基础。</p>
<p data-nodeid="1218">在课程开始之前，我想分享一下关于源码学习的几点经验和建议。第一，很多同学在开始学习源码时面临的第一个问题就是不知道从何下手，这个时候一定不能对着源码毫无意义地四处翻看。建议你可以通过 Hello World 或者 TestCase 作为源码学习的入口，然后再通过 Debug 断点的方式调试并跑通源码。第二，阅读源码一定要有全局观。首先要把握源码的主流程，避免刚开始陷入代码细节的死胡同。第三，源码一定要反复阅读，让自己每一次读都有不同的收获。我们可以通过画图、注释的方式帮助自己更容易理解源码的核心流程，方便后续的复习和回顾。</p>
<p data-nodeid="1219">作为源码解析的第一节课，我们将深入分析 Netty 服务端的启动流程。启动服务的过程中我们可以了解到 Netty 各大核心组件的关系，这将是学习 Netty 源码一个非常好的切入点，让我们一起看看 Netty 的每个零件是如何运转起来的吧。</p>
<blockquote data-nodeid="1220">
<p data-nodeid="1221">说明：本文参考的 Netty 源码版本为 4.1.42.Final。</p>
</blockquote>
<h3 data-nodeid="1222">从 Echo 服务器示例入手</h3>
<p data-nodeid="1223">在《引导器作用：客户端和服务端启动都要做些什么？》的课程中，我们介绍了如何使用引导器搭建服务端的基本框架。在这里我们实现了一个最简单的 Echo 服务器，用于调试 Netty 服务端启动的源码。</p>
<pre class="lang-java" data-nodeid="1224"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">startEchoServer</span><span class="hljs-params">(<span class="hljs-keyword">int</span> port)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; EventLoopGroup bossGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();
&nbsp; &nbsp; &nbsp; &nbsp; EventLoopGroup workerGroup = <span class="hljs-keyword">new</span> NioEventLoopGroup();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ServerBootstrap b = <span class="hljs-keyword">new</span> ServerBootstrap();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; b.group(bossGroup, workerGroup)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .channel(NioServerSocketChannel.class)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .handler(<span class="hljs-keyword">new</span> LoggingHandler(LogLevel.INFO)) <span class="hljs-comment">// 设置ServerSocketChannel 对应的 Handler</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() { <span class="hljs-comment">// 设置 SocketChannel 对应的 Handler</span>
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
}
</code></pre>
<p data-nodeid="1225">我们以引导器 ServerBootstrap 为切入点，开始深入分析 Netty 服务端的启动流程。在服务端启动之前，需要配置 ServerBootstrap 的相关参数，这一步大致可以分为以下几个步骤：</p>
<ul data-nodeid="1226">
<li data-nodeid="1227">
<p data-nodeid="1228">配置 EventLoopGroup 线程组；</p>
</li>
<li data-nodeid="1229">
<p data-nodeid="1230">配置 Channel 的类型；</p>
</li>
<li data-nodeid="1231">
<p data-nodeid="1232">设置 ServerSocketChannel 对应的 Handler；</p>
</li>
<li data-nodeid="1233">
<p data-nodeid="1234">设置网络监听的端口；</p>
</li>
<li data-nodeid="1235">
<p data-nodeid="1236">设置 SocketChannel 对应的 Handler；</p>
</li>
<li data-nodeid="1237">
<p data-nodeid="1238">配置 Channel 参数。</p>
</li>
</ul>
<p data-nodeid="1239">配置 ServerBootstrap 参数的过程非常简单，把参数值保存在 ServerBootstrap 定义的成员变量里就可以了。我们可以看下 ServerBootstrap 的成员变量定义，基本与 ServerBootstrap 暴露出来的配置方法是一一对应的。如下所示，我以注释的形式说明每个成员变量对应的调用方法。</p>
<pre class="lang-java" data-nodeid="1240"><code data-language="java"><span class="hljs-keyword">volatile</span> EventLoopGroup group; <span class="hljs-comment">// group()</span>
<span class="hljs-keyword">volatile</span> EventLoopGroup childGroup; <span class="hljs-comment">// group()</span>
<span class="hljs-keyword">volatile</span> ChannelFactory&lt;? extends C&gt; channelFactory; <span class="hljs-comment">// channel()</span>
<span class="hljs-keyword">volatile</span> SocketAddress localAddress; <span class="hljs-comment">// localAddress</span>
Map&lt;ChannelOption&lt;?&gt;, Object&gt; childOptions = <span class="hljs-keyword">new</span> ConcurrentHashMap&lt;ChannelOption&lt;?&gt;, Object&gt;(); <span class="hljs-comment">// childOption()</span>
<span class="hljs-keyword">volatile</span> ChannelHandler childHandler; <span class="hljs-comment">// childHandler()</span>
ServerBootstrapConfig config = <span class="hljs-keyword">new</span> ServerBootstrapConfig(<span class="hljs-keyword">this</span>);
</code></pre>
<p data-nodeid="1241">关于 ServerBootstrap 如何为每个成员变量保存参数的过程，我们就不一一展开了，你可以理解为这部分工作只是一个前置准备，课后你可以自己跟进下每个方法的源码。今天我们核心聚焦在 b.bind().sync() 这行代码，bind() 才是真正进行服务器端口绑定和启动的入口，sync() 表示阻塞等待服务器启动完成。接下来我们对 bind() 方法进行展开分析。</p>
<p data-nodeid="1242">在开始源码分析之前，我们带着以下几个问题边看边思考：</p>
<ul data-nodeid="1243">
<li data-nodeid="1244">
<p data-nodeid="1245">Netty 自己实现的 Channel 与 JDK 底层的 Channel 是如何产生联系的？</p>
</li>
<li data-nodeid="1246">
<p data-nodeid="1247">ChannelInitializer 这个特殊的 Handler 处理器的作用是什么？</p>
</li>
<li data-nodeid="1248">
<p data-nodeid="1249">Pipeline 初始化的过程是什么样的？</p>
</li>
</ul>
<h3 data-nodeid="1250">服务端启动全过程</h3>
<p data-nodeid="1251">首先我们来看下 ServerBootstrap 中 bind() 方法的源码实现：</p>
<pre class="lang-java" data-nodeid="1252"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ChannelFuture <span class="hljs-title">bind</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; validate();
&nbsp; &nbsp; SocketAddress localAddress = <span class="hljs-keyword">this</span>.localAddress;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (localAddress == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"localAddress not set"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> doBind(localAddress);
}
<span class="hljs-function"><span class="hljs-keyword">private</span> ChannelFuture <span class="hljs-title">doBind</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SocketAddress localAddress)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelFuture regFuture = initAndRegister();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Channel channel = regFuture.channel();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (regFuture.cause() != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> regFuture;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (regFuture.isDone()) {
&nbsp; &nbsp; &nbsp; &nbsp; ChannelPromise promise = channel.newPromise();
&nbsp; &nbsp; &nbsp; &nbsp; doBind0(regFuture, channel, localAddress, promise);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> promise;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> PendingRegistrationPromise promise = <span class="hljs-keyword">new</span> PendingRegistrationPromise(channel);
&nbsp; &nbsp; &nbsp; &nbsp; regFuture.addListener(<span class="hljs-keyword">new</span> ChannelFutureListener() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">operationComplete</span><span class="hljs-params">(ChannelFuture future)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Throwable cause = future.cause();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cause != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; promise.setFailure(cause);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; promise.registered();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; doBind0(regFuture, channel, localAddress, promise);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> promise;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1253">由此可见，doBind() 方法是我们需要分析的重点。我们再一起看下 doBind() 具体做了哪些事情：</p>
<ol data-nodeid="1254">
<li data-nodeid="1255">
<p data-nodeid="1256">调用 initAndRegister() 初始化并注册 Channel，同时返回一个 ChannelFuture 实例 regFuture，所以我们可以猜测出 initAndRegister() 是一个异步的过程。</p>
</li>
<li data-nodeid="1257">
<p data-nodeid="1258">接下来通过 regFuture.cause() 方法判断 initAndRegister() 的过程是否发生异常，如果发生异常则直接返回。</p>
</li>
<li data-nodeid="1259">
<p data-nodeid="1260">regFuture.isDone() 表示 initAndRegister() 是否执行完毕，如果执行完毕则调用 doBind0() 进行 Socket 绑定。如果 initAndRegister() 还没有执行结束，regFuture 会添加一个 ChannelFutureListener 回调监听，当 initAndRegister() 执行结束后会调用 operationComplete()，同样通过 doBind0() 进行端口绑定。</p>
</li>
</ol>
<p data-nodeid="1261">doBind() 整个实现结构非常清晰，其中 initAndRegister() 负责 Channel 初始化和注册，doBind0() 用于端口绑定。这两个过程最为重要，下面我们分别进行详细的介绍。</p>
<h3 data-nodeid="1262">服务端 Channel 初始化及注册</h3>
<p data-nodeid="1263">initAndRegister() 方法顾名思义，主要负责初始化和注册的相关工作，我们具体看下它的源码实现：</p>
<pre class="lang-java" data-nodeid="1264"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">final</span> ChannelFuture <span class="hljs-title">initAndRegister</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; Channel channel = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; channel = channelFactory.newChannel(); <span class="hljs-comment">// 创建 Channel</span>
&nbsp; &nbsp; &nbsp; &nbsp; init(channel); <span class="hljs-comment">// 初始化 Channel</span>
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (channel != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; channel.unsafe().closeForcibly();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultChannelPromise(channel, GlobalEventExecutor.INSTANCE).setFailure(t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultChannelPromise(<span class="hljs-keyword">new</span> FailedChannel(), GlobalEventExecutor.INSTANCE).setFailure(t);
&nbsp; &nbsp; }
&nbsp; &nbsp; ChannelFuture regFuture = config().group().register(channel); <span class="hljs-comment">// 注册 Channel</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (regFuture.cause() != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (channel.isRegistered()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; channel.close();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; channel.unsafe().closeForcibly();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> regFuture;
}
</code></pre>
<p data-nodeid="1265">initAndRegister() 可以分为三步：创建 Channel、初始化 Channel 和注册 Channel，接下来我们一步步进行拆解分析。</p>
<h4 data-nodeid="1266">创建服务端 Channel</h4>
<p data-nodeid="1267">首先看下创建 Channel 的过程，直接跟进 channelFactory.newChannel() 的源码。</p>
<pre class="lang-java" data-nodeid="1268"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReflectiveChannelFactory</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Channel</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">ChannelFactory</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Constructor&lt;? extends T&gt; constructor;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ReflectiveChannelFactory</span><span class="hljs-params">(Class&lt;? extends T&gt; clazz)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; ObjectUtil.checkNotNull(clazz, <span class="hljs-string">"clazz"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.constructor = clazz.getConstructor();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (NoSuchMethodException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"Class "</span> + StringUtil.simpleClassName(clazz) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">" does not have a public non-arg constructor"</span>, e);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">newChannel</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> constructor.newInstance(); <span class="hljs-comment">// 反射创建对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ChannelException(<span class="hljs-string">"Unable to create Channel from class "</span> + constructor.getDeclaringClass(), t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="1269">在前面 Echo 服务器的示例中，我们通过 channel(NioServerSocketChannel.class) 配置 Channel 的类型，工厂类 ReflectiveChannelFactory 是在该过程中被创建的。从 constructor.newInstance() 我们可以看出，ReflectiveChannelFactory 通过反射创建出 NioServerSocketChannel 对象，所以我们重点需要关注 NioServerSocketChannel 的构造函数。</p>
<pre class="lang-java" data-nodeid="1270"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">NioServerSocketChannel</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">this</span>(newSocket(DEFAULT_SELECTOR_PROVIDER));&nbsp;
}
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">NioServerSocketChannel</span><span class="hljs-params">(ServerSocketChannel channel)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(<span class="hljs-keyword">null</span>, channel, SelectionKey.OP_ACCEPT); <span class="hljs-comment">// 调用父类方法</span>
&nbsp; &nbsp; config = <span class="hljs-keyword">new</span> NioServerSocketChannelConfig(<span class="hljs-keyword">this</span>, javaChannel().socket());
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> ServerSocketChannel <span class="hljs-title">newSocket</span><span class="hljs-params">(SelectorProvider provider)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> provider.openServerSocketChannel(); <span class="hljs-comment">// 创建 JDK 底层的 ServerSocketChannel</span>
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (IOException e) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ChannelException(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Failed to open a server socket."</span>, e);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1271">SelectorProvider 是 JDK NIO 中的抽象类实现，通过 openServerSocketChannel() 方法可以用于创建服务端的 ServerSocketChannel。而且 SelectorProvider 会根据操作系统类型和版本的不同，返回不同的实现类，具体可以参考 DefaultSelectorProvider 的源码实现：</p>
<pre class="lang-java" data-nodeid="1272"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> SelectorProvider <span class="hljs-title">create</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; String osname = AccessController
&nbsp; &nbsp; &nbsp; &nbsp; .doPrivileged(<span class="hljs-keyword">new</span> GetPropertyAction(<span class="hljs-string">"os.name"</span>));
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (osname.equals(<span class="hljs-string">"SunOS"</span>))
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> createProvider(<span class="hljs-string">"sun.nio.ch.DevPollSelectorProvider"</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (osname.equals(<span class="hljs-string">"Linux"</span>))
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> createProvider(<span class="hljs-string">"sun.nio.ch.EPollSelectorProvider"</span>);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> sun.nio.ch.PollSelectorProvider();
}
</code></pre>
<p data-nodeid="1273">在这里我们只讨论 Linux 操作系统的场景，在 Linux 内核 2.6版本及以上都会默认采用 EPollSelectorProvider。如果是旧版本则使用 PollSelectorProvider。对于目前的主流 Linux 平台而言，都是采用 Epoll 机制实现的。</p>
<p data-nodeid="1274">创建完 ServerSocketChannel，我们回到 NioServerSocketChannel 的构造函数，接着它会通过 super() 依次调用到父类的构造进行初始化工作，最终我们可以定位到 AbstractNioChannel 和 AbstractChannel 的构造函数：</p>
<pre class="lang-java" data-nodeid="1275"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">AbstractNioChannel</span><span class="hljs-params">(Channel parent, SelectableChannel ch, <span class="hljs-keyword">int</span> readInterestOp)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(parent);
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ch.configureBlocking(<span class="hljs-keyword">false</span>);
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (IOException e) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">AbstractChannel</span><span class="hljs-params">(Channel parent)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.parent = parent;
&nbsp; &nbsp; id = newId(); <span class="hljs-comment">// Channel 全局唯一 id&nbsp;</span>
&nbsp; &nbsp; unsafe = newUnsafe(); <span class="hljs-comment">// unsafe 操作底层读写</span>
&nbsp; &nbsp; pipeline = newChannelPipeline(); <span class="hljs-comment">// pipeline 负责业务处理器编排</span>
}
</code></pre>
<p data-nodeid="1276">首先调用 AbstractChannel 的构造函数创建了三个重要的成员变量，分别为 id、unsafe、pipeline。id 表示全局唯一的 Channel，unsafe 用于操作底层数据的读写操作，pipeline 负责业务处理器的编排。初始化状态，pipeline 的内部结构只包含头尾两个节点，如下图所示。三个核心成员变量创建好之后，会回到 AbstractNioChannel 的构造函数，通过 ch.configureBlocking(false) 设置 Channel 是非阻塞模式。</p>
<p data-nodeid="1277"><img src="https://s0.lgstatic.com/i/image/M00/87/93/Ciqc1F_V_rKAO2pAAANaFwMrmS4362.png" alt="netty17图.png" data-nodeid="1403"></p>
<p data-nodeid="1278">创建服务端 Channel 的过程我们已经讲完了，简单总结下其中几个重要的步骤：</p>
<ol data-nodeid="1279">
<li data-nodeid="1280">
<p data-nodeid="1281">ReflectiveChannelFactory 通过反射创建 NioServerSocketChannel 实例；</p>
</li>
<li data-nodeid="1282">
<p data-nodeid="1283">创建 JDK 底层的 ServerSocketChannel；</p>
</li>
<li data-nodeid="1284">
<p data-nodeid="1285">为 Channel 创建 id、unsafe、pipeline 三个重要的成员变量；</p>
</li>
<li data-nodeid="1286">
<p data-nodeid="1287">设置 Channel 为非阻塞模式。</p>
</li>
</ol>
<h4 data-nodeid="1288">初始化服务端 Channel</h4>
<p data-nodeid="1289">回到 ServerBootstrap 的 initAndRegister() 方法，继续跟进用于初始化服务端 Channel 的 init() 方法源码：</p>
<pre class="lang-java" data-nodeid="1290"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">(Channel channel)</span> </span>{
&nbsp; &nbsp; setChannelOptions(channel, options0().entrySet().toArray(newOptionArray(<span class="hljs-number">0</span>)), logger); <span class="hljs-comment">// 设置 Socket 参数</span>
&nbsp; &nbsp; setAttributes(channel, attrs0().entrySet().toArray(newAttrArray(<span class="hljs-number">0</span>))); <span class="hljs-comment">// 保存用户自定义属性</span>
&nbsp; &nbsp; ChannelPipeline p = channel.pipeline();
&nbsp; &nbsp; <span class="hljs-comment">// 获取 ServerBootstrapAcceptor 的构造参数</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> EventLoopGroup currentChildGroup = childGroup;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelHandler currentChildHandler = childHandler;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Entry&lt;ChannelOption&lt;?&gt;, Object&gt;[] currentChildOptions =
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; childOptions.entrySet().toArray(newOptionArray(<span class="hljs-number">0</span>));
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Entry&lt;AttributeKey&lt;?&gt;, Object&gt;[] currentChildAttrs = childAttrs.entrySet().toArray(newAttrArray(<span class="hljs-number">0</span>));
&nbsp; &nbsp; <span class="hljs-comment">// 添加特殊的 Handler 处理器</span>
&nbsp; &nbsp; p.addLast(<span class="hljs-keyword">new</span> ChannelInitializer&lt;Channel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Channel ch)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelPipeline pipeline = ch.pipeline();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ChannelHandler handler = config.handler();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.addLast(handler);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.eventLoop().execute(<span class="hljs-keyword">new</span> Runnable() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.addLast(<span class="hljs-keyword">new</span> ServerBootstrapAcceptor(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; });
}
</code></pre>
<p data-nodeid="1291">init() 方法的源码比较长，我们依然拆解成两个部分来看：</p>
<p data-nodeid="1292">第一步，设置 Socket 参数以及用户自定义属性。在创建服务端 Channel 时，Channel 的配置参数保存在 NioServerSocketChannelConfig 中，在初始化 Channel 的过程中，Netty 会将这些参数设置到 JDK 底层的 Socket 上，并把用户自定义的属性绑定在 Channel 上。</p>
<p data-nodeid="1293">第二步，添加特殊的 Handler 处理器。首先 ServerBootstrap 为 Pipeline 添加了一个 ChannelInitializer，ChannelInitializer 是实现了 ChannelHandler 接口的匿名类，其中 ChannelInitializer 实现的 initChannel() 方法用于添加 ServerSocketChannel 对应的 Handler。然后 Netty 通过异步 task 的方式又向 Pipeline 一个处理器 ServerBootstrapAcceptor，从 ServerBootstrapAcceptor 的命名可以看出，这是一个连接接入器，专门用于接收新的连接，然后把事件分发给 EventLoop 执行，在这里我们先不做展开。此时服务端的 pipeline 内部结构又发生了变化，如下图所示。</p>
<p data-nodeid="1294"><img src="https://s0.lgstatic.com/i/image/M00/87/94/Ciqc1F_V_y-Adn6LAAKfSPwUO3g505.png" alt="图片1.png" data-nodeid="1416"></p>
<p data-nodeid="1295">思考一个问题，为什么需要 ChannelInitializer 处理器呢？ServerBootstrapAcceptor 的注册过程为什么又需要封装成异步 task 呢？因为我们在初始化时，还没有将 Channel 注册到 Selector 对象上，所以还无法注册 Accept 事件到 Selector 上，所以事先添加了 ChannelInitializer 处理器，等待 Channel 注册完成后，再向 Pipeline 中添加 ServerBootstrapAcceptor 处理器。</p>
<p data-nodeid="1296">服务端 Channel 初始化的过程已经结束了。整体流程比较简单，主要是设置 Socket 参数以及用户自定义属性，并向 Pipeline 中添加了两个特殊的处理器。接下来我们继续分析，如何将初始化好的 Channel 注册到 Selector 对象上？</p>
<h4 data-nodeid="1297">注册服务端 Channel</h4>
<p data-nodeid="1298">回到 initAndRegister() 的主流程，创建完服务端 Channel 之后，继续一层层跟进 register() 方法的源码：</p>
<pre class="lang-java" data-nodeid="1299"><code data-language="java"><span class="hljs-comment">// MultithreadEventLoopGroup#register</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ChannelFuture <span class="hljs-title">register</span><span class="hljs-params">(Channel channel)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> next().register(channel); <span class="hljs-comment">// 选择一个 eventLoop 注册</span>
}
<span class="hljs-comment">// AbstractChannel#register</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">register</span><span class="hljs-params">(EventLoop eventLoop, <span class="hljs-keyword">final</span> ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; AbstractChannel.<span class="hljs-keyword">this</span>.eventLoop = eventLoop;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (eventLoop.inEventLoop()) { <span class="hljs-comment">// Reactor 线程内部调用</span>
&nbsp; &nbsp; &nbsp; &nbsp; register0(promise);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 外部线程调用</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; eventLoop.execute(<span class="hljs-keyword">new</span> Runnable() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; register0(promise);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1300">Netty 会在线程池 EventLoopGroup 中选择一个 EventLoop 与当前 Channel 进行绑定，之后 Channel 生命周期内的所有 I/O 事件都由这个 EventLoop 负责处理，如 accept、connect、read、write 等 I/O 事件。可以看出，不管是 EventLoop 线程本身调用，还是外部线程用，最终都会通过 register0() 方法进行注册：</p>
<pre class="lang-java" data-nodeid="1301"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">register0</span><span class="hljs-params">(ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!promise.setUncancellable() || !ensureOpen(promise)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> firstRegistration = neverRegistered;
&nbsp; &nbsp; &nbsp; &nbsp; doRegister(); <span class="hljs-comment">// 调用 JDK 底层的 register() 进行注册</span>
&nbsp; &nbsp; &nbsp; &nbsp; neverRegistered = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; registered = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; pipeline.invokeHandlerAddedIfNeeded(); <span class="hljs-comment">// 触发 handlerAdded 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; safeSetSuccess(promise);
&nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelRegistered(); <span class="hljs-comment">// 触发 channelRegistered 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 此时 Channel 还未注册绑定地址，所以处于非活跃状态</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isActive()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (firstRegistration) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelActive(); <span class="hljs-comment">// Channel 当前状态为活跃时，触发 channelActive 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (config().isAutoRead()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; beginRead();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1302">register0() 主要做了四件事：调用 JDK 底层进行 Channel 注册、触发 handlerAdded 事件、触发 channelRegistered 事件、Channel 当前状态为活跃时，触发 channelActive 事件。我们对它们逐一进行分析。</p>
<p data-nodeid="1303">首先看下 JDK 底层注册 Channel 的过程，对应 doRegister() 方法的实现逻辑。</p>
<pre class="lang-java" data-nodeid="1304"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doRegister</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> selected = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; selectionKey = javaChannel().register(eventLoop().unwrappedSelector(), <span class="hljs-number">0</span>, <span class="hljs-keyword">this</span>); <span class="hljs-comment">// 调用 JDK 底层的 register() 进行注册</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (CancelledKeyException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> SelectionKey <span class="hljs-title">register</span><span class="hljs-params">(Selector sel, <span class="hljs-keyword">int</span> ops,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Object att)</span>
&nbsp; &nbsp; <span class="hljs-keyword">throws</span> ClosedChannelException
</span>{
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (regLock) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; &nbsp; &nbsp; SelectionKey k = findKey(sel);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (k != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; k.interestOps(ops);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; k.attach(att);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (k == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (keyLock) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!isOpen())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ClosedChannelException();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; k = ((AbstractSelector)sel).register(<span class="hljs-keyword">this</span>, ops, att);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; addKey(k);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> k;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1305">javaChannel().register() 负责调用 JDK 底层，将 Channel 注册到 Selector 上，register() 的第三个入参传入的是 Netty 自己实现的 Channel 对象，调用 register() 方法会将它绑定在 JDK 底层 Channel 的 attachment 上。这样在每次 Selector 对象进行事件循环时，Netty 都可以从返回的 JDK 底层 Channel 中获得自己的 Channel 对象。</p>
<p data-nodeid="1306">完成 Channel 向 Selector 注册后，接下来就会触发 Pipeline 一系列的事件传播。在事件传播之前，用户自定义的业务处理器是如何被添加到 Pipeline 中的呢？答案就在pipeline.invokeHandlerAddedIfNeeded() 当中，我们重点看下 handlerAdded 事件的处理过程。invokeHandlerAddedIfNeeded() 方法的调用层次比较深，推荐你结合上述 Echo 服务端示例，使用 IDE Debug 的方式跟踪调用栈，如下图所示。</p>
<p data-nodeid="1307"><img src="https://s0.lgstatic.com/i/image/M00/87/95/Ciqc1F_V_2-AGYmLAAu7ct-oDd4075.png" alt="图片2.png" data-nodeid="1428"></p>
<p data-nodeid="1308">我们首先抓住 ChannelInitializer 中的核心源码，逐层进行分析。</p>
<pre class="lang-java" data-nodeid="1309"><code data-language="java"><span class="hljs-comment">// ChannelInitializer</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerAdded</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ctx.channel().isRegistered()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (initChannel(ctx)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; removeState(ctx);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (initMap.add(ctx)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; initChannel((C) ctx.channel()); <span class="hljs-comment">// 调用 ChannelInitializer 实现的 initChannel() 方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable cause) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; exceptionCaught(ctx, cause);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ChannelPipeline pipeline = ctx.pipeline();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (pipeline.context(<span class="hljs-keyword">this</span>) != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.remove(<span class="hljs-keyword">this</span>); <span class="hljs-comment">// 将 ChannelInitializer 自身从 Pipeline 中移出</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
</code></pre>
<p data-nodeid="1310">可以看出 ChannelInitializer 首先会调用 initChannel() 抽象方法，然后 Netty 会把 ChannelInitializer 自身从 Pipeline 移出。其中 initChannel() 抽象方法是在哪里实现的呢？这就要跟踪到 ServerBootstrap 之前的 init() 方法，其中有这么一段代码：</p>
<pre class="lang-java" data-nodeid="1311"><code data-language="java">p.addLast(<span class="hljs-keyword">new</span> ChannelInitializer&lt;Channel&gt;() {
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Channel ch)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelPipeline pipeline = ch.pipeline();
&nbsp; &nbsp; &nbsp; &nbsp; ChannelHandler handler = config.handler();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.addLast(handler);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; ch.eventLoop().execute(<span class="hljs-keyword">new</span> Runnable() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.addLast(<span class="hljs-keyword">new</span> ServerBootstrapAcceptor(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch, currentChildGroup, currentChildHandler, currentChildOptions, currentChildAttrs));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
});
</code></pre>
<p data-nodeid="1312">在前面我们已经分析了 initChannel() 方法的实现逻辑，首先向 Pipeline 中添加 ServerSocketChannel 对应的 Handler，然后通过异步 task 的方式向 Pipeline 添加 ServerBootstrapAcceptor 处理器。其中有一个点不要混淆，handler() 方法是添加到服务端的Pipeline 上，而 childHandler() 方法是添加到客户端的 Pipeline 上。所以对应 Echo 服务器示例中，此时被添加的是 LoggingHandler 处理器。</p>
<p data-nodeid="1313">因为添加 ServerBootstrapAcceptor 是一个异步过程，需要 EventLoop 线程负责执行。而当前 EventLoop 线程正在执行 register0() 的注册流程，所以等到 register0() 执行完之后才能被添加到 Pipeline 当中。完成 initChannel() 这一步之后，ServerBootstrapAcceptor 并没有被添加到 Pipeline 中，此时 Pipeline 的内部结构变化如下图所示。<br>
<img src="https://s0.lgstatic.com/i/image/M00/87/95/Ciqc1F_V_5eAUGuoAAJsI9pISJU272.png" alt="图片3.png" data-nodeid="1436"></p>
<p data-nodeid="1314">我们回到 register0() 的主流程，接着向下分析。channelRegistered 事件是由 fireChannelRegistered() 方法触发，沿着 Pipeline 的 Head 节点传播到 Tail 节点，并依次调用每个 ChannelHandler 的 channelRegistered() 方法。然而此时 Channel 还未注册绑定地址，所以处于非活跃状态，所以并不会触发 channelActive 事件。</p>
<p data-nodeid="1315">执行完整个 register0() 的注册流程之后，EventLoop 线程会将 ServerBootstrapAcceptor 添加到 Pipeline 当中，此时 Pipeline 的内部结构又发生了变化，如下图所示。<br>
<img src="https://s0.lgstatic.com/i/image/M00/87/95/Ciqc1F_V_7SAUkiOAAI_yPZd0NI396.png" alt="图片4.png" data-nodeid="1442"></p>
<p data-nodeid="1316">整个服务端 Channel 注册的流程我们已经讲完，注册过程中 Pipeline 结构的变化值得你再反复梳理，从而加深理解。目前服务端还是不能工作的，还差最后一步就是进行端口绑定，我们继续向下分析。</p>
<h4 data-nodeid="1317">端口绑定</h4>
<p data-nodeid="1318">回到 ServerBootstrap 的 bind() 方法，我们继续跟进端口绑定 doBind0() 的源码。</p>
<pre class="lang-java" data-nodeid="1319"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">bind</span><span class="hljs-params">(<span class="hljs-keyword">final</span> SocketAddress localAddress, <span class="hljs-keyword">final</span> ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; assertEventLoop();
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> wasActive = isActive();
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; doBind(localAddress); <span class="hljs-comment">// 调用 JDK 底层进行端口绑定</span>
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; safeSetFailure(promise, t);
&nbsp; &nbsp; &nbsp; &nbsp; closeIfClosed();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!wasActive &amp;&amp; isActive()) {
&nbsp; &nbsp; &nbsp; &nbsp; invokeLater(<span class="hljs-keyword">new</span> Runnable() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelActive(); <span class="hljs-comment">// 触发 channelActive 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
&nbsp; &nbsp; safeSetSuccess(promise);
}
</code></pre>
<p data-nodeid="1320">bind() 方法主要做了两件事，分别为调用 JDK 底层进行端口绑定；绑定成功后并触发 channelActive 事件。下面我们逐一进行分析。</p>
<p data-nodeid="1321">首先看下调用 JDK 底层进行端口绑定的 doBind() 方法：</p>
<pre class="lang-java" data-nodeid="1322"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doBind</span><span class="hljs-params">(SocketAddress localAddress)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (PlatformDependent.javaVersion() &gt;= <span class="hljs-number">7</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; javaChannel().bind(localAddress, config.getBacklog());
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; javaChannel().socket().bind(localAddress, config.getBacklog());
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1323">Netty 会根据 JDK 版本的不同，分别调用 JDK 底层不同的 bind() 方法。我使用的是 JDK8，所以会调用 JDK 原生 Channel 的 bind() 方法。执行完 doBind() 之后，服务端 JDK 原生的 Channel 真正已经完成端口绑定了。</p>
<p data-nodeid="1324">完成端口绑定之后，Channel 处于活跃 Active 状态，然后会调用 pipeline.fireChannelActive() 方法触发 channelActive 事件。我们可以一层层跟进 fireChannelActive() 方法，发现其中比较重要的部分：</p>
<pre class="lang-java" data-nodeid="1325"><code data-language="java"><span class="hljs-comment">// DefaultChannelPipeline#channelActive</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelActive</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> </span>{
&nbsp; &nbsp; ctx.fireChannelActive();
&nbsp; &nbsp; readIfIsAutoRead();
}
<span class="hljs-comment">// AbstractNioChannel#doBeginRead</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doBeginRead</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-comment">// Channel.read() or ChannelHandlerContext.read() was called</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> SelectionKey selectionKey = <span class="hljs-keyword">this</span>.selectionKey;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!selectionKey.isValid()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; readPending = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> interestOps = selectionKey.interestOps();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> ((interestOps &amp; readInterestOp) == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; selectionKey.interestOps(interestOps | readInterestOp); <span class="hljs-comment">// 注册 OP_ACCEPT 事件到服务端 Channel 的事件集合</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1326">可以看出，在执行完 channelActive 事件传播之后，会调用 readIfIsAutoRead() 方法触发 Channel 的 read 事件，而它最终调用到 AbstractNioChannel 中的 doBeginRead() 方法，其中 readInterestOp 参数就是在前面初始化 Channel 所传入的 SelectionKey.OP_ACCEPT 事件，所以 OP_ACCEPT 事件会被注册到 Channel 的事件集合中。</p>
<p data-nodeid="1327">到此为止，整个服务端已经真正启动完毕。我们总结一下服务端启动的全流程，如下图所示。<br>
<img src="https://s0.lgstatic.com/i/image/M00/87/A1/CgqCHl_V_8-AFYYMAAHvZ2ePhWo232.png" alt="图片5.png" data-nodeid="1459"></p>
<ul data-nodeid="1328">
<li data-nodeid="1329">
<p data-nodeid="1330"><strong data-nodeid="1464">创建服务端 Channel</strong>：本质是创建 JDK 底层原生的 Channel，并初始化几个重要的属性，包括 id、unsafe、pipeline 等。</p>
</li>
<li data-nodeid="1331">
<p data-nodeid="1332"><strong data-nodeid="1469">初始化服务端 Channel</strong>：设置 Socket 参数以及用户自定义属性，并添加两个特殊的处理器 ChannelInitializer 和 ServerBootstrapAcceptor。</p>
</li>
<li data-nodeid="1333">
<p data-nodeid="1334"><strong data-nodeid="1474">注册服务端 Channel</strong>：调用 JDK 底层将 Channel 注册到 Selector 上。</p>
</li>
<li data-nodeid="1335">
<p data-nodeid="1336"><strong data-nodeid="1481">端口绑定</strong>：调用 JDK 底层进行端口绑定，并触发 channelActive 事件，把 OP_ACCEPT 事件注册到 Channel 的事件集合中。</p>
</li>
</ul>
<h3 data-nodeid="1337">加餐：服务端如何处理客户端新建连接</h3>
<p data-nodeid="1338">Netty 服务端完全启动后，就可以对外工作了。接下来 Netty 服务端是如何处理客户端新建连接的呢？主要分为四步：</p>
<ol data-nodeid="1339">
<li data-nodeid="1340">
<p data-nodeid="1341">Boss NioEventLoop 线程轮询客户端新连接 OP_ACCEPT 事件；</p>
</li>
<li data-nodeid="1342">
<p data-nodeid="1343">构造 Netty 客户端 NioSocketChannel；</p>
</li>
<li data-nodeid="1344">
<p data-nodeid="1345">注册 Netty 客户端 NioSocketChannel 到 Worker 工作线程中；</p>
</li>
<li data-nodeid="1346">
<p data-nodeid="1347">注册 OP_READ 事件到 NioSocketChannel 的事件集合。</p>
</li>
</ol>
<p data-nodeid="1348">下面我们对每个步骤逐一进行简单的介绍。</p>
<p data-nodeid="1349">Netty 中 Boss NioEventLoop 专门负责接收新的连接，关于 NioEventLoop 的核心源码我们下节课会着重介绍，在这里我们只先了解基本的处理流程。当客户端有新连接接入服务端时，Boss NioEventLoop 会监听到 OP_ACCEPT 事件，源码如下所示：</p>
<pre class="lang-java" data-nodeid="1350"><code data-language="java"><span class="hljs-comment">// NioEventLoop#processSelectedKey</span>
<span class="hljs-keyword">if</span> ((readyOps &amp; (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != <span class="hljs-number">0</span> || readyOps == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; unsafe.read();
}
</code></pre>
<p data-nodeid="1351">NioServerSocketChannel 所持有的 unsafe 是 NioMessageUnsafe 类型，我们看下 NioMessageUnsafe.read() 方法中做了什么事。</p>
<pre class="lang-java" data-nodeid="1352"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">read</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">assert</span> <span class="hljs-title">eventLoop</span><span class="hljs-params">()</span>.<span class="hljs-title">inEventLoop</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelConfig config = config();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelPipeline pipeline = pipeline();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> RecvByteBufAllocator.Handle allocHandle = unsafe().recvBufAllocHandle();&nbsp;
&nbsp; &nbsp; allocHandle.reset(config);
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> closed = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; Throwable exception = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> localRead = doReadMessages(readBuf);&nbsp; <span class="hljs-comment">// while 循环不断读取 Buffer 中的数据</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (localRead == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (localRead &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; closed = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; allocHandle.incMessagesRead(localRead);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">while</span> (allocHandle.continueReading());
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; exception = t;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> size = readBuf.size();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; size; i ++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; readPending = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelRead(readBuf.get(i)); <span class="hljs-comment">// 传播读取事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; readBuf.clear();
&nbsp; &nbsp; &nbsp; &nbsp; allocHandle.readComplete();
&nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelReadComplete(); <span class="hljs-comment">// 传播读取完毕事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!readPending &amp;&amp; !config.isAutoRead()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; removeReadOp();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1353">可以看出 read() 方法的核心逻辑就是通过 while 循环不断读取数据，然后放入 List 中，这里的数据其实就是新连接。需要重点跟进一下 NioServerSocketChannel 的 doReadMessages() 方法。</p>
<pre class="lang-java" data-nodeid="1354"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">int</span> <span class="hljs-title">doReadMessages</span><span class="hljs-params">(List&lt;Object&gt; buf)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; SocketChannel ch = SocketUtils.accept(javaChannel());
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ch != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; buf.add(<span class="hljs-keyword">new</span> NioSocketChannel(<span class="hljs-keyword">this</span>, ch));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-number">1</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; logger.warn(<span class="hljs-string">"Failed to create a new channel from an accepted socket."</span>, t);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.close();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t2) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.warn(<span class="hljs-string">"Failed to close a socket."</span>, t2);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
}
</code></pre>
<p data-nodeid="1355">这时就开始执行第二个步骤：构造 Netty 客户端 NioSocketChannel。Netty 先通过 JDK 底层的 accept() 获取 JDK 原生的 SocketChannel，然后将它封装成 Netty 自己的 NioSocketChannel。新建 Netty 的客户端 Channel 的实现原理与上文中我们讲到的创建服务端 Channel 的过程是类似的，只是服务端 Channel 的类型是 NioServerSocketChannel，而客户端 Channel 的类型是 NioSocketChannel。NioSocketChannel 的创建同样会完成几件事：创建核心成员变量 id、unsafe、pipeline；注册 SelectionKey.OP_READ 事件；设置 Channel 的为非阻塞模式；新建客户端 Channel 的配置。</p>
<p data-nodeid="1356">成功构造客户端 NioSocketChannel 后，接下来会通过 pipeline.fireChannelRead() 触发 channelRead 事件传播。对于服务端来说，此时 Pipeline 的内部结构如下图所示。<br>
<img src="https://s0.lgstatic.com/i/image/M00/87/A1/CgqCHl_V__eAOXbQAALQ0Dt6N64798.png" alt="图片6.png" data-nodeid="1505"></p>
<p data-nodeid="1357">上文中我们提到了一种特殊的处理器 ServerBootstrapAcceptor，在这里它就发挥了重要的作用。channelRead 事件会传播到 ServerBootstrapAcceptor.channelRead() 方法，channelRead() 会将客户端 Channel 分配到工作线程组中去执行。具体实现如下：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="3348"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Channel child = (Channel) msg;
&nbsp; &nbsp; <span class="hljs-comment">// 在客户端 Channel 中添加 childHandler，childHandler 是用户在启动类中通过 childHandler() 方法指定的</span>
&nbsp; &nbsp; child.pipeline().addLast(childHandler);
&nbsp; &nbsp; setChannelOptions(child, childOptions, logger);
&nbsp; &nbsp; setAttributes(child, childAttrs);
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 注册客户端 Channel</span>
&nbsp; &nbsp; &nbsp; &nbsp; childGroup.register(child).addListener(<span class="hljs-keyword">new</span> ChannelFutureListener() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">operationComplete</span><span class="hljs-params">(ChannelFuture future)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!future.isSuccess()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; forceClose(child, future.cause());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; forceClose(child, t);
&nbsp; &nbsp; }
}
</code></pre>




<p data-nodeid="1359">ServerBootstrapAcceptor 开始就把 msg 强制转换为 Channel。难道不会有其他类型的数据吗？因为 ServerBootstrapAcceptor 是服务端 Channel 中一个特殊的处理器，而服务端 Channel 的 channelRead 事件只会在新连接接入时触发，所以这里拿到的数据都是客户端新连接。</p>
<p data-nodeid="1360">ServerBootstrapAcceptor 通过 childGroup.register() 方法会完成第三和第四两个步骤，将 NioSocketChannel 注册到 Worker 工作线程中，并注册 OP_READ 事件到 NioSocketChannel 的事件集合。在注册过程中比较有意思的一点是，它会调用 pipeline.fireChannelRegistered() 方法传播 channelRegistered 事件，然后再调用 pipeline.fireChannelActive() 方法传播 channelActive 事件。兜了一圈，这又会回到之前我们介绍的 readIfIsAutoRead() 方法，此时它会将 SelectionKey.OP_READ 事件注册到 Channel 的事件集合。</p>
<p data-nodeid="1361">关于服务端如何处理客户端新建连接的具体源码，我在此就不继续展开了。这里留一个小任务，建议你亲自动手分析下 childGroup.register() 的相关源码，从而加深对服务端启动以及新连接处理流程的理解。有了服务端启动源码分析的基础，再去理解客户端新建连接的过程会相对容易很多。</p>
<h3 data-nodeid="1362">总结</h3>
<p data-nodeid="1363">本节课我们深入分析了 Netty 服务端启动的全流程，对其中涉及的核心组件有了基本的认识。Netty 服务端启动的相关源码层次比较深，推荐大家在读源码的时候，可以先把主体流程梳理清楚，开始时先不用纠结具体的方法是用来做什么，自顶而下先画出完整的调用链路图（如下图所示），然后再逐一击破。<br>
<img src="https://s0.lgstatic.com/i/image/M00/87/A2/CgqCHl_WABCAJA9VAAIG0Ncq3hs061.png" alt="图片7.png" data-nodeid="1519"></p>
<p data-nodeid="1364" class="">下节课，我们将学习 Netty 最核心的 Reactor 线程模型的源码，推荐你把两节课放在一起再进行复习，可以解答你目前不少的疑问，如异步 task 是如何封装并执行的？事件注册之后是如何被处理的？</p>

---

### 精选评论

##### *开：
> 不错！

##### **杰：
> 很清晰，感谢

##### Q：
> 再确认了一遍，在「注册服务端 Channel」这个小节应该欠缺了一块内容，实际上在当前执行register操作的时候，线程是main主线程，并不会直接执行register0方法，而是执行eventloop的execute方法（这需要到SingleThreadEventExecutor）类中把当前的register0当做一个非IO的task丢到taskQueue中，并且创建一个线程交由executor执行该线程，此时该线程才会从taskQueue中获取当前的注册任务进行进一步的工作应该有入口进行线程的创建和启动，再处理丢给线程的任务的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果当前 Reactor 线程，会直接调用 register0，如果不是会封装成异步 task。你学习的很认真，源码分析我没办法做到每个函数都面面俱到，还是要自己动手去看比较有效果。

##### Q：
> 来回的看了好几遍，再配合基本的Nio工作流程去理解，终于是大致了解了本小结讲解的内容了。也发现了之前自己理解和认识上的一些错误。另外这里面大量的future和promise，能深入的介绍一下这两个玩意也挺重要的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; future和promise确实是非常重要的组件，是非常值得研究源码的。这次专栏里我没有列入，有了专栏学习的基础再去研究 future和promise 的源码应该也会轻松一些。

##### **威：
> 觉得受益匪浅，谢谢老师

