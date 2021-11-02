<p data-nodeid="9777" class="">在上一个小节，我介绍了 Echarts 图表可视化组件基础，了解了 Echarts 和静态页面结合的基本使用方法。本节我们来看一下，如何使用 Python 开发基于 Echarts 的图表组件，这里面需要用到 Echarts 的 Python 语言定制版本 PyEcharts 框架。我们来看一下 PyEcharts 的官方网站、培训教程、源码资源和安装部署。</p>


<h3 data-nodeid="9432">PyEcharts 简介</h3>
<p data-nodeid="9433"><strong data-nodeid="9528">Python 是一门富有表现力的开发语言，是数据科学和人工智能在学习和科研场景下的首选语言</strong>。Python 语言的主要特点包括以下几个方面。</p>
<ul data-nodeid="9434">
<li data-nodeid="9435">
<p data-nodeid="9436"><strong data-nodeid="9533">简单、易学</strong>：Python 语言的语法规则、数据类型相对简单易学，可以快速入门。</p>
</li>
<li data-nodeid="9437">
<p data-nodeid="9438"><strong data-nodeid="9538">开源、免费</strong>：Python 语言是一个基于 C/C++的开源的项目，可以免费使用。</p>
</li>
<li data-nodeid="9439">
<p data-nodeid="9440"><strong data-nodeid="9543">跨平台支持</strong>：Python 语言支持主流的操作系统：Windows、Linux、Mac、IOS、安卓等，Python 程序可以在各个平台之间实现无缝迁移。</p>
</li>
<li data-nodeid="9441">
<p data-nodeid="9442"><strong data-nodeid="9548">资源丰富</strong>：Python 语言具有丰富的开发资源库，可以直接复用，尤其是在数据科学和人工智能方面，资源尤为丰富。</p>
</li>
<li data-nodeid="9443">
<p data-nodeid="9444"><strong data-nodeid="9553">可扩展性</strong>：Python 语言支持跨语言开发，比如 C/C++程序混合编程。</p>
</li>
</ul>
<p data-nodeid="9445">通过上一个小节，我们了解到：Echarts 是一个开源的、免费的、成熟的、商业级的图表可视化框架，它基于 JavaScript 语言开发，是国内使用最多和最为广泛的可视化图表框架之一。而 PyEcharts 则是 Echarts 数据可视化框架，在 Python 开发语言环境下的实现版本，借助 PyEcharts，开发者可以使用 Python 语言直接生成可视化图表。</p>
<h3 data-nodeid="9446">PyEcharts 特点</h3>
<p data-nodeid="9447">PyEcharts 作为 Echarts 的 Python 开发语言定制版本，一方面继承了 Echarts 图表可视化框架的特性，另外一方面，其作为一个开源项目，也具备自己的特性。这两个方面结合构成了 PyEcharts 的五个特点。</p>
<ul data-nodeid="9448">
<li data-nodeid="9449">
<p data-nodeid="9450"><strong data-nodeid="9561">基于 Python 语言设计</strong>：这是 PyEcharts 最大的特点，Python 语言入门简单，适合学生使用；同时它又有丰富的资源并支持跨平台开发，也适合科研人员研究使用。因此，Python 具有非常广泛的用户基础。PyEcharts 基于 Python 语言开发，也很好地继承了 Python 的这一特点。</p>
</li>
<li data-nodeid="9451">
<p data-nodeid="9452"><strong data-nodeid="9566">图表类型丰富</strong>： 虽然 PyEcharts 没有继承 Echarts 的全部图表，但是它也提供了最常用的 30 多种图表类型（具体数量与 PyEcharts 的版本相关，而且在持续更新中）。这些可用的图表已足以满足日常的数据可视化的呈现需求。</p>
</li>
<li data-nodeid="9453">
<p data-nodeid="9454"><strong data-nodeid="9571">源码开源免费</strong>：PyEcharts 是一个开源项目，可以免费用于商业用途。</p>
</li>
<li data-nodeid="9455">
<p data-nodeid="9456"><strong data-nodeid="9576">文档教程健全</strong>：PyEcharts 提供了相对完备的文档教程和示例程序，可以有效降低学习门槛。</p>
</li>
<li data-nodeid="9457">
<p data-nodeid="9458"><strong data-nodeid="9581">Web 集成方便</strong>：PyEcharts 可以很轻松地和 Flask、Django 等 Web 框架整合，以 Web 页面的方式呈现，便于跨团队、跨部门、跨地域的合作和分享。</p>
</li>
</ul>
<h3 data-nodeid="9459">PyEcharts 官网</h3>
<p data-nodeid="9460">PyEcharts 官方网站，提供了丰富的学习资源，包括：文档、教程和案例。官方网站地址为<a href="https://pyecharts.org/#/" data-nodeid="9586">https://pyecharts.org/#/</a></p>
<p data-nodeid="9461"><img src="https://s0.lgstatic.com/i/image/M00/47/B6/CgqCHl9IumuAVgTfAAGOvC3Qpdk427.png" alt="Drawing 0.png" data-nodeid="9589"></p>
<div data-nodeid="9462"><p style="text-align:center">PyEcharts 官方网站</p></div>
<p data-nodeid="9463">PyEcharts 提供了丰富的文档教程，学员可以通过官方教程，快速地掌握它的使用方法和技巧。PyEcharts 官方教程页面如下图所示：</p>
<p data-nodeid="9464"><img src="https://s0.lgstatic.com/i/image/M00/47/AC/Ciqc1F9IvGKAKziUAAGBYp79ru8105.png" alt="Drawing 1.png" data-nodeid="9593"></p>
<div data-nodeid="9465"><p style="text-align:center">PyEcharts 官方教程</p></div>
<p data-nodeid="9466">PyEcharts 除官方教程之外，还有一个案例演示平台：<a href="https://gallery.pyecharts.org/" data-nodeid="9597">https://gallery.pyecharts.org/#/</a>，学员可以通过该平台，查看 PyEcharts 支持的所有图表组件的演示案例，包括源码、配置方法、图表预览等。PyEcharts 案例演示平台如下图所示：</p>
<p data-nodeid="9467"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9IvuiADuKzAAFiuVldI0w205.png" alt="Drawing 2.png" data-nodeid="9601"></p>
<div data-nodeid="9468"><p style="text-align:center">PyEcharts-Gallery</p></div>
<h3 data-nodeid="9469">PyEcharts 源码</h3>
<p data-nodeid="9470">PyEcharts 是一个开源项目，其源码资源托管在 GitHub，源码地址：<a href="https://github.com/pyecharts/pyecharts" data-nodeid="9606">https://github.com/pyecharts/pyecharts</a>，PyEcharts 源码结构如下图所示：</p>
<div data-nodeid="9471"><p style="text-align:center">PyEcharts 源码结构如下图所示：</p></div>
<p data-nodeid="9472"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9IvvqAJ3EyAAGX6EDg61I245.png" alt="Drawing 3.png" data-nodeid="9610"></p>
<div data-nodeid="9473"><p style="text-align:center">PyEcharts 源码结构</p></div>
<h3 data-nodeid="9474">PyEcharts 安装</h3>
<p data-nodeid="9475">使用 PyEcharts 图表组件库之前，需要安装对应的 PyEcharts 库文件，安装的方式非常简单，只需要执行指令：pip install pyecharts 即可。安装完成以后可以通过：pip show pyecharts 指令查看安装结果信息，如下图所示：</p>
<p data-nodeid="9476"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9IvwiAR6JlAACC5BUL3Uw075.png" alt="Drawing 4.png" data-nodeid="9615"></p>
<div data-nodeid="9477"><p style="text-align:center">文件安装</p></div>
<p data-nodeid="9478">PyEcharts 安装完成以后，可以通过 import 指令直接引入库中的组件对象，不过需要注意的是：<strong data-nodeid="9621">PyEcharts 的大版本 v0.5x 和 v1.x 之间，引入方式不同，因为后续 v0.5x 版本不再维护，我们后续所有的示例程序的版本都是基于 v1.0x</strong>。以柱状图为例，我们需要导入的对象如下所示：</p>
<pre class="lang-java" data-nodeid="9479"><code data-language="java">from pyecharts <span class="hljs-keyword">import</span> options as opts
from pyecharts.charts <span class="hljs-keyword">import</span> Bar
from pyecharts.commons.utils <span class="hljs-keyword">import</span> JsCode
from pyecharts.globals <span class="hljs-keyword">import</span> ThemeType
</code></pre>
<p data-nodeid="9480">上面的代码中，opts 为配置参数、Bar 为柱状图组件、JsCode 为代码段对象、ThemeType 为图表主题样式对象。</p>
<h3 data-nodeid="9481">PyEcharts 配置</h3>
<p data-nodeid="9482">PyEcharts 的配置主要是图表的参数设置，内容包括图表的提示框组件参数、图例组件参数、工具箱组件参数、X 轴组件参数、Y 轴组件参数、数据缩放组件参数、视图效果组件参数和主题样式参数，配置好的参数最终体现在可视化的图表中，一个典型的图表参数与页面呈现图表地对应关系，如下图所示：</p>
<p data-nodeid="9483"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9IvxyAcE66AAb0iLjmfrA024.png" alt="Drawing 5.png" data-nodeid="9627"></p>
<div data-nodeid="9484"><p style="text-align:center">PyEcharts 图表参数</p></div>
<p data-nodeid="9485">上图的左侧部分，以红色字体标注出来的图表元素是 PyEcharts 图表的配置项类型，右侧部分代表的是各个配置项在代码中对应的标识符号。在<strong data-nodeid="9637">04 课时</strong>，我们学习了 Echarts 的参数配置内容，可以发现，PyEcharts 的配置参数和图表呈现元素之间的映射关系，与 Echarts 完全相同，差异点只在于实现语言。基于 PyEcharts 配置参数的具体方法，我会在<strong data-nodeid="9638">06 课时</strong>，给大家做详细的介绍。</p>
<h3 data-nodeid="9486">PyEcharts 示例</h3>
<p data-nodeid="9487">了解了 PyEcharts 文件引入和配置参数之后，我们来看一个具体的案例，通过案例学习一下 PyEcharts 图表设计的方法。</p>
<p data-nodeid="9488">PyEcharts 图表开发的过程主要分为四个步骤：<strong data-nodeid="9658">PyEcharts 图表组件引入</strong>、<strong data-nodeid="9659">图表对象声明</strong>、<strong data-nodeid="9660">图表对象参数设置</strong>、<strong data-nodeid="9661">图表对象渲染</strong>。首先我们来看一下案例运行之后的效果，然后我们再逐步拆解实现过程。下图是一个基于 PyEcharts 实现的柱状图的呈现效果：</p>
<p data-nodeid="9489"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9IvyqAK0drAAA9MkhKhW4133.png" alt="Drawing 6.png" data-nodeid="9664"></p>
<div data-nodeid="9490"><p style="text-align:center">PyEcharts 柱状图案例</p></div>
<p data-nodeid="9491">首先是图表组件引入。<strong data-nodeid="9670">要想实现以上的柱状图，需要在源码中，引入 PyEcharts 柱状图组件</strong>，具体的代码如下所示：</p>
<pre class="lang-java" data-nodeid="9492"><code data-language="java">from pyecharts.charts <span class="hljs-keyword">import</span> Bar
</code></pre>
<p data-nodeid="9493"><strong data-nodeid="9675">然后是引入柱状图组件，完成图表对象的声明</strong>，具体代码如下所示：</p>
<pre class="lang-java" data-nodeid="9494"><code data-language="java">bar = Bar()
</code></pre>
<p data-nodeid="9495">上述代码通过类 Bar 声明了一个柱状图图表组件的实例对象：bar。完成对象的声明以后，<strong data-nodeid="9681">接下来需要进行参数配置</strong>，具体的代码如下所示：</p>
<pre class="lang-dart" data-nodeid="9496"><code data-language="dart"># 参数设置：x 轴数据
bar.add_xaxis([<span class="hljs-string">"衬衫"</span>, <span class="hljs-string">"羊毛衫"</span>, <span class="hljs-string">"雪纺衫"</span>, <span class="hljs-string">"裤子"</span>, <span class="hljs-string">"高跟鞋"</span>, <span class="hljs-string">"袜子"</span>])
# 参数设置：y 轴数据
bar.add_yaxis(<span class="hljs-string">"商家 A"</span>, [<span class="hljs-number">5</span>, <span class="hljs-number">20</span>, <span class="hljs-number">36</span>, <span class="hljs-number">10</span>, <span class="hljs-number">75</span>, <span class="hljs-number">90</span>])
</code></pre>
<p data-nodeid="9497">上述代码分别设置了柱状图图表对象的 x 轴数据和 y 轴数据。<strong data-nodeid="9687">参数设置完成以后，就是最后一步：图表对象渲染。图表对象的渲染只需要一行代码</strong>，具体的代码如下所示：</p>
<pre class="lang-yaml" data-nodeid="9498"><code data-language="yaml"><span class="hljs-comment"># 图表渲染：默认文件命：render.html,默认路径：当前目录</span>
<span class="hljs-string">bar.render()</span>
</code></pre>
<p data-nodeid="9499">PyEcharts 图表组件的渲染，在不添加任何参数的情况下，默认会在 Python 程序文件当前目录下，生成一个名为：render.html 的 HTML 页面文件。如下图所示：</p>
<p data-nodeid="9500"><img src="https://s0.lgstatic.com/i/image/M00/47/AE/Ciqc1F9Iv06AVSczAAAi4Asa-mQ610.png" alt="Drawing 7.png" data-nodeid="9691"></p>
<div data-nodeid="9501"><p style="text-align:center">渲染文件</p></div>
<p data-nodeid="9502">该页面文件的源码结构如下所示：</p>
<pre class="lang-js" data-nodeid="9503"><code data-language="js">&lt;!DOCTYPE&nbsp;html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">meta</span>&nbsp;<span class="hljs-attr">charset</span>=<span class="hljs-string">"UTF-8"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Awesome-pyecharts<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">script</span>&nbsp;<span class="hljs-attr">type</span>=<span class="hljs-string">"text/javascript"</span>&nbsp;<span class="hljs-attr">src</span>=<span class="hljs-string">"https://assets.pyecharts.org/assets/echarts.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">div</span>&nbsp;<span class="hljs-attr">id</span>=<span class="hljs-string">"8e37f723c1d747c584bfcae32007d042"</span>&nbsp;<span class="hljs-attr">class</span>=<span class="hljs-string">"chart-container"</span>&nbsp;<span class="hljs-attr">style</span>=<span class="hljs-string">"width:900px;&nbsp;height:500px;"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;chart_8e37f723c1d747c584bfcae32007d042&nbsp;=&nbsp;echarts.init(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'8e37f723c1d747c584bfcae32007d042'</span>),&nbsp;<span class="hljs-string">'white'</span>,&nbsp;{<span class="hljs-attr">renderer</span>:&nbsp;<span class="hljs-string">'canvas'</span>});
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;option_8e37f723c1d747c584bfcae32007d042&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animation"</span>:&nbsp;<span class="hljs-literal">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationThreshold"</span>:&nbsp;<span class="hljs-number">2000</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationDuration"</span>:&nbsp;<span class="hljs-number">1000</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationEasing"</span>:&nbsp;<span class="hljs-string">"cubicOut"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationDelay"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationDurationUpdate"</span>:&nbsp;<span class="hljs-number">300</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationEasingUpdate"</span>:&nbsp;<span class="hljs-string">"cubicOut"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"animationDelayUpdate"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"color"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#c23531"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#2f4554"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#61a0a8"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#d48265"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#749f83"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#ca8622"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#bda29a"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#6e7074"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#546570"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#c4ccd3"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#f05b72"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#ef5b9c"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#f47920"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#905a3d"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#fab27b"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#2a5caa"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#444693"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#726930"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#b2d235"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#6d8346"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#ac6767"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#1d953f"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#6950a1"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"#918597"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"series"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"bar"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"name"</span>:&nbsp;<span class="hljs-string">"\u5546\u5bb6A"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"data"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">5</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">20</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">36</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">10</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">75</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">90</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"barCategoryGap"</span>:&nbsp;<span class="hljs-string">"20%"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"label"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"position"</span>:&nbsp;<span class="hljs-string">"top"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"margin"</span>:&nbsp;<span class="hljs-number">8</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"legend"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"data"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u5546\u5bb6A"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"selected"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u5546\u5bb6A"</span>:&nbsp;<span class="hljs-literal">true</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"tooltip"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"trigger"</span>:&nbsp;<span class="hljs-string">"item"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"triggerOn"</span>:&nbsp;<span class="hljs-string">"mousemove|click"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"axisPointer"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"line"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"textStyle"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"fontSize"</span>:&nbsp;<span class="hljs-number">14</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"borderWidth"</span>:&nbsp;<span class="hljs-number">0</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"xAxis"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"scale"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"nameLocation"</span>:&nbsp;<span class="hljs-string">"end"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"nameGap"</span>:&nbsp;<span class="hljs-number">15</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"gridIndex"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"inverse"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"offset"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"splitNumber"</span>:&nbsp;<span class="hljs-number">5</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"minInterval"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"splitLine"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"lineStyle"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"width"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"opacity"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"curveness"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"solid"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"data"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u886c\u886b"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u7f8a\u6bdb\u886b"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u96ea\u7eba\u886b"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u88e4\u5b50"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u9ad8\u8ddf\u978b"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"\u889c\u5b50"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;],
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"yAxis"</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"scale"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"nameLocation"</span>:&nbsp;<span class="hljs-string">"end"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"nameGap"</span>:&nbsp;<span class="hljs-number">15</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"gridIndex"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"inverse"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"offset"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"splitNumber"</span>:&nbsp;<span class="hljs-number">5</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"minInterval"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"splitLine"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"show"</span>:&nbsp;<span class="hljs-literal">false</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"lineStyle"</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"width"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"opacity"</span>:&nbsp;<span class="hljs-number">1</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"curveness"</span>:&nbsp;<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"type"</span>:&nbsp;<span class="hljs-string">"solid"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;chart_8e37f723c1d747c584bfcae32007d042.setOption(option_8e37f723c1d747c584bfcae32007d042);
&nbsp;&nbsp;&nbsp;&nbsp;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>&nbsp;
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</span></code></pre>
<div data-nodeid="9504"><p style="text-align:center">页面源码</p></div>
<p data-nodeid="9505">通过以上渲染完成以后的代码，对比<strong data-nodeid="9714">04 课时</strong>，Echarts 与 HTML 结合生成的图表页面的源码，我们可以发现 PyEcharts 图表组件的渲染过程，其实是生成 HTML 和 JavaScripts Echarts 静态页面的过程，内容包括：<strong data-nodeid="9715">Echarts 文件引入</strong>、<strong data-nodeid="9716">HTML DOM 对象声明</strong>、<strong data-nodeid="9717">图表对象初始化</strong>和<strong data-nodeid="9718">参数设置</strong>。PyEcharts 通过渲染函数 Render()，实现了从 Python 程序到浏览器默认能够识别的 HTML 和 JavaScripts 脚本程序的转化。</p>
<p data-nodeid="9506">通过以上操作，我们完成了一个 PyEcharts 基本的图表对象的创建工作，通过浏览器即可以直接预览渲染生成的页面文件的内容。该程序完整的代码结构如下所示：</p>
<pre class="lang-dart" data-nodeid="9507"><code data-language="dart">from pyecharts.charts <span class="hljs-keyword">import</span> Bar
# 柱状图对象声明
bar = Bar()
# 参数设置：x 轴数据
bar.add_xaxis([<span class="hljs-string">"衬衫"</span>, <span class="hljs-string">"羊毛衫"</span>, <span class="hljs-string">"雪纺衫"</span>, <span class="hljs-string">"裤子"</span>, <span class="hljs-string">"高跟鞋"</span>, <span class="hljs-string">"袜子"</span>])
# 参数设置：y 轴数据
bar.add_yaxis(<span class="hljs-string">"商家 A"</span>, [<span class="hljs-number">5</span>, <span class="hljs-number">20</span>, <span class="hljs-number">36</span>, <span class="hljs-number">10</span>, <span class="hljs-number">75</span>, <span class="hljs-number">90</span>])

