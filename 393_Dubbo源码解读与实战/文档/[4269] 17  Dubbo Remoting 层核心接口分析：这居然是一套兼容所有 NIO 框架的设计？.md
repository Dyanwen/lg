<p data-nodeid="20279">在本专栏的第二部分，我们深入介绍了 Dubbo 注册中心的相关实现，下面我们开始介绍 dubbo-remoting 模块，该模块提供了多种客户端和服务端通信的功能。在 Dubbo 的整体架构设计图中，我们可以看到最底层红色框选中的部分即为 Remoting 层，其中包括了 Exchange、Transport和Serialize 三个子层次。这里我们要介绍的 dubbo-remoting 模块主要对应 Exchange 和 Transport 两层。</p>
<p data-nodeid="20765"><img src="https://s0.lgstatic.com/i/image/M00/55/06/CgqCHl9ptP2ADxEXAAuW94W_upc465.png" alt="Drawing 0.png" data-nodeid="20769"></p>
<div data-nodeid="20766" class=""><p style="text-align:center">Dubbo 整体架构设计图</p></div>





<p data-nodeid="18605">Dubbo 并没有自己实现一套完整的网络库，而是使用现有的、相对成熟的第三方网络库，例如，Netty、Mina 或是 Grizzly 等 NIO 框架。我们可以根据自己的实际场景和需求修改配置，选择底层使用的 NIO 框架。</p>
<p data-nodeid="21714">下图展示了 dubbo-remoting 模块的结构，其中每个子模块对应一个第三方 NIO 框架，例如，dubbo-remoting-netty4 子模块使用 Netty4 实现 Dubbo 的远程通信，dubbo-remoting-grizzly 子模块使用 Grizzly 实现 Dubbo 的远程通信。</p>
<p data-nodeid="21715" class=""><img src="https://s0.lgstatic.com/i/image/M00/54/FB/Ciqc1F9ptRqAJLQnAABcIxQfCkc811.png" alt="Drawing 1.png" data-nodeid="21719"></p>


<p data-nodeid="18608">其中的 dubbo-remoting-zookeeper，我们在前面第 15 课时介绍基于 Zookeeper 的注册中心实现时已经讲解过了，它使用 Apache Curator 实现了与 Zookeeper 的交互。</p>
<h3 data-nodeid="22196" class="">dubbo-remoting-api 模块</h3>

<p data-nodeid="25055">需要注意的是，<strong data-nodeid="25064">Dubbo 的 dubbo-remoting-api 是其他 dubbo-remoting-* 模块的顶层抽象，其他 dubbo-remoting 子模块都是依赖第三方 NIO 库实现 dubbo-remoting-api 模块的</strong>，依赖关系如下图所示：</p>
<p data-nodeid="49400"><img src="https://s0.lgstatic.com/i/image/M00/55/07/CgqCHl9ptY2ADzl8AAEVDPN3HVo908.png" alt="Drawing 2.png" data-nodeid="49403"></p>


<p data-nodeid="48920">我们先来看一下 dubbo-remoting-api 中对整个 Remoting 层的抽象，dubbo-remoting-api 模块的结构如下图所示：</p>








<p data-nodeid="48449"><img src="https://s0.lgstatic.com/i/image/M00/54/FD/Ciqc1F9ptduASJsQAACrkCpgiGg477.png" alt="Drawing 3.png" data-nodeid="48453"></p>



<p data-nodeid="47971">一般情况下，我们会将功能类似或是相关联的类放到一个包中，所以我们需要先来了解 dubbo-remoting-api 模块中各个包的功能。</p>





