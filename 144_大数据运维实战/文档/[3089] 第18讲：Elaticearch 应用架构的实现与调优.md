<h3 data-nodeid="52004" class="">Elasticsearch 介绍</h3>




<p data-nodeid="48565">Elasticsearch 是一个实时的分布式搜索和分析引擎，它可以用于全文搜索、结构化搜索及分析，并采用 Java 语言编写，它的主要特点如下：</p>
<ul data-nodeid="48566">
<li data-nodeid="48567">
<p data-nodeid="48568">实时搜索、实时分析；</p>
</li>
<li data-nodeid="48569">
<p data-nodeid="48570">分布式架构、实时文件存储，并将每一个字段都编入索引；</p>
</li>
<li data-nodeid="48571">
<p data-nodeid="48572">文档导向，所有的对象全部是文档；</p>
</li>
<li data-nodeid="48573">
<p data-nodeid="48574">高可用性、易扩展，支持集群（Cluster）、分片和复制（Shards and Replicas）；</p>
</li>
<li data-nodeid="48575">
<p data-nodeid="48576">接口友好，支持 JSON。</p>
</li>
</ul>
<h3 data-nodeid="53358" class="">Elasticsearch 集群的安装与基础调优</h3>

<h4 data-nodeid="54690" class="">1. Elasticsearch 集群的架构与角色</h4>

<p data-nodeid="48579">Elasticsearch 集群的一个<strong data-nodeid="48819">主要特点是去中心化</strong>，从字面上理解就是无中心节点，这是从集群外部来说的，因为从外部来看 Elasticsearch 集群，它在逻辑上是一个整体，与任何一个节点的通信和与整个 Elasticsearch 集群通信是完全相同的。另外，从集群内部来看，集群中可以有多个节点，其中有一个为主节点（Master node），该主节点不是通过配置文件定义的，而是通过选举产生的。</p>
<p data-nodeid="57288">下图为 Elasticsearch 集群的运行架构图：</p>
<p data-nodeid="57289" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/62/CgqCHl7x1ZOAOu_oAAGRjSv511Y066.png" alt="image1.png" data-nodeid="57293"></p>


<p data-nodeid="48582">在 ElasticSearch 的架构中，有三类角色，分别是 Client node、Data node 和 Master node。其关系如下：搜索查询的请求一般是经过 Client node 来向 Data node 获取数据；而索引查询首先请求 Master node 节点，然后 Master node 将请求分配到多个 Data node 节点完成一次索引查询。</p>
<p data-nodeid="48583">本课时介绍的 Elasticsearch 架构中，我们只用了 Data node 和 Master node 角色，省去了 Client node 节点，集群采用 3 个节点，每个节点采用 CentOS 7.7 版本。关于主机和对应的各个角色如下表所示：</p>
<table data-nodeid="59831">
<thead data-nodeid="59832">
<tr data-nodeid="59833">
<th data-org-content="节点名称" data-nodeid="59835">节点名称</th>
<th align="center" data-org-content="IP 地址" data-nodeid="59836">IP 地址</th>
<th data-org-content="集群角色" data-nodeid="59837">集群角色</th>
<th align="center" data-org-content="安装软件" data-nodeid="59838">安装软件</th>
</tr>
</thead>
<tbody data-nodeid="59843">
<tr data-nodeid="59844">
<td data-org-content="server1" data-nodeid="59845">server1</td>
<td align="center" data-org-content="172.16.213.152" data-nodeid="59846">172.16.213.152</td>
<td data-org-content="Master node、Data node" data-nodeid="59847">Master node、Data node</td>
<td align="center" data-org-content="elasticsearch-7.7.1" data-nodeid="59848">elasticsearch-7.7.1</td>
</tr>
<tr data-nodeid="59849">
<td data-org-content="server2" data-nodeid="59850">server2</td>
<td align="center" data-org-content="172.16.213.138" data-nodeid="59851">172.16.213.138</td>
<td data-org-content="Master node、Data node" data-nodeid="59852">Master node、Data node</td>
<td align="center" data-org-content="elasticsearch-7.7.1" data-nodeid="59853">elasticsearch-7.7.1</td>
</tr>
<tr data-nodeid="59854">
<td data-org-content="server3" data-nodeid="59855">server3</td>
<td align="center" data-org-content="172.16.213.80" data-nodeid="59856">172.16.213.80</td>
<td data-org-content="Data node" data-nodeid="59857">Data node</td>
<td align="center" data-org-content="elasticsearch-7.7.1" data-nodeid="59858">elasticsearch-7.7.1</td>
</tr>
</tbody>
</table>


