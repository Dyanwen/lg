<p data-nodeid="25275">在前面几讲，我们学习了如何从零开始新建 TypeScript Node.js、Web 项目。</p>


<p data-nodeid="24479">然而，TypeScript 作为 JavaScript 的超集，花费了漫长的时间才替代了 JavaScript，并成长为构建应用的主流技术。比如我自己所负责的项目，最初也主要是基于 ES6（JavaScript） 源码构建的，甚至极个别项目是基于无须转译的 ES5（JavaScript） 代码构建的。</p>
<p data-nodeid="24480">从 JavaScript 进化到 TypeScript，也就意味着需要大量的迁移、重构操作。因此，接下来我们将学习将 JavaScript 技术栈项目迁移到 TypeScript 的操作步骤和实用技巧。</p>
<h3 data-nodeid="24481">迁移步骤</h3>
<h4 data-nodeid="24482">调整项目结构</h4>
<p data-nodeid="24483">首先，我们可以参照 18 讲和 19 讲的内容调整项目结构，比如使用 src 目录组织源码，typings 目录组织类型声明定义，lib 目录作为 Node.js 模块的构建产物，build 目录作为 Web 项目的构建产物。</p>
<p data-nodeid="24484">然后，我们需要在项目根目录下创建一个 tsconfig.json，让源码和单测共享一个配置文件。</p>
<p data-nodeid="24485">因为如今大多数的 JavaScript 项目都是基于 ES6+ 组织源码，再转译为 JavaScript，其项目结构基本可以划分为如下所示，所以我们只需要创建一个 tsconfig.json 即可。</p>
<pre class="lang-java" data-nodeid="24486"><code data-language="java">JavaScript2TypeScriptProject
├── src
│   ├── a.js
│   └── b.js
├── build 或则 lib
├── typings
├── <span class="hljs-keyword">package</span>.json
└── tsconfig.json
</code></pre>
<p data-nodeid="24487">接下来就是如何配置 tsconfig.json 了，下面我们一起探讨一下。</p>
<h4 data-nodeid="24488">配置 tsconfig</h4>
<p data-nodeid="24489">在正式讲解之前，我们先插播一道思考题：Node.js 项目需要如何配置？</p>
<blockquote data-nodeid="24490">
<p data-nodeid="24491">提示信息：区别仅在于 Node.js 项目需要指定 rootDir、outDir。</p>
</blockquote>
<p data-nodeid="24492">以配置 React Web 项目为例，为了尽可能少改动源码、让项目正常运行起来，我们不要一步到位开启严格模式，而应该尽量宽松地配置 tsconfig，如下配置所示。</p>
<pre class="lang-java" data-nodeid="24493"><code data-language="java">{
  <span class="hljs-string">"compilerOptions"</span>: {
    <span class="hljs-string">"target"</span>: <span class="hljs-string">"es5"</span>,
    <span class="hljs-string">"lib"</span>: [
      <span class="hljs-string">"dom"</span>,
      <span class="hljs-string">"dom.iterable"</span>,
      <span class="hljs-string">"esnext"</span>
    ],
    <span class="hljs-string">"allowJs"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"skipLibCheck"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"esModuleInterop"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"allowSyntheticDefaultImports"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"forceConsistentCasingInFileNames"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"module"</span>: <span class="hljs-string">"esnext"</span>,
    <span class="hljs-string">"moduleResolution"</span>: <span class="hljs-string">"node"</span>,
    <span class="hljs-string">"resolveJsonModule"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"isolatedModules"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"noEmit"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"jsx"</span>: <span class="hljs-string">"react"</span>,
    <span class="hljs-string">"typeRoots"</span>: [<span class="hljs-string">"node_modules/@types"</span>, <span class="hljs-string">"./typings"</span>]
  },
  <span class="hljs-string">"include"</span>: [ <span class="hljs-string">"src"</span>, <span class="hljs-string">"typings"</span> ]
}
</code></pre>
<p data-nodeid="24494">其中，比较重要的配置项分为如下 5 个。</p>
<ul data-nodeid="29679">
<li data-nodeid="29680">
<p data-nodeid="29681">第 3 行配置“target”为 "es5"，用来将 TypeScript 转译为低版本、各端兼容性较好的 ES5 代码。</p>
</li>
<li data-nodeid="29682">
<p data-nodeid="29683">第 9 行开启的 allowJs，它允许 JavaScript 和 TypeScript 混用，这使得我们可以分批次、逐模块地迁移代码。</p>
</li>
<li data-nodeid="29684">
<p data-nodeid="29685">第 20 行我们把 typings 目录添加到类型查找路径，让 TypeScript 可以查找到自定义类型声明，比如为缺少类型声明的第三方模块补齐类型声明。</p>
</li>
<li data-nodeid="29686">
<p data-nodeid="29687" class="">第 22 行我们把 src 和 typings 目录添加到 TypeScript 需要识别的文件中（也可以按照实际需要添加其他目录或者文件，比如说独立的单测文件目录 __tests__）。</p>
</li>
<li data-nodeid="29688">
<p data-nodeid="29689">因为是 React Web 项目，所以我们还需要在第 19 行将“jsx”配置为“react”。</p>
</li>
</ul>








