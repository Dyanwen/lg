<p data-nodeid="141852" class="">在前面的第 27 课时中，我们介绍了 ProtocolFilterWrapper 的具体实现，这里简单回顾一下。在 buildInvokerChain() 方法中，ProtocolFilterWrapper 会加载 Dubbo 以及应用程序提供的 Filter 实现类，然后构造成 Filter 链，最后通过装饰者模式在原有 Invoker 对象基础上添加执行 Filter 链的逻辑。</p>
<p data-nodeid="141853">Filter 链的组装逻辑设计得非常灵活，其中可以通过“-”配置手动剔除 Dubbo 原生提供的、默认加载的 Filter，通过“default”来代替 Dubbo 原生提供的 Filter，这样就可以很好地控制哪些 Filter 要加载，以及 Filter 的真正执行顺序。</p>
<p data-nodeid="141854"><strong data-nodeid="142009">Filter 是扩展 Dubbo 功能的首选方案</strong>，并且 Dubbo 自身也提供了非常多的 Filter 实现来扩展自身功能。在回顾了 ProtocolFilterWrapper 加载 Filter 的大致逻辑之后，我们本课时就来深入介绍 Dubbo 内置的多种 Filter 实现类，以及自定义 Filter 扩展 Dubbo 的方式。</p>
<p data-nodeid="142787">在开始介绍 Filter 接口实现之前，我们需要了解一下 Filter 在 Dubbo 架构中的位置，这样才能明确 Filter 链处理请求/响应的位置，如下图红框所示：</p>
<p data-nodeid="142788" class=""><img src="https://s0.lgstatic.com/i/image/M00/68/FD/CgqCHl-lLz2APEb2ABSTPPnfqGQ345.png" alt="Lark20201106-191028.png" data-nodeid="142793"></p>
<div data-nodeid="142789"><p style="text-align:center">Filter 在 Dubbo 架构中的位置</p></div>




<h3 data-nodeid="141858">ConsumerContextFilter</h3>
<p data-nodeid="141859">ConsumerContextFilter 是一个非常简单的 Consumer 端 Filter 实现，它会在当前的 RpcContext 中记录本地调用的一些状态信息（会记录到 LOCAL 对应的 RpcContext 中），例如，调用相关的 Invoker、Invocation 以及调用的本地地址、远端地址信息，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141860"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; RpcContext context = RpcContext.getContext();
&nbsp; &nbsp; context.setInvoker(invoker) <span class="hljs-comment">// 记录Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setInvocation(invocation) <span class="hljs-comment">// 记录Invocation</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 记录本地地址以及远端地址</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setLocalAddress(NetUtils.getLocalHost(), <span class="hljs-number">0</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setRemoteAddress(invoker.getUrl().getHost(), invoker.getUrl().getPort())
            <span class="hljs-comment">// 记录远端应用名称等信息</span>
            .setRemoteApplicationName(invoker.getUrl()
                 .getParameter(REMOTE_APPLICATION_KEY))
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setAttachment(REMOTE_APPLICATION_KEY, invoker.getUrl().getParameter(APPLICATION_KEY));
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invocation <span class="hljs-keyword">instanceof</span> RpcInvocation) {
&nbsp; &nbsp; &nbsp; &nbsp; ((RpcInvocation) invocation).setInvoker(invoker);
&nbsp; &nbsp; }
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 检测是否超时</span>
&nbsp; &nbsp; Object countDown = context.get(TIME_COUNTDOWN_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (countDown != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; TimeoutCountDown timeoutCountDown = (TimeoutCountDown) countDown;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeoutCountDown.isExpired()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> AsyncRpcResult.newDefaultAsyncResult(
              <span class="hljs-keyword">new</span> RpcException(<span class="hljs-string">"...."</span>), invocation);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
}
</code></pre>
<p data-nodeid="141861">这里使用的 TimeoutCountDown 对象用于检测当前调用是否超时，其中有三个字段。</p>
<ul data-nodeid="141862">
<li data-nodeid="141863">
<p data-nodeid="141864">timeoutInMillis（long 类型）：超时时间，单位为毫秒。</p>
</li>
<li data-nodeid="141865">
<p data-nodeid="141866">deadlineInNanos（long 类型）：超时的时间戳，单位为纳秒。</p>
</li>
<li data-nodeid="141867">
<p data-nodeid="141868">expired（boolean 类型）：标识当前 TimeoutCountDown 关联的调用是否已超时。</p>
</li>
</ul>
<p data-nodeid="141869">在 TimeoutCountDown.isExpire() 方法中，会比较当前时间与 deadlineInNanos 字段记录的超时时间戳。正如上面看到的逻辑，如果请求超时，则不再发起远程调用，直接让 AsyncRpcResult 异常结束。</p>
<h3 data-nodeid="141870">ActiveLimitFilter</h3>
<p data-nodeid="141871">ActiveLimitFilter 是 Consumer 端用于限制一个 Consumer 对于一个服务端方法的并发调用量，也可以称为“客户端限流”。下面我们就来看下 ActiveLimitFilter 的具体实现：</p>
<pre class="lang-java" data-nodeid="141872"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; URL url = invoker.getUrl();&nbsp; <span class="hljs-comment">// 获得url对象</span>
&nbsp; &nbsp; String methodName = invocation.getMethodName();<span class="hljs-comment">// 获得方法名称</span>
&nbsp; &nbsp; <span class="hljs-comment">// 获取最大并发数</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> max = invoker.getUrl().getMethodParameter(methodName, ACTIVES_KEY, <span class="hljs-number">0</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 获取该方法的状态信息</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> RpcStatus rpcStatus = RpcStatus.getStatus(invoker.getUrl(), invocation.getMethodName());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!RpcStatus.beginCount(url, methodName, max)) { <span class="hljs-comment">// 尝试并发度加一</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> timeout = invoker.getUrl().getMethodParameter(invocation.getMethodName(), TIMEOUT_KEY, <span class="hljs-number">0</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> start = System.currentTimeMillis();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> remain = timeout;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (rpcStatus) { <span class="hljs-comment">// 加锁</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">while</span> (!RpcStatus.beginCount(url, methodName, max)) { <span class="hljs-comment">// 再次尝试并发度加一</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; rpcStatus.wait(remain); <span class="hljs-comment">// 当前线程阻塞，等待并发度降低</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 检测是否超时</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> elapsed = System.currentTimeMillis() - start;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; remain = timeout - elapsed;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (remain &lt;= <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RpcException(...);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 添加一个attribute</span>
&nbsp; &nbsp; invocation.put(ACTIVELIMIT_FILTER_START_TIME, System.currentTimeMillis());
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
}
</code></pre>
<p data-nodeid="141873">从 ActiveLimitFilter.invoke() 方法的代码中可以看到，其核心实现与 RpcStatus 对象密切相关。RpcStatus 中维护了两个集合，分别是：</p>
<ul data-nodeid="141874">
<li data-nodeid="141875">
<p data-nodeid="141876">SERVICE_STATISTICS 集合（ConcurrentMap&lt;String, RpcStatus&gt; 类型），这个集合记录了当前 Consumer 调用每个服务的状态信息，其中 Key 是 URL，Value 是对应的 RpcStatus 对象；</p>
</li>
<li data-nodeid="141877">
<p data-nodeid="141878">METHOD_STATISTICS 集合（ConcurrentMap&lt;String, ConcurrentMap&lt;String, RpcStatus&gt;&gt; 类型），这个集合记录了当前 Consumer 调用每个服务方法的状态信息，其中第一层 Key 是 URL ，第二层 Key 是方法名称，第三层是对应的 RpcStatus 对象。</p>
</li>
</ul>
<p data-nodeid="141879">RpcStatus 中统计了很多调用相关的信息，核心字段有如下几个。</p>
<ul data-nodeid="141880">
<li data-nodeid="141881">
<p data-nodeid="141882">active（AtomicInteger 类型）：当前并发度。这也是 ActiveLimitFilter 中关注的并发度。</p>
</li>
<li data-nodeid="141883">
<p data-nodeid="141884">total（AtomicLong 类型）：调用的总数。</p>
</li>
<li data-nodeid="141885">
<p data-nodeid="141886">failed（AtomicInteger 类型）：失败的调用数。</p>
</li>
<li data-nodeid="141887">
<p data-nodeid="141888">totalElapsed（AtomicLong 类型）：所有调用的总耗时。</p>
</li>
<li data-nodeid="141889">
<p data-nodeid="141890">failedElapsed（AtomicLong 类型）：所有失败调用的总耗时。</p>
</li>
<li data-nodeid="141891">
<p data-nodeid="141892">maxElapsed（AtomicLong 类型）：所有调用中最长的耗时。</p>
</li>
<li data-nodeid="141893">
<p data-nodeid="141894">failedMaxElapsed（AtomicLong 类型）：所有失败调用中最长的耗时。</p>
</li>
<li data-nodeid="141895">
<p data-nodeid="141896">succeededMaxElapsed（AtomicLong 类型）：所有成功调用中最长的耗时。</p>
</li>
</ul>
<p data-nodeid="141897">另外，RpcStatus 提供了上述字段的 getter/setter 方法，用于读写这些字段值，这里不再展开分析。</p>
<p data-nodeid="141898">RpcStatus 中的 beginCount() 方法会在远程调用开始之前执行，其中会从 SERVICE_STATISTICS 集合和 METHOD_STATISTICS 集合中获取服务和服务方法对应的 RpcStatus 对象，然后分别将它们的 active 字段加一，相关实现如下：</p>
<pre class="lang-java" data-nodeid="141899"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">beginCount</span><span class="hljs-params">(URL url, String methodName, <span class="hljs-keyword">int</span> max)</span> </span>{
&nbsp; &nbsp; max = (max &lt;= <span class="hljs-number">0</span>) ? Integer.MAX_VALUE : max;
    <span class="hljs-comment">// 获取服务对应的RpcStatus对象</span>