<p data-nodeid="48617">这里对集群中每个角色的含义介绍如下。</p>
<ul data-nodeid="48618">
<li data-nodeid="48619">
<p data-nodeid="48620">Master node</p>
</li>
</ul>
<p data-nodeid="48621">可以理解为主节点，主要用于元数据（Metadata）的处理，比如索引的新增、删除、分片分配等，以及管理集群各个节点的状态。Elasticsearch 集群中可以定义多个主节点，但是在同一时刻，只有一个主节点起作用，其他定义的主节点，是作为主节点的候选节点存在。当一个主节点故障后，集群会从候选主节点中选举出<strong data-nodeid="48849">新的主节点</strong>。</p>
<p data-nodeid="48622">由于数据的存储和查询都不会走主节点，所以主节点的压力相对较小，因此主节点的内存分配也可以相对少些，但是主节点却是<strong data-nodeid="48859">最重要的</strong>，因为一旦主节点宕机，整个 Elasticsearch 集群将不可用，所以<strong data-nodeid="48860">一定要保证主节点的稳定性</strong>。</p>
<ul data-nodeid="48623">
<li data-nodeid="48624">
<p data-nodeid="48625">Data node</p>
</li>
</ul>
<p data-nodeid="48626">可以理解为数据节点，这些节点上保存了数据分片。它负责数据相关操作，比如分片的 CRUD、搜索和整合等。数据节点上面执行的操作都比较消耗 CPU、内存和 I/O 资源，因此数据节点服务器要选择较好的硬件配置，才能获取高效的存储和分析性能。</p>
<ul data-nodeid="48627">
<li data-nodeid="48628">
<p data-nodeid="48629">Client node</p>
</li>
</ul>
<p data-nodeid="48630">可以理解为客户端节点，属于可选节点，是作为任务分发用的，它里面也会存元数据，但是它不会对元数据做任何修改。Client node 存在的好处是可以分担 Data node 的一部分压力，因为 Elasticsearch 的查询是两层汇聚的结果，第 1 层是在 Data node 上做查询结果汇聚，然后把结果发给 Client node，接收到 Data node 发来的结果后再做第 2 次的汇聚，然后把最终的查询结果返回给用户。这样，Client node 就替 Data node 分担了部分压力。</p>
<p data-nodeid="48631">由上可以看出，每个节点都有存在的意义，我们只有把相关功能和角色划分清楚了，每种节点各尽其责，才能充分发挥出分布式集群的效果。</p>
<h4 data-nodeid="61169" class="">2. 安装 Elasticsearch 与授权</h4>

<p data-nodeid="48633">Elasticsearch 的安装非常简单，首先点击<a href="https://www.elastic.co/" data-nodeid="48890"> Elastic </a><a href="https://www.elastic.co/" data-nodeid="48893">官网下载页面</a>找到适合版本，然后选择 zip、tar、rpm 等格式安装包进行下载。这里我们下载的软件包为 elasticsearch-7.7.1.tar.gz。安装过程如下：</p>
<pre class="lang-dart" data-nodeid="71838"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# tar -zxvf elasticsearch<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span>.tar.gz -C /usr/local
[root<span class="hljs-meta">@localhost</span> ~]# mv /usr/local/elasticsearch<span class="hljs-number">-7.7</span><span class="hljs-number">.1</span>  /usr/local/elasticsearch
</code></pre>









<p data-nodeid="48635">这里我们将 Elasticsearch 安装到了 /usr/local 目录下。由于 Elasticsearch 可以接收用户输入的脚本并且执行，为了系统安全考虑，需要创建一个单独的用户来运行 Elasticsearch。这里创建的普通用户是 elasticsearch，操作如下：</p>
<pre class="lang-dart" data-nodeid="73093"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# useradd elasticsearch
</code></pre>

<p data-nodeid="48637">然后将 Elasticsearch 的安装目录都授权给该用户，操作如下：</p>
<pre class="lang-dart" data-nodeid="74348"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# chown -R elasticsearch:elasticsearch  /usr/local/elasticsearch
</code></pre>

<h4 data-nodeid="100703" class="">3. 操作系统调优</h4>

