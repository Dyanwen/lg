<p data-nodeid="56837">上一讲我们掌握了基于 Sleuth+Zipkin 对微服务架构实施基于日志的链路追踪，通过 Sleuth 在微服务应用中附加链路数据，再通过 Zipkin 实现链路数据收集与可视化，从而保证开发与运维人员在生产环境了解微服务的执行过程与具体细节，为产品运维提供了有力的保障。</p>


<p data-nodeid="55525">本讲咱们还是围绕链路追踪这个话题，介绍另一款著名的链路追踪产品 SkyWalking，掌握 SkyWalking 的使用方法。本讲咱们将介绍三方面内容：</p>
<ul data-nodeid="55526">
<li data-nodeid="55527">
<p data-nodeid="55528">介绍 APM 与 SkyWalking；</p>
</li>
<li data-nodeid="55529">
<p data-nodeid="55530">部署 SkyWalking 服务端与 Java Agent；</p>
</li>
<li data-nodeid="55531">
<p data-nodeid="55532">介绍 SkyWalking 常用视图。</p>
</li>
</ul>
<h3 data-nodeid="60330" class="">APM 与 SkyWalking</h3>




<p data-nodeid="55534">这些年随着微服务体系的不断完善，链路追踪已经不是什么新兴的概念与技术，很多厂商也提供了自己的链路追踪产品，例如 Spring Cloud Slueth、Zipkin、阿里鹰眼、大众点评 Cat、SkyWalking 等。但这些产品都有一个共同的名字：APM（Application Performance Management），即应用性能管理系统。顾名思义这些产品的根本目的就是对应用程序单点性能与整个分布式应用进行监控，记录每一个环节程序执行状况，通过图表与报表的形式让运维人员随时掌握系统的运行状况，其中在这些著名的产品中我非常推荐各位掌握 SkyWalking 这款 APM 产品，理由很简单，它在简单易用的前提下实现了比 Zipkin 功能更强大的链路追踪、同时拥有更加友好、更详细的监控项，并能自动生成可视化图表。相比 Sleuth+Zipkin 这种不同厂商间混搭组合，SkyWalking 更符合国内软件业的“一站式解决方案”的设计理念，下面咱们来了解下 SKyWalking。</p>
<p data-nodeid="55535">SkyWalking 是中国人吴晟（华为）开源的应用性能管理系统（APM）工具，使用Java语言开发，后来吴晟将其贡献给 Apache，在 Apache 的背书下 SkyWalking 发展迅速，现在已属于 Apache 旗下顶级开源项目，它的官网：<a href="http://skywalking.apache.org/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="55718">http://skywalking.apache.org/</a>。</p>
<p data-nodeid="55536">SkyWalking 提供了分布式追踪、服务网格遥测分析、度量聚合和可视化一体化解决方案。目前在 GitHub 上 SkyWaking 拥有 15.9K Star，最新版本为：8.3.0。</p>
<p data-nodeid="62063" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/B0/CioPOWBld3yAYjnPAAW7sf0vPwg338.png" alt="图片1.png" data-nodeid="62067"></p>
<div data-nodeid="62064"><p style="text-align:center">链路追踪视图</p></div>



<p data-nodeid="63795" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/A8/Cgp9HWBld4SAPAAzAAWVdqvX5xc517.png" alt="图片2.png" data-nodeid="63799"></p>
<div data-nodeid="63796"><p style="text-align:center">指标监控全局视图</p></div>



<p data-nodeid="55541">为了能够让各位有个直观认识，我们通过 Sleuth+Zipkin 与 SkyWalking 做对比，看两者的优劣。</p>
<p data-nodeid="74099" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B3/CioPOWBlfPKAMpDiAAFs6fz0JjQ094.png" alt="202141-155716.png" data-nodeid="74102"></p>

<p data-nodeid="55590" class="">通过比较我们可以发现，在易用性和使用体验上，SkyWalking 明显好于 Zipkin，功能更丰富的同时也更符合国人习惯，但因为迭代速度较快，社区文档相对陈旧，这也导致很多技术问题需要程序员自己研究解决，因此在解决问题方面需要更多的时间。</p>
<h3 data-nodeid="55591">部署 SkyWalking 服务端与 Java Agent</h3>
<p data-nodeid="55592">在了解 SkyWalking 后，咱们正式进入 SkyWalking 的安装与使用吧。</p>
<h4 data-nodeid="67251" class="">部署 SkyWalking 服务端</h4>




