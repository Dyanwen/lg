<p data-nodeid="8649" class="">在前两节课我们介绍了 TCP 拆包/粘包的问题，以及如何使用 Netty 实现自定义协议的编解码。可以看到，网络通信的底层实现，Netty 都已经帮我们封装好了，我们只需要扩展 ChannelHandler 实现自定义的编解码逻辑即可。更加人性化的是，Netty 提供了很多开箱即用的解码器，这些解码器基本覆盖了 TCP 拆包/粘包的通用解决方案。本节课我们将对 Netty 常用的解码器进行讲解，一起探索下它们有哪些用法和技巧。</p>
<p data-nodeid="8650">在本节课开始之前，我们首先回顾一下 TCP 拆包/粘包的主流解决方案。并梳理出 Netty 对应的编码器类。</p>
<h3 data-nodeid="8651">固定长度解码器 FixedLengthFrameDecoder</h3>
<p data-nodeid="8652">固定长度解码器 FixedLengthFrameDecoder 非常简单，直接通过构造函数设置固定长度的大小 frameLength，无论接收方一次获取多大的数据，都会严格按照 frameLength 进行解码。如果累积读取到长度大小为 frameLength 的消息，那么解码器认为已经获取到了一个完整的消息。如果消息长度小于 frameLength，FixedLengthFrameDecoder 解码器会一直等后续数据包的到达，直至获得完整的消息。下面我们通过一个例子感受一下使用 Netty 实现固定长度解码是多么简单。</p>
<pre class="lang-java" data-nodeid="8653"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServer</span> </span>{
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
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> EchoServerHandler());
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
<span class="hljs-meta">@Sharable</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EchoServerHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ChannelInboundHandlerAdapter</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"Receive client : ["</span> + ((ByteBuf) msg).toString(CharsetUtil.UTF_8) + <span class="hljs-string">"]"</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="8654">在上述服务端的代码中使用了固定 10 字节的解码器，并在解码之后通过 EchoServerHandler 打印结果。我们可以启动服务端，通过 telnet 命令像服务端发送数据，观察代码输出的结果。</p>
<p data-nodeid="8655">客户端输入：</p>
<pre class="lang-java" data-nodeid="8656"><code data-language="java">telnet localhost <span class="hljs-number">8088</span>
Trying ::<span class="hljs-number">1</span>...
Connected to localhost.
Escape character is <span class="hljs-string">'^]'</span>.
<span class="hljs-number">1234567890123</span>
<span class="hljs-number">456789012</span>
</code></pre>
<p data-nodeid="8657">服务端输出：</p>
<pre class="lang-java" data-nodeid="8658"><code data-language="java">Receive client : [<span class="hljs-number">1234567890</span>]
Receive client : [<span class="hljs-number">123</span>
<span class="hljs-number">45678</span>]
</code></pre>
<h3 data-nodeid="8659">特殊分隔符解码器 DelimiterBasedFrameDecoder</h3>
<p data-nodeid="8660">使用特殊分隔符解码器 DelimiterBasedFrameDecoder 之前我们需要了解以下几个属性的作用。</p>
<ul data-nodeid="8661">
<li data-nodeid="8662">
<p data-nodeid="8663"><strong data-nodeid="8803">delimiters</strong></p>
</li>
</ul>
<p data-nodeid="8664">delimiters 指定特殊分隔符，通过写入 ByteBuf 作为<strong data-nodeid="8809">参数</strong>传入。delimiters 的类型是 ByteBuf 数组，所以我们可以同时指定多个分隔符，但是最终会选择长度最短的分隔符进行消息拆分。</p>
<p data-nodeid="8665">例如接收方收到的数据为：</p>
<pre class="lang-java" data-nodeid="8666"><code data-language="java">+--------------+
| ABC\nDEF\r\n |
+--------------+
</code></pre>
<p data-nodeid="9525">如果指定的多个分隔符为 \n 和 \r\n，DelimiterBasedFrameDecoder 会退化成使用 LineBasedFrameDecoder 进行解析，那么会解码出两个消息。</p>

