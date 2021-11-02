<p data-nodeid="1073">从今天开始，我们将进入 Serverless 应用的场景案例篇。这一讲我将带你实现 Serverless 中的身份认证。</p>
<p data-nodeid="1074">我们平时用过那么多的网站和 App，很多都需要登录，登录的过程就是身份认证的过程。以电商网站为例，如果你想下单，先要注册账号，然后输入用户名（可能手机号或邮箱）、密码登录。之后你在一段时间内再访问该电商网站，都不用再登录，只有在连续长时间不登录的情况下（比如一个月），才用登录。这样一来，当你长时间不输入密码，很可能换了电脑或手机后，就忘记密码了。</p>
<p data-nodeid="1075">身份认证几乎是每个系统必备能力，所以很多同学开发应用时，实现的第一功能就是登录注册。然而当把应用迁移到 Serverless 架构时，很多同学就犯难了，基于 Serverless 的身份认证功能应该怎么实现呢？</p>
<p data-nodeid="1076">为了让你深入理解 Serverless 架构中的身份认证实现原理，我准备了今天的内容，这一讲我会先带你了解一些身份认证技术方案，然后带你从零到一实现一个 Serverless 的登录注册应用。</p>
<p data-nodeid="1077">话不多说，我们进入今天的学习。</p>
<h3 data-nodeid="1078">身份认证的技术方案</h3>
<p data-nodeid="1079">要实现应用中的身份认证，你首先要详细了解身份认证的技术方案，以及该方案怎么在 Serverless 架构中使用，因为有的技术方案可能不适合 Serverless。</p>
<h4 data-nodeid="1080">Cookie-Session</h4>
<p data-nodeid="1081">早期互联网主要以 Web 为主，客户端是浏览器，所以 Cookie-Session 方式是早期最常用的身份认证方式，直到现在很多 Web 网站依然使用这种方式。其认证流程是：</p>
<ul data-nodeid="1082">
<li data-nodeid="1083">
<p data-nodeid="1084">用户在浏览器中输入账号密码登录；</p>
</li>
<li data-nodeid="1085">
<p data-nodeid="1086">服务端验证通过后，将用户信息保存在 Session 中并生成一个 Session ID；</p>
</li>
<li data-nodeid="1087">
<p data-nodeid="1088">然后服务端将 Session ID 放在 HTTP 响应头的 cookie 字段中；</p>
</li>
<li data-nodeid="1089">
<p data-nodeid="1090">浏览器收到 HTTP 响应后，将 cookie 保存在浏览器中，cookie 内容就是之前登录时生成的 Session ID；</p>
</li>
<li data-nodeid="1091">
<p data-nodeid="1092">用户再访问网站时，浏览器请求头就会自动带上 cookie 信息；</p>
</li>
<li data-nodeid="1093">
<p data-nodeid="1094">服务端接收到请求后，从 cookie 获取到 Session ID，然后根据 Session ID 解析出用户信息。</p>
</li>
</ul>
<p data-nodeid="1095"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxNGAVJ2eAAHjY3afYhk253.png" data-nodeid="1215"></p>
<div data-nodeid="1096"><p style="text-align:center">Cookie-Session 身份认证流程</p></div>
<p data-nodeid="1097">这种方案存在两个主要问题：</p>
<ul data-nodeid="1098">
<li data-nodeid="1099">
<p data-nodeid="1100">服务端的 Session ID 是直接存储在内存中的，在分布式系统中无法共享登录状态；</p>
</li>
<li data-nodeid="1101">
<p data-nodeid="1102">cookie 是浏览器的功能，手机 App 等客户端并不支持 cookie，所以该方案不适用于非浏览器的应用。</p>
</li>
</ul>
<p data-nodeid="1103">第一个问题也是 Cookie-Session 方案应用于 Serverless 架构的主要问题，因为 Serverless 应用是无状态的，内存中的数据用完即销毁，多个请求间无法共享 Session。<strong data-nodeid="1224">解决该问题也比较容易，</strong> 就是用一个共享存储来保存 Session 信息，最常见的就是 Redis，因为 Redis 是一个内存数据库，读写速度很快。</p>
<p data-nodeid="1104">于是 Cookie-Session 的身份认证方案就发生了变化：</p>
<p data-nodeid="1105"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxOKAJBiFAALzfxQb7r8244.png" data-nodeid="1228"></p>
<div data-nodeid="1106"><p style="text-align:center">基于共享存储的 Cookie-Session 身份认证流程</p></div>
<p data-nodeid="1107">与早期方案不同，用户登录时，该方案会把用户信息保存在 Redis 中，而不是内存中，然后服务端依然会将 Session ID 返回给浏览器，浏览器将其保存在 cookie 中。而之后非登录的请求，浏览器依然会将包含 Session ID 的 cookie 放在请求头中发送给服务端，服务端拿到 Session ID 后，从 Redis 中查询出用户信息。这样就可以解决分布式、无状态的系统中用户登录状态共享问题。</p>
<p data-nodeid="1108">不过这个方案依旧无法解决非浏览器场景的身份认证问题，所以 JWT 方案诞生了。</p>
<h4 data-nodeid="1109">JWT</h4>
<p data-nodeid="1110">JWT 是（JSON Web Token）的简称，其原理是：</p>
<ul data-nodeid="1111">
<li data-nodeid="1112">
<p data-nodeid="1113">服务端认证通过后，根据用户信息生成一个 token 返回给客户端；</p>
</li>
<li data-nodeid="1114">
<p data-nodeid="1115">客户端将 token 存储在 cookie 或 localStorage 中；</p>
</li>
<li data-nodeid="1116">
<p data-nodeid="1117">之后客户端每次请求都需要带上 token，通常是将 token 放在 HTTP 请求头的 Authorization 字段中；</p>
</li>
<li data-nodeid="1118">
<p data-nodeid="1119">服务端接收到 token 后，验证 token 的合法性，并从 token 中解析出用户信息。</p>
</li>
</ul>
<p data-nodeid="1120"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/94/46/CgqCHmAXxOyAImLQAAHmAyEZuFk591.png" data-nodeid="1239"></p>
<div data-nodeid="1121"><p style="text-align:center">JWT 身份认证流程</p></div>
<p data-nodeid="1122">token 是个比较长字符串，格式为<code data-nodeid="1241" data-backticks="1">Header.Payload.Signature</code>，由<code data-nodeid="1243" data-backticks="1">.</code>分隔为三部分。下面是一个实际的 token 示例：</p>
<pre class="lang-java" data-nodeid="1123"><code data-language="java">eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwidXNlcm5hbWUiOiJqYWNrIiwiaWF0IjoxNjEwODg1MTcxfQ.KIduc-undaZ0z-Bt4wjGZIK5fMlx1auVHl_G1DvGDCw
</code></pre>
<p data-nodeid="1124"><strong data-nodeid="1254">可能有同学会担忧：</strong> token 是根据用户信息生成的，这样会不会泄露用户信息呢？其实不用担心，因为生成 token 的加密算法是不可逆的，并且 token 也可以设置过期时间，所以 token 字符串本身不会泄露用户信息。<br>
基于 JWT ，客户端可以使用自己特有的存储来保存 token，不依赖 cookie，所以 JWT 可以适用于任意客户端。并且使用 JWT 进行身份认证，服务端就不用存储用户信息了，这样服务端就是无状态的。<strong data-nodeid="1255">因此 JWT 这种身份认证方案，也非常适合 Serverless 应用。</strong></p>
<p data-nodeid="1125">接下来，我就基于 JWT ，带你从 0 到 1实现一个登录注册应用。</p>
<h3 data-nodeid="1126">从 0 到 1 实现一个登录注册应用</h3>
<p class="te-preview-highlight" data-nodeid="6361">为了方便，我们将基于 Express.js 框架进行开发。关于该应用的所有代码你可以在 Github 上查看：<a href="https://github.com/nodejh/serverless-class/tree/master/15/auth-app" data-nodeid="6365">Serverless Authorization</a>。</p>






