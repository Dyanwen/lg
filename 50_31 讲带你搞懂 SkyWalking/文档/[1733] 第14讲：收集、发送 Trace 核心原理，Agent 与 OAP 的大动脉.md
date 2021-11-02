<p>在前面的课时中，我们深入介绍了 SkyWalking 对 Trace 基本概念的实现。本课时我们将继续深入学习 Trace 相关的 BootService 接口实现类以及 Trace 收集和发送的核心逻辑。Trace 相关的 BootService 接口实现类如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/14/4A/Ciqah16hMVaAatgbAABGB3yEwak805.png" alt="sw0.png"></p>
<h4>ContextManager</h4>
<p>ContextManager 的主要职责就是管理前文介绍的 TracingContext，它会通过 ThreadLocal 将 TracingContext 对象与当前线程进行绑定，这样就实现了 TraceSegment、TracingContext 和 线程三方之间的关联。</p>
<p>ContextManager 有三个核心字段：</p>
<ul>
<li><b>CONTEXT（ThreadLocal 类型）</b><br>
：通过该字段可以将一个 TracingContext 对象与一个线程进行关联。</li>
<li><b>RUNTIME_CONTEXT（ThreadLocal 类型）</b><br>
：RuntimeContext 底层封装了一个 ConcurrentHashMap 集合，可以为当前 TracingContext 记录一些附加信息。</li>
<li><strong>EXTEND_SERVICE（ContextManagerExtendService 类型）</strong>：ContextManagerExtendService 也实现了 BootService 接口，它主要负责创建 TracingContext 对象。</li>
</ul>
<p>虽然 ContextManager 实现了 BootService 接口，但是其 prepare()、boot()、onComplete() 方法都为空实现。ContextManager 提供了与 TracingContext 对应的几乎所有方法，基本实现都是委托给当前线程绑定的 TracingContext 对象，这里以 createEntrySpan() 方法为例进行介绍：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> AbstractSpan <span class="hljs-title">createEntrySpan</span><span class="hljs-params">(String operationName,
         ContextCarrier carrier)</span> </span>{
    SamplingService samplingService = ServiceManager.INSTANCE
            .findService(SamplingService<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;     <span class="hljs-comment">// 采样相关</span>
    AbstractSpan span;
    AbstractTracerContext context;
    <span class="hljs-comment">// 检测ContextCarrier是否合法，其实就是检查它的核心字段是否已填充好</span>
    <span class="hljs-keyword">if</span> (carrier != <span class="hljs-keyword">null</span> &amp;&amp; carrier.isValid()) { 
        samplingService.forceSampled();
        <span class="hljs-comment">// 获取当前线程绑定的TracingContext</span>
        context = getOrCreate(operationName, <span class="hljs-keyword">true</span>); 
        <span class="hljs-comment">// 委托给当前线程绑定的TracingContext来创建EntrySpan</span>
        span = context.createEntrySpan(operationName); 
        <span class="hljs-comment">// 从ContextCarrier提取上游服务传播过来的Trace信息</span>
        context.extract(carrier); 
    } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 没有上游服务的场景</span>
        context = getOrCreate(operationName, <span class="hljs-keyword">false</span>);
        span = context.createEntrySpan(operationName);
    }
    <span class="hljs-keyword">return</span> span;
}
</code></pre>
<p>getOrCreate() 方法会从 CONTEXT 字段中获取当前线程绑定的 TracingContext 对象，如果当前线程没有关联 TracingContext 上下文，则会通过 ContextManagerExtendService 新建并绑定。</p>
<p>stopSpan() 方法在关闭 Span 的同时，还会检查当前 TraceSegment 是否结束，TraceSegment 结束时会将存储在 CONTEXT 中的 TracingContext 对象以及 RUNTIME_CONTEXT 中的附加信息一并清除，这也是为了防止内存泄露的一步重要操作。</p>
<h4>Context 生成与采样</h4>
<p>如果不做任何限制，每个请求都应该生成一条完整的 Trace。在面对海量请求时如果也同时产生海量 Trace，就会给网络和存储带来双倍的压力，浪费很多资源。为了解决这个问题，几乎所有的 Trace 系统都会支持采样的功能。SamplingService 就是用来实现采样功能的 BootService 实现。</p>
<p>SamplingService 的采样逻辑依赖 samplingFactorHolder 字段（AtomicInteger 类型）的自增。ContextManagerExtendService 是负责创建 TracingContext 的 BootService 实现，在 ContextManagerExtendService 创建 TracingContext 时，会调用 SamplingService 的 trySampling() 方法递增 samplingFactorHolder 字段（CAS 操作），当增加到阈值（默认值为 3，可以通过 agent.sample_n_per_3_secs 配置进行修改）时会返回 false，表示采样失败，这时 ContextManagerExtendService 就会生成 IgnoredTracerContext，IgnoredTracerContext 是个空 Context 实现，不会记录 Trace 信息。</p>
<p>另外，SamplingService 中会启动一个定时任务，每秒都会将 samplingFactorHolder 字段清零，这样就实现了每秒采样指定条数的 Trace 数据，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/14/49/Ciqah16hMHmAT7-VAALHNKUBOhU815.png" alt="sw1.png"></p>
<h4>Trace 的收集</h4>
<p>这里我们先来回顾一个知识点，当 TracingContext 通过 stopSpan() 方法关闭最后一个 Span 时，会调用 finish() 方法关闭相应的 TraceSegment，与此同时，还会调用 TracingContext.ListenerManager.notifyFinish() 方法通知所有监听 TracingContext 关闭事件的监听器 —— TracingContextListener。TracingContext.finish() 方法的相关实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">finish</span><span class="hljs-params">()</span> </span>{
    TraceSegment finishedSegment = 
          segment.finish(isLimitMechanismWorking());
    TracingContext.ListenerManager.notifyFinish(finishedSegment);
}
</code></pre>
<p>TraceSegmentServiceClient 是 TracingContextListener 接口的唯一实现，其主要功能就是在 TraceSegment 结束时对其进行收集，并发送到后端的 OAP 集群。</p>
<p>下图展示了 TraceSegmentServiceClient 的核心结构：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/14/4A/Ciqah16hMNGAO2HOAAEZleWtX24011.png" alt="sw2.png"></p>
<p>TraceSegmentServiceClient 底层维护了一个 DataCarrier 对象，其底层 Channels 默认有 5 个 Buffer，每个 Buffer 长度为 300，使用的是 IF_POSSIBLE 阻塞写入策略，底层会启动一个 ConsumerThread 线程。</p>
<p>TraceSegmentServiceClient 作为一个 TracingContextListener 接口的实现，会在 notifyFinish() 方法中，将刚刚结束的 TraceSegment 写入到 DataCarrier 中缓存。同时，TraceSegmentServiceClient 实现了前面介绍的 IConsumer 接口，封装了消费 Channels 中数据的逻辑，在 consume() 方法中会首先将消费到的 TraceSegment 对象序列化，然后通过 gRPC 请求发送到后端 OAP 集群。该过程涉及的 gRPC 接口定义如下：</p>
<pre><code data-language="java" class="lang-java">service TraceSegmentReportService {
    <span class="hljs-function">rpc <span class="hljs-title">collect</span> <span class="hljs-params">(stream UpstreamSegment)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(Commands)</span> </span>{
    }
}
</code></pre>
<p>该 gRPC 请求中用到的 UpstreamSegment 结构体包含了 Trace ID 以及 TraceSegment 序列化之后的字节数组，定义如下所示：</p>
<pre><code data-language="java" class="lang-java">message UpstreamSegment {
    repeated UniqueId globalTraceIds = <span class="hljs-number">1</span>;
    bytes segment = <span class="hljs-number">2</span>; <span class="hljs-comment">// TraceSegment信息</span>
}
</code></pre>
<p>这个过程中，TraceSegment 对象会转换成相应的 proto 结构体实例，下图展示了 UpstreamSegment 中包含的具体信息：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/14/4A/Ciqah16hMPmAaAOWAAPeM8ggypA230.png" alt="sw3.png"></p>
<p>既然要发送 gRPC 请求，就必然要依赖网络连接，TraceSegmentServiceClient 实现了 GRPCChannelListener 接口，可以监听底层网络连接的变化情况。在 prepare() 方法中可将其作为 Listener 注册到前文介绍的 GRPCChannelManager 中。</p>
<p>明确了发送 Trace 时的具体数据，以及其涉及的 gRPC 请求和接口定义，我们再来看 consume() 方法的具体实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">consume</span><span class="hljs-params">(List&lt;TraceSegment&gt; data)</span> </span>{
    <span class="hljs-keyword">if</span> (CONNECTED.equals(status)) { <span class="hljs-comment">// 根据底层网络连接的状态决定是否发送</span>
        <span class="hljs-comment">// 创建GRPCStreamServiceStatus对象</span>
        <span class="hljs-keyword">final</span> GRPCStreamServiceStatus status = 
             <span class="hljs-keyword">new</span> GRPCStreamServiceStatus(<span class="hljs-keyword">false</span>);
        StreamObserver&lt;UpstreamSegment&gt; upstreamSegmentStreamObserver
               = serviceStub.collect(<span class="hljs-keyword">new</span> StreamObserver&lt;Commands&gt;() {
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onNext</span><span class="hljs-params">(Commands commands)</span> </span>{}
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onError</span><span class="hljs-params">(Throwable throwable)</span> </span>{
                <span class="hljs-comment">// 发生异常会调用 finished()方法，停止等待</span>
                status.finished();
                <span class="hljs-comment">// 通知GRPCChannelManager重新创建网络连接</span>
                ServiceManager.INSTANCE.findService(
                   GRPCChannelManager<span class="hljs-class">.<span class="hljs-keyword">class</span>).<span class="hljs-title">reportError</span>(<span class="hljs-title">throwable</span>)</span>;
            }
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onCompleted</span><span class="hljs-params">()</span> </span>{
                <span class="hljs-comment">// 发送成功之后，会调用finished()方法结束等待</span>
                status.finished(); 
            }
        });
        <span class="hljs-keyword">for</span> (TraceSegment segment : data) { 
            <span class="hljs-comment">// 将TraceSegment转换成UpstreamSegment对象，然后才能进行序列化以</span>
            <span class="hljs-comment">// 及发送操作transform()方法实现的转换逻辑并不复杂，填充字段而已</span>
            UpstreamSegment upstreamSegment = segment.transform();
            upstreamSegmentStreamObserver.onNext(upstreamSegment);
        }
        upstreamSegmentStreamObserver.onCompleted();
        status.wait4Finish(); <span class="hljs-comment">// 等待全部TraceSegment数据发送结束</span>
        segmentUplinkedCounter += data.size(); <span class="hljs-comment">// 统计发送的数据量</span>
    } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 网络连接断开时，只进行简单统计，数据将被直接抛弃</span>
        segmentAbandonedCounter += data.size();
    }
    printUplinkStatus(); <span class="hljs-comment">// 每隔 30s打印一下发送日志</span>
}
</code></pre>
<p>注意，TraceSegmentServiceClient 在批量发送完 UpstreamSegment 数据之后，会通过 GRPCStreamServiceStatus 进行自旋等待，直至该批 UpstreamSegment 全部发送完毕。</p>
<p>最后总结一下，TraceSegmentServiceClient 同时实现了 BootService、IConsumer、GRPCChannelListener、TracingContextListener 四个接口，如下图所示，这四个接口的实现相互依赖，共同完成 Trace 数据的收集和发送：<br>
<img src="https://s0.lgstatic.com/i/image3/M01/07/1B/CgoCgV6hMS2AQpyZAAFcVM04dCk973.png" alt="sw4.png"></p>
<h4>总结</h4>
<p>本课时我们重点介绍了 Trace 相关的 BootService 接口实现。首先介绍了 ContextManager 的核心实现，理清了它是如何将 TracingContext 与当前线程关联起来的。接下来介绍了 SamplingService 实现客户端 Trace 采样的逻辑。最后介绍了上报 Trace 的 gRPC 接口，深入分析了 TraceSegmentServiceClient 收集和上报 Trace 数据的核心逻辑。</p>

---

### 精选评论


