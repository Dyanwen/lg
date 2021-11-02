<p data-nodeid="1141" class="te-preview-highlight">上一节我介绍了什么是 OpenFeign 通信组件，讲解了如何基于 OpenFeign 实现微服务间的高可用通信。本文我们将继续探讨微服务通信话题，了解阿里巴巴自家的 RPC 框架 Dubbo 是如何与 Spring Cloud Alibaba 协同作业的。在本文我们将介绍以下三方面内容：</p>
<ul data-nodeid="1413">
<li data-nodeid="1414">
<p data-nodeid="1415">对比 RESTful 与 RPC；</p>
</li>
<li data-nodeid="1416">
<p data-nodeid="1417" class="">介绍 Dubbo 的特点；</p>
</li>
<li data-nodeid="1418">
<p data-nodeid="1419">讲解 Dubbo 与 Spring Cloud Alibaba 的整合过程。</p>
</li>
</ul>

<h3 data-nodeid="1149">RESTful 与 RPC 的区别</h3>
<p data-nodeid="1150">在微服务定义中提道，每个小服务运行在自己的进程中，并以<strong data-nodeid="1273">轻量级的机制</strong>进行通信。这里并没有明确给出具体的通信方式，只是要求以轻量级的机制进行通信，虽然作者推荐使用 RESTful 作为首选方案，但微服务间通信本身还有另一个轻量级的选择：以 Dubbo 为代表的 RPC通信方式。</p>
<p data-nodeid="1151">那什么是 RPC 呢？RPC 是远程过程调用（Remote Procedure Call）的缩写形式，RPC 与 RESTful 最大的不同是，RPC 采用<strong data-nodeid="1279">客户端（Client) - 服务端（Server）</strong> 的架构方式实现跨进程通信，实现的通信协议也没有统一的标准，具体实现依托于研发厂商的设计。</p>
<p data-nodeid="1152"><img src="https://s0.lgstatic.com/i/image6/M01/1F/21/Cgp9HWBRmtGAJrWGAAS5A1X6ztc358.png" alt="图片1.png" data-nodeid="1282"></p>
<div data-nodeid="1153"><p style="text-align:center">RPC 基于 C/S 架构实现跨进程通信</p></div>
<p data-nodeid="1154">目前开源市场上 RPC 框架有很多，例如 GoogleRPC、Alibaba Dubbo、Facebook&nbsp;Thrift，每一种产品都有自己的设计与实现方案。</p>
<p data-nodeid="1155">那 RESTful 与 RPC 孰优孰劣呢？我们通过一个表格进行说明：</p>
<p data-nodeid="1156"><img src="https://s0.lgstatic.com/i/image6/M01/1F/22/Cgp9HWBRmt2APWu8AAH9-fU6l5c006.png" alt="图片2.png" data-nodeid="1287"></p>
<p data-nodeid="1157">可以发现，RESTful 通信更适合调用延时不敏感、短连接的场景。而 RPC 则拥有更好的性能，适用于长连接、低延时系统。两者本质是互补的，并不存在孰优孰劣。在微服务架构场景下，因为大多数服务都是轻量级的，同时 90%的任务通过短连接就能实现，因此马丁福勒更推荐使用 RESTful 通信。这只是因为应用场景所决定的，并不代表 RPC 比 RESTful 落后。</p>
<p data-nodeid="1158">在了解 RPC 方式后，我们来聊一聊 RPC领域最具代表性的国产开源框架 Apache Dubbo。</p>
<h3 data-nodeid="1159">Apache Dubbo</h3>
<p data-nodeid="1160">Dubbo 是阿里巴巴开源的高性能、轻量级的开源 Java 框架，目前被 Apache收录，官网是：</p>
<pre class="lang-java" data-nodeid="1161"><code data-language="java">http:<span class="hljs-comment">//dubbo.apache.org/</span>
</code></pre>
<p data-nodeid="1162"><img src="https://s0.lgstatic.com/i/image6/M01/1F/22/Cgp9HWBRmuqAa3CBAAIAHzTPLj0715.png" alt="图片22.png" data-nodeid="1294"></p>
<div data-nodeid="1163"><p style="text-align:center">Dubbo的官方介绍</p></div>
<p data-nodeid="1164">Dubbo 是典型的 RPC 框架的代表，通过客户端/服务端结构实现跨进程应用的高效二进制通信。</p>
<p data-nodeid="1165">Apache Dubbo 提供了六大核心能力：</p>
<ul data-nodeid="1166">
<li data-nodeid="1167">
<p data-nodeid="1168">面向接口代理的高性能 RPC 调用；</p>
</li>
<li data-nodeid="1169">
<p data-nodeid="1170">智能容错和负载均衡；</p>
</li>
<li data-nodeid="1171">
<p data-nodeid="1172">服务自动注册和发现；</p>
</li>
<li data-nodeid="1173">
<p data-nodeid="1174">高度可扩展能力；</p>
</li>
<li data-nodeid="1175">
<p data-nodeid="1176">运行期流量调度；</p>
</li>
<li data-nodeid="1177">
<p data-nodeid="1178">可视化的服务治理与运维。</p>
</li>
</ul>
<p data-nodeid="1179"><img src="https://s0.lgstatic.com/i/image6/M01/1F/1E/CioPOWBRmvaAKTm2AAPyFp0dSf8331.png" alt="图片3.png" data-nodeid="1305"></p>
<div data-nodeid="1180"><p style="text-align:center">Dubbo主要特性</p></div>
<p data-nodeid="1181">下图我们引用 Dubbo 的官方架构图，讲解 ApacheDubbo 的组成。</p>
<p data-nodeid="1182"><img src="https://s0.lgstatic.com/i/image6/M00/1F/1F/CioPOWBRmxWANqxoAAIXlEgpEbQ890.png" alt="图片4.png" data-nodeid="1309"></p>
<div data-nodeid="1183"><p style="text-align:center">Dubbo架构图</p></div>
<p data-nodeid="1184">Dubbo 架构中，包含 5 种角色。</p>
<ol data-nodeid="1185">
<li data-nodeid="1186">
<p data-nodeid="1187"><strong data-nodeid="1315">Provider</strong>：RPC服务提供者，Provider 是消息的最终处理者。</p>
</li>
<li data-nodeid="1188">
<p data-nodeid="1189"><strong data-nodeid="1320">Container</strong>：容器，用于启动、停止 Provider 服务。这里的容器并非 Tomcat、Jetty 等 Web 容器，Dubbo 也并不强制要求 Provider 必须具备 Web 能力。Dubbo 的容器是指对象容器，例如 Dubbo 内置的 SpringContainer 容器就提供了利用 Spring IOC 容器管理 Provider 对象的职能。</p>
</li>
<li data-nodeid="1190">
<p data-nodeid="1191"><strong data-nodeid="1325">Consumer</strong>：消费者，调用的发起者。Consumer 需要在客户端持有 Provider 的通信接口才能完成通信过程。</p>
</li>
<li data-nodeid="1192">
<p data-nodeid="1193"><strong data-nodeid="1330">Registry</strong>：注册中心，Dubbo 架构中注册中心与微服务架构中的注册中心职责类似，提供了 Dubbo Provider 的注册与发现职能，Consumer通过 Registry 可以获取Provider 可用的节点实例的 IP、端口等，并产生直接通信。需要注意的是，前面我们讲解的 Alibaba Nacos 除了可以作为微服务架构中的注册中心外，同样对自家的 Dubbo 提供了 RPC 调用注册发现的职责，这是其他 Spring Cloud 注册中心所不具备的功能。</p>
</li>
<li data-nodeid="1194">
<p data-nodeid="1195"><strong data-nodeid="1335">Monitor</strong>：监控器，监控器提供了Dubbo的监控职责。在 Dubbo 产生通信时，Monitor 进行收集、统计，并通过可视化 UI 界面帮助运维人员了解系统进程间的通信状况。Dubbo Monitor 主流产品有 Dubbo Admin、Dubbo Ops 等。</p>
</li>
</ol>
<p data-nodeid="1196">下面我们通过实例讲解 Dubbo 与 Nacos 如何协同作业实现服务间调用</p>
<h3 data-nodeid="1197">Dubbo与 Nacos 协同作业</h3>
<p data-nodeid="1198">为了方便理解，我们仍然采用 07 讲“订单与库存服务”案例，改由 RPC 方式实现通信。</p>
<p data-nodeid="1199"><img src="https://s0.lgstatic.com/i/image6/M01/1F/22/Cgp9HWBRmyOAQauOAAGVp6w3uG4991.png" alt="图片5.png" data-nodeid="1341"></p>
<div data-nodeid="1200"><p style="text-align:center">订单与仓储服务的业务流程</p></div>
<h4 data-nodeid="1201">开发 Provider仓储服务</h4>
<p data-nodeid="1202">第一步，创建工程引入依赖。</p>
<p data-nodeid="1203">利用 Spring Initializr 向导创建 warehouse-service 工程，确保 pom.xml 引入以下依赖。</p>
<pre class="lang-xml" data-nodeid="1204"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.dubbo<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>dubbo-spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.7.8<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-actuator<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1205">相比标准微服务，需要额外依赖 dubbo-spring-boot-starter 与 spring-boot-starter-actuator。其中 dubbo-spring-boot-starter 是 Dubbo 与 Spring Boot 整合最关键的组件，为 Spring Boot 提供了 Dubbo 的默认支持。而 spring-boot-starter-actuator则为微服务提供了监控指标接口，便于监控系统从应用收集各项运行指标。</p>
<p data-nodeid="1206">第二步，设置微服务、Dubbo 与 Nacos 通信选项。</p>
<p data-nodeid="1207">打开 application.yml 文件，配置 Nacos 地址与 Dubbo通信选项。</p>
<pre class="lang-yaml" data-nodeid="1208"><code data-language="yaml"><span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">8000</span> <span class="hljs-comment">#服务端口</span>
<span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">warehouse-service</span> <span class="hljs-comment">#微服务id</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">nacos:</span> <span class="hljs-comment">#nacos注册地址</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.101</span><span class="hljs-string">:8848</span>
        <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
        <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