<ul data-nodeid="18615">
<li data-nodeid="18616">
<p data-nodeid="18617">buffer 包：定义了缓冲区相关的接口、抽象类以及实现类。缓冲区在NIO框架中是一个不可或缺的角色，在各个 NIO 框架中都有自己的缓冲区实现。这里的 buffer 包在更高的层面，抽象了各个 NIO 框架的缓冲区，同时也提供了一些基础实现。</p>
</li>
<li data-nodeid="18618">
<p data-nodeid="18619">exchange 包：抽象了 Request 和 Response 两个概念，并为其添加很多特性。这是<strong data-nodeid="18714">整个远程调用非常核心的部分</strong>。</p>
</li>
<li data-nodeid="18620">
<p data-nodeid="18621">transport 包：对网络传输层的抽象，但它只负责抽象单向消息的传输，即请求消息由 Client 端发出，Server 端接收；响应消息由 Server 端发出，Client端接收。有很多网络库可以实现网络传输的功能，例如 Netty、Grizzly 等， transport 包是在这些网络库上层的一层抽象。</p>
</li>
<li data-nodeid="18622">
<p data-nodeid="18623">其他接口：Endpoint、Channel、Transporter、Dispatcher 等顶层接口放到了org.apache.dubbo.remoting 这个包，这些接口是 Dubbo Remoting 的核心接口。</p>
</li>
</ul>
<p data-nodeid="18624">下面我们就来介绍 Dubbo 是如何抽象这些核心接口的。</p>
<h3 data-nodeid="27416" class="">传输层核心接口</h3>




<p data-nodeid="18627">在 Dubbo 中会抽象出一个“<strong data-nodeid="18736">端点（Endpoint）</strong>”的概念，我们可以通过一个 ip 和 port  唯一确定一个端点，两个端点之间会创建 TCP 连接，可以双向传输数据。Dubbo 将 Endpoint 之间的 TCP 连接抽象为<strong data-nodeid="18737">通道（Channel）</strong>，将发起请求的 Endpoint 抽象为<strong data-nodeid="18738">客户端（Client）</strong>，将接收请求的 Endpoint 抽象为<strong data-nodeid="18739">服务端（Server）</strong>。这些抽象出来的概念，也是整个 dubbo-remoting-api 模块的基础，下面我们会逐个进行介绍。</p>
<p data-nodeid="47013">Dubbo 中<strong data-nodeid="47020">Endpoint 接口</strong>的定义如下：</p>
<p data-nodeid="47014"><img src="https://s0.lgstatic.com/i/image/M00/55/08/CgqCHl9pteqACl0cAABxWeZ6ox0288.png" alt="Drawing 4.png" data-nodeid="47023"></p>


<p data-nodeid="46530">如上图所示，这里的 get*() 方法是获得 Endpoint 本身的一些属性，其中包括获取 Endpoint 的本地地址、关联的 URL 信息以及底层 Channel 关联的 ChannelHandler。send() 方法负责数据发送，两个重载的区别在后面介绍 Endpoint 实现的时候我们再详细说明。最后两个 close() 方法的重载以及 startClose() 方法用于关闭底层 Channel ，isClosed() 方法用于检测底层 Channel 是否已关闭。</p>




<p data-nodeid="50350">Channel 是对两个 Endpoint 连接的抽象，好比连接两个位置的传送带，两个 Endpoint 传输的消息就好比传送带上的货物，消息发送端会往 Channel 写入消息，而接收端会从 Channel 读取消息。这与第 10 课时介绍的 Netty 中的 Channel 基本一致。</p>
<p data-nodeid="50351" class=""><img src="https://s0.lgstatic.com/i/image/M00/55/09/CgqCHl9ptsaAeodMAACTIzdsI8g890.png" alt="Lark20200922-162359.png" data-nodeid="50355"></p>

<p data-nodeid="50110">下面是<strong data-nodeid="50117">Channel 接口</strong>的定义，我们可以看出两点：一个是 Channel 接口继承了 Endpoint 接口，也具备开关状态以及发送数据的能力；另一个是可以在 Channel 上附加 KV 属性。</p>




