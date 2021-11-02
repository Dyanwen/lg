<p data-nodeid="934" class="">通过前面两节源码课程的学习，我们知道 Netty 在服务端启动时会为创建 NioServerSocketChannel，当客户端新连接接入时又会创建 NioSocketChannel，不管是服务端还是客户端 Channel，在创建时都会初始化自己的 ChannelPipeline。如果把 Netty 比作成一个生产车间，那么 Reactor 线程无疑是车间的中央管控系统，ChannelPipeline 可以看作是车间的流水线，将原材料按顺序进行一步步加工，然后形成一个完整的产品。本节课我将带你完整梳理一遍网络请求在 Netty 中的处理流程，从而加深对前两节课内容的理解，并着重讲解 ChannelPipeline 的工作原理。</p>
<blockquote data-nodeid="935">
<p data-nodeid="936">说明：本文参考的 Netty 源码版本为 4.1.42.Final。</p>
</blockquote>
<h3 data-nodeid="937">事件处理机制回顾</h3>
<p data-nodeid="938">首先我们以服务端接入客户端新连接为例，并结合前两节源码课学习的知识点，一起复习下 Netty 的事件处理流程，如下图所示。</p>
<p data-nodeid="939"><img src="https://s0.lgstatic.com/i/image/M00/8A/4B/CgqCHl_Zyq-ALQoLAA9q2qihg8Q151.png" alt="Drawing 0.png" data-nodeid="1097"></p>
<p data-nodeid="940">Netty 服务端启动后，BossEventLoopGroup 会负责监听客户端的 Accept 事件。当有客户端新连接接入时，BossEventLoopGroup 中的 NioEventLoop 首先会新建客户端 Channel，然后在 NioServerSocketChannel 中触发 channelRead 事件传播，NioServerSocketChannel 中包含了一种特殊的处理器 ServerBootstrapAcceptor，最终通过 ServerBootstrapAcceptor 的 channelRead() 方法将新建的客户端 Channel 分配到 WorkerEventLoopGroup 中。WorkerEventLoopGroup 中包含多个 NioEventLoop，它会选择其中一个 NioEventLoop 与新建的客户端 Channel 绑定。</p>
<p data-nodeid="941">完成客户端连接注册之后，就可以接收客户端的请求数据了。当客户端向服务端发送数据时，NioEventLoop 会监听到 OP_READ 事件，然后分配 ByteBuf 并读取数据，读取完成后将数据传递给 Pipeline 进行处理。一般来说，数据会从 ChannelPipeline 的第一个 ChannelHandler 开始传播，将加工处理后的消息传递给下一个 ChannelHandler，整个过程是串行化执行。</p>
<p data-nodeid="942">在前面两节课中，我们介绍了服务端如何接收客户端新连接，以及 NioEventLoop 的工作流程，接下来我们重点介绍 ChannelPipeline 是如何实现 Netty 事件驱动的，这样 Netty 整个事件处理流程已经可以串成一条主线。</p>
<h4 data-nodeid="943">Pipeline 的初始化</h4>
<p data-nodeid="944">我们知道 ChannelPipeline 是在创建 Channel 时被创建的，它是 Channel 中非常重要的一个成员变量。回到 AbstractChannel 的构造函数，以此为切入点，我们一起看下 ChannelPipeline 是如何一步步被构造出来的。</p>
<pre class="lang-java" data-nodeid="945"><code data-language="java"><span class="hljs-comment">// AbstractChannel</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">AbstractChannel</span><span class="hljs-params">(Channel parent)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.parent = parent;
&nbsp; &nbsp; id = newId();
&nbsp; &nbsp; unsafe = newUnsafe();
&nbsp; &nbsp; pipeline = newChannelPipeline();
}
<span class="hljs-comment">// AbstractChannel#newChannelPipeline</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> DefaultChannelPipeline <span class="hljs-title">newChannelPipeline</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DefaultChannelPipeline(<span class="hljs-keyword">this</span>);
}
<span class="hljs-comment">// DefaultChannelPipeline</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">DefaultChannelPipeline</span><span class="hljs-params">(Channel channel)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.channel = ObjectUtil.checkNotNull(channel, <span class="hljs-string">"channel"</span>);
&nbsp; &nbsp; succeededFuture = <span class="hljs-keyword">new</span> SucceededChannelFuture(channel, <span class="hljs-keyword">null</span>);
&nbsp; &nbsp; voidPromise =&nbsp; <span class="hljs-keyword">new</span> VoidChannelPromise(channel, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; tail = <span class="hljs-keyword">new</span> TailContext(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; head = <span class="hljs-keyword">new</span> HeadContext(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; head.next = tail;
&nbsp; &nbsp; tail.prev = head;
}
</code></pre>
<p data-nodeid="946">当 ChannelPipeline 初始化完成后，会构成一个由 ChannelHandlerContext 对象组成的双向链表，默认 ChannelPipeline 初始化状态的最小结构仅包含 HeadContext 和 TailContext 两个节点，如下图所示。</p>
<p data-nodeid="947"><img src="https://s0.lgstatic.com/i/image/M00/8A/40/Ciqc1F_ZyruACytKAAOEkLnjkRc999.png" alt="Drawing 1.png" data-nodeid="1108"></p>
<p data-nodeid="948">HeadContext 和 TailContext 属于 ChannelPipeline 中两个特殊的节点，它们都继承自 AbstractChannelHandlerContext，根据源码看下 AbstractChannelHandlerContext 有哪些实现类，如下图所示。除了 HeadContext 和 TailContext，还有一个默认实现类 DefaultChannelHandlerContext，我们可以猜到 DefaultChannelHandlerContext 封装的是用户在 Netty 启动配置类中添加的自定义业务处理器，DefaultChannelHandlerContext 会插入到 HeadContext 和 TailContext 之间。</p>
<p data-nodeid="1245" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/B9/Ciqc1F_fiOeALI8eAAVUA-uqBu0074.png" alt="图片3.png" data-nodeid="1248"></p>

<p data-nodeid="950">接着我们比较一下上述三种 AbstractChannelHandlerContext 实现类的内部结构，发现它们都包含当前 ChannelPipeline 的引用、处理器 ChannelHandler。有一点不同的是 HeadContext 节点还包含了用于操作底层数据读写的 unsafe 对象。对于 Inbound 事件，会先从 HeadContext 节点开始传播，所以 unsafe 可以看作是 Inbound 事件的发起者；对于 Outbound 事件，数据最后又会经过 HeadContext 节点返回给客户端，此时 unsafe 可以看作是 Outbound 事件的处理者。</p>
<p data-nodeid="951">接下来我们继续看下用户自定义的处理器是如何加入 ChannelPipeline 的双向链表的。</p>
<h4 data-nodeid="952">Pipeline 添加 Handler</h4>
<p data-nodeid="953">在 Netty 客户端或者服务端启动时，就需要用户配置自定义实现的业务处理器。我们先看一段服务端启动类的代码片段：</p>
<pre class="lang-java" data-nodeid="954"><code data-language="java">ServerBootstrap b = <span class="hljs-keyword">new</span> ServerBootstrap();
b.group(bossGroup, workerGroup)
&nbsp; &nbsp; &nbsp; &nbsp; .channel(NioServerSocketChannel.class)
&nbsp; &nbsp; &nbsp; &nbsp; .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> SampleInboundA());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> SampleInboundB());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> SampleOutboundA());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> SampleOutboundB());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
</code></pre>
<p data-nodeid="955">我们知道 ChannelPipeline 分为入站 ChannelInboundHandler 和出站 ChannelOutboundHandler 两种处理器，它们都会被 ChannelHandlerContext 封装，不管是哪种处理器，最终都是通过双向链表连接，代码示例中构成的 ChannelPipeline 的结构如下。</p>
<p data-nodeid="1871" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/9C/CgpVE1_fiPGASYaAAAJ5IvplmPM854.png" alt="图片4.png" data-nodeid="1874"></p>

