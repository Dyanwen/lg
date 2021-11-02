<p data-nodeid="246879">上一课时我带你学习了什么是联合文件系统，以及 AUFS 的工作原理和配置。我们知道 AUFS  并不在 Linux 内核主干中，所以如果你的操作系统是 CentOS，就不推荐使用 AUFS 作为 Docker 的联合文件系统了。</p>



<p data-nodeid="246306">那在 CentOS 系统中，我们怎么实现镜像和容器的分层结构呢？我们通常使用 Devicemapper 作为 Docker 的联合文件系统。</p>
<h3 data-nodeid="246307">什么是 Devicemapper ？</h3>
<p data-nodeid="246308">Devicemapper 是 Linux 内核提供的框架，从 Linux 内核 2.6.9 版本开始引入，Devicemapper 与 AUFS 不同，AUFS 是一种文件系统，而<strong data-nodeid="246405">Devicemapper 是一种映射块设备的技术框架。</strong></p>
<p data-nodeid="246309">Devicemapper 提供了一种将物理块设备映射到虚拟块设备的机制，目前 Linux 下比较流行的 LVM （Logical Volume Manager 是 Linux 下对磁盘分区进行管理的一种机制）和软件磁盘阵列（将多个较小的磁盘整合成为一个较大的磁盘设备用于扩大磁盘存储和提供数据可用性）都是基于 Devicemapper 机制实现的。</p>
<p data-nodeid="246310">那么 Devicemapper 究竟是如何实现的呢？下面我们首先来了解一下它的关键技术。</p>
<h3 data-nodeid="246311">Devicemapper 的关键技术</h3>
<p data-nodeid="246312">Devicemapper 将主要的工作部分分为用户空间和内核空间。</p>
<ul data-nodeid="246313">
<li data-nodeid="246314">
<p data-nodeid="246315">用户空间负责配置具体的设备映射策略与相关的内核空间控制逻辑，例如逻辑设备 dm-a 如何与物理设备 sda 相关联，怎么建立逻辑设备和物理设备的映射关系等。</p>
</li>
<li data-nodeid="246316">
<p data-nodeid="246317">内核空间则负责用户空间配置的关联关系实现，例如当 IO 请求到达虚拟设备 dm-a 时，内核空间负责接管 IO 请求，然后处理和过滤这些 IO 请求并转发到具体的物理设备 sda 上。</p>
</li>
</ul>
<p data-nodeid="246318">这个架构类似于 C/S （客户端/服务区）架构的工作模式，客户端负责具体的规则定义和配置下发，服务端根据客户端配置的规则来执行具体的处理任务。</p>
<p data-nodeid="246319">Devicemapper 的工作机制主要围绕三个核心概念。</p>
<ul data-nodeid="247627">
<li data-nodeid="247628">
<p data-nodeid="247629">映射设备（mapped device）：即对外提供的逻辑设备，它是由 Devicemapper 模拟的一个虚拟设备，并不是真正存在于宿主机上的物理设备。</p>
</li>
<li data-nodeid="247630">
<p data-nodeid="247631">目标设备（target device）：目标设备是映射设备对应的物理设备或者物理设备的某一个逻辑分段，是真正存在于物理机上的设备。</p>
</li>
<li data-nodeid="247632">
<p data-nodeid="247633">映射表（map table）：映射表记录了映射设备到目标设备的映射关系，它记录了映射设备在目标设备的起始地址、范围和目标设备的类型等变量。</p>
</li>
</ul>
<p data-nodeid="248577"><img src="https://s0.lgstatic.com/i/image/M00/5E/87/CgqCHl-GyFOAG6TPAACE_8cMjoQ585.png" alt="Drawing 1.png" data-nodeid="248581"></p>
<div data-nodeid="248578" class=""><p style="text-align:center">图 1 Devicemapper 核心概念关系图</p></div>







<p data-nodeid="246330">Devicemapper 三个核心概念之间的关系如图 1，<strong data-nodeid="246428">映射设备通过映射表关联到具体的物理目标设备。事实上，映射设备不仅可以通过映射表关联到物理目标设备，也可以关联到虚拟目标设备，然后虚拟目标设备再通过映射表关联到物理目标设备。</strong></p>
<p data-nodeid="246331">Devicemapper 在内核中通过很多模块化的映射驱动（target driver）插件实现了对真正 IO 请求的拦截、过滤和转发工作，比如 Raid、软件加密、瘦供给（Thin Provisioning）等。其中瘦供给模块是 Docker 使用 Devicemapper 技术框架中非常重要的模块，下面我们来详细了解下瘦供给（Thin Provisioning）。</p>
<h4 data-nodeid="248948" class="">瘦供给（Thin Provisioning）</h4>

