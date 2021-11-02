<p>通过前面搭建 SkyWalking 的运行环境我们知道，SkyWalking OAP 后端可以使用多种存储对数据进行持久化，例如 MySQL、TiDB 等，默认使用 ElasticSearch 作为持久化存储，在后面的源码分析过程中也将以 ElasticSearch 作为主要存储进行分析。</p>
<h3>ElasticSearch 基本概念</h3>
<p>本课时将快速介绍一下 ElasticSearch 的基本概念，如果你没有用过 ElasticSearch ，可以通过本小节迅速了解 ElasticSearch 中涉及的基本概念。Elasticsearch 是一个基于 Apache Lucene 的开源搜索引擎，无论在开源还是专业领域，Apache Lucene 可以被认为是迄今为止最先进、性能最好、功能最全的搜索引擎库。但是，Apache Lucene 只是一个工具库，本身使用也比较复杂，直接使用 Apache Lucene 对开发人员的要求比较高，需要检索方面的知识来理解它是如何工作的。ElasticSearch 使用 Java 对 Apache Lucene 进行了封装，提供了简单易用的 RESTful API，隐藏 Apache Lucene 的复杂性，降低了全文检索的编程门槛。</p>
<p>ElasticSearch 中有几个比较核心的概念，为了方便你理解，我将其与数据库中的概念进行映射，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/03/81/Ciqah158kASATlQvAAApiOhG1PE953.png" alt=""></p>
<p>注意：在老版本的 ElasticSearch 中，Index 和 Document 之间还有个 Type 的概念，每个 Index 下可以建立多个 Type，Document 存储时需要指定 Index 和 Type。从 ES 6.0 版本开始单个 Index 中只能有一个 Type，ES 7.0 版本以后将不建议使用 Type，ES 8.0&nbsp;以后完全不支持 Type。</p>
<p>Document 是构建 Index 的基本单元。例如，一条订单数据就可以是一个 Document，其中可以包含多个 Field，例如，订单的创建时间、价格、明细，等等。Document 以 JSON 格式表示，Field 则是这条 JSON 数据中的字段，如下所示：</p>
<pre><code data-language="java" class="lang-java">{
&nbsp;&nbsp;<span class="hljs-string">"_index"</span>:&nbsp;<span class="hljs-string">"order_index"</span>,
&nbsp;&nbsp;<span class="hljs-string">"_type"</span>:&nbsp;<span class="hljs-string">"1"</span>,
&nbsp;&nbsp;<span class="hljs-string">"_id"</span>:&nbsp;<span class="hljs-string">"kg72dW8BOCMUWGurIFiy"</span>,
&nbsp;&nbsp;<span class="hljs-string">"_version"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;<span class="hljs-string">"_score"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;<span class="hljs-string">"_source"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"create_time"</span>:&nbsp;<span class="hljs-string">"2020-02-01&nbsp;17:35:00"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"creator"</span>:&nbsp;<span class="hljs-string">"xxxxx"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"order_status"</span>:&nbsp;<span class="hljs-string">"NEW"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"price"</span>:&nbsp;<span class="hljs-number">100.00</span>,
&nbsp;&nbsp;}
}
</code></pre>
<p>Index 是具有某些类似特征的 Document 的集合，Index 与 Document 之间的关系就类似于数据库中 Table 与 Row 之间的关系。在 Index 中可以存储任意数量的 Document。在后续介绍的示例中可以看到，对 Document 的添加、删除、更新、搜索等操作，都需要明确的指定 Index 名称。</p>
<p>最后，还需要了解 ElasticSearch 中一个叫作 Index Template（模板）的概念。Index Template 一般会包含 settings、mappings、index_patterns 、order、aliases 几部分:</p>
<ul>
<li>index_patterns 负责匹配 Index 名称，Index Template 只会应用到名称与之匹配的 Index 上，而且 ElasticSearch 只会在 Index 创建的时候应用匹配的 Index Template，后续修改 Index Template 时不会影响已有的 Index。通过 index_patterns 匹配可以让多个 Index 重用一个 Index Template。</li>
<li>settings 主要用于设置 Index 中的一些相关配置信息，如分片数、副本数、refresh 间隔等信息（后面会介绍分片数和副本数的概念）；</li>
<li>mappings 主要是一些说明信息，类似于定义该 Index 的 schema 信息，例如，指定每个 Field 字段的数据类型；&gt;</li>
<li>order 主要作用于在多个 Index Template 同时匹配到一个 Index 的情况，如果此时这些Index Template 中的配置出现不一致，则以 order 的最大值为准，order 默认值为 0。另外，创建 Index 的命令中如果自带了 settings 或 mappings 配置，则其优先级最高；</li>
<li>aliases 则是为匹配的 Index 创建别名。我们可以通过请求<a href="http://localhost:9200/_alias/*">&nbsp;http://localhost:9200/_alias/*</a>获取所有别名与 Index 之间的对应关系。</li>
</ul>
<p>下面是 SkyWalking 使用的 segment 模板，它会匹配所有 segment-* 索引，segment-yyyyMMdd 索引是用来存储 Trace 数据的：</p>
<pre><code data-language="java" class="lang-java">{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"segment"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"order"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"index_patterns"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"segment-*"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"settings"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"index"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"refresh_interval"</span>:&nbsp;<span class="hljs-string">"3s"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"number_of_shards"</span>:&nbsp;<span class="hljs-string">"2"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"number_of_replicas"</span>:&nbsp;<span class="hljs-string">"0"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;省略&nbsp;analysis字段设置</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"mappings"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"properties"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"segment_id"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"keyword"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"trace_id"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"keyword"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"service_id"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"integer"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;省略其他字段的设置</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"aliases"</span>:&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;为匹配的Index创建别名</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"segment"</span>:&nbsp;{}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<h3>节点角色</h3>
<p>一个 ElasticSearch 集群是由一个或多个节点组成，这些节点共同存储了集群中的所有数据，并且 ElasticSearch 提供了跨节点的联合索引和搜索功能。集群名称是一个 ElasticSearch 集群的唯一标识，在请求 ElasticSearch 集群时都需要使用到这个集群名称。在同一个网络环境中，需要保证集群名称不重复，否则集群中的节点可能会加入到错误的集群中。</p>
<p>ElasticSearch 集群是去中心化的，ElasticSearch 节点的相互发现是基于 Pull-Push 版本的 Gossip 算法实现的。Zen Discovery 是 ElasticSearch 默认的发现实现，提供了广播和单播的能力，帮助一个集群内的节点完成快速的相互发现。</p>
<p>ElasticSearch 集群中的节点有多个可选的角色，这些角色都是通过在节点的配置文件中配置的。</p>
<ul>
<li>Master Eligible Node （候选主节点）：可以被选举为 Master 的候选节点；</li>
<li>Master Node （主节点）：完成节点发现阶段之后，才会进入主节点选举阶段，为了防止在网络分区的场景下出现脑裂问题，一般采用 quorum 版本的 Bully 算法变体（本课时重点是帮助你快速了解 ElasticSearch 基础知识，不展开该算法的具体原理）。所以，主节点是从候选主节点中选举出来的，主要负责管理 ElasticSearch 集群，通过广播的机制与其他节点维持关系，负责集群中的 DDL 操作（创建/删除索引），管理其他节点上的分片；</li>
<li>Data Node（数据节点）：存放数据的节点，负责数据的增删改查；</li>
<li>Coordinating Node（协调节点）：每个节点都是一个潜在的协调节点，协调节点最大的作用就是响应客户端的请求，将各个分片里的数据汇总起来一并返回给客户端，因此 ElasticSearch 的节点需要有足够的 CPU 和内存资源去处理汇总操作；</li>
<li>Ingest Node（提取节点）：能执行预处理管道，不负责数据也不负责集群相关的事务。</li>
</ul>
<h3>分片&amp;副本</h3>
<p>在 ElasticSearch 中的一个 Index 可以存储海量的 Document，单台机器的磁盘大小是无法存储的，而且在进行数据检索的时候，单台机器也存在性能瓶颈，无法为海量数据提供高效的检索。</p>
<p>为了解决上述问题，ElasticSearch 将单个 Index 分割成多个分片，创建 Index 时，可以按照预估值指定任意数量的分片。虽然逻辑上每个分片都属于一个 Index，但是单个分片都是一个功能齐全且独立的 Index，一个分片可以被分配到集群中的任意节点上。</p>
<p>通过分片的功能，Index 就有了容量水平扩展的能力，运维人员可以通过添加节点的方式扩充整个集群的容量。在处理检索请求时，不同的分片由不同的 ElasticSearch 节点进行检索，可以实现并发操作，这样也就可以大大提高检索性能。</p>
<p>最后，某条 Document 数据具体存储在哪个分片，完全由 ElasticSearch 的分片机制决定。当写入一条 Document 的时候，ElasticSearch 会根据指定的 key （默认是 ElasticSearch 自动生成的 Id，用户也可以手动指定）决定其所在的分片编号，计算公式如下：</p>
<pre><code data-language="java" class="lang-java">分片编号&nbsp;=&nbsp;hash(key)&nbsp;%&nbsp;主分片数量
</code></pre>
<p>主分片的数量决定了 Document 所在的分片编号，所以在创建 Index 之后，主分片数量不能改变。</p>
<p>在进行搜索时，每个分片产生的部分查询结果，也是由 ElasticSearch 集群负责进行聚合的，整个过程对于 Client 来说是透明的，如同操作一个单节点 ElasticSearch 实例。</p>
<p>单台服务器在实际使用中可能会因为这样或那样的原因发生故障，例如意外断电、系统崩溃、磁盘寿命到期等，这些故障是无法预知的。当发生故障时，该节点负责的分片就无法对外提供服务了，此时需要有一定的容错机制，在发生故障时保证此分片可以继续对外提供服务。</p>
<p>ElasticSearch 提供的副本功能就可以很好的解决这一问题，在副本模式下，每个分片分为主分片和副本分片，下图中一个 Index 有两个分片，p0 和 p1 是两个主分片，r0 和 r1 则是相应的副本分片：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/7C/97/Cgq2xl58kASAIlnSAAA2g6WmR_U791.png" alt=""></p>
<p>副本带来了两个好处：一个是在主分片出现故障的时候，可以通过副本继续提供服务（所以，分片副本一般不与主分片分配到同一个节点上）；另一个就是查询操作可以在分片副本上执行，因此可以提升整个 ElasticSearch 查询性能。</p>
<h3>ElasticSearch 写入流程简介</h3>
<p>分片是 ElasticSearch 中最小的数据分配单位，即一个分片总是作为一个整体被分配到集群中的某个节点。继续深入分片的结构会发现，一个分片是由多个 Segment 构成的，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/03/81/Ciqah158kASAPMPyAABiTVfbZ1w517.png" alt=""></p>
<p>Segment 是最小的数据存储单元，ElasticSearch 每隔一段时间会产生一个新的 Segment，用于写入最新的数据。旧的 Segment 是不可改变的，只能用于数据查询，是无法继续向其中写入数据的。</p>
<p>在很多分布式系统中都能看到类似的设计，这种设计有下面几点好处：</p>
<ul>
<li>旧 Segment 不支持修改，那么在读操作的时候就不需要加锁，省去了锁本身以及竞争锁相关的开销；</li>
<li>只有最新的 Segment 支持写入，可以实现顺序写入的效果，增加写入性能；</li>
<li>只有最新的 Segment 支持写入，可以更好的利用文件系统的 Cache 进行缓存，提高写入和查询性能。</li>
</ul>
<p>介绍完分片内部的 Segment 结构之后，接下来简单介绍一下 ElasticSearch 集群处理一个写入请求的大致过程：</p>
<p>写入请求会首先发往协调节点（Coordinating Node），之前提到，协调节点可能是 Client 连接上的任意一个节点，协调节点根据 Document Id 找到对应的主分片所在的节点。</p>
<p>接下来，由主分片所在节点处理写入请求，先是写入 Transaction Log 【很多分布式系统都有 WAL （Write-ahead Log）的概念，可以防止数据丢失】，而后将数据写入内存中，默认情况下每隔一秒会同步到 FileSystem Cache 中，Cache 中的数据在后续查询中已经可以被查询了，默认情况下每隔 30s，会将 FileSystem cache 中的数据写入磁盘中，当然为了降低数据丢失的概率，可以将这个时间缩短，甚至设置成同步的形式，相应地，写入性能也会受到影响。</p>
<p>写入其他副本的方式与写入主分片的方式类似，不再重复。需要注意的是，这里可以设置三种副本写入策略：</p>
<ul>
<li>quorum：默认为 quorum 策略，即超过半数副本写入成功之后，相应写入请求即可返回给客户端；</li>
<li>one ：one 策略是只要成功写入一个副本，即可向客户端返回；</li>
<li>all：all 策略是要成功写入所有副本之后，才能向客户端返回。</li>
</ul>
<p>ElasticSearch 的删除操作只是逻辑删除， 在每个 Segment 中都会维护一个 .del 文件，删除操作会将相应 Document 在 .del 文件中标记为已删除，查询时依然可以查到，但是会在结果中将这些"已删除"的 Document 过滤掉。</p>
<p>由于旧 Segment 文件无法修改，ElasticSearch 是无法直接进行修改的，而是引入了版本的概念，它会将旧版本的 Document 在 .del 文件中标记为已删除，而将新版本的 Document 索引到最新的 Segment 中。</p>
<p>另外，随着数据的不断写入，将产生很多小 Segment 文件，ElasticSearch 会定期进行 Segment Merge，从而减少碎片文件，降低文件打开数，提升 I/O 性能。在 Merge 过程中可以同时根据 .del 文件，将被标记的 Document 真正删除，此时才是真正的物理删除。</p>
<h3>ElasticSearch 查询流程简介</h3>
<p>读操作分为两个阶段：查询阶段和聚合提取阶段。在查询阶段中，协调节点接受到读请求，并将请求分配到相应的分片上（如果没有特殊指定，请求可能落到主分片，也有可能落到副本分片，由协调节点的负载均衡算法来确定）。默认情况下，每个分片会创建一个固定大小的优先级队列（其中只包含 Document Id 以及 Score，并不包含 Document 的具体内容），并以 Score 进行排序，返回给协调节点。如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/7C/97/Cgq2xl58kASAVIIqAADyYyiGJl0221.png" alt=""></p>
<p>在聚合阶段中，协调节点会将拿到的全部优先级队列进行合并排序，然后再通过 Document ID 查询对应的 Document ，并将这些 Document 组装到队列里返回给客户端。</p>
<h3>High Level REST Client 入门</h3>
<p>ElasticSearch 提供了两种 Java Client，分别是 Low Level REST Client 和 High Level REST Client，两者底层都是通过 HTTP 接口与 ElasticSearch 进行交互的：</p>
<ul>
<li>Low Level REST Client&nbsp; 需要使用方自己完成请求的序列化以及响应的反序列化；</li>
<li>High Level REST Client 是基于 Low Level REST Client 实现的，调用方直接使用特定的请求/响应对象即可完成数据的读写，完全屏蔽了底层协议的细节，无需再关心底层的序列化问题。另外， High Level REST Client 提供的 API 都会有同步和异步( async 开头)两个版本，其中同步方法直接返回相应的 response 对象，异步方法需要添加相应的 Listener 来监听并处理返回结果。</li>
</ul>
<p>SkyWalking 中提供的 ElasticSearchClient 是对 High Level REST Client 的封装，本课时将简单介绍 High Level REST Client 的基本操作，你可以将本课时作为 High Level REST Client 的入门参考，更加完整的 API 使用可以参考 ElasticSearch &nbsp;<a href="https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.5/java-rest-high.html">官方文档</a>。</p>
<p>使用 High Level REST Client 的第一步就是初始化 RestHighLevelClient 对象，该过程底层会初始化线程池以及网络请求所需的资源，类似于 JDBC 中的 Connection 对象，相关 API 代码如下：</p>
<pre><code data-language="java" class="lang-java">RestHighLevelClient&nbsp;client&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;RestHighLevelClient(
&nbsp;&nbsp;&nbsp;RestClient.builder(&nbsp;<span class="hljs-comment">//&nbsp;指定&nbsp;ElasticSearch&nbsp;集群各个节点的地址和端口号</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;HttpHost(<span class="hljs-string">"localhost"</span>,&nbsp;<span class="hljs-number">9200</span>,&nbsp;<span class="hljs-string">"http"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;HttpHost(<span class="hljs-string">"localhost"</span>,&nbsp;<span class="hljs-number">9201</span>,&nbsp;<span class="hljs-string">"http"</span>)));
</code></pre>
<p>拿到 RestHighLevelClient 对象之后，我们就可以通过它发送 CreateIndexRequest 请求创建 Index，示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;创建&nbsp;CreateIndexRequest请求，该请求会创建一个名为"skywalking"的&nbsp;Index，</span>
<span class="hljs-comment">//&nbsp;注意，Index&nbsp;的名称必须是小写</span>
CreateIndexRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;CreateIndexRequest(<span class="hljs-string">"skywalking"</span>);
<span class="hljs-comment">//&nbsp;在&nbsp;CreateIndexRequest请求中设置&nbsp;Index的&nbsp;setting信息</span>
request.settings(Settings.builder()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.put(<span class="hljs-string">"index.number_of_shards"</span>,&nbsp;<span class="hljs-number">3</span>)&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置分片数量</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.put(<span class="hljs-string">"index.number_of_replicas"</span>,&nbsp;<span class="hljs-number">2</span>)&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置副本数量</span>
);
<span class="hljs-comment">//&nbsp;在&nbsp;CreateIndexRequest请求中设置&nbsp;Index的&nbsp;Mapping信息，新建的&nbsp;Index里有</span>
<span class="hljs-comment">//&nbsp;个user和message两个字段，都为text类型，还有一个&nbsp;age字段，为&nbsp;integer类型</span>
request.mapping(<span class="hljs-string">"type"</span>,&nbsp;<span class="hljs-string">"user"</span>,&nbsp;<span class="hljs-string">"type=text"</span>,&nbsp;<span class="hljs-string">"age"</span>,&nbsp;<span class="hljs-string">"type=integer"</span>,&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-string">"message"</span>,&nbsp;<span class="hljs-string">"type=text"</span>);
<span class="hljs-comment">//&nbsp;设置请求的超时时间</span>
request.timeout(TimeValue.timeValueSeconds(<span class="hljs-number">5</span>));
<span class="hljs-comment">//&nbsp;发送&nbsp;CreateIndex请求</span>
CreateIndexResponse&nbsp;response&nbsp;=&nbsp;client.indices().create(request);
<span class="hljs-comment">//&nbsp;这里关心&nbsp;CreateIndexResponse响应的&nbsp;isAcknowledged字段值</span>
<span class="hljs-comment">//&nbsp;该字段为&nbsp;true则表示&nbsp;ElasticSearch已处理该请求</span>
<span class="hljs-keyword">boolean</span>&nbsp;acknowledged&nbsp;=&nbsp;response.isAcknowledged();
Assert.assertTrue(acknowledged);
</code></pre>
<p>完成 Index 的创建之后，我们就可以通过 IndexRequest 请求向其中写入 Document 数据了，需要使用&nbsp;RestHighLevelClient 发送&nbsp;IndexRequest 实现 Document 的写入：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;创建&nbsp;IndexRequest请求，这里需要指定&nbsp;Index名称</span>
IndexRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;IndexRequest(<span class="hljs-string">"skywalking"</span>);&nbsp;
request.type(<span class="hljs-string">"type"</span>);
request.id(<span class="hljs-string">"1"</span>);&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Document&nbsp;Id，不指定的话，ElasticSearch&nbsp;为其自动分配</span>
<span class="hljs-comment">//&nbsp;Document的具体内容</span>
String&nbsp;jsonString&nbsp;=&nbsp;<span class="hljs-string">"{"</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\"user\":\"kim\","</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\"postDate\":\"2013-01-30\","</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\"age\":29,"</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\"message\":\"trying&nbsp;out&nbsp;Elasticsearch\""</span>&nbsp;+
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"}"</span>;
request.source(jsonString,&nbsp;XContentType.JSON);
<span class="hljs-comment">//&nbsp;发送&nbsp;IndexRequest请求，不抛异常，就是创建成了</span>
IndexResponse&nbsp;response&nbsp;=&nbsp;client.index(request);
System.out.println(response);
<span class="hljs-comment">//&nbsp;输出如下：</span>
<span class="hljs-comment">//&nbsp;IndexResponse[index=skywalking,type=type,id=1,version=1,</span>
<span class="hljs-comment">//&nbsp;result=created,seqNo=0,primaryTerm=1,shards=</span>
<span class="hljs-comment">//&nbsp;{"total":3,"successful":1,"failed":0}]</span>
<span class="hljs-comment">//&nbsp;-----------------------</span>
<span class="hljs-comment">//&nbsp;IndexResponse&nbsp;中包含写入的&nbsp;Index名称、Document&nbsp;Id，Document&nbsp;版本，创建的</span>
<span class="hljs-comment">//&nbsp;result等重要信息</span>
</code></pre>
<p>在明确知道 Document 的 Id 以及所属的 Index 时，我们可以通过 GetRequest 请求查询该 Document 内容，相关代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;创建&nbsp;GetRequest请求，这里指定&nbsp;Index、type以及&nbsp;Document&nbsp;Id，</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;在高版本中，type参数已经消失</span>
&nbsp;&nbsp;&nbsp;&nbsp;GetRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;GetRequest(<span class="hljs-string">"skywalking"</span>,&nbsp;<span class="hljs-string">"type"</span>,&nbsp;<span class="hljs-string">"1"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;发送&nbsp;GetRequest请求</span>
&nbsp;&nbsp;&nbsp;&nbsp;GetResponse&nbsp;response&nbsp;=&nbsp;client.get(request);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;从&nbsp;GetResponse响应中可以拿到相应的&nbsp;Document以及相关信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;index&nbsp;=&nbsp;response.getIndex();&nbsp;<span class="hljs-comment">//&nbsp;获取&nbsp;Index名称</span>
&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;id&nbsp;=&nbsp;response.getId();<span class="hljs-comment">//&nbsp;获取&nbsp;Document&nbsp;Id</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(response.isExists())&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;检查&nbsp;Document是否存在</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;version&nbsp;=&nbsp;response.getVersion();&nbsp;<span class="hljs-comment">//&nbsp;Document版本</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"version:"</span>&nbsp;+&nbsp;version);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取不同格式的&nbsp;Document内容</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;sourceAsString&nbsp;=&nbsp;response.getSourceAsString();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"sourceAsString:"</span>&nbsp;+&nbsp;sourceAsString);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Map&lt;String,&nbsp;Object&gt;&nbsp;sourceAsMap&nbsp;=&nbsp;response.getSourceAsMap();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"sourceAsMap:"</span>&nbsp;+&nbsp;sourceAsMap);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">byte</span>[]&nbsp;sourceAsBytes&nbsp;=&nbsp;response.getSourceAsBytes();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;按照字段进行遍历</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Map&lt;String,&nbsp;DocumentField&gt;&nbsp;fields&nbsp;=&nbsp;response.getFields();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(Map.Entry&lt;String,&nbsp;DocumentField&gt;&nbsp;entry&nbsp;:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fields.entrySet())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DocumentField&nbsp;documentField&nbsp;=&nbsp;entry.getValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;name&nbsp;=&nbsp;documentField.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Object&nbsp;value&nbsp;=&nbsp;documentField.getValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(name&nbsp;+&nbsp;<span class="hljs-string">":"</span>&nbsp;+&nbsp;value);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"Document&nbsp;Not&nbsp;Exist!"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(ElasticsearchParseException&nbsp;e)&nbsp;{&nbsp;<span class="hljs-comment">//&nbsp;Index不存在的异常</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(e.status()&nbsp;==&nbsp;RestStatus.NOT_FOUND)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.err.println(<span class="hljs-string">"Can&nbsp;find&nbsp;index"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
<span class="hljs-comment">//&nbsp;输出如下：</span>
<span class="hljs-comment">//&nbsp;version:1</span>
<span class="hljs-comment">//&nbsp;sourceAsString:{"user":"kim","postDate":"2013-01-30",</span>
<span class="hljs-comment">//&nbsp;"age":29,"message":"trying&nbsp;out&nbsp;Elasticsearch"}</span>
<span class="hljs-comment">//&nbsp;sourceAsMap:{postDate=2013-01-30,&nbsp;message=trying&nbsp;out&nbsp;Elasticsearch,&nbsp;</span>
<span class="hljs-comment">//&nbsp;user=kim,&nbsp;age=29}</span>
<span class="hljs-comment">//&nbsp;_seq_no:0</span>
<span class="hljs-comment">//&nbsp;_primary_term:1</span>
</code></pre>
<p>有时候，我们只想检测某个 Document 是否存在，并不想查询其具体内容，可以通过 exists()&nbsp;方法实现。该方法发送的还是 GetRequest 请求，但我们可以指明此次请求不查询 Document 的具体内容，在 Document 较大的时候可以节省带宽，具体使用方式如下：</p>
<pre><code data-language="java" class="lang-java">GetRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;GetRequest(<span class="hljs-string">"skywalking"</span>,&nbsp;<span class="hljs-string">"type"</span>,&nbsp;<span class="hljs-string">"1"</span>);
<span class="hljs-comment">//&nbsp;不查询&nbsp;"_source"字段以及其他字段</span>
request.fetchSourceContext(<span class="hljs-keyword">new</span>&nbsp;FetchSourceContext(<span class="hljs-keyword">false</span>));
request.storedFields(<span class="hljs-string">"_none_"</span>);
<span class="hljs-comment">//&nbsp;通过exists()方法发送&nbsp;GetRequest</span>
<span class="hljs-keyword">boolean</span>&nbsp;exists&nbsp;=&nbsp;client.exists(request);
Assert.assertTrue(exists);
</code></pre>
<p>删除一个 Document 的时候，使用的是 DeleteRequest，其中需要指定 Document Id 以及所属 Index 即可，使用方式如下：</p>
<pre><code data-language="java" class="lang-java">DeleteRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;DeleteRequest(<span class="hljs-string">"skywalking"</span>,&nbsp;<span class="hljs-string">"type"</span>,&nbsp;<span class="hljs-string">"1"</span>);
DeleteResponse&nbsp;response&nbsp;=&nbsp;client.delete(request);
System.out.println(response);
<span class="hljs-comment">//&nbsp;输出：</span>
<span class="hljs-comment">//&nbsp;DeleteResponse[index=skywalking,type=type,id=1,version=5,</span>
<span class="hljs-comment">//&nbsp;result=deleted,shards=ShardInfo{total=3,&nbsp;successful=1,</span>
<span class="hljs-comment">//&nbsp;failures=[]}]</span>
</code></pre>
<p>最后来看看更新操作，使用的是 UpdateRequest，示例如下：</p>
<pre><code data-language="java" class="lang-java">XContentBuilder&nbsp;builder&nbsp;=&nbsp;XContentFactory.jsonBuilder();
builder.startObject();
{
&nbsp;&nbsp;&nbsp;&nbsp;builder.field(<span class="hljs-string">"age"</span>,&nbsp;<span class="hljs-number">30</span>);
&nbsp;&nbsp;&nbsp;&nbsp;builder.field(<span class="hljs-string">"message"</span>,&nbsp;<span class="hljs-string">"update&nbsp;Test"</span>);
}
builder.endObject();
<span class="hljs-comment">//&nbsp;创建&nbsp;UpdateReques请求</span>
UpdateRequest&nbsp;request&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;UpdateRequest(<span class="hljs-string">"skywalking"</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>,<span class="hljs-string">"1"</span>).doc(builder);
<span class="hljs-comment">//&nbsp;发送请求</span>
UpdateResponse&nbsp;updateResponse&nbsp;=&nbsp;client.update(request);
System.out.println(updateResponse);
<span class="hljs-comment">//&nbsp;输出：</span>
<span class="hljs-comment">//&nbsp;UpdateResponse[index=skywalking,type=type,id=1,version=2,seqNo=1,</span>
<span class="hljs-comment">//&nbsp;primaryTerm=1,result=updated,shards=ShardInfo{total=3,&nbsp;</span>
<span class="hljs-comment">//&nbsp;successful=1,&nbsp;failures=[]}]</span>
<span class="hljs-comment">//&nbsp;注意，更新之后&nbsp;Document&nbsp;version&nbsp;会增加</span>
</code></pre>
<p>除了通过 GetRequest 请求根据 Document Id 进行查询之外，我们还可以通过 SearchRequest 请求进行检索，在检索请求里面我们可以通过 QueryBuilder 构造具体的检索条件，下面是一个简单的示例：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;创建&nbsp;SearchRequest请求</span>
SearchRequest&nbsp;searchRequest&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;SearchRequest(<span class="hljs-string">"skywalking"</span>);
<span class="hljs-comment">//&nbsp;通过&nbsp;SearchSourceBuilder,用来构造检索条件</span>
SearchSourceBuilder&nbsp;searchSourceBuilder&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;SearchSourceBuilder();
<span class="hljs-comment">//&nbsp;通过id进行检索，这里查询&nbsp;Id为1和2的两个Document</span>
searchSourceBuilder.query(QueryBuilders.idsQuery().addIds(<span class="hljs-string">"1"</span>,&nbsp;<span class="hljs-string">"2"</span>));
searchRequest.source(searchSourceBuilder);
<span class="hljs-comment">//&nbsp;发送&nbsp;SearchRequest&nbsp;请求</span>
SearchResponse&nbsp;searchResponse&nbsp;=&nbsp;client.search(searchRequest);
<span class="hljs-comment">//&nbsp;遍历&nbsp;SearchHit</span>
SearchHit[]&nbsp;searchHits&nbsp;=&nbsp;searchResponse.getHits().getHits();
<span class="hljs-keyword">for</span>&nbsp;(SearchHit&nbsp;searchHit&nbsp;:&nbsp;searchHits)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(searchHit.getId()&nbsp;+&nbsp;<span class="hljs-string">":"</span>&nbsp;+&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;searchHit.getSourceAsMap());
}
<span class="hljs-comment">//&nbsp;输出：</span>
<span class="hljs-comment">//&nbsp;1:{postDate=2013-01-30,&nbsp;message=update&nbsp;Test,&nbsp;user=kim,&nbsp;age=31}</span>
<span class="hljs-comment">//&nbsp;2:{postDate=2020-01-30,&nbsp;message=Test&nbsp;Message,&nbsp;user=Tom,&nbsp;age=51}</span>
</code></pre>
<p>按照&nbsp;Document Id 进行查询之外，ElasticSearch 还提供了一套 Query DSL 语言帮助我们构造复杂的查询条件，查询条件主要分为下面两部分：</p>
<ul>
<li><strong>Leaf 子句</strong>：Leaf 子句一般是针对某个字段的查询，例如：
<ul>
<li><strong>Match Query</strong>：最常用的 Full Text Query 。Match Query 既能处理全文字段，又能处理精确字段（可参考：<a href="https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-match-query.html">Match Query 的官方文档</a>）。</li>
<li><strong>Term Query</strong>：精确匹配字段值的查询（可参考：<a href="https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-term-query.html">Term Query 的官方文档</a>。</li>
<li><strong>Range Query</strong>：针对一个字段的范围查询（可参考：<a href="https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-range-query.html">Range Query 的官方文档</a>。</li>
</ul>
</li>
<li><strong>Compound 子句</strong>：Conpound 子句是由一个或多个 Leaf 子句或 Compound 子句构成的，例如&nbsp;Bool Query，包含多个返回 Boolean 值的子句（可参考：<a href="https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-bool-query.html">Bool Query 的官方文档</a>）。</li>
</ul>
<p>下面使用 ElasticSearch Query DSL 实现一个复杂查询：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;创建&nbsp;SearchRequest请求，查询的Index为skywalking</span>
SearchRequest&nbsp;searchRequest&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;SearchRequest(<span class="hljs-string">"skywalking"</span>);
<span class="hljs-comment">//&nbsp;通过&nbsp;SearchSourceBuilder,用来构造检索条件</span>
SearchSourceBuilder&nbsp;sourceBuilder&nbsp;=&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SearchSourceBuilder.searchSource();
<span class="hljs-comment">//&nbsp;创建&nbsp;BoolQueryBuilder</span>
BoolQueryBuilder&nbsp;boolQueryBuilder&nbsp;=&nbsp;QueryBuilders.boolQuery();
<span class="hljs-comment">//&nbsp;符合条件的&nbsp;Document中&nbsp;age字段的值必须位于[10,40]这个范围</span>
List&lt;QueryBuilder&gt;&nbsp;mustQueryList&nbsp;=&nbsp;boolQueryBuilder.must();
mustQueryList.add(QueryBuilders.rangeQuery(<span class="hljs-string">"age"</span>).gte(<span class="hljs-number">10</span>));
mustQueryList.add(QueryBuilders.rangeQuery(<span class="hljs-string">"age"</span>).lte(<span class="hljs-number">40</span>));
<span class="hljs-comment">//&nbsp;符合条件的&nbsp;Document中&nbsp;user字段的值必须为kim</span>
boolQueryBuilder.must().add(
&nbsp;&nbsp;&nbsp;&nbsp;QueryBuilders.termQuery(<span class="hljs-string">"user"</span>,&nbsp;<span class="hljs-string">"kim"</span>));
sourceBuilder.query(boolQueryBuilder);
searchRequest.source(sourceBuilder);
<span class="hljs-comment">//&nbsp;发送&nbsp;SearchRequest&nbsp;请求</span>
SearchResponse&nbsp;searchResponse&nbsp;=&nbsp;client.search(searchRequest);
SearchHit[]&nbsp;searchHits&nbsp;=&nbsp;searchResponse.getHits().getHits();
<span class="hljs-keyword">for</span>&nbsp;(SearchHit&nbsp;searchHit&nbsp;:&nbsp;searchHits)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(searchHit.getId()&nbsp;+&nbsp;<span class="hljs-string">":"</span>&nbsp;+&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;searchHit.getSourceAsMap());
}
<span class="hljs-comment">//&nbsp;输出:</span>
<span class="hljs-comment">//&nbsp;1:{postDate=2013-01-30,&nbsp;message=update&nbsp;Test,&nbsp;user=kim,&nbsp;age=31}</span>
</code></pre>
<p>示例中匹配的 Document 中 age 字段的值位于[10,40]这个范围中，且 user 字段值为 "kim"，类似于 SQL 语句中的【age &gt;=10 and &nbsp;age &lt;=40 and user == "kim"】语句。</p>
<p>High Level REST Client 还提供了批量操作的 API ——&nbsp;BulkProcessor 会将多个请求积攒成一个 bulk，然后批量发给 ElasticSearch 集群进行处理。使用&nbsp;BulkProcessor&nbsp;批量操作可以减少请求发送次数，提高请求消息的有效负载，降低 ElasticSearch 集群压力。</p>
<p>BulkProcessor 中有几个核心参数需要设置。</p>
<ul>
<li>setBulkActions() 方法：设置每个 BulkRequest &nbsp;包含的请求数量，默认值为 1000；</li>
<li>setBulkSize()：设置每个 BulkRequest 的大小，默认值为 5MB；</li>
<li>setFlushInterval()：设置两次发送 BulkRequest 执行的时间间隔。没有默认值；</li>
<li>setConcurrentRequests()：设置并发请求数。默认是 1，表示允许执行 1 个并发请求，积攒 BulkRequest 和发送 bulk 是异步的，其数值表示发送 bulk 的并发线程数（取值可以是任意大于 0 的数字），若设置为 0 表示二者同步；</li>
<li>setBackoffPolicy()：设置最大重试次数和重试周期。默认最大重试次数为 8 次，初始延迟是 50 ms。</li>
</ul>
<p>下面来看创建&nbsp;BulkProcessor&nbsp;的具体流程：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">//&nbsp;该&nbsp;Listener可以监听每个&nbsp;BulkRequest请求相关事件</span>
BulkProcessor.Listener&nbsp;listener&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;BulkProcessor.Listener()&nbsp;{...}
<span class="hljs-comment">//&nbsp;创建&nbsp;BulkProcessor</span>
BulkProcessor&nbsp;bulkProcessor&nbsp;=&nbsp;BulkProcessor.builder(client::bulkAsync,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;listener).setBulkActions(bulkActions)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setBulkSize(<span class="hljs-keyword">new</span>&nbsp;ByteSizeValue(bulkSize,&nbsp;ByteSizeUnit.MB))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setFlushInterval(TimeValue.timeValueSeconds(flushInterval))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setConcurrentRequests(concurrentRequests)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.setBackoffPolicy(BackoffPolicy.exponentialBackoff(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TimeValue.timeValueMillis(<span class="hljs-number">100</span>),&nbsp;<span class="hljs-number">3</span>)).build();
</code></pre>
<p>这里添加的 BulkProcessor.Listener 实现如下，其中提供了三个方法分别监听 BulkRequest 请求触发的不同事件：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">new</span>&nbsp;BulkProcessor.Listener()&nbsp;{
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">beforeBulk</span><span class="hljs-params">(<span class="hljs-keyword">long</span>&nbsp;executionId,&nbsp;BulkRequest&nbsp;request)</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;在&nbsp;BulkRequest请求发送之前，会触发该方法</span>
&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">afterBulk</span><span class="hljs-params">(<span class="hljs-keyword">long</span>&nbsp;executionId,&nbsp;BulkRequest&nbsp;request,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;BulkResponse&nbsp;response)</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;在收到&nbsp;BulkResponse响应时触发该方法，这里可以通过&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;response.hasFailures()方法判断请求是否失败</span>
&nbsp;&nbsp;&nbsp;}&nbsp;

&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">afterBulk</span><span class="hljs-params">(<span class="hljs-keyword">long</span>&nbsp;executionId,&nbsp;BulkRequest&nbsp;request,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Throwable&nbsp;failure)</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;在&nbsp;BulkRequest请求抛出异常的时候，会触发该方法</span>
&nbsp;&nbsp;&nbsp;}&nbsp;
}
</code></pre>
<p>之后我们就可以通过 BulkProcessor.add() 方法向其中添加请求，这些请求最终会积攒成一个 BulkRequest 请求发出：</p>
<pre><code data-language="java" class="lang-java">bulkProcessor.add(<span class="hljs-keyword">new</span>&nbsp;IndexRequest(...));
bulkProcessor.add(<span class="hljs-keyword">new</span>&nbsp;DeleteRequest(...));
</code></pre>
<p>一般情况下，我们只需要在全局维护一个 BulkProcessor 对象即可，在应用程序关闭之前需要通过 awaitClose() 方法等待全部请求发送出去后再关闭 BulkProcessor 对象。</p>
<h3>总结</h3>
<p>本课时首先介绍了 ElasticSearch 中的核心概念，为了便于理解，将这些概念与数据库进行了对比。接下来介绍了 ElasticSearch 集群中各个节点的角色，以及 ElasticSearch 引入分片和副本功能要解决的问题。之后对 ElasticSearch 读写数据的核心流程进行概述。最后，通过示例的方式介绍了 ElasticSearch&nbsp;High Level REST Client&nbsp;的基本使用方式。在后续课时讲解 SkyWalking OAP 实现时，你会再次看到 High Level REST Client&nbsp;的身影。</p>

---

### 精选评论

##### **武：
> 课程的源码呢？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; https://github.com/xxxlxy2008/skywalking-demo

