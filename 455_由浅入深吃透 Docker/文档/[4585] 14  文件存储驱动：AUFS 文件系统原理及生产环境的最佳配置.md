<p data-nodeid="837" class="">我们知道，Docker 主要是基于 Namespace、cgroups 和联合文件系统这三大核心技术实现的。前面的课时我详细讲解了 Namespace 和 cgroups 的相关原理，那么你知道联合文件系统是什么吗？它的原理又是什么呢？</p>
<p data-nodeid="838">首先我们来了解一下什么是联合文件系统。</p>
<h3 data-nodeid="839">什么是联合文件系统</h3>
<p data-nodeid="840">联合文件系统（Union File System，Unionfs）是一种分层的轻量级文件系统，它可以把多个目录内容联合挂载到同一目录下，从而形成一个单一的文件系统，这种特性可以让使用者像是使用一个目录一样使用联合文件系统。</p>
<p data-nodeid="3553" class="">那联合文件系统对于 Docker 是一个怎样的存在呢？它可以说是 Docker 镜像和容器的基础，因为它可以使 Docker 可以把镜像做成分层的结构，从而使得镜像的每一层可以被共享。例如两个业务镜像都是基于 CentOS 7 镜像构建的，那么这两个业务镜像在物理机上只需要存储一次 CentOS 7 这个基础镜像即可，从而节省大量存储空间。</p>