<p data-nodeid="48640">操作系统及 JVM 调优主要是针对安装 Elasticsearch 的机器，为了获取高效、稳定的性能，需要从操作系统和 JVM 两个方面对 Elasticsearch 进行一个简单调优。</p>
<p data-nodeid="48641">对于操作系统，需要调整几个内核参数，将下面内容添加到 /etc/sysctl.conf 文件中：</p>
<pre class="lang-dart" data-nodeid="75603"><code data-language="dart">fs.file-max=<span class="hljs-number">655360</span>
vm.max_map_count = <span class="hljs-number">262144</span>
</code></pre>

<p data-nodeid="48643">其中，第一个参数 fs.file-max 主要是配置系统最大打开文件描述符数，建议修改为 655360 或者更高；第二个参数 vm.max_map_count 影响 Java 线程数量，用于限制一个进程可以拥有的 VMA（虚拟内存区域）的大小，系统默认是 65530，建议修改成 262144 或者更高。</p>
<p data-nodeid="48644">另外，还需要调整进程最大打开文件描述符（nofile）、最大用户进程数（nproc）和最大锁定内存地址空间（memlock），添加如下内容到 /etc/security/limits.conf 文件中：</p>
<pre class="lang-dart" data-nodeid="76858"><code data-language="dart">* soft nproc <span class="hljs-number">20480</span>
* hard nproc <span class="hljs-number">20480</span>
* soft nofile <span class="hljs-number">65536</span>
* hard nofile <span class="hljs-number">65536</span>
* soft memlock unlimited
* hard memlock unlimited
</code></pre>

<p data-nodeid="48646">最后，还需要修改 /etc/security/limits.d/20-nproc.conf 文件（CentOS 7.x系统），将 * soft nproc 4096 修改为 * soft nproc 20480，或者直接删除 /etc/security/limits.d/20-nproc.conf 文件也行。</p>
<h4 data-nodeid="101943" class="">4. JVM 调优</h4>

<p data-nodeid="48648">JVM 调优主要是针对 Elasticsearch 的 JVM 内存资源进行优化，其内存资源配置文件为 jvm.options，此文件位于 /usr/local/elasticsearch/config 目录下，打开此文件，修改如下内容：</p>
<ul data-nodeid="48649">
<li data-nodeid="48650">
<p data-nodeid="48651">-Xms1g</p>
</li>
<li data-nodeid="48652">
<p data-nodeid="48653">-Xmx1g</p>
</li>
</ul>
<p data-nodeid="48654">可以看到，默认 JVM 内存为 1G，可根据服务器内存大小，修改为合适的值。<strong data-nodeid="48935">一般设置为服务器物理内存的一半最佳。</strong></p>
<h3 data-nodeid="103167" class="">配置 Elasticsearch</h3>

<p data-nodeid="48656">Elasticsearch 的配置文件均在 Elasticsearch 根目录下的 config 文件夹中，这里是 /usr/local/elasticsearch/config 目录，主要有 jvm.options、elasticsearch.yml 和 log4j2.properties 三个主要配置文件。其中 jvm.options 为 JVM 配置文件，log4j2.properties 为日志配置，两者都相对比较简单，这里重点介绍 elasticsearch.yml 一些重要的配置项及其含义。</p>
<p data-nodeid="48657">这里配置的 Elasticsearch.yml 文件内容如下：</p>
<pre class="lang-dart" data-nodeid="78113"><code data-language="dart">cluster.name: my-application
node.name: server1
node.master: <span class="hljs-keyword">true</span>
node.data: <span class="hljs-keyword">true</span>
path.data: /data1/elasticsearch,/data2/elasticsearch
path.logs: /usr/local/elasticsearch/logs
bootstrap.memory_lock: <span class="hljs-keyword">true</span>
network.host: <span class="hljs-number">0.0</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>
http.port: <span class="hljs-number">9200</span>
discovery.seed_hosts: [<span class="hljs-string">"server1"</span>, <span class="hljs-string">"server2"</span>, <span class="hljs-string">"server3"</span>]
cluster.initial_master_nodes: [<span class="hljs-string">"server1"</span>, <span class="hljs-string">"server2"</span>]
</code></pre>

