<p data-nodeid="1645" class="">上一讲我为各位讲解了 Nacos 配置中心的用途及配置技巧。本讲咱们基于上一讲的成果，学习如何在生产环境下通过 Nacos 实现 Sentinel 规则持久化。本讲咱们将介绍三方面内容：</p>
<ul data-nodeid="1646">
<li data-nodeid="1647">
<p data-nodeid="1648">Sentinel 与Nacos整合实现规则持久化；</p>
</li>
<li data-nodeid="1649">
<p data-nodeid="1650">自定义资源点进行熔断保护；</p>
</li>
<li data-nodeid="1651">
<p data-nodeid="1652">开发友好的异常处理程序。</p>
</li>
</ul>
<h3 data-nodeid="1653">Sentinel 与 Nacos 整合实现规则持久化</h3>
<p data-nodeid="4465" class="">细心的你肯定在前面 Sentinel的使用过程中已经发现，当微服务重启以后所有的配置规则都会丢失，其中的根源是默认微服务将 Sentinel 的规则保存在 JVM 内存中，当应用重启后 JVM 内存销毁，规则就会丢失。为了解决这个问题，我们就需要通过某种机制将配置好的规则进行持久化保存，同时这些规则变更后还能及时通知微服务进行变更。</p>




<p data-nodeid="1655" class="">正好，上一讲我们讲解了 Nacos 配置中心的用法，无论是配置数据的持久化特性还是配置中心主动推送的特性都是我们需要的，因此 Nacos 自然就成了 Sentinel 规则持久化的首选。</p>
<p data-nodeid="1656">本讲我们仍然通过实例讲解 Sentinel 与 Nacos 的整合过程。</p>
<h4 data-nodeid="1657">案例准备</h4>
<p data-nodeid="1658">首先，咱们快速构建演示工程 sentinel-sample。</p>
<p data-nodeid="1659"><strong data-nodeid="1814">1</strong>. 利用 Spring Initializr 向导创建 sentinel-sample 工程，pom.xml 增加以下三项依赖。</p>
<pre class="lang-xml" data-nodeid="1660"><code data-language="xml"><span class="hljs-comment">&lt;!-- Nacos 客户端 Starter--&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- Sentinel 客户端 Starter--&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-sentinel<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-comment">&lt;!-- 对外暴露 Spring Boot 监控指标--&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-actuator<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1661"><strong data-nodeid="1819">2</strong>. 配置 Nacos 与 Sentinel 客户端。</p>
<pre class="lang-yaml" data-nodeid="1662"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">sentinel-sample</span> <span class="hljs-comment">#应用名&amp;微服务 id</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">sentinel:</span> <span class="hljs-comment">#Sentinel Dashboard 通信地址</span>
      <span class="hljs-attr">transport:</span>
        <span class="hljs-attr">dashboard:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:9100</span>
      <span class="hljs-attr">eager:</span> <span class="hljs-literal">true</span> <span class="hljs-comment">#取消控制台懒加载</span>
    <span class="hljs-attr">nacos:</span> <span class="hljs-comment">#Nacos 通信地址</span>
      <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:8848</span>
      <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
      <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
  <span class="hljs-attr">jackson:</span>
    <span class="hljs-attr">default-property-inclusion:</span> <span class="hljs-string">non_null</span>
<span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
<span class="hljs-attr">management:</span>
  <span class="hljs-attr">endpoints:</span>
    <span class="hljs-attr">web:</span> <span class="hljs-comment">#将所有可用的监控指标项对外暴露</span>
      <span class="hljs-attr">exposure:</span> <span class="hljs-comment">#可以访问 /actuator/sentinel进行查看Sentinel监控项</span>
        <span class="hljs-attr">include:</span> <span class="hljs-string">'*'</span>
<span class="hljs-attr">logging:</span>
  <span class="hljs-attr">level:</span>
    <span class="hljs-attr">root:</span> <span class="hljs-string">debug</span> <span class="hljs-comment">#开启 debug 是学习需要，生产改为 info 即可</span>
