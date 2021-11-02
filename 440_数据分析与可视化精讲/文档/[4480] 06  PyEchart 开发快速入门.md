<p data-nodeid="9550" class="">本课时是课程“<strong data-nodeid="9556">模块二：环境部署篇</strong>”的第三课，也是最后一个课时。</p>









<p data-nodeid="1862">在上一个小节，我介绍了 PyEcharts 图表可视化框架，带你了解了它的官方网站、文档教程、源码资源、参数配置和安装部署。通过一个最简单的示例程序，我向你演示了 PyEcharts 生成可视化图表的过程。但仅仅掌握这些内容还是不够的，要使用 PyEcharts，我们还需要更多的知识。本节课我会带你学习如何使用 PyEcharts 来构建可视化图表，并掌握其常用的图表类型、参数项配置、图表渲染和主题样式的设置方法。</p>
<h3 data-nodeid="1863">PyEcharts 图表类型</h3>
<p data-nodeid="1864">我们先来看 PyEcharts 图表类型。PyEcharts 支持的图表类型有 30 多种，具体的图表数量和版本相关，需要结合具体的版本，通过 PyEcharts 源码文件最终确定。记录 PyEcharts 支持图表类型的源码文件地址为：<a href="https://github.com/pyecharts/pyecharts/blob/master/pyecharts/globals.py" data-nodeid="2017">https://github.com/pyecharts/pyecharts/blob/master/pyecharts/globals.py</a>，我们使用的 PyEcharts v1.6.2 版本支持的图表类型如下图所示：</p>
<p data-nodeid="1865"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcayAJWLCAAC0RZBQdfY557.png" alt="Drawing 0.png" data-nodeid="2021"></p>
<div data-nodeid="1866"><p style="text-align:center">PyEcharts v1.6.2 支持的图表类型</p></div>
<p data-nodeid="1867">基于 PyEcharts 官方指导教程的分类方式，图表共分为 5 种类型：基本图表、直角坐标系图表、地理位置图表、树型图表和 3D 图表。具体的图表类型和分类关系如下图所示：</p>
<p data-nodeid="1868"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcbyAUmJQAAFpUMkSB90532.png" alt="Drawing 1.png" data-nodeid="2025"></p>
<div data-nodeid="1869"><p style="text-align:center">PyEcharts 图表分类</p></div>
<p data-nodeid="1870">上图中，橙色标注的图表类型是我会在课程的“<strong data-nodeid="2031">模块三</strong>”讲到的内容，其他的图表可以参照我们的课程，结合官方教程学习。了解了 PyEcharts 的图表类型之后，我们来看一下如何导入图表对象。图表对象导入的源码如下所示：</p>
<pre class="lang-java" data-nodeid="1871"><code data-language="java">from pyecharts.charts <span class="hljs-keyword">import</span> Bar
</code></pre>
<p data-nodeid="1872">上述代码中，从 pyecharts.charts 模块导入了图表对象 Bar。通过这段代码，我们可以导入前面讲到的全部图表类型，只需要替换图表对象的类型名称即可。具体的图表类型导入时的图表名称如下图所示：</p>
<p data-nodeid="1873"><img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9PccyAHYPCAAEIBFhBzvM139.png" alt="Drawing 2.png" data-nodeid="2035"></p>
<div data-nodeid="1874"><p style="text-align:center">导入图表名称</p></div>
<p data-nodeid="37683" class=""><strong data-nodeid="37688">导入图表类型以后，声明一个图表对象，再进行相应的参数设置，就可以生成一个可视化图表了</strong>。这部分内容，我接下来会做详细的介绍。</p>































