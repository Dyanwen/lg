<p data-nodeid="1177">上节课我们聊了过去 20 余年里，前端项目开发时的工程化需求，以及对应产生的工具解决方案，其中最广泛运用的构建工具是 Webpack。这节课我们就来深入分析 Webpack 中的效率优化问题。</p>


<p data-nodeid="3">要想全面地分析 Webpack 构建工具的优化方案，首先要先对它的工作流程有一定理解，这样才能针对项目中可能存在的构建问题，进行有目标地分析和优化。</p>
<h3 data-nodeid="4">Webpack 的基本工作流程</h3>
<p data-nodeid="5">我们从两方面来了解 Webpack 的基本工作流程：</p>
<ol data-nodeid="6">
<li data-nodeid="7">
<p data-nodeid="8">通过 Webpack 的源码来了解具体函数执行的逻辑。</p>
</li>
<li data-nodeid="9">
<p data-nodeid="10">通过 Webpack 对外暴露的声明周期 Hooks，理解整体流程的阶段划分。</p>
</li>
</ol>
<p data-nodeid="1959" class="">其中会涉及对 Webpack 源代码的分析，源代码取自 Webpack 仓库的 <a href="https://github.com/webpack/webpack/blob/webpack-4" data-nodeid="1963">webpack-4 分支</a>，而最新的 Webpack 5 中的优化我们会在后续课程中单独分析。</p>

<p data-nodeid="12">通常，在项目中有两种运行 Webpack 的方式：基于命令行的方式或基于代码的方式。</p>
<p data-nodeid="2745" class="">两种示例的代码分别如下（具体示例参照 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/10_webpack_workflow" data-nodeid="2753">10_webpack_workflow</a>）：</p>

<pre class="lang-javascript" data-nodeid="14"><code data-language="javascript"><span class="hljs-comment">//第一种：基于命令行的方式</span>
webpack --config webpack.config.js
<span class="hljs-comment">//第二种：基于代码的方式</span>
<span class="hljs-keyword">var</span> webpack = <span class="hljs-built_in">require</span>(<span class="hljs-string">'webpack'</span>)
<span class="hljs-keyword">var</span> config = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./webpack.config'</span>)
webpack(config, <span class="hljs-function">(<span class="hljs-params">err, stats</span>) =&gt;</span> {})
</code></pre>
<h4 data-nodeid="15">webpack.js 中的基本流程</h4>
<p data-nodeid="4321" class="">无论用哪种方式运行 Webpack，本质上都是 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/webpack.js" data-nodeid="4325">webpack.js</a> 中的 Webpack 函数。</p>


<p data-nodeid="17">这一函数的核心逻辑是：根据配置生成编译器实例 compiler，然后处理参数，执行 WebpackOptionsApply().process，根据参数加载不同内部插件。在有回调函数的情况下，根据是否是 watch 模式来决定要执行 compiler.watch 还是 compiler.run。</p>
<p data-nodeid="18">为了讲解通用的流程，我们以没有 watch 模式的情况进行分析。简化流程后的代码示例如下：</p>
<pre class="lang-javascript" data-nodeid="19"><code data-language="javascript"><span class="hljs-keyword">const</span> webpack = <span class="hljs-function">(<span class="hljs-params">options, callback</span>) =&gt;</span> {
  options = ... <span class="hljs-comment">//处理options默认值</span>
  <span class="hljs-keyword">let</span> compiler = <span class="hljs-keyword">new</span> Compiler(options.context)
  ... <span class="hljs-comment">//处理参数中的插件等</span>
  compiler.options = <span class="hljs-keyword">new</span> WebpackOptionsApply().process(options, compiler); <span class="hljs-comment">//分析参数，加载各内部插件</span>
  ...
  if (callback) {
    ... 
    compiler.run(callback)
  }
  <span class="hljs-keyword">return</span> compiler
}
</code></pre>
<h4 data-nodeid="20">Compiler.js 中的基本流程</h4>
<p data-nodeid="5893" class="">我们再来看下运行编译器实例的内部逻辑，具体源代码在 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/Compiler.js" data-nodeid="5897">Compiler.js</a> 中。</p>


