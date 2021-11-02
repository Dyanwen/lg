<p data-nodeid="9653" class="">前几课时，我介绍了 Linux 的 Namespace 和 Cgroups 技术，利用这两项技术可以实现各种资源的隔离和主机资源的限制，让我们的容器可以像一台虚拟机一样。但这时我们的容器就像一台未联网的电脑，不能被外部访问到，也不能主动与外部通信，这样的容器只能做一些离线的处理任务，无法通过外部访问。所以今天这一讲，我将介绍 Docker 网络相关的知识，使 Docker 容器接通网络。</p>
<h3 data-nodeid="9654">容器网络发展史</h3>
<p data-nodeid="9655">提起 Docker 网络，我们不得不从容器战争说起。Docker 从 2013 年诞生，到后来逐渐成为了容器的代名词，然而 Docker 的野心也不止于此，它还想在更多的领域独占鳌头，比如制定容器的网络和存储标准。</p>
<p data-nodeid="9656" class="">于是 Docker 从 1.7 版本开始，便把网络和存储从 Docker 中正式以插件的形式剥离开来，并且分别为其定义了标准，Docker 定义的网络模型标准称之为 CNM (Container Network Model) 。</p>
<blockquote data-nodeid="10289">
<p data-nodeid="10290" class="te-preview-highlight">Docker 推出 CNM 的同时，CoreOS 推出了 CNI（Container Network Interfac）。起初，以 Kubernetes 为代表的容器编排阵营考虑过使用 CNM 作为容器的网络标准，但是后来由于很多技术和非技术原因（如果你对详细原因感兴趣，可以参考这篇博客），Kubernetes 决定支持 CoreOS 推出的容器网络标准 CNI。</p>
</blockquote>


