<p style="line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">在上一课时我们重点介绍了 Nginx 作为 HTTP 代理网关常见且基本的优化技巧。实际上 Nginx 除了承担代理网关角色外还会应用于 7 层应用上的负载均衡，本课时重点讲解 Nginx 的负载均衡应用架构，及最常见的问题。</span><br></p>
<h1><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;">课前学习提示</span></p></h1>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>学前提示</strong></span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Nginx 作为负载均衡是基于代理模式的基础之上，所以在学习本课时前，你需要对 Nginx 的代理、负载均衡的基本原理及 Nginx 负载均衡配置有基础的了解。有了这个基础以后，学习本课时的内容便会得心应手。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>章节思维导图</strong></span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这里我先画了一张思维导图，我们来一起看下本课计划讲解哪些内容。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/68/66/Cgq2xl5OWo2ASqnDAAEXHWzwSlE436.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先会来讲一讲 Nginx 作为负载均衡在整体服务架构和网站服务架构里所扮演的角色，其次重点介绍 Nginx 作为负载均衡时遇到的一些常见问题，比如客户端 IP 地址获取问题、域名携带问题 等等。</span></p>
<h1><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;">Nginx 负载均衡应用架构</span></p></h1>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">关于 Nginx 负载均衡应用架构在企业应用中主要有两种类型。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">分层入口代理架构</span></p></h2>
<p style="text-align:center"><img src="https://s0.lgstatic.com/i/image3/M01/68/73/CgpOIF5OfKmARBsLAAJDi2DWALc170.png"></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第一类是分层入口代理架构（属于相对传统架构），我们可以对整个后台网站服务系统架构做一个分层。大体可以分为用户请求入口层，以及为用户请求提供逻辑处理的服务层和为用户提供真正相关数据的数据层。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如图所示，我们可以发现入口层是最接近用户请求的，所以在这一层中，Nginx 扮演着重要的角色——入口网关，并承担 7 层负载均衡（LB）的功能。如图所示入口层的 Nginx 之前还有一套 LB，LB 层的主要功能是为了 保证 Nginx 本身的高可用、或承担 TCP/IP 负载均衡功能，所以这里整个入口层的负载均衡模式是一个 4 层 LB+7 层 LB（Nginx），这套架构中把与业务服务的相关功能则由 Nginx 来处理。</span></p>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">哪一些业务服务相关功能交给 Nginx 作合适呢？比如在入口层我们会放一些和用户相关的信息，也比如之前讲过的动静分离（及实现分离网站页面的静态元素和动态元素) ，我们知道静态元素没有必要下沉到数据层获取，可以直接通过 Nginx 实现动态和静态的分流并由 Nginx 直接处理。另外，用户的访问控制、反爬虫等规则也是在入口层的 Nginx 实现的。</span></p>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">服务层同样也需要 Nginx ， 来负载均衡实现上层请求应答上的高可用。 </span></p>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">分层架构的最后一层是数据层，数据层中 Nginx 同样可以实现负载均衡，但数据层中通常使用的 Nginx 较少见，为什么呢？因为这一层更追求数据调用的效率，比如 Memcache、MySQL 等数据库调用更多是基于底层的协议请求，而非更上层的 HTTP 请求。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">但如果如果追求 HTTP 的可靠性、和应用性，是可以借助 Nginx 的负载均衡实现的，如：可Redis 使用 Nginx 做反向代理，通过 Nginx 把前端发送的 HTTP 请求转换成 Redis 协议的请求方式去请求 Redis，这样就完成了 Redis 的反向代理。这种方式，企业可以很好控制Redis的监控、数据的一致性保障、及基于 Hash 算法稳定性保障。</span></p>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总结一下，Nginx 在分层架构里扮演了一个 7 层应用层负载均衡角色。随着软件架构和系统架构是不断演进变化，我们发现企业开始采用 K8s、Docker 这种轻量化、虚拟化部署；还有很多企业更加倾向于微服务架构，支持 set 化等。在这样的架构下，传统的分层负载均衡模式，促使改进去支持服务注册和发现。这个就是Jeson想介绍的第二种Nginx负载均衡模式。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">服务注册发现代理架构</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/68/66/Cgq2xl5OWquAOOSKAAKxOrFbX1w188.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如图所示，图中重点列出了 Nginx 在注册发现场景中扮演的角色。同样，我们可以看到整个流量还是通过 Nginx 来做入口网关的，但是重要的一点是 Nginx 需要支持动态的发现和注册后端服务。</span></p>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">注册是指后端的应用程序（如图中的 App1~App4）需要去往前端的中心存储节点里面注册它的应用服务，当注册上报后，Nginx 动态发现并动态生成发现的配置，然后对入口网关代理负载均衡进行分流调整，我们可以发现在基于 K8s 和 Docker 这种部署模式业务入口网关就应用了这种架构。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">两种负载均衡应用架构就讲完了，我们会发现两种架构最大的区别是后面一种支持后端服务的注册与发现。</span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">接下来我们回顾下 Nginx 负载均衡配置。</span></p>
<h2><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">常用配置</span></p></h2>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">首先，我们先回顾Nginx基础负载均衡配置。</span></p>
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
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_pass&nbsp;http://app_servers;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;….
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>
<p style="line-height: 150%;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在这里，将通过 Upstream 配置分发入口请求流量到了后台三个 IP 节点对应的端口服务。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这是一种常见的 Nginx 负载均衡配置，但是这样配置在实际企业应用时会发生一些问题，我们接下来就来讲解如何解决 Nginx 做负载均衡时的常见问题。</span></p>

---

### 精选评论


