<h3 data-nodeid="85941" class="">Hadoop 日常运维问题及其解决方法</h3>





<h4 data-nodeid="89277" class="">1.如何下线一个 datanode 节点?</h4>





<p data-nodeid="79932">当一个 datanode 节点所在的服务器故障或者将要退役时，你需要在 Hadoop 中下线这个节点，下线一个 datanode 节点的过程如下。</p>
<p data-nodeid="79933"><strong data-nodeid="80092">（1）修改 hdfs-site.xml 文件</strong></p>
<p data-nodeid="79934">如下选项，找到 namenode 节点配置文件 /etc/hadoop/conf/hdfs-site.xml：</p>
<pre class="lang-yaml" data-nodeid="99568"><code data-language="yaml"><span class="hljs-string">&lt;property&gt;</span>     
<span class="hljs-string">&lt;name&gt;dfs.hosts.exclude&lt;/name&gt;</span>      
<span class="hljs-string">&lt;value&gt;/etc/hadoop/conf/hosts-exclude&lt;/value&gt;</span>  
<span class="hljs-string">&lt;/property&gt;</span>  
</code></pre>














<p data-nodeid="79936"><strong data-nodeid="80097">（2）修改 hosts-exclude 文件</strong></p>
<p data-nodeid="79937">执行如下操作，在 hosts-exclude 中添加需要下线的 datanode 主机名：</p>
<pre class="lang-java" data-nodeid="102508"><code data-language="java">vi /etc/hadoop/conf/hosts-exclude  
<span class="hljs-number">172.16</span>.<span class="hljs-number">213.188</span>  
</code></pre>




<p data-nodeid="79939"><strong data-nodeid="80102">（3）刷新配置</strong></p>
<p data-nodeid="79940">在 namenode 上以 hadoop 用户执行下面命令，刷新 hadoop 配置：</p>
<pre class="lang-java" data-nodeid="103243"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> ~]$hdfs dfsadmin -refreshNodes 
</code></pre>

<p data-nodeid="79942"><strong data-nodeid="80107">（4）检查是否完成下线</strong></p>
<p data-nodeid="79943">执行如下命令，检查下线是否完成：</p>
<pre class="lang-java" data-nodeid="103978"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> ~]$hdfs dfsadmin -report 
</code></pre>

<p data-nodeid="104713" class="">也可以通过 NameNode 的 50070 端口访问 Web 界面，查看 HDFS 状态，<strong data-nodeid="104719">需要重点关注退役的节点数，以及复制的块数和进度。</strong></p>
<h4 data-nodeid="108035" class="">2.某个 datanode 节点磁盘坏掉怎么办？</h4>







<p data-nodeid="79947">如果某个 datanode 节点的磁盘出现故障，那么该节点将不能进行写入操作，并导致 datanode 进程退出，针对这个问题，你可以如下解决：</p>
<ul data-nodeid="79948">
<li data-nodeid="79949">
<p data-nodeid="79950">首先，在故障节点上查看 /etc/hadoop/conf/hdfs-site.xml 文件中对应的 dfs.datanode.data.dir 参数设置，去掉故障磁盘对应的目录挂载点；</p>
</li>
<li data-nodeid="79951">
<p data-nodeid="79952">然后，在故障节点上查看 /etc/hadoop/conf/yarn-site.xml 文件中对应的 yarn.nodemanager.local-dirs 参数设置，去掉故障磁盘对应的目录挂载点；</p>
</li>
<li data-nodeid="79953">
<p data-nodeid="79954">最后，重启该节点的 DataNode 服务和 NodeManager 服务即可。</p>
</li>
</ul>
<h4 data-nodeid="111317" class="">3.Hadoop 进入安全模式怎么办?</h4>





<p data-nodeid="79956">Hadoop 刚启动时，由于各个服务的验证和启动还未完成，此时 Hadoop 会进入安全模式，这时文件系统的内容不允许修改，也不允许删除，这种状态会一直持续，直到安全模式结束为止。</p>
<p data-nodeid="112770" class="">而这个<strong data-nodeid="112776">安全模式</strong>主要是为了系统启动时，能够对各个 DataNode 数据块的有效性进行检查，并根据策略对部分数据块进行必要的复制或者删除。</p>


