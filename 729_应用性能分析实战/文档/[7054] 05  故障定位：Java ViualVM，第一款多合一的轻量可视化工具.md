<p data-nodeid="1173" class="">你好，这一讲我将带领你学习 Java VisualVM（All-in-One Java Troubleshooting Tool），VisualVM 是第一款集成了<strong data-nodeid="1290">Java 命令行工具</strong>和<strong data-nodeid="1291">支持实时在线问题分析</strong>的轻量级可视化工具，作为 Oracle Java 6 的一部分对外发布。</p>
<p data-nodeid="1174">在上个年代，VisualVM 因为使用 JMX 规范，所以接入非常简单；而且 VisualVM 是<strong data-nodeid="1305">多合一的生态架构</strong>，有很多<strong data-nodeid="1306">第三方提供的扩展功能</strong>，具备<strong data-nodeid="1307">强大的插件体系</strong>。所以在发布早期，就深受开发人员和企业的关注。</p>
<p data-nodeid="1175">但遗憾的是，尽管 VisualVM 是官方制作且如此经典，可是在定位线上问题和场景剖析时，很多一线开发人员还是没有使用起来这个工具。至于背后的原因、落地、学习路径，今天我就与你系统性地学习一下。</p>
<p data-nodeid="1176">通过落地方案，熟悉 VisualVM 在落地过程中需要规避的风险；通过学习路径，掌握如何学习 VisualVM。争取在定位问题时，提高 VisualVM 的使用率和定位问题的效率。</p>
<h3 data-nodeid="1177">落地方案</h3>
<p data-nodeid="1178">VisualVM 的部署非常简单，在应用服务的启动命令中，增加相应的 JMX 管理功能参数，然后在本地使用 VisualVM 客户端连接 JMX 端口即可。</p>
<p data-nodeid="1179">由于 VisualVM 接入简单且功能非常强大，所以一线开发人员在不正确使用 VisualVM 的情况下，很容易会造成线上故障。</p>
<blockquote data-nodeid="1180">
<p data-nodeid="1181">比如：在没有摘除流量的情况下，点击监视选项卡页面中的“堆 Dump”按钮，进行制作 JVM 内存快照时，会产生“Stop The World”现象，造成服务在短时间内不可用的问题。</p>
</blockquote>
<p data-nodeid="1182">这些不正当使用操作造成的不必要线上故障，我们可以通过以下方案去减少甚至规避。</p>
<h4 data-nodeid="1183">1.操作权限</h4>
<p data-nodeid="1184">VisualVM 不支持多用户-角色模式，所以我们需要尽可能缩小使用 VisualVM 操作应用服务的人员范围。为了实现这个目的，我使用开启 JMX 的认证，并结合产品树系统（负责管理应用服务归属于哪个团队）实现权限的下发。</p>
<p data-nodeid="1185">这样实现的好处是：动态管理产品树上每个应用服务节点上的归属成员，就可以实现 VisualVM 工具权限的统一管理。使用产品树对所有 APM 工具的使用权限进行收口，在团队愈发壮大时，收益也会越来越大。</p>
<h4 data-nodeid="1186">2.安全限制</h4>
<p data-nodeid="1187">操作权限中，我们提到了使用 VisualVM 需要开启 JMX（Java Management Extensions），那 JMX 是什么呢？JMX 是一个规范，它实现了 Java 远程监控管理的接口。</p>
<p data-nodeid="1188">所以应用服务增加了相应的 JMX 管理功能参数后，不仅 VisualVM 可以通过 JMX 来监控管理这个应用服务，还有很多的 APM 工具，以及其他黑客入侵工具可以通过 JMX 来监控管理这个应用服务。</p>
<blockquote data-nodeid="1189">
<p data-nodeid="1190">如果你感兴趣可以通过搜索【基于JMX协议攻击】来进行安全知识的学习。</p>
</blockquote>
<p data-nodeid="1191">所以上文中开启 JMX 认证只是一方面，在网络安全我们也需要联合网络工程师，拒绝外部访问应用服务的 JMX 端口。</p>
<p data-nodeid="1192">对于一线开发人员，我们对 VisualVM 落地方案了解即可。因为整个落地方案需要我们懂得的知识非常少，也非常简单。在需要使用 VisualVM 时，我们只需要找到“产品树”系统，获取相应的权限，然后就可以对线上进程进行剖析了。</p>
<p data-nodeid="1193">所以一线开发人员，需要学习的更多是 VisualVM 工具赋予我们的能力，并能用这些能力去快速解决问题。</p>
<h3 data-nodeid="1194">为什么 VisualVM 让人感到鸡肋？</h3>
<p data-nodeid="1195">而现实情况是越来越多同学已经慢慢忘却了 VisualVM 这一工具。通过我的实地考察以及问卷调查发现，当线上问题（排除可以通过日志解决的问题）出现，需要使用 APM 工具定位问题时，VisualVM 的优先级非常低。</p>
<p data-nodeid="1196">即使用了，他们基本停留在使用原始的 VisualVM，即只会使用原始的四个 VisualVM 选项卡：概览、监视、线程、采样。而这四个选项卡提供的能力很有限，所以其<strong data-nodeid="1332">定位准确率</strong>也不会很高，且其他 APM 工具也基本都具备这四个选项卡的能力。</p>
<p data-nodeid="1197">所以久而久之，一线的开发人员定位问题会优先使用其他 APM 工具，最后连 VisualVM 原始的四个选项卡的能力也逐渐忘却了（稍后我会带你回顾四个选项卡）。不能掌握工具，便会导致使用工具时愈发不顺手。</p>
<p data-nodeid="1198">诚然 VisualVM 有一定的自身原因，比如：文档全部是英文，大部分实践经验没有很好地渗透到国内。加之国内 APM 工具近些年的开源盛世，VisualVM 在国内的一线开发人员中就更加无人问津了。</p>
<p data-nodeid="1199">对于一个 Java 官方制作且每个企业的每个应用服务都有的 APM 工具，难道 VisualVM 是个鸡肋？答案肯定是否定的。正如一开始所说，VisualVM 的插件体系非常强大，非常值得你学习，这也是 VisualVM 是鸡腿不是鸡肋的原因。</p>
<blockquote data-nodeid="1200">
<p data-nodeid="1201">关于 VisualVM 插件体系，我稍后会讲解。</p>
</blockquote>
<h3 data-nodeid="1202">VisualVM 的学习路径</h3>
<p data-nodeid="1203">想让 VisualVM 发挥出其真正价值，在一线程序员内心扎根并提升**定位率，**就需要对 VisualVM 的能力持续宣讲，让使用 VisualVM 定位问题的经典场景持续渗透。所以接下来我就会带你从官方的<a href="https://visualvm.github.io/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1347">宣传语</a>中，通过 All-in-One（多合一）这个关键词，与你重温经典的 VisualVM。</p>
<h4 data-nodeid="1204">1.VisualVM 四个选项卡都有哪些能力？</h4>
<p data-nodeid="1205">我们先回顾下，原生 VisualVM 四个选项卡都有哪些能力。</p>
<ul data-nodeid="1206">
<li data-nodeid="1207">
<p data-nodeid="1208"><strong data-nodeid="1355">概述</strong>：此界面展示进程相关的概览信息、JVM 参数和系统属性。因为企业中开发人员是无法直观查看线上配置，所以概述选项卡很好地展示进程使用的线上环境信息，可以让我们快速确定是否是环境问题造成的线上问题。</p>
</li>
<li data-nodeid="1209">
<p data-nodeid="1210"><strong data-nodeid="1360">监视</strong>：此界面展示实时的 CPU 使用情况、JVM 堆信息，以及 Java 8 的元空间信息、类加载情况、进程中的线程情况。此页面的监视数据不支持追溯历史，所以追溯历史可以使用 CAT 等 APM 工具解决。监视界面的执行 GC、堆 Dump 可以以最快速度进行现场保留。</p>
</li>
<li data-nodeid="1211">
<p data-nodeid="1212"><strong data-nodeid="1365">线程</strong>：此页界面展示实时运行的各个线程情况，主要包括线程在一段时间内的运行状况，通过不同颜色的标识线程分别处于的 5 种常见状态：运行、休眠、等待、驻留、监视；右上角“线程 Dump”按钮，可以对线程进行快照操作。</p>
</li>
<li data-nodeid="1213">
<p data-nodeid="1214"><strong data-nodeid="1370">抽样器</strong>：此界面对进程使用的 CPU 和内存进行采样，进行性能分析。CPU 抽样，通过方法的视角，查看方法占用 CPU 的情况；内存抽样，通过类名归类和线程使用内存的视角，进行内存分配的调优。</p>
</li>
</ul>
<p data-nodeid="1215">原始的四个选项卡，可以说是将 Java 命令集成为可视化的工具。这只是 VisualVM 多合一生态的冰山一角，在揭开真正的庐山面目前，我先由浅入深地问你三个问题，来测试你掌握 VisualVM 的能力。</p>
<ul data-nodeid="1216">
<li data-nodeid="1217">
<p data-nodeid="1218">你的 VisualVM 是不是只有四个选项卡？</p>
</li>
<li data-nodeid="1219">
<p data-nodeid="1220">你有系统地学习过 VisualVM 的插件体系吗？</p>
</li>
<li data-nodeid="1221">
<p data-nodeid="1222">你用 VisualVM 的插件都解决了什么问题呢？</p>
</li>
</ul>
<p data-nodeid="1223">我的调研结果是：很多一线开发人员可以回答第一个问题，但能回答上来用插件解决哪些经典场景的则凤毛麟角。</p>
<p data-nodeid="1224"><strong data-nodeid="1379">我认为插件是 VisualVM 的精华，是多合一（All-in-One）的具体实践，插件化带来的好处是更加轻量级的内核，更加多样化的生态。</strong></p>
<p data-nodeid="1225">经过前几课时的学习，你会发现所有 APM 工具都会选择插件化的实现方案，这种实现都借鉴于 VisualVM。所以可以说 VisualVM 是插件化的先驱者，插件化也是至今 VisualVM 还依然有很多受众的重要原因之一。</p>
<p data-nodeid="1226">既然插件这么重要，那接下来我就带你系统地学习 VisualVM 的插件体系。</p>
<h4 data-nodeid="1227">2.VisualVM 的插件体系</h4>
<p data-nodeid="1228">因为国内大厂基本使用的 Java 版本就是两个版本，Java 8 和 Java 11（Java 8 存量远大于 Java 11）。</p>
<ul data-nodeid="1229">
<li data-nodeid="1230">
<p data-nodeid="1231">对于使用 Java 8 的用户，VisualVM 已经在 JDK 的 bin 目录中了。但当你打开插件仓库，会发现没有可以安装的插件，是因为插件的地址已失效，需要我们手动更新官方的插件中心地址。</p>
</li>
<li data-nodeid="1232">
<p data-nodeid="1233">对于使用 Java 11 的用户，VisualVM 在 JDK9 后，已不再为 Oracle JDK 的一部分，我们需要去<a href="https://visualvm.github.io/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1388">VisualVM</a>官网进行下载。</p>
</li>
</ul>
<p data-nodeid="1234">经过一些小曲折，总算是安装好了。目前 VisualVM 历经了十几年的发展，总共有 20 个多插件，根据插件类型可以分为以下 8 种。</p>
<ul data-nodeid="1235">
<li data-nodeid="1236">
<p data-nodeid="1237">Profiling（性能剖析）：Startup Profiler、<strong data-nodeid="1395">BTrace Workbench</strong></p>
</li>
<li data-nodeid="1238">
<p data-nodeid="1239">Sources（代码资源）：VisualVM-GoToSource</p>
</li>
<li data-nodeid="1240">
<p data-nodeid="1241">Tools（工具）：Threads Inspector、VisualVM-JConsole、VisualVM-BufferMonitor、<strong data-nodeid="1406">VisualVM-TDA-Module</strong>、KillApplication、<strong data-nodeid="1407">VisualVM-Coherence</strong>、Visual GC、VisualVM-MBeans</p>
</li>
<li data-nodeid="1242">
<p data-nodeid="1243">Platform（监控）：VisualVM-Extensions</p>
</li>
<li data-nodeid="1244">
<p data-nodeid="1245">Security（安全）：VisualVM-Security</p>
</li>
<li data-nodeid="1246">
<p data-nodeid="1247">Tracer（追踪）：Tracer-Monitor Probes、Tracer-Collections Probes、Tracer-JavaFX Probes、Tracer-IO Probes、Tracer-Swing Probes、Tracer-Jvmstat Probes、Tracer-JVM Probes</p>
</li>
<li data-nodeid="1248">
<p data-nodeid="1249">UI（页面增强）：OQL Syntax Support</p>
</li>
<li data-nodeid="1250">
<p data-nodeid="1251">Libraries（类库）：GraalJS</p>
</li>
</ul>
<p data-nodeid="1252">看到插件的名字，凭借大家的专业度，我想你肯定能“顾名思义”。比如：Startup Profiler插件是在应用程序的启动阶段进行剖析的工具；GoToSource 插件是支持源代码导航和展示。</p>
<p data-nodeid="1253">那每个插件我就不过多解释了，大家在实际工作过程中，根据实际场景，通过插件类型和说明，不停地实战总结经验即可。不过需要注意的是，如果你使用的是 2.0 以上版本的 VisualVM，在部分场景下第三方插件（上文中已加粗）可能有兼容问题。</p>
<p data-nodeid="1254">VisualVM 的每个插件都非常经典，掌握了每个工具就像掌握了十八般兵器，你便可以“纵横江湖”（处理线上所有的问题）了。</p>
<h4 data-nodeid="1255">3.实战案例：Threads Inspector 定位线上死循环线程</h4>
<p data-nodeid="1256">这里我以 Threads Inspector 定位线上死循环（无法退出）的线程为例，与你分享 Threads Inspector 插件如何通过<strong data-nodeid="1422">可视化线程堆栈</strong>的实现，快速解决死循环问题。</p>
<p data-nodeid="1257">场景是这样的：一套进程混补老服务需要扩容新节点，但是扩容的新节点从 CAT 监控上看，即使凌晨在没有任何请求的时间，总有一些 Http Server 线程无法释放。</p>
<p data-nodeid="1258">这时一线开发人员想用 Arthas 来进行线程 Dump 分析，但发现新节点 Arthas 还没有安装完毕。由于是凌晨，发现问题后流量也全部切至老节点。我建议已开发人员先不要打扰 SRE 同学，先尝试使用 VisualVM 安装 Threads Inspector 插件来定位死循环线程。</p>
<p data-nodeid="1259">Threads Inspector 是工具类插件，VisualVM 在安装此插件后，“线程”选项卡的底部就会增加一个新部分，可以实时分析一个或多个线程的堆栈轨迹。</p>
<ul data-nodeid="1260">
<li data-nodeid="1261">
<p data-nodeid="1262">我们一起先通过“线程”选项卡，在带有 Http Server 前缀的线程中<strong data-nodeid="1431">找到</strong>那些一直处于运行状态且无法退出的流量线程。</p>
</li>
<li data-nodeid="1263">
<p data-nodeid="1264">然后抽样选取少量线程，在底部 Threads Inspector 插件视图中进行<strong data-nodeid="1437">堆栈轨迹的跟踪</strong>，发现有少量用户查看的票务请求打到了新节点。</p>
</li>
<li data-nodeid="1265">
<p data-nodeid="1266">获取票务信息需要通过 SDK 与第三方<strong data-nodeid="1443">建立 Websocket 连接</strong>。如果建立连接失败，不仅异常日志没有打印，而且还会一直处于无限重试建立连接的状态，直到建立连接成功。</p>
</li>
<li data-nodeid="1267">
<p data-nodeid="1268">再次通过 Threads Inspector 插件提供的运行堆栈轨迹，我们很轻松地<strong data-nodeid="1449">找到了相应的代码</strong>块。</p>
</li>
<li data-nodeid="1269">
<p data-nodeid="1270">最后<strong data-nodeid="1455">结合 SDK 文档</strong>，我们查明新扩容的节点是由于没有对第三方 IP 增加白名单导致的。</p>
</li>
</ul>
<p data-nodeid="1271">所以我们在联系 SRE 时，直接将结果同步，在“加白”处理后，再观察类似请求线程，发现很快就完成了与第三方建立连接，也解决了这一线上问题。</p>
<p data-nodeid="1272">可以看出 VisualVM 的接入成本远小于其他 APM 工具。在特定的场景下，相较于 Arthas 需要开发人员主动输入命令的方式，VisualVM 只需要“点点点”，即可引领一线开发人员处理线上问题。</p>
<h3 data-nodeid="1273">小结与思考</h3>
<p data-nodeid="1274">今天的课程，我先带你学习了 VisualVM 的落地方案。虽然 VisualVM 使用 JMX 规范，接入非常简单，但与其他 APM 工具一样，控制安全风险非常必要。</p>
<p data-nodeid="1275">接下来，带你系统地梳理了 VisualVM 的学习路径。VisualVM 的强大在于插件化生态，所以你的学习重点也应该是 VisualVM 的插件系统。</p>
<p data-nodeid="1276">受限于中文学习资料的缺乏，我建议你先阅读官方英文文献，初步了解插件。之后在问题出现时，可以像刚刚讲解的 Threads Inspector 实践一样，根据问题的类型选择适合的插件去针对性实践。在提升自身实践经验同时，多加总结思考，做到循序渐进地掌握每一个插件。</p>
<p data-nodeid="2042" class="te-preview-highlight">你负责的应用服务应该都开放了 JMX 端口，所以你便可以开始尝试使用 VisualVM 进行线上问题定位。那么请你回顾一下过往的线上问题，哪些场景可以使用 VisualVM 可以进行线上问题的快速定位呢？请在留言区写下你的思考，欢迎与我讨论。</p>

---

### 精选评论