<p data-nodeid="55594">首先咱们要理解 SkyWalking 架构图</p>
<p data-nodeid="68971"><img src="https://s0.lgstatic.com/i/image6/M00/2C/B3/CioPOWBlfLeAdmXOAARZeR55FBs329.png" alt="图片3.png" data-nodeid="68975"></p>
<div data-nodeid="68972" class=""><p style="text-align:center">SkyWalking 的架构图</p></div>



<p data-nodeid="55597" class="">SkyWalking 同样采用客户端与服务端架构模式，SkyWalking 服务端用于接收来自 Java Agent 客户端发来的链路跟踪与指标数据，汇总统计后由 SkyWalking UI 负责展现。SkyWalking 服务端同时支持 gRPC 与 HTTP 两种上报方式。其中 gRPC 默认监听服务器 11800 端口，HTTP 默认监听 12800 端口，而 SKyWalking UI 应用则默认监听 8080 端口，这三个端口在生产环境下要在防火墙做放行配置。在存储层面，SkyWalking 底层支持 ElasticSearch 、MySQL、H2等多种数据源，官方优先推荐使用 ElasticSearch，如果此时你不会 ElasticSearch 也没关系，按文中步骤操作也能完成部署。</p>
<p data-nodeid="55598">首先咱们根据架构图部署 SkyWalking 服务端。</p>
<p data-nodeid="55599"><strong data-nodeid="55776">第一步，安装 ElasticSearch 全文检索引擎。</strong></p>
<p data-nodeid="55600">ElasticSearch 简称 ES，是业内最著名的全文检索引擎，常用于构建站内搜索引擎，SkyWalking 官方推荐使用 ES 作为数据存储组件。这里直接访问 ES 官网下载页：</p>
<p data-nodeid="55601"><a href="https://www.elastic.co/cn/downloads/elasticsearch?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="55780">https://www.elastic.co/cn/downloads/elasticsearch</a></p>
<p data-nodeid="55602">下载对应平台的 ES 服务器程序。</p>
<p data-nodeid="71536" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B3/CioPOWBlfM-APF4nAAIJZ6bVGsI987.png" alt="图片4.png" data-nodeid="71540"></p>
<div data-nodeid="71537"><p style="text-align:center">ElasticSearch 下载页</p></div>




<p data-nodeid="55605">下载后会得到 elasticsearch-7.10.2-windows-x86_64.zip 文件，解压缩后无须任何修改直接运行 bin/elasticsearch.bat 文件，如果是 Linux 系统则运行 elasticsearch.sh 文件。</p>
<p data-nodeid="73244" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/AB/Cgp9HWBlfNqAcFzWABLVO06FsDw638.png" alt="图片5.png" data-nodeid="73248"></p>
<div data-nodeid="73245"><p style="text-align:center">ElasticSearch 启动成功画面</p></div>



<p data-nodeid="55608">默认 ES 监听 9200 与 9300 端口，其中 9200 是 ES 对外提供服务的端口；9300 是 ES 进行集群间通信与数据传输的端口，请确保这两个端口没有被占用。</p>
<p data-nodeid="55609"><strong data-nodeid="55797">第二步，下载 SkyWalking。</strong></p>
<p data-nodeid="55610">访问<a href="https://skywalking.apache.org/downloads/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="55801">https://skywalking.apache.org/downloads/</a>，下载最新版 SkyWalking 8.3.0，注意版本选择 v8.3.0 for ES7。</p>
<p data-nodeid="76213" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B3/CioPOWBlfQSAZ1COAAOgO9OuHFY248.png" alt="图片6.png" data-nodeid="76217"></p>
<div data-nodeid="76214"><p style="text-align:center">SkyWalking 下载页</p></div>




<p data-nodeid="55613">下载完毕，解压缩文件得到 apache-skywalking-apm-bin-es7 目录。这里有个重要细节，SkyWalking 路径不要包含任何中文、特殊字符甚至是空格，否则启动时会报“找不到模板文件”的异常。</p>
<p data-nodeid="78325" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfQ2ADrQqAAX_hq6BcNY797.png" alt="图片7.png" data-nodeid="78329"></p>
<div data-nodeid="78326"><p style="text-align:center">SkyWalking 目录</p></div>




