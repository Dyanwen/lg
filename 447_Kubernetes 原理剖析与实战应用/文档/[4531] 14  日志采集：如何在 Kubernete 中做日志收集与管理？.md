<p data-nodeid="266911">说到日志，你应该不陌生。日志中不仅记录了代码运行的实时轨迹，往往还包含着一些关键的数据、错误信息，等等。日志方便我们进行分析统计及监控告警，尤其是在后期问题排查的时候，我们通过日志可以很方便地定位问题、现场复现及问题修复。日志也是做可观测性（Observability）必不可少的一部分。</p>
<p data-nodeid="266912">因此在使用 Kubernetes 的过程中，对应的日志收集也是我们不得不考虑的问题。我们需要日志去了解集群内部的运行状况。</p>
<p data-nodeid="266913">我们先来看看 Kubernetes 的日志收集和以往的日志收集有什么差别，以及为什么我们需要为 Kubernetes 的日志收集单独设计方案。</p>
<h3 data-nodeid="266914">Kubernetes 中的日志收集 VS 传统日志收集</h3>
<p data-nodeid="266915">对于传统的应用来说，它们大都都是直接运行在宿主机上的，会将日志直接写入本地的文件中或者由 systemd-journald 直接管理。在做日志收集的时候，只需要访问这些日志所在的目录即可。此类日志系统解决方案非常多，也相对比较成熟，在这里就不再过多说明。我们重点来看看 Kubernetes 的日志系统建设问题。</p>
<p data-nodeid="266916">在 Kubernetes 中，日志采集相比传统虚拟机、物理机方式要复杂很多。</p>
<p data-nodeid="266917">首先，日志的形式非常多样化。日志需求主要集中在如下三个部分：</p>
<ul data-nodeid="266918">
<li data-nodeid="266919">
<p data-nodeid="266920">系统各组件的日志，比如 Kubernetes 自身各大组件的日志（包括 kubelet、kube-proxy 等），容器运行时的日志（比如 Docker）；</p>
</li>
<li data-nodeid="266921">
<p data-nodeid="266922">以容器化方式运行的应用程序自身的日志，比如 Nginx、Tomcat 的运行日志；</p>
</li>
<li data-nodeid="266923">
<p data-nodeid="266924">Kubernetes 内部各种 Event（事件），比如通过<code data-backticks="1" data-nodeid="267003">kubebctl create</code>创建一个  Pod 后，可以通过<code data-backticks="1" data-nodeid="267005">kubectl describe pod pod-xxx</code>命令查看到的这个 Pod 的 Event 信息。</p>
</li>
</ul>
<p data-nodeid="266925">其次，集群环境时刻在动态变化。我们都知道 Pod “用完即焚”，Pod 销毁后日志也会一同被删除。但是这个时候我们仍然希望可以看到具体的日志，用于查看和分析业务的运行情况，以及帮助我们发现出容器异常的原因。</p>
<p data-nodeid="266926">同时新的 Pod 可能随时会“飘”到别的节点上重新“生长”出来，我们无法提前预知具体是哪个节点。而且 Kubernetes 的节点也会存在宕机等异常情况。所以说，Kubernetes 的日志系统在设计的时候，必须得独立于节点和 Pod 的生命周期，且保证日志数据可以实时采集到服务端，即完全独立于 Kubernetes 系统，使用自己的后端存储和查询工具。</p>
<p data-nodeid="266927">再次，日志规模会越来越大。很多人在 Kubernetes 中喜欢使用 hostpath 来保存 Pod 的日志，并且不做日志轮转（可以配置 Docker 的<code data-backticks="1" data-nodeid="267010">log-opts</code>来<a href="https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver" data-nodeid="267014">设置容器的日志轮转</a><a href="https://docs.docker.com/config/containers/logging/configure/#configure-the-default-logging-driver" data-nodeid="267017"> </a>），这很容易将宿主机的磁盘“打爆”。这里你是不是觉得如果做了轮转，磁盘打爆的问题就可以完美解决了？</p>
<p data-nodeid="266928">其实虽有所缓解，并不会让你安全无忧。虽说日志轮转可以有效减少日志的文件大小，但是你会丢失掉不少日志，后续想要分析和排查问题时就无从下手了。想想看如果容器内的应用出现异常并疯狂报错，这个时候又有大量的并发请求，那么日志就会急剧增多。</p>
<p data-nodeid="266929">配置了日志轮转，会让你丢失很多重要的上下文信息。如果没有配置日志轮转，这些日志很快就会将磁盘打爆。还有可能引发该节点的 Kubelet 异常，导致该节点上的 Pod 被驱逐。我们在一些生产实践中，就遇到过这种情况。同时当 Pod 被删除后，这些 hostpath 的文件并不会被及时删除，会继续占用很多磁盘空间。此外，随着业务逐渐增长，在这个节点上运行过的 Pod 也会变多，这就会残留大量的日志文件。</p>
<p data-nodeid="266930">此外，日志非常分散且种类多变。单纯查找一个应用的日志，就需要查看其关联的分散在各个节点上的各个 Pod 的日志。在出现紧急情况需要排查的时候，这种方式极其低效，会严重影响到问题修复和服务恢复。如果这个应用还通过 Ingress 对外暴露服务，并使用了 Service Mesh 等，那么此时做日志收集就更复杂了。</p>
<p data-nodeid="266931">随着在 Kubernetes 上落地越来越多的微服务，各个服务之间的依赖也越来越多。这个时候各个维度的日志关联也是一个非常困难的问题。那么，下面我们就来看看如何对 Kubernetes 做日志收集。</p>
<h3 data-nodeid="266932">几种常见的 Kubernetes 日志收集架构</h3>
<p data-nodeid="266933">Kubernetes 集群本身其实并没有提供日志收集的解决方案，但依赖 Kubernetes 自身提供的各项能力，可以帮助我们解决日志收集的诉求。根据上面提到的三大基本日志需求，一般来说我们有如下有三种方案来做日志收集：</p>
<ol data-nodeid="266934">
<li data-nodeid="266935">
<p data-nodeid="266936">直接在应用程序中将日志信息推送到采集后端；</p>
</li>
<li data-nodeid="266937">
<p data-nodeid="266938">在节点上运行一个 Agent 来采集节点级别的日志；</p>
</li>
<li data-nodeid="266939">
<p data-nodeid="266940">在应用的 Pod 内使用一个 Sidecar 容器来收集应用日志。</p>
</li>
</ol>
<p data-nodeid="266941">我们来分别看看这三种方式。</p>
<p data-nodeid="272298">先来看<strong data-nodeid="272305">直接在应用程序中将日志信息推送到采集后端，即Pod 内的应用直接将日志写到后端的日志中心</strong>。</p>
<p data-nodeid="272299" class=""><img src="https://s0.lgstatic.com/i/image/M00/59/F7/Ciqc1F9zCLCALLfHAAA7edHbxKE531.png" alt="image (2).png" data-nodeid="272312"></p>






