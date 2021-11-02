<p data-nodeid="677" class="">在第 01 课时“Docker 安装：入门案例带你了解容器技术原理”中，我有介绍到 Docker 是基于 Linux 内核的 Namespace 技术实现资源隔离的，所有的容器都共享主机的内核。其实这与以虚拟机为代表的云计算时代还是有很多区别的，比如虚拟机有着更好的隔离性和安全性，而容器的隔离性和安全性则相对较弱。</p>
<p data-nodeid="678">在讨论容器的安全性之前，我们先了解下容器与虚拟机的区别，这样可以帮助我们更好地了解容器的安全隐患以及如何加固容器安全。</p>
<h3 data-nodeid="1345">Docker 与虚拟机区别</h3>
<p data-nodeid="1346" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/B7/Ciqc1F9sDDSAQhNcAAD8rL1NLXc02.jpeg" alt="WechatIMG1632.jpeg" data-nodeid="1350"></p>




<p data-nodeid="681">从图 1 可以看出，虚拟机是通过管理系统(Hypervisor)模拟出 CPU、内存、网络等硬件，然后在这些模拟的硬件上创建客户内核和操作系统。这样做的好处就是虚拟机有自己的内核和操作系统，并且硬件都是通过虚拟机管理系统模拟出来的，用户程序无法直接使用到主机的操作系统和硬件资源，因此虚拟机也对隔离性和安全性有着更好的保证。</p>
<p data-nodeid="682">而 Docker 容器则是通过 Linux 内核的 Namespace 技术实现了文件系统、进程、设备以及网络的隔离，然后再通过 Cgroups 对 CPU、 内存等资源进行限制，最终实现了容器之间相互不受影响，由于容器的隔离性仅仅依靠内核来提供，因此容器的隔离性也远弱于虚拟机。</p>
<p data-nodeid="683">你可能会问，既然虚拟机安全性这么好，为什么我们还要用容器呢？这是因为容器与虚拟机相比，容器的性能损耗非常小，并且镜像也非常小，而且在业务快速开发和迭代的今天，容器秒级的启动等特性也非常匹配业务快速迭代的业务场景。</p>
<p data-nodeid="684">既然我们要利用容器的优点，那有没有什么办法可以尽量弥补容器弱隔离的安全性缺点呢？要了解如何解决容器的安全问题，我们首先需要了解下容器目前存在的安全问题。</p>
<h3 data-nodeid="685">Docker 容器的安全问题</h3>
<h4 data-nodeid="686">(1) Docker 自身安全</h4>
<p data-nodeid="687">Docker 作为一款容器引擎，本身也会存在一些安全漏洞，CVE 目前已经记录了多项与 Docker 相关的安全漏洞，主要有权限提升、信息泄露等几类安全问题。具体 Docker 官方记录的安全问题可以参考<a href="https://docs.docker.com/engine/security/non-events/" data-nodeid="768">这里</a>。</p>
<blockquote data-nodeid="688">
<p data-nodeid="689">CVE 的维基百科定义：CVE 是公共漏洞和暴露（英语：CVE, Common Vulnerabilities and Exposures）又称常见漏洞与披露，是一个与信息安全有关的数据库，收集各种信息安全弱点及漏洞并给予编号以便于公众查阅。此数据库现由美国非营利组织 MITRE 所属的 National Cybersecurity FFRDC 所营运维护&nbsp;。</p>
</blockquote>
<h4 data-nodeid="690">(2) 镜像安全</h4>
<p data-nodeid="691">由于 Docker 容器是基于镜像创建并启动，因此镜像的安全直接影响到容器的安全。具体影响镜像安全的总结如下。</p>
<ul data-nodeid="692">
<li data-nodeid="693">
<p data-nodeid="694">镜像软件存在安全漏洞：由于容器需要安装基础的软件包，如果软件包存在漏洞，则可能会被不法分子利用并且侵入容器，影响其他容器或主机安全。</p>
</li>
<li data-nodeid="695">
<p data-nodeid="696">仓库漏洞：无论是 Docker 官方的镜像仓库还是我们私有的镜像仓库，都有可能被攻击，然后篡改镜像，当我们使用镜像时，就可能成为攻击者的目标对象。</p>
</li>
<li data-nodeid="697">
<p data-nodeid="698">用户程序漏洞：用户自己构建的软件包可能存在漏洞或者被植入恶意脚本，这样会导致运行时提权影响其他容器或主机安全。</p>
</li>
</ul>
<h4 data-nodeid="699">(3) Linux 内核隔离性不够</h4>
<p data-nodeid="700">尽管目前 Namespace 已经提供了非常多的资源隔离类型，但是仍有部分关键内容没有被完全隔离，其中包括一些系统的关键性目录（如 /sys、/proc 等），这些关键性的目录可能会泄露主机上一些关键性的信息，让攻击者利用这些信息对整个主机甚至云计算中心发起攻击。</p>
<p data-nodeid="701">而且仅仅依靠 Namespace 的隔离是远远不够的，因为一旦内核的 Namespace 被突破，使用者就有可能直接提权获取到主机的超级权限，从而影响主机安全。</p>
<h4 data-nodeid="702">(4) 所有容器共享主机内核</h4>
<p data-nodeid="703">由于同一宿主机上所有容器共享主机内核，所以攻击者可以利用一些特殊手段导致内核崩溃，进而导致主机宕机影响主机上其他服务。</p>
<p data-nodeid="704">既然容器有这么多安全上的问题，那么我们应该如何做才能够既享受到容器的便利性同时也可以保障容器安全呢？下面我带你来逐步了解下如何解决容器的安全问题。</p>
<h3 data-nodeid="705">如何解决容器的安全问题？</h3>
<h4 data-nodeid="706">(1) Docker 自身安全性改进</h4>
<p data-nodeid="707">事实上，Docker 从 2013 年诞生到现在，在安全性上面已经做了非常多的努力。目前 Docker 在默认配置和默认行为下是足够安全的。</p>
<p data-nodeid="708">Docker 自身是基于 Linux 的多种 Namespace 实现的，其中有一个很重要的 Namespace 叫作 User Namespace，User Namespace 主要是用来做容器内用户和主机的用户隔离的。在过去容器里的 root 用户就是主机上的 root 用户，如果容器受到攻击，或者容器本身含有恶意程序，在容器内就可以直接获取到主机 root 权限。Docker 从 1.10 版本开始，使用 User Namespace 做用户隔离，实现了容器中的 root 用户映射到主机上的非 root 用户，从而大大减轻了容器被突破的风险。</p>
<p data-nodeid="709">因此，我们尽可能地使用 Docker 最新版本就可以得到更好的安全保障。</p>
<h4 data-nodeid="710">(2) 保障镜像安全</h4>
<p data-nodeid="711">为保障镜像安全，我们可以在私有镜像仓库安装镜像安全扫描组件，对上传的镜像进行检查，通过与 CVE 数据库对比，一旦发现有漏洞的镜像及时通知用户或阻止非安全镜像继续构建和分发。同时为了确保我们使用的镜像足够安全，在拉取镜像时，要确保只从受信任的镜像仓库拉取，并且与镜像仓库通信一定要使用 HTTPS 协议。</p>
<h4 data-nodeid="712">(3) 加强内核安全和管理</h4>
<p data-nodeid="713">由于仅仅依赖内核的隔离可能会引发安全问题，因此我们对于内核的安全应该更加重视。可以从以下几个方面进行加强。</p>
<p data-nodeid="714"><strong data-nodeid="794">宿主机及时升级内核漏洞</strong></p>
<p data-nodeid="715">宿主机内核应该尽量安装最新补丁，因为更新的内核补丁往往有着更好的安全性和稳定性。</p>
<p data-nodeid="716"><strong data-nodeid="799">使用 Capabilities 划分权限</strong></p>
<p data-nodeid="717">Capabilities 是 Linux 内核的概念，Linux 将系统权限分为了多个 Capabilities，它们都可以单独地开启或关闭，Capabilities 实现了系统更细粒度的访问控制。</p>
<p data-nodeid="718">容器和虚拟机在权限控制上还是有一些区别的，在虚拟机内我们可以赋予用户所有的权限，例如设置 cron 定时任务、操作内核模块、配置网络等权限。而容器则需要针对每一项 Capabilities 更细粒度的去控制权限，例如：</p>
<ul data-nodeid="719">
<li data-nodeid="720">
<p data-nodeid="721">cron 定时任务可以在容器内运行，设置定时任务的权限也仅限于容器内部；</p>
</li>
<li data-nodeid="722">
<p data-nodeid="723">由于容器是共享主机内核的，因此在容器内部一般不允许直接操作主机内核；</p>
</li>
<li data-nodeid="724">
<p data-nodeid="725">容器的网络管理在容器外部，这就意味着一般情况下，我们在容器内部是不需要执行<code data-backticks="1" data-nodeid="805">ifconfig</code>、<code data-backticks="1" data-nodeid="807">route</code>等命令的 。</p>
</li>
</ul>
<p data-nodeid="726">由于容器可以按照需求逐项添加 Capabilities 权限，因此在大多数情况下，容器并不需要主机的 root 权限，Docker 默认情况下也是不开启额外特权的。</p>
<p data-nodeid="727">最后，在执行<code data-backticks="1" data-nodeid="811">docker run</code>命令启动容器时，如非特殊可控情况，--privileged 参数不允许设置为 true，其他特殊权限可以使用 --cap-add 参数，根据使用场景适当添加相应的权限。</p>
<p data-nodeid="728"><strong data-nodeid="816">使用安全加固组件</strong></p>
<p data-nodeid="729">Linux 的 SELinux、AppArmor、GRSecurity 组件都是 Docker 官方推荐的安全加固组件。下面我对这三个组件做简单介绍。</p>
<ul data-nodeid="730">
<li data-nodeid="731">
<p data-nodeid="732">SELinux (Secure Enhanced Linux): 是 Linux 的一个内核安全模块，提供了安全访问的策略机制，通过设置 SELinux 策略可以实现某些进程允许访问某些文件。</p>
</li>
<li data-nodeid="733">
<p data-nodeid="734">AppArmor: 类似于 SELinux，也是一个 Linux 的内核安全模块，普通的访问控制仅能控制到用户的访问权限，而 AppArmor 可以控制到用户程序的访问权限。</p>
</li>
<li data-nodeid="735">
<p data-nodeid="736">GRSecurity: 是一个对内核的安全扩展，可通过智能访问控制，提供内存破坏防御，文件系统增强等多种防御形式。</p>
</li>
</ul>
<p data-nodeid="737">这三个组件可以限制一个容器对主机的内核或其他资源的访问控制。目前，容器报告的一些安全漏洞中，很多都是通过对内核进行加强访问和隔离来实现的。</p>
<p data-nodeid="738"><strong data-nodeid="825">资源限制</strong></p>
<p data-nodeid="739">在生产环境中，建议每个容器都添加相应的资源限制。下面给出一些执行<code data-backticks="1" data-nodeid="827">docker run</code>命令启动容器时可以传递的资源限制参数：</p>
<pre class="lang-yaml" data-nodeid="740"><code data-language="yaml">  <span class="hljs-string">--cpus</span>                          <span class="hljs-string">限制</span> <span class="hljs-string">CPU</span> <span class="hljs-string">配额</span>
  <span class="hljs-string">-m,</span> <span class="hljs-string">--memory</span>                    <span class="hljs-string">限制内存配额</span>
  <span class="hljs-string">--pids-limit</span>                    <span class="hljs-string">限制容器的</span> <span class="hljs-string">PID</span> <span class="hljs-string">个数</span>