<p data-nodeid="55616"><strong data-nodeid="55815">第三步，配置 SkyWalking 数据源。</strong></p>
<p data-nodeid="55617">SKyWalking 收集到的数据要被存储在 ElasticSearch 中，因此需要指定数据源。在 config 目录下找到 application.yml，这是 SkyWalking 的核心配置文件。</p>
<p data-nodeid="79729" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfRWAEC-7AAHrDj6mnPM675.png" alt="图片8.png" data-nodeid="79733"></p>
<div data-nodeid="79730"><p style="text-align:center">SkyWalking 核心配置文件</p></div>



<p data-nodeid="55620">在配置文件 103 行附近可以看到 storage 配置片段。</p>
<pre class="lang-yaml" data-nodeid="55621"><code data-language="yaml"><span class="hljs-attr">storage:</span>
  <span class="hljs-attr">selector:</span> <span class="hljs-string">${SW_STORAGE:h2}</span>
  <span class="hljs-attr">elasticsearch:</span> <span class="hljs-comment">#ES6配置 </span>
    <span class="hljs-string">...</span>
  <span class="hljs-attr">elasticsearch7:</span> <span class="hljs-comment">#ES7配置</span>
    <span class="hljs-attr">nameSpace:</span> <span class="hljs-string">${SW_NAMESPACE:""}</span>
    <span class="hljs-attr">clusterNodes:</span> <span class="hljs-string">${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}</span>
    <span class="hljs-attr">protocol:</span> <span class="hljs-string">${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}</span>
    <span class="hljs-string">...</span>
</code></pre>
<p data-nodeid="80432">默认 SkyWalking 采用内置 H2 数据库存储监控数据，现在需要改为 elasticsearch7，这样就完成了数据源存储的切换，在启动时 SkyWalking 会自动初始化 ES 的索引。</p>
<p data-nodeid="80433">修改前：</p>

<pre class="lang-java" data-nodeid="55623"><code data-language="java">selector: ${SW_STORAGE:h2}
</code></pre>
<p data-nodeid="55624">修改后：</p>
<pre class="lang-java" data-nodeid="55625"><code data-language="java">selector: ${SW_STORAGE:elasticsearch7}
</code></pre>
<p data-nodeid="81134">到这里，SkyWalking 数据源配置成功。</p>
<p data-nodeid="81135"><strong data-nodeid="81140">第四步，启动 SkyWalking 应用。</strong></p>

<p data-nodeid="55627">在 bin 目录下找到 startup.bat 运行，如果是 Linux 系统运行 startup.sh。</p>
<p data-nodeid="83236" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfSeAQNnwAAJkQmq3QUU512.png" alt="图片9.png" data-nodeid="83240"></p>
<div data-nodeid="83237"><p style="text-align:center">SkyWalking 启动文件</p></div>




<p data-nodeid="55630">启动后会产生两个 Java 进程：</p>
<ul data-nodeid="55631">
<li data-nodeid="55632">
<p data-nodeid="55633">Skywalking-Collector 是数据收集服务，默认监听 11800（gRPC）与 12800（HTTP） 端口。</p>
</li>
<li data-nodeid="55634">
<p data-nodeid="55635">Skywalking-Webapp 是 SkyWalking UI，用于展示数据，默认监听 8080 端口。</p>
</li>
</ul>
<p data-nodeid="85330" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfTCAI-XvAAHnlAR2Ayk280.png" alt="图片10.png" data-nodeid="85334"></p>
<div data-nodeid="85331"><p style="text-align:center">Skywalking 应用已启动</p></div>




<p data-nodeid="55638">启动成功后，访问<a href="http://192.168.31.10:8080/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="55847">http://192.168.31.10:8080/</a>，如果看到 SkyWalking UI 首页，则说明服务端配置成功。</p>
<p data-nodeid="86722" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/AB/Cgp9HWBlfTuAJKIFAAIgcUoZfD4258.png" alt="图片11.png" data-nodeid="86726"></p>
<div data-nodeid="87419" class=""><p style="text-align:center">SkyWalking UI 首页</p></div>




<p data-nodeid="55641">到这里，SkyWalking 服务端启动完毕，下面咱们来说明如何通过 SkyWalking Java Agent 上报链路数据。</p>
<h4 data-nodeid="90191" class="">安装 SkyWalking Java Agent</h4>