<p data-nodeid="22">compiler.run(callback) 中的执行逻辑较为复杂，我们把它按流程抽象一下。抽象后的执行流程如下：</p>
<ol data-nodeid="9235">
<li data-nodeid="9236">
<p data-nodeid="9237"><strong data-nodeid="9263">readRecords</strong>：读取<a href="https://webpack.js.org/configuration/other-options/#recordspath" data-nodeid="9261">构建记录</a>，用于分包缓存优化，在未设置 recordsPath 时直接返回。</p>
</li>
<li data-nodeid="9238">
<p data-nodeid="9239"><strong data-nodeid="9268">compile 的主要构建过程</strong>，涉及以下几个环节：</p>
<ol data-nodeid="9240">
<li data-nodeid="9241" class="">
<p data-nodeid="9242"><strong data-nodeid="9273">newCompilationParams</strong>：创建 NormalModule 和 ContextModule 的工厂实例，用于创建后续模块实例。</p>
</li>
<li data-nodeid="9243">
<p data-nodeid="9244"><strong data-nodeid="9278">newCompilation</strong>：创建编译过程 Compilation 实例，传入上一步的两个工厂实例作为参数。</p>
</li>
<li data-nodeid="9245">
<p data-nodeid="9246"><strong data-nodeid="9287">compiler.hooks.make.callAsync</strong>：触发 make 的 Hook，执行所有监听 make 的插件（例如 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/SingleEntryPlugin.js" data-nodeid="9285">SingleEntryPlugin.js</a> 中，会在相应的监听中触发 compilation 的 addEntry 方法）。其中，Hook 的作用，以及其他 Hook 会在下面的小节中再谈到。</p>
</li>
<li data-nodeid="9247">
<p data-nodeid="9248"><strong data-nodeid="9292">compilation.finish</strong>：编译过程实例的 finish 方法，触发相应的 Hook 并报告构建模块的错误和警告。</p>
</li>
<li data-nodeid="9249">
<p data-nodeid="9250"><strong data-nodeid="9297">compilation.seal</strong>：编译过程的 seal 方法，下一节中我会进一步分析。</p>
</li>
</ol>
</li>
<li data-nodeid="9251">
<p data-nodeid="9252"><strong data-nodeid="9302">emitAssets</strong>：调用 compilation.getAssets()，将产物内容写入输出文件中。</p>
</li>
<li data-nodeid="9253">
<p data-nodeid="9254"><strong data-nodeid="9307">emitRecords</strong>：对应第一步的 readRecords，用于写入构建记录，在未设置 recordsPath 时直接返回。</p>
</li>
</ol>




<p data-nodeid="43">在编译器运行的流程里，核心过程是第二步编译。具体流程在生成的 Compilation 实例中进行，接下来我们再来看下这部分的源码逻辑。</p>
<h4 data-nodeid="44">Compilation.js 中的基本流程</h4>
<p data-nodeid="10874" class="">这部分的源码位于 <a href="https://github.com/webpack/webpack/blob/webpack-4/lib/Compilation.js" data-nodeid="10878">Compilation.js</a> 中。其中，在编译执行过程中，我们主要从外部调用的是两个方法：</p>


