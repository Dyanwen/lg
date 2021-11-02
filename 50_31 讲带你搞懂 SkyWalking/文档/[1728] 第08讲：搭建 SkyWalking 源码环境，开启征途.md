<p style="text-align: justify; line-height: 1.75em;"><span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在第一课时中，我们已经成功安装并运行了 SkyWalking 环境，本课时将带你完成 SkyWalking 源码环境的搭建 ，并在 IDEA 中尝试调试 SkyWalking Agent。</span><br></p>
<h1><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">搭建 SkyWalking 源码环境</span></p></h1>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong style="font-size: 12pt;">下载 SkyWalking 源码</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">执行 git clone 命令从 GitHub下载 SkyWalking 源码，如下所示 ：</span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<pre>git&nbsp;clone&nbsp;git@github.com:apache/skywalking.git</pre>
<p><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>切换分支</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">等待 clone 完成之后，我们通过命令行窗口进入 SkyWalking 源码根目录，执行如下命令：</span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<pre>git&nbsp;checkout&nbsp;-b&nbsp;6.2.0&nbsp;v6.2.0</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"> &nbsp; &nbsp; &nbsp;切换到 v6.2.0 tag 的源码，后续源码分析过程都是基于 6.2.0 版本进行分析的。</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>导入 IDEA</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">在 IDEA 中点击“Import Project”，选择 SkyWalking 源码目录导入 IDEA 中。SkyWalking 是一个 Maven，在导入过程中会下载相关的依赖 jar 包，过程可能会比较慢，需要你耐心等待。</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong style="font-size: 12pt;">更新 submodule</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">全部 Maven 依赖下载完成后，在 SkyWalking 源码根目录中执行如下两条命令，更新 submodule：</span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<pre>git&nbsp;submodule&nbsp;init
git&nbsp;submodule&nbsp;update</pre>
<p><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>打包</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">上述操作执行完毕之后，执行如下命令，开始打包：</span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<pre>mvn&nbsp;clean&nbsp;package&nbsp;-DskipTests&nbsp;-Dcheckstyle.skip</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>标记 Generated Source Code 目录</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">在打包过程中，会自动生成一些代码，需要我们将其目录设置为 Generated Source Codes，这样 IDEA 才能识别这些代码，生成代码主要来源于以下两种方式：</span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">SkyWalking Agent 与后端 OAP 之间通信用了 gRPC，其中的 proto 文件会生成一些 Java 代码。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">SkyWalking OAP 中定义了 OAL 语言，打包过程中会生成一些 Java 代码。</span></p></li>
 </ul>
</blockquote>
<blockquote>
 <p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">具体的标记方式如下下图所示：</span></p>
</blockquote>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGjyAQXgGAAeBF-jQ5aw073.png"></span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">需要标记的目录有：</span></p>
<ul>
 <ul style="list-style-type: square;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">apm-protocol/apm-network/target/generated-sources/protobuf 路径下的 grpc-java 目录和 java 目录。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">oap-server/server-core/target/generated-sources/protobuf 路径下的 grpc-java 目录和 java 目录。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">oap-server/server-receiver-plugin/receiver-proto/target/generated-sources/protobuf 路径下的 grpc-java 目录和 java 目录。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">oap-server/exporter/target/generated-sources/protobuf 路径下的 grpc-java 目录和 java 目录。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">oap-server/server-configuration/grpc-configuration-sync/target/generated-sources/protobuf 路径下的 grpc-java 目录和 java 目录。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">/Users/xxx/SW/skywalking/oap-server/generated-analysis/target/generated-sources 路径下的 oal 目录。</span></p></li>
 </ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>安装 ElasticSearch</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">前文已经完成了 ElasticSearch 的安装，这里不再展开。</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>启动 OAP</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">在 IDEA 中，找到 oap-server 模块中 OAPServerStartUp 这个类，右键执行 main() 方法即可。启动过程中无异常日志，并看到如下信息，即表示 OAP 启动成功：</span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<pre>...&nbsp;...&nbsp;//&nbsp;省略其他日志
