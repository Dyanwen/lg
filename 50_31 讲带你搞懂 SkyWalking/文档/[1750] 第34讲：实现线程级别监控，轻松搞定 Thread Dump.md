<p data-nodeid="34727" class="">本课时我们来学习 Thread Dump 功能。</p>


<h3 data-nodeid="37658" class="">背景</h3>





<p data-nodeid="33845">通过前面课时的介绍我们知道，SkyWalking 提供的 Agent 可以收集服务的 Metrics、Trace、Log 等维度的数据，然后发送到后端的 OAP 进行分析并进行持久化存储，我们可以使用 SkyWalking Rocketbot UI（或是直接使用 GraphQL）​ 从不同的维度查询上述数据，评估系统的各项性能和某些具体行为。</p>
<p data-nodeid="33846">例如，我们可以通过 ServiceRespTimeMetrics、ServiceP99Metrics、ServiceCpmMetrics 等 Metrics 了解一个服务的整体吞吐量；可以通过 Trace 信息了解某个具体请求经过的核心组件和服务，以及在这些组件和服务上的耗时情况；可以通过 Trace 上携带的 Log 信息了解相应的异常信息；还可以根据 Trace 信息分析得到 Relation 信息，画出整个服务架构的拓扑图，了解各个服务之间的调用关系以及拓扑图每条调用边上的响应时间、SLA 等信息。这就可以帮助开发和运维人员更好地管理整个服务集群，更快地定位系统的热点和瓶颈，降低运维和问题定位的成本。</p>
<p data-nodeid="38812">SkyWalking 已经满足了我们日常监控和运维的绝大多数需求，但是并没有覆盖到所有运维场景。假设我们发现请求在某个服务中的耗时特别长，远远超过了预期，例如开篇示例中的 demo-webapp ，如下图所示，在 HelloWorldController 在开始调用 Dubbo 服务的前后，会有耗时超过 1s 以上情况：</p>
<p data-nodeid="38813" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/07/CgqCHl8NcY-AR8DqAAE4jw3ZU4w560.png" alt="耗时高Trace截图.png" data-nodeid="38817"></p>


<p data-nodeid="33849">此时，SkyWalking 的 Trace 信息只能提示我们 HelloWorldController.hello() 方法中有一些耗时的逻辑，但是耗时的具体原因是什么无法准确地说明。实际的业务逻辑比较复杂，请求处理耗时高的原因也可能千奇百怪，例如（可能但不限于）：</p>
<ul data-nodeid="33850">
<li data-nodeid="33851">
<p data-nodeid="33852">多个线程并发竞争同一把锁；</p>
</li>
<li data-nodeid="33853">
<p data-nodeid="33854">读写文件，线程等待 I/O 操作；</p>
</li>
<li data-nodeid="33855">
<p data-nodeid="33856">代码逻辑本身的性能有问题，时间复杂度太高。</p>
</li>
</ul>
<p data-nodeid="33857">如果通过 Trace 以及 Metrics 不能明确定位高耗时的问题，我们使用 jstack 工具将线程的栈信息 dump 下来，然后分析线程在哪一个调用中耗时较长。在现实场景中，往往一次 dump 的信息是不足以确认问题的，为了反映线程状态的动态变化，需要连续多次做 Thread Dump，每次间隔根据具体的场景决定，建议至少产生三次以上的 Thread Dump 信息，如果每次 Thread Dump 都指向同一个问题，一般就能够确定具体的问题。</p>
<p data-nodeid="33858">在实际的微服务场景中进行 Thread Dump 时，你可能会遇到几个问题：</p>
<ul data-nodeid="33859">
<li data-nodeid="33860">
<p data-nodeid="33861">如果多个服务都有耗时高的情况，就需要我们去多个服务的机器上进行 Thread Dump，比较麻烦，而且也很难确定不同服务的 Thread Dump 信息是否存在关联。</p>
</li>
<li data-nodeid="33862">
<p data-nodeid="33863">请求一般会经过多个服务端处理，每个服务又是单独的一个集群。如果是某些特殊参数的请求触发了高耗时，我们很难手动捕捉到该请求走到了服务的那个实例上，这台机器上去进行 Thread Dump 就比较困难。</p>
</li>
<li data-nodeid="33864">
<p data-nodeid="33865">如果要求某些服务的响应时延非常低的情况下，虽然服务的延迟高了，但是相对人来说的时间是非常短的，而我们手动 Thread Dump 的速度和次数都是有限的，可能错过问题所在的逻辑，导致问题定位错误。</p>
</li>
</ul>
<h3 data-nodeid="41717" class="">Thread Dump 需求</h3>





