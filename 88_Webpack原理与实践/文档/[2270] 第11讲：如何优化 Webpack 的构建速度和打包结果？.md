<p data-nodeid="1181" class="">你好，我是汪磊，今天我们要一起探索的是 Webpack 在生产模式打包过程中的常用配置以及一些优化插件。</p>

<p data-nodeid="946">在前面的课时中，我们了解到的一些用法和特性都是为了在开发阶段能够拥有更好的开发体验。而随着这些体验的提升，一个新的问题出现在我们面前：我们的打包结果会变得越来越臃肿。</p>
<p data-nodeid="947">这是因为在这个过程中 Webpack 为了实现这些特性，会自动往打包结果中添加一些内容。例如我们之前用到的 Source Map 和 HMR，它们都会在输出结果中添加额外代码来实现各自的功能。</p>
<p data-nodeid="948">但是这些额外的代码对生产环境来说是冗余的。因为生产环境和开发环境有很大的差异，在生产环境中我们强调的是以更少量、更高效的代码完成业务功能，也就是注重运行效率。而开发环境中我们注重的只是开发效率。</p>
<p data-nodeid="949">那针对这个问题，Webpack 4 推出了 mode 的用法，为我们提供了不同模式下的一些预设配置，其中生产模式下就已经包括了很多优化配置。</p>
<p data-nodeid="950">同时 Webpack 也建议我们为不同的工作环境创建不同的配置，以便于让我们的打包结果可以适用于不同的环境。</p>
<p data-nodeid="951">接下来我们一起来探索一下生产环境中的一些优化方式和注意事项。</p>
<h3 data-nodeid="952">不同环境下的配置</h3>
<p data-nodeid="953">我们先为不同的工作环境创建不同的 Webpack 配置。创建不同环境配置的方式主要有两种：</p>
<ul data-nodeid="2134">
<li data-nodeid="2135">
<p data-nodeid="2136" class="">在配置文件中添加相应的判断条件，根据环境不同导出不同配置；</p>
</li>
<li data-nodeid="2137">
<p data-nodeid="2138">为不同环境单独添加一个配置文件，一个环境对应一个配置文件。</p>
</li>
</ul>


