<p data-nodeid="156463" class="">本课时主要讲解“Flink 独立集群模式与 Flink on Yarn 模式应用实战”。</p>
<h3 data-nodeid="156464">Flink 概念及架构介绍</h3>
<p data-nodeid="156465">Flink 是一个高性能、高吞吐、低延迟的流处理框架，用于在无边界和有边界流上进行有状态计算。相对于 MapReduce 和 Spark，<strong data-nodeid="156659">Flink 真正做到了高吞吐、低延迟、高性能</strong>。在国内比较出名的互联网公司如阿里巴巴、美团、滴滴等，都在大规模使用 Flink 作为企业的分布式大数据处理引擎。</p>
<p data-nodeid="156466">在 Flink 中，任何类型的数据都可以形成一种事件流，比如 App 浏览日志、电话呼叫记录、订单日志等，所有这些数据都可称为一种流。在 Flink 中定义了<strong data-nodeid="156669">无边界流</strong>和<strong data-nodeid="156670">有边界流</strong>两种，所谓无边界流，就是有定义流的开始，但没有定义流的结束，数据会持续不断无休止地产生，在任何时候输入都不会完成。这种类型的数据在产生后就需要立即处理，无边界流也称为实时流，通常用于数据的实时分析和处理。</p>
<p data-nodeid="156467">有边界流是有定义流的开始，也有定义流的结束，即处理一个时间段的数据，可以等待数据产生完毕后，再进行处理，有边界流处理通常被称为<strong data-nodeid="156676">批处理</strong>。MapReduce 和 Spark 都是进行批处理的计算框架。</p>
<p data-nodeid="156468">由此可知，Flink 不但支持实时流处理，也支持传统的批处理，精确的时间控制和状态变化使 Flink 能够处理任何无边界流的应用。同时，通过高效的算法和数据结构处理，使得 Flink 也能高效运行批处理任务。</p>
<p data-nodeid="158963">Flink 的集群架构是基于 master/slave 模式，它由一个 Flink Master 和多个 Task Manager 组成，Flink Master 和 Task Manager 是进程级组件，其他的组件都是进程内的组件，内部结构如下图所示。</p>
<p data-nodeid="158964" class=""><img src="https://s0.lgstatic.com/i/image/M00/1F/C5/CgqCHl7nNd6AMeXKAAGBwtm077A086.png" alt="image" data-nodeid="158968"></p>