<p data-nodeid="44647">为了解决在上述场景下手动 Thread Dump 带来的问题，本课时将为 SkyWalking 添加 Thread  Dump 功能。下面先说明一下 Thread Dump 的需求：一般场景中，用户会通过一个外网的入口请求我们的接入层（例如机房的 Nginx 集群），然后接入层会进行负载均衡，将请求发送到后端的 API 服务集群进行处理（例如 Tomcat 集群），API 服务会根据业务需求调用后端的 RPC 服务（例如 Dubbo、gRPC 等），在 RPC 服务中会调用 Service 层、DAO 层等完成存储的读写或是再次调用其他 RPC 服务。单个请求的路径如下图所示：</p>
<p data-nodeid="46128" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/07/CgqCHl8NcbiATQCtAAFX5iKjEXQ354.png" alt="image (16).png" data-nodeid="46135"></p>









<p data-nodeid="33869">为了实现自动 Thread Dump 功能，我们会在入口处为 Http 请求追加一个 Http Header（Key 为 ENABLE_DUMP_FLAG，Value 为“true”），作为是否进行 Thread Dump 的标识。如果请求带有该标识，线程在处理该请求时每隔一段时间（例如 300ms）会被 dump 一次，这些 dump 下来的信息会记录到请求的 Trace 中，一并发送给 SkyWalking OAP 进行持久化存储。在后续通过 query-graphql-plugin 插件查询某条 Trace 信息的时候，可以将这些 dump 信息一起查询出来，在 SkyWalking Rocketbot UI 进行展示时，可以根据 Thread Dump 的时间将其显示在相应的 Span 处，当然，也可以在 OAP 接收到 Trace 数据时对其中的 Thread Dump 信息进出分析并完成与 Span 的关联。</p>
<p data-nodeid="33870">首先要了解，在 Java 代码中使用 ThreadMXBean 即可完成全部线程的 Thread Dump ，下面是一段简单的示例代码：</p>
<pre class="lang-java" data-nodeid="48176"><code data-language="java">ThreadMXBean bean = ManagementFactory.getThreadMXBean();
ThreadInfo[] threadInfos = bean.dumpAllThreads(<span class="hljs-keyword">true</span>, <span class="hljs-keyword">true</span>);
<span class="hljs-keyword">for</span> (ThreadInfo threadInfo : threadInfos) {
    System.out.println(threadInfo);
}
<span class="hljs-comment">// 部分输出如所示，我们可以看到每个线程的状态信息以及具体的调用栈：</span>
<span class="hljs-string">"Reference Handler"</span> Id=<span class="hljs-number">2</span> WAITING on java.lang.ref.Reference$Lock@<span class="hljs-number">1517365</span>b
	at java.lang.Object.wait(Native Method)
	-&nbsp; waiting on java.lang.ref.Reference$Lock@<span class="hljs-number">1517365</span>b
	at java.lang.Object.wait(Object.java:<span class="hljs-number">502</span>)
	at java.lang.ref.Reference.tryHandlePending(Reference.java:<span class="hljs-number">191</span>)
	at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:<span class="hljs-number">153</span>)


<span class="hljs-string">"main"</span> Id=<span class="hljs-number">1</span> RUNNABLE
	at sun.management.ThreadImpl.dumpThreads0(Native Method)
	at sun.management.ThreadImpl.dumpAllThreads(ThreadImpl.java:<span class="hljs-number">454</span>)
	at com.xxx.sw.Main.main(Main.java:<span class="hljs-number">14</span>)
</code></pre>




<h3 data-nodeid="51086" class="">Thread Dump 功能实现</h3>





<p data-nodeid="52232">从 demo-webapp 这个 API 服务处理“/hello/xxx”接口请求的 Trace 中可以看到，请求首先到达了 Spring Boot 内嵌的 Tomcat 容器，然后走到 Spring Container 中调用 HelloWorldController.hello() 方法 ，最后调用 demo-provider 这个 Dubbo 服务。下图展示了请求的全过程、涉及插件以及插件做的事情：</p>
<p data-nodeid="52233" class=""><img src="https://s0.lgstatic.com/i/image/M00/32/07/CgqCHl8NcdWAM9Y7AAHmR7mU2eY385.png" alt="image (17).png" data-nodeid="52241"></p>


