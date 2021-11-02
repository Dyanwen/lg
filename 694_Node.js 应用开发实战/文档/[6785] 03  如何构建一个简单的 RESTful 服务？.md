<p data-nodeid="22200" class="">前面几讲都是一些知识点的阐述，本讲将应用前面讲到的知识点，来实现一个简单版本的 RESTful 系统架构，并在此架构上实现一些简单的应用。</p>
<h3 data-nodeid="22201">基础技术点</h3>
<p data-nodeid="22202">在学习本讲时会涉及一些技术知识点：</p>
<ul data-nodeid="22203">
<li data-nodeid="22204">
<p data-nodeid="22205">什么是 RESTful 规范；</p>
</li>
<li data-nodeid="22206">
<p data-nodeid="22207">数据库的读写处理过程；</p>
</li>
<li data-nodeid="22208">
<p data-nodeid="22209">目前常用的 MVC 架构模式，以及后续本专栏所应用的一套新的、独创的架构模式——MSVC 架构模式。</p>
</li>
</ul>
<h4 data-nodeid="22210">RESTful</h4>
<p data-nodeid="22211">RESTful（Representational State Transfer）是一种架构的<strong data-nodeid="22416">约束条件</strong>和<strong data-nodeid="22417">规则</strong>。在倡导前后端分离后，该架构规范的应用愈加广泛。具体知识点，<a href="https://github.com/aisuhua/restful-api-design-references" data-nodeid="22414">你可以参考这里进行学习</a>。</p>
<h4 data-nodeid="22212">MongoDB</h4>
<p data-nodeid="22213">由于本讲涉及数据库的操作，本专栏主要使用<strong data-nodeid="22436">非关系型数据库</strong>——MongoDB，因此这里需要先了解下 MongoDB 的相关操作，以及安装配置方法，你可以参考<a href="https://www.runoob.com/mongodb/mongodb-osx-install.html" data-nodeid="22426">官网的文档来安装</a>，这里就不细讲。为了使用便利，我们可以直接在官网创建 MongoDB 云服务远程连接，具体请参照<a href="https://cloud.mongodb.com/" data-nodeid="22430">官网</a>，以及 <a href="https://mongodb.github.io/node-mongodb-native/2.0/api/index.html" data-nodeid="22434">API 文档请参考这里</a>。</p>
<h4 data-nodeid="22214">MVC→MSVC</h4>
<p data-nodeid="22215">我们应该都比较熟知 MVC 架构，它在前后端分离中起到了非常重要的作用，我们先来看下传统的 MVC 架构的模式，如图 1 所示。</p>
<p data-nodeid="22216"><img src="https://s0.lgstatic.com/i/image6/M00/17/08/CioPOWBHMl-ASR4aAAAg3opNISU640.png" alt="图片 2.png" data-nodeid="22441"></p>
<p data-nodeid="22217">此模式中：</p>
<ul data-nodeid="22218">
<li data-nodeid="22219">
<p data-nodeid="22220">M（Model）层处理数据库相关的操作（只有数据库操作时）；</p>
</li>
<li data-nodeid="22221">
<p data-nodeid="22222">C（Controller）层处理业务逻辑；</p>
</li>
<li data-nodeid="22223">
<p data-nodeid="22224">V（View）层则是页面显示和交互（本讲不涉及）。</p>
</li>
</ul>
<p data-nodeid="22225">但是在目前服务划分较细的情况下，M 层不仅仅是数据库操作，因此这种架构模式显得有些力不从心，导致开发的数据以及业务逻辑有时候在 M 层，有时候却在 C 层。出现这类情况的核心原因是 C 与 C 之间无法进行复用，如果需要复用则需要放到 M 层，那么业务逻辑就会冗余在 M，代码会显得非常繁杂，如图 2 所示。</p>
<p data-nodeid="22226"><img src="https://s0.lgstatic.com/i/image6/M00/17/0C/Cgp9HWBHMnaAG42WAAA3-AZ5WeM867.png" alt="图片 4.png" data-nodeid="22449"></p>
<div data-nodeid="22227"><p style="text-align:center">图 2 MVC 模式问题</p></div>
<p data-nodeid="22228">为了解决以上问题，在经过一些实践后，我在研发过程中提出了一套新的架构模式，当然也有他人提到过（比如 Eggjs 框架中的模式）。这种模式也会应用在本专栏的整个架构体系中，我们暂且叫作 MSVC（Model、Service、View、Controller）。</p>
<p data-nodeid="22229">我们先来看下 MSVC 的架构模式，如图 3 所示。</p>
<p data-nodeid="22230"><img src="https://s0.lgstatic.com/i/image6/M00/17/09/CioPOWBHMomAfpfmAABDDmUKgC4829.png" alt="图片 6.png" data-nodeid="22454"></p>
<p data-nodeid="22231">将所有数据相关的操作都集中于 M 层，而 M 层复用的业务逻辑则转到新的 S 层，C 层则负责核心业务处理，可以调用 M 和 S 层。以上是相关知识点，接下来我们进行架构的实践设计。</p>
<h3 data-nodeid="22232">系统实践</h3>
<p data-nodeid="22233">我们先实现一个简单版本的 RESTful 服务，其次为了能够更清晰地了解 MVC 架构和 MSVC 架构的优缺点，我们也会分别实现两个版本的 RESTful 服务。</p>
<p data-nodeid="22234">我们要实现的是一个<strong data-nodeid="22463">获取用户发帖的列表信息 API</strong>，该 API 列表的内容包含两部分，一部分是从数据库获取的发帖内容，但是这部分只包含用户 ID，另外一部分则是需要通过 ID 批量拉取用户信息。</p>
<p data-nodeid="22235">我们先来设计 RESTful API，由于是拉取列表内容接口，因此这里设计为一个 GET 接口，根据 RESTful 约束规则设计为：GET /v1/contents；另外还需要设计一个独立的服务用来获取用户信息，将接口设计为：GET /v1/userinfos。</p>
<p data-nodeid="22236">为了更清晰些，我绘制了一个时序图来表示，如图 4 所示。</p>
<p data-nodeid="22237"><img src="https://s0.lgstatic.com/i/image6/M00/17/0C/Cgp9HWBHMpOASLZ7AACJ0Un2XOA103.png" alt="图片 7.png" data-nodeid="22468"></p>
<div data-nodeid="22238"><p style="text-align:center">图 4 例子系统时序图</p></div>
<p data-nodeid="22239">在图 4 中详细的过程是：</p>
<ul data-nodeid="22240">
<li data-nodeid="22241">
<p data-nodeid="22242">用户先调用 /v1/contents API 拉取 restful server 的内容；</p>
</li>
<li data-nodeid="22243">
<p data-nodeid="22244">restful server 会首先去 MongoDB 中获取 contents；</p>
</li>
<li data-nodeid="22245">
<p data-nodeid="22246">拿到 contents 后解析出其中的 userIds；</p>
</li>
<li data-nodeid="22247">
<p data-nodeid="22248">然后再通过 /v1/userinfos API 调用 API server 的服务获取用户信息列表；</p>
</li>
<li data-nodeid="22249">
<p data-nodeid="22250">API server 同样需要和 MongoDB 交互查询到所需要的 userinfos；</p>
</li>
<li data-nodeid="22251">
<p data-nodeid="22252">拿到 userinfos 后通过 addUserinfo 将用户信息整合到 contents 中去；</p>
</li>
<li data-nodeid="22253">
<p data-nodeid="22254">最后将 contents 返回给到调用方。</p>
</li>
</ul>
<p data-nodeid="22255">在不考虑任何架构模式的情况下，我们来实现一个简单版本的 restful 服务，上面分析了需要实现 2 个 server，这里分别叫作 <strong data-nodeid="22486">API server</strong> 和 <strong data-nodeid="22487">restful server</strong>。</p>
<h4 data-nodeid="22256">API server</h4>
<p data-nodeid="22257">server 包含 2 个部分：<strong data-nodeid="22498">解析请求路径</strong>和<strong data-nodeid="22499">解析请求参数</strong>，在 Node.js 中我们可以用以下代码来解析：</p>
<pre class="lang-javascript" data-nodeid="22258"><code data-language="javascript"><span class="hljs-comment">/**
 * 
 * 创建 http 服务，简单返回
 */</span>
