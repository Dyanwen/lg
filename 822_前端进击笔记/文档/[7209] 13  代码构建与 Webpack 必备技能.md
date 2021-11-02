<p data-nodeid="1169" class="">最初的页面开发中，前端实现一个页面只需要在一个文件里完成，包括 HTML/CSS/JavaScript 各种内容。后来，通常将常用的静态资源放置在 CDN，并使用<code data-backticks="1" data-nodeid="1299">&lt;link&gt;</code>和<code data-backticks="1" data-nodeid="1301">&lt;script&gt;</code>的 src 属性引入的方式，来减少页面开发过程中的重复代码编写。</p>
<p data-nodeid="1170">如今前端页面的功能越来越复杂，规模也越来越大。为了提升代码的可读性、项目的可维护性，我们会将一些通用的工具和组件进行抽象，代码被有组织地按照一定规则进行划分，比如按照功能划分为页面、组件、工具库、脚本等。</p>
<p data-nodeid="1171">这个过程便是模块化，而 JavaScript 中的模块规范不止一种。</p>
<h3 data-nodeid="1172">JavaScript 模块</h3>
<p data-nodeid="1173">在 JavaScript 中，我们常说的模块规范包括 CommonJS/AMD/UMD/ES6 Module 四种。这些模块规范和定义之间的区别常常容易搞混，我们先来分别看一下。</p>
<h4 data-nodeid="1174">CommonJS 规范</h4>
<p data-nodeid="1175">CommonJS 规范定义了模块应该怎样进行编写，从而各个模块系统之间可以进行相互操作。</p>
<p data-nodeid="1176">我们来看一个 CommonJS 规范的模块示例：</p>
<pre class="lang-java" data-nodeid="1177"><code data-language="java"><span class="hljs-keyword">var</span> beta = require(<span class="hljs-string">'beta'</span>);
function verb {
  <span class="hljs-keyword">return</span> beta.verb();
}
<span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = {
  verb: verb
};
</code></pre>
<p data-nodeid="1178">在该示例中，使用<code data-backticks="1" data-nodeid="1311">require()</code>载入模块，使用<code data-backticks="1" data-nodeid="1313">module.exports</code>输出模块 。<br>
一般来说，CommonJS 有以下特点：</p>
<ul data-nodeid="1179">
<li data-nodeid="1180">
<p data-nodeid="1181">一个文件就是一个模块；</p>
</li>
<li data-nodeid="1182">
<p data-nodeid="1183">使用<code data-backticks="1" data-nodeid="1319">require()</code>载入模块，使用<code data-backticks="1" data-nodeid="1321">module.exports</code>输出模块，因此各个模块间可以进行交互；</p>
</li>
<li data-nodeid="1184">
<p data-nodeid="1185">不支持异步加载。</p>
</li>
</ul>
<p data-nodeid="1186">或许你已经知道，Node.js 环境使用的便是基于 CommonJS 规范实现的模块系统，而如今我们提到 CommonJS 规范，也基本上认为是 Node.js 系统。</p>
<p data-nodeid="1187">为什么浏览器环境不使用 CommonJS 规范呢？这是因为 CommonJS 不支持异步加载，而前面我们也说过，浏览器环境中同步任务的执行会带来性能问题，但对于异步模块定义（AMD）来说就不存在这样的问题。</p>
<h4 data-nodeid="1188">AMD</h4>
<p data-nodeid="1189">顾名思义，异步模块定义（AMD）主要为了解决异步加载模块而提出，它通过指定模块和依赖项的方式来定义模块。</p>
<p data-nodeid="1190">RequireJS 便是基于 AMD 的实现，我们同样可以看一个模块示例：</p>
<pre class="lang-java" data-nodeid="1191"><code data-language="java">define(<span class="hljs-string">"alpha"</span>, [<span class="hljs-string">"require"</span>, <span class="hljs-string">"exports"</span>, <span class="hljs-string">"beta"</span>], function (
  require,
  <span class="hljs-keyword">exports</span>,
  beta
) {
  <span class="hljs-keyword">exports</span>.verb = function () {
    <span class="hljs-keyword">return</span> beta.verb();
    <span class="hljs-comment">// 或者可以这么写</span>
    <span class="hljs-keyword">return</span> require(<span class="hljs-string">"beta"</span>).verb();
  };
});
</code></pre>
<p data-nodeid="1192">在该示例中，导出 ID 为 alpha 的模块，依赖了 ID 为 beta 的模块。</p>
<p data-nodeid="1193">现在我们知道，Node.js 环境中的模块系统基于 CommonJS 规范，而浏览器环境中需要使用 AMD 实现。</p>
<p data-nodeid="1194">那么如果我们有一个模块，需要同时能运行在 Node.js 环境和浏览器环境中，要怎么办？我们可以使用 UMD 模式。</p>
<h4 data-nodeid="1195">UMD</h4>
<p data-nodeid="1196">为了兼容 AMD 和 CommonJS 的规范，通用模块定义（UMD）模式被提出，它在兼容两者的同时，也支持了传统的全局变量模式。</p>
<p data-nodeid="1197">我们来看一个 UMD 模式的模块示例：</p>
<pre class="lang-java" data-nodeid="1198"><code data-language="java">(function (root, factory) {
  <span class="hljs-keyword">if</span> (typeof define === <span class="hljs-string">"function"</span> &amp;&amp; define.amd) {
    <span class="hljs-comment">// AMD</span>
    define([<span class="hljs-string">"jquery"</span>], factory);
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (typeof <span class="hljs-keyword">exports</span> === <span class="hljs-string">"object"</span>) {
    <span class="hljs-comment">// CommonJS</span>
    <span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = factory(require(<span class="hljs-string">"jquery"</span>));
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 全局变量</span>
    root.returnExports = factory(root.jQuery);
  }
})(<span class="hljs-keyword">this</span>, function ($) {
  <span class="hljs-comment">// ...</span>
});
</code></pre>
<p data-nodeid="1199">可以看到，UMD 模块头部通常都会有用来判断模块加载器环境的代码，并根据不同的环境提供了不同的方式进行加载。</p>
<p data-nodeid="1200">到这里，似乎不管是 Node.js 环境还是浏览器环境，都有支持的模块规范，也有能相互兼容的模块规范了。那么，ES6 模块又是什么呢？</p>
<h4 data-nodeid="1201">ES6 模块</h4>
<p data-nodeid="1202">相比于运行时进行加载的 CommonJS 规范，ES6 模块化主要是为了<strong data-nodeid="1343">在编译阶段就可以确定各个模块之间的依赖关系</strong>。</p>
<p data-nodeid="1203">我们同样来看一个 ES6 模块的示例代码：</p>
<pre class="lang-java" data-nodeid="1204"><code data-language="java"><span class="hljs-comment">// import 导入</span>
<span class="hljs-keyword">import</span> BaseTask, { TaskType } from <span class="hljs-string">"./BaseTask"</span>;
<span class="hljs-comment">// export 导出</span>
export { BaseTask };
</code></pre>
<p data-nodeid="1205">在该示例中，使用<code data-backticks="1" data-nodeid="1346">import</code>加载模块，使用<code data-backticks="1" data-nodeid="1348">export</code>输出模块。</p>
<p data-nodeid="1206">ES6 模块的特点如下：</p>
<ul data-nodeid="1207">
<li data-nodeid="1208">
<p data-nodeid="1209">使用<code data-backticks="1" data-nodeid="1352">import</code>加载和<code data-backticks="1" data-nodeid="1354">export</code>输出；</p>
</li>
<li data-nodeid="1210">
<p data-nodeid="1211">一个模块只会加载一次（CommonJS 也是一样）；</p>
</li>
<li data-nodeid="1212">
<p data-nodeid="1213">导出的模块为变量引用，因此可以在内存中共享。</p>
</li>
</ul>
<p data-nodeid="1214">现在大多数前端项目中都使用 ES6 模块，由于 ES6 模块化目的是编译阶段确定模块间依赖关系，因此我们需要在编译的时候使用 Babel、Webpack 等方式构建依赖关系树。</p>
<p data-nodeid="1215">除此之外，ES6 模块化在各个浏览器里的兼容性差异较大，因此同样需要进行 Babal 编译以及 Webpack 进行打包，这个过程我们称之为代码构建。</p>
<p data-nodeid="1216">我们来总结一下 CommonJS/AMD/UMD/ES6 Module 这四种模块规范：</p>
<ol data-nodeid="1217">
<li data-nodeid="1218">
<p data-nodeid="1219">CommonJS 规范定义了模块应该怎样进行编写，从而各个模块系统之间可以进行相互操作。</p>
</li>
<li data-nodeid="1220">
<p data-nodeid="1221">CommonJS 不支持异步加载，因此异步模块定义（AMD）主要为了解决异步加载模块而提出。</p>
</li>
<li data-nodeid="1222">
<p data-nodeid="1223">通用模块定义（UMD）模式用于兼容 AMD 和 CommonJS 的规范。</p>
</li>
<li data-nodeid="1224">
<p data-nodeid="1225">CommonJS 规范用于运行时进行模块加载，ES6 模块化可以在编译阶段确定各个模块之间的依赖关系。</p>
</li>
</ol>
<p data-nodeid="1226">下面，我们一起来看看 Webpack 这个在前端项目中经常出现的工具。</p>
<h3 data-nodeid="1227">Webpack 工具都做了些什么</h3>
<p data-nodeid="1228">如今前端项目大多数都使用了模块化，而如果想要将多个文件的代码打包成最终可按照预期运行的代码，则需要使用到代码构建工具。</p>
<p data-nodeid="1229">不管项目代码是如何进行组织的，项目中又有多少个文件，最终浏览器依然会从 HTML 内容进行解析和加载，因此我们需要对项目中的代码进行构建（包括编译和打包），生成浏览器可正常解析和加载的内容。</p>
<p data-nodeid="1230">我们先来认识下常见的前端构建相关的工具。</p>
<h4 data-nodeid="1231" class="">常见的前端构建工具</h4>
<p data-nodeid="4965">对于前端开发来说，我们会用到各式各样的构建/打包工具，比如这些。</p>
<p data-nodeid="4966" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/3F/7C/Cgp9HWCeSsSAHiJFAALlqT5qxqA460.png" alt="图片2.png" data-nodeid="4970"></p>


<p data-nodeid="1234">其中，涉及模块化代码打包的主要有 Grunt/Gulp/Webpack/Rollup。很多同学会搞混这几个工具，这里我简单介绍下它们之间的区别。</p>
<ol data-nodeid="1235">
<li data-nodeid="1236">
<p data-nodeid="1237">Gulp/Grunt 是一种能够优化前端工作流程的工具，比如自动刷新页面、combo、压缩 CSS/JavaScript、编译 Less/Sass 等。</p>
</li>
<li data-nodeid="1238">
<p data-nodeid="1239">Webpack/Rollup 是一个 JavaScript 的模块打包器，用于整合编译成最终的代码。</p>
</li>
<li data-nodeid="1240">
<p data-nodeid="1241">其中，Rollup 通常用来构建库，Webpack 更适合用来构建应用程序。</p>
</li>
</ol>
<p data-nodeid="1242">对于业务团队来说，进行代码的模块打包更多情况下会选择 Webpack。那么，下面我们主要围绕 Webpack 工具，来介绍代码模块化打包的过程。</p>
<h3 data-nodeid="1243">认识 Webpack</h3>
<p data-nodeid="1244">相信你肯定也认识 Webpack，要了解一个工具，最好的方式就是从如何使用它开始熟悉。</p>
<p data-nodeid="1245">Webpack 的使用中有 4 个核心概念：入口（entry）、输出（output）、Loader、插件（plugins），我们先来分别看看。</p>
<h4 data-nodeid="1246">入口（entry）</h4>
<p data-nodeid="1247">首先便是入口（entry），entry 指向我们前端应用的第一个启动文件。例如，在 Vue 中是<code data-backticks="1" data-nodeid="1385">new Vue()</code>位置所在的文件，在 Angular 中是启动<code data-backticks="1" data-nodeid="1387">.bootstrap()</code>的文件，在 React 中则是<code data-backticks="1" data-nodeid="1389">ReactDOM.render()</code>或者是<code data-backticks="1" data-nodeid="1391">React.render()</code>的启动文件。</p>
<pre class="lang-java" data-nodeid="1248"><code data-language="java"><span class="hljs-comment">// 将entry指向启动文件即可</span>
<span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = {
  entry: <span class="hljs-string">"./path/to/my/entry/file.js"</span>,
};
</code></pre>
<p data-nodeid="1249">或许你会疑惑，入口的一个文件，又是怎样把整个前端项目中的代码关联起来，并进行打包的呢？</p>
<p data-nodeid="3797">实际上， Webpack 会从 entry 开始，通过解析模块间的依赖关系，递归地构建出一个依赖图。我们如果在项目中使用<code data-backticks="1" data-nodeid="3800">webpack-bundle-analyzer</code>插件，也可以看到生成的这样一个依赖图。</p>
<p data-nodeid="3798" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3F/85/CioPOWCeSrqACnYnABLbVfl-Zds569.png" alt="图片4.png" data-nodeid="3804"></p>


<p data-nodeid="1252">Webpack 会根据依赖图来对各个模块进行整合，最终打包成一个或多个的文件，来提供给浏览器进行加载。</p>
<p data-nodeid="1253" class="">既然有入口，那当然就有出口，Webpack 中的出口由输出（output）字段来描述。</p>
<h4 data-nodeid="1254">输出（output）</h4>
<p data-nodeid="1255">输出（output）字段用于告诉 Webpack 要将打包后的代码生成的文件名是什么（<code data-backticks="1" data-nodeid="1404">filename</code>），以及将它们放在哪里（<code data-backticks="1" data-nodeid="1406">path</code>）。</p>
<pre class="lang-java" data-nodeid="1256"><code data-language="java"><span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = {
  output: {
    filename: <span class="hljs-string">"bundle.js"</span>, <span class="hljs-comment">// 编译文件的文件名，比如 main.js/bundle.js/index.js</span>
    path: <span class="hljs-string">"/home/proj/public/assets"</span>, <span class="hljs-comment">// 对应一个绝对路径，此路径是你希望一次性打包的目录</span>
  },
};
</code></pre>
<p data-nodeid="1257">有了 entry 和 output，我们来看看 Webpack 中间的编译过程中，是怎样用到 Loader 和 Plugins 的。</p>
<h4 data-nodeid="1258">Loader</h4>
<p data-nodeid="1259">要了解 Loader，你需要知道在 Webpack 中，每个文件(<code data-backticks="1" data-nodeid="1411">.css</code>,<code data-backticks="1" data-nodeid="1413">.html</code>,<code data-backticks="1" data-nodeid="1415">.scss</code>,<code data-backticks="1" data-nodeid="1417">.jpg</code>等) 都会被作为模块处理。如果你看过生成的 bundle.js 代码就会发现，Webpack 将所有的模块打包一起，每个模块添加标记 id，通过这样一个 id 去获取所需模块的代码。</p>
<p data-nodeid="1260">但实际上，Webpack 只理解 JavaScript，因此 Loader 的作用就是把不同的模块和文件（比如 HTML、CSS、JSX、Typescript 等）转换为 JavaScript 模块。</p>
<p data-nodeid="2039">而不同的应用场景需要不同的 Loader，比如我们经常会使用到的 CSS 相关 Loader 和其他资源 Loader。</p>
<p data-nodeid="2631"><img src="https://s0.lgstatic.com/i/image6/M01/3F/7C/Cgp9HWCeSqyAWHwvAAHProznCQc246.png" alt="图片3.png" data-nodeid="2635"></p>
<p data-nodeid="2632">前面我们说到，ES6 模块需要依赖 Babel 编译和 Webpack 打包，而 Babel 在 Webpack 中就是使用 Loader 的方式来进行编译的。</p>





<p data-nodeid="1264">babel-loader 将 ES6/ES7 语法编译生成 ES5，其中部分特性还需要 babel-polyfill 支持。这是因为 Babel 默认只转换新的 JavaScript 语法（比如<code data-backticks="1" data-nodeid="1426">const/let</code>），但不会对新的 API（比如<code data-backticks="1" data-nodeid="1428">Promise</code>）进行处理。</p>
<p data-nodeid="1265">Webpack 在编译过程中，支持多个 Loader 通过流水线的方式进行先后编译，编译的顺序为从后往前，最终以 JavaScript 模块的方式输出。</p>
<p data-nodeid="1266">到这里，我们知道 Webpack 以 entry 为入口，链式调用各个 Loader 进行编译生成 JavaScript，最终打包放置在 output 中。其中 Loader 只负责将其他非 JavaScript 模块转换成 JavaScript 模块。</p>
<p data-nodeid="1267">那 Webpack 又是怎样地对这些代码进行组织并生成文件呢？这就是插件 Plugins 负责的事情。</p>
<h4 data-nodeid="1268">插件（plugins）</h4>
<p data-nodeid="1269">插件（plugins）主要负责解决 Loader 无法做到的事情，它可以访问在 Webpack 编译过程中的关键事件，对 Webpack 内部示例的一些数据进行处理，处理完成后回调 Webpack 让其继续。</p>
<p data-nodeid="1270">这样说或许有些抽象，我们直接来看看几个常用的插件就明白了。</p>
<ul data-nodeid="1271">
<li data-nodeid="1272">
<p data-nodeid="1273">HtmlwebpackPlugin：可以生成创建 HTML 入口文件，也可以为 HTML 文件中引入的外部资源如 script、link 动态添加每次编译后的哈希值，防止引用缓存的外部文件问题。</p>
</li>
<li data-nodeid="1274">
<p data-nodeid="1275">CommonsChunkPlugin：用来提取代码中的公共模块，并将这些公共模块按照预期进行打包生成独立的文件。</p>
</li>
<li data-nodeid="1276">
<p data-nodeid="1277">ProvidePlugin：用来定义标识符，当遇到指定标识符的时候自动加载模块，适合引入的全局变量（比如 jQuery）。</p>
</li>
<li data-nodeid="1278">
<p data-nodeid="1279">ExtractTextPlugin：可以将样式从 JavaScript 中抽出，生成单独的 .css 样式文件。</p>
</li>
</ul>
<p data-nodeid="1280">看到这里你应该已经明白了，<strong data-nodeid="1445">插件可以用来控制最终生成的代码是如何进行组织和输出的，包括对代码的打包优化、压缩，甚至是启用模块热替换、重新定义环境中的变量，等等</strong>。</p>
<p data-nodeid="1281">那么，现在我们已经知道 Webpack 到底对项目代码做了什么。</p>
<ol data-nodeid="1282">
<li data-nodeid="1283">
<p data-nodeid="1284">通过 entry 指定的入口开始，解析各个文件模块间的依赖。</p>
</li>
<li data-nodeid="1285">
<p data-nodeid="1286">根据模块间的依赖关系，开始对各个模块进行编译。</p>
</li>
<li data-nodeid="1287">
<p data-nodeid="1288">编译过程中，根据配置的规则对一些模块使用 Loader 进行编译处理。</p>
</li>
<li data-nodeid="1289">
<p data-nodeid="1290">根据插件的配置，对 Loader 编译后的代码进行封装、优化、分块、压缩等。</p>
</li>
<li data-nodeid="1291">
<p data-nodeid="1292">最终 Webpack 整合各个模块，根据依赖关系将它们打包成最终的一个或者多个文件。</p>
</li>
</ol>
<p data-nodeid="1293">这便是 Webpack 做的事情：<strong data-nodeid="1456">让前端项目中模块化的代码能最终在浏览器中进行加载、并正常地工作。</strong></p>
<h3 data-nodeid="1294">小结</h3>
<p data-nodeid="1295">如今几乎大多数框架的代码构建工具（比如 Vue CLI、Create React App 等）底层实现都依赖 Webpack。虽然这些前端框架都提供了完善的脚手架，也提供了丰富的配置功能，但如果想要对自己的项目进行更多优化，我们依然需要自己调整 Webpack 配置，因此对它的掌握也是不可少的。</p>
<p data-nodeid="1296">对于前端来说，自动化工具的出现，大大降低了应用的开发和维护成本，也因此前端生态也日益丰富和完善。善用这些工具来解决开发过程中的痛点，是作为现代前端开发的必备技能。比如，我们可以使用 Webpack 的 Loader 和插件，实现自己的 AST 语法分析和代码处理过程，这也是许多前端框架在做的事情。</p>
<p data-nodeid="1297" class="">如果你要做一个在编译时自动给 Class 类加上指定装饰器的能力，你认为是应该使用 Loader 还是 Plugins 呢？可以在留言区留下你的想法和实现逻辑。</p>

---

### 精选评论

##### *庆：
> 在编译时自动给 Class 类加上指定装饰器的能力: 由于这需要在编译时进行处理，所以应该是使用Loader，因为Loader就是根据特定的规则对模块进行编译处理，而Plugins是对Loader编译后的代码进行处理

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯，可以使用 babel loader 提供的能力实现

##### console_man：
> 涉及到代码修改都是loader，plugin并不负责代码修改

##### *振：
> 我猜应该是插件吧，一旦 webpack 即将处理 js 文件，就加上装饰器，然后返回给 webpack 继续执行。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Loader 可以做到哦，babel loader 就提供了这样的能力

##### *雨：
> 用Plugins，类似HtmlwebpackPlugin，生成文档DOM对象，遍历每个node的class

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以通过 loader 处理的 AST 对象中加上装饰器

##### *军：
> 简单点就是只要涉及改变输出结果的那就是plugin，loader只是辅助webpack进行解析

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实际上，loader 在解析过程中可以拿到 AST，因此也可以对 AST 进行修改，比如 babel 相关的 loader 便会在适当的时候在代码中添加 polyfill。

##### *聪：
> Loader 的作用就是把不同的模块和文件（比如 HTML、CSS、JSX、Typescript 等）转换为 JavaScript 模块。其他的功能应该都是使用Plugins吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Loader 在转换过程中也可以进行自定义的处理哦

