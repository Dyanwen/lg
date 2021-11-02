<p data-nodeid="5041">当系统在运行出现问题时，进行错误排查的首要目标是系统的日志，日志在系统维护中的重要性不言而喻。与单体应用相比，微服务架构应用的每个服务都独立运行，会产生各自的日志。这就要求把来自不同服务的日志记录聚合起来，形成统一的查询视图。云原生应用运行在 Kubernetes 上，对日志记录有不同的要求。本课时将介绍微服务架构的云原生应用，如何使用 Fluentd、ElasticSearch 和 Kibana 来管理日志。</p>
<h3 data-nodeid="5042">记录日志</h3>
<p data-nodeid="5043"><strong data-nodeid="5286">日志记录</strong>是开发中的重要组成部分，这离不开日志库的支持。</p>
<h4 data-nodeid="5044">日志库</h4>
<p data-nodeid="5045">在 Java 平台上，直到 JDK 1.4 版本才在标准库中增加了日志记录的 API，也就是 java.util.logging 包（JUL）。在那之前已经有一些开源日志实现流行起来，如 Apache Log4j，这就造成了在目前的 Java 日志实现中，Java 标准库的 JUL 包的使用者较少，而Log4j 和 Logback 这样的开源实现反而比较流行。</p>
<p data-nodeid="5046">几乎所有的应用和第三方库都需要用到日志的功能，而且可以自由选择所使用的日志实现库，每个日志库都有自己特定的配置方式。当不同的日志实现同时使用时，它们的配置没办法统一起来，还可能产生冲突，这就产生了 Java 平台上特殊的日志 API 抽象层。</p>
<p data-nodeid="5047">日志 API 抽象层（Facade）提供了一个抽象的接口来访问日志相关的功能，不同的日志库都实现该抽象层的接口，从而允许在运行时切换不同的具体日志实现。对于共享库的代码，推荐使用日志抽象层的 API，这就保证了共享库的使用者在选择日志实现时的灵活性。</p>
<p data-nodeid="5048">常用的抽象层库包括早期流行的 <a href="https://commons.apache.org/proper/commons-logging/" data-nodeid="5294">Apache Commons Logging</a> 和目前最常用的 <a href="http://www.slf4j.org/" data-nodeid="5298">SLF4J</a>。日志实现库负责完成实际的日志记录，常用的库包括 Java 标准库提供的 JUL、<a href="http://logging.apache.org/log4j/2.x" data-nodeid="5302">Log4j</a> 和 <a href="http://logback.qos.ch/" data-nodeid="5306">Logback</a> 等。在一般的应用开发中，通常使用日志抽象层加上具体日志实现库的方式。</p>
<p data-nodeid="5049">如果使用 Log4j 2 作为具体的日志实现，那么通常需要用到下表中给出的 3 个 Maven 库。</p>
<table data-nodeid="5051">
<thead data-nodeid="5052">
<tr data-nodeid="5053">
<th data-org-content="**分组**" data-nodeid="5055"><strong data-nodeid="5312">分组</strong></th>
<th data-org-content="**Artifact 名称**" data-nodeid="5056"><strong data-nodeid="5316">Artifact 名称</strong></th>
<th data-org-content="**作用**" data-nodeid="5057"><strong data-nodeid="5320">作用</strong></th>
</tr>
</thead>
<tbody data-nodeid="5061">
<tr data-nodeid="5062">
<td data-org-content="org.slf4j" data-nodeid="5063">org.slf4j</td>
<td data-org-content="slf4j-api" data-nodeid="5064">slf4j-api</td>
<td data-org-content="SLF4J 提供的日志 API" data-nodeid="5065">SLF4J 提供的日志 API</td>
</tr>
<tr data-nodeid="5066">
<td data-org-content="org.apache.logging.log4j" data-nodeid="5067">org.apache.logging.log4j</td>
<td data-org-content="log4j-slf4j-impl" data-nodeid="5068">log4j-slf4j-impl</td>
<td data-org-content="Log4j 2 与 SLF4J API 的适配器" data-nodeid="5069">Log4j 2 与 SLF4J API 的适配器</td>
</tr>
<tr data-nodeid="5070">
<td data-org-content="org.apache.logging.log4j" data-nodeid="5071">org.apache.logging.log4j</td>
<td data-org-content="log4j-core" data-nodeid="5072">log4j-core</td>
<td data-org-content="Log4j 2 的具体实现" data-nodeid="5073">Log4j 2 的具体实现</td>
</tr>
</tbody>
</table>
<p data-nodeid="5074">对于 Spring Boot 应用来说，只需要选择添加下面列表中给出的依赖即可。</p>
<table data-nodeid="5076">
<thead data-nodeid="5077">
<tr data-nodeid="5078">
<th data-org-content="**Spring Boot 依赖名称**" data-nodeid="5080"><strong data-nodeid="5334">Spring Boot 依赖名称</strong></th>
<th data-org-content="**日志实现**" data-nodeid="5081"><strong data-nodeid="5338">日志实现</strong></th>
</tr>
</thead>
<tbody data-nodeid="5084">
<tr data-nodeid="5085">
<td data-org-content="spring-boot-starter-log4j2" data-nodeid="5086">spring-boot-starter-log4j2</td>
<td data-org-content="Log4j 2" data-nodeid="5087">Log4j 2</td>
</tr>
<tr data-nodeid="5088">
<td data-org-content="spring-boot-starter-logging" data-nodeid="5089">spring-boot-starter-logging</td>
<td data-org-content="Logback" data-nodeid="5090">Logback</td>
</tr>
</tbody>
</table>
<p data-nodeid="5091">在应用开发中，可以选择使用 SLF4J 的 API 来记录日志，也可以直接使用某个具体日志实现的 API。使用 SLF4J API 的好处是避免了供应商锁定的问题，与其他第三方库一块使用时不容易产生冲突，不足之处是 SLF4J 的 API 为了保证更广泛的兼容性，其 API 只是提供了最通用的功能，无法使用具体日志实现特有的功能。</p>
<p data-nodeid="5092">在开发共享库时，建议使用 SLF4J 的 API 以提高兼容性；在应用的开发中，一般很少会出现替换日志实现的情况，因此可以选择直接使用日志实现的 API。以 Log4j 2 为例，它提供了对 SLF4J 等其他日志 API 的适配器。即便直接使用 Log4j 2 的 API，也可以通过适配器与其他日志实现库进行交互。</p>
<h4 data-nodeid="5093">日志记录器</h4>
<p data-nodeid="5094">日志 API 的使用者通过记录器（Logger）来发出日志记录请求，并提供日志的内容。在记录日志时，需要指定日志的严重性级别，日志记录 API 都提供了相应的工厂方法来创建记录器对象，每个记录器对象都有名称。一般的做法是使用当前 Java 类的名称或所在包的名称来作为记录器对象的名称。记录器的名称通常是具有层次结构的，与 Java 包的层次结构相对应。</p>
<p data-nodeid="5095">在通过日志记录器对象记录日志时，需要指定日志的严重性级别。根据每个记录器对象的不同配置，低于某个级别的日志消息可能不会被记录下来，该级别是日志 API 的使用者根据日志记录中所包含的信息来自行决定的。当通过记录器对象来记录日志时，只是发出一个日志记录请求，该请求是否会完成取决于请求和记录器对象的严重性级别。记录器使用者产生的低于记录器对象严重性级别的日志消息不会被记录下来，这样的记录请求会被忽略。一般来说，对于 DEBUG 及其以下级别的日志消息，首先需要使用类似 isDebugEnabled 这样的方法来检查日志消息是否会被记录，如下面的代码所示。</p>
<pre class="lang-java" data-nodeid="5096"><code data-language="java"><span class="hljs-keyword">if</span> (LOGGER.isDebugEnabled()) { 
&nbsp;&nbsp;&nbsp;LOGGER.debug(<span class="hljs-string">"This is a debug message."</span>); 
}
</code></pre>
<p data-nodeid="5097">日志记录在产生之后以事件的形式来表示。<strong data-nodeid="5362">输出源（Appender）<strong data-nodeid="5361">负责把日志事件传递到不同的目的地，常用的日志目的地包括文件、控制台、数据库、HTTP 服务和 syslog 等。其中</strong>控制台</strong>和<strong data-nodeid="5363">文件</strong>是最常用的两种，控制台输出用在开发中，滚动文件（Rolling File）在生产环境中用来保存历史日志记录。</p>
<p data-nodeid="5098">在输出日志事件到目的地之前，通常需要对事件进行格式化，这是通过布局（Layout）来完成的。布局负责把事件转换成输出源所需要的格式，常用的布局格式包括字符串、JSON、XML、CSV、HTML 和 YAML 等。</p>
<p data-nodeid="5099">过滤器（Filter）的作用是对日志事件进行过滤，以确定日志事件是否需要被发布。过滤器可以添加在日志记录器或输出源上。</p>
<h4 data-nodeid="5100">MDC 和 NDC</h4>
<p data-nodeid="5101">在多线程和多用户的应用中，同样的代码会处理不同用户的请求。在记录日志时，应该包含与用户相关的信息，当某个用户出现问题时，可以通过用户的标识符在日志中快速查找相关的记录，更方便定位问题。在日志记录中，<strong data-nodeid="5376">映射调试上下文</strong>（Mapped Diagnostic Context，MDC）和<strong data-nodeid="5377">嵌套调试上下文</strong>（Nested Diagnostic Context，NDC）解决了这个问题。正如名字里面所指出的一样，MDC 和 NDC 最早是为了错误调试的需要而引入的，不过现在一般作为通用的数据存储方式。MDC 和 NDC 在实现和作用上是相似，只不过 MDC 用的是哈希表，而 NDC 用的是栈，因此 NDC 中只能包含一个值。MDC 和 NDC 使用 ThreadLocal 来实现，与当前线程绑定。</p>
<p data-nodeid="5102">由于 MDC 比 NDC 更灵活，实际中一般使用 MDC 较多，SLF4J 的 API 提供了对 MDC 和 NDC 的支持。同一个线程中运行的不同代码，可以通过 MDC 来共享数据。以 REST API 为例，当用户通过认证之后，可以在 Spring Security 过滤器的实现中把已认证用户的标识符保存在 MDC 中，后续的代码都可以从 MDC 中获取用户的标识符，而不用通过方法调用时的参数来传递。</p>
<p data-nodeid="5103">MDC 类中包含了对哈希表进行操作的静态方法，如 get、put、remove 和 clear 等。大部分时候把 MDC 当成一个哈希表来使用即可，如下面的代码所示。</p>
<pre class="lang-java" data-nodeid="5104"><code data-language="java">MDC.put(<span class="hljs-string">"value"</span>, <span class="hljs-string">"hello"</span>);
LOGGER.info(<span class="hljs-string">"MDC value : {}"</span>, MDC.get(<span class="hljs-string">"value"</span>));
</code></pre>
<p data-nodeid="5105">由于 MDC 保存在 ThreadLocal 中，如果当前线程通过 Java 中的 ExecutorService 来提交任务，任务的代码由工作线程来运行，有可能无法获取到 MDC 的值。这个时候就需要手动传递 MDC 中的值。</p>
<p data-nodeid="5106">在下面的代码中，首先使用 MDC.getCopyOfContextMap 方法获取到当前线程的 MDC 中数据的拷贝，在任务的代码中使用 MDC.setContextMap 方法来设置 MDC 的值。通过这种方式，可以在不同线程之间传递 MDC。</p>
<pre class="lang-java" data-nodeid="5107"><code data-language="java"><span class="hljs-keyword">final</span> ExecutorService executor = Executors.newSingleThreadExecutor();
<span class="hljs-keyword">final</span> Map&lt;String, String&gt; contextMap = MDC.getCopyOfContextMap();
<span class="hljs-keyword">try</span> {
&nbsp; executor.submit(() -&gt; {
&nbsp; &nbsp; MDC.setContextMap(contextMap);
&nbsp; &nbsp; <span class="hljs-keyword">new</span> MDCGetter().display();
&nbsp; }).get();
} <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> InterruptedException | ExecutionException e) {
&nbsp; e.printStackTrace();
}
executor.shutdown();
</code></pre>
<p data-nodeid="5108">NDC 在使用时更加简单一些，只有 push 和 pop 两个方法，分别进行进栈和出栈操作。NDC 的 API 在 slf4j-ext 库中，其内部实现时实际上使用的是 MDC。</p>
<p data-nodeid="5109">MDC 和 NDC 中的值，除了直接在代码中使用之外，还可以在模式布局中使用，从而出现在日志记录中。在 Log4j 2 中，模式布局支持不同的参数来引用 MDC 和 NDC 的值，如下表所示。</p>
<table data-nodeid="5111">
<thead data-nodeid="5112">
<tr data-nodeid="5113">
<th data-org-content="**参数**" data-nodeid="5115"><strong data-nodeid="5387">参数</strong></th>
<th data-org-content="**说明**" data-nodeid="5116"><strong data-nodeid="5391">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="5119">
<tr data-nodeid="5120">
<td data-org-content="%X" data-nodeid="5121">%X</td>
<td data-org-content="MDC 中的全部值" data-nodeid="5122">MDC 中的全部值</td>
</tr>
<tr data-nodeid="5123">
<td data-org-content="%X{key}" data-nodeid="5124">%X{key}</td>
<td data-org-content="MDC 中特定键对应的值" data-nodeid="5125">MDC 中特定键对应的值</td>
</tr>
<tr data-nodeid="5126">
<td data-org-content="%x" data-nodeid="5127">%x</td>
<td data-org-content="NDC 中的值" data-nodeid="5128">NDC 中的值</td>
</tr>
</tbody>
</table>
<p data-nodeid="5129">需要注意的是，由于 SLF4J 中的 NDC 实际上通过 MDC 来实现，在直接使用 SLF4J 的 API 时，%x 并不能获取到 NDC 中的值。</p>
<p data-nodeid="5130">如果以 Log4j 2 作为日志实现，推荐的做法是直接使用 ThreadContext 类，该类同时提供了对 MDC 和 NDC 的支持。下面的代码展示了 ThreadContext 中 NDC 功能的使用方式。</p>
<pre class="lang-java" data-nodeid="5131"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Log4jThreadContext</span> </span>{
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Logger LOGGER = LogManager.getLogger(<span class="hljs-string">"ThreadContext"</span>);
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">display</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; ThreadContext.push(<span class="hljs-string">"user1"</span>);
&nbsp; &nbsp; LOGGER.info(<span class="hljs-string">"message 1"</span>);
&nbsp; &nbsp; LOGGER.info(<span class="hljs-string">"message 2"</span>);
&nbsp; &nbsp; ThreadContext.pop();
&nbsp; &nbsp; LOGGER.info(<span class="hljs-string">"message 3"</span>); <span class="hljs-comment">// NDC中已经没有值</span>
&nbsp; }
}
</code></pre>
<p data-nodeid="5132">MDC 通常作为任务执行时的上下文。当退出当前的执行上下文之后，MDC 中的内容应该被恢复。Log4j 2 提供了 CloseableThreadContext 类来方便对 ThreadContext 的管理。当 CloseableThreadContext 对象关闭时，对 ThreadContext 所做的修改会被自动恢复。下面代码中 ThreadContextHelper 类的 withContext 方法，可以在指定的上下文对象中，执行 Runnable 表示的代码。</p>
<pre class="lang-java" data-nodeid="5133"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ThreadContextHelper</span> </span>{
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">withContext</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Map&lt;String, String&gt; context, <span class="hljs-keyword">final</span> Runnable action)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> (<span class="hljs-keyword">final</span> Instance ignored = CloseableThreadContext.putAll(context)) {
&nbsp; &nbsp; &nbsp; action.run();
&nbsp; &nbsp; }
&nbsp; }
}
</code></pre>
<p data-nodeid="5134">在下面的代码中，withContext 方法中的两条日志记录可以访问 userId 的值，而最后一条日志记录无法访问。</p>
<pre class="lang-java" data-nodeid="5135"><code data-language="java">ThreadContextHelper.withContext(
&nbsp; &nbsp; ImmutableMap.of(<span class="hljs-string">"userId"</span>, <span class="hljs-string">"12345"</span>), () -&gt; {
&nbsp; &nbsp; &nbsp; LOGGER.info(<span class="hljs-string">"message 1"</span>);
&nbsp; &nbsp; &nbsp; LOGGER.info(<span class="hljs-string">"message 2"</span>);
&nbsp; &nbsp; });
LOGGER.info(<span class="hljs-string">"message 3"</span>);
</code></pre>
<p data-nodeid="5136">SLF4J 中的 MDC.MDCCloseable 类的作用与 CloseableThreadContext 类似，通过 MDC 的 putCloseable 方法来使用，如下面的代码所示。</p>
<pre class="lang-java" data-nodeid="5137"><code data-language="java"><span class="hljs-keyword">try</span>&nbsp;(<span class="hljs-keyword">final</span>&nbsp;MDC.MDCCloseable&nbsp;ignored&nbsp;=&nbsp;MDC.putCloseable(<span class="hljs-string">"userId"</span>,&nbsp;<span class="hljs-string">"12345"</span>))&nbsp;{
&nbsp;&nbsp;LOGGER.info(<span class="hljs-string">"message&nbsp;1"</span>);
}
</code></pre>
<h3 data-nodeid="5138">日志聚合</h3>
<p data-nodeid="5139">在单体应用中，日志通常被写入到文件中。当出现问题时，最直接的做法是在日志文件中根据错误产生的时间和错误消息进行查找，这种做法的效率很低。如果应用同时运行在多个虚拟机之上，需要对多个应用实例产生的日志记录进行聚合，并提供统一的查询视图。有很多的开源和商用解决方案提供了对日志聚合的支持，典型的是 ELK 技术栈，即 Elasticsearch、Logstash 和 Kibana 的集成。这 3 个组成部分代表了日志管理系统的 3 个重要功能，分别是日志的收集、保存与索引、查询。</p>
<p data-nodeid="5140">对于微服务架构的云原生应用来说，日志管理的要求更高，应用被拆分成多个微服务，每个微服务在运行时的实例数量可能很多。在 Kubernetes 上，需要收集的是 Pod 中产生的日志。</p>
<p data-nodeid="5141">在单体应用中，日志消息的主要消费者是开发人员，因此日志消息侧重的是可读性，一般是半结构化的字符串形式。通过模式布局，从日志事件中提取出感兴趣的属性，并格式化成日志消息。日志消息是半结构化的，通过正则表达式可以从中提取相关的信息。</p>
<p data-nodeid="5142">当需要进行日志的聚合时，半结构化的日志消息变得不再适用，因为日志消息的消费者变成了日志收集程序，JSON 这样的结构化日志成了更好的选择。如果可以完全控制日志的格式，推荐使用 JSON。对于来自外部应用的日志消息，如果是纯文本格式的，仍然需要通过工具来解析并转换成 JSON。</p>
<p data-nodeid="5143">当应用在容器中运行时，日志并不需要写到文件中，而是直接写入到标准输出流。Kubernetes 会把容器中产生的输出保存在节点的文件中，可以由工具进行收集。</p>
<h3 data-nodeid="5144">Fluentd</h3>
<p data-nodeid="5145"><a href="https://www.fluentd.org/" data-nodeid="5412">Fluentd</a> 是一个开源的数据收集器，可以提供统一的日志管理；还可以通过灵活的插件架构，与不同的日志数据源和目的地进行集成。</p>
<p data-nodeid="5146">Fluentd 使用 JSON 作为数据格式，同时以事件来表示每条日志记录。事件由下表中给出的 3 个部分组成。</p>
<table data-nodeid="5148">
<thead data-nodeid="5149">
<tr data-nodeid="5150">
<th data-org-content="**属性**" data-nodeid="5152"><strong data-nodeid="5418">属性</strong></th>
<th data-org-content="**说明**" data-nodeid="5153"><strong data-nodeid="5422">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="5156">
<tr data-nodeid="5157">
<td data-org-content="标签" data-nodeid="5158">标签</td>
<td data-org-content="事件源的标识" data-nodeid="5159">事件源的标识</td>
</tr>
<tr data-nodeid="5160">
<td data-org-content="时间戳" data-nodeid="5161">时间戳</td>
<td data-org-content="事件的产生时间" data-nodeid="5162">事件的产生时间</td>
</tr>
<tr data-nodeid="5163">
<td data-org-content="记录" data-nodeid="5164">记录</td>
<td data-org-content="JSON 格式的日志记录" data-nodeid="5165">JSON 格式的日志记录</td>
</tr>
</tbody>
</table>
<p data-nodeid="5166">Fluentd 采用插件化的架构来方便扩展，其插件分为输入、解析、过滤、输出、格式化、存储、服务发现和缓冲等 8 个类别。下表给出了这 8 个类别的说明和插件示例。</p>
<table data-nodeid="5168">
<thead data-nodeid="5169">
<tr data-nodeid="5170">
<th data-org-content="**类别**" data-nodeid="5172"><strong data-nodeid="5433">类别</strong></th>
<th data-org-content="**说明**" data-nodeid="5173"><strong data-nodeid="5437">说明</strong></th>
<th data-org-content="**插件**" data-nodeid="5174"><strong data-nodeid="5441">插件</strong></th>
</tr>
</thead>
<tbody data-nodeid="5178">
<tr data-nodeid="5179">
<td data-org-content="输入" data-nodeid="5180">输入</td>
<td data-org-content="从外部源中获取事件日志" data-nodeid="5181">从外部源中获取事件日志</td>
<td data-org-content="文件、UDP、TCP、HTTP、syslog 等" data-nodeid="5182">文件、UDP、TCP、HTTP、syslog 等</td>
</tr>
<tr data-nodeid="5183">
<td data-org-content="解析" data-nodeid="5184">解析</td>
<td data-org-content="解析事件日志的内容" data-nodeid="5185">解析事件日志的内容</td>
<td data-org-content="正则表达式、Apache 2、Nginx、CSV、JSON" data-nodeid="5186">正则表达式、Apache 2、Nginx、CSV、JSON</td>
</tr>
<tr data-nodeid="5187">
<td data-org-content="过滤" data-nodeid="5188">过滤</td>
<td data-org-content="对事件进行修改，包括提取字段、添加新字段、删除字段" data-nodeid="5189">对事件进行修改，包括提取字段、添加新字段、删除字段</td>
<td data-org-content="grep、记录转换器" data-nodeid="5190">grep、记录转换器</td>
</tr>
<tr data-nodeid="5191">
<td data-org-content="输出" data-nodeid="5192">输出</td>
<td data-org-content="事件日志的输出目的地" data-nodeid="5193">事件日志的输出目的地</td>
<td data-org-content="文件、HTTP、Elasticsearch、Kafka、MongoDB、Amazon S3" data-nodeid="5194">文件、HTTP、Elasticsearch、Kafka、MongoDB、Amazon S3</td>
</tr>
<tr data-nodeid="5195">
<td data-org-content="格式化" data-nodeid="5196">格式化</td>
<td data-org-content="对事件输出进行格式化" data-nodeid="5197">对事件输出进行格式化</td>
<td data-org-content="JSON、CSV、单个值" data-nodeid="5198">JSON、CSV、单个值</td>
</tr>
<tr data-nodeid="5199">
<td data-org-content="存储" data-nodeid="5200">存储</td>
<td data-org-content="保存插件的内部状态" data-nodeid="5201">保存插件的内部状态</td>
<td data-org-content="本地文件" data-nodeid="5202">本地文件</td>
</tr>
<tr data-nodeid="5203">
<td data-org-content="服务发现" data-nodeid="5204">服务发现</td>
<td data-org-content="发现输出的目的地" data-nodeid="5205">发现输出的目的地</td>
<td data-org-content="静态目标、文件" data-nodeid="5206">静态目标、文件</td>
</tr>
<tr data-nodeid="5207">
<td data-org-content="缓冲" data-nodeid="5208">缓冲</td>
<td data-org-content="输出插件的缓冲" data-nodeid="5209">输出插件的缓冲</td>
<td data-org-content="文件、内存" data-nodeid="5210">文件、内存</td>
</tr>
</tbody>
</table>
<p data-nodeid="23083" class="">Fluentd 以流水线的方式来处理日志事件，流水线由 Fluentd 的配置文件来定义。流水线最基本的组成元素是输入、过滤器和输出，分别用下表中的 <code data-backticks="1" data-nodeid="23085">&lt;source&gt;</code>、<code data-backticks="1" data-nodeid="23087">&lt;filter&gt;</code> 和 <code data-backticks="1" data-nodeid="23089">&lt;match&gt;</code> 指令来声明。事件的标签在流水线中很重要，用来选择不同的处理方式。</p>



