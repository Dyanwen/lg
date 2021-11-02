<p data-nodeid="109731">上一课我们聊了 Webpack 的基本工作流程，分析了其中几个主要源码文件的执行过程，并介绍了 Compiler 和 Compilation 两个核心模块中的生命周期 Hooks。</p>


<p data-nodeid="108776">上节课后的思考题是，在 Compiler 和 Compilation 的工作流程里，最耗时的阶段分别是哪个。对于 Compiler 实例而言，耗时最长的显然是生成编译过程实例后的 make 阶段，在这个阶段里，会执行模块编译到优化的完整过程。而对于 Compilation 实例的工作流程来说，不同的项目和配置各有不同，但总体而言，编译模块和后续优化阶段的生成产物并压缩代码的过程都是比较耗时的。</p>
<p data-nodeid="108777">从这个思考题的答案中你也可以发现，不同项目的构建，在整个流程的前期初始化阶段与最后的产物生成阶段的构建时间区别不大。真正影响整个构建效率的还是 Compilation 实例的处理过程，这一过程又可分为两个阶段：编译模块和优化处理。今天我们主要讨论第一个阶段：编译模块阶段的效率提升。</p>
<h3 data-nodeid="108778">优化前的准备工作</h3>
<p data-nodeid="108779">在进入实际优化分析之前，首先需要进行两项准备工作：</p>
<ol data-nodeid="111681">
<li data-nodeid="111682">
<p data-nodeid="111683"><strong data-nodeid="111694">准备基于时间的分析工具</strong>：我们需要一类插件，来帮助我们统计项目构建过程中在编译阶段的耗时情况，这类工具可以是上一课中我们尝试手写的，也可以是使用第三方的工具。例如 <a href="https://github.com/stephencookdev/speed-measure-webpack-plugin" data-nodeid="111692">speed-measure-webpack-plugin</a>。</p>
</li>
<li data-nodeid="111684">
<p data-nodeid="111685" class=""><strong data-nodeid="111703">准备基于产物内容的分析工具</strong>：从产物内容着手分析是另一个可行的方式，因为从中我们可以找到对产物包体积影响最大的包的构成，从而找到那些冗余的、可以被优化的依赖项。通常，减少这些冗余的依赖包模块，不仅能减小最后的包体积大小，也能提升构建模块时的效率。通常可以使用 <a href="https://www.npmjs.com/package/webpack-bundle-analyzer" data-nodeid="111701">webpack-bundle-analyzer</a> 分析产物内容。</p>
</li>
</ol>



<p data-nodeid="112338" class="">在准备好相应的分析工具后，接下来，就开始分析编译阶段的具体提效方向。编译模块阶段所耗的时间是从单个入口点开始，编译每个模块的时间的总和。要提升这一阶段的构建效率，大致可以分为三个方向（这一节课的代码示例参见 <a href="https://github.com/fe-efficiency/lessons_fe_efficiency/tree/master/11_build_efficiency" data-nodeid="112346">11_build_efficiency</a>）：</p>

<ol data-nodeid="108786">
<li data-nodeid="108787">
<p data-nodeid="108788">减少执行编译的模块。</p>
</li>
<li data-nodeid="108789">
<p data-nodeid="108790">提升单个模块构建的速度。</p>
</li>
<li data-nodeid="108791">
<p data-nodeid="108792">并行构建以提升总体效率。</p>
</li>
</ol>
<h3 data-nodeid="108793">减少执行构建的模块</h3>
<p data-nodeid="108794">提升编译模块阶段效率的第一个方向就是减少执行编译的模块。显而易见，如果一个项目每次构建都需要编译 1000 个模块，但是通过分析后发现其中有 500 个不需要编译，显而易见，经过优化后，构建效率可以大幅提升。当然，前提是找到原本不需要进行构建的模块，下面我们就来逐一分析。</p>
<h4 data-nodeid="108795">IgnorePlugin</h4>
<p data-nodeid="114888">有的依赖包，除了项目所需的模块内容外，还会附带一些多余的模块。典型的例子是 <a href="https://www.npmjs.com/package/moment" data-nodeid="114893">moment</a> 这个包，一般情况下在构建时会自动引入其 locale 目录下的多国语言包，如下面的图片所示：</p>
<p data-nodeid="114889" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIKaAFpvlAAFYNtxZyV0507.png" alt="Drawing 0.png" data-nodeid="114897"></p>




<p data-nodeid="116150">但对于大多数情况而言，项目中只需要引入本国语言包即可。而 Webpack 提供的 IgnorePlugin 即可在构建模块时直接剔除那些需要被排除的模块，从而提升构建模块的速度，并减少产物体积，如下面的图片所示。</p>
<p data-nodeid="117099" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fILCATdbnAABZJ_SBA-k160.png" alt="Drawing 1.png" data-nodeid="117102"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fILaAS4hfAAEWkKJEE7E961.png" alt="Drawing 2.png" data-nodeid="117106"></p>





