<p data-nodeid="74771" class="">在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=455#/detail/pc?id=4573" data-nodeid="74775">第 02 课时“ 核心概念：镜像、容器、仓库，彻底掌握 Docker 架构核心设计理念”</a>里。我简单介绍了 Docker 架构的形成，相信你已经对 Docker 的架构有了一个整体的认知。这一讲我将带你深入剖析 Docker 的各个组件的作用及其底层的实现原理。</p>

<p data-nodeid="74563">首先我们来回顾一下 Docker 的组件构成。</p>
<h3 data-nodeid="74564">Docker 的组件构成</h3>
<p data-nodeid="74565" class="te-preview-highlight">Docker 整体架构采用 C/S（客户端 / 服务器）模式，主要由客户端和服务端两大部分组成。客户端负责发送操作指令，服务端负责接收和处理指令。客户端和服务端通信有多种方式，即可以在同一台机器上通过<code data-backticks="1" data-nodeid="74667">UNIX</code>套接字通信，也可以通过网络连接远程通信。</p>
<p data-nodeid="74566"><img src="https://s0.lgstatic.com/i/image/M00/56/40/CgqCHl9rFtSAPGOeAADIK4E6wrc522.png" alt="image.png" data-nodeid="74671"></p>
<div data-nodeid="74567"><p style="text-align:center">图1 Docker 整体架构图</p></div>
<p data-nodeid="74568">从整体架构可知，Docker 组件大体分为 Docker 相关组件，containerd 相关组件和容器运行时相关组件。下面我们深入剖析下各个组件。</p>
<h3 data-nodeid="74569">Docker 组件剖析</h3>
<p data-nodeid="74570">Docker 到底有哪些组件呢？我们可以在 Docker 安装路径下执行 ls 命令，这样可以看到以下与 Docker 有关的组件。</p>
<pre class="lang-java" data-nodeid="74571"><code data-language="java">-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">27941976</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> containerd
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">4964704</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> containerd-shim
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">15678392</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> ctr
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">50683148</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker
-rwxr-xr-x <span class="hljs-number">1</span> root root   <span class="hljs-number">764144</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker-init
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">2837280</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> docker-proxy
-rwxr-xr-x <span class="hljs-number">1</span> root root <span class="hljs-number">54320560</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> dockerd
-rwxr-xr-x <span class="hljs-number">1</span> root root  <span class="hljs-number">7522464</span> Dec <span class="hljs-number">12</span>  <span class="hljs-number">2019</span> runc
</code></pre>
<p data-nodeid="74572">这些组件根据工作职责可以分为以下三大类。</p>
<ol data-nodeid="74573">
<li data-nodeid="74574">
<p data-nodeid="74575">Docker 相关的组件：docker、dockerd、docker-init 和 docker-proxy</p>
</li>
<li data-nodeid="74576">
<p data-nodeid="74577">containerd 相关的组件：containerd、containerd-shim 和 ctr</p>
</li>
<li data-nodeid="74578">
<p data-nodeid="74579">容器运行时相关的组件：runc</p>
</li>
</ol>
<p data-nodeid="74580">下面我们就逐一了解。</p>
<h4 data-nodeid="74581">Docker 相关的组件</h4>
<p data-nodeid="74582"><strong data-nodeid="74684">（1）docker</strong></p>
<p data-nodeid="74583">docker 是 Docker 客户端的一个完整实现，它是一个二进制文件，对用户可见的操作形式为 docker 命令，通过 docker 命令可以完成所有的 Docker 客户端与服务端的通信（还可以通过 REST API、SDK 等多种形式与 Docker 服务端通信）。</p>
<p data-nodeid="74584">Docker 客户端与服务端的交互过程是：docker 组件向服务端发送请求后，服务端根据请求执行具体的动作并将结果返回给 docker，docker 解析服务端的返回结果，并将结果通过命令行标准输出展示给用户。这样一次完整的客户端服务端请求就完成了。</p>
<p data-nodeid="74585"><strong data-nodeid="74690">（2）dockerd</strong></p>
<p data-nodeid="74586">dockerd 是 Docker 服务端的后台常驻进程，用来接收客户端发送的请求，执行具体的处理任务，处理完成后将结果返回给客户端。</p>
<p data-nodeid="74587">Docker 客户端可以通过多种方式向 dockerd 发送请求，我们常用的 Docker 客户端与 dockerd 的交互方式有三种。</p>
<ul data-nodeid="74588">
<li data-nodeid="74589">
<p data-nodeid="74590">通过 UNIX 套接字与服务端通信：配置格式为unix://socket_path，默认 dockerd 生成的 socket 文件路径为 /var/run/docker.sock，该文件只有 root 用户或者 docker 用户组的用户才可以访问，这就是为什么 Docker 刚安装完成后只有 root 用户才能使用 docker 命令的原因。</p>
</li>
<li data-nodeid="74591">
<p data-nodeid="74592">通过 TCP 与服务端通信：配置格式为tcp://host:port，通过这种方式可以实现客户端远程连接服务端，但是在方便的同时也带有安全隐患，因此在生产环境中如果你要使用 TCP 的方式与 Docker 服务端通信，推荐使用 TLS 认证，可以通过设置 Docker 的 TLS 相关参数，来保证数据传输的安全。</p>
</li>
<li data-nodeid="74593">
<p data-nodeid="74594">通过文件描述符的方式与服务端通信：配置格式为：fd://这种格式一般用于 systemd 管理的系统中。</p>
</li>
</ul>
<p data-nodeid="74595">Docker 客户端和服务端的通信形式必须保持一致，否则将无法通信，只有当 dockerd 监听了 UNIX 套接字客户端才可以使用 UNIX 套接字的方式与服务端通信，UNIX 套接字也是 Docker 默认的通信方式，如果你想要通过远程的方式访问 dockerd，可以在 dockerd 启动的时候添加 -H 参数指定监听的 HOST 和 PORT。</p>
<p data-nodeid="74596"><strong data-nodeid="74702">（3）docker-init</strong></p>
<p data-nodeid="74597">如果你熟悉 Linux 系统，你应该知道在 Linux 系统中，1 号进程是 init 进程，是所有进程的父进程。主机上的进程出现问题时，init 进程可以帮我们回收这些问题进程。同样的，在容器内部，当我们自己的业务进程没有回收子进程的能力时，在执行 docker run 启动容器时可以添加 --init 参数，此时 Docker 会使用 docker-init 作为1号进程，帮你管理容器内子进程，例如回收僵尸进程等。</p>
<p data-nodeid="74598">下面我们通过启动一个 busybox 容器来演示下：</p>
<pre class="lang-shell" data-nodeid="74599"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -it busybox sh</span>
/ # ps aux
PID&nbsp; &nbsp;USER&nbsp; &nbsp; &nbsp;TIME&nbsp; COMMAND
&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp; 0:00 sh
&nbsp; &nbsp; 6 root&nbsp; &nbsp; &nbsp; 0:00 ps aux
/ #
</code></pre>
<p data-nodeid="74600">可以看到容器启动时如果没有添加 --init 参数，1 号进程就是 sh 进程。</p>
<p data-nodeid="74601">我们使用 Crtl + D 退出当前容器，重新启动一个新的容器并添加 --init 参数，然后看下进程：</p>
<pre class="lang-shell" data-nodeid="74602"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -it --init busybox sh</span>
/ # ps aux
PID&nbsp; &nbsp;USER&nbsp; &nbsp; &nbsp;TIME&nbsp; COMMAND
&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp; 0:00 /sbin/docker-init -- sh
&nbsp; &nbsp; 6 root&nbsp; &nbsp; &nbsp; 0:00 sh
&nbsp; &nbsp; 7 root&nbsp; &nbsp; &nbsp; 0:00 ps aux
</code></pre>
<p data-nodeid="74603">可以看到此时容器内的 1 号进程已经变为 /sbin/docker-init，而不再是 sh 了。</p>
<p data-nodeid="74604"><strong data-nodeid="74711">（4）docker-proxy</strong></p>
<p data-nodeid="74605">docker-proxy 主要是用来做端口映射的。当我们使用 docker run 命令启动容器时，如果使用了 -p 参数，docker-proxy 组件就会把容器内相应的端口映射到主机上来，底层是依赖于 iptables 实现的。</p>
<p data-nodeid="74606">下面我们通过一个实例演示下。</p>
<p data-nodeid="74607">使用以下命令启动一个 nginx 容器并把容器的 80 端口映射到主机的 8080 端口。</p>
<pre class="lang-shell" data-nodeid="74608"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run --name=nginx -d -p 8080:80 nginx</span>
</code></pre>
<p data-nodeid="74609">然后通过以下命令查看一下启动的容器 IP：</p>
<pre class="lang-shell" data-nodeid="74610"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker inspect --format <span class="hljs-string">'{{ .NetworkSettings.IPAddress }}'</span> nginx</span>
172.17.0.2
</code></pre>
<p data-nodeid="74611">可以看到，我们启动的 nginx 容器 IP 为 172.17.0.2。</p>
<p data-nodeid="74612">此时，我们使用 ps 命令查看一下主机上是否有 docker-proxy 进程：</p>
<pre class="lang-shell" data-nodeid="74613"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo ps aux |grep docker-proxy</span>
root&nbsp; &nbsp; &nbsp; 9100&nbsp; 0.0&nbsp; 0.0 290772&nbsp; 9160 ?&nbsp; &nbsp; &nbsp; &nbsp; Sl&nbsp; &nbsp;07:48&nbsp; &nbsp;0:00 /usr/bin/docker-proxy -proto tcp -host-ip 0.0.0.0 -host-port 8080 -container-ip 172.17.0.2 -container-port 80
root&nbsp; &nbsp; &nbsp; 9192&nbsp; 0.0&nbsp; 0.0 112784&nbsp; &nbsp;992 pts/0&nbsp; &nbsp; S+&nbsp; &nbsp;07:51&nbsp; &nbsp;0:00 grep --color=auto docker-proxy
</code></pre>
<p data-nodeid="74614">可以看到当我们启动一个容器时需要端口映射时， Docker 为我们创建了一个 docker-proxy 进程，并且通过参数把我们的容器 IP 和端口传递给 docker-proxy 进程，然后 docker-proxy 通过 iptables 实现了 nat 转发。</p>
<p data-nodeid="74615">我们通过以下命令查看一下主机上 iptables nat 表的规则：</p>
<pre class="lang-shell" data-nodeid="74616"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash">  sudo iptables -L -nv -t nat</span>
Chain PREROUTING (policy ACCEPT 35 packets, 2214 bytes)
&nbsp;pkts bytes target&nbsp; &nbsp; &nbsp;prot opt in&nbsp; &nbsp; &nbsp;out&nbsp; &nbsp; &nbsp;source&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination
&nbsp; 398 21882 DOCKER&nbsp; &nbsp; &nbsp;all&nbsp; --&nbsp; *&nbsp; &nbsp; &nbsp; *&nbsp; &nbsp; &nbsp; &nbsp;0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 35 packets, 2214 bytes)
&nbsp;pkts bytes target&nbsp; &nbsp; &nbsp;prot opt in&nbsp; &nbsp; &nbsp;out&nbsp; &nbsp; &nbsp;source&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination

