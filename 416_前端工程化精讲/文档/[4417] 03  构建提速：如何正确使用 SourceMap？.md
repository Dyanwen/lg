<p data-nodeid="2441">在上一课时中我们聊了开发时的热更新机制和其中的技术细节，热更新能帮助我们在开发时快速预览代码效果，免去了手动执行编译和刷新浏览器的操作。课后留的思考题是找一个实现了 HMR 的 Loader，查看其中用到哪些热替换的 API，希望通过这个题目能让你加深对相关知识点的印象。</p>
<p data-nodeid="2442">那么除了热更新以外，项目的开发环境还有哪些在影响着我们的开发效率呢？在过去的工作中，公司同事就曾问过我一个问题：为什么我的项目在开发环境下每次构建还是很卡？每次保存完代码都要过 1~2 秒才能看到效果，这是怎么回事呢？其实这里面的原因主要是这位同事在开发时选择的 Source Map 设定不对。今天我们就来具体讨论下这个问题。首先，什么是 <strong data-nodeid="2561">Source Map</strong> 呢？</p>
<h3 data-nodeid="2443">什么是 Source Map</h3>
<p data-nodeid="2444">在前端开发过程中，通常我们编写的源代码会经过多重处理（编译、封装、压缩等），最后形成产物代码。于是在浏览器中调试产物代码时，我们往往会发现代码变得面目全非，例如：</p>
<p data-nodeid="2445"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/42/93/Ciqc1F85_FmAA4UeAABWNiHqsWQ595.png" data-nodeid="2566"></p>
<p data-nodeid="2446">因此，我们需要一种在调试时将产物代码显示回源代码的功能，<strong data-nodeid="2572">source map</strong> 就是实现这一目标的工具。</p>
<p data-nodeid="2447">source-map 的基本原理是，在编译处理的过程中，在生成产物代码的同时生成产物代码中被转换的部分与源代码中相应部分的映射关系表。有了这样一张完整的映射表，我们就可以通过 Chrome 控制台中的"Enable Javascript source map"来实现调试时的显示与定位源代码功能。</p>
<p data-nodeid="2448">对于同一个源文件，根据不同的目标，可以生成不同效果的 source map。它们在<strong data-nodeid="2595">构建速度</strong>、<strong data-nodeid="2596">质量</strong>（反解代码与源代码的接近程度以及调试时行号列号等辅助信息的对应情况）、<strong data-nodeid="2597">访问方式</strong>（在产物文件中或是单独生成 source map 文件）和<strong data-nodeid="2598">文件大小</strong>等方面各不相同。在开发环境和生产环境下，我们对于 source map 功能的期望也有所不同：</p>
<ul data-nodeid="2449">
<li data-nodeid="2450">
<p data-nodeid="2451"><strong data-nodeid="2603">在开发环境中</strong>，通常我们关注的是构建速度快，质量高，以便于提升开发效率，而不关注生成文件的大小和访问方式。</p>
</li>
<li data-nodeid="2452">
<p data-nodeid="2453"><strong data-nodeid="2608">在生产环境中</strong>，通常我们更关注是否需要提供线上 source map , 生成的文件大小和访问方式是否会对页面性能造成影响等，其次才是质量和构建速度。</p>
</li>
</ul>
<h3 data-nodeid="3643">Webpack 中的 source map 预设</h3>


<p data-nodeid="5279">在 Webpack 中，通过设置 devtool 来选择 source map 的预设类型，文档中共有 <a href="https://webpack.js.org/configuration/devtool/#devtool" data-nodeid="5283">20 余种</a> source map 的预设（注意：其中部分预设实际效果与其他预设相同，即页面表格中空白行条目）可供选择，这些预设通常包含了 "eval" "cheap" "module" "inline" "hidden" "nosource" "source-map" 等关键字的组合，这些关键字的具体逻辑如下：</p>