<p data-nodeid="156471">一个 Flink Master 中由一个 Resource Manager 和多个 Job Manager 组成 ，每一个 Job Manager 单独管理一个具体的 Job，Job Manager 中的 Scheduler 组件负责调度执行该 Job 中所有 Task，也就是整个资源调度的起点。</p>
<p data-nodeid="156472">Flink 的资源调度是一个经典的两层模型，其中从集群到 Job 的分配过程是由 Slot Manager 来完成，Job 内部分配给 Task 资源的过程则是由 Scheduler 来完成。Scheduler 向 Slot Pool 发出资源请求，Slot Pool 如果不能满足该资源需求则会进一步请求 Resource Manager，具体来满足该请求的组件是 Slot Manager。</p>
<p data-nodeid="156473">Flink Master 中唯一的 Resource Manager 负责整个 Flink 集群的资源调度以及与外部调度系统对接，这里的外部调度系统指的是 Kubernetes、Yarn 等资源管理系统。</p>
<p data-nodeid="156474">Task Manager 主要负责 Task 的执行过程，其中的 Slot 是 Task Manager 资源的一个子集，也是 Flink 资源管理的基本单位，Slot 的概念贯穿资源调度过程的始终。</p>
<h3 data-nodeid="156475">Flink 的应用场景</h3>
<p data-nodeid="156476">在企业实际应用中，有大量数据持续产生，比如电商订单数据、移动 App 浏览日志数据、银行交易数据、网络流量数据等，这些数据的一个共同点是从不同的数据源中产生，然后再传送到后台的数据分析系统，接着就可以对这些数据进行各种场景的分析。常见的应用场景有实时报表分析、流数据分析、实时监控、实时仓库等。</p>
<p data-nodeid="156477">实时报表分析最主要的应用是实时大屏展示，利用流式计算实时得出结果，然后直接被推送到前端展示，实时显示出重要指标的变换情况。比如，最引人注目的就是天猫双十一直播大屏不停地在更新成交额，别看这一个简单的数据展示，其实在数据处理阶段经历了数据采集、数据计算、数据校验，最终落到大屏上展现，整个过程的处理时间会在 5 秒以内完成。这就是实时报表。</p>
<p data-nodeid="156478">流数据分析可以实时计算各类数据指标，并通过实时的结果反馈 及时调整决策，最典型的应用是实时化分析 Web 类应用或者 App 应用的各项指标。例如，App 的打开率、访问地域、故障分布点等，通过实时结果，可帮助企业实现精细化运营、提升产品质量和体验。</p>
<p data-nodeid="156479">实时监控主要是通过状态数据，实现用户行为预警、App Crash 预警、服务器攻击预警等功能，还可以对用户行为或者相关事件进行实时监测和分析，基于风控规则进行预警。</p>
<p data-nodeid="156480">实时仓库是结合离线数据，通过流处理的优势，对数据进行实时清洗、归并、结构化，为离线数据进行补充和优化。</p>
<h3 data-nodeid="156481">Flink 独立集群的安装与配置</h3>
<p data-nodeid="156482">Flink 支持多种部署方式，常用的有本地模式、独立集群模式及 Yarn 集成模式，<strong data-nodeid="156698">本地模式开箱即用</strong>，主要用于测试，独立集群模式运行不需要依赖外部系统，完全自己独立管理。现在大多数企业因为大数据平台都以 Yarn 作为资源管理器，所以 Fink 也支持运行在 Yarn 上，为了方便管理，很多企业选择了 Flink on Yarn 这种模式。</p>
<p data-nodeid="156483">下面先介绍下 Flink 独立集群模式的部署，然后介绍 Flink on Yarn 模式的应用。</p>
<h4 data-nodeid="156484">1.Flink独立集群的部署过程</h4>
<p data-nodeid="156485">独立模式下的 Flink 集群是基于 Master、Slave 架构的，我这里以 4 台主机为例，1 台 Master 角色，3 台 Slave 角色，Flink 集群主机规划如下表所示：</p>
<table data-nodeid="156487">
<thead data-nodeid="156488">
<tr data-nodeid="156489">
<th data-nodeid="156491"><strong data-nodeid="156705">主机名</strong></th>
<th align="center" data-nodeid="156492"><strong data-nodeid="156709">部署服务</strong></th>
<th data-nodeid="156493"><strong data-nodeid="156713">角色</strong></th>
</tr>
</thead>
<tbody data-nodeid="156497">
<tr data-nodeid="156498">
<td data-nodeid="156499">nnmaster.cloud(172.16.213.151)</td>
<td align="center" data-nodeid="156500">主Namenode、flink、JDK</td>
<td data-nodeid="156501">flink master</td>
</tr>
<tr data-nodeid="156502">
<td data-nodeid="156503">yarnserver.cloud(172.16.213.152)</td>
<td align="center" data-nodeid="156504">备Namenode、flink、JDK</td>
<td data-nodeid="156505">flink slave1</td>
</tr>
<tr data-nodeid="156506">
<td data-nodeid="156507">slave001.cloud(172.16.213.138)</td>
<td align="center" data-nodeid="156508">Datanode、flink、JDK</td>
<td data-nodeid="156509">flink slave2</td>
</tr>
<tr data-nodeid="156510">
<td data-nodeid="156511">slave002.cloud(172.16.213.80)</td>
<td align="center" data-nodeid="156512">Datanode、flink、JDK</td>
<td data-nodeid="156513">flink slave3</td>
</tr>
</tbody>
</table>
<p data-nodeid="156514">所有主机操作系统采用 Centos7.7 版本，硬件配置最低 8 核 8GB 内存。我这里是 16 核 48GB 内存，JDK 版本 1.8 以上。</p>
<p data-nodeid="156515">接着，点击 <a href="https://flink.apache.org/downloads.html" data-nodeid="156730">https://flink.apache.org/downloads.html</a> 下载对应的 Flink 版本，我这里下载的是 flink-1.10.1-bin-scala_2.11.tgz 二进制版本，下载完成后，解压即可完成安装。这里我将 Flink 安装程序放到 /opt/bigdata 目录下，简单操作如下：</p>
<pre class="lang-java" data-nodeid="156516"><code data-language="java">[root@nnmaster ~]# tar zxvf flink-1.10.1-bin-scala_2.11.tgz  -C /opt/bigdata
[root@nnmaster ~]# cd /opt/bigdata/
[root@nnmaster bigdata]# mkdir flink
[root@nnmaster bigdata]# mv flink-1.10.1 flink
[root@nnmaster bigdata]# cd flink
[root@nnmaster flink]# ln -s flink-1.10.1 current
</code></pre>
<p data-nodeid="156517">这样 Flink 在一个节点就安装完成了，安装完成后，先不要着急复制 Flink 程序到其他节点，等 Flink 配置完成后，再复制 Flink 程序到剩余其他节点。</p>
<h4 data-nodeid="156518">2. Flink 独立集群的配置</h4>
<p data-nodeid="156519">Flink 的配置文件位于程序目录的 conf 子目录中，独立集群模式下需要配置的文件有三个，分别是 flink-conf.yaml、masters 和 slaves，分别介绍如下。</p>
<p data-nodeid="156520">首先打开 flink-conf.yaml 文件，修改或添加如下配置：</p>
<pre class="lang-java" data-nodeid="156521"><code data-language="java">jobmanager.rpc.address: nnmaster.cloud
jobmanager.heap.size: <span class="hljs-number">2048</span>m
taskmanager.memory.flink.size: <span class="hljs-number">4096</span>m
taskmanager.numberOfTaskSlots: <span class="hljs-number">10</span>
</code></pre>
<p data-nodeid="156522">对这些选项含义介绍如下：</p>
<ul data-nodeid="156523">
<li data-nodeid="156524">
<p data-nodeid="156525">jobmanager.rpc.address：设置 JobManager 的 IP 地址或者主机名；</p>
</li>
<li data-nodeid="156526">
<p data-nodeid="156527">jobmanager.heap.size：设置 JobManager 的 JVM heap 大小；</p>
</li>
<li data-nodeid="156528">
<p data-nodeid="156529">taskmanager.memory.flink.size：设置此节点上 taskmanager 可使用的总内存大小；</p>
</li>
<li data-nodeid="156530">
<p data-nodeid="156531">taskmanager.numberOfTaskSlots：设置 taskManager 中 taskSlots 个数，最好设置成 taskmanager 节点的 CPU 核数相等。</p>
</li>
</ul>
<p data-nodeid="156532">接着，修改 masters 文件，内容如下：</p>
<pre class="lang-java" data-nodeid="156533"><code data-language="java">nnmaster.cloud:<span class="hljs-number">8081</span>
</code></pre>
<p data-nodeid="156534">这里是指定 Flink 的 master 节点的主机名和端口，Flink 默认的 Web 端口为 8081。</p>
<p data-nodeid="156535">最后，修改 slaves 文件，内容如下：</p>
<pre class="lang-java" data-nodeid="156536"><code data-language="java">yarnserver.cloud
slave001.cloud
slave002.cloud
</code></pre>
<p data-nodeid="156537">这是添加 Fink 集群的 slave 节点，根据之前规划，添加三个 slave 节点，每行为一个节点的主机名。<br>
至此，Flink 的基础配置就完成了，然后将 Flink 整个目录打包，并复制到集群的其他节点。</p>
<p data-nodeid="156538">这里我们以 Hadoop 用户启动 Flink 集群，在启动集群之前，还需要在每个集群节点的 Hadoop 用户下 .bash_profile 文件中添加 Flink 的环境变量信息，内容如下：</p>
<pre class="lang-java" data-nodeid="156539"><code data-language="java">export FLINK_HOME=/opt/bigdata/flink/current
export PATH=$PATH:$FLINK_HOME/bin
</code></pre>
<p data-nodeid="156540">下面就可以启动 Flink 集群服务了，这里我们只需在 Flink 的 Master 节点执行如下命令即可：</p>
<pre class="lang-java" data-nodeid="156541"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> bin]$ /opt/bigdata/flink/current/bin/start-cluster.sh
</code></pre>
<p data-nodeid="156542">start-cluster.sh 脚本会首先启动 master 服务，然后自动连接到 Flink 集群的其他节点，接着逐个启动 TaskManager 服务。但前提是需要做好 master 节点到其他 slave 节点的无密码登录，不然此脚本无法执行。</p>
<p data-nodeid="156543">同样，关闭集群的话，可以执行如下命令：</p>
<pre class="lang-java" data-nodeid="156544"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> bin]$ /opt/bigdata/flink/current/bin/stop-cluster.sh
</code></pre>
<p data-nodeid="156545">此脚本会自动关闭 master 节点和 slave 节点的所有集群服务。</p>
<p data-nodeid="156546">除了通过 start-cluster.sh、stop-cluster.sh 脚本来批量启动集群服务，还可以在每个节点上手动启动对应服务。例如，在 master 节点启动 jobmanager 服务，可单独执行如下脚本：</p>
<pre class="lang-java" data-nodeid="156547"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> bin]$ /opt/bigdata/flink/current/bin/jobmanager.sh  start
</code></pre>
<p data-nodeid="156548">接着，依次在每个 slave 节点启动 taskmanager 服务，可单独执行如下脚本：</p>
<pre class="lang-java" data-nodeid="156549"><code data-language="java">[hadoop<span class="hljs-meta">@yarnserver</span> bin]$ /opt/bigdata/flink/current/bin/taskmanager.sh  start
</code></pre>
<p data-nodeid="156550">服务启动后，可以查看在 master 节点，jobmanager 服务对应的进程名如下：</p>
<pre class="lang-java" data-nodeid="156551"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> bin]$ jps|grep Standalone
<span class="hljs-number">17026</span> StandaloneSessionClusterEntrypoint
</code></pre>
<p data-nodeid="156552">然后查看 slave 节点，taskmanager 服务对应的进程名如下：</p>
<pre class="lang-java" data-nodeid="156553"><code data-language="java">[hadoop<span class="hljs-meta">@yarnserver</span> bin]$ jps|grep TaskManager
<span class="hljs-number">5099</span> TaskManagerRunner
</code></pre>
<p data-nodeid="156554">至此，Flink 独立集群服务已经启动完毕。</p>
<h4 data-nodeid="156555">3. 提交 job 到 Flink 独立集群</h4>
<p data-nodeid="156556">Flink 集群启动后，它自带了一个 Dashboard 监控页面，访问 <a href="http://nnmaster.cloud:8081" data-nodeid="156769">http://nnmaster.cloud:8081</a>，如下图所示：</p>
<p data-nodeid="156557"><img src="https://s0.lgstatic.com/i/image/M00/1F/B5/Ciqc1F7nMn-AUBcpAAEP4EaKlKU780.png" alt="image" data-nodeid="156773"></p>
<p data-nodeid="156558">在此界面中，可以查看集群可用的 Task Slots、运行的 job、已经完成的 job，以及 Task Managers 节点的状态信息、Job Manager 的配置信息等。此界面使用很简单，这里不过多描述了。</p>
<p data-nodeid="156559">接着，提交一个 flink 任务，执行如下命令：</p>
<pre class="lang-java" data-nodeid="156560"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> conf]$ flink run /opt/bigdata/flink/current/examples/batch/WordCount.jar
</code></pre>
<p data-nodeid="156561">此例子是一个 Flink 自带的 wordcount 统计，如果此命令正常执行，那么则有结果输出，此时登录 Flink 的 Web 界面，可看到如下图已经完成的任务：</p>
<p data-nodeid="156562"><img src="https://s0.lgstatic.com/i/image/M00/1F/B5/Ciqc1F7nMouAbehBAACn_Db-PK0368.png" alt="image" data-nodeid="156779"></p>
<p data-nodeid="156563">除了上面的用法外，还可加上输入源和输出路径，命令执行如下：</p>
<pre class="lang-java" data-nodeid="156564"><code data-language="java">[hadoop<span class="hljs-meta">@nnmaster</span> ~]$ flink run /opt/bigdata/flink/current/examples/batch/WordCount.jar  --input /home/hadoop/demo102.txt  --output  /home/hadoop/count1
</code></pre>
<p data-nodeid="156565">其中，/home/hadoop/demo102.txt、/home/hadoop/count1 都是本地系统路径，需要确保每个 Task Managers 节点都存在 /home/hadoop/demo102.txt 这个文件。这是 Flink 读取本地文件的方法，可以看出，读取本地文件非常麻烦，需要每个节点都要有此文件，简单起见，可以使用 HDFS 上的文件。Flink 也支持读取 HDFS 文件系统上的文件，但这需要一个 Hadoop 的依赖 jar 包支持，在 Flink on Yarn 内容中我会重点介绍。</p>
<h3 data-nodeid="156566">Flink 整合到 Yarn 资源管理器</h3>
<p data-nodeid="156567">Flink on Yarn 模式的原理是依靠 Yarn 来调度 Flink 任务的，这种模式的好处是可以充分利用集群资源，提高集群资源的利用率。目前在企业中使用较多。</p>
<p data-nodeid="156568">需要注意，Flink on Yarn 模式需要依赖部署好的 Hadoop 集群，这点跟 Spark 集成到 Yarn 非常类似。</p>
<h4 data-nodeid="156569">1. Flink On Yarn 的内部实现原理</h4>
<p data-nodeid="160633">下图展示了 Flink On Yarn 的实现逻辑：</p>
<p data-nodeid="160634" class=""><img src="https://s0.lgstatic.com/i/image/M00/1F/C5/CgqCHl7nNf2ATZpjAALuXapYD0o199.png" alt="image" data-nodeid="160638"></p>


