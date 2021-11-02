<p>本课时我们来学习安全模块：iptables 企业级安全策略。</p>
<h4>什么是 iptables</h4>
<p>说到 iptables，如果你用过 Linux 会比较了解，它是 Linux 上的一套防火墙服务，调用的其实是 Netfilter 内核模块。Netfilter 是 Linux 操作系统核心层内部的一个数据包处理模块，它具有如下功能：</p>
<ol>
<li>网络地址转换；</li>
<li>数据包内容修改；</li>
<li>数据包过滤防火墙功能。</li>
</ol>
<p>在现实中，其实很多应用服务都调用了 Netfilter。这里我给你画了一张图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/E8/CgqCHl7GXLmAQNIJAAAx_JTgoQQ756.png" alt="image (11).png"></p>
<p>可以看到，在最中央就是 Netfilter 的 Linux 内核模块了，我们看到最下层 iptables 会调用 Netfilter 模块来作为安全防火墙。在 CentOS7 以后，出来一款新的防火墙服务叫作 firewalld，其实也是调用了 Netfilter 模块。总体来说，无论是 iptables，还是 firewalld，它们都是调用了 Linux 的 Netfilter 内核模块。</p>
<p>我们再来看图片左边，这里有一个 ipvs 应用工具，它也调用了 Netfilter，但是它的功能和安全防护的防火墙功能有一些区别，它主要用来实现地址转换这样的一个核心功能，所以 ipvs 会为很多负载均衡提供服务，也就是为 LVS 提供服务，如果你了解过 LVS 就会清楚，其实 LVS 就是用到了 ipvs 的命令模块来对数据包规则进行设置。所以 ipvs 的上一层会供 LVS 调用，它在运维管理中可以作为防火墙服务来使用。所以我们会看到 ipvs 是作为负载均衡的服务来使用的，可以总结为负载均衡的服务其实也是调用了 Netfilter 内核模块。</p>
<p>除此之外，我们在这张图的左下角还会看到一个 Kube-proxy 命令模块，它也调用了 ipvs，同时也调用 iptables。其实 Kube-proxy 是 K8s 里的一个组件，负责管理 K8s 里的 service 模块，也就是后端服务，管理后端服务需要对数据包进行转发，并实现负载均衡。我们会看到在 K8s 整套模块的组件里，Kube-proxy 其实也调用了 ipvs，同时基于 ipvs 去调用 Linux 内核的 Netfilter，那么为什么 Kube-proxy 还要去调用 iptables？这是因为早期的 Kube-proxy 版本是基于 iptables 实现的，但是这里有一个性能问题，因为 iptables 毕竟不是专门用作数据包转发的，所以在 service 高于 1000 个的情况下，性能表现上 ipvs 优于 iptables。所以就逐渐使用了 ipvs 服务来代替 iptables 工具。</p>
<p>在这张图里，我们会看到当前整个 Netfilter 在应用平台服务里面，起到的至关性作用。而 iptables 发布的其实是两款专业的防火墙工具，那么对于 iptables 和 firewalld，这两个防火墙服务之间有两个比较大的差异，这里给你来介绍一下。</p>
<h4>iptables 和 firewalld 之间的差异</h4>
<p>iptables 默认是允许访问服务或者通过数据包进行访问的，如果我们不允许某些数据包访问，就可以设置具体的规则去进行阻断。firewalld 则是默认拒绝所有数据包进行访问，只有我们允许一个数据包满足某个规则的时候，才去做对应的设置。所以说它们是相反的两个默认设置的防火墙规则。</p>
<p>另外就是 firewalld 在对数据安全的防护管理上用到了区域的概念，会把xxx分成很多个区域来进行安全管理，而 iptables 则不会分区，而是直接基于面的方式去进行规则上的设置。本课时我们重点介绍公司层面的安全，并且通过 iptables 来介绍企业对防火墙的规则设置。</p>
<h4>iptables 企业安全规则</h4>
<p>在学习本课时之前，我们先要搞懂 iptables 的一些基础知识，iptables 有一个很重要的设置概念就是 4 表 5 链，我们可以理解为是将一些对数据包设置的不同维度共同组合起来，进行数据包安全功能上的一些具体设置。</p>
<p>有如下的 4 张表，包括一个 Filter 表，一个 NAT 表，一个 Mangle 表，一个 Raw 表。它们分别作用于不同的方向，Filter 表用于过滤数据包（默认表），NAT 表用于网络地址转换（IP、端口），Mangle 表用于修改数据包的服务类型、TTL，并且可以配置路由实现 QOS；Raw 表用于决定数据包是否被状态跟踪机制处理。</p>
<p>5 链分别是 INPUT 链，OUTPUT 链，FORWARD 链，PREROUTING 链和 POSTROUTING 链。这 5 条链分别也对应不同的一些设置规则。</p>
<ul>
<li>INPUT 链：进来的数据包应用此规则链中的规则；</li>
<li>OUTPUT 链：外出的数据包应用此规则链中的规则；</li>
<li>FORWARD 链：转发数据包时应用此规则链中的规则；</li>
<li>PREROUTING 链：对数据包作路由选择前应用此链中的规则；</li>
<li>POSTROUTING 链：对数据包作路由选择后应用此链中的规则。</li>
</ul>
<p>我们可以认为有了表和链共同的组合，才完成了完整的数据包规则设置。通常一个数据包也需要经过对应表中不同链结合起来的规则，从而完成整个流程的处理。</p>
<p>另外需要特别提及的一个地方是自定义链，它跟我们刚刚讲到的 4 表 5 链相比有一个较特殊的地方，它是为用户方便管理而使用的，它并不会被直接使用，而是需要通过被默认的链引用才能使用。我们可以理解为它是用户指定给设置的一个规则链的名称，用于方便进行整体规则上的设置。接下来本课时将为你演示安全防控规则的设置，这通常是通过 iptables 的 Filter 表结合 INPUT 链规则进行设置，我们来看一看通过这样的一种设置方式可以实现哪些 4 层安全上面的功能设置。</p>
<h4>iptables 命令规则设置的常见选项</h4>
<p>那么说到了 iptables 的规则设置，你可能需要了解，通过 iptables 命令进行安全防护规则设置都有哪些选项，下面逐一为你列举了几个具体的一些常见的选项：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/DC/Ciqc1F7GXNaAWc-BAACVfKp2FC8903.png" alt="image (12).png"></p>
<ul>
<li>-A 指向指定链添加一个或多个新规则；</li>
<li>-D 表示从指定链中删除一个或多个规则；</li>
<li>-N 表示创建一个新的用户定义链；</li>
<li>-F 通常是在表初始化时，指定规则链中或所有规则链中的策略；</li>
<li>-X 则是删除指定的用户定义链（需先清空链路中的策略）；</li>
<li>-p 指定具体一些协议，比如 TCP、UDP、ICMP 等常见的一些协议数据包；</li>
<li>-s 用来指定 IP 地址 [/ mask] 源地址；</li>
<li>-d 用来指定 IP 地址 [/ mask] 目标地址；</li>
<li>-i 指定设备数据包进入的接口，如果是多网卡的话，可能需要指定具体的接口。</li>
</ul>
<p>这些对应的选项是你需要提前知道的，如果你不了解的话，建议先看一些 iptables 的基础使用规则，然后再学习后面的部分，否则可能会遇到一些小的问题。</p>
<p>接下来，我们用一些场景来给你讲一讲，在 iptables 里面一些常见的设置安全规则的方式。</p>
<p>第一个模块就是对 iptables 进行初始化，所谓 iptables 初始化就是需要将 iptables 的规则清空，同时又要设置一个默认规则。通常我们需要把规则中的策略进行清空。</p>
<ul>
<li>-X 表示是指定用户自定义链来进行删除；</li>
<li>-Z 则是将指定链路或者策略中的计数归零。</li>
</ul>
<p>这样的话就完成了整体规则的清空。另外，iptables -P 设置了 3 条具体规则的链，一个是 INPUT 链，一个是 OUTPUT 链，一个是 FORWARD 链。那么这 3 条链分别设置 Accept，也就是允许所有数据包进行访问，这就完成了整体的初始化。我们看到初始化里没有做任何安全防护规则，而是把以往的规则进行清空，并且设置一条权限比较大的规则，在这个基础上再进行具体的安全防护规则设置。</p>
<p>首先第一个安全防护规则就是通过 iptables 命令来进行设置的，每次设置完具体的命令后，就会把这些命令转化为 iptables 所能识别的一条规则生成到配置文件里。通常对于用户来说对配置文件的管理可能是非常容易理解的，所以为了方便进行 iptables 设置，我们更愿意把 iptables 命令的使用和选项的使用，封装成一些 Shell 脚本来直接批量化设置。这里会看到我使用的 iptables 命令进行设置的同时，也结合了一些 Shell 脚本的语法格式。我们来看一下：</p>
<pre><code data-language="SQL" class="lang-SQL">iptables -A INPUT -i lo -j ACCEPT&nbsp; //允许本地访问
LOCAL_NET="xxx.xxx.xxx.xxx/xx”&nbsp; //允许内部指定网段访问
if [ "$LOCAL_NET" ]
then
&nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $LOCAL_NET -j ACCEPT
fi
ALLOW_HOSTS=(&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;//允许访问主机Ip开白
&nbsp; &nbsp; &nbsp; &nbsp;"xxx.xxx.xxx.xxx"
&nbsp; &nbsp; &nbsp; &nbsp;"xxx.xxx.xxx.xxx"
)

