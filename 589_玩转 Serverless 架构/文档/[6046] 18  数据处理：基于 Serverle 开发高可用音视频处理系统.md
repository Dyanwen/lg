<p data-nodeid="1177" class="">在推广 Serverless 的过程中，经常有同学问我：除了用来开发后端接口、服务端渲染应用等场景，Serverless 还能用来做什么呢？</p>
<p data-nodeid="1178">其实，Serverless 的应用场景非常广泛，除了上述几种，它还可以用于大数据计算、物联网应用、音视频处理等。为了让你了解到更多的 Serverless 的应用场景，我准备了今天的内容。</p>
<p data-nodeid="1179">音视频处理是一个 CPU 密集型的操作，非常消耗计算资源，以往我们处理视频就要采购大量的高性能服务器，财务成本和维护成本都很高。有了 Serverless 后，就不用再关心计算资源不足的问题，也不用担心服务器的维护，并且还能降低成本。</p>
<p data-nodeid="1180">接下来，我先带你了解传统的音视频处理方案，然后在此基础上再带你学习并实践基于 Serverless 的音视频处理系统，这样你理解得会更加深入。</p>
<h3 data-nodeid="1181">传统音视频处理方案</h3>
<p data-nodeid="1182">近几年，计算机技术和通信技术日新月异，信息传播的媒介也在不断演变，从文字到图片再到视频，各种短视频、直播甚至 AR、VR 等产品百花齐放。在这些产品的背后，离不开音视频处理技术。</p>
<p data-nodeid="1183">得益于云计算的发展，有些云厂商推出了对应的视频解决方案，因此你现在要搭建一个视频处理程序是很容易的（下图就是一个典型的视频处理方案）：</p>
<p data-nodeid="1184"><img src="https://s0.lgstatic.com/i/image6/M01/07/36/Cgp9HWAzRsKAW-EAAATCejrS5YI741.png" alt="Drawing 0.png" data-nodeid="1294"></p>
<div data-nodeid="1185"><p style="text-align:center">传统视频处理解决方案</p></div>
<p data-nodeid="1186">在该方案中，我们用 OSS 来存储海量的视频内容，视频上传后用视频转码服务将不同来源的视频进行转码，以适配各种终端，然后利用 CDN 提升客户端访问视频的速度。</p>
<p data-nodeid="1187">不过，虽然用了视频转码服务，但我们还是要购买大量的服务器，搭建自己的视频处理系统，对视频进行更高级的自定义处理，比如视频转码后将元数据存入数据库、生成视频前几秒的 GIF 图片用来做视频的封面，以及各种格式的音视频转换等。</p>
<p data-nodeid="1188">除此之外，当我们已经在服务器上部署了一套视频处理系统后，可能还会遇到一些问题。比如，如何应对大量并发任务？能否让这个系统有更高的弹性和可用性？这些问题其实超出了视频处理本身的范围，我们的需求只是进行视频处理，但不得不面临繁重的运维工作。并且我们可能为了应对周期大量处理任务或瞬时流量，不得不购买大量的服务器，成本大幅增加，在服务器的闲置期间还造成了不必要的资源浪费。而且我们也无法 100% 利用机器的性能，这也是一种资源浪费。</p>
<p data-nodeid="1189">而 Serverless 就能解决这些问题，基于 Serverless 你可以很轻松实现一个弹性、可扩展、低成本、免运维、高可用的音视频处理系统。</p>
<h3 data-nodeid="1190">基于 Serverless 的音视频处理系统</h3>
<p data-nodeid="1191">从基础设施的角度来看，基于 Serverless 的音视频解决方案，<strong data-nodeid="1304">主要是替换了传统方案中的计算资源，也就是替换了服务器。</strong></p>
<p data-nodeid="1192">此外，我们基于 Serverless 平台提供的丰富的触发器，也能简化编程模型。比如以往我们需要用户将视频上传到 OSS 后，再通过接口主动通知服务器进行视频处理，但在 Serverless 架构中，我们可以为函数设置 OSS 触发器，这样只要有文件被上传到 OSS 中，就可以触发函数执行，进而简化了业务逻辑。</p>
<p data-nodeid="1193">下图就是基于 Serverless 的视频处理系统解决方案：</p>
<p data-nodeid="1194"><img src="https://s0.lgstatic.com/i/image6/M01/07/36/Cgp9HWAzRsuAb0gaAAGQSnOzXmk655.png" alt="Drawing 1.png" data-nodeid="1309"></p>
<div data-nodeid="1195"><p style="text-align:center">基于 Serverless 的视频处理系统</p></div>
<p data-nodeid="1196">用户将视频上传后 OSS 后，触发函数计算中的视频转码函数执行，该函数对视频进行转码后，将元数据存入数据库，然后将转码后的视频再保存到 OSS 中。</p>
<p data-nodeid="1197">接下来我们就实现一个基于 Serverless 的音视频处理系统，系统主要有以下几个功能：</p>
<ul data-nodeid="1198">
<li data-nodeid="1199">
<p data-nodeid="1200">获取视频时长；</p>
</li>
<li data-nodeid="1201">
<p data-nodeid="1202">获取视频元数据；</p>
</li>
<li data-nodeid="1203">
<p data-nodeid="1204">截取视频 GIF 图；</p>
</li>
<li data-nodeid="1205">
<p data-nodeid="1206">为视频添加水印；</p>
</li>
<li data-nodeid="1207">
<p data-nodeid="1208">对视频进行转码。</p>
</li>
</ul>
<p data-nodeid="1209">为了方便你实践，我为你提供了一份<a href="https://github.com/nodejh/serverless-class/tree/master/18/serverless-video" data-nodeid="1320">示例代码</a>，你可以通过 git 下载查看：</p>
<pre class="lang-java" data-nodeid="1210"><code data-language="java">$ git clone https:<span class="hljs-comment">//github.com/nodejh/serverless-class</span>
$ cd <span class="hljs-number">18</span>/serverless-video
</code></pre>
<p data-nodeid="1211">代码结构如下：</p>
<pre class="lang-java" data-nodeid="1212"><code data-language="java">.
├── functions
│&nbsp;&nbsp; ├── common
│&nbsp;&nbsp; │&nbsp;&nbsp; └── utils.js
│&nbsp;&nbsp; ├── get_duration
│&nbsp;&nbsp; │&nbsp;&nbsp; └── index.js
│&nbsp;&nbsp; └── get_meta
│&nbsp;&nbsp;     └── index.js
├── build.js
├── ffmpeg
├── ffprobe
├── <span class="hljs-keyword">package</span>.json
└── template.yml
</code></pre>
<p data-nodeid="1213">其中 functions 中是函数源代码，<code data-backticks="1" data-nodeid="1324">common/utils.js</code>是一些公共方法，<code data-backticks="1" data-nodeid="1326">get_duration</code>、<code data-backticks="1" data-nodeid="1328">get_meta</code>等目录则分别对应的每个具体的功能。<code data-backticks="1" data-nodeid="1330">build.js</code>是用来构建函数的脚本。在代码中，我们会使用 <a href="https://ffmpeg.org/" data-nodeid="1334">FFmpeg</a> 进行视频处理，FFmpeg 是一款功能强大、用途广泛的开源软件，很多视频网站都在用它，比如 Youtube、Bilibili。ffmpeg 和 ffprobe 是 FFmpeg 的两个命令行工具，我们会将其作为依赖部署到 FaaS 平台（函数计算）上，这样在函数中就可以使用这两个命令来处理视频了。<br>
接下来就让我们学习具体如何实现。</p>
<p data-nodeid="1214">由于这几个函数的逻辑基本类似，所以我主要针对“获取视频时长”函数进行讲解，学会了这个函数的实现就很容易理解其他函数了。另外，由于该视频处理系统用到了公共方法及依赖，所以我还会为你介绍如何部署这些函数。</p>
<h4 data-nodeid="1215">获取视频时长函数的实现</h4>
<p data-nodeid="1216">首先是获取视频时长的实现，也就是 get_duration 函数。我们可以通过 ffprobe 来获取视频时长，命令如下：</p>
<pre class="lang-c#" data-nodeid="1217"><code data-language="c#">$ ffprobe -v quiet -show_entries format=duration -print_format json -i video.mp4
{
    <span class="hljs-string">"format"</span>: {
        <span class="hljs-string">"duration"</span>: <span class="hljs-string">"170.859000"</span>
    }
}
</code></pre>
<p data-nodeid="1218">其中<code data-backticks="1" data-nodeid="1344">-print_format json</code>是指以 JSON 格式输出结果，<code data-backticks="1" data-nodeid="1346">-i</code>是指定文件位置，可以是本地文件，也可以是网络上的远程文件。</p>
<p data-nodeid="1219"><strong data-nodeid="1352">所以获取视频时长的函数逻辑就是：</strong> 下载 OSS 中的文件到本地，然后运行 ffprobe 命令得到视频时长，最后返回视频时长。</p>
<p data-nodeid="1220">为了让代码尽可能复用，所以我在<code data-backticks="1" data-nodeid="1354">common/utils.js</code>中实现了一些公共方法，代码大致如下：</p>
<pre class="lang-javascript" data-nodeid="1221"><code data-language="javascript"><span class="hljs-comment">// common/utils.js</span>
<span class="hljs-comment">// ...</span>
<span class="hljs-comment">/**
 * 运行 Linux 命令
 * <span class="hljs-doctag">@param <span class="hljs-type">{string}</span> </span>command 待运行的命令
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">exec</span>(<span class="hljs-params">command</span>) </span>{
  <span class="hljs-built_in">console</span>.log(command)
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> <span class="hljs-built_in">Promise</span>(<span class="hljs-function">(<span class="hljs-params">resolve, reject</span>) =&gt;</span> {
    child_process.exec(command, (err, stdout, stderr) =&gt; {
      <span class="hljs-keyword">if</span> (err) {
        <span class="hljs-built_in">console</span>.error(err)
        <span class="hljs-keyword">return</span> reject(err);
      }
      <span class="hljs-keyword">if</span> (stderr) {
        <span class="hljs-built_in">console</span>.error(stderr)
        <span class="hljs-keyword">return</span> reject(stderr);
      }
      <span class="hljs-built_in">console</span>.log(stdout)
      <span class="hljs-keyword">return</span> resolve(stdout);
    });
  });
}
<span class="hljs-comment">/**
 * 获取 OSS Client
 * <span class="hljs-doctag">@param <span class="hljs-type">{object}</span> </span>context 函数上下文
 */</span>
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getOssClient</span>(<span class="hljs-params">context</span>) </span>{
  <span class="hljs-comment">// 获取函数计算的临时访问凭证</span>
  <span class="hljs-keyword">const</span> accessKeyId = context.credentials.accessKeyId;
  <span class="hljs-keyword">const</span> accessKeySecret = context.credentials.accessKeySecret;
  <span class="hljs-keyword">const</span> securityToken = context.credentials.securityToken;
  <span class="hljs-comment">// 初始化 OSS 客户端</span>
  <span class="hljs-keyword">const</span> client = oss({
    accessKeyId,
    accessKeySecret,
    <span class="hljs-attr">stsToken</span>: securityToken,
    <span class="hljs-attr">bucket</span>: OSS_BUCKET_NAME,
    <span class="hljs-attr">region</span>: OSS_REGION,
  });
  <span class="hljs-keyword">return</span> client;
}
<span class="hljs-built_in">module</span>.exports = {
  exec,
  getOssClient,
  OSS_VIDEO_NAME,
};
</code></pre>
<p data-nodeid="1222"><code data-backticks="1" data-nodeid="1356">common/utils.js</code>的代码主要就包含两个方法：<code data-backticks="1" data-nodeid="1358">exec</code>和<code data-backticks="1" data-nodeid="1360">getOssClient</code>，分别用来执行 Linux 系统命令和获取 OSS 客户端。</p>
<p data-nodeid="1223">这样我们在<code data-backticks="1" data-nodeid="1363">functions/get_duration/index.js</code>中就可以直接引入并使用了：</p>
<pre class="lang-javascript" data-nodeid="1224"><code data-language="javascript"><span class="hljs-comment">// functions/get_duration/index.js</span>
<span class="hljs-keyword">const</span> { exec, getOssClient, OSS_VIDEO_NAME } = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../common/utils"</span>);
<span class="hljs-comment">/**
 * 获取视频元信息
 * @param {object} client OSS client
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getDuration</span>(<span class="hljs-params">client</span>) </span>{
  <span class="hljs-keyword">const</span> filePath = <span class="hljs-string">"/tmp/video.mp4"</span>;
  <span class="hljs-keyword">await</span> client.get(OSS_VIDEO_NAME, filePath);
  <span class="hljs-keyword">const</span> command = <span class="hljs-string">`./ffprobe -v quiet -show_entries format=duration -print_format json -i <span class="hljs-subst">${filePath}</span>`</span>;
  <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> exec(command);
  <span class="hljs-keyword">return</span> res;
}

<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 获取 OSS 客户端</span>
  <span class="hljs-keyword">const</span> client = getOssClient(context);
  getDuration(client)
    .then(<span class="hljs-function">(<span class="hljs-params">res</span>) =&gt;</span> {
      <span class="hljs-built_in">console</span>.log(<span class="hljs-string">"视频时长: \n"</span>, res);
      callback(<span class="hljs-literal">null</span>, res);
    })
    .catch(<span class="hljs-function">(<span class="hljs-params">err</span>) =&gt;</span> callback(err));
};
</code></pre>
<p data-nodeid="1225">首先注意第 20 行，我们通过 getOssClient 获取到 OSS 客户端，然后调用 getDuration 函数执行业务逻辑，也就是获取视频时长。</p>
<p data-nodeid="1226">在 getDuration 中，我们先下载视频到临时目录<code data-backticks="1" data-nodeid="1367">/tmp/video.mp4</code>中，临时目录是可以读写的，当前代码目录只能写不能读。然后在第 13 行，通过 exec 执行了获取视频时长的命令，最后将得到的结果返回。</p>
<p data-nodeid="1227">这样获取视频时长的功能就开发完成了。</p>
<p data-nodeid="1228">获取视频元数据等其他函数与获取视频时长的实现是非常类似的，不同之处主要在于执行的命令，也就是第 12 行的<code data-backticks="1" data-nodeid="1371">command</code>变量。具体实现可以参考我的示例代码，这里就不赘述。</p>
<p data-nodeid="1229">由于该系统包含多个函数，且函数不仅依赖了 ffmpeg ，还依赖了公共的<code data-backticks="1" data-nodeid="1374">common/utils.js</code>，所以很多同学就犯难了，这些函数应该怎么部署呢？</p>
<h4 data-nodeid="1230">音视频处理系统的部署</h4>
<p data-nodeid="1231">让我们先回顾一下 “06 | 依赖管理：Serverless 应用怎么安装依赖？”的内容，这一讲我们学习了函数的依赖需要一起打包上传到 FaaS 平台，所以我们需要将 ffmpeg 或 ffprobe 上传。看起来比较简单，我们直接将其放在函数代码目录并上传就可以了。</p>
<p data-nodeid="1232"><strong data-nodeid="1384">不过这里需要注意的是，</strong> 由于 ffmpeg 和 ffprobe 是可执行文件，最终我们需要用到这两个命令，所以在上传到 FaaS 平台之前，需要为其赋予可执行权限。</p>
<p data-nodeid="1233">你可以通过<code data-backticks="1" data-nodeid="1386">ls -l</code>来查看文件的权限：</p>
<pre class="lang-java" data-nodeid="1234"><code data-language="java">$ ls -l
-rwxr-xr-x    <span class="hljs-number">1</span> root  staff  <span class="hljs-number">39000328</span>  <span class="hljs-number">2</span>  <span class="hljs-number">9</span> <span class="hljs-number">20</span>:<span class="hljs-number">59</span> ffmpeg
-rwxr-xr-x    <span class="hljs-number">1</span> root  staff  <span class="hljs-number">38906056</span>  <span class="hljs-number">2</span>  <span class="hljs-number">9</span> <span class="hljs-number">21</span>:<span class="hljs-number">00</span> ffprobe
</code></pre>
<p data-nodeid="1235"><code data-backticks="1" data-nodeid="1388">-rwxr-xr-x</code>分为四部分：</p>
<ul data-nodeid="1236">
<li data-nodeid="1237">
<p data-nodeid="1238">第 0 位<code data-backticks="1" data-nodeid="1391">-</code>表示文件类型；</p>
</li>
<li data-nodeid="1239">
<p data-nodeid="1240">第 1-3 位<code data-backticks="1" data-nodeid="1394">rwx</code>表示文件所有者的权限；</p>
</li>
<li data-nodeid="1241">
<p data-nodeid="1242">第 4-6 位<code data-backticks="1" data-nodeid="1397">r-x</code>是同组用户的权限；</p>
</li>
<li data-nodeid="1243">
<p data-nodeid="1244">第 7-9<code data-backticks="1" data-nodeid="1400">r-x</code>位表示其他用户的权限。</p>
</li>
</ul>
<p data-nodeid="1245">r 表示读权限，w 表示写权限，x 表示执行权限。从文件权限可以看出，针对所有用户这两个文件都有可执行权限。</p>
<p data-nodeid="1246">如果你的这两个文件没有执行权限，则需要通过下面的命令添加权限：</p>
<pre class="lang-java" data-nodeid="1247"><code data-language="java">$ chmod +x ffmpeg
$ chmod +x ffprobe
</code></pre>
<p data-nodeid="1248">这样在 FaaS 平台上，Node.js 才可以执行这两个命令。<br>
解决了可执行文件的权限问题后，还有一个问题是函数的权限。</p>
<p data-nodeid="3813" class="te-preview-highlight">由于函数需要读写 OSS，所以我们需要为函数设置角色，并为该角色添加管理 OSS 的权限。如果你不清楚如何授权，可以复习一下 “10｜访问控制：如何授权访问其他云服务？”的内容。</p>





<p data-nodeid="1250">在我提供的示例代码中，我在 template.yaml 的第 7 行设置了函数的角色<code data-backticks="1" data-nodeid="1409">acs:ram::1457216987974698:role/aliyunfclogexecutionrole</code>，文件内容如下所示：</p>
<pre class="lang-yaml" data-nodeid="1251"><code data-language="yaml"><span class="hljs-attr">ROSTemplateFormatVersion:</span> <span class="hljs-string">'2015-09-01'</span>
<span class="hljs-attr">Transform:</span> <span class="hljs-string">'Aliyun::Serverless-2018-04-03'</span>
<span class="hljs-attr">Resources:</span>
  <span class="hljs-attr">serverless-video:</span>
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Service'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">Role:</span> <span class="hljs-string">acs:ram::1457216987974698:role/aliyunfclogexecutionrole</span>
      <span class="hljs-attr">Description:</span> <span class="hljs-string">'基于 Serverless 开发高可用音视频处理系统'</span>
    <span class="hljs-attr">get_duration:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">index.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">Timeout:</span> <span class="hljs-number">600</span>
        <span class="hljs-attr">MemorySize:</span> <span class="hljs-number">256</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">./.serverless/get_duration</span>
    <span class="hljs-attr">get_meta:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">index.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">Timeout:</span> <span class="hljs-number">600</span>
        <span class="hljs-attr">MemorySize:</span> <span class="hljs-number">256</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">./.serverless/get_meta</span>

      <span class="hljs-string">......</span>
</code></pre>
<p data-nodeid="1252">细心的你可能发现了，在该 YAML 配置中，函数的 CodeUri 不是<code data-backticks="1" data-nodeid="1412">./functions/get_durtion</code>，而是<code data-backticks="1" data-nodeid="1414">./.serverless/get_meta</code>，这是为什么呢？</p>
<p data-nodeid="1253">这主要是因为我们需要对函数代码进行构建，<code data-backticks="1" data-nodeid="1417">./.serverless/get_duration</code>对应的是构建后的代码。之所以需要构建，是为了解决<code data-backticks="1" data-nodeid="1419">common/utils.js</code>代码共用的问题。</p>
<p data-nodeid="1254">如果不对代码进行构建，直接部署<code data-backticks="1" data-nodeid="1422">functions/get_duration</code>中的代码，函数执行时就会报错：<code data-backticks="1" data-nodeid="1424">Cannot find module '../common/utils</code>，因为<code data-backticks="1" data-nodeid="1426">common/utils.js</code>不在入口函数目录中，没有部署到 FaaS 上。</p>
<p data-nodeid="1255">要解决这个问题，就需要对代码进行构建，将函数及依赖的所有代码构建为单个文件，这样部署时就只需要部署一个文件，不涉及目录和依赖的问题了。</p>
<p data-nodeid="1256">我们可以使用 <a href="https://github.com/vercel/ncc" data-nodeid="1432">ncc</a> 这个工具对函数进行构建，使用方法如下：</p>
<pre class="lang-shell" data-nodeid="1257"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> ncc build ./<span class="hljs-built_in">functions</span>/get_duration/index.js -o ./.serverless/get_duration/ -e ali-oss</span>
</code></pre>
<p data-nodeid="1258">该命令就会将<code data-backticks="1" data-nodeid="1435">functions/get_duration/index.js</code>进行构建，最终会将<code data-backticks="1" data-nodeid="1437">index.js</code>以及缩依赖的 exec、getOSSClient 等方法进行编译，最终合并为一个文件并输出到<code data-backticks="1" data-nodeid="1439">./.serverless/get_duration/</code>目录中。</p>
<p data-nodeid="1259"><strong data-nodeid="1448">这里还需要注意的是</strong><code data-backticks="1" data-nodeid="1444">-e ali-oss</code>这个参数，含义是构建时，排除 ali-oss 这个依赖，也就是不将其编译到最终的<code data-backticks="1" data-nodeid="1446">index.js</code>文件中。这是因为函数计算的 Node.js 运行时内置了 ali-oss 模块，所以我们的构建产物就不需要包含 ali-oss 的代码了。</p>
<p data-nodeid="1260">处理对代码进行构建，我们还需要将 ffmpeg 和 ffprobe 复制到对应的函数目录中。最终我将这些步骤编写到了<code data-backticks="1" data-nodeid="1450">build.js</code>中，内容如下：</p>
<pre class="lang-javascript" data-nodeid="1261"><code data-language="javascript"><span class="hljs-comment">// build.js</span>
<span class="hljs-keyword">const</span> { exec } = <span class="hljs-built_in">require</span>(<span class="hljs-string">"./functions/common/utils"</span>);

<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">build</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-comment">// 清空编译目录</span>
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">"rm -rf .serverless/*"</span>);
  <span class="hljs-comment">// 编译 get_duration 函数</span>
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">"mkdir -p ./.serverless/get_duration"</span>);
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">`ncc build ./functions/get_duration/index.js -o ./.serverless/get_duration/ -e ali-oss`</span>);
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">"cp ./ffprobe ./.serverless/get_duration/ffprobe"</span>);
  <span class="hljs-comment">// 编译 get_meta 函数</span>
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">"mkdir -p ./.serverless/get_meta"</span>);
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">`ncc build ./functions/get_meta/index.js -o ./.serverless/get_meta/ -e ali-oss`</span>);
  <span class="hljs-keyword">await</span> exec(<span class="hljs-string">"cp ./ffprobe ./.serverless/get_meta/ffprobe"</span>);
 
  <span class="hljs-comment">//...</span>
}
build();
</code></pre>
<p data-nodeid="1262">然后我在 package.json 中添加了两个命令：</p>
<ul data-nodeid="1263">
<li data-nodeid="1264">
<p data-nodeid="1265"><code data-backticks="1" data-nodeid="1453">build</code>构建函数</p>
</li>
<li data-nodeid="1266">
<p data-nodeid="1267"><code data-backticks="1" data-nodeid="1455">deploy</code>构建并部署</p>
</li>
</ul>
<p data-nodeid="1268">例如你开发完成后需要部署，就可以直接运行：</p>
<pre class="lang-shell" data-nodeid="1269"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm run deploy</span>
<span class="hljs-meta">&gt;</span><span class="bash"> serverless-video@1.0.0 deploy</span>
<span class="hljs-meta">&gt;</span><span class="bash"> npm run build &amp;&amp; fun deploy</span>
<span class="hljs-meta">&gt;</span><span class="bash"> serverless-video@1.0.0 build</span>
<span class="hljs-meta">&gt;</span><span class="bash"> node build.js</span>
rm -rf .serverless/*
mkdir -p ./.serverless/get_duration
ncc build ./functions/get_duration/index.js -o ./.serverless/get_duration/ -e ali-oss

using template: template.yml
Waiting for service serverless-video to be deployed...
    Waiting for function get_duration to be deployed...
        Waiting for packaging function get_duration code...
        The function get_duration has been packaged. A total of 2 files were compressed and the final size was 15.2 MB
    function get_duration deploy success
......
service serverless-video deploy success
</code></pre>
<p data-nodeid="1270">部署成功后，我们就可以对函数进行测试了，可以直接在控制台上运行函数，也可以通过<code data-backticks="1" data-nodeid="1459">fun invoke</code>执行函数：</p>
<pre class="lang-java" data-nodeid="1271"><code data-language="java">$ fun invoke get_duration
{
    <span class="hljs-string">"format"</span>: {
        <span class="hljs-string">"duration"</span>: <span class="hljs-string">"170.859000"</span>
    }
}
</code></pre>
<h3 data-nodeid="1272">总结</h3>
<p data-nodeid="1273">今天这一讲我们学习了怎么基于 Serverless 实现一个音视频处理系统，在代码中我们使用到了 FFmpeg 进行视频处理。同时我也为你介绍了如何通过 ncc 进行代码构建，通过 ncc 我们可以将分散在多个文件中的函数代码构建为单个文件，这样就不用担心单个函数部署后找不到依赖的问题，同时还能减小代码体积。</p>
<p data-nodeid="1274">总的来说，我想要强调下面几点：</p>
<ul data-nodeid="1275">
<li data-nodeid="1276">
<p data-nodeid="1277">Serverless 除了适合 Web 接口、服务端渲染等场景，还适合 CPU 密集型的任务；</p>
</li>
<li data-nodeid="1278">
<p data-nodeid="1279">基于 Serverless 开发的音视频处理系统，本身就具备弹性、可扩展、低成本、免运维、高可用的能力；</p>
</li>
<li data-nodeid="1280">
<p data-nodeid="1281">对于需要通过代码执行的命令行工具等依赖，部署到 FaaS 平台之前需要为其设置可执行权限；若函数依需要调用其他云产品的接口，需要为函数授予相应权限；</p>
</li>
<li data-nodeid="1282">
<p data-nodeid="1283">对于添加水印、视频转码等消耗资源的操作，需要为函数设置较大的内存和超时时间。</p>
</li>
</ul>
<p data-nodeid="1284" class="">最后，本节课我留给你的作业是：亲自动手实现课上所学的视频处理程序。</p>

---

### 精选评论


