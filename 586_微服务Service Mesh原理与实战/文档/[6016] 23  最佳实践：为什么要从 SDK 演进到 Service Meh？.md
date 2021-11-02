<p data-nodeid="485101">上一讲我们提到了微服务和 Service Mesh 架构在落地中的遇到的问题和困难，也讲解了 Service Mesh 相较于传统微服务架构的优势，以及在落地过程中业务方对 Service Mesh 架构可能提出的一些疑问和处理办法。这一讲我们从实践角度出发，继续分析 Service Mesh 架构到底解决了哪些线上问题。</p>
<h3 data-nodeid="485102">线上故障梳理</h3>
<p data-nodeid="485103">服务治理解决的是一些线上故障导致的微服务集群雪崩问题，为了更好地理解服务治理，我们来梳理一下有哪些常见的线上故障，以及如何通过服务治理的手段解决这些故障，看看这些故障能否通过 Service Mesh 架构得到很好的解决。</p>
<p data-nodeid="485104"><strong data-nodeid="485158">服务变更故障</strong></p>
<p data-nodeid="485105">线上故障中，出现概率最大的就是服务变更故障，一般指的是业务迭代进行版本更新，在更新过程中遇到的故障，也包括项目配置变更，数据变更等。</p>
<p data-nodeid="485106">一般来说，这种问题只要及时回滚就可以解决，所以<strong data-nodeid="485169">最大的难点在于如何用最快的速度发现故障</strong>。这个时候就需要<strong data-nodeid="485170">通过 Metrics 进行故障报警</strong>，在 Service Mesh 架构中，可以不依赖于框架或者业务注入 Metrics，通过收集 Sidecar 中的 Metrics ，达到监控报警的目的。</p>
<p data-nodeid="485107"><strong data-nodeid="485174">突发流量导致的故障</strong></p>
<p data-nodeid="485108">除去服务变更故障，突发流量是另外一个高频发生的故障类型。比如突发热点，或者运营活动，都可能导致这种故障。</p>
<p data-nodeid="485109"><strong data-nodeid="485184">突发热点</strong>比较好理解，比如某个明星的突发新闻，在微博平台就经常会出现突发流量。这种热点问题往往具备不可预测性，指望提前扩容机器不太现实，最好的方式就是通过 Service Mesh 服务治理的手段之一——<strong data-nodeid="485185">限流</strong>，在突发时减少影响，防止整个微服务集群雪崩。虽然会有一定的流量损失，但可以保证平稳地度过热点突发。</p>
<p data-nodeid="485110">另外一种就是<strong data-nodeid="485199">运营活动</strong>，一般可以通过<strong data-nodeid="485200">提前扩容机器解决</strong>。但我们<strong data-nodeid="485201">平时也要做好限流手段</strong>，防止流量预估错误或者运营活动没有提前通知到开发人员。</p>
<p data-nodeid="485111"><strong data-nodeid="485205">依赖故障</strong></p>
<p data-nodeid="485112">依赖故障，主要指的是 upstream 上游服务出现故障时，导致的级联故障，如果没有相应的服务治理手段，将会导致整条微服务链路被拖慢，最终导致集群雪崩。</p>
<p data-nodeid="485113">应对这种问题的办法，就是通过 Service Mesh 服务治理的手段之一——熔断，<strong data-nodeid="485216">在上游服务出现故障的时候，直接熔断，对上游服务的请求直接返回错误</strong>，这样就减少了服务链路的耗时，从而避免微服务集群雪崩。另外熔断也可以<strong data-nodeid="485217">配合降级使用</strong>，在一些场景下，比如 Feed 流，如果上游服务故障触发熔断时，返回默认数据或者调用降级服务，从而避免“开天窗”。</p>
<p data-nodeid="485114"><strong data-nodeid="485221">网络引起的故障</strong></p>
<p data-nodeid="485115">这种故障往往发生在<strong data-nodeid="485227">服务新加节点</strong>的情况，因为网络配置问题，这些节点可能在某些网段不通，但是和注册中心是相通的，这就导致注册中心的健康检查可以通过，但是服务调用的时候却无法联通。</p>
<p data-nodeid="485116">此时就需要 Service Mesh 中的负载均衡器有节点摘除的功能，<strong data-nodeid="485233">在发生网络故障的时候将失败的节点摘除掉</strong>。当然，我们平时也需要做好网络分区的规划，确保服务部署在多个分区，以避免某个网络分区故障引起整个网络瘫痪。</p>
<p data-nodeid="485117"><strong data-nodeid="485237">机器引起的故障</strong></p>
<p data-nodeid="485118">机器引起的故障主要是指机器 hang 住，或者机器 down 机重启的情况。机器故障问题可以通过<strong data-nodeid="485247">注册中心的健康检查解决</strong>。如果是在 Kubernetes 环境中，则需要注意，<strong data-nodeid="485248">不要将一个服务的 Pod 都调度在同一台机器上</strong>，否则机器故障会导致整个服务不可用。</p>
<h3 data-nodeid="485119">如何提升研发效率</h3>
<p data-nodeid="485120">讲完了线上故障，下面我们再从研发效率的角度出发，看一看 Service Mesh 架构如何提升开发的研发效率。Service Mesh 架构层面的优势，让业务方尽可能地少感知基础设施，更关注业务本身，当然以下这些提升点也都是我在实践落地过程中不断总结得出的。</p>
<p data-nodeid="485121"><strong data-nodeid="485254">代理注册</strong></p>
<p data-nodeid="485122">如果没有使用 Kubernetes 内置的注册发现体系，就需要通过框架来注册。但是框架注册免不了业务配置一些参数，比如服务名、运行环境等。一旦这些参数配置错误会引发线上故障，所以通过 Sidecar 代理注册是最好的解决方案。<strong data-nodeid="485260">通过控制面下发注册信息</strong>，避免了业务方手动设置注册参数的过程，减少故障发生的概率。</p>
<p data-nodeid="485123"><strong data-nodeid="485264">安全通信</strong></p>
<p data-nodeid="485124">在内网环境中，内网安全往往没有外网安全那么受重视。云原生的环境，提出了零信任的概念，这个时候在业务代码中加入 TLS 认证或者 Token 认证，都存在不好控制和维护的问题。直接在 Service Mesh 中，集成安全相关的功能，将是最合适的选择，服务间通过<strong data-nodeid="485270">双向 TLS 认证</strong>的方式进行安全通信。</p>
<p data-nodeid="485125"><strong data-nodeid="485274">平滑重启</strong></p>
<p data-nodeid="485126">在传统的虚拟机环境下，服务想要做到平滑重启，除了自身用代码实现相对复杂的平滑重启外，还有一种方式就是通过关闭前反注册摘除流量，启动后再注册的方式实现平滑重启。当然，我们也可以借助 Sidecar 的能力，<strong data-nodeid="485280">通过和 Sidecar 的控制接口通信，调用接口控制注册/反注册的方式，实现平滑重启</strong>。这个功能可以集成在 CD 平台中，从而实现和业务代码的解耦，业务无须关心注册/反注册的过程。</p>
<p data-nodeid="485127"><strong data-nodeid="485284">平滑放量</strong></p>
<p data-nodeid="485128">在一些场景中，比如 Java 语言，启动相对比较慢。如果节点刚启动就放大量流量进入，一些线程池还没来得及预热，就接管流量，会导致大量的错误。此时我们可以<strong data-nodeid="485290">利用 Sidecar 实现平滑放量的功能</strong>，新加入的节点先将权重设置为 1，然后缓慢增加到 100，可以预热 Java 的线程池，达到平滑放量的目的。</p>
<p data-nodeid="485129"><strong data-nodeid="485294">染色</strong></p>
<p data-nodeid="485130">通过染色的功能，可以很好地进行<strong data-nodeid="485300">流量区分</strong>，达到线上真实流量测试的目的，甚至可以配合故障演练使用。将某些符合条件的流量打上特定的标记，通过 header 在微服务之间传递，进而达到流量染色的目的。</p>
<p data-nodeid="485131">至此，如何利用 Service Mesh 架构提高研发效率的相关内容我们就讲完了。下面我们来看一个在 Service Mesh 实践过程中很容易遇到的场景：如何从虚拟机环境迁移到 Kubernetes 环境。</p>
<h3 data-nodeid="490113" class="">如何从虚拟机环境迁移到 Kubernetes 环境</h3>