<p data-nodeid="156572">每个步骤的执行过程如下：</p>
<ul data-nodeid="156573">
<li data-nodeid="156574">
<p data-nodeid="156575" class="">当启动一个 flink on yarn client 会话时，客户端会首先检查请求的资源是否可用，接着会上传 Flink 的配置和相关 jar 文件到 HDFS；</p>
</li>
<li data-nodeid="156576">
<p data-nodeid="156577">客户端开始向 ResourceManager 申请资源，并请求启动一个 ApplicationMaster（AM）；</p>
</li>
<li data-nodeid="156578">
<p data-nodeid="156579">ResourceManager 选取一个 Yarn 节点启动第一个 Container，然后在此 Container 中启动 ApplicationMaster，同时 JobManager 也会在这个 Container 中启动；</p>
</li>
<li data-nodeid="156580">
<p data-nodeid="156581">AM 开始为 Flink 的 TaskManager 分配 Container；</p>
</li>
<li data-nodeid="156582">
<p data-nodeid="156583">TaskManager 从 HDFS 中下载 JAR 文件和各种配置文件，至此，TaskManager 可以接受任务请求了。</p>
</li>
</ul>
<h4 data-nodeid="156584">2. Flink on Yarn 的两种运行模式</h4>
<p data-nodeid="156585">在 Flink on Yarn 模式下，提交 Flink 任务到 Yarn，分为两种模式，即 Session-Cluster 和 Per-Job-Cluster 模式。</p>
<p data-nodeid="156586">（1）Session-Cluster 模式</p>
<p data-nodeid="162303">使用此模式，需要提前在 Yarn 中初始化一个 Flink 集群，并申请指定的集群资源池，以后的 Flink 任务都会提交到这个资源池下运行。该 Flink 集群会常驻在 Yarn 集群中，除非手工停止。大致原理如下图所示：</p>
<p data-nodeid="162304" class=""><img src="https://s0.lgstatic.com/i/image/M00/1F/B9/Ciqc1F7nNgmARH9kAACweKT2y6U225.png" alt="image" data-nodeid="162308"></p>


