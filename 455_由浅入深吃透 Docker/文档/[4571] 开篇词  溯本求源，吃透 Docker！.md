<p data-nodeid="541" class="">你好，我是你的 Docker 老师——郭少。</p>
<p data-nodeid="542">我是从 2015 年开始使用和推广容器技术的，算是国内首批容器践行者。当时，我还在从事 Java 业务开发，业务内部的微服务需要做容器化改造，公司首席架构师牵头成立了云平台组，很荣幸我被选入该小组，从此我认识了 Docker。</p>
<p data-nodeid="543">刚开始接触时，我十分惊叹于 Docker 竟同时拥有业务隔离、软件标准交付等特性，而且又十分轻量，和虚拟机相比，容器化损耗几乎可以忽略不计。</p>
<p data-nodeid="544">接下来 5 年多的时间，我便在容器领域深耕，帮助过多家企业实现业务容器化，其间曾经在 360 推广容器云技术，实现了单集群数万个容器的规模，同时设计和开发了 Kubernetes 多集群管理平台 Wayne（有多家公司将 Wayne 用于生产环境）。2019 年，我被 CNCF 邀约作为嘉宾分享容器化实践经验，那时容器已经成为云计算的主流，以容器为代表的云原生技术，已经成为释放云价值的最短路径。</p>
<p data-nodeid="545">而在平时工作中，我仍然发现很多人在学习和实践 Docker 时，并非一路坦途：</p>
<ul data-nodeid="546">
<li data-nodeid="547">
<p data-nodeid="548">学习 Docker 会顾及较多，比如，我不会 Golang 怎么办？Linux 懂一点行吗？</p>
</li>
<li data-nodeid="549">
<p data-nodeid="550">对 Docker 的知识掌握零零碎碎，不系统，说自己懂吧，但好像也懂得不多，还是经常会查资料。</p>
</li>
<li data-nodeid="551">
<p data-nodeid="552">自己对 Docker 底层原理理解欠缺，核心功能掌握不全，遇到问题时无法定位，耽误时间。</p>
</li>
<li data-nodeid="553">
<p data-nodeid="554">不知道如何使用 Docker 提升从开发到部署的效率？</p>
</li>
<li data-nodeid="555">
<p data-nodeid="556">不同场景下，如何选用最适合的容器编排和调度框架？</p>
</li>
</ul>
<p data-nodeid="557">这些境遇恰是我曾走过的路，对此我也有很多感悟和思考，因此也一直希望有机会分享出来，这个课程正好是一个契机，相信我在这个行业实践的一些方法和思路能给你带来很多启发和帮助。</p>
<h3 data-nodeid="558">我是如何学习 Docker 的？</h3>
<p data-nodeid="559">当今，Docker 技术已经形成了更为成熟的生态圈，各家公司都在积极做业务容器化改造，大家对 Docker 也都已经不再陌生。但在我刚接触 Docker 时，市面上的资料还非常少，甚至官网的资料也不太齐全。为了更深入地学习和了解 Docker，我只能从最笨但也最有效的方式入手，也就是读源码。</p>
<p data-nodeid="560">为什么说这是最笨的方法？因为想研究 Docker 源码，就意味着我需要学习一门新的编程语言 —— Golang。</p>
<p data-nodeid="561">虽然我当时已经掌握了一些编程语言，比如 Java、Scala、C 等，但对 Golang 的确十分陌生。好在 Golang 属于类 C 语言，当时我一边研究 Docker 源码，一边学习 Golang 语法。虽然学习过程有些艰辛，但结果很好。我只用了一周左右，便熟悉了这门新的编程语言，并从此与 Golang 和 Docker 结下了不解之缘。这可以说是我的另一层意外收获。</p>
<p data-nodeid="562">然而，在学习 Docker 源码的过程中我又发现，想要彻底了解 Docker 的底层原理，必须对 Linux 相关的技术有一定了解。例如，我们不了解 Linux 内核的 Cgroups 技术，就无法知道容器是如何做资源（CPU、内存等）限制的；不了解 Linux 的 Namespace 技术，就无法知道容器是如何做主机名、网络、文件等资源隔离的。</p>
<p data-nodeid="563">我记得有一次在生产环境中，告警系统显示一台机器状态为 NotReady，业务无法正常运行，登录机器发现运行<code data-backticks="1" data-nodeid="612">docker ps</code>命令无响应。这是当时线上 Docker 版本信息：</p>
<pre class="lang-shell" data-nodeid="564"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker -v</span>
<span class="hljs-meta">$</span><span class="bash"> Docker version 17.03.2-ce, build f5ec1e2</span>
<span class="hljs-meta">$</span><span class="bash"> docker-containerd -v</span>
<span class="hljs-meta">$</span><span class="bash"> containerd version 0.2.3 commit:4ab9917febca54791c5f071a9d1f404867857fcc</span>
<span class="hljs-meta">$</span><span class="bash"> docker-runc -v</span>
<span class="hljs-meta">$</span><span class="bash"> runc version 1.0.0-rc2</span>
<span class="hljs-meta">$</span><span class="bash"> commit: 54296cf40ad8143b62dbcaa1d90e520a2136ddfe</span>
<span class="hljs-meta">$</span><span class="bash"> spec: 1.0.0-rc2-dev</span>
</code></pre>
<p data-nodeid="565">这里简单介绍下我当时的排查过程。</p>
<p data-nodeid="566">我首先打开 Docker 的调试模式，查看详细日志，我根据调试日志去查找对应的 Docker 代码，发现是 dockerd 请求 containerd 无响应（这里你需要知道 Docker 组件构成和调用关系），然后发送 Linux<code data-backticks="1" data-nodeid="616">SIGUSR1</code>信号量（这里你需要知道 Linux 的一些信号量），打印 Golang 堆栈信息（这里你需要了解 Golang 语言）。最后结合内核 Cgroups 相关日志（这里你需要了解 Cgroups 的工作机制），才最终定位和解决问题。</p>
<p data-nodeid="567">可以看到，排查一个看起来很简单的问题就需要用到非常多的知识，<strong data-nodeid="622">首先需要理解 Docker 架构，需要阅读 Docker 源码，还得懂一些 Linux 内核问题才能完全定位并解决问题。</strong></p>
<p data-nodeid="568">相信大多数了解 Docker 的人都知道，Docker 是基于 Linux Kernel 的 Namespace 和 Cgroups 技术实现的，但究竟什么是 Namespace？什么是 Cgroups？容器是如何一步步创建的？很多人可能都难以回答。你可能在想，我不用理会这些，照样可以正常使用容器呀，但如果你要真正在生产环境中使用容器，你就会发现如果不了解容器的技术原理，生产环境中遇到的问题你很难轻松解决。所以，<strong data-nodeid="628">仅仅掌握容器的一些皮毛是远远不够的，需要我们了解容器的底层技术实现，结合生产实践经验，才能让我们更好地向上攀登</strong>。</p>
<p data-nodeid="569">当然，我知道每个人的基础都不一样，所以在一开始规划这个课程的时候，我就和拉勾教育的团队一起定义好了我们的核心目标，就是“由浅入深带你吃透 Docker”，希望让不同基础的人都能在这个课程中收获满满。</p>
<h3 data-nodeid="570">送你一份“学习路径”</h3>
<p data-nodeid="938">接下来，是我们为你画出的一个学习路径，这也是我们课程设计的核心。</p>
<p data-nodeid="939" class=""><img src="https://s0.lgstatic.com/i/image/M00/4C/CA/Ciqc1F9YoBKAP5TpAAHqwwYYWWc486.png" alt="11.png" data-nodeid="943"></p>