<h3 data-nodeid="1876">PyEcharts 配置参数</h3>
<h4 data-nodeid="1877">概述</h4>
<p data-nodeid="1878">我在“<strong data-nodeid="2050">04 | 图表组件：Echarts数据可视化图表基础</strong>”这一课时中简单介绍了 PyEcharts 的参数配置项和图表元素之间的映射关系：</p>
<p data-nodeid="1879"><img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9Pcd6ASp69AADMlfh7ovg849.png" alt="Drawing 3.png" data-nodeid="2053"></p>
<div data-nodeid="1880"><p style="text-align:center">PyEcharts 图表配置项</p></div>
<p data-nodeid="1881">这里，我将详细介绍具体的配置项的参数设置方法。<strong data-nodeid="2067">PyEcharts 配置项分全局配置项和系列配置项，其中全局配置项作用域为整个图表</strong>（与具体需要呈现的数据内容无关），<strong data-nodeid="2068">可以理解为静态部分</strong>；<strong data-nodeid="2069">系列配置项作用范围为基于数据动态绘制的部分</strong>。首先我们来看一下 PyEcharts 常用的全局配置项和功能列表：</p>
<table data-nodeid="1883">
<thead data-nodeid="1884">
<tr data-nodeid="1885">
<th align="center" data-org-content="序号" data-nodeid="1887">序号</th>
<th data-org-content="类" data-nodeid="1888">类</th>
<th data-org-content="名称" data-nodeid="1889">名称</th>
<th data-org-content="主要功能" data-nodeid="1890">主要功能</th>
</tr>
</thead>
<tbody data-nodeid="1895">
<tr data-nodeid="1896">
<td align="center" data-org-content="1" data-nodeid="1897">1</td>
<td data-org-content="pyecharts.options.InitOpts" data-nodeid="1898">pyecharts.options.InitOpts</td>
<td data-org-content="初始化配置项" data-nodeid="1899">初始化配置项</td>
<td data-org-content="尺寸、主题样式" data-nodeid="1900">尺寸、主题样式</td>
</tr>
<tr data-nodeid="1901">
<td align="center" data-org-content="2" data-nodeid="1902">2</td>
<td data-org-content="pyecharts.options.ToolboxOpts" data-nodeid="1903">pyecharts.options.ToolboxOpts</td>
<td data-org-content="工具箱配置项" data-nodeid="1904">工具箱配置项</td>
<td data-org-content="是否显示、位置、工具项" data-nodeid="1905">是否显示、位置、工具项</td>
</tr>
<tr data-nodeid="1906">
<td align="center" data-org-content="3" data-nodeid="1907">3</td>
<td data-org-content="pyecharts.options.TitleOpts" data-nodeid="1908">pyecharts.options.TitleOpts</td>
<td data-org-content="标题栏配置项" data-nodeid="1909">标题栏配置项</td>
<td data-org-content="标题、副标题、链接" data-nodeid="1910">标题、副标题、链接</td>
</tr>
<tr data-nodeid="1911">
<td align="center" data-org-content="4" data-nodeid="1912">4</td>
<td data-org-content="pyecharts.options.LegendOpts" data-nodeid="1913">pyecharts.options.LegendOpts</td>
<td data-org-content="图例配置项" data-nodeid="1914">图例配置项</td>
<td data-org-content="设置图例参数" data-nodeid="1915">设置图例参数</td>
</tr>
<tr data-nodeid="1916">
<td align="center" data-org-content="5" data-nodeid="1917">5</td>
<td data-org-content="pyecharts.options.DataZoomOpts" data-nodeid="1918">pyecharts.options.DataZoomOpts</td>
<td data-org-content="区域缩放配置项" data-nodeid="1919">区域缩放配置项</td>
<td data-org-content="控制图表缩放" data-nodeid="1920">控制图表缩放</td>
</tr>
<tr data-nodeid="1921">
<td align="center" data-org-content="6" data-nodeid="1922">6</td>
<td data-org-content="pyecharts.options.TooltipOpts" data-nodeid="1923">pyecharts.options.TooltipOpts</td>
<td data-org-content="提示框配置项" data-nodeid="1924">提示框配置项</td>
<td data-org-content="鼠标滑过后弹出的提示信息" data-nodeid="1925">鼠标滑过后弹出的提示信息</td>
</tr>
<tr data-nodeid="1926">
<td align="center" data-org-content="7" data-nodeid="1927">7</td>
<td data-org-content="pyecharts.options.VisualMapOpts" data-nodeid="1928">pyecharts.options.VisualMapOpts</td>
<td data-org-content="视觉映射配置项" data-nodeid="1929">视觉映射配置项</td>
<td data-nodeid="1930"></td>
</tr>
<tr data-nodeid="1931">
<td align="center" data-org-content="8" data-nodeid="1932">8</td>
<td data-org-content="pyecharts.options.AxisOpts" data-nodeid="1933">pyecharts.options.AxisOpts</td>
<td data-org-content="坐标轴配置项" data-nodeid="1934">坐标轴配置项</td>
<td data-org-content="X 轴参数、Y 轴参数" data-nodeid="1935">X 轴参数、Y 轴参数</td>
</tr>
</tbody>
</table>
<p data-nodeid="42227" class="">了解了各配置项的类、名称、功能和位置之后，我们结合一个具体的案例，来看一下各个配置项的配置方法。我会采用上一小节的案例源码，通过逐步添加新的配置项和展示呈现效果的方式来做详细的介绍。由于各配置项的使用方法基本相同，我只重点介绍其中的两个：<strong data-nodeid="42237">初始化配置项</strong>和<strong data-nodeid="42238">标题配置项</strong>，其他部分你可以参考官方教程，结合我介绍的方法进行配置。</p>