<p data-nodeid="246333">瘦供给的意思是动态分配，这跟传统的固定分配不一样。传统的固定分配是无论我们用多少都一次性分配一个较大的空间，这样可能导致空间浪费。而瘦供给是我们需要多少磁盘空间，存储驱动就帮我们分配多少磁盘空间。</p>
<p data-nodeid="246334">这种分配机制就好比我们一群人围着一个大锅吃饭，负责分配食物的人每次都给你一点分量，当你感觉食物不够时再去申请食物，而当你吃饱了就不需要再去申请食物了，从而避免了食物的浪费，节约的食物可以分配给更多需要的人。</p>
<p data-nodeid="246335">那么，你知道 Docker 是如何使用瘦供给来做到像 AUFS 那样分层存储文件的吗？答案就是： Docker 使用了瘦供给的快照（snapshot）技术。</p>
<p data-nodeid="246336">什么是快照（snapshot）技术？这是全球网络存储工业协会 SNIA（StorageNetworking Industry Association）对快照（Snapshot）的定义：</p>
<blockquote data-nodeid="246337">
<p data-nodeid="246338">关于指定数据集合的一个完全可用拷贝，该拷贝包括相应数据在某个时间点（拷贝开始的时间点）的映像。快照可以是其所表示的数据的一个副本，也可以是数据的一个复制品。</p>
</blockquote>
<p data-nodeid="246339">简单来说，<strong data-nodeid="246440">快照是数据在某一个时间点的存储状态。快照的主要作用是对数据进行备份，当存储设备发生故障时，可以使用已经备份的快照将数据恢复到某一个时间点，而 Docker 中的数据分层存储也是基于快照实现的。</strong></p>
<p data-nodeid="246340">以上便是实现 Devicemapper&nbsp;的关键技术，那 Docker 究竟是如何使用 Devicemapper 实现存储数据和镜像分层共享的呢？</p>
<h3 data-nodeid="246341">Devicemapper 是如何数据存储的？</h3>
<p data-nodeid="246342">当 Docker 使用 Devicemapper 作为文件存储驱动时，<strong data-nodeid="246447">Docker 将镜像和容器的文件存储在瘦供给池（thinpool）中，并将这些内容挂载在 /var/lib/docker/devicemapper/ 目录下。</strong></p>
<p data-nodeid="246343">这些目录储存 Docker 的容器和镜像相关数据，目录的数据内容和功能说明如下。</p>
<ul data-nodeid="246344">
<li data-nodeid="246345">
<p data-nodeid="246346">devicemapper 目录（/var/lib/docker/devicemapper/devicemapper/）：存储镜像和容器实际内容，该目录由一个或多个块设备构成。</p>
</li>
<li data-nodeid="246347">
<p data-nodeid="246348">metadata 目录（/var/lib/docker/devicemapper/metadata/）： 包含 Devicemapper  本身配置的元数据信息, 以 json 的形式配置，这些元数据记录了镜像层和容器层之间的关联信息。</p>
</li>
<li data-nodeid="246349">
<p data-nodeid="246350">mnt 目录（ /var/lib/docker/devicemapper/mnt/）：是容器的联合挂载点目录，未生成容器时，该目录为空，而容器存在时，该目录下的内容跟容器中一致。</p>
</li>
</ul>
<h3 data-nodeid="246351">Devicemapper 如何实现镜像分层与共享？</h3>
<p data-nodeid="246352">Devicemapper 使用专用的块设备实现镜像的存储，并且像 AUFS 一样使用了写时复制的技术来保障最大程度节省存储空间，所以 Devicemapper 的镜像分层也是依赖快照来是实现的。</p>
<p data-nodeid="246353">Devicemapper 的每一镜像层都是其下一层的快照，最底层的镜像层是我们的瘦供给池，通过这种方式实现镜像分层有以下优点。</p>
<ul data-nodeid="246354">
<li data-nodeid="246355">
<p data-nodeid="246356">相同的镜像层，仅在磁盘上存储一次。例如，我有 10 个运行中的 busybox 容器，底层都使用了 busybox 镜像，那么 busybox 镜像只需要在磁盘上存储一次即可。</p>
</li>
<li data-nodeid="246357">
<p data-nodeid="246358">快照是写时复制策略的实现，也就是说，当我们需要对文件进行修改时，文件才会被复制到读写层。</p>
</li>
<li data-nodeid="246359">
<p data-nodeid="246360">相比对文件系统加锁的机制，Devicemapper 工作在块级别，因此可以实现同时修改和读写层中的多个块设备，比文件系统效率更高。</p>
</li>
</ul>
<p data-nodeid="246361">当我们需要读取数据时，如果数据存在底层快照中，则向底层快照查询数据并读取。当我们需要写数据时，则向瘦供给池动态申请存储空间生成读写层，然后把数据复制到读写层进行修改。Devicemapper 默认每次申请的大小是 64K 或者 64K 的倍数，因此每次新生成的读写层的大小都是 64K 或者 64K 的倍数。</p>
<p data-nodeid="250386">以下是一个运行中的 Ubuntu 容器示意图。</p>
<p data-nodeid="250387" class=""><img src="https://s0.lgstatic.com/i/image/M00/5E/87/CgqCHl-GyHeAX_zKAABoNW4U26c205.png" alt="Drawing 3.png" data-nodeid="250392"></p>
<div data-nodeid="250388"><p style="text-align:center">图 2  Devicemapper 存储模型</p></div>






