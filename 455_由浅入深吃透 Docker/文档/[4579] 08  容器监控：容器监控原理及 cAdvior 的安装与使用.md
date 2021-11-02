<p data-nodeid="873" class="">生产环境中监控容器的运行状况十分重要，通过监控我们可以随时掌握容器的运行状态，做到线上隐患和问题早发现，早解决。所以今天我就和你分享关于容器监控的知识（原理及工具 cAdvisor）。</p>
<p data-nodeid="874">虽然传统的物理机和虚拟机监控已经有了比较成熟的监控方案，但是容器的监控面临着更大的挑战，因为容器的行为和本质与传统的虚拟机是不一样的，总的来说，容器具有以下特性：</p>
<ul data-nodeid="875">
<li data-nodeid="876">
<p data-nodeid="877">容器是短期存活的，并且可以动态调度；</p>
</li>
<li data-nodeid="878">
<p data-nodeid="879">容器的本质是进程，而不是一个完整操作系统；</p>
</li>
<li data-nodeid="880">
<p data-nodeid="881">由于容器非常轻量，容器的创建和销毁也会比传统虚拟机更加频繁。</p>
</li>
</ul>
<p data-nodeid="882">Docker 容器的监控方案有很多，除了 Docker 自带的<code data-backticks="1" data-nodeid="965">docker stats</code>命令，还有很多开源的解决方案，例如 sysdig、cAdvisor、Prometheus 等，都是非常优秀的监控工具。</p>
<p data-nodeid="883">下面我们首先来看下，不借助任何外部工具，如何用 Docker 自带的<code data-backticks="1" data-nodeid="968">docker stats</code>命令实现容器的监控。</p>
<h3 data-nodeid="884">使用 docker stats 命令</h3>
<p data-nodeid="885">使用 Docker 自带的<code data-backticks="1" data-nodeid="972">docker stats</code>命令可以很方便地看到主机上所有容器的 CPU、内存、网络 IO、磁盘 IO、PID 等资源的使用情况。下面我们可以具体操作看看。</p>
<p data-nodeid="886">首先在主机上使用以下命令启动一个资源限制为 1 核 2G 的 nginx 容器：</p>
<pre class="lang-java" data-nodeid="887"><code data-language="java">$ docker run --cpus=<span class="hljs-number">1</span> -m=<span class="hljs-number">2</span>g --name=nginx&nbsp; -d nginx
</code></pre>
<p data-nodeid="888">容器启动后，可以使用<code data-backticks="1" data-nodeid="976">docker stats</code>命令查看容器的资源使用状态:</p>
<pre class="lang-java" data-nodeid="889"><code data-language="java">$ docker stats&nbsp;nginx
</code></pre>
<p data-nodeid="890">通过<code data-backticks="1" data-nodeid="979">docker stats</code>命令可以看到容器的运行状态如下：</p>
<pre class="lang-java" data-nodeid="891"><code data-language="java">CONTAINER&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;CPU %&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MEM USAGE / LIMIT&nbsp; &nbsp;MEM %&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;NET I/O&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;BLOCK I/O&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;PIDS
f742a467b6d8&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0.00</span>%&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">1.387</span> MiB / <span class="hljs-number">2</span> GiB&nbsp; &nbsp;<span class="hljs-number">0.07</span>%&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">656</span> B / <span class="hljs-number">656</span> B&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span> B / <span class="hljs-number">9.22</span> kB&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>
</code></pre>
<p data-nodeid="892">从容器的运行状态可以看出，<code data-backticks="1" data-nodeid="982">docker stats</code>命令确实可以获取并显示 Docker 容器运行状态。但是它的缺点也很明显，因为它只能获取本机数据，无法查看历史监控数据，没有可视化展示面板。</p>
<p data-nodeid="893">因此，生产环境中我们通常使用另一种容器监控解决方案 cAdvisor。</p>
<h3 data-nodeid="894">cAdvisor</h3>
<p data-nodeid="895">cAdvisor 是谷歌开源的一款通用的容器监控解决方案。cAdvisor 不仅可以采集机器上所有运行的容器信息，还提供了基础的查询界面和 HTTP 接口，更方便与外部系统结合。所以，cAdvisor很快成了容器指标监控最常用组件，并且 Kubernetes 也集成了 cAdvisor 作为容器监控指标的默认工具。</p>
<h4 data-nodeid="896">cAdvisor 的安装与使用</h4>
<p data-nodeid="897">下面我们以 cAdvisor 0.37.0 版本为例，演示一下 cAdvisor 的安装与使用。</p>
<p data-nodeid="898">cAdvisor 官方提供了 Docker 镜像，我们只需要拉取镜像并且启动镜像即可。</p>
<blockquote data-nodeid="899">
<p data-nodeid="900">由于 cAdvisor 镜像存放在谷歌的 gcr.io 镜像仓库中，国内无法访问到。这里我把打好的镜像放在了 Docker Hub。你可以使用 docker pull lagoudocker/cadvisor:v0.37.0 命令从 Docker Hub 拉取。</p>
</blockquote>
<p data-nodeid="901">首先使用以下命令启动 cAdvisor：</p>
<pre class="lang-java" data-nodeid="902"><code data-language="java">$ docker run \
  --volume=/:/rootfs:ro \
  --volume=/<span class="hljs-keyword">var</span>/run:/<span class="hljs-keyword">var</span>/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/<span class="hljs-keyword">var</span>/lib/docker/:/<span class="hljs-keyword">var</span>/lib/docker:ro \
  --volume=/dev/disk/:/dev/disk:ro \
  --publish=<span class="hljs-number">8080</span>:<span class="hljs-number">8080</span> \
  --detach=<span class="hljs-keyword">true</span> \
  --name=cadvisor \
  --privileged \
  --device=/dev/kmsg \
  lagoudocker/cadvisor:v0<span class="hljs-number">.37</span><span class="hljs-number">.0</span>
