<p>前面的课时提到，SkyWalking OAP 底层支持 ElasticSearch、H2、MySQL 等多种持久化存储，同时也支持读取其他分布式链路追踪系统的数据，例如，jaeger（Uber 开源的分布式跟踪系统）、zipkin（Twitter 开源的分布式跟踪系统）。下图展示了 OAP 提供的针对不同持久化存储的插件模块：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F6/Ciqc1F7M1mOAdkinAAUHgWAh2x8972.png" alt="image001.png"></p>
<p>在 server-core 模块中对整个 OAP 的持久化有一个整体的抽象，具体位置如下图所示，而上述这些插件都是在此基础上扩展实现的：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F6/Ciqc1F7M1nCAO5cwAAKxM_2j59I530.png" alt="image003.png"></p>
<p>首先，OAP 存储了两种类型的数据：时间相关的数据和非时间相关的数据（与“时序”这个专有名词区分一下）。注册到 OAP 集群的 Service、ServiceInstance 以及同步的 EndpointName、NetworkAddress 都是非时间相关的数据，一个稳定的服务产生的这些数据是有限的，我们可以用几个固定的 ES 索引（或数据库表）来存储这些数据。</p>
<p>而像 JVM 监控等都属于时间相关的数据，它们的数据量会随时间流逝而线性增加。如果将这些时间相关的数据存储到几个固定的 ES 索引中，就会导致这些 ES 索引（或数据库表）非常大，这种显然是不能落地的。既然一个 ES 索引（或数据库表）存不下，一般会考虑切分，常见的切分方式按照时间窗口以及 DownSampling 进行切分。</p>
<p>这里简单介绍一下 DownSampling 这个概念。DownSampling（翻译为"向下采样"或是"降采样"）是降低数据采样率或分辨率的过程。这里通过一个示例进行说明，假设 Agent 每隔一分钟进行一次 JVM Old GC 时间的采样，并发送到 Skywalking OAP 集群，一小时之后，我们可以到 60 个点（Timestamp 和 Value 构成一个点），在一张二维图（横轴是时间戳，纵轴是 Old GC 耗时）上将这些点连接起来，可以绘制出该 JVM 实例在这一小时内的 Old GC 监控曲线。</p>
<p>假设我们查询该 JVM 实例跨度为一周的 Old GC 监控，将获得 10080 个点，监控图中点与点之间的距离会变得非常密。此时，就可以通过 DownSampling 的方式，将一个时间范围内的多个监控数据点聚合成单个点，从而减少需要绘制的点的个数。这里我们可以按照小时为时间窗口，计算一小时内 60 点的平均值，作为该小时的 DownSampling 聚合结果。下图为 13:00 ~ 14:00 以及 14:00 ~ 15:00 两小时的 DownSampling 示意图，经过 DownSampling 之后，一周的监控数据只有 168 个点了。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F6/Ciqc1F7M1nyAFfhMAAP8mBituI8522.png" alt="image005.png"></p>
<p>另外，Trace 也是时间相关的数据，其数据量会随时间不断增加，但不具备可聚合的特性。</p>
<h3>Model 与 ES 索引</h3>
<p>明确了 OAP 要存储的数据特性，我们回到 server-core 模块继续分析。常见的 ORM 框架会将数据中的表结构映射成 Java 类，表中的一个行数据会映射成一个 Java Bean 对象，在 OAP 的存储抽象中也有类似的操作。SkyWalking 会将 [Metric + DownSampling] 对应的一系列 ES 索引与一个 Model 对象进行映射，下图展示了 instance_jvm_old_gc_time 这个 Metric（指标）涉及的全部 ES 索引以及对应的 Model 对象：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/02/CgqCHl7M1qqAWJwzAAJI_qj62kA075.png" alt="image007.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/02/CgqCHl7M1rCAF_BRAAIjEdyxGKM865.png" alt="image009.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F6/Ciqc1F7M1riAbapoAAIgfkX2Ecw638.png" alt="image011.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/02/CgqCHl7M1saAc6guAAIMU20bW9s987.png" alt="image013.png"></p>
<p>Model 对象记录了对应 ES 索引的核心信息：</p>
<ul>
<li><strong>name（String类型）</strong>：Metric 名称，在 OAP 创建 ES 索引时会使用</li>
<li><strong>columns（List<modelcolumn>类型）</modelcolumn></strong>：ES 索引 中的 Field 集合，一个 ModelColumn 对象记录了一个 Field 的名称、类型等信息。</li>
<li><strong>capableOfTimeSeries（boolean类型）</strong>：对应 ES 索引中存储的数据是否为时间相关的数据。</li>
<li><strong>downsampling（Downsampling类型）</strong>：如果是时间相关的数据，则需要指定其 Downsampling 单位，可选值有 Second、 Minute、 Hour、 Day、 Month。对于非时间相关的数据，则该字段值为 Downsampling.NONE。</li>
<li><strong>deleteHistory（boolean类型）</strong>：是否删除历史数据。</li>
<li><strong>scopeId（int类型）</strong>：对应指标的全局唯一 ID。</li>
</ul>
<p>很明显，ES 索引的名称由三部分构成：Metric 名称、DownSampling、时间窗口（后面两部分只有时序数据才会有），而 ES 索引的别名由 Metric 名称和 DownSampling 构成。</p>
<p>下图展示了 Model.columns 集合与 ES 索引中 Field 之间的映射关系，除了名称之间的映射关系之外，ModelColumn 中记录了每个 Field 的类型。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M1tWAGod6AAXHbG78X9I245.png" alt="image015.png"></p>
<p>在 CoreModuleProvider 启动过程中，会扫描 @Stream 注解，创建相应的 Model 对象，并由 StorageModels 统一管理（底层维护了 List<model> 集合）。 @Stream 扫描过程后面介绍，这里只需知道 @Stream 中会包含 Model 需要的信息即可。StorageModels 同时实现了 IModelSetter、IModelGetter、IModelOverride 三个 Service 接口，如下图所示。</model></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/02/CgqCHl7M1uGAFsn_AAHpuk8RDn4882.png" alt="image017.png"></p>
<p>IModelGetter、IModelSetter 定义了读取和新增 Model 实例的方法，IModelOverride 接口提供的 overrideColumnName() 方法一般用在以 DB 为存储的插件模块中，当列名与数据库关键字冲突时，会通过该方法修改列名。</p>
<p>在 CoreModuleProvider 的 prepare() 方法中会将 StorageModels 作为上述三个 Service 接口的实现注册到 services 集合中。</p>
<h3>初始化存储结构</h3>
<p>很多 ORM 框架（例如，Hibernate ）提供了在应用启动时根据 Java Bean 初始化表结构的功能。SkyWalking OAP 也提供了类似的功能，主要在 ModelInstaller 中完成，核心在 install() 方法中：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">install</span><span class="hljs-params">(Client client)</span> <span class="hljs-keyword">throws</span> StorageException </span>{
    <span class="hljs-comment">// 获取全部Model对象</span>
    IModelGetter modelGetter = moduleManager.find(CoreModule.NAME)
      .provider().getService(IModelGetter<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
    List&lt;Model&gt; models = modelGetter.getModels();
    <span class="hljs-comment">// 根据mode这个环境变量决定是否创建底层存储结构</span>
    <span class="hljs-keyword">if</span> (RunningMode.isNoInitMode()) {  
        ... <span class="hljs-comment">// mode环境变量为no-init，则只会输出日志，不会初始化ES索引</span>
    } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// mode环境变量为非non-init值，走这个分支</span>
        <span class="hljs-keyword">for</span> (Model model : models) {
            <span class="hljs-comment">// 检查Model对应的底层存储结构是否存在，如果不存在则通过</span>
            <span class="hljs-comment">//&nbsp;createTable()方法进行创建</span>
            <span class="hljs-keyword">if</span> (!isExists(client, model)) { 
                createTable(client, model); 
            }
        }
    }
}
</code></pre>
<p>ModelInstaller 是一个抽象类，其中的 createTable() 是个抽象方法，针对具体存储的具体创建流程由对应的子类完成，继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M1vSAKUcCAACnS4z1sy0205.png" alt="image019.png"></p>
<p>StorageEsInstaller 是针对 ElasticSearch 的实现，位于 storage-elasticsearch-plugin 模块中，该模块的 ModuleProvider 实现是 StorageModuleElasticsearchProvider（依赖于 CoreModule），在其 start() 方法中会调用 StorageEsInstaller.intall() 方法完成 ES 索引的初始化。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ModuleStartException </span>{
&nbsp; &nbsp; elasticSearchClient.connect();
&nbsp; &nbsp; StorageEsInstaller installer = <span class="hljs-keyword">new</span> StorageEsInstaller(...);
&nbsp; &nbsp; installer.install(elasticSearchClient);
}
</code></pre>
<p>这里使用的  ElasticSearchClient 是 SkyWalking 在 RestHighLevelClient 之上的封装，帮上层使用方屏蔽了一些通用的 Request 请求构造以及 Response 处理逻辑。</p>
<p>下面深入到 StorageEsInstaller 中的 createTable() 方法， 在存储 Metric 、Trace 等数据时会随时间推移新创建多个索引，一般会先创建统一的模板，指定公共参数、字段类型等，之后再创建索引；存储服务注册等信息的索引都只有一个，直接创建索引即可。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createTable</span><span class="hljs-params">(Client client, Model model)</span> </span>{
&nbsp; &nbsp; ElasticSearchClient esClient = (ElasticSearchClient)client;
&nbsp; &nbsp; <span class="hljs-comment">// 创建settings，其中指定了索引的分片数量、副本数量以及refresh时间间隔</span>
&nbsp; &nbsp; JsonObject settings = createSetting();
&nbsp; &nbsp; <span class="hljs-comment">// 创建mapping，其中指定了各个Field的类型等配置</span>
&nbsp; &nbsp; JsonObject mapping = createMapping(model);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (model.isCapableOfTimeSeries()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 对于时间相关的索引，先创建Template，其中除了指定settings和mapping</span>
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">//&nbsp;之外，还会指定该Template匹配的索引名称(index_patterns)以及</span>
        <span class="hljs-comment">// 别名(aliases)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!esClient.isExistsTemplate(model.getName())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> isAcknowledged = esClient.createTemplate(
                model.getName(), settings, mapping);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!esClient.isExistsIndex(model.getName())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// ElasticSearch中真正的索引名称会添加时间戳后缀</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String timeSeriesIndexName = TimeSeriesUtils
                      .timeSeries(model);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> isAcknowledged = esClient.createIndex(
                      timeSeriesIndexName);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 与时间无关的索引只会创建一个索引，直接使用Model.name作为索引名</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//&nbsp;称创建(没有时间戳后缀)，另外，同样会设置settings和mapping</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> isAcknowledged = esClient.createIndex(model.getName(), 
            settings, mapping);
