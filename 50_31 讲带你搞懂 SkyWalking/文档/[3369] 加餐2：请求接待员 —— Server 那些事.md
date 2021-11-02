<p>我在介绍 SkyWalking Agent 核心原理时提到，用户可以在 agent.config 文件中的 collector.backend_service 配置项指定多个 OAP 服务的地址（逗号分隔），SkyWalking Agent 会切分该配置项，得到 OAP 服务列表，然后从其中随机选择一个 OAP 服务创建长连接，实现后续的数据上报。</p>
<p>在后端的 OAP 服务实例中，会启动一个 Server 来监听 Agent 发起的连接，本课时我们就来一起看看 Server 组件的具体实现。</p>
<h4>Server 核心实现</h4>
<p>Server 接口以及实现类位于 server-library 模块下的 library-server 子模块中，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GH8qAYLAAAAB7Hh0Zq6U618.png" alt="image (4).png"></p>
<p>这里有两个核心接口： Server 接口和 ServerHandler 接口。Server 接口有 GRPCServer 和 JettyServer 两个实现类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/9D/Ciqc1F7GH9GAf3gRAAAeem8rA0s935.png" alt="Server继承关系.png"></p>
<ul>
<li><strong>GRPCServer</strong> 用于接收 SkyWalking Agent 发送的 gRPC 请求。正如前面课时介绍的那样， SkyWalking 6.x 中的 Trace 上报、JVM 监控上报、服务以及服务实例注册请求、心跳请求都是通过 gRPC 请求实现的。</li>
<li><strong>JettyServer</strong> 用于接收 SkyWalking Agent 以及用户的 Http 请求。在 SkyWalking 5.x 版本中，上述交互还可以通过 Http 请求完成。另外，用户从 SkyWalking Rocketbot 界面发起的请求，也是由 JettyServer 处理的。</li>
</ul>
<p>在 GRPCServer 实现中首先会根据配置初始化 NettyServerBuilder 对象（其中会指定服务监听的地址和端口，以及每个连接上的并发请求数等参数），然后创建并启动 io.grpc.Server 接收gRPC 请求。gRPC 的 Java 实现底层是依靠 Netty 实现的，Netty 是一个高性能的开源网络库，由于 Java 本身的 NIO API 使用起来比较麻烦，而 Netty 底层封装了 Java NIO 并对外提供了简单易用的 API，所以很多开源软件的网络模块都是使用 Netty 实现的。</p>
<p>下面是 GRPCServer 中最核心的代码，这里的 initialize() 方法和 start() 方法是在 Server 接口中定义的：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initialize</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">// 创建NettyServerBuilder，设置最大请求并发数、每个请求的最大长度等参数</span>
    InetSocketAddress address = <span class="hljs-keyword">new</span> InetSocketAddress(host, port);
    nettyServerBuilder = NettyServerBuilder.forAddress(address);
    nettyServerBuilder = nettyServerBuilder
     .maxConcurrentCallsPerConnection(maxConcurrentCallsPerConnection)
      .maxMessageSize(maxMessageSize);
}

<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ServerException </span>{
    server = nettyServerBuilder.build(); <span class="hljs-comment">// 创建并启动io.grpc.Server</span>
    server.start();
}
</code></pre>
<p>接下来看 JettyServer 实现，它底层依赖 Jetty 实现对 Http 请求的处理。Jetty 是一个 Servlet 容器，它以 jar 包的方式发布，常被用作嵌入式的 Web 容器。JettyServer 的核心实现如下，这也是 Java 代码中直接启动 Jetty 实例的基本流程：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initialize</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-comment">// 创建org.eclipse.jetty.server.Server对象</span>
    server = <span class="hljs-keyword">new</span> org.eclipse.jetty.server.Server(
        <span class="hljs-keyword">new</span> InetSocketAddress(host, port));
    <span class="hljs-comment">// 创建ServletContextHandler对象，contextPath是其处理的路径</span>
    servletContextHandler = <span class="hljs-keyword">new</span> ServletContextHandler(NO_SESSIONS);
    servletContextHandler.setContextPath(contextPath);
    server.setHandler(servletContextHandler);
}