</code></pre>
<p data-nodeid="903">此时，cAdvisor 已经成功启动，我们可以通过访问 <a href="http://localhost:8080" data-nodeid="995">http://localhost:8080</a> 访问到 cAdvisor 的 Web 界面。</p>
<p data-nodeid="904"><img src="https://s0.lgstatic.com/i/image/M00/56/18/Ciqc1F9rCXSAQEwLAADKlh0at8o307.png" alt="Drawing 0.png" data-nodeid="999"></p>
<div data-nodeid="905"><p style="text-align:center">图1 cAdvisor 首页</p></div>
<p data-nodeid="906">cAdvisor 不仅可以监控容器的资源使用情况，还可以监控主机的资源使用情况。下面我们就先看下它是如何查看主机资源使用情况的。</p>
<h4 data-nodeid="907">使用 cAdvisor 查看主机监控</h4>
<p data-nodeid="908">访问 <a href="http://localhost:8080/containers/" data-nodeid="1005">http://localhost:8080/containers/</a> 地址，在首页可以看到主机的资源使用情况，包含 CPU、内存、文件系统、网络等资源，如下图所示。</p>
<p data-nodeid="909"><img src="https://s0.lgstatic.com/i/image/M00/56/23/CgqCHl9rCX2ANrtaAADIGkeKKPc100.png" alt="Drawing 1.png" data-nodeid="1009"></p>
<div data-nodeid="910"><p style="text-align:center">图2 主机 CPU 使用情况</p></div>
<h4 data-nodeid="911">使用 cAdvisor 查看容器监控</h4>
<p data-nodeid="912">如果你想要查看主机上运行的容器资源使用情况，可以访问 <a href="http://localhost:8080/docker/" data-nodeid="1014">http://localhost:8080/docker/</a>，这个页面会列出 Docker 的基本信息和运行的容器情况，如下图所示。</p>
<p data-nodeid="913"><img src="https://s0.lgstatic.com/i/image/M00/56/18/Ciqc1F9rCZyAN8hYAAGAOL1FGcg401.png" alt="Drawing 2.png" data-nodeid="1018"></p>
<div data-nodeid="914"><p style="text-align:center">图3 Docker 容器</p></div>
<p data-nodeid="915">在上图中的 Subcontainers 下会列出当前主机上运行的所有容器，点击其中一个容器即可查看该容器的详细运行状态，如下图所示。</p>
<p data-nodeid="916"><img src="https://s0.lgstatic.com/i/image/M00/56/23/CgqCHl9rCaWAVSLVAAGGy2lTMqY130.png" alt="Drawing 3.png" data-nodeid="1022"></p>
<div data-nodeid="917"><p style="text-align:center">图4 容器监控状态</p></div>
<p data-nodeid="918">总体来说，使用 cAdvisor 监控容器具有以下特点：</p>
<ul data-nodeid="919">
<li data-nodeid="920">
<p data-nodeid="921">可以同时采集物理机和容器的状态；</p>
</li>
<li data-nodeid="922">
<p data-nodeid="923">可以展示监控历史数据。</p>
</li>
</ul>
<p data-nodeid="924">了解 Docker 的监控工具，你是否想问，这些监控数据是怎么来的呢？下面我就带你了解一下容器监控的原理。</p>
<h3 data-nodeid="925">监控原理</h3>
<p data-nodeid="926">我们知道 Docker 是基于 Namespace、Cgroups 和联合文件系统实现的。其中 Cgroups 不仅可以用于容器资源的限制，还可以提供容器的资源使用率。无论何种监控方案的实现，底层数据都来源于 Cgroups。</p>
<p data-nodeid="927">Cgroups 的工作目录为<code data-backticks="1" data-nodeid="1030">/sys/fs/cgroup</code>，<code data-backticks="1" data-nodeid="1032">/sys/fs/cgroup</code>目录下包含了 Cgroups 的所有内容。Cgroups包含很多子系统，可以用来对不同的资源进行限制。例如对CPU、内存、PID、磁盘 IO等资源进行限制和监控。</p>
<p data-nodeid="928">为了更详细的了解 Cgroups 的子系统，我们通过 ls -l 命令查看<code data-backticks="1" data-nodeid="1035">/sys/fs/cgroup</code>文件夹，可以看到很多目录：</p>
<pre class="lang-java" data-nodeid="929"><code data-language="java">$ sudo ls -l /sys/fs/cgroup/
total <span class="hljs-number">0</span>
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> blkio
lrwxrwxrwx <span class="hljs-number">1</span> root root <span class="hljs-number">11</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> cpu -&gt; cpu,cpuacct
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> cpu,cpuacct
lrwxrwxrwx <span class="hljs-number">1</span> root root <span class="hljs-number">11</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> cpuacct -&gt; cpu,cpuacct
dr-xr-xr-x <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> cpuset
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> devices
dr-xr-xr-x <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> freezer
dr-xr-xr-x <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> hugetlb
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> memory
lrwxrwxrwx <span class="hljs-number">1</span> root root <span class="hljs-number">16</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> net_cls -&gt; net_cls,net_prio
dr-xr-xr-x <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> net_cls,net_prio
lrwxrwxrwx <span class="hljs-number">1</span> root root <span class="hljs-number">16</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> net_prio -&gt; net_cls,net_prio
dr-xr-xr-x <span class="hljs-number">3</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> perf_event
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> pids
dr-xr-xr-x <span class="hljs-number">5</span> root root&nbsp; <span class="hljs-number">0</span> Jul&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">19</span>:<span class="hljs-number">32</span> systemd
</code></pre>
<p data-nodeid="930">这些目录代表了 Cgroups 的子系统，Docker 会在每一个 Cgroups 子系统下创建 docker 文件夹。这里如果你对 Cgroups 子系统不了解的话，不要着急，后续我会在第 10 课时对 Cgroups 子系统做详细讲解，这里你只需要明白容器监控数据来源于 Cgroups 即可。</p>
<h4 data-nodeid="931">监控系统是如何获取容器的内存限制的？</h4>
<p data-nodeid="932">下面我们以 memory 子系统（memory 子系统是Cgroups 众多子系统的一个，主要用来限制内存使用，Cgroups 会在第十课时详细讲解）为例，讲解一下监控组件是如何获取到容器的资源限制和使用状态的（即容器的内存限制）。</p>
<p data-nodeid="933">我们首先在主机上使用以下命令启动一个资源限制为 1 核 2G 的 nginx 容器：</p>
<pre class="lang-shell" data-nodeid="934"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run --name=nginx --cpus=1 -m=2g --name=nginx&nbsp; -d nginx</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 这里输出的是容器 ID</span></span>
51041a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
</code></pre>
<blockquote data-nodeid="935">
<p data-nodeid="936">注意：如果你已经创建过名称为 nginx 的容器，请先使用 docker&nbsp; rm -f nginx 命令删除已经存在的 nginx 容器。</p>
</blockquote>
<p data-nodeid="937">容器启动后，我们通过命令行的输出可以得到容器的 ID，同时 Docker 会在<code data-backticks="1" data-nodeid="1043">/sys/fs/cgroup/memory/docker</code>目录下以容器 ID 为名称创建对应的文件夹。</p>
<p data-nodeid="938">下面我们查看一下<code data-backticks="1" data-nodeid="1046">/sys/fs/cgroup/memory/docker</code>目录下的文件：</p>
<pre class="lang-java" data-nodeid="939"><code data-language="java">$ sudo ls -l /sys/fs/cgroup/memory/docker
total <span class="hljs-number">0</span>
drwxr-xr-x <span class="hljs-number">2</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> <span class="hljs-number">51041</span>a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> cgroup.clone_children
--w--w--w- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> cgroup.event_control
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> cgroup.procs
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.failcnt
--w------- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.force_empty
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.slabinfo
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.tcp.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.tcp.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.tcp.usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.kmem.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.max_usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.memsw.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.memsw.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.memsw.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.memsw.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.move_charge_at_immigrate
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.numa_stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.oom_control
---------- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.pressure_level
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.soft_limit_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.swappiness
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> memory.use_hierarchy
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> notify_on_release
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">14</span>:<span class="hljs-number">57</span> tasks
</code></pre>
<p data-nodeid="940">可以看到 Docker 已经创建了以容器 ID 为名称的目录，我们再使用 ls 命令查看一下该目录的内容：</p>
<pre class="lang-java" data-nodeid="941"><code data-language="java">$ sudo ls -l /sys/fs/cgroup/memory/docker/<span class="hljs-number">51041</span>a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1
total <span class="hljs-number">0</span>
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> cgroup.clone_children
--w--w--w- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">13</span> cgroup.event_control
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> cgroup.procs
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.failcnt
--w------- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.force_empty
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.kmem.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.slabinfo
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.tcp.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.tcp.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.tcp.usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.kmem.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.max_usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.memsw.failcnt
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.memsw.limit_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.memsw.max_usage_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.memsw.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.move_charge_at_immigrate
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.numa_stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">13</span> memory.oom_control
---------- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.pressure_level
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.soft_limit_in_bytes
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.stat
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.swappiness
-r--r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span> memory.usage_in_bytes
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> memory.use_hierarchy
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> notify_on_release
-rw-r--r-- <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">2</span> <span class="hljs-number">15</span>:<span class="hljs-number">21</span> tasks
</code></pre>
<p data-nodeid="942">由上可以看到，容器 ID 的目录下有很多文件，其中 memory.limit_in_bytes 文件代表该容器内存限制大小，单位为 byte，我们使用 cat 命令（cat 命令可以查看文件内容）查看一下文件内容：</p>
<pre class="lang-java" data-nodeid="943"><code data-language="java">$ sudo cat /sys/fs/cgroup/memory/docker/<span class="hljs-number">51041</span>a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1/memory.limit_in_bytes
<span class="hljs-number">2147483648</span>
</code></pre>
<p data-nodeid="944">这里可以看到memory.limit_in_bytes 的值为2147483648，转换单位后正好为 2G，符合我们启动容器时的内存限制 2G。</p>
<p data-nodeid="945">通过 memory 子系统的例子，我们可以知道<strong data-nodeid="1068">监控组件通过读取 memory.limit_in_bytes 文件即可获取到容器内存的限制值</strong>。了解完容器的内存限制我们来了解一下容器的内存使用情况。</p>
<h4 data-nodeid="946">监控系统是如何获取容器的内存使用状态的？</h4>
<p data-nodeid="947">内存使用情况存放在 memory.usage_in_bytes 文件里，同样我们也使用 cat 命令查看一下文件内容:</p>
<pre class="lang-java te-preview-highlight" data-nodeid="8380"><code data-language="java">$ sudo cat /sys/fs/cgroup/memory/docker/<span class="hljs-number">51041</span>a74070e9260e82876974762b8c61c5ed0a51832d74fba6711175f89ede1/memory.usage_in_bytes
<span class="hljs-number">4259840</span>
</code></pre>


















