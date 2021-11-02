<p data-nodeid="25437">今天是我们第一节课，我想和你聊一聊 Serverless 架构兴起的原因。</p>
<p data-nodeid="25438">Serverless 是最近几年业界很火的技术名词，你可以在国内外各种技术大会上看到它的身影，主流云服务商也不断地推出 Serverless 相关的云产品和新功能（比如 AWS Lambda、阿里云函数计算、腾讯云云函数），各种关于 Serverless 的商业和开源产品也层出不穷（比如 Serverless Framework、OpenFaaS、kubeless）。</p>
<p data-nodeid="25439">在这个背景下，我们开始使用 Serverless 的产品，愿意用它来解决实际问题，比如用 Serverless 技术实现自动化运维、开发小程序、开发服务端应用。</p>
<p data-nodeid="25440"><strong data-nodeid="25541">那你是否思考过 Serverless 为什么这么火呢，换句话说，Serverless 架构兴起的必然因素是什么？</strong> 了解这部分知识，可以让你更好地学习和使用 Serverless，接下来我们就带着这个问题学习今天的内容。</p>
<p data-nodeid="25441">说到 Serverless，就不得不说云计算，因为云计算的发展史就是 Serverless 的兴起史。纵观<strong data-nodeid="25547">云计算的历史，我们可以将其分为物理机时代、虚拟机时代、容器时代、Serverless 时代</strong>，所以接下来，就让我们深入云计算的发展史，去寻找 Serverless 架构兴起的必然因素。</p>
<h3 data-nodeid="25442">物理机时代</h3>
<p data-nodeid="25443"><img alt="图片1.png" src="https://s0.lgstatic.com/i/image2/M01/03/E8/CgpVE1_kTlqASiEcAAFnAFtU2HU531.png" data-nodeid="25551"></p>
<div data-nodeid="25669"><p style="text-align:center"><span style="color:#b8b8b8">物理机时代发展历程</span></p></div>

<p data-nodeid="25445">其实，云计算的概念可以追溯到 60 多年前，最早的说法是图中的“分时操作系统”，也就是通过时间片轮转的方式把一个操作系统给多个用户使用。同时还在发展的是虚拟化技术，也就是把一台物理机隔离为多台虚拟机，这样就能把一个硬件给多个用户使用。</p>
<p data-nodeid="25446">1997 年，Ramnath Chellapa 教授把云计算定义成“一种新的计算范式，其中计算的边界将由经济原理决定，而不仅仅是技术限制”，通俗来讲就是，云计算不只是虚拟化技术，还是云服务商提供计算资源，使用者购买计算资源。</p>
<p data-nodeid="25447">总的来说，在 2000 年之前，互联网刚刚兴起，<strong data-nodeid="25559">而云计算还处于理论阶段，也就是物理机时代</strong>。如果这个时候你想在创业，开发一个电商网站，上线就需要经过以下步骤：</p>
<ul data-nodeid="25448">
<li data-nodeid="25449">
<p data-nodeid="25450">购买一台服务器（物理机）；</p>
</li>
<li data-nodeid="25451">
<p data-nodeid="25452">找个机房并给服务器通上电、连上网线；</p>
</li>
<li data-nodeid="25453">
<p data-nodeid="25454">在服务器上安装操作系统；</p>
</li>
<li data-nodeid="25455">
<p data-nodeid="25456">在服务器上安装数据库和网站环境；</p>
</li>
<li data-nodeid="25457">
<p data-nodeid="25458">部署网站；</p>
</li>
<li data-nodeid="25459">
<p data-nodeid="25460">测试；</p>
</li>
<li data-nodeid="25461">
<p data-nodeid="25462">这时你的网站架构是单机版的单体架构，数据库、应用、Nginx 等服务全都在一台你自己管理的服务器上。</p>
</li>
</ul>
<p data-nodeid="25463"><img alt="图片2.png" src="https://s0.lgstatic.com/i/image2/M01/03/E7/Cip5yF_kTmqAaWsbAAso8KYY52c713.png" data-nodeid="25569"></p>
<div data-nodeid="26134"><p style="text-align:center"><span style="color:#b8b8b8">物理机时代网站部署架构</span></p></div>

