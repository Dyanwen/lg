<p data-nodeid="55685">SkyWalking OAP 目前只提供了query-graphql-plugin 这一款查询插件，从名字就可以看出它是使用 GraphQL 实现的查询 API。本课时将深入分析 query-graphql​-plugin 模块的核心原理。</p>



<h3 data-nodeid="56219" class="">启动逻辑</h3>

<p data-nodeid="54879">首先我们需要了解的是 query-graphql-plugin 插件是如何将 GraphQL 与 OAP 自身的 JettyServer &nbsp;Handler 体系进行集成的。这部分集成逻辑是在 GraphQLQueryProvider 中实现的，它是 query-grapql-plugin 插件&nbsp;SPI 文件中指定的唯一一个 ModuleProvider 实现，其中主要完成下面三件事：</p>
<ol data-nodeid="54880">
<li data-nodeid="54881">
<p data-nodeid="54882">通过 GraphQL Java Tools 实现 GraphQL Schema 与 POJO 之间的映射，创建相应的 GraphQLSchema 对象。如何使用 GraphQL Java Tools 以及 Resolver 与 POJO 映射的映射规则在前面的 GraphQL Java Tools&nbsp;入门中已经详细介绍过了。</p>
</li>
<li data-nodeid="54883">
<p data-nodeid="54884">通过 GraphQL Java API 创建 GraphQL 对象，它将处理"/graphql"路径上的全部请求。</p>
</li>
<li data-nodeid="54885">
<p data-nodeid="54886">创建 GraphQLQueryHandler 实例并注册到 JettyServer。GraphQLQueryHandler 会将收到的 Http 请求进行一次转换，并交给 GraphQL 对象进行处理。</p>
</li>
</ol>
<p data-nodeid="54887">GraphQLQueryProvider 的核心实现如下所示：</p>
<pre class="lang-java" data-nodeid="58352"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">GraphQLQueryProvider</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ModuleProvider</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> GraphQLQueryConfig config = <span class="hljs-keyword">new</span> GraphQLQueryConfig();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;GraphQL&nbsp;graphQL;
&nbsp;&nbsp;&nbsp;&nbsp;
    <span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">prepare</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ServiceNotProvidedException, ModuleStartException </span>{
        GraphQLSchema schema = SchemaParser.newParser()
            .file(<span class="hljs-string">"query-protocol/common.graphqls"</span>)
            .resolvers(<span class="hljs-keyword">new</span> Query(), <span class="hljs-keyword">new</span> Mutation())
            ... ... <span class="hljs-comment">// 这里会添加所有 GraphQL Schema以及关联的 Resolver实现，后面会挑选几个展开详述</span>
            .build() .makeExecutableSchema();
        <span class="hljs-comment">// 创建 GraphQL 对象， GraphQL Java提供的API</span>
        <span class="hljs-keyword">this</span>.graphQL = GraphQL.newGraphQL(schema).build();
    }
    <span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> ServiceNotProvidedException, ModuleStartException </span>{
        <span class="hljs-comment">// 创建 GraphQLQueryHandler实例并注册到 JettyServer中</span>
        JettyHandlerRegister service = getManager().find(CoreModule.NAME).provider().getService(JettyHandlerRegister.class);
        // 这里的 path在 application.yml中的 query部分有相应配置项，默认是"/graphql"
        service.addHandler(<span class="hljs-keyword">new</span> GraphQLQueryHandler(config.getPath(), graphQL));
    }
}
</code></pre>




<h4 data-nodeid="58885" class="">GraphQLQueryHandler</h4>

<p data-nodeid="59943">在前面介绍中提到，server-core 模块会启动两个 Server，一个是 GRPCServer，主要用于接收 Agent 发送来的 gRPC 请求，前文介绍的 RegisterServiceHandler、JVMMetricReportServiceHandler、TraceSegmentReportServiceHandler 等都是注册在 GRPCServer 上的 Handler；另一个是 JettyServer，用于接收 Http 请求，本小节介绍的 GraphQLQueryHandler 就是注册在 JettyServer 的 Handler，它继承 JettyJsonHandler 如下图所示：</p>
<p data-nodeid="59944" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/44/CgqCHl7xt6yAHDDLAAIeqgBo9HE860.png" alt="Drawing 0.png" data-nodeid="59948"></p>