&nbsp; &nbsp; RpcStatus appStatus = getStatus(url); 
&nbsp; &nbsp; <span class="hljs-comment">// 获取服务方法对应的RpcStatus对象</span>
&nbsp; &nbsp; RpcStatus methodStatus = getStatus(url, methodName);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (methodStatus.active.get() == Integer.MAX_VALUE) { <span class="hljs-comment">// 并发度溢出</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i; ; ) {
&nbsp; &nbsp; &nbsp; &nbsp; i = methodStatus.active.get();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (i + <span class="hljs-number">1</span> &gt; max) { <span class="hljs-comment">// 并发度超过max上限，直接返回false</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (methodStatus.active.compareAndSet(i, i + <span class="hljs-number">1</span>)) { <span class="hljs-comment">// CAS操作</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; <span class="hljs-comment">// 更新成功后退出当前循环</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; appStatus.active.incrementAndGet(); <span class="hljs-comment">// 单个服务的并发度加一</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
}
</code></pre>
<p data-nodeid="141900">ActiveLimitFilter 在继承 Filter 接口的同时，还继承了 Filter.Listener 这个内部接口，在其 onResponse() 方法的实现中，不仅会调用 RpcStatus.endCount() 方法完成调用监控的统计，还会调用 notifyFinish() 方法唤醒阻塞在对应 RpcStatus 对象上的线程，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141901"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onResponse</span><span class="hljs-params">(Result appResponse, Invoker&lt;?&gt; invoker, Invocation invocation)</span> </span>{
&nbsp; &nbsp; String methodName = invocation.getMethodName(); <span class="hljs-comment">// 获取调用的方法名称</span>
&nbsp; &nbsp; URL url = invoker.getUrl();
&nbsp; &nbsp; <span class="hljs-keyword">int</span> max = invoker.getUrl().getMethodParameter(methodName, ACTIVES_KEY, <span class="hljs-number">0</span>);
    <span class="hljs-comment">// 调用 RpcStatus.endCount() 方法完成调用监控的统计</span>
&nbsp; &nbsp; RpcStatus.endCount(url, methodName, getElapsed(invocation), <span class="hljs-keyword">true</span>);
    <span class="hljs-comment">// 调用 notifyFinish() 方法唤醒阻塞在对应 RpcStatus 对象上的线程</span>
&nbsp; &nbsp; notifyFinish(RpcStatus.getStatus(url, methodName), max);
}
</code></pre>
<p data-nodeid="141902">在 RpcStatus.endCount() 方法中，会对服务和服务方法两个维度的 RpcStatus 中的所有字段进行更新，完成统计：</p>
<pre class="lang-java" data-nodeid="141903"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">endCount</span><span class="hljs-params">(RpcStatus status, <span class="hljs-keyword">long</span> elapsed, <span class="hljs-keyword">boolean</span> succeeded)</span> </span>{
&nbsp; &nbsp; status.active.decrementAndGet(); <span class="hljs-comment">// 请求完成，降低并发度</span>
&nbsp; &nbsp; status.total.incrementAndGet(); <span class="hljs-comment">// 调用总次数增加</span>
&nbsp; &nbsp; status.totalElapsed.addAndGet(elapsed); <span class="hljs-comment">// 调用总耗时增加</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (status.maxElapsed.get() &lt; elapsed) { <span class="hljs-comment">// 更新最大耗时</span>
&nbsp; &nbsp; &nbsp; &nbsp; status.maxElapsed.set(elapsed);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (succeeded) { <span class="hljs-comment">// 如果此次调用成功，则会更新成功调用的最大耗时</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (status.succeededMaxElapsed.get() &lt; elapsed) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; status.succeededMaxElapsed.set(elapsed);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 如果此次调用失败，则会更新失败调用的最大耗时</span>
&nbsp; &nbsp; &nbsp; &nbsp; status.failed.incrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; status.failedElapsed.addAndGet(elapsed);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (status.failedMaxElapsed.get() &lt; elapsed) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; status.failedMaxElapsed.set(elapsed);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="141904">ContextFilter</h3>
<p data-nodeid="141905">在前面第 26 课时介绍 AbstractInvoker 的时候，我们提到其 invoke() 方法中有如下一段逻辑：</p>
<pre class="lang-java" data-nodeid="141906"><code data-language="java">Map&lt;String, Object&gt; contextAttachments = 
      RpcContext.getContext().getObjectAttachments();
<span class="hljs-keyword">if</span> (CollectionUtils.isNotEmptyMap(contextAttachments)) {
&nbsp; &nbsp; invocation.addObjectAttachments(contextAttachments);
}
</code></pre>
<p data-nodeid="141907">这里将 RpcContext 中的附加信息添加到 Invocation 中，一并传递到 Provider 端。那在 Provider 端是如何获取 Invocation 中的附加信息，并设置到 RpcContext 中的呢？</p>
<p data-nodeid="141908"><strong data-nodeid="142062">ContextFilter 是 Provider 端的一个 Filter 实现，它主要用来初始化 Provider 端的 RpcContext。</strong> ContextFilter 首先会从 Invocation 中获取 Attachments 集合，并对该集合中的 Key 进行过滤，其中会将 UNLOADING_KEYS 集合中的全部 Key 过滤掉；之后会初始化 RpcContext 以及 Invocation 的各项信息，例如，Invocation、Attachments、localAddress、remoteApplication、超时时间等；最后调用 Invoker.invoke() 方法执行 Provider 的业务逻辑。ContextFilter.Invoke() 方法的具体逻辑如下所示：</p>
<pre class="lang-java" data-nodeid="141909"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; Map&lt;String, Object&gt; attachments = invocation.getObjectAttachments();
&nbsp; &nbsp; ... ... <span class="hljs-comment">// 省略过滤UNLOADING_KEYS集合的逻辑</span>
&nbsp; &nbsp; RpcContext context = RpcContext.getContext(); <span class="hljs-comment">// 获取RpcContext</span>
&nbsp; &nbsp; context.setInvoker(invoker) <span class="hljs-comment">// 设置RpcContext中的信息</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setInvocation(invocation)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setLocalAddress(invoker.getUrl().getHost(), 
                invoker.getUrl().getPort());