<p data-nodeid="48659">在上述内容中，每个配置项的含义分别介绍如下。</p>
<p data-nodeid="48660">（1）cluster.name: my-application</p>
<p data-nodeid="48661">配置 Elasticsearch 集群名称，默认是 Elasticsearch，这里修改为 my-application。Elasticsearch 会自动发现在同一网段下的集群名为 Esbigdata 的主机，如果在同一网段下有多个集群，就可以通过这个属性来区分不同的集群。</p>
<p data-nodeid="48662">（2）node.name: server1</p>
<p data-nodeid="48663">节点名，任意指定一个即可，这里是 server1，我们这个集群环境中有 3 个节点，即 server1、server2 和 server3，<strong data-nodeid="48954">记得根据主机的不同，要修改相应的节点名称。</strong></p>
<p data-nodeid="48664">（3）node.master: true</p>
<p data-nodeid="48665">指定该节点是否有资格被选举成为 master，默认是 true，Elasticsearch 集群中默认第一台启动的机器为 master 角色，如果这台服务器宕机就会重新选举新的 master。我在这个集群环境中，定义了 server1 和 server2 两个 master 节点，因此这两个节点上 Node.Master 的值要设置为 true。</p>
<p data-nodeid="48666">（4）node.data: true</p>
<p data-nodeid="48667">指定该节点是否存储索引数据，默认为 true，表示数据存储节点，如果节点配 node.master 为 false，并且配置 node.data 为 false，则该节点就是 Client Node，它类似于一个“路由器”，负责将集群层面的请求转发到主节点上，然后将数据相关的请求转发到数据节点。在我们这个集群环境中，定义了 server1、server2 和 server3 均为数据存储节点，因此这三个节点上 node.data 的值要设置为 true。</p>
<p data-nodeid="48668">（5）path.data:/data1/elasticsearch, /data2/elasticsearch</p>
<p data-nodeid="48669">设置索引数据的存储路径，默认是 elasticsearch 根目录下的 data 文件夹，这里自定义了两个路径，可以设置多个存储路径，用逗号隔开。</p>
<p data-nodeid="48670">（6）path.logs: /usr/local/elasticsearch/logs</p>
<p data-nodeid="48671">设置日志文件的存储路径，默认是 elasticsearch 根目录下的 logs 文件夹。</p>
<p data-nodeid="48672">（7）bootstrap.memory_lock: true</p>
<p data-nodeid="48673">此配置项一般设置为 true 用来锁住物理内存。在 Linux 下物理内存的执行效率要远远高于虚拟内存（swap）的执行效率，因此，当 JVM 开始使用 swap 内存时 Elasticsearch 的执行效率会降低很多，所以要保证它不使用 swap，保证机器有足够的物理内存分配给 Elasticsearch。同时也要允许 Elasticsearch 的进程可以锁住物理内存，Linux 下可以通过“ulimit -l”命令查看最大锁定内存地址空间（memlock）是不是 unlimited，这个参数在之前系统调优的时候已经设置过了。</p>
<p data-nodeid="48674">（8）network.host: 172.16.213.151</p>
<p data-nodeid="48675">此配置项设置将 Elasticsearch 服务绑定到哪个网络上，配置为服务器的内网 IP 地址即可。设置为 0.0.0.0 的话，如果服务器有外网 IP，那么也可以通过外网访问 Elasticsearch，这样会非常不安全。</p>
<p data-nodeid="48676">（9）http.port: 9200</p>
<p data-nodeid="48677">设置 Elasticsearch 对外提供服务的 http 端口，默认为 9200。其实，还有一个端口配置选项 transport.tcp.port，此配置项用来设置节点间交互通信的 TCP 端口，默认是 9300。</p>
<p data-nodeid="48678">（10）discovery.seed_hosts</p>
<p data-nodeid="48679">配置集群的所有主机地址，配置之后集群的主机之间可以自动发现。</p>
<p data-nodeid="48680">（11）cluster.initial_master_nodes</p>
<p data-nodeid="48681">用来设置一系列符合 master 节点条件的主机名或 IP 地址来引导启动集群。</p>
<h3 data-nodeid="104377" class="">启动 Elasticsearch</h3>

<p data-nodeid="48683">启动 Elasticsearch 服务需要在一个普通用户下完成，如果通过 root 用户启动 Elasticsearch 的话，可能会收到如下错误：</p>
<pre class="lang-dart" data-nodeid="79368"><code data-language="dart">java.lang.RuntimeException: can not run elasticsearch <span class="hljs-keyword">as</span> root
at org.elasticsearch.bootstrap.Bootstrap.initializeNatives(Bootstrap.java:<span class="hljs-number">106</span>)
~[elasticsearch<span class="hljs-number">-6.3</span><span class="hljs-number">.2</span>.jar:<span class="hljs-number">6.3</span><span class="hljs-number">.2</span>]
at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:<span class="hljs-number">195</span>)
~[elasticsearch<span class="hljs-number">-6.3</span><span class="hljs-number">.2</span>.jar:<span class="hljs-number">6.3</span><span class="hljs-number">.2</span>]
at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:<span class="hljs-number">342</span>)
[elasticsearch<span class="hljs-number">-6.3</span><span class="hljs-number">.2</span>.jar:<span class="hljs-number">6.3</span><span class="hljs-number">.2</span>]
</code></pre>