<pre class="lang-java" data-nodeid="8668"><code data-language="java">+-----+-----+
| ABC | DEF |
+-----+-----+
</code></pre>
<p data-nodeid="8939" class="">如果指定的特定分隔符只有 \r\n，那么只会解码出一个消息：</p>

<pre class="lang-java" data-nodeid="8670"><code data-language="java">+----------+
| ABC\nDEF |
+----------+
</code></pre>
<ul data-nodeid="8671">
<li data-nodeid="8672">
<p data-nodeid="8673"><strong data-nodeid="8822">maxLength</strong></p>
</li>
</ul>
<p data-nodeid="8674">maxLength 是报文最大长度的限制。如果超过 maxLength 还没有检测到指定分隔符，将会抛出 TooLongFrameException。可以说 maxLength 是对程序在极端情况下的一种<strong data-nodeid="8828">保护措施</strong>。</p>
<ul data-nodeid="8675">
<li data-nodeid="8676">
<p data-nodeid="8677"><strong data-nodeid="8832">failFast</strong></p>
</li>
</ul>
<p data-nodeid="8678">failFast 与 maxLength 需要搭配使用，通过设置 failFast 可以控制抛出 TooLongFrameException 的时机，可以说 Netty 在细节上考虑得面面俱到。如果 failFast=true，那么在超出 maxLength 会立即抛出 TooLongFrameException，不再继续进行解码。如果 failFast=false，那么会等到解码出一个完整的消息后才会抛出 TooLongFrameException。</p>
<ul data-nodeid="8679">
<li data-nodeid="8680">
<p data-nodeid="8681"><strong data-nodeid="8837">stripDelimiter</strong></p>
</li>
</ul>
<p data-nodeid="8682">stripDelimiter 的作用是判断解码后得到的消息是否去除分隔符。如果 stripDelimiter=false，特定分隔符为 \n，那么上述数据包解码出的结果为：</p>
<pre class="lang-java" data-nodeid="8683"><code data-language="java">+-------+---------+
| ABC\n | DEF\r\n |
+-------+---------+
</code></pre>
<p data-nodeid="8684">下面我们还是结合代码示例学习 DelimiterBasedFrameDecoder 的用法，依然以固定编码器小节中使用的代码为基础稍做改动，引入特殊分隔符解码器 DelimiterBasedFrameDecoder：</p>
<pre class="lang-java" data-nodeid="8685"><code data-language="java">b.group(bossGroup, workerGroup)
&nbsp; &nbsp; .channel(NioServerSocketChannel.class)
&nbsp; &nbsp; .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ByteBuf delimiter = Unpooled.copiedBuffer(<span class="hljs-string">"&amp;"</span>.getBytes());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> DelimiterBasedFrameDecoder(<span class="hljs-number">10</span>, <span class="hljs-keyword">true</span>, <span class="hljs-keyword">true</span>, delimiter));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline().addLast(<span class="hljs-keyword">new</span> EchoServerHandler());
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; });
</code></pre>
<p data-nodeid="8686">我们依然通过 telnet 模拟客户端发送数据，观察代码输出的结果，可以发现由于 maxLength 设置的只有 10，所以在解析到第三个消息时抛出异常。</p>
<p data-nodeid="8687">客户端输入：</p>
<pre class="lang-java" data-nodeid="8688"><code data-language="java">telnet localhost <span class="hljs-number">8088</span>
Trying ::<span class="hljs-number">1</span>...
Connected to localhost.
Escape character is <span class="hljs-string">'^]'</span>.
hello&amp;world&amp;<span class="hljs-number">1234567890</span>ab
</code></pre>
<p data-nodeid="8689">服务端输出：</p>
<pre class="lang-java" data-nodeid="8690"><code data-language="java">Receive client : [hello]
Receive client : [world]
九月 <span class="hljs-number">25</span>, <span class="hljs-number">2020</span> <span class="hljs-number">8</span>:<span class="hljs-number">46</span>:<span class="hljs-number">01</span> 下午 io.netty.channel.DefaultChannelPipeline onUnhandledInboundException
警告: <span class="hljs-function">An <span class="hljs-title">exceptionCaught</span><span class="hljs-params">()</span> event was fired, and it reached at the tail of the pipeline. It usually means the last handler in the pipeline did not handle the exception.
io.netty.handler.codec.TooLongFrameException: frame length exceeds 10: 13 - discarded
	at io.netty.handler.codec.DelimiterBasedFrameDecoder.<span class="hljs-title">fail</span><span class="hljs-params">(DelimiterBasedFrameDecoder.java:<span class="hljs-number">302</span>)</span>
	at io.netty.handler.codec.DelimiterBasedFrameDecoder.<span class="hljs-title">decode</span><span class="hljs-params">(DelimiterBasedFrameDecoder.java:<span class="hljs-number">268</span>)</span>
	at io.netty.handler.codec.DelimiterBasedFrameDecoder.<span class="hljs-title">decode</span><span class="hljs-params">(DelimiterBasedFrameDecoder.java:<span class="hljs-number">218</span>)</span>
