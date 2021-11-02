<p data-nodeid="1517" class="">今天我想和你分享自己在工作中总结的一些 Serverless 实践经验。</p>
<p data-nodeid="1518">我在开篇词中提到，Serverless 比较新，还处于飞速发展的阶段，业界对 Serverless 的经验输出还比较少，更不用说大型项目的实践经验了。这让很多同学在用 Serverless 时无经验可循，在架构设计和代码开发时很容易踩坑。</p>
<p data-nodeid="1519">虽然我在 09~14 中从性能、安全、成本几个角度带你详细了解了 Serverless 的实践经验，但是并没有全面总结过这些内容，所以在“开发进阶篇”的最后一讲，我想从这几个角度，对前面的知识点做个总结，提炼出 Serverless 应用开发的最佳实践，让你“温故而知新”。</p>
<h3 data-nodeid="1520">怎么提升开发质量</h3>
<p data-nodeid="1521">怎么提升开发质量，让应用代码更 Serverless 化？我提炼了 7 条经验。</p>
<ul data-nodeid="1522">
<li data-nodeid="1523">
<p data-nodeid="1524"><strong data-nodeid="1690">使用代码描述基础设施</strong></p>
</li>
</ul>
<p data-nodeid="1525">在使用 Serverless 时，你经常会用到很多云服务，比如网关、VPC、数据库、角色权限等，如果每次手动去控制台创建这些云服务资源，很容易出错，还不方便扩展。所以建议你用代码的方式描述这些资源，这样当代码更新时也能同步更新云资源，这种技术方案也被称为“基础设施即代码（Infrastructure as Code，IaC）”。基于 IaC，你可以很方便地自动化管理云资源（包括创建、更新和删除）。</p>
<p data-nodeid="1526">比较流行的开源的 IaC 方案是 <a href="https://www.terraform.io/" data-nodeid="1695">terraform</a>，它支持 AWS、Google Cloud、阿里云等多种云平台，另外云厂商也会有自己的 IaC 能力，比如 AWS <a href="https://aws.amazon.com/cn/serverless/sam/" data-nodeid="1699">SAM</a> 和阿里云<a href="https://www.aliyun.com/product/ros" data-nodeid="1703">资源编排</a>：</p>
<ol data-nodeid="1527">
<li data-nodeid="1528">
<p data-nodeid="1529">如果业务中使用多云，建议你用 terraform；</p>
</li>
<li data-nodeid="1530">
<p data-nodeid="1531">如果你主要用国内的云（比如阿里云），使用资源编排更方便，因为 terraform 没有国内源，国内云厂商对其支持也没有国外云厂商完善。</p>
</li>
</ol>
<ul data-nodeid="1532">
<li data-nodeid="1533">
<p data-nodeid="1534"><strong data-nodeid="1710">将业务逻辑抽离到入口函数之外</strong></p>
</li>
</ul>
<p data-nodeid="1535">在入口函数中只应该处理函数参数，不应该处理业务逻辑。把业务逻辑抽离到入口函数之外，可以降低代码和云服务的耦合度，并且入口函数之外的代码还能在容器重用时被重复利用，不用重新初始化（这一点是我在“03｜基础入门：编写你的第一个 Serverless 应用”和“08｜单元测试：Serverless 应用如何进行单元测试？”中反复提及的）。</p>
<p data-nodeid="1536">好的例子：</p>
<pre class="lang-javascript" data-nodeid="1537"><code data-language="javascript"><span class="hljs-comment">// 初始化数据库连接</span>
<span class="hljs-keyword">const</span> connection = <span class="hljs-keyword">await</span> mysql.createConnection({<span class="hljs-attr">host</span>:<span class="hljs-string">'localhost'</span>, <span class="hljs-attr">user</span>: <span class="hljs-string">'root'</span>, <span class="hljs-attr">database</span>: <span class="hljs-string">'test'</span>});
<span class="hljs-comment">// 查询用户信息</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getUser</span>(<span class="hljs-params">name</span>) </span>{
  <span class="hljs-keyword">const</span> [rows, fields] = <span class="hljs-keyword">await</span> connection.execute(<span class="hljs-string">'SELECT * FROM `table` WHERE `name` = ?'</span>, [name]);
  <span class="hljs-keyword">return</span> rows;
}
exports.handler = <span class="hljs-keyword">async</span> (event, callback) =&gt; {
  <span class="hljs-keyword">const</span> name = event.user.name;
  <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> getUser(name);
  <span class="hljs-keyword">return</span> res;
}
</code></pre>
<p data-nodeid="1538">不好的例子：</p>
<pre class="lang-javascript" data-nodeid="1539"><code data-language="javascript">exports.handler = <span class="hljs-keyword">async</span> (event, callback) =&gt; {
   <span class="hljs-keyword">const</span> name = event.user.name;
   <span class="hljs-comment">// 初始化数据库连接</span>
  <span class="hljs-keyword">const</span> connection = <span class="hljs-keyword">await</span> mysql.createConnection({<span class="hljs-attr">host</span>:<span class="hljs-string">'localhost'</span>, <span class="hljs-attr">user</span>: <span class="hljs-string">'root'</span>, <span class="hljs-attr">database</span>: <span class="hljs-string">'test'</span>});
  <span class="hljs-comment">// 查询用户信息</span>
  <span class="hljs-keyword">const</span> [rows, fields] = <span class="hljs-keyword">await</span> connection.execute(<span class="hljs-string">'SELECT * FROM `table` WHERE `name` = ?'</span>, [name]);
  <span class="hljs-keyword">return</span> rows;
}
</code></pre>
<p data-nodeid="1540">你可以看到，在不好的例子中，初始化数据库连接的业务逻辑就放在了入口函数中，这样不利于代码维护。</p>
<ul data-nodeid="1541">
<li data-nodeid="1542">
<p data-nodeid="1543"><strong data-nodeid="1718">函数单一职责</strong></p>
</li>
</ul>
<p data-nodeid="1544">每个函数只负责一件事情，这样才更方便扩展。比如，你的函数里面有 switch 语句，那就说明你的函数不是单一职责，一个函数负责多件事情的问题在于：扩展时会扩展多个业务功能，而不是单一功能，这样会浪费资源。函数职责单一后，方便你对其编写单源测试，这涉及 08 讲的内容。</p>
<ul data-nodeid="1545">
<li data-nodeid="1546">
<p data-nodeid="1547"><strong data-nodeid="1723">函数不能调用其他函数</strong></p>
</li>
</ul>
<p data-nodeid="1548">我看到很多经验不足的同学经常在一个函数中调用另一个函数，这样做会让函数执行成本翻倍，还会导致函数间逻辑耦合。函数与函数间的调用，本质上是函数通信问题，如果两个函数需要通信，我建议你使用消息队列或事件总线等方案。</p>
<ul data-nodeid="1549">
<li data-nodeid="1550">
<p data-nodeid="1551"><strong data-nodeid="1728">为异步函数设置调用目标</strong></p>
</li>
</ul>
<p data-nodeid="1552">异步函数执行过程是异步的，你无法立即知道执行结果，所以要为异步函数设置异步调用目标，以保存执行结果，并对异常情况进行处理。关于函数的异步调用以及调用目标，我在“04 | 运行原理： Serverless 应用是怎么运行的？”中提及过。</p>
<ul data-nodeid="1553">
<li data-nodeid="1554">
<p data-nodeid="1555"><strong data-nodeid="1735">编写详细的测试用例</strong></p>
</li>
</ul>
<p data-nodeid="1556">测试是保障应用质量和稳定性的重要手段，所以建议你为应用编写详细的单元测试和集成测试（关于怎么编写单元测试，我在 08 讲中也详细讲解了）。</p>
<ul data-nodeid="1557">
<li data-nodeid="1558">
<p data-nodeid="1559"><strong data-nodeid="1740">避免使用基于连接的服务</strong></p>
</li>
</ul>
<p data-nodeid="1560">基于连接的服务主要是 RDS 数据库等，建立连接会带来额外的冷启动耗时，并且要在内存中维护连接池。</p>
<p data-nodeid="1561">Serverless 应用由于无状态的特点，并不适合使用连接，因为代码执行完毕后运行环境就会被销毁，所以可能经常要初始化连接，它比较适合的是使用服务，比如直接通过 Restful API 去访问数据库等服务，这也是为什么 DynamoDB、表格存储等在 Serverless 中很受欢迎。</p>
<h3 data-nodeid="1562">怎么提升应用性能</h3>
<p data-nodeid="1563">作为开发者，我们肯定不满足只完成业务功能，在完成基本代码开发后，还会关注应用性能，毕竟性能越好，用户体验也就越好。所以我总结了提升应用性能的 5 条建议。</p>
<ul data-nodeid="1564">
<li data-nodeid="1565">
<p data-nodeid="1566"><strong data-nodeid="1748">为函数设置合适的内存大小</strong></p>
</li>
</ul>
<p data-nodeid="1567">函数执行所需内存是预先指定的。通常在一定范围内，内存越大执行速度越快，但到达某个阈值后，内存增长就不会再带来性能的增长了。并且随着内存增加，代码执行所消耗的成本也是线性增加的。</p>
<p data-nodeid="1568">所以你要为函数选择合适的内存，你可以通过链路追踪或一些开源产品分析函数性能并找到最佳内存，具体怎么分析和确定最佳内存，我在 “09｜性能优化：如何提升 Serverless 应用的性能？”中提到过。</p>
<ul data-nodeid="1569">
<li data-nodeid="1570">
<p data-nodeid="1571"><strong data-nodeid="1754">选择合适的编程语言</strong></p>
</li>
</ul>
<p data-nodeid="1572">不同编程语言的运行性能有所不同。通常来说，Node.js、Python 等解释型语言会比 Java、.NET 等编译型语言性能更好，主要是编译型语言冷启动耗时更长。并且函数执行时间越长，成本也越高。</p>
<p data-nodeid="1573">所以，如果你的应用对延迟非常敏感，容易引发频繁的冷启动，建议你选择解释型语言。如果应用对性能要求不高，也没有流量波峰波谷，建议你选择最熟悉的编程语言。</p>
<ul data-nodeid="1574">
<li data-nodeid="1575">
<p data-nodeid="1576"><strong data-nodeid="1760">优化代码</strong></p>
</li>
</ul>
<p data-nodeid="1577">影响函数性能的因素主要有冷启动和代码逻辑及其依赖。虽然不同业务代码逻辑不同，但一些优化方案是通用的，主要就是利用容器重用，最大程度减少冷启动与代码初始化耗时，比如：</p>
<ul data-nodeid="1578">
<li data-nodeid="1579">
<p data-nodeid="1580">初始化后，将代码中引用到的任何外部配置或依赖性存储下来；</p>
</li>
<li data-nodeid="1581">
<p data-nodeid="1582">减少每次调用时变量、对象的重新初始化，比如可以通过全局变量、静态变量、单例等来实现；</p>
</li>
<li data-nodeid="1583">
<p data-nodeid="1584">重复利用上次函数调用期间建立的连接，如 HTTP、数据库等。</p>
</li>
<li data-nodeid="1585">
<p data-nodeid="1586"><strong data-nodeid="1768">使用链路追踪分析应用性能</strong></p>
</li>
</ul>
<p data-nodeid="1587">为了了解应用中各个组件（包括一个或多个函数以及依赖的服务）的性能，建议你使用链路追踪去分析应用的性能，通过链路追踪可以观察函数冷启动和代码执行各个阶段的执行时间，通过适当的配置后，还能追踪各个依赖项执行耗时。大部分云厂商也都有自己的链路追踪服务，比如 AWS X-Ray 和阿里云链路追踪，此外也还有一些开源的链路追踪产品。如果你没有定制化需求，我比较推荐你直接使用云厂商提供的链路追踪云服务，因为云厂商自己的产品通常配套使用更方便。</p>
<ul data-nodeid="1588">
<li data-nodeid="1589">
<p data-nodeid="1590"><strong data-nodeid="1773">减小代码的体积</strong></p>
</li>
</ul>
<p data-nodeid="1591">代码体积越小，冷启动耗时越短，函数性能越高。代码体积包括我们编写的代码以及代码依赖，在部署函数时需要将代码和依赖打包部署到 Serverless FaaS 平台。通常 FaaS 平台都对函数代码都有大小限制，一般为 50MB，因为体积越大冷启动耗时越长。所以建议你精简代码体积，尤其是精简依赖的大小，要避免部署不必要的依赖。比如 Node.js 应用，你可以使用 Webpack 来精简代码及依赖的体积。</p>
<h3 data-nodeid="1592">怎么增强系统稳定性</h3>
<p data-nodeid="1593">对线上应用来说，稳定性甚至比性能更重要。应用不稳定，性能再好也没用。当然，基于 Serverless 的应用，云厂商会负责计算、网络、存储等资源的稳定性，并且应用本身具备自动弹性伸缩能力，所以你面临的稳定性挑战要比传统应用少很多。但你还是可以通过架构上的一些优化，进一步提升应用稳定性。</p>
<ul data-nodeid="1594">
<li data-nodeid="1595">
<p data-nodeid="1596"><strong data-nodeid="1780">通过多地域部署实现多地域容灾</strong></p>
</li>
</ul>
<p data-nodeid="1597">如果你的应用对可用性要求很高，你可以多地域部署应用，这样一个地域服务不可用，可以切换到其他地域。此外你的应用也可能因法律合规要求，必须多地域部署，比如你的应用即有美国用户也有中国用户，美国的数据不允许存储在中国，你就要在美国也部署应用。</p>
<p data-nodeid="1598">多地域部署的复杂性主要在于：函数以及依赖的各种云服务都要多地域部署。比如你的应用依赖了数据库，则在美国部署应用，函数、触发器和数据库等都必须部署在美国。如果手动维护这些资源，就变得很麻烦，所以我建议你用 IaC 的方式来管理云上资源。</p>
<ul data-nodeid="1599">
<li data-nodeid="1600">
<p data-nodeid="1601"><strong data-nodeid="1786">通过多可用区实现单地域高可用</strong></p>
</li>
</ul>
<p data-nodeid="1602">一个地域通常会有多个可用区。假设你的应用部署在公网，没有使用 VPC，函数实例就是分布在多个可用区的，FaaS 平台可能在任意一个可用区执行你的函数。但如果你使用 VPC，则执行函数的可用区取决于其交换机对应的可用区。一个 VPC 可以创建多个交换机，每个交换机只能指定一个可用区，函数通过交换机来访问其他服务。所以在 VPC 内的函数，如果要实现高可用，建议你为函数指定多个不同可用区的交换机。这样一个可用区出故障，函数还可以在其他可用区执行。</p>
<ul data-nodeid="1603">
<li data-nodeid="1604">
<p data-nodeid="1605"><strong data-nodeid="1791">使用日志服务记录应用日志</strong></p>
</li>
</ul>
<p data-nodeid="1606">日志是我们监控应用运行状态和排查问题的重要依据。传统应用是部署在服务器上的，你可以登录服务器查看日志。但 Serverless 应用屏蔽了服务器，所以就要使用云厂商提供的日志服务来集中记录应用日志，比如 AWS CloudWatch Logs、阿里云日志服务。通过日志服务，可以更方便进行日志索引、查询和处理。</p>
<p data-nodeid="1607">对于 Restful API，建议你针对每个请求和响应都打印日志，这样更方便排查问题。</p>
<ul data-nodeid="1608">
<li data-nodeid="1609">
<p data-nodeid="1610"><strong data-nodeid="1797">设定应用指标和监控</strong></p>
</li>
</ul>
<p data-nodeid="1611">除了基本的业务指标外，Serverless 应用的指标还包括函数调用次数、执行时间、代码出错次数等，你需要为这些指标设置监控报警。</p>
<h3 data-nodeid="1612">怎么保障应用安全</h3>
<p data-nodeid="1613">关于如何提升 Serverless 应用的安全性，我在 “11｜安全生产（一）：Serverless 安全的主要风险是什么？”和 “12 | 安全生产（二）：如何提高 Serverless 应用的安全性？”中详细地带你了解过了，这里我想再强调一些重点内容，并对其他同学容易出现的问题简单说明和补充一下。</p>
<ul data-nodeid="1614">
<li data-nodeid="1615">
<p data-nodeid="1616"><strong data-nodeid="1806">不要相信任何用户输入</strong></p>
</li>
</ul>
<p data-nodeid="1617">由于 Serverless 的输入源更多，不仅包含用户输入，还有触发器输入，所以攻击方式和手段越来越多也越复杂，所以你不能相信任何用户输入，建议你过滤任何的用户输入。</p>
<ul data-nodeid="1618">
<li data-nodeid="1619">
<p data-nodeid="1620"><strong data-nodeid="1811">正确地处理程序异常</strong></p>
</li>
</ul>
<p data-nodeid="1621">虽然 Serverless 函数的声明周期通常很短暂，执行完毕就释放，可以降低程序异常带来的风险。但仍然建议你正确处理程序的异常，并且在生产环境中避免打印错误栈，不然日志泄漏后可能会带来安全隐患。</p>
<ul data-nodeid="1622">
<li data-nodeid="1623">
<p data-nodeid="1624"><strong data-nodeid="1816">对云上敏感数据进行加密</strong></p>
</li>
</ul>
<p data-nodeid="1625">为了避免数据泄漏，建议你对云上的存储的敏感数据以及需要传输的敏感数据进行加密，比如数据库账号秘密、机密文件等。</p>
<ul data-nodeid="1626">
<li data-nodeid="1627">
<p data-nodeid="1628"><strong data-nodeid="1821">为用户和角色配置最小的权限</strong></p>
</li>
</ul>
<p data-nodeid="1629">你的云账号下可能会有多个子用户和角色，建议你为用户和角色配置最小的权限，这样不仅可以避免用户或角色越权，还可以减少访问凭证泄漏后的风险。</p>
<ul data-nodeid="1630">
<li data-nodeid="1631">
<p data-nodeid="1632"><strong data-nodeid="1826">为函数设置最小执行权限</strong></p>
</li>
</ul>
<p data-nodeid="1633">为了避免函数权限过大访问不必要的云服务，带来安全风险，所以要为函数设置最小的执行权限。</p>
<ul data-nodeid="1634">
<li data-nodeid="1635">
<p data-nodeid="1636"><strong data-nodeid="1831">在代码中使用临时访问凭证</strong></p>
</li>
</ul>
<p data-nodeid="1637">你的代码中不应该使用任何长期的访问凭证，而应该使用函数执行角色的临时访问凭证。临时访问凭证可以直接从执行上下文中获取，比如：</p>
<pre class="lang-javascript" data-nodeid="1638"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
    <span class="hljs-comment">// 获取临时访问凭证</span>
    <span class="hljs-keyword">const</span> accessKeyId = context.credentials.accessKeyId;
    <span class="hljs-keyword">const</span> accessKeySecret = context.credentials.accessKeySecret;
    <span class="hljs-keyword">const</span> securityToken = context.credentials.securityToken;
};
</code></pre>
<ul data-nodeid="1639">
<li data-nodeid="1640">
<p data-nodeid="1641"><strong data-nodeid="1836">记录云上操作及资源变化</strong></p>
</li>
</ul>
<p data-nodeid="1642">为了方便排查问题、监控资源变化和资源配置变更，建议你使用操作审计和配置审计监控来监控云上操作及资源变化。</p>
<h3 data-nodeid="1643">怎么优化应用成本</h3>
<p data-nodeid="1644">作为团队 Leader，在进行技术选型时，除了要考虑开发效率、应用性能、稳定性和安全性几个方面，还要考虑技术成本。毕竟大部分情况下，我们开发的软件最终都是要追求商业价值。所以我在“13｜成本优化：Serverless 真的省钱吗？”这一讲中专门讲了成本相关的话题，这里我再重点总结其中的 5 条建议。</p>
<ul data-nodeid="1645">
<li data-nodeid="1646">
<p data-nodeid="1647"><strong data-nodeid="1843">提升函数性能</strong></p>
</li>
</ul>
<p data-nodeid="1648">因为 Serverless 应用是按执行次数和执行时消耗的内存等资源计费。因此函数性能越高，执行时间越短，成本也就越低。</p>
<ul data-nodeid="1649">
<li data-nodeid="1650">
<p data-nodeid="1651"><strong data-nodeid="1848">为函数设置超时时间</strong></p>
</li>
</ul>
<p data-nodeid="1652">为了避免函数因异常而无限运行，所以要为函数设置超时时间。因为 FaaS 是按执行时间计费的，函数执行时间越长费用越多，为函数设置合适的超时时间，可以避免产生不必要的费用。</p>
<ul data-nodeid="1653">
<li data-nodeid="1654">
<p data-nodeid="1655"><strong data-nodeid="1853">选择合适的云服务</strong></p>
</li>
</ul>
<p data-nodeid="1656">Serverless 应用往往依赖很多云服务，比如事件源、消息队列、数据库等，根据你的应用需求，比如请求规模、数据量、延迟等，使用不同云服务的架构总成本，不同云服务的成本可能会有较大差异。在进行架构设计时，要综合考虑应用需求与成本，进而决定使用何种云服务。比如数据库，通常 MySQL 数据库没有自动伸缩能力，而表格存储可以自动扩容，且 MySQL 是按实例付费，而表格存储是按请求量付费，所以对于请求量比较小的业务，选择表格存储成本更低。</p>
<ul data-nodeid="1657">
<li data-nodeid="1658">
<p data-nodeid="1659"><strong data-nodeid="1858">选择合适的计费方式</strong></p>
</li>
</ul>
<p data-nodeid="1660">通常 FaaS 平台的付费方式有按量付费和预付费。按量付费就是按实际函数执行次数和消耗的资源计费，预付费则与传统付费方式类似，预先支付费用购买资源生成函数实例。你可以根据应用特点选择合适的付费方式。比如你的应用流量具有周期性，有时候流量很大，有时候流量又很少，这时就适合按量付费。如果你的应用流量一直很平稳，则预付费可能成本更低。</p>
<ul data-nodeid="1661">
<li data-nodeid="1662">
<p data-nodeid="1663"><strong data-nodeid="1863">关注 FaaS 和 BaaS 等云服务的总成本</strong></p>
</li>
</ul>
<p data-nodeid="1664">对 Serverless 应用进行成本分析很容易陷入的误区就是，只考虑了 FaaS 的成本而忽略了 BaaS 的成本。所以你在进行成本分析和成本优化时，一定要将应用运行所依赖的 FaaS 和 BaaS 以及其他云服务都包含在内。</p>
<h3 data-nodeid="1665">总结</h3>
<p data-nodeid="1666">这一讲我分享了在开发 Serverless 应用过程中的一些实践经验。</p>
<ul data-nodeid="2315">
<li data-nodeid="2316">
<p data-nodeid="2317"><strong data-nodeid="2331">开发：</strong> 你需要根据 Serverless 应用的特点，按照 Serverless 的思维编写 Serverless 代码。</p>
</li>
<li data-nodeid="2318">
<p data-nodeid="2319"><strong data-nodeid="2336">性能：</strong> Serverless 的性能优化核心在于减少冷启动耗时。</p>
</li>
<li data-nodeid="2320">
<p data-nodeid="2321"><strong data-nodeid="2341">稳定性：</strong> 你可以通过多地域多可用区部署进一步提升稳定性，并通过日志服务等记录应用日志并配置监控报警指标。</p>
</li>
<li data-nodeid="2322">
<p data-nodeid="2323"><strong data-nodeid="2346">安全：</strong> Serverless 应用的安全风险主要是攻击面和攻击方式越来越复杂，所以需要对用户输入进行过滤、对云上数据进行加密、对用户和角色权限进行管控、对资源操作和变更进行监控。</p>
</li>
<li data-nodeid="2324">
<p data-nodeid="2325"><strong data-nodeid="2351">成本：</strong> Serverless 的成本包括 FaaS 成本以及依赖的云服务成本，你可以从这两方面进一步进行成本优化。</p>
</li>
</ul>
<p data-nodeid="3887" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/02/2C/Cgp9HWAc-YOAWaxcAAYd9jKtOg0867.png" alt="Serverless 实践经验 - 竖-01.png" data-nodeid="3890"></p>






<p data-nodeid="1679" class="">最后，关于如何更好地进行 Serverless 应用开发，我也还在持续探索中，所以这一讲的作业是：你还知道哪些 Serverless 的实践经验呢？欢迎在留言区分享，我们下一讲见。</p>

---

### 精选评论