<p data-nodeid="79958">如果 Hadoop 的启动和验证都正常，那么只需等待一会儿，Hadoop 便将自动结束安全模式。</p>
<p data-nodeid="79959">当然，执行如下命令，也可以手动结束安全模式：</p>
<pre class="lang-java" data-nodeid="113499"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span>  conf]$ hdfs dfsadmin -safemode leave 
</code></pre>

<h4 data-nodeid="116753" class="">4.NodeManager 出现 Java heap space 错误怎么办？</h4>





<p data-nodeid="79962">这种错误，一般是 JVM 内存不够导致的，所以你需要修改所有 DataNode 和 NodeManager 的 JVM 内存大小，至于设置具体多大的内存，需要根据服务器的实际环境而定。</p>
<p data-nodeid="79963">如果设置的 JVM 值已经很大，但还是出现该问题，则需要查看 NodeManager 运行日志，具体是什么原因导致的，需要具体问题具体分析，当然，最直接的方法就是重启此节点的 NodeManager 服务。</p>
<h4 data-nodeid="119981" class="">5.DataNode 节点出现 Too many fetch-failures 错误的原因是什么？</h4>





<p data-nodeid="79965">出现这个问题的原因主要是，DataNode 节点间的连通性不够通畅，或者网络环境不太稳定。</p>
<p data-nodeid="79966">你可以从如下方面查找原因，便基本能判断问题所在：</p>
<ul data-nodeid="79967">
<li data-nodeid="79968">
<p data-nodeid="79969">检查 DataNode 节点和 NameNode 节点之间的网络延时；</p>
</li>
<li data-nodeid="79970">
<p data-nodeid="79971">通过 Nslookup 命令测试 DNS 解析主机名情况；</p>
</li>
<li data-nodeid="79972">
<p data-nodeid="79973">检查 /etc/hosts 和对应的主机名信息；</p>
</li>
<li data-nodeid="79974">
<p data-nodeid="79975">检查 NameNode 到 DataNode 节点的 SSH 单向信任情况。</p>
</li>
</ul>
<h4 data-nodeid="123182" class="">6.出现 No route to host 怎么办？</h4>





<p data-nodeid="79977">这个问题一般会在 DataNode 连接不上 NameNode，从而导致 DataNode 无法启动的情况下发生，问题发生时可在 DataNode 日志中看到如下类似信息：</p>
<pre class="lang-dart" data-nodeid="128823"><code data-language="dart">ERROR org.apache.hadoop.hdfs.server.datanode.DataNode: java.io.IOException: Call to ... failed <span class="hljs-keyword">on</span> local exception: java.net.NoRouteToHostException: No route to host 
</code></pre>








<p data-nodeid="79979">这个问题，一般是本机防火墙、本机网络，或系统的 selinux 导致的，所以你可以关闭本机防火墙或者 selinux，然后检查本机与 NameNode 之间的连通性，从而你便能判断出问题症结所在。</p>
<h4 data-nodeid="131996" class="">7.如何新增一个 DataNode 节点到 Hadoop 集群？</h4>





<p data-nodeid="79981">当集群资源不够时，需要新增几台机器加入集群，这是 Hadoop 运维最常见的处理方式之一。那么如何将新增的服务器加入 Hadoop 集群呢，主要有以下步骤。</p>
<p data-nodeid="79982"><strong data-nodeid="80165">（1）新节点部署 Hadoop 环境</strong></p>
<p data-nodeid="79983">新增节点在系统安装完成后，要进行一系列的操作，比如系统基本优化设置、Hadoop 环境的部署和安装、JDK 的安装等，这些基础工作都需要你事先完成。</p>
<p data-nodeid="79984"><strong data-nodeid="80170">（2）修改 hdfs-site.xml 文件</strong></p>
<p data-nodeid="79985">在 NameNode 查看 /etc/hadoop/conf/hdfs-site.xml 文件，找到如下内容：</p>
<pre class="lang-yaml" data-nodeid="143880"><code data-language="yaml"><span class="hljs-string">&lt;property&gt;</span> 
  <span class="hljs-string">&lt;name&gt;dfs.hosts&lt;/name&gt;</span> 
  <span class="hljs-string">&lt;value&gt;/etc/hadoop/conf/hosts&lt;/value&gt;</span> 
