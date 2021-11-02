<p data-nodeid="5474" class="">Docker 虽然在容器领域有着不可撼动的地位，然而在容器的编排领域，却有着另外一个事实标准，那就是 Kubernetes。本课时，我就带你一起来认识下 Kubernetes。</p>
<h3 data-nodeid="5475">Kubernetes 的前生今世</h3>
<p data-nodeid="5476">说起 Kubernetes，这一切还得从云计算这个词说起，云计算这个概念是 2006 年由 Google 提起的，近些年被提及的频率也越来越高。云计算从起初的概念演变为现在的 AWS、阿里云等实实在在的云产品（主要是虚拟机和相关的网络、存储服务），可见已经变得非常成熟和稳定。</p>
<p data-nodeid="5477">正当大家以为云计算领域已经变成了以虚拟机为代表的云平台时，Docker 在 2013 年横空出世，Docker 提出了镜像、仓库等核心概念，规范了服务的交付标准，使得复杂服务的落地变得更加简单，之后 Docker 又定义了 OCI 标准，可以说在容器领域 Docker 已经成了事实的标准。</p>
<p data-nodeid="5478">然而 Docker 诞生只是帮助我们定义了开发和交付标准，如果想要在生产环境中大批量的使用容器，还离不开的容器的编排技术。于是，在 2014 年 6 月 7 日，Kubernetes（Kubernetes 简称为 K8S，8 代表 ubernete 8个字母） 的第一个 commit（提交）拉开了容器编排标准定义的序幕。</p>
<p data-nodeid="5479">Kubernetes 是舵手的意思，我们把 Docker 比喻成一个个集装箱，而 Kubernetes 正是运输这些集装箱的舵手。早期的 Kubernetes 主要参考 Google 内部的 Borg 系统，Kubernetes 刚刚诞生时，提出了 Pod、Sidecar 等概念，这些都是 Google 内部多年实战和沉淀所积累的精华。经过将近一年的沉淀和积累，Kubernetes 于 2015 年 7 月 21 日对外发布了第一个正式版本 v1.0，正式走入了大众的视线。</p>
<p data-nodeid="5480">很荣幸，我也是在 2015 年下半年正式开始了 Kubernetes 和 Docker 的研发之路。时至今日，Kubernetes 经过 6 年的沉淀，已经成为了事实的编排技术标准。</p>
<p data-nodeid="5481">接下来，我们就看来看看，究竟是什么样的架构使得 Kubernetes 在容器编排领域成为了王者？</p>
<h3 data-nodeid="5482">Kubernetes 架构</h3>
<p data-nodeid="5483">Kubernetes 采用声明式 API 来工作，所有组件的运行过程都是异步的，整个工作过程大致为用户声明想要的状态，然后 Kubernetes 各个组件相互配合并且努力达到用户想要的状态。</p>
<p data-nodeid="5484">Kubernetes 采用典型的主从架构，分为 Master 和 Node 两个角色。</p>
<ul data-nodeid="5485">
<li data-nodeid="5486">
<p data-nodeid="5487">Mater 是 Kubernetes 集群的控制节点，负责整个集群的管理和控制功能。</p>
</li>
<li data-nodeid="5488">
<p data-nodeid="5489">Node 为工作节点，负责业务容器的生命周期管理。</p>
</li>
</ul>
<p data-nodeid="5490">整体架构如下图：</p>
<p data-nodeid="5491"><img src="https://s0.lgstatic.com/i/image/M00/68/D6/Ciqc1F-k_FqAdHbtAAFVTi8cyOE246.png" alt="image (1).png" data-nodeid="5607"></p>
<div data-nodeid="5492"><p style="text-align:center">图 1 Kubernetes 架构图（来源：Kubernetes 官网）</p></div>
<h4 data-nodeid="5493">Master 节点</h4>
<p data-nodeid="5494">Master 节点负责对集群中所有容器的调度，各种资源对象的控制，以及响应集群的所有请求。Master 节点包含三个重要的组件： kube-apiserver、kube-scheduler、kube-controller-manager。下面我对这三个组件逐一介绍。</p>
<ul data-nodeid="5495">
<li data-nodeid="5496">
<p data-nodeid="5497"><strong data-nodeid="5613">kube-apiserver</strong></p>
</li>
</ul>
<p data-nodeid="5498">kube-apiserver 主要负责提供 Kubernetes 的 API 服务，所有的组件都需要与 kube-apiserver 交互获取或者更新资源信息，它是 Kubernetes Master 中最前端组件。</p>
<p data-nodeid="5499">kube-apiserver 的所有数据都存储在 <a href="https://etcd.io/" data-nodeid="5618">etcd</a> 中，etcd 是一种采用 Go 语言编写的高可用 Key-Value 数据库，由 CoreOS 开发。etcd 虽然不是 Kubernetes 的组件，但是它在 Kubernetes 中却扮演着至关重要的角色，它是 Kubernetes 的数据大脑。可以说 etcd 的稳定性直接关系着 Kubernetes 集群的稳定性，因此生产环境中 etcd 一定要部署多个实例以确保集群的高可用。</p>
<ul data-nodeid="5500">
<li data-nodeid="5501">
<p data-nodeid="5502"><strong data-nodeid="5623">kube-scheduler</strong></p>
</li>
</ul>
<p data-nodeid="5503">kube-scheduler 用于监听未被调度的 Pod，然后根据一定调度策略将 Pod 调度到合适的 Node 节点上运行。</p>
<ul data-nodeid="5504">
<li data-nodeid="5505">
<p data-nodeid="5506"><strong data-nodeid="5628">kube-controller-manager</strong></p>
</li>
</ul>
<p data-nodeid="5507">kube-controller-manager 负责维护整个集群的状态和资源的管理。例如多个副本数量的保证，Pod 的滚动更新等。每种资源的控制器都是一个独立协程。kube-controller-manager 实际上是一系列资源控制器的总称。</p>
<blockquote data-nodeid="5508">
<p data-nodeid="5509">为了保证 Kubernetes 集群的高可用，Master 组件需要部署在多个节点上，由于 Kubernetes 所有数据都存在于 etcd 中，Etcd 是基于 Raft 协议实现，因此生产环境中 Master 通常建议至少三个节点（如果你想要更高的可用性，可以使用 5 个或者 7 个节点）。</p>
</blockquote>
<h4 data-nodeid="5510">Node 节点</h4>
<p data-nodeid="5511">Node 节点是 Kubernetes 的工作节点，负责运行业务容器。Node 节点主要包含两个组件 ：kubelet 和 kube-proxy。</p>
<ul data-nodeid="5512">
<li data-nodeid="5513">
<p data-nodeid="5514"><strong data-nodeid="5636">kubelet</strong></p>
</li>
</ul>
<p data-nodeid="5515">Kubelet 是在每个工作节点运行的代理，它负责管理容器的生命周期。Kubelet 通过监听分配到自己运行的主机上的 Pod 对象，确保这些 Pod 处于运行状态，并且负责定期检查 Pod 的运行状态，将 Pod 的运行状态更新到 Pod 对象中。</p>
<ul data-nodeid="5516">
<li data-nodeid="5517">
<p data-nodeid="5518"><strong data-nodeid="5641">kube-proxy</strong></p>
</li>
</ul>
<p data-nodeid="5519">Kube-proxy 是在每个工作节点的网络插件，它实现了 Kubernetes 的 <a href="https://kubernetes.io/docs/concepts/services-networking/service/" data-nodeid="5645">Service</a> 的概念。Kube-proxy 通过维护集群上的网络规则，实现集群内部可以通过负载均衡的方式访问到后端的容器。</p>
<p data-nodeid="5520">Kubernetes 的成功不仅得益于其优秀的架构设计，更加重要的是 Kubernetes 提出了很多核心的概念，这些核心概念构成了容器编排的主要模型。</p>
<h3 data-nodeid="5521">Kubernetes 核心概念</h3>
<p data-nodeid="5522">Kubernetes 这些概念是 Google 多年的技术沉淀和积累，理解 Kubernetes 的核心概念有助于我们更好的理解 Kubernetes 的设计理念。</p>
<h4 data-nodeid="5523">（1）集群</h4>
<p data-nodeid="5524">集群是一组被 Kubernetes 统一管理和调度的节点，被 Kubernetes 纳管的节点可以是物理机或者虚拟机。集群其中一部分节点作为 Master 节点，负责集群状态的管理和协调，另一部分作为 Node 节点，负责执行具体的任务，实现用户服务的启停等功能。</p>
<h4 data-nodeid="5525">（2）标签（Label）</h4>
<p data-nodeid="5526">Label 是一组键值对，每一个资源对象都会拥有此字段。Kubernetes 中使用 Label 对资源进行标记，然后根据 Label 对资源进行分类和筛选。</p>
<h4 data-nodeid="5527">（3）命名空间（Namespace）</h4>
<p data-nodeid="5528">Kubernetes 中通过命名空间来实现资源的虚拟化隔离，将一组相关联的资源放到同一个命名空间内，避免不同租户的资源发生命名冲突，从逻辑上实现了多租户的资源隔离。</p>
<h4 data-nodeid="5529">（4）容器组（Pod）</h4>
<p data-nodeid="5530">Pod 是 Kubernetes 中的最小调度单位，它由一个或多个容器组成，一个 Pod 内的容器共享相同的网络命名空间和存储卷。Pod 是真正的业务进程的载体，在 Pod 运行前，Kubernetes 会先启动一个 Pause 容器开辟一个网络命名空间，完成网络和存储相关资源的初始化，然后再运行业务容器。</p>
<h4 data-nodeid="5531">（5）部署（Deployment）</h4>
<p data-nodeid="5532">Deployment 是一组 Pod 的抽象，通过 Deployment 控制器保障用户指定数量的容器副本正常运行，并且实现了滚动更新等高级功能，当我们需要更新业务版本时，Deployment 会按照我们指定策略自动的杀死旧版本的 Pod 并且启动新版本的 Pod。</p>
<h4 data-nodeid="5533">（6）状态副本集（StatefulSet）</h4>
<p data-nodeid="5534">StatefulSet 和 Deployment 类似，也是一组 Pod 的抽象，但是 StatefulSet 主要用于有状态应用的管理，StatefulSet 生成的 Pod 名称是固定且有序的，确保每个 Pod 独一无二的身份标识。</p>
<h4 data-nodeid="5535">（7）守护进程集（DaemonSet）</h4>
<p data-nodeid="5536">DaemonSet 确保每个 Node 节点上运行一个 Pod，当我们集群有新加入的 Node 节点时，Kubernetes 会自动帮助我们在新的节点上运行一个 Pod。一般用于日志采集，节点监控等场景。</p>
<h4 data-nodeid="5537">（8）任务（Job）</h4>
<p data-nodeid="5538">Job 可以帮助我们创建一个 Pod 并且保证 Pod 的正常退出，如果 Pod 运行过程中出现了错误，Job 控制器可以帮助我们创建新的 Pod，直到 Pod 执行成功或者达到指定重试次数。</p>
<h4 data-nodeid="5539">（9）服务（Service）</h4>
<p data-nodeid="5540">Service 是一组 Pod 访问配置的抽象。由于 Pod 的地址是动态变化的，我们不能直接通过 Pod 的 IP 去访问某个服务，Service 通过在主机上配置一定的网络规则，帮助我们实现通过一个固定的地址访问一组 Pod。</p>
<h4 data-nodeid="5541">（10）配置集（ConfigMap）</h4>
<p data-nodeid="5542">ConfigMap 用于存放我们业务的配置信息，使用 Key-Value 的方式存放于 Kubernetes 中，使用 ConfigMap 可以帮助我们将配置数据和应用程序代码分开。</p>
<h4 data-nodeid="5543">（11）加密字典（Secret）</h4>
<p data-nodeid="5544">Secret 用于存放我们业务的敏感配置信息，类似于 ConfigMap，使用 Key-Value 的方式存在于 Kubernetes 中，主要用于存放密码和证书等敏感信息。</p>
<p data-nodeid="5545">了解完 Kubernetes 的架构和核心概念，你是不是已经迫不及待地想要体验下了。下面就让我们动手安装一个 Kubernetes 集群，来体验下 Kubernetes 的强大之处吧。</p>
<h3 data-nodeid="5546">安装 Kubernetes</h3>
<p data-nodeid="5547">Kubernetes 目前已经支持在多种环境下安装，我们可以在公有云，私有云，甚至裸金属中安装 Kubernetes。下面，我们通过 minikube 来演示一下如何快速安装和启动一个 Kubernetes 集群，minikube 是官方提供的一个快速搭建本地 Kubernetes 集群的工具，主要用于本地开发和调试。</p>
<p data-nodeid="5548">下面，我以 Linux 平台为例，演示一下如何使用 minikube 安装一个 Kubernetes 集群。</p>
<blockquote data-nodeid="5549">
<p data-nodeid="5550">如果你想要在其他平台使用 minikube 安装 Kubernetes，请参考官网<a href="https://minikube.sigs.k8s.io/docs/start/" data-nodeid="5679">安装教程</a>。<br>
在使用 minikube 安装 Kubernetes 之前，请确保我们的机器已经正确安装并且启动 Docker。</p>
</blockquote>
<p data-nodeid="5551">第一步，安装 minikube 和 kubectl。首先执行以下命令安装 minikube。</p>
<pre class="lang-java" data-nodeid="5552"><code data-language="java">$ curl -LO https:<span class="hljs-comment">//github.com/kubernetes/minikube/releases/download/v1.13.1/minikube-linux-amd64</span>
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
</code></pre>
<p data-nodeid="5553">Kubectl 是 Kubernetes 官方的命令行工具，可以实现对 Kubernetes 集群的管理和控制。<br>
我们使用以下命令来安装 kubectl：</p>
<pre class="lang-java" data-nodeid="5554"><code data-language="java">$ curl -LO https:<span class="hljs-comment">//dl.k8s.io/v1.19.2/kubernetes-client-linux-amd64.tar.gz</span>
$ tar -xvf kubernetes-client-linux-amd64.tar.gz
kubernetes/
kubernetes/client/
kubernetes/client/bin/
kubernetes/client/bin/kubectl
$ sudo install kubernetes/client/bin/kubectl /usr/local/bin/kubectl
</code></pre>
<p data-nodeid="5555">第二步，安装 Kubernetes 集群。<br>
执行以下命令使用 minikube 安装 Kubernetes 集群：</p>
<pre data-nodeid="5556"><code>$ minikube start
</code></pre>
<p data-nodeid="5557">执行完上述命令后，minikube 会自动帮助我们创建并启动一个 Kubernetes 集群。命令输出如下，当命令行输出 Done 时，代表集群已经部署完成。</p>
<p data-nodeid="5558"><img src="https://s0.lgstatic.com/i/image/M00/68/FE/CgqCHl-lL_WABqFRAAE7sPUop9w125.png" alt="111.png" data-nodeid="5693"></p>
<p data-nodeid="5559">第三步，检查集群状态。集群安装成功后，我们可以使用以下命令检查 Kubernetes 集群是否成功启动。</p>
<pre class="lang-java" data-nodeid="5560"><code data-language="java">$ kubectl cluster-info
Kubernetes master is running at https:<span class="hljs-comment">//172.17.0.3:8443</span>
KubeDNS is running at https:<span class="hljs-comment">//172.17.0.3:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy</span>