<p data-nodeid="33875">请求的第一站是 Tomcat ，我们可以在 tomcat-7.x-8.x-plugin 插件创建 TracingContext 之前将请求 Header 中携带的 ENABLE_DUMP_FLAG 标记提取出来，并记录到 Trace 的 RuntimeContext 上下文中，这样就可以让 ENABLE_DUMP_FLAG 标记随 Trace 在当前线程继续传播了，实现如下：</p>
<pre class="lang-java" data-nodeid="54581"><code data-language="java"><span class="hljs-comment">// ENABLE_DUMP_FLAG 标记在Http Header 和 RuntimeContext中使用相应的Key</span>
<span class="hljs-comment">// 可以将"ENABLE_DUMP_FLAG"字符串抽到 Constants作为常量，后续可以重复使用</span>
ContextManager.getRuntimeContext().put(<span class="hljs-string">"ENABLE_DUMP_FLAG"</span>,
   request.getHeader(<span class="hljs-string">"ENABLE_DUMP_FLAG"</span>)
);
<span class="hljs-comment">// 对 ContextCarrier 的处理后面会介绍</span>
</code></pre>




<p data-nodeid="33877">前面在介绍 SkyWalking Agent 的时候提到，在 TracingContext 关闭的时候会回调全部 TracingContextListener 监听器，其中就包括 TraceSegmentServiceClient，它会将该 Trace 发送到后端的 OAP 服务，这是典型的观察者模式的应用。我们可以参考这种实现，定义一个 TracingContextPostConstructListener 接口来处理 TracingContext 创建的事件，如下所示：</p>
<pre class="lang-java" data-nodeid="56921"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">TracingContextPostConstructListener</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">postConstruct</span><span class="hljs-params">(TracingContext tracingContext)</span></span>;
}
</code></pre>




<p data-nodeid="33879">在 TracingContext 中新增 postConstruct() 方法回调全部 TracingContextPostConstructListener 实现，并在 TracingContext 构造方法最后调用该方法，其具体剩下如下：</p>
<pre class="lang-java" data-nodeid="59261"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">postConstruct</span><span class="hljs-params">()</span> </span>{ 
    TracingContext.ListenerManager.notifyPostConstruct(<span class="hljs-keyword">this</span>);
}
</code></pre>




<p data-nodeid="61622">在 TracingContext.ListenerManager 中会新增 POST_CONSTRUCT_LISTENERS 字段（List <code data-backticks="1" data-nodeid="61629">&lt;TracingContextPostConstructListener&gt;</code> 类型）来记录当前全部的 TracingContextPostConstructListener 对象，并提供相应 add()、addFirst() 、remove() 等方法，这里的 notifyPostConstruct() 方法会回调全部的 TracingContextPostConstructListener 对象，通知它们该 TracingContext 对象构造完毕。这与 TracingContext.ListenerManager 处理 TracingContextListener 的方式一模一样，具体实现不再展开。</p>
<p data-nodeid="61623">下面我们需要提供了一个 TracingContextPostConstructListener 接口的实现 ——  ThreadDumpManager，它同时实现了 BootService、TracingContextListener、TracingContextPostConstructListener 三个接口，如下图所示，下面将详细分析该实现针对每个接口的实现逻辑：</p>



<p data-nodeid="59846" class=""><img src="https://s0.lgstatic.com/i/image/M00/31/FC/Ciqc1F8NcZ6AU8e4AABDtSvDmRA656.png" alt="ThreadDumpManager继承关系图.png" data-nodeid="59849"></p>

<p data-nodeid="33883">首先在 onComplete() 方法（对 BootService 接口的实现）中会启动一个单独的线程执行一个定时任务，该定时任务主要做两件事：</p>
<ul data-nodeid="33884">
<li data-nodeid="33885">
<p data-nodeid="33886">定时通过 ThreadMXBean 获取线程的 dump 信息。</p>
</li>
<li data-nodeid="33887">
<p data-nodeid="33888">查找到处理 ENABLE_DUMP_FLAG 标记请求的线程，并将该线程的 dump 信息与 Trace 关联起来。</p>
</li>
</ul>
<p data-nodeid="33889">具体实现如下：</p>
<pre class="lang-java" data-nodeid="63971"><code data-language="java"><span class="hljs-comment">// 其中 Key处理标记请求的线程 ID，Value是线程的 dump 信息，该线程在处理标记请求</span>
<span class="hljs-comment">// 的过程中，可能会被 dump 多次，所以 Value 是 List&lt;ThreadDump&gt;集合</span>
<span class="hljs-keyword">private</span> Map&lt;Long, List&lt;ThreadDump&gt;&gt; dumpStore = 
    Maps.newConcurrentMap(); 
<span class="hljs-keyword">private</span> ScheduledExecutorService scheduledExecutorService;

<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onComplete</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
    <span class="hljs-comment">// 创建并启动后台线程</span>
&nbsp; &nbsp; scheduledExecutorService = Executors
            .newSingleThreadScheduledExecutor();