<blockquote data-nodeid="24506">
<p data-nodeid="24507">注意：因为 Web 项目中不会直接使用 tsc 转译 TypeScript，所以我们无需配置 rootDir、outDir，甚至可以开启 noEmit 配置（如上边配置第 18 行所示，开启该配置 tsc 不会生成转译产物）。</p>
</blockquote>
<p data-nodeid="24508">接下来，我们需要结合项目所使用的构建工具集成 TypeScript 构建环境。</p>
<h4 data-nodeid="24509">构建工具集成 TypeScript</h4>
<p data-nodeid="24510">下面我们以非常常见的构建工具 Webpack 集成 TypeScript 为例。</p>
<p data-nodeid="24511">首先我们需要安装如下所示依赖，比如所有用到的第三模块类型声明（通过“npm i -D @types/模块名”进行安装）以及需要用来加载并转译 TypeScript 代码的 Webpack Loader。</p>
<pre class="lang-java" data-nodeid="24512"><code data-language="java">npm install -D typescript;
npm install -D <span class="hljs-meta">@types</span>/react;
npm install -D <span class="hljs-meta">@types</span>/react-dom;
... <span class="hljs-comment">// 其他必要依赖</span>
npm install -D ts-loader;
</code></pre>
<p data-nodeid="24513">然后，我们选择 ts-loader 作为 TypeScript 加载器，并在 webpack.config.js 配置文件中添加 resolve 和 module 规则，如下配置所示：</p>
<pre class="lang-javascript" data-nodeid="24514"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-comment">// 其他配置 ...,</span>
  <span class="hljs-attr">resolve</span>: {
    <span class="hljs-attr">extensions</span>: [<span class="hljs-string">".ts"</span>, <span class="hljs-string">".tsx"</span>, <span class="hljs-string">".js"</span>, <span class="hljs-string">".jsx"</span>, <span class="hljs-string">".json"</span>],
  },
  <span class="hljs-attr">module</span>: {
    <span class="hljs-attr">rules</span>: [
      <span class="hljs-comment">// 其他配置&nbsp;loader 规则...,</span>
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.tsx?$/</span>,
        use: [
          {
            <span class="hljs-attr">loader</span>: <span class="hljs-string">"ts-loader"</span>,
            <span class="hljs-attr">options</span>: { <span class="hljs-attr">transpileOnly</span>: <span class="hljs-literal">true</span> }
          }
        ]
      }
    ],
  },
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-comment">// ...其他配置</span>
    <span class="hljs-keyword">new</span> <span class="hljs-built_in">require</span>(<span class="hljs-string">'fork-ts-checker-webpack-plugin'</span>)({
      <span class="hljs-attr">async</span>: <span class="hljs-literal">false</span>,
      <span class="hljs-attr">tsconfig</span>: <span class="hljs-string">'...'</span> <span class="hljs-comment">// tsconfig.json 文件地址</span>
    });
  ]
  <span class="hljs-comment">// 其他配置...</span>
};
</code></pre>
<p data-nodeid="24515">首先，我们在第 4 行的 extensions 配置中添加了 .ts、.tsx 文件后缀名，是为了让 Webpack 在解析模块的时候同时识别 TypeScript 文件。</p>
<blockquote data-nodeid="24516">
<p data-nodeid="24517">注意：因为 Webpack 是以从左到右的顺序读取 extensions 配置并查找文件，所以按照如上配置，当碰到模块同名的情况，Webpack 将优先解析到 TypeScript 模块。</p>
</blockquote>
<p data-nodeid="24518">然后，我们在 17~19 行的 rules 配置中添加了 ts-loader，是为了让 Webpack 使用 ts-loader 加载和转译 .ts、.tsx 文件。</p>
<blockquote data-nodeid="30763">
<p data-nodeid="30764">一个比较好的实践是，我们可以开启 ts-loader 的 transpileOnly 配置，让 ts-loader 在处理 TypeScript 文件时，只转译而不进行静态类型检测，这样就可以提升构建速度了。</p>
<p data-nodeid="30765" class="">不过，这并不意味着构建时静态检测不重要，相反这是保证类型安全的最后一道防线。此时，我们可以通过其他性能更优的插件做静态类型检测。</p>
</blockquote>


