<p data-nodeid="64549" class="">欢迎进入高并发架构设计模块。这一讲，我将为你介绍流量负载均衡器中 LVS 和 Nginx 原理和使用方法。</p>
<p data-nodeid="64550">首先，请思考个问题：当大量用户打开秒杀页面，向秒杀服务发起请求的时候，这些请求是如何分布在后端多台机器上的呢？</p>
<p data-nodeid="64551">这是个典型的高并发下流量负载均衡的问题。所谓流量负载均衡，是指让流量比较均衡地到达后端各服务器，确保各服务器负载相对均衡，不会导致某一台服务器负载太高而被压垮。</p>
<p data-nodeid="64552">负载均衡有特定的算法，常用的有轮询法、随机法、源地址哈希法、加权轮询法、加权随机法、最小连接数法等。通常，我们会用一些组件来提供负载均衡的能力，比如 LVS 和 Nginx。那么，它们都是如何工作的呢？接下来我给你重点介绍下。</p>
<h3 data-nodeid="64553">负载均衡是如何实现的？</h3>
<p data-nodeid="64554">首先，我们来了解下 LVS 和 Nginx 常用的负载均衡算法的实现原理。</p>
<h4 data-nodeid="64555">轮询法和加权轮询法</h4>
<p data-nodeid="64556">什么是轮询法呢？<strong data-nodeid="64623">轮询法就是假设后端服务器性能都一样，以依次循环的方式将请求调度到不同的服务器上。</strong> 它的特点是实现简单，缺点是如果后端服务器性能不一样，性能好的服务器利用率较低，无法发挥最大性能。</p>
<p data-nodeid="64557">为了解决服务器性能不一致出现负载不均衡的问题，加权轮询法出现了。它的思想是按服务器性能分配权重，性能好的权重大，性能差的权重小。</p>
<p data-nodeid="64558">比如，有三台服务器 A、B、C，处理能力分别是 2500 QPS、5000 QPS、2500 QPS。假如总共有 3 万个请求，采用轮询法的时候，顺序是 ABCABC，各分得 1 万个请求；如果采用加权轮询法，权重就可能分别是总流量的 25%、50%、25%，顺序变成 ABCBABCB，处理的请求数分别是 7500、15000、7500。</p>
<h4 data-nodeid="64559">随机法和加权随机法</h4>
<p data-nodeid="64560">所谓随机法，是指在转发请求的时候不是按顺序挑选后端服务器，而是随机挑选，它的优缺点跟轮询法一样。</p>
<p data-nodeid="64561">假如有三台服务器A、B、C，随机法顺序可能是 CABACB。采用加权随机法，顺序可能就会变成 CBABABCB。</p>
<p data-nodeid="64562">与轮询法有些区别的是，随机法不需要用变量来记录当前轮询到的节点。而轮询法则需要该变量，以便查找到下一次轮询的节点，比如当前轮询到了 A，则记下来，下一次轮询到 B。</p>
<h4 data-nodeid="64563">最小连接数法和最低延迟法</h4>
<p data-nodeid="66448"><strong data-nodeid="66453">如果说随机法和轮询法都是按照服务器理论容量来转发流量的，那么，最小连接数法和最低延迟法则可以基于服务器实际压力来均衡负载。</strong></p>
<p data-nodeid="66777"><img src="https://s0.lgstatic.com/i/image2/M01/03/FC/Cip5yF_lz26AKZULAAVg-m1TrWk153.png" alt="Lark20201225-193856.png" data-nodeid="66781"></p>
<p data-nodeid="66778">所谓最小连接数法，是指将新请求转发给所有服务器中连接数最小的服务器。因为连接数小通常说明该服务器正在处理的请求最少，也就意味着压力最小。最小连接数法通常用在使用了 WebSocket、HTTP2.0、TCP 等长连接的业务场景。</p>





