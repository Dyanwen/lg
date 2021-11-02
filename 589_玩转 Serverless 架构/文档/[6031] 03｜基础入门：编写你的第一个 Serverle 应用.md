<p data-nodeid="5123" class="">从今天开始，我们正式进入 Serverless 的开发阶段。</p>
<p data-nodeid="5124">学习一门新技术，除了了解其基础概念，更重要的是把理论转化为实践，所以学会开发 Serverless 应用尤为重要。考虑到很多刚开始接触 Serverless 开发的同学在短时间很难适应 Serverless 的开发思想，知识也不够体系化，所以我除了带你实现一个 Serverless 应用之外，还会介绍应用开发时涉及的重要知识点，让你更深刻地理解 Serverless，建立属于自己的知识体系。</p>
<h3 data-nodeid="5125">选择适合的 FaaS 平台</h3>
<p data-nodeid="5126">在开发 Serverless 应用之前，你需要了解并选择一个 Serverless FaaS 平台，因为你要用 FaaS 运行代码。</p>
<p data-nodeid="5127">目前主流的 FaaS 产品有 AWS Lambda、阿里云函数计算等。不同 FaaS 支持的编程语言和触发器不尽相同，为了让你更快地了解它们异同点，我提供给你一个简单的对比图：</p>
<p data-nodeid="5128"><img src="https://s0.lgstatic.com/i/image2/M01/04/20/Cip5yF_puUaAJfw3AANPRrI1kS820.jpeg" alt="Lark20201228-185348.jpeg" data-nodeid="5262"></p>
<p data-nodeid="5129">从表格中，你可以总结出这样几点信息。</p>
<ul data-nodeid="5130">
<li data-nodeid="5131">
<p data-nodeid="5132">FaaS 平台都支持 Node.js、Python 、Java 等编程语言；</p>
</li>
<li data-nodeid="5133">
<p data-nodeid="5134">FaaS 平台都支持 HTTP 和定时触发器（这两个触发器最常用）。此外各厂商的 FaaS 支持与自己云产品相关的触发器，函数计算支持阿里云表格存储等触发器；</p>
</li>
<li data-nodeid="5135">
<p data-nodeid="5136">FaaS 的计费都差不多，且每个月都提供一定的免费额度。其中 GB-s 是指函数每秒消耗的内存大小，比如1G-s 的含义就是函数以 1G 内存执行 1 秒钟。超出免费额度后，费用基本都是 0.0133元/万次，0.00003167元/GB-s。所以，用 FaaS 整体费用非常便宜，对一个小应用来说，几乎是免费的。</p>
</li>
</ul>
<p data-nodeid="5137">总的来说，国外开发者经常用 Lambda，相关的第三方产品和社区更完善，国内经常用函数计算，因为函数计算使用方式更符合国内开发者的习惯。</p>
<p data-nodeid="5138">了解了主流 FaaS 的基本信息之后，<strong data-nodeid="5273">怎么用 FaaS 去开发 Serverless 应用呢？</strong> 为了让你对不同 FaaS 有更多了解，方便以后进行技术选型，这一讲我主要用函数计算做演示，同时也会讲述基于 Lambda 的实现。</p>
<h3 data-nodeid="5139">开发 Serverless 应用的步骤</h3>
<p data-nodeid="5140"><strong data-nodeid="5281">这个 Serverless 应用的功能是：</strong> 提供一个所有人都可访问的 “Hello World!” 接口，并且能够根据请求参数进行响应（比如请求 http://example.com/?name=Serverless 返回 Hello, Serverless&nbsp;）。</p>
<p data-nodeid="5141">在学这一讲之前，你可能直接用 Node.js 中的 Express.js 框架定义一个路由，接收 HTTP 请求，然后处理请求参数并返回结果，代码如下：</p>
<pre class="lang-javascript" data-nodeid="5142"><code data-language="javascript"><span class="hljs-comment">// index.js</span>
<span class="hljs-keyword">const</span> express = <span class="hljs-built_in">require</span>(<span class="hljs-string">'express'</span>)
<span class="hljs-keyword">const</span> app = express()
<span class="hljs-keyword">const</span> port = <span class="hljs-number">3000</span>
<span class="hljs-comment">// 定义路由</span>
app.get(<span class="hljs-string">'/hello'</span>, (req, res) =&gt; {
  <span class="hljs-keyword">const</span> { name } = req.request.query;
  res.send(<span class="hljs-string">`Hello <span class="hljs-subst">${name}</span>!`</span>);
});
<span class="hljs-comment">// 启动服务</span>
app.listen(port, () =&gt; {
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Example app listening at http://localhost:<span class="hljs-subst">${port}</span>`</span>)
});
</code></pre>
<p data-nodeid="5143"><strong data-nodeid="5296">那怎么把接口分享给别人呢？</strong>（我在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=589#/detail/pc?id=6030" data-nodeid="5291">02 | 概念新知：到底什么是 Serverless?</a>”的时候提到了这部分内容，你可以回顾一下，我就不多说了）。当然如果你对域名解析、Nginx 配置等流程不熟悉，那就需要耗费很多时间和精力了。<strong data-nodeid="5297">可能代码开发几分钟，部署上线几小时。</strong></p>
<p data-nodeid="5479" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/28/Cip5yF_qmTqAJI5SAAGC15t2JGc253.png" alt="1.png" data-nodeid="5483"></p>
<div data-nodeid="5480"><p style="text-align:center"><span style="color:#b8b8b8">传统应用开发流程</span></p></div>