<p data-nodeid="25465">但是网站上线之后，你将会遇到各种各样的问题，一旦停电，就会导致网络中断，服务器也会停机，那网站就没办法访问了，用户不能再买买买。于是为了避免断电、断网，你大概率会选择把服务器托管到电信机房，那里停电的概率低很多，但是每个月得多付一些租金。</p>
<p data-nodeid="25466">可是没想到半年后，问题又出现了，服务器 CPU 烧毁了！这不是简单换一台服务器就能解决的事情，原来服务器上的数据如何迁移？新的环境如何与原来保持一致？怎么保证网站持续可用，各种问题接连而至......</p>
<p data-nodeid="25467">总的来说，物理机时代，网站上线和稳定运行面临的最大问题就是服务器等硬件问题，你既要购买服务器，还要承担服务器的场地、电力、网络等开销，并且还需负责服务器的维护。<strong data-nodeid="25576">好在随后几年随着虚拟化技术逐渐成熟，云计算逐渐进入虚拟机时代，这也给我们带来了希望。</strong></p>
<h3 data-nodeid="25468">虚拟机时代</h3>
<p data-nodeid="25469"><img alt="图片3.png" src="https://s0.lgstatic.com/i/image2/M01/03/E8/CgpVE1_kTnSAKGKvAAH0WbMrVNg381.png" data-nodeid="25580"></p>
<div data-nodeid="26599"><p style="text-align:center"><span style="color:#b8b8b8">虚拟机时代发展历程</span></p></div>

<p data-nodeid="25471">上图是虚拟机发展历程中的重要节点。其中一个重要里程碑之一就是 2001 年 VMWare 带来的针对 x86 服务器的虚拟化产品，使虚拟化技术逐渐普及。对云厂商来说，通过虚拟化技术，它可以把一台物理机分割成多台虚拟机提供给多用户使用，充分利用硬件资源，而且创建速度和弹性也远超物理机。对于开发者来说，就不用再买硬件了，直接在云平台买虚拟机，成本更低了。</p>
<p data-nodeid="25472"><strong data-nodeid="25586">2001 年之后，虚拟化技术日渐成熟，</strong> 因此也出现了很多基于虚拟化的云厂商和产品（开篇我也提到了）。最初云厂商都是卖硬件，AWS 的 EC2、阿里云 ECS、Azure Virtual Machines，这种云计算形态也被叫作 IaaS（基础设施即服务）。后面随着业务形态发展，云厂商发现可以抽象出一些通用的平台，比如中间件、数据库等，于是就把这些功能做成服务，也放在云上去卖，这就是 PaaS（平台即服务）。</p>
<p data-nodeid="25473"><strong data-nodeid="25591">那有了 IaaS 之后，你就可以把电商网站迁移到虚拟机上了，</strong> 再也不用担心断电断网和硬件故障。不过，当你的电商网站越做越强，用户越来越多，数据库每天都有几千万条数据写入，数据库性能很快就会达到瓶颈，就会出现用户因付款太慢放弃付款的情况；除此之外，每天也有上百万图片存到磁盘，磁盘也快要耗尽了。如果网站出现崩溃，就直接导致用户流失，甚至资损。</p>
<p data-nodeid="25474">好在作为专业的技术人员，我们已经对云计算了如指掌，为了避免这些问题，基于 IaaS 和 PaaS 重新设计了网站架构：</p>
<p data-nodeid="25475"><img alt="图片4.png" src="https://s0.lgstatic.com/i/image2/M01/03/E8/CgpVE1_kTn2AeXnYABSdMxS0Ddo391.png" data-nodeid="25595"></p>
<div data-nodeid="27064"><p style="text-align:center"><span style="color:#b8b8b8">虚拟机时代网站部署架构</span></p></div>

