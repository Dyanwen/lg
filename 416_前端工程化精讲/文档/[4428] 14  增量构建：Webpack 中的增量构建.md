<p data-nodeid="30091">开始课程前，我先来解答上一节课的思考题：课程中介绍的几种支持缓存的插件（TerserWebpackPlugin，CSSMinimizerWebpackPlugin）和 Loader（babel-loader，cache-loader）在缓存方面有哪些相同的配置项呢？</p>


<p data-nodeid="29271">通过对比不难发现，这些工具通常至少包含两个配置项：第一项用于指定是否开启缓存，以及指定缓存目录（值为 true 时使用默认目录，指定目录时也表示开启），配置名称通常是 cache 或 cacheDirectory；第二项用于指定缓存标识符的计算参数，通常默认值是一个包含多维度参数的对象，例如这个工具模块的版本号、配置项对象、文件路径和内容等。这个配置项是为了确保缓存使用的安全性，防止当源代码不变但相关构建参数发生变化时对旧缓存的误用。</p>
<p data-nodeid="29272">下面开始本节课的学习。曾经有同事问我一个问题：为什么我只改了一行代码，却需要花 5 分钟才能构建完成？</p>
<p data-nodeid="29273">你可能也有同样的疑问，但经过前面几节关于 Webpack 构建原理和优化的课程后，相信已经可以解答。尽管只改动了一行代码，但是在执行构建时，要完整执行所有模块的编译、优化和生成产物的处理过程，而不是只需要处理所改动的文件。大多数情况下，我们能做的是像前面几节课中讨论的那样，通过各种优化方案提升整体构建的效率。</p>
<p data-nodeid="30637" class="">但是只编译打包所改动的文件真的不能实现吗？这节课我们就来讨论这个话题（课程里完整的示例代码参见 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/14_incremental_build" data-nodeid="30645">14_incremental_build</a>）。</p>

<h3 data-nodeid="29275">Webpack 中的增量构建</h3>
<p data-nodeid="31719">上述只构建改动文件的处理过程在 Webpack 中是实际存在的，你可能也很熟悉，那就是在<strong data-nodeid="31734">开启 devServer</strong>的时候，当我们执行 webpack-dev-server 命令后，Webpack 会进行一次初始化的构建，构建完成后启动服务并进入到等待更新的状态。当本地文件有变更时，Webpack 几乎瞬间将变更的文件进行编译，并将编译后的代码内容推送到浏览器端。你会发现，这个文件变更后的处理过程就符合上面所说的只编译打包改动的文件的操作，这就称为“<strong data-nodeid="31735">增量构建”</strong>。我们通过示例代码进行验证（<em data-nodeid="31736">npm run dev</em>），如下面的图片：</p>
<p data-nodeid="32548" class=""><img src="https://s0.lgstatic.com/i/image/M00/57/0E/CgqCHl9sTsWAbetxAAGoldlDrIw704.png" alt="Drawing 0.png" data-nodeid="32551"><br>
<img src="https://s0.lgstatic.com/i/image/M00/57/03/Ciqc1F9sTsmAJc8YAADz9x_Zsvo780.png" alt="Drawing 1.png" data-nodeid="32555"></p>





<p data-nodeid="29279">可以看到，在开发服务模式下，初次构建编译了 47 个模块，完整的构建时间为 3306ms。当我们改动其中一个源码文件后，日志显示 Webpack 只再次构建了这一个模块，因此再次构建的时间非常短（24ms）。那么为什么在开发服务模式下可以实现增量构建的效果，而在生产环境下不行呢？下面我们来分析影响结果的因素。</p>
<h3 data-nodeid="29280">增量构建的影响因素</h3>
<h4 data-nodeid="29281">watch 配置</h4>
<p data-nodeid="33628">在上面的增量构建过程中，第一个想到的就是<strong data-nodeid="33647">需要监控文件的变化</strong>。显然，只有得知变更的是哪个文件后，才能进行后续的针对性处理。要实现这一点也很简单，在“第 2 课时|界面调试：热更新技术如何开着飞机修引擎？”中已经介绍过，在 Webpack 中<strong data-nodeid="33648">启用 watch 配置</strong>即可，此外在使用 devServer 的情况下，该选项会<strong data-nodeid="33649">默认开启</strong>。那么，如果在生产模式下开启 watch 配置，是不是再次构建时，就会按增量的方式执行呢？我们仍然通过示例验证（<em data-nodeid="33650">npm run build:watch</em>），如下面的图片所示：</p>
<p data-nodeid="34462" class=""><img src="https://s0.lgstatic.com/i/image/M00/57/0E/CgqCHl9sTtOAPzPRAAHMQJnGHlo474.png" alt="Drawing 2.png" data-nodeid="34465"><br>
<img src="https://s0.lgstatic.com/i/image/M00/57/0E/CgqCHl9sTtiAB2seAAG0v0B0ORQ594.png" alt="Drawing 3.png" data-nodeid="34469"></p>