<p data-nodeid="246366">这个 Ubuntu 镜像一共有四层，每一层镜像都是下一层的快照，镜像的最底层是基础设备的快照。当容器运行时，容器是基于镜像的快照。综上，Devicemapper 实现镜像分层的根本原理就是快照。</p>
<p data-nodeid="246367">接下来，我们看下如何配置 Docker 的 Devicemapper 模式。</p>
<h3 data-nodeid="246368">如何在 Docker 中配置 Devicemapper</h3>
<p data-nodeid="246369">Docker 的 Devicemapper 模式有两种：第一种是 loop-lvm 模式，该模式主要用来开发和测试使用；第二种是 direct-lvm 模式，该模式推荐在生产环境中使用。</p>
<p data-nodeid="246370">下面我们逐一配置，首先来看下如何配置  loop-lvm 模式。</p>
<h4 data-nodeid="246371">配置 loop-lvm 模式</h4>
<p data-nodeid="246372">1.使用以下命令停止已经运行的 Docker：</p>
<pre class="lang-shell" data-nodeid="261261"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo systemctl stop docker</span>
</code></pre>
<p data-nodeid="261262">2.编辑 /etc/docker/daemon.json 文件，如果该文件不存在，则创建该文件，并添加以下配置：</p>
<pre class="lang-java" data-nodeid="261263"><code data-language="java">{
  <span class="hljs-string">"storage-driver"</span>: <span class="hljs-string">"devicemapper"</span>
}
</code></pre>
<p data-nodeid="261264">3.启动 Docker：</p>
<pre class="lang-shell" data-nodeid="261265"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo systemctl start docker</span>
</code></pre>
<p data-nodeid="261266">4.验证 Docker 的文件驱动模式：</p>
<pre class="lang-java" data-nodeid="264007"><code data-language="java">$ docker info
Client:
 Debug Mode: <span class="hljs-keyword">false</span>
Server:
 Containers: <span class="hljs-number">1</span>
  Running: <span class="hljs-number">0</span>
  Paused: <span class="hljs-number">0</span>
  Stopped: <span class="hljs-number">1</span>
 Images: <span class="hljs-number">1</span>
 Server Version: <span class="hljs-number">19.03</span>.<span class="hljs-number">12</span>
 Storage Driver: devicemapper
  Pool Name: docker-<span class="hljs-number">253</span>:<span class="hljs-number">1</span>-<span class="hljs-number">423624832</span>-pool
  Pool Blocksize: <span class="hljs-number">65.54</span>kB
  Base Device Size: <span class="hljs-number">10.74</span>GB
  Backing Filesystem: xfs
  Udev Sync Supported: <span class="hljs-keyword">true</span>
  Data file: /dev/loop0
  Metadata file: /dev/loop1
  Data loop file: /<span class="hljs-keyword">var</span>/lib/docker/devicemapper/devicemapper/data
  Metadata loop file: /<span class="hljs-keyword">var</span>/lib/docker/devicemapper/devicemapper/metadata
  Data Space Used: <span class="hljs-number">22.61</span>MB
  Data Space Total: <span class="hljs-number">107.4</span>GB
  Data Space Available: <span class="hljs-number">107.4</span>GB
  Metadata Space Used: <span class="hljs-number">17.37</span>MB
  Metadata Space Total: <span class="hljs-number">2.147</span>GB
  Metadata Space Available: <span class="hljs-number">2.13</span>GB
  Thin Pool Minimum Free Space: <span class="hljs-number">10.74</span>GB
  Deferred Removal Enabled: <span class="hljs-keyword">true</span>
  Deferred Deletion Enabled: <span class="hljs-keyword">true</span>
  Deferred Deleted Device Count: <span class="hljs-number">0</span>
  Library Version: <span class="hljs-number">1.02</span>.<span class="hljs-number">164</span>-RHEL7 (<span class="hljs-number">2019</span>-<span class="hljs-number">08</span>-<span class="hljs-number">27</span>)
