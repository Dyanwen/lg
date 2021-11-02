<p>在上一课时中，我们介绍了 SkyWalking 的整体架构以及 Service、Endpoint、ServiceInstance 等核心概念。本课时将带领同学们搭建 SkyWalking 的环境搭建，并上手使用 SkyWalking。</p>
<h3>SkyWalking 环境搭建</h3>
<p>在本课时中，我们将安装并体验 SkyWalking 的基本使用，下面是使用到的相关软件包：</p>
<ul>
<li>apache-skywalking-apm-6.2.0.tar.gz</li>
</ul>
<p>下载地址：<a href="https://archive.apache.org/dist/skywalking/6.2.0/">https://archive.apache.org/dist/skywalking/6.2.0/</a></p>
<ul>
<li>elasticsearch-6.6.1.tar.gz</li>
</ul>
<p>下载地址：<a href="https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.1.tar.gz">https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.6.1.tar.gz</a></p>
<ul>
<li>kibana-6.6.1-darwin-x86_64.tar.gz</li>
</ul>
<p>下载地址：<a href="https://artifacts.elastic.co/downloads/kibana/kibana-6.6.1-darwin-x86_64.tar.gz">https://artifacts.elastic.co/downloads/kibana/kibana-6.6.1-darwin-x86_64.tar.gz</a></p>
<h3>ElasticSearch 安装</h3>
<p>下载完 elasticsearch-6.6.1.tar.gz 包之后，使用如下命令进行解压缩：</p>
<pre><code data-language="java" class="lang-java">tar&nbsp;-zxf&nbsp;elasticsearch-<span class="hljs-number">6.6</span><span class="hljs-number">.1</span>.tar.gz
</code></pre>
<p>解压完成之后，进入得到的 elasticsearch-6.6.1 目录中，执行如下命令后台启动 ElasticSearch 服务：</p>
<pre><code data-language="js" class="lang-js">./bin/elasticsearch&nbsp;-d
</code></pre>
<p>ElasticSearch 启动的相关日志可以通过下面的命令进行查看：</p>
<pre><code data-language="java" class="lang-java">tail&nbsp;-f&nbsp;logs/elasticsearch.log
</code></pre>
<p>最后，我们可以请求 localhost:9200 这地址，看到下图输出的这段 JSON 即安装成功：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMCAYb-sAABorqyP-yg322.png" alt=""></p>
<h3>Kibana 安装</h3>
<p>Kibana 是一个开源的分析和可视化平台，主要用于和 Elasticsearch 一起工作，轻松实现 ElasticSearch 的查询和管理。这里使用 ElasticSearch 作为 SkyWalking 的后端存储，在后续调试 SkyWalking 源码时，可能会直接查询 ElasticSearch 中的某些索引，所以这里一并安装 Kibana。</p>
<p>下载完 kibana-6.6.1-darwin-x86_64.tar.gz 安装包之后，我们使用如下命令进行解压：</p>
<pre><code data-language="java" class="lang-java">tar&nbsp;-zxf&nbsp;&nbsp;kibana-<span class="hljs-number">6.6</span><span class="hljs-number">.1</span>-darwin-x86_64.tar.gz
</code></pre>
<p>解压完成后进入 kibana-6.6.1-darwin-x86_64 目录，修改 config/kibana.yml 文件：</p>
<pre><code data-language="java" class="lang-java">#&nbsp;指定上述&nbsp;ElasticSearch监听的地址，其他配置不变
elasticsearch.hosts:&nbsp;["http://localhost:9200"]
</code></pre>
<p>之后执行如下命令，启动 Kibana 服务：</p>
<pre><code data-language="java" class="lang-java">./bin/kibana
</code></pre>
<p>最后我们通过访问 http://localhost:5601/ 地址即可进入 Kibana 界面：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMCAABXxAAEuKrTbLVw100.png" alt=""></p>
<h3>SkyWalking 安装</h3>
<p>下载完成 apache-skywalking-apm-6.2.0.tar.gz 包之后，执行如下命令解压缩：</p>
<pre><code data-language="java" class="lang-java">tar&nbsp;-zxf&nbsp;apache-skywalking-apm-<span class="hljs-number">6.2</span><span class="hljs-number">.0</span>.tar.gz
</code></pre>
<p>解压完成之后进入 apache-skywalking-apm-bin 目录，编辑 config/application.yml 文件，将其中 ElasticSearch 配置项以及其子项的全部注释去掉，将 h2 配置项及其子项全部注释掉，如下图所示，这样 SkyWalking 就从默认的存储 h2 切换成了 ElasticSearch ：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMCAH0O5AAfpg2pNpsw425.png" alt=""></p>
<p>接下来执行 ./bin/startup.sh 文件即可启动 SkyWalking OAP 以及 UI 界面，看到的输出如下：</p>
<pre><code data-language="js" class="lang-js">&gt;./bin/startup.sh
SkyWalking&nbsp;OAP&nbsp;started&nbsp;successfully!
SkyWalking&nbsp;Web&nbsp;Application&nbsp;started&nbsp;successfully!
</code></pre>
<p>我们可以在 logs/skywalking-oap-server.log 以及 logs/webapp.log&nbsp;两个日志文件中查看到 SkyWalking OAP 以及 UI 项目的相关日志，这里不再展开。</p>
<p>最后访问 http://127.0.0.1:8080/ 即可看到 SkyWalking 的 Rocketbot UI界面。</p>
<h3>Skywalking Agent 目录结构</h3>
<p>SkyWalking Agent 使用了 Java &nbsp;Agent 技术，可以在无需手工埋点的情况下，通过 JVM 接口在运行时将监控代码段插入已有 Java 应用中，实现对 Java 应用的监控。SkyWalking Agent 会将服务运行过程中获得的监控数据通过 gRPC 发送给后端的 OAP 集群进行分析和存储。</p>
<p>SkyWalking 目前提供的 Agent 插件在 apache-skywalking-apm-bin/agent 目录下：</p>
<pre><code data-language="java" class="lang-java">agent
&nbsp;&nbsp;&nbsp;&nbsp;├──&nbsp;activations
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;apm-toolkit-log4j-1.x-activation-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;...
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;└──&nbsp;apm-toolkit-trace-activation-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;├──&nbsp;config&nbsp;#&nbsp;Agent&nbsp;配置文件
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;└──&nbsp;agent.config
&nbsp;&nbsp;&nbsp;&nbsp;├──&nbsp;logs&nbsp;#&nbsp;日志文件
&nbsp;&nbsp;&nbsp;&nbsp;├──&nbsp;optional-plugins&nbsp;#&nbsp;可选插件
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;apm-customize-enhance-plugin-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;apm-gson-2.x-plugin-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;└──&nbsp;...&nbsp;...
&nbsp;&nbsp;&nbsp;&nbsp;├──&nbsp;plugins&nbsp;#&nbsp;当前生效插件
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;apm-activemq-5.x-plugin-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;tomcat-7.x-8.x-plugin-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;├──&nbsp;spring-commons-6.2.0.jar
&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;└──&nbsp;...&nbsp;...
&nbsp;&nbsp;&nbsp;&nbsp;└──&nbsp;skywalking-agent.jar
</code></pre>
<p>其中，agent.config 文件是 SkyWalking Agent 的唯一配置文件。plugins 目录存储了当前 Agent 生效的插件。optional-plugins 目录存储了一些可选的插件（这些插件可能会影响整个系统的性能或是有版权问题），如果需要使用这些插件，需将相应 jar 包移动到 plugins 目录下。最后的 skywalking-agent.jar 是 Agent 的核心 jar 包，由它负责读取 agent.config 配置文件，加载上述插件 jar 包，运行时收集到 的 Trace 和 Metrics 数据也是由它发送到 OAP 集群的。</p>
<h3>skywalking-demo 示例</h3>
<p>下面搭建 demo-webapp、demo-provider 两个 Spring-Boot 项目，并且接入 SkyWalking Agent 进行监控，具体结构如下：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMCAB9f7AAB00fhMQNk918.png" alt=""></p>
<p>demo-webapp 会 Dubbo 远程调用 demo-provider 的接口，而 Dubbo 依赖了 Zookeeper，所以要先安装 Zookeeper。首先下载 zookeeper-3.4.14.tar.gz 包（下载地址：<a href="https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/">https://archive.apache.org/dist/zookeeper/zookeeper-3.4.14/</a>）。下载完成之后执行如下命令解压缩：</p>
<pre><code data-language="java" class="lang-java">tar&nbsp;-zxf&nbsp;zookeeper-<span class="hljs-number">3.4</span><span class="hljs-number">.14</span>.tar.gz
</code></pre>
<p>解压完成之后，进入 zookeeper-3.4.14 目录，拷贝 conf/zoo_sample.cfg 文件并重命名为 conf/zoo.cfg，之后执行如下命令启动 Zookeeper：</p>
<pre><code data-language="java" class="lang-java">&gt;./bin/zkServer.sh&nbsp;start
#&nbsp;下面为输出内容
ZooKeeper&nbsp;JMX&nbsp;enabled&nbsp;by&nbsp;default
Using&nbsp;config:&nbsp;/Users/xxx/zookeeper-3.4.14/bin/../conf/zoo.cfg&nbsp;#&nbsp;配置文件
Starting&nbsp;zookeeper&nbsp;...&nbsp;STARTED&nbsp;#&nbsp;启动成功
</code></pre>
<p>下面在 IDEA 中创建 skywalking-demo 项目，并在其中创建 demo-api、demo-webapp、demo-provider 两个 Module，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMCALuSFAADAqK6QHPc083.png" alt=""></p>
<p>在 skywalking-demo 下面的 pom.xml 中，将父 pom 指向 spring-boot-starter-parent 并添加 demo-api 作为公共依赖，如下所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">project</span>&nbsp;<span class="hljs-attr">xmlns</span>=<span class="hljs-string">...</span>"&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">modelVersion</span>&gt;</span>4.0.0<span class="hljs-tag">&lt;/<span class="hljs-name">modelVersion</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">parent</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-parent<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.1.1.RELEASE<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">parent</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.xxx<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>skywalking-demo<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">packaging</span>&gt;</span>pom<span class="hljs-tag">&lt;/<span class="hljs-name">packaging</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.0-SNAPSHOT<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">modules</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">module</span>&gt;</span>demo-api<span class="hljs-tag">&lt;/<span class="hljs-name">module</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">module</span>&gt;</span>demo-webapp<span class="hljs-tag">&lt;/<span class="hljs-name">module</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">module</span>&gt;</span>demo-provider<span class="hljs-tag">&lt;/<span class="hljs-name">module</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">modules</span>&gt;</span>

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependencyManagement</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependencies</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.xxx<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>demo-api<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.0-SNAPSHOT<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependencies</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependencyManagement</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">project</span>&gt;</span>
</code></pre>
<p>在 demo-api 中只定义了 HelloService 接口，它是 Dubbo Provider 和 Dubbo Consumer 依赖的公共接口，如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">interface</span>&nbsp;<span class="hljs-title">HelloService</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">String&nbsp;<span class="hljs-title">say</span><span class="hljs-params">(String&nbsp;name)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception</span>;
}
</code></pre>
<h3>demo-provider 模块</h3>
<p>这里的 demo-provider 扮演了 Dubbo Provider 的角色，在其 pom.xml 文件中引入了 Spring Boot 以及集成 Dubbo 相关的依赖，如下所示：</p>
<pre><code data-language="html" class="lang-html"><span class="hljs-tag">&lt;<span class="hljs-name">dependencies</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;引入公共API接口&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.xxx<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>demo-api<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.0-SNAPSHOT<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;引入spring-boot-starter以及dubbo和curator的依赖&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>dubbo-spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>0.2.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">&lt;!--&nbsp;Spring&nbsp;Boot相关依赖&nbsp;--&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependencies</span>&gt;</span>
</code></pre>
<p>demo-provider 模块中的 DefaultHelloService 实现了 HelloService 接口，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Service</span>
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">DefaultHelloService</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">HelloService</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">say</span><span class="hljs-params">(String&nbsp;name)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">2000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-string">"hello"</span>&nbsp;+&nbsp;name;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>在 resource/application.yml 配置文件中将 DefaultHelloService 实现注册到 Zookeeper上对外暴露为 Dubbo Provider，具体配置如下：</p>
<pre><code data-language="js" class="lang-js">dubbo:
&nbsp;&nbsp;application:
&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;demo-provider&nbsp;#&nbsp;Dubbo&nbsp;Provider&nbsp;的名字
&nbsp;&nbsp;registry:
&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;注册中心地址，即前面启动的Zookeeper地址
&nbsp;&nbsp;&nbsp;&nbsp;address:&nbsp;zookeeper://127.0.0.1:2181&nbsp;
&nbsp;&nbsp;protocol:
&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;dubbo&nbsp;#&nbsp;指定通信协议
&nbsp;&nbsp;&nbsp;&nbsp;port:&nbsp;20880&nbsp;#&nbsp;通信端口，这里指的是与消费者间的通信协议与端口
&nbsp;&nbsp;provider:
&nbsp;&nbsp;&nbsp;&nbsp;timeout:&nbsp;10000&nbsp;#&nbsp;配置全局调用服务超时时间，dubbo默认是1s，肯定不够用呀
&nbsp;&nbsp;&nbsp;&nbsp;retries:&nbsp;0&nbsp;#&nbsp;不进行重试
&nbsp;&nbsp;&nbsp;&nbsp;delay:&nbsp;-1
</code></pre>
<p>在 DemoProviderApplication 中提供 Spring Boot 的启动 main() 方法，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@EnableDubbo</span>&nbsp;<span class="hljs-comment">//&nbsp;添加对&nbsp;Dubbo支持的注解</span>
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">DemoProviderApplication</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpringApplication.run(DemoProviderApplication<span class="hljs-class">.<span class="hljs-keyword">class</span>,&nbsp;<span class="hljs-title">args</span>)</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>为了引入 Skywalking Agent 插件，还需要将 apache-skywalking-apm-bin/agent/config 目录下的 agent.config 配置文件拷贝到 demo-provider 模块的 resource 目录下，并修改其中的 agent.service_name：</p>
<pre><code data-language="js" class="lang-js">#&nbsp;The&nbsp;service&nbsp;name&nbsp;in&nbsp;UI
agent.service_name=${SW_AGENT_NAME:demo-provider}
</code></pre>
<p>很明显，agent.config 是一个 KV 结构的配置文件，类似于 properties 文件，value 部分使用 "${}" 包裹，其中使用冒号（":"）分为两部分，前半部分是可以覆盖该配置项的系统环境变量名称，后半部分为默认值。例如这里的 agent.service_name 配置项，如果系统环境变量中指定了 SW_AGENT_NAME 值（注意，全是大写），则优先使用环境变量中指定的值，如果环境变量未指定，则使用 demo-provider 这个默认值。</p>
<p>除了系统环境变量的覆盖方式，SkyWalking Agent 还支持另外两种覆盖默认值的方式：</p>
<ul>
<li><strong>JVM 配置覆盖</strong></li>
</ul>
<p>例如这里的 agent.service_name 配置项，如果在 JVM 启动之前，明确中指定了下面的 JVM 配置：</p>
<pre><code data-language="js" class="lang-js">-Dskywalking.agent.service_name&nbsp;=&nbsp;demo-provider
#&nbsp;"skywalking."是&nbsp;Skywalking环境变量的默认前缀
</code></pre>
<p>则会使用该配置值覆盖 agent.config 配置文件中默认值。</p>
<ul>
<li><strong>探针配置覆盖</strong></li>
</ul>
<p>如果将 Java Agent 配置为如下：</p>
<pre><code data-language="java" class="lang-java">-javaagent:/path/skywalking-agent.jar=agent.service_name=demo-provider
#&nbsp;默认格式是&nbsp;-javaagent:agent.jar=[option1]=[value1],[option2]=[value2]
</code></pre>
<p>则会使用该 Java Agent 配置值覆盖 agent.config 配置文件中 agent.service_name 默认值。</p>
<p>如果四种配置同时出现，则优先级如下：</p>
<pre><code>探针配置&nbsp;&gt;&nbsp;JVM配置&nbsp;&gt;&nbsp;系统环境变量配置&nbsp;&gt;&nbsp;agent.config文件默认值
</code></pre>
<p>编辑好 agent.config 配置文件之后，我们需要在启动 demo-provider 之前通过参数告诉 JVM SkyWalking Agent 配置文件的位置，IDEA 中的配置如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMCAPGmdAAJd59aKb9w948.png" alt=""></p>
<p>最后启动 DemoProviderApplication 这个入口类，可以看到如下输出：</p>
<pre><code data-language="java" class="lang-java">#&nbsp;查找到&nbsp;agent.config&nbsp;配置文件
INFO&nbsp;2020-02-01&nbsp;12:12:07:574&nbsp;main&nbsp;SnifferConfigInitializer&nbsp;:&nbsp;&nbsp;Config&nbsp;file&nbsp;found&nbsp;in&nbsp;...&nbsp;agent.config.&nbsp;
#&nbsp;查找到&nbsp;agent目录
DEBUG&nbsp;2020-02-01&nbsp;12:12:07:650&nbsp;main&nbsp;AgentPackagePath&nbsp;:&nbsp;&nbsp;The&nbsp;beacon&nbsp;class&nbsp;location&nbsp;is&nbsp;jar:file:/Users/xxx/...&nbsp;
#&nbsp;Dubbo&nbsp;Provider&nbsp;注册成功
2020-02-01&nbsp;12:12:16.105&nbsp;&nbsp;INFO&nbsp;58600&nbsp;---&nbsp;[main]&nbsp;c.a.d.r.zookeeper.ZookeeperRegistry&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;:&nbsp;&nbsp;[DUBBO]&nbsp;Register:&nbsp;dubbo://172.17.32.91:20880/com.xxx.service.HelloService
#&nbsp;demo-provider&nbsp;启动成功
2020-02-01&nbsp;12:12:16.269&nbsp;&nbsp;INFO&nbsp;58600&nbsp;---&nbsp;[&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;main]&nbsp;com.xxx.DemoProviderApplication&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;:&nbsp;Started&nbsp;DemoProviderApplication&nbsp;in&nbsp;4.635&nbsp;seconds&nbsp;(JVM&nbsp;running&nbsp;for&nbsp;9.005)
</code></pre>
<h3>demo-webapp 模块</h3>
<p>完成 demo-provider 模块的启动之后，我们继续来开发 demo-webapp 模块，其 pom.xml 与 demo-provider 中的 pom.xml 相比，多引入了 spring-boot 对 Web开发的依赖，以及 SkyWalking 提供的 apm-toolkit-trace 依赖用来获取 TraceId：</p>
<pre><code data-language="html" class="lang-html">&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>

