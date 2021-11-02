<p data-nodeid="1373" class="">今天我们将从<strong data-nodeid="1475">稳定性</strong>角度深挖 <strong data-nodeid="1476">TCP 协议的运作机制</strong>。如今，大半个互联网都建立在 TCP 协议之上，我们使用的 HTTP 协议、消息队列、存储、缓存，都需要用到 TCP 协议——这是因为 <strong data-nodeid="1477">TCP 协议提供了可靠性</strong>。简单来说，可靠性就是让数据无损送达。但若是考虑到成本，就会变得非常复杂——因为还需要尽可能地提升吞吐量、降低延迟、减少丢包率。</p>
<p data-nodeid="1374">TCP 协议具有很强的实用性，而可靠性又是 TCP 最核心的能力，所以理所当然成为面试官们津津乐道的问题。具体来说，从一个终端有序地发出多个数据包，经过一个复杂的网络环境，到达目的地的时候会变得无序，而可靠性要求数据恢复到原始的顺序。这里我先给你提出两个问题：</p>
<ul data-nodeid="1375">
<li data-nodeid="1376">
<p data-nodeid="1377">TCP 协议是如何恢复数据的顺序的？</p>
</li>
<li data-nodeid="1378">
<p data-nodeid="1379">拆包和粘包的作用是什么？</p>
</li>
</ul>
<p data-nodeid="1380">下面请你带着这两个问题开始今天的学习。</p>
<h3 data-nodeid="1381">TCP 的拆包和粘包</h3>
<p data-nodeid="1382"><strong data-nodeid="1487">TCP 是一个传输层协议</strong>。TCP 发送数据的时候，往往不会将数据一次性发送，像下图这样：</p>
<p data-nodeid="1383"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3C/Cgp9HWB-mySAMiRJAACvL4JE7Ow394.png" alt="Drawing 1.png" data-nodeid="1490"></p>
<p data-nodeid="1384">而是将数据拆分成很多个部分，然后再逐个发送。像下图这样：</p>
<p data-nodeid="1385"><img src="https://s0.lgstatic.com/i/image6/M00/3A/44/CioPOWB-myyARws0AADwpYVdoRk460.png" alt="Drawing 3.png" data-nodeid="1494"></p>
<p data-nodeid="1386">同样的，在目的地，TCP 协议又需要逐个接收数据。<strong data-nodeid="1500">请你思考，TCP 为什么不一次发送完所有的数据</strong>？比如我们要传一个大小为 10M 的文件，对于应用层而言，就是一次传送完成的。而传输层的协议为什么不选择将这个文件一次发送完呢？</p>
<p data-nodeid="1387">这里有很多原因，比如为了稳定性，一次发送的数据越多，出错的概率越大。再比如说为了效率，网络中有时候存在着并行的路径，拆分数据包就能更好地利用这些并行的路径。再有，比如发送和接收数据的时候，都存在着缓冲区。如下图所示：</p>
<p data-nodeid="1388"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3C/Cgp9HWB-mz2ALAO6AAFJNuQ9-SU088.png" alt="Drawing 5.png" data-nodeid="1504"></p>
<p data-nodeid="1389">缓冲区是在内存中开辟的一块区域，目的是缓冲。因为大量的应用频繁地通过网卡收发数据，这个时候，网卡只能一个一个处理应用的请求。当网卡忙不过来的时候，数据就需要排队，也就是将数据放入缓冲区。如果每个应用都随意发送很大的数据，可能导致其他应用实时性遭到破坏。</p>
<p data-nodeid="1390">还有一些原因我们在<a href="https://shenceyun.lagou.com/t/Axo?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1509">《重学操作系统》</a>专栏的“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4633&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1515">24 | 虚拟内存 ：一个程序最多能使用多少内存？</a>”中讨论过。比如内存的最小分配单位是页表，如果数据的大小超过一个页表，可能会存在页面置换问题，造成性能的损失。如果你对这一部分的知识感兴趣，可以学习我在拉勾教育推出的<a href="https://shenceyun.lagou.com/t/Axo?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1519">《重学操作系统》</a>专栏。</p>
<p data-nodeid="1391">总之，方方面面的原因：<strong data-nodeid="1530">在传输层封包不能太大</strong>。这种限制，往往是以缓冲区大小为单位的。也就是 TCP 协议，会将数据拆分成不超过缓冲区大小的一个个部分。每个部分有一个独特的名词，叫作 <strong data-nodeid="1531">TCP 段（TCP Segment）</strong>。</p>
<p data-nodeid="1392">在接收数据的时候，一个个 TCP 段又被重组成原来的数据。</p>
<p data-nodeid="1393">像这样，数据经过拆分，然后传输，然后在目的地重组，俗称<strong data-nodeid="1542">拆包</strong>。所以拆包是将数据拆分成多个 TCP 段传输。那么粘包是什么呢？有时候，如果发往一个目的地的多个数据太小了，为了防止多次发送占用资源，TCP 协议有可能将它们合并成一个 TCP 段发送，在目的地再还原成多个数据，这个过程俗称<strong data-nodeid="1543">粘包</strong>。所以粘包是将多个数据合并成一个 TCP 段发送。</p>
<h3 data-nodeid="1394">TCP Segment</h3>
<p data-nodeid="1395">那么一个 TCP 段长什么样子呢？下图是一个 TCP 段的格式：</p>
<p data-nodeid="1396"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3C/Cgp9HWB-m0mARV-VAAZgGUE4aeU706.png" alt="Drawing 7.png" data-nodeid="1548"></p>
<p data-nodeid="1397">我们可以看到，TCP 的很多配置选项和数据粘在了一起，作为一个 TCP 段。</p>
<p data-nodeid="1398">显然，让你把每一部分都记住似乎不太现实，但是我会带你把其中最主要的部分理解。<strong data-nodeid="1555">TCP 协议就是依靠每一个 TCP 段工作的，所以你每认识一个 TCP 的能力，几乎都会找到在 TCP Segment 中与之对应的字段</strong>。接下来我先带你认识下它们。</p>
<ol data-nodeid="1399">
<li data-nodeid="1400">
<p data-nodeid="1401">Source Port/Destination Port 描述的是发送端口号和目标端口号，代表发送数据的应用程序和接收数据的应用程序。比如 80 往往代表 HTTP 服务，22 往往是 SSH 服务……</p>
</li>
<li data-nodeid="1402">
<p data-nodeid="1403">Sequence Number 和 Achnowledgment Number 是保证可靠性的两个关键。具体见下文的讨论。</p>
</li>
<li data-nodeid="1404">
<p data-nodeid="1405">Data Offset 是一个偏移量。这个量存在的原因是 TCP Header 部分的长度是可变的，因此需要一个数值来描述数据从哪个字节开始。</p>
</li>
<li data-nodeid="1406">
<p data-nodeid="1407">Reserved 是很多协议设计会保留的一个区域，用于日后扩展能力。</p>
</li>
<li data-nodeid="1408">
<p data-nodeid="1409">URG/ACK/PSH/RST/SYN/FIN 是几个标志位，用于描述 TCP 段的行为。也就是一个 TCP 封包到底是做什么用的？</p>
</li>
</ol>
<p data-nodeid="1410">1）URG 代表这是一个紧急数据，比如远程操作的时候，用户按下了 Ctrl+C，要求终止程序，这种请求需要紧急处理。</p>
<p data-nodeid="1411">2）ACK 代表响应，我们在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7266&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1567">02 | 传输层协议 TCP：TCP 为什么握手是 3 次、挥手是 4 次？</a>”讲到过，所有的消息都必须有 ACK，这是 TCP 协议确保稳定性的一环。</p>
<p data-nodeid="1412">3）PSH 代表数据推送，也就是在传输数据的意思。</p>
<p data-nodeid="1413">4）SYN 同步请求，也就是申请握手。</p>
<p data-nodeid="1414">5）FIN 终止请求，也就是挥手。</p>
<p data-nodeid="1415"><strong data-nodeid="1576">特别说明一下：以上这 5 个标志位，每个占了一个比特，可以混合使用。比如 ACK 和 SYN 同时为 1，代表同步请求和响应被合并了。这也是 TCP 协议，为什么是三次握手的原因之一</strong>。</p>
<p data-nodeid="1416">6） Window 也是 TCP 保证稳定性并进行流量控制的工具，我们会在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7268" data-nodeid="1582">04 | TCP 的稳定性：滑动窗口和流速控制是怎么回事？</a>”中详细介绍。</p>
<p data-nodeid="1417">7）Checksum 是校验和，用于校验 TCP 段有没有损坏。</p>
<p data-nodeid="1418">8）Urgent Pointer 指向最后一个紧急数据的序号（Sequence Number）。它存在的原因是：有时候紧急数据是连续的很多个段，所以需要提前告诉接收方进行准备。</p>
<p data-nodeid="5505" class="te-preview-highlight">9）Options 中存储了一些可选字段，比如接下来我们要讨论的 MSS（Maximun Segment Size）。<br>
10）Padding 存在的意义是因为 Options 的长度不固定，需要 Pading 进行对齐。</p>



