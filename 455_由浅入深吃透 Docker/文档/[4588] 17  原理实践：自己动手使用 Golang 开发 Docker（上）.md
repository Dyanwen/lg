<p data-nodeid="761" class="">第一模块，我们从 Docker 基础概念讲到 Docker 的基本操作。第二模块，我们详细剖析了 Docker 的三大关键技术（ Namespace、cgroups 和联合文件系统）的实现原理，并且讲解了 Docker 的网络模型等关键性技术。相信此时的你已经对 Docker 有了一个新的认识。</p>
<p data-nodeid="762">接下来的两课时，我就趁热打铁，带你动手使用 Golang 编写一个 Docker。学习这两节的内容需要你能够熟练使用 Golang 语言，如果你没有 Golang 编程基础，建议先学习一下 Golang 的基本语法。那么 Golang 究竟是什么呢? Golang 应该如何安装使用？下面我带你一一学习。</p>
<h3 data-nodeid="763">Golang 是什么?</h3>
<p data-nodeid="764">Golang 又称为 Go，是 Google 开源的一种静态编译型语言，Golang 自带内存管理机制，相比于 C 和 C++ 语言，我们不需要关心内存的分配和回收。</p>
<p data-nodeid="765">Golang 是新一代的互联网编程语言，在 Golang 诞生前，C 或 C++ 作为服务端高性能编程语言，使用 C 或 C++ 开发的业务具有非常高的执行效率，但是编译和开发效率却不尽人意，Java、.NET 等语言的诞生大大提高了软件开发速度，但是运行效率和资源占用却不如 C 和 C++。</p>
<p data-nodeid="766">这时 Golang 横空出世，由于 Golang 较高的开发效率和执行效率，很快便从众多编程语言中脱颖而出，成为众多互联网公司的新宠儿。滴滴、知乎、阿里等众多大型互联网公司都在大量使用 Golang。 同时，Docker 和 Kubernetes 等众多明星项目也都是使用 Golang 开发的。因此，熟练掌握 Golang 将会为你加分很多。</p>
<p data-nodeid="767">这么好的编程语言，你是不是已经迫不及待地想要安装体验一下了？别着急，下面我带你来安装一个 Golang 环境。</p>
<h3 data-nodeid="768">Golang 安装</h3>
<p data-nodeid="769">安装信息如下：</p>
<ul data-nodeid="770">
<li data-nodeid="771">
<p data-nodeid="772">CentOS 7系统</p>
</li>
<li data-nodeid="773">
<p data-nodeid="774">Golang 版本 1.15.2</p>
</li>
</ul>
<p data-nodeid="775">首先我们到<a href="https://golang.org/" data-nodeid="865">Golang 官网</a>（由于国内无法访问 Golang 官网，推荐到<a href="https://studygolang.com/dl" data-nodeid="869">Golang 中文网</a>下载安装包）下载一个对应操作系统的安装包。</p>
<pre class="lang-java" data-nodeid="776"><code data-language="java">$ cd /tmp &amp;&amp; wget https:<span class="hljs-comment">//studygolang.com/dl/golang/go1.15.2.linux-amd64.tar.gz</span>
</code></pre>
<p data-nodeid="777">解压缩安装包：</p>
<pre class="lang-java" data-nodeid="778"><code data-language="java">$ sudo tar -C /usr/local -xzf&nbsp;go1.<span class="hljs-number">15.2</span>.linux-amd64.tar.gz
</code></pre>
<p data-nodeid="779">在 $HOME/.bashrc 文件末尾添加以下内容，将 Golang 可执行文件目录添加到系统 PATH 中：</p>
<pre class="lang-java" data-nodeid="780"><code data-language="java">export PATH=$PATH:/usr/local/go/bin
</code></pre>
<p data-nodeid="781">将 go 的安装路径添加到系统 PATH 中后，就可以在命令行直接使用 go 命令了。配置好 go 命令后，我们还需要配置 GOPATH 才能正确存放和编译我们的 go 代码。</p>
<h4 data-nodeid="782">配置 GOPATH</h4>
<p data-nodeid="783">GOPATH 是 Golang 的源码和相关编译文件的存放路径，GOPATH 路径下有三个文件夹 src、pkg 和 bin，它们的用途分别是：</p>
<table data-nodeid="785">
<thead data-nodeid="786">
<tr data-nodeid="787">
<th data-org-content="**目录**" data-nodeid="789"><strong data-nodeid="879">目录</strong></th>
<th data-org-content="**用途**" data-nodeid="790"><strong data-nodeid="883">用途</strong></th>
</tr>
</thead>
<tbody data-nodeid="793">
<tr data-nodeid="794">
<td data-org-content="src" data-nodeid="795">src</td>
<td data-org-content="源代码存放路径或者引用的外部库" data-nodeid="796">源代码存放路径或者引用的外部库</td>
</tr>
<tr data-nodeid="797">
<td data-org-content="pkg" data-nodeid="798">pkg</td>
<td data-org-content="编译时生成的对象文件" data-nodeid="799">编译时生成的对象文件</td>
</tr>
<tr data-nodeid="800">
<td data-org-content="bin" data-nodeid="801">bin</td>
<td data-org-content="编译后的可执行二进制" data-nodeid="802">编译后的可执行二进制</td>
</tr>
</tbody>
</table>
<p data-nodeid="803">这里我们开始配置 GOPATH 路径为 /go。首先准备相关的目录：</p>
<pre class="lang-java" data-nodeid="804"><code data-language="java">$ sudo mkdir /go
$ sudo mkdir /go/src
$ sudo mkdir /go/pkg
$ sudo mkdir /go/bin
</code></pre>
<p data-nodeid="805">然后将 GOPATH 添加到 $HOME/.bashrc 文件末尾，并且把 GOPATH 下的 bin 目录也添加到系统的 PATH 中，这样方便程序编译后直接使用。添加的内容如下：</p>
<pre class="lang-java" data-nodeid="806"><code data-language="java">export GOPATH=/go
export PATH=$PATH:$GOPATH/bin
# 设置 Golang 的代理，方便我们顺利下载依赖包
export GOPROXY="https://goproxy.io,direct"
</code></pre>
<p data-nodeid="807">接下来，使用 source $HOME/.bashrc 命令生效一下我们的配置，然后我们再使用 go env 命令查看一下我们的配置结果：</p>
<pre class="lang-java" data-nodeid="808"><code data-language="java">$ go env
GO111MODULE=<span class="hljs-string">""</span>
GOARCH=<span class="hljs-string">"amd64"</span>
GOBIN=<span class="hljs-string">""</span>
GOCACHE=<span class="hljs-string">"/root/.cache/go-build"</span>
GOENV=<span class="hljs-string">"/root/.config/go/env"</span>
GOEXE=<span class="hljs-string">""</span>
GOFLAGS=<span class="hljs-string">""</span>
GOHOSTARCH=<span class="hljs-string">"amd64"</span>
GOHOSTOS=<span class="hljs-string">"linux"</span>
GOINSECURE=<span class="hljs-string">""</span>
GOMODCACHE=<span class="hljs-string">"/go/pkg/mod"</span>
GONOPROXY=<span class="hljs-string">""</span>
GONOSUMDB=<span class="hljs-string">""</span>
GOOS=<span class="hljs-string">"linux"</span>
GOPATH=<span class="hljs-string">"/go"</span>
GOPRIVATE=<span class="hljs-string">""</span>
GOPROXY=<span class="hljs-string">"https://goproxy.io,direct"</span>
GOROOT=<span class="hljs-string">"/usr/local/go"</span>
GOSUMDB=<span class="hljs-string">"sum.golang.org"</span>
GOTMPDIR=<span class="hljs-string">""</span>
GOTOOLDIR=<span class="hljs-string">"/usr/local/go/pkg/tool/linux_amd64"</span>
GCCGO=<span class="hljs-string">"gccgo"</span>
AR=<span class="hljs-string">"ar"</span>
CC=<span class="hljs-string">"gcc"</span>
CXX=<span class="hljs-string">"g++"</span>
CGO_ENABLED=<span class="hljs-string">"1"</span>
GOMOD=<span class="hljs-string">""</span>
CGO_CFLAGS=<span class="hljs-string">"-g -O2"</span>
CGO_CPPFLAGS=<span class="hljs-string">""</span>
CGO_CXXFLAGS=<span class="hljs-string">"-g -O2"</span>
CGO_FFLAGS=<span class="hljs-string">"-g -O2"</span>
CGO_LDFLAGS=<span class="hljs-string">"-g -O2"</span>
PKG_CONFIG=<span class="hljs-string">"pkg-config"</span>
GOGCCFLAGS=<span class="hljs-string">"-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build352828668=/tmp/go-build -gno-record-gcc-switches"</span>
</code></pre>
<p data-nodeid="809">从 GOPATH 和 GOPROXY 两个变量的结果，可以看到 GOPATH 和 GOPROXY 均已经生效。到此，我们的 Golang 已经安装完毕。下面，我们就开始真正的 Docker 编写之旅吧。</p>
<h3 data-nodeid="810">编写 Docker</h3>
<p data-nodeid="811">在开始编写 Docker 之前，我先介绍几个基础知识，如果你对这些基础知识已经很熟悉了，可以直接跳过这块的基础知识。</p>
<h4 data-nodeid="812">Linux Proc 文件系统</h4>
<p data-nodeid="813">Linux 系统中，/proc 目录是一种“文件系统”，这里我用了引号，其实 /proc 目录并不是一个真正的文件系统。<strong data-nodeid="901">/proc 目录存放于内存中，是一个虚拟的文件系统，该目录存放了当前内核运行状态的一系列特殊的文件，你可以通过这些文件查看当前的进程信息。</strong></p>
<p data-nodeid="814">下面，我们通过 ls 命令查看一下 /proc 目录下的内容：</p>
<pre class="lang-java" data-nodeid="815"><code data-language="java">$ sudo ls -l /proc
total <span class="hljs-number">0</span>
dr-xr-xr-x&nbsp; <span class="hljs-number">9</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> <span class="hljs-number">1</span>
dr-xr-xr-x&nbsp; <span class="hljs-number">9</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> <span class="hljs-number">30097</span>
...省略部分输出
dr-xr-xr-x&nbsp; <span class="hljs-number">9</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> <span class="hljs-number">8</span>
dr-xr-xr-x&nbsp; <span class="hljs-number">9</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> <span class="hljs-number">9</span>
dr-xr-xr-x&nbsp; <span class="hljs-number">9</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> <span class="hljs-number">97</span>
dr-xr-xr-x&nbsp; <span class="hljs-number">2</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> acpi
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> buddyinfo
dr-xr-xr-x&nbsp; <span class="hljs-number">4</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> bus
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> cgroups
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> cmdline
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> consoles
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> cpuinfo
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> crypto
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> devices
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> diskstats
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> dma
dr-xr-xr-x&nbsp; <span class="hljs-number">2</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> driver
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> execdomains
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> fb
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> filesystems
dr-xr-xr-x&nbsp; <span class="hljs-number">5</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> fs
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> interrupts
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> iomem
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> ioports
dr-xr-xr-x <span class="hljs-number">27</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> irq
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> kallsyms
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; <span class="hljs-number">140737486266368</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> kcore
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> key-users
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> keys
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> kmsg
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> kpagecount
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> kpageflags
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> loadavg
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> locks
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> mdstat
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> meminfo
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> misc
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> modules
lrwxrwxrwx&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">11</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> mounts -&gt; self/mounts
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> mtrr
lrwxrwxrwx&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">8</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> net -&gt; self/net
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> pagetypeinfo
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> partitions
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> sched_debug
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> schedstat
dr-xr-xr-x&nbsp; <span class="hljs-number">2</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> scsi
lrwxrwxrwx&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> self -&gt; <span class="hljs-number">30097</span>
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> slabinfo
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> softirqs
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> stat
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> swaps
dr-xr-xr-x&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">34</span> sys
--w-------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> sysrq-trigger
dr-xr-xr-x&nbsp; <span class="hljs-number">2</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> sysvipc
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> timer_list
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> timer_stats
dr-xr-xr-x&nbsp; <span class="hljs-number">4</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> tty
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> uptime
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> version
-r--------&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> vmallocinfo
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> vmstat
-r--r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">27</span> zoneinfo
</code></pre>
<p data-nodeid="816">可以看到，这个目录下有很多数字，这些数字目录实际上是以进程 ID 命名的。除了这些以进程 ID 命名的目录，还有一些特殊的目录，这里我讲解一下与我们编写 Docker 有关的文件和目录。</p>
<ul data-nodeid="817">
<li data-nodeid="818">
<p data-nodeid="819"><strong data-nodeid="908">self 目录</strong>：它是连接到当前正在运行的进程目录，比如我当前的进程 ID 为 30097，则 self 目录实际连接到 /proc/30097 这个目录。</p>
</li>
<li data-nodeid="820">
<p data-nodeid="821"><strong data-nodeid="913">/proc/{PID}/exe 文件</strong>：exe 连接到进程执行的命令文件，例如 30097 这个进程的运行命令为 docker，则执行 /proc/30097/exe ps 等同于执行 docker ps。</p>
</li>
</ul>
<p data-nodeid="822">好了，了解完这些基础知识后，我们就开始行动吧！因为我们的精简版 Docker 是使用 Golang 编写，这里就给我们编写的 Docker 命名为 gocker 吧。</p>
<h4 data-nodeid="823">实现 gocker 的 run 命令</h4>
<p data-nodeid="824">通过前面的章节，我们学习了要运行一个容器，必须先有镜像。这里我们首先准备一个 busybox 镜像，以便我们运行 gocker 容器。</p>
<pre class="lang-java" data-nodeid="825"><code data-language="java">$ mkdir /tmp/busybox &amp;&amp; cd /tmp/busybox
$ docker export $(docker create busybox) -o busybox.tar
$ tar -xf busybox.tar
</code></pre>
<p data-nodeid="826">以上是我们在 /tmp/busybox 目录，使用 docker export 命令导出的一个 busybox 镜像文件，然后对镜像文件包进行解压，解压后 /tmp/busybox 目录内容如下：</p>
<pre class="lang-java" data-nodeid="827"><code data-language="java">$ ls -l /tmp/busybox/
total <span class="hljs-number">1472</span>
drwxr-xr-x <span class="hljs-number">2</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12288</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">02</span>:<span class="hljs-number">09</span> bin
-rw------- <span class="hljs-number">1</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; <span class="hljs-number">1455104</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">22</span>:<span class="hljs-number">47</span> busybox.tar
drwxr-xr-x <span class="hljs-number">4</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">16</span>:<span class="hljs-number">41</span> dev
drwxr-xr-x <span class="hljs-number">3</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">16</span>:<span class="hljs-number">41</span> etc
drwxr-xr-x <span class="hljs-number">2</span> nfsnobody nfsnobody&nbsp; &nbsp; <span class="hljs-number">4096</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">02</span>:<span class="hljs-number">09</span> home
drwxr-xr-x <span class="hljs-number">2</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">16</span>:<span class="hljs-number">41</span> proc
drwx------ <span class="hljs-number">2</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">21</span>:<span class="hljs-number">07</span> root
drwxr-xr-x <span class="hljs-number">2</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep <span class="hljs-number">19</span> <span class="hljs-number">16</span>:<span class="hljs-number">41</span> sys
drwxrwxrwt <span class="hljs-number">2</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">02</span>:<span class="hljs-number">09</span> tmp
drwxr-xr-x <span class="hljs-number">3</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">02</span>:<span class="hljs-number">09</span> usr
drwxr-xr-x <span class="hljs-number">4</span> root&nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">4096</span> Sep&nbsp; <span class="hljs-number">9</span> <span class="hljs-number">02</span>:<span class="hljs-number">09</span> <span class="hljs-keyword">var</span>
</code></pre>
<p data-nodeid="828">准备好镜像文件后，把我为你准备好的 gocker 代码下载下来吧，这里我使用手动下载源码的方式克隆代码：</p>
<pre class="lang-java" data-nodeid="829"><code data-language="java">$ mkdir -p /go/src/github.com/wilhelmguo
$ cd /go/src/github.com/wilhelmguo &amp;&amp; git clone https:<span class="hljs-comment">//github.com/wilhelmguo/gocker.git</span>
$ cd gocker
$ git checkout lesson-<span class="hljs-number">17</span>
</code></pre>
<blockquote data-nodeid="830">
<p data-nodeid="831">我的 GOPATH 在 /go 目录下，如果你的 GOPATH 跟我不一致，请根据 GOPATH 存放和编译源码。本课时的源码存放在<a href="https://github.com/wilhelmguo/gocker/tree/lesson-17" data-nodeid="922">这里</a>，你也可以在线阅读。</p>
</blockquote>
<p data-nodeid="832">代码下载完后，我们进入 gocker 的目录，查看下源码文件：</p>
<pre class="lang-java" data-nodeid="833"><code data-language="java">$ tree .
.
|-- go.mod
|-- go.sum
|-- main.go
|-- README.md
|-- runc
|&nbsp; &nbsp;`-- run.go
`-- vendor
... 省略 vendor 目录结构
<span class="hljs-number">15</span> directories, <span class="hljs-number">59</span> files
</code></pre>
<blockquote data-nodeid="834">
<p data-nodeid="835">本项目使用 go mod 管理包依赖，go mod 是在 golang 1.11 版本加入的新的特性，是用来管理包的依赖的，也是目前官方的包依赖管理工具。如果你想学习更多个 go mod 使用方法，可以参考<a href="https://golang.org/ref/mod" data-nodeid="928">官网</a>。</p>
</blockquote>
<p data-nodeid="836">可以看到该源码下有两个主要文件：一个是 main.go 文件，这是 gocker 的主入口函数；另外一个是 run.go ，这个文件是 gocker run 命令的具体实现。</p>
<p data-nodeid="837">下面我们使用 go install 命令来编译一下我们的 gocker 项目：</p>
<pre class="lang-java" data-nodeid="838"><code data-language="java">$ go install
</code></pre>
<p data-nodeid="839">执行完 go install 后， Golang 会自动帮助我们编译当前项目下的代码，编译后的二进制文件存放在 $GOPATH/bin 目录下。由于我们之前在 $HOME/.bashrc 文件下把 $GOPATH/bin 放入了系统 PATH 中，所以此时你可以直接使用 gocker 命令了。<br>
接下来我们使用 gocker 来启动一个容器：</p>
<pre class="lang-java" data-nodeid="951"><code data-language="java"># gocker run -it -rootfs=/tmp/busybox /bin/sh
2020/09/19 23:46:27 Current path is&nbsp; /tmp/busybox
2020/09/19 23:46:27 CmdArray is&nbsp; [/bin/sh]
/ #
</code></pre>
<blockquote data-nodeid="2551" class="">
<p data-nodeid="2552">如果出现 pivotRoot error pivot_root invalid argument 的报错，可以先执行 unshare -m 命令，然后使用 rm -rf /tmp/busybox/.pivot_root 命令删除临时文件，再次重试即可。</p>
</blockquote>
<p data-nodeid="2553">这里我们使用 it 参数指定以命令行交互的模式启动容器，rootfs 指定准备好的镜像目录。执行完上面的命令后 busybox 容器就成功启动了。<br>
这时候，我们使用 ps 命令查看一下当前进程信息：</p>