<p data-nodeid="54892">JettyJsonHandler 使用模板方法模式将真正的请求处理逻辑延迟到子类实现，而在其&nbsp;doGet() 方法和 doPost() 方法中只完成了下面几项通用的逻辑：</p>
<ol data-nodeid="54893">
<li data-nodeid="54894">
<p data-nodeid="54895">设置 HttpResponse 响应头；</p>
</li>
<li data-nodeid="54896">
<p data-nodeid="54897">将请求处理结果（JSON&nbsp;数据）写入返回给客户端；</p>
</li>
<li data-nodeid="54898">
<p data-nodeid="54899">如果请求处理过程中出现异常，则在响应的 JSON 中携带 error-message 字段记录简单的异常信息。</p>
</li>
</ol>
<p data-nodeid="54900">GraphQLQueryHandler 只支持 POST 请求，不支持 GET 请求，其 doPost() 方法中首先会读取 JSON 格式的请求体，并用其中数据创建 ExecutionInput 对象，execute() 方法是 GraphQL 对象处理请求的入口，ExecutionInput 是其唯一的参数，execute() 方法返回 ExecutionResult 对象，其中封装了查询得到的 GraphQL Schema 对象（正常情况）以及错误信息（异常情况），具体实现如下：</p>
<pre class="lang-java" data-nodeid="60481"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> JsonElement <span class="hljs-title">doPost</span><span class="hljs-params">(HttpServletRequest req)</span> <span class="hljs-keyword">throws</span> IOException </span>{
    BufferedReader reader = <span class="hljs-keyword">new</span> BufferedReader(<span class="hljs-keyword">new</span> InputStreamReader(req.getInputStream()));
    StringBuilder request = <span class="hljs-keyword">new</span> StringBuilder();
    <span class="hljs-comment">// 省略读取reader的过程</span>
    JsonObject requestJson = gson.fromJson(request.toString(), JsonObject.class);
    ExecutionInput executionInput = ExecutionInput.newExecutionInput()
        .query(requestJson.get(QUERY).getAsString())
        .variables(gson.fromJson(requestJson.get(VARIABLES), mapOfStringObjectType))
        .build();
    // 在前文的示例中，Spring Boot 帮我们屏蔽了 execute()方法的调用，这里需要自己通过GraphQL Java API进行调用
    ExecutionResult executionResult = graphQL.execute(executionInput);
    Object data = executionResult.getData(); // 正常查询结果
    List&lt;GraphQLError&gt; errors = executionResult.getErrors(); // 异常信息
    JsonObject jsonObject = <span class="hljs-keyword">new</span> JsonObject();
    <span class="hljs-comment">// 将正常查询结果记录到"data"字段，将异常信息记录到"error"(略)</span>
    <span class="hljs-keyword">return</span> jsonObject;
}
</code></pre>

<p data-nodeid="62596">到此处为止，query-graphql-plugin 插件处理查询请求的核心流程就介绍完了，通过下面一张图，可以很好地总结该流程：</p>
<p data-nodeid="63668"><img src="https://s0.lgstatic.com/i/image/M00/26/38/Ciqc1F7xt76AflHaAANfIPXqD0Q765.png" alt="Drawing 1.png" data-nodeid="63671"></p>





<h3 data-nodeid="63134" class="">GraphQL Schema 鸟瞰</h3>

<p data-nodeid="64462">在 resouces/query-protocol 目录中包含了 query-graphql-plugin 插件的全部 GraphQL Schema 文件，其结构如下图所示，该结构图是通过 GraphQL Voyager 工具生成的，如果你感兴趣可以查找相关资料进行了解。</p>
<p data-nodeid="64463" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/39/Ciqc1F7xt8uAa-I6AAjKWi_tgPI783.png" alt="Drawing 2.png" data-nodeid="64467"></p>


<p data-nodeid="54906">在学习了前面介绍的 GraphQL Schema 基本语法和示例之后，相信你已经完全能够读懂上图涉及的全部 GraphQL Schema 定义，这里就不再一一展开分析，我们将重点放在关联的 Resolver 以及具体的查询实现上。</p>
<h3 data-nodeid="65000" class="">MetadataQuery</h3>

<p data-nodeid="66058">query-graphql-plugin 插件中提供了三个查询 Service 的方法，如下图所示：</p>
<p data-nodeid="66059" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/39/Ciqc1F7xt9WABcbdAAI-KmsR4xQ745.png" alt="Drawing 3.png" data-nodeid="66063"></p>


