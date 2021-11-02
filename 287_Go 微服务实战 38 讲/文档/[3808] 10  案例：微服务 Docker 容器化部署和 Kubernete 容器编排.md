<p data-nodeid="1824" class="">今天我们主要来介绍微服务 Docker 容器化部署和 Kubernetes 容器编排的案例。</p>


<p data-nodeid="1028">微服务架构“分而治之”的手段将大型系统按业务分割为多个互相协作的微服务，每个微服务关注于自身业务职责，可独立开发、部署和维护，从而更好地应对频繁的需求变更和迭代。但是数量众多的微服务实例给运维带来了巨大的挑战，如果没有好的办法快速部署和启动微服务，那么微服务架构带来的好处将所剩无几。而<strong data-nodeid="1164">容器化</strong>和<strong data-nodeid="1165">容器编排</strong>的兴起正好填补了这个缺点。</p>
<p data-nodeid="1029">在上一课时中，我们运用 DDD 将货运平台应用划分了 4 个微服务进行开发，为了解决大量微服务实例部署的问题，在本课时我们将介绍 Docker 和 Kubernetes，并使用它们对 Go 应用进行容器化部署和容器编排。</p>
<h3 data-nodeid="2354" class="">Docker 简介</h3>

<p data-nodeid="1031">Docker 是当前最火热的开源应用容器引擎，没有之一。Docker 作为容器化技术，相比于虚拟化显得<strong data-nodeid="1181">更加轻量</strong>，它在最初受到用户追捧，正是由于它比虚拟化更加节省内存、启动更快。虚拟化在<strong data-nodeid="1182">硬件级别</strong>隔离应用，能够提供更好的资源隔离性，但资源利用率比较低；而 Docker 是在<strong data-nodeid="1183">操作系统</strong>上进行资源隔离，资源消耗低，能够快速启动，非常适合在一台主机上部署大量隔离环境的应用程序。</p>
<p data-nodeid="1032">Docker 中存在 3 个非常重要的核心概念：</p>
<ul data-nodeid="1033">
<li data-nodeid="1034">
<p data-nodeid="1035">镜像（Image）</p>
</li>
<li data-nodeid="1036">
<p data-nodeid="1037">容器（Container）</p>
</li>
<li data-nodeid="1038">
<p data-nodeid="1039">仓库（Repository）</p>
</li>
</ul>
<p data-nodeid="1040">Docker 能够将应用程序所需要的所有依赖、配置和环境变量打包成<strong data-nodeid="1193">镜像</strong>，为应用程序提供运行环境。 镜像是由多个镜像层叠加 而成的文件系统，其底层为 UnionFS 与 AUFS 文件联合系统， 这是一种分层、支持通过一层一层叠加的方式集成的轻量级且高性能文件系统。 镜像是一个只读的模板，我们可以在一个基础镜像上进行叠加，制作出多种多样不同的镜像。</p>
<p data-nodeid="1041"><strong data-nodeid="1198">容器</strong>通过镜像启动，是镜像的运行实例，一个镜像可以启动多个容器。容器之间相互隔离，它运行并隔离应用。容器可以被创建、运行、停止、删除、暂停和重启。我们可以简单将容器理解为为应用程序提供沙箱运行环境的 Linux 操作系统。</p>
<p data-nodeid="1042"><strong data-nodeid="1203">仓库</strong>是管理和存储镜像的地方，分为公有仓库和私有仓库。目前最常用的公有仓库为官方的 Docker Hub，其内提供了大量优质的官方镜像。但由于国内网络的问题，很难直接从官方的 Docker Hub 中拉取镜像，这种情况下我推荐你使用阿里云或者网易云的镜像仓库。不过使用阿里云的镜像仓库，需要你首先注册阿里云账号；但网易云的镜像仓库不需要你注册，可直接使用，地址为：http://hub-mirror.c.163.com 。</p>
<p data-nodeid="2884" class="">Docker 的镜像可以在任何安装 Docker 的 Linux 的机器上运行容器，达到“Build once，Run anywhere”（一次搭建、到处可用）的目的，大大降低了应用程序运行环境配置的难度，为大规模部署运行微服务提供了解决方案。</p>

