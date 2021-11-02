<p data-nodeid="1421" class="">今天我们继续聊一聊关于 Serverless 应用安全生产的话题。</p>
<p data-nodeid="1422">上一讲，我们讨论了 Serverless 应用安全性面临的挑战，以及主要风险，相信你对 Serverless 应用的安全性已经有了初步的了解，这一讲我就带你针对性地规避这些风险。</p>
<p data-nodeid="4384">我们先简单回顾一下上一讲中 10 个主要的风险：</p>
<p data-nodeid="5752" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/92/44/Ciqc1GARKLOAQ01ZAAKkVYv1kfc133.png" alt="图片1.png" data-nodeid="5755"></p>





<p data-nodeid="1452">针对这 10 个风险，我总结了一些应对措施，主要涉及减少代码漏洞、正确使用访问控制、增强数据安全防护、提升应用可观测性四个方面。</p>
<h3 data-nodeid="1453">减少应用代码漏洞</h3>
<p data-nodeid="1454">和传统应用一样，Serverless 应用的很多安全风险都是代码漏洞导致的，因此正确编程、减少应用代码漏洞，就可以避免很多安全风险。接下来，我分享一些自己的实践经验，希望给你启发。</p>
<ul data-nodeid="1455">
<li data-nodeid="1456">
<p data-nodeid="1457"><strong data-nodeid="1604">不要相信任何用户输入</strong></p>
</li>
</ul>
<p data-nodeid="1458">由于 Serverless 应用的攻击面更多，攻击手段更复杂，所以你永远不要相信任何输入，也不能对输入进行有效性假设。这就要求你验证用户输入或触发器数据（比如用户参数），或执行 SQL 时对参数进行预处理。</p>
<ul data-nodeid="1459">
<li data-nodeid="1460">
<p data-nodeid="1461"><strong data-nodeid="1609">使用安全的第三方依赖</strong></p>
</li>
</ul>
<p data-nodeid="1462">为了避免第三方依赖带来的风险，你需要使用安全的依赖，做法包括：</p>
<ol data-nodeid="1463">
<li data-nodeid="1464">
<p data-nodeid="1465">维护项目的依赖及版本，比如 pakcage.json 或 pom.xml 的作用都是维护依赖版本；</p>
</li>
<li data-nodeid="1466">
<p data-nodeid="1467">扫描依赖项，找出并去掉存在漏洞的版本，并且我建议你将依赖版本扫描作为 CI/CD 的一部分，这样在发布时就可以去掉有漏洞的依赖；</p>
</li>
<li data-nodeid="1468">
<p data-nodeid="1469">删除不必要的依赖，尤其是当 Serverless 应用不需要此依赖时；</p>
</li>
<li data-nodeid="1470">
<p data-nodeid="1471">仅从可信赖的资源中使用第三方依赖；</p>
</li>
<li data-nodeid="1472">
<p data-nodeid="1473">将不推荐使用的依赖更新到最新版本。</p>
</li>
</ol>
<p data-nodeid="1474">另外，你也可以从一些公开的安全漏洞披露平台中浏览某个依赖有没有安全漏洞，比如：</p>
<ul data-nodeid="1475">
<li data-nodeid="1476">
<p data-nodeid="1477"><a href="https://nodesecurity.io/advisories" data-nodeid="1619">Node.js 模块中的已知漏洞</a>；</p>
</li>
<li data-nodeid="1478">
<p data-nodeid="1479"><a href="https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=java" data-nodeid="1623">Java 中的已知漏洞</a>；</p>
</li>
<li data-nodeid="1480">
<p data-nodeid="1481"><a href="https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=python" data-nodeid="1627">Python 相关技术中的已知漏洞</a>。</p>
</li>
<li data-nodeid="1482">
<p data-nodeid="1483"><strong data-nodeid="1632">正确地处理程序异常</strong></p>
</li>
</ul>
<p data-nodeid="1484">对于生产环境中的代码，需要避免打印冗长的错误信息。在可以自定义错误的场景下（比如通过 API 提供 HTTP 请求响应），建议你只为用户提供简单的错误消息，不要显示有关任何内部实现的堆栈或环境变量的详细信息。</p>
<p data-nodeid="1485">此外应用也要进行合适的错误处理，当出现意外输入时，也要确保代码不会 “挂起”，并且需要严格测试所有边界情况，考虑所有可能导致函数超时的输入。</p>
<ul data-nodeid="1486">
<li data-nodeid="1487">
<p data-nodeid="1488"><strong data-nodeid="1638">通过 API 网关确保 API 安全</strong></p>
</li>
</ul>
<p data-nodeid="1489">使用 Serverless 最常见的场景之一就是构建 API。</p>
<p data-nodeid="1490"><strong data-nodeid="1644">基于 Serverless 构建 API 时，我建议你将函数和 API 网关一起使用。</strong> API 网关可以用来创建、发布、维护、监控和保护 API，通过 API 网关可以很容易实现 API 的流量控制、监控报警、版本记录等功能。几乎所有 FaaS 平台都支持通过 API 网关触发函数执行。基于 API 网关触发器时，用户发起一个请求后，请求首先会被 API 网关处理，然后 API 网关将请求转发到函数，这样就达到了保护 API 的目的。</p>
<p data-nodeid="1491"><img src="https://s0.lgstatic.com/i/image/M00/90/4F/Ciqc1GAKr9uAM8zoAAGDujlP-TA203.png" alt="Drawing 0.png" data-nodeid="1647"></p>
<div data-nodeid="1492"><p style="text-align:center">API 网关触发器</p></div>
<h3 data-nodeid="1493">正确使用访问控制</h3>
<p data-nodeid="1494">访问控制是使用云产品时非常重要的功能，尤其是在规模比较大的团队中，访问控制可以用来限制不同用户能够操作的资源。</p>
<p data-nodeid="1495">但很多同学对访问控制使用得不合理，这会增加 Serverless 应用的安全风险。接下来，我带你学习怎么正确使用访问控制，进而提升 Serverless 应用的安全性。</p>
<ul data-nodeid="1496">
<li data-nodeid="1497">
<p data-nodeid="1498"><strong data-nodeid="1654">为用户和角色配置最小的权限</strong></p>
</li>
</ul>
<p data-nodeid="1499">为了减少访问凭证泄漏带来的风险，建议你使用访问控制来管理每个函数具有的权限，并确保每个函数都有且仅有运行时所需要的最小权限。这个过程可能会很复杂，但对应用安全是十分必要的。要正确地配置函数的权限，你需要对云厂商的访问控制有深入的了解（我在 10 讲已经介绍了，如果你忘了，可以复习一下）。</p>
<p data-nodeid="1500">除了人工配置每个函数的权限，你也可以使用一些开源的工具来保证函数的最小权限，比如&nbsp;PureSec 的最小权限插件 <a href="https://github.com/puresec/serverless-puresec-cli/" data-nodeid="1659">serverless-puresec-cli</a>，不过该插件只适用于 AWS Lambda。</p>
<ul data-nodeid="1501">
<li data-nodeid="1502">
<p data-nodeid="1503"><strong data-nodeid="1664">使用临时访问凭证</strong></p>
</li>
</ul>
<p data-nodeid="1504">当使用代码对云产品进行编程访问时，就会用到访问凭证（即 AccessKeyId 和 AccessKeySecret）。在 Serverless 函数中，建议你尽可能使用临时访问凭证，而不是直接在代码中配置固定的访问凭证。因为临时访问凭证有过期时间，这样就降低了访问凭证泄漏后的风险，进而提高 Serverless 安全性。</p>
<p data-nodeid="1505">举个例子，假如你需要在函数中访问读写存储中的文件，可以使用固定访问凭证，代码如下：</p>
<pre class="lang-javascript" data-nodeid="1506"><code data-language="javascript"><span class="hljs-comment">/**
 * 使用固定访问凭证
 */</span>