<h3 data-nodeid="1420">Sequence Number 和 Acknowledgement Number</h3>
<p data-nodeid="1421">在 TCP 协议的设计当中，数据被拆分成很多个部分，部分增加了协议头。合并成为一个 TCP 段，进行传输。这个过程，我们俗称<strong data-nodeid="1595">拆包</strong>。这些 TCP 段经过复杂的网络结构，由底层的 IP 协议，负责传输到目的地，然后再进行重组。</p>
<p data-nodeid="1422">这里请你思考一个问题：稳定性要求数据无损地传输，也就是说拆包获得数据，又需要恢复到原来的样子。而在复杂的网络环境当中，即便所有的段是顺序发出的，也不能保证它们顺序到达，因此，发出的每一个 TCP 段都需要有序号。这个序号，就是 Sequence Number（Seq）。</p>
<p data-nodeid="1423"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3C/Cgp9HWB-m4CAaJ61AADgfscwtY8443.png" alt="Drawing 9.png" data-nodeid="1599"></p>
<p data-nodeid="1424">如上图所示。发送数据的时候，为每一个 TCP 段分配一个自增的 Sequence Number。接收数据的时候，虽然得到的是乱序的 TCP 段，但是可以通过 Seq 进行排序。</p>
<p data-nodeid="1425">但是这样又会产生一个新的问题——接收方如果要回复发送方，也需要这个 Seq。而网络的两个终端，去同步一个自增的序号是非常困难的。因为任何两个网络主体间，时间都不能做到完全同步，又没有公共的存储空间，无法共享数据，更别说实现一个分布式的自增序号了。</p>
<p data-nodeid="1426">其实这个问题的本质就好像两个人在说话一样，我们要确保他们说出去的话，和回答之间的顺序。因为 TCP 是一个双工的协议，两边可能会同时说话。所以聪明的科学家想到了确定一句话的顺序，需要两个值去描述——<strong data-nodeid="1607">也就是发送的字节数和接收的字节数</strong>。</p>
<p data-nodeid="1427"><img src="https://s0.lgstatic.com/i/image6/M01/3A/45/CioPOWB-m8WAN4r7AAG-F3w2k2k929.png" alt="Drawing 11.png" data-nodeid="1610"></p>
<p data-nodeid="1428">我们重新定义一下 Seq（如上图所示），对于任何一个接收方，如果知道了发送者发送某个 TCP 段时，已经发送了多少字节的数据，那么就可以确定发送者发送数据的顺序。</p>
<p data-nodeid="1429">但是这里有一个问题。如果接收方也向发送者发送了数据请求（或者说双方在对话），接收方就不知道发送者发送的数据到底对应哪一条自己发送的数据了。</p>
<p data-nodeid="1430">举个例子：下面 A 和 B 的对话中，我们可以确定他们彼此之间接收数据的顺序。但是无法确定数据之间的关联关系，所以只有 Sequence Number 是不够的。</p>
<pre class="lang-java" data-nodeid="1431"><code data-language="java">A：今天天气好吗？
A：今天你开心吗？
B：开心
B：天气不好
</code></pre>
<p data-nodeid="1432">人类很容易理解这几句话的顺序，但是对于机器来说就需要特别的标注。因此我们还需要另一个数据，就是每个 TCP 段发送时，发送方已经接收了多少数据。用 Acknowledgement Number 表示，下面简写为 ACK。</p>
<p data-nodeid="1433">下图中，终端发送了三条数据，并且接收到四条数据，通过观察，根据接收到的数据中的 Seq 和 ACK，将发送和接收的数据进行排序。</p>
<p data-nodeid="1434"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3D/Cgp9HWB-m82AUJiLAAHfbaP08JE788.png" alt="Drawing 13.png" data-nodeid="1618"></p>
<p data-nodeid="1435">例如上图中，发送方发送了 100 字节的数据，而接收到的（Seq = 0 和 Seq =100）的两个封包，都是针对发送方（Seq = 0）这个封包的。发送 100 个字节，所以接收到的 ACK 刚好是 100。说明（Seq= 0 和 Seq= 100）这两个封包是针对接收到第 100 个字节数据后，发送回来的。这样就确定了整体的顺序。</p>
<p data-nodeid="1436"><strong data-nodeid="1631">注意，无论 S<b><strong data-nodeid="1630">eq</strong></b> 还是 ACK，都是针对“对方”而言的。是对方发送的数据和对方接收到的数据</strong>。我们在实际的工作当中，可以通过 Whireshark 调试工具观察两个 TCP 连接的 Seq和 ACK。</p>
<p data-nodeid="1437">具体的使用方法，我会在“09 | TCP 实战：如何进行 TCP 抓包调试？"中和你讨论。</p>
<p data-nodeid="1438"><img src="https://s0.lgstatic.com/i/image6/M01/3A/3D/Cgp9HWB-m9WAO8jwAAUDwhNzXjU379.png" alt="Drawing 15.png" data-nodeid="1639"></p>
<h3 data-nodeid="3780" class="">MSS（Maximun Segment Size）</h3>




