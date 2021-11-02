<p data-nodeid="909" class="">在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=393#/detail/pc?id=4269" data-nodeid="1005">第 17 课时</a>中，我们详细介绍了 dubbo-remoting-api 模块中 Transporter 相关的核心抽象接口，本课时将继续介绍 dubbo-remoting-api 模块的其他内容。这里我们依旧从 Transporter 层的 RemotingServer、Client、Channel、ChannelHandler 等核心接口出发，介绍这些核心接口的实现。</p>
<h3 data-nodeid="910">AbstractPeer 抽象类</h3>
<p data-nodeid="911">首先，我们来看 AbstractPeer 这个抽象类，它同时实现了 Endpoint 接口和 ChannelHandler 接口，如下图所示，它也是 AbstractChannel、AbstractEndpoint 抽象类的父类。</p>
<p data-nodeid="912"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wb8eAHyD_AAFkwn8xp18694.png" alt="Drawing 0.png" data-nodeid="1011"></p>
<div data-nodeid="913"><p style="text-align:center">AbstractPeer 继承关系</p></div>
<blockquote data-nodeid="914">
<p data-nodeid="915">Netty 中也有 ChannelHandler、Channel 等接口，但无特殊说明的情况下，这里的接口指的都是 Dubbo 中定义的接口。如果涉及 Netty 中的接口，会进行特殊说明。</p>
</blockquote>
<p data-nodeid="916">AbstractPeer 中有四个字段：一个是表示该端点自身的 URL 类型的字段，还有两个 Boolean 类型的字段（closing 和 closed）用来记录当前端点的状态，这三个字段都与 Endpoint 接口相关；第四个字段指向了一个 ChannelHandler 对象，AbstractPeer 对 ChannelHandler 接口的所有实现，都是委托给了这个 ChannelHandler 对象。从上面的继承关系图中，我们可以得出这样一个结论：AbstractChannel、AbstractServer、AbstractClient 都是要关联一个 ChannelHandler 对象的。</p>
<h3 data-nodeid="917">AbstractEndpoint 抽象类</h3>
<p data-nodeid="918">我们顺着上图的继承关系继续向下看，AbstractEndpoint 继承了 AbstractPeer 这个抽象类。AbstractEndpoint 中维护了一个 Codec2 对象（codec 字段）和两个超时时间（timeout 字段和 connectTimeout 字段），在 AbstractEndpoint 的构造方法中会根据传入的 URL 初始化这三个字段：</p>
<pre class="lang-java" data-nodeid="919"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AbstractEndpoint</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(url, handler); <span class="hljs-comment">// 调用父类AbstractPeer的构造方法</span>
    <span class="hljs-comment">// 根据URL中的codec参数值，确定此处具体的Codec2实现类</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.codec = getChannelCodec(url);
    <span class="hljs-comment">// 根据URL中的timeout参数确定timeout字段的值，默认1000</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.timeout = url.getPositiveParameter(TIMEOUT_KEY,
         DEFAULT_TIMEOUT);
    <span class="hljs-comment">// 根据URL中的connect.timeout参数确定connectTimeout字段的值，默认3000</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.connectTimeout = url.getPositiveParameter(
    Constants.CONNECT_TIMEOUT_KEY, Constants.DEFAULT_CONNECT_TIMEOUT);
}
</code></pre>
<p data-nodeid="920">在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=393#/detail/pc?id=4269" data-nodeid="1019">第 17 课时</a>介绍 Codec2 接口的时候提到它是一个 SPI 扩展点，这里的 AbstractEndpoint.getChannelCodec() 方法就是基于 Dubbo SPI 选择其扩展实现的，具体实现如下：</p>
<pre class="lang-java" data-nodeid="921"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">static</span> Codec2 <span class="hljs-title">getChannelCodec</span><span class="hljs-params">(URL url)</span> </span>{
    <span class="hljs-comment">// 根据URL的codec参数获取扩展名</span>