<p data-nodeid="25477">为了降低服务器负载，我们把数据库迁移到了云厂商提供的云数据库上，把图片存储迁移到对象存储：</p>
<ul data-nodeid="25478">
<li data-nodeid="25479">
<p data-nodeid="25480">云数据库有专门的服务器，并且还提供了备份容灾，比自己在服务器上安装数据库更稳性能更强。</p>
</li>
<li data-nodeid="25481">
<p data-nodeid="25482">对象存储能无限扩容，不用担心磁盘不够了。</p>
</li>
</ul>
<p data-nodeid="25483">这样一来，服务器就只负责处理用户的请求，把计算和存储分离开来，既降低了系统负载，也提升了数据安全性。并且单机应用升级为了集群应用，通过负载均衡，会把用户流量均匀分配到每台服务器上。</p>
<p data-nodeid="25484"><strong data-nodeid="25604">不过在服务器扩容的过程中，你还是会遇到一些麻烦。</strong> 比如购买服务器时，会发现之前服务器型号没有了，只有新的型号，并且每次新扩容一台服务器，都需要在上面初始化软件环境和配置，还需要保证所有服务器运行环境一致，这是个非常复杂还容易出错的工作。</p>
<p data-nodeid="25485">总的来说，虚拟机可以让你不用关心底层硬件，但是如果能让我们不用关心运行环境就更好了。<strong data-nodeid="25609">于是，容器技术诞生了。</strong></p>
<h3 data-nodeid="25486">容器时代</h3>
<p data-nodeid="25487"><img alt="图片5.png" src="https://s0.lgstatic.com/i/image2/M01/03/E8/CgpVE1_kToWARsK1AAF3_oNUZVE232.png" data-nodeid="25613"></p>
<div data-nodeid="27529"><p style="text-align:center"><span style="color:#b8b8b8">容器时代发展历程</span></p></div>

<p data-nodeid="25489"><strong data-nodeid="25618">2013 年 Docker 的发布，代表着容器技术替代了虚拟化技术，云计算进入容器时代。</strong> 容器就是把代码和运行环境打包在一起，这样代码就可以在任何地方运行。有了容器技术，你在服务器上部署的就不再是应用了，而是容器。当容器多了的时候，如何管理就成了一个问题，于是出现了容器编排技术，比如 2014 年 Google 开源的 Kubernetes。</p>
<p data-nodeid="25490">基于容器，你部署网站的方式也有了改变：</p>
<ul data-nodeid="25491">
<li data-nodeid="25492">
<p data-nodeid="25493">搭建 Kubernetes 集群；</p>
</li>
<li data-nodeid="25494">
<p data-nodeid="25495">构建容器镜像；</p>
</li>
<li data-nodeid="25496">
<p data-nodeid="25497">部署镜像。</p>
</li>
</ul>
<p data-nodeid="25498">你的网站部署架构也演进得更现代化了：</p>
<p data-nodeid="25499"><img alt="图片6.png" src="https://s0.lgstatic.com/i/image2/M01/03/E7/Cip5yF_kTo2ASrz9AAi8F5WTUZQ237.png" data-nodeid="25626"></p>
<div data-nodeid="27994"><p style="text-align:center"><span style="color:#b8b8b8">容器时代网站部署架构</span></p></div>

