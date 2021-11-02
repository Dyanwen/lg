<p data-nodeid="1257" class="">你好，我是若地。通过前面课程的学习，我们已经知道 Netty 高性能的奥秘在于其 <strong data-nodeid="1373">Reactor 线程模型。</strong> EventLoop 是 Netty Reactor 线程模型的核心处理引擎，那么它是如何高效地实现事件循环和任务处理机制的呢？本节课我们就一起学习 EventLoop 的实现原理和最佳实践。</p>
<h3 data-nodeid="1258">再谈 Reactor 线程模型</h3>
<p data-nodeid="1259">网络框架的设计离不开 I/O 线程模型，线程模型的优劣直接决定了系统的吞吐量、可扩展性、安全性等。目前主流的网络框架几乎都采用了 I/O 多路复用的方案。Reactor 模式作为其中的事件分发器，负责将读写事件分发给对应的读写事件处理者。大名鼎鼎的 Java 并发包作者 Doug Lea，在 <em data-nodeid="1384">Scalable I/O in Java</em> 一文中阐述了服务端开发中 I/O 模型的演进过程。Netty 中<strong data-nodeid="1385">三种 Reactor 线程模型</strong>也来源于这篇经典文章。下面我们对这三种 Reactor 线程模型做一个详细的分析。</p>
<h4 data-nodeid="1260">单线程模型</h4>
<p data-nodeid="2859"><img src="https://s0.lgstatic.com/i/image/M00/64/C9/Ciqc1F-ZNCCANWF0AAG4qWOzD48243.png" alt="1.png" data-nodeid="2862"><br>
（<a href="http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf%5D" data-nodeid="2867">摘自 Lea D. Scalable IO in Java</a>）</p>




<p data-nodeid="1262">上图描述了 Reactor 的单线程模型结构，在 Reactor 单线程模型中，所有 I/O 操作（包括连接建立、数据读写、事件分发等），都是由一个线程完成的。单线程模型逻辑简单，缺陷也十分明显：</p>
<ul data-nodeid="1263">
<li data-nodeid="1264">
<p data-nodeid="1265">一个线程支持处理的连接数非常有限，CPU 很容易打满，性能方面有明显瓶颈；</p>
</li>
<li data-nodeid="1266">
<p data-nodeid="1267">当多个事件被同时触发时，只要有一个事件没有处理完，其他后面的事件就无法执行，这就会造成消息积压及请求超时；</p>
</li>
<li data-nodeid="1268">
<p data-nodeid="1269">线程在处理 I/O 事件时，Select 无法同时处理连接建立、事件分发等操作；</p>
</li>
<li data-nodeid="1270">
<p data-nodeid="1271">如果 I/O 线程一直处于满负荷状态，很可能造成服务端节点不可用。</p>
</li>
</ul>
<h4 data-nodeid="1272">多线程模型</h4>
<p data-nodeid="4785"><img src="https://s0.lgstatic.com/i/image/M00/64/C9/Ciqc1F-ZNCyAeFxmAAH_Xaxv5gc975.png" alt="2.png" data-nodeid="4788"><br>
（<a href="http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf%5D" data-nodeid="4793">摘自 Lea D. Scalable IO in Java</a>）</p>




<p data-nodeid="1274">由于单线程模型有性能方面的瓶颈，多线程模型作为解决方案就应运而生了。Reactor 多线程模型将业务逻辑交给多个线程进行处理。除此之外，多线程模型其他的操作与单线程模型是类似的，例如读取数据依然保留了串行化的设计。当客户端有数据发送至服务端时，Select 会监听到可读事件，数据读取完毕后提交到业务线程池中并发处理。</p>
<h4 data-nodeid="1275">主从多线程模型</h4>
<p data-nodeid="6711"><img src="https://s0.lgstatic.com/i/image/M00/64/D5/CgqCHl-ZNDiAPgGOAAHx74H-t44265.png" alt="3.png" data-nodeid="6714"><br>
（<a href="http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf%5D" data-nodeid="6719">摘自 Lea D. Scalable IO in Java</a>）</p>