<p data-nodeid="108801">除了 moment 包以外，其他一些带有国际化模块的依赖包，例如之前介绍 Mock 工具中提到的 Faker.js 等都可以应用这一优化方式。</p>
<h4 data-nodeid="108802">按需引入类库模块</h4>
<p data-nodeid="118367">第二种典型的减少执行模块的方式是按需引入。这种方式一般适用于工具类库性质的依赖包的优化，典型例子是<a href="https://www.npmjs.com/package/lodash" data-nodeid="118372"> lodash </a>依赖包。通常在项目里我们只用到了少数几个 lodash 的方法，但是构建时却发现构建时引入了整个依赖包，如下图所示：</p>
<p data-nodeid="118368" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIMWAfBHWAAD0TYKbsl8944.png" alt="Drawing 3.png" data-nodeid="118376"></p>


<p data-nodeid="119637">要解决这个问题，效果最佳的方式是在导入声明时只导入依赖包内的特定模块，这样就可以大大减少构建时间，以及产物的体积，如下图所示。</p>
<p data-nodeid="119638" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIMyAfUzpAADukgQoyfw559.png" alt="Drawing 4.png" data-nodeid="119642"></p>


<p data-nodeid="108807">除了在导入时声明特定模块之外，还可以使用 babel-plugin-lodash 或 babel-plugin-import 等插件达到同样的效果。</p>
<p data-nodeid="120277" class="">另外，有同学也许会想到 <a href="https://webpack.js.org/guides/tree-shaking/" data-nodeid="120281">Tree Shaking</a>，这一特性也能减少产物包的体积，但是这里有两点需要注意：</p>

<ol data-nodeid="108809">
<li data-nodeid="108810">
<p data-nodeid="108811">Tree Shaking 需要相应导入的依赖包使用 ES6 模块化，而 lodash 还是基于 CommonJS ，需要替换为 lodash-es 才能生效。</p>
</li>
<li data-nodeid="108812">
<p data-nodeid="108813">相应的操作是在优化阶段进行的，换句话说，Tree Shaking 并不能减少模块编译阶段的构建时间。</p>
</li>
</ol>
<h4 data-nodeid="108814">DllPlugin</h4>
<p data-nodeid="158650">DllPlugin 是另一类减少构建模块的方式，它的核心思想是将项目依赖的框架等模块单独构建打包，与普通构建流程区分开。例如，原先一个依赖 React 与 react-dom 的文件，在构建时，会如下图般处理：</p>


<p data-nodeid="158005" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIOSAYnmjAAFH8Ofyt34986.png" alt="Drawing 5.png" data-nodeid="158011"><br>
而在通过 DllPlugin 和 DllReferencePlugin 分别配置后的构建时间就变成如下图所示，由于构建时减少了最耗时的模块，构建效率瞬间提升十倍。</p>






<p data-nodeid="156729" class=""><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIPOALYMeAAFQB_4TuTU987.png" alt="Drawing 6.png" data-nodeid="156736"></p>
<h4 data-nodeid="156730">Externals</h4>








<p data-nodeid="108821">Webpack 配置中的 externals 和 DllPlugin 解决的是同一类问题：将依赖的框架等模块从构建过程中移除。它们的区别在于：</p>
<ol data-nodeid="108822">
<li data-nodeid="108823">
<p data-nodeid="108824">在 Webpack 的配置方面，externals 更简单，而 DllPlugin 需要独立的配置文件。</p>
</li>
<li data-nodeid="108825">
<p data-nodeid="108826">DllPlugin 包含了依赖包的独立构建流程，而 externals 配置中不包含依赖框架的生成方式，通常使用已传入 CDN 的依赖包。</p>
</li>
<li data-nodeid="108827">
<p data-nodeid="108828">externals 配置的依赖包需要单独指定依赖模块的加载方式：全局对象、CommonJS、AMD 等。</p>
</li>
<li data-nodeid="108829">
<p data-nodeid="108830">在引用依赖包的子模块时，DllPlugin 无须更改，而 externals 则会将子模块打入项目包中。</p>
</li>
</ol>
<p data-nodeid="156090">externals 的示例如下面两张图，可以看到经过 externals 配置后，构建速度有了很大提升。</p>
<p data-nodeid="156091"><img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fIPiAJx62AAEEeJ5yROI594.png" alt="Drawing 7.png" data-nodeid="156095"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fIQSAAB3_AAD6KAV5S6M930.png" alt="Drawing 8.png" data-nodeid="156099"></p>








<h3 data-nodeid="125965">提升单个模块构建的速度</h3>


