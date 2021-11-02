<p data-nodeid="63933" class="">今天我想和你聊一聊怎么基于 Serverless 构建弹性可扩展的 Restful API。</p>


<p data-nodeid="62660" class="">API 是使用 Serverless 最常见，也是最适合的场景之一。和 Serverful 架构的 &nbsp;API 相比，用 Serverless 开发 API 好处很多：</p>
<ul data-nodeid="62661">
<li data-nodeid="62662">
<p data-nodeid="62663">不用购买、管理服务器等基础设施，不用关心服务器的运维，节省人力成本；</p>
</li>
<li data-nodeid="62664">
<p data-nodeid="62665">基于 Serverless 的 API，具备自动弹性伸缩的能力，能根据请求流量弹性扩缩容，让你不再担心流量波峰、波谷；</p>
</li>
<li data-nodeid="62666">
<p data-nodeid="62667">基于 Serverless 的 API 按实际资源使用量来付费，节省财务成本。</p>
</li>
</ul>
<p data-nodeid="62668">因为好处很多，很多开发者跃跃欲试，但在实践过程中却遇到了很多问题，比如怎么设计最优的架构？怎么组织代码？怎么管理多个函数？所以今天我就以开发一个内容管理系统为例，带你学习怎么基于 Serverless 去开发一个 Restful API，解决上述共性问题。</p>
<p data-nodeid="62669">首先，我们需要对内容管理系统进行架构设计。</p>
<h3 data-nodeid="62670">内容管理系统的架构设计</h3>
<p data-nodeid="62671">在进行架构设计前，你要明确系统的需求。对于一个内容管理系统，最核心的功能（也是这一讲要实现的功能），主要有这样几个：</p>
<ul data-nodeid="62672">
<li data-nodeid="62673">
<p data-nodeid="62674">用户注册；</p>
</li>
<li data-nodeid="62675">
<p data-nodeid="62676">用户登录；</p>
</li>
<li data-nodeid="62677">
<p data-nodeid="62678">发布文章；</p>
</li>
<li data-nodeid="62679">
<p data-nodeid="62680">修改文章；</p>
</li>
<li data-nodeid="62681">
<p data-nodeid="62682">删除文章；</p>
</li>
<li data-nodeid="62683">
<p data-nodeid="62684">查询文章。</p>
</li>
</ul>
<p data-nodeid="62685">这 6 个功能分别对应了我们要实现的 Restful API。为了方便统一管理 API，在 Serverless 架构中我们通常会用到 API 网关，通过 API 网关触发函数执行，并且基于 &nbsp;API 网关我们还可以实现参数控制、超时时间、IP 黑名单、流量控制等高级功能。</p>
<p data-nodeid="62686">对于文章管理相关的 Restful API，用户发布文章前需要先登录，在 15 讲，你已经知道在 Serverless 中可以用 JWT 进行身份认证，咱们的管理系统中的登录注册功能也将沿用上一讲的内容。</p>
<p data-nodeid="62687">在传统的 Serverful 架构中，通常会用 MySQL 等关系型数据库存储数据，但因为关系型数据库要在代码中维护连接状态及连接池，且一般不能自动扩容，并不适合 Serverless 应用，所以在 Serverless 架构中，通常选用表格存储等 Serverless NoSQL 数据来存储数据。</p>
<p data-nodeid="62688">基于 JWT 的身份认证方案、数据存储方案，我们可以画出 Serverless 的内容管理系统架构图：</p>
<p data-nodeid="64781" class=""><img src="https://s0.lgstatic.com/i/image2/M01/0C/35/CgpVE2AXxXCASAwbAAKdD1n4Tyk774.png" alt="Drawing 0.png" data-nodeid="64784"></p>

<p data-nodeid="65631" class=""><strong data-nodeid="65636">图中主要表达的意思是：</strong> 通过 API 网关承接用户请求，并驱动函数执行。每个函数分别实现一个具体功能，并通过 JWT 实现身份认证，最后表格存储作为数据库。</p>

<p data-nodeid="62691">其中，数据库中存储的数据主要是用户数据和文章数据。假设用户有 username（用户名） 和 password（密码） 两个属性；文章有 article_id（文章 ID）、username（创建者）、title（文章标题）、content（文章内容）、create_date（创建时间）、update_date（更新时间）这几个属性。</p>
<p data-nodeid="68197" class=""><img src="https://s0.lgstatic.com/i/image2/M01/0C/35/CgpVE2AXxXyAUsksAAD9II6PlcU787.png" alt="Drawing 1.png" data-nodeid="68200"></p>

<p data-nodeid="67342" class="">接下来，你可以在表格存储中创建对应的数据表（你可以在表格存储控制台创建，也可以直接用我提供的这段代码进行创建）：</p>