<h3 data-nodeid="3414" class="">Docker 部署 user 服务</h3>

<p data-nodeid="1045">接下来我们将使用 Docker 对前面 08 课时中介绍的 user 服务进行部署和运行，至于 09 课时介绍的货运平台中的 4 个微服务的部署和运行也与此类似。</p>
<p data-nodeid="1046">user 服务依赖于 Redis 和 MySQL 数据库，我们先通过 Docker 部署好这两个底层数据依赖。通过 docker search 命令，可以从 Docker Hub 查找镜像，比如要查找 Redis 的镜像，就可以执行以下命令：</p>
<pre class="lang-powershell" data-nodeid="1047"><code data-language="powershell">docker search redis 
</code></pre>
<p data-nodeid="1048">其实还有更简单的方式，那就是从 https://hub.docker.com/r/library/ 网站上搜索相应的镜像版本并使用其提供命令拉取。使用 docker pull 命令可以将 Docker Hub 中的镜像拉取到本地，该命令可以指定拉取的镜像源和拉取版本，比如拉取 5.0 版本的 Redis 命令如下所示：</p>
<pre class="lang-powershell" data-nodeid="1049"><code data-language="powershell">docker pull redis:<span class="hljs-number">5.0</span> 
</code></pre>
<p data-nodeid="1050">拉取成功后，我们可以通过 docker images 命令查看本地的镜像列表，比如查看 Redis 的镜像列表命令如下：</p>
<pre class="lang-powershell" data-nodeid="1051"><code data-language="powershell">docker images redis 
</code></pre>
<p data-nodeid="1052">接下来我们就可以通过 docker run 命令启动以 redis:5.0 镜像为蓝本的容器，命令如下所示：</p>
<pre class="lang-java" data-nodeid="6589"><code data-language="java">docker run -itd --name redis-<span class="hljs-number">5.0</span> -p <span class="hljs-number">6379</span>:<span class="hljs-number">6379</span> redis:<span class="hljs-number">5.0</span> 
</code></pre>






<p data-nodeid="1054">上述命令中还出现了其他的选项，它们的意义如下：</p>
<ul data-nodeid="1055">
<li data-nodeid="1056">
<p data-nodeid="1057">-d 选项指定容器以后台方式运行，启动后返回容器的 ID；</p>
</li>
<li data-nodeid="1058">
<p data-nodeid="1059">-i 选项让容器的标准输入保持打开，而 -t 选项让 Docker 分配一个伪终端并绑定到容器的标准输入上，这两个选项一般配合使用；</p>
</li>
<li data-nodeid="1060">
<p data-nodeid="1061">--name 指令用以指定容器名，要求每个容器都不一样；</p>
</li>
<li data-nodeid="1062">
<p data-nodeid="1063">-p 指令将容器的 6379 端口映射到宿主机的 6379 端口，在外部我们可以直接通过宿主机6379 端口访问 Redis。</p>
</li>
</ul>
<p data-nodeid="1064">容器启动后，我们可以通过 docker ps 命令查看容器的运行情况，获取到容器 ID 和镜像等信息。然后再使用 docker exec 命令在容器中执行命令，比如我们可以在容器中启动一个 /bin/bash 交互式终端，从而进入到容器中执行各种命令，如下所示：</p>
<pre class="lang-java" data-nodeid="8705"><code data-language="java"> docker exec -it redis-<span class="hljs-number">5.0</span> /bin/bash 
</code></pre>