<p data-nodeid="24523">最后，我们在第 22 行引入了 fork-ts-checker-webpack-plugin 专门对 TypeScript 文件进行构建时静态类型检测（可以通过如下命令，安装该插件）。这样，只要出现任何 TypeScript 类型错误，构建就会失败并提示错误信息。</p>
<p data-nodeid="24524">我们可以通过如下命令安装 fork-ts-checker-webpack-plugin 插件。</p>
<pre class="lang-shell" data-nodeid="24525"><code data-language="shell">npm install -D fork-ts-checker-webpack-plugin;
</code></pre>
<p data-nodeid="24526">实际上，静态类型检测确实会耗费性能和时间，尤其是项目特别庞大的时候，这个损耗会极大地降低开发体验。此时，我们可以根据实际情况优化 Webpack 配置，比如仅在生产构建时开启静态类型检测、开发构建时关闭静态类型检测，这样既可以保证开发体验，也能保证生产构建的安全性。</p>
<p data-nodeid="24527">除了使用 ts-loader 之外，现在我们也可以使用版本号大于 7 的 babel-loader 作为 TypeScript 的加载器。</p>
<p data-nodeid="24528">具体操作：首先，我们可以通过如下命令安装处理 TypeScript 的 babel preset。</p>
<pre class="lang-java" data-nodeid="24529"><code data-language="java"><span class="hljs-comment">// npm i -D babel-loader; // 确保安装版本 &gt; 7</span>
npm i -D <span class="hljs-meta">@babel</span>/preset-typescript;
</code></pre>
<blockquote data-nodeid="24530">
<p data-nodeid="24531"><strong data-nodeid="24657">注意：因为 React Web 项目必然已经安装了 babel-loader（必须依赖），所以我们不用重新安装 babel-loader，只需确保 babel-loader 的版本号大于 7 即可。</strong></p>
</blockquote>
<p data-nodeid="24532">然后，我们在 webpack.config.js 中添加支持 TypeScript 的配置，如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="24533"><code data-language="javascript"><span class="hljs-built_in">module</span>.exports = {
  ​<span class="hljs-comment">// 其他配置 ...</span>
  ​resolve: 
    ​extensions: [<span class="hljs-string">".ts"</span>, <span class="hljs-string">".tsx"</span>, <span class="hljs-string">".js"</span>, <span class="hljs-string">".jsx"</span>, <span class="hljs-string">".json"</span>]
  ​},
  ​<span class="hljs-built_in">module</span>: {
    ​rules: [
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.(js|jsx|ts|tsx)$/</span>,
        use: [<span class="hljs-string">'babel-loader'</span>]
      },
      <span class="hljs-comment">// ...其他配置</span>
    ​]
  ​},
  ​<span class="hljs-comment">// ...其他配置</span>
};
</code></pre>
<p data-nodeid="24534">在以上配置中的第 4 行、第 9 行，我们配置并使用了 babel-loader 来转换 .ts、.tsx 文件。</p>
<p data-nodeid="24535">最后，我们在 babel 配置文件中添加了如下所示的 typescript presets（参见第 5 行）。</p>
<pre class="lang-java" data-nodeid="24536"><code data-language="java">{
   <span class="hljs-string">"presets"</span>: [
     <span class="hljs-string">"@babel/preset-env"</span>,
     <span class="hljs-string">"@babel/preset-react"</span>,
     [<span class="hljs-string">'@babel/preset-typescript'</span>, { allowNamespaces: <span class="hljs-keyword">true</span> }]
   ],
   <span class="hljs-comment">// ...其他配置</span>
}
</code></pre>
<blockquote data-nodeid="24537">
<p data-nodeid="24538">注意：因为每个项目中使用的模板不同，所以 babel 配置项可能在 .babelrc、babel.config.js 单独的配置文件中或者内置在 package.json 中。</p>
</blockquote>
<p data-nodeid="24539">这样，babel-loader 就可以加载并转换 TypeScript 代码了。</p>
<blockquote data-nodeid="24540">
<p data-nodeid="24541">需要注意：因为 babel-loader 也是只对 TypeScript 代码做转换，而不进行静态类型检测，所以我们同样需要引入 fork-ts-checker-webpack-plugin 插件做静态类型检测。</p>
</blockquote>
<p data-nodeid="24542">配置好构建工具后，接下来需要迁移 JavaScript 代码，我把这个过程形容为“愚公移山”。</p>
<h4 data-nodeid="24543">愚公移山</h4>
<p data-nodeid="24544">为什么形容为“愚公移山”？因为将 JavaScript 迁移到 TypeScript 是一项个<strong data-nodeid="24671">没有太大技术含量的体力活，同时也是一项长久、渐进的过程</strong>。</p>
<p data-nodeid="24545">迁移 JavaScript 代码的具体操作：首先，我们需要逐个将 .js 文件重名为 .ts、.jsx 文件重名为 .tsx，比如将项目的主入口文件 index.js 改成 index.ts（相应的 webpack.config.js 也需要更改）。然后，我们启动本地服务（npm start）。</p>
<p data-nodeid="24546">不出意外的话，IDE（比如我们推荐的 VS Code）和 fork-ts-checker-webpack-plugin 都会提示 index.ts 有 N 个各式各样的类型错误。</p>
<p data-nodeid="24547">如果我们希望前期始于一个比较高且好的起点，比如在 tsconfig.json 中配置 noImplicitAny 为 true（禁用隐式 any），这样就会提示更多的类型错误。</p>
<blockquote data-nodeid="24548">
<p data-nodeid="24549">注意：作为过来人，建议你在 tsconfig.json 的配置上一步到位开启 strict 严格模式。一方面因为我们的课程是基于严格模式编写的，学以致用，另一方面是为了后续无需重复迁移过程，一步到位。当然，你也可以根据项目的实际诉求，选择开启严格模式一步到位或宽松配置 tsconfig。</p>
</blockquote>
<h3 data-nodeid="24550">解决错误</h3>
<p data-nodeid="24551">接下来我们要做的事情就是综合利用前面课程的知识（例如 17 讲中介绍的较为常见的错误和分析），逐个解决迁移后的 TypeScript 文件中的各种类型错误。</p>
<h4 data-nodeid="24552">缺少类型注解</h4>
<p data-nodeid="24553">我们看到的第一个错误大概率是缺少某个模块的类型声明文件 ts(7016)，比如说缺少路由组件 react-router-dom 的类型声明。</p>
<p data-nodeid="24554">此时，我们可以先通过以下命令尝试安装 DefinitelyTyped 上可能存在的类型声明依赖。</p>
<pre class="lang-java" data-nodeid="24555"><code data-language="java">npm i -D <span class="hljs-meta">@types</span>/react-router-dom;
</code></pre>
<p data-nodeid="24556">如果命令执行成功，则说明类型声明存在，并且安装成功，这也意味着我们快速且低成本地解决了一个错误。如果 DefinitelyTyped 上恰好没有定义好的依赖类型声明，那么我们就需要自己解决这个问题了。</p>
<p data-nodeid="24557">回想一下 18 讲中是如何解决依赖的 ecstatic 模块缺少类型声明问题的，首先我们需要频繁使用 declare module 补齐类型声明。然后，我们将各种补齐类型声明的文件统一放在 typings 目录中，比如示例 1 中自定义的 jQuery.d.ts（注意：DefinitelyTyped 有 jQuery 类型定义），示例 2 中声明的静态资源 svg、png、jpg、gif 文件模块的 images.d.ts。</p>
<p data-nodeid="24558">示例 1</p>
<pre class="lang-java" data-nodeid="24559"><code data-language="java"><span class="hljs-comment">// jQuery.d.ts</span>
declare <span class="hljs-keyword">module</span> <span class="hljs-string">'jQuery'</span>;
</code></pre>
<p data-nodeid="24560">示例 2</p>
<pre class="lang-typescript" data-nodeid="24561"><code data-language="typescript"><span class="hljs-comment">// images.d.ts</span>
<span class="hljs-keyword">declare</span> <span class="hljs-keyword">module</span> '*.svg';
declare <span class="hljs-keyword">module</span> '*.png';
declare <span class="hljs-keyword">module</span> '*.jpg';
declare <span class="hljs-keyword">module</span> '*.gif';
</code></pre>
<p data-nodeid="24562">关于全局变量、属性缺少类型定义的错误，我们也可以使用 declare 或者扩充相应的接口类型进行解决。</p>
<p data-nodeid="24563">首先我们可以创建一个 global.d.ts 补齐缺少的类型声明，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="24564"><code data-language="typescript"><span class="hljs-keyword">declare</span> <span class="hljs-keyword">var</span> $: <span class="hljs-built_in">any</span>;
<span class="hljs-keyword">interface</span> Window {
  __REDUX_DEVTOOLS_EXTENSION__: <span class="hljs-built_in">any</span>;
}
<span class="hljs-keyword">interface</span> NodeModule {
  hot?: {
    accept: <span class="hljs-function">(<span class="hljs-params">id: <span class="hljs-built_in">string</span>, callback: (<span class="hljs-params">...args: <span class="hljs-built_in">any</span></span>) =&gt; <span class="hljs-built_in">void</span></span>) =&gt;</span> <span class="hljs-built_in">void</span>;
  };
}
</code></pre>
<p data-nodeid="24565">在示例中的第 1 行，我们声明了全局变量 $，从而解决了找不到 $ 的 ts(2581) 错误。在第 2~4 行，我们扩充了 Window 接口，从而解决了访问 window.<strong data-nodeid="24698">REDUX_DEVTOOLS_EXTENSION</strong> 时提示属性不存在的 ts(2339) 错误。然后在第 5 到第 9 行，我们扩充了 NodeModule 接口，从而解决了调用 module.hot.accept 方法时提示的 ts(2339) 错误。</p>
<blockquote data-nodeid="31825">
<p data-nodeid="31826">注意：不要在 global.d.ts 内添加顶层的 import 或者 export 语句。</p>
<p data-nodeid="31827" class="">插播一道思考题：回忆一下 TypeScript 中 script 和 module 的区别。</p>
</blockquote>
<h4 data-nodeid="31828">隐式 any</h4>




