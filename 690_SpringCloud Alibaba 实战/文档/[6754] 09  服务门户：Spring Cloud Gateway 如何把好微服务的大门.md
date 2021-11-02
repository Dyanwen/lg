<p data-nodeid="1361" class="">上一讲我们学习了 Dubbo 是如何与 Nacos 协同作业。通过对比 RESTful 与 RPC，我们介绍了两种通信方式的区别，再通过实例讲解如何将 Dubbo 与 Nacos 进行整合。但你是否发现无论是基于 OpenFeign 的 RESTful 通信，还是基于 Dubbo 的 RPC 通信，它们都在强调的是微服务间的信息传递，属于微服务架构内部的事情。而对于用户端从外侧访问微服务如何有效管理，微服务又是如何将接口暴露给用户呢？这就需要通过 API 网关实现需求了，本讲咱们就针对 API 网关学习三方面知识：</p>
<ol data-nodeid="1362">
<li data-nodeid="1363">
<p data-nodeid="1364">介绍 API 网关的用途与产品；</p>
</li>
<li data-nodeid="1365">
<p data-nodeid="1366">讲解 Spring Cloud Gateway 的配置技巧；</p>
</li>
<li data-nodeid="1367">
<p data-nodeid="1368">讲解 Gateway执行原理与自定义过滤器（Filter）。</p>
</li>
</ol>
<h3 data-nodeid="1369">API 网关的作用</h3>
<p data-nodeid="1370">如下图所示，对于整个微服务来说如果将每一个微服务的接口直接暴露给用户是错误的做法，这里主要体现出三个问题：</p>
<ul data-nodeid="1371">
<li data-nodeid="1372">
<p data-nodeid="1373">服务将所有 API 接口对外直接暴露给用户端，这本身就是不安全和不可控的，用户可能越权访问不属于它的功能，例如普通的用户去访问管理员的高级功能。</p>
</li>
<li data-nodeid="1374">
<p data-nodeid="1375">后台服务可能采用不同的通信方式，如服务 A 采用 RESTful 通信，服务 B 采用 RPC 通信，不同的接入方式让用户端接入困难。尤其是 App 端接入 RPC 过程更为复杂。</p>
</li>
<li data-nodeid="1376">
<p data-nodeid="1377">在服务访问前很难做到统一的前置处理，如服务访问前需要对用户进行鉴权，这就必须将鉴权代码分散到每个服务模块中，随着服务数量增加代码将难以维护。</p>
</li>
</ul>
<p data-nodeid="1378"><img src="https://s0.lgstatic.com/i/image6/M00/23/99/Cgp9HWBXaFSANx-tAAE1dQdYjME754.png" alt="图片1.png" data-nodeid="1555"></p>
<div data-nodeid="1379"><p style="text-align:center">用户端直接访问微服务</p></div>
<p data-nodeid="1380">为了解决以上问题，API 网关应运而生，加入网关后应用架构变为下图所示。</p>
<p data-nodeid="1697" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/23/ED/CioPOWBX-fWAHNqWAAB1dyZpFjI441.png" alt="微信图片_20210322095840.png" data-nodeid="1703"></p>
<div data-nodeid="1698"><p style="text-align:center">引入 API 网关后的微服务架构</p></div>


