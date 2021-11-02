<p data-nodeid="203000" class="">在前面的课程中，我们深入介绍了 Dubbo Remoting 中的 Transport 层，了解了 Dubbo 抽象出来的端到端的统一传输层接口，并分析了以 Netty 为基础的相关实现。当然，其他 NIO 框架的接入也是类似的，本课程就不再展开赘述了。</p>
<p data-nodeid="203001">在本课时中，我们将介绍 Transport 层的上一层，也是 Dubbo Remoting 层中的最顶层—— Exchange 层。<strong data-nodeid="203088">Dubbo 将信息交换行为抽象成 Exchange 层，官方文档对这一层的说明是：封装了请求-响应的语义，即关注一问一答的交互模式，实现了同步转异步</strong>。在 Exchange 这一层，以 Request 和 Response 为中心，针对 Channel、ChannelHandler、Client、RemotingServer 等接口进行实现。</p>
<p data-nodeid="203002">下面我们从 Request 和 Response 这一对基础类开始，依次介绍 Exchange 层中 ExchangeChannel、HeaderExchangeHandler 的核心实现。</p>
<h3 data-nodeid="203003">Request 和 Response</h3>
<p data-nodeid="203004">Exchange 层的 Request 和 Response 这两个类是 Exchange 层的核心对象，是对请求和响应的抽象。我们先来看<strong data-nodeid="203096">Request 类</strong>的核心字段：</p>
<pre class="lang-java" data-nodeid="203005"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Request</span> </span>{
    <span class="hljs-comment">// 用于生成请求的自增ID，当递增到Long.MAX_VALUE之后，会溢出到Long.MIN_VALUE，我们可以继续使用该负数作为消息ID</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> AtomicLong INVOKE_ID = <span class="hljs-keyword">new</span> AtomicLong(<span class="hljs-number">0</span>);
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> mId; <span class="hljs-comment">// 请求的ID</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String mVersion; <span class="hljs-comment">// 请求版本号</span>
    <span class="hljs-comment">// 请求的双向标识，如果该字段设置为true，则Server端在收到请求后，</span>
    <span class="hljs-comment">//&nbsp;需要给Client返回一个响应</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> mTwoWay = <span class="hljs-keyword">true</span>; 
    <span class="hljs-comment">// 事件标识，例如心跳请求、只读请求等，都会带有这个标识</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> mEvent = <span class="hljs-keyword">false</span>; 
    <span class="hljs-comment">// 请求发送到Server之后，由Decoder将二进制数据解码成Request对象，</span>
    <span class="hljs-comment">// 如果解码环节遇到异常，则会设置该标识，然后交由其他ChannelHandler根据</span>
    <span class="hljs-comment">// 该标识做进一步处理</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> mBroken = <span class="hljs-keyword">false</span>;
    <span class="hljs-comment">// 请求体，可以是任何Java类型的对象,也可以是null</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> Object mData; 
}
</code></pre>
<p data-nodeid="203006">接下来是 Response 的核心字段：</p>
<pre class="lang-java" data-nodeid="203007"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Response</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 响应ID，与相应请求的ID一致</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> mId = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-comment">// 当前协议的版本号，与请求消息的版本号一致</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String mVersion;
    <span class="hljs-comment">// 响应状态码，有OK、CLIENT_TIMEOUT、SERVER_TIMEOUT等10多个可选值</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">byte</span> mStatus = OK; 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> mEvent = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String mErrorMsg; <span class="hljs-comment">// 可读的错误响应消息</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> Object mResult; <span class="hljs-comment">// 响应体</span>
}
</code></pre>
<h3 data-nodeid="203008">ExchangeChannel &amp; DefaultFuture</h3>
<p data-nodeid="203009">在前面的课时中，我们介绍了 Channel 接口的功能以及 Transport 层对 Channel 接口的实现。在 Exchange 层中定义了 ExchangeChannel 接口，它在 Channel 接口之上抽象了 Exchange 层的网络连接。ExchangeChannel 接口的定义如下：</p>
<p data-nodeid="203010"><img src="https://s0.lgstatic.com/i/image/M00/5A/32/Ciqc1F90Q-OAE4K1AADklLgEs0k481.png" alt="Drawing 0.png" data-nodeid="203104"></p>
<div data-nodeid="203011"><p style="text-align:center">ExchangeChannel 接口</p></div>
<p data-nodeid="203012">其中，request() 方法负责发送请求，从图中可以看到这里有两个重载，其中一个重载可以指定请求的超时时间，返回值都是 Future 对象。</p>
<p data-nodeid="203013"><img src="https://s0.lgstatic.com/i/image/M00/5A/3D/CgqCHl90Q_SAIt4sAAAzhH5TZiw571.png" alt="Drawing 1.png" data-nodeid="203108"></p>
<div data-nodeid="203014"><p style="text-align:center">HeaderExchangeChannel 继承关系图</p></div>
<p data-nodeid="203015"><strong data-nodeid="203113">从上图中可以看出，HeaderExchangeChannel 是 ExchangeChannel 的实现</strong>，它本身是 Channel 的装饰器，封装了一个 Channel 对象，其 send() 方法和 request() 方法的实现都是依赖底层修饰的这个 Channel 对象实现的。</p>
<pre class="lang-java" data-nodeid="203016"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(Object message, <span class="hljs-keyword">boolean</span> sent)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (message <span class="hljs-keyword">instanceof</span> Request || message <span class="hljs-keyword">instanceof</span> Response
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; || message <span class="hljs-keyword">instanceof</span> String) {
&nbsp; &nbsp; &nbsp; &nbsp; channel.send(message, sent);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Request request = <span class="hljs-keyword">new</span> Request();
&nbsp; &nbsp; &nbsp; &nbsp; request.setVersion(Version.getProtocolVersion());
&nbsp; &nbsp; &nbsp; &nbsp; request.setTwoWay(<span class="hljs-keyword">false</span>);
&nbsp; &nbsp; &nbsp; &nbsp; request.setData(message);
&nbsp; &nbsp; &nbsp; &nbsp; channel.send(request, sent);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">public</span> CompletableFuture&lt;Object&gt; <span class="hljs-title">request</span><span class="hljs-params">(Object request, <span class="hljs-keyword">int</span> timeout, ExecutorService executor)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; Request req = <span class="hljs-keyword">new</span> Request(); &nbsp;<span class="hljs-comment">// 创建Request对象</span>
&nbsp; &nbsp; req.setVersion(Version.getProtocolVersion());
&nbsp; &nbsp; req.setTwoWay(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; req.setData(request);
&nbsp; &nbsp; DefaultFuture future = DefaultFuture.newFuture(channel, 
       req, timeout, executor); <span class="hljs-comment">// 创建DefaultFuture</span>
&nbsp; &nbsp; channel.send(req);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> future;
}
</code></pre>
<p data-nodeid="203017">注意这里的 request() 方法，它返回的是一个 DefaultFuture 对象。通过前面课时的介绍我们知道，io.netty.channel.Channel 的 send() 方法会返回一个 ChannelFuture 方法，表示此次发送操作是否完成，而这里的<strong data-nodeid="203119">DefaultFuture 就表示此次请求-响应是否完成，也就是说，要收到响应为 Future 才算完成</strong>。</p>
<p data-nodeid="203018">下面我们就来深入介绍一下请求发送过程中涉及的 DefaultFuture 以及HeaderExchangeChannel的内容。</p>
<p data-nodeid="203019">首先来了解一下 DefaultFuture 的具体实现，它继承了 JDK 中的 CompletableFuture，其中维护了两个 static 集合。</p>
<ul data-nodeid="203020">
<li data-nodeid="203021">
<p data-nodeid="203022">CHANNELS（Map&lt;Long, Channel&gt;集合）：管理请求与 Channel 之间的关联关系，其中 Key 为请求 ID，Value 为发送请求的 Channel。</p>
</li>
<li data-nodeid="203023">
<p data-nodeid="203024">FUTURES（Map&lt;Long, Channel&gt;集合）：管理请求与 DefaultFuture 之间的关联关系，其中 Key 为请求 ID，Value 为请求对应的 Future。</p>
</li>
</ul>
<p data-nodeid="203025">DefaultFuture 中核心的实例字段包括如下几个。</p>
<ul data-nodeid="203026">
<li data-nodeid="203027">
<p data-nodeid="203028">request（Request 类型）和 id（Long 类型）：对应请求以及请求的 ID。</p>
</li>
<li data-nodeid="203029">
<p data-nodeid="203030">channel（Channel 类型）：发送请求的 Channel。</p>
</li>
<li data-nodeid="203031">
<p data-nodeid="203032">timeout（int 类型）：整个请求-响应交互完成的超时时间。</p>
</li>
<li data-nodeid="203033">
<p data-nodeid="203034">start（long 类型）：该 DefaultFuture 的创建时间。</p>
</li>
<li data-nodeid="203035">
<p data-nodeid="203036">sent（volatile long 类型）：请求发送的时间。</p>
</li>
<li data-nodeid="203037">
<p data-nodeid="203038">timeoutCheckTask（Timeout 类型）：该定时任务到期时，表示对端响应超时。</p>
</li>
<li data-nodeid="203039">
<p data-nodeid="203040">executor（ExecutorService 类型）：请求关联的线程池。</p>
</li>
</ul>
<p data-nodeid="203041">DefaultFuture.newFuture() 方法创建 DefaultFuture 对象时，需要先初始化上述字段，并创建请求相应的超时定时任务：</p>
<pre class="lang-java" data-nodeid="203042"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> DefaultFuture <span class="hljs-title">newFuture</span><span class="hljs-params">(Channel channel, Request request, <span class="hljs-keyword">int</span> timeout, ExecutorService executor)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 创建DefaultFuture对象，并初始化其中的核心字段</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> DefaultFuture future = <span class="hljs-keyword">new</span> DefaultFuture(channel, request, timeout);
&nbsp; &nbsp; future.setExecutor(executor); 
&nbsp; &nbsp; <span class="hljs-comment">// 对于ThreadlessExecutor的特殊处理，ThreadlessExecutor可以关联一个waitingFuture，就是这里创建DefaultFuture对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor <span class="hljs-keyword">instanceof</span> ThreadlessExecutor) {
&nbsp; &nbsp; &nbsp; &nbsp; ((ThreadlessExecutor) executor).setWaitingFuture(future);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 创建一个定时任务，用处理响应超时的情况</span>
&nbsp; &nbsp; timeoutCheck(future);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> future;
}
</code></pre>
<p data-nodeid="203043">在 HeaderExchangeChannel.request() 方法中完成 DefaultFuture 对象的创建之后，会将请求通过底层的 Dubbo Channel 发送出去，发送过程中会触发沿途 ChannelHandler 的 sent() 方法，其中的 HeaderExchangeHandler 会调用 DefaultFuture.sent() 方法更新 sent 字段，记录请求发送的时间戳。后续如果响应超时，则会将该发送时间戳添加到提示信息中。</p>
<p data-nodeid="203044">过一段时间之后，Consumer 会收到对端返回的响应，在读取到完整响应之后，会触发 Dubbo Channel 中各个 ChannelHandler 的 received() 方法，其中就包括上一课时介绍的 WrappedChannelHandler。例如，AllChannelHandler 子类会将后续 ChannelHandler.received() 方法的调用封装成任务提交到线程池中，响应会提交到 DefaultFuture 关联的线程池中，如上一课时介绍的 ThreadlessExecutor，然后由业务线程继续后续的 ChannelHandler 调用。（你也可以回顾一下上一课时对 Transport 层 Dispatcher 以及 ThreadlessExecutor 的介绍。）</p>
<p data-nodeid="203045">当响应传递到 HeaderExchangeHandler 的时候，会通过调用 handleResponse() 方法进行处理，其中调用了 DefaultFuture.received() 方法，该方法会找到响应关联的 DefaultFuture 对象（根据请求 ID 从 FUTURES 集合查找）并调用 doReceived() 方法，将 DefaultFuture 设置为完成状态。</p>
<pre class="lang-java" data-nodeid="203046"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">received</span><span class="hljs-params">(Channel channel, Response response, <span class="hljs-keyword">boolean</span> timeout)</span> </span>{ <span class="hljs-comment">// 省略try/finally代码块</span>
&nbsp; &nbsp; <span class="hljs-comment">// 清理FUTURES中记录的请求ID与DefaultFuture之间的映射关系</span>
&nbsp; &nbsp; DefaultFuture future = FUTURES.remove(response.getId());&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (future != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; Timeout t = future.timeoutCheckTask;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!timeout) { <span class="hljs-comment">// 未超时，取消定时任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; t.cancel();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; future.doReceived(response); <span class="hljs-comment">// 调用doReceived()方法</span>
&nbsp; &nbsp; }<span class="hljs-keyword">else</span>{ <span class="hljs-comment">// 查找不到关联的DefaultFuture会打印日志(略)}</span>
&nbsp; &nbsp; <span class="hljs-comment">// 清理CHANNELS中记录的请求ID与Channel之间的映射关系</span>
&nbsp; &nbsp; CHANNELS.remove(response.getId());&nbsp;
}
<span class="hljs-comment">// DefaultFuture.doReceived()方法的代码片段</span>
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doReceived</span><span class="hljs-params">(Response res)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (res == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"response cannot be null"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (res.getStatus() == Response.OK) { <span class="hljs-comment">// 正常响应</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.complete(res.getResult());
&nbsp; &nbsp; }&nbsp;<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (res.getStatus() == Response.CLIENT_TIMEOUT || res.getStatus() == Response.SERVER_TIMEOUT) { <span class="hljs-comment">// 超时</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.completeExceptionally(<span class="hljs-keyword">new</span> TimeoutException(res.getStatus() == Response.SERVER_TIMEOUT, channel, res.getErrorMessage()));
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 其他异常</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.completeExceptionally(<span class="hljs-keyword">new</span> RemotingException(channel, res.getErrorMessage()));
&nbsp; &nbsp; }
    <span class="hljs-comment">// 下面是针对ThreadlessExecutor的兜底处理，主要是防止业务线程一直阻塞在ThreadlessExecutor上</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor != <span class="hljs-keyword">null</span> &amp;&amp; executor <span class="hljs-keyword">instanceof</span> ThreadlessExecutor) {
&nbsp; &nbsp; &nbsp; &nbsp; ThreadlessExecutor threadlessExecutor = (ThreadlessExecutor) executor;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadlessExecutor.isWaiting()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// notifyReturn()方法会向ThreadlessExecutor提交一个任务，这样业务线程就不会阻塞了，提交的任务会尝试将DefaultFuture设置为异常结束</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; threadlessExecutor.notifyReturn(<span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"The result has returned..."</span>));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="203047">下面我们再来看看响应超时的场景。在创建 DefaultFuture 时调用的 timeoutCheck() 方法中，会创建 TimeoutCheckTask 定时任务，并添加到时间轮中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="203048"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">timeoutCheck</span><span class="hljs-params">(DefaultFuture future)</span> </span>{
&nbsp; &nbsp; TimeoutCheckTask task = <span class="hljs-keyword">new</span> TimeoutCheckTask(future.getId());
&nbsp; &nbsp; future.timeoutCheckTask = TIME_OUT_TIMER.newTimeout(task, future.getTimeout(), TimeUnit.MILLISECONDS);
}
</code></pre>
<p data-nodeid="203049">TIME_OUT_TIMER 是一个 HashedWheelTimer 对象，即 Dubbo 中对时间轮的实现，这是一个 static 字段，所有 DefaultFuture 对象共用一个。</p>
<p data-nodeid="203050">TimeoutCheckTask 是 DefaultFuture 中的内部类，实现了 TimerTask 接口，可以提交到时间轮中等待执行。当响应超时的时候，TimeoutCheckTask 会创建一个 Response，并调用前面介绍的 DefaultFuture.received() 方法。示例代码如下：</p>
<pre class="lang-java" data-nodeid="203051"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(Timeout timeout)</span> </span>{
    <span class="hljs-comment">// 检查该任务关联的DefaultFuture对象是否已经完成</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (future.getExecutor() != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 提交到线程池执行，注意ThreadlessExecutor的情况</span>
&nbsp; &nbsp; &nbsp; &nbsp; future.getExecutor().execute(() -&gt; notifyTimeout(future));
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; notifyTimeout(future);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">notifyTimeout</span><span class="hljs-params">(DefaultFuture future)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 没有收到对端的响应，这里会创建一个Response，表示超时的响应</span>
&nbsp; &nbsp; Response timeoutResponse = <span class="hljs-keyword">new</span> Response(future.getId());
&nbsp; &nbsp; timeoutResponse.setStatus(future.isSent() ? Response.SERVER_TIMEOUT : Response.CLIENT_TIMEOUT);
&nbsp; &nbsp; timeoutResponse.setErrorMessage(future.getTimeoutMessage(<span class="hljs-keyword">true</span>));
&nbsp; &nbsp; <span class="hljs-comment">// 将关联的DefaultFuture标记为超时异常完成</span>
&nbsp; &nbsp; DefaultFuture.received(future.getChannel(), timeoutResponse, <span class="hljs-keyword">true</span>);
}
</code></pre>
<h3 data-nodeid="203052">HeaderExchangeHandler</h3>
<p data-nodeid="203053">在前面介绍 DefaultFuture 时，我们简单说明了请求-响应的流程，其实无论是发送请求还是处理响应，都会涉及 HeaderExchangeHandler，所以这里我们就来介绍一下 HeaderExchangeHandler 的内容。</p>
<p data-nodeid="203054"><strong data-nodeid="203153">HeaderExchangeHandler 是 ExchangeHandler 的装饰器</strong>，其中维护了一个 ExchangeHandler 对象，ExchangeHandler 接口是 Exchange 层与上层交互的接口之一，上层调用方可以实现该接口完成自身的功能；然后再由 HeaderExchangeHandler 修饰，具备 Exchange 层处理 Request-Response 的能力；最后再由 Transport ChannelHandler 修饰，具备 Transport 层的能力。如下图所示：</p>
<p data-nodeid="204097"><img src="https://s0.lgstatic.com/i/image/M00/5D/D2/Ciqc1F-FWUqAVkr0AADiEwO4wK4124.png" alt="Lark20201013-153600.png" data-nodeid="204100"></p>

