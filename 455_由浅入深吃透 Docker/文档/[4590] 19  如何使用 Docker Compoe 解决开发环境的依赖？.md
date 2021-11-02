<p data-nodeid="232500">前两个模块，我们从 Docker 的基本操作到 Docker 的实现原理，为你一步一步揭开了 Docker 神秘的面纱。然而目前为止，我们所有的操作都是围绕单个容器进行的，但当我们的业务越来越复杂时，需要多个容器相互配合，甚至需要多个主机组成容器集群才能满足我们的业务需求，这个时候就需要用到容器的编排工具了。因为容器编排工具可以帮助我们批量地创建、调度和管理容器，帮助我们解决规模化容器的部署问题。</p>



<p data-nodeid="231636">从这一课时开始，我将向你介绍 Docker 三种常用的编排工具：Docker Compose、Docker Swarm 和 Kubernetes。了解这些编排工具，可以让你在不同的环境中选择最优的编排框架。</p>
<p data-nodeid="231637">本课时我们先来学习一个在开发时经常用到的编排工具——Docker Compose。合理地使用 Docker Compose 可以极大地帮助我们提升开发效率。那么 Docker Compose 究竟是什么呢？</p>
<h3 data-nodeid="231638">Docker Compose 的前世今生</h3>
<p data-nodeid="231639">Docker Compose 的前身是 Orchard 公司开发的 Fig，2014 年 Docker 收购了 Orchard 公司，然后将 Fig 重命名为 Docker Compose。现阶段 Docker Compose 是 Docker 官方的单机多容器管理系统，它本质是一个 Python 脚本，它通过解析用户编写的 yaml 文件，调用 Docker API 实现动态的创建和管理多个容器。</p>
<p data-nodeid="231640">要想使用 Docker Compose，需要我们先安装一个 Docker Compose。</p>
<h3 data-nodeid="231641">安装 Docker Compose</h3>
<p data-nodeid="231642">Docker Compose 可以安装在 macOS、 Windows 和 Linux 系统中，其中在 macOS 和 Windows 系统下 ，Docker Compose 都是随着 Docker 的安装一起安装好的，这里就不再详细介绍。 下面我重点介绍下如何在 Linux 系统下安装 Docker Compose。</p>
<h4 data-nodeid="231643">Linux 系统下安装 Docker Compose</h4>
<p data-nodeid="231644">在安装  Docker Compose 之前，请确保你的机器已经正确运行了 Docker，如果你的机器还没有安装 Docker，请参考<a href="https://docs.docker.com/engine/install/" data-nodeid="231763">官方网站</a>安装 Docker。</p>
<p data-nodeid="231645">要在 Linux 平台上安装  Docker Compose，我们需要到 Compose 的 Github 页面下载对应版本的安装包。这里我以 1.27.3 版本为例，带你安装一个 Docker Compose。</p>
<p data-nodeid="231646">（1）使用 curl 命令（一种发送 http 请求的命令行工具）下载&nbsp;Docker Compose 的安装包：</p>
<pre class="lang-shell" data-nodeid="231647"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo curl -L <span class="hljs-string">"https://github.com/docker/compose/releases/download/1.27.3/docker-compose-<span class="hljs-subst">$(uname -s)</span>-<span class="hljs-subst">$(uname -m)</span>"</span> -o /usr/<span class="hljs-built_in">local</span>/bin/docker-compose</span>
</code></pre>
<blockquote data-nodeid="231648">
<p data-nodeid="231649">如果你想要安装其他版本的 Docker Compose，将 1.27.3 替换为你想要安装的版本即可。</p>
</blockquote>
<p data-nodeid="231650">（2）修改  Docker Compose 执行权限：</p>
<pre class="lang-shell" data-nodeid="231651"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo chmod +x /usr/<span class="hljs-built_in">local</span>/bin/docker-compose</span>
</code></pre>
<p data-nodeid="231652">（3）检查  Docker Compose 是否安装成功：</p>
<pre class="lang-plain" data-nodeid="231653"><code data-language="plain">$ docker-compose --version
docker-compose version 1.27.3, build 1110ad01
</code></pre>
<p data-nodeid="233072">当我们执行完上述命令后，如果 Docker Compose 输出了当前版本号，就表示我们的 Docker Compose 已经安装成功。 Docker Compose 安装成功后，我们就可以很方便地使用它了。</p>
<p data-nodeid="233073">在使用  Docker Compose 之前，我们首先需要先编写 Docker Compose 模板文件，因为 Docker Compose 运行的时候是根据  Docker Compose 模板文件中的定义来运行的。</p>