<p data-nodeid="5146">而基于 Serverless FaaS 平台进行开发就很简单了，你开发完的函数代码部署到 FaaS 平台并为函数配置 HTTP 触发器，FaaS 会自动帮你初始化运行环境，并且 HTTP 触发器会自动为你提供一个测试域名。</p>
<p data-nodeid="6196" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/50/CgqCHl_qmUWAbbsDAADn5znl1KY310.png" alt="2.png" data-nodeid="6200"></p>
<div data-nodeid="6197"><p style="text-align:center"><span style="color:#b8b8b8">Serverless 应用开发流程</span></p></div>


<p data-nodeid="5149"><strong data-nodeid="5316">以函数计算为例，</strong> 你可以直接点击<a href="https://fc.console.aliyun.com/fc/service/cn-hangzhou/function/create" data-nodeid="5311">新建函数</a>的链接进入函数计算控制台，新建一个函数。<strong data-nodeid="5317">请注意，你会用到 HTTP 触发器，函数计算的 HTTP 触发器要在创建函数时就指定。</strong></p>
<p data-nodeid="5150"><img src="https://s0.lgstatic.com/i/image2/M01/03/D5/CgpVE1_jBXmAb9TJAAPuNbsOSKo931.png" alt="Drawing 2.png" data-nodeid="5320"></p>
<p data-nodeid="5151">函数创建成功后，就会进入到代码编辑页面，在编辑器中写入如下代码：</p>
<pre class="lang-javascript" data-nodeid="5152"><code data-language="javascript"><span class="hljs-comment">// 函数计算</span>
exports.handler = <span class="hljs-function">(<span class="hljs-params">request, response, context</span>) =&gt;</span> {
    <span class="hljs-comment">// 从 request 中获取</span>
    <span class="hljs-keyword">const</span> { name } = request.queries;
    <span class="hljs-comment">// 设置 HTTP 响应</span>
    response.setStatusCode(<span class="hljs-number">200</span>);
    response.setHeader(<span class="hljs-string">"Content-Type"</span>, <span class="hljs-string">"application/json"</span>);
    response.send(<span class="hljs-built_in">JSON</span>.stringify({ <span class="hljs-attr">message</span>: <span class="hljs-string">`Hello, <span class="hljs-subst">${name}</span>`</span> }));
 }
