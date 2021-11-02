<h3 data-nodeid="51774" class="">Kafka 基础与入门</h3>
<h4 data-nodeid="51775">1. Kafka 基本概念</h4>
<p data-nodeid="51776">Kafka 官方的定义：是一种高吞吐量的分布式发布/订阅消息系统。这样说起来可能不太好理解，这里简单举个例子：现在是个大数据时代，各种商业、社交、搜索、浏览都会产生大量的数据，那么如何快速收集这些数据，如何实时地分析这些数据，是一个必须要解决的问题。同时，这也形成了一个业务需求模型，即生产者生产（Produce）各种数据，消费者（Consume）消费（分析、处理）这些数据。面对这些需求，如何高效、稳定地完成数据的生产和消费呢？这就需要在生产者与消费者之间，建立一个通信的桥梁，这个桥梁就是<strong data-nodeid="51955">消息系统</strong>。从微观层面来说，这种业务需求也可理解为不同系统之间如何传递消息。</p>
<h4 data-nodeid="51777">2. Kafka 角色术语</h4>
<p data-nodeid="51778">在介绍架构之前，先来了解下 Kafka 中的一些核心概念和各种角色。</p>
<ul data-nodeid="51779">
<li data-nodeid="51780">
<p data-nodeid="51781">Broker：Kafka 集群包含一个或多个服务器，每个服务器被称为 Broker。</p>
</li>
<li data-nodeid="51782">
<p data-nodeid="51783">Topic：每条发布到 Kafka 集群的消息都有一个分类，这个类别被称为 Topic（主题）。</p>
</li>
<li data-nodeid="51784">
<p data-nodeid="51785">Producer：消息的生产者，负责发布消息到 Kafka Broker。</p>
</li>
<li data-nodeid="51786">
<p data-nodeid="51787">Consumer：消息的消费者，从 Kafka Broker 拉取数据，并消费这些已发布的消息。</p>
</li>
<li data-nodeid="51788">
<p data-nodeid="51789">Partition：其是物理上的概念，每个 Topic 包含一个或多个 Partition，每个 Partition 都是一个有序的队列。Partition 中的每条消息都会被分配一个有序的 ID（称为 Offset）。</p>
</li>
<li data-nodeid="51790">
<p data-nodeid="51791">Consumer：消费者组，可以给每个 Consumer 指定消费者组，若不指定，则属于默认的 Group。</p>
</li>
<li data-nodeid="51792">
<p data-nodeid="51793">Message：消息，通信的基本单位，每个 Producer 可以向一个 Topic 发布一些消息。</p>
</li>
</ul>
<p data-nodeid="51794">这些概念和基本术语对于理解 Kafka 架构和运行原理非常重要，一定要牢记。</p>
<h4 data-nodeid="51795">3. Kafka 拓扑架构</h4>
<p data-nodeid="56011">一个典型的 Kafka 集群包含若干 Producer、Broker、Consumer Group，以及一个 ZooKeeper 集群。Kafka 通过 ZooKeeper 管理集群配置，选举 Leader，以及在 Consumer Group 发生变化时进行 Rebalance。Producer 使用 push 模式将消息发布到 Broker，Consumer 使用 pull 模式从 Broker 订阅并消费消息。典型架构图如下图所示：</p>
<p data-nodeid="56012" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/29/9F/CgqCHl767x6AXxk3AAKplkX7pcA534.png" alt="2.png" data-nodeid="56016"></p>


