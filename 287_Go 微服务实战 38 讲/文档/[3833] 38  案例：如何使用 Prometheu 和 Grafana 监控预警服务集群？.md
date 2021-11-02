<p data-nodeid="86460" class="">你好，我是 aoho，今天我和你分享的是如何使用 Prometheus 和 Grafana 监控预警服务集群的案例。</p>
<p data-nodeid="86461"><strong data-nodeid="86571">监控和预警平台</strong>是互联网公司较为重要的后端架构组成之一，是整个运维乃至整个产品生命周期中最重要的一环，它能够事前及时预警发现故障，事后提供详实的数据用于追查定位问题。<strong data-nodeid="86572">Prometheus 和 Grafana 相结合是开源服务监控和预警平台的主流方案之一</strong>。</p>
<p data-nodeid="86462">监控和预警平台的重要性在《SRE：Google 运维解密》一书中就有体现：开发人员可以通过监控和预警平台了解服务内部的实际运行状态，通过对指标的观察可以预判所出现问题的可能原因，并且能够在系统发生故障时快速通知相关的人员。</p>
<p data-nodeid="86463">建立完善的监控体系，可以达到以下四个目的：</p>
<ul data-nodeid="86464">
<li data-nodeid="86465">
<p data-nodeid="86466"><strong data-nodeid="86579">数据可视化。</strong> 开发人员可以通过可视化仪表盘直观地了解系统的运行状态、资源使用情况和内部性能指标等信息。</p>
</li>
<li data-nodeid="86467">
<p data-nodeid="86468"><strong data-nodeid="86584">跟踪和分析程序的长期指标趋势。</strong> 监控系统通过对监控样本数据的长期收集和统计，来进行长期趋势分析，以及预判操作。例如，开发人员可以通过对磁盘空间占用增长率长期数据的分析，提前预测需要在哪个时间点对磁盘进行清理操作。</p>
</li>
<li data-nodeid="86469">
<p data-nodeid="86470"><strong data-nodeid="86589">故障和异常告警。</strong> 当系统将要或者已经出现故障或异常状态时，监控系统能迅速感知并通知开发人员，从而能够让开发人员对问题进行快速处理，避免出现对业务的严重影响。</p>
</li>
<li data-nodeid="86471">
<p data-nodeid="86472"><strong data-nodeid="86594">故障和异常分析与定位。</strong> 当故障或者异常发生后，开发人员可以通过当时的监控数据对其进行调查和处理，最终找到故障和异常产生的根源问题。</p>
</li>
</ul>
<p data-nodeid="87272">今天我们就以一个简单的 Go 服务为例，从 0 部署和搭建基于 Prometheus 和 Grafana 的监控和预警平台，通过钉钉机器人发送报警信息。</p>
<p data-nodeid="87273" class=""><img src="https://s0.lgstatic.com/i/image/M00/68/E8/CgqCHl-lBcSAT3LdAAMKsps4g0A385.png" alt="Drawing 1.png" data-nodeid="87278"></p>
<div data-nodeid="87274"><p style="text-align:center">监控和报警平台架构图</p></div>




