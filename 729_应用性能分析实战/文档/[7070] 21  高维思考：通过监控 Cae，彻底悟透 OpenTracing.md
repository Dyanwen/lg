<p data-nodeid="5000">你好，这一讲我将带你通过三个监控 Case，让你实现 OpenTracing 的能力更上一台阶。</p>
<p data-nodeid="5001">由于分布式追踪是基于插件扩展实现的，而绝大多数时候，插件很难在企业的应用服务集群中面面俱到，也就是不能实现 100% 的链路串联。所以很多时候在落地 APM 工具后，还是避免不了二次开发。</p>
<blockquote data-nodeid="5002">
<p data-nodeid="5003">通过前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=729#/detail/pc?id=7058&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="5094">09</a>、<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=729#/detail/pc?id=7063&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="5098">14</a>课时的学习，我们已经掌握了 OpenTracing 的基础概念，以及二次开发企业内部插件的必备能力。</p>
</blockquote>
<p data-nodeid="5004">但是很多时候现实往往与已有认知不同，这时我们就需要变通思考了。在接下来的三个 case 中，我将通过改变载体、增强语言、转化思维这三种套路，实现对特定场景的支持。</p>
<p data-nodeid="5005">改变载体：不拘泥载体，Kafka 低版本的链路串联</p>
<p data-nodeid="5006">增强语言：不拘泥语言，SQL 语句支持链路追踪</p>
<p data-nodeid="5007">转化思维：不拘泥常规，链路溯源打通网关基建</p>
<h3 data-nodeid="5008">不拘泥载体，Kafka 低版本的链路串联</h3>
<p data-nodeid="5009">第一个案例是实现 Kafka 低版本的分布式链路监控，我们知道 Kafka 消息经历了三次版本迭代。</p>
<ul data-nodeid="5010">
<li data-nodeid="5011">
<p data-nodeid="5012">v0 版本：它是 Kafka 消息格式的第一个版本，在 Kafka 0.10.0 版本之前的消息体都采用 V0 版本进行传输。v0 版本由于没有给用户扩展的可传输协议头部，所以很难无侵入实现分布式链路追踪。</p>
</li>
<li data-nodeid="5013">
<p data-nodeid="5014">v1 版本：在 Kafka 0.10.0~0.11.0 版本之间，我们将其使用的消息格式的版本称为 v1 版本。相比 v0 版本的消息格式，v1 版本增加了时间戳属性，它依然是很难实现无侵入监控的版本。</p>
</li>
<li data-nodeid="5015">
<p data-nodeid="5016">v2 版本：Kafka 0.11.0 版本后，消息格式全部默认使用 v2 版本。相比前两个版本，v2 版本的改动是最大的。最大改动之一，就是这个字段给予了 Kafka 消息体像 Http 消息体（Header属性）一样的扩展性。SkyWalking 及其他主流 APM 工具都是在 V2 版本上支持的。</p>
</li>
</ul>
<p data-nodeid="5017">难道分布式链路监控，只有在可传输的载体中才能实现吗？Kafka 低版本如何支持分布式链路监控呢？接下来讲述的，就是如何实现低版本 Kafka 分布式链路监控。</p>
<p data-nodeid="5018">以 SkyWalking 的分布式链路为例，通过<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=729#/detail/pc?id=7058&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="5115">09</a>课时，我们知道 Kafka 链路的串联是通过 SW3 属性。通过 Kafka V2 版本的 Header 属性携带分布式链路信息，实现了信息的串联打通。低版本 Kafka 消息格式，由于没有提供用于传输的消息头部，所以不支持分布式链路监控。</p>
<blockquote data-nodeid="5019">
<p data-nodeid="5020">但由于集群滚动升级有一定风险，所以国内还有很多企业的 Kafka 基建，还是在使用 V0 版本或是 V1 版本的。</p>
</blockquote>
<p data-nodeid="5021">这时，如果想要串联链路，就不能拘泥于载体。通过扩展消息的 Body 属性，存放分布式链路信息属性实现串联。</p>
<p data-nodeid="5022"><strong data-nodeid="5122">那你可能会问了：扩展 Body 属性风险很高，很容易出现前后不兼容等问题啊？</strong></p>
<p data-nodeid="5023">的确是的，如何解决这个问题呢？你可以通过扩展 SkyWalking Toolkit 工具包，让用户手动获取 SW3 属性。如果消息体是对象，那我们可以扩展对象；但是如果消息是基础类型，那此时就要先将消息从“基础类型”升级为“对象类型”。</p>
<p data-nodeid="5024">上下游兼容后再扩展消息，具体流程如下：</p>
<ol data-nodeid="5025">
<li data-nodeid="5026">
<p data-nodeid="5027">确认 Kafka 消息类型为对象类型，并扩展出用于存放 SW3 的属性；</p>
</li>
<li data-nodeid="5028">
<p data-nodeid="5029">二次开发 SkyWalking ToolKit 工具包，开放出手动埋点 Kafka 的方法；</p>
</li>
<li data-nodeid="5030">
<p data-nodeid="5031">生产者通过切面技术和打点方法，在发送消息过程中将 SW3 存储消息对象；</p>
</li>
<li data-nodeid="5032">
<p data-nodeid="5033">消费者通过切面技术和打点方法，将接收到的消息中的 SW3 与本地链路绑定，最终完成链路的串联。</p>
</li>
</ol>
<p data-nodeid="5034">通过串联 Kafka 低版本链路案例，我们可以了解到，<strong data-nodeid="5134">手动串联</strong>可以让链路串联得到更好的兼容性，用于存放分布式链路信息的载体，不必拘泥于传输的载体和无侵入的实现。</p>
<p data-nodeid="5035"><strong data-nodeid="5138">只要方案可以有很好的兼容性，就是好的方案。此思想可以适用于没有传输消息头部载体的框架。</strong></p>
<h3 data-nodeid="5036">不拘泥语言，SQL 语句支持链路追踪</h3>
<p data-nodeid="5037">这个案例，我们要实现<strong data-nodeid="5149">关系型数据库基建</strong>与<strong data-nodeid="5150">分布式链路</strong>的打通。</p>
<p data-nodeid="5038">在工作中，实现业务的工程项目是最多的，而工程项目往往都会依赖 Mysql 等关系型数据库。应用服务在与关系型数据库交互时，会使用通用的 SQL 语言操作，这些 SQL 语言有着使用方便、可用于复杂查询的特征。</p>
<p data-nodeid="5039">对于目前的 APM 监控来说，以 SkyWalking 为例，都只监控到“如何调用 Mysql 服务器的过程”（这里带引号的原因是里面还包含了网络等因素），但与Mysql服务器的基建是不打通的。</p>
<blockquote data-nodeid="5040">
<p data-nodeid="5041">举一个最浅显的例子，DBA 会定期对“慢查”进行治理，但是排查出来的只有慢查语句，无法关联出慢查现场，这导致了 SkyWalking 与数据库基建的 APM 数据割裂独立，降低排查问题效率。</p>
</blockquote>
<p data-nodeid="5042">那如何串联数据呢？我们先会看下应用服务与关系型数据库的交互过程：</p>
<ol data-nodeid="5043">
<li data-nodeid="5044">
<p data-nodeid="5045">应用服务响应用户查询请求，在 WEB 容器处开启分布式链路追踪；</p>
</li>
<li data-nodeid="5046">
<p data-nodeid="5047">应用服务的业务逻辑代码，将查询请求转化为 SQL 查询语句；</p>
</li>
<li data-nodeid="5048">
<p data-nodeid="5049">应用服务通过数据库驱动，使用 SQL 语句调用数据库；</p>
</li>
<li data-nodeid="5050">
<p data-nodeid="5051">在调用数据库过程中，应用服务和数据库服务被各自的 APM 工具监控着；</p>
</li>
<li data-nodeid="5052">
<p data-nodeid="5053">调用完成后，将数据库中数据转化为对象实体返回给前端。</p>
</li>
</ol>
<p data-nodeid="5054">不难发现，打通链路的关键点就在步骤 3，由于应用服务调用的请求只有数据库执行语言，没有任何用于传输链路信息载体。这时我们就要<strong data-nodeid="5165">增强执行语句</strong>，将链路信息通过执行语句带入数据库服务中。</p>
<p data-nodeid="5055">针对 SQL 语言的结构学习，以下面代码段为例，我们可以从以下两个方案增强 SQL 语句：</p>
<pre class="lang-sql" data-nodeid="5056"><code data-language="sql"><span class="hljs-comment">#traceId</span>
<span class="hljs-keyword">SELECT</span>
<span class="hljs-comment">/**traceId**/</span>
	<span class="hljs-string">'slow sql'</span>