<pre class="lang-javascript" data-nodeid="62693"><code data-language="javascript"><span class="hljs-comment">// index.js</span>
<span class="hljs-keyword">const</span> TableStore = <span class="hljs-built_in">require</span>(<span class="hljs-string">"tablestore"</span>);
<span class="hljs-comment">// 初始化 TableStore client</span>
<span class="hljs-keyword">const</span> client = <span class="hljs-keyword">new</span> TableStore.Client({
  <span class="hljs-attr">accessKeyId</span>: <span class="hljs-string">'&lt;your access key&gt;'</span>,
  <span class="hljs-attr">accessKeySecret</span>: <span class="hljs-string">'&lt;your access secret&gt;'</span>,
  <span class="hljs-attr">endpoint</span>: <span class="hljs-string">"https://serverless-app.cn-shanghai.ots.aliyuncs.com"</span>,
  <span class="hljs-attr">instancename</span>: <span class="hljs-string">"serverless-cms"</span>,
});
<span class="hljs-comment">/**
 * 创建 user 表
 *
 * 参考文档： https://help.aliyun.com/document_detail/100594.html
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">createUserTable</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> table = {
    <span class="hljs-attr">tableMeta</span>: {
      <span class="hljs-attr">tableName</span>: <span class="hljs-string">"user"</span>,
      <span class="hljs-attr">primaryKey</span>: [
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"username"</span>, <span class="hljs-comment">// 用户名</span>
          <span class="hljs-attr">type</span>: TableStore.PrimaryKeyType.STRING,
        },
      ],
      <span class="hljs-attr">definedColumn</span>: [
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"password"</span>, <span class="hljs-comment">// 密码</span>
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
      ],
    },
    <span class="hljs-comment">// 为数据表配置预留读吞吐量或预留写吞吐量。0 表示不预留吞吐量，完全按量付费</span>
    <span class="hljs-attr">reservedThroughput</span>: {
      <span class="hljs-attr">capacityUnit</span>: {
        <span class="hljs-attr">read</span>: <span class="hljs-number">0</span>,
        <span class="hljs-attr">write</span>: <span class="hljs-number">0</span>,
      },
    },
    <span class="hljs-attr">tableOptions</span>: {
      <span class="hljs-comment">// 数据的过期时间，单位为秒，-1表示永不过期</span>
      <span class="hljs-attr">timeToLive</span>: <span class="hljs-number">-1</span>,
      <span class="hljs-comment">// 保存的最大版本数，1 表示每列上最多保存一个版本即保存最新的版本</span>
      <span class="hljs-attr">maxVersions</span>: <span class="hljs-number">1</span>,
    },
  };
  <span class="hljs-keyword">await</span> client.createTable(table);
}
<span class="hljs-comment">/**
 * 创建文章表
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">createArticleTable</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> table = {
    <span class="hljs-attr">tableMeta</span>: {
      <span class="hljs-attr">tableName</span>: <span class="hljs-string">"article"</span>,
      <span class="hljs-attr">primaryKey</span>: [
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"article_id"</span>, <span class="hljs-comment">// 文章 ID，唯一字符串</span>
          <span class="hljs-attr">type</span>: TableStore.PrimaryKeyType.STRING,
        },
      ],
      <span class="hljs-attr">definedColumn</span>: [
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"title"</span>,
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"username"</span>,
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"content"</span>,
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"create_date"</span>,
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
        {
          <span class="hljs-attr">name</span>: <span class="hljs-string">"update_date"</span>,
          <span class="hljs-attr">type</span>: TableStore.DefinedColumnType.DCT_STRING,
        },
      ],
    },
    <span class="hljs-comment">// 为数据表配置预留读吞吐量或预留写吞吐量。0 表示不预留吞吐量，完全按量付费</span>
    <span class="hljs-attr">reservedThroughput</span>: {
      <span class="hljs-attr">capacityUnit</span>: {
        <span class="hljs-attr">read</span>: <span class="hljs-number">0</span>,
        <span class="hljs-attr">write</span>: <span class="hljs-number">0</span>,
      },
    },
    <span class="hljs-attr">tableOptions</span>: {
      <span class="hljs-comment">// 数据的过期时间，单位为秒，-1表示永不过期</span>
      <span class="hljs-attr">timeToLive</span>: <span class="hljs-number">-1</span>,
      <span class="hljs-comment">// 保存的最大版本数，1 表示每列上最多保存一个版本即保存最新的版本</span>
      <span class="hljs-attr">maxVersions</span>: <span class="hljs-number">1</span>,
    },
  };
  <span class="hljs-keyword">await</span> client.createTable(table);
}
(<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">await</span> createUserTable();
  <span class="hljs-keyword">await</span> createArticleTable();
})();
</code></pre>
<p data-nodeid="69051">这段代码主要创建了 user 和 article 两张表，其中 user 表的主键是 username，article 表的主键是 article_id，主键的作用是方便查询。除了主键，我还定义了几个列。其实对于表格存储，默认也可以不创建列，表格存储是宽表，除主键外，数据列可以随意扩展。</p>
<p data-nodeid="69052">在完成了数据库表的创建后，我们就可以开始进行系统实现了。</p>

<h3 data-nodeid="62695">内容管理系统的实现</h3>
<p data-nodeid="62696">为了方便你学习，我为你提供了完整代码（<a href="https://github.com/nodejh/serverless-class/tree/master/15/cms" data-nodeid="62906">代码地址</a>），你可以参考着学习。</p>
<pre class="lang-java" data-nodeid="62697"><code data-language="java">$ git clone https:<span class="hljs-comment">//github.com/nodejh/serverless-class</span>
$ cd <span class="hljs-number">15</span>/cms
</code></pre>
<p data-nodeid="62698">整个代码目录结构如下：</p>
<pre class="lang-java" data-nodeid="62699"><code data-language="java">.
├── <span class="hljs-keyword">package</span>.json
├── src
│&nbsp;&nbsp; ├── config
│&nbsp;&nbsp; │&nbsp;&nbsp; └── index.js
│&nbsp;&nbsp; ├── db
│&nbsp;&nbsp; │&nbsp;&nbsp; └── client.js
│&nbsp;&nbsp; ├── function
│&nbsp;&nbsp; │&nbsp;&nbsp; ├── article
│&nbsp;&nbsp; │&nbsp;&nbsp; │&nbsp;&nbsp; ├── create.js
│&nbsp;&nbsp; │&nbsp;&nbsp; │&nbsp;&nbsp; ├── delete.js
│&nbsp;&nbsp; │&nbsp;&nbsp; │&nbsp;&nbsp; ├── detail.js
│&nbsp;&nbsp; │&nbsp;&nbsp; │&nbsp;&nbsp; └── update.js
│&nbsp;&nbsp; │&nbsp;&nbsp; └── user
│&nbsp;&nbsp; │&nbsp;&nbsp;     ├── login.js
│&nbsp;&nbsp; │&nbsp;&nbsp;     └── register.js
│&nbsp;&nbsp; └── middleware
│&nbsp;&nbsp;     └── auth.js
└── template.yml
</code></pre>
<p data-nodeid="62700">其中，所有业务代码都放在 src 目录中：</p>
<ul data-nodeid="62701">
<li data-nodeid="62702">
<p data-nodeid="62703">config/index.js 是配置文件，里面包含身份凭证等配置信息；</p>
</li>
<li data-nodeid="62704">
<p data-nodeid="62705">db/client.js 对表格存储的增删改查操作进行了封装，方便在函数中使用（将数据库的操作封装还有一个好处是，如果你之后想要迁移到其他数据库，只要修改 db/client.js 中的逻辑，不用修改业务代码）；</p>
</li>
<li data-nodeid="62706">
<p data-nodeid="62707">middleware 目录中是一些中间件，比如 auth.js，用于身份认证；</p>
</li>
<li data-nodeid="62708">
<p data-nodeid="62709">functions 目录中就是所有函数，登录、注册、创建文章等，每个功能分别对应一个函数；</p>
</li>
<li data-nodeid="62710">
<p data-nodeid="62711">template.yaml 是应用配置文件，包括函数和 API 网关的配置。</p>
</li>
</ul>
<p data-nodeid="62712">根据前面梳理的系统功能，我们需要实现以下几个 API：</p>
<table data-nodeid="62714">
<thead data-nodeid="62715">
<tr data-nodeid="62716">
<th data-nodeid="62718">用户注册</th>
<th data-nodeid="62719">POST /user/register</th>
</tr>
</thead>
<tbody data-nodeid="62722">
<tr data-nodeid="62723">
<td data-nodeid="62724">用户登录</td>
<td data-nodeid="62725">POST /user/login</td>
</tr>
<tr data-nodeid="62726">
<td data-nodeid="62727">发布文章</td>
<td data-nodeid="62728">POST /article/create</td>
</tr>
<tr data-nodeid="62729">
<td data-nodeid="62730">查询文章</td>
<td data-nodeid="62731">GET /article/detail/[article_id]</td>
</tr>
<tr data-nodeid="62732">
<td data-nodeid="62733">更新文章</td>
<td data-nodeid="62734">POST /article/update</td>
</tr>
<tr data-nodeid="62735">
<td data-nodeid="62736">删除文章</td>
<td data-nodeid="62737">PUT /article/delete/[article_id]</td>
</tr>
</tbody>
</table>
<p data-nodeid="62738">每个 API 对应一个具体的函数，每个函数也都有一个与之对应的 API 网关触发器。由于这些函数属于同一个应用，所以我们可以通过一个 template.yaml 来定义所有函数。同时也可以在 template.yaml 中定义函数的 API 网关触发器，这样部署函数时，就会自动创建 API 网关。</p>
<p data-nodeid="62739">内容管理系统的 template.yaml 格式如下：</p>
<pre class="lang-yaml" data-nodeid="62740"><code data-language="yaml"><span class="hljs-attr">ROSTemplateFormatVersion:</span> <span class="hljs-string">'2015-09-01'</span>
<span class="hljs-attr">Transform:</span> <span class="hljs-string">'Aliyun::Serverless-2018-04-03'</span>
<span class="hljs-attr">Resources:</span>
  <span class="hljs-comment"># 函数服务，该服务中的函数都是内容管理系统的函数</span>
  <span class="hljs-attr">serverless-cms:</span>
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Service'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">Description:</span> <span class="hljs-string">'Serverless 内容管理系统'</span>
    <span class="hljs-comment"># 函数名称</span>
    <span class="hljs-string">[functionName]:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-comment"># 函数路径</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">&lt;functionPath&gt;.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
  <span class="hljs-comment"># API 网关分组，分钟中的所有 API 都是内容管理系统的 API</span>
  <span class="hljs-attr">ServerlessCMSGroup:</span> 
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Api'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">StageName:</span> <span class="hljs-string">RELEASE</span>
      <span class="hljs-attr">DefinitionBody:</span>
        <span class="hljs-string">&lt;Path&gt;:</span> <span class="hljs-comment"># 请求的 path</span>
          <span class="hljs-attr">post:</span> <span class="hljs-comment"># 请求的 method</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">user_register</span> <span class="hljs-comment"># API 名称</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span> <span class="hljs-comment"># 当请求该 API 时，要触发的函数，</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${&lt;functionName&gt;.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
</code></pre>
<p data-nodeid="70767"><strong data-nodeid="70773">template.yaml &nbsp;主要分为两部分：</strong> 函数定义和 API 网关定义，每个函数都有一个与之对应的 API 网关。我们用 serverless-cms 服务来表示内容管理系统这个应用，服务内的所有函数都是内容管理系统的函数。同理，ServerlessCMSGroup 这个 API 网关分组中的所有 API 都是内容管理系统的 API。</p>
<p data-nodeid="70768">完整的 template.yaml 配置如下：</p>


<pre class="lang-yaml" data-nodeid="62742"><code data-language="yaml"><span class="hljs-attr">ROSTemplateFormatVersion:</span> <span class="hljs-string">'2015-09-01'</span>
<span class="hljs-attr">Transform:</span> <span class="hljs-string">'Aliyun::Serverless-2018-04-03'</span>
<span class="hljs-attr">Resources:</span>
  <span class="hljs-comment"># 函数服务</span>
  <span class="hljs-attr">serverless-cms:</span>
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Service'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">Description:</span> <span class="hljs-string">'Serverless 内容管理系统'</span>
    <span class="hljs-attr">user-register:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/user/register.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
    <span class="hljs-attr">user-login:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/user/login.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
    <span class="hljs-attr">article-create:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/article/create.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
    <span class="hljs-attr">article-detail:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/article/detail.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
    <span class="hljs-attr">article-update:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/article/update.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
    <span class="hljs-attr">article-delete:</span>
      <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Function'</span>
      <span class="hljs-attr">Properties:</span>
        <span class="hljs-attr">Handler:</span> <span class="hljs-string">src/function/article/delete.handler</span>
        <span class="hljs-attr">Runtime:</span> <span class="hljs-string">nodejs12</span>
        <span class="hljs-attr">CodeUri:</span> <span class="hljs-string">'./'</span>
  <span class="hljs-comment"># API 网关分组</span>
  <span class="hljs-attr">ServerlessCMSGroup:</span> 
    <span class="hljs-attr">Type:</span> <span class="hljs-string">'Aliyun::Serverless::Api'</span>
    <span class="hljs-attr">Properties:</span>
      <span class="hljs-attr">StageName:</span> <span class="hljs-string">RELEASE</span>
      <span class="hljs-attr">DefinitionBody:</span>
        <span class="hljs-string">'/user/register'</span><span class="hljs-string">:</span> <span class="hljs-comment"># 请求的 path</span>
          <span class="hljs-attr">post:</span> <span class="hljs-comment"># 请求的 method</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">user_register</span> <span class="hljs-comment"># API 名称</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span> <span class="hljs-comment"># 当请求该 API 时，要触发的函数，</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${user-register.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
        <span class="hljs-string">'/user/login'</span><span class="hljs-string">:</span>
          <span class="hljs-attr">post:</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">user_login</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${user-login.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
        <span class="hljs-string">'/article/create'</span><span class="hljs-string">:</span>
          <span class="hljs-attr">post:</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">article_create</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${article-create.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
        <span class="hljs-string">'/article/detail/[article_id]'</span><span class="hljs-string">:</span>
          <span class="hljs-attr">GET:</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">article_detail</span>
            <span class="hljs-attr">x-aliyun-apigateway-request-parameters:</span>
              <span class="hljs-bullet">-</span> <span class="hljs-attr">apiParameterName:</span> <span class="hljs-string">'article_id'</span>
                <span class="hljs-attr">location:</span> <span class="hljs-string">'Path'</span>
                <span class="hljs-attr">parameterType:</span> <span class="hljs-string">'String'</span>
                <span class="hljs-attr">required:</span> <span class="hljs-string">'REQUIRED'</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${article-detail.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
        <span class="hljs-string">'/article/update/[article_id]'</span><span class="hljs-string">:</span>
          <span class="hljs-attr">PUT:</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">article_update</span>
            <span class="hljs-attr">x-aliyun-apigateway-request-parameters:</span>
              <span class="hljs-bullet">-</span> <span class="hljs-attr">apiParameterName:</span> <span class="hljs-string">'article_id'</span>
                <span class="hljs-attr">location:</span> <span class="hljs-string">'Path'</span>
                <span class="hljs-attr">parameterType:</span> <span class="hljs-string">'String'</span>
                <span class="hljs-attr">required:</span> <span class="hljs-string">'REQUIRED'</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${article-update.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
        <span class="hljs-string">'/article/delete/[article_id]'</span><span class="hljs-string">:</span>
          <span class="hljs-attr">DELETE:</span>
            <span class="hljs-attr">x-aliyun-apigateway-api-name:</span> <span class="hljs-string">article_update</span>
            <span class="hljs-attr">x-aliyun-apigateway-request-parameters:</span>
              <span class="hljs-bullet">-</span> <span class="hljs-attr">apiParameterName:</span> <span class="hljs-string">'article_id'</span>
                <span class="hljs-attr">location:</span> <span class="hljs-string">'Path'</span>
                <span class="hljs-attr">parameterType:</span> <span class="hljs-string">'String'</span>
                <span class="hljs-attr">required:</span> <span class="hljs-string">'REQUIRED'</span>
            <span class="hljs-attr">x-aliyun-apigateway-fc:</span>
              <span class="hljs-attr">arn:</span> <span class="hljs-string">acs:fc:::services/${serverless-cms.Arn}/functions/${article-delete.Arn}/</span>
              <span class="hljs-attr">timeout:</span> <span class="hljs-number">3000</span>
            
</code></pre>
<p data-nodeid="62743">在这份配置中，需要注意两个地方：</p>
<ul data-nodeid="62744">
<li data-nodeid="62745">
<p data-nodeid="62746">函数的 Handler 配置，Handler 可以写函数路径，比如<code data-backticks="1" data-nodeid="62948">src/function/user/register.handler</code>表示<code data-backticks="1" data-nodeid="62950">src/function/user/</code>目录中的 register.js 文件中的 handler 方法；</p>
</li>
<li data-nodeid="62747">
<p data-nodeid="62748">API 网关配置中的<code data-backticks="1" data-nodeid="62953">/article/detail/[article_id]</code>Path，这种带参数的 PATH，必须使用<code data-backticks="1" data-nodeid="62955">x-aliyun-apigateway-request-parameters</code>指定 Path 参数。</p>
</li>
</ul>
<p data-nodeid="62749">接下来，我们就来实现内容管理系统的各个 API，也就是  template.yaml 中定义的各个函数。</p>
<h4 data-nodeid="74186" class="">用户注册</h4>




<p data-nodeid="62751">用户注册接口定义如下。</p>
<ul data-nodeid="62752">
<li data-nodeid="62753">
<p data-nodeid="62754">请求方法：POST。</p>
</li>
<li data-nodeid="62755">
<p data-nodeid="62756">Path：<code data-backticks="1" data-nodeid="62965">/user/register</code></p>
</li>
<li data-nodeid="62757">
<p data-nodeid="62758">Body参数：username 用户名、password 密码。</p>
</li>
</ul>
<p data-nodeid="62759">整体代码很简单，在入口函数 handler 中，通过 event 得到 API 网关传递过来的 HTTP 请求 body 数据，然后从中得到 username、password，再将用户信息写入数据库。</p>
<pre class="lang-javascript" data-nodeid="62760"><code data-language="javascript"><span class="hljs-comment">// src/function/user/register</span>
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 用户注册
 * @param {string} username 用户名
 * @param {string} password 密码
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">register</span>(<span class="hljs-params">username, password</span>) </span>{
  <span class="hljs-keyword">await</span> client.createRow(<span class="hljs-string">"user"</span>, { username }, { password });
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 从 event 中获取 API 网关传递 HTTP 请求 body 数据</span>
  <span class="hljs-keyword">const</span> body = <span class="hljs-built_in">JSON</span>.parse(<span class="hljs-built_in">JSON</span>.parse(event.toString()).body);
  <span class="hljs-keyword">const</span> { username, password } = body;
  register(username, password)
    .then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> callback(<span class="hljs-literal">null</span>, { <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span> }))
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, { <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>, <span class="hljs-attr">message</span>: <span class="hljs-string">"用户注册失败"</span> })
    );
};
</code></pre>
<p data-nodeid="62761">代码完成后，就可以将应用部署到函数计算：</p>
<pre class="lang-java" data-nodeid="62762"><code data-language="java"># 部署应用
$ fun deploy
Waiting for service serverless-cms to be deployed...
...
service serverless-cms deploy success
Waiting for api gateway ServerlessCMSGroup to be deployed...
...
api gateway ServerlessCMSGroup deploy success
</code></pre>
<p data-nodeid="75034">部署过程中，如果看到函数服务 serverless-cms &nbsp;和 API 网关 ServerlessCMSGroup 都成功部署了，就说明应用部署完成。部署完成后，API 网关会提供一个用来测试的 API Endpoint，当然你也可以绑定自定义域名。</p>
<p data-nodeid="75035">我们可以通过 curl 测试一下：</p>

<pre class="lang-java" data-nodeid="62764"><code data-language="java">$ curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/user/register \</span>
-X POST \
-d <span class="hljs-string">"username=Jack&amp;password=123456"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>}
</code></pre>
<p data-nodeid="62765">返回 <code data-backticks="1" data-nodeid="62973">{"success": true}</code> ，说明用户注册成功。这时在表格存储控制台也可以看到刚注册的用户。</p>
<p data-nodeid="79267" class=""><img src="https://s0.lgstatic.com/i/image/M00/94/46/CgqCHmAXxaeAe89-AADufUP1UJA961.png" alt="Drawing 2.png" data-nodeid="79270"></p>

<h4 data-nodeid="78425" class="">用户登录</h4>




<p data-nodeid="62768">完成用户注册函数开发后，就可以接着开发登录。用户登录的接口定义如下。</p>
<ul data-nodeid="62769">
<li data-nodeid="62770">
<p data-nodeid="62771">请求方法：POST。</p>
</li>
<li data-nodeid="62772">
<p data-nodeid="62773">Path：<code data-backticks="1" data-nodeid="62985">/user/login</code></p>
</li>
<li data-nodeid="62774">
<p data-nodeid="62775">Body 参数：username 用户名、password 密码。</p>
</li>
</ul>
<p data-nodeid="62776">登录的逻辑就是根据用户输入的密码是否正确，如果正确就生成一个 token 返回给调用方。代码实现如下：</p>
<pre class="lang-javascript" data-nodeid="62777"><code data-language="javascript"><span class="hljs-comment">// src/function/user/login</span>
<span class="hljs-keyword">const</span> assert = <span class="hljs-built_in">require</span>(<span class="hljs-string">"assert"</span>);
<span class="hljs-keyword">const</span> jwt = <span class="hljs-built_in">require</span>(<span class="hljs-string">'jsonwebtoken'</span>);
<span class="hljs-keyword">const</span> { jwt_secret } = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../config"</span>);
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 用户登录
 * @param {string} username 用户名
 * @param {string} password 密码
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">login</span>(<span class="hljs-params">username, password</span>) </span>{
  <span class="hljs-keyword">const</span> user = <span class="hljs-keyword">await</span> client.getRow(<span class="hljs-string">"user"</span>, { username });
  assert(user &amp;&amp; user.password === password);
  <span class="hljs-keyword">const</span> token = jwt.sign({ <span class="hljs-attr">username</span>: user.username }, jwt_secret);
  <span class="hljs-keyword">return</span> token;
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-keyword">const</span> body = <span class="hljs-built_in">JSON</span>.parse(<span class="hljs-built_in">JSON</span>.parse(event.toString()).body);
  <span class="hljs-keyword">const</span> { username, password } = body;
  login(username, password)
    .then(<span class="hljs-function">(<span class="hljs-params">token</span>) =&gt;</span> callback(<span class="hljs-literal">null</span>, { <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>, <span class="hljs-attr">data</span>: { token } }))
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, { <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>, <span class="hljs-attr">message</span>: <span class="hljs-string">"用户登录失败"</span> })
    );
};
</code></pre>
<p data-nodeid="62778">将其部署到函数计算后，我们也可以使用 curl 命令进行测试：</p>
<pre class="lang-java" data-nodeid="62779"><code data-language="java">$ curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/user/login \</span>
-X POST \
-d <span class="hljs-string">"username=Jack&amp;password=123456"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>,<span class="hljs-string">"data"</span>:{<span class="hljs-string">"token"</span>:<span class="hljs-string">"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkphY2siLCJpYXQiOjE2MTE0OTI2ODF9.c56Xm4RBLYl5yVtR_Vk0IZOL0yijofcyE-P7vjKf4nA"</span>}}
</code></pre>
<h4 data-nodeid="83463" class="">身份认证</h4>






<p data-nodeid="62782">在完成了注册登录接口后，我们再来看一下内容管理系统中，身份认证应该怎么实现。</p>
<p data-nodeid="62783">在 15 讲，我们实现了一个 Express.js 框架的身份认证中间件，用来拦截所有请求，身份认证通过后才能进执行后面的代码逻辑。在内容管理系统中，你也可以参考 Express.js 的思想，实现一个 auth.js 专门用于身份认证，代码如下：</p>
<pre class="lang-javascript" data-nodeid="62784"><code data-language="javascript"><span class="hljs-comment">// src/middleware/auth.js</span>
<span class="hljs-keyword">const</span> jwt = <span class="hljs-built_in">require</span>(<span class="hljs-string">"jsonwebtoken"</span>);
<span class="hljs-keyword">const</span> { jwt_secret } = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../config/index"</span>);
<span class="hljs-comment">/**
 * 身份认证
 * @param {object} event API 网关的 event 对象
 * @return {object} 认证通过后返回 user 信息；认证失败则返回 false
 */</span>