<p data-nodeid="949">可以看到当前内存的使用大小为 4259840 byte，约为 4 M。了解了内存的监控，下面我们来了解下网络的监控数据来源。</p>
<p data-nodeid="950">网络的监控数据来源是从 /proc/{PID}/net/dev 目录下读取的，其中 PID 为容器在主机上的进程 ID。下面我们首先使用 docker inspect 命令查看一下上面启动的 nginx 容器的 PID，命令如下：</p>
<pre class="lang-java" data-nodeid="951"><code data-language="java">$ docker&nbsp;inspect nginx |grep Pid
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Pid"</span>: <span class="hljs-number">27348</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"PidMode"</span>: <span class="hljs-string">""</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"PidsLimit"</span>: <span class="hljs-number">0</span>,
</code></pre>
<p data-nodeid="952">可以看到容器的 PID 为 27348，使用 cat 命令查看一下 /proc/27348/net/dev 的内容：</p>
<pre class="lang-java" data-nodeid="953"><code data-language="java">$ sudo cat /proc/<span class="hljs-number">27348</span>/net/dev
Inter-|&nbsp; &nbsp;Receive&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; |&nbsp; Transmit
&nbsp;face |bytes&nbsp; &nbsp; packets errs drop fifo frame compressed multicast|bytes&nbsp; &nbsp; packets errs drop fifo colls carrier compressed
&nbsp; &nbsp; lo:&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>
&nbsp; eth0:&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="954">/proc/27348/net/dev 文件记录了该容器里每一个网卡的流量接收和发送情况，以及错误数、丢包数等信息。可见容器的网络监控数据都是定时从这里读取并展示的。</p>
<p data-nodeid="955">总结一下，<strong data-nodeid="1083">容器的监控原理其实就是定时读取 Linux 主机上相关的文件并展示给用户。</strong></p>
<h3 data-nodeid="956">结语</h3>
<p data-nodeid="957">到此，相信你已经可以使用 docker stats 和 cAdvisor 监控并查看容器的状态了；也可以自己启动一个 cAdvisor 容器来监控主机和主机上的容器，并对监控系统的原理有了较深的了解。</p>
<p data-nodeid="958" class="">试想下，cAdvisor 虽然可以临时存储一段历史监控数据，并且提供了一个简版的监控面板，在大规模的容器集群中，cAdvisor 有什么明显的不足吗？思考后，把你的想法写在留言区。</p>

