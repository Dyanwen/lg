<p data-nodeid="1" class="">本课时主要介绍 Hadoop 发行版选型以及伪分布式平台的构建。</p>
<h3 data-nodeid="2">Hadoop 发行版介绍与选择</h3>
<p data-nodeid="3">到目前为止，你应该初步了解了大数据以及 Hadoop 相关的概念了。本课时我将介绍 Hadoop 如何快速使用，由于 Hadoop 平台的构建过程相当复杂，它涉及系统、网络、存储、配置与调优，但为了能让你尽快尝鲜体验一下 Hadoop 的功能和特性，我们先一起构建一个伪分布式 Hadoop 集群，也就是一个假的 Hadoop 集群，麻雀虽小，但五脏俱全。</p>
<p data-nodeid="4">伪分布式 Hadoop 集群能够实现 Hadoop 的所有功能，并且部署简单，因此非常适合新手进行学习、开发、测试等工作。</p>
<h4 data-nodeid="5">Hadoop 有哪些发行版</h4>
<p data-nodeid="6">与 Linux 有众多发行版类似，Hadoop 也有很多发行版本，但基本上分为两类，即<strong data-nodeid="184">开源社区版</strong>和<strong data-nodeid="185">商业付费版</strong>。社区版是指由 Apache 软件基金会维护的版本，是官方维护的版本体系；商业版付费版是指由第三方商业公司在社区版 Hadoop 基础上进行了一些修改、整合以及各个服务组件兼容性测试而发行的稳定版本，比较著名的有 Cloudera 的 CDP、CDH、Hortonworks 的 Hortonworks Data Platform（HDP）、mapR 等。</p>
<p data-nodeid="7">在这些商业 Hadoop 发行版中，为了吸引用户的使用，厂商也提供了一些开源产品作为诱饵，比如 Cloudera 的 CDH 发行版、Hortonworks 的 HDP 发行版等，所以，目前而言，不收费的 Hadoop 版本主要有三个，即 Apache Hadoop、Cloudera 的 CDH 版本、Hortonworks 的 HDP。</p>
<p data-nodeid="8">经过多年的发展，Cloudera 的 CDH 版本和 Hortonworks 的 HDP 版本在大数据开源社区互为竞争，两分天下，占据了国内、外 90% 以上的大数据市场，但随着公有云市场趋于成熟，很多云厂商在云端也提供了 Hadoop 服务，比如亚马逊的 Elastic MapReduce（EMR）、Microsoft Azure Hadoop、阿里云 E-MapReduce（Elastic MapReduce，EMR）等，这些基于云的大数据服务抢走了 Cloudera 和 Hortonworks 的大部分客户，所谓天下大势，分久必合，合久必分，最终，Cloudera 和 Hortonworks 从竞争走到了一起，他们相爱了。</p>
<p data-nodeid="9">下面我们来聊下常用的三个 Hadoop 发行版本，看看他们的产品特点以及如何选型。</p>
<h4 data-nodeid="10">Apache Hadoop 发行版本</h4>
<p data-nodeid="11"><a href="http://hadoop.apache.org/" data-nodeid="192">Apache Hadoop</a> 是最原始的 Hadoop 发行版本，目前总共发行了三个大版本，即 Hadoop1.x、Hadoop2.x、Hadoop3.x，每个版本的功能特性如下表所示：</p>
<p data-nodeid="12"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/CgqCHl6qvImAddQkAADd9uJZm7c786.png" alt="1.png" data-nodeid="196"></p>
<p data-nodeid="13">Apache Hadoop 发行版提供源码包和二进制包两种形式下载，对我们来说，下载二进制包更加方便，点击<a href="https://archive.apache.org/dist/hadoop/common/" data-nodeid="200">这里</a>获得下载。</p>
<h4 data-nodeid="14">Hortonworks Hadoop 发行版</h4>
<p data-nodeid="15">Hortonworks 的主打产品是 HDP，同样是 100% 开源的产品，它最接近 Apache Hadoop 的版本，除此之外，HDP 还包含了 Ambari，这是一款开源的 Hadoop 管理系统。它可以实现统一部署、自动配置、自动化扩容、实时状态监控等，是个功能完备的大数据运维管理平台。</p>
<p data-nodeid="16">在使用 HDP 发行版时，可以通过 Ambari 管理功能，实现 Hadoop 的快速安装和部署，并且对大数据平台的运维也有很大帮助，可以说 Ambari 实现了与 HDP 的无缝整合。</p>
<p data-nodeid="17">HDP 至今也发行了三个版本，即 HDP1.x、HDP2.x 和 HDP3.x，跟 Apache Hadoop 发行的大版本遥相呼应，而 HDP 发行版的安装是基于 Ambari 实现的，通过 HDP 提供的 rpm 文件，可以在 Ambari 平台实现自动化的安装和扩容，后面会做详细介绍。</p>
<h4 data-nodeid="18">Cloudera Hadoop 发行版</h4>
<p data-nodeid="19">Cloudera 是最早将 Hadoop 商用的公司，目前旗下的产品主要有 CDH、Cloudera Manager、Cloudera Data Platform（CDP）等，下表简单介绍了这些产品的特点。</p>
<p data-nodeid="20"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/CgqCHl6qvLqAFucIAADw0F70qQU286.png" alt="2.png" data-nodeid="210"></p>
<p data-nodeid="21">CDH 支持 yum/apt 包、RPM 包、tarball 包、Cloudera Manager 四种方式安装，但在最新版本 CDH6 中已经不提供 tarball 方式了，这也是 Cloudera 进行产品整合的一个信号。</p>
<p data-nodeid="22">Cloudera 在宣布与 Hortonworks 合并后，他们使用了类似红帽公司的开源战略，提供了订阅机制来收费，同时为开发人员和试用提供了无支援的免费版本，并向商业用户提供订阅付费的版本。至此，Cloudera 成为全球第二大开源公司（红帽第一）。</p>
<p data-nodeid="23">看到这里，也许会担心，这么好的开源版本，后面是不是就不能免费使用了呢，答案是否定的，Cloudera 承诺 CDH 和 HDP 平台将可以继续使用，直到 2022 年。</p>
<h4 data-nodeid="24">如何选择发行版</h4>
<p data-nodeid="25">作为用户，应该如何选择呢，经过多年对 Hadoop 的使用，我的建议是：对于初学入门的话，建议选择 Apache Hadoop 版本最好，因为它的社区活跃、文档、资料详实。而如果要在企业生产环境下使用的话，建议需要考虑以下几个因素：</p>
<ul data-nodeid="26">
<li data-nodeid="27">
<p data-nodeid="28">是否为开源产品（是否免费），这点很重要；</p>
</li>
<li data-nodeid="29">
<p data-nodeid="30">是否有稳定的发行版本，开发版是不能用在生产上的；</p>
</li>
<li data-nodeid="31">
<p data-nodeid="32">是否已经接受过实践的检验，看看是否有大公司在用（自己不能当小白鼠）；</p>
</li>
<li data-nodeid="33">
<p data-nodeid="34">是否有活跃的社区支持、充足的资料，因为遇到问题，我们可以通过社区、搜索等网络资源来解决问题。</p>
</li>
</ul>
<p data-nodeid="35">在国内大型互联网企业中，使用较多的是 CDH 或 HDP 发行版本，个人推荐采用 HDP 发行版本，原因是部署简单、性能稳定。</p>
<h3 data-nodeid="36">伪分布式安装 Hadoop 集群</h3>
<p data-nodeid="37">为了让你快速了解 Hadoop 功能和用途，先通过伪分布式来安装一个 <strong data-nodeid="227">Hadoop 集群</strong>，这里采用 Apache Hadoop 发行版的二进制包进行快速部署。完全分布式 Hadoop 集群后面将会进行更深入的介绍。</p>
<h4 data-nodeid="38">安装规划</h4>
<p data-nodeid="39">伪分布式安装 Hadoop 只需要一台机器，硬件配置最低为 4 核 CPU、8G 内存即可，我们采用 Hadoop-3.2.1 版本，此版本要求 Java 版本至少是 JDK8，这里以 JDK1.8.0_171、CentOS7.6 为例进行介绍。根据运维经验以及后续的升级、自动化运维需要，将 Hadoop 程序安装到 /opt/hadoop 目录下，Hadoop 配置文件放到 /etc/hadoop 目录下。</p>
<h4 data-nodeid="40">安装过程</h4>
<p data-nodeid="41">点击<a href="https://mirror.bit.edu.cn/apache/hadoop/core/" data-nodeid="236">这里</a>下载 Apache Hadoop 发行版本的 hadoop-3.2.1.tar.gz 二进制版本文件，其安装非常简单，只需解压文件即可完成安装，操作过程如下：</p>
<pre class="lang-java" data-nodeid="42"><code data-language="java">[root@hadoop3server hadoop]#useradd hadoop
[root@hadoop3server ~]#mkdir /opt/hadoop
[root@hadoop3server ~]#cd /opt/hadoop
[root@hadoop3server hadoop]#tar zxvf hadoop-3.2.1.tar.gz
[root@hadoop3server hadoop]#ln -s hadoop-3.2.1 current
[root@hadoop3server hadoop]#chown -R hadoop:hadoop /opt/hadoop
</code></pre>
<p data-nodeid="43"><strong data-nodeid="242">注意</strong>，将解压开的 hadoop-3.2.1.tar.gz 目录软链接到 current 是为了后续运维方便，因为可能涉及 Hadoop 版本升级、自动化运维等操作，这样设置后，可以大大减轻运维工作量。</p>
<p data-nodeid="44">Hadoop 程序安装完成后，还需要拷贝配置文件到 /etc/hadoop 目录下，执行如下操作：</p>
<pre class="lang-java" data-nodeid="45"><code data-language="java">[root@hadoop3server ~]#mkdir /etc/hadoop
[root@hadoop3server hadoop]#cp -r /opt/hadoop/current/etc/hadoop /etc/hadoop/conf
[root@hadoop3server hadoop]# chown -R hadoop:hadoop  /etc/hadoop
</code></pre>
<p data-nodeid="46">这样，就将配置文件放到 /etc/hadoop/conf 目录下了。</p>
<p data-nodeid="47">接着，还需要安装一个 JDK，这里使用的是 JDK 1.8.0_171，将其安装到 /usr/java 目录下，操作过程如下：</p>
<pre class="lang-java" data-nodeid="48"><code data-language="java">[root@hadoop3server ~]#mkdir /usr/java
[root@hadoop3server ~]#cd /usr/java
[root@hadoop3server java]#tar zxvf jdk-8u171-linux-x64.tar.gz
[root@hadoop3server java]#ln -s jdk1.8.0_171 default
</code></pre>
<p data-nodeid="49">这个操作过程的最后一步，做这个软连接，也是为了后续运维自动化配置、升级方便。</p>
<p data-nodeid="50">最后一步，还需要创建一个 Hadoop 用户，然后设置 Hadoop 用户的环境变量，配置如下：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="97425"><code data-language="java">[root@hadoop3server ~]#useradd hadoop
[root@hadoop3server ~]# more /home/hadoop/.bashrc 
# .bashrc
# Source global definitions
if [ -f /etc/bashrc ]; then
        . /etc/bashrc