<span class="hljs-keyword">const</span> auth = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event</span>) </span>{
  <span class="hljs-keyword">try</span> {
    <span class="hljs-keyword">const</span> data = <span class="hljs-built_in">JSON</span>.parse(event.toString());
    <span class="hljs-keyword">if</span> (data.headers &amp;&amp; data.headers.Authorization) {
      <span class="hljs-keyword">const</span> token = <span class="hljs-built_in">JSON</span>.parse(event.toString())
        .headers.Authorization.split(<span class="hljs-string">" "</span>)
        .pop();
      <span class="hljs-keyword">const</span> user = jwt.verify(token, jwt_secret);
      <span class="hljs-keyword">return</span> user;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
  } <span class="hljs-keyword">catch</span> (error) {
    <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
  }
};
<span class="hljs-built_in">module</span>.exports = auth;
</code></pre>
<p data-nodeid="83881">其原理很简单，就是从 API 网关的 event 对象中获取 token，然后验证 token 是否正常。如果认证通过，就返回 user 信息，失败就返回 false。</p>
<p data-nodeid="83882">这样在需要身份认证的函数中，你只引入 auth.js 并传入 event 对象就可以了。下面是一个简单的示例：</p>

<pre class="lang-javascript" data-nodeid="62786"><code data-language="javascript"><span class="hljs-keyword">const</span> auth = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./middleware/auth'</span>);
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 使用 auth 进行身份认证</span>
  <span class="hljs-keyword">const</span> user = auth(event);
  <span class="hljs-keyword">if</span> (!user) {
    <span class="hljs-comment">// 若认证失败则直接返回</span>
    <span class="hljs-keyword">return</span> callback(<span class="hljs-string">'身份认证失败!'</span>)
  }
  <span class="hljs-comment">// 通过身份认证后的业务逻辑</span>
  <span class="hljs-comment">// ...</span>
  callback(<span class="hljs-literal">null</span>);
};
</code></pre>
<p data-nodeid="62787">除了登录注册，其他接口都需要身份认证，所以接下来我们就通过实现“发布文章”函数来实际使用 auth.js。</p>
<h4 data-nodeid="87216" class="">发布文章</h4>




