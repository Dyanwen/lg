<p data-nodeid="5483" class=""><strong data-nodeid="5603">我看到过一道关于 Linux 指令的面试题：如何查看一个域名有哪些 NS 记录</strong>？</p>
<p data-nodeid="5484">这类题目是根据一个场景，考察一件具体的事情如何处理。虽然你可以通过查资料找到解决方案，但是，这类问题在面试中还是有必要穿插一下，用于确定求职者技能是否熟练、经验是否丰富。特别是计算机网络相关的指令，平时在远程操作、开发、联调、Debug 线上问题的时候，会经常用到。</p>
<p data-nodeid="5485">Linux 中提供了不少网络相关的指令，因为网络指令比较分散，本课时会从下面几个维度给你介绍，帮助你梳理常用的网络指令：</p>
<ul data-nodeid="5486">
<li data-nodeid="5487">
<p data-nodeid="5488">远程操作；</p>
</li>
<li data-nodeid="5489">
<p data-nodeid="5490">查看本地网络状态；</p>
</li>
<li data-nodeid="5491">
<p data-nodeid="5492">网络测试；</p>
</li>
<li data-nodeid="5493">
<p data-nodeid="5494">DNS 查询；</p>
</li>
<li data-nodeid="5495">
<p data-nodeid="5496">HTTP。</p>
</li>
</ul>
<p data-nodeid="5497">这块知识从体系上属于 Linux 指令，同时也关联了很多计算机网络的知识，比如说 TCP/IP 协议、UDP 协议，我会在“<strong data-nodeid="5616">模块七</strong>”为你简要介绍。</p>
<p data-nodeid="5498">如果你对这部分指令背后的网络原理有什么困惑，可以在评论区提问。另外，你也可以关注我将在拉勾教育推出的《<strong data-nodeid="5622">计算机网络</strong>》课程。下面我们开始学习今天的内容。</p>
<h3 data-nodeid="5499">远程操作指令</h3>
<p data-nodeid="7673" class="">远程操作指令用得最多的是<code data-backticks="1" data-nodeid="7675">ssh</code>，<code data-backticks="1" data-nodeid="7677">ssh</code>指令允许远程登录到目标计算机并进行远程操作和管理。还有一个比较常用的远程指令是<code data-backticks="1" data-nodeid="7679">scp</code>，<code data-backticks="1" data-nodeid="7681">scp</code>帮助我们远程传送文件。</p>



