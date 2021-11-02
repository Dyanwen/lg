<p data-nodeid="274721">上一课时，我介绍了 Docker 的单节点引擎工具 Docker Compose，它能够在单一节点上管理和编排多个容器，当我们的服务和容器数量较小时可以使用 Docker Compose 来管理容器。</p>



<p data-nodeid="273824">然而随着我们的业务规模越来越大，我们的容器规模也逐渐增大时，数量庞大的容器管理将给我们带来许多挑战。Docker 官方为了解决多容器管理的问题推出了 Docker Swarm ，我们可以用它来管理规模更大的容器集群。</p>
<h3 data-nodeid="273825">Swarm 的前生今世</h3>
<p data-nodeid="273826">2014 年 Docker 在容器界越来越火，这时容器的编排工具 Mesos 和 Kubernetes 也开始崭露头角。此时，Docker 公司也开始筹划容器的编排和集群管理工具，推出了自己的通信协议项目 Beam。后来，通过改进 Beam，Beam 成为一个允许使用 Docker API 来控制的一种分布式系统，之后项目被重命名为 libswarm。然而在 2014 年 11 月，Docker 公司又对 libswarm 进行了重新设计，支持了远程调用 API，并且被重新命名为 Swarm。到此我们称之为 Swarm V1。</p>
<p data-nodeid="273827">在 2016 年，为了解决中央服务可扩展性的问题，Docker 团队重新设计了 Swarm，并称之为  Swarm V2。此时的  Docker Swarm 已经可以支持超过 1000 多个节点的集群规模，并且 Docker 团队在发布 Docker 1.12 版本时，将 Docker Swarm 默认集成到了 Docker 引擎中。</p>
<p data-nodeid="273828">由于 Swarm 是 Docker 官方推出的容器集群管理工具，因此 Swarm 最大的优势之一就是原生支持 Docker API，给用户带来了极大的便利，原来的 Docker 用户可以很方便地将服务迁移到 Swarm 中来。</p>
<p data-nodeid="273829">与此同时，Swarm 还内置了对 Docker 网络插件的支持，因此用户可以很方便地部署需要跨主机通信的容器集群。其实 Swarm 的优点远远不止这些，还有很多，例如以下优点。</p>
<ul data-nodeid="276554">
<li data-nodeid="276555">
<p data-nodeid="276556"><strong data-nodeid="276569">分布式：</strong> Swarm 使用<a href="https://raft.github.io/" data-nodeid="276567">Raft</a>（一种分布式一致性协议）协议来做集群间数据一致性保障，使用多个容器节点组成管理集群，从而避免单点故障。</p>
</li>
<li data-nodeid="276557">
<p data-nodeid="276558"><strong data-nodeid="276574">安全：</strong> Swarm 使用 TLS 双向认证来确保节点之间通信的安全，它可以利用双向 TLS 进行节点之间的身份认证，角色授权和加密传输，并且可以自动执行证书的颁发和更换。</p>
</li>
<li data-nodeid="276559">
<p data-nodeid="276560" class=""><strong data-nodeid="276579">简单：</strong> Swarm 的操作非常简单，并且除 Docker 外基本无其他外部依赖，而且从 Docker 1.12 版本后， Swarm 直接被内置到了 Docker 中，可以说真正做到了开箱即用。</p>
</li>
</ul>



<p data-nodeid="273837">Swarm 的这些优点得益于它优美的架构设计，下面我们来了解一下 Swarm 的架构。</p>
<h3 data-nodeid="273838">Swarm 的架构</h3>
<p data-nodeid="277770">Swarm 的架构整体分为<strong data-nodeid="277781">管理节点</strong>（Manager Nodes）和<strong data-nodeid="277782">工作节点</strong>（Worker Nodes），整体架构如下图：</p>
<p data-nodeid="278999"><img src="https://s0.lgstatic.com/i/image/M00/67/E1/CgqCHl-iZxSAbYhzAABiA3_fQM8971.png" alt="image.png" data-nodeid="279003"></p>
<div data-nodeid="279000" class=""><p style="text-align:center">图 1 Swarm 架构图</p></div>






