<h3 data-nodeid="361393">Flume 概述</h3>



<p data-nodeid="360925">Flume 是 Hadoop 生态圈子中的一个重要组件，在上一课时中提过，它是一个分布式的、高可靠的、高可用的日志采集工具。</p>
<p data-nodeid="360926">Flume 具有基于流式数据的简单灵活的架构，同时兼具高可靠性、高可用机制和故障转移机制。当我们使用 Flume 收集数据的速度超过下游的写入速度时，Flume 会自动做调整，使得数据的采集和推送能够平稳进行。</p>
<p data-nodeid="360927">Flume 支持多路径采集、多管道数据接入和多管道数据输出。数据源可以是 HBase、HDFS 和文本文件，也可以是 Kafka 或者其他的 Flume 客户端。</p>
<h4 data-nodeid="360928">Flume 的组件介绍</h4>
<p data-nodeid="361999">Flume 中有很多组件和概念，下面我把 Flume 中的核心组件一一进行介绍：</p>
<p data-nodeid="362000" class=""><img src="https://s0.lgstatic.com/i/image/M00/38/E2/Ciqc1F8ejV6Aa7StAAAyjk9Eo1Y114.png" alt="Drawing 0.png" data-nodeid="362004"></p>


<ul data-nodeid="360931">
<li data-nodeid="360932">
<p data-nodeid="360933">Client：客户端，用来运行 Flume Agent。</p>
</li>
<li data-nodeid="360934">
<p data-nodeid="360935">Event：Flume 中的数据单位，可以是一行日志、一条消息。</p>
</li>
<li data-nodeid="360936">
<p data-nodeid="360937">Agent：代表一个独立的 Flume 进程，包含三个组件：Source、Channel 和 Sink。</p>
</li>
<li data-nodeid="360938">
<p data-nodeid="360939">Source：数据的收集入口，用来获取 Event 并且传递给 Channel。</p>
</li>
<li data-nodeid="360940">
<p data-nodeid="360941">Channel：Event 的一个临时存储，是数据的临时通道，可以认为是一个队列。</p>
</li>
<li data-nodeid="360942">
<p data-nodeid="360943">Sink：从 Channel 中读取 Event，将 Event 中的数据传递给下游。</p>
</li>
<li data-nodeid="360944">
<p data-nodeid="360945">Flow：一个抽象概念，可以认为是一个从 Source 到达 Sink 的数据流向图。</p>
</li>
</ul>
<h3 data-nodeid="360946">Flume 本地环境搭建</h3>
<p data-nodeid="360947">我们在 Flume 的 <a href="http://archive.apache.org/dist/flume/" data-nodeid="361020">官网</a>下载安装包，在这里下载一个 1.8.0 的稳定版本，然后进行解压：</p>
<pre class="lang-java" data-nodeid="363070"><code data-language="java">tar zxf apache-flume-<span class="hljs-number">1.8</span>.<span class="hljs-number">0</span>-bin.tar.gz 
</code></pre>
<p data-nodeid="363071" class=""><img src="https://s0.lgstatic.com/i/image/M00/38/E2/Ciqc1F8ejXGAQBp_AASfWl6sP68860.png" alt="Drawing 1.png" data-nodeid="363074"></p>




<p data-nodeid="360950">可以看到有几个关键的目录，其中 conf/ 目录则是我们存放配置文件的目录。</p>
<h4 data-nodeid="360951">Flume 测试</h4>
<p data-nodeid="360952">我们在下载 Flume 后，需要进行测试，可以通过监听本地的端口输入，并且在控制台进行打印。</p>
<p data-nodeid="360953">首先，需要修改 conf/ 目录下的&nbsp;flume-env.sh，在里面配置 JAVA_HOME 等配置：</p>
<pre class="lang-powershell" data-nodeid="360954"><code data-language="powershell"><span class="hljs-built_in">cd</span>&nbsp;/usr/local/apache<span class="hljs-literal">-flume</span><span class="hljs-literal">-1</span>.<span class="hljs-number">8.0</span><span class="hljs-literal">-bin</span>/conf 
<span class="hljs-built_in">cp</span>　flume<span class="hljs-literal">-env</span>.sh.template　flume<span class="hljs-literal">-env</span>.sh 
</code></pre>
<p data-nodeid="360955">然后在 flume-env.sh 里面设置 JAVA_HOME 和 FLUME_CLASSPATH 变量：</p>
<pre class="lang-powershell" data-nodeid="360956"><code data-language="powershell">export&nbsp;JAVA_HOME=/usr/local/java/jdk1.<span class="hljs-number">8</span> 
FLUME_CLASSPATH=<span class="hljs-string">"/usr/local/apache-flume-1.8.0-bin/"</span> 
</code></pre>
<p data-nodeid="360957">创建一个配置文件 nc_logger.conf :</p>
<pre class="lang-java" data-nodeid="368549"><code data-language="java">vim nc_logger.conf 
</code></pre>