</code></pre>
<p data-nodeid="5153">这段代码就是 Serverless 应用的全部代码了，它本质上是一个函数，函数有三个参数。</p>
<ul data-nodeid="5154">
<li data-nodeid="5155">
<p data-nodeid="5156">request 是请求信息，你可以从中获取到请求参数；</p>
</li>
<li data-nodeid="5157">
<p data-nodeid="5158">response 是响应对象，你可以用它来设置 HTTP 响应数据；</p>
</li>
<li data-nodeid="5159">
<p data-nodeid="5160">context&nbsp; 是函数上下文，后面会讲到。</p>
</li>
</ul>
<p data-nodeid="5161">由此可见，相比 Express.js 框架，基于 FaaS 的 Serverless 应用的代码更简单，就像写一个普通函数，接收参数并处理，然后返回。写完代码后，你就可以在“触发器”标签下看到函数计算为你默认创建的 API Endpoint：</p>
<p data-nodeid="5162" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/F1/Ciqc1F_jBYOAFmttAAJZChiCxxw894.png" alt="Drawing 3.png" data-nodeid="5329"></p>
<p data-nodeid="5163">然后你可以用该 API Endpoint 对函数进行测试。我们通过 curl 命令测试一下：</p>
<pre class="lang-shell" data-nodeid="5164"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> curl https://1457216987974698.cn-hangzhou.fc.aliyuncs.com/2016-08-15/proxy/serverless/hello-world/\?name\=Serverless</span>
Hello Serverless!
</code></pre>
<p data-nodeid="5165">接口也按照预期返回了 Hello Serverless! 。<br>
<strong data-nodeid="5343">需要注意的是，</strong> 如果你直接用浏览器访问函数计算默认提供的 API Endpoint ，函数计算 HTTP 触发器会默认在 response headers 中强制添加 content-disposition: attachment 字段，这会让返回结果在浏览器中以附件的方式下载，<strong data-nodeid="5344">解决的方法就是使用自定义域名。</strong> 所以如果应用是一个线上服务，还是建议你购买自定义域名绑定到你的函数上。</p>
<p data-nodeid="5166">至此你就基于函数计算完成了 Serverless 应用的开发、部署和测试。</p>
<p data-nodeid="5167"><strong data-nodeid="5354">如果你用的是 Lambda，</strong> 需要在<a href="https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions" data-nodeid="5352">Lambda 控制台</a>先创建一个函数，然后为函数添加 API 网关触发器。Lambda 没有直接提供 HTTP 触发器，但可以用 API 网关触发器来实现通过 HTTP 请求触发函数执行。同样 Lambda 也会默认提供给你一个测试域名，代码如下：</p>
<pre class="lang-javascript" data-nodeid="5168"><code data-language="javascript"><span class="hljs-comment">// Lambda</span>
exports.handler = <span class="hljs-function">(<span class="hljs-params">event, context, callback</span>) =&gt;</span> {
    <span class="hljs-comment">// 从 event 中获取 URL query 参数</span>
    <span class="hljs-keyword">const</span> { name } = event.queryStringParameters;
    <span class="hljs-comment">// 定义 HTTP Response</span>
    <span class="hljs-keyword">const</span> response = {
        <span class="hljs-attr">statusCode</span>: <span class="hljs-number">200</span>,
        <span class="hljs-attr">headers</span>: {
            <span class="hljs-string">"Content-Type"</span>: <span class="hljs-string">"application/json"</span>
            },
        <span class="hljs-attr">body</span>: <span class="hljs-built_in">JSON</span>.stringify({ <span class="hljs-attr">message</span>: <span class="hljs-string">`Hello <span class="hljs-subst">${name}</span>!`</span>} ),
    };
    callback(<span class="hljs-literal">null</span>, response);
};
</code></pre>
<p data-nodeid="5169">与 HTTP 触发器不同的是，API 网关触发器入参是 event&nbsp;，event 对象就是 HTTP 请求信息。相比而言，函数计算的 HTTP 触发器请求处理方式，和你用 Express.js 框架来处理请求更类似，而 API 网关触发器则是普通函数。<strong data-nodeid="5359">这也是我更喜欢函数计算的一个原因。</strong></p>
<p data-nodeid="5170">总的来说，开发一个 Serverless 应用可以分为三个步骤：代码开发、函数部署、触发器创建。那在这个过程中，你有没有思考过这样几个问题：为什么函数名是 handler？函数的参数又是怎么定义的呢？要弄清楚这些问题，你需要学习背后的基础知识。</p>
<h3 data-nodeid="5171">开发 Serverless 应用的基础知识</h3>
<h4 data-nodeid="5172">入口函数</h4>
<p data-nodeid="5173">我们开发传统应用时，编写的第一行代码一般都是入口函数，比如 Java 的 main 函数，而 Serverless 应用是由一个个函数组成的，与 main 函数对应的就是 FaaS 中的入口函数。</p>
<p data-nodeid="5174">在你创建 FaaS 时，会填写一个 “函数入口”，默认是 index.handler 含义为：运行 index 文件中的 handler 函数，handler 函数就是 “入口函数”。 FaaS 运行函数时，会根据“函数入口”加载代码中的“入口函数”。<strong data-nodeid="5368">这也就是为什么例子中的函数名是 handler。</strong></p>
<p data-nodeid="5175">在开发 “Hello World” 应用时，你只编写了一个 index.js 文件，文件中也只有一个 handler 方法，函数入口就是 index.handler 。</p>
<p data-nodeid="5176">当然，FaaS 函数可以包含多个源文件，然后按照编程语言的模块机制相互引入，这和传统的编程没有区别。比如 Hello World 应用可以拆分一个 sayHello 函数到 hello.js 文件中，然后在 index.js 中引入 hello.js ，函数的入口还是 index.hanlder 。</p>
<pre class="lang-javascript" data-nodeid="5177"><code data-language="javascript"><span class="hljs-comment">// logic.js</span>
exports.sayHello = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">name</span>) </span>{
  <span class="hljs-keyword">return</span> <span class="hljs-string">`Hello, <span class="hljs-subst">${name}</span>!`</span>;
}
</code></pre>
<pre data-nodeid="5178"><code>// index.js
const logic = require('./logic');
exports.handler = (request, response, context) =&gt; {
  // 从 request 中获取
  const { name } = request.queries;

  // 处理业务逻辑
  const message = logic.sayHello(name)

  // 设置 HTTP 响应
  response.setStatusCode(200);
  response.setHeader("Content-Type", "application/json");
  response.send(JSON.stringify({ message })); 
}
</code></pre>
<p data-nodeid="5179"><strong data-nodeid="5375">这也是我的建议：把业务逻辑拆分到入口函数之外</strong>。另外，你也可以用一份源文件去创建多个函数。因为应用通常是由多个函数组成，为了方便管理，你可能会在一个源文件中编写所有的入口函数。</p>
<p data-nodeid="6913" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/50/CgqCHl_qmV-AKjTHAAGR7LVSJSs782.png" alt="3.png" data-nodeid="6917"></p>
<div data-nodeid="6914"><p style="text-align:center"><span style="color:#d8d8d8"><span style="color:#b8b8b8">使用一份源文件创建多个函数</span></span></p></div>