<p data-nodeid="9659">从此，容器的网络标准便分为两大阵营，一个是以 Docker 公司为代表的 CNM，另一个便是以 Google、Kubernetes、CoreOS 为代表的 CNI 网络标准。</p>
<h3 data-nodeid="9660">CNM</h3>
<p data-nodeid="9661">CNM (Container Network Model) 是 Docker 发布的容器网络标准，意在规范和指定容器网络发展标准，CNM 抽象了容器的网络接口 ，使得只要满足 CNM 接口的网络方案都可以接入到 Docker 容器网络，更好地满足了用户网络模型多样化的需求。</p>
<p data-nodeid="9662">CNM 只是定义了网络标准，对于底层的具体实现并不太关心，这样便解耦了容器和网络，使得容器的网络模型更加灵活。</p>
<p data-nodeid="9663">CNM 定义的网络标准包含三个重要元素。</p>
<ul data-nodeid="9664">
<li data-nodeid="9665">
<p data-nodeid="9666"><strong data-nodeid="9774">沙箱（Sandbox）</strong>：沙箱代表了一系列网络堆栈的配置，其中包含路由信息、网络接口等网络资源的管理，沙箱的实现通常是 Linux 的 Net Namespace，但也可以通过其他技术来实现，比如 <a href="https://zh.wikipedia.org/wiki/FreeBSD_jail" data-nodeid="9772">FreeBSD jail</a> 等。</p>
</li>
<li data-nodeid="9667">
<p data-nodeid="9668"><strong data-nodeid="9779">接入点（Endpoint）</strong>：接入点将沙箱连接到网络中，代表容器的网络接口，接入点的实现通常是 Linux 的 veth 设备对。</p>
</li>
<li data-nodeid="9669">
<p data-nodeid="9670"><strong data-nodeid="9784">网络（Network</strong>）：网络是一组可以互相通信的接入点，它将多接入点组成一个子网，并且多个接入点之间可以相互通信。</p>
</li>
</ul>
<p data-nodeid="9671">CNM 的三个要素基本抽象了所有网络模型，使得网络模型的开发更加规范。</p>
<p data-nodeid="9672">为了更好地构建容器网络标准，Docker 团队把网络功能从 Docker 中剥离出来，成为独立的项目 libnetwork，它通过插件的形式为 Docker 提供网络功能。Libnetwork 是开源的，使用 Golang 编写，它完全遵循 CNM 网络规范，是 CNM 的官方实现。Libnetwork 的工作流程也是完全围绕 CNM 的三个要素进行的，下面我们来详细了解一下 Libnetwork 是如何围绕 CNM 的三要素工作的。</p>
<h3 data-nodeid="9673">Libnetwork 的工作流程</h3>
<p data-nodeid="9674">Libnetwork 是 Docker 启动容器时，用来为 Docker 容器提供网络接入功能的插件，它可以让 Docker 容器顺利接入网络，实现主机和容器网络的互通。下面，我们来详细了解一下 Libnetwork 是如何为 Docker 容器提供网络的。</p>
<p data-nodeid="9675">第一步：Docker 通过调用 libnetwork.New 函数来创建 NetworkController 实例。NetworkController 是一个接口类型，提供了各种接口，代码如下：</p>
<pre class="lang-java" data-nodeid="9676"><code data-language="java">type NetworkController <span class="hljs-class"><span class="hljs-keyword">interface</span> </span>{
   <span class="hljs-comment">// 创建一个新的网络。 options 参数用于指定特性类型的网络选项。</span>
   NewNetwork(networkType, name string, id string, options ...NetworkOption) (Network, error)
   <span class="hljs-comment">// ... 此次省略部分接口</span>
}
</code></pre>
<p data-nodeid="9677">第二步：通过调用 NewNetwork 函数创建指定名称和类型的 Network，其中 Network 也是接口类型，代码如下:</p>
<pre class="lang-java" data-nodeid="9678"><code data-language="java">type Network <span class="hljs-class"><span class="hljs-keyword">interface</span> </span>{
   <span class="hljs-comment">// 为该网络创建一个具有唯一指定名称的接入点（Endpoint）</span>
   CreateEndpoint(name string, options ...EndpointOption) (Endpoint, error)

   <span class="hljs-comment">// 删除网络</span>
   Delete() error
<span class="hljs-comment">// ... 此次省略部分接口</span>
}
</code></pre>
<p data-nodeid="9679">第三步：通过调用 CreateEndpoint 来创建接入点（Endpoint）。在 CreateEndpoint 函数中为容器分配了 IP 和网卡接口。其中 Endpoint 也是接口类型，代码如下：</p>
<pre class="lang-java" data-nodeid="9680"><code data-language="java"><span class="hljs-comment">// Endpoint 表示网络和沙箱之间的逻辑连接。</span>
type Endpoint <span class="hljs-class"><span class="hljs-keyword">interface</span> </span>{
   <span class="hljs-comment">// 将沙箱连接到接入点，并将为接入点分配的网络资源填充到沙箱中。</span>
   <span class="hljs-comment">// the network resources allocated for the endpoint.</span>
   Join(sandbox Sandbox, options ...EndpointOption) error
   <span class="hljs-comment">// 删除接入点</span>
   Delete(force bool) error
   <span class="hljs-comment">// ... 此次省略部分接口</span>
}
</code></pre>
<p data-nodeid="9681">第四步：调用 NewSandbox 来创建容器沙箱，主要是初始化 Namespace 相关的资源。</p>
<p data-nodeid="9682">第五步：调用 Endpoint 的 Join 函数将沙箱和网络接入点关联起来，此时容器就加入了 Docker 网络并具备了网络访问能力。</p>
<p data-nodeid="9683">Libnetwork 基于以上工作流程可以构建出多种网络模式，以满足我们的在不同场景下的需求，下面我们来详细了解一下 Libnetwork 提供的常见的四种网络模式。</p>
<h3 data-nodeid="9684">Libnetwork 常见网络模式</h3>
<p data-nodeid="9685">Libnetwork 比较典型的网络模式主要有四种，这四种网络模式基本满足了我们单机容器的所有场景。</p>
<ol data-nodeid="9686">
<li data-nodeid="9687">
<p data-nodeid="9688">null 空网络模式：可以帮助我们构建一个没有网络接入的容器环境，以保障数据安全。</p>
</li>
<li data-nodeid="9689">
<p data-nodeid="9690">bridge 桥接模式：可以打通容器与容器间网络通信的需求。</p>
</li>
<li data-nodeid="9691">
<p data-nodeid="9692">host 主机网络模式：可以让容器内的进程共享主机网络，从而监听或修改主机网络。</p>
</li>
<li data-nodeid="9693">
<p data-nodeid="9694">container 网络模式：可以将两个容器放在同一个网络命名空间内，让两个业务通过 localhost 即可实现访问。</p>
</li>
</ol>
<p data-nodeid="9695">下面我们对 libnetwork 的四种网络模式逐一讲解：</p>
<h4 data-nodeid="9696">（1）null 空网络模式</h4>
<p data-nodeid="9697">有时候，我们需要处理一些保密数据，出于安全考虑，我们需要一个隔离的网络环境执行一些纯计算任务。这时候 null 网络模式就派上用场了，这时候我们的容器就像一个没有联网的电脑，处于一个相对较安全的环境，确保我们的数据不被他人从网络窃取。</p>
<p data-nodeid="9698">使用 Docker 创建 null 空网络模式的容器时，容器拥有自己独立的 Net Namespace，但是此时的容器并没有任何网络配置。在这种模式下，Docker 除了为容器创建了 Net Namespace 外，没有创建任何网卡接口、IP 地址、路由等网络配置。我们可以一起来验证下。</p>
<p data-nodeid="9699">我们使用 <code data-backticks="1" data-nodeid="9806">docker run</code> 命令启动时，添加 --net=none 参数启动一个空网络模式的容器，命令如下：</p>
<pre class="lang-c" data-nodeid="9700"><code data-language="c">$ docker run --net=none -it busybox
/ #
</code></pre>
<p data-nodeid="9701">容器启动后，我们使用 <code data-backticks="1" data-nodeid="9809">ifconfig</code> 命令查看一下容器内网络配置信息：</p>
<pre class="lang-c" data-nodeid="9702"><code data-language="c">/ <span class="hljs-meta"># ifconfig</span>
lo        Link encap:Local Loopback
          inet addr:<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>  Mask:<span class="hljs-number">255.0</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
          UP LOOPBACK RUNNING  MTU:<span class="hljs-number">65536</span>  Metric:<span class="hljs-number">1</span>
          RX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> frame:<span class="hljs-number">0</span>
          TX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> carrier:<span class="hljs-number">0</span>
          collisions:<span class="hljs-number">0</span> txqueuelen:<span class="hljs-number">1000</span>
          RX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)  TX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)
