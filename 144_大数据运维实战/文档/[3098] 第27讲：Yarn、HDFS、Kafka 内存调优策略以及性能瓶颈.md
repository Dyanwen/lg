<p data-nodeid="176474">Hadoop 性能调优是一项复杂烦琐、难度极大的工作，不仅要求对 Hadoop 本身有深刻理解，还涉及底层硬件、网络、操作系统、Java 虚拟机等方面的调优工作。</p>
<p data-nodeid="176475">Hadoop 性能调优，不仅靠运维，还需开发人员一起参与：</p>
<ul data-nodeid="176476">
<li data-nodeid="176477">
<p data-nodeid="176478">运维人员负责为用户提供一个高效稳定的任务运行环境；</p>
</li>
<li data-nodeid="176479">
<p data-nodeid="176480">开发人员则需要根据自己任务的特点写出好的程序，让任务快速、高效地完成。</p>
</li>
</ul>
<h3 data-nodeid="183497" class="">Yarn 的内存和 CPU 调优</h3>



<p data-nodeid="176482">Yarn 集群同时支持内存和 CPU 两种资源的调度，因此对 <strong data-nodeid="176748">Yarn 集群的调优</strong> 也分为 <strong data-nodeid="176749">Yarn 的内存参数</strong> 和 <strong data-nodeid="176750">CPU 参数调优</strong> 两部分。</p>
<p data-nodeid="176483">Yarn 作为一个 <strong data-nodeid="176764">资源管理器</strong> ，主要管理的资源为 <strong data-nodeid="176765">Container</strong> ，而它是 Yarn 里面资源分配的 <strong data-nodeid="176766">基本单位</strong> ，包含了一定的内存及 CPU 资源。</p>
<p data-nodeid="176484">在进行调优之前，有如下问题需要你关注：</p>
<ul data-nodeid="176485">
<li data-nodeid="176486">
<p data-nodeid="176487">一个集群节点的物理内存、CPU 如何分配给 Yarn？分配多少？</p>
</li>
<li data-nodeid="176488">
<p data-nodeid="176489">一个 Yarn 节点可以运行多少个 Container，每个 Container 占用多少内存和 CPU 合适？</p>
</li>
<li data-nodeid="176490">
<p data-nodeid="176491">一个 Yarn 节点上磁盘的数与计算性能是否有关系？</p>
</li>
<li data-nodeid="176492">
<p data-nodeid="176493">Yarn 上可配置的 CPU、内存资源有哪些？</p>
</li>
</ul>
<p data-nodeid="187308" class="">其实，在 Yarn 集群中，平衡内存、CPU、磁盘这三者的资源分配是很重要的，<strong data-nodeid="187314">最基本的经验就是：</strong> 每两个 Container 使用一块磁盘、一个 CPU 核，这种配置，可以使集群的资源得到一个比较好的平衡利用。</p>



<p data-nodeid="176495">接下来我会分别从内存、CPU 两方面介绍如何对 Yarn 集群进行资源优化。</p>
<h4 data-nodeid="194287" class="">1. Yarn 内存优化配置</h4>






