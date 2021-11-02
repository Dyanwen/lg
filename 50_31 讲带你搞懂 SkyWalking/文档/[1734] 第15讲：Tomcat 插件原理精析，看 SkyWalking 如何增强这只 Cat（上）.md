<p>通过前面几课时的学习，我们已经了解了 SkyWalking Agent 中最底层的 apm-agent-core 模块的核心实现，相信同学们已经了解下面几个知识点：</p>
<ul>
<li>SkyWalking Agent 的整体架构、启动流程。</li>
<li>插件埋点的基本原理，其中深入讲解了对静态方法、构造方法以及实例方法的拦截和增强，并结合 mysql-8.x 插件进行了串讲。</li>
<li>Trace 基本概念在 SkyWalking 中的落地，其中讲解了 Trace ID、Span、TraceSegment、TracingContext 等核心组件的实现，并结合 demo-webapp 进行了分析。</li>
<li>核心 BootService 实现的深入剖析，其中包括了网络连接的封装和管理、服务以及服务实例的注册流程、定期心跳、EndpointName、NetworkAddress 定期同步、Context 生成与管理、客户端采样的功能、Trace 的收集与发送。</li>
<li>DataCarrier 核心原理的深入剖析。</li>
</ul>
<p>其中最重要的是，理解 SkyWalking Agent 中，Trace 的相关组件是如何系统工作的，数据流向是什么样儿的。在接下来的几课时中，我们将从 apm-sdk-plugin 模块中选取几个比较有代表性的插件进行剖析，使你能够了解这些 SkyWalking Agent 插件是如何与 apm-agent-core 模块配合工作的。</p>
<p>本课时重点要介绍的是 tomcat-7.x-8.x-plugin 插件，如果你想要看懂 Tomcat 插件的原理，需要对 Tomcat 本身的结构有一些了解。</p>
<h3>Tomcat 架构基础</h3>
<p>Tomcat 的核心架构如下图所示，最顶层的 Server 代表整个 Tomcat 服务器，它可以包含多个 Service：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/5A/CgoCgV6nzhOAav2wAAJz0wrnIb8899.png" alt="1.png"></p>
<p>每个 Service 都会包含两个部分：Connector 和 Container，其中 Connector 用于处理连接相关的事情，并提供 Socket 与 Request 和 Response 相关的转化。Container 用于封装和管理Servlet，以及具体处理 Request 请求的业务。</p>
<p>一个 Service 可以有多个 Connector 连接器，这主要是因为一个服务可以支持多种网络协议，如下图所示的 HTTP、HTTPS 等，当然也可以在不同端口支持相同的协议。我们在工作中写 Spring MVC Controller 时用到的 HttpRequest 和 HttpResponse 对象就是由 Connector 创建的，这些 HTTP 请求的后续处理，则是由 Container 来负责的。</p>
<p>SkyWalking 提供的 tomcat-7.x-8.x-plugin 插件与 Tomcat Connector 组件没有任何关系，这里不再深入剖析 Connector 的原理，只要知道其功能是处理 Socket 网络连接与 Reques 和 Response 之间的转换即可。</p>
<p>Container 是容器的父接口，所有子容器都必须实现这个接口。Tomcat 中有四个子容器组件，分别是：Engine、Host、Context、Wrapper，这四个组件之间不是平行关系，而是父子关系。Engine 包含 Host，Host 包含 Context，Context 包含 Wrapper。下面是四个 Container 的核心功能。</p>
<ul>
<li>Engine：用于管理多个站点，一个 Service 最多只能有一个 Engine。</li>
<li>Host：代表一个站点，也可以叫虚拟主机，通过在 server.xml 配置文件就可以添加 Host，一个 Host 下可以运行多个 Context，但是在实践中，单 JVM 的处理能力有限，一般一个 Tomcat 实例只会配置一个 Host，也只会配置一个 Context。</li>
<li>Context：代表一个应用程序，对应你在日常开发的一个 Web 应用。Context 最重要的功能就是管理它里面的 Servlet 实例，并为 Request 匹配正确的 Servlet。Servlet 实例在 Context 中是以 Wrapper 出现的。</li>
<li>Wrapper：一个 Wrapper 负责管理一个 Servlet，包括 Servlet 的装载、初始化、执行以及资源回收。Wrapper 是最底层的容器，它没有子容器了。</li>
</ul>
<p>下面这张图大致展示了从 Connector 开始接收请求，经过 Engine、Host、Context、Wrapper，最终到 Servlet 的流程，这里需要关注的是拿到 Request 请求对象之后的处理：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/5B/CgoCgV6nzlmAEVwIAAL7sBPspMA637.png" alt="2.png"></p>
<p>Container 中真正处理请求的是 Valve，一组 Valve 组成一个 Pipeline，这是典型的责任链模式。责任链模式是指在一个请求处理的过程中会有很多处理器依次对请求进行处理，每个处理器只负责处理自己相应的部分，当对应的部分被处理完成之后，会将请求交给下一个处理器继续处理，直至请求完全处理完成。</p>
<p>以现实生活中汽车组装为例，整个责任链就像是汽车的生产线，责任链上的每个处理器则对应每个组装车间，每个组装车间只组装汽车的一部分，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/0A/5B/CgoCgV6nzmmAcpAbAACdLrM19_c471.png" alt="3.png"></p>
<p>在每个 Container 的 Pipeline 中，我们可以增加任意多个 Valve，处理请求的 Tomcat 线程会依次执行这些 Valve，并最终完成请求的处理。在上图中我们可以看到，每个 Pipeline 都有一个特定的 Valve（即图中的 StandEngineValve、StandHostValve、StandContextValve、StandWrapperValve），而且这些 Valve 是在 Pipeline 中最后一个执行，这种 Valve 叫作BaseValve。我们可以在 Tomcat 的 server.xml 文件中自定义 Pipeline 中的 Valve，但上述四个 BaseValve 是不可删除的。这些 BaseValve 会负责调用子容器的 Pipeline，将请求传给子容器，以保证处理逻辑能继续向下执行。Valve 接口与四个标准 Valve 实现的继承关系如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/89/Ciqah16nznSAYGnWAAETh59IW7M573.png" alt="4.png"></p>
<h3>tomcat-7.x-8.x-plugin 插件</h3>
<p>Tomcat 一般作为服务入口接收 HTTP 或 HTTPS 请求，tomcat-7.x-8.x-plugin 插件要做的事情也比较明确：</p>
<ol>
<li>在请求进入 Web 项目之前进行拦截。</li>
<li>检测当前请求是否处于一个 Trace 之中，也就是检测当前请求是否携带了 ContextCarrier。如果携带了 ContextCarrier，则在创建 TracingContext 时恢复上下文信息，保持实现 Trace 跨进程传递；如果没携带 ContextCarrier，则会开启一个全新的 TracingContext。</li>
<li>创建（或 restart） EntrySpan。</li>
<li>记录一些额外的信息，例如，请求相关的 Tags 信息（请求的 URL、Method 信息等），记录当前组件的类型（即 Tomcat）等。</li>
</ol>
<p>通过对 Tomcat 结构的分析，以及对 tomcat-7.x-8.x-plugin 插件的功能定位分析，相信你已经发现，tomcat-7.x-8.x-plugin 插件在 StandardHostValve 处拦截请求是合适的。Valve 接口中定义的 invoke(Request request, Response response) 方法是每个 Valve 的核心逻辑，例如，根据请求信息进行过滤、修改请求的特殊字段、打印 access log 等，正如前文介绍的，那些特殊功能的 Valve 实现是可插拔的，而标准 Valve 实现不可删除，这里 StandardHostValve 实现的 invoke() 方法只负责选择合适的 Context 继续处理请求，下面是其核心实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(Request request, Response response)</span></span>{
    <span class="hljs-comment">// 根据请求选择Context</span>
    Context context = request.getContext(); 
    <span class="hljs-comment">// 获取Context中第一个Valve，并调用其invoke()方法</span>
    context.getPipeline().getFirst().invoke(request, response);
    Throwable t = (Throwable) request
         .getAttribute(RequestDispatcher.ERROR_EXCEPTION);
    <span class="hljs-keyword">if</span> (response.isErrorReportRequired()) {
       <span class="hljs-keyword">if</span> (t != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 出现异常的话，会调用throwable()方法处理</span>
           throwable(request, response, t);
        }
    }
}
</code></pre>
<p>因此，tomcat-7.x-8.x-plugin 插件拦截 StandardHostValve 的 invoke() 方法即可满足之前的需求。</p>
<p>tomcat-7.x-8.x-plugin 插件在 SkyWalking 项目中的位置如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/8A/Ciqah16nzs-AaQEyAANfduRtXP8265.png" alt="5.png"></p>
<p>在前文介绍 SkyWalking Agent 启动流程时提到，SkyWalking Agent 启动时会扫描 agent 目录下的全部插件 jar 包，并根据每个插件 jar 包中的 skywalking-plugin.def 配置文件加载指定的 AbstractClassEnhancePluginDefine 实现。tomcat-7.x-8.x-plugin 插件的 skywalking-plugin.def 配置文件如下：</p>
<pre><code data-language="java" class="lang-java">tomcat-<span class="hljs-number">7</span>.x/<span class="hljs-number">8</span>.x=org.apache.skywalking.apm.plugin.tomcat78x.define
.TomcatInstrumentation