<p data-nodeid="24571">接下来就是大量函数参数具有隐式 any 类型的 ts(7006) 错误，此时我们需要给所有函数添加类型注解。在解决这些错误时，如果我们结合 05 讲的知识（比如可选参数、剩余参数函数和函数重载）将会得心应手。</p>
<blockquote data-nodeid="24572">
<p data-nodeid="24573">一个好的实践建议：如果我们确实需要暂时使用万金油类型 any 来绕过静态类型检测，则可以声明一个具有特殊含义的全局类型 AnyToFix 来代替 any。比如我们可以在 global.d.ts 内添加如下所示的 AnyToFix 类型别名定义。</p>
</blockquote>
<pre class="lang-typescript" data-nodeid="24574"><code data-language="typescript"><span class="hljs-comment">/** 需要替换成更明确的类型 */</span>
<span class="hljs-keyword">type</span> AnyToFix = <span class="hljs-built_in">any</span>;
</code></pre>
<p data-nodeid="24575">这样，我们就可以在任何地方使用 AnyToFix 替代 any ，比如下图中的 func 函数参数 arg 的类型就是 AnyToFix。并且在条件成熟时，我们可以很方便地筛选出需要类型重构的 func 函数，然后将其参数类型修改为更明确的类型。</p>
<p data-nodeid="32356" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/4A/53/CioPOWDe5jaAcn06AAE09h1BBNU745.png" alt="Drawing 0.png" data-nodeid="32359"></p>