To further debug and diagnose cluster problems, use <span class="hljs-string">'kubectl cluster-info dump'</span>.
</code></pre>
<p data-nodeid="5561">执行<code data-backticks="1" data-nodeid="5696">kubectl cluster-info</code>命令后，输出 "Kubernetes master is running" 表示我们的集群已经成功运行。</p>
<blockquote data-nodeid="5562">
<p data-nodeid="5563">172.17.0.3 为演示环境机器的 IP 地址，这个 IP 会根据你的实际 IP 地址而变化。</p>
</blockquote>
<h3 data-nodeid="5564">创建第一个应用</h3>
<p data-nodeid="5565">集群搭建好后，下面我们来试着使用 Kubernetes 来创建我们的第一个应用。</p>
<p data-nodeid="5566">这里我们使用 Deployment 来定义应用的部署信息，使用 Service 暴露我们的应用到集群外部，从而使得我们的应用可以从外部访问到。</p>
<p data-nodeid="5567">第一步，创建 deployment.yaml 文件，并且定义启动的副本数（replicas）为 3。</p>
<pre class="lang-yaml" data-nodeid="5568"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span>
<span class="hljs-attr">metadata:</span>
&nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">hello-world</span>
<span class="hljs-attr">spec:</span>
&nbsp; <span class="hljs-attr">replicas:</span> <span class="hljs-number">3</span>
&nbsp; <span class="hljs-attr">selector:</span>
&nbsp; &nbsp; <span class="hljs-attr">matchLabels:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">hello-world</span>
&nbsp; <span class="hljs-attr">template:</span>
&nbsp; &nbsp; <span class="hljs-attr">metadata:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">hello-world</span>
&nbsp; &nbsp; <span class="hljs-attr">spec:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">containers:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">hello-world</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">image:</span> <span class="hljs-string">wilhelmguo/nginx-hello:v1</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">ports:</span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="5569">第二步，发布部署文件到 Kubernetes 集群中。</p>
<pre class="lang-java" data-nodeid="5570"><code data-language="java">$ kubectl create -f deployment.yaml
</code></pre>
<p data-nodeid="5571">部署发布完成后，我们可以使用 kubectl 来查看一下 Pod 是否被成功启动。</p>
<pre class="lang-java" data-nodeid="5572"><code data-language="java">$ kubectl get pod -o wide
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE&nbsp; &nbsp; &nbsp;IP&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;NODE&nbsp; &nbsp; &nbsp; &nbsp;NOMINATED NODE&nbsp; &nbsp;READINESS GATES
hello-world-<span class="hljs-number">57968f</span>9979-xbmzt&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">3</span>m19s&nbsp; &nbsp;<span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.7</span>&nbsp; &nbsp;minikube&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
hello-world-<span class="hljs-number">57968f</span>9979-xq5w4&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">3</span>m18s&nbsp; &nbsp;<span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.5</span>&nbsp; &nbsp;minikube&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
hello-world-<span class="hljs-number">57968f</span>9979-zwvgg&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">4</span>m14s&nbsp; &nbsp;<span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.6</span>&nbsp; &nbsp;minikube&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
</code></pre>
<p data-nodeid="5573">这里可以看到 Kubernetes 帮助我们创建了 3 个 Pod 实例。</p>
<p data-nodeid="5574">第三步，创建 service.yaml 文件，帮助我们将服务暴露出去，内容如下：</p>
<pre class="lang-yaml" data-nodeid="5575"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Service</span>
<span class="hljs-attr">metadata:</span>
&nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">hello-world</span>
<span class="hljs-attr">spec:</span>
&nbsp; <span class="hljs-attr">type:</span> <span class="hljs-string">NodePort</span>
&nbsp; <span class="hljs-attr">ports:</span>
&nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
&nbsp; &nbsp; <span class="hljs-attr">targetPort:</span> <span class="hljs-number">80</span>
&nbsp; <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">app:</span> <span class="hljs-string">hello-world</span>
</code></pre>
<p data-nodeid="8290">然后执行如下命令在 Kubernetes 中创建 Service：</p>
<pre data-nodeid="9065" class="te-preview-highlight"><code>kubectl create -f service.yaml
</code></pre>