<p data-nodeid="959">我们分别尝试一下通过这两种方式，为开发环境和生产环境创建不同配置。</p>
<p data-nodeid="960">首先我们来看在配置文件中添加判断的方式。我们回到配置文件中，Webpack 配置文件还支持导出一个函数，然后在函数中返回所需要的配置对象。这个函数可以接收两个参数，第一个是 env，是我们通过 CLI 传递的环境名参数，第二个是 argv，是运行 CLI 过程中的所有参数。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="961"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-built_in">module</span>.exports = <span class="hljs-function">(<span class="hljs-params">env, argv</span>) =&gt;</span> {
  <span class="hljs-keyword">return</span> {
    <span class="hljs-comment">// ... webpack 配置</span>
  }
}
</code></pre>
<p data-nodeid="962">那我们就可以借助这个特点，为开发环境和生产环境创建不同配置。我先将不同模式下公共的配置定义为一个 config 对象，具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="963"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-built_in">module</span>.exports = <span class="hljs-function">(<span class="hljs-params">env, argv</span>) =&gt;</span> {
  <span class="hljs-keyword">const</span> config = {
    <span class="hljs-comment">// ... 不同模式下的公共配置</span>
  }
  <span class="hljs-keyword">return</span> config
}
</code></pre>
<p data-nodeid="964">然后通过判断，再为 config 对象添加不同环境下的特殊配置。具体如下：</p>
<pre class="lang-javascript" data-nodeid="965"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-built_in">module</span>.exports = <span class="hljs-function">(<span class="hljs-params">env, argv</span>) =&gt;</span> {
  <span class="hljs-keyword">const</span> config = {
    <span class="hljs-comment">// ... 不同模式下的公共配置</span>
  }

  <span class="hljs-keyword">if</span> (env === <span class="hljs-string">'development'</span>) {
    <span class="hljs-comment">// 为 config 添加开发模式下的特殊配置</span>
    config.mode = <span class="hljs-string">'development'</span>
    config.devtool = <span class="hljs-string">'cheap-eval-module-source-map'</span>
  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (env === <span class="hljs-string">'production'</span>) {
    <span class="hljs-comment">// 为 config 添加生产模式下的特殊配置</span>
    config.mode = <span class="hljs-string">'production'</span>
    config.devtool = <span class="hljs-string">'nosources-source-map'</span>
  }

  <span class="hljs-keyword">return</span> config
}
</code></pre>
<p data-nodeid="966">例如这里，我们判断 env 等于 development（开发模式）的时候，我们将 mode 设置为 development，将 devtool 设置为 cheap-eval-module-source-map；而当 env 等于 production（生产模式）时，我们又将 mode 和 devtool 设置为生产模式下需要的值。</p>
<p data-nodeid="967">当然，你还可以分别为不同模式设置其他不同的属性、插件，这也都是类似的。</p>
<p data-nodeid="968">通过这种方式完成配置过后，我们打开命令行终端，这里我们再去执行 webpack 命令时就可以通过 --env 参数去指定具体的环境名称，从而实现在不同环境中使用不同的配置。</p>
<p data-nodeid="969">那这就是通过在 Webpack 配置文件导出的函数中对环境进行判断，从而实现不同环境对应不同配置。这种方式是 Webpack 建议的方式。</p>
<p data-nodeid="970">你也可以直接定义环境变量，然后在全局判断环境变量，根据环境变量的不同导出不同配置。这种方式也是类似的，这里我们就不做过多介绍了。</p>
<h4 data-nodeid="971">不同环境的配置文件</h4>
<p data-nodeid="972">通过判断环境名参数返回不同配置对象的方式只适用于中小型项目，因为一旦项目变得复杂，我们的配置也会一起变得复杂起来。所以对于大型的项目来说，还是建议使用不同环境对应不同配置文件的方式来实现。</p>
<p data-nodeid="973">一般在这种方式下，项目中最少会有三个 webpack 的配置文件。其中两个用来分别适配开发环境和生产环境，另外一个则是公共配置。因为开发环境和生产环境的配置并不是完全不同的，所以需要一个公共文件来抽象两者相同的配置。具体配置文件结构如下：</p>
<pre class="lang-java" data-nodeid="974"><code data-language="java">.
├── webpack.common.js ···························· 公共配置
├── webpack.dev.js ······························· 开发模式配置
└── webpack.prod.js ······························ 生产模式配置
</code></pre>
<p data-nodeid="975">首先我们在项目根目录下新建一个 webpack.common.js，在这个文件中导出不同模式下的公共配置；然后再来创建一个 webpack.dev.js 和一个 webpack.prod.js 分别定义开发和生产环境特殊的配置。</p>
<p data-nodeid="976">在不同环境的具体配置中我们先导入公共配置对象，然后这里可以使用 Object.assign 方法把公共配置对象复制到具体环境的配置对象中，并且同时去覆盖其中的一些配置。具体如下：</p>
<pre class="lang-javascript" data-nodeid="977"><code data-language="javascript"><span class="hljs-comment">// ./webpack.common.js</span>
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-comment">// ... 公共配置</span>
}
<span class="hljs-comment">// ./webpack.prod.js</span>
<span class="hljs-keyword">const</span> common = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./webpack.common'</span>)
<span class="hljs-built_in">module</span>.exports = <span class="hljs-built_in">Object</span>.assign(common, {
  <span class="hljs-comment">// 生产模式配置</span>
})
<span class="hljs-comment">// ./webpack.dev.js</span>
<span class="hljs-keyword">const</span> common = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./webpack.common'</span>)
<span class="hljs-built_in">module</span>.exports = <span class="hljs-built_in">Object</span>.assign(common, {
  <span class="hljs-comment">// 开发模式配置</span>
})
</code></pre>
<p data-nodeid="978">如果你熟悉 Object.assign 方法，就应该知道，这个方法会完全覆盖掉前一个对象中的同名属性。这个特点对于普通值类型属性的覆盖都没有什么问题。但是像配置中的 plugins 这种数组，我们只是希望在原有公共配置的插件基础上添加一些插件，那 Object.assign 就做不到了。</p>
<p data-nodeid="979">所以我们需要更合适的方法来合并这里的配置与公共的配置。你可以使用 <a href="http://lodash.com" data-nodeid="1081">Lodash</a> 提供的 merge 函数来实现，不过社区中提供了更为专业的模块 <a href="https://github.com/survivejs/webpack-merge" data-nodeid="1085">webpack-merge</a>，它专门用来满足我们这里合并 Webpack 配置的需求。</p>
<p data-nodeid="980">我们可以先通过 npm 安装一下 webpack-merge 模块。具体命令如下：</p>
<pre class="lang-shell" data-nodeid="981"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm i webpack-merge --save-dev </span>
<span class="hljs-meta">#</span><span class="bash"> or yarn add webpack-merge --dev</span>
</code></pre>
<p data-nodeid="5917" class="">安装完成过后我们回到配置文件中，这里先载入这个模块。那这个模块导出的就是一个 merge 函数，我们使用这个函数来合并这里的配置与公共的配置。具体代码如下：</p>