<span class="hljs-string">&lt;/property&gt;</span> 
</code></pre>

















<p data-nodeid="79987"><strong data-nodeid="80175">（3）修改 hosts 文件</strong></p>
<p data-nodeid="79988">在 NameNode 修改 /etc/hadoop/conf/hosts 文件，添加新增的节点主机名，操作如下：</p>
<pre class="lang-dart" data-nodeid="149472"><code data-language="dart">vi /etc/hadoop/conf/hosts 
slave0191.iivey.cloud  
</code></pre>








<p data-nodeid="79990">最后，将配置同步到所有 DataNode 节点的机器上。<br>
<strong data-nodeid="80182">（4）使配置生效</strong></p>
<p data-nodeid="79991">新增节点后，要让 NameNode 识别新的节点，则需要在 NameNode 上刷新配置，执行如下操作：</p>
<pre class="lang-java" data-nodeid="150171"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> ~]$hdfs dfsadmin -refreshNodes  
</code></pre>

<p data-nodeid="79993"><strong data-nodeid="80187">（5）在新节点启动 dn 服务</strong></p>
<p data-nodeid="79994">在 NameNode 完成配置后，还需在新增节点上启动 DataNode 服务，执行如下操作：</p>
<pre class="lang-java" data-nodeid="150870"><code data-language="java">[hadoop<span class="hljs-meta">@slave0191</span>.iivey.cloud ~]$ hdfs --daemon start datanode 
</code></pre>

<p data-nodeid="79996">这样，一个新的节点就增加到集群了，Hadoop 的这种机制，可以在不影响现有集群运行的状态下，新增或者删除任意节点，非常方便。</p>
<h4 data-nodeid="154016" class="">8.NameNode 服务器故障了怎么办？</h4>





<p data-nodeid="79998">在 HDFS 集群中，NameNode 主机上存储了所有元数据信息，一旦这些信息丢失，那么 HDFS 上面的所有数据都将无法使用。</p>
<p data-nodeid="79999">所以 NameNode 服务器发生故障无法启动时，有两种方法可以解决：</p>
<ul data-nodeid="80000">
<li data-nodeid="80001">
<p data-nodeid="80002">NameNode 做了高可用服务的情况下，主 NameNode 故障后，NameNode 服务会自动切换到备用的 NameNode 上，这个过程是自动的，无须手工介入；</p>
</li>
<li data-nodeid="80003">
<p data-nodeid="80004">Namenode 没做高可用服务的情况下，可以借助 SecondaryNameNode 服务，在 SecondaryNameNode 主机中找到元数据信息，然后直接在此节点启动 Namenode 服务即可；由于 SecondaryNameNode 实现的是 Namenode 冷备份，所以这种方式可能无法找回所有数据，依旧会有部分数据丢失。</p>
</li>
</ul>
<p data-nodeid="80005">由此可知，对 NameNode 进行容灾备份至关重要，在生产环境下，我建议通过 standby  NameNode 实现 NameNode 的高可用热备份。</p>
<h4 data-nodeid="157136" class="">9.为什么集群节点被 Yarn 标记为不健康？</h4>