<p data-nodeid="8807" class="">服务创建完成后，Kubernetes 会随机帮助我们分配一个外部访问端口，可以通过以下命令查看服务信息：</p>








<pre class="lang-java" data-nodeid="5577"><code data-language="java">$ kubectl&nbsp; get service -o wide
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TYPE&nbsp; &nbsp; &nbsp; &nbsp; CLUSTER-IP&nbsp; &nbsp; &nbsp;EXTERNAL-<span class="hljs-function">IP&nbsp; &nbsp;<span class="hljs-title">PORT</span><span class="hljs-params">(S)</span>&nbsp; &nbsp; &nbsp; &nbsp; AGE&nbsp; &nbsp;SELECTOR
hello-world&nbsp; &nbsp;NodePort&nbsp; &nbsp; 10.101.83.18&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; 80:32391/TCP&nbsp; &nbsp;12s&nbsp; &nbsp;app</span>=hello-world
kubernetes&nbsp; &nbsp; ClusterIP&nbsp; &nbsp;<span class="hljs-number">10.96</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>&nbsp; &nbsp; &nbsp; &lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">443</span>/TCP&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">40</span>m&nbsp; &nbsp;&lt;none&gt;
</code></pre>
<p data-nodeid="5578">由于我们的集群使用 minikube 安装，要想集群中的服务可以通过外部访问，还需要执行以下命令：</p>
<pre data-nodeid="5579" class=""><code>$ minikube service hello-world
</code></pre>
<p data-nodeid="5580">输出如下：</p>
<p data-nodeid="5581"><img src="https://s0.lgstatic.com/i/image/M00/68/D8/Ciqc1F-k_seAeN4RAACePALnr0Q662.png" alt="Lark20201106-154358.png" data-nodeid="5716"></p>
<p data-nodeid="5582">可以看到 minikube 将我们的服务暴露在了 32391 端口上，我们通过 http://{YOUR-IP}:32391 可以访问到我们启动的服务，如下图所示。</p>
<p data-nodeid="5583"><img src="https://s0.lgstatic.com/i/image/M00/68/D6/Ciqc1F-k_J-AWWQyAABkHB5NA0A837.png" alt="image (2).png" data-nodeid="5720"></p>
<div data-nodeid="5584"><p style="text-align:center">图 2 服务请求结果</p></div>
<p data-nodeid="5585">总结下，我们首先使用 Deployment 创建了三个 nginx-hello 的实例，然后使用 Service 的方式随机负载到后端的三个实例，并将服务通过 NodePort 的方式暴露在主机上，使得我们可以直接使用主机的端口访问到容器中的服务。</p>
<h3 data-nodeid="5586">结语</h3>
<p data-nodeid="5587">Kubernetes 从诞生到现在已经经历了 6 个年头，起初由于它的超前理念被世人误认为设计过度复杂，使得 Kubernetes 的入门门槛非常高。然而 6 年后的今天， Kubernetes 已经拥有了非常完善的社区和工具集，它可以帮助我们一键搭建 Kubernetes 集群，并且围绕 Kubernetes 构建的各种应用也是越来越丰富。</p>
<p data-nodeid="5588">Kubernetes 的目标一直很明确，那就是对标 Borg，可以支撑数亿容器的运行。目前来看，要达到这个目标，Kubernetes 还有很长的路要走，但是当我们谈及云原生，谈及容器云时都必然会提到 Kubernetes，显然它已经成为容器编排的标准和标杆，目前大多数公有云也有支持 Kubernetes。容器的未来一定是美好的，而使用 Kubernetes 来调度容器则更是未来云计算的一个重要风向标。</p>
<p data-nodeid="5589">那么，你的朋友中有没有人从事过 Kubernetes 或 Docker 相关的项目研发，现在这些项目发展得怎么样了呢？欢迎留言和我一起讨论容器圈创业那点事。</p>
<p data-nodeid="5590" class="">下一课时，我将为你带来 Docker 的综合实战案例，Docker 下如何实现镜像多阶级构建？</p>

