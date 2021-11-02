<h3 data-nodeid="75555" class="">概述</h3>
<p data-nodeid="75556">上一节，我介绍了“<strong data-nodeid="75668">模块四：数据发布篇</strong>”的第一课，Python Flask Web 框架基础理论，带你了解了 Flask 框架的主要特性、源码资源、安装部署和基本使用方法。</p>
<p data-nodeid="78483">接下来，我会介绍该部分的第二节，<strong data-nodeid="78492">PyEcharts &amp; Flask 框架集成</strong>，包括 PyEcharts 与 Flask 框架整合的两种方法和数据刷新机制。完整的知识结构如下图所示：</p>
<p data-nodeid="78484"><img src="https://s0.lgstatic.com/i/image/M00/58/E8/CgqCHl9wUPeAPJcNAACYrxSbijo289.png" alt="图片2.png" data-nodeid="78495"></p>






<div data-nodeid="76641"><p style="text-align:center">图 1：章节知识结构图</p></div>




<p data-nodeid="75560">PyEcharts 与 Flask 整合的方式有两种：前后端混合模式和前后端分离模式。</p>
<p data-nodeid="75561"><strong data-nodeid="75689">前后端混合模式</strong>是指前端页面设计和后台服务响应设计糅合在一起，页面内容的渲染由后台程序控制；<strong data-nodeid="75690">前后端分离模式</strong>是指前端页面设计和后台服务响应设计解耦，各自独立开发，互不干扰，最后通过接口调用的方式，进行数据交互。</p>
<p data-nodeid="75562">PyEcharts 和 Flask 整合后，发布的 Web 应用涉及数据刷新机制的问题，目前支持的方式包括两种：定时全量刷新和定时增量刷新。</p>
<p data-nodeid="75563"><strong data-nodeid="75700">定时全量刷新</strong>是指 PyEcharts 图表数据，执行定时全量的重新读取和图表内容的重新渲染；<strong data-nodeid="75701">定时增量刷新</strong>是指 PyEcharts 图表数据，执行定时指定数据项的重新读取和图表项的重新渲染。</p>
<p data-nodeid="75564">我会在下面对这 4 个内容一一介绍。</p>
<h3 data-nodeid="75565">前后端混合模式</h3>
<h4 data-nodeid="75566">操作流程</h4>
<p data-nodeid="75567">PyEcharts 与 Flask 框架整合的前后端混合模式，其操作流程如下图所示：</p>
<p data-nodeid="79958"><img src="https://s0.lgstatic.com/i/image/M00/58/E8/CgqCHl9wUQKALX3FAACHyVpi0P8593.png" alt="图片3.png" data-nodeid="79961"></p>

<div data-nodeid="79222"><p style="text-align:center">图 2：前后端混合模式操作流程</p></div>




