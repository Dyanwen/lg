<p>今天我将继续和你分享 Webpack 另外的一个高级特性，Code Splitting（分块打包）。</p>
<h3>All in One 的弊端</h3>
<p>通过 Webpack 实现前端项目整体模块化的优势固然明显，但是它也会存在一些弊端：它最终会将我们所有的代码打包到一起。试想一下，如果我们的应用非常复杂，模块非常多，那么这种 All in One 的方式就会导致打包的结果过大，甚至超过 4～5M。</p>
<p>在绝大多数的情况下，应用刚开始工作时，并不是所有的模块都是必需的。如果这些模块全部被打包到一起，即便应用只需要一两个模块工作，也必须先把 bundle.js 整体加载进来，而且前端应用一般都是运行在浏览器端，这也就意味着应用的响应速度会受到影响，也会浪费大量的流量和带宽。</p>
<p>所以这种 All in One 的方式并不合理，更为合理的方案是<strong>把打包的结果按照一定的规则分离到多个 bundle 中，然后</strong>根据<strong>应用的运行需要按需加载</strong>。这样就可以降低启动成本，提高响应速度。</p>
<p>可能你会联想到我们在开篇词中讲过，Webpack 就是通过把项目中散落的模块打包到一起，从而提高加载效率，那么为什么这里又要分离？这不是自相矛盾吗？</p>
<p>其实这并不矛盾，只是物极必反罢了。Web 应用中的资源受环境所限，太大不行，太碎更不行。因为我们开发过程中划分模块的颗粒度一般都会非常的细，很多时候一个模块只是提供了一个小工具函数，并不能形成一个完整的功能单元。</p>
<p>如果我们不将这些资源模块打包，直接按照开发过程中划分的模块颗粒度进行加载，那么运行一个小小的功能，就需要加载非常多的资源模块。</p>
<p>再者，目前主流的 HTTP 1.1 本身就存在一些缺陷，例如：</p>
<ul>
<li>同一个域名下的并行请求是有限制的；</li>
<li>每次请求本身都会有一定的延迟；</li>
<li>每次请求除了传输内容，还有额外的请求头，大量请求的情况下，这些请求头加在一起也会浪费流量和带宽。</li>
</ul>
<p>综上所述，模块打包肯定是必要的，但当应用体积越来越大时，我们也要学会变通。</p>
<h3>Code Splitting</h3>
<p>为了解决打包结果过大导致的问题，Webpack 设计了一种分包功能：Code Splitting（代码分割）。</p>
<p>Code Splitting 通过把项目中的资源模块按照我们设计的规则打包到不同的 bundle 中，从而降低应用的启动成本，提高响应速度。</p>
<p>Webpack 实现分包的方式主要有两种：</p>
<ul>
<li>根据业务不同配置多个打包入口，输出多个打包结果；</li>
<li>结合 ES Modules 的动态导入（Dynamic Imports）特性，按需加载模块。</li>
</ul>
<h4>多入口打包</h4>
<p>多入口打包一般适用于传统的多页应用程序，最常见的划分规则就是一个页面对应一个打包入口，对于不同页面间公用的部分，再提取到公共的结果中。</p>
<p>Webpack 配置多入口打包的方式非常简单，这里我准备了一个<a href="https://github.com/zce/webpack-multi-entry">相应的示例</a>，具体结构如下：</p>
<blockquote>
<p>示例源码</p>
<ul>
<li>GitHub：<a href="https://github.com/zce/webpack-multi-entry">https://github.com/zce/webpack-multi-entry</a></li>
<li>CodeSandbox：<a href="https://codesandbox.io/s/github/zce/webpack-multi-entry">https://codesandbox.io/s/github/zce/webpack-multi-entry</a></li>
</ul>
</blockquote>
<pre><code data-language="java" class="lang-java">.
├── dist
├── src
│   ├── common
│   │   ├── fetch.js
│   │   └── global.css
│   ├── album.css
│   ├── album.html
│   ├── album.js
│   ├── index.css
│   ├── index.html
│   └── index.js
├── <span class="hljs-keyword">package</span>.json
└── webpack.config.js
</code></pre>
<p>这个示例中有两个页面，分别是 index 和 album。代码组织的逻辑也很简单：</p>
<ul>
<li>index.js 负责实现 index 页面功能逻辑；</li>
<li>album.js 负责实现 album 页面功能逻辑；</li>
<li>global.css 是公用的样式文件；</li>
<li>fetch.js 是一个公用的模块，负责请求 API。</li>
</ul>
<p>我们回到配置文件中，这里我们尝试为这个案例配置多入口打包，具体配置如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> HtmlWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'html-webpack-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">index</span>: <span class="hljs-string">'./src/index.js'</span>,
    <span class="hljs-attr">album</span>: <span class="hljs-string">'./src/album.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span> <span class="hljs-comment">// [name] 是入口名称</span>
  },
  <span class="hljs-comment">// ... 其他配置</span>
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> HtmlWebpackPlugin({
      <span class="hljs-attr">title</span>: <span class="hljs-string">'Multi Entry'</span>,
      <span class="hljs-attr">template</span>: <span class="hljs-string">'./src/index.html'</span>,
      <span class="hljs-attr">filename</span>: <span class="hljs-string">'index.html'</span>
    }),
    <span class="hljs-keyword">new</span> HtmlWebpackPlugin({
      <span class="hljs-attr">title</span>: <span class="hljs-string">'Multi Entry'</span>,
      <span class="hljs-attr">template</span>: <span class="hljs-string">'./src/album.html'</span>,
      <span class="hljs-attr">filename</span>: <span class="hljs-string">'album.html'</span>
    })
  ]
}
</code></pre>
<p>一般 entry 属性中只会配置一个打包入口，如果我们需要配置多个入口，可以把 entry 定义成一个对象。</p>
<blockquote>
<p>注意：这里 entry 是定义为对象而不是数组，如果是数组的话就是把多个文件打包到一起，还是一个入口。</p>
</blockquote>
<p>在这个对象中一个属性就是一个入口，属性名称就是这个入口的名称，值就是这个入口对应的文件路径。那我们这里配置的就是 index 和 album 页面所对应的 JS 文件路径。</p>
<p>一旦我们的入口配置为多入口形式，那输出文件名也需要修改，因为两个入口就有两个打包结果，不能都叫 bundle.js。我们可以在这里使用 [name] 这种占位符来输出动态的文件名，[name] 最终会被替换为入口的名称。</p>
<p>除此之外，在配置中还通过 html-webpack-plugin 分别为 index 和 album 页面生成了对应的 HTML 文件。</p>
<p>完成配置之后，我们就可以打开命令行终端，运行 Webpack 打包，那此次打包会有两个入口。打包完成后，我们找到输出目录，这里就能看到两个入口文件各自的打包结果了，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/EA/CgqCHl7Ez4KAXb6IAAB9gPjr4iw703.png" alt="image (11).png"></p>
<p>但是这里还有一个小问题，我们打开任意一个输出的 HTML 文件，具体结果如下图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/DE/Ciqc1F7Ez4yAFCd6AAFu5g0zQLk550.png" alt="image (12).png"></p>
<p>你就会发现 index 和 album 两个打包结果都被页面载入了，而我们希望的是每个页面只使用它对应的那个输出结果。</p>
<p>所以这里还需要修改配置文件，我们回到配置文件中，找到输出 HTML 的插件，默认这个插件会自动注入所有的打包结果，如果需要指定所使用的 bundle，我们可以通过 HtmlWebpackPlugin 的 chunks 属性来设置。我们分别为两个页面配置使用不同的 chunk，具体配置如下：</p>
<blockquote>
<p>TIPS：每个打包入口都会形成一个独立的 chunk（块）。</p>
</blockquote>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> HtmlWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'html-webpack-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">index</span>: <span class="hljs-string">'./src/index.js'</span>,
    <span class="hljs-attr">album</span>: <span class="hljs-string">'./src/album.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span> <span class="hljs-comment">// [name] 是入口名称</span>
  },
  <span class="hljs-comment">// ... 其他配置</span>
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> HtmlWebpackPlugin({
      <span class="hljs-attr">title</span>: <span class="hljs-string">'Multi Entry'</span>,
      <span class="hljs-attr">template</span>: <span class="hljs-string">'./src/index.html'</span>,
      <span class="hljs-attr">filename</span>: <span class="hljs-string">'index.html'</span>,
      <span class="hljs-attr">chunks</span>: [<span class="hljs-string">'index'</span>] <span class="hljs-comment">// 指定使用 index.bundle.js</span>
    }),
    <span class="hljs-keyword">new</span> HtmlWebpackPlugin({
      <span class="hljs-attr">title</span>: <span class="hljs-string">'Multi Entry'</span>,
      <span class="hljs-attr">template</span>: <span class="hljs-string">'./src/album.html'</span>,
      <span class="hljs-attr">filename</span>: <span class="hljs-string">'album.html'</span>,
      <span class="hljs-attr">chunks</span>: [<span class="hljs-string">'album'</span>] <span class="hljs-comment">// 指定使用 album.bundle.js</span>
    })
  ]
}
</code></pre>
<p>完成以后我们再次回到命令行终端，然后运行打包，打包结果如下图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/EA/CgqCHl7Ez6OAIz0wAAFGLXodY44387.png" alt="image (13).png"></p>
<p>这一次打包的结果就完全正常了。</p>
<p>那这就是配置多入口打包的方法，以及如何指定在 HTML 中注入的 bundle。</p>
<p><strong>提取公共模块</strong></p>
<p>多入口打包本身非常容易理解和使用，但是它也存在一个小问题，就是不同的入口中一定会存在一些公共使用的模块，如果按照目前这种多入口打包的方式，就会出现多个打包结果中有相同的模块的情况。</p>
<p>例如我们上述案例中，index 入口和 album 入口中就共同使用了 global.css 和 fetch.js 这两个公共的模块。这里是因为我们的示例比较简单，所以重复的影响没有那么大，但是如果我们公共使用的是 jQuery 或者 Vue.js 这些体积较大的模块，那影响就会比较大，不利于公共模块的缓存。</p>
<p>所以我们还需要把这些公共的模块提取到一个单独的 bundle 中。Webpack 中实现公共模块提取非常简单，我们只需要在优化配置中开启 splitChunks 功能就可以了，具体配置如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">index</span>: <span class="hljs-string">'./src/index.js'</span>,
    <span class="hljs-attr">album</span>: <span class="hljs-string">'./src/album.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span> <span class="hljs-comment">// [name] 是入口名称</span>
  },
  <span class="hljs-attr">optimization</span>: {
    <span class="hljs-attr">splitChunks</span>: {
      <span class="hljs-comment">// 自动提取所有公共模块到单独 bundle</span>
      <span class="hljs-attr">chunks</span>: <span class="hljs-string">'all'</span>
    }
  }
  <span class="hljs-comment">// ... 其他配置</span>
}
</code></pre>
<p>我们回到配置文件中，这里在 optimization 属性中添加 splitChunks 属性，那这个属性的值是一个对象，这个对象需要配置一个 chunks 属性，我们这里将它设置为 all，表示所有公共模块都可以被提取。</p>
<p>完成以后我们打开命令行终端，再次运行 Webpack 打包，打包结果如下图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/EA/CgqCHl7Ez8-ALpdfAABzb5Czbeo871.png" alt="image (14).png"></p>
<p>此时在我们的 dist 下就会额外生成一个 JS 文件，在这个文件中就是 index 和 album 中公共的模块部分了。</p>
<p>除此之外，splitChunks 还支持很多高级的用法，可以实现各种各样的分包策略，这些我们可以在<a href="https://webpack.js.org/plugins/split-chunks-plugin/">文档</a>中找到对应的介绍。</p>
<h4>动态导入</h4>
<p>除了多入口打包的方式，Code Splitting 更常见的实现方式还是结合 ES Modules 的动态导入特性，从而实现按需加载。</p>
<p>按需加载是开发浏览器应用中一个非常常见的需求。一般我们常说的按需加载指的是加载数据或者加载图片，但是我们这里所说的按需加载，指的是在应用运行过程中，需要某个资源模块时，才去加载这个模块。这种方式极大地降低了应用启动时需要加载的资源体积，提高了应用的响应速度，同时也节省了带宽和流量。</p>
<p>Webpack 中支持使用动态导入的方式实现模块的按需加载，而且所有动态导入的模块都会被自动提取到单独的 bundle 中，从而实现分包。</p>
<p>相比于多入口的方式，动态导入更为灵活，因为我们可以通过代码中的逻辑去控制需不需要加载某个模块，或者什么时候加载某个模块。而且我们分包的目的中，很重要的一点就是让模块实现按需加载，从而提高应用的响应速度。</p>
<p>接下来，我们具体来看如何使用动态导入特性，这里我已经设计了一个可以发挥按需加载作用的场景，具体效果如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/EA/CgqCHl7Ez--AA39kACvspSgItKU114.gif" alt="example.gif"></p>
<p>在这个应用的主体区域，如果我们访问的是首页，它显示的是一个文章列表，如果我们访问的是相册页，它显示的就是相册列表。</p>
<p>回到代码中，我们来看目前的实现方式，具体结构如下：</p>
<pre><code data-language="java" class="lang-java">.
├── src
│   ├── album
│   │   ├── album.css
│   │   └── album.js
│   ├── common
│   │   ├── fetch.js
│   │   └── global.css
│   ├── posts
│   │   ├── posts.css
│   │   └── posts.js
│   ├── index.html
│   └── index.js
├── <span class="hljs-keyword">package</span>.json
└── webpack.config.js
</code></pre>
<p>文章列表对应的是这里的 posts 组件，而相册列表对应的是 album 组件。我在打包入口（index.js）中同时导入了这两个模块，然后根据页面锚点的变化决定显示哪个组件，核心代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// ./src/index.js</span>
<span class="hljs-keyword">import</span> posts <span class="hljs-keyword">from</span> <span class="hljs-string">'./posts/posts'</span>
<span class="hljs-keyword">import</span> album <span class="hljs-keyword">from</span> <span class="hljs-string">'./album/album'</span>
<span class="hljs-keyword">const</span> update = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
  <span class="hljs-keyword">const</span> hash = <span class="hljs-built_in">window</span>.location.hash || <span class="hljs-string">'#posts'</span>
  <span class="hljs-keyword">const</span> mainElement = <span class="hljs-built_in">document</span>.querySelector(<span class="hljs-string">'.main'</span>)
  mainElement.innerHTML = <span class="hljs-string">''</span>
  <span class="hljs-keyword">if</span> (hash === <span class="hljs-string">'#posts'</span>) {
    mainElement.appendChild(posts())
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (hash === <span class="hljs-string">'#album'</span>) {
    mainElement.appendChild(album())
  }
}
<span class="hljs-built_in">window</span>.addEventListener(<span class="hljs-string">'hashchange'</span>, update)
update()
</code></pre>
<p>在这种情况下，就可能产生资源浪费。试想一下：如果用户只需要访问其中一个页面，那么加载另外一个页面对应的组件就是浪费。</p>
<p>如果我们采用动态导入的方式，就不会产生浪费的问题了，因为所有的组件都是惰性加载，只有用到的时候才会去加载。具体实现代码如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// ./src/index.js</span>
<span class="hljs-comment">// import posts from './posts/posts'</span>
<span class="hljs-comment">// import album from './album/album'</span>
<span class="hljs-keyword">const</span> update = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
  <span class="hljs-keyword">const</span> hash = <span class="hljs-built_in">window</span>.location.hash || <span class="hljs-string">'#posts'</span>
  <span class="hljs-keyword">const</span> mainElement = <span class="hljs-built_in">document</span>.querySelector(<span class="hljs-string">'.main'</span>)
  mainElement.innerHTML = <span class="hljs-string">''</span>
  <span class="hljs-keyword">if</span> (hash === <span class="hljs-string">'#posts'</span>) {
    <span class="hljs-comment">// mainElement.appendChild(posts())</span>
    <span class="hljs-keyword">import</span>(<span class="hljs-string">'./posts/posts'</span>).then(<span class="hljs-function">(<span class="hljs-params">{ <span class="hljs-keyword">default</span>: posts }</span>) =&gt;</span> {
      mainElement.appendChild(posts())
    })
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (hash === <span class="hljs-string">'#album'</span>) {
    <span class="hljs-comment">// mainElement.appendChild(album())</span>
    <span class="hljs-keyword">import</span>(<span class="hljs-string">'./album/album'</span>).then(<span class="hljs-function">(<span class="hljs-params">{ <span class="hljs-keyword">default</span>: album }</span>) =&gt;</span> {
      mainElement.appendChild(album())
    })
  }
}
<span class="hljs-built_in">window</span>.addEventListener(<span class="hljs-string">'hashchange'</span>, update)
update()
</code></pre>
<blockquote>
<p>P.S. 为了动态导入模块，可以将 import 关键字作为函数调用。当以这种方式使用时，import 函数返回一个 Promise 对象。这就是 ES Modules 标准中的 <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#Dynamic_Imports">Dynamic Imports</a>。</p>
</blockquote>
<p>这里我们先移除 import 这种静态导入，然后在需要使用组件的地方通过 import 函数导入指定路径，那这个方法返回的是一个 Promise。在这个 Promise 的 then 方法中我们能够拿到模块对象。由于我们这里的 posts 和 album 模块是以默认成员导出，所以我们需要解构模块对象中的 default，先拿到导出成员，然后再正常使用这个导出成员。</p>
<p>完成以后，Webpack Dev Server 自动重新打包，我们再次回到浏览器，此时应用仍然是可以正常工作的。</p>
<p>那我们再回到命令行终端，重新运行打包，然后看看此时的打包结果具体是怎样的。打包完成以后我们打开 dist 目录，具体结果如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/EA/CgqCHl7E0AiAT-imAAHBvVxQKR4701.png" alt="image (15).png"></p>
<p>此时 dist 目录下就会额外多出三个 JS 文件，其中有两个文件是动态导入的模块，另外一个文件是动态导入模块中公共的模块，这三个文件就是由动态导入自动分包产生的。</p>
<p>以上就是动态导入在 Webpack 中的使用。整个过程我们无需额外配置任何地方，只需要按照 ES Modules 动态导入的方式去导入模块就可以了，Webpack 内部会自动处理分包和按需加载。</p>
<p>如果你使用的是 Vue.js 之类的 SPA 开发框架的话，那你项目中路由映射的组件就可以通过这种动态导入的方式实现按需加载，从而实现分包。</p>
<h4>魔法注释</h4>
<p>默认通过动态导入产生的 bundle 文件，它的 name 就是一个序号，这并没有什么不好，因为大多数时候，在生产环境中我们根本不用关心资源文件的名称。</p>
<p>但是如果你还是需要给这些 bundle 命名的话，就可以使用 Webpack 所特有的魔法注释去实现。具体方式如下：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// 魔法注释</span>
<span class="hljs-keyword">import</span>(<span class="hljs-comment">/* webpackChunkName: 'posts' */</span><span class="hljs-string">'./posts/posts'</span>)
  .then(<span class="hljs-function">(<span class="hljs-params">{ <span class="hljs-keyword">default</span>: posts }</span>) =&gt;</span> {
    mainElement.appendChild(posts())
  })