---

### 精选评论

##### **8621：
> 打卡

##### *锋：
> 在阿里云ecs上实验minikube，执行minikuber start时，出现这个错误：————————————————————————————————————————————————[root@iZuf6afa9xpt8kqver67p6Z ~]# minikube start* minikube v1.13.1 on Centos 7.3.1611* Unable to pick a default driver. Here is what was considered, in preference order: - kvm2: Not installed: exec: "virsh": executable file not found in $PATH - podman: Not installed: exec: "podman": executable file not found in $PATH - virtualbox: Not installed: unable to find VBoxManage in $PATH - vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in $PATH - docker: Not healthy: "docker version --format {{.Server.Os}}-{{.Server.Version}}" exit status 1: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?X Exiting due to DRV_NOT_DETECTED: No possible driver was detected. Try specifying --driver, or see https://minikube.sigs.k8s.io/docs/start/

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是 minikube 默认使用 kvm 的方式启动集群，这是因为没有安装 kvm，需要先安装 kvm 或者使用容器模式启动。

##### *冲：
> minikube service list|----------------------|---------------------------|--------------|-------------------------||  ||----------------------|---------------------------|--------------|-------------------------|| default  80 | http://172.17.0.3:30535 || default | No node port || kube-system | No node port || kubernetes-dashboard | dashboard-metrics-scraper | No node port || kubernetes-dashboard | kubernetes-dashboard | No node port ||----------------------|---------------------------|--------------|-------------------------|我的hello-world服务开启来了,可是通过http://{YOUR-IP}:32391 可以访问到我们启动的服务,总是被拒绝curl 192.168.10.102:80curl: (7) Failed connect to 192.168.10.102:80; 拒绝连接[lc@slave2 ~]$ curl 192.168.10.102:30535curl: (7) Failed connect to 192.168.10.102:30535; 拒绝连接[lc@slave2 ~]$ curl localhost:80curl: (7) Failed connect to localhost:80; 拒绝连接[lc@slave2 ~]$ curl http://localhost:80curl: (7) Failed connect to localhost:80; 拒绝连接[lc@slave2 ~]$ curl http://localhost:30535curl: (7) Failed connect to localhost:30535; 拒绝连接[lc@slave2 ~]$ curl http://192.168.10.102:30535curl: (7) Failed connect to 192.168.10.102:30535; 拒绝连接防火墙我是关掉的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker volume 的实现实际上就是主机上的特定目录，通常在/var/lib/docker/volumes 目录下