<h4 data-nodeid="5501">ssh（Secure Shell）</h4>
<p data-nodeid="5502">有一种场景需要远程登录一个 Linux 系统，这时我们会用到<code data-backticks="1" data-nodeid="5635">ssh</code>指令。比如你想远程登录一台机器，可以使用<code data-backticks="1" data-nodeid="5637">ssh user@ip</code>的方式，如下图所示：</p>
<p data-nodeid="5503"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92j8GAMNHAAAPCrIyhHHk744.png" alt="Drawing 0.png" data-nodeid="5641"></p>
<p data-nodeid="5504">上图中，我在使用<code data-backticks="1" data-nodeid="5643">ssh</code>指令从机器<code data-backticks="1" data-nodeid="5645">u1</code>登录我的另一台虚拟机<code data-backticks="1" data-nodeid="5647">u2</code>。这里<code data-backticks="1" data-nodeid="5649">u1</code>和<code data-backticks="1" data-nodeid="5651">u2</code>对应着 IP 地址，是我在<code data-backticks="1" data-nodeid="5653">/etc/hosts</code>中设置的，如下图所示：</p>
<p data-nodeid="5505"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92j8mAIMPdAACTOATTrQM694.png" alt="Drawing 1.png" data-nodeid="5657"></p>
<p data-nodeid="5506"><code data-backticks="1" data-nodeid="5658">/etc/hosts</code>这个文件可以设置 IP 地址对应的域名。我这里是一个小集群，总共有两台机器，因此我设置了方便记忆和操作的名字。</p>
<h4 data-nodeid="5507">scp</h4>
<p data-nodeid="5508">另一种场景是我需要拷贝一个文件到远程，这时可以使用<code data-backticks="1" data-nodeid="5662">scp</code>指令，如下图，我使用<code data-backticks="1" data-nodeid="5664">scp</code>指令将本地计算机的一个文件拷贝到了 ubuntu 虚拟机用户的家目录中。</p>
<p data-nodeid="5509">比如从<code data-backticks="1" data-nodeid="5667">u1</code>拷贝家目录下的文件<code data-backticks="1" data-nodeid="5669">a.txt</code>到<code data-backticks="1" data-nodeid="5671">u2</code>。家目录有一个简写，就是用<code data-backticks="1" data-nodeid="5673">~</code>。具体指令见下图：</p>
<p data-nodeid="5510"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92j9OADjTcAAPER8w5DNg904.png" alt="Drawing 2.png" data-nodeid="5677"></p>
<p data-nodeid="5511">输入 scp 指令之后会弹出一个提示，要求输入密码，系统验证通过后文件会被成功拷贝。</p>
<h3 data-nodeid="5512">查看本地网络状态</h3>
<p data-nodeid="5513">如果你想要了解本地的网络状态，比较常用的网络指令是<code data-backticks="1" data-nodeid="5681">ifconfig</code>和<code data-backticks="1" data-nodeid="5683">netstat</code>。</p>
<h4 data-nodeid="5514">ifconfig</h4>
<p data-nodeid="5515">当你想知道本地<code data-backticks="1" data-nodeid="5687">ip</code>以及本地有哪些网络接口时，就可以使用<code data-backticks="1" data-nodeid="5689">ifconfig</code>指令。你可以把一个网络接口理解成一个网卡，有时候虚拟机会装虚拟网卡，虚拟网卡是用软件模拟的网卡。</p>
<p data-nodeid="5516">比如：VMware 为每个虚拟机创造一个虚拟网卡，通过虚拟网卡接入虚拟网络。当然物理机也可以接入虚拟网络，它可以通过虚拟网络向虚拟机的虚拟网卡上发送信息。</p>
<p data-nodeid="5517">下图是我的 ubuntu 虚拟机用 ifconfig 查看网络接口信息。</p>
<p data-nodeid="5518"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92j9yAaioXAAbz00ZJYlw555.png" alt="Drawing 3.png" data-nodeid="5695"></p>
<p data-nodeid="5519" class="te-preview-highlight">可以看到我的这台 ubuntu 虚拟机一共有 2 个网卡，ens33 和 lo。<code data-backticks="1" data-nodeid="5697">lo</code>是本地回路（local lookback），发送给<code data-backticks="1" data-nodeid="5699">lo</code>就相当于发送给本机。<code data-backticks="1" data-nodeid="5701">ens33</code>是一块连接着真实网络的虚拟网卡。</p>
<h4 data-nodeid="5520">netstat</h4>
<p data-nodeid="5521">另一个查看网络状态的场景是想看目前本机的网络使用情况，这个时候可以用<code data-backticks="1" data-nodeid="5705">netstat</code>。</p>
<p data-nodeid="5522"><strong data-nodeid="5710">默认行为</strong></p>
<p data-nodeid="5523">不传任何参数的<code data-backticks="1" data-nodeid="5712">netstat</code>帮助查询所有的本地 socket，下图是<code data-backticks="1" data-nodeid="5714">netstat | less</code>的结果。</p>
<p data-nodeid="5524"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92j-aAAZlfAAizLye7uc4727.png" alt="Drawing 4.png" data-nodeid="5718"></p>
<p data-nodeid="5525">如上图，我们看到的是 socket 文件。socket 是网络插槽被抽象成了文件，负责在客户端、服务器之间收发数据。当客户端和服务端发生连接时，客户端和服务端会同时各自生成一个 socket 文件，用于管理这个连接。这里，可以用<code data-backticks="1" data-nodeid="5720">wc -l</code>数一下有多少个<code data-backticks="1" data-nodeid="5722">socket</code>。</p>
<p data-nodeid="5526"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92j-2AVEYjAAA8xcVMQzc068.png" alt="Drawing 5.png" data-nodeid="5726"></p>
<p data-nodeid="5527">你可以看到一共有 615 个 socket 文件，因为有很多 socket 在解决进程间的通信。就是将两个进程一个想象成客户端，一个想象成服务端。并不是真的有 600 多个连接着互联网的请求。</p>
<p data-nodeid="5528"><strong data-nodeid="5731">查看 TCP 连接</strong></p>
<p data-nodeid="5529">如果想看有哪些 TCP 连接，可以使用<code data-backticks="1" data-nodeid="5733">netstat -t</code>。比如下面我通过<code data-backticks="1" data-nodeid="5735">netstat -t</code>看<code data-backticks="1" data-nodeid="5737">tcp</code>协议的网络情况：</p>
<p data-nodeid="5530"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92j_aAbxdlAAEAdzG3a2s636.png" alt="Drawing 6.png" data-nodeid="5741"></p>
<p data-nodeid="5531">这里没有找到连接中的<code data-backticks="1" data-nodeid="5743">tcp</code>，因为我们这台虚拟机当时没有发生任何的网络连接。因此我们尝试从机器<code data-backticks="1" data-nodeid="5745">u2</code>（另一台机器）ssh 登录进<code data-backticks="1" data-nodeid="5747">u1</code>，再看一次：</p>
<p data-nodeid="5532"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92kAaAMuMDAAFWQdSNGfk978.png" alt="Drawing 7.png" data-nodeid="5751"></p>
<p data-nodeid="5533">如上图所示，可以看到有一个 TCP 连接了。</p>
<p data-nodeid="5534"><strong data-nodeid="5756">查看端口占用</strong></p>
<p data-nodeid="5535">还有一种非常常见的情形，我们想知道某个端口是哪个应用在占用。如下图所示：</p>
<p data-nodeid="5536"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92kBKAHr2RAAEnmEOZ8RM010.png" alt="Drawing 8.png" data-nodeid="5760"></p>
<p data-nodeid="5537">这里我们看到 22 端口被 sshd，也就是远程登录模块被占用了。<code data-backticks="1" data-nodeid="5762">-n</code>是将一些特殊的端口号用数字显示，<code data-backticks="1" data-nodeid="5764">-t</code>是指看 TCP 协议，<code data-backticks="1" data-nodeid="5766">-l</code>是只显示连接中的连接，<code data-backticks="1" data-nodeid="5768">-p</code>是显示程序名称。</p>
<h3 data-nodeid="5538">网络测试</h3>
<p data-nodeid="5539">当我们需要测试网络延迟、测试服务是否可用时，可能会用到<code data-backticks="1" data-nodeid="5772">ping</code>和<code data-backticks="1" data-nodeid="5774">telnet</code>指令。</p>
<h4 data-nodeid="5540">ping</h4>
<p data-nodeid="5541">想知道本机到某个网站的网络延迟，就可以使用<code data-backticks="1" data-nodeid="5778">ping</code>指令。如下图所示：</p>
<p data-nodeid="5542"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92kB-ARKR5AAP30Xk0nBg068.png" alt="Drawing 9.png" data-nodeid="5782"></p>
<p data-nodeid="5543"><code data-backticks="1" data-nodeid="5783">ping</code>一个网站需要使用 ICMP 协议。因此你可以在上图中看到 icmp 序号。 这里的时间<code data-backticks="1" data-nodeid="5785">time</code>是往返一次的时间。<code data-backticks="1" data-nodeid="5787">ttl</code>叫作 time to live，是封包的生存时间。就是说，一个封包从发出就开始倒计时，如果途中超过 128ms，这个包就会被丢弃。如果包被丢弃，就会被算进丢包率。</p>
<p data-nodeid="5544">另外<code data-backticks="1" data-nodeid="5790">ping</code>还可以帮助我们看到一个网址的 IP 地址。 通过网址获得 IP 地址的过程叫作 DNS Lookup（DNS 查询）。<code data-backticks="1" data-nodeid="5792">ping</code>利用了 DNS 查询，但是没有显示全部的 DNS 查询结果。</p>
<h4 data-nodeid="5545">telnet</h4>
<p data-nodeid="5546">有时候我们想知道本机到某个 IP + 端口的网络是否通畅，也就是想知道对方服务器是否在这个端口上提供了服务。这个时候可以用<code data-backticks="1" data-nodeid="5796">telnet</code>指令。 如下图所示：</p>
<p data-nodeid="5547"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92kCmAcPQzAADcRdxOtdw609.png" alt="Drawing 10.png" data-nodeid="5800"></p>
<p data-nodeid="5548">telnet 执行后会进入一个交互式的界面，比如这个时候，我们输入下图中的文字就可以发送 HTTP 请求了。如果你对 HTTP 协议还不太了解，建议自学一下 HTTP 协议。如果希望和林老师一起学习，可以等待下我之后的《<strong data-nodeid="5806">计算机网络</strong>》专栏。</p>
<p data-nodeid="5549"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92kDKAcYUbAASLFyOyBg4948.png" alt="Drawing 11.png" data-nodeid="5809"></p>
<p data-nodeid="5550">如上图所示，第 5 行的<code data-backticks="1" data-nodeid="5811">GET</code> 和第 6 行的<code data-backticks="1" data-nodeid="5813">HOST</code>是我输入的。 拉勾网返回了一个 301 永久跳转。这是因为拉勾网尝试把<code data-backticks="1" data-nodeid="5815">http</code>协议链接重定向到<code data-backticks="1" data-nodeid="5817">https</code>。</p>
<h3 data-nodeid="5551">DNS 查询</h3>
<p data-nodeid="5552">我们排查网络故障时想要进行一次 DNS Lookup，想知道一个网址 DNS 的解析过程。这个时候有多个指令可以用。</p>
<h4 data-nodeid="5553">host</h4>
<p data-nodeid="5554">host 就是一个 DNS 查询工具。比如我们查询拉勾网的 DNS，如下图所示：</p>
<p data-nodeid="5555"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92kD6AOJPQAAGW1va0D9c041.png" alt="Drawing 12.png" data-nodeid="5825"></p>
<p data-nodeid="5556">我们看到拉勾网 <a href="http://www.lagou.comw" data-nodeid="5829">www.lagou.com</a> 是一个别名，它的原名是 lgmain 开头的一个域名，这说明拉勾网有可能在用 CDN 分发主页（关于 CDN，我们《计算机网络》专栏见）。</p>
<p data-nodeid="5557">上图中，可以找到 3 个域名对应的 IP 地址。</p>
<p data-nodeid="5558">如果想追查某种类型的记录，可以使用<code data-backticks="1" data-nodeid="5833">host -t</code>。比如下图我们追查拉勾的 AAAA 记录，因为拉勾网还没有部署 IPv6，所以没有找到。</p>
<p data-nodeid="5559"><img src="https://s0.lgstatic.com/i/image/M00/5A/68/CgqCHl92kFWAHIqAAACvpo6qaOs100.png" alt="Drawing 13.png" data-nodeid="5837"></p>
<h4 data-nodeid="5560">dig</h4>
<p data-nodeid="5561"><code data-backticks="1" data-nodeid="5839">dig</code>指令也是一个做 DNS 查询的。不过<code data-backticks="1" data-nodeid="5841">dig</code>指令显示的内容更详细。下图是<code data-backticks="1" data-nodeid="5843">dig</code>拉勾网的结果。</p>
<p data-nodeid="5562"><img src="https://s0.lgstatic.com/i/image/M00/5A/69/CgqCHl92kGaADOhxAAR-BfryZ5g689.png" alt="Drawing 14.png" data-nodeid="5847"></p>
<p data-nodeid="5563">从结果可以看到<a href="http://www.lagou.c" data-nodeid="5851">www.lagou.com</a> 有一个别名，用 CNAME 记录定义 lgmain 开头的一个域名，然后有 3 条 A 记录，通常这种情况是为了均衡负载或者分发内容。</p>
<h3 data-nodeid="5564">HTTP 相关</h3>
<p data-nodeid="5565">最后我们来说说<code data-backticks="1" data-nodeid="5855">http</code>协议相关的指令。</p>
<h4 data-nodeid="5566">curl</h4>
<p data-nodeid="5567">如果要在命令行请求一个网页，或者请求一个接口，可以用<code data-backticks="1" data-nodeid="5859">curl</code>指令。<code data-backticks="1" data-nodeid="5861">curl</code>支持很多种协议，比如 LDAP、SMTP、FTP、HTTP 等。</p>
<p data-nodeid="5568">我们可以直接使用 curl 请求一个网址，获取资源，比如我用 curl 直接获取了拉勾网的主页，如下图所示：</p>
<p data-nodeid="5569"><img src="https://s0.lgstatic.com/i/image/M00/5A/5D/Ciqc1F92kG-AJPyrAANJZYQ4u5w784.png" alt="Drawing 15.png" data-nodeid="5866"></p>
<p data-nodeid="5570">如果只想看 HTTP 返回头，可以使用<code data-backticks="1" data-nodeid="5868">curl -I</code>。</p>
<p data-nodeid="5571">另外<code data-backticks="1" data-nodeid="5871">curl</code>还可以执行 POST 请求，比如下面这个语句：</p>
<pre class="lang-java" data-nodeid="5572"><code data-language="java">curl -d <span class="hljs-string">'{"x" : 1}'</span> -H <span class="hljs-string">"Content-Type: application/json"</span> -X POST http:<span class="hljs-comment">//localhost:3000/api</span>
</code></pre>
<p data-nodeid="5573">curl在向<code data-backticks="1" data-nodeid="5874">localhost:3000</code>发送 POST 请求。<code data-backticks="1" data-nodeid="5876">-d</code>后面跟着要发送的数据， -<code data-backticks="1" data-nodeid="5878">X</code>后面是用到的 HTTP 方法，<code data-backticks="1" data-nodeid="5880">-H</code>是指定自定义的请求头。</p>
<h3 data-nodeid="5574">总结</h3>
<p data-nodeid="5575">这节课我们学习了不少网络相关的 Linux 指令，这些指令是将来开发和调试的强大工具。这里再给你复习一下这些指令：</p>
<ul data-nodeid="5576">
<li data-nodeid="5577">
<p data-nodeid="5578">远程登录的 ssh 指令；</p>
</li>
<li data-nodeid="5579">
<p data-nodeid="5580">远程拷贝文件的 scp 指令；</p>
</li>
<li data-nodeid="5581">
<p data-nodeid="5582">查看网络接口的 ifconfig 指令；</p>
</li>
<li data-nodeid="5583">
<p data-nodeid="5584">查看网络状态的 netstat 指令；</p>
</li>
<li data-nodeid="5585">
<p data-nodeid="5586">测试网络延迟的 ping 指令；</p>
</li>
<li data-nodeid="5587">
<p data-nodeid="5588">可以交互式调试和服务端的 telnet 指令；</p>
</li>
<li data-nodeid="5589">
<p data-nodeid="5590">两个 DNS 查询指令 host 和 dig；</p>
</li>
<li data-nodeid="5591">
<p data-nodeid="5592">可以发送各种请求包括 HTTPS 的 curl 指令。</p>
</li>
</ul>
<p data-nodeid="5593"><strong data-nodeid="5895">那么通过这节课的学习，你现在可以来回答本节关联的面试题目：如何查看一个域名有哪些 NS 记录了吗？</strong></p>
<p data-nodeid="5594">老规矩，请你先在脑海里构思下给面试官的表述，并把你的思考写在留言区，然后再来看我接下来的分析。</p>
<p data-nodeid="5595"><strong data-nodeid="5907">【解析】</strong> host 指令提供了一个<code data-backticks="1" data-nodeid="5901">-t</code>参数指定需要查找的记录类型。我们可以使用<code data-backticks="1" data-nodeid="5903">host -t ns {网址}</code>。另外 dig 也提供了同样的能力。如果你感兴趣，还可以使用<code data-backticks="1" data-nodeid="5905">man</code>对系统进行操作。</p>
<h3 data-nodeid="5596">思考题</h3>
<p data-nodeid="5597"><strong data-nodeid="5915">最后我再给你出一道需要查资料的思考题目：如何查看正在 TIME_WAIT 状态的连接数量</strong>？</p>
<p data-nodeid="5598" class="">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### *李：
> 希望后续的计算机网络课程能够由浅入深附带有实际操作实践就更好了

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢同学的建议，已经反馈给老师哦，我们会听取大家的建议，努力做出更优质的内容！！