<pre class="lang-java" data-nodeid="842"><code data-language="java">/ # /bin/ps -ef
PID&nbsp; &nbsp;USER&nbsp; &nbsp; &nbsp;TIME&nbsp; COMMAND
&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp; 0:00 /bin/sh
&nbsp; &nbsp; 5 root&nbsp; &nbsp; &nbsp; 0:00 /bin/ps -ef
</code></pre>
<p data-nodeid="843">此时，容器内的进程已经与主机完全隔离。<br>
我们再查看一下当前目录下的内容：</p>
<pre class="lang-java" data-nodeid="844"><code data-language="java">/ # pwd
/
/ # /bin/ls -l
total 1468
drwxr-xr-x&nbsp; &nbsp; 2 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;12288 Sep&nbsp; 8 18:09 bin
-rw-------&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp;1455104 Sep 19 14:47 busybox.tar
drwxr-xr-x&nbsp; &nbsp; 4 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep 19 08:41 dev
drwxr-xr-x&nbsp; &nbsp; 3 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep 19 08:41 etc
drwxr-xr-x&nbsp; &nbsp; 2 nobody&nbsp; &nbsp;nobody&nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep&nbsp; 8 18:09 home
dr-xr-xr-x&nbsp; 122 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0 Sep 19 15:46 proc
drwx------&nbsp; &nbsp; 2 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep 19 13:07 root
drwxr-xr-x&nbsp; &nbsp; 2 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep 19 08:41 sys
drwxrwxrwt&nbsp; &nbsp; 2 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep&nbsp; 8 18:09 tmp
drwxr-xr-x&nbsp; &nbsp; 3 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep&nbsp; 8 18:09 usr
drwxr-xr-x&nbsp; &nbsp; 4 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4096 Sep&nbsp; 8 18:09 var
</code></pre>
<p data-nodeid="845">可以看到当前目录已经为根目录，并且根目录下的文件就是我们上面准备的 busybox 镜像文件。<br>
到此，一个完全由我们自己编写的 gocker 已经可以启动容器了。</p>
<h3 data-nodeid="846">结语</h3>
<p data-nodeid="847">本课时我们讲解了 Golang 是什么, 并且配置好了 Golang 环境，编译了 gocker，也了解了 Linux /proc 文件系统的一些重要功能，最后使用 gocker 成功启动了一个 busybox 容器。</p>
<p data-nodeid="848">那么你知道，为什么 Docker 会选择使用 Golang 来开发吗？思考后，把你的想法写在留言区。</p>
<p data-nodeid="849">下一课时我将为你全面剖析 gocker 的源码以及它的实现原理，让你能够自己动手把它写出来，到时见。</p>
<p data-nodeid="850" class=""><a href="https://github.com/wilhelmguo/gocker/tree/lesson-17" data-nodeid="950">点击链接，即可查看本课时的源码。</a></p>