<p data-nodeid="1066">进入到 redis-5.0 容器之后，就可以通过 redis-cli 命令访问容器内的 Redis 数据库。除此之外，还有一些其他常用命令：docker stop 命令用于停止容器，docker kill 命令用于杀死容器，docker rm 命令用于移除容器，docker attach 命令用于连接运行中容器，等等。</p>
<p data-nodeid="1067">对于一般的 Docker 镜像，我们都可以直接从官方中拉取使用，比如上述的 Redis 镜像。但如果需要对镜像进行一些自定义操作，这时就需要借助 Dockerfile 描述镜像的构建过程，比如下面的 Dockerfile 将用于构建部署和运行 user 服务的 Go 镜像：</p>
<pre class="lang-dockerfile" data-nodeid="1068"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> golang:latest 
<span class="hljs-keyword">WORKDIR</span><span class="bash"> /root/micro-go-course/section10/user </span>
<span class="hljs-keyword">COPY</span><span class="bash"> / /root/micro-go-course/section10/user </span>
<span class="hljs-keyword">RUN</span><span class="bash"> go env -w GOPROXY=https://goproxy.cn,direct </span>
<span class="hljs-keyword">RUN</span><span class="bash"> go build -o user </span>
<span class="hljs-keyword">ENTRYPOINT</span><span class="bash"> [<span class="hljs-string">"./user"</span>] </span>
</code></pre>
<p data-nodeid="1069">在上面的 Dockerfile 中出现了五种指令。</p>
<ul data-nodeid="1070">
<li data-nodeid="1071">
<p data-nodeid="1072">From：Dockerfile 中必须出现的第一个指令，用于指定基础镜像，在上述例子中我们指定了基础镜像为 golang:latest 版本。</p>
</li>
<li data-nodeid="1073">
<p data-nodeid="1074">WORKDIR：指定工作目录，之后的命令将会在该目录下执行。</p>
</li>
<li data-nodeid="1075">
<p data-nodeid="1076">COPY：将本地文件添加到容器指定位置中。</p>
</li>
<li data-nodeid="1077">
<p data-nodeid="1078">RUN：创建镜像执行的命令，一个 Dockerfile 可以有多个 RUN 命令。在上述 RUN 指令中我们指定了 Go 的代理，并通过 go build 构建了 user 服务。</p>
</li>
<li data-nodeid="1079">
<p data-nodeid="1080">ENTRYPOINT：容器被启动后执行的命令，每个 Dockerfile 只有一个。我们通过该命令在容器启动后，又启动了 user 服务。</p>
</li>
</ul>
<p data-nodeid="1081">把 Dockerfile 放在 user 服务的代码下，即可通过以下命令构建一个带 user 服务的镜像：</p>
<pre class="lang-powershell" data-nodeid="1082"><code data-language="powershell">docker build <span class="hljs-literal">-t</span> user . 
</code></pre>
<p data-nodeid="1083">其中，-t 选项用于指定镜像的名称和标签，不指定标签默认为 latest；命令最后的 . 为 Dockerfile 所在的地址。</p>
<p data-nodeid="1084">我们同样可以使用 Dockerfile 为 user 服务定制一个已经初始化数据库的 MySQL 镜像，Dockerfile 如下所示：</p>
<pre class="lang-dockerfile" data-nodeid="1085"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> mysql:<span class="hljs-number">5.7</span> 
<span class="hljs-keyword">WORKDIR</span><span class="bash"> /docker-entrypoint-initdb.d </span>
<span class="hljs-keyword">ENV</span> LANG=C.UTF-<span class="hljs-number">8</span> 
<span class="hljs-keyword">COPY</span><span class="bash"> init.sql . </span>
</code></pre>
<p data-nodeid="1086">MySQL 的官方镜像支持在容器启动的时候自动执行指定的 sql 脚本或者 shell 脚本，只要在构建容器时将相关 sql 脚本或者 shell 脚本复制到 /docker-entrypoint-initdb.d 目录下即可。比如上述例子中，我们把初始化 init.sql 放到该目录下，让容器在启动时帮我们初始化好 user 数据库。</p>
<pre class="lang-powershell" data-nodeid="1087"><code data-language="powershell">docker build <span class="hljs-literal">-t</span> mysql<span class="hljs-literal">-for</span><span class="hljs-literal">-user</span> . 
docker run  <span class="hljs-literal">-itd</span> -<span class="hljs-literal">-name</span> mysql<span class="hljs-literal">-for</span><span class="hljs-literal">-user</span> <span class="hljs-literal">-p</span> <span class="hljs-number">3306</span>:<span class="hljs-number">3306</span> <span class="hljs-literal">-e</span> MYSQL_ROOT_PASSWORD=<span class="hljs-number">123456</span> mysql<span class="hljs-literal">-for</span><span class="hljs-literal">-user</span> 
</code></pre>
<p data-nodeid="1088">通过以上命令就构建和启动了 mysql-for-user 镜像，并指定 MySQL 的 root 账户密码为 123456，然后我们再通过以下命令启动 user 容器：</p>
<pre class="lang-powershell" data-nodeid="1089"><code data-language="powershell">docker run <span class="hljs-literal">-itd</span> -<span class="hljs-literal">-name</span> user -<span class="hljs-literal">-network</span> host user 
</code></pre>
<p data-nodeid="1090">与前面的启动命令不同，我们将使用 host 网络模式启动 user 容器，这意味着 user 容器的网络与宿主机的网络是一样的，这样启动的目的是方便我们在 user 容器的代码中直接使用 localhost 访问部署在同一台宿主机的 Redis 和 MySQL 容器。user 容器启动成功后，我们就可以通过进入到 user 容器中或者在宿主机中访问 user 服务中的接口。非 Linux 的宿主机不支持 host 网络模式，无法将容器暴露的端口直接绑定到宿主机，只能进入到 user 容器内访问接口。可通过以下指令访问 user 容器：</p>
<pre class="lang-powershell" data-nodeid="1091"><code data-language="powershell"> docker exec <span class="hljs-literal">-it</span> user /bin/bash 
</code></pre>
<p data-nodeid="1092">对于单主机上的多个 Docker 容器的部署，可以使用 Docker-Compose 进行管理和编排。比如上述例子中我们部署的 3 个容器就可以通过 Docker-Compose 进行统一部署和编排，若你有兴趣的话可以额外了解一下。</p>
<p data-nodeid="1093">虽然 Docker 极大地方便了应用程序的部署和运行，但是在具体应用实践会带来另一个问题：大量的容器如何进行编排、管理和调度？此时就需要我们的另一个主角 Kubernetes 登场了。</p>
<h3 data-nodeid="9234" class="">Kubernetes 简介</h3>