</code></pre>
<p data-nodeid="1663"><strong data-nodeid="1824">3</strong>. 在 sentinel-sample服务中，增加 SentinelSampleController 类，用于演示运行效果。</p>
<pre class="lang-java" data-nodeid="1664"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SentinelSampleController</span> </span>{
    <span class="hljs-comment">//演示用的业务逻辑类</span>
    <span class="hljs-meta">@Resource</span>
    <span class="hljs-keyword">private</span> SampleService sampleService;
    <span class="hljs-comment">/**
     * 流控测试接口
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@GetMapping("/test_flow_rule")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> ResponseObject <span class="hljs-title">testFlowRule</span><span class="hljs-params">()</span></span>{
        <span class="hljs-comment">//code=0 代表服务器处理成功</span>
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ResponseObject(<span class="hljs-string">"0"</span>,<span class="hljs-string">"success!"</span>);
    }

    <span class="hljs-comment">/**
     * 熔断测试接口
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@GetMapping("/test_degrade_rule")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> ResponseObject <span class="hljs-title">testDegradeRule</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">try</span> {
            sampleService.createOrder();
        }<span class="hljs-keyword">catch</span> (IllegalStateException e){
            <span class="hljs-comment">//当 createOrder 业务处理过程中产生错误时会抛出IllegalStateException</span>
            <span class="hljs-comment">//IllegalStateException 是 JAVA 内置状态异常，在项目开发时可以更换为自己项目的自定义异常</span>
            <span class="hljs-comment">//出现错误后将异常封装为响应对象后JSON输出</span>
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ResponseObject(e.getClass().getSimpleName(),e.getMessage());
        }
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ResponseObject(<span class="hljs-string">"0"</span>,<span class="hljs-string">"order created!"</span>);
    }
}
</code></pre>
<p data-nodeid="1665">此外，ResponseObject 对象封装了响应的数据。</p>
<pre class="lang-java" data-nodeid="1666"><code data-language="java"><span class="hljs-comment">/**
 * 封装响应数据的对象
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ResponseObject</span> </span>{
    <span class="hljs-keyword">private</span> String code; <span class="hljs-comment">//结果编码，0-固定代表处理成功</span>
    <span class="hljs-keyword">private</span> String message;<span class="hljs-comment">//响应消息</span>
    <span class="hljs-keyword">private</span> Object data;<span class="hljs-comment">//响应附加数据（可选）</span>
 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ResponseObject</span><span class="hljs-params">(String code, String message)</span> </span>{
        <span class="hljs-keyword">this</span>.code = code;
        <span class="hljs-keyword">this</span>.message = message;
    }
    <span class="hljs-comment">//Getter/Setter省略</span>
}
</code></pre>
<p data-nodeid="1667"><strong data-nodeid="1830">4</strong>. 额外增加 SampleService 用于模拟业务逻辑，等下我们将用它讲解自定义资源点与熔断设置。</p>
<pre class="lang-java" data-nodeid="1668"><code data-language="java"><span class="hljs-comment">/**
 * 演示用的业务逻辑类
 */</span>
<span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SampleService</span> </span>{
    <span class="hljs-comment">//模拟创建订单业务</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createOrder</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">try</span> {
            <span class="hljs-comment">//模拟处理业务逻辑需要101毫秒</span>
            Thread.sleep(<span class="hljs-number">101</span>);
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(<span class="hljs-string">"订单已创建"</span>);
    }
}
</code></pre>
<p data-nodeid="1669">启动 sentinel-sample，访问<a href="http://localhost/test_flow_rule?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1838">http://localhost/test_flow_rule</a>，浏览器出现code=0 的 JSON 响应，说明 sentinel-sample 服务启动成功。</p>
<pre class="lang-json" data-nodeid="1670"><code data-language="json">{
    code:&nbsp;"0",
    message:&nbsp;"success!"
}
</code></pre>
<h4 data-nodeid="1671">流控规则持久化</h4>
<p data-nodeid="1672">工程创建完成，下面咱们将 Sentinel接入 Nacos 配置中心。</p>
<p data-nodeid="1673">第一步，pom.xml 新增 sentinel-datasource-nacos 依赖。</p>
<pre class="lang-xml" data-nodeid="1674"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.csp<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>sentinel-datasource-nacos<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="1675">sentinel-datasource-nacos 是 Sentinel 为 Nacos 扩展的数据源模块，允许将规则数据存储在 Nacos 配置中心，在微服务启动时利用该模块 Sentinel 会自动在 Nacos下载对应的规则数据。</p>
<p data-nodeid="1676">第二步，在application.yml 文件中增加 Nacos下载规则，在原有的sentinel 配置下新增 datasource 选项。这里咱们暂时只对流控规则进行设置，重要配置项我在代码中进行了注释，请同学们仔细阅读。</p>
<pre class="lang-yaml" data-nodeid="1677"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">sentinel-sample</span> <span class="hljs-comment">#应用名&amp;微服务id</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">sentinel:</span> <span class="hljs-comment">#Sentinel Dashboard通信地址</span>
      <span class="hljs-attr">transport:</span>
        <span class="hljs-attr">dashboard:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:9100</span>
      <span class="hljs-attr">eager:</span> <span class="hljs-literal">true</span> <span class="hljs-comment">#取消控制台懒加载</span>
      <span class="hljs-attr">datasource:</span>
        <span class="hljs-attr">flow:</span> <span class="hljs-comment">#数据源名称，可以自定义</span>
          <span class="hljs-attr">nacos:</span> <span class="hljs-comment">#nacos配置中心</span>
            <span class="hljs-comment">#Nacos内置配置中心，因此重用即可</span>
            <span class="hljs-attr">server-addr:</span> <span class="hljs-string">${spring.cloud.nacos.server-addr}</span> 
            <span class="hljs-attr">dataId:</span> <span class="hljs-string">${spring.application.name}-flow-rules</span> <span class="hljs-comment">#定义流控规则data-id，完整名称为:sentinel-sample-flow-rules，在配置中心设置时data-id必须对应。</span>
            <span class="hljs-attr">groupId:</span> <span class="hljs-string">SAMPLE_GROUP</span> <span class="hljs-comment">#gourpId对应配置文件分组，对应配置中心groups项</span>
            <span class="hljs-attr">rule-type:</span> <span class="hljs-string">flow</span> <span class="hljs-comment">#flow固定写死，说明这个配置是流控规则</span>
            <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span> <span class="hljs-comment">#nacos通信的用户名与密码</span>
            <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
    <span class="hljs-attr">nacos:</span> <span class="hljs-comment">#Nacos通信地址</span>
      <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:8848</span>
      <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
      <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
    <span class="hljs-string">...</span>