</code></pre>
<p data-nodeid="741">例如我想要启动一个 1 核 2G 的容器，并且限制在容器内最多只能创建 1000 个 PID，启动命令如下：</p>
<pre class="lang-shell" data-nodeid="742"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -it --cpus=1 -m=2048m --pids-limit=1000 busybox sh</span>
</code></pre>
<p data-nodeid="743">推荐在生产环境中限制 CPU、内存、PID 等资源，这样即便应用程序有漏洞，也不会导致主机的资源完全耗尽，最大限度降低安全风险。</p>
<h4 data-nodeid="744">(4) 使用安全容器</h4>
<p data-nodeid="745">容器有着轻便快速启动的优点，虚拟机有着安全隔离的优点，有没有一种技术可以兼顾两者的优点，做到既轻量又安全呢？</p>
<p data-nodeid="746">答案是有，那就是安全容器。安全容器是相较于普通容器的，安全容器与普通容器的主要区别在于，安全容器中的每个容器都运行在一个单独的微型虚拟机中，拥有独立的操作系统和内核，并且有虚拟化层的安全隔离。</p>
<p data-nodeid="747">安全容器目前推荐的技术方案是 <a href="https://github.com/kata-containers" data-nodeid="837">Kata Containers</a>，Kata Container 并不包含一个完整的操作系统，只有一个精简版的 Guest Kernel 运行着容器本身的应用，并且通过减少不必要的内存，尽量共享可以共享的内存来进一步减少内存的开销。另外，Kata Container 实现了 OCI 规范，可以直接使用 Docker 的镜像启动 Kata 容器，具有开销更小、秒级启动、安全隔离等许多优点。</p>
<h3 data-nodeid="748">结语</h3>
<p data-nodeid="749">容器技术带来的技术革新是空前的，但是随之而来的容器安全问题也是我们必须要足够重视的。本课时解决 Docker 安全问题的精华我帮你总结如下：</p>
<p data-nodeid="750"><img src="https://s0.lgstatic.com/i/image/M00/51/28/Ciqc1F9keVSAHuDTAADaB11MKbU710.png" alt="Lark20200918-170906.png" data-nodeid="843"></p>
<p data-nodeid="751">到此，相信你已经了解了 Docker 与虚拟机的本质区别，也知道了容器目前存在的一些安全隐患以及如何在生产环境中尽量避免这些安全隐患。</p>
<p data-nodeid="752" class="">目前除了 Kata Container 外，你还知道其他的安全容器解决方案吗？知道的同学，可以把你的想法写在留言区。</p>