<p data-nodeid="46055"><img src="https://s0.lgstatic.com/i/image/M00/54/FD/Ciqc1F9ptfKAeNrwAADvN7mxisw072.png" alt="Drawing 5.png" data-nodeid="46059"></p>
<p data-nodeid="46056"><strong data-nodeid="46064">ChannelHandler 是注册在 Channel 上的消息处理器</strong>，在 Netty 中也有类似的抽象，相信你对此应该不会陌生。下图展示了 ChannelHandler 接口的定义，在 ChannelHandler 中可以处理 Channel 的连接建立以及连接断开事件，还可以处理读取到的数据、发送的数据以及捕获到的异常。从这些方法的命名可以看到，它们都是动词的过去式，说明相应事件已经发生过了。</p>







<p data-nodeid="45095"><img src="https://s0.lgstatic.com/i/image/M00/55/08/CgqCHl9ptf-AM7HwAABIy1ahqFw153.png" alt="Drawing 6.png" data-nodeid="45103"></p>


<p data-nodeid="44613">需要注意的是：ChannelHandler 接口被 @SPI 注解修饰，表示该接口是一个扩展点。</p>




<p data-nodeid="18637">在前面课时介绍 Netty 的时候，我们提到过有一类特殊的 ChannelHandler 专门负责实现编解码功能，从而实现字节数据与有意义的消息之间的转换，或是消息之间的相互转换。在dubbo-remoting-api 中也有相似的抽象，如下所示：</p>
<pre class="lang-java" data-nodeid="18638"><code data-language="java"><span class="hljs-meta">@SPI</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Codec2</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.CODEC_KEY})</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">encode</span><span class="hljs-params">(Channel channel, ChannelBuffer buffer, Object message)</span> 
        <span class="hljs-keyword">throws</span> IOException</span>;
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.CODEC_KEY})</span>
&nbsp; &nbsp; <span class="hljs-function">Object <span class="hljs-title">decode</span><span class="hljs-params">(Channel channel, ChannelBuffer buffer)</span>
        <span class="hljs-keyword">throws</span> IOException</span>;
        
&nbsp; &nbsp; <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">DecodeResult</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; NEED_MORE_INPUT, SKIP_SOME_INPUT
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="18639">这里需要关注的是 Codec2 接口被 @SPI 接口修饰了，表示该接口是一个扩展接口，同时其 encode() 方法和 decode() 方法都被 @Adaptive 注解修饰，也就会生成适配器类，其中会根据 URL 中的 codec 值确定具体的扩展实现类。</p>
<p data-nodeid="18640">DecodeResult 这个枚举是在处理 TCP 传输时粘包和拆包使用的，之前简易版本 RPC 也处理过这种问题，例如，当前能读取到的数据不足以构成一个消息时，就会使用 NEED_MORE_INPUT 这个枚举。</p>
<p data-nodeid="43655">接下来看<strong data-nodeid="43662">Client 和 RemotingServer 两个接口</strong>，分别抽象了客户端和服务端，两者都继承了 Channel、Resetable 等接口，也就是说两者都具备了读写数据能力。</p>
<p data-nodeid="44142"><img src="https://s0.lgstatic.com/i/image/M00/55/08/CgqCHl9ptgaAPRDbAAA7kgy1X5k082.png" alt="Drawing 7.png" data-nodeid="44146"></p>
<p data-nodeid="44143">Client 和 Server 本身都是 Endpoint，只不过在语义上区分了请求和响应的职责，两者都具备发送的能力，所以都继承了 Endpoint 接口。Client 和 Server 的主要区别是 Client 只能关联一个 Channel，而 Server 可以接收多个 Client 发起的 Channel 连接。所以在 RemotingServer 接口中定义了查询 Channel 的相关方法，如下图所示：</p>






<p data-nodeid="43186"><img src="https://s0.lgstatic.com/i/image/M00/54/FD/Ciqc1F9pthSAPWv0AAA0yX1lW-Y033.png" alt="Drawing 8.png" data-nodeid="43190"></p>