<ol data-nodeid="46">
<li data-nodeid="47">
<p data-nodeid="48"><strong data-nodeid="261">addEntry</strong>：从 entry 开始递归添加和构建模块。</p>
</li>
<li data-nodeid="49">
<p data-nodeid="50"><strong data-nodeid="266">seal</strong>：冻结模块，进行一系列优化，以及触发各优化阶段的 Hooks。</p>
</li>
</ol>
<p data-nodeid="51">以上就是执行 Webpack 构建时的基本流程，这里再稍做总结：</p>
<ol data-nodeid="52">
<li data-nodeid="53">
<p data-nodeid="54">创建编译器 Compiler 实例。</p>
</li>
<li data-nodeid="55">
<p data-nodeid="56">根据 Webpack 参数加载参数中的插件，以及程序内置插件。</p>
</li>
<li data-nodeid="57">
<p data-nodeid="58">执行编译流程：创建编译过程 Compilation 实例，从入口递归添加与构建模块，模块构建完成后冻结模块，并进行优化。</p>
</li>
<li data-nodeid="59">
<p data-nodeid="60">构建与优化过程结束后提交产物，将产物内容写到输出文件中。</p>
</li>
</ol>
<p data-nodeid="61">除了了解上面的基本工作流程外，还有两个相关的概念需要理解：Webpack 的生命周期和插件系统。</p>
<h3 data-nodeid="62">读懂 Webpack 的生命周期</h3>
<p data-nodeid="63">Webpack 工作流程中最核心的两个模块：Compiler 和 Compilation 都扩展自 Tapable 类，用于实现工作流程中的生命周期划分，以便在不同的生命周期节点上注册和调用<strong data-nodeid="283">插件</strong>。其中所暴露出来的生命周期节点称为<strong data-nodeid="284">Hook</strong>（俗称钩子）。</p>
<h4 data-nodeid="64">Webpack 中的插件</h4>
<p data-nodeid="65">Webpack 引擎基于插件系统搭建而成，不同的插件各司其职，在 Webpack 工作流程的某一个或多个时间点上，对构建流程的某个方面进行处理。Webpack 就是通过这样的工作方式，在各生命周期中，经一系列插件将源代码逐步变成最后的产物代码。</p>
<p data-nodeid="66">一个 Webpack 插件是一个包含 apply 方法的 JavaScript 对象。这个 apply 方法的执行逻辑，通常是注册 Webpack 工作流程中某一生命周期 Hook，并添加对应 Hook 中该插件的实际处理函数。例如下面的代码：</p>
<pre class="lang-javascript" data-nodeid="67"><code data-language="javascript"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HelloWorldPlugin</span> </span>{
  apply(compiler) {
    compiler.hooks.run.tap(<span class="hljs-string">"HelloWorldPlugin"</span>, <span class="hljs-function"><span class="hljs-params">compilation</span> =&gt;</span> {
      <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'hello world'</span>);
    })
  }
}
<span class="hljs-built_in">module</span>.exports = HelloWorldPlugin;
</code></pre>
<h4 data-nodeid="68">Hook 的使用方式</h4>
<p data-nodeid="69">Hook 的使用分为四步：</p>
<ol data-nodeid="70">
<li data-nodeid="71">
<p data-nodeid="72">在构造函数中定义 Hook 类型和参数，生成 Hook 对象。</p>
</li>
<li data-nodeid="73">
<p data-nodeid="74">在插件中注册 Hook，添加对应 Hook 触发时的执行函数。</p>
</li>
<li data-nodeid="75">
<p data-nodeid="76">生成插件实例，运行 apply 方法。</p>
</li>
<li data-nodeid="77">
<p data-nodeid="78">在运行到对应生命周期节点时调用 Hook，执行注册过的插件的回调函数。如下面的代码所示：</p>
</li>
</ol>
<pre class="lang-javascript" data-nodeid="79"><code data-language="javascript">lib/Compiler.js
<span class="hljs-built_in">this</span>.hooks = {
  ...
  make: <span class="hljs-keyword">new</span> SyncHook([<span class="hljs-string">'compilation'</span>, <span class="hljs-string">'params'</span>]), <span class="hljs-comment">//1. 定义Hook</span>
  ...
}
...
this.hooks.compilation.call(compilation, params); <span class="hljs-comment">//4. 调用Hook</span>
...
lib/dependencies/CommonJsPlugin.js
<span class="hljs-comment">//2. 在插件中注册Hook</span>
compiler.hooks.compilation.tap(<span class="hljs-string">"CommonJSPlugin"</span>, <span class="hljs-function">(<span class="hljs-params">compilation, { contextModuleFactory, normalModuleFactory }</span>) =&gt;</span> {
  ...
})
lib/WebpackOptionsApply.js
<span class="hljs-comment">//3. 生成插件实例，运行apply方法</span>
<span class="hljs-keyword">new</span> CommonJsPlugin(options.module).apply(compiler);
</code></pre>
<p data-nodeid="80">以上就是 Webpack 中 Hook 的一般使用方式。正是通过这种方式，Webpack 将编译器和编译过程的生命周期节点提供给外部插件，从而搭建起弹性化的工作引擎。</p>
<p data-nodeid="81">Hook 的类型按照同步或异步、是否接收上一插件的返回值等情况分为 9 种。不同类型的 Hook 接收注册的方法也不同，更多信息可参照<a href="https://github.com/webpack/tapable#tapable" data-nodeid="298">官方文档</a>。下面我们来具体介绍 Compiler 和 Compilation 中的 Hooks。</p>
<h4 data-nodeid="82">Compiler Hooks</h4>
<p data-nodeid="83">构建器实例的生命周期可以分为 3 个阶段：初始化阶段、构建过程阶段、产物生成阶段。下面我们就来大致介绍下这些不同阶段的 Hooks ：</p>
<p data-nodeid="84"><strong data-nodeid="305">初始化阶段</strong></p>
<ul data-nodeid="85">
<li data-nodeid="86">
<p data-nodeid="87">environment、afterEnvironment：在创建完 compiler 实例且执行了配置内定义的插件的 apply 方法后触发。</p>
</li>
<li data-nodeid="88">
<p data-nodeid="89">entryOption、afterPlugins、afterResolvers：在 WebpackOptionsApply.js 中，这 3 个 Hooks 分别在执行 EntryOptions 插件和其他 Webpack 内置插件，以及解析了 resolver 配置后触发。</p>
</li>
</ul>
<p data-nodeid="90"><strong data-nodeid="311">构建过程阶段</strong></p>
<ul data-nodeid="91">
<li data-nodeid="92">
<p data-nodeid="93">normalModuleFactory、contextModuleFactory：在两类模块工厂创建后触发。</p>
</li>
<li data-nodeid="94">
<p data-nodeid="95">beforeRun、run、watchRun、beforeCompile、compile、thisCompilation、compilation、make、afterCompile：在运行构建过程中触发。</p>
</li>
</ul>
<p data-nodeid="96"><strong data-nodeid="317">产物生成阶段</strong></p>
<ul data-nodeid="97">
<li data-nodeid="98">
<p data-nodeid="99">shouldEmit、emit、assetEmitted、afterEmit：在构建完成后，处理产物的过程中触发。</p>
</li>
<li data-nodeid="100">
<p data-nodeid="101">failed、done：在达到最终结果状态时触发。</p>
</li>
</ul>
<h4 data-nodeid="102">Compilation Hooks</h4>
<p data-nodeid="103">构建过程实例的生命周期我们分为两个阶段：</p>
<p data-nodeid="104"><strong data-nodeid="325">构建阶段</strong></p>
<ul data-nodeid="105">
<li data-nodeid="106">
<p data-nodeid="107">addEntry、failedEntry、succeedEntry：在添加入口和添加入口结束时触发（Webpack 5 中移除）。</p>
</li>
<li data-nodeid="108">
<p data-nodeid="109">buildModule、rebuildModule、finishRebuildingModule、failedModule、succeedModule：在构建单个模块时触发。</p>
</li>
<li data-nodeid="110">
<p data-nodeid="111">finishModules：在所有模块构建完成后触发。</p>
</li>
</ul>
<p data-nodeid="112"><strong data-nodeid="332">优化阶段</strong></p>
<p data-nodeid="12424">优化阶段在 seal 函数中共有 12 个主要的处理过程，如下图所示：</p>
<p data-nodeid="12425" class=""><img src="https://s0.lgstatic.com/i/image/M00/4D/B4/Ciqc1F9bGtqAJo4uAABnYGwsyYs218.png" alt="image (4).png" data-nodeid="12433"></p>