<p data-nodeid="1095">仅仅有容器还是不够的，虽然解决了应用程序运行环境的问题，但大量的容器还是需要人工部署，带来了较大的人力成本和较高的出错率，对此就需要一定的容器编排工具对大量运行的容器进行管理。这就引出了我们接下来要介绍的 Kubernetes。</p>
<p data-nodeid="1096" class=""><strong data-nodeid="1247">Kubernetes</strong>是由 Google 开源的，<strong data-nodeid="1248">目的是管理公司内部运行的成千上万的服务器，降低应用程序部署管理的成本</strong>。Kubernetes 将基础设施抽象，简化了应用开发、部署和运维等工作，提高了硬件资源的利用率，<strong data-nodeid="1249">是一款优秀的容器管理和编排系统</strong>。</p>
<p data-nodeid="13724">Kubernetes 主要有由两类节点组成：Master 节点主要负责管理和控制，是 Kubernetes 的调度中心；Node 节点受 Master 节点管理，属于工作节点，负责运行具体的容器应用。整体结构图如下所示：</p>
<p data-nodeid="14000"><img src="https://s0.lgstatic.com/i/image/M00/3B/6F/CgqCHl8j-j-AL1E1AADTHvJdwEQ723.png" alt="10_Kubernetes_集群架构图.png" data-nodeid="14008"></p>
<div data-nodeid="14535" class=""><p style="text-align:center">Kubernetes 集群架构图</p> </div>

