##### *帅：
> 老师的《计算机网络》大概多久上啊

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 感谢小伙伴的关注哦，我们正在全力推进中，不会让大家等太久：）

##### *兴：
> netstat -an | grep TIME_WAIT | wc -l

##### **杰：
> 补充一个自己常用的域名解析命令：nslookup

##### **9660：
> ttl=128指的是路由次数超过128次之后包被丢弃的意思吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 查了一下RFC 791（IP协议的RFC）。This field indicates the maximum time the datagram is allowed to
    remain in the internet system.  不是路由次数。
 具体欢迎在“重学操作系统交流群”里和我继续讨论。

##### *达：
> 很不错啊，受益匪浅

##### **江：
> 计算机使用二进制是因为二进制只有两个状态吧，非0即1，所以计算机运算速度才能这么快，而且在电路中只用比较电平的高低，而不用比较具体的值。

##### **磊：
> “ttl=128表示延迟超过128ms就会被丢弃”这个说法是否有误？印象中这个值表示的是最大跳数

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 查了一下RFC 791（IP协议的RFC）。This field indicates the maximum time the datagram is allowed to
    remain in the internet system.  不是路由次数。
 具体欢迎在“重学操作系统交流群”里和我继续讨论。

##### yang：
> netstat -anp | grep "TIME_WAIT" | wc -l