... 省略部分输出
</code></pre>
<p data-nodeid="264398">可以看到 Storage Driver 为 devicemapper，这表示 Docker 已经被配置为 Devicemapper 模式。</p>
<p data-nodeid="264399">但是这里输出的 Data file 为 /dev/loop0，这表示我们目前在使用的模式为 loop-lvm。但是由于  loop-lvm 性能比较差，因此不推荐在生产环境中使用 loop-lvm 模式。下面我们看下生产环境中应该如何配置 Devicemapper 的 direct-lvm 模式。</p>

<h4 data-nodeid="264009">配置 direct-lvm 模式</h4>
<p data-nodeid="264010">1.使用以下命令停止已经运行的 Docker：</p>
<pre class="lang-shell" data-nodeid="266286"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo systemctl stop docker</span>
</code></pre>
<p data-nodeid="266287">2.编辑 /etc/docker/daemon.json 文件，如果该文件不存在，则创建该文件，并添加以下配置：</p>
<pre class="lang-java" data-nodeid="266668"><code data-language="java">{
  <span class="hljs-string">"storage-driver"</span>: <span class="hljs-string">"devicemapper"</span>,
  <span class="hljs-string">"storage-opts"</span>: [
    <span class="hljs-string">"dm.directlvm_device=/dev/xdf"</span>,
    <span class="hljs-string">"dm.thinp_percent=95"</span>,
    <span class="hljs-string">"dm.thinp_metapercent=1"</span>,
    <span class="hljs-string">"dm.thinp_autoextend_threshold=80"</span>,
    <span class="hljs-string">"dm.thinp_autoextend_percent=20"</span>,
    <span class="hljs-string">"dm.directlvm_device_force=false"</span>
  ]
}
</code></pre>
<p data-nodeid="266669">其中 directlvm_device 指定需要用作 Docker 存储的磁盘路径，Docker 会动态为我们创建对应的存储池。例如这里我想把 /dev/xdf 设备作为我的 Docker 存储盘，directlvm_device 则配置为  /dev/xdf。</p>
<p data-nodeid="266670">3.启动 Docker：</p>
<pre class="lang-shell" data-nodeid="268527"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo systemctl start docker</span>
</code></pre>
<p data-nodeid="268528">4.验证 Docker 的文件驱动模式：</p>
<pre class="lang-java" data-nodeid="270365"><code data-language="java">$ docker info
Client:
 Debug Mode: <span class="hljs-keyword">false</span>
Server:
 Containers: <span class="hljs-number">1</span>
  Running: <span class="hljs-number">0</span>
  Paused: <span class="hljs-number">0</span>
  Stopped: <span class="hljs-number">1</span>
 Images: <span class="hljs-number">1</span>
 Server Version: <span class="hljs-number">19.03</span>.<span class="hljs-number">12</span>
 Storage Driver: devicemapper
  Pool Name: docker-thinpool
  Pool Blocksize: <span class="hljs-number">65.54</span>kB
  Base Device Size: <span class="hljs-number">10.74</span>GB
  Backing Filesystem: xfs
  Udev Sync Supported: <span class="hljs-keyword">true</span>
  Data file:
  Metadata file:
  Data loop file: /<span class="hljs-keyword">var</span>/lib/docker/devicemapper/devicemapper/data
  Metadata loop file: /<span class="hljs-keyword">var</span>/lib/docker/devicemapper/devicemapper/metadata
  Data Space Used: <span class="hljs-number">22.61</span>MB
  Data Space Total: <span class="hljs-number">107.4</span>GB
  Data Space Available: <span class="hljs-number">107.4</span>GB
  Metadata Space Used: <span class="hljs-number">17.37</span>MB
  Metadata Space Total: <span class="hljs-number">2.147</span>GB
  Metadata Space Available: <span class="hljs-number">2.13</span>GB
  Thin Pool Minimum Free Space: <span class="hljs-number">10.74</span>GB
  Deferred Removal Enabled: <span class="hljs-keyword">true</span>
  Deferred Deletion Enabled: <span class="hljs-keyword">true</span>
  Deferred Deleted Device Count: <span class="hljs-number">0</span>
  Library Version: <span class="hljs-number">1.02</span>.<span class="hljs-number">164</span>-RHEL7 (<span class="hljs-number">2019</span>-<span class="hljs-number">08</span>-<span class="hljs-number">27</span>)
