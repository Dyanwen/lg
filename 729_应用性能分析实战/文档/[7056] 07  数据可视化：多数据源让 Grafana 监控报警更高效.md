<p data-nodeid="1229" class="">这一讲我将带领你学习可视化监控套件 Grafana。Grafana 是一个开源的数据可视化的平台，所以它既不会监控应用，也不会产生监控数据，更不会对接原始数据进行分析存储。<strong data-nodeid="1365">它仅专注数据可视化本身</strong>。</p>
<p data-nodeid="1230">本节内容，会先通过与上一节 Kibana 可视化套件对比，来讲述 Grafana 的核心设计。然后以实战视角，分享应用服务如何使用 Grafana 生成应用监控指标和落地实践。</p>
<p data-nodeid="1231">为什么提起 Grafana 的设计，我就会关联上一节中的 APM 可视化平台 Kibana 呢？</p>
<p data-nodeid="1232">有用过这两个可视化套件的同学可能觉得这两个产品有种说不出来的相似。比如仪表盘的构建基础都有丰富的可视化面板，开发人员都是在仪表盘通过动态的调整布局完成最终的仪表盘构建。</p>
<blockquote data-nodeid="1233">
<p data-nodeid="1234">虽然 Grafana 默认主题是暗色，Kibana 主题是明亮色，但产品主题的外表差异，并不能掩盖内在相似这一事实。</p>
</blockquote>
<p data-nodeid="1235">那回到问题的答案。其相似的原因便是：Grafana 就是以 Kibana 3.0 的分支为基准进行产品演进的。</p>
<p data-nodeid="1236">目前 Kibana 已经完全融入 Elastic 生态，而 Grafana 也站在巨人的肩膀上，在多数据源和面板，以及相应插件的生态上，有着非常亮眼的优势。</p>
<ul data-nodeid="1237">
<li data-nodeid="1238">
<p data-nodeid="1239">如果是通过 Elastic Stack 对日志进行分析，或是为 Elasticsearch 集群提供可视化，那应该选取 Kibana 作为可视化监控平台；</p>
</li>
<li data-nodeid="1240">
<p data-nodeid="1241">反之，如果被监控的场景需要聚合各种数据源，那应该选取 Grafana。</p>
</li>
</ul>
<h3 data-nodeid="1242">Grafana 的核心设计</h3>
<p data-nodeid="1243">接下来，我们就选取 Grafana 核心的两个设计：整合数据源和报警进行讲述。</p>
<h4 data-nodeid="1244">1.整合数据源</h4>
<p data-nodeid="1245">目前 Grafana 官方已经支持了超过 30 多个数据源，像 InfluxDB、Prometheus、Elasticsearch 的数据源驱动都已经内置在安装包中，即使被监控的数据源没有在官方的支持列表中，也可以通过社区强大的插件生态，快速完成集成。</p>
<p data-nodeid="1246">Grafana 在监控服务器节点的场景中，最常见的数据源是 Prometheus 和 Zabbix。</p>
<ul data-nodeid="1247">
<li data-nodeid="1248">
<p data-nodeid="1249"><strong data-nodeid="1382">Prometheus</strong></p>
</li>
</ul>
<p data-nodeid="1250">前者 Prometheus 数据源已经在官方的数据源支持列表中，</p>
<blockquote data-nodeid="1251">
<p data-nodeid="1252">对于 Prometheus 如何集成指标数据，Grafana 在对接数据源后如何展示，你可以通过<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=410#/detail/pc?id=4337&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1389">《16 | 指标体系：Prometheus 如何更完美地显示指标体系？》</a>进行学习。</p>
</blockquote>
<p data-nodeid="1253">Prometheus 对容器资源的监控、支持非常友好，必定是未来容器化监控的主流监控基建。</p>
<ul data-nodeid="1254">
<li data-nodeid="1255">
<p data-nodeid="1256"><strong data-nodeid="1395">Zabbix</strong></p>
</li>
</ul>
<p data-nodeid="1257">但当下对物理节点的监控诉求还是很需要的。接下来，我将讲述 Grafana 如何通过插件生态，集成 Zabbix 数据源，完成对物理节点的性能指标监控。</p>
<p data-nodeid="1258">Zabbix 是监控分布式系统的解决方案，其具备物理机性能监控和对网络状况的监控。使用客户端、服务端模式进行收集数据，即物理机安装 Zabbix Agent 客户端采集数据传送给服务端，通过 web 服务直接对接服务端完成展示。由于 Zabbix web 展示监控数据不理想，所以解决方案是通过 Grafana 集成 Zabbix 数据源完成监控数据的展示。</p>
<p data-nodeid="1259">Zabbix 数据源属于非官方支持的数据源，需要从开放的插件生态引入，重启 Grafana 服务后，即可在数据源中看到 Zabbix 数据源。</p>
<h4 data-nodeid="1260">2.多维度报警</h4>
<p data-nodeid="1261">相较于 Kibana 需要将用户升级为黄金用户以上级别才支持报警功能，Grafana 报警功能是完全开放的，只要将 Grafana 升级到 4.0 版本以上即可。</p>
<p data-nodeid="1262">在报警配置界面，可以将报警信息发送至聊天工具 Slack、事件管理平台 PagerDuty 等在 Github 主流的报警媒介；也可以按照企业微信和钉钉的 API 开发文档，对接国内主流办公平台。通过配置报警媒介并集成监控数据源后，便能依据可视化面板的指标配置报警的规则了。</p>
<p data-nodeid="1263">Grafana<strong data-nodeid="1407">报警规则的设计</strong>是通过后台的一个调度程序，定时地通过查询引擎识别出当前可视化面板中的指标是否触及报警规则。如果触及，则调用报警媒介下达报警通知。</p>
<p data-nodeid="1264">由于 Grafana 是可视化平台，监控数据的获取方式可视作为“拉模式”，且报警设计通过后台调度程序完成，所以此方案的设计来报警会稍许滞后。</p>
<p data-nodeid="1265">下图为在平均响应时间超过 250 毫秒时，进行报警规则呈现。</p>
<p data-nodeid="1266"><img src="https://s0.lgstatic.com/i/image6/M00/37/53/CioPOWB2rFaAc3AqAAGWdoD5bZo254.png" alt="Drawing 0.png" data-nodeid="1412"></p>
<p data-nodeid="1267">除了<strong data-nodeid="1422">可视化</strong>面板这一特色，Grafana 基于<strong data-nodeid="1423">多种数据源数据指标</strong>的联动，更具备了多维度报警这一优势。基于丰富的数据源驱动的支持，一线开发通过 Grafana 便能快速完成应用服务的指标监控。这便是 Grafana 区别其他可视化监控平台的重要设计。</p>
<h3 data-nodeid="1268">应用服务配置 Grafana 指标</h3>
<p data-nodeid="1269">互联网已经逐步告别了风口流量红利时代，现今留下来的多为行业独角兽。此时去研究竞争对手，还不如深耕自我实力，通过应用服务的提升，来提升用户使用体验。那如何深耕、提升自我的应用服务呢？</p>
<p data-nodeid="1270">Grafana 就是不二之选，受益于协同开发和丰富数据源的设计，应用服务的干系人可以集思广益，一站式地将应用服务的核心指标监控梳理在一起。</p>
<p data-nodeid="1271">下面以常见的用户行为监控和核心场景监控为例，与你分享我的实践经验。</p>
<h4 data-nodeid="1272">1.用户行为监控</h4>
<p data-nodeid="1273"><strong data-nodeid="1432">1）用户生命周期监控</strong></p>
<p data-nodeid="1274">常见生命周期分为如下图的 5 层，从用户接触产品到遗忘产品的整个过程中，我们可以通过监控分析用户在不同阶段的表现，有的放矢地优化产品体验，引导用户完成运营策略。</p>
<p data-nodeid="1275"><img src="https://s0.lgstatic.com/i/image6/M01/37/4B/Cgp9HWB2rGCACurDAAFei_kgyOE245.png" alt="Drawing 1.png" data-nodeid="1436"></p>
<p data-nodeid="1276">以电商 App 为例，通过 Grafana 快速构建起用户生命周期监控。</p>
<ul data-nodeid="1277">
<li data-nodeid="1278">
<p data-nodeid="1279">导入期：在新用户调用注册接口时，通过埋点“来源”字段绘制折线，从而分析运营活动的拉新效果。</p>
</li>
<li data-nodeid="1280">
<p data-nodeid="1281">成长期：新用户会通过新手任务来熟悉 App 的关键用法，并在这一过程中转化为核心用户。所以新手任务中每个步骤的友好度/易操作性非常重要。开发人员可以通过在各个步骤中埋点，通过柱状图分析出每个步骤的操作时间，从而推算出这一过程是否存在操作流程问题以及任务设置问题。</p>
</li>
<li data-nodeid="1282">
<p data-nodeid="1283">成熟期：成熟期的用户标识有很多，如近三十天通过 App 购买过商品的即为活跃用户，我们可以通过集成订单表中的数据源完成监控。</p>
</li>
<li data-nodeid="1284">
<p data-nodeid="1285">休眠&amp;流失期：如多少天内不活跃的用户便可以标识为此层用户，开发人员可以通过集成操作日志数据源来完成此层用户的监控。</p>
</li>
</ul>
<p data-nodeid="1286">通过监控用户的这一完整生命周期，项目的干系人可以用相同的数据基准来驱动应用目标实现，并在建立数据的基础上完成自动机械化的精细运营策略投放，进而提升效率。</p>
<p data-nodeid="1287"><strong data-nodeid="1448">2）活跃用户监控</strong></p>
<p data-nodeid="1288">如果把用户生命周期比作在用户时间轴向的监控，那活跃用户监控就是对时间轴上每个点的切面监控。我们可以使用 Grafana 基于单点登录项目的用户会话数据源，来完成活跃用户的监控。</p>
<p data-nodeid="1289">在当下应用服务容器化的背景下，企业通过对用户活跃度监控，可以更合理地根据用户活跃情况调整资源重心。</p>
<ul data-nodeid="1290">
<li data-nodeid="1291">
<p data-nodeid="1292">在用户活跃度较高时，将资源倾斜到用户侧，扩容应用服务集群，给用户带来更好的产品体验。</p>
</li>
<li data-nodeid="1293">
<p data-nodeid="1294">当用户活跃度较低时，资源倾斜到算法测，如通过机器学习完成用户画像的计算。研发也可以在此时上线发布新功能，对核心场景进行压测，确保线上用户负面反馈最小。</p>
</li>
</ul>
<h4 data-nodeid="1295">2.核心场景监控</h4>
<p data-nodeid="1296">业务场景的监控反映着用户使用产品的真实感受，有助于帮助开发人员优化相应的接口，进而提升系统的性能和稳定性。</p>
<p data-nodeid="1297">核心场景监控可以通过如下四个指标进行开展。</p>
<ul data-nodeid="2464">
<li data-nodeid="2465">
<p data-nodeid="2466">业务场景：通过一组相关联的数据或事件，来反映自然语言描述的业务场景。</p>
</li>
<li data-nodeid="2467">
<p data-nodeid="2468">业务假设：通过业务场景可能在未来发生的事情做一系列的业务假设。</p>
</li>
<li data-nodeid="2469">
<p data-nodeid="2470">监控方案：以业务场景的数据指标为依据，监控业务假设是否成立。</p>
</li>
<li data-nodeid="2471" class="te-preview-highlight">
<p data-nodeid="2472">关键举措：当假设成立时，启动一些应急举措。</p>
</li>
</ul>
<p data-nodeid="2473">下面我们以电商应用服务的通用场景为例，来简述核心场景的监控配置：</p>




