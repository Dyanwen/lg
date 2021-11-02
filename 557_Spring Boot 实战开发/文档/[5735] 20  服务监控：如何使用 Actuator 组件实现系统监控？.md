<p data-nodeid="2394">这一讲我们将介绍 Spring Boot 中一个非常有特色的主题——系统监控。</p>
<p data-nodeid="2395">系统监控是 Spring Boot 中引入的一项全新功能，它对应用程序运行状态的管理非常有效。而 Spring Boot Actuator 组件主要通过一系列 HTTP 端点提供的系统监控功能来实现系统监控。因此，接下来我们将引入 Spring Boot Actuator 组件，介绍如何使用它进行系统监控，以及如何对 Actuator 端点进行扩展。</p>
<h3 data-nodeid="2396">引入 Spring Boot Actuator 组件</h3>
<p data-nodeid="2397">在初始化 Spring Boot 系统监控功能之前，首先我们需要引入 Spring Boot Actuator 组件，具体操作为在 pom 中添加如下所示的 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="2398"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-actuator<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="2399">请注意，引入 Spring Boot Actuator 组件后，并不是所有的端点都对外暴露。例如，启动 customer-service 时，我们就可以在启动日志中发现如下所示内容：</p>
<pre class="lang-xml" data-nodeid="2400"><code data-language="xml">Exposing 2 endpoint(s) beneath base path '/actuator'
</code></pre>
<p data-nodeid="7168" class="">访问 <a href="http://localhost:8080/actuator" data-nodeid="7172">http://localhost:8080/actuator</a> 端点后，我们也会得到如下所示结果。</p>





<pre class="lang-xml" data-nodeid="2402"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"_links":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"self":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"health-path":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/health/{*path}",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":true
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"health":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/health",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"info":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/info",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;}
 }
</code></pre>
<p data-nodeid="2403">这种结果就是 HATEOAS 风格的 HTTP 响应。如果我们想看到默认情况下看不到的所有端点，则需要在配置文件中添加如下所示的配置信息。</p>
<pre class="lang-xml" data-nodeid="2404"><code data-language="xml">management:
&nbsp; endpoints:
&nbsp;&nbsp;&nbsp; web:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; exposure:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; include: "*"&nbsp; 
</code></pre>
<p data-nodeid="2405">重启应用后，我们就能获取到 Spring Boot Actuator 暴露的所有端点，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2406"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"_links":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"self":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"beans":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/beans",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"health":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/health",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"health-path":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/health/{*path}",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":true
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"info":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/info",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"conditions":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/conditions",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"configprops":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/configprops",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"env":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/env",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"env-toMatch":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/env/{toMatch}",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":true
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"loggers":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/loggers",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"loggers-name":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/loggers/{name}",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":true
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"heapdump":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/heapdump",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"threaddump":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/threaddump",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"metrics-requiredMetricName":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/metrics/{requiredMetricName}",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":true
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"metrics":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/metrics",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"scheduledtasks":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/scheduledtasks",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"mappings":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"href":"http://localhost:8080/actuator/mappings",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"templated":false
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;}
 }
</code></pre>
<p data-nodeid="2407">根据端点所起到的作用，我们把 Spring Boot Actuator 提供的原生端点分为如下三类。</p>
<ul data-nodeid="9411">
<li data-nodeid="9412">
<p data-nodeid="9413"><strong data-nodeid="9422">应用配置类：</strong> 主要用来获取应用程序中加载的应用配置、环境变量、自动化配置报告等配置类信息，它们与 Spring Boot 应用密切相关。</p>
</li>
<li data-nodeid="9414">
<p data-nodeid="9415"><strong data-nodeid="9427">度量指标类：</strong> 主要用来获取应用程序运行过程中用于监控的度量指标，比如内存信息、线程池信息、HTTP 请求统计等。</p>
</li>
<li data-nodeid="9416">
<p data-nodeid="9417" class=""><strong data-nodeid="9432">操作控制类：</strong> 在原生端点中只提供了一个关闭应用的端点，即 /shutdown 端点。</p>
</li>
</ul>