<p data-nodeid="1277">主从多线程模型由多个 Reactor 线程组成，每个 Reactor 线程都有独立的 Selector 对象。MainReactor 仅负责处理客户端连接的 Accept 事件，连接建立成功后将新创建的连接对象注册至 SubReactor。再由 SubReactor 分配线程池中的 I/O 线程与其连接绑定，它将负责连接生命周期内所有的 I/O 事件。</p>
<p data-nodeid="1278">Netty 推荐使用主从多线程模型，这样就可以轻松达到成千上万规模的客户端连接。在海量客户端并发请求的场景下，主从多线程模式甚至可以适当增加 SubReactor 线程的数量，从而利用多核能力提升系统的吞吐量。</p>
<p data-nodeid="1279">介绍了上述三种 Reactor 线程模型，再结合它们各自的架构图，我们能大致总结出 Reactor 线程模型运行机制的四个步骤，分别为<strong data-nodeid="1441">连接注册</strong>、<strong data-nodeid="1442">事件轮询</strong>、<strong data-nodeid="1443">事件分发</strong>、<strong data-nodeid="1444">任务处理</strong>，如下图所示。</p>
<p data-nodeid="8311"><img src="https://s0.lgstatic.com/i/image/M00/64/D5/CgqCHl-ZNEGAMU-zAAEsYdWKArA085.png" alt="4.png" data-nodeid="8314"></p>



<ul data-nodeid="1281">
<li data-nodeid="1282">
<p data-nodeid="1283">连接注册：Channel 建立后，注册至 Reactor 线程中的 Selector 选择器。</p>
</li>
<li data-nodeid="1284">
<p data-nodeid="1285">事件轮询：轮询 Selector 选择器中已注册的所有 Channel 的 I/O 事件。</p>
</li>
<li data-nodeid="1286">
<p data-nodeid="1287">事件分发：为准备就绪的 I/O 事件分配相应的处理线程。</p>
</li>
<li data-nodeid="1288">
<p data-nodeid="1289">任务处理：Reactor 线程还负责任务队列中的非 I/O 任务，每个 Worker 线程从各自维护的任务队列中取出任务异步执行。</p>
</li>
</ul>
<p data-nodeid="1290">以上介绍了 Reactor 线程模型的演进过程和基本原理，Netty 也同样遵循 Reactor 线程模型的运行机制，下面我们来了解一下 Netty 是如何实现 Reactor 线程模型的。</p>
<h3 data-nodeid="1291">Netty EventLoop 实现原理</h3>
<h4 data-nodeid="1292">EventLoop 是什么</h4>
<p data-nodeid="1293">EventLoop 这个概念其实并不是 Netty 独有的，它是一种<strong data-nodeid="1460">事件等待和处理的程序模型</strong>，可以解决多线程资源消耗高的问题。例如 Node.js 就采用了 EventLoop 的运行机制，不仅占用资源低，而且能够支撑了大规模的流量访问。</p>
<p data-nodeid="1294">下图展示了 EventLoop 通用的运行模式。每当事件发生时，应用程序都会将产生的事件放入事件队列当中，然后 EventLoop 会轮询从队列中取出事件执行或者将事件分发给相应的事件监听者执行。事件执行的方式通常分为<strong data-nodeid="1466">立即执行、延后执行、定期执行</strong>几种。</p>
<p data-nodeid="9905"><img src="https://s0.lgstatic.com/i/image/M00/64/C9/Ciqc1F-ZNFKAAZr4AANvWWMqnKw586.png" alt="5.png" data-nodeid="9908"></p>