<p data-nodeid="176497">Yarn 的内存优化主要涉及 ResourceManager、NodeManager 相关进程以及 Yarn 的一些配置参数。下面先从这些进程对应的 JVM 内存优化讲起。</p>
<p data-nodeid="176498">需要注意，对于内存资源的分配，要依据集群上每个节点运行的服务进行，因为一个服务器节点上会同时运行 HDFS、NodeManager、ResourceManager、HBase 等多项服务，这些服务都需要若干内存资源。</p>
<p data-nodeid="176499">所以不能将内存都分配给某个服务；此外，操作系统运行也需要一部分内存资源，我们还需要给系统预留足够的内存。</p>
<p data-nodeid="176500"><strong data-nodeid="176789">（1）进程堆内存优化</strong></p>
<ul data-nodeid="176501">
<li data-nodeid="176502">
<p data-nodeid="176503"><strong data-nodeid="176793">ResourceManager 堆内存参数</strong></p>
</li>
</ul>
<p data-nodeid="176504">在整个 Yarn 集群中只有一个 ResourceManager，它主要负责和客户端进行通信，为每个节点提供资源的调度和分配，同时与 AppMaster 一起进行资源的分配。</p>
<p data-nodeid="176505">如果 ResourceManager 的堆内存资源设置得不合理，那么垃圾回收时间可能过长，进而导致与 ZooKeeper 的连接超时；如果对 ResourceManager 做了主、备，那么还会引发 Active ResourceManager 的频繁切换问题，此过程中会导致一些任务执行失败。</p>
<p data-nodeid="176506">因此，对 ResourceManager 堆内存的大小设置一定要合理，通常建议将 ResourceManager 独立运行在一台服务器上，所以只考虑操作系统对内存的占用即可。</p>
<p data-nodeid="176507">在 hadoop 配置文件 yarn-env.sh 中添加如下内容，可设置 ResourceManager 堆内存大小：</p>
<pre class="lang-java" data-nodeid="196180"><code data-language="java">export YARN_RESOURCEMANAGER_HEAPSIZE=<span class="hljs-number">8192</span> 
export YARN_RESOURCEMANAGER_OPTS=<span class="hljs-string">"-server -Xms${YARN_RESOURCEMANAGER_HEAPSIZE}m -Xmx${YARN_RESOURCEMANAGER_HEAPSIZE}m"</span> 
</code></pre>


<p data-nodeid="176509">建议将 ResourceManager 堆内存大小设置在 <strong data-nodeid="176803">8GB 以上</strong> 。</p>
<ul data-nodeid="176510">
<li data-nodeid="176511">
<p data-nodeid="176512"><strong data-nodeid="176807">NesourceManager 堆内存参数</strong></p>
</li>
</ul>
<p data-nodeid="176513">NodeManager 充当 shuffle 任务的 server 端时，内存应该调大，否则会出现任务连接错误。同样修改 hadoop 配置文件 yarn-env.sh，应添加如下内容：</p>
<pre class="lang-java" data-nodeid="197441"><code data-language="java">export YARN_NODEMANAGER_HEAPSIZE=<span class="hljs-number">4096</span> 
export YARN_NODEMANAGER_OPTS=<span class="hljs-string">"-Xms${YARN_NODEMANAGER_HEAPSIZE}m -Xmx${YARN_NODEMANAGER_HEAPSIZE}m"</span> 
</code></pre>