Chain OUTPUT (policy ACCEPT 1 packets, 76 bytes)
&nbsp;pkts bytes target&nbsp; &nbsp; &nbsp;prot opt in&nbsp; &nbsp; &nbsp;out&nbsp; &nbsp; &nbsp;source&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination
&nbsp; &nbsp; 0&nbsp; &nbsp; &nbsp;0 DOCKER&nbsp; &nbsp; &nbsp;all&nbsp; --&nbsp; *&nbsp; &nbsp; &nbsp; *&nbsp; &nbsp; &nbsp; &nbsp;0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;!127.0.0.0/8&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 1 packets, 76 bytes)
&nbsp;pkts bytes target&nbsp; &nbsp; &nbsp;prot opt in&nbsp; &nbsp; &nbsp;out&nbsp; &nbsp; &nbsp;source&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination
&nbsp; &nbsp; 0&nbsp; &nbsp; &nbsp;0 MASQUERADE&nbsp; all&nbsp; --&nbsp; *&nbsp; &nbsp; &nbsp; !docker0&nbsp; 172.17.0.0/16&nbsp; &nbsp; &nbsp; &nbsp; 0.0.0.0/0
&nbsp; &nbsp; 0&nbsp; &nbsp; &nbsp;0 MASQUERADE&nbsp; tcp&nbsp; --&nbsp; *&nbsp; &nbsp; &nbsp; *&nbsp; &nbsp; &nbsp; &nbsp;172.17.0.2&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;172.17.0.2&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;tcp dpt:80