<p data-nodeid="279600" class=""><strong data-nodeid="279605">管理节点：</strong> 管理节点负责接受用户的请求，用户的请求中包含用户定义的容器运行状态描述，然后 Swarm 负责调度和管理容器，并且努力达到用户所期望的状态。</p>

<p data-nodeid="280204" class=""><strong data-nodeid="280209">工作节点：</strong> 工作节点运行执行器（Executor）负责执行具体的容器管理任务（Task），例如容器的启动、停止、删除等操作。</p>

<blockquote data-nodeid="273844">
<p data-nodeid="273845">管理节点和工作节点的角色并不是一成不变的，你可以手动将工作节点转换为管理节点，也可以将管理节点转换为工作节点。</p>
</blockquote>
<h3 data-nodeid="273846">Swarm 核心概念</h3>
<p data-nodeid="273847">在真正使用 Swarm 之前，我们需要了解几个 Swarm 的核心概念，这些核心概念可以帮助我们更好地学习和理解 Swarm 的设计理念。</p>
<h4 data-nodeid="273848">Swarm 集群</h4>
<p data-nodeid="273849">Swarm 集群是一组被 Swarm 统一管理和调度的节点，被 Swarm纳管的节点可以是物理机或者虚拟机。其中一部分节点作为管理节点，负责集群状态的管理和协调，另一部分作为工作节点，负责执行具体的任务来管理容器，实现用户服务的启停等功能。</p>
<h4 data-nodeid="273850">节点</h4>
<p data-nodeid="273851">Swarm 集群中的每一台物理机或者虚拟机称为节点。节点按照工作职责分为<strong data-nodeid="274025">管理节点</strong>和<strong data-nodeid="274026">工作节点</strong>，管理节点由于需要使用 Raft 协议来协商节点状态，生产环境中通常建议将管理节点的数量设置为奇数个，一般为 3 个、5 个或 7 个。</p>
<h4 data-nodeid="273852">服务</h4>
<p data-nodeid="273853">服务是为了支持容器编排所提出的概念，它是一系列复杂容器环境互相协作的统称。一个服务的声明通常包含容器的启动方式、启动的副本数、环境变量、存储、配置、网络等一系列配置，用户通过声明一个服务，将它交给 Swarm，Swarm 负责将用户声明的服务实现。</p>
<p data-nodeid="273854">服务分为全局服务（global services）和副本服务（replicated services）。</p>
<ul data-nodeid="280810">
<li data-nodeid="280811">
<p data-nodeid="280812" class="">全局服务：每个工作节点上都会运行一个任务，类似于 Kubernetes 中的 <a href="https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/" data-nodeid="280818">Daemonset</a>。</p>
</li>
<li data-nodeid="280813">
<p data-nodeid="280814">副本服务：按照指定的副本数在整个集群中调度运行。</p>
</li>
</ul>

