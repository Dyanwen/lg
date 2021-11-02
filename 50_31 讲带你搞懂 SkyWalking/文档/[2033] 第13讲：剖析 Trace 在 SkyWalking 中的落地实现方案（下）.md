<p style="line-height: 1.75em; text-align: justify;"><span></span></p>
<h1><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">TraceSegmentRef</span></p></h1>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">TraceSegment 中除了 Span 之外，还有另一个需要介绍的重要依赖 —— TraceSegmentRef，TraceSegment 通过 refs 集合记录父 TraceSegment 的信息，它的核心字段大概可以分为 3 类：</span></p>
<ul>
 <li><p><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">父 Span 信息</span></strong></p></li>
 <ul style="list-style-type: square;">
  <li><p><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">traceSegmentId（ID 类型）</span></strong></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：父 TraceSegment 的 ID。</span></p></li>
  <li><p><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">spanId（int 类型）</span></strong></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：父 Span 的 ID，与 traceSegmentId 结合就可以确定父 Span。</span></p></li>
  <li><p><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">type（SegmentRefType 类型）</span></strong></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：SegmentRefType 是个枚举，可选值有：CROSS_PROCESS、CROSS_THREAD，分别表示跨进程调用和跨线程调用。</span></p></li>
 </ul>
 <li><p><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">父应用（或者说，上游调用方）信息</span></strong></span></span></span></span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">peerId 和 peerHos</span></strong></span></span></span></span></span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">t</span></strong></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：父应用（即上游调用方）的地址信息。</span></p></li>
  <li><p><span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">parentServiceInstanceId（int 类型）</span></strong></span></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：父应用（即上游应用）的 ServiceInstanceId。</span></p></li>
  <li><p><span><span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">parentEndpointName 和 parentEndpointId</span></strong></span></span></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：父应用的（即上游应用）的 Endpoint 信息。</span></p></li>
 </ul>
 <li><p><span><span><span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">入口信息</span></strong></span></span></span></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">（在整条 Trace 中都会传递该信息)</span></p></li>
 <ul style="list-style-type: square;">
  <li><p><span><span><span><span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">entryServiceInstanceId</span></strong></span></span></span></span></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：入口应用的 ServiceInstanceId。</span></p></li>
  <li><p><span><span><span><span><span><span><span><span><span><span><span><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">entryEndpointName 和 entryEndpointId</span></strong></span></span></span></span></span></span></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">入口 Endpoint 信息。</span></p></li>
 </ul>