</code></pre>
<p data-nodeid="1678" class="">通过这一份配置，微服务在启动时就会自动从 Nacos 配置中心 SAMPLE_GROUP 分组下载 data-id 为 sentinel-sample-flow-rules 的配置信息并将其作为流控规则生效。<br>
第三步，在 Nacos 配置中心页面，新增 data-id 为sentinel-sample-flow-rules 的配置项。</p>
<p data-nodeid="1679"><img src="https://s0.lgstatic.com/i/image6/M00/27/B7/Cgp9HWBdlU2AH_74AADL3_g2qEo343.png" alt="Drawing 0.png" data-nodeid="1852"></p>
<div data-nodeid="1680"><p style="text-align:center">流控规则设置</p></div>
<p data-nodeid="1681" class="">这里 data-id 与 groups 与微服务应用的配置保持对应，最核心的配置内容采用 JSON 格式进行书写，我们来分析下这段流控规则。</p>
<pre class="lang-json" data-nodeid="1682"><code data-language="json">[
    {
        "resource":"/test_flow_rule", #资源名，说明对那个URI进行流控
        "limitApp":"default",  #命名空间，默认default
        "grade":1, #类型 0-线程 1-QPS
        "count":2, #超过2个QPS限流将被限流
        "strategy":0, #限流策略: 0-直接 1-关联 2-链路
        "controlBehavior":0, #控制行为: 0-快速失败 1-WarmUp 2-排队等待
        "clusterMode":false #是否集群模式
    }
]
</code></pre>
<p data-nodeid="12588" class="te-preview-highlight">仔细观察不难发现，这些配置项与 Dashboard UI 中的选项是对应的，其实使用 DashboardUI 最终目的是为了生成这段 JSON 数据而已，只不过通过 UI 更容易使用罢了。</p>