<p data-nodeid="231655">下面我们首先来学习一下如何编写一个  Docker Compose 模板文件。</p>
<h3 data-nodeid="231656">编写 Docker Compose 模板文件</h3>
<p data-nodeid="231657">在使用 Docker Compose 启动容器时， Docker Compose 会默认使用 docker-compose.yml 文件， docker-compose.yml 文件的格式为 yaml（类似于 json，一种标记语言）。</p>
<p data-nodeid="231658">Docker Compose 模板文件一共有三个版本： v1、v2 和 v3。目前最新的版本为 v3，也是功能最全面的一个版本，下面我主要围绕 v3 版本介绍一下如何编写 Docker Compose 文件。</p>
<p data-nodeid="231659">Docker Compose 文件主要分为三部分： services（服务）、networks（网络） 和 volumes（数据卷）。</p>
<ul data-nodeid="231660">
<li data-nodeid="231661">
<p data-nodeid="231662">services（服务）：服务定义了容器启动的各项配置，就像我们执行<code data-backticks="1" data-nodeid="231779">docker run</code>命令时传递的容器启动的参数一样，指定了容器应该如何启动，例如容器的启动参数，容器的镜像和环境变量等。</p>
</li>
<li data-nodeid="231663">
<p data-nodeid="231664">networks（网络）：网络定义了容器的网络配置，就像我们执行<code data-backticks="1" data-nodeid="231782">docker network create</code>命令创建网络配置一样。</p>
</li>
<li data-nodeid="231665">
<p data-nodeid="231666">volumes（数据卷）：数据卷定义了容器的卷配置，就像我们执行<code data-backticks="1" data-nodeid="231785">docker volume create</code>命令创建数据卷一样。</p>
</li>
</ul>
<p data-nodeid="231667">一个典型的 Docker Compose 文件结构如下：</p>
<pre class="lang-yaml" data-nodeid="231668"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">nginx:</span>
    <span class="hljs-comment">## ... 省略部分配置</span>
<span class="hljs-attr">networks:</span>
  <span class="hljs-attr">frontend:</span>
  <span class="hljs-attr">backend:</span>
<span class="hljs-attr">volumes:</span>
  <span class="hljs-attr">db-data:</span>
</code></pre>
<p data-nodeid="231669">下面我们首先来学习一下如何编写 services 部分的配置。</p>
<h4 data-nodeid="231670">编写 Service 配置</h4>
<p data-nodeid="231671">services 下，首先需要定义服务名称，例如你这个服务是 nginx 服务，你可以定义 service 名称为 nginx，格式如下：</p>
<pre class="lang-yaml" data-nodeid="231672"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3.8"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">nginx:</span> 
</code></pre>
<p data-nodeid="234228">服务名称定义完毕后，我们需要在服务名称的下一级定义当前服务的各项配置，使得我们的服务可以按照配置正常启动。常用的 16 种 service 配置如下。如果你比较了解，可以直接跳过看 Volume 配置和后续实操即可。</p>
<p data-nodeid="234229"><strong data-nodeid="234237">build：</strong> 用于构建 Docker 镜像，类似于<code data-backticks="1" data-nodeid="234235">docker build</code>命令，build 可以指定 Dockerfile 文件路径，然后根据 Dockerfile 命令来构建文件。使用方法如下：</p>


<pre class="lang-yaml" data-nodeid="231674"><code data-language="yaml"><span class="hljs-attr">build:</span>
  <span class="hljs-comment">## 构建执行的上下文目录</span>
  <span class="hljs-attr">context:</span> <span class="hljs-string">.</span>
  <span class="hljs-comment">## Dockerfile 名称</span>
  <span class="hljs-attr">dockerfile:</span> <span class="hljs-string">Dockerfile-name</span>