<p data-nodeid="573">用一句话总结，我希望这个课程从 Docker<strong data-nodeid="648">基础知识点</strong>到<strong data-nodeid="649">底层原理</strong>，再到<strong data-nodeid="650">编排实践</strong>，层层递进地展开介绍，最大程度帮你吸收和掌握 Docker 知识。</p>
<ul data-nodeid="574">
<li data-nodeid="575">
<p data-nodeid="576"><strong data-nodeid="654">模块一：基础概念与操作</strong></p>
</li>
</ul>
<p data-nodeid="577">在模块一，我首先会带你了解 Docker 基础知识以及一些基本的操作，比如拉取镜像，创建并启动容器等基本操作。这样可以让你对 Docker 有一个整体的认识，并且掌握 Docker 的基本概念和基本操作。这些内容可以满足你日常的开发和使用。</p>
<ul data-nodeid="578">
<li data-nodeid="579">
<p data-nodeid="580"><strong data-nodeid="659">模块二：底层实现原理及关键技术</strong></p>
</li>
</ul>
<p data-nodeid="581">在对 Docker 有个基本了解后，我们就进入重点部分—— Docker 的实现原理和关键性技术。比如，Namespace 和 Cgroups 原理剖析，Docker 是如何使用不同覆盖文件系统的（Overlay2、AUFS、Devicemapper），Docker 的网络模型等。当然，在这里我会趁热打铁，教你动手写一个精简版的 Docker，这能进一步加深你对 Docker 原理的认知。学习这些知识可以让你在生产环境中遇到问题时快速定位并解决问题。</p>
<ul data-nodeid="582">
<li data-nodeid="583">
<p data-nodeid="584"><strong data-nodeid="664">模块三：编排技术三剑客</strong></p>
</li>
</ul>
<p data-nodeid="585">仅仅有单机的容器只能解决基本的资源隔离需求，真正想在生产环境中大批量使用容器技术，还是需要有对容器进行调度和编排的能力。所以在这时，我会从 Dcoker Compose 到 Docker Swarm 再到 Kubernetes，一步步带你探索容器编排技术，这些知识可以让你在不同的环境中选择最优的编排框架。</p>
<ul data-nodeid="586">
<li data-nodeid="587">
<p data-nodeid="588"><strong data-nodeid="669">模块四：综合实战案例</strong></p>
</li>
</ul>
<p data-nodeid="589">在对容器技术原理和容器编排有一定了解后，我会教你将这些技术应用于 DevOps 中，最后会通过一个 CI/CD 实例让你了解容器的强大之处。</p>
<p data-nodeid="590">我希望这样的讲解框架，既能让你巩固基础的概念和知识，又能让你对 Docker 有更深一步的认识，同时也能让你体会容器结合编排后的强大力量。最重要的是，你不用再自己去研究这么多繁杂的技术点，不用再自己去头痛地读源码，因为这些事情正好我都提前帮你做了。</p>
<h3 data-nodeid="591">寄语</h3>
<p data-nodeid="592">现阶段，很多公司的业务都在使用容器技术搭建自己的云平台，使用容器云来支撑业务运行也成为一种趋势，所以公司都会比较在意业务人员对 Docker 的掌握情况。那我希望这个课程，能够像及时雨一样，帮你彻底解决 Docker 相关的难题。</p>
<p data-nodeid="593">如果说，我们已经错过了互联网技术大爆发的时代，也没有在以虚拟机为代表的云计算时代分得一杯羹。那么，这次以 “容器” 为代表的历史变革正呼之欲出，你又有什么理由错过呢？</p>
<p data-nodeid="594" class="">好了，我说了这么多，最后我也希望听你来说一说，告诉我：你在学习 Docker 的路上踩到过哪些坑？你在Docker的使用中又有哪些成功的经验，可以分享给大家？写在留言区，我们一起交流。</p>

