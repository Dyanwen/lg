<h3 data-nodeid="21195" class="">Filebeat 简介</h3>
<p data-nodeid="21196">Filebeat 是一个开源的文本日志收集器，Elastic 公司 Beats 数据采集产品的一个子产品，采用 Go 语言开发。一般安装在业务服务器上作为代理来监测日志目录或特定的日志文件，并把它们发送到 Logstash、Elasticsearch、Redis 或 Kafka 等。<a href="https://www.elastic.co/downloads/beats" data-nodeid="21201">点击官方地址下载各个版本的 Filebeat</a>。</p>



<h3 data-nodeid="19302" class="">Filebeat 架构与运行原理</h3>
<p data-nodeid="22439">Filebeat 是一个轻量级的日志监测、传输工具，它最大的特点是性能稳定、配置简单、占用系统资源很少，这也是强烈推荐 Filebeat 的原因。下图是官方给出的 Filebeat 架构图：</p>
<p data-nodeid="22440" class=""><img src="https://s0.lgstatic.com/i/image/M00/22/B4/Ciqc1F7saKKAQ8YZAAEBev585Hk521.png" alt="image1.png" data-nodeid="22444"></p>





<p data-nodeid="15819">从图中可以看出，Filebeat 主要由两个组件构成：<strong data-nodeid="15941">Prospector（探测器）和 Harvester（收集器）</strong>，这两类组件一起协作完成 Filebeat 的工作。</p>
<p data-nodeid="15820">其中，收集器负责进行单个文件的内容收集，在运行过程中，每一个收集器会对一个文件逐行进行内容读取，并且把读写到的内容发送到配置的 output 中。当收集器开始进行文件的读取后，将会负责这个文件的打开和关闭操作，因此，在收集器运行过程中，文件都处于打开状态。如果在收集过程中，删除了这个文件或者是对文件进行了重命名，Filebeat 依然会继续对这个文件进行读取，这时候将会一直占用着文件所对应的磁盘空间，直到收集器关闭。</p>
<p data-nodeid="15821">探测器负责管理收集器，它会找到所有需要进行读取的数据源，然后交给收集器进行内容收集，如果 input type 配置的是 log 类型，探测器将会去配置路径下查找所有能匹配上的文件，然后为每一个文件创建一个收集器。</p>
<p data-nodeid="15822">因此，filebeat 的工作流程为：当开启 filebeat 程序的时候，它会启动一个或多个探测器去检测指定的日志目录或文件，对于探测器找出的每一个日志文件，filebeat 会启动收集进程，每一个收集进程读取一个日志文件的内容，然后将这些日志数据发送到后台处理程序（spooler），后台处理程序会集合这些事件，最后发送集合的数据到 output 指定的目的地。</p>
<h3 data-nodeid="26223" class="">安装并配置 Filebeat</h3>
<h4 data-nodeid="26849" class="">1. 为什么要使用 Filebeat</h4>










<p data-nodeid="15824">Logstash 功能虽然强大，但是它依赖 Java，在数据量大的时候，Logstash 进程会消耗过多的系统资源，这将严重影响业务系统的性能。而 Filebeat 就是一个完美的替代者，它也是 Beat 成员之一，基于 Go 语言，没有任何依赖，配置文件简单、格式明了，同时，Filebeat 比 Logstash 更加轻量级，所以占用系统资源极少，非常适合安装在生产机器上。这也是推荐使用 Filebeat 来作为日志收集软件的原因。</p>
<h4 data-nodeid="29318" class="">2. 下载与安装 Filebeat</h4>




<p data-nodeid="15826"><a href="https://www.elastic.co/downloads/beats/filebeat" data-nodeid="15958">你可以从 Elastic 官网获取 Filebeat 安装包</a>，这里下载的版本是 filebeat-7.7.1-linux-x86_64.tar.gz，将下载下来的安装包直接解压到一个路径下即可完成安装。这里我将 Filebeat 安装到 filebeatserver 主机（172.16.213.156）上，设定将 Filebeat 安装到 /usr/local 目录下，基本操作过程如下：</p>
<pre class="lang-dart" data-nodeid="34513"><code data-language="dart">[root<span class="hljs-meta">@filebeatserver</span> ~]# tar -zxvf filebeat<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span>-linux-x86_64.tar.gz -C /usr/local
[root<span class="hljs-meta">@filebeatserver</span> ~]# mv /usr/local/filebeat<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span>-linux-x86_64 /usr/local/filebeat
</code></pre>









<h4 data-nodeid="36957" class="">3. 配置 Filebeat</h4>