<p data-nodeid="62789">发布文章的接口定义如下。</p>
<ul data-nodeid="62790">
<li data-nodeid="62791">
<p data-nodeid="62792">请求方法：POST。</p>
</li>
<li data-nodeid="62793">
<p data-nodeid="62794">Path：<code data-backticks="1" data-nodeid="63006">/article/create</code></p>
</li>
<li data-nodeid="62795">
<p data-nodeid="62796">Headers 参数: Authorization token。</p>
</li>
<li data-nodeid="62797">
<p data-nodeid="62798">Body 参数：title、content。</p>
</li>
</ul>
<p data-nodeid="62799">由于登录后才能发布文章，所以要先通过登录接口获取 token，然后调用 <code data-backticks="1" data-nodeid="63010">/article/create</code> 接口时，再将 token 放在 HTTP Headers 参数中。发布文章的代码实现如下：</p>
<pre class="lang-javascript" data-nodeid="62800"><code data-language="javascript"><span class="hljs-comment">// src/function/article/auth</span>
<span class="hljs-keyword">const</span> uuid = <span class="hljs-built_in">require</span>(<span class="hljs-string">"uuid"</span>);
<span class="hljs-keyword">const</span> auth = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../middleware/auth"</span>);
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 创建文章
 * @param {string} username 用户名
 * @param {string} title 文章标题
 * @param {string} content 文章内容
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">createArticle</span>(<span class="hljs-params">username, title, content</span>) </span>{
  <span class="hljs-keyword">const</span> article_id = uuid.v4();
  <span class="hljs-keyword">const</span> now = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Date</span>().toLocaleString();
  <span class="hljs-keyword">await</span> client.createRow(
    <span class="hljs-string">"article"</span>,
    {
      article_id,
    },
    {
      username,
      title,
      content,
      <span class="hljs-attr">create_date</span>: now,
      <span class="hljs-attr">update_date</span>: now,
    }
  );
  <span class="hljs-keyword">return</span> article_id;
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 身份认证</span>
  <span class="hljs-keyword">const</span> user = auth(event);
  <span class="hljs-keyword">if</span> (!user) {
    <span class="hljs-comment">// 若认证失败则直接返回</span>
    <span class="hljs-keyword">return</span> callback(<span class="hljs-string">"身份认证失败"</span>);
  }
  <span class="hljs-comment">// 从 user 中获取 username</span>
  <span class="hljs-keyword">const</span> { username } = user;
  <span class="hljs-keyword">const</span> body = <span class="hljs-built_in">JSON</span>.parse(<span class="hljs-built_in">JSON</span>.parse(event.toString()).body);
  <span class="hljs-keyword">const</span> { title, content } = body;
  createArticle(username, title, content)
    .then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span>
      callback(<span class="hljs-literal">null</span>, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
      })
    )
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">message</span>: <span class="hljs-string">"创建文章失败"</span>,
      })
    );
};
</code></pre>
<p data-nodeid="88044">首先是使用 auth.js 进行身份认证，认证通过后就可以从 user 中获取 username。然后再从请求体中获取文章标题和文章内容数据，将其存入数据库。</p>
<p data-nodeid="88045">接下来我们依旧可以将函数部署和使用 curl 进行测试：</p>

