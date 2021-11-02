<p data-nodeid="224821" class="">严格意义上来说，Serverless 并不属于前端技术，但对于那些想提升自己知识广度，想往全栈工程师方向发展的前端工程师而言，是一个非常高效的工具。而对于那些只想专注于前端领域的工程师而言，了解 Serverless 背后的思想，对提升开发思维也会有一定的帮助。</p>
<p data-nodeid="224822">下面我们就通过相关概念和具体案例来揭开 Serverless 的神秘面纱吧。</p>
<h3 data-nodeid="224823">什么是 Serverless</h3>
<p data-nodeid="224824">Serverless 是由“Server”和“less”两个单词组合而成，翻译成中文就是“无服务器”的意思，所谓无服务器并非脱离服务器的 Web 离线应用，也不是说前端页面绕过服务端直接读写数据库，而是开发者不用再考虑服务器环境搭建和维护等问题，只需要专注于开发即可。也就是说 Serverless 不是语言或框架，而是一种软件的部署方式。传统的应用需要部署在服务器或虚拟机上，安装运行环境之后以进程的方式启动，而使用 Serverless 则可以省略这个过程，直接使用云服务厂商提供的运行环境。</p>
<h3 data-nodeid="224825">Serverless 从何而来</h3>
<p data-nodeid="224826">Serverless 并不是一个全新的产物，而是一种构建和管理基于微服务架构的完整流程，允许在服务部署级别而不是在服务器部署级别来管理你的应用部署。与传统架构的不同之处在于，它完全由云厂商管理，由事件触发，以无状态的方式运行（可能只存在于一次调用的过程中）在容器内。</p>
<h3 data-nodeid="224827">Serverless 的组成</h3>
<p data-nodeid="224828">Serverless 架构由两部分组成，即 FaaS 和 BaaS。</p>
<p data-nodeid="224829">FaaS（Function-as-a-Service）函数即服务，一个函数就是一个服务，函数可以由任何语言编写，除此之外不需要关心任何运维细节，比如计算资源、弹性扩容，而且还可以按量计费，且支持事件驱动。</p>
<p data-nodeid="224830">BaaS（Backend-as-a-Service）后端即服务，集成了许多中间件技术，比如数据即服务（数据库服务）、缓存、网关等。</p>
<h3 data-nodeid="224831">Serverless 的特点</h3>
<p data-nodeid="224832">了解完 Serverless 的基础概念，我们再来看看它有哪些特点。</p>
<h4 data-nodeid="224833">免维护</h4>
<p data-nodeid="224834">Serverless 不仅提供了运行代码的环境，还能自动实现负载均衡、弹性伸缩这类高级功能，极大地降低了服务搭建的复杂性，有效提升开发和迭代的速度。</p>
<h4 data-nodeid="224835">费用</h4>
<p data-nodeid="224836">如果只考虑 FaaS，那么费用是比较便宜的。</p>
<p data-nodeid="224837">以阿里云的函数计算为例，费用包括调用次数、执行时间、公网流量 3 个因素。调用次数和执行时间都有免费额度，即使超过，单次/每秒的费用也非常低，这种按需收费的方式就避免了资源的浪费。</p>
<p data-nodeid="226013">流量费用则稍贵，达到 0.8 元/GB，而且没有免费额度，所以对于通信数据量比较大的场景还是要慎重使用。</p>
<p data-nodeid="226287"><img src="https://s0.lgstatic.com/i/image/M00/4D/DA/Ciqc1F9bVNSABY8aAADMDBAJigk871.png" alt="Lark20200911-184216.png" data-nodeid="226290"></p>