<p data-nodeid="1684"><img src="https://s0.lgstatic.com/i/image6/M00/27/B4/CioPOWBdlVmAUv8YAADoB7vbrZs676.png" alt="Drawing 1.png" data-nodeid="1857"></p>
<div data-nodeid="1685"><p style="text-align:center">Sentinel Dashboard 流控设置界面</p></div>
<p data-nodeid="1686">关于这些设置项的各种细节，有兴趣的同学可以访问 Sentinel 的 GitHub 文档进行学习。</p>
<p data-nodeid="1687"><a href="https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1861">https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6</a></p>
<p data-nodeid="1688">最后，我们启动应用来验证流控效果。</p>
<p data-nodeid="1689">访问 <a href="http://localhost/test_flow_rule?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1870">http://localhost/test_flow_rule</a>，在日志中将会看到这条 Debug 信息，说明在服务启动时已向 Nacos 配置中心获取到流控规则。</p>
<pre class="lang-java" data-nodeid="1690"><code data-language="java">DEBUG <span class="hljs-number">12728</span> --- [main] s.n.www.protocol.http.HttpURLConnection  : sun.net.www.MessageHeader@<span class="hljs-number">5432948015</span> pairs: {GET /nacos/v1/cs/configs?dataId=sentinel-sample-flow-rules&amp;accessToken=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTYxMDg3NTA1M30.Hq561OkXuAqPI20IBsnPIn0ia86R9sZgdWwa_K8zwvw&amp;group=SAMPLE_GROUP HTTP/<span class="hljs-number">1.1</span>: <span class="hljs-keyword">null</span>}...
</code></pre>
<p data-nodeid="1691">咱们在浏览器反复刷新，当 test_flow_rule 接口每秒超过 2 次访问，就会出现“Blocked by Sentinel (flow limiting)”的错误信息，说明流控规则已生效。</p>
<p data-nodeid="1692">之后我们再来验证能否自动推送新规则，将Nacos 配置中心中流控规则 count 选项改为 1。</p>
<pre class="lang-json" data-nodeid="1693"><code data-language="json">[
    {
        "resource":"/test_flow_rule", 
        "limitApp":"default",
        "grade":1, 
        "count":1, #2改为1 
        "strategy":0, 
        "controlBehavior":0, 
        "clusterMode":false 
    }
]
</code></pre>
<div data-nodeid="1694"><p style="text-align:center">修改后的流控规则</p></div>
<p data-nodeid="1695">当新规则发布后，sentinel-sample控制台会立即收到下面的日志，说明新的流控规则即时生效。</p>
<pre class="lang-java" data-nodeid="1696"><code data-language="java">DEBUG <span class="hljs-number">12728</span> --- [.<span class="hljs-number">168.31</span><span class="hljs-number">.10_8848</span>] s.n.www.protocol.http.HttpURLConnection  : sun.net.www.MessageHeader@<span class="hljs-number">41257f</span>3915 pairs: {GET /nacos/v1/cs/configs?dataId=sentinel-sample-flow-rules&amp;accessToken=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTYxMDg3NTA1M30.Hq561OkXuAqPI20IBsnPIn0ia86R9sZgdWwa_K8zwvw&amp;group=SAMPLE_GROUP HTTP/<span class="hljs-number">1.1</span>: <span class="hljs-keyword">null</span>}
</code></pre>
<p data-nodeid="1697">在无须重启的情况下，test_flow_rule 接口 QPS 超过 1 就会直接报错。</p>
<p data-nodeid="1698">与此同时，我们还可以通过 Spring Boot Actuator 提供的监控指标确认流控规则已生效。</p>
<p data-nodeid="1699">访问 <a href="http://localhost/actuator/sentinel?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1888">http://localhost/actuator/sentinel</a>，在 flowRules 这个数组中，可以看到 test_flow_rule 的限流规则，每一次流控规则产生改变时 Nacos 都会主动推送到微服务并立即生效。</p>
<p data-nodeid="1700"><img src="https://s0.lgstatic.com/i/image6/M01/27/B7/Cgp9HWBdlZCALcRVAADl835eOig018.png" alt="Drawing 2.png" data-nodeid="1896"></p>
<div data-nodeid="1701"><p style="text-align:center">新的流控规则</p></div>
<h3 data-nodeid="1702">自定义资源点进行熔断保护</h3>
<p data-nodeid="1703">讲到这，我们已经实现了 test_flow_rule 接口的流控规则，但你发现了没有，在前面一系列 Sentinel 的讲解中我们都是针对 RESTful 的接口进行限流熔断设置，但是在项目中有的时候是要针对某一个 Service 业务逻辑方法进行限流熔断等规则设置，这要怎么办呢？</p>
<p data-nodeid="1704">在 sentinel-core 客户端中为开发者提供了 @SentinelResource 注解，该注解允许在程序代码中自定义 Sentinel 资源点来实现对特定方法的保护，下面我们以熔断降级规则为例来进行讲解。熔断降级是指在某个服务接口在执行过程中频繁出现故障的情况下，在一段时间内将服务停用的保护措施。</p>
<p data-nodeid="1705">在 sentinel-core 中基于 Spring AOP（面向切面技术）可在目标 Service 方法执行前按熔断规则进行检查，只允许有效的数据被送入目标方法进行处理。</p>
<p data-nodeid="1706">还是以 sentinel-sample 工程为例，我希望对 SampleSerivce.createOrder方法进行熔断保护，该如何设置呢？</p>
<p data-nodeid="1707"><strong data-nodeid="1909">第一步，声明切面类。</strong></p>
<p data-nodeid="1708">在应用入口 SentinelSampleApplication声明 SentinelResourceAspect，SentinelResourceAspect就是 Sentinel 提供的切面类，用于进行熔断的前置检查。</p>
<pre class="lang-java" data-nodeid="1709"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SentinelSampleApplication</span> </span>{
    <span class="hljs-comment">// 注解支持的配置Bean</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> SentinelResourceAspect <span class="hljs-title">sentinelResourceAspect</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SentinelResourceAspect();
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
        SpringApplication.run(SentinelSampleApplication.class, args);
    }
}
</code></pre>
<p data-nodeid="1710">第二步，声明资源点。</p>
<p data-nodeid="1711">在 SampleService.createOrder 方法上增加 @SentinelResource 注解用于声明 Sentinel 资源点，只有声明了资源点，Sentinel 才能实施限流熔断等保护措施。</p>
<pre class="lang-java" data-nodeid="1712"><code data-language="java"><span class="hljs-comment">/**
 * 演示用的业务逻辑类
 */</span>