<p data-nodeid="51798">从图中可以看出，典型的消息系统由生产者（Producer）、存储系统（Broker）和消费者（Consumer）组成。Kafka 作为分布式的消息系统支持多个生产者和消费者，生产者可以将消息分布到集群中不同节点的不同 Partition 上，消费者也可以消费集群中多个节点上的多个 Partition。在写消息时允许多个生产者写到同一个 Partition 中，但是读消息时一个 Partition 只允许被一个消费组中的一个消费者所消费，而一个消费者可以消费多个 Partition。也就是说同一个消费组下消费者对 Partition 是互斥的，而不同消费组之间是共享的。</p>
<p data-nodeid="51799">Kafka 支持消息持久化存储，持久化数据保存在 Kafka 的日志文件中，在生产者生产消息后，Kafka 不会直接把消息传递给消费者，而是先要在 Broker 中进行存储。为了减少磁盘写入的次数，Broker 会将消息暂时缓存起来，当消息的个数或尺寸、大小达到一定阈值时，再统一写到磁盘上。这样不但提高了 Kafka 的执行效率，也减少了磁盘 IO 调用次数。</p>
<p data-nodeid="51800">Kafka 中每条消息写到 Partition 中时，是顺序写入磁盘的，这个很重要。因为在机械盘中如果是随机写入的话，效率将很低，但如果顺序写入，那么效率却非常高。这种顺序写入磁盘机制是 Kafka 高吞吐率的一个很重要的保证。</p>
<h4 data-nodeid="51801">4. Topic 与 Partition</h4>
<p data-nodeid="51802">Kafka 中的 Topic 是以 Partition 的形式存放的，每一个 Topic 都可以设置它的 Partition 数量，该数量决定了组成 Topic 的 Log 的数量。推荐 Partition 的数量一定要大于同时运行的 Consumer 数量。另外，建议 Partition 的数量大于集群 Broker 的数量，这样消息数据就可以均匀地分布在各个 Broker 中了。</p>
<p data-nodeid="51803">那么，Topic 为什么要设置多个 Partition 呢？这是因为 Kafka 是基于文件存储的，通过配置多个 Partition 可以将消息内容分散存储到多个 Broker 上，这样可以避免文件尺寸达到单机磁盘的上限。同时，将一个 Topic 切分成任意多个 Partitions，可以保证消息存储、消息消费的效率，因为越多的 Partitions 可以容纳更多的 Consumer，可有效提升 Kafka 的吞吐率。因此，将 Topic 切分成多个 Partitions 的好处是可以将大量的消息分成多批数据同时写到不同节点上，将写请求分担负载到各个集群节点。</p>
<p data-nodeid="51804">在存储结构上，每个 Partition 在物理上对应一个文件夹，该文件夹下存储这个 Partition 的所有消息和索引文件。Partiton 命名规则为 Topic 名称 + 序号，第一个 Partiton 序号从 0 开始，序号最大值为 Partitions 数量减 1。</p>
<p data-nodeid="51805">在每个 Partition（文件夹）中有多个大小相等的 Segment（段）数据文件，每个 Segment 的大小是相同的，但是每条消息的大小可能不相同。因此 Segment 数据文件中消息数量不一定相等。Segment 数据文件有两个部分组成：index file 和 data file，此两个文件一一对应，成对出现，后缀“.index”和“.log”分别表示为 Segment 索引文件和数据文件。</p>
<h4 data-nodeid="51806">5. Producer 生产机制</h4>
<p data-nodeid="51807">Producer 是消息和数据的生产者，当它发送消息到 Broker 时，会根据 Paritition 机制选择将其存储到哪一个 Partition。如果 Partition 机制设置的合理，所有消息都可以均匀分布到不同的 Partition 里，这样就实现了数据的负载均衡。如果一个 Topic 对应一个文件，那这个文件所在的机器 I/O 将会成为这个 Topic 的性能瓶颈；而有了 Partition 后，不同的消息可以并行写入不同 Broker 的不同 Partition 里，极大地提高了吞吐率。</p>
<h4 data-nodeid="51808">6. Consumer 消费机制</h4>
<p data-nodeid="51809">Kafka 发布消息通常有两种模式：<strong data-nodeid="52002">队列模式（Queuing）<strong data-nodeid="52001">和</strong>发布/订阅模式</strong>（Publish-Subscribe）。在队列模式下，只有一个消费组，而这个消费组有多个消费者，一条消息只能被这个消费组中的一个消费者所消费；而在发布/订阅模式下，可有多个消费组，每个消费组只有一个消费者，同一条消息可被多个消费组消费。</p>
<p data-nodeid="51810">Kafka 中的 Producer 和 Consumer 采用的是 push（推送）、pull（拉取）的模式，即 Producer 只是向 Broker push 消息，Consumer 只是从 Broker pull 消息，push 和 pull 对于消息的生产和消费是异步进行的。pull 模式的一个好处是 Consumer 可自主控制消费消息的速率，同时 Consumer 还可以自己控制消费消息的方式是批量地从 Broker 拉取数据还是逐条消费数据。</p>
<h3 data-nodeid="51811">Kafka 分布式集群构建</h3>
<h4 data-nodeid="51812">1. Kakfa 集群部署规划</h4>
<p data-nodeid="51813">这里我采用三台服务器来构建 Kakfa 集群，由于该集群依赖 ZooKeeper，所以在部署 Kafka 之前，需要部署好 ZooKeeper 集群。ZooKeeper 部署在前面的 Hadoop 第 4 课时中已经介绍过了，这里就不过多说明了。这里我将 Kafka 和 ZooKeeper 部署在了一起，Kakfa 集群节点操作系统仍然采用 Centos 7.7 版本，各个主机角色和软件版本如下表所示：</p>
<table data-nodeid="51815">
<thead data-nodeid="51816">
<tr data-nodeid="51817">
<th data-org-content="**IP 地址**" data-nodeid="51819"><strong data-nodeid="52012">IP 地址</strong></th>
<th align="center" data-org-content="**主机名**" data-nodeid="51820"><strong data-nodeid="52016">主机名</strong></th>
<th data-org-content="**安装软件**" data-nodeid="51821"><strong data-nodeid="52020">安装软件</strong></th>
<th align="center" data-org-content="**软件版本**" data-nodeid="51822"><strong data-nodeid="52024">软件版本</strong></th>
</tr>
</thead>
<tbody data-nodeid="51827">
<tr data-nodeid="51828">
<td data-org-content="172.16.213.31" data-nodeid="51829">172.16.213.31</td>
<td align="center" data-org-content="kafkazk1" data-nodeid="51830">kafkazk1</td>
<td data-org-content="Kafka+ ZooKeeper" data-nodeid="51831">Kafka+ ZooKeeper</td>
<td align="center" data-org-content="kafka\_2.11-2.4.1、zookeeper-3.5.8" data-nodeid="51832">kafka_2.11-2.4.1、zookeeper-3.5.8</td>
</tr>
<tr data-nodeid="51833">
<td data-org-content="172.16.213.41" data-nodeid="51834">172.16.213.41</td>
<td align="center" data-org-content="kafkazk2" data-nodeid="51835">kafkazk2</td>
<td data-org-content="Kafka+ ZooKeeper" data-nodeid="51836">Kafka+ ZooKeeper</td>
<td align="center" data-org-content="kafka\_2.11-2.4.1、zookeeper-3.5.8" data-nodeid="51837">kafka_2.11-2.4.1、zookeeper-3.5.8</td>
</tr>
<tr data-nodeid="51838">
<td data-org-content="172.16.213.70" data-nodeid="51839">172.16.213.70</td>
<td align="center" data-org-content="kafkazk3" data-nodeid="51840">kafkazk3</td>
<td data-org-content="Kafka+ ZooKeeper" data-nodeid="51841">Kafka+ ZooKeeper</td>
<td align="center" data-org-content="kafka\_2.11-2.4.1、zookeeper-3.5.8" data-nodeid="51842">kafka_2.11-2.4.1、zookeeper-3.5.8</td>
</tr>
</tbody>
</table>
<p data-nodeid="51843">这里需要注意：Kafka 和 ZooKeeper 的版本，默认 Kafka2.11 版本自带的 ZooKeeper 依赖 jar 包版本为 3.5.7，因此 ZooKeeper 的版本至少在 3.5.7 及以上。</p>
<h4 data-nodeid="51844">2. 下载与安装 Kafka</h4>
<p data-nodeid="51845">Kafka 也需要安装 Java 运行环境，这在前面课程已经介绍过了，这里不再介绍，<a href="https://kafka.apache.org/downloads" data-nodeid="52050">你可以点击 Kafka 官网获取 Kafka 安装包</a>，推荐的版本是 kafka_2.11-2.4.1.tgz。将下载下来的安装包直接解压到一个路径下即可完成 Kafka 的安装，这里统一将 Kafka 安装到 /usr/local 目录下，我以在 kakfazk1 主机为例，基本操作过程如下：</p>
<pre class="lang-shell" data-nodeid="51846"><code data-language="shell">[root@kafkazk1~]# tar -zxvf kafka_2.11-2.4.1.tgz  -C /usr/local
[root@kafkazk1~]# mv /usr/local/kafka_2.11-2.4.1  /usr/local/kafka
</code></pre>
<p data-nodeid="51847">这里我创建了一个 Kafka 用户，用来管理和维护 Kakfa 集群，后面所有对 Kafka 的操作都通过此用户来完成，执行如下操作进行创建用户和授权：</p>
<pre class="lang-shell" data-nodeid="51848"><code data-language="shell">[root@kafkazk1~]# useradd kafka
[root@kafkazk1~]# chown -R kafka:kafka /usr/local/kafka
</code></pre>
<p data-nodeid="51849">在 kafkazk1 节点安装完成 Kafka 后，先不着急复制安装程序到其他两个节点，因为还没有配置 Kakfa，等 Kafka 配置完成，再统一打包复制到其他两个节点上。</p>
<h4 data-nodeid="51850">3. 配置 Kafka 集群</h4>
<p data-nodeid="51851">Kafka 安装完成后，接着进入配置阶段，这里我们将 Kafka 安装到 /usr/local 目录下，因此，其主配置文件为 /usr/local/kafka/config/server.properties。这里以节点 kafkazk1 为例，重点介绍一些常用配置项的含义：</p>
<pre class="lang-java" data-nodeid="51852"><code data-language="java">broker.id=<span class="hljs-number">1</span>
listeners=PLAINTEXT:<span class="hljs-comment">//172.16.213.31:9092</span>
log.dirs=/usr/local/kafka/logs
num.partitions=<span class="hljs-number">6</span>
log.retention.hours=<span class="hljs-number">60</span>
log.segment.bytes=<span class="hljs-number">1073741824</span>
zookeeper.connect=<span class="hljs-number">172.16</span>.<span class="hljs-number">213.31</span>:<span class="hljs-number">2181</span>,<span class="hljs-number">172.16</span>.<span class="hljs-number">213.41</span>:<span class="hljs-number">2181</span>,<span class="hljs-number">172.16</span>.<span class="hljs-number">213.70</span>:<span class="hljs-number">2181</span>
auto.create.topics.enable=<span class="hljs-keyword">true</span>
delete.topic.enable=<span class="hljs-keyword">true</span>
</code></pre>
<p data-nodeid="54053">其中，每个配置项含义如下表所示：</p>
<p data-nodeid="54054"><img src="https://s0.lgstatic.com/i/image/M00/29/93/Ciqc1F767vWAd94AAAPqcj_UCiU303.png" alt="1.png" data-nodeid="54058"></p>