<h4 data-nodeid="1937">初始化配置项</h4>
<p data-nodeid="49431" class="">图表的初始化配置项，负责图表对象的尺寸、渲染风格、主题样式和图表背景颜色等的设置。首先我们来看一下初始化配置项的详细参数说明，如下图所示：</p>








<p data-nodeid="1939"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcfmAUiFmAADTx0foryY322.png" alt="Drawing 4.png" data-nodeid="2126"></p>
<div data-nodeid="1940"><p style="text-align:center">初始化配置项参数</p></div>
<p data-nodeid="1941">我们重点看一下，Python 程序中是如何配置这些参数的。上述内容出自官方的用户手册说明文档，地址为<a href="https://pyecharts.org/" data-nodeid="2132">https://pyecharts.org/#/zh-cn/global_options</a>，截图如下：</p>
<p data-nodeid="1942"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcgCAOvbqAAFybmUd2nQ904.png" alt="Drawing 5.png" data-nodeid="2136"></p>
<div data-nodeid="1943"><p style="text-align:center">初始化配置项教程</p></div>
<p data-nodeid="1944">接下来，我们通过一个案例，来具体了解初始化配置项的设置。首先来看一下案例执行之后的效果，初始化配置项的设置效果如下所示：</p>
<p data-nodeid="1945"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcgqAfzVvAABzmvkbS_o506.png" alt="Drawing 6.png" data-nodeid="2140"></p>
<div data-nodeid="1946"><p style="text-align:center">初始化配置项示例</p></div>
<p data-nodeid="1947" class="">上图中，我们实现了图片宽度、高度、背景色和主题样式设置。完整的实现代码如下：</p>
<pre class="lang-java" data-nodeid="1948"><code data-language="java">from&nbsp;pyecharts&nbsp;<span class="hljs-keyword">import</span>&nbsp;options&nbsp;as&nbsp;opts
from&nbsp;pyecharts.charts&nbsp;<span class="hljs-keyword">import</span>&nbsp;Bar
from&nbsp;pyecharts.faker&nbsp;<span class="hljs-keyword">import</span>&nbsp;Faker


from&nbsp;pyecharts.globals&nbsp;<span class="hljs-keyword">import</span>&nbsp;ThemeType