<p data-nodeid="5182">既然你知道了函数名 handler 的由来， 那handler 函数具体是怎么定义的呢？</p>
<h4 data-nodeid="5183">函数定义</h4>
<p data-nodeid="5184">你能发现，在 HTTP 触发器中函数定义是 function(request, response, context)，在 API 网关触发器中，函数定义是 function(event, context) ，<strong data-nodeid="5385">所以，函数定义本质上是由触发器和编程语言决定的。</strong></p>
<p data-nodeid="5185">标准的函数定义是 function(event, context)。event 是事件对象。在 Serverless 中，触发器被称为“事件源”，英文是 event，这解释了为什么函数参数名是 event。</p>
<p data-nodeid="5186">触发器不同，event 的值可能不同。部分特殊触发器的函数定义可能和标准函数定义不一样，比如 HTTP 触发器，它其实是对标准函数的进一步封装，主要是为了方便对 HTTP 请求进行处理。这也是为什么在函数计算中，HTTP 触发器必须在创建函数时就指定，并且一旦创建后就不能修改（关于 event 的属性，后面我会详细讲）。</p>
<p data-nodeid="5187">第二个参数 context 是函数上下文，包含了函数运行时的一些信息（比如函数名称、运行函数的账号信息等）。同一 FaaS 平台中所有触发器的 context 对象都类似，来看一个例子：</p>
<pre class="lang-javascript" data-nodeid="5188"><code data-language="javascript">{
  <span class="hljs-attr">requestId</span>: <span class="hljs-string">'cd234b13-a145-468a-b640-17a8bf5e2ef2'</span>,
  <span class="hljs-attr">credentials</span>: {
    <span class="hljs-attr">accessKeyId</span>: <span class="hljs-string">'***'</span>,
    <span class="hljs-attr">accessKeySecret</span>: <span class="hljs-string">'***'</span>,
    <span class="hljs-attr">securityToken</span>: <span class="hljs-string">'***'</span>
  },
  <span class="hljs-attr">function</span>: {
    <span class="hljs-attr">name</span>: <span class="hljs-string">'hello-world'</span>,
    <span class="hljs-attr">handler</span>: <span class="hljs-string">'index.handler'</span>,
    <span class="hljs-attr">memory</span>: <span class="hljs-number">512</span>,
    <span class="hljs-attr">timeout</span>: <span class="hljs-number">60</span>,
    <span class="hljs-attr">initializer</span>: <span class="hljs-literal">undefined</span>,
    <span class="hljs-attr">initializationTimeout</span>: <span class="hljs-literal">NaN</span>
  },
  <span class="hljs-attr">service</span>: {
    <span class="hljs-attr">name</span>: <span class="hljs-string">'serverless'</span>,
    <span class="hljs-attr">logProject</span>: <span class="hljs-string">'aliyun-fc-cn-hangzhou-***'</span>,
    <span class="hljs-attr">logStore</span>: <span class="hljs-string">'function-log'</span>,
    <span class="hljs-attr">qualifier</span>: <span class="hljs-string">'LATEST'</span>,
    <span class="hljs-attr">versionId</span>: <span class="hljs-string">''</span>
  },
  <span class="hljs-attr">region</span>: <span class="hljs-string">'cn-hangzhou'</span>,
  <span class="hljs-attr">accountId</span>: <span class="hljs-string">'145***'</span>,
  <span class="hljs-attr">retryCount</span>: <span class="hljs-number">0</span>
}
</code></pre>
<p data-nodeid="5189">为了让你更形象地了解函数定义，我列举了几种不同编程语言的入口函数示例：</p>
<pre class="lang-javascript" data-nodeid="5190"><code data-language="javascript"><span class="hljs-comment">// Node.js 异步函数</span>
exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
    callback(<span class="hljs-literal">null</span>, <span class="hljs-string">"Hello World!"</span>);
}
</code></pre>
<pre class="lang-javascript" data-nodeid="5191"><code data-language="javascript"><span class="hljs-comment">// Node.js 同步函数，目前仅 Lambda 支持</span>
exports.handler = <span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context</span>) </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-string">"Hello World!"</span>
}
</code></pre>
<pre class="lang-javascript" data-nodeid="5192"><code data-language="javascript"><span class="hljs-comment">// Python</span>
def handler(event, context):
  <span class="hljs-keyword">return</span> <span class="hljs-string">"Hello World!"</span>