<p data-nodeid="842">说到这儿，你有没有发现，联合文件系统只是一个概念，真正实现联合文件系统才是关键，那如何实现呢？其实实现方案有很多，Docker 中最常用的联合文件系统有三种：AUFS、Devicemapper 和 OverlayFS。</p>
<p data-nodeid="843">今天我主要讲解 Docker 中最常用的联合文件系统里的 AUFS，为什么呢？因为 AUFS 是 Docker 最早使用的文件系统驱动，多用于 Ubuntu 和 Debian 系统中。在 Docker 早期，OverlayFS 和 Devicemapper 相对不够成熟，AUFS 是最早也是最稳定的文件系统驱动。 Devicemapper 和 OverlayFS 联合文件系统，我将在第 15 和 16 课时为你详细剖析 。</p>
<p data-nodeid="844">接下来，我们就看看如何配置 Docker 的 AUFS 模式。</p>
<h3 data-nodeid="845">如何配置 Docker 的 AUFS 模式</h3>
<p data-nodeid="846">AUFS 目前并未被合并到 Linux 内核主线，因此只有 Ubuntu 和 Debian 等少数操作系统支持 AUFS。你可以使用以下命令查看你的系统是否支持 AUFS：</p>
<pre class="lang-java" data-nodeid="847"><code data-language="java">$ grep aufs /proc/filesystems
nodev   aufs
</code></pre>
<p data-nodeid="848">执行以上命令后，如果输出结果包含<code data-backticks="1" data-nodeid="953">aufs</code>，则代表当前操作系统支持 AUFS。AUFS 推荐在 Ubuntu 或 Debian 操作系统下使用，如果你想要在 CentOS 等操作系统下使用 AUFS，需要单独安装 AUFS 模块（生产环境不推荐在 CentOS 下使用 AUFS，如果你想在 CentOS 下安装 AUFS 用于研究和测试，可以参考这个<a href="https://github.com/bnied/kernel-ml-aufs" data-nodeid="957">链接</a>），安装完成后使用上述命令输出结果中有<code data-backticks="1" data-nodeid="959">aufs</code>即可。<br>
当确认完操作系统支持 AUFS 后，你就可以配置 Docker 的启动参数了。</p>
<p data-nodeid="849">先在 /etc/docker 下新建 daemon.json 文件，并写入以下内容：</p>
<pre class="lang-json" data-nodeid="850"><code data-language="json">{
&nbsp; <span class="hljs-attr">"storage-driver"</span>: <span class="hljs-string">"aufs"</span>
}
</code></pre>
<p data-nodeid="851">然后使用以下命令重启 Docker：</p>
<pre class="lang-java" data-nodeid="852"><code data-language="java">$ sudo systemctl restart docker
</code></pre>
<p data-nodeid="853">Docker 重启以后使用<code data-backticks="1" data-nodeid="966">docker info</code>命令即可查看配置是否生效：</p>
<pre class="lang-java" data-nodeid="854"><code data-language="java">$ sudo docker info
Client:
&nbsp;Debug Mode: <span class="hljs-keyword">false</span>
Server:
&nbsp;Containers: <span class="hljs-number">0</span>
&nbsp; Running: <span class="hljs-number">0</span>
&nbsp; Paused: <span class="hljs-number">0</span>
&nbsp; Stopped: <span class="hljs-number">0</span>
&nbsp;Images: <span class="hljs-number">1</span>
&nbsp;Server Version: <span class="hljs-number">19.03</span>.<span class="hljs-number">12</span>
&nbsp;Storage Driver: aufs
&nbsp; Root Dir: /<span class="hljs-keyword">var</span>/lib/docker/aufs
&nbsp; Backing Filesystem: extfs
&nbsp; Dirs: <span class="hljs-number">1</span>
&nbsp; Dirperm1 Supported: <span class="hljs-keyword">true</span>
</code></pre>
<p data-nodeid="855">可以看到 Storage Driver 已经变为 aufs，证明配置已经生效，配置生效后就可以使用 AUFS 为 Docker 提供联合文件系统了。</p>
<p data-nodeid="856">配置好 Docker 的 AUFS 联合文件系统后，你一定很好奇 AUFS 到底是如何工作的呢？下面我带你详细学习一下 AUFS 的工作原理。</p>
<h3 data-nodeid="857">AUFS 工作原理</h3>
<h4 data-nodeid="858">AUFS 是如何存储文件的？</h4>
<p data-nodeid="859">AUFS 是联合文件系统，意味着它在主机上使用多层目录存储，<strong data-nodeid="976">每一个目录在 AUFS 中都叫作分支，而在 Docker 中则称之为层（layer），但最终呈现给用户的则是一个普通单层的文件系统，我们把多层以单一层的方式呈现出来的过程叫作联合挂载。</strong></p>
<p data-nodeid="860"><img src="https://s0.lgstatic.com/i/image/M00/5E/82/CgqCHl-GwcCAOu4aAABzKSlpRlI180.png" alt="Lark20201014-171313.png" data-nodeid="979"></p>
<div data-nodeid="861"><p style="text-align:center">图 1 AUFS 工作原理示意图</p></div>
<p data-nodeid="862">如图 1 所示，每一个镜像层和容器层都是 /var/lib/docker 下的一个子目录，镜像层和容器层都在 aufs/diff 目录下，每一层的目录名称是镜像或容器的 ID 值，联合挂载点在 aufs/mnt 目录下，mnt 目录是真正的容器工作目录。</p>
<p data-nodeid="863">下面我们针对 aufs 文件夹下的各目录结构，在创建容器前后的变化做详细讲述。</p>
<p data-nodeid="864">当一个镜像未生成容器时，AUFS 的存储结构如下。</p>
<ul data-nodeid="865">
<li data-nodeid="866">
<p data-nodeid="867">diff 文件夹：存储镜像内容，每一层都存储在以镜像层 ID 命名的子文件夹中。</p>
</li>
<li data-nodeid="868">
<p data-nodeid="869">layers 文件夹：存储镜像层关系的元数据，在 diif 文件夹下的每个镜像层在这里都会有一个文件，文件的内容为该层镜像的父级镜像的 ID。</p>
</li>
<li data-nodeid="870">
<p data-nodeid="871">mnt 文件夹：联合挂载点目录，未生成容器时，该目录为空。</p>
</li>
</ul>
<p data-nodeid="1463" class="">当一个镜像已经生成容器时，AUFS 存储结构会发生如下变化。</p>