<p data-nodeid="29285">从结果中可以发现，在生产模式下开启 watch 配置后，相比初次构建，再次构建所编译的模块数量并未减少，即使只改动了一个文件，也仍然会对所有模块进行编译。因此可以得出结论，在生产环境下只开启 watch 配置后的再次构建<strong data-nodeid="29400">并不能</strong>实现增量构建。</p>
<h4 data-nodeid="29286">cache 配置</h4>
<p data-nodeid="36144" class="">仔细查阅 Webpack 的配置项文档，会在菜单最下方的“其他选项”一栏中找到 <a href="https://v4.webpack.js.org/configuration/other-options/#cache" data-nodeid="36148">cache</a> 选项（需要注意的是我们查阅的是 <strong data-nodeid="36162">Webpack 4 版本的文档</strong>，Webpack 5 中这一选项会有大的改变，会在下一节课中展开讨论）。这一选项的值有两种类型：布尔值和对象类型。一般情况下默认为<strong data-nodeid="36163">false</strong>，即不使用缓存，但在开发模式开启 watch 配置的情况下，cache 的默认值变更为<strong data-nodeid="36164">true</strong>。此外，如果 cache 传值为对象类型，则表示使用该对象来作为缓存对象，这往往用于多个编译器 compiler 的调用情况。</p>



<p data-nodeid="37237">下面我们就来看一下，在生产模式下，如果watch 和 cache 都为 true，结果会如何（npm run build:watch-cache）？如下面的图片所示：</p>
<p data-nodeid="38051" class=""><img src="https://s0.lgstatic.com/i/image/M00/57/0F/CgqCHl9sTuuAc0_4AAHBe2Lt3do732.png" alt="Drawing 4.png" data-nodeid="38054"><br>
<img src="https://s0.lgstatic.com/i/image/M00/57/03/Ciqc1F9sTvCAY2NvAAEtJYxCA_8121.png" alt="Drawing 5.png" data-nodeid="38058"></p>





<p data-nodeid="29291">正如我们所期望的，再次构建时，在编译模块阶段只对有变化的文件进行了重新编译，实现了<strong data-nodeid="29434">增量编译</strong>的效果。</p>
<p data-nodeid="29292">但是美中不足的是，在优化阶段压缩代码时仍然耗费了较多的时间。这一点很容易理解：</p>
<p data-nodeid="39131">体积最大的 react、react-dom 等模块和入口模块打入了同一个 Chunk 中，即使修改的模块是单独分离的 bar.js，但它的产物名称的变化仍然需要反映在入口 Chunk 的 runtime 模块中。因此入口 Chunk 也需要跟着重新压缩而无法复用压缩缓存数据。根据前面几节课的知识点，我们对配置再做一些优化，将 vendor 分离后再来看看效果，如下面的图片所示：</p>
<p data-nodeid="39945" class=""><img src="https://s0.lgstatic.com/i/image/M00/57/0F/CgqCHl9sTvqAP1oIAAG2kbb-DGY688.png" alt="Drawing 6.png" data-nodeid="39948"><br>
<img src="https://s0.lgstatic.com/i/image/M00/57/0F/CgqCHl9sTv6AYxTKAAFAsmUEZMg953.png" alt="Drawing 7.png" data-nodeid="39952"></p>