<div data-nodeid="203651"><p style="text-align:center">ChannelHandler 继承关系总览图</p></div>




<p data-nodeid="203057">HeaderExchangeHandler 作为一个装饰器，其 connected()、disconnected()、sent()、received()、caught() 方法最终都会转发给上层提供的 ExchangeHandler 进行处理。这里我们需要聚焦的是 HeaderExchangeHandler 本身对 Request 和 Response 的处理逻辑。</p>
<p data-nodeid="205198"><img src="https://s0.lgstatic.com/i/image/M00/5D/D2/Ciqc1F-FWVeAbsckAAGeD-_NNHc225.png" alt="Lark20201013-153557.png" data-nodeid="205201"></p>

<div data-nodeid="204752"><p style="text-align:center">received() 方法处理的消息分类</p></div>




<p data-nodeid="203060">结合上图，我们可以看到在<strong data-nodeid="203166">received() 方法</strong>中，对收到的消息进行了分类处理。</p>
<ul data-nodeid="203061">
<li data-nodeid="203062">
<p data-nodeid="203063">只读请求会由<strong data-nodeid="203172">handlerEvent() 方法</strong>进行处理，它会在 Channel 上设置 channel.readonly 标志，后续介绍的上层调用中会读取该值。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="203064"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">handlerEvent</span><span class="hljs-params">(Channel channel, Request req)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (req.getData() != <span class="hljs-keyword">null</span> &amp;&amp; req.getData().equals(READONLY_EVENT)) {
&nbsp; &nbsp; &nbsp; &nbsp; channel.setAttribute(Constants.CHANNEL_ATTRIBUTE_READONLY_KEY, Boolean.TRUE);
&nbsp; &nbsp; }
}
</code></pre>
<ul data-nodeid="203065">
<li data-nodeid="203066">
<p data-nodeid="203067">双向请求由<strong data-nodeid="203178">handleRequest() 方法</strong>进行处理，会先对解码失败的请求进行处理，返回异常响应；然后将正常解码的请求交给上层实现的 ExchangeHandler 进行处理，并添加回调。上层 ExchangeHandler 处理完请求后，会触发回调，根据处理结果填充响应结果和响应码，并向对端发送。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="203068"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">handleRequest</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ExchangeChannel channel, Request req)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; Response res = <span class="hljs-keyword">new</span> Response(req.getId(), req.getVersion());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (req.isBroken()) { <span class="hljs-comment">// 请求解码失败</span>
&nbsp; &nbsp; &nbsp; &nbsp; Object data = req.getData();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 设置异常信息和响应码</span>
&nbsp; &nbsp; &nbsp; &nbsp; res.setErrorMessage(<span class="hljs-string">"Fail to decode request due to: "</span> + msg);
&nbsp; &nbsp; &nbsp; &nbsp; res.setStatus(Response.BAD_REQUEST);&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; channel.send(res); <span class="hljs-comment">// 将异常响应返回给对端</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; Object msg = req.getData();
&nbsp; &nbsp; <span class="hljs-comment">// 交给上层实现的ExchangeHandler进行处理</span>
&nbsp; &nbsp; CompletionStage&lt;Object&gt; future = handler.reply(channel, msg);
&nbsp; &nbsp; future.whenComplete((appResult, t) -&gt; { <span class="hljs-comment">// 处理结束后的回调</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (t == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 返回正常响应</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res.setStatus(Response.OK);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res.setResult(appResult);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 处理过程发生异常，设置异常信息和错误码</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res.setStatus(Response.SERVICE_ERROR);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; res.setErrorMessage(StringUtils.toString(t));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; channel.send(res); <span class="hljs-comment">// 发送响应</span>
&nbsp; &nbsp; });
}
</code></pre>
<ul data-nodeid="203069">
<li data-nodeid="203070">
<p data-nodeid="203071">单向请求直接委托给<strong data-nodeid="203184">上层 ExchangeHandler 实现的 received() 方法</strong>进行处理，由于不需要响应，HeaderExchangeHandler 不会关注处理结果。</p>
</li>
<li data-nodeid="203072">
<p data-nodeid="203073">对于 Response 的处理，前文已提到了，HeaderExchangeHandler 会通过<strong data-nodeid="203190">handleResponse() 方法</strong>将关联的 DefaultFuture 设置为完成状态（或是异常完成状态），具体内容这里不再展开讲述。</p>
</li>
<li data-nodeid="203074">
<p data-nodeid="203075">对于 String 类型的消息，HeaderExchangeHandler 会根据当前服务的角色进行分类，具体与 Dubbo 对 telnet 的支持相关，后面的课时会详细介绍，这里就不展开分析了。</p>
</li>
</ul>
<p data-nodeid="203076">接下来我们再来看<strong data-nodeid="203197">sent() 方法</strong>，该方法会通知上层 ExchangeHandler 实现的 sent() 方法，同时还会针对 Request 请求调用 DefaultFuture.sent() 方法记录请求的具体发送时间，该逻辑在前文也已经介绍过了，这里不再重复。</p>
<p data-nodeid="203077">在<strong data-nodeid="203203">connected() 方法</strong>中，会为 Dubbo Channel 创建相应的 HeaderExchangeChannel，并将两者绑定，然后通知上层 ExchangeHandler 处理 connect 事件。</p>
<p data-nodeid="203078">在<strong data-nodeid="203211">disconnected() 方法</strong>中，首先通知上层 ExchangeHandler 进行处理，之后在 DefaultFuture.closeChannel() 通知 DefaultFuture 连接断开（其实就是创建并传递一个 Response，该 Response 的状态码为 CHANNEL_INACTIVE），这样就不会继续阻塞业务线程了，最后再将 HeaderExchangeChannel 与底层的 Dubbo Channel 解绑。</p>
<h3 data-nodeid="203079">总结</h3>
<p data-nodeid="203080">本课时我们重点介绍了 Dubbo Exchange 层中对 Channel 和 ChannelHandler 接口的实现。</p>
<p data-nodeid="203081" class="">我们首先介绍了 Exchange 层中请求-响应模型的基本抽象，即 Request 类和 Response 类。然后又介绍了 ExchangeChannel 对 Channel 接口的实现，同时还说明了发送请求之后得到的 DefaultFuture 对象，这也是上一课时遗留的小问题。最后，讲解了 HeaderExchangeHandler 是如何将 Transporter 层的 ChannelHandler 对象与上层的 ExchangeHandler 对象相关联的。</p>

---

### 精选评论

##### **杰：
> HeaderExchangeChannel和netty连接的channel是什么对应关系哈！服务端和客户端之间的连接数是怎么对应的！

##### **生：
> “发送过程中会触发沿途 ChannelHandler 的 sent() 方法” debug调试了下，没走到DefaultFuture的future.doSent方法，也没有更新sent字段求解！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 示例中传入的values集合，明确指定了demoFilter3在第一位