##### **文：
> 越想深入学习，就越觉得计算机组成原理、操作系统这些基础知识非常重要，期待老师的“计算机网络”课程😃

##### **钻：
> 请教老师个问题，这里说的NS记录是指哪些东西？看得不太懂，可以举个例子吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我会接下来的计算机网络的课程中讲解这块知识，如果等不及可以自己搜索哈！

##### **钻：
> netstat -an|grep TIME_WAIT|wc -l可以查看TIME_WAIT状态的连接数量。

##### **威：
> ttl128不是指128毫秒，是指可以经过128个路由器吧。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 查了一下RFC 791（IP协议的RFC）。This field indicates the maximum time the datagram is allowed to
    remain in the internet system.  不是路由次数。

##### *浩：
> 这些指令真是太有用了

##### *华：
> 《计算机网络》等得睡不着觉啊

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 好饭不怕晚，好课不怕等：）小伙伴耐心等待一下哦，《计算机网络》4月19日会在拉勾教育上线哦

##### **宇：
> netstat -t | grep TIME_WAIT | wc -l

##### *铭：
> 计算机网络什么时候出呢。。最近在补这些东西，虽然经常用，但是原理还不是很明白。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 三月左右哦

##### *帅：
> tracert tracroute route 也挺常用

##### *程：
> netstat -an | grep TIME_WAIT | wc -l

##### **安：
> 查看time_wait连接数：`netstat -n | awk '/^tcp/ {++S[$NF]} END {for(a in S) print a, S[a]}'`

##### **队长：
> 老师要以加一个ip和ss命令么😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 赞！很好的建议。

##### **俊：
> host 和 dig 命名在 centos 上默认是没有的，需要手动进行安装yum install bind-utils

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; windows，ubuntu里需要这个。

##### *贝：
> netstat |grep TIME_WAIT |wc -l