<h4 data-nodeid="1128">应用初始化</h4>
<p data-nodeid="1129">首先安装 express、body-parser 和 @webserverless/fc-express 等依赖：</p>
<pre class="lang-java" data-nodeid="1130"><code data-language="java">$ npm i express body-parser <span class="hljs-meta">@webserverless</span>/fc-express -S
</code></pre>
<p data-nodeid="1131">@webserverless/fc-express 的作用是将函数计算的 HTTP 或 API 网关触发器参数转换为 Express.js 框架的参数，这样你就可以很方便在函数计算中使用 Express.js 了。</p>
<p data-nodeid="1132">然后我们初始化一个 template.yaml 模板，该模板定义了 auth-app 这个函数，函数触发器为 HTTP 触发器，支持 GET 和 POST 请求：</p>
<pre class="lang-yaml" data-nodeid="1133"><code data-language="yaml"><span class="hljs-attr">ROSTemplateFormatVersion:</span> <span class="hljs-string">'2015-09-01'</span>
<span class="hljs-attr">Transform:</span> <span class="hljs-string">'Aliyun::Serverless-2018-04-03'</span>
<span class="hljs-attr">Resources:</span>
  <span class="hljs-attr">serverless:</span>
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Service'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">Description:</span> <span class="hljs-string">'Serverless Authorization App'</span>
    <span class="hljs-attr">auth-app:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">index.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
        <span class="hljs-attr">Timeout:</span> <span class="hljs-number">10</span>
      <span class="hljs-attr">Events:</span>
        <span class="hljs-attr">httpTrigger:</span>
          <span class="hljs-attr">Type:</span> <span class="hljs-string">HTTP</span>
          <span class="hljs-attr">Properties:</span>
            <span class="hljs-attr">AuthType:</span> <span class="hljs-string">ANONYMOUS</span>
            <span class="hljs-attr">Methods:</span> <span class="hljs-string">['POST',</span> <span class="hljs-string">'GET'</span><span class="hljs-string">]</span>
