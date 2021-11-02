<p data-nodeid="621" class="">咱们第一课时就先聊聊 Docker 的基础内容：Docker 能做什么，怎么安装 Docker，以及容器技术的原理。</p>
<h3 data-nodeid="622">Docker 能做什么？</h3>
<p data-nodeid="623">众所周知，Docker 是一个用于开发，发布和运行应用程序的开放平台。通俗地讲，Docker 类似于集装箱。在一艘大船上，各种货物要想被整齐摆放并且相互不受到影响，我们就需要把各种货物进行集装箱标准化。有了集装箱，我们就不需要专门运输水果或者化学用品的船了。我们可以把各种货品通过集装箱打包，然后统一放到一艘船上运输。Docker 要做的就是把各种软件打包成一个集装箱（镜像），然后分发，且在运行的时候可以相互隔离。</p>
<p data-nodeid="624">到此，相信你已经迫不及待想要体验下了，下面就让我们来安装一个 Docker。</p>
<h3 data-nodeid="625">CentOS 下安装 Docker</h3>
<p data-nodeid="626">Docker 是跨平台的解决方案，它支持在当前主流的各大平台安装，包括 Ubuntu、RHEL、CentOS、Debian 等 Linux 发行版，同时也可以在 OSX 、Microsoft Windows 等非 Linux 平台下安装使用。</p>
<p data-nodeid="627">因为 Linux 是 Docker 的原生支持平台，所以推荐你在 Linux 上使用 Docker。由于生产环境中我们使用 CentOS 较多，下面主要针对在 CentOS 平台下安装和使用 Docker 展开介绍。</p>
<h4 data-nodeid="628">操作系统要求</h4>
<p data-nodeid="629">要安装 Docker，我们需要 CentOS 7 及以上的发行版本。建议使用<code data-backticks="1" data-nodeid="712">overlay2</code>存储驱动程序。</p>
<h4 data-nodeid="630">卸载已有 Docker</h4>
<p data-nodeid="631">如果你已经安装过旧版的 Docker，可以先执行以下命令卸载旧版 Docker。</p>
<pre class="lang-shell" data-nodeid="632"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo yum remove docker \</span>
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
</code></pre>
<h4 data-nodeid="633">安装 Docker</h4>
<p data-nodeid="634">首次安装 Docker 之前，需要添加 Docker 安装源。添加之后，我们就可以从已经配置好的源，安装和更新 Docker。添加 Docker 安装源的命令如下：</p>
<pre class="lang-shell" data-nodeid="635"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo yum-config-manager \</span>
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
</code></pre>
<p data-nodeid="636">正常情况下，直接安装最新版本的 Docker 即可，因为最新版本的 Docker 有着更好的稳定性和安全性。你可以使用以下命令安装最新版本的 Docker。</p>
<pre class="lang-shell" data-nodeid="637"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo yum install docker-ce docker-ce-cli containerd.io</span>
</code></pre>
<p data-nodeid="638">如果你想要安装指定版本的 Docker，可以使用以下命令：</p>
<pre class="lang-shell" data-nodeid="639"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo yum list docker-ce --showduplicates | sort -r</span>
docker-ce.x86_64            18.06.1.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                   docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos            docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos            docker-ce-stable
</code></pre>
<p data-nodeid="640">然后选取想要的版本执行以下命令：</p>
<pre class="lang-shell" data-nodeid="641"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo yum install docker-ce-&lt;VERSION_STRING&gt; docker-ce-cli-&lt;VERSION_STRING&gt; containerd.io</span>
</code></pre>
<p data-nodeid="642">安装完成后，使用以下命令启动 Docker。</p>
<pre class="lang-shell" data-nodeid="643"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo systemctl start docker</span>
</code></pre>
<p data-nodeid="644">这里有一个国际惯例，安装完成后，我们需要使用以下命令启动一个 hello world 的容器。</p>
<pre class="lang-shell" data-nodeid="645"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo docker run hello-world</span>
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete
Digest: sha256:7f0a9f93b4aa3022c3a4c147a449bf11e0941a1fd0bf4a8e6c9408b2600777c5
Status: Downloaded newer image for hello-world:latest
Hello from Docker!
</code></pre>
<p data-nodeid="646">运行上述命令，Docker 首先会检查本地是否有<code data-backticks="1" data-nodeid="724">hello-world</code>这个镜像，如果发现本地没有这个镜像，Docker 就会去 Docker Hub 官方仓库下载此镜像，然后运行它。最后我们看到该镜像输出 "Hello from Docker!" 并退出。</p>
<blockquote data-nodeid="647">
<p data-nodeid="648">安装完成后默认 docker 命令只能以 root 用户执行，如果想允许普通用户执行 docker 命令，需要执行以下命令 sudo groupadd docker &amp;&amp; sudo gpasswd -a ${USER} docker &amp;&amp; sudo systemctl restart docker ，执行完命令后，退出当前命令行窗口并打开新的窗口即可。</p>
</blockquote>
<p data-nodeid="649">安装完 Docker，先不着急使用，先来了解下容器的技术原理，这样才能知其所以然。</p>
<h3 data-nodeid="650">容器技术原理</h3>
<p data-nodeid="651">提起容器就不得不说 chroot，因为 chroot 是最早的容器雏形。chroot 意味着切换根目录，有了 chroot 就意味着我们可以把任何目录更改为当前进程的根目录，这与容器非常相似，下面我们通过一个实例了解下 chroot。</p>
<h4 data-nodeid="652">chroot</h4>
<p data-nodeid="653">什么是 chroot 呢？下面是 chroot 维基百科定义：</p>
<blockquote data-nodeid="654">
<p data-nodeid="655">chroot 是在 Unix 和 Linux 系统的一个操作，针对正在运作的软件行程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由 chroot 设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。</p>
</blockquote>
<p data-nodeid="656">通俗地说 ，chroot 就是可以改变某进程的根目录，使这个程序不能访问目录之外的其他目录，这个跟我们在一个容器中是很相似的。下面我们通过一个实例来演示下 chroot。</p>
<p data-nodeid="657">首先我们在当前目录下创建一个 rootfs 目录：</p>
<pre class="lang-shell" data-nodeid="658"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> mkdir rootfs</span>
</code></pre>
<p data-nodeid="659">这里为了方便演示，我使用现成的 busybox 镜像来创建一个系统，镜像的概念和组成后面我会详细讲解，如果你没有 Docker 基础可以把下面的操作命令理解成在 rootfs 下创建了一些目录和放置了一些二进制文件。</p>
<pre class="lang-shell" data-nodeid="660"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> <span class="hljs-built_in">cd</span> rootfs </span>
<span class="hljs-meta">$</span><span class="bash"> docker <span class="hljs-built_in">export</span> $(docker create busybox) -o busybox.tar</span>
<span class="hljs-meta">$</span><span class="bash"> tar -xf busybox.tar</span>
</code></pre>
<p data-nodeid="661">执行完上面的命令后，在 rootfs 目录下，我们会得到一些目录和文件。下面我们使用 ls 命令查看一下 rootfs 目录下的内容。</p>
<pre class="lang-shell" data-nodeid="662"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> ls</span>
bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  var
</code></pre>
<p data-nodeid="663">可以看到我们在 rootfs 目录下初始化了一些目录，下面让我们通过一条命令来见证 chroot 的神奇之处。使用以下命令，可以启动一个 sh 进程，并且把 /home/centos/rootfs 作为 sh 进程的根目录。</p>
<pre class="lang-shell" data-nodeid="664"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> chroot /home/centos/rootfs /bin/sh</span>
</code></pre>
<p data-nodeid="665">此时，我们的命令行窗口已经处于上述命令启动的 sh 进程中。在当前 sh 命令行窗口下，我们使用 ls 命令查看一下当前进程，看是否真的与主机上的其他目录隔离开了。</p>
<pre class="lang-dart" data-nodeid="666"><code data-language="dart">/ # /bin/ls /
bin  busybox.tar  dev  etc  home  proc  root  sys  tmp  usr  <span class="hljs-keyword">var</span>
</code></pre>
<p data-nodeid="667">这里可以看到当前进程的根目录已经变成了主机上的 /home/centos/rootfs 目录。这样就实现了当前进程与主机的隔离。到此为止，一个目录隔离的容器就完成了。<br>
但是，此时还不能称之为一个容器，为什么呢？你可以在上一步（使用 chroot 启动命令行窗口）执行以下命令，查看如下路由信息：</p>
<pre class="lang-dart" data-nodeid="776"><code data-language="dart">/etc # /bin/ip route
<span class="hljs-keyword">default</span> via <span class="hljs-number">172.20</span><span class="hljs-number">.1</span><span class="hljs-number">.1</span> dev eth0
<span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>/<span class="hljs-number">16</span> dev docker0 scope link  src <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>
<span class="hljs-number">172.20</span><span class="hljs-number">.1</span><span class="hljs-number">.0</span>/<span class="hljs-number">24</span> dev eth0 scope link  src <span class="hljs-number">172.20</span><span class="hljs-number">.1</span><span class="hljs-number">.3</span>
</code></pre>