<p data-nodeid="1101">Master 节点主要由以下几部分组成：</p>
<ul data-nodeid="1102">
<li data-nodeid="1103">
<p data-nodeid="1104">API Server，对外提供 Kubernetes 的服务接口，供各类客户端使用；</p>
</li>
<li data-nodeid="1105">
<p data-nodeid="1106">Scheduler，负责对集群内部的资源进行调度，按照预设的策略将 Pod 调度到相应的 Node 节点；</p>
</li>
<li data-nodeid="1107">
<p data-nodeid="1108">Controller Manager，作为管理控制器，负责维护整个集群的状态；</p>
</li>
<li data-nodeid="1109">
<p data-nodeid="1110">etcd，保存整个集群的状态数据。</p>
</li>
</ul>
<p data-nodeid="1111">Node 节点的主要组成部分为：</p>
<ul data-nodeid="1112">
<li data-nodeid="1113">
<p data-nodeid="1114">Pod，Kubernetes 创建和部署的基本操作单位，它代表了集群中运行的一个进程，内部由一个或者多个共享资源的容器组成，我们可以简单将 Pod 理解成一台虚拟主机，主机内的容器共享网络、存储等资源；</p>
</li>
<li data-nodeid="1115">
<p data-nodeid="1116">Docker，是 Pod 中最常见的容器 runtime，Pod 也支持其他容器 runtime；</p>
</li>
<li data-nodeid="1117">
<p data-nodeid="1118">Kubelet，负责维护调度到它所在 Node 节点的 Pod 的生命周期，包括创建、修改、删除和监控等；</p>
</li>
<li data-nodeid="1119">
<p data-nodeid="1120">Kube-proxy，负责为 Pod 提供代理，为 Service 提供集群内部的服务发现和负载均衡，Service 可以看作一组提供相同服务的 Pod 的对外访问接口。</p>
</li>
</ul>
<h3 data-nodeid="15062" class="">Kubernetes 部署 user 服务</h3>

<p data-nodeid="1122">接下来我们将使用创建 Pod 部署 user 服务以及它依赖的 Redis 和 MySQL 数据库。我们可以通过 yaml 文件描述配置过程和使用 kubectl 命令行工具访问 Kubernetes 的接口，user 服务的 Pod 描述如下：</p>
<pre class="lang-yaml" data-nodeid="1123"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span> 
<span class="hljs-attr">kind:</span> <span class="hljs-string">Pod</span> 
<span class="hljs-attr">metadata:</span> 
  <span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span> 
  <span class="hljs-attr">labels:</span> 
    <span class="hljs-attr">name:</span> <span class="hljs-string">user-service</span> 
<span class="hljs-attr">spec:</span> 
  <span class="hljs-attr">containers:</span>                    <span class="hljs-comment">#定义user容器，开放10086端口 </span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">user</span> 
      <span class="hljs-attr">image:</span> <span class="hljs-string">user</span> 
      <span class="hljs-attr">ports:</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">10086</span> 
      <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">mysql</span>                     <span class="hljs-comment">#定义MySQL容器，开放3306端口 </span>
      <span class="hljs-attr">image:</span> <span class="hljs-string">mysql-for-user</span> 
      <span class="hljs-attr">ports:</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">3306</span> 
      <span class="hljs-attr">env:</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">MYSQL_ROOT_PASSWORD</span> 
          <span class="hljs-attr">value:</span> <span class="hljs-string">"123456"</span> 
      <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">redis</span>                     <span class="hljs-comment">#定义Redis容器，开放6379端口 </span>
      <span class="hljs-attr">image:</span> <span class="hljs-string">redis:5.0</span> 
      <span class="hljs-attr">ports:</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">6379</span> 
      <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