# 图表渲染：默认文件命：render.html,默认路径：当前目录
bar.render()
</code></pre>
<h3 data-nodeid="9508">小结</h3>
<p data-nodeid="9509">本节课，我介绍了 PyEcharts 图表组件的官方网站、源码资源、安装方式、配置方法和实例程序。通过以上内容的学习，我想你应该掌握了 PyEcharts 开发环境的安装方法。</p>
<p data-nodeid="9510">依照惯例，我会在这里将一些共性的问题展示出来，以便你能更好地理解这门课的内容。</p>
<p data-nodeid="9511"><strong data-nodeid="9730">问题 1</strong>：<strong data-nodeid="9731">老师，我很好奇，大数据和 AI 的区别是什么呀？大数据，再深化下去，并和其他很多方面整合，才是 AI 吗？大数据的话，需要的编程语言、技术栈、框架等，都是需要些什么的呢？</strong></p>
<p data-nodeid="9512">这是一个相对复杂的问题，了解这个之前需要明确两点：<strong data-nodeid="9741">一是你从哪个视角来看</strong>，是业务视角还是技术视角；<strong data-nodeid="9742">二是你是否具备关于 AI 和大数据的完整的知识体系</strong>。</p>
<p data-nodeid="9513"><strong data-nodeid="9751">从业务视角</strong>，大数据也好，AI 也好，都是手段，不是目标，<strong data-nodeid="9752">业务用户关注的是最终提供给终端用户的服务能力</strong>；</p>
<p data-nodeid="9514"><strong data-nodeid="9757">从技术视角</strong>，大数据特指四个层次的东西：分布式存储和计算、数据仓库、工具平台（类似 redash）、数据应用（报表、BI）；AI 特指：神经网络模型、深度学习框架（TF、PyTorch、Caffe 等）、拟人化能力（语音识别、机器视觉、NLP、知识图谱等）。</p>
<p data-nodeid="9515">综上所述，二者是不同方向上的两条技术线，彼此有交集，有依赖，也相互独立，可以共同构建业务服务能力，对外输出。</p>
<p data-nodeid="9516"><strong data-nodeid="9766">问题 2</strong>：<strong data-nodeid="9767">老师，可以在 Windows 系统下安装 redash 吗？或者说 Windows 系统下有什么可视化工具吗？</strong></p>
<p data-nodeid="9517">Windows 安装是可以的，Redash7.0 实际部署成功过，不过用的 python2.7，相对比较复杂。Celery4.x 以后的版本不支持在 windows 上运行，需要特殊处理。GitHub 上直接下载的源码，如果使用 python3.x 版本调试，但需要修正大量的代码，这就比较考验个人的代码调试和错误处理能力了。</p>
<p data-nodeid="9518"><strong data-nodeid="9772">问题 3：Redash 展现的图表可以随着数据的变化而变化吗？</strong></p>
<p data-nodeid="9519">Redash 的仪表盘支持定时刷新机制，可以满足这个需求。如下图所示：</p>
<p data-nodeid="9520" class=""><img src="https://s0.lgstatic.com/i/image/M00/47/BA/CgqCHl9Iv3WAb5xxAADvSadzMjc298.png" alt="Drawing 8.png" data-nodeid="9776"></p>

---

### 精选评论