</code></pre>
<p data-nodeid="234810" class=""><strong data-nodeid="234892">cap_add、cap_drop：</strong> 指定容器可以使用到哪些内核能力（capabilities）。使用格式如下：</p>
<pre class="lang-yaml" data-nodeid="234811"><code data-language="yaml"><span class="hljs-attr">cap_add:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">NET_ADMIN</span>
<span class="hljs-attr">cap_drop:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">SYS_ADMIN</span>
</code></pre>
<p data-nodeid="235582" class=""><strong data-nodeid="235658">command：</strong> 用于覆盖容器默认的启动命令，它和 Dockerfile 中的 CMD 用法类似，也有两种使用方式：</p>
<pre class="lang-yaml" data-nodeid="235583"><code data-language="yaml"><span class="hljs-attr">command:</span> <span class="hljs-string">sleep</span> <span class="hljs-number">3000</span>
</code></pre>
<pre data-nodeid="235584"><code>command: ["sleep", "3000"]
</code></pre>
<p data-nodeid="236346" class=""><strong data-nodeid="236421">container_name：</strong> 用于指定容器启动时容器的名称。使用格式如下：</p>
<pre class="lang-yaml" data-nodeid="236347"><code data-language="yaml"><span class="hljs-attr">container_name:</span> <span class="hljs-string">nginx</span>
</code></pre>
<p data-nodeid="237105" class=""><strong data-nodeid="237180">depends_on：</strong> 用于指定服务间的依赖关系，这样可以先启动被依赖的服务。例如，我们的服务依赖数据库服务 db，可以指定 depends_on 为 db。使用格式如下：</p>
<pre class="lang-yaml" data-nodeid="237106"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3.8"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">my-web:</span>
    <span class="hljs-attr">build:</span> <span class="hljs-string">.</span>
    <span class="hljs-attr">depends_on:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-string">db</span>
  <span class="hljs-attr">db:</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">mysql</span>
</code></pre>
<p data-nodeid="237858" class=""><strong data-nodeid="237927">devices：</strong> 挂载主机的设备到容器中。使用格式如下：</p>
<pre class="lang-yaml" data-nodeid="237859"><code data-language="yaml"><span class="hljs-attr">devices:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"/dev/sba:/dev/sda"</span>
</code></pre>
<p data-nodeid="238603" class=""><strong data-nodeid="238670">dns：</strong> 自定义容器中的 dns 配置。</p>
<pre class="lang-yaml" data-nodeid="238604"><code data-language="yaml"><span class="hljs-attr">dns:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-number">8.8</span><span class="hljs-number">.8</span><span class="hljs-number">.8</span>
  <span class="hljs-bullet">-</span> <span class="hljs-number">114.114</span><span class="hljs-number">.114</span><span class="hljs-number">.114</span>
