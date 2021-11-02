<p data-nodeid="10136" class="">Docker 的操作围绕镜像、容器、仓库三大核心概念。在学架构设计之前，我们需要先了解 Docker 的三个核心概念。</p>
<h3 data-nodeid="10137">Docker 核心概念</h3>
<h4 data-nodeid="10138">镜像</h4>
<p data-nodeid="10139">镜像是什么呢？通俗地讲，它是一个只读的文件和文件夹组合。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。所以你想启动一个容器，那首先必须要有一个镜像。<strong data-nodeid="10206">镜像是 Docker 容器启动的先决条件。</strong></p>
<p data-nodeid="10140">如果你想要使用一个镜像，你可以用这两种方式：</p>
<ol data-nodeid="10141">
<li data-nodeid="10142">
<p data-nodeid="10143">自己创建镜像。通常情况下，一个镜像是基于一个基础镜像构建的，你可以在基础镜像上添加一些用户自定义的内容。例如你可以基于<code data-backticks="1" data-nodeid="10209">centos</code>镜像制作你自己的业务镜像，首先安装<code data-backticks="1" data-nodeid="10211">nginx</code>服务，然后部署你的应用程序，最后做一些自定义配置，这样一个业务镜像就做好了。</p>
</li>
<li data-nodeid="10144">
<p data-nodeid="10145">从功能镜像仓库拉取别人制作好的镜像。一些常用的软件或者系统都会有官方已经制作好的镜像，例如<code data-backticks="1" data-nodeid="10214">nginx</code>、<code data-backticks="1" data-nodeid="10216">ubuntu</code>、<code data-backticks="1" data-nodeid="10218">centos</code>、<code data-backticks="1" data-nodeid="10220">mysql</code>等，你可以到 <a href="https://hub.docker.com/" data-nodeid="10224">Docker Hub</a> 搜索并下载它们。</p>
</li>
</ol>
<h4 data-nodeid="10146">容器</h4>
<p data-nodeid="10147">容器是什么呢？容器是 Docker 的另一个核心概念。通俗地讲，容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即<strong data-nodeid="10231">容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。</strong></p>
<p data-nodeid="10148">虽然容器的本质是主机上运行的一个进程，但是容器有自己独立的命名空间隔离和资源限制。也就是说，在容器内部，无法看到主机上的进程、环境变量、网络等信息，这是容器与直接运行在主机上进程的本质区别。</p>
<h4 data-nodeid="10149">仓库</h4>
<p data-nodeid="10150">Docker 的镜像仓库类似于代码仓库，用来存储和分发 Docker 镜像。镜像仓库分为公共镜像仓库和私有镜像仓库。</p>
<p data-nodeid="10151">目前，<a href="https://hub.docker.com/" data-nodeid="10238">Docker Hub</a> 是 Docker 官方的公开镜像仓库，它不仅有很多应用或者操作系统的官方镜像，还有很多组织或者个人开发的镜像供我们免费存放、下载、研究和使用。除了公开镜像仓库，你也可以构建自己的私有镜像仓库，在第 5 课时，我会带你搭建一个私有的镜像仓库。</p>
<h4 data-nodeid="10152">镜像、容器、仓库，三者之间的联系</h4>
<p data-nodeid="10153"><img src="https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYryALHVmAABihjRzo4c527.png" alt="Drawing 1.png" data-nodeid="10243"></p>
<p data-nodeid="10154">从图 1 可以看到，镜像是容器的基石，容器是由镜像创建的。一个镜像可以创建多个容器，容器是镜像运行的实体。仓库就非常好理解了，就是用来存放和分发镜像的。</p>
<p data-nodeid="10155">了解了 Docker 的三大核心概念，接下来认识下 Docker 的核心架构和一些重要的组件。</p>
<h3 data-nodeid="10156">Docker 架构</h3>
<p data-nodeid="10157">在了解 Docker 架构前，我先说下相关的背景知识——容器的发展史。</p>
<p data-nodeid="10158">容器技术随着 Docker 的出现变得炙手可热，所有公司都在积极拥抱容器技术。此时市场上除了有 Docker 容器，还有很多其他的容器技术，比如 CoreOS 的 rkt、lxc 等。容器技术百花齐放是好事，但也出现了很多问题。比如容器技术的标准到底是什么？容器标准应该由谁来制定？</p>
<p data-nodeid="10159">也许你可能会说， Docker 已经成为了事实标准，把 Docker 作为容器技术的标准不就好了？事实并没有想象的那么简单。因为那时候不仅有容器标准之争，编排技术之争也十分激烈。当时的编排技术有三大主力，分别是 Docker Swarm、Kubernetes 和 Mesos 。Swarm 毋庸置疑，肯定愿意把 Docker 作为唯一的容器运行时，但是 Kubernetes 和 Mesos 就不同意了，因为它们不希望调度的形式过度单一。</p>
<p data-nodeid="10160">在这样的背景下，最终爆发了容器大战，<code data-backticks="1" data-nodeid="10251">OCI</code>也正是在这样的背景下应运而生。</p>
<p data-nodeid="10161"><code data-backticks="1" data-nodeid="10253">OCI</code>全称为开放容器标准（Open Container Initiative），它是一个轻量级、开放的治理结构。<code data-backticks="1" data-nodeid="10255">OCI</code>组织在 Linux 基金会的大力支持下，于 2015 年 6 月份正式注册成立。基金会旨在为用户围绕工业化容器的格式和镜像运行时，制定一个开放的容器标准。目前主要有两个标准文档：<strong data-nodeid="10266">容器运行时标准 （runtime spec）<strong data-nodeid="10265">和</strong>容器镜像标准（image spec）</strong>。</p>
<p data-nodeid="10162">正是由于容器的战争，才导致 Docker 不得不在战争中改变一些技术架构。最终形成了下图所示的技术架构。</p>
<p data-nodeid="10163"><img src="https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYtCAC1GSAADIK4E6wrc368.png" alt="Drawing 2.png" data-nodeid="10270"></p>
<div data-nodeid="10164"><p style="text-align:center">图2  Docker 架构图</p></div>
<p data-nodeid="10165">我们可以看到，Docker 整体架构采用 C/S（客户端 / 服务器）模式，主要由客户端和服务端两大部分组成。客户端负责发送操作指令，服务端负责接收和处理指令。客户端和服务端通信有多种方式，既可以在同一台机器上通过<code data-backticks="1" data-nodeid="10272">UNIX</code>套接字通信，也可以通过网络连接远程通信。</p>
<p data-nodeid="10166">下面我逐一介绍客户端和服务端。</p>
<h4 data-nodeid="10167">Docker 客户端</h4>
<p data-nodeid="10168">Docker 客户端其实是一种泛称。其中 docker 命令是 Docker 用户与 Docker 服务端交互的主要方式。除了使用 docker 命令的方式，还可以使用直接请求 REST API 的方式与 Docker 服务端交互，甚至还可以使用各种语言的 SDK 与 Docker 服务端交互。目前社区维护着 Go、Java、Python、PHP 等数十种语言的 SDK，足以满足你的日常需求。</p>
<h4 data-nodeid="10169">Docker 服务端</h4>
<p data-nodeid="10170">Docker 服务端是 Docker 所有后台服务的统称。其中 dockerd 是一个非常重要的后台管理进程，它负责响应和处理来自 Docker 客户端的请求，然后将客户端的请求转化为 Docker 的具体操作。例如镜像、容器、网络和挂载卷等具体对象的操作和管理。</p>
<p data-nodeid="10171">Docker 从诞生到现在，服务端经历了多次架构重构。起初，服务端的组件是全部集成在 docker 二进制里。但是从 1.11 版本开始， dockerd 已经成了独立的二进制，此时的容器也不是直接由 dockerd 来启动了，而是集成了 containerd、runC 等多个组件。</p>
<p data-nodeid="10172">虽然 Docker 的架构在不停重构，但是各个模块的基本功能和定位并没有变化。它和一般的 C/S 架构系统一样，Docker 服务端模块负责和 Docker 客户端交互，并管理 Docker 的容器、镜像、网络等资源。</p>
<h4 data-nodeid="10173">Docker 重要组件</h4>
<p data-nodeid="10174">下面，我以 Docker 的 18.09.2 版本为例，看下 Docker 都有哪些工具和组件。在 Docker 安装路径下执行 ls 命令可以看到以下与 docker 有关的二进制文件。</p>
<pre class="lang-java" data-nodeid="10175"><code data-language="java">-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">27941976</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> containerd
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">4964704</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> containerd-shim
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">15678392</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> ctr
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">50683148</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker
-rwxr-xr-x <span class="hljs-number">1</span> root root   <span class="hljs-number">764144</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker-init
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">2837280</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker-proxy
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">54320560</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> dockerd
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">7522464</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> runc
</code></pre>
<p data-nodeid="10176">可以看到，Docker 目前已经有了非常多的组件和工具。这里我不对它们逐一介绍，因为在第 11 课时，我会带你深入剖析每一个组件和工具。<br>
这里我先介绍一下 Docker 的两个至关重要的组件：<code data-backticks="1" data-nodeid="10286">runC</code>和<code data-backticks="1" data-nodeid="10288">containerd</code>。</p>
<ul data-nodeid="10177">
<li data-nodeid="10178">
<p data-nodeid="10179"><code data-backticks="1" data-nodeid="10290">runC</code>是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。</p>
</li>
<li data-nodeid="10180">
<p data-nodeid="10181"><code data-backticks="1" data-nodeid="10292">containerd</code>是 Docker 服务端的一个核心组件，它是从<code data-backticks="1" data-nodeid="10294">dockerd</code>中剥离出来的 ，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。<code data-backticks="1" data-nodeid="10296">containerd</code>通过 containerd-shim 启动并管理 runC，可以说<code data-backticks="1" data-nodeid="10298">containerd</code>真正管理了容器的生命周期。</p>
</li>
</ul>
<p data-nodeid="10182"><img src="https://s0.lgstatic.com/i/image/M00/49/93/Ciqc1F9PYuuAQINxAAA236heaL0459.png" alt="Drawing 3.png" data-nodeid="10302"></p>
<div data-nodeid="10183"><p style="text-align:center">图3 Docker 服务端组件调用关系图</p></div>
<p data-nodeid="10184">通过上图，可以看到，<code data-backticks="1" data-nodeid="10304">dockerd</code>通过 gRPC 与<code data-backticks="1" data-nodeid="10306">containerd</code>通信，由于<code data-backticks="1" data-nodeid="10308">dockerd</code>与真正的容器运行时，<code data-backticks="1" data-nodeid="10310">runC</code>中间有了<code data-backticks="1" data-nodeid="10312">containerd</code>这一 OCI 标准层，使得<code data-backticks="1" data-nodeid="10314">dockerd</code>可以确保接口向下兼容。</p>
<blockquote data-nodeid="18145">
<p data-nodeid="18146" class=""><a href="https://grpc.io" data-nodeid="18149">gRPC</a> 是一种远程服务调用。想了解更多信息可以参考<a href="https://grpc.io/" data-nodeid="18153">https://grpc.io</a><br>
containerd-shim&nbsp;的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim&nbsp;的主要作用是将&nbsp;containerd&nbsp;和真正的容器进程解耦，使用&nbsp;containerd-shim&nbsp;作为容器进程的父进程，从而实现重启&nbsp;dockerd&nbsp;不影响已经启动的容器进程。</p>
</blockquote>




