<table data-nodeid="12707">
<thead data-nodeid="12708">
<tr data-nodeid="12709">
<th data-org-content="**指令**" data-nodeid="12711"><strong data-nodeid="12728">指令</strong></th>
<th data-org-content="**说明**" data-nodeid="12712"><strong data-nodeid="12732">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="12715">
<tr data-nodeid="12716">
<td data-org-content="`<source>`" data-nodeid="12717"><code data-backticks="1" data-nodeid="12733">&lt;source&gt;</code></td>
<td data-org-content="事件的输入源" data-nodeid="12718">事件的输入源</td>
</tr>
<tr data-nodeid="12719">
<td data-org-content="`<filter>`" data-nodeid="12720"><code data-backticks="1" data-nodeid="12735">&lt;filter&gt;</code></td>
<td data-org-content="对事件进行处理，与事件的标签匹配" data-nodeid="12721">对事件进行处理，与事件的标签匹配</td>
</tr>
<tr data-nodeid="12722">
<td data-org-content="`<match>`" data-nodeid="12723" class=""><code data-backticks="1" data-nodeid="12737">&lt;match&gt;</code></td>
<td data-org-content="对事件进行处理和输出，与事件的标签匹配" data-nodeid="12724">对事件进行处理和输出，与事件的标签匹配</td>
</tr>
</tbody>
</table>







