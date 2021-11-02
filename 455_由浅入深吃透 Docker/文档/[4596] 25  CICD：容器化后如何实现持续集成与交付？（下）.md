<p data-nodeid="92472">上一讲，我介绍了 CI 和 CD 的相关概念，并且使用 Docker+Jenkins+GitLab 搭建了我们的 CI/CD 环境，今天我们就来使用已经构建好的环境来实际构建和部署一个应用。</p>
<p data-nodeid="92473">构建和部署一个应用的流程可以分为五部分。</p>
<ol data-nodeid="92474">
<li data-nodeid="92475">
<p data-nodeid="92476">我们首先需要配置 GitLab SSH 访问公钥，使得我们可以直接通过 SSH 拉取或推送代码到 GitLab。</p>
</li>
<li data-nodeid="92477">
<p data-nodeid="92478">接着将代码通过 SSH 上传到 GitLab。</p>
</li>
<li data-nodeid="92479">
<p data-nodeid="92480">再在 Jenkins 创建构建任务，使得 Jenkins 可以成功拉取 GitLab 的代码并进行构建。</p>
</li>
<li data-nodeid="92481">
<p data-nodeid="92482">然后配置代码变更自动构建流程，使得代码变更可以触发自动构建 Docker 镜像。</p>
</li>
<li data-nodeid="92483">
<p data-nodeid="92484">最后配置自动部署流程，镜像构建完成后自动将镜像发布到测试或生产环境。</p>
</li>
</ol>
<p data-nodeid="92485">接下来我们逐一操作。</p>
<h3 data-nodeid="93592" class="">1. 配置 GitLab SSH 访问公钥</h3>

<p data-nodeid="92489">为了能够让 Jenkins 顺利从 GitLab 拉取代码，我们需要先生成 ssh 密钥。我们可以使用 ssh-keygen 命令来生成 2048 位的 ras 密钥。在 Linux 上执行如下命令：</p>
<pre class="lang-java" data-nodeid="92490"><code data-language="java">$ ssh-keygen -o -t rsa -b 2048 -C "email@example.com"
# 输入上面命令后系统会提示我们密钥保存的位置等信息，只需要按回车即可。
Generating public/private rsa key pair.
Enter file in which to save the key (/home/centos/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/centos/.ssh/id_rsa.
Your public key has been saved in /home/centos/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:A+d0NQQrjxV2h+zR3BQIJxT23puXoLi1RiTKJm16+rg email@example.com
The key's randomart image is:
+---[RSA 2048]----+
|&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; =XB=o+o|
|&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;..=B+o .|
|&nbsp; &nbsp; &nbsp; . + +. o&nbsp; &nbsp;|
|&nbsp; &nbsp; &nbsp; &nbsp;= B .o .&nbsp; |
|&nbsp; &nbsp; &nbsp; o S +&nbsp; o . |
|&nbsp; &nbsp; &nbsp;. * .... . +|
|&nbsp; &nbsp; &nbsp; =&nbsp; ..o&nbsp; &nbsp;+.|
|&nbsp; &nbsp; &nbsp;...&nbsp; o..&nbsp; &nbsp;.|
|&nbsp; &nbsp; &nbsp;E=. ...&nbsp; &nbsp; &nbsp;|
+----[SHA256]-----+
</code></pre>
<p data-nodeid="92491">执行完上述命令后 ，$HOME/.ssh/ 目录下会自动生成两个文件：id_rsa.pub 文件为公钥文件，id_rsa 文件为私钥文件。我们可以通过 cat 命令来查看公钥文件内容：</p>
<pre class="lang-java" data-nodeid="92492"><code data-language="java">$ cat $HOME/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDljSlDNHnUr4ursYISKXK5j2mWTYnt100mvYeJCLpr6tpeSarGyr7FnTc6sLM721plU2xq0bqlFEU5/<span class="hljs-number">0</span>SSvFdLTht7bcfm/Hf31EdAuIqZuy/guP06ijpidfX6lVDxLWx/sO3Wbj3t7xgj4sfCFTiv+OOFP0NxKr5wy+emojm6KIaXkhjbPeJDgph5bvluFnKAtesMUkdhceAdN9grE3nkBOnwWw6G4dCtbrKt2o9wSyzgkDwPjj2qjFhcE9571/<span class="hljs-number">61</span>/Nr8v9iqSHvcb/d7WZ0Qq7a2LYds6hQkpBg2RCDDJA16fFVs8Q5eNCpDQwGG3IbhHMUwvpKDf0OYrS9iftc5 email<span class="hljs-meta">@example</span>.com
</code></pre>
<p data-nodeid="94468">然后将公钥文件拷贝到 GitLab 的个人设置 -&gt; SSH Keys 中，点击添加按钮，将我们的公钥添加到 GitLab 中。</p>
<p data-nodeid="94469" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A8/CgqCHl-2P_qAO6VIAAIAcpA55IY226.png" alt="Drawing 0.png" data-nodeid="94473"></p>


<h3 data-nodeid="95358" class="">2. 上传服务代码到 GitLab</h3>


<p data-nodeid="92497">这里，我使用 Golang 编写了一个 HTTP 服务，代码如下：</p>
<pre class="lang-go" data-nodeid="92498"><code data-language="go"><span class="hljs-keyword">package</span> main

<span class="hljs-keyword">import</span> (
&nbsp; &nbsp; <span class="hljs-string">"fmt"</span>
&nbsp; &nbsp; <span class="hljs-string">"net/http"</span>
)

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">hello</span><span class="hljs-params">(w http.ResponseWriter, req *http.Request)</span></span> {

&nbsp; &nbsp; fmt.Fprintf(w, <span class="hljs-string">"hello\n"</span>)
}

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">headers</span><span class="hljs-params">(w http.ResponseWriter, req *http.Request)</span></span> {