<p data-nodeid="360959">更改配置如下：</p>
<pre class="lang-powershell" data-nodeid="360960"><code data-language="powershell"><span class="hljs-comment"># 定义这个 agent 中各组件的名字 </span>
a1.sources = r1 
a1.sinks = k1 
a1.channels = c1 
<span class="hljs-comment"># 描述和配置 source 组件：r1 </span>
a1.sources.r1.type = netcat 
a1.sources.r1.bind = localhost 
a1.sources.r1.port = <span class="hljs-number">9000</span> 
<span class="hljs-comment"># 描述和配置 sink 组件：k1 </span>
a1.sinks.k1.type = logger 
<span class="hljs-comment"># 描述和配置channel组件，此处使用是内存缓存的方式 </span>
a1.channels.c1.type = memory 
a1.channels.c1.capacity = <span class="hljs-number">1000</span> 
a1.channels.c1.transactionCapacity = <span class="hljs-number">100</span> 
<span class="hljs-comment"># 描述和配置 source channel sink 之间的连接关系 </span>
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1 
</code></pre>
<p data-nodeid="360961">我们使用如下命令启动一个 Flume Agent:</p>
<pre class="lang-powershell" data-nodeid="360962"><code data-language="powershell">bin/flume<span class="hljs-literal">-ng</span> agent  
<span class="hljs-literal">-c</span> conf  
<span class="hljs-operator">-f</span> conf/nc_logger.conf  
<span class="hljs-literal">-n</span> a1 <span class="hljs-literal">-Dflume</span>.root.logger=INFO,console 
</code></pre>
<p data-nodeid="360963">其中有几个关键的命令：</p>
<ul data-nodeid="360964">
<li data-nodeid="360965">
<p data-nodeid="360966">–conf (-c) 用来指定配置文件夹路径；</p>
</li>
<li data-nodeid="360967">
<p data-nodeid="360968">–conf-file(-f) 用来指定采集方案文件；</p>
</li>
<li data-nodeid="360969">
<p data-nodeid="360970">–name(-n) 用来指定 agent 名字；</p>
</li>
<li data-nodeid="360971">
<p data-nodeid="360972">-Dflume.root.logger=INFO,console 开启 flume 日志输出到终端。</p>
</li>
</ul>
<p data-nodeid="360973">用 nc 命令打开本地 9000 端口：</p>
<pre class="lang-plain" data-nodeid="370794"><code data-language="plain">nc localhost 9000 
</code></pre>
<p data-nodeid="370795" class=""><img src="https://s0.lgstatic.com/i/image/M00/38/EE/CgqCHl8ejbqACoRbAAAux1bCItY816.png" alt="Drawing 2.png" data-nodeid="370798"></p>






























































<p data-nodeid="371403">向端口输入几个单词，可以在另一端的控制台看到输出结果，如下图所示：</p>
<p data-nodeid="371404" class=""><img src="https://s0.lgstatic.com/i/image/M00/38/E3/Ciqc1F8ejdiAXvsmAAMGjSgfLtE499.png" alt="Drawing 3.png" data-nodeid="371408"></p>


<p data-nodeid="360978">我们可以看到 9000 端口中输入的数据已经被打印出来了。</p>
<h3 data-nodeid="372013">Flume + Kafka 整合</h3>
<p data-nodeid="372014" class=""><img src="https://s0.lgstatic.com/i/image/M00/38/EE/CgqCHl8ejeCAA124AACyS8QBaJI555.png" alt="Drawing 4.png" data-nodeid="372018"></p>


