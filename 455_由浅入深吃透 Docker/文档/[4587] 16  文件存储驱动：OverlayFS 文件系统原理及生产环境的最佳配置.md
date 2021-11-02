<p data-nodeid="231181">前面课时我分别介绍了 Docker 常见的联合文件系统解决方案： AUFS 和 Devicemapper。今天我给你介绍一个性能更好的联合文件系统解决方案—— OverlayFS。</p>



<p data-nodeid="230350">OverlayFS 的发展分为两个阶段。2014 年，OverlayFS 第一个版本被合并到 Linux 内核 3.18 版本中，此时的 OverlayFS 在 Docker 中被称为<code data-backticks="1" data-nodeid="230474">overlay</code>文件驱动。由于第一版的<code data-backticks="1" data-nodeid="230476">overlay</code>文件系统存在很多弊端（例如运行一段时间后Docker 会报 "too many links problem" 的错误）， Linux 内核在 4.0 版本对<code data-backticks="1" data-nodeid="230482">overlay</code>做了很多必要的改进，此时的 OverlayFS 被称之为<code data-backticks="1" data-nodeid="230484">overlay2</code>。</p>
<p data-nodeid="230351">因此，在 Docker 中 OverlayFS 文件驱动被分为了两种，一种是早期的<code data-backticks="1" data-nodeid="230487">overlay</code>，不推荐在生产环境中使用，另一种是更新和更稳定的<code data-backticks="1" data-nodeid="230489">overlay2</code>，推荐在生产环境中使用。下面的内容我们主要围绕<code data-backticks="1" data-nodeid="230491">overlay2</code>展开。</p>
<h3 data-nodeid="230352">使用 overlay2 的先决条件</h3>
<p data-nodeid="230353"><code data-backticks="1" data-nodeid="230494">overlay2</code>虽然很好，但是它的使用是有一定条件限制的。</p>
<ul data-nodeid="230354">
<li data-nodeid="230355">
<p data-nodeid="230356">要想使用<code data-backticks="1" data-nodeid="230497">overlay2</code>，Docker 版本必须高于 17.06.02。</p>
</li>
<li data-nodeid="230357">
<p data-nodeid="230358">如果你的操作系统是 RHEL 或 CentOS，Linux 内核版本必须使用 3.10.0-514 或者更高版本，其他 Linux 发行版的内核版本必须高于 4.0（例如 Ubuntu 或 Debian），你可以使用<code data-backticks="1" data-nodeid="230500">uname -a</code>查看当前系统的内核版本。</p>
</li>
<li data-nodeid="230359">
<p data-nodeid="230360"><code data-backticks="1" data-nodeid="230502">overlay2</code>最好搭配 xfs 文件系统使用，并且使用 xfs 作为底层文件系统时，d_type必须开启，可以使用以下命令验证 d_type 是否开启：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="230361"><code data-language="java">$ xfs_info /<span class="hljs-keyword">var</span>/lib/docker | grep ftype
naming&nbsp; &nbsp;=version <span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; bsize=<span class="hljs-number">4096</span>&nbsp; &nbsp;ascii-ci=<span class="hljs-number">0</span> ftype=<span class="hljs-number">1</span>
</code></pre>
<p data-nodeid="230362">当输出结果中有 ftype=1 时，表示 d_type 已经开启。如果你的输出结果为 ftype=0，则需要重新格式化磁盘目录，命令如下：</p>
<pre class="lang-java" data-nodeid="230363"><code data-language="java">$ sudo mkfs.xfs -f -n ftype=<span class="hljs-number">1</span> /path/to/disk
</code></pre>
<p data-nodeid="230364">另外，在生产环境中，推荐挂载 /var/lib/docker 目录到单独的磁盘或者磁盘分区，这样可以避免该目录写满影响主机的文件写入，并且把挂载信息写入到 /etc/fstab，防止机器重启后挂载信息丢失。</p>
<p data-nodeid="230365">挂载配置中推荐开启 pquota，这样可以防止某个容器写文件溢出导致整个容器目录空间被占满。写入到 /etc/fstab 中的内容如下：</p>
<pre class="lang-java" data-nodeid="230366"><code data-language="java">$UUID /<span class="hljs-keyword">var</span>/lib/docker xfs defaults,pquota <span class="hljs-number">0</span> <span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="230367">其中 UUID 为 /var/lib/docker 所在磁盘或者分区的 UUID 或者磁盘路径。<br>
如果你的操作系统无法满足上面的任何一个条件，那我推荐你使用 AUFS 或者 Devicemapper 作为你的 Docker 文件系统驱动。</p>
<blockquote data-nodeid="230368">
<p data-nodeid="230369">通常情况下， overlay2 会比 AUFS 和 Devicemapper 性能更好，而且更加稳定，因为 overlay2 在 inode 优化上更加高效。因此在生产环境中推荐使用 overlay2 作为 Docker 的文件驱动。</p>
</blockquote>
<p data-nodeid="230370">下面我通过实例来教你如何初始化  /var/lib/docker 目录，为后面配置 Docker 的<code data-backticks="1" data-nodeid="230518">overlay2</code>文件驱动做准备。</p>
<h4 data-nodeid="230371">准备  /var/lib/docker 目录</h4>
<p data-nodeid="233225" class="">1.使用 lsblk（Linux 查看磁盘和块设备信息命令）命令查看本机磁盘信息：</p>
<pre class="lang-java" data-nodeid="233226"><code data-language="java">$ lsblk
NAME&nbsp; &nbsp;MAJ:MIN RM&nbsp; SIZE RO TYPE MOUNTPOINT
vda&nbsp; &nbsp; <span class="hljs-number">253</span>:<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> disk
`-vda1 <span class="hljs-number">253</span>:<span class="hljs-number">1</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> part /
vdb&nbsp; &nbsp; <span class="hljs-number">253</span>:<span class="hljs-number">16</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> disk
`-vdb1 <span class="hljs-number">253</span>:<span class="hljs-number">17</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">8</span>G&nbsp; <span class="hljs-number">0</span> part
</code></pre>
<p data-nodeid="235435">可以看到，我的机器有两块磁盘，一块是 vda，一块是 vdb。其中 vda 已经被用来挂载系统根目录，这里我想把  /var/lib/docker  挂载到 vdb1 分区上。</p>
<p data-nodeid="235436">2.使用 mkfs 命令格式化磁盘 vdb1：</p>