fi

# User specific aliases and functions
export JAVA_HOME=/usr/java/default
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
export HADOOP_HOME=/opt/hadoop/current
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_YARN_HOME=${HADOOP_HOME}
export CATALINA_BASE=${HTTPFS_CATALINA_HOME}
export HADOOP_CONF_DIR=/etc/hadoop/conf
export HTTPFS_CONFIG=/etc/hadoop/conf
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
</code></pre>
































































































































































<p data-nodeid="52">这里创建的 Hadoop 用户，就是以后管理 Hadoop 平台的管理员用户，<strong data-nodeid="255">所有对 Hadoop 的管理操作都需要通过这个用户来完成</strong>，这一点需注意。</p>
<p data-nodeid="53">另外，在配置的环境变量中，以下两个要特别注意，如果没有配置或者配置错误，将导致某些服务无法启动：</p>
<ul data-nodeid="54">
<li data-nodeid="55">
<p data-nodeid="56">HADOOP_HOME 是指定 Hadoop 安装程序的目录</p>
</li>
<li data-nodeid="57">
<p data-nodeid="58">HADOOP_CONF_DIR 是指定 Hadoop 配置文件目录</p>
</li>
</ul>
<p data-nodeid="59">到这里为止，Hadoop 已经基本安装完成了，是不是很简单！</p>
<h4 data-nodeid="60">配置 Hadoop 参数</h4>
<p data-nodeid="61">Hadoop 安装完成后，先来了解一下其安装目录下几个重要的目录和文件，这里将 Hadoop 安装在了 /opt/hadoop/current 目录下，打开这个目录，需要掌握的几个目录如下表所示：</p>
<p data-nodeid="62"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/CgqCHl6qvUmABT4CAAEJKFoZOco427.png" alt="3.png" data-nodeid="270"></p>
<p data-nodeid="63">了解完目录的功能后，就开始进行配置操作了，Hadoop 的配置相当复杂，不过这些是后面要讲的内容。而在伪分布模式下，仅仅需要修改一个配置文件即可，该文件是 core-site.xml，此文件目前位于 /etc/hadoop/conf 目录下，在此文件 标签下增加如下内容：</p>
<pre class="lang-html" data-nodeid="64"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>fs.defaultFS<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>hdfs://hadoop3server<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
</code></pre>
<p data-nodeid="65">其中，fs.defaultFS 属性描述的是访问 HDFS 文件系统的 URI 加一个 RPC 端口， 不加端口的话，默认是 8020。另外，hadoop3server 可以是服务器的主机名，也可以是任意字符，但都需要将此标识在服务器的 /etc/hosts 进行解析，也就是添加如下内容：</p>
<pre class="lang-java" data-nodeid="66"><code data-language="java"><span class="hljs-number">172.16</span>.<span class="hljs-number">213.232</span>  hadoop3server
</code></pre>
<p data-nodeid="67">这里的 172.16.213.232 就是安装 Hadoop 软件的服务器 IP 地址。</p>
<h4 data-nodeid="68">启动 Hadoop 服务</h4>
<p data-nodeid="69">配置操作完成后，下面就可以启动 Hadoop 服务了，虽然是伪分布模式，但 Hadoop 所有的服务都必须要启动，需要启动的服务有如下几个。</p>
<p data-nodeid="70"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/CgqCHl6qvYyAO4ovAAEVE9kN8xY217.png" alt="4.png" data-nodeid="278"></p>
<p data-nodeid="71">服务的功能和用途，先介绍这么多，后面将会进行更深入的阐述。接下来，要启动 Hadoop 集群的服务，必须以 Hadoop 用户来执行，并且每个服务的启动是有先后顺序的，下面依次启动每个服务。</p>
<p data-nodeid="72">（1）启动 NameNode 服务</p>
<p data-nodeid="73">首先需要对 NameNode 进行格式化，命令如下：</p>
<pre class="lang-java" data-nodeid="74"><code data-language="java">[root@hadoop3server ~]#su - hadoop
[hadoop@hadoop3server ~]$ cd /opt/hadoop/current/bin
[hadoop@hadoop3server bin]$ hdfs  namenode -format
</code></pre>
<p data-nodeid="75">然后就可以启动 NameNode 服务了，操作过程如下：</p>
<pre class="lang-java" data-nodeid="76"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> conf]$ hdfs --daemon start namenode
[hadoop<span class="hljs-meta">@hadoop3server</span> conf]$ jps|grep NameNode
<span class="hljs-number">27319</span> NameNode
</code></pre>
<p data-nodeid="77">通过 jps 命令查看 NameNode 进程是否正常启动，如果无法正常启动，可以查看 NameNode 启动日志文件，检查是否有异常信息抛出，这里的日志文件路径是：/opt/hadoop/current/logs/hadoop-hadoop-namenode-hadoop3server.log。</p>
<p data-nodeid="78">NameNode 启动完成后，就可以通过 Web 页面查看状态了，默认会启动一个 http 端口 9870，可通过访问地址：http://172.16.213.232:9870 查看 NameNode 服务状态，如下图所示：</p>
<p data-nodeid="79"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/CgqCHl6qvauAJDJBAAEI3H31odc585.png" alt="image1.png" data-nodeid="287"></p>
<p data-nodeid="80">在上图中，红框标注的几个重点信息需要关注，第一个是 Hadoop 中 namenode 的访问地址为 hdfs://hadoop3server:8020，这是我们在配置文件中指定过的；另外还有 Hadoop 的版本、运行模式、容量、“Live node”及“Dead node”，下面逐个解释。</p>
<p data-nodeid="81"><strong data-nodeid="293">运行模式</strong>显示“Safe mode is ON”，这表示目前 namenode 处于安全模式下了，为什么呢，其实图中已经说明原因了，Namenode 在启动时，会检查 DataNode 的状态，如果 DataNode 上报的 block 个数达到了元数据记录的 block 个数的 0.999 倍才可以离开安全模式，否则一直运行在安全模式。安全模式也叫只读模式，此模式下，对 HDFS 上的数据无法进行写操作。因为现在还没启动 DataNode 服务，所以肯定是处于安全模式下。</p>
<p data-nodeid="82"><strong data-nodeid="298">HDFS 容量</strong>，Configured Capacity 目前显示为 0，这也是因为还没启动 DataNode 服务导致的，等启动后，应该就有容量显示了。</p>
<p data-nodeid="83">“<strong data-nodeid="308">Live node</strong>”及“<strong data-nodeid="309">Dead node</strong>”分别显示目前集群中活跃的 DataNode 节点和故障（死）DataNode 节点，运维经常通过监控这个页面中“Dead node”的值来判断集群是否出现异常。</p>
<p data-nodeid="84">（2）启动 secondarynamenode 服务</p>
<p data-nodeid="85">在 NameNode 服务启动完成后，就可以启动 secondarynamenode 服务了，直接执行如下命令：</p>
<pre class="lang-java" data-nodeid="86"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hdfs --daemon start secondarynamenode
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps|grep SecondaryNameNode
<span class="hljs-number">29705</span> SecondaryNameNode
</code></pre>
<p data-nodeid="87">与 NameNode 类似，如果无法启动 secondarynamenode 进程，可以通过 /opt/hadoop/current/logs/hadoop-hadoop-secondarynamenode-hadoop3server.log 文件检查 secondarynamenode 启动日志中是否存在异常。</p>
<p data-nodeid="88">（3）启动 DataNode 服务</p>
<p data-nodeid="89">现在是时候启动 DataNode 服务了，直接执行如下命令：</p>
<pre class="lang-java" data-nodeid="90"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hdfs --daemon start datanode
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps|grep DataNode
<span class="hljs-number">3876</span> DataNode
</code></pre>
<p data-nodeid="91">如果无法启动，可通过查看 /opt/hadoop/current/logs/hadoop-hadoop-datanode-hadoop3server.log 文件检查 datanode 启动过程是否存在异常。</p>
<p data-nodeid="92">到这里为止，分布式文件系统 HDFS 服务已经启动完成，可以对 HDFS 文件系统进行读、写操作了。现在再次通过 http://172.16.213.232:9870 查看 NameNode 服务状态页面，如图所示：</p>
<p data-nodeid="93"><img src="https://s0.lgstatic.com/i/image/M00/00/F4/Ciqc1F6qvdiAcx1fAAEELGkVn3g251.png" alt="image2.png" data-nodeid="319"></p>
<p data-nodeid="94">从图中可以看出，HDFS 集群中安全模式已经关闭，并且集群容量和活跃节点已经有数据了，这是因为 datanode 服务已经正常启动了。</p>
<p data-nodeid="95">（4）启动 ResourceManager 服务</p>
<p data-nodeid="96">接下来，还需要启动分布式计算服务，首先启动的是 ResourceManager，启动方式如下：</p>
<pre class="lang-java" data-nodeid="97"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ yarn --daemon start resourcemanager
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps|grep ResourceManager
<span class="hljs-number">4726</span> ResourceManager
</code></pre>
<p data-nodeid="98">注意，启动 resourcemanager 服务的命令变成了 yarn，而不是 hdfs，记住这个细节。</p>
<p data-nodeid="99">同理，如果 ResourceManager 进程无法启动，可以通过检查 /opt/hadoop/current/logs/hadoop-hadoop-resourcemanager-hadoop3server.log 日志文件来排查 ResourceManager 启动问题。</p>
<p data-nodeid="100">ResourceManager 服务启动后，会默认启动一个 http 端口 8088，可通过访问 http://172.16.213.232:8088 查看 ResourceManager 的 Web 状态页面，如下图所示：</p>
<p data-nodeid="101"><img src="https://s0.lgstatic.com/i/image/M00/00/F5/Ciqc1F6qvfKAGW9fAAGSrA9yJ0s930.png" alt="image3.png" data-nodeid="328"></p>
<p data-nodeid="102">在上图中，需要重点关注的是 ResourceManager 中可用的内存资源、CPU 资源数及活跃节点数，目前看来，这些数据都是 0，是因为还没有 NodeManager 服务启动。</p>
<p data-nodeid="103">（5）启动 NodeManager 服务</p>
<p data-nodeid="104">在启动完成 ResourceManager 服务后，就可以启动 NodeManager 服务了，操作过程如下：</p>
<pre class="lang-java" data-nodeid="105"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ yarn --daemon start nodemanager
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps|grep NodeManager
<span class="hljs-number">8853</span> NodeManager
</code></pre>
<p data-nodeid="106">如果有异常，可通过检查 /opt/hadoop/current/logs/hadoop-hadoop-nodemanager-hadoop3server.log 文件来排查 NodeManager 问题。</p>
<p data-nodeid="107">（6）启动 Jobhistoryserver 服务</p>
<p data-nodeid="108">等待 ResourceManager 和 NodeManager 服务启动完毕后，最后还需要启动一个 Jobhistoryserver 服务，操作过程如下：</p>
<pre class="lang-java" data-nodeid="109"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ mapred  --daemon start historyserver
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps|grep JobHistoryServer
<span class="hljs-number">1027</span> JobHistoryServer
</code></pre>
<p data-nodeid="110"><strong data-nodeid="339">注意</strong>，启动 Jobhistoryserver 服务的命令变成了 mapred，而非 yarn。这是因为 Jobhistoryserver 服务是基于 MapReduce 的，Jobhistoryserver 服务启动后，会运行一个 http 端口，默认端口号是 19888，可以通过访问此端口查看每个任务的历史运行情况，如下图所示：</p>
<p data-nodeid="111"><img src="https://s0.lgstatic.com/i/image/M00/00/F5/CgqCHl6qvhSAZhOZAAEaRsNStmg076.png" alt="image4.png" data-nodeid="342"></p>
<p data-nodeid="112">至此，Hadoop 伪分布式已经运行起来了，可通过 jps 命令查看各个进程的启动信息：</p>
<pre class="lang-java" data-nodeid="113"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ jps
<span class="hljs-number">12288</span> DataNode
<span class="hljs-number">1027</span> JobHistoryServer
<span class="hljs-number">11333</span> NameNode
<span class="hljs-number">1158</span> Jps
<span class="hljs-number">29705</span> SecondaryNameNode
<span class="hljs-number">18634</span> NodeManager
<span class="hljs-number">19357</span> ResourceManager
</code></pre>
<p data-nodeid="114">不出意外的话，会输出每个服务的进程名信息，这些输出表明 Hadoop 服务都已经正常启动了。现在，可以在 Hadoop 下愉快的玩耍了。</p>
<h4 data-nodeid="115">运用 Hadoop HDFS 命令进行分布式存储</h4>
<p data-nodeid="116">Hadoop 的 HDFS 是一个分布式文件系统，要对 HDFS 进行操作，需要执行 HDFS Shell，跟 Linux 命令很类似，因此，只要熟悉 Linux 命令，可以很快掌握 HDFS Shell 的操作。</p>
<p data-nodeid="117">下面看几个例子，你就能迅速知道 HDFS Shell 的用法， 需要注意，执行 HDFS Shell 建议在 Hadoop 用户或其他普用用户下执行。</p>
<p data-nodeid="118">（1）查看 hdfs 根目录数据，可通过如下命令：</p>
<pre class="lang-java" data-nodeid="119"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -ls /
</code></pre>
<p data-nodeid="120">通过这个命令的输出可知，刚刚创建起来的 HDFS 文件系统是没有任何数据的，不过可以自己创建文件或目录。</p>
<p data-nodeid="121">（2）在 hdfs 根目录创建一个 logs 目录，可执行如下命令：</p>
<pre class="lang-java" data-nodeid="122"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -mkdir /logs
</code></pre>
<p data-nodeid="123">（3）从本地上传一个文件到 hdfs 的 /logs 目录下，可执行如下命令：</p>
<pre class="lang-java" data-nodeid="124"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -put /data/test.txt /logs
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -put /data/db.gz  /logs
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -ls /logs
Found <span class="hljs-number">2</span> items
-rw-r--r--   <span class="hljs-number">3</span> hadoop supergroup     <span class="hljs-number">150569</span> <span class="hljs-number">2020</span>-<span class="hljs-number">03</span>-<span class="hljs-number">19</span> <span class="hljs-number">07</span>:<span class="hljs-number">11</span> /logs/test.txt
-rw-r--r--   <span class="hljs-number">3</span> hadoop supergroup         <span class="hljs-number">95</span> <span class="hljs-number">2020</span>-<span class="hljs-number">03</span>-<span class="hljs-number">24</span> <span class="hljs-number">05</span>:<span class="hljs-number">11</span> /logs/db.gz
</code></pre>
<p data-nodeid="125"><strong data-nodeid="356">注意</strong>，这里的 /data/test.txt 及 db.gz 是操作系统下的一个本地文件，通过执行 put 命令，可以看到，文件已经从本地磁盘传到 HDFS 上了。</p>
<p data-nodeid="126">（4）要查看 hdfs 中一个文本文件的内容，可执行如下命令：</p>
<pre class="lang-java" data-nodeid="127"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -cat /logs/test.txt
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -text /logs/db.gz
</code></pre>
<p data-nodeid="128">可以看到，在 HDFS 上的压缩文件通过“-text”参数也能直接查看，因为默认情况下 Hadoop 会自动识别常见的压缩格式。</p>
<p data-nodeid="129">（5）删除 hdfs 上一个文件，可执行如下命令：</p>
<pre class="lang-java" data-nodeid="130"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs  -rm  -r /logs/test.txt
</code></pre>
<p data-nodeid="131">注意，HDFS 上面的文件，只能创建和删除，无法更新一个存在的文件，如果要更新 HDFS 上的文件，需要先删除这个文件，然后提交最新的文件即可。除上面介绍的命令之外，HDFS Shell 还有很多常用的命令，这个在后面会有专门课时来讲解。</p>
<h4 data-nodeid="132">在 Hadoop 中运行 MapreDuce 程序</h4>
<p data-nodeid="133">要体验 Hadoop 的分布式计算功能，这里借用 Hadoop 安装包中附带的一个 mapreduce 的 demo 程序，做个简单的 MR 计算。</p>
<p data-nodeid="134">这个 demo 程序位于 $HADOOP_HOME/share/hadoop/mapreduce 路径下，这个环境下的路径为 /opt/hadoop/current/share/hadoop/mapreduce，在此目录下找到一个名为 hadoop-mapreduce-examples-3.2.1.jar 的 jar 文件，有了这个文件下面的操作就简单多了。</p>
<p data-nodeid="135">单词计数是最简单也是最能体现 MapReduce 思想的程序之一，可以称为 MapReduce 版“Hello World”，hadoop-mapreduce-examples-3.2.1.jar 文件中包含了一个 wordcount 功能，它主要功能是用来统计一系列文本文件中每个单词出现的次数。下面开始执行分析计算。</p>
<p data-nodeid="136">（1）创建一个新文件</p>
<p data-nodeid="137">创建一个测试文件 demo.txt，内容如下:</p>
<pre class="lang-java" data-nodeid="138"><code data-language="java">Linux Unix windows
hadoop Linux spark
hive hadoop Unix
MapReduce hadoop  Linux hive
windows hadoop spark
</code></pre>
<p data-nodeid="139">（2）将创建的文件存入 HDFS</p>
<pre class="lang-java" data-nodeid="140"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -mkdir /demo
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -put /opt/demo.txt /demo
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -ls /demo
Found <span class="hljs-number">1</span> items
-rw-r--r--   <span class="hljs-number">3</span> hadoop supergroup        <span class="hljs-number">105</span> <span class="hljs-number">2020</span>-<span class="hljs-number">03</span>-<span class="hljs-number">24</span> <span class="hljs-number">06</span>:<span class="hljs-number">02</span> /demo/demo.txt
</code></pre>
<p data-nodeid="141">这里在 HDFS 上创建了一个目录 /demo，然后将刚才创建好的本地文件 put 到 HDFS 上，这里举例是一个文件，如果要统计多个文件内容，将多个文件都上传到 HDFS 的 /demo 目录即可。</p>
<p data-nodeid="142">（3）执行分析计算任务</p>
<p data-nodeid="143">下面开始执行分析任务：</p>
<pre class="lang-java" data-nodeid="144"><code data-language="java">[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop jar /opt/hadoop/current/share/hadoop/mapreduce/hadoop-mapreduce-examples-<span class="hljs-number">3.2</span>.<span class="hljs-number">1.</span>jar  wordcount /demo  /output
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -ls /output
Found <span class="hljs-number">2</span> items
-rw-r--r--   <span class="hljs-number">3</span> hadoop supergroup          <span class="hljs-number">0</span> <span class="hljs-number">2020</span>-<span class="hljs-number">03</span>-<span class="hljs-number">24</span> <span class="hljs-number">06</span>:<span class="hljs-number">05</span> /output/_SUCCESS
-rw-r--r--   <span class="hljs-number">3</span> hadoop supergroup         <span class="hljs-number">61</span> <span class="hljs-number">2020</span>-<span class="hljs-number">03</span>-<span class="hljs-number">24</span> <span class="hljs-number">06</span>:<span class="hljs-number">05</span> /output/part-r-<span class="hljs-number">00000</span>
[hadoop<span class="hljs-meta">@hadoop3server</span> ~]$ hadoop fs -text /output/part-r-<span class="hljs-number">00000</span>
Linux   <span class="hljs-number">3</span>
MapReduce     <span class="hljs-number">1</span>
Unix    <span class="hljs-number">2</span>
hadoop  <span class="hljs-number">4</span>
hive    <span class="hljs-number">2</span>
spark   <span class="hljs-number">2</span>
windows <span class="hljs-number">2</span>
</code></pre>
<p data-nodeid="145">在上面的操作中，通过执行“hadoop jar”后面跟上 jar 包示例文件，并给出执行的功能是 wordcount，即可完成任务的执行，请注意，最后的两个路径都是 HDFS 上的路径，第一个路径是分析读取文件的目录，必须存在；第二个路径是分析任务输出结果的存放路径，必须不存在，分析任务会自动创建这个目录。</p>
<p data-nodeid="146">任务执行完毕后，可以查看 /output 目录下有两个文件，其中：</p>
<ul data-nodeid="147">
<li data-nodeid="148">
<p data-nodeid="149">_SUCCESS，任务完成标识，表示执行成功；</p>
</li>
<li data-nodeid="150">
<p data-nodeid="151">part-r-00000，表示输出文件名，常见的名称有 part-m-00000、part-r-00001，其中，带 m 标识的文件是 mapper 输出，带 r 标识的文件是 reduce 输出的，00000 为 job 任务编号，part-r-00000 整个文件为结果输出文件。</p>
</li>
</ul>
<p data-nodeid="152">通过查看 part-r-00000 文件内容，可以看到 wordcount 的统计结果。左边一列是统计的单词，右边一列是在文件中单词出现的次数。</p>
<p data-nodeid="153">（4）在 ResourceManager 的 Web 页面展示运行任务</p>
<p data-nodeid="154">细心的你可能已经发现了，上面在命令行执行的 wordcount 统计任务虽然最后显示是执行成功了，统计结果也正常，但是在 ResourceManager 的 Web 页面并没有显示出来。</p>
<p data-nodeid="155">究其原因，其实很简单：这是因为那个 mapreduce 任务并没有真正提交到 yarn 上来，因为默认 mapreduce 的运行环境是 local（本地），要让 mapreduce 在 yarn 上运行，需要做几个参数配置就行了。</p>
<p data-nodeid="156">需要修改的配置文件有两个，即 mapred-site.xml 和 yarn-site.xml，在你的配置文件目录，找到它们。</p>
<p data-nodeid="157">打开 mapred-site.xml 文件，在 标签内添加如下内容：</p>
<pre class="lang-html" data-nodeid="158"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>mapreduce.framework.name<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>yarn<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>

<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.app.mapreduce.am.env<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>HADOOP_MAPRED_HOME=${HADOOP_HOME}<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>

<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>mapreduce.map.env<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>HADOOP_MAPRED_HOME=${HADOOP_HOME}<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>

<span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>mapreduce.reduce.env<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>HADOOP_MAPRED_HOME=${HADOOP_HOME}<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
</code></pre>
<p data-nodeid="159">其中，mapreduce.framework.name 选项就是用来指定 mapreduce 的运行时环境，指定为 yarn 即可，下面的三个选项是指定 mapreduce 运行时一些环境信息。</p>
<p data-nodeid="160">最后，修改另一个文件 yarn-site.xml，添加如下内容到 标签中：</p>
<pre class="lang-html" data-nodeid="161"><code data-language="html"><span class="hljs-tag">&lt;<span class="hljs-name">property</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.nodemanager.aux-services<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>mapreduce_shuffle<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">property</span>&gt;</span>
</code></pre>
<p data-nodeid="162">其中，yarn.nodemanager.aux-services 选项代表可在 NodeManager 上运行的扩展服务，需配置成 mapreduce_shuffle，才可运行 MapReduce 程序。</p>
<p data-nodeid="163">配置修改完成后，需要重启 ResourceManager 与 nodemanager 服务才能使配置生效。</p>
<p data-nodeid="164">现在，我们再次运行刚才的那个 mapreduce 的 wordcount 统计，所有执行的任务都会在 ResourceManager 的 Web 页面展示出来，如下图所示：</p>
<p data-nodeid="165"><img src="https://s0.lgstatic.com/i/image/M00/00/F5/Ciqc1F6qvuSAdk0yAAGEDVAOIY0370.png" alt="image5.png" data-nodeid="393"></p>
<p data-nodeid="166">从图中可以清晰的看出，执行任务的 ID 名、执行任务的用户、程序名、任务类型、队列、优先级、启动时间、完成时间、最终状态等信息。从运维角度来说，这个页面有很多信息都需要引起关注，比如任务最终状态是否有失败的，如果有，可以点击倒数第二列“Tracking UI”下面的 History 链接查看日志进行排查问题。</p>
<p data-nodeid="167">Namenode 的 Web 页面和 ResourceManager 的 Web 页面在进行大数据运维工作中，经常会用到，这些 Web 界面主要用来状态监控、故障排查，更多使用细节和技巧，后面课时会做更加详细的介绍。</p>
<h3 data-nodeid="168">小结</h3>
<p data-nodeid="169">怎么样，现在可以感受到 Hadoop 集群的应用场景了吧！虽然本课时介绍的是伪分别式环境，但与真实的完全分布式 Hadoop 环境实现的功能完全一样。上面的例子中我只是统计了一个小文本中单词的数量，你可能会说，这么点数据，手动几秒钟就算出来了，真没看到分布式计算有什么优势。没错，在小量数据环境中，使用 Yarn 分析是没有意义的，而如果你有上百 GB 甚至 TB 级别的数据时，就能深刻感受到分布式计算的威力了。但有一点请注意，不管数据量大小，分析的方法都是一样的，所以，你可以按照上面执行 wordcount 的方法去读取 GB 甚至 TB 级别的数据。</p>

---

### 精选评论

##### **雄：
> 真的是手把手式的详细教学，要是以前有这篇文章带我入门就好了，相见恨晚！😭

##### **5448：
> Hadoop安装让我螺旋升天

##### **3801：
> 配置文件完成、服务重启后，如何查看mapreduce 任务有没有在 yarn环境中执行，通过ResourceManager 的 Web 查看不到mapreduce的任务，如何查找原因呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果在yarn界面看不到，一般原因是任务是单机运行的，没走集群，伪分布模式下跑mR，需要添加几个配置，这个课程里面介绍了，添加配置后，就能在yarn中看到运行任务了。

##### *磊：
> 文中说到：Cloudera 承诺 CDH 和 HDP 平台将可以继续使用，直到 2022 年，那么这个时间过了怎么办？Cloudera还会继续免费？还是新建的集群应首选apache社区版本？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; CDH的策略不是很清楚，说的2022年，应该是技术支持和升级吧，后面是否会存在开源版本不得而知，个人感觉应该会一直存在吧

##### **强：
> 我怎么觉得拉勾的质量比极客好，极客的偏理论些~

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢支持～

##### **晨：
> 老师 你好 我运行hadoop 命令时候 会报出INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false这样的情况 是因为什么验证的原因吗？

##### **川：
> 文中下载hadoop用的是 hadoop/core 下的，apache 官网给出的是 hadoop/common 下的链接，请问有什么区别

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 两个地址其实是一样的

##### **飞：
> namenode网页打不开，防火墙也关了，不知道咋回事

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个要看具体的日志信息和故障截图，是不是namenode服务没启动成功呢，确认服务是否正常启动。

##### *庆：
> <span style="font-size: 16.0125px;">老师你好，我按照步骤操作的，启动NameNode服务后，运行模式就显示“Safemode is off”了，DataNode服务还未启动，不知道为什么呢？</span>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第一次启动就是这样，因为没有数据啊，后面hdfs有数据的话，就会校验了。

##### *峰：
> 我们公司用的cdh，离线安装有什么好办法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 自己搭建个本地yum仓库就行了，后续从本地下载安装或者升级。

##### **保：
> 打卡

##### **星：
> 目前负责大数据运维，但是对大数据运维的岗位职责及职业规划不是很清晰，希望能借助这个专栏找到答案。PS：给作者点赞。

##### test：
> 赞

##### *岩：
> 课件做得很好

##### **俊：
> 劳动节好好学习，给好文章点赞。

##### **7962：
> 每天期待更新