<p data-nodeid="10187">了解了 dockerd、containerd 和 runC 之间的关系，下面可以通过启动一个 Docker 容器，来验证它们进程之间的关系。</p>
<h4 data-nodeid="10188">Docker 各组件之间的关系</h4>
<p data-nodeid="10189">首先通过以下命令来启动一个 busybox 容器：</p>
<pre class="lang-shell" data-nodeid="10190"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -d busybox sleep 3600</span>
</code></pre>
<p data-nodeid="10191">容器启动后，通过以下命令查看一下 dockerd 的 PID：</p>
<pre class="lang-shell" data-nodeid="10192"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo ps aux |grep dockerd</span>
root&nbsp; &nbsp; &nbsp; 4147&nbsp; 0.3&nbsp; 0.2 1447892 83236 ?&nbsp; &nbsp; &nbsp; &nbsp;Ssl&nbsp; Jul09 245:59 /usr/bin/dockerd
</code></pre>
<p data-nodeid="10193">通过上面的输出结果可以得知 dockerd 的 PID 为 4147。为了验证图 3 中 Docker 各组件之间的调用关系，下面使用 pstree 命令查看一下进程父子关系：</p>
<pre class="lang-shell" data-nodeid="10194"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo pstree -l -a -A 4147</span>
dockerd
&nbsp; |-containerd --config /var/run/docker/containerd/containerd.toml --log-level info
&nbsp; |&nbsp; &nbsp;|-containerd-shim -namespace moby -workdir /var/lib/docker/containerd/daemon/io.containerd.runtime.v1.linux/moby/d14d20507073e5743e607efd616571c834f1a914f903db6279b8de4b5ba3a45a -address /var/run/docker/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
&nbsp; |&nbsp; &nbsp;|&nbsp; &nbsp;|-sleep 3600
</code></pre>
<p data-nodeid="10195">事实上，dockerd 启动的时候， containerd 就随之启动了，dockerd 与 containerd 一直存在。当执行 docker run 命令（通过 busybox 镜像创建并启动容器）时，containerd 会创建 containerd-shim 充当 “垫片”进程，然后启动容器的真正进程 sleep 3600 。这个过程和架构图是完全一致的。</p>
<h4 data-nodeid="10196">结语</h4>
<p data-nodeid="10197">本课时有基础、有架构，是一篇为后续打基础的文章。如果你有什么知识点没理解到位，有疑问，可写在留言处，我回复置顶，给他人参考。</p>
<p data-nodeid="10198" class="">如果你理解到位，相信你对 Docker 的三大核心概念镜像、容器、仓库有了一个清楚的认识，并对 Dokcer 的架构有了一定的了解。那么你知道为什么 Docker 公司要把<code data-backticks="1" data-nodeid="10334">containerd</code>拆分并捐献给社区吗？思考后，也可以把你的想法写在留言区。</p>

