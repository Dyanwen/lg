<p data-nodeid="1917" class="">在上一课时，我们介绍了分布式链路追踪组件的相关背景和概念。市面上有多款流行的分布式链路追踪组件，包括 Zipkin、Jaeger、SkyWalking 和 Pinpoint 等，那它们具体的工作特性是怎样的呢？在实际工作中，我们又应该如何选型呢？</p>
<p data-nodeid="1918">下面我们就首先介绍下分布式链路追踪中的 OpenTracing 规范，然后再分析下这几款组件的相关特性，以及选型时的对比指标。</p>
<h3 data-nodeid="1919">分布式链路追踪规范：OpenTracing</h3>
<p data-nodeid="1920">Tracing 是在 20 世纪 90 年代就已出现的技术，虽然提出的时间很早，但真正让该领域流行起来的还是源于 Google 关于 Dapper 组件的一篇论文。分布式链路追踪组件经过了多年的发展，特别是微服务架构出现之后，各大公司也加大了对这块儿的投入和研发，但是对于链路追踪组件来说，其核心步骤有：<strong data-nodeid="2147">代码埋点、数据存储</strong>和<strong data-nodeid="2148">查询展示</strong>。链路追踪组件的组成，如下图所示：</p>
<p data-nodeid="4299"><img src="https://s0.lgstatic.com/i/image/M00/62/8C/CgqCHl-SjWKARLMtAACUh0mRaIk002.png" alt="图片1.png" data-nodeid="4302"></p>

<div data-nodeid="3339"><p style="text-align:center">链路追踪组件的组成</p></div>