&nbsp; &nbsp; scheduledExecutorService.scheduleAtFixedRate(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> RunnableWithExceptionProtection(<span class="hljs-keyword">this</span>::doThreadDump,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; t -&gt; logger.error(<span class="hljs-string">"thread dump error."</span>, t)), 
             dumpPeriod, dumpPeriod, TimeUnit.MILLISECONDS);
    <span class="hljs-comment">// 将 ThreadDumpManager作为 TracingContextListener接口实现进行注册</span>
&nbsp; &nbsp; TracingContext.ListenerManager.addFirst(
            (TracingContextListener) <span class="hljs-keyword">this</span>);
    <span class="hljs-comment">// 将 ThreadDumpManager作为 TracingContextPostConstructListener</span>
    <span class="hljs-comment">// 接口实现进行注册</span>
&nbsp; &nbsp; TracingContext.ListenerManager.addFirst(
            (TracingContextPostConstructListener) <span class="hljs-keyword">this</span>);
}
</code></pre>




<p data-nodeid="33891">ThreadDumpManager.doThreadDump() 方法是后台线程的核心，它会请求 ThreadMXBean 获取线程的 dump 信息并记录到 dumpStore 集合中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="66311"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doThreadDump</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">long</span> dumpTimestamp = System.currentTimeMillis();
&nbsp; &nbsp; ThreadMXBean bean = ManagementFactory.getThreadMXBean();
&nbsp; &nbsp; <span class="hljs-comment">// 获取全部线程的 dump信息</span>
&nbsp; &nbsp; ThreadInfo[] threadInfos = bean.dumpAllThreads(<span class="hljs-keyword">true</span>, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (ThreadInfo threadInfo : threadInfos) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> threadId = threadInfo.getThreadId();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据监控的线程ID，将相应的 dump信息记录到 dumpStore集合中</span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ThreadDump&gt; threadDumps = <span class="hljs-keyword">this</span>.dumpStore.get(threadId);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadDumps != <span class="hljs-keyword">null</span>) {
            <span class="hljs-comment">// 创建 ThreadDump来记录线程 dump信息</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ThreadDump threadDump = ThreadDump.newBuilder()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setDumpTime(dumpTimestamp)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setThreadInfo(threadInfo.toString()).build();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; threadDumps.add(threadDump);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>




<p data-nodeid="33893">这里使用到了一个新类型 —— ThreadDump ，然后将其定义添加到 trace.proto 文件中，并在 SegmentObject 中添加一个 threadDumps 字段，如下所示：</p>
<pre class="lang-java" data-nodeid="68651"><code data-language="java">message SegmentObject {
    UniqueId traceSegmentId = <span class="hljs-number">1</span>;
    repeated SpanObjectV2 spans = <span class="hljs-number">2</span>;
    ... ...<span class="hljs-comment">// 省略3~5的字段</span>
    repeated ThreadDump threadDumps = <span class="hljs-number">6</span>;
}

message ThreadDump {
    int64 dumpTime = <span class="hljs-number">1</span>;
    string threadInfo = <span class="hljs-number">2</span>;
}
</code></pre>




<p data-nodeid="69236">ThreadDump 主要用于记录一条 Thread Dump 数据以及 dump 操作的时间戳，后续会在 ThreadDumpManager 中将其添加到关联的 SegmentObject 对象中。</p>
<p data-nodeid="69237">回到 ThreadDumpManager，作为 TracingContextPostConstructListener 接口的实现，在其 postConstruct() 方法中会从 TracingContext 关联的 RuntimeContext 中获取 ENABLE_DUMP_FLAG 标记，如果标记为 true，则将当前线程 ID 添加到 dumpStore 集合中存储，具体实现如下：</p>

<pre class="lang-java" data-nodeid="71583"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">postConstruct</span><span class="hljs-params">(TracingContext tracingContext)</span> </span>{
    RuntimeContext runtimeContext= ContextManager.getRuntimeContext();
    Object enableDumpFlag = 
           runtimeContext.get(Constants.ENABLE_DUMP_FLAG);
    <span class="hljs-keyword">if</span> (Constants.ENABLE_DUMP_VAULES.equals(
               enableDumpFlag.toString().toLowerCase())) {
       <span class="hljs-comment">// 将当前线程的ID添加到 dumpStore集合中</span>
       dumpStore.put(Thread.currentThread().getId(), 
            Lists.newLinkedList());
       <span class="hljs-comment">// 在TracingContext中也添加了显影的</span>
       tracingContext.setEnableDumpFlag(Constants.ENABLE_DUMP_VAULES);
    }
}
</code></pre>