<p data-nodeid="485133"><strong data-nodeid="485314">服务发现问题</strong></p>
<p data-nodeid="485134">在虚拟机迁移 Kubernetes 的过程中，第一个面临的问题就是<strong data-nodeid="485324">服务发现</strong>。首先是方案选型，在虚拟机环境中，往往使用独立的注册中心，而 Kubernetes 中也提供了注册发现的功能。这里我比较建议<strong data-nodeid="485325">沿用独立注册中心的方案</strong>。</p>
<p data-nodeid="485135">在灰度过程中，往往会面临两个环境同时注册的问题。为了稳定性和排查问题方便，一般会将环境隔离，两个环境分别使用不同的注册中心集群。这样也导致了相互发现的问题，这里我们可以<strong data-nodeid="485331">通过控制面聚合多个注册中心数据的方式解决，发现方式则统一到 Envoy EDS 协议（节点发现服务）</strong>。</p>
<p data-nodeid="485136"><strong data-nodeid="485335">负载均衡问题</strong></p>
<p data-nodeid="485137">虚拟机迁移到容器环境后，传统的负载均衡算法会因为 Pod 宿主机机器配置的问题，导致流量均衡，而 CPU 负载不均衡的问题。这个时候我们可以通过 P2C（Pick of 2 choices）的算法解决。</p>
<p data-nodeid="485138"><strong data-nodeid="485340">限流设置问题</strong></p>
<p data-nodeid="485139">虚拟机迁移容器环境后，因为容器环境常常会配置 HPA（自动扩缩容），HPA 会根据 CPU 的水位对 Pod 的数量进行调整，在流量上涨的情况，会进行扩容操作。此时如果限流值配置不合理，会影响扩容操作，导致不必要的流量损失。这个时候需要<strong data-nodeid="485346">引入自适应限流的算法</strong>，根据延时、CPU 水位等信息，自动调整限流值，避免手动配置引起的故障。</p>
<p data-nodeid="485140"><strong data-nodeid="485350">灰度流量问题</strong></p>
<p data-nodeid="485141">在虚拟机向容器迁移的过程中，不可避免会涉及如何灰度流量的问题。在传统的 SDK 模式中，需要业务反复发版或者配置中心修改才能调整服务节点的权重。但是在两种环境下，配置中心针对两种环境设置不同的配置才可以做到，服务代码发版也需要在两个环境发版，因为同一个服务代码配置不同，很容易在生产环境产生人为故障。</p>
<p data-nodeid="485142">上述问题都可以通过 Service Mesh 控制面解决，<strong data-nodeid="485357">控制面支持两种环境不同的权重配置</strong>，结合服务治理平台提供的 Web UI，可以让运维非常方便地操作，而不需要业务方修改代码或者配置中心。</p>
<p data-nodeid="485143">至此，如何利用 Service Mesh 的能力，平滑地从虚拟机环境迁移到 Kubernetes 环境的内容就讲完了。下面我们对今天的内容做一个简单的总结。</p>
<h3 data-nodeid="485144">总结</h3>
<p data-nodeid="485145">这一讲我介绍了在实践中 Service Mesh 主要解决了哪些业务痛点。相信通过今天的内容，你能够学习到如何利用 Service Mesh 的强大能力，解决研发痛点、提高微服务的健壮性，以及利用 Service Mesh 的能力让架构平滑地从虚拟机环境迁移到 Kubernetes 环境。</p>
<p data-nodeid="485146">本讲内容总结如下：</p>
<p data-nodeid="490627" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/13/54/CioPOWBCBymAbElLAAHzHabcr4w419.png" alt="最佳实践：为什么要从 SDK  演进到 Service Mesh？.png" data-nodeid="490630"></p>

<p data-nodeid="485148">结合今天的内容，你能谈一谈在现实的工作场景中，还有哪些问题可以用 Service Mesh 的架构解决吗？欢迎在留言区和我分享你的想法。</p>
<p data-nodeid="485149">今天内容到这里就结束了，下一讲我们将迎来专栏的最后内容《未来展望：中间件 Mesh 化的可行性》。此外，在下一讲中我还会讲解 FaaS 和 Service Mesh 的结合。请你继续阅读。</p>

---

### 精选评论

##### **军：
> 希望继续更新啊 哈哈哈

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 嘿嘿，今天就是最后一讲啦！

