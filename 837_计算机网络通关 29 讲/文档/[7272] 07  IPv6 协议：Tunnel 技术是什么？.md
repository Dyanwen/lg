<p data-nodeid="175776">IPv4 用 32 位整数描述地址，最多只能支持 43 亿设备，显然是不够用的，这也被称作 IP 地址耗尽问题。</p>
<p data-nodeid="175777">为了解决这个问题，有一种可行的方法是<strong data-nodeid="175895">拆分子网</strong>。拆分子网，会带来很多问题，比如说内外网数据交互，需要网络地址转换协议（NAT 协议），增加传输成本。再比如说，多级网络会增加数据的路由和传输链路，降低网络的速度。理想的状态当然是所有设备在一个网络中，互相可以通过地址访问。</p>
<p data-nodeid="175778">为了解决这个问题，1998 年互联网工程工作小组推出了全新款的 IP 协议——IPv6 协议。但是目前 IPv6 的普及程度还不够高，2019 年据中国互联网络信息中心（CNNIC）统计，IPv6 协议目前在我国普及率为 60%，已经位居世界首位。</p>
<p data-nodeid="176769" class="">既然不能做到完全普及，也就引出了<strong data-nodeid="176775">本讲关联的一道面试题目：什么是 Tunnel 技术</strong>？下面请你带着这个问题，开启今天的学习吧！</p>

<h3 data-nodeid="175780">IPv4 和 IPv6 相似点</h3>
<p data-nodeid="175781">IPv6 的工作原理和 IPv4 类似，分成切片（Segmentation）、增加封包头、路由（寻址）这样几个阶段去工作。IPv6 同样接收上方主机到主机（Host-to-Host）协议传递来的数据，比如一个 TCP  段（Segment），然后将 TCP 段再次切片做成一个个的 IPv6 封包（Datagram or Packet），再调用底层局域网能力（数据链路层）传输数据。具体的过程如下图所示：</p>
<p data-nodeid="177438" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3C/0B/CioPOWCH4u-AWVEAAAH_xR5D6lU716.png" alt="Drawing 1.png" data-nodeid="177441"></p>


<p data-nodeid="175784"><strong data-nodeid="175915">作为网络层协议的 IPv6，最核心的能力是确保数据可以从发送主机到达接收主机</strong>。因此，和 IPv4 类似，IPv6同样需要定义地址的格式，以及路由算法如何工作。</p>
<h3 data-nodeid="175785">IPv6 地址</h3>
<p data-nodeid="175786">接下来我们重点说说地址格式的区别。</p>
<p data-nodeid="175787">IPv4 的地址是 4 个 8 位（octet），总共 32 位。 IPv6 的地址是 8 个 16 位（hextet），总共 128 位。从这个设计来看，IPv6 可以支持的地址数量是 IPv4 的很多倍。就算将 IPv6 的地址分给每个人，每个人拥有的地址数量，依旧是今天总地址数量的很多倍。</p>
<p data-nodeid="175788">格式上，IPv4 的地址用<code data-backticks="1" data-nodeid="175920">.</code>分割，如<code data-backticks="1" data-nodeid="175922">103.28.7.35</code>。每一个是 8 位，用 0-255 的数字表示。</p>
<p data-nodeid="175789">IPv6 的地址用<code data-backticks="1" data-nodeid="175925">:</code>分割，如<code data-backticks="1" data-nodeid="175927">0123:4567:89ab:cdef:0123:4567:89ab:cdef</code>，总共 8 个 16 位的数字，通常用 16 进制表示。</p>
<p data-nodeid="175790">#图片需要重绘，并参考下方中英翻译，在图中标出对应中文</p>
<ul data-nodeid="175791">
<li data-nodeid="175792">
<p data-nodeid="175793">Hexadecimal notation：十六进制表示</p>
</li>
<li data-nodeid="175794">
<p data-nodeid="175795">Quartet：16 位</p>
</li>
<li data-nodeid="175796">
<p data-nodeid="175797">Most significant：最高有效位</p>
</li>
<li data-nodeid="175798">
<p data-nodeid="175799">Binary notation：二进制表示</p>
</li>
</ul>
<p data-nodeid="178096" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/0B/CioPOWCH4wGAT3bUAALH_YQ0Q-U502.png" alt="Drawing 3.png" data-nodeid="178099"></p>