<p data-nodeid="29296">可以看到，通过上面这一系列的配置后（<strong data-nodeid="29452">watch + cache</strong>），在生产模式下，最终呈现出了我们期望的<strong data-nodeid="29453">增量构建</strong>效果：有文件发生变化时会自动编译变更的模块，并只对该模块影响到的少量 Chunk 进行优化并更新产物文件版本，而其他产物文件则保持之前的版本。如此，整个构建过程的速度大大提升。</p>
<h3 data-nodeid="29297">增量构建的实现原理</h3>
<p data-nodeid="29298">为什么在配置项中需要同时启用 watch 和 cache 配置才能获得增量构建的效果呢？接下来我们从源码层面分析。</p>
<h4 data-nodeid="29299">watch 配置的作用</h4>
<p data-nodeid="41049" class="">watch 配置的具体逻辑在 Webpack 的 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/Watching.js" data-nodeid="41053">Watching.js</a> 中。查看源码可以看到，在它构建相关的 _go 方法中，执行的依然是 compiler实例的 compile 方法，这一点与普通构建流程并无区别。真正的区别在于，在 watch 模式下，构建完成后并不自动退出，因此构建上下文的对象（包括前一次构建后的缓存数据对象）都可以保留在内存中，并在 rebuild 时重复使用，如下面的代码所示：</p>


