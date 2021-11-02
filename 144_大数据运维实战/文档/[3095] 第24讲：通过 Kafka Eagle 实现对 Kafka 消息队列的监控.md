<p data-nodeid="86702">在企业实际应用中，如果业务比较复杂，那么管理的 Consumer Group 和 Topic 数也会随之增加，此时如果再使用 Kafka 提供的命令行工具，可能会出现力不从心的感觉。</p>



<p data-nodeid="85499">因此，我们需要一款更加智能的 Kafka 监控系统，目前企业使用比较多的有 Kafka Manager、Kafka Eagle 等。</p>
<h3 data-nodeid="88595" class="">Kafka Eagle 介绍</h3>




<p data-nodeid="85501">由于 Kafka Eagle 比 Kafka Manager 更简单、更好用，所以这一课时将重点介绍 Kafka Eagle 这款 Kafka 可视化管理工具。</p>
<p data-nodeid="85502"><strong data-nodeid="85610">Kafka Eagle</strong> 是一款用来管理 Kafka 集群的可视化工具，它可以支持管理多个 Kafka 集群，还可以管理 Kafka 的 Topic（查看、删除、创建等），也可以对消费者状态进行监控，并可实现消息阻塞告警、集群健康状态检测等功能。</p>
<h3 data-nodeid="90464" class="">Kafka Eagle 安装</h3>




<p data-nodeid="85504"><a href="https://www.kafka-eagle.org/" data-nodeid="85617">点击这里下载 Kafka-Eagle 安装包</a>，我下载的是 kafka-eagle-bin-2.0.0.tar.gz 版本，下载完成后，将安装包解压到 172.16.213.151 主机上，即可完成安装，操作如下：</p>
<pre class="lang-dart" data-nodeid="94845"><code data-language="dart">[root<span class="hljs-meta">@nnmaster</span> ~]#tar zxvf kafka-eagle-bin<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>.tar.gz
[root<span class="hljs-meta">@nnmaster</span> ~]#cd kafka-eagle-bin<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>
[root<span class="hljs-meta">@nnmaster</span> kafka-eagle-bin<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>]# tar zxvf kafka-eagle-web<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>-bin.tar.gz -C /usr/local
[root<span class="hljs-meta">@nnmaster</span> kafka-eagle-bin<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>]# mv /usr/local/kafka-eagle-web<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>   /usr/local/kafka-eagle
[root<span class="hljs-meta">@nnmaster</span> kafka-eagle-bin<span class="hljs-number">-2.0</span><span class="hljs-number">.0</span>]# chown -R kafka:kafka /usr/local/kafka-eagle
</code></pre>










<p data-nodeid="85506">上面这个操作步骤是解压 Kafka Eagle，然后安装到 /usr/local 目录下，最后，授权 kafka-eagle 目录为 Kafka 用户权限，后面的管理配置操作都通过 Kafka 用户来完成。</p>
<p data-nodeid="85507">Kafka Eagle 安装完成后，还需要配置 Java 环境变量和设置 Kafka Eagle 目录，将下面内容加入 /home/kafka/.bashrc 文件中：</p>
<pre class="lang-js" data-nodeid="98533"><code data-language="js"><span class="hljs-keyword">export</span> KE_HOME=<span class="hljs-regexp">/usr/</span>local/kafka-eagle
<span class="hljs-keyword">export</span> PATH=$PATH:$KE_HOME/bin
<span class="hljs-keyword">export</span> JAVA_HOME=<span class="hljs-regexp">/opt/</span>bigdata/jdk
</code></pre>