<p data-nodeid="86476">如上图所示，Prometheus 从不同类型的 Exporter 上收集到业务服务或者中间件的数据，然后 Grafana 从 Prometheus 中获取数据并进行可视化展示，还会根据报警规则将报警信息通过邮件、钉钉或者短信发送给开发人员。</p>
<h4 data-nodeid="86477">部署和配置 Prometheus</h4>
<p data-nodeid="86478"><strong data-nodeid="86605">Prometheus 是一套开源的基于时间序列数据库的数据监控报警平台</strong>。2012 年，Prometheus 由 SoundCloud 公司着手开发，后续又被多家公司和组织接受和采用，最终将其独立为开源项目，并成立独立公司来运作。开源之后，Prometheus 项目及其社区一直非常活跃，并获得众多开发人员的支持和参与。Prometheus 最终于 2016 年加入了云计算基金会，成为继 Kubernetes 之后的第二个云计算基金会托管的开源项目。</p>
<p data-nodeid="86479">总之，Prometheus 是一款十分成功的开源服务监控预警系统，它具有以下优点：</p>
<ul data-nodeid="86480">
<li data-nodeid="86481">
<p data-nodeid="86482">多维数据模型，时序列数据由 metric 名和一组 key/value 组成，方便存储不同格式和模型的数据；</p>
</li>
<li data-nodeid="86483">
<p data-nodeid="86484">可以使用自建的 PromQL 在多维度上灵活地进行数据查询，方便数据检索和查询；</p>
</li>
<li data-nodeid="86485">
<p data-nodeid="86486">不依赖分布式存储，单主节点工作，方便自身部署和运维；</p>
</li>
<li data-nodeid="86487">
<p data-nodeid="86488">可以采用 Push 和 PushGateway 结合的方式进行数据收集，方便特殊网络状况下的数据收集；</p>
</li>
<li data-nodeid="86489">
<p data-nodeid="86490">可以通过服务发现或者静态配置来获取要进行数据采集的目标服务或机器，方便动态配置数据采集的目标；</p>
</li>
<li data-nodeid="86491">
<p data-nodeid="86492">支持多种可视化图表及表盘，方便数据可视化。</p>
</li>
</ul>
<p data-nodeid="86493">此外，Prometheus 采集数据时使用拉模型（Pull），通过 HTTP 协议调用 Exporter 的接口来采集数据指标，只要服务能够提供 HTTP 接口就可以接入 Prometheus 监控系统，相较于私有协议或二进制协议来说，接入成本低且开发简易，这无疑简化了应用系统纳入该系统的工作量，并降低了难度。</p>
<p data-nodeid="86494">首先，我们使用 Docker 系统来部署 Prometheus：</p>
<pre class="lang-java" data-nodeid="86495"><code data-language="java">sudo docker run -d -p <span class="hljs-number">9090</span>:<span class="hljs-number">9090</span> -v /prom_etc/prometheus.yml:/etc/prometheus/prometheus.yml -v /prom_etc/alert.rules:/etc/prometheus/alert.rules --name prometheus prom/prometheus --config.file=/etc/prometheus/prometheus.yml --web.enable-lifecycle
</code></pre>
<p data-nodeid="86496">启动时加上“--web.enable-lifecycle”会启用远程热加载配置文件功能，调用指令是 curl -X POST http://localhost:9090/-/reload。/prom_etc/prometheus.yml 文件是主要的配置文件，其具体配置和解释如下所示：</p>
<pre class="lang-dart" data-nodeid="86497"><code data-language="dart">global:
  scrape_interval: &nbsp; &nbsp; <span class="hljs-number">15</span>s # 抓取数据的间隔，默认为<span class="hljs-number">1</span>分钟
  evaluation_interval: <span class="hljs-number">15</span>s # 评估数据的间隔
  external_labels:
 &nbsp; &nbsp;  monitor: <span class="hljs-string">'codelab-monitor'</span>

# 数据抓取相关配置
scrape_configs:
 &nbsp;# prometheus 自身数据的抓取配置
  - job_name: <span class="hljs-string">'prometheus'</span>
 &nbsp;  static_configs:
 &nbsp; &nbsp;  - targets: [<span class="hljs-string">'localhost:9090'</span>]
  # Go 程序数据的抓取配置
  - job_name: go
 &nbsp;  scrape_interval: <span class="hljs-number">10</span>s  # 抓取间隔
 &nbsp;  metrics_path: /metrics  # 抓取数据的URL路径
 &nbsp;  static_configs: 
 &nbsp; &nbsp;  - targets: [<span class="hljs-string">'192.168.0.104:9100'</span>] # 抓取数据的IP地址
 &nbsp; &nbsp; &nbsp;  labels:
 &nbsp; &nbsp; &nbsp; &nbsp;  instance: go1
</code></pre>
<p data-nodeid="86498">scrape_configs 中配置的就是可以抓取数据的任务，比如对 Prometheus 自身数据的抓取、对 Go 程序数据的抓取、对 MySQL 数据的抓取和对 Redis 性能数据的抓取，等等。Prometheus 特意显示了诸多 exporter 来对这些程序或者中间件的数据抓取，例如：MySQL server exporter、Memcached exporter 和 JMX exporter 等。</p>
<p data-nodeid="86499">我们可以在 Prometheus 的 Targets 页面查看这些抓取任务的状态，具体如下图所示：</p>
<p data-nodeid="88666"><img src="https://s0.lgstatic.com/i/image/M00/68/DC/Ciqc1F-lBdOAGf0fAAO2GhMXnPI434.png" alt="Drawing 3.png" data-nodeid="88669"></p>

