<p data-nodeid="8744">通过前面课程的学习，我们知道 Docker 镜像是分层的，并且每一层镜像都会额外占用存储空间，一个 Docker 镜像层数越多，这个镜像占用的存储空间则会越多。镜像构建最重要的一个原则就是要保持镜像体积尽可能小，要实现这个目标通常可以从两个方面入手：</p>
<ul data-nodeid="8745">
<li data-nodeid="8746">
<p data-nodeid="8747">基础镜像体积应该尽量小；</p>
</li>
<li data-nodeid="8748">
<p data-nodeid="8749">尽量减少 Dockerfile 的行数，因为 Dockerfile 的每一条指令都会生成一个镜像层。</p>
</li>
</ul>
<p data-nodeid="8750">在 Docker 的早期版本中，对于编译型语言（例如 C、Java、Go）的镜像构建，我们只能将应用的编译和运行环境的准备，全部都放到一个 Dockerfile 中，这就导致我们构建出来的镜像体积很大，从而增加了镜像的存储和分发成本，这显然与我们的镜像构建原则不符。</p>
<p data-nodeid="8751">为了减小镜像体积，我们需要借助一个额外的脚本，将镜像的编译过程和运行过程分开。</p>
<ul data-nodeid="8752">
<li data-nodeid="8753">
<p data-nodeid="8754">编译阶段：负责将我们的代码编译成可执行对象。</p>
</li>
<li data-nodeid="8755">
<p data-nodeid="8756">运行时构建阶段：准备应用程序运行的依赖环境，然后将编译后的可执行对象拷贝到镜像中。</p>
</li>
</ul>
<p data-nodeid="8757">我以 Go 语言开发的一个 HTTP 服务为例，代码如下：</p>
<pre class="lang-go" data-nodeid="8758"><code data-language="go"><span class="hljs-keyword">package</span> main
<span class="hljs-keyword">import</span> (
   <span class="hljs-string">"fmt"</span>
   <span class="hljs-string">"net/http"</span>
)
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">hello</span><span class="hljs-params">(w http.ResponseWriter, req *http.Request)</span></span> {
   fmt.Fprintf(w, <span class="hljs-string">"hello world!\n"</span>)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   http.HandleFunc(<span class="hljs-string">"/"</span>, hello)
   http.ListenAndServe(<span class="hljs-string">":8080"</span>, <span class="hljs-literal">nil</span>)
}
</code></pre>
<p data-nodeid="9624">我将这个 Go 服务构建成镜像分为两个阶段：代码的编译阶段和镜像构建阶段。</p>
<p data-nodeid="9625">我们构建镜像时，镜像中需要包含 Go 语言编译环境，应用的编译阶段我们可以使用 Dockerfile.build 文件来构建镜像。Dockerfile.build 的内容如下：</p>

<pre class="lang-go" data-nodeid="8760"><code data-language="go">FROM golang:<span class="hljs-number">1.13</span>
WORKDIR /<span class="hljs-keyword">go</span>/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.<span class="hljs-keyword">go</span> .
RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux <span class="hljs-keyword">go</span> build -o http-server .
</code></pre>
<p data-nodeid="8761">Dockerfile.build 可以帮助我们把代码编译成可以执行的二进制文件，我们使用以下 Dockerfile 构建一个运行环境：</p>
<pre class="lang-java" data-nodeid="8762"><code data-language="java">FROM alpine:latest  
WORKDIR /root/
COPY http-server .
CMD [<span class="hljs-string">"./http-server"</span>] 
</code></pre>
<p data-nodeid="8763">然后，我们将应用的编译和运行环境的准备步骤，都放到一个 build.sh 脚本文件中，内容如下：</p>
<pre class="lang-shell" data-nodeid="8764"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/bin/sh</span>
echo Building http-server:build
docker build -t http-server:build . -f Dockerfile.build
docker create --name builder http-server:build  
docker cp builder:/go/src/github.com/wilhelmguo/multi-stage-demo/http-server ./http-server  
docker rm -f builder
echo Building http-server:latest
docker build -t http-server:latest .
rm ./http-server
</code></pre>
<p data-nodeid="9976">下面，我带你来逐步分析下这个脚本。</p>
<p data-nodeid="9977">第一步，声明 shell 文件，然后输出开始构建信息。</p>