</code></pre>
<p data-nodeid="239344" class=""><strong data-nodeid="239411">dns_search：</strong> 配置 dns 的搜索域。</p>
<pre class="lang-yaml" data-nodeid="239345"><code data-language="yaml"><span class="hljs-attr">dns_search:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">svc.cluster.com</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">svc1.cluster.com</span>
</code></pre>
<p data-nodeid="240081" class=""><strong data-nodeid="240144">entrypoint：</strong> 覆盖容器的 entrypoint 命令。</p>
<pre class="lang-yaml" data-nodeid="240082"><code data-language="yaml"><span class="hljs-attr">entrypoint:</span> <span class="hljs-string">sleep</span> <span class="hljs-number">3000</span>
</code></pre>
<p data-nodeid="240083">或</p>
<pre class="lang-yaml" data-nodeid="240084"><code data-language="yaml"><span class="hljs-attr">entrypoint:</span> [<span class="hljs-string">"sleep"</span>, <span class="hljs-string">"3000"</span>]
</code></pre>
<p data-nodeid="240812" class=""><strong data-nodeid="240873">env_file：</strong> 指定容器的环境变量文件，启动时会把该文件中的环境变量值注入容器中。</p>
<pre class="lang-yaml" data-nodeid="240813"><code data-language="yaml"><span class="hljs-attr">env_file:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">./dbs.env</span>
</code></pre>
<p data-nodeid="240814">env 文件的内容格式如下：</p>
<pre class="lang-yaml" data-nodeid="240815"><code data-language="yaml"><span class="hljs-string">KEY_ENV=values</span>
</code></pre>
<p data-nodeid="241536" class=""><strong data-nodeid="241591">environment：</strong> 指定容器启动时的环境变量。</p>
<pre class="lang-yaml" data-nodeid="241537"><code data-language="yaml"><span class="hljs-attr">environment:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">KEY_ENV=values</span>
</code></pre>
<p data-nodeid="242251" class=""><strong data-nodeid="242304">image：</strong> 指定容器镜像的地址。</p>
<pre class="lang-yaml" data-nodeid="242252"><code data-language="yaml"><span class="hljs-attr">image:</span> <span class="hljs-string">busybox:latest</span>
</code></pre>
<p data-nodeid="242962" class=""><strong data-nodeid="243013">pid：</strong> 共享主机的进程命名空间，像在主机上直接启动进程一样，可以看到主机的进程信息。</p>
<pre class="lang-yaml" data-nodeid="242963"><code data-language="yaml"><span class="hljs-attr">pid:</span> <span class="hljs-string">"host"</span>
</code></pre>
<p data-nodeid="243669" class=""><strong data-nodeid="243718">ports：</strong> 暴露端口信息，使用格式为 HOST:CONTAINER，前面填写要映射到主机上的端口，后面填写对应的容器内的端口。</p>
<pre class="lang-yaml" data-nodeid="243670"><code data-language="yaml"><span class="hljs-attr">ports:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"1000"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"1000-1005"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"8080:8080"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"8888-8890:8888-8890"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"2222:22"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"127.0.0.1:9999:9999"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"127.0.0.1:3000-3005:3000-3005"</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">"6789:6789/udp"</span>
</code></pre>
<p data-nodeid="244372" class=""><strong data-nodeid="244419">networks：</strong> 这是服务要使用的网络名称，对应顶级的 networks 中的配置。</p>
<pre class="lang-yaml" data-nodeid="244373"><code data-language="yaml"><span class="hljs-attr">services:</span>
  <span class="hljs-attr">my-service:</span>
    <span class="hljs-attr">networks:</span>
     <span class="hljs-bullet">-</span> <span class="hljs-string">hello-network</span>
     <span class="hljs-bullet">-</span> <span class="hljs-string">hello1-network</span>
</code></pre>
<p data-nodeid="245071" class=""><strong data-nodeid="245118">volumes：</strong> 不仅可以挂载主机数据卷到容器中，也可以直接挂载主机的目录到容器中，使用方式类似于使用<code data-backticks="1" data-nodeid="245116">docker run</code>启动容器时添加 -v 参数。</p>
<pre class="lang-yaml" data-nodeid="245072"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">db:</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">mysql:5.6</span>
    <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">type:</span> <span class="hljs-string">volume</span>
        <span class="hljs-attr">source:</span> <span class="hljs-string">/var/lib/mysql</span>
        <span class="hljs-attr">target:</span> <span class="hljs-string">/var/lib/mysql</span>
</code></pre>
<p data-nodeid="245073">volumes 除了上面介绍的长语法外，还支持短语法的书写方式，例如上面的写法可以精简为：</p>
<pre class="lang-yaml" data-nodeid="245074"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">db:</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">mysql:5.6</span>
    <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-string">/var/lib/mysql:/var/lib/mysql</span>
</code></pre>
<h4 data-nodeid="245075">编写 Volume 配置</h4>
<p data-nodeid="245076">如果你想在多个容器间共享数据卷，则需要在外部声明数据卷，然后在容器里声明使用数据卷。例如我想在两个服务间共享日志目录，则使用以下配置：</p>
<pre class="lang-yaml" data-nodeid="245077"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">my-service1:</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">service:v1</span>
    <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">type:</span> <span class="hljs-string">volume</span>
        <span class="hljs-attr">source:</span> <span class="hljs-string">logdata</span>
        <span class="hljs-attr">target:</span> <span class="hljs-string">/var/log/mylog</span>
  <span class="hljs-attr">my-service2:</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">service:v2</span>
    <span class="hljs-attr">volumes:</span>
      <span class="hljs-bullet">-</span> <span class="hljs-attr">type:</span> <span class="hljs-string">volume</span>
        <span class="hljs-attr">source:</span> <span class="hljs-string">logdata</span>
        <span class="hljs-attr">target:</span> <span class="hljs-string">/var/log/mylog</span>