c&nbsp;=&nbsp;(
&nbsp;&nbsp;&nbsp;&nbsp;Bar(init_opts=opts.InitOpts(theme=ThemeType.LIGHT,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;width=<span class="hljs-string">"1024px"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;height=<span class="hljs-string">"600px"</span>,

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bg_color=<span class="hljs-string">"#d9d6c3"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;))
&nbsp;&nbsp;&nbsp;&nbsp;.add_xaxis(Faker.choose())
&nbsp;&nbsp;&nbsp;&nbsp;.add_yaxis(<span class="hljs-string">"商家 A"</span>,&nbsp;Faker.values())
&nbsp;&nbsp;&nbsp;&nbsp;.add_yaxis(<span class="hljs-string">"商家 B"</span>,&nbsp;Faker.values())
&nbsp;&nbsp;&nbsp;&nbsp;.render(<span class="hljs-string">"demo_bar_01.html"</span>)
)
</code></pre>
<p data-nodeid="1949">上述代码中，<strong data-nodeid="2151">我们通过 opts.InitOpts 类声明了一个初始化配置项对象 init_opts，初始化配置项参数按照 key = value 的方式，进行了赋值操作</strong>。其中：theme 代表主题样式，可选的范围我会在本课后面的“主题样式”小节进行详细介绍；width 和 height 分别代表图表对象在网页中的宽度和高度；bg_color 代表图表背景颜色，具体的色值，可以根据自己的需要选择。</p>
<h4 data-nodeid="1950">标题配置项</h4>
<p data-nodeid="1951">标题配置项，<strong data-nodeid="2158">负责配置图表对象的标题内容</strong>，包括主标题文本、主标题链接、主标题链接打开方式；副标题文本、副标题链接、副标题链接打开方式；标题元素的位置和边距等。首先我们看一下标题配置项的参数说明，详细的参数说明如下图所示：</p>
<p data-nodeid="1952"><img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9PciqAMBx9AACudY2IHhs755.png" alt="Drawing 7.png" data-nodeid="2161"><br>
<img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9Pci-AbzxbAAC5Gqg11MM625.png" alt="Drawing 8.png" data-nodeid="2165"><br>
<img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcjSAREx0AAC1s26chAk606.png" alt="Drawing 9.png" data-nodeid="2169"></p>
<div data-nodeid="1953"><p style="text-align:center">标题配置项参数</p></div>
<p data-nodeid="1954">关于上述标题配置项的各个参数，图中已经作了详细的注释和说明，我们不做过多的介绍。和初始化配置项一样，我们来看一下 Python 程序中是如何配置的。上述内容出自官方的用户手册说明文档，地址：<a href="https://pyecharts.org/" data-nodeid="2175">https://pyecharts.org/#/zh-cn/global_options</a>，截图如下：</p>
<p data-nodeid="1955"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9Pcj-AH_72AAGWG-ws90U429.png" alt="Drawing 10.png" data-nodeid="2179"></p>
<div data-nodeid="1956"><p style="text-align:center">标题配置项教程</p></div>
<p data-nodeid="1957">标题配置项的主要参数是：标题文本、标题链接、标题跳转方式、副标题文本、副标题链接、副标题跳转方式、标题位置和标题样式，对应的参数名称详见标题配置项参数。接下来，我们通过一个案例，来具体了解标题配置项的设置。首先来看一下案例执行之后的效果，标题栏配置项的设置效果如下所示：</p>
<p data-nodeid="1958"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PckWADH-UAABh1k1kZ80957.png" alt="Drawing 11.png" data-nodeid="2183"></p>
<div data-nodeid="1959"><p style="text-align:center">标题配置项示例</p></div>
<p data-nodeid="52983" class="">上图中，我们实现了主标题文本、主标题链接、主标题打开方式、副标题文本、副标题链接、副标题链接打开方式、标题位置的设置。完整的实现代码如下所示：</p>




<pre class="lang-dart" data-nodeid="1961"><code data-language="dart">from&nbsp;pyecharts&nbsp;<span class="hljs-keyword">import</span>&nbsp;options&nbsp;<span class="hljs-keyword">as</span>&nbsp;opts
from&nbsp;pyecharts.charts&nbsp;<span class="hljs-keyword">import</span>&nbsp;Bar
from&nbsp;pyecharts.faker&nbsp;<span class="hljs-keyword">import</span>&nbsp;Faker