<p data-nodeid="175802">上图中的地址是一个 IPv6 地址的完全态，其实也有简写的方式。比如:</p>
<pre class="lang-java" data-nodeid="175803"><code data-language="java"><span class="hljs-number">0123</span>:<span class="hljs-number">4567</span>:<span class="hljs-number">0000</span>:<span class="hljs-number">0000</span>:<span class="hljs-number">0123</span>:<span class="hljs-number">4567</span>:<span class="hljs-number">0000</span>:cdef
</code></pre>
<p data-nodeid="175804">可以省略前 64 字节的<code data-backticks="1" data-nodeid="175942">0000:0000</code>简写为：</p>
<pre class="lang-java" data-nodeid="175805"><code data-language="java"><span class="hljs-number">0123</span>:<span class="hljs-number">4567</span>::<span class="hljs-number">0123</span>:<span class="hljs-number">4567</span>:<span class="hljs-number">0000</span>:cdef
</code></pre>
<p data-nodeid="175806"><code data-backticks="1" data-nodeid="175944">::</code>只能出现一次，相当于省略了若干组<code data-backticks="1" data-nodeid="175946">0000</code>。比如说<code data-backticks="1" data-nodeid="175948">1111::2222</code>相当于中间省略了 6 组<code data-backticks="1" data-nodeid="175950">0000</code>。为什么不能出现两个<code data-backticks="1" data-nodeid="175952">::</code>呢？因为如果有两个<code data-backticks="1" data-nodeid="175954">::</code>，就会对省略的<code data-backticks="1" data-nodeid="175956">0000</code>的位置产生歧义。比如说<code data-backticks="1" data-nodeid="175958">1111::2222:3333</code>，你就不知道究竟<code data-backticks="1" data-nodeid="175960">0000</code>在<code data-backticks="1" data-nodeid="175962">1111::2222</code>和<code data-backticks="1" data-nodeid="175964">2222::3333</code>是怎么分布的。</p>
<p data-nodeid="175807">开头的 0 也可以简写，就变成如下的样子：</p>
<pre class="lang-java" data-nodeid="175808"><code data-language="java"><span class="hljs-number">123</span>:<span class="hljs-number">4567</span>::<span class="hljs-number">123</span>:<span class="hljs-number">4567</span>:<span class="hljs-number">0</span>:cdef
</code></pre>
<p data-nodeid="175809">还有一种情况我们想要后面部分都填<code data-backticks="1" data-nodeid="175968">0</code>，比如说<code data-backticks="1" data-nodeid="175970">3c4d::/16</code>，这个代表只有前<code data-backticks="1" data-nodeid="175972">16</code>位有数据，后面是<code data-backticks="1" data-nodeid="175974">0</code>；<code data-backticks="1" data-nodeid="175976">1234:5878:abcd/64</code>代表只有左边<code data-backticks="1" data-nodeid="175978">64</code>位有数据，后面是 0；再比如<code data-backticks="1" data-nodeid="175980">ff00/8</code>，只有左边 8 位是有数据的。</p>
<h3 data-nodeid="175810">IPv6 的寻址</h3>
<p data-nodeid="175811">接下来我们讨论下寻址，和 IPv4 相同，寻址的目的是找到设备，以及规划到设备途经的路径。和 IPv4 相同，IPv6寻址最核心的内容就是要对网络进行划分。IPv6 地址很充裕，因此对网络的划分和 IPv4 有很显著的差异。</p>
<p data-nodeid="175812">IPv6 的寻址分成了几种类型：</p>
<ul data-nodeid="175813">
<li data-nodeid="175814">
<p data-nodeid="175815">全局单播寻址（和 IPv4 地址作用差不多，在互联网中通过地址查找一个设备，简单来说，单播就是 1  对  1）；</p>
</li>
<li data-nodeid="175816">
<p data-nodeid="175817">本地单播（类似 IPv4 里的一个内部网络，要求地址必须以<code data-backticks="1" data-nodeid="175987">fe80</code>开头，类似我们 IPv4 中<code data-backticks="1" data-nodeid="175989">127</code>开头的地址）；</p>
</li>
<li data-nodeid="175818">
<p data-nodeid="175819">分组多播（Group Multicast），类似今天我们说的广播，将消息发送给多个接收者；</p>
</li>
<li data-nodeid="175820">
<p data-nodeid="175821">任意播（Anycast），这个方式比较特殊，接下来我们会详细讲解。</p>
</li>
</ul>
<h4 data-nodeid="180687" class="">全局单播</h4>