<span class="hljs-attr">volumes:</span>
  <span class="hljs-attr">logdata:</span>
 
</code></pre>
<h4 data-nodeid="245078">编写 Network 配置</h4>
<p data-nodeid="245079">Docker Compose 文件顶级声明的 networks 允许你创建自定义的网络，类似于<code data-backticks="1" data-nodeid="245124">docker network create</code>命令。</p>
<p data-nodeid="245080">例如你想声明一个自定义 bridge 网络配置，并且在服务中使用它，使用格式如下：</p>
<pre class="lang-yaml" data-nodeid="245081"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">"3"</span>
<span class="hljs-attr">services:</span>
  <span class="hljs-attr">web:</span>
    <span class="hljs-attr">networks:</span>
      <span class="hljs-attr">mybridge:</span> 
        <span class="hljs-attr">ipv4_address:</span> <span class="hljs-number">172.16</span><span class="hljs-number">.1</span><span class="hljs-number">.11</span>
<span class="hljs-attr">networks:</span>
  <span class="hljs-attr">mybridge:</span>
    <span class="hljs-attr">driver:</span> <span class="hljs-string">bridge</span>
    <span class="hljs-attr">ipam:</span> 
      <span class="hljs-attr">driver:</span> <span class="hljs-string">default</span>
      <span class="hljs-attr">config:</span>
        <span class="hljs-attr">subnet:</span> <span class="hljs-number">172.16</span><span class="hljs-number">.1</span><span class="hljs-number">.0</span><span class="hljs-string">/24</span>
      
