<p data-nodeid="1121" class="">广域网是由很多的局域网组成的，比如公司网络、家庭网络、校园网络等。之前我们一直在讨论广域网的设计，今天我们到微观层面，看看局域网是如何工作的。</p>
<p data-nodeid="1122">IPv4 的地址不够，因此需要设计子网。当一个公司申请得到一个公网 IP 后，会在自己的公司内部设计一个局域网。这个局域网所有设备的 IP 地址，通常会以 192.168 开头。这个时候，假设你的职工小明，上班时间玩王者荣耀。当他用 UDP 协议向王者荣耀的服务器发送信息时，消息的源 IP 地址是一个内网 IP 地址，而王者荣耀的服务，是一个外网 IP 地址。</p>
<p data-nodeid="1123"><strong data-nodeid="1205">这里我先向你提一个问题，数据到王者荣耀服务器可以通过寻址和路由找到目的地，但是数据从王者荣耀服务器回来的时候，王者荣耀服务器如何知道</strong><code data-backticks="1" data-nodeid="1203">192.168</code>开头的地址应该如何寻址呢？</p>
<p data-nodeid="1124">要想回答这个问题，就涉及网络地址转换协议（NAT 协议）。下面请你带着这个问题，开启今天的学习吧。</p>
<h3 data-nodeid="1125">内部网络和外部网络</h3>
<p data-nodeid="1126">对一个组织、机构、家庭来说，我们通常把内部网络称为局域网，外部网络就叫作外网。下图是一个公司多个部门的网络架构。</p>
<p data-nodeid="1400" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/39/Cgp9HWCJIMaACsQqAAKt__fEo9U561.png" alt="图片11.png" data-nodeid="1403"></p>

<p data-nodeid="1128">我们会看到外网通过路由器接入整个公司的局域网，和路由器关联的是三台交换机，代表公司的三个部门。<strong data-nodeid="1217">交换机，或者称为链路层交换机，通常工作在链路层；而路由器通常也具有交换机的能力，工作在网络层和链路层</strong>。关于它们的详细区别，我们会在本文的后续讨论。</p>
<p data-nodeid="1129">光纤是一种透明的导光介质，多束光可以在一个介质中并行传播，不仅信号容量大，重量轻，并行度高而且传播距离远。当然，光纤不能弯曲，因此办公室里用来连接交换机和个人电脑的线路肯定不能是光纤，<strong data-nodeid="1223">光线通常都用于主干网络</strong>。</p>
<h3 data-nodeid="1130">局域网数据交换（MAC 地址）</h3>
<p data-nodeid="1131">接下来我们讨论下同一个局域网中的设备如何交换消息。</p>
<p data-nodeid="1132">首先，我们先明确一个概念，设备间通信的本质其实是设备拥有的网络接口（网卡）间的通信。<strong data-nodeid="1231">为了区别每个网络接口，互联网工程任务组（IETF）要求每个设备拥有一个唯一的编号，这个就是 MAC 地址</strong>。</p>
<p data-nodeid="1133"><strong data-nodeid="1236">你可能会问：IP 地址不也是唯一的吗</strong>？其实不然，一旦设备更换位置，比如你把你的电脑从北京邮寄的广州，那么 IP 地址就变了，而电脑网卡的 MAC 地址不会发生变化。总的来说，IP 地址更像现实生活中的地址，而 MAC 地址更像你的身份证号。</p>
<p data-nodeid="1134">然后，我们再明确另一个基本的概念。<strong data-nodeid="1242">在一个局域网中，我们不可以将消息从一个接口（网卡）发送到另一个接口（网卡），而是要通过交换机</strong>。为什么是这样呢？因为两个网卡间没有线啊！所以数据交换，必须经过交换机，毕竟线路都是由网卡连接交换机的。</p>
<p data-nodeid="1962" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3C/42/CioPOWCJINCAWTthAACzGJa966I160.png" alt="图片2.png" data-nodeid="1965"></p>