<p data-nodeid="181329" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/03/Cgp9HWCH4w-AEinAAAHIfeF4_II848.png" alt="Drawing 5.png" data-nodeid="181332"></p>



<p data-nodeid="175826">全局单播，就是将消息从一个设备传到另一个设备，这和 IPv4 发送/接收消息大同小异。而全局单播地址，目标就是定位网络中的设备，这个地址和 IPv4 的地址作用相同，只不过格式略有差异。<strong data-nodeid="176009">总的来说，IPv6 地址太多，因此不再需要子网掩码，而是直接将 IPv6 的地址分区即可</strong>。</p>
<p data-nodeid="175827">在实现全局单播时，IPv6 地址通常分成 3 个部分：</p>
<ul data-nodeid="175828">
<li data-nodeid="175829">
<p data-nodeid="175830">站点前缀（Site Prefix）48bit，一般是由 ISP（Internet Service Providor，运营商）或者RIR（Regional Internet Registry， 地区性互联网注册机构），RIR 将 IP 地址分配给运营商；</p>
</li>
<li data-nodeid="175831">
<p data-nodeid="175832">子网号（Subnet ID），16bit，用于站点内部区分子网；</p>
</li>
<li data-nodeid="175833">
<p data-nodeid="175834">接口号（Interface ID）， 64bit，用于站点内部区分设备。</p>
</li>
</ul>
<p data-nodeid="175835">因此 IPv6 也是一个树状结构，站点前缀需要一定资质，子网号和接口号内部定义。IPv6 的寻址过程就是先通过站点前缀找到站点，然后追踪子网，再找到接口（也就是设备的网卡）。</p>
<p data-nodeid="175836">从上面全局单播的分区，我们可以看出，IPv6 分给站点的地址非常多。一个站点，有 16bit 的子网，相当于 65535 个子网；每个子网中，还可以用 64 位整数表示设备。</p>
<h4 data-nodeid="183848" class="">本地单播</h4>




<p data-nodeid="175838">理论上，虽然 IPv6 可以将所有的设备都连入一个网络。但在实际场景中，很多公司还是需要一个内部网络的。这种情况在 IPv6 的设计中属于局域网络。</p>
<p data-nodeid="175839">在局域网络中，实现设备到设备的通信，就是本地单播。IPv6 的本地单播地址组成如下图所示：</p>
<p data-nodeid="184472" class=""><img src="https://s0.lgstatic.com/i/image6/M01/3C/03/Cgp9HWCH4x6AJJxNAAEMhuOKNmY768.png" alt="Drawing 7.png" data-nodeid="184475"></p>


<p data-nodeid="175842">这种协议比较简单，本地单播地址必须以<code data-backticks="1" data-nodeid="176029">fe80</code>开头，后面 64 位的 0，然后接上 54 位的设备编号。上图中的 Interface 可以理解成网络接口，其实就是网卡。</p>
<h4 data-nodeid="186935" class="">分组多播</h4>




<p data-nodeid="175844">有时候，我们需要实现广播。所谓广播，就是将消息同时发送给多个接收者。</p>
<p data-nodeid="175845">IPv6 中设计了分组多播，来实现广播的能力。当 IP 地址以 8 个 1 开头，也就是<code data-backticks="1" data-nodeid="176037">ff00</code>开头，后面会跟上一个分组的编号时，就是在进行分组多播。</p>
<p data-nodeid="175846">这个时候，我们需要一个广播设备，在这个设备中已经定义了这些分组编号，并且拥有分组下所有设备的清单，这个广播设备会帮助我们将消息发送给对应分组下的所有设备。</p>
<h4 data-nodeid="189372" class="">任意播（Anycast）</h4>