<span class="hljs-keyword">FROM</span>
DUAL&nbsp;
<span class="hljs-keyword">WHERE</span>
	<span class="hljs-string">'traceId'</span> = <span class="hljs-string">'traceId'</span>
</code></pre>
<ul data-nodeid="5057">
<li data-nodeid="5058">
<p data-nodeid="5059">方案 1：如代码段中的第 1 行和第 3 行，通过在执行 SQL 中织入链路 ID 注释的方式，实现链路信息的串联。</p>
</li>
<li data-nodeid="5060">
<p data-nodeid="5061">方案 2：如代码的第 8 行，通过增加通配前缀的方式，织入链路信息，此方案借鉴 ORM 框架的通配 where 条件的动态拼接。</p>
</li>
</ul>
<p data-nodeid="5062">方案 1 可通过框架暴露出的拦截器，无侵入实现；方案 2 可以通过手动织入链路信息，让一线开发人员在可能出现问题的场景增加相关埋点。</p>
<p data-nodeid="5063">通过打通 SkyWalking 和 Mysql 监控，我们了解到 APM 基建的打通不要局限于 OpenTracing 的固有思想。因为很多基建是不支持 OpenTracing 规范的，硬要实现 ROI 非常低，而且不现实。</p>
<p data-nodeid="5064">而通过变通，我们只要在 SQL 语言中织入分布式链路 ID；在出现慢查时，Mysql 基建反映出的慢查 SQL 语句中能提分布式链路信息，让 Mysql 基建打通 SkyWalking，就轻而易举地实现了慢查场景的 SQL 语句追踪。</p>
<h3 data-nodeid="5065">不拘泥思维，链路溯源打通网关基建</h3>
<p data-nodeid="5066">在这个案例中，我们要实现 SkyWalking 与网关 NGINX 的打通。在生产环境中，故障组初步认定故障都是通过 NGINX 网关日志中的 4XX、5XX 来定义的，因为 NGINX 网关是应用服务集群中，与用户交互的第一次基建。所以通过短连接响应码，可以直观反映用户体验。</p>
<p data-nodeid="5067">可现实是，当用户请求应用服务，返回响应码是 4XX 或 5XX 时，我们能看到的异常报警信息有：用户请求的路径和请求参数，此请求引发的应用服务集群的调用过程无法得知。</p>
<p data-nodeid="5068"><strong data-nodeid="5178">你可能会问了：应用服务集群不是接入了 SkyWalking 监控工具了吗？</strong></p>
<p data-nodeid="5069">应用服务的确接入了 SkyWalking，但通过请求路径和参数，很难精确找到应用的异常请求。不仅如此，还存在着更严重的问题，就是 SkyWalking 并没有监控和存储到这一异常请求的数据。</p>
<p data-nodeid="5070"><strong data-nodeid="5183">那么什么场景会造成此问题呢？</strong></p>
<p data-nodeid="5071">其实场景也很简单，比如网络抖动，NGINX 根本转发不到目标应用服务。如在应用不可用的情况下，请求无法打到埋点，探针也就更没法收集数据了。</p>
<p data-nodeid="5072"><strong data-nodeid="5188">那如何打通数据呢？</strong></p>
<p data-nodeid="5073">如果是 NGINX 支持 SkyWalking，需要实现 SkyWalking 存储规范，且要对接 SkyWalking 的收集端，需要创造性的工作量非常大，而且有肉眼可见的性能损失。</p>
<p data-nodeid="5074">这时我们可以转换思维：通过 lua 脚本，在 NGINX 转发短连接请求时，织入一个随机唯一 ID，并修改微服务集群中的 WEB 容器组件。如 Spring MVC 在接收请求生成 OpenTracing 数据时，将 NGINX 生成的随机唯一 ID 保存到 SkyWalking 存储模型的 Tags 属性中。</p>
<p data-nodeid="5075">这样在出现异常响应码时，通过拿到 NGINX 日志中的唯一 ID；然后在 SkyWalking 的 Tag 中搜索，就可以实现链路溯源，并打通网关基建，这样对整体性能的影响就非常低。</p>
<h3 data-nodeid="5076">小结与思考</h3>
<p data-nodeid="5077">今天的课程，我带你通过三种类型的分布式监控 Case，学习了如何进行分布式链路的追踪。</p>
<ul data-nodeid="5078">
<li data-nodeid="5079">
<p data-nodeid="5080">在 Kafka 低版本的链路串联 Case 中，通过修改携带 OpenTracing 的载体，以及手动埋点的方式，实现了分布式链路追踪。</p>
</li>
<li data-nodeid="5081">
<p data-nodeid="5082">在打通 SkyWalking 与 Mysql 监控基建的 Case 中，我们可以通过 SQL 执行语句携带注释或增加通配前缀的方式，将全局链路 ID 织入进去，从而实现慢查场景。</p>
</li>
<li data-nodeid="5083">
<p data-nodeid="5084">最后是 NGINX 网关访问日志与 SkyWalking 打通的 Case，网关通过在 Header 中增加访问唯一 ID，SkyWalking 的 SpringMVC 组件识别此 ID 并绑定到资源标签中，从而实现了网关基建的打通。</p>
</li>
</ul>
<blockquote data-nodeid="5085">
<p data-nodeid="5086">打通过程不必拘泥 OpenTracing 规范思想，因为市场上有非常多没有实现 OpenTracing 思想的监控工具，我们要做的是：转换思维打通基建壁垒，提高 APM 数据价值才是重点。</p>
</blockquote>
<p data-nodeid="5087">那么你打通过什么 APM 基建吗？如何打通呢？在打通过程中有遇到什么问题吗？欢迎在评论区写下你的思考，期待与你的讨论。</p>

---

### 精选评论