---

### 精选评论

##### *李：
> k8s后面使用metrics server代替cADvisor了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; cAdvisor 是提供底层数据的，metrics-server 底层数据来源是 cAdvisor

##### **鸣：
> 我在mac上安装了cAdvisor能够监控数据，但是mac上并没有/proc 和/sys目录，请问mac下cAdvisor监控的什么文件

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Mac 上的 Docker 实际上是在虚拟机里运行的，所以读取的是虚拟机里的文件。

##### *帆：
> cAdvisor只能监控某一台主机及其上面的容器，虽然有可视化和存储，但是大规模情况下不集中，无聚合

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，一般都是 cadvisor 结合 prometheus 一起使用

##### **远：
> docker 运行 CAdvisor 日志信息，求教">W0928 06:55:43.2692021 nvidia.go:61] NVIDIA GPU metrics will not be available: no NVIDIA devices found

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这这是一个警告信息哈，不会影响容器的运行

##### *欢：
> 这个跟promethues相比，有什么区别吗？感觉现在好多公司用的都是promethues

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; cAdvisor 是提供监控数据的，Prometheus 是负责采集的数据的，这两个作用是不一样的，生产集群中一般都是 cAdvisor 配合 Prometheus 一起使用

##### **辉：
> mac系统，老师您说监控数据是从虚拟机读取的，那怎么像文章中说的一样，验证下从虚拟机读取呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Docker 自带的 VM 无法进入到虚拟机,请在 Linux 下执行相关操作

##### **4118：
> 老师牛逼

##### **升：
> 打卡

##### **杰：
> grafana +promethues

##### **生：
> 最新版本的的 docker 的/sys/fs/cgroup/memory/system.slice/docker-容器id.scope如：/sys/fs/cgroup/memory/system.slice/docker-57029507a73468785dd90ae9e727c4bf8318f5b237528a36500cd2cd0fa805a4.scop

##### **旺：
> panic: cannot statfs cgroup root