<p data-nodeid="25501">你不仅使用了容器，你还使用了 Kubernetes 来做管理容器集群。基于 Kubernetes 和云厂商提供的弹性能力，你可以实现网站的自动弹性伸缩。这样在流量洪峰到来时，就可以自动弹出更多的资源；当流量低谷时，自动释放多余的资源。</p>
<p data-nodeid="25502">想到这儿，你多多少少有些兴奋，但时间一久，问题也随之出现。因为你需要去规划节点和 Pod 的 CPU、内存、磁盘等资源，需要编写复杂的 YAML 去部署 Pod、服务，需要经常排查 Pod 出现的异常，需要学习专业的运维知识，<strong data-nodeid="25633">渐渐地，你好像变成了 Kubernetes 运维工程师，</strong> 如果能完全不关心运维，只专注于产品的开发就好了，这样能节省很多时间，以更快的速度完成产品迭代上线。</p>
<p data-nodeid="25503">而且你也没想到，由于提前准备不够充分，双十一来的时候，零点的订单量远超预期，网站又崩了！集群虽然感知到了需要弹出更多的资源，但由于服务器弹出需要一定时间，没来得及应对这种瞬时流量，要是能够支持秒级弹性就好了。<strong data-nodeid="25638">于是，Serverless 时代来临了。</strong></p>
<h3 data-nodeid="25504">Serverless 时代</h3>
<p data-nodeid="25505"><img alt="图片7.png" src="https://s0.lgstatic.com/i/image2/M01/03/E8/CgpVE1_kTpSAK_EgAAD8YeiJPZk736.png" data-nodeid="25642"></p>
<div data-nodeid="28459"><p style="text-align:center"><span style="color:#b8b8b8">Serverless 时代发展历程</span></p></div>

<p data-nodeid="25507">在我看来，Serverless 是指构建和运行不需要服务器管理的一种概念。前面三种电商网站部署的方式，都属于 Serverful 的架构，它就像使用低级的汇编语言编程。而 Serverless 的架构就像使用 Python 这样的高级语言进行编程。比如 c = a + b 这样简单的表达式，如果用汇编描述，就必须先选择几个寄存器，把值加载到寄存器，进行数学计算，再存储结果。这就好比今天在 Serverful 的架构下，开发首先需要分配或找到可用的资源，然后加载代码和数据，再执行计算，将计算的结果存储起来，最后还需要管理资源的释放。</p>
<p data-nodeid="25508">对于 Serverless，目前我们得到的还是一个比较抽象的概念，这是因为这项技术尚处于发展阶段。现阶段关于 Serverless 的实现主要是基于 FaaS（函数即服务） 和 BaaS （后端即服务）的方案。</p>
<ul data-nodeid="25509">
<li data-nodeid="25510">
<p data-nodeid="25511">FaaS 提供了运行函数代码的能力，并且具有自动弹性伸缩。基于 FaaS，我们应用的组成就不再是集众多功能于一身的集合体，而是一个个独立的函数。每个函数实现各自的业务逻辑，由这些函数组成复杂的应用。</p>
</li>
<li data-nodeid="25512">
<p data-nodeid="25513">BaaS 是将后端能力封装成了服务，并以接口的形式提供服务。比如数据库、文件存储等。通过 BaaS 平台的接口，我们运行在 FaaS 中的函数就能调用各种后端服务，进而以更低开发成本实现复杂的业务逻辑。</p>
</li>
</ul>
<p data-nodeid="25514">基于 Serverless，<strong data-nodeid="25651">我们的电商网站部署架构就演变为了下面这样：</strong></p>
<p data-nodeid="25515"><img alt="图片8.png" src="https://s0.lgstatic.com/i/image2/M01/03/E7/Cip5yF_kTp2AOUW_ABVeKCaUwYU699.png" data-nodeid="25654"></p>
<div class="te-preview-highlight" data-nodeid="28924"><p style="text-align:center"><span style="color:#b8b8b8">Serverless 时代部署架构图</span></p></div>