&nbsp; &nbsp; String remoteApplication = (String) invocation.getAttachment(REMOTE_APPLICATION_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (StringUtils.isNotEmpty(remoteApplication)) {
&nbsp; &nbsp; &nbsp; &nbsp; context.setRemoteApplicationName(remoteApplication);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; context.setRemoteApplicationName((String) context.getAttachment(REMOTE_APPLICATION_KEY));
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">long</span> timeout = RpcUtils.getTimeout(invocation, -<span class="hljs-number">1</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout != -<span class="hljs-number">1</span>) { <span class="hljs-comment">// 设置超时时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.set(TIME_COUNTDOWN_KEY, TimeoutCountDown.newCountDown(timeout, TimeUnit.MILLISECONDS));
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (attachments != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 向RpcContext中设置Attachments</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (context.getObjectAttachments() != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; context.getObjectAttachments().putAll(attachments);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; context.setObjectAttachments(attachments);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (invocation <span class="hljs-keyword">instanceof</span> RpcInvocation) { <span class="hljs-comment">// 向Invocation设置Invoker</span>
&nbsp; &nbsp; &nbsp; &nbsp; ((RpcInvocation) invocation).setInvoker(invoker);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
        <span class="hljs-comment">// 在整个调用过程中，需要保持当前RpcContext不被删除，这里会将remove开关关掉，这样，removeContext()方法不会删除LOCAL RpcContext了</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.clearAfterEachInvoke(<span class="hljs-keyword">false</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 重置remove开关</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.clearAfterEachInvoke(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//&nbsp;清理RpcContext，当前线程处理下一个调用的时候，会创建新的RpcContext</span>
&nbsp; &nbsp; &nbsp; &nbsp; RpcContext.removeContext(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; RpcContext.removeServerContext();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141910">ContextFilter 继承了 Filter 接口的同时，还继承了 Filter.Listener 这个内部接口。在 ContextFilter.onResponse() 方法中，会将 SERVER_LOCAL 这个 RpcContext 中的附加信息添加到 AppResponse 的 attachments 字段中，返回给 Consumer。</p>
<pre class="lang-java" data-nodeid="141911"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onResponse</span><span class="hljs-params">(Result appResponse, Invoker&lt;?&gt; invoker, Invocation invocation)</span> </span>{
appResponse.addObjectAttachments(RpcContext.getServerContext().getObjectAttachments());
}
</code></pre>
<h3 data-nodeid="141912">AccessLogFilter</h3>
<p data-nodeid="141913">AccessLogFilter 主要用于记录日志，它的主要功能是将 Provider 或者 Consumer 的日志信息写入文件中。AccessLogFilter 会先将日志消息放入内存日志集合中缓存，当缓存大小超过一定阈值之后，会触发日志的写入。若长时间未触发日志文件写入，则由定时任务定时写入。</p>
<p data-nodeid="141914">AccessLogFilter.invoke() 方法的核心实现如下：</p>
<pre class="lang-java" data-nodeid="141915"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation inv)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; String accessLogKey = invoker.getUrl().getParameter(ACCESS_LOG_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ConfigUtils.isNotEmpty(accessLogKey)) { <span class="hljs-comment">// 获取ACCESS_LOG_KEY</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 构造AccessLogData对象，其中记录了日志信息，例如，调用的服务名称、方法名称、version等</span>
&nbsp; &nbsp; &nbsp; &nbsp; AccessLogData logData = buildAccessLogData(invoker, inv);
&nbsp; &nbsp; &nbsp; &nbsp; log(accessLogKey, logData);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 调用下一个Invoker</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(inv);
}
</code></pre>
<p data-nodeid="141916">在 log() 方法中，会按照 ACCESS_LOG_KEY 的值，找到对应的 AccessLogData 集合，然后完成缓存写入；如果缓存大小超过阈值，则触发文件写入。具体实现如下：</p>
<pre class="lang-java" data-nodeid="141917"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">log</span><span class="hljs-params">(String accessLog, AccessLogData accessLogData)</span> </span>{
    <span class="hljs-comment">// 根据ACCESS_LOG_KEY获取对应的缓存集合</span>
&nbsp; &nbsp; Set&lt;AccessLogData&gt; logSet = LOG_ENTRIES.computeIfAbsent(accessLog, k -&gt; <span class="hljs-keyword">new</span> ConcurrentHashSet&lt;&gt;());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (logSet.size() &lt; LOG_MAX_BUFFER) { <span class="hljs-comment">// 缓存大小未超过阈值</span>
&nbsp; &nbsp; &nbsp; &nbsp; logSet.add(accessLogData);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 缓存大小超过阈值，触发缓存数据写入文件</span>
&nbsp; &nbsp; &nbsp; &nbsp; writeLogSetToFile(accessLog, logSet);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 完成文件写入之后，再次写入缓存</span>
&nbsp; &nbsp; &nbsp; &nbsp; logSet.add(accessLogData);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141918">在 writeLogSetToFile() 方法中，会按照 ACCESS_LOG_KEY 的值将日志信息写入不同的日志文件中：</p>
<ul data-nodeid="141919">
<li data-nodeid="141920">
<p data-nodeid="141921">如果 ACCESS_LOG_KEY 配置的值为 true 或 default，会使用 Dubbo 默认提供的统一日志框架，输出到日志文件中；</p>
</li>
<li data-nodeid="141922">
<p data-nodeid="141923">如果 ACCESS_LOG_KEY 配置的值不为 true 或 default，则 ACCESS_LOG_KEY 配置值会被当作 access log 文件的名称，AccessLogFilter 会创建相应的目录和文件，并完成日志的输出。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="141924"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">writeLogSetToFile</span><span class="hljs-params">(String accessLog, Set&lt;AccessLogData&gt; logSet)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ConfigUtils.isDefault(accessLog)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// ACCESS_LOG_KEY配置值为true或是default</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processWithServiceLogger(logSet);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// ACCESS_LOG_KEY配置既不是true也不是default的时候</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; File file = <span class="hljs-keyword">new</span> File(accessLog);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; createIfLogDirAbsent(file); <span class="hljs-comment">// 创建目录</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; renameFile(file); <span class="hljs-comment">// 创建日志文件，这里会以日期为后缀，滚动创建</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 遍历logSet集合，将日志逐条写入文件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processWithAccessKeyLogger(logSet, file);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; &nbsp; &nbsp; logger.error(e.getMessage(), e);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processWithAccessKeyLogger</span><span class="hljs-params">(Set&lt;AccessLogData&gt; logSet, File file)</span> <span class="hljs-keyword">throws</span> IOException </span>{
    <span class="hljs-comment">// 创建FileWriter，写入指定的日志文件</span>
&nbsp; &nbsp; <span class="hljs-keyword">try</span> (FileWriter writer = <span class="hljs-keyword">new</span> FileWriter(file, <span class="hljs-keyword">true</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Iterator&lt;AccessLogData&gt; iterator = logSet.iterator();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;iterator.hasNext();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;iterator.remove()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; writer.write(iterator.next().getLogMessage());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; writer.write(System.getProperty(<span class="hljs-string">"line.separator"</span>));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; writer.flush();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141925">在 AccessLogFilter 的构造方法中，会启动一个定时任务，定时调用上面介绍的 writeLogSetToFile() 方法，定时写入日志，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141926"><code data-language="java"><span class="hljs-comment">// 启动一个线程池</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ScheduledExecutorService LOG_SCHEDULED =
 Executors.newSingleThreadScheduledExecutor(<span class="hljs-keyword">new</span> NamedThreadFactory(<span class="hljs-string">"Dubbo-Access-Log"</span>, <span class="hljs-keyword">true</span>));
<span class="hljs-comment">// 启动一个定时任务，定期执行writeLogSetToFile()方法，完成日志写入</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AccessLogFilter</span><span class="hljs-params">()</span> </span>{
    LOG_SCHEDULED.scheduleWithFixedDelay(
        <span class="hljs-keyword">this</span>::writeLogToFile, LOG_OUTPUT_INTERVAL, 
           LOG_OUTPUT_INTERVAL, TimeUnit.MILLISECONDS);
}
</code></pre>
<p data-nodeid="141927">为便于你更好地理解这部分内容，下面我们再来看一下 Dubbo 对各种日志框架的支持，在 processWithServiceLogger() 方法中我们可以看到 Dubbo 是通过 LoggerFactory 来支持各种第三方日志框架的：</p>
<pre class="lang-java" data-nodeid="141928"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processWithServiceLogger</span><span class="hljs-params">(Set&lt;AccessLogData&gt; logSet)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Iterator&lt;AccessLogData&gt; iterator = logSet.iterator();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;iterator.hasNext();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;iterator.remove()) { <span class="hljs-comment">// 遍历logSet集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; AccessLogData logData = iterator.next();
        <span class="hljs-comment">// 通过LoggerFactory获取Logger对象，并写入日志</span>
&nbsp; &nbsp; &nbsp; &nbsp; LoggerFactory.getLogger(LOG_KEY + <span class="hljs-string">"."</span> + logData.getServiceName()).info(logData.getLogMessage());
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141929">在 LoggerFactory 中维护了一个 LOGGERS 集合（Map&lt;String, FailsafeLogger&gt; 类型），其中维护了当前使用的全部 FailsafeLogger 对象；FailsafeLogger 对象中封装了一个 Logger 对象，这个 Logger 接口是 Dubbo 自己定义的接口，Dubbo 针对每种第三方框架都提供了一个 Logger 接口的实现，如下图所示：</p>
<p data-nodeid="146330"><img src="https://s0.lgstatic.com/i/image/M00/68/F2/Ciqc1F-lL4eAGvorAAEnucS-mWg399.png" alt="Lark20201106-191032.png" data-nodeid="146334"></p>
<div data-nodeid="146331"><p style="text-align:center">Logger 接口的实现</p></div>










<p data-nodeid="141932">FailsafeLogger 是 Logger 对象的装饰器，它在每个 Logger 日志写入操作之外，都添加了 try/catch 异常处理。其他的 Dubbo Logger 实现类则是封装了相应第三方的 Logger 对象，并将日志输出操作委托给第三方的 Logger 对象完成。这里我们以 Log4j2Logger 为例进行简单分析：</p>
<pre class="lang-java" data-nodeid="141933"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Log4j2Logger</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Logger</span> </span>{
    <span class="hljs-comment">// 维护了一个log4j日志框架中的Logger对象，实现了适配器的功能</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> org.apache.logging.log4j.Logger logger;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">Log4j2Logger</span><span class="hljs-params">(org.apache.logging.log4j.Logger logger)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.logger = logger;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">info</span><span class="hljs-params">(String msg, Throwable e)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; logger.info(msg, e); <span class="hljs-comment">// 直接调用log4j日志框架的Logger写入日志</span>
&nbsp; &nbsp; }

    ... <span class="hljs-comment">// 省略info()方法的其他重载，省略error、trace、warn、debug等方法</span>
}
</code></pre>
<p data-nodeid="141934">在 LoggerFactory.getLogger() 方法中，是通过其中的 LOGGER_ADAPTER 字段（LoggerAdapter 类型） 获取 Logger 实现对象的：</p>
<pre class="lang-java" data-nodeid="141935"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Logger <span class="hljs-title">getLogger</span><span class="hljs-params">(String key)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> LOGGERS.computeIfAbsent(key, k -&gt; 
        <span class="hljs-keyword">new</span> FailsafeLogger(LOGGER_ADAPTER.getLogger(k)));
}
</code></pre>
<p data-nodeid="141936">LOGGER_ADAPTER 字段在 LoggerFactory.setLogger() 方法中，通过 SPI 机制初始化：</p>
<pre class="lang-java" data-nodeid="141937"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setLoggerAdapter</span><span class="hljs-params">(String loggerAdapter)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (loggerAdapter != <span class="hljs-keyword">null</span> &amp;&amp; loggerAdapter.length() &gt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; setLoggerAdapter(ExtensionLoader.getExtensionLoader(
           LoggerAdapter.class).getExtension(loggerAdapter));
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141938">LoggerAdapter 被 @SPI 注解修饰，是一个扩展接口，如下图所示，LoggerAdapter 对应每个第三方框架的一个相应实现，用于创建相应的 Dubbo Logger 实现对象。</p>
<p data-nodeid="146963" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/68/FE/CgqCHl-lL4GAWy4JAAFMZJwzrp8801.png" alt="Lark20201106-191036.png" data-nodeid="146967"></p>
<div data-nodeid="146964"><p style="text-align:center">LoggerAdapter 接口实现</p></div>


<p data-nodeid="141941">以 Log4j2LoggerAdapter 为例，其核心在 getLogger() 方法中，主要是创建 Log4j2Logger 对象，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141942"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Log4j2LoggerAdapter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">LoggerAdapter</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Logger <span class="hljs-title">getLogger</span><span class="hljs-params">(String key)</span> </span>{ <span class="hljs-comment">// 创建Log4j2Logger适配器</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Log4j2Logger(LogManager.getLogger(key));
    }
}
</code></pre>
<h3 data-nodeid="141943">ClassLoaderFilter</h3>
<p data-nodeid="141944">ClassLoaderFilter 是 Provider 端的一个 Filter 实现，主要功能是切换类加载器。</p>
<p data-nodeid="141945">在 ClassLoaderFilter.invoke() 方法中，首先获取当前线程关联的 contextClassLoader，然后将其 ContextClassLoader 设置为 invoker.getInterface().getClassLoader()，也就是加载服务接口类的类加载器；之后执行 invoker.invoke() 方法，执行后续的 Filter 逻辑以及业务逻辑；最后，将当前线程关联的 contextClassLoader 重置为原来的 contextClassLoader。ClassLoaderFilter 的核心逻辑如下：</p>
<pre class="lang-java" data-nodeid="141946"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; ClassLoader ocl = Thread.currentThread().getContextClassLoader();
&nbsp; &nbsp; <span class="hljs-comment">// 更新当前线程绑定的ClassLoader Thread.currentThread().setContextClassLoader(invoker.getInterface().getClassLoader());</span>
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; Thread.currentThread().setContextClassLoader(ocl);
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="141947">ExecuteLimitFilter</h3>
<p data-nodeid="141948"><strong data-nodeid="142121">ExecuteLimitFilter 是 Dubbo 在 Provider 端限流的实现</strong>，与 Consumer 端的限流实现 ActiveLimitFilter 相对应。ExecuteLimitFilter 的核心实现与 ActiveLimitFilter类似，也是依赖 RpcStatus 的 beginCount() 方法和 endCount() 方法来实现 RpcStatus.active 字段的增减，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141949"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; URL url = invoker.getUrl();
&nbsp; &nbsp; String methodName = invocation.getMethodName();
&nbsp; &nbsp; <span class="hljs-keyword">int</span> max = url.getMethodParameter(methodName, EXECUTES_KEY, <span class="hljs-number">0</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 尝试增加active的值，当并发度达到executes配置指定的阈值，则直接抛出异常</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!RpcStatus.beginCount(url, methodName, max)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RpcException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; invocation.put(EXECUTE_LIMIT_FILTER_START_TIME, System.currentTimeMillis());
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation); <span class="hljs-comment">// 执行后续Filter以及业务逻辑</span>
}
</code></pre>
<p data-nodeid="141950">ExecuteLimitFilter 同时还实现了 Filter 内部的 Listener 接口，在 onResponse() 方法和 onError() 方法中会调用 RpcStatus.endCount() 方法，减小 active 的值，同时完成对一次调用的统计，具体实现比较简单，这里就不再展示。</p>
<h3 data-nodeid="141951">TimeoutFilter</h3>
<p data-nodeid="141952">在前文介绍 ConsumerContextFilter 的时候可以看到，如果通过 TIME_COUNTDOWN_KEY 在 RpcContext 中配置了 TimeCountDown，就会对 TimeoutCountDown 进行检查，判定此次请求是否超时。然后，在 DubboInvoker 的 doInvoker() 方法实现中可以看到，在发起请求之前会调用 calculateTimeout() 方法确定该请求还有多久过期：</p>
<pre class="lang-java" data-nodeid="141953"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> <span class="hljs-title">calculateTimeout</span><span class="hljs-params">(Invocation invocation, String methodName)</span> </span>{
&nbsp; &nbsp; Object countdown = RpcContext.getContext().get(TIME_COUNTDOWN_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">int</span> timeout = DEFAULT_TIMEOUT;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (countdown == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// RpcContext中没有指定TIME_COUNTDOWN_KEY，则使用timeout配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取timeout配置指定的超时时长，默认值为1秒</span>
&nbsp; &nbsp; &nbsp; &nbsp; timeout = (<span class="hljs-keyword">int</span>) RpcUtils.getTimeout(getUrl(), methodName, RpcContext.getContext(), DEFAULT_TIMEOUT);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (getUrl().getParameter(ENABLE_TIMEOUT_COUNTDOWN_KEY, <span class="hljs-keyword">false</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果开启了ENABLE_TIMEOUT_COUNTDOWN_KEY，则通过TIMEOUT_ATTACHENT_KEY将超时时间传递给Provider端</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; invocation.setObjectAttachment(TIMEOUT_ATTACHENT_KEY, timeout);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 当前RpcContext中已经通过TIME_COUNTDOWN_KEY指定了超时时间，则使用该值作为超时时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; TimeoutCountDown timeoutCountDown = (TimeoutCountDown) countdown;
&nbsp; &nbsp; &nbsp; &nbsp; timeout = (<span class="hljs-keyword">int</span>) timeoutCountDown.timeRemaining(TimeUnit.MILLISECONDS);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将剩余超时时间放入attachment中，传递给Provider端</span>
&nbsp; &nbsp; &nbsp; &nbsp; invocation.setObjectAttachment(TIMEOUT_ATTACHENT_KEY, timeout);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> timeout;
}
</code></pre>
<p data-nodeid="141954">当请求到达 Provider 时，ContextFilter 会根据 Invocation 中的 attachment 恢复 RpcContext 的attachment，其中就包含 TIMEOUT_ATTACHENT_KEY（对应的 Value 会恢复成 TimeoutCountDown 对象）。</p>
<p data-nodeid="141955">TimeoutFilter 是 Provider 端另一个涉及超时时间的 Filter 实现，其 invoke() 方法实现比较简单，直接将请求转发给后续 Filter 处理。在 TimeoutFilter 对 onResponse() 方法的实现中，会从 RpcContext 中读取上述 TimeoutCountDown 对象，并检查此次请求是否超时。如果请求已经超时，则会将 AppResponse 中的结果清空，同时打印一条警告日志，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141956"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onResponse</span><span class="hljs-params">(Result appResponse, Invoker&lt;?&gt; invoker, Invocation invocation)</span> </span>{
&nbsp; &nbsp; Object obj = RpcContext.getContext().get(TIME_COUNTDOWN_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (obj != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; TimeoutCountDown countDown = (TimeoutCountDown) obj;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (countDown.isExpired()) { <span class="hljs-comment">// 检查结果是否超时</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ((AppResponse) appResponse).clear(); <span class="hljs-comment">// 清理结果信息</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (logger.isWarnEnabled()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.warn(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="141957">TpsLimitFilter</h3>
<p data-nodeid="141958"><strong data-nodeid="142140">TpsLimitFilter 是 Provider 端对 TPS 限流的实现</strong>。TpsLimitFilter 中维护了一个 TPSLimiter 接口类型的对象，其默认实现是 DefaultTPSLimiter，由它来控制 Provider 端的 TPS 上限值为多少。TpsLimitFilter.invoke() 方法的具体实现如下所示：</p>
<pre class="lang-java" data-nodeid="141959"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 超过限流之后，直接抛出异常</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!tpsLimiter.isAllowable(invoker.getUrl(), invocation)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RpcException(<span class="hljs-string">"... "</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
}
</code></pre>
<p data-nodeid="141960">TPSLimiter 接口中的核心是 isAllowable() 方法。在 DefaultTPSLimiter 实现中，使用ConcurrentHashMap（stats 字段）为每个 ServiceKey 维护了一个相应的 StatItem 对象；在 isAllowable() 方法实现中，会从 URL 中读取 tps 参数值（默认为 -1，即没有限流），对于需要限流的请求，会从 stats 集合中获取（或创建）相应 StatItem 对象，然后调用 StatItem 对象的isAllowable() 方法判断是否被限流，具体实现如下：</p>
<pre class="lang-java" data-nodeid="141961"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAllowable</span><span class="hljs-params">(URL url, Invocation invocation)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> rate = url.getParameter(TPS_LIMIT_RATE_KEY, -<span class="hljs-number">1</span>);
&nbsp; &nbsp; <span class="hljs-keyword">long</span> interval = url.getParameter(TPS_LIMIT_INTERVAL_KEY, DEFAULT_TPS_LIMIT_INTERVAL);
&nbsp; &nbsp; String serviceKey = url.getServiceKey();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (rate &gt; <span class="hljs-number">0</span>) { <span class="hljs-comment">// 需要限流，尝试从stats集合中获取相应的StatItem对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; StatItem statItem = stats.get(serviceKey);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (statItem == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 查询stats集合失败，则创建新的StatItem对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; stats.putIfAbsent(serviceKey, <span class="hljs-keyword">new</span> StatItem(serviceKey, rate, interval));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; statItem = stats.get(serviceKey);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// URL中参数发生变化时，会重建对应的StatItem</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (statItem.getRate() != rate || statItem.getInterval() != interval) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; stats.put(serviceKey, <span class="hljs-keyword">new</span> StatItem(serviceKey, rate, interval));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; statItem = stats.get(serviceKey);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> statItem.isAllowable();&nbsp;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 不需要限流，则从stats集合中清除相应的StatItem对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; StatItem statItem = stats.get(serviceKey);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (statItem != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; stats.remove(serviceKey);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
}
</code></pre>
<p data-nodeid="141962">在 StatItem 中会记录如下一些关键信息。</p>
<ul data-nodeid="141963">
<li data-nodeid="141964">
<p data-nodeid="141965">name（String 类型）：对应的 ServiceKey。</p>
</li>
<li data-nodeid="141966">
<p data-nodeid="141967">rate（int 类型）：一段时间内能通过的 TPS 上限。</p>
</li>
<li data-nodeid="141968">
<p data-nodeid="141969">token（LongAdder 类型）：初始值为 rate 值，每通过一个请求 token 递减一，当减为 0 时，不再通过任何请求，实现限流的作用。</p>
</li>
<li data-nodeid="141970">
<p data-nodeid="141971">interval（long 类型）：重置 token 值的时间周期，这样就实现了在 interval 时间段内能够通过 rate 个请求的效果。</p>
</li>
</ul>
<p data-nodeid="141972">下面我们来看 StatItem 中 isAllowable() 方法的实现：</p>
<pre class="lang-java" data-nodeid="141973"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAllowable</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> now = System.currentTimeMillis();
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (now &gt; lastResetTime + interval) { <span class="hljs-comment">// 周期性重置token</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; token = buildLongAdder(rate);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; lastResetTime = now; <span class="hljs-comment">// 记录最近一次重置token的时间戳</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (token.sum() &lt; <span class="hljs-number">0</span>) { <span class="hljs-comment">// 请求限流</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; token.decrement(); <span class="hljs-comment">// 请求正常通过</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
</code></pre>
<p data-nodeid="141974">到这里，Dubbo 中提供的核心 Filter 实现就介绍完了。不过，还有 EchoFilter 和 ExceptionFilter 这两个实现没有详细介绍，就留给你自行分析了，相信在了解上述 Filter 实现之后，你就可以非常轻松地阅读这两个 Filter 的源码。</p>
<h3 data-nodeid="141975">自定义 Filter 实践</h3>
<p data-nodeid="141976">在了解完 Dubbo 加载 Filter 的原理以及 Dubbo 提供的多种 Filter 实现之后，下面我们就开始动手实现一个自定义的 Filter 实现，来进一步扩展 Dubbo 的功能。这里我们编写两个自定义的 Filter 实现类—— JarVersionConsumerFilter 和 JarVersionProviderFilter。</p>
<ul data-nodeid="141977">
<li data-nodeid="141978">
<p data-nodeid="141979">JarVersionConsumerFilter 会获取服务接口所在 jar 包的版本，并作为 attachment 随请求发送到 Provider 端。</p>
</li>
<li data-nodeid="141980">
<p data-nodeid="141981">JarVersionProviderFilter 会统计请求中携带的 jar 包版本，并周期性打印（实践中一般会和监控数据一起生成报表）。</p>
</li>
</ul>
<p data-nodeid="141982">在实践中，我们可以通过这两个 Filter 实现，搞清楚当前所有 Consumer 端升级接口 jar 包的情况。</p>
<p data-nodeid="141983">首先，我们来看 JarVersionConsumerFilter 实现中的几个关键点。</p>
<ul data-nodeid="141984">
<li data-nodeid="141985">
<p data-nodeid="141986">JarVersionConsumerFilter 被 @Activate 注解修饰，其中的 group 字段值为 CommonConstants.CONSUMER，会在 Consumer 端自动激活，order 字段值为 -1 ，是最后执行的 Filter。</p>
</li>
<li data-nodeid="141987">
<p data-nodeid="141988">JarVersionConsumerFilter 中维护了一个 LoadingCache 用于缓存各个业务接口与对应 jar 包版本号之间的映射关系。</p>
</li>
<li data-nodeid="141989">
<p data-nodeid="141990">在 invoke() 方法的实现中，会通过 LoadingCache 查询接口所在 jar 包的版本号，然后记录到 Invocation 的 attachment 之中，发送到 Provider 端。</p>
</li>
</ul>
<p data-nodeid="141991">下面是 JarVersionConsumerFilter 的具体实现：</p>
<pre class="lang-java" data-nodeid="141992"><code data-language="java"><span class="hljs-meta">@Activate(group = {CommonConstants.CONSUMER}, order = -1)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JarVersionConsumerFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span> </span>{

&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String JAR_VERSION_NAME_KEY = <span class="hljs-string">"dubbo.jar.version"</span>;
    <span class="hljs-comment">// 通过一个LoadingCache缓存各个Class所在的jar包版本</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> LoadingCache&lt;Class&lt;?&gt;, String&gt; versionCache = CacheBuilder.newBuilder()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .maximumSize(<span class="hljs-number">1024</span>).build(<span class="hljs-keyword">new</span> CacheLoader&lt;Class&lt;?&gt;, String&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">load</span><span class="hljs-params">(Class&lt;?&gt; key)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getJarVersion(key);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });

&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, String&gt; attachments = invocation.getAttachments();
&nbsp; &nbsp; &nbsp; &nbsp; String version = versionCache.getUnchecked(invoker.getInterface());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!StringUtils.isBlank(version)) { <span class="hljs-comment">// 添加版本号</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; attachments.put(JAR_VERSION_NAME_KEY, version);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 读取Classpath下的"/META-INF/MANIFEST.MF"文件，获取jar包版本</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">getJarVersion</span><span class="hljs-params">(Class clazz)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> (BufferedReader reader = <span class="hljs-keyword">new</span> BufferedReader(<span class="hljs-keyword">new</span> InputStreamReader(clazz.getResourceAsStream(<span class="hljs-string">"/META-INF/MANIFEST.MF"</span>)))) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String s = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">while</span> ((s = reader.readLine()) != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> i = s.indexOf(<span class="hljs-string">"Implementation-Version:"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (i &gt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> s.substring(i);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (IOException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略异常处理逻辑</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">""</span>;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141993">JarVersionProviderFilter 的实现就非常简单了，它会读取请求中的版本信息，并将关联的计数器加一。另外，JarVersionProviderFilter 的构造方法中会启动一个定时任务，每隔一分钟执行一次，将统计结果打印到日志中（在生产环境一般会将这些统计数据生成报表展示）。</p>
<p data-nodeid="141994">JarVersionProviderFilter 既然要运行在 Provider 端，那就需要将其 @Activate 注解的 group 字段设置为 CommonConstants.PROVIDER 常量。JarVersionProviderFilter 的具体实现如下：</p>
<pre class="lang-java" data-nodeid="141995"><code data-language="java"><span class="hljs-meta">@Activate(group = {CommonConstants.PROVIDER}, order = -1)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JarVersionProviderFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Filter</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String JAR_VERSION_NAME_KEY = <span class="hljs-string">"dubbo.jar.version"</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Map&lt;String, AtomicLong&gt; versionState = <span class="hljs-keyword">new</span> ConcurrentHashMap&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ScheduledExecutorService SCHEDULED_EXECUTOR_SERVICE = Executors.newScheduledThreadPool(<span class="hljs-number">1</span>);
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">JarVersionProviderFilter</span><span class="hljs-params">()</span> </span>{ <span class="hljs-comment">// 启动定时任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; SCHEDULED_EXECUTOR_SERVICE.schedule(() -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;String, AtomicLong&gt; entry : versionState.entrySet()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(entry.getKey() + <span class="hljs-string">":"</span> + entry.getValue().getAndSet(<span class="hljs-number">0</span>)); <span class="hljs-comment">// 打印日志并将统计数据重置</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }, <span class="hljs-number">1</span>, TimeUnit.MINUTES);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Result <span class="hljs-title">invoke</span><span class="hljs-params">(Invoker&lt;?&gt; invoker, Invocation invocation)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; String versionAttachment = invocation.getAttachment(JAR_VERSION_NAME_KEY);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!StringUtils.isBlank(versionAttachment)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AtomicLong count = versionState.computeIfAbsent(versionAttachment, v -&gt; <span class="hljs-keyword">new</span> AtomicLong(<span class="hljs-number">0L</span>));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; count.getAndIncrement(); <span class="hljs-comment">// 递增该版本的统计值</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker.invoke(invocation);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="141996">最后，我们需要在 Provider 项目的 /resources/META-INF/dubbo 目录下添加一个 SPI 配置文件，文件名称为 org.apache.dubbo.rpc.Filter，具体内容如下：</p>
<pre class="lang-java" data-nodeid="141997"><code data-language="java">version-provider=org.apache.dubbo.demo.provider.JarVersionProviderFilter
</code></pre>
<p data-nodeid="141998">同样，也需要在 Consumer 项目相同位置添加相同的 SPI 配置文件（文件名称也相同），具体内容如下：</p>
<pre class="lang-java" data-nodeid="141999"><code data-language="java">version-consumer=org.apache.dubbo.demo.consumer.JarVersionConsumerFilter
</code></pre>
<h3 data-nodeid="142000">总结</h3>
<p data-nodeid="142001">本课时重点介绍了 Dubbo 中 Filter 接口的相关实现。首先，我们回顾了 Filter 链的加载流程实现；然后详细分析了 Dubbo 中多个内置的 Filter 实现，这些内置 Filter 对于实现 Dubbo 核心功能是不可或缺的；最后，我们还阐述了自定义 Filter 扩展 Dubbo 功能的流程，并通过一个统计 jar 包版本的示例进行说明。</p>
<p data-nodeid="142002" class="">在下一课时，我们将开始介绍 Dubbo 中 Cluster 层的内容。</p>

---

### 精选评论

##### **8001：
> 学习了

