<p data-nodeid="16524">20 讲中我们引入了 Spring Boot Actuator 组件来满足 Spring Boot 应用程序的系统监控功能，并重点介绍了如何扩展常见的 Info 和 Health 监控端点的实现方法。</p>


<p data-nodeid="15878">这一讲我们继续讨论如何扩展 Actuator 端点，但更多关注与度量指标相关的内容。同时，我们还将给出如何创建自定义 Actuator 的实现方法，以便应对默认端点无法满足需求的应用场景。</p>
<h3 data-nodeid="15879">Actuator 中的度量指标</h3>
<p data-nodeid="16954" class=""><strong data-nodeid="16959">对于系统监控而言，度量是一个很重要的维度。</strong> 在 Spring Boot 2.X 版本中，Actuator 组件主要使用内置的 Micrometer 库实现度量指标的收集和分析。</p>

<h4 data-nodeid="15881">Micrometer 度量库</h4>
<p data-nodeid="15882">Micrometer 是一款监控指标的度量类库，为 Java 平台上的性能数据收集提供了一套通用的 API。在应用程序中，我们只使用 Micrometer 提供的通用 API 即可收集度量指标。</p>
<p data-nodeid="15883">下面我们先来简要介绍 Micrometer 中包含的几个核心概念。</p>
<p data-nodeid="15884">首先我们需要介绍的是计量器 Meter，它是一个接口，代表的是需要收集的性能指标数据。关于 Meter 的定义如下：</p>
<pre class="lang-java" data-nodeid="15885"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Meter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AutoCloseable</span> </span>{
&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//Meter 的唯一标识，是名称和标签的一种组合</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Id <span class="hljs-title">getId</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//一组测量结果</span>
	<span class="hljs-function">Iterable&lt;Measurement&gt; <span class="hljs-title">measure</span><span class="hljs-params">()</span></span>;
	&nbsp;
	<span class="hljs-comment">//Meter 的类型枚举值</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">enum</span> Type {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COUNTER,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; GAUGE,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LONG_TASK_TIMER,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; TIMER,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DISTRIBUTION_SUMMARY,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; OTHER
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="15886">通过上述代码，我们注意到 Meter 中存在一个 Id 对象，该对象的作用是定义 Meter 的名称和标签。从 Type 的枚举值中，我们不难看出 Micrometer 中包含的所有计量器类型。</p>
<p data-nodeid="15887">接下来我们先说明两个概念。</p>
<blockquote data-nodeid="18710">
<p data-nodeid="18711" class=""><strong data-nodeid="18721">Meter 的名称</strong>：对于计量器来说，每个计量器都有自己的名称，而且在创建时它们都可以指定一系列标签。<br>
<strong data-nodeid="18722">Meter 的标签</strong>：标签的作用在于监控系统可以通过这些标签对度量进行分类过滤。</p>
</blockquote>




<p data-nodeid="15892"><strong data-nodeid="15989">在日常开发过程中，常用的计量器类型主要分为计数器 Counter、计量仪 Gauge 和计时器 Timer 这三种。</strong></p>
<ul data-nodeid="15893">
<li data-nodeid="15894">
<p data-nodeid="15895"><strong data-nodeid="15994">Counter</strong>：这个计量器的作用和它的名称一样，就是一个不断递增的累加器，我们可以通过它的 increment 方法实现累加逻辑。</p>
</li>
<li data-nodeid="15896">
<p data-nodeid="15897"><strong data-nodeid="15999">Gauge</strong>：与 Counter 不同，Gauge 所度量的值并不一定是累加的，我们可以通过它的 gauge 方法指定数值。</p>
</li>
<li data-nodeid="15898">
<p data-nodeid="15899"><strong data-nodeid="16004">Timer</strong>：这个计量器比较简单，就是用来记录事件的持续时间。</p>
</li>
</ul>
<p data-nodeid="15900">既然我们已经明确了常用的计量器及其使用场景，那么如何创建这些计量器呢？</p>
<p data-nodeid="15901">在 Micrometer 中，我们提供了一个计量器注册表 MeterRegistry，它主要负责创建和维护各种计量器。关于 MeterRegistry 的创建方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="15902"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MeterRegistry</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AutoCloseable</span> </span>{
&nbsp;
    <span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> &lt;T&gt; <span class="hljs-function">Gauge <span class="hljs-title">newGauge</span><span class="hljs-params">(Meter.Id id, <span class="hljs-meta">@Nullable</span> T obj, ToDoubleFunction&lt;T&gt; valueFunction)</span></span>;
&nbsp;
    <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> Counter <span class="hljs-title">newCounter</span><span class="hljs-params">(Meter.Id id)</span></span>;
&nbsp;
    <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> Timer <span class="hljs-title">newTimer</span><span class="hljs-params">(Meter.Id id, DistributionStatisticConfig distributionStatisticConfig, PauseDetector pauseDetector)</span></span>;
…
}
</code></pre>
<p data-nodeid="15903">以上代码只是创建 Meter 的一种途径，从中我们可以看到 MeterRegistry 针对不同的 Meter 提供了对应的创建方法。</p>
<p data-nodeid="15904">而创建 Meter 的另一种途径是使用某个 Meter 的具体 builder 方法。以 Counter 为例，它的定义中包含了一个 builder 方法和一个 register 方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="15905"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Counter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Meter</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">static</span> Builder <span class="hljs-title">builder</span><span class="hljs-params">(String name)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Builder(name);
	}
	&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">void</span> <span class="hljs-title">increment</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; increment(<span class="hljs-number">1.0</span>);
	}
	&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">increment</span><span class="hljs-params">(<span class="hljs-keyword">double</span> amount)</span></span>;
	&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">double</span> <span class="hljs-title">count</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> Iterable&lt;Measurement&gt; <span class="hljs-title">measure</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.singletonList(<span class="hljs-keyword">new</span> Measurement(<span class="hljs-keyword">this</span>::count, Statistic.COUNT));
	}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Counter <span class="hljs-title">register</span><span class="hljs-params">(MeterRegistry registry)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> registry.counter(<span class="hljs-keyword">new</span> Meter.Id(name, tags, baseUnit, description, Type.COUNTER));
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="15906">注意到最后的 register 方法就是将当前的 Counter 注册到 MeterRegistry 中，因此我们需要创建一个 Counter。通常，我们会采用如下所示代码进行创建。</p>
<pre class="lang-java" data-nodeid="15907"><code data-language="java">Counter counter = Counter.builder(<span class="hljs-string">"counter1"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.tag(<span class="hljs-string">"tag1"</span>, <span class="hljs-string">"value1"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.register(registry);
</code></pre>
<p data-nodeid="15908">了解了 Micrometer 框架的基本概念后，接下来我们回到 Spring Boot Actuator，一起来看看它提供的专门针对度量指标管理的 Metrics 端点。</p>
<h4 data-nodeid="15909">扩展 Metrics 端点</h4>
<p data-nodeid="15910">在 Spring Boot 中，它为我们提供了一个 Metrics 端点用于实现生产级的度量工具。访问 actuator/metrics 端点后，我们将得到如下所示的一系列度量指标。</p>
<pre class="lang-xml" data-nodeid="15911"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"names":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.memory.max",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.threads.states",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jdbc.connections.active",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.gc.memory.promoted",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.memory.used",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.gc.max.data.size",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jdbc.connections.max",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jdbc.connections.min",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.memory.committed",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"system.cpu.count",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"logback.events",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"http.server.requests",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.buffer.memory.used",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.created",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.threads.daemon",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"system.cpu.usage",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.gc.memory.allocated",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.idle",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.pending",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jdbc.connections.idle",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.expired",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.threads.live",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.threads.peak",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.active",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.creation",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"process.uptime",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.rejected",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"process.cpu.usage",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.classes.loaded",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.max",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.min",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.gc.pause",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.classes.unloaded",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.active.current",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.alive.max",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.gc.live.data.size",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.usage",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.timeout",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.buffer.count",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"jvm.buffer.total.capacity",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tomcat.sessions.active.max",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"hikaricp.connections.acquire",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"process.start.time"
 &nbsp;&nbsp;&nbsp;&nbsp;]
 }
