<p data-nodeid="1077" class="">今天我想和你聊一聊怎么用 Serverless 开发一个服务端渲染（SSR）应用。</p>
<p data-nodeid="1078">对前端工程师来说，Serverless 最大的应用场景之一就是开发服务端渲染（SSR）应用。因为传统的服务端渲染应用要由前端工程师负责服务器的运维，但往往前端工程师并不擅长这一点，基于 Serverless 开发服务端渲染应用的话，就可以减轻这个负担。希望你学完今天的内容之后，能够学会如何去使用 Serverless 开发一个服务端渲染应用。</p>
<p data-nodeid="1079">话不多说，我们开始今天的学习。</p>
<h3 data-nodeid="1080">基于 Serverless 的服务端渲染架构</h3>
<p data-nodeid="1081">现在的主流前端框架是 React.js、Vue.js 等，基于这些框架开发的都是单页应用，其渲染方式都是客户端渲染：代码开发完成后，构建出一个或多个 JS 资源，页面渲染时加载这些 JS 资源，然后再执行 JS 渲染页面。虽然这些框架可以极大提升前端开发效率，但也带来了一些新的问题。</p>
<ul data-nodeid="1082">
<li data-nodeid="1083">
<p data-nodeid="1084"><strong data-nodeid="1219">不利于 SEO：</strong> 页面源码不再是HTML，而是渲染 HTML 的 JavaScript，这就导致搜索引擎爬虫难以解析其中的内容；</p>
</li>
<li data-nodeid="1085">
<p data-nodeid="1086"><strong data-nodeid="1224">初始化性能差：</strong> 通常单元应用的 JS 文件体积都比较大、加载耗时比较长，导致页面白屏。</p>
</li>
</ul>
<p data-nodeid="1087">为了解决这些问题，很多框架和开发者就开始尝试服务端渲染的方式：页面加载时，由服务端先生成 HTML 返回给浏览器，浏览器直接渲染 HTML。在传统的服务端渲染架构中，通常需要前端同学使用 Node.js 去实现一个服务端的渲染应用。在应用内，每个请求的 path 对应着服务端的每个路由，由该路由实现对应 path 的 HTML 文档渲染：</p>
<p data-nodeid="1088"><img src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxlCABEjhAAGFmUrRE68475.png" alt="Drawing 0.png" data-nodeid="1228"></p>
<div data-nodeid="1089"><p style="text-align:center">传统服务端渲染架构</p></div>
<p data-nodeid="1090">对前端工程师来说，要实现一个服务端渲染应用，通常面临着一些问题：</p>
<ul data-nodeid="1091">
<li data-nodeid="1092">
<p data-nodeid="1093">部署服务端渲染应用需要购买服务器，并配置服务器环境，要对服务器进行运维；</p>
</li>
<li data-nodeid="1094">
<p data-nodeid="1095">需要关注业务量，考虑有没有高并发场景、服务器有没有扩容机制；</p>
</li>
<li data-nodeid="1096">
<p data-nodeid="1097">需要实现负载均衡、流量控制等复杂后端能力等。</p>
</li>
</ul>
<p data-nodeid="1098">开篇我也提到，而且是服务端的工作，很多前端同学都不擅长，好在有了 Serverless。</p>
<p data-nodeid="1099">用 Serverless 做服务端渲染，就是将以往的每个路由，都拆分为一个个函数，再在 FaaS 上部署对应的函数，这样用户请求的 path，对应的就是每个单独的函数。通过这种方式，就将运维操作转移到了 FaaS 平台，前端同学开发服务端渲染应用，就再也不用关心服务端程序的运维部署了。并且在 FaaS 平台中运行的函数，天然具有弹性伸缩的能力，你也不用担心流量波峰波谷了。</p>
<p data-nodeid="1100"><img src="https://s0.lgstatic.com/i/image/M00/94/46/CgqCHmAXxluARkcHAAF-S8PNwUE730.png" alt="Drawing 1.png" data-nodeid="1237"></p>
<div data-nodeid="1101"><p style="text-align:center">基于 Serverless 的服务选渲染架构</p></div>
<p data-nodeid="1102">如图所示，FaaS 函数接收请求后直接执行代码渲染出 HTML 并返回给浏览器，这是最基本的架构，虽然它可以满足大部分场景，但要追求极致的性能，你通常要加入缓存。</p>
<p data-nodeid="1103"><img src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxmOATGYqAAGjC4CgYTw981.png" alt="Drawing 2.png" data-nodeid="1241"></p>
<div data-nodeid="1104"><p style="text-align:center">进阶版基于 Serverless 的服务端渲染架构</p></div>
<p data-nodeid="1105">首先我们会使用 CDN 做缓存，基于 CDN 的缓存可以减少函数执行次数，进而避免函数冷启动带来的性能损耗。如果 CDN 中没有 SSR HTML 页面的缓存，则继续由网关处理请求，网关再去触发函数执行。</p>
<p data-nodeid="1106">函数首先会判读缓存数据库中是否有 SSR HTML 的缓存，如果有直接返回；如果没有再渲染出 HTML 并返回。基于数据库的缓存，可以减少函数渲染 HTML 的时间，从而页面加载提升性能。</p>
<p data-nodeid="1107">讲了这么多，具体怎么基于 Serverless 实现一个服务端渲染应用呢？</p>
<h3 data-nodeid="1108">实现一个 Serverless 的服务端渲染应用</h3>
<p data-nodeid="1109">在 16 讲中，我们实现了一个内容管理系统的 Restful API，但没有前端界面，所以今天我们的目标就基于 Serverless 实现一个内容管理系统的前端界面（如图所示）。</p>
<p data-nodeid="1110"><img src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXxoiANTROADtU9yybMQY209.gif" alt="ssr.gif" data-nodeid="1249"></p>
<p data-nodeid="1111">该应用主要包含两个页面：</p>
<ul data-nodeid="1112">
<li data-nodeid="1113">
<p data-nodeid="1114">首页，展示文章列表；</p>
</li>
<li data-nodeid="1115">
<p data-nodeid="1116">详情页，展示文章详情。</p>
</li>
</ul>
<p data-nodeid="1117">为了方便你进行实践，我为你提供了一份示例代码，你可以直接下载并使用：</p>
<pre class="lang-java" data-nodeid="1118"><code data-language="java"># 下载代码
$ git clone https://github.com/nodejh/serverless-class
# 进入服务端渲染应用目录
$ cd 16/serverless-ssr-cms
</code></pre>
<p data-nodeid="1119">代码结构如下：</p>
<pre class="lang-java" data-nodeid="1120"><code data-language="java">.
├── config.js
├── f.yml
├── <span class="hljs-keyword">package</span>-lock.json
├── <span class="hljs-keyword">package</span>.json
├── src
│&nbsp; &nbsp;├── api.ts
│&nbsp; &nbsp;├── config
│&nbsp; &nbsp;│&nbsp; &nbsp;└── config.<span class="hljs-keyword">default</span>.ts
│&nbsp; &nbsp;├── configuration.ts
│&nbsp; &nbsp;├── index.ts
│&nbsp; &nbsp;├── <span class="hljs-class"><span class="hljs-keyword">interface</span>
│&nbsp; &nbsp;│&nbsp; &nbsp;├── <span class="hljs-title">detail</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp;│&nbsp; &nbsp;└── <span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp;├── <span class="hljs-title">mock</span>
│&nbsp; &nbsp;│&nbsp; &nbsp;├── <span class="hljs-title">detail</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp;│&nbsp; &nbsp;└── <span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp;├── <span class="hljs-title">render</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp;└── <span class="hljs-title">service</span>
│&nbsp; &nbsp; &nbsp; &nbsp;├── <span class="hljs-title">detail</span>.<span class="hljs-title">ts</span>
│&nbsp; &nbsp; &nbsp; &nbsp;└── <span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
├── <span class="hljs-title">tsconfig</span>.<span class="hljs-title">json</span>
├── <span class="hljs-title">tsconfig</span>.<span class="hljs-title">lint</span>.<span class="hljs-title">json</span>
└── <span class="hljs-title">web</span>
&nbsp; &nbsp; ├── @<span class="hljs-title">types</span>
&nbsp; &nbsp; │&nbsp; &nbsp;└── <span class="hljs-title">global</span>.<span class="hljs-title">d</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; ├── <span class="hljs-title">common</span>.<span class="hljs-title">less</span>
&nbsp; &nbsp; ├── <span class="hljs-title">components</span>
&nbsp; &nbsp; │&nbsp; &nbsp;├── <span class="hljs-title">layout</span>
&nbsp; &nbsp; │&nbsp; &nbsp;│&nbsp; &nbsp;├── <span class="hljs-title">index</span>.<span class="hljs-title">less</span>
&nbsp; &nbsp; │&nbsp; &nbsp;│&nbsp; &nbsp;└── <span class="hljs-title">index</span>.<span class="hljs-title">tsx</span>
&nbsp; &nbsp; │&nbsp; &nbsp;└── <span class="hljs-title">title</span>
&nbsp; &nbsp; │&nbsp; &nbsp; &nbsp; &nbsp;├── <span class="hljs-title">index</span>.<span class="hljs-title">less</span>
&nbsp; &nbsp; │&nbsp; &nbsp; &nbsp; &nbsp;└── <span class="hljs-title">index</span>.<span class="hljs-title">tsx</span>
&nbsp; &nbsp; ├── <span class="hljs-title">interface</span>
&nbsp; &nbsp; │&nbsp; &nbsp;├── <span class="hljs-title">detail</span>-<span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; │&nbsp; &nbsp;├── <span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; │&nbsp; &nbsp;└── <span class="hljs-title">page</span>-<span class="hljs-title">index</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; ├── <span class="hljs-title">pages</span>
&nbsp; &nbsp; │&nbsp; &nbsp;├── <span class="hljs-title">detail</span>
&nbsp; &nbsp; │&nbsp; &nbsp;│&nbsp; &nbsp;├── <span class="hljs-title">fetch</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; │&nbsp; &nbsp;│&nbsp; &nbsp;├── <span class="hljs-title">index</span>.<span class="hljs-title">less</span>
&nbsp; &nbsp; │&nbsp; &nbsp;│&nbsp; &nbsp;└── <span class="hljs-title">render</span>$<span class="hljs-title">id</span>.<span class="hljs-title">tsx</span>
&nbsp; &nbsp; │&nbsp; &nbsp;└── <span class="hljs-title">index</span>
&nbsp; &nbsp; │&nbsp; &nbsp; &nbsp; &nbsp;├── <span class="hljs-title">fetch</span>.<span class="hljs-title">ts</span>
&nbsp; &nbsp; │&nbsp; &nbsp; &nbsp; &nbsp;├── <span class="hljs-title">index</span>.<span class="hljs-title">less</span>
&nbsp; &nbsp; │&nbsp; &nbsp; &nbsp; &nbsp;└── <span class="hljs-title">render</span>.<span class="hljs-title">tsx</span>
    └── <span class="hljs-title">tsconfig</span>.<span class="hljs-title">json</span>
