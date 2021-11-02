<p data-nodeid="47893" class="">在专栏 Part 1 部分，我们学习了 7 个最流行的 APM 工具。在工作中，使用这些工具能帮助你高效地解决问题，并激发你对工具的更多兴趣，驱使你去了解原理。那如何“走捷径”快速掌握原理呢？</p>
<p data-nodeid="47894">这就进入专栏 Part 2 部分了。作为过来人，我将手把手带你，通过学习核心设计快速掌握原理。这种学习方式，不仅能免去读源码的麻烦，还能让你高效地掌握 APM 工具的精髓。</p>
<blockquote data-nodeid="47895">
<p data-nodeid="47896">像线程追踪模型、Opentracing 设计、无侵入实现等 APM 核心原理，大量文献处于外文的翻译或源码的代码级解读的情况。一个过于粗狂，让读者获取不到合适的学习路径；一个又过于细致，完全不适合入门。</p>
</blockquote>
<p data-nodeid="47897">诚然对这些知识的学习是必要的，但初、中级工程师，直接上手这些知识，只会更加“云里雾里”。</p>
<p data-nodeid="47898">所以 Part 2 内容将摒弃这些知识，以填完坑的过来人口吻，向你进行原理解密。Part 2 的四门课程，将围绕着 Skywalking 和 Sentinel 展开，讲述如何学习 Opentracing 原理，并结合 ThreadLocal 实现无侵入设计，再进行 APM 系统的低性能损耗实践.</p>
<p data-nodeid="47899">那现在就正式开始 Part 2 课程了，今天第一讲我将为你揭秘分布式链路跟踪 OpenTracing。</p>
<h3 data-nodeid="47900">什么是分布式链路跟踪？</h3>
<p data-nodeid="47901">如果让我用一句话概括分布式跟踪的话，那就是：<strong data-nodeid="47996">分布式追踪就是把一次用户请求引起的，在分布式集群中的相关日志，串联起来的过程。</strong></p>
<p data-nodeid="47902">市场上流行的分布式追踪实现是：通过链路标识，将一次用户请求，在分布式集群中各个应用节点打印的日志串联起来的过程，就是分布式链路跟踪。</p>
<p data-nodeid="47903">追踪又可以简单分为以下两种。</p>
<ul data-nodeid="47904">
<li data-nodeid="47905">
<p data-nodeid="47906">对于<strong data-nodeid="48004">跨进程</strong>的链路追踪方案，最优的技术方案是在远程调用发生时，通过对请求协议增加链路标识，将上下游调用链绑定起来，就实现了分布式链路追踪的跨进程追踪。</p>
</li>
<li data-nodeid="47907">
<p data-nodeid="47908">对于<strong data-nodeid="48010">跨线程</strong>的链路追踪方案，如 Java 语言的微服务应用，我们可以在主线程创建子线程时，将主线程的链路标识代入子线程，将主-子线程绑定起来，这样就实现了跨线程追踪。</p>
</li>
</ul>
<p data-nodeid="47909">理解追踪模型是学习分布式链路追踪的基础，目前以 SkyWalking、Zipkin 为代表的分布式追踪系统已经被广泛地接入到线上应用服务了。</p>
<h3 data-nodeid="47910">早期对典型追踪场景的论述</h3>
<p data-nodeid="47911">经典的追踪模型理论源自 Google 公司在 2010 年初发表的 Dapper 论文，它详细讲述了分布式场景下树形追踪模型的落地实践，这便是后世分布式监控的启蒙。</p>
<p data-nodeid="47912">Google 在出具这一经典的分布式追踪模型理论之前，业内对分布式链路追踪就已经出现了两种不是很成体系的分布式链路跟踪理论了，分为标记方案与黑盒方案。</p>
<h4 data-nodeid="47913">1.标记方案（有侵入、准确度高）</h4>
<p data-nodeid="47914">标记方案是目前最常见的实现方案，目前的 SkyWalking、CAT 等分布式追踪监控都是使用标记方案实现的。</p>
<p data-nodeid="47915">标记法顾名思义，就是对应用监控日志打标记。在应用程序中显式地对日志打标记，或者在框架拦截器中对请求进行标记。通过这些标记，将一次请求引起的应用服务集群中的监控日志全部串联汇总起来的过程就是标记方案。</p>
<p data-nodeid="47916">如下图，Dapper 所论述的示例：</p>
<ul data-nodeid="47917">
<li data-nodeid="47918">
<p data-nodeid="47919">前台服务 A 收到 RequestX 请求，Http Server 框架发现请求中没有标记，那么对这次流量进行标记，并识别为用户请求进行传播。</p>
</li>
<li data-nodeid="47920">
<p data-nodeid="47921">当请求传播到 B、C、D、E，再打印监控日志时，监控日志就带着与 A 衍生出来的标记，这样这次用户的请求通过 A 服务生成的标记就很容易被收集起来了。</p>
</li>
</ul>
<p data-nodeid="47922"><img src="https://s0.lgstatic.com/i/image6/M01/3B/0F/Cgp9HWCCe6SAcfDJAABhm1rEzis920.png" alt="Drawing 0.png" data-nodeid="48023"></p>
<p data-nodeid="47923">不难看出，标记法的优点是精准；但缺点也显而易见，需要打标记。那你可能就会问了：</p>
<blockquote data-nodeid="47924">
<p data-nodeid="47925">我在使用 SkyWalking、Sentinel 时，并没有打标记呀，而且很多博客都是说它们是无侵入的呀！</p>
</blockquote>
<p data-nodeid="48754" class="te-preview-highlight"><strong data-nodeid="48759">其实这里的“无侵入”需要打上引号，因为这些打标记的代码确确实实存在。</strong> 虽然通过 Agent，或一些解耦技术，可以做到用户无感知，但这些打标记的埋点代码都是在 APM 客户端代码实现的。得益于近些年的框架越来越标准，通过使用线程模型的追踪方式，很容易就实现了“无侵入”的 APM 系统。</p>