<pre class="lang-javascript" data-nodeid="2456"><code data-language="javascript">webpack/lib/WebpackOptionsApply.js:<span class="hljs-number">232</span> 
<span class="hljs-keyword">if</span> (options.devtool.includes(<span class="hljs-string">"source-map"</span>)) { 
  <span class="hljs-keyword">const</span> hidden = options.devtool.includes(<span class="hljs-string">"hidden"</span>); 
  <span class="hljs-keyword">const</span> inline = options.devtool.includes(<span class="hljs-string">"inline"</span>); 
  <span class="hljs-keyword">const</span> evalWrapped = options.devtool.includes(<span class="hljs-string">"eval"</span>); 
  <span class="hljs-keyword">const</span> cheap = options.devtool.includes(<span class="hljs-string">"cheap"</span>); 
  <span class="hljs-keyword">const</span> moduleMaps = options.devtool.includes(<span class="hljs-string">"module"</span>); 
  <span class="hljs-keyword">const</span> noSources = options.devtool.includes(<span class="hljs-string">"nosources"</span>); 

  <span class="hljs-keyword">const</span> Plugin = evalWrapped 
    ? <span class="hljs-built_in">require</span>(<span class="hljs-string">"./EvalSourceMapDevToolPlugin"</span>) 
    : <span class="hljs-built_in">require</span>(<span class="hljs-string">"./SourceMapDevToolPlugin"</span>); 

  <span class="hljs-keyword">new</span> Plugin({ 
    <span class="hljs-attr">filename</span>: inline ? <span class="hljs-literal">null</span> : options.output.sourceMapFilename, 
    <span class="hljs-attr">moduleFilenameTemplate</span>: options.output.devtoolModuleFilenameTemplate, 
    <span class="hljs-attr">fallbackModuleFilenameTemplate</span>: 
      options.output.devtoolFallbackModuleFilenameTemplate, 
    <span class="hljs-attr">append</span>: hidden ? <span class="hljs-literal">false</span> : <span class="hljs-literal">undefined</span>, 
    <span class="hljs-attr">module</span>: moduleMaps ? <span class="hljs-literal">true</span> : cheap ? <span class="hljs-literal">false</span> : <span class="hljs-literal">true</span>, 
    <span class="hljs-attr">columns</span>: cheap ? <span class="hljs-literal">false</span> : <span class="hljs-literal">true</span>, 
    <span class="hljs-attr">noSources</span>: noSources, 
    <span class="hljs-attr">namespace</span>: options.output.devtoolNamespace 
  }).apply(compiler); 
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (options.devtool.includes(<span class="hljs-string">"eval"</span>)) { 
  <span class="hljs-keyword">const</span> EvalDevToolModulePlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">"./EvalDevToolModulePlugin"</span>); 
  <span class="hljs-keyword">new</span> EvalDevToolModulePlugin({ 
    <span class="hljs-attr">moduleFilenameTemplate</span>: options.output.devtoolModuleFilenameTemplate, 
    <span class="hljs-attr">namespace</span>: options.output.devtoolNamespace 
  }).apply(compiler); 
}
</code></pre>
<p data-nodeid="2457">如上面的代码所示， devtool 的值匹配并非精确匹配，某个关键字只要包含在赋值中即可获得匹配，例如：'foo-eval-bar' 等同于 'eval'，'cheapfoo-source-map' 等同于 'cheap-source-map'。</p>
<h4 data-nodeid="2458">Source Map 名称关键字</h4>
<p data-nodeid="2459">各字段的作用各不相同，为了便于记忆，我们在这里简单整理下这些关键字的作用：</p>
<ul data-nodeid="2460">
<li data-nodeid="2461">
<p data-nodeid="2462"><strong data-nodeid="2666">false</strong>：即不开启 source map 功能，其他不符合上述规则的赋值也等价于 false。</p>
</li>
<li data-nodeid="2463">
<p data-nodeid="2464"><strong data-nodeid="2671">eval</strong>：是指在编译器中使用 EvalDevToolModulePlugin 作为 source map 的处理插件。</p>
</li>
<li data-nodeid="2465">
<p data-nodeid="2466"><strong data-nodeid="2683">[xxx-...]source-map</strong>：根据 devtool 对应值中是否有 <strong data-nodeid="2684">eval</strong> 关键字来决定使用 EvalSourceMapDevToolPlugin 或 SourceMapDevToolPlugin 作为 source map 的处理插件，其余关键字则决定传入到插件的相关字段赋值。</p>
</li>
<li data-nodeid="2467">
<p data-nodeid="2468"><strong data-nodeid="2693">inline</strong>：决定是否传入插件的 filename 参数，作用是决定单独生成 source map 文件还是在行内显示，<strong data-nodeid="2694">该参数在 eval- 参数存在时无效</strong>。</p>
</li>
<li data-nodeid="2469">
<p data-nodeid="2470"><strong data-nodeid="2703">hidden</strong>：决定传入插件 append 的赋值，作用是判断是否添加 SourceMappingURL 的注释，<strong data-nodeid="2704">该参数在 eval- 参数存在时无效</strong>。</p>
</li>
<li data-nodeid="2471">
<p data-nodeid="2472"><strong data-nodeid="2709">module</strong>：为 true 时传入插件的 module 为 true ，作用是为加载器（Loaders）生成 source map。</p>
</li>
<li data-nodeid="2473">
<p data-nodeid="2474"><strong data-nodeid="2714">cheap</strong>：这个关键字有两处作用。首先，当 module 为 false 时，它决定插件 module 参数的最终取值，最终取值与 cheap 相反。其次，它决定插件 columns 参数的取值，作用是决定生成的 source map 中是否包含列信息，在不包含列信息的情况下，调试时只能定位到指定代码所在的行而定位不到所在的列。</p>
</li>
<li data-nodeid="2475">
<p data-nodeid="2476"><strong data-nodeid="2719">nosource</strong>：nosource 决定了插件中 noSource 变量的取值，作用是决定生成的 source map 中是否包含源代码信息，不包含源码情况下只能显示调用堆栈信息。</p>
</li>
</ul>
<h4 data-nodeid="2477">Source Map 处理插件</h4>
<p data-nodeid="6915">从上面的规则中我们还可以看到，根据不同规则，实际上 Webpack 是从三种插件中选择其一作为 source map 的处理插件。</p>


<ul data-nodeid="2479">
<li data-nodeid="2480">
<p data-nodeid="2481"><a href="https://github.com/webpack/webpack/blob/master/lib/EvalDevToolModulePlugin.js" data-nodeid="2724">EvalDevToolModulePlugin</a>：模块代码后添加 sourceURL=webpack:///+ 模块引用路径，不生成 source map 内容，模块产物代码通过 eval() 封装。</p>
</li>
<li data-nodeid="2482">
<p data-nodeid="2483"><a href="https://github.com/webpack/webpack/blob/master/lib/EvalSourceMapDevToolPlugin.js" data-nodeid="2728">EvalSourceMapDevToolPlugin</a>：生成 base64 格式的 source map 并附加在模块代码之后， source map 后添加 sourceURL=webpack:///+ 模块引用路径，不单独生成文件，模块产物代码通过 eval() 封装。</p>
</li>
<li data-nodeid="2484">
<p data-nodeid="2485"><a href="https://github.com/webpack/webpack/blob/master/lib/SourceMapDevToolPlugin.js" data-nodeid="2732">SourceMapDevToolPlugin</a>：生成单独的 .map 文件，模块产物代码不通过 eval 封装。</p>
</li>
</ul>
<p data-nodeid="8519">通过上面的代码分析，我们了解了不同参数在 Webpack 运行时起到的作用。那么这些不同参数组合下的各种预设对我们的 source map 生成又各自会产生什么样的效果呢？下面我们通过示例来看一下。</p>


<h3 data-nodeid="2487">不同预设的示例结果对比</h3>
<p data-nodeid="2488">下面，以课程示例代码 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/03_develop_environment" data-nodeid="2743">03_develop_environment</a> 为例，我们来对比下几种常用预设的差异（为了使时间差异更明显，示例中引入了几个大的类库文件）：</p>
<p data-nodeid="2489"><img alt="12.png" src="https://s0.lgstatic.com/i/image/M00/43/5E/Ciqc1F87kyiAZvHdAAIGvohk2F4144.png" data-nodeid="2747"></p>
<blockquote data-nodeid="2490">
<p data-nodeid="2491">*注1：“/”前后分别表示产物 js 大小和对应 .map 大小。<br>
*注2：“/”前后分别表示初次构建时间和开启 watch 模式下 rebuild 时间。对应统计的都是 development 模式下的笔者机器环境下几次构建时间的平均值，只作为相对快慢与量级的比较。</p>
</blockquote>
<h4 data-nodeid="2492">不同预设的效果总结</h4>
<p data-nodeid="2493">从上图的数据中我们不难发现，选择不同的 devtool 类型在以下几个方面会产生不同的效果。</p>
<ul data-nodeid="2494">
<li data-nodeid="2495">
<p data-nodeid="2496">质量：生成的 source map 的质量分为 5 个级别，对应的调试便捷性依次降低：源代码 &gt; 缺少列信息的源代码 &gt; loader 转换后的代码 &gt; 生成后的产物代码 &gt; 无法显示代码（具体参见下面的<strong data-nodeid="2760">不同质量的源码示例</strong>小节）。对应对质量产生影响的预设关键字优先级为 souce-map = eval-source-map &gt; cheap-module- &gt; cheap- &gt; eval = none &gt; nosource-。</p>
</li>
<li data-nodeid="2497">
<p data-nodeid="2498">构建的速度：再次构建速度都要显著快于初次构建速度。不同环境下关注的速度也不同：</p>
<ul data-nodeid="2499">
<li data-nodeid="2500">
<p data-nodeid="2501">在开发环境下：一直开着 devServer，再次构建的速度对我们的效率影响远大于初次构建的速度。从结果中可以看到，eval- 对应的 EvalSourceMapDevToolPlugin 整体要快于不带 eval- 的 SourceMapDevToolPlugin。尤其在质量最佳的配置下，eval-source-map 的再次构建速度要远快于其他几种。而同样插件配置下，不同质量配置与构建速度成反比，但差异程度有限，更多是看具体项目的大小而定。</p>
</li>
<li data-nodeid="2502">
<p data-nodeid="2503">在生产环境下：通常不会开启再次构建，因此相比再次构建，初次构建的速度更值得关注，甚至对构建速度以外因素的考虑要优先于对构建速度的考虑，这一部分我们在之后的构建优化的课程里会再次讨论到。</p>
</li>
</ul>
</li>
<li data-nodeid="2504">
<p data-nodeid="2505">包的大小和生成方式：在开发环境下我们并不需要关注这些因素，正如在开发环境下也通常不考虑使用分包等优化方式。我们需要关注速度和质量来保证我们的高效开发体验，而其他的部分则是在生产环境下需要考虑的问题。</p>
</li>
</ul>
<h4 data-nodeid="2506">不同质量的源码示例</h4>
<ul data-nodeid="2507">
<li data-nodeid="2508">
<p data-nodeid="2509">源码且包含列信息</p>
</li>
</ul>
<p data-nodeid="2510"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/42/9E/CgqCHl85_KuANSVfAADSE8VO7Qg572.png" data-nodeid="2769"></p>
<ul data-nodeid="2511">
<li data-nodeid="2512">
<p data-nodeid="2513">源码不包含列信息</p>
</li>
</ul>
<p data-nodeid="2514"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/42/93/Ciqc1F85_LCAMTlgAADhqpZ4v9o628.png" data-nodeid="2773"></p>
<ul data-nodeid="2515">
<li data-nodeid="2516">
<p data-nodeid="2517">Loader转换后代码</p>
</li>
</ul>
<p data-nodeid="2518"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/42/9E/CgqCHl85_LqAPrYzAADfmUwS_JE006.png" data-nodeid="2777"></p>
<ul data-nodeid="2519">
<li data-nodeid="2520">
<p data-nodeid="2521">生成后的产物代码</p>
</li>
</ul>
<p data-nodeid="2522"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/42/9E/CgqCHl85_MGAHhmMAAKGwvDeXIM418.png" data-nodeid="2781"></p>
<h4 data-nodeid="2523">开发环境下 Source Map 推荐预设</h4>
<p data-nodeid="2524">在这里我们对开发环境下使用的推荐预设做一个总结（生产环境的预设我们将在之后的构建效率篇中再具体分析）：</p>
<ul data-nodeid="2525">
<li data-nodeid="2526">
<p data-nodeid="2527">通常来说，开发环境首选哪一种预设取决于 source map 对于我们的帮助程度。</p>
</li>
<li data-nodeid="2528">
<p data-nodeid="2529">如果对项目代码了如指掌，查看产物代码也可以无障碍地了解对应源代码的部分，那就可以关闭 devtool 或使用 eval 来获得最快构建速度。</p>
</li>
<li data-nodeid="2530">
<p data-nodeid="2531">反之如果在调试时，需要通过 source map 来快速定位到源代码，则优先考虑使用 <strong data-nodeid="2791">eval-cheap-modulesource-map</strong>，它的质量与初次/再次构建速度都属于次优级，以牺牲定位到列的功能为代价换取更快的构建速度通常也是值得的。</p>
</li>
<li data-nodeid="2532">
<p data-nodeid="2533">在其他情况下，根据对质量要求更高或是对速度要求更高的不同情况，可以分别考虑使用 <strong data-nodeid="2801">eval-source-map</strong> 或 <strong data-nodeid="2802">eval-cheap-source-map</strong>。</p>
</li>
</ul>
<p data-nodeid="2534">了解了开发环境下如何选择 source map 预设后，我们再来补充几种工具和脚手架中的默认预设：</p>
<ul data-nodeid="10137">
<li data-nodeid="10138">
<p data-nodeid="10139">Webpack 配置中，如果不设定 devtool，则使用默认值 eval，即速度与 devtool:false 几乎相同、但模块代码后多了 sourceURL 以帮助定位模块的文件名称。</p>
</li>
<li data-nodeid="10140">
<p data-nodeid="10141"><a href="https://github.com/facebook/create-react-app/blob/fa648daca1dedd97aec4fa3bae8752c4dcf37e6f/packages/react-scripts/config/webpack.config.js" data-nodeid="10147">create-react-app 中</a>，在生产环境下，根据 shouldUseSourceMap 参数决定使用‘source-map’或 false；在开发环境下使用‘cheap-module-source-map’（不包含列信息的源代码，但更快）。</p>
</li>
<li data-nodeid="10142">
<p data-nodeid="10143"><a href="https://github.com/vuejs/vue-cli/blob/36f961e43dc76705878659247b563e2af83138ce/packages/%40vue/cli-service/lib/commands/serve.js" data-nodeid="10151">vue-cli-service 中</a>，与 creat-react-app 中相同。</p>
</li>
</ul>


<p data-nodeid="11755">除了上面讨论的这些简单的预设外，Webpack 还允许开发者直接使用对应插件来进行更精细化的 source map 控制，在开发环境下我们首选的还是 EvalSourceMapDevToolPlugin。下面我们再来看看如何直接使用这个插件进行优化。</p>


<h4 data-nodeid="2543">EvalSourceMapDevToolPlugin 的使用</h4>
<p data-nodeid="2544">在 EvalSourceMapDevToolPlugin 的 <a href="https://webpack.js.org/plugins/eval-source-map-dev-tool-plugin/" data-nodeid="2818">传入参数</a>中，除了上面和预设相关的 filename、append、module、columns 外，还有影响注释内容的 moduleFilenameTemplate 和 protocol，以及影响处理范围的 test、include、exclude。这里重点看处理范围的参数，因为通常我们需要调试的是开发的业务代码部分，而非依赖的第三方模块部分。因此在生成 source map 的时候如果可以排除第三方模块的部分而只生成业务代码的 source map，无疑能进一步提升构建的速度，例如示例：</p>
<pre class="lang-javascript" data-nodeid="2545"><code data-language="javascript">webpack.config.js 
  ... 
  <span class="hljs-comment">//devtool: 'eval-source-map', </span>
  <span class="hljs-attr">devtool</span>: <span class="hljs-literal">false</span>, 
  <span class="hljs-attr">plugins</span>: [ 
    <span class="hljs-keyword">new</span> webpack.EvalSourceMapDevToolPlugin({ 
      <span class="hljs-attr">exclude</span>: <span class="hljs-regexp">/node_modules/</span>, 
      <span class="hljs-built_in">module</span>: <span class="hljs-literal">true</span>, 
      <span class="hljs-attr">columns</span>: <span class="hljs-literal">false</span> 
    }) 
  ], 
  ...
</code></pre>
<p data-nodeid="2546">在上面的示例中，我们将 devtool 设为 false，而直接使用 EvalSourceMapDevToolPlugin，通过传入 module: true 和 column:false，达到和预设 eval-cheap-module-source-map 一样的质量，同时传入 exclude 参数，排除第三方依赖包的 source map 生成。保存设定后通过运行可以看到，在文件体积减小（尽管开发环境并不关注文件大小）的同时，再次构建的速度相比上面表格中的速度提升了将近一倍，达到了最快一级。</p>
<p data-nodeid="2547"><img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image/M00/42/9E/CgqCHl85_N2AUkcpAAEqvMKhgVQ549.png" data-nodeid="2823"></p>
<p data-nodeid="2548">类似这样的优化可以帮助我们在一些大型项目中，通过自定义设置来获取比预设更好的开发体验。</p>
<h3 data-nodeid="2549">总结</h3>
<p data-nodeid="13359">在今天这一课时中，我们主要了解了提升开发效率的另一个重要工具——source map 的用途和使用方法。我们分析了 Webpack 中 devtool 的各种参数预设的组合规则、使用效果及其背后的原理。对于开发环境，我们根据一组示例对比分析来了解通常情况下的最佳选择，也知道了如何直接使用插件来达到更细致的优化。</p>


<p data-nodeid="2551">限于篇幅原因，关于 source map 这一课时还有两个与提效无关的小细节没有提到，一个是生成的 source map 的内容，即浏览器工具是如何将 source map 内容映射回源文件的，如果你感兴趣可以通过这个<a href="http://www.ruanyifeng.com/blog/2013/01/javascript_source_map.html" data-nodeid="2830">链接</a>进一步了解；另一个是我们在控制台的网络面板中通常看不到 source map 文件的请求，其原因是出于安全考虑 Chrome 隐藏了 source map 的请求，需要通过 <a href="chrome://net-export/" data-nodeid="2834">net-log</a> 来查询。</p>
<p data-nodeid="2552"><strong data-nodeid="2840">最后还是留一个小作业</strong>：不知道你有没有留意过自己项目里的 source map 使用的是哪一种生成方式吗？可以根据这一课时的内容对它进行调整和观察效果，也欢迎你在课后留言区讨论项目里对 source map 的优化方案。</p>

---

### 精选评论

##### **贵：
> sourceMap确实是个好东西😆

##### **斌：
> 不同质量的源码示例第一个例子 源码包含列信息和不包含列信息的区别 没看懂，没看出来有啥区别？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是这样，包含列信息的报错代码，在控制台Console面板中可以显示出错的位置在第几行第几列，然后用户点击跳转到Sources面板中后光标也会直接停留到对应行列的位置上。而不在列信息的代码，则只显示行信息，并且点击跳转后也只能定位到行首。如果仔细看的话，示例代码的第一张图中new前面有一小竖线，就是光标：）

##### **8621：
> 除了学习到了sourcemap的配置方式，更重要的是学习到了讲师的思考问题和学习的方式，受益匪浅。

##### **宗：
> https://github.com/webpack/webpack/tree/master/examples/source-maphttps://www.webpackjs.com/configuration/devtool/ 官网也有给比较的diff