</code></pre>
<p data-nodeid="15912">以上代码中涉及的指标包括常规的系统内存总量、空闲内存数量、处理器数量、系统正常运行时间、堆信息等，也包含我们引入 JDBC 和 HikariCP 数据源组件之后的数据库连接信息等。此时，如果我们想了解某项指标的详细信息，在 actuator/metrics 端点后添加对应指标的名称即可。</p>
<p data-nodeid="15913">例如我们想了解当前内存的使用情况，就可以通过 actuator/metrics/jvm.memory.used 端点进行获取，如下代码所示。</p>
<pre class="lang-xml" data-nodeid="15914"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"name":"jvm.memory.used",
 &nbsp;&nbsp;&nbsp;&nbsp;"description":"The&nbsp;amount&nbsp;of&nbsp;used&nbsp;memory",
 &nbsp;&nbsp;&nbsp;&nbsp;"baseUnit":"bytes",
 &nbsp;&nbsp;&nbsp;&nbsp;"measurements":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"statistic":"VALUE",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"value":115520544
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;],
 &nbsp;&nbsp;&nbsp;&nbsp;"availableTags":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tag":"area",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"values":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"heap",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"nonheap"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"tag":"id",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"values":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Compressed&nbsp;Class&nbsp;Space",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PS&nbsp;Survivor&nbsp;Space",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PS&nbsp;Old&nbsp;Gen",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Metaspace",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"PS&nbsp;Eden&nbsp;Space",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"Code&nbsp;Cache"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;]
 }