<pre class="lang-shell" data-nodeid="8766"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/bin/sh</span>
echo Building http-server:build
</code></pre>
<p data-nodeid="8767">第二步，使用 Dockerfile.build 文件来构建一个临时镜像 http-server:build。</p>
<pre class="lang-shell" data-nodeid="8768"><code data-language="shell">docker build -t http-server:build . -f Dockerfile.build
</code></pre>
<p data-nodeid="8769">第三步，使用 http-server:build 镜像创建一个名称为 builder 的容器，该容器包含编译后的 http-server 二进制文件。</p>
<pre class="lang-shell" data-nodeid="8770"><code data-language="shell">docker create --name builder http-server:build  
</code></pre>
<p data-nodeid="8771">第四步，使用<code data-backticks="1" data-nodeid="8850">docker cp</code>命令从 builder 容器中拷贝 http-server 文件到当前构建目录下，并且删除名称为 builder 的临时容器。</p>
<pre class="lang-shell" data-nodeid="8772"><code data-language="shell">docker cp builder:/go/src/github.com/wilhelmguo/multi-stage-demo/http-server ./http-server  
docker rm -f builder
</code></pre>
<p data-nodeid="8773">第五步，输出开始构建镜像信息。</p>
<pre class="lang-java" data-nodeid="8774"><code data-language="java">echo Building http-server:latest
</code></pre>
<p data-nodeid="8775">第六步，构建运行时镜像，然后删除临时文件 http-server。</p>
<pre class="lang-java" data-nodeid="8776"><code data-language="java">docker build -t http-server:latest .
rm ./http-server
</code></pre>
<p data-nodeid="8777">我这里总结一下，我们是使用 Dockerfile.build 文件来编译应用程序，使用 Dockerfile 文件来构建应用的运行环境。然后我们通过创建一个临时容器，把编译后的  http-server 文件拷贝到当前构建目录中，然后再把这个文件拷贝到运行环境的镜像中，最后指定容器的启动命令为  http-server。</p>
<p data-nodeid="8778">使用这种方式虽然可以实现分离镜像的编译和运行环境，但是我们需要额外引入一个 build.sh 脚本文件，而且构建过程中，还需要创建临时容器 builder 拷贝编译后的 http-server 文件，这使得整个构建过程比较复杂，并且整个构建过程也不够透明。</p>
<p data-nodeid="8779">为了解决这种问题， Docker 在 17.05 推出了多阶段构建（multistage-build）的解决方案。</p>
<h3 data-nodeid="8780">使用多阶段构建</h3>
<p data-nodeid="8781">Docker 允许我们在 Dockerfile 中使用多个 FROM 语句，而每个 FROM 语句都可以使用不同基础镜像。最终生成的镜像，是以最后一条 FROM 为准，所以我们可以在一个 Dockerfile 中声明多个 FROM，然后选择性地将一个阶段生成的文件拷贝到另外一个阶段中，从而实现最终的镜像只保留我们需要的环境和文件。多阶段构建的主要使用场景是<strong data-nodeid="8862">分离编译环境和运行环境。</strong></p>
<p data-nodeid="8782">接下来，我们使用多阶段构建的特性，将上述未使用多阶段构建的过程精简成如下 Dockerfile：</p>
<pre class="lang-java" data-nodeid="8783"><code data-language="java">FROM golang:<span class="hljs-number">1.13</span>
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.go .
RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux go build -o http-server .
FROM alpine:latest  
WORKDIR /root/
COPY --from=<span class="hljs-number">0</span> /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
CMD [<span class="hljs-string">"./http-server"</span>] 
</code></pre>
<p data-nodeid="8784">然后，我们将这个 Dockerfile 拆解成两步进行分析。</p>
<p data-nodeid="8785">第一步，编译代码。</p>
<pre class="lang-java" data-nodeid="8786"><code data-language="java">FROM golang:<span class="hljs-number">1.13</span>
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.go .
RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux go build -o http-server .
</code></pre>
<p data-nodeid="10328">将代码拷贝到 golang:1.13 镜像（已经安装好了 go）中，并且使用<code data-backticks="1" data-nodeid="10331">go build</code>命令编译代码生成 http-server 文件。</p>
<p data-nodeid="10329">第二步，构建运行时镜像。</p>