<pre class="lang-java" data-nodeid="234703"><code data-language="java">$ sudo mkfs.xfs -f -n ftype=<span class="hljs-number">1</span> /dev/vdb1
</code></pre>
<p data-nodeid="235979" class="">3.将挂载信息写入到 /etc/fstab，保证机器重启挂载目录不丢失：</p>
<pre class="lang-java" data-nodeid="235980"><code data-language="java">$ sudo echo <span class="hljs-string">"/dev/vdb1 /var/lib/docker xfs defaults,pquota 0 0"</span> &gt;&gt; /etc/fstab
</code></pre>
<p data-nodeid="237418" class="">4.使用 mount 命令使得挂载目录生效：</p>
<pre class="lang-java" data-nodeid="237419"><code data-language="java">$ sudo mount -a
</code></pre>
<p data-nodeid="238845" class="">5.查看挂载信息：</p>
<pre class="lang-java" data-nodeid="238846"><code data-language="java">$ lsblk
NAME&nbsp; &nbsp;MAJ:MIN RM&nbsp; SIZE RO TYPE MOUNTPOINT
vda&nbsp; &nbsp; <span class="hljs-number">253</span>:<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> disk
`-vda1 <span class="hljs-number">253</span>:<span class="hljs-number">1</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> part /
vdb&nbsp; &nbsp; <span class="hljs-number">253</span>:<span class="hljs-number">16</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; <span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">0</span> disk
`-vdb1 <span class="hljs-number">253</span>:<span class="hljs-number">17</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">8</span>G&nbsp; <span class="hljs-number">0</span> part /<span class="hljs-keyword">var</span>/lib/docker
</code></pre>
<p data-nodeid="238847">可以看到此时  /var/lib/docker 目录已经被挂载到了 vdb1 这个磁盘分区上。我们使用 xfs_info 命令验证下 d_type 是否已经成功开启：</p>
<pre class="lang-java" data-nodeid="238848"><code data-language="java">$ xfs_info /<span class="hljs-keyword">var</span>/lib/docker | grep ftype
naming&nbsp; &nbsp;=version <span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; bsize=<span class="hljs-number">4096</span>&nbsp; &nbsp;ascii-ci=<span class="hljs-number">0</span> ftype=<span class="hljs-number">1</span>
</code></pre>
<p data-nodeid="239551">可以看到输出结果为  ftype=1，证明 d_type 已经被成功开启。</p>
<p data-nodeid="239552">准备好  /var/lib/docker 目录后，我们就可以配置 Docker 的文件驱动为 overlay2，并且启动 Docker 了。</p>