&nbsp; &nbsp; }
}
</code></pre>
<p>这里有几个点需要关注一下，首先是模板的相关内容，以 instance_jvm_old_gc_time 这个指标为例，每个 DownSampling 维度对应一个模板，每个模板中的 index_patterns 、aliases 配置以及匹配的索引名称，如下表所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M1wGAI3qgAACMKwr5nRI057.png" alt="image021.png"></p>
<p>可以看到模板中指定的别名与 Model.name 是相同的，在后续查询操作中，我们可以直接通过别名查询该模板匹配的全部索引，但是写入时还是要明确指定具体索引名称的。</p>
<p>另一个点是索引名称的时间后缀。在 TimeBucket 工具类中会根据指定的 DownSampling 将毫秒级时间进行整理，得到相应的时间窗口，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/02/CgqCHl7M1yCAbH7cAAmLbCfxgJc857.png" alt="image023.png"></p>
<p>拿到时间戳所在的窗口之后，TimeSeriesUtils 工具类会将其转换成对应索引的时间戳后缀：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> String <span class="hljs-title">timeSeries</span><span class="hljs-params">(String modelName, 
        <span class="hljs-keyword">long</span> timeBucket, Downsampling downsampling)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">switch</span> (downsampling) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> None:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> modelName;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> Hour: <span class="hljs-comment">// 一天内的Hour级别数据存储到同一个索引之中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> modelName + Const.LINE + timeBucket / <span class="hljs-number">100</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> Minute: <span class="hljs-comment">// 一天内的分钟级数据存储到一个索引之中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> modelName + Const.LINE + timeBucket / <span class="hljs-number">10000</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> Second: <span class="hljs-comment">// 一天内的秒级数据存储到一个索引之中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> modelName + Const.LINE + timeBucket / <span class="hljs-number">1000000</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">default</span>: <span class="hljs-comment">// 对于Day、Month以及Year不做处理，直接用时间窗口作为后缀</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> modelName + Const.LINE + timeBucket;