<p data-nodeid="1440">接下来，我们讨论下 MSS，它也是面试经常会问到的一个 TCP Header 中的可选项（Options），这个可选项控制了 TCP 段的大小，它是一个协商字段（Negotiate）。协议是双方都要遵循的标准，因此配置往往不能由单方决定，需要双方协商。</p>
<p data-nodeid="1441">TCP 段的大小（MSS）涉及发送、接收缓冲区的大小设置，双方实际发送接收封包的大小，对拆包和粘包的过程有指导作用，因此需要双方去协商。</p>
<p data-nodeid="1442">如果这个字段设置得非常大，就会带来一些影响。</p>
<p data-nodeid="1443">首先对方可能会拒绝，作为服务的提供方，你可能不会愿意接收太大的 TCP 段。<strong data-nodeid="1653">因为大的 TCP 段，会降低性能，比如内存使用的性能</strong>。具体你可以参考<a href="https://shenceyun.lagou.com/t/Axo?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1651">《重学操作系统》</a>课程中关于页表的讨论。</p>
<ul data-nodeid="1444">
<li data-nodeid="1445">
<p data-nodeid="1446"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4633&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1658">24 | 虚拟内存 ：一个程序最多能使用多少内存？</a></p>
</li>
<li data-nodeid="1447">
<p data-nodeid="1448"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4634&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1663">25 | 内存管理单元： 什么情况下使用大内存分页？</a></p>
</li>
<li data-nodeid="1449">
<p data-nodeid="1450"><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478#/detail/pc?id=4635&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1668">26 | 缓存置换算法： LRU 用什么数据结构实现更合理？</a></p>
</li>
</ul>
<p data-nodeid="1451">还有就是<strong data-nodeid="1674">资源的占用</strong>。一个用户占用服务器太多的资源，意味着其他的用户就需要等待或者降低他们的服务质量。</p>
<p data-nodeid="1452"><strong data-nodeid="1679">其次，支持 TCP 协议工作的 IP 协议，工作效率会下降</strong>。TCP 协议不肯拆包，IP 协议就需要拆出大量的包。那么 IP 协议为什么需要拆包呢？这是因为在网络中，每次能够传输的数据不可能太大，这受限于具体的网络传输设备，也就是物理特性。但是 IP 协议，拆分太多的封包并没有意义。因为可能会导致属于同个 TCP 段的封包被不同的网络路线传输，这会加大延迟。同时，拆包，还需要消耗硬件和计算资源。</p>
<p data-nodeid="1453">那是不是 MSS 越小越好呢？MSS 太小的情况下，会浪费传输资源（降低吞吐量）。因为数据被拆分之后，每一份数据都要增加一个头部。如果 MSS 太小，那头部的数据占比会上升，这让吞吐量成为一个灾难。<strong data-nodeid="1685">所以在使用的过程当中，MSS 的配置，往往都是一个折中的方案</strong>。而根据 Unix 的哲学，不要去猜想什么样的方案是最合理的，而是要尝试去用实验证明它，一切都要用实验依据说话。</p>
<h3 data-nodeid="1454">总结</h3>
<p data-nodeid="1455">TCP 协议的设计像一台巨大而严密的机器，每次我重新温习 TCP 协议，都会感叹“它庞大，而且很琐碎”。每一个细节的设计，都有很深的思考。比如 Sequence Number 和 Acknowledge Number 的设计，就非常巧妙地利用发送字节数和接收字节数解决了顺序的问题。</p>
<p data-nodeid="1456">那么现在你可以尝试来回答本讲关联的面试题目：<strong data-nodeid="1693">TCP 协议是如何恢复数据的顺序的，TCP 拆包和粘包的作用是什么</strong>？</p>
<p data-nodeid="1457">【<strong data-nodeid="1705">解析</strong>】TCP 拆包的作用是将任务拆分处理，降低整体任务出错的概率，以及减小底层网络处理的压力。拆包过程需要保证数据经过网络的传输，又能恢复到原始的顺序。这中间，需要数学提供保证顺序的理论依据。TCP 利用（发送字节数、接收字节数）的唯一性来确定封包之间的顺序关系。具体的算法，我们会在下一讲“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=837#/detail/pc?id=7268" data-nodeid="1703">04 | TCP 的稳定性解决方案 ：滑动窗口和流量控制是怎么回事？</a>”中给出。粘包是为了防止数据量过小，导致大量的传输，而将多个 TCP 段合并成一个发送。</p>
<h3 data-nodeid="1458">思考题</h3>
<p data-nodeid="1459"><strong data-nodeid="1711">最后再给你留一道练习查资料的思考题：有哪些好用的压测工具</strong>？</p>
<p data-nodeid="1460">可以把你的答案、思路或者课后总结写在留言区，这个输出的过程不仅能够帮助你产生更多的思考，也是构建知识体系的根基，只有内容输出了，才会形成自己的观点。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，记得分享给身边的朋友。期待看到你的思考！</p>
<p data-nodeid="1461" class="">这一讲就到这里，发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲我们将学习“04 | TCP 的稳定性：滑动窗口和流速控制是怎么回事？”。再见！</p>