<p data-nodeid="51894" class="">Kakfa 配置文件修改完成后，接着打包 Kakfa 安装程序，将程序复制到其他两个节点，然后进行解压即可。注意，在其他两个节点上，broker.id 务必要修改，Kafka 集群中 broker.id 不能有相同的。</p>
<h4 data-nodeid="51895">4. 启动 Kafka 集群</h4>
<p data-nodeid="51896">三个节点的 Kafka 配置完成后，就可以启动了，但在启动 Kafka 集群前，需要确保 ZooKeeper 集群已经正常启动。接着，依次在 Kafka 各个节点上执行如下命令即可：</p>
<pre class="lang-sql" data-nodeid="51897"><code data-language="sql">[root@kafkazk1~]<span class="hljs-comment"># cd /usr/local/kafka</span>
[root@kafkazk1 kafka]<span class="hljs-comment"># nohup bin/kafka-server-start.sh  config/server.properties &amp;</span>
[root@kafkazk1 kafka]<span class="hljs-comment"># jps</span>
21840 Kafka
15593 Jps
15789 QuorumPeerMain
</code></pre>
<p data-nodeid="51898">这里将 Kafka 放到后台运行，启动后，会在启动 Kafka 的当前目录下生成一个 nohup.out 文件，可通过此文件查看 Kafka 的启动和运行状态。通过 jps 指令，可以看到有个 Kafka 标识，这是 Kafka 进程成功启动的标志。</p>
<h3 data-nodeid="51899">Kafka 集群基本命令操作</h3>
<p data-nodeid="51900">Kafka 提供了多个命令用于查看、创建、修改、删除 Topic 信息，也可以通过命令测试如何生产消息、消费消息等，这些命令位于 Kafka 安装目录的 bin 目录下，这里是 /usr/local/kafka/bin。登录任意一台 Kafka 集群节点，切换到此目录下，即可进行命令操作，下面简单列举了 Kafka 的一些常用命令的使用方法。</p>
<p data-nodeid="51901">（1）显示 topic 列表</p>
<p data-nodeid="51902">命令执行方式如下所示：</p>
<pre class="lang-shell" data-nodeid="51903"><code data-language="shell">[kafka@kafkazk1 bin]$ ./kafka-topics.sh  --bootstrap-server 172.16.213.31:9092,172.16.213.41:9092,172.16.213.70:9092 --list
</code></pre>
<p data-nodeid="51904">其中，“--bootstrap-server”参数后面跟的是 Kakfa 集群的主机列表和端口。具体操作实例如下图所示：</p>
<p data-nodeid="51905" class=""><img src="https://s0.lgstatic.com/i/image/M00/29/09/Ciqc1F75zUGAYfoiAABCAUEAIlY280.png" alt="Drawing 1.png" data-nodeid="52100"></p>
<p data-nodeid="51906">（2）创建一个 topic，并指定 topic 属性（副本数、分区数等）</p>
<p data-nodeid="51907">命令执行方式如下图所示：</p>
<p data-nodeid="51908"><img src="https://s0.lgstatic.com/i/image/M00/29/09/Ciqc1F75zUqAKe83AABSjfkpAhc990.png" alt="Drawing 2.png" data-nodeid="52105"></p>
<p data-nodeid="51909">其中：</p>
<ul data-nodeid="51910">
<li data-nodeid="51911">
<p data-nodeid="51912">--create 表示创建一个 topic；</p>
</li>
<li data-nodeid="51913">
<p data-nodeid="51914">--replication-factor 表示这个 topic 的副本数，这里设置为 1 个；</p>
</li>
<li data-nodeid="51915">
<p data-nodeid="51916">--partitions 指定 topic 的分区数，一般设置为小于或等于 Kafka 集群节点数即可；</p>
</li>
<li data-nodeid="51917">
<p data-nodeid="51918">--topic 指定要创建的 topic 名称。</p>
</li>
</ul>
<p data-nodeid="51919">（3）查看某个 topic 的状态</p>
<p data-nodeid="51920">命令执行方式如下图所示：</p>
<p data-nodeid="51921"><img src="https://s0.lgstatic.com/i/image/M00/29/14/CgqCHl75zVOAEBvZAABcNUxHq34583.png" alt="Drawing 3.png" data-nodeid="52115"></p>
<p data-nodeid="51922">这里通过“--describe”选项查看刚刚创建好的 testtopic 状态，其中：</p>
<ul data-nodeid="51923">
<li data-nodeid="51924">
<p data-nodeid="51925">Partition 表示分区 ID，通过输出可以看到 testtopic 有三个分区、一个副本，这刚好和我们创建 testtopic 时指定的配置吻合；</p>
</li>
<li data-nodeid="51926">
<p data-nodeid="51927">Leader 表示当前负责读写的 Leader broker；</p>
</li>
<li data-nodeid="51928">
<p data-nodeid="51929">Replicas 表示当前分区所有副本对应的 broker列表；</p>
</li>
<li data-nodeid="51930">
<p data-nodeid="51931">Isr 表示处于活动状态的 broker。</p>
</li>
</ul>
<p data-nodeid="51932">（4）生产消息</p>
<p data-nodeid="51933">命令执行方式如下图所示：</p>
<p data-nodeid="51934"><img src="https://s0.lgstatic.com/i/image/M00/29/09/Ciqc1F75zWCALnmeAABJGSilU-I224.png" alt="Drawing 4.png" data-nodeid="52125"></p>
<p data-nodeid="51935">这里需要注意，“--broker-list”后面跟的内容是 Kafka 集群的 IP 和端口，当输入这条命令后，光标处于可写状态，接着就可以写入一些测试数据，每行一条。这里输入的内容如上图红框所示。</p>
<p data-nodeid="51936">（5）消费消息</p>
<p data-nodeid="51937">在生产消息的同时，再登录任意一台 Kafka 集群节点，执行消费消息的命令，结果如下图所示：</p>
<p data-nodeid="51938"><img src="https://s0.lgstatic.com/i/image/M00/29/14/CgqCHl75zWmAYn6KAABJjpXJyzc277.png" alt="Drawing 5.png" data-nodeid="52131"></p>
<p data-nodeid="51939">可以看到，在第四步中，输入的消息在这里原样输出了，这样就完成了消息的消费。</p>
<p data-nodeid="51940">（6）删除 topic</p>
<p data-nodeid="51941">命令执行方式如下图所示：</p>
<p data-nodeid="51942"><img src="https://s0.lgstatic.com/i/image/M00/29/09/Ciqc1F75zW-ABZUcAABRh9IDyZQ707.png" alt="Drawing 6.png" data-nodeid="52137"></p>
<p data-nodeid="51943">注意这里的“--delete”选项，用来删除一个指定的 topic。然后通过“--list”查看发现这个 topic 确实已经被删除了。</p>
<h3 data-nodeid="51944">总结</h3>
<p data-nodeid="51945">本课时主要讲解了 Kafka 的概念、术语及 Kafka 分布式集群的构建，最后介绍了 Kafka 的基本操作指令，要熟练使用 Kafka，重要的是了解其内部实现机制及运行原理。Kafka 作为一个消息中间件，在集群架构环境中经常使用，特别是在大数据架构环境中。因此，对于本课时介绍的 Kafka 知识点要求熟练掌握。</p>