<p data-nodeid="85509">配置完成，执行 source /home/kafka/.bashrc 命令，使配置生效。</p>
<p data-nodeid="85510">最后一个步骤，需要安装一个数据库，因为 Kafka Eagle 需要数据库的支持，这里我采用 MySQL 数据库，版本为 mysql5.7.30，不过安装过程这里不再介绍。安装完成后，需要做如下配置和授权：</p>
<pre class="lang-java" data-nodeid="100377"><code data-language="java">mysql&gt; create database ke character set utf8;
mysql&gt; create user <span class="hljs-string">'root'</span>@<span class="hljs-string">'172.16.213.151'</span> identified by <span class="hljs-string">'xxxxxx'</span>;
mysql&gt; grant all privileges on ke.* to <span class="hljs-string">'root'</span>@<span class="hljs-string">'172.16.213.151'</span>;
mysql&gt; flush privileges;
</code></pre>




<p data-nodeid="85512">在这个步骤中，首先创建了一个 ke 库，是给 Kafka Eagle 使用的，然后创建了一个 root 用户，并给 root 用户授权给管理 ke 库，最后刷新配置生效。</p>
<h3 data-nodeid="102221" class="">配置 Kafka Eagle 监控 Kafka 集群</h3>




<p data-nodeid="85514">Kafka Eagle 的配置文件位于 $KE_HOME/conf/ 目录下，配置文件为 system-config.properties，配置完成的文件内容如下：</p>
<pre class="lang-java" data-nodeid="104042"><code data-language="java">kafka.eagle.zk.cluster.alias=cluster1
cluster1.zk.list=<span class="hljs-number">172.16</span>.<span class="hljs-number">213.152</span>:<span class="hljs-number">2181</span>,<span class="hljs-number">172.16</span>.<span class="hljs-number">213.138</span>:<span class="hljs-number">2181</span>,<span class="hljs-number">172.16</span>.<span class="hljs-number">213.80</span>:<span class="hljs-number">2181</span>
kafka.zk.limit.size=<span class="hljs-number">25</span>
kafka.eagle.webui.port=<span class="hljs-number">8048</span>
cluster1.kafka.eagle.offset.storage=kafka
kafka.eagle.metrics.charts=<span class="hljs-keyword">true</span>
kafka.eagle.metrics.retain=<span class="hljs-number">15</span>
kafka.eagle.sql.topic.records.max=<span class="hljs-number">5000</span>
kafka.eagle.sql.fix.error=<span class="hljs-keyword">true</span>
kafka.eagle.topic.token=keadmin
kafka.eagle.driver=com.mysql.jdbc.Driver
kafka.eagle.url=jdbc:mysql:<span class="hljs-comment">//172.16.213.151:3306/ke?useUnicode=true&amp;characterEncoding=UTF-8&amp;zeroDateTimeBehavior=convertToNull</span>
kafka.eagle.username=root
kafka.eagle.password=<span class="hljs-number">123456</span>
</code></pre>




