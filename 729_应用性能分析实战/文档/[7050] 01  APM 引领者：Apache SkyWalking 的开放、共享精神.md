<p data-nodeid="1025" class="">你好，欢迎你来到专栏第一课。这一讲我将带领你学习 Apache SkyWalking，那为什么我会以 Apache SkyWalking 作为第一讲呢？</p>
<p data-nodeid="2830" class="te-preview-highlight">首先 Apache SkyWalking 在国际上是非常受欢迎的 <a href="https://github.com/topics/apm?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2834">APM</a> 系统，在 APM 的核心领域（如<a href="https://github.com/topics/distributed-tracing?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2838">全链路追踪</a>、<a href="https://github.com/topics/web-performance?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2842">网站性能</a>）都非常具有前瞻性。</p>


<p data-nodeid="1027">它采用字节码增强技术，极大地降低了接入成本；支持 Java、.Net 等多语言探针，并提供 Dubbo、gRPC 等多种开源框架插件；诞生于社区，没有企业束缚和语言障碍。其开放、共享精神让其成为当之无愧的“APM 引领者”。</p>
<p data-nodeid="1028">其次我也是 Apache SkyWalking PMC，对该项目有非常丰富的经验，从 2018 年就开始使用 Apache SkyWalking 的孵化版本，在贝壳找房进行大规模的企业级实践；同时我也亲力亲为，见证了社区的关键里程碑，比如在 SkyWalking 成为 Apache 的顶级项目后，我参与编写《Apache SkyWalking 实战》图书等。</p>
<p data-nodeid="1029">接下来，我将通过 Apache SkyWalking 的发展历程和企业级的实践经验，来讲述 APM 引领者 SkyWaling 是如何进行大规模落地的。</p>
<h3 data-nodeid="1030">Apache SkyWalking 发展历程</h3>
<p data-nodeid="1031">SkyWalking 项目从 2017 年 12 月进入 Apache 孵化器，到现在的 8.X 版本已经迭代了 4 个大版本。</p>
<p data-nodeid="1032">下面我带你回顾下这 4 个大版本，让你了解在过去三年，Apache SkyWalking 在哪些 APM 领域都进行了哪些核心功能上的建设，进而总结下 Apache SkyWalking 现在与未来，建设的重点是什么。</p>
<p data-nodeid="1033">相信你在学习完这一讲后，会对“为什么有人对项目提出的建议或设计，会引起社区的强烈共鸣，而有些建议则无人问津”有了更高维度的认知；进而对 Apache SkyWalking 的 APM 落地进程中，会遇到的问题及其解决方法有更清楚的了解。</p>
<blockquote data-nodeid="1034">
<p data-nodeid="1035">比如，什么问题可以得到社区帮助，什么问题需要自己解决。</p>
</blockquote>
<p data-nodeid="1036">这里我先对下面内容使用的版本号，进行简要说明。版本后缀为什么是“.X”？其实软件的发布版本号分为三个部分，即“X.Y.Z”：</p>
<ul data-nodeid="1037">
<li data-nodeid="1038">
<p data-nodeid="1039">“X”代表主要版本，此版本号的升级代表软件有重大特性发布，且特性很可能不向前兼容；</p>
</li>
<li data-nodeid="1040">
<p data-nodeid="1041">“Y”代表次要版本，这个版本通常不提供重要的功能特性，而会对主要版本带来的重大特性，进行迭代升级；</p>
</li>
<li data-nodeid="1042">
<p data-nodeid="1043">“Z”代表修复版本，用于 Bug 修复。</p>
</li>
</ul>
<p data-nodeid="1044">下面只对 SkyWalking 的主要版本和次要版本进行介绍。</p>
<h4 data-nodeid="1045">5.X 版本</h4>
<p data-nodeid="1046">2017 年 12 月，SkyWalking 通过 Apache 软件基金会投票，其 SkyWalking 和 SkyWalking-UI 两个项目也正式进入 Apache 进行孵化，此时项目的贡献基本来自初始 10 人构成的 Committer 团队，目标是毕业成为 Apache 顶级项目。</p>
<ul data-nodeid="1047">
<li data-nodeid="1048">
<p data-nodeid="1049">SkyWalking 项目是通过 Java Agent 可对 Java 语言的服务进行无侵入的数据收集，以及数据的分析和持久化，核心数据为基于 OpenTracing 实现的分布式链路跟踪数据。</p>
</li>
<li data-nodeid="1050">
<p data-nodeid="1051">SkyWalking-UI 项目实现了 SkyWalking APM 系统的 UI 展示。</p>
</li>
</ul>
<p data-nodeid="1052">接下来 2018 年初，SkyWalking 社区孵化出 .NET Core Agent 和 NodeJS Server Agent。同时 SkyWalking 社区与 Zipkin 社区合作，实现了收集端可收集 Zipkin 数据，进行分析、持久化，以及在 SkyWalking APM 系统的 UI 进行展示。</p>
<h4 data-nodeid="1053">6.X 版本</h4>
<p data-nodeid="1054">6.X 版本是 SkyWalking 在 2018 年下半年开始发行的版本，此版本持续了一年多，是目前 SkyWalking 次要版本迭代最多，对社区意义最重大的版本。它见证了 SkyWalking 项目运作走向规范、社区愈发成熟的进程，并顺利毕业成为 Apache 顶级项目。</p>
<p data-nodeid="1055">6.X 版本的建设重点是 OAP（Observability Analysis Platform）思想的落地。</p>
<ul data-nodeid="1056">
<li data-nodeid="1057">
<p data-nodeid="1058">在数据协议上，SkyWalking 支持了指标数据（Metric data）、分布式链路跟踪数据（Tracing data）和日志数据（Logging data）。</p>
</li>
<li data-nodeid="1059">
<p data-nodeid="1060">在 APM 平台易用性上，SkyWalking 屏蔽了语言壁垒。通过实现可观测分析脚本语言，让用户通过自定义脚本的方式替代原始的修改代码，然后编译项目方式，就可以快速落地自己想要的可观察分析平台了。</p>
</li>
<li data-nodeid="1061">
<p data-nodeid="1062">在探针语言的丰富性上，SkyWalking 支持了 PHP 语言。在社区建设上，项目架构内核更加轻量化，扩展实现更加模块化、插件化。同时主库 Java 探针和后端分析存储项目也支持了 E2E 测试。</p>
</li>
</ul>
<h4 data-nodeid="1063">7.X 版本</h4>
<p data-nodeid="1064">7.X 版本为 SkyWalking 在 2020 年初发行的版本。</p>
<ul data-nodeid="1065">
<li data-nodeid="1066">
<p data-nodeid="1067">在数据协议上，SkyWalking 支持了收集性能剖析类数据，并落地了 Java 语言在线代码行级维度的性能剖析；支持用户通过 SkyWalking 进行自定义数据的传播和分析。</p>
</li>
<li data-nodeid="1068">
<p data-nodeid="1069">在探针语言丰富性上，SkyWalking 支持了 GO 和 Nginx。</p>
</li>
<li data-nodeid="1070">
<p data-nodeid="1071">在社区建设上，更多的探针语言支持了 E2E 测试。</p>
</li>
</ul>
<h4 data-nodeid="1072">8.X 版本</h4>
<p data-nodeid="1073">8.X 版本为 SkyWalking 在 2020年 中至今发行的版本。</p>
<ul data-nodeid="1074">
<li data-nodeid="1075">
<p data-nodeid="1076">在数据协议上，SkyWalking 移除了注册相关机制，这极大地减少了接入服务的性能损耗。支持了应用日志数据收集。</p>
</li>
<li data-nodeid="1077">
<p data-nodeid="1078">在日志功能上，支持了 Group 维度的核心概念，并在 APM 应用拓扑领域率先落地。</p>
</li>
<li data-nodeid="1079">
<p data-nodeid="1080">在探针语言丰富性上，SkyWalking 支持了前端和 Python，同时也在更好地拥抱云原生。</p>
</li>
</ul>
<p data-nodeid="1081">我将上述内容整理为下表可供你参考：</p>
<p data-nodeid="1082"><img src="https://s0.lgstatic.com/i/image6/M01/27/BF/Cgp9HWBdpeaAYqHMAAImIKpX6kY556.png" alt="Drawing 0.png" data-nodeid="1185"></p>
<p data-nodeid="1083">回顾 SkyWalking 过去三年的发展历史，可以看出 SkyWalking 每次大版本迭代，都是使用通用的设计方案实践 APM 领域的前沿技术。以 8.0 版本 SkyWalking 发布新的数据协议为例，在官方发版前，社区内的各个语言探针，就以极快的速度完成了新数据协议升级。云厂商 AlibabaCloud 也计划发布新版本，来支持 8.0 未发布的新协议。</p>
<p data-nodeid="1084">试想，SkyWalking 在没有企业背书的情况下，能够快速地调集社区资源完成大版本的迭代，也印证了其设计的技术方案有多通用易懂。<strong data-nodeid="1192">当然不断实践 APM 领域前沿技术只是 SkyWalking 的左手在做的事情，其右手则是一直践行着“社区大于一切”的思想，着重面向贡献者友好进行建设</strong>。</p>
<p data-nodeid="1085">以社区建设的 E2E 测试为例，因为要对某个应用框架进行性能监控，必须要对这个框架足够熟悉才能设计出很好的无侵入埋点。即使 SkyWalking 目前已拥有近 400 名贡献者，也不能对浩如烟海的应用框架兼容得完美无缺。所以通过 E2E 测试的建设，让贡献者向社区证明自己的代码是可用的，并且能兼容所有的 E2E 测试用例，那社区就会接受这次贡献。</p>
<p data-nodeid="1086">SkyWalking 的左、右手在做的事情我都介绍了，你肯定会发现在企业级落地，还缺少很多关键的功能。所以接下来，<strong data-nodeid="1199">我将挑选我在贝壳找房使用 SkyWalking 大规模落地过程中，遇到的非常有代表性的问题与你分享</strong>，希望能抛砖引玉，让你之后在遇到类似问题时可以茅塞顿开。</p>
<h3 data-nodeid="1087">企业级落地</h3>
<p data-nodeid="1088">由于 APM 工具需要 RD 逐渐熟悉上手后，才能带来提效，所以短期很难看到显著收益。但是<strong data-nodeid="1206">如果早期落地方案不好，短期看到的都是 APM 工具带来的问题，对 APM 日后的推广非常不利</strong>。</p>
<p data-nodeid="1089">所以我们在落地之初，我们就需要针对性地解决这些后顾之忧。接下来，我会以落地时，如何快速止损；落地中，如何解决水土不服；落地后，如何解决信息泄露的过程，阐述我的企业级落地实践。</p>
<h4 data-nodeid="1090">1.探针不稳定，如何及时止损并修复？</h4>
<p data-nodeid="1091">如果没有足够的知识储备和必要的定位问题的方法论，那么你在大规模应用部署探针时便会遇到各种各样的问题。相对于编写业务使用的 SSM 框架技术，编写 APM 探针所使用的字节码增强技术就相对冷门，可系统学习的资料较少。</p>
<p data-nodeid="1092">所以在业务项目接入探针出现问题，并得不到及时的根因分析和解决时，接入方甚至在主导 APM 落地的你，就会觉得：探针是不是与生俱来就很不稳定？</p>
<p data-nodeid="1093">我也有过同样的经历，知识的储备需要时间，定位问题的方法论需要实战经验，<strong data-nodeid="1215">所以我的解决思路是：实现秒级别的故障恢复和问题现场保留，然后通过持续的实践，最终找到适合自己团队的方法论。</strong></p>
<p data-nodeid="1094">针对探针不稳定这一特性，我制定了简单鲁棒的止损方案：业务方接入 SkyWalking 探针时，需要进行一段时间的灰度接入，确保灰度一段时间没有问题，才可以全量接入。而如果灰度节点出现了问题，一些 CASE 自动（如进程挂掉）和兜底手动将通过脚本使用 Dump 命令对现场进行保留，然后打通发布平台快速摘除探针并上线。</p>
<p data-nodeid="1095">举个实施这个方案的案例，我在大规模部署 SkyWalking 探针初期，发现线上环境出现了两起偶发的进程死锁，导致进程无法启动，而这些进程唯一的共性就是最近部署了 SkyWalking 探针。业务方按照上述方案的步骤进行了现场留痕，然后快速止损；并将现场与我分享，我也很快定位到这是 SkyWalking 探针内核与 Web 容器的类加载器发生的死锁造成的。</p>
<p data-nodeid="1794" class="">如果你对此案例感兴趣，可以访问这个 <a href="https://github.com/apache/skywalking/issues/3784?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1798">GitHub 链接</a>，了解这个案例的前因后果和完整过程，详细学习问题定位和修复过程。</p>