<h3 data-nodeid="238850">如何在 Docker 中配置 overlay2？</h3>
<p data-nodeid="238851">当你的系统满足上面的条件后，就可以配置你的 Docker 存储驱动为 overlay2 了，具体配置步骤如下。</p>
<p data-nodeid="240085" class="">1.停止已经运行的 Docker：</p>
<pre class="lang-java" data-nodeid="240086"><code data-language="java">$ sudo systemctl stop docker
</code></pre>
<p data-nodeid="240765" class="">2.备份 /var/lib/docker 目录：</p>
<pre class="lang-java" data-nodeid="240766"><code data-language="java">$ sudo cp -au /<span class="hljs-keyword">var</span>/lib/docker /<span class="hljs-keyword">var</span>/lib/docker.back
</code></pre>
<p data-nodeid="241436" class="">3.在 /etc/docker 目录下创建 daemon.json 文件，如果该文件已经存在，则修改配置为以下内容：</p>
<pre class="lang-json" data-nodeid="241437"><code data-language="json">{
  <span class="hljs-attr">"storage-driver"</span>: <span class="hljs-string">"overlay2"</span>,
  <span class="hljs-attr">"storage-opts"</span>: [
    <span class="hljs-string">"overlay2.size=20G"</span>,
    <span class="hljs-string">"overlay2.override_kernel_check=true"</span>
  ]
}
</code></pre>
<p data-nodeid="241438">其中 storage-driver 参数指定使用 overlay2 文件驱动，overlay2.size 参数表示限制每个容器根目录大小为 20G。限制每个容器的磁盘空间大小是通过 xfs 的 pquota 特性实现，overlay2.size 可以根据不同的生产环境来设置这个值的大小。我推荐你在生产环境中开启此参数，防止某个容器写入文件过大，导致整个 Docker 目录空间溢出。</p>
<p data-nodeid="242749" class="">4.启动 Docker：</p>
<pre class="lang-java" data-nodeid="242750"><code data-language="java">$ sudo systemctl start docker
</code></pre>
<p data-nodeid="244048" class="">5.检查配置是否生效：</p>
<pre class="lang-java" data-nodeid="244049"><code data-language="java">$ docker info
Client:
&nbsp;Debug Mode: <span class="hljs-keyword">false</span>
Server:
&nbsp;Containers: <span class="hljs-number">1</span>
&nbsp; Running: <span class="hljs-number">0</span>
&nbsp; Paused: <span class="hljs-number">0</span>
&nbsp; Stopped: <span class="hljs-number">1</span>
&nbsp;Images: <span class="hljs-number">1</span>
&nbsp;Server Version: <span class="hljs-number">19.03</span>.<span class="hljs-number">12</span>
&nbsp;Storage Driver: overlay2
&nbsp; Backing Filesystem: xfs
&nbsp; Supports d_type: <span class="hljs-keyword">true</span>
&nbsp; Native Overlay Diff: <span class="hljs-keyword">true</span>
&nbsp;Logging Driver: json-file
&nbsp;Cgroup Driver: cgroupfs
&nbsp;... 省略部分无用输出
</code></pre>
<p data-nodeid="244050">可以看到 Storage Driver 已经变为 overlay2，并且 d_type 也是 true。至此，你的 Docker 已经配置完成。下面我们看下 overlay2 是如何工作的。</p>
<h3 data-nodeid="244051">overlay2 工作原理</h3>
<h4 data-nodeid="244052">overlay2 是如何存储文件的？</h4>
<p data-nodeid="244053">overlay2 和 AUFS 类似，它将所有目录称之为层（layer），overlay2 的目录是镜像和容器分层的基础，而把这些层统一展现到同一的目录下的过程称为联合挂载（union mount）。overlay2 把目录的下一层叫作<code data-backticks="1" data-nodeid="244108">lowerdir</code>，上一层叫作<code data-backticks="1" data-nodeid="244110">upperdir</code>，联合挂载后的结果叫作<code data-backticks="1" data-nodeid="244112">merged</code>。</p>
<blockquote data-nodeid="244054">
<p data-nodeid="244055">overlay2 文件系统最多支持 128 个层数叠加，也就是说你的 Dockerfile 最多只能写 128 行，不过这在日常使用中足够了。</p>
</blockquote>
<p data-nodeid="244056">下面我们通过拉取一个 Ubuntu 操作系统的镜像来看下 overlay2 是如何存放镜像文件的。</p>
<p data-nodeid="244057">首先，我们通过以下命令拉取 Ubuntu 镜像：</p>
<pre class="lang-java" data-nodeid="244058"><code data-language="java">$ docker pull ubuntu:<span class="hljs-number">16.04</span>
<span class="hljs-number">16.04</span>: Pulling from library/ubuntu
<span class="hljs-number">8e097</span>b52bfb8: Pull complete
a613a9b4553c: Pull complete
acc000f01536: Pull complete
<span class="hljs-number">73</span>eef93b7466: Pull complete
Digest: sha256:<span class="hljs-number">3d</span>d44f7ca10f07f86add9d0dc611998a1641f501833692a2651c96defe8db940
Status: Downloaded newer image <span class="hljs-keyword">for</span> ubuntu:<span class="hljs-number">16.04</span>
docker.io/library/ubuntu:<span class="hljs-number">16.04</span>
</code></pre>
<p data-nodeid="244059">可以看到镜像一共被分为四层拉取，拉取完镜像后我们查看一下 overlay2 的目录：</p>
<pre class="lang-java" data-nodeid="244060"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/overlay2/
total <span class="hljs-number">0</span>
drwx------. <span class="hljs-number">3</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">47</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">01</span>946de89606800dac8530e3480b32be9d7c66b493a1cdf558df52d7a1476d4a
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">55</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">0849d</span>aa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">94222</span>a2fa3b2405cb00459285dd0d0ba7e6936d9b693ed18fbb0d08b93dc272f
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">9d</span>392cf38f245d37699bdd7672daaaa76a7d702083694fa8be380087bda5e396
brw-------. <span class="hljs-number">1</span> root root <span class="hljs-number">253</span>, <span class="hljs-number">17</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">14</span> backingFsBlockDev
drwx------. <span class="hljs-number">2</span> root root&nbsp; &nbsp; &nbsp;<span class="hljs-number">142</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> l
</code></pre>
<p data-nodeid="244061">可以看到 overlay2 目录下出现了四个镜像层目录和一个<code data-backticks="1" data-nodeid="244119">l</code>目录，我们首先来查看一下<code data-backticks="1" data-nodeid="244121">l</code>目录的内容：</p>
<pre class="lang-java" data-nodeid="244062"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/overlay2/l
total <span class="hljs-number">0</span>
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> FWGSYEA56RNMS53EUCKEQIKVLQ -&gt; ../<span class="hljs-number">9d</span>392cf38f245d37699bdd7672daaaa76a7d702083694fa8be380087bda5e396/diff
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> RNN2FM3YISKADNAZFRONVNWTIS -&gt; ../<span class="hljs-number">0849d</span>aa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/diff
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> SHAQ5GYA3UZLJJVEGXEZM34KEE -&gt; ../<span class="hljs-number">01</span>946de89606800dac8530e3480b32be9d7c66b493a1cdf558df52d7a1476d4a/diff
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> VQSNH735KNX4YK2TCMBAJRFTGT -&gt; ../<span class="hljs-number">94222</span>a2fa3b2405cb00459285dd0d0ba7e6936d9b693ed18fbb0d08b93dc272f/diff
</code></pre>
<p data-nodeid="244063">可以看到<code data-backticks="1" data-nodeid="244124">l</code>目录是一堆软连接，把一些较短的随机串软连到镜像层的 diff 文件夹下，这样做是为了避免达到<code data-backticks="1" data-nodeid="244126">mount</code>命令参数的长度限制。<br>
下面我们查看任意一个镜像层下的文件内容：</p>
<pre class="lang-java" data-nodeid="244064"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/overlay2/<span class="hljs-number">0849d</span>aa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/
total <span class="hljs-number">8</span>
drwxr-xr-x. <span class="hljs-number">3</span> root root <span class="hljs-number">17</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> diff
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">26</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> link
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">86</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> lower
drwx------. <span class="hljs-number">2</span> root root&nbsp; <span class="hljs-number">6</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> work
</code></pre>
<p data-nodeid="244065"><strong data-nodeid="244137">镜像层的 link 文件内容为该镜像层的短 ID，diff 文件夹为该镜像层的改动内容，lower 文件为该层的所有父层镜像的短 ID。</strong><br>
我们可以通过<code data-backticks="1" data-nodeid="244135">docker image inspect</code>命令来查看某个镜像的层级关系，例如我想查看刚刚下载的 Ubuntu 镜像之间的层级关系，可以使用以下命令：</p>
<pre class="lang-java" data-nodeid="244066"><code data-language="java">$ docker image inspect ubuntu:<span class="hljs-number">16.04</span>
...省略部分输出
<span class="hljs-string">"GraphDriver"</span>: {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Data"</span>: {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"LowerDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/9d392cf38f245d37699bdd7672daaaa76a7d702083694fa8be380087bda5e396/diff:/var/lib/docker/overlay2/94222a2fa3b2405cb00459285dd0d0ba7e6936d9b693ed18fbb0d08b93dc272f/diff:/var/lib/docker/overlay2/01946de89606800dac8530e3480b32be9d7c66b493a1cdf558df52d7a1476d4a/diff"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"MergedDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/0849daa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/merged"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"UpperDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/0849daa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/diff"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"WorkDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/0849daa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/work"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Name"</span>: <span class="hljs-string">"overlay2"</span>
&nbsp; &nbsp; &nbsp; &nbsp; },
...省略部分输出
</code></pre>
<p data-nodeid="244690">其中 MergedDir 代表当前镜像层在 overlay2 存储下的目录，LowerDir 代表当前镜像的父层关系，使用冒号分隔，冒号最后代表该镜像的最底层。</p>
<p data-nodeid="244691">下面我们将镜像运行起来成为容器：</p>