</code></pre>
<pre class="lang-javascript" data-nodeid="5193"><code data-language="javascript"><span class="hljs-comment">// Java</span>
package example;
<span class="hljs-keyword">import</span> com.aliyun.fc.runtime.Context;
<span class="hljs-keyword">import</span> com.aliyun.fc.runtime.StreamRequestHandler;
<span class="hljs-keyword">import</span> java.io.IOException;
<span class="hljs-keyword">import</span> java.io.InputStream;
<span class="hljs-keyword">import</span> java.io.OutputStream;
public <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloWorld</span> <span class="hljs-title">implements</span> <span class="hljs-title">StreamRequestHandler</span> </span>{
    @Override
    public <span class="hljs-keyword">void</span> handleRequest(
            InputStream inputStream, OutputStream outputStream, Context context) throws IOException {
        outputStream.write(<span class="hljs-keyword">new</span> <span class="hljs-built_in">String</span>(<span class="hljs-string">"hello world"</span>).getBytes());
    }
}
</code></pre>
<p data-nodeid="5194">由此可见，Node.js 和 Python 的代码是最简洁的。 Node.js 编写简单，开发人员众多，前后端同学都很容易上手，所以它在 Serverless 中最受欢迎。</p>
<p data-nodeid="5195">不过在 Node.js 中，目前仅 Lambda 的入口函数支持支持异步 async 写法，其他 FaaS 平台入口函数只能是同步写法，对于异步操作只能使用回调函数实现，所以入口函数有第三个参数 callback 。callback 就是你平时写 JavaScript 代码中的回调函数，第一个参数是错误信息，第二个参数是返回值。如果函数运行正常没有报错，则 callback 第一个参数传入null。当然，除了入口函数，其他代码支持异步，这由 Node.js 的版本决定。举个例子，下面的代码入口函数 handler 是回调函数，而 sayHello 函数都是 async 函数：</p>
<pre class="lang-javascript" data-nodeid="5196"><code data-language="javascript"><span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">sayHello</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">return</span> <span class="hljs-string">'hello world'</span>;
}
exports.handler = <span class="hljs-function">(<span class="hljs-params">event, context, callback</span>) =&gt;</span> {
  sayHello()
    .then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> callback(<span class="hljs-literal">null</span>, res))
    .catch(<span class="hljs-function"><span class="hljs-params">error</span> =&gt;</span> callback(error));
}
</code></pre>
<h4 data-nodeid="5197">触发器及事件对象</h4>
<p data-nodeid="5198">前面我提到，FaaS 接收到触发器产生的事件后，会根据触发器信息生成 event 对象，然后以 event 为参数调用函数。接下来我就带你了解常见的触发器，及其 event 对象属性。</p>
<ul data-nodeid="5199">
<li data-nodeid="5200">
<p data-nodeid="5201"><strong data-nodeid="5397">HTTP 触发器</strong></p>
</li>
</ul>
<p data-nodeid="5202">在众多 FaaS 平台中，函数计算直接提供了 HTTP 触发器，HTTP 触发器通过发送 HTTP 请求来触发函数执行，一般都会支持 POST、GET、PUT、HEAD 等方法。所以你可以用 HTTP 触发器来构建 Restful 接口或 Web 系统。</p>
<p data-nodeid="7630" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/50/CgqCHl_qmXCARXH9AAF-_DuU4lw021.png" alt="4.png" data-nodeid="7634"></p>
<div data-nodeid="7631"><p style="text-align:center"><span style="color:#b8b8b8">HTTP 触发器</span></p></div>