<p data-nodeid="48685">这是出于系统安全考虑，Elasticsearch 服务必须通过普通用户来启动，在上面第一小节中，已经创建了一个普通用户 elasticsearch，直接切换到这个用户下启动 Elasticsearch 集群即可。分别登录到 server1、server2 和 server3 三台主机上，执行如下操作：</p>
<pre class="lang-dart" data-nodeid="80623"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# su - elasticsearch
[elasticsearch<span class="hljs-meta">@localhost</span> ~]$ cd /usr/local/elasticsearch/
[elasticsearch<span class="hljs-meta">@localhost</span> elasticsearch]$ bin/elasticsearch -d
</code></pre>

<p data-nodeid="48687">其中，“-d” 参数的意思是将 Elasticsearch 放到后台运行。</p>
<h3 data-nodeid="105573" class="">验证 Elasticsearch 集群的正确性</h3>

<p data-nodeid="48689">将所有 Elasticsearch 节点的服务启动后，在任意一个节点执行如下命令：</p>
<pre class="lang-dart" data-nodeid="81878"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# curl http:<span class="hljs-comment">//172.16.213.152:9200</span>
</code></pre>

<p data-nodeid="107901">如果返回类似如下结果，则表示 Elasticsearch 集群运行正常。</p>
<p data-nodeid="107902" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/63/CgqCHl7x1paAeDKuAABMVLoLmig568.png" alt="image2.png" data-nodeid="107906"></p>


<p data-nodeid="48692">从安装、基础调优、配置，再到启动、验证，Elasticsearch 集群安装与配置的讲解已经完成了。下面讲解如何安装 Head 插件。</p>
<h3 data-nodeid="109073" class="">安装 Head 插件</h3>

<h4 data-nodeid="110213" class="">1. 安装 Head 插件</h4>

<p data-nodeid="111345" class="">Head 插件是 Elasticsearch 的图形化界面工具，通过此插件可以很方便地对数据进行增删改查等数据交互操作。在 Elasticsearch 5.X 版本以后，Head 插件已经是一个独立的 Web App 了，所以不需要和 Elasticsearch 进行集成，可以将 Head 插件安装到任何一台机器上。这里将 Head 插件安装到 172.16.213.138（server2）机器上，可以点击 <a href="https://github.com/mobz/elasticsearch-head" data-nodeid="111349">GitHub 网站</a> 获取插件安装包。</p>

<p data-nodeid="48696">由于 Head 插件本质上是一个 Node.js 工程，因此需要先安装 Node.js，使用 npm 工具来安装依赖的包。</p>
<p data-nodeid="48697">在 CentOS 7.X 系统上，可以直接通过 yum 在线安装 Node.js（前提是你的机器要能上网），操作如下：</p>
<pre class="lang-dart" data-nodeid="83133"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# curl --silent --location https:<span class="hljs-comment">//rpm.nodesource.com/setup_10.x | sudo bash -</span>
[root<span class="hljs-meta">@localhost</span> ~]# yum install -y nodejs
</code></pre>

<p data-nodeid="48699">这里我们通过 git 克隆方式下载 Head 插件，所以还需要安装一个 git 命令工具，执行如下命令即可：</p>
<pre class="lang-dart" data-nodeid="84388"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# yum install -y git
</code></pre>

<p data-nodeid="48701">接着，开始安装 Head 插件，这里将 Head 插件安装到 /usr/local 目录下，操作过程如下：</p>
<pre class="lang-dart" data-nodeid="85643"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# cd /usr/local
[root<span class="hljs-meta">@localhost</span> local]# git clone git:<span class="hljs-comment">//github.com/mobz/elasticsearch-head.git</span>
[root<span class="hljs-meta">@localhost</span> local]# npm config <span class="hljs-keyword">set</span> registry “http:<span class="hljs-comment">//registry.npm.taobao.org/”</span>
[root<span class="hljs-meta">@localhost</span> local]# cd elasticsearch-head
[root<span class="hljs-meta">@localhost</span> elasticsearch-head]# npm install
</code></pre>