<p data-nodeid="5231">除了上表中的 3 个基本指令之外，下表还给出了两个内嵌指令的说明。</p>
<table data-nodeid="20018">
<thead data-nodeid="20019">
<tr data-nodeid="20020">
<th data-org-content="**指令**" data-nodeid="20022"><strong data-nodeid="20040">指令</strong></th>
<th data-org-content="**说明**" data-nodeid="20023"><strong data-nodeid="20044">说明</strong></th>
<th data-org-content="**可能的父指令**" data-nodeid="20024"><strong data-nodeid="20048">可能的父指令</strong></th>
</tr>
</thead>
<tbody data-nodeid="20028">
<tr data-nodeid="20029">
<td data-org-content="`<parse>`" data-nodeid="20030"><code data-backticks="1" data-nodeid="20049">&lt;parse&gt;</code></td>
<td data-org-content="使用解析插件" data-nodeid="20031">使用解析插件</td>
<td data-org-content="`<source>`、`<filter>`和`<match>`" data-nodeid="20032"><code data-backticks="1" data-nodeid="20051">&lt;source&gt;</code>、<code data-backticks="1" data-nodeid="20053">&lt;filter&gt;</code>和<code data-backticks="1" data-nodeid="20055">&lt;match&gt;</code></td>
</tr>
<tr data-nodeid="20033">
<td data-org-content="` <format>`" data-nodeid="20034"><code data-backticks="1" data-nodeid="20056"> &lt;format&gt;</code></td>
<td data-org-content="使用格式化插件" data-nodeid="20035">使用格式化插件</td>
<td data-org-content="`<filter>`和`<match>`" data-nodeid="20036" class=""><code data-backticks="1" data-nodeid="20058">&lt;filter&gt;</code>和<code data-backticks="1" data-nodeid="20060">&lt;match&gt;</code></td>
</tr>
</tbody>
</table>







