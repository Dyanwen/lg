<p data-nodeid="301673">微服务架构“分而治之”的手段将大型系统按业务分割成多个互相协作的微服务，每个微服务关注于自身业务职责，可以独立开发、部署和维护，从而更好地应对频繁的需求变更和迭代。但是数量众多的微服务实例给运维带来了巨大的挑战，如果没有好的办法快速部署和启动微服务，微服务架构带来的好处将所剩无几。而<strong data-nodeid="301827">容器化</strong>和<strong data-nodeid="301828">容器编排</strong>的兴起正好弥补了这个不足。</p>
<p data-nodeid="301674">etcd 是 Kubernetes 中的重要组件，用作<strong data-nodeid="301834">存储集群状态</strong>的数据库。etcd 作为配置存储中心使用，Kubernetes 可以更加专注容器编排的核心功能。</p>
<p data-nodeid="301675">这一讲我们就来介绍 Kubernetes 的相关概念，以及 etcd 在 Kubernetes 中的部署方式。我们也将通过一个用户服务部署的案例，介绍 etcd 在 Kubernetes 创建 Pod 过程中的作用。</p>
<h3 data-nodeid="301676">什么是 Kubernetes？</h3>
<p data-nodeid="301677">Docker 作为容器化技术，相比于虚拟化显得<strong data-nodeid="301842">更加轻量</strong>。然而仅有容器还是不够的，虽然它解决了应用程序运行环境的集成问题，但大量的容器需要人工部署，导致人力成本和出错率的增加，对此我们需要一定的容器编排工具对大量运行的容器进行管理。Kubernetes 就是这样一款工具。</p>
<p data-nodeid="301678">Kubernetes 由 Google 开源，目的是管理公司内部运行的成千上万台服务器，降低应用程序部署管理的成本。Kubernetes 将基础设施抽象化，简化了应用开发、部署和运维等工作，提高了硬件资源的利用率，是一款优秀的容器管理和编排系统。</p>
<p data-nodeid="301679">Kubernetes 主要由两类节点组成：<strong data-nodeid="301853">Master 节点</strong>主要负责管理和控制，是 Kubernetes 的调度中心；<strong data-nodeid="301854">Node 节点</strong>受 Master 节点管理，属于工作节点，负责运行具体的容器应用。整体结构图如下所示：</p>
<p data-nodeid="304650" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/C7/Cgp9HWBm9BCAXpFdAADbjd3yt1A943.png" alt="Drawing 0.png" data-nodeid="304654"></p>
<div data-nodeid="304651"><p style="text-align:center">Kubernetes 结构图</p></div>



<p data-nodeid="301682">Master 节点主要由以下几部分组成。</p>
<ul data-nodeid="301683">
<li data-nodeid="301684">
<p data-nodeid="301685"><strong data-nodeid="301864">API Server</strong>，对外提供 Kubernetes 的服务接口和 watch 监听机制，以供各类客户端使用，是 Kubernetes 中各组件交互的枢纽。当收到创建 Pod 请求时会对 API Server 进行认证检查，然后写入 etcd。</p>
</li>
<li data-nodeid="301686">
<p data-nodeid="301687"><strong data-nodeid="301869">Scheduler</strong>，负责对集群内部的资源进行调度，按照预设的策略将 Pod 调度到相应的 Node 节点。</p>
</li>
<li data-nodeid="301688">
<p data-nodeid="301689"><strong data-nodeid="301874">Controller Manager</strong>，作为管理控制器，负责维护整个集群的状态。</p>
</li>
<li data-nodeid="301690">
<p data-nodeid="301691"><strong data-nodeid="301879">etcd</strong>，保存整个集群的状态数据。</p>
</li>
</ul>
<p data-nodeid="301692">Node 节点的主要组成部分为：</p>
<ul data-nodeid="301693">
<li data-nodeid="301694">
<p data-nodeid="301695"><strong data-nodeid="301885">Pod</strong>，Kubernetes 创建和部署的基本操作单位，它代表了集群中运行的一个进程，内部由一个或多个共享资源的容器组成，我们可以简单将 Pod 理解成一台虚拟主机，主机内的容器共享网络、存储等资源；</p>
</li>
<li data-nodeid="301696">
<p data-nodeid="301697"><strong data-nodeid="301890">Docker</strong>，是 Pod 中最常见的容器 runtime，Pod 也支持其他容器 runtime；</p>
</li>
<li data-nodeid="301698">
<p data-nodeid="301699"><strong data-nodeid="301895">Kubelet</strong>，负责维护调度到它所在 Node 节点的 Pod 的生命周期，包括创建、修改、删除和监控等；</p>
</li>
<li data-nodeid="301700">
<p data-nodeid="301701"><strong data-nodeid="301900">Kube-proxy</strong>，负责为 Pod 提供代理，为 Service 提供集群内部的服务发现和负载均衡，Service 可以看作一组提供相同服务的 Pod 的对外访问接口。</p>
</li>
</ul>
<h3 data-nodeid="301702">etcd 在 Kubernetes 中的部署</h3>
<p data-nodeid="301703">前面我们介绍了 Kubernetes 的基本架构，在 Kubernetes 集群中，etcd 有两种部署方式，一种是 etcd 实例可以<strong data-nodeid="301907">作为 Pod 部署在 Master 节点上</strong>，如下图所示：</p>
<p data-nodeid="305501" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/C7/Cgp9HWBm9BiALFB3AAKN-4rjhns935.png" alt="Drawing 1.png" data-nodeid="305504"></p>