<pre class="lang-java" data-nodeid="62802"><code data-language="java">$ curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/article/create \</span>
-X POST \
-d <span class="hljs-string">"title=这是文章标题&amp;content=内容内容内容......"</span> \
-H <span class="hljs-string">"Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkphY2siLCJpYXQiOjE2MTE0OTI2ODF9.c56Xm4RBLYl5yVtR_Vk0IZOL0yijofcyE-P7vjKf4nA"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>,<span class="hljs-string">"data"</span>:{<span class="hljs-string">"article_id"</span>:<span class="hljs-string">"d4b9bad8-a0ed-499d-b3c6-c57f16eaa193"</span>}}
</code></pre>
<p data-nodeid="89694">在测试时，我们需要将 token 放在 HTTP 请求头的 Authorization 属性中。文章发布成功后，你就可以在表格存储中看到对应的数据了。</p>
<p data-nodeid="89695" class=""><img src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxceARRiPAACAwtaSp94526.png" alt="Drawing 3.png" data-nodeid="89699"></p>


<h4 data-nodeid="93007" class="">查询文章</h4>




<p data-nodeid="62805">发布文章的接口开发完成后，我们继续开发一个查询文章的接口，这样就可以查询出刚才创建的文章。查询文章接口定义如下。</p>
<ul data-nodeid="62806">
<li data-nodeid="62807">
<p data-nodeid="62808">请求方法：GET。</p>
</li>
<li data-nodeid="62809">
<p data-nodeid="62810">Path：<code data-backticks="1" data-nodeid="63027">/article/detail/[article_id]</code></p>
</li>
<li data-nodeid="62811">
<p data-nodeid="62812">Headers 参数: Authorization token。</p>
</li>
</ul>
<p data-nodeid="62813">在查询文章接口中，我们需要在 Path 中定义文章 ID 参数，即 article_id。这样在函数代码中，你就可以通过 event 对象的 pathParameters 中获取 article_id 参数，然后根据 article_id 来查询文章详情了。完整代码如下：</p>
<pre class="lang-javascript" data-nodeid="62814"><code data-language="javascript"><span class="hljs-keyword">const</span> uuid = <span class="hljs-built_in">require</span>(<span class="hljs-string">"uuid"</span>);
<span class="hljs-keyword">const</span> auth = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../middleware/auth"</span>);
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 获取文章详情
 * @param {string} title 文章 ID
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getArticle</span>(<span class="hljs-params">article_id</span>) </span>{
  <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> client.getRow(
    <span class="hljs-string">"article"</span>,
    {
      article_id,
    },
  );
  <span class="hljs-keyword">return</span> res;
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 身份认证</span>
  <span class="hljs-keyword">const</span> user = auth(event);
  <span class="hljs-keyword">if</span> (!user) {
    <span class="hljs-comment">// 若认证失败则直接返回</span>
    <span class="hljs-keyword">return</span> callback(<span class="hljs-string">"身份认证失败"</span>);
  }
  
  <span class="hljs-comment">// 从 event 对象中获取文章 ID</span>
  <span class="hljs-keyword">const</span> article_id = <span class="hljs-built_in">JSON</span>.parse(event.toString()).pathParameters[<span class="hljs-string">'article_id'</span>];
  getArticle(article_id)
    .then(<span class="hljs-function">(<span class="hljs-params">detail</span>) =&gt;</span>
      callback(<span class="hljs-literal">null</span>, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
        <span class="hljs-attr">data</span>: detail
      })
    )
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">message</span>: <span class="hljs-string">"创建文章失败"</span>,
      })
    );
};
</code></pre>
<p data-nodeid="62815">开发完成后，我们可以将其部署到函数计算，再用 curl 命令进行测试：</p>
<pre class="lang-java" data-nodeid="62816"><code data-language="java">$ curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/article/detail/d4b9bad8-a0ed-499d-b3c6-c57f16eaa193 \</span>
-H <span class="hljs-string">"Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkphY2siLCJpYXQiOjE2MTE0OTI2ODF9.c56Xm4RBLYl5yVtR_Vk0IZOL0yijofcyE-P7vjKf4nA"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>,<span class="hljs-string">"data"</span>:{<span class="hljs-string">"article_id"</span>:<span class="hljs-string">"d4b9bad8-a0ed-499d-b3c6-c57f16eaa193"</span>,<span class="hljs-string">"content"</span>:<span class="hljs-string">"内容内容内容......"</span>,<span class="hljs-string">"create_date"</span>:<span class="hljs-string">"1/24/2021, 2:05:46 PM"</span>,<span class="hljs-string">"title"</span>:<span class="hljs-string">"这是文章标题"</span>,<span class="hljs-string">"update_date"</span>:<span class="hljs-string">"1/24/2021, 2:05:46 PM"</span>,<span class="hljs-string">"username"</span>:<span class="hljs-string">"Jack"</span>}}
</code></pre>
<p data-nodeid="62817">如上所示，查询文章的接口按照预期返回了文章详情。</p>
<h4 data-nodeid="96292" class="">更新文章</h4>