<p data-nodeid="1136">总结下，数据的发送方，将自己的 MAC 地址、目的地 MAC 地址，以及数据作为一个分组（Packet），也称作 Frame 或者封包，发送给交换机。交换机再根据目的地 MAC 地址，将数据转发到目的地的网络接口（网卡）。</p>
<p data-nodeid="1137"><strong data-nodeid="1255">最后一个问题，你可能问，这个分组或者 Frame，是不是 IP 协议的分组呢</strong>？——不是，这里提到的是链路层的数据交换，它支持 IP 协议工作，是网络层的底层。所以，<strong data-nodeid="1256">如果 IP 协议要传输数据，就要将数据转换成为链路层的分组，然后才可以在链路层传输</strong>。</p>
<p data-nodeid="1138">链路层分组大小受限于链路层的网络设备、线路以及使用了链路层协议的设计。你有时候可能会看到 MTU 这个缩写词，它指的是 Maximun Transmission Unit，最大传输单元，意思是链路层网络允许的最大传输数据分组的大小。<strong data-nodeid="1262">因此 IP 协议要根据 MTU 拆分封包</strong>。</p>
<p data-nodeid="1139">之前在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7268&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1268">04 | TCP 的稳定性：滑动窗口和流速控制是怎么回事？</a>”介绍 TCP 协议滑动窗口的时候，还提到过一个词，叫作 MSS，这里我们复习下。MSS（Maximun Segment Size，最大段大小）是 TCP 段，或者称为 TCP 分组（TCP Packet）的最大大小。<strong data-nodeid="1274">MSS 是传输层概念，MTU 是链路层概念</strong>。</p>
<p data-nodeid="1140">聪明的同学可以能会意识到，这不就是下面这样一个数学关系吗？</p>
<pre class="lang-java" data-nodeid="1141"><code data-language="java">MTU = MSS + TCP Header + IP Header
</code></pre>
<p data-nodeid="1142"><strong data-nodeid="1284">这个思路有一定道理，但是不对</strong>。先说说这个思路怎么来的，你可能会这么思考：TCP 传输数据大于 MSS，就拆包。每个封包加上 TCP Header ，之后经过 IP 协议，再加上 IP Header。于是这个加上 IP 头的分组（Packet）不能超过 MTU。固然这个思路很有道理，可惜是错的。<strong data-nodeid="1285">因为 TCP 解决的是广域网的问题，MTU 是一个链路层的概念，要知道不同网络 MTU 是不同的，所以二者不可能产生关联。这也是为什么 IP 协议还可能会再拆包的原因</strong>。</p>
<h3 data-nodeid="1143">地址解析协议（ARP）</h3>
<p data-nodeid="1144">上面我们讨论了 MAC 地址，链路层通过 MAC 地址定位网络接口（网卡）。在一个网络接口向另一个网络接口发送数据的时候，至少要提供这样 3 个字段：</p>
<ol data-nodeid="1145">
<li data-nodeid="1146">
<p data-nodeid="1147">源 MAC 地址</p>
</li>
<li data-nodeid="1148">
<p data-nodeid="1149">目标 MAC 地址</p>
</li>
<li data-nodeid="1150">
<p data-nodeid="1151">数据</p>
</li>
</ol>
<p data-nodeid="1152"><strong data-nodeid="1295">这里我们一起再来思考一个问题，对于一个网络接口，它如何能知道目标接口的 MAC 地址呢</strong>？我们在使用传输层协议的时候，清楚地知道目的地的 IP 地址，但是我们不知道 MAC 地址。这个时候，就需要一个中间服务帮助根据 IP 地址找到 MAC 地址——这就是地址解析协议（Address Resolution Protocol，ARP）。</p>
<p data-nodeid="1153">整个工作过程和 DNS 非常类似，如果一个网络接口已经知道目标 IP 地址对应的 MAC 地址了，它会将数据直接发送给交换机，交换机将数据转发给目的地，这个过程如下图所示：</p>
<p data-nodeid="2524" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/39/Cgp9HWCJIN6ALcIOAAGvaaKiqtM412.png" alt="图片3.png" data-nodeid="2528"></p>
<div data-nodeid="2525"><p style="text-align:center">已知目的地 MAC 可以直接发送</p></div>


<p data-nodeid="1156">那么如果网络接口不知道目的地地址呢？这个时候，地址解析协议就开始工作了。发送接口会发送一个广播查询给到交换机，交换机将查询转发给所有接口。</p>
<p data-nodeid="3087" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3C/42/CioPOWCJIOaAWL2vAAIjZo8-JVY343.png" alt="图片4.png" data-nodeid="3090"></p>

<p data-nodeid="1158">如果某个接口发现自己就是对方要查询的接口，则会将自己的 MAC 地址回传。接下来，会在交换机和发送接口的 ARP 表中，增加一个缓存条目。也就是说，接下来发送接口再次向 IP 地址 2.2.2.2 发送数据时，不需要再广播一次查询了。</p>
<p data-nodeid="3649" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/39/Cgp9HWCJIO-ASE1gAAIXUxAepRs289.png" alt="图片5.png" data-nodeid="3652"></p>