<pre class="lang-java" data-nodeid="8788"><code data-language="java">FROM alpine:latest  
WORKDIR /root/
COPY --from=<span class="hljs-number">0</span> /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
CMD [<span class="hljs-string">"./http-server"</span>]
</code></pre>
<p data-nodeid="10682">使用第二个 FROM 命令表示镜像构建的第二阶段，使用 COPY 指令拷贝编译后的文件到  alpine 镜像中，--from=0 表示从第一阶段构建结果中拷贝文件到当前构建阶段。</p>
<p data-nodeid="10683">最后，我们只需要使用以下命令，即可实现整个镜像的构建：</p>

<pre class="lang-java" data-nodeid="8790"><code data-language="java">docker build -t http-server:latest .
</code></pre>
<p data-nodeid="8791">构建出来的镜像与未使用多阶段构建之前构建的镜像大小一致，为了验证这一结论，我们分别使用这两种方式来构建镜像，最后对比一下镜像构建的结果。</p>
<h3 data-nodeid="8792">镜像构建对比</h3>
<p data-nodeid="8793">使用多阶段构建前后的代码我都已经放在了<a href="https://github.com/wilhelmguo/multi-stage-demo" data-nodeid="8879">Github</a>，你只需要克隆代码到本地即可。</p>
<pre class="lang-java" data-nodeid="8794"><code data-language="java">$ mkdir /go/src/github.com/wilhelmguo
$ cd /go/src/github.com/wilhelmguo
$ git clone https:<span class="hljs-comment">//github.com/wilhelmguo/multi-stage-demo.git</span>
</code></pre>
<p data-nodeid="8795">代码克隆完成后，我们首先切换到without-multi-stage分支：</p>
<pre class="lang-java" data-nodeid="8796"><code data-language="java">$ cd without-multi-stage
$ git checkout without-multi-stage
</code></pre>
<p data-nodeid="8797">这个分支是未使用多阶段构建技术构建镜像的代码，我们可以通过执行 build.sh 文件构建镜像：</p>
<pre class="lang-java" data-nodeid="8798"><code data-language="java">$ &nbsp;chmod +x build.sh &amp;&amp; ./build.sh
Building http-server:build
Sending build context to Docker daemon&nbsp; <span class="hljs-number">96.26</span>kB
Step <span class="hljs-number">1</span>/<span class="hljs-number">4</span> : FROM golang:<span class="hljs-number">1.13</span>
<span class="hljs-number">1.13</span>: Pulling from library/golang
d6ff36c9ec48: Pull complete&nbsp;
c958d65b3090: Pull complete&nbsp;
edaf0a6b092f: Pull complete&nbsp;
<span class="hljs-number">80931</span>cf68816: Pull complete&nbsp;
<span class="hljs-number">813643441356</span>: Pull complete&nbsp;
<span class="hljs-number">799f</span>41bb59c9: Pull complete&nbsp;
<span class="hljs-number">16</span>b5038bccc8: Pull complete&nbsp;
Digest: sha256:<span class="hljs-number">8</span>ebb6d5a48deef738381b56b1d4cd33d99a5d608e0d03c5fe8dfa3f68d41a1f8
Status: Downloaded newer image <span class="hljs-keyword">for</span> golang:<span class="hljs-number">1.13</span>
&nbsp;---&gt; d6f3656320fe
Step <span class="hljs-number">2</span>/<span class="hljs-number">4</span> : WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
&nbsp;---&gt; Running in fa3da5ffb0c0
Removing intermediate container fa3da5ffb0c0
&nbsp;---&gt; <span class="hljs-number">97245</span>cbb773f
Step <span class="hljs-number">3</span>/<span class="hljs-number">4</span> : COPY main.go .
&nbsp;---&gt; a021d2f2a5bb
Step <span class="hljs-number">4</span>/<span class="hljs-number">4</span> : RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux go build -o http-server .
&nbsp;---&gt; Running in b5c36bb67b9c
Removing intermediate container b5c36bb67b9c
&nbsp;---&gt; <span class="hljs-number">76</span>c0c88a5cf7
Successfully built <span class="hljs-number">76</span>c0c88a5cf7
Successfully tagged http-server:build
<span class="hljs-number">4</span>b0387b270bc4a4da570e1667fe6f9baac765f6b80c68f32007494c6255d9e5b
builder
Building http-server:latest
Sending build context to Docker daemon&nbsp; <span class="hljs-number">7.496</span>MB
Step <span class="hljs-number">1</span>/<span class="hljs-number">4</span> : FROM alpine:latest
latest: Pulling from library/alpine
df20fa9351a1: Already exists&nbsp;
Digest: sha256:<span class="hljs-number">185518070891758909</span>c9f839cf4ca393ee977ac378609f700f60a771a2dfe321
Status: Downloaded newer image <span class="hljs-keyword">for</span> alpine:latest
&nbsp;---&gt; a24bb4013296
Step <span class="hljs-number">2</span>/<span class="hljs-number">4</span> : WORKDIR /root/
&nbsp;---&gt; Running in <span class="hljs-number">0</span>b25ffe603b8
Removing intermediate container <span class="hljs-number">0</span>b25ffe603b8
&nbsp;---&gt; <span class="hljs-number">80</span>da40d3a0b4
Step <span class="hljs-number">3</span>/<span class="hljs-number">4</span> : COPY http-server .
&nbsp;---&gt; <span class="hljs-number">3f</span>2300210b7b
Step <span class="hljs-number">4</span>/<span class="hljs-number">4</span> : CMD [<span class="hljs-string">"./http-server"</span>]
&nbsp;---&gt; Running in <span class="hljs-number">045</span>cea651dde
Removing intermediate container <span class="hljs-number">045</span>cea651dde
&nbsp;---&gt; <span class="hljs-number">5</span>c73883177e7
Successfully built <span class="hljs-number">5</span>c73883177e7
Successfully tagged http-server:latest
</code></pre>
<p data-nodeid="8799">经过一段时间的等待，我们的镜像就构建完成了。<br>
镜像构建完成后，我们使用<code data-backticks="1" data-nodeid="8886">docker image ls</code>命令查看一下刚才构建的镜像大小：</p>
<pre class="lang-java" data-nodeid="8800"><code data-language="java">$ docker image ls http-server
REPOSITORY&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TAG&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;IMAGE ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; CREATED&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;SIZE
http-server&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;latest&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">5</span>c73883177e7&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">3</span> minutes ago&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">13</span>MB
http-server&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;build&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">76</span>c0c88a5cf7&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">3</span> minutes ago&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">819</span>MB
</code></pre>
<p data-nodeid="11034">可以看到，http-server:latest 镜像只有 13M，而我们的编译镜像 http-server:build 则为 819M，虽然我们编写了很复杂的脚本 build.sh，但是这个脚本确实帮助我们将镜像体积减小了很多。</p>
<p data-nodeid="11035">下面，我们将代码切换到多阶段构建分支：</p>

