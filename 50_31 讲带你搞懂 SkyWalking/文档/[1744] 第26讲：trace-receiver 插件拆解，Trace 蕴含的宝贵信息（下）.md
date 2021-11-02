<h3>MultiScopesSpanListener</h3>
<p>接下来重点分析 MultiScopesSpanListener 这条支线对 TraceSegment 数据的处理。MultiScopesSpanListener 继承了 GlobalTraceIdsListener、ExitSpanListener、EntrySpanListener 三个接口，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C8/Ciqc1F7fR2GAbKuNAABGmRcouQo855.png" alt="Drawing 6.png"></p>
<p>从 containsPoint() 方法实现中也可能看出， MultiScopesSpanListener 能够处理 TraceSegment 中的 TraceId、Entry 类型 Span 以及 Exit 类型 Span。按照前面&nbsp;SegmentParseV2&nbsp;解析 TraceSegment 的流程，下面将会按照&nbsp;parseGlobalTraceId() 方法、parseEntry() 方法、parseExit() 方法、build() 方法的顺序依次介绍 MultiScopesSpanListener 逻辑。</p>
<h4>parseGlobalTraceId</h4>
<p>parseGlobalTraceId() 方法比较简单，主要是将 TraceSegment 中的第一个 UniqueId</p>
<p>拼接成 String 记录到 traceId 字段中，这里不再展开分析。</p>
<h4>parseEntry</h4>
<p>前文介绍中提到，在 EntrySpan 的 refs 字段中记录的一个（或多个） TraceSegmentReference ，分别指向了上一个（或是多个）上游系统的 TraceSegment，SkyWalking OAP 可以通过 TraceSegmentReference 将两个系统的调用关系串联起来。</p>
<p>MultiScopesSpanListener.parseEntry() 方法的核心就是来解析 EntrySpan.refs 字段中记录的全部 TraceSegmentReference 对象，为每一个 TraceSegmentReference 生成一个相应的 SourceBuilder 对象并记录下来（ entrySourceBuilders 字段，List<sourcebuilder> 类型）。在 build() 方法中还会继续处理这些 SourceBuilder 对象。</sourcebuilder></p>
<p>在 SourceBuilder 中记录了上下游两个系统的基础信息，核心字段如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 上游系统的 service 信息、serviceInstance 信息以及 Endpoint 信息</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> sourceServiceId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String sourceServiceName;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> sourceServiceInstanceId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String sourceServiceInstanceName;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> sourceEndpointId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String sourceEndpointName;
<span class="hljs-comment">// 下游系统的 service 信息、serviceInstance 信息以及 Endpoint 信息</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> destServiceId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String destServiceName;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> destServiceInstanceId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String destServiceInstanceName;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> destEndpointId;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> String destEndpointName;
<span class="hljs-comment">// 当前系统的组件类型</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> componentId;
<span class="hljs-comment">// 在当前系统中的耗时</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> latency;
<span class="hljs-comment">// 当前系统是否发生Error</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> status;
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> responseCode; <span class="hljs-comment">// 默认为0</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> RequestType type; <span class="hljs-comment">// 请求类型</span>
<span class="hljs-comment">// 调用关系中的角色，是调用方(Client)、被调用方(Server)还是代理(Proxy)</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> DetectPoint detectPoint; 
<span class="hljs-comment">// TraceSegment 起始时间所在分钟级时间窗口</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> timeBucket;
</code></pre>
<p>parseEntry() 方法解析 EntrySpan 中全部 TraceSegmentReference 的核心逻辑如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;记录&nbsp;TraceSegment 起始的分钟级时间窗口</span>
<span class="hljs-keyword">this</span>.minuteTimeBucket = segmentCoreInfo.getMinuteTimeBucket();
<span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; spanDecorator.getRefsCount(); i++) {
    <span class="hljs-comment">// 获取 TraceSegmentReference对应的的ReferenceDecorator</span>
    ReferenceDecorator reference = spanDecorator.getRefs(i);
    <span class="hljs-comment">// 创建对应的SourceBuilder</span>
    SourceBuilder sourceBuilder = <span class="hljs-keyword">new</span> SourceBuilder();
    <span class="hljs-comment">// 记录上游系统的 serviceId、serviceInstanceId 以及 endpointId</span>
&nbsp;&nbsp;&nbsp;&nbsp;sourceBuilder.setSourceEndpointId(reference.getParentEndpointId());
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;这里省略了针对&nbsp;MQ 组件的特殊处理，如果你感兴趣可以翻一下代码</span>
&nbsp;&nbsp;&nbsp;&nbsp;sourceBuilder.setSourceServiceInstanceId(reference.getParentServiceInstanceId());
&nbsp;&nbsp;&nbsp;&nbsp;sourceBuilder.setSourceServiceId(instanceInventoryCache.get(reference.getParentServiceInstanceId()).getServiceId());
    <span class="hljs-comment">// 记录下游(当前)系统的 serviceId、serviceInstanceId 以及 endpointId</span>
    sourceBuilder.setDestEndpointId(spanDecorator.getOperationNameId());
    sourceBuilder.setDestServiceInstanceId(segmentCoreInfo.getServiceInstanceId());
    sourceBuilder.setDestServiceId(segmentCoreInfo.getServiceId());
    sourceBuilder.setDetectPoint(DetectPoint.SERVER);
    <span class="hljs-comment">// 记录当前组件的类型</span>
    sourceBuilder.setComponentId(spanDecorator.getComponentId());
    <span class="hljs-comment">// 这里将解析 EntrySpan 和 ExitSpan 都会设置的一些公共信息封装到了setPublicAttrs中</span>
    setPublicAttrs(sourceBuilder, spanDecorator);
    <span class="hljs-comment">// 将 SourceBuilder 记录到 entrySourceBuilders 集合中</span>
    entrySourceBuilders.add(sourceBuilder);
}
</code></pre>
<h4>parseExit</h4>
<p>接下来，&nbsp;parseExit() 方法主要处理&nbsp;ExitSpan&nbsp;，其中会为每个 ExitSpan 创建相应的 SourceBuilder 对象，并将上游（当前）系统和下游系统的基础信息记录到其中，最后会将该 SourceBuilder 对象填充到&nbsp;exitSourceBuilders&nbsp;集合中，等待在 build() 方法中处理。</p>
<p>parseExit() 方法的实现逻辑与 parseEntry() 方法非常类似，大致如下：</p>
<pre><code data-language="java" class="lang-java">SourceBuilder sourceBuilder = <span class="hljs-keyword">new</span> SourceBuilder();
<span class="hljs-comment">// 记录下游系统的 serviceId、serviceInstanceId以及endpointId</span>
<span class="hljs-keyword">int</span> peerId = spanDecorator.getPeerId(); 
<span class="hljs-keyword">int</span> destServiceId = serviceInventoryCache.getServiceId(peerId);
<span class="hljs-keyword">int</span> mappingServiceId = serviceInventoryCache.get(destServiceId).getMappingServiceId();
<span class="hljs-comment">// 如果存在 mappingServiceId,则更新下游系统的 serviceId，mappingServiceId 相关讲解在后面介绍</span>
sourceBuilder.setDestServiceId(mappingServiceId);
<span class="hljs-keyword">int</span> destInstanceId = instanceInventoryCache.getServiceInstanceId(destServiceId, peerId);
sourceBuilder.setDestEndpointId(spanDecorator.getOperationNameId());
sourceBuilder.setDestServiceInstanceId(destInstanceId);
<span class="hljs-comment">// 记录当前(上游)系统的 serviceId、serviceInstanceId 以及 endpointId</span>
sourceBuilder.setSourceEndpointId(Const.USER_ENDPOINT_ID);
sourceBuilder.setSourceServiceInstanceId(segmentCoreInfo.getServiceInstanceId());
sourceBuilder.setSourceServiceId(segmentCoreInfo.getServiceId());
sourceBuilder.setDetectPoint(DetectPoint.CLIENT);
<span class="hljs-comment">//&nbsp;Client 角色</span>
sourceBuilder.setComponentId(spanDecorator.getComponentId());
setPublicAttrs(sourceBuilder, spanDecorator);
<span class="hljs-comment">//&nbsp;将 SourceBuilder 记录到 exitSourceBuilders 集合中</span>
exitSourceBuilders.add(sourceBuilder);
<span class="hljs-comment">//&nbsp;这里省略了对&nbsp;DB 慢查询的特殊处理</span>
</code></pre>
<p>在 SourceBuilder 中，除了填充上述 id 字段之外，还会记录这些 id 对应的字符串，这些字符串都是在 setPublicAttrs() 这个方法进行填充的，该方法在 parseEntry() 和 parseExit() 方法中都会出现，其核心实现代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setPublicAttrs</span><span class="hljs-params">(SourceBuilder sourceBuilder, SpanDecorator spanDecorator)</span> </span>{
    <span class="hljs-comment">// 计算 latency，即当前 Span 的时间跨度，在后面计算系统延迟的时候会用到</span>
    <span class="hljs-keyword">long</span> latency = spanDecorator.getEndTime() - spanDecorator.getStartTime();
    sourceBuilder.setLatency((<span class="hljs-keyword">int</span>)latency);
    <span class="hljs-comment">// 填充上下游系统中的 serviceName、serviceInstanceName 以及 endpointName</span>
    sourceBuilder.setSourceServiceName(serviceInventoryCache.get(sourceBuilder.getSourceServiceId()).getName());
    sourceBuilder.setSourceServiceInstanceName(instanceInventoryCache.get(sourceBuilder.getSourceServiceInstanceId()).getName());
    sourceBuilder.setSourceEndpointName(endpointInventoryCache.get(sourceBuilder.getSourceEndpointId()).getName());
    sourceBuilder.setDestServiceName(serviceInventoryCache.get(sourceBuilder.getDestServiceId()).getName());
    sourceBuilder.setDestServiceInstanceName(instanceInventoryCache.get(sourceBuilder.getDestServiceInstanceId()).getName());
    sourceBuilder.setDestEndpointName(endpointInventoryCache.get(sourceBuilder.getDestEndpointId()).getName());
}
</code></pre>
<p>另外，setPublicAttrs() 方法中还会 latency 耗时，这个比较重要，后面计算相应监控值时比较关键，会详细说明该值的作用。</p>
<h3>All 相关指标</h3>
<p>MultiScopesSpanListener.build() 方法主要负责将 entrySourceBuilders 和 exitSourceBuilders 两个集合中的全部 SourceBuilder 转换成相应的 Source 对象，然后交给 SourceReceiver 走 MetricsStreamProcessor 处理。</p>
<p>entrySourceBuilders 集合中的每个 SourceBuilder 对象都会生成多个 Source 对象，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR4CAWJP2AAQ7w0mOVog425.png" alt="Drawing 7.png"></p>
<p>AllDispatcher 会将一个 All 对象转换成多个 AllP*Metrics 对象，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR4eAUSGvAAIzGoF5axs249.png" alt="Drawing 8.png"></p>
<p>前文提到，Metrics 抽象类是所有监控类的父类，其中只记录了该监控数据所在的分钟级窗口（timeBucket 字段），&nbsp;前面介绍的 JVM 相关的监控类涉及&nbsp;SumMetrics 和&nbsp;LongAvgMetrics 两个抽象类，分别用于计算 sum 值和平均值，具体的分析不再展开。</p>
<p>这里的 AllP*Metrics 继承了PxxMetrics 抽象类，如下图所示，而 PxxMetrics 也继承了 Metrics 抽象类。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D4/CgqCHl7fR5KAF9wYAAI4moSl37k284.png" alt="Drawing 9.png"></p>
<p>PxxMetrics 的核心字段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;监控值的精度，默认是10毫秒级的监控</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = <span class="hljs-string">"precision"</span>) <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> precision;
<span class="hljs-comment">//&nbsp;记录当前监控在时间窗口内的全部数据，IntKeyLongValueArray 继承了 ArrayList，其中每个元素都是 IntKeyLongValue 类型</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = <span class="hljs-string">"detail_group"</span>) <span class="hljs-keyword">private</span> IntKeyLongValueArray detailGroup;
<span class="hljs-comment">// 计算之后的监控结果</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = <span class="hljs-string">"value"</span>, isValue = <span class="hljs-keyword">true</span>, function = Function.Avg) <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> value;
<span class="hljs-comment">// 分位数，例如，P99Metrics 中该字段值为 99，P90Metrics 中该字段值为90</span>
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> percentileRank;
<span class="hljs-comment">//&nbsp;用于合并相同监控值，其中 key 为监控值(降精度的)，value 记录了对应的IntKeyLongValue对象</span>
<span class="hljs-keyword">private</span> Map&lt;Integer, IntKeyLongValue&gt; detailIndex;
</code></pre>
<p>在 detailIndex 和 detailGroup 两个集合中的元素都是 IntKeyLongValue 类型，IntKeyLongValue 是一个KV结构，key 是降精度的监控值，value 是该监控值在当前时间窗口中出现的次数。</p>
<p>PxxMetrics 的核心是 combine() 方法，具体实现代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">combine</span><span class="hljs-params">(@SourceFrom <span class="hljs-keyword">int</span> value, @Arg <span class="hljs-keyword">int</span> precision)</span> </span>{
    <span class="hljs-keyword">this</span>.precision = precision; <span class="hljs-comment">// 确定监控精度，默认为10，即10毫秒级别</span>
    <span class="hljs-keyword">if</span> (detailIndex == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 初始化 detailIndex这个Map</span>
        detailIndex = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
        detailGroup.forEach(element -&gt; detailIndex.put(element.getKey(), element));
    }
    <span class="hljs-keyword">int</span> index = value / precision;
    IntKeyLongValue element = detailIndex.get(index);
    <span class="hljs-keyword">if</span> (element == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 创建 IntKeyLongValue 对象</span>
        element = <span class="hljs-keyword">new</span> IntKeyLongValue();
        element.setKey(index);
        element.setValue(<span class="hljs-number">1</span>);
        <span class="hljs-comment">// 记录到 detailGroup 和 detailIndex 集合中</span>
        detailGroup.add(element);
        detailIndex.put(element.getKey(), element);
    } <span class="hljs-keyword">else</span> {
        element.addValue(<span class="hljs-number">1</span>); <span class="hljs-comment">// 递增 value</span>
    }
}
</code></pre>
<p>下面通过一个示例介绍 combine() 方法的执行过程。如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D4/CgqCHl7fR5yAewjnAADQAPU9JCQ196.png" alt="Drawing 10.png"></p>
<p>假设此时的一个 AllP90Metrics 对象通过 combine() 方法合并另一个监控数据（latency=850ms, precision=10）时，会先进行降精度得到 index=85，然后在&nbsp;detailIndex 集合中未查找到对应的元素，最后新建 IntKeyLongValue 并记录到&nbsp;detailIndex 和 detailGroup 集合，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR6aAcH_jAALKBr8ccz8545.png" alt="Drawing 11.png"></p>
<p>接下来又收到一个监控数据（latency=300ms, precision=10），在通过 combine() 方法合并时，detailIndex.get(300/10) 可以查找到对应的 IntKeyLongValue 元素，直接递增其 value 即可，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR62ANtWZAAJLuUbjfPw407.png" alt="Drawing 12.png"></p>
<p>在开始介绍 PxxMetric 的 calculate() 方法实现之前，先来介绍一下 Pxx 这个指标的含义，这里以 P90 为例说明：计算&nbsp;P90 是在一个监控序列中，找到一个最小的值，大于序列中 90% 的监控值。</p>
<p>PxxMetric.calculate() 方法具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">calculate</span><span class="hljs-params">()</span> </span>{
    Collections.sort(detailGroup); <span class="hljs-comment">// 排序detailGroup</span>
    <span class="hljs-comment">// 计算该窗口监控点的总个数</span>
    <span class="hljs-keyword">int</span> total = detailGroup.stream().mapToInt(element -&gt; (<span class="hljs-keyword">int</span>)element.getValue()).sum();
    <span class="hljs-comment">// 查找指定分位数的位置</span>
    <span class="hljs-keyword">int</span> roof = Math.round(total * percentileRank * <span class="hljs-number">1.0f</span> / <span class="hljs-number">100</span>);
    <span class="hljs-keyword">int</span> count = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">for</span> (IntKeyLongValue element : detailGroup) {
        <span class="hljs-comment">// 累加监控点个数，直至到达(或超过)上面的 roof 值，此时的监控值即为指定分位数监控值</span>
        count += element.getValue(); 
        <span class="hljs-keyword">if</span> (count &gt;= roof) {
            value = element.getKey() * precision;
            <span class="hljs-keyword">return</span>;
        }
    }
}
</code></pre>
<p>这里接着上例进行分析，该 AllP90Metrics 在计算 P90 值的时候，会先根据监控值（即IntKeyLongValue 中的 key）对 detailGroup 进行排序（如下图所示），然后计算 roof 值得到 9 ，最后累加 count 找到 &gt;= roof 值的位置，此位置记录的监控值即为该时间窗口的 P90 值（即图中 100*10=1000ms）。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR8GARLdiAAEbWCXQjI0237.png" alt="Drawing 13.png"></p>
<p>PxxMetrics 的核心实现到这里就介绍完了，这也是 P<em>Metrics 抽象类以及 AllP</em>Metrics &nbsp;实现类的核心逻辑。</p>
<p>最后，根据前文对 Model 的介绍，每个 AllP*Metrics 在 ES 中有四个对应的 Index 名称分别是：</p>
<pre><code data-language="java" class="lang-java">all_p*-<span class="hljs-number">20200115</span>
all_p*_hour-<span class="hljs-number">20200115</span>
all_p*_day-<span class="hljs-number">20200115</span>
all_p*_month-<span class="hljs-number">202001</span>
</code></pre>
<p>前文提到，每个 Metrics 对象对应 Document 的 id 是由其&nbsp;id() 方法生成的，&nbsp;AllP*Metrics.id() 方法都是直接其所在的时间窗口：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">id</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> String.valueOf(getTimeBucket());
}
</code></pre>
<p>MetricsAggregateWorker 和 MetricsPersistentWorker 处理 Metrics 的时候，会根据 Map.containKey() 方法确定需要合并的 Metrics 对象，最终是通过 Metrics 实现的 hashCode() 和 equals() 方法进行比较的，多数 Metrics 实现的 equals() 方法比较的字段就是 id() 方法中用于构成 Document Id 的字段，例如：这里的&nbsp;AllP* Metrics.equals() 方法比较的是时间窗口（即 getTimeBucket() 方法的返回值），JVM 小节介绍的 InstanceJvmOldGcTimeMetrics 对应的 Document Id 是由 serviceInstanceId 和 timeBucket 两部分构成，equals() 方法也是比较这两个字段。</p>
<p>最后，我们以 AllP90Metrics 为例，看一下 AllP* Metrics 在 ElasticSearch 存储的 Document 内容是什么样子：</p>
<pre><code data-language="java" class="lang-java">{
&nbsp;&nbsp;"_index":&nbsp;"all_p90-20191209",&nbsp;&nbsp;# Index 名称
&nbsp;&nbsp;"_type":&nbsp;"type",
&nbsp;&nbsp;"_id":&nbsp;"201912091056",&nbsp;# Document Id
  "_version": 2, 
  "_score": 1,
&nbsp;&nbsp;"_source":&nbsp;{&nbsp;
    "precision": 10, # 精度，10ms 精度
    "time_bucket": 201912091056, # 时间窗口
&nbsp;&nbsp;&nbsp;&nbsp;"value":&nbsp;2010,&nbsp;&nbsp;&nbsp;#&nbsp;计算之后的value值，即 P90 值
&nbsp;&nbsp;&nbsp;&nbsp;"detail_group":&nbsp;"200,1|201,1"&nbsp;#&nbsp;该时间窗口内的全部监控数据(10ms精度)
  }
}
</code></pre>
<p>在 &nbsp;AllDispatcher 中除了生成多个 AllP*Metrics 对象之外，还会生成一个 AllHeatmapMetrics 对象。AllHeatmapMetrics 提供了热图（Heatmap）功能，热图的核心逻辑是在其父类 ThermodynamicMetrics 中实现的&nbsp;，其继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D5/CgqCHl7fR8uAUZkXAAAyr8M-ZaI851.png" alt="Drawing 14.png"></p>
<p>抽象类 ThermodynamicMetrics&nbsp;的核心逻辑是将监控点按照 latency 分为多个区间，每个区间的跨度是 100ms，总共 20 个区间，并统计每个区间中监控点的个数。</p>
<p>ThermodynamicMetrics 的核心字段与前面介绍的 PxxMetrics 类似，只不过 IntKeyLongValue 中的 key 值含义变成了区间编号：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = STEP) <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> step = <span class="hljs-number">0</span>; <span class="hljs-comment">// 每个区间的时间跨度，默认 100ms</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = NUM_OF_STEPS) <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> numOfSteps = <span class="hljs-number">0</span>; <span class="hljs-comment">// 区间总数，默认20</span>
<span class="hljs-comment">// 用于记录每个区间中监控点的个数，IntKeyLongValue 中，key 是区间编号，value 是该区间点的个数</span>
<span class="hljs-meta">@Getter</span> <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Column</span>(columnName = DETAIL_GROUP, isValue = <span class="hljs-keyword">true</span>) <span class="hljs-keyword">private</span> IntKeyLongValueArray detailGroup = <span class="hljs-keyword">new</span> IntKeyLongValueArray(<span class="hljs-number">30</span>);
<span class="hljs-comment">// key 是区间编号，value 是区间对应的 IntKeyLongValue 对象，用于快速查找区间数据</span>
<span class="hljs-keyword">private</span> Map&lt;Integer, IntKeyLongValue&gt; detailIndex;
</code></pre>
<p>ThermodynamicMetrics.combine() 方法的实现与 PxxMetrics 基本类似，calculate() 方法为空实现（因为 heatmap 结果已经在&nbsp;detailGroup 字段中存储了，无须单独计算）。这里不再展开分析，如果你感兴趣可以参考源码进行分析。</p>
<h3>Service 相关指标</h3>
<p>MultiScopesSpanListener.entrySourceBuilders 集合中的每个 SourceBuilder 元素还会生成 一个 Service 对象，在 ServiceDispatcher 中会将一个 Service 对象转换成多个 Metrics 对象，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR9aAdAr3AAIOl5wa4Bo103.png" alt="Drawing 15.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR92AfwMGAAAARmu_22A208.png" alt="Drawing 16.png"></p>
<p>ServiceP* Metrics 与前面介绍的 AllP* Metrics 类似，也继承了 PxxMetrics 抽象类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR-eAO-uWAAJvFQm4-p0823.png" alt="Drawing 17.png"></p>
<p>与 AllP*Metrics 的不同之处主要有两点：</p>
<ol>
<li>Index 名称。ServiceP*Metrics 对应的 Index 名称为：</li>
</ol>
<pre><code data-language="java" class="lang-java">service_p*-<span class="hljs-number">20200115</span>
service_p*_hour-<span class="hljs-number">20200115</span>
service_p*_day-<span class="hljs-number">20200115</span>
service_p*_month-<span class="hljs-number">20200115</span>
</code></pre>
<ol start="2">
<li>Document ID 结构。ServiceP*Metrics 对应的 Document Id 由 serviceId + timeBucket 两部分构成。显然，计算的维度不同，例如，ServiceP90Metrics 会将同一时间窗口内、同一 Service 的 P90 监控数据合并到一个 Document 中存储，而 AllP90Metrics 则是将同一时间窗口内的全部 Service 的 P90 监控数据合并到一个 Document 中存储。</li>
</ol>
<p>接下来看 ServiceCpmMetrics，它统计的是一个 Service 一分钟请求的次数，继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/C9/Ciqc1F7fR_WAHPZxAAB7CBwbRyA669.png" alt="Drawing 18.png"></p>
<p>SkyWalking 不仅会统计 Service 级别的 cpm 监控，还会统计 ServiceInstance 、Endpoint 以及 Relation 的 cpm 监控，它们都继承了 CPMMetrics 抽象类，CPMMetrics 有两个核心字段：一个是 total 字段（long 类型）记录了请求总量，在 combine() 方法中会累加该 total 字段；另一个是 value 字段（long 类型），它记录了 total / 分钟数的结果，该计算是在 calculate() 方法中完成的。</p>
<p>这就是所有 CPMMetrics 实现的核心逻辑，不同之处只在于 Index 名称以及 Document Id 的构成。这里，ServiceCpmMetrics 对应的 Index 为：</p>
<pre><code data-language="java" class="lang-java">service_cpm-<span class="hljs-number">20200115</span>
service_cpm_hour-<span class="hljs-number">20200115</span>
service_cpm_day-<span class="hljs-number">20200115</span>
service_cpm_month-<span class="hljs-number">202001</span>
</code></pre>
<p>其中 Document Id 也是由 ServiceId 以及 timeBucket 两部分构成（后面介绍的其他 ServiceSlaMetrics、ServiceRespTimeMetrics 皆是如此）。</p>
<p>ServiceSlaMetrics&nbsp;用于计算一个 Service 的&nbsp;Sla。Sla（Service level agreement，服务等级协议）对于互联网公司来说，其实就是一个服务可用性的保证。一般用百分比表示，该值越大，表示服务可用时间越长，服务更可靠，停机时间越短，反之亦然。</p>
<p>SkyWalking 除了会计算 Service 的 Sla 指标，还会计算 ServiceInstance、Endpoint 等的 Sla，如下图所示，它们都继承了 PercentMetrics 抽象类，PercentMetrics 实现了计算 Sla 的核心逻辑：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D5/CgqCHl7fSACAK908AAAARmu_22A884.png" alt="Drawing 19.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D5/CgqCHl7fSAqAI_LVAAE9CtYG6Z8919.png" alt="Drawing 20.png"></p>
<p>在 PercentMetrics 中的 total 字段（long 类型）记录了请求总量，match 字段（long 类型）记录了正常请求（EntrySpan.isError = true）的总量，combine() 方法合并两个 PercentMetrics 时，实际上是累加这两个字段。 percentage 字段记录了&nbsp;match *10000 / total 的结果值（即&nbsp;sla 值，该运算在 calculate() 方法中完成）。</p>
<p>ServiceSlaMetrics 对应的 Index 名称以及其中 Document Id 的格式与前面介绍的 ServiceCpmMetrics 类似，这里不再重复。</p>
<p>Service 相关的最后一个监控指标是 ServiceRespTimeMetrics ，用于统计 Service 的响应时间。ServiceRespTimeMetrics 与前面介绍的 InstanceJvmOldGcTimeMetrics 基本类似，继承了 LongAvgMetrics，其中记录了一个 Service 在一个时间窗口内的总耗时（summation 字段）以及请求个数（count，即监控点的个数），并计算平均值（value = summation / count）作为 Service 响应耗时。</p>
<p>ServiceRespTimeMetrics&nbsp;对应的 Index 名称以及其中 Document Id 的格式与前面介绍的 ServiceCpmMetrics 类似，这里不再重复。</p>
<h3>ServiceInstance 相关指标</h3>
<p>这里 ServiceInstance 相关的有 ServiceInstanceSlaMetrics、ServiceInstanceRespTimeMetrics、ServiceInstanceCpmMetrics&nbsp;三个指标，它们分别从 Sla、响应时间（Response Time）以及每分钟请求数（Cpm）三个角度去衡量一个&nbsp;ServiceInstance。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D5/CgqCHl7fSBSADOVYAADp1oyXZNI781.png" alt="Drawing 21.png"></p>
<p>这三种指标的计算方式在前面的 Service 相关指标小节已经详细介绍过了，这里不再重复。需要注意的是 Index 名称的变化，以及 Document Id 的变化（由 InstanceId + timebucket 两部分构成）。</p>
<h3>Endpoint 相关指标</h3>
<p>这里 Endpoint&nbsp;相关的有 EndpointSlaMetrics、EndpointAvgMetrics、EndpointCpmMetrics、EndpointP*Metrics 四类指标，它们分别从 Sla、平均响应时间、每分钟请求数（Cpm）以及多个分位数四个角度去衡量一个 Endpoint。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CA/Ciqc1F7fSCeAeOOCAADZWpkOhmE460.png" alt="Drawing 22.png"></p>
<p>这些指标的相关实现不再展开分析，上述 Endpoint 指标对应的 Document 中，除了记录对应监控数据本身，还会记录关联的 serviceId 和 serviceInstanceId。</p>
<h3>Relation 指标</h3>
<p>entrySourceBuilders 中的 SourceBuilder 对象会生成三个 Relation 类型的 Source 对象，Relation 表示的某种调用关系，例如：一个 ServiceRelation 对象记录了某两个 Service 之间的调用关系，一个 ServiceInstanceRelation 对象记录了某两个 ServiceInstance 之间的调用关系，一个 EndpointRelation 对象记录了某两个 Endpoint 之间的调用关系。</p>
<p>本课时将以 ServiceRelation 为例介绍 Relation 相关的指标，下图按照指标生成的位置分成了 Server 端和 Client 端两类：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CA/Ciqc1F7fSDCAdW4GAAKBRnkhCFc134.png" alt="Drawing 23.png"></p>
<p>其中，Server 类型指标产生的位置对应被调用系统（接收请求者）的 EntrySpan，而 Client 类型指标产生位置对应调用系统（发起请求者）的 ExitSpan，大致如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CA/Ciqc1F7fSDeAAHV4AAAARmu_22A770.png" alt="Drawing 24.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/CA/Ciqc1F7fSD6AbkzBAAEoxUOvcbE477.png" alt="Drawing 25.png"></p>
<p>这样的话，在 entrySourceBuilders 集合中的 SourceBuilder 生成的都是 Server 端的 Relaiton 指标，而无法生成 Client 端的 Relation 指标，因为 entrySourceBuilders 集合是 EntrySpan 的处理结果。同理可知 exitSourceBuilders 集合，只会生成 Client 端的 Relation 指标。</p>
<p>理清 Server 端和 Client 端两个分类之后，下面选取&nbsp;ServiceRelationServerCpmMetrics 为例继续分析，&nbsp;ServiceRelationServerCpmMetrics&nbsp;这个指标表示的是上游系统（sourceService）每分钟调用下游系统（destService）的次数，继承了 CPMMetrics，核心计算方式不必多说，对应的 Index 与前面介绍的格式也是类似的，关键在于其 Document Id 的格式：</p>
<pre><code data-language="java" class="lang-java">timebucket_sourceServiceId_destServiceId
</code></pre>
<p>通过 Document Id 可以明确知道该 Document 记录了哪个上游系统在哪一个时间段调用了哪个下游系统多少次。其他的 Relation 指标也是类似的，具体指标的计算方式在前面都介绍过了，这里不再重复。<br>
MultiScopesSpanListener.build() 方法对 entrySourceBuilders 集合的处理逻辑到这里就分析完了。exitSourceBuilders 集合中的 SourceBuilder 对象会生成的 Client 端的 Relation 监控指标，这里就不再展开分析了，如果你感兴趣可以参考代码进行分析。</p>
<h3>ServiceMappingSpanListener</h3>
<p>在 Agent 创建 ExitSpan 的时候，只知道下游系统的 peer 信息（remotePeer 或 peerId），在 MultiScopesSpanListener.parseExit()&nbsp; 方法中处理 ExitSpan 的时候，会通过下面这段代码将 peerId 转换成下游系统的 ServiceId：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;获取&nbsp;peerId&nbsp;在&nbsp;service_inventory&nbsp;中对应的&nbsp;id</span>
<span class="hljs-keyword">int</span>&nbsp;destServiceId&nbsp;=&nbsp;serviceInventoryCache.getServiceId(peerId);
<span class="hljs-comment">//&nbsp;在&nbsp;ServiceInventory&nbsp;中有一个 mappingServiceId 字段，记录了 peerId 关联的真的 Service 的Id</span>
<span class="hljs-keyword">int</span>&nbsp;mappingServiceId&nbsp;=&nbsp;serviceInventoryCache.get(destServiceId).getMappingServiceId();
</code></pre>
<p>将 peerId 与对应 Service 的 serviceId 进行关联的逻辑是在&nbsp;ServiceMappingSpanListener 中实现的。ServiceMappingSpanListener 的集成关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/D5/CgqCHl7fSEeAUq1LAAB7wN6gSQQ910.png" alt="Drawing 26.png"></p>
<p>在 ServiceMappingSpanListener.parseEntry() 方法中，会遍历 EntrySpan.refs 字段中记录的全部 SegmentReference。正如前面内容所示，TraceSegmentRef&nbsp;指向了上一个 TraceSegment，而上一个 TraceSegment 的基本信息会被封装成&nbsp;ContextCarrier 对象随着请求一起传递过来的。上游系统的&nbsp;Agent &nbsp;在&nbsp;ExitSpan 中创建 ContextCarrier 对象的时候，会将下游系统的 peerId（或是 peer 字符串）一起带上。这样的话，从&nbsp;SegmentReference 中拿到的 peerId&nbsp;即为当前服务暴露的地址对应的 peerId，将该 peerId 与当前服务的 ServiceId 的绑定即可。</p>
<p>ServiceMappingSpanListener.parseEntry() 方法的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">if</span> (spanDecorator.getRefsCount() &gt; <span class="hljs-number">0</span>) {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; spanDecorator.getRefsCount(); i++) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取&nbsp;peerId (即 addressId)对应的&nbsp;serviceId</span>
        <span class="hljs-keyword">int</span> serviceId = serviceInventoryCache.getServiceId(spanDecorator.getRefs(i).getNetworkAddressId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取&nbsp;addressId 关联的&nbsp;serviceId，如果发现不一致，则创建ServiceMapping 对象并记录下来，等待更新该关联关系</span>
        <span class="hljs-keyword">int</span> mappingServiceId = serviceInventoryCache.get(serviceId).getMappingServiceId();
        <span class="hljs-keyword">if</span> (mappingServiceId != segmentCoreInfo.getServiceId()) {
            ServiceMapping serviceMapping = <span class="hljs-keyword">new</span> ServiceMapping();
            serviceMapping.setServiceId(serviceId);
            <span class="hljs-comment">// 将 peerId (即addressId)与当前 serviceId 进行关联</span>
            serviceMapping.setMappingServiceId(segmentCoreInfo.getServiceId());
            serviceMappings.add(serviceMapping);
        }
    }
}
</code></pre>
<p>ServiceMappingSpanListener.build() 方法实现比较简单：它会根据 serviceMappings 集合的记录更新 service_inventory 索引中的映射关系，这里不再展开分析，如果你感兴趣可以参考源码进行分析。</p>

---

### 精选评论