---

### 精选评论

##### **1001：
> 例如上图中，发送方发送了 100 字节的数据，而接收到的（Seq = 0 和 Seq =100）的两个封包，都是针对发送方（Seq = 0）这个封包的。发送 100 个字节，所以接收到的 ACK 刚好是 100。说明（Seq= 0 和 Seq= 100）这两个封包是针对接收到第 100 个字节数据后，发送回来的。这样就确定了整体的顺序。老师你好！这段话和对应的图不太理解，可以在说明一下吗，比如seq=0,ack=100回应发送方seq=100,数据100 ，为什么seq=100,ack=100也是呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 补充一点理解：这里其实不需要死记Seq，Ack的计算步骤。核心是要理解一个顺序问题，需要理解的核心问题是，TCP如何排序的。换句话说，面试时候，没人会死扣Seq/ACK的计算你关系，核心是你能说出用累计值确认顺序这样一个理解。有了这个理解，相信计算过程是可以推导的。
思考这样一个问题，你和小明聊天， 你会发现，你发出的消息，你可以确定顺序。因为用了聊天工具，接收到消息的时间不确定，你需要理解内容才能确认小明具体那句话回复的是你的哪句话。那么如果是机器，如何确定小明哪句话是针对哪句话回复呢？那就需要小明的回复带上(Seq, Ack)，看到小明的Ack，就知道小明是收到多少自己发送的消息后才进行的回复。通过ACK，你知道小明回复的是你的第1条？第2条？第3条？……还是第k条信息。ACK是小明累计收到的信息数。 如果你要针对小明的回复（Seq, ACK, 内容）进行回复，你就把ACK设置成小明的Seq。