</code></pre>
<p data-nodeid="1134">接下来在 index.js 中编写初始化代码，如下所示：</p>
<pre class="lang-javascript" data-nodeid="1135"><code data-language="javascript"><span class="hljs-keyword">const</span> proxy = <span class="hljs-built_in">require</span>(<span class="hljs-string">'@webserverless/fc-express'</span>)
<span class="hljs-keyword">const</span> express = <span class="hljs-built_in">require</span>(<span class="hljs-string">'express'</span>);
<span class="hljs-keyword">const</span> bodyParser = <span class="hljs-built_in">require</span>(<span class="hljs-string">'body-parser'</span>);
<span class="hljs-keyword">const</span> app = express();
app.use(bodyParser.urlencoded({
  <span class="hljs-attr">extended</span>: <span class="hljs-literal">true</span>
}));

<span class="hljs-comment">// 定义 / 路由，返回 Hello Serverless!</span>
app.get(<span class="hljs-string">'/'</span>, (req, res) =&gt; {
    res.json({
        <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
        <span class="hljs-attr">data</span>: <span class="hljs-string">'Hello Serverless!'</span>,
    });
});


<span class="hljs-keyword">const</span> server = <span class="hljs-keyword">new</span> proxy.Server(app);
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">req, res, context</span>) </span>{
    <span class="hljs-comment">// 使用 @webserverless/fc-express 来将函数计算的请求转发给 Express.js 应用</span>
    <span class="hljs-comment">// @webserverless/fc-express 可以将函数参数转换为 Express.js 的路由参数</span>
    server.httpProxy(req, res, context);
};
</code></pre>
<p data-nodeid="1136"><strong data-nodeid="1271">这段代码主要实现两个功能：</strong></p>
<ul data-nodeid="1137">
<li data-nodeid="1138">
<p data-nodeid="1139">定义了 <code data-nodeid="1273" data-backticks="1">/</code>&nbsp; 路由，该路由返回了 <code data-nodeid="1275" data-backticks="1">Hello Serverless!</code>&nbsp;字符串，我们之后可以用它来测试代码是否正常运行；</p>
</li>
<li data-nodeid="1140">
<p data-nodeid="1141">使用 @webserverless/fc-express 将函数计算的请求转发给 Express.js 应用，@webserverless/fc-express 可以将函数参数转换为 Express.js 的路由参数。</p>
</li>
</ul>
<p data-nodeid="1142">然后通过 <code data-nodeid="1279" data-backticks="1">fun deploy</code>部署应用：</p>
<pre class="lang-java" data-nodeid="1143"><code data-language="java"># 部署应用
$ fun deploy -y
Waiting for service serverless to be deployed...
        Waiting for function auth-app to be deployed...
                Waiting for packaging function auth-app code...
                The function auth-app has been packaged. A total of 419 files were compressed and the final size was 724.49 KB
                Waiting for HTTP trigger httpTrigger to be deployed...
                triggerName: httpTrigger
                methods: [ 'POST', 'GET' ]
                url: https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/
                trigger httpTrigger deploy success
        function auth-app deploy success