<h4 data-nodeid="24577">动态类型</h4>
<p data-nodeid="24578">另一类极有可能出现的错误是 JavaScript 动态类型特性造成的。</p>
<p data-nodeid="24579">如下示例第 1~3 行所示，我们习惯先定义一个空对象，再动态添加属性，迁移到 TypeScript 后就会提示一个对象上属性不存在的 ts(2339) 错误 。</p>
<pre class="lang-javascript" data-nodeid="24580"><code data-language="javascript"><span class="hljs-keyword">const</span> obj = {};
obj.id = <span class="hljs-number">1</span>; <span class="hljs-comment">// ts(2339)</span>
obj.name = <span class="hljs-string">'乾元亨利贞'</span>; <span class="hljs-comment">// ts(2339)</span>
</code></pre>
<p data-nodeid="24581">此时，我们需要通过重构代码解决这个问题，具体操作是预先定义完整的对象结构或类型断言。</p>
<p data-nodeid="24582">代码重构后的示例如下：</p>
<pre class="lang-javascript" data-nodeid="24583"><code data-language="javascript">interface IUserInfo {
  <span class="hljs-attr">id</span>: number;
  name: string;
}
<span class="hljs-keyword">const</span> obj = {} <span class="hljs-keyword">as</span> IUserInfo;
obj.id = <span class="hljs-number">1</span>; <span class="hljs-comment">// ok</span>
obj.name = <span class="hljs-string">'乾元亨利贞'</span>; <span class="hljs-comment">// ok</span>
</code></pre>
<p data-nodeid="24584">在第 5 行中，我们使用了类型断言解决了 ts(2339) 错误。</p>
<h4 data-nodeid="24585">有用的坏习惯</h4>
<p data-nodeid="24586">必要时，我们可以使用 // @ts-ignore 注释强制关闭下一行代码静态类型检测，但这绝对是一个坏习惯，示例如下：</p>
<blockquote data-nodeid="24587">
<p data-nodeid="24588"><strong data-nodeid="24719">Tips：我们需要铭记所有绕过静态类型检测的方法都是魔鬼，尽量避免使用。</strong></p>
</blockquote>
<pre class="lang-typescript" data-nodeid="24589"><code data-language="typescript"><span class="hljs-keyword">const</span> objString = {
  toString: <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-string">'乾元亨利贞'</span>
}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getString</span>(<span class="hljs-params">str: <span class="hljs-built_in">string</span></span>) </span>{
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`<span class="hljs-subst">${str}</span>`</span>);
}
<span class="hljs-comment">// @ts-ignore</span>
getString(objString); <span class="hljs-comment">// ts(2345)</span>
</code></pre>
<p data-nodeid="24590">在示例中的第 7 行，因为我们使用了 // @ts-ignore 注释强行关闭第 8 行的静态类型检测，所以第 8 行并不会提示 ts(2345) 错误。</p>
<p data-nodeid="24591">另外，我们还可以使用 // @ts-nocheck 注释强制关闭整个文件静态类型检测。<strong data-nodeid="24725">不过，我建议任何时候都不要使用这个注释。</strong></p>
<p data-nodeid="24592">另外一个有用的坏习惯是双重类型断言，即先把源类型值断言为 unknown，再把 unknown 断言为目标类型。比如上边使用 // @ts-ignore 注释的示例，我们也可以将它改造为双重类型断言，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="24593"><code data-language="typescript">getString(objString <span class="hljs-keyword">as</span> unknown <span class="hljs-keyword">as</span> <span class="hljs-built_in">string</span>);&nbsp;
</code></pre>
<p data-nodeid="24594">这样也不会提示 ts(2345) 错误了。</p>
<h4 data-nodeid="24595">自动迁移工具</h4>
<p data-nodeid="24596">如上边所提到，迁移过程是一项没有技术含量的体力活，因为其中存在很多重复、简单、有规律的操作，比如说将 JavaScript 文件修改为 TypeScript 文件、将模块引入方式从 ES5 require 改为 ES 6 import、将参数隐式 any 类型改为显式 any，这就意味着我们可以借助程序自动完成部分重复的迁移操作。</p>
<p data-nodeid="24597">比如我们可以使用 Airebnb 开源迁移工具<a href="https://github.com/airbnb/ts-migrate" data-nodeid="24733">ts-migrate</a>，快速地将 JavaScript 项目转换为基本可运行的 TypeScript 项目。因为该工具通过语法分析，可以快速推断出逻辑比较简单的函数/对象/类的类型。如下示例 1 中，JavaScript 函数 mult 经 ts-migrate 自动转换为 TypeScript 后，如下示例 2 中所示的参数 first、second 以及函数返回值类型都被明确为 number。</p>
<p data-nodeid="24598">示例 1</p>
<pre class="lang-javascript" data-nodeid="24599"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">mult</span>(<span class="hljs-params">first, second</span>) </span>{
    <span class="hljs-keyword">return</span> first * second;
}
</code></pre>
<p data-nodeid="24600">示例 2</p>
<pre class="lang-typescript" data-nodeid="24601"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">mult</span>(<span class="hljs-params">first: <span class="hljs-built_in">number</span>, second: <span class="hljs-built_in">number</span></span>): <span class="hljs-title">number</span> </span>{
    <span class="hljs-keyword">return</span> first * second;
}
</code></pre>
<h3 data-nodeid="24602">小结和预告</h3>
<p data-nodeid="24603">以上就是JavaScript 应用迁移到 TypeScript 的全部内容，在实际迁移过程中，你需要综合利用前面所有课程的知识才能得心应手。课后建议你挑选一个中小型 JavaScript 项目尝试迁移。</p>
<p data-nodeid="24604">插播一道思考题：将 JavaScript 应用迁移到 TypeScript 的过程中，最常见的错误有哪些？都是如何解决的？</p>
<p data-nodeid="24605">下一讲是结束语，我们将一起回顾过往课程中涉及的一些重难知识点、实用技巧，以及概览 TypeScript 新版本中新增的若干重要特性，敬请期待。</p>
<p data-nodeid="24606">如果你觉得本专栏有价值，欢迎分享给更多好友~</p>

---

### 精选评论