<p data-nodeid="18645">Dubbo 在 Client 和 Server 之上又封装了一层<strong data-nodeid="18796">Transporter 接口</strong>，其具体定义如下：</p>
<pre class="lang-java" data-nodeid="18646"><code data-language="java"><span class="hljs-meta">@SPI("netty")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Transporter</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})</span>
&nbsp; &nbsp; <span class="hljs-function">RemotingServer <span class="hljs-title">bind</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> 
        <span class="hljs-keyword">throws</span> RemotingException</span>;
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})</span>
&nbsp; &nbsp; <span class="hljs-function">Client <span class="hljs-title">connect</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> 
        <span class="hljs-keyword">throws</span> RemotingException</span>;
}
</code></pre>
<p data-nodeid="18647">我们看到 Transporter 接口上有 @SPI 注解，它是一个扩展接口，默认使用“netty”这个扩展名，@Adaptive 注解的出现表示动态生成适配器类，会先后根据“server”“transporter”的值确定 RemotingServer 的扩展实现类，先后根据“client”“transporter”的值确定 Client 接口的扩展实现。</p>
<p data-nodeid="41751">Transporter 接口的实现有哪些呢？如下图所示，针对每个支持的 NIO 库，都有一个 Transporter 接口实现，散落在各个 dubbo-remoting-* 实现模块中。</p>
<p data-nodeid="42235"><img src="https://s0.lgstatic.com/i/image/M00/55/08/CgqCHl9pthuAFNMOAABRJaJXls0493.png" alt="Drawing 9.png" data-nodeid="42239"></p>
<p data-nodeid="42236">这些 Transporter 接口实现返回的 Client 和 RemotingServer 具体是什么呢？如下图所示，返回的是 NIO 库对应的 RemotingServer 实现和 Client 实现。</p>






<p data-nodeid="41278"><img src="https://s0.lgstatic.com/i/image/M00/54/FD/Ciqc1F9ptiCAHkUSAADCSKg5KhY994.png" alt="Drawing 10.png" data-nodeid="41282"><br>
<img src="https://s0.lgstatic.com/i/image/M00/55/08/CgqCHl9pti-AHj3DAACwPfuEgm8435.png" alt="Drawing 11.png" data-nodeid="41286"></p>


<p data-nodeid="40792">相信看到这里，你应该已经发现 Transporter 这一层抽象出来的接口，与 Netty 的核心接口是非常相似的。那为什么要单独抽象出 Transporter层，而不是像简易版 RPC 框架那样，直接让上层使用 Netty 呢？</p>