<p data-nodeid="156589">此种模式下创建的 Flink 集群会独占资源，不管有没有 Flink 任务在执行，Yarn 上面的其他任务都无法共享使用这些资源。</p>
<p data-nodeid="156590">（2）Per-Job-Cluster 模式</p>
<p data-nodeid="163973">此模式每次提交 Flink 任务，都会创建一个新的 Flink 集群，每个 Flink 任务之间相互独立、互不影响，管理方便。大致原理如下图所示：</p>
<p data-nodeid="163974" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/1F/C5/CgqCHl7nNhCAJXT0AADT0HW3bSs714.png" alt="image" data-nodeid="163978"></p>


<p data-nodeid="156593">此模式下，任务执行完成之后创建的 Flink 集群资源也会随之释放，不会额外占用资源，这种按需使用模式，可以使集群资源利用率达到最大，因此，工作中推荐使用此模式。</p>
<h4 data-nodeid="156594">3. Flink on Yarn 部署架构</h4>
<p data-nodeid="156595">Flink on Yarn 的部署类似于 Spark on Yarn 的部署。首先需要一个部署好的 Hadoop 集群，然后选取一个 Hadoop 节点作为 Flink 的客户端，只需要将此 Flink 程序部署在这个节点即可。</p>
<p data-nodeid="156596">接着，还需要 Flink 提交到 Hadoop 的连接器，其实就是一个 jar 包，将此 jar 文件复制到 Flink 的 lib 目录下即可。这个 jar 包在 flink-1.8 版本之前，是集成到 Flink 安装包里面的，而在 flink-1.8 版本之后，需要下载或者编译对应 Hadoop 版本的 jar 文件。Flink 官方仅提供了基于 Hadoop2.8.3 以及之前的 Hadoop 版本对应的 jar 包，这里我的 Hadoop 版本是 3.2.1，所以需要重新编译，才能生成基于 3.2.1 的 jar 文件，不然兼容性会有问题。</p>
<p data-nodeid="156597">flink-shaded 包含了 Flink 的很多依赖，其中就有 flink-shaded-hadoop-2，<a href="https://archive.apache.org/dist/flink/flink-shaded-9.0/flink-shaded-9.0-src.tgz" data-nodeid="156822">点击这里从 Flink 官网下载版本源码</a>，然后手动编译， 在编译之前，需要修改一下源码，解压源码，进入 flink-shaded-hadoop-2-uber 子目录，找到 pom.xml 文件，在此文件的 dependencyManagement 标签中添加如下内容：</p>
<pre class="lang-java" data-nodeid="156598"><code data-language="java">&lt;dependency&gt;
    &lt;groupId&gt;commons-cli&lt;/groupId&gt;
    &lt;artifactId&gt;commons-cli&lt;/artifactId&gt;
    &lt;version&gt;1.3.1&lt;/version&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="156599">接着，开始编译，操作如下：</p>