<p data-nodeid="55643">在前面提到，SkyWalking 可以在不修改应用源码的前提下，无侵入的实现链路追踪与 JVM 指标监控，它是怎么做到的？这里涉及一个 Java1.5 新增的特性，Java Agent 探针技术，想必对于很多工作多年 Java 工程师来说，Java Agent 也是一个陌生的东西。</p>
<p data-nodeid="55644">Java Agent 探针说白了就是 Java 提供的一种“外挂”技术，允许在应用开发的时候在通过启动时增加 javaagent 参数来外挂一些额外的程序。</p>
<p data-nodeid="55645">Java Agent 并不复杂，其扩展类有这严格的规范，必须创建名为 premain 的方法，该方法将在目标应用 main 方法前执行，下面就是最简单的 Java Agent 扩展类。</p>
<pre class="lang-java" data-nodeid="55646"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SimpleAgent</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">premain</span><span class="hljs-params">(String agentArgs, Instrumentation inst)</span> </span>{
        System.out.println(<span class="hljs-string">"=========开始执行premain============"</span>);
    }
}
</code></pre>
<p data-nodeid="55647">要完成 Java Agent，还需要提供正确的 MANIFEST.MF，以便 JVM 能够选择正确的类。在 META-INF 目录下找到你的 MANIFEST.MF 文件：</p>
<pre class="lang-java" data-nodeid="55648"><code data-language="java">Manifest-Version: <span class="hljs-number">1.0</span>
Premain-Class: com.lagou.agent.SimpleAgent
</code></pre>
<p data-nodeid="55649">之后我们将这个类打包为 agent.jar，假设原始应用为 oa.jar，在 oa.jar 启动时需要在额外附加 javaagent 参数，如下所示：</p>
<pre class="lang-java" data-nodeid="55650"><code data-language="java">java -javaagent:agent.jar -jar oa.jar
</code></pre>
<p data-nodeid="55651">在应用启动时 Java 控制台会输出如下日志。</p>
<pre class="lang-java" data-nodeid="55652"><code data-language="java">=========开始执行 premain============
正在启动 OA 办公自动化系统...
....
</code></pre>
<p data-nodeid="90879">通过结果你会发现 java agent 在目标应用main执行前先执行了premain，实现了不修改OA源码的前提下增加了新的功能。</p>
<p data-nodeid="90880">SkyWalking 也是利用 Java Agent 的特性，在 premain 中通过字节码增强技术对目标方法进行扩展，当目标方法执行时自动收集链路追踪及监控数据并发往 SkyWalking 服务端。</p>

<p data-nodeid="55654">下面咱们来讲解如何加载并使用 SkyWalking Java Agent，我们还是以实例进行讲解，因为 Java Agent 是无侵入的，并不需要源码，这里我就直接给出调用关系图帮助咱们理解。</p>
<p data-nodeid="92254" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfU6AGr3LAAFCNrZQT4I548.png" alt="图片12.png" data-nodeid="92258"></p>
<div data-nodeid="92255"><p style="text-align:center">调用关系图</p></div>



<p data-nodeid="55657">简单介绍下，用户访问 a 服务的 a 接口，a 服务通过 OpenFeign 远程调用 b 服务的 b 接口，b 服务通过 OpenFeign 调用 c 服务的 c 接口，最后 c 接口通过 JDBC 将业务数据存储到 MySQL 数据库。</p>
<p data-nodeid="55658">下面咱们演示 SkyWalking Java Agent 的用法，在 skywalking 的 agent 目录下存在 skywalking-agent.jar，这就是 skywalking 提供的 Java Agent 扩展类。</p>
<p data-nodeid="93626" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/AB/Cgp9HWBlfVeAUDQOAALeSr7VkDQ887.png" alt="图片13.png" data-nodeid="93629"></p>

<div data-nodeid="92943" class=""><p style="text-align:center">SkyWalking Java Agent</p></div>