<p data-nodeid="2415">根据 Spring Boot Actuator 默认提供的端点列表，我们将部分常见端点的类型、路径和描述梳理在如下表格中，仅供参考。</p>
<p data-nodeid="10169" class=""><img src="https://s0.lgstatic.com/i/image2/M01/08/30/Cip5yGAKfl6Af_yWAAIDoRxLU2E765.png" alt="Drawing 0.png" data-nodeid="10172"></p>


<p data-nodeid="2468">通过访问上表中的各个端点，我们就可以获取自己感兴趣的监控信息了。例如访问了<a href="http://localhost:8082/actuator/health" data-nodeid="2642">http://localhost:8082/actuator/health</a>端点，我们就可以得到如下所示的 account-service 基本状态。</p>
<pre class="lang-xml" data-nodeid="2469"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"status":"UP"
 }
</code></pre>
<p data-nodeid="2470">此时，我们看到这个健康状态信息非常简单。</p>
<p data-nodeid="2471">那有没有什么办法可以获取更详细的状态信息呢？答案是：有，而且办法很简单，我们只需要在配置文件中添加如下所示的配置项即可。</p>
<pre class="lang-xml" data-nodeid="2472"><code data-language="xml">management: 
&nbsp; endpoint:
&nbsp;&nbsp;&nbsp; health:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; show-details: always
</code></pre>
<p data-nodeid="11895" class="">上述配置项指定了针对 health 端点需要显示它的详细信息。这时，如果我们重启 Spring Boot 应用程序，并重新访问 <a href="http://localhost:8082/actuator/health" data-nodeid="11899">http://localhost:8082/actuator/health</a> 端点，就可以获取如下所示的详细信息。</p>



<pre class="lang-xml" data-nodeid="2474"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"status":"UP",
 &nbsp;&nbsp;&nbsp;&nbsp;"components":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"diskSpace":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"status":"UP",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"details":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"total":201649549312,
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"free":3434250240,
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"threshold":10485760
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"ping":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"status":"UP"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;}
 }
</code></pre>
<p data-nodeid="2475">如果 Spring Boot Actuator 默认提供的端点信息不能满足业务需求，我们可以对其进行修改和扩展。此时，常见实现方案有两种，一种是扩展现有的监控端点，另一种是自定义新的监控端点。这两种方案我们都会逐一介绍，不过这一讲先来关注如何在现有的监控端点上添加定制化功能。</p>
<h3 data-nodeid="2476">扩展 Actuator 端点</h3>
<p data-nodeid="2477">前面我们介绍了 Spring Boot 默认暴露了日常开发中最常见的两个端点：Info 端点和 Health 端点。接下来，我们讨论下如何对这两个端点进行扩展。</p>
<h4 data-nodeid="2478">扩展 Info 端点</h4>
<p data-nodeid="14211" class="">Info 端点用于暴露 Spring Boot 应用的自身信息。在 Spring Boot 内部，它把这部分工作委托给了一系列 <a href="https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java" data-nodeid="14215">InfoContributor</a> 对象，而 Info 端点会暴露所有 <a href="https://github.com/spring-projects/spring-boot/tree/v1.4.1.RELEASE/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/info/InfoContributor.java" data-nodeid="14219">InfoContributor</a> 对象所收集的各种信息。</p>




<p data-nodeid="2480">在Spring Boot 中包含了很多自动配置的 InfoContributor 对象，常见的 InfoContributor 及其描述如下表所示：</p>
<p data-nodeid="14791" class=""><img src="https://s0.lgstatic.com/i/image/M00/90/47/CgqCHmAKfoOARrjaAADoOGMdQb4610.png" alt="Drawing 1.png" data-nodeid="14794"></p>


