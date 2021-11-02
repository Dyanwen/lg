<p data-nodeid="82401">在上一课时，我们以 DubboProtocol 实现为基础，详细介绍了 Dubbo 服务发布的核心流程。在本课时，我们继续介绍 DubboProtocol 中<strong data-nodeid="82407">服务引用</strong>相关的实现。</p>


<h3 data-nodeid="81657">refer 流程</h3>
<p data-nodeid="81658">下面我们开始介绍 DubboProtocol 中引用服务的相关实现，其核心实现在 protocolBindingRefer() 方法中：</p>
<pre class="lang-java" data-nodeid="81659"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">Invoker&lt;T&gt; <span class="hljs-title">protocolBindingRefer</span><span class="hljs-params">(Class&lt;T&gt; serviceType, URL url)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; optimizeSerialization(url); <span class="hljs-comment">// 进行序列化优化，注册需要优化的类</span>
&nbsp; &nbsp; <span class="hljs-comment">// 创建DubboInvoker对象</span>
&nbsp; &nbsp; DubboInvoker&lt;T&gt; invoker = <span class="hljs-keyword">new</span> DubboInvoker&lt;T&gt;(serviceType, url, getClients(url), invokers);
    <span class="hljs-comment">// 将上面创建DubboInvoker对象添加到invoker集合之中</span>
