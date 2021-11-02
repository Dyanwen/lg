<p>在第 11 课时中，我介绍了 Agent 中 JVMService 的核心原理，它会定期通过 JMX 获取 JVM 监控信息，然后通过 JVMMetricReportService 这个 gRPC 接口上报到后端 OAP 集群。</p>
<p>本节课我将深入分析 SkyWalking OAP 对 JVM 监控数据的处理。</p>
<h3>JVMMetricReportServiceHandler</h3>
<p>在 SkyWalking OAP 提供了 jvm-receiver-plugin 插件用于接收 Agent 发送的 JVMMetric 。jvm-receiver-plugin 插件的 SPI 配置文件中指定的 ModuleDefine 实现是 JVMModule（名称为 receiver-jvm），ModuleProvider 实现是 JVMModuleProvider（名称为 default）。在 JVMModuleProvider 的 start() 方法中会将 JVMMetricReportServiceHandler 注册到 GRPCServer 中。JVMMetricReportServiceHandler 实现了 JVMMetric.proto 文件中定义的 JVMMetricReportService gRPC 接口，其 collect() 方法负责处理 JVMMetric 对象。</p>
<p>首先，会通过 TimeBucket 工具类整理对齐每个 JVMMetric 所在的时间窗口，TimeBucket 会根据指定的 DownSampling 精度生成不同格式的时间窗口，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/79/Ciqc1F7YtnGAMAYGAAJwEbFWmzY337.png" alt="image (5).png"></p>
<p>JVMMetricReportServiceHandler 中默认使用的 DownSampling 值为 Minute。</p>
<p>接下来，JVMMetricReportServiceHandler 会将 JVMMetrics 交给 JVMSourceDispatcher 处理，JVMSourceDispatcher 会按照 CPU、Memory、MemoryPool、GC 四个大类对监控数据进行拆分并转发：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">sendMetric</span><span class="hljs-params">(<span class="hljs-keyword">int</span> serviceInstanceId, <span class="hljs-keyword">long</span> minuteTimeBucket, 
      JVMMetric metrics)</span> </span>{
    <span class="hljs-comment">// 获取 JVMMetric 对应的 ServiceId</span>
    ServiceInstanceInventory serviceInstanceInventory = 
          instanceInventoryCache.get(serviceInstanceId);
    <span class="hljs-keyword">int</span> serviceId = serviceInstanceInventory.getServiceId();
    <span class="hljs-comment">//将 JVMMetric 分类转发</span>
    <span class="hljs-keyword">this</span>.sendToCpuMetricProcess(serviceId, serviceInstanceId, 
          minuteTimeBucket, metrics.getCpu());
    <span class="hljs-keyword">this</span>.sendToMemoryMetricProcess(serviceId, serviceInstanceId, 
          minuteTimeBucket, metrics.getMemoryList());
    <span class="hljs-keyword">this</span>.sendToMemoryPoolMetricProcess(serviceId, serviceInstanceId, 
          minuteTimeBucket, metrics.getMemoryPoolList());
    <span class="hljs-keyword">this</span>.sendToGCMetricProcess(serviceId, serviceInstanceId, 
          minuteTimeBucket, metrics.getGcList());
}
</code></pre>
<p>这里先以 JVM GC 的监控数据为例进行分析。在 sendToGCMetricProcess() 方法中会将 GC 对象转换为 ServiceInstanceJVMGC 对象（ServiceInstanceJVMGC 中除了包含 GC 对象中的监控数据，还记录了 serviceId 以及 serviceInstanceId，也就明确了这些监控数据的归属）。然后， DispatcherManager 会将 ServiceInstanceJVMGC 转发到相应的 SourceDispatcher。</p>
<p>同理，JVMMetric 中关于 CPU、Memory、MemoryPool 的三类监控数据分别填充到了 ServiceInstanceJVMCPU、ServiceInstanceJVMMemory、ServiceInstanceJVMMemoryPool 对象中，继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/79/Ciqc1F7YtoOABepsAAEMFZwAVUo208.png" alt="image (6).png"></p>
<h3>Dispatcher &amp; DispatcherManager</h3>
<p>在 DispatchManager&nbsp;中维护了一个&nbsp;Map&lt;Integer, List<sourcedispatcher>&gt; 集合，该集合记录了各个 Source 类型对应的 Dispatcher 实现，其中 Key 是&nbsp;Source&nbsp;类型对应的 scope 值，Source 的不同子类对应不同的 scope 值，例如：ServiceInstanceJVMGC&nbsp;对应的 scope 值为 11，ServiceInstanceJVMCPU&nbsp;对应的 scope 值为 8。其中的 Value 是处理该 Source 子类的 Dispatcher 集合，例如：ServiceInstanceJVMGCDispatcher 就是负责分发 ServiceInstanceJVMGC 的 SourceDispatcher 实现，ServiceInstanceJVMGCDispatcher 的定义如下：</sourcedispatcher></p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">ServiceInstanceJVMGCDispatcher</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">SourceDispatcher</span>&lt;<span class="hljs-title">ServiceInstanceJVMGC</span>&gt;&nbsp;</span>{...}
</code></pre>
<p>在 CoreModuleProvider 启动的时候（即 start() 方法），会扫描 classpath 下全部 SourceDispatcher 实现类，并识别其处理的 Source 子类类型进行分类并填充 Map&lt;Integer, List<sourcedispatcher>&gt; 集合。具体的扫描逻辑在 DispatcherManager.scan() 方法中，如果你感兴趣可以翻一下代码。</sourcedispatcher></p>
<blockquote>
<p>还有需要注意的是，这些 SourceDispatcher 的部分实现是通过 OAL 脚本生成的，OAL 语言的内容在后面会展开分析，这里先专注于监控指标的处理流程上。</p>
</blockquote>
<p>回到&nbsp;ServiceInstanceJVMGC 的处理流程上，默认与它对应的 SourceDispatcher 实现只有&nbsp;ServiceInstanceJVMGCDispatcher，其 dispatch() 方法会将&nbsp;ServiceInstanceJVMGC 对象转换成相应的 Metrics&nbsp;对象，实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">dispatch</span><span class="hljs-params">(ServiceInstanceJVMGC source)</span> </span>{
    doInstanceJvmYoungGcTime(source);
    doInstanceJvmOldGcTime(source);
    doInstanceJvmYoungGcCount(source);
    doInstanceJvmOldGcCount(source);
}
</code></pre>
<p>这里的 doInstanceJvm*() 方法是将 ServiceInstanceJVMGC 转换成相应的 Metrics ，我们可以看到，在 ServiceInstanceJVMGC 中包含了 GCPhrase、GC 时间、 GC 次数三个维度的数据，而转换后的一个 Metrics 子类型只表示一个维度的监控数据。这里涉及的 Metrics 子类如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/79/Ciqc1F7Ytp6AYxkTAAE2IyTdv1A381.png" alt="image (7).png"></p>
<p>在前面的“SkyWalking OAP 存储体系剖析”课时中提到了 Metrics 抽象类，你可以回顾一下，Metrics 抽象类是所有监控指标的顶级抽象，其中定义了一个 TimeBucket 字段（long 类型），用于记录该监控数据所在的分钟级窗口。</p>
<p>下面来看上图涉及的 Metrics 子类，在 LongAvgMetrics 抽象类中增加了下面三个字段：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Column</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> summation; <span class="hljs-comment">// 总和</span>
<span class="hljs-meta">@Column</span>&nbsp;<span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> count; <span class="hljs-comment">// 次数</span>
<span class="hljs-meta">@Column</span>&nbsp;<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> value; <span class="hljs-comment">// 平均值</span>
</code></pre>
<p>在 combine() 方法实现中，会将传入的 LongAvgMetrics 对象的 summation 和 count 字段累加到当前 LongAvgMetrics 对象中。在 calculate() 方法中会计算 value 字段的值：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">this</span>.value = <span class="hljs-keyword">this</span>.summation / <span class="hljs-keyword">this</span>.count;
</code></pre>
<p>下面会以 Old GC Time 监控数据继续分析，这里的 doInstanceJvmOldGcTime() 方法会将 ServiceInstanceJVMGC 转换成 InstanceJvmOldGcTimeMetrics，其中又添加了 serviceId 和 entityid（用于构造 Document Id，InstanceJvmOldGcTimeMetrics 中就是 serviceInstanceId）两个字段，用于记录该 GC 监控数据所属的服务实例：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Column</span>(columnName = <span class="hljs-string">"entity_id"</span>) <span class="hljs-meta">@IDColumn</span> <span class="hljs-keyword">private</span> String entityId;
<span class="hljs-meta">@Column</span>(columnName = <span class="hljs-string">"service_id"</span>)  <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> serviceId;
</code></pre>
<p>SkyWalking OAP 中很多其他类型的监控数据，例如：</p>
<ul>
<li><strong>SumMetrics</strong> 计算的是时间窗口内的总和。</li>
<li><strong>MaxDoubleMetrics、MaxLongMetrics</strong> 计算的是时间窗口内的最大值。</li>
<li><strong>PercentMetrics</strong> 计算的是时间窗口内符合条件数据所占的百分比（即 match / total）。</li>
<li><strong>PxxMetrics</strong> 计算的是时间窗口内的分位数，例如： P99Metrics、P95Metrics、P70Metrics等。</li>
<li><strong>CPMMetrics</strong> 计算的是应用的吞吐量，默认是通过分钟级别的调用次数计算的。</li>
</ul>
<p>最后，依旧以 InstanceJvmOldGcTimeMetrics 为例，看看 Metrics 实现类中定义的 ElasticSearch 索引名称以及各个字段对应的 Field 名称：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/85/CgqCHl7YtreATj7GAARBmbdpdkw991.png" alt="image (8).png"></p>
<p>回到 GC 监控数据的处理流程中，在 doInstanceJvmOldGcTime() 方法完成监控数据粒度的细分之后，会将细分后的 InstanceJvmOldGcTimeMetrics 对象交给 MetricsStreamProcessor 处理。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doInstanceJvmOldGcTime</span><span class="hljs-params">(ServiceInstanceJVMGC source)</span> </span>{
    <span class="hljs-comment">// 创建 InstanceJvmOldGcTimeMetrics 对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;InstanceJvmOldGcTimeMetrics&nbsp;metrics&nbsp;=&nbsp;
          <span class="hljs-keyword">new</span>&nbsp;InstanceJvmOldGcTimeMetrics();
    <span class="hljs-keyword">if</span> (!<span class="hljs-keyword">new</span> EqualMatch().setLeft(source.getPhrase())
            .setRight(GCPhrase.OLD).match()) {
        <span class="hljs-keyword">return</span>;     <span class="hljs-comment">// 只处理 Old GC</span>
    }
    metrics.setTimeBucket(source.getTimeBucket()); <span class="hljs-comment">// 分钟级别的时间窗口</span>
    metrics.setEntityId(source.getEntityId()); <span class="hljs-comment">// serviceInstanceId</span>
    metrics.setServiceId(source.getServiceId()); <span class="hljs-comment">// serviceId</span>
    metrics.combine(source.getTime(), <span class="hljs-number">1</span>); <span class="hljs-comment">// 记录 GC 时间，count 为1</span>
    <span class="hljs-comment">// 交给 MetricsStreamProcessor 继续后续处理</span>
    MetricsStreamProcessor.getInstance().in(metrics);
}
</code></pre>
<h3>MetricsStreamProcessor</h3>
<p>前面在介绍服务注册流程的时候，分析过 InventoryStreamProcessor 处理注册请求的核心逻辑，在这里，MetricsStreamProcessor 处理 Metrics 数据的流程也有异曲同工之处。</p>
<p>MetricsStreamProcessor 中为每个 Metrics 类型维护了一个 Worker 链，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> Map&lt;Class&lt;? extends Metrics&gt;, MetricsAggregateWorker&gt; entryWorkers = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
</code></pre>
<p>MetricsStreamProcessor 初始化 entryWorkers 集合的核心逻辑也是在 create() 方法中，下图展示了 InstanceJvmOldGcTimeMetrics 对应的 Worker 链结构：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/7A/Ciqc1F7Yts6AUU4bAABZiou-upc728.png" alt="image (9).png"></p>
<p>具体代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 创建 minutePersistentWorker</span>
MetricsPersistentWorker minutePersistentWorker =   
  minutePersistentWorker(moduleDefineHolder, metricsDAO, model);