<div data-nodeid="88104"><p style="text-align:center">Prometheus 的 Targets 状态页面 1</p></div>




<p data-nodeid="86502">由上图可以看到，Prometheus 自身的数据抓取已经成功，但是对 Go 程序的抓取一直处于失败状态，所以，接下来我们需要部署 Go 服务程序，并提供相应的数据抓取网络接口。</p>
<h4 data-nodeid="86503">部署 Go 服务的 Exporter</h4>
<p data-nodeid="86504">我们可以直接使用 Prometheus 提供的 Go 语言 Client 来实现 Go 程序数据的 Exporter，只要引入 prometheus/client_golang 包，并建立网络服务即可，具体代码如下所示：</p>
<pre class="lang-java" data-nodeid="86505"><code data-language="java"><span class="hljs-keyword">import</span> (
  <span class="hljs-string">"net/http"</span>
  <span class="hljs-string">"github.com/prometheus/client_golang/prometheus/promhttp"</span>
)

<span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
  http.Handle(<span class="hljs-string">"/metrics"</span>, promhttp.Handler())
  http.ListenAndServe(<span class="hljs-string">":9100"</span>, nil) <span class="hljs-comment">// 监听 9100 端口，提供数据抓取服务</span>
}
</code></pre>
<p data-nodeid="86506">该程序运行起来后，再去 Prometheus 的 Targets 界面查看，就会发现，Prometheus 已经能够获取到 Go 服务进程相关的数据了。</p>
<p data-nodeid="90057"><img src="https://s0.lgstatic.com/i/image/M00/68/DC/Ciqc1F-lBemAV7huAASa6GYhyx8056.png" alt="Drawing 5.png" data-nodeid="90060"></p>

<div data-nodeid="89495"><p style="text-align:center">Prometheus 的 Targets 状态页面 2</p></div>




<p data-nodeid="86509">虽然 Prometheus 自带数据的可视化界面，但是图表格式较为基础，且只能临时查看，总之是不够好用，所以<strong data-nodeid="86639">业界一般使用 Grafana 作为监控数据的展示平台</strong>。</p>
<h4 data-nodeid="86510">部署和配置 Grafana</h4>
<p data-nodeid="86511"><strong data-nodeid="86645">Grafana 是一款专门的数据可视化工具</strong>。它有着非常漂亮的布局和图表展示，以及功能齐全的度量仪表盘和图形编辑器。除此之外，Grafana 还能够根据查询条件设置聚合规则，在对应的图表上进行展示，并且支持多个图表共同组建成一个看板。</p>
<p data-nodeid="86512">Grafana 的主要优点有：</p>
<ul data-nodeid="86513">
<li data-nodeid="86514">
<p data-nodeid="86515"><strong data-nodeid="86651">图表展示方式灵活</strong>。Grafana 能够支持多种可视化指标和看板，拥有丰富的仪表盘插件，比如热图、折线图和图表等多种展示方式。</p>
</li>
<li data-nodeid="86516">
<p data-nodeid="86517"><strong data-nodeid="86656">支持多种数据源</strong>。Grafana 支持多种数据源，可以将 Graphite、InfluxDB、OpenTSDB、Prometheus、Elasticsearch、CloudWatch 和 KairosDB 等系统作为数据源。</p>
</li>
<li data-nodeid="86518">
<p data-nodeid="86519"><strong data-nodeid="86661">强大的报警规则配置和通知功能</strong>。Grafana 可以配置相应指标的警报规则，并每隔固定时间计算判断是否触发规则，在数据触发规则时通过钉钉、邮件、短信等方式发送报警信息。</p>
</li>
<li data-nodeid="86520">
<p data-nodeid="86521"><strong data-nodeid="86666">支持多数据源混合展示</strong>。在同一图表或者看板中可以使用不同数据源的数据，每个查询都可以特殊指定数据源，甚至可以自定义数据源。</p>
</li>
</ul>
<p data-nodeid="86522">下面，我们就来部署和配置 Grafana 平台，并以 Prometheus 为数据源，建立对 Go 程序的数据展示面板。首先，还是使用 Docker 部署 Grafana 镜像：</p>
<pre class="lang-java" data-nodeid="86523"><code data-language="java">docker run -d -p <span class="hljs-number">3001</span>:<span class="hljs-number">3000</span> --name=grafana544 grafana/grafana
</code></pre>
<p data-nodeid="86524">然后登录网站<a href="http://127.0.0.1:3001/" data-nodeid="86671">http://127.0.0.1:3001/</a>，用户名和密码默认都是 admin，登录后要修改登录密码。</p>
<p data-nodeid="86525">接着我们配置 Grafana 的数据源，从列表中选择 Prometheus，然后 URL 一栏填写其服务器地址，Access 一栏选择 Server 项，再点击 Save &amp; Test 按钮来检查并保存，这样我们就将上文配置的 Prometheus 设置为 Grafana 的数据源了。</p>
<p data-nodeid="91448"><img src="https://s0.lgstatic.com/i/image/M00/68/E8/CgqCHl-lBfuAV4UxAAJAa595YgI567.png" alt="Drawing 7.png" data-nodeid="91451"></p>