<p data-nodeid="669">执行 ip route 命令后，你可以看到网络信息并没有隔离，实际上进程等信息此时也并未隔离。要想实现一个完整的容器，我们还需要 Linux 的其他三项技术： Namespace、Cgroups 和联合文件系统。</p>
<p data-nodeid="670">Docker 是利用 Linux 的 Namespace 、Cgroups 和联合文件系统三大机制来保证实现的， 所以它的原理是使用 Namespace 做主机名、网络、PID 等资源的隔离，使用 Cgroups 对进程或者进程组做资源（例如：CPU、内存等）的限制，联合文件系统用于镜像构建和容器运行环境。</p>
<p data-nodeid="671">后面我会对这些技术进行详细讲解，这里我就简单解释下它们的作用。</p>
<h4 data-nodeid="672">Namespace</h4>
<p data-nodeid="673">Namespace 是 Linux 内核的一项功能，该功能对内核资源进行隔离，使得容器中的进程都可以在单独的命名空间中运行，并且只可以访问当前容器命名空间的资源。Namespace 可以隔离进程 ID、主机名、用户 ID、文件名、网络访问和进程间通信等相关资源。</p>
<p data-nodeid="674">Docker 主要用到以下五种命名空间。</p>
<ul data-nodeid="675">
<li data-nodeid="676">
<p data-nodeid="677">pid namespace：用于隔离进程 ID。</p>
</li>
<li data-nodeid="678">
<p data-nodeid="679">net namespace：隔离网络接口，在虚拟的 net namespace 内用户可以拥有自己独立的 IP、路由、端口等。</p>
</li>
<li data-nodeid="680">
<p data-nodeid="681">mnt namespace：文件系统挂载点隔离。</p>
</li>
<li data-nodeid="682">
<p data-nodeid="683">ipc namespace：信号量,消息队列和共享内存的隔离。</p>
</li>
<li data-nodeid="684">
<p data-nodeid="685">uts namespace：主机名和域名的隔离。</p>
</li>
</ul>
<h4 data-nodeid="686">Cgroups</h4>
<p data-nodeid="687">Cgroups 是一种 Linux 内核功能，可以限制和隔离进程的资源使用情况（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用。</p>
<h4 data-nodeid="688">联合文件系统</h4>
<p data-nodeid="689">联合文件系统，又叫 UnionFS，是一种通过创建文件层进程操作的文件系统，因此，联合文件系统非常轻快。Docker 使用联合文件系统为容器提供构建层，使得容器可以实现写时复制以及镜像的分层构建和存储。常用的联合文件系统有 AUFS、Overlay 和 Devicemapper 等。</p>
<h3 data-nodeid="690">结语</h3>
<p data-nodeid="691">容器技术从 1979 年 chroot 的首次问世便已崭露头角，但是到了 2013 年，Dokcer 的横空出世才使得容器技术迅速发展，可见 Docker 对于容器技术的推动力和影响力。</p>
<blockquote data-nodeid="692">
<p data-nodeid="693">另外， Docker 还提供了工具和平台来管理容器的生命周期：</p>
<ol data-nodeid="694">
<li data-nodeid="695">
<p data-nodeid="696">使用容器开发应用程序及其支持组件。</p>
</li>
<li data-nodeid="697">
<p data-nodeid="698">容器成为分发和测试你的应用程序的单元。</p>
</li>
<li data-nodeid="699">
<p data-nodeid="700">可以将应用程序作为容器或协调服务部署到生产环境中。无论您的生产环境是本地数据中心，云提供商还是两者的混合，其工作原理都相同。</p>
</li>
</ol>
</blockquote>
<p data-nodeid="701">到此，相信你已经了解了实现容器的基本技术原理，并且对 Docker 的作用有了一定认知。那么你知道为什么容器技术在 Docker 出现之前一直没有爆发的根本原因吗？思考后，可以把你的想法写在留言区。</p>
<p data-nodeid="702" class="">下一课时，我将讲解 Docker 的架构设计以及 Docker 的三大核心概念。</p>

