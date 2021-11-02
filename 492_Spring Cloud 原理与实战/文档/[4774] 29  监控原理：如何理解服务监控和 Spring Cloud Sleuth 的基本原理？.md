<p data-nodeid="31467" class="">从本课时开始，我们将讨论一个在微服务架构中非常重要的话题，即服务监控。今天我们将简要分析服务监控的基本原理，这是理解服务监控相关工具和框架的基础。同时，作为 Spring Cloud 中用于实现服务监控的专用工具，Spring Cloud Sleuth 为实现这些基本原理提供了完整而强大的解决方案。</p>
<h3 data-nodeid="31468">服务监控基本原理</h3>
<p data-nodeid="31469">在微服务架构中，我们基于业务划分服务并对外暴露服务访问接口。试想这样一个场景，如果我们发现某一个业务接口在访问过程中发生了错误，一般的处理过程就是快速定位到问题所发生的服务并进行解决。但在如下所示中大型系统中，一个业务接口背后可能会调用一批其他业务体系中的业务接口或基础设施类的底层接口，这时候我们如何能够做到快速定位问题呢？</p>
<p data-nodeid="31470"><img src="https://s0.lgstatic.com/i/image2/M01/03/BC/CgpVE1_hnXCAXNCHAABJ4_O33aw538.png" alt="Drawing 0.png" data-nodeid="31520"></p>
<div data-nodeid="31471"><p style="text-align:center">微服务调用链路示意图</p></div>
<p data-nodeid="31472">传统的做法是通过查阅服务器的日志来定位问题，但在中大型系统中，这种做法可操作性并不强，主要原因是我们很难找到包含错误日志的那台服务器。一方面，开发人员可能都不知道整个服务调用链路中具体有几个服务，也就无法找到是哪个服务发生了错误。就算找到了目标服务，在分布式集群的环境下，我们也不建议直接通过访问某台服务器来定位问题。服务监控的需求就应运而生。</p>
<p data-nodeid="31473">分布式服务跟踪和监控的运行原理上实际上并不复杂，我们首先需要引入两个基本概念，即 SpanId 和 TraceId。</p>
<h4 data-nodeid="31474">SpanId</h4>
<p data-nodeid="31475">SpanId 一般被称为跨度 Id。在上图中，针对服务 A 的访问请求，通过 SpanId 来标识该请求的到达并返回的具体过程。显然，对于这个 Span 而言，势必需要明确 Span 的开始时间和结束时间，这两个时间之间的差值就是服务 A 对这个请求的处理时间。</p>
<h4 data-nodeid="31476">TraceId</h4>
<p data-nodeid="31477">除了 SpanId 外，我们还需要 TraceId，也就是跟踪 Id。同样是在上图中，要想监控整个链路，我们不光需要关注服务 A 中的 Span，而是需要把请求通过所有服务的 Span 都串联起来。这时候就需要为这个请求生成一个全局的唯一性 Id，通过这个 Id 可以串联起如上图所示，从服务 A 到服务 F 的整个调用，这个唯一性 Id 就是 TraceId。</p>
<p data-nodeid="31478">同时，我们也应该关注于各个 Span 之间的顺序关系。显然，在上图中，服务 A 位于服务 B 的上游，所以访问服务 A 所生成的 SpanId 应该是访问服务 B 所生成的 SpanId 的父 SpanId，服务 B 和服务 C 的调用关系以此类推。这样，我们通过获取请求的唯一性 TraceId，并通过各个父 SpanId 与子 SpanId 之间的关联关系就可以构建出一条完整的服务调用链路。</p>
<p data-nodeid="31479">关于 Span，业界一般使用四种关键事件记录每个服务的客户端请求和服务器响应过程。我们可以基于这四种关键事件来剖析一个 Span 中的时间表示方式，如下所示：</p>
<p data-nodeid="31480"><img src="https://s0.lgstatic.com/i/image2/M01/03/BB/Cip5yF_hnYOASaQaAACYZEepUCw895.png" alt="Drawing 1.png" data-nodeid="31531"></p>
<div data-nodeid="31481"><p style="text-align:center">Span 中的四种关键事件示意图</p></div>
<p data-nodeid="31482">在上图中，cs 表示 Client Send，也就是客户端向服务 A 发起了一个请求，代表了一个 Span 的开始。sr 代表 Server Receive，表示服务端接收客户端的请求并开始处理它。一旦请求到达了服务器端，服务器端对请求进行处理，并返回结果给客户端，这时候就会 ss 事件，也就是 Server Send。最后的 cr 表示 Client Receive，表示客户端接收到了服务器端返回的结果，代表着一个 Span 的完成。</p>
<p data-nodeid="31483">我们可以通过计算这四个关键时间之前的差值来获取 Span 中的时间信息。显然，sr-cs 值等于请求的网络延迟，ss-sr 值表示服务端处理请求的时间，而 cr-sr 值则代表客户端接收服务端数据的时间。</p>
<p data-nodeid="31484">通过这些关键事件我们就可以发现服务调用链路中存在的问题，目前主流的服务监控实现工具都对这些关键事件做了支持和封装。</p>
<p data-nodeid="31485">通过前面所介绍的服务监控基本原理，我们明确了分布式环境下服务跟踪的载体，即 TraceId 和 SpanId。但要实现服务跟踪，我们还需要围绕这两个载体做进一步的分析和挖掘。首先，我们需要对整个调用过程的所有服务进行埋点并生成事件，并对这些事件数据进行采集。同时，由于事件数据量一般都很大，不仅要能实现存储，还需要能提供快速查询。然后，在此基础上，我们还需要对采集到的事件数据进行各种指标运算，并将运算结果保存起来，并提供各种排序、阈值设置和警告等功能。</p>
<p data-nodeid="31486">这些工作不是简单一个工具和框架能全部完成的，我们也不想自己从无到有实现这样一整套解决方案。幸好在 Spring Cloud 中存在一个组件能够帮忙我们简化实现，这个组件就是 Spring Cloud Sleuth。在 Spring Cloud Sleuth 中，也把这些事件称为注解（Annotation），请你注意名称上的不同叫法。</p>
<h3 data-nodeid="31487">使用 Spring Cloud Sleuth 实现服务监控</h3>
<p data-nodeid="31488">Spring Cloud Sleuth 是 Spring Cloud 的组成部分之一，对于分布式环境下的服务调用链路，我们可以通过该框架来完成服务监控和跟踪方面的各种需求。</p>
<p data-nodeid="31489">当我们将 Spring Cloud Sleuth 添加到系统的类路径，该框架便会自动建立日志收集渠道，不仅包括常见的使用 RestTemplate 发出的请求，同时也能无缝支持通过 API 网关 Zuul 以及 Spring Cloud Stream 所发送的请求。本课程前面的课时已经对这些组件做了详细的介绍。</p>
<p data-nodeid="31490">针对监控数据的管理，Spring Cloud Sleuth 可以设置常见的日志格式来输出 TraceId 和 SpanId。我们也可以利用诸如 Logstash 等日志发布组件将日志发布到 ELK 等日志分析工具中进行处理。同时，Spring Cloud Sleuth 也兼容了 Zipkin、HTrace 等第三方工具的应用和集成。在下一课时中，我们就将集成 Spring Cloud Sleuth 与 Zipkin 来提供可视化的链路跟踪系统。</p>
<p data-nodeid="31491">接下来，就让我们引入 Spring Cloud Sleuth 框架。借助于 Spring Cloud Sleuth 中即插即用的服务调用链路构建过程，我们想要在某个微服务中添加服务监控功能，要做的事情只有一件，即把 spring-cloud-starter-sleuth 组件添加到 Maven 依赖中即可，如下所示。</p>
<pre class="lang-xml" data-nodeid="31492"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-sleuth<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="31493">初始化工作完成之后，接下来就来看一下引入 Spring Cloud Sleuth 之后为我们带来的变化，首当其冲的切入点是控制台日志分析。在 SpringHealth 案例系统中，我们通过发起一次请求操作来观察 Spring Cloud Sleuth 的运行时效果。</p>
<p data-nodeid="31494">我们知道 intervention-service 分别调用了 user-service 和 device-service。如果我们调用<a href="http://localhost:5555/springhealth/intervention/interventions/springhealth_admin/device1" data-nodeid="31548">http://localhost:5555/springhealth/intervention/interventions/springhealth_admin/device1</a> 端点。那么，在 user-service 中，在控制台上可以看到如下日志信息：</p>
<pre class="lang-xml" data-nodeid="31495"><code data-language="xml">INFO [userservice,81d66b6e43e71faa,6df220755223fb6e,true] 18100 --- [nio-8082-exec-8] c.s.user.controller.UserController&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; : Get user by userName from 8082 port of userservice instance
</code></pre>
<p data-nodeid="31496">我们关注于上述日志信息中的斜体部分内容，包括了四段内容，即服务名称、TraceId、SpanId 和 Zipkin 标志位，它是格式如下所示：</p>
<pre class="lang-xml" data-nodeid="31497"><code data-language="xml">[服务名称, TraceId, SpanId, Zipkin 标志位]
</code></pre>
<p data-nodeid="31498">显然，第一段中的 userservice 代表着该服务的名称，使用的就是在 bootstrap.yml 中 spring.application.name 指定的服务名称。考虑到服务跟踪的需求，为服务指定一个统一而友好的名称是一项最佳实践。</p>
<p data-nodeid="31499">第二段中的 TraceId 代表一次完整请求的唯一编号，上例中的 81d66b6e43e71faa 就是该次请求的唯一编号。在诸如 Zipkin 等可视化工具中，可以通过 TraceId 查看完整的服务调用链路。</p>
<p data-nodeid="31500">在一个完整的服务调用链路中，每一个服务之间的调用过程都可以通过 SpanId 进行唯一标识，例如上例中位于第三段的 6df220755223fb6e。所以 TraceId 和 SpanId 是一对多的关系，即一个 TraceId 一般都会包含多个 SpanId，每一个 SpanId 都从属于特定的 TraceId。当然，也可以通过 SpanId 查看某一个服务调用过程的详细信息。</p>
<p data-nodeid="31501">最后的第四段代表 Zipkin 标志位，该标志位用于识别是否将服务跟踪信息同步到 Zipkin， Zipkin 是一个可视化工具，可以将服务跟踪信息通过一定的图形化形式展示出来。因为在请求运行过程中，我们已经启动了 Zipkin 服务器，所以上例中的标志位值为 true，意味着所有跟踪信息将被同步到 Zipkin 中。关于 Zipkin 的详细介绍请参考下一课时的内容。</p>
<p data-nodeid="31502">如果你执行过前面课时中的代码，你会发现以上四段内容的日志显示效果为 [userservice,,,]，也就说默认请求下 TraceId、SpanId 和 Zipkin 标志位都为空，这些内容都是在引入 Spring Cloud Sleuth 之后被自动添加到了每一次服务调用中。</p>
<p data-nodeid="31503">同样，我们再来看一下 device-service 和 intervention-service 中的日志信息。device-service 中的控制台输出日志如下，同样可以看到用斜体表示的完整四段内容，其中 TraceId 为81d66b6e43e71faa，SpanId 为 e1dffdb86c81cc3c，Zipkin 标志位为 true：</p>
<pre class="lang-xml" data-nodeid="31504"><code data-language="xml">INFO [deviceservice,81d66b6e43e71faa,e1dffdb86c81cc3c,true] 18656 --- [nio-8081-exec-2] c.s.device.controller.DeviceController&nbsp;&nbsp; : Get device by code: device1 from port: 8081
</code></pre>
<p data-nodeid="31505">同样，在 intervention-service 中的日志信息中，我们也发现了类似的一条记录，其中 TraceId 为 81d66b6e43e71faa，SpanId 为 992aec60c399ece2，Zipkin 标志位也为 true：</p>
<pre class="lang-xml" data-nodeid="31506"><code data-language="xml">INFO [interventionservice,81d66b6e43e71faa,992aec60c399ece2,true] 28648 --- [nio-8081-exec-2] c.s.intervention.controller.InterventionController&nbsp;&nbsp; : Generate intervention for userName: springhealth_admin and deviceCode: device1.
</code></pre>
<p data-nodeid="31507">请注意，以上三段日志中的 TraceId 都是 81d66b6e43e71faa，也就是它们属于同一个服务调用链路，而不同的 SpanId 代表着整个链路中的具体某一个服务调用。我们从日志中的时间上也不难看出三者之间的调用时序。基于这三个服务以及 TraceId、SpanId 所生成的服务调用时序链路效果如下所示：</p>
<p data-nodeid="31997" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/04/33/Cip5yF_q9YKAXrG5AAGe1ZklSJw015.png" alt="Lark20201229-172231.png" data-nodeid="32001"></p>
<div data-nodeid="31998"><p style="text-align:center">三个服务调用链路效果图</p></div>






<p data-nodeid="31510">关于该链路的可视化效果和更详细的数据信息我们在下一课时中还会有具体展开。</p>
<h3 data-nodeid="31511">小结与预告</h3>
<p data-nodeid="31512">构建服务监控和链路跟踪在微服务系统开发过程中是一项基础设施类工作，而我们可以借助于 Spring Cloud Sleuth 来轻松完成这项工作。Spring Cloud Sleuth 内置了日志采集和分析机制，能够帮忙我们自动化建立 TraceId 和 SpanId 之间的关联关系。本课时对如何在业务开发过程中引入 Spring Cloud Sleuth 做了全面介绍。</p>
<p data-nodeid="31513">这里给你留一道思考题：你能描述服务监控过程中的 TraceId、SpanId、四大关键事件的概念和作用吗？</p>
<p data-nodeid="31514" class="">Spring Cloud Sleuth 是一个集成化的框架，可以与其他第三方组件进行无缝集成从而提供更加强大的链路跟踪功能。在下一课时中，我们就将通过集成 Zipkin 来实现可视化的链路跟踪效果。</p>

---

### 精选评论