<p data-nodeid="48703">其中，第一步是通过 git 命令从 GitHub 上克隆 Head 插件程序，第二步是修改源地址为淘宝 NPM 镜像，因为默认 NPM 的官方源为 https://registry.npmjs.org/， 国内下载速度会很慢，所以建议切换到淘宝的 NPM 镜像站点。第三步是安装 Head 插件所需的库和第三方框架。</p>
<p data-nodeid="48704">克隆下来的 Head 插件程序目录名为 elasticsearch-head，进入此目录，修改配置文件 /usr/local/elasticsearch-head/_site/app.js，找到如下内容：</p>
<pre class="lang-dart" data-nodeid="86898"><code data-language="dart"><span class="hljs-keyword">this</span>.base_uri = <span class="hljs-keyword">this</span>.config.base_uri || <span class="hljs-keyword">this</span>.prefs.<span class="hljs-keyword">get</span>(<span class="hljs-string">"app-base_uri"</span>) || <span class="hljs-string">"http://localhost:9200"</span>;
</code></pre>

<p data-nodeid="48706">将其中的 http://localhost:9200，修改为 Elasticsearch 集群中任意一台主机的 IP 地址，这里修改为:</p>
<pre class="lang-dart" data-nodeid="88153"><code data-language="dart">&lt;http:<span class="hljs-comment">//172.16.213.138:9200&gt;</span>
</code></pre>

<p data-nodeid="48708">表示的意思是 Head 插件将通过 172.16.213.138（server2）访问 Elasticsearch 集群。注意，访问 Elasticsearch 集群中的任意一个节点，都能获取集群的所有信息。</p>
<h4 data-nodeid="112469" class="">2. 修改 Elasticsearch 配置</h4>

<p data-nodeid="48710">在上面的配置中，将 Head 插件访问集群的地址配置为 172.16.213.138（server2）主机，下面还需要修改此主机上 Elasticsearch 的配置，添加跨域访问支持。</p>
<p data-nodeid="48711">修改 Elasticsearch 配置文件，允许 Head 插件跨域访问 Elasticsearch，在 Elasticsearch.yml 文件最后添加如下内容：</p>
<pre class="lang-dart" data-nodeid="89408"><code data-language="dart">http.cors.enabled: <span class="hljs-keyword">true</span>
http.cors.allow-origin: <span class="hljs-string">"*"</span>
</code></pre>

<p data-nodeid="48713">其中：</p>
<ul data-nodeid="48714">
<li data-nodeid="48715">
<p data-nodeid="48716">http.cors.enabled 表示开启跨域访问支持，此值默认为 false；</p>
</li>
<li data-nodeid="48717">
<p data-nodeid="48718">http.cors.allow-origin 表示跨域访问允许的域名地址，可以使用正则表达式，这里的“*”表示允许所有域名访问。</p>
</li>
</ul>
<h4 data-nodeid="113563" class="">3. 启动 Head 插件服务</h4>

<p data-nodeid="48720">所有配置完成之后，就可以启动插件服务了，执行如下操作：</p>
<pre class="lang-dart" data-nodeid="90663"><code data-language="dart">[root<span class="hljs-meta">@localhost</span> ~]# cd /usr/local/elasticsearch-head
[root<span class="hljs-meta">@localhost</span> elasticsearch-head]# npm run start
</code></pre>

<p data-nodeid="48722">Head 插件服务启动之后，默认的访问端口为 9100，直接访问 http://172.16.213.138:9100 就可以访问 Head 插件了。</p>
<p data-nodeid="115663">下图是配置完成后的一个 Head 插件截图：</p>
<p data-nodeid="115664" class=""><img src="https://s0.lgstatic.com/i/image/M00/26/63/CgqCHl7x1tWAVw01AADCWjPJ2oA913.png" alt="image3.png" data-nodeid="115668"><br>
配置完成后的 Head 插件图</p>




<p data-nodeid="48726">从上图可以看到，Elasticsearch 集群有 server1、server2 和 server3 三个节点，其中，server1 是目前的主节点。点击图上的信息按钮，可查看节点详细信息。</p>
<p data-nodeid="48727">其次，从这个页面上可以看到 Elasticsearch 基本的分片信息，比如主分片、副本分片等，以及多少可用分片。由于在 Elasticsearch 配置中设置了 5 个分片、一个副本分片，因此可以看到每个索引都有 10 个分片，每个分片都用 0、1、2、3、4 等数字加方框表示，其中，粗体方框是主分片，细体方框是副本分片。</p>
<p data-nodeid="48728">再者，图中 my-application 是集群的名称，后面的“集群健康值”通过不同的颜色表示集群的健康状态：其中，绿色表示主分片和副本分片都可用；黄色表示只有主分片可用，没有副本分片；红色表示主分片中的部分索引不可用，但是某些索引还可以继续访问。正常情况下都显示绿色。</p>
<h3 data-nodeid="116723" class="">Elasticsearch 的优化</h3>