<p data-nodeid="108836">提升编译阶段效率的第二个方向，是在保持构建模块数量不变的情况下，提升单个模块构建的速度。具体来说，是通过减少构建单个模块时的一些处理逻辑来提升速度。这个方向的优化主要有以下几种：</p>
<h4 data-nodeid="108837">include/exclude</h4>
<p data-nodeid="108838">Webpack 加载器配置中的 include/exclude，是常用的优化特定模块构建速度的方式之一。</p>
<p data-nodeid="127211">include 的用途是只对符合条件的模块使用指定 Loader 进行转换处理。而 exclude 则相反，不对特定条件的模块使用该 Loader（例如不使用 babel-loader 处理 node_modules 中的模块）。如下面两张图片所示。</p>
<p data-nodeid="154804"><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIQmAVCu5AAH_1DmTw5Q884.png" alt="Drawing 9.png" data-nodeid="154807"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIRmAYw1PAAG8nEHHA1k680.png" alt="Drawing 10.png" data-nodeid="154811"><br>
这里有两点需要注意：</p>








<ol data-nodeid="108843">
<li data-nodeid="108844">
<p data-nodeid="108845">从上面的第二张图中可以看到，jquery 和 lodash 的编译过程仍然花费了数百毫秒，说明通过 include/exclude 排除的模块，并非不进行编译，而是使用 Webpack 默认的 js 模块编译器进行编译（例如推断依赖包的模块类型，加上装饰代码等）。</p>
</li>
<li data-nodeid="108846">
<p data-nodeid="108847">在一个 loader 中的 include 与 exclude 配置存在冲突的情况下，优先使用 exclude 的配置，而忽略冲突的 include 部分的配置，具体可以参照示例代码中的 webpack.inexclude.config.js。</p>
</li>
</ol>
<h4 data-nodeid="128794">noParse</h4>


<p data-nodeid="130036">Webpack 配置中的 module.noParse 则是在上述 include/exclude 的基础上，进一步省略了使用默认 js 模块编译器进行编译的时间，如下面两张图片所示。</p>
<p data-nodeid="153518"><img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIR-AABfPAAGe7gdO_nc998.png" alt="Drawing 11.png" data-nodeid="153521"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/DA/CgqCHl9fIS2ARrYXAAFGpNGygsY433.png" alt="Drawing 12.png" data-nodeid="153525"></p>

<h4 data-nodeid="152869">Source Map</h4>







<p data-nodeid="108854">Source Map 对于构建时间的影响在第三课中已经展开讨论过，这里再稍做总结：对于生产环境的代码构建而言，会根据项目实际情况判断是否开启 Source Map。在开启 Source Map 的情况下，优先选择与源文件分离的类型，例如 "source-map"。有条件也可以配合错误监控系统，将 Source Map 的构建和使用在线下监控后台中进行，以提升普通构建部署流程的速度。</p>
<h4 data-nodeid="108855">TypeScript 编译优化</h4>
<p data-nodeid="132232">Webpack 中编译 TS 有两种方式：使用 ts-loader 或使用 babel-loader。其中，在使用 ts-loader 时，由于 ts-loader 默认在编译前进行类型检查，因此编译时间往往比较慢，如下面的图片所示。</p>
<p data-nodeid="152234"><img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fITOAXQGlAAEcMk0PqdY814.png" alt="Drawing 13.png" data-nodeid="152237"><br>
通过加上配置项 transpileOnly: true，可以在编译时忽略类型检查，从而大大提升 TS 模块的编译速度，如下面的图片所示。</p>





<p data-nodeid="150954"><img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fITqAO9uoAAEDJx7jQcA803.png" alt="Drawing 14.png" data-nodeid="150957"><br>
而 babel-loader 则需要单独安装 @babel/preset-typescript 来支持编译 TS（Babel 7 之前的版本则还是需要使用 ts-loader）。babel-loader 的编译效率与上述 ts-loader 优化后的效率相当，如下面的图片所示。</p>





<p data-nodeid="149674"><img src="https://s0.lgstatic.com/i/image/M00/4E/DB/CgqCHl9fIUqAGSCpAAD9Llg28C8211.png" alt="Drawing 15.png" data-nodeid="149677"><br>
不过单独使用这一功能就丧失了 TS 中重要的类型检查功能，因此在许多脚手架中往往配合 ForkTsCheckerWebpackPlugin 一同使用。</p>