<p data-nodeid="1923">目前流行的链路追踪组件有<strong data-nodeid="2161">Jaeger、Zipkin、SkyWalking</strong>和<strong data-nodeid="2162">Pinpoint</strong>。但是，在数据采集过程中，对用户代码的入侵和不同系统 API 的兼容性，导致切换链路追踪系统需要巨大的成本。</p>
<p data-nodeid="1924">OpenTracing 的诞生主要是为了解决不同的分布式链路追踪平台的 API 兼容问题。OpenTracing 通过提供与平台和厂商无关 API 的方式，使得开发人员能够很方便地切换追踪系统。</p>
<p data-nodeid="1925">那到底什么是 OpenTracing 呢？根据官方的介绍，OpenTracing 是一个轻量级的标准化层，位于应用程序/类库和追踪或日志分析程序之间。它提供了多种语言库的支持，包括 Go、Java、Python、Objective-C 和 C++ 等，通过引入这些通信标准库，我们就可以将追踪的信息发送到指定的组件。</p>
<p data-nodeid="1926">OpenTracing 的架构如下图所示，从图中我们可以看到 OpenTracing 支持 Zipkin、LightStep 和 Appdash 等追踪组件，并且可以轻松集成到开源的框架中，例如 gRPC、Flask、Django 和 Go-kit 等。</p>
<p data-nodeid="1927"><img src="https://s0.lgstatic.com/i/image/M00/5E/80/CgqCHl-Gvg-ACKv9AAHli59gxME260.png" alt="Drawing 1.png" data-nodeid="2168"></p>
<div data-nodeid="1928"><p style="text-align:center">OpenTracing 架构</p></div>
<p data-nodeid="1929">OpenTracing 是一个 Library 库，定义了一套通用的数据上报接口，要求各个分布式追踪系统都来实现这套接口。这样一来，应用程序只需要对接 OpenTracing，而无须关心后端采用的到底是什么样的分布式追踪系统，因此<strong data-nodeid="2173">开发者可以无缝切换分布式追踪系统，也使得在通用代码库增加对分布式追踪的支持成为可能。</strong></p>
<p data-nodeid="1930">OpenTracing 于 2016 年 10 月加入 CNCF 基金会，是继 Kubernetes 和 Prometheus 之后，第三个加入 CNCF 的开源项目。它是<strong data-nodeid="2179">一个中立的（厂商无关、平台无关）分布式追踪的 API 规范</strong>，提供统一接口，可方便开发者在自己的服务中集成一种或多种分布式追踪的实现。</p>
<h3 data-nodeid="1931">流行的分布式链路追踪组件</h3>
<p data-nodeid="1932">在熟悉了分布式链路追踪中的一些基础概念之后，下面我们就来具体了解一下这几种流行的分布式链路追踪组件。</p>
<h4 data-nodeid="1933">1. 简单易上手的 Zipkin</h4>
<p data-nodeid="1934">Zipkin 是一款由 Twitter 开源的分布式链路追踪组件，同样也兼容 OpenTracing API：基于 Google Dapper 论文而设计，很多公司都在用，文档资料也很丰富。其架构如下图所示：</p>
<p data-nodeid="1935"><img src="https://s0.lgstatic.com/i/image/M00/5E/80/CgqCHl-Gvh6AVPPFAABoGNmX9as934.png" alt="Drawing 2.png" data-nodeid="2188"></p>
<div data-nodeid="1936"><p style="text-align:center">Zipkin 架构图</p></div>
<p data-nodeid="1937">从 Zipkin 架构图可知，Zipkin 包含 Collector、Storage、API 和 Web UI 这 4 个部分。</p>
<ul data-nodeid="1938">
<li data-nodeid="1939">
<p data-nodeid="1940">Collector：存储和索引报上来的链路数据，以供后续查找。</p>
</li>
<li data-nodeid="1941">
<p data-nodeid="1942">Storage：Zipkin 的服务端存储，除了 Cassandra，Zipkin 还原生支持 ElasticSearch 和 MySQL。</p>
</li>
<li data-nodeid="1943">
<p data-nodeid="1944">Zipkin Query Service（API）：Zipkin 为 Web UI 提供的一个简单查询 API，用于查找和检索链路调用信息。</p>
</li>
<li data-nodeid="1945">
<p data-nodeid="1946">Web UI：Zipkin 查询链路追踪的界面。Web UI 提供了一种基于服务、时间和注解查看 Trace 记录的方法。</p>
</li>
</ul>
<p data-nodeid="1947"><strong data-nodeid="2202">Zipkin 分布式链路监控的优势是：语言无关性，整体实现较为简单</strong>。Zipkin 支持 Java、PHP、Go 和 Node.js 等语言客户端。社区支持的插件较为丰富，包括 RabbitMQ、MySQL 和 HTTPClient 等（具体参见<a href="https://github.com/openzipkin/brave/tree/master/instrumentation%EF%BC%89" data-nodeid="2200">https://github.com/openzipkin/brave/tree/master/instrumentation</a>）。因为 Zipkin UI 界面功能较为简单，本身无告警功能，所以对于有丰富的统计维度以及报警等功能的需求开发者来说，可能需要二次开发。</p>
<h4 data-nodeid="1948">2. 云原生链路监控组件 Jaeger</h4>
<p data-nodeid="1949">Jaeger 是 CNCF 云原生项目之一，受 Dapper 和 OpenZipkin 的启发，是由 Uber 开源的分布式追踪系统，兼容 Open Tracing API。Jaeger 主要用于微服务的监控和请求追踪，支持分布式上下文传播、请求报错分析、服务的调用网络分析以及性能/延迟优化。</p>
<p data-nodeid="1950">Jaeger 的服务端使用 Go 语言实现，其存储支持 Cassandra、Elasticsearch 和内存，并提供了 Go、Java、Node、Python 和 C++ 等语言的客户端库。Jaeger 具有如下的特性：</p>
<ul data-nodeid="1951">
<li data-nodeid="1952">
<p data-nodeid="1953"><strong data-nodeid="2212">高扩展性</strong>。Jaeger 后端的分布式设计，可以根据业务需求进行扩展。例如，Uber 任意一个 Jaeger 每天通常要处理数十亿个跨度。</p>
</li>
<li data-nodeid="1954">
<p data-nodeid="1955"><strong data-nodeid="2221">原生支持 OpenTracing</strong>。Jaeger 后端、Web UI 和工具库的设计支持 <a href="https://opentracing.io/specification/" data-nodeid="2219">OpenTracing 标准</a>。其特性包括：①通过跨度引用将轨迹表示为有向无环图（不仅是树）；②支持强类型的跨度标签和结构化日志；③通过baggage支持通用的分布式上下文传播机制。</p>
</li>
<li data-nodeid="1956">
<p data-nodeid="1957"><strong data-nodeid="2226">支持多个存储后端</strong>。Jaeger 支持 Cassandra 3.4+ 和 Elasticsearch 5.x/6.x 这两种流行的开源 NoSQL 数据库作为跟踪存储后端。</p>
</li>
<li data-nodeid="1958">
<p data-nodeid="1959"><strong data-nodeid="2231">现代化 Web UI</strong>。Jaeger Web UI 是使用流行的开源框架实现的。v1.0 中发布了几项性能改进，以允许 UI 有效处理大量数据，并能够显示上万跨度的链路跟踪。</p>
</li>
<li data-nodeid="1960">
<p data-nodeid="1961"><strong data-nodeid="2236">支持云原生部署</strong>。Jaeger 后端支持 Docker 镜像部署，很容易部署到 Kubernetes 集群。</p>
</li>
<li data-nodeid="1962">
<p data-nodeid="1963"><strong data-nodeid="2241">可观察性</strong>。默认情况下，所有 Jaeger 后端组件均开放 Prometheus 监控（也支持其他指标后端），并支持使用结构化日志库 zap 将日志标准输出。</p>
</li>
<li data-nodeid="1964">
<p data-nodeid="1965"><strong data-nodeid="2246">与 Zipkin 向后兼容</strong>。如果我们要从 Zipkin 库切换到 Jaeger，客户端不必重写所有代码。Jaeger 通过在 HTTP 上接受 Zipkin 格式（Thrift 或 JSON v1 / v2）的跨度来提供与 Zipkin 的向后兼容性，因此从 Zipkin 后端切换到 Jaeger 后端就变得很简单。</p>
</li>
</ul>
<p data-nodeid="1966">Jaeger 的架构如下图所示：</p>
<p data-nodeid="1967"><img src="https://s0.lgstatic.com/i/image/M00/5E/80/CgqCHl-GvjaAElw5AAEfQ3MA-FU057.png" alt="Drawing 3.png" data-nodeid="2250"></p>
<div data-nodeid="1968"><p style="text-align:center">Jaeger 架构图</p></div>
<p data-nodeid="1969">我们来分析一下 Jaeger 的架构图，Jaeger 主要由 client、agent、collector、DB 和 query 这几部分组成。</p>
<ul data-nodeid="1970">
<li data-nodeid="1971">
<p data-nodeid="1972">jaeger-client，即 Jaeger 客户端，各个语言的应用程序通过客户端 API 写入数据，并将链路追踪的数据发送给 jaeger-agent。</p>
</li>
<li data-nodeid="1973">
<p data-nodeid="1974">jaeger-agent，Jaeger 代理服务，每个物理机都会部署 agent，是一个守护进程，用于将数据发送给 Jaeger collector。jaeger-agent 是 client 和 collector 之间的桥梁，负责将二者解耦。</p>
</li>
<li data-nodeid="1975">
<p data-nodeid="1976">jaeger-collector，负责将客户端数据写入后端存储。</p>
</li>
<li data-nodeid="1977">
<p data-nodeid="1978">Data Store，服务端存储，目前支持 Cassandra 和 ElasticSearch 两种方式的数据存储。</p>
</li>
<li data-nodeid="1979">
<p data-nodeid="1980">jaeger-query，用于检索链路调用信息，通过 UI 进行展示。</p>
</li>
</ul>
<p data-nodeid="1981">下图为 Jaeger UI 中的统计视图，我们可以点击进去查看请求的链路调用详情。</p>
<p data-nodeid="1982"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/Ciqc1F-Gvj6AI-5SAAHxKyQQHhk929.png" alt="Drawing 4.png" data-nodeid="2260"></p>
<div data-nodeid="1983"><p style="text-align:center">Jaeger 链路监控页面</p></div>
<p data-nodeid="1984">该列表展示了请求的追踪记录，包括每次请求的时间、涉及的服务名和 Span 数量。通过统计的散列图，我们可以很清楚地看到请求的响应时间分布。<strong data-nodeid="2265">相比于 Zipkin，Jaeger 在界面上较为丰富，但是也无告警功能。</strong></p>
<h4 data-nodeid="1985">3. SkyWalking</h4>
<p data-nodeid="1986">SkyWalking 是一个国产的 APM 开源组件，具有监控、跟踪和诊断云原生架构中分布式系统的功能。SkyWalking 支持多个来源以及多种格式收集 Trace 和 Metric 数据，包括：</p>
<ul data-nodeid="1987">
<li data-nodeid="1988">
<p data-nodeid="1989">Java、.NET Core、Node.js 和 PHP 语言自动织入的 SkyWalking 格式；</p>
</li>
<li data-nodeid="1990">
<p data-nodeid="1991">手动织入的 Go 客户端 SkyWalking 格式；</p>
</li>
<li data-nodeid="1992">
<p data-nodeid="1993">Istio 追踪的格式；</p>
</li>
<li data-nodeid="1994">
<p data-nodeid="1995">Zipkin v1/v2 格式；</p>
</li>
<li data-nodeid="1996">
<p data-nodeid="1997">Jaeger gRPC 格式。</p>
</li>
</ul>
<p data-nodeid="1998">SkyWalking 的客户端通过 HTTP 或 gRPC 方式向 SkyWalking 收集器发送链路调用数据。SkyWalking 收集器对接收到的链路信息进行分析和聚合，并存储到服务端数据库。SkyWalking UI 提供了链路调用信息的可视化和检索。</p>
<p data-nodeid="1999">除了接收来自 SkyWalking Agent 的数据外，SkyWalking 还支持从 Zipkin v1/v2、Istio 、Envoy 等多个来源和多种格式收集数据。</p>
<p data-nodeid="2000">SkyWalking 整体架构的模块较多，但是结构比较清晰，主要就是通过收集各种格式的数据进行存储，然后展示。如下图为 SkyWalking 6.x 的架构图：</p>
<p data-nodeid="2001"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/Ciqc1F-GvlOARTWZAANEwHNPMrQ811.png" alt="Drawing 5.png" data-nodeid="2280"></p>
<div data-nodeid="2002"><p style="text-align:center">SkyWalking 6.x 架构图</p></div>
<p data-nodeid="2003">SkyWalking 支持的存储组件有：ES、H2、MySQL、TiDB 和 Sharding Sphere。SkyWalking 的 UI 界面提供的链路追踪查询较为简单，但 SkyWalking 拥有非常活跃的中文社区，支持多种语言的探针，且对国产开源软件全面支持。<strong data-nodeid="2286">SkyWalking 在探针性能方面表现优异，并且探针的性能损耗较低</strong>。</p>
<h4 data-nodeid="2004">4. 链路统计详细的 Pinpoint</h4>
<p data-nodeid="2005">Pinpoint 是一个 APM 工具，适用于 Java 、PHP 编写的大型分布式系统进行链路追踪。也就是说，<strong data-nodeid="2295">Go 语言项目不能直接应用 Pinpoint，需要使用代理进行改造</strong>。这里进行简单介绍，因为其链路追踪的分析较为完善。Pinpoint 也是受 Dapper 的启发，通过追踪分布式应用程序之间的调用链，帮助分析分布式系统的整体结构，以及服务和组件相互之间的依赖关系，如下图所示：</p>
<p data-nodeid="2006"><img src="https://s0.lgstatic.com/i/image/M00/5E/75/Ciqc1F-GvmSAYLmbAAI33y2CSDk491.png" alt="Drawing 6.png" data-nodeid="2298"></p>
<div data-nodeid="2007"><p style="text-align:center">Pinpoint 链路监控页面</p></div>
<p data-nodeid="2008"><strong data-nodeid="2307">Pinpoint 的追踪数据粒度非常细，用户界面功能强大，并且服务调用展示做得非常丰富</strong>，在这方面它优于市面上大多数组件。Pinpoint 使用 HBase 作为存储带来了海量存储的能力，但丰富的数据背后，必然需要大量的数据采集，因此在这几款常用链路追踪组件中，<strong data-nodeid="2308">Pinpoint 的探针性能最低</strong>，在生产环境需要注意应用服务的采样率，过高的话就会影响系统的吞吐量。</p>
<p data-nodeid="2009">另外，Pinpoint 目前仅支持 Java 和 PHP 语言，采用字节码增强方式去埋点，所以在埋点时不需要修改业务代码，是非侵入式的，非常适合项目已经完成之后再增加调用链监控的实践场景。Pinpoint 并不支持除 Java、PHP 语言之外的探针，在 Go 语言项目中应用需要基于 Pinpoint 进行二次封装开发。</p>
<h4 data-nodeid="2010">5. 组件的指标对比</h4>
<p data-nodeid="5890">上面我们对 4 种当前流行的链路追踪组件进行了简单介绍，相信你已经对每个组件的组成和特性已经有了大概的了解。为便于你更好地理解，下面我们再根据如下的几个指标对它们进行直观的对比。</p>
<p data-nodeid="5891" class=""><img src="https://s0.lgstatic.com/i/image/M00/62/81/Ciqc1F-SjXKAKnGuAAGS0Nd0F1o697.png" alt="图片2.png" data-nodeid="6003"></p>