<h4 data-nodeid="117771" class="">1. JVM 内存的优化</h4>

<p data-nodeid="48731">首先，作为一个 Java 应用，就脱离不开 JVM 和 GC。</p>
<p data-nodeid="48732">下面先做一些简单地 JVM 介绍。Java 中的堆是 JVM 所管理的最大的一块内存空间，主要用于存放各种类的实例对象。在 Java 中，堆被划分成两个不同的区域：新生代（Young）、老年代（Old），其目的是使 JVM 能够更好地管理堆内存中的对象，包括内存的分配以及回收。</p>
<p data-nodeid="48733">然后再对 GC 进行简单介绍。设置堆内存的唯一目的是创建对象实例，所有的对象实例和数组都要在堆上分配，堆由垃圾回收（Garbage Collect）来负责，因此也叫作 “GC 堆”，垃圾回收采用分代算法，堆由此分为<strong data-nodeid="49131">新生代</strong>和<strong data-nodeid="49132">老年代</strong>。堆的优势是可以动态地分配内存大小，生存期也不必事先告诉编译器，因为它是在运行时动态分配内存的，Java 的垃圾回收器会自动回收这些不再使用的数据。</p>
<p data-nodeid="48734">了解了堆内存的概念和作用后，下面来说下 Elasticsearch 的 JVM 设置堆内存的方法。默认情况下，Elasticsearch JVM 使用堆内存最小和最大值为 1G，可以在 Elasticsearch 的配置目录下 jvm.options 文件中找到如下内容：</p>
<pre class="lang-dart" data-nodeid="91918"><code data-language="dart">-Xms1g
-Xmx1g
</code></pre>

<p data-nodeid="48736">其中，Xms 是设置堆最小内存，Xmx 是设置堆最大内存，很明显，默认的这个值太小，我们生产系统环境中必须要修改这个值。修改方法很简单，直接修改 jvm.options 文件中这两个值即可，比如修改为 16G，可以这么写：</p>
<pre class="lang-dart" data-nodeid="93173"><code data-language="dart">-Xms16g
-Xmx16g
</code></pre>

<p data-nodeid="48738">然后再次重启 Elasticsearch，配置才能生效。</p>
<p data-nodeid="48739">那么问题来了，这个最小堆大小（Xms）和最大堆大小（Xmx）设置多少合适呢？继续往下看。</p>
<p data-nodeid="118803" class=""><strong data-nodeid="118808">堆内存值的设置取决于服务器上可用的内存大小。</strong> 原则上来说，堆内存设置的越大，Elasticsearch 可用的堆就越多，可用于缓存的内存就越多，但不能无限大，太多的堆内存可能会使垃圾回收机制暂停，并且还会浪费大量内存。</p>

<p data-nodeid="48741">根据经验，将最小堆大小（Xms）和最大堆大小（Xmx）设置为相同值，可以防止堆在运行时调整大小，因为这是一个非常消耗性能的过程。</p>
<p data-nodeid="119841" class=""><strong data-nodeid="119845">那么设置堆内存多大合适呢？一个经验值是：不超过物理内存的 50％，以确保有足够的物理内存留给内核文件系统缓存，一般设置对内存最大不超过 32GB。</strong></p>