Server&nbsp;started,&nbsp;host&nbsp;0.0.0.0&nbsp;listening&nbsp;on&nbsp;11800</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>启动 SkyWalking Rocketbot</strong> </span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">在 IDEA 中，找到 apm-webapp 模块，这是 Spring Boot 的 Web项目，执行 ApplicationStartUp&nbsp; 中的 main() 方法。正常启动之后，访问 localhost:8080，看到 SkyWalking Rocketbot 的 UI 界面即为启动成功。</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>启动 demo-webapp 和 demo-provider</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">为了验证后端的 OAP 以及前面打包生成的 SkyWalking Agent 是否可用，这里需要启动 demo-webapp 和 demo-provider 两个示例 demo。</span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">首先将整个 skywalking-demo 项目移动到与 SkyWalking 源码项目同一级目录，并导入 IDEA 中，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj2AHgB8AAAjQ0VaZ8E159.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">然后修改 demo-provider 和 demo-webapp 模块的 VM options 参数，将其中 -javaagent: 命令指向的 skywalking-agent.jar 换成 SkyWalking 源码项目中的 skywalking-agent.jar，具体路径如下所示：</span></p></li>
 </ul>
</blockquote>
<p><br></p>
<p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"></span></p>
<pre>SkyWalking源码目录/skywalking-agent/skywalking-agent.jar</pre>
<p><br></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">其他配置无需修改，依次启动 Zookeeper、demo-provider、demo-webapp。启动成功后访问 </span><a class="ql-link ql-author-23067508" href="http://localhost:8000/hello/xxx" target="_blank" rel="noopener noreferrer nofollow" style="text-decoration: underline; color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">http://localhost:8000/hello/xxx</span></a><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">待请求正常响应后，在上一步启动的 SkyWalking Rocketbot 中可以查询到相应的 Trace 信息以及两个项目相关的 Metrics 监控信息，即表示整个源码环境搭建完成。</span></p></li>
 </ul>
</blockquote>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>Debug SkyWalking 源码</strong></span></p></li>
</ul>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">按照上述方式成功搭建 SkyWalking 源码环境之后，我们尝试 Debug SkyWalking 源码。</span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">首先在 SkyWalking 源码项目中找到 SkyWalkingAgent.java 这个类（位于 apm-sniffer 模块下的 apm-agent 子模块中），该类是 SkyWalking Agent 的入口，提供了 premain() 方法实现，我们可以在其中打一个断点，然后以 Debug 模式重启 demo-webapp，此时 demo-webapp 会停在该断点处，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="margin-left: 26px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj2AHzBqAAGUjaX3Mcg617.png"></span></p>
<h1><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">SkyWalking源码结构</span></p></h1>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">完成 SkyWalking 源码环境的搭建以及 Debug 的测试之后，我们回到 SkyWalking 源码项目，简单介绍一下 SkyWalking 源码中各模块的基本功能。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">SkyWalking 源码的整体结构如下图所示：</span></p>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj2AZaUkAABKcoi4_5c009.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-application-toolkit 模块：</strong>SkyWalking 提供给用户调用的工具箱。该模块提供了对 log4j、log4j2、logback 等常见日志框架的接入接口，提供了 @Trace 注解等。apm-application-toolkit<strong> </strong>模块类似于暴露 API 定义，对应的处理逻辑在 apm-sniffer/apm-toolkit-activation 模块中实现，如下图所示：</span></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj2AYS6GAABWyHp4-sA345.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-commons 模块：</strong>SkyWalking 的公共组件和工具类。如下图所示，其中包含两个子模块，apm-datacarrier 模块提供了一个生产者-消费者模式的缓存组件（DataCarrier），无论是在 Agent 端还是 OAP 端都依赖该组件。apm-util 模块则提供了一些常用的工具类，例如，字符串处理工具类（StringUtil）、占位符处理的工具类（PropertyPlaceholderHelper、PlaceholderConfigurerSupport）等等。</span></p></li>
</ul>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj2AKJXJAAALnVoaWe4943.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apache-skywalking-apm 目录</strong><strong>：</strong>SkyWalking 打包后使用的命令文件都在此目录中，例如，前文启动 OAP 和 SkyWalking Rocketbot 使用的 startup.sh 文件。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-protocol 模块：</strong>该模块中只有一个 apm-network 模块，我们需要关注的是其中定义的 .proto 文件，定义 Agent 与后端 OAP 使用 gRPC 交互时的协议。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-sniffer 模块：</strong>apm-protocol 模块中有 4 个子模块，如下图所示：</span></p></li>
</ul>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj2AeTteAAATxoHTmQg700.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-agent 模块：</strong>其中包含了刚才使用的 SkyWalkingAgent 这个类，是整个 Agent 的入口。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-agent-core 模块</strong>：SkyWalking Agent 的核心实现都在该模块中，也是本课程第二部分重点分析的模块之一。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-sdk-plugin 模块</strong>：SkyWalking Agent 使用了微内核+插件的架构，该模块下包含了 SkyWalking Agent 的全部插件，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj2AZUcYAABRh7Zo8YU356.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-toolkit-activation 模块：</strong>apm-application-toolkit 模块的具体实现，不再赘述。</span></p></li>
 </ul>