---

### 精选评论

##### *磊：
> Docker爆发的原因是，在同类产品的基础上加入了镜像功能。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没错哈，Docker 爆发的原因是加入了镜像功能，并且封装了镜像仓库使得镜像分发更加方便

##### **飞：
> ppt能发一下？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 亲，请关注拉勾教育公众号，回复“Docker”获取课件哦。

##### *锚：
> 硬件性能的发展阻碍了容器的发展吧，当硬件性能大量过剩时，虚拟机、容器化极速发展壮大，云产品也是快速扩张，极大的降低了各种资源入门成本，为中小企业以及个人创新创造了巨大的便利

##### **仙：
> 【安装问题】想问下老师如果安装在其他目录，要怎么配置才能直接使用docker ps 命令呢？我把docker安装在其他目录了，然后系统找不到docker命令

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以把 docker 二进制文件通过软连的方式 连接到 /usr/bin/docker 下

##### *俊：
> 终于可以系统的学习docker了

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油加油，有啥不会、不懂、有疑问的随时留言，郭少老师一一回复。

##### **平：
> docker容器技术的爆发：主要是还是对开发环境的依赖打包发布，让部署一个应用不在那么麻烦，需要按照各种环境依赖，耗费大量的时间。记得我曾经做LAMP环境部署的时候，php的各种依赖包，一个环境安装一整天是常有的事儿，有了docker后，一次安装，之后只要更新容器内的代码就可以啦。