##### *冲：
> kubectl get service -o wide执行这个命令的时候显示的是:NAME  SELECTORkubernetes 正好少了一行..执行 minikube service hello-worldminikube service hello-world

X Exiting due to SVC_NOT_FOUND: Service 'hello-world' was not found in 'default' namespace.
You may select another namespace by using 'minikube service hello-world -n '. Or list out all the services using 'minikube service list'
说没有这个服务列出所有服务minikube service list|-------------|------------|--------------|-----|| | URL ||-------------|------------|--------------|-----|| default  | kubernetes | No node port || kube-system | kube-dns  | No node port ||-------------|------------|--------------|-----|还是没有hello-world服务

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要执行 kubectl create -f service.yaml  命令先创建 service 哈

##### *冲：
> minikube start执行的时候报错* Centos 7.9.2009 上的 minikube v1.13.1* Unable to pick a default driver. Here is what was considered, in preference order: - none: Not healthy: the 'none' driver must be run as the root user - podman: Not installed: exec: "podman": executable file not found in $PATH - virtualbox: Not installed: unable to find VBoxManage in $PATH - vmware: Not installed: exec: "docker-machine-driver-vmware": executable file not found in $PATH - docker: Not healthy: "docker version --format {{.Server.Os}}-{{.Server.Version}}" exit status 1: Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get http://%2Fvar%2Frun%2Fdocker.sock/v1.24/version: dial unix /var/run/docker.sock: connect: permission denied - kvm2: Not installed: exec: "virsh": executable file not found in $PATHX Exiting due to DRV_NOT_DETECTED: No possible driver was detected. Try specifying --driver, or see https://minikube.sigs.k8s.io/docs/start/我的解决minikube start --driver=none* Centos 7.9.2009 上的 minikube v1.13.1* 根据用户配置使用 none 驱动程序X Exiting due to PROVIDER_NONE_NOT_RUNNING: the 'none' driver must be run as the root user* 建议：For non-root usage, try the newer 'docker' driver嘎minikube start --driver=docker

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以尝试切换到 root 用户执行 或者执行命令时添加 sudo

##### **伟：
> kubectl create -f deployment.yaml 报错error: error validating "deployment.yaml": error validating data: [ValidationError(Deployment): unknown field "\u00a0 name" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 replicas" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 \u00a0 \u00a0 - name" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 \u00a0 \u00a0 app" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 \u00a0 \u00a0 \u00a0 app" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 \u00a0 \u00a0 \u00a0 image" in io.k8s.api.apps.v1.Deployment, ValidationError(Deployment): unknown field "\u00a0 \u00a0 \u00a0 \u00a0 ports" in io.k8s.api.apps.v1.Deployment]; if you choose to ignore these errors, turn validation off with --validate=false

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是 yaml 文件缩进有问题，可以检查一下缩进

##### *冲：
> node节点那一章是不是写反了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有写反呀？请问是哪里有问题呢？

##### **卫：
> service.yml创建完成后，不用调用minikubel发布出去就可以通过端口访问服务了吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要调用 minikube 的，文章中有写哈，调用命令为 minikube service hello-world

##### **峰：
> 2015年我还在读书，连k8s都没听说过