<p data-nodeid="5205">HTTP 触发器会根据 HTTP 请求和请求参数生成事件，然后以参数形式传递给函数。那么 HTTP 触发器的入口函数参数中的 request 和 response 参数具体有哪些属性呢？</p>
<p data-nodeid="5206">其实， request 和 response 参数本质上与 Express.js 框架的 request 和 response 类似，具体可以参考其文档。下面是 request 参数的示例：</p>
<pre class="lang-javascript" data-nodeid="5207"><code data-language="javascript"><span class="hljs-comment">// HTTP 触发器 request 参数</span>
{
    <span class="hljs-string">"method"</span>: <span class="hljs-string">"GET"</span>,
    <span class="hljs-string">"clientIP"</span>: <span class="hljs-string">"42.120.75.133"</span>,
    <span class="hljs-string">"url"</span>: <span class="hljs-string">"/2016-08-15/proxy/serverless/hello-world/"</span>,
    <span class="hljs-string">"path"</span>: <span class="hljs-string">"/"</span>,
    <span class="hljs-string">"queries"</span>: { <span class="hljs-string">"name"</span>: <span class="hljs-string">"World"</span> },
    <span class="hljs-string">"headers"</span>: {
    <span class="hljs-string">"accept"</span>: <span class="hljs-string">"*/*"</span>,
        <span class="hljs-string">"content-length"</span>: <span class="hljs-string">"0"</span>,
        <span class="hljs-string">"content-type"</span>: <span class="hljs-string">"application/x-www-form-urlencoded"</span>,
        <span class="hljs-string">"host"</span>: <span class="hljs-string">"1457216987974698.cn-hangzhou.fc.aliyuncs.com"</span>,
        <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"curl/7.64.1"</span>,
        <span class="hljs-string">"x-forwarded-proto"</span>: <span class="hljs-string">"https"</span>
  }
  <span class="hljs-comment">// ...</span>
}
</code></pre>
<p data-nodeid="5208">如果你用 POST 请求，直接从 request 中取 body 参数比较麻烦，所以你可以用 raw-body 这个包来处理请求参数，代码示例如下：</p>
<pre class="lang-javascript" data-nodeid="5209"><code data-language="javascript"><span class="hljs-keyword">var</span> getRawBody = <span class="hljs-built_in">require</span>(<span class="hljs-string">'raw-body'</span>)
exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">request, response, context</span>) </span>{
    <span class="hljs-comment">// 获取请求参数</span>
    getRawBody(request, <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">err, data</span>) </span>{
        <span class="hljs-keyword">var</span> params = {
            <span class="hljs-attr">path</span>: request.path,
            <span class="hljs-attr">queries</span>: request.queries, <span class="hljs-comment">// HTTP 请求 query 参数</span>
            <span class="hljs-attr">headers</span>: request.headers,
            <span class="hljs-attr">method</span>: request.method,
            <span class="hljs-attr">body</span>: data, <span class="hljs-comment">// data 就是 HTTP 请求 body 参数</span>
            <span class="hljs-attr">url</span>: request.url,
            <span class="hljs-attr">clientIP</span>: request.clientIP,
        }
        <span class="hljs-comment">// 设置 HTTP 返回值</span>
        <span class="hljs-keyword">var</span> respBody = <span class="hljs-keyword">new</span> Buffer.from(<span class="hljs-built_in">JSON</span>.stringify(params));
        response.setStatusCode(<span class="hljs-number">200</span>)
        response.setHeader(<span class="hljs-string">'content-type'</span>, <span class="hljs-string">'application/json'</span>)
        response.send(respBody)
    })
};
</code></pre>
<ul data-nodeid="5210">
<li data-nodeid="5211">
<p data-nodeid="5212"><strong data-nodeid="5408">API 网关触发器</strong></p>
</li>
</ul>
<p data-nodeid="5213">API 网关触发器与 HTTP 触发器类似，它主要用于构建 Web 系统。本质是利用 API 网关接收 HTTP 请求，然后再产生事件，将事件传递给 FaaS。FaaS 将函数执行完毕后将函数返回值传递给 API 网关，API 网关再将返回值包装为 HTTP 响应返回给用户。</p>
<p data-nodeid="8347" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/50/CgqCHl_qmXuAZBHIAAGGcHsdzWI696.png" alt="5.png" data-nodeid="8351"></p>
<div data-nodeid="8348"><p style="text-align:center"><span style="color:#b8b8b8">API 网关触发器</span></p></div>


<p data-nodeid="5216">下面是 Lambda 的 API 网关触发器 event 参数示例：</p>
<pre class="lang-javascript" data-nodeid="5217"><code data-language="javascript"><span class="hljs-comment">// Lambda API 网关触发器 event 参数示例</span>
{
    <span class="hljs-string">"version"</span>: <span class="hljs-string">"2.0"</span>,
    <span class="hljs-string">"routeKey"</span>: <span class="hljs-string">"ANY /hello-world"</span>,
    <span class="hljs-string">"rawPath"</span>: <span class="hljs-string">"/default/hello-world"</span>,
    <span class="hljs-string">"rawQueryString"</span>: <span class="hljs-string">"name=1"</span>,
    <span class="hljs-string">"headers"</span>: {
        <span class="hljs-string">"accept"</span>: <span class="hljs-string">"*/*"</span>,
        <span class="hljs-string">"content-length"</span>: <span class="hljs-string">"0"</span>,
        <span class="hljs-string">"host"</span>: <span class="hljs-string">"gwfk38f70e.execute-api.us-east-1.amazonaws.com"</span>,
        <span class="hljs-string">"user-agent"</span>: <span class="hljs-string">"curl/7.64.1"</span>,
        <span class="hljs-string">"x-amzn-trace-id"</span>: <span class="hljs-string">"Root=1-5fb47a82-5cf8a8f3573b039d538fdea2"</span>,
        <span class="hljs-string">"x-forwarded-for"</span>: <span class="hljs-string">"42.120.75.133"</span>,
        <span class="hljs-string">"x-forwarded-port"</span>: <span class="hljs-string">"443"</span>,
        <span class="hljs-string">"x-forwarded-proto"</span>: <span class="hljs-string">"https"</span>
    },
    <span class="hljs-string">"queryStringParameters"</span>: {
        <span class="hljs-string">"name"</span>: <span class="hljs-string">"1"</span>
    },
    <span class="hljs-string">"requestContext"</span>: {},
    <span class="hljs-string">"isBase64Encoded"</span>: <span class="hljs-literal">false</span>
}
</code></pre>
<p data-nodeid="5218">相比而言，HTTP 触发器用起来更简单， API 网关触发器功能更丰富，比如可以实现 IP 黑名单、流量控制等高级功能。所以，如果你只是实现简单的 Web 接口，可以使用 HTTP 触发器，如果你需要一些高级功能，可以用 API 网关触发器。</p>
<ul data-nodeid="5219">
<li data-nodeid="5220">
<p data-nodeid="5221"><strong data-nodeid="5418">定时触发器</strong></p>
</li>
</ul>
<p data-nodeid="5222">定时触发器就是定时执行函数，它经常用来做一些周期任务，比如每天定时查询天气并给自己发送通知、每小时定时处理分析日志等等。</p>
<p data-nodeid="9064" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/45/Ciqc1F_qmYWAE9fxAAD1A7kgncU172.png" alt="6.png" data-nodeid="9068"></p>
<div data-nodeid="9065"><p style="text-align:center"><span style="color:#b8b8b8">定时触发器</span></p></div>