<p data-nodeid="80007">Yarn 集群在长期运行任务后，某些节点会突然被标记为不健康节点，并从 Yarn 集群中剔除出去，之后便不会再有任务提交至此节点。</p>
<p data-nodeid="80008">那么什么情况下，节点会被标记不健康呢？</p>
<p data-nodeid="80009">在 Yarn 配置中，有个参数 yarn.nodemanager.local-dirs，用来存储 NodeManager 应用程序运行的中间结果；还有另一个参数 yarn.nodemanager.log-dirs，用来指定 NodeManager 的日志文件存放目录列表。这两个参数都可以配置多个目录，并使用逗号将多个目录分隔开。</p>
<p data-nodeid="80010">Yarn 会定期进行磁盘状态检查，如果这两个参数指定目录的可用空间，低于 Yarn 指定的阈值，NodeManager 将不会在这些节点上启动任何新容器。</p>
<p data-nodeid="80011">本地目录健康检测主要涉及以下两个参数：</p>
<ul data-nodeid="80012">
<li data-nodeid="80013">
<p data-nodeid="80014">yarn.nodemanager.disk-health-checker.min-healthy-disks</p>
</li>
</ul>
<p data-nodeid="80015">此参数默认值为 0.25，表示正常目录在总目录中的数目占比，低于 0.25 则判定此节点处于不正常状态。比如，指定了十二个目录（磁盘），那么它们当中，至少有 3 个目录处于正常状态， NodeManager 才会在该节点上启动新容器。</p>
<ul data-nodeid="80016">
<li data-nodeid="80017">
<p data-nodeid="80018">yarn.nodemanager.disk-health-checker.max-disk-utilization-per-disk-percentage</p>
</li>
</ul>
<p data-nodeid="80019">此参数默认值为 90（也可以将此参数设置为 0 到 100 之间的任意数）。它表示 yarn.nodemanager.local-dirs 配置项下的路径或者 yarn.nodemanager.log-dirs 配置项下的路径的磁盘使用率达到了 90% 以上时，此台机器上的 nodemanager 将被标志为 unhealthy。</p>
<p data-nodeid="80020">解决方法很简单：清理对应目录下的临时数据，使磁盘占用率降至 90% 以下；修改 90 这一默认参数值，重设磁盘使用率。</p>
<p data-nodeid="80021" class=""><strong data-nodeid="80217">我有个经验可以与你分享：</strong> 最好别将 Yarn 的日志目录或中间结果目录，与 HDFS 的数据存储目录放至同一个磁盘，这样做能减少很多不必要的麻烦。</p>
<h4 data-nodeid="160229" class="">10.datanode 节点磁盘存储不均衡怎么解决？</h4>





<p data-nodeid="80023">在 HDFS 集群中，磁盘损坏是家常便饭，磁盘故障后，我们一般的策略是更换新的硬盘，新硬盘更换后，只有新数据会写入这个硬盘，而之前的老数据不会自动将数据平衡过来。</p>
<p data-nodeid="80024">如此下去，更换的硬盘越多，节点之间以及每个节点的各个磁盘之间的数据将越来越不平衡；此外，集群中添加新的数据节点，也会导致 HDFS 出现数据不平衡。</p>
<p data-nodeid="80025">那么如何让 HDFS 集群重新达到一个平衡的状态呢？可以使用 Hadoop 提供的 Balancer 程序，执行命令如下：</p>
<pre class="lang-java" data-nodeid="160911"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> sbin]$ $HADOOP_HOME/bin/start-balancer.sh  -t  <span class="hljs-number">5</span>% 
</code></pre>

<p data-nodeid="80027">或者：</p>
<pre class="lang-java" data-nodeid="161592"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> sbin]$ hdfs balancer -threshold <span class="hljs-number">5</span> 
</code></pre>

<p data-nodeid="80029">这个命令中 -t 参数后面跟的是，HDFS 达到平衡状态的磁盘使用率偏差值，如果节点与节点之间磁盘使用率偏差小于 5%，那么我们就认为 HDFS 集群已达到了平衡状态。</p>
<h4 data-nodeid="164657" class="">11.Yarn 集群中发现任务分配不均衡怎么办？</h4>





<p data-nodeid="80031">有时候，你通过 Yarn 集群运行数据分析任务时，会发现这样一个问题：各节点的负载会不均衡（也就是任务数目不同），有的节点有很多任务在执行忙碌，而有的节点没有任务执行，那么如何平衡各节点运行的任务数目呢？</p>
<p data-nodeid="80032">这种问题的发生与你采用的 Yarn 资源调度策略息息相关。</p>
<p data-nodeid="80033" class="">如果是上述情况，其原因应该是采用了默认的容量调度策略（Capacity Scheduler），容量调度会尽可能将任务分配到有资源的节点，而不考虑任务均衡因素。所以这种情况下，我建议将其设置为公平调度策略，此调度模式可以将任务均匀分配到集群的每个节点。</p>
<p data-nodeid="80034">其实，从 Hadoop 集群利用率角度看，该问题发生的概率比较低。因为一般情况下，任务会持续提交到集群，集群会时刻处于忙碌状态，不会出现节点一直空闲的情况，所以任务分配不均的情况也就难以发生。</p>
<h4 data-nodeid="167696" class="">12.HDFS 下有 missing blocks，应该如何解决？</h4>