Chain DOCKER (2 references)
&nbsp;pkts bytes target&nbsp; &nbsp; &nbsp;prot opt in&nbsp; &nbsp; &nbsp;out&nbsp; &nbsp; &nbsp;source&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;destination
&nbsp; &nbsp; 0&nbsp; &nbsp; &nbsp;0 RETURN&nbsp; &nbsp; &nbsp;all&nbsp; --&nbsp; docker0 *&nbsp; &nbsp; &nbsp; &nbsp;0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0.0.0.0/0
&nbsp; &nbsp; 0&nbsp; &nbsp; &nbsp;0 DNAT&nbsp; &nbsp; &nbsp; &nbsp;tcp&nbsp; --&nbsp; !docker0 *&nbsp; &nbsp; &nbsp; &nbsp;0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0.0.0.0/0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; tcp dpt:8080 to:172.17.0.2:80
</code></pre>
<p data-nodeid="74617">通过最后一行规则我们可以得知，当我们访问主机的 8080 端口时，iptables 会把流量转发到 172.17.0.2 的 80 端口，从而实现了我们从主机上可以直接访问到容器内的业务。</p>
<p data-nodeid="74618">我们通过 curl 命令访问一下 nginx 容器：</p>
<pre class="lang-shell" data-nodeid="74619"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> curl http://localhost:8080</span>
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Welcome to nginx!&lt;/title&gt;
&lt;style&gt;
&nbsp; &nbsp; body {
&nbsp; &nbsp; &nbsp; &nbsp; width: 35em;
&nbsp; &nbsp; &nbsp; &nbsp; margin: 0 auto;
&nbsp; &nbsp; &nbsp; &nbsp; font-family: Tahoma, Verdana, Arial, sans-serif;
&nbsp; &nbsp; }
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;Welcome to nginx!&lt;/h1&gt;
&lt;p&gt;If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.&lt;/p&gt;