<span class="hljs-keyword">const</span> accessKeyId = <span class="hljs-string">'xxx'</span>;
<span class="hljs-keyword">const</span> accessKeySecret = <span class="hljs-string">'xxx'</span>;
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
    <span class="hljs-comment">// 初始化 OSS 客户端</span>
    <span class="hljs-keyword">const</span> store = oss({
        accessKeyId,
        accessKeySecret,
        <span class="hljs-attr">stsToken</span>: securityToken,
        <span class="hljs-attr">bucket</span>: <span class="hljs-string">'role-test'</span>,
        <span class="hljs-attr">region</span>: <span class="hljs-string">'oss-cn-beijing'</span>
    });
    <span class="hljs-comment">// 获取文件</span>
    <span class="hljs-keyword">const</span> result = <span class="hljs-keyword">await</span> store.get(<span class="hljs-string">'hello.txt'</span>);
    <span class="hljs-keyword">return</span> result.content.toString();
};
</code></pre>
<p data-nodeid="1507">也可以使用函数执行时的临时访问凭证，代码如下：</p>
<pre class="lang-javascript" data-nodeid="1508"><code data-language="javascript"><span class="hljs-comment">/**
 * 使用临时访问凭证
 */</span>
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
    <span class="hljs-comment">// 获取函数计算的临时访问凭证</span>
    <span class="hljs-keyword">const</span> accessKeyId = context.credentials.accessKeyId;
    <span class="hljs-keyword">const</span> accessKeySecret = context.credentials.accessKeySecret;
    <span class="hljs-keyword">const</span> securityToken = context.credentials.securityToken;
    <span class="hljs-comment">// 初始化 OSS 客户端</span>
    <span class="hljs-keyword">const</span> store = oss({
        accessKeyId,
        accessKeySecret,
        <span class="hljs-attr">stsToken</span>: securityToken,
        <span class="hljs-attr">bucket</span>: <span class="hljs-string">'role-test'</span>,
        <span class="hljs-attr">region</span>: <span class="hljs-string">'oss-cn-beijing'</span>
    });
    <span class="hljs-comment">// 获取文件</span>
    <span class="hljs-keyword">const</span> result = <span class="hljs-keyword">await</span> store.get(<span class="hljs-string">'hello.txt'</span>);
    <span class="hljs-keyword">return</span> result.content.toString();
};
</code></pre>
<p data-nodeid="1509">由于固定访问凭证是写在代码中的，代码泄漏了，访问凭证就泄漏了。而临时访问凭证是每次执行时生成的，所以更安全。</p>
<h3 data-nodeid="1510">增强数据安全防护</h3>
<p data-nodeid="1511">在上一讲中，我提到了云上安全责任分担模型，应用所有者要负责运行在云上的应用、代码、数据……的安全，所以接下来我就带你了解几种云上数据安全防护的方案。</p>
<ul data-nodeid="1512">
<li data-nodeid="1513">
<p data-nodeid="1514"><strong data-nodeid="1674">对云上的数据进行加密</strong></p>
</li>
</ul>
<p data-nodeid="1515">云上的数据主要有两部分：云上存储的数据（比如存放在云存储中的数据）；云上传输的数据(比如函数间进行数据通信)。</p>
<p data-nodeid="1516">为了避免敏感数据从云存储等基础云服务中泄漏，很多云厂商强化了云存储的配置，提供了多重身份认证、数据加密等功能。比如 AWS S3 支持 KMS 加密、阿里云对象存储也支持阿里云 KMS 加密。</p>
<p data-nodeid="1517">为了避免中间人攻击，导致传输过程中的数据泄漏，建议你使用 TLS 协议对函数通信、网络请求等传输数据进行加密。</p>
<ul data-nodeid="1518">
<li data-nodeid="1519">
<p data-nodeid="1520"><strong data-nodeid="1681">对应用配置进行加密</strong></p>
</li>
</ul>
<p data-nodeid="1521">几乎每个应用都会有很多配置，比如数据库账号密码、访问凭证等，这些信息会在不同的应用、代码、环境变量中进行传输，只要其中有一个环节有漏洞，就会导致数据泄漏。应用配置数据泄漏，会造成很严重的后果， 因此一定要避免在代码中定义明文配置，要对这些配置进行加密。那么如何对配置进行加密呢？你可以使用云厂商提供的一些密钥管理服务，例如：</p>
<ol data-nodeid="1522">
<li data-nodeid="1523">
<p data-nodeid="1524">AWS Secrets Manager；</p>
</li>
<li data-nodeid="1525">
<p data-nodeid="1526">Azure 密钥保管库；</p>
</li>
<li data-nodeid="1527">
<p data-nodeid="1528">阿里云密钥管理服务；</p>
</li>
<li data-nodeid="1529">
<p data-nodeid="1530">Google Cloud KMS。</p>
</li>
</ol>
<ul data-nodeid="1531">
<li data-nodeid="1532">
<p data-nodeid="1533"><strong data-nodeid="1690">使用 Serverless 相关的身份认证服务</strong></p>
</li>
</ul>
<p data-nodeid="1534">为了避免没有权限的用户访问数据，你也需要对访问请求进行身份认证。我上一讲一直强调： Serverless 应用由成百上千离散的函数组成，身份认证十分复杂，所以我不建议你构建自己的身份认证方案，建议用 Serverless 相关运行身份验证功能，比如：</p>
<ol data-nodeid="1535">
<li data-nodeid="1536">
<p data-nodeid="1537">AWS Cognito 或单点登录；</p>
</li>
<li data-nodeid="1538">
<p data-nodeid="1539">AWS API Gateway；</p>
</li>
<li data-nodeid="1540">
<p data-nodeid="1541">阿里云 API 网关；</p>
</li>
<li data-nodeid="1542">
<p data-nodeid="1543">腾讯云 API 网关；</p>
</li>
<li data-nodeid="1544">
<p data-nodeid="1545">Azure 应用服务身份认证和授权；</p>
</li>
<li data-nodeid="1546">
<p data-nodeid="1547">Google Firebase 身份认证。</p>
</li>
</ol>
<p data-nodeid="1548">另外，在不能进行交互式身份认证（比如通过 API 认证）的情况下，你应该使用安全的访问凭证、客户端证书等方式进行身份认证。</p>
<h3 data-nodeid="1549">提升应用可观测性</h3>
<p data-nodeid="1550">由于 Serverless 应用可观测性不足，遇到安全攻击可能难以第一时间发现，发现安全问题也难以快速定位并恢复，所以导致 Serverless 面临着比传统应用更大的安全风险。下面就为你介绍如何提升 Serverless 应用的可观测性。</p>
<ul data-nodeid="1551">
<li data-nodeid="1552">
<p data-nodeid="1553"><strong data-nodeid="1704">记录函数日志并设置报警</strong></p>
</li>
</ul>
<p data-nodeid="1554">传统应用直接运行在服务器上，你可以直接将日志输出到机器上，遇到问题后可以直接登录机器查看日志。但 Serverless 屏蔽了底层机器，所以你需要将日志输出到统一的日志存储服务，这样才能更方便查看问题。比如 Lambda 可以将日志输出到 CloudWatch Logs，函数计算可以将日志输出到日志服务。</p>
<p data-nodeid="1555">统一记录日志后，建议你对日志配置报警，这样才能及时发现问题。为了实时了解函数的运行情况，你需要针对函数的并发、节流、超时等异常情况设置报警。另外我也强烈建议你针对云账号的账单设置报警，尽可能避免预期外的费用。</p>
<ul data-nodeid="1556">
<li data-nodeid="1557">
<p data-nodeid="1558"><strong data-nodeid="1710">使用配置审计监控资源的配置变化</strong></p>
</li>
</ul>
<p data-nodeid="1559">由于 Serverless 函数是在云上运行的，函数本身就有很多配置，比如内存、CPU 和并发限制等。此外，函数依赖的云服务也有很多配置，比如数据库规格等。一旦某个配置发生变化，很可能对应用运行造成影响，<strong data-nodeid="1716">所以我建议你</strong>使用云厂商提供的配置审计功能来检测云资源的配置变化，比如 AWS Config 和阿里云配置审计。</p>
<p data-nodeid="1560">基于配置审计，你可以持续监控函数配置或代码更改，并通过规则判断配置更改是否符合预期，我总结了一些常见的规则，希望给你启发。</p>
<ol data-nodeid="1561">
<li data-nodeid="1562">
<p data-nodeid="1563"><strong data-nodeid="1722">检测函数是否通过控制台创建：</strong> 对于复杂的 Serverless 应用，通常会有一套完善的 CI/CD 流程，而不是直接在控制台上创建函数，因此可以使用此规则找出在控制台上创建的不合规的函数。</p>
</li>
<li data-nodeid="1564">
<p data-nodeid="1565"><strong data-nodeid="1727">检测使用同一个角色的多个函数：</strong> 通常我们会保持一个函数一个角色的原则，这样才能更好地实现函数权限最小化。</p>
</li>
<li data-nodeid="1566">
<p data-nodeid="1567"><strong data-nodeid="1732">检测配置了多个不同触发器的函数：</strong> 通常我们会保持一个函数一个触发器，这样才能保证函数单一职责，如果函数有多个触发器，则会增加函数的攻击面。</p>
</li>
<li data-nodeid="1568">
<p data-nodeid="1569"><strong data-nodeid="1739">检测使用通配符 (*) 权限的函数：</strong> 通配符通常意味着更多的权限，这就违背了函数最低权限的原则。</p>
</li>
</ol>
<p data-nodeid="1570">除了上面 4 条，你还可以定义很多自己的规则。总的来说，基于配置审计，你可以持续发现资源配置的变化，一旦某个资源配置不符合预期，你能及时发现并进行改进了。</p>
<ul data-nodeid="1571">
<li data-nodeid="1572">
<p data-nodeid="1573"><strong data-nodeid="1744">使用操作审计记录云上所有操作事件</strong></p>
</li>
</ul>
<p data-nodeid="1574">对于云上的函数，除了配置变化，你还要关注资源本身的变化，比如函数创建删除，以及其他任何资源的创建、删除和更新。如果某个用户误删或恶意删除了某个资源，你要及时收到报警并处理，尽可能避免影响变大。<strong data-nodeid="1750">这时你就可以使用云厂商的操作审计服务了，比如 AWS</strong>CloudTrail、阿里云操作审计。</p>
<p data-nodeid="1575">操作审计持续记录了云账号在云上的所有操作日志，操作日志通常包含了资源的变更时间以及操作者，你可以对这些数据进行实时导出并进行分析处理，这样基于操作日志，你就可以对敏感操作（比如删除函数）进行报警，并且也能基于操作日志进行问题排查。</p>
<h3 data-nodeid="1576">总结</h3>
<p data-nodeid="1577">今天，我针对 11 讲提及的10个主要风险，分享了一些实践经验，希望让你有所收获，因为11 讲和 12 讲的信息偏多，所以我整理了一张导图，帮你回顾上两讲的重点内容：</p>
<p data-nodeid="1578"><img src="https://s0.lgstatic.com/i/image/M00/92/40/CgqCHmARAaKAdEvcAAzvYrkQgpQ210.png" alt="Serverless 安全生产-竖.png" data-nodeid="1756"></p>
<p data-nodeid="2877">由于国外 Serverless 技术起步比较早，大家对安全性也比较重视，所以国外已经有一些第三方的 Serverless 安全性相关的产品，比如 <a href="https://www.paloaltonetworks.com/prisma/cloud/cloud-workload-protection-platform" data-nodeid="2882">Prisma Cloud</a>、<a href="https://www.checkpoint.com/products/cloudguard-serverless-security/" data-nodeid="2886">CloudGuard</a>、<a href="https://www.aquasec.com/products/serverless-container-functions/" data-nodeid="2890">Aquasec</a>，而国内目前几乎是一片空白，很多安全防护方案都需要开发者自己建设，所以“怎么提高 Serverless 应用的安全性？&nbsp;”在我看来，就是多了解云厂商 Serverless 相关产品的功能和使用方式，<strong data-nodeid="2895">用好，用对才是最重要。</strong></p>
<p data-nodeid="2878" class=""><img src="https://s0.lgstatic.com/i/image/M00/92/4F/CgqCHmARKJiAS7WoAAE8KFXevfk360.png" alt="玩转 Serverless 架构11金句.png" data-nodeid="2898"></p>

<p data-nodeid="2504">本节课我留给你的作业是，除了本节课中讲到的，你还知道哪些针对 Serverless 架构中的安全风险的解决方案呢？感谢你的阅读，我们下一讲见。</p>

---

### 精选评论