<p data-nodeid="55661">如果是生产环境下在启动应用时附加 javaagent 参数即可。</p>
<p data-nodeid="55662">a 服务启动命令：</p>
<pre class="lang-java" data-nodeid="55663"><code data-language="java">java -javaagent:D:\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar -Dskywalking.agent.service_name=a-service -Dskywalking.collector.backend_service=<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span>:<span class="hljs-number">11800</span> -Dskywalking.logging.file_name=a-service-api.log -jar a-service.jar
</code></pre>
<p data-nodeid="55664">b 服务启动命令：</p>
<pre class="lang-java" data-nodeid="55665"><code data-language="java">java -javaagent:D:\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar -Dskywalking.agent.service_name=b-service -Dskywalking.collector.backend_service=<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span>:<span class="hljs-number">11800</span> -Dskywalking.logging.file_name=b-service-api.log -jar b-service.jar
</code></pre>
<p data-nodeid="55666">c 服务启动命令：</p>
<pre class="lang-java" data-nodeid="55667"><code data-language="java">java -javaagent:D:\apache-skywalking-apm-bin-es7\agent\skywalking-agent.jar -Dskywalking.agent.service_name=c-service -Dskywalking.collector.backend_service=<span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span>:<span class="hljs-number">11800</span> -Dskywalking.logging.file_name=c-service-api.log -jar c-service.jar
</code></pre>
<p data-nodeid="99758">如果是在 idea 开发环境运行，需要在 VM options 附加 javaagent。</p>
<p data-nodeid="99759" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/AB/Cgp9HWBlfXyAYRUQAAH2GtG-8us746.png" alt="图片14.png" data-nodeid="99764"></p>
<div data-nodeid="99760"><p style="text-align:center">IDEA 中使用 SkyWalking Java Agent</p></div>















<p data-nodeid="55670">除了 javaagent 指定具体 agent 文件外，agent 本身也支持一系列配置参数，在刚才的启动时涉及 3 个。</p>
<ul data-nodeid="55671">
<li data-nodeid="55672">
<p data-nodeid="55673"><strong data-nodeid="55895">skywalking.agent.service_name</strong>：指定在 SkyWalking 上报数据时的服务名。</p>
</li>
<li data-nodeid="55674">
<p data-nodeid="55675"><strong data-nodeid="55902">skywalking.collector.backend_service</strong>：指定 SkyWalking 服务端的通信IP与端口。</p>
</li>
<li data-nodeid="55676">
<p data-nodeid="55677"><strong data-nodeid="55909">skywalking.logging.file_name</strong>：指定 agent 生成的上报日志文件名，日志文件保存 agent 的 logs 目录中。</p>
</li>
</ul>
<h4 data-nodeid="55678">介绍 SkyWalking 常用视图</h4>
<p data-nodeid="55679">当服务启动后，为了演示需要，我利用 PostMan 对 a 接口模拟 10000次 用户访问，看 SkyWalking UI 中产生哪些变化。</p>
<p data-nodeid="101806" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfYWAAmpGAADp5LHFtb4470.png" alt="图片15.png" data-nodeid="101810"></p>
<div data-nodeid="101807"><p style="text-align:center">PostMan 压力测试</p></div>




<p data-nodeid="55682">此时访问<a href="http://192.168.31.10:8080/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="55919">http://192.168.31.10:8080/</a>，打开 SKyWalking UI，默认显示全局的应用性能，具体用途我已在图中标出，其中我认为比较重要的是服务状态指数与高延迟端点列表两项，服务状态指数越接近 1 代表该服务运行状况越好，而高延迟端点列表则将处理延迟高的 API 端点列出，这可能是我们重点排查与优化的对象。</p>
<p data-nodeid="103166" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfY6AGeULAAM7ej7_VNE922.png" alt="图片16.png" data-nodeid="103170"></p>
<div data-nodeid="103167"><p style="text-align:center">全局监控</p></div>



<p data-nodeid="55685">除了全局监控外，SkyWalking 链路追踪的展示也非常友好，点击“拓扑”按钮可以查看访问拓扑图。服务间依赖关系一目了然。</p>
<p data-nodeid="104522" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfZeAIXHIAAFgEZVxafQ379.png" alt="图片17.png" data-nodeid="104526"></p>
<div data-nodeid="104523"><p style="text-align:center">拓扑图</p></div>



