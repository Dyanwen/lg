<p data-nodeid="25746" class="">在上一课时我们讲解了 Protocol 的核心接口，那本课时我们就以 Protocol 接口为核心，详细介绍整个 Protocol 的核心实现。下图展示了 Protocol 接口的继承关系：</p>
<p data-nodeid="25747"><img src="https://s0.lgstatic.com/i/image/M00/5D/C5/Ciqc1F-FTHGAOVGKAAJe5PD5u9A015.png" alt="Drawing 0.png" data-nodeid="25864"></p>
<div data-nodeid="25748"><p style="text-align:center">Protocol 接口继承关系图</p></div>
<p data-nodeid="25749">其中，<strong data-nodeid="25870">AbstractProtocol</strong>提供了一些 Protocol 实现需要的公共能力以及公共字段，它的核心字段有如下三个。</p>
<ul data-nodeid="25750">
<li data-nodeid="25751">
<p data-nodeid="25752">exporterMap（Map&lt;String, Exporter&lt;?&gt;&gt;类型）：用于存储出去的服务集合，其中的 Key 通过 ProtocolUtils.serviceKey() 方法创建的服务标识，在 ProtocolUtils 中维护了多层的 Map 结构（如下图所示）。首先按照 group 分组，在实践中我们可以根据需求设置 group，例如，按照机房、地域等进行 group 划分，做到就近调用；在 GroupServiceKeyCache 中，依次按照 serviceName、serviceVersion、port 进行分类，最终缓存的 serviceKey 是前面三者拼接而成的。</p>
</li>
</ul>
<p data-nodeid="28924"><img src="https://s0.lgstatic.com/i/image/M00/5F/74/Ciqc1F-JXfmAJK8RAAHUliqXmBc629.png" alt="Lark20201016-164613.png" data-nodeid="28927"></p>

<div data-nodeid="28340"><p style="text-align:center">groupServiceKeyCacheMap 结构图</p></div>