<div data-nodeid="90886"><p style="text-align:center">Grafana 数据源配置页面</p></div>




<p data-nodeid="86528">最后，我们来建立 Go 程序数据的可视化看板，从界面左侧的加号点击，可以从 0 开始新建可视化看板，也可以导入其他人开源的可视化看板。</p>
<p data-nodeid="86529">新建看板时，只要在 Metrics 中输入自己想要监控的数据名称即可，比如我们想要监控 Go 程序中协程的数量，则输入 go_goroutines，然后上方图标就会显示出对应的数据，一个是 Prometheus 平台自身的协程数据，另外一个则是 Go 程序的协程数据。如果只想看 Go 程序的，则可以输入 go_goroutines{job="go"}，即可限定只显示 Go 程序的数据。</p>
<p data-nodeid="92839"><img src="https://s0.lgstatic.com/i/image/M00/68/DD/Ciqc1F-lBgyAPP-DAAMkHLk4ibY306.png" alt="Drawing 9.png" data-nodeid="92842"></p>

<div data-nodeid="92277"><p style="text-align:center">Grafana 看板配置页面</p></div>




<p data-nodeid="86532">新建看板虽然入门简单，但是想要建立起完备真实可用的看板也是要花费一定时间的，所以我们往往都是直接导入其他人开源的看板配置。那怎么导入呢？</p>
<p data-nodeid="86533">首先，我们在<a href="https://grafana.com/grafana/dashboards" data-nodeid="86696">https://grafana.com/grafana/dashboards</a>上搜索 Go 相关的看板配置，然后检查这个看板配置是否和我们所收集数据的 Exporter 相互吻合，确定之后，复制该看板详情页面的网络 URL。</p>
<p data-nodeid="93667" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/D6/CgqCHl-iWsCADYe0AAz3hA18tsc589.png" alt="Drawing 5.png" data-nodeid="93671"></p>
<div data-nodeid="93668"><p style="text-align:center">Grafana 网站页面</p></div>




<p data-nodeid="86536">将 URL 贴到 Grafana 的导入页面上，点击加载，就会加入一些看板名称、数据来源等配置，点击保存，我们就直接获得了 Go 程序的监控数据可视化看板界面。</p>
<p data-nodeid="95900"><img src="https://s0.lgstatic.com/i/image/M00/68/E8/CgqCHl-lBjGAScg7AANRWsSQuNI689.png" alt="Drawing 12.png" data-nodeid="95903"></p>

<div data-nodeid="95338"><p style="text-align:center">Grafana 看板导入界面</p></div>








<p data-nodeid="86539">从可视化看板中，我们可以清晰地看到 Go 程序相关的性能指标曲线图，比如 process memory、 go memstats、Goroutines 和 GC duration 等。</p>
<p data-nodeid="97291"><img src="https://s0.lgstatic.com/i/image/M00/68/DD/Ciqc1F-lBjyAY-kcAARXdD9UNp0756.png" alt="Drawing 14.png" data-nodeid="97294"></p>

