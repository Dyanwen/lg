<p data-nodeid="605" class="">今天使用的电商、直播、社交工具、视频网站中都含有大量的图片、视频、文档等，这些资源需要分发给用户。对于一些体量较大的应用来说，如果把大量资源集中到单一节点进行分发，恐怕很难有某个机房可以支撑得住这么大的流量。例如一个日活在 100W 的小型互联网产品，如果每次请求需要 1M 的数据，那就刚好是近 1TB 数据。对于这样的数据规模而言，完全由单一节点进行分发是不现实的。因此现在互联网应用在分发内容的时候，并不是从自己架设的服务器上直分发内容，而是走一个叫作<strong data-nodeid="646">内容分发网络</strong>（Content Dilivery Network）的互联网底层建设。</p>
<p data-nodeid="606">这一讲，我们就以“<strong data-nodeid="652">CDN 回源是如何工作的</strong>”为引，开启今天的学习，和你一起探索 CDN 的原理和场景。</p>
<h3 data-nodeid="607">CDN 是什么?</h3>
<p data-nodeid="608"><strong data-nodeid="662">和域名系统类似，内容分发网络（Content Dilivery Network，CDN）是一个专门用来分发内容的分布式应用</strong>。CDN 构建在现有的互联网之上，通过在各地部署数据中心，让不同地域的用户可以就近获取内容。这里的内容通常指的是文件、图片、视频、声音、应用程序安装包等，它们具有一个显著的特征——<strong data-nodeid="663">无状态，或者说是静态的</strong>。这些资源不像订单数据、库存数据等，它们一旦发布，就很少会发生变化。另一个显著的特征，是这些资源往往会被大量的用户需要，因此分发它们的流量成本是较高的。</p>
<p data-nodeid="609"><strong data-nodeid="668">为什么不能集中提供这些静态资源呢</strong>？这和域名系统的 DNS 记录不能集中提供是一个道理，需要考虑到流量、单点故障、延迟等因素。在离用户更近的地理位置提供资源，可以减少延迟。按照地理位置分散地提供资源，也可以降低中心化带来的服务压力。</p>
<p data-nodeid="610">因此，CDN 的服务商会选择在全球布点，或者在某个国家布点。具体要看 CDN 服务提供商的服务范围。目前国内的阿里云、腾讯云等也在提供 CDN 业务。</p>
<h3 data-nodeid="611">内容的分发</h3>
<p data-nodeid="612">CDN 是一个分布式的内容分发网络。当用户请求一个网络资源时，用户请求的是 CDN 提供的资源。和域名系统类似，当用户请求一个资源时，首先会接触到一个类似域名系统中目录的服务，这个服务会告诉用户究竟去哪个 IP 获取这个资源。</p>
<p data-nodeid="613"><strong data-nodeid="682">事实上，很多大型的应用，会把 DNS 解析作为一种负载均衡的手段</strong>。当用户请求一个网址的时候，会从该网站提供的智能 DNS 中获取网站的 IP。例如当你请求拉勾的时候，具体连接到哪个拉勾的 IP，是由拉勾使用的智能 DNS 服务决定的。域名系统允许网站自己为自己的产品提供 DNS 解析，具体可以参考<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7281&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="680">《14 | DNS 域名解析系统：CNAME 记录的作用是？》</a>介绍的 ns 记录。</p>
<p data-nodeid="614">所以总体静态资源的使用路径如下图所示：</p>
<p data-nodeid="756" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/41/EE/CioPOWCvAPCAKWMAAAM_tmZAhpc658.png" alt="图片1.png" data-nodeid="759"></p>

