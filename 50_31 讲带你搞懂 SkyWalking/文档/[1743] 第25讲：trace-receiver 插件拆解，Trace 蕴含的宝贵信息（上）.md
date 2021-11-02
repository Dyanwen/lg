<p>在上一课时中，我介绍了 SkyWalking OAP 处理 Metrics 监控数据的完整流程。本课时将开始介绍 Trace 处理的相关内容。</p>
<h3>TraceModuleProvider</h3>
<p>本课时重点内容是来看 OAP 中的 trace-receiver-plugin 如何接收 TraceSegment 数据。</p>
<p>目前 trace-receiver-plugin 插件同时支持处理 V1 和 V2 两个版本的 TraceSegment。本课时重点分析 V2 版本 TraceSegment，后面不进行特殊说明的情况下，都是指 V2 版本 TraceSegment。</p>
<p>在 trace-receiver-plugin 插件的 SPI 文件中指定的 ModuleProvider 实现是 TraceModuleProvider，在 prepare() 方法中主要初始化 SegmentParseV2 解析器，SegmentParseV2 主要负责解析 TraceSegment 数据，具体实现后面会详细分析。</p>
<p>TraceModuleProvider 的 start() 方法核心是将 TraceSegmentReportServiceHandler 注册到 GRPCHandlerRegister 中。TraceSegmentReportServiceHandler 负责接收 Agent 发送来的 TraceSegment 数据，并调用 SegmentParseV2.parse() 方法进行解析，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D7/CgqCHl7fSYGAKb4lAAEg6Xckx78215.png" alt="Drawing 0.png"></p>
<h3>SegmentParseV2</h3>
<p>每处理一个 UpstreamSegment 都会创建一个相应的 SegmentParseV2 解析器对象，其核心字段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 在解析过程中产生的 Segment 核心数据都会记录到 SegmentCoreInfo 中</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> SegmentCoreInfo segmentCoreInfo;
<span class="hljs-comment">//&nbsp;在解析 TraceSegment 过程中会碰到不同类型的 Span</span>
<span class="hljs-comment">// 会通过不同的 SpanListener 执行不同的操作</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">final</span>&nbsp;List&lt;SpanListener&gt;&nbsp;spanListeners;
</code></pre>
<p>SegmentCoreInfo 是一个 POJO，用于记录解析 TraceSegment 时产生的核心数据，作用类似于一个 DTO（后面会看到各个 SpanListener 都会从它这里拷贝 TraceSegment 的数据），其核心字段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// TraceSegment 编号，即 TraceSegment.traceSegmentId 。</span>
<span class="hljs-keyword">private</span>&nbsp;String&nbsp;segmentId;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> serviceId; <span class="hljs-comment">// Segment 所属的 Service 以及 ServiceInstance</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;serviceInstanceId;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> startTime; <span class="hljs-comment">// Segment 的开始时间和结束时间</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> endTime;
<span class="hljs-comment">// 如果 TraceSegment 范围内的任意一个 Span 被标记了 Error，则该字段会被设置为true</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isError;
<span class="hljs-comment">// TraceSegment 开始时间窗口(即第一个 Span 开始时间所处的分钟级时间窗口)</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> minuteTimeBucket;
<span class="hljs-comment">//&nbsp;整个 TraceSegment 的字节数据</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">byte</span>[]&nbsp;dataBinary;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isV2; <span class="hljs-comment">// 是否为 V2 版本的 TraceSegment 数据</span>
</code></pre>
<p>这里的 SpanListener 接口分为 GlobalTraceIdsListener、FirstSpanListener、LocalSpanListener、EntrySpanListener、ExitSpanListener 五个子接口，不同类型的子接口监听 SegmentParseV2 解析过程中产生的不同数据，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D7/CgqCHl7fSYqALG1gAAGuE9qIaPk072.png" alt="Drawing 1.png"></p>
<p>而这五个子接口的真正实现类只有 SegmentSpanListener 三个，默认情况下都会添加到 spanListeners 集合中。</p>
<p>在 SegmentParseV2.parse() 中的第一步就是初始化 spanListeners 集合，将这三个实现类的对象添加到 spanListeners 集合中。</p>
<p>第二步会从 UpstreamSegment 中读取 TraceSegment 的数据，其中主要包括：</p>
<ol>
<li>与该 TraceSegment 关联的全部 TraceId 。</li>
<li>反序列化 TraceSegment 关联的元数据以及 Span 数据，得到对应的 SegmentObject 对象。</li>
</ol>
<p>该步骤相关代码如下：</p>
<pre><code data-language="java" class="lang-java">UpstreamSegment upstreamSegment = bufferData.getMessageType();
<span class="hljs-comment">// 获取该 TraceSegment 关联的全部 TraceId</span>
List&lt;UniqueId&gt; traceIds = upstreamSegment.getGlobalTraceIdsList();
<span class="hljs-comment">// 反序列化 UpstreamSegment.segment，得到 SegmentObject 对象</span>
SegmentObject segmentObject = parseBinarySegment(upstreamSegment));
<span class="hljs-comment">// 将 SegmentObject 封装成 SegmentDecorator</span>
SegmentDecorator segmentDecorator = <span class="hljs-keyword">new</span> SegmentDecorator(segmentObject);
</code></pre>
<p>这里先帮助你回顾一下，Unique 中封装了三个 long，这三个 long 拼接起来就是一个 TraceId，这与前面介绍的 DistributedTraceId 一致。</p>
<p>SegmentObject 中封装了 TraceSegment 的全部信息，其中包括 TraceSegmentId、serviceId、serviceInstanceId 以及每个 Span 的数据（封装在 SpanObjectV2 中）。在 SpanObjectV2 中封装了一个 Span 的全部信息，例如：spanId、parentSpanId、startTime、endTime 等等（全部字段可以参考 apm-protocol/apm-network/src/main/proto/language-agent-v2/trace.proto 文件）。</p>
<h3>StandardBuilder</h3>
<p>SegmentParseV2 解析得到的 SegmentObject 会立即封装到 SegmentDecorator 中，它是 StandardBuilder 接口的实现类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D7/CgqCHl7fSZuAcM1EAAG3Hx3xP4Y420.png" alt="Drawing 2.png"></p>
<p>为了更清晰的说明 StandardBuilder 对象的作用，这里以 SpanDecorator 为例进行说明。在 SpanDecorator 底层封装了&nbsp;SpanObjectV2&nbsp;对象以及其关联的 Builder 对象，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 底层封装的 SpanObjectV2 对象</span>
<span class="hljs-keyword">private</span> SpanObjectV2 spanObjectV2;
<span class="hljs-comment">// SpanObjectV2 关联的 Builder</span>
<span class="hljs-keyword">private</span> SpanObjectV2.Builder spanBuilderV2;
<span class="hljs-comment">// Builder 是否已经创建</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isOrigin = <span class="hljs-keyword">true</span>;
</code></pre>
<p>SpanDecorator&nbsp;暴露出来的主要是&nbsp;SpanObjectV2&nbsp;相应字段的 getter/setter 方法：</p>
<ol>
<li>其 getter 方法底层调用 SpanObjectV2 或是 Builder 相应的 getter 方法读取相应字段数据；</li>
<li>其 setter 方法底层只能通过 Builder 的 setter 方法修改相应字段的数据（通过 isOrigin 字段确定&nbsp;SpanBuilderV2&nbsp;是否已经初始化，若未初始化则先进行初始化）。</li>
</ol>
<p>以 SpanDecorator 中的 setComponentId() 方法为例进行介绍，其他方法的实现与其类似：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setComponentId</span><span class="hljs-params">(<span class="hljs-keyword">int</span> value)</span> </span>{
    <span class="hljs-keyword">if</span> (isOrigin) { <span class="hljs-comment">// 先检查 isOrigin，确定 spanBuilderV2 字段是否已经初始化</span>
        toBuilder(); <span class="hljs-comment">// 初始化 spanBuilderV2 字段，即创建 SpanObjectV2 关联的 Builder 对象</span>
    }
   <span class="hljs-comment">// 通过 Builder 完成更新（这里省略 V1 版本的相关代码）</span>
   spanBuilderV2.setComponentId(value);
}
</code></pre>
<p>通过 SpanDecorator 装饰器的封装，屏蔽了底层 SpanObjectV2 和 SpanObjectV.Builder 切换的实现细节，外层可以像 POJO 那样直接使用 getter/setter 修改字段值。</p>
<p>最后，为什么不用 SegmentDecorator 进行举例呢？因为 SegmentDecorator 只提供了 getter 方法，而没有 setter 方法，用它举例可能会造成误导。</p>
<h4>preBuild</h4>
<p>下面回到 SegmentParseV2 继续介绍解析 TraceSegment 的过程。完成 TraceSegment 数据的读取和反序列化之后，接下来就是预构建（preBuild）操作。预构建操作分为很多步，我们一步步进行分析。整个过程会涉及上面提到的三个 SpanListener 接口实现类，TraceSegment 的数据会从 SegmentCoreInfo 中拷贝多份，流向不同的处理流程。这里为了防止混乱，先以 SegmentSpanListener 为主线进行分析，等你了解 SegmentParseV2 的整个处理流程之后，再介绍另外两个 SpanListener 实现的具体实现。</p>
<p>预构建首先会通过 notifyGlobalsListener() 方法将该 TraceSegment 关联的全部 TraceId 交给 GlobalTraceIdsListener 进行解析。GlobalTraceIdsListener 接口只定义了一个 parseGlobalTraceId() 方法，SegmentSpanListener 、MultiScopesSpanListener 都是该接口的实现类。SegmentSpanListener 首先会对 TraceId 进行采样，采样逻辑在 TraceSegmentSampler 中实现，采样的大致逻辑是计算 TraceId 中第三部分的 long 值（即线程内递增值）与 10000&nbsp;取模，小于 sampleRate 即被采样（sampleRate 是在 application.yml 文件中配置的万分比采样率），采样的代码片段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">long</span>&nbsp;sampleValue&nbsp;=&nbsp;lastLong&nbsp;%&nbsp;<span class="hljs-number">10000</span>;&nbsp;
<span class="hljs-keyword">if</span> (sampleValue &lt; sampleRate) {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
}
<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">false</span>;
</code></pre>
<p>被采样的 TraceId 会记录到 SegmentSpanListener.segment 字段（Segment 类型）中：</p>
<pre><code data-language="java" class="lang-java">segment.setTraceId(traceIdBuilder.toString()); <span class="hljs-comment">// 记录traceId</span>
</code></pre>
<p>不仅是 TraceId，在后面会看到 SegmentSpanListener 将解析到的所有 TraceSegment 数据都记录到一个 Segment 对象（segment 字段）中。Segment 继承了 Source 抽象类，看到 Source 抽象类是不是很熟悉？当分析完一个 TraceSegment 之后，SegmentSpanListener 会将 Segment 传递给 SourceReceiver（sourceReceiver 字段）处理。SourceReceiver 前面也已经分析过了，其底层封装的 DispatcherManager 会根据 Source 的类型选择相应的 SourceDispatcher 进行分发。</p>
<p>预构建接下来的步骤叫&nbsp;"exchange" ，该步骤主要实现字符串到 id 的转换。前文提到，Agent 会定期将 EndpointName、NetworkAddress 等一系列字符串同步到 OAP 生成相应 id，后续 Agent 生成的 Span 将不再携带这些字符串，而是携带相应的 id。如果在生成 Span 的时候，OAP 还未能及时为这些字符串生成相应 id，则 Agent 会继续使用字符串填充上述字段，后续由 OAP 完成字符串到 id 的转换，也就是这里的 exchange 步骤。</p>
<p>这里的 exchange 过程是通过 IdExchanger 接口实现的，IdExchange 接口有两个实现类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CC/Ciqc1F7fSaaAbOcSAAApWn6Htjw623.png" alt="Drawing 3.png"></p>
<p>IdExchanger 接口只定义了一个 exchange() 方法，也是整个 exchange 过程的核心。下面先来分析 SpanIdExchanger 的实现，其核心字段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ServiceInventoryCache serviceInventoryCacheDAO;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> IServiceInventoryRegister serviceInventoryRegister;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> IEndpointInventoryRegister endpointInventoryRegister;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> INetworkAddressInventoryRegister networkAddressInventoryRegister;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> IComponentLibraryCatalogService componentLibraryCatalogService;
</code></pre>
<p>SpanIdExchanger 会依赖这些 Service 获取不同类型字符串对应的唯一 id 值。在&nbsp;exchange() 方法实现主要处理了 Span 中三种数据的 id 转换：component、peer、operationName。这里以 peer 为例进行分析：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 这里的 standardBuilder 就是前面介绍的 SpanDecorator，用于读写SpanObjectV2 中的数据</span>
<span class="hljs-keyword">int</span> peerId = standardBuilder.getPeerId();
<span class="hljs-comment">// 检测该 SpanObjectV2 的 peer 是否需要进行转换</span>
<span class="hljs-keyword">if</span> (peerId == <span class="hljs-number">0</span> &amp;&amp; !Strings.isNullOrEmpty(standardBuilder.getPeer())) {
    <span class="hljs-comment">// 通过 NetworkAddressInventoryRegister 获取 peer 对应的 id</span>
    peerId = networkAddressInventoryRegister.getOrCreate(standardBuilder.getPeer(),
            buildServiceProperties(standardBuilder));

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(peerId&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{&nbsp;<span class="hljs-comment">// 该 peer 字符串没有对应的id</span>
        exchanged = <span class="hljs-keyword">false</span>; 
    } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 记录 peerId，并清空 peer 字段</span>
        standardBuilder.toBuilder();
        standardBuilder.setPeerId(peerId);
        standardBuilder.setPeer(Const.EMPTY_STRING);
    }
}
</code></pre>
<p>component 和 endpointName 的 id 转换逻辑类似，这里不再展开分析。</p>
<p>ReferenceIdExchanger 主要处理 TraceSegmentReference 中的 endpointName、parentEndpointName 以及 NetworkAddress 的 id 转换，具体的转换逻辑与上述逻辑类似，不再展开分析。</p>
<p>完成 exchange 操作之后，SegmentParseV2 会将解析到的 Segment 信息拷贝到 segmentCoreInfo 中暂存。</p>
<p>在预构建的最后，会根据 TraceSegment 中各个 Span 的类型，交给不同的 SpanListener 进行处理，大致实现代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;segmentDecorator.getSpansCount();&nbsp;i++)&nbsp;{
    SpanDecorator spanDecorator = segmentDecorator.getSpans(i);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;针对&nbsp;TraceSegment 中第一个 Span 的处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(spanDecorator.getSpanId()&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{&nbsp;
        notifyFirstListener(spanDecorator);
    }
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;根据&nbsp;SpanType 处理各个 Span</span>
    <span class="hljs-keyword">if</span> (SpanType.Exit.equals(spanDecorator.getSpanType())) {
        notifyExitListener(spanDecorator);
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (SpanType.Entry.equals(spanDecorator.getSpanType())) {
        notifyEntryListener(spanDecorator);
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (SpanType.Local.equals(spanDecorator.getSpanType())) {
        notifyLocalListener(spanDecorator);
    } 
}
</code></pre>
<p>下面我们深入到 notify*Listener() 方法中，分析不同 SpanListener 实现对各个类型的 Span 的处理逻辑。</p>
<h4>notifyFirstListener</h4>
<p>notifyFirstListener() 方法会调用所有 FirstSpanListener 的 parseFirst() 方法处理 TraceSegment 中的第一个 Span，这里只有 SegmentSpanListener 实现了该方法，具体实现分为三步：</p>
<ol>
<li>检测当前 TraceSegment 是否被成功采样。</li>
<li>将 segmentCoreInfo 中记录的 TraceSegment 数据拷贝到 segment 字段中。</li>
<li>将 endpointNameId 记录到 firstEndpointId 字段，通过前面的分析我们知道，endpointNameId 在 Spring MVC 里面对应的是 URL，在 Dubbo 里面对应的是 RPC 请求 path 与方法名称的拼接。</li>
</ol>
<p>SegmentSpanListener.parseFirst() 方法的具体实现代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">parseFirst</span><span class="hljs-params">(SpanDecorator spanDecorator, SegmentCoreInfo segmentCoreInfo)</span> </span>{
    <span class="hljs-keyword">if</span> (sampleStatus.equals(SAMPLE_STATUS.IGNORE)) { 
        <span class="hljs-keyword">return</span>; <span class="hljs-comment">// 检测是否采样成功</span>
    }
    <span class="hljs-keyword">long</span> timeBucket = TimeBucket.getSecondTimeBucket(segmentCoreInfo.getStartTime());
    <span class="hljs-comment">// 将 segmentCoreInfo 中记录的数据全部拷贝到 Segment 中</span>
    segment.setSegmentId(segmentCoreInfo.getSegmentId());
    segment.setServiceId(segmentCoreInfo.getServiceId());
    segment.setServiceInstanceId(segmentCoreInfo.getServiceInstanceId());
    segment.setLatency((<span class="hljs-keyword">int</span>)(segmentCoreInfo.getEndTime() - segmentCoreInfo.getStartTime()));
    segment.setStartTime(segmentCoreInfo.getStartTime());
    segment.setEndTime(segmentCoreInfo.getEndTime());
    segment.setIsError(BooleanUtils.booleanToValue(segmentCoreInfo.isError()));
    segment.setTimeBucket(timeBucket);
    segment.setDataBinary(segmentCoreInfo.getDataBinary());
    segment.setVersion(segmentCoreInfo.isV2() ? <span class="hljs-number">2</span> : <span class="hljs-number">1</span>);
    <span class="hljs-comment">// 记录 endpointNameId</span>
    firstEndpointId = spanDecorator.getOperationNameId();
}
</code></pre>
<h4>notifyEntryListener</h4>
<p>在 notifyEntryListener() 方法中，会调用所有 EntrySpanListener 实现的 parseEntry() 方法对于 Entry 类型的 Span 进行处理。SegmentSpanListener.parseEntry() 方法只做了一件事，就是将 endpointNameId 记录到 entryEndpointId 字段中：</p>
<pre><code data-language="java" class="lang-java">entryEndpointId = spanDecorator.getOperationNameId();
</code></pre>
<h4>notifyLocalListener</h4>
<p>上述的三个 SpanListener 实现类中，全部都没有实现 LocalSpanListener 接口，所以在 trace-receiver-plugin 插件中并不会处理 Local 类型的 Span。</p>
<h4>notifyExitListener</h4>
<p>trace-receiver-plugin 插件中，只有 MultiScopesSpanListener 实现了 ExitSpanListener 接口，后面会单独介绍 MultiScopesSpanListener 对各类 Span 的处理，这里暂不展开。</p>
<p>到此，预构建（preBuild）操作就到此结束了。最后需要提醒的是，preBuild() 方法的返回值是一个 boolean 值，表示 exchange 操作是否已经将全部字符串转换成 id。</p>
<h4>notifyListenerToBuild</h4>
<p>如果预构建（preBuild）中的 exchange 过程已经将全部字符串转换成了相应的 id，则会通过 notifyListenerToBuild() 方法调用所有 SpanListener 实现的 build() 方法。这里重点来看 SegmentSpanListener 的实现：</p>
<ol>
<li>首先会检测 TraceSegment 是否已被采样，它只会处理被采样的 TraceSegment。</li>
<li>设置 Segment 的 endpointName 字段。</li>
<li>将 Segment 交给 SourceReceiver 继续处理。</li>
</ol>
<h3>RecordStreamProcessor</h3>
<p>SourceReceiver 底层封装的 DispatcherManager &nbsp;会根据 Segment 选择相应的 SourceDispatcher 实现 —— SegmentDispatcher 进行分发。</p>
<p>SegmentDispatcher.dispatch() 方法中会将 Segment 中的数据拷贝到 SegmentRecord 对象中。</p>
<p>SegmentRecord 继承了 StorageData 接口，与前面介绍的 RegisterSource 以及 Metrics 的实现类似，通过注解指明了 Trace 数据存储的 index 名称的前缀（最终写入的 index 是由该前缀以及 TimeBucket 后缀两部分共同构成）以及各个字段对应的 field 名称，如下所示：</p>
<pre><code data-language="java" class="lang-java">//&nbsp;@Stream 注解的 name 属性指定了&nbsp;index 的名称(index 前缀)，processor&nbsp;指定了处理该类型数据的&nbsp;StreamProcessor 实现
@Stream(name = "segment", processor = RecordStreamProcessor.class...)
public class SegmentRecord extends Record {
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;@Column 注解中指定了该字段在&nbsp;index 中对应的&nbsp;field 名称
    @Setter @Getter @Column(columnName = "segment_id") private String segmentId;
    @Setter @Getter @Column(columnName = "trace_id") private String traceId;
    @Setter @Getter @Column(columnName = "service_id") private int serviceId;
    @Setter @Getter @Column(columnName = "service_instance_id") private int serviceInstanceId;
    @Setter @Getter @Column(columnName = "endpoint_name, matchQuery = true) private String endpointName;
    @Setter @Getter @Column(columnName = "endpoint_id") private int endpointId;
    @Setter @Getter @Column(columnName = "start_time") private long startTime;
    @Setter @Getter @Column(columnName = "end_time") private long endTime;
    @Setter @Getter @Column(columnName = "latency") private int latency;
    @Setter @Getter @Column(columnName = "is_error") private int isError;
    @Setter @Getter @Column(columnName = "data_binary") private byte[] dataBinary;
    @Setter @Getter @Column(columnName = "version") private int version;
}
</code></pre>
<p>在 SegmentRecord 的父类 —— Record 中还定义了一个 timeBucket 字段（long 类型），对应的 field 名称是 "time_bucket"。</p>
<p>RecordStreamProcessor 的核心功能是为每个 Record 类型创建相应的 worker 链，这与前面介绍的 InventoryStreamProcessor 以及 MetricsStreamProcessor 类似。在 RecordStreamProcessor 中，每个 Record 类型对应的 worker 链中只有一个worker 实例 —— RecordPersistentWorker。</p>
<p>与前面介绍的 MetricsPersistentWorker 类型，RecordPersistentWorker 负责 SegmentRecord 数据的持久化：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D7/CgqCHl7fSbWALRYDAAEn3HDWLfM382.png" alt="Drawing 4.png"></p>
<p>如上图所示，RecordPersistentWorker 也继承了 PersistentWorker，写入流程大致如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CC/Ciqc1F7fSbuAPPvbAAG3YJeCl48222.png" alt="Drawing 5.png"></p>
<p>RecordPersistentWorker 有两个地方与 MetricsPersistentWorker 有些区别：</p>
<ol>
<li>RecordPersistentWorker 中使用的 DataCache（以及 Window）实现是 NoMergeDataCache，它与 MergeDataCache 的唯一区别就是没有提供判断数据是否存在的 containKey()&nbsp;方法，这样就只提供了缓存数据的功能，调用方无法合并重复数据。</li>
<li>当 NoMergeDataCache 中缓存的数据到达阈值之后，RecordPersistentWorker 会通过 RecordDAO 生成批量的 IndexRequest 请求，Trace 数据没有合并的情况，所以 RecordDAO 以及 IRecordDAO 接口没有定义 prepareBatchUpdate() 方法。</li>
</ol>
<p>RecordDAO.perpareBatchInsert() 方法的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> IndexRequest <span class="hljs-title">prepareBatchInsert</span><span class="hljs-params">(Model model, Record record)</span> <span class="hljs-keyword">throws</span> IOException </span>{
    XContentBuilder builder = map2builder(storageBuilder.data2Map(record));
    <span class="hljs-comment">// 生成的是最终 Index 名称，这里的 Index 由前缀字符串(即"segment")+TimeBucket 两部分构成</span>
    String modelName = TimeSeriesUtils.timeSeries(model, record.getTimeBucket());
    <span class="hljs-comment">// 创建 IndexRequest 请求</span>
    <span class="hljs-keyword">return</span> getClient().prepareInsert(modelName, record.id(), builder);
}
</code></pre>
<p>与 MetricsPersistentWorker 一样，RecordPersistentWorker 生成的全部 IndexRequest 请求会交给全局唯一的 BatchProcessEsDAO 实例批量发送到 ES ，完成写入。</p>
<p>到此，以 SegmentSpanListener 为主的 TraceSegment 处理线就介绍完了。</p>

---

### 精选评论