<p data-nodeid="80036">HDFS 集群出现 missing blocks 错误，是一个经常发生的问题，并且一旦发生往往意味着有元数据丢失或者损坏，想要将其恢复，难度很大甚至无法恢复。</p>
<p data-nodeid="80037">所以我们的解决方法往往不是恢复数据，而是删除相关文件，具体如何解决如下所示，执行下列命令：</p>
<pre class="lang-java" data-nodeid="168366"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> sbin]$ hdfs fsck /blocks-path/ 
</code></pre>

<p data-nodeid="80039">此命令会检查 HDFS 下的所有块状态，并向你列出有哪些文件发生了块丢失或损坏。<br>
然后执行如下命令，删除这些文件即可：</p>
<pre class="lang-java" data-nodeid="169035"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> sbin]$ hdfs fsck -fs hdfs:<span class="hljs-comment">//bigdata/logs/mv.log  -delete </span>
</code></pre>

<p data-nodeid="80041">上面删除了 HDFS 上 mv.log 这个文件，因为此文件元数据丢失，无法恢复，所以只能删除。</p>
<h3 data-nodeid="173380" class="">Hadoop 调优之操作系统调优</h3>







<h4 data-nodeid="176365" class="">1.调整操作系统打开文件描述符的上限</h4>





<p data-nodeid="80044">Hadoop 的任务分析经常需要读写大量文件，因此需要增大打开文件描述符的上限，可通过 ulimit -n 查看目前系统的打开文件描述符的上限值。CentOS 7 系统默认值是 1024，这个值太小了，建议修改为 655360 或者更大。</p>
<p data-nodeid="80045">通过命令“ulimit -a”可以看到所有系统资源参数，这里面需要重点设置的是“open files”和“max user processes”，其他可以酌情设置。</p>
<p data-nodeid="80046">要永久设置资源参数，主要通过下列文件实现：</p>
<ul data-nodeid="80047">
<li data-nodeid="80048">
<p data-nodeid="80049">/etc/security/limits.conf</p>
</li>
<li data-nodeid="80050">
<p data-nodeid="80051">/etc/security/limits.d/90-nproc.conf(centos6.x)</p>
</li>
<li data-nodeid="80052">
<p data-nodeid="80053">/etc/security/limits.d/20-nproc.conf(centos7.x)</p>
</li>
</ul>
<p data-nodeid="80054">将下面内容添加到 /etc/security/limits.conf 中，然后退出 shell，重新登录即可生效。</p>
<pre class="lang-java" data-nodeid="177023"><code data-language="java">*        soft    nproc           <span class="hljs-number">204800</span> 
*        hard    nproc           <span class="hljs-number">204800</span> 
*        soft    nofile          <span class="hljs-number">655360</span> 
*        hard    nofile          <span class="hljs-number">655360</span> 
*        soft    memlock         unlimited 
*        hard    memlock         unlimited 
</code></pre>

<p data-nodeid="80056"><strong data-nodeid="80264">需要注意的是：</strong> CentOS 6.x 版本中，有个 90-nproc.conf 文件；CentOS 7.x 版本中，有个 20-nproc.conf 文件，由于里面已经默认配置了最大用户进程数，对这两个的设置也就没必要，所以直接删除这两个文件即可。</p>
<h4 data-nodeid="179980" class="">2.修改 net.core.somaxconn 参数</h4>





<p data-nodeid="80058" class="">此内核参数对应的具体文件路径为 /proc/sys/net/core/somaxconn，它用来设置 <strong data-nodeid="80278">socket</strong> 监听（listen）的  <strong data-nodeid="80279">backlog</strong>  上限。</p>
<p data-nodeid="180632" class="">什么是 backlog 呢？就是 Socket 的监听队列，当一个请求（Request）未被处理或建立时，便会进入 backlog。而  socket server 可以一次性处理 backlog 中的所有请求，处理后的请求不再位于监听队列中。</p>