<p data-nodeid="33897">ThreadDumpManager 作为 TracingContextListener 的实现，其 afterFinished() 方法实现会在 TracingContext 关闭之后，立即关联相应的 Thread Dump 信息，具体实现如下：</p>
<pre class="lang-java" data-nodeid="73923"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">afterFinished</span><span class="hljs-params">(TraceSegment traceSegment)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">long</span> threadId = Thread.currentThread().getId();
&nbsp; &nbsp; List&lt;ThreadDump&gt; threadDumps = dumpStore.get(threadId);
&nbsp; &nbsp; traceSegment.setThreadDumps(threadDumps);
&nbsp; &nbsp; removeThread(threadId);
}
</code></pre>




<p data-nodeid="33899">注意，ThreadDumpManager 作为 TracingContextListener 需要先于 TraceSegmentServiceClient 这个监听执行，否则是在 Trace 数据发送出去之后再进行关联，后端 OAP 感知不到 ThreadDump 信息。</p>
<h3 data-nodeid="74508" class="">跨进程/跨线程传播</h3>

<p data-nodeid="33901">在上一课时，我们重点处理了入口 Http 服务携带的 ENABLE_DUMP_FLAG 标记。在微服务架构中，如果入口的 Http 请求带了 ENABLE_DUMP_FLAG 标记，后续跨进程的 RPC 调用也是需要传递该标记的。这里将改造 ContextCarrier 以及 TracingContext 的相关方法，实现传播 ENABLE_DUMP_FLAG 标记的功能。</p>
<p data-nodeid="33902">首先需要修改一下 ContextCarrier 序列化之后的字符串结构，SkyWalking 原始的 ContextCarrier 持久化后的字符串包括下面 9 个部分，且相互之间通过字符串“-”连接起来：</p>
<ol data-nodeid="33903">
<li data-nodeid="33904">
<p data-nodeid="33905">固定字符串“1”</p>
</li>
<li data-nodeid="33906">
<p data-nodeid="33907">TraceId</p>
</li>
<li data-nodeid="33908">
<p data-nodeid="33909">TraceSegmentId</p>
</li>
<li data-nodeid="33910">
<p data-nodeid="33911">SpanId</p>
</li>
<li data-nodeid="33912">
<p data-nodeid="33913">ParentServiceInstanceId</p>
</li>
<li data-nodeid="33914">
<p data-nodeid="33915">EntryServiceInstanceId</p>
</li>
<li data-nodeid="33916">
<p data-nodeid="33917">PeerHost</p>
</li>
<li data-nodeid="33918">
<p data-nodeid="33919">EntryEndpointName</p>
</li>
<li data-nodeid="33920">
<p data-nodeid="33921">ParentEndpointName</p>
</li>
</ol>
<p data-nodeid="33922">这里需要添加一个新的部分用于记录当前线程的 ENABLE_DUMP_FLAG 标记。在 serialize() 方法中针对 V2 版本 ContextCarrier 持久化逻辑修改如下：</p>
<pre class="lang-java" data-nodeid="76849"><code data-language="java"><span class="hljs-function">String <span class="hljs-title">serialize</span><span class="hljs-params">(HeaderVersion version)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> StringUtil.join(<span class="hljs-string">'-'</span>, <span class="hljs-string">"1"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; Base64.encode(<span class="hljs-keyword">this</span>.getPrimaryDistributedTraceId().encode()),
&nbsp; &nbsp; &nbsp; &nbsp; Base64.encode(<span class="hljs-keyword">this</span>.getTraceSegmentId().encode()),
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.getSpanId() + <span class="hljs-string">""</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.getParentServiceInstanceId() + <span class="hljs-string">""</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.getEntryServiceInstanceId() + <span class="hljs-string">""</span>,
&nbsp; &nbsp; &nbsp; &nbsp; Base64.encode(<span class="hljs-keyword">this</span>.getPeerHost()),
&nbsp; &nbsp; &nbsp; &nbsp; Base64.encode(<span class="hljs-keyword">this</span>.getEntryEndpointName()),
&nbsp; &nbsp; &nbsp; &nbsp; Base64.encode(<span class="hljs-keyword">this</span>.getParentEndpointName()),
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.enableDumpFlag); <span class="hljs-comment">// 新增 enableDumpFlag 部分</span>
}
</code></pre>




<p data-nodeid="79774">enableDumpFlag 是 ContextCarrier 中新增的一个字段，用于记录当前线程的标记信息。</p>
<p data-nodeid="79775">接下来看 deserialize() 方法，其中兼容了原始 ContextCarrier 字符串以及上述改造后的 ContextCarrier 字符串，如下所示：</p>

