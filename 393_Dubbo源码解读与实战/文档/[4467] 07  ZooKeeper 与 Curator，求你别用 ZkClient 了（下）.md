<p data-nodeid="7863" class="">在上一课时我们介绍了 ZooKeeper 的核心概念以及工作原理，这里我们再简单了解一下 ZooKeeper 客户端的相关内容，毕竟在实际工作中，直接使用客户端与 ZooKeeper 进行交互的次数比深入 ZooKeeper 底层进行扩展和二次开发的次数要多得多。从 ZooKeeper 架构的角度看，使用 Dubbo 的业务节点也只是一个 ZooKeeper 客户端罢了。</p>
<p data-nodeid="7864">ZooKeeper 官方提供的客户端支持了一些基本操作，例如，创建会话、创建节点、读取节点、更新数据、删除节点和检查节点是否存在等，但在实际开发中只有这些简单功能是根本不够的。而且，ZooKeeper 本身的一些 API 也存在不足，例如：</p>
<ul data-nodeid="7865">
<li data-nodeid="7866">
<p data-nodeid="7867">ZooKeeper 的 Watcher 是一次性的，每次触发之后都需要重新进行注册。</p>
</li>
<li data-nodeid="7868">
<p data-nodeid="7869">会话超时之后，没有实现自动重连的机制。</p>
</li>
<li data-nodeid="7870">
<p data-nodeid="7871">ZooKeeper 提供了非常详细的异常，异常处理显得非常烦琐，对开发新手来说，非常不友好。</p>
</li>
<li data-nodeid="7872">
<p data-nodeid="7873">只提供了简单的 byte[] 数组的接口，没有提供基本类型以及对象级别的序列化。</p>
</li>
<li data-nodeid="7874">
<p data-nodeid="7875">创建节点时，如果节点存在抛出异常，需要自行检查节点是否存在。</p>
</li>
<li data-nodeid="7876">
<p data-nodeid="7877">删除节点就无法实现级联删除。</p>
</li>
</ul>
<p data-nodeid="7878"><strong data-nodeid="7995">常见的第三方开源 ZooKeeper 客户端有 ZkClient 和 Apache Curator</strong>。</p>
<p data-nodeid="7879">ZkClient 是在 ZooKeeper 原生 API 接口的基础上进行了包装，虽然 ZkClient 解决了 ZooKeeper 原生 API 接口的很多问题，提供了非常简洁的 API 接口，实现了会话超时自动重连的机制，解决了 Watcher 反复注册等问题，但其缺陷也非常明显。例如，文档不全、重试机制难用、异常全部转换成了 RuntimeException、没有足够的参考示例等。可见，一个简单易用、高效可靠的 ZooKeeper 客户端是多么重要。</p>
<h3 data-nodeid="7880">Apache Curator 基础</h3>
<p data-nodeid="7881"><strong data-nodeid="8001">Apache Curator 是 Apache 基金会提供的一款 ZooKeeper 客户端，它提供了一套易用性和可读性非常强的 Fluent 风格的客户端 API ，可以帮助我们快速搭建稳定可靠的 ZooKeeper 客户端程序。</strong></p>
<p data-nodeid="7882">为便于你更全面了解 Curator 的功能，我整理出了如下表格，展示了 Curator 提供的 jar 包：</p>
<p data-nodeid="7883"><img src="https://s0.lgstatic.com/i/image/M00/43/54/Ciqc1F87iUKAAAs2AAE2Xaps_KE511.png" alt="1.png" data-nodeid="8005"></p>
<p data-nodeid="7884">下面我们从最基础的使用展开，逐一介绍 Apache Curator 在实践中常用的核心功能，开始我们的 Apache Curator 之旅。</p>
<h4 data-nodeid="7885">1. 基本操作</h4>
<p data-nodeid="7886">简单了解了 Apache Curator 各个组件的定位之后，下面我们立刻通过一个示例上手使用 Curator。首先，我们创建一个 Maven 项目，并添加 Apache Curator 的依赖：</p>
<pre class="lang-xml" data-nodeid="7887"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.curator<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>curator-recipes<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>4.0.1<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="7888">然后写一个 main 方法，其中会说明 Curator 提供的基础 API 的使用：</p>
<pre class="lang-java" data-nodeid="7889"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Zookeeper集群地址，多个节点地址可以用逗号分隔 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String zkAddress = <span class="hljs-string">"127.0.0.1:2181"</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 重试策略，如果连接不上ZooKeeper集群，会重试三次，重试间隔会递增 </span>
&nbsp; &nbsp; &nbsp; &nbsp; RetryPolicy retryPolicy =
              <span class="hljs-keyword">new</span> ExponentialBackoffRetry(<span class="hljs-number">1000</span>, <span class="hljs-number">3</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建Curator Client并启动，启动成功之后，就可以与Zookeeper进行交互了 </span>
&nbsp; &nbsp; &nbsp; &nbsp; CuratorFramework client =
            CuratorFrameworkFactory.newClient(zkAddress, retryPolicy); 
&nbsp; &nbsp; &nbsp; &nbsp; client.start(); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 下面简单说明Curator中常用的API </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// create()方法创建ZNode，可以调用额外方法来设置节点类型、添加Watcher </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 下面是创建一个名为"user"的持久节点，其中会存储一个test字符串 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String path = client.create().withMode(CreateMode.PERSISTENT) 
            .forPath(<span class="hljs-string">"/user"</span>, <span class="hljs-string">"test"</span>.getBytes()); 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(path); 
        <span class="hljs-comment">// 输出:/user </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// checkExists()方法可以检查一个节点是否存在 </span>
&nbsp; &nbsp; &nbsp; &nbsp; Stat stat = client.checkExists().forPath(<span class="hljs-string">"/user"</span>); 
        System.out.println(stat!=<span class="hljs-keyword">null</span>); 
        <span class="hljs-comment">// 输出:true，返回的Stat不为null，即表示节点存在 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// getData()方法可以获取一个节点中的数据 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">byte</span>[] data = client.getData().forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-keyword">new</span> String(data)); 
        <span class="hljs-comment">// 输出:test </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// setData()方法可以设置一个节点中的数据 </span>
&nbsp; &nbsp; &nbsp; &nbsp; stat = client.setData().forPath(<span class="hljs-string">"/user"</span>,<span class="hljs-string">"data"</span>.getBytes()); 
        data = client.getData().forPath(<span class="hljs-string">"/user"</span>); 
        System.out.println(<span class="hljs-keyword">new</span> String(data)); 
        <span class="hljs-comment">// 输出:data </span>
        <span class="hljs-comment">// 在/user节点下，创建多个临时顺序节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">3</span>; i++) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; client.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL) 
                .forPath(<span class="hljs-string">"/user/child-"</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取所有子节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;String&gt; children = client.getChildren().forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(children); 
        <span class="hljs-comment">// 输出：[child-0000000002, child-0000000001, child-0000000000] </span>
        <span class="hljs-comment">// delete()方法可以删除指定节点，deletingChildrenIfNeeded()方法 </span>
        <span class="hljs-comment">// 会级联删除子节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; client.delete().deletingChildrenIfNeeded().forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; } 
}
</code></pre>
<h4 data-nodeid="7890">2. Background</h4>
<p data-nodeid="7891">上面介绍的创建、删除、更新、读取等方法都是同步的，Curator 提供异步接口，引入了BackgroundCallback 这个回调接口以及 CuratorListener 这个监听器，用于处理 Background 调用之后服务端返回的结果信息。BackgroundCallback 接口和 CuratorListener 监听器中接收一个 CuratorEvent 的参数，里面包含事件类型、响应码、节点路径等详细信息。</p>
<p data-nodeid="7892">下面我们通过一个示例说明 BackgroundCallback 接口以及 CuratorListener 监听器的基本使用：</p>
<pre class="lang-java" data-nodeid="7893"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main2</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Zookeeper集群地址，多个节点地址可以用逗号分隔 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String zkAddress = <span class="hljs-string">"127.0.0.1:2181"</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 重试策略，如果连接不上ZooKeeper集群，会重试三次，重试间隔会递增 </span>
&nbsp; &nbsp; &nbsp; &nbsp; RetryPolicy retryPolicy = <span class="hljs-keyword">new</span> ExponentialBackoffRetry(<span class="hljs-number">1000</span>,<span class="hljs-number">3</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建Curator Client并启动，启动成功之后，就可以与Zookeeper进行交互了 </span>
&nbsp; &nbsp; &nbsp; &nbsp; CuratorFramework client = CuratorFrameworkFactory 
            .newClient(zkAddress, retryPolicy); 
&nbsp; &nbsp; &nbsp; &nbsp; client.start(); 
        <span class="hljs-comment">// 添加CuratorListener监听器，针对不同的事件进行处理 </span>
&nbsp; &nbsp; &nbsp; &nbsp; client.getCuratorListenable().addListener( 
          <span class="hljs-keyword">new</span> CuratorListener() { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">eventReceived</span><span class="hljs-params">(CuratorFramework client,
                  CuratorEvent event)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">switch</span> (event.getType()) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> CREATE: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"CREATE:"</span> +
                              event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> DELETE: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"DELETE:"</span> +
                               event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> EXISTS: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"EXISTS:"</span> +
                                event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> GET_DATA: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"GET_DATA:"</span> +
                          event.getPath() + <span class="hljs-string">","</span>
                              + <span class="hljs-keyword">new</span> String(event.getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> SET_DATA: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"SET_DATA:"</span> + 
                                 <span class="hljs-keyword">new</span> String(event.getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> CHILDREN: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"CHILDREN:"</span> +
                                event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">default</span>: 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; }); 
        <span class="hljs-comment">// 注意:下面所有的操作都添加了inBackground()方法，转换为后台操作 </span>
        client.create().withMode(CreateMode.PERSISTENT) 
            .inBackground().forPath(<span class="hljs-string">"/user"</span>, <span class="hljs-string">"test"</span>.getBytes()); 
&nbsp; &nbsp; &nbsp; &nbsp; client.checkExists().inBackground().forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; client.setData().inBackground().forPath(<span class="hljs-string">"/user"</span>,
              <span class="hljs-string">"setData-Test"</span>.getBytes()); 
&nbsp; &nbsp; &nbsp; &nbsp; client.getData().inBackground().forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">3</span>; i++) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; client.create().withMode(CreateMode.EPHEMERAL_SEQUENTIAL) 
                .inBackground().forPath(<span class="hljs-string">"/user/child-"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; client.getChildren().inBackground().forPath(<span class="hljs-string">"/user"</span>); 
        <span class="hljs-comment">// 添加BackgroundCallback </span>
        client.getChildren().inBackground(<span class="hljs-keyword">new</span> BackgroundCallback() { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processResult</span><span class="hljs-params">(CuratorFramework client,
                CuratorEvent event)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"in background:"</span>
                      + event.getType() + <span class="hljs-string">","</span> + event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; }).forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; client.delete().deletingChildrenIfNeeded().inBackground() 
              .forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; System.in.read(); 
&nbsp; &nbsp; } 
} 
<span class="hljs-comment">// 输出： </span>
<span class="hljs-comment">// CREATE:/user </span>
<span class="hljs-comment">// EXISTS:/user </span>
<span class="hljs-comment">// GET_DATA:/user,setData-Test </span>
<span class="hljs-comment">// CREATE:/user/child- </span>
<span class="hljs-comment">// CREATE:/user/child- </span>
<span class="hljs-comment">// CREATE:/user/child- </span>
<span class="hljs-comment">// CHILDREN:/user </span>
<span class="hljs-comment">// DELETE:/user</span>
</code></pre>
<h4 data-nodeid="7894">3. 连接状态监听</h4>
<p data-nodeid="7895">除了基础的数据操作，Curator 还提供了<strong data-nodeid="8025">监听连接状态的监听器——ConnectionStateListener</strong>，它主要是处理 Curator 客户端和 ZooKeeper 服务器间连接的异常情况，例如， 短暂或者长时间断开连接。</p>
<p data-nodeid="7896">短暂断开连接时，ZooKeeper 客户端会检测到与服务端的连接已经断开，但是服务端维护的客户端 Session 尚未过期，之后客户端和服务端重新建立了连接；当客户端重新连接后，由于 Session 没有过期，ZooKeeper 能够保证连接恢复后保持正常服务。</p>
<p data-nodeid="7897">而长时间断开连接时，Session 已过期，与先前 Session 相关的 Watcher 和临时节点都会丢失。当 Curator 重新创建了与 ZooKeeper 的连接时，会获取到 Session 过期的相关异常，Curator 会销毁老 Session，并且创建一个新的 Session。由于老 Session 关联的数据不存在了，在 ConnectionStateListener 监听到 LOST 事件时，就可以依靠本地存储的数据恢复 Session 了。</p>
<p data-nodeid="7898"><strong data-nodeid="8032">这里 Session 指的是 ZooKeeper 服务器与客户端的会话</strong>。客户端启动的时候会与服务器建立一个 TCP 连接，从第一次连接建立开始，客户端会话的生命周期也开始了。客户端能够通过心跳检测与服务器保持有效的会话，也能够向 ZooKeeper 服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的 Watch 事件通知。</p>
<p data-nodeid="7899">我们可以设置客户端会话的超时时间（sessionTimeout），当服务器压力太大、网络故障或是客户端主动断开连接等原因导致连接断开时，只要客户端在 sessionTimeout 规定的时间内能够重新连接到 ZooKeeper 集群中任意一个实例，那么之前创建的会话仍然有效。ZooKeeper 通过 sessionID 唯一标识 Session，所以在 ZooKeeper 集群中，sessionID 需要保证全局唯一。 由于 ZooKeeper 会将 Session 信息存放到硬盘中，即使节点重启，之前未过期的 Session 仍然会存在。</p>
<pre class="lang-java" data-nodeid="7900"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main3</span> </span>{ 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-comment">// Zookeeper集群地址，多个节点地址可以用逗号分隔 </span>
        String zkAddress = <span class="hljs-string">"127.0.0.1:2181"</span>; 
        <span class="hljs-comment">// 重试策略，如果连接不上ZooKeeper集群，会重试三次，重试间隔会递增 </span>
        RetryPolicy retryPolicy = <span class="hljs-keyword">new</span> ExponentialBackoffRetry(<span class="hljs-number">1000</span>,<span class="hljs-number">3</span>); 
        <span class="hljs-comment">// 创建Curator Client并启动，启动成功之后，就可以与Zookeeper进行交互了 </span>
        CuratorFramework client = CuratorFrameworkFactory 
            .newClient(zkAddress, retryPolicy); 
        client.start(); 
        <span class="hljs-comment">// 添加ConnectionStateListener监听器 </span>
        client.getConnectionStateListenable().addListener( 
          <span class="hljs-keyword">new</span> ConnectionStateListener() { 
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">stateChanged</span><span class="hljs-params">(CuratorFramework client,
                    ConnectionState newState)</span> </span>{ 
                <span class="hljs-comment">// 这里我们可以针对不同的连接状态进行特殊的处理 </span>
                <span class="hljs-keyword">switch</span> (newState) {
                    <span class="hljs-keyword">case</span> CONNECTED: 
                        <span class="hljs-comment">// 第一次成功连接到ZooKeeper之后会进入该状态。 </span>
                        <span class="hljs-comment">// 对于每个CuratorFramework对象，此状态仅出现一次 </span>
                        <span class="hljs-keyword">break</span>; 
                    <span class="hljs-keyword">case</span> SUSPENDED: <span class="hljs-comment">//   ZooKeeper的连接丢失 </span>
                        <span class="hljs-keyword">break</span>; 
                    <span class="hljs-keyword">case</span> RECONNECTED: <span class="hljs-comment">// 丢失的连接被重新建立 </span>
                        <span class="hljs-keyword">break</span>; 
                    <span class="hljs-keyword">case</span> LOST:
                        <span class="hljs-comment">// 当Curator认为会话已经过期时，则进入此状态 </span>
                        <span class="hljs-keyword">break</span>; 
                    <span class="hljs-keyword">case</span> READ_ONLY: <span class="hljs-comment">// 连接进入只读模式 </span>
                        <span class="hljs-keyword">break</span>; 
                } 
            } 
        }); 
   } 
}
</code></pre>
<h4 data-nodeid="7901">4. Watcher</h4>
<p data-nodeid="7902">Watcher 监听机制是 ZooKeeper 中非常重要的特性，可以监听某个节点上发生的特定事件，例如，监听节点数据变更、节点删除、子节点状态变更等事件。当相应事件发生时，ZooKeeper 会产生一个 Watcher 事件，并且发送到客户端。通过 Watcher 机制，就可以使用 ZooKeeper 实现分布式锁、集群管理等功能。</p>
<p data-nodeid="7903">在 Curator 客户端中，我们可以使用 usingWatcher() 方法添加 Watcher，前面示例中，能够添加 Watcher 的有 checkExists()、getData()以及 getChildren() 三个方法，下面我们来看一个具体的示例：</p>
<pre class="lang-java" data-nodeid="7904"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main4</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Zookeeper集群地址，多个节点地址可以用逗号分隔 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String zkAddress = <span class="hljs-string">"127.0.0.1:2181"</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 重试策略，如果连接不上ZooKeeper集群，会重试三次，重试间隔会递增 </span>
&nbsp; &nbsp; &nbsp; &nbsp; RetryPolicy retryPolicy = <span class="hljs-keyword">new</span> ExponentialBackoffRetry(<span class="hljs-number">1000</span>,<span class="hljs-number">3</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建Curator Client并启动，启动成功之后，就可以与Zookeeper进行交互了 </span>
&nbsp; &nbsp; &nbsp; &nbsp; CuratorFramework client = CuratorFrameworkFactory 
              .newClient(zkAddress, retryPolicy); 
&nbsp; &nbsp; &nbsp; &nbsp; client.start(); 
       &nbsp;<span class="hljs-keyword">try</span> { 
           client.create().withMode(CreateMode.PERSISTENT) 
                 .forPath(<span class="hljs-string">"/user"</span>, <span class="hljs-string">"test"</span>.getBytes()); 
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) { 
&nbsp; &nbsp; &nbsp; &nbsp; } 
        <span class="hljs-comment">// 这里通过usingWatcher()方法添加一个Watcher </span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;String&gt; children = client.getChildren().usingWatcher( 
          <span class="hljs-keyword">new</span> CuratorWatcher() { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">process</span><span class="hljs-params">(WatchedEvent event)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(event.getType() + <span class="hljs-string">","</span> +
                    event.getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; }).forPath(<span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(children); 
&nbsp; &nbsp; &nbsp; &nbsp; System.in.read(); 
&nbsp; &nbsp; } 
}
</code></pre>
<p data-nodeid="7905">接下来，我们打开 ZooKeeper 的命令行客户端，在 /user 节点下先后添加两个子节点，如下所示：</p>
<p data-nodeid="7906"><img src="https://s0.lgstatic.com/i/image/M00/43/54/Ciqc1F87iXuAQVanAABhI9RRD8M252.png" alt="Drawing 0.png" data-nodeid="8042"></p>
<p data-nodeid="7907">此时我们只得到一行输出：</p>
<pre class="lang-java" data-nodeid="7908"><code data-language="java">NodeChildrenChanged,/user
</code></pre>
<p data-nodeid="7909">之所以这样，是因为通过 usingWatcher() 方法添加的 CuratorWatcher 只会触发一次，触发完毕后就会销毁。checkExists() 方法、getData() 方法通过 usingWatcher() 方法添加的 Watcher 也是一样的原理，只不过监听的事件不同，你若感兴趣的话，可以自行尝试一下。</p>
<p data-nodeid="7910">相信你已经感受到，直接通过注册 Watcher 进行事件监听不是特别方便，需要我们自己反复注册 Watcher。<strong data-nodeid="8050">Apache Curator 引入了 Cache 来实现对 ZooKeeper 服务端事件的监听</strong>。Cache 是 Curator 中对事件监听的包装，其对事件的监听其实可以近似看作是一个本地缓存视图和远程ZooKeeper 视图的对比过程。同时，Curator 能够自动为开发人员处理反复注册监听，从而大大简化了代码的复杂程度。</p>
<p data-nodeid="7911">实践中常用的 Cache 有三大类：</p>
<ul data-nodeid="7912">
<li data-nodeid="7913">
<p data-nodeid="7914"><strong data-nodeid="8056">NodeCache。</strong> 对一个节点进行监听，监听事件包括指定节点的增删改操作。注意哦，NodeCache 不仅可以监听数据节点的内容变更，也能监听指定节点是否存在，如果原本节点不存在，那么 Cache 就会在节点被创建后触发 NodeCacheListener，删除操作亦然。</p>
</li>
<li data-nodeid="7915">
<p data-nodeid="7916"><strong data-nodeid="8061">PathChildrenCache。</strong> 对指定节点的一级子节点进行监听，监听事件包括子节点的增删改操作，但是不对该节点的操作监听。</p>
</li>
<li data-nodeid="7917">
<p data-nodeid="7918"><strong data-nodeid="8066">TreeCache。</strong> 综合 NodeCache 和 PathChildrenCache 的功能，是对指定节点以及其子节点进行监听，同时还可以设置监听的深度。</p>
</li>
</ul>
<p data-nodeid="7919">下面通过示例介绍上述三种 Cache 的基本使用：</p>
<pre class="lang-java" data-nodeid="7920"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main5</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Zookeeper集群地址，多个节点地址可以用逗号分隔 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String zkAddress = <span class="hljs-string">"127.0.0.1:2181"</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 重试策略，如果连接不上ZooKeeper集群，会重试三次，重试间隔会递增 </span>
&nbsp; &nbsp; &nbsp; &nbsp; RetryPolicy retryPolicy = <span class="hljs-keyword">new</span> ExponentialBackoffRetry(<span class="hljs-number">1000</span>,<span class="hljs-number">3</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建Curator Client并启动，启动成功之后，就可以与Zookeeper进行交互了 </span>
&nbsp; &nbsp; &nbsp; &nbsp; CuratorFramework client = CuratorFrameworkFactory 
           .newClient(zkAddress, retryPolicy); 
&nbsp; &nbsp; &nbsp; &nbsp; client.start(); 

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建NodeCache，监听的是"/user"这个节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; NodeCache nodeCache = <span class="hljs-keyword">new</span> NodeCache(client, <span class="hljs-string">"/user"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// start()方法有个boolean类型的参数，默认是false。如果设置为true， </span>
        <span class="hljs-comment">// 那么NodeCache在第一次启动的时候就会立刻从ZooKeeper上读取对应节点的 </span>
        <span class="hljs-comment">// 数据内容，并保存在Cache中。 </span>
&nbsp; &nbsp; &nbsp; &nbsp; nodeCache.start(<span class="hljs-keyword">true</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (nodeCache.getCurrentData() != <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"NodeCache节点初始化数据为："</span>
                + <span class="hljs-keyword">new</span> String(nodeCache.getCurrentData().getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"NodeCache节点数据为空"</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; } 

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 添加监听器 </span>
&nbsp; &nbsp; &nbsp; &nbsp; nodeCache.getListenable().addListener(() -&gt; { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String data = <span class="hljs-keyword">new</span> String(nodeCache.getCurrentData().getData()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"NodeCache节点路径："</span> + nodeCache.getCurrentData().getPath() 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + <span class="hljs-string">"，节点数据为："</span> + data); 
&nbsp; &nbsp; &nbsp; &nbsp; }); 

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建PathChildrenCache实例，监听的是"user"这个节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; PathChildrenCache childrenCache = <span class="hljs-keyword">new</span> PathChildrenCache(client, <span class="hljs-string">"/user"</span>, <span class="hljs-keyword">true</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// StartMode指定的初始化的模式 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// NORMAL:普通异步初始化 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// BUILD_INITIAL_CACHE:同步初始化 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// POST_INITIALIZED_EVENT:异步初始化，初始化之后会触发事件 </span>
&nbsp; &nbsp; &nbsp; &nbsp; childrenCache.start(PathChildrenCache.StartMode.BUILD_INITIAL_CACHE); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// childrenCache.start(PathChildrenCache.StartMode.POST_INITIALIZED_EVENT); </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// childrenCache.start(PathChildrenCache.StartMode.NORMAL); </span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ChildData&gt; children = childrenCache.getCurrentData(); 
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"获取子节点列表："</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果是BUILD_INITIAL_CACHE可以获取这个数据，如果不是就不行 </span>
&nbsp; &nbsp; &nbsp; &nbsp; children.forEach(childData -&gt; { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-keyword">new</span> String(childData.getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; }); 
&nbsp; &nbsp; &nbsp; &nbsp; childrenCache.getListenable().addListener(((client1, event) -&gt; { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(LocalDateTime.now() + <span class="hljs-string">"&nbsp; "</span> + event.getType()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (event.getType().equals(PathChildrenCacheEvent.Type.INITIALIZED)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache:子节点初始化成功..."</span>); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event.getType().equals(PathChildrenCacheEvent.Type.CHILD_ADDED)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String path = event.getData().getPath(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache添加子节点:"</span> + event.getData().getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache子节点数据:"</span> + <span class="hljs-keyword">new</span> String(event.getData().getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event.getType().equals(PathChildrenCacheEvent.Type.CHILD_REMOVED)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache删除子节点:"</span> + event.getData().getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (event.getType().equals(PathChildrenCacheEvent.Type.CHILD_UPDATED)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache修改子节点路径:"</span> + event.getData().getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"PathChildrenCache修改子节点数据:"</span> + <span class="hljs-keyword">new</span> String(event.getData().getData())); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; })); 

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建TreeCache实例监听"user"节点 </span>
&nbsp; &nbsp; &nbsp; &nbsp; TreeCache cache = TreeCache.newBuilder(client, <span class="hljs-string">"/user"</span>).setCacheData(<span class="hljs-keyword">false</span>).build(); 
&nbsp; &nbsp; &nbsp; &nbsp; cache.getListenable().addListener((c, event) -&gt; { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (event.getData() != <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"TreeCache,type="</span> + event.getType() + <span class="hljs-string">" path="</span> + event.getData().getPath()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"TreeCache,type="</span> + event.getType()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; }); 
&nbsp; &nbsp; &nbsp; &nbsp; cache.start(); 

&nbsp; &nbsp; &nbsp; &nbsp; System.in.read(); 
&nbsp; &nbsp; } 
}
</code></pre>
<p data-nodeid="7921">此时，ZooKeeper 集群中存在 /user/test1 和 /user/test2 两个节点，启动上述测试代码，得到的输出如下：</p>
<pre class="lang-java" data-nodeid="7922"><code data-language="java">NodeCache节点初始化数据为：test <span class="hljs-comment">//NodeCache的相关输出</span>
获取子节点列表：<span class="hljs-comment">// PathChildrenCache的相关输出 </span>
xxx 
xxx2 
<span class="hljs-comment">// TreeCache监听到的事件 </span>
TreeCache,type=NODE_ADDED path=/user 
TreeCache,type=NODE_ADDED path=/user/test1 
TreeCache,type=NODE_ADDED path=/user/test2 
TreeCache,type=INITIALIZED
</code></pre>
<p data-nodeid="7923">接下来，我们在 ZooKeeper 命令行客户端中<strong data-nodeid="8074">更新 /user 节点中的数据</strong>：</p>
<p data-nodeid="7924"><img src="https://s0.lgstatic.com/i/image/M00/43/54/Ciqc1F87iY6ACWnvAAA8jA9QVgM875.png" alt="Drawing 1.png" data-nodeid="8077"></p>
<p data-nodeid="7925">得到如下输出：</p>
<pre class="lang-java" data-nodeid="7926"><code data-language="java">TreeCache,type=NODE_UPDATED path=/user 
NodeCache节点路径：/user，节点数据为：userData
</code></pre>
<p data-nodeid="7927"><strong data-nodeid="8083">创建 /user/test3 节点</strong>：</p>
<p data-nodeid="7928"><img src="https://s0.lgstatic.com/i/image/M00/43/60/CgqCHl87iZqAaG93AABwFnQJA7o497.png" alt="Drawing 2.png" data-nodeid="8086"></p>
<p data-nodeid="7929">得到输出：</p>
<pre class="lang-java" data-nodeid="7930"><code data-language="java">TreeCache,type=NODE_ADDED path=/user/test3 
<span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">26</span>T08:<span class="hljs-number">35</span>:<span class="hljs-number">22.393</span>&nbsp; CHILD_ADDED 
PathChildrenCache添加子节点:/user/test3 
PathChildrenCache子节点数据:xxx3
</code></pre>
<p data-nodeid="7931"><strong data-nodeid="8092">更新 /user/test3 节点的数据</strong>：</p>
<p data-nodeid="7932"><img src="https://s0.lgstatic.com/i/image/M00/43/54/Ciqc1F87iaSAFZLpAABDyAm7vuE120.png" alt="Drawing 3.png" data-nodeid="8095"></p>
<p data-nodeid="7933">得到输出：</p>
<pre class="lang-java" data-nodeid="7934"><code data-language="java">TreeCache,type=NODE_UPDATED path=/user/test3 
<span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">26</span>T08:<span class="hljs-number">43</span>:<span class="hljs-number">54.604</span>&nbsp; CHILD_UPDATED 
PathChildrenCache修改子节点路径:/user/test3 
PathChildrenCache修改子节点数据:xxx33
</code></pre>
<p data-nodeid="7935"><strong data-nodeid="8101">删除 /user/test3 节点</strong>：</p>
<p data-nodeid="7936"><img src="https://s0.lgstatic.com/i/image/M00/43/60/CgqCHl87ia6AYvijAABBmFLfzx4213.png" alt="Drawing 4.png" data-nodeid="8104"></p>
<p data-nodeid="7937">得到输出：</p>
<pre class="lang-java" data-nodeid="7938"><code data-language="java">TreeCache,type=NODE_REMOVED path=/user/test3 
<span class="hljs-number">2020</span>-<span class="hljs-number">06</span>-<span class="hljs-number">26</span>T08:<span class="hljs-number">44</span>:<span class="hljs-number">06.329</span>&nbsp; CHILD_REMOVED 
PathChildrenCache删除子节点:/user/test3
</code></pre>
<h3 data-nodeid="7939">curator-x-discovery 扩展库</h3>
<p data-nodeid="7940">为了避免&nbsp;curator-framework 包过于膨胀，Curator 将很多其他解决方案都拆出来了，作为单独的一个包，例如：curator-recipes、curator-x-discovery、curator-x-rpc 等。</p>
<p data-nodeid="7941">在后面我们会使用到 curator-x-discovery 来完成一个简易 RPC 框架的注册中心模块。<strong data-nodeid="8113">curator-x-discovery 扩展包是一个服务发现的解决方案</strong>。在 ZooKeeper 中，我们可以使用临时节点实现一个服务注册机制。当服务启动后在 ZooKeeper 的指定 Path 下创建临时节点，服务断掉与 ZooKeeper 的会话之后，其相应的临时节点就会被删除。这个 curator-x-discovery 扩展包抽象了这种功能，并提供了一套简单的 API 来实现服务发现机制。curator-x-discovery 扩展包的核心概念如下：</p>
<ul data-nodeid="7942">
<li data-nodeid="7943">
<p data-nodeid="7944"><strong data-nodeid="8118">ServiceInstance。</strong> 这是 curator-x-discovery 扩展包对服务实例的抽象，由 name、id、address、port 以及一个可选的 payload 属性构成。其存储在 ZooKeeper 中的方式如下图展示的这样。</p>
</li>
</ul>
<p data-nodeid="7945"><img src="https://s0.lgstatic.com/i/image/M00/43/60/CgqCHl87icOABt59AADHccHcE1Q955.png" alt="Drawing 5.png" data-nodeid="8121"></p>
<ul data-nodeid="7946">
<li data-nodeid="7947">
<p data-nodeid="7948"><strong data-nodeid="8126">ServiceProvider。</strong> 这是 curator-x-discovery 扩展包的核心组件之一，提供了多种不同策略的服务发现方式，具体策略有轮询调度、随机和黏性（总是选择相同的一个）。得到 ServiceProvider 对象之后，我们可以调用其 getInstance() 方法，按照指定策略获取 ServiceInstance 对象（即发现可用服务实例）；还可以调用 getAllInstances() 方法，获取所有 ServiceInstance 对象（即获取全部可用服务实例）。</p>
</li>
<li data-nodeid="7949">
<p data-nodeid="7950"><strong data-nodeid="8131">ServiceDiscovery。</strong> 这是 curator-x-discovery 扩展包的入口类。开始必须调用 start() 方法，当使用完成应该调用 close() 方法进行销毁。</p>
</li>
<li data-nodeid="7951">
<p data-nodeid="7952"><strong data-nodeid="8136">ServiceCache。</strong> 如果程序中会频繁地查询 ServiceInstance 对象，我们可以添加 ServiceCache 缓存，ServiceCache 会在内存中缓存 ServiceInstance 实例的列表，并且添加相应的 Watcher 来同步更新缓存。查询 ServiceCache 的方式也是 getInstances() 方法。另外，ServiceCache 上还可以添加 Listener 来监听缓存变化。</p>
</li>
</ul>
<p data-nodeid="7953">下面通过一个简单示例来说明一下 curator-x-discovery 包的使用，该示例中的 ServerInfo 记录了一个服务的 host、port 以及描述信息。</p>
<pre class="lang-java" data-nodeid="7954"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ZookeeperCoordinator</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> ServiceDiscovery&lt;ServerInfo&gt; serviceDiscovery; 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> ServiceCache&lt;ServerInfo&gt; serviceCache; 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> CuratorFramework client; 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String root; 
    <span class="hljs-comment">// 这里的JsonInstanceSerializer是将ServerInfo序列化成Json </span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> InstanceSerializer serializer =
        <span class="hljs-keyword">new</span> JsonInstanceSerializer&lt;&gt;(ServerInfo.class); 
&nbsp; &nbsp; ZookeeperCoordinator(Config config) <span class="hljs-keyword">throws</span> Exception { 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.root = config.getPath(); 
&nbsp; &nbsp; &nbsp; &nbsp; // 创建Curator客户端 
&nbsp; &nbsp; &nbsp; &nbsp; client = CuratorFrameworkFactory.newClient( 
            config.getHostPort(),  <span class="hljs-keyword">new</span> ExponentialBackoffRetry(...)); 
        client.start(); <span class="hljs-comment">// 启动Curator客户端</span>
        client.blockUntilConnected();&nbsp; <span class="hljs-comment">// 阻塞当前线程，等待连接成功 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建ServiceDiscovery </span>
&nbsp; &nbsp; &nbsp; &nbsp; serviceDiscovery = ServiceDiscoveryBuilder 
                .builder(ServerInfo.class) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .client(client) // 依赖Curator客户端 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .basePath(root) // 管理的Zk路径 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .watchInstances(<span class="hljs-keyword">true</span>) // 当ServiceInstance加载 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .serializer(serializer) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;serviceDiscovery.start(); // 启动ServiceDiscovery 
&nbsp; &nbsp; &nbsp; &nbsp; // 创建ServiceCache，监Zookeeper相应节点的变化，也方便后续的读取 
&nbsp; &nbsp; &nbsp; &nbsp; serviceCache = serviceDiscovery.serviceCacheBuilder() 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .name(root) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build();
&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; serviceCache.start(); // 启动ServiceCache 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerRemote</span><span class="hljs-params">(ServerInfo serverInfo)</span><span class="hljs-keyword">throws</span> Exception</span>{ 
&nbsp; &nbsp; &nbsp;&nbsp; &nbsp; <span class="hljs-comment">// 将ServerInfo对象转换成ServiceInstance对象 </span>
&nbsp;  &nbsp; &nbsp; &nbsp; ServiceInstance&lt;ServerInfo&gt; thisInstance =
            ServiceInstance.&lt;ServerInfo&gt;builder() 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .name(root) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .id(UUID.randomUUID().toString())&nbsp;<span class="hljs-comment">// 随机生成的UUID </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .address(serverInfo.getHost()) <span class="hljs-comment">// host </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .port(serverInfo.getPort()) <span class="hljs-comment">// port </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .payload(serverInfo) <span class="hljs-comment">// payload </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build(); 
&nbsp; &nbsp;&nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将ServiceInstance写入到Zookeeper中 </span>
&nbsp; &nbsp;&nbsp; &nbsp; &nbsp; serviceDiscovery.registerService(thisInstance); 
&nbsp; &nbsp; } 

&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;ServerInfo&gt; <span class="hljs-title">queryRemoteNodes</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ServerInfo&gt; ServerInfoDetails = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 查询 ServiceCache 获取全部的 ServiceInstance 对象 </span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ServiceInstance&lt;ServerInfo&gt;&gt; serviceInstances =
            serviceCache.getInstances(); 
&nbsp; &nbsp; &nbsp; &nbsp; serviceInstances.forEach(serviceInstance -&gt; { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 从每个ServiceInstance对象的playload字段中反序列化得 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 到ServerInfo实例 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ServerInfo instance = serviceInstance.getPayload(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ServerInfoDetails.add(instance); 
&nbsp; &nbsp; &nbsp; &nbsp; }); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ServerInfoDetails; 
&nbsp; &nbsp; } 
}
</code></pre>
<h3 data-nodeid="7955">curator-recipes 简介</h3>
<p data-nodeid="7956">Recipes 是 Curator 对常见分布式场景的解决方案，这里我们只是简单介绍一下，具体的使用和原理，就先不做深入分析了。</p>
<ul data-nodeid="7957">
<li data-nodeid="7958">
<p data-nodeid="7959"><strong data-nodeid="8144">Queues</strong>。提供了多种的分布式队列解决方法，比如：权重队列、延迟队列等。在生产环境中，很少将 ZooKeeper 用作分布式队列，只适合在压力非常小的情况下，才使用该解决方案，所以建议你要适度使用。</p>
</li>
<li data-nodeid="7960">
<p data-nodeid="7961"><strong data-nodeid="8149">Counters</strong>。全局计数器是分布式系统中很常用的工具，curator-recipes 提供了 SharedCount、DistributedAtomicLong 等组件，帮助开发人员实现分布式计数器功能。</p>
</li>
<li data-nodeid="7962">
<p data-nodeid="7963"><strong data-nodeid="8154">Locks</strong>。java.util.concurrent.locks 中提供的各种锁相信你已经有所了解了，在微服务架构中，分布式锁也是一项非常基础的服务组件，curator-recipes 提供了多种基于 ZooKeeper 实现的分布式锁，满足日常工作中对分布式锁的需求。</p>
</li>
<li data-nodeid="7964">
<p data-nodeid="7965"><strong data-nodeid="8159">Barries</strong>。curator-recipes 提供的分布式栅栏可以实现多个服务之间协同工作，具体实现有 DistributedBarrier 和 DistributedDoubleBarrier。</p>
</li>
<li data-nodeid="7966">
<p data-nodeid="7967"><strong data-nodeid="8164">Elections</strong>。实现的主要功能是在多个参与者中选举出 Leader，然后由 Leader 节点作为操作调度、任务监控或是队列消费的执行者。curator-recipes 给出的实现是 LeaderLatch。</p>
</li>
</ul>
<h3 data-nodeid="7968">总结</h3>
<p data-nodeid="7969">本课时我们重点介绍了 Apache Curator 相关的内容：</p>
<ul data-nodeid="7970">
<li data-nodeid="7971">
<p data-nodeid="7972">首先将 Apache Curator 与其他 ZooKeeper 客户端进行了对比，Apache Curator 的易用性是选择 Apache Curator 的重要原因。</p>
</li>
<li data-nodeid="7973">
<p data-nodeid="7974">接下来，我们通过示例介绍了 Apache Curator 的基本使用方式以及实际使用过程中的一些注意点。</p>
</li>
<li data-nodeid="7975">
<p data-nodeid="7976">然后，介绍了 curator-x-discovery 扩展库的基本概念和使用。</p>
</li>
<li data-nodeid="7977">
<p data-nodeid="7978">最后，简单介绍了 curator-recipes 提供的强大功能。</p>
</li>
</ul>
<p data-nodeid="8794">关于 Apache Curator，你有什么其他的见解？欢迎你在评论区给我留言，与我分享。</p>
<p data-nodeid="12594" class="te-preview-highlight"><span style="color:#ab4642">zk-demo 链接：<a href="https://github.com/xxxlxy2008/zk-demo" data-nodeid="12599">https://github.com/xxxlxy2008/zk-demo</a> 。</span></p>

---

### 精选评论

##### *志：
> 你好老师，想问一下这套视频例子的的源码在哪里，每节课都有对应的例子，但是没有找到源码在哪里？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 具体地址：https://github.com/xxxlxy2008/zk-demo

##### **志：
> Exception in thread "main" org.apache.zookeeper.KeeperException$ConnectionLossException: KeeperErrorCode = ConnectionLoss for /user	at org.apache.zookeeper.KeeperException.create(KeeperException.java:102)	at org.apache.zookeeper.KeeperException.create(KeeperException.java:54)	at org.apache.zookeeper.ZooKeeper.create(ZooKeeper.java:792)	at org.apache.curator.framework.imps.CreateBuilderImpl$17.call(CreateBuilderImpl.java:1177)	at org.apache.curator.framework.imps.CreateBuilderImpl$17.call(CreateBuilderImpl.java:1158)	at org.apache.curator.connection.StandardConnectionHandlingPolicy.callWithRetry(StandardConnectionHandlingPolicy.java:64)	at org.apache.curator.RetryLoop.callWithRetry(RetryLoop.java:100)	at org.apache.curator.framework.imps.CreateBuilderImpl.pathInForeground(CreateBuilderImpl.java:1155)	at org.apache.curator.framework.imps.CreateBuilderImpl.protectedPathInForeground(CreateBuilderImpl.java:605)	at org.apache.curator.framework.imps.CreateBuilderImpl.forPath(CreateBuilderImpl.java:595)	at org.apache.curator.framework.imps.CreateBuilderImpl.forPath(CreateBuilderImpl.java:49)	at com.xxx.curator.CuratorMain01.main(CuratorMain01.java:19)老师，第一个例子，我报这个错，是因为啥啊，我百度了一下，也没有解决

##### **升：
> 老师您好，能说明下这个Curator版本对应的zookeeper是什么版本的嘛？？？我这一直报NoSuchMethod

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Zookeeper 3.6.1 版本，curator 用的是 4.0.1 版本