<span class="hljs-keyword">const</span> server = http.createServer(<span class="hljs-keyword">async</span> (req, res) =&gt; {
    <span class="hljs-comment">// 获取 get 参数</span>
    <span class="hljs-keyword">const</span> pathname = url.parse(req.url).pathname;
    paramStr = url.parse(req.url).query,
    param = querystring.parse(paramStr);
    <span class="hljs-comment">// 过滤非拉取用户信息请求</span>
    <span class="hljs-keyword">if</span>(<span class="hljs-string">'/v1/userinfos'</span> != pathname) {
      <span class="hljs-keyword">return</span> setResInfo(res, <span class="hljs-literal">false</span>, <span class="hljs-string">'path not found'</span>);
    }
    <span class="hljs-comment">// 参数校验，没有包含参数时返回错误</span>
    <span class="hljs-keyword">if</span>(!param || !param[<span class="hljs-string">'user_ids'</span>]) {
      <span class="hljs-keyword">return</span> setResInfo(res, <span class="hljs-literal">false</span>, <span class="hljs-string">'params error'</span>);
    }
});
</code></pre>
<p data-nodeid="22259">上面代码中使用 Node.js 的 url 模块来获取请求路径和 GET 字符串，拿到 GET 的字符串后还需要使用 Node.js 的 querystring 将字符串解析为参数的 JSON 对象。</p>
<p data-nodeid="22260">参数和请求路径解析成功后，再进行路径的判断和校验，如果不满足我们当前的要求，调用 setResInfo 报错返回相应的数据给到前端。setResInfo 这个函数实现比较简单，使用 res 对象来设置返回的数据，具体你可以前往 <a href="https://github.com/love-flutter/nodejs-column" data-nodeid="22504">GitHub 源码</a>中查看。</p>
<p data-nodeid="22261">路径和参数解析成功后，我们再根据当前参数查询 MongoDB 中的 userinfo 数据，具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="22262"><code data-language="javascript"><span class="hljs-keyword">const</span> baseMongo = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./lib/baseMongodb'</span>)();
<span class="hljs-keyword">const</span> server = http.createServer(<span class="hljs-keyword">async</span> (req, res) =&gt; {
    <span class="hljs-comment">// ...省略上面部分代码</span>
    <span class="hljs-comment">// 从 db 查询数据，并获取，有可能返回空数据</span>
    <span class="hljs-keyword">const</span> userInfo = <span class="hljs-keyword">await</span> queryData({<span class="hljs-string">'id'</span> : { <span class="hljs-attr">$in</span> : param[<span class="hljs-string">'user_ids'</span>].split(<span class="hljs-string">','</span>)}});
    <span class="hljs-keyword">return</span> setResInfo(res, <span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, userInfo);
});
<span class="hljs-comment">/**
 * 
 * @description db 数据查询
 * @param object queryOption 
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">queryData</span>(<span class="hljs-params">queryOption</span>) </span>{
  <span class="hljs-keyword">const</span> client = <span class="hljs-keyword">await</span> baseMongo.getClient();
  <span class="hljs-keyword">const</span> collection = client.db(<span class="hljs-string">"nodejs_cloumn"</span>).collection(<span class="hljs-string">"user"</span>);
  <span class="hljs-keyword">const</span> queryArr = <span class="hljs-keyword">await</span> collection.find(queryOption).toArray();
  <span class="hljs-keyword">return</span> queryArr;
}
</code></pre>
<p data-nodeid="22263">这一代码中使用了 baseMongodb 这个自己封装的库，该库主要基于 mongo 的基础库进行了本地封装处理。在 queryData 中通过 mongo 来查询 nodejs_cloumn 库中的 user 表，并带上查询条件，查询语法你可以参考 <a href="https://mongodb.github.io/node-mongodb-native/2.0/api/index.html" data-nodeid="22512">API 文档</a>。</p>
<blockquote data-nodeid="22264">
<p data-nodeid="22265">注意上面代码中，find 查询返回的数据需要使用 toArray 进行转化处理。拿到 MongoDB 查询结果后，再调用 setResInfo 返回查询结果给到前端。</p>
</blockquote>
<p data-nodeid="22266">接下来我们继续实现 restful server。</p>
<h4 data-nodeid="22267">restful server</h4>
<p data-nodeid="22268">和 API server 相似，前面 2 个过程是解析请求路径和请求参数，解析成功后，根据时序图先从 MongoDB 中拉取 10 条 content 数据，代码如下：</p>
<pre class="lang-javascript" data-nodeid="22269"><code data-language="javascript"><span class="hljs-keyword">const</span> server = http.createServer(<span class="hljs-keyword">async</span> (req, res) =&gt; {
    <span class="hljs-comment">// 获取 get 参数</span>
    <span class="hljs-keyword">const</span> pathname = url.parse(req.url).pathname;
    paramStr = url.parse(req.url).query,
    param = querystring.parse(paramStr);
    <span class="hljs-comment">// 过滤非拉取用户信息请求</span>
    <span class="hljs-keyword">if</span>(<span class="hljs-string">'/v1/contents'</span> != pathname) {
      <span class="hljs-keyword">return</span> setResInfo(res, <span class="hljs-literal">false</span>, <span class="hljs-string">'path not found'</span>, <span class="hljs-literal">null</span>, <span class="hljs-string">'404'</span>);
    }
    <span class="hljs-comment">// 从 db 查询数据，并获取，有可能返回空数据</span>
    <span class="hljs-keyword">let</span> contents = <span class="hljs-keyword">await</span> queryData({}, {<span class="hljs-attr">limit</span>: <span class="hljs-number">10</span>});

    contents = <span class="hljs-keyword">await</span> filterUserinfo(contents);
    <span class="hljs-keyword">return</span> setResInfo(res, <span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, contents);
});
</code></pre>
<p data-nodeid="22270">在 MongoDB 中查询到具体的 contents 后，再调用 filterUserinfo 这个函数将 contents 中的 user_id 转化为 userinfo，具体代码如图 5 所示（为了代码简洁，我使用了截图，源代码请参考 <a href="https://github.com/love-flutter/nodejs-column" data-nodeid="22523">GitHub</a> 上的）：</p>
<p data-nodeid="22271"><img src="https://s0.lgstatic.com/i/image6/M01/17/09/CioPOWBHMqSAeQfTAAF5ba4WuFA603.png" alt="图片 8.png" data-nodeid="22527"></p>
<div data-nodeid="22272"><p style="text-align:center">图 5 filterUserinfo 代码实现</p></div>
<p data-nodeid="22273">在上面代码中的第 52 行是调用 API server 将用户的 userIds 转化为 userinfos，最后在 64 行，将获取的 userinfos 添加到 contents 中。</p>
<p data-nodeid="22274">最后我们打开两个命令行窗口，分别进入到两个 server 下，运行如下命令启动服务。</p>
<pre class="lang-java" data-nodeid="22275"><code data-language="java">node index
</code></pre>
<p data-nodeid="22276">运行成功后，我们在浏览器中打开如下地址：</p>
<pre class="lang-java" data-nodeid="22277"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:5000/v1/userinfos?user_ids=1001,1002</span>
</code></pre>
<p data-nodeid="22278">你将会看到一个 JSON 的返回结构，如图 6 所示。</p>
<p data-nodeid="22279"><img src="https://s0.lgstatic.com/i/image6/M00/17/0D/Cgp9HWBHMqyAFEfxAACiD747m4U810.png" alt="图片 9.png" data-nodeid="22534"></p>
<div data-nodeid="22280"><p style="text-align:center">图 6 API server 返回信息</p></div>
<p data-nodeid="22281">接下来我们访问如下地址，并且打开 chrome 的控制台的 network 状态栏。</p>
<pre class="lang-java" data-nodeid="22282"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:5000/v1/test</span>
</code></pre>
<p data-nodeid="22283">你将会看到返回的状态码是 404，如图 7 所示，这也是 restful 的规范之一，即正确地使用 http 状态码。</p>
<p data-nodeid="22284"><img src="https://s0.lgstatic.com/i/image6/M01/17/0D/Cgp9HWBHMrWAHu5_AAFIdt9MME4795.png" alt="图片 10.png" data-nodeid="22539"></p>
<div data-nodeid="22285"><p style="text-align:center">图 7 异常响应返回</p></div>
<p data-nodeid="22286">接下来我们请求 restful server 的 API，同样使用浏览器打开如下接口地址：</p>
<pre class="lang-java" data-nodeid="22287"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:4000/v1/contents</span>
</code></pre>
<p data-nodeid="22288">你将会看到如图 8 所示的响应结果。</p>
<p data-nodeid="22289"><img src="https://s0.lgstatic.com/i/image6/M01/17/0D/Cgp9HWBHMr-AKdSMAAFW1ay8vPI075.png" alt="图片 11.png" data-nodeid="22544"></p>
<div data-nodeid="22290"><p style="text-align:center">图 8 contents 响应结果</p></div>
<p data-nodeid="22291">以上就实现了一个简单 restful 服务的功能，你可以看到代码都堆积在 index.js 中，并且代码逻辑还比较简单，如果稍微复杂一些，这种架构模式根本没法进行团队合作，或者后期维护，因此就需要 MVC 和 MVCS 架构模式来优化这种场景。</p>
<p data-nodeid="22292">接下来我们先来看看使用 MVC 来优化。</p>
<h3 data-nodeid="22293">进阶实现</h3>
<p data-nodeid="22294">没有架构模式虽然也能按照需求满足接口要求，但是代码是<strong data-nodeid="22553">不可维护</strong>的。而 MVC 已经被实践证明是非常好的架构模式，但是在现阶段也存在一些问题，接下来我们就逐步进行优化，让我们的架构和代码更加优秀。</p>
<h4 data-nodeid="22295">MVC</h4>
<p data-nodeid="22296">既然是 M 和 C，我们就先思考下，上面的 restful server 中哪些是 M 层的逻辑，哪些是 C 层的逻辑。</p>
<p data-nodeid="22297"><img src="https://s0.lgstatic.com/i/image6/M01/17/0D/Cgp9HWBHMsyAF-PaAAB-xAx-32s648.png" alt="图片 12.png" data-nodeid="22558"></p>
<p data-nodeid="22298">以上是所有的逻辑，根据表格，我们首先创建两个目录分别是 <strong data-nodeid="22568">model</strong> 和 <strong data-nodeid="22569">Controller</strong>：</p>
<ul data-nodeid="22299">
<li data-nodeid="22300">
<p data-nodeid="22301">在 model 中创建一个 content.js 用来处理 content model 逻辑；</p>
</li>
<li data-nodeid="22302">
<p data-nodeid="22303">在 Controller 中也创建一个 content.js 用来处理 content 的 Controller 逻辑。</p>
</li>
</ul>
<p data-nodeid="22304">在源代码中有一个 index.js 文件，在没有架构模式时，基本上处理了所有的业务，但是根据当前架构模式，如表 1 所示，只适合处理 url 路径解析、路由判断及转发，因此需要简化原来的逻辑，和第一部分代码一样，我们就不再列举了，主要看路由判断。首先需要根据 restful url 路由配置一份路由转发逻辑，配置如下：</p>
<pre class="lang-javascript" data-nodeid="22305"><code data-language="javascript"><span class="hljs-keyword">const</span> routerMapping = {
    <span class="hljs-string">'/v1/contents'</span> : {
        <span class="hljs-string">'Controller'</span> : <span class="hljs-string">'content'</span>,
        <span class="hljs-string">'method'</span> : <span class="hljs-string">'list'</span>
    },
    <span class="hljs-string">'/v1/test'</span> : {
        <span class="hljs-string">'Controller'</span> : <span class="hljs-string">'content'</span>,
        <span class="hljs-string">'method'</span> : <span class="hljs-string">'test'</span>
    }
};
</code></pre>
<p data-nodeid="22306">上面代码的意思是：</p>
<ul data-nodeid="22307">
<li data-nodeid="22308">
<p data-nodeid="22309">如果请求路径是 /v1/contents 就转发到 content.js 这个 Controller，并且调用其 list 方法；</p>
</li>
<li data-nodeid="22310">
<p data-nodeid="22311">如果是 /v1/test 则也转发到 content.js 这个 Controller，但调用的是 test 方法。</p>
</li>
</ul>
<blockquote data-nodeid="22312">
<p data-nodeid="22313">注意：其中 test 是一个同步方法，list 是一个异步方法。</p>
</blockquote>
<p data-nodeid="22314">路由配置完成以后，就需要根据路由配置，将请求路径、转发到处理相应功能的模块或者类、函数中去，代码如图 9 所示。</p>
<p data-nodeid="22315"><img src="https://s0.lgstatic.com/i/image6/M01/17/0D/Cgp9HWBHMtiAAs7YAAJNxih_ssE949.png" alt="图片 13.png" data-nodeid="22580"></p>
<div data-nodeid="22316"><p style="text-align:center">图 9 index 核心逻辑</p></div>
<ul data-nodeid="22317">
<li data-nodeid="22318">
<p data-nodeid="22319">第一个红色框内的部分，判断的是路由是否在配置内，不存在则返回 404；</p>
</li>
<li data-nodeid="22320">
<p data-nodeid="22321">第二个红色框内的部分，加载对应的 Controller 模块；</p>
</li>
<li data-nodeid="22322">
<p data-nodeid="22323">第三个红色框内的部分，表示判断所调用的方法类型是异步还是同步，如果是异步使用 await 来获取执行结果，如果是同步则直接调用获取返回结果。</p>
</li>
</ul>
<blockquote data-nodeid="22324">
<p data-nodeid="22325">注意：这里使用 try catch 的目的是确保调用安全，避免 crash 问题。</p>
</blockquote>
<p data-nodeid="22326">接下来我们实现一个 Controller，为了合理性，我们先实现一个基类，然后让每个 Controller 继承这个基类：</p>
<ul data-nodeid="22327">
<li data-nodeid="22328">
<p data-nodeid="22329">在项目根目录下我们创建一个 core 文件夹，并创建一个 Controller.js 作为基类；</p>
</li>
<li data-nodeid="22330">
<p data-nodeid="22331">然后我们把一些相同的功能放入这个基类，比如 res 和 req 的赋值，以及通用返回处理，还有 url 参数解析等。</p>
</li>
</ul>
<p data-nodeid="22332">我们来看下这部分代码，如图 10 所示。</p>
<p data-nodeid="22333"><img src="https://s0.lgstatic.com/i/image6/M01/17/0B/CioPOWBHMxSAAxc8AAF_MvEbU10031.png" alt="图片 14.png" data-nodeid="22591"></p>
<div data-nodeid="22334"><p style="text-align:center">图 10 Controller 基类</p></div>
<p data-nodeid="22335">功能还是比较简单的，只是提炼了一些 Controller 共同的部分。接下来我们再来实现 content.js 这个 Controller，代码如图 11 所示：</p>
<p data-nodeid="22336"><img src="https://s0.lgstatic.com/i/image6/M01/17/0E/Cgp9HWBHMwaAMQK8AAD-onTOdVQ611.png" alt="图片 15.png" data-nodeid="22595"></p>
<div data-nodeid="22337"><p style="text-align:center">图 11 content.js Controller</p></div>
<p data-nodeid="22338">我们在初次实现时，可以不关注图 11 中的第 2 和 3 行，实现红色框内的代码即可。可以将 list 暂时设置为空，实现完成后，我们在根目录运行以下命令，启动服务。</p>
<pre class="lang-java" data-nodeid="22339"><code data-language="java">node index
</code></pre>
<p data-nodeid="22340">接下来打开浏览器访问：</p>
<pre class="lang-java" data-nodeid="22341"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:3000/v1/test</span>
</code></pre>
<p data-nodeid="22342">你就可以看到响应了一个 JSON 数据，这样就实现了 Controller 部分了。如下代码所示：</p>
<pre class="lang-json" data-nodeid="22343"><code data-language="json">{
  ret:&nbsp;0,
  message:&nbsp;"good",
  data: { }
}
</code></pre>
<p data-nodeid="22344">接下来我们再来实现 Model 层部分，和 Controller 类似，我们也需要一个基类来处理 Model 层相似的逻辑，然后其他 Model 来继承这个基类，这部分如图 12 所示。</p>
<p data-nodeid="22345"><img src="https://s0.lgstatic.com/i/image6/M00/17/0B/CioPOWBHMx2AHsKzAAEYhjLBhO4974.png" alt="图片 16.png" data-nodeid="22602"></p>
<div data-nodeid="22346"><p style="text-align:center">图 12 Model 基类</p></div>
<p data-nodeid="22347">这个基类首先设置了 db 名称，其次定义了一个 GET 方法来获取表的操作句柄，这部分代码与上面简单 restful 服务的类似。完成基类后，我们再来完善 model 中的 content.js 逻辑。</p>
<p data-nodeid="22348"><img src="https://s0.lgstatic.com/i/image6/M01/17/0E/Cgp9HWBHMyeAPElfAAEk45BmKsI006.png" alt="图片 17.png" data-nodeid="22606"></p>
<div data-nodeid="22349"><p style="text-align:center">图 13 model content.js 代码实现</p></div>
<p data-nodeid="22684" class="te-preview-highlight">这部分代码主要方法是 <strong data-nodeid="22690">getList</strong>，原理和简单 restful server 中的查询类似，在第 11 行通过父类的 GET 方法获取表 content 的操作句柄，再调用 MongoDB 的 find 方法查询 contents。有了 model content 后，我们再回去完善 content.js Controller 中的 list 函数部分逻辑，代码封装的比较简洁，如下所示：</p>

<pre class="lang-javascript" data-nodeid="22351"><code data-language="javascript">    <span class="hljs-keyword">async</span> list() {
        <span class="hljs-keyword">let</span> contentList = <span class="hljs-keyword">await</span> <span class="hljs-keyword">new</span> ContentModel().getList();
        contentList = <span class="hljs-keyword">await</span> <span class="hljs-keyword">this</span>._filterUserinfo(contentList);

        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resAPI(<span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, contentList);
    }
</code></pre>
<p data-nodeid="22352">上面代码中的第 4 行，只能在当前 Controller 下实现一个私有方法 _filterUserinfo 来处理用户信息部分，这部分逻辑也和简单 restful 服务的一样。</p>
<p data-nodeid="22353">这样就实现了一个 MVC 的架构，将原来的复杂不可扩展性的代码，转化为<strong data-nodeid="22629">可扩展</strong>、<strong data-nodeid="22630">易维护</strong>的代码，这部分核心代码可以参考 <a href="https://github.com/love-flutter/nodejs-column" data-nodeid="22627">GitHub 源码</a>。</p>
<h4 data-nodeid="22354">MVCS</h4>
<p data-nodeid="22355">在上面的代码中存在一个问题，就是 _filterUserinfo 是放在 Controller 来处理，这个方法又会涉及调用 API server 的逻辑，看起来也是数据处理部分，从原理上说这部分不适合放在 Controller。其次在其他 Controller 也需要 _filterUserinfo 时，这时候就比较懵逼了，比如我们现在有另外一个 Controller 叫作 recommend.js，这里面也是拉取推荐的 content，也需要这个 _filterUserinfo 方法，如图 14 所示。</p>
<p data-nodeid="22356"><img src="https://s0.lgstatic.com/i/image6/M00/17/0B/CioPOWBHMzCANv2nAAFcFfow9m4167.png" alt="图片 18.png" data-nodeid="22641"></p>
<div data-nodeid="22357"><p style="text-align:center">图 14 MVC 复用性问题例子</p></div>
<p data-nodeid="22358">其中左边是存在的矛盾，因为 _filterUserinfo 在 Controller 是私有方法，recommend Controller 调用不到，那么为了复用，我们只能将该方法封装到 content-model 中，并且将数据也集中在 Model 层去。</p>
<p data-nodeid="22359">虽然解决了问题，但是你会发现：</p>
<ul data-nodeid="22360">
<li data-nodeid="22361">
<p data-nodeid="22362">Model 层不干净了，它现在既要负责数据处理，又要负责业务逻辑；</p>
</li>
<li data-nodeid="22363">
<p data-nodeid="22364">Controller 层的业务减少了，但是分层不明确了，有些业务放在 Model，有些又在 Controller 层，对于后期代码的维护或者扩展都非常困难了。</p>
</li>
</ul>
<p data-nodeid="22365">为了解决这个问题，有一个新的概念——Service 层，具体如图 15 所示。</p>
<p data-nodeid="22366"><img src="https://s0.lgstatic.com/i/image6/M00/17/0E/Cgp9HWBHMzqAc1JJAAFspSJGcu8417.png" alt="图片 19.png" data-nodeid="22651"></p>
<div data-nodeid="22367"><p style="text-align:center">图 15 MSVC 优化效果</p></div>
<ul data-nodeid="22368">
<li data-nodeid="22369">
<p data-nodeid="22370">图中的浅红色框内，就是新架构模式的 M 层；</p>
</li>
<li data-nodeid="22371">
<p data-nodeid="22372">两个绿色框内为 C 层；</p>
</li>
<li data-nodeid="22373">
<p data-nodeid="22374">最上面的浅蓝色框则为 Service 层。</p>
</li>
</ul>
<p data-nodeid="22375">这样就可以复用 _filterUserinfo，并解决 M 与 C 层不明确的问题。接下来我们来实践这部分代码：</p>
<ul data-nodeid="22376">
<li data-nodeid="22377">
<p data-nodeid="22378">首先我们需要创建一个文件夹 service 来存放相应的 Service 层代码；</p>
</li>
<li data-nodeid="22379">
<p data-nodeid="22380">然后创建一个 content.js 来表示 content-service 这个模块；</p>
</li>
<li data-nodeid="22381">
<p data-nodeid="22382">再将原来代码中的 _filterUserinfo 逻辑转到 content-service 中去；</p>
</li>
<li data-nodeid="22383">
<p data-nodeid="22384">最后修改 Controller 代码。</p>
</li>
</ul>
<p data-nodeid="22385">如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="22386"><code data-language="javascript"> <span class="hljs-keyword">async</span> list() {
        <span class="hljs-keyword">let</span> contentList = <span class="hljs-keyword">await</span> <span class="hljs-keyword">new</span> ContentModel().getList();
        contentList = <span class="hljs-keyword">await</span> contentService.filterUserinfo(contentList);

        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resAPI(<span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, contentList);
    }
</code></pre>
<p data-nodeid="22387">注意代码中的第 4 行，从原来调用本类的方法，修改为调用 contentService 的 filterUserinfo。</p>
<h3 data-nodeid="22388">总结</h3>
<p data-nodeid="22389">本讲最开始介绍了一些技术知识点，这些是你开始学习本专栏必需巩固的技术，接下来根据实践开发了一个微型的 restful 服务，由于代码的不可维护性以及不可扩展性，我们接下来就应用了 MVC 架构设计模式进行了优化，最后由于 MVC 的缺陷，进而提出了使用 MSVC 来解决 MVC 中 M 和 C 业务界定不清晰的问题。</p>
<p data-nodeid="22390">学完本讲后，你就能自己写一个 restful API 了，并且能够掌握 MVC 和 MSVC 的架构原理，同时能够开发出轻量版本的框架。在实践过程中有任何问题或者心得，都可以在留言区留言。</p>
<p data-nodeid="22391">讲解完我们自身设计的简版框架后，在下一讲要介绍 Node.js 目前业界使用最广的三个框架，并且进行深入对比分析其优缺点。</p>
<hr data-nodeid="22392">
<p data-nodeid="22393"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="22674"><img src="https://s0.lgstatic.com/i/image6/M00/12/FA/CioPOWBBrAKAAod-AASyC72ZqWw233.png" alt="Drawing 2.png" data-nodeid="22673"></a></p>
<p data-nodeid="22394"><strong data-nodeid="22678">《大前端高薪训练营》</strong></p>
<p data-nodeid="22395" class="">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="22682">点击链接</a>，快来领取！</p>

---

### 精选评论

##### *宇：
> 这一章看得很过瘾啊，麻雀虽小五脏俱全🙇

##### *振：
> 同步异步都可以用await吧

##### **文：
> 还是没理解为什么一个restful，一个api层😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里只是一个演示，为了告诉大家获取数据的方式可能有多种，一种是来自数据库，一种是来自其他服务，而这里的api层，就是代表其他服务层。

##### **3813：
> MVC的图中，为什么V和M之间也有交互呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在目前这种前后端分离的场景是比较少的，在以往没有前后端分离的时候，比如说PHP或者JSP的时候，都是直接在前端页面中使用模版引擎，那样事可以直接调用 Model 层的数据的。而你说的就是进阶版 MVP 了，那就是阻隔了 M 与 V。

##### **业：
> 很精炼

##### console_man：
> 醍醐灌顶呀，受益匪浅

##### **菁：
> 赞👍

##### **博：
> 老师你的画图工具用的什么呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; draw.io 需要 VPN 才能访问，国内的话建议使用 processon 。

##### **用户2267：
> 为什要拆成两个server呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以哈，没有问题。