&nbsp; &nbsp; invokers.add(invoker); 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker;
}
</code></pre>
<p data-nodeid="81660">关于 DubboInvoker 的具体实现，我们先暂时不做深入分析。这里我们需要先关注的是<strong data-nodeid="81736">getClients() 方法</strong>，它创建了底层发送请求和接收响应的 Client 集合，其核心分为了两个部分，一个是针对<strong data-nodeid="81737">共享连接</strong>的处理，另一个是针对<strong data-nodeid="81738">独享连接</strong>的处理，具体实现如下：</p>
<pre class="lang-java" data-nodeid="81661"><code data-language="java"><span class="hljs-keyword">private</span> ExchangeClient[] getClients(URL url) {
&nbsp; &nbsp; <span class="hljs-comment">// 是否使用共享连接</span>
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> useShareConnect = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; <span class="hljs-comment">// CONNECTIONS_KEY参数值决定了后续建立连接的数量</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> connections = url.getParameter(CONNECTIONS_KEY, <span class="hljs-number">0</span>);
&nbsp; &nbsp; List&lt;ReferenceCountExchangeClient&gt; shareClients = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (connections == <span class="hljs-number">0</span>) { <span class="hljs-comment">// 如果没有连接数的相关配置，默认使用共享连接的方式</span>
&nbsp; &nbsp; &nbsp; &nbsp; useShareConnect = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 确定建立共享连接的条数，默认只建立一条共享连接</span>
&nbsp; &nbsp; &nbsp; &nbsp; String shareConnectionsStr = url.getParameter(SHARE_CONNECTIONS_KEY, (String) <span class="hljs-keyword">null</span>);
&nbsp; &nbsp; &nbsp; &nbsp; connections = Integer.parseInt(StringUtils.isBlank(shareConnectionsStr) ? ConfigUtils.getProperty(SHARE_CONNECTIONS_KEY,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DEFAULT_SHARE_CONNECTIONS) : shareConnectionsStr);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建公共ExchangeClient集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; shareClients = getSharedClient(url, connections);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 整理要返回的ExchangeClient集合</span>
&nbsp; &nbsp; ExchangeClient[] clients = <span class="hljs-keyword">new</span> ExchangeClient[connections];
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; clients.length; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (useShareConnect) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; clients[i] = shareClients.get(i);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 不使用公共连接的情况下，会创建单独的ExchangeClient实例</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; clients[i] = initClient(url);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> clients;
}
</code></pre>
<p data-nodeid="82962">当使用独享连接的时候，对每个 Service 建立固定数量的 Client，每个 Client 维护一个底层连接。如下图所示，就是针对每个 Service 都启动了两个独享连接：</p>
<p data-nodeid="84095"><img src="https://s0.lgstatic.com/i/image/M00/61/11/CgqCHl-OqnqAD_WFAAGYtk5Nou4688.png" alt="Lark20201020-171207.png" data-nodeid="84099"></p>
<div data-nodeid="84096" class=""><p style="text-align:center">Service 独享连接示意图</p></div>





<p data-nodeid="83524">当使用共享连接的时候，会区分不同的网络地址（host:port），一个地址只建立固定数量的共享连接。如下图所示，Provider 1 暴露了多个服务，Consumer 引用了 Provider 1 中的多个服务，共享连接是说 Consumer 调用 Provider 1 中的多个服务时，是通过固定数量的共享 TCP 长连接进行数据传输，这样就可以达到减少服务端连接数的目的。</p>
<p data-nodeid="83812"><img src="https://s0.lgstatic.com/i/image/M00/61/06/Ciqc1F-OqoOAHURKAAF2m0HX5qU972.png" alt="Lark20201020-171159.png" data-nodeid="83816"></p>
<div data-nodeid="83813" class=""><p style="text-align:center">Service 共享连接示意图</p></div>





<p data-nodeid="84376" class="">那怎么去创建共享连接呢？<strong data-nodeid="84386">创建共享连接的实现细节是在 getSharedClient() 方法中</strong>，它首先从 referenceClientMap 缓存（Map&lt;String, List<code data-backticks="1" data-nodeid="84384">&lt;ReferenceCountExchangeClient&gt;</code>&gt; 类型）中查询 Key（host 和 port 拼接成的字符串）对应的共享 Client 集合，如果查找到的 Client 集合全部可用，则直接使用这些缓存的 Client，否则要创建新的 Client 来补充替换缓存中不可用的 Client。示例代码如下：</p>

<pre class="lang-java" data-nodeid="81669"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> List&lt;ReferenceCountExchangeClient&gt; <span class="hljs-title">getSharedClient</span><span class="hljs-params">(URL url, <span class="hljs-keyword">int</span> connectNum)</span> </span>{
&nbsp; &nbsp; String key = url.getAddress(); <span class="hljs-comment">// 获取对端的地址(host:port)</span>
&nbsp; &nbsp; <span class="hljs-comment">// 从referenceClientMap集合中，获取与该地址连接的ReferenceCountExchangeClient集合</span>
&nbsp; &nbsp; List&lt;ReferenceCountExchangeClient&gt; clients = referenceClientMap.get(key);
    <span class="hljs-comment">// checkClientCanUse()方法中会检测clients集合中的客户端是否全部可用</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (checkClientCanUse(clients)) { 
&nbsp; &nbsp; &nbsp; &nbsp; batchClientRefIncr(clients); <span class="hljs-comment">// 客户端全部可用时</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> clients;
&nbsp; &nbsp; }
&nbsp; &nbsp; locks.putIfAbsent(key, <span class="hljs-keyword">new</span> Object());
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (locks.get(key)) { <span class="hljs-comment">// 针对指定地址的客户端进行加锁，分区加锁可以提高并发度</span>
&nbsp; &nbsp; &nbsp; &nbsp; clients = referenceClientMap.get(key);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (checkClientCanUse(clients)) { <span class="hljs-comment">// double check，再次检测客户端是否全部可用</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; batchClientRefIncr(clients); <span class="hljs-comment">// 增加应用Client的次数</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> clients;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; connectNum = Math.max(connectNum, <span class="hljs-number">1</span>); <span class="hljs-comment">// 至少一个共享连接</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果当前Clients集合为空，则直接通过initClient()方法初始化所有共享客户端</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (CollectionUtils.isEmpty(clients)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; clients = buildReferenceCountExchangeClientList(url, connectNum);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; referenceClientMap.put(key, clients);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 如果只有部分共享客户端不可用，则只需要处理这些不可用的客户端</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; clients.size(); i++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ReferenceCountExchangeClient referenceCountExchangeClient = clients.get(i);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (referenceCountExchangeClient == <span class="hljs-keyword">null</span> || referenceCountExchangeClient.isClosed()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; clients.set(i, buildReferenceCountExchangeClient(url));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 增加引用</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; referenceCountExchangeClient.incrementAndGetCount();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 清理locks集合中的锁对象，防止内存泄漏，如果key对应的服务宕机或是下线，</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 这里不进行清理的话，这个用于加锁的Object对象是无法被GC的，从而出现内存泄漏</span>
&nbsp; &nbsp; &nbsp; &nbsp; locks.remove(key);&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> clients;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="81670">这里使用的 ExchangeClient 实现是 ReferenceCountExchangeClient，它是 ExchangeClient 的一个装饰器，在原始 ExchangeClient 对象基础上添加了引用计数的功能。</p>
<p data-nodeid="85205">ReferenceCountExchangeClient 中除了持有被修饰的 ExchangeClient 对象外，还有一个 referenceCount 字段（AtomicInteger 类型），用于记录该 Client 被应用的次数。从下图中我们可以看到，在 ReferenceCountExchangeClient 的构造方法以及 incrementAndGetCount() 方法中会增加引用次数，在 close() 方法中则会减少引用次数。</p>
<p data-nodeid="85206" class=""><img src="https://s0.lgstatic.com/i/image/M00/61/06/Ciqc1F-OqqeAHAStAAF3BXy1LnA608.png" alt="Drawing 2.png" data-nodeid="85211"></p>
<div data-nodeid="85207"><p style="text-align:center">referenceCount 修改调用栈</p></div>





<p data-nodeid="81674">这样，对于同一个地址的共享连接，就可以满足两个基本需求：</p>
<ol data-nodeid="81675">
<li data-nodeid="81676">
<p data-nodeid="81677">当引用次数减到 0 的时候，ExchangeClient 连接关闭；</p>
</li>
<li data-nodeid="81678">
<p data-nodeid="81679">当引用次数未减到 0 的时候，底层的 ExchangeClient 不能关闭。</p>
</li>
</ol>
<p data-nodeid="81680">还有一个需要注意的细节是 ReferenceCountExchangeClient.close() 方法，在关闭底层 ExchangeClient 对象之后，会立即创建一个 LazyConnectExchangeClient ，也有人称其为“幽灵连接”。具体逻辑如下所示，这里的 LazyConnectExchangeClient 主要用于异常情况的兜底：</p>
<pre class="lang-java" data-nodeid="81681"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">(<span class="hljs-keyword">int</span> timeout)</span> </span>{
    <span class="hljs-comment">// 引用次数减到0，关闭底层的ExchangeClient，具体操作有：停掉心跳任务、重连任务以及关闭底层Channel，这些在前文介绍HeaderExchangeClient的时候已经详细分析过了，这里不再赘述</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (referenceCount.decrementAndGet() &lt;= <span class="hljs-number">0</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; client.close();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; client.close(timeout);
&nbsp; &nbsp; &nbsp; &nbsp; } 
        <span class="hljs-comment">// 创建LazyConnectExchangeClient，并将client字段指向该对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; replaceWithLazyClient(); 
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">replaceWithLazyClient</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 在原有的URL之上，添加一些LazyConnectExchangeClient特有的参数</span>
&nbsp; &nbsp; URL lazyUrl = URLBuilder.from(url)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(LAZY_CONNECT_INITIAL_STATE_KEY, Boolean.TRUE)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(RECONNECT_KEY, Boolean.FALSE)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(SEND_RECONNECT_KEY, Boolean.TRUE.toString())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(<span class="hljs-string">"warning"</span>, Boolean.TRUE.toString())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(LazyConnectExchangeClient.REQUEST_WITH_WARNING_KEY, <span class="hljs-keyword">true</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(<span class="hljs-string">"_client_memo"</span>, <span class="hljs-string">"referencecounthandler.replacewithlazyclient"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build();
&nbsp; &nbsp; <span class="hljs-comment">// 如果当前client字段已经指向了LazyConnectExchangeClient，则不需要再次创建LazyConnectExchangeClient兜底了</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!(client <span class="hljs-keyword">instanceof</span> LazyConnectExchangeClient) || client.isClosed()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// ChannelHandler依旧使用原始ExchangeClient使用的Handler，即DubboProtocol中的requestHandler字段</span>
&nbsp; &nbsp; &nbsp; &nbsp; client = <span class="hljs-keyword">new</span> LazyConnectExchangeClient(lazyUrl, client.getExchangeHandler());
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="86019">LazyConnectExchangeClient 也是 ExchangeClient 的装饰器，它会在原有 ExchangeClient 对象的基础上添加懒加载的功能。LazyConnectExchangeClient 在构造方法中不会创建底层持有连接的 Client，而是在需要发送请求的时候，才会调用 initClient() 方法进行 Client 的创建，如下图调用关系所示：</p>
<p data-nodeid="86020" class=""><img src="https://s0.lgstatic.com/i/image/M00/61/11/CgqCHl-OqrqAHcvUAAC9KpqKEBQ887.png" alt="Drawing 3.png" data-nodeid="86025"></p>
<div data-nodeid="86021"><p style="text-align:center">initClient() 方法的调用位置</p></div>





<p data-nodeid="81685">initClient() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="81686"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initClient</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (client != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 底层Client已经初始化过了，这里不再初始化</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; connectLock.lock();
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (client != <span class="hljs-keyword">null</span>) { <span class="hljs-keyword">return</span>; } <span class="hljs-comment">// double check</span>
        <span class="hljs-comment">// 通过Exchangers门面类，创建ExchangeClient对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.client = Exchangers.connect(url, requestHandler);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; connectLock.unlock();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="81687">在这些发送请求的方法中，除了通过 initClient() 方法初始化底层 ExchangeClient 外，还会调用warning() 方法，其会根据当前 URL 携带的参数决定是否打印 WARN 级别日志。为了防止瞬间打印大量日志的情况发生，这里有打印的频率限制，默认每发送 5000 次请求打印 1 条日志。你可以看到在前面展示的兜底场景中，我们就开启了打印日志的选项。</p>
<p data-nodeid="81688"><strong data-nodeid="81779">分析完 getSharedClient() 方法创建共享 Client 的核心流程之后，我们回到 DubboProtocol 中，继续介绍创建独享 Client 的流程。</strong></p>
<p data-nodeid="81689">创建独享 Client 的入口在<strong data-nodeid="81789">DubboProtocol.initClient() 方法</strong>，它首先会在 URL 中设置一些默认的参数，然后根据 LAZY_CONNECT_KEY 参数决定是否使用 LazyConnectExchangeClient 进行封装，实现懒加载功能，如下代码所示：</p>
<pre class="lang-java" data-nodeid="81690"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> ExchangeClient <span class="hljs-title">initClient</span><span class="hljs-params">(URL url)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 获取客户端扩展名并进行检查，省略检测的逻辑</span>
&nbsp; &nbsp; String str = url.getParameter(CLIENT_KEY, url.getParameter(SERVER_KEY, DEFAULT_REMOTING_CLIENT));
&nbsp; &nbsp; <span class="hljs-comment">// 设置Codec2的扩展名</span>
&nbsp; &nbsp; url = url.addParameter(CODEC_KEY, DubboCodec.NAME);
&nbsp; &nbsp; <span class="hljs-comment">// 设置默认的心跳间隔</span>
&nbsp; &nbsp; url = url.addParameterIfAbsent(HEARTBEAT_KEY, String.valueOf(DEFAULT_HEARTBEAT));
&nbsp; &nbsp; ExchangeClient client;&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">// 如果配置了延迟创建连接的特性，则创建LazyConnectExchangeClient</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (url.getParameter(LAZY_CONNECT_KEY, <span class="hljs-keyword">false</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; client = <span class="hljs-keyword">new</span> LazyConnectExchangeClient(url, requestHandler);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 未使用延迟连接功能，则直接创建HeaderExchangeClient</span>
&nbsp; &nbsp; &nbsp; &nbsp; client = Exchangers.connect(url, requestHandler);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> client;
}
</code></pre>
<p data-nodeid="81691">这里涉及的 LazyConnectExchangeClient 装饰器以及 Exchangers 门面类在前面已经深入分析过了，就不再赘述了。</p>
<p data-nodeid="81692">DubboProtocol 中还剩下几个方法没有介绍，这里你只需要简单了解一下它们的实现即可。</p>
<ul data-nodeid="81693">
<li data-nodeid="81694">
<p data-nodeid="81695">batchClientRefIncr() 方法：会遍历传入的集合，将其中的每个 ReferenceCountExchangeClient 对象的引用加一。</p>
</li>
<li data-nodeid="81696">
<p data-nodeid="81697">buildReferenceCountExchangeClient() 方法：会调用上面介绍的 initClient() 创建 Client 对象，然后再包装一层 ReferenceCountExchangeClient 进行修饰，最后返回。该方法主要用于创建共享 Client。</p>
</li>
</ul>
<h3 data-nodeid="81698">destroy方法</h3>
<p data-nodeid="81699">在 DubboProtocol 销毁的时候，会调用 destroy() 方法释放底层资源，其中就涉及 export 流程中创建的 ProtocolServer 对象以及 refer 流程中创建的 Client。</p>
<p data-nodeid="81700">DubboProtocol.destroy() 方法首先会逐个关闭 serverMap 集合中的 ProtocolServer 对象，相关代码片段如下：</p>
<pre class="lang-java" data-nodeid="81701"><code data-language="java"><span class="hljs-keyword">for</span> (String key : <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(serverMap.keySet())) {
&nbsp; &nbsp; ProtocolServer protocolServer = serverMap.remove(key);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (protocolServer == <span class="hljs-keyword">null</span>) { <span class="hljs-keyword">continue</span>;}
&nbsp; &nbsp; RemotingServer server = protocolServer.getRemotingServer();
    <span class="hljs-comment">// 在close()方法中，发送ReadOnly请求、阻塞指定时间、关闭底层的定时任务、关闭相关线程池，最终，会断开所有连接，关闭Server。这些逻辑在前文介绍HeaderExchangeServer、NettyServer等实现的时候，已经详细分析过了，这里不再展开</span>
&nbsp; &nbsp; server.close(ConfigurationUtils.getServerShutdownTimeout());
}
</code></pre>
<p data-nodeid="81702">ConfigurationUtils.getServerShutdownTimeout() 方法返回的阻塞时长默认是 10 秒，我们可以通过 dubbo.service.shutdown.wait 或是 dubbo.service.shutdown.wait.seconds 进行配置。</p>
<p data-nodeid="81703">之后，DubboProtocol.destroy() 方法会逐个关闭 referenceClientMap 集合中的 Client，逻辑与上述关闭ProtocolServer的逻辑相同，这里不再重复。只不过需要注意前面我们提到的 ReferenceCountExchangeClient 的存在，只有引用减到 0，底层的 Client 才会真正销毁。</p>
<p data-nodeid="81704">最后，DubboProtocol.destroy() 方法会调用父类 AbstractProtocol 的 destroy() 方法，销毁全部 Invoker 对象，前面已经介绍过 AbstractProtocol.destroy() 方法的实现，这里也不再重复。</p>
<h3 data-nodeid="81705">总结</h3>
<p data-nodeid="81706">本课时我们继续上一课时的话题，以 DubboProtocol 为例，介绍了 Dubbo 在 Protocol 层实现服务引用的核心流程。我们首先介绍了 DubboProtocol 初始化 Client 的核心逻辑，分析了共享连接和独立连接的模型，后续还讲解了ReferenceCountExchangeClient、LazyConnectExchangeClient 等装饰器的功能和实现，最后说明了 destroy() 方法释放底层资源的相关实现。</p>
<p data-nodeid="81707">关于 DubboProtocol，你若还有什么疑问或想法，欢迎你留言跟我分享。下一课时，我们将开始深入介绍 Dubbo 的“心脏”—— Invoker 接口的相关实现，这是我们的一篇加餐文章，记得按时来听课。</p>

---

### 精选评论