<p data-nodeid="5225">定时触发器的<code data-backticks="1" data-nodeid="5424">event</code>对象很简单：</p>
<pre class="lang-json" data-nodeid="5226"><code data-language="json"><span class="hljs-comment">// 函数计算定时触发器 event 参数示例</span>
{
  <span class="hljs-attr">"triggerTime"</span>: <span class="hljs-string">"2020-11-22T17:42:20Z"</span>,
  <span class="hljs-attr">"triggerName"</span>: <span class="hljs-string">"timer"</span>,
  <span class="hljs-attr">"payload"</span>: <span class="hljs-string">""</span>
}
</code></pre>
<p data-nodeid="5227">除了这三种触发器，各个云厂商也都实现了与自己云产品相关的触发器，比如，文件存储触发器、消息触发器、数据库触发器。</p>
<p data-nodeid="5228">总的来说，触发器决定了一个 Serverless 应用如何提供服务，也决定了代码应该如何编写。丰富的触发器，可以让应用功能更强大，但不同触发器的 event 属性不同，也为编程带来了麻烦。<strong data-nodeid="5432">记得我的建议吗？将业务逻辑拆分到入口函数之外。</strong> 保持入口函数的简洁，这样业务代码就与触发器无关了。</p>
<h4 data-nodeid="5229">日志输出</h4>
<p data-nodeid="5230">无论你用什么编写语言开发 Serverless 应用，你都要在合适的时候输出合适的日志信息，方便调试应用、排查问题。在 Serverless 中，日志输出和传统应用的日志输出没有太大区别，<strong data-nodeid="5438">只是日志的存储和查询方式变了。</strong></p>
<p data-nodeid="5231">以函数计算为例，如果你在控制台创建函数，则函数计算默认会使用日志服务来为你存储日志。在 “日志查询” 标签下可以查看函数调用日志。日志服务是一个日志采集、分析产品，所以如果你要实现业务监控，则可以将日志输出到日志服务，然后在日志服务中对日志进行分析，并设置报警项。</p>
<p data-nodeid="5232"><img src="https://s0.lgstatic.com/i/image/M00/8B/F2/Ciqc1F_jBdKAQddrAANmkQx2nJs904.png" alt="Drawing 8.png" data-nodeid="5442"></p>
<div data-nodeid="5233"><p style="text-align:center"><span style="color:#b8b8b8">函数调用日志</span></p></div>
<p data-nodeid="5234">基本上，各个云厂商的 FaaS 会选择自己的日志服务来存储函数日志， FaaS 平台也提供了基本的函数监控，包括函数的运行时长、内存使用等。国外也有很多第三方的 SaaS 产品帮你实现 Serverless 应用的日志存储分析、系统监控报警，比如 <a href="https://dashbird.io/" data-nodeid="5446">dashbird</a>、<a href="https://www.thundra.io/" data-nodeid="5450">thundra</a>。<strong data-nodeid="5455">国内这方面的产品非常少，我觉得这对你我来说是一个机会。</strong></p>
<h4 data-nodeid="5235">异常处理</h4>
<p data-nodeid="5236">函数在运行过程中，会出现异常情况。当函数执行异常或主动抛出异常时，FaaS 平台会捕捉到异常信息，记录异常日志，并将异常信息以 JSON 对象返回。下面是一个 Node.js 代码抛出异常的示例，在这个例子中，我使用 throw new Error() 主动抛出一个异常的日志：</p>
<p data-nodeid="5237"><img src="https://s0.lgstatic.com/i/image/M00/8B/FD/CgqCHl_jBd6AdHX_AAKNGxRYIKQ705.png" alt="Drawing 9.png" data-nodeid="5460"></p>
<p data-nodeid="5238">其中 Response 就是函数返回值，Function Logs 就是调用日志。</p>
<p data-nodeid="5239">在传统应用中，一个函数的异常可能会让整个应用崩溃，但在 Serverless 应用中，一个函数异常，只会影响这一次函数的执行。这也是为什么 Serverless 能够提升应用的整体稳定性。<strong data-nodeid="5466">但我还是建议你编写代码时，充分考虑程序的异常，保证代码的健壮性，进一步提升系统稳定性。</strong></p>
<h3 data-nodeid="5240">总结</h3>
<p data-nodeid="5241">这一讲我从宏观上带你学习了如何开发一个 Serverless 应用，以及开发过程中涉及的基础知识。相信通过今天的学习，你可以体会到 Serverless 应用开发与传统开发的区别，比如代码编辑可以在云端进行，而不仅是在本地进行，应用的组成是单个独立的函数，而不是所有功能的集合体。</p>
<p data-nodeid="5242">此外我觉得，对一个 Serverless FaaS 平台来说，除了要具备基本的函数执行能力外，还要提供便利的开发工具、丰富的触发器、完善的日志监控以及与其他服务集成等各方面能力。你在进行技术选型时，也需要考虑这些方面。这一讲我强调这样几个重点：</p>
<p data-nodeid="9781" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8C/50/CgqCHl_qmZCAdIFmAAFS6TtGLlQ407.png" alt="7.png" data-nodeid="9784"></p>