<p data-nodeid="116">每个过程都暴露了相应的 Hooks，分别如下:</p>
<ul data-nodeid="13214">
<li data-nodeid="13215">
<p data-nodeid="13216">seal、needAdditionalSeal、unseal、afterSeal：分别在 seal 函数的起始和结束的位置触发。</p>
</li>
<li data-nodeid="13217">
<p data-nodeid="13218">optimizeDependencies、afterOptimizeDependencies：触发优化依赖的插件执行，例如FlagDependencyUsagePlugin。</p>
</li>
<li data-nodeid="13219">
<p data-nodeid="13220">beforeChunks、afterChunks：分别在生成 Chunks 的过程的前后触发。</p>
</li>
<li data-nodeid="13221">
<p data-nodeid="13222">optimize：在生成 chunks 之后，开始执行优化处理的阶段触发。</p>
</li>
<li data-nodeid="13223">
<p data-nodeid="13224">optimizeModule、afterOptimizeModule：在优化模块过程的前后触发。</p>
</li>
<li data-nodeid="13225">
<p data-nodeid="13226" class="">optimizeChunks、afterOptimizeChunks：在优化 Chunk 过程的前后触发，用于 <a href="https://webpack.js.org/guides/tree-shaking/" data-nodeid="13253">Tree Shaking</a>。</p>
</li>
<li data-nodeid="13227">
<p data-nodeid="13228">optimizeTree、afterOptimizeTree：在优化模块和 Chunk 树过程的前后触发。</p>
</li>
<li data-nodeid="13229">
<p data-nodeid="13230">optimizeChunkModules、afterOptimizeChunkModules：在优化 ChunkModules 的过程前后触发，例如 ModuleConcatenationPlugin，利用这一 Hook 来做<a href="https://webpack.js.org/plugins/module-concatenation-plugin/#optimization-bailouts" data-nodeid="13259">Scope Hoisting</a>的优化。</p>
</li>
<li data-nodeid="13231">
<p data-nodeid="13232">shouldRecord、recordModules、recordChunks、recordHash：在 shouldRecord 返回为 true 的情况下，依次触发 recordModules、recordChunks、recordHash。</p>
</li>
<li data-nodeid="13233">
<p data-nodeid="13234">reviveModules、beforeModuleIds、moduleIds、optimizeModuleIds、afterOptimizeModuleIds：在生成模块 Id 过程的前后触发。</p>
</li>
<li data-nodeid="13235">
<p data-nodeid="13236">reviveChunks、beforeChunkIds、optimizeChunkIds、afterOptimizeChunkIds：在生成 Chunk id 过程的前后触发。</p>
</li>
<li data-nodeid="13237">
<p data-nodeid="13238">beforeHash、afterHash：在生成模块与 Chunk 的 hash 过程的前后触发。</p>
</li>
<li data-nodeid="13239">
<p data-nodeid="13240">beforeModuleAssets、moduleAsset：在生成模块产物数据过程的前后触发。</p>
</li>
<li data-nodeid="13241">
<p data-nodeid="13242">shouldGenerateChunkAssets、beforeChunkAssets、chunkAsset：在创建 Chunk 产物数据过程的前后触发。</p>
</li>
<li data-nodeid="13243">
<p data-nodeid="13244">additionalAssets、optimizeChunkAssets、afterOptimizeChunkAssets、optimizeAssets、afterOptimizeAssets：在优化产物过程的前后触发，例如在 TerserPlugin 的<a href="https://github.com/webpack-contrib/terser-webpack-plugin/blob/master/src/index.js" data-nodeid="13270">压缩代码</a>插件的执行过程中，就用到了 optimizeChunkAssets。</p>
</li>
</ul>