<p data-nodeid="62819">更新文章的 API Path 参数和查询文章一样，都需要 Path 中定义 article_id。而其 body 参数则与创建文章相同。此外，更新文章的请求 method 是 PUT，因为在 Restful API 规范中，我们通常使用 POST 来表示创建， 使用 PUT 来表示更新。</p>
<p data-nodeid="62820">更新文章的接口定义如下。</p>
<ul data-nodeid="62821">
<li data-nodeid="62822">
<p data-nodeid="62823">请求方法：PUT。</p>
</li>
<li data-nodeid="62824">
<p data-nodeid="62825">Path：<code data-backticks="1" data-nodeid="63048">/article/update/[article_id]</code></p>
</li>
<li data-nodeid="62826">
<p data-nodeid="62827">Headers 参数: Authorization token。</p>
</li>
<li data-nodeid="62828">
<p data-nodeid="62829">Body 参数：title、content。</p>
</li>
</ul>
<p data-nodeid="62830">更新文章的逻辑就是根据 &nbsp;article_id 去更新一行数据。代码如下：</p>
<pre class="lang-javascript" data-nodeid="62831"><code data-language="javascript"><span class="hljs-keyword">const</span> auth = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../middleware/auth"</span>);
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 更新文章
 * @param {string} article_id 待更新的文章 ID
 * @param {string} title 文章标题
 * @param {string} content 文章内容
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">updateArticle</span>(<span class="hljs-params">article_id, title, content</span>) </span>{
  <span class="hljs-keyword">const</span> now = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Date</span>().toLocaleString();
  <span class="hljs-keyword">await</span> client.updateRow(
    <span class="hljs-string">"article"</span>,
    {
      article_id,
    },
    {
      title,
      content,
      <span class="hljs-attr">update_date</span>: now,
    }
  );
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 身份认证</span>
  <span class="hljs-keyword">const</span> user = auth(event);
  <span class="hljs-keyword">if</span> (!user) {
    <span class="hljs-comment">// 若认证失败则直接返回</span>
    <span class="hljs-keyword">return</span> callback(<span class="hljs-string">"身份认证失败"</span>);
  }
  <span class="hljs-keyword">const</span> eventObject = <span class="hljs-built_in">JSON</span>.parse(event.toString())
  <span class="hljs-comment">// 从 event 对象的 pathParameters 中获取 Path 参数</span>
  <span class="hljs-keyword">const</span> article_id = eventObject.pathParameters[<span class="hljs-string">'article_id'</span>];
  <span class="hljs-keyword">const</span> body = <span class="hljs-built_in">JSON</span>.parse(eventObject.body);
  <span class="hljs-comment">// 从 event 对象的 body 中获取请求体参数</span>
  <span class="hljs-keyword">const</span> { title, content } = body;
  updateArticle(article_id, title, content)
    .then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span>
      callback(<span class="hljs-literal">null</span>, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
      })
    )
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">message</span>: <span class="hljs-string">"更新文章失败"</span>,
      })
    );
};
</code></pre>
<p data-nodeid="62832">开发并部署完成后，使用 curl 命令进行测试：</p>
<pre class="lang-java" data-nodeid="62833"><code data-language="java">$ curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/article/update/d4b9bad8-a0ed-499d-b3c6-c57f16eaa193 \</span>
-X PUT \
-d <span class="hljs-string">"title=这是文章标题&amp;content=更新的内容......"</span> \
-H <span class="hljs-string">"Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkphY2siLCJpYXQiOjE2MTE0OTI2ODF9.c56Xm4RBLYl5yVtR_Vk0IZOL0yijofcyE-P7vjKf4nA"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>}
</code></pre>
<p data-nodeid="62834">返回 <code data-backticks="1" data-nodeid="63056">{"success":true}</code> 则说明更新成功。</p>
<h4 data-nodeid="99553" class="">删除文章</h4>