<p data-nodeid="25517">我们通过网关承接用户流量，并将流量转发到在 FaaS 平台运行的函数中。每个函数都是一个特定的接口，实现单一业务逻辑。并且基于 BaaS 实现复杂业务功能。而函数本身，也还可以调用其他微服务。基于这样的架构，我们就完全不用关心运维，并且 FaaS 平台的弹性伸缩能力，就能实现业务的秒级弹性。</p>
<p data-nodeid="25518">总的来说，基于 Serverless开发者就只需要关心业务逻辑的开发。进行应用部署时也不再需要关心服务器，不需要关心后续的运维，应用也天然具备了弹性伸缩的能力，并且实现了按需使用，按量付费，也更能进一步节省成本。</p>
<h3 data-nodeid="25519">小结</h3>
<p data-nodeid="25520">通过上面的学习，相信你对云计算的发展历程，也是 Serverless 的兴起过程有了一个大概的了解。让我们再围绕网站的部署架构进行简单的回顾。</p>
<ul data-nodeid="25521">
<li data-nodeid="25522">
<p data-nodeid="25523">物理机时代：2000 年之前，我们需要通过物理机部署网站。</p>
</li>
<li data-nodeid="25524">
<p data-nodeid="25525">虚拟机时代：2000 年之后，虚拟化技术发展成熟，云计算行业蓬勃发展，我们可以基于 IaaS 和 PaaS 部署应用，提高稳定性。</p>
</li>
<li data-nodeid="25526">
<p data-nodeid="25527">容器时代：2013 年云计算进入容器时代，我们可以通过容器技术打包应用及运行依赖，不用关心运行环境。</p>
</li>
<li data-nodeid="25528">
<p data-nodeid="25529">Serverless 时代：最近几年，云计算进入 Serverless 时代，我们不再需要关心服务器，应用也天然具有弹性。</p>
</li>
</ul>
<p data-nodeid="25530">纵观云计算的发展史，从物理机到虚拟机，从 IaaS、PaaS 到 FaaS，从容器到 Serverless，都是一个去服务器的一个过程。有了 IaaS，我们不需要关注物理机；有了 PaaS，我们不需要关注操作系统；有了容器，我们不需要关心运行环境；而 Serverless 技术的出现，能够让我们不再关心传统的运维工作，让我们更专注于业务的实现，把时间精力花在更有意义的事情上，让我们以更快的速度、更低的成本完成应用的开发迭代，进而创造出更大的价值。而这也正是 Serverless 架构兴起的必然因素。</p>
<p data-nodeid="25531"><img alt="Lark20201224-184403.png" src="https://s0.lgstatic.com/i/image/M00/8C/12/CgqCHl_kcRCAcyqiAAFbdq9wbQM393.png" data-nodeid="25666"></p>
<p data-nodeid="25532">另外，我希望你学完今天的内容可以对云计算的发展历史有了初步的了解，对 Serverless 架构的兴起过程也有初步的认识。</p>
<p data-nodeid="25533">今天的作业是希望你能在课下做功课，总结一下，IaaS、PaaS、SaaS、FaaS、BaaS 它们到底是什么，有什么区别？这会帮你更好地理解今天的内容，欢迎在留言区与我互动。当然了，我今天只是简单地提了一下 Serverless 是什么，还未对它的概念展开来讲解，下一节课，我会深入剖析Serverless的含义，我们下节课见。</p>

---

### 精选评论

##### **的小叶酱：
> 更关注Serverless如何做到只需要关注业务

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢同学你的留言，我说一下我的想法：

因为 Serverless 把业务运行所依赖的底层计算、存储、网络等基础设施都隐藏起来了，这些部分全都由 Serverless 平台进行调度和管理。

在以往我们开发一个应用，上线前都需要先购买服务器、初始化服务器环境、配置网络环境等，上线后还需要时负责这些底层资源的的维护。当业务复杂后，还需要实现弹性伸缩、负载均衡等复杂功能。但基于 Serverless 技术，这些都不用开发者关注，因为这些都是 Serverless 的基本能力，所以我们可以只关注业务了。

欢迎多多在留言区互动交流哦

##### **辉：
> 我还处于虚拟机时代😂

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 努努力就到Serverless 时代了哦~欢迎随时在留言区互动留言~