<p data-nodeid="273023" class="">（<a href="https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-from-application.png" data-nodeid="273027">https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-from-application.png</a>）</p>

<p data-nodeid="266945">通常有两种做法，一个就是应用程序通过对应的日志 SDK 进行接入，不过这种做法一般不推荐，和应用本身耦合太严重，也不方便后续对接其他的日志系统。</p>
<p data-nodeid="266946">还有一种做法就是通过容器运行时提供的 Logging  Driver 来实现。以最常用的 Docker 为例，目前已经支持了<a href="https://docs.docker.com/config/containers/logging/configure/#supported-logging-drivers" data-nodeid="267132">十多种 Logging  Driver</a>。比如你可以配置为<code data-backticks="1" data-nodeid="267134">fluentd</code>，这个时候 Docker 就会将容器的标准输出日志（stdout、stderr）直接写到<code data-backticks="1" data-nodeid="267136">fluentd</code>中。你也可以设置成<code data-backticks="1" data-nodeid="267138">awslogs</code>，这样就会直接将日志写到 Amazon CloudWatch Logs 中。但是在和 Kubernetes 一起使用的时候，使用较多的是<code data-backticks="1" data-nodeid="267140">json-file</code>，这也是 Docker 默认的 Logging Driver。</p>
<pre class="lang-shell" data-nodeid="266947"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker info |grep <span class="hljs-string">'Logging Driver'</span></span>
Logging Driver: json-file
</code></pre>
<p data-nodeid="266948">你经常使用的<code data-backticks="1" data-nodeid="267143">kubectl logs</code>就是基于<code data-backticks="1" data-nodeid="267145">json-flle</code>这种 Logging Driver 来实现的，目前 Kubernetes 也只支持<code data-backticks="1" data-nodeid="267147">json-flle</code>这一种 Logging Driver。</p>
<p data-nodeid="266949">所以在 Kubernetes 的这套体系中，直接将日志写到后端日志采集系统中去，并不是特别好的做法。</p>
<p data-nodeid="274141">我们来看第二种方法，<strong data-nodeid="274150">在节点上运行一个 Agent 来采集节点级别的日志</strong>。如下图所示，我们可以在每一个 Kubernetes Node 上都部署一个 Agent，该 Agent 负责对该节点上运行的所有容器进行日志收集，并推送到后端的日志存储系统里。这个 Agent 通常需要可以访问到宿主机上的指定目录，比如<code data-backticks="1" data-nodeid="274148">/var/lib/docker/containers/</code>。</p>
<p data-nodeid="274142" class=""><img src="https://s0.lgstatic.com/i/image/M00/5A/02/CgqCHl9zCNiAEVCiAABrnSxfaQg197.png" alt="image (3).png" data-nodeid="274157"></p>


