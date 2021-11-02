<h3 data-nodeid="421077" class="">概述</h3>
<p data-nodeid="421078">上一节，我基于 PyEcharts 的官方案例，学习了 PyEcharts 与 Flask 整合的两种方法和数据刷新的两种实现机制。本节我会结合<strong data-nodeid="421169">模块三：典型案例篇</strong>中的实际案例，带你了解如何基于 PyEcharts + Flask + Bootstrap，生成一个完整的数据可视化系统。本节内容的知识结构如下图所示：</p>
<p data-nodeid="421079"><img src="https://s0.lgstatic.com/i/image/M00/57/FF/Ciqc1F9t0oqAeQ8MAADMbEXfveE881.png" alt="Drawing 0.png" data-nodeid="421172"></p>
<div data-nodeid="421080"><p style="text-align:center">图 1：章节知识结构</p></div>
<p data-nodeid="421081">PyEcharts 与 Flask 框架整合实战案例的介绍，我将其分为两部分，前端页面设计和后台服务设计，其中又各有 4 项内容。</p>
<p data-nodeid="421082">通过上一个小节，我们了解到 PyEcharts 与 Flask 的整合方法，有前端混合模式和前后端分离模式两种。本节内容，我会采用前后端分离模式，带你生成一个完整的数据可视化系统。</p>
<p data-nodeid="421083">前端页面设计部分，我们需要解决的问题是主题模板选择、导航菜单设计、图表元素设计和图表事件设计；后台应用设计部分，我们需要完成的操作包括：服务接口设计、页面请求响应设计、数据请求响应设计和异常请求处理设计。</p>
<h3 data-nodeid="421084">运行演示</h3>
<p data-nodeid="421085">PyEcharts 与 Flask 框架整合以后，结合<strong data-nodeid="421182">模块三</strong>的实战案例，我们可以整合出一个完整的数据可视化系统。系统运行以后的效果如下图所示：</p>
<p data-nodeid="421086"><img src="https://s0.lgstatic.com/i/image/M00/58/0B/CgqCHl9t0pOAXZIIAARpY9Abtz8653.png" alt="Drawing 1.png" data-nodeid="421185"></p>
<div data-nodeid="421087"><p style="text-align:center">图 2：客户地理位置分布图</p></div>
<p data-nodeid="421088"><img src="https://s0.lgstatic.com/i/image/M00/57/FF/Ciqc1F9t0pqAFZaeAAHS9Y5AZP4036.png" alt="Drawing 2.png" data-nodeid="421188"></p>
<div data-nodeid="421089"><p style="text-align:center">图 3：历史数据变化趋势</p></div>
<p data-nodeid="421090"><img src="https://s0.lgstatic.com/i/image/M00/58/0B/CgqCHl9t00SAaa7dAAIXyQzJeAA583.png" alt="Drawing 3.png" data-nodeid="421191"></p>
<div data-nodeid="421091"><p style="text-align:center">图 4：订单商品构成模型</p></div>
<p data-nodeid="421092">上述截图分别呈现了实时指标监控、历史数据变化趋势、客户地理位置分布和订单商品构成。我们通过一个页面导航的方式，把它们组织在一起，形成了一个完整的数据可视化系统。接下来，我将带你学习如何实现这个数据可视化系统。</p>
<h3 data-nodeid="421093">源码结构</h3>
<p data-nodeid="421094">数据可视化分析系统的开发过程，我会采用前后端分离的开发模式，融合<strong data-nodeid="421199">模块三</strong>的 6 个实战案例。其完整的源码结构如下图所示：</p>
<p data-nodeid="421095"><img src="https://s0.lgstatic.com/i/image/M00/58/0B/CgqCHl9t1CSAY_xSAAIMAyI1xXo853.png" alt="Drawing 4.png" data-nodeid="421202"></p>
<div data-nodeid="421096"><p style="text-align:center">图 5：数据可视化系统源码结构</p></div>
<p data-nodeid="421097">如上图所示，整个数据可视化系统的源码位于 Flask 项目下，文件夹包括：apps、data、model、static 和 templates。其中 apps 是业务逻辑处理模块文件目录，data 是数据库操作脚本文件目录，model 是数据库查询模块文件目录，static 是资源文件目录，templates 是模板文件目录。</p>
<h3 data-nodeid="421098">操作流程</h3>
<p data-nodeid="421099">数据可视化系统的开发流程包括<strong data-nodeid="421226">创建一个空白项目</strong>，<strong data-nodeid="421227">复制 PyEcharts 模板文件到项目 templates 文件目录</strong>，<strong data-nodeid="421228">前端页面设计</strong>，<strong data-nodeid="421229">后台应用设计</strong>和<strong data-nodeid="421230">前后端联调</strong>五大步骤。</p>
<p data-nodeid="421100">完成一个图表页面的开发之后，我们会重新回到前端页面设计环节，通过循环实现新的图表页面。数据可视化系统完整的开发流程，如下图所示：</p>
<p data-nodeid="421101"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1CuAflZ3AABiPQa-c3o820.png" alt="Drawing 5.png" data-nodeid="421234"></p>
<div data-nodeid="421102"><p style="text-align:center">图 6：数据可视化系统开发流程</p></div>
<p data-nodeid="421103">我们可以看到，数据可视化系统开发的流程首先需要创建一个 Flask 项目，并且创建相应的文件目录结构。合理的源码组织，有助于我们管理源码、理解源码，Flask 项目的创建和模块四的第一小节：<strong data-nodeid="421242">13 | Flask Web 框架基础</strong>保持一致。具体的创建过程，可以参考该节的内容。</p>
<p data-nodeid="421104">创建完空白 Flask 项目之后，我们需要复制 PyEcharts 的模板文件到 Flask 项目的 templates 文件夹。具体的方法我在“<strong data-nodeid="421252">14 | PyEcharts &amp; Flask 框架集成</strong>”中已经做过详细的介绍。</p>
<p data-nodeid="421105">完成 PyEcharts 模板复制之后，我们接下来需要做的操作是，前端页面的设计和后台应用的设计，设计方法参考“<strong data-nodeid="421258">14 课时</strong>”的 PyEcharts 与 Flask 整合前后端分离模式的操作方法。</p>
<p data-nodeid="421106">前端页面程序和后台应用程序设计完成以后，我们需要进行前后端的联合调试，一方面确保数据接口的联通，另外一方面验证功能和数据的正确性。</p>
<h3 data-nodeid="421107">前端页面设计</h3>
<h4 data-nodeid="421108">主题模板选择</h4>
<p data-nodeid="421109"><strong data-nodeid="421266">构建基于网页的 Web 类应用系统，如果有漂亮、美观的界面，主题分明的配色方案，清晰明了的内容组织，可以给观看者带来良好的用户体验</strong>。</p>
<p data-nodeid="421110"><strong data-nodeid="421271">为了实现这样的效果，我们需要引入主题模板</strong>。主题模板定义了页面的组件元素、样式和配色。本案例中，为了实现比较美观的呈现效果，我选择 Bootstrap 的主题样式模板：Matrix Admin 开源免费版本。其官方网站和下载地址如下图所示：</p>
<p data-nodeid="421111"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1DWAReUqAAdhi5bhA3I724.png" alt="Drawing 6.png" data-nodeid="421274"></p>
<div data-nodeid="421112"><p style="text-align:center">图 7：Bootstrap 主题样式文件</p></div>
<p data-nodeid="421113">Matrix Admin 分为开源版本和商业版本，开源版本的下载地址为：<a href="http://matrixadmin.wrappixel.com/matrix-admin-package-full.zip" data-nodeid="421278">http://matrixadmin.wrappixel.com/matrix-admin-package-full.zip</a>。Matrix Admin 文件解压以后的目录结构如下图所示：</p>
<p data-nodeid="421114"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1D6AaVr8AAIkElprB_0784.png" alt="Drawing 7.png" data-nodeid="421282"></p>
<div data-nodeid="421115"><p style="text-align:center">图 8：Matrix Admin 文件目录结构</p></div>
<p data-nodeid="421116">上图中左侧是 Matrix Admin 的文件目录，共分为 3 个文件夹：asserts、dist 和 html。其中 asserts 是第三方资源依赖文件目录，dist 存储的是页面资源文件，html 存储的是示例程序。Matrix Admin 示例程序可以直接通过浏览器查看，运行效果如下：</p>
<p data-nodeid="421117"><img src="https://s0.lgstatic.com/i/image/M00/58/0B/CgqCHl9t1EaAE6haAAKBX2F6D24688.png" alt="Drawing 8.png" data-nodeid="421286"></p>
<div data-nodeid="421118"><p style="text-align:center">图 9：Matrix Admin 运行演示</p></div>
<p data-nodeid="421119">上述示例程序展现了该主题模板支持的页面元素、配色方案和主题样式。我们可以通过复用其页面元素、配色方案和主题样式，结合 PyEcharts 图表设置，设计我们自己的数据可视化系统。</p>
<h4 data-nodeid="421120">页面导航设计</h4>
<p data-nodeid="421121"><strong data-nodeid="421293">页面导航用于在各个业务模块之间进行内容切换，是多页面内容组织的一种有效方式。页面导航栏的使用，可以有效地将页面内容按照类型分组</strong>。</p>
<p data-nodeid="421122">数据可视化系统中，按照图表类型组织内容，除实时监控数据指标卡外，一个图表类型对应一个业务场景。设计完成之后的页面导航栏，如下图所示：</p>
<p data-nodeid="421123"><img src="https://s0.lgstatic.com/i/image/M00/58/0B/CgqCHl9t1EyAaGtgAAAe4uD0ucs240.png" alt="Drawing 9.png" data-nodeid="421297"></p>
<div data-nodeid="421124"><p style="text-align:center">图 10：数据可视化页面导航栏</p></div>
<p data-nodeid="428209" class="">页面导航栏用到了 Matrix Admin 主题模板中的页面元素组件：<code data-backticks="1" data-nodeid="428211">&lt;Nav&gt;&lt;/Nav&gt;</code>。实现上述页面导航栏的源码如下所示：</p>
<nav></nav>。实现上述页面导航栏的源码如下所示：<p></p>
<nav>&lt;\Nav&gt;。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>&lt;\Nav。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>&lt;\Na。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>&lt;\N。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>&lt;\。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>&lt;。实现上述页面导航栏的源码如下所示：<p></p>
</nav><nav>。实现上述页面导航栏的源码如下所示：<p></p>
</nav>