&nbsp; &nbsp; }
}
</code></pre>
<p>到此为止，StorageEsInstaller 初始化 ES 索引的核心流程就介绍完了。</p>
<h3>数据抽象</h3>
<p>了解了索引（或是数据库表结构）的初始化流程之后，再来看 SkyWalking OAP是如何对持久化数据进行抽象的。</p>
<p>SkyWalking 与 ORM 类似，会将索引中的一个 Document （或是数据库表中的一条数据）映射为一个 StorageData 对象。StorageData 接口中定义了一个 id() 方法，负责返回该条数据的唯一标识。实现 StorageData 接口的有三类数据，分别对应三个抽象类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M1y2AAEnMAACsBUVPIcg418.png" alt="image025.png"></p>
<ul>
<li>
<p><strong>Metrics</strong>：所有监控指标的顶级抽象，其中定义了一个 timeBucket 字段（long类型），它是所有监控指标的公共字段，用于表示该监控点所处的时间窗口。另外，timeBucket 字段被 @Column 注解标记，在 OAP 启动时会被扫描到转换成 Model 中的 ModelColumn，在初始化 ES 索引时就会创建相应的 Field。Metrics 抽象类中还定义了计算监控数据的一些公共方法：</p>
<ul>
<li>
<p>calculate() 方法：大部分 Metrics 都会有一个 value 字段来记录该监控点的值，例如，CountMetrics 中的 value 字段记录了时间窗口内事件的次数，MaxLongMetrics 中的 value 字段记录时间窗口内的最大值。calculate() 方法就是用来计算该 value 值。</p>
</li>
<li>
<p>combine() 方法：合并两个监控点。对于不同含义的监控数据，合并方式也有所不同，例如，CountMetrics 中 combine() 方法的实现是将两个监控点的值进行求和；MaxLongMetrics 中 combine() 方法的实现是取两个监控点的最大值。</p>
</li>
</ul>
</li>
<li>
<p><strong>Record</strong>：抽象了所有记录类型的数据，其子类如下图所示。其中 SegmentRecord 对应的是 TraceSegment 数据、AlarmRecord 对应一条告警、TopNDatabaseStatement 对应一条慢查询记录，这些数据都是一条条的记录。</p>
</li>
</ul>
<p><img src="https://s0.lgstatic.com/i/image/M00/12/03/CgqCHl7M1ziAUV3lAADO8_KdBjs730.png" alt="image027.png"></p>
<p>上述记录类型的数据也有一个公共字段 —— timeBucket（long 类型），表示该条记录数据所在的时间窗口，也同样被 @Column 注解标记。Record 抽象类中没有定义其他的公共方法。</p>
<ul>
<li><strong>RegisterSource</strong>：抽象了服务注册、服务实例注册、EndpointName（以及 NetworkAddress）同步三个过程中产生的数据。其中，定义了三个公共字段，且这三个字段都被 @Column 注解标注了：
<ul>
<li>sequence（int 类型）：上述三个过程中的数据都会产生一个全局唯一的 ID，该全局 ID 就保存在该字段中。</li>
<li>registerTime（long 类型）：第一注册（或同步）时的时间戳。</li>
<li>heartbeatTime（long 类型）：心跳时间戳。</li>
</ul>
</li>
</ul>
<p>在这三个抽象类下还有很多具体的实现类，这些实现类会根据对应的具体数据，扩展新的字段和方法，在后面介绍具体模块时会展开细说。</p>
<p>最后，我们来看 StorageBuilder 这个接口，它与 StorageData 接口的关系非常紧密，在 StorageData 的全部实现类中，都有一个内部 Builder 类实现类了 StorageBuilder 接口。StorageBuilder 接口中定义了 map2Data() 和 data2Map() 两个方法，实现了 StorageData 对象与 Map 之间的相互转换。</p>
<h3>DAO 层架构</h3>
<p>在创建 ES 索引时使用到的 ElasticSearchClient， 是对 RestHighLevelClient 进行了一层封装，位于 library-client 模块，其中还提供了 JDBC 以及 gRPC 的 Client，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M10iAQOQeAACK78d1tQw384.png" alt="image029.png"></p>
<p>GRPCClient 底层会与指定地址创建 gRPC ManagedChannel 连接，在后面的课程中会看到， OAP 节点之间的通信就是依赖 GRPCClient 实现的。</p>
<p>JDBCHikariCPClient 底层封装了 HikariCP 连接池，对外提供了 execute()、executeQuery() 等执行 SQL 语句的方法。</p>
<p>library-client 模块相对独立，如果同学们在实际开发中需要封装 ElasticSearch 客户端或是 JDBC 客户端，都可以直接拿来使用。</p>
<p>虽然  ElasticSearchClient 对 RestHighLevelClient 的通用操作进行了封装，但如果上层逻辑直接使用，还是必须了解 RestHighLevelClient 的相关概念，所以在实践中会针对业务封装出一套 DAO 层。在 DAO 层会完成业务概念与存储概念的转换，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M11iAEBxFAAGikNTBius816.png" alt="image031.png"></p>
<p>SkyWalking OAP 也提供了DAO 层抽象，如下图所示，大致可以分为三类：Receiver DAO、Cache DAO、Query DAO。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M12iAYJt9AAIIw288AD4900.png" alt="image033.png"></p>
<ul>
<li><strong>Receiver DAO 接口</strong>：在各个 receiver 模块接收到 Agent 上报的 Metrics 数据、Trace 数据以及注册请求的时候，会通过 Receiver DAO 将数据持久化到底层存储，具体的接口如下图所示：</li>
</ul>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M13SAGmxvAADxT4960Jo386.png" alt="image035.png"></p>
<ul>
<li><strong>IRegisterDAO 接口</strong>：负责增删改查 RegisterSource 数据。</li>
<li><strong>IMetricsDAO 接口</strong>：负责增删改查 Metrics 数据。</li>
<li><strong>IRecordDAO 接口</strong>：负责增删改查 Record 数据。</li>
<li><strong>IBatchDAO 接口</strong>：上述接口都是单条数据的操作接口，IBatchDAO 接口提供了批量写入的功能。</li>
<li><strong>IRegisterLockDAO 接口</strong>：在处理 RegisterSource 数据的时候，需要维护一把全局的锁，这样才能保证写入生成的 ID 全局唯一。IRegisterLockDAO 接口的实现就提供了全局锁的功能。</li>
<li><strong>Cache DAO 接口</strong>：在稳定集群中，RegisterSource 类型数据基本不会变化，这里的 Cache DAO 接口主要提供了 RegisterSource 数据的查询，相较于 IRegisterDAO，Cache DAO 的接口提供了更多维度的查询方式。</li>
</ul>
<p>如下图所示，每一种 RegisterSource 数据都对应一个 Cache，通过 Cache 的方式提高查询性能，每个 Cache 底层都关联了一个 Cache DAO 接口实现。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M132AMFlHAACbaBYDnW8720.png" alt="image037.png"></p>
<p>另外，在 CoreModule 中还会启动 CacheUpdateTimer，其中会启动一个后台线程，定期更新 ServiceInventoryCache 缓存。</p>
<ul>
<li><strong>Query DAO 接口</strong>：Query DAO 接口负责支撑 query 模块处理 SkyWalking Rocketbot 发来的查询请求，接口参数更加贴近用户请求参数。下图展示了所有 Query DAO 接口，从接口名称即可看出查询的是哪些数据。</li>
</ul>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M14eAfVPdAABls_BG5tE854.png" alt="image039.png"></p>
<p>SkyWalking OAP 中 DAO 层的整体架构以及核心接口就介绍完了。</p>
<h3>数据 TTL</h3>
<p>前文提到，Metrics、Trace 等（时间相关的数据）对应的 ES 索引都是按照时间进行切分的，随着时间的推移，ES 索引会越来越多。为了解决这个问题，SkyWalking 只会在 ES 中存储一段时间内的数据，CoreModuleProvider 会启动 DataTTLKeeperTimer 定时清理过期数据。</p>
<p>在 DataTTLKeeperTimer 中会启动一个后台线程，每 5 分钟执行一次清理操作，具体执行步骤如下：</p>
<ol>
<li>通过 ClusterNodesQuery 查询到当前 OAP 集群中全部节点列表，如果当前节点为列表中第一个节点，才能继续执行后续的清理操作。这就保证不会出现多个 OAP 节点并发执行清理任务。</li>
<li>从 IModelGetter 中拿到全部 Model 对象，根据 Model 数据的 DownSampling 计算每个 Model 保留 ES 索引的范围。这里使用到了 StorageTTL 接口，其中会根据每个 Model 不同的 DownSampling 返回相应的 TTLCalculator，如下图所示：</li>
</ol>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/F7/Ciqc1F7M15KASkiEAABTiYqEwqw229.png" alt="image041.png"></p>
<p>对于一个 Model，通过 TTLCalculator 计算之后会得到一个时间，在该时间之前的 ES 索引将被删除。</p>
<p>在删除 ES 索引的过程成还有一些细节：</p>
<ol>
<li>首先根据模板别名（也就是 Model.name）拿到该 Model 对应的全部 ES 索引。</li>
<li>根据 ES 索引的时间后缀，以及 TTLCalculator 计算得到的时间，确定哪些索引会要被删除。</li>
<li>如果该 Model 对应的全部索引都要被删除，则会创建一个新索引，为后面写入数据做准备。</li>
<li>循环调用 ElasticSearchClient.delete() 方法删除索引。</li>
</ol>
<h3>总结</h3>
<p>本课时重点介绍了 SkyWalking OAP 持久化存储方面的架构设计以及核心接口功能。首先介绍了 Model 与底层 ES 索引之间的映射关系，之后介绍了 ModelInstaller 是如何在 OAP 启动时初始化 ES 索引的。接下来，介绍了 OAP 对不同持久化数据的抽象，以及对应的 DAO 层设计。最后介绍了定期删除过期数据的核心原理。</p>
<p>特别注意，本课时重在顶层设计，不在具体实现，在后面深入分析各个模块的具体实现时，会深入上述接口的实现细节。</p>

---

### 精选评论