##### *超：
> 老师，启动时遇到的问题：# systemctl start dockerFailed to get D-Bus connection: Operation not permitted环境WIN10 WSL CENTOS7.6

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请确认下启动用户是否是 root 用户，使用 root 用户启动即可

##### **0588：
> 市场的需要催生了应用的变革，硬件飞速发展的速度奠定了基础。封装后的镜像仓库更便于移植。

##### **民：
> 坚持学习

##### **龙：
> 用了chroot切换后，为什么命令都失效了，找不到命令行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用ctrl+c 命令可以退出当前运行环境。

##### **杰：
> 老师，安装docker，服务器至少要啥配置啊，我一核一G的centos7.2能安装么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以安装，但是推荐至少 2 核 4G 的配置

##### **7036：
> 老师的这篇文章写的通俗易懂，所有命令可直接拷贝运行，非常用心。

##### **小菜鸟：
> PPT挺漂亮的

##### **安：
> chroot 命令运行过后，运行ls ，提示ls:not found. 得到不老师演示的结果

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以尝试执行 /bin/ls 命令

##### **5080：
> 为什么 执行：docker export $(docker create busybox) -o busybox.tar 没有任何反应，什么都没有

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 查看一下执行命令的目录，会生成一个 busybox.tar 文件

##### *杨：
> 辛苦老师帮忙看下~Mac执行：(TestEnv) otakz@OtakzdeMBP env % sudo chroot rootfs /bin/shchroot: /bin/sh: Exec format error

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 无法直接在 mac 下运行，请在 Linux 系统下执行相关命令

##### *四：
> 联合文件系统，是一种通过创建文件层进程操作的文件系统老师，请问这句话怎么解析啊！理解不了！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以参考 14 讲的联合文件系统内容

##### Ddd：
> 老师，版本比较老，1.12.3版本，离线安装的，如何升级到最新版本？

##### *冲：
> sudo ps aux|grep dockerdroot  0:03 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sockroot  0:00 grep --color=auto dockerdpstree -l -a -A 1198dockerd -H fd:// --containerd=/run/containerd/containerd.sock `-18*[{dockerd}]这种是什么问题?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 根据您的 docker 版本决定使用  sudo ps aux|grep dockerd  或者 sudo ps aux|grep containerd 查看 docker 组件间的进程关系。如果 docker 是 19 版本以后，使用 sudo ps aux|grep containerd 查看

##### *冲：
> chroot和namespace是什么关系？还是仅仅相似

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; chroot 是早期的一个类似于容器的实现工具，和 namespace 并没有本质联系哈

##### **的蜗牛：
> MacBook Pro能按讲义里的步骤操作吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 最好使用 Linux 操作系统来做后面的实验哈，Mac 可以安装一个 Linux 虚拟机

##### *冲：
> overlay2存储驱动程序相比于overlay有什么好处吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; overlay2 修复了  overlay 中的一些硬性限制，譬如文件链接数的限制。
overlay 有很多设计缺陷，不建议生产环境使用。

##### *杰：
> chroot /home/centos/rootfs/bin/sh的时候说cannotchange

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请详细描述一下错误哈，chroot /home/centos/rootfs 和 /bin/sh 之前是有空格的

##### **9465：
> 请问命名空间为什么没有用到user namespace？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 早期的 Docker 版本没有用到 User Namespace，最新版本的 Docker 已经用到了~

##### **丰：
> 老师您好，这个问题解决了，我将docker0网卡的网段改了重启docker就好了，docker的默认网段和vpn的冲突了，所以开启后收不到回包了，导致的ssh登陆失败

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 很棒！