<p data-nodeid="1383">当引入 API 网关后，在用户端与微服务之间建立了一道屏障，通过 API 网关为微服务访问提供了统一的访问入口，所有用户端的请求被 API 网关拦截并在此基础上可以实现额外功能，例如：</p>
<ul data-nodeid="1384">
<li data-nodeid="1385">
<p data-nodeid="1386">针对所有请求进行统一鉴权、熔断、限流、日志等前置处理，让微服务专注自己的业务。</p>
</li>
<li data-nodeid="1387">
<p data-nodeid="1388">统一调用风格，通常 API 网关对外提供 RESTful 风格 URL 接口。用户传入请求后，由 API 网关负责转换为后端服务需要的 RESTful、RPC、WebService 等方式，这样便大幅度简化用户的接入难度。</p>
</li>
<li data-nodeid="1389">
<p data-nodeid="1390">更好的安全性，在通过 API 网关鉴权后，可以控制不同角色用户访问后端服务的权利，实现了服务更细粒度的权限控制。</p>
</li>
<li data-nodeid="1391">
<p data-nodeid="1392">API 网关是用户端访问 API 的唯一入口，从用户的角度来说只需关注 API 网关暴露哪些接口，至于后端服务的处理细节，用户是不需要知道的。从这方面讲，微服务架构通过引入 API 网关，将用户端与微服务的具体实现进行了解耦。</p>
</li>
</ul>
<p data-nodeid="1393">以上便是 API 网关的作用，那 API 网关有哪些产品呢？</p>
<h4 data-nodeid="1394">API 网关主流产品</h4>
<p data-nodeid="1395">API 网关是微服务架构中必要的组件，具体的实现产品在软件市场上层出不穷，下面我列举三款在国内外主流的开源产品。</p>
<h4 data-nodeid="1396">OpenResty</h4>
<p data-nodeid="1397">OpenResty 是一个强大的 Web 应用服务器，Web 开发人员可以使用 Lua 脚本语言调动 Nginx 支持的各种 C 以及 Lua 模块,更主要的是在性能方面，OpenResty可以快速构造出足以胜任 10K 以上并发连接响应的超高性能 Web 应用系统。360、UPYUN、阿里云、新浪、腾讯网、去哪儿网、酷狗音乐等都是 OpenResty 的深度用户。</p>
<p data-nodeid="1398">OpenResty 因为性能强大在微服务架构早期深得架构师的喜爱。但 OpenResty 是一款独立的产品，与主流的注册中心存在一定兼容问题，需要架构师独立实现其服务注册、发现的功能。后来 Spring Cloud 官方极力推崇自家封装的 Zuul 或者 Spring Cloud Gateway，渐渐 OpenResty 便淡出了我们的视野。但不能否认，OpenResty 仍是一款优秀的 API 网关产品。</p>
<h4 data-nodeid="1399">Spring Cloud Zuul</h4>
<p data-nodeid="1400">Zuul 是 Netflix 开源的微服务网关，它的主要职责是对用户请求进行路由转发与过滤。早期Spring Cloud 与 Netfilx 合作，使用 Zuul 作为微服务架构的首选网关产品。Zuul 是基于 J2EE Servlet 实现路由转发，网络通信采用同步方式，使用简单部署方便。经过 Spring Cloud 对 Zuul 的封装，Spring Cloud Zuul 应运而生。Spring Cloud Zuul 在原有 Zuul 的基础上，增加对注册中心的支持，同时在基于 Spring Boot Starter 机制基础上，可以在极短的时间内完成 API 网关的开发部署任务。</p>
<p data-nodeid="1401"><img src="https://s0.lgstatic.com/i/image6/M00/23/96/CioPOWBXaHKAXbZsAAOHtjWJwBo401.png" alt="图片3.png" data-nodeid="1575"></p>
<div data-nodeid="1402"><p style="text-align:center">Zuul 基于 Servlet 的请求响应处理过程</p></div>
<p data-nodeid="1403">但好景不长，后来 Netflix 内部产生分歧，Netflix 官方宣布 Zuul 停止维护，这让 Spring 机构也必须转型。于是 Spring Cloud 团队决定开发自己的第二代 API 网关产品：Spring Cloud Gateway。</p>
<h4 data-nodeid="1404">Spring Cloud Gateway</h4>
<p data-nodeid="1405">与 Zuul 是“别人家的孩子”不同，Spring Cloud Gateway 是 Spring 自己开发的新一代 API 网关产品。它基于 NIO 异步处理，摒弃了 Zuul 基于 Servlet 同步通信的设计，因此拥有更好的性能。同时，Spring Cloud Gateway 对配置进行了进一步精简，比 Zuul 更加简单实用。</p>
<p data-nodeid="1406">以下是 Spring Cloud Gateway 的关键特征：</p>
<ul data-nodeid="1407">
<li data-nodeid="1408">
<p data-nodeid="1409">基于 JDK 8+ 开发；</p>
</li>
<li data-nodeid="1410">
<p data-nodeid="1411">基于 Spring Framework 5 + Project Reactor + Spring Boot 2.0 构建；</p>
</li>
<li data-nodeid="1412">
<p data-nodeid="1413">支持动态路由，能够匹配任何请求属性上的路由；</p>
</li>
<li data-nodeid="1414">
<p data-nodeid="1415">支持基于 HTTP 请求的路由匹配（Path、Method、Header、Host 等）；</p>
</li>
<li data-nodeid="1416">
<p data-nodeid="1417">过滤器可以修改 HTTP 请求和 HTTP 响应（增加/修改 Header、增加/修改请求参数、改写请求 Path 等等）；</p>
</li>
<li data-nodeid="1418">
<p data-nodeid="1419">...</p>
</li>
</ul>
<p data-nodeid="1420">当下 Spring Cloud Gateway 已然是 Spring Cloud 体系上API 网关标准组件。Spring Cloud Gateway 十分优秀，Spring Cloud Alibaba 也默认选用该组件作为网关产品，下面我们就通过实例讲解 Spring Cloud Gateway 的使用办法。</p>
<h3 data-nodeid="1421">Spring Cloud Gateway的配置技巧</h3>
<h4 data-nodeid="1422">Spring Cloud Gateway使用入门</h4>
<p data-nodeid="1423">示例说明：</p>
<p data-nodeid="1424">假设“service-a”微服务提供了三个 RESTful 接口。</p>
<p data-nodeid="1425"><img src="https://s0.lgstatic.com/i/image6/M00/23/99/Cgp9HWBXaICAUVGbAACopAyWXv0370.png" alt="图片4.png" data-nodeid="1593"></p>
<p data-nodeid="1426">假设 “service-b” 微服务提供了三个 RESTful 接口。</p>
<p data-nodeid="1427"><img src="https://s0.lgstatic.com/i/image6/M00/23/96/CioPOWBXaI2APUbjAACpocL8xRE677.png" alt="图片5.png" data-nodeid="1597"></p>
<p data-nodeid="1428">如何通过部署 Spring Cloud Gateway 实现 API 路由功能来屏蔽后端细节呢？</p>
<p data-nodeid="1429">第一步，利用 Spring Initializr 向导创建 Gateway 工程，确保 pom.xml 引入以下依赖：</p>
<pre class="lang-xml" data-nodeid="1430"><code data-language="xml"><span class="hljs-comment">&lt;!-- Nacos客户端 --&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- Spring Cloud Gateway Starter --&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-gateway<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- 对外提供Gateway应用监控指标 --&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-actuator<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1431">第二步，在 application.yml 增加如下配置。</p>
<pre class="lang-yaml" data-nodeid="1432"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">gateway</span> <span class="hljs-comment">#配置微服务id</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">nacos:</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.101</span><span class="hljs-string">:8848</span> <span class="hljs-comment">#nacos通信地址</span>
        <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
        <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
    <span class="hljs-attr">gateway:</span> <span class="hljs-comment">#让gateway通过nacos实现自动路由转发</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">locator:</span>
          <span class="hljs-attr">enabled:</span> <span class="hljs-literal">true</span> <span class="hljs-comment">#locator.enabled是自动根据URL规则实现路由转发</span>