<pre class="lang-java" data-nodeid="79189"><code data-language="java"><span class="hljs-function">ContextCarrier <span class="hljs-title">deserialize</span><span class="hljs-params">(String text, HeaderVersion version)</span> </span>{
&nbsp; &nbsp; String[] parts = text.split(<span class="hljs-string">"\\-"</span>, <span class="hljs-number">10</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (parts.length == <span class="hljs-number">9</span> || parts.length == <span class="hljs-number">10</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// parts[0] is sample flag, always trace if header exists.</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.primaryDistributedTraceId = 
            <span class="hljs-keyword">new</span> PropagatedTraceId(Base64.decode2UTFString(parts[<span class="hljs-number">1</span>]));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.traceSegmentId = 
            <span class="hljs-keyword">new</span> ID(Base64.decode2UTFString(parts[<span class="hljs-number">2</span>]));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.spanId = Integer.parseInt(parts[<span class="hljs-number">3</span>]);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.parentServiceInstanceId = Integer.parseInt(parts[<span class="hljs-number">4</span>]);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.entryServiceInstanceId = Integer.parseInt(parts[<span class="hljs-number">5</span>]);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.peerHost = Base64.decode2UTFString(parts[<span class="hljs-number">6</span>]);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.entryEndpointName = Base64.decode2UTFString(parts[<span class="hljs-number">7</span>]);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.parentEndpointName = Base64.decode2UTFString(parts[<span class="hljs-number">8</span>]);
        <span class="hljs-keyword">if</span> (parts.length == <span class="hljs-number">10</span>) {
            <span class="hljs-keyword">this</span>.enableDumpFlag = parts[<span class="hljs-number">9</span>];
        }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
}
</code></pre>




<p data-nodeid="33926">最后，在 TracingContext.inject() 方法填充 ContextCarrier 对象的时候，需要同时填充 enableDumpFlag 字段，如下所示：</p>
<pre class="lang-java" data-nodeid="82117"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">inject</span><span class="hljs-params">(ContextCarrier carrier)</span> </span>{
&nbsp; &nbsp; ... ...<span class="hljs-comment">// 省略前面的原始代码</span>
&nbsp; &nbsp; Object enableDumpFlag = ContextManager.getRuntimeContext()
          .get(Constants.ENABLE_DUMP_FLAG);
&nbsp; &nbsp; carrier.setEnableDumpFlag(enableDumpFlag == <span class="hljs-keyword">null</span> ? <span class="hljs-string">""</span> : 
       enableDumpFlag.toString());
}
</code></pre>




<p data-nodeid="82702">我们可以在 TracingContext.extract() 方法中检测 ContextCarrier 对象携带的 ENABLE_DUMP_FLAG 标记值，之后通过前文介绍的 ThreadDumpManager 记录线程 ID 并由后台线程进行 dump。这种实现方式会与前文添加的 TracingContextPostConstructListener 接口的目的冲突。</p>
<p data-nodeid="82703">另一种实现方式是在各个 agent-plugin 的入口（创建 TracingContext 之前）处理 ContextCarrier 对象携带的 ENABLE_DUMP_FLAG 标记，并设置到 RuntimeContext 中。例如通过 Dubbo 实现的 RPC 调用，我们可以对 apm-dubbo-plugin 插件中的 DubboInterceptor 进行如下修改：</p>

<pre class="lang-java" data-nodeid="85053"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">beforeMethod</span><span class="hljs-params">(...)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isConsumer) {
&nbsp; &nbsp; &nbsp; &nbsp; ... ... <span class="hljs-comment">// 省略Consumer创建 ExitSpan以及ContextCarrier的逻辑</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; ... ... <span class="hljs-comment">// 从 RpcContext中获取 ContextCarrier字符串并反序列化(略)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将ENABLE_DUMP_FLAG标记记录到 RuntimeContext中</span>
        ContextManager.getRuntimeContext().put(Constants
            .ENABLE_DUMP_FLAG,contextCarrier.getEnableDumpFlag());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建 TracingContext，其中会触发</span>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;TracingContextPostConstructListener，从而记录需要dump的线程</span>
&nbsp; &nbsp; &nbsp; &nbsp; span = ContextManager.createEntrySpan(generateOperationName(
            requestURL, invocation), contextCarrier);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 省略后续设置 Tag、Component以及SpanLayer的相关代码</span>
}
</code></pre>




<p data-nodeid="33930">最后，在跨线程调用的时候，TracingContext 信息会通过其 capture() 方法生成的 ContextSnapshot 对象传递，在接收调用的线程中会通过 TracingContext.continued() 方法从 ContextSnapshot 中还原数据。这里需要在 ContextSnapshot 中添加相应字段并改造 capture() 方法以及 continued() 方法，具体逻辑与跨进程调用类似，这里就不再重复，留给你动手实践。</p>
<h3 data-nodeid="85638" class="">OAP 改造</h3>