##### **2657：
> 老师，能不能补充一下seq和ack的变化，不太理解其序号变化，谢谢！另外，如果这样做得话，那ack和seq是不是会迅速增大（比如说传一个稍微大一点的数据块），那序号的上限以及不够用的时候怎么处理呢？如果是序号冲突的话（虽然是小概率事件），那是通过什么办法解决冲突呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对于一个端来说，seq是累计的发送字节数，ack是累计的接收字节数。 按照这个方法累计即可。如果达到了最大值，称作序列号回绕问题，内核会处理，具体可以自己再查下资料。 序列号号在初始化的时候会分配随机数，每次连接有自己的序号，因此不存在冲突问题。

##### **用户4774：
> 序号 和 确认号 这部分，有点没整明白

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 补充一点理解：这里其实不需要死记Seq，Ack的计算步骤。核心是要理解一个顺序问题，需要理解的核心问题是，TCP如何排序的。换句话说，面试时候，没人会死扣Seq/ACK的计算你关系，核心是你能说出用累计值确认顺序这样一个理解。有了这个理解，相信计算过程是可以推导的。思考这样一个问题，你和小明聊天， 你会发现，你发出的消息，你可以确定顺序。因为用了聊天工具，接收到消息的时间不确定，你需要理解内容才能确认小明具体那句话回复的是你的哪句话。那么如果是机器，如何确定小明哪句话是针对哪句话回复呢？那就需要小明的回复带上(Seq, Ack)，看到小明的Ack，就知道小明是收到多少自己发送的消息后才进行的回复。通过ACK，你知道小明回复的是你的第1条？第2条？第3条？……还是第k条信息。ACK是小明累计收到的信息数。 如果你要针对小明的回复（Seq, ACK, 内容）进行回复，你就把ACK设置成小明的Seq。