<p data-nodeid="80060">如果 server 处理请求较慢，以至于监听队列被填满时，那么新来的请求会被拒绝，所以必须增大这个值，此参数默认值为 128。作为网络参数的基础优化，建议修改为如下值：</p>
<pre class="lang-java" data-nodeid="181280"><code data-language="java">echo <span class="hljs-number">4096</span> &gt;/proc/sys/net/core/somaxconn 
</code></pre>

<h4 data-nodeid="184192" class="">3.调整操作系统使用 swap 的比例</h4>





<p data-nodeid="80063">swap 原本是作为物理内存的扩展，但如今内存一般都很充足，swap 也就很少会应用；再加上数据交换至 swap，导致操作超时，从而影响 Hadoop 的读写以及数据分析性能。所以以上两点，导致如今使用 swap 的场景越来越少。</p>
<p data-nodeid="80064">我们可以通过系统内核参数 /proc/sys/vm/swappiness 来调整使用 swap 的比例。swappiness=0 的时候表示最大限度使用物理内存，然后才是 swap 空间；swappiness＝100 的时候表示积极地使用 swap 分区，并且把内存上的数据及时地搬运到 swap 空间里。</p>
<p data-nodeid="80065">Linux 基本默认设置为 60，表示你的物理内存使用到 100-60=40% 的时候，swap 交换分区便开始应用起来。对于内存需求较高的服务器（比如 Hadoop、Redis、HBase 机器），Linux 值需要设置得足够小（0~10 之间），这样才能最大限度使用物理内存。</p>
<h4 data-nodeid="186757" class="">4.禁用 THP（Transparent Huge Pages）功能</h4>




<p data-nodeid="80067">THP 的本意是为提升内存的性能，但是在 Hadoop 环境中发现，此功能会将 CPU 占用率增大，进而影响 Hadoop 性能，因此建议将其关闭。</p>
<p data-nodeid="80068">首先检查 THP 的启用状态：</p>
<pre class="lang-dart" data-nodeid="193108"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# cat /sys/kernel/mm/transparent_hugepage/defrag 
[always] madvise never 
[root<span class="hljs-meta">@localhost</span> ~]# cat /sys/kernel/mm/transparent_hugepage/enabled 
[always] madvise never 
</code></pre>










<p data-nodeid="80070">这里显示 always，表示 THP 目前是启用状态。要禁用 THP，可打开 /etc/rc.d/rc.local 文件，然后添加如下内容：</p>
<pre class="lang-java" data-nodeid="193743"><code data-language="java"><span class="hljs-keyword">if</span> test -f /sys/kernel/mm/transparent_hugepage/enabled; then 
echo never &gt; /sys/kernel/mm/transparent_hugepage/enabled 
fi 
<span class="hljs-keyword">if</span> test -f /sys/kernel/mm/transparent_hugepage/defrag; then 
echo never &gt; /sys/kernel/mm/transparent_hugepage/defrag 
fi 
</code></pre>

<p data-nodeid="194699">然后保存退出。</p>
<p data-nodeid="194700">最后，赋予 rc.local 文件执行权限，执行如下命令：</p>



<pre class="lang-dart" data-nodeid="201687"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# chmod +x /etc/rc.d/rc.local  
[root<span class="hljs-meta">@localhost</span> ~]# source /etc/rc.local 
</code></pre>











<p data-nodeid="80074" class="">此时，THP 功能便已经被禁用了。</p>
<h3 data-nodeid="80075">小结</h3>
<p data-nodeid="81068">本课时，我主要介绍了 Hadoop 大数据运维中，常见的运维故障以及解决问题的方法和思路；以及在 Hadoop 平台下，如何对 Linux 操作系统进行深度配置和优化。由于 Hadoop 的高效稳定运行，离不开 Linux 系统的性能优化，所以希望你课下能多多练习，提升自己的技能水平。</p>

---

### 精选评论