&lt;p&gt;For online documentation and support please refer to
&lt;a href="http://nginx.org/"&gt;nginx.org&lt;/a&gt;.&lt;br/&gt;
Commercial support is available at
&lt;a href="http://nginx.com/"&gt;nginx.com&lt;/a&gt;.&lt;/p&gt;

&lt;p&gt;&lt;em&gt;Thank you for using nginx.&lt;/em&gt;&lt;/p&gt;
&lt;/body&gt;
&lt;/html&gt;
</code></pre>
<p data-nodeid="74620">通过上面的输出可以得知我们已经成功访问到了 nginx 容器。</p>
<p data-nodeid="74621">总体来说，docker 是官方实现的标准客户端，dockerd 是 Docker 服务端的入口，负责接收客户端发送的指令并返回相应结果，而 docker-init 在业务主进程没有进程回收功能时则十分有用，docker-proxy 组件则是实现 Docker 网络访问的重要组件。</p>
<p data-nodeid="74622">了解完 docker 相关的组件，下面我来介绍下 containerd 相关的组件。</p>
<h4 data-nodeid="74623">containerd 相关的组件</h4>
<p data-nodeid="74624"><strong data-nodeid="74729">（1）containerd</strong></p>
<p data-nodeid="74625"><a href="https://github.com/containerd/containerd" data-nodeid="74732">containerd</a> 组件是从 Docker 1.11 版本正式从 dockerd 中剥离出来的，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。containerd 完全遵循了 OCI 标准，并且是完全社区化运营的，因此被容器界广泛采用。</p>
<p data-nodeid="74626">containerd 不仅负责容器生命周期的管理，同时还负责一些其他的功能：</p>
<ul data-nodeid="74627">
<li data-nodeid="74628">
<p data-nodeid="74629">镜像的管理，例如容器运行前从镜像仓库拉取镜像到本地；</p>
</li>
<li data-nodeid="74630">
<p data-nodeid="74631">接收 dockerd 的请求，通过适当的参数调用 runc 启动容器；</p>
</li>
<li data-nodeid="74632">
<p data-nodeid="74633">管理存储相关资源；</p>
</li>
<li data-nodeid="74634">
<p data-nodeid="74635">管理网络相关资源。</p>
</li>
</ul>
<p data-nodeid="74636">containerd 包含一个后台常驻进程，默认的 socket 路径为 /run/containerd/containerd.sock，dockerd 通过 UNIX 套接字向 containerd 发送请求，containerd 接收到请求后负责执行相关的动作并把执行结果返回给 dockerd。</p>
<p data-nodeid="74637">如果你不想使用 dockerd，也可以直接使用 containerd 来管理容器，由于 containerd 更加简单和轻量，生产环境中越来越多的人开始直接使用 containerd 来管理容器。</p>
<p data-nodeid="74638"><strong data-nodeid="74744">（2）containerd-shim</strong></p>
<p data-nodeid="74639">containerd-shim 的意思是垫片，类似于拧螺丝时夹在螺丝和螺母之间的垫片。containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd 不影响已经启动的容器进程。</p>
<p data-nodeid="74640"><strong data-nodeid="74749">（3）ctr</strong></p>
<p data-nodeid="74641">ctr 实际上是 containerd-ctr，它是 containerd 的客户端，主要用来开发和调试，在没有 dockerd 的环境中，ctr 可以充当 docker 客户端的部分角色，直接向 containerd 守护进程发送操作容器的请求。</p>
<p data-nodeid="74642">了解完 containerd 相关的组件，我们来了解一下容器的真正运行时 runc。</p>
<h4 data-nodeid="74643">容器运行时组件runc</h4>
<p data-nodeid="74644">runc 是一个标准的 OCI 容器运行时的实现，它是一个命令行工具，可以直接用来创建和运行容器。</p>
<p data-nodeid="74645">下面我们通过一个实例来演示一下 runc 的神奇之处。</p>
<p data-nodeid="74646">第一步，准备容器运行时文件：进入 /home/centos 目录下，创建 runc 文件夹，并导入 busybox 镜像文件。</p>
<pre class="lang-shell" data-nodeid="74647"><code data-language="shell"><span class="hljs-meta"> $</span><span class="bash"> <span class="hljs-built_in">cd</span> /home/centos</span>
<span class="hljs-meta"> #</span><span class="bash"><span class="hljs-comment"># 创建 runc 运行根目录</span></span>
<span class="hljs-meta"> $</span><span class="bash"> mkdir runc</span>
<span class="hljs-meta"> #</span><span class="bash"><span class="hljs-comment"># 导入 rootfs 镜像文件</span></span>
<span class="hljs-meta"> $</span><span class="bash"> mkdir rootfs &amp;&amp; docker <span class="hljs-built_in">export</span> $(docker create busybox) | tar -C rootfs -xvf -</span>
</code></pre>
<p data-nodeid="74648">第二步，生成 runc config 文件。我们可以使用 runc spec 命令根据文件系统生成对应的 config.json 文件。命令如下：</p>
<pre class="lang-shell" data-nodeid="74649"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> runc spec</span>
</code></pre>
<p data-nodeid="74650">此时会在当前目录下生成 config.json 文件，我们可以使用 cat 命令查看一下 config.json 的内容：</p>
<pre class="lang-shell" data-nodeid="74651"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat config.json</span>
{
	"ociVersion": "1.0.1-dev",
	"process": {
		"terminal": true,
		"user": {
			"uid": 0,
			"gid": 0
		},
		"args": [
			"sh"
		],
		"env": [
			"PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
			"TERM=xterm"
		],
		"cwd": "/",
		"capabilities": {
			"bounding": [
				"CAP_AUDIT_WRITE",
				"CAP_KILL",
				"CAP_NET_BIND_SERVICE"
			],
			"effective": [
				"CAP_AUDIT_WRITE",
				"CAP_KILL",
				"CAP_NET_BIND_SERVICE"
			],
			"inheritable": [
				"CAP_AUDIT_WRITE",
				"CAP_KILL",
				"CAP_NET_BIND_SERVICE"
			],
			"permitted": [
				"CAP_AUDIT_WRITE",
				"CAP_KILL",
				"CAP_NET_BIND_SERVICE"
			],
			"ambient": [
				"CAP_AUDIT_WRITE",
				"CAP_KILL",
				"CAP_NET_BIND_SERVICE"
			]
		},
		"rlimits": [
			{
				"type": "RLIMIT_NOFILE",
				"hard": 1024,
				"soft": 1024
			}
		],
		"noNewPrivileges": true
	},
	"root": {
		"path": "rootfs",
		"readonly": true
	},
	"hostname": "runc",
	"mounts": [
		{
			"destination": "/proc",
			"type": "proc",
			"source": "proc"
		},
		{
			"destination": "/dev",
			"type": "tmpfs",
			"source": "tmpfs",
			"options": [
				"nosuid",
				"strictatime",
				"mode=755",
				"size=65536k"
			]
		},
		{
			"destination": "/dev/pts",
			"type": "devpts",
			"source": "devpts",
			"options": [
				"nosuid",
				"noexec",
				"newinstance",
				"ptmxmode=0666",
				"mode=0620",
				"gid=5"
			]
		},
		{
			"destination": "/dev/shm",
			"type": "tmpfs",
			"source": "shm",
			"options": [
				"nosuid",
				"noexec",
				"nodev",
				"mode=1777",
				"size=65536k"
			]
		},
		{
			"destination": "/dev/mqueue",
			"type": "mqueue",
			"source": "mqueue",
			"options": [
				"nosuid",
				"noexec",
				"nodev"
			]
		},
		{
			"destination": "/sys",
			"type": "sysfs",
			"source": "sysfs",
			"options": [
				"nosuid",
				"noexec",
				"nodev",
				"ro"
			]
		},
		{
			"destination": "/sys/fs/cgroup",
			"type": "cgroup",
			"source": "cgroup",
			"options": [
				"nosuid",
				"noexec",
				"nodev",
				"relatime",
				"ro"
			]
		}
	],
	"linux": {
		"resources": {
			"devices": [
				{
					"allow": false,
					"access": "rwm"
				}
			]
		},
		"namespaces": [
			{
				"type": "pid"
			},
			{
				"type": "network"
			},
			{
				"type": "ipc"
			},
			{
				"type": "uts"
			},
			{
				"type": "mount"
			}
		],
		"maskedPaths": [
			"/proc/acpi",
			"/proc/asound",
			"/proc/kcore",
			"/proc/keys",
			"/proc/latency_stats",
			"/proc/timer_list",
			"/proc/timer_stats",
			"/proc/sched_debug",
			"/sys/firmware",
			"/proc/scsi"
		],
		"readonlyPaths": [
			"/proc/bus",
			"/proc/fs",
			"/proc/irq",
			"/proc/sys",
			"/proc/sysrq-trigger"
		]
	}
}
</code></pre>
<p data-nodeid="74652">config.json 文件定义了 runc 启动容器时的一些配置，如根目录的路径，文件挂载路径等配置。<br>
第三步，使用 runc 启动容器。我们可以使用 runc run 命令直接启动 busybox 容器。</p>
<pre class="lang-shell" data-nodeid="74653"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> runc run busybox</span>
/ #
</code></pre>
<p data-nodeid="74654">此时，我们已经创建并启动了一个 busybox 容器。</p>
<p data-nodeid="74655">我们新打开一个命令行窗口，可以使用 run list 命令看到刚才启动的容器。</p>
<pre class="lang-shell" data-nodeid="74656"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> <span class="hljs-built_in">cd</span> /home/centos/runc/</span>
<span class="hljs-meta">$</span><span class="bash"> runc list</span>
D&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; PID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; BUNDLE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; CREATED&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; OWNER
busybox&nbsp; &nbsp; &nbsp;9778&nbsp; &nbsp; &nbsp; &nbsp; running&nbsp; &nbsp; &nbsp;/home/centos/runc&nbsp; &nbsp;2020-09-06T09:25:32.441957273Z&nbsp; &nbsp;root
</code></pre>
<p data-nodeid="74657">通过上面的输出，我们可以看到，当前已经有一个 busybox 容器处于运行状态。</p>
<p data-nodeid="74658">总体来说，Docker 的组件虽然很多，但每个组件都有自己清晰的工作职责，Docker 相关的组件负责发送和接受 Docker 请求，contianerd 相关的组件负责管理容器的生命周期，而 runc 负责真正意义上创建和启动容器。这些组件相互配合，才使得 Docker 顺利完成了容器的管理工作。</p>
<h3 data-nodeid="74659">总结</h3>
<p data-nodeid="74660">到此，相信你已经完全掌握了 Docker 的组件构成，各个组件的作用和工作原理。本节课时的重点我帮你总结如下。</p>
<p data-nodeid="74661"><img src="https://s0.lgstatic.com/i/image/M00/59/E6/Ciqc1F9y4vGAVzmAAADk1nlHpUA424.png" alt="7.png" data-nodeid="74769"></p>
<p data-nodeid="74662" class="">那么，你知道 Docker 当前的架构有什么弊端吗？思考后，把你的想法写在留言区。</p>