<p data-nodeid="616">当用户请求一个静态资源的时候，首先会触发域名系统的解析。域名系统会将解析的责任交由 CDN 提供商来处理，CDN 的智能 DNS 服务会帮助用户选择离自己距离最近的节点，返回这个节点的 A（或 AAAA）记录。然后客户端会向 CDN 的资源节点发起请求，最终获得资源。</p>
<p data-nodeid="617">在上面整个过程当中，CDN 的智能 DNS 还充当了负载均衡的作用。如果一个节点压力过大，则可以将流量导向其他的节点。</p>
<h3 data-nodeid="618">回源</h3>
<p data-nodeid="619">目前我们已经讨论了 CDN 的主要设计和架构，但是还有一个问题没有解决——就是资源怎么进入内容分发网络。资源的生产者，也是 CDN 的购买者，目的是向用户提供网络服务。那么服务提供者的静态资源如何进入 CDN 呢？ 手动上传、用接口推送，还是通过其他别的方式呢？</p>
<p data-nodeid="620">你可以把 CDN 想象成一个分布式的分级缓存，再加上数据库的两层设计，如下图所示：</p>
<p data-nodeid="621"><img src="https://s0.lgstatic.com/i/image6/M00/41/91/CioPOWCsuKeAIBZBAAGS1a5eHTk676.png" alt="Drawing 3.png" data-nodeid="694"></p>
<p data-nodeid="622">用户的请求先到达缓存层，如果缓存被穿透，才到达最终的存储层。缓存的设计必须是分布式的，因为绝大多数的资源使用都会发生在缓存上，只有极少数的请求才会穿透到底层的存储。通常这种设计，我们期望缓存层至少需要帮挡住 99% 的流量。既然缓存层能挡住 99% 的流量，那么实际的数据存储就可以交由源站点完成。</p>
<p data-nodeid="623">值得一提的是，在程序设计当中有一个核心的原则，叫作<strong data-nodeid="705">单一数据源（Single Souce of Truth， SSOT）</strong>。<strong data-nodeid="706">这个原则指的是，在程序设计中，应该尽可能地减少数据的来源，最好每个数据来源只有单独一份</strong>。这样能够避免大量的数据不一致以及同步数据的问题。基于这样的设计，谁来提供资源的存储呢？谁来提供这个单一的数据源呢？当然是服务提供者本身。如果 CDN 再提供 一份资源的存储，不就有两个数据源了吗？而且，只有服务的提供者才能更好地维护这个资源仓库。</p>
<p data-nodeid="624">在 CDN 的设计当中，CDN 实际上提供的是数据的缓存。而原始数据，则由服务的提供者提供。举个例子，当用户请求一张拉勾网上的图片，看上去这张图片的网址就是拉勾的一个网址：<a href="http://s0.lgstatic.com?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="710">s0.lgstatic.com</a>。而实际上，如果你用 DIG 命令去查看这个网址，会看到如下图所示的结果：</p>
<p data-nodeid="625"><img src="https://s0.lgstatic.com/i/image6/M01/41/89/Cgp9HWCsuK6AFxN_AAUoFziC3xU348.png" alt="Drawing 4.png" data-nodeid="714"></p>
<p data-nodeid="626">上面的结果中，拉勾网的静态资源域名<a href="http://s0.lgstatic.com/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="718">s0.lgstatic.com</a>被 CNAME 到了<a href="http://s0.lgstatic.com.wswebpic.com%E3%80%82%E8%BF%99%E8%AF%B4%E6%98%8E%E5%BD%93%E7%94%A8%E6%88%B7%E5%9C%A8%E8%AF%B7%E6%B1%82s0.lgstatic.com%EF%BC%88%E4%B8%80%E4%B8%AA%E6%8B%89%E5%8B%BE%E5%9F%9F%E5%90%8D%EF%BC%89%E7%9A%84%E8%B5%84%E6%BA%90%E6%97%B6%EF%BC%8C%E5%AE%9E%E9%99%85%E8%AF%B7%E6%B1%82%E7%9A%84CDN%E6%9C%8D%E5%8A%A1%E6%8F%90%E4%BE%9B%E5%95%86%E7%9A%84%E5%9F%9F%E5%90%8D%E3%80%82?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="722">s0.lgstatic.com.wswebpic.com</a>。这说明当用户在请求 s0.lgstatic.com（一个拉勾域名）的资源时，实际请求的 CDN 服务提供商的域名。当用户向 CDN 请求资源的时候，CDN 的智能 DNS 服务就会帮助用户选最优的节点（比如地理上最临近的，或者当前较空闲的）。如果 CDN 资源节点中已经存在了用户拥有的资源，那么就直接返回资源给用户。如果 CDN 中尚未缓存这个资源， 此时 CDN 节点就会向拉勾请求资源。也就是说，拉勾网需要有所有的原始数据，并提供出来可以让 CDN 服务访问。</p>
<p data-nodeid="627">如下图所示，整个过程是 4 个层级。用户请求静态资源通常用自己的域名（防止跨域和一些安全问题）。为了让用户请求的是自己的网站，而使用的是 CDN 的服务，这里会使用 CNAME 让自己的域名作为 CDN 域名的一个别名。当请求到 CDN 服务的时候，会首先由 CDN 的 DNS 服务帮助用户选择一个最优的节点，这个 DNS 服务还充当了负载均衡的作用。接下来，用户开始向 CDN 节点请求资源。如果这个时候资源已经过期或者还没有在 CDN 节点上，就会从源站读取数据，这个步骤称为<strong data-nodeid="729">CDN 的回源</strong>。</p>
<p data-nodeid="628"><img src="https://s0.lgstatic.com/i/image6/M01/41/89/Cgp9HWCsuLeABoBAAAJfmJcMOc0952.png" alt="Drawing 6.png" data-nodeid="732"></p>
<p data-nodeid="629">另一方面，CDN 上缓存的资源通常也会伴随失效时间的设置，当失效之后同样会触发回源。另一种情况是可以通过开放的 API 或者 CDN 管理后台直接删除缓存（让资源失效），这个操作结束后，同样会触发回源。</p>
<h3 data-nodeid="630">总结</h3>
<p data-nodeid="631">总结一下，CDN 是一种网络应用，作用是分发互联网上的资源。CDN 服务的提供商，会在世界（或国家）范围内设立数据中心，帮助分发资源。用户请求的资源会被 CDN 分发到最临近的节点获取。</p>
<p data-nodeid="632">CDN 作为一门生意，CDN 的服务商会大批量的从运营商处获取流量，然后再以较高但是可以接受的价格卖给服务提供方。对于中小型互联网公司来说，购买一定的 CDN 流量成本可控，比如 1G 流量在 1 元以内。对于大型的互联网公司，特别是对 CDN 依赖严重的公司，可能还需要自己建设。比如 2021 年抖音每天分发的数据量在 50PB 左右（1PB=1024TB），如此庞大的数据量如果换算成钱是非常高的。按照阿里云的报价，50PB 的价格是 480W 人民币。按照这种体量计算，抖音每天要花 480W 人民币，一年是 17 亿。</p>
<p data-nodeid="633">所以当你设计一个内容分发的方案时，除了要考虑到其中的技术细节，也要从成本上进行思考，看看能不能从数据压缩、资源格式角度做一些文章。之前我参与的一个项目就考虑将图片从 jpg 格式替换为 webp 格式，一年节省了 500W 元的 CDN 费用。</p>
<p data-nodeid="634"><strong data-nodeid="742">那么现在你可以尝试来回答本讲关联的面试题目：请简述 CDN 回源是如何工作的</strong>？</p>
<p data-nodeid="635">【<strong data-nodeid="748">解析</strong>】CDN 回源就是 CDN 节点到源站请求资源，重新设置缓存。通常服务提供方在使用 CDN 的时候，会在自己的某个域名发布静态资源，然后将这个域名交给 CDN。</p>
<p data-nodeid="636">比如源站在 s.example.com 中发布静态资源，然后在 CDN 管理后台配置了这个源站。在使用 CDN 时，服务提供方会使用另一个域名，比如说 b.example.com。然后配置将 b.example.com 用 CNAME 记录指向 CDN 的智能 DNS。这个时候，如果用户下载b.example.com/a.jpg，CDN 的智能 DNS 会帮用户选择一个最优的 IP 地址（最优的 CDN 节点）响应这次资源的请求。如果这个 CDN 节点没有 a.jpg，CDN 就会到 s.example.com 源站去下载，缓存到 CDN 节点，然后再返回给用户。</p>
<p data-nodeid="637">CDN 回源有 3 种情况，一种是 CDN 节点没有对应资源时主动到源站获取资源；另一种是缓存失效后，CDN 节点到源站获取资源；还有一种情况是在 CDN 管理后台或者使用开放接口主动刷新触发回源。</p>
<h3 data-nodeid="638">思考题</h3>
<p data-nodeid="639">最后再给你出一道需要查资料的思考题：如果你的应用需要智能 DNS 服务，你将如何实现？</p>
<p data-nodeid="640" class="">这一讲就到这里，发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲我们将学习《16 | HTTP 协议面试通关 ：强制缓存和协商缓存的区别是？》，再见！</p>

---

### 精选评论

##### **超：
> 一遍看不懂,就在看一遍

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油！

##### **帆：
> 我已经是二刷了，每次都有新收获。

