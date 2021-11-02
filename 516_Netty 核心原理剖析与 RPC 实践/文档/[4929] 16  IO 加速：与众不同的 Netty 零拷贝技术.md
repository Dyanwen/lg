<p data-nodeid="841" class="">今天的课程我们继续讨论 Netty 实现高性能的另一个高阶特性——零拷贝。零拷贝是一个耳熟能详的词语，在 Linux、Kafka、RocketMQ 等知名的产品中都有使用，通常用于提升 I/O 性能。而且零拷贝也是面试过程中的高频问题，那么你知道零拷贝体现在哪些地方吗？Netty 的零拷贝技术又是如何实现的呢？接下来我们就针对 Netty 零拷贝特性进行详细地分析。</p>
<h3 data-nodeid="842">传统 Linux 中的零拷贝技术</h3>
<p data-nodeid="843">在介绍 Netty 零拷贝特性之前，我们有必要学习下传统 Linux 中零拷贝的工作原理。所谓零拷贝，就是在数据操作时，不需要将数据从一个内存位置拷贝到另外一个内存位置，这样可以减少一次内存拷贝的损耗，从而节省了 CPU 时钟周期和内存带宽。</p>
<p data-nodeid="844">我们模拟一个场景，从文件中读取数据，然后将数据传输到网络上，那么传统的数据拷贝过程会分为哪几个阶段呢？具体如下图所示。</p>
<p data-nodeid="845"><img src="https://s0.lgstatic.com/i/image/M00/80/0B/Ciqc1F_Qbz2AD4uMAARnlgeSFc4993.png" alt="Drawing 0.png" data-nodeid="931"></p>
<p data-nodeid="846">从上图中可以看出，从数据读取到发送一共经历了<strong data-nodeid="937">四次数据拷贝</strong>，具体流程如下：</p>
<ol data-nodeid="847">
<li data-nodeid="848">
<p data-nodeid="849">当用户进程发起 read() 调用后，上下文从用户态切换至内核态。DMA 引擎从文件中读取数据，并存储到内核态缓冲区，这里是<strong data-nodeid="943">第一次数据拷贝</strong>。</p>
</li>
<li data-nodeid="850">
<p data-nodeid="851">请求的数据从内核态缓冲区拷贝到用户态缓冲区，然后返回给用户进程。第二次数据拷贝的过程同时，会导致上下文从内核态再次切换到用户态。</p>
</li>
<li data-nodeid="852">
<p data-nodeid="853">用户进程调用 send() 方法期望将数据发送到网络中，此时会触发第三次线程切换，用户态会再次切换到内核态，请求的数据从用户态缓冲区被拷贝到 Socket 缓冲区。</p>
</li>
<li data-nodeid="854">
<p data-nodeid="855">最终 send() 系统调用结束返回给用户进程，发生了第四次上下文切换。第四次拷贝会异步执行，从 Socket 缓冲区拷贝到协议引擎中。</p>
</li>
</ol>
<blockquote data-nodeid="856">
<p data-nodeid="857">说明：DMA（Direct Memory Access，直接内存存取）是现代大部分硬盘都支持的特性，DMA 接管了数据读写的工作，不需要 CPU 再参与 I/O 中断的处理，从而减轻了 CPU 的负担。</p>
</blockquote>
<p data-nodeid="858">传统的数据拷贝过程为什么不是将数据直接传输到用户缓冲区呢？其实引入内核缓冲区可以充当缓存的作用，这样就可以实现文件数据的预读，提升 I/O 的性能。但是当请求数据量大于内核缓冲区大小时，在完成一次数据的读取到发送可能要经历数倍次数的数据拷贝，这就造成严重的性能损耗。</p>
<p data-nodeid="859">接下来我们介绍下使用零拷贝技术之后数据传输的流程。重新回顾一遍传统数据拷贝的过程，可以发现第二次和第三次拷贝是可以去除的，DMA 引擎从文件读取数据后放入到内核缓冲区，然后可以直接从内核缓冲区传输到 Socket 缓冲区，从而减少内存拷贝的次数。</p>
<p data-nodeid="860">在 Linux 中系统调用 sendfile() 可以实现将数据从一个文件描述符传输到另一个文件描述符，从而实现了零拷贝技术。在 Java 中也使用了零拷贝技术，它就是 NIO FileChannel 类中的 transferTo() 方法，transferTo() 底层就依赖了操作系统零拷贝的机制，它可以将数据从 FileChannel 直接传输到另外一个 Channel。transferTo() 方法的定义如下：</p>
<pre class="lang-java" data-nodeid="861"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-keyword">long</span> <span class="hljs-title">transferTo</span><span class="hljs-params">(<span class="hljs-keyword">long</span> position, <span class="hljs-keyword">long</span> count, WritableByteChannel target)</span> <span class="hljs-keyword">throws</span> IOException</span>;
</code></pre>
<p data-nodeid="862">FileChannel#transferTo() 的使用也非常简单，我们直接看如下的代码示例，通过 transferTo() 将 from.data 传输到 to.data()，等于实现了文件拷贝的功能。</p>
<pre class="lang-java" data-nodeid="863"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testTransferTo</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp; &nbsp; RandomAccessFile fromFile = <span class="hljs-keyword">new</span> RandomAccessFile(<span class="hljs-string">"from.data"</span>, <span class="hljs-string">"rw"</span>);
&nbsp; &nbsp; FileChannel fromChannel = fromFile.getChannel();
&nbsp; &nbsp; RandomAccessFile toFile = <span class="hljs-keyword">new</span> RandomAccessFile(<span class="hljs-string">"to.data"</span>, <span class="hljs-string">"rw"</span>);
&nbsp; &nbsp; FileChannel toChannel = toFile.getChannel();
&nbsp; &nbsp; <span class="hljs-keyword">long</span> position = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">long</span> count = fromChannel.size();
&nbsp; &nbsp; fromChannel.transferTo(position, count, toChannel);
}
</code></pre>
<p data-nodeid="864">在使用了 FileChannel#transferTo() 传输数据之后，我们看下数据拷贝流程发生了哪些变化，如下图所示：</p>
<p data-nodeid="865"><img src="https://s0.lgstatic.com/i/image/M00/80/16/CgqCHl_Qb0mANyjrAATEtVu9f6c390.png" alt="Drawing 1.png" data-nodeid="955"></p>
<p data-nodeid="866">比较大的一个变化是，DMA 引擎从文件中读取数据拷贝到内核态缓冲区之后，由操作系统直接拷贝到 Socket 缓冲区，不再拷贝到用户态缓冲区，所以数据拷贝的次数从之前的 4 次减少到 3 次。</p>
<p data-nodeid="867">但是上述的优化离达到零拷贝的要求还是有差距的，能否继续减少内核中的数据拷贝次数呢？在 Linux 2.4 版本之后，开发者对 Socket Buffer 追加一些 Descriptor 信息来进一步减少内核数据的复制。如下图所示，DMA 引擎读取文件内容并拷贝到内核缓冲区，然后并没有再拷贝到 Socket 缓冲区，只是将数据的长度以及位置信息被追加到 Socket 缓冲区，然后 DMA 引擎根据这些描述信息，直接从内核缓冲区读取数据并传输到协议引擎中，从而消除最后一次 CPU 拷贝。</p>
<p data-nodeid="868"><img src="https://s0.lgstatic.com/i/image/M00/80/16/CgqCHl_Qb2eASFBJAAT4WPf__Us976.png" alt="Drawing 2.png" data-nodeid="960"></p>
<p data-nodeid="6977" class="te-preview-highlight">通过上述 Linux 零拷贝技术的介绍，你也许还会存在疑问，最终使用零拷贝之后，不是还存在着数据拷贝操作吗？其实从 Linux 操作系统的角度来说，零拷贝就是为了避免用户态和内核态之间的数据拷贝。无论是传统的数据拷贝还是使用零拷贝技术，其中有 2 次 DMA 的数据拷贝必不可少，只是这 2 次 DMA 拷贝都是依赖硬件来完成，不需要 CPU 参与。所以，在这里我们讨论的零拷贝是个广义的概念，只要能够减少不必要的 CPU 拷贝，都可以被称为零拷贝。</p>