c&nbsp;=&nbsp;(
&nbsp;&nbsp;&nbsp;&nbsp;Bar()
&nbsp;&nbsp;&nbsp;&nbsp;.add_xaxis(Faker.choose())
&nbsp;&nbsp;&nbsp;&nbsp;.add_yaxis(<span class="hljs-string">"商家 A"</span>,&nbsp;Faker.values())
&nbsp;&nbsp;&nbsp;&nbsp;.add_yaxis(<span class="hljs-string">"商家 B"</span>,&nbsp;Faker.values())
&nbsp;&nbsp;&nbsp;&nbsp;.set_global_opts(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;title_opts=opts.TitleOpts(title=<span class="hljs-string">"图表标题"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;title_link=<span class="hljs-string">"www.baidu.com"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;title_target=<span class="hljs-string">"blank"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;subtitle=<span class="hljs-string">"图表副标题"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;subtitle_link=<span class="hljs-string">"www.baidu.com"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;subtitle_target=<span class="hljs-string">"blank"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pos_left=<span class="hljs-string">"left"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;pos_right=<span class="hljs-string">""</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#&nbsp;pos_bottom=<span class="hljs-string">"40"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;padding=[<span class="hljs-number">5</span>,&nbsp;<span class="hljs-number">10</span>,&nbsp;<span class="hljs-number">5</span>,&nbsp;<span class="hljs-number">10</span>])&nbsp;&nbsp;&nbsp;#&nbsp;上右下左
&nbsp;&nbsp;&nbsp;&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;.render(<span class="hljs-string">"demo_bar_02.html"</span>)
)
</code></pre>
<p data-nodeid="1962">上述代码中，<strong data-nodeid="2198">我们通过 opt.TitleOptions 方法声明了一个标题配置项对象，标题配置项参数按照 Key = value 的方式，进行了赋值操作</strong>。其中：title、title_url、title_target 定义了主标题文本、主标题链接、主标题打开方式；subtitle、subtitle_url、subtitle_target 定义了副标题文本、副标题链接和副标题打开方式，其他参数可以参照官方提供的用户手册关于标题配置项的参数进行设定。</p>
<h3 data-nodeid="1963">PyEcharts 图表渲染</h3>
<p data-nodeid="1964">PyEcharts 图表渲染支持两种模式：HTML 页面模式和图片文件模式。默认情况下，PyEcharts 图表渲染，是在当前文件夹下，生成一个名称为：render.html 的 HTML 文件，具体的渲染语句如下：</p>
<pre class="lang-dart" data-nodeid="1965"><code data-language="dart"># 默认模式
bar.render()
</code></pre>
<p data-nodeid="53871" class="">默认模式之外，我们可以给 render( )函数一个包含路径信息的文件名参数，然后生成的图表文件将以该路径为保存路径，以该文件名为名称。具体的渲染语句如下：</p>

<pre class="lang-dart" data-nodeid="1967"><code data-language="dart"># 指定文件名（可以包括路径）
bar.render(<span class="hljs-string">"mycharts.html"</span>)
</code></pre>
<p data-nodeid="1968">以上两种方式生成的都是 HTML 页面，PyEcharts 图表渲染也支持渲染为图片，但渲染为图片的时候需要额外的第三方程序包支持，包名：selenium，可以通过执行：pip install selenium 执行安装。渲染为图片的时候，需要用的函数为：make_snapshot，该函数具体参数如下图所示：</p>
<p data-nodeid="1969"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcnGAXhCUAACVA4qH1nE087.png" alt="Drawing 12.png" data-nodeid="2207"></p>
<div data-nodeid="1970"><p style="text-align:center">Make_snapshot 具体参数</p></div>
<p data-nodeid="1971">Make_snapshot 函数必选参数包括<strong data-nodeid="2219">传入 HTML 文件路径</strong>和<strong data-nodeid="2220">输出图片路径</strong>，我们可以通过一个案例的方式，学习其具体的使用方法。案例源码如下：</p>
<pre class="lang-dart" data-nodeid="1972"><code data-language="dart">from&nbsp;pyecharts.charts&nbsp;<span class="hljs-keyword">import</span>&nbsp;Bar

#&nbsp;导出图片，需要引入以下对象
from&nbsp;pyecharts.render&nbsp;<span class="hljs-keyword">import</span>&nbsp;make_snapshot
from&nbsp;snapshot_selenium&nbsp;<span class="hljs-keyword">import</span>&nbsp;snapshot

bar&nbsp;=&nbsp;Bar()
bar.add_xaxis([<span class="hljs-string">"衬衫"</span>,&nbsp;<span class="hljs-string">"羊毛衫"</span>,&nbsp;<span class="hljs-string">"雪纺衫"</span>,&nbsp;<span class="hljs-string">"裤子"</span>,&nbsp;<span class="hljs-string">"高跟鞋"</span>,&nbsp;<span class="hljs-string">"袜子"</span>])
bar.add_yaxis(<span class="hljs-string">"商家 A"</span>,&nbsp;[<span class="hljs-number">5</span>,&nbsp;<span class="hljs-number">20</span>,&nbsp;<span class="hljs-number">36</span>,&nbsp;<span class="hljs-number">10</span>,&nbsp;<span class="hljs-number">75</span>,&nbsp;<span class="hljs-number">90</span>])
#&nbsp;render&nbsp;会生成本地&nbsp;HTML&nbsp;文件，默认会在当前目录生成&nbsp;render.html&nbsp;文件
#&nbsp;也可以传入路径参数，如&nbsp;bar.render(<span class="hljs-string">"mycharts.html"</span>)
#&nbsp;默认模式
bar.render()
#&nbsp;指定路径
bar.render(<span class="hljs-string">"mycharts.html"</span>)

#&nbsp;渲染成图片&nbsp;
make_snapshot(snapshot,&nbsp;bar.render(),&nbsp;<span class="hljs-string">"bar0.png"</span>)
</code></pre>
<p data-nodeid="1973">上述程序演示了 PyEcharts 的 3 种渲染方式：默认情况下生成 render.html 文件；设置参数的情况下，在指定目录下，生成了 mycharts.html 文件；调用函数 make_snapshot 生成了图片文件 bar0.png。</p>
<h3 data-nodeid="1974">PyEcharts 主题样式</h3>
<p data-nodeid="1975">PyEcharts 提供了超过 15 种的主题样式，具体的样式数量会因为版本的不同而有差异，你可以通过主题样式的源码文件查看，具体的源码文件地址为：<a href="https://github.com/pyecharts/pyecharts/blob/master/pyecharts/globals.py" data-nodeid="2228">https://github.com/pyecharts/pyecharts/blob/master/pyecharts/globals.py</a>。该文件定义了 PyEcharts 当前版本支持的主题样式的名字，具体主题样式类型，以该文件内容为准。我们当前使用的版本 v1.6.2，支持的主题样式共 15 种，具体的样式名称如下图所示：</p>
<p data-nodeid="1976"><img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9Pco6APexSAABw92QvSQU319.png" alt="Drawing 13.png" data-nodeid="2232"></p>
<div data-nodeid="1977"><p style="text-align:center">主题样式类型</p></div>
<p data-nodeid="1978">每一种主题样式的图表效果，可以在 PyEcharts 实例程序的官方站点中查看，地址为<a href="https://pyecharts.org/#/zh-cn/themes" data-nodeid="2236">https://pyecharts.org/#/zh-cn/themes</a>。<strong data-nodeid="2242">主题样式可以根据自己的业务场景和个人偏好选择，但要遵循一个基本的原则：美观大方、颜色鲜明、对比清晰</strong>。我个人比较喜欢的几个主题展示图表展示效果如下：</p>
<p data-nodeid="1979"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9PcqWAJ8uvAACYnhCl0aU057.png" alt="Drawing 14.png" data-nodeid="2245"></p>
<div data-nodeid="1980"><p style="text-align:center">主题样式-Light</p></div>
<p data-nodeid="1981"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9Pcq2AENcdAACQycRa3TE148.png" alt="Drawing 15.png" data-nodeid="2248"></p>
<div data-nodeid="1982"><p style="text-align:center">主题样式-Dark</p></div>
<p data-nodeid="1983"><img src="https://s0.lgstatic.com/i/image/M00/49/9E/Ciqc1F9PcraAUqKRAACTP4H83E0188.png" alt="Drawing 16.png" data-nodeid="2251"></p>
<div data-nodeid="1984"><p style="text-align:center">主题样式-chalk</p></div>
<p data-nodeid="1985"><img src="https://s0.lgstatic.com/i/image/M00/49/A9/CgqCHl9Pcr2ABeg7AACxu160ElQ044.png" alt="Drawing 17.png" data-nodeid="2254"></p>
<div data-nodeid="1986"><p style="text-align:center">主题样式-shine</p></div>
<p data-nodeid="1987">了解了主题样式的类型，结合上文中讲到的配置参数，你就可以设置自己喜欢的主题样式了。</p>
<h3 data-nodeid="1988">小结</h3>
<p data-nodeid="1989">本节我们介绍了基于 PyEcharts 图表可视化框架进行图表开发快速入门相关的知识，内容包括 PyEcharts 图表类型、图表配置、图表渲染和主题样式。</p>
<p data-nodeid="1990"><strong data-nodeid="2274">图表类型部分</strong>，我介绍了 PyEcharts 支持的图表类型清单，同时介绍了各种图表类的导入方法；<strong data-nodeid="2275">图表配置部分</strong>，我重点介绍了初始化配置项和标题配置项的接口说明和配置方法，希望你能够掌握配置的思路，结合官方文档，触类旁通，能够掌握其他配置项的使用方法；<strong data-nodeid="2276">图表渲染部分</strong>，我介绍 PyEcharts 图表对象渲染的两类输出，三种方法；<strong data-nodeid="2277">主题样式部分</strong>，则是介绍了 PyEcharts 支持的主题样式清单和设置方法。</p>
<p data-nodeid="1991">在课程的最后，我会展现一些共性的问题，希望能对你学习本门课程有所帮助。</p>
<p data-nodeid="1992">有位学员问我该如何着手搭建一个可视化报表，日常又应该如何去训练这个技能。我想也许不止他一个人有这样的疑惑，所以我把我的回答放在了这里，希望有同样疑惑的同学也能看到。</p>
<p data-nodeid="1993"><strong data-nodeid="2283">这里面涉及的是一个工作思路和方法论的问题，即如何构建一个企业级的数据报表体系。需要具备的完整知识体系包括几个方面。</strong></p>
<ol data-nodeid="1994">
<li data-nodeid="1995">
<p data-nodeid="1996"><strong data-nodeid="2287">业务建模：业务过程、业务逻辑、业务活动（通过业务建模，了解到底哪些业务主题需要构建可视化的报表）；</strong></p>
</li>
<li data-nodeid="1997">
<p data-nodeid="1998"><strong data-nodeid="2291">指标定义：明确各个业务活动，需要哪些衡量指标（例如：业绩指标、人效指标、财务指标等）；</strong></p>
</li>
<li data-nodeid="1999">
<p data-nodeid="2000"><strong data-nodeid="2295">维度定义：时间维度、对比维度、分布维度、组成维度等；</strong></p>
</li>
<li data-nodeid="2001">
<p data-nodeid="2002"><strong data-nodeid="2299">创建可视化报表；</strong></p>
</li>
<li data-nodeid="2003">
<p data-nodeid="2004"><strong data-nodeid="2303">分析和洞察。</strong></p>
</li>
</ol>
<p data-nodeid="2005" class="">以上内容我们会在课程“<strong data-nodeid="2309">模块三：典型案例篇</strong>”详细介绍，敬请期待。</p>

---

### 精选评论