<h3 data-nodeid="148">代码实践：编写一个简单的统计插件</h3>
<p data-nodeid="149">在了解了 Webpack 的工作流程后，下面我们进行一个简单的实践。</p>
<p data-nodeid="14052" class="">编写一个统计构建过程生命周期耗时的插件，这类插件会作为后续优化构建效率的准备工作。插件片段示例如下（完整代码参见 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/10_webpack_workflow" data-nodeid="14060">10_webpack_workflow</a>）：</p>

<pre class="lang-java" data-nodeid="17251"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SamplePlugin</span> </span>{
  apply(compiler) {
    <span class="hljs-keyword">var</span> start = Date.now()
    <span class="hljs-keyword">var</span> statsHooks = [<span class="hljs-string">'environment'</span>, <span class="hljs-string">'entryOption'</span>, <span class="hljs-string">'afterPlugins'</span>, <span class="hljs-string">'compile'</span>]
    <span class="hljs-keyword">var</span> statsAsyncHooks = [
      <span class="hljs-string">'beforeRun'</span>,
      <span class="hljs-string">'beforeCompile'</span>,
      <span class="hljs-string">'make'</span>,
      <span class="hljs-string">'afterCompile'</span>,
      <span class="hljs-string">'emit'</span>,
      <span class="hljs-string">'done'</span>,
    ]

    statsHooks.forEach((hookName) =&gt; {
      compiler.hooks[hookName].tap(<span class="hljs-string">'Sample Plugin'</span>, () =&gt; {
        console.log(`Compiler Hook ${hookName}, Time: ${Date.now() - start}ms`)
      })
    })
    ...
  }
})
<span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = SamplePlugin;
</code></pre>
<p data-nodeid="18826">执行构建后，可以看到在控制台输出了相应的统计时间结果（这里的时间是从构建起始到各阶段 Hook 触发为止的耗时），如下图所示：</p>
<p data-nodeid="18827" class=""><img src="https://s0.lgstatic.com/i/image/M00/4D/B4/Ciqc1F9bGvGAFRmpAAGFrvBhTHE475.png" alt="image (5).png" data-nodeid="18835"></p>