<p data-nodeid="33932">完成 apm-agent-core 以及 tomcat-7.x-8.x-plugin、apm-dubbo-plugin 插件的改造之后，带有 Thread Dump 的 Trace 可以重构传递到后端 OAP 服务。</p>
<p data-nodeid="33933">通过前文对 trace-receiver-plugin 插件的介绍我们知道，其中的 SegmentParseV2 会解析收到的 UpstreamSegment 得到相应的 TraceSegment，然后交给所有 RecordStreamProcessor 处理，如果存储选择 ElasticSearch，则 TraceSegment 的全部数据最终会按照序列化的格式存储到 segment-yyyyMMdd 索引中的 data_binary 字段中，当然也包括前面新增的 Thread Dump 信息。因此，整个 trace-receiver-plugin 插件以及 OAP 中存储相关的逻辑是无须进行改动的。</p>
<p data-nodeid="33934">需要改动的是 OAP 查询 Trace 的相关逻辑。首先是 query-graphql-plugin 插件，在 trace.graphqls 中我们新增一个 ThreadDump 类型用于展示线程 dump 信息，具体实现如下所示：</p>
<pre class="lang-java" data-nodeid="87979"><code data-language="java">type ThreadDump{
    dumpTimestamp: Long!
    threadInfo: String!
}
type Trace {
    spans: [Span!]!
    threadDumps: [ThreadDump!]! <span class="hljs-comment">// 在 Trace 中添加 threadDumps集合</span>
}
</code></pre>




<p data-nodeid="33936">相应的，需要修改 GraphQL​ 相应的 Java 对象。首先在 server-core 模块的org.apache.skywalking.oap.server.core.query.entity 包添加一个 ThreadDump 对象，如下所示：</p>
<pre class="lang-java" data-nodeid="90319"><code data-language="java"><span class="hljs-meta">@Getter</span>
<span class="hljs-meta">@Setter</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ThreadDump</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> dumpTimestamp;
    <span class="hljs-keyword">private</span> String threadInfo;
}
</code></pre>




<p data-nodeid="92671" class="">相应的 Trace 类中也要添加 threadDumps 字段（List <code data-backticks="1" data-nodeid="92674">&lt;ThreadDump&gt;</code> 类型），如下所示：</p>
<pre class="lang-java" data-nodeid="95015"><code data-language="java"><span class="hljs-meta">@Getter</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Trace</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;Span&gt; spans;
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;ThreadDump&gt; threadDumps;
}
</code></pre>












<p data-nodeid="33940">接下来就是填充该 ThreadDump 集合，TraceQuery.queryTrace() 方法是查询 Trace 详细信息的入口。在其中完成所有 SegmentRecord 的查询之后，我们可以将每个 Segment 携带的 ThreadDump 取出来填充上述 ThreadDump 集合。具体实现如下所示：</p>
<pre class="lang-java" data-nodeid="97355"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Trace <span class="hljs-title">queryTrace</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String traceId)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp; &nbsp; Trace trace = <span class="hljs-keyword">new</span> Trace();
&nbsp; &nbsp; <span class="hljs-comment">// 根据traceId查询所有关联的 SegmentObject</span>
&nbsp; &nbsp; List&lt;SegmentRecord&gt; segmentRecords =
          getTraceQueryDAO().queryByTraceId(traceId);
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (SegmentRecord segment : segmentRecords) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 反序列化 SegmentObject</span>
&nbsp; &nbsp; &nbsp; &nbsp; SegmentObject segmentObject = 
               SegmentObject.parseFrom(segment.getDataBinary());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 解析 SegmentObject中的 Span，填充到 Trace中</span>
&nbsp; &nbsp; &nbsp; &nbsp; trace.getSpans().addAll(buildSpanV2List(traceId, 
              segment.getSegmentId(), segment.getServiceId(), 
                 segmentObject.getSpansList()));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 填充 ThreadDump集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; trace.getThreadDumps().addAll(
          buildThreadDumpList(segmentObject.getThreadDumpsList()));
&nbsp; &nbsp; }
&nbsp; &nbsp; ... ...<span class="hljs-comment">// 省略整理Trace中Span的顺序等操作，</span>
           <span class="hljs-comment">// 这些逻辑在前文f分析query-graphql-plugin插件时已经详细分析过</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> trace;
}
</code></pre>




<p data-nodeid="33942">最后，在 SkyWalking 源码目录下执行如下 maven 命令重新编译打包：</p>
<pre class="lang-java" data-nodeid="99695"><code data-language="java">mvn clean
mvn <span class="hljs-keyword">package</span> -Dcheckstyle.skip -DskipTests
</code></pre>