if [ "${ALLOW_HOSTS}" ]
then
&nbsp; &nbsp; &nbsp; &nbsp; for allow_host in ${ALLOW_HOSTS[@]}
&nbsp; &nbsp; &nbsp; &nbsp; do
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $allow_host -j ACCEPT
&nbsp; &nbsp; &nbsp; &nbsp; done
fi
</code></pre>
<p>首先在前面我们会看到有一个 -A，表示对 INPUT 这一条链以及 -i（本地回放网卡）允许访问，也就是对允许访问本机规则默认开放。在下面有一个 LOCAL_NET，这里其实是定义了一个 Shell 变量，在这个变量里我们可以进行一些自定义配置，比如本地的网络段属于哪一个网站。如果允许本地网络段进行访问，就需要定义一整串网段信息，并且把网段信息复制给 LOCAL_NET，然后判断这个变量是否存在。如果存在的话，则允许本地的网络访问 iptables -A INPUT，-p tcp 表示 TCP 协议， -s 表示原地址，也就是允许所有本地网络原地址来访问这台主机。<br>
下面同样也是白名单设置 Shell 的实施方式：</p>
<pre><code data-language="plain" class="lang-plain">DENY_HOSTS=(&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; //直接丢弃列表
&nbsp; &nbsp; &nbsp; &nbsp;"xxx.xxx.xxx.xxx"
&nbsp; &nbsp; &nbsp; &nbsp;"xxx.xxx.xxx.xxx"
)
if [ "${DENY_HOSTS}" ]
then
&nbsp;   &nbsp; &nbsp; for deny_host in ${DENY_HOSTS[@]}
&nbsp; &nbsp; &nbsp; &nbsp; do
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -s $deny_host -m limit --limit 1/s -j LOG --log-prefix "deny_host: "
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -s $deny_host -j DROP
&nbsp; &nbsp; &nbsp; &nbsp; done
fi
</code></pre>
<p>这里首先定义了一个元组变量，里面包含了多个 IP。然后进行判断，同样对某一些 IP 进行开白，我们会看到这一块其实同样设置好白名单，允许哪一些 IP 来进行访问，这里其实设置的规则更多的是白名单类型的，而在下面就会有设置需要具体拒绝的名单规则，同样是按照 Shell 的方式来进行设置的。</p>
<p>DENY_HOSTS 是另外一个元组变量了，这里主要是用于设置不允许一些数据包 IP 进行访问，并且对其进行限频。我们看到在 do 这个条件判断里面，如果是属于 deny_host，会把 -s 原地址做一个调用，-m 代表进行状态上的一些设置，limit 表示做速度上的限制。limit 1/s 则表示只允许每秒进行一次访问，这里做了一个限频，如果是每秒一次的速度，则会做 -j LOG，这样的执行动作就表示，如果超过了一秒，就会进行日志打印，同时这里也会做一个完全的访问控制。那么 iptables 通过原地址符合条件的包来访问主机，那么这里会再加一个规则，就是直接 drop，表示既做日志记录，也把这个数据包进行丢弃。</p>
<p>我们看到，这一块主要做的是黑名单的安全规则，那么下面加了一条额外的规则：</p>
<pre><code data-language="java" class="lang-java">iptables -A INPUT&nbsp; -p tcp -m state --state ESTABLISHED,RELATED -j ACCEPT&nbsp; &nbsp; &nbsp;<span class="hljs-comment">//建立会话后允许数据包通信</span>
</code></pre>
<p>这里有一个 -m state，它是 iptables 里通常需要加入进去的，那么它的特殊的情况是什么呢？它表示建立会话的包是允许进行通信的，比如设置 iptables 是规则主机，那么访问机制是这台主机主动去访问这些黑名单主机里面的服务，由于是主动发出的，如果建立起了会话，服务端这些黑名单的主机及时回包到我的主机上，也是允许进行访问的。</p>
<p>这和黑名单的差异在于没有建立会话，这些黑名单的主机则主动对我的主机进行建联，这样是不允许的。所以同时加上这两点的话会更加符合一些实际的 iptables 规则的设置需求。这样如果主机想要主动去访问任何一个地址，都可以通过这样的规则进行优先匹配了。</p>
<p>其实上面这些设置更多是基于源地址、目的地址、IP 地址这样的访问控制进行设置。那么下面我们会来看一看，对于一些常见的 4 层的攻击，比如说 SYN 的攻击，iptables 里有哪一些设置可以帮助到我们？</p>
<pre><code data-language="java" class="lang-java">iptables -N SYN_FLOOD
iptables -A SYN_FLOOD -p tcp --syn \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-m hashlimit \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit <span class="hljs-number">200</span>/s \&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">//每秒最多200个连接</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-burst <span class="hljs-number">3</span> \&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//如果连接数连续三次超过上述限制，则将有一个限制。</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-htable-expire <span class="hljs-number">300000</span> \&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//管理表中记录的有效期限（单位：毫秒）</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-mode srcip \&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">//通过源地址管理请求数</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-name t_SYN_FLOOD \&nbsp; &nbsp; <span class="hljs-comment">//哈希表名存储在/proc/net /ipt_hashlimit</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-j RETURN
iptables -A SYN_FLOOD -j LOG --log-prefix <span class="hljs-string">"syn_flood_attack: "</span>
iptables -A SYN_FLOOD -j DROP
iptables -A INPUT -p tcp --syn -j SYN_FLOOD
</code></pre>
<p>这里， iptables 有一段规则，加了 -N SYN_FLOOD。 -N 表示建立了一条自定义的规则链，我们来看一下这条自定义的规则里添加了哪些具体策略。首先添加了 tcp --syn 协议的数据包，hashlimit 也就是限频，每秒最多 200 次访问，同时如果超过了连接 3 次的设置，则会发起一个限制，这个限制就是返回告诉它不允许直接把包还给客户端。</p>
<p>另外这里对 hashlimit 这个表的数据做了一个过期时间设置，也就是说，因为 hashlimit 表是需要记录这样的一个状态的，所以需要设置它的有效期。</p>
<p>--hashlimit-mode srcip \ 表示通过源地址管理请求书 。也就是说基于客户的源地址来进行具体唯一性的标识来进行判断。那么这一串命令主要来进行频次上具体协议包的限制，也就是不允许过多的 tcp --syn 单个地址发起过多的请求，从而导致发起过多的连接。</p>
<p>我们刚刚讲到的是设置一条自定义链，这种自定义链都需要通过默认链来进行调用。我们看到在默认链里面则是使用 INPUT 链来进行具体调用， -j 后面加了默认的自定义链，这样就完成了整个的自链的设置。</p>
<p>我们刚刚讲到的其实是对包的频次进行设置，那么对于其他协议的包其实也可以用同样的一些办法，我们可以看一下这里的 ICMP设置：</p>
<pre><code data-language="java" class="lang-java">iptables -N PING_OF_DEATH&nbsp; <span class="hljs-comment">//丢弃ICMP超出限制</span>
iptables -A PING_OF_DEATH -p icmp --icmp-type echo-request \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-m hashlimit \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit <span class="hljs-number">1</span>/s \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-burst <span class="hljs-number">10</span> \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-htable-expire <span class="hljs-number">300000</span> \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-mode srcip \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-name t_PING_OF_DEATH \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-j RETURN
iptables -A PING_OF_DEATH -j LOG --log-prefix <span class="hljs-string">"ping_of_death_attack: "</span>
iptables -A PING_OF_DEATH -j DROP
iptables -A INPUT -p icmp --icmp-type echo-request -j PING_OF_DEATH
</code></pre>
<p>如果我们是想对 ICMP 的连接进行具体的规则设置的话，同样也需要新建一条自定义链，然后在里面设置它的频次，并且进行具体策略设置，对协议进行默认链 INPUT 链调用自定义链，这样就实现了对于 ICMP 超出规则的设置，我们会看到对这个设置每秒只能允许一次访问，那么如果超过了 10 次的话，则会开启频次限制。</p>
<p>我们刚刚讲到了对于 ICMP 规则的设置，接下来也可以对 HTTP 的位置来进行设置，这里有一个疑问了，可能你会认为 HTTP 是基层的应用层协议安全，既然我们讲的是 4 层安全，怎么会有对 HTTP 进行访问的限制？其实对于 HTTP 协议现在也是基于4 层的策略来进行限制的，这里其实也是基于 TCP 的协议来进行设置。</p>
<pre><code data-language="java" class="lang-java">iptables -N HTTP_DOS
iptables -A HTTP_DOS -p tcp -m multiport --dports $HTTP \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-m hashlimit \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit <span class="hljs-number">1</span>/s \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-burst <span class="hljs-number">100</span> \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-htable-expire <span class="hljs-number">300000</span> \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-mode srcip \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;--hashlimit-name t_HTTP_DOS \ <span class="hljs-comment">//哈希表存储在/ proc / net / ipt_hashlimit</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;-j RETURN
iptables -A HTTP_DOS -j LOG --log-prefix <span class="hljs-string">"http_dos_attack: "</span>
iptables -A HTTP_DOS -j DROP
iptables -A INPUT -p tcp -m multiport --dports $HTTP -j HTTP_DOS
</code></pre>
<p>因为我们这里是 -p tcp，只不过端口 -dports 是基于 HTTP 端口，也就是说其实是基于 4 层来限制的 7层，并不是基于 7 层会话的 cookie session 去进行设置的。所以这里也是进行的一个 TCP 连接，只不过是指定了访问具体的某一个服务端口，从而进行频次限制。</p>
<p>所以基于底层的连接式的频次限制，同样有助于我们应用层的安全，基于这样的一个理念，对于 SSH 的安全设置，我们同样也可以通过这样的方式来设置。SSH 最大的问题就是会存在暴力破解，所以我们就需要设置在指定时间内允许多少次访问，否则就需要做具体的安全限制了。</p>
<pre><code data-language="java" class="lang-java">SSH=<span class="hljs-number">22</span>
iptables -A INPUT -p tcp --syn -m multiport --dports $SSH -m recent --name ssh_attack –set
iptables -A INPUT -p tcp --syn -m multiport --dports $SSH -m recent --name ssh_attack --rcheck --seconds <span class="hljs-number">60</span> --hitcount <span class="hljs-number">5</span> -j LOG --log-prefix <span class="hljs-string">"ssh_brute_force: "</span>
iptables -A INPUT -p tcp --syn -m multiport --dports $SSH -m recent --name ssh_attack --rcheck --seconds <span class="hljs-number">60</span> --hitcount <span class="hljs-number">5</span> -j REJECT --reject-with tcp-reset
</code></pre>
<p>这里会看到同样是基于 TCP 的协议请求，访问 SSH 端口的时候，会判断最近 60 秒之内访问了多少次数，如果是超过了次数限制的话，那么就会进行 REJECT，也就是拒绝 IP 来进行访问了。所以 SSH 还有 FTP 都可以通过这样的一种方式来防止暴力破解的攻击。</p>
<p>接下来我们需要在机器上面设置，哪一些服务是需要正常对外提供服务的，这里我们可以来具体设置一些需要对外提供的服务，比如需要允许某一些网段访问指定的一些服务，或是访问 SSH 服务等。如果是有一些 HTTP 服务，需要公共对外服务的话，同样可以把网段的源地址限制去掉，而把公共服务对外暴露，那也就是把服务设置成对外能够允许提供正常服务，不因为 iptables 规则而阻止，所以这里我们主要是围绕服务来进行具体设置的。</p>
<pre><code data-language="java" class="lang-java">iptables -A INPUT -p tcp -m multiport --dports $HTTP -j ACCEPT
……
LIMITED_LOCAL_NET="xxx.xxx.xxx.xxx/xx"
if [ "$LIMITED_LOCAL_NET" ]
then
&nbsp; &nbsp; &nbsp; &nbsp; # SSH
&nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $LIMITED_LOCAL_NET -m multiport --dports $SSH -j ACCEPT # LIMITED_LOCAL_NET -&gt; SELF
&nbsp; &nbsp; &nbsp; &nbsp; # FTP
&nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $LIMITED_LOCAL_NET -m multiport --dports $FTP -j ACCEPT # LIMITED_LOCAL_NET -&gt; SELF
&nbsp; &nbsp; &nbsp; &nbsp; # MySQL
&nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $LIMITED_LOCAL_NET -m multiport --dports $MYSQL -j ACCEPT # LIMITED_LOCAL_NET -&gt; SELF
fi
ZABBIX_IP="xxx.xxx.xxx.xxx"
if [ "$ZABBIX_IP" ]
then
&nbsp; &nbsp; &nbsp; &nbsp; iptables -A INPUT -p tcp -s $ZABBIX_IP --dport 10050 -j ACCEPT&nbsp;
fi
</code></pre>
<p>我们刚刚讲到的 IP 规则设置了基于底层协议的限制规则，还帮助应用层来做安全频率上的一些规则限制，刚刚我们也讲到了，如何允许正常服务对外提供访问设置，最后我们只要进行一条规则来整体收尾，也就是要设置除了这些规则以外的其他的所有服务，默认不允许访问，因为我们知道 iptables 默认不做设置的话，它是允许对外提供服务的。</p>
<p>假设有 300 的端口，我们没有允许这些文件共享的服务对外提供服务，它默认也是可以对外来提供的，因为 iptables 默认是允许对外提供访问的。所以在最后我们需要设计一条收尾规则，就是除了我们刚刚设置的这些规则以外，其他的访问的数据包一概是 drop 掉的，不允许访问的。所以我们就加一条规则：</p>
<pre><code data-language="java" class="lang-java">ptables -A INPUT -p tcp -m multiport --dports $HTTP -j ACCEPT 
</code></pre>
<p>这样就完成了一个完整的规则设置了。本课时我们讲解的 iptables 的规则设置，还有他的一些整个使用关系，以及具体的安全设置上的一些设置样例，你可以再多多理解，下个课时我将讲解“应用安全：基于 HTTP、HTTPS 请求过程中常见 waf 攻防策略”，记得按时听课。</p>

---

### 精选评论