<ul data-nodeid="873">
<li data-nodeid="874">
<p data-nodeid="875">diff 文件夹：当容器运行时，会在 diff 目录下生成容器层。</p>
</li>
<li data-nodeid="876">
<p data-nodeid="877">layers 文件夹：增加容器层相关的元数据。</p>
</li>
<li data-nodeid="878">
<p data-nodeid="879">mnt 文件夹：容器的联合挂载点，这和容器中看到的文件内容一致。</p>
</li>
</ul>
<p data-nodeid="880">以上便是 AUFS 的工作原理，那你知道容器的在工作过程中是如何使用 AUFS 的吗？</p>
<h4 data-nodeid="881">AUFS 是如何工作的？</h4>
<p data-nodeid="882">AUFS 的工作过程中对文件的操作分为读取文件和修改文件。下面我们分别来看下 AUFS 对于不同的文件操作是如何工作的。</p>
<h5 data-nodeid="883">1. 读取文件</h5>
<p data-nodeid="884">当我们在容器中读取文件时，可能会有以下场景。</p>
<ul data-nodeid="885">
<li data-nodeid="886">
<p data-nodeid="887">文件在容器层中存在时：当文件存在于容器层时，直接从容器层读取。</p>
</li>
<li data-nodeid="888">
<p data-nodeid="889">当文件在容器层中不存在时：当容器运行时需要读取某个文件，如果容器层中不存在时，则从镜像层查找该文件，然后读取文件内容。</p>
</li>
<li data-nodeid="890">
<p data-nodeid="891">文件既存在于镜像层，又存在于容器层：当我们读取的文件既存在于镜像层，又存在于容器层时，将会从容器层读取该文件。</p>
</li>
</ul>
<h5 data-nodeid="892">2. 修改文件或目录</h5>
<p data-nodeid="893">AUFS 对文件的修改采用的是写时复制的工作机制，这种工作机制可以最大程度节省存储空间。</p>
<p data-nodeid="2299" class="">具体的文件操作机制如下。</p>