<p data-nodeid="75570"><strong data-nodeid="75721">创建项目环节</strong>会创建一个空白的 Flask 项目，<strong data-nodeid="75722">复制模板环节</strong>会复制 PyEcharts 模板文件到 Flask 项目模板文件目录 templates，<strong data-nodeid="75723">渲染图表环节</strong>通过后台程序，基于 PyEcharts 模板文件，渲染可视化图表。</p>
<p data-nodeid="75571">我们依次来看。</p>
<h4 data-nodeid="75572">创建项目</h4>
<p data-nodeid="75573"><strong data-nodeid="75734">创建项目的方式同普通 Flask 项目的创建一样，我们只需要创建一个空白的文件夹即可</strong>。<strong data-nodeid="75735">考虑到模板的默认访问目录，我们需要在空白文件夹下，创建一个 templates 文件夹，用来存放 PyEcharts 模板文件</strong>。创建项目完成之后的文件结构如下图所示：</p>
<p data-nodeid="75574"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0LKAOvTJAAA0f798Ivw655.png" alt="Drawing 2.png" data-nodeid="75738"></p>
<div data-nodeid="75575"><p style="text-align:center">图 3：创建空白项目</p></div>
<h4 data-nodeid="75576">复制模板</h4>
<p data-nodeid="75577"><strong data-nodeid="75744">复制模板指复制 PyEcharts 模板文件到新创建的 Flask 空白项目模板文件夹中</strong>。具体的操作方式分为 2 个步骤：</p>
<ol data-nodeid="75578">
<li data-nodeid="75579">
<p data-nodeid="75580">找到 PyEcharts 模板文件；</p>
</li>
<li data-nodeid="75581">
<p data-nodeid="75582">复制 PyEcharts 模板文件到 Flask 空白项目的 templates 文件夹。</p>
</li>
</ol>
<p data-nodeid="75583">查找 PyEcharts 模板文件的路径，可以通过 pip show pyecharts 指令，查询其安装位置。具体的操作指令执行界面如下图所示：</p>
<p data-nodeid="75584"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0L-ANHELAABvlvueCuo798.png" alt="Drawing 3.png" data-nodeid="75750"></p>
<div data-nodeid="75585"><p style="text-align:center">图 4：PyEcharts 安装目录查询</p></div>
<p data-nodeid="75586">通过上述指令我们可以看到 PyEcharts 的安装目录位于：d:\program files\python36\lib\site-packages，我们可以通过文件管理器打开该目录。对应的模板文件如下图所示：</p>
<p data-nodeid="75587"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0MmAaIFPAAC6W0usm8o339.png" alt="Drawing 4.png" data-nodeid="75762"></p>
<div data-nodeid="75588"><p style="text-align:center">图 5：PyEcharts 模板文件</p></div>
<p data-nodeid="75589">复制上述文件到新创建的 Flask 项目模板文件夹下，默认文件夹名称为 templates。复制后的目录结构如下图所示：</p>
<p data-nodeid="75590"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0NOAPVSaAAC36DYwtFU846.png" alt="Drawing 5.png" data-nodeid="75766"></p>
<div data-nodeid="75591"><p style="text-align:center">图 6：项目模板文件</p></div>
<h4 data-nodeid="75592">渲染图表</h4>
<p data-nodeid="75593"><strong data-nodeid="75772">前后端混合模式的图表渲染，是 Flask 应用创建过程和 PyEcharts 图表创建过程的整合</strong>。一个简单的示例程序如下：</p>
<p data-nodeid="75594"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0OCAesZIAADKoeb566s443.png" alt="Drawing 6.png" data-nodeid="75775"></p>
<div data-nodeid="75595"><p style="text-align:center">图 7：渲染图表</p></div>
<p data-nodeid="75596">上图中，首先创建了一个 Flask 应用对象，设定了默认模板目录为 templates，然后定义了一个图表生成函数，接下来通过业务响应函数 index，调用图表生成函数 bar_base()，最后通过 Flask 函数 Markup()进行图表渲染。</p>
<h4 data-nodeid="75597">路由设置</h4>
<p data-nodeid="75598">PyEcharts 与 Flask 整合之后的路由设置方式，完全采用的是 Flask 的路由设置方式，没有任何区别。相关内容参见“<strong data-nodeid="75787">13 | Flask Web 框架基础</strong>”。</p>
<h4 data-nodeid="75599">运行演示</h4>
<p data-nodeid="75600">PyEcharts 与 Flask 整合之后，可以通过启动 Flask 应用程序的方式直接启动服务，然后通过浏览器进行访问。上述示例程序运行效果如下图所示：</p>
<p data-nodeid="75601"><img src="https://s0.lgstatic.com/i/image/M00/57/FE/Ciqc1F9t0OuAUTKCAACTL74lAV8174.png" alt="Drawing 7.png" data-nodeid="75792"></p>
<div data-nodeid="75602"><p style="text-align:center">图 8：PyEcharts 与 Flask 整合案例运行演示</p></div>
<p data-nodeid="75603">通过上述内容，我们可以看到 PyEcharts 与 Flask 框架整合，影响到只是业务响应环节，在这一部分，进行 PyEcharts 图表生成，并调用 Flask 模板渲染函数进行页面渲染，其他部分相比标准得 Flask 应用程序，没有任何区别。</p>
<h3 data-nodeid="75604">前后端分离模式</h3>
<h4 data-nodeid="75605">操作流程</h4>
<p data-nodeid="82874">PyEcharts 与 Flask 框架整合的前后端分离模式，其操作流程如下图所示：</p>
<p data-nodeid="82875"><img src="https://s0.lgstatic.com/i/image/M00/58/DD/Ciqc1F9wURCAXaZXAACmB2qprw8064.png" alt="图片4.png" data-nodeid="82879"></p>





<div data-nodeid="81048"><p style="text-align:center">图 9：前后端分离模式</p></div>