<span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">80</span> <span class="hljs-comment">#服务端口号</span>
<span class="hljs-attr">management:</span> 
  <span class="hljs-attr">endpoints:</span>
    <span class="hljs-attr">web:</span>
      <span class="hljs-attr">exposure:</span>
        <span class="hljs-attr">include:</span> <span class="hljs-string">'*'</span> <span class="hljs-comment">#对外暴露actuator所有监控指标，便于监控系统收集跟踪</span>
</code></pre>
<p data-nodeid="1433">在上面的配置中最重要的一句是：</p>
<pre class="lang-java" data-nodeid="1434"><code data-language="java">spring.cloud.gateway.discovery.locator.enabled=<span class="hljs-keyword">true</span>
</code></pre>
<p data-nodeid="1435">这是一个自动项，允许 Gateway 自动实现后端微服务路由转发， Gateway 工程启动后，在浏览器地址栏按下面格式访问后端服务。</p>
<pre class="lang-java" data-nodeid="1436"><code data-language="java">http:<span class="hljs-comment">//网关IP:端口/微服务id/URI</span>
</code></pre>
<p data-nodeid="1437">例如，网关 IP 为：192.168.31.103，我们需要通过网关执行 service-a 的 list 方法，具体写法为：</p>
<pre class="lang-java" data-nodeid="1438"><code data-language="java">http:<span class="hljs-comment">//192.168.31.103:80/service-a/list</span>
</code></pre>
<p data-nodeid="1439">访问后 Gateway 按下图流程进行请求路由转发。</p>
<p data-nodeid="1440"><img src="https://s0.lgstatic.com/i/image6/M00/23/99/Cgp9HWBXaViAFTHsAAHX8DX4Vz8328.png" alt="图片6.png" data-nodeid="1607"></p>
<div data-nodeid="1441"><p style="text-align:center">基于网关自动路由处理流程</p></div>
<p data-nodeid="1442">咱们来梳理下路由转发流程：</p>
<ol data-nodeid="1443">
<li data-nodeid="1444">
<p data-nodeid="1445">Gateway、service-a 这些都是微服务实例，在启动时向 Nacos 注册登记；</p>
</li>
<li data-nodeid="1446">
<p data-nodeid="1447">用户端向 Gateway 发起请求，请求地址<a href="http://192.168.31.103:80/service-a/list" data-nodeid="1613">http://192.168.31.103:80/service-a/list</a>；</p>
</li>
<li data-nodeid="1448">
<p data-nodeid="1449">Gateway 网关实例收到请求，解析其中第二部分 service-a，即微服务 Id，第三部分 URI 为“/list”。之后向 Nacos 查询 service-a 可用实例列表；</p>
</li>
<li data-nodeid="1450">
<p data-nodeid="1451">Nacos 返回 120 与 121 两个可用微服务实例信息；</p>
</li>
<li data-nodeid="1452">
<p data-nodeid="1453">Spring Cloud Gateway 内置 Ribbon，根据默认轮询策略将请求转发至 120 实例，转发的完整 URL 将附加用户的 URI，即<a href="http://192.168.31.120:80/list" data-nodeid="1620">http://192.168.31.120:80/list</a>；</p>
</li>
<li data-nodeid="1454">
<p data-nodeid="1455">120 实例处理后返回 JSON 响应数据给 Gateway；</p>
</li>
<li data-nodeid="1456">
<p data-nodeid="1457">Gateway 返回给用户端，完成一次完整的请求路由转发过程。</p>
</li>
</ol>
<p data-nodeid="1458">讲到这，我们已理解了 Spring Cloud Gateway 的执行过程。但是真实项目中，存在着各种特殊的路由转发规则，而非自动路由能简单解决的，在 Spring Cloud Gateway 项目中内置了强大的“谓词”系统，可以满足企业应用中的各种转发规则要求，下一小节咱们就来介绍常见的谓词用法。</p>
<h4 data-nodeid="1459">谓词（Predicate）与过滤器（Filter）</h4>
<p data-nodeid="1460">在讲解前需要引入 Gateway 网关三个关键名词：路由（Route）、谓词（Predicate）、过滤器（Filter）。</p>
<p data-nodeid="1461">路由（Route）是指一个完整的网关地址映射与处理过程。一个完整的路由包含两部分配置：谓词（Predicate）与过滤器（Filter）。前端应用发来的请求要被转发到哪个微服务上，是由谓词决定的；而转发过程中请求、响应数据被网关如何加工处理是由过滤器决定的。说起来有些晦涩，我们通过实例进行讲解就容易理解了。</p>
<p data-nodeid="1462"><strong data-nodeid="1631">谓词（Predicate）</strong></p>
<p data-nodeid="1463">这里我们给出一个实例，将原有 Gateway 工程的 application.yml 文件修改为下面的设置：</p>
<pre class="lang-yaml" data-nodeid="1464"><code data-language="yaml"><span class="hljs-attr">spring:</span>
&nbsp; <span class="hljs-attr">application:</span>
&nbsp; &nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">gateway</span>
&nbsp; <span class="hljs-attr">cloud:</span>
&nbsp; &nbsp; <span class="hljs-attr">nacos:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">discovery:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:8848</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
&nbsp; &nbsp; <span class="hljs-string">gateway:</span>&nbsp;
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">discovery:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">locator:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">enabled:</span> <span class="hljs-literal">false</span> <span class="hljs-comment">#不再需要Gateway路由转发</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-string">routes:</span>&nbsp; <span class="hljs-comment">#路由规则配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#第一个路由配置，service-a路由规则</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">id:</span> <span class="hljs-string">service_a_route</span> <span class="hljs-comment">#路由唯一标识</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#lb开头代表基于gateway的负载均衡策略选择实例</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">uri:</span> <span class="hljs-string">lb://service-a</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#谓词配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">predicates:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#Path路径谓词，代表用户端URI如果以/a开头便会转发到service-a实例</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">Path=/a/**</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#After生效时间谓词，2020年10月15日后该路由才能在网关对外暴露</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">After=2020-10-05T00:00:00.000+08:00[Asia/Shanghai]</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#谓词配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">filters:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#忽略掉第一层前缀进行转发</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">StripPrefix=1</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#为响应头附加X-Response=Blue</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">AddResponseHeader=X-Response,Blue</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">#第二个路由配置，service-b路由规则</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">id:</span> <span class="hljs-string">service_b_route</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">uri:</span> <span class="hljs-string">lb://service-b</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">predicates:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">Path=/b/**</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">filters:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-string">StripPrefix=1</span>
<span class="hljs-attr">server:</span>
&nbsp; <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
<span class="hljs-attr">management:</span>
&nbsp; <span class="hljs-attr">endpoints:</span>
&nbsp; &nbsp; <span class="hljs-attr">web:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">exposure:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">include:</span> <span class="hljs-string">'*'</span>
</code></pre>
<p data-nodeid="1465">我来翻译下上面的配置：<br>
在 2020 年 10 月 15 日后，当用户端发来/a/...开头的请求时，Spring Cloud Gateway 会自动获取 service-a 可用实例，默认采用轮询方式将URI附加至实例地址后，形成新地址，service-a处理后 Gateway 网关自动在响应头附加 X-Response=Blue。至于第二个 service_b_route，比较简单，只说明当用户访问/b开头 URL 时，转发到 service-b 可用实例。</p>
<p data-nodeid="1466">完整的路由配置格式固定如下：</p>
<pre class="lang-yaml" data-nodeid="1467"><code data-language="yaml"><span class="hljs-attr">spring:</span>
&nbsp; &nbsp; <span class="hljs-string">gateway:</span>&nbsp;
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">discovery:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">locator:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">enabled:</span> <span class="hljs-literal">false</span>&nbsp; <span class="hljs-comment">#不再需要Gateway路由转发</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-string">routes:</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">id:</span> <span class="hljs-string">xxx</span> <span class="hljs-comment">#路由规则id</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">uri:</span> <span class="hljs-string">lb://微服务id</span>&nbsp; <span class="hljs-comment">#路由转发至哪个微服务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">predicates:</span>&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">//具体的谓词</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-attr">filters:</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">//具体的过滤器</span>
</code></pre>
<p data-nodeid="1468">其中 predicates 是重点，说明路由生效条件，在这里我将常见的谓词使用形式列出来。</p>
<ul data-nodeid="1469">
<li data-nodeid="1470">
<p data-nodeid="1471">After 代表在指定时点后路由规则生效。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1472"><code data-language="java">predicates:
&nbsp; &nbsp; - After=<span class="hljs-number">2020</span>-<span class="hljs-number">10</span>-<span class="hljs-number">04</span>T00:<span class="hljs-number">00</span>:<span class="hljs-number">00.000</span>+<span class="hljs-number">08</span>:<span class="hljs-number">00</span>
</code></pre>
<ul data-nodeid="1473">
<li data-nodeid="1474">
<p data-nodeid="1475">Before 代表在指定时点前路由规则生效。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1476"><code data-language="java">predicates:
&nbsp; &nbsp; - Before=<span class="hljs-number">2020</span>-<span class="hljs-number">01</span>-<span class="hljs-number">20</span>T17:<span class="hljs-number">42</span>:<span class="hljs-number">47.789</span>-<span class="hljs-number">07</span>:<span class="hljs-number">00</span>[America/Denver]
</code></pre>
<ul data-nodeid="1477">
<li data-nodeid="1478">
<p data-nodeid="1479">Path 代表 URI 符合映射规则时生效。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1480"><code data-language="java">predicates:
&nbsp; &nbsp; - Path=/b<span class="hljs-comment">/**
</span></code></pre>
<ul data-nodeid="1481">
<li data-nodeid="1482">
<p data-nodeid="1483">Header 代表包含指定请求头时生效。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1484"><code data-language="java">predicates:
&nbsp; &nbsp; - Header=X-Request-Id, \d+
</code></pre>
<p data-nodeid="1485">这里额外解释下，如果请求具有名为 X-Request-Id 的 Header，其值与\d+正则表达式匹配（具有一个或多个数字的值），则该路由匹配。</p>
<ul data-nodeid="1486">
<li data-nodeid="1487">
<p data-nodeid="1488">Method 代表要求 HTTP 方法符合规定时生效。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1489"><code data-language="java">predicates:
&nbsp; &nbsp; - Method=GET
</code></pre>
<p data-nodeid="1490">谓词是 Gateway 网关中最灵活的部分，刚才列举的是最常用的谓词，还有很多谓词是在文中没有提到，如果你对这部分感兴趣可以翻阅<a href="https://spring.io/projects/spring-cloud-gateway%E5%AE%98%E6%96%B9%E6%96%87%E6%A1%A3%E8%BF%9B%E8%A1%8C%E5%AD%A6%E4%B9%A0" data-nodeid="1653">https://spring.io/projects/spring-cloud-gateway</a>进行学习。</p>
<p data-nodeid="1491"><strong data-nodeid="1658">过滤器（Filter）</strong></p>
<p data-nodeid="1492">过滤器（Filter）可以对请求或响应的数据进行额外处理，这里我们列出三个最常用的内置过滤器进行说明。</p>
<ul data-nodeid="1493">
<li data-nodeid="1494">
<p data-nodeid="1495">AddRequestParameter 是对所有匹配的请求添加一个查询参数。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1496"><code data-language="java">filters:
- AddRequestParameter=foo,bar #在请求参数中追加foo=bar
</code></pre>
<ul data-nodeid="1497">
<li data-nodeid="1498">
<p data-nodeid="1499">AddResponseHeader 会对所有匹配的请求，在返回结果给客户端之前，在 Header 中添加响应的数据。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1500"><code data-language="java">#在Response中添加Header头，key=X-Response-Foo，Value=Bar。
filters:
- AddResponseHeader=X-Response,Blue
</code></pre>
<ul data-nodeid="1501">
<li data-nodeid="1502">
<p data-nodeid="1503">Retry 为重试过滤器，当后端服务不可用时，网关会根据配置参数来发起重试请求。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1504"><code data-language="java">filters:
#涉及过滤器参数时，采用name-args的完整写法
- name: Retry #name是内置的过滤器名
&nbsp; args: #参数部分使用args说明
&nbsp; &nbsp; retries: 3
&nbsp; &nbsp; status: 503
</code></pre>
<p data-nodeid="1505">以上片段含义为，当后端服务返回 503 状态码的响应后，Retry 过滤器会重新发起请求，最多重试 3 次。</p>
<p data-nodeid="1506">以上是三种最常用的内置过滤器的使用案例，因为 Spring Cloud 内置过滤器将近 30 个，这里咱们就不一一列举，有兴趣的同学可以查询官方资料。</p>
<p data-nodeid="1507"><a href="https://docs.spring.io/spring-cloud-gateway/docs/2.2.6.RELEASE/reference/html/#gatewayfilter-factories" data-nodeid="1667">https://docs.spring.io/spring-cloud-gateway/docs/2.2.6.RELEASE/reference/html/#gatewayfilter-factories</a></p>
<h3 data-nodeid="1508">Gateway 的执行原理与自定义过滤器</h3>
<h4 data-nodeid="1509">Spring Cloud Gateway 的执行原理</h4>
<p data-nodeid="1510">在初步掌握 Spring Cloud Gateway 的配置技巧与谓词用法后，我们来关注 Gateway 底层的实现细节。</p>
<p data-nodeid="1511">下图是 Spring Cloud Gateway 的执行流程。</p>
<p data-nodeid="1512"><img src="https://s0.lgstatic.com/i/image6/M00/23/96/CioPOWBXaLWAYmNDAADePiHh5QM390.png" alt="图片7.png" data-nodeid="1674"></p>
<p data-nodeid="1513">按执行顺序可以拆解以下几步：</p>
<ol data-nodeid="1514">
<li data-nodeid="1515">
<p data-nodeid="1516">Spring Cloud Gateway 启动时基于 Netty Server 监听指定的端口（该端口可以通过 server.port 属性自定义）。当前端应用发送一个请求到网关时，进入 Gateway Handler Mapping 处理过程，网关会根据当前 Gateway 所配置的谓词（Predicate）来决定是由哪个微服务进行处理。</p>
</li>
<li data-nodeid="1517">
<p data-nodeid="1518">确定微服务后，请求向后进入 Gateway Web Handler 处理过程，该过程中 Gateway 根据过滤器（Filters）配置，将请求按前后顺序依次交给 Filter 过滤链进行前置（Pre）处理，前置处理通常是对请求进行前置检查，例如：判断是否包含某个指定请求头、检查请求的 IP 来源是否合法、请求包含的参数是否正确等。</p>
</li>
<li data-nodeid="1519">
<p data-nodeid="1520">当过滤链前置（Pre）处理完毕后，请求会被 Gateway 转发到真正的微服务实例进行处理，微服务处理后会返回响应数据，这些响应数据会按原路径返回被 Gateway 配置的过滤链进行后置处理（Post），后置处理通常是对响应进行额外处理，例如：将处理过程写入日志、为响应附加额外的响应头或者流量监控等。</p>
</li>
</ol>
<p data-nodeid="1521">可以看到，在整个处理过程中谓词（Predicate）与过滤器（Filter）起到了重要作用，谓词决定了路径的匹配规则，让 Gateway 确定应用哪个微服务，而 Filter 则是对请求或响应作出实质的前置、后置处理。</p>
<p data-nodeid="1522">在项目中功能场景多种多样，像日常的用户身份鉴权、日志记录、黑白名单、反爬虫等基础功能都可以通过自定义 Filter 为 Gateway 进行功能扩展，下面我们通过“计时过滤器”为例，讲解如何为 Gateway 绑定自定义全局过滤器。</p>
<h4 data-nodeid="1523">自定义全局过滤器</h4>
<p data-nodeid="1524">在 Spring Cloud Gateway 中，自定义过滤器分为两种，全局过滤器与局部过滤器。两者唯一的区别是：全局过滤器默认应用在所有路由（Route）上，而局部过滤器可以为指定的路由绑定。下面咱们通过“计时过滤器”这个案例讲解全局过滤器的配置。所谓计时过滤器是指任何从网关访问的请求，都要在日志中记录下从请求进入到响应退出的执行时间，通过这个时间运维人员便可以收集并分析哪些功能进行了慢处理，以此为依据进行进一步优化。下面是计时过滤器的代码，重要的部分我通过注释进行了说明。</p>
<pre class="lang-java" data-nodeid="1525"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.gateway.filter;
<span class="hljs-keyword">import</span> org.slf4j.Logger;
<span class="hljs-keyword">import</span> org.slf4j.LoggerFactory;
<span class="hljs-keyword">import</span> org.springframework.cloud.gateway.filter.GatewayFilterChain;
<span class="hljs-keyword">import</span> org.springframework.cloud.gateway.filter.GlobalFilter;
<span class="hljs-keyword">import</span> org.springframework.core.Ordered;
<span class="hljs-keyword">import</span> org.springframework.stereotype.Component;
<span class="hljs-keyword">import</span> org.springframework.web.server.ServerWebExchange;
<span class="hljs-keyword">import</span> reactor.core.publisher.Mono;
<span class="hljs-meta">@Component</span> <span class="hljs-comment">//自动实例化并被Spring IOC容器管理</span>
<span class="hljs-comment">//全局过滤器必须实现两个接口：GlobalFilter、Ordered</span>
<span class="hljs-comment">//GlobalFilter是全局过滤器接口，实现类要实现filter()方法进行功能扩展</span>
<span class="hljs-comment">//Ordered接口用于排序，通过实现getOrder()方法返回整数代表执行当前过滤器的前后顺序</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ElapsedFilter</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">GlobalFilter</span>, <span class="hljs-title">Ordered</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">//基于slf4j.Logger实现日志输出</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Logger logger = LoggerFactory.getLogger(ElapsedFilter.class);
&nbsp; &nbsp; //起始时间属性名
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String ELAPSED_TIME_BEGIN = "elapsedTimeBegin";
&nbsp; &nbsp; /**
&nbsp; &nbsp; &nbsp;* 实现filter()方法记录处理时间
&nbsp; &nbsp; &nbsp;* <span class="hljs-meta">@param</span> exchange 用于获取与当前请求、响应相关的数据，以及设置过滤器间传递的上下文数据
&nbsp; &nbsp; &nbsp;* <span class="hljs-meta">@param</span> chain Gateway过滤器链对象
&nbsp; &nbsp; &nbsp;* <span class="hljs-meta">@return</span> Mono对应一个异步任务，因为Gateway是基于Netty Server异步处理的,Mono对就代表异步处理完毕的情况。
&nbsp; &nbsp; &nbsp;*/
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Mono&lt;Void&gt; <span class="hljs-title">filter</span><span class="hljs-params">(ServerWebExchange exchange, GatewayFilterChain chain)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//Pre前置处理部分</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//在请求到达时，往ServerWebExchange上下文环境中放入了一个属性elapsedTimeBegin，保存请求执行前的时间戳</span>
&nbsp; &nbsp; &nbsp; &nbsp; exchange.getAttributes().put(ELAPSED_TIME_BEGIN, System.currentTimeMillis());

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//chain.filter(exchange).then()对应Post后置处理部分</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//当响应产生后，记录结束与elapsedTimeBegin起始时间比对，获取RESTful API的实际执行时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> chain.filter(exchange).then(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Mono.fromRunnable(() -&gt; { <span class="hljs-comment">//当前过滤器得到响应时，计算并打印时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Long startTime = exchange.getAttribute(ELAPSED_TIME_BEGIN);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (startTime != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.info(exchange.getRequest().getRemoteAddress() <span class="hljs-comment">//远程访问的用户地址</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + <span class="hljs-string">" | "</span> +&nbsp; exchange.getRequest().getPath()&nbsp; <span class="hljs-comment">//Gateway URI</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + <span class="hljs-string">" | cost "</span> + (System.currentTimeMillis() - startTime) + <span class="hljs-string">"ms"</span>); <span class="hljs-comment">//处理时间</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; })
&nbsp; &nbsp; &nbsp; &nbsp; );
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">//设置为最高优先级，最先执行ElapsedFilter过滤器</span>
&nbsp; &nbsp; <span class="hljs-comment">//return Ordered.LOWEST_PRECEDENCE; 代表设置为最低优先级</span>
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getOrder</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Ordered.HIGHEST_PRECEDENCE;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1526">运行后通过 Gateway 访问任意微服务便会输出日志：</p>
<pre class="lang-java" data-nodeid="1527"><code data-language="java"><span class="hljs-number">2021</span>-<span class="hljs-number">01</span>-<span class="hljs-number">10</span> <span class="hljs-number">12</span>:<span class="hljs-number">36</span>:<span class="hljs-number">01.765</span>&nbsp; INFO <span class="hljs-number">14052</span> --- [ctor-http-nio-<span class="hljs-number">4</span>] com.lagou.gateway.filter.ElapsedFilter&nbsp; &nbsp;: /<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">0</span>:<span class="hljs-number">1</span>:<span class="hljs-number">57873</span> | /test-service/test | cost <span class="hljs-number">821</span>ms
</code></pre>
<p data-nodeid="1528">日志包含四部分：</p>
<ul data-nodeid="1529">
<li data-nodeid="1530">
<p data-nodeid="1531">日志的基础信息包括时间、日志级别、线程、产生的类与方法等。</p>
</li>
<li data-nodeid="1532">
<p data-nodeid="1533">/0:0:0:0:0:0:0:1:57873 代表访问者的远程 IP 端口等信息。</p>
</li>
<li data-nodeid="1534">
<p data-nodeid="1535">/test-service/test 是通过 Gateway 访问的完整 URI，第一部分是服务名，第二部分是 RESTful 接口。</p>
</li>
<li data-nodeid="1536">
<p data-nodeid="1537">cost 821ms 是具体的执行时间。</p>
</li>
</ul>
<p data-nodeid="1538">以上就是全局过滤器的开发方法，至于局部过滤器的配置方法与全局过滤器极为相似，有兴趣的同学通过下面的官方文档了解更详细的内容。</p>
<p data-nodeid="1539"><a href="https://docs.spring.io/spring-cloud-gateway/docs/2.2.7.BUILD-SNAPSHOT/reference/html/%E4%BA%86%E8%A7%A3%E6%9B%B4%E5%A4%9A%E7%BB%86%E8%8A%82" data-nodeid="1692">https://docs.spring.io/spring-cloud-gateway/docs/2.2.7.BUILD-SNAPSHOT/reference/html/</a></p>
<h3 data-nodeid="1540">小结与预告</h3>
<p data-nodeid="1541">这一讲咱们学了三方面内容，首先讲解了什么是 API 网关，API 网关是负责微服务请求统一路由转发的组件，用户端所有的请求都要经过 API 网关路由、加工、过滤后送达给后端微服务。其次，讲解了 Spring Cloud Gateway 网关的部署方式，了解到谓词（Predicate）与过滤器（Filter）的作用。最后，咱们通过实例介绍了 Spring Cloud Gateway 的执行原理并实现了计时功能的全局过滤器。</p>
<p data-nodeid="1542">这里给你出一道思考题：结合你当前的项目思考下，API 网关除了路由还能额外实现哪些功能呢？</p>
<p data-nodeid="1543" class="">下一讲我们将开始一个新篇章，介绍在微服务环境下如何通过服务降级、熔断等机制保护我们的微服务架构，避免雪崩效应的产生。</p>

