<h3>安装部署独立模式的 Spark</h3>
<p>Spark 现在已经广泛使用在各个企业中，常见的应用模式有两种，分别是独立集群模式，以及与 Yarn 整合使用模式，下面分别介绍这两种模式的使用。</p>
<h4>1. Spark 集群运行架构</h4>
<p>从集群部署的角度看，Spark 集群由集群管理器（Cluster Manager）、工作节点（Worker）、执行器（Executor）、驱动器（Driver）、应用程序（Application）等部分组成，其整体关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/14/Ciqc1F7eFeyAVscGAAHpX-ShiyY637.png" alt="1.png"></p>
<p>下面我将对每个部分的功能进行介绍。</p>
<ul>
<li>Cluster Manager</li>
</ul>
<p>Spark 的集群管理器，主要负责整个集群资源的分配与管理。Cluster Manager 在 Yarn 部署模式下为 ResourceManager；在 Mesos 部署模式下为 Mesos Master；在 Standalone 部署模式下为 Master。Cluster Manager 分配的资源属于一级分配，它将各个 Worker 上的内存、CPU 等资源分配给 Application，但是并不负责对 Executor 的资源分配。Standalone 部署模式下的 Master 会直接给 Application 分配内存、CPU 及 Executor 等资源。目前，Standalone、Yarn、Mesos、EC2 等都可以作为 Spark 的集群管理器。</p>
<ul>
<li>Worker</li>
</ul>
<p>Worker 是 Spark 的工作节点，在 Yarn 部署模式下实际由 NodeManager 替代。Worker 节点主要负责以下工作：</p>
<ul>
<li>将自己的内存、CPU 等资源通过注册机制告知 Cluster Manager；</li>
<li>创建 Executor，将资源和任务进一步分配给 Executor；</li>
<li>同步资源信息、Executor 状态信息给 Cluster Manager。</li>
</ul>
<p>在独立部署模式下，Master 将 Worker 上的内存、CPU 及 Executor 等资源分配给 Application 后，将命令 Worker 启动 CoarseGrainedExecutorBackend 进程（此进程会创建 Executor 实例）。</p>
<ul>
<li>Executor</li>
</ul>
<p>Executor 是 Spark 任务（Task）的执行单元，运行在 worker 上，实际上它是一组计算资源（CPU 核心、Memory）的集合。一个 Worker 上的 Memory、CPU 由多个 Executor 共同分摊。同时，Executor 还负责与 Worker、Driver 的信息同步。其实这个 Executor 跟 Yarn 资源管理器中的 Container 实现的功能类似。</p>
<ul>
<li>Driver</li>
</ul>
<p>Driver 可以理解为 Application 的驱动程序，Application 通过 Driver 与 Cluster Manager、Executor 进行通信。Driver 可以运行在 Application 中，也可以由 Application 提交给 Cluster Manager，并由 Cluster Manager 安排在 Worker 中运行。</p>
<ul>
<li>Application</li>
</ul>
<p>用户使用 Spark 提供的 API 编写的应用程序，Application 通过 Spark API 将进行 RDD 的转换和 DAG 的构建，并通过 Driver 将 Application 注册到 Cluster Manager。Cluster Manager 将会根据 Application 的资源需求，通过一级分配将 Executor、内存、CPU 等资源分配给 Application。Driver 通过二级分配将 Executor 等资源分配给每一个任务，Application 最后通过 Driver 告诉 Executor 运行任务。</p>
<h4>2. 安装 Spark</h4>
<p>Spark 的安装和 Hadoop 类似，主要是添加环境变量，修改配置文件等操作。需要注意，独立模式的 Spark 集群需要 HDFS 的支持，因此在部署 Spark 集群之前，要确保 HDFS 集群已经成功启动。</p>
<p><strong>Spark 集群部署环境介绍</strong></p>
<p>这里我在一套部署好 Hadoop 集群的主机上部署 Spark 集群，主机系统是 Centos7.7 版本，HDFS 分布式文件系统已经正常启动，Yarn 服务无须启动，相关主机信息如下表所示：</p>
<table>
<thead>
<tr>
<th align="left">主机名</th>
<th align="center">部署服务</th>
<th align="left">角色</th>
</tr>
</thead>
<tbody>
<tr>
<td align="left">nnmaster.cloud(172.16.213.151)</td>
<td align="center">主 Namenode、Spark</td>
<td align="left">Spark master</td>
</tr>
<tr>
<td align="left">yarnserver.cloud(172.16.213.152)</td>
<td align="center">备 Namenode、Spark</td>
<td align="left">Spark work1</td>
</tr>
<tr>
<td align="left">slave001.cloud(172.16.213.138)</td>
<td align="center">Datanode、Spark</td>
<td align="left">Spark work2</td>
</tr>
<tr>
<td align="left">slave002.cloud(172.16.213.80)</td>
<td align="center">Datanode、Spark</td>
<td align="left">Spark work3</td>
</tr>
</tbody>
</table>
<p><strong>Spark 程序的下载与安装</strong></p>
<p>你可从 <a href="https://spark.apache.org/downloads.html">https://spark.apache.org/downloads.html</a> 下载各个版本的 Spark，这里我们下载最新版本 Spark-3.0.0-preview2-bin-hadoop3.2.tgz 版本。老规矩，程序放置路径为：/opt/bigdata/spark/，在上面四个节点均安装 Spark 程序，安装过程如下：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@nnmaster opt]<span class="hljs-comment">#mkdir -p /opt/bigdata/spark</span>
[root@nnmaster opt]<span class="hljs-comment"># tar zxvf spark-3.0.0-preview2-bin-hadoop3.2.tgz -C /opt/ bigdata/spark/</span>
[root@nnmaster opt]<span class="hljs-comment"># cd /opt/ bigdata/spark/</span>
[root@nnmaster spark]<span class="hljs-comment"># ln -s  spark-3.0.0-preview2-bin-hadoop3.2  current</span>
</code></pre>
<p>对于 Spark 集群的安装配置，一个安装经验是：在一个节点安装程序，安装完成后，开始进行参数配置，参数配置完成，将此节点的 Spark 程序进行打包，然后统一复制到其他集群节点。复制方法可采用 ansible 批量完成。</p>
<p>Spark 的安装就是上面这个步骤，安装完成后，下面进入配置阶段。</p>
<p><strong>添加环境变量</strong></p>
<p>首先，我们要在 Spark 集群的每个节点上修改 /etc/profile 文件，添加 Spark 环境变量，内容如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">export</span> SPARK_HOME=<span class="hljs-regexp">/opt/</span>bigdata/spark/current
<span class="hljs-keyword">export</span> PATH=$PATH:$SPARK_HOME/bin
</code></pre>
<p>接下来执行 source 命令使其生效：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@nnmaster conf]<span class="hljs-comment">#source /etc/profile</span>
</code></pre>
<p>这样就添加好环境变量了，下面我们开始配置 Spark 集群。</p>
<h4>3. 配置 Spark 集群</h4>
<p>下面的配置过程在 Spark 集群每个节点都有操作，或者你可以在一个节点配置完成，然后批量复制到其他所有节点。</p>
<p>Spark 配置文件在 /opt/bigdata/spark/current/conf 文件夹下，默认配置文件都是以 template 为后缀的模板文件，需要将模板文件重命名一下，操作如下：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@nnmaster opt]<span class="hljs-comment">#cd /opt/bigdata/spark/current/conf </span>
[root@nnmaster opt]<span class="hljs-comment">#mv spark-env.sh.template  spark-env.sh</span>
[root@nnmaster opt]<span class="hljs-comment">#mv spark-defaults.conf.template  spark-defaults.conf</span>
[root@nnmaster opt]<span class="hljs-comment"># mv slaves.template  slaves</span>
</code></pre>
<p>这里需要配置三个文件，分别是：spark-env.sh、spark-defaults.conf 和 slave 文件。首先修改配置文件 spark-env.sh，此文件用来配置 Spark 运行时的一些环境变量信息，内容如下：</p>
<pre><code data-language="sql" class="lang-sql">[root@nnmaster opt]<span class="hljs-comment"># more spark-env.sh</span>
export JAVA_HOME=/opt/bigdata/jdk
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/bigdata/hadoop/current/lib/native
export SPARK_LIBRARY_PATH=$SPARK_LIBRARY_PATH
export SPARK_CLASSPATH=$SPARK_CLASSPATH
export HADOOP_HOME=/opt/bigdata/hadoop/current
export HADOOP_CONF_DIR=/etc/hadoop/conf
export SPARK_MASTER_IP=nnmaster.cloud
export SPARK_WORKER_INSTANCES=1
export SPARK_WORKER_MEMORY=20g
export SPARK_WORKER_CORES=20
export SPARK_EXECUTOR_CORES=2
export SPARK_EXECUTOR_MEMORY=2g
</code></pre>
<p>这段配置主要设置了 jdk 环境变量、Hadoop 相关环境变量信息以及 Spark 相关配置信息。对每个变量含义说明如下：</p>
<ul>
<li>JAVA_HOME，指定 Java 安装目录；</li>
<li>HADOOP_HOME，指定 Hadoop 安装目录；</li>
<li>HADOOP_CONF_DIR，指定 Hadoop 集群的配置文件的目录；</li>
<li>SPARK_MASTER_IP，指定 Spark 集群的 Master 节点的 IP 地址；</li>
<li>SPARK_WORKER_INSTANCES，设置每台机器上开启的 Worker 节点的数目；</li>
<li>SPARK_WORKER_MEMORY，设置每个 Worker 节点能够最大分配的系统内存大小；</li>
<li>SPARK_WORKER_CORES，设置每个 Worker 节点能够最大分配的系统 CPU 核数；</li>
<li>SPARK_EXECUTOR_CORES，设置每个 Executor 的 CPU 内核个数，分配更多的内核意味着 Executor 并发能力越强，能够同时执行更多的 Task；</li>
<li>SPARK_EXECUTOR_MEMORY，配置每个 Executor 可使用的内存大小。</li>
</ul>
<p>最后这四个参数是互为依赖的，从它们的关系中可以算出每个 Worker 节点最大的 Executor 数，要保证每个 Worker 节点上启动的 Executor 均衡。如果不均衡的话会造成数据倾斜，拉慢任务的整体速度。</p>
<p>接着配置 spark-defaults.conf 文件，此文件是 Spark 核心配置文件，主要配置 Spark 运行时的一些参数设置，常见的配置参数在下面这段代码中做了展示：</p>
<pre><code data-language="sql" class="lang-sql">spark.driver.extraClassPath      /opt/bigdata/spark/current/libexec<span class="hljs-comment">/*
spark.executor.extraClassPath   /opt/bigdata/spark/current/libexec/*
spark.local.dirs  /data1/sparktmp, /data2/sparktmp
spark.master spark://nnmaster.cloud:7077
</span></code></pre>
<p>对几个重要参数解释如下：</p>
<ul>
<li>spark.driver.extraClassPath，将指定的 jar 包路径注册到 driver 的 classpath 中；</li>
<li>spark.executor.extraClassPath，将指定的 jar 包路径注册到 Executor 的 classpath 中；</li>
<li>spark.local.dirs，用于存储 Spark 临时输出文件和 RDD 缓存文件，常配置在 SSD 等存储设备上，可以通过逗号分隔指定多个目录；</li>
<li>spark.master，指定 Spark 集群 master 节点的地址和端口。</li>
</ul>
<p>最后，修改 slave 文件，内容如下：</p>
<pre><code data-language="sql" class="lang-sql">yarnserver.cloud
slave001.cloud
slave002.cloud
</code></pre>
<p>此文件用来指定 Spark 集群的成员，每个 Spark 成员通过主机名形式来指定，一行一个。</p>
<h4>4. Spark 日志输出定义</h4>
<p>用户可以定义 Spark 日志的输出级别和方式，进入到 Spark 目录下的 conf 文件夹，此时有一个 log4j.properties.template 文件，我们执行如下命令，将其复制一份为 log4j.properties，并对 log4j.properties 文件进行修改。</p>
<pre><code data-language="sql" class="lang-sql">[root@nnmaster opt]<span class="hljs-comment"># cp log4j.properties.template log4j.properties</span>
[root@nnmaster opt]<span class="hljs-comment"># vim log4j.properties</span>
</code></pre>
<p>如上面的代码所示，找到 log4j.rootCategory=INFO, console，修改为：log4j.rootCategory=WARN, console。也就是将 INFO 改为 WARN，这样日志信息在有 WARN 信息的时候才输出。</p>
<h4>5. 测试 Spark集群</h4>
<p>要测试 Spark 集群，需要首先保证 Hadoop 的 HDFS 服务正常运行，这个不再多说。接着，就可以启动 Spark 了。</p>
<p>首先启动 Spark 的 Master 节点，执行如下命令：</p>
<pre><code data-language="sql" class="lang-sql">[hadoop@nnmaster ~]$ /opt/bigdata/spark/current/sbin/<span class="hljs-keyword">start</span>-master.sh
[hadoop@nnmaster bigdata]$ jps
<span class="hljs-number">12993</span> NameNode
<span class="hljs-number">14275</span> <span class="hljs-keyword">Master</span>
<span class="hljs-number">18278</span> Jps
<span class="hljs-number">13198</span> DFSZKFailoverController
</code></pre>
<p>可以发现，有一个 Master 服务，这个就是 Spark 集群的 Master 节点服务进程。</p>
<p>然后在所有 works 节点启动 work 服务：</p>
<pre><code data-language="sql" class="lang-sql">[hadoop@nnmaster ~]$ /opt/spark/current/sbin/<span class="hljs-keyword">start</span>-slave.sh  spark://nnmaster.cloud:<span class="hljs-number">7077</span>
[hadoop@slave001 sparktmp]$ jps
<span class="hljs-number">10450</span> Jps
<span class="hljs-number">32435</span> QuorumPeerMain
<span class="hljs-number">614</span> DataNode
<span class="hljs-number">4169</span> Worker
<span class="hljs-number">32702</span> JournalNode
</code></pre>
<p>可以发现，有一个 Worker 进程，这个就是 Spark 集群中 work 节点对应的服务名。</p>
<p>如果不出问题的话，此时你的 Spark 集群已经成功启动，查看集群情况，可访问 [http:// nnmaster.cloud:8080/](http:// nnmaster.cloud:8080/) ，这里的主机名换成自己环境的 Master 地址即可。如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/13/CgqCHl7eCEWAIQYkAAF6TniV8_Y016.png" alt="Drawing 1.png"></p>
<p>从图中可以看出，整个 Spark 集群的运行状态、活跃节点数，以及可用的 CPU 和内存资源。在下面的 Workers 列表中，显示了 Spark 集群目前活跃的节点数以及每个节点可用的系统资源。</p>
<h3>spark-shell、SparkSQL 与 Spark-Submit 的使用</h3>
<p>Spark 集群搭建起来后，如何提交任务到 Spark 上呢，这就是接下来要介绍的内容，执行 Spark 任务常见的有 spark-shell、SparkSQL 与 Spark-Submit 几种方式，下面分别进行介绍。</p>
<h4>1. SparkSQL 与 Hive 整合</h4>
<p>Spark SQL 是 Spark 用来处理结构化数据的一个模块，它提供了一个编程抽象作为分布式 SQL 的查询引擎，Spark SQL 跟 Hive 有点类似，但比 Hive 查询性能高很多。因此，在实际的应用中，如果你已经在使用 Hive，那么 Spark SQL 可以通过 Hive metastore 获取 Hive 表的元数据。</p>
<p>当然，Spark SQL 自己也可创建元数据库，并不一定要依赖 Hive 的元数据库，但在具体使用中，我们一般是维护一份元数据，因此，可以让 Spark SQL 共享 Hive 的元数据，如果仅仅是执行查询，那么只需要连接到 Hive 元数据库即可，不需要启动 Hiveserver2 服务。但是如果要像 Hive 一样持久化文件与表的关系，就要使用 Hiveserver2 服务。</p>
<p>要实现 Spark SQL 访问 Hive 元数据，需要执行两个步骤的操作，第一步是将 Hive 安装包中 conf/hive-site.xml 配置文件复制到 Spark 安装包的 conf 目录下；第二步是将 Hive 中连接MySQL 的驱动 jar 包复制到 Spark 安装目录下的 jars 目录下。</p>
<p>执行完成上面两个步骤，即可启动 spark-sql 了，下面这段代码展示了它的操作过程：</p>
<pre><code data-language="shell" class="lang-shell"> [hadoop@slave002 ~]$  spark-sql  --master spark://nnmaster.cloud:7077 
<span class="hljs-meta">spark-sql&gt;</span><span class="bash"> select count(1) FROM test_table_002;</span>
</code></pre>
<p>正常情况下，应该很快得到查询结果。<br>
Spark SQL 模式提交 Spark 任务一般用于测试、开发等临时应用的交互场景。</p>
<h4>2. spark-shell 的使用</h4>
<p>Spark 的 shell 作为一个强大的交互式数据分析工具，提供了一个简单的方式学习 API。它可以使用 Scala 或 Python，使用 spark-shell 的两种模式，分别是本地模式和集群模式，先看本地模式的使用方法，如下所示：</p>
<pre><code data-language="shell" class="lang-shell">[hadoop@slave002 ~]$ spark-shell --master local
<span class="hljs-meta">scala&gt;</span><span class="bash"> val textFile = sc.textFile( <span class="hljs-string">"file:///home/hadoop/weblog010"</span> )</span>
<span class="hljs-meta">scala&gt;</span><span class="bash"> textFile.count()</span>
res0: Long = 140207
</code></pre>
<p><strong>注意：file:///home/hadoop/weblog010，首部的 file 代表本地目录，注意 file: 后有三个斜杠 (/)。</strong> 要使用本地文件，spark-shell 必须运行在本地模式下。如果运行在本地模式下，就不会在 Master 的 8080 页面显示 Application 运行状态。</p>
<p>接着，使用集群模式启动 spark-shell，执行如下命令：</p>
<pre><code data-language="shell" class="lang-shell">[hadoop@SparkWorker2 jars]$ spark-shell --master spark://nnmaster.cloud:7077
Spark context available as 'sc' (master = spark://nnmaster.cloud:7077, app id = app-20200601175518-0009).
Spark session available as 'spark'.
<span class="hljs-meta">scala&gt;</span><span class="bash"> val textFile = sc.textFile(<span class="hljs-string">"/logs/weblog003"</span>)</span>
<span class="hljs-meta">scala&gt;</span><span class="bash"> textFile.count()</span>
res0: Long = 140741
<span class="hljs-meta">scala&gt;</span><span class="bash"> textFile.first()</span>
res1: String = # Apache Spark
</code></pre>
<p>这里的路径：/logs/weblog003 是 HDFS 文件系统的路径。其中，count 代表 RDD 中的总数据条数；first 代表 RDD 中的第一行数据。</p>
<p>从上面的 spark-shell 输出，可以看出此任务的 APP id = app-20200601175518-0009，打开 Spark 的 8080 状态界面，可以发现一个 app-20200601175518-0009 正在运行。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/08/Ciqc1F7eCKCAJB2VAAGkrRokxa8933.png" alt="Drawing 2.png"></p>
<p>还有下面的用法，这是对指定的一个目录进行统计分析：</p>
<pre><code data-language="shell" class="lang-shell"><span class="hljs-meta">scala&gt;</span><span class="bash"> val textFile = sc.textFile(“/logs/web*<span class="hljs-string">")</span></span>
<span class="hljs-meta">scala&gt;</span><span class="bash"><span class="hljs-string"> textFile.count()</span></span>
res1: Long = 212674560
</code></pre>
<p>spark-shell 跟 spark-sql 类似，主要用于做交互式分析，生产环境一般不使用这种方式。</p>
<h4>3. Spark-Submit 的使用</h4>
<p>Spark-Submit 用来提交在 IDEA 中编写并且打包成 jar 的包到集群中运行。一般在生产环境使用这种方式。下面是一个 Spark 自带的例子：</p>
<pre><code data-language="sql" class="lang-sql">[hadoop@slave002 jars]$ spark-submit <span class="hljs-comment">--executor-memory 5g --total-executor-cores 10 --class org.apache.spark.examples.SparkPi --master spark://nnmaster.cloud:7077 /opt/bigdata/spark/current/examples/jars /spark-examples_2.11-2.3.1.jar</span>
</code></pre>
<p>这个例子中用了几个参数，主要是对资源参数的设置，含义如下：</p>
<ul>
<li>--executor-memory：指定每个 Executor 的内存，此值可在 spark-env.sh 中进行配置，也可在执行 Spark 任务时指定，如果不指定则默认读取 spark-env.sh 中的设置。</li>
<li>--total-executor-cores：指定运行此任务需要所有 Executor 总共使用的 CPU 核数。</li>
</ul>
<p>除此之外，还可以通过 --executor-cores 指定每个 Executor 所占用 core 的数量。除了 spark-submit，spark-sql、spark-shell 运行任务时也可以指定这些参数来设置任务运行时需要的资源。</p>
<h3>Spark 多种运行模式分析</h3>
<p>在 Spark 独立集群模式下，可以在本地模式、客户端模式以及完全集群模式下执行 Spark 任务，每个运行模式都有相关应用场景和细微区别，下面我将分别介绍。</p>
<h4>1. 本地或者单机运行模式</h4>
<p>该模式被称为 Local 模式，是用单机的多个线程来模拟 Spark 分布式计算，通常用来验证开发出来的应用程序逻辑上是否有问题。运行该模式非常简单，只需要把 Spark 的安装包解压后，改一些常用的配置即可使用，而不用启动 Spark 的 Master、Worker 守护进程（只有使用独立集群模式时，才需要这两个角色），也不用启动 Hadoop 的各服务（除非你要用到 HDFS），这是和其他模式的区别。</p>
<p>我们用如下命令提交作业：</p>
<pre><code data-language="sql" class="lang-sql">[hadoop@slave002 ~]$ spark-submit <span class="hljs-comment">--master local  --class org.apache.spark.examples.SparkPi  /opt/bigdata/spark/current/examples/jars/spark-examples_2.12-3.0.0-preview2.jar</span>
</code></pre>
<p>在程序执行过程中，可以看到，只会生成一个 SparkSubmit 进程。这个 SparkSubmit 进程既是客户提交任务的 Client 进程、又是 Spark 的 driver 程序、还充当着 Spark 执行 Task 的 Executor 角色。</p>
<h4>2. Spark 客户端集群模式</h4>
<p>和单机运行的模式不同，在使用客户端集群模式时，必须在执行任务前，先启动 Spark 的 Master 和 Worker 守护进程。在这种运行模式，可以使用 Spark 的 8080 web ui 来观察集群资源和应用程序的执行情况。</p>
<p>看下面一个例子：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@slave</span>002 conf]$ spark-submit --master spark:<span class="hljs-comment">//nnmaster.cloud:7077 --deploy-mode client --class org.apache.spark.examples.SparkPi /opt/bigdata/spark/current/examples/jars/spark-examples_2.12-3.0.0-preview2.jar </span>
Pi is roughly <span class="hljs-number">3.133655668278341</span>
</code></pre>
<p>此命令执行后，执行结果会输出到屏幕。此例子是 spark-submit 在独立集群中以 Client 模式来运行的。可以看到这里添加了“--deploy-mode client”这个参数，其实如果不指定“--deploy-mode”模式，默认就是 Client 模式。</p>
<p>在客户端集群模式下，程序运行期间，会在执行此程序的客户端机器上生成一个 SparkSubmit 进程，并运行 4040 端口，此进程作为 Client 端并运行 driver 程序，通过访问此客户端的 4040 端口，可以看到 driver 运行在哪个节点，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/08/Ciqc1F7eCPaAYt3xAAGOaN-9t_k453.png" alt="Drawing 3.png"></p>
<p>上面的操作我是在 slave002 节点进行的，因此 driver 程序就运行在此节点上，还可以看到每个 Executor 的资源使用情况。当任务运行完毕后，4040 端口会自动关闭。</p>
<p>下面分析下客户端集群模式下，Spark 任务的执行过程，叙述如下：</p>
<ul>
<li>任务提交后，在客户端（slave002）上会生成一个 SparkSubmit 进程，同时 driver 程序也运行在此客户端上；</li>
<li>driver 程序运行起来后，就会在其他 work 节点启动 CoarseGrainedExecutorBackend 进程，此进程用来创建和维护 Executor，CoarseGrainedExecutorBackend 和 Executor 是一一对应的关系；</li>
<li>Executor 创建起来后，就开始处理 Task 对象，Executor 内部通过线程池的方式来完成 Task 的计算；</li>
<li>所以任务执行完成后，CoarseGrainedExecutorBackend 进程自动退出，客户端上的 driver 程序也自动退出。</li>
</ul>
<h4>3. Spark 完全集群模式</h4>
<p>此运行模式需要启动 Spark 的 Master、Worker 守护进程，然后执行如下命令：</p>
<pre><code data-language="java" class="lang-java">[hadoop<span class="hljs-meta">@slave</span>002 conf]$ spark-submit --master spark:<span class="hljs-comment">//nnmaster.cloud:7077 --deploy-mode cluster --class org.apache.spark.examples.SparkPi /opt/bigdata/spark/current/examples/jars/spark-examples_2.12-3.0.0-preview2.jar</span>
</code></pre>
<p>此命令执行完毕后，直接退出命令行，执行结果不会输出到屏幕。要查看执行结果，可以到运行 driver 程序的节点对应的 stdout 输出中查看。下图是完全集群模式下 Spark 任务的运行状态：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/08/Ciqc1F7eCRWAExIQAAH3sX9VMy8453.png" alt="Drawing 4.png"></p>
<p>从上图可以看到，master 选择了“worker-20200601142643-172.16.213.138-33869”这个节点来运行 driver 程序，点开这个节点连接，页面拉到最后，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/08/Ciqc1F7eCS2ACYQ7AADtGJZDP44063.png" alt="Drawing 5.png"></p>
<p>上图显示的是已经完成的 driver 程序，查看第一个 driver，点开 Logs 列下面的 stdout 链接，如下图所示，这就是上面命令的执行结果。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/1B/14/CgqCHl7eCSaAeDFsAACjualC02g847.png" alt="Drawing 6.png"></p>
<p>下面分析下完全集群模式下，Spark 任务的执行过程，叙述如下：</p>
<ul>
<li>任务提交后，此客户端（slave002）会生成一个 SparkSubmit 进程，此进程会在应用程序提交给集群之后就退出；</li>
<li>Master 会在集群中选择一个 Worker 节点生成一个子进程 DriverWrapper 来启动 driver 程序，该 DriverWrapper 进程会占用 Worker 节点的一个 core；</li>
<li>driver 程序运行起来后，就会在其他 work 节点启动 CoarseGrainedExecutorBackend 进程，此进程用来创建和维护 Executor，CoarseGrainedExecutorBackend 和 Executor 是一对一的关系；</li>
<li>Executor创建起来后，就开始处理 Task 对象，Executor 内部通过线程池的方式来完成 Task 的计算；</li>
<li>所有任务执行完成，CoarseGrainedExecutorBackend 进程自动退出。</li>
</ul>
<h3>总结</h3>
<p>本课时主要讲解了 Spark 独立集群的部署以及 spark-sql、spark-shell 和 Spark-Submit 的使用，最后还分析了每种使用方式在 Spark 内部的执行流程，了解这些内部执行流程，对于故障排除和性能调优非常重要。</p>

---

### 精选评论

##### *贺：
> 老师你好，请问spark env中这几个变量有何作用export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/bigdata/hadoop/current/lib/nativeexport SPARK_LIBRARY_PATH=$SPARK_LIBRARY_PATHexport SPARK_CLASSPATH=$SPARK_CLASSPATH

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是第三方的一些系统库文件，支持hadoop的

##### **9134：
> spark 部署模式中client和cluster部署这块没理解透彻，老师，能用几句话概述一下吗?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 简单来说，client模式的话，提交任务的客户端会一直运行，直到任务运行完成，结果在提交任务的客户端主机上输出，而cluster模式的话，提交任务后，客户端进程马上退出，任务执行结果不会输出到提交任务的客户端，会在某个集群节点输出，两种模式使用场景不同，client模式可用于开发/测试环境，cluster主要用于生产环境下。