<p data-nodeid="2501">以上表中的 EnvironmentInfoContributor 为例，通过在配置文件中添加格式以“info”作为前缀的配置段，我们就可以定义 Info 端点暴露的数据。添加完成后，我们将看到所有在“info”配置段下的属性都将被自动暴露。</p>
<p data-nodeid="2502">比如你可以将如下所示配置信息添加到配置文件 application.yml 中：</p>
<pre class="lang-xml" data-nodeid="2503"><code data-language="xml">info:
	app:
	&nbsp;&nbsp;&nbsp; encoding: UTF-8
	&nbsp;&nbsp;&nbsp; java:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; source: 1.8.0_31
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; target: 1.8.0_31
</code></pre>
<p data-nodeid="2504">现在访问 Info 端点，我们就能得到如下的 Environment 信息。</p>
<pre class="lang-xml" data-nodeid="2505"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp; "app":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "encoding":"UTF-8",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "java":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "source":"1.8.0_31",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "target":"1.8.0_31"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
 &nbsp;&nbsp;&nbsp; }
 }
</code></pre>
<p data-nodeid="2506">同时，我们还可以在服务构建时扩展 Info 属性，而不是硬编码这些值。假设使用 Maven，我们就可以按照如下所示的配置重写前面的示例并得到同样的效果。</p>
<pre class="lang-xml" data-nodeid="2507"><code data-language="xml">info: 
	app:
	&nbsp;&nbsp;&nbsp; encoding: @project.build.sourceEncoding@
	&nbsp;&nbsp;&nbsp; java:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; source: @java.version@
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; target: @java.version@
</code></pre>
<p data-nodeid="2508">很多时候，Spring Boot 自身提供的 Info 端点并不能满足我们的业务需求，这就需要我们编写一个自定义的 InfoContributor 对象。</p>
<p data-nodeid="2509">方法也很简单，我们直接实现 InfoContributor 接口的 contribute() 方法即可。例如，我们希望在 Info 端点中暴露该应用的构建时间，就可以采用如下所示的代码进行操作。</p>
<pre class="lang-java" data-nodeid="2510"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomBuildInfoContributor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">InfoContributor</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">contribute</span><span class="hljs-params">(Builder builder)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; builder.withDetail(<span class="hljs-string">"build"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Collections.singletonMap(<span class="hljs-string">"timestamp"</span>, <span class="hljs-keyword">new</span> Date())); 
  }
}
</code></pre>
<p data-nodeid="2511">重新构建应用并访问 Info 端口后，我们就能获取如下所示信息。</p>
<pre class="lang-xml" data-nodeid="2512"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp; "app":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "encoding":"UTF-8",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "java":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "source":"1.8.0_31",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "target":"1.8.0_31"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
 &nbsp;&nbsp;&nbsp; },
 &nbsp;&nbsp;&nbsp; "build":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "timestamp":1604307503710
 &nbsp;&nbsp;&nbsp; }
 }