<p data-nodeid="64566">同样的道理，最低延迟法是将最新的请求转发给延迟最低的服务器。因为延迟最低说明该服务器性能最好，或者负载最低。</p>
<p data-nodeid="64567">最小连接数法和最低延迟法的优点是能达到实际上的负载均衡，缺点是需要维护各节点负载状态。</p>
<h4 data-nodeid="64568">源地址哈希法</h4>
<p data-nodeid="64569">有时候，我们需要将来自同一个地区的用户的请求转发到特定的服务器上，而我们前面提到的那些算法并不能实现这个功能，怎么办呢？这就需要用到源地址哈希法了。</p>
<p data-nodeid="64570">源地址哈希法的主要原理是：根据请求来源 IP ，按照某个规则将请求转发到特定的服务器上。换句话说，它通常是对来源 IP ，采用哈希算法来选取服务器，比如将 12.12.12.12 和 12.12.12.13 这两个 IP 分别哈希到服务器 A 和 B 上。</p>
<p data-nodeid="64571">它的好处是可以通过人为干预的方式控制流量，通常用于将特定流量转发到特定机器上实现灰度发布。缺点是同一个用户的请求始终在固定的服务器上，假如该服务器出现故障了，哈希到该服务器的所有用户的请求都会失败。</p>
<p data-nodeid="64572">以上便是常用的负载均衡算法。那么，LVS 和 Nginx 里是如何利用这些负载均衡算法来工作的呢？</p>
<h3 data-nodeid="64573">LVS 是如何工作的？</h3>
<p data-nodeid="64574">LVS 是 Linux Virtual Server 的简称，也就是 Linux 虚拟服务器，它是 Linux 内核的一部分。</p>
<p data-nodeid="64575">熟悉网络模型的应该知道，网络模型分为 OSI 七层模型和 TCP/IP 四层模型。<strong data-nodeid="64649">LVS 主要是工作在七层网络模型中的第四层，也就是传输层，能负责转发 TCP、UDP 协议。</strong></p>
<p data-nodeid="64576">在网络通信里，网络设备可以利用报文中的目标 IP ，通过 ARP（Address Resolution Protocol，地址解析协议）来搜寻下一跳设备的 MAC 地址，而数据接收方利用来源 IP 识别网络会话，并在应答时将收到的来源 IP 填写到目标 IP 中。四层负载均衡器正是基于这个原理，通过修改来源 IP、目标 IP、MAC 地址等方式将流量转发给数据接收方的。</p>
<p data-nodeid="64577">LVS 有三种工作模式：<strong data-nodeid="64656">NAT（Network Address Translation，网络地址转换）、DR（Direct Route，直接路由）、TUN（Tunnel，隧道）</strong>。其中 NAT 模式和 DR 模式要求 LVS 与 RS（Real Server，真实服务器）在同一个网段，使用同一个 VIP（虚拟 IP），并将 VIP 暴露到网络中。</p>
<p data-nodeid="64578">NAT 模式下，所有入网和出网的数据包都会经过 LVS 节点。对于客户端发起的入网数据包，目标 IP 是服务端的 VIP。LVS 收到入网数据包后，会将目标 IP 修改为某台 RS 的 IP ，并将数据包投递给该 RS。对于出网数据包，来源 IP 是 RS 的 IP，LVS 会将来源 IP 修改为 VIP，以便客户端关联上对应的网络会话，并正确处理数据。</p>
<p data-nodeid="64579">DR 模式下，目标服务器与 LVS 共用 VIP 。客户端发起的网络报文会经过 LVS，LVS 只会将网络报文中 MAC 地址修改为目标服务器的 MAC 地址，并转发给目标服务器。</p>
<p data-nodeid="64580">由于源 IP 和 目标 IP 都未修改，且 MAC 地址是目标服务器的 MAC 地址，目标服务器会正常处理接收到的数据。在目标服务器处理完请求后，回传的报文目标 IP 不是 LVS 的 IP，而是客户端的 IP。而且，来源 MAC 地址不是 LVS 的 MAC 地址，回传的网络报文也就不会经过 LVS。因为 DR 模式下 LVS 不需要处理 RS 回传的报文，所以性能非常高，也是最常用的模式。</p>
<p data-nodeid="64581">TUN 模式下，不要求 LVS 与 RS 在同一网段，但需要 RS 支持 TUN。当 LVS 收到入网数据包时，它会在数据包的基础上再封装一层 IP 协议，协议中的目标 IP 地址是某台 RS 的 IP 地址，以便 RS 收到并正常处理。RS 收到数据包后，先拆开 LVS 封装的 IP 包，然后再拆开客户端的 IP 包并处理数据。处理完数据后，RS 再按照 DR 模式下那样正常返回报文，该报文不再经过 LVS 处理，而是直接投递给客户端。</p>
<p data-nodeid="64582">在以上三种模式下，都涉及如何选取 RS 的问题。实际上，LVS 提供了数十种负载均衡策略，以便应对不同业务场景。</p>
<p data-nodeid="64583">在秒杀系统中，LVS 是如何发挥作用的呢？</p>
<p data-nodeid="64584">秒杀接口服务并发能力达到千万级别，节点数超过 50 台，必然需要负载均衡器来确保每个节点负载均衡，而且是多个负载均衡器来做负载均衡。由于 LVS 性能非常高，它可以作为云架构中的 ELB（Elastic Load Balancing，弹性负载均衡器）为秒杀系统提供服务，也可以利用它的多种负载均衡模式为秒杀服务均衡流量。并且，LVS 工作在网络四层，它对秒杀请求延迟影响很小。</p>
<p data-nodeid="64585">当然，一台 LVS 是无法扛住千万并发的，通常需要 10 台左右，其上再用 DDNS 来做域名解析负载均衡。由于秒杀流量大，对机器网卡性能也有较高要求，而 LVS 在搭配高性能网卡时，能够提供单机百万以上并发能力。</p>
<p data-nodeid="64586">需要注意的是，虽然 LVS 能做网络四层协议转发，但它无法按 HTTP 协议中的请求路径做负载均衡，所以还需要 Nginx 。</p>
<h3 data-nodeid="64587">Nginx 是如何工作的？</h3>
<p data-nodeid="64588">与 LVS 不同的是，Nnginx 虽然也能工作在网络四层，但其无法修改 IP 地址和 MAC 地址，只能做简单的数据转发。但 Nginx 网络七层协议处理能力非常强，比如处理 HTTP 、HTTPS、HTTP2 等协议。所以，Nginx 常用于做网络七层协议的负载均衡。</p>
<p data-nodeid="64589">与 LVS 工作在 Linux 内核空间不同，Nginx 工作在用户空间，这也意味着它的性能比 LVS 要稍微差一些。Nginx 在用户空间利用多进程和 Epoll 模型，只要机器性能足够，单机并发能力甚至能达到惊人的百万级别。</p>
<p data-nodeid="64590">Nginx 是如何配置负载均衡的呢？它主要有四种负载均衡模式：轮询法、加权轮询法、来源 IP 哈希法、最小连接数法。接下来我们看下各种负载均衡是如何配置的。</p>
<p data-nodeid="64591">Nginx 默认提供轮询模式负载均衡，配置示例如下：</p>
<pre class="lang-java" data-nodeid="64592"><code data-language="java">upstream seckill-admin {
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8081</span>;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8082</span>;
}
</code></pre>
<p data-nodeid="64593">当在每个 server 后面加上 weight 参数后，轮询模式变成加权轮询模式：</p>
<pre class="lang-java" data-nodeid="64594"><code data-language="java">upstream seckill-admin {
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8081</span> weight=<span class="hljs-number">4</span>;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8082</span> weight=<span class="hljs-number">6</span>;
}
</code></pre>
<p data-nodeid="64595">当在轮询模式的配置中加上 ip_hash，Nginx 将会按来源 IP 哈希做负载均衡。配置如下：</p>
<pre class="lang-java" data-nodeid="64596"><code data-language="java">upstream seckill-admin {
  ip_hash;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8081</span>;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8082</span>;
}
</code></pre>
<p data-nodeid="64597">同样，如果在配置中加上 least_conn，Nginx 会采用最小连接数法做负载均衡。配置如下：</p>
<pre class="lang-java" data-nodeid="64598"><code data-language="java">upstream seckill-admin {
  least_conn;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8081</span>;
  server <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8082</span>;
}
</code></pre>
<p data-nodeid="64599">在秒杀系统中，秒杀接口服务要求高并发、高性能，它已经有一层 ELB 为其做负载均衡了，如果再加一层 Nginx，会带来两方面的问题：首先，多一层处理，意味着请求延迟的增加，性能下降；其次，千万级别的高并发下，需要至少 10 台 Nginx，也就会增加不少成本。</p>
<p data-nodeid="64600">那我们如何在秒杀系统中用 Nginx 呢？秒杀管理后台功能较多，需要提供不少 HTTP 接口，其中还包括文件上传接口。同时，也需要提供多个节点确保其可用性。那么，我们可以使用 Nginx 来作为它的负载均衡器，采用轮询法即可满足要求。</p>
<h3 data-nodeid="64601">小结</h3>
<p data-nodeid="64602">以上就是 LVS 和 Nginx 的多种负载均衡算法和使用方法，你是否理解了负载均衡器的工作原理呢？是否明白了在哪种业务场景下使用哪种负载均衡？</p>
<p data-nodeid="65648">总的来说，在长连接场景下，我们应该使用最小连接数来控制连接的负载均衡。而在 HTTP1.x 等短连接场景下，我们需要使用轮询或者加权轮询来做负载均衡。</p>
<p data-nodeid="65972"><img src="https://s0.lgstatic.com/i/image/M00/8C/24/CgqCHl_lz3uALCBGAAVdwJqcbEY525.png" alt="Lark20201225-193851.png" data-nodeid="65975"></p>