</code></pre>
<p data-nodeid="245082">编写完 Docker Compose 模板文件后，需要使用 docker-compose 命令来运行这些文件。下面我们来学习下 docker-compose 都有哪些操作命令。</p>
<h3 data-nodeid="245083">Docker Compose 操作命令</h3>
<p data-nodeid="245084">我们可以使用<code data-backticks="1" data-nodeid="245130">docker-compose -h</code>命令来查看 docker-compose 的用法，docker-compose 的基本使用格式如下：</p>
<pre class="lang-dart" data-nodeid="247749"><code data-language="dart">docker-compose [-f &lt;arg&gt;...] [options] [--] [COMMAND] [ARGS...]
</code></pre>
<p data-nodeid="247750">其中 options 是 docker-compose 的参数，支持的参数和功能说明如下：</p>
<pre class="lang-java" data-nodeid="270124"><code data-language="java">&nbsp; -f, --file FILE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;指定 docker-compose 文件，默认为 docker-compose.yml
&nbsp; -p, --project-name NAME&nbsp; &nbsp; &nbsp;指定项目名称，默认使用当前目录名称作为项目名称
&nbsp; --verbose&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;输出调试信息
&nbsp; --log-level LEVEL&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;日志级别 (DEBUG, INFO, WARNING, ERROR, CRITICAL)
&nbsp; -v, --version&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;输出当前版本并退出
&nbsp; -H, --host HOST&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;指定要连接的 Docker 地址
&nbsp; --tls&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;启用 TLS 认证
&nbsp; --tlscacert CA_PATH&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;TLS CA 证书路径
&nbsp; --tlscert CLIENT_CERT_PATH&nbsp; TLS 公钥证书问价
&nbsp; --tlskey TLS_KEY_PATH&nbsp; &nbsp; &nbsp; &nbsp;TLS 私钥证书文件
&nbsp; --tlsverify&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;使用 TLS 校验对端
&nbsp; --skip-hostname-check&nbsp; &nbsp; &nbsp; &nbsp;不校验主机名
&nbsp; --project-directory PATH&nbsp; &nbsp; 指定工作目录，默认是 Compose 文件所在路径。
</code></pre>
<p data-nodeid="270125">COMMAND 为 docker-compose 支持的命令。支持的命令如下：</p>
<pre class="lang-dart" data-nodeid="279297"><code data-language="dart">&nbsp; build&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 构建服务
&nbsp; config&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;校验和查看 Compose 文件
&nbsp; create&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;创建服务
&nbsp; down&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;停止服务，并且删除相关资源
&nbsp; events&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;实时监控容器的时间信息
&nbsp; exec&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;在一个运行的容器中运行指定命令
&nbsp; help&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;获取帮助
&nbsp; images&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;列出镜像
&nbsp; kill&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;杀死容器
&nbsp; logs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;查看容器输出
&nbsp; pause&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 暂停容器
&nbsp; port&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;打印容器端口所映射出的公共端口
&nbsp; ps&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;列出项目中的容器列表
&nbsp; pull&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;拉取服务中的所有镜像
&nbsp; push&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;推送服务中的所有镜像
&nbsp; restart&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 重启服务
&nbsp; rm&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;删除项目中已经停止的容器
&nbsp; run&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 在指定服务上运行一个命令
&nbsp; scale&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 设置服务运行的容器个数
&nbsp; start&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 启动服务
&nbsp; stop&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;停止服务
&nbsp; top&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 限制服务中正在运行中的进程信息
&nbsp; unpause&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 恢复暂停的容器
&nbsp; up&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;创建并且启动服务
&nbsp; version&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 打印版本信息并退出
</code></pre>
<p data-nodeid="279298">好了，学习完 Docker Compose 模板的编写和 docker-compose 命令的使用方法，下面我们编写一个 Docker Compose 模板文件，实现一键启动 WordPress 服务（一种博客系统），来搭建一个属于我们自己的博客系统。</p>
<h3 data-nodeid="279299">使用 Docker Compose 管理 WordPress</h3>
<h4 data-nodeid="279300">启动 WordPress</h4>
<p data-nodeid="279301">第一步，创建项目目录。首先我们在 /tmp 目录下创建一个 WordPress 的目录，这个目录将作为我们的工作目录。</p>
<pre class="lang-plain" data-nodeid="279302"><code data-language="plain">$ mkdir /tmp/wordpress
</code></pre>
<p data-nodeid="279303">第二步，进入工作目录。</p>
<pre class="lang-plain" data-nodeid="279304"><code data-language="plain">$ cd /tmp/wordpress
</code></pre>
<p data-nodeid="279305">第三步，创建 docker-compose.yml 文件。</p>
<pre class="lang-plain" data-nodeid="279306"><code data-language="plain">$ touch docker-compose.yml
</code></pre>
<p data-nodeid="279307">然后写入以下内容：</p>
<pre class="lang-yaml" data-nodeid="279308"><code data-language="yaml"><span class="hljs-attr">version:</span> <span class="hljs-string">'3'</span>
<span class="hljs-attr">services:</span>
   <span class="hljs-attr">mysql:</span>
     <span class="hljs-attr">image:</span> <span class="hljs-string">mysql:5.7</span>
     <span class="hljs-attr">volumes:</span>
       <span class="hljs-bullet">-</span> <span class="hljs-string">mysql_data:/var/lib/mysql</span>
     <span class="hljs-attr">restart:</span> <span class="hljs-string">always</span>
     <span class="hljs-attr">environment:</span>
       <span class="hljs-attr">MYSQL_ROOT_PASSWORD:</span> <span class="hljs-string">root</span>
       <span class="hljs-attr">MYSQL_DATABASE:</span> <span class="hljs-string">mywordpress</span>
       <span class="hljs-attr">MYSQL_USER:</span> <span class="hljs-string">mywordpress</span>
       <span class="hljs-attr">MYSQL_PASSWORD:</span> <span class="hljs-string">mywordpress</span>
   <span class="hljs-attr">wordpress:</span>
     <span class="hljs-attr">depends_on:</span>
       <span class="hljs-bullet">-</span> <span class="hljs-string">mysql</span>
     <span class="hljs-attr">image:</span> <span class="hljs-string">wordpress:php7.4</span>
     <span class="hljs-attr">ports:</span>
       <span class="hljs-bullet">-</span> <span class="hljs-string">"8080:80"</span>
     <span class="hljs-attr">restart:</span> <span class="hljs-string">always</span>
     <span class="hljs-attr">environment:</span>
       <span class="hljs-attr">WORDPRESS_DB_HOST:</span> <span class="hljs-string">mysql:3306</span>
       <span class="hljs-attr">WORDPRESS_DB_USER:</span> <span class="hljs-string">mywordpress</span>
       <span class="hljs-attr">WORDPRESS_DB_PASSWORD:</span> <span class="hljs-string">mywordpress</span>
       <span class="hljs-attr">WORDPRESS_DB_NAME:</span> <span class="hljs-string">mywordpress</span>