<ul data-nodeid="25755">
<li data-nodeid="25756">
<p data-nodeid="25757">serverMap（Map&lt;String, ProtocolServer&gt;类型）：记录了全部的 ProtocolServer 实例，其中的 Key 是 host 和 port 组成的字符串，Value 是监听该地址的 ProtocolServer。ProtocolServer 就是对 RemotingServer 的一层简单封装，表示一个服务端。</p>
</li>
<li data-nodeid="25758">
<p data-nodeid="25759">invokers（Set&lt;Invoker&lt;?&gt;&gt;类型）：服务引用的集合。</p>
</li>
</ul>
<p data-nodeid="25760">AbstractProtocol 没有对 Protocol 的 export() 方法进行实现，对 refer() 方法的实现也是委托给了 protocolBindingRefer() 这个抽象方法，然后由子类实现。AbstractProtocol 唯一实现的方法就是 destory() 方法，其首先会遍历 Invokers 集合，销毁全部的服务引用，然后遍历全部的 exporterMap 集合，销毁发布出去的服务，具体实现如下：</p>
<pre class="lang-java" data-nodeid="25761"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">destroy</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Invoker&lt;?&gt; invoker : invokers) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (invoker != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invokers.remove(invoker);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invoker.destroy(); <span class="hljs-comment">// 关闭全部的服务引用</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (String key : <span class="hljs-keyword">new</span> ArrayList&lt;String&gt;(exporterMap.keySet())) {
&nbsp; &nbsp; &nbsp; &nbsp; Exporter&lt;?&gt; exporter = exporterMap.remove(key);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (exporter != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; exporter.unexport(); <span class="hljs-comment">// 关闭暴露出去的服务</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="25762">export 流程简析</h3>
<p data-nodeid="25763">了解了 AbstractProtocol 提供的公共能力之后，我们再来分析<strong data-nodeid="25894">Dubbo 默认使用的 Protocol 实现类—— DubboProtocol 实现</strong>。这里我们首先关注 DubboProtocol 的 export() 方法，也就是服务发布的相关实现，如下所示：</p>
<pre class="lang-java" data-nodeid="25764"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">Exporter&lt;T&gt; <span class="hljs-title">export</span><span class="hljs-params">(Invoker&lt;T&gt; invoker)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; URL url = invoker.getUrl();
    <span class="hljs-comment">// 创建ServiceKey，其核心实现在前文已经详细分析过了，这里不再重复</span>
&nbsp; &nbsp; String key = serviceKey(url); 
&nbsp; &nbsp; <span class="hljs-comment">// 将上层传入的Invoker对象封装成DubboExporter对象，然后记录到exporterMap集合中</span>
&nbsp; &nbsp; DubboExporter&lt;T&gt; exporter = <span class="hljs-keyword">new</span> DubboExporter&lt;T&gt;(invoker, key, exporterMap);
&nbsp; &nbsp; exporterMap.put(key, exporter);
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略一些日志操作</span>

&nbsp; &nbsp; <span class="hljs-comment">// 启动ProtocolServer</span>
&nbsp; &nbsp; openServer(url);
&nbsp; &nbsp; <span class="hljs-comment">// 进行序列化的优化处理</span>
&nbsp; &nbsp; optimizeSerialization(url);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> exporter;
}
</code></pre>
<h4 data-nodeid="25765">1. DubboExporter</h4>
<p data-nodeid="25766">这里涉及的第一个点是 DubboExporter 对 Invoker 的封装，DubboExporter 的继承关系如下图所示：</p>
<p data-nodeid="25767"><img src="https://s0.lgstatic.com/i/image/M00/5D/D0/CgqCHl-FTJSAd9oTAAAm0DgOmVo715.png" alt="Drawing 2.png" data-nodeid="25901"></p>
<div data-nodeid="25768"><p style="text-align:center">DubboExporter 继承关系图</p></div>
<p data-nodeid="25769">AbstractExporter 中维护了一个 Invoker 对象，以及一个 unexported 字段（boolean 类型），在 unexport() 方法中会设置 unexported 字段为 true，并调用 Invoker 对象的 destory() 方法进行销毁。</p>
<p data-nodeid="25770">DubboExporter 也比较简单，其中会维护底层 Invoker 对应的 ServiceKey 以及 DubboProtocol 中的 exportMap 集合，在其 unexport() 方法中除了会调用父类 AbstractExporter 的 unexport() 方法之外，还会清理该 DubboExporter 实例在 exportMap 中相应的元素。</p>
<h4 data-nodeid="25771">2. 服务端初始化</h4>
<p data-nodeid="25772">了解了 Exporter 实现之后，我们继续看 DubboProtocol 中服务发布的流程。从下面这张调用关系图中可以看出，openServer() 方法会一路调用前面介绍的 Exchange 层、Transport 层，并最终创建 NettyServer 来接收客户端的请求。</p>
<p data-nodeid="25773"><img src="https://s0.lgstatic.com/i/image/M00/5D/D0/CgqCHl-FTKGAJNO8AAElldtvsRM104.png" alt="Drawing 3.png" data-nodeid="25910"></p>
<div data-nodeid="25774"><p style="text-align:center">export() 方法调用栈</p></div>
<p data-nodeid="25775">下面我们将逐个介绍 export() 方法栈中的每个被调用的方法。</p>
<p data-nodeid="25776">首先，在 openServer() 方法中会根据 URL 判断当前是否为服务端，只有服务端才能创建 ProtocolServer 并对外服务。如果是来自服务端的调用，会依靠 serverMap 集合检查是否已有 ProtocolServer 在监听 URL 指定的地址；如果没有，会调用 createServer() 方法进行创建。openServer() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="25777"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">openServer</span><span class="hljs-params">(URL url)</span> </span>{
&nbsp; &nbsp; String key = url.getAddress(); <span class="hljs-comment">// 获取host:port这个地址</span>
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> isServer = url.getParameter(IS_SERVER_KEY, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isServer) { <span class="hljs-comment">// 只有Server端才能启动Server对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; ProtocolServer server = serverMap.get(key);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (server == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 无ProtocolServer监听该地址</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) { <span class="hljs-comment">// DoubleCheck，防止并发问题</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; server = serverMap.get(key);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (server == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用createServer()方法创建ProtocolServer对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; serverMap.put(key, createServer(url));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果已有ProtocolServer实例，则尝试根据URL信息重置ProtocolServer</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; server.reset(url);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="25778">createServer() 方法首先会为 URL 添加一些默认值，同时会进行一些参数值的检测，主要有五个。</p>
<ul data-nodeid="25779">
<li data-nodeid="25780">
<p data-nodeid="25781">HEARTBEAT_KEY 参数值，默认值为 60000，表示默认的心跳时间间隔为 60 秒。</p>
</li>
<li data-nodeid="25782">
<p data-nodeid="25783">CHANNEL_READONLYEVENT_SENT_KEY 参数值，默认值为 true，表示 ReadOnly 请求需要阻塞等待响应返回。在 Server 关闭的时候，只能发送 ReadOnly 请求，这些 ReadOnly 请求由这里设置的 CHANNEL_READONLYEVENT_SENT_KEY 参数值决定是否需要等待响应返回。</p>
</li>
<li data-nodeid="25784">
<p data-nodeid="25785">CODEC_KEY 参数值，默认值为 dubbo。你可以回顾 Codec2 接口中 @Adaptive 注解的参数，都是获取该 URL 中的 CODEC_KEY 参数值。</p>
</li>
<li data-nodeid="25786">
<p data-nodeid="25787">检测 SERVER_KEY 参数指定的扩展实现名称是否合法，默认值为 netty。你可以回顾 Transporter 接口中 @Adaptive 注解的参数，它决定了 Transport 层使用的网络库实现，默认使用 Netty 4 实现。</p>
</li>
<li data-nodeid="25788">
<p data-nodeid="25789">检测 CLIENT_KEY 参数指定的扩展实现名称是否合法。同 SERVER_KEY 参数的检查流程。</p>
</li>
</ul>
<p data-nodeid="25790">完成上述默认参数值的设置之后，我们就可以通过 Exchangers 门面类创建 ExchangeServer，并封装成 DubboProtocolServer 返回。</p>
<pre class="lang-java" data-nodeid="25791"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> ProtocolServer <span class="hljs-title">createServer</span><span class="hljs-params">(URL url)</span> </span>{
&nbsp; &nbsp; url = URLBuilder.from(url)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// ReadOnly请求是否阻塞等待</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameterIfAbsent(CHANNEL_READONLYEVENT_SENT_KEY, Boolean.TRUE.toString())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 心跳间隔</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameterIfAbsent(HEARTBEAT_KEY, String.valueOf(DEFAULT_HEARTBEAT))
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addParameter(CODEC_KEY, DubboCodec.NAME) <span class="hljs-comment">// Codec2扩展实现</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build();
&nbsp; &nbsp; <span class="hljs-comment">// 检测SERVER_KEY参数指定的Transporter扩展实现是否合法</span>
&nbsp; &nbsp; String str = url.getParameter(SERVER_KEY, DEFAULT_REMOTING_SERVER);&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (str != <span class="hljs-keyword">null</span> &amp;&amp; str.length() &gt; <span class="hljs-number">0</span> &amp;&amp; !ExtensionLoader.getExtensionLoader(Transporter.class).hasExtension(str)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RpcException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 通过Exchangers门面类，创建ExchangeServer对象</span>
&nbsp; &nbsp; ExchangeServer server = Exchangers.bind(url, requestHandler);
&nbsp; &nbsp; ... <span class="hljs-comment">// 检测CLIENT_KEY参数指定的Transporter扩展实现是否合法(略)</span>
&nbsp; &nbsp; <span class="hljs-comment">// 将ExchangeServer封装成DubboProtocolServer返回</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DubboProtocolServer(server);
}
</code></pre>
<p data-nodeid="25792">在 createServer() 方法中还有几个细节需要展开分析一下。第一个是创建 ExchangeServer 时，使用的 Codec2 接口实现实际上是 DubboCountCodec，对应的 SPI 配置文件如下：</p>
<p data-nodeid="25793"><img src="https://s0.lgstatic.com/i/image/M00/5D/D0/CgqCHl-FTK-AUlLCAADTWhhySe8432.png" alt="Drawing 4.png" data-nodeid="25947"></p>
<div data-nodeid="25794"><p style="text-align:center">Codec2 SPI 配置文件</p></div>
<p data-nodeid="25795">DubboCountCodec 中维护了一个 DubboCodec 对象，编解码的能力都是 DubboCodec 提供的，DubboCountCodec 只负责在解码过程中 ChannelBuffer 的 readerIndex 指针控制，具体实现如下：</p>
<pre class="lang-java" data-nodeid="25796"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">decode</span><span class="hljs-params">(Channel channel, ChannelBuffer buffer)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> save = buffer.readerIndex(); <span class="hljs-comment">// 首先保存readerIndex指针位置</span>
&nbsp; &nbsp; <span class="hljs-comment">// 创建MultiMessage对象，其中可以存储多条消息</span>
&nbsp; &nbsp; MultiMessage result = MultiMessage.create();&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 通过DubboCodec提供的解码能力解码一条消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; Object obj = codec.decode(channel, buffer);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果可读字节数不足一条消息，则会重置readerIndex指针</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (Codec2.DecodeResult.NEED_MORE_INPUT == obj) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; buffer.readerIndex(save);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 将成功解码的消息添加到MultiMessage中暂存</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; result.addMessage(obj);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logMessageLength(obj, buffer.readerIndex() - save);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; save = buffer.readerIndex();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">while</span> (<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (result.isEmpty()) { <span class="hljs-comment">// 一条消息也未解码出来，则返回NEED_MORE_INPUT错误码</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Codec2.DecodeResult.NEED_MORE_INPUT;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (result.size() == <span class="hljs-number">1</span>) { <span class="hljs-comment">// 只解码出来一条消息，则直接返回该条消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> result.get(<span class="hljs-number">0</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 解码出多条消息的话，会将MultiMessage返回</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="25797">DubboCountCodec、DubboCodec 都实现了第 22 课时介绍的 Codec2 接口，其中 DubboCodec 是 ExchangeCodec 的子类。</p>
<p data-nodeid="25798"><img src="https://s0.lgstatic.com/i/image/M00/5D/C5/Ciqc1F-FTLuAZ-AoAACeZ02hpEg723.png" alt="Drawing 5.png" data-nodeid="25952"></p>
<div data-nodeid="25799"><p style="text-align:center">DubboCountCodec 及 DubboCodec 继承关系图</p></div>
<p data-nodeid="25800">我们知道 ExchangeCodec 只处理了 Dubbo 协议的请求头，而 DubboCodec 则是通过继承的方式，在 ExchangeCodec 基础之上，添加了解析 Dubbo 消息体的功能。在第 22 课时介绍 ExchangeCodec 实现的时候，我们重点分析了 encodeRequest() 方法，即 Request 请求的编码实现，其中会调用 encodeRequestData() 方法完成请求体的编码。</p>
<p data-nodeid="25801">DubboCodec 中就覆盖了 encodeRequestData() 方法，按照 Dubbo 协议的格式编码 Request 请求体，具体实现如下：</p>
<pre class="lang-java" data-nodeid="25802"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">encodeRequestData</span><span class="hljs-params">(Channel channel, ObjectOutput out, Object data, String version)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 请求体相关的内容，都封装在了RpcInvocation</span>
&nbsp; &nbsp; RpcInvocation inv = (RpcInvocation) data;&nbsp;
&nbsp; &nbsp; out.writeUTF(version); <span class="hljs-comment">// 写入版本号</span>
&nbsp; &nbsp; String serviceName = inv.getAttachment(INTERFACE_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (serviceName == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; serviceName = inv.getAttachment(PATH_KEY);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 写入服务名称</span>
&nbsp; &nbsp; out.writeUTF(serviceName);
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;写入Service版本号</span>
&nbsp; &nbsp; out.writeUTF(inv.getAttachment(VERSION_KEY));
&nbsp; &nbsp; <span class="hljs-comment">// 写入方法名称</span>
&nbsp; &nbsp; out.writeUTF(inv.getMethodName());
&nbsp; &nbsp; <span class="hljs-comment">// 写入参数类型列表</span>
&nbsp; &nbsp; out.writeUTF(inv.getParameterTypesDesc());
&nbsp; &nbsp; <span class="hljs-comment">// 依次写入全部参数</span>
&nbsp; &nbsp; Object[] args = inv.getArguments();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (args != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; args.length; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.writeObject(encodeInvocationArgument(channel, inv, i));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 依次写入全部的附加信息</span>
&nbsp; &nbsp; out.writeAttachments(inv.getObjectAttachments());
}
</code></pre>
<p data-nodeid="25803">RpcInvocation 实现了上一课时介绍的 Invocation 接口，如下图所示：</p>
<p data-nodeid="27761" class=""><img src="https://s0.lgstatic.com/i/image/M00/5D/D0/CgqCHl-FTMSAYeP7AAA_pzU2CPA016.png" alt="Drawing 6.png" data-nodeid="27765"></p>
<div data-nodeid="27762"><p style="text-align:center">RpcInvocation 继承关系图</p></div>








<p data-nodeid="25806">下面是 RpcInvocation 中的核心字段，通过读写这些字段即可实现 Invocation 接口的全部方法。</p>
<ul data-nodeid="25807">
<li data-nodeid="25808">
<p data-nodeid="25809">targetServiceUniqueName（String类型）：要调用的唯一服务名称，其实就是 ServiceKey，即 interface/group:version 三部分构成的字符串。</p>
</li>
<li data-nodeid="25810">
<p data-nodeid="25811">methodName（String类型）：调用的目标方法名称。</p>
</li>
<li data-nodeid="25812">
<p data-nodeid="25813">serviceName（String类型）：调用的目标服务名称，示例中就是org.apache.dubbo.demo.DemoService。</p>
</li>
<li data-nodeid="25814">
<p data-nodeid="25815">parameterTypes（Class&lt;?&gt;[]类型）：记录了目标方法的全部参数类型。</p>
</li>
<li data-nodeid="25816">
<p data-nodeid="25817">parameterTypesDesc（String类型）：参数列表签名。</p>
</li>
<li data-nodeid="25818">
<p data-nodeid="25819">arguments（Object[]类型）：具体参数值。</p>
</li>
<li data-nodeid="25820">
<p data-nodeid="25821">attachments（Map&lt;String, Object&gt;类型）：此次调用的附加信息，可以被序列化到请求中。</p>
</li>
<li data-nodeid="25822">
<p data-nodeid="25823">attributes（Map&lt;Object, Object&gt;类型）：此次调用的属性信息，这些信息不能被发送出去。</p>
</li>
<li data-nodeid="25824">
<p data-nodeid="25825">invoker（Invoker&lt;?&gt;类型）：此次调用关联的 Invoker 对象。</p>
</li>
<li data-nodeid="25826">
<p data-nodeid="25827">returnType（Class&lt;?&gt;类型）：返回值的类型。</p>
</li>
<li data-nodeid="25828">
<p data-nodeid="25829">invokeMode（InvokeMode类型）：此次调用的模式，分为 SYNC、ASYNC 和 FUTURE 三类。</p>
</li>
</ul>
<p data-nodeid="25830">我们在上面的继承图中看到 RpcInvocation 的一个子类—— DecodeableRpcInvocation，它是用来支持解码的，其实现的 decode() 方法正好是 DubboCodec.encodeRequestData() 方法对应的解码操作，在 DubboCodec.decodeBody() 方法中就调用了这个方法，调用关系如下图所示：</p>
<p data-nodeid="25831"><img src="https://s0.lgstatic.com/i/image/M00/5D/C5/Ciqc1F-FTM2Ae73pAAC0_daI0N4088.png" alt="Drawing 7.png" data-nodeid="25990"></p>
<div data-nodeid="25832"><p style="text-align:center">decode() 方法调用栈</p></div>
<p data-nodeid="25833">这个解码过程中有个细节，在 DubboCodec.decodeBody() 方法中有如下代码片段，其中会根据 DECODE_IN_IO_THREAD_KEY 这个参数决定是否在 DubboCodec 中进行解码（DubboCodec 是在 IO 线程中调用的）。</p>
<pre class="lang-java" data-nodeid="25834"><code data-language="java"><span class="hljs-comment">// decode request.</span>
Request req = <span class="hljs-keyword">new</span> Request(id);
... <span class="hljs-comment">// 省略Request中其他字段的设置</span>
Object data;
DecodeableRpcInvocation inv;
<span class="hljs-comment">// 这里会检查DECODE_IN_IO_THREAD_KEY参数</span>
<span class="hljs-keyword">if</span> (channel.getUrl().getParameter(DECODE_IN_IO_THREAD_KEY, DEFAULT_DECODE_IN_IO_THREAD)) {
&nbsp; &nbsp; inv = <span class="hljs-keyword">new</span> DecodeableRpcInvocation(channel, req, is, proto);
&nbsp; &nbsp; inv.decode(); <span class="hljs-comment">// 直接调用decode()方法在当前IO线程中解码</span>
} <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 这里只是读取数据，不会调用decode()方法在当前IO线程中进行解码</span>
&nbsp; &nbsp; inv = <span class="hljs-keyword">new</span> DecodeableRpcInvocation(channel, req,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> UnsafeByteArrayInputStream(readMessageData(is)), proto);
}
data = inv;
req.setData(data); <span class="hljs-comment">// 设置到Request请求的data字段</span>
<span class="hljs-keyword">return</span> req;
</code></pre>
<p data-nodeid="25835">如果不在 DubboCodec 中解码，那会在哪里解码呢？你可以回顾第 20 课时介绍的 DecodeHandler（Transport 层），它的 received() 方法也是可以进行解码的，另外，DecodeableRpcInvocation 中有一个 hasDecoded 字段来判断当前是否已经完成解码，这样，三者配合就可以根据 DECODE_IN_IO_THREAD_KEY 参数决定执行解码操作的线程了。</p>
<p data-nodeid="25836">如果你对线程模型不清楚，可以依次回顾一下 Exchangers、HeaderExchanger、Transporters 三个门面类的 bind() 方法，以及 Dispatcher 各实现提供的线程模型，搞清楚各个 ChannelHandler 是由哪个线程执行的，这些知识点在前面课时都介绍过了，不再重复。这里我们就直接以 AllDispatcher 实现为例给出结论。</p>
<ul data-nodeid="25837">
<li data-nodeid="25838">
<p data-nodeid="25839">IO 线程内执行的 ChannelHandler 实现依次有：InternalEncoder、InternalDecoder（两者底层都是调用 DubboCodec）、IdleStateHandler、MultiMessageHandler、HeartbeatHandler 和 NettyServerHandler。</p>
</li>
<li data-nodeid="25840">
<p data-nodeid="25841">在非 IO 线程内执行的 ChannelHandler 实现依次有：DecodeHandler、HeaderExchangeHandler 和 DubboProtocol$requestHandler。</p>
</li>
</ul>
<p data-nodeid="25842">在 DubboProtocol 中有一个 requestHandler 字段，它是一个实现了 ExchangeHandlerAdapter 抽象类的匿名内部类的实例，间接实现了 ExchangeHandler 接口，其核心是 reply() 方法，具体实现如下：</p>
<pre class="lang-java" data-nodeid="25843"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> CompletableFuture&lt;Object&gt; <span class="hljs-title">reply</span><span class="hljs-params">(ExchangeChannel channel, Object message)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; ... <span class="hljs-comment">// 这里省略了检查message类型的逻辑，通过前面Handler的处理，这里收到的message必须是Invocation类型的对象</span>
&nbsp; &nbsp; Invocation inv = (Invocation) message;
&nbsp; &nbsp; <span class="hljs-comment">// 获取此次调用Invoker对象</span>
&nbsp; &nbsp; Invoker&lt;?&gt; invoker = getInvoker(channel, inv);
&nbsp; &nbsp; ... <span class="hljs-comment">// 针对客户端回调的内容，在后面详细介绍，这里不再展开分析</span>
&nbsp; &nbsp; <span class="hljs-comment">// 将客户端的地址记录到RpcContext中</span>
&nbsp; &nbsp; RpcContext.getContext().setRemoteAddress(channel.getRemoteAddress());
&nbsp; &nbsp; <span class="hljs-comment">// 执行真正的调用</span>
&nbsp; &nbsp; Result result = invoker.invoke(inv);
&nbsp; &nbsp; <span class="hljs-comment">// 返回结果</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> result.thenApply(Function.identity());
}
</code></pre>
<p data-nodeid="25844">其中 getInvoker() 方法会先根据 Invocation 携带的信息构造 ServiceKey，然后从 exporterMap 集合中查找对应的 DubboExporter 对象，并从中获取底层的 Invoker 对象返回，具体实现如下：</p>
<pre class="lang-java" data-nodeid="25845"><code data-language="java">Invoker&lt;?&gt; getInvoker(Channel channel, Invocation inv) <span class="hljs-keyword">throws</span> RemotingException {
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略对客户端Callback以及stub的处理逻辑，后面单独介绍</span>
&nbsp; &nbsp; String serviceKey = serviceKey(port, path, (String) inv.getObjectAttachments().get(VERSION_KEY),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; (String) inv.getObjectAttachments().get(GROUP_KEY));
&nbsp; &nbsp; DubboExporter&lt;?&gt; exporter = (DubboExporter&lt;?&gt;) exporterMap.get(serviceKey);
&nbsp; &nbsp; ... <span class="hljs-comment">//&nbsp; 查找不到相应的DubboExporter对象时，会直接抛出异常，这里省略了这个检测</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> exporter.getInvoker(); <span class="hljs-comment">// 获取exporter中获取Invoker对象</span>
}
</code></pre>
<p data-nodeid="25846">到这里，我们终于见到了对 Invoker 对象的调用，对 Invoker 实现的介绍和分析，在后面课时我们会深入介绍，这里就先专注于 DubboProtocol 的相关内容。</p>
<h4 data-nodeid="25847">3. 序列化优化处理</h4>
<p data-nodeid="25848">下面我们回到 DubboProtocol.export() 方法继续分析，在完成 ProtocolServer 的启动之后，export() 方法最后会调用 optimizeSerialization() 方法对指定的序列化算法进行优化。</p>
<p data-nodeid="25849">这里先介绍一个基础知识，在使用某些序列化算法（例如， Kryo、FST 等）时，为了让其能发挥出最佳的性能，最好将那些需要被序列化的类提前注册到 Dubbo 系统中。例如，我们可以通过一个实现了 SerializationOptimizer 接口的优化器，并在配置中指定该优化器，如下示例代码：</p>
<pre class="lang-java" data-nodeid="25850"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SerializationOptimizerImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SerializationOptimizer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Collection&lt;Class&gt; <span class="hljs-title">getSerializableClasses</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;Class&gt; classes = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; classes.add(xxxx.class); <span class="hljs-comment">// 添加需要被序列化的类</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> classes;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="25851">在 DubboProtocol.optimizeSerialization() 方法中，就会获取该优化器中注册的类，通知底层的序列化算法进行优化，序列化的性能将会被大大提升。当然，在进行序列化的时候，难免会级联到很多 Java 内部的类（例如，数组、各种集合类型等），Kryo、FST 等序列化算法已经自动将JDK 中的常用类进行了注册，所以无须重复注册它们。</p>
<p data-nodeid="25852">下面我们回头来看 optimizeSerialization() 方法，分析序列化优化操作的具体实现细节：</p>
<pre class="lang-java" data-nodeid="25853"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">optimizeSerialization</span><span class="hljs-params">(URL url)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
    <span class="hljs-comment">// 根据URL中的optimizer参数值，确定SerializationOptimizer接口的实现类</span>
&nbsp; &nbsp; String className = url.getParameter(OPTIMIZER_KEY, <span class="hljs-string">""</span>);
&nbsp; &nbsp; Class clazz = Thread.currentThread().getContextClassLoader().loadClass(className);
    <span class="hljs-comment">// 创建SerializationOptimizer实现类的对象</span>
&nbsp; &nbsp; SerializationOptimizer optimizer = (SerializationOptimizer) clazz.newInstance();
    <span class="hljs-comment">// 调用getSerializableClasses()方法获取需要注册的类</span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Class c : optimizer.getSerializableClasses()) {
&nbsp; &nbsp; &nbsp; &nbsp; SerializableClassRegistry.registerClass(c); 
&nbsp; &nbsp; }
&nbsp; &nbsp; optimizers.add(className);
}
</code></pre>
<p data-nodeid="25854">SerializableClassRegistry 底层维护了一个 static 的 Map（REGISTRATIONS 字段），registerClass() 方法就是将待优化的类写入该集合中暂存，在使用 Kryo、FST 等序列化算法时，会读取该集合中的类，完成注册操作，相关的调用关系如下图所示：</p>
<p data-nodeid="25855"><img src="https://s0.lgstatic.com/i/image/M00/5D/C5/Ciqc1F-FTOGAEWu7AADOU3xBmjA069.png" alt="Drawing 8.png" data-nodeid="26025"></p>
<div data-nodeid="25856"><p style="text-align:center">getRegisteredClasses() 方法的调用位置</p></div>
<p data-nodeid="25857">按照 Dubbo 官方文档的说法，即使不注册任何类进行优化，Kryo 和 FST 的性能依然普遍优于Hessian2 和 Dubbo 序列化。</p>
<h3 data-nodeid="25858">总结</h3>
<p data-nodeid="25859">本课时我们重点介绍了 DubboProtocol 发布一个 Dubbo 服务的核心流程。首先，我们介绍了 AbstractProtocol 这个抽象类为 Protocol 实现类提供的公共能力和字段，然后我们结合 Dubbo 协议对应的 DubboProtocol 实现，讲解了发布一个 Dubbo 服务的核心流程，其中涉及整个服务端核心启动流程、RpcInvocation 实现、DubboProtocol.requestHandler 字段调用 Invoker 对象以及序列化相关的优化处理等内容。</p>
<p data-nodeid="25860" class="">下一课时，我们将继续介绍 DubboProtocol 引用服务的相关实现。</p>

---

### 精选评论

##### **豹：
> 老师您好，跟着课程一路学到现在，收获很大，课程制作非常用心。课程中老师方法调用栈的截图很精确，比如export() 方法调用栈，这个是怎么做到的呀？我这边进行方法调用栈追踪会有很多分支，希望老师指导下，谢谢！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以从最底层的 NettyServer 用 Call Hierarchy 往上找，也可以用 debug 的方式拿到这个调用栈

