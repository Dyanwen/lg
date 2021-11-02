<p data-nodeid="1093" class="">目前市面上的应用集群稳定性的工具，不仅有 Sentinel 还有 Hystrix。前者是 Alibaba 出品，后者是 Spring-Cloud 生态使用的稳定性组件。</p>
<p data-nodeid="1094">两者都是通过控制流量（熔断、限流、降级等手段），在流量洪峰或异常情况下，实现应用服务稳定性。但它们使用原理封装出的策略，及用户体验都截然不同。</p>
<p data-nodeid="1095">今天我主要带你学习 Hystrix，然后再通过对 Sentinel 与 Hystrix 的比较，带你更深入地理解 APM 稳定性工具在并发编程上的作为和思考。</p>
<h3 data-nodeid="1096">简单回顾 Sentinel</h3>
<p data-nodeid="1097">Sentinel 以流量为切入点，提供流量控制、流量塑形、熔断降级、过载保护等维度的高可用保障策略。在架构上，被识别的流量会转换为 Sentinel Context 对象，通过使用<strong data-nodeid="1211">责任链模式</strong>，在一系列的功能插槽链中完成稳定性策略验证。</p>
<p data-nodeid="1098">关于 Sentinel 的技术骨架和责任链模式，你可回顾“11 | 资源节点树：如何通过 Sentinel 无侵入实现流量链生成规则？”</p>
<h3 data-nodeid="1099">详解 Hystrix 组件</h3>
<p data-nodeid="1100">Hystrix 是为了公司的服务集群不稳定而设计的，是国外视频网站 Netflix 开源的组件。</p>
<p data-nodeid="1101">Hystrix 提供了多种实现策略，这些策略的设计主旨是<strong data-nodeid="1222">监控应用程序的核心执行单元</strong>，也就是代码块的执行情况。当执行情况非符合预期时，将执行单元的调用切换为调用一个符合预期的，或是可正确处理的备选响应。</p>
<p data-nodeid="1102">这样的设计的好处是，在故障持续的时间内，流量会被切换至降级执行单元，保证在一段时间内，不会有持续的报错进而导致故障的发生。</p>
<p data-nodeid="1103">而且 Hystrix 还支持<strong data-nodeid="1229">健康探测</strong>，也就是会定期将小流量引导至非降级的核心模块上去执行。当监测到核心模块可以健康执行后，应用服务接收到的新流量就会自动切换回去。</p>
<p data-nodeid="1104">如下图所示，Hystrix 的架构设计非常清晰。</p>
<ul data-nodeid="1105">
<li data-nodeid="1106">
<p data-nodeid="1107">首先应用服务在引入 Hystrix 组件后，通过注解或是继承，将核心执行单元封装成 Hystrix Command 类；</p>
</li>
<li data-nodeid="1108">
<p data-nodeid="1109">然后在执行前，Hystrix 会先后检查应用服务内部“熔断”和“线程池”两个策略是否开启；</p>
</li>
<li data-nodeid="1110">
<p data-nodeid="1111">若通过策略的验证，将监视执行单元的执行过程；</p>
</li>
<li data-nodeid="1112">
<p data-nodeid="1113">并在出现异常或是验证策略失败时，执行注解或继承中降级策略。</p>
</li>
</ul>
<p data-nodeid="1366" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/3D/3A/Cgp9HWCTlcmAW02OAACQw3pf9vU693.png" alt="202156-1579.png" data-nodeid="1369"></p>