<p data-nodeid="33944">执行完成后，首先启动 ElasticSearch 和 ZooKeeper 两个服务，然后依次启动 OAP、demo-provider、demo-webapp 以及 apm-webapp，请求 <a href="http://localhost:8000/hello/xxx" data-nodeid="34106">http://localhost:8000/hello/xxx</a> 地址可以正常相应，且能够在 SkyWalking Rocketbot UI 中看到相应 Trace 信息表名修改未破坏对原始 Trace 的兼容。我们还可以使用 PostMan 在 Http 请求中携带 ENABLE_DUMP_FLAG:true 的 Header，然后通过 GraphQL Playground 查询，可以得到类似如下的结果：</p>
<pre class="lang-java" data-nodeid="102035"><code data-language="java">{
&nbsp; <span class="hljs-string">"data"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"trace"</span>: {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"spans"</span>: [
&nbsp; &nbsp; &nbsp; &nbsp; ... ... <span class="hljs-comment">// 省略该 Trace中的Span信息&nbsp;</span>
&nbsp; &nbsp; &nbsp; ],
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"threadDumps"</span>: [ <span class="hljs-comment">// 该Trace携带的ThreadDump信息</span>
&nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"dumpTimestamp"</span>: <span class="hljs-number">1580029989057</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"threadInfo"</span>: <span class="hljs-string">"\"DubboServerHandler-172.17.32.91:20880-thread-36\" Id=106 TIMED_WAITING\n\tat java.lang.Thread.sleep(Native Method)\n\tat com.xxx.service.DefaultHelloService.say$original$MUzxmS45(DefaultHelloService.java:15)\n\t ... ..."</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
        <span class="hljs-comment">// 省略其他 ThreadDump信息</span>
&nbsp; &nbsp; &nbsp; ]
&nbsp; &nbsp; }
&nbsp; }
}
</code></pre>




<h3 data-nodeid="102620" class="">总结</h3>

<p data-nodeid="103782">本课时最后将通过一张图来总结 Thread Dump 功能的关键点：</p>
<p data-nodeid="103783" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/31/FD/Ciqc1F8NcpiAEXK3AANCpmKvbm8849.png" alt="image (18).png" data-nodeid="103791"></p>


<ul data-nodeid="33949">
<li data-nodeid="33950">
<p data-nodeid="33951">Http 请求进入 demo-webapp 之后，tomcat-7.x-8.x-plugin 插件会从其 Header 中查找 ENABLE_DUMP_FLAG 标记并记录到 RuntimeContext 中。之后通过 ContextManager 创建此次请求对应的 TracingContext 对象以及 EntrySpan，在完成 TracingContext 的初始化之后会触发 TracingContextPostConstructListener，即 ThreadDumpManager，记录需要进行 dump 的线程 ID。后续请求执行过程中会调用 create*Span() 方法创建 Span，同时 ThreadDumpManager 中的后台线程也会定时 dump 线程信息，如图中（3）和（4）处所示。</p>
</li>
<li data-nodeid="33952">
<p data-nodeid="33953">接下来，在 demo-webapp 通过 Dubbo 调用 demo-provider 服务的时候，会将生成的 ContextCarrier 对象（包含 ENABLE_DUMP_FLAG 标记）序列化成字符串添加到 RpcContext 中，随 Dubbo 请求发送到下游的 demo-provider。在 demo-provider 服务的 dubbo-plugin 插件中会处理 ContextCarrier，当然也会处理  ENABLE_DUMP_FLAG 标记。</p>
</li>
<li data-nodeid="33954">
<p data-nodeid="33955">回到（5）处，demo-webapp 处理完请求后会关闭 TracingContext，同时会触发所有 TraceContextListener 监听器，其中 ThreadDumpManager 会根据记录的线程 ID 关联 ThreadDump 与 TraceSegment，TraceSegmentServiceClient 则负责通过 gRPC 将序列化后的 TraceSegment 数据发送到后端的 OAP 集群。</p>
</li>
<li data-nodeid="33956">
<p data-nodeid="33957">OAP 服务中的 trace-receiver-plugin 负责接收 Agent 发送的 TraceSegment 数据，解析之后会由 RecordStreamProcessor 存储到 ElasticSearch 中。</p>
</li>
<li data-nodeid="33958">
<p data-nodeid="33959">OAP 服务中的 query-graphql-plugin 插件负责处理查询 Trace 的请求，这里会从 SegmentObject 中获取全部 ThreadDump 填充到 Trace 中返回给用户。</p>
</li>
</ul>
<p data-nodeid="33960">好了，本专栏的全部内容就讲完了，最后的彩蛋我将带你回顾 SkyWalking 架构并展望未来。</p>

---

### 精选评论