<p data-nodeid="85516">其中，每个配置项含义如下。</p>
<ul data-nodeid="85517">
<li data-nodeid="85518">
<p data-nodeid="85519">kafka.eagle.zk.cluster.alias：用来指定需要配置的 Kafka 集群名称，可以配置多个，用逗号分隔。</p>
</li>
<li data-nodeid="85520">
<p data-nodeid="85521">cluster1.zk.list：用来配置多个 Kafka 集群所对应的 ZooKeeper 集群列表。注意这个写法，这是配置 cluster1 集群，如果有多个集群，依次填写每个 Kafka 集群对应的 ZooKeeper 集群列表即可。</p>
</li>
<li data-nodeid="85522">
<p data-nodeid="85523">kafka.zk.limit.size：设置 ZooKeeper 客户端最大连接数。</p>
</li>
<li data-nodeid="85524">
<p data-nodeid="85525">kafka.eagle.webui.port：设置 Kafka Eagle 的 Web 访问端口，默认是 8048。</p>
</li>
<li data-nodeid="85526">
<p data-nodeid="85527">cluster1.kafka.eagle.offset.storage：设置存储消费信息的类型，一般在 Kafka0.9 版本之前，消费信息会默认存储在 ZooKeeper 中，所以存储类型设置 zookeeper 即可。如果你使用的是 Kafka0.10 之后的版本，那么，消费者信息默认存储在 Kafka 中，所以存储类型需要设置为 kafka。同时，在使用消费者 API 时，尽量保证客户端 Kafka API 版本和 Kafka 服务端的版本一致。</p>
</li>
<li data-nodeid="85528">
<p data-nodeid="85529">kafka.eagle.metrics.charts：设置是否开启 Kafka Eagle 的监控趋势图，默认是不启用方式，如果需要查看 Kafka 监控趋势图，则需要设置为 true。</p>
</li>
<li data-nodeid="85530">
<p data-nodeid="85531">kafka.eagle.metrics.retain：设置数据默认保留时间，这里的 15 表示 15 天。</p>
</li>
<li data-nodeid="85532">
<p data-nodeid="85533">kafka.eagle.sql.fix.error：在使用 KSQL 查询 Topic 时，如果遇到错误，可以开启这个属性，默认不开启。</p>
</li>
<li data-nodeid="85534">
<p data-nodeid="85535">kafka.eagle.sql.topic.records.max：KSQL 查询 Topic 数据默认是最新的 5000 条，如果在使用 KSQL 查询的过程中出现异常，可以将 kafka.eagle.sql.fix.error 的值设置为 true，Kafka Eagle 会在系统中自动修复错误。1</p>
</li>
<li data-nodeid="85536">
<p data-nodeid="85537">kafka.eagle.topic.token：设置在删除 Kafka Topic 时的 Token 令牌，需要记住这个值。</p>
</li>
<li data-nodeid="85538">
<p data-nodeid="85539">kafka.eagle.driver：设置连接数据库的驱动信息。</p>
</li>
<li data-nodeid="85540">
<p data-nodeid="85541">kafka.eagle.url：设置 jdbc 连接 MySQL 数据库的地址。</p>
</li>
<li data-nodeid="85542">
<p data-nodeid="85543">kafka.eagle.username：设置连接到 MySQL 数据库的用户名。</p>
</li>
<li data-nodeid="85544">
<p data-nodeid="85545">kafka.eagle.password：设置连接到 MySQL 数据库的用户密码。</p>
</li>
</ul>
<p data-nodeid="85546">上面配置中，我开启了 Kafka Eagle 的监控趋势图，因此，还需要开启 Kafka 系统的 JMX 端口，默认 Kafka 没有开启 JMX 端口，可修改 Kafka 启动脚本 kafka-server-start.sh，添加如下内容：</p>
<pre class="lang-java" data-nodeid="106772"><code data-language="java"><span class="hljs-keyword">if</span> [ <span class="hljs-string">"x$KAFKA_HEAP_OPTS"</span> = <span class="hljs-string">"x"</span> ]; then
    export KAFKA_HEAP_OPTS=<span class="hljs-string">"-server -Xms2G -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreads=5 -XX:InitiatingHeapOccupancyPercent=70"</span>
    export JMX_PORT=<span class="hljs-string">"9999"</span>
fi
</code></pre>






<p data-nodeid="85548">注意，这个 9999 端口可修改为任意其他端口。在 Kafka 集群中所有节点添加这个配置，添加完成后，重启所有 Kafka 集群节点以使配置生效。</p>
<h3 data-nodeid="108592" class="">Kafka Eagle 使用</h3>




<h4 data-nodeid="110389" class="">1.启动 Kafka Eagle 服务</h4>




<p data-nodeid="85551">所有配置完成后，就可以启动 Kafka Eagle 服务了，执行如下命令启动服务：</p>
<pre class="lang-java" data-nodeid="110833"><code data-language="java">[kafka<span class="hljs-meta">@nnmaster</span> kafka-eagle]$  cd /usr/local/kafka-eagle/bin
[kafka<span class="hljs-meta">@nnmaster</span> bin]$ ./ke.sh  start
[kafka<span class="hljs-meta">@nnmaster</span> bin]$ jps
<span class="hljs-number">7557</span> Jps
<span class="hljs-number">15855</span> KafkaEagle
</code></pre>