<h4 data-nodeid="273860">任务</h4>
<p data-nodeid="273861">任务是集群中的最小调度单位，它包含一个真正运行中的 Docker 容器。当管理节点根据服务中声明的副本数将任务调度到节点时，任务则开始在该节点启动和运行，当节点出现异常时，任务会运行失败。此时调度器会把失败的任务重新调度到其他正常的节点上正常运行，以确保运行中的容器副本数满足用户所期望的副本数。</p>
<h4 data-nodeid="273862">服务外部访问</h4>
<p data-nodeid="273863">由于容器的 IP 只能在集群内部访问到，而且容器又是用后马上销毁，这样容器的 IP 也会动态变化，因此容器集群内部的服务想要被集群外部的用户访问到，服务必须要映射到主机上的固定端口。Swarm 使用入口负载均衡（ingress load balancing）的模式将服务暴露在主机上，该模式下，每一个服务会被分配一个公开端口（PublishedPort），你可以指定使用某个未被占用的公开端口，也可以让 Swarm 自动分配一个。</p>
<p data-nodeid="273864">Swarm 集群的公开端口可以从集群内的任意节点上访问到，当请求达到集群中的一个节点时，如果该节点没有要请求的服务，则会将请求转发到实际运行该服务的节点上，从而响应用户的请求。公有云的云负载均衡器（cloud load balancers）可以利用这一特性将流量导入到集群中的一个或多个节点，从而实现利用公有云的云负载均衡器将流量导入到集群中的服务。</p>
<h3 data-nodeid="273865">搭建 Swarm 集群</h3>
<p data-nodeid="273866">要想使用 Swarm 集群有如下一些要求：</p>
<ul data-nodeid="273867">
<li data-nodeid="273868">
<p data-nodeid="273869">Docker 版本大于 1.12，推荐使用最新稳定版 Docker；</p>
</li>
<li data-nodeid="273870">
<p data-nodeid="273871">主机需要开放一些端口（TCP：2377 UDP:4789 TCP 和 UDP:7946）。</p>
</li>
</ul>
<p data-nodeid="281927">下面我通过四台机器来搭建一个 Swarm 集群，演示的节点规划如下：</p>
<p data-nodeid="281928" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/D6/Ciqc1F-iZ0KAdrQoAABINXCXUv0846.png" alt="Lark20201104-162431.png" data-nodeid="281932"></p>



<blockquote data-nodeid="273902">
<p data-nodeid="273903" class="">生产环境中推荐使用至少三个 manager 作为管理节点。</p>
</blockquote>
<ul data-nodeid="273904">
<li data-nodeid="273905" class="">
<p data-nodeid="273906" class="">第一步：初始化集群</p>
</li>
</ul>
<p data-nodeid="273907" class="">Docker 1.12 版本后， Swarm 已经默认集成到了 Docker 中，因此我们可以直接使用 Docker 命令来初始化 Swarm，集群初始化的命令格式如下：</p>
<pre class="lang-js" data-nodeid="285547"><code data-language="js">docker swarm init --advertise-addr &lt;YOUR-IP&gt;
</code></pre>
<blockquote data-nodeid="285548">
<p data-nodeid="285549">advertise-addr 一般用于主机有多块网卡的情况，如果你的主机只有一块网卡，可以忽略此参数。</p>
</blockquote>
<p data-nodeid="285550">在管理节点上，通过以下命令初始化集群：</p>
<pre class="lang-js" data-nodeid="286781"><code data-language="js">$ docker swarm init
Swarm initialized: current node (<span class="hljs-number">1</span>ehtnlcf3emncktgjzpoux5ga) is now a manager.

To add a worker to <span class="hljs-built_in">this</span> swarm, run the following command:

&nbsp; &nbsp; docker swarm join --token SWMTKN-<span class="hljs-number">1</span>-<span class="hljs-number">1</span>kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-<span class="hljs-number">8</span>j4l0t2is9ok9jwwc3tovtxbp <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.100</span>:<span class="hljs-number">2377</span>

To add a manager to <span class="hljs-built_in">this</span> swarm, run <span class="hljs-string">'docker swarm join-token manager'</span> and follow the instructions.
</code></pre>
<p data-nodeid="286782">集群初始化后， Swarm 会提示我们当前节点已经作为一个管理节点了，并且提示了如何把一台主机加入集群成为工作节点。</p>
<ul data-nodeid="286783">
<li data-nodeid="286784">
<p data-nodeid="286785">第二步：加入工作节点</p>
</li>
</ul>
<p data-nodeid="286786">按照第一步集群初始化后输出的提示，只需要复制其中的命令即可，然后在剩余的三台工作节点上分别执行如下命令：</p>
<pre class="lang-js" data-nodeid="288000"><code data-language="js">$ docker swarm join --token SWMTKN-<span class="hljs-number">1</span>-<span class="hljs-number">1</span>kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-<span class="hljs-number">8</span>j4l0t2is9ok9jwwc3tovtxbp <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.100</span>:<span class="hljs-number">2377</span>
This node joined a swarm <span class="hljs-keyword">as</span> a worker.
</code></pre>
<p data-nodeid="288001">默认加入的节点为工作节点，如果是生产环境，我们可以使用<code data-backticks="1" data-nodeid="288043">docker swarm join-token manager</code>命令来查看如何加入管理节点：</p>
<pre class="lang-js" data-nodeid="289205"><code data-language="js">$ docker swarm join-to ken manager
To add a manager to <span class="hljs-built_in">this</span> swarm, run the following command:

