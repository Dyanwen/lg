<p data-nodeid="1269" class="">上一讲，我们讲解了分布式事务的解决方案以及 Seata 分布式事务中间件AT模式的实现原理，在后面的实战篇，我们还将围绕 Seata 进行进一步的学习。本讲我们先来介绍分布式架构下另外一块重要拼图：消息队列 RocketMQ。</p>
<p data-nodeid="1270">本讲咱们将学习以下三方面内容：</p>
<ul data-nodeid="1271">
<li data-nodeid="1272">
<p data-nodeid="1273">介绍消息队列与 Alibaba RocketMQ；</p>
</li>
<li data-nodeid="1274">
<p data-nodeid="1275">掌握 RocketMQ 的部署方式；</p>
</li>
<li data-nodeid="1276">
<p data-nodeid="1277">讲解微服务接入 RocketMQ 的开发技巧；</p>
</li>
</ul>
<p data-nodeid="1278">首先咱们先来认识什么是消息队列 MQ 呢？</p>
<h3 data-nodeid="1279">消息队列与 RocketMQ</h3>
<h4 data-nodeid="1280">消息队列 MQ</h4>
<p data-nodeid="1281">消息队列（Message Queue）简称 MQ，是一种跨进程的通信机制，通常用于应用程序间进行数据的异步传输，MQ 产品在架构中通常也被叫作“消息中间件”。它的最主要职责就是保证服务间进行可靠的数据传输，同时实现服务间的解耦。</p>
<p data-nodeid="1282">这么说太过学术，我们看一个项目的实际案例，假设市级税务系统向省级税务系统上报本年度税务汇总数据，按以往的设计市级税务系统作为数据的生产者需要了解省级税务系统的 IP、端口、接口等诸多细节，然后通过 RPC、RESTful 等方式同步向省级税务系统发送数据，省级税务系统作为数据的消费者接受后响应“数据已接收”。</p>
<p data-nodeid="1580" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/1C/Cgp9HWBu2EuANOnqAAEJZsgHoCk159.png" alt="图片1.png" data-nodeid="1584"></p>
<div data-nodeid="1581"><p style="text-align:center">系统间跨进程通信</p></div>


<p data-nodeid="1285" class="">虽然从逻辑上是没有问题的，但是从技术层面却衍生出三个新问题：</p>
<ul data-nodeid="1286">
<li data-nodeid="1287">
<p data-nodeid="1288">假如上报时省级税务系统正在升级维护，市级税务系统就必须设计额外的重发机制保证数据的完整性；</p>
</li>
<li data-nodeid="1289">
<p data-nodeid="1290">假如省级税务系统接收数据需要 1 分钟处理时间，市级税务系统采用同步通信，则市级税务系统传输线程就要阻塞 1 分钟，在高并发场景下如此长时间的阻塞很容易造成系统的崩溃；</p>
</li>
<li data-nodeid="1291">
<p data-nodeid="1292">假如省级税务系统接口的调用方式、接口、IP、端口有任何改变，都必须立即通知市级税务系统进行调整，否则就会出现通信失败。</p>
</li>
</ul>
<p data-nodeid="1293">从以上三个问题可以看出，省级系统产生的变化直接影响到市级税务系统的执行，两者产生了强耦合，如果问题放在互联网的微服务架构中，几十个服务进行串联调用，每个服务间如果都产生类似的强耦合，系统必然难以维护。</p>
<p data-nodeid="1294">为了解决这种情况，我们需要在架构中部署消息中间件，这个组件应提供可靠的、稳定的、与业务无关的特性，使进程间通信解耦，而这一类消息中间件的代表产品就是 MQ 消息队列。当引入 MQ 消息队列后，消息传递过程会产生以下变化。</p>
<p data-nodeid="2207" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/24/CioPOWBu2FaAD6pQAAEtpzXgzW8765.png" alt="图片2.png" data-nodeid="2211"></p>
<div data-nodeid="2208"><p style="text-align:center">引入 MQ 后通信过程</p></div>