<span class="hljs-attr">dubbo:</span> <span class="hljs-comment">#dubbo与nacos的通信配置</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">warehouse-dubbo</span> <span class="hljs-comment">#provider在Nacos中的应用id</span>
  <span class="hljs-attr">registry:</span> <span class="hljs-comment">#Provider与Nacos通信地址，与spring.cloud.nacos地址一致</span>
    <span class="hljs-attr">address:</span> <span class="hljs-string">nacos://192.168.31.101:8848</span>
  <span class="hljs-attr">protocol:</span> 
    <span class="hljs-attr">name:</span> <span class="hljs-string">dubbo</span> <span class="hljs-comment">#通信协议名</span>
    <span class="hljs-attr">port:</span> <span class="hljs-number">20880</span> <span class="hljs-comment">#配置通信端口，默认为20880</span>
  <span class="hljs-attr">scan:</span> 
    <span class="hljs-attr">base-packages:</span> <span class="hljs-string">com.lagou.warehouseservice.dubbo</span>
</code></pre>
<p data-nodeid="1209">很多人不明白，为什么上面配置了 spring.cloud.nacos.discovery.server-addr 还要在下面配置 dubbo.registry.address 呢？前面介绍架构时介绍了，Dubbo 需要依托 Container（容器）对外暴露服务，而这个容器配置与微服务配置是分开的，需要额外占用一个网络端口20880提供服务。</p>
<p data-nodeid="1210"><img src="https://s0.lgstatic.com/i/image6/M00/1F/22/Cgp9HWBRmziAAd_LAAE40sLBEjo538.png" alt="图片6.png" data-nodeid="1351"></p>
<div data-nodeid="1211"><p style="text-align:center">Dubbo Provider 端启动时向 Nacos 注册两次</p></div>
<p data-nodeid="1212">这里还有一个配置点需要特别注意：dubbo.scan.base-packages 代表在 Dubbo 容器启动时自动扫描 com.lagou.warehouseservice.dubbo 包下的接口与实现类，并将这些接口信息在Nacos 进行登记，因此 Dubbo 对外暴露的接口必须放在该包下。</p>
<p data-nodeid="1213">第三步，开发接口与实现类。</p>
<p data-nodeid="1214"><strong data-nodeid="1358">1</strong>. 开发 Stock 库存商品对象。</p>
<pre class="lang-java" data-nodeid="1215"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.warehouseservice.dto;
<span class="hljs-keyword">import</span> java.io.Serializable;
<span class="hljs-comment">//库存商品对象</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Stock</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
    <span class="hljs-keyword">private</span> Long skuId; <span class="hljs-comment">//商品品类编号</span>
    <span class="hljs-keyword">private</span> String title; <span class="hljs-comment">//商品与品类名称</span>
    <span class="hljs-keyword">private</span> Integer quantity; <span class="hljs-comment">//库存数量</span>
    <span class="hljs-keyword">private</span> String unit; <span class="hljs-comment">//单位</span>
    <span class="hljs-keyword">private</span> String description; <span class="hljs-comment">//描述信息</span>
    <span class="hljs-comment">//带参构造函数</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">Stock</span><span class="hljs-params">(Long skuId, String title, Integer quantity, String unit)</span> </span>{
        <span class="hljs-keyword">this</span>.skuId = skuId;
        <span class="hljs-keyword">this</span>.title = title;
        <span class="hljs-keyword">this</span>.quantity = quantity;
        <span class="hljs-keyword">this</span>.unit = unit;
    }
    <span class="hljs-comment">//getter 与 setter...</span>
}
</code></pre>
<p data-nodeid="1216">注意：Dubbo 在对象传输过程中使用了 JDK 序列化，对象必须实现 Serializable 接口。</p>
<p data-nodeid="1217"><strong data-nodeid="1364">2</strong>. 在 com.lagou.warehouseservice.dubbo包下创建 WarehouseService 接口并声明方法。包名要与 dubbo.scan.base-packages 保持一致。</p>
<pre class="lang-java" data-nodeid="1218"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.warehouseservice.dubbo;
<span class="hljs-keyword">import</span> com.lagou.warehouseservice.dto.Stock;
<span class="hljs-comment">//Provider接口</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">WarehouseService</span> </span>{
    <span class="hljs-comment">//查询库存</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Stock <span class="hljs-title">getStock</span><span class="hljs-params">(Long skuId)</span></span>;
}
</code></pre>
<p data-nodeid="1219"><strong data-nodeid="1369">3</strong>. 在 com.lagou.warehouseservice.dubbo.impl 包下创建实现类 WarehouseServiceImpl。</p>
<pre class="lang-java" data-nodeid="1220"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.warehouseservice.dubbo.impl;
<span class="hljs-keyword">import</span> com.lagou.warehouseservice.dto.Stock;
<span class="hljs-keyword">import</span> com.lagou.warehouseservice.dubbo.WarehouseService;
<span class="hljs-keyword">import</span> org.apache.dubbo.config.annotation.DubboService;
<span class="hljs-keyword">import</span> org.springframework.stereotype.Service;
<span class="hljs-keyword">import</span> java.util.HashMap;
<span class="hljs-keyword">import</span> java.util.Map;
<span class="hljs-meta">@DubboService</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WarehouseServiceImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WarehouseService</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> Stock <span class="hljs-title">getStock</span><span class="hljs-params">(Long skuId)</span></span>{
        Map result = <span class="hljs-keyword">new</span> HashMap();
        Stock stock = <span class="hljs-keyword">null</span>;
        <span class="hljs-keyword">if</span>(skuId == <span class="hljs-number">1101l</span>){
            <span class="hljs-comment">//模拟有库存商品</span>
            stock = <span class="hljs-keyword">new</span> Stock(<span class="hljs-number">1101l</span>, <span class="hljs-string">"Apple iPhone 11 128GB 紫色"</span>, <span class="hljs-number">32</span>, <span class="hljs-string">"台"</span>);
            stock.setDescription(<span class="hljs-string">"Apple 11 紫色版对应商品描述"</span>);
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(skuId == <span class="hljs-number">1102l</span>){
            <span class="hljs-comment">//模拟无库存商品</span>
            stock = <span class="hljs-keyword">new</span> Stock(<span class="hljs-number">1102l</span>, <span class="hljs-string">"Apple iPhone 11 256GB 白色"</span>, <span class="hljs-number">0</span>, <span class="hljs-string">"台"</span>);
            stock.setDescription(<span class="hljs-string">"Apple 11 白色版对应商品描述"</span>);
        }<span class="hljs-keyword">else</span>{
            <span class="hljs-comment">//演示案例，暂不考虑无对应skuId的情况</span>
        }
        <span class="hljs-keyword">return</span> stock;
    }
}
</code></pre>
<p data-nodeid="1221">实现逻辑非常简单，不做赘述，重点是在实现类上需要额外增加 @DubboService注解。@DubboService 是 Provider 注解，说明该类所有方法都是服务提供者，加入该注解会自动将类与方法的信息在 Nacos中注册为 Provider。</p>
<p data-nodeid="1222">第四步，启动微服务，验证 Nacos 注册信息。</p>
<p data-nodeid="1223">启动 WarehouseServiceApplication.main()，之后打开下面网址访问 Nacos 服务列表。</p>
<pre class="lang-java" data-nodeid="1224"><code data-language="java">http:<span class="hljs-comment">//192.168.31.101:8848/nacos/#/serviceManagement?dataId=&amp;group=&amp;appName=&amp;namespace=</span>
</code></pre>
<p data-nodeid="1225"><img src="https://s0.lgstatic.com/i/image6/M01/1F/1F/CioPOWBRm0mAKUgNAALNIHodMII419.png" alt="图片7.png" data-nodeid="1375"></p>
<div data-nodeid="1226"><p style="text-align:center">验证Nacos注册信息</p></div>
<p data-nodeid="1227">此时在服务列表中出现了 2 条数据，warehouse-service 是仓储微服务的注册信息，providers 开头的是 Dubbo 在 Nacos 的注册 Provider 信息，这也验证了前面介绍 Dubbo 的注册过程。</p>
<p data-nodeid="1228">而查看 Provider详情后，你会得到更多信息，其中包含 Provider 的 IP、端口、接口、方法、pid、版本的明细，方便开发、运维人员对应用进行管理。</p>
<p data-nodeid="1229"><img src="https://s0.lgstatic.com/i/image6/M01/1F/1F/CioPOWBRm1SAKLjbAAGK7S9aIpY685.png" alt="图片8.png" data-nodeid="1380"></p>
<div data-nodeid="1230"><p style="text-align:center">Provider详情</p></div>
<p data-nodeid="1231">到这里，仓储服务与 Dubbo Provider 的开发已完成。下面咱们开发Consumer消费者。</p>
<h4 data-nodeid="1232"><strong data-nodeid="1385">开发 Consumer 订单服务</strong></h4>
<p data-nodeid="1233">第一步，创建工程引入依赖。</p>
<p data-nodeid="1234">利用 Spring Initializr 向导创建 order-service 工程，确保 pom.xml 引入以下依赖，依赖部分与 warehouse-service 保持一致。</p>
<pre class="lang-xml" data-nodeid="1235"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.dubbo<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>dubbo-spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.7.8<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-actuator<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1236">第二步，设置微服务、Dubbo 与 Nacos通信选项。<br>
打开 application.yml文件，配置 Nacos 与 Dubbo 通信选项，因为 order-service 是消费者，因此不需要专门配置端口与 base-packages选项。</p>
<pre class="lang-yaml" data-nodeid="1237"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">order-service</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">nacos:</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.101</span><span class="hljs-string">:8848</span>
        <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
        <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