<p data-nodeid="176515">建议将 NodeManager 堆内存大小设置在 <strong data-nodeid="176814">4GB 以上</strong> 。</p>
<p data-nodeid="176516">所有配置参数修改完成后，重启 ResourceManager、NodeManager 服务，配置即可生效。</p>
<p data-nodeid="176517"><strong data-nodeid="176819">（2）Yarn 内存配置参数优化</strong></p>
<p data-nodeid="176518">接着，来看一下需要优化的 Yarn 内存参数，主要有以下几个。</p>
<p data-nodeid="176519"><strong data-nodeid="176824">RM 的内存资源配置</strong></p>
<ul data-nodeid="176520">
<li data-nodeid="176521">
<p data-nodeid="176522">RM1:yarn.scheduler.minimum-allocation-mb，表示单个容器可以申请的最小内存，默认 1024MB。</p>
</li>
<li data-nodeid="176523">
<p data-nodeid="176524">RM2:yarn.scheduler.maximum-allocation-mb，表示单个容器可以申请的最大内存，默认8291MB。</p>
</li>
<li data-nodeid="176525">
<p data-nodeid="176526">APP1:yarn.app.mapreduce.am.resource.mb，MR 运行于 Yarn 上时，为 AM 分配多少内存。</p>
</li>
<li data-nodeid="176527">
<p data-nodeid="176528">APP2:yarn.app.mapreduce.am.command-opts，运行 MRAppMaster 时的 jvm 参数，可设置 -Xmx、-Xms 等选项。</p>
</li>
</ul>
<p data-nodeid="176529"><strong data-nodeid="176832">NM 的内存资源配置</strong></p>
<ul data-nodeid="176530">
<li data-nodeid="176531">
<p data-nodeid="176532">NM1:yarn.nodemanager.resource.memory-mb，表示节点可用的最大内存。</p>
</li>
<li data-nodeid="176533">
<p data-nodeid="176534">NM2:yarn.nodemanager.vmem-pmem-ratio，表示虚拟内存率，默认 2.1。</p>
</li>
</ul>
<p data-nodeid="176535"><strong data-nodeid="176838">AM 内存配置相关参数</strong></p>
<ul data-nodeid="176536">
<li data-nodeid="176537">
<p data-nodeid="176538">AM1:mapreduce.map.memory.mb，分配给 map Container 的内存大小。</p>
</li>
<li data-nodeid="176539">
<p data-nodeid="176540">AM2:mapreduce.map.java.opts，运行 map 任务的 jvm 参数，可设置 -Xmx、-Xms 等选项。</p>
</li>
<li data-nodeid="176541">
<p data-nodeid="176542">AM3:mapreduce.reduce.memory.mb，分配给 reduce Container 的内存大小。</p>
</li>
<li data-nodeid="176543">
<p data-nodeid="176544">AM4:mapreduce.reduce.java.opts，运行 reduce 任务的 jvm 参数，可设置 -Xmx、-Xms 等选项。</p>
</li>
</ul>
<p data-nodeid="176545"><strong data-nodeid="176846">需要注意</strong></p>
<ul data-nodeid="176546">
<li data-nodeid="176547">
<p data-nodeid="176548">RM1、RM2 的值均不能大于 NM1 的值</p>
</li>
<li data-nodeid="176549">
<p data-nodeid="176550">AM1 和 AM3 的值应该在 RM1 和 RM2 这两个值之间</p>
</li>
<li data-nodeid="176551">
<p data-nodeid="176552">AM3 的值最好为 AM1 的两倍</p>
</li>
<li data-nodeid="176553">
<p data-nodeid="176554">AM2 和 AM4 的值应该在 AM1 和 AM3 之间</p>
</li>
</ul>
<p data-nodeid="176555">根据上面给出的规则，我们给出一个例子，来说明下相关参数如何进行配置，当然还需要在运行测试过程中不断检验和调整。</p>
<p data-nodeid="176556"><strong data-nodeid="176855">（3）Yarn 参数优化实例</strong></p>
<p data-nodeid="176557">假定我们的 Yarn 集群有 100 个节点，每个节点配置 128GB 物理内存、32 核 CPU，并挂载 8 块磁盘，同时再运行 datanode 和 nodemanager 两个服务。</p>
<p data-nodeid="176558">要规划 Yarn 可使用的内存，首先需要去掉系统占用的内存，还需去掉 datanode、nodemanager 的 JVM 占用的内存，一般情况下给系统预留 20% 左右内存即可，而 datanode、nodemanager 的 JVM 内存都按照 4GB 来配置即可。</p>
<p data-nodeid="176559">这样算下来，Yarn 可用的内存为如下公式结果：128-(128*20%)-4-4≈94GB。</p>
<p data-nodeid="176560">此时通过如下公式，可计算出 <strong data-nodeid="176866">每个节点最多能拥有多少个 container</strong> ：</p>
<pre class="lang-java" data-nodeid="198702"><code data-language="java">containers = min (<span class="hljs-number">2</span>*CORES, <span class="hljs-number">1.8</span>*DISKS, (Total available RAM) / MIN_CONTAINER_SIZE) 
</code></pre>

<p data-nodeid="176562">其中：</p>
<ul data-nodeid="176563">
<li data-nodeid="176564">
<p data-nodeid="176565">CORES 为机器 CPU 的核数；</p>
</li>
<li data-nodeid="176566">
<p data-nodeid="176567">DISKS 为机器上挂载的磁盘个数；</p>
</li>
<li data-nodeid="176568">
<p data-nodeid="176569">Total available RAM 为 Yarn 可用内存；</p>
</li>
<li data-nodeid="176570">
<p data-nodeid="176571">MIN_CONTAINER_SIZE 是指 Container 最小的容量大小，24GB 物理内存以上机器的 Container 最小值建议为 2GB。</p>
</li>
</ul>
<p data-nodeid="205648" class="">根据上面的计算公式，每个节点可以拥有 14（min (2*32, 1.8*8 , (94)/2) = min (64, 14, 47) =14 个 container。</p>