<h3 data-nodeid="870">Netty 的零拷贝技术</h3>
<p data-nodeid="871">介绍完传统 Linux 的零拷贝技术之后，我们再来学习下 Netty 中的零拷贝如何实现。Netty 中的零拷贝和传统 Linux 的零拷贝不太一样。Netty 中的零拷贝技术除了操作系统级别的功能封装，更多的是面向用户态的数据操作优化，主要体现在以下 5 个方面：</p>
<ul data-nodeid="872">
<li data-nodeid="873">
<p data-nodeid="874">堆外内存，避免 JVM 堆内存到堆外内存的数据拷贝。</p>
</li>
<li data-nodeid="875">
<p data-nodeid="876">CompositeByteBuf 类，可以组合多个 Buffer 对象合并成一个逻辑上的对象，避免通过传统内存拷贝的方式将几个 Buffer 合并成一个大的 Buffer。</p>
</li>
<li data-nodeid="877">
<p data-nodeid="878">通过 Unpooled.wrappedBuffer 可以将 byte 数组包装成 ByteBuf 对象，包装过程中不会产生内存拷贝。</p>
</li>
<li data-nodeid="879">
<p data-nodeid="880">ByteBuf.slice 操作与 Unpooled.wrappedBuffer 相反，slice 操作可以将一个 ByteBuf 对象切分成多个 ByteBuf 对象，切分过程中不会产生内存拷贝，底层共享一个 byte 数组的存储空间。</p>
</li>
<li data-nodeid="881">
<p data-nodeid="882">Netty 使用 FileRegion 实现文件传输，FileRegion 底层封装了 FileChannel#transferTo() 方法，可以将文件缓冲区的数据直接传输到目标 Channel，避免内核缓冲区和用户态缓冲区之间的数据拷贝，这属于操作系统级别的零拷贝。</p>
</li>
</ul>
<p data-nodeid="883">下面我们从以上 5 个方面逐一进行介绍。</p>
<h4 data-nodeid="884">堆外内存</h4>
<p data-nodeid="885">如果在 JVM 内部执行 I/O 操作时，必须将数据拷贝到堆外内存，才能执行系统调用。这是所有 VM 语言都会存在的问题。那么为什么操作系统不能直接使用 JVM 堆内存进行 I/O 的读写呢？主要有两点原因：第一，操作系统并不感知 JVM 的堆内存，而且 JVM 的内存布局与操作系统所分配的是不一样的，操作系统并不会按照 JVM 的行为来读写数据。第二，同一个对象的内存地址随着 JVM GC 的执行可能会随时发生变化，例如 JVM GC 的过程中会通过压缩来减少内存碎片，这就涉及对象移动的问题了。</p>
<p data-nodeid="886">Netty 在进行 I/O 操作时都是使用的堆外内存，可以避免数据从 JVM 堆内存到堆外内存的拷贝。</p>
<h4 data-nodeid="887">CompositeByteBuf</h4>
<p data-nodeid="888">CompositeByteBuf 是 Netty 中实现零拷贝机制非常重要的一个数据结构，CompositeByteBuf 可以理解为一个虚拟的 Buffer 对象，它是由多个 ByteBuf 组合而成，但是在 CompositeByteBuf 内部保存着每个 ByteBuf 的引用关系，从逻辑上构成一个整体。比较常见的像 HTTP 协议数据可以分为<strong data-nodeid="983">头部信息 header</strong>和<strong data-nodeid="984">消息体数据 body</strong>，分别存在两个不同的 ByteBuf 中，通常我们需要将两个 ByteBuf 合并成一个完整的协议数据进行发送，可以使用如下方式完成：</p>
<pre class="lang-java" data-nodeid="889"><code data-language="java">ByteBuf httpBuf = Unpooled.buffer(header.readableBytes() + body.readableBytes());
httpBuf.writeBytes(header);
httpBuf.writeBytes(body);
</code></pre>
<p data-nodeid="890">可以看出，如果想实现 header 和 body 这两个 ByteBuf 的合并，需要先初始化一个新的 httpBuf，然后再将 header 和 body 分别拷贝到新的 httpBuf。合并过程中涉及两次 CPU 拷贝，这非常浪费性能。如果使用 CompositeByteBuf 如何实现类似的需求呢？如下所示：</p>
<pre class="lang-java" data-nodeid="891"><code data-language="java">CompositeByteBuf httpBuf = Unpooled.compositeBuffer();
httpBuf.addComponents(<span class="hljs-keyword">true</span>, header, body);
</code></pre>
<p data-nodeid="892">CompositeByteBuf 通过调用 addComponents() 方法来添加多个 ByteBuf，但是底层的 byte 数组是复用的，不会发生内存拷贝。但对于用户来说，它可以当作一个整体进行操作。那么 CompositeByteBuf 内部是如何存放这些 ByteBuf，并且如何进行合并的呢？我们先通过一张图看下 CompositeByteBuf 的内部结构：</p>
<p data-nodeid="893"><img src="https://s0.lgstatic.com/i/image/M00/80/0B/Ciqc1F_Qb3SAP4vUAAZG1WvALhY410.png" alt="Drawing 3.png" data-nodeid="989"></p>
<p data-nodeid="894">从图上可以看出，CompositeByteBuf 内部维护了一个 Components 数组。在每个 Component 中存放着不同的 ByteBuf，各个 ByteBuf 独立维护自己的读写索引，而 CompositeByteBuf 自身也会单独维护一个读写索引。由此可见，Component 是实现 CompositeByteBuf 的关键所在，下面看下 Component 结构定义：</p>
<pre class="lang-java" data-nodeid="895"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Component</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ByteBuf srcBuf; <span class="hljs-comment">// 原始的 ByteBuf</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ByteBuf buf; <span class="hljs-comment">// srcBuf 去除包装之后的 ByteBuf</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> srcAdjustment; <span class="hljs-comment">// CompositeByteBuf 的起始索引相对于 srcBuf 读索引的偏移</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> adjustment; <span class="hljs-comment">// CompositeByteBuf 的起始索引相对于 buf 的读索引的偏移</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> offset; <span class="hljs-comment">// Component 相对于 CompositeByteBuf 的起始索引位置</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> endOffset; <span class="hljs-comment">// Component 相对于 CompositeByteBuf 的结束索引位置</span>
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="896">为了方便理解上述 Component 中的属性含义，我同样以 HTTP 协议中 header 和 body 为示例，通过一张图来描述 CompositeByteBuf 组合后其中 Component 的布局情况，如下所示：</p>
<p data-nodeid="897"><img src="https://s0.lgstatic.com/i/image/M00/80/0B/Ciqc1F_Qb3yAUwbLAAVl7ZwmfJ0669.png" alt="Drawing 4.png" data-nodeid="994"></p>
<p data-nodeid="898">从图中可以看出，header 和 body 分别对应两个 ByteBuf，假设 ByteBuf 的内容分别为 "header" 和 "body"，那么 header ByteBuf 中 offset~endOffset 为 0~6，body ByteBuf 对应的 offset~endOffset 为 0~10。由此可见，Component 中的 offset 和 endOffset 可以表示当前 ByteBuf 可以读取的范围，通过 offset 和 endOffset 可以将每一个 Component 所对应的 ByteBuf 连接起来，形成一个逻辑整体。</p>
<p data-nodeid="899">此外 Component 中 srcAdjustment 和 adjustment 表示 CompositeByteBuf 起始索引相对于 ByteBuf 读索引的偏移。初始 adjustment = readIndex - offset，这样通过 CompositeByteBuf 的起始索引就可以直接定位到 Component 中 ByteBuf 的读索引位置。当 header ByteBuf 读取 1 个字节，body ByteBuf 读取 2 个字节，此时每个 Component 的属性又会发生什么变化呢？如下图所示。</p>
<p data-nodeid="900"><img src="https://s0.lgstatic.com/i/image/M00/80/17/CgqCHl_Qb4WAK864AAZiyrv77BY848.png" alt="Drawing 5.png" data-nodeid="1015"></p>
<p data-nodeid="901">至此，CompositeByteBuf 的基本原理我们已经介绍完了，关于具体 CompositeByteBuf 数据操作的细节在这里就不做展开了，有兴趣的同学可以自己深入研究 CompositeByteBuf 的源码。</p>
<h4 data-nodeid="902">Unpooled.wrappedBuffer 操作</h4>
<p data-nodeid="903">介绍完 CompositeByteBuf 之后，再来理解 Unpooled.wrappedBuffer 操作就非常容易了，Unpooled.wrappedBuffer 同时也是创建 CompositeByteBuf 对象的另一种推荐做法。</p>
<p data-nodeid="904">Unpooled 提供了一系列用于包装数据源的 wrappedBuffer 方法，如下所示：</p>
<p data-nodeid="905"><img src="https://s0.lgstatic.com/i/image/M00/80/17/CgqCHl_Qb46AeweXAAV1hNnjjTQ381.png" alt="Drawing 6.png" data-nodeid="1022"></p>
<p data-nodeid="906">Unpooled.wrappedBuffer 方法可以将不同的数据源的一个或者多个数据包装成一个大的 ByteBuf 对象，其中数据源的类型包括 byte[]、ByteBuf、ByteBuffer。包装的过程中不会发生数据拷贝操作，包装后生成的 ByteBuf 对象和原始 ByteBuf 对象是共享底层的 byte 数组。</p>
<h4 data-nodeid="907">ByteBuf.slice 操作</h4>
<p data-nodeid="908">ByteBuf.slice 和 Unpooled.wrappedBuffer 的逻辑正好相反，ByteBuf.slice 是将一个 ByteBuf 对象切分成多个共享同一个底层存储的 ByteBuf 对象。</p>
<p data-nodeid="909">ByteBuf 提供了两个 slice 切分方法:</p>
<pre class="lang-java" data-nodeid="910"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ByteBuf <span class="hljs-title">slice</span><span class="hljs-params">()</span></span>;
<span class="hljs-function"><span class="hljs-keyword">public</span> ByteBuf <span class="hljs-title">slice</span><span class="hljs-params">(<span class="hljs-keyword">int</span> index, <span class="hljs-keyword">int</span> length)</span></span>;
</code></pre>
<p data-nodeid="911">假设我们已经有一份完整的 HTTP 数据，可以通过 slice 方法切分获得 header 和 body 两个 ByteBuf 对象，对应的内容分别为 "header" 和 "body"，实现方式如下：</p>
<pre class="lang-java" data-nodeid="912"><code data-language="java">ByteBuf httpBuf = ...
ByteBuf header = httpBuf.slice(<span class="hljs-number">0</span>, <span class="hljs-number">6</span>);
ByteBuf body = httpBuf.slice(<span class="hljs-number">6</span>, <span class="hljs-number">4</span>);
</code></pre>
<p data-nodeid="913">通过 slice 切分后都会返回一个新的 ByteBuf 对象，而且新的对象有自己独立的 readerIndex、writerIndex 索引，如下图所示。由于新的 ByteBuf 对象与原始的 ByteBuf 对象数据是共享的，所以通过新的 ByteBuf 对象进行数据操作也会对原始 ByteBuf 对象生效。</p>
<p data-nodeid="914"><img src="https://s0.lgstatic.com/i/image/M00/80/29/Ciqc1F_QgzOAPobiAAKi9x9FxTM445.png" alt="图片8.png" data-nodeid="1042"></p>
<h4 data-nodeid="915">文件传输 FileRegion</h4>
<p data-nodeid="916">在 Netty 源码的 example 包中，提供了 FileRegion 的使用示例，以下代码片段摘自 FileServerHandler.java。</p>
<pre class="lang-java" data-nodeid="917"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead0</span><span class="hljs-params">(ChannelHandlerContext ctx, String msg)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; RandomAccessFile raf = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">long</span> length = -<span class="hljs-number">1</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; raf = <span class="hljs-keyword">new</span> RandomAccessFile(msg, <span class="hljs-string">"r"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; length = raf.length();
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; &nbsp; &nbsp; ctx.writeAndFlush(<span class="hljs-string">"ERR: "</span> + e.getClass().getSimpleName() + <span class="hljs-string">": "</span> + e.getMessage() + <span class="hljs-string">'\n'</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (length &lt; <span class="hljs-number">0</span> &amp;&amp; raf != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; raf.close();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; ctx.write(<span class="hljs-string">"OK: "</span> + raf.length() + <span class="hljs-string">'\n'</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ctx.pipeline().get(SslHandler.class) == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; // SSL not enabled - can use zero-copy file transfer.
&nbsp; &nbsp; &nbsp; &nbsp; ctx.write(<span class="hljs-keyword">new</span> DefaultFileRegion(raf.getChannel(), <span class="hljs-number">0</span>, length));
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// SSL enabled - cannot use zero-copy file transfer.</span>
&nbsp; &nbsp; &nbsp; &nbsp; ctx.write(<span class="hljs-keyword">new</span> ChunkedFile(raf));
&nbsp; &nbsp; }
&nbsp; &nbsp; ctx.writeAndFlush(<span class="hljs-string">"\n"</span>);
}
</code></pre>
<p data-nodeid="918">从 FileRegion 的使用示例可以看出，Netty 使用 FileRegion 实现文件传输的零拷贝。FileRegion 的默认实现类是 DefaultFileRegion，通过 DefaultFileRegion 将文件内容写入到 NioSocketChannel。那么 FileRegion 是如何实现零拷贝的呢？我们通过源码看看 FileRegion 到底使用了什么黑科技。</p>
<pre class="lang-java" data-nodeid="919"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultFileRegion</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractReferenceCounted</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">FileRegion</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> File f; <span class="hljs-comment">// 传输的文件</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> position; <span class="hljs-comment">// 文件的起始位置</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> count; <span class="hljs-comment">// 传输的字节数</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> transferred; <span class="hljs-comment">// 已经写入的字节数</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> FileChannel file; <span class="hljs-comment">// 文件对应的 FileChannel</span>

&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">transferTo</span><span class="hljs-params">(WritableByteChannel target, <span class="hljs-keyword">long</span> position)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> count = <span class="hljs-keyword">this</span>.count - position;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (count &lt; <span class="hljs-number">0</span> || position &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"position out of range: "</span> + position +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">" (expected: 0 - "</span> + (<span class="hljs-keyword">this</span>.count - <span class="hljs-number">1</span>) + <span class="hljs-string">')'</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (count == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-number">0L</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (refCnt() == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalReferenceCountException(<span class="hljs-number">0</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; open();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> written = file.transferTo(<span class="hljs-keyword">this</span>.position + position, count, target);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (written &gt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; transferred += written;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (written == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; validate(<span class="hljs-keyword">this</span>, position);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> written;
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="920">从源码可以看出，FileRegion 其实就是对 FileChannel 的包装，并没有什么特殊操作，底层使用的是 JDK NIO 中的 FileChannel#transferTo() 方法实现文件传输，所以 FileRegion 是操作系统级别的零拷贝，对于传输大文件会很有帮助。</p>
<p data-nodeid="921">到此为止，Netty 相关的零拷贝技术都已经介绍完了，可以看出 Netty 对于 ByteBuf 做了更多精进的设计和优化。</p>
<h3 data-nodeid="922">总结</h3>
<p data-nodeid="923">零拷贝是网络编程中一种常用的技术，可以用于优化网络数据传输的性能。本文介绍了操作系统 Linux 和 Netty 中的零拷贝技术，Netty 除了支持操作系统级别的零拷贝，更多提供了面向用户态的零拷贝特性，主要体现在 5 个方面：堆外内存、CompositeByteBuf、Unpooled.wrappedBuffer、ByteBuf.slice 以及 FileRegion。以操作系统的角度来看，零拷贝是一个广义的概念，可以认为只要能够减少不必要的 CPU 拷贝，都可以理解为是零拷贝。</p>
<p data-nodeid="924" class="">最后，留一个思考题，使用具备零拷贝特性的 transfer() 方法拷贝文件，一定会比传统 I/O 的方式更高效吗？</p>

---

### 精选评论

##### 无：
> 2020-12-10 打卡，这章写得不错，通俗易懂。

##### **亮：
> 零拷贝是不是就是说明当前的一个用户发起的操作，举例通过浏览器打开一个网络图片。正常应该是用户通过网络发起读取请求。第一次系统dma拷贝：读取图片，系统磁盘到系统内核缓冲，第二次cpu拷贝：系统内核缓冲拷贝到用户缓冲，第三次cpu拷贝：用户执行读取并发送，用户缓冲拷贝到系统socket缓冲，第四次系统dma拷贝：cpu拷贝系统通过socket发送，socket缓冲拷贝到网卡。这里面的cpu拷贝可以省略掉，也就是直接使用系统的DMA(直接内存访问，网卡设备可以直接访问内存)拷贝访问磁盘到内核缓冲，DMA拷贝socket缓冲到网卡设备。不再进行cpu拷贝(内核空间和用户空间的切换)。这就是零拷贝的原理吧。

##### **用户0488：
> 真香，目前看过最通俗最深入最全面的Netty技术资料

##### **浩：
> 深入浅出，nice

##### **3997：
> 老师，能说一下这几种分配ByteBuf的应用场景吗？最佳实践是怎样的？是不是默认用最后一种就可以了？ByteBuf buffer1 = Unpooled.buffer();ByteBuf buffer2 = PooledByteBufAllocator.DEFAULT.buffer();ByteBuf buffer3 = ByteBufAllocator.DEFAULT.buffer();

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看下 11 课 ByteBuf 的分类，还是需要根据实际情况进行选择。

##### **5134：
> 真好