<pre class="lang-javascript" data-nodeid="983"><code data-language="javascript"><span class="hljs-comment">// ./webpack.common.js</span>
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-comment">// ... 公共配置</span>
}
<span class="hljs-comment">// ./webpack.prod.js</span>
<span class="hljs-keyword">const</span> merge = <span class="hljs-built_in">require</span>(<span class="hljs-string">'webpack-merge'</span>)
<span class="hljs-keyword">const</span> common = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./webpack.common'</span>)
<span class="hljs-built_in">module</span>.exports = merge(common, {
  <span class="hljs-comment">// 生产模式配置</span>
})
<span class="hljs-comment">// ./webpack.dev.jss</span>
<span class="hljs-keyword">const</span> merge = <span class="hljs-built_in">require</span>(<span class="hljs-string">'webpack-merge'</span>)
<span class="hljs-keyword">const</span> common = <span class="hljs-built_in">require</span>(<span class="hljs-string">'./webpack.common'</span>)
<span class="hljs-built_in">module</span>.exports = merge(common, {
  <span class="hljs-comment">// 开发模式配置</span>
})
</code></pre>
<p data-nodeid="984">使用 webpack-merge 过后，我们这里的配置对象就可以跟普通的 webpack 配置一样，需要什么就配置什么，merge 函数内部会自动处理合并的逻辑。</p>
<p data-nodeid="985">分别配置完成过后，我们再次回到命令行终端，然后尝试运行 webpack 打包。不过因为这里已经没有默认的配置文件了，所以我们需要通过 --config 参数来指定我们所使用的配置文件路径。例如：</p>
<pre class="lang-shell" data-nodeid="986"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> webpack --config webpack.prod.js</span>
</code></pre>
<p data-nodeid="3087" class="">当然，如果你觉得这样操作让我们的命令变得更复杂了，那你可以把这个构建命令定义到 npm scripts 中，方便使用。</p>