##### **4562：
> 那底层的那些机器是由谁来维护呢？如果他们不支持秒级扩容，那一切都是空谈？或者说serverless一定是建立在容器的基础上吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢同学你的留言，我说一下我的看法：

我们使用 Serverless 有两个途径，一是使用公有云 Serverless 产品，二是自己私有化部署 Serverless 平台。本节课主要指的是前者。如果使用公有云 Serverless 产品，底层机器全都由云厂商维护。Serverless 的弹性能力也有云厂商实现，如果云厂商不支持秒级弹性，我们的应用也就没有弹性能力了。但这个不用担心，因为秒级弹性是 Serverless 的核心能力和主要竞争力之一，主流云厂商的 Serverless 产品（如 Lambda、函数计算等）都支持。

如果你不想使用公有云上的 Serverless 产品，你也可以在自己的服务器集群上部署 Serverless 平台（私有化部署），但这样就需要自己维护底层机器了，运维成本和经济成本都很高。

相比私有化部署 Serverless 平台，使用云厂商提供的公有云上的 Serverless，可以充分利用公有云的弹性，因为在公有云上你基本不用关心服务器不够的情况。此外你也能完全不用关心底层机器运维了。缺点就是你需要依赖云厂商。当然，在云计算的发展过程中，我们的应用就是逐步从线下迁移到云上，我们本身就很依赖云厂商，Serverless 则是提供了新的方式让我们更合理地使用云计算。就算不依赖云，我们也需要依赖服务器、机房、网络等。

希望我的回答能使你满意哦

##### **8621：
> 为什么前端和FaaS之间需要加一个网关来承接用户流量，是为了负载均衡？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为现在很多 FaaS （比如 AWS Lambda、腾讯云云函数）的 HTTP 触发器实现，是基于网关来实现的，而阿里云函数计算是直接提供 HTTP 触发器。在 FaaS 上层的网关，主要作用是通过网关来驱动函数执行，主要是用于 Web 场景。基于网关，就可以实现你说的负载均衡以及流量控制、白黑名单、权限校验等复杂功能了。

##### *旨：
> Serverless会取代k8s时代，成为下一代主流么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好问题。答案是肯定的。因为目前使用 k8s 的成本还比较高，还需要开发者自己去维护集群，编写繁多的 YAML 来部署应用。在 02 讲中我也提到了，我觉得 k8s 是介于 Serverful 和 Serverless 中间的一种架构。一是 k8s 有一定运维成本，而是 k8s 的付费模式还是按应用运行所需要的资源数量来付费，而不是按应用运行实际使用资源来付费。成本和效率一直所开发者和企业关注的问题，Serverless 的理念就是为了解决这两个问题，所以我认为 Serverless 一定会成文下一代的主流技术架构。

##### **4418：
> SaaS这里好像没提到过啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好问题。因为是按照应用部署架构演进的时间线来介绍的，而 SaaS 的话，是厂商直接提供软件服务，用户花钱购买使用，也就不用自己设计网站架构了，所以没有拿出来介绍，刚好借此问题简单补充一下。使用 SaaS 的优点就是方便简单，缺点也很明显，定制差。SaaS 在云计算早起就出现了，第一家通过互联网提供应用程序的公司是 Salesforce.com，它在 1999 年就成了。

##### **2498：
> 使用公有云产品构建serverless只适合个人或那些中小型公司吧?像政府机关或大型国有企业，他们都有自建机房的，而且各种业务系统都是在内网运行，不可能访问到互联网。这种情况下只能自建后端服务了吧。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 自建机房的企业可以私有化部署 Serverless 平台。有的云厂商也提供了私有化的 Serverless 解决方案，比如 Azure Functions，既支持公有云，也支持私有云。有能力的企业也可以使用开源产品（如 OpenFaaS、Fission 等）或自研 Serverless 平台。不过整体我觉得 Serverless 要有足够的规模，才能更大发挥出它的价值。