&nbsp; &nbsp; String codecName = url.getParameter(Constants.CODEC_KEY, <span class="hljs-string">"telnet"</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ExtensionLoader.getExtensionLoader(Codec2.class).hasExtension(codecName)) { // 通过ExtensionLoader加载并实例化Codec2的具体扩展实现
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ExtensionLoader.getExtensionLoader(Codec2.class).getExtension(codecName);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// Codec2接口不存在相应的扩展名，就尝试从Codec这个老接口的扩展名中查找，目前Codec接口已经废弃了，所以省略这部分逻辑</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="922">另外，AbstractEndpoint 还实现了 Resetable 接口（只有一个 reset() 方法需要实现），虽然 AbstractEndpoint 中的 reset() 方法比较长，但是逻辑非常简单，就是根据传入的 URL 参数重置 AbstractEndpoint 的三个字段。下面是重置 codec 字段的代码片段，还是调用 getChannelCodec() 方法实现的：</p>
<pre class="lang-java" data-nodeid="923"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">reset</span><span class="hljs-params">(URL url)</span> </span>{
    <span class="hljs-comment">// 检测当前AbstractEndpoint是否已经关闭(略)</span>
    <span class="hljs-comment">// 省略重置timeout、connectTimeout两个字段的逻辑</span>
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (url.hasParameter(Constants.CODEC_KEY)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.codec = getChannelCodec(url);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; logger.error(t.getMessage(), t);
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="924">Server 继承路线分析</h3>
<p data-nodeid="5169" class="">AbstractServer 和 AbstractClient 都实现了 AbstractEndpoint 抽象类，我们先来看 AbstractServer 的实现。AbstractServer 在继承了 AbstractEndpoint 的同时，还实现了 RemotingServer 接口，如下图所示：</p>










<p data-nodeid="926"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wb-iAMAgtAACJWi59iSc812.png" alt="Drawing 1.png" data-nodeid="1026"></p>
<div data-nodeid="927"><p style="text-align:center">AbstractServer 继承关系图</p></div>
<p data-nodeid="928"><strong data-nodeid="1031">AbstractServer 是对服务端的抽象，实现了服务端的公共逻辑</strong>。AbstractServer 的核心字段有下面几个。</p>
<ul data-nodeid="929">
<li data-nodeid="930">
<p data-nodeid="931">localAddress、bindAddress（InetSocketAddress 类型）：分别对应该 Server 的本地地址和绑定的地址，都是从 URL 中的参数中获取。bindAddress 默认值与 localAddress 一致。</p>
</li>
<li data-nodeid="932">
<p data-nodeid="933">accepts（int 类型）：该 Server 能接收的最大连接数，从 URL 的 accepts 参数中获取，默认值为 0，表示没有限制。</p>
</li>
<li data-nodeid="934">
<p data-nodeid="935">executorRepository（ExecutorRepository 类型）：负责管理线程池，后面我们会深入介绍 ExecutorRepository 的具体实现。</p>
</li>
<li data-nodeid="936">
<p data-nodeid="937">executor（ExecutorService 类型）：当前 Server 关联的线程池，由上面的 ExecutorRepository 创建并管理。</p>
</li>
</ul>
<p data-nodeid="938">在 AbstractServer 的构造方法中会根据传入的 URL初始化上述字段，并调用 doOpen() 这个抽象方法完成该 Server 的启动，具体实现如下：</p>
<pre class="lang-java" data-nodeid="939"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AbstractServer</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(url, handler); <span class="hljs-comment">// 调用父类的构造方法</span>
    <span class="hljs-comment">// 根据传入的URL初始化localAddress和bindAddress</span>
&nbsp; &nbsp; localAddress = getUrl().toInetSocketAddress();
&nbsp; &nbsp; String bindIp = getUrl().getParameter(Constants.BIND_IP_KEY, getUrl().getHost());
&nbsp; &nbsp; <span class="hljs-keyword">int</span> bindPort = getUrl().getParameter(Constants.BIND_PORT_KEY, getUrl().getPort());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (url.getParameter(ANYHOST_KEY, <span class="hljs-keyword">false</span>) || NetUtils.isInvalidLocalHost(bindIp)) {
&nbsp; &nbsp; &nbsp; &nbsp; bindIp = ANYHOST_VALUE;
&nbsp; &nbsp; }
&nbsp; &nbsp; bindAddress = <span class="hljs-keyword">new</span> InetSocketAddress(bindIp, bindPort);
    <span class="hljs-comment">//&nbsp;初始化accepts等字段</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.accepts = url.getParameter(ACCEPTS_KEY, DEFAULT_ACCEPTS);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.idleTimeout = url.getParameter(IDLE_TIMEOUT_KEY, DEFAULT_IDLE_TIMEOUT);
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; doOpen(); <span class="hljs-comment">// 调用doOpen()这个抽象方法，启动该Server</span>
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RemotingException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
    <span class="hljs-comment">// 获取该Server关联的线程池</span>
&nbsp; &nbsp; executor = executorRepository.createExecutorIfAbsent(url);
}
</code></pre>
<h4 data-nodeid="940">ExecutorRepository</h4>
<p data-nodeid="941">在继续分析 AbstractServer 的具体实现类之前，我们先来了解一下 ExecutorRepository 这个接口。</p>
<p data-nodeid="942">ExecutorRepository 负责创建并管理 Dubbo 中的线程池，该接口虽然是个 SPI 扩展点，但是只有一个默认实现—— DefaultExecutorRepository。在该默认实现中维护了一个 ConcurrentMap&lt;String, ConcurrentMap&lt;Integer, ExecutorService&gt;&gt; 集合（data 字段）缓存已有的线程池，第一层 Key 值表示线程池属于 Provider 端还是 Consumer 端，第二层 Key 值表示线程池关联服务的端口。</p>
<p data-nodeid="943">DefaultExecutorRepository.createExecutorIfAbsent() 方法会根据 URL 参数创建相应的线程池并缓存在合适的位置，具体实现如下：</p>
<pre class="lang-java" data-nodeid="944"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> ExecutorService <span class="hljs-title">createExecutorIfAbsent</span><span class="hljs-params">(URL url)</span> </span>{
    <span class="hljs-comment">// 根据URL中的side参数值决定第一层key</span>
&nbsp; &nbsp; String componentKey = EXECUTOR_SERVICE_COMPONENT_KEY;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (CONSUMER_SIDE.equalsIgnoreCase(url.getParameter(SIDE_KEY))) {
&nbsp; &nbsp; &nbsp; &nbsp; componentKey = CONSUMER_SIDE; 
&nbsp; &nbsp; }
&nbsp; &nbsp; Map&lt;Integer, ExecutorService&gt; executors = data.computeIfAbsent(componentKey, k -&gt; <span class="hljs-keyword">new</span> ConcurrentHashMap&lt;&gt;());
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;根据URL中的port值确定第二层key</span>
&nbsp; &nbsp; Integer portKey = url.getPort();
&nbsp; &nbsp; ExecutorService executor = executors.computeIfAbsent(portKey, k -&gt; createExecutor(url));
&nbsp; &nbsp; <span class="hljs-comment">// 如果缓存中相应的线程池已关闭，则同样需要调用createExecutor()方法</span>
    <span class="hljs-comment">// 创建新的线程池，并替换掉缓存中已关闭的线程持，这里省略这段逻辑</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> executor;
}
</code></pre>
<p data-nodeid="945">在 createExecutor() 方法中，会通过 Dubbo SPI 查找 ThreadPool 接口的扩展实现，并调用其 getExecutor() 方法创建线程池。ThreadPool 接口被 @SPI 注解修饰，默认使用 FixedThreadPool 实现，但是 ThreadPool 接口中的 getExecutor() 方法被 @Adaptive 注解修饰，动态生成的适配器类会优先根据 URL 中的 threadpool 参数选择 ThreadPool 的扩展实现。ThreadPool 接口的实现类如下图所示：</p>
<p data-nodeid="946"><img src="https://s0.lgstatic.com/i/image/M00/58/FE/CgqCHl9wcBeAYMZ1AABRTGzl5uY627.png" alt="Drawing 2.png" data-nodeid="1048"></p>
<div data-nodeid="947"><p style="text-align:center">ThreadPool 继承关系图</p></div>
<p data-nodeid="948">不同实现会根据 URL 参数创建不同特性的线程池，这里以<strong data-nodeid="1054">CacheThreadPool</strong>为例进行分析：</p>
<pre class="lang-java" data-nodeid="949"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Executor <span class="hljs-title">getExecutor</span><span class="hljs-params">(URL url)</span> </span>{
&nbsp; &nbsp; String name = url.getParameter(THREAD_NAME_KEY, DEFAULT_THREAD_NAME);
    <span class="hljs-comment">// 核心线程数量</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> cores = url.getParameter(CORE_THREADS_KEY, DEFAULT_CORE_THREADS);
    <span class="hljs-comment">// 最大线程数量</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> threads = url.getParameter(THREADS_KEY, Integer.MAX_VALUE);
    <span class="hljs-comment">// 缓冲队列的最大长度</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> queues = url.getParameter(QUEUES_KEY, DEFAULT_QUEUES);
    <span class="hljs-comment">// 非核心线程的最大空闲时长，当非核心线程空闲时间超过该值时，会被回收</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> alive = url.getParameter(ALIVE_KEY, DEFAULT_ALIVE);
    <span class="hljs-comment">// 下面就是依赖JDK的ThreadPoolExecutor创建指定特性的线程池并返回</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ThreadPoolExecutor(cores, threads, alive, TimeUnit.MILLISECONDS,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; queues == <span class="hljs-number">0</span> ? <span class="hljs-keyword">new</span> SynchronousQueue&lt;Runnable&gt;() :
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; (queues &lt; <span class="hljs-number">0</span> ? <span class="hljs-keyword">new</span> LinkedBlockingQueue&lt;Runnable&gt;()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; : <span class="hljs-keyword">new</span> LinkedBlockingQueue&lt;Runnable&gt;(queues)),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> NamedInternalThreadFactory(name, <span class="hljs-keyword">true</span>), <span class="hljs-keyword">new</span> AbortPolicyWithReport(name, url));
}
</code></pre>
<p data-nodeid="950">再简单说一下其他 ThreadPool 实现创建的线程池。</p>
<ul data-nodeid="951">
<li data-nodeid="952">
<p data-nodeid="953"><strong data-nodeid="1060">LimitedThreadPool</strong>：与 CacheThreadPool 一样，可以指定核心线程数、最大线程数以及缓冲队列长度。区别在于，LimitedThreadPool 创建的线程池的非核心线程不会被回收。</p>
</li>
<li data-nodeid="954">
<p data-nodeid="955"><strong data-nodeid="1065">FixedThreadPool</strong>：核心线程数和最大线程数一致，且不会被回收。</p>
</li>
</ul>
<p data-nodeid="956">上述三种类型的线程池都是基于 JDK ThreadPoolExecutor 线程池，在核心线程全部被占用的时候，会优先将任务放到缓冲队列中缓存，在缓冲队列满了之后，才会尝试创建新线程来处理任务。</p>
<p data-nodeid="957">EagerThreadPool 创建的线程池是 EagerThreadPoolExecutor（继承了 JDK 提供的 ThreadPoolExecutor），使用的队列是 TaskQueue（继承了LinkedBlockingQueue）。该线程池与 ThreadPoolExecutor 不同的是：在线程数没有达到最大线程数的前提下，EagerThreadPoolExecutor 会优先创建线程来执行任务，而不是放到缓冲队列中；当线程数达到最大值时，EagerThreadPoolExecutor 会将任务放入缓冲队列，等待空闲线程。</p>
<p data-nodeid="958">EagerThreadPoolExecutor 覆盖了 ThreadPoolExecutor 中的两个方法：execute() 方法和 afterExecute() 方法，具体实现如下，我们可以看到其中维护了一个 submittedTaskCount 字段（AtomicInteger 类型），用来记录当前在线程池中的任务总数（正在线程中执行的任务数+队列中等待的任务数）。</p>
<pre class="lang-java" data-nodeid="959"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">execute</span><span class="hljs-params">(Runnable command)</span> </span>{
    <span class="hljs-comment">// 任务提交之前，递增submittedTaskCount</span>