<p data-nodeid="75609"><strong data-nodeid="75822">创建项目环节</strong>会创建一个空白的 Flask 项目，<strong data-nodeid="75823">复制模板环节</strong>会复制 PyEcharts 模板文件到 Flask 项目模板文件目录 templates**，前端页面设计环节**设计需要展示的页面布局，<strong data-nodeid="75824">图表渲染环节</strong>配置 PyEcharts 图表参数，<strong data-nodeid="75825">路由设计环节</strong>设计客户端请求路由。</p>
<p data-nodeid="75610">创建路由和复制模板环节的操作方式与前后端混合模式完全一样，我就不再赘述，接下来的介绍会从前端页面设计开始。</p>
<h4 data-nodeid="75611">前端页面设计</h4>
<p data-nodeid="75612"><strong data-nodeid="75832">前端页面设计导入 Echarts 图表库文件，声明图表对象占位符，绑定页面元素，访问远程接口，完成图表对象的参数设置和渲染</strong>。一个典型的前端页面文件示例如下图所示：</p>
<pre class="lang-js" data-nodeid="75613"><code data-language="js">&lt;!DOCTYPE html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">charset</span>=<span class="hljs-string">"UTF-8"</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Awesome-pyecharts<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"text/javascript"</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://assets.pyecharts.org/assets/echarts.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"bar"</span> <span class="hljs-attr">style</span>=<span class="hljs-string">"width:1000px; height:600px;"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">var</span> chart = echarts.init(<span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'bar'</span>), <span class="hljs-string">'white'</span>, {<span class="hljs-attr">renderer</span>: <span class="hljs-string">'canvas'</span>});
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $.ajax({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">type</span>: <span class="hljs-string">"GET"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">url</span>: <span class="hljs-string">"http://127.0.0.1:5000/barChart"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">dataType</span>: <span class="hljs-string">'json'</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">success</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">result</span>) </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; chart.setOption(result);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; )
&nbsp;&nbsp;&nbsp; </span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</span></code></pre>
<h4 data-nodeid="75614">服务接口设计</h4>
<p data-nodeid="75615"><strong data-nodeid="75838">服务接口设计是针对前端页面的数据请求设计，该接口请求的数据为 Echarts 的图表配置参数</strong>。具体的服务路由和响应函数如下图所示：</p>
<pre class="lang-java" data-nodeid="75616"><code data-language="java"><span class="hljs-function">def <span class="hljs-title">bar_base</span><span class="hljs-params">()</span> -&gt; Bar:
c </span>= (
Bar()
.add_xaxis([<span class="hljs-string">"衬衫"</span>, <span class="hljs-string">"羊毛衫"</span>, <span class="hljs-string">"雪纺衫"</span>, <span class="hljs-string">"裤子"</span>, <span class="hljs-string">"高跟鞋"</span>, <span class="hljs-string">"袜子"</span>])
.add_yaxis(<span class="hljs-string">"商家 A"</span>, [randrange(<span class="hljs-number">0</span>, <span class="hljs-number">100</span>) <span class="hljs-function"><span class="hljs-keyword">for</span> _ in <span class="hljs-title">range</span><span class="hljs-params">(<span class="hljs-number">6</span>)</span>])
.<span class="hljs-title">add_yaxis</span><span class="hljs-params">(<span class="hljs-string">"商家 B"</span>, [randrange(<span class="hljs-number">0</span>, <span class="hljs-number">100</span>)</span> <span class="hljs-keyword">for</span> _ in <span class="hljs-title">range</span><span class="hljs-params">(<span class="hljs-number">6</span>)</span>])
.<span class="hljs-title">set_global_opts</span><span class="hljs-params">(title_opts=opts.TitleOpts(title=<span class="hljs-string">"Bar-基本示例"</span>, subtitle=<span class="hljs-string">"Flask 整合：前后端分离模式"</span>)</span>)
)
return c

# 图表参数服务接口
@app.<span class="hljs-title">route</span><span class="hljs-params">(<span class="hljs-string">"/barChart"</span>)</span>
def <span class="hljs-title">get_bar_chart</span><span class="hljs-params">()</span>:
c </span>= bar_base()
<span class="hljs-keyword">return</span> c.dump_options_with_quotes()
</code></pre>
<p data-nodeid="75617">当浏览器页面打开地址：http://127.0.0.1:5000/ 时，页面脚本会主动调用服务器端接口: http://127.0.0.1:5000/barChart，获取图表组件配置参数，并进行页面渲染。</p>
<h4 data-nodeid="75618">路由设计</h4>
<p data-nodeid="75619">PyEcharts 与 Flask 整合：前后端分离模式，路由设置包括两部分：</p>
<ol data-nodeid="75620">
<li data-nodeid="75621">
<p data-nodeid="75622">页面路由请求；</p>
</li>
<li data-nodeid="75623">
<p data-nodeid="75624">数据路由请求。</p>
</li>
</ol>
<p data-nodeid="75625">页面路由请求完成页面内容的渲染，数据路由请求完成图表参数的传递。上述示例程序，完整的路由设置如下所示：</p>
<pre class="lang-dart" data-nodeid="75626"><code data-language="dart"># 数据服务接口：图表
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/barChart"</span>)
def get_bar_chart():
c = bar_base()
<span class="hljs-keyword">return</span> c.dump_options_with_quotes()

# 首页渲染
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/"</span>)
def index():
<span class="hljs-keyword">return</span> render_template(<span class="hljs-string">"index.html"</span>)
</code></pre>
<p data-nodeid="75627">页面路由设置了页面根目录的访问方式，同时绑定了业务响应函数：index()，该函数以模板页面为参数，执行了页面渲染。数据路由设置了图表数据请求的响应接口，并绑定了业务响应函数 get_bar_chart()。</p>
<h4 data-nodeid="75628">运行演示</h4>
<p data-nodeid="75629">PyEcharts 与 Flask 整合之后，无论是前后端混合模式还是前后端分离模式，都可以通过启动 Flask 应用程序的方式直接启动服务，然后通过浏览器进行访问。程序运行效果如下图所示：</p>
<p data-nodeid="75630"><img src="https://s0.lgstatic.com/i/image/M00/57/FF/Ciqc1F9t0VuAXUd8AABtKhU07LE773.png" alt="Drawing 9.png" data-nodeid="75854"></p>
<div data-nodeid="75631"><p style="text-align:center">图 10：前后端分离模式运行演示</p></div>
<p data-nodeid="75632">PyEcharts 与 Flask 整合的前后端分离模式，首先是前端页面和后台服务程序之间的解耦，彼此之间约定好数据服务接口，即可执行前后端的各自独立开发。</p>
<p data-nodeid="75633"><strong data-nodeid="75860">前后端分离的开发模式，是目前主流的开发模式，可以实现并行开发，从而加快项目的开发进展</strong>。</p>
<p data-nodeid="75634">实际业务需求中，我们经常需要定时刷新页面数据，尤其是在一些实时监控业务场景下，定时刷新数据成为一个常规需求。</p>
<p data-nodeid="75635">接下来，我们来看 PyEcharts 和 Flask 整合后，如何实现页面数据的刷新。</p>
<h3 data-nodeid="75636">定时全量更新</h3>
<p data-nodeid="75637"><strong data-nodeid="75868">定时全量更新是前端页面发起的，周期性更新整个图表内容的刷新方式，更新的内容包括整个图表的配置参数</strong>。</p>
<p data-nodeid="75638"><strong data-nodeid="75873">实现原理是在前端页面调用 HTML 定时器函数：setInterval()，重新发起图表配置参数的接口调用</strong>。</p>
<p data-nodeid="75639">定时全量更新模式下，后台应用程序不需要做出调整，只需要在前端页面增加基于定时器的调度函数。一个完整的前端页面示例程序如下图所示：</p>
<pre class="lang-js" data-nodeid="75640"><code data-language="js">&lt;!DOCTYPE html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">charset</span>=<span class="hljs-string">"UTF-8"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Awesome-pyecharts<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"text/javascript"</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://assets.pyecharts.org/assets/echarts.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"bar"</span> <span class="hljs-attr">style</span>=<span class="hljs-string">"width:1000px; height:600px;"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
        <span class="hljs-keyword">var</span> chart = echarts.init(<span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'bar'</span>), <span class="hljs-string">'white'</span>, {<span class="hljs-attr">renderer</span>: <span class="hljs-string">'canvas'</span>});
        $(
            <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
                fetchData(chart);
                <span class="hljs-built_in">setInterval</span>(fetchData, <span class="hljs-number">2000</span>);
            }
        );
        <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fetchData</span>(<span class="hljs-params"></span>) </span>{
            $.ajax({
                <span class="hljs-attr">type</span>: <span class="hljs-string">"GET"</span>,
                <span class="hljs-attr">url</span>: <span class="hljs-string">"http://127.0.0.1:5000/barChart"</span>,
                <span class="hljs-attr">dataType</span>: <span class="hljs-string">'json'</span>,
                <span class="hljs-attr">success</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">result</span>) </span>{
                    chart.setOption(result);
                }
            });
        }
    </span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</span></code></pre>
