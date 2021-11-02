<p>上一课时我们从技术细节的角度分析了 CSS 布局的相关内容。这一课时我们提升一下思考维度，从组织管理的角度探讨如何管理好项目中的 CSS 代码。</p>
<p>接下来我们先解决 CSS 原生语法未能很好实现的模块化和作用域的问题，然后再对代码结构进行优化，提升代码的复用率。</p>
<h3>如何组织样式文件</h3>
<p>尽管 CSS 提供了 import 命令支持文件引用，但由于其存在一些问题（比如影响浏览器并行下载、加载顺序错乱等）导致使用率极低。更常见的做法是通过预处理器或编译工具插件来引入样式文件，因此本课时的讨论将不局限于以 .css 为后缀的样式文件。</p>
<p><strong>管理样式文件的目的就是为了让开发人员更方便地维护代码。</strong></p>
<p>具体来说就是将样式文件进行分类，把相关的文件放在一起。让工程师在修改样式的时候更容易找到对应的样式文件，在创建样式文件的时候更容易找到对应的目录。</p>
<p>下面我们来看看热门的开源项目都是怎么来管理样式文件的。</p>
<h4>开源项目中的样式文件</h4>
<p>我们先来看看著名的 UI 相关的开源项目是怎么管理样式文件的。</p>
<p>以 <a href="https://github.com/twbs/bootstrap">Bootstrap 4.4</a> 为例，下图是项目样式代码结构，可以看出项目使用的是 Sass 预处理器。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/12/Ciqc1F7Lg2KAL_EGAABIIQsIyiQ803.png" alt="image (9).png"></p>
<p>该目录包括了 5 个目录、组件样式文件和一些全局样式。再来分析下目录及内容：</p>
<ul>
<li>forms/，表单组件相关样式；</li>
<li>helpers/，公共样式，包括定位、清除等；</li>
<li>mixins/，可以理解为生成最终样式的函数；</li>
<li>utilities/，媒体查询相关样式；</li>
<li>vendor/，依赖的外部第三方样式。</li>
</ul>
<p>根目录存放了组件样式文件和目录，其他样式文件放在不同的目录中。目录中的文件分类清晰，但目录结构相对于大多数实际项目而言过于简单（只有样式文件）。</p>
<p>我们再来看一个更符合大多数情况的开源项目 <a href="https://github.com/ant-design/ant-design/">ant-design 4.2</a>，该项目采用 Less 预处理器，主要源码放在 /components 目录下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/1D/CgqCHl7Lg2qAA71pAAA-LE1MpA8895.png" alt="image (10).png"></p>
<p>从目录名称上不难猜测，各个组件代码通过文件夹区分，点击其中的 alert 文件夹查看也确实如此，组件相关的代码、测试代码、demo 示例、样式文件、描述文档都在里面。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/12/Ciqc1F7Lg3KACdLvAAAhZ58r7rs506.png" alt="image (11).png"></p>
<p>至于全局样式和公共样式则在 /components/style 目录下：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/1D/CgqCHl7Lg3iAW5XnAABJl9DhW5U953.png" alt="image (12).png"></p>
<p>其中包括 4 个目录：</p>
<ul>
<li>color/，颜色相关的变量与函数；</li>
<li>core/，全局样式，根标签样式、字体样式等；</li>
<li>mixins/，样式生成函数；</li>
<li>themes/，主题相关的样式变量。</li>
</ul>
<p>将组件代码及相关样式放在一起，开发的时候修改会很方便。 但在组件目录 /comnponents 下设置 style 目录存放全局和公共样式，在逻辑上就有些说不通了，这些“样式”文件并不是一个单独的“组件”。再看 style 目录内部结构，相对于设置单独的 color 目录来管理样式中的颜色，更推荐像 Bootstrap 一样设立专门的目录或文件来管理变量。</p>
<p>最后来看看依赖 Vue.js 实现的热门 UI 库 <a href="https://github.com/ElemeFE/element">element 2.13.1</a> 的目录结构。项目根路径下的 packages 目录按组件划分目录来存放其源码，但和 ant-design 不同的是，组件样式文件并没有和组件代码放在一起。下图是 /packages 目录下的部分内容。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/1D/CgqCHl7Lg4GALTUpAAA1dlUb1Vo822.png" alt="image (13).png"></p>
<p>element 将样式文件统一放入了 /packages/theme-chalk 目录下，目录部分内容如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/12/Ciqc1F7Lg4iAEEjuAABKZlcUAZw345.png" alt="image (14).png"></p>
<p>其中包含 4 个目录：</p>
<ul>
<li>common/，一些全局样式和公共变量；</li>
<li>date-picker/，日期组件相关样式；</li>
<li>fonts/，字体文件；</li>
<li>mixins/，样式生成函数及相关变量。</li>
</ul>
<p>和 antd 有同样的问题，把样式当成“组件”看待，组件同级目录设立了 theme-chalk 目录存放样式文件。theme-chalk 目录下的全局样式 reset.scss 与组件样式同级，这也有些欠妥。这种为了将样式打包成模块，在独立项目中直接嵌入另一个独立项目（可以简单理解为一个项目不要有多个 package.json 文件）并不推荐，更符合 Git 使用规范的做法，即是以<a href="https://git-scm.com/book/zh/v2/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97">子模块</a>的方式引用进项目。 而且将组件样式和源码分离这种方式开发的时候也不方便，经常需要跨多层目录查找和修改样式。</p>
<h4>样式文件管理模式</h4>
<p>除了开源项目之外，<a href="https://sass-guidelin.es/#architecture">Sass Guidelines</a> 曾经提出过一个用来划分样式文件目录结构的 7-1 模式也很有参考意义。这种模式建议将目录结构划分为 7 个目录和 1 个文件，这 1 个文件是样式的入口文件，它会将项目所用到的所有样式都引入进来，一般命名为 main.scss。</p>
<p>剩下的 7 个目录及作用如下：</p>
<ul>
<li>base/，模板代码，比如默认标签样式重置；</li>
<li>components/，组件相关样式；</li>
<li>layout/，布局相关，包括头部、尾部、导航栏、侧边栏等；</li>
<li>pages/，页面相关样式；</li>
<li>themes/，主题样式，即使有的项目没有多个主题，也可以进行预留；</li>
<li>abstracts/，其他样式文件生成的依赖函数及 mixin，不能直接生成 css 样式；</li>
<li>vendors/，第三方样式文件。</li>
</ul>
<p><a href="https://github.com/HugoGiraudel/sass-boilerplate">点击这里获取示例项目地址</a>，截图如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/12/Ciqc1F7Lg5CAFS5jAAB-ZPy2xPc135.png" alt="image (15).png"></p>
<p>由于这个划分模式是专门针对使用 Sass 项目提出的，从样式文件名称看出还留有 jQuery 时代的影子，为了更加符合单页应用的项目结构，我们可以稍作优化。</p>
<ul>
<li>main.scss 文件存在意义不大，页面样式、组件样式、布局样式都可以在页面和组件中引用，全局样式也可以在根组件中引用。而且每次添加、修改样式文件都需要在 main.scss 文件中同步，这种过度中心化的配置方式也不方便。</li>
<li>layout 目录也可以去除，因为像 footer、header 这些布局相关的样式，放入对应的组件中来引用会更好，至于不能被组件化的“_grid”样式存在性也不大。因为对于页面布局，既可以通过下面介绍的方法来拆分成全局样式，也可以依赖第三方 UI 库来实现。所以说这个目录可以去除。</li>
<li>themes/ 目录也可以去除，毕竟大部分前端项目是不需要设置主题的，即使有主题也可以新建一个样式文件来管理样式变量。</li>
<li>vendors/ 目录可以根据需求添加。因为将外部样式复制到项目中的情况比较少，更多的是通过 npm 来安装引入 UI 库或者通过 webpack 插件来写入对应的 cdn 地址。</li>
</ul>
<p>所以优化后的目录结构如下所示：</p>
<pre><code data-language="java" class="lang-java">src/
|
|– abstracts/
|&nbsp; &nbsp;|– _variables.scss&nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;|– _functions.scss&nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;|– _mixins.scss&nbsp; &nbsp; &nbsp; &nbsp;
|&nbsp; &nbsp;|– _placeholders.scss&nbsp;
|
|– base/
|&nbsp; &nbsp;|– _reset.scss&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;|– _typography.scss&nbsp; &nbsp;
|&nbsp; &nbsp;…&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
|
|– components/
|&nbsp; &nbsp;|– _buttons.scss&nbsp; &nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;|– _carousel.scss&nbsp; &nbsp; &nbsp;
|&nbsp; &nbsp;|– _cover.scss&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;|– _dropdown.scss&nbsp; &nbsp; &nbsp;
|&nbsp; &nbsp;|- header/
|&nbsp; &nbsp; &nbsp; |- header.tsx
|&nbsp; &nbsp; &nbsp; |- header.sass
|&nbsp; &nbsp;|- footer/
|&nbsp; &nbsp; &nbsp; |- footer.tsx
|&nbsp; &nbsp; &nbsp; |- footer.sass
|&nbsp; &nbsp;…&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
|
|– pages/
|&nbsp; &nbsp;|– _home.scss&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
|&nbsp; &nbsp;|– _contact.scss&nbsp; &nbsp; &nbsp;&nbsp;
|&nbsp; &nbsp;…&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
|
</code></pre>
<p>这只是推荐的一种目录结构，具体使用可以根据实际情况进行调整。比如我在项目的 src 目录下创建了模块目录，按照模块来拆分路由以及页面、组件，所以每个模块目录下都会有 pages/ 目录和 components/ 目录。</p>
<h3>如何避免样式冲突</h3>
<p>由于 CSS 的规则是全局的，任何一个样式规则，都对整个页面有效，所以如果不对选择器的命名加以管控会很容易产生冲突。</p>
<h4>手动命名</h4>
<p>最简单有效的命名管理方式就是制定一些命名规则，比如 <a href="http://oocss.org/">OOCSS</a>、<a href="http://getbem.com/">BEM</a>、<a href="https://amcss.github.io/">AMCSS</a>，其中推荐比较常用的 BEM。</p>
<p>这里简单补充一下 BEM 相关知识，熟悉 BEM 的可以直接跳过。</p>
<p>BEM 是 Block、Element、Modifier 三个单词的缩写，Block 代表独立的功能组件，Element 代表功能组件的一个组成部分，Modifier 对应状态信息。</p>
<p>下图是官方给出的示例代码：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/11/1E/CgqCHl7Lg5qAeYYDAAA8eZ2PEKM297.png" alt="image (16).png"></p>
<p>从命名可以看到 Element 和 Modifier 是可选的，各个单词通过双横线（也可以用双下划线）连接（双横线虽然能和单词的连字符进行区分，但确实有些冗余，可以考虑直接用下划线代替）。BEM 的命名方式具有语义，很容易理解，非常适用于组件样式类。</p>
<h4>工具命名</h4>
<p>通过命名规范来避免冲突的方式固然是好的，但这种规范约束也不能绝对保证样式名的唯一性，而且也没有有效的校验工具来保证命名正确无冲突。所以，聪明的开发者想到了通过插件将原命名转化成不重复的随机命名，从根本上避免命名冲突。比较著名的解决方案就是 CSS Modules。</p>
<p>下面是一段 css 样式代码：</p>
<pre><code data-language="css" class="lang-css"><span class="hljs-comment">/* style.css */</span>
<span class="hljs-selector-class">.className</span> {
  <span class="hljs-attribute">color</span>: green;
}
</code></pre>
<p>借助 css Modules 插件，可以将 css 以 JSON 对象的形式引用和使用。</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-keyword">import</span> styles <span class="hljs-keyword">from</span> <span class="hljs-string">"./style.css"</span>;
<span class="hljs-comment">// import { className } from "./style.css";</span>
element.innerHTML = <span class="hljs-string">'&lt;div class="'</span> + styles.className + <span class="hljs-string">'"&gt;'</span>;
</code></pre>
<p>编译之后的代码，样式类名被转化成了随机名称：</p>
<pre><code data-language="xml" class="lang-xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"_3zyde4l1yATCOkgn-DBWEL"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">style</span>&gt;</span><span class="css">
<span class="hljs-selector-class">._3zyde4l1yATCOkgn-DBWEL</span> {
  <span class="hljs-attribute">color</span>: green;
}
</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span>
</code></pre>
<p>但这种命名方式带来了一个问题，那就是如果想在引用组件的同时，覆盖它的样式会变得困难，因为编译后的样式名是随机。例如，在上面的示例代码中，如果想在另一个组件中覆盖 className 样式就很困难，而在手动命名情况下则可以直接重新定义 className 样式进行覆盖。</p>
<h3>如何高效复用样式</h3>
<p>如果你有一些项目开发经历，一定发现了某些样式会经常被重复使用，比如：</p>
<ul>
<li>display:inline-block</li>
<li>clear:both</li>
<li>position:relative</li>
<li>......</li>
</ul>
<p>这违背了 DRY（Don't Repeat Yourself）原则，完全可以通过设置为全局公共样式来减少重复定义。</p>
<p>哪些样式规则可以设置为全局公共样式呢？</p>
<ul>
<li>首先是具有枚举值的属性，除了上面提到的，还包括 cursor:pointer、float:left 等。</li>
<li>其次是那些特定数值的样式属性值，比如 margin: 0、left: 0、height: 100%。</li>
<li>最后是设计规范所使用的属性，比如设计稿中规定的几种颜色。</li>
</ul>
<p>样式按照小粒度拆分之后命名规范也很重要，合理的命名规范可以避免公共样式重复定义，开发时方便快速引用。</p>
<p>前面提到的语义化命名方式 BEM 显然不太适合。首先全局样式是基于样式属性和值的，是无语义的；其次对于这种复用率很高的样式应该尽量保证命名简短方便记忆，所以推荐使用更简短、更方便记忆的命名规则。比如我们团队所使用的就是“属性名首字母 + 横线 + 属性值首字母”的方式进行命名。</p>
<p>举个例子，比如对于 display:inline-block 的样式属性值，它的属性为“display”缩写为“d”，值为“inline-block”，缩写为“ib”，通过短横线连接起来就可以命名成“d-ib”；同样，如果工程师想设置一个 float:left 的样式，也很容易想到使用“f-l”的样式名。</p>
<p>那会不会出现重复定义呢？这个问题很好解决，按照字母序升序定义样式类就可以了。</p>
<h3>延伸：值得关注的 CSS in JavaScript</h3>
<p>我们都知道 Web 标准提倡结构、样式、行为分离（分别对应 HTML、CSS、JavaScript 三种语言），但 React.js 的一出现就开始颠覆了这个原则。</p>
<p>先是通过 JSX 将 HTML 代码嵌入进 JavaScript 组件，然后又通过 CSS in JavaScript 的方式将 CSS 代码也嵌入进 JavaScript 组件。这种“all in JavaScript”的方式确实有悖 Web 标准。但这种编写方式和日益盛行的组件化概念非常契合，具有“高内聚”的特性，所以未来标准有所改变也未尝不可能。这也正是我们需要关注 CSS in JavaScript 技术的原因。</p>
<p>相对于使用预处理语言编写样式，CSS in JavaScript 具有两个不那么明显的优势：</p>
<ul>
<li>可以通过随机命名解决作用域问题，但命名规则和 CSS Modules 都可以解决这个问题；</li>
<li>样式可以使用 JavaScript 语言特性，比如函数、循环，实现元素不同的样式效果可以通过新建不同样式类，修改元素样式类来实现。</li>
</ul>
<p>我们以 <a href="https://styled-components.com/">styled-compoents</a> 为例进行说明，下面是示例代码，第一段是源代码：</p>
<pre><code data-language="javascript" class="lang-javascript"><span class="hljs-comment">// 源代码</span>
<span class="hljs-keyword">const</span> Button = styled.button<span class="hljs-string">`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0.5em 1em;
  padding: 0.25em 1em;
  <span class="hljs-subst">${props =&gt; props.primary &amp;&amp; css`<span class="css">
    <span class="hljs-selector-tag">background</span>: <span class="hljs-selector-tag">palevioletred</span>;
    <span class="hljs-selector-tag">color</span>: <span class="hljs-selector-tag">white</span>;
  `</span>}</span>
`</span>;
<span class="hljs-keyword">const</span> Container = styled.div<span class="hljs-string">`
  text-align: center;
`</span>
render(
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">Container</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">Button</span>&gt;</span>Normal Button<span class="hljs-tag">&lt;/<span class="hljs-name">Button</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">Button</span> <span class="hljs-attr">primary</span>&gt;</span>Primary Button<span class="hljs-tag">&lt;/<span class="hljs-name">Button</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">Container</span>&gt;</span></span>
);
</code></pre>
<p>第二段是编译后生成的：</p>
<pre><code data-language="xml" class="lang-xml"><span class="hljs-comment">&lt;!--HTML 代码--&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"sc-fzXfNJ ciXJHl"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"sc-fzXfNl hvaMnE"</span>&gt;</span>Normal Button<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"sc-fzXfNl kiyAbM"</span>&gt;</span>Primary Button<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
/*CSS 代码*/
.ciXJHl&nbsp;{
  text-align: center;
}
.hvaMnE&nbsp;{
  color: palevioletred;
  background: transparent;
  border-radius: 3px;
  border-width: 2px;
  border-style: solid;
  border-color: palevioletred;
  border-image: initial;
  margin: 0.5em 1em;
  padding: 0.25em 1em;
}
.kiyAbM&nbsp;{
  color: white;
  border-radius: 3px;
  border-width: 2px;
  border-style: solid;
  border-color: palevioletred;
  border-image: initial;
  margin: 0.5em 1em;
  padding: 0.25em 1em;
  background: palevioletred;
}
</code></pre>
<p>对比以上两段代码很容易发现，在编译后的样式代码中有很多重复的样式规则。这并不友好，不仅增加了编写样式的复杂度和代码量，连编译后也增加了冗余代码。</p>
<p>styled-components 只是 CSS in JavaScript 的一种解决方案，其他解决方案还有很多，<a href="https://github.com/MicheleBertoli/css-in-js">有兴趣的</a><a href="https://github.com/MicheleBertoli/css-in-js">同学</a><a href="https://github.com/MicheleBertoli/css-in-js">可以点击这里查阅 GitHub 上的资料学习</a>，上面收录了现有的 CSS in JavaScript 解决方案。</p>
<h3>总结</h3>
<p>对于样式文件的管理，推荐使用 7-1 模式简化后的目录结构，包括 pages/、components/、abastracts/、base/ 4 个目录。对于样式命名，可以采用 BEM 来命名组件、面向属性的方式来命名公共样式。</p>
<p>最后留一道思考题：说说你在项目中是如何管理样式代码的？</p>

---

### 精选评论

##### Mr.W：
> 在评价element的样式文件管理时说element的样式文件与组件文件分离这种方式开发的时候也不方便，经常需要跨多层目录查找和修改样式。但是，根据Sass Guidelines优化后得到的样式文件组织结构不也是与组件文件分离的吗？也会造成开发不方便的问题吧。期望能够得到回复与解答。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是样式与组件分离，同一个组件的样式与模板逻辑都会放在一起、不会出现跨目录层级查找的情况。
你说的开发方便是指“All in js“吗？
这种编写方式比较适用于小组件，一般情况下还是推荐按照 W3C 的规范，将组件的模板、逻辑和样式都分离开来。
一方面降低代码耦合度，另一方面也方便复用。

##### **鑫：
> 我个人是使用boostrap的css库，直接才用多class并行 比如：最后一个放的是当前div的私有样式，个人认为，用这样写很快，也好维护，写div就把涉及重排的样式都写上，最后剩下的定制css就很少了 。缺点是很class很长，丑

##### Warn：
> 个人项目中样式还是偏向BEM命名多一些

##### **威：
> 公共样式应放在哪个目录？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 全局的公共样式可以放在  src/base 目录哦~

##### **其：
> 打卡！