<p data-nodeid="176573">通过如下公式，可计算 <strong data-nodeid="176887">每个 Container 占用的内存大小</strong> ：</p>
<pre class="lang-yaml" data-nodeid="262308"><code data-language="yaml"><span class="hljs-string">RAM-per-container</span> <span class="hljs-string">=</span> <span class="hljs-string">max(MIN_CONTAINER_SIZE,</span> <span class="hljs-string">(Total</span> <span class="hljs-string">Available</span> <span class="hljs-string">RAM)/containers)</span> 
</code></pre>













































<p data-nodeid="176575">因此，RAM-per-container = max (2, (94)/14) = max (2, 7) = 7，也就是说每个 Container 的内存大小为 7GB，由于采用四舍五入算法，所以我也可以设置每个 Container 为 6GB 内存。</p>
<p data-nodeid="176576">知道了每个节点上的 Container 数以及每个 Container 占用的内存后，就可以算出 Yarn 几个内存参数占用的资源量了。计算方法如下表所示：</p>
<table data-nodeid="266314">
<thead data-nodeid="266315">
<tr data-nodeid="266316">
<th data-org-content="配置文件" data-nodeid="266318">配置文件</th>
<th data-org-content="配置项" data-nodeid="266319">配置项</th>
<th data-org-content="计算公式" data-nodeid="266320">计算公式</th>
<th data-org-content="计算结果" data-nodeid="266321">计算结果</th>
</tr>
</thead>
<tbody data-nodeid="266326">
<tr data-nodeid="266327">
<td data-org-content="yarn-site.xml" data-nodeid="266328">yarn-site.xml</td>
<td data-org-content="yarn.nodemanager.resource.memory-mb" data-nodeid="266329">yarn.nodemanager.resource.memory-mb</td>
<td data-org-content="containers * RAM-per-container" data-nodeid="266330">containers * RAM-per-container</td>
<td data-org-content="84GB" data-nodeid="266331">84GB</td>
</tr>
<tr data-nodeid="266332">
<td data-org-content="yarn-site.xml" data-nodeid="266333">yarn-site.xml</td>
<td data-org-content="yarn.scheduler.minimum-allocation-mb" data-nodeid="266334">yarn.scheduler.minimum-allocation-mb</td>
<td data-org-content="RAM-per-container" data-nodeid="266335">RAM-per-container</td>
<td data-org-content="6GB" data-nodeid="266336">6GB</td>
</tr>
<tr data-nodeid="266337">
<td data-org-content="yarn-site.xml" data-nodeid="266338">yarn-site.xml</td>
<td data-org-content="yarn.scheduler.maximum-allocation-mb" data-nodeid="266339">yarn.scheduler.maximum-allocation-mb</td>
<td data-org-content="containers * RAM-per-container" data-nodeid="266340">containers * RAM-per-container</td>
<td data-org-content="84GB" data-nodeid="266341">84GB</td>
</tr>
<tr data-nodeid="266342">
<td data-org-content="yarn-site.xml" data-nodeid="266343">yarn-site.xml</td>
<td data-org-content="yarn.app.mapreduce.am.resource.mb" data-nodeid="266344">yarn.app.mapreduce.am.resource.mb</td>
<td data-org-content="2 * RAM-per-container" data-nodeid="266345">2 * RAM-per-container</td>
<td data-org-content="12GB" data-nodeid="266346">12GB</td>
</tr>
<tr data-nodeid="266347">
<td data-org-content="yarn-site.xml" data-nodeid="266348">yarn-site.xml</td>
<td data-org-content="yarn.app.mapreduce.am.command-opts" data-nodeid="266349">yarn.app.mapreduce.am.command-opts</td>
<td data-org-content="0.8 * 2 * RAM-per-container" data-nodeid="266350">0.8 * 2 * RAM-per-container</td>
<td data-org-content="9.6GB" data-nodeid="266351">9.6GB</td>
</tr>
<tr data-nodeid="266352">
<td data-org-content="mapred-site.xml" data-nodeid="266353">mapred-site.xml</td>
<td data-org-content="mapreduce.map.memory.mb" data-nodeid="266354">mapreduce.map.memory.mb</td>
<td data-org-content="RAM-per-container" data-nodeid="266355">RAM-per-container</td>
<td data-org-content="6GB" data-nodeid="266356">6GB</td>
</tr>
<tr data-nodeid="266357">
<td data-org-content="mapred-site.xml" data-nodeid="266358">mapred-site.xml</td>
<td data-org-content="mapreduce.reduce.memory.mb" data-nodeid="266359">mapreduce.reduce.memory.mb</td>
<td data-org-content="2 * RAM-per-container" data-nodeid="266360">2 * RAM-per-container</td>
<td data-org-content="12GB" data-nodeid="266361">12GB</td>
</tr>
<tr data-nodeid="266362">
<td data-org-content="mapred-site.xml" data-nodeid="266363">mapred-site.xml</td>
<td data-org-content="mapreduce.map.java.opts" data-nodeid="266364">mapreduce.map.java.opts</td>
<td data-org-content="0.8 * RAM-per-container" data-nodeid="266365">0.8 * RAM-per-container</td>
<td data-org-content="4.8GB" data-nodeid="266366">4.8GB</td>
</tr>
<tr data-nodeid="266367">
<td data-org-content="mapred-site.xml" data-nodeid="266368">mapred-site.xml</td>
<td data-org-content="mapreduce.reduce.java.opts" data-nodeid="266369">mapreduce.reduce.java.opts</td>
<td data-org-content="0.8 * 2 * RAM-per-container" data-nodeid="266370">0.8 * 2 * RAM-per-container</td>
<td data-org-content="9.6GB" data-nodeid="266371">9.6GB</td>
</tr>
</tbody>
</table>