<p data-nodeid="1160"><strong data-nodeid="1316">前面提到这个过程和 DNS 非常相似，采用的是逐级缓存的设计减少 ARP 请求</strong>。发送接口先查询本地的 ARP 表，如果本地没有数据，然后广播 ARP 查询。这个时候如果交换机中有数据，那么查询交换机的 ARP 表；如果交换机中没有数据，才去广播消息给其他接口。<strong data-nodeid="1317">注意，ARP 表是一种缓存，也要考虑缓存的设计</strong>。通常缓存的设计要考虑缓存的失效时间、更新策略、数据结构等。</p>
<p data-nodeid="1161">比如可以考虑用 TTL（Time To Live）的设计，为每个缓存条目增加一个失效时间。另外，更新策略可以考虑利用老化（Aging）算法模拟 LRU。</p>
<p data-nodeid="1162">最后请你思考路由器和交换机的异同点。不知道你有没有在网上订购过家用无线路由器，通常这种家用设备也会提供局域网，具备交换机的能力。同时，这种设备又具有路由器的能力。所以，很多同学可能会分不清路由器和交换机。</p>
<p data-nodeid="1163">总的来说，家用的路由器，也具备交换机的功能。但是当 ARP 表很大的时候，就需要专门的、能够承载大量网络接口的交换设备。就好比，如果用数组实现 ARP 表，数据量小的时候，遍历即可；但如果数据量大的话，就需要设计更高效的查询结构和设计缓存。</p>
<p data-nodeid="1164">详细的缓存设计原理的介绍，可以参考<a href="https://shenceyun.lagou.com/t/Axo?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1324">《重学操作系统》</a>专栏中关于 CPU 缓存的设计，以及 MMU 中 TLB 的设计的内容，分别在以下 3 讲：</p>
<ul data-nodeid="1165">
<li data-nodeid="1166">
<p data-nodeid="1167"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4610&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1330">05 | 存储器分级：L1 Cache 比内存和 SSD 快多少倍？</a></p>
</li>
<li data-nodeid="1168">
<p data-nodeid="1169"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4634&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1335">25 | 内存管理单元： 什么情况下使用大内存分页？</a></p>
</li>
<li data-nodeid="1170">
<p data-nodeid="1171"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4635&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1340">26 | 缓存置换算法： LRU 用什么数据结构实现更合理？</a></p>
</li>
</ul>
<h3 data-nodeid="1172">连接内网</h3>
<p data-nodeid="1173">有时候，公司内部有多个子网。这个时候一个子网如果要访问另一个子网，就需要通过路由器。</p>
<p data-nodeid="4211" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3C/42/CioPOWCJIPqACRqBAAJZJ8-Xz9M520.png" alt="图片66.png" data-nodeid="4214"></p>

<p data-nodeid="1175">也就是说，图中的路由器，其实充当了两个子网通信的桥梁。在上述过程中，发送接口不能直接通过 MAC 地址发送数据到接收接口，因为子网 1 的交换机不知道子网 2 的接口。这个时候，发送接口需要通过 IP 协议，将数据发送到路由器，再由路由器转发信息到子网 2 的交换机。这里提一个问题，<strong data-nodeid="1351">子网 2 的交换机如何根据 IP 地址找到接收接口呢</strong>？答案是通过查询 ARP 表。</p>
<h3 data-nodeid="1176">连接外网（网络地址转换技术，NAT）</h3>
<p data-nodeid="1177">最后我们讨论下连接外网的问题。</p>
<p data-nodeid="1178">IPv4 协议因为存在网络地址耗尽的问题，不能为一个公司提供足够的地址，因此内网 IP 可能会和外网重复。比如内网 IP 地址<code data-backticks="1" data-nodeid="1355">192.168.0.1</code>发送信息给<code data-backticks="1" data-nodeid="1357">22.22.22.22</code>，这个时候，其实是跨着网络的。</p>
<p data-nodeid="4773" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/3A/Cgp9HWCJIQWATnKCAAJiZ0IiUQw856.png" alt="图片6.png" data-nodeid="4776"></p>

<p data-nodeid="1180">跨网络必然会通过多次路由，最终将消息转发到目的地。但是这里存在一个问题，寻找的目标 IP 地址<code data-backticks="1" data-nodeid="1363">22.22.22.22</code>是一个公网 IP，可以通过正常的寻址 + 路由算法定位。当<code data-backticks="1" data-nodeid="1365">22.22.22.22</code>寻找<code data-backticks="1" data-nodeid="1367">192.168.0.1</code>的时候，是寻找一个私网 IP，这个时候是找不到的。解决方案就是网络地址转换技术（Network Address Translation）。</p>
<p data-nodeid="5335" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/3C/3A/Cgp9HWCJIQ6AX2bSAAF-MBsPxPo191.png" alt="图片7.png" data-nodeid="5338"></p>

