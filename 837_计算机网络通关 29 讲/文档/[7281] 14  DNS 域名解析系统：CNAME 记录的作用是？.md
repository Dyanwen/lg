<p data-nodeid="1081" class="">当你在浏览器中输入一个 URL，或者用<code data-backticks="1" data-nodeid="1184">curl</code>请求一个网址……域名系统（Domain Name System）就开始工作了。作为互联网的一个重要成员，域名系统是将互联网资源和地址关联起来的一个分布式数据库。</p>
<p data-nodeid="1082">在日常工作中，作为一名研发工程师，经常需要配置 DNS 域名解析。特别是 CNAME，几乎我每年都会碰到需要配置它的场景。这次我们就以“CNAME 记录的作用”为引，开启今天的学习，将域名这块的相关知识一网打尽。</p>
<h3 data-nodeid="1083">DNS 和统一资源你定位符（URL）</h3>
<p data-nodeid="1084"><strong data-nodeid="1196">域名系统本质是定位资源</strong>。互联网中有各种各样的资源，比如视频、图片、文件、网页……为了准确地定位资源，人们发明了统一<strong data-nodeid="1197">资源定位符</strong>（URL，Uniform Resource Locator），这样我们就可以通过字符串定位一个互联网的资源。</p>
<p data-nodeid="1085">下图是一个 URL 的示例：</p>
<p data-nodeid="1086"><img src="https://s0.lgstatic.com/i/image6/M01/40/72/Cgp9HWCk6Z-ASLtEAAH_Ssm7xjk737.png" alt="Drawing 1.png" data-nodeid="1201"></p>
<ul data-nodeid="1087">
<li data-nodeid="1088">
<p data-nodeid="1089">Scheme 部分代表协议，不只有 https，还有 ftp、ssh 等。不同协议代表着不同类型的应用在提供资源。</p>
</li>
<li data-nodeid="1090">
<p data-nodeid="1091">Host 部分代表站点，我们今天介绍的 DNS 主要作用就是根据 Host 查找 IP 地址。</p>
</li>
<li data-nodeid="1092">
<p data-nodeid="1093">Port 是端口，代表提供服务的应用。</p>
</li>
<li data-nodeid="1094">
<p data-nodeid="1095">Path 是路径，代表资源在服务中的路径。</p>
</li>
<li data-nodeid="1096">
<p data-nodeid="1097">Query 是查询条件，代表需要的是资源中的某一个部分。</p>
</li>
<li data-nodeid="1098">
<p data-nodeid="1099">Fragment 是二级查询条件，通常不在服务端响应，而是用于前端展示定位内容。</p>
</li>
</ul>
<p data-nodeid="1100">总的来说，URL 是一种树状的设计， Host 代表主机（对应的 IP 地址由 DNS 服务提供）；Port 代表应用；Path 代表资源在应用中的路径；Query 代表对资源的查询条件。通过这种设计，互联网中万亿级别的资源都可以得到有效区分。</p>
<p data-nodeid="1101">值得一提的是，树状的设计在今天计算机中也非常常见，比如文件目录的设计、源代码块的嵌套设计、JSON 和 XML 的设计，都是树状关系。这源于人类的思考方式天然地喜欢把事物放到互斥的分类当中。</p>
<p data-nodeid="1102">不过需要注意的是，<strong data-nodeid="1215">树状的分类解决不了一个东西在多个类别的情况，而这种情况在多数时候却是真实存在的</strong>。真实世界中事物是普遍联系的，所以本质上事物之间的联系应该是图。但是通常情况下，我们会用树处理某一个方面的诉求。比如用 URL 描述资源的位置，然后用搜索引擎通过关键字反查 URL（另一个方面，维度等）。</p>
<h3 data-nodeid="1103">域名系统</h3>
<p data-nodeid="1104"><strong data-nodeid="1221">DNS（Domain Name System，域名系统）是一个将域名和 IP 地址相互映射的分布式服务</strong>。比如说你想知道 lagou.com 的 IP 地址，就需要通过 DNS 服务获得。这样凡是访问 lagou 的用户，就不需要在浏览器中输入 lagou 的 IP 地址，而是通过一个方便人们记忆的域名。</p>
<h4 data-nodeid="1105">根域名服务器</h4>
<p data-nodeid="1106">DNS 本身是一个出色的分布式架构。</p>
<p data-nodeid="1107">位于最顶层的是根域名服务器（Root Name Server）。人们在全世界范围内搭建了多台根域名服务器，2016 年的统计数据中，全世界目前有 13 台 IPv4 根服务器，25 台 IPv6 根服务器。</p>
<p data-nodeid="1108">根域名服务器存储的不是域名和 IP 的映射关系，而是一个目录。如果将所有的域名记录都存放到根域名服务器，从存储量上来说，不会非常巨大。要知道一个域名记录——域名、IP 地址和额外少量信息，并不需要大量存储空间。但是如果全世界所有的 DNS 请求都集中在少量的根服务器上，这个访问流量就会过于巨大。而且一旦发生故障，很容易导致大面积瘫痪。而且因为根服务器较少，所以如果全部都走根服务器，不同客户端距离根服务器距离不同，感受到的延迟也不一样，这样对用户来说不太友好。</p>
<p data-nodeid="1109"><strong data-nodeid="1230">因此，因为流量、防止单点故障、平衡地理分布等问题，根域名服务器只是一个目录，并不提供具体的数据</strong>。</p>
<h4 data-nodeid="1110">域名分级和数据分区</h4>
<p data-nodeid="1111">我们知道中文字典可以按照偏旁部首以及拼音索引，和字典类似，根服务器提供的目录也有一定的索引规则。在域名的世界中，通过分级域名的策略建立索引。伴随着域名的分级策略，实际上是域名数据库的拆分。通过域名的分级，可以将数据库划分成一个个区域。</p>
<p data-nodeid="1112">平时我们看到的<code data-backticks="1" data-nodeid="1234">.com``.cn``.net</code>等，称为顶级域名。比如对于 www.laogu.com 这个网址来说，<code data-backticks="1" data-nodeid="1236">com</code>是顶级域名，<code data-backticks="1" data-nodeid="1238">lagou</code>是二级域名，<code data-backticks="1" data-nodeid="1240">www</code>是三级域名。域名分级当然是为了建立目录和索引，并对数据存储进行分区。</p>
<p data-nodeid="1113"><img src="https://s0.lgstatic.com/i/image6/M00/40/7A/CioPOWCk6auABUkFAAKhjoCbm9k527.png" alt="Drawing 3.png" data-nodeid="1244"></p>
<p data-nodeid="1114">从上图中可以看到，DNS 的存储设计是一个树状结构。叶子节点中才存放真实的映射关系，中间节点都是目录。存储分成 3 层：</p>
<ul data-nodeid="1115">
<li data-nodeid="1116">
<p data-nodeid="1117">顶部第一级是根 DNS 存储，存储的是顶级域的目录，被称作<strong data-nodeid="1251">根 DNS 服务器</strong>；</p>
</li>
<li data-nodeid="1118">
<p data-nodeid="1119">第二级是顶级域存储，存储的是二级域的目录，被称作<strong data-nodeid="1257">顶级域 DNS 服务器（Top Level DNS，TLD）</strong>；</p>
</li>
<li data-nodeid="1120">
<p data-nodeid="1121">最后一级是叶子节点，存储的是具体的 DNS 记录，也被称作<strong data-nodeid="1263">权威 DNS 服务器</strong>。</p>
</li>
</ul>
<h3 data-nodeid="1122">DNS 查询过程</h3>
<p data-nodeid="1123">当用户在浏览器中输入一个网址，就会触发 DNS 查询。这个时候在上述的 3 个层级中，还会增加<strong data-nodeid="1270">本地 DNS 服务器</strong>层级。本地 DNS 服务器包括用户自己路由器中的 DNS 缓存、小区的 DNS 服务器、ISP 的 DNS 服务器等。</p>
<p data-nodeid="1124">查询过程如下图所示：</p>
<p data-nodeid="1125"><img src="https://s0.lgstatic.com/i/image6/M01/40/72/Cgp9HWCk6bGABepRAAHGRO0l88o350.png" alt="Drawing 5.png" data-nodeid="1274"></p>
<p data-nodeid="1126">结合上图展示的DNS 查询过程，我们再来具体介绍一下 。</p>
<ol data-nodeid="1127">
<li data-nodeid="1128">
<p data-nodeid="1129">用户输入网址，查询本地 DNS。本地 DNS 是一系列 DNS 的合集，比如 ISP 提供的 DNS、公司网络提供的 DNS。本地 DNS 是一个代理，将 DNS 请求转发到 DNS 网络中。如果本地 DNS 中已经存在需要的记录，也就是本地 DNS 缓存中找到了对应的 DNS 条目，就会直接返回，而跳过之后的步骤。</p>
</li>
<li data-nodeid="1130">
<p data-nodeid="1131">客户端请求根 DNS 服务器。如果本地 DNS 中没有对应的记录，那么请求会被转发到根 DNS 服务器。根 DNS 服务器只解析顶级域，以“<a href="http://www.lagou.com%60?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1280">www.lagou.com</a>”为例，根 DNS 服务器只看 com 部分。</p>
</li>
<li data-nodeid="1132">
<p data-nodeid="1133">根 DNS 服务器返回顶级 DNS 服务器的 IP。</p>
</li>
<li data-nodeid="1134">
<p data-nodeid="1135">客户端请求顶级 DNS 服务器，顶级 DNS 服务器中是具体域名的目录。</p>
</li>
<li data-nodeid="1136">
<p data-nodeid="1137">顶级 DNS 服务器返回权威 DNS 服务器的 IP。</p>
</li>
<li data-nodeid="1138">
<p data-nodeid="1139">客户端请求权威 DNS 服务器。在权威 DNS 服务器上存有具体的 DNS 记录。以 lagou 为例，权威 DNS 服务器中可能有和 lagou.com 相关的上百条甚至更多的 DNS 记录，会根据不同的 DNS 查询条件返回。</p>
</li>
<li data-nodeid="1140">
<p data-nodeid="1141">权威 DNS 服务器返回 DNS 记录到本地 DNS 服务器。</p>
</li>
<li data-nodeid="1142">
<p data-nodeid="1143">本地 DNS 服务器返回具体的 DNS 记录给客户端。</p>
</li>
</ol>
<p data-nodeid="1144">在上述 8 个过程全部结束后，客户端通过 DNS 记录中的 IP 地址，可以找到请求服务的主机。在本文的例子中，客户端最终可以找到拉勾网对应的 IP 地址，从而获得 Web 服务。</p>
<h4 data-nodeid="1145">关于缓存</h4>
<p data-nodeid="1146">在上面的例子当中，每一步都有缓存的设计。浏览器会缓存 DNS，此外，操作系统、路由器、本地 DNS 服务器也会……因此，绝大多数情况，请求不会到达根 DNS 服务器。</p>
<p data-nodeid="1147">以拉勾为例，如果在某个时刻同一个区域内有一个用户触发过上述 1~8 的过程，另一个同区域的用户就可以在本地 DNS 服务器中获得 DNS 记录，而不需要再走到根 DNS 服务器。这种设计，我们称作<strong data-nodeid="1298">分级缓存策略</strong>。</p>
<p data-nodeid="1148">在分级缓存策略中，每一层都会进行缓存，经过一层层的缓存，最终命中根 DNS 服务、顶级 DNS 服务器以及权威 DNS 服务的请求少之又少。这样，互联网中庞大流量的 DNS 查询就不需要大量集中的资源去响应。</p>
<h3 data-nodeid="1149">DNS 记录</h3>
<p data-nodeid="1150">学到这里，我们来看看一个 DNS 记录具体长什么样子：</p>
<pre class="lang-java" data-nodeid="1151"><code data-language="java">; 定义www.example.com的ip地址
www.example.com.&nbsp;&nbsp;&nbsp;&nbsp; IN&nbsp;&nbsp;&nbsp;&nbsp; A&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">139.18</span><span class="hljs-number">.28</span><span class="hljs-number">.5</span>;
</code></pre>
<p data-nodeid="1152">上面的就是一条 DNS 记录，纯文本即可。IN 代表记录用于互联网，是 Intenet 的缩写。在历史上 Internet 起源于阿帕网，在同时代有很多竞争的网络，IN 这个描述也就保留了下来。</p>
<p data-nodeid="1153"><a href="http://www.example.com%E6%98%AF%E8%A6%81%E8%A7%A3%E6%9E%90%E7%9A%84%E5%9F%9F%E5%90%8D%E3%80%82?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1305">www.example.com 是要解析的域名。</a>A 是记录的类型，A 记录代表着这是一条用于解析 IPv4 地址的记录。从这条记录可知，<a href="http://www.example.comw?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1309">www.example.com</a>的 IP 地址是 139.18.28.5。<code data-backticks="1" data-nodeid="1311">;</code>是语句块的结尾，也是注释。</p>
<p data-nodeid="1154">那么除了 A 记录，还有哪些 DNS 记录的类型呢？DNS 记录的类型非常多，有 30 多种。其中比较常见的有 A、AAAA、CNAME、MX，以及 NS 等。接下来我为你一个个介绍。</p>
<h4 data-nodeid="1155">CNAME</h4>
<p data-nodeid="1156">CNAME（Canonical Name Record）用于定义域名的别名，如下面这条 DNS 记录：</p>
<pre class="lang-java" data-nodeid="1157"><code data-language="java">; 定义www.example.com的别名
a.example.com.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IN&nbsp;&nbsp;&nbsp;&nbsp; CNAME&nbsp;&nbsp; b.example.com.
</code></pre>
<p data-nodeid="1158">这条 DNS 记录定义了 a.example.com 是 b.example.com 的别名。用户在浏览器中输入 a.example.com 时候，通过 DNS 查询会知道 a.example.com 是 b.example.com 的别名，因此需要实际 IP 的时候，会去拿 b.example.com 的 A 记录。</p>
<p data-nodeid="1159">这样用户如果在浏览器中输入 a.example.com 实际打开的就是 b.example.com。因为走的是 DNS 查询的路径，速度很快（因为有缓存），不需要 HTTP 重定向等操作。</p>
<p data-nodeid="1351" class="te-preview-highlight">当你想把一个网站迁移到新域名，旧域名仍然保留的时候；还有当你想将自己的静态资源放到 CDN 上的时候，CNAME 就非常有用。具体 CDN 的操作，我们会在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7282" data-nodeid="1357">15 | 内容分发网络 ：请简述 CDN 回源如何工作？</a>”中详细讲解。</p>