<p data-nodeid="421126"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1FSAcEBSAAErXPpLtj4657.png" alt="Drawing 10.png" data-nodeid="421301"></p>
<div data-nodeid="421127"><p style="text-align:center">图 11：导航栏页面实现源码</p></div>
<p data-nodeid="421128">上述代码，声明了一个页面导航栏，共包括 6 个导航菜单项，每一个导航菜单项有相应的主题样式、超级链接地址、菜单标题和折叠状态。</p>
<h4 data-nodeid="421129">图表元素设计</h4>
<p data-nodeid="421130">完成导航栏菜单的设计之后，我们接下来看具体的图表页面的设计。<strong data-nodeid="421309">图表页面的设计包括页面元素设计和图表事件设计</strong>，我们先来看一下图表元素设计。以历史数据变化趋势图为例，图表元素设计的实现代码如下所示：</p>
<pre class="lang-js" data-nodeid="421131"><code data-language="js">&lt;!-- 图表可视化组件 --&gt;
&lt;!--================================================== --&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"row"</span>  &gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"col-md-12"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"card"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"card-body"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"row"</span>&gt;</span>
                    <span class="hljs-comment">&lt;!-- column --&gt;</span>
                    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"col-lg-12"</span>&gt;</span>
                        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"flot-chart"</span> &gt;</span>
                            <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"flot-chart-content"</span> <span class="hljs-attr">style</span>=<span class="hljs-string">"text-align:center;"</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"map"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
                        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
                    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
                    <span class="hljs-comment">&lt;!-- column --&gt;</span>
                <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
            <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