<p data-nodeid="175848">任意播，本质是将消息发送给多个接收方，并选择一条最优的路径。这样说有点抽象，接下来我具体解释一下。</p>
<p data-nodeid="175849">比如说在一个网络中有多个授时服务，这些授时服务都共享了一个任播地址。当一个客户端想要获取时间，就可以将请求发送到这个任播地址。客户端的请求扩散出去后，可能会找到授时服务中的一个或者多个，但是距离最近的往往会先被发现。这个时候，客户端就使用它第一次收到的授时信息修正自己的时间。</p>
<h3 data-nodeid="189976">IPv6 和 IPv4 的兼容</h3>


<p data-nodeid="175852">目前 IPv6 还没有完全普及，大部分知名的网站都是同时支持 IPv6 和  IPv4。这个时候我们可以分成 2 种情况讨论：</p>
<ol data-nodeid="175853">
<li data-nodeid="175854">
<p data-nodeid="175855">一个 IPv4 的网络和一个 IPv6 的网络通信；</p>
</li>
<li data-nodeid="175856">
<p data-nodeid="175857">一个 IPv6 的网络和一个 IPv6 的网络通信，但是中间需要经过一个 IPv4 的网络。</p>
</li>
</ol>
<p data-nodeid="175858">下面我们具体分析一下。</p>
<p data-nodeid="175859"><strong data-nodeid="176054">情况 1：IPv4 网络和 IPv6 网络通信</strong></p>
<p data-nodeid="175860">例如一个 IPv6 的客户端，想要访问 IPv4 的服务器，步骤如下图所示：</p>
<p data-nodeid="190578" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3C/0C/CioPOWCH4y-AUMRWAAMbF03aDqY454.png" alt="Drawing 9.png" data-nodeid="190581"></p>


<ol data-nodeid="175863">
<li data-nodeid="175864">
<p data-nodeid="175865">客户端通过 DNS64 服务器查询 AAAA 记录。DNS64 是国际互联网工程任务组（IETF）提供的一种解决 IPv4 和 IPv6 兼容问题的 DNS 服务。这个 DNS 查询服务会把 IPv4 地址和 IPv6 地址同时返回。</p>
</li>
<li data-nodeid="175866">
<p data-nodeid="175867">DNS64 服务器返回含 IPv4 地址的 AAAA 记录。</p>
</li>
<li data-nodeid="175868">
<p data-nodeid="175869">客户端将对应的 IPv4 地址请求发送给一个 NAT64 路由器</p>
</li>
<li data-nodeid="175870">
<p data-nodeid="175871">由这个 NAT64 路由器将 IPv6 地址转换为 IPv4 地址，从而访问 IPv4 网络，并收集结果。</p>
</li>
<li data-nodeid="175872">
<p data-nodeid="175873">消息返回到客户端。</p>
</li>
</ol>
<p data-nodeid="175874"><strong data-nodeid="176070">情况 2：两个 IPv6 网络被 IPv4 隔离</strong></p>
<p data-nodeid="175875">这种情况在普及 IPv6 的过程中比较常见，IPv6 的网络一开始是一个个孤岛，IPv6 网络需要通信，就需要一些特别的手段。</p>
<p data-nodeid="175876">不知道你有没有联想到坐火车穿越隧道的感觉，连接两个孤岛 IPv6 网络，其实就是在 IPv4 网络中建立一条隧道。如下图所示：</p>
<p data-nodeid="191174" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/3C/03/Cgp9HWCH4ziAD-hYAAMdJ6IgvWE780.png" alt="Drawing 11.png" data-nodeid="191177"></p>