<pre class="lang-java" data-nodeid="244068"><code data-language="java">$ docker run --name=ubuntu -d ubuntu:<span class="hljs-number">16.04</span> sleep <span class="hljs-number">3600</span>
</code></pre>
<p data-nodeid="244069">我们使用<code data-backticks="1" data-nodeid="244142">docker inspect</code>命令来查看一下容器的工作目录：</p>
<pre class="lang-java" data-nodeid="244070"><code data-language="java">$ docker inspect&nbsp;ubuntu
...省略部分输出
&nbsp;<span class="hljs-string">"GraphDriver"</span>: {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Data"</span>: {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"LowerDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/4753c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2-init/diff:/var/lib/docker/overlay2/0849daa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb/diff:/var/lib/docker/overlay2/9d392cf38f245d37699bdd7672daaaa76a7d702083694fa8be380087bda5e396/diff:/var/lib/docker/overlay2/94222a2fa3b2405cb00459285dd0d0ba7e6936d9b693ed18fbb0d08b93dc272f/diff:/var/lib/docker/overlay2/01946de89606800dac8530e3480b32be9d7c66b493a1cdf558df52d7a1476d4a/diff"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"MergedDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/4753c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2/merged"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"UpperDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/4753c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2/diff"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"WorkDir"</span>: <span class="hljs-string">"/var/lib/docker/overlay2/4753c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2/work"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Name"</span>: <span class="hljs-string">"overlay2"</span>
&nbsp; &nbsp; &nbsp; &nbsp; },
...省略部分输出
</code></pre>
<p data-nodeid="245202" class=""><strong data-nodeid="245236">MergedDir 后面的内容即为容器层的工作目录，LowerDir 为容器所依赖的镜像层目录。</strong> 然后我们查看下 overlay2 目录下的内容：</p>
<pre class="lang-java" data-nodeid="245203"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/overlay2/
total <span class="hljs-number">0</span>
drwx------. <span class="hljs-number">3</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">47</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">01</span>946de89606800dac8530e3480b32be9d7c66b493a1cdf558df52d7a1476d4a
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> <span class="hljs-number">0849d</span>aa41598a333101f6a411755907d182a7fcef780c7f048f15d335b774deb
drwx------. <span class="hljs-number">5</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">69</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> <span class="hljs-number">4753</span>c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> <span class="hljs-number">4753</span>c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2-init
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">94222</span>a2fa3b2405cb00459285dd0d0ba7e6936d9b693ed18fbb0d08b93dc272f
drwx------. <span class="hljs-number">4</span> root root&nbsp; &nbsp; &nbsp; <span class="hljs-number">72</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">16</span> <span class="hljs-number">9d</span>392cf38f245d37699bdd7672daaaa76a7d702083694fa8be380087bda5e396
brw-------. <span class="hljs-number">1</span> root root <span class="hljs-number">253</span>, <span class="hljs-number">17</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">14</span> backingFsBlockDev
drwx------. <span class="hljs-number">2</span> root root&nbsp; &nbsp; &nbsp;<span class="hljs-number">210</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> l
</code></pre>
<p data-nodeid="245204">可以看到 overlay2 目录下增加了容器层相关的目录，我们再来查看一下容器层下的内容：</p>
<pre class="lang-java" data-nodeid="245205"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/overlay2/<span class="hljs-number">4753</span>c2aa5bdb20c97cddd6978ee3b1d07ef149e3cc2bbdbd4d11da60685fe9b2
total <span class="hljs-number">8</span>
drwxr-xr-x. <span class="hljs-number">2</span> root root&nbsp; &nbsp;<span class="hljs-number">6</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> diff
-rw-r--r--. <span class="hljs-number">1</span> root root&nbsp; <span class="hljs-number">26</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> link
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">144</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> lower
drwxr-xr-x. <span class="hljs-number">1</span> root root&nbsp; &nbsp;<span class="hljs-number">6</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> merged
drwx------. <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">18</span> Sep <span class="hljs-number">13</span> <span class="hljs-number">08</span>:<span class="hljs-number">47</span> work
</code></pre>
<p data-nodeid="245781">link 和 lower 文件与镜像层的功能一致，****link 文件内容为该容器层的短 ID，lower 文件为该层的所有父层镜像的短 ID 。<strong data-nodeid="245789">diff 目录为容器的读写层，容器内修改的文件都会在 diff 中出现，merged 目录为分层文件联合挂载后的结果，也是容器内的工作目录。</strong></p>
<p data-nodeid="245782">总体来说，overlay2 是这样储存文件的：<code data-backticks="1" data-nodeid="245791">overlay2</code>将镜像层和容器层都放在单独的目录，并且有唯一 ID，每一层仅存储发生变化的文件，最终使用联合挂载技术将容器层和镜像层的所有文件统一挂载到容器中，使得容器中看到完整的系统文件。</p>