<p data-nodeid="67120">GraphQL Java Tools&nbsp;会将上述三个查询 Service 的方法映射到&nbsp;MetadataQuery 中的同名方法，如下图所示，MetadataQuery 会将请求委托给 MetadataQueryService 的同名方法处理，而 MetadataQueryService 中也没有其他逻辑，直接将请求委托给 MetadataQueryEsDAO 的同名方法：</p>
<p data-nodeid="67121" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/44/CgqCHl7xt92AR6MbAAT0t4jgEsA600.png" alt="Drawing 4.png" data-nodeid="67125"></p>


<p data-nodeid="54912">在 MetadataQuery 的这三个方法中都有一个 Duration 入参，在 metadata.graphqls 文件中定义了 Duration 这个 input 类型，该参数指定了查询的起止时间以及时间单位。</p>
<p data-nodeid="54913">MetadataQueryEsDAO&nbsp;底层通过&nbsp;High Level REST Client 对 ElasticSearch 的查询。先来看 searchServices() 方法的具体实现，其中会根据时间范围以及 serviceName 进行匹配：</p>
<pre class="lang-java" data-nodeid="67658"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Service&gt; <span class="hljs-title">searchServices</span><span class="hljs-params">(<span class="hljs-keyword">long</span> startTimestamp, <span class="hljs-keyword">long</span> endTimestamp,
    String keyword)</span> <span class="hljs-keyword">throws</span> IOException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;SearchSourceBuilder&nbsp;sourceBuilder&nbsp;=&nbsp;SearchSourceBuilder.searchSource();
    BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
    <span class="hljs-comment">// 查询的时间范围</span>
    boolQueryBuilder.must().add(timeRangeQueryBuild(startTimestamp, endTimestamp));
    <span class="hljs-comment">// 不查询 NetWorkAddress在 service_inventory索引中的数据 </span>
&nbsp;&nbsp;&nbsp;&nbsp;boolQueryBuilder.must().add(QueryBuilders.termQuery(ServiceInventory.IS_ADDRESS,&nbsp;BooleanUtils.FALSE));
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(!Strings.isNullOrEmpty(keyword))&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;serviceName匹配用户指定的关键字(keyword)</span>
        String matchCName = MatchCNameBuilder.INSTANCE.build(ServiceInventory.NAME);
        boolQueryBuilder.must().add(QueryBuilders.matchQuery(matchCName, keyword));
&nbsp;&nbsp;&nbsp;&nbsp;}
    sourceBuilder.query(boolQueryBuilder);
    sourceBuilder.size(queryMaxSize); <span class="hljs-comment">// 查询返回Document的个数上限，默认上限5000个</span>
    <span class="hljs-comment">// 通过 RestHighLevelClient 执行 SearchRequest查询</span>
    SearchResponse response = getClient().search(ServiceInventory.INDEX_NAME, sourceBuilder);
    <span class="hljs-comment">// 从 SearchResponse响应中获取相应的 Service信息并返回</span>
    <span class="hljs-keyword">return</span> buildServices(response);
}
</code></pre>

<p data-nodeid="68717">下图展示了 timeRangeQueryBuild() 方法构造的查询时间范围：</p>
<p data-nodeid="68718" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/39/Ciqc1F7xt-uAYJIjAANCIMBQIGg737.png" alt="Drawing 5.png" data-nodeid="68722"></p>