<p data-nodeid="75641">上述程序中，通过 HTML 函数 setInterval()，设置了一个定时器对象，每隔 2 秒钟，重新调用一次图表对象渲染函数 fetchData()，实现定时全量更新图表对象。</p>
<p data-nodeid="75642">定时全量更新程序执行的效果，无法从一张图中直观体现出来，你可以自己运行程序后，看动态效果。我们通过相隔 5 秒钟的两张截图（图 12、图 13）来看：</p>
<p data-nodeid="75643"><img src="https://s0.lgstatic.com/i/image/M00/57/FF/Ciqc1F9t0XeAWMdWAABl9U8S-L0392.png" alt="Drawing 10.png" data-nodeid="75879"></p>
<div data-nodeid="75644"><p style="text-align:center">图 11：定时全量更新图一</p></div>
<p data-nodeid="75645"><img src="https://s0.lgstatic.com/i/image/M00/58/0A/CgqCHl9t0X6AS_aPAABzMABheDk482.png" alt="Drawing 11.png" data-nodeid="75882"></p>
<div data-nodeid="75646"><p style="text-align:center">图 12：定时全量更新图二</p></div>
<p data-nodeid="75647">通过以上示例程序，我们可以看到<strong data-nodeid="75888">定时全量更新图表是通过前端页面设置定时器函数，定时重新访问图表配置参数接口实现的</strong>。</p>
<h3 data-nodeid="75648">定时增量更新</h3>
<p data-nodeid="75649"><strong data-nodeid="75894">定时增量刷新是前端页面发起的、周期性更新整个图表内容的刷新方式，更新的内容只包括图表的数据部分</strong>。</p>
<p data-nodeid="75650"><strong data-nodeid="75899">实现的原理是在前端页面调用 HTML 定时器函数：setInterval()，重新发起图表配置数据项参数的接口调用</strong>。</p>
<p data-nodeid="75651">定时增量更新模式下，需要前台页面和后台应用程序同时做出调整。一方面需要前端页面增加基于定时器的调度函数，另外一方面需要后台应用程序设置单独的增量数据服务接口。一个定时增量更新完整的前端页面示例程序如下所示：</p>
<pre class="lang-js" data-nodeid="75652"><code data-language="js">&lt;!DOCTYPE html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">charset</span>=<span class="hljs-string">"UTF-8"</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Awesome-pyecharts<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://cdn.bootcss.com/jquery/3.0.0/jquery.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"text/javascript"</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://assets.pyecharts.org/assets/echarts.min.js"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>