<span class="hljs-comment">// 创建 MetricsTransWorker，后续 worker 指向 minutePersistenceWorker 对象(以及</span>
<span class="hljs-comment">// hour、day、monthPersistentWorker)</span>
MetricsTransWorker transWorker = 
    <span class="hljs-keyword">new</span> MetricsTransWorker(moduleDefineHolder, stream.name(), 
        minutePersistentWorker, hourPersistentWorker, 
            dayPersistentWorker, monthPersistentWorker);
<span class="hljs-comment">// 创建 MetricsRemoteWorker，并将 nextWorker 指向上面的 MetricsTransWorker对象</span>
MetricsRemoteWorker remoteWorker = <span class="hljs-keyword">new</span> 
  MetricsRemoteWorker(moduleDefineHolder, transWorker, stream.name());
<span class="hljs-comment">// 创建 MetricsAggregateWorker，并将 nextWorker 指向上面的</span>
<span class="hljs-comment">// MetricsRemoteWorker 对象</span>
MetricsAggregateWorker aggregateWorker =
     <span class="hljs-keyword">new</span> MetricsAggregateWorker(moduleDefineHolder, remoteWorker, 
         stream.name());
<span class="hljs-comment">// 将上述 worker 链与指定 Metrics 类型绑定</span>
entryWorkers.put(metricsClass, aggregateWorker);
</code></pre>
<p>其中 minutePersistentWorker 是一定会存在的，其他 DownSampling（Hour、Day、Month） 对应的 PersistentWorker 则会根据配置的创建并添加，在 CoreModuleProvider.prepare() 方法中有下面这行代码，会获取 Downsampling 配置并保存于 DownsamplingConfigService 对象中配置。后续创建上述 Worker 时，会从中获取配置的 DownSampling。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">this</span>.registerServiceImplementation(DownsamplingConfigService<span class="hljs-class">.<span class="hljs-keyword">class</span>, <span class="hljs-title">new</span> <span class="hljs-title">DownsamplingConfigService</span>(<span class="hljs-title">moduleConfig</span>.<span class="hljs-title">getDownsampling</span>()))</span>;
</code></pre>
<h3>MergeDataCache 缓冲区设计与实现</h3>
<p>在深入介绍上述 Worker 的实现之前，需要先要来介绍一下其中使用到的 MergeDataCache 缓冲区组件的设计与实现。</p>
<p>Window 抽象类是 MergeDataCache&nbsp;缓冲区的基类，使用双缓冲队列的结构实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span>&nbsp;SWCollection&lt;DATA&gt;&nbsp;pointer;&nbsp;<span class="hljs-comment">//&nbsp;指向当前正在写入的缓冲队列</span>
<span class="hljs-keyword">private</span> SWCollection&lt;DATA&gt; windowDataA; <span class="hljs-comment">// A、B两个缓冲队列</span>
<span class="hljs-keyword">private</span>&nbsp;SWCollection&lt;DATA&gt;&nbsp;windowDataB;
</code></pre>
<p>为了便于后面的描述，这里简单区分两个缓冲队列，其中 pointer 指向的队列称为 "current 队列"（一般是有空闲空间的队列，主要负责缓冲新写入数据），另一个队列称为 "last 队列"（一般填充了一定量的数据，会有其他线程从中读取数据进行消费）。</p>
<p>SWCollection 接口定义了缓冲队列的基本行为，下面是其继承关系图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/7A/Ciqc1F7YtumAfQZeAAFcY5KP8TM078.png" alt="image (10).png"></p>
<p>这里重点分析 MergeDataCollection 实现类，它底层是通过一个 HashMap 实现的，一对 KV 中的 Key 和 Value 指向的是同一个 StreamData 对象。MergeDataCollection 暴露了 Map 的基本方法，例如：put、get、containKey 等方法。另外，它还封装了两个 volatile boolean 类型的字段 —— reading、writing，用于标记该缓冲队列的状态，也提供了这两个状态字段相应的 getter/setter 方法。简单说明一下这两个状态字段的含义：</p>
<ul>
<li>当队列的 reading 被设置为 true 时，处于 reading 状态，表示可能有线程在从该队列中读取数据。</li>
<li>当队列的 writing 被设置为 true 时，处于 writing 状态，表示可能有线程在向该队列中写入数据。</li>
</ul>
<p>通过 MergeDataCollection 类暴露方法，我们完全可以让一个队列同时处于 reading 和 writing 两个状态，但是这样会出现并发问题。所以在后面用到 MergeDataCollection 及 MergeDataCache 的地方我们也可以看到，即使一个队列偶尔同时处于两个状态，也会通过循环等待的方式，等待其中一个状态退出。</p>
<p>LimitSizeDateCollection 与 MergeDataCollection 的大致实现类似，区别在于 LimitSizeDateCollection 底层的 Map 类型是 HashMap&lt;T, LinkedList<t>&gt;，Value 中的 LinkedList 都是有序队列。在 LimitSizeDateCollection.put() 方法中会限制每个 LinkedList 的长度，当超过指定的长度时，只会保留最大的 TOP N。</t></p>
<p>回到 Window 抽象类，其中还定义了一个控制 pointer 指针切换的字段，如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> AtomicInteger windowSwitch = <span class="hljs-keyword">new</span> AtomicInteger(<span class="hljs-number">0</span>);
</code></pre>
<p>以及相应的切换检查方法，如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 检查 windowSwitch 字段，以及 last 队列是否处于可读状态</span>
<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;<span class="hljs-title">trySwitchPointer</span><span class="hljs-params">()</span>&nbsp;</span>{
    <span class="hljs-keyword">return</span> windowSwitch.incrementAndGet() == <span class="hljs-number">1</span> 
              &amp;&amp; !getLast().isReading();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;如果此时 last 队列处于 reading 状态，切换后，last 队列会变成current队列，</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;就会出现两个线程(一个读线程、一个写线程)并发操作该队列的可能，所以需要进行</span>
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;reading 状态的检测</span>
}