##### **用户3329：
> 老师，有没有什么比较好的计算机网络的小项目推荐，想通过实战加深理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以写一个RPC框架，这个比较简单千行代码就可以搞个雏形。

##### **园：
> TCP数据报最大分片是由数据链路层决定的，这也是为什么要分片的原因。常见的数据链路层协议比如以太网的帧MTU是1500字节，假设IP层和TCP层都是最短首部，即20字节+20字节，所以一般来说TCP的最大分片大小是1500 - 20 - 20 = 1460字节。

##### **瑞：
> jmeter和loadrunner，一个小巧便捷插件丰富，一个功能强大报告好看

##### Yeira：
> 压测工具：LoadRunner、Jmeter、NeoLoad、WebLOAD、Loadster

##### **芸：
> 老师的重学操作系统在哪里可以看到啊

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 在拉勾教育官网或者拉勾教育App都可以看哈，具体链接如下：
https://kaiwu.lagou.com/course/courseInfo.htm?courseId=478&sid=20-h5Url-0&lgec_type=website&lgec_sign=86228E00A960E2EB44DCA4027393428B&buyFrom=2&pageId=1pz4#/sale

##### **曦：
> Sequence Number 和 Acknowledgement Number顺序这里还是没太看懂，为什么send端发送第一个seq=0，100字节发包时会受到两个ack响应？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 双工协议，都可以发送、接收。并不是两个ACK，是每个封包都会有Seq和ACK字段。你说的两次ACK，其实是一个ACK一个PSH。