</code></pre>
<p data-nodeid="22683" class="">前面介绍 Micrometer 时，我们已经提到 Metrics 指标体系中包含支持 Counter 和 Gauge 这两种级别的度量指标。通过将 <a href="https://github.com/spring-projects/spring-boot/blob/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/CounterService.java" data-nodeid="22687">Counter</a> 或 <a href="http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/GaugeService.java" data-nodeid="22691">Gauge </a> 注入业务代码中，我们就可以记录自己想要的度量指标。其中，<a href="https://github.com/spring-projects/spring-boot/blob/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/CounterService.java" data-nodeid="22695">Counter</a> 用来暴露 increment() 方法，而 <a href="http://github.com/spring-projects/spring-boot/tree/master/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/metrics/GaugeService.java" data-nodeid="22699">Gauge</a> 用来提供一个 value() 方法。</p>









<p data-nodeid="15916">下面我们以 Counter 为例介绍在业务代码中嵌入自定义 Metrics 指标的方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="15917"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CounterService</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CounterService</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Metrics.addRegistry(<span class="hljs-keyword">new</span> SimpleMeterRegistry());
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">counter</span><span class="hljs-params">(String name, String... tags)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Counter counter = Metrics.counter(name, tags);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; counter.increment();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="15918">在这段代码中，我们构建了一个公共服务 CounterService，并开放了一个 Counter 方法供业务系统进行使用。当然，你也可以自己实现类似的工具类完成对各种计量器的封装。</p>
<p data-nodeid="15919">另外，Micrometer 还提供了一个 MeterRegistry 工具类供我们创建度量指标。因此，我们也十分推荐使用 MeterRegistry 对各种自定义度量指标的创建过程进行简化。</p>
<h4 data-nodeid="15920">使用 MeterRegistry</h4>
<p data-nodeid="15921">再次回到 SpringCSS 案例，此次我们来到 customer-service 的 CustomerTicketService 中。</p>
<p data-nodeid="15922">比如我们希望系统每创建一个客服工单，就对所创建的工单进行计数，并作为系统运行时的一项度量指标，该效果的实现方式如下代码所示：</p>
<pre class="lang-java" data-nodeid="15923"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomerTicketService</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> MeterRegistry meterRegistry;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> CustomerTicket <span class="hljs-title">generateCustomerTicket</span><span class="hljs-params">(Long accountId, String orderNumber)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; CustomerTicket customerTicket = <span class="hljs-keyword">new</span> CustomerTicket();
&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; meterRegistry.summary(<span class="hljs-string">"customerTickets.generated.count"</span>).record(<span class="hljs-number">1</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> customerTicket;
&nbsp;&nbsp;&nbsp; }&nbsp;&nbsp; 
}
</code></pre>
<p data-nodeid="15924">在上述 generateCustomerTicket 方法中，通过 MeterRegistry 我们实现了每次创建 CustomerTicket 时自动添加一个计数的功能。</p>
<p data-nodeid="15925">而且，MeterRegistry 还提供了一些类工具方法用于创建自定义度量指标。这些类工具方法除了常规的 counter、gauge、timer 等对应具体 Meter 的工具方法之外，还包括上述代码中的 summary 方法，且 Summary 方法返回的是一个 DistributionSummary 对象，关于它的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="15926"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DistributionSummary</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Meter</span>, <span class="hljs-title">HistogramSupport</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">static</span> Builder <span class="hljs-title">builder</span><span class="hljs-params">(String name)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Builder(name);
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//记录数据</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">record</span><span class="hljs-params">(<span class="hljs-keyword">double</span> amount)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//记录操作执行的次数</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//记录数据的数量</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">double</span> <span class="hljs-title">totalAmount</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//记录数据的平均值</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">double</span> <span class="hljs-title">mean</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> count() == <span class="hljs-number">0</span> ? <span class="hljs-number">0</span> : totalAmount() / count();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//记录数据的最大值</span>
	<span class="hljs-function"><span class="hljs-keyword">double</span> <span class="hljs-title">max</span><span class="hljs-params">()</span></span>;
	…
}
</code></pre>
<p data-nodeid="15927">因为 DistributionSummary 的作用是记录一系列的事件并对这些事件进行处理，所以在 CustomerTicketService 中添加的meterRegistry.summary("customertickets.generated.count").record(1) 这行代码相当于每次调用 generateCustomerTicket 方法时，我们都会对这次调用进行记录。</p>
<p data-nodeid="15928">现在访问 actuator/metrics/customertickets.generated.count 端点，我们就能看到如下所示的随着服务调用不断递增的度量信息。</p>
<pre class="lang-xml" data-nodeid="15929"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp; "name":"customertickets.generated.count",
 &nbsp;&nbsp;&nbsp; "measurements":[
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "statistic":"Count",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "value":1
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; {
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "statistic":"Total",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "value":19
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
 &nbsp;&nbsp;&nbsp; ] 
 }