<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"bar"</span> <span class="hljs-attr">style</span>=<span class="hljs-string">"width:1000px; height:600px;"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> chart = echarts.init(<span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'bar'</span>), <span class="hljs-string">'white'</span>, {<span class="hljs-attr">renderer</span>: <span class="hljs-string">'canvas'</span>});
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">var</span> old_data = [];
&nbsp; &nbsp; &nbsp; &nbsp; $(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fetchData(chart);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">setInterval</span>(getDynamicData, <span class="hljs-number">2000</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; );
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 图表渲染：初始全量获取配置参数</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">fetchData</span>(<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; $.ajax({
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">type</span>: <span class="hljs-string">"GET"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">url</span>: <span class="hljs-string">"http://127.0.0.1:5000/lineChart"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">dataType</span>: <span class="hljs-string">"json"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">success</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">result</span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; chart.setOption(result);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; old_data = chart.getOption().series[<span class="hljs-number">0</span>].data;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 增量数据：定时重置数据属性</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getDynamicData</span>(<span class="hljs-params"></span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; $.ajax({
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">type</span>: <span class="hljs-string">"GET"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">url</span>: <span class="hljs-string">"http://127.0.0.1:5000/lineDynamicData"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">dataType</span>: <span class="hljs-string">"json"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">success</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">result</span>) </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; old_data.push([result.name, result.value]);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; chart.setOption({
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">series</span>: [{<span class="hljs-attr">data</span>: old_data}]
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; }