<h4 data-nodeid="108863">Resolve</h4>
<p data-nodeid="108864">Webpack 中的 resolve 配置制定的是在构建时指定查找模块文件的规则，例如：</p>
<ul data-nodeid="136033">
<li data-nodeid="136034">
<p data-nodeid="136035"><strong data-nodeid="136046">resolve.modules</strong>：指定查找模块的目录范围。</p>
</li>
<li data-nodeid="136036">
<p data-nodeid="136037" class=""><strong data-nodeid="136051">resolve.extensions</strong>：指定查找模块的文件类型范围。</p>
</li>
<li data-nodeid="136038">
<p data-nodeid="136039"><strong data-nodeid="136056">resolve.mainFields</strong>：指定查找模块的 package.json 中主文件的属性名。</p>
</li>
<li data-nodeid="136040">
<p data-nodeid="136041"><strong data-nodeid="136061">resolve.symlinks</strong>：指定在查找模块时是否处理软连接。</p>
</li>
</ul>


<p data-nodeid="137306">这些规则在处理每个模块时都会有所应用，因此尽管对小型项目的构建速度来说影响不大，但对于大型的模块众多的项目而言，这些配置的变化就可能产生客观的构建时长区别。例如下面的示例就展示了使用默认配置和增加了大量无效范围后，构建时长的变化情况：</p>
<p data-nodeid="148392"><img src="https://s0.lgstatic.com/i/image/M00/4E/DB/CgqCHl9fIU-AGs1fAAErO09KCQg428.png" alt="Drawing 16.png" data-nodeid="148395"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fIWCAKxMyAAErVYo_MgQ418.png" alt="Drawing 17.png" data-nodeid="148399"></p>

<h3 data-nodeid="147743">并行构建以提升总体效率</h3>







<p data-nodeid="108878">第三个编译阶段提效的方向是使用并行的方式来提升构建的效率。并行构建的方案早在 Webpack 2 时代已经出现，随着目前最新稳定版本 Webpack 4 的发布，人们发现在一般项目的开发阶段和小型项目的各构建流程中<a href="https://blog.johnnyreilly.com/2018/12/you-might-not-need-thread-loader.html" data-nodeid="109066">已经用不到这种并发的思路</a>了，因为在这些情况下，并发所需要的多进程管理与通信所带来的额外时间成本可能会超过使用工具带来的收益。但是在大中型项目的生产环境构建时，这类工具仍有发挥作用的空间。这里我们介绍两类并行构建的工具： HappyPack 与 thread-loader，以及 parallel-webpack。</p>
<h4 data-nodeid="108879">HappyPack 与 thread-loader</h4>
<p data-nodeid="140127">这两种工具的本质作用相同，都作用于模块编译的 Loader 上，用于在特定 Loader 的编译过程中，以开启多进程的方式加速编译。HappyPack 诞生较早，而 thread-loader 参照它的效果实现了更符合 Webpack 中 Loader 的编写方式。下面就以 thread-loader 为例，来看下应用前后的构建时长对比，如下面的两张图所示。</p>
<p data-nodeid="147106"><img src="https://s0.lgstatic.com/i/image/M00/4E/DB/CgqCHl9fIWaAKvjDAAGxNVse3m4379.png" alt="Drawing 18.png" data-nodeid="147109"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/CF/Ciqc1F9fIXOAHx6XAAIyabhj3_g078.png" alt="Drawing 19.png" data-nodeid="147113"></p>

<h4 data-nodeid="146457">parallel-webpack</h4>












<p data-nodeid="143274">并发构建的第二种场景是针对与多配置构建。Webpack 的配置文件可以是一个包含多个子配置对象的数组，在执行这类多配置构建时，默认串行执行，而通过 parallel-webpack，就能实现相关配置的并行处理。从下图的示例中可以看到，通过不同配置的并行构建，构建时长缩短了 30%：</p>
<p data-nodeid="145820"><img src="https://s0.lgstatic.com/i/image/M00/4E/DB/CgqCHl9fIXuARXhnAADx6PzQuE0879.png" alt="Drawing 20.png" data-nodeid="145823"><br>
<img src="https://s0.lgstatic.com/i/image/M00/4E/D0/Ciqc1F9fIbCAL6knAAEbXZ1tRpw256.png" alt="Drawing 21.png" data-nodeid="145827"></p>








<h3 data-nodeid="108888">总结</h3>
<p data-nodeid="108889">这节课我们整理了 Webpack 构建中编译模块阶段的构建效率优化方案。对于这一阶段的构建效率优化可以分为三个方向：以减少执行构建的模块数量为目的的方向、以提升单个模块构建速度为目的的方向，以及通过并行构建以提升整体构建效率的方向。每个方向都包含了若干解决工具和配置。</p>
<p data-nodeid="108890">今天课后的<strong data-nodeid="109091">思考题是</strong>：你的项目中是否都用到了这些解决方案呢？希望你结合课程的内容，和所开发的项目中用到的优化方案进行对比，查漏补缺。如果有这个主题方面其他新的解决方案，也欢迎在留言区讨论分享。</p>

---

### 精选评论

##### **喝王老吉：
> 这个很干货

##### *策：
> 有好几个优化的经验，我感觉优化效果最明显的，还是缓存