<p data-nodeid="2121" class="">从这张对比表可以看出：</p>
<ul data-nodeid="2122">
<li data-nodeid="2123">
<p data-nodeid="2124">Zipkin 和 Jaeger 在各个方面都差不多（Jaeger 是在 Zipkin 的基础上改进了 Web UI 和传输协议等，且支持更多的客户端语言）。</p>
</li>
<li data-nodeid="2125">
<p data-nodeid="2126">相对前面两种组件来说，SkyWalking 功能较为齐全，探针性能损耗低，同时也支持多种语言的客户端。</p>
</li>
<li data-nodeid="2127">
<p data-nodeid="2128">Pinpoint 在 Web UI 的丰富性上完胜其他三种，然而其不支持 Go 语言客户端，实际应用需要进行改造；除此之外，它性能和可扩展性方面的不足也值得我们在选型时考虑和权衡。</p>
</li>
</ul>
<p data-nodeid="2129">总之，每种组件都有其优势和劣势，我建议你在链路追踪组件的选型时，要根据自身业务系统的实际情况，多想想哪些不能妥协、哪些可以舍弃，从而选择一款最适合的组件。</p>
<p data-nodeid="2130">当然，除了通过修改应用程序代码增加分布式追踪之外，还有一种不需要修改代码的非入侵方式，那就是<strong data-nodeid="2384">Service Mesh</strong>。Service Mesh 在网络层面拦截，通过 Sidecar（以额外的容器来扩展或增强主容器）的方式为各个微服务增加一层代理，通过这层网络代理来实现一些服务治理的功能，因为工作在网络层面，所以可以做到跨语言、非入侵。</p>
<h3 data-nodeid="2131">小结</h3>
<p data-nodeid="2132">本课时我们主要介绍了分布式链路追踪的 OpenTracing 规范，以及几种常见的分布式链路追踪组件选型。OpenTracing 规范的意义在于，它使得我们可以自由切换遵循 OpenTracing 规范的链路追踪组件，而目前主流的链路追踪组件也都是遵循 OpenTracing 规范的。</p>
<p data-nodeid="2133">在 Go 语言中，可以通过修改应用程序代码的方式增加分布式追踪，这种方式有一定的侵入性，但也是目前使用最多的分布式链路追踪方式。下一课时将进入实践环节，我们会通过一个案例演示如何应用 Zipkin 来追踪微服务请求的细节。</p>
<p data-nodeid="2134" class="">那学完本课时后，关于分布式链路追踪的选型，你觉得哪一款分布式链路追踪组件适合你的业务场景呢？为什么呢？欢迎你在留言区和我分享。</p>

---

### 精选评论

##### **龙：
> 个人更看好Jaeger，但是这些也都没有显示SQL，elastic的链路追踪是可以显示的，这点比较好

##### **8500：
> 为什么评论的同学这么少支持一下，老司机业务熟练、课程提纲挈领