<h4 data-nodeid="1097">2.针对不同中间件，如何设计差异化落地方案？</h4>
<p data-nodeid="1098">APM 监控存储的数据都是海量级别的。如果全记录下来，需要投入大量的设备资源，但带来的收益却很小，所以我们需要“因地制宜”地制定差异化落地方案。</p>
<p data-nodeid="1099">假设要设计一款针对 Redis 的 APM 系统，也就是要做到能监控到的数据最小粒度为 Redis 操作命令，那势必要做出性能比 Redis 还要好的产品才可以。而 SkyWalking 存储数据的维度为任务线程维度，在任务线程内的 APM 数据会被打包，一定程度可以将这种“不可能”变为“可能”。</p>
<p data-nodeid="1100">但线上不只有 Redis 这些高性能中间件，还有像 MongoDB、TiDB 和访问 CDN 等各种高性能的中间件。所以在落地时，我们不可能让所有应用使用统一的方案进行 SkyWalking 落地，那如何设计差异化落地方案呢？</p>
<p data-nodeid="1101">这就需要根据企业的应用服务特点，将应用服务分类，进行差异化接入。</p>
<p data-nodeid="1102">结合国内互联网公司（非云厂商、服务厂商）的应用服务特点，我们可以根据流量和业务复杂度两个特点，将要监控的业务系统快速的分为两类：</p>
<ul data-nodeid="1103">
<li data-nodeid="1104">
<p data-nodeid="1105">高 QPS 低业务编排能力的底层系统</p>
</li>
<li data-nodeid="1106">
<p data-nodeid="1107">低 QPS 高业务赋能的业务系统</p>
</li>
</ul>
<p data-nodeid="1108">其中底层系统只会开启应用系统全链路跟踪所必要的组件，如 HttpServer、RPC 框架、Kafka 等。如果底层系统还是占用了过多的资源，就会针对性地开启采样。</p>
<blockquote data-nodeid="1109">
<p data-nodeid="1110">“采样”相关内容，我将在《16 | 采样设计：资源有限，如何实现数据的低损耗、高收集？》中与你分享。</p>
</blockquote>
<h4 data-nodeid="1111">3.数据暴露的同时，如何保障信息安全？</h4>
<p data-nodeid="1112">数据是公司的机密，商业模式转化为机器语言是公司的核心竞争力。当你引入 SkyWalking 后，会发现固定的账户密码登录后，这些涉密信息便一览无余。</p>
<p data-nodeid="1113">面对这种情况，安全组肯定不允许 APM 落地，那如何在保障信息安全的前提下，让数据展示出来呢？</p>
<p data-nodeid="1114">我的落地方案是<strong data-nodeid="1243">3 个步骤加 1 个定论</strong>。</p>
<p data-nodeid="1115"><strong data-nodeid="1247">【3 个步骤】</strong></p>
<ul data-nodeid="1116">
<li data-nodeid="1117">
<p data-nodeid="1118">SkyWalking 接入公司的登录系统，每个 APM 数据都必须有相应的应用归属。当用户查询数据时，只有用户所在的应用归属与 APM 数据的应用归属一致才可以展示。</p>
</li>
<li data-nodeid="1119">
<p data-nodeid="1120">APM 在定位问题时，需要相关资源的数据联动展示，才能发挥出真正的价值。所以在应用拓扑和全链路追踪时，会展示相关联的技术数据，如端点信息、耗时等；但不会展示核心数据，如接口的出入参，执行 SQL 具体信息等。</p>
</li>
<li data-nodeid="1121">
<p data-nodeid="1122">同时也有部分相关联的数据是不对外展示的，如公司的人事、薪酬等，只对本应用负责的 RD 展示。</p>
</li>
</ul>
<p data-nodeid="1123"><strong data-nodeid="1254">【1 个定论】</strong></p>
<p data-nodeid="1281" class="">我们需要带上 SkyWalking 和信息安全功能的功能配置，与安全组进行信息拉齐，在整体通过安全组审核后，才可以将应用 APM 数据的展示配置开放给业务线。**切记道路千万条，安全第一条。</p>