... 省略部分输出
</code></pre>
<p data-nodeid="270366">当我们看到 Storage Driver 为 devicemapper，并且 Pool Name 为 docker-thinpool 时，这表示 Devicemapper 的 direct-lvm 模式已经配置成功。</p>
<h3 data-nodeid="270367">结语</h3>
<p data-nodeid="270368">Devicemapper 使用块设备来存储文件，运行速度会比直接操作文件系统更快，因此很长一段时间内在 Red Hat 或 CentOS 系统中，Devicemapper 一直作为 Docker 默认的联合文件系统驱动，为 Docker 在 Red Hat 或 CentOS 稳定运行提供强有力的保障。</p>
<p data-nodeid="270369">那么你知道使用 Devicemapper 作为 Docker 联合文件系统的一种解方案是哪家公司在推动吗？ 思考后，可以把你的想法写在留言区。</p>
<p data-nodeid="270370">下一课时，我将讲解 Docker 的另一个文件存储驱动：OverlayFS 文件系统原理及生产环境的最佳配置。</p>

---

### 精选评论

##### **平：
> 早期的Docker运行在Ubuntu和Debian Linux上并使用AUFS作为后端存储。Docker流行之后，越来越多的的公司希望在Red Hat Enterprise Linux这类企业级的操作系统上面运行Docker，但可惜的是RHEL的内核并不支持AUFS。这个时候红帽公司出手了，决定和Docker公司合作去开发一种基于Device Mapper技术的后端存储，也就是现在的devicemapper

##### *冲：
> 配置 direct-lvm 模式不能成功。。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以查看具体报错信息，然后根据报错解决问题。

##### *冲：
> /dev/xdf 设备是什么设备, 换成一个什么别的目录行吗. 修改完daemon.json后服务一起不能启动,我怀疑是这个配置的问题

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; /dev/xdf  是老师测试机器的设备，你可以根据自己的设备来具体配置。

##### *冲：
> systemctl start dockerJob for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.我把daemon.json配置文件清了, systemctl daemon_reload了还是 不能开启docker服务,经常碰到这个问题,是docker不稳定吗?还是什么神秘力量!https://github.com/docker/for-linux/issues/162 这个issue也没好的解决方法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以根据提示使用  "systemctl status docker.service" 和 "journalctl -xe" 命令查看具体报错

##### *冲：
> 快照可以是其所表示的数据的一个副本，也可以是数据的一个复制品。这个怎么理解？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以理解为在某个时刻将磁盘的数据备份到某个地方。

##### **生：
> Devicemapper应该主要是RH公司推动的，当时RH内核不支持AUFS，早期，RH公司想将Docker的AUFS合并内核，其后不知具体原因，他们使用自己原有的后端存储技术，其后与Docker公司合作，贡献了这个第二个驱动。另：devicemapper出现的，当时的背景需求是：一个系统上只能连接有限的常用存储设备（e.g 只有一个硬盘），其后此出现了devicemapper技术，以此来解决当时的问题

##### *奇：
> 老师好，我用运行容器的时候用-p 将用到的端口映射出来，同一个镜像同样的命令在centos，ubuntu下都是没问题的，在Mac 下访问不到映射出来端口的服务，这个Mac有什么特殊的设置吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Mac 下的 Docker 是通过虚拟机实现的，所以 Docker 实际上是运行在虚拟机中的，但是默认情况下通过 -p 映射出来后可以通过 localhost 加端口的方式访问到。

##### **艺：
> 这个 Ubuntu 镜像一共有四层，每一层镜像都是下一层的快照？这句不是很理解...

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 快照是写时复制策略的实现，也就是说，当我们需要对文件进行修改时，文件才会被复制到读写层。

##### *帅：
> 赞