<h4 data-nodeid="224840">深度绑定</h4>
<p data-nodeid="224841">通常使用某个云厂商的 Serverless 产品时，可能会包括多种产品，如函数计算、对象存储、数据库等，而这些产品和云厂商又深度绑定，所以如果要进行迁移，成本相对于部署在服务器而言会增加很多。</p>
<h4 data-nodeid="224842">运行时长限制</h4>
<p data-nodeid="224843">通常云厂商对于 Serverless 中的函数执行时间是有限制的，如阿里云的函数计算产品，最大执行时长为 10 分钟，如果执行长时间任务，还需要单独申请调整时长上限，或者自行将超时函数拆分成粒度更小的函数，但是这种方式会增加一定的开发成本。</p>
<h4 data-nodeid="224844">冷启动</h4>
<p data-nodeid="224845">由于函数是按需执行的，首次执行时会创建运行容器，一般这个创建环境的时间在几百毫秒，在延迟敏感的业务场景下需要进行优化，比如定时触发函数或者设置预留实例。</p>
<h3 data-nodeid="224846">Serverless 的应用场景</h3>
<p data-nodeid="224847">以阿里云的函数计算为例，可以分为两类：事件函数和 HTTP 函数。</p>
<ul data-nodeid="224848">
<li data-nodeid="224849">
<p data-nodeid="224850"><strong data-nodeid="224917">事件函数</strong>。事件函数的执行方式有两种，一种是通过 SDK 提供的 API 函数调用它，基于此可以进行一些轻量计算操作，比如对图片进行压缩、格式转换，又或者执行一些 AI 训练任务；另一种是通过配置时间和间隔，让其自动执行，一些常见的自动执行场景包括文件备份、数据统计等。</p>
</li>
<li data-nodeid="224851">
<p data-nodeid="224852"><strong data-nodeid="224922">HTTP 函数</strong>。每一个 HTTP 函数都有特定的域名来供外部访问，当这个域名被访问时，函数将会被创建并执行。可以使用 HTTP 函数为前后端分离架构的 Web 应用提供后端数据支撑，比如提供获取天气 API 查看实时天气，或者提供 API 来读写数据库。</p>
</li>
</ul>
<h3 data-nodeid="224853">Serverless 实例</h3>
<p data-nodeid="225211">下面来分析讲解一个使用阿里云函数计算来实现代码自动部署的例子。函数要实现的功能就是，当 GitHub 仓库中的某个分支有新的提交时，拉取最新代码并编译，然后将编译生成的代码部署到 OSS 存储的静态服务器上。</p>
<p data-nodeid="225746" class=""><img src="https://s0.lgstatic.com/i/image/M00/4D/E5/CgqCHl9bVLaATst6AADK7ETww2g973.png" alt="Lark20200911-184226.png" data-nodeid="225750"></p>
<div data-nodeid="225747"><p style="text-align:center">自动部署流程图</p></div>








<p data-nodeid="224857">在这个例子中，两种函数都会用到。</p>
<p data-nodeid="224858">首先是 HTTP 函数负责接收 GitHub 发出的 webhook 请求，当收到请求后使用内部模块调用一个事件函数，在这个事件函数中执行具体的操作。虽然理论上一个 HTTP 函数可以实现，但拆解成两个函数可以有效避免函数执行时间过长导致的 webhook 请求超时报错。</p>
<p data-nodeid="224859">下面是 HTTP 函数源码：</p>
<pre class="lang-javascript" data-nodeid="224860"><code data-language="javascript"><span class="hljs-comment">/**
&nbsp;* ACCOUNT_ID 主账号ID
&nbsp;* ACCESS_KEY_ID 访问 bucket 所需要的 key
&nbsp;* ACCESS_KEY_SECRET 访问 bucket 所需要的 secret
&nbsp;* REGION bucket 所在的 region
&nbsp;* BUCKET 用于存储配置文件的 bucket
&nbsp;*/</span>
<span class="hljs-keyword">const</span> {
&nbsp; ACCOUNT_ID,
&nbsp; ACCESS_KEY_ID,
&nbsp; ACCESS_KEY_SECRET,
&nbsp; REGION,
&nbsp; BUCKET
} = process.env
<span class="hljs-keyword">const</span> FCClient = <span class="hljs-built_in">require</span>(<span class="hljs-string">'@alicloud/fc2'</span>);
<span class="hljs-keyword">const</span> OSS = <span class="hljs-built_in">require</span>(<span class="hljs-string">'ali-oss'</span>)
<span class="hljs-keyword">const</span> getRawBody = <span class="hljs-built_in">require</span>(<span class="hljs-string">'raw-body'</span>)
<span class="hljs-comment">/**
&nbsp;*&nbsp;
&nbsp;* <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> </span>filePath 函数计算配置文件路径
&nbsp;*/</span>
<span class="hljs-keyword">const</span> getOSSConfigFile = <span class="hljs-keyword">async</span> (filePath) =&gt; {
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> OSS({
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">region</span>: REGION,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeyId</span>: ACCESS_KEY_ID,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeySecret</span>: ACCESS_KEY_SECRET,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">bucket</span>: BUCKET
&nbsp; &nbsp; });
&nbsp; &nbsp; <span class="hljs-keyword">const</span> result = <span class="hljs-keyword">await</span> client.get(filePath);
&nbsp; &nbsp; <span class="hljs-keyword">const</span> content = result.content ? result.content.toString() : <span class="hljs-string">'{}'</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-built_in">JSON</span>.parse(content)
&nbsp; } <span class="hljs-keyword">catch</span>(e) {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e)
&nbsp; &nbsp; <span class="hljs-keyword">return</span> {}
&nbsp; }
}

