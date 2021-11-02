<p data-nodeid="1781" class="">今天我将围绕 Docker 核心概念镜像展开，首先重点讲解一下镜像的基本操作，然后介绍一下镜像的实现原理。首先说明，咱们本课时的镜像均指 Docker 镜像。</p>
<p data-nodeid="1782">你是否还记得镜像是什么？我们先回顾一下。</p>
<p data-nodeid="1783">镜像是一个只读的 Docker 容器模板，包含启动容器所需要的所有文件系统结构和内容。简单来讲，镜像是一个特殊的文件系统，它提供了容器运行时所需的程序、软件库、资源、配置等静态数据。即<strong data-nodeid="1969">镜像不包含任何动态数据，镜像内容在构建后不会被改变</strong>。</p>
<p data-nodeid="1784">然后我们来看下如何操作镜像。</p>
<h3 data-nodeid="1785">镜像操作</h3>
<p data-nodeid="1786"><img src="https://s0.lgstatic.com/i/image/M00/4A/AD/CgqCHl9SDkWAaxh7AAFaMgWI7cI029.png" alt="Lark20200904-175130.png" data-nodeid="1974"></p>
<div data-nodeid="1787"><p style="text-align:center">图 1 镜像操作</p></div>
<p data-nodeid="1788">从图中可知，镜像的操作可分为：</p>
<ul data-nodeid="1789">
<li data-nodeid="1790">
<p data-nodeid="1791">拉取镜像，使用<code data-backticks="1" data-nodeid="1977">docker pull</code>命令拉取远程仓库的镜像到本地 ；</p>
</li>
<li data-nodeid="1792">
<p data-nodeid="1793">重命名镜像，使用<code data-backticks="1" data-nodeid="1980">docker tag</code>命令“重命名”镜像 ；</p>
</li>
<li data-nodeid="1794">
<p data-nodeid="1795">查看镜像，使用<code data-backticks="1" data-nodeid="1983">docker image ls</code>或<code data-backticks="1" data-nodeid="1985">docker images</code>命令查看本地已经存在的镜像 ；</p>
</li>
<li data-nodeid="1796">
<p data-nodeid="1797">删除镜像，使用<code data-backticks="1" data-nodeid="1988">docker rmi</code>命令删除无用镜像 ；</p>
</li>
<li data-nodeid="1798">
<p data-nodeid="1799">构建镜像，构建镜像有两种方式。第一种方式是使用<code data-backticks="1" data-nodeid="1991">docker build</code>命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用<code data-backticks="1" data-nodeid="1993">docker commit</code>命令基于已经运行的容器提交为镜像。</p>
</li>
</ul>
<p data-nodeid="1800">下面，我们逐一详细介绍。</p>
<h4 data-nodeid="1801">拉取镜像</h4>
<p data-nodeid="1802">Docker 镜像的拉取使用<code data-backticks="1" data-nodeid="1998">docker pull</code>命令， 命令格式一般为 docker pull [Registry]/[Repository]/[Image]:[Tag]。</p>
<ul data-nodeid="1803">
<li data-nodeid="1804">
<p data-nodeid="1805">Registry 为注册服务器，Docker 默认会从 docker.io 拉取镜像，如果你有自己的镜像仓库，可以把 Registry 替换为自己的注册服务器。</p>
</li>
<li data-nodeid="1806">
<p data-nodeid="1807">Repository 为镜像仓库，通常把一组相关联的镜像归为一个镜像仓库，<code data-backticks="1" data-nodeid="2018">library</code>为 Docker 默认的镜像仓库。</p>
</li>
<li data-nodeid="1808">
<p data-nodeid="1809">Image 为镜像名称。</p>
</li>
<li data-nodeid="1810">
<p data-nodeid="1811">Tag 为镜像的标签，如果你不指定拉取镜像的标签，默认为<code data-backticks="1" data-nodeid="2022">latest</code>。</p>
</li>
</ul>
<p data-nodeid="1812">例如，我们需要获取一个 busybox 镜像，可以执行以下命令：</p>
<blockquote data-nodeid="1813">
<p data-nodeid="1814">busybox 是一个集成了数百个 Linux 命令（例如 curl、grep、mount、telnet 等）的精简工具箱，只有几兆大小，被誉为 Linux 系统的瑞士军刀。我经常会使用 busybox 做调试来查找生产环境中遇到的问题。</p>
</blockquote>
<pre class="lang-java" data-nodeid="1815"><code data-language="java">$ docker pull busybox
Using <span class="hljs-keyword">default</span> tag: latest
latest: Pulling from library/busybox
<span class="hljs-number">61</span>c5ed1cbdf8: Pull complete
Digest: sha256:<span class="hljs-number">4f</span>47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Downloaded newer image <span class="hljs-keyword">for</span> busybox:latest
docker.io/library/busybox:latest
</code></pre>
<p data-nodeid="1816">实际上执行<code data-backticks="1" data-nodeid="2027">docker pull busybox</code>命令，都是先从本地搜索，如果本地搜索不到<code data-backticks="1" data-nodeid="2029">busybox</code>镜像则从 Docker Hub 下载镜像。</p>
<p data-nodeid="1817">拉取完镜像，如果你想查看镜像，应该怎么操作呢？</p>
<h4 data-nodeid="1818">查看镜像</h4>
<p data-nodeid="1819">Docker 镜像查看使用<code data-backticks="1" data-nodeid="2034">docker images</code>或者<code data-backticks="1" data-nodeid="2036">docker image ls</code>命令。</p>
<p data-nodeid="1820">下面我们使用<code data-backticks="1" data-nodeid="2039">docker images</code>命令列出本地所有的镜像。</p>
<pre class="lang-java" data-nodeid="1821"><code data-language="java">$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              <span class="hljs-number">4</span>bb46517cac3        <span class="hljs-number">9</span> days ago          <span class="hljs-number">133</span>MB
nginx               <span class="hljs-number">1.15</span>                <span class="hljs-number">53f</span>3fd8007f7        <span class="hljs-number">15</span> months ago       <span class="hljs-number">109</span>MB
busybox             latest              <span class="hljs-number">01</span>8c9d7b792b        <span class="hljs-number">3</span> weeks ago         <span class="hljs-number">1.22</span>MB
</code></pre>
<p data-nodeid="1822">如果我们想要查询指定的镜像，可以使用<code data-backticks="1" data-nodeid="2042">docker image ls</code>命令来查询。</p>
<pre class="lang-java" data-nodeid="1823"><code data-language="java">$ docker image ls busybox
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              <span class="hljs-number">01</span>8c9d7b792b        <span class="hljs-number">3</span> weeks ago         <span class="hljs-number">1.22</span>MB
</code></pre>
<p data-nodeid="1824">当然你也可以使用<code data-backticks="1" data-nodeid="2045">docker images</code>命令列出所有镜像，然后使用<code data-backticks="1" data-nodeid="2047">grep</code>命令进行过滤。使用方法如下：</p>
<pre class="lang-java" data-nodeid="1825"><code data-language="java">$ docker images |grep busybox
busybox             latest              <span class="hljs-number">01</span>8c9d7b792b        <span class="hljs-number">3</span> weeks ago         <span class="hljs-number">1.22</span>MB
</code></pre>
<h4 data-nodeid="1826">“重命名”镜像</h4>
<p data-nodeid="1827">如果你想要自定义镜像名称或者推送镜像到其他镜像仓库，你可以使用<code data-backticks="1" data-nodeid="2051">docker tag</code>命令将镜像重命名。<code data-backticks="1" data-nodeid="2053">docker tag</code>的命令格式为 docker tag [SOURCE_IMAGE][:TAG] [TARGET_IMAGE][:TAG]。</p>
<p data-nodeid="1828">下面我们通过实例演示一下：</p>
<pre class="lang-plain" data-nodeid="1829"><code data-language="plain">$ docker tag busybox:latest mybusybox:latest
</code></pre>
<p data-nodeid="1830">执行完<code data-backticks="1" data-nodeid="2075">docker tag</code>命令后，可以使用查询镜像命令查看一下镜像列表：</p>
<pre class="lang-plain" data-nodeid="1831"><code data-language="plain">docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              018c9d7b792b        3 weeks ago         1.22MB
mybusybox           latest              018c9d7b792b        3 weeks ago         1.22MB
</code></pre>
<p data-nodeid="1832">可以看到，镜像列表中多了一个<code data-backticks="1" data-nodeid="2078">mybusybox</code>的镜像。但细心的同学可能已经发现，<code data-backticks="1" data-nodeid="2080">busybox</code>和<code data-backticks="1" data-nodeid="2082">mybusybox</code>这两个镜像的 IMAGE ID 是完全一样的。为什么呢？实际上它们指向了同一个镜像文件，只是别名不同而已。<br>
如果我不需要<code data-backticks="1" data-nodeid="2086">mybusybox</code>镜像了，想删除它，应该怎么操作呢？</p>
<h4 data-nodeid="1833">删除镜像</h4>
<p data-nodeid="1834">你可以使用<code data-backticks="1" data-nodeid="2090">docker rmi</code>或者<code data-backticks="1" data-nodeid="2092">docker image rm</code>命令删除镜像。</p>
<p data-nodeid="1835">举例：你可以使用以下命令删除<code data-backticks="1" data-nodeid="2095">mybusybox</code>镜像。</p>
<pre class="lang-js" data-nodeid="1836"><code data-language="js">$ docker rmi mybusybox
<span class="hljs-attr">Untagged</span>: mybusybox:latest
</code></pre>
<p data-nodeid="1837">此时，再次使用<code data-backticks="1" data-nodeid="2098">docker images</code>命令查看一下我们机器上的镜像列表。</p>
<pre class="lang-java" data-nodeid="1838"><code data-language="java">$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
busybox             latest              <span class="hljs-number">01</span>8c9d7b792b        <span class="hljs-number">3</span> weeks ago         <span class="hljs-number">1.22</span>MB
</code></pre>
<p data-nodeid="1839">通过上面的输出，我们可以看到，<code data-backticks="1" data-nodeid="2101">mybusybox</code>镜像已经被删除。<br>
如果你想构建属于自己的镜像，应该怎么做呢？</p>
<h4 data-nodeid="1840">构建镜像</h4>
<p data-nodeid="1841">构建镜像主要有两种方式：</p>
<ol data-nodeid="1842">
<li data-nodeid="1843">
<p data-nodeid="1844">使用<code data-backticks="1" data-nodeid="2108">docker commit</code>命令从运行中的容器提交为镜像；</p>
</li>
<li data-nodeid="1845">
<p data-nodeid="1846">使用<code data-backticks="1" data-nodeid="2111">docker build</code>命令从 Dockerfile 构建镜像。</p>
</li>
</ol>
<p data-nodeid="1847">首先介绍下如何从运行中的容器提交为镜像。我依旧使用 busybox 镜像举例，使用以下命令创建一个名为 busybox 的容器并进入 busybox 容器。</p>
<pre class="lang-dart" data-nodeid="1848"><code data-language="dart">$ docker run --rm --name=busybox -it busybox sh
/ #
</code></pre>
<p data-nodeid="1849">执行完上面的命令后，当前窗口会启动一个 busybox 容器并且进入容器中。在容器中，执行以下命令创建一个文件并写入内容：</p>
<pre class="lang-dart" data-nodeid="1850"><code data-language="dart">/ # touch hello.txt &amp;&amp; echo <span class="hljs-string">"I love Docker. "</span> &gt; hello.txt
/ #
</code></pre>
<p data-nodeid="1851">此时在容器的根目录下，已经创建了一个 hello.txt 文件，并写入了 "I love Docker. "。下面，我们新打开另一个命令行窗口，运行以下命令提交镜像：</p>
<pre class="lang-plain" data-nodeid="1852"><code data-language="plain">$ docker commit busybox busybox:hello
sha256:cbc6406aaef080d1dd3087d4ea1e6c6c9915ee0ee0f5dd9e0a90b03e2215e81c
</code></pre>
<p data-nodeid="1853">然后使用上面讲到的<code data-backticks="1" data-nodeid="2121">docker image ls</code>命令查看镜像：</p>
<pre class="lang-java" data-nodeid="1854"><code data-language="java">$ docker image ls busybox
REPOSITORY&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TAG&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;IMAGE ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; CREATED&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;SIZE
busybox&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;hello&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cbc6406aaef0&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">2</span> minutes ago&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">1.22</span>MB
busybox&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;latest&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">01</span>8c9d7b792b&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">4</span> weeks ago&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">1.22</span>MB
</code></pre>
<p data-nodeid="1855">此时我们可以看到主机上新生成了 busybox:hello 这个镜像。</p>
<p data-nodeid="1856">第二种方式是最重要也是最常用的镜像构建方式：Dockerfile。Dockerfile 是一个包含了用户所有构建命令的文本。通过<code data-backticks="1" data-nodeid="2125">docker build</code>命令可以从 Dockerfile 生成镜像。</p>
<p data-nodeid="1857">使用 Dockerfile 构建镜像具有以下特性：</p>
<ul data-nodeid="1858">
<li data-nodeid="1859">
<p data-nodeid="1860">Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID；</p>
</li>
<li data-nodeid="1861">
<p data-nodeid="1862">Dockerfile 的命令是完全透明的，通过查看 Dockerfile 的内容，就可以知道镜像是如何一步步构建的；</p>
</li>
<li data-nodeid="1863">
<p data-nodeid="1864">Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理。</p>
</li>
</ul>
<p data-nodeid="1865">看到使用 Dockerfile 的方式构建镜像有这么多好的特性，你是不是已经迫不及待想知道如何使用了。别着急，我们先学习下 Dockerfile 常用的指令。</p>
<table data-nodeid="1867">
<thead data-nodeid="1868">
<tr data-nodeid="1869">
<th data-org-content="Dockerfile 指令" data-nodeid="1871">Dockerfile 指令</th>
<th data-org-content="指令简介" data-nodeid="1872">指令简介</th>
</tr>
</thead>
<tbody data-nodeid="1875">
<tr data-nodeid="1876">
<td data-org-content="FROM" data-nodeid="1877">FROM</td>
<td data-org-content="Dockerfile 除了注释第一行必须是 FROM ，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。" data-nodeid="1878">Dockerfile 除了注释第一行必须是 FROM ，FROM 后面跟镜像名称，代表我们要基于哪个基础镜像构建我们的容器。</td>
</tr>
<tr data-nodeid="1879">
<td data-org-content="RUN" data-nodeid="1880">RUN</td>
<td data-org-content="RUN 后面跟一个具体的命令，类似于 Linux 命令行执行命令。" data-nodeid="1881">RUN 后面跟一个具体的命令，类似于 Linux 命令行执行命令。</td>
</tr>
<tr data-nodeid="1882">
<td data-org-content="ADD" data-nodeid="1883">ADD</td>
<td data-org-content="拷贝本机文件或者远程文件到镜像内" data-nodeid="1884">拷贝本机文件或者远程文件到镜像内</td>
</tr>
<tr data-nodeid="1885">
<td data-org-content="COPY" data-nodeid="1886">COPY</td>
<td data-org-content="拷贝本机文件到镜像内" data-nodeid="1887">拷贝本机文件到镜像内</td>
</tr>
<tr data-nodeid="1888">
<td data-org-content="USER" data-nodeid="1889">USER</td>
<td data-org-content="指定容器启动的用户" data-nodeid="1890">指定容器启动的用户</td>
</tr>
<tr data-nodeid="1891">
<td data-org-content="ENTRYPOINT" data-nodeid="1892">ENTRYPOINT</td>
<td data-org-content="容器的启动命令" data-nodeid="1893">容器的启动命令</td>
</tr>
<tr data-nodeid="1894">
<td data-org-content="CMD" data-nodeid="1895">CMD</td>
<td data-org-content="CMD 为 ENTRYPOINT 指令提供默认参数，也可以单独使用 CMD 指定容器启动参数" data-nodeid="1896">CMD 为 ENTRYPOINT 指令提供默认参数，也可以单独使用 CMD 指定容器启动参数</td>
</tr>
<tr data-nodeid="1897">
<td data-org-content="ENV" data-nodeid="1898">ENV</td>
<td data-org-content="指定容器运行时的环境变量，格式为 key=value" data-nodeid="1899">指定容器运行时的环境变量，格式为 key=value</td>
</tr>
<tr data-nodeid="1900">
<td data-org-content="ARG" data-nodeid="1901">ARG</td>
<td data-org-content="定义外部变量，构建镜像时可以使用 build-arg = 的格式传递参数用于构建" data-nodeid="1902">定义外部变量，构建镜像时可以使用 build-arg = 的格式传递参数用于构建</td>
</tr>
<tr data-nodeid="1903">
<td data-org-content="EXPOSE" data-nodeid="1904">EXPOSE</td>
<td data-org-content="指定容器监听的端口，格式为 [port]/tcp 或者 [port]/udp" data-nodeid="1905">指定容器监听的端口，格式为 [port]/tcp 或者 [port]/udp</td>
</tr>
<tr data-nodeid="1906">
<td data-org-content="WORKDIR" data-nodeid="1907">WORKDIR</td>
<td data-org-content="为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。" data-nodeid="1908">为 Dockerfile 中跟在其后的所有 RUN、CMD、ENTRYPOINT、COPY 和 ADD 命令设置工作目录。</td>
</tr>
</tbody>
</table>
<p data-nodeid="1909">看了这么多指令，感觉有点懵？别担心，我通过一个实例让你来熟悉它们。这是一个 Dockerfile：</p>
<pre class="lang-dockerfile" data-nodeid="1910"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> centos:<span class="hljs-number">7</span>
<span class="hljs-keyword">COPY</span><span class="bash"> nginx.repo /etc/yum.repos.d/nginx.repo</span>
<span class="hljs-keyword">RUN</span><span class="bash"> yum install -y nginx</span>
<span class="hljs-keyword">EXPOSE</span> <span class="hljs-number">80</span>
<span class="hljs-keyword">ENV</span> HOST=mynginx
<span class="hljs-keyword">CMD</span><span class="bash"> [<span class="hljs-string">"nginx"</span>,<span class="hljs-string">"-g"</span>,<span class="hljs-string">"daemon off;"</span>]</span>
</code></pre>
<p data-nodeid="1911">好，我来逐行分析一下上述的 Dockerfile。</p>
<ul data-nodeid="1912">
<li data-nodeid="1913">
<p data-nodeid="1914">第一行表示我要基于 centos:7 这个镜像来构建自定义镜像。这里需要注意，每个 Dockerfile 的第一行除了注释都必须以 FROM 开头。</p>
</li>
<li data-nodeid="1915">
<p data-nodeid="1916">第二行表示拷贝本地文件 nginx.repo 文件到容器内的 /etc/yum.repos.d 目录下。这里拷贝 nginx.repo 文件是为了添加 nginx 的安装源。</p>
</li>
<li data-nodeid="1917">
<p data-nodeid="1918">第三行表示在容器内运行<code data-backticks="1" data-nodeid="2169">yum install -y nginx</code>命令，安装 nginx 服务到容器内，执行完第三行命令，容器内的 nginx 已经安装完成。</p>
</li>
<li data-nodeid="1919">
<p data-nodeid="1920">第四行声明容器内业务（nginx）使用 80 端口对外提供服务。</p>
</li>
<li data-nodeid="1921">
<p data-nodeid="1922">第五行定义容器启动时的环境变量 HOST=mynginx，容器启动后可以获取到环境变量 HOST 的值为 mynginx。</p>
</li>
<li data-nodeid="1923">
<p data-nodeid="1924">第六行定义容器的启动命令，命令格式为 json 数组。这里设置了容器的启动命令为 nginx ，并且添加了 nginx 的启动参数 -g 'daemon off;' ，使得 nginx 以前台的方式启动。</p>
</li>
</ul>
<p data-nodeid="1925">上面这个 Dockerfile 的例子基本涵盖了常用的镜像构建指令，代码我已经放在 <a href="https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles" data-nodeid="2181">GitHub</a>上，如果你感兴趣可以到 <a href="https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles" data-nodeid="2185">GitHub 下载源码</a>并尝试构建这个镜像。</p>
<p data-nodeid="1926">学习了镜像的各种操作，下面我们深入了解一下镜像的实现原理。</p>
<h3 data-nodeid="1927">镜像的实现原理</h3>
<p data-nodeid="1928">其实 Docker 镜像是由一系列镜像层（layer）组成的，每一层代表了镜像构建过程中的一次提交。下面以一个镜像构建的 Dockerfile 来说明镜像是如何分层的。</p>
<pre class="lang-dockerfile" data-nodeid="1929"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> busybox
<span class="hljs-keyword">COPY</span><span class="bash"> <span class="hljs-built_in">test</span> /tmp/<span class="hljs-built_in">test</span></span>
<span class="hljs-keyword">RUN</span><span class="bash"> mkdir /tmp/testdir</span>
</code></pre>
<p data-nodeid="1930">上面的 Dockerfile 由三步组成：</p>
<p data-nodeid="1931">第一行基于 busybox 创建一个镜像层；</p>
<p data-nodeid="1932">第二行拷贝本机 test 文件到镜像内；</p>
<p data-nodeid="8440" class="">第三行在 /tmp 文件夹下创建一个目录 testdir。</p>