<p data-nodeid="17254">根据这样的输出结果，我们就可以分析项目里各阶段的耗时情况，再进行针对性地优化。这个统计插件将在后面几课的优化实践中运用。</p>
<p data-nodeid="17255">除了这类自己编写的统计插件外，Webpack 社区中也有一些较成熟的统计插件，例如<a href="https://github.com/stephencookdev/speed-measure-webpack-plugin" data-nodeid="17268">speed-measure-webpack-plugin</a>等，感兴趣的话，你可以进一步了解。</p>
<h3 data-nodeid="17256">总结</h3>
<p data-nodeid="17257">这一课时起，我们进入了 Webpack 构建优化的主题。在这节课中，我主要为你勾画了一个 Webpack 工作流程的轮廓，通过对三个源码文件的分析，让你对执行构建命令后的内部流程有一个基本概念。然后我们讨论了 Compiler 和 Compilation 工作流程中的生命周期 Hooks，以及插件的基本工作方式。最后，我们编写了一个简单的统计插件，用于实践上面所讲的课程内容。</p>
<p data-nodeid="17258">今天的课后思考题是：在今天介绍的 Compiler 和 Compilation 的各生命周期阶段里，通常耗时最长的分别是哪个阶段呢？可以结合自己所在的项目测试分析一下。</p>

---

### 精选评论

##### *忠：
> 看不太懂，不应该从入口开始分析吗，一步一步，loader怎么用起来的，plugin会在什么时机执行，依赖的模块是怎么被加载进来的，列一堆API并解释其含义，很难把这些流程串起来。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个主题的切入角度是效率优化方面的，篇幅关系不会包含所有的webpack中所有内容的工作原理，例如loader部分我们并不关注它的内部工作过程，而是在后面课程中会讲到如何利用一些loader的缓存机制。而对于plugin的执行时机，则正是课程中提到的相关hook决定的。只看这一课的内容觉得不容易理解也没关系，后面几课会结合实际优化的示例来帮助进一步理解。

##### **棋：
> 非常棒，先对 webpack 工作有一个大体的了解，然后再了解里面的 Hook😀😀😀