&nbsp; &nbsp; submittedTaskCount.incrementAndGet(); 
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>.execute(command); <span class="hljs-comment">// 提交任务</span>
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (RejectedExecutionException rx) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> TaskQueue queue = (TaskQueue) <span class="hljs-keyword">super</span>.getQueue();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 任务被拒绝之后，会尝试再次放入队列中缓存，等待空闲线程执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!queue.retryOffer(command, <span class="hljs-number">0</span>, TimeUnit.MILLISECONDS)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 再次入队被拒绝，则队列已满，无法执行任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;递减submittedTaskCount</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; submittedTaskCount.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RejectedExecutionException(<span class="hljs-string">"Queue capacity is full."</span>, rx);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (InterruptedException x) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 再次入队列异常，递减submittedTaskCount</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; submittedTaskCount.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RejectedExecutionException(x);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) { <span class="hljs-comment">// 任务提交异常，递减submittedTaskCount</span>
&nbsp; &nbsp; &nbsp; &nbsp; submittedTaskCount.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> t;
&nbsp; &nbsp; }
}

<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">afterExecute</span><span class="hljs-params">(Runnable r, Throwable t)</span> </span>{
    <span class="hljs-comment">// 任务指定结束，递减submittedTaskCount</span>
&nbsp; &nbsp; submittedTaskCount.decrementAndGet(); 
}
</code></pre>
<p data-nodeid="960">看到这里，你可能会有些疑惑：没有看到优先创建线程执行任务的逻辑啊。其实重点在关联的 TaskQueue 实现中，它覆盖了 LinkedBlockingQueue.offer() 方法，会判断线程池的 submittedTaskCount 值是否已经达到最大线程数，如果未超过，则会返回 false，迫使线程池创建新线程来执行任务。示例代码如下：</p>
<pre class="lang-java" data-nodeid="961"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">offer</span><span class="hljs-params">(Runnable runnable)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;获取当前线程池中的活跃线程数</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> currentPoolThreadSize = executor.getPoolSize();
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;当前有线程空闲，直接将任务提交到队列中，空闲线程会直接从中获取任务执行</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (executor.getSubmittedTaskCount() &lt; currentPoolThreadSize) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.offer(runnable);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 当前没有空闲线程，但是还可以创建新线程，则返回false，迫使线程池创建</span>
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;新线程来执行任务</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (currentPoolThreadSize &lt; executor.getMaximumPoolSize()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 当前线程数已经达到上限，只能放到队列中缓存了</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">super</span>.offer(runnable);
}
</code></pre>
<p data-nodeid="962">线程池最后一个相关的小细节是 AbortPolicyWithReport ，它继承了 ThreadPoolExecutor.AbortPolicy，覆盖的 rejectedExecution 方法中会输出包含线程池相关信息的 WARN 级别日志，然后进行 dumpJStack() 方法，最后才会抛出RejectedExecutionException 异常。</p>
<p data-nodeid="963">我们回到 Server 的继承线上，下面来看基于 Netty 4 实现的 NettyServer，它继承了前文介绍的 AbstractServer，实现了 doOpen() 方法和 doClose() 方法。这里重点看 doOpen() 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="964"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doOpen</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
    <span class="hljs-comment">// 创建ServerBootstrap</span>