<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ServerException </span>{
    server.start();
}
</code></pre>
<p>最后，无论是 GRPCServer 处理 gRPC 请求的逻辑，还是 JettyServer 处理 Http 请求的逻辑，都是封装在了 ServerHandler 实现之中的。我们可以通过两者的 addHandler() 方法，为指定请求添加相应的 ServerHandler 实现。</p>
<p>例如，前面介绍的 Agent 上报 Trace 的 gRPC 请求，是由 TraceSegmentReportServiceHandler 这个 GRPCHandler 进行处理的，它继承了 PB 生成的服务端辅助类，也同时实现了 GRPCHandler 接口，如下图所示。这里的 ServerHandler 接口和 GRPCHandler 接口中没有定义任何方法，只是一个标识而已。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/9D/Ciqc1F7GH9yAJDw0AADhn-9G47E874.png" alt="image (5).png"></p>
<p>通过下图我们可以看出， SkyWalking Agent 发出的每种 gRPC 请求，都有一个对应的 GRPCHandler 实现，这些实现同时也继承了 PB 生成的服务端辅助类，实现了 BindableService接口。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GH-SAfec0AAHpmMYGuCk769.png" alt="image (6).png"></p>
<p>在后面介绍 OAP 中其他上层模块时会看到，在启动时都会将对应的 GRPCHandler 实现（也是 BindableServer 实现）添加到 GRPCServer 上。这样，GRPCServer 在收到 gRPC 请求时才能找到相应的处理模块，这些 GRPCHandler 实现（BindableServer 实现）即为相应上层模块入口。</p>
<p>JettyServer 也是类似的，在后面介绍 SkyWalking Rocketbot 查询请求的相关模块时会看到，前端的请求是通过 GraphQL​QueryHandler 进行处理的，它本身是个 HttpServlet 实现，同时实现了 ServerHandler 接口，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GH-2AT3uCAADSyb16jy0664.png" alt="GraphQLQueryHandler.png"></p>
<p>在低版本中，SkyWalking Agent 与后端 OAP 的交互还可以通过 Http 请求完成，每种类型的请求都对应一个 JettyHandler 实现，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GH_SALGuQAAI7EvvVlKg105.png" alt="JettyHandler.png"></p>
<p>如果 OAP 中的一个模块需要处理 Http 请求，就需要提供一个 JettyHandler 实现并注册到 JettyServer 中。</p>
<h4>相关 Service 实现</h4>
<p>通过上一节的介绍我们知道，library-server 是一个相对独立的模块，没有绑定任何 SkyWalking 中的概念，所以它是可以单独打成 jar 包给其他应用使用的（前面介绍的 DataCarrier 模块也是如此）。</p>
<p>OAP 其他模块在使用 library-server 模块提供的 Server 组件时，需要对其进行一层封装，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/9E/Ciqc1F7GH_2AY1ESAAA-AcXju3w625.png" alt="image (7).png"></p>
<p>OAP 的 server-core 模块中定义了一个 GRPCHandlerRegister 接口，其实现中封装了一个 GRPCServer，并继承了 Service 接口，这样就将 library-server 模块引入到 OAP 的体系中，server-core 模块同样也为 JettyServer 进行了相应的封装，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GIASAZhbOAAA_KXoo8XI235.png" alt="image (8).png"></p>
<p>最后，这里的 GRPCHandlerRegister、JettyHandlerRegister 接口只会对外暴露底层 Server 的 addHandler() 方法，并没有暴露其他任何方法。</p>
<h4>Server 启动流程</h4>
<p>了解了 Server 的核心实现以及 server-core 模块如何将 Server 组件引入到 OAP 体系之后，我们再来介绍一下 OAP 是如何初始化 Server 组件的。</p>
<p>server-core 模块是 SkyWalking OAP 中的核心模块，其中会初始化很多核心 Module、ModuleProvider 以及 Service。在 server-core 模块中的 ModuleDefine SPI 配置文件中指定了多个 ModuleDefine 实现，如下所示：</p>
<pre><code data-language="java" class="lang-java">org.apache.skywalking.oap.server.core.storage.StorageModule
org.apache.skywalking.oap.server.core.cluster.ClusterModule
org.apache.skywalking.oap.server.core.CoreModule
org.apache.skywalking.oap.server.core.query.QueryModule
org.apache.skywalking.oap.server.core.alarm.AlarmModule
org.apache.skywalking.oap.server.core.exporter.ExporterModule
</code></pre>
<p>可以看到前文分析 ClusterModule 是在 server-core 模块初始化的时候被加载的。通过这些 ModuleDefine 的名称，我们可以大致推测出其他的 Module 的核心功能，例如，StorageModule 负责实现 OAP 底层存储相关的功能，QueryModule 负责实现查询相关的功能，AlarmModule 负责实现告警功能。这些 Module 的具体实现在后面会逐个展开分析。</p>
<p>这里重点来看 CoreModule ，在其 services() 方法返回的 Service 数组中，包含了 GRPCHandlerRegister、JettyHandlerRegister 两个 Service 接口，相关代码片段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> Class[] services() {
    List&lt;Class&gt; classes = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
    addServerInterface(classes);
    <span class="hljs-comment">// 省略向classes集合中添加其他Service接口的代码</span>
    <span class="hljs-keyword">return</span> classes.toArray(<span class="hljs-keyword">new</span> Class[] {});
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addServerInterface</span><span class="hljs-params">(List&lt;Class&gt; classes)</span> </span>{
    classes.add(GRPCHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
    classes.add(JettyHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
}
</code></pre>
<p>也就是说其 ModuleProvider 必须提供这两个 Service 的相关实现。在 server-core 模块的 ModuleProvider SPI 文件中配置了 CoreModuleProvider 这个实现类，在其 prepare() 方法中会创建前文介绍的 GRPCServer 以及 JettyServer 实例并完成底层的初始化操作：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">prepare</span><span class="hljs-params">()</span>  </span>{
&nbsp; &nbsp; grpcServer = <span class="hljs-keyword">new</span> GRPCServer(...); <span class="hljs-comment">// 创建并初始化GRPCServer</span>
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略设置GRPServer的代码</span>
&nbsp; &nbsp; grpcServer.initialize();
&nbsp; &nbsp; <span class="hljs-comment">// 创建并初始化 JettyServer</span>
&nbsp; &nbsp; jettyServer = <span class="hljs-keyword">new</span> JettyServer(...); <span class="hljs-comment">// 省略相关配置</span>
&nbsp; &nbsp; jettyServer.initialize();
&nbsp; &nbsp; <span class="hljs-comment">// GRPCServer封装成到GRPCHandlerRegisterImpl之中，然后注册成</span>
    <span class="hljs-comment">// GRPCHandlerRegister这个Service的实现</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.registerServiceImplementation(GRPCHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>, 
          <span class="hljs-title">new</span> <span class="hljs-title">GRPCHandlerRegisterImpl</span>(<span class="hljs-title">grpcServer</span>))</span>;
&nbsp; &nbsp; <span class="hljs-comment">// JettyServer封装成到JettyHandlerRegisterImpl之中，然后注册成</span>
    <span class="hljs-comment">// JettyHandlerRegister这个Service的实现</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.registerServiceImplementation(JettyHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>, 
          <span class="hljs-title">new</span> <span class="hljs-title">JettyHandlerRegisterImpl</span>(<span class="hljs-title">jettyServer</span>))</span>;
}
</code></pre>
<p>通过前文对 BootstrapFlow 的介绍可知，在 prepare() 方法完成之后，会调用 ModuleProvider 的 start() 方法，CoreModuleProvider 会在其实现中为 GRPCServer 添加两个基础的 GRPCHandler 实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ModuleStartException </span>{
    grpcServer.addHandler(<span class="hljs-keyword">new</span> RemoteServiceHandler(getManager()));
    grpcServer.addHandler(<span class="hljs-keyword">new</span> HealthCheckServiceHandler());
}
</code></pre>
<p>这里简单介绍一下这两个 GRPCHandler 的功能：</p>
<ul>
<li><strong>HealthCheckServiceHandler</strong>：在 Cluster 模块中提供了支持多种第三方服务发展组件的实现，前文介绍的 cluster-zookeeper-plugin 模块使用的 curator-x-discovery 扩展库底层是依赖 Zookeeper 的 Watcher 来监听一个 OAP 服务实例是否可用。但是有的第三方服务发现组件（例如 Consul）会依靠定期健康检查（Health Check）来检查一个 OAP 服务实例是否可用，此时 OAP 就需要保留一个接口来处理健康检查请求，这就是 HealthCheckServiceHandler 的核心功能。对 Consul 感兴趣的同学，可以参考下面两篇文档：</li>
</ul>
<blockquote>
<p>https://www.consul.io/docs/agent/checks.html<br>
https://github.com/grpc/grpc/blob/master/doc/health-checking.md</p>
</blockquote>
<ul>
<li><strong>RemoteServiceHandler</strong>：OAP 集群中各个 OAP 实例节点之间通信的接口，在后面会详细介绍该 GRPCHandler 的实现以及通信方式。</li>
</ul>
<p>CoreModuleProvider 中并没有为 JettyServer 添加任何 JettyHandler 实现，在后续分析其他 ModuleProvider 的 start() 方法时，会看到向 JettyServer 添加 JettyHandler 的行为。</p>
<p>最后，在全部 ModuleProvider 初始化结束之后，BootstrapFlow 会通过 notifyAfterCompleted() 方法通知所有 ModuleProvider 开始对外提供服务，在 CoreModuleProvider 的实现中，其 notifyAfterCompleted() 方法会调用 GRPCServer 和 JettyServer 的start() 方法，启动这两个 Server 。至此，OAP 服务中的 GRPCServer 和 JettyServer 实例启动完毕了。</p>
<h4>sharing-server-plugin 中的 Server 实例</h4>
<p>SkyWalking OAP 需要接收外部请求的地方还是挺多的，例如 Agent 上报的监控数据、 SkyWalking Rocketbot 的查询请求、OAP 集群中节点之间的相互通信，等等。除了 CoreModuleProvider 中会启动 Server 组件之外，sharing-server-plugin 模块中也可以启动单独的一套 Server 实例，并监听独立的端口，这套 Server 实例主要用来接收 Agent 的注册请求、上报的 JVM 监控数据以及 Trace 数据。</p>
<p>与 CoreModule 相同，SharingServerModule（sharing-server-plugin 模块的 Module 实现类）的 services() 方法指明了 SharingServerModuleProvider （sharing-server-plugin 模块的 ModuleProvider 实现类）必须提供 GRPCHandlerRegister 和 JettyHandlerRegister 两个接口的实例。</p>
<p>在 SharingServerModuleProvider 的 prepare() 方法中，会根据配置信息决定是否启动新的 Server 组件，核心代码片段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">prepare</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (config.getGRPCPort() != <span class="hljs-number">0</span>){ <span class="hljs-comment">// 配置了独立端口，则启动独立GRPCServer</span>
&nbsp; &nbsp; &nbsp; &nbsp; grpcServer = <span class="hljs-keyword">new</span> GRPCServer(...);
&nbsp; &nbsp; &nbsp; &nbsp; grpcServer.initialize();
        <span class="hljs-comment">// GRPCServer封装成到GRPCHandlerRegisterImpl之中，然后注册成</span>
        <span class="hljs-comment">// GRPCHandlerRegister这个Service的实现，与CoreModuleProvider相同</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.registerServiceImplementation(GRPCHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>,
           <span class="hljs-title">new</span> <span class="hljs-title">GRPCHandlerRegisterImpl</span>(<span class="hljs-title">grpcServer</span>))</span>;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 未指定独立GRPCServer的端口，则与CoreModule共用一个GRPCServer实例</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.receiverGRPCHandlerRegister = 
            <span class="hljs-keyword">new</span> ReceiverGRPCHandlerRegister();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.registerServiceImplementation(GRPCHandlerRegister<span class="hljs-class">.<span class="hljs-keyword">class</span>, 
          <span class="hljs-title">receiverGRPCHandlerRegister</span>)</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 对JettyServer的处理相同，不再展开</span>
}
</code></pre>
<p>可见，在未指定独立端口的时候，sharing-server-plugin 模块并没有启动新 Server，而是和 server-core 模块共用一套 Server 实例，这里的 ReceiverGRPCHandlerRegister 就是对 GRPCHandlerRegister 的封装，如下图所示，添加的 GRPCHandler 也都会添加到同一个 GRPCServer 实例上：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GIBaARC57AAJ11UYHvq8272.png" alt="image (9).png"></p>
<p>如果使用了独立的 Server 实例，则与 CoreModuleProvider 相同，会在SharingServerModuleProvider 的 notifyAfterCompleted() 方法中启动，代码不再重复。</p>
<h4>Server 的相关配置</h4>
<p>本课时最后，看一下 application.yml 中与 Server 相关的配置项。下图展示了 CoreModuleProvider 中启动的 Server 实例的配置以及 CoreModuleConfig 中的对应字段。 restHost、restPort、restContextPath 是 JettyServer 监听的 host 地址、端口号以及处理的 URL Path，gRPCHost、gRPCPort 是 GRPCServer 监听的 host 地址和端口号。maxConcurrentCallsPerConnection、maxMessageSize 是 GRPCServer 中单个连接的最大请求数以及单个消息的最大长度。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/9E/Ciqc1F7GIB6ANSDJABPfQZGq5oU867.png" alt="image (10).png"></p>
<p>下图展示了 SharingServerModuleProvider 中启动的 Server 实例的配置以及 SharingServerConfig 中的对应字段，具体含义与 server-core 模块相同，不再重复。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A9/CgqCHl7GICWAd2pVAA4OF-xGTK0682.png" alt="image (11).png"></p>
<h4>总结</h4>
<p>本课时深入介绍了 OAP 中接收 gRPC 请求的 GRPCServer 组件以及接收 Http 请求的 JettyServer 组件，分析了 server-core 模块如何对 library-server 模块的 Server 进行封装并引入到 OAP 体系。然后剖析了 OAP 在初始化流程中启动 Server 组件的核心流程（其中包括 server-core 以及 sharing-server-plugin 模块中的两组 Server），还介绍了这两组 Server 实例对应的配置信息。</p>

---

### 精选评论