<p data-nodeid="54916">另外两个查询 Service 元数据的方法：getAllServices() 方法只根据时间范围进行查询，searchService() 方法只根据 serviceName 的关键字进行匹配，实现方式类似，这里不再展开详细分析。</p>
<p data-nodeid="54917">除了查询 Service，MetadataQuery&nbsp;还提供了查询其他多种基础元数据的相应方法：</p>
<ul data-nodeid="54918">
<li data-nodeid="54919">
<p data-nodeid="54920"><strong data-nodeid="55042">查询 ServiceInstance</strong></p>
</li>
</ul>
<p data-nodeid="54921">getServiceInstances() 方法可以按照时间范围和 serviceId 查询关联的 ServiceInstance 集合。</p>
<ul data-nodeid="54922">
<li data-nodeid="54923">
<p data-nodeid="54924"><strong data-nodeid="55047">查询 Endpoint</strong></p>
<ul data-nodeid="54925">
<li data-nodeid="54926">
<p data-nodeid="54927">searchEndpoint() 方法会根据 serviceId 以及关键字查询相应的 Endpoint 集合。</p>
</li>
<li data-nodeid="54928">
<p data-nodeid="54929">getEndpointInfo() 方法会根据指定的 endpointId 查询 Endpoint 信息。</p>
</li>
</ul>
</li>
<li data-nodeid="54930">
<p data-nodeid="54931"><strong data-nodeid="55053">查询 Databases</strong></p>
</li>
</ul>
<p data-nodeid="54932">getAllDatabases() 方法其实也是查询 Service ，只不过指定了 node_type 字段的取值为 Database 而已。</p>
<ul data-nodeid="54933">
<li data-nodeid="54934">
<p data-nodeid="54935"><strong data-nodeid="55060">查询 ClusterBrief</strong></p>
</li>
</ul>
<p data-nodeid="54936">getGlobalBrief() 方法会按照时间范围查询整个 OAP 集群所能感知到的各类组件的个数，然后封装成 ClusterBrief 对象返回。在 ClusterBrief 中包括 Service 数量、 Endpoint 数量、Database 数量、Cache 数量以及 MQ 数量。</p>
<p data-nodeid="54937">查询上述元数据的请求最终会委托给 MetadataQueryEsDAO 中的同名方法，然后依赖 High Level Rest Client 请求 ElasticSearch 进行查询，具体代码实现并不复杂，如果你感兴趣可以参考源码进行学习。</p>
<h3 data-nodeid="69255" class="">MetricQuery</h3>

<p data-nodeid="70313">在前面介绍 jvm-receiver-plugin 以及 trace-receiver-plugin 的章节中，我们详细介绍了 SkyWalking 中多种监控指标的计算方式以及存储实现，在 query-graphql-plugin 插件中自然是关注这些指标是如何查询的，在 metric.graphqls 文件中定义了下图三个查询监控指标的相关方法。</p>
<p data-nodeid="70314" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/44/CgqCHl7xuAKAGJpGAAEsPKwo6_0843.png" alt="Drawing 6.png" data-nodeid="70318"></p>


<ul data-nodeid="54941">
<li data-nodeid="54942">
<p data-nodeid="54943">getValues() 方法：返回一个聚合后的单值，例如，一个 Service 在一段时间内 SLA 的平均值。</p>
</li>
<li data-nodeid="54944">
<p data-nodeid="54945">getLinearIntValues() 方法：返回一条时序数据（即每个时间单位一个点，这些连续的点可以组成一张二维的监控图）。</p>
</li>
<li data-nodeid="54946">
<p data-nodeid="54947">getThermodynamic() 方法：返回的 heatmap（热力图）。</p>
</li>
</ul>
<h4 data-nodeid="70851" class="">查询单个聚合值</h4>

<p data-nodeid="71909">首先来看 MetricQuery.getValues() 方法，请求该方法的位置是在 SkyWalking Rocketbot  的拓扑图中，如下图所示：</p>
<p data-nodeid="71910" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/39/Ciqc1F7xuA2AZu34AAD4-d0xhHI072.png" alt="Drawing 7.png" data-nodeid="71914"></p>


<p data-nodeid="54951">图中的“每分钟请求量”“SLA”以及“延迟”三个值都是分别请求 getValues() 方法获得的，这三个值都是计算查询时间段内响应指标的平均值。</p>
<p data-nodeid="54952">getValues() 方法有两个入参，一个是 Duration 类型入参，用于指定查询时间范围，另一个是 是&nbsp;BatchMetricConditions 类型入参，其中指定了查询的 index alias 以及 entity_id 字段集合。以上图中&nbsp;SLA 这个指标为例，其 BatchMetricConditions.name 值为“service_sla”，entity_id 字段集合为 [2,3]（图中 demo-webapp 和 demo-provider 对应的 ServiceId 分别为 2 和 3）。</p>
<p data-nodeid="54953">MetricQuery 最终会将 getValues()&nbsp;请求委托给 MetricsQueryEsDAO 的同名方法，下面以查询 demo-provider 和 demo-webapp 两个 Service 在 2020 年 01 月 05 日 19:10~19:40&nbsp;的 SLA 为例，分析 MetricsQueryEsDAO.getValues() 方法的执行流程：</p>
<p data-nodeid="95888" class="">1、指定 time_bucket 字段的时间范围，即 time_bucket 字段值必须在 startTB 和 endTB 之间。相关代码片段如下：</p>
<pre class="lang-java" data-nodeid="95889"><code data-language="java">RangeQueryBuilder rangeQueryBuilder = QueryBuilders.rangeQuery(Metrics.TIME_BUCKET).gte(startTB).lte(endTB);
BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
boolQuery.must().add(rangeQueryBuilder);&nbsp;&nbsp;
</code></pre>


