&nbsp; &nbsp; docker swarm join --token SWMTKN-<span class="hljs-number">1</span>-<span class="hljs-number">1</span>kal5b1iozbfmnnhx3kjfd3y6yqcjjjpcftrlg69pm2g8hw5vx-<span class="hljs-number">8</span>fq89jxo2axwggryvom5a337t <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.100</span>:<span class="hljs-number">2377</span>
</code></pre>
<p data-nodeid="289206">复制 Swarm 输出的结果即可加入管理节点到集群中。</p>
<blockquote data-nodeid="289207">
<p data-nodeid="289208">注意：管理节点的数量必须为奇数，生产环境推荐使用3个、5个或7个管理节点来管理 Swarm 集群。</p>
</blockquote>
<ul data-nodeid="289209">
<li data-nodeid="289210">
<p data-nodeid="289211">第三步：节点查看</p>
</li>
</ul>
<p data-nodeid="289212">节点添加完成后，我们使用以下命令可以查看当前节点的状态：</p>
<pre class="lang-dart" data-nodeid="296861"><code data-language="dart">$ ]# docker node ls
ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; HOSTNAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; AVAILABILITY&nbsp; &nbsp; &nbsp; &nbsp; MANAGER STATUS&nbsp; &nbsp; &nbsp; ENGINE VERSION
<span class="hljs-number">1</span>ehtnlcf3emncktgjzpoux5ga *&nbsp; &nbsp;swarm-manager&nbsp; &nbsp; &nbsp; &nbsp;Ready&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Active&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Leader&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">19.03</span><span class="hljs-number">.12</span>
pn7gdm847sfzydqhcv3vma97y *&nbsp; &nbsp;swarm-node1&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Ready&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Active&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">19.03</span><span class="hljs-number">.12</span>
<span class="hljs-number">4</span>dtc9pw5quyjs5yf25ccgr8uh *&nbsp; &nbsp;swarm-node2&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Ready&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Active&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">19.03</span><span class="hljs-number">.12</span>
est7ww3gngna4u7td22g9m2k5 *&nbsp; &nbsp;swarm-node3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Ready&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Active&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">19.03</span><span class="hljs-number">.12</span>
</code></pre>
<p data-nodeid="296862">到此，一个包含 1 个管理节点，3 个工作节点的 Swarm 集群已经搭建完成。</p>
<h3 data-nodeid="296863">使用 Swarm</h3>
<p data-nodeid="296864">集群搭建完成后，我们就可以在 Swarm 集群中创建服务了，Swarm 集群中常用的服务部署方式有以下两种。</p>
<h4 data-nodeid="296865">（1）通过 docker service 命令创建服务</h4>
<p data-nodeid="296866">使用<code data-backticks="1" data-nodeid="296898">docker service create</code>命令可以创建服务，创建服务的命令如下：</p>
<pre class="lang-js" data-nodeid="298024"><code data-language="js">$ docker service create --replicas <span class="hljs-number">1</span> --name hello-world nginx
<span class="hljs-number">24</span>f9ng83m9sq4ml3e92k4g5by
overall progress: <span class="hljs-number">1</span> out <span class="hljs-keyword">of</span> <span class="hljs-number">1</span> tasks
<span class="hljs-number">1</span>/<span class="hljs-number">1</span>: running&nbsp; &nbsp;[==================================================&gt;]
<span class="hljs-attr">verify</span>: Service converged
</code></pre>
<p data-nodeid="298025">此时我们已经创建好了一个服务，使用<code data-backticks="1" data-nodeid="298051">docker service ls</code>命令可以查看已经启动的服务：</p>
<pre class="lang-js" data-nodeid="299169"><code data-language="js">$ docker service ls
ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MODE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; REPLICAS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IMAGE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;PORTS
<span class="hljs-number">24</span>f9ng83m9sq&nbsp; &nbsp; &nbsp; &nbsp; hello-world&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;replicated&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;nginx:latest
</code></pre>
<p data-nodeid="299170">当我们不再需要这个服务了，可以使用<code data-backticks="1" data-nodeid="299194">docker service rm</code>命令来删除服务：</p>
<pre class="lang-js" data-nodeid="301434"><code data-language="js">$ docker service rm hello-world
hello-world
</code></pre>
<p data-nodeid="301435">此时 hello-world 这个服务已经成功地从集群中删除。<br>
想要了解更多的<code data-backticks="1" data-nodeid="301459">docker service</code>命令的相关操作，可以参考<a href="https://docs.docker.com/engine/swarm/swarm-tutorial/deploy-service/" data-nodeid="301463">这里</a>。</p>
<p data-nodeid="301436">生产环境中，我们推荐使用 docker-compose 模板文件来部署服务，这样服务的管理会更加方便并且可追踪，而且可以同时创建和管理多个服务，更加适合生产环境中依赖关系较复杂的部署模式。</p>
<h4 data-nodeid="301437">（2）通过 docker stack 命令创建服务</h4>
<p data-nodeid="301438">我们在 19 课时中创建了 docker-compose 的模板文件，成功的使用该模板文件创建并启动了 MySQL 服务和 WordPress 两个服务。这里我们将 19 讲中的 docker-compose 模板文件略微改造一下：</p>
<pre class="lang-yaml" data-nodeid="301439"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">'3'</span>