<pre class="lang-java" data-nodeid="8802"><code data-language="java">$ git checkout with-multi-stage
Switched to branch <span class="hljs-string">'with-multi-stage'</span>
</code></pre>
<p data-nodeid="8803">为了避免镜像名称重复，我们将多阶段构建的镜像命名为 http-server-with-multi-stage:latest ，并且禁用缓存，避免缓存干扰构建结果，构建命令如下：</p>
<pre class="lang-java" data-nodeid="8804"><code data-language="java">$ docker build --no-cache -t http-server-with-multi-stage:latest .
Sending build context to Docker daemon&nbsp; <span class="hljs-number">96.77</span>kB
Step <span class="hljs-number">1</span>/<span class="hljs-number">8</span> : FROM golang:<span class="hljs-number">1.13</span>
&nbsp;---&gt; d6f3656320fe
Step <span class="hljs-number">2</span>/<span class="hljs-number">8</span> : WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
&nbsp;---&gt; Running in <span class="hljs-number">640</span>da7a92a62
Removing intermediate container <span class="hljs-number">640</span>da7a92a62
&nbsp;---&gt; <span class="hljs-number">9</span>c27b4606da0
Step <span class="hljs-number">3</span>/<span class="hljs-number">8</span> : COPY main.go .
&nbsp;---&gt; bd9ce4af24cb
Step <span class="hljs-number">4</span>/<span class="hljs-number">8</span> : RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux go build -o http-server .
&nbsp;---&gt; Running in <span class="hljs-number">6</span>b441b4cc6b7
Removing intermediate container <span class="hljs-number">6</span>b441b4cc6b7
&nbsp;---&gt; <span class="hljs-number">759</span>acbf6c9a6
Step <span class="hljs-number">5</span>/<span class="hljs-number">8</span> : FROM alpine:latest
&nbsp;---&gt; a24bb4013296
Step <span class="hljs-number">6</span>/<span class="hljs-number">8</span> : WORKDIR /root/
&nbsp;---&gt; Running in c2aa2168acd8
Removing intermediate container c2aa2168acd8
&nbsp;---&gt; f026884acda6
Step <span class="hljs-number">7</span>/<span class="hljs-number">8</span> : COPY --from=<span class="hljs-number">0</span> /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
&nbsp;---&gt; <span class="hljs-number">667503e6</span>bc14
Step <span class="hljs-number">8</span>/<span class="hljs-number">8</span> : CMD [<span class="hljs-string">"./http-server"</span>]
&nbsp;---&gt; Running in <span class="hljs-number">15</span>c4cc359144
Removing intermediate container <span class="hljs-number">15</span>c4cc359144
&nbsp;---&gt; b73cc4d99088
Successfully built b73cc4d99088
Successfully tagged http-server-with-multi-stage:latest
</code></pre>
<p data-nodeid="8805">镜像构建完成后，我们同样使用<code data-backticks="1" data-nodeid="8893">docker image ls</code>命令查看一下镜像构建结果：</p>
<pre class="lang-java" data-nodeid="8806"><code data-language="java">$ docker image ls http-server-with-multi-stage:latest
REPOSITORY&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;TAG&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;IMAGE ID&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; CREATED&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;SIZE
http-server-with-multi-stage&nbsp; &nbsp;latest&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; b73cc4d99088&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">2</span> minutes ago&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">13</span>MB
</code></pre>
<p data-nodeid="8807">可以看到，使用多阶段构建的镜像大小与上一步构建的镜像大小一致，都为 13M。但是使用多阶段构建后，却大大减少了我们的构建步骤，使得构建过程更加清晰可读。</p>
<h3 data-nodeid="8808">多阶段构建的其他使用方式</h3>
<p data-nodeid="8809">多阶段构建除了我们上面讲解的使用方式，还有更多其他的使用方式，这些使用方式，可以使得多阶段构建实现更多的功能。</p>
<h4 data-nodeid="8810">为构建阶段命名</h4>
<p data-nodeid="8811">默认情况下，每一个构建阶段都没有被命名，你可以通过 FROM 指令出现的顺序来引用这些构建阶段，构建阶段的序号是从 0 开始的。然而，为了提高 Dockerfile 的可读性，我们需要为某些构建阶段起一个名称，这样即便后面我们对 Dockerfile 中的内容进程重新排序或者添加了新的构建阶段，其他构建过程中的  COPY 指令也不需要修改。</p>
<p data-nodeid="8812">上面的 Dockerfile 我们可以优化成如下内容：</p>
<pre class="lang-java" data-nodeid="8813"><code data-language="java">FROM golang:<span class="hljs-number">1.13</span> AS builder
WORKDIR /go/src/github.com/wilhelmguo/multi-stage-demo/
COPY main.go .
RUN CGO_ENABLED=<span class="hljs-number">0</span> GOOS=linux go build -o http-server .