<p data-nodeid="1297" class="">可以看到，引入消息队列后，生产者与消费者都只面向消息队列进行数据处理，数据生产者根本不需要了解具体消费者的信息，只要把数据按事先约定放在指定的队列中即可。而消费者也是一样的，消费者端监听消息队列，如果队列中产生新的数据，MQ 就会通过“推送 PUSH”或者“抽取 PULL”的方式让消费者获取到新数据进行后续处理。</p>
<p data-nodeid="1298">通过示意图可以看到，只要消息队列产品是稳定可靠的，那消息通信的过程就是有保障的。在架构领域，很多厂商都开发了自己的 MQ 产品，最具代表性的开源产品有：</p>
<ul data-nodeid="1299">
<li data-nodeid="1300">
<p data-nodeid="1301">Kafka</p>
</li>
<li data-nodeid="1302">
<p data-nodeid="1303">ActiveMQ</p>
</li>
<li data-nodeid="1304">
<p data-nodeid="1305">ZeroMQ</p>
</li>
<li data-nodeid="1306">
<p data-nodeid="1307">RabbitMQ</p>
</li>
<li data-nodeid="1308">
<p data-nodeid="1309">RocketMQ</p>
</li>
</ul>
<p data-nodeid="1310">每一种产品都有自己不同的设计与实现原理，但根本的目标都是相同的：为进程间通信提供可靠的异步传输机制。RocketMQ 作为阿里系产品天然被整合进 Spring Cloud Alibaba 生态，在经历过多次双 11 的考验后，RocketMQ 在性能、可靠性、易用性方面都是非常优秀的，下面咱们来了解下 RocketMQ 吧。</p>
<h4 data-nodeid="1311">RocketMQ</h4>
<p data-nodeid="1312">RocketMQ 是一款分布式消息队列中间件，官方地址为<a href="http://rocketmq.apache.org/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1463">http://rocketmq.apache.org/</a>，目前最新版本为4.8.0。RocketMQ 最初设计是为了满足阿里巴巴自身业务对异步消息传递的需要，在 3.X 版本后正式开源并捐献给 Apache，目前已孵化为 Apache 顶级项目，同时也是国内使用最广泛、使用人数最多的 MQ 产品之一。</p>
<p data-nodeid="2834" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/1C/Cgp9HWBu2GKABJW5AAPatFf4EbA571.png" alt="图片3.png" data-nodeid="2837"></p>

<p data-nodeid="1314" class="">RocketMQ 有很多优秀的特性，在可用性方面，RocketMQ 强调集群无单点，任意一点高可用，客户端具备负载均衡能力，可以轻松实现水平扩容；在性能方面，在天猫双 11 大促背后的亿级消息处理就是通过 RocketMQ 提供的保障；在 API 方面，提供了丰富的功能，可以实现异步消息、同步消息、顺序消息、事务消息等丰富的功能，能满足大多数应用场景；在可靠性方面，提供了消息持久化、失败重试机制、消息查询追溯的功能，进一步为可靠性提供保障。</p>
<p data-nodeid="1315">了解 RocketMQ 的诸多特性后，咱们来理解 RocketMQ 几个重要的概念：</p>
<ul data-nodeid="1316">
<li data-nodeid="1317">
<p data-nodeid="1318">消息 Message：消息在广义上就是进程间传递的业务数据，在狭义上不同的 MQ 产品对消息又附加了额外属性如：Topic（主题）、Tags（标签）等；</p>
</li>
<li data-nodeid="1319">
<p data-nodeid="1320">消息生产者 Producer：指代负责生产数据的角色，在前面案例中市级税务系统就充当了消息生产者的角色；</p>
</li>
<li data-nodeid="1321">
<p data-nodeid="1322">消息消费者 Consumer：指代使用数据的角色，前面案例的省级税务系统就是消息消费者；</p>
</li>
<li data-nodeid="1323">
<p data-nodeid="1324">MQ消息服务 Broker：MQ 消息服务器的统称，用于消息存储与消息转发；</p>
</li>
<li data-nodeid="1325">
<p data-nodeid="1326">生产者组 Producer Group：对于发送同一类消息的生产者，RocketMQ 对其分组，成为生产者组；</p>
</li>
<li data-nodeid="1327">
<p data-nodeid="1328">消费者组 Consumer Group：对于消费同一类消息的消费者，RocketMQ 对其分组，成为消费者组。</p>
</li>
</ul>
<p data-nodeid="3460" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/1C/Cgp9HWBu2G-ASqw3AAELZIiTELk603.png" alt="图片4.png" data-nodeid="3464"></p>
<div data-nodeid="3461"><p style="text-align:center">RocketMQ 组成示意图</p></div>


