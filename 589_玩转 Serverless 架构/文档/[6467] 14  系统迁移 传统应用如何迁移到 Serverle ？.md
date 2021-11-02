<p data-nodeid="801">Serverless 有很多优点，可以让你不用关心运维、按量付费节省成本……所以很多同学一直想把已有应用迁移到 Serverless 架构上。但因为 Serverless 是一项新的技术，和传统开发方式区别很大，迁移成本也很大。</p>
<p data-nodeid="802">另外，基于 Serverless 架构的应用是由 FaaS 和 BaaS 组成的（FaaS 提供计算资源，BaaS 提供数据存储和服务），而传统应用的计算和存储都在同一台服务器上，所以传统应用要想迁移到 Serverless 架构上，就要进行相关的改造。</p>
<p data-nodeid="803">这一讲我会先从整体上带你了解传统应用迁移到 Serverless 架构的几个重要改造点，然后带你了解怎么把传统 Web 服务进行 Serverless 改造。希望你学完今天的内容之后，能知道自己的应用适不适合迁移到 Serverless 架构、具体怎么实现，以及迁移过程中有哪些需要注意的地方。</p>
<h3 data-nodeid="804">传统应用应该如何迁移到 Serverless</h3>
<p data-nodeid="805">传统应用的典型特点就是：应用进程是持续运行在服务器上的。</p>
<p data-nodeid="806">以 Web 服务为例，要部署一个应用，你要买服务器，然后把代码部署到服务器上，启动服务进程，监听服务器相关的端口，然后等待客户端请求，收到请求后进行处理并返回处理结构。这个服务进程是常驻的，就算没有客户端的请求，也会占用服务器资源。</p>
<p data-nodeid="807"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image2/M01/0C/93/Cip5yGAZAUSAcAJEAAE7WwPpQs0967.png" data-nodeid="887"></p>
<div data-nodeid="808"><p style="text-align:center">传统 Web 服务架构</p></div>
<p data-nodeid="809">因为应用进程常驻，同一个服务器上的内存可以共享，所以传统应用通常可以在内存中缓存数据，以便提升计算性能（比如在内存中保存用户信息），这样每次处理用户请求时，就可以从内存读取用户信息，不用查询数据库了。但基于 Serverless 架构的应用，内存缓存通常没有意义，因为函数生命周期有限，且函数实例之间无法共享内存。<strong data-nodeid="892">所以传统应用迁移到 Serverless 架构面临的第一个改造点就是内存缓存问题。</strong></p>
<p data-nodeid="810">在 Serverless 架构的应用中，我们一般不会用内存做缓存，而是用缓存数据库（比如 Redis）。当然，基于 Redis 的缓存，读写数据还是会经过网络请求，性能相比内存缓存有一定损耗。不过我个人认为，这一点不用特别担心，在传统应用中（尤其分布式应用），大部分时候我们也会使用缓存数据库，因为服务器和服务器之间，也无法共享内存，所以内存缓存也仅作用于当前服务器处理的所有请求。</p>
<p data-nodeid="811">此外，在传统应用中，我们通常也会使用二级缓存，同时将数据缓存在内存和缓存数据库中。读取缓存时，首先读取内存缓存，如果内存中没有数据，再读取缓存数据库中的数据，如果缓存数据库中也没有数据，再通过网络请求从远程读取数据。</p>
<p data-nodeid="812"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image2/M01/0C/96/CgpVE2AZAU6AIsGDAAF6SNrHN1c948.png" data-nodeid="897"></p>
<div data-nodeid="813"><p style="text-align:center">二级缓存</p></div>
<p data-nodeid="814">缓存带来的另一个问题就是身份认证，<strong data-nodeid="903">身份认证是传统应用迁移到 Serverless 的第二个改造点</strong>。传统应用的身份认证通常有 cookie-session 和 JWT 两种方式。</p>
<p data-nodeid="815">基于 cookie-session 的认证方式，通常是把身份信息保存在服务端的 session 中。对于只有一台服务器的应用，有的同学可能会把 session 保存在内存中，但在 Serverless 中就会有问题了，因为内存缓存是很短暂的。当然，现在大部分 cookie-session 的身份认证，也会将 session 存储在缓存数据库，这样就降低了迁移成本。另外，由于 JWT 的认证方式本身是无状态的，客户端和服务端通过一个加密后的 token 交换信息，所以比较适合 Serverelss 架构，可以无缝迁移（关于身份认证，我 15 讲会细说）。</p>
<p data-nodeid="816">除了对内存读写，一些传统应用可能还会对磁盘有很多读写操作。比如我们可能会基于磁盘做重试，当一条数据处理失败后，我们就将其写入磁盘，然后启动另一个线程读取磁盘数据进行重试。部署传统应用的磁盘是直接挂载到服务器上的，所以就算应用重启了，服务器和磁盘也依旧存在，所以将数据直接写入磁盘不会造成数据丢失。</p>
<p data-nodeid="817">而 Serverless 函数是运行在 FaaS 平台上的，函数运行时只会有一个临时目录的读写权限，一旦运行环境被释放，该临时目录也会被释放，所以磁盘数据无法持续存储。并且和内存问题类似，不同函数实例的临时目录也是独立的。<strong data-nodeid="914">那么对于有读写磁盘需求的应用，应该如何迁移到 Serverless 架构呢？</strong> 要解决这个问题，我们可以为 Serverless 函数挂载一个持久存储，比如云盘或 NAS 等，这些持久化存储和 FaaS 平台是相互独立的，只要不释放数据可以永久保存。并且不同函数可以共用同一个持久化存储，这样不同函数就可以读写同一份数据了，甚至函数间还可以基于持久化存储进行通信。采用持久化存储还有一个好处就是，计算和存储分离了，这样更利于应用扩缩容。<strong data-nodeid="915">总的来说，数据持久化是传统应用迁移到 Serverless 的第三个改造点。</strong></p>
<p data-nodeid="818"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image2/M01/0C/93/Cip5yGAZAVqAEQ_ZAARFv9c7O6A392.png" data-nodeid="918"></p>
<p data-nodeid="819">其实不难看出，如果传统应用本身是分布式架构，很容易满足前面三点。因为分布式架构的应用就需要考虑内存缓存、身份认证、持久化存储等问题，而 Serverless 架构本身也是分布式的。</p>
<p data-nodeid="820">对于传统分布式应用，要对外提供 HTTP 服务，通常也会有一个统一接入层来实现负载均衡、高可用等，例如我们会通过负载均衡使用户流量均衡分配到背后的每台服务器上，其中可能会使用到 Nginx、SLB 等产品。而对于 Serverless 架构的应用，我们可以使用 API 网关来做统一接入，由 API 网关承接用户请求，然后触发具体函数的执行。</p>
<p data-nodeid="821"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image2/M01/0C/93/Cip5yGAZAWCALB5LAAG1Ybf7G7c701.png" data-nodeid="923"></p>
<p data-nodeid="822">使用 API 网关做统一接入是架构上的改造，除此之外，应用代码也需要改造。因为在传统应用中，运行在服务器上的应用是直接处理来自用户的 HTTP 请求的，而在 Serverless 应用中，函数处理的是 API 网关的事件，两者的数据结构和请求响应方式都有很大差异。你要对传统应用提供 HTTP 服务的代码进行改造，才能使其部署在 Serverless 平台上，<strong data-nodeid="928">所以将传统 Web 服务 Serverless 化是传统应用迁移到 Serverless 架构的又一个改造点。</strong></p>
<p data-nodeid="823">接下来我就以一个具体的开发框架 Express.js 为例，通过将 Express.js 框架 Serverless 化，为你详细介绍如何将传统 Web 服务 Serverless 化。</p>
<h3 data-nodeid="824">Web 服务如何 Serverless 化</h3>
<p data-nodeid="825">传统的 Web 服务请求参数与 Serverless 函数参数有较大差异，所以将 Web 服务 Serverless 化的核心工作就是开发一个适配层，通过适配层将函数的事件对象转化为标准的 Web 请求，这样我们就可以接着用传统 Web 服务去处理用户请求和响应了。整体流程如下图所示：</p>
<p data-nodeid="826"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image2/M01/0C/96/CgpVE2AZAWmAchYZAAHYDXbY_9c062.png" data-nodeid="934"></p>
<div data-nodeid="827"><p style="text-align:center">传统 Web 服务 Serverless 化流程</p></div>
<p data-nodeid="828">在 03 和 04 讲中，我们已经学习了 Serverless 函数是由事件触发的，事件形式上就是一个 JSON 数据，不同事件的数据不一样，Serverless 平台接收到事件后，会以事件对象为参数来执行入口函数。</p>
<p data-nodeid="829">函数的事件对象就是入口函数的参数，下面是函数计算 API 网关的入口函数定义：</p>
<pre class="lang-javascript" data-nodeid="830"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function">(<span class="hljs-params">event, context, callback</span>) =&gt;</span> {
  <span class="hljs-comment">// 处理业务逻辑</span>
  callback(<span class="hljs-literal">null</span>, res);
};
</code></pre>
<p data-nodeid="831">在传统应用中，与入口函数对应的概念就是入口文件，比如 Express.js 入口文件定义：</p>
<pre class="lang-javascript" data-nodeid="832"><code data-language="javascript"><span class="hljs-keyword">const</span> express = <span class="hljs-built_in">require</span>(<span class="hljs-string">'express'</span>)
<span class="hljs-keyword">const</span> app = express()
<span class="hljs-keyword">const</span> port = <span class="hljs-number">3000</span>
<span class="hljs-comment">// 监听 / 路由，处理请求</span>
app.get(<span class="hljs-string">'/'</span>, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">request, response</span>) </span>{
  response.send(<span class="hljs-string">'Hello World!'</span>)
})
<span class="hljs-comment">// 监听 3000 端口，启动 HTTP 服务</span>
app.listen(port, () =&gt; {
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Example app listening at http://localhost:<span class="hljs-subst">${port}</span>`</span>)
})
</code></pre>
<p data-nodeid="833">在 Express.js 入口文件中，我们在第 6 行监听了<code data-nodeid="939" data-backticks="1">/</code>路由，该路由对应的请求由回调函数<code data-nodeid="941" data-backticks="1">function(request, response)</code>进行处理，所以我们需要把 Serverless 函数的 event 对象转换为 Express.js 框架的<code data-nodeid="943" data-backticks="1">request</code>对象，然后将 Epxress.js 框架的<code data-nodeid="945" data-backticks="1">response</code>对象转换为函数的 callback 参数。在 Express.js 入口文件的第 11 行，我们通过<code data-nodeid="947" data-backticks="1">app.listen</code>启动了 HTTP 服务，其本质上是调用的 Node.js http 模块<code data-nodeid="949" data-backticks="1">createServer()</code>方法。</p>
<p data-nodeid="834">所以要想将 event 对象转换为 request 参数，我们首先就需要创建一个自定义的 HTTP &nbsp;服务来代替 Express.js 的<code data-nodeid="952" data-backticks="1">app.listen</code>。大致代码如下：</p>
<pre class="lang-javascript" data-nodeid="835"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">createServer</span>(<span class="hljs-params">requestListener, serverListenCallback</span>) </span>{
    <span class="hljs-comment">// 创建一个自定义的 HTTP 服务（Node.js Server）</span>
    <span class="hljs-keyword">const</span> server = http.createServer(requestListener);
    <span class="hljs-comment">// 生成一个 Unix Domain Socket</span>
    server.socketPathSuffix = getRandomString();
    server.on(<span class="hljs-string">"listening"</span>, () =&gt; {
        server.isListening = <span class="hljs-literal">true</span>;
        <span class="hljs-keyword">if</span> (serverListenCallback) serverListenCallback();
    });
    server
        .on(<span class="hljs-string">"close"</span>, () =&gt; {
            server.isListening = <span class="hljs-literal">false</span>;
        })
        .on(<span class="hljs-string">"error"</span>, (error) =&gt; {
            <span class="hljs-comment">// 异常处理，例如判读 socket 是否已被监听</span>
        });
    <span class="hljs-comment">// 监听 Unix Domain Socket，启动 Node.js Server</span>
    server.listen(<span class="hljs-string">`/tmp/server-<span class="hljs-subst">${server.socketPathSuffix}</span>.sock`</span>);
    <span class="hljs-keyword">return</span> server;
}
</code></pre>
<p data-nodeid="836">在上述代码中，重点在第 22 行。我们首先创建了一个 Node.js Server，然后随机生成了一个随机的 &nbsp;Unix Domain Socket，最后监听该 Socket 启动 HTTP 服务 （Node.js Server）。</p>
<p data-nodeid="837">然后我们就可以将函数的事件对象 event 转换为 Express.js 的 request 请求：</p>
<pre class="lang-javascript" data-nodeid="838"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">forwardRequestToNodeServer</span>(<span class="hljs-params">server, event, context, resolver</span>) </span>{
  <span class="hljs-keyword">try</span> {
    <span class="hljs-comment">// 将 API 网关事件转换为 HTTP Request</span>
    <span class="hljs-keyword">const</span> requestOptions = mapApiGatewayEventToHttpRequest(
      event,
      context,
      getSocketPath(server.socketPathSuffix)
    );
    <span class="hljs-comment">// 将 HTTP 请求转发到 Node.js Server</span>
    <span class="hljs-keyword">const</span> req = http.request(requestOptions, (response) =&gt;
      forwardResponseToApiGateway(server, response, resolver)
    );
    req
      .on(<span class="hljs-string">"error"</span>, (error) =&gt; {
        <span class="hljs-comment">// 处理异常</span>
        <span class="hljs-keyword">return</span> forwardLibraryErrorResponse(error, resolver);
      })
      .end();
  } <span class="hljs-keyword">catch</span> (error) {
    <span class="hljs-comment">// 处理异常</span>
    <span class="hljs-keyword">return</span> forwardLibraryErrorResponse(error, resolver);
  }
}
</code></pre>
<p data-nodeid="839">**这段代码的主要功能就是：**先将 event 对象转换为 HTTP 请求参数，然后以该参数向 Node.js Server 发起请求，这样函数的请求就会被 Node.js Server 接管，由 Node.js 再进行处理。</p>
<p data-nodeid="840">当 Node.js Server 处理完毕后，函数还需要对 Node.js Server 的 response 做处理，将 response 信息转换为 callback 函数的参数，该参数会直接返回给 API 网关：</p>
<pre class="lang-javascript" data-nodeid="841"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">forwardResponseToApiGateway</span>(<span class="hljs-params">server, response, resolver</span>) </span>{
  <span class="hljs-keyword">const</span> buf = [];
  response
    .on(<span class="hljs-string">"data"</span>, (chunk) =&gt; buf.push(chunk))
    .on(<span class="hljs-string">"end"</span>, () =&gt; {
      <span class="hljs-comment">// 根据 response 构造函数执行结果</span>
      <span class="hljs-keyword">const</span> data = {
        <span class="hljs-attr">statusCode</span>: response.statusCode,
        <span class="hljs-attr">body</span>: Buffer.concat(buf),
        <span class="hljs-attr">headers</span>: getResponseHeaders(response),
        <span class="hljs-attr">isBase64Encoded</span>: isContentTypeBinaryMimeType(response),
      };
      <span class="hljs-comment">// 返回函数执行结果</span>
      resolver(data);
    });
}
</code></pre>
<p data-nodeid="842">这段代码重点就在与第 7 行和 第 14 行。功能分别是根据 response 构建函数执行结果和返回函数执行结果。<br>
至此，Express.js 框架 Serverless 化的核心代码就完成了。你可以像下面这样使用：</p>
<pre class="lang-javascript" data-nodeid="843"><code data-language="javascript"><span class="hljs-keyword">const</span> express = <span class="hljs-built_in">require</span>(<span class="hljs-string">'express'</span>);
 
