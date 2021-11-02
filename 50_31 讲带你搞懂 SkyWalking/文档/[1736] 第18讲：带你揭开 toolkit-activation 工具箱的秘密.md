<p>在前面两课时中，我们详细介绍了 tomcat-7.x-8.x-plugin 插件以及 dubbo-2.7.x-plugin 插件的核心实现。但是在有些场景中，不仅需要通过插件收集开源组件的 Trace 数据，还需要收集某些关键业务逻辑的 Trace 数据。我们可以通过开发新插件的方式来实现该需求，但是成本是非常高的，尤其是当业务代码发生重构时（例如，方法名或是类名改变了），插件也需要随之修改、发布 jar 包，非常麻烦。</p>
<h3>toolkit-trace 插件</h3>
<p>SkyWalking 为了解决上述问题，提供了一个 @Trace 注解，我们只要将该注解添加到需要监控的业务方法之上，即可收集到该方法相关的 Trace 数据。</p>
<p>下面我们先通过 demo-webapp 介绍 @Trace 注解的使用和效果。首先，我们定义一个 Service 类—— DemoService：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Service</span> <span class="hljs-comment">// Spring的@Service注解</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DemoService</span> </span>{
    <span class="hljs-comment">// 添加@Trace注解，使用该注解需要引入apm-toolkit-trace依赖，</span>
    <span class="hljs-comment">// 在搭建demo-webapp项目时已经介绍过了，pom文件不再展示</span>
    <span class="hljs-meta">@Trace</span>(operationName = <span class="hljs-string">"default-trace-method"</span>)
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">traceMethod</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
        Thread.sleep(<span class="hljs-number">1000</span>);
        ActiveSpan.tag(<span class="hljs-string">"trace-method"</span>, 
             String.valueOf(System.currentTimeMillis()));
        ActiveSpan.info(<span class="hljs-string">"traceMethod info Message"</span>);
        System.out.println(TraceContext.traceId()); <span class="hljs-comment">// 打印Trace ID</span>
    }
}
</code></pre>
<p>然后在 HelloWorldController 中注入 DemoService，并在 "/hello/{words}" 接口中调用 traceMethod() 方法：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping</span>(<span class="hljs-string">"/"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloWorldController</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> DemoService demoService;

    <span class="hljs-meta">@GetMapping</span>(<span class="hljs-string">"/hello/{words}"</span>)
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">hello</span><span class="hljs-params">(@PathVariable(<span class="hljs-string">"words"</span>)</span> String words) </span>{
        ... 
        demoService.traceMethod();
        ... <span class="hljs-comment">// 省略其他方法</span>
    }
}
</code></pre>
<p>接下来访问 "localhost:8000/hello/xxx" 这个地址等待片刻之后，即可在 SkyWalking Rocketbot 界面中看到相应的 Span 数据，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvL-AJxsiAALX11rEFcE040.png" alt="image.png"></p>
<p>点击该 Span，可以看到具体的 Tag 信息以及 Log 信息，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/Ciqc1F6zvMeAIijwAAG4HeGpP1w020.png" alt="image (1).png"></p>
<h4>深入工具类原理</h4>
<p>了解了 @Trace 注解的使用之后，我们来分析其底层实现。首先我们跳转到 SkyWalking 项目的 apm-toolkit-trace 模块，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvNCANNntAAF2iT3mNig327.png" alt="image (2).png"></p>
<p>该模块就有前面使用到的 @Trace 注解以及 ActiveSpan、TraceContext 工具类，打开这两个工具类会发现，全部是空实现，那添加 Tag、获取 Trace ID 等操作是如何完成的呢？我在前面介绍 SkyWalking 源码各个模块功能时提到，apm-application-toolkit 模块类似于暴露 API 定义，对应的处理逻辑在 apm-sniffer/apm-toolkit-activation 模块中实现。</p>
<p>在 apm-toolkit-trace-activation 模块的 skywalking-plugin.def 文件中定义了四个 ClassEnhancePluginDefine 实现类：</p>
<ul>
<li>ActiveSpanActivation</li>
<li>TraceAnnotationActivation</li>
<li>TraceContextActivation</li>
<li>CallableOrRunnableActivation</li>
</ul>
<p>这四个 ClassEnhancePluginDefine 实现类的继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvNmAEeMhAAHH-Xli_I8395.png" alt="image (3).png"></p>
<p>TraceAnnotationActivation 会拦截所有被 @Trace 注解标记的方法所在的类，在 TraceAnnotationActivation 覆盖的 enhanceClass() 方法中可以看到相关实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">static</span> ClassMatch <span class="hljs-title">byMethodAnnotationMatch</span><span class="hljs-params">(String[] annotations)</span></span>{
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MethodAnnotationMatch(annotations); 
}
</code></pre>
<p>MethodAnnotationMatch 在判断一个类是否符合条件时，会遍历类中的全部方法，只要发现一个被 @Trace 注解标记的方法，则该类符合拦截条件。</p>
<p>从 getInstanceMethodsInterceptPoints() 方法中可以看到，@Trace 注解的相关增强逻辑定义在 TraceAnnotationMethodInterceptor 中，其 beforeMethod() 方法会调用 ContextManager.createLocalSpan() 方法创建 LocalSpan（注意，EndpointName 优先从注解配置中获取）。在 afterMethod() 方法中会关闭该 LocalSpan，在 handleMethodException() 方法会将异常堆栈作为 Log 记录在该 LocalSpan 中。</p>
<p>TraceContextActivation 拦截的是 TraceContext.traceId() 这个 static 静态方法，具体增强逻辑在 TraceContextInterceptor 中，其 afterMethod() 方法会调用 ContextManager.getGlobalTraceId() 方法获取当前线程绑定的 Trace ID 并替换 TraceContext.traceId() 方法返回的空字符串。</p>
<p>ActiveSpanActivation 会拦截 ActiveSpan 类中 static 静态方法并交给不同的 Interceptor 进行增强，具体的 static 静态方法与 Interceptor 之间的映射关系如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvOSAJMFjAAIONP7amFM561.png" alt="image (4).png"></p>
<p>这里以 tag() 方法为例，在 ActiveSpanTagInterceptor 的 beforeMethod() 方法中，会获取 activeSpanStack 栈顶的 Span 对象，并调用其 tag() 方法记录 Tag 信息。其他的 ActiveSpan*Interceptor 会通过 Span.log() 方法记录 Log，这里不再展开。</p>
<h4>跨线程传播</h4>
<p>前面的课时已经详细介绍了 Trace 信息跨进程传播的实现原理，这里我们简单看一下跨线程传播的场景。这里我们在 HelloWorldService 中启动一个线程池，并改造 DemoService 的调用方式：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping</span>(<span class="hljs-string">"/"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloWorldController</span> </span>{
    <span class="hljs-comment">// 启动一个单线程的线程池</span>
    <span class="hljs-keyword">private</span> ExecutorService executorService = 
            Executors.newSingleThreadScheduledExecutor();

    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> DemoService demoService;

    <span class="hljs-meta">@GetMapping</span>(<span class="hljs-string">"/hello/{words}"</span>)
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">hello</span><span class="hljs-params">(@PathVariable(<span class="hljs-string">"words"</span>)</span> String words)</span>{
        ... <span class="hljs-comment">// 省略其他调用</span>
        executorService.submit( <span class="hljs-comment">// 省略try/catch代码块</span>
            <span class="hljs-comment">// 使用RunnableWrapper对Runnable进行包装，实现Trace跨线程传播</span>
            RunnableWrapper.of(() -&gt; demoService.traceMethod())
        );
        ...
    }
}
</code></pre>
<p>此时再访问 "<a href="http://localhost:8000/hello/xxx">http://localhost:8000/hello/xxx</a>" 地址，稍等片刻之后，会在 SkyWalking Rocketbot 上看到下图这种分叉的 Trace，其中下面那条 Trace 分支就是通过跨线程传播过去的：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/Ciqc1F6zvO-AEq9VAAFo-ZWXkGM186.png" alt="跨线程传播.png"></p>
<p>除了通过 RunnableWrapper 包装 Runnable 之外，我们可以通过 CallableWrapper 包装 Callable 实现 Trace 的跨线程传播。下图展示了 Trace 信息跨线程传播的核心原理：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/Ciqc1F6zvPqAKHZ1AAFedhJrbC8809.png" alt="image (5).png"></p>
<p>下面来看 RunnableWrapper 和 CallableWrapper 的实现原理。toolkit-trace-activation 中的 CallableOrRunnableActivation 会拦截被 @TraceCrossThread 注解标记的类（RunnableWrapper 和 CallableWrapper 都标注了 @TraceCrossThread 注解）。</p>
<p>目标类的构造方法会由 CallableOrRunnableConstructInterceptor 进行增强，其中会调用 capture() 方法将当前 TracingContext 的核心信息填充到 ContextSnapshot 中，并记录到_$EnhancedClassField_ws 字段中：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onConstruct</span><span class="hljs-params">(EnhancedInstance objInst, 
       Object[] allArguments)</span> </span>{
    <span class="hljs-keyword">if</span> (ContextManager.isActive()) {
        objInst.setSkyWalkingDynamicField(ContextManager.capture());
    }
}
</code></pre>
<p>此时的 Runnable（或 Callable）对象就携带了当前线程关联的 Trace 信息。</p>
<p>目标类的 run() 方法或是 callable() 方法由 CallableOrRunnableInvokeInterceptor 进行增强，其 before() 方法会创建 LocalSpan，在上面的 demo-webapp 示例中，线程池中的工作线程没有关联的 TracingContext 也会新创建，之后从增强的 _$EnhancedClassField_ws 字段中获取 ContextSnapshot 对象，将上游线程的数据恢复到新建的 TracingContext 中，具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">beforeMethod</span><span class="hljs-params">(EnhancedInstance objInst, Method method,
        Object[] allArguments, Class&lt;?&gt;[] argumentsTypes,
            MethodInterceptResult result)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
    <span class="hljs-comment">// 该调用中会先创建TracingContext，然后创建LocalSpan</span>
    ContextManager.createLocalSpan(<span class="hljs-string">"Thread/"</span> + 
         objInst.getClass().getName() + <span class="hljs-string">"/"</span> + method.getName());
    ContextSnapshot cachedObjects = 
        (ContextSnapshot)objInst.getSkyWalkingDynamicField();
    <span class="hljs-keyword">if</span> (cachedObjects != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 恢复Trace信息</span>
        ContextManager.continued(cachedObjects);
    }
}
</code></pre>
<p>在 afterMethod() 中会关闭前置增强逻辑中创建的 LocalSpan，同时，为了防止内存泄漏，会清空 _$EnhancedClassField_ws 字段。</p>
<h3>Trace ID 与日志</h3>
<p>在实际定位问题的时候，我们可能需要将某个用户的某个请求的 Trace 监控以及相关的日志结合起来进行分析，毕竟 Trace 携带 Log 有限，不会携带请求整个生命周期中全部的日志。为了方便将 Trace 和日志进行关联，一般会在日志开头的固定位置打印 Trace ID，<br>
application-toolkit 工具箱目前支持 logback、log4j-1.x、log4j-2.x 三个日志框架，下面以 logback 为例演示并分析原理。</p>
<h4>日志集成 Trace ID</h4>
<p>这里依然通过 demo-webapp 模块为例，介绍如何在日志文件中自动输出 Trace ID。首先我们引入 apm-toolkit-logback-1.x 这个依赖，如下所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.skywalking<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>apm-toolkit-logback-1.x<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>6.2.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p>接下来在 resource 目录下添加 logback.xml 配置文件，指定日志的输出格式：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvQeAdq60AAJd-oywE_o373.png" alt="image (6).png"></p>
<p>该配置文件有两个地方需要注意，一个是使用的 layout 为 TraceIdPatternLogbackLayout，该类位于 apm-toolkit-logback-1.x.jar 这个依赖包中，另一个在 pattern 配置中添加了 [%tid] 占位符。</p>
<p>在 HelloWorldController 中的 "/hello/world" 接口中，我们添加一条日志输出：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Logger LOGGER = 
       LoggerFactory.getLogger(HelloWorldController<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;

<span class="hljs-meta">@GetMapping</span>(<span class="hljs-string">"/hello/{words}"</span>)
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">hello</span><span class="hljs-params">(@PathVariable(<span class="hljs-string">"words"</span>)</span> String words)
    LOGGER.<span class="hljs-title">info</span><span class="hljs-params">(<span class="hljs-string">"this is an info log,{}"</span>, words)</span></span>;
    ... <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p>最后重启 demo-webapp 项目，访问 "localhost:8000/hello/xxx" 这个地址，就可以在控制台看到如下输出：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/Ciqc1F6zvRKAMCK4AAHOkonOBic764.png" alt="image (7).png"></p>
<h4>Logback 核心概念</h4>
<p>Logback 日志框架分为三个模块：logback-core、logback-classic 和 logback-access：</p>
<ul>
<li><strong>core 模块</strong>是整个 logback 的核心基础。</li>
<li><strong>classic 模块</strong>是在 core 模块上的扩展，classic 模块实现了 SLF4J API。</li>
<li><strong>access 模块</strong>主要用于与 Servlet 容器进行集成，实现记录 access-log 的功能。</li>
</ul>
<p>Logback 日志框架中有三个核心类：Logger、Appender 和 Layout。Logger 主要用来接收要输出的日志内容。每个 Logger 实例都有名字，而 Logger 的继承关系与其名称的层级关系保持一致。例如，现在有 3 个 Logger 实例 L1、L2、L3，L1 的名字为 "com"，L2 的名字为 "com.xxx"，L3 的名字为 "com.xxx.Main"，那么三者的继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/Ciqc1F6zvRqAVReiAAEEjIqgllA872.png" alt="image (8).png"></p>
<p>其中，名为 "ROOT" 的 Logger 实例是顶层 Logger，它是所有其他 Logger 实例的祖先。</p>
<p>每个 Logger 实例都有对应的 Level 级别，如果未明确指定 Logger 实例的 Level 级别，则默认沿用上层 Logger 实例的 Level 级别，常用的 Level 级别以及 Level 优先级如下 ：</p>
<pre><code data-language="html" class="lang-html">TRACE &lt; DEBUG &lt; INFO &lt; WARN &lt; ERROR
</code></pre>
<p>在调用 Logger 实例记录日志时会产生对应的日志记录请求，每个日志记录请求也有一个 Level 属性。只有日志记录请求的 Level 属性值大于或等于相应的 Logger 实例的 Level 级别时，该日志记录请求才是有效的。例如，有一个 Logger 实例的 Level 级别为 INFO，调用它的 error() 方法产生的日志记录请求的 Level 级别为 ERROR，ERROR &gt; INFO，所以该日志记录可以正常输出；如果调用其 debug() 方法，则产生的日志记录请求 Level 级别为 DEBUG，DEBUG &lt; INFO，则该日志记录无法正常输出。</p>
<p>另外，当一个 Logger 实例的 Level 级别为 OFF 时，任何在该 Logger 实例上产生的日志记录请求都是无效的；当一个 Logger 实例的 Level 级别为 ALL 时，任何在该 Logger 实例上产生的日志记录请求都是有效的。</p>
<p>在使用 Logback 时，我们都是通过下面的方式获取 Logger 实例：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Logger LOGGER =
      LoggerFactory.getLogger(HelloWorldController<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
</code></pre>
<p>这个过程底层会查找 LoggerContext 维护的缓存（loggerCache，Map&lt;String, Logger&gt; 类型，其中 Key 是 Logger 实例的名字，Value 为相应 Logger 实例）。如果 loggerCache 中存在相应 Logger 实例，会直接返回；否则会创建相应的 Logger 实例并返回，同时也会将该新建的 Logger 实例缓存到 loggerCache 中，也就是说，同名的 Logger 实例全局只有一个实例。另外，在新建 Logger 实例时，会同时把 loggerCache 中不存在的父 Logger 实例都创建好。</p>
<p>Appender 是对日志输出目的地的抽象，在示例中使用的 ConsoleAppender 会将日志打印到控制台，实践中常用的 FileAppender、RollingFileAppender 等会将日志输出到 log 文件中，还有 Appender 可以将日志输出到 MySQL 等持久化存储中，这里不再一一列举 。</p>
<p>一个 Logger 实例上可以绑定多个 Appender 实例，当在 Logger 实例上产生有效的日志记录请求时，日志记录请求会被发送到所有绑定的 Appender 实例上，然后由 Appender 实例进行输出。另外，Logger 实例上绑定的 Appender 实例还可以继承自上层 Logger 实例的 Appender 绑定。</p>
<p>在老版本的 Logback 中， Appender 会通过 Layout 将日志事件转换成字符串，然后输出到 java.io.Writer 中，实现控制日志输出格式的目的。在新版本的 Logback 中，Appender 不再直接使用 Layout，而是使用 Encoder 实现日志事件到字节数组的转换。Encoder 同时会将转换后的字节数组输出到 Appender 维护的 Outputstream 中。</p>
<p>最常用的 Encoder 实现是 PatternLayoutEncoder，继承关系如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/04/26/CgqCHl6zvSWAA-ZRAAB1DP_LB58112.png" alt="image (9).png"></p>
<p>从 LayoutWrapperEncoder 中 encode() 方法的实现就可以看出，上述 Encoder 底层还是依赖 Layout 确定日志的格式：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">byte</span>[] encode(E event) {
    String txt = layout.doLayout(event); <span class="hljs-comment">// 依赖Layout将日志事件转换字符串</span>
    <span class="hljs-keyword">return</span> convertToBytes(txt); <span class="hljs-comment">// 将字符串转换成字节数组</span>
}
</code></pre>
<p>PatternLayoutEncoder 底层就是直接依赖 PatternLayout 确定日志格式的。当然，我们可以使用 LayoutWrappingEncoder 并指定其他自定义的 Layout ，实现自定义格式的日志。</p>
<p>那 PatternLayout 是如何根据指定的日志输出格式呢？在示例中， 标签下会配置 标签，其中指定了日志的格式。在 PatternLayoutBase 初始化的时候，会解析 字符串，并创建相应的 Converter，其中每个占位符对应一个 Converter，相关代码片段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">// 解析pattern字符串</span>
    Parser&lt;E&gt; p = <span class="hljs-keyword">new</span> Parser&lt;E&gt;(pattern);
    Node t = p.parse();
    <span class="hljs-comment">// 根据解析后的pattern创建Converter链表</span>
    <span class="hljs-keyword">this</span>.head = p.compile(t, getEffectiveConverterMap());
    ... ... <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p>Logback 自带的 Converter 实现都在 PatternLayout.defaultConverterMap 集合之中，先来展示了部分 Converter 的功能：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">static</span> {
    <span class="hljs-comment">// DateConverter处理pattern字符串中的"%d"或是"%date"占位符</span>
    defaultConverterMap.put(<span class="hljs-string">"d"</span>, DateConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    defaultConverterMap.put(<span class="hljs-string">"date"</span>, DateConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    <span class="hljs-comment">// ThreadConverter处理pattern字符串中的"%t"或是"%thread"占位符</span>
    defaultConverterMap.put(<span class="hljs-string">"t"</span>, ThreadConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    defaultConverterMap.put(<span class="hljs-string">"thread"</span>, 
         ThreadConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    <span class="hljs-comment">// MessageConverter处理"%m"、"%msg"、"message"占位符</span>
    defaultConverterMap.put(<span class="hljs-string">"m"</span>, MessageConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    defaultConverterMap.put(<span class="hljs-string">"msg"</span>, MessageConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    defaultConverterMap.put(<span class="hljs-string">"message"</span>, 
         MessageConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    <span class="hljs-comment">// 省略其他占位符对应的Converter</span>
}
</code></pre>
<p>Converter 的核心是 convert() 方法，它负责从日志事件中提取相关信息填充占位符，例如， DateConverter.convert() 方法的实现就是获取日志时间来填充 %d（或 %date）占位符：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">convert</span><span class="hljs-params">(ILoggingEvent le)</span> </span>{
    <span class="hljs-keyword">long</span> timestamp = le.getTimeStamp(); <span class="hljs-comment">// 获取日志事件</span>
    <span class="hljs-keyword">return</span> cachingDateFormatter.format(timestamp); <span class="hljs-comment">// 格式化</span>
}
MessageConverter 就是获取日志格式化信息来填充 %m、%msg 或 %message 占位符：
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">convert</span><span class="hljs-params">(ILoggingEvent event)</span> </span>{
    <span class="hljs-keyword">return</span> event.getFormattedMessage();
}
</code></pre>
<h4>toolkit-logback-1.x</h4>
<p>了解了 Logback 日志框架的核心概念之后，我们回到 demo-webapp 中的 logback.xml 配置文件，这里使用的 Encoder 实现是 LayoutWrappingEncoder，其中指定的 Layout 实现为 SkyWalking 提供的自定义 Layout 实现 —— TraceIdPatternLogbackLayout，它继承了 PatternLayout 并向 defaultConverterMap 中注册了 %tid 占位符对应的 Converter，具体代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TraceIdPatternLogbackLayout</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">PatternLayout</span> </span>{
    <span class="hljs-keyword">static</span> {
        defaultConverterMap.put(<span class="hljs-string">"tid"</span>, 
             LogbackPatternConverter<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())</span>;
    }
}
</code></pre>
<p>LogbackPatternConverter 中的 convert() 方法实现直接返回了 "TID: N/A"。</p>
<p>下面我们跳转到 apm-toolkit-logback-1.x-activation 模块，其 skywalking-plugin.def 文件中指定的 LogbackPatternConverterActivation 会拦截 LogbackPatternConverter 的 convert() 方法，并由 PrintTraceIdInterceptor 进行增强。PrintTraceIdInterceptor.afterMethod() 方法实现中会用当前的 Trace ID 替换 "TID: N/A"返回值：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">afterMethod</span><span class="hljs-params">(EnhancedInstance objInst, Method method, 
    Object[] allArguments, Class&lt;?&gt;[] argumentsTypes, Object ret)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-string">"TID:"</span> + ContextManager.getGlobalTraceId(); <span class="hljs-comment">// 获取 Trace ID</span>
}
</code></pre>
<h3>总结</h3>
<p>本课时重点介绍了 SkyWalking 中 application-toolkit 工具箱的核心原理。首先介绍了 toolkit-trace 模块中 @Trace 注解、 TraceContext 以及 ActiveSpan 工具类的使用方式，然后深入介绍了它们的核心实现。接下来，通过示例介绍了 SkyWalking 与 Logback 日志框架集成方式，深入分析了 Logback 日志框架的核心概念，最后深入介绍了 toolkit-trace-activation 模块的核心原理。</p>

---

### 精选评论