<pre class="lang-java" data-nodeid="156600"><code data-language="java">[root@slave002 hadoop]# cd flink-shaded-9.0
[root@slave002 hadoop]# /usr/local/maven/bin/mvn clean install -Dmaven.test.skip=true -Dhadoop.version=3.2.1 -Dmaven.javadoc.skip=true -Dcheckstyle.skip=true
</code></pre>
<p data-nodeid="156601">此编译过程很快，几分钟即可完成，编译完成后，在 flink-shaded-9.0/flink-shaded-hadoop-2-uber 目录下找到 target 子目录，可以发现有一个 flink-shaded-hadoop-2-uber-3.2.1-9.0.jar 文件，此 jar 文件就是我们所需要的。</p>
<p data-nodeid="156602">将此 jar 文件复制到 Flink 安装目录下对应的 lib子目录中。</p>
<h4 data-nodeid="156603">4. Session-Cluster 模式操作实践</h4>
<p data-nodeid="156604">要使用 Session-Cluster 模式，需要在 Hadoop 集群正常运行的前提下，在 Flink 安装目录的 bin 目录下找到一个 yarn-session.sh 脚本，然后启动它，操作如下：</p>
<pre class="lang-java" data-nodeid="156605"><code data-language="java">[hadoop<span class="hljs-meta">@slave002</span> bin]$ ./yarn-session.sh  -s <span class="hljs-number">8</span> -jm <span class="hljs-number">2048</span> -tm <span class="hljs-number">4096</span>  -d
</code></pre>
<p data-nodeid="156606">对上面几个参数介绍如下：</p>
<ul data-nodeid="156607">
<li data-nodeid="156608">
<p data-nodeid="156609">-d，表示让这个 job 在后台独立运行；</p>
</li>
<li data-nodeid="156610">
<p data-nodeid="156611">-s，设置每个 TaskManager 可以使用的 slot 数量；</p>
</li>
<li data-nodeid="156612">
<p data-nodeid="156613">-tm，设置每个 TaskManager 可用的内存，单位是 MB；</p>
</li>
<li data-nodeid="156614">
<p data-nodeid="156615">-jm，设置每个 JobManager 可用的内存，单位是 MB。</p>
</li>
</ul>
<p data-nodeid="156616">注意，上面这些参数可指定，也可不指定。在不指定情况下会默认读取 Flink 配置文件 flink-conf.yaml 中配置的内容；若指定的话，会覆盖 flink-conf.yaml 中的配置，这里需要关注两个默认参数：</p>
<pre class="lang-java" data-nodeid="156617"><code data-language="java">taskmanager.numberOfTaskSlots: <span class="hljs-number">10</span>
parallelism.<span class="hljs-keyword">default</span>: <span class="hljs-number">30</span>
</code></pre>
<p data-nodeid="156618">此参数指定每个 taskmanager 上可使用的 Slot，默认是 10 个，这是设定 taskmanager 的并发执行能力，而 parallelism 是设置 taskmanager 实际使用的并发能力，这个默认值 30 过于大了。如果你的集群中只有三个 taskmanager 节点，每个节点设置 10 个 Slot，那么当 parallelism 设置为 30 时，每个节点将会并发启动 10 个 Slot，此时 Yarn 会出现异常，因为在 Yarn 中，有个配置参数，如下所示：</p>
<pre class="lang-java" data-nodeid="156619"><code data-language="java">yarn.scheduler.maximum-allocation-vcores
</code></pre>
<p data-nodeid="156620">此参数表示最大可申请 CPU 核数，默认值为 8，而根据上面的场景需要在一个节点使用 10 个 CPU 核，因此就出现了资源无法申请到的故障。</p>
<p data-nodeid="156621">因此，在实际使用中，taskmanager.numberOfTaskSlots 的值最好和 Yarn 中最大可申请 CPU 核数保持一致。</p>
<p data-nodeid="156622">回到之前的话题，在上面命令执行完毕后，会有输出日志信息，可以从日志中找到如下图所示的信息：</p>
<p data-nodeid="156623"><img src="https://s0.lgstatic.com/i/image/M00/1F/B6/Ciqc1F7nMvOAFceWAAB-hvMtNfA437.png" alt="image" data-nodeid="156845"></p>
<p data-nodeid="156624">这里提示在 slave001.cloud 启动了一个 web httpd 36873 端口。还可以看到，由于这个 job 是放到后台运行的，所以，最后还给出了几个提示，告诉我们，怎么关闭这个 job，如果要关闭，推荐使用 yarn shell 命令关闭。</p>
<p data-nodeid="156625">根据上图的提示，通过访问 36873 端口，可以打开 yarn 一个内嵌的 Flink 的 Dashboard，如下图所示：</p>
<p data-nodeid="156626"><img src="https://s0.lgstatic.com/i/image/M00/1F/C1/CgqCHl7nMvyAU3XjAADFzBu0bTU689.png" alt="image" data-nodeid="156850"></p>
<p data-nodeid="156627">我们可以通过访问此页面来查看 Flink 任务的运行状态。此时，在 Yarn 的 8080 端口界面下，也可以发现有任务运行，如下图所示：</p>
<p data-nodeid="156628"><img src="https://s0.lgstatic.com/i/image/M00/1F/C1/CgqCHl7nMwSASX5OAACTLV3fx_Q143.png" alt="image" data-nodeid="156854"></p>
<p data-nodeid="156629">可以看出，这个 Session-Cluster 模式在 Yarn 下相当于启动了一个任务，任务的名称为 Flink session cluster，任务类型为 apache flink，一直处于运行状态。此任务会常驻在 Yarn 中，现在，可以在这个 Session-Cluster 模式下运行一个 flink 任务，执行如下命令：</p>
<pre class="lang-java" data-nodeid="156630"><code data-language="java">[hadoop<span class="hljs-meta">@slave002</span> bin]$ ./flink run /opt/bigdata/flink/current/examples/batch/WordCount.jar --input  hdfs:<span class="hljs-comment">//bigdata/logs/demo103.txt  --output  hdfs://bigdata/logs/count</span>
</code></pre>
<p data-nodeid="156631">在此任务执行过程中，访问 http://slave001.cloud:36873 页面，即可看到此任务的执行状态，如下图所示：</p>
<p data-nodeid="156632"><img src="https://s0.lgstatic.com/i/image/M00/1F/C2/CgqCHl7nMw-AAM2oAADY6YPrTFk158.png" alt="image" data-nodeid="156859"></p>
<p data-nodeid="156633">此界面显示了目前正在运行的任务，以及 Flink 剩余的资源、已经使用的资源等信息。</p>
<h4 data-nodeid="156634">5. Pre-Job-Cluster 模式操作实践</h4>
<p data-nodeid="156635">这种模式下不需要先启动 yarn-session。因此，我们需要把前面启动的 yarn-session 集群先停止，停止的命令如下:</p>
<pre class="lang-java" data-nodeid="156636"><code data-language="java">[hadoop<span class="hljs-meta">@slave002</span> bin]$ yarn application -kill application_1591691583827_0010
</code></pre>
<p data-nodeid="156637">接着，在 Hadoop 集群所有服务（HDFS/YARN）运行状态正常的情况下提交如下 job 到 Yarn 集群：</p>
<pre class="lang-java" data-nodeid="156638"><code data-language="java">[hadoop<span class="hljs-meta">@slave002</span> bin]$./flink run -m yarn-cluster -ys <span class="hljs-number">4</span>   -yjm <span class="hljs-number">2048</span>  -ytm <span class="hljs-number">3072</span> ../examples/batch/WordCount.jar
</code></pre>
<p data-nodeid="156639">这里的三个参数含义如下：</p>
<ul data-nodeid="156640">
<li data-nodeid="156641">
<p data-nodeid="156642">-ys，设置每个 TaskManager 可以使用的 slot 数量；</p>
</li>
<li data-nodeid="156643">
<p data-nodeid="156644">-ytm，设置每个 TaskManager 可用的内存，单位是 MB；</p>
</li>
<li data-nodeid="156645">
<p data-nodeid="156646">-yjm，设置每个 JobManager 可用的内存，单位是 MB。</p>
</li>
</ul>
<p data-nodeid="156647">此任务提交后，在 Yarn 的 8080 界面下，可以看到提交的 Flink 任务，如下图所示：</p>
<p data-nodeid="156648"><img src="https://s0.lgstatic.com/i/image/M00/1F/B6/Ciqc1F7nMyOAArfaAABNBvmrEKc414.png" alt="image" data-nodeid="156877"></p>
<p data-nodeid="156649">在 Pre-Job-Cluster 模式下，Flink 任务名称变成了 Flink pre-Job Cluster，此任务运行结束后，任务自动退出，占用资源自动释放。</p>
<h3 data-nodeid="156650">总结</h3>
<p data-nodeid="156651">本课时主要讲述了 Flink 的应用架构、独立集群的使用以及 Flink on Yarn 模式的使用，其中，Flink on Yarn 模式的使用是本课时讲述的重点，在企业实际应用中，都是以 Yarn 来作为统一的资源管理器，在大数据快速发展和应用的今天，你应该尝试下 Flink 给企业带来的便利和高效。</p>

---

### 精选评论