<p data-nodeid="314010" class="">etcd 作为 Pod 部署的方式（图片<a href="https://betterprogramming.pub/a-closer-look-at-etcd-the-brain-of-a-kubernetes-cluster-788c8ea759a5?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="314014">来源</a>）</p>











<p data-nodeid="301706">另一种是将 etcd<strong data-nodeid="301921">部署在集群外部</strong>，用以增加可靠性和安全性，如下图所示：</p>
<p data-nodeid="314862" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/D0/CioPOWBm9DqABTi2AAE9o4GAgqY183.png" alt="Drawing 2.png" data-nodeid="314865"></p>

<p data-nodeid="301708">etcd 集群单独部署（图片<a href="https://betterprogramming.pub/a-closer-look-at-etcd-the-brain-of-a-kubernetes-cluster-788c8ea759a5?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="301928">来源</a>）</p>
<h3 data-nodeid="301709">Kubernetes 部署 user 服务</h3>
<p data-nodeid="301710">接下来我们创建 Pod 部署 user 服务以及它依赖的 Redis 和 MySQL 数据库，通过这个例子熟悉 Kubernetes 的基本用法。我们可以基于 YAML 文件描述配置过程，使用 kubectl 命令行工具访问 Kubernetes 的接口。user 服务的 Pod 描述如下：</p>
<pre class="lang-yaml" data-nodeid="301711"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Pod</span>
<span class="hljs-attr">metadata:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">labels:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">spec:</span>
<span class="hljs-attr">containers:</span>                    <span class="hljs-comment"># 定义 user 容器，开放 10086 端口</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">user</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">user</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">10086</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">mysql</span>                     <span class="hljs-comment"># 定义 MySQL 容器，开放 3306 端口</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">mysql-for-user</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">3306</span>
<span class="hljs-attr">env:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">MYSQL_ROOT_PASSWORD</span>
<span class="hljs-attr">value:</span> <span class="hljs-string">"123456"</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">redis</span>                     <span class="hljs-comment"># 定义 Redis 容器，开放 6379 端口</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">redis:5.0</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">6379</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
</code></pre>
<p data-nodeid="301712">上述 YAML 文件中，展示了部署 user、MySQL 和 Redis 3 个容器应用的简单 Pod。由于在同一个 Pod 中的多个容器是<strong data-nodeid="301937">并发启动</strong>的，为了保证 user 服务启动时 Redis 和 MySQL 数据库已经部署启动完成，在 user 服务的 main 函数中增加了 time.Sleep，延迟了 user 服务的启动。通过 kubectl create 命令和 YAML 描述启动 Pod。命令如下所示：</p>
<pre class="lang-java" data-nodeid="301713"><code data-language="java">kubectl create -f user-service.yaml
</code></pre>
<p data-nodeid="301714">此操作将在 Kubernetes 集群的 Node 节点中创建单个 Pod。通过以下两个命令我们可以查看 user-service Pod 的信息并进入到 Pod 中：</p>
<pre class="lang-java" data-nodeid="301715"><code data-language="java">kubectl get pod user-service
kubectl exec -ti user-service -n <span class="hljs-keyword">default</span>  -- /bin/bash
</code></pre>
<p data-nodeid="301716">如上创建 Pod 的操作步骤一般会产生五个事件，按照时间顺序梳理如下：</p>
<ul data-nodeid="301717">
<li data-nodeid="301718">
<p data-nodeid="301719">调度&nbsp;user-service 到对应的 Node；</p>
</li>
<li data-nodeid="301720">
<p data-nodeid="301721">拉取镜像 "user:latest"；</p>
</li>
<li data-nodeid="301722">
<p data-nodeid="301723">成功拉取镜像 ；</p>
</li>
<li data-nodeid="301724">
<p data-nodeid="301725">创建容器&nbsp;user；</p>
</li>
<li data-nodeid="301726">
<p data-nodeid="301727">启动&nbsp;Started container user。</p>
</li>
</ul>
<p data-nodeid="301728">单个 Pod 不具备自我恢复的能力，当 Pod 所在的 Node 出现问题时，Pod 很可能被删除，这就会导致 Pod 中容器提供的服务被终止。为了避免这种情况的发生，可以使用<strong data-nodeid="301954">Controller</strong>管理 Pod，Controller 提供创建和管理多个 Pod 的能力，帮助被管理的 Pod 自愈和更新。常见的 Controller 有以下几种：</p>
<ul data-nodeid="301729">
<li data-nodeid="301730">
<p data-nodeid="301731"><strong data-nodeid="301959">Replication Controller</strong>，确保用户定义的 Pod 副本数保持不变；</p>
</li>
<li data-nodeid="301732">
<p data-nodeid="301733"><strong data-nodeid="301964">ReplicaSet</strong>，是 RC 的升级版，在选择器（Selector）的支持上优于 RC，RC 只支持基于等式的选择器，但 RS 还支持基于集合的选择器；</p>
</li>
<li data-nodeid="301734">
<p data-nodeid="301735"><strong data-nodeid="301969">Deployment</strong>，在 RS 的基础上提供了 Pod 的更新能力，在 Deployment 配置文件中 Pod template 发生变化时，它能将现在集群的状态逐步更新成 Deployment 中定义的目标状态；</p>
</li>
<li data-nodeid="301736">
<p data-nodeid="301737"><strong data-nodeid="301974">StatefulSets</strong>，其中的 Pod 是有序部署且具备稳定的标识，是一组存在状态的 Pod 副本。</p>
</li>
</ul>
<p data-nodeid="301738">比如我们可以使用 DeploymentController 为我们管理 user-service Pod，配置如下：</p>
<pre class="lang-yaml" data-nodeid="301739"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span>
<span class="hljs-attr">metadata:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">labels:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">spec:</span>
<span class="hljs-attr">replicas:</span> <span class="hljs-number">3</span>
<span class="hljs-attr">selector:</span>
<span class="hljs-attr">matchLabels:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">template:</span>
<span class="hljs-attr">metadata:</span>
<span class="hljs-attr">labels:</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span>
<span class="hljs-attr">spec:</span>
<span class="hljs-attr">containers:</span>                    <span class="hljs-comment"># 定义 user 容器，开放 10086 端口</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">user</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">user</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">10086</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">mysql</span>                     <span class="hljs-comment"># 定义 MySQL 容器，开放 3306 端口</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">mysql-for-user</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">3306</span>
<span class="hljs-attr">env:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">MYSQL_ROOT_PASSWORD</span>
<span class="hljs-attr">value:</span> <span class="hljs-string">"123456"</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">redis</span>                     <span class="hljs-comment"># 定义 Redis 容器，开放 6379 端口</span>
<span class="hljs-attr">image:</span> <span class="hljs-string">redis:5.0</span>
<span class="hljs-attr">ports:</span>
<span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">6379</span>
<span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span>
</code></pre>
<p data-nodeid="301740">在上述配置中，我们指定了 kind 的类型为 Deployment，副本的数量为 3，选择器为匹配标签 name: user-service。可以发现原来 Pod 的配置放到了 template 标签下，并添加 name: user-service 的标签。Deployment Controller 将会使用 template 下的 Pod 配置来创建 Pod 副本，并通过标签选择器监控 Pod 副本的数量。当副本数不足时，将会根据 template 创建 Pod。</p>
<p data-nodeid="301741">执行以下命令即可通过 Deployment Controller 管理 user-service Pod，命令如下：</p>
<pre class="lang-java" data-nodeid="301742"><code data-language="java">kubectl create -f user-service-deployment.yaml
</code></pre>
<p data-nodeid="301743">可以通过  kubectl get Deployment 命令查看 user-service 的 Pod 副本状态，如下所示：</p>
<pre class="lang-java" data-nodeid="301744"><code data-language="java">kubectl get Deployment user-service
</code></pre>
<p data-nodeid="301745">Deployment Controller 默认使用 RollingUpdate 策略更新 Pod，也就是<strong data-nodeid="301988">滚动更新</strong>的方式；另一种更新策略是<strong data-nodeid="301989">Recreate</strong>，创建出新的 Pod 之前会先杀掉所有已存在的 Pod，可以通过 spec.strategy.type 标签指定更新策略。当且仅当 Deployment 的 Pod template 中的 label 更新或者镜像更改时，Deployment 的 rollout 将被触发。比如我们希望更新 Redis 的版本：</p>
<pre class="lang-java" data-nodeid="301746"><code data-language="java">kubectl set image deployment/user-service redis=redis:<span class="hljs-number">6.0</span>
</code></pre>
<p data-nodeid="301747">这将触发 user-service Pod 的<strong data-nodeid="301995">重新部署</strong>。当 Pod 被 Deployment Controller 管理时，单独使用 kubectl delete pod 无法删除相关 Pod，Deployment Controller 会维持 Pod 副本数量不变，这时需要通过 kubectl delete Deployment 删除相关 Deployment 配置，比如删除 user-service 的 Deployment 配置，如下命令所示：</p>
<pre class="lang-java" data-nodeid="301748"><code data-language="java">kubectl delete Deployment user-service
</code></pre>
<h3 data-nodeid="301749">创建 Pod 流程分析</h3>
<p data-nodeid="301750">通过前面的 Kubernetes 集群架构图，可以发现各组件都通过 API Server 实现数据交互，且依赖 API Server 提供的资源变化监听机制。而 API Server 对外提供的监听机制，则是由 etcd watch 提供的底层支持。</p>
<p data-nodeid="301751">为了更好地理解创建 Pod 的过程，我绘制了下面这张时序图：</p>
<p data-nodeid="316557" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/C7/Cgp9HWBm9EiACq-OAABI1H01pJ0565.png" alt="Drawing 3.png" data-nodeid="316561"></p>
<div data-nodeid="316558"><p style="text-align:center">创建 Pod 的流程图</p></div>



