<p data-nodeid="15965">tsconfig.json 是 TypeScript 项目的配置文件。如果一个目录下存在一个 tsconfig.json 文件，那么往往意味着这个目录就是 TypeScript 项目的根目录。</p>
<p data-nodeid="15966">tsconfig.json 包含 TypeScript 编译的相关配置，通过更改编译配置项，我们可以让 TypeScript 编译出 ES6、ES5、node 的代码。</p>
<p data-nodeid="15967">这一讲我们将分别介绍 tsconfig.json 中的相关配置选项，并对比较重要的编译选项进行着重介绍。</p>
<h3 data-nodeid="15968">compilerOptions</h3>
<p data-nodeid="15969">编译选项是 TypeScript 配置的核心部分，compilerOptions 内的配置根据功能可以分为 6 个部分，接下来我们分别介绍一下。</p>
<h4 data-nodeid="15970">项目选项</h4>
<p data-nodeid="15971">这些选项用于配置项目的运行时期望、转译 JavaScript 的输出方式和位置，以及与现有 JavaScript 代码的集成级别。</p>
<p data-nodeid="15972"><strong data-nodeid="16128">target</strong></p>
<p data-nodeid="15973">target 选项用来指定 TypeScript 编译代码的目标，不同的目标将影响代码中使用的特性是否会被降级。</p>
<p data-nodeid="15974">target 的可选值包括<code data-backticks="1" data-nodeid="16131">ES3</code>、<code data-backticks="1" data-nodeid="16133">ES5</code>、<code data-backticks="1" data-nodeid="16135">ES6</code>、<code data-backticks="1" data-nodeid="16137">ES7</code>、<code data-backticks="1" data-nodeid="16139">ES2017</code>、<code data-backticks="1" data-nodeid="16141">ES2018</code>、<code data-backticks="1" data-nodeid="16143">ES2019</code>、<code data-backticks="1" data-nodeid="16145">ES2020</code>、<code data-backticks="1" data-nodeid="16147">ESNext</code>这几种。</p>
<p data-nodeid="15975">一般情况下，target 的默认值为<code data-backticks="1" data-nodeid="16150">ES3</code>，如果不配置选项的话，代码中使用的<code data-backticks="1" data-nodeid="16152">ES6</code>特性，比如箭头函数会被转换成等价的函数表达式。</p>
<p data-nodeid="15976"><strong data-nodeid="16157">module</strong></p>
<p data-nodeid="15977">module 选项可以用来设置 TypeScript 代码所使用的模块系统。</p>
<p data-nodeid="15978">如果 target 的值设置为 ES3、ES5 ，那么 module 的默认值则为 CommonJS；如果 target 的值为 ES6 或者更高，那么 module 的默认值则为 ES6。</p>
<p data-nodeid="15979">另外，module 还支持 ES2020、UMD、AMD、System、ESNext、None 的选项。</p>
<p data-nodeid="15980"><strong data-nodeid="16164">jsx</strong></p>
<p data-nodeid="15981">jsx 选项用来控制 jsx 文件转译成 JavaScript 的输出方式。该选项只影响<code data-backticks="1" data-nodeid="16166">.tsx</code>文件的 JS 文件输出，并且没有默认值选项。</p>
<ul data-nodeid="15982">
<li data-nodeid="15983">
<p data-nodeid="15984">react: 将 jsx 改为等价的对 React.createElement 的调用，并生成 .js 文件。</p>
</li>
<li data-nodeid="15985">
<p data-nodeid="15986">react-jsx: 改为 __jsx 调用，并生成 .js 文件。</p>
</li>
<li data-nodeid="15987">
<p data-nodeid="15988">react-jsxdev: 改为 __jsx 调用，并生成 .js 文件。</p>
</li>
<li data-nodeid="15989">
<p data-nodeid="15990">preserve: 不对 jsx 进行改变，并生成 .jsx 文件。</p>
</li>
<li data-nodeid="15991">
<p data-nodeid="15992">react-native: 不对 jsx 进行改变，并生成 .js 文件。</p>
</li>
</ul>
<p data-nodeid="15993"><strong data-nodeid="16180">incremental</strong></p>
<p data-nodeid="15994">incremental 选项用来表示是否启动增量编译。incremental 为<code data-backticks="1" data-nodeid="16182">true</code>时，则会将上次编译的工程图信息保存到磁盘上的文件中。</p>
<p data-nodeid="15995"><strong data-nodeid="16187">declaration</strong></p>
<p data-nodeid="15996">declaration 选项用来表示是否为项目中的 TypeScript 或 JavaScript 文件生成 .d.ts 文件，这些 .d.ts 文件描述了模块导出的 API 类型。</p>
<p data-nodeid="15997">具体的行为你可以在<a href="http://www.typescriptlang.org/play" data-nodeid="16192">Playground</a>中编写代码，并在右侧的 .D.TS 观察输出。</p>
<p data-nodeid="15998"><strong data-nodeid="16197">sourceMap</strong></p>
<p data-nodeid="15999">sourceMap 选项用来表示是否生成<a href="https://developer.mozilla.org/docs/Tools/Debugger/How_to/Use_a_source_map" data-nodeid="16201">sourcemap 文件</a>，这些文件允许调试器和其他工具在使用实际生成的 JavaScript 文件时，显示原始的 TypeScript 代码。</p>
<p data-nodeid="16000">Source map 文件以 .js.map （或 .jsx.map）文件的形式被生成到与 .js 文件相对应的同一个目录下。</p>
<p data-nodeid="16001"><strong data-nodeid="16207">lib</strong></p>
<p data-nodeid="16002">在 13 讲中我们介绍过，安装 TypeScript 时会顺带安装一个 lib.d.ts 声明文件，并且默认包含了 ES5、DOM、WebWorker、ScriptHost 的库定义。</p>
<p data-nodeid="16003">lib 配置项允许我们更细粒度地控制代码运行时的库定义文件，比如说 Node.js 程序，由于并不依赖浏览器环境，因此不需要包含 DOM 类型定义；而如果需要使用一些最新的、高级 ES 特性，则需要包含 ESNext  类型。</p>
<p data-nodeid="16004">具体的详情你可以在<a href="https://github.com/microsoft/TypeScript/tree/master/lib" data-nodeid="16213">TypeScript 源码</a>中查看完整的列表，并且自定义编译需要的<code data-backticks="1" data-nodeid="16215">lib</code>类型定义。</p>
<h4 data-nodeid="16005">严格模式</h4>
<p data-nodeid="16006">TypeScript 兼容 JavaScript 的代码，默认选项允许相当大的灵活性来适应这些模式。</p>
<p data-nodeid="16007">在迁移 JavaScript 代码时，你可以先暂时关闭一些严格模式的设置。在正式的 TypeScript 项目中，我推荐开启 strict 设置启用更严格的类型检查，以减少错误的发生。</p>
<p data-nodeid="16008"><strong data-nodeid="16223">strict</strong></p>
<p data-nodeid="16009">开启 strict 选项时，一般我们会同时开启一系列的类型检查选项，以便更好地保证程序的正确性。</p>
<p data-nodeid="16010">strict 为 true 时，一般我们会开启以下编译配置。</p>
<ul data-nodeid="16011">
<li data-nodeid="16012">
<p data-nodeid="16013">alwaysStrict：保证编译出的文件是 ECMAScript 的严格模式，并且每个文件的头部会添加 'use strict'。</p>
</li>
<li data-nodeid="16014">
<p data-nodeid="16015">strictNullChecks：更严格地检查 null 和 undefined 类型，比如数组的 find 方法的返回类型将是更严格的 T | undefined。</p>
</li>
<li data-nodeid="16016">
<p data-nodeid="16017">strictBindCallApply：更严格地检查 call、bind、apply 函数的调用，比如会检查参数的类型与函数类型是否一致。</p>
</li>
<li data-nodeid="16018">
<p data-nodeid="16019">strictFunctionTypes：更严格地检查函数参数类型和类型兼容性。</p>
</li>
<li data-nodeid="16020">
<p data-nodeid="16021">strictPropertyInitialization：更严格地检查类属性初始化，如果类的属性没有初始化，则会提示错误。</p>
</li>
<li data-nodeid="16022">
<p data-nodeid="16023">noImplicitAny：禁止隐式 any 类型，需要显式指定类型。TypeScript 在不能根据上下文推断出类型时，会回退到 any 类型。</p>
</li>
<li data-nodeid="16024">
<p data-nodeid="16025">noImplicitThis：禁止隐式 this 类型，需要显示指定 this 的类型。</p>
</li>
</ul>
<blockquote data-nodeid="16026">
<p data-nodeid="16027"><strong data-nodeid="16244">注意：将</strong><code data-backticks="1" data-nodeid="16240">strict</code>设置为<code data-backticks="1" data-nodeid="16242">true</code>，开启严格模式，是本课程极力推荐的最佳实践。</p>
</blockquote>
<h4 data-nodeid="16028">额外检查</h4>
<p data-nodeid="16029">TypeScript 支持一些额外的代码检查，在某种程度上介于编译器与静态分析工具之间。如果你想要更多的代码检查，可能更适合使用 ESLint 这类工具。</p>
<ul data-nodeid="16030">
<li data-nodeid="16031">
<p data-nodeid="16032"><strong data-nodeid="16251">noImplicitReturns</strong>：禁止隐式返回。如果代码的逻辑分支中有返回，则所有的逻辑分支都应该有返回。</p>
</li>
<li data-nodeid="16033">
<p data-nodeid="16034"><strong data-nodeid="16256">noUnusedLocals</strong>：禁止未使用的本地变量。如果一个本地变量声明未被使用，则会抛出错误。</p>
</li>
<li data-nodeid="16035">
<p data-nodeid="16036"><strong data-nodeid="16261">noUnusedParameters</strong>：禁止未使用的函数参数。如果函数的参数未被使用，则会抛出错误。</p>
</li>
<li data-nodeid="16037">
<p data-nodeid="16038"><strong data-nodeid="16266">noFallthroughCasesInSwitch</strong>：禁止 switch 语句中的穿透的情况。开启 noFallthroughCasesInSwitch 后，如果 switch 语句的流程分支中没有 break 或 return ，则会抛出错误，从而避免了意外的 swtich 判断穿透导致的问题。</p>
</li>
</ul>
<h4 data-nodeid="16039">模块解析</h4>
<p data-nodeid="16040">模块解析部分的编译配置会影响代码中模块导入以及编译相关的配置。</p>
<p data-nodeid="16041"><strong data-nodeid="16272">moduleResolution</strong></p>
<p data-nodeid="16042">moduleResolution 用来指定模块解析策略。</p>
<p data-nodeid="16043">module 配置值为 AMD、UMD、System、ES6 时，moduleResolution 默认为 classic，否则为 node。在目前的新代码中，我们一般都是使用 node，而不使用classic。</p>
<p data-nodeid="16044">具体的模块解析策略，你可以查看<a href="https://www.typescriptlang.org/docs/handbook/module-resolution.html#module-resolution-strategies" data-nodeid="16278">模块解析策略</a>。</p>
<p data-nodeid="16045"><strong data-nodeid="16283">baseUrl</strong></p>
<p data-nodeid="16046">baseUrl 指的是基准目录，用来设置解析非绝对路径模块名时的基准目录。比如设置 baseUrl 为 './' 时，TypeScript 将会从 tsconfig.json 所在的目录开始查找文件。</p>
<p data-nodeid="16047"><strong data-nodeid="16292">paths</strong></p>
<p data-nodeid="16048">paths 指的是路径设置，用来将模块路径重新映射到相对于 baseUrl 定位的其他路径配置。这里我们可以将 paths 理解为 webpack 的 alias 别名配置。</p>
<p data-nodeid="16049">下面我们看一个具体的示例：</p>
<pre class="lang-json" data-nodeid="16050"><code data-language="json">{
  <span class="hljs-attr">"compilerOptions"</span>: {
    <span class="hljs-attr">"paths"</span>: {
      <span class="hljs-attr">"@src/*"</span>: [<span class="hljs-string">"src/*"</span>],
      <span class="hljs-attr">"@utils/*"</span>: [<span class="hljs-string">"src/utils/*"</span>]
    }
  }
}
</code></pre>
<p data-nodeid="16051">在上面的例子中，TypeScript 模块解析支持以一些自定义前缀来寻找模块，避免在代码中出现过长的相对路径。</p>
<blockquote data-nodeid="16052">
<p data-nodeid="16053"><strong data-nodeid="16299">注意：因为 paths 中配置的别名仅在类型检测时生效，所以在使用 tsc 转译或者 webpack 构建 TypeScript 代码时，我们需要引入额外的插件将源码中的别名替换成正确的相对路径。</strong></p>
</blockquote>
<p data-nodeid="16054"><strong data-nodeid="16303">rootDirs</strong></p>
<p data-nodeid="16055">rootDirs 可以指定多个目录作为根目录。这将允许编译器在这些“虚拟”目录中解析相对应的模块导入，就像它们被合并到同一目录中一样。</p>
<p data-nodeid="16056"><strong data-nodeid="16308">typeRoots</strong></p>
<p data-nodeid="16057">typeRoots 用来指定类型文件的根目录。</p>
<p data-nodeid="16058">在默认情况下，所有 node_modules/@types 中的任何包都被认为是可见的。如果手动指定了 typeRoots ，则仅会从指定的目录里查找类型文件。</p>
<p data-nodeid="16059"><strong data-nodeid="16316">types</strong></p>
<p data-nodeid="16060">在默认情况下，所有的 typeRoots 包都将被包含在编译过程中。</p>
<p data-nodeid="16061">手动指定 types 时，只有列出的包才会被包含在全局范围内，如下示例：</p>
<pre class="lang-json" data-nodeid="16062"><code data-language="json">{
  <span class="hljs-attr">"compilerOptions"</span>: {
    <span class="hljs-attr">"types"</span>: [<span class="hljs-string">"node"</span>, <span class="hljs-string">"jest"</span>, <span class="hljs-string">"express"</span>]
  }
}
</code></pre>
<p data-nodeid="16063">在上述示例中可以看到，手动指定 types 时 ，仅包含了 node、jest、express 三个  node 模块的类型包。</p>
<p data-nodeid="16064"><strong data-nodeid="16323">allowSyntheticDefaultImports</strong></p>
<p data-nodeid="16065">allowSyntheticDefaultImports****允许合成默认导出。</p>
<p data-nodeid="16066">当 allowSyntheticDefaultImports 设置为 true，即使一个模块没有默认导出（export default），我们也可以在其他模块中像导入包含默认导出模块一样的方式导入这个模块，如下示例：</p>
<pre class="lang-typescript" data-nodeid="16067"><code data-language="typescript"><span class="hljs-comment">// allowSyntheticDefaultImports: true 可以使用</span>
<span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-comment">// allowSyntheticDefaultImports: false</span>
<span class="hljs-keyword">import</span> * <span class="hljs-keyword">as</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
</code></pre>
<p data-nodeid="16068">在上面的示例中，对于没有默认导出的模块 react，如果设置了 allowSyntheticDefaultImports 为 true，则可以直接通过 import 导入 react；但如果设置 allowSyntheticDefaultImports 为 false，则需要通过 import * as 导入 react。</p>
<p data-nodeid="16069"><strong data-nodeid="16334">esModuleInterop</strong></p>
<p data-nodeid="16070">esModuleInterop 指的是 ES 模块的互操作性。</p>
<p data-nodeid="16071">在默认情况下，TypeScript 像 ES6 模块一样对待 CommonJS / AMD / UMD，但是此时的 TypeScript 代码转移会导致不符合 ES6 模块规范。不过，开启 esModuleInterop 后，这些问题都将得到修复。</p>
<p data-nodeid="16072">一般情况下，在启用 esModuleInterop 时，我们将同时启用 allowSyntheticDefaultImports。</p>
<h4 data-nodeid="16073">Source Maps</h4>
<p data-nodeid="16074">为了支持丰富的调试工具，并为开发人员提供有意义的崩溃报告，TypeScript 支持生成符合 JavaScript Source Map 标准的附加文件（即 .map 文件）。</p>
<p data-nodeid="16075"><strong data-nodeid="16343">sourceRoot</strong></p>
<p data-nodeid="16076">sourceRoot 用来指定调试器需要定位的 TypeScript 文件位置，而不是相对于源文件的路径。</p>
<p data-nodeid="16077">sourceRoot<strong data-nodeid="16350">的</strong>取值可以是路径或者 URL。</p>
<p data-nodeid="16078"><strong data-nodeid="16354">mapRoot</strong></p>
<p data-nodeid="16079">mapRoot 用来指定调试器需要定位的 source map 文件的位置，而不是生成的文件位置。</p>
<p data-nodeid="16080"><strong data-nodeid="16359">inlineSourceMap</strong></p>
<p data-nodeid="16081">开启 inlineSourceMap 选项时，将不会生成 .js.map 文件，而是将 source map 文件内容生成内联字符串写入对应的 .js 文件中。虽然这样会生成较大的 JS 文件，但是在不支持 .map 调试的环境下将会很方便。</p>
<p data-nodeid="16082"><strong data-nodeid="16364">inlineSources</strong></p>
<p data-nodeid="16083">开启 inlineSources 选项时，将会把源文件的所有内容生成内联字符串并写入 source map 中。这个选项的用途和 inlineSourceMap 是一样的。</p>
<h4 data-nodeid="16084">实验选项</h4>
<p data-nodeid="16085">TypeScript 支持一些尚未在 JavaScript 提案中稳定的语言特性，因此在 TypeScript 中实验选项是作为实验特性存在的。</p>
<p data-nodeid="16086"><strong data-nodeid="16371">experimentalDecorators</strong></p>
<p data-nodeid="16087">experimentalDecorators****选项会开启<a href="https://github.com/tc39/proposal-decorators" data-nodeid="16377">装饰器提案</a>的特性。</p>
<p data-nodeid="16088"><strong data-nodeid="16382">目前，装饰器提案在 stage 2 仍未完全批准到 JavaScript 规范中，且 TypeScript 实现的装饰器版本可能和 JavaScript 有所不同。</strong></p>
<p data-nodeid="16089"><strong data-nodeid="16386">emitDecoratorMetadata</strong></p>
<p data-nodeid="16090">emitDecoratorMetadata****选项允许装饰器使用反射数据的特性。</p>
<h4 data-nodeid="16091">高级选项</h4>
<p data-nodeid="16092"><strong data-nodeid="16394">skipLibCheck</strong></p>
<p data-nodeid="16093">开启 skipLibCheck****选项，表示可以跳过检查声明文件。</p>
<p data-nodeid="16094">如果我们开启了这个选项，则可以节省编译期的时间，但可能会牺牲类型系统的准确性。<strong data-nodeid="16406">在设置该选项时，我推荐值为</strong>true**。**</p>
<p data-nodeid="16095"><strong data-nodeid="16410">forceConsistentCasingInFileNames</strong></p>
<p data-nodeid="16096">TypeScript 对文件的大小写是敏感的。如果有一部分的开发人员在大小写敏感的系统开发，而另一部分的开发人员在大小写不敏感的系统开发，则可能会出现问题。</p>
<p data-nodeid="16097"><strong data-nodeid="16415">开启此选项后，如果开发人员正在使用和系统不一致的大小写规则，则会抛出错误。</strong></p>
<h3 data-nodeid="16098">include</h3>
<p data-nodeid="16099">include 用来指定需要包括在 TypeScript 项目中的文件或者文件匹配路径。如果我们指定了 files 配置项，则 include 的 默认值为 []，否则 include 默认值为 ["**/*"] ，即包含了目录下的所有文件。</p>
<p data-nodeid="16100">如果 glob 匹配的文件中没有包含文件的扩展名，则只有 files 支持的扩展名会被包含。</p>
<p data-nodeid="16101">一般来说，include 的默认值为.ts、.tsx 和 .d.ts。如果我们开启了 allowJs 选项，还包括 .js 和 .jsx 文件。</p>
<h3 data-nodeid="16102">exclude</h3>
<p data-nodeid="16103">exclude 用来指定解析 include 配置中需要跳过的文件或者文件匹配路径。一般来说，exclude 的默认值为 ["node_modules", "bower_components", "jspm_packages"]。</p>
<blockquote data-nodeid="16104">
<p data-nodeid="16105"><strong data-nodeid="16461">需要注意</strong>：<code data-backticks="1" data-nodeid="16457">exclude</code>配置项只会改变<code data-backticks="1" data-nodeid="16459">include</code>配置项中的结果。</p>
</blockquote>
<h3 data-nodeid="16106">files</h3>
<p data-nodeid="16107">files 选项用来指定 TypeScript 项目中需要包含的文件列表。</p>
<p data-nodeid="16108">如果项目非常小，那么我们可以使用 files<strong data-nodeid="16472">指定项目的文件，否则更适合使用</strong>include<strong data-nodeid="16473">指定项目文件。</strong></p>
<h3 data-nodeid="16109">extends</h3>
<p data-nodeid="16110">extends 配置项的值是一个字符串，用来声明当前配置需要继承的另外一个配置的路径，这个路径使用 Node.js 风格的解析模式。TypeScript 首先会加载 extends 的配置文件，然后使用当前的 tsconfig.json 文件里的配置覆盖继承的文件里的配置。</p>
<p data-nodeid="16111">TypeScript 会基于当前 tsconfig.json 配置文件的路径解析所继承的配置文件中出现的相对路径。</p>
<h3 data-nodeid="16112">小结和预告</h3>
<p data-nodeid="16113">tsconfig.json 是 TypeScript 项目非常重要的配置文件，这一讲我们着重介绍了编译选项中不同功能的常用选项，更多的 TypeScript 配置可以在<a href="https://www.typescriptlang.org/tsconfig" data-nodeid="16481">TSConfig Reference</a>中查看学习。</p>
<p data-nodeid="16114">你也可以结合这一讲的内容新建项目并更改 tsconfig.json 实践学习。</p>
<p data-nodeid="16115">17 讲我们将介绍解析在 TypeScript 项目开发中常见的类型错误以及如何为 TypeScript 类型编写单元测试，敬请期待！</p>
<p data-nodeid="16116">另外，如果你觉得本专栏有价值，欢迎分享给更多好友。</p>

---

### 精选评论


