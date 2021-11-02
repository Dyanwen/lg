<p style="line-height: 1.75em; text-align: justify;"><strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">Nginx 负载均衡常见问题</span></strong></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那么，Nginx 负载均衡的通常配置会出现哪些问题呢？这里列出几种比较常见的问题：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">客户端 IP 地址获取问题</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">域名携带问题</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">负载均衡导致 session 丢失问题</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">动态负载均衡问题</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">真实的 Realserver 状态检测</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来，我们就重点讲解下 Nginx 作为负载均衡的这几个问题，值得一提的是，这五个问题中的动态负载均衡问题将在下一课时重点讲解，所以本课时就不再讲解了。</span></p> 
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>客户端IP地址获取问题</strong></span></p></h2> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第一个问题是客户端 IP 地址获取问题，为什么会存在客户端 IP 地址获取问题呢？</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><img src="https://s0.lgstatic.com/i/image3/M01/6A/4D/CgpOIF5U0tqAdFH3AAFLYg4Sm0c598.png"></p> 
<p style="text-align:center"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们来看这样一张模拟图，图中请求从用户到 Nginx，再到后端服务。我们可以看到用户的 IP 地址是 100.100.100.100，Nginx 的地址是 192.168.0.1，通过 Nginx 负载均衡到三台后端服务，它们的 IP 分别是 192.168.1.1、192.168.1.2、192.168.1.3。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">那么客户端的 IP 地址为什么会无法被后端服务获取呢？原因是我们获取方式在Nginx的加入负载均衡后出现了差异，具体如下：</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们了解后端服务获取IP的方式，第一种方式是由后端服务通过 4 层 TCP 协议获取源 IP 地址，你会发现，通过这种方式，后端服务只能获取 Nginx 的 IP 地址，而无法获取到 100.100.100.100 这个 IP 地址，原因在于 Nginx 做了一层反向代理，而反向代理修改了客户的源地址和源端口的包头，所以在后端服务，通过这种方式只能获取到 Nginx 的 IP，而无法获取到用户的 IP。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">另外一种方式是后端服务通过 HTTP 标准请求头信息 X-forwarded-for 获取用户IP信息，但是通过代理这一层时，有可能把 X-forwarded-for 改写或丢失用户的请求地址，这样就有可能导致 X-forwarded-for 在后端无法获取用户信息。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"> X-forwarded-for 头的格式，它会一层一层往后添加信息，比如第一层是客户端的 IP 地址，然后是通过代理后的代理层IP，代理层后一层一层的追加信息。X-forwarded-for需要通过 Nginx 配置添加，从而使得标准协议将以 X-forwarded-for 头为载体，把用户的 IP 地址和代理层 IP 地址添加到这个载体中，这样后端才能通过X-forwarded-for 获取到详细的 IP 地址信息。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>解决办法</strong></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">知道了问题发生的原因，我们如何来解决呢？第一种方式的解决办法是在 Nginx 负载均衡的基础上添加了一个转发到后端的标准 head 信息，把用户的 IP 信息通过 X-Forwarded-For &nbsp;方式传递过去。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二种方式是就是添加一个 X-Real 的自定义头，自定义头我们可以随意的命名，一般情况下命名为 X-Real_IP，把用户的真实四层 IP地址赋值给它，如何做到呢？</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">remote_addr 是一个 Nginx 的内置变量，它获取到的是 Nginx 层前端的用户 IP 地址，这个地址是一个 4 层的 IP 地址，我们来看这张图，Nginx 的 IP 地址是 192.168.0.1，它的 remote_addr 是什么呢？它就是用户的 100.100.100.100 这个地址，也就是直接将 TCP/IP 协议的用户 IP 地址（源地址）以 remote_addr 变量的方式赋值给 X-Real_IP 的自定义变量，后端直接通过 X-Real 获取 X-Real IP 信息，便可获取到用户地址。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">更深一层的分析，这两种解决方式虽然解决地址获取的问题，但各有优劣，X-Forwarded-For 是通过 Nginx 中的 $proxy_add_x_forwarded_for 内置变量获取的头信息，通过 X-Forwarded-For传递到后端。在 Nginx 通过 $proxy_add_x_forwarded_for 内置变量获取的 X-Forwarded-For 信息本身可能不准确，因为前端用户可以作篡改 HTTP 的请求头，所以如果被一些恶意用户篡改了请求的 X-Forwarded 信息就会导致信息影响获取真实的用户 IP 地址信息。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二种方式相对于第一种方式而言更加准确，因为 remote_addr 是直接获取第一层代理的用户 IP 地址，如果直接把这个地址传递给 X-Real，这样就会更加准确。但是它有什么劣势呢？如果是多级代理的话，用户如果不是直接请求到最终的代理层，而是在中间通过了 n 层带来转发过来的话，此时 remote_addr 可能获取的不是用户的信息，而是 Nginx 最近一层代理过来的 IP 地址，此时同样没有获取到真实的用户 IP 地址信息。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">可以看到，这两种方式各有优劣，通常你也可以把这两种配置都加进去，然后交给后端综合分析，这就是对于客户端 IP 地址携带通常的配置方式。</span></p> 
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>如何解决域名携带</strong></span></p></h2> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来，我们来看第二个问题，也就是 Nginx 作为负载均衡是如何把请求域名信息携带到后端的？这样的问题是什么样的场景呢？首先我们来看一张示意图。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/6A/4D/Cgq2xl5U0uGAe2_GAAFMYE51VWw660.png"></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">用户的 IP 同样是 100.100.100.100，Nginx 对外 IP 地址是 200.200.200.200，Nginx 本身内部的地址是 192.168.0.1，同样分发给三个后端服务。当用户请求某一 Nginx 入口域名时，会携带一个头信息，叫作 Host 头信息。如果通过域名方式，当用户请求的是 http://www.jesonc.com，它的 Host 头就是 &nbsp;www.jesonc.com，以这样的方式请求到代理层时，由于 Nginx 没有将头信息携带到后端，而是把配置的头信息置为空，这样就导致后端服务想要获取不到用户请求的 Host 信息，导致服务不可用。这时需要 Nginx 将用户携带的 Host 头信息真实的传递给后端。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二种情况是假设用户请求用的 IP 地址方式（非域名），此时会发现，用户请求的 IP 地址的 Host 头信息本身就是空的，所以如果后端需要用到 Host 头信息，这就需要在 Nginx 中把 Host 头改写为一个要求的头信息，我们来看下这个场景配置：</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>http&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;upstream&nbsp;app_servers&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip1:port1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip2:port2;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip3:port3;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;location&nbsp;/&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_set_header&nbsp;Host&nbsp;$host&nbsp;;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy&nbsp;set_header&nbsp;Host&nbsp;www.vipumi.com;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_pass&nbsp;http://app_servers;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;….
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们通过 proxy_set_header 配置方式加入 Host 头部字段，如果用户的请求携带了域名，就把这个域名的 head 头以标准的 HTTP 方式将头信息传递到后端，然后让后端获取 Host 头信息，这样访问就不会受影响。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">然后，如果用户没有携带头信息，而后端又要求指定域名，那么我们就可以在 proxy_set_header 下，将 Host 头信息指定成一个固定的域名，保证满足后台对 Host 头信息的要求，这样就可以解决域名携带问题。</span></p> 
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>负载均衡导致session丢失问题</strong></span></p></h2> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/6A/4D/CgpOIF5U0tOAeMMQAAGg2smEGW8440.png" style="text-align: center; white-space: normal;"></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">另外一个是 session 丢失问题，我们知道，session 是用户的一个会话信息，服务端和用户端通信需要一个访问认证，以及对鉴权处理的时候，给客户端分配一串 session 信息，这个 session 信息会在客户端以 cookie 的方式承载到服务端中进行校验。如果加入 Nginx 负载均衡，会默认开启一个轮询策略。假如这样的一种场景，用户请求到 Nginx，第一次请求会分发给 App server 1，第二次请求分发给 App server 2，但用户的 session 保留在 App1 上，此时这条请求再去请求 App2 的话，由于 App2 上没有 session 信息，就导致会话丢失，用户需要重新登录。我们看到 Nginx 作为负载均衡 &nbsp;+ Java 后台应用中遇见的一种问题，我们该怎样解决呢？</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">解决 session 丢失的问题，通常有以下几种做法：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">解决方案 1：Session 保持</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第一种方式是做 Session 保持，就是把负载均衡策略基于原有轮询数的基础上，改用 ip_hash、URL_hash 来解决。ip_hash 就是基于用户 IP 来做 hash，一个用户的请求统一分发到一台机器上。URL_hash 用于用户请求固定页面时，将用户请求固定到具体 后端 上，就保证了 Session 不会被丢失。</span></p> 
<ul> 
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">解决方案 2：Session 复制</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第二种方式是 Session 复制。所谓 Session 复制是在后台应用的基础上，让 Session 之间可以以传播的方式进行复制。也就是 App1 上如果有一个 Session，那么它可以复制给 App2、App3，无论怎样轮询，三个 App 上都会有同样的 Session 信息，不至于因为轮询导致丢失会话失效。</span></p> 
<ul> 
 <li><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">解决方案 3：Session 共享</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">Session 共享是由程序完成的，我们把 Session 信息不放在本地，通过应用程序把 Session 信息放入共享的 K/V 存储中，这样就不会产生 Session 丢失情况了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">我们可以看到，如果 Nginx 解决 Session 丢失问题基于 Session 保持来解决，所以怎样配置 Nginx 呢？</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>http&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;upstream&nbsp;app_servers&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ip_hash;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip1:port1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip2:port2;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip3:port3;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;location&nbsp;/&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_pass&nbsp;http://app_servers;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;….
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这个配置中我们在负载均衡 Upstream 对应的地址池中加入一个&nbsp;ip_hash 策略，这样就可以基于&nbsp;ip_hash 的方式进行轮询了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>http&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;upstream&nbsp;app_servers&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hash&nbsp;$request_uri;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip1:port1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip2:port2;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;ip3:port3;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;…
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;location&nbsp;/&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_pass&nbsp;http://app_servers;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;….
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><br></span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">第二种方式基于 URL_hash，URL_hash 方式就是添加一个自定义 hash，通过设置 Nginx 内部的请求变量 $request_url 进行 hash 运算，这样就可以基于这个请求的 URL 地址来做 hash 了，也就做到 Session 的保持的方案。</span></p> 
<h2><p style="line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);"><strong>真实的 Realserver 状态检测</strong></span></p></h2> 
<p style="text-align:center"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><img src="https://s0.lgstatic.com/i/image3/M01/6A/4D/Cgq2xl5U0sqAES8QAAFVZjwh4gg899.png" style="text-align: left; color: rgb(51, 51, 51);"></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">最后一个常见问题是如何检测后端的 Realserver 状态，我们首先需要了解 Nginx 默认真实检测后台服务状态是怎么实现的.</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">生产环境中，一个 Nginx 作为负载均衡如果发现后端某一个节点出现问题，那么它会把这个节点剔除，并检测下一个节点，那么 Nginx 是如何检测到节点有问题呢？默认情况下，Nginx 基于 TCP 端口和连接方式检测，也就是在服务 ping 不通，或无法建立 TCP 链接，以及端口服务完全不可用的状态下，才会认为这个服务不可用。</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">你可以想一想，在 Nginx 的这种检测机制下通常是有遗漏的，如果响应状态或返回状态有问题，对于 Nginx 代理层而言，它并没有做到一个有效的容错，只是检测底层 TCP 连接方式，是不是也存在一定的局限性。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">这个时候，你可能会想是否可以加入一个 proxy_next_upstream 就可以把后端节点剔除呢？proxy_next_upstream 虽然能够检测到服务端返回到前端的状态码，但是无法做到真正自动摘除故障节点。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(63, 63, 63);">那么怎么做呢？这里就需要依赖一个第三方模块了，这个第三方模块就是 nginx_upstream_check_module，它是由淘宝技术团队开发并开源的，提供了更加细致的对后台服务真实状态的检测。你可以把这个模块编译到 Nginx 中，或是使用淘宝的 Tengine，Tengine 也是基于 Nginx 1.6 版本开发开源的。我们接下来看下具体配置：</span></p> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p> 
<pre>check&nbsp;interval=3000&nbsp;rise=2&nbsp;fall=5&nbsp;timeout=1000&nbsp;type=http;&nbsp;&nbsp;//定义检查间隔、周期、时间
check_keepalive_requests&nbsp;100;&nbsp;&nbsp;//一个连接发送的请求数
check_http_send&nbsp;“HEAD&nbsp;/&nbsp;HTTP/1.1\r\nConnection:&nbsp;keep-alive\r\n\r\n”;&nbsp;//定义健康检查方式
check_http_expect_alive&nbsp;http_2xx&nbsp;http_3xx;&nbsp;&nbsp;//判断后端返回状态码</pre> 
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">通过这样的配置，我们定义了检测间隔、周期和超时时间；定义了一个连接发送的请求数；然后是定义健康检查方式，比如检查 head 头的请求信息；最后判断后端返回状态码，如果是非 200 或 300 的话，就认为后台服务不健康，需要把这个问题服务除掉，避免用户请求到问题节点。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本专栏课中的所有案例配置及源代码,你可以课后通过这个</span><a class="ql-link ql-author-25848023" href="http://www.jesonc.com/jeson/2020/02/07/ywgs36/" target="_blank" rel="noopener noreferrer nofollow" style="color: rgb(63, 63, 63); text-decoration: underline; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">http://www.jesonc.com/jeson/2020/02/07/ywgs36/</span></a><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">自己下载,密码为:mukelaoshi。</span></p>

---

### 精选评论

##### **传：
> 源代码无法下载，密码不对<br>

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 你可以通过这个地址 http://www.jesonc.com/jeson/2020/02/07/ywgs36/ 自己下载，密码为：mukelaoshi。如果下载失败可关注 拉勾教育 公众号 加入学习群和大家一起讨论解决