<pre class="lang-javascript" data-nodeid="29301"><code data-language="javascript">lib/Watching.js
...
_go() {
  ...
  this.compiler.hooks.watchRun.callAsync(<span class="hljs-built_in">this</span>.compiler, <span class="hljs-function"><span class="hljs-params">err</span> =&gt;</span> {
    <span class="hljs-keyword">const</span> onCompiled = <span class="hljs-function">(<span class="hljs-params">err, compilation</span>) =&gt;</span> {
      ...
    }
    <span class="hljs-built_in">this</span>.compiler.compile(onCompiled);
  }
}
</code></pre>
<h4 data-nodeid="41601">cache 配置的作用</h4>


<p data-nodeid="42697" class="">cache 配置的源码逻辑主要涉及两个文件：<a href="https://github.com/webpack/webpack/blob/webpack-4/lib/CachePlugin.js" data-nodeid="42701">CachePlugin.js</a> 和 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/Compilation.js" data-nodeid="42705">Compilation.js</a>。其中 CachePlugin.js 的核心作用是将该插件实例的 cache 属性传入 compilation 实例中，如下面的代码所示：</p>


<pre class="lang-javascript" data-nodeid="29305"><code data-language="javascript">lib/CachePlugin.js
...
compiler.hooks.thisCompilation.tap(<span class="hljs-string">"CachePlugin"</span>, <span class="hljs-function"><span class="hljs-params">compilation</span> =&gt;</span> {
  compilation.cache = cache;
  ...
}
</code></pre>
<p data-nodeid="29306">而在 Compilation.js 中，运用 cache 的地方有两处：</p>
<ol data-nodeid="29307">
<li data-nodeid="29308">
<p data-nodeid="29309">在<strong data-nodeid="29484">编译阶段添加模块时</strong>，若命中缓存<strong data-nodeid="29485">module</strong>，则直接跳过该模块的编译过程（与 cache-loader 等作用于加载器的缓存不同，此处的缓存可直接跳过 Webpack 内置的编译阶段）。</p>
</li>
<li data-nodeid="29310">
<p data-nodeid="29311">在创建 Chunk 产物代码阶段，若命中缓存<strong data-nodeid="29491">Chunk</strong>，则直接跳过该 Chunk 的产物代码生成过程。</p>
</li>
</ol>
<p data-nodeid="29312">如下面的代码所示：</p>
<pre class="lang-javascript" data-nodeid="29313"><code data-language="javascript">lib/Compilation.js
...
addModule(<span class="hljs-built_in">module</span>, cacheGroup) {
  ...
  if (<span class="hljs-built_in">this</span>.cache &amp;&amp; <span class="hljs-built_in">this</span>.cache[cacheName]) {
    <span class="hljs-keyword">const</span> cacheModule = <span class="hljs-built_in">this</span>.cache[cacheName];
    ...
    <span class="hljs-comment">//缓存模块存在情况下判断是否需要rebuild</span>
    rebuild = ...
    if (!rebuild) {
      ...
      <span class="hljs-comment">//无须rebuild情况下返回cacheModule，并标记build:false</span>
      <span class="hljs-keyword">return</span> {
		<span class="hljs-attr">module</span>: cacheModule,
		<span class="hljs-attr">issuer</span>: <span class="hljs-literal">true</span>,
		<span class="hljs-attr">build</span>: <span class="hljs-literal">false</span>,
		<span class="hljs-attr">dependencies</span>: <span class="hljs-literal">true</span>
	  }      
    }
    ...
  }
  <span class="hljs-keyword">if</span> (<span class="hljs-built_in">this</span>.cache) {
    <span class="hljs-built_in">this</span>.cache[cacheName] = <span class="hljs-built_in">module</span>;
  }
  ...
  <span class="hljs-comment">//无缓存或需要rebuild情况下返回module，并标记build:true</span>
  <span class="hljs-keyword">return</span> {
	<span class="hljs-attr">module</span>: <span class="hljs-built_in">module</span>,
	<span class="hljs-attr">issuer</span>: <span class="hljs-literal">true</span>,
	<span class="hljs-attr">build</span>: <span class="hljs-literal">true</span>,
	<span class="hljs-attr">dependencies</span>: <span class="hljs-literal">true</span>
  };
}
...
createChunkAssets() {
  ...
  if ( <span class="hljs-built_in">this</span>.cache &amp;&amp; <span class="hljs-built_in">this</span>.cache[cacheName] &amp;&amp; <span class="hljs-built_in">this</span>.cache[cacheName].hash === usedHash ) {
    source = <span class="hljs-built_in">this</span>.cache[cacheName].source;
  } <span class="hljs-keyword">else</span> {
	source = fileManifest.render();
    ...
  }
}
</code></pre>
<p data-nodeid="29314">以上就是 Webpack 4 中 watch 和 cache 配置的作用原理。通过 Webpack 内置的 cache 插件，将整个构建中相对耗时的两个内部处理环节——编译模块和生成产物，进行缓存的读写处理，从而实现增量构建处理。那么我们是不是就可以在生产环境下直接使用这个方案呢？</p>
<h3 data-nodeid="29315">生产环境下使用增量构建的阻碍</h3>
<p data-nodeid="29316">增量构建之所以快是因为将构建所需的数据（项目文件、node_modules 中的文件数据、历史构建后的缓存数据等）都<strong data-nodeid="29502">保留在内存中</strong>。在 watch 模式下保留着构建使用的 Node 进程，使得下一次构建时可以直接读取内存中的数据。</p>
<p data-nodeid="29317">而生产环境下的构建通常在集成部署系统中进行。对于管理多项目的构建系统而言，构建过程是任务式的：任务结束后即结束进程并回收系统资源。对于这样的系统而言，增量构建所需的保留进程与长时间占用内存，通常都是<strong data-nodeid="29508">不可接受的</strong>。</p>
<p data-nodeid="29318">因此，基于内存的缓存数据注定无法运用到生产环境中。要想在生产环境下提升构建速度，<strong data-nodeid="29514">首要条件是将缓存写入到文件系统中</strong>。只有将文件系统中的缓存数据持久化，才能脱离对保持进程的依赖，你只需要在每次构建时将缓存数据读取到内存中进行处理即可。事实上，这也是上一课时中讲到的那些 Loader 与插件中的缓存数据的存储方式。</p>
<p data-nodeid="29319">遗憾的是，Webpack 4 中的 cache 配置<strong data-nodeid="29524">只支持基于内存的缓存</strong>，并不支持文件系统的缓存。因此，我们只能通过上节课讲到的一些支持缓存的第三方处理插件将局部的构建环节应用“<strong data-nodeid="29525">增量处理</strong>”。</p>
<p data-nodeid="29320">不过好消息是 Webpack 5 中<strong data-nodeid="29531">正式支持基于文件系统的持久化缓存</strong>（Persistent Cache）。我们会在下一课时详细讨论包括这一特性在内的 Webpack 5 中的优化点。</p>
<h3 data-nodeid="29321">总结</h3>
<p data-nodeid="29322">这节课我们主要讨论了构建处理的一种理想情况：增量构建。增量构建在每次执行构建时，只编译处理内容有修改的少量文件，从而极大地提升构建效率。</p>
<p data-nodeid="29323">在 Webpack 4 中，有两个配置项与增量构建相关：watch 和 cache。当我们启用开发服务器时，这两个选项都是默认启用的，因此可以在开发模式下体验到增量构建带来的速度提升。</p>
<p data-nodeid="29324">从内部原理的角度分析，watch 的作用是保留进程，使得初次构建后的数据对象能够在再次构建时复用。而 cache 的作用则体现在构建过程中，在添加模块与生成产物代码时可以利用 cache 对象进行相应阶段结果数据的读写。显然，这种基于内存的缓存方式无法在生产环境下广泛使用。</p>
<p data-nodeid="43789">今天的<strong data-nodeid="43795">课后思考题</strong>是：在启用增量构建的情况下有时候可能还会遇到 rebuild 很慢的情况，试着分析原因。</p>

---

### 精选评论