<p data-nodeid="1331" class="">在理解这些基本概念后，咱们正式进入 RocketMQ 的部署与使用环节，通过案例代码理解 RocketMQ 的执行过程。对于 RocketMQ 来说，使用它需要两个阶段：搭建 RocketMQ 服务器集群与应用接入 RocketMQ 队列，首先咱们来部署 RocketMQ 集群。</p>
<h3 data-nodeid="1332">部署 RocketMQ 集群</h3>
<p data-nodeid="1333">RocketMQ 天然采用集群模式，常见的 RocketMQ 集群有三种形式：<strong data-nodeid="1494">多 Master 模式</strong>、<strong data-nodeid="1495">多 Master 多 Slave- 异步复制模式</strong>、<strong data-nodeid="1496">多 Master 多 Slave- 同步双写模式</strong>，这三种模式各自的优缺点如下。</p>
<ul data-nodeid="1334">
<li data-nodeid="1335">
<p data-nodeid="1336">多 Master 模式是配置最简单的模式，同时也是使用最多的形式。优点是单个 Master 宕机或重启维护对应用无影响，在磁盘配置为 RAID10 时，即使机器宕机不可恢复情况下，由于 RAID10 磁盘非常可靠，同步刷盘消息也不会丢失，性能也是最高的；缺点是单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订阅，消息实时性会受到影响。</p>
</li>
<li data-nodeid="1337">
<p data-nodeid="1338">多 Master 多 Slave 异步复制模式。每个 Master 配置一个 Slave，有多对 Master-Slave，HA 采用异步复制方式，主备有短暂消息毫秒级延迟，即使磁盘损坏只会丢失少量消息，且消息实时性不会受影响。同时 Master 宕机后，消费者仍然可以从 Slave 消费，而且此过程对应用透明，不需要人工干预，性能同多 Master 模式几乎一样；缺点是 Master 宕机，磁盘损坏情况下会丢失少量消息。</p>
</li>
<li data-nodeid="1339">
<p data-nodeid="1340">多 Master 多 Slave 同步双写模式，HA 采用同步双写方式，即只有主备都写成功，才向应用返回成功，该模式数据与服务都无单点故障，Master 宕机情况下，消息无延迟，服务可用性与数据可用性都非常高；缺点是性能比异步复制模式低 10% 左右，发送单个消息的执行时间会略高，且目前版本在主节点宕机后，备机不能自动切换为主机。</p>
</li>
</ul>
<p data-nodeid="1341">本讲我们将搭建一个双 Master 服务器集群，首先来看一下部署架构图：</p>
<p data-nodeid="4087" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/24/CioPOWBu2HyAJB6-AACJ2Or_yLg890.png" alt="图片5.png" data-nodeid="4091"></p>
<div data-nodeid="4088"><p style="text-align:center">双 Master 架构图</p></div>