<h3 data-nodeid="1296">Netty 如何实现 EventLoop</h3>
<p data-nodeid="1297">在 Netty 中 EventLoop 可以理解为 Reactor 线程模型的事件处理引擎，每个 EventLoop 线程都维护一个 Selector 选择器和任务队列 taskQueue。它主要负责处理 I/O 事件、普通任务和定时任务。</p>
<p data-nodeid="1298">Netty 中推荐使用 NioEventLoop 作为实现类，那么 Netty 是如何实现 NioEventLoop 的呢？首先我们来看 NioEventLoop 最核心的 run() 方法源码，本节课我们不会对源码做深入的分析，只是先了解 NioEventLoop 的实现结构。</p>
<pre class="lang-java" data-nodeid="1299"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">switch</span> (selectStrategy.calculateStrategy(selectNowSupplier, hasTasks())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> SelectStrategy.CONTINUE:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> SelectStrategy.BUSY_WAIT:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> SelectStrategy.SELECT:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; select(wakenUp.getAndSet(<span class="hljs-keyword">false</span>)); <span class="hljs-comment">// 轮询 I/O 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (wakenUp.get()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; selector.wakeup();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">default</span>:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (IOException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; rebuildSelector0();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handleLoopException(e);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cancelledKeys = <span class="hljs-number">0</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; needsToSelectAgain = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> ioRatio = <span class="hljs-keyword">this</span>.ioRatio;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ioRatio == <span class="hljs-number">100</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processSelectedKeys(); <span class="hljs-comment">// 处理 I/O 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; runAllTasks(); <span class="hljs-comment">// 处理所有任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> ioStartTime = System.nanoTime();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processSelectedKeys(); <span class="hljs-comment">// 处理 I/O 事件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> ioTime = System.nanoTime() - ioStartTime;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; runAllTasks(ioTime * (<span class="hljs-number">100</span> - ioRatio) / ioRatio); <span class="hljs-comment">// 处理完 I/O 事件，再处理异步任务队列</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handleLoopException(t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isShuttingDown()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; closeAll();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (confirmShutdown()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handleLoopException(t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1300">上述源码的结构比较清晰，NioEventLoop 每次循环的处理流程都包含事件轮询 select、事件处理 processSelectedKeys、任务处理 runAllTasks 几个步骤，是典型的 Reactor 线程模型的运行机制。而且 Netty 提供了一个参数 ioRatio，可以调整 I/O 事件处理和任务处理的时间比例。下面我们将着重从<strong data-nodeid="1482">事件处理</strong>和<strong data-nodeid="1483">任务处理</strong>两个核心部分出发，详细介绍 Netty EventLoop 的实现原理。</p>
<h4 data-nodeid="1301">事件处理机制</h4>
<p data-nodeid="11499"><img src="https://s0.lgstatic.com/i/image/M00/64/C9/Ciqc1F-ZNGGAcJWSAATcxrhDB1U168.png" alt="6.png" data-nodeid="11502"></p>



<p data-nodeid="1303">结合 Netty 的整体架构，我们一起看下 EventLoop 的事件流转图，以便更好地理解 Netty EventLoop 的设计原理。NioEventLoop 的事件处理机制采用的是<strong data-nodeid="1493">无锁串行化的设计思路</strong>。</p>
<ul data-nodeid="1304">
<li data-nodeid="1305">
<p data-nodeid="1306"><strong data-nodeid="1506">BossEventLoopGroup</strong> 和 <strong data-nodeid="1507">WorkerEventLoopGroup</strong> 包含一个或者多个 NioEventLoop。BossEventLoopGroup 负责监听客户端的 Accept 事件，当事件触发时，将事件注册至 WorkerEventLoopGroup 中的一个 NioEventLoop 上。每新建一个 Channel， 只选择一个 NioEventLoop 与其绑定。所以说 Channel 生命周期的所有事件处理都是<strong data-nodeid="1508">线程独立</strong>的，不同的 NioEventLoop 线程之间不会发生任何交集。</p>
</li>
<li data-nodeid="1307">
<p data-nodeid="1308">NioEventLoop 完成数据读取后，会调用绑定的 ChannelPipeline 进行事件传播，ChannelPipeline 也是<strong data-nodeid="1518">线程安全</strong>的，数据会被传递到 ChannelPipeline 的第一个 ChannelHandler 中。数据处理完成后，将加工完成的数据再传递给下一个 ChannelHandler，整个过程是<strong data-nodeid="1519">串行化</strong>执行，不会发生线程上下文切换的问题。</p>
</li>
</ul>
<p data-nodeid="1309">NioEventLoop 无锁串行化的设计不仅使系统吞吐量达到最大化，而且降低了用户开发业务逻辑的难度，不需要花太多精力关心线程安全问题。虽然单线程执行避免了线程切换，但是它的缺陷就是不能执行时间过长的 I/O 操作，一旦某个 I/O 事件发生阻塞，那么后续的所有 I/O 事件都无法执行，甚至造成事件积压。在使用 Netty 进行程序开发时，我们一定要对 ChannelHandler 的实现逻辑有充分的风险意识。</p>
<p data-nodeid="1310">NioEventLoop 线程的可靠性至关重要，一旦 NioEventLoop 发生阻塞或者陷入空轮询，就会导致整个系统不可用。在 JDK 中， Epoll 的实现是存在漏洞的，即使 Selector 轮询的事件列表为空，NIO 线程一样可以被唤醒，导致 CPU 100% 占用。这就是臭名昭著的 JDK epoll 空轮询的 Bug。Netty 作为一个高性能、高可靠的网络框架，需要保证 I/O 线程的安全性。那么它是如何解决 JDK epoll 空轮询的 Bug 呢？实际上 Netty 并没有从根源上解决该问题，而是巧妙地规避了这个问题。</p>
<p data-nodeid="1311">我们抛开其他细枝末节，直接定位到事件轮询 select() 方法中的最后一部分代码，一起看下 Netty 是如何解决 epoll 空轮询的 Bug。</p>
<pre class="lang-java" data-nodeid="1312"><code data-language="java"><span class="hljs-keyword">long</span> time = System.nanoTime();
<span class="hljs-keyword">if</span> (time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) &gt;= currentTimeNanos) {
&nbsp; &nbsp; selectCnt = <span class="hljs-number">1</span>;
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (SELECTOR_AUTO_REBUILD_THRESHOLD &gt; <span class="hljs-number">0</span> &amp;&amp;
&nbsp; &nbsp; &nbsp; &nbsp; selectCnt &gt;= SELECTOR_AUTO_REBUILD_THRESHOLD) {
&nbsp; &nbsp; selector = selectRebuildSelector(selectCnt);
&nbsp; &nbsp; selectCnt = <span class="hljs-number">1</span>;
&nbsp; &nbsp; <span class="hljs-keyword">break</span>;
}
</code></pre>
<p data-nodeid="1313">Netty 提供了一种检测机制判断线程是否可能陷入空轮询，具体的实现方式如下：</p>
<ol data-nodeid="1314">
<li data-nodeid="1315">
<p data-nodeid="1316">每次执行 Select 操作之前记录当前时间 currentTimeNanos。</p>
</li>
<li data-nodeid="1317">
<p data-nodeid="1318">time - TimeUnit.MILLISECONDS.toNanos(timeoutMillis) &gt;= currentTimeNanos，如果事件轮询的持续时间大于等于 timeoutMillis，那么说明是正常的，否则表明阻塞时间并未达到预期，可能触发了空轮询的 Bug。</p>
</li>
<li data-nodeid="1319">
<p data-nodeid="1320">Netty 引入了计数变量 selectCnt。在正常情况下，selectCnt 会重置，否则会对 selectCnt 自增计数。当 selectCnt 达到 SELECTOR_AUTO_REBUILD_THRESHOLD（默认512） 阈值时，会触发重建 Selector 对象。</p>
</li>
</ol>
<p data-nodeid="1321">Netty 采用这种方法巧妙地规避了 JDK Bug。异常的 Selector 中所有的 SelectionKey 会重新注册到新建的 Selector 上，重建完成之后异常的 Selector 就可以废弃了。</p>
<h4 data-nodeid="1322">任务处理机制</h4>
<p data-nodeid="1323">NioEventLoop 不仅负责处理 I/O 事件，还要兼顾执行任务队列中的任务。任务队列遵循 FIFO 规则，可以保证任务执行的公平性。NioEventLoop 处理的任务类型基本可以分为三类。</p>
<ol data-nodeid="1324">
<li data-nodeid="1325">
<p data-nodeid="1326"><strong data-nodeid="1540">普通任务</strong>：通过 NioEventLoop 的 execute() 方法向任务队列 taskQueue 中添加任务。例如 Netty 在写数据时会封装 WriteAndFlushTask 提交给 taskQueue。taskQueue 的实现类是多生产者单消费者队列 MpscChunkedArrayQueue，在多线程并发添加任务时，可以保证线程安全。</p>
</li>
<li data-nodeid="1327">
<p data-nodeid="1328"><strong data-nodeid="1545">定时任务</strong>：通过调用 NioEventLoop 的 schedule() 方法向定时任务队列 scheduledTaskQueue 添加一个定时任务，用于周期性执行该任务。例如，心跳消息发送等。定时任务队列 scheduledTaskQueue 采用优先队列 PriorityQueue 实现。</p>
</li>
<li data-nodeid="1329">
<p data-nodeid="1330"><strong data-nodeid="1550">尾部队列</strong>：tailTasks 相比于普通任务队列优先级较低，在每次执行完 taskQueue 中任务后会去获取尾部队列中任务执行。尾部任务并不常用，主要用于做一些收尾工作，例如统计事件循环的执行时间、监控信息上报等。</p>
</li>
</ol>
<p data-nodeid="1331">下面结合任务处理 runAllTasks 的源码结构，分析下 NioEventLoop 处理任务的逻辑，源码实现如下：</p>
<pre class="lang-java" data-nodeid="1332"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">runAllTasks</span><span class="hljs-params">(<span class="hljs-keyword">long</span> timeoutNanos)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 1. 合并定时任务到普通任务队列</span>
&nbsp; &nbsp; fetchFromScheduledTaskQueue();
&nbsp; &nbsp; <span class="hljs-comment">// 2. 从普通任务队列中取出任务</span>
&nbsp; &nbsp; Runnable task = pollTask();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (task == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; afterRunningAllTasks();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 3. 计算任务处理的超时时间</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> deadline = ScheduledFutureTask.nanoTime() + timeoutNanos;
&nbsp; &nbsp; <span class="hljs-keyword">long</span> runTasks = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">long</span> lastExecutionTime;
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 4. 安全执行任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; safeExecute(task);
&nbsp; &nbsp; &nbsp; &nbsp; runTasks ++;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 5. 每执行 64 个任务检查一下是否超时</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> ((runTasks &amp; <span class="hljs-number">0x3F</span>) == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; lastExecutionTime = ScheduledFutureTask.nanoTime();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (lastExecutionTime &gt;= deadline) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; task = pollTask();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (task == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; lastExecutionTime = ScheduledFutureTask.nanoTime();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 6. 收尾工作</span>
&nbsp; &nbsp; afterRunningAllTasks();
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.lastExecutionTime = lastExecutionTime;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
}
</code></pre>
<p data-nodeid="1333">我在代码中以注释的方式标注了具体的实现步骤，可以分为 6 个步骤。</p>
<ol data-nodeid="1334">
<li data-nodeid="1335">
<p data-nodeid="1336">fetchFromScheduledTaskQueue 函数：将定时任务从 scheduledTaskQueue 中取出，聚合放入普通任务队列 taskQueue 中，只有定时任务的截止时间小于当前时间才可以被合并。</p>
</li>
<li data-nodeid="1337">
<p data-nodeid="1338">从普通任务队列 taskQueue 中取出任务。</p>
</li>
<li data-nodeid="1339">
<p data-nodeid="1340">计算任务执行的最大超时时间。</p>
</li>
<li data-nodeid="1341">
<p data-nodeid="1342">safeExecute 函数：安全执行任务，实际直接调用的 Runnable 的 run() 方法。</p>
</li>
<li data-nodeid="1343">
<p data-nodeid="1344">每执行 64 个任务进行超时时间的检查，如果执行时间大于最大超时时间，则立即停止执行任务，避免影响下一轮的 I/O 事件的处理。</p>
</li>
<li data-nodeid="1345">
<p data-nodeid="1346">最后获取尾部队列中的任务执行。</p>
</li>
</ol>
<h3 data-nodeid="1347">EventLoop 最佳实践</h3>
<p data-nodeid="1348">在日常开发中用好 EventLoop 至关重要，这里结合实际工作中的经验给出一些 EventLoop 的最佳实践方案。</p>
<ol data-nodeid="1349">
<li data-nodeid="1350">
<p data-nodeid="1351">网络连接建立过程中三次握手、安全认证的过程会消耗不少时间。这里建议采用 Boss 和 Worker 两个 EventLoopGroup，有助于分担 Reactor 线程的压力。</p>
</li>
<li data-nodeid="1352">
<p data-nodeid="1353">由于 Reactor 线程模式适合处理耗时短的任务场景，对于耗时较长的 ChannelHandler 可以考虑维护一个业务线程池，将编解码后的数据封装成 Task 进行异步处理，避免 ChannelHandler 阻塞而造成 EventLoop 不可用。</p>
</li>
<li data-nodeid="1354">
<p data-nodeid="1355">如果业务逻辑执行时间较短，建议直接在 ChannelHandler 中执行。例如编解码操作，这样可以避免过度设计而造成架构的复杂性。</p>
</li>
<li data-nodeid="1356">
<p data-nodeid="1357">不宜设计过多的 ChannelHandler。对于系统性能和可维护性都会存在问题，在设计业务架构的时候，需要明确业务分层和 Netty 分层之间的界限。不要一味地将业务逻辑都添加到 ChannelHandler 中。</p>
</li>
</ol>
<h3 data-nodeid="1358">总结</h3>
<p data-nodeid="1359">本节课我们一起学习了 Netty Reactor 线程模型的核心处理引擎 EventLoop，熟悉了 EventLoop 的来龙去脉。结合 Reactor 主从多线程模型，我们对 Netty EventLoop 的功能用处做一个简单的归纳总结。</p>
<ul data-nodeid="1360">
<li data-nodeid="1361">
<p data-nodeid="1362">MainReactor 线程：处理客户端请求接入。</p>
</li>
<li data-nodeid="1363">
<p data-nodeid="1364">SubReactor 线程：数据读取、I/O 事件的分发与执行。</p>
</li>
<li data-nodeid="1365">
<p data-nodeid="1366">任务处理线程：用于执行普通任务或者定时任务，如空闲连接检测、心跳上报等。</p>
</li>
</ul>
<p data-nodeid="1367" class="">EventLoop 的设计思想被运用于较多的高性能框架中，如 Redis、Nginx、Node.js 等，它的设计原理是否对你有所启发呢？在后续源码篇的章节中我们将进一步介绍 EventLoop 的源码实现，吃透 EventLoop 这个死循环，可以说你就是一个 Netty 专家了。</p>

---

### 精选评论

##### *君：
> 老师您好，我看了下netty源码，发现服务端bind时只使用了BossEventLoopGroup中的一个eventloop。即使BossEventLoopGroup设置了多个线程，最终也只用到其中一个？那么是不是只需要设置一个线程就好了，给多了也是浪费。

##### *镇：
> 老师，eventloop负责事件轮询和事件执行，任务执行，那事件分发是谁负责尼，还有nioserversocketchannel 和niosocketchannel到底啥区别尼，服务端有两个通道吗，nioeventloop完成数据读取，是指从哪个通道完成数据读取尼，，求大佬解答

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 事件分发和执行可以统一理解，都是EventLoop负责的，Reactor 主从多线程模型中，主 Reactor 负责接收客户端连接，并负责分发到从 Reactor 中。2. Netty 服务端 Channel 的类型是 NioServerSocketChannel，而客户端 Channel 的类型是 NioSocketChannel。3. 数据是从客户端 Channel 读取的。

##### logan：
> 如果客户端连接过多，而 WorkerEventLoop不够的时候，是怎么处理的？例如4核cpu默认初始化的NioEventLoop是8个，有20个客户端连接了，这时候是不是会创建20个NioSocketChannel,然后绑定到这8个EventLoop上，那必然会有channel 共享eventLoop吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你分析的是对的，海量 Channel 共享一定数量的 EventLoop

##### **新：
> 老师求回复，目前看了4章了，感觉老师画的图好漂亮啊，可以告知用什么软件吗，求回复😀

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; Sketch

##### **嵩：
> 一个线程支持处理的连接数非常有限，CPU 很容易打满，性能方面有明显瓶颈；这句话的意思是单线程模型，即便是多核CPU也只是使用一个物理CPU，不会切换着使用多核CPU吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 比较准确地说是造成单核CPU 100%。如果双核机器中只有这一个线程，那么它同一时刻只能被调度到一个CPU核上，该线程即使一直处于死循环状态，从整体看 CPU 是使用率可能只有 50%。

##### **长生：
> 老师你好，你在其他人的回复中有说道海量 Channel 共享一定数量的 EventLoop。比如十个channel由一个eventloop处理，其中有个channel里的IO操作耗时很长，是会阻塞其他九个channel吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是会阻塞的

##### Q：
> 老师好，问些问题~1、文中所指的业务线程池，是在某些channelHandler中把task丢到自定义的线程池中么，那什么时候拿到对应的结果输出到outbound中进行返回？2、每一个eventloop都有关联的taskQueue，类似于一个线程一个任务队列，和线程池多个线程一个任务队列的设计不太一样，那么boss应该也有分配任务给各个eventloop的分配策略吧？3、后续能否对NGINX等工具关于IO多路复用的设计点进行一下拓展~

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 当异步线程处理完后，可以主动触发数据回写，例如调用 writeAndFlush，只要你持有 ctx 的引用即可。
2. Boss 会为客户端 Channel 选择一个 Worker，Boss 和 Worker 都是操作各自的任务队列。
3. Nginx 是多进程模型，有异曲同工之处，之前排了 Nginx 和 Redis I/O 模型和内存模型，由于篇幅原因删除了，怕偏离主线。没关系，我们可以课后一起探讨。

##### **平：
> 老师，课程内容全部掌握消化简历可以写熟练掌握netty吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果课程内容全部都消化，可以达到熟练掌握的水平，还是要自己多加实践积累经验。

##### **丁：
> 老师这个只有io事件处理完，才会执行任务队列吧，这个任务队列是新开一个线程还是和处理io事件一样的线程会阻塞

##### **宇：
> 既然单线程模型有问题，为什么Redis用的是单线程呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. Redis 是纯内存操作。
2. 协议足够简单，大部分操作都非常简单。
3. 单线程避免上下文切换开销。这一点 Netty 也有异曲同工之处，在每一个 Reactor 线程内都是串行无锁化的，避免线程切换。
4. Redis 6 也出现了多线程的特性，只能说 Redis 单线程已经可以达到很高的性能，比较吃紧的是带宽和内存。

##### **安：
> 打卡

##### coder：
> 图跟着画一遍，理解更清晰

##### *乐：
> Channelhandle里面如果就是需要做io 操作，是不是应该自己在代码中，在异步开启新的线程处理那个业务，或者通过其他的方式，实现异步操作？常见的方案有哪些呀？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 耗时的 I/O 操作一般可以采用 JDK 原生的线程池或者 Netty 自己的非 I/O 线程池，Netty 也比较推荐自己的实现的非业务线程池。

##### *凯：
> 这么好看的流程图，用什么软件画的

