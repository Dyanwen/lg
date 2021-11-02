<p data-nodeid="5712">你好，我是俊鹏。从今天开始，我会用四节课的时间带你学习微信小程序在工程化方面的实践方案。</p>
<p data-nodeid="5713">小程序发展到今天已经成为很多产品的重要流量入口，随着用户量的增加，功能不断复杂化，小程序的体量不断增长，原始的“手工作坊式”研发模式就很难跟得上源源不断的业务需求，必然会走向工程化的道路，这就是我把工程化相关的课程单独划分为一个模块的原因。</p>
<p class="te-preview-highlight" data-nodeid="6452">今天这节课我会带你对比官方的微信 IDE 存在哪些不足，以及如何使用目前最流行的构建工具 Webpack 去解决这些问题。引入构建工具能够在很大程度上提高小程序的研发效率，当然了，我们并不是要学习怎么编写 Webpack 的配置代码，而是要了解需要用 Webpack 做哪些事情，因为一旦明确了目标，剩下的就是查文档、编写配置文件这种小事了。</p>


<p data-nodeid="5715">其实，微信 IDE 提供了一些非常有用的工具（比如 04 讲提到的小程序体验评分功能），这些工具可以辅助开发者写出更好的代码。而且微信 IDE 的迭代速度非常快，官方团队在不断完善 IDE 的功能，提供给开发者更好的研发体验。但目前版本在完整度方面仍有欠缺，开发者单纯依赖微信 IDE 本身还难以打造较“完整”的研发模式（你要注意，这里的“完整”是与前端开发领域作对比的，前端的工具生态非常完善，并且有很高的自由度供开发者打造适合自身的研发模式）。</p>
<p data-nodeid="5716">而我们改善研发体模式的目标自然是为了解决业务需求，但现实中业务类型的差异对研发模式的具体形态有一定影响，我们在课程中没办法覆盖所有类型的业务，所以只挑选了几个通用性较高、也是需求度最高的关键环节来讲：</p>
<ul data-nodeid="5717">
<li data-nodeid="5718">
<p data-nodeid="5719">管理第三方 npm 模块</p>
</li>
<li data-nodeid="5720">
<p data-nodeid="5721">使用 TypeScript 编写源码</p>
</li>
<li data-nodeid="5722">
<p data-nodeid="5723">使用预处理器编写模块化的样式</p>
</li>
<li data-nodeid="5724">
<p data-nodeid="5725">使用 lint 工具统一源码规范</p>
</li>
<li data-nodeid="5726">
<p data-nodeid="5727">图片压缩</p>
</li>
<li data-nodeid="5728">
<p data-nodeid="5729">多环境支持</p>
</li>
</ul>
<h4 data-nodeid="5730">管理第三方 npm 模块</h4>
<p data-nodeid="5731">微信小程序的早期版本不支持使用第三方 npm 包，在基础库 2.2.1 版本才开始支持。但是微信小程序使用 npm 模块的方式与 Node.js 的并不完全相同，虽然同样可以用包管理工具（npm/yarn）安装 npm 模块，但是在小程序源码中引入（require）npm 模块的路径并不是node_modules，而是 miniprogram_npm 目录。</p>
<p data-nodeid="5732">开发者在使用 npm 模块之前必须使用微信 IDE 菜单栏中的“工具”-“构建 npm”，将原始的 npm 模块（即node_modules 目录中的模块）进行一次预构建，预构建的产出目录便是 miniprogram_npm ，最后才可以在小程序源码中引入（流程如下图所示）：</p>
<p data-nodeid="5733"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/6D/91/Ciqc1F-uPmSAHDYTAABv_sOtC6E405.png" data-nodeid="5836"></p>
<p data-nodeid="5734">而且，小程序预构建 npm 模块的过程并不是简单地将原始模块从拷贝 node_modules 目录到miniprogram_npm 目录，而是会将原始模块的所有散列文件打包成一个单 js 文件，然后再将这个 js 文件作为模块入口暴露出去，整个预编译的流程如下：</p>
<ol data-nodeid="5735">
<li data-nodeid="5736">
<p data-nodeid="5737">读取小程序项目的 package.json 文件（位于 miniprogram/package.json）中有哪些依赖（dependencies）。</p>
</li>
<li data-nodeid="5738">
<p data-nodeid="5739">在 node_modules 目录内依次寻找这些依赖的原始 npm 模块，读取模块的 package.json文件，搜寻 main 字段指定的入口 js 文件。</p>
</li>
<li data-nodeid="5740">
<p data-nodeid="5741">分析模块的入口 js 文件引用了哪些子文件。</p>
</li>
<li data-nodeid="5742">
<p data-nodeid="5743">将所有文件打包为一个单 js 文件（如下图）。</p>
</li>
</ol>
<p data-nodeid="5744"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/6D/91/Ciqc1F-uPmyARZYuAAB84i1b-ro900.png" data-nodeid="5850"></p>
<p data-nodeid="5745"><strong data-nodeid="5855">你要注意，在执行第四步时，微信 IDE 并不会将原始 npm 模块所依赖的其他 npm 模块一并打包。</strong> 比如，现实工作中的网络请求模块 axios ，这个基础模块可能被某个 npm 模块（假设为模块 A）依赖，如下：</p>
<pre class="lang-java" data-nodeid="5746"><code data-language="java"><span class="hljs-keyword">import</span> axios from <span class="hljs-string">'axios'</span>;
</code></pre>
<p data-nodeid="5747">那么微信 IDE 在编译 A 的时候并不会将 axios 的代码一起打包为单 js 文件，而是会保留代码中对于 axios 模块的引用。同时会根据依赖关系寻找 axios 模块的 package.json 文件，然后执行上述的 2~3 步骤，也就是把 axios 模块也编译为单 js 文件。</p>
<p data-nodeid="5748">最终的效果就是 miniprogram_npm 目录中存在模块 A 和模块 axios 两个子目录。<strong data-nodeid="5866">这样就存在一个很严重的问题：</strong> 通常我们的代码中只使用了第三方 npm 模块的一个或几个 API 而不是全部，微信 IDE 方式却始终会把 npm 模块内的全部代码进行打包，最终造成的后果是代码体积增大。</p>
<p data-nodeid="5749">又因为小程序对于代码体积有严格的限制（目前是 2M），使用微信 IDE 打包后很可能会超过上限。虽然在前端开发领域内，一些构建工具（比如 Webpack）会通过 Tree Shaking 机制在打包过程中将没有用到的代码片段舍弃，减少打包后的文件体积，但微信 IDE 目前却并没有这种特性。</p>
<p data-nodeid="5750">那么对于习惯了标准 npm 使用方式的前端开发者来说，微信小程序这种 npm 模块的管理和打包方案是很难接受的，单纯从研发效率的角度出发，这个方案也几乎没有可取之处。</p>
<p data-nodeid="5751">所以，业内普遍的做法就是：<strong data-nodeid="5874">放弃微信 IDE 的 npm 管理方案，使用前端构建工具打造一套构建体系</strong>。</p>
<p data-nodeid="5752">一个未经修改的微信小程序源码目录如下图所示：</p>
<p data-nodeid="5753"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/6D/9C/CgqCHl-uPn-AKQnbAAA5YigCdH0607.png" data-nodeid="5878"></p>
<p data-nodeid="5754">其中 cloudfunctions 是云函数的根目录（相关的知识会在模块四讲解，在这里我就不多费口舌了）。miniprogram 中的文件是小程序本体的源码，包括小程序的业务代码和 npm 模块。</p>
<p data-nodeid="5755">使用 Webpack 打造的构建体系通常会<strong data-nodeid="5884">另外建立一个与 cloudfunctions 和 miniprogram 平行的目录用于管理源码，然后将 miniprogram 目录作为构建产出目录，如下：</strong></p>
<p data-nodeid="5756"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/6D/91/Ciqc1F-uPoaAWgOGAAAulRAEVKU221.png" data-nodeid="5887"></p>
<p data-nodeid="5757">同时禁用微信 IDE 编译相关的功能，把这些工作全部交给 Webpack：</p>
<p data-nodeid="5758"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/6D/91/Ciqc1F-uPoyADaa-AAA91X7XW4I841.png" data-nodeid="5891"></p>
<p data-nodeid="5759">这样一来，在自建的构建体系下，我们不仅可以使用标准的 npm 模块管理方式，同时可以发挥 Webpack 对于研发效率的加持，比如 Tree-Shaking 减小打包文件体积、结合 Babel 使用最新的 ECMAScript 特性、结合 Lint 工具统一代码规范等。这是接下来我们要讨论使用 Webpack 完成的几项具体工作的基础。</p>
<h4 data-nodeid="5760">使用 TypeScript 编写源码</h4>
<p data-nodeid="5761">首先是我个人非常喜欢的一个“工具”：TypeScript。这里之所以加引号是因为业内对于 TypeScript 的定位和理解并没有绝对的统一。有的人认为它是一种工具，有的人认为它是一种语言。当然，不论哪种理解，都不影响对 TypeScript 的使用。</p>
<p data-nodeid="5762">目前 TypeScript 在前端领域内的普及程度丝毫不亚于 Vue/React 这类流行框架，微信小程序官方也支持 TypeScript ，每次修改源码后需要调用 TypeScript 编译器 tsc 将源码编译为 JavaScript 原生代码，这项工作同样可以集成到 Webpack 构建体系中，不过需要一定的配置工作。</p>
<p data-nodeid="5763">微信小程序的开发框架会暴露一些 JavaScript 对象供开发者使用，比如小程序的各种构造器（App/Page/Component），这些对象并不是 JavaScript 语言的一部分，所以TypeScript 无法识别这些语法进而会抛出编译错误。<strong data-nodeid="5901">解决这个编译错误最简单粗暴的办法是：</strong> 调用小程序相关接口的代码行加上 @ts-ginore 注释语句，相当于告诉 TypeScript 编译器：这些代码不参与编译。</p>
<p data-nodeid="5764">不过如果使用这种办法，同时也就舍弃了小程序接口的相关语法提示，对于研发效率来说是有负面影响的。更优雅的解决方案是引入类似 @types/node 这种接口定义模块，微信小程序官方提供了<a href="https://github.com/wechat-miniprogram/api-typings" data-nodeid="5905">小程序框架接口的定义模块</a>，将其作为 devDependency 引入支持源码编写阶段的语法提示，有了这些 TypeScript 类型声明，在编译的时候就能够令 TypeScript 编译器“认识”小程序专有的语法，进而不会抛出错误。</p>
<h4 data-nodeid="5765">使用预处理器编写模块化的样式</h4>
<p data-nodeid="5766">与 WXML 不同的是，WXSS 与 CSS 几乎完全一致。WXML 是语法类似 HTML 的一种标记语言（Markup Language），虽然最终渲染仍旧是还原为 HTML ，但在编写源码阶段必须依照WXML 的语法进行。而 WXSS 则不同，它与 CSS 唯一的差别就是小程序独有的像素单位 rpx，除此之外所有的语法都与 CSS 一致。WXSS 在编写代码的方式上也并没有任何创新，使用的是最原始的 CSS 编码模式，而这种模式早在 5 年前就已经被淘汰了。</p>
<p data-nodeid="5767">如今，前端开发者极少会直接编写 CSS 代码，通常会借助预编译（Less/Scss）或后编译（PostCSS）工具提升 CSS 的研发效率。这些工具对于 CSS 编码的改善主要从以下几个方面入手：</p>
<ul data-nodeid="5768">
<li data-nodeid="5769">
<p data-nodeid="5770"><strong data-nodeid="5913">嵌套</strong></p>
</li>
</ul>
<p data-nodeid="5771">嵌套的优势是能够帮助开发者以类似组件的方式编写代码，令父节点和子节点的关系更明确，这样可以提升代码的维护效率。对于同一个父节点的元素可以通过嵌套的方式编写 CSS 选择器，比如 class 为 father 的节点可以通过以下语法（Less）为其子节点定义样式：</p>
<pre class="lang-java" data-nodeid="5772"><code data-language="java">.father{
&nbsp; background-color: #999;
&nbsp; .child_1{
&nbsp; &nbsp; color: #ddd;
&nbsp; }
&nbsp; .child_2{
&nbsp; &nbsp; color: #fff;
&nbsp; }
}
</code></pre>
<p data-nodeid="5773">而如果使用原生 CSS 则必须分别为每个子节点添加样式，如下：</p>
<pre class="lang-java" data-nodeid="5774"><code data-language="java">.father{
&nbsp; background-color: #999;
}
.father .child_1{
&nbsp; color: #ddd;
}
.father .child_2{
&nbsp; color: #fff;
}
</code></pre>
<p data-nodeid="5775">既不方便编写也不利于维护。</p>
<ul data-nodeid="5776">
<li data-nodeid="5777">
<p data-nodeid="5778"><strong data-nodeid="5920">继承</strong></p>
</li>
</ul>
<p data-nodeid="5779">继承指的是同一套样式规则可以通过简单的继承语法被应用到多个样式规则中，大多数预编译工具都支持样式规则之间的继承，比如 Sass 可以通过以下方式实现继承：</p>
<pre class="lang-java" data-nodeid="5780"><code data-language="java">.classA {
&nbsp; backgroud-color: #999;
}
.classB{
&nbsp; @extend .classA;
&nbsp; color: #fff;
}
</code></pre>
<p data-nodeid="5781">最终编译为 CSS 为：</p>
<pre class="lang-java" data-nodeid="5782"><code data-language="java">.classA .classB{
&nbsp; backgroud-color: #999;
}
.classB{
&nbsp; color: #fff;
}
</code></pre>
<ul data-nodeid="5783">
<li data-nodeid="5784">
<p data-nodeid="5785"><strong data-nodeid="5926">模块化</strong></p>
</li>
</ul>
<p data-nodeid="5786">CSS 不支持模块化（虽然有个古老的 import 机制，但是它并不是一种模块化机制，而是用于引入第三方样式文件。非常难用，如果你感兴趣可以课后搜索相关资料），CSS 代码的管理和复用是一件非常耗时耗力的事情，对于历史悠久的老项目而言更加困难。</p>
<p data-nodeid="5787">CSS 预编译和后编译工具可以提供类似 ES 6 的静态模块体系，将相对独立的样式抽离为单独的模块，通过 import 导入，然后编译为最终的 CSS 代码。</p>
<p data-nodeid="5788">除了以上三点以外 ，CSS 预编译或后编译工具对于研发效率的提升还在其他正面的影响，比如处理 CSS 的兼容性，<strong data-nodeid="5933">不过由于小程序中不存在这类问题，所以最主要的还是嵌套、继承的模块化。</strong></p>
<h4 data-nodeid="5789">使用 lint 工具统一编码规范</h4>
<p data-nodeid="5790">统一的编码规范是一个团队必备的因素，如果你是个人开发者可以在写代码的时候天马行空，但是在一个多人团队中则必须严格遵守统一的规范。而且，一个研发团队从小作坊迈向工程化不仅仅需要技术维度的考量，同样也需要形成严格的、一致的规范，编码规范便是其中之一。</p>
<p data-nodeid="5791"><img alt="小程序 05--金句.png" src="https://s0.lgstatic.com/i/image/M00/6E/4E/Ciqc1F-yL7KAJgnbAAE8MX8pSck595.png" data-nodeid="5938"></p>
<p data-nodeid="5792">在既定的技术栈前提下（比如你的团队选定了使用 TypeScript、Less 开发小程序），下一步就是针对这些技术栈制定代码规范审查流程，首先是选定合适的工具，比如 ESLint、StyleLint 等；然后需要制定规范审查的触发时机，最简单的方法是有人为触发，在提交之前手动执行 lint命令。</p>
<p data-nodeid="5793">但是人为触发的方式很容易被疏忽，更好的方案是借助一些工具，比如为 Git 的 Commit 行为添加 Hook 触发 lint 命令，这样能够保证每次提交都会执行一次代码审查。</p>
<h4 data-nodeid="5794">图片压缩</h4>
<p data-nodeid="5795">虽然我并不建议在小程序的源码中存放图片资源（因为图片体积太大，很容易超过小程序的代码体积上限），但是如果在必要的场景下仍然难免会使用，比如预加载期间的 loading 图标通常是直接存放在小程序的代码目录中。</p>
<p data-nodeid="5796">对于这类图片的体积控制一定要非常谨慎，在保证图片质量的前提下应该尽量减小体积。通过微信 IDE 上传的代码包中如果存在图片，微信并不会将其压缩，这个工作可以交给 Webpack 完成。在构建阶段，Webpack 将图片进行合理的压缩，对于体积太大的图片还可以发出告警，尽量保证了图片体积的可控性。</p>
<h4 data-nodeid="5797">多环境支持</h4>
<p data-nodeid="5798">通常我们在写代码的时候至少存在两种环境：开发环境和生产环境。在开发环境中编写项目时会有一些为了提高调试效率而存在的功能，比如只打包不混淆、通过 SourceMap 直接定位到源码行等等。</p>
<p data-nodeid="5799">而生产环境中会将这些功能关闭，只关注于代码逻辑本身。对于两种环境区分的一个正面例子是React，在开发环境中如果使用了一些影响性能或者未来将被淘汰的 API ，React 会在控制台打印必要的提示信息，这些信息通常是以 Erorr 抛出；而在生产环境中，React 会隐藏这些提示，或者以 Warning 的形式给出，这样不会影响代码逻辑的正常执行。</p>
<p data-nodeid="5800">Webpack 的 v4 版本提供了一键配置（mode）切换开发和生产环境，这是自建的构建体系相对于微信 IDE 的另一个效能提升点。你可以在调用 Webpack 进行构建时通过命令行的方式传入环境类型（比如借助process.env.NODE_ENV），然后根据环境类型分配对应的 Webpack 配置文件，实现针对不同环境的构建需求。</p>
<h4 data-nodeid="5801">总结</h4>
<p data-nodeid="5802">微信IDE提供了一些便利功能，但还不足以形成完整的研发体系，尤其是对外部资源（npm模块）的管理方面。</p>
<p data-nodeid="5803">小程序与前端的技术栈非常相近，在工具生态方面也有很多相似之处，这部分可通用的工具可以帮助开发者将前端领域内对于研发效率的提升经验“拿”到小程序开发领域中。</p>
<p data-nodeid="5804">Webpack 作为目前功能最强大的前端工具之一，可以解决小程序开发在工程化方面的一些问题，比如 npm 模块的管理、语法编译、多环境支持等等。今天这节课我想强调这样几个重点：</p>
<ol data-nodeid="5805">
<li data-nodeid="5806">
<p data-nodeid="5807">微信 IDE 对npm模块的打包方式会造成代码体积增大，这也是我们要借助 Webpack 这样的构建工具改善研发模式的主要原因之一；</p>
</li>
<li data-nodeid="5808">
<p data-nodeid="5809">Webpack背后是前端领域庞大的工程生态，不论是针对代码编写（脚本/样式）、资源管理（图片）还是多环境支持，都有很完善的工程化工具，这些工具能够帮助你更好、更快地开发小程序。</p>
</li>
</ol>
<p data-nodeid="5810">总的来说，我希望你在学习完这节课之后，能够明确一款成熟小程序的研发模式从小作坊迈向工程化是必然的，然后通过我分享的内容能够了解搭建小程序工程化研发模式会哪些功能需求和解决思路。其实我并没有花费很多篇幅介绍如何编写 Webpack 的配置，而是将主要精力投入到问题和解决思路上面，这些理论层面的思想是与工具无关的，可以应用到任何工具上面。所以即便你不习惯使用 Webpack，更倾向于其他工具（比如 Gulp 、Rollup 等），按照本节课的指导也能够清晰要做哪些事情，剩下的就是查文档、改配置这种小事了。</p>
<p data-nodeid="5811">那么今天的课后作业便是：选择一款你喜欢的构建工具（Gulp/Webpack/Rollup），实现本节课提到的几个功能点。</p>