</code></pre>
<p data-nodeid="15930">显然，通过 MeterRegistry 实现自定义度量指标的使用方法更加简单。这里，你也可以结合业务需求尝试该类的不同功能。</p>
<p data-nodeid="15931">接下来我们再来看一个相对比较复杂的使用方式。在 customer-service 中，我们同样希望系统存在一个度量值，该度量值用于记录所有新增的 CustomerTicket 个数，这次的示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="15932"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomerTicketMetrics</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractRepositoryEventListener</span>&lt;<span class="hljs-title">CustomerTicket</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> MeterRegistry meterRegistry;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CustomerTicketMetrics</span><span class="hljs-params">(MeterRegistry meterRegistry)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.meterRegistry = meterRegistry;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onAfterCreate</span><span class="hljs-params">(CustomerTicket customerTicket)</span> </span>{ &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; meterRegistry.counter(<span class="hljs-string">"customerTicket.created.count"</span>).increment();&nbsp; 
	}
}
</code></pre>
<p data-nodeid="15933">首先，这里我们使用了 MeterRegistry 的 Counter 方法初始化一个 counter，然后调用它的 increment 方法增加度量计数（这部分内容我们已经很熟悉了）。</p>
<p data-nodeid="15934"><strong data-nodeid="16052">注意到这里，我们同时还引入了一个 AbstractRepositoryEventListener 抽象类，这个抽象类能够监控 Spring Data 中 Repository 层操作所触发的事件 RepositoryEvent，例如实体创建前后的 BeforeCreateEvent 和 AfterCreateEvent 事件、实体保存前后的 BeforeSaveEvent 和 AfterSaveEvent 事件等。</strong></p>
<p data-nodeid="15935">针对这些事件，AbstractRepositoryEventListener 能捕捉并调用对应的回调函数。关于 AbstractRepositoryEventListener 类的部分实现如下代码所示：</p>
<pre class="lang-java" data-nodeid="15936"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AbstractRepositoryEventListener</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">ApplicationListener</span>&lt;<span class="hljs-title">RepositoryEvent</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onApplicationEvent</span><span class="hljs-params">(RepositoryEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Class&lt;?&gt; srcType = event.getSource().getClass();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (event <span class="hljs-keyword">instanceof</span> BeforeSaveEvent) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; onBeforeSave((T) event.getSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event <span class="hljs-keyword">instanceof</span> BeforeCreateEvent) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; onBeforeCreate((T) event.getSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event <span class="hljs-keyword">instanceof</span> AfterCreateEvent) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; onAfterCreate((T) event.getSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event <span class="hljs-keyword">instanceof</span> AfterSaveEvent) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; onAfterSave((T) event.getSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
	}
}
</code></pre>
<p data-nodeid="15937">在这段代码中，我们可以看到 AbstractRepositoryEventListener 直接实现了 Spring 容器中的 ApplicationListener 监听器接口，并在 onApplicationEvent 方法中根据所传入的事件类型触发了回调函数。</p>
<p data-nodeid="15938">以案例中的需求场景为例，我们可以在创建 Account 实体之后执行度量操作。也就是说，可以把度量操作的代码放在 onAfterCreate 回调函数中，正如案例代码中所展示那样。</p>
<p data-nodeid="15939">现在我们执行生成客户工单操作，并访问对应的 Actuator 端点，同样可以看到度量数据在不断上升。</p>
<h3 data-nodeid="15940">自定义 Actuator 端点</h3>
<p data-nodeid="15941">在日常开发过程中，扩展现有端点有时并不一定能满足业务需求，而自定义 Spring Boot Actuator 监控端点算是一种更灵活的方法。</p>
<p data-nodeid="15942">假设我们需要提供一个监控端点以获取当前系统的用户信息和计算机名称，就可以通过一个独立的 MySystemEndPoint 进行实现，如下代码所示：</p>
<pre class="lang-java" data-nodeid="15943"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@Endpoint(id = "mysystem", enableByDefault=true)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MySystemEndpoint</span> </span>{ 
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@ReadOperation</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Map&lt;String, Object&gt; <span class="hljs-title">getMySystemInfo</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String,Object&gt; result= <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, String&gt; map = System.getenv();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(<span class="hljs-string">"username"</span>,map.get(<span class="hljs-string">"USERNAME"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(<span class="hljs-string">"computername"</span>,map.get(<span class="hljs-string">"COMPUTERNAME"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="15944">在这段代码中我们可以看到，MySystemEndpoint 主要通过系统环境变量获取所需监控信息。</p>
<p data-nodeid="15945">注意，这里我们引入了一个新的注解 @Endpoint，该注解定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="15946"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Endpoint {
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//端点 id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">id</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//是否默认启动标志位</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">enableByDefault</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>
<p data-nodeid="15947" class="">这段代码中的 @Endpoint 注解主要用于设置端点 id 及是否默认启动的标志位。且在案例中，我们指定了 id 为“mysystem”，enableByDefault 标志为 true。</p>
<p data-nodeid="15948">事实上，在 Actuator 中也存在一批类似 @Endpoint 的端点注解。其中被 @Endpoint 注解的端点可以通过 JMX 和 Web 访问应用程序，对应的被 @JmxEndpoint 注解的端点只能通过 JMX 访问，而被 @WebEndpoint 注解的端点只能通过 Web 访问。</p>
<p data-nodeid="15949">在示例代码中，我们还看到了一个 @ReadOperation 注解，该注解作用于方法，用于标识读取数据操作。<strong data-nodeid="16068">在 Actuator 中，除了提供 @ReadOperation 注解之外，还提供 @WriteOperation 和 @DeleteOperation 注解，它们分别对应写入操作和删除操作。</strong></p>
<p data-nodeid="23125" class="te-preview-highlight">现在，通过访问 <a href="http://localhost:8080/" data-nodeid="23129">http://localhost:8080/</a>actuator/mysystem，我们就能获取如下所示监控信息。</p>

<pre class="lang-xml" data-nodeid="15951"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"computername":"LAPTOP-EQB59J5P",
 &nbsp;&nbsp;&nbsp;&nbsp;"username":"user"
 }
</code></pre>
<p data-nodeid="15952">有时为了获取特定的度量信息，我们需要对某个端点传递参数，而 Actuator 专门提供了一个 @Selector 注解标识输入参数，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="15953"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@Endpoint(id = "account", enableByDefault = true)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AccountEndpoint</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AccountRepository accountRepository;&nbsp;&nbsp;&nbsp; 
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@ReadOperation</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Map&lt;String, Object&gt; <span class="hljs-title">getMySystemInfo</span><span class="hljs-params">(<span class="hljs-meta">@Selector</span> String arg0)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; result = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(accountName, accountRepository.findAccountByAccountName(arg0));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="15954">这段代码的逻辑非常简单，就是根据传入的 accountName 获取用户账户信息。</p>
<p data-nodeid="15955"><strong data-nodeid="16085">这里请注意，通过 @Selector 注解，我们就可以使用</strong><a href="http://localhost:8080/" data-nodeid="16081">http://localhost:8080/</a><strong data-nodeid="16086">actuator/ account/account1 这样的端口地址触发度量操作了。</strong></p>
<h3 data-nodeid="15956">小结与预告</h3>
<p data-nodeid="15957">度量是我们观测一个应用程序运行时状态的核心手段。这一讲我们介绍了 Spring Boot 中新引入的 Micrometer 度量库，以及该库中提供的各种度量组件。同时，我们还基于 Micrometer 中的核心工具类 MeterRegistry 完成了在业务系统中嵌入度量指标的实现过程。最后，我们还简要介绍了如何自定义一个 Actuator 端点的开发方法。</p>
<p data-nodeid="15958">这里给你留一道思考题：在使用 Micrometer 时，实现度量数据的采集方法有哪些？欢迎你在留言区进行互动、交流。另外，如果你觉得本专栏有价值，欢迎分享给更多好友哦~</p>
<p data-nodeid="15959">讲完度量，22 讲我们将关注对应用系统运行时状态的管理，并介绍如何使用 Admin Server 组件对 Spring Boot 应用程序进行有效管理。</p>

---

### 精选评论