<span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SampleService</span> </span>{
    <span class="hljs-comment">//资源点名称为createOrder</span>
    <span class="hljs-meta">@SentinelResource("createOrder")</span>
    <span class="hljs-comment">/**
     * 模拟创建订单业务
     * 抛出IllegalStateException异常用于模拟业务逻辑执行失败的情况
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createOrder</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> IllegalStateException</span>{
        <span class="hljs-keyword">try</span> {
            <span class="hljs-comment">//模拟处理业务逻辑需要101毫秒</span>
            Thread.sleep(<span class="hljs-number">101</span>);
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(<span class="hljs-string">"订单已创建"</span>);
    }
}
</code></pre>
<p data-nodeid="1713">修改完毕，启动服务访问 <a href="http://localhost/test_degrade_rule?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1920">http://localhost/test_degrade_rule</a>，当见到 code=0 的JSON 响应便代表应用运行正常。</p>
<pre class="lang-java" data-nodeid="1714"><code data-language="java">{
    code:&nbsp;<span class="hljs-string">"0"</span>,
    message:&nbsp;<span class="hljs-string">"order created!"</span>
}
</code></pre>
<p data-nodeid="1715">然后打开访问 Sentinel Dashboard，在簇点链路中要确认 createOrder资源点已存在。</p>
<p data-nodeid="1716"><img src="https://s0.lgstatic.com/i/image6/M01/27/B7/Cgp9HWBdlaqATmTiAAFaj_pNHQg367.png" alt="Drawing 3.png" data-nodeid="1925"></p>
<div data-nodeid="1717"><p style="text-align:center">createOrder 资源点已生效</p></div>
<p data-nodeid="1718">第三步，获取熔断规则。</p>
<p data-nodeid="1719">打开sentinel-sample 工程的 application.yml 文件，将服务接入 Nacos 配置中心的参数以获取熔断规则。</p>
<pre class="lang-yaml" data-nodeid="1720"><code data-language="yaml"><span class="hljs-attr">datasource:</span>
  <span class="hljs-attr">flow:</span> <span class="hljs-comment">#之前的流控规则，直接忽略</span>
    <span class="hljs-string">...</span>
  <span class="hljs-attr">degrade:</span> <span class="hljs-comment">#熔断规则</span>
    <span class="hljs-attr">nacos:</span>
      <span class="hljs-attr">server-addr:</span> <span class="hljs-string">${spring.cloud.nacos.server-addr}</span>
      <span class="hljs-attr">dataId:</span> <span class="hljs-string">${spring.application.name}-degrade-rules</span>
      <span class="hljs-attr">groupId:</span> <span class="hljs-string">SAMPLE_GROUP</span>
      <span class="hljs-attr">rule-type:</span> <span class="hljs-string">degrade</span> <span class="hljs-comment">#代表熔断</span>
     <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
     <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
</code></pre>
<p data-nodeid="1721">熔断规则的获取过程和前面流控规则类似，只不过 data-id 改为sentinel-sample-degrade-rules，以及 rule-type 更改为 degrade。</p>
<p data-nodeid="1722">启动过程中，出现下面日志代表配置成功。</p>
<pre class="lang-java" data-nodeid="1723"><code data-language="java">[main] s.n.www.protocol.http.HttpURLConnection  : sun.net.www.MessageHeader<span class="hljs-meta">@d96945215</span> pairs: {GET /nacos/v1/cs/configs?dataId=sentinel-sample-degrade-rules&amp;accessToken=eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJuYWNvcyIsImV4cCI6MTYxMDg5MDMwNH0.ooHkFb4zX14klmHMuLXTDkHSoCrwI8LtN7ex__9tMHg&amp;group=SAMPLE_GROUP HTTP/<span class="hljs-number">1.1</span>: <span class="hljs-keyword">null</span>}...
</code></pre>
<p data-nodeid="1724">第四步，在 Nacos 配置熔断规则。<br>
设置 data-id 为 sentinel-sample-degrade-rules，Groups 为 SAMPLE_GROUP与微服务的设置保持一致。</p>
<p data-nodeid="1725"><img src="https://s0.lgstatic.com/i/image6/M01/27/B7/Cgp9HWBdlbaAND8LAAChJna7ALA579.png" alt="Drawing 4.png" data-nodeid="1937"></p>
<p data-nodeid="1726">配置内容如下，我对每一项也做了说明。</p>
<pre class="lang-json" data-nodeid="1727"><code data-language="json">[{
    "resource":&nbsp;"createOrder", #自定义资源名
    "limitApp":&nbsp;"default", #命名空间
    "grade":&nbsp;0, #0-慢调用比例 1-异常比例 2-异常数
    "count":&nbsp;100, #最大RT 100毫秒执行时间
    "timeWindow":&nbsp;5, #时间窗口5秒
    "minRequestAmount":&nbsp;1, #最小请求数
    "slowRatioThreshold":&nbsp;0.1 #比例阈值
}]
</code></pre>
<p data-nodeid="1728">上面这段 JSON 完整的含义是：在过去 1 秒内，如果 createOrder资源被访问 1 次后便开启熔断检查，如果其中有 10% 的访问处理时间超过 100 毫秒，则触发熔断 5 秒钟，这期间访问该方法所有请求都将直接抛出 DegradeException，5 秒后该资源点回到“半开”状态等待新的访问，如果下一次访问处理成功，资源点恢复正常状态，如果下一次处理失败，则继续熔断 5 秒钟。<br>
<img src="https://s0.lgstatic.com/i/image6/M01/2A/57/Cgp9HWBihaeAcu_rAADj7f0dzWU991.png" alt="图片1-.png" data-nodeid="1943"></p>
<div data-nodeid="1729"><p style="text-align:center">熔断机制示意图</p></div>
<p data-nodeid="1730">设置成功，访问 Spring Boot Actuator<a href="http://localhost/actuator/sentinel?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1947">http://localhost/actuator/sentinel</a>，可以看到此时 gradeRules 数组下 createOrder 资源点的熔断规则已被 Nacos推送并立即生效。</p>
<p data-nodeid="1731"><img src="https://s0.lgstatic.com/i/image6/M01/27/B4/CioPOWBdldiAfwYZAADXzZezOVY956.png" alt="Drawing 6.png" data-nodeid="1951"></p>
<div data-nodeid="1732"><p style="text-align:center">自定义资源点熔断规则</p></div>
<p data-nodeid="1733">下面咱们来验证下，因为规则允许最大时长为 100 毫秒，而在 createOrder 方法中模拟业务处理需要 101 毫秒，显然会触发熔断。</p>
<pre class="lang-java" data-nodeid="1734"><code data-language="java"><span class="hljs-meta">@SentinelResource("createOrder")</span>
<span class="hljs-comment">//模拟创建订单业务</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createOrder</span><span class="hljs-params">()</span></span>{
    <span class="hljs-keyword">try</span> {
        <span class="hljs-comment">//模拟处理业务逻辑需要101毫秒</span>
        Thread.sleep(<span class="hljs-number">101</span>);
    } <span class="hljs-keyword">catch</span> (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println(<span class="hljs-string">"订单已创建"</span>);
}
</code></pre>
<p data-nodeid="1735">连续访问 <a href="http://localhost/test_degrade_rule?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1960">http://localhost/test_degrade_rule</a>，当第二次访问时便会出现 500 错误。</p>
<p data-nodeid="1736"><img src="https://s0.lgstatic.com/i/image6/M01/27/B4/CioPOWBdlfWASfRYAACZB-pUThM812.png" alt="Drawing 7.png" data-nodeid="1964"></p>
<div data-nodeid="1737"><p style="text-align:center">已触发熔断的错误提示</p></div>
<p data-nodeid="1738">在控制台日志也看到了 ERROR 日志，说明熔断已生效。</p>
<pre class="lang-java" data-nodeid="1739"><code data-language="java"><span class="hljs-number">2021</span>-<span class="hljs-number">01</span>-<span class="hljs-number">17</span> <span class="hljs-number">17</span>:<span class="hljs-number">19</span>:<span class="hljs-number">44.795</span> ERROR <span class="hljs-number">13244</span> --- [p-nio-<span class="hljs-number">80</span>-exec-<span class="hljs-number">3</span>] o.a.c.c.C.[.[.[/].[dispatcherServlet]    : Servlet.service() <span class="hljs-keyword">for</span> servlet [dispatcherServlet] in context with path [] threw exception [Request processing failed; nested exception is java.lang.reflect.UndeclaredThrowableException] with root cause
com.alibaba.csp.sentinel.slots.block.degrade.DegradeException: <span class="hljs-keyword">null</span>
</code></pre>
<p data-nodeid="1740">看到 DegradeException 就说明之前配置的熔断规则已经生效。</p>
<p data-nodeid="1741">讲到这，我们已经将 Sentinel 规则在 Nacos配置中心进行了持久化，并通过 Nacos 的推送机制做到新规则的实时更新，但在刚才的过程中，我们在触发流控或熔断后默认的错误提示是非常不友好的，在真正的项目中还需要对异常进行二次处理才能满足要求。</p>
<h3 data-nodeid="1742">开发友好的异常处理程序</h3>
<p data-nodeid="1743">对于 Sentinel 的异常处理程序要区分为两种情况：</p>
<ul data-nodeid="1744">
<li data-nodeid="1745">
<p data-nodeid="1746">针对 RESTful 接口的异常处理；</p>
</li>
<li data-nodeid="1747">
<p data-nodeid="1748">针对自定义资源点的异常处理。</p>
</li>
</ul>
<h4 data-nodeid="1749">针对 RESTful 接口的异常处理</h4>
<p data-nodeid="1750">默认情况下如果访问触发了流控规则，则会直接响应异常信息“BlockedbySentinel (flow limiting)”。</p>
<p data-nodeid="1751"><img src="https://s0.lgstatic.com/i/image6/M00/27/B7/Cgp9HWBdlgWAeKZlAAAq3uzq-uA090.png" alt="Drawing 8.png" data-nodeid="1976"></p>
<div data-nodeid="1752"><p style="text-align:center">触发流控的默认错误信息</p></div>
<p data-nodeid="1753">我们都知道，RESTful接口默认应返回 JSON 格式数据，如果直接返回纯文本，调用者将无法正确解析，所以咱们要对其进行封装提供更友好的 JSON 格式数据。</p>
<p data-nodeid="1754">针对 RESTful 接口的统一异常处理需要实现 BlockExceptionHandler，我们先给出完整代码。</p>
<pre class="lang-java" data-nodeid="1755"><code data-language="java"><span class="hljs-meta">@Component</span> <span class="hljs-comment">//Spring IOC实例化并管理该对象</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UrlBlockHandler</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">BlockExceptionHandler</span> </span>{
    <span class="hljs-comment">/**
     * RESTFul异常信息处理器
     * <span class="hljs-doctag">@param</span> httpServletRequest
     * <span class="hljs-doctag">@param</span> httpServletResponse
     * <span class="hljs-doctag">@param</span> e
     * <span class="hljs-doctag">@throws</span> Exception
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handle</span><span class="hljs-params">(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        String msg = <span class="hljs-keyword">null</span>;
        <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> FlowException){<span class="hljs-comment">//限流异常</span>
            msg = <span class="hljs-string">"接口已被限流"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> DegradeException){<span class="hljs-comment">//熔断异常</span>
            msg = <span class="hljs-string">"接口已被熔断,请稍后再试"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> ParamFlowException){ <span class="hljs-comment">//热点参数限流</span>
            msg = <span class="hljs-string">"热点参数限流"</span>; 
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> SystemBlockException){ <span class="hljs-comment">//系统规则异常</span>
            msg = <span class="hljs-string">"系统规则(负载/....不满足要求)"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> AuthorityException){ <span class="hljs-comment">//授权规则异常</span>
            msg = <span class="hljs-string">"授权规则不通过"</span>; 
        }
        httpServletResponse.setStatus(<span class="hljs-number">500</span>);
        httpServletResponse.setCharacterEncoding(<span class="hljs-string">"UTF-8"</span>);
        httpServletResponse.setContentType(<span class="hljs-string">"application/json;charset=utf-8"</span>);
        <span class="hljs-comment">//ObjectMapper是内置Jackson的序列化工具类,这用于将对象转为JSON字符串</span>
        ObjectMapper mapper = <span class="hljs-keyword">new</span> ObjectMapper();
        <span class="hljs-comment">//某个对象属性为null时不进行序列化输出</span>
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        mapper.writeValue(httpServletResponse.getWriter(),
                <span class="hljs-keyword">new</span> ResponseObject(e.getClass().getSimpleName(), msg)
        );
    }
}
</code></pre>
<p data-nodeid="1756">BlockExceptionHandler.handle() 方法第三个参数类型是 BlockException，它有五种子类代表不同类型的规则异常。</p>
<p data-nodeid="1757"><strong data-nodeid="1994">1</strong>. FlowException：流控规则异常。<br>
<strong data-nodeid="1995">2</strong>. DegradeException：熔断规则异常。<br>
<strong data-nodeid="1996">3</strong>. ParamFlowException：热点参数规则异常。</p>
<p data-nodeid="1758">例如：针对 id=5 的冷门商品编号时不开启限流，针对 id=10 的热门商品编号则需要进行限流，当 10 号商品被限流时抛出热点参数异常。</p>
<p data-nodeid="1759"><strong data-nodeid="2002">4</strong>.SystemBlockException：系统规则异常。</p>
<p data-nodeid="1760">例如：服务器 CPU 负载超过80%，抛出系统规则异常。</p>
<p data-nodeid="1761"><strong data-nodeid="2008">5</strong>. AuthorityException：授权规则异常。</p>
<p data-nodeid="1762">例如：某个 IP 被列入黑名单，该 IP 在访问时就会抛出授权规则异常。</p>
<p data-nodeid="1763">我们利用 instanceof关键字确定具体的规则异常后，便通过response 响应对象将封装好的 ResponseObject对象返回给应用前端，此时响应中 code 值不再为 0，而是对应的异常类型。</p>
<p data-nodeid="1764">例如，当 RESTful触发流控规则后，前端响应如下：</p>
<pre class="lang-json" data-nodeid="1765"><code data-language="json">{
    code:&nbsp;"FlowException",
    message:&nbsp;"接口已被限流"
}
</code></pre>
<p data-nodeid="1766">当触发熔断规则后，前端响应如下。</p>
<pre class="lang-json" data-nodeid="1767"><code data-language="json">{
    code:&nbsp;"DegradeException",
    message:&nbsp;"接口已被熔断,请稍后再试"
}
</code></pre>
<p data-nodeid="1768">通过这种统一而通用的异常处理机制，对RESTful 屏蔽了sentinel-core默认的错误文本，让项目采用统一的 JSON 规范进行输出。</p>
<p data-nodeid="1769">说完了 RESTful 的异常处理，咱们再来说一说自定义资源点如何控制异常信息。</p>
<h4 data-nodeid="1770">自定义资源点的异常处理</h4>
<p data-nodeid="1771">自定义资源点的异常处理与 RESTful 接口处理略有不同，我们需要在 @SentinelResource 注解上额外附加 blockHandler属性进行异常处理，这里先给出完整代码。</p>
<pre class="lang-java" data-nodeid="1772"><code data-language="java"><span class="hljs-comment">/**
 * 演示用的业务逻辑类
 */</span>
<span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SampleService</span> </span>{
    <span class="hljs-meta">@SentinelResource(value = "createOrder",blockHandler = "createOrderBlockHandler")</span>
    <span class="hljs-comment">/**
     * 模拟创建订单业务
     * 抛出 IllegalStateException 异常用于模拟业务逻辑执行失败的情况
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createOrder</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> IllegalStateException</span>{
        <span class="hljs-keyword">try</span> {
            <span class="hljs-comment">//模拟处理业务逻辑需要 101 毫秒</span>
            Thread.sleep(<span class="hljs-number">101</span>);
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(<span class="hljs-string">"订单已创建"</span>);
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createOrderBlockHandler</span><span class="hljs-params">(BlockException e)</span> <span class="hljs-keyword">throws</span> IllegalStateException</span>{
        String msg = <span class="hljs-keyword">null</span>;
        <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> FlowException){<span class="hljs-comment">//限流异常</span>
            msg = <span class="hljs-string">"资源已被限流"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> DegradeException){<span class="hljs-comment">//熔断异常</span>
            msg = <span class="hljs-string">"资源已被熔断,请稍后再试"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> ParamFlowException){ <span class="hljs-comment">//热点参数限流</span>
            msg = <span class="hljs-string">"热点参数限流"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> SystemBlockException){ <span class="hljs-comment">//系统规则异常</span>
            msg = <span class="hljs-string">"系统规则(负载/....不满足要求)"</span>;
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(e <span class="hljs-keyword">instanceof</span> AuthorityException){ <span class="hljs-comment">//授权规则异常</span>
            msg = <span class="hljs-string">"授权规则不通过"</span>;
        }
        <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(msg);
    }
}
</code></pre>
<p data-nodeid="1773">在这份代码中可以清楚地看到以下两点变化。</p>
<p data-nodeid="1774">第一，我们为 @SentinelResource 附加了 blockHandler 属性，这个属性指向 createOrderBlockHandler 方法名，它的作用是当 sentinel-core 触发规则异常后，自动执行 createOrderBlockHandler 方法进行异常处理。</p>
<p data-nodeid="1775">第二，createOrderBlockHandler 方法的书写有两个要求：</p>
<ul data-nodeid="11777">
<li data-nodeid="11778">
<p data-nodeid="11779">方法返回值、访问修饰符、抛出异常要与原始的 createOrder 方法完全相同。</p>
</li>
<li data-nodeid="11780">
<p data-nodeid="11781" class="">createOrderBlockHandler 方法名允许自定义，但最后一个参数必须是 BlockException 对象，这是所有规则异常的父类，通过判断 BlockException 我们就知道触发了哪种规则异常。</p>
</li>
</ul>









<p data-nodeid="1781">至于 createOrderBlockHandler 方法的代码和 RESTful 异常处理基本一致，先判断规则异常的种类再对外抛出 IllegalStateException异常。SampleController 会对 IllegalStateException 异常进行捕获，对外输出为 JSON 响应。</p>
<p data-nodeid="1782">当程序运行后，咱们看一下结果。</p>
<p data-nodeid="1783">访问 <a href="http://localhost/test_degrade_rule?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2031">http://localhost/test_degrade_rule</a> 当触发流控规则后，响应如下：</p>
<pre class="lang-json" data-nodeid="1784"><code data-language="json">{
    code:&nbsp;"IllegalStateException",
    message:&nbsp;"资源已被限流"
}
</code></pre>
<p data-nodeid="1785">当触发熔断规则后，响应如下：</p>
<pre class="lang-json" data-nodeid="1786"><code data-language="json">{
    code:&nbsp;"IllegalStateException",
    message:&nbsp;"资源已被熔断,请稍后再试"
}
</code></pre>
<p data-nodeid="1787">讲到这里，我们针对在生产实践中 Sentinel 必然会涉及的应用场景进行了讲解，下面咱们进行总结。</p>
<h3 data-nodeid="1788">小结与预告</h3>
<p data-nodeid="1789">本讲咱们学习了三方面内容，首先通过流控案例说明如何利用 Nacos 配置中心将 Sentinel 规则持久化存储，并利用 Nacos 的推送功能实现新规则的实时更新；其次咱们通过熔断规则学习了 @SentinelResource 注解的用法，同时将自定义资源点的熔断规则放入 Nacos 进行持久化；最后针对流控熔断中默认非常不友好的异常输出，咱们利用Sentinel 自带的机制逐一进行的优化，生成了符合项目要求的友好的 JSON 格式数据。</p>
<p data-nodeid="1790">在这里我为你留一道自习题：Sentinel 除了流控与熔断外，还有三种不常用的规则：</p>
<ul data-nodeid="1791">
<li data-nodeid="1792">
<p data-nodeid="1793">热点参数流控；</p>
</li>
<li data-nodeid="1794">
<p data-nodeid="1795">系统规则；</p>
</li>
<li data-nodeid="1796">
<p data-nodeid="1797">授权规则（黑白名单）。</p>
</li>
</ul>
<p data-nodeid="1798">请你自行前往 Sentinel 官网文档<a href="https://github.com/alibaba/Sentinel/wiki?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2044">https://github.com/alibaba/Sentinel/wiki</a> 进行学习。</p>
<p data-nodeid="1799" class="">到这里关于 Sentinel 的话题咱们告一段落。下一讲开始进入新的篇章，学习微服务架构下的链路追踪技术。</p>

---

### 精选评论

##### *华：
> sentinel配置完成，能直接同步到nacos吗？每次都用json配置感觉很麻烦

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个默认是单向的,但可以通过程序扩展,通过自定义Publisher就可以啦

##### **雄：
> 可以说下 这种底层实现原理 和同类型产品的对比吗 场景选择之类的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一般Sentinel都拿来和Hystrix进行对比, Sentinel更像拦截器,在请求送达前利用AOP实现方法的前置拦截.
而Hystrix则是嵌入到线程中,深度与Spring Cloud微服务线程执行过程进行融合. Hystrix执行效率确实更高,Sentinel使用更简单,功能更强大.

##### *双：
> 请问，Sentinel如何做类似Hystrix的降级功能，比如FallbackFactory

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Sentinel没有Hystrix这样的实现机制,毕竟底层实现系统保护的原理不同