<p data-nodeid="1934">为了验证镜像的存储结构，我们使用<code data-backticks="1" data-nodeid="2195">docker build</code>命令在上面 Dockerfile 所在目录构建一个镜像：</p>
<pre class="lang-shell" data-nodeid="1935"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker build -t mybusybox .</span>
</code></pre>
<p data-nodeid="1936">这里我的 Docker 使用的是 overlay2 文件驱动，进入到<code data-backticks="1" data-nodeid="2198">/var/lib/docker/overlay2</code>目录下使用<code data-backticks="1" data-nodeid="2200">tree .</code>命令查看产生的镜像文件：</p>
<pre class="lang-dart" data-nodeid="1937"><code data-language="dart">$ tree .
# 以下为 tree . 命令输出内容
|-- <span class="hljs-number">3e89</span>b959f921227acab94f5ab4524252ae0a829ff8a3687178e3aca56d605679
|   |-- diff  # 这一层为基础层，对应上述 Dockerfile 第一行，包含 busybox 镜像所有文件内容，例如 /etc,/bin,/<span class="hljs-keyword">var</span> 等目录
... 此次省略部分原始镜像文件内容
|   `-- link 
|-- <span class="hljs-number">6591</span>d4e47eb2488e6297a0a07a2439f550cdb22845b6d2ddb1be2466ae7a9391
|   |-- diff   # 这一层对应上述 Dockerfile 第二行，拷贝 test 文件到 /tmp 文件夹下，因此 diff 文件夹下有了 /tmp/test 文件
|   |   `-- tmp
|   |       `-- test
|   |-- link
|   |-- lower
|   `-- work
|-- backingFsBlockDev
|-- bec6a018080f7b808565728dee8447b9e86b3093b16ad5e6a1ac3976528a8bb1
|   |-- diff &nbsp;# 这一层对应上述 Dockerfile 第三行，在 /tmp 文件夹下创建 testdir 文件夹，因此 diff 文件夹下有了 /tmp/testdir 文件夹
|   |   `-- tmp
|   |       `-- testdir
|   |-- link
|   |-- lower
|   `-- work
...
</code></pre>
<p data-nodeid="1938">通过上面的目录结构可以看到，Dockerfile 的每一行命令，都生成了一个镜像层，每一层的 diff 夹下只存放了增量数据，如图 2 所示。</p>
<p data-nodeid="1939"><img src="https://s0.lgstatic.com/i/image/M00/4A/AD/CgqCHl9SDmGACBEjAABkgtnn_hE625.png" alt="Lark20200904-175137.png" data-nodeid="2205"></p>
<div data-nodeid="1940"><p style="text-align:center">图 2 镜像文件系统</p></div>
<p data-nodeid="1941">分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。</p>
<p data-nodeid="1942">总结一下， Docker 镜像是静态的分层管理的文件组合，镜像底层的实现依赖于联合文件系统（UnionFS）。充分掌握镜像的原理，可以帮助我们在生产实践中构建出最优的镜像，同时也可以帮助我们更好地理解容器和镜像的关系。</p>
<h4 data-nodeid="1943">总结</h4>
<p data-nodeid="1944">到此，相信你已经对 Docker 镜像这一核心概念有了较深的了解，并熟悉了 Docker 镜像的常用操作（拉取、查看、“重命名”、删除和构建自定义镜像）及底层实现原理。</p>
<p data-nodeid="1945">本课时内容精华，我帮你总结如下：</p>
<blockquote data-nodeid="1946">
<p data-nodeid="1947">镜像操作命令：</p>
<ol data-nodeid="1948">
<li data-nodeid="1949">
<p data-nodeid="1950">拉取镜像，使用 docker pull 命令拉取远程仓库的镜像到本地 ；</p>
</li>
<li data-nodeid="1951">
<p data-nodeid="1952">重命名镜像，使用 docker tag 命令“重命名”镜像 ；</p>
</li>
<li data-nodeid="1953">
<p data-nodeid="1954">查看镜像，使用 docker image ls 或 docker images 命令查看本地已经存在的镜像；</p>
</li>
<li data-nodeid="1955">
<p data-nodeid="1956">删除镜像，使用 docker rmi 命令删除无用镜像 ；</p>
</li>
<li data-nodeid="1957">
<p data-nodeid="1958">构建镜像，构建镜像有两种方式。第一种方式是使用 docker build 命令基于 Dockerfile 构建镜像，也是我比较推荐的镜像构建方式；第二种方式是使用 docker commit 命令基于已经运行的容器提交为镜像。</p>
</li>
</ol>
<p data-nodeid="1959">镜像的实现原理：<br>
镜像是由一系列的镜像层（layer ）组成，每一层代表了镜像构建过程中的一次提交，当我们需要修改镜像内的某个文件时，只需要在当前镜像层的基础上新建一个镜像层，并且只存放修改过的文件内容。分层结构使得镜像间共享镜像层变得非常简单和方便。</p>
</blockquote>
<p data-nodeid="1960">最后试想下，如果有一天我们机器存储空间不足，那你知道使用什么命令可以清理本地无用的镜像和容器文件吗？思考后，可以把你的想法写在留言区。</p>
<p data-nodeid="1961" class=""><a href="https://github.com/wilhelmguo/docker-demo/tree/master/dockerfiles" data-nodeid="2223">点击即可查看本课时相关源码</a></p>

---

### 精选评论

##### *豆：
> 清理容器多余数据相关的命令有两条，分别是：docker image prune -af 仅仅清除没有被容器使用的镜像文件docker system prune -f 清除多余的数据，包括停止的容器、多余的镜像、未被使用的volume等等

##### *鑫：
> 老师关于docker的logs的清理有什么好的方法吗？运行久了发现logs比镜像还大

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以通过 配置 docker 的 log-opts 参数设置每个容器产生的日志文件大小和数量

##### **军：
> 老师 ，对于已经下载的镜像怎么看dockerfile 呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 镜像办法直接查看 Dockerfile，不过可以通过 docker history 命令查看镜像构建历史

##### *凡：
> 老师，Dockerfile将您的busybox换成了rabbitmq报错误error checking context: 'file ('/proc/39633/fd/5') not found or excluded by .dockerignore'.，网上没找到有用的解决方法，不知道什么问题Dockerfile只有FROM rabbitmqCOPY test /tmp/testRUN mkdir /tmp/testdir麻烦老师帮忙看下，谢谢！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 确认下执行命令的目录下是否有 test 文件

##### *昕：
> 请教 ：1 如何判断 下载到本地的tag是latest的镜像 对应registry中具体哪个镜像(哪个功能tag，哪个OS/ARCH)。2 Digest 的作用。本地镜像和registry 中 digest不一样 ，对应不上 。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 通常不建议使用latest 作为镜像标签，想要区分镜像是否与仓库一致可以使用镜像 ID 来对比。2. Digest 是 manifest内容的sha256sum

##### **狗：
> 如果我想从上面busybox:hello里面copy 那个hello.txt到新build的镜像中，我的Dockerfile如果写成：FROM busybox:helloCOPY /hello.txt /tmp/111.txt运行命令docker build -t xbox .之后得到提示COPY failed: stat /var/lib/docker/tmp/docker-builder074112902/hello.txt: no such file or directory请问这怎解决？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; COPY 命令只能从本地目录拷贝文件，想要从其它镜像拷贝文件，需要用到多阶段构建技术（
multistage-build），我会在 22 课时详细讲解多阶段构建技术

##### *啸：
> 请问from这个指令是在哪里执行的，在控制台不行，进入容器的控制台也不行，我的运行环境是docker windows

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; FROM 指令是写在 Dockerfile 里的，不是直接执行的命令。

##### *森：
> 按照教程docker tag的时候提示no such id，将仓库名称换成镜像ID就可以了，请问老师这正常吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为你本地原来没有 busybox:latest 镜像吧，需要先把这个镜像拉取下来

##### **大：
> 删除镜像时出现了，，看上去镜像只是名字被清空了，没有删掉，是啥原因

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可能还有别的容器或者镜像在引用相同的层，如果想强制删除，建议使用 -f 选项

##### **旺：
> /var/lib/docker/overlay2 我竟然没有这个目录

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果你的文件驱动不是 overlay2 就没有这个目录，Docker 的文件系统我会在 14，15，16 讲详细讲解

##### **0860：
> 老师，有疑问，不是说镜像只是只读吗？为什么能修改哦？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 镜像不能修改，docker tag 只是给镜像起个别名

##### **7885：
> 请教。link. lower. Work 有什么用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请问你说的 link. lower. Work 分别指什么？命令还是文件夹？

##### **俊：
> 对层的概念不太理解，每一句都是一层？比如host 80也是？这样做的好处是？除了刚才说的共享，说到共享，还想问一下。这个共享是docker会自动共享还是需要dockerfile里面用命令设置

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 分层主要就是为了共享，镜像层共享是 Docker 默认的机制，无需设置

##### **用户1280：
> 删除镜像：docker rmi或docker image rm 镜像名

##### *伟：
> docker rm -f 容器docker rmi  镜像