<p data-nodeid="5252">在配置插件时，通过 @type 参数来指定插件的名称。下面的代码是 Fluentd 的配置文件的示例，其中定义了一个从 HTTP 输入到文件输出的处理流水线。输入源是运行在 8280 端口的 HTTP 服务；过滤操作匹配标签为 app.log 的事件，并添加 hostname 字段；输出目的地是文件，并通过格式化插件 json 转换为 JSON 格式。发送 HTTP POST 请求到 URL http://localhost:8280/app.log 可以发布新的事件，POST 请求的路径 app.log 是事件的标签。</p>
<pre class="lang-xml" data-nodeid="5253"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">source</span>&gt;</span>
&nbsp;&nbsp;@type&nbsp;http
&nbsp;&nbsp;port&nbsp;8280
&nbsp;&nbsp;bind&nbsp;0.0.0.0
<span class="hljs-tag">&lt;/<span class="hljs-name">source</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">filter</span>&nbsp;<span class="hljs-attr">app.log</span>&gt;</span>
&nbsp;&nbsp;@type&nbsp;record_transformer
&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">record</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;hostname&nbsp;"#{Socket.gethostname}"
&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">record</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">filter</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">match</span>&nbsp;<span class="hljs-attr">app.log</span>&gt;</span>
&nbsp;&nbsp;@type&nbsp;file
&nbsp;&nbsp;path&nbsp;/opt/app/log
&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">format</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;@type&nbsp;json
&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">format</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">match</span>&gt;</span>
</code></pre>
<p data-nodeid="5254">除了 Fluentd 之外，还可以使用 <a href="https://www.elastic.co/beats/filebeat" data-nodeid="5516">Filebeat</a> 或 <a href="https://www.elastic.co/logstash" data-nodeid="5520">Logstash</a> 来收集日志。</p>
<h3 data-nodeid="5255">Elasticsearch 和 Kibana</h3>
<p data-nodeid="5256">当收集到来自不同源的日志事件之后，还需要进行存储和搜索。在流行的日志处理技术栈中，Elasticsearch 和 Kibana 是两个常用的选择，前者提供了日志事件的存储和搜索，而后者则提供了日志查询和结果的展示。</p>
<p data-nodeid="5257">在 Kubernetes 上，可以使用 Helm 来安装 Elasticsearch 和 Kibana。不过更推荐的做法是使用 <a href="https://www.elastic.co/guide/en/cloud-on-k8s/current/index.html" data-nodeid="5527">Elastic Cloud on Kubernetes</a>（ECK）。ECK 基于 Kubernetes 上的操作员模式来实现，提供了更好的可伸缩性和可维护性，类似第 28 课时介绍的 Prometheus Operator。</p>
<p data-nodeid="5258">首先使用下面的命令安装 ECK 的自定义资源定义。</p>
<pre class="lang-java" data-nodeid="24595"><code data-language="java">kubectl apply -f https:<span class="hljs-comment">//download.elastic.co/downloads/eck/1.1.2/all-in-one.yaml</span>
</code></pre>