---

### 精选评论

##### **波：
> 如果containerd 挂了，整个不能使用了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当前 docker 的问题就是调用链过长，出现问题需要分析很多组件才能定位问题。

##### *李：
> 抽象分层过分的话调用返回势必带来性能的损耗 23333

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，任何事物都有两面性。抽象分层就需要写时复制，性能是有一定损耗的

##### **听：
> 大佬 ，拉取的redis 启动后 外面能连接 但是一执行命令 就报 这个错误“Error: Connection reset by peer” 怎么解？[root@localhost ~]# /usr/local/redis/bin/redis-cli -h localhost -p 32781localhost:32781 keys *Error: Connection reset by peer

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; redis 版本以及启动配置是什么呢？

##### **平：
> 请问下老师，像php-fpm这种主进程fork子进程的这种工作模式，当子进程oom的时候，容器并不会被kill，导致报错，类似于Python也是这种工作模式，这种怎么解决呢？谢谢老师，望老师回复

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以通过健康检查的方式，子进程oom后，健康检查失败后重启容器

##### *波：
> 但是看到老师写到docker组件的时候，为什么没有包含Driver,libContainer以及Registery等

##### **1670：
> 老师，我在运行runc run busybox 的时候会显示ERRO[0000] rootless container requires user namespaces，启动不起来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请使用 root 用户运行，应该可以解决这个报错