<p data-nodeid="175879"><strong data-nodeid="176083">隧道的本质就是在两个 IPv6 的网络出口网关处，实现一段地址转换的程序</strong>。</p>
<h3 data-nodeid="175880">总结</h3>
<p data-nodeid="175881">总结下，<strong data-nodeid="176090">IPv6 解决的是地址耗尽的问题</strong>。因为解决了地址耗尽的问题，所以很多其他问题也得到了解决，比如说减少了子网，更小的封包头部体积，最终提升了性能等。</p>
<p data-nodeid="175882">除了本讲介绍的内容，下一讲你还会从局域网络中看到更多对 NAT 技术的解读、对路由器的作用的探讨。随着 IPv6 彻底普及，你可以想象一下，运营商可以给到每个家庭一大批固定的 IP 地址，发布网页似乎可以利用家庭服务器……总之，林䭽也不知道最终会发生什么，我也对未来充满了期待，让我们拭目以待吧。</p>
<p data-nodeid="175883">那么，通过这一讲的学习，你可以尝试回答本讲关联的面试题目：Tunnel 技术是什么了吗？</p>
<p data-nodeid="175884">【<strong data-nodeid="176098">解析</strong>】Tunnel 就是隧道，这和现实中的隧道是很相似的。隧道不是只有一辆车通过，而是每天都有大量的车辆来来往往。两个网络，用隧道连接，位于两个网络中的设备通信，都可以使用这个隧道。隧道是两个网络间用程序定义的一种通道。具体来说，如果两个 IPv6 网络被 IPv4 分隔开，那么两个 IPv6 网络的出口处（和 IPv4 网络的网关处）就可以用程序（或硬件）实现一个隧道，方便两个网络中设备的通信。</p>
<h3 data-nodeid="175885">思考题</h3>
<p data-nodeid="175886"><strong data-nodeid="176104">最后，我再给你出一道需要查资料的思考题：请你总结下 IPv6 和 IPv4 究竟有哪些区别</strong>？</p>
<p data-nodeid="175887">我建议你拿出几分钟的时间，把这两者的区别写在留言区。这个输出的过程不仅能够帮助你产生更多的思考，也是构建知识体系的根基。如果你对本次课程有什么建议和疑问，可以在评论区留言。如果你有所收获，也可以推荐给你的朋友。</p>
<p data-nodeid="175888">这一讲就到这里。发现求知的乐趣，我是林䭽，感谢你学习本次课程。下一讲我们将学习“08 | 局域网：NAT 是如何工作的？”再见！</p>

---

### 精选评论

##### **8490：
> 最大的区最大的区别就是可枚举的个数的区别（地址数），一个32位2^32，一个128位2^128，这样的好处是可以给更多的设备一个唯一的标识码（地址），这样的坏处是更耗资源了，也就是号码本变厚了。

##### **男：
> IPv6与IPv4的区别主要有以下几点：1.IPv6的地址空间更大。IPv4中规定IP地址长度为32,即有2^32-1个地址；而IPv6中IP地址的长度为128,即有2^128-1个地址。夸张点说就是，如果IPV6被广泛应用以后，全世界的每一粒沙子都会有相对应的一个IP地址。2.IPv6的路由表更小。IPv6的地址分配一开始就遵循聚类(Aggregation)的原则,这使得路由器能在路由表中用一条记录(Entry)表示一片子网,大大减小了路由器中路由表的长度,提高了路由器转发数据包的速度。3.IPv6的组播支持以及对流的支持增强。这使得网络上的多媒体应用有了长足发展的机会，为服务质量控制提供了良好的网络平台。4.IPv6加入了对自动配置的支持。这是对DHCP协议的改进和扩展，使得网络(尤其是局域网)的管理更加方便和快捷。5.IPv6具有更高的安全性。在使用IPv6网络中，用户可以对网络层的数据进行加密并对IP报文进行校验，这极大地增强了网络安全

##### **洋：
> 相同点：1.工作原理类似，都是接收上方协议传递来的数据，再切片、封包、调用数据链路层的能力传输数据2. 寻址机制仍保留，还是要对网络进行划分，但是划分方法与IP v4不同。区别：1. 格式不同：由IP v4的4个8位（用 . 符号分割），变成 8 个 16位（用 : 符号 分割）2. 寻址时对网络划分不同：不同的寻址类型有固定的地址前缀。

##### **超：
> 很好