---

### 精选评论

##### **6342：
> Wayne是开源的吗？有什么介绍资料吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Wayne 是开源的，项目地址为 https://github.com/Qihoo360/wayne

##### *羊：
> 买了K8s，又来买 Docker 的筒子们举手。

##### **爱我：
> Docker+K8s ，拉勾这能的挺全啊，感觉我这 VIP 值了。

##### **弟：
> 学Docker，感觉上手比较容易，原理是硬伤，期待这个内容

##### **7036：
> 老师你的课件做的非常高大上，用什么做的呢？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; PPT

##### **9050：
> 公司搬家，测试服务器断电，docker怎么都起不来，最后重装了一下环境，搞了2天时间

##### *帅：
> 老师，wayne 是不是内部支持力度不够了，一年半了，还是不支持k8s 1.16以上的版本

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 老师从 360 离开了，后面 Wayne 的就少了，Wayne 已经做了新的架构，功能更加丰富。后面有重新开源的计划

##### **web前端：
> 像webassembly会不会蚕食docker的市场，能不能比较下相关的替代技术

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; webassembly 和 Docker 没有本质的替代关系，市场交叉不大，Docker 的可以替代品为 podman，containerd 等项目

##### **强：
> 同样是2015年接触docker 为什么你如此优秀😂