</code></pre>
<p data-nodeid="1124">上述 yaml 中，展示了部署 user、mysql 和 redis 3 个容器应用的简单Pod。由于在同一个 Pod 中的多个容器是并发启动的，为了保证 user 服务启动时 Redis 和 MySQL 数据库已经部署启动完成，在 user 服务的 main 函数中增加了 time.Sleep 延迟了 user 服务的启动。通过  kubectl create 命令和 yaml 描述启动 Pod，命令如下所示：</p>
<pre class="lang-java" data-nodeid="17171"><code data-language="java"> kubectl create -f user-service.yaml 
</code></pre>




<p data-nodeid="1126">这将在 Kubernetes 集群的 Node 节点中创建单个 Pod。通过以下两个命令可分别查看 user-service Pod 的信息和进入到 Pod 中：</p>
<pre class="lang-java" data-nodeid="17698"><code data-language="java"> kubectl get pod user-service  
 kubectl exec -ti user-service -n <span class="hljs-keyword">default</span>  -- /bin/bash 
</code></pre>

<p data-nodeid="1128">单个 Pod 不具备自我恢复的能力，当 Pod 所在的 Node 出现问题，Pod 就很可能被删除，这就会导致 Pod 中容器提供的服务被终止。为了避免这种情况的发生，可以使用 Controller 来管理 Pod，Controller 提供创建和管理多个 Pod 的能力，从而使得被管理的 Pod 具备自愈和更新的能力。常见的 Controller 有以下几种：</p>
<ul data-nodeid="1129">
<li data-nodeid="1130">
<p data-nodeid="1131">Replication Controller，确保用户定义的 Pod 副本数保持不变；</p>
</li>
<li data-nodeid="1132">
<p data-nodeid="1133">ReplicaSet，是 RC 的升级版，在选择器（Selector）的支持上优于 RC，RC 只支持基于等式的选择器，但 RS 还支持基于集合的选择器；</p>
</li>
<li data-nodeid="1134">
<p data-nodeid="1135">Deployment，在 RS 的基础上提供了 Pod 的更新能力，在 Deployment 配置文件中 Pod template 发生变化时，它能将现在集群的状态逐步更新成 Deployment 中定义的目标状态；</p>
</li>
<li data-nodeid="1136">
<p data-nodeid="1137">StatefulSets，其中的 Pod 是有序部署和具备稳定的标识，是一组存在状态的 Pod 副本。</p>
</li>
</ul>
<p data-nodeid="1138">比如我们可以使用 Deployment Controller 为我们管理 user-service Pod，配置如下：</p>
<pre class="lang-yaml" data-nodeid="1139"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span> 
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
      <span class="hljs-attr">containers:</span>                    <span class="hljs-comment">#定义user容器，开放10086端口 </span>
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">user</span> 
          <span class="hljs-attr">image:</span> <span class="hljs-string">user</span> 
          <span class="hljs-attr">ports:</span> 
            <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">10086</span> 
          <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">mysql</span>                     <span class="hljs-comment">#定义MySQL容器，开放3306端口 </span>
          <span class="hljs-attr">image:</span> <span class="hljs-string">mysql-for-user</span> 
          <span class="hljs-attr">ports:</span> 
            <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">3306</span> 
          <span class="hljs-attr">env:</span> 
            <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">MYSQL_ROOT_PASSWORD</span> 
              <span class="hljs-attr">value:</span> <span class="hljs-string">"123456"</span> 
          <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
        <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">redis</span>                     <span class="hljs-comment">#定义Redis容器，开放6379端口 </span>
          <span class="hljs-attr">image:</span> <span class="hljs-string">redis:5.0</span> 
          <span class="hljs-attr">ports:</span> 
            <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">6379</span> 
          <span class="hljs-attr">imagePullPolicy:</span> <span class="hljs-string">IfNotPresent</span> 
