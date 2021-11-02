<p data-nodeid="1349" class="">今天我要和你分享的内容是提供可观测性的另一个组件：Metrics 监控指标。</p>
<p data-nodeid="1350">在上一讲中，我们学习了可观测性组件之一：Trace，Trace 丰富了微服务的排障手段和可观测性。今天介绍的 Metrics 主要可以解决微服务的实时监控和告警问题。</p>
<p data-nodeid="1351">为了更好地理解 Metrics 监控指标的作用和不同之处，我们先来看一下传统的监控告警系统。</p>
<h3 data-nodeid="1352">传统监控系统</h3>
<p data-nodeid="1353">监控系统通常由指标采集子系统和数据处理子系统组成。指标采集子系统主要负责信息采集、过滤、汇总和存储；数据处理子系统主要负责数据分析、展现、预警和告警等。传统的监控系统一般有ELK 和 Nagios &amp; Zabbix，下面我们就来详细了解一下。</p>
<h4 data-nodeid="1354">ELK</h4>
<p data-nodeid="1355">ELK 是 Elastic 公司推出的监控系统，包含了 Elasticsearch、Logstash 和 Kibana，分别作为存储引擎、日志收集和前端展示系统。其中<strong data-nodeid="1454">Elasticsearch 是核心</strong>，其他两个都可以替换，较常见的替换方案是 Filebeat 和 Graylog。</p>
<p data-nodeid="1356">其中<strong data-nodeid="1464">Filebeat 是由 Go 语言编写的轻量级日志收集工具</strong>。日志可以直接传输到 Elasticsearch 中，可以传输到 Logstash 进行一些数据处理。而<strong data-nodeid="1465">Graylog 是一套代替 ELK 的日志收集分析系统</strong>，其中保留了 Elasticsearch 的存储部分，提供了 collector-sidecar 用于日志收集，并继承了 Web 图形界面，用于搜索和图形化展示。</p>
<p data-nodeid="1357">总体来说，ELK 是一套传统的可观测性系统，主要利用我们之前提到的可观测性组件之一——<strong data-nodeid="1475">日志来提供监控功能</strong>，优点是<strong data-nodeid="1476">数据准确性高</strong>，但由于日志需要存储和建立索引，成本会比较高。这套系统在互联网的中期使用非常多，因为对原有系统没有任何侵入性，也非常容易接入。</p>
<h4 data-nodeid="1358">Nagios &amp; Zabbix</h4>
<p data-nodeid="1359">Nagios 和 Zabbix 都是传统的运维监控工具，主要监控主机、网络，以及业务进程端口的信息和状态。主要<strong data-nodeid="1485">以插件的方式进行扩展</strong>，通过插件扩展可以检测主机的 CPU、内存、硬盘等信息，也可以检测各种不同的网络协议，但对于检测微服务应用程序内的信息几乎是无能为力的。</p>
<p data-nodeid="1360">随着微服务架构和云原生系统的发展，大家对应用程序内的可观测性提出了更高的要求，不仅仅是系统级别的监控，包括应用自身的黄金指标（<strong data-nodeid="1491">延时、通信量、饱和度、错误率</strong>）和一些业务级别的指标都需要实时的可观测。</p>
<h3 data-nodeid="1361">现代化常用的 Metrics 系统</h3>
<p data-nodeid="1362">Metrics 主要是用<strong data-nodeid="1502">时序性数据库</strong>记录每个时间点的监控数据，通过主动拉取或者程序上报的方式记录，然后实时计算一段时间的数据，并通过<strong data-nodeid="1503">图形界面</strong>的方式展现出来。它的特点是实时性强、可观测指标丰富，适合查看一段时间内的指标趋势。</p>
<h4 data-nodeid="1363">StatsD+Graphite</h4>
<p data-nodeid="1364">StatsD 是 Node.js 编写的一个<strong data-nodeid="1510">Metrics 指标采集系统</strong>，通过 UDP 协议将信息传输到后端程序，聚合后存储到 Graphite。</p>
<p data-nodeid="1365">StatsD 包含三个组成部分，分别是：</p>
<ul data-nodeid="1366">
<li data-nodeid="1367">
<p data-nodeid="1368">Client，各种语言使用的 SDK，用于将 Metrics 信息通过 UDP 的方式传送到 Server。</p>
</li>
<li data-nodeid="1369">
<p data-nodeid="1370">Server，收集客户端传输的 Metrics 信息，聚合后发送到 Backend，也就是 Graphite。</p>
</li>
<li data-nodeid="1371">
<p data-nodeid="1372">Backend，也就是上面提到的 Graphite，Graphite 是一个时序数据库，负责存储 Metrics 信息。</p>
</li>
</ul>
<h4 data-nodeid="1373">InfluxDB+Telegraf</h4>
<p data-nodeid="1374">InfluxDB 是一个时序数据库，负责 Metrics 信息的存储；Telegraf 是一套 Metrics 收集系统，默认自带非常丰富的插件，用于采集系统级别的信息，如果要采集应用维度的信息，则需要编写插件。</p>
<h4 data-nodeid="1375">Prometheus</h4>
<p data-nodeid="1376">Prometheus 是 Cloud Native Computing Foundation（简称：CNCF）生态圈的一员，也是整个 Kubernetes 生态中重要的一环，现已经广泛用于 Kubernetes 集群监控中。Prometheus 与上面的两个 Metrics 系统最大的不同，是<strong data-nodeid="1527">提供了一整套完整的监控告警解决方案</strong>，包含了<strong data-nodeid="1528">数据采集、数据存储和告警</strong>。</p>
<p data-nodeid="1377">另外还有两个非常重要的特性，也是现代化监控系统的必要标志：</p>
<ul data-nodeid="1378">
<li data-nodeid="1379">
<p data-nodeid="1380">采用了 pull 拉取的模型采集数据，简化了客户端的代码编写成本，并且 HTTP 协议非常利于调试。</p>
</li>
<li data-nodeid="1381">
<p data-nodeid="1382">支持服务发现，默认支持了 Consul 和 Kubernetes 原生发现系统，避免了繁杂的机器信息配置。</p>
</li>
</ul>
<p data-nodeid="1383">至此，我们介绍了常见的 Metrics 指标监控系统，下面针对现在最流行的 Prometheus 展开做详细的讲解。</p>
<h3 data-nodeid="1384">Prometheus 组成和架构</h3>
<p data-nodeid="1385">Prometheus 架构由以下模块构成。</p>
<ul data-nodeid="1386">
<li data-nodeid="1387">
<p data-nodeid="1388"><strong data-nodeid="1539">Prometheus Server</strong>：用于收集和存储 Metrics 数据。</p>
</li>
<li data-nodeid="1389">
<p data-nodeid="1390"><strong data-nodeid="1544">Service Disvovery</strong>：用于服务发现，获取服务对应的 EndPoint 用于数据采集，默认支持 DNS、Consul、Kubernestes 等多种发现方案。</p>
</li>
<li data-nodeid="1391">
<p data-nodeid="1392"><strong data-nodeid="1549">PushGateway</strong>：提供推送的方式采集数据，主要用于 Job 类，Job 类程序可能存活时间比较短，不适合采用拉取的方式。另外一些非常驻进程的脚本语言，比如 PHP，也需要使用此种方式。</p>
</li>
<li data-nodeid="1393">
<p data-nodeid="1394"><strong data-nodeid="1554">Exporters</strong>：用于一些第三方服务，通过转换提供符合 Prometheus 规范的 Metrics 信息，比如 Node Exporter 提供节点相关的信息。BlackBox 方便用户使用 HTTP、TCP 等方式对应用进程探活，以监控应用状态。</p>
</li>
<li data-nodeid="1395">
<p data-nodeid="1396"><strong data-nodeid="1559">Client Library</strong>：为各种语言提供的客户端库，提供 HTTP 服务的 Metrics 接口，当&nbsp;Prometheus Server 拉取时，提供实时的 Metrics 数据。</p>
</li>
<li data-nodeid="1397">
<p data-nodeid="1398"><strong data-nodeid="1564">Alertmanager</strong>：告警管理器，接收 Prometheus Server 发来的告警信息，去除重复信息，分组后发送给相应的人员。通知方式支持 Email 和 WebHook 自定义等，一般会通过 WebHook 接入钉钉或者企业微信以达到实时报警的目的。</p>
</li>
</ul>
<p data-nodeid="1399"><img src="https://s0.lgstatic.com/i/image/M00/90/4C/CgqCHmAKh7WAS3kyAAFtuGRBoi0895.png" alt="Drawing 0.png" data-nodeid="1567"></p>
<div data-nodeid="1400"><p style="text-align:center">Prometheus 架构图</p></div>
<h4 data-nodeid="1401">Prometheus 数据模型</h4>
<p data-nodeid="1402">下面，我们通过一条简单的 Metrics 数据，来了解它由哪些模块组成：</p>
<pre class="lang-java" data-nodeid="1403"><code data-language="java">negri_http_request_total{client=<span class="hljs-string">"serviceA"</span>,code=<span class="hljs-string">"200"</span>,exported_service=<span class="hljs-string">"serviceB"</span>,path=<span class="hljs-string">"/ping"</span>}
</code></pre>
<p data-nodeid="1404">Metrics 名字：首先是 Metrics 的名称，Metrics 的名称表明了这条数据的用途，比如上面这个数据是 Negri Sidecar 统计的总的请求数量。</p>
<p data-nodeid="1405">Label 标签：这条数据中的 client、code、exported_service 和 path 都是标签，通过标签可以组成不同的查询语句，从不同维度获取数据。</p>
<h4 data-nodeid="1406">Prometheus 的 Metrics 类型</h4>
<p data-nodeid="1407">Prometheus 由四种不同的 Metrics 类型组成，这些 Metrics 类型适用于不同的应用场景，下面我就结合应用场景来聊一聊这四种类型。</p>
<p data-nodeid="1408"><strong data-nodeid="1579">Counter</strong></p>
<p data-nodeid="1409">累加值，非常适合统计 QPS。这个数据从服务开始就不停地累加，比如上面提到的 negri_http_request_total 就是一种 Counter 类型。它<strong data-nodeid="1595">统计了服务启动至目前所有请求的数据，通过 rate 或者 irate 就可以计算出一段时间内的 QPS</strong>。Counter 类型是 Prometheus Server 端计算的，相对于下面讲到的 Gauge，<strong data-nodeid="1596">占用服务自身更少</strong>，建议高性能的微服务首选此种类型。</p>
<p data-nodeid="1410"><strong data-nodeid="1600">Gauge</strong></p>
<p data-nodeid="1411"><strong data-nodeid="1609">适合记录瞬时值</strong>，比如统计一些突发事件，例如是否产生了熔断、限流等事件。因为是客户端 SDK 计算的，不太适合一些经常变化的数据。<strong data-nodeid="1610">如果数据是一直增加的，建议使用 Counter</strong>；当然如果数据有增有减，也比较适合，因为监控中很少遇到增减比较频繁的数据。</p>
<pre class="lang-java" data-nodeid="1412"><code data-language="java">degrade_events{event=<span class="hljs-string">"eventBreakerOpenStatus"</span>,service=<span class="hljs-string">"serviceB"</span>}
</code></pre>
<p data-nodeid="1413"><strong data-nodeid="1614">Histogram</strong></p>
<p data-nodeid="1414">柱状图，适合统计服务的 99 延时等信息，比如下面的例子就是用于统计服务的延时状况：</p>
<pre class="lang-java" data-nodeid="1415"><code data-language="java">negri_http_response_time_us_bucket{client=<span class="hljs-string">"serviceA"</span>,exported_service=<span class="hljs-string">"serviceB"</span>,le=<span class="hljs-string">"0.5"</span>,path=<span class="hljs-string">"/ping"</span>}
</code></pre>
<p data-nodeid="1416">如上述内容，可以通过查询语句绘制出 90 延时、95 延时、99 延时等指标， Histogram 和 Counter 类型一样，都是 Prometheus Server 端计算的，所以非常<strong data-nodeid="1621">适合高性能的场景使用</strong>，相对于下面讲解的 Summary 有更小的损耗。</p>
<p data-nodeid="1417"><strong data-nodeid="1625">Summary</strong></p>
<p data-nodeid="1418">类似于 Histogram，适合统计服务的 99 延时信息。Summary 和 Gauge 类型一样，都是在程序内计算的，所以并不能像 Histogram 一样，在绘制图形的时候灵活的设置百分位，但相对来说，<strong data-nodeid="1631">Summary 的数据也更加精准</strong>。</p>
<p data-nodeid="1419">这里讲完了 Metrics 的常用数据结构，下面我们来看一下如果利用 Grafana 展示收集的 Metrics 信息。</p>
<h3 data-nodeid="1420">Grafana 图形化展示</h3>
<p data-nodeid="1421">Grafana 是一套可视化监控指标工具，主要用于时序数据的展示，包括 Graphite、Elasticsearch、InfluxDB、Prometheus、CloudWatch、MySQL 和 OpenTSDB。这些数据源大多数我们在前面做过介绍。其中最常见的就是 Prometheus+Grafana 的组合。</p>
<p data-nodeid="1422">通过 Grafana的 Web 图形界面，可以配置不同的面板，以多种方式展示数据源。</p>
<p data-nodeid="1423"><img src="https://s0.lgstatic.com/i/image/M00/90/4C/CgqCHmAKh9uAQAMSAAHVS5sXmD0847.png" alt="Drawing 1.png" data-nodeid="1638"></p>
<p data-nodeid="1424">通过 Grafana 展示服务一段时间内的运行状态，包括常见的 QPS、延时等信息。</p>
<p data-nodeid="1425"><img src="https://s0.lgstatic.com/i/image/M00/90/41/Ciqc1GAKh-KAR5LLAAMgpQx_lWg972.png" alt="Drawing 2.png" data-nodeid="1642"></p>
<h3 data-nodeid="1426">常见问题</h3>
<h4 data-nodeid="1427">Prometheus 适合什么样的监控场景？</h4>
<p data-nodeid="1428">Metrics 数据监控，并不适合要求数据 100% 准确的场景，更多的是反映一段时间内<strong data-nodeid="1654">某个数据指标的趋势和大概值</strong>。采集数据是存在间隔的，计算数据也是有时间区间的，所以很难反映某一时刻的准确值，比如 CPU 的峰值，大概率会被平均掉。<strong data-nodeid="1655">如果要查看精准数据，最好通过日志的方式收集后检索查看</strong>。</p>
<h4 data-nodeid="1429">单个服务节点 Metrics 数量限制</h4>
<p data-nodeid="1430">Prometheus 虽然性能强大，但如果无规范的使用，随着采集数据节点增多，依然难以保证其稳定性。比如一些 RESTful 规范的 path（比如 /test/123，其中 123 为变量），如果不采取一些措施聚合，会造成 Metrics 爆炸。<strong data-nodeid="1662">Metrics 的爆炸不仅会导致服务 OOM，也会导致 Prometheus 的内存使用量增大和检索变慢</strong>。</p>
<p data-nodeid="1431">另外过多的标签变量、过多的 Histogram bucket，都会导致 Metrics 数量的增加。比如<strong data-nodeid="1668">不合理地记录了调用端的来源 IP，同时也记录了服务自身的 IP，两者的 Metrics 数量就会变成乘积关系</strong>。这些不合理的指标不光会影响服务自身，也会影响 Prometheus 的稳定性，所以在输出 Metrics 信息的时候一定要注意，应该限制程序 Metrics 数量，加一个默认的上限条件。</p>
<h4 data-nodeid="1432">RESTful path 如何处理？</h4>
<p data-nodeid="1433">无论是在 Metrics 系统中，还是 Trace 系统中，都会遇到此类问题。RESTful 格式的 path 在某一阶段非常流行，特别是在 PC 互联网时代非常适合搜索引擎的 SEO，而且看起来美观，所以被广泛使用。</p>
<p data-nodeid="1434">如果说在 Web 客户端使用还有因可循，但在微服务架构的内网使用就完全不可理解了，毕竟内网无法被搜索引擎搜索到。所以<strong data-nodeid="1676">在微服务架构中，如果使用 HTTP 协议作为通信协议，建议抛弃 RESTful 的做法</strong>。</p>
<p data-nodeid="1435">当然对于残留的一些服务，我们也使用 Trie Tree 的方式做了数据聚合，当某一层 path 大于一定阈值的时候，就抛弃之前的记录，合并为一个节点存储。这其实是 HTTP Server 中路由组件的一种逆向操作，在路由组件中一般会采用 Trie Tree 做路由匹配。</p>
<h3 data-nodeid="1436">总结</h3>
<p data-nodeid="2363">这一讲我主要介绍了微服务中 Metrics 监控告警，通过今天的内容，相信你已经对服务的可观测性有了进一步的认识。在程序 Metrics 的输出打点中，我们要特别注意 Metrics 的数量，注意 Metrics 标签中的变量，如变量变化过多，会造成服务和 Prometheus 不稳定。</p>
<p data-nodeid="3729" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/91/3A/Ciqc1GAN5YmAb5kSAAL1ncn8r90570.png" alt="金句11.png" data-nodeid="3732"></p>






<p data-nodeid="1439">本讲内容到这里就结束了，下一讲我会讲解 Service Mesh 的技术选型，我们一起来看看如何在众多 Service Mesh 开源产品中选择适合你的产品。</p>
<p data-nodeid="1440" class="">结合这节内容的讲解，如果让你统计服务的基础信息，你会统计输出哪些信息呢？欢迎在留言区和我分享你的观点。我们下一讲再见！</p>

---

### 精选评论