---

### 精选评论

##### *磊：
> 1. 推荐 Partition 的数量一定要大于同时运行的 Consumer 数量。另外，建议 Partition 的数量大于集群 Broker 的数量，这样消息数据就可以均匀地分布在各个 Broker 中了。2.--partitions 指定 topic 的分区数，一般设置为小于或等于 Kafka 集群节点数即可1和2说的是一个东西吗？分区数量应该是大于broker数量还是小于？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 分区数应该大于或等于Broker数量，等于最好。1和2说的大概意思一样。

##### **绿：
> 课程详尽，对于kafka的学习获益匪浅，谢谢老师。老师，在文中介绍Produer生产消息时，提到的Partition机制，对于此机制能否做下介绍，谢谢老师！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Kafka中的topic是以partition的形式存放的，每一个topic都可以设置它的partition数量，Partition的数量决定了组成topic的log的数量。推荐partition的数量一定要大于同时运行的consumer的数量。另外，建议partition的数量大于集群broker的数量，这样消息数据就可以均匀的分布在各个broker中。
那么，Topic为什么要设置多个Partition呢，这是因为kafka是基于文件存储的，通过配置多个partition可以将消息内容分散存储到多个broker上，这样可以避免文件尺寸达到单机磁盘的上限。同时，将一个topic切分成任意多个partitions，可以保证消息存储、消息消费的效率，因为越多的partitions可以容纳更多的consumer，可有效提升Kafka的吞吐率。因此，将Topic切分成多个partitions的好处是可以将大量的消息分成多批数据同时写到不同节点上，将写请求分担负载到各个集群节点。