&nbsp; &nbsp; </span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</span></code></pre>
<p data-nodeid="75653">上述程序定义了一个 HTML定时器，每隔 2 秒钟，刷新一次。刷新的过程是：重新设置 Echarts 图表对象的数据属性，其他属性如标题、工具栏、提示框等参数维持不变。</p>
<p data-nodeid="75654">相应的，我们需要在后台应用程序中，增加 Echarts 图表配置项的数据属性查询的数据服务接口。与上述前端页面程序相对应的，后台应用服务器程序的完整的源码结构如下所示：</p>
<pre class="lang-dart" data-nodeid="75655"><code data-language="dart"># 定时增量数据更新示例
from random <span class="hljs-keyword">import</span> randrange
from flask.json <span class="hljs-keyword">import</span> jsonify
from flask <span class="hljs-keyword">import</span> Flask, render_template
from pyecharts <span class="hljs-keyword">import</span> options <span class="hljs-keyword">as</span> opts
from pyecharts.charts <span class="hljs-keyword">import</span> Line
# <span class="hljs-number">01</span>-创建一个 Flask 应用
app = Flask(__name__, static_folder=<span class="hljs-string">"templates"</span>)

# <span class="hljs-number">02</span>-图表对象配置项参数设置
def line_base() -&gt; Line:
line = (
Line()
.add_xaxis([<span class="hljs-string">"{}"</span>.format(i) <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> range(<span class="hljs-number">10</span>)])
.add_yaxis(
series_name=<span class="hljs-string">""</span>,
y_axis=[randrange(<span class="hljs-number">50</span>, <span class="hljs-number">80</span>) <span class="hljs-keyword">for</span> _ <span class="hljs-keyword">in</span> range(<span class="hljs-number">10</span>)],
is_smooth=True,
label_opts=opts.LabelOpts(is_show=False),
)
.set_global_opts(
title_opts=opts.TitleOpts(title=<span class="hljs-string">"动态数据"</span>),
xaxis_opts=opts.AxisOpts(type_=<span class="hljs-string">"value"</span>),
yaxis_opts=opts.AxisOpts(type_=<span class="hljs-string">"value"</span>),
)
)
<span class="hljs-keyword">return</span> line

# 页面-路由设置
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/"</span>)
def index():
<span class="hljs-keyword">return</span> render_template(<span class="hljs-string">"index.html"</span>)

# 数据-图表配置属性参数：路由设置
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/lineChart"</span>)
def get_line_chart():
c = line_base()
<span class="hljs-keyword">return</span> c.dump_options_with_quotes()

idx = <span class="hljs-number">9</span>

# 数据-图表数据属性配置项更新：路由设置
<span class="hljs-meta">@app</span>.route(<span class="hljs-string">"/lineDynamicData"</span>)
def update_line_data():
global idx
idx = idx + <span class="hljs-number">1</span>
<span class="hljs-keyword">return</span> jsonify({<span class="hljs-string">"name"</span>: idx, <span class="hljs-string">"value"</span>: randrange(<span class="hljs-number">50</span>, <span class="hljs-number">80</span>)})

# 主函数
<span class="hljs-keyword">if</span> __name__ == <span class="hljs-string">"__main__"</span>:
    app.run()
</code></pre>
<p data-nodeid="75656">上述程序中，共声明了 3 个服务接口：页面渲染服务接口、Echarts 图表参数配置查询接口和 Echarts 图表数据项配置参数更新接口，并且绑定了对应的业务响应函数。</p>
<p data-nodeid="75657">定时增量更新案例程序执行以后的效果，我们同样需要在页面上才可以看到动态效果。在此，我只截取一个页面，呈现具体的动态执行的效果。你可以自行执行应用程序查看。定时增量更新程序运行以后的效果如下所示：</p>
<p data-nodeid="75658"><img src="https://s0.lgstatic.com/i/image/M00/57/FF/Ciqc1F9t0cWAQgAZAABsNl56X2g348.png" alt="Drawing 12.png" data-nodeid="75907"></p>
<div data-nodeid="75659"><p style="text-align:center">图 13：定时增量更新程序运行执行演示</p></div>
<h3 data-nodeid="75660">小结</h3>
<p data-nodeid="75661" class="">本小节，我带你了解了 PyEcharts 与 Flask Web 框架的两种整合方式和两种数据刷新机制，案例部分采用了 PyEcharts 的官方示例程序。下一节，我向你介绍 PyEcharts 和 Flask web 框架整合的实战案例，结合<strong data-nodeid="75914">模块三</strong>的 6 大案例，生成一个完整的可视化报表系统。</p>

---

### 精选评论

##### *钡：
> 老师，请问怎么同事更新两个图表数据？我在操作过程中设置更新两个图表，有一个显示不出来了！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以设置定时更新，pyechart官方案例里面有这个方法，可以参考一下

##### **平：
> 老师您好，请问下 y 轴数据太长，显示不全，有什么参数可以指定么？情景：文章示例中的y轴是域名，域名有点长就显示不全了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有好的办法，可以考虑把域名的www部分去掉。