<p data-nodeid="266952">（<a href="https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-with-node-agent.png" data-nodeid="267164">https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-with-node-agent.png</a>）</p>
<p data-nodeid="266953">由于这样的 Agent 需要在每个 Node 上都运行，因此我们都是通过 上节课讲到的<code data-backticks="1" data-nodeid="267167">DaemonSet</code>的方式来部署的。对于 Kubernetes 集群来说，这种使用节点级的<code data-backticks="1" data-nodeid="267169">DaemonSet</code>日志代理是最常用，也是最被推荐的方式，不仅可以节约资源，而且对于应用来说也是无侵入的。</p>
<p data-nodeid="275303">但是这种方式也有个缺点，就是只适应于容器内应用日志是标准输出的场景，即应用把日志输出到<code data-backticks="1" data-nodeid="275306">stdout</code>和<code data-backticks="1" data-nodeid="275308">stderr</code>。</p>
<p data-nodeid="275304">最后来看<strong data-nodeid="275319">通过 Sidecar 来收集容器日志</strong>。 在 Pod 里面，容器的输出日志可以是<code data-backticks="1" data-nodeid="275315">stdout</code>、<code data-backticks="1" data-nodeid="275317">stderr</code>和日志文件。那么基于这三种形式，我们可以借助于 Sidecar 容器</p>




<p data-nodeid="266957">将基于文件的日志来帮助我们。</p>
<ul data-nodeid="276159">
<li data-nodeid="276160">
<p data-nodeid="276161">通过Sidecar 容器读取日志文件，并定向到自己的标准输出。如下图所示，这里<code data-backticks="1" data-nodeid="276164">streaming container</code>就是一个 Sidecar 容器，可以将<code data-backticks="1" data-nodeid="276166">app-container</code>的日志文件重新定向到自己的标准输出。同时还可以归并多个日志文件。而且这里也可以使用多个 Sidecar 容器，你可以参考这个<a href="https://github.com/kubernetes/website/blob/master/content/en/examples/admin/logging/two-files-counter-pod-streaming-sidecar.yaml" data-nodeid="276170">例子</a>。</p>
</li>
</ul>
<p data-nodeid="276162" class=""><img src="https://s0.lgstatic.com/i/image/M00/5A/02/CgqCHl9zCOWAI1-SAAB3nPjxdMA390.png" alt="image (4).png" data-nodeid="276178"></p>


<ul data-nodeid="277319">
<li data-nodeid="277320">
<p data-nodeid="277321">Sidecar容器运行一个日志代理，配置该日志代理以便从应用容器收集日志。这种方式就解决了我们上面方案一的问题，将日志处理部分和应用程序本身进行了解耦，可以方便切换到其他的日志系统中。可以参考这个<a href="https://github.com/kubernetes/website/blob/master/content/en/examples/admin/logging/two-files-counter-pod-agent-sidecar.yaml" data-nodeid="277326">使用 fluentd 的例子</a>。</p>
</li>
</ul>
<p data-nodeid="277322" class=""><img src="https://s0.lgstatic.com/i/image/M00/5A/02/CgqCHl9zCOuAFQm_AABLBPDcBz4058.png" alt="image (5).png" data-nodeid="277334"></p>