</code></pre>
<p data-nodeid="1140">在上述配置中，我们指定了 kind 的类型为 Deployment、副本的数量为 3 和选择器为匹配标签 name: user-service。可以发现原来 Pod 的配置放到了 template 标签下，并添加 name: user-service 的标签，Deployment Controller 将会使用 template 下的 Pod 配置来创建 Pod 副本，并通过标签选择器来监控 Pod 副本的数量，当副本数不足时，将会根据 template 创建 Pod。</p>
<p data-nodeid="1141">通过以下命令即可通过 Deployment Controller 管理 user-service Pod，命令如下：</p>
<pre class="lang-java" data-nodeid="18225"><code data-language="java">kubectl create -f user-service-deployment.yaml 
</code></pre>

<p data-nodeid="1143">可以通过  kubectl get Deployment 命令查看 user-service 的 Pod 副本状态，如下所示：</p>
<pre class="lang-java" data-nodeid="18752"><code data-language="java"> kubectl get Deployment user-service 
</code></pre>

<p data-nodeid="1145">Deployment Controller 默认使用 RollingUpdate 策略更新 Pod，也就是滚动更新的方式；另一种更新策略是 Recreate，创建出新的 Pod 之前会先杀掉所有已存在的 Pod，可以通过 spec.strategy.type 标签指定更新策略。Deployment 的 rollout 当且仅当 Deployment 的 Pod template 中的 label 更新或者镜像更改时被触发，比如我们希望更新 redis 的版本：</p>
<pre class="lang-java" data-nodeid="19279"><code data-language="java">kubectl set image deployment/user-service redis=redis:<span class="hljs-number">6.0</span> 
</code></pre>

<p data-nodeid="1147">这将触发 user-service Pod 的重新更新部署。当 Pod 被 Deployment Controller 管理时，单独使用 kubectl delete pod 无法删除相关 Pod，Deployment Controller 会维持 Pod 副本数量不变，这时则需要通过 kubectl delete Deployment 删除相关 Deployment 配置，比如删除 user-service 的 Deployment 配置，如下命令所示：</p>
<pre class="lang-java" data-nodeid="19806"><code data-language="java"> kubectl delete Deployment user-service 
</code></pre>

<h3 data-nodeid="20333" class="">小结</h3>

<p data-nodeid="1150">在微服务架构演进的过程中，随着微服务数量的增加，很容易给服务的部署和运维带来巨大的挑战。而 Docker 和 Kubernetes 的出现大大弥补了这部分缺陷：<strong data-nodeid="1289">Docker 使得应用程序运行环境的隔离和迁移变得更加简单，Kubernetes 则提高了大规模部署和运行容器的效率</strong>。</p>
<p data-nodeid="1151">在本课时中，我们首先介绍了 Docker 和通过 Docker 部署了 user 服务以及它的依赖服务，接着介绍了 Kubernetes 和使用 Kubernetes 将 user 服务以及它的依赖服务的容器打包到同一个 Pod 中进行容器编排。希望通过本课时的学习，能够加深你对容器运行和编排的认识，我们案例应用的微服务也会采用容器化的方式进行部署和运维，简化微服务部署和运维的工作成本。</p>
<p data-nodeid="1152">最后，对于 Docker 和 Kubernetes，你有什么独到的见解？欢迎在评论区与我分享。</p>

---

### 精选评论

##### **升：
> 赚了啊，学go 还能学docker和k8s

##### *杰：
> 据我了解，镜像那里可以做成多阶段构建，然后用最小镜像跑go build 出来的产物，打包出来的镜像会更小。

##### **成：
> docker的话，建议使用docker-compose 做后台服务。kubernet对外服务需要创建service 及ingress

##### *嵩：
> 不推荐用latest的docker镜像tag不管是作为基础镜像还是构建镜像