---

### 精选评论

##### **翰：
> [root@mgr_1 bin]# ps aux|grep dockerdroot     15436  0.0  3.7 518172 71188 ?        Ssl  14:58   0:00 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock[root@mgr_1 bin]# pstree -l -a -A 15436dockerd -H fd:// --containerd=/run/containerd/containerd.sock `-9*[{dockerd}]为啥我看到的是这样的 版本是19.03.12

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker 19.03.12 版本的 dockerd 和containerd 组件已经不是父子关系，可以使用以下命令查看，sudo ps aux |grep containerd , 然后使用 pstree 查看 containerd 的 PID

##### *城：
> 讲课思路清晰，有理有据，能让人听懂，有时候点太多，我觉得能说清一点就很好了，期待后续的课程！已经强过这里很多课程了！

##### **1180：
> containerd 捐赠拆分， 在一定程度上让开发者更容易的去接触到一些特性，使得‘标准’这个概念也更加深刻吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，containerd 捐赠主要是为了让大家更好的接受 docker 标准

##### **彪：
> 既然无法抵抗标准的诞生，不如由我来主导标准

##### *豆：
> Docker为了表示对于社区和生态的诚意，特意强调了containerd中立的地位，符合各方利益。可以预见containerd将成为Docker平台的一块重要组件。阿里云, AWS, Google，IBM和Microsoft将参与到containerd的开发中。