<p data-nodeid="266966">（<a href="https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-with-sidecar-agent.png" data-nodeid="267210">https://github.com/kubernetes/website/blob/master/static/images/docs/user-guide/logging/logging-with-sidecar-agent.png</a>）</p>
<p data-nodeid="266967">可以看到,通过 Sidecar 的方式来收集日志，会增加额外的开销。在集群规模较小的情况下可以忽略不计,但是对于大规模集群来说，这些开销还是不可忽略的。</p>
<p data-nodeid="266968">那么，上面的这几套方案，在实际使用的时候，又该如何选择呢？社区又有什么推荐方案呢？</p>
<h3 data-nodeid="266969">基于 Fluentd + ElasticSearch 的日志收集方案</h3>
<p data-nodeid="278491">Kubernetes 社区官方推荐的方案是<a href="https://kubernetes.io/zh/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/" data-nodeid="278496">使用 Fluentd+ElasticSearch+Kibana 进行日志的收集和管理</a>，通过<a href="https://www.fluentd.org/" data-nodeid="278500">Fluentd</a>将日志导入到<a href="https://www.elastic.co/products/elasticsearch" data-nodeid="278504">Elasticsearch</a>中，用户可以通过<a href="https://www.elastic.co/products/kibana" data-nodeid="278508">Kibana</a>来查看到所有的日志。</p>
<p data-nodeid="278492" class=""><img src="https://s0.lgstatic.com/i/image/M00/59/F7/Ciqc1F9zCPSATIwRAAA_ddgWuO0667.png" alt="image (6).png" data-nodeid="278516"></p>


