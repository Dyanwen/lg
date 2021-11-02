<p>本课时将紧接上一课时的内容，继续介绍服务实例注册请求、EndpointName 以及 NetworkAddress 同步请求的处理。</p>
<h3>IRegisterLockDAO原理分析</h3>
<p>紧接上一课时，当一个服务的注册请求首次到达 RegisterPersistentWorker 时，会通过 IRegisterLockDAO 为其生成全局唯一 ID 。IRegisterLockDAO 接口有两个实现类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/D1/Ciqc1F7WHyuAehevAADKLauus4I955.png" alt="image (13).png"></p>
<p>这里要重点分析的是 RegisterLockDAOImpl 这个实现类，其底层是依赖 ElasticSearch 的 version 机制实现的乐观锁。</p>
<p>RegisterLockDAOImpl 这个分布式乐观锁底层使用的索引名称是 register_lock，它为每个 RegisterSource 都分配了一个单独的 Document。Document ID 就是 @Stream 注解中的 scopeId 的值，例如，ServiceInventory 的 scopeId 就是 14，ServiceInstanceInventory 的 scopeId 就是 15，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/DD/CgqCHl7WHzWAQfHPAAYFXpVpP-Y709.png" alt="image (14).png"></p>
<p>在 register_lock 索引中只有一个 sequence 字段，它是一个 int 值，初始值为 0，RegisterLockDAOImpl 就是通过自增该字段来分配唯一 ID。</p>
<p>在 getId() 方法中，RegisterLockDAOImpl 会根据 RegisterSource 查找相应的 Document，并获取 sequence 字段值以及 Document Version，然后递增 sequence 值，写回到对应的 Document 中。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getId</span><span class="hljs-params">(<span class="hljs-keyword">int</span> scopeId, RegisterSource registerSource)</span> </span>{
&nbsp; &nbsp; String id = scopeId + <span class="hljs-string">""</span>; <span class="hljs-comment">// Document Id</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> sequence = Const.NONE;
&nbsp; &nbsp; <span class="hljs-comment">// 发送GetRequest请求，获取对应的Document</span>
&nbsp; &nbsp; GetResponse response = getClient().get(<span class="hljs-string">"register_lock"</span>, id);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (response.isExists()) {
&nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, Object&gt; source = response.getSource();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取sequence字段的值</span>
&nbsp; &nbsp; &nbsp; &nbsp; sequence = ((Number)source.get(<span class="hljs-string">"sequence"</span>)).intValue();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取ServiceInventory对应 Document的版本号</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> version = response.getVersion();
&nbsp; &nbsp; &nbsp; &nbsp; sequence++; <span class="hljs-comment">// 递增sequence，即为该 ServiceInventory分配的唯一ID</span>
&nbsp; &nbsp; &nbsp; &nbsp; lock(id, sequence, version);&nbsp;<span class="hljs-comment">// 更新sequence字段值</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> sequence; <span class="hljs-comment">// 更新成功，返回该sequence值</span>
}
</code></pre>
<p>lock() 方法写回 sequence 值时，会发送一个带 version 的 UpdateRequest 请求，ElasticSearch 集群会比较该 version 是否发生变化，如果 version 发生变化，表示有其他线程并发操作，占用了该 sequence 值，就会抛出异常，后续 Agent 会重新为该服务发起注册请求。如果 version 未发生变化，表示无并发操作，即可将该 sequence 值分配给该服务，由 RegisterPersistentWorker 持久化该映射关系，具体实现过程不再重复。</p>
<blockquote>
<p>在高版本的 ElasticSearch 中，不再推荐使用 version 的方式实现乐观锁，而是使用 _seq_no 和 _primary_term 两个字段来实现乐观锁，具体的实现方式与使用 version 的方式类似。这里简单介绍一下这两个字段：_primary_term 主要用于记录 Document 所在的主分片，每当主分片发生重新分配时，比如重启、Primary 选举等，_primary_term 会递增 1。_seq_no 字段和旧版本中的 _version 字段作用类似，是严格递增的顺序号，每个 Document 在分片级别内对应一个，且严格递增，以保证后写入的 Document 的 _seq_no 值大于先写入的 Document 的 _seq_no 值。 加上 _primary_term 这个字段可以提高并发的性能，但由于一个 Document 只会位于某一个特定的主分片中，所以由所在主分片分配序列号比之前通过 ElasticSearch 集群全局统一管理 _version 的性能会更高效。<br>
更多相关内容，可以参考：<a href="https://github.com/elastic/elasticsearch/issues/19269#issuecomment-488598561">https://github.com/elastic/elasticsearch/issues/19269#issuecomment-488598561</a><br>
<a href="https://www.elastic.co/guide/en/elasticsearch/reference/7.x/optimistic-concurrency-control.html">https://www.elastic.co/guide/en/elasticsearch/reference/7.x/optimistic-concurrency-control.html</a></p>
</blockquote>
<p>最后，给你留一个思考题：在一些极端情况下，同一个服务会被 OAP 集群分配两个不同的ServiceId 吗？</p>
<h3>服务实例注册</h3>
<p>通过本课程第二部分的介绍可知，SkyWalking Agent 在完成服务注册之后，会立即进行服务实例的注册。 SkyWalking OAP 处理服务实例注册请求的大致流程与前文介绍的服务注册处理流程基本类似，如下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/DD/CgqCHl7WHz-AadgsAAFmWx71MBY433.png" alt="image (15).png"></p>
<p>服务实例注册的 gRPC 请求由 RegisterServiceHandler.doServiceInstanceRegister() 方法进行处理：</p>
<ol>
<li>根据请求中携带的 ServiceId，从 ServiceInventoryCache 中获取 Service 的相关信息，主要获取的是服务名称，它将是构成服务实例名称的一部分。</li>
<li>处理请求携带的服务实例的附加信息，例如，系统名称、HostName、IP、进程 ID 等。</li>
<li>生成服务实例名称。服务实例名称一般是由服务名称、进程 ID、HostName 三部分构成。</li>
<li>将上述服务实例信息交给 ServiceInstanceInventoryRegister 进行处理。</li>
<li>返回 ServiceInstance UUID 与 ServiceInstanceId 的映射关系。</li>
</ol>
<p>ServiceInstanceInventoryRegister 处理服务实例注册请求的逻辑与前文介绍的 ServiceInventoryRegister 核心逻辑基本一致：</p>
<ol>
<li>先查询该 ServiceInstanceName 是否已分配了 ServiceInstanceId。这里同样是先查缓存、缓存 miss ，再查底层持久化存储。</li>
<li>如果已分配 ServiceInstanceId，直接将其返回。</li>
<li>如果未分配 ServiceInstanceId，则将 ServiceInstance 相关信息封装成 ServiceInstanceInventory 对象交给 InventoryStreamProcessor 进行处理。</li>
<li>在 InventoryStreamProcessor 中会为 ServiceInstanceInventory 数据分配相应的 Worker 链以完成 L1 、L2 聚合以及持久化存储。</li>
</ol>
<p>在上述过程中，使用到的 Worker、Cache、DAO 实现都与 ServiceInventory 一致，这里就不再展开分析了，如果你感兴趣可以翻看一下源码。</p>
<p>最后，我们一起来看一下 ServiceInstanceInventory 中的核心字段，以及与相应 ES 索引字段的映射关系，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/D1/Ciqc1F7WH0mAIjcKAAefOW_jMVc517.png" alt="image (16).png"></p>
<h3>NetWorkAddress、EndpointName 同步</h3>
<p>在前面分析 Skywalking Agent 时曾提到，Agent 将 Trace 数据中用到的 NetworkAddress、EndpointName 等字符串信息定时同步到后端 OAP，然后由后端 OAP 集群为其统一分配全局唯一的 ID。之后，在 Agent 上报数据时，会使用这些全局唯一 ID 替换相应的字符串，从而提高上报消息的有效负载，提高传输效率。</p>
<p>你可以先回忆一下，当 Agent 在使用 NetworkAddress 时，例如，创建 Exitpan 的时候会记录 remotePeer 信息，remotePeer 一般是 Host + Port 或 URL等字符串数据，该信息会暂存在 NetworkAddressDictionary 中，并定期发送 NetworkAddress 同步请求与 OAP 同步。</p>
<p>NetworkAddress 同步请求的处理流程如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/DD/CgqCHl7WH1KASBKJAAH06v4jr4g358.png" alt="image (17).png"></p>
<p>NetworkAddressInventoryRegister.getOrCreate() 方法的处理步骤如下：</p>
<ol>
<li>查找指定 NetworkAddress 字符串在 network_address_inventory 索引中的对应 ID（addressId）。查询时先查询 NetworkAddressInventoryCache 缓存，再查询底层的 ElasticSearch 索引。若查找失败，会通过 InventoryStreamProcessor在 network_address_inventory 索引中为该 NetworkAddress 字符串生成相应 ID，此时getOrCreate() 方法返回 0；若查找 addressId 成功，继续执行步骤 2。</li>
<li>根据步骤 1 得到的 addressId 以及 NetworkAddress 字符串，在 service_inventory 索引中查找 NetworkAddress 与服务之间的绑定关系。若查找失败，则通过 InventoryStreamProcessor 创建这个绑定关系；若查询成功，则继续执行步骤 3。</li>
<li>根据步骤 2 查询到的 ServiceId 以及 addressId，在 service_instance_inventory 索引中查找该 NetworkAddress 与服务实例的绑定关系。若查询失败，则由 InventoryStreamProcessor 创建该绑定关系；若查询成功，则返回步骤 1 中得到的addressId。</li>
</ol>
<p>从上述流程来看，一个 NetworkAddress 字符串除了在 network_address_inventory 索引中分配唯一对应的 addressId 外，还会在 service_inventory 和 service_instance_inventory 两个索引中创建与服务、服务实例的绑定。</p>
<p>处理 EndpointName同步请求的流程与处理服务注册请求的逻辑类似，不再展开分析。</p>
<h3>心跳请求</h3>
<p>完成服务注册以及服务实例的注册之后，Agent 会定时调用 ServiceInstancePing.doPing() 这个 gRPC 接口发送心跳请求，以通知后端 OAP 集群当前 Agent 的在线状态。Agent 的心跳逻辑在前面介绍过了，所以这里重点来看后端 OAP 中心跳请求的处理逻辑。</p>
<p>你可以先来看一下 RegisterModuleProvider.start() 方法，它不仅将前面分析的 RegisterServiceHandler 注册到 GRPCServer 上，同时还会将处理心跳请求的 ServiceInstancePingServiceHandler 注册上去。</p>
<p>ServiceInstancePingServiceHandler.doPing() 方法会从心跳请求中解析出客户端对应的 ServiceId 以及 ServiceInstanceId，然后更新服务和服务实例的 heartbeat_time 字段，大致实现如下：</p>
<pre><code data-language="java" class="lang-java"> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doPing</span><span class="hljs-params">(ServiceInstancePingPkg request, 
         StreamObserver&lt;Commands&gt; responseObserver)</span> </span>{
    <span class="hljs-comment">// 从心跳请求中获取 serviceInstanceId</span>
    <span class="hljs-keyword">int</span> serviceInstanceId = request.getServiceInstanceId(); 
    <span class="hljs-comment">// 心跳请求的发送时间</span>
    <span class="hljs-keyword">long</span> heartBeatTime = request.getTime();
    <span class="hljs-comment">// 更新服务实例的心跳时间(service_instance_inventory索引中相应Document</span>
    <span class="hljs-comment">//&nbsp;的heartbeat_time字段） </span>
    serviceInstanceInventoryRegister.heartbeat(serviceInstanceId, 
        heartBeatTime);
    ServiceInstanceInventory serviceInstanceInventory = 
        serviceInstanceInventoryCache.get(serviceInstanceId);
    <span class="hljs-keyword">if</span> (Objects.nonNull(serviceInstanceInventory)) {
        <span class="hljs-comment">// 更新相应服务的心跳时间(service_inventory索引中相应Document的</span>
        <span class="hljs-comment">//&nbsp;heartbeat_time字段）</span>
        serviceInventoryRegister.heartbeat(
          serviceInstanceInventory.getServiceId(), heartBeatTime);
    } <span class="hljs-keyword">else</span> {
        logger.warn(<span class="hljs-string">"..."</span>, serviceInstanceId);
    }
    responseObserver.onNext(Commands.getDefaultInstance());
    responseObserver.onCompleted();
}
</code></pre>
<h3>总结</h3>
<p>本课时紧接上一课时的内容，首先通过介绍 RegisterLockDAOImpl 的核心原理，完成了对整个服务注册流程的介绍，接下来，又介绍了服务实例注册请求、NetworkAddress 同步请求以及心跳请求的处理流程。</p>

---

### 精选评论

##### **斌：
> 回答思考题：先说结论 在一些极端情况下，同一个服务oap集群是会为其分配不同的serviceId首先oap是通过es中register_lock索引中registerSource对应的scopeId对应的文档中的seqNo分配serviceId。这里我能够想到的极端情况就是多个相同的service注册请求到达不同的oap集群端点，这时正好register_lock的主分片挂了并且数据还没来得及通过到副本分片，重新选取主分片。这时只用文档的version并不能作为乐观锁的条件来证明文档是否被其他线程修改过。相当于没有加锁。所以可能会出现oap集群为同一service分配不同的serviceId的情况。