<p data-nodeid="62836">最后就还是一个删除文章的 API 了。删除文章的 API 也需要在 Path 中定义 article_id 参数，并且其 HTTP method 是 DELETE。具体接口定义如下。</p>
<ul data-nodeid="62837">
<li data-nodeid="62838">
<p data-nodeid="62839">请求方法：DELETE。</p>
</li>
<li data-nodeid="62840">
<p data-nodeid="62841">Path：<code data-backticks="1" data-nodeid="63067">/article/delete/[article_id]</code></p>
</li>
<li data-nodeid="62842">
<p data-nodeid="62843">Headers 参数: Authorization token，</p>
</li>
</ul>
<p data-nodeid="62844">删除文章很简单，就是根据 article_id 删除一行数据，代码如下：</p>
<pre class="lang-javascript" data-nodeid="62845"><code data-language="javascript"><span class="hljs-keyword">const</span> uuid = <span class="hljs-built_in">require</span>(<span class="hljs-string">"uuid"</span>);
<span class="hljs-keyword">const</span> auth = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../middleware/auth"</span>);
<span class="hljs-keyword">const</span> client = <span class="hljs-built_in">require</span>(<span class="hljs-string">"../../db/client"</span>);
<span class="hljs-comment">/**
 * 删除文章
 * @param {string} title 文章 ID
 */</span>
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">deleteArticle</span>(<span class="hljs-params">article_id</span>) </span>{
  <span class="hljs-keyword">const</span> res = <span class="hljs-keyword">await</span> client.deleteRow(
    <span class="hljs-string">"article"</span>,
    {
      article_id,
    },
  );
  <span class="hljs-keyword">return</span> res;
}
<span class="hljs-built_in">module</span>.exports.handler = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">event, context, callback</span>) </span>{
  <span class="hljs-comment">// 身份认证</span>
  <span class="hljs-keyword">const</span> user = auth(event);
  <span class="hljs-keyword">if</span> (!user) {
    <span class="hljs-comment">// 若认证失败则直接返回</span>
    <span class="hljs-keyword">return</span> callback(<span class="hljs-string">"身份认证失败"</span>);
  }
  
  <span class="hljs-comment">// 从 event 对象中获取文章 ID</span>
  <span class="hljs-keyword">const</span> article_id = <span class="hljs-built_in">JSON</span>.parse(event.toString()).pathParameters[<span class="hljs-string">'article_id'</span>];
  deleteArticle(article_id)
    .then(<span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span>
      callback(<span class="hljs-literal">null</span>, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">true</span>,
      })
    )
    .catch(<span class="hljs-function">(<span class="hljs-params">error</span>) =&gt;</span>
      callback(error, {
        <span class="hljs-attr">success</span>: <span class="hljs-literal">false</span>,
        <span class="hljs-attr">message</span>: <span class="hljs-string">"删除文章失败"</span>,
      })
    );
};
</code></pre>
<p data-nodeid="62846">同样我们可以通过 curl 命令进行测试：</p>
<pre class="lang-java" data-nodeid="62847"><code data-language="java">curl http:<span class="hljs-comment">//a88f7e84f71749958100997b77b3e2f6-cn-beijing.alicloudapi.com/article/delete/d4b9bad8-a0ed-499d-b3c6-c57f16eaa193 \</span>
-X DELETE \
-H <span class="hljs-string">"Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6IkphY2siLCJpYXQiOjE2MTE0OTI2ODF9.c56Xm4RBLYl5yVtR_Vk0IZOL0yijofcyE-P7vjKf4nA"</span>
{<span class="hljs-string">"success"</span>:<span class="hljs-keyword">true</span>}
</code></pre>
<p data-nodeid="62848">删除成功后，再去表格存储中就找不到这行记录了。至此，内容管理系统的 Restful API 就开发完毕了。</p>
<h3 data-nodeid="102790" class="te-preview-highlight">总结</h3>