<p data-nodeid="957">那么 ChannelPipeline 在添加 Handler 时是如何区分 Inbound 和 Outbound 类型的呢？我们一起跟进 ch.pipeline().addLast() 方法源码，定位到核心代码如下。</p>
<pre class="lang-java" data-nodeid="958"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelPipeline <span class="hljs-title">addLast</span><span class="hljs-params">(EventExecutorGroup group, String name, ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> AbstractChannelHandlerContext newCtx;
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 1. 检查是否重复添加 Handler</span>
&nbsp; &nbsp; &nbsp; &nbsp; checkMultiplicity(handler);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 2. 创建新的 DefaultChannelHandlerContext 节点</span>
&nbsp; &nbsp; &nbsp; &nbsp; newCtx = newContext(group, filterName(name, handler), handler);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 3. 添加新的 DefaultChannelHandlerContext 节点到 ChannelPipeline</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;addLast0(newCtx);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 4. 回调用户方法</span>
&nbsp; &nbsp; callHandlerAdded0(newCtx);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
</code></pre>
<p data-nodeid="959">addLast() 主要做了以下四件事：</p>
<ol data-nodeid="960">
<li data-nodeid="961">
<p data-nodeid="962">检查是否重复添加 Handler。</p>
</li>
<li data-nodeid="963">
<p data-nodeid="964">创建新的 DefaultChannelHandlerContext 节点。</p>
</li>
<li data-nodeid="965">
<p data-nodeid="966">添加新的 DefaultChannelHandlerContext 节点到 ChannelPipeline。</p>
</li>
<li data-nodeid="967">
<p data-nodeid="968">回调用户方法。</p>
</li>
</ol>
<p data-nodeid="969">前三个步骤通过 synchronized 加锁完成的，为了防止多线程并发操作 ChannelPipeline 底层双向链表。下面我们一步步进行拆解介绍。</p>
<p data-nodeid="970">首先在添加 Handler 时，ChannelPipeline 会检查该 Handler 有没有被添加过。如果一个非线程安全的 Handler 被添加到 ChannelPipeline 中，那么当多线程访问时会造成线程安全问题。Netty 具体检查重复性的逻辑由 checkMultiplicity() 方法实现：</p>
<pre class="lang-java" data-nodeid="971"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">checkMultiplicity</span><span class="hljs-params">(ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler <span class="hljs-keyword">instanceof</span> ChannelHandlerAdapter) {
&nbsp; &nbsp; &nbsp; &nbsp; ChannelHandlerAdapter h = (ChannelHandlerAdapter) handler;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!h.isSharable() &amp;&amp; h.added) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ChannelPipelineException(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; h.getClass().getName() +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">" is not a @Sharable handler, so can't be added or removed multiple times."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; h.added = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="972">用户自定义实现的处理一般都继承于 ChannelHandlerAdapter，ChannelHandlerAdapter 中使用 added 变量标识该 Handler 是否被添加过。如果当前添加的 Handler 是非共享且已被添加过，那么就会抛出异常，否则将当前 Handler 标记为已添加。</p>
<p data-nodeid="973">h.isSharable() 用于判断 Handler 是否是共享的，所谓共享就是这个 Handler 可以被重复添加到不同的 ChannelPipeline 中，共享的 Handler 必须要确保是线程安全的。如果我们想实现一个共享的 Handler，只需要在 Handler 中添加 @Sharable 注解即可，如下所示：</p>
<pre class="lang-java" data-nodeid="974"><code data-language="java"><span class="hljs-meta">@ChannelHandler</span>.Sharable
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SampleInBoundHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelInboundHandlerAdapter</span> </span>{}
</code></pre>
<p data-nodeid="975">接下来我们分析 addLast() 的第二步，创建新的 DefaultChannelHandlerContext 节点。在执行 newContext() 方法之前，会通过 filterName() 为 Handler 创建一个唯一的名称，一起先看下 Netty 生成名称的策略是怎样的。</p>
<pre class="lang-java" data-nodeid="976"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">filterName</span><span class="hljs-params">(String name, ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (name == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> generateName(handler);
&nbsp; &nbsp; }
&nbsp; &nbsp; checkDuplicateName(name);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> name;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">generateName</span><span class="hljs-params">(ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; Map&lt;Class&lt;?&gt;, String&gt; cache = nameCaches.get();
&nbsp; &nbsp; Class&lt;?&gt; handlerType = handler.getClass();
&nbsp; &nbsp; String name = cache.get(handlerType);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (name == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; name = generateName0(handlerType);
&nbsp; &nbsp; &nbsp; &nbsp; cache.put(handlerType, name);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (context0(name) != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; String baseName = name.substring(<span class="hljs-number">0</span>, name.length() - <span class="hljs-number">1</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">1</span>;; i ++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String newName = baseName + i;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (context0(newName) == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; name = newName;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> name;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> String <span class="hljs-title">generateName0</span><span class="hljs-params">(Class&lt;?&gt; handlerType)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> StringUtil.simpleClassName(handlerType) + <span class="hljs-string">"#0"</span>;
}
</code></pre>
<p data-nodeid="977">Netty 会使用 FastThreadLocal 缓存 Handler 和名称的映射关系，在为 Handler 生成默认名称的之前，会先从缓存中查找是否已经存在，如果不存在，会调用 generateName0() 方法生成默认名称后，并加入缓存。可以看出 Netty 生成名称的默认规则是 “简单类名#0”，例如 HeadContext 的默认名称为 “DefaultChannelPipeline$HeadContext#0”。</p>
<p data-nodeid="3121" class="">为 Handler 生成完默认名称之后，还会通过 context0() 方法检查生成的名称是否和 ChannelPipeline 已有的名称出现冲突，查重的过程很简单，就是对双向链表进行线性搜索。如果存在冲突现象，Netty 会将名称最后的序列号截取出来，一直递增直至生成不冲突的名称为止，例如 “简单类名#1” “简单类名#2” “简单类名#3” 等等。</p>


<p data-nodeid="979">接下来回到 newContext() 创建节点的流程，可以定位到 AbstractChannelHandlerContext 的构造函数：</p>
<pre class="lang-java" data-nodeid="980"><code data-language="java">AbstractChannelHandlerContext(DefaultChannelPipeline pipeline, EventExecutor executor,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String name, Class&lt;? extends ChannelHandler&gt; handlerClass) {
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.name = ObjectUtil.checkNotNull(name, <span class="hljs-string">"name"</span>);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.pipeline = pipeline;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.executor = executor;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.executionMask = mask(handlerClass);
&nbsp; &nbsp; ordered = executor == <span class="hljs-keyword">null</span> || executor <span class="hljs-keyword">instanceof</span> OrderedEventExecutor;
}
</code></pre>
<p data-nodeid="981">AbstractChannelHandlerContext 中有一个 executionMask 属性并不是很好理解，它其实是一种常用的掩码运算操作，看下 mask() 方法是如何生成掩码的呢？</p>
<pre class="lang-java" data-nodeid="982"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> <span class="hljs-title">mask0</span><span class="hljs-params">(Class&lt;? extends ChannelHandler&gt; handlerType)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> mask = MASK_EXCEPTION_CAUGHT;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ChannelInboundHandler.class.isAssignableFrom(handlerType)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 如果是 ChannelInboundHandler 实例，所有 Inbound 事件置为 <span class="hljs-number">1</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask |= MASK_ALL_INBOUND;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 排除 Handler 不感兴趣的 Inbound 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, <span class="hljs-string">"channelRegistered"</span>, ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_REGISTERED;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelUnregistered", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_UNREGISTERED;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelActive", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_ACTIVE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelInactive", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_INACTIVE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelRead", ChannelHandlerContext.class, Object.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_READ;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelReadComplete", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_READ_COMPLETE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "channelWritabilityChanged", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CHANNEL_WRITABILITY_CHANGED;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "userEventTriggered", ChannelHandlerContext.class, Object.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_USER_EVENT_TRIGGERED;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ChannelOutboundHandler.class.isAssignableFrom(handlerType)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 如果是 ChannelOutboundHandler 实例，所有 Outbound 事件置为 <span class="hljs-number">1</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask |= MASK_ALL_OUTBOUND;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 排除 Handler 不感兴趣的 Outbound 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, <span class="hljs-string">"bind"</span>, ChannelHandlerContext.class,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SocketAddress.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_BIND;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "connect", ChannelHandlerContext.class, SocketAddress.class,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; SocketAddress.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CONNECT;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "disconnect", ChannelHandlerContext.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_DISCONNECT;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "close", ChannelHandlerContext.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_CLOSE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "deregister", ChannelHandlerContext.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_DEREGISTER;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "read", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_READ;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "write", ChannelHandlerContext.class,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Object.class, ChannelPromise.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_WRITE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "flush", ChannelHandlerContext.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_FLUSH;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSkippable(handlerType, "exceptionCaught", ChannelHandlerContext.class, Throwable.class)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mask &amp;= ~MASK_EXCEPTION_CAUGHT;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; &nbsp; &nbsp; PlatformDependent.throwException(e);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> mask;
}
</code></pre>
<p data-nodeid="983">Netty 中分别有多种 Inbound 事件和 Outbound 事件，如 Inbound 事件有 channelRegistered、channelActive、channelRead 等等。Netty 会判断 Handler 的类型是否是 ChannelInboundHandler 的实例，如果是会把所有 Inbound 事件先置为 1，然后排除 Handler 不感兴趣的方法。同理，Handler 类型如果是 ChannelOutboundHandler，也是这么实现的。</p>
<p data-nodeid="984">那么如何排除 Handler 不感兴趣的事件呢？Handler 对应事件的方法上如果有 @Skip 注解，Netty 认为该事件是需要排除的。大部分情况下，用户自定义实现的 Handler 只需要关心个别事件，那么剩余不关心的方法都需要加上 @Skip 注解吗？Netty 其实已经在 ChannelHandlerAdapter 中默认都添加好了，所以用户如果继承了 ChannelHandlerAdapter，默认没有重写的方法都是加上 @Skip 的，只有用户重写的方法才是 Handler 关心的事件。</p>
<p data-nodeid="985">回到 addLast() 的主流程，接着需要将新创建的 DefaultChannelHandlerContext 节点添加到 ChannelPipeline 中，跟进 addLast0() 方法的源码。</p>
<pre class="lang-java" data-nodeid="986"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addLast0</span><span class="hljs-params">(AbstractChannelHandlerContext newCtx)</span> </span>{
&nbsp; &nbsp; AbstractChannelHandlerContext prev = tail.prev;
&nbsp; &nbsp; newCtx.prev = prev;
&nbsp; &nbsp; newCtx.next = tail;
&nbsp; &nbsp; prev.next = newCtx;
&nbsp; &nbsp; tail.prev = newCtx;
}
</code></pre>
<p data-nodeid="987">addLast0() 非常简单，就是向 ChannelPipeline 中双向链表的尾部插入新的节点，其中 HeadContext 和 TailContext 一直是链表的头和尾，新的节点被插入到 HeadContext 和 TailContext 之间。例如代码示例中 SampleOutboundA 被添加时，双向链表的结构变化如下所示。</p>
<p data-nodeid="988"><img src="https://s0.lgstatic.com/i/image/M00/8A/40/Ciqc1F_ZywCAHl6DAAeLykJFOKE463.png" alt="Drawing 4.png" data-nodeid="1142"></p>
<p data-nodeid="989">最后，添加完节点后，就到了回调用户方法，定位到 callHandlerAdded() 的核心源码：</p>
<pre class="lang-java" data-nodeid="990"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">callHandlerAdded</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (setAddComplete()) {
&nbsp; &nbsp; &nbsp; &nbsp; handler().handlerAdded(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">setAddComplete</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> oldState = handlerState;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (oldState == REMOVE_COMPLETE) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (HANDLER_STATE_UPDATER.compareAndSet(<span class="hljs-keyword">this</span>, oldState, ADD_COMPLETE)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="991">Netty 会通过 CAS 修改节点的状态直至 REMOVE_COMPLETE 或者 ADD_COMPLETE，如果修改节点为 ADD_COMPLETE 状态，表示节点已经添加成功，然后会回调用户 Handler 中实现的 handlerAdded() 方法。</p>
<p data-nodeid="992">至此，Pipeline 添加 Handler 的实现原理我们已经讲完了，下面接着看下 Pipeline 删除 Handler 的场景。</p>
<h4 data-nodeid="993">Pipeline 删除 Handler</h4>
<p data-nodeid="994">在《源码篇：从 Linux 出发深入剖析服务端启动流程》的课程中我们介绍了一种特殊的处理器 ChannelInitializer，ChannelInitializer 在服务端 Channel 注册完成之后会从 Pipeline 的双向链表中移除，我们一起回顾下这段代码：</p>
<pre class="lang-java" data-nodeid="995"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(ChannelHandlerContext ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
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
<p data-nodeid="996">继续跟进 pipeline.remove() 的源码。</p>
<pre class="lang-java" data-nodeid="997"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelPipeline <span class="hljs-title">remove</span><span class="hljs-params">(ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 1. getContextOrDie 用于查找需要删除的节点</span>
&nbsp; &nbsp; remove(getContextOrDie(handler));
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> AbstractChannelHandlerContext <span class="hljs-title">remove</span><span class="hljs-params">(<span class="hljs-keyword">final</span> AbstractChannelHandlerContext ctx)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">assert</span> ctx != head &amp;&amp; ctx != tail;
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 删除双向链表中的 Handler 节点</span>
&nbsp; &nbsp; &nbsp; &nbsp; atomicRemoveFromHandlerList(ctx);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!registered) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; callHandlerCallbackLater(ctx, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; EventExecutor executor = ctx.executor();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!executor.inEventLoop()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; executor.execute(<span class="hljs-keyword">new</span> Runnable() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; callHandlerRemoved0(ctx);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 3. 回调用户函数</span>
&nbsp; &nbsp; callHandlerRemoved0(ctx);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
}
</code></pre>
<p data-nodeid="998">整个删除 Handler 的过程可以分为三步，分别为：</p>
<ol data-nodeid="999">
<li data-nodeid="1000">
<p data-nodeid="1001">查找需要删除的 Handler 节点；</p>
</li>
<li data-nodeid="1002">
<p data-nodeid="1003">然后删除双向链表中的 Handler 节点；</p>
</li>
<li data-nodeid="1004">
<p data-nodeid="1005">最后回调用户函数。</p>
</li>
</ol>
<p data-nodeid="1006">我们对每一步逐一进行拆解。</p>
<p data-nodeid="1007">第一步查找需要删除的 Handler 节点，我们自然可以想到通过遍历双向链表实现。一起看下 getContextOrDie() 方法的源码：</p>
<pre class="lang-java" data-nodeid="1008"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> AbstractChannelHandlerContext <span class="hljs-title">getContextOrDie</span><span class="hljs-params">(ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; AbstractChannelHandlerContext ctx = (AbstractChannelHandlerContext) context(handler);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ctx == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NoSuchElementException(handler.getClass().getName());
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelHandlerContext <span class="hljs-title">context</span><span class="hljs-params">(ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NullPointerException(<span class="hljs-string">"handler"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 遍历双向链表查找</span>
&nbsp; &nbsp; AbstractChannelHandlerContext ctx = head.next;
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ctx == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果 Handler 相同，返回当前的 Context 节点</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ctx.handler() == handler) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; ctx = ctx.next;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1009">Netty 确实是从双向链表的头结点开始依次遍历，如果当前 Context 节点的 Handler 要被删除的 Handler 相同，那么便找到了要删除的 Handler，然后返回当前 Context 节点。</p>
<p data-nodeid="1010">找到需要删除的 Handler 节点之后，接下来就是将节点从双向链表中删除，再跟进atomicRemoveFromHandlerList() 方法的源码：</p>
<pre class="lang-java" data-nodeid="1011"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">synchronized</span> <span class="hljs-keyword">void</span> <span class="hljs-title">atomicRemoveFromHandlerList</span><span class="hljs-params">(AbstractChannelHandlerContext ctx)</span> </span>{
&nbsp; &nbsp; AbstractChannelHandlerContext prev = ctx.prev;
&nbsp; &nbsp; AbstractChannelHandlerContext next = ctx.next;
&nbsp; &nbsp; prev.next = next;
&nbsp; &nbsp; next.prev = prev;
}
</code></pre>
<p data-nodeid="1012">删除节点和添加节点类似，都是基本的链表操作，通过调整双向链表的指针即可实现。假设现在需要删除 SampleOutboundA 节点，我们以一幅图来表示删除时指针的变化过程，如下所示。</p>
<p data-nodeid="3745" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/C4/CgqCHl_fiRyAI2KKAAH7-GYFUt4327.png" alt="图片6.png" data-nodeid="3748"></p>

<p data-nodeid="1014">删除完节点之后，最后 Netty 会回调用户自定义实现的 handlerRemoved() 方法，回调的实现过程与添加节点时是类似的，在这里我就不赘述了。</p>
<p data-nodeid="1015">到此为止，我们已经学会了 ChannelPipeline 内部结构的基本操作，只需要基本的链表操作就可以实现 Handler 节点的添加和删除，添加时通过掩码运算的方式排出 Handler 不关心的事件。 ChannelPipeline 是如何调度 Handler 的呢？接下来我们继续学习。</p>
<h3 data-nodeid="1016">数据在 Pipeline 中的运转</h3>
<p data-nodeid="1017">我们知道，根据数据的流向，ChannelPipeline 分为入站 ChannelInboundHandler 和出站 ChannelOutboundHandler 两种处理器。Inbound 事件和 Outbound 事件的传播方向相反，Inbound 事件的传播方向为 Head -&gt; Tail，而 Outbound 事件传播方向是 Tail -&gt; Head。今天我们就以客户端和服务端请求-响应的场景，深入研究 ChannelPipeline 的事件传播机制。</p>
<h4 data-nodeid="1018">Inbound 事件传播</h4>
<p data-nodeid="1019">当客户端向服务端发送数据时，服务端是如何接收的呢？回顾下之前我们所学习的 Netty Reactor 线程模型，首先 NioEventLoop 会不断轮询 OP_ACCEPT 和 OP_READ 事件，当事件就绪时，NioEventLoop 会及时响应。首先定位到 NioEventLoop 中源码的入口：</p>
<pre class="lang-java" data-nodeid="1020"><code data-language="java"><span class="hljs-comment">// NioEventLoop#processSelectedKey</span>
<span class="hljs-keyword">if</span> ((readyOps &amp; (SelectionKey.OP_READ | SelectionKey.OP_ACCEPT)) != <span class="hljs-number">0</span> || readyOps == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; unsafe.read();
}
</code></pre>
<p data-nodeid="1021">可以看出 unsafe.read() 会触发后续事件的处理，有一点需要避免混淆，在服务端 Channel 和客户端 Channel 中绑定的 unsafe 对象是不一样的，因为服务端 Channel 只关心如何接收客户端连接，而客户端 Channel 需要关心数据的读写。这里我们重点分析一下客户端 Channel 读取数据的过程，跟进 unsafe.read() 的源码：</p>
<pre class="lang-java" data-nodeid="1022"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">read</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelConfig config = config();
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ChannelPipeline pipeline = pipeline();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ByteBufAllocator allocator = config.getAllocator();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> RecvByteBufAllocator.Handle allocHandle = recvBufAllocHandle();
&nbsp; &nbsp; allocHandle.reset(config);
&nbsp; &nbsp; ByteBuf byteBuf = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> close = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; byteBuf = allocHandle.allocate(allocator); <span class="hljs-comment">// 分配 ByteBuf</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; allocHandle.lastBytesRead(doReadBytes(byteBuf)); <span class="hljs-comment">// 将 Channel 中的数据读到 ByteBuf 中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (allocHandle.lastBytesRead() &lt;= <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; byteBuf.release();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; byteBuf = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; close = allocHandle.lastBytesRead() &lt; <span class="hljs-number">0</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (close) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; readPending = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; allocHandle.incMessagesRead(<span class="hljs-number">1</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; readPending = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelRead(byteBuf); <span class="hljs-comment">// 传播 ChannelRead 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; byteBuf = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">while</span> (allocHandle.continueReading());
&nbsp; &nbsp; &nbsp; &nbsp; allocHandle.readComplete();
&nbsp; &nbsp; &nbsp; &nbsp; pipeline.fireChannelReadComplete(); <span class="hljs-comment">// 传播 readComplete 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (close) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; closeOnRead(pipeline);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; handleReadException(pipeline, byteBuf, t, close, allocHandle);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!readPending &amp;&amp; !config.isAutoRead()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; removeReadOp();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1023">Netty 会不断从 Channel 中读取数据到分配的 ByteBuf 中，然后通过 pipeline.fireChannelRead() 方法触发 ChannelRead 事件的传播，fireChannelRead() 是我们需要重点分析的对象。</p>
<pre class="lang-java" data-nodeid="1024"><code data-language="java"><span class="hljs-comment">// DefaultChannelPipeline</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelPipeline <span class="hljs-title">fireChannelRead</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; AbstractChannelHandlerContext.invokeChannelRead(head, msg);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
<span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeChannelRead</span><span class="hljs-params">(<span class="hljs-keyword">final</span> AbstractChannelHandlerContext next, Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Object m = next.pipeline.touch(ObjectUtil.checkNotNull(msg, <span class="hljs-string">"msg"</span>), next);
&nbsp; &nbsp; EventExecutor executor = next.executor();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor.inEventLoop()) { <span class="hljs-comment">// 当前在 Reactor 线程内部，直接执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; next.invokeChannelRead(m);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; executor.execute(<span class="hljs-keyword">new</span> Runnable() { <span class="hljs-comment">// 如果是外部线程，则提交给异步任务队列</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; next.invokeChannelRead(m);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1025">Netty 首先会以 Head 节点为入参，直接调用一个静态方法 invokeChannelRead()。如果当前是在 Reactor 线程内部，会直接执行 next.invokeChannelRead() 方法。如果是外部线程发起的调用，Netty 会把 next.invokeChannelRead() 调用封装成异步任务提交到任务队列。通过之前对 NioEventLoop 源码的学习，我们知道这样可以保证执行流程全部控制在当前 NioEventLoop 线程内部串行化执行，确保线程安全性。我们抓住核心逻辑 next.invokeChannelRead() 继续跟进。</p>
<pre class="lang-java" data-nodeid="1026"><code data-language="java"><span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeChannelRead</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokeHandler()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ((ChannelInboundHandler) handler()).channelRead(<span class="hljs-keyword">this</span>, msg);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; notifyHandlerException(t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; fireChannelRead(msg);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1027">可以看出，当前 ChannelHandlerContext 节点会取出自身对应的 Handler，执行 Handler 的 channelRead 方法。此时当前节点是 HeadContext，所以 Inbound 事件是从 HeadContext 节点开始进行传播的，看下 HeadContext.channelRead() 是如何实现的。</p>
<pre class="lang-java" data-nodeid="1028"><code data-language="java"><span class="hljs-comment">// HeadContext</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> </span>{
&nbsp; &nbsp; ctx.fireChannelRead(msg);
}
<span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ChannelHandlerContext <span class="hljs-title">fireChannelRead</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 找到下一个节点，执行 invokeChannelRead</span>
&nbsp; &nbsp; invokeChannelRead(findContextInbound(MASK_CHANNEL_READ), msg);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
</code></pre>
<p data-nodeid="1029">我们发现 HeadContext.channelRead() 并没有做什么特殊操作，而是直接通过 fireChannelRead() 方法继续将读事件继续传播下去。接下来 Netty 会通过 findContextInbound(MASK_CHANNEL_READ), msg) 找到 HeadContext 的下一个节点，然后继续执行我们之前介绍的静态方法 invokeChannelRead()，从而进入一个递归调用的过程，直至某个条件结束。以上 channelRead 的执行过程我们可以梳理成一幅流程图：</p>
<p data-nodeid="1030"><img src="https://s0.lgstatic.com/i/image/M00/8A/40/Ciqc1F_Zyz-AR9gUAAYhisGwxMo407.png" alt="Drawing 6.png" data-nodeid="1188"></p>
<p data-nodeid="1031">Netty 是如何判断 InboundHandler 是否关心 channelRead 事件呢？这就涉及findContextInbound(MASK_CHANNEL_READ), msg) 中的一个知识点，和上文中我们介绍的 executionMask 掩码运算是息息相关的。首先看下 findContextInbound() 的源码：</p>
<pre class="lang-java" data-nodeid="1032"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> AbstractChannelHandlerContext <span class="hljs-title">findContextInbound</span><span class="hljs-params">(<span class="hljs-keyword">int</span> mask)</span> </span>{
&nbsp; &nbsp; AbstractChannelHandlerContext ctx = <span class="hljs-keyword">this</span>;
&nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ctx = ctx.next;
&nbsp; &nbsp; } <span class="hljs-keyword">while</span> ((ctx.executionMask &amp; mask) == <span class="hljs-number">0</span>);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> ctx;
}
</code></pre>
<p data-nodeid="1033">MASK_CHANNEL_READ 的值为 1 &lt;&lt; 5，表示 channelRead 事件所在的二进制位已被置为 1。在代码示例中，SampleInboundA 是我们添加的 Inbound 类型的自定义处理器，它所对应的 executionMask 掩码和 MASK_CHANNEL_READ 进行与运算的结果如果不为 0，表示 SampleInboundA 对 channelRead 事件感兴趣，需要触发执行 SampleInboundA 的 channelRead() 方法。</p>
<p data-nodeid="1034">Inbound 事件在上述递归调用的流程中什么时候能够结束呢？有以下两种情况：</p>
<ol data-nodeid="1035">
<li data-nodeid="1036">
<p data-nodeid="1037">用户自定义的 Handler 没有执行 fireChannelRead() 操作，则在当前 Handler 终止 Inbound 事件传播。</p>
</li>
<li data-nodeid="1038">
<p data-nodeid="1039">如果用户自定义的 Handler 都执行了 fireChannelRead() 操作，Inbound 事件传播最终会在 TailContext 节点终止。</p>
</li>
</ol>
<p data-nodeid="1040">接下来，我们着重看下 TailContext 节点做了哪些工作。</p>
<pre class="lang-java" data-nodeid="1041"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> </span>{
&nbsp; &nbsp; onUnhandledInboundMessage(ctx, msg);
}
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onUnhandledInboundMessage</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; logger.debug(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Discarded inbound message {} that reached at the tail of the pipeline. "</span> +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Please check your pipeline configuration."</span>, msg);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ReferenceCountUtil.release(msg);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1042">可以看出 TailContext 只是日志记录了丢弃的 Inbound 消息，并释放 ByteBuf 做一个兜底保护，防止内存泄漏。</p>
<p data-nodeid="1043">到此为止，Inbound 事件的传播流程已经介绍完了，Inbound 事件在 ChannelPipeline 中的传播方向是 Head -&gt; Tail。Netty 会从 ChannelPipeline 中找到对传播事件感兴趣的 Inbound 处理器，执行事件回调方法，然后继续向下一个节点传播，整个事件传播流程是一个递归调用的过程。</p>
<h4 data-nodeid="1044">Outbound 事件传播</h4>
<p data-nodeid="1045">分析完 Inbound 事件的传播流程之后，再学习 Outbound 事件传播就会简单很多。Outbound 事件传播的方向是从 Tail -&gt; Head，与 Inbound 事件的传播方向恰恰是相反的。Outbound 事件最常见的就是写事件，执行 writeAndFlush() 方法时就会触发 Outbound 事件传播。我们直接从 TailContext 跟进 writeAndFlush() 源码：</p>
<pre class="lang-java" data-nodeid="1046"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> ChannelFuture <span class="hljs-title">writeAndFlush</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> tail.writeAndFlush(msg);
}
</code></pre>
<p data-nodeid="1047">继续跟进 tail.writeAndFlush() 的源码，最终会定位到 AbstractChannelHandlerContext 中的 write 方法。该方法是 writeAndFlush 的核心逻辑，具体源码如下。</p>
<pre class="lang-java" data-nodeid="1048"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(Object msg, <span class="hljs-keyword">boolean</span> flush, ChannelPromise promise)</span> </span>{
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
<p data-nodeid="1049">在《数据传输：writeAndFlush 处理流程剖析》的课程中，我们已经对 write() 方法做了深入分析，这里抛开其他技术细节，我们只分析 Outbound 事件传播的过程。</p>
<p data-nodeid="1050">假设我们在代码示例中 SampleOutboundB 调用了 writeAndFlush() 方法，那么 Netty 会调用 findContextOutbound() 方法找到 Pipeline 链表中下一个 Outbound 类型的 ChannelHandler，对应上述代码示例中下一个 Outbound 节点是 SampleOutboundA，然后调用 next.invokeWriteAndFlush(m, promise)，我们跟进去：</p>
<pre class="lang-java" data-nodeid="1051"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeWriteAndFlush</span><span class="hljs-params">(Object msg, ChannelPromise promise)</span> </span>{
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
<p data-nodeid="1052">我们发现，invokeWriteAndFlush() 方法最终会它会执行下一个 ChannelHandler 节点的 write 方法。一般情况下，用户在实现 outBound 类型的 ChannelHandler 时都会继承 ChannelOutboundHandlerAdapter，一起看下它的 write() 方法是如何处理 outBound 事件的。</p>
<pre class="lang-java" data-nodeid="1053"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; ctx.write(msg, promise);
}
</code></pre>
<p data-nodeid="1054">ChannelOutboundHandlerAdapter.write() 只是调用了 AbstractChannelHandlerContext 的 write() 方法，是不是似曾相识？与之前介绍的 Inbound 事件处理流程类似，此时流程又回到了 AbstractChannelHandlerContext 中重复执行 write 方法，继续寻找下一个 Outbound 节点，也是一个递归调用的过程。</p>
<p data-nodeid="1055">编码器是用户经常需要自定义实现的处理器，然而为什么用户的编码器里并没有重写 write()，只是重写一个 encode() 方法呢？在《Netty 如何实现自定义通信协议》课程中，我们所介绍的 MessageToByteEncoder 源码，用户自定义的编码器基本都会继承 MessageToByteEncoder，MessageToByteEncoder 重写了 ChanneOutboundHandler 的 write() 方法，其中会调用子类实现的 encode 方法完成数据编码，这里我们不再赘述了。</p>
<p data-nodeid="1056">那么 OutBound 事件什么时候传播结束呢？也许你已经猜到了，OutBound 事件最终会传播到 HeadContext 节点。所以 HeadContext 节点既是 Inbound 处理器，又是 OutBound 处理器，继续看下 HeadContext 是如何拦截和处理 write 事件的。</p>
<pre class="lang-java" data-nodeid="1057"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)</span> </span>{
&nbsp; &nbsp; unsafe.write(msg, promise);
}
</code></pre>
<p data-nodeid="1058">HeadContext 最终调用了底层的 unsafe 写入数据，数据在执行 write() 方法时，只会写入到一个底层的缓冲数据结构，然后等待 flush 操作将数据冲刷到 Channel 中。关于 write 和 flush 是如何操作缓存数据结构的，快去复习一遍《数据传输：writeAndFlush 处理流程剖析》吧，将知识点形成一个完整的体系。</p>
<p data-nodeid="1059">到此为止，outbound 事件传播也介绍完了，它的传播方向是 Tail -&gt; Head，与 Inbound 事件的传播是相反的。MessageToByteEncoder 是用户在实现编码时经常用到的一个抽象类，MessageToByteEncoder 中已经重写了 ChanneOutboundHandler 的 write() 方法，大部分情况下用户只需要重写 encode() 即可。</p>
<h4 data-nodeid="1060">异常事件传播</h4>
<p data-nodeid="1061">在《服务编排层：Pipeline 如何协调各类 Handler》中，我们已经初步介绍了 Netty 实现统一异常拦截和处理的最佳实践，首先回顾下异常拦截器的简单实现。</p>
<pre class="lang-java" data-nodeid="1062"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ExceptionHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelDuplexHandler</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">exceptionCaught</span><span class="hljs-params">(ChannelHandlerContext ctx, Throwable cause)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cause <span class="hljs-keyword">instanceof</span> RuntimeException) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"Handle Business Exception Success."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1063">异常处理器 ExceptionHandler 一般会继承 ChannelDuplexHandler，ChannelDuplexHandler 既是一个 Inbound 处理器，又是一个 Outbound 处理器。ExceptionHandler 应该被添加在自定义处理器的尾部，如下图所示：</p>
<p data-nodeid="4371" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/03/9C/CgpVE1_fiTOATVOAAAJopxQR7mU966.png" alt="图片8.png" data-nodeid="4374"></p>

<p data-nodeid="1065">那么异常处理器 ExceptionHandler 什么时候被执行呢？我们分别从 Inbound 异常事件传播和 Outbound 异常事件传播两种场景进行分析。</p>
<p data-nodeid="1066">首先看下 Inbound 异常事件的传播。还是从数据读取的场景入手，发现 Inbound 事件传播的时候有异常处理的相关逻辑，我们再一起重新分析下数据读取环节的源码。</p>
<pre class="lang-java" data-nodeid="1067"><code data-language="java"><span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeChannelRead</span><span class="hljs-params">(Object msg)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokeHandler()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ((ChannelInboundHandler) handler()).channelRead(<span class="hljs-keyword">this</span>, msg);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; notifyHandlerException(t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; fireChannelRead(msg);
&nbsp; &nbsp; }
}
<span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">notifyHandlerException</span><span class="hljs-params">(Throwable cause)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; invokeExceptionCaught(cause);
}
<span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeExceptionCaught</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Throwable cause)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invokeHandler()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler().exceptionCaught(<span class="hljs-keyword">this</span>, cause); <span class="hljs-comment">// 调用 Handler 实现的 exceptionCaught 方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable error) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; fireExceptionCaught(cause);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1068">如果 SampleInboundA 在读取数据时发生了异常，invokeChannelRead 会捕获异常，并执行 notifyHandlerException() 方法进行异常处理。我们一步步跟进，发现最终会调用 Handler 的 exceptionCaught() 方法，所以用户可以通过重写 exceptionCaught() 实现自定义的异常处理。</p>
<p data-nodeid="1069">我们知道，统一异常处理器 ExceptionHandler 是在 ChannelPipeline 的末端，SampleInboundA 并没有重写 exceptionCaught() 方法，那么 SampleInboundA 产生的异常是如何传播到 ExceptionHandler 中呢？用户实现的 Inbound 处理器一般都会继承 ChannelInboundHandlerAdapter 抽象类，果然我们在 ChannelInboundHandlerAdapter 中发现了 exceptionCaught() 的实现：</p>
<pre class="lang-java" data-nodeid="1070"><code data-language="java"><span class="hljs-comment">// ChannelInboundHandlerAdapter</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">exceptionCaught</span><span class="hljs-params">(ChannelHandlerContext ctx, Throwable cause)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; ctx.fireExceptionCaught(cause);
}
<span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> ChannelHandlerContext <span class="hljs-title">fireExceptionCaught</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Throwable cause)</span> </span>{
&nbsp; &nbsp; invokeExceptionCaught(findContextInbound(MASK_EXCEPTION_CAUGHT), cause);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
</code></pre>
<p data-nodeid="1071">ChannelInboundHandlerAdapter 默认调用 fireExceptionCaught() 方法传播异常事件，而 fireExceptionCaught() 执行时会先调用 findContextInbound() 方法找到下一个对异常事件关注的 Inbound 处理器，然后继续向下传播异常。所以这里应该明白为什么统一异常处理器 ExceptionHandler 为什么需要添加在 ChannelPipeline 的末端了吧？这样 ExceptionHandler 可以接收所有 Inbound 处理器发生的异常。</p>
<p data-nodeid="1072">接下来，我们分析 Outbound 异常事件传播。你可能此时就会有一个疑问，Outbound 事件的传播方向与 Inbound 事件是相反的，为什么统一异常处理器 ExceptionHandler 没有添加在 ChannelPipeline 的头部呢？我们通过 writeAndFlush() 的调用过程再来一探究竟。</p>
<pre class="lang-java" data-nodeid="1073"><code data-language="java"><span class="hljs-comment">// AbstractChannelHandlerContext</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invokeFlush0</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ((ChannelOutboundHandler) handler()).flush(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; notifyHandlerException(t);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1074">我们发现，flush 发送数据时如果发生异常，那么异常也会被捕获并交由同样的 notifyHandlerException() 方法进行处理。因为 notifyHandlerException() 方法中会向下寻找 Inbound 处理器，此时又会回到 Inbound 异常事件的传播流程。所以说，异常事件的传播方向与 Inbound 事件几乎是一样的，最后一定会传播到统一异常处理器 ExceptionHandler。</p>
<p data-nodeid="1075">到这里，整个异常事件的传播过程已经分析完了。你需要记住的是，异常事件的传播顺序与 ChannelHandler 的添加顺序相同，会依次向后传播，与 Inbound 事件和 Outbound 事件无关。</p>
<h3 data-nodeid="1076">总结</h3>
<p data-nodeid="1077">这节点我们学习了数据在 Netty 中的完整处理流程，其中重点分析了数据是如何在 ChannelPipeline 中流转的。我们做一个知识点总结：</p>
<ul data-nodeid="1078">
<li data-nodeid="1079">
<p data-nodeid="1080">ChannelPipeline 是双向链表结构，包含 ChannelInboundHandler 和 ChannelOutboundHandler 两种处理器。</p>
</li>
<li data-nodeid="1081">
<p data-nodeid="1082">Inbound 事件和 Outbound 事件的传播方向相反，Inbound 事件的传播方向为 Head -&gt; Tail，而 Outbound 事件传播方向是 Tail -&gt; Head。</p>
</li>
<li data-nodeid="1083">
<p data-nodeid="1084">异常事件的处理顺序与 ChannelHandler 的添加顺序相同，会依次向后传播，与 Inbound 事件和 Outbound 事件无关。</p>
</li>
</ul>
<p data-nodeid="1085">再整体回顾下 ChannelPipeline 中事件传播的实现原理：</p>
<ul data-nodeid="1086">
<li data-nodeid="1087">
<p data-nodeid="1088">Inbound 事件传播从 HeadContext 节点开始，Outbound 事件传播从 TailContext 节点开始。</p>
</li>
<li data-nodeid="1089">
<p data-nodeid="1090" class="">AbstractChannelHandlerContext 抽象类中实现了一系列 fire 和 invoke 方法，如果想让事件想下传播，只需要调用 fire 系列的方法即可。fire 和 invoke 的系列方法结合 findContextInbound() 和 findContextOutbound() 可以控制 Inbound 和 Outbound 事件的传播方向，整个过程是一个递归调用。</p>
</li>
</ul>

---

### 精选评论

##### **成：
> 我自己看源码把@skip注解忽视了！向老司机系统性学习还是有必要的😊

##### **耀：
> 非常好的一门课程，跟着老师深入学习，受益良多！😂