<h4 data-nodeid="1161">AAAA 记录</h4>
<p data-nodeid="1162">前面我们提到，A 记录是域名和 IPv4 地址的映射关系。和 A 记录类似，AAAA 记录则是域名和 IPv6 地址的映射关系。</p>
<h4 data-nodeid="1163">MX 记录（Mail Exchanger Record）</h4>
<p data-nodeid="1164">MX 记录是邮件记录，用来描述邮件服务器的域名。</p>
<p data-nodeid="1165">在工作中，我们经常会发邮件到某个同事的邮箱。比如说，发送一封邮件到 xiaoming@lagou.com，那么拉勾网如何知道哪个 IP 地址是邮件服务器呢？</p>
<p data-nodeid="1166">这个时候就可以用到下面这条 MX 记录：</p>
<pre class="lang-java" data-nodeid="1167"><code data-language="java">IN MX mail.lagou.com
</code></pre>
<p data-nodeid="1168">这样凡是 @lagou 的邮件都会发送到 mail.lagou.com 中，而 mail.lagou.com 的 IP 地址可以通过查询 mail.lagou.com 的 A 记录和 AAAA 记录获得。</p>
<h4 data-nodeid="1169">NS 记录</h4>
<p data-nodeid="1170">NS（Name Server）记录是描述 DNS 服务器网址。从 DNS 的存储结构上说，Name Server 中含有权威 DNS 服务的目录。也就是说，NS 记录指定哪台 Server 是回答 DNS 查询的权威域名服务器。</p>
<p data-nodeid="1171">当一个 DNS 查询看到 NS 记录的时候，会再去 NS 记录配置的 DNS 服务器查询，得到最终的记录。如下面这个例子：</p>
<pre class="lang-java" data-nodeid="1172"><code data-language="java">a.com.&nbsp;&nbsp;&nbsp;&nbsp; IN&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ns1.a.com.
a.com.&nbsp;&nbsp;&nbsp;&nbsp; IN&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ns2.a.com.
</code></pre>
<p data-nodeid="1173">当解析 a.com 地址时，我们看到 a.com 有两个 NS 记录，所以确定最终 a.com 的记录在 ns1.a.com 和 ns2.a.com 上。从设计上看，ns1 和 ns2 是网站 a.com 提供的智能 DNS 服务器，可以提供负载均衡、分布式 Sharding 等服务。比如当一个北京的用户想要访问 a.com 的时候，ns1 看到这是一个北京的 IP 就返回一个离北京最近的机房 IP。</p>
<p data-nodeid="1174">上面代码中 a.com 配置了两个 NS 记录。通常 NS 不会只有一个，这是为了保证高可用，一个挂了另一个还能继续服务。通常数字小的 NS 记录优先级更高，也就是 ns1 会优先于 ns2 响应。</p>
<p data-nodeid="1175">配置了上面的 NS 记录后，如果还配置了 a.com 的 A 记录，那么这个 A 记录会被 NS 记录覆盖。</p>
<h3 data-nodeid="1176">总结</h3>
<p data-nodeid="1177">总结一下，用树状结构来分类和索引符合人类的直觉和习惯，URL 的设计遵循的依然是人的思考方式。URL 中的 HOST 部分需要被解析为 IP 地址，于是就有了域名系统（DNS）。域名系统是一个分级的分布式系统，整体设计也是一个树状结构。顶层的根域名服务器和中间的顶级域名服务器，存储的是目录，最终的 DNS 记录由权威域名服务器提供。DNS 记录并不仅仅只有映射 IP 一种能力，DNS 记录还可以设置网站的别名、邮件服务器、DNS 记录位置等能力。</p>
<p data-nodeid="1178"><strong data-nodeid="1339">那么，通过这一讲的学习，你可以尝试来回答本讲关联的面试题目：CNAME 记录的作用是？</strong></p>
<p data-nodeid="1179">【<strong data-nodeid="1345">解析</strong>】CNAME 是一种 DNS 记录，它的作用是将一个域名映射到另一个域名。域名解析的时候，如果看到 CNAME 记录，则会从映射目标重新开始查询。</p>
<h3 data-nodeid="1180">思考题</h3>
<p data-nodeid="1181">最后我再给你出一道需要查资料的思考题：DNS 工作在互联网协议群的哪一层？</p>
<p data-nodeid="1182" class="">这一讲就到这里，发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲我们将学习《15 | 内容分发网络 ：请简述 CDN 回源如何工作？》，再见！</p>

---

### 精选评论

##### **男：
> DNS工作在应用层

##### *超：
> 肯定在ip上面，不然没法找到ip😂😂😂

##### *纯：
> DNS 工作在互联网协议群的应用层。DNS 实际就是一种资源服务器只是不像 http 协议资源对应的是 文本 图片 或者音视屏. DNS 需要获得的资源是一段 域名 所映射实际 ip 地址的信息. DNS 整体的结构其实跟我们 web 服务器架构几乎一样.

##### *浩：
> 老师牛皮