---

### 精选评论

##### *诚：
> 老师，你的地址失效了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://github.com/qiyisoft/sca
这里应该可以

##### *西：
> 网关中自定义过滤器，加上@Component注解，被ioc容器管理后，就能自动生效？ 不需要其他配置？ 这个生效原理能解释下？谢谢。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个就是基于Spring Boot的按需注入，你可以开启debug=true，看一下启动的组件加在日志就全明白啦

##### **聪：
> Spring cloud gateway如何实现dubbo 路由

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 抱歉，据我所知目前Gateway对Dubbo是支持的。但需要少许改造，你可以参考下这个帖子
百度搜索：Dubbo想要个网关怎么办？

##### **尧：
> 考虑不同域名访问的问题应该有跨域，考虑服务不可用的情况应该有容错机制

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 跨域目前只会出现在浏览器的安全策略中，微服务间RESTful通信不存在跨域问题的。

##### **飞：
> 请问有代码地址吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 兄弟，我在整理4月6日统一上传到github

##### **1406：
> 老师请问，gateway有无预估的并发量支持数？线上环境应当如何评估？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; gateway估算要有历史数据,那只能先模拟一个目标进行推算.
举个例子: 某电商有1000W会员, 按日常订单200000笔.
我们现在第一次搞大促,要按下面步骤进行测算:
1.需要产品/项目经理预估一个基准性能倍率,比如:3倍
2. 根据产品或者相关从业者经验,大促时时单天订单量集中爆发,可能会有平时日订单10倍
3. 根据消费者日常作息,晚上8~12点是订单高峰期,会产生70%的订单量,这时你的网关要能抗住下面这个公式的要求
(200000 * 10 * 0.7)  / ( 4 * 60 * 60)  * 3 ≈ 291.6 . 
4. 在压测时网关要提供超过300才能达到性能要求