<span class="hljs-comment">&lt;!--&nbsp;apm-toolkit-trace&nbsp;这个依赖主要用来获取&nbsp;TraceId&nbsp;--&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.skywalking<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>apm-toolkit-trace<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>6.2.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p>首先，在 HelloWorldController 中提供了两个接口：</p>
<ul>
<li><strong>/hello/{words} 接口</strong>：通过 Dubbo 远程调用 demo-provider 暴露的接口。</li>
<li><strong>/err 接口</strong>：直接抛出 RuntimeException 异常。</li>
</ul>
<p>HelloWorldController 的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping</span>(<span class="hljs-string">"/"</span>)
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">HelloWorldController</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Reference</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;HelloService&nbsp;helloService;

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@GetMapping</span>(<span class="hljs-string">"/hello/{words}"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">hello</span><span class="hljs-params">(@PathVariable(<span class="hljs-string">"words"</span>)</span>&nbsp;String&nbsp;words)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;TraceContext&nbsp;工具类定义在&nbsp;apm-toolkit-trace&nbsp;依赖包中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;log.info(<span class="hljs-string">"traceId:{}"</span>,&nbsp;TraceContext.traceId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ActiveSpan.tag(<span class="hljs-string">"hello-trace"</span>,&nbsp;words);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;say&nbsp;=&nbsp;helloService.say(words);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;say;
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@GetMapping</span>(<span class="hljs-string">"/err"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">err</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;traceId&nbsp;=&nbsp;&nbsp;TraceContext.traceId();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;log.info(<span class="hljs-string">"traceId:{}"</span>,&nbsp;traceId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ActiveSpan.tag(<span class="hljs-string">"error-trace&nbsp;activation"</span>,&nbsp;<span class="hljs-string">"error"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span>&nbsp;<span class="hljs-keyword">new</span>&nbsp;RuntimeException(<span class="hljs-string">"err"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
</code></pre>
<p>在 resources/application.yml 文件中会配置 demo-webapp 监听的端口、Zookeeper 地址以及 Dubbo Consumer 的名称等等，具体配置如下：</p>
<pre><code data-language="js" class="lang-js">server:
&nbsp;&nbsp;port:&nbsp;8000

dubbo:
&nbsp;&nbsp;application:
&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;demo-webapp&nbsp;#&nbsp;Dubbo&nbsp;Consumer名字
&nbsp;&nbsp;registry:
&nbsp;&nbsp;&nbsp;&nbsp;address:&nbsp;zookeeper://127.0.0.1:2181&nbsp;#&nbsp;注册中心地址，即&nbsp;Zookeeper地址
</code></pre>
<p>demo-webpp 模块也需要在 resource 目录下添加 agent.config 配置文件，并修改其 agent.service_name 配置项，如下所示：</p>
<pre><code data-language="java" class="lang-java">#&nbsp;The&nbsp;service&nbsp;name&nbsp;in&nbsp;UI
agent.service_name=${SW_AGENT_NAME:demo-webapp}
</code></pre>
<p>demo-webpp 模块的入口 main() 方法与 demo-provider 相同，不再赘述。</p>
<p>为了接入 SkyWalking Agent，启动 demo-webapp 项目之前也需要配置相应的 VM options 参数，指定 agent.config 配置文件的地址，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMCAJ8EKAAIq6sznBy0160.png" alt=""></p>
<p>最后，启动 demo-webapp 项目，通过浏览器访问&nbsp;<a href="http://localhost:8000/hello/xxx">http://localhost:8000/hello/xxx</a>&nbsp;地址得到正常相应，访问&nbsp;<a href="http://localhost:8000/err">http://localhost:8000/err</a>&nbsp;得到 500 响应，即表示启动成功。</p>
<p>到此为止，SkyWalking Agent 的基本接入方式就介绍完了，在后面分析和改造 SkyWalking 源码时，还可以使用 demo-webapp 和 demo-provider 这两个应用来产生 Trace 和 Metrics 数据。</p>
<h3>SkyWalking Rocketbot 使用</h3>
<p>搭建完 SkyWalking 环境以及相关示例之后，我们来看如何使用 SkyWalking 提供的 UI 界面—— Skywalking Rocketbot。在前面执行的 ./bin/startup.sh 脚本，除了启动后端 OAP 服务，同时还会启动 Skywalking Rocketbot（位于 webapp 目录下的 skywalking-webapp.jar）。</p>
<p>如下图所示，在 Skywalking Rocketbot 首页顶部（1）处，有四个主 Tab 页，在【仪表盘】这个 Tab 中，（2）处可以选择查询的服务（Service）、端点（Endpoint） 以及服务实例（ServiceInstance）。在（3）处可以选择展示的不同维度，下图展示了 Global 这个全局视图：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMCASh6sAAFikMiwg_o077.png" alt=""></p>
<p>其中有五个面板（（4）~（8）），分别是：</p>
<ul>
<li><strong>Global Heatmap 面板</strong>：热力图，从全局展示了某段时间请求的热度。</li>
<li><strong>Global Percent Response 面板</strong>&nbsp;：展示了全局请求响应时间的 P99、P95、P75 等分位数。</li>
<li><strong>Global Brief 面板</strong>：展示了 SkyWalking 能感知到的 Service、Endpoint 的个数。</li>
<li><strong>Global Top Troughput 面板</strong>：展示了吞吐量前几名的服务。</li>
<li><strong>Global Top Slow Endpoint 面板</strong>：展示了耗时前几名的 Endpoint。</li>
</ul>
<p>除了 SkyWalking Rocketbot 默认提供的这些面板，我们还可以点击（2）处左边的锁型按钮，自定义 Global 面板。另外，我们还可以通过（9）处的时间选择框选择自定义查询的时间段。</p>
<p>将（3）处切换到 Service 面板，可以看到针对 Service 的监控面板，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMGACMCRAAEzi1SuLKw557.png" alt=""></p>
<ul>
<li><strong>Service (Avg) ResponseTime 面板</strong>：展示了指定服务的（平均）耗时。</li>
<li><strong>Service (Avg) Throughput 面板</strong>：展示了指定服务的（平均）吞吐量。</li>
<li><strong>Service (Avg) SLA 面板</strong>：展示了指定服务的（平均）SLA（Service Level Agreement，服务等级协议）。</li>
<li><strong>Service Percent Response 面板</strong>：展示了指定服务响应时间的分位数。</li>
<li><strong>Service Slow Endpoint 面板</strong>：展示了指定服务中耗时比较长的 Endpoint 信息。</li>
<li><strong>Running ServiceInstance 面板</strong>：展示了指定服务下的实例信息。</li>
</ul>
<p>将（3）处切换到 Endpoint 面板，可以看到针对 Endpoint 的监控面板，基本功能与 Service 面板类似，这里不再展开。</p>
<p>将（3）处切换到 Instance 面板，可以看到针对 ServiceInstance 的监控面板，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMGAGZlwAAHCkizkpVs483.png" alt=""> &nbsp; &nbsp; &nbsp;<br>
在 ServiceInstance 面板中展示了很多 ServiceInstance 相关的监控信息，例如，JVM 内存使用情况、GC 次数、GC 耗时、CPU 使用率、ServiceInstance SLA 等等信息，这里不再一一展开介绍。</p>
<p>下面我们切换到【拓扑图】这个主 Tab，如下图所示，在（1）处展示当前整个业务服务的拓扑图。点击拓扑图中的任意节点，可在（2）处看到服务相应的状态信息，其中包括响应的平均耗时、SLA 等监控信息。点击拓扑图中任意一条边，可在（3）处看到一条调用链路的监控信息，其中会分别从客户端（上游调用方）和服务端（下游接收方）来观测这条调用链路的状态，其中展示了该条链路的耗时、吞吐量、SLA 等信息：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMGAceZOAAHozO2Mq14310.png" alt=""></p>
<p>下面我们切换到【追踪】这个主 Tab来查询 Trace 信息，如下图所示。在（1）、（2）处可以选择 Trace 的查询条件，其中可以指定 Trace 涉及到的 Service、ServiceInstance、Endpoint 以及Trace 的状态继续模糊查询，还可以指定 TraceId 和时间范围进行精确查询。在（3）处展示了 Trace 的简略信息，下图中 "/err" 接口这条 Trace 被显示为红色表示该 Trace 关联的请求出现了异常。在（4）和（5）处展示了 Trace 的具体信息以及所有 Span 信息，我们可以通过（6）处按钮调整 Span 的展示方式：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMGAUPpGAAEyWm6Aqo8753.png" alt=""></p>
<p>点击 Trace 中的 Span，就可以将该 Span 的具体信息展示出来，如下下图所示，点击"/err" 接口相关 Trace 中的 Span，即可看到相应的 TRuntimeException 异常信息：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/CgpOIF5nHMGAb41NAAHBIhP98Z0352.png" alt=""></p>
<p>最后，我们将主 Tab 也切换到【告警】，这里展示了 Skywalking 发出来的告警信息，如下图所示，这里也提供了相应的查询条件和关键字搜索框。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/71/BD/Cgq2xl5nHMGAVek0AAM9HlRD-nQ059.png" alt=""></p>
<h3>总结</h3>
<p>本课时搭建 SkyWalking 的运行环境，完成 ElasticSearch、Kibana、Skywalking 等的安装，并搭建了 skywalking-demo 项目作为演示示例，带同学们上手体验了 Skywalking Agent 的接入的流程。</p>
<p>最后介绍了 SkyWalking Rocketbot UI 界面强大的功能，包括 Service、Endpoint、ServiceInstance 等不同级别的监控，展示了整个服务的拓扑图、Trace 查询以及告警信息查询等功能。</p>

---

### 精选评论

##### *宇：
> 有人获取到demo的代码了么？分享下啊！

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 源码地址：https://github.com/xxxlxy2008/skywalking-demo

##### *征：
> 我配置的跟你的一模一样，为什么skywalking&nbsp;web界面 始终无法获取到Java 服务信息？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看一下日志，是否有异常或是 ERROR 日志

##### **洁：
> 老师讲的特别清晰。我搭建了环境，skywalking上有四五个应用，起初都能监控到请求，跑了一天后，之后的skywalking会空白。是es配置问题导致？虚拟机硬盘空间还很多，这个能作为线上监控？是不是要定期清理什么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看一下日志，是否有异常或是 ERROR 日志

##### **驰：
> 能不能把代码也同步提供下，更新能不能快点

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以关注拉勾教育公众号，跟客服要课件～
已反馈给讲师，后期会加快更新节凑

##### **驰：
> 能不能把代码也分享下

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注 拉勾教育 公众号 咨询小助手获取课件

##### **民：
> <div>ActiveSpan这个为啥写到代码里面？</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是用来添加 Tag 和 Log 的工具箱，可以删掉，不影响这里的 Demo 功能。第二部分的最后，会介绍 ActiveSpan 的实现原理

##### **7083：
> 图片看不了大图

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 点击图片后就可以放大图片了，或者可以关注拉勾教育，咨询小助手获取课件哈

##### *康：
> 周末想学习，麻烦更新快点

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 已反馈给讲师，后期会加快更新节凑

##### **4438：
> 统一用7.x版本的可以吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在开始写的时候，最新的版本6.x，其实 7.x 和 6.x 的核心原理基本相同的。以一人之力，追一个开源社区的更新，很快就追不上了

##### OneThin：
> <div>用自己的项目集成了一下，实例的指标数据有了，可以看到jvm的信息，但是请求统计数据没有，调用链数据也没有，可能是什么原因？springmvc+dubbo的项目，调用是成功的，</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 具体要看一下 dubbo 版本以及 springmvc 版本与 SkyWalking 的 插件版本是否匹配

##### *军：
> 建议每节课的源代码放在git上面，这样大家都可以去下载，更加方便...

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 收到建议！可以关注拉勾教育公众号 咨询小助手获取源代码～

##### **办法：
> 请问为什么不使用SW6.6.0

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在开始写的时候，最新的版本6.3，其实 7.x 和 6.x 的核心原理基本相同的。以一人之力，追一个开源社区的更新，很快就追不上了

##### **麟：
> 我使用h2默认有数据进来。按你的方式在application.yml把elastisearch7节点取消注释，把里面的es地址改成自己现有的es地址，h2节点注释。重新启动startup.bat ，oapservice这个一闪而过。数据进不来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看一下日志，是否有异常或是 ERROR 日志

##### **9882：
> 1. 老师 可否放出代码的demo 下载&nbsp;<div>2. 同时 skywalking 启动后是英文的 和你的不一样 是怎么设置的</div><div>3. 老师少说了一个地方 在启动elasticsearch的时候 需要在elasticsearch.yml 文件中增加一行&nbsp;<span style="color: rgb(40, 240, 239); background-color: rgba(0, 0, 0, 0.85098); -webkit-text-size-adjust: 100%; font-family: CourierNewPS-BoldMT; font-weight: bold; font-size: 16.88px; font-variant-ligatures: no-common-ligatures;">xpack.ml.enabled</span><span style="font-family: &quot;Courier New&quot;; background-color: rgba(0, 0, 0, 0.85098); -webkit-text-size-adjust: 100%; font-size: 16.88px; font-variant-ligatures: no-common-ligatures; color: rgb(255, 221, 221);">:</span><span style="font-family: &quot;Courier New&quot;; background-color: rgba(0, 0, 0, 0.85098); -webkit-text-size-adjust: 100%; font-size: 16.88px; font-variant-ligatures: no-common-ligatures; color: rgb(244, 244, 244);"> </span><span style="font-family: &quot;Courier New&quot;; background-color: rgba(0, 0, 0, 0.85098); -webkit-text-size-adjust: 100%; font-size: 16.88px; font-variant-ligatures: no-common-ligatures; color: rgb(255, 60, 255);">false</span></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; sky walking 源码地址：https://github.com/xxxlxy2008/skywalking-demo

嗯，ElasticSearch 默认配置也可以运行，也可以按照您说的，自定义配置

##### *鑫：
> 求更新快点😁

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 每周二、四更新，一周更新两个课时哈

##### *榕：
> 你好，SLA是代表了什么的指标呢，服务可靠度吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，对互联网公司来说就是网站服务可用性，一般是4个9或是5个9，9越多代表全年服务可用时间越长，服务更可靠，停机时间越短。

##### **斌：
> 明天亲自搭一下试试，实践一下

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 实践是检验真理的唯一标准，加油啊～

##### **达：
> <span style="font-size: 16.0125px;">咨询下&nbsp;</span>示例代码，有可供下载且能直接使用的吗？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; sky walking 源码地址：https://github.com/xxxlxy2008/skywalking-demo