FROM alpine:latest  
WORKDIR /root/
COPY --from=builder /go/src/github.com/wilhelmguo/multi-stage-demo/http-server .
CMD [<span class="hljs-string">"./http-server"</span>]
</code></pre>
<p data-nodeid="8814">我们在第一个构建阶段，使用 AS 指令将这个阶段命名为 builder。然后在第二个构建阶段使用 --from=builder 指令，即可从第一个构建阶段中拷贝文件，使得 Dockerfile 更加清晰可读。</p>
<h4 data-nodeid="8815">停止在特定的构建阶段</h4>
<p data-nodeid="8816">有时候，我们的构建阶段非常复杂，我们想在代码编译阶段进行调试，但是多阶段构建默认构建 Dockerfile 的所有阶段，为了减少每次调试的构建时间，我们可以使用 target 参数来指定构建停止的阶段。</p>
<p data-nodeid="8817">例如，我只想在编译阶段调试 Dockerfile 文件，可以使用如下命令：</p>
<pre class="lang-java" data-nodeid="8818"><code data-language="java">$ docker build --target&nbsp;builder -t http-server:latest .
</code></pre>
<p data-nodeid="8819">在执行<code data-backticks="1" data-nodeid="8906">docker build</code>命令时添加 target 参数，可以将构建阶段停止在指定阶段，从而方便我们调试代码编译过程。</p>
<h4 data-nodeid="8820">使用现有镜像作为构建阶段</h4>
<p data-nodeid="8821">使用多阶段构建时，不仅可以从 Dockerfile 中已经定义的阶段中拷贝文件，还可以使用<code data-backticks="1" data-nodeid="8910">COPY --from</code>指令从一个指定的镜像中拷贝文件，指定的镜像可以是本地已经存在的镜像，也可以是远程镜像仓库上的镜像。</p>
<p data-nodeid="8822">例如，当我们想要拷贝 nginx 官方镜像的配置文件到我们自己的镜像中时，可以在 Dockerfile 中使用以下指令：</p>
<pre class="lang-java" data-nodeid="8823"><code data-language="java">COPY --from=nginx:latest /etc/nginx/nginx.conf /etc/local/nginx.conf
</code></pre>
<p data-nodeid="8824">从现有镜像中拷贝文件还有一些其他的使用场景。例如，有些工具没有我们使用的操作系统的安装源，或者安装源太老，需要我们自己下载源码并编译这些工具，但是这些工具可能依赖的编译环境非常复杂，而网上又有别人已经编译好的镜像。这时我们就可以使用<code data-backticks="1" data-nodeid="8914">COPY --from</code>指令从编译好的镜像中将工具拷贝到我们自己的镜像中，很方便地使用这些工具了。</p>
<h3 data-nodeid="8825">结语</h3>
<p data-nodeid="8826">多阶段构建可以让我们通过一个 Dockerfile 很方便地构建出体积更小的镜像，并且我们只需要编写 Dockerfile 文件即可，无须借助外部脚本文件。这使得镜像构建过程更加简单透明，但要提醒一点：使用多阶段构建的唯一限制条件是我们使用的 Docker 版本必须高于 17.05 。</p>
<p data-nodeid="11386">那么，你知道多阶段构建还有哪些应用场景吗？欢迎评论区留言讨论。</p>

---

### 精选评论

##### **德：
> github地址上面的内容为空

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 同学记得切换分支，切换分支就有了，步骤上有喔

##### **达：
> 想问一下像go这种编译完可以直接把二进制文件拿走运行，那像python，java这种需要安装运行环境的有没有优化的方法啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; python 这种只能再镜像中安装 python 运行环境了，java 可以把 jdk 做为编译环境，然后把编译完的包放到 jre 环境中即可，只有 jre 的环境镜像体积会小很多。

##### *帅：
> 网上又有别人已经编译好的镜像。这时我们就可以使用COPY --from指令从编译好的镜像中将工具拷贝到我们自己的镜像中这种情况虽然方便，出于安全性考虑的话，应该还是得经过充分验证才能使用吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，考虑安全性的话，是需要经过充分验证的，可以结合镜像扫描工具验证其安全性

##### **升：
> 打卡

##### **6250：
> RHEL分支维护的docker版本，支持多阶级构建吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Docker 版本大于 17.05 都是可以的