<p data-nodeid="176640">这个计算方法是官方推荐的，但有些参数的内存设置可能过大了。例如，单个容器可以申请的最大内存，这个一般情况下不需要配置这么大，可根据情况进行降低，其他参数也同理。</p>
<h4 data-nodeid="273308" class="">2. CPU 优化参数</h4>






<p data-nodeid="176642">由于每个服务器的 CPU 计算能力可能不同，因此 Yarn 引入了虚拟 CPU 的概念，将物理 CPU 划分为多个虚拟 CPU。这里的虚拟 CPU，可通过如下命令查看：</p>
<pre class="lang-dart" data-nodeid="283269"><code data-language="dart">[root<span class="hljs-meta">@nnmaster</span> scripts]#cat /proc/cpuinfo| grep <span class="hljs-string">"cpu cores"</span> |uniq  
</code></pre>








<p data-nodeid="176644">这个命令得到的结果是 <strong data-nodeid="176967">每颗 CPU 的核数，然后</strong> 乘以 <strong data-nodeid="176968">CPU 个数</strong> ，就得到此服务器有多少个可用的 <strong data-nodeid="176969">CPU core</strong> ，知道了每个集群节点的 CPU 核数后，就可以对 CPU 进行相关配置优化了。</p>
<p data-nodeid="176645">在 Yarn 中，和 CPU 配置相关的参数有如下几个：</p>
<ul data-nodeid="176646">
<li data-nodeid="176647">
<p data-nodeid="176648">yarn.nodemanager.resource.cpu-vcores 表示该节点服务器上 Yarn 可使用的虚拟 CPU 个数，默认为 8。由于需要给操作系统、datanode、nodemanager 进程预留一定量的 CPU，所以一般将系统 CPU 的 90% 留给此参数即可，例如集群节点有 32 个 core，那么分配 28 个 core 给 yarn。</p>
</li>
<li data-nodeid="176649">
<p data-nodeid="176650">yarn.scheduler.minimum-allocation-vcores 表示单个任务可申请的最小虚拟核数，默认为 1。</p>
</li>
<li data-nodeid="176651">
<p data-nodeid="176652">yarn.scheduler.maximum-allocation-vcores 表示单个任务可申请的最大虚拟核数，默认为 4。</p>
</li>
</ul>
<p data-nodeid="176653">这里我们结合上文介绍的内存调优案例讲一下，由于每个节点最多可运行 14 个 container，每个 container 占用 6GB 内存。如果设置每个 container 占用 2 个 CPU core，那么每个节点刚好分配了 28 个 CPU core。通过这样的配置，CPU、内存资源刚好充分得到了利用，没有资源浪费。</p>
<h3 data-nodeid="288872" class="">HDFS 性能调优</h3>