##### *河：
> 请教一下，Serverless 是微服务的扩展，还是容器的延伸呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这三者是不同纬度的概念。微服务是一种软件架构模式，我们可以使用 Serverless 来实现微服务，比如一个函数就是一个微服务。目前绝大多数 Serverless 平台（如公有云的 Lambda、函数计算及开源的 Fisson、OpenFaaS 等）底层实现，都是基于容器技术的，Serverless 平台可以基于容器技术实现函数的资源隔离、CPU 和内存限制等。

##### **斌：
> 请教一下，baas与paas的区别在哪里？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; BaaS 是简化应用开发，PaaS 是简化应用部署。BaaS 抽象了开发者需要自建的各种后端服务，比如我们要实现用户身份认证，首先就需要一个数据库来存用户信息，然后开发认证代码，如果使用 BaaS，这些都不用我们开发了，只需要调 BaaS 接口。PaaS 主要提供了一个开发和部署平台，负责执行代码和管理应用。

##### **东：
> 请教老师：k8s也有自己的弹性什么为什么说不支持秒级弹性伸缩呢，假如我的serverless底层用的就是k8s的pod，他能做到不就代表k8s也能做到吗？不太理解，望解答

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 主要是 Serverless 平台对底层 Pod 或容器启动做了优化。一方面是优化了容器本身的启动速度，举个例子，通常一个 Docker 容器的启动极限速度是 200ms 左右，而 Lambda 的 Node.js 运行环境冷启动已经优化到了 100 多毫秒；另一方面是通过容器重用、执行上下文重用等方案，尽可能避免从零初始化一个容器，这样 Serverless 函数就可以实现几十毫秒甚至几毫秒的弹性伸缩。而 k8s 的弹性有两种，一种是 Pod 的伸缩，快的话可能几秒钟，这与镜像大小、网络环境等因素有关，并且每次 Pod 扩容都是创建一个全新的 Pod；另一种是节点的伸缩，需要创建新的机器，通常需要几分钟，而 Serverless 则不需要我们关心机器。所以整体上，我认为 Serverless 是秒级弹性伸缩，而 k8s 是分钟级。

##### *伟：
> faas和saas实现上是可以在一个服务上吗，就是不用搞一个faas的服务，再搞个saas的服务

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; FaaS 和 SaaS 是可以分开使用的。FaaS 是函数运行平台，你需要编写自己的函数代码，然后部署到 FaaS 上运行。而 SaaS 是软件即服务，是厂商提供开发好的软件，使用者花钱购买使用。

##### **波：
> 大厂搭建自己的机房，k8s这种成熟体系可以自己应用，但是serverless这种自己搞成本会不会过高，但是又绝对不会把自己的数据交给云服务商

##### **汉：
> 我感觉这有点像计算机的分层概念，把负责的功能抽象成一层，每层只负责单一的功能。然而Serverless不是终点，随着互联网需求的发展肯定还会有新的层次出来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 很独特的视角。我觉得技术的发展始是围绕成本和效率进行的，我们也始终有对更高效率和更低成本的追求，所以一定会有更新的技术不断涌现。

##### *中：
> Faas弹性扩容针对业务层，但实际web应用开发中，性能瓶颈在Backend ，如何弹性扩容

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我理解 Backend 应该指的是云厂商提供的 BaaS？BaaS 性能瓶颈，的确是现在基于 FaaS 和 BaaS 这种 Serverless 实现面临的一个问题。基于 FaaS 我们可以实现计算资源的自动弹性伸缩，而 BaaS 多种多样，存储的弹性伸缩面临的挑战也更大。使用 BaaS，我们就需要依赖 BaaS 服务的能力，有些 BaaS 比如 DynamoDB、表格存储等 NoSQL 都支持自动弹性扩容，而 RDS 则不具备弹性能力，这时就需要人工根据业务流量进行容量规划了。

