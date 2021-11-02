<p data-nodeid="63757" class="">今天我们开始学习模块六：文件系统。学习文件系统的意义在于文件系统有很多设计思路可以迁移到实际的工作场景中，比如：</p>
<ul data-nodeid="63758">
<li data-nodeid="63759">
<p data-nodeid="63760">MySQL 的 binlog 和 Redis AOF 都像极了日志文件系统的设计；</p>
</li>
<li data-nodeid="63761">
<p data-nodeid="63762">B Tree 用于加速磁盘数据访问的设计，对于索引设计也有通用的意义。</p>
</li>
</ul>
<p data-nodeid="63763">特别是近年来分布式系统的普及，学习分布式文件系统，也是理解分布式架构最核心的一个环节。其实文件系统最精彩的还是虚拟文件系统的设计，比如 Linux 可以支持每个目录用不同的文件系统。这些文件看上去是一个个目录和文件，实际上可能是磁盘、内存、网络文件系统、远程磁盘、网卡、随机数产生器、输入输出设备等，这样虚拟文件系统就成了整合一切设备资源的平台。大量的操作都可以抽象成对文件的操作，程序的书写就会完整而统一，且扩展性强。</p>
<p data-nodeid="63764">这一讲，我会从 Linux 的目录结构和用途开始，带你认识 Linux 的文件系统。Linux 所有的文件都建立在虚拟文件系统（Virtual File System ，VFS）之上，如下图所示：</p>
<p data-nodeid="63765"><img src="https://s0.lgstatic.com/i/image2/M01/03/D3/Cip5yF_jAd-APzhvAADyJAEGLTc170.png" alt="Lark20201223-163616.png" data-nodeid="63844"></p>
<p data-nodeid="63766">当你访问一个目录或者文件，虽然用的是 Linux 标准的文件 API 对文件进行操作，但实际操作的可能是磁盘、内存、网络或者数据库等。<strong data-nodeid="63850">因此，Linux 上不同的目录可能是不同的磁盘，不同的文件可能是不同的设备</strong>。</p>
<h3 data-nodeid="63767">分区结构</h3>
<p data-nodeid="63768">在 Linux 中，<code data-backticks="1" data-nodeid="63853">/</code>是根目录。之前我们在“<strong data-nodeid="63863">08 讲</strong>”提到过，每个目录可以是不同的文件系统（不同的磁盘或者设备）。你可能会问我，<code data-backticks="1" data-nodeid="63859">/</code>是对应一个磁盘还是多个磁盘呢？在<code data-backticks="1" data-nodeid="63861">/</code>创建目录的时候，目录属于哪个磁盘呢？</p>
<p data-nodeid="63769"><img src="https://s0.lgstatic.com/i/image2/M01/03/D5/CgpVE1_jAeqAern4AAH5hspmQ0Y638.png" alt="Drawing 1.png" data-nodeid="63866"></p>
<p data-nodeid="63770">你可以用<code data-backticks="1" data-nodeid="63868">df -h</code>查看上面两个问题的答案，在上图中我的<code data-backticks="1" data-nodeid="63870">/</code>挂载到了<code data-backticks="1" data-nodeid="63872">/dev/sda5</code>上。如果你想要看到更多信息，可以使用<code data-backticks="1" data-nodeid="63874">df -T</code>，如下图所示：</p>
<p data-nodeid="63771"><img src="https://s0.lgstatic.com/i/image2/M01/03/D5/CgpVE1_jAfGAf6BqAAGJaAmhd0Q927.png" alt="Drawing 2.png" data-nodeid="63878"></p>
<p data-nodeid="63772"><code data-backticks="1" data-nodeid="63879">/</code>的文件系统类型是<code data-backticks="1" data-nodeid="63881">ext4</code>。这是一种常用的日志文件系统。关于日志文件系统，我会在“**30 讲”**为你介绍。然后你可能还会有一个疑问，<code data-backticks="1" data-nodeid="63889">/dev/sda5</code>究竟是一块磁盘还是别的什么？这个时候你可以用<code data-backticks="1" data-nodeid="63891">fdisk -l</code>查看，结果如下图：</p>
<p data-nodeid="63773"><img src="https://s0.lgstatic.com/i/image/M00/8B/FC/CgqCHl_jAf-AGBtKAANDnVrYDh0934.png" alt="Drawing 3.png" data-nodeid="63895"></p>
<p data-nodeid="63774">你可以看到我的 Linux 虚拟机上，有一块 30G 的硬盘（当然是虚拟的）。然后这块硬盘下有 3 个设备（Device）：/dev/sda1, /dev/sda2 和 /dev/sda5。在 Linux 中，数字 1~4 结尾的是主分区，通常一块磁盘最多只能有 4 个主分区用于系统启动。主分区之下，还可以再分成若干个逻辑分区，4 以上的数字都是逻辑分区。因此<code data-backticks="1" data-nodeid="63899">/dev/sda2</code>和<code data-backticks="1" data-nodeid="63901">/dev/sda5</code>是主分区包含逻辑分区的关系。</p>
<h3 data-nodeid="63775">挂载</h3>
<p data-nodeid="63776">分区结构最终需要最终挂载到目录上。上面例子中<code data-backticks="1" data-nodeid="63905">/dev/sda5</code>分区被挂载到了<code data-backticks="1" data-nodeid="63907">/</code>下。 这样在<code data-backticks="1" data-nodeid="63909">/</code>创建的文件都属于这个<code data-backticks="1" data-nodeid="63911">/dev/sda5</code>分区。 另外，<code data-backticks="1" data-nodeid="63913">/dev/sda5</code>采用<code data-backticks="1" data-nodeid="63915">ext4</code>文件系统。可见<strong data-nodeid="63921">不同的目录可以采用不同的文件系统</strong>。</p>
<p data-nodeid="63777">将一个文件系统映射到某个目录的过程叫作挂载（Mount）。当然这里的文件系统可以是某个分区、某个 USB 设备，也可以是某个读卡器等。你可以用<code data-backticks="1" data-nodeid="63923">mount -l</code>查看已经挂载的文件系统。</p>
<p data-nodeid="63778"><img src="https://s0.lgstatic.com/i/image2/M01/03/D3/Cip5yF_jAfeAIaUWAANFrmAEXQM991.png" alt="Drawing 4.png" data-nodeid="63927"></p>
<p data-nodeid="63779">上图中的<code data-backticks="1" data-nodeid="63929">sysfs``proc``devtmpfs``tmpfs``ext4</code>都是不同的文件系统，下面我们来说说它们的作用。</p>
<ul data-nodeid="63780">
<li data-nodeid="63781">
<p data-nodeid="63782"><code data-backticks="1" data-nodeid="63931">sysfs</code>让用户通过文件访问和设置设备驱动信息。</p>
</li>
<li data-nodeid="63783">
<p data-nodeid="63784"><code data-backticks="1" data-nodeid="63933">proc</code>是一个虚拟文件系统，让用户可以通过文件访问内核中的进程信息。</p>
</li>
<li data-nodeid="63785">
<p data-nodeid="63786"><code data-backticks="1" data-nodeid="63935">devtmpfs</code>在内存中创造设备文件节点。</p>
</li>
<li data-nodeid="63787">
<p data-nodeid="63788"><code data-backticks="1" data-nodeid="63937">tmpfs</code>用内存模拟磁盘文件。</p>
</li>
<li data-nodeid="63789">
<p data-nodeid="63790"><code data-backticks="1" data-nodeid="63939">ext4</code>是一个通常意义上我们认为的文件系统，也是管理磁盘上文件用的系统。</p>
</li>
</ul>
<p data-nodeid="63791">你可以看到挂载记录中不仅有文件系统类型，挂载的目录（on 后面部分），还有读写的权限等。你也可以用<code data-backticks="1" data-nodeid="63942">mount</code>指令挂载一个文件系统到某个目录，比如说：</p>
<pre class="lang-plain" data-nodeid="63792"><code data-language="plain">mount /dev/sda6 /abc
</code></pre>
<p data-nodeid="63793">上面这个命令将<code data-backticks="1" data-nodeid="63945">/dev/sda6</code>挂载到目录<code data-backticks="1" data-nodeid="63947">abc</code>。</p>
<h3 data-nodeid="63794">目录结构</h3>
<p data-nodeid="63795">因为 Linux 内文件系统较多，用途繁杂，Linux 对文件系统中的目录进行了一定的归类，如下图所示：</p>
<p data-nodeid="63796"><img src="https://s0.lgstatic.com/i/image/M00/8B/F1/Ciqc1F_jAhGADnWLAAFf1qd349k816.png" alt="Lark20201223-163621.png" data-nodeid="63953"></p>
<p data-nodeid="63797"><strong data-nodeid="63962">最顶层的目录称作根目录，</strong> 用<code data-backticks="1" data-nodeid="63958">/</code>表示。<code data-backticks="1" data-nodeid="63960">/</code>目录下用户可以再创建目录，但是有一些目录随着系统创建就已经存在，接下来我会和你一起讨论下它们的用途。</p>
<p data-nodeid="63798"><strong data-nodeid="63973">/bin（二进制</strong>）包含了许多所有用户都可以访问的可执行文件，如 ls, cp, cd 等。这里的大多数程序都是二进制格式的，因此称作<code data-backticks="1" data-nodeid="63967">bin</code>目录。<code data-backticks="1" data-nodeid="63969">bin</code>是一个命名习惯，比如说<code data-backticks="1" data-nodeid="63971">nginx</code>中的可执行文件会在 Nginx 安装目录的 bin 文件夹下面。</p>
<p data-nodeid="63799"><strong data-nodeid="63980">/dev（设备文件）</strong> 通常挂载在<code data-backticks="1" data-nodeid="63978">devtmpfs</code>文件系统上，里面存放的是设备文件节点。通常直接和内存进行映射，而不是存在物理磁盘上。</p>
<p data-nodeid="63800">值得一提的是其中有几个有趣的文件，它们是虚拟设备。</p>
<p data-nodeid="63801"><strong data-nodeid="63990">/dev/null</strong>是可以用来销毁任何输出的虚拟设备。你可以用<code data-backticks="1" data-nodeid="63986">&gt;</code>重定向符号将任何输出流重定向到<code data-backticks="1" data-nodeid="63988">/dev/null</code>来忽略输出的结果。</p>
<p data-nodeid="63802"><strong data-nodeid="63995">/dev/zero</strong>是一个产生数字 0 的虚拟设备。无论你对它进行多少次读取，都会读到 0。</p>
<p data-nodeid="63803"><strong data-nodeid="64000">/dev/ramdom</strong>是一个产生随机数的虚拟设备。读取这个文件中数据，你会得到一个随机数。你不停地读取这个文件，就会得到一个随机数的序列。</p>
<p data-nodeid="63804"><strong data-nodeid="64014">/etc（配置文件），</strong><code data-backticks="1" data-nodeid="64004">/etc</code>名字的含义是<code data-backticks="1" data-nodeid="64006">and so on……</code>，也就是“等等及其他”，Linux 用它来保管程序的配置。比如说<code data-backticks="1" data-nodeid="64008">mysql</code>通常会在<code data-backticks="1" data-nodeid="64010">/etc/mysql</code>下创建配置。再比如说<code data-backticks="1" data-nodeid="64012">/etc/passwd</code>是系统的用户配置，存储了用户信息。</p>
<p data-nodeid="63805"><strong data-nodeid="64025">/proc（进程和内核文件）</strong> 存储了执行中进程和内核的信息。比如你可以通过<code data-backticks="1" data-nodeid="64019">/proc/1122</code>目录找到和进程<code data-backticks="1" data-nodeid="64021">1122</code>关联的全部信息。还可以在<code data-backticks="1" data-nodeid="64023">/proc/cpuinfo</code>下找到和 CPU 相关的全部信息。</p>
<p data-nodeid="63806"><strong data-nodeid="64032">/sbin（系统二进制）</strong> 和<code data-backticks="1" data-nodeid="64030">/bin</code>类似，通常是系统启动必需的指令，也可以包括管理员才会使用的指令。</p>
<p data-nodeid="63807"><strong data-nodeid="64043">/tmp（临时文件）</strong> 用于存放应用的临时文件，通常用的是<code data-backticks="1" data-nodeid="64037">tmpfs</code>文件系统。因为<code data-backticks="1" data-nodeid="64039">tmpfs</code>是一个内存文件系统，系统重启的时候清除<code data-backticks="1" data-nodeid="64041">/tmp</code>文件，所以这个目录不能放应用和重要的数据。</p>
<p data-nodeid="63808"><strong data-nodeid="64054">/var （Variable data file,，可变数据文件）</strong> 用于存储运行时的数据，比如日志通常会存放在<code data-backticks="1" data-nodeid="64048">/var/log</code>目录下面。再比如应用的缓存文件、用户的登录行为等，都可以放到<code data-backticks="1" data-nodeid="64050">/var</code>目录下，<code data-backticks="1" data-nodeid="64052">/var</code>下的文件会长期保存。</p>
<p data-nodeid="63809"><strong data-nodeid="64059">/boot（启动）</strong> 目录下存放了 Linux 的内核文件和启动镜像，通常这个目录会写入磁盘最头部的分区，启动的时候需要加载目录内的文件。</p>
<p data-nodeid="63810"><strong data-nodeid="64064">/opt（Optional Software，可选软件）</strong> 通常会把第三方软件安装到这个目录。以后你安装软件的时候，可以考虑在这个目录下创建。</p>
<p data-nodeid="63811"><strong data-nodeid="64073">/root（root 用户家目录）</strong> 为了防止误操作，Linux 设计中 root 用户的家目录没有设计在<code data-backticks="1" data-nodeid="64069">/home/root</code>下，而是放到了<code data-backticks="1" data-nodeid="64071">/root</code>目录。</p>
<p data-nodeid="63812"><strong data-nodeid="64084">/home（家目录）</strong> 用于存放用户的个人数据，比如用户<code data-backticks="1" data-nodeid="64078">lagou</code>的个人数据会存放到<code data-backticks="1" data-nodeid="64080">/home/lagou</code>下面。并且通常在用户登录，或者执行<code data-backticks="1" data-nodeid="64082">cd</code>指令后，都会在家目录下工作。 用户通常会对自己的家目录拥有管理权限，而无法访问其他用户的家目录。</p>
<p data-nodeid="63813"><strong data-nodeid="64093">/media（媒体）</strong> 自动挂载的设备通常会出现在<code data-backticks="1" data-nodeid="64089">/media</code>目录下。比如你插入 U 盘，通常较新版本的 Linux 都会帮你自动完成挂载，也就是在<code data-backticks="1" data-nodeid="64091">/media</code>下创建一个目录代表 U 盘。</p>
<p data-nodeid="63814"><strong data-nodeid="64102">/mnt（Mount，挂载）</strong> 我们习惯把手动挂载的设备放到这个目录。比如你插入 U 盘后，如果 Linux 没有帮你完成自动挂载，可以用<code data-backticks="1" data-nodeid="64098">mount</code>命令手动将 U 盘内容挂载到<code data-backticks="1" data-nodeid="64100">/mnt</code>目录下。</p>
<p data-nodeid="63815"><strong data-nodeid="64111">/svr（Service Data,，服务数据）</strong> 通常用来存放服务数据，比如说你开发的网站资源文件（脚本、网页等）。不过现在很多团队的习惯发生了变化， 有的团队会把网站相关的资源放到<code data-backticks="1" data-nodeid="64107">/www</code>目录下，也有的团队会放到<code data-backticks="1" data-nodeid="64109">/data</code>下。总之，在存放资源的角度，还是比较灵活的。</p>
<p data-nodeid="63816"><strong data-nodeid="64116">/usr（Unix System Resource）</strong> 包含系统需要的资源文件，通常应用程序会把后来安装的可执行文件也放到这个目录下，比如说</p>
<ul data-nodeid="63817">
<li data-nodeid="63818">
<p data-nodeid="63819"><code data-backticks="1" data-nodeid="64117">vim</code>编辑器的可执行文件通常会在<code data-backticks="1" data-nodeid="64119">/usr/bin</code>目录下，区别于<code data-backticks="1" data-nodeid="64121">ls</code>会在<code data-backticks="1" data-nodeid="64123">/bin</code>目录下</p>
</li>
<li data-nodeid="63820">
<p data-nodeid="63821"><code data-backticks="1" data-nodeid="64125">/usr/sbin</code>中会包含有通常系统管理员才会使用的指令。</p>
</li>
<li data-nodeid="63822">
<p data-nodeid="63823"><code data-backticks="1" data-nodeid="64127">/usr/lib</code>目录中存放系统的库文件，比如一些重要的对象和动态链接库文件。</p>
</li>
<li data-nodeid="63824">
<p data-nodeid="63825"><code data-backticks="1" data-nodeid="64129">/usr/lib</code>目录下会有大量的<code data-backticks="1" data-nodeid="64131">.so</code>文件，这些叫作<code data-backticks="1" data-nodeid="64133">Shared Object</code>，类似<code data-backticks="1" data-nodeid="64135">windows</code>下的<code data-backticks="1" data-nodeid="64137">dll</code>文件。</p>
</li>
<li data-nodeid="63826">
<p data-nodeid="63827"><code data-backticks="1" data-nodeid="64139">/usr/share</code>目录下主要是文档，比如说 man 的文档都在<code data-backticks="1" data-nodeid="64141">/usr/share/man</code>下面。</p>
</li>
</ul>
<h3 data-nodeid="63828">总结</h3>
<p data-nodeid="63829">这一讲我们了解了 Linux 虚拟文件系统的设计，并且熟悉了 Linux 的目录结构。我曾经看到不少程序员把程序装到了<code data-backticks="1" data-nodeid="64145">/home</code>目录，也看到过不少程序员将数据放到了<code data-backticks="1" data-nodeid="64147">/root</code>目录。这样做并不会带来致命性问题，但是会给其他和你一起工作的同事带来困扰。</p>
<p data-nodeid="63830">今天我们讲到的这些规范是整个世界通用的，如果每个人都能遵循规范的原则，工作起来就会有很好的默契。登录一台<code data-backticks="1" data-nodeid="64150">linux</code>服务器，你可以通过目录结构快速熟悉。你可以查阅<code data-backticks="1" data-nodeid="64152">/etc</code>下的配置，看看<code data-backticks="1" data-nodeid="64154">/opt</code>下装了什么软件，这就是规范的好处。</p>
<p data-nodeid="63831"><strong data-nodeid="64160">那么通过这节课的学习，你现在可以尝试来回答本节标题中的试题目：Linux下各个目录有什么作用了吗</strong>？</p>
<p data-nodeid="63832">【<strong data-nodeid="64166">解析</strong>】通常面试官会挑选其中一部分对你进行抽查，如果你快要面试了，再 Review 一下本讲的内容吧。</p>
<h3 data-nodeid="63833">思考题</h3>
<p data-nodeid="63834"><strong data-nodeid="64172">最后我再给你出一道需要查资料的思考题：socket 文件都存在哪里</strong>？</p>
<p data-nodeid="65427" class="te-preview-highlight">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### **春：
> socket 是可变数据文件，默认应该是/var/lib/socket

##### yoe：
> /proc/fd

##### *炳：
> /var/run

##### *碹：
> 套接字对象好像跟进程相关的，应该是再proc下

##### **棋：
> 我查的在 /run/systemd/journal/socket😅

##### **队长：
> 只知道使用socket后会产生对应的fd，是放在proc下面的，至于socket库，还真没研究过