&nbsp; &nbsp; <span class="hljs-keyword">for</span> name, headers := <span class="hljs-keyword">range</span> req.Header {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> _, h := <span class="hljs-keyword">range</span> headers {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fmt.Fprintf(w, <span class="hljs-string">"%v: %v\n"</span>, name, h)
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}

<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {

&nbsp; &nbsp; http.HandleFunc(<span class="hljs-string">"/hello"</span>, hello)
&nbsp; &nbsp; http.HandleFunc(<span class="hljs-string">"/headers"</span>, headers)

&nbsp; &nbsp; http.ListenAndServe(<span class="hljs-string">":8090"</span>, <span class="hljs-literal">nil</span>)
}
</code></pre>
<p data-nodeid="92499">然后编写一个 Dockerfile，利用多阶段构建将我们的 Go 编译，并将编译后的二进制文件拷贝到 scratch（scratch 是一个空镜像，用于构建其他镜像，体积非常小）的基础镜像中。Dockerfile 的内容如下：</p>
<pre class="lang-dockerfile" data-nodeid="92500"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> golang:<span class="hljs-number">1.14</span> as builder
<span class="hljs-keyword">WORKDIR</span><span class="bash"> /go/src/github.com/wilhelmguo/devops-demo/</span>
<span class="hljs-keyword">COPY</span><span class="bash"> main.go .</span>
<span class="hljs-keyword">RUN</span><span class="bash"> CGO_ENABLED=0 GOOS=linux go build -o /tmp/http-server .</span>
<span class="hljs-keyword">FROM</span> scratch
<span class="hljs-keyword">WORKDIR</span><span class="bash"> /root/</span>
<span class="hljs-keyword">COPY</span><span class="bash"> --from=builder /tmp/http-server .</span>
<span class="hljs-keyword">CMD</span><span class="bash"> [<span class="hljs-string">"./http-server"</span>]</span>
</code></pre>
<p data-nodeid="92501">编写完 Go HTTP 文件和 Dockerfile 文件后，代码目录内容如下：</p>
<pre class="lang-java" data-nodeid="92502"><code data-language="java">$ ls -lh
total <span class="hljs-number">24</span>
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; root&nbsp; &nbsp;<span class="hljs-number">243</span>B Nov&nbsp; <span class="hljs-number">3</span> <span class="hljs-number">22</span>:<span class="hljs-number">03</span> Dockerfile
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; root&nbsp; &nbsp; <span class="hljs-number">26</span>B Nov&nbsp; <span class="hljs-number">3</span> <span class="hljs-number">22</span>:<span class="hljs-number">06</span> README.md
-rw-r--r--&nbsp; <span class="hljs-number">1</span> root&nbsp; root&nbsp; &nbsp;<span class="hljs-number">441</span>B Nov&nbsp; <span class="hljs-number">3</span> <span class="hljs-number">22</span>:<span class="hljs-number">03</span> main.go
</code></pre>
<blockquote data-nodeid="92503">
<p data-nodeid="92504">源码详见<a href="https://github.com/wilhelmguo/devops-demo" data-nodeid="92596">这里</a></p>
</blockquote>
<p data-nodeid="96216">然后，我们在 GitLab 上创建一个 hello 项目，并将代码上传。</p>
<p data-nodeid="96868" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A8/CgqCHl-2QA2ARz39AADE_fukgio780.png" alt="Drawing 1.png" data-nodeid="96871"><br>
<img src="https://s0.lgstatic.com/i/image/M00/6F/A9/CgqCHl-2QQCAZUxWAAF7KHvN2DI582.png" alt="Drawing 2.png" data-nodeid="96875"></p>





<p data-nodeid="97740">项目创建完成后，GitLab 会自动跳转到项目详情页面。</p>
<p data-nodeid="97741" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9D/Ciqc1F-2QQeAXsbVAAELrFGkphU008.png" alt="Drawing 3.png" data-nodeid="97745"></p>


<h3 data-nodeid="99492" class="">3. 创建 Jenkins 任务</h3>


<p data-nodeid="98610">在 Jenkins 中添加一个自由风格的任务。</p>
<p data-nodeid="98611" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9D/Ciqc1F-2QRGAIS83AAGKHDb05xE232.png" alt="Drawing 4.png" data-nodeid="98615"></p>


<p data-nodeid="100352">点击确定，然后到源码管理选择 Git，填写 GitLab 项目的 URL。此时 Jenkins 会提示没有访问 GitLab 的相关权限，我们需要点击添加按钮将私钥添加到 Jenkins 中用以鉴权。</p>
<p data-nodeid="100353" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9D/Ciqc1F-2QSWAceMNAADnnjcKzCo548.png" alt="Drawing 5.png" data-nodeid="100357"></p>



<blockquote data-nodeid="92517">
<p data-nodeid="92518">由于部署 GitLab 的宿主机 ssh 默认端口为 22，为了避免与宿主机的 ssh 端口冲突，我们的 GitLab ssh 端口配置为 2222，因此 Jenkins 连接 GitLab 的 URL 中需要包含端口号 2222， 配置格式为 ssh://git@172.20.1.6:2222/root/hello.git。</p>
</blockquote>
<p data-nodeid="101214">选择添加的密钥类型为 "SSH Username with private key"，Username 设置为 jenkins，然后将私钥粘贴到 Private Key 输入框中，点击添加即可。</p>
<p data-nodeid="101215" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9D/Ciqc1F-2QTSARpg5AAET_4BGb-0066.png" alt="Drawing 6.png" data-nodeid="101223"></p>


<p data-nodeid="102080">添加完成后，认证名称选择 jenkins 后，红色报错提示就会消失。这证明此时 Jenkins 和 GitLab 已经认证成功，可以成功从 GitLab 拉取代码了。</p>
<p data-nodeid="102081" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A9/CgqCHl-2QTqAQf8RAACXxIBN-Z8663.png" alt="Drawing 7.png" data-nodeid="102085"></p>


<p data-nodeid="92523">下面我们使用 shell 脚本来构建我们的应用镜像，在构建中增加一个 Shell 类型的构建步骤，并且填入以下信息，将 USER 替换为目标镜像仓库的用户名，将 PASSWORD 替换为镜像仓库的密码。</p>
<pre class="lang-java" data-nodeid="102942"><code data-language="java"># 第一步，登录镜像仓库
$ docker login -u {USER} -p&nbsp; {PASSWORD}
# 第二步，使用 docker build 命令构建镜像
$ docker build -t lagoudocker/devops-demo .&nbsp;
# 第三步, 使用 docker push 命令推送镜像
$ docker push lagoudocker/devops-demo&nbsp;
</code></pre>
<p data-nodeid="102943" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A9/CgqCHl-2QUKAJ-psAABwghmp76g949.png" alt="Drawing 8.png" data-nodeid="102946"></p>


<p data-nodeid="104239">完成后点击保存，此时任务已经成功添加到 Jenkins 中。回到任务首页，点击构建按钮即可开始构建。第一次构建需要下载依赖的基础镜像，这个过程可能比较慢。构建过程中，我们也可以点击控制台查看构建输出的内容：</p>
<p data-nodeid="104240" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9D/Ciqc1F-2QUuAEXcXAAGe5l9e2h0928.png" alt="Drawing 9.png" data-nodeid="104244"></p>



<h3 data-nodeid="105113" class="">4. 配置自动构建</h3>


<p data-nodeid="105963">点击上一步创建的任务，点击配置进入任务配置界面，到构建触发器下勾选 GitLab 相关的选项，点击 Generate 按钮生成一个 GitLab 回调 Jenkins 的 token。记录下 Jenkins 的回调地址和生成的 token 信息。</p>
<p data-nodeid="105964" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9E/Ciqc1F-2QWCABHzrAAFQCgpFnLs787.png" alt="Drawing 10.png" data-nodeid="105968"></p>


<p data-nodeid="106817">在 GitLab 项目设置中，选择 Webhooks，将 Jenkins 的回调地址和 token 信息添加到 Webhooks 的配置中，点击添加即可。</p>
<p data-nodeid="106818" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/9E/Ciqc1F-2QWiAFOVBAAI93Lelr38996.png" alt="Drawing 11.png" data-nodeid="106822"></p>


<p data-nodeid="92535">后面我们的每次提交都会触发自动构建。</p>
<p data-nodeid="92536">为了实现根据 git 的 tag 自动构建相应版本的镜像，我们需要修改 Jenkins 构建步骤中的 shell 脚本为以下内容：</p>
<pre class="lang-shell" data-nodeid="92537"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> 需要推送的镜像名称</span>
IMAGE_NAME="lagoudocker/devops-demo" 
<span class="hljs-meta">#</span><span class="bash"> 获取当前构建的版本号</span>
GIT_VERSION=`git describe --always --tag`
<span class="hljs-meta">#</span><span class="bash"> 生成完整的镜像 URL 变量，用于构建和推送镜像</span>
REPOSITORY=docker.io/${IMAGE_NAME}:${GIT_VERSION} 
<span class="hljs-meta">#</span><span class="bash"> 构建Docker镜像 </span>
docker build -t $REPOSITORY -f Dockerfile . 
<span class="hljs-meta">#</span><span class="bash"> 登录镜像仓库，username 跟 password 为目标镜像仓库的用户名和密码</span>
docker login --username=xxxxx --password=xxxxxx docker.io
<span class="hljs-meta">#</span><span class="bash"> 推送 Docker 镜像到目标镜像仓库</span>
docker push $REPOSITORY 
</code></pre>
<p data-nodeid="92538">好了，到此我们已经完成了 GitLab -&gt; Jenkins -&gt; Docker 镜像仓库的自动构建和推送。当我们推送代码到 GitLab 中时，会自动触发 Webhooks，然后 GitLab 会根据配置的 Webhooks 调用 Jenkins 开始构建镜像，镜像构建完成后自动将镜像上传到我们的镜像仓库。</p>
<h3 data-nodeid="107683" class="">5. 配置自动部署</h3>


<p data-nodeid="92542">镜像构建完成后，我们还需要将镜像发布到测试或生产环境中将镜像运行起来。发布到环境的过程可以设置为自动发布，每当我们推送代码到 master 中时，即开始自动构建镜像，并将构建后的镜像发布到测试环境中。</p>
<p data-nodeid="92543">在镜像构建过程中，实际上 Jenkins 是通过执行我们编写的 shell 脚本完成的，要想实现镜像构建完成后自动在远程服务器上运行最新的镜像，我们需要借助一个 Jenkins 插件 Publish Over SSH，这个插件可以帮助我们自动登录远程服务器，并执行一段脚本将我们的服务启动。</p>
<p data-nodeid="92544">下面我们来实际操作下这个插件。</p>
<p data-nodeid="108959"><strong data-nodeid="108965">第一步，在 Jenkins 中安装 Publish Over SSH 插件。</strong> 在 Jenkins 系统管理，插件管理中，搜索 Publish Over SSH，然后点击安装并重启 Jenkins 服务。</p>
<p data-nodeid="108960" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A9/CgqCHl-2QfmAc4iBAACDzvOoPWI585.png" alt="Drawing 12.png" data-nodeid="108968"></p>



<p data-nodeid="110249"><strong data-nodeid="110255">第二步，配置 Publish Over SSH 插件。</strong> 插件安装完成后，在 Jenkins 系统管理的系统设置下，找到 Publish Over SSH 功能模块，添加远程服务器节点，这里我使用密码验证的方式添加一台服务器。配置好后，我们可以使用测试按钮测试服务器是否可以正常连接，显示Success 代表服务器可以正常连接，测试连接成功后，点击保存按钮保存配置。</p>
<p data-nodeid="110250" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/A9/CgqCHl-2QgSAVk0bAAC6abody2k836.png" alt="Drawing 13.png" data-nodeid="110258"></p>



<p data-nodeid="110687" class=""><strong data-nodeid="110692">第三步，修改之前 shell 任务中脚本，</strong> 添加部署相关的内容：</p>

<pre class="lang-shell" data-nodeid="92550"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> 需要推送的镜像名称</span>
IMAGE_NAME="lagoudocker/devops-demo" 
<span class="hljs-meta">#</span><span class="bash"> 获取当前构建的版本号</span>
GIT_VERSION=`git describe --always --tag`
<span class="hljs-meta">#</span><span class="bash"> 生成完整的镜像 URL 变量，用于构建和推送镜像</span>
REPOSITORY=docker.io/${IMAGE_NAME}:${GIT_VERSION} 
<span class="hljs-meta">#</span><span class="bash"> 构建Docker镜像 </span>
docker build -t $REPOSITORY -f Dockerfile . 
<span class="hljs-meta">#</span><span class="bash"> 登录镜像仓库，username 跟 password 为目标镜像仓库的用户名和密码</span>
docker login --username={USER} --password={PASSWORD} docker.io
<span class="hljs-meta">#</span><span class="bash"> 推送 Docker 镜像到目标镜像仓库</span>
docker push $REPOSITORY 
mkdir -p ./shell &amp;&amp; echo \ 
"docker login --username={USER} --password={PASSWORD} \n"\ 
"docker pull $REPOSITORY\n"\ 
"docker kill hello \n"\&nbsp;
"docker run --rm --name=hello -p 8090:8090 -d $REPOSITORY" &gt;&gt; ./shell/release
</code></pre>
<p data-nodeid="111123">我们在 docker push 命令后，增加一个输出 shell 脚本到 release 文件的命令，这个脚本会发送到远端的服务器上并执行，通过执行这个脚本文件可以在远端服务器上，拉取最新镜像并且重新启动容器。</p>
<p data-nodeid="111988"><strong data-nodeid="111999">第四步，配置远程执行。<strong data-nodeid="111998">在 Jenkins 的 hello 项目中，点击配置，在执行步骤中点击添加</strong>Send files or execute commands over SSH</strong>的步骤，选择之前添加的服务器，并且按照以下内容填写相关信息。</p>
<p data-nodeid="111989" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/6F/9E/Ciqc1F-2QhKAPblBAAC4Bp33K2Y632.png" alt="Drawing 14.png" data-nodeid="112002"></p>



<ul data-nodeid="92553">
<li data-nodeid="92554">
<p data-nodeid="92555">Source file 就是我们要传递的 shell 脚本信息，这里填写我们上面生成的 shell 脚本文件即可。</p>
</li>
<li data-nodeid="92556">
<p data-nodeid="92557">Remove prefix 是需要过滤的目录，这里我们填写 shell。</p>
</li>
<li data-nodeid="92558">
<p data-nodeid="92559">Remote directory 为远程执行脚本的目录。</p>
</li>
</ul>
<p data-nodeid="92560">最后点击保存，保存我们的配置即可。配置完成后，我们就完成了推送代码到 GitLab，Jenkins 自动构建镜像，之后推送镜像到镜像仓库，最后自动在远程服务器上拉取并重新部署容器。</p>
<blockquote data-nodeid="92561">
<p data-nodeid="92562">如果你是生产环境中使用的 Kubernetes 管理服务，可以在 Jenkins 中安装 Kubernetes 的插件，然后构建完成后直接发布镜像到 Kubernetes 集群中。</p>
</blockquote>
<h3 data-nodeid="92563">结语</h3>
<p data-nodeid="92564">本课时我们使用 Go 开发了一个简单的 HTTP 服务，并将代码托管在了 GitLab 中。然后通过配置 GitLab 和 Jenkins 的相互调用，实现了推送代码到 GitLab 代码仓库自动触发构建镜像并将镜像推送到远程镜像仓库中，最后将最新版本镜像发布到远程服务器上。</p>
<p data-nodeid="92565">DevOps 是一个非常棒的指导思想，而 CI/CD 是整个 DevOps 流程中最重要的部分，目前 CI/CD 的市场已经非常成熟，CI/CD 的工具链也非常完善，因此，无论是小团队还是大团队，都有必要去学习和掌握 CI/CD，以便帮助我们改善团队的效能，一切可以自动化的流程，都应该尽量避免人工参与。</p>
<p data-nodeid="92566">那么，你知道如何使用 Jenkins 将构建后的镜像发布到 Kubernetes 中吗?</p>

---

### 精选评论

##### **生：
> 不知道啊，你倒是说说怎么使用 Jenkins 将构建后的镜像发布到 Kubernetes 中

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Jenkins 有 Kubernetes 的插件，可以调用 Kubernetes 接口发布到 Kubernetes 中

##### *冲：
> 第一步，登录镜像仓库$ docker login -u {USER} -p {PASSWORD}# 第二步，使用 docker build 命令构建镜像$ docker build -t lagoudocker/devops-demo .# 第三步, 使用 docker push 命令推送镜像$ docker push lagoudocker/devops-demo 这些命令是在jenkins里敲的命令吗？构建那个图是哪里？找不到。。。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在构建-> 增加构建步骤，使用 shell ，就可以看到页面新增加了一个 shell 框

##### **帆：
> 老师，docker 能搭建一个 Flutter 的开发环境吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以，Flutter 相关的环境搭建可以参考 Flutter 官网哈~ https://flutter.dev/docs/deployment/cd

##### **龙：
> 老师，我基于gitlab CI/CD做的自动部署，docker push到gitlab的镜像仓库时特别慢，有没有什么解决方案呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; gitlab 并没有镜像仓库呀？这位同学说的是推送镜像到 docker hub 慢么？如果是推送到 docker hub 慢的话可以给 docker 添加代理。

##### **磊：
> 其实没必要用 jekins，gitlab已经支持CI/CD了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，gitlab已经支持CI/CD了，但是由于 Jenkins 插件比较丰富, 很多还是选择了使用 Jenkins，CI/CD 只是一个概念，具体的实现方式多种多样，用户根据自己的场景灵活选择就好。

##### **1195：
> SSH: Connecting from host [d2b74f26e796] SSH: Connecting with configuration [txy] ... SSH: EXEC: completed after 200 ms SSH: Disconnecting configuration [txy] ... ERROR: Exception when publishing, exception message [Exec exit status not zero. Status [1]] Build step 'Send files or execute commands over SSH' changed build result to UNSTABLE，老师，一直出现这个错误，可能是什么原因？另外/shell/release脚本这里应该是覆盖而不是追加吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可能是用户名或密码不正确，可以使用 Test Configuration 重新验证下服务器。

##### **6687：
> github的gitaction是不是也是类似的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 流程基本都一致，只不过使用的技术稍微有些差异

##### *冲：
> 容器一般生命周期比较短？持续集成像是一个长期的工作，总感觉有些不搭？但又不知道问题出在哪里

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这两者并没有本质关联吧，容器声明周期短指的是容器比较灵活，可以快速删除重建，并不是说一定要隔一段时间就杀掉。

##### **卫：
> 老师，上面docker login 登陆目标镜像仓库是gitlab吗？我写的gitlab用户密码构建失败

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 登录的是推送容器的镜像仓库哈，譬如 dockerhub 或者私有的镜像仓库等。

##### **伟：
> 第一次运行docker kill hello时，没有hello这个容器，会导致shell命令终止吧？这个问题怎么解决呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在执行 kiil 命令之前可以加一个判断，例如写成如下形式  [ "$(docker ps | grep hello)" ] && docker kill hello

##### **升：
> 打卡