<p data-nodeid="86129">这里的 startTB 和 endTB 已经经过格式化，与查询的 Index 中使用的时间格式对齐。示例中的 startTB 和 endTB 分别是 202001051910 和 202001051940。</p>
<p data-nodeid="96419" class="">2、精确匹配 Document 中的 entity_id 字段值，示例中 entity_id 字段分别为 2 和 3，&nbsp;相关代码片段如下：</p>
<pre class="lang-java" data-nodeid="96420"><code data-language="java">where.getKeyValues().forEach(keyValues -&gt; {
    <span class="hljs-keyword">if</span> (keyValues.getValues().size() &gt; <span class="hljs-number">1</span>) {  
        boolQuery.must().add(QueryBuilders.termsQuery(keyValues.getKey(), keyValues.getValues()));
    } <span class="hljs-keyword">else</span> {
        boolQuery.must().add(QueryBuilders.termQuery(keyValues.getKey(), keyValues.getValues().get(<span class="hljs-number">0</span>)));
    }
});
</code></pre>











<p data-nodeid="96946" class="">3、按照 entity_id 分组聚合查询到的 SLA 值（即 Document 中的 percentage 字段），具体聚合方式是计算平均值，相关片段如下：</p>
<pre class="lang-java" data-nodeid="96947"><code data-language="java">TermsAggregationBuilder&nbsp;entityIdAggregation&nbsp;=&nbsp;AggregationBuilders.terms(Metrics.ENTITY_ID).field(Metrics.ENTITY_ID).size(<span class="hljs-number">1000</span>);
parentAggBuilder.subAggregation(AggregationBuilders.avg(valueCName).field(valueCName));
</code></pre>















<p data-nodeid="97994" class="">4、将上述构造的查询条件和聚合函数构造成 SearchRequest 请求发送给 ElasticSearch 集群完成查询，相关片段如下：</p>
<pre class="lang-java" data-nodeid="98517"><code data-language="java">SearchSourceBuilder sourceBuilder = SearchSourceBuilder.searchSource();
sourceBuilder.query(boolQuery);
sourceBuilder.size(<span class="hljs-number">0</span>);
sourceBuilder.aggregation(entityIdAggregation);
SearchResponse response = getClient().search(indexName, sourceBuilder);
</code></pre>



















<p data-nodeid="99038">注意，这里查询的 indexName 是 Index 别名，在前面介绍 Index Template 的时候已经简单介绍了 Index alias 的作用，这里不再重复。</p>
<p data-nodeid="99039">5、解析 SearchResponse 得到查询结果，即示例中每个 Service 的 SLA 平均值，相关代码片段如下：</p>

<pre class="lang-java" data-nodeid="99562"><code data-language="java">IntValues intValues = <span class="hljs-keyword">new</span> IntValues();
Terms idTerms = response.getAggregations().get(Metrics.ENTITY_ID);
<span class="hljs-keyword">for</span> (Terms.Bucket idBucket : idTerms.getBuckets()) {
    KVInt kvInt = <span class="hljs-keyword">new</span> KVInt();
    <span class="hljs-comment">// key为 entity_id，即示例中的serviceId</span>
    kvInt.setId(idBucket.getKeyAsString()); 
    <span class="hljs-comment">// value为该 entity_id对应的 SLA平均值</span>
&nbsp;&nbsp;&nbsp;&nbsp;kvInt.setValue(idBucket.getAggregations().get(valueCName).getValue());
&nbsp;&nbsp;&nbsp;&nbsp;intValues.getValues().add(kvInt);&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;记录上述查询解析结果</span>
}
<span class="hljs-keyword">return</span> intValues;
</code></pre>

<p data-nodeid="54971">注意，上述执行过程只展示了针对 Avg 计算的相关代码，其他监控指标可能会用到其他聚合函数（例如：Sum、Max 等），就可能会走到其他代码分支，但核心逻辑类似，这里就不再重复展示了。</p>
<h4 data-nodeid="100083" class="">查询时序</h4>