<h3 data-nodeid="988">生产模式下的优化插件</h3>
<p data-nodeid="989">在 Webpack 4 中新增的 production 模式下，内部就自动开启了很多通用的优化功能。对于使用者而言，开箱即用是非常方便的，但是对于学习者而言，这种开箱即用会导致我们忽略掉很多需要了解的东西。以至于出现问题无从下手。</p>
<p data-nodeid="990"><br>
如果你想要深入了解 Webpack 的使用，我建议你去单独研究每一个配置背后的作用。这里我们先一起学习 production 模式下几个主要的优化功能，顺便了解一下 Webpack 如何优化打包结果。</p>
<h4 data-nodeid="991">Define Plugin</h4>
<p data-nodeid="992">首先是 DefinePlugin，DefinePlugin 是用来为我们代码中注入全局成员的。在 production 模式下，默认通过这个插件往代码中注入了一个 process.env.NODE_ENV。很多第三方模块都是通过这个成员去判断运行环境，从而决定是否执行例如打印日志之类的操作。</p>
<p data-nodeid="993">这里我们来单独使用一下这个插件。我们回到配置文件中，DefinePlugin 是一个内置的插件，所以我们先导入 webpack 模块，然后再到 plugins 中添加这个插件。这个插件的构造函数接收一个对象参数，对象中的成员都可以被注入到代码中。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="994"><code data-language="javascript">// ./webpack.config.js
const webpack = require('webpack')
module.exports = {
/  // ... 其他配置
  plugins: [
    new webpack.DefinePlugin({
      API_BASE_URL: 'https://api.example.com'
    })
  ]
}
</code></pre>
<p data-nodeid="995">例如我们这里通过 DefinePlugin 定义一个 API_BASE_URL，用来为我们的代码注入 API 服务地址，它的值是一个字符串。</p>
<p data-nodeid="996">然后我们回到代码中打印这个 API_BASE_URL。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="997"><code data-language="javascript"><span class="hljs-comment">// ./src/main.js</span>
<span class="hljs-built_in">console</span>.log(API_BASE_URL)
</code></pre>
<p data-nodeid="998">完成以后我们打开控制台，然后运行 webpack 打包。打包完成过后我们找到打包的结果，然后找到 main.js 对应的模块。具体结果如下：</p>
<p data-nodeid="999"><img src="https://s0.lgstatic.com/i/image/M00/10/E9/Ciqc1F7LaHWAV84JAADT7IV7CvE825.png" alt="image (3).png" data-nodeid="1115"></p>
<p data-nodeid="1000">这里我们发现 DefinePlugin 其实就是把我们配置的字符串内容直接替换到了代码中，而目前这个字符串的内容为 https://api.example.com，字符串中并没有包含引号，所以替换进来语法自然有问题。</p>
<p data-nodeid="1001">正确的做法是传入一个字符串字面量语句。具体实现如下：</p>
<pre class="lang-javascript" data-nodeid="1002"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> webpack = <span class="hljs-built_in">require</span>(<span class="hljs-string">'webpack'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-comment">// ... 其他配置</span>
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> webpack.DefinePlugin({
      <span class="hljs-comment">// 值要求的是一个代码片段</span>
      <span class="hljs-attr">API_BASE_URL</span>: <span class="hljs-string">'"https://api.example.com"'</span>
    })
  ]
}
</code></pre>
<p data-nodeid="1003">这样代码内的 API_BASE_URL 就会被替换为 "https://api.example.com"。具体结果如下：</p>
<p data-nodeid="1004"><img src="https://s0.lgstatic.com/i/image/M00/10/F5/CgqCHl7LaH-AMPKrAADQsK6wHzM717.png" alt="image (4).png" data-nodeid="1129"></p>
<p data-nodeid="1005">另外，这里有一个非常常用的小技巧，如果我们需要注入的是一个值，就可以通过 JSON.stringify 的方式来得到表示这个值的字面量。这样就不容易出错了。具体实现如下：</p>
<pre class="lang-javascript" data-nodeid="1006"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> webpack = <span class="hljs-built_in">require</span>(<span class="hljs-string">'webpack'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-comment">// ... 其他配置</span>
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> webpack.DefinePlugin({
      <span class="hljs-comment">// 值要求的是一个代码片段</span>
      <span class="hljs-attr">API_BASE_URL</span>: <span class="hljs-built_in">JSON</span>.stringify(<span class="hljs-string">'https://api.example.com'</span>)
    })
  ]
}
</code></pre>
<p data-nodeid="1007">DefinePlugin 的作用虽然简单，但是却非常有用，我们可以用它在代码中注入一些可能变化的值。</p>
<h3 data-nodeid="1008">Mini CSS Extract Plugin</h3>
<p data-nodeid="1009">对于 CSS 文件的打包，一般我们会使用 style-loader 进行处理，这种处理方式最终的打包结果就是 CSS 代码会内嵌到 JS 代码中。</p>
<p data-nodeid="4507" class="">mini-css-extract-plugin 是一个可以将 CSS 代码从打包结果中提取出来的插件，它的使用非常简单，同样也需要先通过 npm 安装一下这个插件。具体命令如下：</p>