<h3 data-nodeid="1125">小结与思考</h3>
<p data-nodeid="1126">这一讲，我带你从产品功能视角纵观了 Apache SkyWalking 的完整发展史，希望你也可以温故知新，从 SkyWalking 的发展过程中预见 SkyWalking 未来的发展方向。</p>
<blockquote data-nodeid="1127">
<p data-nodeid="1128">你可以在拉勾教育另一专栏<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=50&amp;sid=20-h5Url-0#/content&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1266">《31 讲带你搞懂 SkyWalking》</a>中，继续更系统、深入的 SkyWalking 专题学习。</p>
</blockquote>
<p data-nodeid="1129">希望学完之后，你在企业级落地的过程中，能以提出通用解决方案为目标，不仅努力克服遇到的各种难题，还能将这种经验沉淀为方法论，并通过 Apache Way 回馈给 SkyWalking 社区，获得社区的认可。成为持续贡献的 Apache Committer，赢得 GitHub 专属 Apache 成员认证和 IntelliJ IDEA 全家桶终身免费账户。</p>
<p data-nodeid="1130">那么你的团队目前有没有正在使用，或者正准备使用 Apache SkyWalking 来完成应用集群 APM 系统的落地？</p>
<p data-nodeid="1131">如果有的话，请访问<a href="https://github.com/apache/skywalking/blob/v8.3.0/docs/en/setup/service-agent/java-agent/Supported-list.md?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1273">Apache SkyWalking V8.3 最新版本支持监控的框架</a>，对比你负责的应用服务使用的框架是否已被 Apache SkyWalking 支持。若不在支持的列表内，请你设计出实现的路径。</p>
<p data-nodeid="1132">由于 SkyWalking 采用的是“无侵入字节码增强”方式去实现 APM 监控数据的采集，而非“拦截器接口”这一方式，所以在应用框架大版本迭代后会很难兼容。因此，请你注意 Apache SkyWalking 是否支持相应监控的应用框架版本号。</p>
<p data-nodeid="1133" class="">如果你遇到了此类问题，可以通过<a href="https://github.com/apache/skywalking/tree/v8.3.0#contact-us?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1279">Apache SkyWalking 团队的联系方式</a>或在留言区与我讨论。</p>

---

### 精选评论

##### 赵禹光：
> SkyWalking 在一站式 APM、Sentinel 在集群流量治理上绝对是当下已经未来流行的趋势，作为核心作者，专栏很多课程的内容就是基于上面这两个工具的设计举例说明。你负责的服务也很可能在使用这两个工具，那一定要利用好专栏的特色，以这两个非常有代表性的工具为起点，快速提升 APM 工具的认知，完成“从 0 到 2”的工具掌握。希望你在学习之余，也能多多到社区内进行相应训练，给自己二次加餐。

##### *哲：
> RD是什么？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 开发同学就称为 RD

##### **春：
> 可以贡献给CNCF社区，CNCF已经在孵化open tracing

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; SkyWalking是Apache社区的顶级项目。

##### xxx：
> 老师好，问个问题，7.x和8.x协议变了，怎么平滑升级到8.x

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 协议变化没有优雅平滑升级方案，我的推荐方案是部署两套OAP集群，在低峰期让重启应用服务集群，让新的8.X探针打到新集群，然后留一台老集群节点用于历史数据查询。这方案的好处是，新老数据都在，切换速度很快且具备回滚方案。

