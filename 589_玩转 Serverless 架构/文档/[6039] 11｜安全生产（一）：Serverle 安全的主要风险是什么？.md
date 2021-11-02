<p data-nodeid="1265" class="">今天这一讲我想和你聊一聊关于 Serverless 应用安全生产的话题。</p>
<p data-nodeid="1266">根据 <a href="https://www.oreilly.com/" data-nodeid="1401">O'Reilly</a> 的最新调查显示，企业使用 Serverless 最关心的就是安全问题。很多同学也问过我：用 Serverless 开发的应用安全吗？答案是：Serverless 是否安全取决于怎么去做，虽然本质上应用会更安全，但这建立在你正确地进行架构设计、代码实现的基础上。出于很多同学不知道怎么去保证 Serverless 应用的安全，所以我准备了今天这一讲，<strong data-nodeid="1406">希望你能提高安全意识，掌握这份安全生产指导手册。</strong></p>
<p data-nodeid="1267">对于一个应用来说，安全性非常重要，为了让你明白 Serverless 面临的安全风险，并学会如何去解决这些风险，我将用两讲的时间，讲解 Serverless 安全问题，包括 Serverless 安全面临的挑战、Serverless 安全的主要风险，以及如何提升 Serverless 的安全性。</p>
<p data-nodeid="1268">今天我们就先来了解前两个话题。</p>
<h3 data-nodeid="1269">Serverless 安全面临的挑战</h3>
<p data-nodeid="1270">从软件开发的角度来看，使用 Serverless 架构，你可以专注产品功能的实现，完全不用考虑底层服务器、操作系统以及软件运行环境。基于 Serverless 架构的应用，你也无须为服务和操作系统安装安全补丁，这些工作由提供 Serverless 服务的云厂商负责。</p>
<p data-nodeid="1271">那么对于使用 Serverless 开发的应用，在安全性方面，哪些部分由云厂商负责，哪些部分由开发者负责呢？</p>
<p data-nodeid="1272"><img src="https://s0.lgstatic.com/i/image2/M01/08/44/CgpVE2AKrtGAHEPJAA9UpTZveIw457.png" alt="Drawing 0.png" data-nodeid="1414"></p>
<div data-nodeid="1273"><p style="text-align:center">Serverless 安全责任分担模型</p></div>
<p data-nodeid="1274">从图中可以看到，在 Serverless 架构中，Serverless&nbsp;提供商（云厂商）负责维护数据中心、计算、存储、网络以及操作系统等云本身的安全性，而运行在云上的应用、代码、数据等，需要应用所有者自己负责。</p>
<p data-nodeid="1275">虽然使用 Serverless，你不用关心底层资源的安全性，减少一部分安全工作，但这也带来了一些新的挑战：</p>
<ul data-nodeid="1276">
<li data-nodeid="1277">
<p data-nodeid="1278"><strong data-nodeid="1420">攻击面越来越广</strong></p>
</li>
</ul>
<p data-nodeid="1279">由于 Serverless 中函数的数据来自多种数据源（比如 HTTP 触发器、消息队列、云存储、云服务……）这就极大增加了攻击面，特别是当数据源消息结构非常复杂时，传统的 Web 防火墙方式就很难对数据进行校验。</p>
<ul data-nodeid="1280">
<li data-nodeid="1281">
<p data-nodeid="1282"><strong data-nodeid="1425">攻击方式越来越复杂</strong></p>
</li>
</ul>
<p data-nodeid="1283">数据源的增多，不仅增加了攻击面，还使攻击方式也越来越复杂。除了传统的 DDoS 攻击、数据注入等攻击方式，Serverless 还面临着事件注入、流程劫持等新的攻击方式。又因为 Serverless 概念还比较新，所以很多开发者和架构师很难理解这些攻击方式，在面对攻击时比较缺乏经验。</p>
<ul data-nodeid="1284">
<li data-nodeid="1285">
<p data-nodeid="1286"><strong data-nodeid="1430">可观测性不足</strong></p>
</li>
</ul>
<p data-nodeid="1287">可观测性（Oberservability）是指你的系统发生的所有事情都能被观测到，其中包括日志、监控指标、链路追踪等。因为 Serverless 对开发者来说是屏蔽了底层基础设施，且应用是由分布式的函数组成，所以 Serverless 应用的可观测性比传统应用更复杂。</p>
<ul data-nodeid="1288">
<li data-nodeid="1289">
<p data-nodeid="1290"><strong data-nodeid="1435">传统安全测试方法不适用</strong></p>
</li>
</ul>
<p data-nodeid="1291">对 Serverless 应用进行安全测试更为复杂，尤其是当 Serverless 应用依赖了第三方服务或云服务（如云数据库、云存储等）。虽然在单元测试时这些依赖可以被模拟，但进行安全测试却不能模拟。此外，传统的安全测试工具（主要有DAST、SAST、IAST三类）对 Serverless 应用也不适用。</p>
<ol data-nodeid="1292">
<li data-nodeid="1293">
<p data-nodeid="1294"><strong data-nodeid="1441">DAST（动态应用程序安全性测试）：</strong> 这类工具主要是扫描 HTTP 接口进行安全测试。但很多 Serverless 应用的事件源都不是 HTTP 触发器，因此 DAST 工具很难对非 HTTP 的 Serverless 应用进行扫描。</p>
</li>
<li data-nodeid="1295">
<p data-nodeid="1296"><strong data-nodeid="1446">SAST（静态应用程序安全性测试）：</strong> 这类工具主要是通过分析代码语法、结构、接口、控制流等来检测程序等漏洞，由于 Serverless 应用使用了很多触发器和云服务，很多静态分析工具并没有考虑这些情况，所以很容易出现误报。</p>
</li>
<li data-nodeid="1297">
<p data-nodeid="1298"><strong data-nodeid="1451">IAST（交互式应用程序安全性测试）：</strong> 这类工具通过将流量代理到测试服务器等方式，进而可以得到更高的准确率、更低的误报率，但这种方式对于非 HTTP 接口或依赖了云服务的 Serverless 应用依旧不适用，你很难进行流量代理。</p>
</li>
</ol>
<ul data-nodeid="1299">
<li data-nodeid="1300">
<p data-nodeid="1301"><strong data-nodeid="1455">传统安全防护方案不兼容</strong></p>
</li>
</ul>
<p data-nodeid="1302">基于 Serverless 的应用无法访问物理机或虚拟机，因此你无法随意部署传统的安全层，比如端点防护、Web 防火墙等，大多数传统的安全防护方案不兼容 Serverless 应用。</p>
<p data-nodeid="1303"><img src="https://s0.lgstatic.com/i/image2/M01/08/42/Cip5yGAKruSAJep6AA4Z0pvq8e4553.png" alt="Drawing 1.png" data-nodeid="1459"></p>
<div data-nodeid="1304"><p style="text-align:center">传统安全防护方案</p></div>
<p data-nodeid="1305"><img src="https://s0.lgstatic.com/i/image2/M01/08/44/CgpVE2AKru2ADCkDAAgIPXf-L0M018.png" alt="Drawing 2.png" data-nodeid="1462"></p>
<div data-nodeid="1306"><p style="text-align:center">传统安全防护方案与 Serverless 架构不兼容</p></div>
<p data-nodeid="1307">总的来说，上面几个挑战让 Serverless 应用面临了一些新的安全风险，接下来我就带你了解 Serverless 安全性的主要风险。</p>
<h3 data-nodeid="1308">Serverless 安全的主要风险</h3>
<p data-nodeid="1309">我总结了 10 种风险类型，其严重程度由高到低（需要注意的是，这 10 种风险类型并不是 Serverless 安全风险的全部）：函数事件注入、身份认证无效、应用配置不安全、用户或角色权限过高、函数日志和监控能力不足、第三方依赖不安全、敏感信息泄露、DDoS和资损、函数执行、流程操纵、错误处理不当。</p>
<ul data-nodeid="1310">
<li data-nodeid="1311">
<p data-nodeid="1312"><strong data-nodeid="1469">函数事件数据注入</strong></p>
</li>
</ul>
<p data-nodeid="1313">数据注入是最常见的安全风险，比如 SQL 注入、XSS 注入。不过在传统应用中，这些数据注入都是用户输入的数据。而 Serverless 应用的数据并不局限于用户输入，比如 API 的请求参数。</p>
<p data-nodeid="1314">Serverless 应用还有大量的触发器提供了丰富的数据源，这些数据源都可以触发 Serverless 函数的执行，比如定时触发器、NoSQL 事件（如 AWS DynamoDB、阿里云表格存储等）、云存储事件（如 AWS S3、阿里云对象存储等）消息队列事件（如 AWS SQS、阿里云 MNS 等）、SMS消息通知（PUSH通知，电子邮件等）。</p>
<p data-nodeid="1315">这些不同数据源的数据，会以参数的形式传递给 Serverless 函数，作为函数的输入，不同数据源的通信协议、编码方式、数据格式也不尽相同。丰富的数据源不仅增加了潜在的攻击面，也增加了安全防护的复杂性。因为这些数据可能包含了攻击者的输入或其他危险输入，开发者很难判断哪些是正常数据，哪些是危险数据。比如在传统应用中，你可以通过消息头、请求参数来判断数据危险与否（例如请求中必须带上认证后的 cookie 或 token），但 Serverless 架构中的非 HTTP 数据源就很难简单通过请求信息来判断了。</p>
<ul data-nodeid="1316">
<li data-nodeid="1317">
<p data-nodeid="1318"><strong data-nodeid="1476">身份认证无效</strong></p>
</li>
</ul>
<p data-nodeid="1319">Serverless 架构的应用是由几十甚至上百个函数组成，每个函数实现特定的业务功能，这些函数组合完成整体业务逻辑。一些函数可能会公开其 Web API，需要进行身份认证，另一些则可能只允许内部调用，所以不用进行身份认证，这就使 Serverless 应用的整体身份认证变得复杂，你要为每个函数及事件源提供合理的身份认证机制，一不小心就会出错。</p>
<p data-nodeid="1320">举个例子，如果一个函数提供了公开的 API 给用户使用，并且该 API 也有正确的身份认证逻辑，用户访问函数后，函数内部首先会从云存储读文件，然后将读取后的数据作为输入源调用内部函数，内部函数无须身份认证，如果云存储没有设置合适的身份认证，攻击者可能直接向云存储注入数据进行攻击。</p>
<p data-nodeid="1321"><img src="https://s0.lgstatic.com/i/image/M00/90/4F/Ciqc1GAKrvyAcvzwAAlW-r1KZiM211.png" alt="Drawing 3.png" data-nodeid="1481"></p>
<div data-nodeid="1322"><p style="text-align:center">身份认证无效的示例</p></div>
<ul data-nodeid="1323">
<li data-nodeid="1324">
<p data-nodeid="1325"><strong data-nodeid="1485">应用配置不安全</strong></p>
</li>
</ul>
<p data-nodeid="1326">在云上运行的应用，尤其是 Serverless 应用，经常会使用到很多应用配置，比如数据库账号，另外你还会基于配置来实现环境区分、功能开关等逻辑。通常我们会将这些配置放在云厂商提供的配置中心（比如阿里云的 ACM），甚至直接存放在云存储中。</p>
<p data-nodeid="1327">配置对应用运行的影响非常大，最常见的问题就是：对于配置中心或存放应用配置的云存储授权不当。这很可能造成应用敏感信息泄露，或没有权限的用户不小心修改配置导致应用无法运行。</p>
<ul data-nodeid="1328">
<li data-nodeid="1329">
<p data-nodeid="1330"><strong data-nodeid="1491">用户或角色权限过高</strong></p>
</li>
</ul>
<p data-nodeid="1331">在第 10 讲中我提到，Serverless 应用应该秉持最小权限的原则。也就是仅给函数提供其执行时所必需的权限。不过 Serverless 应用是由几十上百个函数组成，要给每个函数都设置最小权限，管理函数的角色和权限就非常复杂，所以很多开发者或团队为了方便，就为函数设置了统一的较大的权限，这就让函数可以访问很多非必要的云服务，如果一个应用中函数权限过高，那单个函数的漏洞就可能造成系统级灾难。</p>
<p data-nodeid="1332">比如，某个函数只需要读数据库，如果给该函数赋予了写数据库的权限，则一旦该函数有漏洞，攻击者就可以利用该函数向数据库写入数据了，进而造成整个应用的异常。</p>
<ul data-nodeid="1333">
<li data-nodeid="1334">
<p data-nodeid="1335"><strong data-nodeid="1497">函数日志和监控能力不足</strong></p>
</li>
</ul>
<p data-nodeid="1336">虽然云厂商都对函数提供了日志和监控功能，但这些工具都很新，提供的能力也有限，要想利用这些工具来可视化 Serverless 架构的运行情况还非常困难。</p>
<p data-nodeid="1337">传统架构下，对于一个比较复杂的应用，你可能就只有 10 台服务器，这时你可以比较容易知道它们是否正常运行，但当你有成百上千个函数时，就很难确定是否所有函数都按预期执行了，而且这些函数还会产生大量的日志，你也很难确定这些日志哪些是重要的，函数级别的日志分析工具也还比较缺乏。</p>
<p data-nodeid="1338">函数日志和监控能力的不足，就会导致面临攻击时，你就很难针对 Serverless 攻击进行报警，也很难通过日志去分析、排查并解决问题。</p>
<ul data-nodeid="1339">
<li data-nodeid="1340">
<p data-nodeid="1341"><strong data-nodeid="1504">第三方依赖不安全</strong></p>
</li>
</ul>
<p data-nodeid="1342">通常 Serverless 函数都是执行单个离散任务的一小段代码，很多时候为了完成业务逻辑，函数就要依赖第三方软件包、开源库，甚至通过 API 调用第三方远程 Web 服务。</p>
<p data-nodeid="1343">虽然 Serverless 函数的执行环境是一个安全隔离的环境，但如果函数的第三方依赖不安全，也很可能导致函数不安全。</p>
<ul data-nodeid="1344">
<li data-nodeid="1345">
<p data-nodeid="1346"><strong data-nodeid="1510">敏感信息泄露</strong></p>
</li>
</ul>
<p data-nodeid="1347">随着应用规模和复杂性的增长，应用需要维护越来越多的敏感信息，比如访问凭证（AccessKeyId、AccessKeySecret 和 SecrityToken）、数据库密码、加密密钥等。</p>
<p data-nodeid="1348"><strong data-nodeid="1516">常见的错误做法是：</strong> 把这些敏感信息简单地放在项目配置文件中，随代码一起上传。这样任何能访问代码的开发者都能访问这些敏感信息。另外，有的开发者不小心把这些敏感信息上传到了公开的代码仓库中（如 Github）。我曾经目睹很多访问凭证泄露的案例，公司的机密信息直接暴露给了未经授权的用户，更严重的是，公共的搜索引擎还对敏感数据进行了索引，使每个人都能轻松获取这些数据。</p>
<ul data-nodeid="1349">
<li data-nodeid="1350">
<p data-nodeid="1351"><strong data-nodeid="1520">DDoS 和资损</strong></p>
</li>
</ul>
<p data-nodeid="1352">DDoS 几乎成了每个暴露在互联网上的服务面临的主要风险之一。虽然 Serverless 的自动弹性伸缩让你不用担心面对流量高峰时的系统扩展能力，但也给你带来了一些问题，比如大量恶意运行函数造成资损。</p>
<p data-nodeid="1353">之前有一个 aws-lambda-multipart-parser 的包中就有一个漏洞，攻击者就可以使用该漏洞对 Lambda 函数发起 DDoS 攻击，使函数一直挂起直到超时。该包中有这样一段代码：</p>
<pre class="lang-javascript" data-nodeid="1354"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports.parse = <span class="hljs-function">(<span class="hljs-params">event, spotText</span>) =&gt;</span> {
    <span class="hljs-comment">// 从 Content-Type 中获取 boundary 属性</span>
    <span class="hljs-keyword">const</span> boundary = getValueIgnoringKeyCase(event.headers, <span class="hljs-string">'Content-Type'</span>).split(<span class="hljs-string">'='</span>)[<span class="hljs-number">1</span>];
    <span class="hljs-comment">// 根据 boundary 从消息体中解析数据</span>
    <span class="hljs-keyword">const</span> body = (event.isBase64Encoded ? Buffer.from(event.body, <span class="hljs-string">'base64'</span>).toString(<span class="hljs-string">'binary'</span>) : event.body)
        .split(<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(boundary))
        .filter(<span class="hljs-function"><span class="hljs-params">item</span> =&gt;</span> item.match(<span class="hljs-regexp">/Content-Disposition/</span>))
}
</code></pre>
<p data-nodeid="1355"><strong data-nodeid="1527">这段代码逻辑很简单：</strong> 首先从 HTTP Headers 中的 Content-Type 字段里面获取 boundary 属性，然后根据&nbsp;boundary 属性来构造正则表达式，从 HTTP body 中解析数据。</p>
<p data-nodeid="1356">由于&nbsp;boundary 属性完全在调用方的控制之下，因此恶意用户就可以利用&nbsp;boundary 构造请求数据对函数发起 DDoS 攻击：</p>
<pre class="lang-js" data-nodeid="1357"><code data-language="js">POST /app HTTP/<span class="hljs-number">1.1</span>
<span class="hljs-attr">Host</span>: xxxxxxxxxxx.us-east<span class="hljs-number">-1.</span>amazonaws.com
Content-Length: <span class="hljs-number">327</span>
Content-Type: multipart/form-data; boundary=(.+)+$
<span class="hljs-attr">Connection</span>: keep-alive
(.+)+$
Content-Disposition: form-data; name=<span class="hljs-string">"text"</span>
Content
(.+)+$
Content-Disposition: form-data; name=<span class="hljs-string">"file1"</span>; filename=<span class="hljs-string">"a.txt"</span>
Content-Type: text/plain
Content <span class="hljs-keyword">of</span> a.txt.
(.+)+$
Content-Disposition: form-data; name=<span class="hljs-string">"file2"</span>; filename=<span class="hljs-string">"a.html"</span>
Content-Type: text/html
&lt;!DOCTYPE html&gt;<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Content of a.html.<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span></span>
(.+)+$
</code></pre>
<p data-nodeid="1358">当攻击者传入<code data-backticks="1" data-nodeid="1530">(.+)+$</code>这个字符串时，代码中就会使用<code data-backticks="1" data-nodeid="1532">(.+)+$</code>去构造正则表达式解析请求体，由于<code data-backticks="1" data-nodeid="1534">(.+)+$</code>的效率极低，就会导致程序一直卡在 split 方法，进而造成 CPU 使用率持续为 100%。<strong data-nodeid="1542">我在自己 3.5 GHz Intel Core i7 CPU 的电脑上测试了，该段代码运行了 10 分钟都没有结束。</strong> 将该函数放在 Lambda 上运行，函数最终会超时被 Lambda 强制结束。<br>
这时，攻击者就可以对该函数发起大量的恶意请求，从而生成大量函数实例，直到超过函数最大并发和实例数限制，并且每个函数实例都会持续运行直到超时，进而导致其他用户无法正常访问。由于函数是按实际使用时间计费，这也会增加你的费用。</p>
<ul data-nodeid="1359">
<li data-nodeid="1360">
<p data-nodeid="1361"><strong data-nodeid="1546">函数执行流程操纵</strong></p>
</li>
</ul>
<p data-nodeid="1362">通过操纵函数执行流程，攻击者可以破坏应用逻辑，并且还可以利用该方式绕过访问控制、提升用户权限、甚至发起 DDoS 攻击。</p>
<p data-nodeid="1363">执行流程操纵这种攻击方式在传统架构中也很常见，<strong data-nodeid="1553">但在 Serverless 中风险可能更大。</strong> 因为 Serverless 应用是由很多离散的函数组成，这些函数按特定顺序编排到一起，形成整体应用。在应用有多个函数、且函数间有依赖的情况下，调用顺序就尤为重要。此外，如果函数依赖他云服务来存储状态，则状态存储和共享过程也可能成为攻击目标。</p>
<p data-nodeid="1364">举个例子，假设有个应用需要对用户上传到云存储的文件进行加密，应用逻辑如下：</p>
<ol data-nodeid="1365">
<li data-nodeid="1366">
<p data-nodeid="1367">用户通过身份认证后上传文件到云存储；</p>
</li>
<li data-nodeid="1368">
<p data-nodeid="1369">函数1 校验用户上传的文件的完整性，例如检查文件大小是否为 64 KB，校验通过则向消息队列发送一个消息，消息内容是待加密的文件名；</p>
</li>
<li data-nodeid="1370">
<p data-nodeid="1371">函数2 由消息触发器触发执行，函数2 接收到消息后，从消息内容中解析出文件名称，然后对文件进行加密。</p>
</li>
</ol>
<p data-nodeid="1372"><img src="https://s0.lgstatic.com/i/image/M00/90/5A/CgqCHmAKryiAcAXoAAZWzgsF--Y088.png" alt="Drawing 4.png" data-nodeid="1560"></p>
<div data-nodeid="1373"><p style="text-align:center">对用户上传的文件进行加密的应用</p></div>
<p data-nodeid="1374">对于该应用，恶意用户就可以通过两种方式来操纵应用逻辑：</p>
<ol data-nodeid="1375">
<li data-nodeid="1376">
<p data-nodeid="1377">如果云存储没有设置合适的访问控制，任何用户都可以绕过文件完整性校验而直接上传文件，这样就会导致用户恶意上传大量文件；</p>
</li>
<li data-nodeid="1378">
<p data-nodeid="1379">如果消息队列没有设置合适的访问控制，任何用户都可以发送大量“文件已上传”的消息，使函数2 恶意执行。</p>
</li>
</ol>
<ul data-nodeid="1380">
<li data-nodeid="1381">
<p data-nodeid="1382"><strong data-nodeid="1567">异常处理不当</strong></p>
</li>
</ul>
<p data-nodeid="1383">由于 Serverless 的调试方式还比较有限，所以很多开发者喜欢直接在 FaaS 平台上打印函数运行时的日志，以及冗长的错误信息。这样在生产环境中，如果一些敏感信息没有被清除，可能导致敏感信息泄露在日志中，或日志中记录了详细的错误堆栈，暴露代码的漏洞。此外还有一些错误处理不当，导致代码被“挂起”，程序一直无法运行结束。</p>
<h3 data-nodeid="1384">总结</h3>
<p data-nodeid="1385">不管是传统应用还是 Serverless 架构的应用，都存在安全风险，因此你要先深入了解应用架构中的风险点，这样才能对症下药，解决问题。关于今天这一讲，我想要强调以下几点：</p>
<ul data-nodeid="3204">
<li data-nodeid="3205">
<p data-nodeid="3206">在云上运行的应用，云厂商负责计算、网络、存储等底层资源的安全性，应用所有者负责应用本身的安全性；</p>
</li>
<li data-nodeid="3207">
<p data-nodeid="3208">Serverless 安全性面临的主要挑战是：越来越多的攻击面、越来越复杂的攻击方式、可观测性不足以及传统安全测试方法和防护方案不适用于 Serverless 架构；</p>
</li>
<li data-nodeid="3209" class="">
<p data-nodeid="3210">对于 Serverless 架构的安全风险需要深入理解，才能更好地规避。</p>
</li>
</ul>
<p data-nodeid="3850"><img src="https://s0.lgstatic.com/i/image/M00/91/88/CgqCHmAOq2iAeo7SAAFEDNPnwwM925.png" alt="玩转 Serverless 架构11金句.png" data-nodeid="3854"></p>
<p data-nodeid="3851">总的来说，Serverless 虽然可以让我们不用关心底层计算、网络、存储等资源，但由于是一个新的架构，传统安全解决方案不适用 Serverless，在安全上就给我们带来了一些新的风险点，那么如何解决这些风险呢？我们下一讲继续探讨。</p>