</code></pre>
<p data-nodeid="9703">可以看到容器内除了 Net Namespace 自带的 lo 网卡并没有创建任何虚拟网卡，然后我们再使用 <code data-backticks="1" data-nodeid="9812"> route -n</code> 命令查看一下容器内的路由信息:</p>
<pre class="lang-c" data-nodeid="9704"><code data-language="c">/ <span class="hljs-meta"># route -n</span>
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
</code></pre>
<p data-nodeid="9705">可以看到，容器内也并没有配置任何路由信息。</p>
<h4 data-nodeid="9706">（2）bridge 桥接模式</h4>
<p data-nodeid="9707">Docker 的 bridge 网络是启动容器时默认的网络模式，使用 bridge 网络可以实现容器与容器的互通，可以从一个容器直接通过容器 IP 访问到另外一个容器。同时使用 bridge 网络可以实现主机与容器的互通，我们在容器内启动的业务，可以从主机直接请求。</p>
<p data-nodeid="9708">在介绍 Docker 的 bridge 桥接模式前，我们需要先了解一下 Linux 的 veth 和 bridge 相关的技术，因为 Docker 的 bridge 模式正是由这两种技术实现的。</p>
<ul data-nodeid="9709">
<li data-nodeid="9710">
<p data-nodeid="9711">Linux veth</p>
</li>
</ul>
<p data-nodeid="9712">veth 是 Linux 中的虚拟设备接口，veth 都是成对出现的，它在容器中，通常充当一个桥梁。veth 可以用来连接虚拟网络设备，例如 veth 可以用来连通两个 Net Namespace，从而使得两个 Net Namespace 之间可以互相访问。</p>
<ul data-nodeid="9713">
<li data-nodeid="9714">
<p data-nodeid="9715">Linux bridge</p>
</li>
</ul>
<p data-nodeid="9716">Linux bridge 是一个虚拟设备，是用来连接网络的设备，相当于物理网络环境中的交换机。Linux bridge 可以用来转发两个 Net Namespace 内的流量。</p>
<ul data-nodeid="9717">
<li data-nodeid="9718">
<p data-nodeid="9719">veth 与 bridge 的关系</p>
</li>
</ul>
<p data-nodeid="9720"><img src="https://s0.lgstatic.com/i/image/M00/59/ED/Ciqc1F9y8IKAa-1NAABjDM-2kBk665.png" alt="Lark20200929-162853.png" data-nodeid="9825"></p>
<p data-nodeid="9721">通过图 1 ，我们可以看到，bridge 就像一台交换机，而 veth 就像一根网线，通过交换机和网线可以把两个不同 Net Namespace 的容器连通，使得它们可以互相通信。</p>
<p data-nodeid="9722">Docker 的 bridge 模式也是这种原理。Docker 启动时，libnetwork 会在主机上创建 docker0 网桥，docker0 网桥就相当于图 1 中的交换机，而 Docker 创建出的 brige 模式的容器则都会连接 docker0 上，从而实现网络互通。</p>
<p data-nodeid="9723"><strong data-nodeid="9831">bridge 桥接模式是 Docker 的默认网络模式，当我们创建容器时不指定任何网络模式，Docker 启动容器默认的网络模式为 bridge。</strong></p>
<h4 data-nodeid="9724">（3）host 主机网络模式</h4>
<p data-nodeid="9725">容器内的网络并不是希望永远跟主机是隔离的，有些基础业务需要创建或更新主机的网络配置，我们的程序必须以主机网络模式运行才能够修改主机网络，这时候就需要用到 Docker 的 host 主机网络模式。</p>
<p data-nodeid="9726">使用 host 主机网络模式时：</p>
<ul data-nodeid="9727">
<li data-nodeid="9728">
<p data-nodeid="9729">libnetwork 不会为容器创建新的网络配置和 Net Namespace。</p>
</li>
<li data-nodeid="9730">
<p data-nodeid="9731">Docker 容器中的进程直接共享主机的网络配置，可以直接使用主机的网络信息，此时，在容器内监听的端口，也将直接占用到主机的端口。</p>
</li>
<li data-nodeid="9732">
<p data-nodeid="9733">除了网络共享主机的网络外，其他的包括进程、文件系统、主机名等都是与主机隔离的。</p>
</li>
</ul>
<p data-nodeid="9734">host 主机网络模式通常适用于想要使用主机网络，但又不想把运行环境直接安装到主机上的场景中。例如我想在主机上运行一个 busybox 服务，但又不想直接把 busybox 安装到主机上污染主机环境，此时我可以使用以下命令启动一个主机网络模式的 busybox 镜像：</p>
<pre class="lang-c" data-nodeid="9735"><code data-language="c">$ docker run -it --net=host busybox
/ #
</code></pre>
<p data-nodeid="9736">然后我们使用<code data-backticks="1" data-nodeid="9840">ip a</code> 命令查看一下容器内的网络环境：</p>
<pre class="lang-c" data-nodeid="9737"><code data-language="c">/ <span class="hljs-meta"># ip a</span>
<span class="hljs-number">1</span>: lo: &lt;LOOPBACK,UP,LOWER\_UP&gt; mtu <span class="hljs-number">65536</span> qdisc noqueue qlen <span class="hljs-number">1000</span>
link/loopback <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span> brd <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>
inet <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>/<span class="hljs-number">8</span> scope host lo
valid\_lft forever preferred\_lft forever
inet6 ::<span class="hljs-number">1</span>/<span class="hljs-number">128</span> scope host
valid\_lft forever preferred\_lft forever
<span class="hljs-number">2</span>: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER\_UP&gt; mtu <span class="hljs-number">1500</span> qdisc pfifo\_fast qlen <span class="hljs-number">1000</span>
link/ether <span class="hljs-number">02</span>:<span class="hljs-number">11</span>:b0:<span class="hljs-number">14</span>:<span class="hljs-number">01</span>:<span class="hljs-number">0</span>c brd ff:ff:ff:ff:ff:ff
inet <span class="hljs-number">172.20</span><span class="hljs-number">.1</span><span class="hljs-number">.11</span>/<span class="hljs-number">24</span> brd <span class="hljs-number">172.20</span><span class="hljs-number">.1</span><span class="hljs-number">.255</span> scope global dynamic eth0
valid\_lft <span class="hljs-number">85785286</span>sec preferred\_lft <span class="hljs-number">85785286</span>sec
inet6 fe80::<span class="hljs-number">11</span>:b0ff:fe14:<span class="hljs-number">10</span>c/<span class="hljs-number">64</span> scope link
valid\_lft forever preferred\_lft forever
<span class="hljs-number">3</span>: docker0: \&lt;NO-CARRIER,BROADCAST,MULTICAST,UP&gt; mtu <span class="hljs-number">1500</span> qdisc noqueue
link/ether <span class="hljs-number">02</span>:<span class="hljs-number">42</span>:<span class="hljs-number">82</span>:<span class="hljs-number">8</span>d:a0:df brd ff:ff:ff:ff:ff:ff
inet <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>/<span class="hljs-number">16</span> scope global docker0
valid\_lft forever preferred\_lft forever
inet6 fe80::<span class="hljs-number">42</span>:<span class="hljs-number">82f</span>f:fe8d:a0df/<span class="hljs-number">64</span> scope link
valid\_lft forever preferred\_lft forever
</code></pre>
<p data-nodeid="9738">可以看到容器内的网络环境与主机完全一致。</p>
<h4 data-nodeid="9739">（4）container 网络模式</h4>
<p data-nodeid="9740">container 网络模式允许一个容器共享另一个容器的网络命名空间。当两个容器需要共享网络，但其他资源仍然需要隔离时就可以使用 container 网络模式，例如我们开发了一个 http 服务，但又想使用 nginx 的一些特性，让 nginx 代理外部的请求然后转发给自己的业务，这时我们使用 container 网络模式将自己开发的服务和 nginx 服务部署到同一个网络命名空间中。</p>
<p data-nodeid="9741">下面我举例说明。首先我们使用以下命令启动一个 busybox1 容器：</p>
<pre class="lang-c" data-nodeid="9742"><code data-language="c">$ docker run -d --name=busybox1 busybox sleep <span class="hljs-number">3600</span>
</code></pre>
<p data-nodeid="9743">然后我们使用 <code data-backticks="1" data-nodeid="9847">docker exec</code> 命令进入到 centos 容器中查看一下网络配置：</p>
<pre class="lang-c" data-nodeid="9744"><code data-language="c">$ docker exec -it busybox1 sh
/ <span class="hljs-meta"># ifconfig</span>
eth0 Link encap:Ethernet HWaddr <span class="hljs-number">02</span>:<span class="hljs-number">42</span>:AC:<span class="hljs-number">11</span>:<span class="hljs-number">00</span>:<span class="hljs-number">02</span>
inet addr:<span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span> Bcast:<span class="hljs-number">172.17</span><span class="hljs-number">.255</span><span class="hljs-number">.255</span> Mask:<span class="hljs-number">255.255</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
UP BROADCAST RUNNING MULTICAST MTU:<span class="hljs-number">1500</span> Metric:<span class="hljs-number">1</span>
RX packets:<span class="hljs-number">11</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> frame:<span class="hljs-number">0</span>
TX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> carrier:<span class="hljs-number">0</span>
collisions:<span class="hljs-number">0</span> txqueuelen:<span class="hljs-number">0</span>
RX bytes:<span class="hljs-number">906</span> (<span class="hljs-number">906.0</span> B) TX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)