<ul data-nodeid="5244">
<li data-nodeid="5245">
<p data-nodeid="5246">Serverless 的应用基本组成单位是函数，函数之间互相独立，因此 Serverless 能提高应用稳定性；</p>
</li>
<li data-nodeid="5247">
<p data-nodeid="5248">函数定义与触发器和编程语言相关，不同 FaaS 平台的实现不尽相同；</p>
</li>
<li data-nodeid="5249">
<p data-nodeid="5250">为了使代码扩展性更强，建议你将业务逻辑拆分到入口函数之外；</p>
</li>
<li data-nodeid="5251">
<p data-nodeid="5252">为了使应用稳定性更好，建议你编写函数代码时充分考虑程序异常。</p>
</li>
</ul>
<p data-nodeid="5253">在实际工作中，我经常用 Serverless 来处理业务逻辑，比如快速开发一个测试接口、实时处理日志等，如果你有类似需求也可以尝试使用 Serverless 来实现。</p>
<p data-nodeid="5254" class="">本节课的作业相对来讲比较简单，那就是动手实现你的第一个 Serverless 应用，希望你能夯实基础，游刃有余地学习接下来的内容，我们下一讲见。</p>

---

### 精选评论

##### **俊：
> 对于企业级的项目怎么更好使用serverless的？接口数据庞大

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这时就需要一些开发工具、开发框架来辅助开发了，需要使用工具来帮助管理、调试部署应用。你可以使用一些开源的或云厂商提供的工具，比如 Serverless Framework、fun 等。另外如果企业有实力，也可以开发自己的 Serverless 工具链。关于 Serverless 工具链和开发框架，我会在 05 专门提到。

##### **艺：
> 入口函数不同语言的示例 异步和同步注释是不是写反了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 您好，没有写反哈。这里的同步异步指的是编程方式。基于回调的是异步函数，我们需要通过 callback 返回函数的执行结果。基于 async，我们可以像同步编程一样，来编写异步代码。

##### liminjun88@qq.com：
> API 网关触发器，参数透传还是不透传，可以进一步讲解一下。国内的云函数也基本上支持API网关触发器了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 您好，感谢您的建议，关于参数透传，我在这里补充一下。如果设置参数透传，则客户端请求的参数会直接转发到云函数；如果不透传，则需要在 API 网关中配置参数，如果客户端传递了未在配置中的参数，参数就会被过滤，不会转发到云函数。我建议尽量不透传参数。参数透传的优点是简单、方便，但不利于维护。而不透传参数，虽需要提前对 API 的参数进行定义，但能提升可维护性，API 网关上的参数定义就是接口文档；同时也能提升 API 安全性，参数都是可预期的。目前大部分 Serverless 开发框架也都提供了根据配置自动生成 API 及参数定义的功能，很多 API 网关产品也提供了导入 Swagger 文档等功能，帮助我们更简单地配置 API 参数。

##### **3191：
> 我还以为老师会教我从零开始学习 原来是进阶篇😭

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 您好，有些内容确实需要有一点基础才更容易学会。如果您在学习过程中遇到任何问题，都可以留言哦，我看到后都会及时回复，尽可能解答您的疑惑。