##### **峰：
> containerd 作为Docker在容器领域架构的一个重要组件，保持上层架构，为兼容其他容器提供可能！

##### *悦：
> containerd-shim是不是可以理解为containerd和runC之间的接口；containerd和runC都可以被具备相同功能的其他组件代替？（可以这么理解吗）

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; containerd-shim 是真正容器的进程的父进程，这么做为了不让真正的容器进程作为 containerd  的子进程，从而可以实现重启 containerd 而不影响已经运行的容器

##### **思：
> 讲的太好了。👍

##### **玮：
> 老师，没明白containerd、containerd-shim和runc具体是干什么的，相互之间什么关系，能再描述一下么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; containerd 是管理容器声明周期的，containerd-shim 是为了解耦 containerd 与业务进程，runc 是真正启动容器进程的，11 讲会具体讲解

##### **飞：
> pstree这儿貌似有问题，pstree -l -a -A 4147这句只能查出dockerd进程的信息，而查不到containerd进程的信息。我理解的为当启动docker的时候（service docker start）其实就是启动了dockerd和containerd两个同级别的进程。而当使用docker run命令时，containerd进程将创建子进程，这个子进程里面就会创建containerd-shim进程和通过runc去真正启动容器了。不晓得我说的对不对？看看哇

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker 19.03.12 版本的 dockerd 和containerd 组件已经不是父子关系，可以使用以下命令查看，sudo ps aux |grep cotainerd , 然后使用 pstree 查看 containerd 的 PID