&nbsp; &nbsp; bootstrap = <span class="hljs-keyword">new</span> ServerBootstrap(); 
    <span class="hljs-comment">//&nbsp;创建boss EventLoopGroup</span>
&nbsp; &nbsp; bossGroup = NettyEventLoopFactory.eventLoopGroup(<span class="hljs-number">1</span>, <span class="hljs-string">"NettyServerBoss"</span>);
    <span class="hljs-comment">//&nbsp;创建worker EventLoopGroup</span>
&nbsp; &nbsp; workerGroup = NettyEventLoopFactory.eventLoopGroup(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getUrl().getPositiveParameter(IO_THREADS_KEY, Constants.DEFAULT_IO_THREADS),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"NettyServerWorker"</span>);
    <span class="hljs-comment">// 创建NettyServerHandler，它是一个Netty中的ChannelHandler实现，</span>
    <span class="hljs-comment">//&nbsp;不是Dubbo Remoting层的ChannelHandler接口的实现</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> NettyServerHandler nettyServerHandler = <span class="hljs-keyword">new</span> NettyServerHandler(getUrl(), <span class="hljs-keyword">this</span>);
    <span class="hljs-comment">// 获取当前NettyServer创建的所有Channel，这里的channels集合中的</span>
    <span class="hljs-comment">// Channel不是Netty中的Channel对象，而是Dubbo Remoting层的Channel对象</span>