<p data-nodeid="101117">下图是 demo-provider （serviceId = 3）响应时间的监控图，如前文所述，图中的时序数据是通过 getLinearIntValues() 方法查询得到的：</p>
<p data-nodeid="101118" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/3A/Ciqc1F7xuIyAZc4MAABhTIGGqHg410.png" alt="Drawing 8.png" data-nodeid="101122"></p>


<p data-nodeid="54975">下面将以该图为例，详细分析 getLinearIntValues()&nbsp;方法的查询流程：</p>
<p data-nodeid="54976">1、首先根据查询的起止时间以及 entity_id，确定要查询的 Document Id，具体实现如下：</p>
<pre class="lang-java" data-nodeid="101643"><code data-language="java"><span class="hljs-comment">// 按照 DownSampling单位以及查询时间范围，确定有多少个Document需要查询</span>
List&lt;DurationPoint&gt; durationPoints = DurationUtils.INSTANCE.getDurationPoints(downsampling, startTB, endTB);
List&lt;String&gt; ids = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
<span class="hljs-comment">// 构造每个 DurationPoint对应的 Document Id</span>
durationPoints.forEach(durationPoint&nbsp;-&gt;&nbsp;ids.add(durationPoint.getPoint()&nbsp;+&nbsp;Const.ID_SPLIT&nbsp;+&nbsp;id));
</code></pre>

<p data-nodeid="102680">示例中的 DownSampling 值为 Minute，查询的时间范围为 20:44~ 20:59，生成的 DurationPoint 以及 Document Id 如下图所示：</p>
<p data-nodeid="102681" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/45/CgqCHl7xuJmAapVFAAC9IVj0ets069.png" alt="Drawing 9.png" data-nodeid="102687"></p>


<p data-nodeid="54979">2、创建 SearchRequst 请求进行查询。</p>
<pre class="lang-java" data-nodeid="103208"><code data-language="java">SearchRequest searchRequest = <span class="hljs-keyword">new</span> SearchRequest(indexName);
searchRequest.types(TYPE);
<span class="hljs-comment">// 指定查询的 Document Id</span>
searchRequest.source().query(QueryBuilders.idsQuery().addIds(ids)).size(ids.length);
SearchResponse response = client.search(searchRequest);
<span class="hljs-comment">//&nbsp;将返回的&nbsp;SearchResponse转换成&nbsp;Map后返回，第一层&nbsp;Key是Document&nbsp;Id，第二层&nbsp;Key是&nbsp;Field名称，第二层&nbsp;Value是字段对应的&nbsp;Value值</span>
Map&lt;String, Map&lt;String, Object&gt;&gt; result = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
SearchHit[] hits = response.getHits().getHits();
<span class="hljs-keyword">for</span> (SearchHit hit : hits) {
    result.put(hit.getId(), hit.getSourceAsMap());
}
<span class="hljs-keyword">return</span> result;
</code></pre>

<p data-nodeid="104249">示例中会根据步骤 1 生成的 Document Id 精确查找 demo-webapp 的 service_resp_time 指标每分钟（20:44~ 20:59 范围）对应的 Document，如下图所示：</p>
<p data-nodeid="104250" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/3A/Ciqc1F7xuKiAfaUbAAr-NQ2X_L4640.png" alt="Drawing 10.png" data-nodeid="104260"></p>


<p data-nodeid="105293">3、将步骤 2 的查询结果整理成 IntValues（底层是 KVInt 列表），相关代码实现比较简单，不再展示。示例中的整理结果如下图所示，其中每个 KVInt 的 Key 为 Document Id，Value 为相应的 summation 值：</p>
<p data-nodeid="105294" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/45/CgqCHl7xuLKAFb6VAAwzofyYQew138.png" alt="Drawing 11.png" data-nodeid="105298"></p>


<p data-nodeid="54984">前端拿到上述  KVInt 列表之后，即可绘制出示例中的 Service Response Time 监控图。</p>
<h4 data-nodeid="105819" class="te-preview-highlight">查询 heatmap</h4>

<p data-nodeid="54986">MetricQuery 中最后一个查询方法是&nbsp;getThermodynamic() 方法，该方法用于查询热力图，具体查询方式与 getLinearIntValues() 方法类似，这里不再展开分析。</p>

---

### 精选评论