<p data-nodeid="18654">其实这个问题的答案也呼之欲出了，Netty、Mina、Grizzly 这个 NIO 库对外接口和使用方式不一样，如果在上层直接依赖了 Netty 或是 Grizzly，就依赖了具体的 NIO 库实现，而不是依赖一个有传输能力的抽象，后续要切换实现的话，就需要修改依赖和接入的相关代码，非常容易改出 Bug。这也不符合设计模式中的开放-封闭原则。</p>
<p data-nodeid="18655">有了 Transporter 层之后，我们可以通过 Dubbo SPI 修改使用的具体 Transporter 扩展实现，从而切换到不同的 Client 和 RemotingServer 实现，达到底层 NIO 库切换的目的，而且无须修改任何代码。即使有更先进的 NIO 库出现，我们也只需要开发相应的 dubbo-remoting-* 实现模块提供 Transporter、Client、RemotingServer 等核心接口的实现，即可接入，完全符合开放-封闭原则。</p>
<p data-nodeid="18656">在最后，我们还要看一个类——<strong data-nodeid="18825">Transporters</strong>，它不是一个接口，而是门面类，其中<strong data-nodeid="18826">封装了 Transporter 对象的创建（通过 Dubbo SPI）以及 ChannelHandler 的处理</strong>，如下所示：</p>
<pre class="lang-java" data-nodeid="18657"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Transporters</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">Transporters</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">// 省略bind()和connect()方法的重载</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> RemotingServer <span class="hljs-title">bind</span><span class="hljs-params">(URL url, 
            ChannelHandler... handlers)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; ChannelHandler handler;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (handlers.length == <span class="hljs-number">1</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler = handlers[<span class="hljs-number">0</span>];
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler = <span class="hljs-keyword">new</span> ChannelHandlerDispatcher(handlers);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getTransporter().bind(url, handler);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Client <span class="hljs-title">connect</span><span class="hljs-params">(URL url, ChannelHandler... handlers)</span>
           <span class="hljs-keyword">throws</span> RemotingException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; ChannelHandler handler;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (handlers == <span class="hljs-keyword">null</span> || handlers.length == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler = <span class="hljs-keyword">new</span> ChannelHandlerAdapter();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (handlers.length == <span class="hljs-number">1</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler = handlers[<span class="hljs-number">0</span>];
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// ChannelHandlerDispatcher</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler = <span class="hljs-keyword">new</span> ChannelHandlerDispatcher(handlers);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getTransporter().connect(url, handler);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Transporter <span class="hljs-title">getTransporter</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-comment">// 自动生成Transporter适配器并加载</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ExtensionLoader.getExtensionLoader(Transporter.class)
            .getAdaptiveExtension();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="18658">在创建 Client 和 RemotingServer 的时候，可以指定多个 ChannelHandler 绑定到 Channel 来处理其中传输的数据。Transporters.connect() 方法和 bind() 方法中，会将多个 ChannelHandler 封装成一个 ChannelHandlerDispatcher 对象。</p>
<p data-nodeid="18659">ChannelHandlerDispatcher 也是 ChannelHandler 接口的实现类之一，维护了一个 CopyOnWriteArraySet<channelhandler> 集合，它所有的 ChannelHandler 接口实现都会调用其中每个 ChannelHandler 元素的相应方法。另外，ChannelHandlerDispatcher 还提供了增删该 ChannelHandler 集合的相关方法。</channelhandler></p>
<p data-nodeid="18660">到此为止，Dubbo Transport 层的核心接口就介绍完了，这里简单总结一下：</p>
<ul data-nodeid="18661">
<li data-nodeid="18662">
<p data-nodeid="18663">Endpoint 接口抽象了“端点”的概念，这是所有抽象接口的基础。</p>
</li>
<li data-nodeid="18664">
<p data-nodeid="18665">上层使用方会通过 Transporters 门面类获取到 Transporter 的具体扩展实现，然后通过 Transporter 拿到相应的 Client 和 RemotingServer 实现，就可以建立（或接收）Channel 与远端进行交互了。</p>
</li>
<li data-nodeid="18666">
<p data-nodeid="18667">无论是 Client 还是 RemotingServer，都会使用 ChannelHandler 处理 Channel 中传输的数据，其中负责编解码的 ChannelHandler 被抽象出为 Codec2 接口。</p>
</li>
</ul>
<p data-nodeid="35836">整个架构如下图所示，与 Netty 的架构非常类似。</p>
<p data-nodeid="39837"><img src="https://s0.lgstatic.com/i/image/M00/55/09/CgqCHl9ptlyABsjpAAGGk7pFIzQ293.png" alt="Lark20200922-162354.png" data-nodeid="39840"></p>

<div data-nodeid="39357"><p style="text-align:center">Transporter 层整体结构图</p></div>









<h3 data-nodeid="38187" class="">总结</h3>




<p data-nodeid="18673">本课时我们首先介绍了 dubbo-remoting 模块在 Dubbo 架构中的位置，以及 dubbo-remoting 模块的结构。接下来分析了 dubbo-remoting 模块中各个子模块之间的依赖关系，并重点介绍了 dubbo-remoting-api 子模块中各个包的核心功能。最后我们还深入分析了整个 Transport 层的核心接口，以及这些接口抽象出来的 Transporter 架构。</p>
<p data-nodeid="38653" class="">关于本课时，你若还有什么疑问或想法，欢迎你留言跟我分享。</p>

---

### 精选评论

##### **贵：
> 这一块钱也太值了

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢肯定，希望你也好好坚持下去哦，后面还有很多干货文章的，加油！

##### **龙：
> 非常清晰

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢你的肯定，继续努力，继续加油哈~

##### 魏：
> 打卡

##### **伟：
> 这一部分写的非常清晰，真值

##### **2895：
> 讲解的非常好，一直弄不懂的网络弄懂了