<span class="hljs-comment">// 在 trySwitchPointer()方法尝试之后，需要在 finally 代码块中恢复windowSwitch</span>
<span class="hljs-comment">//&nbsp;字段的值，为下次检查做准备</span>
<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">trySwitchPointerFinally</span><span class="hljs-params">()</span>&nbsp;</span>{
    windowSwitch.addAndGet(-<span class="hljs-number">1</span>);
}
</code></pre>
<p>在 switchPointer() 方法中实现了 pointer 字段的切换，同时也会更新 last 队列的状态：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">switchPointer</span><span class="hljs-params">()</span>&nbsp;</span>{
    <span class="hljs-keyword">if</span> (pointer == windowDataA) { <span class="hljs-comment">// 根据 pointer 当前的指向，进行修改</span>
        pointer = windowDataB;
    } <span class="hljs-keyword">else</span> {
        pointer = windowDataA;
    }
    getLast().reading(); <span class="hljs-comment">// 修改 last 队列的状态</span>
}
</code></pre>
<p>接下来重点看 MergeDataCache 类的实现，有两个点需要注意：</p>
<ul>
<li>它将 A、B 两个队列初始化为 MergeDataCollection 队列。</li>
<li>它维护了一个 lockedMergeDataCollection 字段。在开始写入的时候，会先调用 writing() 方法将 lockedMergeDataCollection 字段指向当前的 current 队列，直至写入操作完成。即使在写入操作过程中发生了 pointer 的切换，lockedMergeDataCollection 字段的指向也不会发生变化。在写入操作完成之后，会调用 finishWriting() 方法将 lockedMergeDataCollection 字段设置为 null。</li>
</ul>
<p>LimitedSizeDataCache 与 MergeDataCache 的实现有些类似，但功能上有所区别，在后面介绍慢查询的处理（TopNStreamProcessor）时，会介绍 LimitedSizeDataCache 的核心实现。</p>
<h3>MetricsAggregateWorker</h3>
<p>回到&nbsp;InstanceJvmOldGcTimeMetrics&nbsp;的处理流程上继续分析，Worker 链中的第一个是 MetricsAggregateWorker，其功能就是进行简单的聚合，模型如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/86/CgqCHl7YtwqAOhwEAAGQdRPvCuM193.png" alt="image (11).png"></p>
<p>MetricsAggregateWorker&nbsp;在收到 Metrics 数据的时候，会先写到内部的 DataCarrier 中缓存，然后由 Consumer 线程（都属于名为 “METRICS_L1_AGGREGATION” 的 BulkConsumePool）消费并进行聚合，并将聚合结果写入到 MergeDataCache 中的 current 队列暂存。</p>
<p>同时，Consumer 会定期（默认1秒，通过 METRICS_L1_AGGREGATION_SEND_CYCLE 配置修改）触发 current 队列和 last 队列的切换，然后读取 last 队列中暂存的数据，并发送到下一个 Worker 中处理。</p>
<p>上图中写入 DataCarrier 的逻辑在前面已经分析过了，这里不再赘述。下面深入分析两个点：</p>
<ol>
<li>Consumer 线程消费 DataCarrier 并聚合监控数据的相关实现。</li>
<li>Consumer&nbsp;线程定期清理&nbsp;MergeDataCache&nbsp;缓冲区并发送监控数据的相关实现。</li>
</ol>
<p>Consumer 线程在消费 DataCarrier 数据的时候，首先会进行 Metrics 聚合（即相同 Metrics 合并成一个），然后写入 MergeDataCache 中，实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">aggregate</span><span class="hljs-params">(Metrics metrics)</span> </span>{
    <span class="hljs-comment">// 将 lockedMergeDataCollection 指向 current 队列，并设置其 writing标记</span>