<h4 data-nodeid="245207">overlay2 如何读取、修改文件？</h4>
<p data-nodeid="245208">overlay2 的工作过程中对文件的操作分为读取文件和修改文件。</p>
<p data-nodeid="245209"><strong data-nodeid="245254">读取文件</strong></p>
<p data-nodeid="245210">容器内进程读取文件分为以下三种情况。</p>
<ul data-nodeid="245211">
<li data-nodeid="245212">
<p data-nodeid="245213">文件在容器层中存在：当文件存在于容器层并且不存在于镜像层时，直接从容器层读取文件；</p>
</li>
<li data-nodeid="245214">
<p data-nodeid="245215">当文件在容器层中不存在：当容器中的进程需要读取某个文件时，如果容器层中不存在该文件，则从镜像层查找该文件，然后读取文件内容；</p>
</li>
<li data-nodeid="245216">
<p data-nodeid="245217">文件既存在于镜像层，又存在于容器层：当我们读取的文件既存在于镜像层，又存在于容器层时，将会从容器层读取该文件。</p>
</li>
</ul>
<p data-nodeid="245218"><strong data-nodeid="245262">修改文件或目录</strong></p>
<p data-nodeid="245219">overlay2 对文件的修改采用的是写时复制的工作机制，这种工作机制可以最大程度节省存储空间。具体的文件操作机制如下。</p>
<ul data-nodeid="245220">
<li data-nodeid="245221">
<p data-nodeid="245222">第一次修改文件：当我们第一次在容器中修改某个文件时，overlay2 会触发写时复制操作，overlay2 首先从镜像层复制文件到容器层，然后在容器层执行对应的文件修改操作。</p>
</li>
</ul>
<blockquote data-nodeid="245223">
<p data-nodeid="245224">overlay2 写时复制的操作将会复制整个文件，如果文件过大，将会大大降低文件系统的性能，因此当我们有大量文件需要被修改时，overlay2 可能会出现明显的延迟。好在，写时复制操作只在第一次修改文件时触发，对日常使用没有太大影响。</p>
</blockquote>
<ul data-nodeid="245225">
<li data-nodeid="245226">
<p data-nodeid="245227">删除文件或目录：当文件或目录被删除时，overlay2 并不会真正从镜像中删除它，因为镜像层是只读的，overlay2 会创建一个特殊的文件或目录，这种特殊的文件或目录会阻止容器的访问。</p>
</li>
</ul>
<h3 data-nodeid="245228">结语</h3>
<p data-nodeid="245229">overlay2 目前已经是 Docker 官方推荐的文件系统了，也是目前安装 Docker 时默认的文件系统，因为 overlay2 在生产环境中不仅有着较高的性能，它的稳定性也极其突出。但是 overlay2 的使用还是有一些限制条件的，例如要求 Docker 版本必须高于 17.06.02，内核版本必须高于 4.0 等。因此，在生产环境中，如果你的环境满足使用 overlay2 的条件，请尽量使用 overlay2 作为 Docker 的联合文件系统。</p>
<p data-nodeid="245230">那么你知道除了我介绍的这三种联合文件系统外，Docker 还可以使用哪些联合文件系统吗？ 思考后，可以把你的想法写在留言区。</p>
<p data-nodeid="245231">下一课时，我将带你进入 Docker 原理实践，自己动手使用 Golang 开发 Docker。</p>

---

### 精选评论

##### *策：
> 这个回复有点晚啦，现在docker还支持zfs文件系统

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，zfs 文件系统也是 docker 所支持的文件系统之一

##### *冲：
> 前置环境都没问题,问题是daemon.json配置完毕之后,重启docker服务起不来...配置文件也对照了好几遍.不知道什么情况Job for docker.service failed because the control process exited with error code. See "systemctl status docker.service" and "journalctl -xe" for details.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请根据提示使用  "systemctl status docker.service" 和 "journalctl -xe" 命令查看具体报错

##### 辉：
> 请问tmp下存储的是什么内容？/var/lib/docker/overlay2/XXX/merged/tmp/和/var/lib/docker/overlay2/XXX/diff/tmp/容器是正常运行的，这两个目录使用空间增长迅速是什么原因呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是在容器内 /tmp 目录下产生了大文件导致的

##### **生：
> 打卡