<p data-nodeid="266972">(<a href="https://docs.fluentd.org/container-deployment/kubernetes" data-nodeid="267238">https://docs.fluentd.org/container-deployment/kubernetes</a>)</p>
<p data-nodeid="266973">关于 ElasticSearch 和 Kibana 如何部署，在此就不过多地介绍了，你可以通过添加 helm 的 repo即<code data-backticks="1" data-nodeid="267241">helm repo add elastic https://helm.elastic.co</code>，然后通过 helm来快速地自行部署，相关的 Chart 见<a href="https://github.com/elastic/helm-charts/blob/master/elasticsearch/README.md" data-nodeid="267245">https://github.com/elastic/helm-charts</a>。</p>
<p data-nodeid="266974">现在我们就来看看这套方案的几个技术点。</p>
<p data-nodeid="266975">Fluentd 提供了强大的日志统一接入能力，同时内置了插件，可以对接 ElasticSearch。这里 Fluentd 主要有如下四个配置：</p>
<ul data-nodeid="266976">
<li data-nodeid="266977">
<p data-nodeid="266978"><code data-backticks="1" data-nodeid="267249">fluent.conf</code>这个文件主要是用来设置一些地址，比如 ElasticSearch 的地址等；.</p>
</li>
<li data-nodeid="266979">
<p data-nodeid="266980"><code data-backticks="1" data-nodeid="267251">kubernetes.conf</code>这个文件记录了与 Kubernetes 相关的配置，比如 Kubernetes 各组件的日志配置、容器的日志收集规则，等等；</p>
</li>
<li data-nodeid="266981">
<p data-nodeid="266982"><code data-backticks="1" data-nodeid="267253">prometheus.conf</code>这个文件定义了 Prometheus 的地址，方便 Fluentd 暴露自己的统计指标；</p>
</li>
<li data-nodeid="266983">
<p data-nodeid="266984"><code data-backticks="1" data-nodeid="267255">systemd.conf</code>这个文件可以配置 Fluentd 通过 systemd-journal 来收集哪些服务的日志，比如 Docker 的日志、Kubelet 的日志等。</p>
</li>
</ul>
<p data-nodeid="266985">上面的这些配置，都默认内置到了<a href="https://hub.docker.com/r/fluent/fluentd-kubernetes-daemonset/tags?page=1&amp;name=elasticsearch" data-nodeid="267260"> fluent/fluentd-kubernetes-daemonset</a>的镜像中，你可以使用官方的默认配置。如果你想要定制化更改一些，可以参照<a href="https://github.com/fluent/fluentd-kubernetes-daemonset/tree/master/docker-image/v1.11/debian-elasticsearch7/conf" data-nodeid="267264">这份默认示例配置</a>。</p>
<p data-nodeid="266986">如下的 YAML 是一段 fluentd 的 DaemonSet 定义，源自<a href="https://github.com/fluent/fluentd-kubernetes-daemonset/blob/master/fluentd-daemonset-elasticsearch.yaml" data-nodeid="267269">这里</a>：</p>
<pre class="lang-yaml" data-nodeid="266987"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">DaemonSet</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">fluentd</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">kube-system</span>
  <span class="hljs-attr">labels:</span>
    <span class="hljs-attr">k8s-app:</span> <span class="hljs-string">fluentd-logging</span>
    <span class="hljs-attr">version:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">matchLabels:</span>
      <span class="hljs-attr">k8s-app:</span> <span class="hljs-string">fluentd-logging</span>
      <span class="hljs-attr">version:</span> <span class="hljs-string">v1</span>
  <span class="hljs-attr">template:</span>
    <span class="hljs-attr">metadata:</span>
      <span class="hljs-attr">labels:</span>
        <span class="hljs-attr">k8s-app:</span> <span class="hljs-string">fluentd-logging</span>
        <span class="hljs-attr">version:</span> <span class="hljs-string">v1</span>
    <span class="hljs-attr">spec:</span>
      <span class="hljs-attr">tolerations:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">key:</span> <span class="hljs-string">node-role.kubernetes.io/master</span>
        <span class="hljs-attr">effect:</span> <span class="hljs-string">NoSchedule</span>
      <span class="hljs-attr">containers:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">fluentd</span>
        <span class="hljs-attr">image:</span> <span class="hljs-string">fluent/fluentd-kubernetes-daemonset:v1-debian-elasticsearch</span>
        <span class="hljs-attr">env:</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span>  <span class="hljs-string">FLUENT_ELASTICSEARCH_HOST</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"elasticsearch-logging"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span>  <span class="hljs-string">FLUENT_ELASTICSEARCH_PORT</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"9200"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_SCHEME</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"http"</span>
          <span class="hljs-comment"># Option to configure elasticsearch plugin with self signed certs</span>
          <span class="hljs-comment"># ================================================================</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_SSL_VERIFY</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"true"</span>
          <span class="hljs-comment"># Option to configure elasticsearch plugin with tls</span>
          <span class="hljs-comment"># ================================================================</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_SSL_VERSION</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"TLSv1_2"</span>
          <span class="hljs-comment"># X-Pack Authentication</span>
          <span class="hljs-comment"># =====================</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_USER</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"elastic"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">FLUENT_ELASTICSEARCH_PASSWORD</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"changeme"</span>
          <span class="hljs-comment"># Logz.io Authentication</span>
          <span class="hljs-comment"># ======================</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">LOGZIO_TOKEN</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"ThisIsASuperLongToken"</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">LOGZIO_LOGTYPE</span>
            <span class="hljs-attr">value:</span> <span class="hljs-string">"kubernetes"</span>
        <span class="hljs-attr">resources:</span>
          <span class="hljs-attr">limits:</span>
            <span class="hljs-attr">memory:</span> <span class="hljs-string">200Mi</span>
          <span class="hljs-attr">requests:</span>
            <span class="hljs-attr">cpu:</span> <span class="hljs-string">100m</span>
            <span class="hljs-attr">memory:</span> <span class="hljs-string">200Mi</span>
        <span class="hljs-attr">volumeMounts:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlog</span>
          <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/var/log</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlibdockercontainers</span>
          <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/var/lib/docker/containers</span>
          <span class="hljs-attr">readOnly:</span> <span class="hljs-literal">true</span>
      <span class="hljs-attr">terminationGracePeriodSeconds:</span> <span class="hljs-number">30</span>
      <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlog</span>
        <span class="hljs-attr">hostPath:</span>
          <span class="hljs-attr">path:</span> <span class="hljs-string">/var/log</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">varlibdockercontainers</span>
        <span class="hljs-attr">hostPath:</span>
          <span class="hljs-attr">path:</span> <span class="hljs-string">/var/lib/docker/containers</span>
</code></pre>
<h3 data-nodeid="266988">写在最后</h3>
<p data-nodeid="266989">在实际采集日志的时候，你可以根据自己的场景和集群规模选择适用的方案。或者也可以将上面的这几种方案进行合理地组合。</p>
<p data-nodeid="266990">到这里这节课就结束了，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **0530：
> 一般程序的业务日志都不是标准输出流的……都是自定义的file的日志，还是使用每个node挂一个agent最常用，可以结合ELK使用哈