##### **鹏：
> 79年没爆发起来，应该是互联网还没发展起来，没有很迫切的需求，而今大公司的业务系统复杂度特别高，需要做业务分离。

##### **丰：
> 按照上面说的安装完毕后在系统层面产生一个"docker0"的网卡，导致系统层面无法被ssh，想请问这个网卡是必须的么，还是可以配置关闭的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker0 网卡默认不会对主机网络造成影响，导致主机无法登录是否因为 Docker 的网络与内网网段冲突了？

##### **涛：
> 请问centos7用root用户执行chroot命令后执行ls提示/bin/sh ls not found是什么问题？chroot /opt/rootfs /bin/sh

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请确认 /opt/rootfs 目录下是否准备好了 busybox 的文件系统

##### felixwon：
> 镜像可以在不同服务（项目）间共享吗？比如MySQL镜像或redis镜像

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 镜像本来就是共享的哈，容器层都是写时复制的机制。

##### **7228：
> 之前没有爆发的根本原因应该是跟镜像无关，能够爆发只是因为信息化的深入与计算机的算力的相互提升，带来的架构设计模式，开发模式的不断演进，所自然而然成长起来的。

##### **鸿：
> 很多人说Docker环境搬迁，避免环境问题导致不同的结果？这是怎么理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为 Docker 不仅打包了应用程序，还打包了应用程序所需要的运行环境，因此 Docker 相对于传统的业务部署模式，可以做到无论运行在哪里运行结果都是一致的。

##### *毅：
> 这一章用ubuntu装会影响后续的学习吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 只要是 linux 操作系统都不影响后面的学习哈~

##### **腾：
> 镜像仓库等功能使得Docker的生态更加强大。

##### *新：
> centos7.8 使用root运行$ docker export $(docker create busybox) -o busybox.tar这条命令没有问题，使用非root用户运行不论是否加sudo都报错Got permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Post http://%2Fvar%2Frun%2Fdocker.sock/v1.40/containers/create: dial unix /var/run/docker.sock: connect: permission denied"docker export" requires exactly 1 argument.See 'docker export --help'.Usage: docker export [OPTIONS] CONTAINERExport a container's filesystem as a tar archive请问是什么问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; CentOS 7 默认 docker 命令只有 root 用户可以操作，请使用 sudo

##### **飞：
> 请问centos7用root用户执行chroot命令后执行ls提示/bin/sh ls not found是什么问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 具体执行的命令和目录是什么样的？

##### *博：
> 隔壁老王推荐的。错不了！

##### **8200：
> 请问怎么理解docker的源呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 简单来讲 Docker 安装源定义了去哪里下载 Docker 的安装包

##### **华：
> Docker爆发的原因：镜像功能，镜像仓库，对开发很友好。

##### Edison Zhou：
> 如果只有容器，那只做到了标准化隔离运行，而标准化打包这个开发者最关心的问题没有解决，直到容器镜像出现，镜像仓库就成为了标配，才能把集装箱的特质发挥出来。

##### **0187：
> 老师，请教一下云原生和这个docker有什么关系吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 云原生的定义可以参考这里，https://github.com/cncf/toc/blob/master/DEFINITION.md，容器是是云原生的重要代表技术

##### **雄：
> 停过好几个版本的docker课，只有老师把底层讲出来了

##### **3336：
> 可以直接 windows 安装嘛，还是先虚拟机在安装

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 推荐先安装虚拟机，在虚拟机中安装 Docker

##### **彬：
> ubuntu 18.04上，创建目录，然后执行chroot，失败：```[~]$chroot /home/ubuntu/rootfs /bin/shchroot: failed to run command ‘/bin/sh’: No such file or directory[~]$ ll /bin/shlrwxrwxrwx 1 root root 4 Dec dash```实际sh是存在的，请问是什么原因

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请确保按照课程命令一步一步执行。这个报错原因可能是因为 /home/ubuntu/rootfs 目录下 没有 /bin/sh 文件

##### **用户2411：
> 火的原因是不是也可以这么理解，是docker 镜像仓库诞生以及官方镜像的支持，加速了docker 的发展

##### *峰：
> 涨知识了

##### **宣：
> 之前直接用 yum install docker-ce直接安装，这与老师的安装方式有什么不同吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 直接 yum install 安装的 Docker 可能不是最新的，添加官方源可以保证安装到最新版本的 Docker

##### *娃：
> 是因为之前内核不支持namespace？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 爆发的原因是加入了镜像功能，并且封装了镜像仓库使得镜像分发更加方便