</ul>
<h1><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Context</span></p></h1>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">SkyWalking 中的每个 TraceSegment 都与一个 Context 上下文对象一对一绑定，Context 上下文不仅记录了 TraceSegment 的上下文信息，还提供了管理 TraceSegment 生命周期、创建 Span 以及跨进程（跨线程）传播相关的功能。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">AbstractTracerContext&nbsp;是对上下文概念的抽象，其中定义了 Context 上下文的基本行为：</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>inject(ContextCarrier) 方法</strong>：在跨进程调用之前，调用方会通过 inject() 方法将当前 Context 上下文记录的全部信息注入到 ContextCarrier 参数中，Agent 后续会将 ContextCarrier 序列化并随远程调用进行传播。ContextCarrier 的具体实现在后面会详细分析。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>extract(ContextCarrier)</strong><strong> </strong><strong>方法</strong>：跨进程调用的接收方会反序列化得到 ContextCarrier 对象，然后通过 extract() 方法从 ContextCarrier 中读取上游传递下来的 Trace 信息并记录到当前的 Context 上下文中。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>ContextSnapshot capture()</strong><strong> </strong><strong>方法</strong>：在跨线程调用之前，SkyWalking Agent 会通过 capture() 方法将当前 Context 进行快照，然后将快照传递给其他线程。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>continued(ContextSnapshot)</strong><strong> </strong><strong>方法</strong>：跨线程调用的接收方会从收到的 ContextSnapshot 中读取 Trace 信息并填充到当前 Context 上下文中。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>getReadableGlobalTraceId()</strong><strong> </strong><strong>方法</strong>： 用于获取当前 Context 关联的 TraceId。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>createEntrySpan()、createLocalSpan() 方法、createExitSpan()</strong><strong> </strong><strong>方法</strong>：用于创建 Span。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>activeSpan()</strong><strong> </strong><strong>方法：</strong>用于获得当前活跃的 Span。在 TraceSegment 中，Span 也是按照栈的方式进行维护的，因为 Span 的生命周期符合栈的特性，即：先创建的 Span 后结束。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>stopSpan(AbstractSpan)</strong><strong> </strong><strong>方法</strong>：用于停止指定 Span。</span></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">AbstractTraceContext 有两个实现类，如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/89/94/Cgq2xl6X_6-ATLgTAAApyYi2z4g447.png"></span></p>
<p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(51, 51, 51); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">IgnoredTracerContext 表示该 Trace 将会被丢失，所以其中不会记录任何信息，里面所有方法也都是空实现。这里重点来看 TracingContext，其核心字段如下：</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>samplingService（SamplingService</strong><strong> </strong><strong>类型）</strong>：负责完成 Agent 端的 Trace 采样，后面会展开介绍具体的采样逻辑。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>segment（TraceSegment</strong><strong> </strong><strong>类型）</strong>：它是与当前 Context 上下文关联的 TraceSegment 对象，在 TracingContext 的构造方法中会创建该对象。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>activeSpanStack（LinkedList&lt;AbstractSpan&gt;</strong><strong> </strong><strong>类型）</strong>：用于记录当前 TraceSegment 中所有活跃的 Span（即未关闭的 Span）。实际上 activeSpanStack 字段是作为栈使用的，TracingContext 提供了 push() 、pop() 、peek() 三个标准的栈方法，以及 first() 方法来访问栈底元素。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>spanIdGenerator（int</strong><strong> </strong><strong>类型）</strong>：它是 Span ID 自增序列，初始值为 0。该字段的自增操作都是在一个线程中完成的，所以无需加锁。</span></p></li>
</ul>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">管理 Span </span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">一般情况下，在 Agent 插件的前置处理逻辑中，会调用 createEntrySpan() 方法创建 EntrySpan，在 TracingContext 的实现中，会检测 EntrySpan 是否已创建，如果是，则不会创建新的 EntrySpan，只是重新调用一下其 start() 方法即可。TracingContext.createEntrySpan() 方法的大致实现如下：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<pre>public&nbsp;AbstractSpan&nbsp;createEntrySpan(final&nbsp;String&nbsp;operationName)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(isLimitMechanismWorking())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;前面提到过，默认配置下，每个TraceSegment只能放300个Span
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NoopSpan&nbsp;span&nbsp;=&nbsp;new&nbsp;NoopSpan();&nbsp;//&nbsp;超过300就放&nbsp;NoopSpan
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;push(span);&nbsp;//&nbsp;将Span记录到activeSpanStack这个栈中
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;AbstractSpan&nbsp;entrySpan;
&nbsp;&nbsp;&nbsp;&nbsp;final&nbsp;AbstractSpan&nbsp;parentSpan&nbsp;=&nbsp;peek();&nbsp;//&nbsp;读取栈顶Span，即当前Span
&nbsp;&nbsp;&nbsp;&nbsp;final&nbsp;int&nbsp;parentSpanId&nbsp;=&nbsp;parentSpan&nbsp;==&nbsp;null&nbsp;?&nbsp;-1&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parentSpan.getSpanId();
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(parentSpan&nbsp;!=&nbsp;null&nbsp;&amp;&amp;&nbsp;parentSpan.isEntry())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;更新&nbsp;operationId(省略operationName的处理逻辑)，省略
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;EndpointNameDictionary&nbsp;的处理，其核心逻辑在前面的小节已经介绍过了。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;entrySpan&nbsp;=&nbsp;parentSpan.setOperationId(operationId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;重新调用&nbsp;start()方法，前面提到过，start()方法会重置
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;operationId(以及或operationName)之外的其他字段
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;entrySpan.start();
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;else&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;新建&nbsp;EntrySpan对象，spanIdGenerator生成Span&nbsp;ID并递增
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;entrySpan&nbsp;=&nbsp;new&nbsp;EntrySpan(spanIdGenerator++,&nbsp;parentSpanId,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;operationId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;调用&nbsp;start()方法，第一次调用start()方法时会设置startTime
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;entrySpan.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;将新建的Span添加到activeSpanStack栈的栈顶
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;push(entrySpan);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">前面通过 demo-webapp 示例介绍了多次调用 EntrySpan.start() 方法中栈相关的概念，这里依旧通过 demo-webapp 示例简单介绍一下 activeSpanStack 这个栈的工作原理，示例 Trace 如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/03/50/CgoCgV6X_7CAIrmBAABEHpBozXI630.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">当请求经过 Tomcat 插件时会创建 EntrySpan（调用 start() 方法）并入栈到 activeSpanStack 中；请求经过 Spring MVC 插件时不会创建新的 EntrySpan，只会重新调用 start() 方法。接下来在调用 first() 方法时会创建相应的 LocalSpan 并入栈，first() 方法调用结束之后会将该 LocalSpan 出栈；调用 second() 方法时与 Span 出入栈逻辑相同；最后在通过 Dubbo 远程调用 HelloService.say() 方法的时候，会创建相应的 ExitSpan 并入栈，结束 Dubbo 调用之后其相应的 ExitSpan 会出栈，此时整个 activeSpanStack 栈空了，TraceSegment 也就结束了。整个过程如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/10/7E/Ciqah16X_7CADwCpAAQ3K_wSU4k128.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(51, 51, 51); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">createLocalSpan() 方法负责创建 LocalSpan 对象并添加到 activeSpanStack 集合中，LocalSpan 的 start() 方法中没有栈的概念，存在多次调用的情况，只在这里调用一次即可。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(51, 51, 51); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">createExitSpan() 方法负责创建 ExitSpan，与 createEntrySpan() 方法类似：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(51, 51, 51); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>public&nbsp;AbstractSpan&nbsp;createExitSpan(String&nbsp;operationName,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;remotePeer)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;AbstractSpan&nbsp;exitSpan;
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;从activeSpanStack栈顶获取当前Span
&nbsp;&nbsp;&nbsp;&nbsp;AbstractSpan&nbsp;parentSpan&nbsp;=&nbsp;peek();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(parentSpan&nbsp;!=&nbsp;null&nbsp;&amp;&amp;&nbsp;parentSpan.isExit())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;当前Span已经是ExitSpan，则不再新建ExitSpan，而是调用其start()方法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exitSpan&nbsp;=&nbsp;parentSpan;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;else&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;当前Span不是&nbsp;ExitSpan，就新建一个ExitSpan
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;final&nbsp;int&nbsp;parentSpanId&nbsp;=&nbsp;parentSpan&nbsp;==&nbsp;null&nbsp;?&nbsp;-1&nbsp;:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parentSpan.getSpanId();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;exitSpan&nbsp;=&nbsp;&nbsp;new&nbsp;ExitSpan(spanIdGenerator++,&nbsp;parentSpanId,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;operationId,&nbsp;peerId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;push(exitSpan);&nbsp;//&nbsp;将新建的ExitSpan入栈
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;exitSpan.start();//&nbsp;调用start()方法
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;exitSpan;
}</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">了解了 TracingContext 创建以及维护 3 类 Span 的实现之后，我们来看关闭 Span 的方法 —— stopSpan() 方法，它会将当前 </span>activeSpanStack <span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">栈顶的 Span 关闭并出栈，同时在整个 activeSpanStack 栈空了之后，会尝试关闭当前 TraceSegment，具体实现如下：</span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<pre>public&nbsp;boolean&nbsp;stopSpan(AbstractSpan&nbsp;span)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;AbstractSpan&nbsp;lastSpan&nbsp;=&nbsp;peek();&nbsp;//&nbsp;获取当前栈顶的Span对象
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(lastSpan&nbsp;==&nbsp;span)&nbsp;{&nbsp;//&nbsp;只能关闭当前活跃Span对象，否则抛异常
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(lastSpan&nbsp;instanceof&nbsp;AbstractTracingSpan)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(lastSpan.finish(segment))&nbsp;{&nbsp;//&nbsp;尝试关闭Span
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//当Span完全关闭之后，会将其出栈(即从activeSpanStack中删除）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pop();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;else&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pop();&nbsp;//&nbsp;针对NoopSpan类型Span的处理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;else&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;throw&nbsp;new&nbsp;IllegalStateException("Stopping&nbsp;the&nbsp;unexpected...");
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;TraceSegment中全部Span都关闭(且异步状态的Span也关闭了)，则当前
&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;TraceSegment也会关闭，该关闭会触发TraceSegment上传操作，后面详述
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(checkFinishConditions())&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;finish();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;activeSpanStack.isEmpty();
}</pre>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">跨进程(跨线程)传播</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在开始介绍 Context 与跨进程传播相关的实现之前，需要先介绍一下它们的参数 —— ContextCarrier。从类名就可以看出 ContextCarrier 是 Context 上下文的搬运工（Carrier），它实现了 Serializable 接口，负责在进程之间搬运 TracingContext 的一些基本信息，跨进程调用涉及 Client 和 Server 两个系统，所以 ContextCarrier 中的字段 Client 和 Server 含义不同：</span></p>
<ul>
 <li><p><span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>traceSegmentId（ID 类型）</strong></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：它记录了 Client 中 TraceSegment ID；从 Server 角度看，记录的是父 TraceSegment 的 ID。</span></p></li>
 <li><p><span><span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>spanId（int 类型）</strong></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：从 Client 角度看，它记录了当前 ExitSpan 的 ID；从 Server 角度，看记录的是父 Span ID。</span></p></li>
 <li><p><span><span><span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>parentServiceInstanceId（int 类型）</strong></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：它记录的是 Client 服务实例的 ID。</span></p></li>
 <li><p><span><span><span><span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>peerHost（String 类型）</strong></span></span></span></span></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">：它记录了 Server 端的地址（这里 peerName 和 peerId 共用了同一个字段）。以 "#" 开头时记录的是 peerName，否则记录的是 peerId，在 inject() 方法（或 extract() 方法）中填充（或读取）该字段时会专门判断处理开头的"#"字符。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>entryEndpointName（String</strong><strong> </strong><strong>类型）</strong>：它记录整个 Trace 的入口 EndpointName，该值在整个 Trace 中传播。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>parentEndpointName（String</strong><strong> </strong><strong>类型）</strong>：它记录了 Client &nbsp;入口 EndpointName（或 EndpointId）。以 "#" 开头的时候，记录的是 EndpointName，否则记录的是 EndpointId。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>primaryDistributedTraceId（DistributedTraceId</strong><strong> </strong><strong>类型）</strong>：它记录了当前 Trace ID。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>entryServiceInstanceId（int</strong><strong> </strong><strong>类型）</strong>：它记录了当前 Trace 的入口服务实例 ID。</span></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">跨进程传播 Context 上下文信息的核心流程大致为：远程调用的 Client 端会调用 inject(ContextCarrier) 方法，将当前 TracingContext 中记录的 Trace 上下文信息填充到传入的 ContextCarrier 对象。后续 Client 端的插件会将 ContextCarrier 对象序列化成字符串并将其作为附加信息添加到请求中，这样，ContextCarrier 字符串就会和请求一并到达 Server 端。Server 端的入口插件会检查请求中是否携带了 ContextCarrier 字符串，如果存在 ContextCarrier 字符串，就会将其进行反序列化，然后调用 extract() 方法从 ContextCarrier 对象中取出 Context 上下文信息，填充到当前 TracingContext（以及 TraceSegmentRef) 中。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">例如在 demo-webapp 和 demo-provider 的示例中，ContextCarrier 的传播过程如图所示，序列化之后的 ContextCarrier 字符串会放到 RpcContext 中：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/89/95/Cgq2xl6X_7CATU_aAAFxCpVbciQ707.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这里需要深入介绍一下 ContextCarrier 序列化之后的格式，具体实现在其 serialize() 方法中：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<pre>//&nbsp;有多个版本的结构，这里只关注最新的V2版本
String&nbsp;serialize(HeaderVersion&nbsp;version)&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;return&nbsp;StringUtil.join('-',&nbsp;"1",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base64.encode(this.getPrimaryDistributedTraceId().encode()),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base64.encode(this.getTraceSegmentId().encode()),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.getSpanId()&nbsp;+&nbsp;"",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.getParentServiceInstanceId()&nbsp;+&nbsp;"",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.getEntryServiceInstanceId()&nbsp;+&nbsp;"",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base64.encode(this.getPeerHost()),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base64.encode(this.getEntryEndpointName()),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Base64.encode(this.getParentEndpointName()));
}</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">ContextCarrier 序列化之后得到的字符串分为 9 个部分，每个部分通过"-"（中划线）连接。在 deserialize() 方法中实现了 ContextCarrier 反序列化的逻辑，即将上述字符串进行切分并赋值到对应的字段中，具体逻辑为 serialize() 方法的逆操作，这里不再展开分析。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面来看 TracingContext 对跨线程传播的支持，这里涉及 capture() 方法和 continued() 方法。跨线程传播时使用 ContextSnapshot 为 Context 上下文创建快照，因为是在一个 JVM 中，所以 ContextSnapshot 不涉及序列化的问题，也无需携带服务实例 ID 以及 peerHost 信息，其他核心字段与 ContextCarrier 类似，这里不再展开介绍。</span></p>
<h1><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总结</span></p></h1>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">这个课时我们主要学习了 SkyWalking 对 Trace 基本概念的实现，首先介绍了 Trace ID 的实现结构，之后分析了 TraceSegment 如何维护底层 Span 集合以及父子关系，接下来深入剖析了 3 </span><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(41, 50, 57);">种类型的 Span 以及 </span>StackBasedTracingSpan 引入的<span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(41, 50, 57);">栈的概念。</span>最后剖析了与 TraceSegment 相对应的 TracingContext 的实现，它管理着 3 类 Span 的生命周期，提供了跨进程/跨线程传播的基本方法。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在后面的课时中，我们将深入学习与 Trace 相关的 BootService 实现，分析 SkyWalking Agent 如何在这些基础组件上有条不紊的收集并发送 Trace 数据。</span></p>
<p></p>

---

### 精选评论