<p data-nodeid="360981">整体整合思路为，我们的两个 Flume Agent 分别部署在两台 Web 服务器上，用来采集两台服务器的业务日志，并且 Sink 到另一台 Flume Agent 上，然后将数据 Sink 到 Kafka 集群。在这里需要配置三个 Flume Agent。</p>
<p data-nodeid="360982">首先在 Flume Agent 1 和 Flume Agent 2 上创建配置文件，具体如下。</p>
<p data-nodeid="360983">修改 source、channel 和 sink 的配置，vim log_kafka.conf 代码如下：</p>
<pre class="lang-powershell" data-nodeid="360984"><code data-language="powershell"><span class="hljs-comment"># 定义这个 agent 中各组件的名字 </span>
a1.sources = r1 
a1.sinks = k1 
a1.channels = c1 
<span class="hljs-comment"># source的配置，监听日志文件中的新增数据 </span>
a1.sources.r1.type = exec 
a1.sources.r1.command  = tail <span class="hljs-operator">-F</span> /home/logs/access.log 
<span class="hljs-comment">#sink配置，使用avro日志做数据的消费 </span>
a1.sinks.k1.type = avro 
a1.sinks.k1.hostname = flumeagent03 
a1.sinks.k1.port = <span class="hljs-number">9000</span> 
<span class="hljs-comment">#channel配置，使用文件做数据的临时缓存 </span>
a1.channels.c1.type = file 
a1.channels.c1.checkpointDir = /home/temp/flume/checkpoint 
a1.channels.c1.dataDirs = /home/temp/flume/<span class="hljs-keyword">data</span> 
<span class="hljs-comment">#描述和配置 source channel sink 之间的连接关系 </span>
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c 
</code></pre>
<p data-nodeid="360985">上述配置会监听 /home/logs/access.log 文件中的数据变化，并且将数据 Sink 到 flumeagent03 的 9000 端口。</p>
<p data-nodeid="360986">然后我们分别启动 Flume Agent 1 和 Flume Agent 2，命令如下：</p>
<pre class="lang-powershell" data-nodeid="360987"><code data-language="powershell"><span class="hljs-variable">$</span> flume<span class="hljs-literal">-ng</span> agent  
<span class="hljs-literal">-c</span> conf  
<span class="hljs-literal">-n</span> a1  
<span class="hljs-operator">-f</span> conf/log_kafka.conf &gt;/dev/null <span class="hljs-number">2</span>&gt;&amp;<span class="hljs-number">1</span> &amp; 
</code></pre>
<p data-nodeid="360988">第三个 Flume Agent 用来接收上述两个 Agent 的数据，并且发送到 Kafka。我们需要启动本地 Kafka，并且创建一个名为 log_kafka 的 Topic。</p>
<p data-nodeid="360989">然后，我们创建 Flume 配置文件，具体如下。</p>
<p data-nodeid="360990">修改 source、channel 和 sink 的配置，vim flume_kafka.conf 代码如下：</p>
<pre class="lang-powershell" data-nodeid="360991"><code data-language="powershell"><span class="hljs-comment"># 定义这个 agent 中各组件的名字 </span>
a1.sources = r1 
a1.sinks = k1 
a1.channels = c1 
<span class="hljs-comment">#source配置 </span>
a1.sources.r1.type = avro 
a1.sources.r1.bind = <span class="hljs-number">0.0</span>.<span class="hljs-number">0.0</span> 
a1.sources.r1.port = <span class="hljs-number">9000</span> 
<span class="hljs-comment">#sink配置 </span>
a1.sinks.k1.type = org.apache.flume.sink.kafka.KafkaSink 
a1.sinks.k1.topic = log_kafka 
a1.sinks.k1.brokerList = <span class="hljs-number">127.0</span>.<span class="hljs-number">0.1</span>:<span class="hljs-number">9092</span> 
a1.sinks.k1.requiredAcks = <span class="hljs-number">1</span> 
a1.sinks.k1.batchSize = <span class="hljs-number">20</span> 
<span class="hljs-comment">#channel配置 </span>
a1.channels.c1.type = memory 
a1.channels.c1.capacity = <span class="hljs-number">1000</span> 
a1.channels.c1.transactionCapacity = <span class="hljs-number">100</span> 
<span class="hljs-comment">#描述和配置 source channel sink 之间的连接关系 </span>
a1.sources.r1.channels = c1 
a1.sinks.k1.channel = c1     
</code></pre>
<p data-nodeid="360992">配置完成后，我们启动该 Flume Agent：</p>
<pre class="lang-powershell" data-nodeid="360993"><code data-language="powershell"><span class="hljs-variable">$</span> flume<span class="hljs-literal">-ng</span> agent  
<span class="hljs-literal">-c</span> conf  
<span class="hljs-literal">-n</span> a1  
<span class="hljs-operator">-f</span> conf/flume_kafka.conf &gt;/dev/null <span class="hljs-number">2</span>&gt;&amp;<span class="hljs-number">1</span> &amp; 
</code></pre>
<p data-nodeid="360994">我们在第 24 课时“Flink 消费 Kafka 数据开发” 中详细讲解了 Flink 消费 Kafka 数据的开发。当 Flume Agent 1 和 2 中监听到新的日志数据后，数据就会被 Sink 到 Kafka 指定的 Topic，我们就可以消费 Kafka 中的数据了。</p>
<h3 data-nodeid="360995">总结</h3>
<p data-nodeid="372325">这一课时首先介绍了 Flume 和 Kafka 的整合和部署，然后详细介绍了 Flume 的组件并且搭建了 Flume 的本地环境进行测试，最后介绍了 Flume 和 Kafka 整合的配置文件编写。通过本课时的学习，相信你对 Flume 有了深入了解，在实际应用中可以正确配置 Flume Agent。</p>

---

### 精选评论

##### *帅：
> 为什么agent1和agent2不直接sink到kafka呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在实际生产环境中，我们需要Flume Consolidation Agent做Flume的高可用

##### **冰：
> 还有个问题，你说第三个FLUME是做高可用，这一个单节点如何高可用了？kafka是集群的，不是更符合高可用的要求吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Consolidation这种模式，通常用来处理日志数据来源是几十上百个节点的这种情况，故障发生主要在collector。你考虑的很对，可以通过kafka这种中间件实现故障转移。因为flume高可用方案重点就不再agent上。

##### **冰：
> 老师，这里我有个问题，source采用tail -f 读取更新的数据，这种重启服务时会有数据丢失吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; tail -f 也就是我们在日常环境做做测试，实际生产中是不会这么玩的。

##### **伯：
> 您好，我想咨询一下Flume配置的时候为什么要启动第三个FLUME而不是前面两个Flume收集到的日志直接推送到kafka中。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 小数据量规模是可以的，但是生产商一般会做Flume的高可用。