##### *彬：
> containerd的拆分是基于开源，并且containerd已经是OCI的标准下的产物，实现了向下兼容的功能，可以基于其进行开发和优化。

##### *峥：
> 突然发现自己 之气学的是皮毛中的皮毛😂

##### lpzh：
> 为何我的执行结果自由一行(vagrant安装的centos7环境):[root@localhost vagrant]# pstree -l -a -A 2199dockerd -H fd:// --containerd=/run/containerd/containerd.sock `-9*[{dockerd}]

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker 19.03.12 版本的 dockerd 和containerd 组件已经不是父子关系，可以使用以下命令查看，sudo ps aux |grep cotainerd , 然后使用 pstree 查看 containerd 的 PID

##### *政：
> 容器的本质是主机上的一个进程，那在容器内有多进程吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 必然是有多进程的，通常容器主入口是一个主进程，shim 进程，用户的进程都是 shim 进程的子进程。

##### Ddd：
> 老师你上面说的：“containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 dockerd 不影响已经启动的容器进程。”是不是改成“containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd不影响已经启动的容器进程。”

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，如果只使用 containerd 不使用 dockerd 可以这么理解。

##### *冲：
> 我的docker20.10.0 版本，ls /var/lib/docker看到的组件和老师看到的差距好大！？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; /var/lib/docker 目录下并不是 docker 的组件，那是 docker 的工作目录，docker 的组件是一堆二进制文件，在 /usr/bin/ 目录下

##### *俊：
> 核心概念

##### **强：
> 讲的太好了 看文档都能思路清晰，有理有据，知其所以然，才是好课程。期待后面的课程