<p data-nodeid="62850">可以看到，基于 Serverless 开发 Restful API 的整个代码非常简单，每个函数只负责一个独立的业务，职责单一、逻辑清晰。关于这一讲，我想强调这样几个重点：</p>
<ul data-nodeid="62851">
<li data-nodeid="62852">
<p data-nodeid="62853">基于 Serverless 开发 API 时，建议你使用 API 网关进行 API 的管理；</p>
</li>
<li data-nodeid="62854">
<p data-nodeid="62855">对于数据库等第三方服务，建议对其基本操作进行封装，这样更方便进行扩展；</p>
</li>
<li data-nodeid="62856">
<p data-nodeid="62857">Serverless 函数需要保持简单、独立、单一职责。</p>
</li>
</ul>
<p data-nodeid="62858">最后，我留给你的作业就是，亲自动手实现一个基于 Serverless 的具有 Restful API 的内容管理系统。我们下一讲见。</p>

---

### 精选评论

##### **栋：
> 修改代码之后, 如何进行升级发布呢? 我继续用fun deploy 返回下列错误PUT /services/serverless-cms failed with 400. requestid: 8f9453e8-86bc-42da-8f53-98c7f7bfe74f, message: Both project and logstore are required for enabling request metrics.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 您好，这个报错可能是您之前在控制台中为函数服务（Service）配置了日志服务，但项目的 template.yaml 中没有配置日志服务。可以参考这个文档，为服务添加 LogConfig 配置 https://github.com/alibaba/funcraft/blob/master/docs/specs/2018-04-03-zh-cn.md#aliyunserverlessservice