</code></pre>
<p data-nodeid="2513">这里我们可以看到，CustomBuildInfoContributor 为 Info 端口新增了时间属性。</p>
<h4 data-nodeid="2514">扩展 Health 端点</h4>
<p data-nodeid="2515">Health 端点用于检查正在运行的应用程序健康状态，而健康状态信息由 HealthIndicator 对象从 Spring 的 ApplicationContext 中获取。</p>
<p data-nodeid="2516">和 Info 端点一样，Spring Boot 内部也提供了一系列 HealthIndicator 对象供我们实现定制化。在默认情况下，HealthAggregator 会根据 HealthIndicator 的有序列表对每个状态进行排序，从而得到最终的系统状态。</p>
<p data-nodeid="2517">常见的 HealthIndicator 如下表所示：</p>
<table data-nodeid="15300">
<thead data-nodeid="15301">
<tr data-nodeid="15302">
<th data-nodeid="15304"><strong data-nodeid="15339">HealthIndicator 名称</strong></th>
<th data-nodeid="15305"><strong data-nodeid="15343">描述</strong></th>
</tr>
</thead>
<tbody data-nodeid="15308">
<tr data-nodeid="15309">
<td data-nodeid="15310"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/system/DiskSpaceHealthIndicator.java" data-nodeid="15346">DiskSpaceHealthIndicator</a></td>
<td data-nodeid="15311">检查磁盘空间是否足够</td>
</tr>
<tr data-nodeid="15312">
<td data-nodeid="15313"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jdbc/DataSourceHealthIndicator.java" data-nodeid="15350">DataSourceHealthIndicator</a></td>
<td data-nodeid="15314">检查是否可以获得连接 DataSource</td>
</tr>
<tr data-nodeid="15315">
<td data-nodeid="15316"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/elasticsearch/ElasticsearchHealthIndicator.java" data-nodeid="15354">ElasticsearchHealthIndicator</a></td>
<td data-nodeid="15317">检查 Elasticsearch 集群是否启动</td>
</tr>
<tr data-nodeid="15318">
<td data-nodeid="15319"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/jms/JmsHealthIndicator.java" data-nodeid="15358">JmsHealthIndicator</a></td>
<td data-nodeid="15320">检查 JMS 代理是否启动</td>
</tr>
<tr data-nodeid="15321">
<td data-nodeid="15322"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mail/MailHealthIndicator.java" data-nodeid="15362">MailHealthIndicator</a></td>
<td data-nodeid="15323">检查邮件服务器是否启动</td>
</tr>
<tr data-nodeid="15324">
<td data-nodeid="15325"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/mongo/MongoHealthIndicator.java" data-nodeid="15366">MongoHealthIndicator</a></td>
<td data-nodeid="15326">检查 Mongo 数据库是否启动</td>
</tr>
<tr data-nodeid="15327">
<td data-nodeid="15328"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/amqp/RabbitHealthIndicator.java" data-nodeid="15370">RabbitHealthIndicator</a></td>
<td data-nodeid="15329">检查 RabbitMQ 服务器是否启动</td>
</tr>
<tr data-nodeid="15330">
<td data-nodeid="15331"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/redis/RedisHealthIndicator.java" data-nodeid="15374">RedisHealthIndicator</a></td>
<td data-nodeid="15332">检查 Redis 服务器是否启动</td>
</tr>
<tr data-nodeid="15333">
<td data-nodeid="15334"><a href="https://github.com/spring-projects/spring-boot/tree/v2.0.1.RELEASE/spring-boot-project/spring-boot-actuator/src/main/java/org/springframework/boot/actuate/solr/SolrHealthIndicator.java" data-nodeid="15378">SolrHealthIndicator</a></td>
<td data-nodeid="15335">检查 Solr 服务器是否已启动</td>
</tr>
</tbody>
</table>