<p data-nodeid="176655">对 HDFS 文件进行性能优化，你需要对 HDFS 的构成和运行原理有一定的了解，我在前面的课时中对此已做了详细介绍，你可以回顾复习一下。</p>
<p data-nodeid="176656">接下来我会从 HDFS 配置参数、JVM 堆内存两方面介绍如何进行 HDFS 性能优化。</p>
<h4 data-nodeid="295687" class="">1. HDFS 性能参数配置</h4>






<ul data-nodeid="176658">
<li data-nodeid="176659">
<p data-nodeid="176660"><strong data-nodeid="176988">dfs.replication</strong></p>
</li>
</ul>
<p data-nodeid="176661">此参数用来设置 <strong data-nodeid="176994">文件副本数</strong> ，副本数通常设为 3，不建议修改。这个参数可用来保障 HDFS 数据安全，副本数越多，浪费的磁盘存储空间越多，但数据安全性也会更高。</p>
<p data-nodeid="176662">所以，此参数要根据整体情况来权衡设置，比如对于 100 个节点以内的小集群，将参数设为 3 便是个不错的选择，随着集群节点数的增多，你可适当增加副本数。</p>
<ul data-nodeid="176663">
<li data-nodeid="176664">
<p data-nodeid="176665"><strong data-nodeid="176999">dfs.block.size</strong></p>
</li>
</ul>
<p data-nodeid="176666">此参数用来设置 HDFS 中 <strong data-nodeid="177005">数据块的大小</strong> ，默认为 128M，所以，存储到 HDFS 的数据最好都大于 128M 或者是 128 的整数倍，这是最理想的情况；对于数据量较大的集群，可设为 256MB 或  512MB。数据块设置太小，会增加 NameNode 的压力；数据块设置过大，会增加定位数据的时间。</p>
<ul data-nodeid="176667">
<li data-nodeid="176668">
<p data-nodeid="176669"><strong data-nodeid="177009">dfs.datanode.data.dir</strong></p>
</li>
</ul>
<p data-nodeid="176670">此参数是设置 HDFS <strong data-nodeid="177015">数据块的存储路径</strong> ，配置的值应当是分布在各个独立磁盘上的目录，这样可以充分利用节点的 IO 读写能力，提高 HDFS 读写性能。</p>
<ul data-nodeid="176671">
<li data-nodeid="176672">
<p data-nodeid="176673"><strong data-nodeid="177019">dfs.datanode.max.transfer.threads</strong></p>
</li>
</ul>
<p data-nodeid="176674">这个值是配置 datanode 可 <strong data-nodeid="177025">同时处理的最大文件数量</strong> ，建议将这个值尽量调大，最大值可以配置为 65535。</p>
<h4 data-nodeid="309859" class="">2. HDFS 内存资源调优</h4>