<ul data-nodeid="895">
<li data-nodeid="896">
<p data-nodeid="897">第一次修改文件：当我们第一次在容器中修改某个文件时，AUFS 会触发写时复制操作，AUFS 首先从镜像层复制文件到容器层，然后再执行对应的修改操作。</p>
</li>
</ul>
<blockquote data-nodeid="898">
<p data-nodeid="899">AUFS 写时复制的操作将会复制整个文件，如果文件过大，将会大大降低文件系统的性能，因此当我们有大量文件需要被修改时，AUFS 可能会出现明显的延迟。好在，写时复制操作只在第一次修改文件时触发，对日常使用没有太大影响。</p>
</blockquote>
<ul data-nodeid="900">
<li data-nodeid="901">
<p data-nodeid="902">删除文件或目录：当文件或目录被删除时，AUFS 并不会真正从镜像中删除它，因为镜像层是只读的，AUFS 会创建一个特殊的文件或文件夹，这种特殊的文件或文件夹会阻止容器的访问。</p>
</li>
</ul>
<p data-nodeid="903">下面我们通过一个实例来演示一下 AUFS 。</p>
<h3 data-nodeid="904">AUFS 演示</h3>
<h4 data-nodeid="905">准备演示目录和文件</h4>
<p data-nodeid="906">首先我们在 /tmp 目录下创建 aufs 目录：</p>
<pre class="lang-java" data-nodeid="907"><code data-language="java">$ cd /tmp
/tmp$ mkdir aufs
</code></pre>
<p data-nodeid="908">准备挂载点目录：</p>
<pre class="lang-java" data-nodeid="909"><code data-language="java">/tmp$ cd aufs
/tmp/aufs$ mkdir mnt
</code></pre>
<p data-nodeid="910">接下来准备容器层内容：</p>
<pre class="lang-java" data-nodeid="911"><code data-language="java">## 创建镜像层目录
/tmp/aufs$ mkdir container1
## 在镜像层目录下准备一个文件
/tmp/aufs$&nbsp;echo Hello, Container layer! &gt; container1/container1.txt
</code></pre>
<p data-nodeid="912">最后准备镜像层内容：</p>
<pre class="lang-java" data-nodeid="913"><code data-language="java">## 创建两个镜像层目录
/tmp/aufs$&nbsp;mkdir image1 &amp;&amp; mkdir image2
## 分别写入数据
/tmp/aufs$&nbsp;echo Hello, Image layer1! &gt; image1/image1.txt
/tmp/aufs$&nbsp;echo Hello, Image layer2! &gt; image2/image2.txt
</code></pre>
<p data-nodeid="914">准备好的目录和文件结构如下：</p>
<pre class="lang-java" data-nodeid="915"><code data-language="java">/tmp/aufs$ tree .
.
|-- container1
|&nbsp; &nbsp;`-- container1.txt
|-- image1
|&nbsp; &nbsp;`-- image1.txt
|-- image2
|&nbsp; &nbsp;`-- image2.txt
`-- mnt
<span class="hljs-number">4</span> directories, <span class="hljs-number">3</span> files
</code></pre>
<h4 data-nodeid="916">创建 AUFS 联合文件系统</h4>
<p data-nodeid="917">使用 mount 命令可以创建 AUFS 类型的文件系统，命令如下：</p>
<pre class="lang-java" data-nodeid="918"><code data-language="java">/tmp/aufs$ sudo mount -t aufs -o dirs=./container1:./image2:./image1&nbsp; none ./mnt
</code></pre>
<p data-nodeid="919">mount 命令创建 AUFS 类型文件系统时，这里要注意，<strong data-nodeid="1022">dirs 参数第一个冒号默认为读写权限，后面的目录均为只读权限，与 Docker 容器使用 AUFS 的模式一致。</strong></p>
<p data-nodeid="920">执行完上述命令后，mnt 变成了 AUFS 的联合挂载目录，我们可以使用 mount 命令查看一下已经创建的 AUFS 文件系统：</p>
<pre class="lang-java" data-nodeid="921"><code data-language="java">/tmp/aufs$ mount -t aufs
none on /tmp/aufs/<span class="hljs-function">mnt type <span class="hljs-title">aufs</span> <span class="hljs-params">(rw,relatime,si=<span class="hljs-number">4174</span>b83d649ffb7c)</span>
</span></code></pre>
<p data-nodeid="922">我们每创建一个 AUFS 文件系统，AUFS 都会为我们生成一个 ID，这个 ID 在 /sys/fs/aufs/ 会创建对应的目录，在这个 ID 的目录下可以查看文件挂载的权限。</p>
<pre class="lang-java" data-nodeid="923"><code data-language="java">tmp/aufs$ cat /sys/fs/aufs/si_4174b83d649ffb7c<span class="hljs-comment">/*
/tmp/aufs/container1=rw
/tmp/aufs/image2=ro
/tmp/aufs/image1=ro
64
65
66
</span></code></pre>
<p data-nodeid="924">可以看到 container1 目录的权限为 rw（代表可读写），image1 和 image2 的权限为 ro（代表只读）。</p>
<p data-nodeid="925">为了验证 mnt 目录下可以看到 container1、image1 和 image2 目录下的所有内容，我们使用 ls 命令查看一下 mnt 目录：</p>
<pre class="lang-java" data-nodeid="926"><code data-language="java">/tmp/aufs$&nbsp;ls -l mnt/
total <span class="hljs-number">12</span>
-rw-rw-r-- <span class="hljs-number">1</span> ubuntu ubuntu <span class="hljs-number">24</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">16</span>:<span class="hljs-number">55</span> container1.txt
-rw-rw-r-- <span class="hljs-number">1</span> ubuntu ubuntu <span class="hljs-number">21</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">16</span>:<span class="hljs-number">59</span> image1.txt
-rw-rw-r-- <span class="hljs-number">1</span> ubuntu ubuntu <span class="hljs-number">21</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">16</span>:<span class="hljs-number">59</span> image2.txt
</code></pre>
<p data-nodeid="927">可以看到 mnt 目录下已经出现了我们准备的所有镜像层和容器层的文件。下面让我们来验证一下 AUFS 的写时复制。</p>
<h4 data-nodeid="928">验证 AUFS 的写时复制</h4>
<p data-nodeid="929">AUFS 的写时复制是指在容器中，只有需要修改某个文件时，才会把文件从镜像层复制到容器层，下面我们通过修改联合挂载目录 mnt 下的内容来验证下这个过程。</p>
<p data-nodeid="930">我们使用以下命令修改 mnt 目录下的 image1.txt 文件：</p>
<pre class="lang-java" data-nodeid="931"><code data-language="java">/tmp/aufs$ echo Hello, Image layer1 changed! &gt; mnt/image1.txt
</code></pre>
<p data-nodeid="932">然后我们查看下 image1/image1.txt 文件内容：</p>
<pre class="lang-java" data-nodeid="933"><code data-language="java">/tmp/aufs$ cat image1/image1.txt
Hello, Image layer1!
</code></pre>
<p data-nodeid="934">发现“镜像层”的 image1.txt 文件并未被修改。<br>
然后我们查看一下"容器层"对应的 image1.txt 文件内容：</p>
<pre class="lang-java" data-nodeid="935"><code data-language="java">/tmp/aufs$ ls -l container1/
total 8
-rw-rw-r-- 1 ubuntu ubuntu 24 Sep&nbsp; 9 16:55 container1.txt
-rw-rw-r-- 1 ubuntu ubuntu 29 Sep&nbsp; 9 17:21 image1.txt
## 查看文件内容
/tmp/aufs$ cat container1/image1.txt
Hello, Image layer1 changed!
</code></pre>
<p data-nodeid="936">发现 AUFS 在“容器层”自动创建了 image1.txt 文件，并且内容为我们刚才写入的内容。</p>
<p data-nodeid="937">至此，我们完成了 AUFS 写时复制的验证。我们在第一次修改镜像内某个文件时，AUFS 会复制这个文件到容器层，然后在容器层对该文件进行修改操作，这就是 AUFS 最典型的特性写时复制。</p>
<h3 data-nodeid="938">结语</h3>
<p data-nodeid="939">到此，相信你知道了联合文件系统是一种分层的轻量级文件系统，它可以把多个目录内容联合挂载到同一目录下，从而形成一个单一的文件系统。同时也学会了如何配置 Docker 使用 AUFS ，并且明白了 AUFS 的工作原理。</p>
<p data-nodeid="940">那么你知道 AUFS 为什么一直没能成功进入 Linux 内核主线吗？ 思考后，可以把你的想法写在留言区。</p>
<p data-nodeid="941" class="">下一课时，我将讲解 Docker 的另一个文件存储驱动：Devicemapper 文件系统原理及生产环境的最佳配置。</p>

---

### 精选评论

##### *豆：
> 没进入主线，是linus 认为他的代码写的太烂，后面有更好的取代品

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，提了好多次，Linus 都没有通过。

##### **8589：
> 老师，为什么我挂载aufs会报错呢[root@docker aufs]# mount -t aufs -o dirs=./container1:./image2:./image1 none ./mntmount: wrong fs type, bad option, bad superblock on none,missing codepage or helper program, or other errorIn some cases useful info is found in syslog - trydmesg | tail or so.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是内核或者文件系统不支持,可以提供下系统版本和内核版本吗?

##### *宁：
> daemon.json还原，docker服务起不来了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以查看下具体报错信息，一般是驱动参数配置错误导致的。

##### **7143：
> Z aufs]# cat /etc/reredhat-lsb/[root@iZ2vceej7yes1abpm7yec2Z aufs]# cat /etc/redhat-releaseCentOS Linux release 8.2.2004 (Core)执行这个 mount -t aufs -o dirs=./container1:./image2:./image1 none ./mnt报错:mount: /tmp/aufs/mnt: unknown filesystem type 'aufs'.我应该下载哪个版本的内核才支持aufs

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 想使用 aufs 最好使用 ubuntu 系统，centos 版本较低，需要升级内核

##### **艺：
> 评论说aufs代码写得太烂的麻烦尊重一下他人的技术成果以及贡献，你也不一定能作出这些贡献

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 并不是说代码烂哈，只是不符合 Linux 的设计哲学

##### *涛：
> grep aufs /proc/filesystems无输出结果，应该是 Debian10.7 不支持AUFS

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不支持 AUFS  的，你可以自行安装 AUFS 相关的模块

##### *冲：
> deepin不也是Ubuntu系的Linux吗,但deepin貌似也不支持aufs..

##### **升：
> 打卡

##### *瑞：
> 查了下资料，大概是因为 Linus 认为 AUFS 的代码写的太烂，不好维护，所以一直拒绝将其合并到 Linux 内核主线；除了 Debian Linux 合并了外，其它的都没有合并。