<span class="hljs-attr">services:</span>
&nbsp; &nbsp;<span class="hljs-attr">mysql:</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">image:</span> <span class="hljs-string">mysql:5.7</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">volumes:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-bullet">-</span> <span class="hljs-string">mysql_data:/var/lib/mysql</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">restart:</span> <span class="hljs-string">always</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">environment:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">MYSQL_ROOT_PASSWORD:</span> <span class="hljs-string">root</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">MYSQL_DATABASE:</span> <span class="hljs-string">mywordpress</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">MYSQL_USER:</span> <span class="hljs-string">mywordpress</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">MYSQL_PASSWORD:</span> <span class="hljs-string">mywordpress</span>

&nbsp; &nbsp;<span class="hljs-attr">wordpress:</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">depends_on:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-bullet">-</span> <span class="hljs-string">mysql</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">image:</span> <span class="hljs-string">wordpress:php7.4</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">deploy:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">mode:</span> <span class="hljs-string">replicated</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">replicas:</span> <span class="hljs-number">2</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">ports:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-bullet">-</span> <span class="hljs-string">"8080:80"</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">restart:</span> <span class="hljs-string">always</span>
&nbsp; &nbsp; &nbsp;<span class="hljs-attr">environment:</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">WORDPRESS_DB_HOST:</span> <span class="hljs-string">mysql:3306</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">WORDPRESS_DB_USER:</span> <span class="hljs-string">mywordpress</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">WORDPRESS_DB_PASSWORD:</span> <span class="hljs-string">mywordpress</span>
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">WORDPRESS_DB_NAME:</span> <span class="hljs-string">mywordpress</span>
<span class="hljs-attr">volumes:</span>
&nbsp; &nbsp; <span class="hljs-attr">mysql_data:</span> {}

</code></pre>
<p data-nodeid="301440">我在服务模板文件中添加了 deploy 指令，并且指定使用副本服务（replicated）的方式启动两个 WordPress 实例。</p>
<p data-nodeid="301441">准备好启动 WordPress 服务的配置后，我们在 /tmp 目下新建 docker-compose.yml 文件，并且写入以上的内容，然后我们使用以下命令启动服务：</p>
<pre class="lang-java" data-nodeid="312859"><code data-language="java">$ docker stack deploy -c docker-compose.yml wordpress
Ignoring unsupported options: restart