<p data-nodeid="301754">该时序图展示了创建 Pod 的流程，基本流程描述如下。</p>
<ol data-nodeid="301755">
<li data-nodeid="301756">
<p data-nodeid="301757">用户提交创建 Pod 的请求，可以通过 API Server 的 REST API ，也可用 kubectl 命令行工具，支持 JSON 和 YAML 两种格式。</p>
</li>
<li data-nodeid="301758">
<p data-nodeid="301759">API Server 处理用户请求，存储 Pod 数据到 etcd。</p>
</li>
<li data-nodeid="301760">
<p data-nodeid="301761">调度器通过 API Server 的 watch 机制检测到新的 Pod 之后，尝试为 Pod 绑定 Node。</p>
</li>
<li data-nodeid="301762">
<p data-nodeid="301763">调度器根据规则过滤掉不符合要求的主机。比如 Pod 指定了所需要的资源，就要过滤掉资源不够的主机。</p>
</li>
<li data-nodeid="301764">
<p data-nodeid="301765">调度器根据整体优化的策略，比如把一个 Replication Controller 的副本分布到不同的主机上，来选择最低负载的主机。</p>
</li>
<li data-nodeid="301766">
<p data-nodeid="301767">根据选定的主机，进行 Pod 绑定操作，并将结果存储到 etcd 中。</p>
</li>
<li data-nodeid="301768">
<p data-nodeid="301769">Kubelet 根据调度结果执行 Pod 创建操作。Pod 绑定成功后，会执行 docker run 命令启动容器。Scheduler 调用 API Server 的 API 在 etcd 中创建一个 bound Pod 对象，描述在一个工作节点上绑定运行的所有 Pod 信息。运行在每个工作节点上的 Kubelet 也会定期与 etcd 同步 bound Pod 信息，一旦发现应该在该工作节点上运行的 bound Pod 对象没有更新，则调用 Docker API 创建并启动 Pod 内的容器。</p>
</li>
</ol>
<p data-nodeid="301770">总的来说，用户通过 API Server 创建 Pod，然后 API Server 将其写入etcd。调度器 watch 到一个“未绑定”的 Pod，会决定在哪个节点上运行该 Pod，随后将绑定信息写回到 API Server。Kubelet  watch 绑定到其节点上 Pod 的更改事件，并通过 Docker 运行容器。Kubelet 通过 docker runtime 监视 Pod 的状态。当出现变更事件时，Kubelet 会将当前状态的变更反馈给 API Server。</p>
<p data-nodeid="301771">下面我们具体分析创建 Pod 所涉及的核心流程。</p>
<h4 data-nodeid="301772">etcd 如何存储 Kubernetes 的数据</h4>
<p data-nodeid="301773">我们首先来看 etcd 如何存储 Kubernetes 集群中的元数据。通过如下的命令，你可以获取 Kubernetes 存储在 etcd 中的 keys。</p>
<pre class="lang-java" data-nodeid="301774"><code data-language="java">$ etcdctl --endpoints=https://192.168.10.124:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key get / --prefix --keys-only  |grep -Ev "^$"
# 得到的部分结果
/registry/apiregistration.k8s.io/apiservices/v1.
/registry/apiregistration.k8s.io/apiservices/v1.apps
/registry/apiregistration.k8s.io/apiservices/v1.authentication.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.autoscaling
/registry/apiregistration.k8s.io/apiservices/v1.batch
/registry/apiregistration.k8s.io/apiservices/v1.coordination.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.networking.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.rbac.authorization.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.scheduling.k8s.io
/registry/apiregistration.k8s.io/apiservices/v1.storage.k8s.io
/registry/secrets/kube-system/kubernetes-dashboard-certs
/registry/secrets/kube-system/kubernetes-dashboard-key-holder
/registry/secrets/kube-system/kubernetes-dashboard-token-rtr49
/registry/secrets/kube-system/namespace-controller-token-prfx2
</code></pre>
<p data-nodeid="301775">由于结果很多，还有很多键的省略。但是通过上述结果可以知道，这些键定义了集群中所有资源的配置和状态：</p>
<ul data-nodeid="301776">
<li data-nodeid="301777">
<p data-nodeid="301778">Nodes</p>
</li>
<li data-nodeid="301779">
<p data-nodeid="301780">Namespaces</p>
</li>
<li data-nodeid="301781">
<p data-nodeid="301782">ServiceAccounts</p>
</li>
<li data-nodeid="301783">
<p data-nodeid="301784">Roles and RoleBindings, ClusterRoles / ClusterRoleBindings</p>
</li>
<li data-nodeid="301785">
<p data-nodeid="301786">ConfigMaps</p>
</li>
<li data-nodeid="301787">
<p data-nodeid="301788">Secrets</p>
</li>
<li data-nodeid="301789">
<p data-nodeid="301790">Workloads: Deployments, DaemonSets, Pods, …</p>
</li>
<li data-nodeid="301791">
<p data-nodeid="301792">Cluster's certificates</p>
</li>
<li data-nodeid="301793">
<p data-nodeid="301794">The resources within each apiVersion</p>
</li>
<li data-nodeid="301795">
<p data-nodeid="301796">The events that bring the cluster in the current state</p>
</li>
</ul>
<p data-nodeid="301797">元数据资源在 etcd 中的存储格式由前缀、资源类型、namespace 和具体资源名组成。你可以根据<strong data-nodeid="302037">具体资源名和 namespace</strong>查询存储在 etcd 中的元数据。API Server 提供了<code data-backticks="1" data-nodeid="302033">--etcd-prefix</code>配置项，用以自定义配置 etcd prefix，默认为<code data-backticks="1" data-nodeid="302035">registry</code>并支持将资源存储在多个 etcd 集群。</p>
<p data-nodeid="301798">除此之外，<code data-backticks="1" data-nodeid="302039">kubectl</code>支持使用标签查看 Pods 信息。比如我们执行<code data-backticks="1" data-nodeid="302041">kubectl get po -l app=user</code>命令，指定了标签查询，此时会向 etcd 发起一个遍历 default namespace 下的 Pods 操作。因此，当某个 namespace 中创建的 Pods 资源数量很大时，通过<code data-backticks="1" data-nodeid="302043">kubectl</code>使用标签频繁查询会影响 etcd 的性能。</p>
<h4 data-nodeid="301799">API Server 策略层的处理</h4>
<p data-nodeid="301800">再来看 API Server 如何将元数据写入 etcd。API Server 是一个策略组件，提供对 etcd 的访问控制。API Server 是 Kubernetes 中的核心协调组件，能够让 Kubernetes 以<strong data-nodeid="302051">松耦合</strong>的方式实现组件之间的交互。</p>
<p data-nodeid="301801">请求到达 API Server，首先会做相应的<strong data-nodeid="302057">校验</strong>，包括创建请求的合法性、权限校验等。主要包括：认证模块、限速模块、审计模块、授权模块以及控制模块。</p>
<p data-nodeid="301802">API Server 定义了一套 ORM 机制 ，实现了资源对象和 etcd 存储对象的映射、资源对象之间关联的关系。如某个 Pod 属于某个资源，资源对应具体的 Deployment，用 OwnerReference 字段定义所属的 object，同时定义对应的缓存机制、Callback 机制、验证机制等，实现了查询操作集合。</p>
<p data-nodeid="301803">创建 Pods 的请求经过准入校验后，由相应的资源逻辑进行处理。API Server 封装了资源创建、更新、删除的策略接口，新增一个资源只需要实现对应的策略。创建一个资源主要由 BeforeCreate、Storage.Create 以及 AfterCreate 三大步骤组成。</p>
<p data-nodeid="301804">Kubernetes 集群使用了<strong data-nodeid="302065">事务 Txn 接口</strong>防止并发创建、更新被覆盖等问题。当执行完 BeforeCreate 策略后，API Server 会调用 Storage 模块的 Create 接口写入资源。Storage.Create 接口调用底层存储模块 etcd3，将 user Deployment 资源对象写入 etcd。</p>
<h4 data-nodeid="301805">watch 机制</h4>
<p data-nodeid="301806">通过 Txn 接口成功将数据写入 etcd 后，kubectl create 命令执行完毕，返回给 client。通过上面的时序图可以知道，API Server 并没有任何逻辑去真正创建 Pod，Controller Manager 组件中的一系列<strong data-nodeid="302072">控制器</strong>将根据 watch 的结果进行后续的 Pod 创建、调度以及运行。</p>
<p data-nodeid="301807">Kubernetes 使用 watch 机制获取数据变化的事件，etcd watch 机制提供了<strong data-nodeid="302078">流式推送机制</strong>，相比于定时轮询减少了高昂的查询开销，方便 API Server 实现数据的监听。服务器端的 store 对象利用 etcd 的 watch 机制，当 watch 机制触发时，数据的变化信息将封装成 event 对象并打包发送出去，客户端则通过不停地监听尝试读取 event chunk。</p>
<p data-nodeid="301808">需要注意的是，Kubernetes 社区提供了通用的<strong data-nodeid="302084">Informer 组件</strong>，实现了客户端与 API Server 之间的资源和事件同步。Informer 机制的 Reflector 封装了 Watch、List 操作，结合本地 Cache、Indexer，控制器加载完初始状态数据后，接下来的其他操作只需从本地缓存读取，极大降低了 API Server 和 etcd 的压力。</p>
<h3 data-nodeid="301809">小结</h3>
<p data-nodeid="301810">这一讲，我们首先介绍了 Kubernetes 相关的概念及其架构。API Server 是 Kubernetes 的核心组件，也是唯一与 etcd 直接交互的组件。API Server 一个重要特性是<strong data-nodeid="302091">支持 watch 机制</strong>，使得 API Server 的客户端可以使用与 etcd 相同的协调模式。</p>
<p data-nodeid="301811">接着我们使用 Kubernetes 将 user 服务以及它的依赖服务的容器打包到同一个 Pod 中进行容器编排。通过部署 user 服务，相信你对 Kubernetes 的使用已经有了一个大概的了解。</p>
<p data-nodeid="301812">最后我们介绍了 Pod 创建过程，以及 etcd 在存储元数据中起到的作用，涉及 etcd 存储 Kubernetes 集群数据的形式、API Server 策略层的处理以及 watch 机制。Scheduler 监听到待调度的 Pod，最后完成分配 Node 以及绑定 IP 的过程。希望通过今天内容的学习，能够加深你对容器运行和编排的认识。</p>
<p data-nodeid="319084">本讲内容总结如下：</p>
<p data-nodeid="319085" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/2D/C7/Cgp9HWBm9FaARqT2AAFQlIIQ-Gk573.png" alt="Drawing 4.png" data-nodeid="319089"></p>



<p data-nodeid="301815">最后，你觉得还有哪些 etcd 在 Kubernetes 中的独特优势，欢迎在评论区与我分享。这一讲是我们专栏正文的最后一篇了，很高兴你能坚持学习到这里。在结束语中，除了对 etcd 在服务端架构中的展望，我也有一些在专栏写作期间的心得和你分享，希望你能坚持阅读，我们结束语再见。</p>

---

### 精选评论

##### *庚：
> listwatch是informer的核心