---

### 精选评论

##### **强：
> 使用 webpack 构建，目前可以支持分包等形式嘛，还有如果使用 wxs 事件响应等一系列原生能力。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以，webpack跟小程序的分包其实没什么关系，webpack的工作是支持更友好的编码方式，可以简单理解为“预编译”，编译之后产出的代码对于小程序来说仍旧是源码，真正执行从源码到可运行代码的编译是小程序本身的IDE完成。

##### **用户8046：
> WXSS和CSS差异最大的就是选择器的支持

##### *超：
> 几个功能点包括，images压缩，npm模块打包优化，使用如less 的预编译方式，使用 lint统一开发代码规范，多环境配置支持。

##### **晓：
> 请问后面会带着搭一套吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 课程最后的商城小程序会使用到webpack

##### *会：
> 05讲打卡😁

##### **锋：
> 同时禁用微信 IDE 编译相关的功能，把这些工作全部交给 Webpack？这个有没有相关详细介绍！！！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 就是禁用微信IDE的ES6转ES5、增强编译、压缩混淆扥功能，把这些工作交给webpack。

##### *森：
> 提个问题 wepack好像解决不了动态入口的问题吧 在现阶段处理小程序的插件和loader好像没几个 用webpack去处理小程序的话 wxs和wxml里面的import还得自己写loader去处理 json里面引用的组件的依赖关系也得自己写插件或者loader处理

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 目前webpack生态中有相关的loader并且比较稳定，而且小程序框架并不是完全封闭的，举个例子：wxss其实跟sass的语法和机制很接近，使用sass-loader+css-loader就可以。