lo Link encap:Local Loopback
inet addr:<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span> Mask:<span class="hljs-number">255.0</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
UP LOOPBACK RUNNING MTU:<span class="hljs-number">65536</span> Metric:<span class="hljs-number">1</span>
RX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> frame:<span class="hljs-number">0</span>
TX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> carrier:<span class="hljs-number">0</span>
collisions:<span class="hljs-number">0</span> txqueuelen:<span class="hljs-number">1000</span>
RX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B) TX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)
</code></pre>
<p data-nodeid="9745">可以看到 busybox1 的 IP 地址为 172.17.0.2。</p>
<p data-nodeid="9746">然后我们新打开一个命令行窗口，再启动一个 busybox2 容器，通过 container 网络模式连接到 busybox1 的网络，命令如下：</p>
<pre class="lang-c" data-nodeid="9747"><code data-language="c">$ docker run -it --net=container:busybox1 --name=busybox2 busybox sh
/ #
</code></pre>
<p data-nodeid="9748">在 busybox2 容器内同样使用 ifconfig 命令查看一下容器内的网络配置：</p>
<pre class="lang-c" data-nodeid="9749"><code data-language="c">/ <span class="hljs-meta"># ifconfig</span>
eth0 Link encap:Ethernet HWaddr <span class="hljs-number">02</span>:<span class="hljs-number">42</span>:AC:<span class="hljs-number">11</span>:<span class="hljs-number">00</span>:<span class="hljs-number">02</span>
inet addr:<span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span> Bcast:<span class="hljs-number">172.17</span><span class="hljs-number">.255</span><span class="hljs-number">.255</span> Mask:<span class="hljs-number">255.255</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
UP BROADCAST RUNNING MULTICAST MTU:<span class="hljs-number">1500</span> Metric:<span class="hljs-number">1</span>
RX packets:<span class="hljs-number">14</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> frame:<span class="hljs-number">0</span>
TX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> carrier:<span class="hljs-number">0</span>
collisions:<span class="hljs-number">0</span> txqueuelen:<span class="hljs-number">0</span>
RX bytes:<span class="hljs-number">1116</span> (<span class="hljs-number">1.0</span> KiB) TX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)