##### **龙：
> 老师你url地址访问404，我发现还要加上网关的微服务id才能访问到，为什么？http://网关IP:端口/微服务id/URI这是你给的，正确需要这样http://网关IP:端口/网关微服务id/具体服务的微服务id/URI

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 同学，你是不是设置了ContextPath，如果设置了那就必须按你的写法。我这里的Contextpath都是默认的 /

##### **华：
> 如果服务A采用 RESTful 通信，服务B采用 RPC 通信。那服务A与服务B之间怎么互相调用？A调B时要用B的RPC通信；B调A时要用A的RESTful通信，这样吗？？就是要自己支持服务提供方的通信方式？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你这种一边RPC一边REST，那就加中间的适配器Adapter吧。要么都转为RPC、要么都REST。

##### **鹏：
> github的地址在哪儿啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://github.com/qiyisoft/sca

##### *诚：
> 老师，麻烦问一下，服务内部相互调用接口，这块的用户校验怎么做，是每个服务都写拦截器吗，还是全部走网关

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 服务内部互相调用不用走网关,一般做拦截器即可.或者写基于注解的AOP切面也是好的选择.

##### *诚：
> 按照老师配置，访问以后是404

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 检查URL,请注意大小写

##### **国：
> 设置了路由后，访问http://localhost/a/list 404，找不到。什么问题？spring:  application:    name: gateway #配置微服务id  cloud:    nacos:      discovery:        server-addr: localhost:8848 #nacos通信地址        username: nacos        password: nacos    gateway: #让gateway通过nacos实现自动路由转发      discovery:        locator:          enabled: false #locator.enabled是自动根据URL规则实现路由转发      routes:         - id: service_a_route #路由唯一标识           uri: lb://demo-service           predicates:            - Path=/a/**#            - After=2020-10-05T00:00:00.000+08:00[Asia/Shanghai]           filters:            - StripPrefix=1            - AddResponseHeader=X-Response,Blue         - id: service_b_route           uri: lb://service-b           predicates:            - Path=/b/**           filters:            - StripPrefix=1

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 同学，https://www.cnblogs.com/xybaby/p/10124083.html
这是RAFT的算法实现细节，看完这篇文章应该可以解答对你选举算法的疑惑。

##### *博：
> 那这样gateway每次需要调整路由策略的时候不是都要重启吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果是放在配置文件中，调整后确实需要重启，还有一种做法就是自定义谓词策略，基于数据库或者其他源动态配置

##### **伟：
> Spring cloud项目必须用api网关吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，网关是API与微服务架构内部的切口，通过网关屏蔽后端复杂度

##### *风：
> 老师，git地址有吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 大家等待已久的源码已整理，有需要的同学可以登录下载哦
链接：https://pan.baidu.com/s/16avfOynyMNljeCVpjl8IXQ 
提取码：307t

##### orange：
> 如果采用Openfeign进行服务间的调用的话会不会经过Gateway？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，OpenFeign是强调服务间通信，是微服务架构内部的事情。绝大多数情况下不需要再走网关。

##### *刚：
> 谓词还可以动态化设置吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里需要自己扩展一个谓词工厂。