tomcat-<span class="hljs-number">7</span>.x/<span class="hljs-number">8</span>.x=org.apache.skywalking.apm.plugin.tomcat78x.define
.ApplicationDispatcherInstrumentation
</code></pre>
<p>这两个类都继承了 ClassInstanceMethodsEnhancePluginDefine 抽象类，同时间接继承了 ClassEnhancePluginDefine 类，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/17/8A/Ciqah16nzu2AXC4aAACqCsqjVe0795.png" alt="6.png"></p>
<p>这里先简单回顾一下 ClassEnhancePluginDefine 这个类，ClassEnhancePluginDefine 抽象类使用了模板方法模式：只定义了增强 static 静态方法、构造方法、实例方法（以及增加字段）的流程，具体增强哪些方法则由子类实现，也就是说，ClassEnhancePluginDefine 的子类需要实现下面三个方法：</p>
<ul>
<li><strong>getStaticMethodsInterceptPoints()方法</strong>：用于获取 static 静态方法增强点，也就是说，指定了增强哪些类的哪些 static 静态方法。</li>
<li><strong>getConstructorsInterceptPoints()方法</strong>：用于获取构造方法增强点，也就是说，指定增强哪些类的哪些构造方法。</li>
<li><strong>getInstanceMethodsInterceptPoints()方法</strong>：用于获取实例方法增强点，也就是说，指定增强哪些类的哪些实例方法。</li>
</ul>
<p>ClassInstanceMethodsEnhancePluginDefine 只实现了 getStaticMethodsInterceptPoints() 方法，且具体实现为空实现，也就是说，它的所有子类都不会增强 static 静态方法，只会增强构造方法或实例方法，例如下面即将要介绍的 TomcatInstrumentation 实现。</p>
<p>而 ClassStaticMethodsEnhancePluginDefine 则正好相反，它实现了 getConstructorsInterceptPoints() 和 getInstanceMethodsInterceptPoints() 两个方法，并且这两个方法都是空实现，也就是说，它的所有子类都不会增强构造方法和实例方法，只会增强 static 静态方法，例如后面我们将要介绍的 apm-toolkit-activation 工具箱中的 TraceContextActivation 实现。</p>
<p>你可以回顾一下 AbstractMysqlInstrumentation 这个类，它同时实现了上述三个方法（且三个方法都是空实现），然后由子类根据具体情况进行覆盖。在实践中你可以比较 AbstractMysqlInstrumentation 与 ClassInstanceMethodsEnhancePluginDefine、ClassStaticMethodsEnhancePluginDefine 的设计方式，根据实际情况进行折中选择。</p>

---

### 精选评论

##### **0097：
> 学习

