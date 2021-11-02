<p data-nodeid="1029" class="">之前我们在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7266&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1126">02 | 传输层协议 TCP ： TCP 为什么握手是 3 次、挥手是 4 次？</a>”提到过，TCP 和 UDP 是今天应用最广泛的传输层协议，拥有最核心的垄断地位。<strong data-nodeid="1132">TCP 最核心的价值是提供了可靠性，而 UDP 最核心的价值是灵活，你几乎可以用它来做任何事情</strong>。例如：HTTP 协议 1.1 和 2.0 都基于 TCP，而到了 HTTP 3.0 就开始用 UDP 了。</p>
<p data-nodeid="1813" class="te-preview-highlight">如果你打开 <a href="https://tools.ietf.org/html/rfc793?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1817">TCP 协议的 RFC</a><a href="https://tools.ietf.org/html/rfc793?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1820">文档</a>，可以看到目录中一共有 85 页；如果你打开 <a href="https://tools.ietf.org/html/rfc768?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1824">UDP 的 RFC 文档</a>，会看到目录中只有 3 页。一个只有 3 页的协议，能够成为今天最主流的传输层协议之一，那么它一定有非常值得我们学习的地方。</p>


<p data-nodeid="1031">UDP 在数据传输、网络控制、音视频、Web 技术中，都有很重要的地位，因此它也是面试常考的内容。设计系统时候，UDP 经常拿来和 TCP 比较。这一讲，就以 TCP 协议和 UDP 协议的优势和劣势为引，开启我们今天的学习。</p>
<h3 data-nodeid="1032">UDP 协议</h3>
<p data-nodeid="1033"><strong data-nodeid="1151">UDP（User Datagram Protocol），目标是在传输层提供直接发送报文（Datagram）的能力</strong>。Datagram 是数据传输的最小单位。UDP 协议不会帮助拆分数据，它的目标只有一个，就是发送报文。</p>
<p data-nodeid="1034">有细心的同学可能会问： 为什么不直接调用 IP 协议呢？ 如果裸发数据，IP 协议不香吗？</p>
<p data-nodeid="1035">这是因为传输层协议在承接上方应用层的调用，需要提供应用到应用的通信——因此要附上端口号。每个端口，代表不同的应用。传输层下层的 IP 协议，承接传输层的调用，将数据从主机传输到主机。IP 层不能区分应用，导致哪怕是在 IP 协议上进行简单封装，也需要单独一个协议。这就构成了 UDP 协议的市场空间。</p>
<h3 data-nodeid="1036">UDP 的封包格式</h3>
<p data-nodeid="1037">UDP 的设计目标就是在允许用户直接发送报文的情况下，最大限度地简化应用的设计。下图是 UDP 的报文格式。</p>
<p data-nodeid="1038"><img src="https://s0.lgstatic.com/i/image6/M01/3B/0F/Cgp9HWCCfQeAGOF3AACK2Gf5t6I606.png" alt="图片1.png" data-nodeid="1158"></p>
<p data-nodeid="1039">你可以看到，UDP 的报文非常简化，只有 5 个部分。</p>
<ul data-nodeid="1040">
<li data-nodeid="1041">
<p data-nodeid="1042">Source Port 是源端口号。因为 UDP 协议的特性（不需要 ACK），因此这个字段是可以省略的。但有时候对于防火墙、代理来说，Source Port 有很重要的意义，它们需要用这个字段行过滤和路由。</p>
</li>
<li data-nodeid="1043">
<p data-nodeid="1044">Destination Port 是目标端口号（这个字段不可以省略）。</p>
</li>
<li data-nodeid="1045">
<p data-nodeid="1046">Length 是消息体长度。</p>
</li>
<li data-nodeid="1047">
<p data-nodeid="1048">Checksum 是校验和，作用是检查封包是否出错。</p>
</li>
<li data-nodeid="1049">
<p data-nodeid="1050">Data octets 就是一个字节一个字节的数据，Octet 是 8 位。</p>
</li>
</ul>
<p data-nodeid="1051">下面我们先简单聊聊校验和（Checksum）机制，这个机制在很多的网络协议中都会存在，因为校验数据在传输过程中有没有丢失、损坏是一个普遍需求。在一次网络会话中，我们传输的内容可能是：“你好！”，但事实上传输的是 01 组成的二进制。请你思考这样一个算法，我们把数据分成一个一个 byte，然后将所有 byte 相加，再将最终的结果取反。</p>
<p data-nodeid="1052">比如现在数据有 4 个 byte：a,b,c,d，那么一种最简单的校验和就是：</p>
<pre class="lang-java" data-nodeid="1053"><code data-language="java">checksum=(a+b+c+d) ^ <span class="hljs-number">0xff</span>
</code></pre>
<p data-nodeid="1054"><strong data-nodeid="1175">如果发送方用上述方式计算出 Checksum，并将 a,b,c,d 和 Checksum 一起发送给接收方，接收方就可以用同样的算法再计算一遍，这样就可以确定数据有没有发生损坏</strong>（<strong data-nodeid="1176">变化</strong>）。当然 Checksum 的做法，只适用于数据发生少量变化的情况。如果数据发生较大的变动，校验和也可能发生碰撞。</p>
<p data-nodeid="1055">你可以看到 UDP 的可靠性保证仅仅就是 Checksum 一种。如果一个数据封包 Datagram 发生了数据损坏，UDP 可以通过 Checksum 纠错或者修复。 但是 UDP 没有提供再多的任何机制，比如 ACK、顺序保证以及流控等。</p>
<h3 data-nodeid="1056">UDP 与 TCP的区别</h3>
<p data-nodeid="1057">接下来我们说说 UDP 和 TCP 的区别。</p>
<h4 data-nodeid="1058">1. 目的差异</h4>
<p data-nodeid="1059">首先，这两个协议的目的不同：TCP 协议的核心目标是提供可靠的网络传输，而 UDP 的目标是在提供报文交换能力基础上尽可能地简化协议轻装上阵。</p>
<h4 data-nodeid="1060">2. 可靠性差异</h4>
<p data-nodeid="1061">TCP 核心是要在保证可靠性提供更好的服务。TCP 会有握手的过程，需要建立连接，保证双方同时在线。而且TCP 有时间窗口持续收集无序的数据，直到这一批数据都可以合理地排序组成连续的结果。</p>
<p data-nodeid="1062">UDP 并不具备以上这些特性，它只管发送数据封包，而且 UDP 不需要 ACK，这意味着消息发送出去成功与否 UDP 是不管的。</p>
<h4 data-nodeid="1063">3. 连接 vs 无连接</h4>
<p data-nodeid="1064">TCP 是一个面向连接的协议（Connection-oriented Protocol），传输数据必须先建立连接。 UDP 是一个无连接协议（Connection-less Protocol），数据随时都可以发送，只提供发送封包（Datagram）的能力。</p>
<h4 data-nodeid="1065">4. 流控技术（Flow Control）</h4>
<p data-nodeid="1066">TCP 使用了流控技术来确保发送方不会因为一次发送过多的数据包而使接收方不堪重负。TCP 在发送缓冲区中存储数据，并在接收缓冲区中接收数据。当应用程序准备就绪时，它将从接收缓冲区读取数据。如果接收缓冲区已满，接收方将无法处理更多数据，并将其丢弃。UDP 没有提供类似的能力。</p>
<h4 data-nodeid="1067">5. 传输速度</h4>
<p data-nodeid="1068">UDP 协议简化，封包小，没有连接、可靠性检查等，因此单纯从传输速度上讲，UDP 更快。</p>
<h4 data-nodeid="1069">6. 场景差异</h4>
<p data-nodeid="1070">TCP 每个数据封包都需要确认，因此天然不适应高速数据传输场景，比如观看视频（流媒体应用）、网络游戏（TCP 有延迟）等。具体来说，如果网络游戏用 TCP，每个封包都需要确认，可能会造成一定的延迟；再比如音、视频传输天生就允许一定的丢包率；Ping 和 DNSLookup，这类型的操作只需要一次简单的请求/返回，不需要建立连接，用 UDP 就足够了。</p>
<p data-nodeid="1071">近些年有一个趋势，TCP/UDP 的边界逐渐变得模糊，UDP 应用越来越多。比如传输文件，如果考虑希望文件无损到达，可以用 TCP。如果考虑希望传输足够块，就可能会用 UDP。再比如 HTTP 协议，如果考虑请求/返回的可靠性，用 TCP 比较合适。但是像 HTTP 3.0 这类应用层协议，从功能性上思考，暂时没有找到太多的优化点，但是想要把网络优化到极致，就会用 UDP 作为底层技术，然后在 UDP 基础上解决可靠性。</p>
<p data-nodeid="1072"><strong data-nodeid="1210">所以理论上，任何一个用 TCP 协议构造的成熟应用层协议，都可以用 UDP 重构</strong>。这就好比，本来用一个工具可以解决所有问题，但是如果某一类问题体量非常大，就会专门为这类问题创造工具。因此，UDP 非常适合需要定制工具的场景。</p>
<p data-nodeid="1073">下面我把场景分成三类，TCP 应用场景、UDP 应用场景、模糊地带（TCP、UDP 都可以考虑），你可以参考。</p>
<p data-nodeid="1074"><strong data-nodeid="1215">第一类：TCP 场景</strong></p>
<ul data-nodeid="1075">
<li data-nodeid="1076">
<p data-nodeid="1077">远程控制（SSH）</p>
</li>
<li data-nodeid="1078">
<p data-nodeid="1079">File Transfer Protocol（FTP）</p>
</li>
<li data-nodeid="1080">
<p data-nodeid="1081">邮件（SMTP、IMAP）等</p>
</li>
<li data-nodeid="1082">
<p data-nodeid="1083">点对点文件传出（微信等）</p>
</li>
</ul>
<p data-nodeid="1084"><strong data-nodeid="1223">第二类：UDP 场景</strong></p>
<ul data-nodeid="1085">
<li data-nodeid="1086">
<p data-nodeid="1087">网络游戏</p>
</li>
<li data-nodeid="1088">
<p data-nodeid="1089">音视频传输</p>
</li>
<li data-nodeid="1090">
<p data-nodeid="1091">DNS</p>
</li>
<li data-nodeid="1092">
<p data-nodeid="1093">Ping</p>
</li>
<li data-nodeid="1094">
<p data-nodeid="1095">直播</p>
</li>
</ul>
<p data-nodeid="1096"><strong data-nodeid="1232">第三类：模糊地带</strong></p>
<ul data-nodeid="1097">
<li data-nodeid="1098">
<p data-nodeid="1099">HTTP（目前以 TCP 为主）</p>
</li>
<li data-nodeid="1100">
<p data-nodeid="1101">文件传输</p>
</li>
</ul>
<p data-nodeid="1102">以上我们从多个方面了解了 TCP 和 UDP 的区别，最后再来总结一下。UDP 不提供可靠性，不代表我们不能解决可靠性。UDP 的核心价值是灵活、轻量，构造了最小版本的传输层协议。在这个之上，还可以实现连接（Connection），实现会话（Session），实现可靠性（Reliability）……</p>
<h3 data-nodeid="1103">总结</h3>
<p data-nodeid="1104">这一讲我们针对 UDP 协议的内容进行了探讨，到这里互联网协议群的传输层讲解就结束了。协议对于我们来说是非常重要的，协议的制定让所有参与者一致、有序地工作。</p>
<p data-nodeid="1105">学习协议的设计，对你的工作非常有帮助。比如：</p>
<ul data-nodeid="1106">
<li data-nodeid="1107">
<p data-nodeid="1108">学习 TCP 协议可以培养你思维的缜密性——序号的设计、滑动窗口的设计、快速重发的设计、内在状态机的设计，都是非常精妙的想法；</p>
</li>
<li data-nodeid="1109">
<p data-nodeid="1110">学习 UDP 协议可以带动我们反思自己的技术架构，有时候简单的工具更受欢迎。Linux 下每个工具都是那么简单、专注，容易理解。相比 TCP 协议，UDP 更容易理解。</p>
</li>
</ul>
<p data-nodeid="1111">从程序架构上来说，今天我们更倾向于简单专注的设计，我们更期望有解决报文传输的工具、有解决可靠性的工具、有解决流量控制的工具、有解决连接和会话的工具……我相信这应该是未来的趋势——由大量优质的工具逐渐取代历史上沉淀下来完整统一的系统。从这个角度，我希望通过学习传输层的知识，能够帮助你重新审视自己的系统设计，看看自己还有哪些进步的空间。</p>
<p data-nodeid="1112">那么通过这一讲的学习，你可以尝试来回答 TCP 协议和 UDP 协议的优势和劣势？</p>
<p data-nodeid="1113">【<strong data-nodeid="1252">解析</strong>】<strong data-nodeid="1253">TCP 最核心的价值就是提供封装好的一套解决可靠性的优秀方案</strong>。 在前面 3 讲中，你可以看到解决可靠性是非常复杂的，要考虑非常多的因素。TCP 帮助我们在确保吞吐量、延迟、丢包率的基础上，保证可靠性。</p>
<p data-nodeid="1114">历史上 TCP 也是靠可靠性起家的，有一次著名的实验，TCP 协议的设计者做了一次演示——利用 TCP 协议将数据在卫星和地面之间传播了很多次，没有发生任何数据损坏。从那个时候开始，研发人员开始大量选择 TCP 协议。然后随着生态的发展，逐渐提供了流控等能力。<strong data-nodeid="1259">TCP 的成功在于它给人们提供了很多现成、好用的能力</strong>。</p>
<p data-nodeid="1115"><strong data-nodeid="1268">UDP 则不同，UDP 提供了最小版的实现，只支持 Checksum</strong>。<strong data-nodeid="1269">UDP 最核心的价值是灵活、轻量、传输速度快</strong>。考虑到不同应用的特性，如果不使用一个大而全的方案，为自己的应用特性量身定做，可能会做得更好。比如网络游戏中游戏客户端不断向服务端发送玩家的位置，如果某一次消息丢失了，只要这个消息不影响最终的游戏结果，就可以只看下一个消息。不同应用有不同的特性，需要的可靠性级别不一样，这就是越来越多的应用开始使用 UDP 的原因之一。</p>
<p data-nodeid="1116">其实对于我们来说，TCP 协议和 UDP 协议根本不存在什么优势和劣势，只不过是场景不同，选择不同而已。<strong data-nodeid="1275">最后还有一个非常重要的考虑因素就是成本，如果没有足够专业的团队解决网络问题，TCP 无疑会是更好的选择</strong>。</p>
<h3 data-nodeid="1117">思考题</h3>
<p data-nodeid="1118"><strong data-nodeid="1281">最后我再给你出一道需要查资料的思考题：Moba 类游戏的网络应该用 TCP 还是 UDP</strong>？</p>
<p data-nodeid="1119">可以把你的想法写在留言区，我们一起讨论。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。如果你对本次课程有什么建议和疑问，可以在评论区留言和我讨论。</p>
<p data-nodeid="1120" class="">这一讲就到这里。发现求知的乐趣，我是林䭽，感谢你学习本次课程。下一讲，我们将学习“06 | IPv4 协议：路由和寻址的区别是什么？”，一起看看传输层下面的网络层如何工作。</p>

---

### 精选评论

##### **2657：
> Moba类游戏的传输协议是基于UDP的封装。首先Moba类游戏一般对实时性有要求，如果使用了TCP，再怎么优化也受TCP连接效率的影响。而UDP传输效率相对来说较高，但可靠性欠佳。因此思路应该是基于UDP协议做一些优化，牺牲部分的传输效率，保证其可靠性，但是又不需要像TCP协议那样有一套完善的机制来保证其可靠性。

##### **男：
> 基于开源项目kcp(在udp的基础上实现可靠性等)

##### **生：
> UDP吧，毕竟我经常丢包……

##### *晨：
> kcp

##### **旭：
> UDP.评论都没有前面多了，继续加油，坚持可贵

##### **1101：
> 应该是用UDP，毕竟MOBA类游戏涉及到多人参与，玩家之间的实时互动需要得到及时的反馈，如果是TCP协议，在一个简单的两个玩家交互场景中，就因为握手和挥手浪费很多带宽，反应还不快；而UDP能很好地解决这个问题，在此之上提高UDP连接的可靠性即可。