</code></pre>
<p>所谓魔法注释，就是在 import 函数的形式参数位置，添加一个行内注释，这个注释有一个特定的格式：webpackChunkName: '<chunk-name>'，这样就可以给分包的 chunk 起名字了。</chunk-name></p>
<p>完成过后，我们再次打开命令行终端，运行 Webpack 打包，那此时我们生成 bundle 的 name 就会使用刚刚注释中提供的名称了，具体结果如下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/DF/Ciqc1F7E0DuAVFEdAAHM4Eurysw225.png" alt="image (16).png"></p>
<p>除此之外，魔法注释还有个特殊用途：如果你的 chunkName 相同的话，那相同的 chunkName 最终就会被打包到一起，例如我们这里可以把这两个 chunkName 都设置为 components，然后再次运行打包，那此时这两个模块都会被打包到一个文件中，具体操作如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0D/DF/Ciqc1F7E0EOAB4zFAALyynED1R4662.png" alt="image (17).png"></p>
<p>借助这个特点，你就可以根据自己的实际情况，灵活组织动态加载的模块了。</p>
<h3>写在最后</h3>
<p>最后我们来总结一下今天的核心内容，我们介绍了为什么要进行分包，以及 Webpack Code Splitting 的两种实现方式，分别是多入口打包和动态导入，其中动态导入会更常用到。</p>
<p>在这里，我想跟你再额外聊几句我的看法，其实从事开发工作就是不断“制造”问题，再不断解决问题。也正是在这样的一个制造问题解决问题的过程中，行业的技术、标准、工具不断迭代，不断完善，这是一个向好的过程。作为开发人员千万不要怕麻烦，应该多思考，多积累，才能更好地适应，甚至是引领行业的变化。</p>