<p data-nodeid="48743">为什么呢？继续往下看。</p>
<p data-nodeid="48744">堆内存对于 Elasticsearch 来说非常重要，它可以为数据提供快速操作，但还有另外一个非常重要的内存使用者：Lucene。</p>
<p data-nodeid="48745">Lucene 是一个开源的全文检索引擎工具包，而 Elasticsearch 底层是基于 Lucene 的，并对其进行了扩展，提供了比 Lucene 更丰富的查询语言，以及简单易用的 restful api 接口、java api 接口等。一句话概括就是，Elasticsearch 是 Lucene 面向企业搜索应用的扩展，可极大地缩短研发周期。</p>
<p data-nodeid="48746">Lucene 设计的目的是把底层操作系统数据缓存到内存中，而 Lucene 段（segment）存储在单个文件中，这些文件都不会发生变化，所以更利于缓存，同时操作系统也会把这些热段（hot segments）保留在内存中，以便更快地访问，这些热段包括倒排索引（用于全文搜索）和文档值（用于聚合）。</p>
<p data-nodeid="48747">由此可知，Lucene 的性能依赖于与操作系统的这种交互。如果把所有可用的内存都给了 Elasticsearch 的堆，那么 Lucene 就不会有任何剩余的内存，这将严重影响性能。</p>
<p data-nodeid="48748">所以正如前部分所说：建议将可用内存的 50％ 提供给 Elasticsearch 堆，而将其他 50％ 空闲，但这剩余 50% 的空闲内存不会被真的闲置，因为 Lucene 正等待使用这剩余 50% 的内存资源。</p>
<h4 data-nodeid="123841" class="">2. 操作系统内存优化</h4>




<p data-nodeid="48750">操作系统作为运行 Elasticsearch 的基础，为保证性能，最好禁用系统的 Swap 使用，方法如下。</p>
<p data-nodeid="48751">（1）暂时关闭 Swap，重启后恢复。</p>
<pre class="lang-dart" data-nodeid="94428"><code data-language="dart"> swapoff   -a
</code></pre>

<p data-nodeid="48753">（2）永久关闭 Swap</p>
<p data-nodeid="48754">编辑 /etc/fstab，注释掉如下 Swap 分区项即可。</p>
<pre class="lang-dart" data-nodeid="95683"><code data-language="dart">#UUID=<span class="hljs-number">0</span>b55fdb8-a9d8<span class="hljs-number">-4215</span><span class="hljs-number">-80</span>f7-f42f75644f87 none  swap    sw      <span class="hljs-number">0</span>       <span class="hljs-number">0</span>
</code></pre>

<p data-nodeid="48756">如果是混合服务器，不能完全禁用 Swap 的话，可以尝试降低 swappiness，该值控制操作系统尝试交换内存的积极性。当 swappiness=0 的时候，则表示最大限度使用物理内存，然后才是 Swap 空间；当 swappiness＝100 的时候，则表示积极的使用 Swap 分区，并且把内存上的数据及时地搬运到 Swap 空间里面。</p>
<p data-nodeid="48757">Linux 的基本默认设置为 60，具体如下：</p>
<pre class="lang-dart" data-nodeid="96938"><code data-language="dart">cat /proc/sys/vm/swappiness
</code></pre>

<p data-nodeid="124835">也就是说，你的内存在使用到 100-60=40% 的时候，就开始出现有交换分区的使用。</p>
<p data-nodeid="124836">操作系统层面要尽可能使用内存，此时就需要对该参数进行调整。</p>

<p data-nodeid="48760">临时调整的方法如下，比如调成 10，可以在命令行执行如下：</p>
<pre class="lang-dart" data-nodeid="98193"><code data-language="dart">sysctl vm.swappiness=<span class="hljs-number">10</span>
</code></pre>

<p data-nodeid="48762">要想永久调整的话，则需要将在 /etc/sysctl.conf 修改，加上：</p>
<pre class="lang-dart" data-nodeid="99448"><code data-language="dart">cat /etc/sysctl.conf
vm.swappiness=<span class="hljs-number">10</span>
</code></pre>

<h4 data-nodeid="128810" class="">3. 使用更快的硬件</h4>




<p data-nodeid="48765">要保证 Elasticsearch 的性能，在硬件上配置 SSD 硬盘是最有效果的，如果有多个 SSD 硬盘，则可以配置成 RAID 0 阵列以获得更佳的 IO 性能，但是任何一个 SSD 损坏都有可能破坏 index。</p>
<p data-nodeid="129798" class=""><strong data-nodeid="129802">因此，通常正确的做法是优化单的 shard 存储性能，然后添加 replicat 放在不同的节点，同时使用 snapshot 快照和 restore 功能去备份 index。</strong></p>

<h3 data-nodeid="48767">总结</h3>
<p data-nodeid="130683">本课时注意讲解了 Elasticsearch 的集群安装、配置与基础调优，以及 Head 插件的使用，其中，Elasticsearch 集群的部署，以及每个参数的含义是需要熟练掌握的内容，因为对于故障的处理和优化，必须要掌握 Elasticsearch 每个参数的含义。</p>

---

### 精选评论

##### *敏：
> 大内存尽量配置GC使用G1垃圾收集器