Creating network wordpress_default
Creating service wordpress_mysql
Creating service wordpress_wordpress
</code></pre>
<p data-nodeid="312860">执行完以上命令后，我们成功启动了两个服务：</p>
<ol data-nodeid="312861">
<li data-nodeid="312862">
<p data-nodeid="312863">MySQL 服务，默认启动了一个副本。</p>
</li>
<li data-nodeid="312864">
<p data-nodeid="312865">WordPress 服务，根据我们 docker-compose 模板的定义启动了两个副本。</p>
</li>
</ol>
<p data-nodeid="312866">下面我们用<code data-backticks="1" data-nodeid="312877">docker service ls</code>命令查看一下当前启动的服务。</p>
<pre class="lang-java" data-nodeid="314989"><code data-language="java">$ docker service ls
ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MODE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; REPLICAS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; IMAGE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;PORTS
v8i0pzb4e3tc&nbsp; &nbsp; &nbsp; &nbsp; wordpress_mysql&nbsp; &nbsp; &nbsp; &nbsp;replicated&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;mysql:<span class="hljs-number">5.7</span>
<span class="hljs-number">96</span>m8xfyeqzr5&nbsp; &nbsp; &nbsp; &nbsp; wordpress_wordpress&nbsp; &nbsp;replicated&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;wordpress:php7.<span class="hljs-number">4</span>&nbsp; &nbsp; *:<span class="hljs-number">8080</span>-&gt;<span class="hljs-number">80</span>/tcp
</code></pre>
<p data-nodeid="314990">可以看到，Swarm 已经为我们成功启动了一个 MySQL 服务，并且启动了两个 WordPress 实例。WordPress 实例通过 8080 端口暴露在了主机上，我们通过访问集群中的任意节点的 IP 加 8080 端口即可访问到 WordPress 服务。例如，我们访问<a href="http://192.168.31.101:8080" data-nodeid="314998">http://192.168.31.101:8080</a>即可成功访问到我们搭建的 WordPress 服务。</p>
<h3 data-nodeid="314991">结语</h3>
<p data-nodeid="314992">Docker Swarm 是一个用来定义复杂应用的集群编排工具，可以帮我们把多台主机组成一个 Swarm 集群，并且帮助我们管理和调度复杂的容器服务。由于 Swarm 已经被内置于 Docker 中，因此 Swarm 的安装和使用也变得非常简单，只要你有 Docker 的使用经验，就可以很快地将你的应用迁移到 Swarm 集群中。</p>
<p data-nodeid="314993">那么，学完本课时内容，你可以试着构建一个高可用（管理节点扩展为 3 个或 5 个）的 Swarm 集群吗？</p>
<p data-nodeid="314994">下一课时，我将为你讲解目前使用最多的容器编排系统Kubernetes，再会。</p>

---

### 精选评论

##### lagogo：
> 文章里要弄四台机器.我尝试用docker machine.不过有个缺点,ip是动态的. 想问一下如何方便地弄成静态ip? 命令如下:docker-machine --debug --native-ssh create --driver hyperv --hyperv-virtual-switch "Default Switch" manager

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以用 virtualbox 或者 vmware 创建虚拟机，如果实在没有这么多资源，少创建一些机器做单 master 的集群也是可以的。

##### *冲：
> ID PORTSpmwyqxyc1ku7 kj8dyvsy6iyj zp04dp06mic9 80/tcp为啥我的WordPress服务没有副本? 并且对应的网址不能访问. 防火墙是关着的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 docker service ls 命令查看下服务是否已经正常启动，如果没有正常启动，请根据课程中的步骤启动服务即可。如果启动有问题 docker 会报响应的错误。

##### *冲：
> line 7: could not find expected ':'这个docker-compose.yml文件第七行报错,说是: - 符号后边要有空格,是有的呀, 复制和自己手写都试了一下, 还是不行,执行docker stack deploy -c docker-compose.yml wordpress 报错

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是网站代码复制的问题，你可以手动去除换行符。我让小编更正下～

##### **超：
> 小白想问下为什么管理节点必须是奇数呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为要选主，超过一半的投票才可以被选为主节点，偶数节点会出现一半一半的情况

##### **9660：
> 内容很棒，由浅入深，学到很多。

##### **升：
> 打卡