<p data-nodeid="2556">Health 端点信息的丰富程度取决于当下应用程序所处的环境，而一个真实的 Health 端点信息如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2557"><code data-language="xml">{
 &nbsp;&nbsp;&nbsp;&nbsp;"status":"UP",
 &nbsp;&nbsp;&nbsp;&nbsp;"components":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"db":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"status":"UP",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"details":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"database":"MySQL",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"result":1,
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"validationQuery":"/*&nbsp;ping&nbsp;*/&nbsp;SELECT&nbsp;1"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"diskSpace":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"status":"UP",
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"details":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"total":201649549312,
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"free":3491287040,
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"threshold":10485760
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"ping":{
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"status":"UP"
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
	}
}
</code></pre>
<p data-nodeid="2558">通过以上这些信息，我们就可以判断该环境中是否包含了 MySQL 数据库。</p>
<p data-nodeid="2559">现在，我们还想在 Health 端点中暴露 customer-service 当前运行时状态。</p>
<p data-nodeid="2560">为了进一步明确该服务的状态，我们可以自定义一个 CustomerServiceHealthIndicator 端点专门展示 customer-service 的状态信息，CustomerServiceHealthIndicator 的定义如下所示：</p>
<pre class="lang-java" data-nodeid="2561"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomerServiceHealthIndicator</span> <span class="hljs-keyword">implements</span> 
	<span class="hljs-title">HealthIndicator</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Health <span class="hljs-title">health</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
	URL url = <span class="hljs-keyword">new</span> 
	URL(<span class="hljs-string">"http://localhost:8083/health/"</span>);
	HttpURLConnection conn = (HttpURLConnection) 
	url.openConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">int</span> statusCode = conn.getResponseCode();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (statusCode &gt;= <span class="hljs-number">200</span> &amp;&amp; statusCode &lt; <span class="hljs-number">300</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Health.up().build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Health.down().withDetail(<span class="hljs-string">"HTTP Status Code"</span>, statusCode).build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (IOException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Health.down(e).build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2562">我们需要提供 health() 方法的具体实现并返回一个 Health 结果。该 Health 结果应该包括一个状态，并且可以根据需要添加任何细节信息。</p>
<p data-nodeid="2563">以上代码中，我们使用了一种简单且直接的方式判断配置中心服务“customerservice”是否正在运行。然后我们构建一个 HTTP 请求，并根据 HTTP 响应得出了健康诊断的结论。</p>
<p data-nodeid="2564">如果 HTTP 响应的状态码处于 200~300 之间，我们认为该服务正在运行，此时，Health.up().build() 方法就会返回一种 Up 响应，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2565"><code data-language="xml">{
&nbsp;&nbsp;&nbsp; "status": "UP",
&nbsp;&nbsp;&nbsp; "details": {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "customerservice":{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "status": "UP"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2566">如果状态码不处于这个区间（例如返回 404，代表服务不可用），Health.down().withDetail().build() 方法就会返回一个 Down 响应，并给出具体的状态码，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2567"><code data-language="xml">{
&nbsp;&nbsp;&nbsp; "status": "DOWN",
&nbsp;&nbsp;&nbsp; "details": {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "customerservice":{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "status": "DOWN",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "details": {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "HTTP Status Code": "404"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2568">如果 HTTP 请求直接抛出了异常，Health.down().build() 方法同样会返回一个 Down 响应，并返回异常信息，效果如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2569"><code data-language="xml">{
&nbsp;&nbsp;&nbsp; "status": "DOWN",
&nbsp;&nbsp;&nbsp; "details": {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "customerservice":{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "status": "DOWN",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "details": {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; "error": "java.net.ConnectException: Connection refused: connect"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; },
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2570">显然，通过扩展 Health 端点为我们实时监控系统中各个服务的正常运行状态提供了很好的支持，我们也可以根据需要构建一系列有用的 HealthIndicator 实现类，并添加报警等监控手段。</p>
<h3 data-nodeid="2571">小结与预告</h3>
<p data-nodeid="2572">Spring Boot 内置的 Actuator 组件使得开发人员在管理应用程序运行的状态有了更加直接且高效的手段。</p>
<p data-nodeid="2573">这一讲，我们引入了 Actuator 组件并介绍了该组件提供的一系列核心端点，同时重点分析了 Info 和 Health 这两个基础端点，并给出了对它们进行扩展的系统方法。</p>
<p data-nodeid="2574">系统监控的一大目标是收集和分析系统运行时的度量指标，并基于这些指标判断当前的运行时状态，因此，21 讲我们将讨论如何在系统中嵌入自定义度量指标的实现技巧。</p>
<p data-nodeid="2575">这里给你留一道思考题：在使用 Spring Boot 时，如何实现自定义的健康监测功能？欢迎你在留言区与我互动、交流。</p>
<p data-nodeid="2576">另外，如果你觉得本专栏有价值，欢迎分享给更多好友看到哦~</p>

---

### 精选评论