<span class="hljs-built_in">exports</span>.handler = <span class="hljs-function">(<span class="hljs-params">req, resp</span>) =&gt;</span> {
&nbsp; getRawBody(req, <span class="hljs-keyword">async</span> (e, payload) =&gt; {
&nbsp; &nbsp; <span class="hljs-keyword">const</span> body = <span class="hljs-built_in">JSON</span>.parse(payload)
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (e) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e)
&nbsp; &nbsp; &nbsp; resp.setStatusCode(<span class="hljs-number">400</span>)
&nbsp; &nbsp; &nbsp; resp.send(<span class="hljs-string">'请求体解析失败'</span>)
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">let</span> cfg
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">let</span> config
&nbsp; &nbsp; &nbsp; config = <span class="hljs-keyword">await</span> getOSSConfigFile(<span class="hljs-string">`/config/<span class="hljs-subst">${body.repository.name}</span>.json`</span>) || {}
&nbsp; &nbsp; &nbsp; cfg = config.action[body.action]
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!cfg) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(config.action, body.action)
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-built_in">Error</span>(<span class="hljs-string">'未找到对应仓库的配置信息.'</span>)
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (e) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e)
&nbsp; &nbsp; &nbsp; resp.setStatusCode(<span class="hljs-number">500</span>)
&nbsp; &nbsp; &nbsp; resp.send(e.message)
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cfg) {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> FCClient(ACCOUNT_ID, {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeyID</span>: ACCESS_KEY_ID,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeySecret</span>: ACCESS_KEY_SECRET,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">region</span>: cfg.region
&nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; client.invokeFunction(cfg.service, cfg.name, <span class="hljs-built_in">JSON</span>.stringify(cfg)).catch(<span class="hljs-built_in">console</span>.error)
&nbsp; &nbsp; &nbsp; resp.send(<span class="hljs-string">`client.invokeFunction(<span class="hljs-subst">${cfg.service}</span>, <span class="hljs-subst">${cfg.name}</span>, <span class="hljs-subst">${<span class="hljs-built_in">JSON</span>.stringify(cfg)}</span>)`</span>)
&nbsp; &nbsp; }
&nbsp; })
}
</code></pre>
<p data-nodeid="224861">简单的实现方式就是解析 webhook 请求体内部的参数，获取仓库名和分支名传递给事件函数，但考虑可扩展性，对每个项目仓库使用了单独的配置文件。具体到代码中就是调用 getOSSConfigFile() 函数来从 OSS 存储上读取仓库相关的配置文件信息，然后通过 invokeFunction() 函数调用事件函数并将配置信息传递给事件函数。</p>
<p data-nodeid="224862">这样的好处在于，之后要新增其他仓库或其他分支的时候，只需要新增一个配置文件就可以了。</p>
<p data-nodeid="224863">再来看看事件函数的入口函数实现。</p>
<p data-nodeid="224864">前面在讲 Serverless 函数冷启动问题的时候提到过，函数执行完成后会存活一段时间，在这段时间内再次调用会执行之前创建的函数，短时间内重复执行的话会因为已经存在目录导致拉取失败，所以创建了随机目录并修改工作目录到随机目录下以获取写权限。</p>
<p data-nodeid="224865">然后再根据配置文件中的信息，按串行加载对应的执行模块并传入参数。</p>
<pre class="lang-javascript" data-nodeid="224866"><code data-language="javascript"><span class="hljs-keyword">const</span>&nbsp;fs&nbsp;=&nbsp;<span class="hljs-built_in">require</span>(<span class="hljs-string">'fs'</span>)
<span class="hljs-comment">/**
&nbsp;*&nbsp;
&nbsp;*&nbsp;<span class="hljs-doctag">@param&nbsp;<span class="hljs-type">{*}</span>&nbsp;</span>event&nbsp;
&nbsp;*&nbsp;&nbsp;&nbsp;{
&nbsp;*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;repo&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仓库地址
&nbsp;*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;region&nbsp;&nbsp;&nbsp;bucket&nbsp;所在区域
&nbsp;*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bucket&nbsp;&nbsp;&nbsp;编译后部署的bucket
&nbsp;*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;command&nbsp;&nbsp;编译命令
&nbsp;*&nbsp;&nbsp;&nbsp;}
&nbsp;*&nbsp;<span class="hljs-doctag">@param&nbsp;<span class="hljs-type">{*}</span>&nbsp;</span>context&nbsp;
&nbsp;*&nbsp;<span class="hljs-doctag">@param&nbsp;<span class="hljs-type">{*}</span>&nbsp;</span>callback&nbsp;
&nbsp;*/</span>
<span class="hljs-built_in">exports</span>.handler&nbsp;=&nbsp;<span class="hljs-keyword">async</span>&nbsp;(event,&nbsp;context,&nbsp;callback)&nbsp;=&gt;&nbsp;{
&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;{events}&nbsp;=&nbsp;Buffer.isBuffer(event)&nbsp;?&nbsp;<span class="hljs-built_in">JSON</span>.parse(event.toString())&nbsp;:&nbsp;event
&nbsp;&nbsp;<span class="hljs-keyword">let</span>&nbsp;dir&nbsp;=&nbsp;<span class="hljs-built_in">Math</span>.random().toString(<span class="hljs-number">36</span>).substr(<span class="hljs-number">6</span>)
&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置随机临时工作目录，避免容器未销毁的情况下，重复拉取仓库失败</span>
&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;workDir&nbsp;=&nbsp;<span class="hljs-string">`/tmp/<span class="hljs-subst">${dir}</span>`</span>
&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;为了保证后续流程能找到临时工作目录，设置为全局变量</span>
&nbsp;&nbsp;<span class="hljs-built_in">global</span>.workDir&nbsp;=&nbsp;workDir&nbsp;
&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;fs.mkdirSync(workDir)
&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>(e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">console</span>.error(e)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>
&nbsp;&nbsp;}
&nbsp;&nbsp;process.chdir(workDir);
&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;events.reduce(<span class="hljs-keyword">async</span>&nbsp;(acc,&nbsp;cur)&nbsp;=&gt;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">await</span>&nbsp;acc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-built_in">require</span>(<span class="hljs-string">`./<span class="hljs-subst">${cur.<span class="hljs-built_in">module</span>}</span>`</span>)(cur)
&nbsp;&nbsp;&nbsp;&nbsp;},&nbsp;<span class="hljs-built_in">Promise</span>.resolve())
&nbsp;&nbsp;&nbsp;&nbsp;callback(<span class="hljs-literal">null</span>,&nbsp;<span class="hljs-string">`自动部署成功.`</span>);
&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>(e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;callback(e)
&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="224867">具体有哪些模块呢？一般的部署过程主要包括 3 步：拉取仓库代码、安装依赖并构建、将生成的代码上传部署。</p>
<p data-nodeid="224868">拉取代码对于手动操作而言很简单，一条 git clone 命令就搞定了，但在自动化实现的时候会碰到一些麻烦的细节问题。</p>
<p data-nodeid="224869">首先身份认证问题。由于是私有仓库，所以只能通过密钥文件或账号密码的形式来认证访问权限。如果通过账号密码的形式登录，则要模拟键终端交互，这个相对而言实现成本较高，所以采用了配置 ssh key 的方式。具体是根据前面传入的配置信息找到私钥文件所在地址并下载到本地，但是因为权限问题，并不能直接保存到当前用户的 .ssh 目录下。</p>
<p data-nodeid="224870">另一个问题是在首次进行 git clone 操作的时候，终端会出现一个是否添加 known hosts 的提示，在终端中操作的时候需要键盘输入 “Y” 或者 “N” 来继续后面的操作。这里可以通过一个选项本来关闭这个提示。</p>
<p data-nodeid="224871">具体实现参考下面的 shell 脚本。</p>
<pre class="lang-shell" data-nodeid="224872"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash">!/bin/sh</span>
ID_RSA=/tmp/id_rsa
exec /usr/bin/ssh -o StrictHostKeyChecking=no -o GSSAPIAuthentication=no -i $ID_RSA "$@"
</code></pre>
<p data-nodeid="224873">解决了比较麻烦的身份认证和终端交互问题之后，剩下的逻辑就比较简单了，执行 git clone 命令拉取仓库代码就行，为了加快速下载速度，可以通过设置 --depth 1 这个参数来指定只拉取最新提交的代码。</p>
<pre class="lang-javascript" data-nodeid="224874"><code data-language="javascript"><span class="hljs-keyword">const</span> OSS = <span class="hljs-built_in">require</span>(<span class="hljs-string">'ali-oss'</span>)
<span class="hljs-keyword">const</span> cp = <span class="hljs-built_in">require</span>(<span class="hljs-string">'child_process'</span>)
<span class="hljs-keyword">const</span> { BUCKET, REGION, ACCESS_KEY_ID, ACCESS_KEY_SECRET } = process.env
<span class="hljs-keyword">const</span> shellFile = <span class="hljs-string">'ssh.sh'</span>
<span class="hljs-comment">/**
&nbsp;*&nbsp;
&nbsp;* <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> </span>repoURL 代码仓库地址
&nbsp;* <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> </span>repoKey 访问代码仓库所需要的密钥文件路径
&nbsp;* <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> </span>branch&nbsp; 分支名称
&nbsp;*/</span>
<span class="hljs-keyword">const</span> downloadRepo = <span class="hljs-keyword">async</span> ({repoURL, repoKey, branch=<span class="hljs-string">'master'</span>}, retryTimes = <span class="hljs-number">0</span>) =&gt; {
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Download repo <span class="hljs-subst">${repoURL}</span>`</span>);
&nbsp; &nbsp; process.chdir(<span class="hljs-built_in">global</span>.workDir)
&nbsp; &nbsp; <span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> OSS({
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeyId</span>: ACCESS_KEY_ID,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">accessKeySecret</span>: ACCESS_KEY_SECRET,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">region</span>: REGION,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">bucket</span>: BUCKET
&nbsp; &nbsp; });
&nbsp; &nbsp; <span class="hljs-keyword">await</span> client.get(repoKey, <span class="hljs-string">`./id_rsa`</span>);
&nbsp; &nbsp; <span class="hljs-keyword">await</span> client.get(shellFile, <span class="hljs-string">`./<span class="hljs-subst">${shellFile}</span>`</span>);
&nbsp; &nbsp; cp.execSync(<span class="hljs-string">`chmod 0600 ./id_rsa`</span>);
&nbsp; &nbsp; cp.execSync(<span class="hljs-string">`chmod +x ./<span class="hljs-subst">${shellFile}</span>`</span>);
&nbsp; &nbsp; cp.execSync(<span class="hljs-string">`GIT_SSH="./<span class="hljs-subst">${shellFile}</span>" git clone -b <span class="hljs-subst">${branch}</span> --depth 1 <span class="hljs-subst">${repoURL}</span>`</span>);
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'downloaded'</span>);
&nbsp; } <span class="hljs-keyword">catch</span> (e) {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (retryTimes &lt; <span class="hljs-number">2</span>) {
&nbsp; &nbsp; &nbsp; downloadRepo({repoURL, repoKey, branch}, retryTimes++);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> e
&nbsp; &nbsp; }
&nbsp; }
};
<span class="hljs-built_in">module</span>.exports = downloadRepo
</code></pre>
<p data-nodeid="224875">安装依赖并构建这个步骤没有太多复杂的地方，通过子进程调用 yarn install --check-files 命令，然后执行 package.json 文件中配置的脚本任务即可。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="224876"><code data-language="javascript"><span class="hljs-keyword">const</span> cp = <span class="hljs-built_in">require</span>(<span class="hljs-string">'child_process'</span>)

<span class="hljs-keyword">const</span> install = <span class="hljs-function">(<span class="hljs-params">repoName, retryTimes = <span class="hljs-number">0</span></span>) =&gt;</span> {
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Install dependencies.'</span>);
&nbsp; &nbsp; cp.execSync(<span class="hljs-string">`yarn install --check-files`</span>);
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Installed.'</span>);
&nbsp; &nbsp; retryTimes = <span class="hljs-number">0</span>
&nbsp; } <span class="hljs-keyword">catch</span> (e) {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e.message);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (retryTimes &lt; <span class="hljs-number">2</span>) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Retry install...'</span>);
&nbsp; &nbsp; &nbsp; install(repoName, ++retryTimes);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> e
&nbsp; &nbsp; }
&nbsp; }
}
<span class="hljs-keyword">const</span> build = <span class="hljs-function">(<span class="hljs-params">command, retryTimes = <span class="hljs-number">0</span></span>) =&gt;</span> {
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Build code.'</span>)
&nbsp; &nbsp; cp.execSync(<span class="hljs-string">`<span class="hljs-subst">${command}</span>`</span>);
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Built.'</span>);
&nbsp; } <span class="hljs-keyword">catch</span> (e) {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e.message);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (retryTimes &lt; <span class="hljs-number">2</span>) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Retry build...'</span>);
&nbsp; &nbsp; &nbsp; build(command, ++retryTimes);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> e
&nbsp; &nbsp; }
&nbsp; }
};

<span class="hljs-built_in">module</span>.exports = <span class="hljs-function">(<span class="hljs-params">{
&nbsp; repoName,
&nbsp; command
}</span>) =&gt;</span> {
&nbsp; <span class="hljs-keyword">const</span> {
&nbsp; &nbsp; workDir
&nbsp; } = <span class="hljs-built_in">global</span>
&nbsp; process.chdir(<span class="hljs-string">`<span class="hljs-subst">${workDir}</span>/<span class="hljs-subst">${repoName}</span>`</span>)
&nbsp; install(repoName)
&nbsp; build(command)
}
</code></pre>
<p data-nodeid="224877">最后上传部署可以根据不同的场景编写不同的模块，比如有的可能部署在 OSS 存储上，会需要调用 OSS 对应的 SDK 进行上传，有的可能部署在某台服务器上，需要通过 scp 命令来传输。</p>
<p data-nodeid="224878">下面是一个部署到 OSS 存储的例子。</p>
<pre class="lang-javascript" data-nodeid="224879"><code data-language="javascript"><span class="hljs-keyword">const</span> path = <span class="hljs-built_in">require</span>(<span class="hljs-string">'path'</span>);
<span class="hljs-keyword">const</span> OSS = <span class="hljs-built_in">require</span>(<span class="hljs-string">'ali-oss'</span>);
<span class="hljs-comment">// 遍历函数</span>
<span class="hljs-keyword">const</span> traverse = <span class="hljs-function">(<span class="hljs-params">dirPath, arr = []</span>) =&gt;</span> {
&nbsp; <span class="hljs-keyword">var</span> filesList = fs.readdirSync(dirPath);
&nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">var</span> i = <span class="hljs-number">0</span>; i &lt; filesList.length; i++) {
&nbsp; &nbsp; <span class="hljs-keyword">var</span> fileObj = {};
&nbsp; &nbsp; fileObj.name = path.join(dirPath, filesList[i]);
&nbsp; &nbsp; <span class="hljs-keyword">var</span> filePath = path.join(dirPath, filesList[i]);
&nbsp; &nbsp; <span class="hljs-keyword">var</span> stats = fs.statSync(filePath);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (stats.isDirectory()) {
&nbsp; &nbsp; &nbsp; traverse(filePath, arr);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; fileObj.type = path.extname(filesList[i]).substring(<span class="hljs-number">1</span>);
&nbsp; &nbsp; &nbsp; arr.push(fileObj);
&nbsp; &nbsp; }
&nbsp; }
&nbsp; <span class="hljs-keyword">return</span> arr
}
<span class="hljs-comment">/**
&nbsp;*&nbsp;
&nbsp;* <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> <span class="hljs-variable">repoName</span></span>
&nbsp;*&nbsp;
&nbsp;*/</span>
<span class="hljs-keyword">const</span> deploy = <span class="hljs-function">(<span class="hljs-params">{ dist = <span class="hljs-string">''</span>, source, region, accessKeyId, accessKeySecret, bucket, repoName }, retryTimes = <span class="hljs-number">0</span></span>) =&gt;</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-keyword">async</span> (res) =&gt; {
&nbsp; <span class="hljs-keyword">const</span> { workDir } = <span class="hljs-built_in">global</span>
&nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Deploy.'</span>);
&nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; <span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> OSS({
&nbsp; &nbsp; &nbsp; region,
&nbsp; &nbsp; &nbsp; accessKeyId,
&nbsp; &nbsp; &nbsp; accessKeySecret,
&nbsp; &nbsp; &nbsp; bucket
&nbsp; &nbsp; });
&nbsp; &nbsp; process.chdir(<span class="hljs-string">`<span class="hljs-subst">${workDir}</span>/<span class="hljs-subst">${repoName}</span>`</span>)
&nbsp; &nbsp; <span class="hljs-keyword">const</span> root = path.join(process.cwd(), source)
&nbsp; &nbsp; <span class="hljs-keyword">let</span> files = traverse(root, []);
&nbsp; &nbsp; <span class="hljs-keyword">await</span> <span class="hljs-built_in">Promise</span>.all(files.map(<span class="hljs-function">(<span class="hljs-params">{ name }, index</span>) =&gt;</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">const</span> remotePath = path.join(dist, name.replace(root + <span class="hljs-string">'/'</span>, <span class="hljs-string">''</span>));
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`[<span class="hljs-subst">${index}</span>] uploaded <span class="hljs-subst">${name}</span> to <span class="hljs-subst">${remotePath}</span>`</span>);
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> client.put(remotePath, name);
&nbsp; &nbsp; }));
&nbsp; &nbsp; res();
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Deployed.'</span>);
&nbsp; } <span class="hljs-keyword">catch</span> (e) {
&nbsp; &nbsp; <span class="hljs-built_in">console</span>.error(e);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (retryTimes &lt; <span class="hljs-number">2</span>) {
&nbsp; &nbsp; &nbsp; <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'Retry deploy.'</span>);
&nbsp; &nbsp; &nbsp; deploy({ dist, source, region, accessKeyId, accessKeySecret, bucket }, ++retryTimes);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> e
&nbsp; &nbsp; }
&nbsp; }
})
<span class="hljs-built_in">module</span>.exports = deploy
</code></pre>
<p data-nodeid="224880">由于未找到阿里云 OSS SDK 中上传目录的功能，所以只能通过深度遍历的方式来逐个将文件进行上传。考虑编译后生成地文件数量并不多，这里没有做并发数限制，而是将全部文件进行批量上传。</p>
<h3 data-nodeid="224881">总结</h3>
<p data-nodeid="224882">虽然 Serverless 并不属于前端开发范畴，但确实是一个具有通用性、开箱即用的产品。本课时的主要目的是起到一个抛砖引玉的作用，通过概念介绍以及函数计算的具体实例让你对其有一个初步的认识和了解。希望你在工作中能通过具体实践，不断探索它的使用边界和场景。</p>
<p data-nodeid="224883" class="">最后布置一道作业题：尝试部署一个 Serverless 服务。</p>

---

### 精选评论