service serverless deploy success
$ curl https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/
{"success":true,"data":"Hello Serverless!"}
</code></pre>
<p data-nodeid="1144">部署成功后，我们就可以获取到函数计算提供的测试 HTTP Endpoint，然后就可以通过 curl 命令进行测试应用是否正常运行：</p>
<pre class="lang-java" data-nodeid="1145"><code data-language="java">$ curl https:<span class="hljs-comment">//1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>,<span class="hljs-string">"data"</span>:<span class="hljs-string">"Hello Serverless!"</span>}
</code></pre>
<p data-nodeid="1146">如果你和我一样返回了上述 JSON 字符串，就说明应用正常运行了，接下来我们就可以继续实现注册功能了。</p>
<h4 data-nodeid="1147">实现注册功能</h4>
<p data-nodeid="1148">注册的逻辑是：先获取用户输入的用户名和密码，然后判断用户是否存在，如果不存在就将其存入表格存储数据库。</p>
<p data-nodeid="3456"><strong data-nodeid="3465">这里我们使用的数据库是表格存储。</strong> 可能你使用的比较多的是 MySQL，之所以选用表格存储而不是 MySQL，是因为表格存储可以直接通过 Restful API 进行读写，并且弹性可扩展，更适合 Serverless 应用。使用表格存储时，你要先创建一个表格存储实例，然后创建一个 user 表。为了方便，我也给你提供了一个创建 user 表的脚本：<a href="https://github.com/nodejh/serverless-class/tree/master/15/create-table" data-nodeid="3463">create-table</a>。</p>





<p data-nodeid="1150">接下来继续编写代码。由于要使用表格存储，所以首先需要安装 tablestore 依赖，然后在 index.js 中初始化表格存储 client：</p>
<pre class="lang-java" data-nodeid="1151"><code data-language="java"># 安装 tablestore 依赖
# tablestore 封装了表格存储的 API
$ npm i tablestore -S
</code></pre>
<pre class="lang-javascript" data-nodeid="1152"><code data-language="javascript"><span class="hljs-comment">// index.js</span>
<span class="hljs-comment">// ...</span>
<span class="hljs-keyword">const</span> TableStore = <span class="hljs-built_in">require</span>(<span class="hljs-string">'tablestore'</span>);

<span class="hljs-comment">// 初始化 TableStore client</span>
<span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> TableStore.Client({
  <span class="hljs-attr">accessKeyId</span>: <span class="hljs-string">'&lt;your access key&gt;'</span>,
  <span class="hljs-attr">accessKeySecret</span>: <span class="hljs-string">'your access secret'</span>,
  <span class="hljs-attr">endpoint</span>: <span class="hljs-string">'https://serverless-app.cn-shanghai.ots.aliyuncs.com'</span>,
  <span class="hljs-attr">instancename</span>: <span class="hljs-string">'serverless-app'</span>,
});
</code></pre>
<p data-nodeid="1153">现在我们就可以定义一个路由来处理用户的注册请求了。代码如下所示，首先我们根据 name 从表格存储中查询用户信息，如果用户已存在，则直接返回；如果用户不存在，则将用户信息写入表格存储。</p>
<pre class="lang-javascript" data-nodeid="1154"><code data-language="javascript"><span class="hljs-comment">// 定义 /register 路由，处理注册请求</span>
app.post(<span class="hljs-string">'/register'</span>, <span class="hljs-keyword">async</span> (req, res) =&gt; {
  <span class="hljs-comment">// 从请求体中获取用户信息</span>
  <span class="hljs-keyword">const</span> name = req.body.name;
  <span class="hljs-keyword">const</span> password = req.body.password;
  <span class="hljs-keyword">const</span> age = req.body.age;
  <span class="hljs-comment">// 判断用户是否已经存在</span>
  <span class="hljs-keyword">const</span> { row } = <span class="hljs-keyword">await</span> client.getRow({
    <span class="hljs-attr">tableName</span>: <span class="hljs-string">"user"</span>,
    <span class="hljs-attr">primaryKey</span>: [{
      name
    }]
  });
  <span class="hljs-keyword">if</span> (row.primaryKey) {
    <span class="hljs-comment">// 如果用户已存在，则直接返回</span>
    <span class="hljs-keyword">return</span> res.json({
      <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
      <span class="hljs-attr">message</span>: <span class="hljs-string">'用户已存在'</span>
    });
  }
  <span class="hljs-comment">// 创建用户，将用户信息写入到表格存储中</span>
  <span class="hljs-keyword">await</span> client.putRow({
    <span class="hljs-attr">tableName</span>: <span class="hljs-string">"user"</span>,
    <span class="hljs-attr">condition</span>: <span class="hljs-keyword">new</span> TableStore.Condition(TableStore.RowExistenceExpectation.EXPECT_NOT_EXIST, <span class="hljs-literal">null</span>),
    <span class="hljs-attr">primaryKey</span>: [{
      name
    }],
    <span class="hljs-attr">attributeColumns</span>: [{
      password
    }, {
      age
    }]
  });
  <span class="hljs-comment">// 返回创建成功</span>
  <span class="hljs-keyword">return</span> res.send({
    <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
  });
});
</code></pre>
<p data-nodeid="1155">至此注册功能就完成了，你可以将代码部署到函数计算上，像下面这样通过 curl 命令来模拟用户请求，验证功能是否正常：</p>
<pre class="lang-javascript" data-nodeid="1156"><code data-language="javascript">$ curl https:<span class="hljs-comment">//1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth/login \</span>
-d <span class="hljs-string">"name=jack&amp;password=123456&amp;age=18"</span> \
-X POST
{<span class="hljs-string">"success"</span>:<span class="hljs-literal">true</span>}