<p data-nodeid="55688">除此之外，链路追踪的展示也非常强大，服务间的 API 调用关系与执行时间、调用状态清晰列出，而且因为 SkyWalking 是方法层面上的扩展，会提供更加详细的方法间的调用过程。</p>
<p data-nodeid="105874" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfZ-AJuu5AALJXp72suc036.png" alt="图片18.png" data-nodeid="105878"></p>
<div data-nodeid="105875"><p style="text-align:center">链路追踪图</p></div>



<p data-nodeid="107222" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2C/B4/CioPOWBlfaeAZYPdAAMKjEV-8kM730.png" alt="图片19.png" data-nodeid="107226"></p>
<div data-nodeid="107223"><p style="text-align:center">提供不同维度的视图</p></div>



<p data-nodeid="55693">SkyWalking 基于 Java Agent 对数据库的运行指标也进行收集，点击"database"便进入数据库指标监控。</p>
<p data-nodeid="108566" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/AC/Cgp9HWBlfa-ALznUAAMWPMZaMuM498.png" alt="图片20.png" data-nodeid="108570"></p>
<div data-nodeid="108567"><p style="text-align:center">数据库视图</p></div>



<p data-nodeid="55696">如果你用过 SkyWalking 一定会被它简单的使用方法与强大的功能所折服，在SkyWalking提供了多达几十种不同维度、不同方式的数据展示方案，例如基于服务实例的JVM检测仪表盘就能让我们了解该服务 JVM 的资源分配过程，分析其中潜在的问题。</p>
<p data-nodeid="112592" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2C/B4/CioPOWBlfcCAS2PoAAO4VHJx5Po460.png" alt="图片21.png" data-nodeid="112596"></p>
<div data-nodeid="112593"><p style="text-align:center">服务实例的 JVM 监控</p></div>




<p style="text-align:center">JVM 监控</p>


<p data-nodeid="55699">讲到这，咱们已经完成了 SkyWalking 的安装部署与应用接入，同时也对各种监控图表进行了介绍。因为篇幅有限，只能带着大家对 SkyWalking 进行入门讲解。当然 SKyWalking 也不是全能的，在生产环境下 SkyWalking 还需要额外考虑很多问题，如 SkyWalking 的集群管理、访问权限控制、自监控、风险预警等都要逐步完善，因此很多互联网公司也基于 SkyWalking 做二次开发以满足自身的需求，希望你也能在使用过程中对 SkyWalking 的潜力进行挖掘、了解。</p>
<h3 data-nodeid="115264" class="te-preview-highlight">小结与预告</h3>




<p data-nodeid="55701">本讲咱们学习了三方面内容，首先了解了 APM 与 SkyWalking 的作用；其次讲解了 SkyWalking 的部署过程与接入过程，介绍了 Java Agent 探针技术；最后对 SkyWalking UI 提供的各种图表进行了说明。</p>
<p data-nodeid="55702">在这里我为你准备了一个有趣的讨论题：你的领导希望项目使用 Sleuth+Zipkin 实现链路追踪，而你作为架构师更希望引入 SkyWalking，你有什么办法在不得罪领导的前提下让他改变想法呢？这是所有架构师都要面对的问题，欢迎在评论区一起探讨。</p>
<p data-nodeid="55703">下一讲，咱们开始一个新话题：在微服务（分布式）架构下如何保证数据一致性。</p>

---

### 精选评论

##### **9720：
> 告诉领导加这个不用改代码😇

##### **用户0256：
> 1、两种技术详细且权威的信息，可以让领导信任你2、换位思考，从同一角度出发与领导共鸣，这两种技术是APM的实现3、利用群体或者个体关系去改变领导对该技术的看法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 必须点赞~！这是老司机了，首先要建立共识，换位思考更容易促进彼此理解。

##### *拓：
> skywalking可以和多种日志框架如：logback等配置，在日志中输出 trace-id ,spanid等信息，可以通过日志串起来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

##### **雄：
> sleuth+zipkin 像工作流 flowwork, 有迹可循 。skywalking 根据探针管理。不是很明白顺序

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; skywalking收集数据是基于方法的,利用Java Agent监听线程堆栈. 这个过程却是对外不可见

##### **兴：
> SkyWalking支持像spring cloud sleuth一样通过traceId串联日志的功能吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不支持，实现的机制不一样的。SkyWalking采用Java Agent机制上报，而不是读取日志

##### **亮：
> 两种方案的对比整理出来。以及各自接入需要做的工作也整理出来。给领导说清楚sw的优势就可以啦。