lo Link encap:Local Loopback
inet addr:<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span> Mask:<span class="hljs-number">255.0</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
UP LOOPBACK RUNNING MTU:<span class="hljs-number">65536</span> Metric:<span class="hljs-number">1</span>
RX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> frame:<span class="hljs-number">0</span>
TX packets:<span class="hljs-number">0</span> errors:<span class="hljs-number">0</span> dropped:<span class="hljs-number">0</span> overruns:<span class="hljs-number">0</span> carrier:<span class="hljs-number">0</span>
collisions:<span class="hljs-number">0</span> txqueuelen:<span class="hljs-number">1000</span>
RX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B) TX bytes:<span class="hljs-number">0</span> (<span class="hljs-number">0.0</span> B)
</code></pre>
<p data-nodeid="9750">可以看到 busybox2 容器的网络 IP 也为 172.17.0.2，与 busybox1 的网络一致。</p>
<p data-nodeid="9751">以上就是 Libnetwork 常见的四种网络模式，它们的作用及业务场景帮你总结如下：</p>
<p data-nodeid="9752"><img src="https://s0.lgstatic.com/i/image/M00/59/ED/Ciqc1F9y8HGAaH1iAAClKDUq5FY736.png" alt="Lark20200929-162901.png" data-nodeid="9856"></p>
<h3 data-nodeid="9753">结语</h3>
<p data-nodeid="9754">我上面有说到 Libnetwork 的工作流程是完全围绕 CNM 的三个要素进行的，CNM 制定标准之初不仅仅是为了单台主机上的容器互通，更多的是为了定义跨主机之间的容器通信标准。但是后来由于 Kubernetes 逐渐成为了容器编排的标准，而 Kubernetes 最终选择了 CNI 作为容器网络的定义标准（具体原因可以参考<a href="https://kubernetes.io/blog/2016/01/why-kubernetes-doesnt-use-libnetwork/" data-nodeid="9861">这里</a>），很遗憾 CNM 最终没有成为跨主机容器通信的标准，但是CNM 却为推动容器网络标准做出了重大贡献，且 Libnetwork 也是 Docker 的默认网络实现，提供了单独使用 Docker 容器时的多种网络接入功能。</p>
<p data-nodeid="9755" class="">那你知道 libnetwork 除了我讲的四种网络模式外，还有什么网络模式吗？思考后，把你的想法写在留言区。</p>

---

### 精选评论

##### *李：
> 虽然最后k8s的CNI成为了容器网络标准，但是libnetwork的容器网络模式使得pod中容器共享网络环境成为可能

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，理解的非常正确

##### **成：
> 老师，请教一下dockerfor windows上面，在宿主机无法通过docker 容器的ip访问容器，linux可以，请问这中间是有什么限制吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Windows docker 默认推荐的是通过端口映射的访问, 相关的说明请参考 https://docs.docker.com/docker-for-windows/networking/

##### firzen：
> 在unraid上使用docker时，网络选项有一个Custom:br0的设置，可以获取到一个独立的ip。这是种什么样的模式？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Custom:br0 使得docker容器有独立的ip，虽然共用一个网口，一根网线连路由器，但在路由器看来却像是两台设备在访问，会分配两个ip地址

##### **鹏：
> 谢谢老师

##### *冲：
> 这个libnetwork.New函数在哪里呢？