<p data-nodeid="5260">接着可以使用 ECK 提供的自定义资源定义来创建 Elasticsearch 集群。下面的代码创建了一个名为 default 包含一个节点的 Elasticsearch 集群。</p>
<pre class="lang-yaml" data-nodeid="5261"><code data-language="yaml"><span class="hljs-string">apiVersion:</span>&nbsp;<span class="hljs-string">elasticsearch.k8s.elastic.co/v1</span>
<span class="hljs-string">kind:</span>&nbsp;<span class="hljs-string">Elasticsearch</span>
<span class="hljs-attr">metadata:</span>
&nbsp;&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">default</span>
<span class="hljs-attr">spec:</span>
&nbsp;&nbsp;<span class="hljs-string">version:</span>&nbsp;<span class="hljs-number">7.8</span><span class="hljs-number">.0</span>
&nbsp;&nbsp;<span class="hljs-attr">nodeSets:</span>
&nbsp;&nbsp;<span class="hljs-string">-</span>&nbsp;<span class="hljs-string">name:</span>&nbsp;<span class="hljs-string">default</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">count:</span>&nbsp;<span class="hljs-number">1</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">config:</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">node.master:</span>&nbsp;<span class="hljs-literal">true</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">node.data:</span>&nbsp;<span class="hljs-literal">true</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">node.ingest:</span>&nbsp;<span class="hljs-literal">true</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">node.ml:</span>&nbsp;<span class="hljs-literal">false</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">xpack.ml.enabled:</span>&nbsp;<span class="hljs-literal">false</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">node.store.allow_mmap:</span>&nbsp;<span class="hljs-literal">false</span>
</code></pre>
<p data-nodeid="5262">在 Elasticsearch 集群创建之后，可以使用 kubectl get elasticsearch 命令来查看集群的状态，输出结果如下面的代码所示。</p>
<pre class="lang-java" data-nodeid="25598"><code data-language="java">NAME&nbsp; &nbsp; &nbsp; HEALTH&nbsp; &nbsp;NODES&nbsp; &nbsp;VERSION&nbsp; &nbsp;PHASE&nbsp; &nbsp;AGE
<span class="hljs-keyword">default</span>&nbsp; &nbsp;green&nbsp; &nbsp; <span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">7.8</span>.<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp;Ready&nbsp; &nbsp;<span class="hljs-number">21</span>m
</code></pre>