<p data-nodeid="15829">由上一步可知，Filebeat 的配置文件目录为 /usr/local/filebeat/filebeat.yml。Filebeat 的配置非常简单，这里仅列出常用的配置项，内容如下：</p>
<pre class="lang-js" data-nodeid="39378"><code data-language="js">filebeat.inputs:
- type: log
  <span class="hljs-attr">enabled</span>: <span class="hljs-literal">true</span>
  <span class="hljs-attr">paths</span>:
   - <span class="hljs-regexp">/var/</span>log/secure
  <span class="hljs-attr">fields</span>:
log_topic: osmessages
filebeat.config.modules:
  path: ${path.config}/modules.d<span class="hljs-comment">/*.yml
  reload.enabled: false
setup.template.settings:
  index.number_of_shards: 1
name: "172.16.213.156"
setup.kibana:
output.kafka:
  enabled: true
  hosts: ["172.16.213.51:9092", "172.16.213.75:9092", "172.16.213.109:9092"]
  version: "0.10"
  topic: '%{[fields][log_topic]}'
  partition.round_robin:
    reachable_only: true
  worker: 2
  required_acks: 1
  compression: gzip
  max_message_bytes: 10000000
logging.level: debug
</span></code></pre>




<p data-nodeid="40361">每个配置项的含义介绍如下：</p>
<p data-nodeid="42382" class=""><img src="https://s0.lgstatic.com/i/image/M00/22/B5/Ciqc1F7sakCAZRRhAAVtPJ5WfBg187.png" alt="01.png" data-nodeid="42385"><br>
<img src="https://s0.lgstatic.com/i/image/M00/22/B5/Ciqc1F7sakqAYIedAAU0BudFkBA057.png" alt="02.png" data-nodeid="42389"><br>
<img src="https://s0.lgstatic.com/i/image/M00/22/C1/CgqCHl7salKAH_SMAAb9p7oShcE030.png" alt="03.png" data-nodeid="42393"></p>








<p data-nodeid="15874">注意，这个配置是将 filbeat 输出到 Kafka 中，因此你需要先配置好 Kafka 集群，并且将 Kafka 中的 topic 设置为可自动创建。</p>
<p data-nodeid="15875">Filebeat 会将收集到的日志自动转成 json 格式输出，因此我们在 Kafka 中看到的日志应该是 json 格式的，该格式会占用部分空间，但是有助于后期对日志的过滤处理。如果你想将 Filebeat 获取到的日志原样输出到 Kafka 中，也是可以的，只需对 output 部分做如下修改：</p>
<pre class="lang-java" data-nodeid="44005"><code data-language="java">output.kafka:
  enabled: <span class="hljs-keyword">true</span>
  hosts: [<span class="hljs-string">"172.16.213.51:9092"</span>, <span class="hljs-string">"172.16.213.75:9092"</span>, <span class="hljs-string">"172.16.213.109:9092"</span>]
  version: <span class="hljs-string">"0.10"</span>
  topic: <span class="hljs-string">'%{[fields][log_topic]}'</span>
  codec.format.string: <span class="hljs-string">'%{[message]}'</span>
  partition.round_robin:
    reachable_only: <span class="hljs-keyword">true</span>
  worker: <span class="hljs-number">2</span>
  required_acks: <span class="hljs-number">1</span>
  compression: gzip
  max_message_bytes: <span class="hljs-number">10000000</span>
</code></pre>




<p data-nodeid="15877">其实就是在 output 部分添加了 codec.format 这个配置，此配置表示指定写入 Kafka 集群的消息格式，后面只添加了 message 列，表示仅输出 json 中 message 字段的数据，而 message 字段就是最原始的日志内容，所以这个配置就是将原始的日志内容原样输出到 Kafka 中。</p>
<h4 data-nodeid="45617" class="">4. 启动 Filebeat 收集日志</h4>




<p data-nodeid="15879">所有配置完成之后，就可以启动 Filebeat，开启收集日志进程了，启动方式如下：</p>
<pre class="lang-dart" data-nodeid="49985"><code data-language="dart">[root<span class="hljs-meta">@filebeatserver</span> ~]# cd /usr/local/filebeat
[root<span class="hljs-meta">@filebeatserver</span> filebeat]# nohup ./filebeat -e -c filebeat.yml &amp;
</code></pre>











<p data-nodeid="15881">这样，就把 Filebeat 进程放到后台运行起来了。启动后，在当前目录下会生成一个 nohup.out 文件，可以查看 Filebeat 启动日志和运行状态。</p>
<h4 data-nodeid="51573" class="">5. Filebeat 输出信息格式解读</h4>




<p data-nodeid="15883">这里以操作系统中 /var/log/secure 文件的日志格式为例，选取一个 SSH 登录系统失败的日志，内容如下：</p>
<pre class="lang-dart" data-nodeid="53138"><code data-language="dart">Jun <span class="hljs-number">11</span> <span class="hljs-number">16</span>:<span class="hljs-number">17</span>:<span class="hljs-number">43</span> localhost sshd[<span class="hljs-number">8160</span>]: Failed password <span class="hljs-keyword">for</span> root from <span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.229</span> port <span class="hljs-number">43860</span> ssh2
</code></pre>




<p data-nodeid="15885">Filebeat 接收到 /var/log/secure 日志后，会将上面日志发送到 Kafka 集群，在 Kafka 任意一个节点上，消费输出日志内容如下：</p>
<pre class="lang-dart" data-nodeid="54702"><code data-language="dart">{<span class="hljs-string">"@timestamp"</span>: <span class="hljs-string">"2020-06-11T08:17:47.848Z"</span>,
  <span class="hljs-string">"@metadata"</span>: {
    <span class="hljs-string">"beat"</span>: <span class="hljs-string">"filebeat"</span>,
    <span class="hljs-string">"type"</span>: <span class="hljs-string">"_doc"</span>,
    <span class="hljs-string">"version"</span>: <span class="hljs-string">"7.7.1"</span>
  },
  <span class="hljs-string">"ecs"</span>: {
    <span class="hljs-string">"version"</span>: <span class="hljs-string">"1.5.0"</span>
  },
  <span class="hljs-string">"host"</span>: {
    <span class="hljs-string">"name"</span>: <span class="hljs-string">"172.16.213.156"</span>
  },
  <span class="hljs-string">"agent"</span>: {
    <span class="hljs-string">"ephemeral_id"</span>: <span class="hljs-string">"a7e4d4f8-6508-40af-97a1-00504522c059"</span>,
    <span class="hljs-string">"hostname"</span>: <span class="hljs-string">"filebeatserver"</span>,
    <span class="hljs-string">"id"</span>: <span class="hljs-string">"958ac24b-c552-44e6-9a4a-1636f15ae0b6"</span>,
    <span class="hljs-string">"version"</span>: <span class="hljs-string">"7.7.1"</span>,
    <span class="hljs-string">"name"</span>: <span class="hljs-string">"172.16.213.156"</span>,
    <span class="hljs-string">"type"</span>: <span class="hljs-string">"filebeat"</span>
  },
  <span class="hljs-string">"log"</span>: {
    <span class="hljs-string">"offset"</span>: <span class="hljs-number">41480</span>,
    <span class="hljs-string">"file"</span>: {
      <span class="hljs-string">"path"</span>: <span class="hljs-string">"/var/log/secure"</span>
    }
  },
  <span class="hljs-string">"message"</span>: <span class="hljs-string">"Jun 11 16:17:43 localhost sshd[8160]: Failed password for root from 172.16.213.229 port 43860 ssh2"</span>,
  <span class="hljs-string">"input"</span>: {
    <span class="hljs-string">"type"</span>: <span class="hljs-string">"log"</span>
  },
  <span class="hljs-string">"fields"</span>: {
    <span class="hljs-string">"log_topic"</span>: <span class="hljs-string">"osmessages"</span>
  }
}
</code></pre>




<p data-nodeid="64826" class="">从这个输出可以看到，输出日志被修改成了 JSON 格式，日志总共分为八个大字段，分别是 @timestamp、ecs、host、agent、log、message、input 和 fileds 字段，每个字段含义如下：</p>



























<ul data-nodeid="15888">
<li data-nodeid="15889">
<p data-nodeid="15890">@timestamp，时间字段，表示读取到该行内容的时间；</p>
</li>
<li data-nodeid="15891">
<p data-nodeid="15892">ecs：ecs 版本字段，这是 Filebeat7 版本新增的一个字段，通过 ECS 可让用户对不同的数据源执行统一的搜索、可视化和分析；</p>
</li>
<li data-nodeid="15893">
<p data-nodeid="15894">host，主机名字段，输出主机名，如果没主机名，输出主机对应的 IP；</p>
</li>
<li data-nodeid="15895">
<p data-nodeid="15896">agent，表示 agent 所在主机的事件信息，比如主机名、IP 等；</p>
</li>
<li data-nodeid="15897">
<p data-nodeid="15898">log，表示读取的日志文件信息及偏移量信息；</p>
</li>
<li data-nodeid="15899">
<p data-nodeid="15900">message，表示真正的日志内容；</p>
</li>
<li data-nodeid="15901">
<p data-nodeid="15902">input，日志输入的类型，可以有多种输入类型，如 Log、Stdin、redis、Docker、TCP/UDP 等；</p>
</li>
<li data-nodeid="15903">
<p data-nodeid="15904">fields，topic 对应的消息字段或自定义增加的字段。</p>
</li>
</ul>
<p data-nodeid="15905">通过 Filebeat 接收到的内容，默认增加了不少字段，但是有些字段对数据分析来说没有太大用处，所以有时候需要删除这些没用的字段。在 Filebeat 配置文件中添加如下配置，即可删除不需要的字段：</p>
<pre class="lang-dart" data-nodeid="66151"><code data-language="dart">processors:
 - drop_fields:
    fields: [<span class="hljs-string">"input"</span>, <span class="hljs-string">"host"</span>, <span class="hljs-string">"agent.type"</span>, <span class="hljs-string">"agent.ephemeral_id"</span>, <span class="hljs-string">"agent.id"</span>, <span class="hljs-string">"agent.version"</span>, <span class="hljs-string">"ecs"</span>]
</code></pre>




<p data-nodeid="69088" class="">这个设置表示删除  input、host、agent.type、agent.ephemeral_id 等指定字段，其中，@timestamp 和 @metadata 字段是不能删除的。<br>
注意这里使用了 processors，Filebeat 对于收集的每行日志都封装成 event，event 发送到 output 之前，可在配置文件中定义 processors 去处理 event。processor 主要用来减少输出的字段、添加其他的 metadata 等操作，每个 processor 会接收一个 event，并将一些定义好的规则应用到 event，然后发送到 output。</p>









<p data-nodeid="15908">这里配置的 drop_fields 就是删除一些不必要的字段，具体删除哪些字段，要根据应用需求而定。如果将 Filebeat 的所有字段都输出到 Kafka，不但影响性能，而且还占用空间。</p>
<p data-nodeid="15909">做完这个设置后，再次查看 Kafka 中的输出日志，已经不再输出这四个字段信息了。</p>
<p data-nodeid="15910"><strong data-nodeid="16123">Filebeat 输出到 Redis 的配置</strong>Filebeat 除了可以发送日志数据到 Kafka，还可以发送数据到 Redis。如果你需要发送数据给 Redis，可以按照如下配置文件进行配置：</p>
<pre class="lang-dart" data-nodeid="70289"><code data-language="dart">filebeat.inputs:
- type: log
  enabled: <span class="hljs-keyword">true</span>
  paths:
   - /<span class="hljs-keyword">var</span>/log/messages
   - /<span class="hljs-keyword">var</span>/log/secure
name: <span class="hljs-string">"172.16.213.156"</span>
output.redis:
  hosts: [<span class="hljs-string">"172.16.213.227"</span>]
  db: <span class="hljs-string">"0"</span>
  key: <span class="hljs-string">"filebeat-log"</span>
  timeout: <span class="hljs-number">5</span>
logging.level: debug
</code></pre>




<p data-nodeid="70878">在这个例子中，我监控了两个日志文件 /var/log/messages 和 /var/log/secure，并将这两个文件的内容实时输出到 Redis 中，重点看 output.redis 这部分的配置。Hosts 指定 Redis 服务器的地址，默认端口是 6379，db 指定将日志写到那个库中，key 指定 key 名称，timeout 是连接超时时间。<br>
配置完成，启动 filebeat 服务，然后尝试 ssh 登录 172.16.213.156 主机，让 /var/log/secure 文件产生日志，接着登录 Redis 服务器，查看是否有数据写入，如下图所示：</p>
<p data-nodeid="70879" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/22/B6/Ciqc1F7saviAIgKIAACKia52tP0739.png" alt="image2.png" data-nodeid="70885"></p>


<p data-nodeid="15914">可以看到，Filebeat 已经将收集到的数据传到了 Redis 服务器上。</p>
<h3 data-nodeid="15915">总结</h3>
<p data-nodeid="15916">本课时主要讲解了 Filebeat 的安装、配置及如何使用。在经典的 EFK 架构中，Filebeat 经常作为前端收集日志，然后将日志传给 Kafka，然后 Logstash 再从 Kafka 消费日志中将数据发送到 Elasticsearch 中，最后在 Kibana 中进行可视化展示。在后面的课时内容中我将陆续介绍这些架构的实现细节。</p>

---

### 精选评论