</blockquote>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>apm-webapp 模块：</strong>SkyWalking Rocketbot 对应的后端。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>oap-server 模块</strong>：SkyWalking OAP 的全部实现都在 oap-server 模块，其中包含了多个子模块，如下图所示：</span></p></li>
</ul>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj2AXOXhAAAsDflYjvk001.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>exporter 模块</strong>：负责导出数据。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>generate-tool、</strong><strong>generate-tool-grammar、generated-analysis 三个模块</strong>：与 SkyWalking 自定义的 OAL 语言有关，后面的课时将对 OAL 进行详细介绍。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-alarm-plugin 模块</strong>：负责实现 SkyWalking 的告警功能。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-cluster-pulgin 模块</strong>：负责 OAP 的集群信息管理，其中提供了接入多种第三方组件的相关插件，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj2Ad7RDAAA3iUNIOfA774.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-configuration 模块</strong>：负责管理 OAP 的配置信息，也提供了接入多种配置管理组件的相关插件，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj2AF3cKAAAzFmh2Hhw354.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-core模块</strong>：SkyWalking OAP 的核心实现都在该模块中。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-library 模块</strong>：OAP 以及 OAP 各个插件依赖的公共模块，其中提供了双队列 Buffer、请求远端的 Client 等工具类，这些模块都是对立于 SkyWalking OAP 体系之外的类库，我们可以直接拿走使用。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-query-plugin 模块</strong>：SkyWalking Rocketbot 发送的请求首先由该模块接收处理，目前该模块只支持 GraphQL 查询。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-receiver-plugin 模块</strong>：SkyWalking Agent 发送来的 Metrics、Trace 以及 Register 等写入请求都是首先由该模块接收处理的，不仅如此，该模块还提供了多种接收其他格式写入请求的插件，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/7F/D0/Cgq2xl6DGj6Ab69uAAB3rQXuurE063.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<blockquote>
 <ul style="list-style-type: circle;">
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-starter 模块</strong>：OAP 服务启动的入口。</span></p></li>
  <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>server-storage-plugin 模块：</strong>OAP 服务底层可以使用多种存储来保存 Metrics 数据以及Trace 数据，该模块中包含了接入相关存储的插件，如下图所示：</span></p></li>
 </ul>
</blockquote>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><img src="https://s0.lgstatic.com/i/image3/M01/06/BA/Ciqah16DGj6ADeNJAABfQxiOcoo599.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>skywalking-agent 目录</strong>：SkyWalking Agent 编译后生成的 jar 包都会放到该目录中。</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><strong>skywalking-ui 目录</strong>：SkyWalking Rocketbot 的前端。</span></p></li>
</ul>
<h1><p><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">总结</span></p></h1>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">本课时重点介绍了 SkyWalking 源码环境的搭建流程，并在搭建完成之后，启动 skywalking-demo 项目进行了简单的测试。之后深入介绍了 SkyWalking 源码中各个模块的核心功能，了解各模块的主要功能可以让你对后续的源码分析更加游刃有余。</span></p>

---

### 精选评论

##### **强：
> /Users/xxx/SW/skywalking/oap-server/generated-analysis/target/generated-sources 路径下的 oal 目录。<div>以上课程中描述错误了、应该是使用作者的电脑路径了。直接使用</div><div>skywalking/oap-server/generated-analysis/target/generated-sources 路径下的 oal 目录。<br></div><div>就可以了。</div><div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课程里面写的是个绝对路径，你描述的是个相对路径，其实是一回事

##### **强：
> 源码模块讲解非常清晰明了，受用了！

##### **桂：
> 写的很精彩，娓娓道来！

##### *振：
> sharding-sphere-4.x-plugin ......................... FAILURE [ 1.984 s]