<p data-nodeid="176676">优化 HDFS 内存资源，主要是配置 Namenode 的堆内存。Namenode 堆内存配置过小，会导致频繁产生 Full GC，进而导致 namenode 宕机；而在 Hadoop 中，数据的读取、写入都要经过 namenode，所以 namenode 的堆内存需要足够多，尤其是在海量数据读写场景中。</p>
<p data-nodeid="176677">此外 Hadoop 集群中的文件数越多，Namenode 的内存压力也会越来越大，这也是 Hadoop 不适于存储海量小文件的原因。</p>
<p data-nodeid="176678">企业应用中，一般将 Namenode 运行在一台独立的服务器上，要设置 Namenode 堆内存大小，可通过在 Hadoop 配置文件 hadoop-env.sh 中添加如下内容实现：</p>
<pre class="lang-java" data-nodeid="311087"><code data-language="java">export HADOOP_HEAPSIZE_MAX=<span class="hljs-number">30720</span> 
export HADOOP_HEAPSIZE_MIN=<span class="hljs-number">30720</span> 
</code></pre>

<p data-nodeid="176680">这里建议 Namenode 堆内存大小设置为物理内存的一半以上。</p>
<p data-nodeid="176681">接着，是对 Datanode 堆内存参数的设置，同样修改 Hadoop 配置文件 hadoop-env.sh，添加如下内容：</p>
<pre class="lang-java" data-nodeid="312314"><code data-language="java">export HDFS_DATANODE_HEAPSIZE=<span class="hljs-number">4096</span> 
export HDFS_DATANODE_OPTS=<span class="hljs-string">"-Xms${HDFS_DATANODE_HEAPSIZE}m -Xmx${HDFS_DATANODE_HEAPSIZE}m"</span> 
</code></pre>

<p data-nodeid="176683">建议 Datanode 堆内存大小设置为 4GB 以上。</p>
<p data-nodeid="176684">所有配置参数修改完成后，重启 Namenode、Datanode 服务以使配置生效。</p>
<h3 data-nodeid="316605" class="">Kafka 性能调优</h3>




<p data-nodeid="176686">Kafka 是一个高吞吐量分布式消息系统，并且提供了持久化存储功能，其高性能有两个重要特征：</p>
<ul data-nodeid="176687">
<li data-nodeid="176688">
<p data-nodeid="176689">磁盘连续读写性能远远高于随机读写的特点；</p>
</li>
<li data-nodeid="176690">
<p data-nodeid="176691">通过将一个 topic 拆分多个 partition，可提供并发和吞吐量。</p>
</li>
</ul>
<p data-nodeid="176692">要充分发挥 Kafka 的性能，就需要满足以上两个条件，常见的调优方式有如下。</p>
<h4 data-nodeid="323326" class="">1. 内存优化</h4>






<p data-nodeid="176694">对内存的调优，主要是对 Linux 系统下内存的优化，这里需要关注两个操作系统参数。</p>
<p data-nodeid="176695"><strong data-nodeid="177057">（1）vm.dirty_background_ratio</strong></p>
<p data-nodeid="176696">这个参数指的是，当文件系统缓存脏页（Page Cache 中的数据称为脏页数据）数量在系统中的内存占比达到一定比例（默认 10%）时，就会触发后台回写进程，将一定缓存的脏页异步地刷入磁盘。增、减这个值是最主要的调优手段。</p>
<p data-nodeid="176697"><strong data-nodeid="177064">（2）vm.dirty_ratio</strong></p>
<p data-nodeid="176698">这个参数指的是，当文件系统缓存脏页数量在系统中的内存占比达到一定比例（默认 20%）时，为了避免数据丢失，系统就不得不开始处理缓存脏页，将过多的脏页刷入磁盘。在这一过程中，可能会由于系统刷新内存数据到磁盘，导致应用进程出现 IO 阻塞。</p>
<p data-nodeid="176699">上述两个系统参数对应的内核参数文件如下：</p>
<pre class="lang-java" data-nodeid="342767"><code data-language="java">vm.dirty_background_ratio：/proc/sys/vm/dirty_background_ratio 
vm.dirty_ratio：/proc/sys/vm/dirty_ratio 
</code></pre>
















<p data-nodeid="176701">建议将 vm.dirty_background_ratio 设置为 5%，vm.dirty_ratio 设置为 10%，但还是要根据环境的不同，具体问题具体分析，总之需要你进行测试、测试，再测试。</p>
<p data-nodeid="176702">最后，Kafka 虽然也是 Java 应用，但是它需要的堆内存较小（因为数据不放入 JVM 中），因此，建议将物理内存的 60% 以上分配给给操作系统，来保证 page cache 的分配。</p>
<h4 data-nodeid="349454" class="">2. topic 的拆分</h4>