<p data-nodeid="5264">Kibana 的部署方式类似于 Elasticsearch，如下面的代码所示。属性 elasticsearchRef 的值用来配置 Kibana，引用之前创建的 Elasticsearch 集群。</p>
<pre class="lang-yaml" data-nodeid="5265"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">kibana.k8s.elastic.co/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Kibana</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">default</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">version:</span> <span class="hljs-number">7.8</span><span class="hljs-number">.0</span>
  <span class="hljs-attr">count:</span> <span class="hljs-number">1</span>
  <span class="hljs-attr">elasticsearchRef:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">default</span>
</code></pre>
<p data-nodeid="5266">当 Kibana 部署完成之后，可以在本地机器上使用 kubectl port-forward 来访问 Kibana 界面，如下面的代码所示：</p>
<pre class="lang-java" data-nodeid="26601"><code data-language="java">kubectl port-forward svc/<span class="hljs-keyword">default</span>-kb-http <span class="hljs-number">5601</span>
</code></pre>

<p data-nodeid="5268">使用浏览器访问 https://localhost:5601 即可。需要注意的是，Kibana 服务器默认使用了自签名的 SSL 证书，浏览器会给出警告，在开发环境中可以忽略。Kibana 的登录用户名是 elastic，而密码需要从 Kubernetes 的 Secret 中获取，使用下面的代码可以获取到密码。</p>
<pre class="lang-java" data-nodeid="27604"><code data-language="java">kubectl get secret <span class="hljs-keyword">default</span>-es-elastic-user -o go-template=<span class="hljs-string">'{{.data.elastic | base64decode}}'</span>
</code></pre>