</span></code></pre>
<h3 data-nodeid="8691">长度域解码器 LengthFieldBasedFrameDecoder</h3>
<p data-nodeid="8692">长度域解码器 LengthFieldBasedFrameDecoder 是解决 TCP 拆包/粘包问题最常用的**解码器。**它基本上可以覆盖大部分基于长度拆包场景，开源消息中间件 RocketMQ 就是使用 LengthFieldBasedFrameDecoder 进行解码的。LengthFieldBasedFrameDecoder 相比 FixedLengthFrameDecoder 和 DelimiterBasedFrameDecoder 要复杂一些，接下来我们就一起学习下这个强大的解码器。</p>
<p data-nodeid="8693">首先我们同样先了解 LengthFieldBasedFrameDecoder 中的几个重要属性，这里我主要把它们分为两个部分：<strong data-nodeid="8862">长度域解码器特有属性</strong>以及<strong data-nodeid="8863">与其他解码器（如特定分隔符解码器）的相似的属性</strong>。</p>
<ul data-nodeid="8694">
<li data-nodeid="8695">
<p data-nodeid="8696"><strong data-nodeid="8867">长度域解码器特有属性。</strong></p>
</li>
</ul>
<pre class="lang-java" data-nodeid="8697"><code data-language="java"><span class="hljs-comment">// 长度字段的偏移量，也就是存放长度数据的起始位置</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> lengthFieldOffset;&nbsp;
<span class="hljs-comment">// 长度字段所占用的字节数</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> lengthFieldLength;&nbsp;
<span class="hljs-comment">/*
&nbsp;* 消息长度的修正值
&nbsp;*
&nbsp;* 在很多较为复杂一些的协议设计中，长度域不仅仅包含消息的长度，而且包含其他的数据，如版本号、数据类型、数据状态等，那么这时候我们需要使用 lengthAdjustment 进行修正
&nbsp;*&nbsp;
&nbsp;* lengthAdjustment = 包体的长度值 - 长度域的值
&nbsp;*
&nbsp;*/</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> lengthAdjustment;&nbsp;
<span class="hljs-comment">// 解码后需要跳过的初始字节数，也就是消息内容字段的起始位置</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> initialBytesToStrip;
<span class="hljs-comment">// 长度字段结束的偏移量，lengthFieldEndOffset = lengthFieldOffset + lengthFieldLength</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> lengthFieldEndOffset;
</code></pre>
<ul data-nodeid="8698">
<li data-nodeid="8699">
<p data-nodeid="8700"><strong data-nodeid="8871">与固定长度解码器和特定分隔符解码器相似的属性。</strong></p>
</li>
</ul>
<pre class="lang-java" data-nodeid="8701"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> maxFrameLength; <span class="hljs-comment">// 报文最大限制长度</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> failFast; <span class="hljs-comment">// 是否立即抛出 TooLongFrameException，与 maxFrameLength 搭配使用</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> discardingTooLongFrame; <span class="hljs-comment">// 是否处于丢弃模式</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> tooLongFrameLength; <span class="hljs-comment">// 需要丢弃的字节数</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> bytesToDiscard; <span class="hljs-comment">// 累计丢弃的字节数</span>
</code></pre>
<p data-nodeid="8702">下面我们结合具体的示例来解释下每种参数的组合，其实在 Netty LengthFieldBasedFrameDecoder 源码的注释中已经描述得非常详细，一共给出了 7 个场景示例，理解了这些示例基本上可以真正掌握 LengthFieldBasedFrameDecoder 的参数用法。</p>
<p data-nodeid="8703"><strong data-nodeid="8876">示例 1：典型的基于消息长度 + 消息内容的解码。</strong></p>
<pre class="lang-java" data-nodeid="8704"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">14</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">14</span> bytes)</span>
+--------+----------------+&nbsp; &nbsp; &nbsp; +--------+----------------+
| Length | Actual Content |-----&gt;| Length | Actual Content |
| 0x000C | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | 0x000C | "HELLO, WORLD" |
+--------+----------------+&nbsp; &nbsp; &nbsp; +--------+----------------+
</span></code></pre>
<p data-nodeid="8705">上述协议是最基本的格式，报文只包含消息长度 Length 和消息内容 Content 字段，其中 Length 为 16 进制表示，共占用 2 字节，Length 的值 0x000C 代表 Content 占用 12 字节。该协议对应的解码器参数组合如下：</p>
<ul data-nodeid="8706">
<li data-nodeid="8707">
<p data-nodeid="8708">lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。</p>
</li>
<li data-nodeid="8709">
<p data-nodeid="8710">lengthFieldLength = 2，协议设计的固定长度。</p>
</li>
<li data-nodeid="8711">
<p data-nodeid="8712">lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。</p>
</li>
<li data-nodeid="8713">
<p data-nodeid="8714">initialBytesToStrip = 0，解码后内容依然是 Length + Content，不需要跳过任何初始字节。</p>
</li>
</ul>
<p data-nodeid="8715"><strong data-nodeid="8885">示例 2：解码结果需要截断。</strong></p>
<pre class="lang-java" data-nodeid="8716"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">14</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">12</span> bytes)</span>
+--------+----------------+&nbsp; &nbsp; &nbsp; +----------------+
| Length | Actual Content |-----&gt;| Actual Content |
| 0x000C | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | "HELLO, WORLD" |
+--------+----------------+&nbsp; &nbsp; &nbsp; +----------------+
</span></code></pre>
<p data-nodeid="8717">示例 2 和示例 1 的区别在于解码后的结果只包含消息内容，其他的部分是不变的。该协议对应的解码器参数组合如下：</p>
<ul data-nodeid="8718">
<li data-nodeid="8719">
<p data-nodeid="8720">lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。</p>
</li>
<li data-nodeid="8721">
<p data-nodeid="8722">lengthFieldLength = 2，协议设计的固定长度。</p>
</li>
<li data-nodeid="8723">
<p data-nodeid="8724">lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。</p>
</li>
<li data-nodeid="8725">
<p data-nodeid="8726">initialBytesToStrip = 2，跳过 Length 字段的字节长度，解码后 ByteBuf 中只包含 Content字段。</p>
</li>
</ul>
<p data-nodeid="8727"><strong data-nodeid="8894">示例 3：长度字段包含消息长度和消息内容所占的字节。</strong></p>
<pre class="lang-java" data-nodeid="8728"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">14</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">14</span> bytes)</span>
+--------+----------------+&nbsp; &nbsp; &nbsp; +--------+----------------+
| Length | Actual Content |-----&gt;| Length | Actual Content |
| 0x000E | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | 0x000E | "HELLO, WORLD" |
+--------+----------------+&nbsp; &nbsp; &nbsp; +--------+----------------+
</span></code></pre>
<p data-nodeid="8729">与前两个示例不同的是，示例 3 的 Length 字段包含 Length 字段自身的固定长度以及 Content 字段所占用的字节数，Length 的值为 0x000E（2 + 12 = 14 字节），在 Length 字段值（14 字节）的基础上做 lengthAdjustment（-2）的修正，才能得到真实的 Content 字段长度，所以对应的解码器参数组合如下：</p>
<ul data-nodeid="8730">
<li data-nodeid="8731">
<p data-nodeid="8732">lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。</p>
</li>
<li data-nodeid="8733">
<p data-nodeid="8734">lengthFieldLength = 2，协议设计的固定长度。</p>
</li>
<li data-nodeid="8735">
<p data-nodeid="8736">lengthAdjustment = -2，长度字段为 14 字节，需要减 2 才是拆包所需要的长度。</p>
</li>
<li data-nodeid="8737">
<p data-nodeid="8738">initialBytesToStrip = 0，解码后内容依然是 Length + Content，不需要跳过任何初始字节。</p>
</li>
</ul>
<p data-nodeid="8739"><strong data-nodeid="8903">示例 4：基于长度字段偏移的解码。</strong></p>
<pre class="lang-java" data-nodeid="8740"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">17</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">17</span> bytes)</span>
+----------+----------+----------------+&nbsp; &nbsp; &nbsp; +----------+----------+----------------+
| Header 1 |&nbsp; Length&nbsp; | Actual Content |-----&gt;| Header 1 |&nbsp; Length&nbsp; | Actual Content |
|&nbsp; 0xCAFE&nbsp; | 0x00000C | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; |&nbsp; 0xCAFE&nbsp; | 0x00000C | "HELLO, WORLD" |
+----------+----------+----------------+&nbsp; &nbsp; &nbsp; +----------+----------+----------------+
</span></code></pre>
<p data-nodeid="8741">示例 4 中 Length 字段不再是报文的起始位置，Length 字段的值为 0x00000C，表示 Content 字段占用 12 字节，该协议对应的解码器参数组合如下：</p>
<ul data-nodeid="8742">
<li data-nodeid="8743">
<p data-nodeid="8744">lengthFieldOffset = 2，需要跳过 Header 1 所占用的 2 字节，才是 Length 的起始位置。</p>
</li>
<li data-nodeid="8745">
<p data-nodeid="8746">lengthFieldLength = 3，协议设计的固定长度。</p>
</li>
<li data-nodeid="8747">
<p data-nodeid="8748">lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。</p>
</li>
<li data-nodeid="8749">
<p data-nodeid="8750">initialBytesToStrip = 0，解码后内容依然是完整的报文，不需要跳过任何初始字节。</p>
</li>
</ul>
<p data-nodeid="8751"><strong data-nodeid="8912">示例 5：长度字段与内容字段不再相邻。</strong></p>
<pre class="lang-java" data-nodeid="8752"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">17</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">17</span> bytes)</span>
+----------+----------+----------------+&nbsp; &nbsp; &nbsp; +----------+----------+----------------+
|&nbsp; Length&nbsp; | Header 1 | Actual Content |-----&gt;|&nbsp; Length&nbsp; | Header 1 | Actual Content |
| 0x00000C |&nbsp; 0xCAFE&nbsp; | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | 0x00000C |&nbsp; 0xCAFE&nbsp; | "HELLO, WORLD" |
+----------+----------+----------------+&nbsp; &nbsp; &nbsp; +----------+----------+----------------+
</span></code></pre>
<p data-nodeid="8753">示例 5 中的 Length 字段之后是 Header 1，Length 与 Content 字段不再相邻。Length 字段所表示的内容略过了 Header 1 字段，所以也需要通过 lengthAdjustment 修正才能得到 Header + Content 的内容。示例 5 所对应的解码器参数组合如下：</p>
<ul data-nodeid="8754">
<li data-nodeid="8755">
<p data-nodeid="8756">lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。</p>
</li>
<li data-nodeid="8757">
<p data-nodeid="8758">lengthFieldLength = 3，协议设计的固定长度。</p>
</li>
<li data-nodeid="8759">
<p data-nodeid="8760">lengthAdjustment = 2，由于 Header + Content 一共占用 2 + 12 = 14 字节，所以 Length 字段值（12 字节）加上 lengthAdjustment（2 字节）才能得到 Header + Content 的内容（14 字节）。</p>
</li>
<li data-nodeid="8761">
<p data-nodeid="8762">initialBytesToStrip = 0，解码后内容依然是完整的报文，不需要跳过任何初始字节。</p>
</li>
</ul>
<p data-nodeid="8763"><strong data-nodeid="8921">示例 6：基于长度偏移和长度修正的解码。</strong></p>
<pre class="lang-java" data-nodeid="8764"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">16</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">13</span> bytes)</span>
+------+--------+------+----------------+&nbsp; &nbsp; &nbsp; +------+----------------+
| HDR1 | Length | HDR2 | Actual Content |-----&gt;| HDR2 | Actual Content |
| 0xCA | 0x000C | 0xFE | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | 0xFE | "HELLO, WORLD" |
+------+--------+------+----------------+&nbsp; &nbsp; &nbsp; +------+----------------+
</span></code></pre>
<p data-nodeid="8765">示例 6 中 Length 字段前后分为别 HDR1 和 HDR2 字段，各占用 1 字节，所以既需要做长度字段的偏移，也需要做 lengthAdjustment 修正，具体修正的过程与 示例 5 类似。对应的解码器参数组合如下：</p>
<ul data-nodeid="8766">
<li data-nodeid="8767">
<p data-nodeid="8768">lengthFieldOffset = 1，需要跳过 HDR1 所占用的 1 字节，才是 Length 的起始位置。</p>
</li>
<li data-nodeid="8769">
<p data-nodeid="8770">lengthFieldLength = 2，协议设计的固定长度。</p>
</li>
<li data-nodeid="8771">
<p data-nodeid="8772">lengthAdjustment = 1，由于 HDR2 + Content 一共占用 1 + 12 = 13 字节，所以 Length 字段值（12 字节）加上 lengthAdjustment（1）才能得到 HDR2 + Content 的内容（13 字节）。</p>
</li>
<li data-nodeid="8773">
<p data-nodeid="8774">initialBytesToStrip = 3，解码后跳过 HDR1 和 Length 字段，共占用 3 字节。</p>
</li>
</ul>
<p data-nodeid="8775"><strong data-nodeid="8930">示例 7：长度字段包含除 Content 外的多个其他字段。</strong></p>
<pre class="lang-java" data-nodeid="8776"><code data-language="java"><span class="hljs-function">BEFORE <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">16</span> bytes)</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AFTER <span class="hljs-title">DECODE</span> <span class="hljs-params">(<span class="hljs-number">13</span> bytes)</span>
+------+--------+------+----------------+&nbsp; &nbsp; &nbsp; +------+----------------+
| HDR1 | Length | HDR2 | Actual Content |-----&gt;| HDR2 | Actual Content |
| 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |&nbsp; &nbsp; &nbsp; | 0xFE | "HELLO, WORLD" |
+------+--------+------+----------------+&nbsp; &nbsp; &nbsp; +------+----------------+
</span></code></pre>
<p data-nodeid="8777">示例 7 与 示例 6 的区别在于 Length 字段记录了整个报文的长度，包含 Length 自身所占字节、HDR1 、HDR2 以及 Content 字段的长度，解码器需要知道如何进行 lengthAdjustment 调整，才能得到 HDR2 和 Content 的内容。所以我们可以采用如下的解码器参数组合：</p>
<ul data-nodeid="8778">
<li data-nodeid="8779">
<p data-nodeid="8780">lengthFieldOffset = 1，需要跳过 HDR1 所占用的 1 字节，才是 Length 的起始位置。</p>
</li>
<li data-nodeid="8781">
<p data-nodeid="8782">lengthFieldLength = 2，协议设计的固定长度。</p>
</li>
<li data-nodeid="8783">
<p data-nodeid="8784">lengthAdjustment = -3，Length 字段值（16 字节）需要减去 HDR1（1 字节） 和 Length 自身所占字节长度（2 字节）才能得到 HDR2 和 Content 的内容（1 + 12 = 13 字节）。</p>
</li>
<li data-nodeid="8785">
<p data-nodeid="8786">initialBytesToStrip = 3，解码后跳过 HDR1 和 Length 字段，共占用 3 字节。</p>
</li>
</ul>
<p data-nodeid="8787">以上 7 种示例涵盖了 LengthFieldBasedFrameDecoder 大部分的使用场景，你是否学会了呢？最后留一个小任务，在上一节课程中我们设计了一个较为通用的协议，如下所示。如何使用长度域解码器 LengthFieldBasedFrameDecoder 完成该协议的解码呢？抓紧自己尝试下吧。</p>
<pre class="lang-java" data-nodeid="8788"><code data-language="java">+---------------------------------------------------------------+
| 魔数 <span class="hljs-number">2</span><span class="hljs-keyword">byte</span> | 协议版本号 <span class="hljs-number">1</span><span class="hljs-keyword">byte</span> | 序列化算法 <span class="hljs-number">1</span><span class="hljs-keyword">byte</span> | 报文类型 <span class="hljs-number">1</span><span class="hljs-keyword">byte</span>&nbsp; |
+---------------------------------------------------------------+
| 状态 <span class="hljs-number">1</span><span class="hljs-keyword">byte</span> |&nbsp; &nbsp; &nbsp; &nbsp; 保留字段 <span class="hljs-number">4</span><span class="hljs-keyword">byte</span>&nbsp; &nbsp; &nbsp;|&nbsp; &nbsp; &nbsp; 数据长度 <span class="hljs-number">4</span><span class="hljs-keyword">byte</span>&nbsp; &nbsp; &nbsp;|&nbsp;
+---------------------------------------------------------------+
|&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;数据内容 （长度不定）&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |
+---------------------------------------------------------------+
</code></pre>
<h3 data-nodeid="8789">总结</h3>
<p data-nodeid="8790" class="">本节课我们介绍了三种常用的解码器，从中我们可以体会到 Netty 在设计上的优雅，只需要调整参数就可以轻松实现各种功能。在健壮性上，Netty 也考虑得非常全面，很多边界情况 Netty 都贴心地增加了保护性措施。实现一个健壮的解码器并不容易，很可能因为一次解析错误就会导致解码器一直处理错乱的状态。如果你使用了基于长度编码的二进制协议，那么推荐你使用 LengthFieldBasedFrameDecoder，它已经可以满足实际项目中的大部分场景，基本不需要再自定义实现了。希望朋友们在项目开发中能够学以致用。</p>