$ curl https:<span class="hljs-comment">//1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth/login \</span>
-d <span class="hljs-string">"name=jack&amp;password=123456&amp;age=18"</span> \
-X POST
{<span class="hljs-string">"success"</span>:<span class="hljs-literal">false</span>,<span class="hljs-string">"message"</span>:<span class="hljs-string">"用户已存在"</span>}
</code></pre>
<p data-nodeid="1157">注册功能完成后，就可以继续实现登录功能了。</p>
<h4 data-nodeid="1158">实现登录功能</h4>
<p data-nodeid="1159">登录就是验证用户输入的用户名密码是否正确。</p>
<p data-nodeid="1160">首先根据用户输入的 name 从表格存储中查询出用户信息，然后对比用户密码与数据库中的用户密码是否一致，如果一致，则登录成功；否则登录失败。登录成功后，还需要根据用户信息生成一个 token 返回给用户。<strong data-nodeid="1304">具体怎么实现呢？</strong></p>
<p data-nodeid="1161">前面我们提到，Serverless 中最通用的身份认证方案是 JWT，所以我们首先需要安装 Node.js 中的 JWT 依赖包 jsonwebtoken：</p>
<pre class="lang-java" data-nodeid="1162"><code data-language="java">$ npm install jsonwebtoken -S
</code></pre>
<p data-nodeid="1163">然后在代码中引入 jsonwebtoken ，并定义 SECRET。SECRET 是用来加密和解密 token 的密钥，非常重要，且不能泄露。<br>
接下来在代码中定义 <code data-nodeid="1309" data-backticks="1">/login</code>&nbsp;路由来处理用户请求。这段代码中，我们首先验证了用户密码是否正确，密码正确后，再使用 jwt.sign() 方法，根据用户信息生成了 token，最后将 token 返回给客户端，客户端需要将 token 保存下来。之后客户端每次请求，都需要带上 token 进行身份认证。</p>
<pre class="lang-javascript" data-nodeid="1164"><code data-language="javascript"><span class="hljs-comment">// index.js</span>
<span class="hljs-comment">// ...</span>
<span class="hljs-keyword">const</span> jwt = <span class="hljs-built_in">require</span>(<span class="hljs-string">'jsonwebtoken'</span>)
<span class="hljs-comment">// 设置密钥，非常重要，不能泄露</span>
<span class="hljs-keyword">const</span> SECRET = <span class="hljs-string">'token_secret_xd2dasf19df='</span>
<span class="hljs-comment">// ...</span>
<span class="hljs-comment">// 定义 /login 路由，用来实现登录功能</span>
app.post(<span class="hljs-string">'/login'</span>, <span class="hljs-keyword">async</span> (req, res) =&gt; {
  <span class="hljs-comment">// 从请求体中获取用户名和密码</span>
  <span class="hljs-keyword">const</span> name = req.body.name;
  <span class="hljs-keyword">const</span> password = req.body.password;
  <span class="hljs-comment">// 根据用户名查询用户信息</span>
  <span class="hljs-keyword">const</span> {
    row
  } = <span class="hljs-keyword">await</span> client.getRow({
    <span class="hljs-attr">tableName</span>: <span class="hljs-string">'user'</span>,
    <span class="hljs-attr">primaryKey</span>: [{
      name
    }]
  })
  <span class="hljs-comment">// 如果查询结果为空，则直接返回用户不存在</span>
  <span class="hljs-keyword">if</span> (!row.primaryKey) {
    <span class="hljs-keyword">return</span> res.json({
      <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
      <span class="hljs-attr">message</span>: <span class="hljs-string">'用户不存在'</span>
    })
  }
  <span class="hljs-comment">// 从查询结果中构造用户信息</span>
  <span class="hljs-keyword">const</span> user = {
    name
  };
  row.attributes.forEach(<span class="hljs-function"><span class="hljs-params">item</span> =&gt;</span> user[item.columnName] = item.columnValue);
  <span class="hljs-comment">// 判断密码是否正确</span>
  <span class="hljs-keyword">if</span> (password !== user.password) {
    <span class="hljs-keyword">return</span> res.json({
      <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
      <span class="hljs-attr">message</span>: <span class="hljs-string">'密码错误'</span>
    })
  }
  user.password = <span class="hljs-string">'******'</span>;
  <span class="hljs-comment">/**
   * 生成 token
   * jwt.sign() 接受两个参数，一个是传入的对象，一个是自定义的密钥
   */</span>
  <span class="hljs-keyword">const</span> token = jwt.sign(user, SECRET)
  <span class="hljs-keyword">return</span> res.json({
    <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
    <span class="hljs-attr">data</span>: { token }
  })
});
</code></pre>
<p data-nodeid="1165">代码编写完成后，部署到函数计算并进行测试，如下所示：</p>
<pre class="lang-java" data-nodeid="1166"><code data-language="java">curl https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/login \
-d "name=jack&amp;password=123456" \ 
-X POST
{"success":true,"data":{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiamFjayIsImFnZSI6IjE4IiwicGFzc3dvcmQiOiIqKioqKioiLCJpYXQiOjE2MTA5MDY5MTJ9.qzNZarWbpDUA8-SO6nLd4ffEUR1IVOWKGXiocHV7MkU"}}

# 使用错误的密码登录
$ curl https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/login \
-d "name=jack&amp;password=1234561" \
-X POST
{"success":false,"message":"密码错误"}
</code></pre>
<p data-nodeid="1167">那么问题来了：对于需要登录后才能访问的接口，应该怎么根据 token 验证用户身份呢？别急，我们继续下面的学习。</p>
<h4 data-nodeid="1168">验证用户身份</h4>
<p data-nodeid="1169">前面提到，登录成功后，客户端需要将 token 保存下来，然后在接下来的请求中，都需要带上 token。通常会将 token 放在 HTTP 请求头中，格式通常为：</p>
<pre class="lang-java" data-nodeid="1170"><code data-language="java">Authorization: Bearer token
</code></pre>
<p data-nodeid="1171">这时假设我们要实现一个新的接口，获取当前登录用户信息，该接口也只能登录后才能使用。那么代码实现就是下面这样：</p>
<pre class="lang-javascript" data-nodeid="1172"><code data-language="javascript"><span class="hljs-comment">// 定义 /user 路由，获取当前登录的用户信息</span>
app.get(<span class="hljs-string">'/user'</span>, (req, res) =&gt; {
  <span class="hljs-comment">// 从 HTTP 请求头中获取 token 信息</span>
  <span class="hljs-keyword">const</span> token = req
    .headers
    .authorization
    .split(<span class="hljs-string">' '</span>)
    .pop();
  <span class="hljs-keyword">try</span> {
    <span class="hljs-comment">// 验证 token 并解析出用户信息</span>
    <span class="hljs-keyword">const</span> user = jwt.verify(token, SECRET);
    <span class="hljs-keyword">return</span> res.json({
      <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
      <span class="hljs-attr">data</span>: user
    })
  } <span class="hljs-keyword">catch</span> (error) {
    <span class="hljs-keyword">return</span> res.json({
      <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
      <span class="hljs-attr">data</span>: <span class="hljs-string">'身份认证失败'</span>
    })
  }
});
</code></pre>
<p data-nodeid="1173">首先我们定义了 <code data-nodeid="1317" data-backticks="1">/user</code>&nbsp;路由，然后通过请求头拿到 token 信息，最后使用 <code data-nodeid="1319" data-backticks="1">jwt.verify()</code>&nbsp;对 token 进行解密，并从中得到用户信息，如果用户传入的 token 无法解析，则说明用户身份异常。</p>
<p data-nodeid="1174">同样，我们可以将代码部署到函数计算并进行测试：</p>
<pre class="lang-java" data-nodeid="1175"><code data-language="java">curl https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/user \
-H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiamFjayIsImFnZSI6IjE4IiwicGFzc3dvcmQiOiIqKioqKioiLCJpYXQiOjE2MTA5MDY5MTJ9.qzNZarWbpDUA8-SO6nLd4ffEUR1IVOWKGXiocHV7MkU"
{"success":true,"data":{"name":"jack","age":"18","password":"******","iat":1610905944}}

# 使用错误的 token 进行身份认证
$ curl https://1457216987974698.cn-shanghai.fc.aliyuncs.com/2016-08-15/proxy/serverless/auth-app/user -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiamFjayIsImFnZSI6IjE4IiwicGFzc3dvcmQiOiIqKioqKioiLCJpYXQiOjE2MTA5MDY5MTJ9.qzNZarWbpDUA8-SO6nLd4ffEUR1IVOWKGXiocHV7Mk"
{"success":false,"data":"身份认证失败"}
</code></pre>
<p data-nodeid="1176">到此为止，一个 Serverless 架构的登录注册功能就完成了，我们也基于 JWT 实现了 Serverless 中的身份认证。</p>
<h3 data-nodeid="1177">总结</h3>
<p data-nodeid="1178">这一讲，我首先为你介绍了常用的身份认证方案，并分析了这些方案的特点，以及如何在 Serverless 架构中使用这些方案。然后我带你从0到1实现了一个 Serverless 的登录注册应用，通过该场景实践，让你深入理解 Serverless 中的身份认证。</p>
<p data-nodeid="1179">Serverless 应用的身份认证，本质上是要将有状态的认证方案改为无状态的。如何改为无状态呢？</p>
<ul data-nodeid="1180">
<li data-nodeid="1181">
<p data-nodeid="1182">使用共享存储来保存登录状态，比如将 Session 信息存储到 Reids，这样应用就不用存储状态了；</p>
</li>
<li data-nodeid="1183">
<p data-nodeid="1184">使用无状态的身份认证方案，比如 JWT。</p>
</li>
</ul>
<p data-nodeid="1185"><img alt="玩转 Serverless 架构15金句.png" src="https://s0.lgstatic.com/i/image6/M01/04/32/CioPOWAj2_2AUoDKAAEU4YVNqFA780.png" data-nodeid="1330"></p>
<p data-nodeid="1186">当然了，除了自己实现应用的身份认证，你也可以使用第三方的身份认证服务，比如 AWS Cognito、Google Firebase 等，基于它们，你就不用自己开发用户管理及身份认证功能了。</p>
<p data-nodeid="1187">关于这一讲，我想强调这样几点：</p>
<ul data-nodeid="1188">
<li data-nodeid="1189">
<p data-nodeid="1190">Cookie-Session 的身份认证方式，是在服务端存储 Session 信息，客户端（浏览器）通过 cookie 存储 Session ID；</p>
</li>
<li data-nodeid="1191">
<p data-nodeid="1192">JWT 的身份认证方式，是在服务端根据用户信息生成 token，客户端保存 token；</p>
</li>
<li data-nodeid="1193">
<p data-nodeid="1194">Cookie-Session 的认证方案通常是有状态的，对于分布式、无状态的应用，需要将 Session 保存在共享存储中；</p>
</li>
<li data-nodeid="1195">
<p data-nodeid="1196">JWT 的认证方式通常是无状态的，所以比较适合 Serverless 应用。</p>
</li>
</ul>
<p data-nodeid="1197">最后，希望通过今天的学习，你能深入了解 Serverless 应用的身份认证，今天留给你的作业就是：亲自动手实现一个 Serverless 的登录注册应用，我们下一讲见。</p>

---

### 精选评论

##### *乐：
> 为什么要转发消息到Express.js呢？直接在函数计算中加密得到token，解密token得到信息不好么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用  Express.js 我们可以更方便地构建一个 Web 应用，本节课基于 Express.js 开发也是讲解如何基于 Express.js 开发 Serverless 应用，这样学完本节课之后，您就会学到如何将已有的 Express.js 应用迁移到 Serverless 平台。此外下节课中我就讲解了如何不使用框架直接开发 Serverless 应用。