&nbsp; &nbsp; mergeDataCache.writing(); 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (mergeDataCache.containsKey(metrics)) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 存在重复的监控数据，则进行合并，</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 不同 Metrics子类的 combine()方法实现有所不同，</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 这里的 InstanceJvmOldGcTimeMetrics 的实现就 summation 的累加、</span>
        <span class="hljs-comment">// count 加一</span>
&nbsp; &nbsp; &nbsp; &nbsp; mergeDataCache.get(metrics).combine(metrics);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 该 Metrics 第一次出现，直接写入到</span>
&nbsp; &nbsp; &nbsp; &nbsp; mergeDataCache.put(metrics);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 清理 current 队列的 writing 标记，之后清理 lockedMergeDataCollection&nbsp;</span>
&nbsp; &nbsp; mergeDataCache.finishWriting();
}
</code></pre>
<p>将聚合后的 Metrics 写入 MergeDataCache 之后，Consumer 线程会每隔一秒将 MergeDataCache 中的数据发送到下一个 Worker 处理，相关实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendToNext</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 首先进行队列切换，之后会设置 last 队列的 reading 状态</span>
&nbsp; &nbsp; mergeDataCache.switchPointer();
&nbsp; &nbsp; <span class="hljs-comment">// 此时可能其他的 Consumer 线程还在写入 last队列，需要等待写入完成</span>
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (mergeDataCache.getLast().isWriting()) {
&nbsp; &nbsp; &nbsp; &nbsp; Thread.sleep(<span class="hljs-number">10</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 开始读取 last 队列中的全部 Metrics 数据并发送到下一个 worker 处理</span>
&nbsp; &nbsp; mergeDataCache.getLast().collection().forEach(data -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; nextWorker.in(data);
&nbsp; &nbsp; });
&nbsp; &nbsp; <span class="hljs-comment">// 读取完成后，清空 last 队列以及其 reading 状态</span>
&nbsp; &nbsp; mergeDataCache.finishReadingLast();
}
</code></pre>
<p>MetricsAggregateWorker 的核心逻辑到这里就分析完了。</p>
<p>MetricsAggregateWorker 指向的下一个 Worker 是 MetricsRemoteWorker ，其实现与 RegisterRemoteWorker 类似，底层也是通过 RemoteSenderService 将监控数据发送到远端节点，具体实现不再展开。</p>
<h3>MetricsTransWorker</h3>
<p>MetricsRemoteWorker&nbsp;之后的下一个 worker 是 MetricsTransWorker，其中有四个字段分别指向四个不同 Downsampling 粒度的 PersistenceWorker 对象，如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> MetricsPersistentWorker minutePersistenceWorker;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> MetricsPersistentWorker hourPersistenceWorker;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> MetricsPersistentWorker dayPersistenceWorker;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> MetricsPersistentWorker monthPersistenceWorker;
</code></pre>
<p>MetricsTransWorker.in() 方法会根据上述字段是否为空，将 Metrics 数据分别转发到不同的 PersistenceWorker 中进行处理：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">in</span><span class="hljs-params">(Metrics metrics)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 检测 Hour、Day、Month 对应的 PersistenceWorker 是否为空，若不为空，</span>
&nbsp; &nbsp; <span class="hljs-comment">// 则将 Metrics 数据拷贝一份并调整时间窗口粒度，交到相应的 </span>
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;PersistenceWorker 处理，这里省略了具体逻辑</span>
&nbsp; &nbsp; <span class="hljs-comment">// 最后，直接转发给 minutePersistenceWorker 进行处理</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (Objects.nonNull(minutePersistenceWorker)) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; aggregationMinCounter.inc();
&nbsp; &nbsp; &nbsp; &nbsp; minutePersistenceWorker.in(metrics);
&nbsp; &nbsp; }
}
</code></pre>
<h3>MetricsPersistentWorker</h3>
<p>MetricsPersistentWorker 主要负责 Metrics 数据的持久化，其核心结构如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/7A/Ciqc1F7YtziAPKFvAAFUA42eQDc822.png" alt="image (12).png"></p>
<p>与前文介绍的&nbsp;MetricsAggregateWorker 处理流程类似，MetricsPersistentWorker&nbsp;在接收到 Metrics 数据的时候先将其暂存到 DataCarrier 中，然后由后续 Consumer 线程消费。</p>
<p></p>
<p>Consumer&nbsp;线程实际上调用的是&nbsp;PersistenceWorker.onWork() 方法，PersistenceWorker是&nbsp;MetricsPersistentWorker 的父类，继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/86/CgqCHl7Yt0SAW_XHAAGAflXWY6Q440.png" alt="image (13).png"></p>
<p>RecordPersistenceWorker 等子类在后面会详细分析。</p>
<p>PersistenceWorker.onWork() 方法的逻辑是将 Metrics 数据写入 MergeDataCache 中暂存，待其中积累的数据量到达阈值（固定值 1000）之后，会进行一次批量写入 ElasticSearch 的操作，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">onWork</span><span class="hljs-params">(INPUT input)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 检测 current 队列中缓冲的数据量是否打到阈值</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (getCache().currentCollectionSize() &gt;= batchSize) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 检测是否符合切换缓冲队列的条件，在分析 Windows 抽象类时也说过，</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// trySwitchPointer()会检测 windowSwitch 标记以及 last 队列的 reading状态</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (getCache().trySwitchPointer()) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 切换 current 缓冲队列，同时会设置切换后的 last 队列的 reading标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getCache().switchPointer();&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建一批请求并批量执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; List&lt;?&gt; collection = buildBatchCollection();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; batchDAO.batchPersistence(collection);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">finally</span> { <span class="hljs-comment">// trySwitchPointerFinally()方法会重置 windowSwitch标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; getCache().trySwitchPointerFinally();&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; cacheData(input); <span class="hljs-comment">// 写入缓存</span>
}
</code></pre>
<p>下面深入 onWorker() 方法的细节实现中进行分析。先来看 cacheData() 方法，MetricsPersistentWorker 在该方法实现中提供了标准的写入 MergeDataCache 操作：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cacheData</span><span class="hljs-params">(Metrics input)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 将 lockedMergeDataCollection 指向 current 队列，并设置其 writing 标记</span>
&nbsp; &nbsp; mergeDataCache.writing();&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (mergeDataCache.containsKey(input)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 存在重复的监控数据，则进行合并</span>
&nbsp; &nbsp; &nbsp; &nbsp; Metrics metrics = mergeDataCache.get(input);
&nbsp; &nbsp; &nbsp; &nbsp; metrics.combine(input);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 重新计算该监控值，不同 Metrics 实现的计算方式不同，例如，</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// LongAvgMetrics.calculate()方法就是计算平均值</span>
&nbsp; &nbsp; &nbsp; &nbsp; metrics.calculate();&nbsp;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; input.calculate(); <span class="hljs-comment">// 第一次计算该监控值</span>
&nbsp; &nbsp; &nbsp; &nbsp; mergeDataCache.put(input);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 更新 lockedMergeDataCollection 队列的 writing 状态，然后清空lockedMergeDataCollection</span>
&nbsp; &nbsp; mergeDataCache.finishWriting();
}
</code></pre>
<p>接下来看&nbsp;buildBatchCollection() 方法的实现。在前面的 trySwitchPointer() 检测中只保证了 last 队列已退出 reading 状态，并未检查 current 队列是否已经退出了writing 状态，所以在切换完成后，第一件事就是先循环等待 last 队列（切换前是 current 队列）退出 writing 状态：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">while</span> (getCache().getLast().isWriting()) {&nbsp;
&nbsp; &nbsp; Thread.sleep(<span class="hljs-number">10</span>);&nbsp; &nbsp;<span class="hljs-comment">// 循环检测 last 队列的 writing 状态</span>
}
</code></pre>
<p>当 last 队列解除了 writing 状态的时候，上面的循环会退出，然后执行 prepareBatch() 方法遍历 last 队列，为每一个 Metrics 对象生成一条相应的 IndexRequest 或 UpdateRequest，具体的处理如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 根据id从底层存储中查询 Metrics</span>
Metrics dbData = metricsDAO.get(model, data);
<span class="hljs-keyword">if</span> (nonNull(dbData)) { <span class="hljs-comment">// 已存在相应的 Document</span>
&nbsp; &nbsp; data.combine(dbData); <span class="hljs-comment">// 已存在则进行合并</span>
&nbsp; &nbsp; data.calculate(); <span class="hljs-comment">// 重新计算 value 值</span>
&nbsp; &nbsp; <span class="hljs-comment">// 产生相应的 UpdateRequest 请求，并添加到 batchCollection 集合中</span>
&nbsp; &nbsp; batchCollection.add(metricsDAO.prepareBatchUpdate(model, data));
} <span class="hljs-keyword">else</span> {&nbsp;
&nbsp; &nbsp; <span class="hljs-comment">// 产生相应的 IndexRequest 请求，并添加到 batchCollection 集合中</span>
&nbsp; &nbsp; batchCollection.add(metricsDAO.prepareBatchInsert(model, data));
}
</code></pre>
<p>这里产生的 batchCollection 集合接下来会交给 BatchProcessEsDAO 批量执行，其底层是通过 ES High Level Client 提供的&nbsp;BulkProcessor 实现批量操作的，该部分实现位于&nbsp;BatchProcessEsDAO.batchPersistence() 方法中，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">batchPersistence</span><span class="hljs-params">(List&lt;?&gt; batchCollection)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (bulkProcessor == <span class="hljs-keyword">null</span>) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建 BulkProcessor，创建方式与前面"ElasticSearch基础入门"小节中展示的示例相同，不再重复</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.bulkProcessor = getClient().createBulkProcessor(bulkActions, bulkSize, flushInterval, concurrentRequests);
&nbsp; &nbsp; }
&nbsp; &nbsp; batchCollection.forEach(builder -&gt; { <span class="hljs-comment">// 遍历batchCollection，将 Request添加到BulkProcessor中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (builder <span class="hljs-keyword">instanceof</span> IndexRequest) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.bulkProcessor.add((IndexRequest)builder);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (builder <span class="hljs-keyword">instanceof</span> UpdateRequest) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.bulkProcessor.add((UpdateRequest)builder);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; });
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.bulkProcessor.flush(); <span class="hljs-comment">// 将上面添加的请求发送到 ElasticSearch 集群执行</span>
}
</code></pre>
<p>从上面的代码中我们可以看到，在一个 OAP 实例中只会创建一个&nbsp;BulkProcessor 对象，然后将其封装到 BatchProcessEsDAO 中循环使用。</p>
<h3>总结</h3>
<p>到这里，SkyWalking OAP 处理 Metrics 监控数据的整个流程就分析完了。下面通过一张图总结整个处理流程：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/18/7B/Ciqc1F7Yt2OAJnGpAABb_jqSXp0511.png" alt="image (14).png"></p>
<p>JVMMetricReportServiceHandler 在收到 JVM Metrics 请求时，由 DispatcherManager 对 JVMMetric 进行分类（&nbsp;CPU、Memory、MemoryPool、GC 四类）并转换成相应的 Source 对象，接下来根据 Source 类型查找相应的 SourceDispatcher 集合进行处理。</p>
<p>在 SourceDispatcher 中会将监控数据再次拆分，转换单一维度的 Metrics 对象，例如，在 ServiceInstanceJVMGCDispatcher 中会将 GC 监控拆分成 Old GC Time、Old GC Count、New GC Time、New GC Count 四类。之后，SourceDispatcher 会将 Metrics 对象传递给 MetricsStreamProcessor 中的 worker 进行处理。</p>
<p>MetricsAggregateWorker 通过 MergeDataCache 对 Metrics 数据进行暂存以及简单聚合。</p>
<p>MetricsRemoteWorker 通过底层的 RemoteSenderService 将 Metrics 数据送到 OAP 集群中的其他远端节点。</p>
<p>MetricsTransWorker 会将 Metrics 数据复制多份，转发到各个 DownSampling 对应的 MetricsPersistentWorker 中实现持久化。</p>
<p>MetricsPersistentWorker 会先将数据缓存在 MergeDataCache 中，当缓存数据量到达一定阈值，执行批量写入（或更新） ElasticSearch 操作，批量操作是通过 High Level Client 中的 BulkProcessor 实现的。</p>

---

### 精选评论