---

### 精选评论

##### *薄：
> lengthFieldOffset = 10，需要跳过魔数 2byte+协议版本号 1byte+序列化算法 1byte +报文类型 1byte+状态 1byte +保留字段 4byte = 10，才是 Length 的起始位置。lengthFieldLength = 4，协议设计的固定长度。lengthAdjustment =0initialBytesToStrip = 0

##### *铁：
> 个人觉得思考题的长度区域偏移量10，长度区域4，调整0，初始化跳过0，就可以解出数据区

##### **9520：
> +--------------+| ABC\nDEF\r\n |+--------------+ 按\n分割为+-----+-----+
| ABC | DEF |
+-----+-----+为啥第2个消息不是DEF\r

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该描述更准确些，如果同时制定\n和\r\n，其实会退化成LineBasedFrameDecoder，就会截取出ABC 和 DEF。如果只是选取\r\n作为特定分隔符，只会截取出ABC\nDEF

##### **帆：
> 2020.11.12 偏向固定长度，即 前14字节固定。

##### **航：
> 2020.11.12打卡

##### 无：
> 2020-11-12 打卡

##### Q：
> 感觉length字段设置为整个请求的长度好一些吧整个结构可以认为是 前缀+长度+请求业务数据lengthFieldOffset = 10，需要跳过前面的10 字节，才是 Length 的起始位置。lengthFieldLength = 4，协议设计的固定长度lengthAdjustment = -4，因为length的长度是整个长度，需要通过-4知道前缀+业务数据的长度initialBytesToStrip = 0，不需要跳过，为什么不跳过呢，因为前缀中包含了魔术、版本号等数据，需要通过解析该值进行数据的合法性校验操作

##### **升：
> 打卡，但是我看大家都是读取固定字节长度的数据，没读到就退出下次再读

##### *磊：
> 老师，可以将协议头的魔数作为特殊分隔符来进行解码吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不可以，还是同样的问题，魔术也可能会出现在报文内容中，导致解码错误。