<table data-nodeid="1310">
<thead data-nodeid="1311">
<tr data-nodeid="1312">
<th align="center" data-nodeid="1314">业务场景</th>
<th data-nodeid="1315">业务假设</th>
<th data-nodeid="1316">监控方案</th>
<th data-nodeid="1317">关键举措</th>
</tr>
</thead>
<tbody data-nodeid="1322">
<tr data-nodeid="1323">
<td align="center" data-nodeid="1324">购买</td>
<td data-nodeid="1325"><ul><li>购买成功率小于四个九</li><li>平均耗时大于几百毫秒</li></ul></td>
<td data-nodeid="1326"><ul><li>接口埋点</li></ul><ul><li>Grafana 折线图</li></ul></td>
<td data-nodeid="1327">故障跟进</td>
</tr>
<tr data-nodeid="1328">
<td align="center" data-nodeid="1329">优惠券</td>
<td data-nodeid="1330"><ul><li>抵扣粒度大于一定比例</li></ul><ul><li>某个商品红包大于一定金额</li></ul></td>
<td data-nodeid="1331"><ul><li>巡检数据</li></ul></td>
<td data-nodeid="1332">下架商品</td>
</tr>
<tr data-nodeid="1333">
<td align="center" data-nodeid="1334">库存</td>
<td data-nodeid="1335"><ul><li>库存小于一定比例</li></ul></td>
<td data-nodeid="1336"><ul><li>库存数据源监控</li></ul></td>
<td data-nodeid="1337">商家提醒</td>
</tr>
</tbody>
</table>
<h3 data-nodeid="1338">Grafana 落地实践</h3>
<p data-nodeid="1339">Grafana 是非常易用的可视化监控平台，在多数据的支撑下，项目干系人快速完成应用服务的指标监控。但是绘制可视化面板的指标查询命令非常占用资源，且每个应用的数据只有相关干系人才可以修改。</p>
<p data-nodeid="1340">所以在下面的落地实践内容里，我将向你分享“预防拖垮核心资源”和“权限隔离”的实践经验。</p>
<h4 data-nodeid="1341">1.预防拖垮核心资源</h4>
<p data-nodeid="1342">仪表盘中存在大量的面板，每个可视化面板都会调用聚合命令查询指标数据，所以仪表盘数据的展示非常占用系统资源。而且监控数据的查询时效要求相对较低，所以 Grafana 需要向所有项目干系人屏蔽“开放数据源配置”页面，由开发统一配置数据源的对应读库或统计库。</p>
<p data-nodeid="1343"><strong data-nodeid="1523">这样的优势在于所有使用 Grafana 的人员依然可以如以前一样，且使用的数据源有专人配置，所有指标查询语句均不会造成核心资源的占用。</strong></p>
<p data-nodeid="1344">开发人员可以根据数据源的类型梳理出对应资源占用少的数据源配置。</p>
<ul data-nodeid="1345">
<li data-nodeid="1346">
<p data-nodeid="1347">对于关系型数据源：可以使用读写分离架构下的读库或离线库。</p>
</li>
<li data-nodeid="1348">
<p data-nodeid="1349">对于搜索数据源：通过实时计算引擎将原始数据加工成指标数据进行存储。</p>
</li>
<li data-nodeid="1350">
<p data-nodeid="1351">对于监控数据源：监控数据源专为 APM 而生，只需要拆解需求、查询即可，无须过度优化。</p>
</li>
</ul>
<h4 data-nodeid="1352">2.用户权限隔离</h4>
<p data-nodeid="1353">在落地 APM 可视化监控的过程中，我建议适当做一些防止数据泄漏的建设。固定用户类型权限设置显得过于简陋，而集成公司的登录系统和权限系统又不符合 APM 建设的重点。</p>
<p data-nodeid="1354">那关于 Grafana 应该如何合理地控制权限呢？</p>
<p data-nodeid="1355">我建议使用官方的“组织-用户”模式进行权限隔离。“组织”可看作是“应用服务”，用户则是通过在“组织”页面菜单中的邀请功能添加进来的。这样，不同组织的人员登录 Grafana 便能看到自己权限内部的仪表盘，当一个用户隶属于多个组织时，通过左上角的按钮即可进行组织的切换。</p>
<h3 data-nodeid="1356">小结与思考</h3>
<p data-nodeid="1357">今天的课程，我带你学习了开箱即用的可视化平台 Grafana。对比上一讲的 Kibana，Grafana 在整合多数据源支持和报警能力上都有非常出色的表现。通过 Grafana 我们可以快速完成应用程序的指标监控，比如用户生命周期监控、业务的核心场景监控等。</p>
<p data-nodeid="1358">相信你在工作中使用 Grafana 落地监控可视化的场景非常多。那么在仪表盘中，你整合了哪些数据源？完成了哪些监控场景？过程中得到了哪些收益，又遇到了哪些问题呢？</p>
<p data-nodeid="1359" class="">欢迎在评论区写下你的思考，期待与你讨论～</p>

---

### 精选评论