<p data-nodeid="1344" class="">在双 Master 架构中，出现了一个新角色 NameServer（命名服务器），NameServer 是 RocketMQ 自带的轻量级路由注册中心，支持 Broker 的动态注册与发现。在 Broker 启动后会自动向 NameServer 发送心跳包，通知 Broker 上线。当 Provider 向 NameServer 获取路由信息，然后向指定 Broker 建立长连接完成数据发送。</p>
<p data-nodeid="1345">为了避免单节点瓶颈，通常 NameServer 会部署两台以上作为高可用冗余。NameServer 本身是无状态的，各实例间不进行通信，因此在 Broker 集群配置时要配置所有 NameServer 节点以保证状态同步。</p>
<p data-nodeid="1346">部署 RocketMQ 集群要分两步：部署 NameServer 与部署 Broker 集群。</p>
<h4 data-nodeid="1347">第一步，部署 NameServer 集群。</h4>
<p data-nodeid="1348">我们创建两台 CentOS7 虚拟机，IP 地址分别为 192.168.31.200 与 192.168.31.201，要求这两台虚拟机内存大于 2G，并安装好 64 位 JDK1.8，具体过程不再演示。</p>
<p data-nodeid="1349">之后访问 Apache RocketMQ 下载页：</p>
<p data-nodeid="1350"><a href="https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1512">https://www.apache.org/dyn/closer.cgi?path=rocketmq/4.8.0/rocketmq-all-4.8.0-bin-release.zip</a></p>
<p data-nodeid="1351">获取 RocketMQ 最新版 rocketmq-all-4.8.0-bin-release.zip，解压后编辑 rocketmq-all-4.8.0-bin-release/bin/runserver.sh 文件，因为 RocketMQ 是服务器软件，默认为其配置 8G 内存，这是 PC 机及或者笔记本吃不消的，所以在 82 行附近将 JVM 内存缩小到 1GB 以方便演示。</p>
<p data-nodeid="1352">修改前：</p>
<pre class="lang-java" data-nodeid="1353"><code data-language="java">JAVA_OPT=<span class="hljs-string">"${JAVA_OPT} -server -Xms8g -Xmx8g -Xmn4g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"</span>
</code></pre>
<p data-nodeid="1354">修改后：</p>
<pre class="lang-java" data-nodeid="1355"><code data-language="java">JAVA_OPT=<span class="hljs-string">"${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"</span>
</code></pre>
<p data-nodeid="1356">修改完毕，将 rocketmq-all-4.8.0-bin-release 上传到两台 NameServer 虚拟机的 /usr/local 目录下，执行 bin 目录下的 mqnamesrv 命令。</p>
<pre class="lang-java" data-nodeid="1357"><code data-language="java">cd /usr/local/rocketmq-all-<span class="hljs-number">4.8</span><span class="hljs-number">.0</span>-bin-release/bin/
sh mqnamesrv
</code></pre>
<p data-nodeid="1358">mqnamesrv 是 RocketMQ 自带 NameServer 的启动命令，执行后看到 The Name Server boot success. serializeType=JSON 就代表 NameServer 启动成功，NameServer 将占用 9876 端口提供服务，不要忘记在防火墙设置放行。之后如法炮制在另一台 201 设备上部署 NameServer，构成 NameServer 集群。</p>
<h4 data-nodeid="1359">第二步，部署 Broker 集群。</h4>
<p data-nodeid="1360">我们再额外创建两台 CentOS7 虚拟机，IP 地址分别为 192.168.31.210 与 192.168.31.211，同样要求这两台虚拟机内存大于 2G，并安装好 64 位 JDK1.8。</p>
<p data-nodeid="1361">打开 rocketmq-all-4.8.0-bin-release 目录，编辑 /bin/runbroker.sh 文件，同样将启动 Broker 默认占用内存从 8G 缩小到 1G，将 64 行调整为以下内容：</p>
<pre class="lang-java" data-nodeid="1362"><code data-language="java">JAVA_OPT=<span class="hljs-string">"${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m"</span>
</code></pre>
<p data-nodeid="1363">在 conf 目录下，RocketMQ 已经给我们贴心的准备好三组集群配置模板：</p>
<ul data-nodeid="1364">
<li data-nodeid="1365">
<p data-nodeid="1366">2m-2s-async 代表双主双从异步复制模式；</p>
</li>
<li data-nodeid="1367">
<p data-nodeid="1368">2m-2s-sync 代表双主双从同步双写模式；</p>
</li>
<li data-nodeid="1369">
<p data-nodeid="1370">2m-noslave 代表双主模式。</p>
</li>
</ul>
<p data-nodeid="1371">我们在 2m-noslave 双主模式目录中，在 broker-a.properties 与 broker-b.properties 末尾追加 NameServer 集群的地址，为了方便理解我也将模板里面每一项的含义进行注释，首先是 broker-a.properties 的完整内容如下：</p>
<pre class="lang-java" data-nodeid="1372"><code data-language="java">#集群名称，同一个集群下的 broker 要求统一
brokerClusterName=DefaultCluster
#broker 名称
brokerName=broker-a
#brokerId=0 代表主节点，大于零代表从节点
brokerId=0
#删除日志文件时间点，默认凌晨 4 点
deleteWhen=04
#日志文件保留时间，默认 48 小时
fileReservedTime=48
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘，性能好宕机会丢数
#- SYNC_FLUSH 同步刷盘，性能较差不会丢数
flushDiskType=ASYNC_FLUSH
#末尾追加，NameServer 节点列表，使用分号分割
namesrvAddr=192.168.31.200:9876;192.168.31.201:9876
</code></pre>
<p data-nodeid="1373">broker-b.properties 只有 brokerName 不同，如下所示：</p>
<pre class="lang-java" data-nodeid="1374"><code data-language="java">brokerClusterName=DefaultCluster
brokerName=broker-b
brokerId=0
deleteWhen=04
fileReservedTime=48
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
#末尾追加，NameServer 节点列表，使用分号分割
namesrvAddr=192.168.31.200:9876;192.168.31.201:9876
</code></pre>
<p data-nodeid="1375">之后将 rocketmq-all-4.8.0-bin-release 目录上传到 /usr/local 目录，运行下面命令启动 broker 节点 a。</p>
<pre class="lang-java" data-nodeid="1376"><code data-language="java">cd /usr/local/rocketmq-all-<span class="hljs-number">4.8</span><span class="hljs-number">.0</span>-bin-release/
sh bin/mqbroker -c ./conf/<span class="hljs-number">2</span>m-noslave/broker-a.properties
</code></pre>
<p data-nodeid="1377">在 mqbroker 启动命令后增加 c 参数说明要加载哪个 Broker 配置文件。</p>
<p data-nodeid="1378">启动成功会看到下面的日志，Broker 将占用 10911 端口提供服务，请设置防火墙放行。</p>
<pre class="lang-java" data-nodeid="1379"><code data-language="java">The broker[broker-a, <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.210</span>:<span class="hljs-number">10911</span>] boot success. serializeType=JSON and name server is <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.200</span>:<span class="hljs-number">9876</span>;<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.201</span>:<span class="hljs-number">9876</span>
</code></pre>
<p data-nodeid="1380">同样的，在另一台 Master 执行下面命令，启动并加载 broker-b 配置文件。</p>
<pre class="lang-java" data-nodeid="1381"><code data-language="java">cd /usr/local/rocketmq-all-<span class="hljs-number">4.8</span><span class="hljs-number">.0</span>-bin-release/
sh bin/mqbroker -c ./conf/<span class="hljs-number">2</span>m-noslave/broker-b.properties
</code></pre>
<p data-nodeid="1382">到这里 NameServer 集群与 Broker 集群就部署好了，下面执行两个命令验证下。</p>
<p data-nodeid="1383">第一个，使用 mqadmin 命令查看集群状态。</p>
<p data-nodeid="1384">在 bin 目录下存在 mqadmin 命令用于管理 RocketMQ 集群，我们可以使用 clusterList 查看集群节点，命令如下：</p>
<pre class="lang-java" data-nodeid="1385"><code data-language="java">sh mqadmin clusterList -n <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.200</span>:<span class="hljs-number">9876</span>
</code></pre>
<p data-nodeid="1386">通过查询 NameServer 上的注册信息，得到以下结果。</p>
<p data-nodeid="4714" class=""><img src="https://s0.lgstatic.com/i/image6/M00/33/1C/Cgp9HWBu2J6APWfJAAH7nUt8GHs198.png" alt="图片6.png" data-nodeid="4718"></p>
<div data-nodeid="4715"><p style="text-align:center">Broker 集群信息</p></div>