&nbsp; &nbsp; channels = nettyServerHandler.getChannels();
    <span class="hljs-comment">// 初始化ServerBootstrap，指定boss和worker EventLoopGroup</span>
&nbsp; &nbsp; bootstrap.group(bossGroup, workerGroup)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .channel(NettyEventLoopFactory.serverSocketChannelClass())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .option(ChannelOption.SO_REUSEADDR, Boolean.TRUE)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .childOption(ChannelOption.TCP_NODELAY, Boolean.TRUE)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .childOption(ChannelOption.ALLOCATOR, PooledByteBufAllocator.DEFAULT)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .childHandler(<span class="hljs-keyword">new</span> ChannelInitializer&lt;SocketChannel&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initChannel</span><span class="hljs-params">(SocketChannel ch)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 连接空闲超时时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> idleTimeout = UrlUtils.getIdleTimeout(getUrl());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;NettyCodecAdapter中会创建Decoder和Encoder</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NettyCodecAdapter adapter = <span class="hljs-keyword">new</span> NettyCodecAdapter(getCodec(), getUrl(), NettyServer.<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ch.pipeline()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 注册Decoder和Encoder</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"decoder"</span>, adapter.getDecoder())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"encoder"</span>, adapter.getEncoder())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 注册IdleStateHandler</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"server-idle-handler"</span>, <span class="hljs-keyword">new</span> IdleStateHandler(<span class="hljs-number">0</span>, <span class="hljs-number">0</span>, idleTimeout, MILLISECONDS))
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;注册NettyServerHandler</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addLast(<span class="hljs-string">"handler"</span>, nettyServerHandler);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; <span class="hljs-comment">// 绑定指定的地址和端口</span>
&nbsp; &nbsp; ChannelFuture channelFuture = bootstrap.bind(getBindAddress());
&nbsp; &nbsp; channelFuture.syncUninterruptibly(); <span class="hljs-comment">// 等待bind操作完成</span>
&nbsp; &nbsp; channel = channelFuture.channel();
}
</code></pre>
<p data-nodeid="965">看完 NettyServer 实现的 doOpen() 方法之后，你会发现它和简易版 RPC 框架中启动一个 Netty 的 Server 端基本流程类似：初始化 ServerBootstrap、创建 Boss EventLoopGroup 和 Worker EventLoopGroup、创建 ChannelInitializer 指定如何初始化 Channel 上的 ChannelHandler 等一系列 Netty 使用的标准化流程。</p>
<p data-nodeid="966">其实在 Transporter 这一层看，功能的不同其实就是注册在 Channel 上的 ChannelHandler 不同，通过 doOpen() 方法得到的 Server 端结构如下：</p>
<p data-nodeid="967"><img src="https://s0.lgstatic.com/i/image/M00/59/E4/Ciqc1F9y4LaAIHSsAADBytWDQ3U695.png" alt="5.png" data-nodeid="1076"></p>
<div data-nodeid="968"><p style="text-align:center">NettyServer 模型</p></div>
<h4 data-nodeid="969">核心 ChannelHandler</h4>
<p data-nodeid="970">下面我们来逐个看看这四个 ChannelHandler 的核心功能。</p>
<p data-nodeid="971">首先是<strong data-nodeid="1084">decoder 和 encoder</strong>，它们都是 NettyCodecAdapter 的内部类，如下图所示，分别继承了 Netty 中的 ByteToMessageDecoder 和 MessageToByteEncoder：</p>
<p data-nodeid="972"><img src="https://s0.lgstatic.com/i/image/M00/58/FE/CgqCHl9wcESANfPCAABDUdzhtNU066.png" alt="Drawing 4.png" data-nodeid="1087"></p>
<p data-nodeid="973">还记得 AbstractEndpoint 抽象类中的 codec 字段（Codec2 类型）吗？InternalDecoder 和 InternalEncoder 会将真正的编解码功能委托给 NettyServer 关联的这个 Codec2 对象去处理，这里以 InternalDecoder 为例进行分析：</p>
<pre class="lang-java" data-nodeid="974"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InternalDecoder</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ByteToMessageDecoder</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">decode</span><span class="hljs-params">(ChannelHandlerContext ctx, ByteBuf input, List&lt;Object&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 将ByteBuf封装成统一的ChannelBuffer</span>
&nbsp; &nbsp; &nbsp; &nbsp; ChannelBuffer message = <span class="hljs-keyword">new</span> NettyBackedChannelBuffer(input);
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;拿到关联的Channel</span>
&nbsp; &nbsp; &nbsp; &nbsp; NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;    <span class="hljs-comment">//&nbsp;记录当前readerIndex的位置</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> saveReaderIndex = message.readerIndex();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;委托给Codec2进行解码</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Object msg = codec.decode(channel, message);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;当前接收到的数据不足一个消息的长度，会返回NEED_MORE_INPUT，</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;这里会重置readerIndex，继续等待接收更多的数据</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (msg == Codec2.DecodeResult.NEED_MORE_INPUT) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; message.readerIndex(saveReaderIndex);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (msg != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 将读取到的消息传递给后面的Handler处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; out.add(msg);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">while</span> (message.readable());
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="975">你是不是发现 InternalDecoder 的实现与我们简易版 RPC 的 Decoder 实现非常相似呢？</p>
<p data-nodeid="976">InternalEncoder 的具体实现就不再展开讲解了，你若感兴趣可以翻看源码进行研究和分析。</p>
<p data-nodeid="977">接下来是<strong data-nodeid="1096">IdleStateHandler</strong>，它是 Netty 提供的一个工具型 ChannelHandler，用于定时心跳请求的功能或是自动关闭长时间空闲连接的功能。它的原理到底是怎样的呢？在 IdleStateHandler 中通过 lastReadTime、lastWriteTime 等几个字段，记录了最近一次读/写事件的时间，IdleStateHandler 初始化的时候，会创建一个定时任务，定时检测当前时间与最后一次读/写时间的差值。如果超过我们设置的阈值（也就是上面 NettyServer 中设置的 idleTimeout），就会触发 IdleStateEvent 事件，并传递给后续的 ChannelHandler 进行处理。后续 ChannelHandler 的 userEventTriggered() 方法会根据接收到的 IdleStateEvent 事件，决定是关闭长时间空闲的连接，还是发送心跳探活。</p>
<p data-nodeid="978">最后来看<strong data-nodeid="1102">NettyServerHandler</strong>，它继承了 ChannelDuplexHandler，这是 Netty 提供的一个同时处理 Inbound 数据和 Outbound 数据的 ChannelHandler，从下面的继承图就能看出来。</p>
<p data-nodeid="979"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wcFKAQQZ3AAB282frbWw282.png" alt="Drawing 5.png" data-nodeid="1105"></p>
<div data-nodeid="980"><p style="text-align:center">NettyServerHandler 继承关系图</p></div>
<p data-nodeid="981">在 NettyServerHandler 中有 channels 和 handler 两个核心字段。</p>
<ul data-nodeid="982">
<li data-nodeid="983">
<p data-nodeid="984">channels（Map&lt;String,Channel&gt;集合）：记录了当前 Server 创建的所有 Channel，从下图中可以看到，连接创建（触发 channelActive() 方法）、连接断开（触发 channelInactive()方法）会操作 channels 集合进行相应的增删。</p>
</li>
</ul>
<p data-nodeid="985"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wcFuABJWsAAaIoTwCIA0958.png" alt="Drawing 6.png" data-nodeid="1112"></p>
<ul data-nodeid="986">
<li data-nodeid="987">
<p data-nodeid="988">handler（ChannelHandler 类型）：NettyServerHandler 内几乎所有方法都会触发该 Dubbo ChannelHandler 对象（如下图）。</p>
</li>
</ul>
<p data-nodeid="989"><img src="https://s0.lgstatic.com/i/image/M00/58/FE/CgqCHl9wcGOAE_ykAAFvy5a4X58367.png" alt="Drawing 7.png" data-nodeid="1116"></p>
<p data-nodeid="990">这里以 write() 方法为例进行简单分析：</p>
<pre class="lang-java" data-nodeid="991"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">(ChannelHandlerContext ctx, Object msg, ChannelPromise promise)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">super</span>.write(ctx, msg, promise); <span class="hljs-comment">// 将发送的数据继续向下传递</span>
    <span class="hljs-comment">// 并不影响消息的继续发送，只是触发sent()方法进行相关的处理，这也是方法</span>
    <span class="hljs-comment">//&nbsp;名称是动词过去式的原因，可以仔细体会一下。其他方法可能没有那么明显，</span>
    <span class="hljs-comment">// 这里以write()方法为例进行说明</span>