<h4 data-nodeid="47927">2.黑盒方案（无侵入、准确度低）</h4>
<p data-nodeid="47928">黑盒方案是另一种分布式追踪监控的实现方法，因为 SkyWalking、Dapper 的流行，标记方案便先入为主了。</p>
<p data-nodeid="47929">两个方案对比下，黑盒方案在无侵入的实现上，高于标记方案一个维度。黑盒方案认为应用服务是黑盒的，我们只关心可见的请求与日志，通过机器学习的方法，将这些请求和日志关联起来。</p>
<blockquote data-nodeid="47930">
<p data-nodeid="47931">最常见的机器学习方式用回归分析等统计学算法，将这些碎片化的请求和日志通过推测，重新组合成完整的链路。</p>
</blockquote>
<ul data-nodeid="47932">
<li data-nodeid="47933">
<p data-nodeid="47934">黑盒方案的优势显而易见，不需要任何应用服务的架构修改和监控日志的变更；</p>
</li>
<li data-nodeid="47935">
<p data-nodeid="47936">黑盒方案的缺点很明显，机器学习的精度是逐渐提高的。所以上线新功能时，往往分布式监控可用度非常低，而且机器学习会消耗很多的运算资源。</p>
</li>
</ul>
<p data-nodeid="47937">综上所述，结合 2010 年的时代背景，以及当时的两种分布式链路跟踪理论方案，<strong data-nodeid="48042">Dapper 采样了标记方案实现了分布式追踪</strong>。</p>
<h3 data-nodeid="47938">Dapper 的树形追踪模型</h3>
<p data-nodeid="47939">通常 Dapper 的追踪场景为追踪用户搜索行为，所以链路形态就如上图所示，呈现以用户发起请求为始，打到的前端系统为根节点，然后层层展开的<strong data-nodeid="48049">树形结构</strong>。</p>
<p data-nodeid="47940">在简单的场景下，如跟踪用户的搜索行为在集群中的链路的这种场景，树形跟踪的链路优势显而易见。像 CAT、pinpoint 等 APM 工具在分布式追踪也都是采用这种模型实现的。</p>
<p data-nodeid="47941">我们可以从学习成本和实现成本这两个视角，来看下树形追踪模型在早年为什么如此流行。</p>
<p data-nodeid="47942"><strong data-nodeid="48055">1.学习角度</strong></p>
<p data-nodeid="47943">首先对于垂直单一应用，处理的用户发起的每一个 HTTP 请求都是一个 HTTP Server 框架的任务线程完成的。</p>
<p data-nodeid="47944">在单一线程的应用日志，打点是顺序的链式结构；当应用服务的部分代码块以微服务的形式迁移出去后，链式结构的相应节点也是通过标记方案中的父子关系进行链路的关联，形成的树形结构。这种每人都会的数据结构在应用到分布式追踪模型时，基本没有学习成本的。</p>
<p data-nodeid="47945"><strong data-nodeid="48061">2.实现成本</strong></p>
<p data-nodeid="47946">树形追踪的链路由一个个代码块打点数据组成，Dapper 论文中称之为 Span。如下图所示：每一个 Span 有这样几个基础属性：代码段执行时间、代码段描述、父 SpanId、当前 SpanID。</p>
<p data-nodeid="47947"><img src="https://s0.lgstatic.com/i/image6/M00/3B/17/CioPOWCCe7KAHSLXAAKzybjRV_A322.png" alt="Drawing 1.png" data-nodeid="48065"></p>
<p data-nodeid="47948">这些数据都非常容易理解。每个代码段的连接是这样完成的：通过 RPC 框架暴露出的面向传输消息体，在消息头部中设置 SpanID 就完成了 Span 节点的串联。尤其是在 Java 这种非常成熟的生态环境中，通过框架暴露出的拦截器，就可以轻松实现 SpanID 的埋点。</p>
<p data-nodeid="47949">在现场还原上，常用的富客户端都支持树形数据结构展示，下面是 Dapper 的用户交互页面。</p>
<p data-nodeid="47950"><img src="https://s0.lgstatic.com/i/image6/M01/3B/0F/Cgp9HWCCe7iAUjouAAcbVZEai4U865.png" alt="Drawing 2.png" data-nodeid="48070"></p>
<p data-nodeid="47951">可以看出每一个“加号”按钮，代表链路发生了跨进程。通过点击“加号”或“减号”按钮，可以快速跟踪请求在各个分布式应用服务中的执行情况。</p>
<h3 data-nodeid="47952">SkyWalking 的图形追踪模型</h3>
<p data-nodeid="47953">受 Dapper 对典型追踪场景的实践启发，市场上涌现出很多开源作品，SkyWalking 就是其中之一。SkyWalking 沿用了 Dapper 的标记方案，但对树形追踪的链路模型进行了非常大的改进，将追踪模型扩展为<strong data-nodeid="48077">兼容树形模型的图形追踪模型。</strong></p>
<h4 data-nodeid="47954">1.克服树形追踪的不足——无法批处理</h4>
<p data-nodeid="47955">据我们所知，<strong data-nodeid="48084">图与树最大的区别就是每个节点可以有多个父节点</strong>。这样的实现有什么好处呢？</p>
<p data-nodeid="47956">Dapper 追踪的最小维度是 Span，也就是树形结构的每一个节点，它代表了监控的一段代码块（Span）。那什么情况下一个段代码的执行由多个请求引发呢？</p>
<p data-nodeid="47957">仔细想一想，相信你已经有了答案了。那就是——框架中存在<strong data-nodeid="48091">批处理</strong>场景。</p>
<p data-nodeid="47958">常见的应用服务在消费 Kafka 记录时，若是单条消费遇到性能瓶颈，这时就需要开启批量消费，而批量的消息可能来自各个异构的系统。如果此时还用树形模型强行绘制链路，检录不仅会绘制得不够形象，还会出现“笛卡尔积”情况，导致链路数量暴增等问题。</p>
<p data-nodeid="47959"><strong data-nodeid="48096">【预测系统场景】</strong></p>
<p data-nodeid="47960">一个典型的场景就是预测系统，通过各个系统的行为数据对产品的未来做出预测，如下图所示。</p>
<p data-nodeid="47961"><img src="https://s0.lgstatic.com/i/image6/M00/3B/17/CioPOWCCe8GALAUFAAGrr9sjvGk368.png" alt="Drawing 3.png" data-nodeid="48100"></p>
<p data-nodeid="47962">用户对各个系统的操作引发了一系列数据，这些数据会推送到消息队列，然后预测服务批量消费。可以看到预测服务的一次预测算法代码块的执行，是由多个业务服务引发的。那从预测服务这个点来看，如何与多个生产者服务串联链路呢？树形追踪模型显然是不能满足的。</p>
<p data-nodeid="47963">所以接下来我们就以 SkyWalking 链路模型的设计为例，详细展开对图形追踪的讲解。</p>
<h4 data-nodeid="47964">2.图形追踪的实现——支持批处理和“断链”场景</h4>
<p data-nodeid="47965">上文中，我们理解了<strong data-nodeid="48109">树形追踪模型无法支持批处理场景</strong>，进而出现了图形追踪模型。接下来再理解图形追踪的实现，其实很简单了。</p>
<p data-nodeid="47966">正所谓“知其然并知其所以然”：图形追踪，将树形追踪有关父节点的所有描述，由只支持存储 1 个父节点描述，改为支持存储多个父节点描述。</p>
<p data-nodeid="47967">以 SkyWalking 的存储模型为例。</p>
<ul data-nodeid="47968">
<li data-nodeid="47969">
<p data-nodeid="47970"><strong data-nodeid="48116">Span 打包</strong>：SkyWalking 将一个任务线程中的所有 Span 打包为 Segment。一个 Segment 中的 Span 由 0 递增，且具有相同父节点依赖。</p>
</li>
<li data-nodeid="47971">
<p data-nodeid="47972"><strong data-nodeid="48121">父节点描述</strong>：SkyWalking 父节点描述由多个属性组成。Refs 属性：描述当前 Segment 与相关生产者的 Segment 标识。在 RPC 调用的树形追踪链路中，为 1 个元素；但是当追踪场景有批处理框架时，链路从批处理框架起就为多个元素。</p>
</li>
<li data-nodeid="47973">
<p data-nodeid="47974"><strong data-nodeid="48126">分布式链路 ID</strong>：分布式链路 ID 是重要的链路标记，其数据结构为数组。如预测场景中，商品类目和交易服务等业务服务的分布式链路 ID 的数组长度为 1 个；当预测服务拉取一批用户请求行为后，分布式链路跟踪 ID 就是多个了。数量和拉取消息中的父节点描述数量有关。</p>
</li>
</ul>
<p data-nodeid="47975">SkyWalking 使用图形追踪模型，便是其在分布式链路跟踪最底层——数据层这点上，与其他分布式追踪产品的根本区别。</p>
<p data-nodeid="47976">基于图形追踪模型的设计，消息队列、分布式事务等涉及批处理的框架便可以很好地接入 SkyWalking 的性能监控体系。这也就是 SkyWalking 相比其他众多应用性能监控产品，支持的监控场景最多、最复杂的原因。</p>
<h3 data-nodeid="47977">小结与思考</h3>
<p data-nodeid="47978">本节课程的内容思路大致如下图所示，可供你回顾参考。</p>
<p data-nodeid="47979"><img src="https://s0.lgstatic.com/i/image6/M01/3B/DA/Cgp9HWCHc9OAfTM0AALZleEvn0A440.png" alt="图片1.png" data-nodeid="48133"></p>
<p data-nodeid="47980">今天我先带你先学习了 2010 年的分布式追踪方案。包括无侵入的，但是准确度相对较低的<strong data-nodeid="48143">黑盒方案</strong>；以及准确度较高，但有侵入的<strong data-nodeid="48144">标记方案</strong>。</p>
<p data-nodeid="47981"><strong data-nodeid="48161">Google</strong>公司使用标记方案，并通过<strong data-nodeid="48162">树形模型</strong>实现了分布式追踪监控，并通过 Dapper 论文对外分享。之后<strong data-nodeid="48163">SkyWalking</strong>受其启发，将树形跟踪模型扩展为<strong data-nodeid="48164">图形追踪模型</strong>。</p>
<p data-nodeid="47982">在批处理的场景下，链路绘制的还原度非常高，而<strong data-nodeid="48170">批处理</strong>只是图形追踪实现的优势之一。</p>
<p data-nodeid="47983">图形追踪的另一个优势就是<strong data-nodeid="48176">对“断链”场景的支持</strong>。因为我们通常在讨论分布式链路追踪时，预设的场景都是应用服务都接入了 APM 监控。但线上应用集群很难做到 100% 接入，所以断链的场景会非常多。断链后的应用服务在接收请求时，发现没有获取到分布式追踪 ID，会识别为人工请求，造成链路还原度很低，而图形的追踪模型对这种场景的支持则更加友好。</p>
<p data-nodeid="47984" class="">相信你在使用分布式链路监控时，遇到过“断链”或是链路无法绘制的问题也很多，欢迎在评论区写下你的问题和思考，期待与你讨论。</p>

---

### 精选评论

##### *磊：
> 好好

##### *磊：
> 这个tracesegment对象里面已经没有refs了，在8.5版本中

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 为避免OOM refs属性已经放在AbstractTracingSpan对象中了，并加入限制list大小字段，更详细的设计可以参考https://github.com/apache/skywalking/pull/6715

##### *磊：
> refs这个值在最近版本怎么成一个对象了，只记录第一个，那以后批处理怎么关联？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; refs存储是在TraceSegment对象中

##### **在：
> 请问skywalking对于断链支持更加友好这一点能详细说明下么 ，还是不是很理解，因为按照我的想法，如果链路断开了，上游服务是不知道下游的traceid的，这个时候拓扑图应该是不会串联起来的吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 探针技术，提供更高效的接入能力

##### **聪：
> 微服务中某个组件升级后，造成apm工具无法侵入，这时和没接入apm监控没区别，形成“断链”

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以详细描述下场景？什么组件，什么APM工具，除了升级组件还做了什么环境变更等

##### **明：
> 小结中对于黑盒和标记的描述，与上面小标题中的描述不一致

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢指错，两个写反了，现在已更正

