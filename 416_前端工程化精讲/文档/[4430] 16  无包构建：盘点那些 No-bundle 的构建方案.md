<p data-nodeid="44158" class="">上节课我们讨论了 Webpack 的最新版本 Webpack 5 所带来的提效新功能。思考题是 Webpack 5 中的持久化缓存究竟会影响哪些构建环节呢？</p>
<p data-nodeid="44159">通过对 compiler.cache.hook.get 的追踪不难发现：持久化缓存一共影响下面这些环节与内置的插件：</p>
<ul data-nodeid="44160">
<li data-nodeid="44161">
<p data-nodeid="44162">编译模块：ResolverCachePlugin、Compilation/modules。</p>
</li>
<li data-nodeid="44163">
<p data-nodeid="44164">优化模块：FlagDependencyExportsPlugin、ModuleConcatenationPlugin。</p>
</li>
<li data-nodeid="44165">
<p data-nodeid="44166">生成代码：Compilation/codeGeneration、Compilation/assets。</p>
</li>
<li data-nodeid="44167">
<p data-nodeid="44168">优化产物：TerserWebpackPlugin、RealContentHashPlugin。</p>
</li>
</ul>
<p data-nodeid="44169">正是通过这样多环节的缓存读写控制，才打造出 Webpack 5 高效的持久化缓存功能。</p>
<p data-nodeid="44170">在之前的课程里我们详细分解了 Webpack 构建工具的效率优化方案，这节课我们来聊一聊今年比较火的另一种构建工具思路：无包构建（No-Bundle/Unbundle）。</p>
<h3 data-nodeid="44171">什么是无包构建</h3>
<p data-nodeid="44172">什么是无包构建呢？这是一个与基于模块化打包的构建方案相对的概念。</p>
<p data-nodeid="44173">在“ 第 9 课时|构建总览：前端构建工具的演进”中谈到过，目前主流的构建工具，例如 Webpack、Rollup 等都是基于一个或多个入口点模块，通过依赖分析将有依赖关系的模块<strong data-nodeid="44255">打包到一起</strong>，最后形成少数几个产物代码包，因此这些工具也被称为<strong data-nodeid="44256">打包工具</strong>。只不过，这些工具的构建过程除了打包外，还包括了模块编译和代码优化等，因此称为打包式构建工具或许更恰当。</p>
<p data-nodeid="44174">而<strong data-nodeid="44272">无包构建</strong>是指这样一类构建方式：在构建时只需处理模块的编译而<strong data-nodeid="44273">无须打包</strong>，把模块间的**依赖关系完全交给浏览器来处理。**浏览器会加载入口模块，分析依赖后，再通过网络请求加载被依赖的模块。通过这样的方式简化构建时的处理过程，提升构建效率。</p>
<p data-nodeid="44175">这种通过浏览器原生的模块进行解析的方式又称为 Native-ESM（Native ES Module）。下面我们就通过一个简单示例来展示这种基于浏览器的模块加载过程（<a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/16_nobundle" data-nodeid="44279">16_nobundle</a>/simple-esm），如下面的代码和图片所示：</p>
<pre class="lang-javascript" data-nodeid="44176"><code data-language="javascript"><span class="hljs-comment">//./src/index.html</span>
...
&lt;script type=<span class="hljs-string">"module"</span> src=<span class="hljs-string">"./modules/foo.js"</span>&gt;&lt;/script&gt;
...
<span class="hljs-comment">//.src/modules/foo.js</span>
<span class="hljs-keyword">import</span> { bar } <span class="hljs-keyword">from</span> <span class="hljs-string">'./bar.js'</span>
<span class="hljs-keyword">import</span> { appendHTML } <span class="hljs-keyword">from</span> <span class="hljs-string">'./common.js'</span>
...
import(<span class="hljs-string">'https://cdn.jsdelivr.net/npm/lodash-es@4.17.15/slice.js'</span>).then(<span class="hljs-function">(<span class="hljs-params"><span class="hljs-built_in">module</span></span>) =&gt;</span> {...})
</code></pre>
<p data-nodeid="44177"><img src="https://s0.lgstatic.com/i/image/M00/59/D5/CgqCHl9yo46AYuszAANDvM6jRMk647.png" alt="Drawing 0.png" data-nodeid="44283"></p>
<p data-nodeid="44178">从示例中可以看到，在没有任何构建工具处理的情况下，在页面中引入带有 type="module" 属性的 script，浏览器就会在加载入口模块时依次加载了所有被依赖的模块。下面我们就来深入了解一下这种基于浏览器加载 JS 模块的技术的细节。</p>
<h3 data-nodeid="44179">基于浏览器的 JS 模块加载功能</h3>
<p data-nodeid="44180">从 caniuse 网站中可以看到，目前大部分主流的浏览器都已支持 JavaScript modules 这一特性，如下图所示：</p>
<p data-nodeid="44181"><img src="https://s0.lgstatic.com/i/image/M00/59/CA/Ciqc1F9yo5aADhYKAAMTR4GJTG8708.png" alt="Drawing 1.png" data-nodeid="44293"></p>
<p data-nodeid="44182">[图片来源：<a href="https://caniuse.com/es6-module" data-nodeid="44298">https://caniuse.com/es6-module</a>]</p>
<p data-nodeid="44183">我们来总结这种加载方式的注意点。</p>
<h4 data-nodeid="44184">HTML 中的 Script 引用</h4>
<ul data-nodeid="44185">
<li data-nodeid="44186">
<p data-nodeid="44187">入口模块文件在页面中引用时需要带上**type="module"**属性。对应的，存在 type="nomodule"，即支持 ES Module 的现代浏览器，它会忽略 type="nomodule" 属性的 script，因此可以用作旧浏览器中的降级方案。</p>
</li>
<li data-nodeid="44188">
<p data-nodeid="44189">带有 type="module" 属性的 script在浏览器中<strong data-nodeid="44329">通过 defer 的方式异步执行</strong>（异步下载，不阻塞 HTML，顺次执行），即使是行内的 script 代码也遵循这一原则（而普通的行内 script 代码则忽略 defer 属性）。</p>
</li>
<li data-nodeid="44190">
<p data-nodeid="44191">带有 type="module" 属性且带有<strong data-nodeid="44347">async</strong>属性的 script，在浏览器中<strong data-nodeid="44348">通过 async 的方式异步执行</strong>（异步下载，不阻塞 HTML，按该<strong data-nodeid="44349">模块和所依赖的模块</strong>下载完成的先后顺序执行，无视 DOM 中的加载顺序），即使是行内的 script 代码，也遵循这一原则（而普通的行内 script 代码则忽略 async 属性）。</p>
</li>
<li data-nodeid="44192">
<p data-nodeid="44193">即使多次加载相同模块，也只会执行一次。</p>
</li>
</ul>
<h4 data-nodeid="44194">模块内依赖的引用</h4>
<ul data-nodeid="44195">
<li data-nodeid="44196">
<p data-nodeid="44197">只能使用 import ... from '...' 的 ES6 风格的模块导入方式，或者使用 import(...).then(...) 的 ES6 动态导入方式，不支持其他模块化规范的引用方式（例如 require、define 等）。</p>
</li>
<li data-nodeid="44198">
<p data-nodeid="44199">导入的模块只支持使用相对路径（'/xxx', './xxx', '../xxx'）和 URL 方式（'https://xxx', 'http://xxx'）进行引用，不支持直接使用包名开头的方式（'xxxx', 'xxx/xxx'）。</p>
</li>
<li data-nodeid="44200">
<p data-nodeid="44201">只支持引用MIME Type为 text/javascript 方式的模块，不支持其他类型文件的加载（例如 CSS 等）。</p>
</li>
</ul>
<h4 data-nodeid="44202">为什么需要构建工具</h4>
<p data-nodeid="44203">从上面的技术细节中我们会发现，对于一个普通的项目而言，要使用这种加载方案仍然有几个主要问题：</p>
<ol data-nodeid="44204">
<li data-nodeid="44205">
<p data-nodeid="44206">许多其他类型的文件需要编译处理为 ES6 模块才能被浏览器正常加载（JSX、Vue、TS、CSS、Image 等）。</p>
</li>
<li data-nodeid="44207">
<p data-nodeid="44208">许多第三方依赖包在通过第三方 URL 引用时，不仅过程烦琐，而且往往难以进行灵活的版本控制与更新，因此需要合适的方式来解决引用路径的问题。</p>
</li>
<li data-nodeid="44209">
<p data-nodeid="44210">对于现实中的项目开发而言，一些便利的辅助开发技术，例如热更新等还是需要由构建工具来提供。</p>
</li>
</ol>
<p data-nodeid="44211">下面，我们分析 Vite 和 Snowpack 这两个有代表性的构建工具是如何解决上面的问题的。</p>
<h3 data-nodeid="44212">Vite</h3>
<p data-nodeid="44213"><a href="https://github.com/vitejs/vite" data-nodeid="44396">Vite</a> 是 Vue 框架的作者尤雨溪最新推出的基于 Native-ESM 的 Web 构建工具。它在开发环境下基于 Native-ESM 处理构建过程，只编译不打包，在生产环境下则基于 Rollup 打包。我们还是先通过 Vite 的官方示例来观察它的使用效果，如下面的代码和图片所示（示例代码参见 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/16_nobundle/example-vite" data-nodeid="44400">example-vite</a>）：</p>
<pre class="lang-java" data-nodeid="44214"><code data-language="java">npm init vite-app example-vite
cd example-vite
npm install
npm run dev
</code></pre>
<p data-nodeid="44215"><img src="https://s0.lgstatic.com/i/image/M00/59/D5/CgqCHl9yo-mAWrIzAAOaSZguuaM643.png" alt="Drawing 2.png" data-nodeid="44404"></p>
<p data-nodeid="44216">可以看到，运行示例代码后，在浏览器中只引入了 src/main.js 这一个入口模块，但是在网络面板中却依次加载了若干依赖模块，包括外部模块 vue 和 css。依赖图如下：</p>
<p data-nodeid="44217"><img src="https://s0.lgstatic.com/i/image/M00/59/CA/Ciqc1F9yo_GAWATTAACYUvrJKL4148.png" alt="Drawing 4.png" data-nodeid="44408"></p>
<p data-nodeid="44218">可以看到，经过 Vite 处理后，浏览器中加载的模块与源代码中导入的模块相比发生了变化，这些变化包括对外部依赖包的处理，对 vue 文件的处理，对 css 文件的处理等。下面我们就来逐个分析其中的变化。</p>
<h4 data-nodeid="44219">对导入模块的解析</h4>
<p data-nodeid="44220"><strong data-nodeid="44414">对 HTML 文件的预处理</strong></p>
<p data-nodeid="44221">当启动 Vite 时，会通过 <a href="https://github.com/vitejs/vite/blob/master/src/node/server/serverPluginHtml.ts" data-nodeid="44418">serverPluginHtml</a>.ts 注入 <a href="https://github.com/vitejs/vite/blob/master/src/client/client.ts" data-nodeid="44422">/vite/client</a> 运行时的依赖模块，该模块用于处理热更新，以及提供更新 CSS 的方法 updateStyle。</p>
<p data-nodeid="44222"><strong data-nodeid="44427">对外部依赖包的解析</strong></p>
<p data-nodeid="44223">首先是对不带路径前缀的外部依赖包（也称为<strong data-nodeid="44437">Bare Modules</strong>）的解析，例如上图中在示例源代码中导入了 'vue' 模块，但是在浏览器的网络请求中变为了请求 /@module/vue。</p>
<p data-nodeid="44224">这个解析过程在 Vite 中主要通过三个文件来处理：</p>
<ul data-nodeid="44225">
<li data-nodeid="44226">
<p data-nodeid="44227"><a href="https://github.com/vitejs/vite/blob/master/src/node/resolver.ts#L464" data-nodeid="44441">resolver</a>.ts 负责找到对应在 node_modules 中的真实依赖包代码（Vite 会在启动服务时对项目 package.json 中的 dependencies 做预处理读取并存入缓存目录 node_modules/.vite_opt_cache 中）。</p>
</li>
<li data-nodeid="44228">
<p data-nodeid="44229"><a href="https://github.com/vitejs/vite/blob/master/src/node/server/serverPluginModuleRewrite.ts" data-nodeid="44453">serverPluginModuleRewrite</a>.ts 负责把源码中的 bare modules 加上 /@module/ 前缀。</p>
</li>
<li data-nodeid="44230">
<p data-nodeid="44231"><a href="https://github.com/vitejs/vite/blob/master/src/node/server/serverPluginModuleResolve.ts" data-nodeid="44457">serverPluginModuleResolve</a>.ts 负责解析加上前缀后的模块。</p>
</li>
</ul>
<p data-nodeid="44232"><strong data-nodeid="44462">对 Vue文件的解析</strong></p>
<p data-nodeid="48398" class="">对 Vue 文件的解析是通过 <a href="https://github.com/vitejs/vite/blob/master/src/node/server/serverPluginVue.ts" data-nodeid="48402">serverPluginVue</a>.ts 处理的，分离出 Vue 代码中的 script/template/style 代码片段，并分别转换为 JS 模块，然后将 template/style 模块的 import写到script 模块代码的头部。因此在浏览器访问时，一个 Vue 源代码文件会分裂为 2~3 的关联请求（例如上面的 /src/App.vue 和 /src/App.vue?type=template，如果 App.vue 中包含<code data-backticks="1" data-nodeid="48406"> &lt;style&gt;</code> 则会产生第 3 个请求 /src/App.vue?type=style）。</p>

<p data-nodeid="47074"><strong data-nodeid="47153">对 CSS 文件的解析</strong></p>
<p data-nodeid="49494" class="">对 CSS 文件的解析是通过 <a href="https://github.com/vitejs/vite/blob/master/src/node/server/serverPluginCss.ts" data-nodeid="49498">serverPluginCSS</a>.ts 处理的，解析过程主要是将 CSS 文件的内容转换为下面的 JS 代码模块，其中的 updateStyle 由注入 HTML 中的 /vite/client 模块提供，如下面的代码所示：</p>

<pre class="lang-javascript" data-nodeid="47076"><code data-language="javascript"><span class="hljs-keyword">import</span> { updateStyle } <span class="hljs-keyword">from</span> <span class="hljs-string">"/vite/client"</span>
<span class="hljs-keyword">const</span> css = <span class="hljs-string">"..."</span>
updateStyle(<span class="hljs-string">"\"...\""</span>, css) <span class="hljs-comment">// id, cssContent</span>
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> css
</code></pre>
<p data-nodeid="47077">以上就是示例代码中主要文件类型的基本解析逻辑，可以看到，Vite 正是通过这些解析器来解决不同类型文件以 JS 模块的方式在浏览器中加载的问题。在 Vite 源码中还包含了其他更多文件类型的解析器，例如 JSON、TS、SASS 等，这里就不一一列举了，感兴趣的话，你可以进一步查阅<a href="https://github.com/vitejs/vite" data-nodeid="47162">官方文档</a>。</p>
<h4 data-nodeid="47078">Vite 中的其他辅助功能</h4>
<p data-nodeid="47079">除了提供这些解析器的能力外，Vite 还提供了其他便捷的构建功能，大致整理如下：</p>
<ul data-nodeid="51730">
<li data-nodeid="51731">
<p data-nodeid="51732"><strong data-nodeid="51750">多框架</strong>：除了在默认的 Vue 中使用外，还支持在 React 和 Preact 项目中使用。工具默认提供了 Vue、React 和 Preact 对应的脚手架模板。</p>
</li>
<li data-nodeid="51733">
<p data-nodeid="51734"><strong data-nodeid="51755">热更新（HMR）</strong>：默认提供的 3 种框架的脚手架模板中都内置了 HMR 功能，同时也提供了 HMR 的 API 供第三方插件或项目代码使用。</p>
</li>
<li data-nodeid="51735">
<p data-nodeid="51736"><strong data-nodeid="51764">自定义配置文件</strong>：支持使用自定义配置文件来细化构建配置，配置项功能参考 <a href="https://github.com/vuejs/vite/blob/master/src/node/config.ts" data-nodeid="51762">config.ts</a>。</p>
</li>
<li data-nodeid="51737">
<p data-nodeid="51738"><strong data-nodeid="51769">HTTPS 与 HTTP/2</strong>：支持使用 --https 启动参数来开启使用 HTTPS 和 HTTP/2 协议的开发服务器。</p>
</li>
<li data-nodeid="51739">
<p data-nodeid="51740"><strong data-nodeid="51774">服务代理</strong>：在自定义配置中支持配置代理，将部分请求代理到第三方服务。</p>
</li>
<li data-nodeid="51741">
<p data-nodeid="51742"><strong data-nodeid="51779">模式与环境变量</strong>：支持通过 mode 来指定构建模式为 development 或 production。相应模式下自动读取 dotenv 类型的环境变量配置文件（例如 .env.production.local）。</p>
</li>
<li data-nodeid="51743">
<p data-nodeid="51744" class=""><strong data-nodeid="51788">生产环境打包</strong>：生产环境使用 Rollup 进行打包，支持传入自定义配置，配置项功能参考 <a href="https://github.com/vitejs/vite/blob/master/src/node/build/index.ts" data-nodeid="51786">build/index.ts</a>。</p>
</li>
</ul>
<h4 data-nodeid="51745">Vite 的使用限制</h4>



<p data-nodeid="47096">Vite 的使用限制如下：</p>
<ul data-nodeid="53977">
<li data-nodeid="53978">
<p data-nodeid="53979" class="">面向支持 ES6 的现代浏览器，在生产环境下，编译目标参数 esBuildTarget 的默认值为 es2019，最低支持版本为 es2015（因为内部会使用 <a href="https://github.com/vitejs/vite/blob/master/src/node/esbuildService.ts" data-nodeid="53985">esbuild</a> 处理编译压缩，用来获得<a href="https://github.com/evanw/esbuild#why-is-it-fast" data-nodeid="53989">最快的构建速度</a>）。</p>
</li>
<li data-nodeid="53980">
<p data-nodeid="53981">对 Vue 框架的支持目前仅限于最新的 Vue 3 版本，不兼容更低版本。</p>
</li>
</ul>


<h3 data-nodeid="47102">Snowpack</h3>
<p data-nodeid="55078" class="">Snowpack 是另一个比较知名的无包构建工具，从整体功能来说和上述 Vite工具提供的功能大致相同，主要差异点在 Snowpack 在生产环境下默认使用无包构建而非打包模式（可以通过引入打包插件例如 @snowpack/plugin-webpack 来实现打包模式），而 Vite 仅在开发模式下使用。示例代码参见 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/16_nobundle/example-vite" data-nodeid="55082">example-snow</a>。下面我们简单整理下两者的异同。</p>

<h4 data-nodeid="47104">与 Vite 相同的功能点</h4>
<p data-nodeid="47105">两者都支持各种代码转换加载器、热更新、环境变量（需要安装 dotenv 插件）、服务代理、HTTPS 与 HTTP/2 等。</p>
<h4 data-nodeid="47106">与 Vite 的差异点</h4>
<ul data-nodeid="47107">
<li data-nodeid="47108">
<p data-nodeid="47109"><strong data-nodeid="47240">相同的功能，实现细节不同</strong>：例如对 Bare Module 的处理，除了转换后前缀名称不同外（Vite 使用 /@module/ 前缀，而 Snowpack 使用 /web_modules/ 前缀)，Vite 支持类似 "AAA/BBB" 类型的子模块引用方式，而 Snowpack 目前尚不支持。</p>
</li>
<li data-nodeid="47110">
<p data-nodeid="47111"><strong data-nodeid="47245">工具稳定性</strong>：截止写稿的时间点（2020 年 9 月 21 日），Vite 的最新版本为 v1.0.0-rc4，仍未发布第一个稳定版本。而 Snowpack 自年初发布第一个稳定版本以来，已经更新到了 v2.11.1 版本。</p>
</li>
<li data-nodeid="47112">
<p data-nodeid="47113"><strong data-nodeid="47250">插件体系</strong>：除了版本差异外，Snowpack 提供了较完善的插件体系，支持用户和社区发布自定义插件，而 Vite 虽然也内置了许多插件，但目前并没有提供自定义插件的相关文档。</p>
</li>
<li data-nodeid="47114">
<p data-nodeid="47115"><strong data-nodeid="47255">打包工具</strong>：在生产环境下，Vite 使用 Rollup 作为打包工具，而 Snowpack 则需要引入插件来实现打包功能，官方支持的打包插件有 @snowpack/plugin-webpack 和 @snowpack/plugin-parcel，暂未提供 Rollup 对应的插件。</p>
</li>
<li data-nodeid="47116">
<p data-nodeid="47117"><strong data-nodeid="47260">特殊优化</strong>：Vite 中内置了对 Vue 的大量构建优化，因此对 Vue 项目而言，选择 Vite 通常可以获得更好的开发体验。</p>
</li>
</ul>
<h3 data-nodeid="47118">无包构建与打包构建</h3>
<p data-nodeid="47119">通过上面的 Vite 等无包构建工具的功能介绍可以发现，同 Webpack 等主流打包构建工具相比，无包构建流程的优缺点都十分明显。</p>
<h4 data-nodeid="47120">无包构建的优点</h4>
<p data-nodeid="47121">无包构建的最大优势在于<strong data-nodeid="47273">构建速度快</strong>，尤其是启动服务的<strong data-nodeid="47274">初次构建速度</strong>要比目前主流的打包构建工具要快很多，原因如下：</p>
<ul data-nodeid="47122">
<li data-nodeid="47123">
<p data-nodeid="47124"><strong data-nodeid="47283">初次构建启动快</strong>：打包构建流程在初次启动时需要进行一系列的模块依赖分析与编译，而在无包构建流程中，这些工作都是<strong data-nodeid="47284">在浏览器渲染页面时异步处理的</strong>，启动服务时只需要做少量的优化处理即可（例如缓存项目依赖的 Bare Modules），所以启动非常快。</p>
</li>
<li data-nodeid="47125">
<p data-nodeid="47126"><strong data-nodeid="47289">按需编译</strong>：在打包构建流程中，启动服务时即需要完整编译打包所有模块，而无包构建流程是在浏览器渲染时，根据入口模块分析加载所需模块，编译过程按需处理，因此相比之下处理内容更少，速度也会更快</p>
</li>
<li data-nodeid="47127">
<p data-nodeid="47128"><strong data-nodeid="47294">增量构建速度快</strong>：在修改代码后的 rebuild 过程中，主流的打包构建中仍然包含编译被修改的模块和打包产物这两个主要流程，因此相比之下，只需处理编译单个模块的无包构建在速度上也会更胜一筹（尽管在打包构建工具中，也可以通过分包等方式尽可能地减少两者的差距）。</p>
</li>
</ul>
<h4 data-nodeid="47129">无包构建的缺点</h4>
<ul data-nodeid="47130">
<li data-nodeid="47131">
<p data-nodeid="47132"><strong data-nodeid="47300">浏览器网络请求数量剧增</strong>：无包构建最主要面对的问题是，它的运行模式决定了在一般项目里，渲染页面所需发起的请求数远比打包构建要多得多，使得打开页面会产生瀑布式的大量网络请求，将对页面的渲染造成延迟。这对于服务稳定性和访问性能要求更高的生产环境而言，通常是不太能接受的，尤其对不支持 HTTP/2 的服务器而言，这种处理更是灾难性的。因此，一般是在开发环境下才使用无包构建，在生产环境下则仍旧使用打包构建。</p>
</li>
<li data-nodeid="47133">
<p data-nodeid="47134"><strong data-nodeid="47305">浏览器的兼容性</strong>：无包构建要求浏览器支持 JavaScript module 特性，尽管目前的主流浏览器已大多支持，但是对于需要兼容旧浏览器的项目而言，仍然不可能在生产环境下使用。而在开发环境下则通常没有这种顾虑。</p>
</li>
</ul>
<h3 data-nodeid="47135">总结</h3>
<p data-nodeid="47136">这节课我们主要讨论了今年比较热门的无包构建。</p>
<p data-nodeid="47137">无包构建产生的基础是浏览器对 JS 模块加载的支持，这样才可能把构建过程中分析模块依赖关系并打包的过程变为在浏览器中逐个加载引用的模块。但是这种加载模块的方式在实际项目应用场景下还存在一些阻碍，于是有了无包构建工具。</p>
<p data-nodeid="47138">在这些工具里，我们主要介绍了 Vite 和 Snowpack，希望通过介绍他们的开发模式的基本工作流程和差异点，让你对这类工具的功能特点有一个基本的了解。</p>
<p data-nodeid="47139">今天的课后思考题是，为什么 Vite/Snowpack 这样的无包构建工具要比 Webpack 这样的打包构建工具速度更快呢？</p>
<p data-nodeid="56170" class="">随着这节课的结束，构建优化模块也就告一段落了。下节课开始我们将进入部署优化模块。</p>

---

### 精选评论

##### *程：
> 个人觉得是，类似webpack这种工具，是把所有代码打包成几个文件，换言之，你项目编写的代码都需要经过处理，这个很耗时间。而是像Vite/Snowpack 这种，则由浏览器进行模快处理并且按需加载