<p data-nodeid="1182">NAT 技术转换的是 IP 地址，私有 IP 通过 NAT 转换为公网 IP 发送到服务器。服务器的响应，通过 NAT 转换为私有 IP，返回给客户端。通过这种方式，就解决了内网和外网的通信问题。</p>
<h3 data-nodeid="1183">总结</h3>
<p data-nodeid="1184">总结一下，链路层发送数据靠的是 MAC 地址，MAC 地址就好像人的身份证一样。局域网中，数据不可能从一个终端直达另一个终端，而是必须经过交换机交换。交换机也叫作链路层交换机，它的工作就是不断接收数据，然后转发数据。<strong data-nodeid="1379">通常意义上，交换机不具有路由功能，路由器往往具有交换功能</strong>。但是往往路由器交换的效率，不如交换机。已知 IP 地址，找到 MAC 地址的协议，叫作地址解析协议（ARP）。</p>
<p data-nodeid="1185">网络和网络的衔接，必须有路由器（或者等价的设备）。一个网络的设备不能直接发送链路层分组给另一个网络的设备，而是需要通过 IP 协议让路由器转发。</p>
<p data-nodeid="1186">那么，通过这一讲的学习，你可以来回答本讲关联的面试题目：网络地址转换协议是如何工作的？</p>
<p data-nodeid="1187">【<strong data-nodeid="1387">解析</strong>】网络地址解析协议（NAT）解决的是内外网通信的问题。NAT 通常发生在内网和外网衔接的路由器中，由路由器中的 NAT 模块提供网络地址转换能力。从设计上看，NAT 最核心的能力，就是能够将内网中某个 IP 地址映射到外网 IP，然后再把数据发送给外网的服务器。当服务器返回数据的时候，NAT 又能够准确地判断外网服务器的数据返回给哪个内网 IP。</p>
<p data-nodeid="1188">你可以思考下 NAT 是如何做到这点的呢？需要做两件事。</p>
<ol data-nodeid="1189">
<li data-nodeid="1190">
<p data-nodeid="1191">NAT 需要作为一个中间层替换 IP 地址。 发送的时候，NAT 替换源 IP 地址（也就是将内网 IP 替换为出口 IP）；接收的时候，NAT 替换目标 IP 地址（也就是将出口 IP 替换回内网 IP 地址）。</p>
</li>
<li data-nodeid="1192">
<p data-nodeid="1193">NAT 需要缓存内网 IP 地址和出口 IP 地址 + 端口的对应关系。也就是说，发送的时候，NAT 要为每个替换的内网 IP 地址分配不同的端口，确保出口 IP 地址+ 端口的唯一性，这样当服务器返回数据的时候，就可以根据出口 IP 地址 + 端口找到内网 IP。</p>
</li>
</ol>
<h3 data-nodeid="1194">思考题</h3>
<p data-nodeid="1195"><strong data-nodeid="1395">最后再给你提一道需要查资料的思考题：IPv6 协议还需要 NAT 吗？</strong></p>
<p data-nodeid="1196">我建议你拿出几分钟的时间去查一下资料，然后把答案整理在留言区，我们一起讨论。如果你对本次课程有什么建议和疑问，可以在评论区留言。如果你有所收获，也可以推荐给你的朋友。</p>
<p data-nodeid="1197" class="">这一讲就到这里。发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲我们将学习“09 | TCP 实战：如何进行 TCP 抓包调试？”再见！</p>

---

### 精选评论

##### **兴：
> 虽然IPV6地址足够使用，但是有时候我们会希望隐藏内网的拓扑结构，因此NAT依然有其使用的意义

##### **钊：
> 到底需要不?? 我是来看评论 学习的

##### **聪：
> IPv6 NAT是必要的主要原因有４条：１）重编号，一个用户换ISP重编号的解决方案，“草根”技术NAT目前是最好的，IETF推荐的“贵族”技术PI技术不敢真正使用(主要是路由扩展性问题)。２）Multihoming，IETF同样无解，NAT目前最佳；３）内部拓扑的隐藏，别的一些技术可能可以做到，NAT也可以；４）计算主机的数量，有了NAT后就不太好计算了。

##### **阳：
> IPV4地址不够用所以需要NAT。IPV6下地址是够用的，可以全部用公网IP，也就可以去掉NAT。

##### **男：
> IPV4存在地址耗尽，内外网地址重复，需要NAT，IPV6可以去掉NAT模块

##### **铧：
> NAT存在的原因是ip地址不足，需要ip转换和内外网映射，ipv6地址充足，不不需要转换

##### **棋：
> NAT是用来解决IP地址不够用的情况，内外网重复的问题，而IPV6IP足够，所以IPV6不需要用NAT。

##### *超：
> 理论上应该是不需要了，每个设备都有自己的ip了，就不需要转换了呀。路由器直接找到相应的设备。

##### **元：
> 准确来说，解决 IPv4 不够用的是 NAPT