<pre class="lang-shell" data-nodeid="1012"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm i mini-css-extract-plugin --save-dev</span>
</code></pre>
<p data-nodeid="1013">安装完成过后，我们回到 Webpack 的配置文件。具体配置如下：</p>
<pre class="lang-javascript" data-nodeid="1014"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> MiniCssExtractPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mini-css-extract-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">mode</span>: <span class="hljs-string">'none'</span>,
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">main</span>: <span class="hljs-string">'./src/index.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span>
  },
  <span class="hljs-attr">module</span>: {
    <span class="hljs-attr">rules</span>: [
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.css$/</span>,
        use: [
          <span class="hljs-comment">// 'style-loader', // 将样式通过 style 标签注入</span>
          MiniCssExtractPlugin.loader,
          <span class="hljs-string">'css-loader'</span>
        ]
      }
    ]
  },
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> MiniCssExtractPlugin()
  ]
}
</code></pre>
<p data-nodeid="1015">我们这里先导入这个插件模块，导入过后我们就可以将这个插件添加到配置对象的 plugins 数组中了。这样 Mini CSS Extract Plugin 在工作时就会自动提取代码中的 CSS 了。</p>
<p data-nodeid="1016">除此以外，Mini CSS Extract Plugin 还需要我们使用 MiniCssExtractPlugin 中提供的 loader 去替换掉 style-loader，以此来捕获到所有的样式。</p>
<p data-nodeid="1017">这样的话，打包过后，样式就会存放在独立的文件中，直接通过 link 标签引入页面。</p>
<p data-nodeid="1018" class="">不过这里需要注意的是，如果你的 CSS 体积不是很大的话，提取到单个文件中，效果可能适得其反，因为单独的文件就需要单独请求一次。个人经验是如果 CSS 超过 200KB 才需要考虑是否提取出来，作为单独的文件。</p>
<h4 data-nodeid="1019">Optimize CSS Assets Webpack Plugin</h4>
<p data-nodeid="1020">使用了 Mini CSS Extract Plugin 过后，样式就被提取到单独的 CSS 文件中了。但是这里同样有一个小问题。</p>
<p data-nodeid="1021">我们回到命令行，这里我们以生产模式运行打包。那按照之前的了解，生产模式下会自动压缩输出的结果，我们可以打开打包生成的 JS 文件。具体结果如下：</p>
<p data-nodeid="1022"><img src="https://s0.lgstatic.com/i/image/M00/10/E9/Ciqc1F7LaIqATAzSAAQyyz8qCXE919.png" alt="image (5).png" data-nodeid="1146"></p>
<p data-nodeid="1023">然后我们再打开输出的样式文件。具体结果如下：</p>
<p data-nodeid="1024"><img src="https://s0.lgstatic.com/i/image/M00/10/E9/Ciqc1F7LaJGAKXO2AAEBLBn8-rQ140.png" alt="image (6).png" data-nodeid="1150"></p>
<p data-nodeid="1025">这里我们发现 JavaScript 文件正常被压缩了，而样式文件并没有被压缩。</p>
<p data-nodeid="1026">这是因为，Webpack 内置的压缩插件仅仅是针对 JS 文件的压缩，其他资源文件的压缩都需要额外的插件。</p>
<p data-nodeid="1027">Webpack 官方推荐了一个 <a href="https://www.npmjs.com/package/optimize-css-assets-webpack-plugin" data-nodeid="1156">Optimize CSS Assets Webpack Plugin</a> 插件。我们可以使用这个插件来压缩我们的样式文件。</p>
<p data-nodeid="1028">我们回到命令行，先来安装这个插件，具体命令如下：</p>
<pre class="lang-shell" data-nodeid="1029"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm i optimize-css-assets-webpack-plugin --save-dev</span>
</code></pre>
<p data-nodeid="1030">安装完成过后，我们回到配置文件中，添加对应的配置。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="1031"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> MiniCssExtractPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mini-css-extract-plugin'</span>)
<span class="hljs-keyword">const</span> OptimizeCssAssetsWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'optimize-css-assets-webpack-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">mode</span>: <span class="hljs-string">'none'</span>,
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">main</span>: <span class="hljs-string">'./src/index.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span>
  },
  <span class="hljs-attr">module</span>: {
    <span class="hljs-attr">rules</span>: [
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.css$/</span>,
        use: [
          MiniCssExtractPlugin.loader,
          <span class="hljs-string">'css-loader'</span>
        ]
      }
    ]
  },
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> MiniCssExtractPlugin(),
    <span class="hljs-keyword">new</span> OptimizeCssAssetsWebpackPlugin()
  ]
}
</code></pre>
<p data-nodeid="1032">这里同样先导入这个插件，导入完成以后我们把这个插件添加到 plugins 数组中。</p>
<p data-nodeid="1033">那此时我们再次回到命令行运行打包。</p>
<p data-nodeid="1034">打包完成过后，我们的样式文件就会以压缩格式输出了。具体结果如下：</p>
<p data-nodeid="1035"><img src="https://s0.lgstatic.com/i/image/M00/10/E9/Ciqc1F7LaJqAAsfEAAJeBFjqfT8020.png" alt="image (7).png" data-nodeid="1165"></p>
<p data-nodeid="1036">不过这里还有个额外的小点，可能你会在这个插件的官方文档中发现，文档中的这个插件并不是配置在 plugins 数组中的，而是添加到了 optimization 对象中的 minimizer 属性中。具体如下：</p>
<pre class="lang-javascript" data-nodeid="1037"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> MiniCssExtractPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mini-css-extract-plugin'</span>)
<span class="hljs-keyword">const</span> OptimizeCssAssetsWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'optimize-css-assets-webpack-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">mode</span>: <span class="hljs-string">'none'</span>,
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">main</span>: <span class="hljs-string">'./src/index.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span>
  },
  <span class="hljs-attr">optimization</span>: {
    <span class="hljs-attr">minimizer</span>: [
      <span class="hljs-keyword">new</span> OptimizeCssAssetsWebpackPlugin()
    ]
  },
  <span class="hljs-attr">module</span>: {
    <span class="hljs-attr">rules</span>: [
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.css$/</span>,
        use: [
          MiniCssExtractPlugin.loader,
          <span class="hljs-string">'css-loader'</span>
        ]
      }
    ]
  },
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> MiniCssExtractPlugin()
  ]
}
</code></pre>
<p data-nodeid="1038">那这是为什么呢？</p>
<p data-nodeid="1039">其实也很简单，如果我们配置到 plugins 属性中，那么这个插件在任何情况下都会工作。而配置到 minimizer 中，就只会在 minimize 特性开启时才工作。</p>
<p data-nodeid="1040">所以 Webpack 建议像这种压缩插件，应该我们配置到 minimizer 中，便于 minimize 选项的统一控制。</p>
<p data-nodeid="1041">但是这么配置也有个缺点，此时我们再次运行生产模式打包，打包完成后再来看一眼输出的 JS 文件，此时你会发现，原本可以自动压缩的 JS，现在却不能压缩了。具体 JS 的输出结果如下：</p>
<p data-nodeid="1042"><img src="https://s0.lgstatic.com/i/image/M00/10/F5/CgqCHl7LaKeAOdc4AAJbN7rhhA8880.png" alt="image (8).png" data-nodeid="1173"></p>
<p data-nodeid="1043">那这是因为我们设置了 minimizer，Webpack 认为我们需要使用自定义压缩器插件，那内部的 JS 压缩器就会被覆盖掉。我们必须手动再添加回来。</p>
<p data-nodeid="1044">内置的 JS 压缩插件叫作 terser-webpack-plugin，我们回到命令行手动安装一下这个模块。</p>
<pre class="lang-shell" data-nodeid="1045"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> npm i terser-webpack-plugin --save-dev</span>
</code></pre>
<p data-nodeid="1046">安装完成过后，这里我们再手动添加这个模块到 minimizer 配置当中。具体代码如下：</p>
<pre class="lang-javascript" data-nodeid="1047"><code data-language="javascript"><span class="hljs-comment">// ./webpack.config.js</span>
<span class="hljs-keyword">const</span> MiniCssExtractPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'mini-css-extract-plugin'</span>)
<span class="hljs-keyword">const</span> OptimizeCssAssetsWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'optimize-css-assets-webpack-plugin'</span>)
<span class="hljs-keyword">const</span> TerserWebpackPlugin = <span class="hljs-built_in">require</span>(<span class="hljs-string">'terser-webpack-plugin'</span>)
<span class="hljs-built_in">module</span>.exports = {
  <span class="hljs-attr">mode</span>: <span class="hljs-string">'none'</span>,
  <span class="hljs-attr">entry</span>: {
    <span class="hljs-attr">main</span>: <span class="hljs-string">'./src/index.js'</span>
  },
  <span class="hljs-attr">output</span>: {
    <span class="hljs-attr">filename</span>: <span class="hljs-string">'[name].bundle.js'</span>
  },
  <span class="hljs-attr">optimization</span>: {
    <span class="hljs-attr">minimizer</span>: [
      <span class="hljs-keyword">new</span> TerserWebpackPlugin(),
      <span class="hljs-keyword">new</span> OptimizeCssAssetsWebpackPlugin()
    ]
  },
  <span class="hljs-attr">module</span>: {
    <span class="hljs-attr">rules</span>: [
      {
        <span class="hljs-attr">test</span>: <span class="hljs-regexp">/\.css$/</span>,
        use: [
          MiniCssExtractPlugin.loader,
          <span class="hljs-string">'css-loader'</span>
        ]
      }
    ]
  },
  <span class="hljs-attr">plugins</span>: [
    <span class="hljs-keyword">new</span> MiniCssExtractPlugin()
  ]
}
</code></pre>
<p data-nodeid="1048">那这样的话，我们再次以生产模式运行打包，JS 文件和 CSS 文件就都可以正常压缩了。</p>
<h3 data-nodeid="1049">写在最后</h3>
<p data-nodeid="1050">最后再来简单总结一下，本课时我们介绍了如何为 Webpack 添加不同环境下的不同配置，以及在生产模式打包时我们经常用到的几个插件。</p>
<p data-nodeid="1051" class="">这当中需要你理解的地方并没有太多，更多的是了解这些插件的具体作用和使用方法。除此之外，你也需要更多地了解社区当中其他的常用插件。</p>