<p data-nodeid="1389" class="">可以看到在 DefaultCluster 集群中存在两个 Broker，因为 BID 编号为 0，代表它们都是 Master 主节点。</p>
<p data-nodeid="1390">第二个，利用 RocketMQ 自带的 tools.sh 工具通过生成演示数据来测试 MQ 实际的运行情况。在 bin 目录下使用下面命令。</p>
<pre class="lang-java" data-nodeid="1391"><code data-language="java">export NAMESRV_ADDR=<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.200</span>:<span class="hljs-number">9876</span>
sh tools.sh org.apache.rocketmq.example.quickstart.Producer
</code></pre>
<p data-nodeid="1392">你会看到屏幕输出日志：</p>
<pre class="lang-java" data-nodeid="1393"><code data-language="java">SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CF540000, offsetMsgId=C0A81FD200002A9F00000000000413B6, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=<span class="hljs-number">1</span>], queueOffset=<span class="hljs-number">0</span>]
SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CF9B0001, offsetMsgId=C0A81FD200002A9F000000000004147F, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=<span class="hljs-number">2</span>], queueOffset=<span class="hljs-number">0</span>]
SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CFA30002, offsetMsgId=C0A81FD200002A9F0000000000041548, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-a, queueId=<span class="hljs-number">3</span>], queueOffset=<span class="hljs-number">0</span>]
SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CFA70003, offsetMsgId=C0A81FD300002A9F0000000000033C56, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-b, queueId=<span class="hljs-number">0</span>], queueOffset=<span class="hljs-number">0</span>]
SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CFD60004, offsetMsgId=C0A81FD300002A9F0000000000033D1F, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-b, queueId=<span class="hljs-number">1</span>], queueOffset=<span class="hljs-number">0</span>]
SendResult [sendStatus=SEND_OK, msgId=<span class="hljs-number">7F</span>0000010B664DC639969F28CFDB0005, offsetMsgId=C0A81FD300002A9F0000000000033DE8, messageQueue=MessageQueue [topic=TopicTest, brokerName=broker-b, queueId=<span class="hljs-number">2</span>], queueOffset=<span class="hljs-number">0</span>]
...
</code></pre>
<p data-nodeid="1394">其中<strong data-nodeid="1546">broker-a、broker-b 交替出现</strong>说明集群生效了。</p>
<p data-nodeid="1395">前面测试的是服务提供者，下面测试消费者，运行下面命令：</p>
<pre class="lang-java" data-nodeid="1396"><code data-language="java">export NAMESRV_ADDR=<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.200</span>:<span class="hljs-number">9876</span>
sh tools.sh org.apache.rocketmq.example.quickstart.Consumer
</code></pre>
<p data-nodeid="1397">会看到消费者也获取到数据，到这里 RocketMQ 双 Master 集群的搭建就完成了，至于多 Master 多 Slave 的配置也是相似的，大家查阅官方文档相信也能很快上手。</p>
<pre class="lang-java" data-nodeid="1398"><code data-language="java">ConsumeMessageThread_11 Receive New Messages: [MessageExt [brokerName=broker-b, queueId=<span class="hljs-number">2</span>, storeSize=<span class="hljs-number">203</span>, queueOffset=<span class="hljs-number">157</span>, sysFlag=<span class="hljs-number">0</span>, bornTimestamp=<span class="hljs-number">1612100880154</span>, bornHost=/<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.210</span>:<span class="hljs-number">54104</span>, storeTimestamp=<span class="hljs-number">1612100880159</span>, storeHost=/<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.211</span>:<span class="hljs-number">10911</span>, msgId=C0A81FD300002A9F0000000000053509, commitLogOffset=<span class="hljs-number">341257</span>, bodyCRC=<span class="hljs-number">1116443590</span>, reconsumeTimes=<span class="hljs-number">0</span>, preparedTransactionOffset=<span class="hljs-number">0</span>, toString()=Message{topic=<span class="hljs-string">'TopicTest'</span>, flag=<span class="hljs-number">0</span>, properties={MIN_OFFSET=<span class="hljs-number">0</span>, MAX_OFFSET=<span class="hljs-number">158</span>, CONSUME_START_TIME=<span class="hljs-number">1612100880161</span>, UNIQ_KEY=<span class="hljs-number">7F</span>0000010DA64DC639969F2C4B1A0314, CLUSTER=DefaultCluster, WAIT=<span class="hljs-keyword">true</span>, TAGS=TagA}, body=[<span class="hljs-number">72</span>, <span class="hljs-number">101</span>, <span class="hljs-number">108</span>, <span class="hljs-number">108</span>, <span class="hljs-number">111</span>, <span class="hljs-number">32</span>, <span class="hljs-number">82</span>, <span class="hljs-number">111</span>, <span class="hljs-number">99</span>, <span class="hljs-number">107</span>, <span class="hljs-number">101</span>, <span class="hljs-number">116</span>, <span class="hljs-number">77</span>, <span class="hljs-number">81</span>, <span class="hljs-number">32</span>, <span class="hljs-number">55</span>, <span class="hljs-number">56</span>, <span class="hljs-number">56</span>], transactionId=<span class="hljs-string">'null'</span>}]] 
ConsumeMessageThread_12 Receive New Messages: [MessageExt [brokerName=broker-b, queueId=<span class="hljs-number">3</span>, storeSize=<span class="hljs-number">203</span>, queueOffset=<span class="hljs-number">157</span>, sysFlag=<span class="hljs-number">0</span>, bornTimestamp=<span class="hljs-number">1612100880161</span>, bornHost=/<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.210</span>:<span class="hljs-number">54104</span>, storeTimestamp=<span class="hljs-number">1612100880162</span>, storeHost=/<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.211</span>:<span class="hljs-number">10911</span>, msgId=C0A81FD300002A9F00000000000535D4, commitLogOffset=<span class="hljs-number">341460</span>, bodyCRC=<span class="hljs-number">898409296</span>, reconsumeTimes=<span class="hljs-number">0</span>, preparedTransactionOffset=<span class="hljs-number">0</span>, toString()=Message{topic=<span class="hljs-string">'TopicTest'</span>, flag=<span class="hljs-number">0</span>, properties={MIN_OFFSET=<span class="hljs-number">0</span>, MAX_OFFSET=<span class="hljs-number">158</span>, CONSUME_START_TIME=<span class="hljs-number">1612100880164</span>, UNIQ_KEY=<span class="hljs-number">7F</span>0000010DA64DC639969F2C4B210315, CLUSTER=DefaultCluster, WAIT=<span class="hljs-keyword">true</span>, TAGS=TagA}, body=[<span class="hljs-number">72</span>, <span class="hljs-number">101</span>, <span class="hljs-number">108</span>, <span class="hljs-number">108</span>, <span class="hljs-number">111</span>, <span class="hljs-number">32</span>, <span class="hljs-number">82</span>, <span class="hljs-number">111</span>, <span class="hljs-number">99</span>, <span class="hljs-number">107</span>, <span class="hljs-number">101</span>, <span class="hljs-number">116</span>, <span class="hljs-number">77</span>, <span class="hljs-number">81</span>, <span class="hljs-number">32</span>, <span class="hljs-number">55</span>, <span class="hljs-number">56</span>, <span class="hljs-number">57</span>], transactionId=<span class="hljs-string">'null'</span>}]]
</code></pre>
<p data-nodeid="1399">集群部署好，那如何使用 RocketMQ 进行消息收发呢？我们结合 Spring Boot 代码进行讲解。</p>
<h3 data-nodeid="1400">应用接入 RocketMQ 集群</h3>
<p data-nodeid="5341" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/33/24/CioPOWBu2KuAeQ6nAAEtpzXgzW8352.png" alt="图片2.png" data-nodeid="5345"></p>
<div data-nodeid="5342"><p style="text-align:center">案例说明</p></div>