<p data-nodeid="5270">接着需要在 Kubernetes 上运行 Fluentd。Fluentd 以守护进程集（DaemonSet）的形式来运行，确保在每个节点上都可以运行；同时它会收集容器中产生的日志，并发送到 Elasticsearch。</p>
<p data-nodeid="5271">下面的代码是创建 Fluentd 的守护进程集的 YAML 文件。通过卷的绑定，Fluentd 可以读取节点上 /var/log 和 /var/lib/docker/containers 目录下的日志文件。</p>
<pre class="lang-yaml" data-nodeid="5272"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">DaemonSet</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">fluentd</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">default</span>
  <span class="hljs-attr">labels:</span>
    <span class="hljs-attr">app.kubernetes.io/name:</span> <span class="hljs-string">fluentd-logging</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">matchLabels:</span>
      <span class="hljs-attr">app.kubernetes.io/name:</span> <span class="hljs-string">fluentd-logging</span>
  <span class="hljs-attr">template:</span>
    <span class="hljs-attr">metadata:</span>
      <span class="hljs-attr">labels:</span>
        <span class="hljs-attr">app.kubernetes.io/name:</span> <span class="hljs-string">fluentd-logging</span>
    <span class="hljs-attr">spec:</span>
      <span class="hljs-attr">containers:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">fluentd</span>
        <span class="hljs-attr">image:</span> <span class="hljs-string">fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch</span>
        <span class="hljs-attr">env:</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENTD_SYSTEMD_CONF</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"disabled"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span>  <span class="hljs-string">FLUENT_ELASTICSEARCH_HOST</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"default-es-http"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span>  <span class="hljs-string">FLUENT_ELASTICSEARCH_PORT</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"9200"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_SCHEME</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"https"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_SSL_VERIFY</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"false"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_USER</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"elastic"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_PASSWORD</span>
            <span class="hljs-attr">valueFrom:</span>
              <span class="hljs-attr">secretKeyRef:</span>
                <span class="hljs-attr">name:</span> <span class="hljs-string">"default-es-elastic-user"</span>
                <span class="hljs-attr">key:</span> <span class="hljs-string">"elastic"</span> 
        <span class="hljs-attr">resources:</span>
          <span class="hljs-attr">limits:</span>
            <span class="hljs-attr">memory:</span> <span class="hljs-string">200Mi</span>
          <span class="hljs-attr">requests:</span>
            <span class="hljs-attr">cpu:</span> <span class="hljs-string">100m</span>
            <span class="hljs-attr">memory:</span> <span class="hljs-string">200Mi</span>
        <span class="hljs-attr">volumeMounts:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlog</span>
          <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/var/log</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlibdockercontainers</span>
          <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/var/lib/docker/containers</span>
          <span class="hljs-attr">readOnly:</span> <span class="hljs-literal">true</span>
      <span class="hljs-attr">terminationGracePeriodSeconds:</span> <span class="hljs-number">30</span>
      <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlog</span>
        <span class="hljs-attr">hostPath:</span>
          <span class="hljs-attr">path:</span> <span class="hljs-string">/var/log</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlibdockercontainers</span>
        <span class="hljs-attr">hostPath:</span>
          <span class="hljs-attr">path:</span> <span class="hljs-string">/var/lib/docker/containers</span>
</code></pre>
<p data-nodeid="29601">在首次使用 Kibana 时，需要配置索引的模式，使用 logstash-* 作为模式即可。下图是 Kibana 查询日志的界面，可以通过标签来快速对日志消息进行过滤。</p>
<p data-nodeid="29602" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/26/C2/Ciqc1F7y-cuAFBf_AAJiFWEnj7g432.png" alt="kibana.png" data-nodeid="29608"></p>


<h3 data-nodeid="5275">总结</h3>
<p data-nodeid="5276">应用的开发和维护都离不开日志的支持，对于微服务架构的云原生应用来说，完整的日志聚合、分析和查询的技术栈是必不可少的。通过本课时的学习，你可以掌握 Java 应用中记录日志的方式和最佳实践，还可以了解如何基于 Fluentd、Elasticsearch 和 Kibana，在 Kubernetes 上构建自己的日志聚合、分析和查询的完整技术栈。</p>

---

### 精选评论