##### **狗：
> 老师：docker run hello-world 命令启动了这个容器之后用pstree -l -a -A 进程id 看之后，并没有controllerd-shim，如下： |-docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 80 -container-ip 172.17.0.3 -container-port 80`-6*[{docker-proxy}] |-docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 9090 -container-ip 172.17.0.2 -container-port 9090`-5*[{docker-proxy}] `-33*[{dockerd}]如果和视频一样，用命令docker run -d busybox sleep 3600之后，进程还是只有刚才那个helloworld的容器进程，这是为啥呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问使用的 docker 版本是 19 版本吗？如果是的话可以使用 pstree -l -a -A 后面跟 containerd 的 PID 进行查看。

##### **东：
> containerd 通过开启子进程 containerd-shim 来启动真正的业务进程 sleep 3600containerd-shim 的作用是将 containerd 与 真正的业务进程解耦, 从而实现重启 containerd 不影响已启动的容器进程我的问题是: containerd-shim 是 containerd 的子进程, 重启 containerd 不会连带重启 containerd-shim 吗? 进程的重启它的子进程会如何表现,这点不太了解,请老师解释下,谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 docker 的话可以实现重启 dockerd 不重启业务容器，具体的配置可以参考官网相关说明：https://docs.docker.com/config/containers/live-restore/

##### **毅：
> Mac 环境下。执行命令：sudo ps aux |grep containerd 每次都不一样，好像会递增的，这个 PID

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Mac 的 Docker 是运行在虚拟里的，所以主机上你是看不到的。你看到的 PID 是你执行 shell 命令的 PID

##### **汇：
> container-shim是容器的父进程，重启containerd不影响容器进程。我的问题是，container-shim的父进程是containerd，containerd重启不是会影响shim吗，从而级联影响

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 docker 的话可以实现重启 dockerd 不重启业务容器，具体的配置可以参考官网相关说明：https://docs.docker.com/config/containers/live-restore/

##### **恒：
> 真的是通俗易懂，老师强啊

##### **9313：
> 容器是怎样运行一个基础系统镜像的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 容器的本质是进程，根据指定的命令在镜像这个“文件系统中“ 启动进程。

##### **兵：
> 老师您好，按照01 Docker安装的方式，安装路径在哪个目录？我安装的版本是19.03.12。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Docker 的二进制默认安装在 /usr/bin/ 目录下

##### **他：
> Docker客户端和服务端通信，可以通过网络连接远程通信.那就是说，我在本地，可以通过Docker客户端可以操作远程主机上的Docker服务端？这种场景，一般出现在什么情况下？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这种场景一般用于客户端是自己电脑，但是需要远程连接开发机来开发业务。

##### **飞：
> 为什么pstree 查看dockerd进程，查询出来的层级结果为dockerd -- docker-proxy这种，而不是dockerd - containerd -- containerd-shim -runc这种？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker 19.03.12 版本的 dockerd 和containerd 组件已经不是父子关系，可以使用以下命令查看，sudo ps aux |grep cotainerd , 然后使用 pstree 查看 containerd 的 PID

##### **飞：
> 为啥子为执行pstree后，看不出来dockerd、containerd、containerd-shim的关系。如下截图所示：root@k8s-master:~# pstree -l -a -A 22213dockerd -H fd:// --containerd=/run/containerd/containerd.sock |-docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8051 -container-ip 172.17.0.2 -container-port 8051`-7*[{docker-proxy}] |-docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8050 -container-ip 172.17.0.2 -container-port 8050`-7*[{docker-proxy}] |-docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 5023 -container-ip 172.17.0.2 -container-port 5023`-7*[{docker-proxy}] `-18*[{dockerd}]

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker 19.03.12 版本的 dockerd 和containerd 组件已经不是父子关系，可以使用以下命令查看，sudo ps aux |grep cotainerd , 然后使用 pstree 查看 containerd 的 PID

##### **飞：
> containerd-shim不影响已经启动的容器进程。这一句话有点没太理解，为什么重启containerd不影响已经启动的容器进程？ 重启containerd是不是可以理解为重启整个docker呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用 containerd-shim 就是为了不要业务进程作为 containerd 的子进程，这样重启 containerd 就不会重启业务进程了

##### **豪：
> 想问下，，在docker容器中搭建大数据平台，如hadoop，hive等，然后，在生产环境中，直接拉取使用，，想问下，这种方式可不可以，有没有这么干的 ？， 这样会有什么影响吗，和直接在服务器中搭建hadoop环境的区别在哪里，以及后期维护，安全性有什么不同呢，，谢谢！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 基本是一样的，但是Docker 由于实现的特殊性很多和物理机或虚拟是不一样的，例如在 Docker 无法使用 systemctl 管理组件，无法定制内核等。如果你的业务不依赖这些，完全可以把环境直接放 Docker 里，如果环境太复杂，推荐使用虚拟机结合虚拟机镜像的方式。

##### **俊：
> 对于 Mac 和 Widnow 怎么只下载 docker client 端，不下载 server 端的？ 对于 window 低版本的系统，只想下载 client 然后连接其他地方的 Server

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Mac 版本可以到这里下载 https://download.docker.com/mac/static/stable/x86_64/ 二进制版本，Windows 官方暂未提供二进制，需要自行编译

##### **定：
> 我觉得containerd捐献给社区，是通过大家的潜能把docker推向更远，更强大