<span class="hljs-attr">volumes:</span>
    <span class="hljs-attr">mysql_data:</span> {}
</code></pre>
<p data-nodeid="279309">第四步，启动 MySQL 数据库和 WordPress 服务。</p>
<pre class="lang-plain" data-nodeid="279310"><code data-language="plain">$ docker-compose up -d
Starting wordpress_mysql_1 ... done
Starting wordpress_wordpress_1 ... done
</code></pre>
<p data-nodeid="281778">执行完以上命令后，Docker Compose 首先会为我们启动一个 MySQL 数据库，按照 MySQL 服务中声明的环境变量来设置 MySQL 数据库的用户名和密码。然后等待 MySQL 数据库启动后，再启动 WordPress 服务。WordPress 服务启动后，我们就可以通过 <a href="http://localhost:8080" data-nodeid="281783">http://localhost:8080</a> 访问它了，访问成功后，我们就可以看到以下界面，然后按照提示一步一步设置就可以拥有属于自己的专属博客系统了。</p>
<p data-nodeid="282392"><img src="https://s0.lgstatic.com/i/image/M00/63/AF/Ciqc1F-WuHqAMc6gAAGSco9Zyvc339.png" alt="image.png" data-nodeid="282396"></p>
<div data-nodeid="282393" class=""><p style="text-align:center">图 1 WordPress 启动界面</p></div>







<h4 data-nodeid="279313">停止 WordPress</h4>
<p data-nodeid="279314">如果你不再需要 WordPress 服务了，可以使用<code data-backticks="1" data-nodeid="279340">docker-compose stop</code>命令来停止已启动的服务。</p>
<pre class="lang-plain" data-nodeid="279315"><code data-language="plain">$ docker-compose stop
Stopping wordpress_wordpress_1 ... done
Stopping wordpress_mysql_1&nbsp; &nbsp; &nbsp;... done
</code></pre>
<h3 data-nodeid="279316">结语</h3>
<p data-nodeid="279317">Docker Compose 是一个用来定义复杂应用的单机编排工具，通常用于服务依赖关系复杂的开发和测试环境，如果你还在为配置复杂的开发环境而烦恼，Docker Compose 可以轻松帮你搞定复杂的开发环境。你只需要把复杂的开发环境使用 Docker Compose 模板文件描述出来，之后无论你在哪里可以轻松的一键启动开发和测试环境，极大地提高了我们的开发效率，同时也避免了污染我们开发机器的配置。</p>
<p data-nodeid="285409" class="">那么，学完本课时的课程，你可以试着使用 Docker Compose 一键启动一个 <a href="https://baike.baidu.com/item/LNMP" data-nodeid="285413">LNMP</a> 开发环境吗？</p>


<p data-nodeid="284201" class="">下一课时，我将为你讲解容器的另一个编排系统 Docker Swarm。</p>

---

### 精选评论

##### *策：
> 老师，我问一下，线上是不是也可以用docker compose呀，还是有更好的解决方案呢😀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 线上单机部署也可以使用 docker-compose，但是 docker-compose 无法做到集群调度，想要把多台服务组成容器集群，推荐使用 kubernetes

##### *冲：
> 在启动WordPress的时候">docker-compose up -d报错/var/lib/docker/tmp/GetImageBlob105292471: no space left on device...最后一脚df -h 查看磁盘空间并没有满

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是/var/lib/docker/ 这个目录满了，请检查

##### *瑞：
> docker compose 在非集群情况下可以控制CPU和内存的使用量吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用  deploy:
      resources:
        limits:
          cpus: '0.001'
          memory: 50M  即可限制 CPU 和内存使用量

##### **升：
> 打卡

