<p>在&nbsp;08 课时“SkyWalking Agent 启动流程剖析”中我详细介绍了 ServiceManager 加载并初始化 BootService 实现的核心逻辑。下图展示了 BootService 接口的所有实现类，本课时将深入分析这些 BootService 实现类的具体逻辑：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/85/D6/Cgq2xl6OzXqACRPAAAEm5IQEH5Y241.png" alt=""></p>
<h3>网络连接管理</h3>
<p>在前面的介绍中提到 SkyWalking Agent 会定期将收集到的 JVM 监控和 Trace 数据定期发送到后端的 OAP 集群，GRPCChannelManager 负责维护 Agent 与后端 OAP 集群通信时使用的网络连接。这里首先说一下 gRPC 里面的两个组件：</p>
<ul>
<li>**ManagedChanne l：它是 gRPC 客户端的核心类之一，它逻辑上表示一个 Channel，底层持有一个 TCP 链接，并负责维护此连接的活性。也就是说，在 RPC 调用的任何时机，如果检测到底层连接处于关闭状态（terminated），将会尝试重建连接。通常情况下，我们不需要在 RPC 调用结束后就关闭 Channel ，该 Channel 可以被一直重用，直到整个客户端程序关闭。当然，我们可以在客户端内以连接池的方式使用多个 ManagedChannel ，在每次 RPC 请求时选择使用轮训或是随机等算法选择一个可用的 Channel，这样可以提高客户端整体的并发能力。</li>
<li>**ManagedChannelBuilder：**它负责创建客户端 Channel，ManagedChannelBuilder 使用了 provider 机制，具体是创建了哪种 Channel 由 provider 决定，常用的 ManagedChannelBuilder 有三种：NettyChannelBuilder、OkHttpChannelBuilder、InProcessChannelBuilder，如下图所示：</li>
</ul>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0C/C0/Ciqah16OzXqAYIu0AABBDZDMT8c843.png" alt=""></p>
<p>在 SkyWalking Agent 中用的是 NettyChannelBuilder，其创建的 Channel 底层是基于 Netty 实现的。OkHttpChannelBuilder 创建的 Channel 底层是基于 OkHttp 库实现的。InProcessChannelBuilder 用于创建进程内通信使用的 Channel。</p>
<p>SkyWalking 在 ManagedChannel 的基础上封装了自己的 Channel 实现 —— GRPCChannel ，可以添加一些装饰器，目前就只有一个验证权限的修饰器，实现比较简单，这里就不展开分析了。</p>
<p>介绍完基础知识之后，回到 GRPCChannelManager，其中维护了一个 GRPCChannel 连接以及注册在其上的 Listener 监听，另外还会维护一个后台线程定期检测该 GRPCChannel 的连接状态，如果发现连接断开，则会进行重连。GRPCChannelManager 的核心字段如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;封装了上面介绍的gRPC&nbsp;Channel</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;GRPCChannel&nbsp;managedChannel&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;&nbsp;
<span class="hljs-comment">//&nbsp;定时检查&nbsp;GRPCChannel的连接状态重连gRPC&nbsp;Server的定时任务</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;ScheduledFuture&lt;?&gt;&nbsp;connectCheckFuture;&nbsp;
<span class="hljs-comment">//&nbsp;是否重连。当&nbsp;GRPCChannel断开时会标记&nbsp;reconnect为&nbsp;true，后台线程会根据该标</span>
<span class="hljs-comment">//&nbsp;识决定是否进行重连</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;reconnect&nbsp;=&nbsp;<span class="hljs-keyword">true</span>;&nbsp;
<span class="hljs-comment">//&nbsp;加在&nbsp;Channel上的监听器，主要是监听&nbsp;Channel的状态变化</span>
<span class="hljs-keyword">private</span>&nbsp;List&lt;GRPCChannelListener&gt;&nbsp;listeners;
&nbsp;<span class="hljs-comment">//&nbsp;可选的&nbsp;gRPC&nbsp;Server集合，即后端OAP集群中各个OAP实例的地址</span>
<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;List&lt;String&gt;&nbsp;grpcServers;
</code></pre>
<p>前文介绍 ServiceManager 时提到，Agent 启动过程中会依次调用 BootService 实现的 prepare() 方法 → boot() 方法 → onComplete() 方法之后，才能真正对外提供服务。GRPCChannelManager 的 prepare() 方法 、onComplete() 方法都是空实现，在 boot() 方法中首先会解析 agent.config 配置文件指定的后端 OAP 实例地址初始化 grpcServers 字段，然后会初始化这个定时任务，初次会立即执行，之后每隔 30s 执行一次，具体执行的就是 GRPCChannelManager.run() 方法，其核心逻辑是检测链接状态并适时重连：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(reconnect&nbsp;&amp;&amp;&nbsp;grpcServers.size()&nbsp;&gt;&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;根据配置，连接指定OAP实例的IP和端口</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;managedChannel&nbsp;=&nbsp;GRPCChannel.newBuilder(ipAndPort[<span class="hljs-number">0</span>],&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Integer.parseInt(ipAndPort[<span class="hljs-number">1</span>]))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.addManagedChannelBuilder(<span class="hljs-keyword">new</span>&nbsp;StandardChannelBuilder())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.addManagedChannelBuilder(<span class="hljs-keyword">new</span>&nbsp;TLSChannelBuilder())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.addChannelDecorator(<span class="hljs-keyword">new</span>&nbsp;AuthenticationDecorator())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;notify()方法会循环调用所有注册在当前连接上的GRPCChannelListener实</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;例(记录在listeners集合中)的statusChanged()方法，通知它们连接创建</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;成功的事件</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notify(GRPCChannelStatus.CONNECTED);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置&nbsp;reconnect字段为false，暂时不会再重建连接了</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;reconnect&nbsp;=&nbsp;<span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>GRPCChannelListener 是一个监听器接口，有多个需要发送网络请求的 BootService 实现类同时实现了该接口，如下图所示，后面会详细介绍这些 BootService 实现类的具体功能。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/85/D6/Cgq2xl6OzXqAXhA7AACNDBlVUrQ730.png" alt=""></p>
<p>最后，GRPCChannelManager 对外提供了 reportError() 方法，在其他依赖该网络连接的 BootService 实现发送请求失败时，可以通过该方法将 reconnect 字段设置为 true，并由后台线程重新创建 GRPCChannel。</p>
<h3>注册协议及实现</h3>
<p>介绍完 Agent 与后端 OAP 集群基本的连接方式之后，还需要了解 Agent 和 Server 交互的流程和协议。在 Agent 与后端 OAP 创建连接成功后的第一件事是进行注册流程。</p>
<p>首先来介绍注册协议涉及的 proto 的定义—— Register.proto 文件，其位置如下：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0C/C0/Ciqah16OzXuAO6enAAJXETLoQmo373.png" alt=""></p>
<p>Register.proto 中定义了 Register 服务，这里先关注 doServiceRegister() 和 doServiceInstanceRegister() 两个 RPC 接口，如下所示：</p>
<pre><code data-language="java" class="lang-java">service&nbsp;Register&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;rpc&nbsp;doServiceRegister(Services)returns(ServiceRegisterMapping)&nbsp;{}

&nbsp;&nbsp;&nbsp;&nbsp;rpc&nbsp;doServiceInstanceRegister&nbsp;(ServiceInstances)&nbsp;returns
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(ServiceInstanceRegisterMapping)&nbsp;{}

&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;还有三个方法，这里省略一下，后面会介绍
}
</code></pre>
<ul>
<li><strong>doServiceRegister() 接口</strong>：将服务（Service）的名称注册到后端的 OAP 集群。参数 Services 中会携带当前服务的名称（其中还可以附加一些 KV 格式的信息）；返回的 ServiceRegisterMapping 其实是多个 KV，其中就包含了后端 OAP 服务端生成的ServiceId。</li>
<li><strong>doServiceInstanceRegister() 接口</strong>：将服务实例（ServiceInstance）的名称注册到后端 OAP 集群。参数 ServiceInstances 中会携带服务实例（ServiceInstance）的名称，以及 serviceId、时间戳等信息；返回的 ServiceInstanceRegisterMapping 本质也是一堆 KV，其中包含 OAP 为服务实例生成的 ServiceInstanceId。</li>
</ul>
<blockquote>
<p>Service、ServiceInstance 的概念可以回顾本课程的第一课时“Skywalking 初体验”。</p>
</blockquote>
<p>与 Service 注册流程相关的 BootService 实现是 ServiceAndEndpointRegisterClient。ServiceAndEndpointRegisterClient 实现了 GRPCChannelListener 接口，在其 prepare() 方法中首先会将其注册到 GRPCChannelManager 来监听网络连接，然后生成当前 ServiceInstance 的唯一标识：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">prepare</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Throwable&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;查找&nbsp;GRPCChannelManager实例(前面介的ServiceManager.bootedServices</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;集合会按照类型维护BootService实例，查找也是查找该集合)，然后将&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;ServiceAndEndpointRegisterClient注册成Listener</span>
&nbsp;&nbsp;&nbsp;&nbsp;ServiceManager.INSTANCE.findService(GRPCChannelManager<span class="hljs-class">.<span class="hljs-keyword">class</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.<span class="hljs-title">addChannelListener</span>(<span class="hljs-title">this</span>)</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;确定INSTANCE_UUID，优先使用gent.config文件中配置的INSTANCE_UUID，</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;若未配置则随机生成</span>
&nbsp;&nbsp;&nbsp;&nbsp;INSTANCE_UUID&nbsp;=&nbsp;StringUtil.isEmpty(Config.Agent.INSTANCE_UUID)&nbsp;?&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UUID.randomUUID().toString().replaceAll(<span class="hljs-string">"-"</span>,&nbsp;<span class="hljs-string">""</span>)&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Config.Agent.INSTANCE_UUID;
}
</code></pre>
<p>在网络连接建立之后，会通知其 statusChanged() 方法更新 registerBlockingStub 字段：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">statusChanged</span><span class="hljs-params">(GRPCChannelStatus&nbsp;status)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(GRPCChannelStatus.CONNECTED.equals(status))&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;网络连接创建成功时，会依赖该连接创建两个stub客户端</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Channel&nbsp;channel&nbsp;=&nbsp;ServiceManager.INSTANCE.findService(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GRPCChannelManager<span class="hljs-class">.<span class="hljs-keyword">class</span>).<span class="hljs-title">getChannel</span>()</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registerBlockingStub&nbsp;=&nbsp;RegisterGrpc.newBlockingStub(channel);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceInstancePingStub&nbsp;=&nbsp;ServiceInstancePingGrpc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.newBlockingStub(channel);
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;网络连接断开时，更新两个stub字段（它们都是volatile修饰）</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registerBlockingStub&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceInstancePingStub&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.status&nbsp;=&nbsp;status;&nbsp;<span class="hljs-comment">//&nbsp;更新status字段，记录网络状态</span>
}
</code></pre>
<p>registerBlockingStub 是 gRPC 框架生成的一个客户端辅助类，可以帮助我们轻松的完成请求的序列化、数据发送以及响应的反序列化。gRPC 的基础使用这里不再展开。</p>
<p>ServiceAndEndpointRegisterClient 也同时实现了 Runnable 接口，<span class="colour" style="color:rgb(51, 51, 51)"><span class="font" style="font-family:&quot;Microsoft YaHei&quot;, sans-serif">在 boot() 方法中会启动一个定时任务，默认每 3s 执行一次其 run() 方法，该定时任务首先会通过 doServiceRegister() 接口完成 Service 注册，相关实现片段如下：</span></span></p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">while</span>&nbsp;(GRPCChannelStatus.CONNECTED.equals(status)&nbsp;&amp;&amp;&nbsp;shouldTry)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;shouldTry&nbsp;=&nbsp;<span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;检测当前Agent是否已完成了Service注册</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(RemoteDownstreamConfig.Agent.SERVICE_ID&nbsp;==&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DictionaryUtil.nullValue())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(registerBlockingStub&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;第二次检查网络状态</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;通过doServiceRegister()接口进行Service注册</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ServiceRegisterMapping&nbsp;serviceRegisterMapping&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registerBlockingStub.doServiceRegister(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Services.newBuilder().addServices(Service.newBuilder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setServiceName(Config.Agent.SERVICE_NAME)).build());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(KeyIntValuePair&nbsp;registered&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceRegisterMapping.getServicesList())&nbsp;{<span class="hljs-comment">//&nbsp;遍历所有KV</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(Config.Agent.SERVICE_NAME
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.equals(registered.getKey()))&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RemoteDownstreamConfig.Agent.SERVICE_ID&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registered.getValue();&nbsp;<span class="hljs-comment">//&nbsp;记录serviceId</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置shouldTry，紧跟着会执行服务实例注册</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shouldTry&nbsp;=&nbsp;<span class="hljs-keyword">true</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;后续会执行服务实例注册以及心跳操作</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;...&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>通过分析这段代码，我们可以知道：</p>
<ul>
<li>如果在 agent.config 配置文件中直接配置了 serviceId 是无需进行服务注册的。</li>
<li>ServiceAndEndpointRegisterClient 会根据监听 GRPCChannel 的连接状态，决定是否发送服务注册请求、服务实例注册请求以及心跳请求。</li>
</ul>
<p>完成服务（Service）注册之后，ServiceAndEndpointRegisterClient 定时任务会立即进行服务实例（ServiceInstance）注册，具体逻辑如下，逻辑与服务注册非常类似：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">while</span>&nbsp;(GRPCChannelStatus.CONNECTED.equals(status)&nbsp;&amp;&amp;&nbsp;shouldTry)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(RemoteDownstreamConfig.Agent.SERVICE_ID&nbsp;==&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DictionaryUtil.nullValue())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;省略服务注册逻辑</span>
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(RemoteDownstreamConfig.Agent.SERVICE_INSTANCE_ID&nbsp;==&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DictionaryUtil.nullValue())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;调用&nbsp;doServiceInstanceRegister()接口，用serviceId和</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;INSTANCE_UUID换取SERVICE_INSTANCE_ID</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ServiceInstanceRegisterMapping&nbsp;instanceMapping&nbsp;=
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registerBlockingStub.doServiceInstanceRegister(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ServiceInstances.newBuilder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.addInstances(ServiceInstance.newBuilder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setServiceId(RemoteDownstreamConfig.Agent.SERVICE_ID)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;除了serviceId，还会传递uuid、时间戳以及系统信息之类的</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setInstanceUUID(INSTANCE_UUID)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setTime(System.currentTimeMillis())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.addAllProperties(OSUtil.buildOSInfo())).build());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(KeyIntValuePair&nbsp;serviceInstance&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;instanceMapping.getServiceInstancesList())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(INSTANCE_UUID.equals(serviceInstance.getKey()))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;记录serviceIntanceId</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;RemoteDownstreamConfig.Agent.SERVICE_INSTANCE_ID&nbsp;=
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceInstance.getValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="hljs-keyword">else</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;省略心跳的相关逻辑</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<h3>心跳协议及实现</h3>
<p>在完成服务注册以及服务实例操作之后，ServiceAndEndpointRegisterClient 会开始定期发送心跳请求，通知后端 OAP 服务当前 Agent 在线。心跳涉及一个新的 gRPC 接口如下（定义在 InstancePing.proto 文件中）：</p>
<pre><code data-language="java" class="lang-java">service&nbsp;ServiceInstancePing&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">rpc&nbsp;<span class="hljs-title">doPing</span>&nbsp;<span class="hljs-params">(ServiceInstancePingPkg)</span>&nbsp;<span class="hljs-title">returns</span>&nbsp;<span class="hljs-params">(Commands)</span>&nbsp;</span>{}
}
</code></pre>
<p>ServiceAndEndpointRegisterClient 中心跳请求相关的代码片段如下，心跳请求会携带serviceInstanceId、时间戳、INSTANCE_UUID 三个信息：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">while</span>&nbsp;(GRPCChannelStatus.CONNECTED.equals(status)&nbsp;&amp;&amp;&nbsp;shouldTry)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(Agent.SERVICE_ID&nbsp;==&nbsp;DictionaryUtil.nullValue())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;省略服务注册逻辑</span>
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(Agent.SERVICE_INSTANCE_ID&nbsp;==&nbsp;DictionaryUtil.nullValue())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;省略服务实例注册逻辑</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="hljs-keyword">else</span>{&nbsp;<span class="hljs-comment">//&nbsp;并没有对心跳请求的响应做处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceInstancePingStub.doPing(ServiceInstancePingPkg
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.newBuilder().setServiceInstanceId(SERVICE_INSTANCE_ID)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setTime(System.currentTimeMillis())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setServiceInstanceUUID(INSTANCE_UUID).build());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<h3>Endpoint、NetWorkAddress 同步</h3>
<p>在前文介绍 SkyWalking 基本使用时看到，Trace 数据中包含了请求的 URL 地址、RPC 接口名称、HTTP 服务或 RPC 服务的地址、数据库的 IP 以及端口等信息，这些信息在整个服务上线稳定之后，不会经常发生变动。而在海量 Trace 中包含这些重复的字符串，会非常浪费网络带宽以及存储资源，常见的解决方案是将字符串映射成数字编号并维护一张映射表，在传输、存储时使用映射后的数字编号，在展示时根据映射表查询真正的字符串进行展示即可。这类似于编码、解码的思想。SkyWalking 也是如此处理 Trace 中重复字符串的。</p>
<p>SkyWalking 中有两个 DictionaryManager：</p>
<ul>
<li><strong>EndpointNameDictionary</strong>：用于同步 Endpoint 字符串的映射关系。</li>
<li><strong>NetworkAddressDictionary</strong>：用于同步网络地址的映射关系。</li>
</ul>
<p>EndpointNameDictionary 中维护了两个集合：</p>
<ul>
<li><strong>endpointDictionary</strong>：记录已知的 Endpoint 名称映射的数字编号。</li>
<li><strong>unRegisterEndpoints</strong>：记录了未知的 Endpoint 名称。</li>
</ul>
<p>EndpointNameDictionary 对外提供了两类操作，一个是查询操作（核心实现在 find0() 方法），在查询时首先去 endpointDictionary 集合查找指定 Endpoint 名称是否已存在相应的数字编号，若存在则正常返回数字编号，若不存在则记录到 unRegisterEndpoints 集合中。为了防止占用过多内存导致频繁 GC，endpointDictionary 和 unRegisterEndpoints 集合大小是有上限的（默认两者之和为 10^7）。</p>
<p>另一个操作是同步操作（核心实现在 syncRemoteDictionary() 方法），在 ServiceAndEndpointRegisterClient 收到心跳响应之后，会将 unRegisterEndpoints 集合中未知的 Endpoint 名称发送到 OAP 集群，然后由 OAP 集群统一分配相应的数字编码，具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">syncRemoteDictionary</span><span class="hljs-params">(RegisterGrpc.RegisterBlockingStub&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceNameDiscoveryServiceBlockingStub)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;创建请求，每个Endpoint中都封装了&nbsp;Endpoint名称以及关联的serviceId</span>
&nbsp;&nbsp;&nbsp;&nbsp;Endpoints.Builder&nbsp;builder&nbsp;=&nbsp;Endpoints.newBuilder();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(OperationNameKey&nbsp;operationNameKey&nbsp;:&nbsp;unRegisterEndpoints)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Endpoint&nbsp;endpoint&nbsp;=&nbsp;Endpoint.newBuilder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setServiceId(operationNameKey.getServiceId())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setEndpointName(operationNameKey.getEndpointName())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setFrom(operationNameKey.getSpanType())&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.addEndpoints(endpoint);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;发送同步请求</span>
&nbsp;&nbsp;&nbsp;&nbsp;EndpointMapping&nbsp;serviceNameMappingCollection&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceNameDiscoveryServiceBlockingStub
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.doEndpointRegister(builder.build());
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(EndpointMappingElement&nbsp;element&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;serviceNameMappingCollection.getElementsList())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将返回的映射关系，记录到&nbsp;endpointDictionary集合中，并从&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;unRegisterEndpoints集合中删除Endpoint信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;OperationNameKey&nbsp;key&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;OperationNameKey(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;element.getServiceId(),&nbsp;element.getEndpointName(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DetectPoint.server.equals(element.getFrom()),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DetectPoint.client.equals(element.getFrom()));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;unRegisterEndpoints.remove(key);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endpointDictionary.put(key,&nbsp;element.getEndpointId());
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>NetworkAddressDictionary 实现与 EndpointNameDictionary 类似，就留给你自己进行分析，这里不再展开。</p>
<h3>收集 JVM 监控</h3>
<p>在前面介绍 SkyWalking Rocketbot 时看到 SkyWalking 可以监控服务实例的 CPU、堆内存使用情况以及 GC 信息，这些信息都是通过 JVMService 收集的。JVMService 也实现了 BootService 接口。JVMService 的结构大致如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/85/D6/Cgq2xl6OzXuAWM3MAAHZFmJeMwM515.png" alt=""></p>
<p>在 JVMService 中会启动一个独立 collectMetric 线程请求 MXBean 来收集 JVM 的监控数据，然后将监控数据保存到 queue 队列（LinkedBlockingQueue 类型）中缓存，之后会启动另一个线程读取 queue 队列并将监控数据通过 gRPC 请求发送到后端的 OAP 集群。</p>
<p>在 JVMService.boot() 方法的实现中会初始化 queue 缓冲队列以及 Sender 对象，如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">prepare</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Throwable&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;queue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedBlockingQueue(Config.Jvm.BUFFER_SIZE);
&nbsp;&nbsp;&nbsp;&nbsp;sender&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Sender();
&nbsp;&nbsp;&nbsp;&nbsp;ServiceManager.INSTANCE.findService(GRPCChannelManager<span class="hljs-class">.<span class="hljs-keyword">class</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.<span class="hljs-title">addChannelListener</span>(<span class="hljs-title">sender</span>)</span>;&nbsp;<span class="hljs-comment">//&nbsp;sender会监听底层的连接状态</span>
}
</code></pre>
<p>在 Sender.run() 方法中封装了读取 queue 队列以及发送 gRPC 请求的逻辑，其中会根据底层的 GRPCChannel 连接状态、当前服务以及服务实例的注册情况决定该任务是否执行。Sender.run() 方法的核心逻辑如下（省略全部边界检查逻辑）：</p>
<pre><code data-language="java" class="lang-java">JVMMetricCollection.Builder&nbsp;builder&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JVMMetricCollection.newBuilder();
LinkedList&lt;JVMMetric&gt;&nbsp;buffer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedList&lt;JVMMetric&gt;();
<span class="hljs-comment">//&nbsp;将&nbsp;queue队列中缓存的全部监控数据填充到&nbsp;buffer中</span>
queue.drainTo(buffer);
<span class="hljs-comment">//&nbsp;创建&nbsp;gRPC请求参数</span>
builder.addAllMetrics(buffer);
builder.setServiceInstanceId(Agent.SERVICE_INSTANCE_ID);&nbsp;<span class="hljs-comment">//&nbsp;</span>
<span class="hljs-comment">//&nbsp;通过&nbsp;gRPC调用将JVM监控数据发送到后端&nbsp;OAP集群</span>
stub.collect(builder.build());
</code></pre>
<p>简单看一下该 gRPC 接口以及参数的定义，非常简单：</p>
<pre><code data-language="java" class="lang-java">service&nbsp;JVMMetricReportService&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">rpc&nbsp;<span class="hljs-title">collect</span>&nbsp;<span class="hljs-params">(JVMMetricCollection)</span>&nbsp;<span class="hljs-title">returns</span>&nbsp;<span class="hljs-params">(Commands)</span>&nbsp;</span>{}
}
</code></pre>
<p>在 JVMMetricCollection 中封装了 ServiceInstanceId 以及 JVMMetric 集合，JVMMetric 封装了一个时刻抓取到的 CPU、堆内存使用情况以及 GC 等信息，这里不再展开。</p>
<p>了解了发送过程之后，我们再来看收集 JVM 监控的原理。在 collectMetric 线程中会定期执行 JVMService.run() 方法，其中通过 MXBean 抓取 JVM 监控信息，然后填充到 JVMMetric 对象中并记录到 queue 缓冲队列中，大致实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;通过JMX获取CPU、Memory、GC的信息，然后组装成JVMMetric</span>
JVMMetric.Builder&nbsp;jvmBuilder&nbsp;=&nbsp;JVMMetric.newBuilder();
jvmBuilder.setTime(currentTimeMillis);
<span class="hljs-comment">//&nbsp;通过&nbsp;MXBean获取&nbsp;CPU、内存以及GC相关的信息，并填充到&nbsp;JVMMetric</span>
jvmBuilder.setCpu(CPUProvider.INSTANCE.getCpuMetric());
jvmBuilder.addAllMemory(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MemoryProvider.INSTANCE.getMemoryMetricList());
jvmBuilder.addAllMemoryPool(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MemoryPoolProvider.INSTANCE.getMemoryPoolMetricsList());
jvmBuilder.addAllGc(GCProvider.INSTANCE.getGCList());
JVMMetric&nbsp;jvmMetric&nbsp;=&nbsp;jvmBuilder.build();
<span class="hljs-comment">//&nbsp;将JVMMetric写入到queue缓冲队列中</span>
<span class="hljs-keyword">if</span>&nbsp;(!queue.offer(jvmMetric))&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;queue缓冲队列的长度默认为600</span>
&nbsp;&nbsp;&nbsp;&nbsp;queue.poll();&nbsp;<span class="hljs-comment">//&nbsp;如果queue队列被填满，则抛弃最老的监控信息，保留最新的</span>
&nbsp;&nbsp;&nbsp;&nbsp;queue.offer(jvmMetric);
}
</code></pre>
<p>这里深入介绍一下 JDK 提供的 MXBean 是如何使用的。我们以 GCProvider 为例进行分析（获取其他监控指标的方式也是类似的）。GCProvider 通过枚举方式实现了单例，在其构造方法中，会先获取 MXBean 并封装成相应的 GCMetricAccessor 实现，大致实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;获取GC相关的MXBean</span>
beans&nbsp;=&nbsp;ManagementFactory.getGarbageCollectorMXBeans();
<span class="hljs-keyword">for</span>&nbsp;(GarbageCollectorMXBean&nbsp;bean&nbsp;:&nbsp;beans)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;name&nbsp;=&nbsp;bean.getName();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;解析MXBean的名称即可得知当前使用的是哪种垃圾收集器，我们就可以创建相应</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;的GCMetricAccessor实现</span>
&nbsp;&nbsp;&nbsp;&nbsp;GCMetricAccessor&nbsp;accessor&nbsp;=&nbsp;findByBeanName(name);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(accessor&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;metricAccessor&nbsp;=&nbsp;accessor;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>GCMetricAccessor&nbsp;接口的实现类如下图所示，针对每一种垃圾收集器都有相应的实现：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0C/C0/Ciqah16OzXuAH1obAACj_oQjWi4810.png" alt=""></p>
<p>GCMetricAccessor 接口中提供了一个&nbsp;getGCList() 方法用于读取&nbsp;MXBean 获取&nbsp;GC 的信息，在抽象类 GCModule 中实现了 getGCList() 方法的核心逻辑：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;List&lt;GC&gt;&nbsp;<span class="hljs-title">getGCList</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;GC&gt;&nbsp;gcList&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedList&lt;GC&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(GarbageCollectorMXBean&nbsp;bean&nbsp;:&nbsp;beans)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;name&nbsp;=&nbsp;bean.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GCPhrase&nbsp;phrase;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;gcCount&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;gcTime&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;下面根据&nbsp;MXBean的名称判断具体的GC信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(name.equals(getNewGCName()))&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;Young&nbsp;GC的信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;phrase&nbsp;=&nbsp;GCPhrase.NEW;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;计算GC次数，从MXBean直接拿到的是GC总次数</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;collectionCount&nbsp;=&nbsp;bean.getCollectionCount();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gcCount&nbsp;=&nbsp;collectionCount&nbsp;-&nbsp;lastYGCCount;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lastYGCCount&nbsp;=&nbsp;collectionCount;&nbsp;<span class="hljs-comment">//&nbsp;更新&nbsp;lastYGCCount</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;计算GC时间，从&nbsp;MXBean直接拿到的是GC总时间</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;time&nbsp;=&nbsp;bean.getCollectionTime();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gcTime&nbsp;=&nbsp;time&nbsp;-&nbsp;lastYGCCollectionTime;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lastYGCCollectionTime&nbsp;=&nbsp;time;&nbsp;<span class="hljs-comment">//&nbsp;更新lastYGCCollectionTime</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;<span class="hljs-keyword">if</span>&nbsp;(name.equals(getOldGCName()))&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;Old&nbsp;GC的信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;phrase&nbsp;=&nbsp;GCPhrase.OLD;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;Old&nbsp;GC的计算方式与Young&nbsp;GC的计算方式相同，不再重复</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;gcList.add(&nbsp;<span class="hljs-comment">//&nbsp;最后将&nbsp;GC信息封装成List返回</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GC.newBuilder().setPhrase(phrase)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setCount(gcCount).setTime(gcTime).build()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;gcList;
}
</code></pre>
<p>在不同类型的垃圾收集器中，Young GC、Old GC 的名称不太相同（例如，G1 中叫 G1 Young Generation 和 G1 Old Generation，在 CMS 中则叫 ParNew 和 ConcurrentMarkSweep），所以这里调用的 getNewGCName() 方法和 getOldGCName() 方法都是抽象方法，延迟到了 GCModule 子类中才真正实现，这是典型的模板方法模式。</p>
<h3>其他</h3>
<p>BootService 接口的剩余四个实现与 Trace 数据的收集和发送相关，在后面的课时中会详细展开分析，这里简单说一下它们的核心功能。</p>
<ul>
<li>ContextManager：负责管理一个 SkyWalking Agent 中所有的 Context 对象。</li>
<li>ContextManagerExtendService：负责创建 Context 对象。</li>
<li>TraceSegmentServiceClient：负责将 Trace 数据序列化并发送到 OAP 集群。</li>
<li>SamplingService：负责实现 Trace 的采样。</li>
</ul>
<h3>总结</h3>
<p>本课时深入介绍了 BootService 的三个核心实现。 GRPCChannelManager 负责管理 Agent 到 OAP 集群的网络连接，并实时的通知注册的 Listener。ServiceAndEndpointRegisterClient 的核心功能有三项。</p>
<ol>
<li>注册功能：其中包括服务注册和服务实例注册两次请求。</li>
<li>定期发送心跳请求：与后端 OAP 集群维持定期探活，让后端 OAP 集群知道该 Agent 正常在线。</li>
<li>定期同步 Endpoint 名称以及网络地址：维护当前 Agent 中字符串与数字编号的映射关系，减少后续 Trace 数据传输的网络压力，提高请求的有效负载。</li>
</ol>
<p>JVMService 负责定期请求 MXBean 获取 JVM 监控信息并发送到后端的 OAP 集群。</p>

---

### 精选评论