<span class="hljs-keyword">const</span> app = express();
app.all(<span class="hljs-string">'*'</span>, (req, res) =&gt; {
  res.send(<span class="hljs-string">'hello world!'</span>);
});
 
<span class="hljs-comment">// 创建一个自定义 Node.js Server</span>
<span class="hljs-keyword">const</span> server = createServer(app);
 
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 将 event 对象转换为 HTTP reqest 并转发到 Node.js Server</span>
  forwardRequestToNodeServer(server, event, context, callback)
};
</code></pre>
<p data-nodeid="844">首先如第 9 行所示，创建一个自定义的 Node.js Server，然后再在函数里面将 event 对象转换为 HTTP request 并转发到 Node.js Server。</p>
<p data-nodeid="845">传统 Web &nbsp;框架 Serverless 化比较麻烦的地方就在于你需要完全理解 Web 框架和 Serverless 函数事件的每个参数，所以各个云厂商为了方便我们进行开发，也提供了相应的依赖包，比如阿里云函数计算提供的<a href="https://github.com/awesome-fc/webserverless" data-nodeid="970">@webserverless/fc-express</a>和腾讯云云函数提供的<a href="https://github.com/serverless-plus/tencent-serverless-http" data-nodeid="974">tencent-serverless-http</a>。</p>
<p data-nodeid="846">虽然这里我只举了 Express.js 框架的例子，但其他 Web 框架 Serverless 化的原理也基本类似，大体需要三个步骤：</p>
<ul data-nodeid="847">
<li data-nodeid="848">
<p data-nodeid="849">创建一个自定义 HTTP Server ；</p>
</li>
<li data-nodeid="850">
<p data-nodeid="851">将事件对象转换为 HTTP 请求参数，并转发到自定义的 HTTP Server；</p>
</li>
<li data-nodeid="852">
<p data-nodeid="853">将 HTTP 响应转换为函数返回值。</p>
</li>
</ul>
<p data-nodeid="854"><img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image2/M01/0C/93/Cip5yGAZAX6AFyFKAALzp61gPVQ447.png" data-nodeid="982"></p>
<div data-nodeid="855"><p style="text-align:center">Web 服务 Serverless 化的实现原理</p></div>
<p data-nodeid="856">说到这儿，不知道你有没有想起我们前面所讲的内容？我们在“07｜运行时：使用自定义运行时支持自定义编程语言”学习了如何实现一个自定义运行时，其中自定义运行时的原理，本质上也是在函数中实现一个 HTTP 服务，接下来 FaaS 平台会将事件转发到我们的 HTTP 服务上。不难发现，传统 Web 服务 Serverless 化的原理与自定义运行时的原理是非常类似的。因此基于自定义运行时，我们也可以很轻松将传统 Web 服务 Serverless 化，这样还不用开发适配层。</p>
<p data-nodeid="857">自定义运行时有两种方式，一种是在函数中直接创建 HTTP 服务，另一种是通过 Docker 镜像启动一个 HTTP 服务。所以总的来说，我们有下面几种方案可以将传统 Web 服务 Serverless 化：</p>
<ul data-nodeid="858">
<li data-nodeid="859">
<p data-nodeid="860">通过适配层，由适配层将事件对象转换为标准 Web 请求；</p>
</li>
<li data-nodeid="861">
<p data-nodeid="862">通过自定义运行时，在函数中创建一个 HTTP 服务，该 HTTP 服务将函数事件处理后转发给传统 Web 服务；</p>
</li>
<li data-nodeid="863">
<p data-nodeid="864">通过自定义运行时，将传统 Web 服务构建为自定义镜像。</p>
</li>
</ul>
<p data-nodeid="865">如果你的应用比较简单，可以使用第一种或第二种方案，开发起来最方便；如果你的应用有很多依赖，或者需要使用 FaaS 平台不支持的编程语言，建议使用第三种方案，这样可以把依赖都打包到 Docker 镜像中，不过这就需要依赖容器镜像服务了。</p>
<h3 data-nodeid="866">总结</h3>
<p data-nodeid="867">这一讲，我们学习了怎么把传统应用迁移到 Serverless，并且我通过将 Express.js 框架 Serverless 化的例子，为你详细介绍了应该如何把一个传统 Web 服务迁移到 Serverless。关于这一讲的内容，我想要强调这样几点：</p>
<ul data-nodeid="3850">
<li data-nodeid="3851">
<p class="te-preview-highlight" data-nodeid="3852">传统应用迁移到 Serverless，需要考虑内存缓存、身份认证、持久化存储、Web 服务 Serverless 化等改造点；</p>
</li>
<li data-nodeid="3853">
<p data-nodeid="3854">如果一个应用本身就是分布式部署的，且在架构上是计算和存储分离的，则比较容易迁移到 Serverless；</p>
</li>
<li data-nodeid="3855">
<p data-nodeid="3856">Web 服务 Serverless 化主要原理是实现一个自定义 HTTP 服务，通过该 HTTP 服务处理事件对象和 Web 请求的差异；</p>
</li>
<li data-nodeid="3857">
<p data-nodeid="3858">我们可以通过实现适配层和自定义运行时等方案，实现 Web 服务 Serverless 化。</p>
</li>
</ul>








<p data-nodeid="877">虽然这一讲学习了很多关于将传统应用迁移到 Serverless 的方案，但实际操作起来可能还会遇到一些未知困难。不过我觉得相比困难，Serverless 带给我们的收益是值得去尝试的。</p>
<p data-nodeid="878">最后，我留给你的作业是：开发一个简单的 Express.js 应用，并将其部署到 Serverless 平台上。我们下一讲见。</p>

---

### 精选评论