</code></pre>
<p data-nodeid="421132">上述代码，声明了一个带有 ID 的占位符 div 元素，该元素通过图表事件，实现了与 PyEcharts 图表对象的绑定。</p>
<h4 data-nodeid="421133">图表事件设计</h4>
<p data-nodeid="421134"><strong data-nodeid="421316">定义完图表元素之后，我们接下来需要设计的是页面图表响应事件</strong>。通过图表响应事件，我们实现了页面图表元素和 PyEcharts 图表对象之间的绑定，并且为图表对象的参数获取设置了远程访问接口。历史数据变化趋势图的图表事件设计源码如下所示：</p>
<pre class="lang-js" data-nodeid="421135"><code data-language="js">&lt;!-- 历史数据变化趋势图 --&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
    $(
        <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">var</span> chart = echarts.init(<span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'map'</span>), <span class="hljs-string">'white'</span>, {<span class="hljs-attr">renderer</span>: <span class="hljs-string">'canvas'</span>});
            $.ajax({
                <span class="hljs-attr">type</span>: <span class="hljs-string">"GET"</span>,
                <span class="hljs-attr">url</span>: <span class="hljs-string">"http://127.0.0.1:5000/histChart"</span>,
                <span class="hljs-attr">dataType</span>: <span class="hljs-string">'json'</span>,
                <span class="hljs-attr">success</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">result</span>) </span>{
                    chart.setOption(result);
                }
            });
        }
    )
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>
<p data-nodeid="421136">图表事件源码实现了图表对象和页面指定 ID 元素的绑定，并且通过 ajax 技术，调用服务器端接口程序：http://127.0.0.1:5000/histChart，实现了与服务器端程序的图表配置参数数据交互。</p>
<h3 data-nodeid="421137">后台服务设计</h3>
<h4 data-nodeid="421138">服务接口设计</h4>
<p data-nodeid="421139"><strong data-nodeid="421324">服务接口设计包括页面请求接口和数据请求接口，页面请求接口是浏览器对应的该页面的访问地址，数据请求接口对应的是图表对象的数据请求地址</strong>。如下图所示：</p>
<p data-nodeid="421140"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1G-AeJBpAABgo0D0p8k237.png" alt="Drawing 11.png" data-nodeid="421327"></p>
<div data-nodeid="421141"><p style="text-align:center">图 12：服务接口类型</p></div>
<h4 data-nodeid="421142">页面请求接口设计</h4>
<p data-nodeid="421143">历史数据变化趋势图的页面请求接口定义如下：</p>
<pre class="lang-dart" data-nodeid="421144"><code data-language="dart"># <span class="hljs-number">01.</span> 页面数据请求服务
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/"</span>)
def index():
    cur = rt_index_base()
    # <span class="hljs-keyword">return</span> render_template(<span class="hljs-string">"dashboard.html"</span>, curnumber=cur)
    <span class="hljs-keyword">return</span> render_template(<span class="hljs-string">"index3.html"</span>, curnumber=cur)