---

### 精选评论

##### **平：
> 首先正如文中所说开发效率和执行效率选Go，其次无需像java和php或者python那样安装运行环境，只需一个二进制文件就能在Linux系统跑起来

##### *锋：
> gocker run -it -rootfs=/tmp/busybox /bin/sh出现下面的错误：2021/01/12 19:09:48 Current path is /tmp/busybox2021/01/12 19:09:48 CmdArray is [/bin/sh]2021/01/12 19:09:48 Exec loop path error exec: "/bin/sh": stat /bin/sh: no such file or directory2021/01/12 19:09:48 exec: "/bin/sh": stat /bin/sh: no such file or directory

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 麻烦这位同学确认一下 /tmp/busybox/bin/sh 文件是否存在。

##### **飞：
> gocker里面是如何编写的，请说明下把

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 18讲里有对源代码的详细剖析

##### **平：
> [root@VM_0_11_centos gocker]# gocker run -it -rootfs=/tmp/busybox /bin/sh2020/10/30 14:06:15 Current path is /tmp/busybox2020/10/30 14:06:15 CmdArray is [/bin/sh]2020/10/30 14:06:15 pivotRoot error pivot_root invalid argument2020/10/30 14:06:15 pivot_root invalid argument老师你好，请问下我这边按照你的步骤报这个错，希望老师帮忙看一下

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以先执行下 unshare -m 命令，然后再执行 gocker 命令就可以了

##### **辉：
> 执行go install 时报错runc/run.go:45:4: unknown field 'Cloneflags' in struct literal of type syscall.SysProcAttr
runc/run.go:97:24: undefined: syscall.MS_NOEXEC
runc/run.go:97:44: undefined: syscall.MS_NOSUID
runc/run.go:97:64: undefined: syscall.MS_NODEV
runc/run.go:98:3: undefined: syscall.Mount
runc/run.go:125:12: undefined: syscall.Mount
runc/run.go:125:46: undefined: syscall.MS_BIND
runc/run.go:125:62: undefined: syscall.MS_REC
runc/run.go:135:12: undefined: syscall.PivotRoot
runc/run.go:145:38: undefined: syscall.MNT_DETACH
runc/run.go:98:3: too many errors

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 操作系统不是 Linux 的吧，必须在 Linux 下执行或者设置 GOOS=linux

##### *冲：
> self 目录：它是连接到当前正在运行的进程目录，比如我当前的进程 ID 为 30097，则 self 目录实际连接到 /proc/30097 这个目录。这个当前进程id怎么看

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以通过 ps 命令查看进程 ID