##### *邪：
> 刚刚转到做后台开发，是云计算方向，这块知识相对薄弱，正巧借这个课程去全面系统的学习一个Docker技术，了解其原理，加油

##### **文：
> Docker😀

##### *鹅：
> 现在很多公司项目部署都用 K8S + Docker ，面试被问的概率确实有点高，如果被问到，有点懵 Docker ，基本是没戏。

##### Rayom：
> 进来就有喜欢的Docker！yes

##### **6595：
> 开始整起来

##### **天：
> 这门课覆盖面挺全的呀

##### **友：
> 2021-03-24 肝起来

##### *俊：
> 初学

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 欢迎欢迎，记得坚持喔，很多同学都学完了，fitting！

##### **溢：
> 会讲dockerfiel怎么写吗？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 同学，06课时就是讲dockerfiel的最佳实践喔！

##### **鑫：
> 对容器内部分配的内存盘与宿主机的挂载，不是很清楚😀😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哪里不清楚的可以详细提出问题哈~

##### *霖：
> 正好公司再用docker，还是我负责管理，现在只会简单的应用，缺乏处理经验，并且需要管理好几十个，也不知道怎么管理好，此课真是及时雨。现在公司领导说docker不安全，是这样的么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 安全只是相对的，私有云 Docker 安全性完全足够。公有云可以考虑安全容器。

##### *睿：
> 老师docker和现在逐渐形成的serverless趋势相比有什么本质的优势呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Docker和 Serverless 不是冲突的概念，大多数 Serverless 是通过 Docker 实现的。

##### *火：
> 听了老师的讲解，才发现自己对docker的了解只是皮毛

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢支持，加油加油！

##### **民：
> 开始系统学习

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油！奥利给！

##### **福：
> 小鲸鱼，k8s，你看这套餐行不行😂😂

##### **定：
> 我在mac中安装了docker桌面版，但是docker-compose经常找不到，需要重新安装

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Mac 下的 Docker 默认是自带 docker-compose 的哈，可以尝试升级或者重新安装 Docker

##### **将：
> 老师，请教一下：本专栏会结合Docker源码进行讲解嘛？如果涉及的话，有什么好的博客书籍，可以推荐一下嘛？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 会讲些一些源码，但不会太多，主要是原理，了解了 docker 原理，使用其它语言一样可以编写 docker

##### **鹏：
> docker在国内大厂被美国限制了吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 限制的是 docker-ee 版本，开源版本的 docker-ce 不会受影响，不影响我们的日常使用

##### *欣：
> 期待老师，总是看到什么容器，云原生，容器编排...，这部分真的对我是及时雨了

##### 周：
> win10系统怎么做到docker和安卓模拟器共存啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Windows 系统建议安装虚拟机，在虚拟机中使用 Docker

##### **9276：
> 签到

##### **雨：
> 第一次用docker是学习爬虫，平台制作成了镜像，拉过来，映射端口就能直接用了。最近一次安装ubuntu20.4，因为mysql装不上5.7，我就想到了docker，结果拉过来不会用😔，映射端口后也连不上，我就换回18.04了。后来反应过来，可能是我没安装客户端的原因，购买这门课的目的很简单，想少走点冤枉路!

##### *哼：
> 前排，搭建环境的运维来了，这个分销海报简直了哈哈哈哈

##### **锦：
> 学这个性价比很高呀