</code></pre>
<p data-nodeid="421145">上述程序设置页面请求路由的同时，定义了该页面请求的业务响应逻辑，通过调用实时数据监控指标查询函数，获取实时数据监控指标，并以参数的形式传递给页面渲染函数，从而进行页面渲染。</p>
<h4 data-nodeid="421146">数据请求接口设计</h4>
<p data-nodeid="421147">数据请求接口设计，同样以历史数据变化趋势图为例，其数据请求接口设计源码如下所示：</p>
<pre class="lang-dart" data-nodeid="421148"><code data-language="dart"># <span class="hljs-number">02.</span> 图表数据查询
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/histChart"</span>)
def get_hist_order_chart():
    c = hist_order_base()
    <span class="hljs-keyword">return</span> c.dump_options_with_quotes()
</code></pre>
<p data-nodeid="421149">上述程序定义了一个图表数据查询接口，包括请求响应地址和业务响应逻辑两部分内容。请求响应地址与前端页面设计部分的图表事件设计相对应，具体的数据查询函数名称为hist_order_base()，返回的内容为图表对象的配置参数，包括数据部分和非数据部分。其函数实现如下所示：</p>
<pre class="lang-dart" data-nodeid="421150"><code data-language="dart"># <span class="hljs-number">01.</span> 订单量历史数据变化趋势
def hist_order_base():
&nbsp;&nbsp;&nbsp; dataX, dataY&nbsp; = order_sum_query()
&nbsp;&nbsp;&nbsp; # 订单数据
&nbsp;&nbsp;&nbsp; c = (
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Line(init_opts=opts.InitOpts(theme=ThemeType.LIGHT))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .add_xaxis(dataX)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .add_yaxis(<span class="hljs-string">"订单量"</span>, dataY, is_smooth=True)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .set_global_opts(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; title_opts=opts.TitleOpts(title=<span class="hljs-string">"日订单量历史数据趋势图"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; yaxis_opts=opts.AxisOpts(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; type_=<span class="hljs-string">"value"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; axistick_opts=opts.AxisTickOpts(is_show=True),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; splitline_opts=opts.SplitLineOpts(is_show=True),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xaxis_opts=opts.AxisOpts(type_=<span class="hljs-string">"category"</span>, boundary_gap=False),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;)
&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> c
</code></pre>
<p data-nodeid="421151">上述程序实现了图表对象的参数设置，并且调用数据库查询函数，获取了订单历史数据，其实现函数为order_sum_query()。该函数从数据库中读取数据内容，并返回给调用函数。该函数的代码实现如下所示：</p>
<pre class="lang-dart" data-nodeid="421152"><code data-language="dart"># 交易量查询
def order_sum_query():
&nbsp; &nbsp; # 连接到数据库
&nbsp; &nbsp; connection = pymysql.connect(host=<span class="hljs-string">'localhost'</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;port=<span class="hljs-number">3306</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;user=<span class="hljs-string">'root'</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;password=<span class="hljs-string">'密码'</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;db=<span class="hljs-string">'sakila'</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;charset=<span class="hljs-string">'utf8'</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cursorclass=pymysql.cursors.DictCursor)
&nbsp; &nbsp; <span class="hljs-keyword">try</span>:
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">with</span> connection.cursor() <span class="hljs-keyword">as</span> cursor:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # SQL 查询语句：简化程序只返回第一行
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sql = <span class="hljs-string">"SELECT * FROM dm_rental_day ORDER BY 日期 DESC limit 1 "</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span>:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # 执行SQL语句，返回影响的行数
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; row_count = cursor.execute(sql)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # 获取所有记录列表
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; results = cursor.fetchall()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> row <span class="hljs-keyword">in</span> results:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # 此处不可以用索引访问：row[<span class="hljs-number">0</span>]
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; orderdate = row[<span class="hljs-string">"日期"</span>]
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ordersum = row[<span class="hljs-string">"订单量"</span>]
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; # 打印结果
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">"日期：%s,交易额：%d"</span> % (orderdate, ordersum))
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ordersum
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; except:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">"错误：数据查询操作失败"</span>)
&nbsp; &nbsp; <span class="hljs-keyword">finally</span>:
&nbsp; &nbsp; &nbsp; &nbsp; connection.close()
</code></pre>
<h4 data-nodeid="421153">异常处理接口设计</h4>
<p data-nodeid="421154">一个线上系统，在某些特定的情况下，会出现一些异常，比如网页不存在、网络不可用等特殊情况。默认浏览器提供的异常页面并不友好，原因是无法准确地反馈给用户具体的错误原因。因此，针对常见错误和异常，我们需要自己设计异常提示程序。</p>
<p data-nodeid="421155">Flask 提供了常见异常捕获的函数和方法，通过调用该函数，可以自定义指定错误码的响应程序。具体的调用方式如下所示：</p>
<pre class="lang-dart" data-nodeid="421156"><code data-language="dart"># <span class="hljs-number">403</span> 错误
<span class="hljs-meta">@app</span>.errorhandler(<span class="hljs-number">403</span>)
def miss(e):
<span class="hljs-keyword">return</span> render_template(<span class="hljs-string">'error-403.html'</span>), <span class="hljs-number">404</span>

# <span class="hljs-number">404</span> 错误
<span class="hljs-meta">@app</span>.errorhandler(<span class="hljs-number">404</span>)
def error(e):
<span class="hljs-keyword">return</span> render_template(<span class="hljs-string">'error-404.html'</span>), <span class="hljs-number">500</span>

# <span class="hljs-number">500</span> 错误
<span class="hljs-meta">@app</span>.errorhandler(<span class="hljs-number">500</span>)
def error(e):
<span class="hljs-keyword">return</span> render_template(<span class="hljs-string">'error-500.html'</span>), <span class="hljs-number">500</span>
</code></pre>
<p data-nodeid="421157">上述程序中，我们通过在函数声明前添加装饰器的调用，实现了对于异常错误码的捕获，并且绑定了相应的异常处理函数。当对应错误码的网络访问异常发生时，会调用对应的异常处理函数，渲染对应的异常提示页面。一个自定义的异常提示页面如下图所示：</p>
<p data-nodeid="421158"><img src="https://s0.lgstatic.com/i/image/M00/58/00/Ciqc1F9t1JiAWmznAABqehHCrhA181.png" alt="Drawing 12.png" data-nodeid="421349"></p>
<div data-nodeid="421159"><p style="text-align:center">图 13：自定义异常处理</p></div>
<h3 data-nodeid="421160">小结</h3>
<p data-nodeid="421161">本小节，我结合了前面全部的内容，并整合了相关案例，基于 Python + Flask + PyEcharts + Bootstrap 生成了一个完整的数据可视化系统。因为大部分内容我都介绍过，因此我把重点放在了如何串联各个知识点上。</p>
<p data-nodeid="421162" class="">我通过一个实例：历史数据变化趋势图页面设计，讲解了一个完整的数据可视化系统的设计流程，你可以通过这个范例，举一反三，生成其他内容。完整的项目源码，我放到了开源的代码托管平台 Github 上，具体地址：<a href="https://github.com/zhangziliang04/datastudio" data-nodeid="421355">https://github.com/zhangziliang04/datastudio</a>，你可以自行下载学习。</p>

---

### 精选评论