</span></code></pre>
<p data-nodeid="1121">文件很多，不过不用担心，你只需重点关注 web/pages/ 和 src/service 两个目录：</p>
<ul data-nodeid="1122">
<li data-nodeid="1123">
<p data-nodeid="1124">web/ 目录中主要是前端页面的代码， web/pages/ 中的文件分别对应着我们要实现的 index（首页）和 detail（详情页）两个页面，这两个页面会使用到 components 目录中的公共组件；</p>
</li>
<li data-nodeid="1125">
<p data-nodeid="1126">src/ 目录中主要是后端代码，src/service 目录中的 index.ts &nbsp;和 detail.ts 则定义了两个页面分别需要用到的接口，为了简单起见，接口数据我使用了 src/mock/ 目录中的 mock 数据。</p>
</li>
</ul>
<p data-nodeid="1127">当我一个人又负责前端页面也负责后端接口的开发时，通常习惯先实现接口，再开发前端页面，这样方便调试。接下来就让我们看一下具体是怎么实现的。</p>
<h4 data-nodeid="1128">首页接口的实现</h4>
<p data-nodeid="1129">其源码在 src/service/index.ts 文件中，代码如下：</p>
<pre class="lang-typescript" data-nodeid="1130"><code data-language="typescript"><span class="hljs-comment">// src/service/index.ts</span>
<span class="hljs-keyword">import</span> { provide } <span class="hljs-keyword">from</span> <span class="hljs-string">'@midwayjs/faas'</span>
<span class="hljs-keyword">import</span> { IApiService } <span class="hljs-keyword">from</span> <span class="hljs-string">'../interface'</span>
<span class="hljs-keyword">import</span> mock <span class="hljs-keyword">from</span> <span class="hljs-string">'../mock'</span>
<span class="hljs-meta">@provide</span>(<span class="hljs-string">'ApiService'</span>)
<span class="hljs-keyword">export</span> <span class="hljs-keyword">class</span> ApiService <span class="hljs-keyword">implements</span> IApiService {
  <span class="hljs-keyword">async</span> index (): <span class="hljs-built_in">Promise</span>&lt;<span class="hljs-built_in">any</span>&gt; {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">await</span> <span class="hljs-built_in">Promise</span>.resolve(mock)
  }
}
</code></pre>
<p data-nodeid="1131">这段代码实现了一个 ApiService 类以及 index 方法，该方法会返回首页的文章列表。数据结构如下：</p>
<pre class="lang-json" data-nodeid="1132"><code data-language="json">{
&nbsp; &nbsp; <span class="hljs-attr">"data"</span>:[
&nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"id"</span>:<span class="hljs-string">"3f8a198c-60a2-11eb-8932-9b95cd7afc2d"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"title"</span>:<span class="hljs-string">"开篇词：Serverless 大热，程序员面临的新机遇与挑战"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"content"</span>:<span class="hljs-string">"可能你会认为 Serverless 是最近两年兴起的技术......"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"date"</span>:<span class="hljs-string">"2020-12-23"</span>
&nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"id"</span>:<span class="hljs-string">"5158b100-5fee-11eb-9afa-9b5f85523067"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"title"</span>:<span class="hljs-string">"基础入门：编写你的第一个 Serverless 应用"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"content"</span>:<span class="hljs-string">"学习一门新技术，除了了解其基础概念，更重要的是把理论转化为实践..."</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"date"</span>:<span class="hljs-string">"2020-12-29"</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; ]
}
</code></pre>
<p data-nodeid="1133">在进行服务端渲染时，你可以通过 ctx 获取到 ApiService 实例，进而调用其中的方法，获取文章列表数据。此外，ApiService 也会被 src/api.ts 调用，src/api.ts 则直接对外提供了 HTTP 接口。</p>
<h4 data-nodeid="1134">首页页面的实现</h4>
<p data-nodeid="1135">有了接口后，我们就可以继续实现首页的前端页面了。首页页面的代码在 web/pages/ 目录中，该目录下有三个文件：</p>
<ul data-nodeid="1136">
<li data-nodeid="1137">
<p data-nodeid="1138">fetch.ts，获取首页数据；</p>
</li>
<li data-nodeid="1139">
<p data-nodeid="1140">render.tsx 首页页面 UI 组件代码；</p>
</li>
<li data-nodeid="1141">
<p data-nodeid="1142">index.less 样式代码。</p>
</li>
</ul>
<p data-nodeid="1143">首先来看一下 fetch.ts：</p>
<pre class="lang-typescript" data-nodeid="1144"><code data-language="typescript"><span class="hljs-comment">// web/pages/index/fetch.ts</span>
<span class="hljs-keyword">import</span> { IFaaSContext } <span class="hljs-keyword">from</span> <span class="hljs-string">'ssr-types'</span>
<span class="hljs-keyword">import</span> { IndexData } <span class="hljs-keyword">from</span> <span class="hljs-string">'@/interface'</span>
<span class="hljs-keyword">interface</span> IApiService {
  index: <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-built_in">Promise</span>&lt;IndexData&gt;
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">async</span> (ctx: IFaaSContext&lt;{
  apiService?: IApiService
}&gt;) =&gt; {
  <span class="hljs-keyword">const</span> data = __isBrowser__ ? <span class="hljs-keyword">await</span> (<span class="hljs-keyword">await</span> <span class="hljs-built_in">window</span>.fetch(<span class="hljs-string">'/api/index'</span>)).json() : <span class="hljs-keyword">await</span> ctx.apiService?.index()
  <span class="hljs-keyword">return</span> {
    indexData: data
  }
}
</code></pre>
<p data-nodeid="1145"><strong data-nodeid="1277">这段代码的逻辑比较简单，核心点在第 10 行</strong>，如果是浏览器，就用浏览器自带的 fetch 方法请求<code data-backticks="1" data-nodeid="1273">/api/index</code>接口获取数据；如果不是浏览器，即服务端渲染，可以直接调用 apiService 中的 index 方法。获取到数据后，将其存入 state.indexData 中，这样在 UI 组件中就可以使用了。<br>
首页的 UI 组件 render.tsx 代码如下：</p>
<pre class="lang-javascript" data-nodeid="1146"><code data-language="javascript"><span class="hljs-comment">// web/pages/index/render.tsx</span>
<span class="hljs-keyword">import</span> React, { useContext } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> { SProps, IContext } <span class="hljs-keyword">from</span> <span class="hljs-string">"ssr-types"</span>;
<span class="hljs-keyword">import</span> Navbar <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/navbar"</span>;
<span class="hljs-keyword">import</span> Header <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/header"</span>;
<span class="hljs-keyword">import</span> Item <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/item"</span>;
<span class="hljs-keyword">import</span> { IData } <span class="hljs-keyword">from</span> <span class="hljs-string">"@/interface"</span>;
<span class="hljs-keyword">import</span> styles <span class="hljs-keyword">from</span> <span class="hljs-string">"./index.less"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> (props: SProps) =&gt; {
  <span class="hljs-keyword">const</span> { state } = useContext&lt;IContext&lt;IData&gt;&gt;(<span class="hljs-built_in">window</span>.STORE_CONTEXT);
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">Navbar</span> {<span class="hljs-attr">...props</span>} <span class="hljs-attr">isHomePage</span>=<span class="hljs-string">{true}</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Navbar</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">Header</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Header</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">{styles.container}</span>&gt;</span>
        {state?.indexData?.data.map((item) =&gt; (
          <span class="hljs-tag">&lt;<span class="hljs-name">Item</span>
            {<span class="hljs-attr">...props</span>}
            <span class="hljs-attr">id</span>=<span class="hljs-string">{item.id}</span>
            <span class="hljs-attr">key</span>=<span class="hljs-string">{item.id}</span>
            <span class="hljs-attr">title</span>=<span class="hljs-string">{item.title}</span>
            <span class="hljs-attr">content</span>=<span class="hljs-string">{item.content}</span>
            <span class="hljs-attr">date</span>=<span class="hljs-string">{item.date}</span>
          &gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Item</span>&gt;</span>
        ))}
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
};
</code></pre>
<p data-nodeid="1147">在 UI 组件中，我们可以通过 useContext 获取刚才由 fetch.ts 存入 state 的数据，然后利用数据渲染 UI。UI 组件主要由三部分组成。</p>
<ul data-nodeid="1148">
<li data-nodeid="1149">
<p data-nodeid="1150">Navbar：导航条。</p>
</li>
<li data-nodeid="1151">
<p data-nodeid="1152">Header：页面标题。</p>
</li>
<li data-nodeid="1153">
<p data-nodeid="1154">Item：每篇文章的简介。</p>
</li>
</ul>
<p data-nodeid="1155"><img src="https://s0.lgstatic.com/i/image/M00/94/46/CgqCHmAXx1WAV0BxAAM5rls-jc4377.png" alt="Drawing 4.png" data-nodeid="1284"></p>
<h4 data-nodeid="1156">详情页接口的实现</h4>
<p data-nodeid="1157">完成了首页后，就可以实现详情页了。详情页与首页整体类似，区别就在于详情页需要传入参数查询某条数据。</p>
<p data-nodeid="1158">详情页接口在 src/service/detail.ts 中 ，代码如下所示：</p>
<pre class="lang-typescript" data-nodeid="1159"><code data-language="typescript"><span class="hljs-comment">// src/service/detail.ts</span>
<span class="hljs-keyword">import</span> { provide } <span class="hljs-keyword">from</span> <span class="hljs-string">'@midwayjs/faas'</span>
<span class="hljs-keyword">import</span> { IApiDetailService } <span class="hljs-keyword">from</span> <span class="hljs-string">'../interface/detail'</span>
<span class="hljs-keyword">import</span> mock <span class="hljs-keyword">from</span> <span class="hljs-string">'../mock/detail'</span>
<span class="hljs-meta">@provide</span>(<span class="hljs-string">'ApiDetailService'</span>)
<span class="hljs-keyword">export</span> <span class="hljs-keyword">class</span> ApiDetailService <span class="hljs-keyword">implements</span> IApiDetailService {
  <span class="hljs-keyword">async</span> index (id): <span class="hljs-built_in">Promise</span>&lt;<span class="hljs-built_in">any</span>&gt; {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">await</span> <span class="hljs-built_in">Promise</span>.resolve(mock.data[id])
  }
}
</code></pre>
<p data-nodeid="1160">在这段代码中，我们实现了一个 ApiDetailService 类以及 index 方法，index 方法的如参 id 即文章 ID，然后根据文章 ID 从 mock 数据中查询文章详情。</p>
<p data-nodeid="1161">文章详情数据如下：</p>
<pre class="lang-json" data-nodeid="1162"><code data-language="json">{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">"title"</span>:<span class="hljs-string">"Serverless&nbsp;大热，程序员面临的新机遇与挑战"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">"wordCount"</span>:<span class="hljs-number">2540</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">"readingTime"</span>:<span class="hljs-number">10</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">"date"</span>:<span class="hljs-string">"2020-12-23&nbsp;12:00:00"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">"content"</span>:<span class="hljs-string">"可能你会认为&nbsp;Serverless&nbsp;是最近两年兴起的技术，实际上，Serverless&nbsp;概念从&nbsp;2012&nbsp;年就提出来了，随后&nbsp;AWS&nbsp;在&nbsp;2014&nbsp;年推出了第一款&nbsp;Serverless&nbsp;产品&nbsp;Lambda，开启了&nbsp;Serverless&nbsp;元年...&nbsp;"</span>
}
</code></pre>
<h4 data-nodeid="1163">详情页页面的实现</h4>
<p data-nodeid="1164">和首页一样，详情页也包含数据请求、UI 组件和样式代码三个文件。</p>
<p data-nodeid="1165">数据请求代码文件的命名和首页一样，都是 fetch.ts。与首页不同的是，详情页我们需要从上下文（服务端渲染场景）或 URL 中（浏览器场景）获取到文章 ID，然后根据文章 ID 获取文章详情数据。代码如下：</p>
<pre class="lang-typescript" data-nodeid="1166"><code data-language="typescript"><span class="hljs-keyword">import</span> { RouteComponentProps } <span class="hljs-keyword">from</span> <span class="hljs-string">"react-router"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">async</span> (ctx) =&gt; {
  <span class="hljs-keyword">let</span> data;
  <span class="hljs-keyword">if</span> (__isBrowser__) {
    <span class="hljs-keyword">const</span> id = (ctx <span class="hljs-keyword">as</span> RouteComponentProps&lt;{ id: <span class="hljs-built_in">string</span> }&gt;).match.params.id;
    data = <span class="hljs-keyword">await</span> (<span class="hljs-keyword">await</span> <span class="hljs-built_in">window</span>.fetch(<span class="hljs-string">`/api/detail/<span class="hljs-subst">${id}</span>`</span>)).json()
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-keyword">const</span> id = <span class="hljs-regexp">/detail\/(.*)(\?|\/)?/</span>.exec(ctx.req.path)[<span class="hljs-number">1</span>];
    data = <span class="hljs-keyword">await</span> ctx.apiDeatilservice.index(id);
  }

  <span class="hljs-keyword">return</span> {
    detailData: data,
  };
};
</code></pre>
<p data-nodeid="1167">详情页的 UI 组件名称为<code data-backticks="1" data-nodeid="1294">render$id.tsx</code>的文件，<code data-backticks="1" data-nodeid="1296">$id</code>表示该组件的参数是 id，这样访问 /detail/ 这个路由（id 是变量）时，就会匹配到 web/pages/detail/render$id.tsx 这个页面了。</p>
<p data-nodeid="1168"><br>
<code data-backticks="1" data-nodeid="1300">render$id.tsx</code>详细代码如下：</p>
<pre class="lang-javascript" data-nodeid="1169"><code data-language="javascript"><span class="hljs-keyword">import</span> React, { useContext } <span class="hljs-keyword">from</span> <span class="hljs-string">"react"</span>;
<span class="hljs-keyword">import</span> { IContext, SProps } <span class="hljs-keyword">from</span> <span class="hljs-string">"ssr-types"</span>;
<span class="hljs-keyword">import</span> { Data } <span class="hljs-keyword">from</span> <span class="hljs-string">"@/interface"</span>;
<span class="hljs-keyword">import</span> Navbar <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/navbar"</span>;
<span class="hljs-keyword">import</span> Content <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/content"</span>;
<span class="hljs-keyword">import</span> Title <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/title"</span>;
<span class="hljs-keyword">import</span> Tip <span class="hljs-keyword">from</span> <span class="hljs-string">"@/components/tip"</span>;
<span class="hljs-keyword">import</span> styles <span class="hljs-keyword">from</span> <span class="hljs-string">"./index.less"</span>;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> (props: SProps) =&gt; {
  <span class="hljs-keyword">const</span> { state } = useContext&lt;IContext&lt;Data&gt;&gt;(<span class="hljs-built_in">window</span>.STORE_CONTEXT);
  <span class="hljs-keyword">return</span> (
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">Navbar</span> {<span class="hljs-attr">...props</span>}&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">Navbar</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">className</span>=<span class="hljs-string">{styles.container}</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">Title</span>&gt;</span>{state?.detailData?.title}<span class="hljs-tag">&lt;/<span class="hljs-name">Title</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">Tip</span>
          <span class="hljs-attr">date</span>=<span class="hljs-string">{state?.detailData?.date}</span>
          <span class="hljs-attr">wordCount</span>=<span class="hljs-string">{state?.detailData?.wordCount}</span>
          <span class="hljs-attr">readingTime</span>=<span class="hljs-string">{state?.detailData?.readingTime}</span>
        /&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">Content</span>&gt;</span>{state?.detailData?.content}<span class="hljs-tag">&lt;/<span class="hljs-name">Content</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  );
};
</code></pre>
<p data-nodeid="1170">详情页的 UI 组件由四部分组成。</p>
<ul data-nodeid="1171">
<li data-nodeid="1172">
<p data-nodeid="1173">Navbar：导航条。</p>
</li>
<li data-nodeid="1174">
<p data-nodeid="1175">Title：文章标题。</p>
</li>
<li data-nodeid="1176">
<p data-nodeid="1177">Tip：文章发布时间、字数等提示。</p>
</li>
<li data-nodeid="1178">
<p data-nodeid="1179">Content：文章内容。</p>
</li>
</ul>
<p data-nodeid="1180"><img src="https://s0.lgstatic.com/i/image/M00/94/3B/Ciqc1GAXx3GADE4AAAO-vJ2TBH0389.png" alt="Drawing 5.png" data-nodeid="1309"></p>
<h4 data-nodeid="1181">应用部署</h4>
<p data-nodeid="1182">代码开发完成后，你可以通过下面的命令在本地启动应用：</p>
<pre class="lang-java" data-nodeid="1183"><code data-language="java">$ npm start
...
[HPM] Proxy created: /asset-manifest  -&gt; http:<span class="hljs-comment">//127.0.0.1:8000</span>
 Server is listening on http:<span class="hljs-comment">//localhost:3000</span>
</code></pre>
<p data-nodeid="1184">应用启动后就可以打开浏览器输入 <a href="http://localhost:3000" data-nodeid="1315">http://localhost:3000</a> 查看效果了。<br>
在本地开发测试完成后，接下来就需要将其部署到函数计算。你可以运行 npm run deploy 命令进部署：</p>
<pre class="lang-java" data-nodeid="1185"><code data-language="java">$ npm run deploy
...
service  serverless-ssr-cms deploy success
......
The assigned temporary domain is http:<span class="hljs-comment">//41506101-1457216987974698.test.functioncompute.com，expired at 2021-02-04 00:35:01, limited by 1000 per day.</span>
......
Deploy success
</code></pre>
<p data-nodeid="1186"><code data-backticks="1" data-nodeid="1319">npm run deploy</code>其实是执行了构建代码和部署应用两个步骤，这两个步骤都是在本机执行的。但这就存在一个隐藏风险，如果团队同学本地开发环境不同，就可能导致构建产物不同，进而导致部署到线上的代码存在风险。<strong data-nodeid="1324">所以更好的实践是：实现一个业务的持续集成流程，统一构建部署。</strong></p>
<p data-nodeid="1187">应用部署成功后，会自动创建一个测试的域名，例如<a href="http://41506101-1457216987974698.test.functioncompute.com" data-nodeid="1328">http://41506101-1457216987974698.test.functioncompute.com</a>，我们可以打开该域名查看最终效果。</p>
<p data-nodeid="1188">讲到这儿，基于 Serverless 的服务端渲染应用就开发完成了。</p>
<h3 data-nodeid="1189">总结</h3>
<p data-nodeid="2149">总的来说，基于 Serverless 的服务端渲染应用实现也比较简单。如果你想要追求更好的用户体验，我也建议你对核心业务做服务端渲染的优化。基于 Serverless 的服务端渲染，可以让我们不用再像以前一样担心服务器的运维和扩容，大大提高了生产力。同时有了服务端渲染后，我也建议你完善业务的持续集成流程，将整个研发链路打通，降低代码构建发布的风险，提升从开发到测试再到部署的效率。</p>
<p data-nodeid="2150" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/04/7F/CioPOWAsxEeANlL-AAEkPyzgS2s711.png" alt="玩转 Serverless 架构17金句.png" data-nodeid="2154"></p>

<p data-nodeid="1880">当然，要达到页面的极致体验，我们还需要做很多工作，比如：</p>




<ul data-nodeid="1192">
<li data-nodeid="1193">
<p data-nodeid="1194">将静态资源部署到 CDN，提升资源加载速度；</p>
</li>
<li data-nodeid="1195">
<p data-nodeid="1196">针对页面进行缓存，减少函数冷启动对性能的影响；</p>
</li>
<li data-nodeid="1197">
<p data-nodeid="1198">对服务端异常进行降级处理等等。</p>
</li>
</ul>
<p data-nodeid="1199">但不管我们用不用 Serverless，都需要做这些工作。关于这一讲，我想要强调以下几点：</p>
<ul data-nodeid="1200">
<li data-nodeid="1201">
<p data-nodeid="1202">基于 Serverless 的服务端渲染应用，可以让我们不用关心服务器的运维，应用也天然具有弹性；</p>
</li>
<li data-nodeid="1203">
<p data-nodeid="1204">基于 Serverless 开发服务端渲染应用，建议你完善业务的持续集成流程；</p>
</li>
<li data-nodeid="1205">
<p data-nodeid="1206">要达到页面的极致性能，还需要考虑将静态资源部署到 CDN、对页面进行缓存等技术；</p>
</li>
<li data-nodeid="1207">
<p data-nodeid="1208">对于服务端渲染应用，建议你完善业务的服务降级能力，进一步提高稳定性。</p>
</li>
</ul>
<p data-nodeid="1209" class="">最后，我给你的作业是，实现一个服务端渲染应用。 我们下一讲见。</p>

---

### 精选评论