<p data-nodeid="85553">服务启动成功后，可以看到有个 KafkaEagle 进程标识，表示启动成功，要查看 Kafka Eagle 启动日志，可访问 Kafka Eagle 的 logs 目录，主要查看 log.log、error.log 及 ke_console.out 三个文件。如果启动失败，文件中会有失败信息及失败原因。</p>
<p data-nodeid="85554">要关闭 Kafka Eagle 服务，执行“./ke.sh  stop”即可。</p>
<p data-nodeid="85555">Kafka Eagle 服务启动后，即可打开 Web 界面，默认 Web 登录用户名为 admin，密码为 123456，登录端口为 8048，访问 <a href="http://172.16.213.151:8048" data-nodeid="85664">http://172.16.213.151:8048</a> 地址即可打开登录界面。</p>
<h4 data-nodeid="112605" class="">2.Kafka Eagle 的使用</h4>




<p data-nodeid="113471">Kafka Eagle 的使用比较简单，下面是一个功能预览界面图：</p>
<p data-nodeid="113472" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWa2AQ597AADfl30QncI077.png" alt="Drawing 0.png" data-nodeid="113476"></p>


<p data-nodeid="85559">在上图中，对于左侧导航栏，我已经将每个栏目的功能做了标注，整体分为 5 个部分，分别是全局视图、Kakfa 消息管理、Kafka 集群监控、Kafka 监控告警及系统设置。</p>
<p data-nodeid="114341">点击上图导航栏中的 BScreen 链接，即可进入下图界面：</p>
<p data-nodeid="114342" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWbeAORQBAAlDnMWCVwI984.png" alt="Drawing 1.png" data-nodeid="114346"></p>


<p data-nodeid="85562">此界面是个 Kafka 实时状态监控图，主要展示了消息生产的状态和消费状态，数据实时更新，非常炫酷。</p>
<p data-nodeid="115211">接着前面的功能预览界面图，在此图右侧部分，显示了 Kafka 集群 Brokers 的数量、Topic 的数量、ZooKeeper 集群的节点数及消费者数量，点击 Brokers 的数量链接，显示下图界面：</p>
<p data-nodeid="115212" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/1C/Ciqc1F8RWcGAJCXtAADdhhAt69Y878.png" alt="Drawing 2.png" data-nodeid="115216"></p>


<p data-nodeid="85565">此界面显示了 Kafka 集群的状态信息，主要是 Kakfa 占用的系统 CPU、内存等资源信息，要获得这些信息，需要在 Kafka 上开启 JMX 端口，这里设置的 JMX 端口为 9999。</p>
<p data-nodeid="116081">接着，回到功能预览界面图，再点击 Topic 的链接，来到下图所示的界面：</p>
<p data-nodeid="116082" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWcmAWsZQAADQaJT1aIw478.png" alt="Drawing 3.png" data-nodeid="116086"></p>


<p data-nodeid="116951">此界面显示了 Kafka 集群中所有的 Topic 信息，包含 partitions 数量、数据偏差等，点击任意一个 Topic 链接，来到下图所示的界面：</p>
<p data-nodeid="116952" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWc-AV1dLAADXF7wAPKk061.png" alt="Drawing 4.png" data-nodeid="116956"></p>


<p data-nodeid="85570">此界面主要显示了某个 Topic 中目前的消息数、容量以及每个 Partition 中存储的消息数。</p>
<p data-nodeid="118905">最后，再点击功能预览界面图中的消费者链接，来到下图所示的界面：</p>
<p data-nodeid="118906" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWeCAd6KVAACdPx7Jf0Q608.png" alt="Drawing 5.png" data-nodeid="118910"></p>






<p data-nodeid="119775">此界面主要显示了目前有哪些消费组，可以看到这里有一个 Logstash 消费者组，点击这个消费者组，然后点击 Runing 按钮，来到如下的界面：</p>
<p data-nodeid="119776" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWeeAXM3OAAEgJuuQ3KU227.png" alt="Drawing 6.png" data-nodeid="119780"></p>