<div data-nodeid="96729"><p style="text-align:center">Grafana 看板详情界面</p></div>




<p data-nodeid="86542">基于这些性能指标曲线图，我们就可以清楚地了解 Go 程序在过去的某一时刻某些指标是否异常，从而给线上 Go 程序异常分析工作提供巨大的帮助。</p>
<p data-nodeid="86543">除了协助异常诊断，Grafana 还可以根据性能指标进行报警，比如当 process memory 超过一定数值时，通过钉钉或者邮件向开发者发出报警信息，有助于开发者提前感知线上程序的异常状态，从而尽快处理，以减少线上程序异常情况发生的频率。</p>
<h4 data-nodeid="86544">配置报警规则和通道</h4>
<p data-nodeid="86545">接下来，我们就来基于 Grafana 配置对于 Go 程序的性能数据异常的报警渠道。</p>
<p data-nodeid="86546">首先，我们编辑想要报警指标的看板，切换到 Alert 一栏，在 Conditions 下面我们可以进行均值相关的配置，如下图所示，我们就做了这样的配置：如果 Go 程序线程数量在一分钟内的均值超过 8 则报警，对应的上方图标就有一条红线，表示警戒线。</p>
<p data-nodeid="99793"><img src="https://s0.lgstatic.com/i/image/M00/68/DD/Ciqc1F-lBkuAMUKgAAKPmQSuBFc967.png" alt="Drawing 16.png" data-nodeid="99796"></p>

<div data-nodeid="99231"><p style="text-align:center">Grafana 报警规则配置界面</p></div>








<p data-nodeid="86549">接着，我们需要配置报警渠道，可以选择钉钉或者邮件渠道。我们在需要发送报警消息的群中首先添加一个自定义机器人，安全设置就勾选“自定义关键词”选项，关键词就选择“报警”二字。</p>
<p data-nodeid="101184"><img src="https://s0.lgstatic.com/i/image/M00/68/E8/CgqCHl-lBliAe0zFAAFUeXbC5RE342.png" alt="Drawing 18.png" data-nodeid="101187"></p>

<div data-nodeid="100622"><p style="text-align:center">钉钉机器人配置界面</p></div>




<p data-nodeid="86552">添加后，我们可以获得该机器人的 Webhook 地址，然后进入 Grafana 的报警渠道配置页面：新建 DingDing 类型的报警渠道，将 Webhook 地址填入 Url 一栏，点击 Test 没问题后再点击 Save 进行保存。</p>
<p data-nodeid="102575"><img src="https://s0.lgstatic.com/i/image/M00/68/E8/CgqCHl-lBmmAK15XAAKG9lUP1nY683.png" alt="Drawing 20.png" data-nodeid="102578"></p>

<div data-nodeid="102013"><p style="text-align:center">Grafana 报警通道配置界面</p></div>




<p data-nodeid="86555">然后在配置报警规则时，报警渠道选择“钉钉报警渠道”，文本消息中必须包含”报警“二字，否则不能通过钉钉机器人的安全配置校验，从而导致无法发送报警信息，具体配置如下图所示：</p>
<p data-nodeid="103966"><img src="https://s0.lgstatic.com/i/image/M00/68/DD/Ciqc1F-lBnKAXfRbAAI4pLYImMY057.png" alt="Drawing 22.png" data-nodeid="103969"></p>

<div data-nodeid="103404"><p style="text-align:center">Grafana 报警规则配置页面</p></div>




<p data-nodeid="86558">综上，Grafana 每隔 1 分钟对 Go 服务的协程数量进行判断，如果其数值持续超过 8 到达 5 分钟，则通过钉钉发送报警消息。</p>
<h4 data-nodeid="86559">小结</h4>
<p data-nodeid="86560">本课时主要讲解了如何使用 Prometheus 和 Grafana 建立 Go 程序性能数据监控和报警平台。根据系统的需要，对业务服务、中间件或者数据库的数据基表进行收集和展示，方便开发人员查看和了解整体系统的动态和数据指标，协助排查线上异常问题和提前发现异常状态。</p>
<p data-nodeid="86561" class="">除了 Prometheus 和 Grafana，你还对什么其他的监控平台有所了解？欢你迎积极在下方留言，我们一起交流。</p>

---

### 精选评论