##### **士星矢：
> 问题：发送方发送了 100 字节的数据，而接收到的（Seq = 0 和 Seq =100）的两个封包，都是针对发送方（Seq = 0）这个封包的。为什么接收方需要发送两个封包呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这不是接收方两个封包。这个是接收方返回了一个ACK，然后又发了一个PSH。双工协议，双方平等的。

##### *西：
> Sequence Number 和 Acknowledgement Number这一段没看懂，感觉写的比较模糊，没说清楚。😂

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 补充一点理解：这里其实不需要死记Seq，Ack的计算步骤。核心是要理解一个顺序问题，需要理解的核心问题是，TCP如何排序的。换句话说，面试时候，没人会死扣Seq/ACK的计算你关系，核心是你能说出用累计值确认顺序这样一个理解。有了这个理解，相信计算过程是可以推导的。思考这样一个问题，你和小明聊天， 你会发现，你发出的消息，你可以确定顺序。因为用了聊天工具，接收到消息的时间不确定，你需要理解内容才能确认小明具体那句话回复的是你的哪句话。那么如果是机器，如何确定小明哪句话是针对哪句话回复呢？那就需要小明的回复带上(Seq, Ack)，看到小明的Ack，就知道小明是收到多少自己发送的消息后才进行的回复。通过ACK，你知道小明回复的是你的第1条？第2条？第3条？……还是第k条信息。ACK是小明累计收到的信息数。 如果你要针对小明的回复（Seq, ACK, 内容）进行回复，你就把ACK设置成小明的Seq。

##### *旭：
> 为啥发一个包，有两个ack呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这不是接收方两个封包。这个是接收方返回了一个ACK，然后又发了一个PSH。双工协议，双方平等的。

##### *航：
> 😀

##### *振：
> 不好意思，上一条评论搞错了。。

##### **强：
> 仅仅接触过jmeter

##### **男：
> 压测工具jmeter和loadrunner用的较多

##### **8746：
> 图中send 800字节receive 700对不上了，没有画全？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; ack和seq 本身就已经是累计值， 不需要再加和。

##### **阳：
> 三次握手中 接收方对发送方的第三次发送的ack不做回复发送方再发请求的时候响应号和第三次握手的响应号相同吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Seq和Ack从第一次握手就存在了。

##### centos：
> 搜了一下，jmeter，ab

##### outer199：
> ab wrk

##### *西：
> 发送方只发送了三个包，为什么会收到4个，甚至更多个ack包呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这不是接收方两个封包。这个是接收方返回了一个ACK，然后又发了一个PSH。双工协议，双方平等的。

##### **臣：
> ab

##### **杰：
> 有哪些好用的压测工具？abtest、jmeter、loadrunner

##### *伟：
> Jmeter

##### *西：
> 压测工具？我只会用jemeter😈