<p data-nodeid="1115">可以看出 Hystrix 的执行原理非常清晰。正因为 Sentinel 和 Hystrix 的架构实现都非常简单，一经发布就在社区得到了非常多用户的拥趸。</p>
<p data-nodeid="1116">那接下来，我们就对比 Hystrix，看下 Sentinel 在并发编程上使用了哪些设计，使其在流量洪峰下，依旧计算精准且性能损耗低。</p>
<h3 data-nodeid="1117">在并发编程上，流量控制工具如何作为？</h3>
<p data-nodeid="1118"><strong data-nodeid="1245">接下来我将以 Sentinel 为主，专注流量控制工具在并发编程上的思考</strong>。我会对 “并发请求隔离技术”和“吞吐的并发流量计算技术”进行展开；最后再以原理视角，重新审视 Sentinel 和 Hystrix 在产品形态上的异同。</p>
<p data-nodeid="1119">下图是 Sentinel 官方对 Sentinel 和 Hystrix 的对比。</p>
<p data-nodeid="1120"><img src="https://s0.lgstatic.com/i/image6/M01/3C/86/CioPOWCKiAGAC9RhAAHoNdERHTI462.png" alt="Drawing 1.png" data-nodeid="1249"></p>
<blockquote data-nodeid="1121">
<p data-nodeid="1122">可进入<a href="https://sentinelguard.io/zh-cn/blog/sentinel-vs-hystrix.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1253">“官方 Sentinel 和 Hystrix 对比”</a>了解更多。</p>
</blockquote>
<h4 data-nodeid="1123">1.并发隔离</h4>
<p data-nodeid="1124">通过上面的对比表，我们可以看出：在并发流量的隔离原理上，两者都可使用了<strong data-nodeid="1265">信号量隔离技术</strong>和<strong data-nodeid="1266">线程池隔离技术</strong>。由此可见，这两个技术在并发流量的隔离领域非常常见。</p>
<blockquote data-nodeid="1125">
<p data-nodeid="1126">但这两个技术对一线开发还是很陌生，所以有必要对这两个技术进行讲解。</p>
</blockquote>
<p data-nodeid="1127"><strong data-nodeid="1271">1）信号量隔离</strong></p>
<p data-nodeid="1128">信号量隔离比较简单，在 Java 中对应的类名为 Semaphore。常见的 APM 工具，在处理并发时多有涉及该技术。只不过由于业务开发用不到这些技术，所以他们对此多感到陌生。</p>
<p data-nodeid="1129">信号量技术可以理解为计数器，比如我们设计的被监视的执行单元 QPS 是 100，那对应的信号量计数器就设置为 100。</p>
<ul data-nodeid="1130">
<li data-nodeid="1131">
<p data-nodeid="1132">每一次调用执行单元时，就会去申请信号量，造成计数器减 1；</p>
</li>
<li data-nodeid="1133">
<p data-nodeid="1134">当退出执行单元时，计数器会加 1；</p>
</li>
<li data-nodeid="1135">
<p data-nodeid="1136">当申请信号量不足时，会无法申请到信号量，进而阻断执行单元被调用。</p>
</li>
</ul>
<p data-nodeid="1137">总的来说，请求会转化为申请信号量的操作。在信号量不足时，会拒绝申请。</p>
<p data-nodeid="1138"><strong data-nodeid="1281">2）线程池隔离</strong></p>
<p data-nodeid="1139">顾名思义，就是在流量到达被监视的执行单元时，为执行单元的执行单独创建线程池，进而实现隔离。</p>
<p data-nodeid="1140">常见的策略：1.多个执行单元有对等不同的线程池。2.单一执行单元由于调用来源不同，有着不同的线程池。这样的编程结果是：当接收到并发流量时，由于策略不同，执行单元的任务线程会在不同的线程池里面执行，从而实现了以线程池为维度的并发流量隔离。</p>
<p data-nodeid="1141">综上，这两种隔离技术的核心思想都是：识别并发流量，然后申请对应的统一资源（申请到资源即可执行），并保持申请资源之间的相互隔离；反观没有隔离技术的话，应用服务中的任意一个模块不稳定，都会造成集群的雪崩。</p>
<p data-nodeid="1142">我们再对比一下信号量隔离和线程池隔离，更形象深入地认识一下它们。</p>
<ul data-nodeid="1143">
<li data-nodeid="1144">
<p data-nodeid="1145">信号量隔离，如同高并发下的精准计数器，使架构更加轻量，让引入开销最小。</p>
</li>
<li data-nodeid="1146">
<p data-nodeid="1147">线程池隔离，将被执行单元封装到指定线程池执行，让隔离更彻底。<br>
由于线程池技术的存在，被执行单元支持在异步线程池排队。在任务线程执行超时时，可以主动断掉工作线程等场景，但这样也会使线程模型架构更加复杂。</p>
</li>
</ul>
<p data-nodeid="1148"><img src="https://s0.lgstatic.com/i/image6/M01/3C/86/CioPOWCKiBCALmjFAAKau0ImkMw506.png" alt="Drawing 2.png" data-nodeid="1292"></p>
<h4 data-nodeid="1149">2.并发控制</h4>
<p data-nodeid="1150">在流量洪峰下想要精准限流，就必须使用高性能的并发计算对象。而 Sentinel 底层便是通过<strong data-nodeid="1299">LongAddr 对象</strong>，解决了并发流量计算的两个难题——精确度难题和性能难题。</p>
<ul data-nodeid="1151">
<li data-nodeid="1152">
<p data-nodeid="1153"><strong data-nodeid="1303">精准度难题</strong></p>
</li>
</ul>
<p data-nodeid="1154">说到精准度，就要提到<strong data-nodeid="1309">CAS（Compare and Swap）原理</strong>了。Compare 表示比较，Swap 表示交换，两者通过 and 连接。其意思是：在替换一个值时，需要使用“原始值想要替换的值”一起进行并发操作；过程中伴随着“比较原始值是否发生变化”，然后“锁住变化的值进行更新”这两个操作。</p>
<ul data-nodeid="1155">
<li data-nodeid="1156">
<p data-nodeid="1157"><strong data-nodeid="1313">性能难题</strong></p>
</li>
</ul>
<p data-nodeid="1158">在我们的业务开发中，解决流量洪峰下的精准限流场景的方法是使用<strong data-nodeid="1319">AtomicLong 对象</strong>。AtomicLong 对象在实现 CAS 上，使用操作系统的 lock 信号，从而保证原子性。在竞争激烈的并发场景下，Atomic 原子类通过自旋锁实现值的累加，过程中会存在大量的操作失败尝试，也会带来极大的 CPU 消耗。</p>
<p data-nodeid="1159">而 LongAddr 对象为了在性能上超越 AtomicLong 原子对象，便在内部封装了一个 Cell 元素数组。并通过并发线程的 ID 的哈希值，分散访问数组中的元素，从而进行 CAS 操作。</p>
<p data-nodeid="1160">如下图所示，其原理就是利用哈希算法分散操作 CAS 对象带来的竞争。</p>
<p data-nodeid="1161"><img src="https://s0.lgstatic.com/i/image6/M01/3C/7E/Cgp9HWCKiByANXwuAAD-l4OAyPk504.png" alt="Drawing 3.png" data-nodeid="1324"></p>
<ul data-nodeid="1162">
<li data-nodeid="1163">
<p data-nodeid="1164">在使用 AtomicLong 计算并发流量的 QPS 时，所有线程都会访问同一个 CAS 变量进行 CAS 操作。</p>
</li>
<li data-nodeid="1165">
<p data-nodeid="1166">改用 LongAddr 后，再计算并发流量的 QPS 时，通过 Hash 算法将其分为“线程 1、2”和“线程 3、4”两组，从而去操作 CAS 元素数组中的不同变量。</p>
</li>
</ul>
<p data-nodeid="1167">原理易懂，但问题也明显：在并发线程同时操作数组的 CAS 元素时，统计数据会有误差。这时你可能会问：那在读取数据时，可否增加写锁来保证读取的正确性呢？答案是否定的。</p>
<p data-nodeid="1168">因为限流降级会频繁读取对象中的数据。增加写锁的话，LongAddr 就会被降级为类似 AtomicLong 原子对象，所带来的并发优势便荡然无存了。</p>
<p data-nodeid="1169">因此，根据场景来选型合适的并发控制对象非常必要。因为 Sentinel 仅用 LongAddr 对象进行统计，这在高并发场景下其性能损耗优先级最高，所以较小的误差是被允许的。</p>
<h4 data-nodeid="1170">3.控制台界面</h4>
<p data-nodeid="1171">关于这两个流量控制工具的底层编程实现，我们的讲解就告一段落了，我们再回看下两个组件的控制台。</p>
<p data-nodeid="1172"><strong data-nodeid="1335">1）Hystrix 控制台</strong></p>
<p data-nodeid="1173"><img src="https://s0.lgstatic.com/i/image6/M01/3C/7E/Cgp9HWCKiCmAUw79AAbA5hMcjKA409.png" alt="Drawing 4.png" data-nodeid="1338"></p>
<p data-nodeid="1174">上图为 Hystrix 的示例控制台，图中展示了九个区域。</p>
<ul data-nodeid="1175">
<li data-nodeid="1176">
<p data-nodeid="1177">蓝色趋势线：表示被监控的执行代码，显示的是最近两分钟内的并发流量的请求速率。</p>
</li>
<li data-nodeid="1178">
<p data-nodeid="1179">圆圈的大小标识 QPS：圆圈越大表示被执行的次数越多，而颜色表示执行单元的当前健康度。</p>
</li>
<li data-nodeid="1180">
<p data-nodeid="1181">右上侧数据：是最近十秒的异常情况分析。</p>
</li>
<li data-nodeid="1182">
<p data-nodeid="1183">右下侧数据：为最近 1 分钟的延迟情况</p>
</li>
</ul>
<p data-nodeid="1184">用简短的话描述 Hystrix 控制台，那就是可以实时展示并发流量引起监控的核心执行单元的执行情况。</p>
<blockquote data-nodeid="1185">
<p data-nodeid="1186">更详尽的介绍可以参见<a href="https://github.com/Netflix-Skunkworks/hystrix-dashboard/wiki?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1348">Hystrix 控制台介绍</a>。</p>
</blockquote>
<p data-nodeid="1187"><strong data-nodeid="1353">2）Sentinel 控制台</strong></p>
<p data-nodeid="1188"><img src="https://s0.lgstatic.com/i/image6/M01/3C/86/CioPOWCKiDGARqU1AAIjrJ0VPww924.png" alt="Drawing 5.png" data-nodeid="1356"></p>
<p data-nodeid="1189">Sentinel 控制台不仅可以对并发流量进行实时监控，它还有很多其他的丰富功能。</p>
<ul data-nodeid="1190">
<li data-nodeid="1191">
<p data-nodeid="1192">簇点链路：正如上节课时所讲的 AOP 思想，通过线程本地变量（也就是 ThreadLocal 技术），在应用框架暴露出的拦截器或是过滤器中埋下监测点，无侵入地实现簇点链路的自发现。</p>
</li>
<li data-nodeid="1193">
<p data-nodeid="1194">流控降级规则：异常情况支持响应时间、异常比例、异常数、多规则配置的应用降级策略。</p>
</li>
<li data-nodeid="1195">
<p data-nodeid="1196">系统保护规则：通过获取应用系统的负载情况，使应用服务的整体指标处于安全水位。</p>
</li>
</ul>
<p data-nodeid="1197">可以看出，Sentinel 在动态调控的规则上，通过同台数据源，实现了控制端和服务端的远程实时连接，从而赢得了更多先天优势；而且 Sentinel 在绝大多数场景下，是包含了 Hystrix 的各种功能能力的。</p>
<p data-nodeid="1198">所以若你需要对应用集群进行稳定性工具的产品选型的话，我更向你推荐 Sentinel。</p>
<h3 data-nodeid="1199">小结与思考</h3>
<p data-nodeid="1200">今天，我先带你学习了 Hystrix 的架构原理；之后又学习了稳定性工具在并发编程上的常用技术手段，包括信号量隔离技术、线程池隔离技术、高性能并发控制对象的设计等；最后通过产品功能，全面对比了 Sentinel 和 Hystrix，并得出 Sentinel 相对更优的结论。</p>
<p data-nodeid="1201" class="">在工作中，你肯定也使用过 Sentinel 和 Hystrix。那么你使用了什么规则呢？又应用到什么场景呢？欢迎你在评论区写下你的思考，非常期待与你讨论。</p>

---

### 精选评论