<p data-nodeid="1403">我们以前面的报税为例，利用 Spring Boot 集成 MQ 客户端实现消息收发，首先咱们模拟生产者 Producer。</p>
<h4 data-nodeid="1404">生产者 Producer 发送消息</h4>
<p data-nodeid="1405">第一步，利用 Spring Initializr 向导创建 rocketmq-provider 工程，确保 pom.xml 引入以下依赖。</p>
<pre class="lang-xml" data-nodeid="1406"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- RocketMQ客户端，版本与Broker保持一致 --&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.rocketmq<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>rocketmq-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>4.8.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1407">第二步，配置应用 application.yml。</p>
<p data-nodeid="1408">rocketmq-client 主要通过编码实现通信，因此无须在 application.yml 做额外配置。</p>
<pre class="lang-yaml" data-nodeid="1409"><code data-language="yaml"><span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">8000</span>
<span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">rocketmq-producer</span>
</code></pre>
<p data-nodeid="1410">第三步，创建 Controller，生产者发送消息。</p>
<pre class="lang-java" data-nodeid="1411"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ProviderController</span> </span>{
    Logger logger = LoggerFactory.getLogger(ProviderController.class);
    <span class="hljs-meta">@GetMapping(value = "/send_s1_tax")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">send1</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> MQClientException </span>{
        <span class="hljs-comment">//创建DefaultMQProducer消息生产者对象</span>
        DefaultMQProducer producer = <span class="hljs-keyword">new</span> DefaultMQProducer(<span class="hljs-string">"producer-group"</span>);
        <span class="hljs-comment">//设置NameServer节点地址，多个节点间用分号分割</span>
        producer.setNamesrvAddr(<span class="hljs-string">"192.168.31.200:9876;192.168.31.201:9876"</span>);
        <span class="hljs-comment">//与NameServer建立长连接</span>
        producer.start();
        <span class="hljs-keyword">try</span> {
            <span class="hljs-comment">//发送一百条数据</span>
            <span class="hljs-keyword">for</span>(<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span> ; i&lt; <span class="hljs-number">100</span> ; i++) {
                <span class="hljs-comment">//数据正文</span>
                String data = <span class="hljs-string">"{\"title\":\"X市2021年度第一季度税务汇总数据\"}"</span>;
                <span class="hljs-comment">/*创建消息
                    Message消息三个参数
                    topic 代表消息主题，自定义为tax-data-topic说明是税务数据
                    tags 代表标志，用于消费者接收数据时进行数据筛选。2021S1代表2021年第一季度数据
                    body 代表消息内容
                */</span>
                Message message = <span class="hljs-keyword">new</span> Message(<span class="hljs-string">"tax-data-topic"</span>, <span class="hljs-string">"2021S1"</span>, data.getBytes());
                <span class="hljs-comment">//发送消息，获取发送结果</span>
                SendResult result = producer.send(message);
                <span class="hljs-comment">//将发送结果对象打印在控制台</span>
                logger.info(<span class="hljs-string">"消息已发送：MsgId:"</span> + result.getMsgId() + <span class="hljs-string">"，发送状态:"</span> + result.getSendStatus());
            }
        } <span class="hljs-keyword">catch</span> (RemotingException e) {
            e.printStackTrace();
        } <span class="hljs-keyword">catch</span> (MQBrokerException e) {
            e.printStackTrace();
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        } <span class="hljs-keyword">finally</span> {
            producer.shutdown();
        }
        <span class="hljs-keyword">return</span> <span class="hljs-string">"success"</span>;
    }
}
</code></pre>
<p data-nodeid="1412">在程序运行后，访问<a href="http://localhost:8000/send_s1_tax?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1567">http://localhost:8000/send_s1_tax</a>，在控制台会看到如下输出说明数据已被 Broker 接收，Broker 接收后 Producer 端任务已完成。</p>
<pre class="lang-java" data-nodeid="1413"><code data-language="java">消息已发送：MsgId:<span class="hljs-number">7F</span>00000144E018B4AAC29F3B7B280062，发送状态:SEND_OK
消息已发送：MsgId:<span class="hljs-number">7F</span>00000144E018B4AAC29F3B7B2A0063，发送状态:SEND_OK
</code></pre>
<p data-nodeid="1414">下面咱们开发消费者 Consumer。</p>
<h4 data-nodeid="1415">消费者 Consumer 接收消息</h4>
<p data-nodeid="1416">第一步，利用 Spring Initializr 向导创建 rocketmq-consumer 工程，确保 pom.xml 引入以下依赖。</p>
<pre class="lang-xml" data-nodeid="1417"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- RocketMQ客户端，版本与Broker保持一致 --&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.rocketmq<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>rocketmq-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>4.8.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1418">第二步，application.yml 同样无须做额外设置。</p>
<pre class="lang-yaml" data-nodeid="1419"><code data-language="yaml"><span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">9000</span>
<span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">rocketmq-consumer</span>
</code></pre>
<p data-nodeid="1420">第三步，在应用启动入口 RocketmqConsumerApplication 增加消费者监听代码，关键的代码都已做好注释。</p>
<pre class="lang-java" data-nodeid="1421"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RocketmqConsumerApplication</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Logger logger = LoggerFactory.getLogger(RocketmqConsumerApplication.class);
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> MQClientException </span>{
        SpringApplication.run(RocketmqConsumerApplication.class, args);
        //创建消费者对象
        DefaultMQPushConsumer consumer = <span class="hljs-keyword">new</span> DefaultMQPushConsumer(<span class="hljs-string">"consumer-group"</span>);
        <span class="hljs-comment">//设置NameServer节点</span>
        consumer.setNamesrvAddr(<span class="hljs-string">"192.168.31.200:9876;192.168.31.201:9876"</span>);
        <span class="hljs-comment">/*订阅主题，
        consumer.subscribe包含两个参数：
        topic: 说明消费者从Broker订阅哪一个主题，这一项要与Provider保持一致。
        subExpression: 子表达式用于筛选tags。
            同一个主题下可以包含很多不同的tags，subExpression用于筛选符合条件的tags进行接收。
            例如：设置为*，则代表接收所有tags数据。
            例如：设置为2020S1，则Broker中只有tags=2020S1的消息会被接收，而2020S2就会被排除在外。
        */</span>
        consumer.subscribe(<span class="hljs-string">"tax-data-topic"</span>, <span class="hljs-string">"*"</span>);
        <span class="hljs-comment">//创建监听，当有新的消息监听程序会及时捕捉并加以处理。</span>
        consumer.registerMessageListener(<span class="hljs-keyword">new</span> MessageListenerConcurrently() {
            <span class="hljs-function"><span class="hljs-keyword">public</span> ConsumeConcurrentlyStatus <span class="hljs-title">consumeMessage</span><span class="hljs-params">(
                    List&lt;MessageExt&gt; msgs, ConsumeConcurrentlyContext context)</span> </span>{
                <span class="hljs-comment">//批量数据处理</span>
                <span class="hljs-keyword">for</span> (MessageExt msg : msgs) {
                    logger.info(<span class="hljs-string">"消费者消费数据:"</span>+<span class="hljs-keyword">new</span> String(msg.getBody()));
                }
                <span class="hljs-comment">//返回数据已接收标识</span>
                <span class="hljs-keyword">return</span> ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        <span class="hljs-comment">//启动消费者，与Broker建立长连接，开始监听。</span>
        consumer.start();
    }
}
</code></pre>
<p data-nodeid="1422">当应用启动后，Provider 产生新消息的同时，Consumer 端就会立即消费掉，控制台产生输出。</p>
<pre class="lang-java" data-nodeid="1423"><code data-language="java"><span class="hljs-number">2021</span>-<span class="hljs-number">01</span>-<span class="hljs-number">31</span> <span class="hljs-number">22</span>:<span class="hljs-number">25</span>:<span class="hljs-number">14.212</span>  INFO <span class="hljs-number">17328</span> --- [MessageThread_3] c.l.r.RocketmqConsumerApplication        : 消费者消费数据:{<span class="hljs-string">"title"</span>:<span class="hljs-string">"X市2021年度第一季度税务汇总数据"</span>}
<span class="hljs-number">2021</span>-<span class="hljs-number">01</span>-<span class="hljs-number">31</span> <span class="hljs-number">22</span>:<span class="hljs-number">25</span>:<span class="hljs-number">14.217</span>  INFO <span class="hljs-number">17328</span> --- [MessageThread_2] c.l.r.RocketmqConsumerApplication        : 消费者消费数据:{<span class="hljs-string">"title"</span>:<span class="hljs-string">"X市2021年度第一季度税务汇总数据"</span>}
</code></pre>
<p data-nodeid="1424">以上便是 Spring Boot 接入 RocketMQ 集群的过程。对于当前的案例我们是通过代码方式控制消息收发，在 Spring Cloud 生态中还提供了 Spring Cloud Stream 模块，允许程序员采用“声明式”的开发方式实现与 MQ 更轻松的接入，但 Spring Cloud Stream 本身封装度太高，很多 RocketMQ 的细节也被隐藏了，这对于入门来说并不是一件好事。在掌握 RocketMQ 的相关内容后再去学习 Spring Cloud Stream 你会理解得更加透彻。</p>
<h3 data-nodeid="1425">小结与预告</h3>
<p data-nodeid="1426">本讲咱们学习了三方面内容，首先介绍了什么是 MQ 以及 Alibaba RocketMQ 的特性；其次详细讲解了 RocketMQ 双主集群的部署过程；最后通过 Spring Boot 应用中引入 RocketMQ-Client 实现消息的收发。</p>
<p data-nodeid="1427">这里为你留一道思考题：目前主流的 MQ 产品有 RocketMQ、RabbitMQ、Kafka、ActiveMQ、ZeroMQ……不同的产品有不同的设计，假设在银行的金融交易基于 MQ 实现，对 MQ 的可靠性与一致性要求较高，但对数据的响应时间不敏感。如果你是架构师该如何选型，欢迎你把自己的思考写在评论区和大家一起分享。</p>
<p data-nodeid="1428" class="">下一讲我们将开始一个新的篇章，将之前学过的 Spring Cloud Alibaba 综合运用，看在实际项目中有哪些成熟的经验可以为我所用。</p>

---

### 精选评论

##### **1001：
> 老师你好！请问一下Product Group怎么理解，主要作用是什么，不同组与同一组发送的相同的topic消息有什么区别？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 　　生产者集合，表示发送同类消息的多个 Producer 实例（通常为发送一类消息，且发送逻辑一致），一个 Producer Group 下包含多个 Producer 实例，可以是多台机器，也可以是一台机器的多个进程，或者一个进程的多个 Producer 对象； 一个 Producer Group 可以发送多个 Topic 消息；
　　　　Producer  Group  作用如下：
　　　　　　1. 标识一类 Producer，多个生产者可以并行发消息，提高性能；
　　　　　　2. 可以通过运维工具查询这个发送消息应用下有多个 Producer 实例；
　　　　　　3. 发送分布式事务消息时，如果 Producer 中途意外宕机，Broker 会主动回调 Producer Group 内的任意一台机器来确认事务状态（即事务消息，broker会主动回调生产者，主动的rpc调用producer，做一个check操作）

##### *俊：
> 不应该增加使用官方封装的stream starter 来讲解吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Stream我在文后解释到了。 Steam封装程度过高并不适合作为入门介绍。同时Stream是个门面，很多RocketMQ的独特高级特性是无法通过StreamAPI进行处理的。
PS：作为Stream 的API设计真的很难用

##### **国：
> rocketmq三种部署方式，一般生成上用那种可靠呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 压力小多Master就可以了,压力大多Master-Slave异步双写,数据一致性要求高多Master-Slave同步双写即可.