<p data-nodeid="85575">此界面主要显示了目前消费者消费 Nginxlogs 这个 Topic 的状态信息，重点关注 Lag 列的信息，此信息表示消费延时，如果延时值过大，则表示消费速度过慢，需要引起注意。</p>
<p data-nodeid="120645">下面再来看下 Kafka Eagle 提供的 KSQL 功能，如下图所示：</p>
<p data-nodeid="120646" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/1C/Ciqc1F8RWfaAairZAAEEUcme4_0353.png" alt="Drawing 7.png" data-nodeid="120650"></p>


<p data-nodeid="85578" class="">此界面中，右边的 select 语句，nginxlogs 表示 topic 的名称，“<code data-backticks="1" data-nodeid="85708">partition</code> in (0)”表示查询哪个 partition 中的数据，最后的“limit 5”表示显示前 5 条消息。</p>
<p data-nodeid="121515">接着再来看下 Kafka 的监控指标，如下图所示：</p>
<p data-nodeid="121516" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/1C/Ciqc1F8RWfyAFSK-AAFwH8ENnKw658.png" alt="Drawing 8.png" data-nodeid="121520"></p>


<p data-nodeid="85581">上图曲线显示了每分钟推送到 Kafka 的消息数，另外还可以看到有很多其他监控指标，比如每秒生产多少数据量、每秒消费多少数据量等。</p>
<p data-nodeid="122385">最后，再来看下如何配置监控告警，首先在告警栏中选择创建一个告警组，如下图所示：</p>
<p data-nodeid="122386" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/27/CgqCHl8RWgSAHxTjAACkzctvXWs371.png" alt="Drawing 9.png" data-nodeid="122390"></p>


<p data-nodeid="85584">此界面设置告警组名称、告警方式和告警接口，支持钉钉、微信和邮件告警，推荐钉钉告警，配置比较简单，直接给出一个告警接口地址即可。</p>
<p data-nodeid="123255">接着，配置一个消费者延时告警，如下图所示：</p>
<p data-nodeid="123256" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/28/CgqCHl8RWguAUvmYAACq3eB4nx8993.png" alt="Drawing 10.png" data-nodeid="123260"></p>


<p data-nodeid="85587">此界面中主要配置的是消费延时值，上面配置为 1，也就是说延时值超过 1 就进行告警，这个值要根据实际情况进行设置。在海量数据环境下（上亿条数据），有几千条消息延时，其实也是正常的，而在小量数据环境下（几十万条数据），延时消息超过百条，可能就是消费延时了，需要关注延时的原因。</p>
<p data-nodeid="124125">最后，再来看下如何设置集群的告警，如下图所示：</p>
<p data-nodeid="124126" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/28/CgqCHl8RWhOAVNSkAACb9qXjWuY931.png" alt="Drawing 11.png" data-nodeid="124130"></p>


<p data-nodeid="85590">此界面中，配置的是对 Kafka 集群的告警，也可以设置 ZooKeeper、Topic、Producer 等的告警，告警可以设置告警级别、告警次数及告警组等信息。</p>
<p data-nodeid="124995">所有配置完成后，就可以实现故障告警了，下图是一个钉钉告警截图：</p>
<p data-nodeid="124996" class=""><img src="https://s0.lgstatic.com/i/image/M00/34/1C/Ciqc1F8RWhuAWMiFAAEaw2H9zcg810.png" alt="Drawing 12.png" data-nodeid="125000"></p>


<p data-nodeid="85593">从图中可以看出，此告警是消费延时导致的，我们设置的最大消费延时为 1，而目前延时是 793，这已经是第三次告警了，后面将不再进行告警，因为我配置的最大告警次数为 3。</p>
<h3 data-nodeid="85594">小结</h3>
<p data-nodeid="85595">本课时主要讲解了 Kafka Eagle 的安装、配置与使用，其是目前最流行的 Kafka 监控利器，很多大企业都在使用它来监控和管理 Kafka，快速、熟练使用 Kafka Eagle 是本课程的重点。</p>

---

### 精选评论