---

### 精选评论

##### *新：
> 讲道理，这个讲的真的不错，一开始我就挺抗拒学webpack，但是项目有用到，不会也不行，看了很多教程都是一知半解，许久不用也就忘了。但是这个课程，真的很不错，对我个人需求很契合。<span style="font-size: 0.427rem;">真的很有深入浅出的特点，我一般是看一遍，在听一遍，加深影响，入门之后，踩坑了在看一遍。讲的非常不错。</span><div><span style="font-size: 0.427rem;">我相信网上大多数讲师水平是有的，但是出的课程没有一个像这样课程规划的好，至少我目前看到第10讲，也并没有一节很敷衍的课程，每一节课出的很稳，很舒服，听起来没有跳跃的负担。很不错，给个赞！</span></div>

##### **俊：
> 内容串联的很自然，很舒服，学起来没有心智负担，赞！

##### **青：
> 老师，调试的过程中发现打包后，公共模块（album~index.bundle.js）并没有引入到index.html或者album.html中，这是需要加什么配置么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; chunks 中加入公共模块即可

##### **民：
> 学习学习

##### **峰：
> 看得津津有味，打算开始系统的学习webpack了

##### **园：
> 老师，vue cli这种构建的项目没有设置splitChunks 是怎么自动分包的呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; vue-cli 内部配置了 splitChunks 选项，你可以使用 vue-cli-service inspect 命令查看 vue-cli 最终内部的配置

##### *盼：
> 老师讲得很好，不浮于表面，有深度

##### **随行：
> 一口气看到这，写得很棒👍🏻👍🏻👍🏻

##### *汉：
> 实话说 这是我学过的最扎实最浅显易懂强烈推荐的webpack精品课

##### **4501：
> 请问老师，vue-cli3中如何做到资源按需加载呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; vue-cli 生成的项目采用的是 vue-cli-service 实现的构建，这个 vue-cli-service 内部集成的就是 webpack，所以我们介绍的按需加载方式，在这种类型的项目中同样可以正常使用

##### 2r：
> 讲的不错