---

### 精选评论

##### **军：
> 谷歌开源的gVisor是另一种安全容器技术，通过实现Linux的API，拦截容器对底层Linux接口的调用，实现容器安全的目标

##### Marton：
> 访问控制这块建议再深入介绍，可否出几个例子

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是想了解更多 Capabilities 的权限划分么？如果想了解这块的话，后面可以补充介绍一下。

##### **卫：
> 老师，想问一下我按照你上面的docker run启动一个1核2G的容器，进入之后查看内存和cpu核数，怎么看到的还是主机的数据，不是限制后容器实际能使用的数据呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; top 命令或者 free 等命令读取的是 /proc 目录下的内存和 CPU 数据，而 docker 容器内 /proc 目录实际上挂载的是主机的 /proc 目录，因此读取的 CPU 内存数据就是主机的，但实际 CPU 内存是 cgrous 限制的，因此只要启动 docker 时传递了限制 CPU或者内存参数，就可以生效。

##### *伟：
> 我看k8s社区主推podman，老师能大致说下这俩以后的发展吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; podman 只是对容器形态的一种补充，目前生产环境还是 Docker 使用最多。我们只要掌握了容器的原理，就能得心应手的使用任何形式的容器

##### *坤：
> 老师，我想问一下：1.镜像占用的是磁盘资源，一旦创建了容器并且启动容器才会占用服务器CPU、内存、网络等资源，可以这么理解吗？2.如果我启动容器的时候不配置cpu、内存等参数，默认使用的资源就是宿主机器的资源吗？如果宿主机器有多个启用容器就会资源竞争？3.前面讲到的感觉使用docker部署nginx这种有端口暴露的服务挺方便的，那jdk这种不需要启动就可以用的为啥也需要docker管理？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 是的 
2. 默认是不限制CPU 内存的, 相当于主机上直接启动进程,资源由内核调度和保障
3. JDK 可以做为镜像构建环境，真正运行的时候只需要 JRE 就可以了

##### **升：
> 打卡

##### **用户1366：
> 赞

##### **勇：
> 老师，对于linux内核方面的学习了解，有什么好的建议或者书籍推荐一下。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 《Linux内核设计与实现》这本书不错，另外可以结合源码学习内核相关的实现

##### Tangroo：
> docker版本：19.03.13老师我看了下 容器的/sys 目录和宿主机/sys目录是隔离的/proc 目录确实是没有隔离的是做了版本的优化吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 老师确认了下，19.03.13 的 docker /sys 目录是没有隔离的~