---

### 精选评论

##### **相：
> 老师，您好。就像你最后说的，多去了解社区中其他常用的插件，我也是每次遇到别人写的不认识的配置才去查官网，后面也容易忘。而且，每次去直接翻阅官网上的配置时，太多了又看不下去，那您一般是如何发现优秀的插件或者配置方式的呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一般我有一个明确的需求过后，我都会根据需求提炼出关键词，然后通过 Google / GitHub 搜索，最后再根据找到模块的使用情况以及更新情况决定是否使用

##### **春：
> 老师，你的git仓库地址是啥。想看看代码

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://github.com/zce/webpack-demo

##### **5547：
> Css文件超过200kb才考虑是否提取出来是指整个项目的css文件还是单个css文件

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 单个 CSS 文件超过 200KB

##### **程：
> 我想问下loader是倒序执行的，那optimization的minimizer也是倒序 执行 吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 每种资源最终只需要使用一个 minimizer，不回涉及到多个压缩器同时压缩同一个资源的情况，也就不会有先后执行的问题

##### gjd：
> 程序是如何识别rc文件的？webpack如何让程序只设置rc文件就可以代替webpack.config.js

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Webpack 的配置只支持 js 文件的方式，至于其他常见工具的配置，很多会支持 rc 类型文件，这些工具内部都是使用的：https://www.npmjs.com/package/cosmiconfig 这个模块解析的配置文件

##### **德：
> webpack5可以用 '...' 把默认的minimizer展开，方便很多了

##### **冰：
> 打包过后，样式就会存放在独立的文件中，直接通过 link 标签引入页面老师，这样有什么好处，相比于内嵌在js中引入？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 主要的原因是样式打包到 js 内部，如果样式代码太多，就会导致单个 JS 文件过大，请求时间过长

##### **冰：
> 为什么要分离出css文件呢，有什么好处？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 主要的原因是样式打包到 js 内部，如果样式代码太多，就会导致单个 JS 文件过大，请求时间过长

##### **龙：
> 老师, 通过 DefinePlugin 定义的变量 跟环境变量有什么区别吗？特别是在接口请求上，测试跟正式接口地址是不一样的，因而通过环境变量进行处理，但是都是使用 process.env.*** 来进行获取， 而通过 DefinePlugin 是直接使用变量来获取。这块应该如何处理较好？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; DefinePlugin 是往我们的代码中注入可能发生变化的成员，跟 process.env 没有关系，我们编写的业务代码并不是运行在 Node 下的