<p data-nodeid="176704">Kafka 读写的单位是 partition，将一个 topic 拆分为多个 partition 可以提高系统的吞吐量， <strong data-nodeid="177084">但有一个前提条件：不同的 partition 要分布在不同的磁盘上。</strong> 如果多个 partition 分布在同一个磁盘上，那么多个进程就会同时对一个磁盘的多个文件进行读写，这会使操作系统对磁盘的读写进行频繁的调度，从而破坏磁盘读写的连续性。</p>
<p data-nodeid="176705">要将不同的 partition 分布在不同在磁盘上，可以将磁盘的多个目录配置到 broker 的 log.dirs，例如下面的配置：</p>
<pre class="lang-java" data-nodeid="350664"><code data-language="java">log.dirs=/disk1/logs,/disk2/logs,/disk3/logs 
</code></pre>

<p data-nodeid="176707">Kafka 会在新建 partition 的时候，将新 partition 分布在 partition 数量最少的目录上，因此，一般不能将同一个磁盘的多个目录设置到 log.dirs。</p>
<h4 data-nodeid="357312" class="">3. 配置参数优化</h4>






<p data-nodeid="176709"><strong data-nodeid="177094">（1）日志保留策略配置优化</strong></p>
<p data-nodeid="176710">当 Kafka Server 被写入海量消息后，会生成很多数据文件，且占用大量磁盘空间，如果你不及时清理，磁盘空间可能会不够用。Kafka 默认保留天数是 7 天，但我建议尽量减少日志保留时间，推荐设置为 3 天即可。</p>
<p data-nodeid="176711">可如下，设置日志保留天数（以 3 天为例）：</p>
<pre class="lang-java" data-nodeid="358516"><code data-language="java"> log.retention.hours=<span class="hljs-number">72</span> 
</code></pre>

<p data-nodeid="176713"><strong data-nodeid="177100">（2）段文件大小优化</strong></p>
<p data-nodeid="176714">段文件配置 1GB，有利于快速回收磁盘空间，重启 Kafka 加载也会加快；相反，如果文件过小，则文件数量就会比较多，Kafka 启动时，是单线程扫描目录（log.dir）下所有数据文件。在这种情况下，扫描文件就会消耗很多时间，影响 Kafka 启动速度。</p>
<p data-nodeid="176715">配置段文件大小，可在 Kafka 配置文件中添加如下内容：</p>
<pre class="lang-java" data-nodeid="359719"><code data-language="java">log.segment.bytes=<span class="hljs-number">1073741824</span> 
</code></pre>

<p data-nodeid="176717"><strong data-nodeid="177106">（3）log 数据文件刷盘策略优化</strong></p>
<p data-nodeid="176718">为大幅度提高 producer 写入吞吐量，需要定期批量写文件，优化建议如下。</p>
<p data-nodeid="176719">每当 producer 写入 10000 条消息时，刷数据到磁盘，可通过如下选项配置：log.flush.interval.messages=10000。</p>
<p data-nodeid="176720">每间隔 1 秒钟时间，刷数据到磁盘，可通过如下选项配置：log.flush.interval.ms=1000。</p>
<p data-nodeid="176721">对 Kafka 参数的优化，并不是固定不变的，你要根据应用场景的不同，进行多次修改、测试，反复调试，直到最终找出最合理的配置。</p>
<h3 data-nodeid="176722">小结</h3>
<p data-nodeid="178401">本课时主要讲解了对 Yarn 集群、HDFS 集群、Kafka 集群的调优思路和具体方法，调优讲究的是因地制宜、具体问题具体分析。如果你的集群还没有暴露出性能问题，那么不要贸然进行调优，这将毫无意义，只有出现了性能问题，才需要进行调优，此时根据现象判断问题，然后找到合适的点进行优化。这是调优的一个基础宗旨。</p>

---

### 精选评论