<p data-nodeid="1394" class="">今天的课后作业是：除了我在这一讲中提到的 10 种风险类型，你还知道其他的风险吗？欢迎在留言区留言与我一同讨论。</p>

---

### 精选评论

##### **来嗟：
> 文中所说的风险都是假设开发者是弱智的，谁会这么弱鸡让文件服务器打开让别人乱上传文件，服了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 您好，感谢回复。在使用 Serverless 时，我们通常不会自建文件服务器，而是使用云存储，比如阿里云 OSS、AWS S3，使用云存储时，若权限设置不当或密钥泄漏，都有可能导致用户恶意上传文件。甚至正常的上传文件流程，也可能由于鉴权或限流不当，被恶意利用。举个例子，某个公司使用了 OSS 来存储生产环境的文件，公司内每个开发同学使用单独的子账号，为了方便就给所有子账号都设置了 OSS 的读写权限，并且没有将权限精确到具体的  Bucket，这样若其中某个子账号的 AK 泄漏了，就可能导致 OSS 中的用于生产环境的文件不安全。

本文的风险假设也并不是认为开发者都会犯低级错误，而是我个人踩过的一些坑的总结和分享，比如代码中敏感配置一定要加密，但实际工作中我发现很多同学为了简单，并没有这么做，所以我也提出来了。这里也可以再举个真实的例子，曾经很多企业在某 Gitlab 平台上托管代码，Gitlab 的权限有 Private、Internal 和 Public 三种权限，由于很多开发者误以为 Internal 是指 “只有企业内部员工” 才能看到，所以代码仓库都是 Internal 权限，并且代码中的密码都是明文的，最终导致了严重的数据泄漏。实际上 Private 才能组织内部可见，而 Internal 是指登录 Gitlab 的用户都可见。

虽然权限问题、密钥泄漏，包括您说的打开文件服务器，这些看起来都是很低级的问题，但我也亲生经历过由密钥泄漏等低级错误导致的重大安全事故，所以我觉得这些风险有必要跟大家强调。