<p data-nodeid="64856">接下来，请你做个思考题：在粘性会话场景中，或者希望提升内存缓存命中率的情况下，你应该使用哪种负载均衡模式呢？</p>




<p data-nodeid="64605">你可以将答案写在留言区。期待你的回答哦！</p>
<p data-nodeid="64606">这一讲就到这里了，下一讲我将给你介绍“连接池和协程池为何能提升并发能力”。到时见！</p>
<hr data-nodeid="64607">
<p data-nodeid="64608"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="64690"><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgX2AHJo_ACRP1TPc6yM423.png" alt="Drawing 15.png" data-nodeid="64689"></a></p>
<p data-nodeid="64609"><strong data-nodeid="64694">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="64610" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="64698">点击链接，提升自己</a>！</p>

---

### 精选评论

##### *坤：
> nginx配置固定访问配置sticky-module模块

##### *铁：
> 提高命中率应该是ip hash吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 客户端IP，可能会变。对于HTTP请求，通常是启用粘性连接。

##### **生：
> lvs和nginx的网络模型的部分一直没搞懂过，看来需要好好补一下基础。

##### **4062：
> iphash的话，如果一台机器挂掉，如何自动将这台机器剔除出hash呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 加上健康检查，如果机器挂掉，nginx的健康检查探测到服务异常，会自动将其摘除掉，直到健康检查恢复。

##### **星：
> 提高缓存命中率要保证同一个用户的请求落到一台机器上，用ip_hash?求指教

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以在代理层启用粘性会话，浏览器第一次访问接口时，代理层会给前端生成一个哈希值，后续浏览器每次请求后端接口都会带上，代理层会利用生成的哈希值将请求路由到固定的节点上。云厂商的一些负载均衡器和网关支持粘性会话，比如AWS 的 ALB。nginx 里启用 ip_hash 也有效果，但用户的 IP 可能会变。

##### XX：
> 思考题：选择source_addr负载均衡算法