<span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">9000</span>
<span class="hljs-attr">dubbo:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">order-service-dubbo</span>
  <span class="hljs-attr">registry:</span>
    <span class="hljs-attr">address:</span> <span class="hljs-string">nacos://192.168.31.101:8848</span>
</code></pre>
<p data-nodeid="1238">第三步，将 Provider 端接口 WarehouseService 以及依赖的 Stock类复制到 order-service 工程，注意要求包名、类名及代码保持完全一致。当然我这种做法比较原始，在项目环境通常是将接口与依赖的类发布到 Maven 仓库，由 Consumer 直接依赖即可。</p>
<p data-nodeid="1239"><img src="https://s0.lgstatic.com/i/image6/M01/1F/20/CioPOWBRm2KAIpCRAAJoR7cA3Hw642.png" alt="图片9.png" data-nodeid="1394"></p>
<div data-nodeid="1240"><p style="text-align:center">必须保证 Provider 与 Consumer 接口一致</p></div>
<p data-nodeid="1241">第四步，Consumer 调用接口实现业务逻辑。</p>
<p data-nodeid="1242">这一步最为关键，先给出代码再进行分析。</p>
<pre class="lang-java" data-nodeid="1243"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderController</span> </span>{
    <span class="hljs-meta">@DubboReference</span>
    <span class="hljs-keyword">private</span> WarehouseService warehouseService;
    <span class="hljs-comment">/**
     * 创建订单业务逻辑
     * <span class="hljs-doctag">@param</span> skuId 商品类别编号
     * <span class="hljs-doctag">@param</span> salesQuantity 销售数量
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@GetMapping("/create_order")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Map <span class="hljs-title">createOrder</span><span class="hljs-params">(Long skuId , Long salesQuantity)</span></span>{
        Map result = <span class="hljs-keyword">new</span> LinkedHashMap();
        <span class="hljs-comment">//查询商品库存，像调用本地方法一样完成业务逻辑。</span>
        Stock stock = warehouseService.getStock(skuId);
        System.out.println(stock);
        <span class="hljs-keyword">if</span>(salesQuantity &lt;= stock.getQuantity()){
            <span class="hljs-comment">//创建订单相关代码，此处省略</span>
            <span class="hljs-comment">//CODE=SUCCESS代表订单创建成功</span>
            result.put(<span class="hljs-string">"code"</span> , <span class="hljs-string">"SUCCESS"</span>);
            result.put(<span class="hljs-string">"skuId"</span>, skuId);
            result.put(<span class="hljs-string">"message"</span>, <span class="hljs-string">"订单创建成功"</span>);
        }<span class="hljs-keyword">else</span>{
            <span class="hljs-comment">//code=NOT_ENOUGN_STOCK代表库存不足</span>
            result.put(<span class="hljs-string">"code"</span>, <span class="hljs-string">"NOT_ENOUGH_STOCK"</span>);
            result.put(<span class="hljs-string">"skuId"</span>, skuId);
            result.put(<span class="hljs-string">"message"</span>, <span class="hljs-string">"商品库存数量不足"</span>);
        }
        <span class="hljs-keyword">return</span> result;
    }
}
</code></pre>
<p data-nodeid="1244">业务逻辑非常简单，前文讲过不再赘述，关键点是第三行 @DubboReference 注解。该注解用在 Consumer 端，说明 WarehouseService 是 Dubbo Consumer 接口，Spring 会自动生成 WarehouseService 接口的代理实现类，并隐藏远程通信细节，处理流程如下图所示：</p>
<p data-nodeid="1245"><img src="https://s0.lgstatic.com/i/image6/M00/1F/21/CioPOWBRm8WAUi4pAAFRuMBBSJc896.png" alt="图片10 (2).png" data-nodeid="1400"></p>
<div data-nodeid="1246"><p style="text-align:center">Dubbo Consumer 处理流程</p></div>
<p data-nodeid="1247">第五步，启动微服务，验证 Nacos 注册信息。</p>
<p data-nodeid="1248">启动 OrderServiceApplication.main()，之后打开下面网址访问 Nacos 服务列表。</p>
<pre class="lang-java" data-nodeid="1249"><code data-language="java">http:<span class="hljs-comment">//192.168.31.101:8848/nacos/#/serviceManagement?dataId=&amp;group=&amp;appName=&amp;namespace=</span>
</code></pre>
<p data-nodeid="1250"><img src="https://s0.lgstatic.com/i/image6/M01/1F/20/CioPOWBRm3uAW-qFAAJ0jHj71gk377.png" alt="图片11.png" data-nodeid="1405"></p>
<div data-nodeid="1251"><p style="text-align:center">DubboConsumer 注册成功</p></div>
<p data-nodeid="1252">此时 Consumer 已在服务列表中出现，说明消费者已注册成功。</p>
<p data-nodeid="1253">最后，打开浏览器访问下面网址验证结果。</p>
<pre class="lang-javascript" data-nodeid="1254"><code data-language="javascript">http:<span class="hljs-comment">//192.168.31.106:9000/create_order?skuId=1101&amp;salesQuantity=10</span>
{
<span class="hljs-attr">code</span>:&nbsp;<span class="hljs-string">"SUCCESS"</span>,
<span class="hljs-attr">skuId</span>:&nbsp;<span class="hljs-number">1101</span>,
<span class="hljs-attr">message</span>:&nbsp;<span class="hljs-string">"订单创建成功"</span>
}
</code></pre>
<div data-nodeid="1255"><p style="text-align:center">订单创建成功</p></div>
<pre class="lang-javascript" data-nodeid="1256"><code data-language="javascript">http:<span class="hljs-comment">//192.168.31.106:9000/create_order?skuId=1102&amp;salesQuantity=1</span>
{
<span class="hljs-attr">code</span>:&nbsp;<span class="hljs-string">"NOT_ENOUGH_STOCK"</span>,
<span class="hljs-attr">skuId</span>:&nbsp;<span class="hljs-number">1102</span>,
<span class="hljs-attr">message</span>:&nbsp;<span class="hljs-string">"商品库存数量不足"</span>
}
</code></pre>
<div data-nodeid="1257"><p style="text-align:center">订单创建失败</p></div>
<p data-nodeid="1258">至此，Dubbo 与 Nacos 完整的接入流程与开发技巧我们通过案例方式进行了讲解，最后咱们来总结一下。</p>
<h3 data-nodeid="1259">小结与预告</h3>
<p data-nodeid="1260">本节主要学习了三方面知识，首先介绍 RPC 与 RESTful 的区别，之后对 Dubbo 架构进行了介绍，最后通过案例详细讲解了 Dubbo 与 Nacos的协同作业过程。</p>
<p data-nodeid="1261">在这里我为你准备了一道思考题：目前以微信、QQ 为代表的即时通信类软件，为保证低延时，通信方式使用 RESTful 还是 RPC 呢？请你把思考后的原因和结果写在评论中，与其他同学们一起交流。</p>
<p data-nodeid="1262" class="">课程预告，下一节咱们将学习微服务的门户：API 网关。理解通过网关如何对用户屏蔽微服务架构的实现细节。</p>

---

### 精选评论

##### **镔：
> 老师在用nacos和dubbo时，有没有发现提供者下线后，再上线。消费者就再也执行不到提供者了，不管等多久。需要消费者重启才行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，可能以前我们都在做RESTful，少量RPC的场景还真没出现过你说的问题，我找个时间搭个环境看看。

##### ccqstark：
> 微信、QQ这种用的应该是RPC，因为要长链接、低延时

##### **6881：
> 课程的demo工程 在哪里下载啊？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 兄弟，我在整理4月6日统一上传到github

##### *佳：
> 微信、qq都有两种通讯模式，双方同时在线和只有其中一方在线。单纯只使用rpc并不能满足业务需求，同时在线时可以使用rpc。只有一方在线时可以使用类似udp这样的通信协议。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，我们可以根据场景来决定采用哪种通信。但项目中最好不要选择udp，udp不靠谱哇 ：）

##### **街牛牛：
> 先启动消费者(关闭了检查)再启动提供者消费者始终找不到提供者。消费者必须在提供者之后启动才能感知到吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是必须的,过一小段时间消费者会自动发现.

##### Null：
> 为什么spring.cloud.nacos.discovery.server-addr 还要在下面配置 dubbo.registry.address？这个地方没明白，他们有什么区别？各自有什么作用？能举个例子吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一个是Nacos服务发现的地址,一个是Dubbo的地址.在绝大多数情况下两者都是相同的,指向Nacos.

##### **用户5069：
> 老师，DUBBO可以替换成其他序列化方式么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; DUBBO支持多种协议,如hessian rmi  这些序列化方式都不一样.

##### **政：
> 老师，08 Consumer 启动报错@DubboReference注入失败，在nacos控制台只能看到三个服务，2个provide与一个consumer。我用的是这个spring-cloud-starter-dubbo依赖，换了Apache的dubbo依赖，nacos控制台会没有dubbo的服务注册上去。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 兄弟，能放下控制台日志吗，没有日志不好判断呀 ：）

##### **伟：
> 什么是长连接应用，短连接应用？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 长连接只是客户端与服务端长期持有一个连接进行通信。短连接是用完了马上就关掉。
长连接典型场景比如网游，短连接场景比如标准的HTTP通信。

##### *爽：
> 之前没有用过dubbo，以为rpc使用很难，但现在觉得也就那样

##### **良：
> 老师我想问问这里的dubbo如何实现客户端负载均衡？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯，这是个好问题。一般用Dubbo都是基于Nacos做服务器负载均衡。至于客户端负载均衡还得看Dubbo官方对Nacos的支持啦。

##### **6754：
> dubbo调用时，只需要用dubbo方式注册到nacos，微服务可以不用注册到nacos吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; dubbo通信时从nacos获取dubbo通信端口即可，但微服务必须注册，微服务作为dubbo的容器，如果没能注册nacos就无法服务发现，后面的通信都是空谈。

##### **冰茶：
> 老师，示例代码可以放在git一份吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 兄弟，我在整理4月6日统一上传到github

##### **1001：
> 老师你好！请问一下dubbo调用可以获取当前登录用户信息吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以，但要在传递时将用户数据手动封装成Bean进行传递。

##### **军：
> 为啥不用 spring-cloud-starter-dubbo这个依赖来整合啊?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 都可以啊，spring-cloud-stater-dubbo 和dubbo-spring-boot-starter本质都是一样的，通过springboot自动装配简化配置过程。相对后者更新一点，这个看个人喜好就行。