&nbsp; &nbsp; NettyChannel channel = NettyChannel.getOrAddChannel(ctx.channel(), url, handler);
&nbsp; &nbsp; handler.sent(channel, msg);
}
</code></pre>
<p data-nodeid="992">在 NettyServer 创建 NettyServerHandler 的时候，可以看到下面的这行代码：</p>
<pre class="lang-java" data-nodeid="993"><code data-language="java"><span class="hljs-keyword">final</span> NettyServerHandler nettyServerHandler = <span class="hljs-keyword">new</span> NettyServerHandler(getUrl(), <span class="hljs-keyword">this</span>);
</code></pre>
<p data-nodeid="994">其中第二个参数传入的是 NettyServer 这个对象，你可以追溯一下 NettyServer 的继承结构，会发现它的最顶层父类 AbstractPeer 实现了 ChannelHandler，并且将所有的方法委托给其中封装的 ChannelHandler 对象，如下图所示：</p>
<p data-nodeid="995"><img src="https://s0.lgstatic.com/i/image/M00/58/F3/Ciqc1F9wcGuADQi3AAD6EEURlNU871.png" alt="Drawing 8.png" data-nodeid="1122"></p>
<p data-nodeid="996">也就是说，NettyServerHandler 会将数据委托给这个 ChannelHandler。</p>
<p data-nodeid="997">到此为止，Server 这条继承线就介绍完了。你可以回顾一下，从 AbstractPeer 开始往下，一路继承下来，NettyServer 拥有了 Endpoint、ChannelHandler 以及RemotingServer多个接口的能力，关联了一个 ChannelHandler 对象以及 Codec2 对象，并最终将数据委托给这两个对象进行处理。所以，上层调用方只需要实现 ChannelHandler 和 Codec2 这两个接口就可以了。</p>
<p data-nodeid="998"><img src="https://s0.lgstatic.com/i/image/M00/59/E4/Ciqc1F9y4MyAR8XLAABTLdOZqrc228.png" alt="6.png" data-nodeid="1127"></p>
<h3 data-nodeid="999">总结</h3>
<p data-nodeid="1000">本课时重点介绍了 Dubbo Transporter 层中 Server 相关的实现。</p>
<p data-nodeid="1001" class="">首先，我们介绍了 AbstractPeer 这个最顶层的抽象类，了解了 Server、Client 和 Channel 的公共属性。接下来，介绍了 AbstractEndpoint 抽象类，它提供了编解码等 Server 和 Client 所需的公共能力。最后，我们深入分析了 AbstractServer 抽象类以及基于 Netty 4 实现的 NettyServer，同时，还深入剖析了涉及的各种组件，例如，ExecutorRepository、NettyServerHandler 等。</p>

---

### 精选评论

##### **辉：
> 写的很细呀👍

##### *泽：
> 打卡

