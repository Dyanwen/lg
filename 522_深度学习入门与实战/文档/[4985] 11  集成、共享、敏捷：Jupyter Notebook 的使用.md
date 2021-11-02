<p data-nodeid="48447" class="te-preview-highlight">在此之前，你已经学习了有关深度学习的理论部分了。从今天开始，你将接触更多与实际编程开发相关的内容，就让我们从常用的工具说起。</p>

<h3 data-nodeid="47836">深度学习编程常用工具</h3>
<p data-nodeid="47837">我们先来看 4 个常用的编程工具：Sublime Text、Pycharm、Vim、Jupyter。虽然我介绍的是 Jupyter，但并不是要求你必须使用它，你也可以根据自己的喜好自由选择。</p>
<h4 data-nodeid="47838">Sublime Text</h4>
<p data-nodeid="47839">第一个是 Sublime Text，它是一个非常轻量且强大的文本编辑工具，内置了很多快捷的功能，同时还支持很丰富的插件功能，对我们来说非常方便。</p>
<p data-nodeid="47840"><img src="https://s0.lgstatic.com/i/image/M00/71/6E/CgqCHl--Ij-Aa73yAANg7SHXDZI730.png" alt="Drawing 0.png" data-nodeid="48081"></p>
<div data-nodeid="47841"><p style="text-align:center">图 1：Sublime Text</p></div>
<p data-nodeid="47842">如上图所示，它可以自动为项目中的类、方法和函数生成索引，我们让我们可以跟踪代码。可以通过它的 goto anything 功能，根据一些关键字查找到项目中的对应的代码行。</p>
<h4 data-nodeid="47843">Pycharm</h4>
<p data-nodeid="47844">第二个是 Pycharm，这里我不做过多的介绍，它与 R Studio 一样都是针对某一编程语言的特定 IDE。</p>
<h4 data-nodeid="47845">Vim</h4>
<p data-nodeid="47846">第三个是 Vim，它是 Linux 系统中的文本编辑工具，方便快捷且强大，我们在项目中经常会使用到。</p>
<p data-nodeid="47847">在我们的项目中，经常需要登录到服务器上进行开发，而服务器一般都是 Linux 系统，不会有 Sublime Text 与 Pycharm，所以我们可以直接用 Vim 打开代码进行编辑。对于没有接触过 Linux 或者是一直使用 IDE 进行编程开发的同学，一开始可能觉得不是很方便，但 Vim 的快捷键十分丰富，对于 Shell 与 Python 的开发来说非常便捷。</p>
<p data-nodeid="47848">Vim 的缺点正如刚才所说，有一点点门槛，需要你去学习它的使用方法。只要你学会了，我保证你将对它爱不释手。</p>
<h4 data-nodeid="47849">Jupyter Notebook &amp; Lab</h4>
<p data-nodeid="47850">最后是咱们这一讲要介绍的 Jupyter Notebook 了，它是一个开源的 Web 应用，能够让你创建、分享包含可执行代码、可视化结构和文字说明的文档。</p>
<p data-nodeid="47851">Jupyter Notebook 的应用非常广泛，它可以用在数据清理与转换、数字模拟、统计模型、数据可视化、机器学习等方面。</p>
<p data-nodeid="47852"><img src="https://s0.lgstatic.com/i/image/M00/71/6E/CgqCHl--IlaAY-TuAAT9uypLxA8221.png" alt="Drawing 1.png" data-nodeid="48096"></p>
<div data-nodeid="47853"><p style="text-align:center">图 2：Jupyter Lab 官方截图</p></div>
<p data-nodeid="47854">Jupyter Notebooks 非常活跃于深度学习领域。在项目的实验测试阶段，它相比于用 py 文件来直接编程还是方便一些。在项目结束之后如果要写项目报告，用 Jupyter 也比较合适。</p>
<p data-nodeid="47855">简单介绍之后，我们接下来就从 Jupyter 的功能、Jupyter 的安装与启动与 Jupyter Lab 的操作这 3 个方面学习 Jupyter。</p>
<h3 data-nodeid="47856">Jupyter Notebook &amp; Lab 的功能</h3>
<p data-nodeid="47857">Jupyter 主要有以下 3 点的作用：执行代码、数据可视化以及使用 Markdown 功能写报告。</p>
<ul data-nodeid="47858">
<li data-nodeid="47859">
<p data-nodeid="47860"><strong data-nodeid="48107">执行代码</strong>。一般是 Python 程序，也可以添加新的编程语言。</p>
</li>
<li data-nodeid="47861">
<p data-nodeid="47862"><strong data-nodeid="48112">数据可视化</strong>。设想一下，我们经常在 Linux 环境编程开发，如果需要对数据可视化该怎么办呢？是不是只能把图片保存下来，然后下载到本地进行查看？使用 Jupyter Notebook 就不用多此一举，我们可以直接在页面中查看。如下图所示：</p>
</li>
</ul>
<p data-nodeid="47863"><img src="https://s0.lgstatic.com/i/image/M00/71/6E/CgqCHl--ImSALN8hAANgFG3PSuY661.png" alt="Drawing 2.png" data-nodeid="48115"></p>
<div data-nodeid="47864"><p style="text-align:center">图 3：Jupyter 数据可视化效果图</p></div>
<ul data-nodeid="47865">
<li data-nodeid="47866">
<p data-nodeid="47867"><strong data-nodeid="48120">使用 Markdown 功能写文档，或者制作 PPT</strong>。这些文档中还包含代码以及代码执行后的结果，非常有助于你书写项目报告。</p>
</li>
</ul>
<h3 data-nodeid="47868">Jupyter Notebook &amp; Lab 的安装与启动</h3>
<p data-nodeid="47869">了解了 Jupyter 的功能之后，我们来看看具体要如何进行安装与启动。这一节我介绍了 3 种安装和启动的方式，分别是 Anaconda、Docker 和 pip。</p>
<h4 data-nodeid="47870">使用 Anaconda 安装与启动</h4>
<p data-nodeid="47871">我们先来看如何使用 Anaconda 来安装与启动。</p>
<p data-nodeid="47872"><strong data-nodeid="48130">安装</strong></p>
<p data-nodeid="47873">最简单的方法是通过安装 Anaconda 来使用 Jupyter Notebook &amp; Lab。Anaconda 已自动安装了 Jupter Notebook 及其他工具，还有 Python 中超过 180 个科学包及其依赖项。你可以通过 <a href="https://www.anaconda.com/" data-nodeid="48136">Anaconda 的官方网站</a>得到 Anaconda 的下载工具。</p>
<p data-nodeid="47874"><strong data-nodeid="48141">启动</strong></p>
<p data-nodeid="47875">这里我会分 MacOS 系统和 Win 环境来讲解。</p>
<p data-nodeid="47876">（1）MacOS 系统</p>
<p data-nodeid="47877">安装完 Anaconda 之后，打开终端后系统会默认进入 base 环境。</p>
<p data-nodeid="47878">在命令行最前面有个**(base)**的标志则表示代码进入 base 环境了，如果没有就需要通过下面的命令激活 base 环境：</p>
<pre class="lang-java" data-nodeid="47879"><code data-language="java">conda activate base
</code></pre>
<p data-nodeid="47880">在 base 环境下执行下面的命令，会自动进入 Jupyte Notebook 的开发环境。</p>
<pre class="lang-plain" data-nodeid="47881"><code data-language="plain">jupyter notebook
</code></pre>
<p data-nodeid="47882">执行下面的命令，则会自动进入到 Jupyter Lab 的开发环境。</p>
<pre class="lang-plain" data-nodeid="47883"><code data-language="plain">jupyter lab
</code></pre>
<p data-nodeid="47884">还有一种方法，我们可以通过 Anaconda Navigator 启动 Juypter Notebook 或者 Jupyter Lab。</p>
<p data-nodeid="47885"><img src="https://s0.lgstatic.com/i/image/M00/71/63/Ciqc1F--IpuAFSSKAATPoCzn8uw475.png" alt="Drawing 3.png" data-nodeid="48157"></p>
<div data-nodeid="47886"><p style="text-align:center">图 4：Anaconda navigator 截图</p></div>
<p data-nodeid="47887">（2）Win 环境</p>
<p data-nodeid="47888">Windows 环境中的启动方式与 MacOS 基本一样。</p>
<p data-nodeid="47889">当你想通过命令 Jupyter Notebook 或 Jupyter Lab 启动时，你需要在 Anaconda Prompt 中执行。</p>
<p data-nodeid="47890"><img src="https://s0.lgstatic.com/i/image/M00/71/63/Ciqc1F--IsWAI2idAAdgXm8M9JI722.png" alt="Drawing 4.png" data-nodeid="48163"></p>
<div data-nodeid="47891"><p style="text-align:center">图 5：Windows 系统 Anaconda 截图</p></div>
<p data-nodeid="47892">通过 Anaconda Navigator 启动的方式与 MacOS 一样。</p>
<h4 data-nodeid="47893">使用 Docker</h4>
<p data-nodeid="47894">通过 Docker 使用 Jupyter 也非常简单，连安装都不需要，但前提是你要有 Docker 相关的知识。</p>
<p data-nodeid="47895">该<a href="https://hub.docker.com/u/jupyter" data-nodeid="48170">地址</a>中有很多的 Jupyter 的 Docker 镜像，你可以在这里找到你需要的，然后拉取一个到你的服务器或者本地，直接启动就可以了。</p>
<h4 data-nodeid="47896">使用 pip 安装与启动</h4>
<p data-nodeid="47897">了解完 Anaconda 和 Docker 的安装与启动方式后，我们最后来看 pip 是如何安装和启动的。</p>
<p data-nodeid="47898"><strong data-nodeid="48177">安装</strong></p>
<p data-nodeid="47899">通过 pip 安装 Jupyter Notebook：</p>
<pre class="lang-plain" data-nodeid="47900"><code data-language="plain">pip install Jupyter
</code></pre>
<p data-nodeid="47901">通过 pip 安装 Jupyter Lab：</p>
<pre class="lang-plain" data-nodeid="47902"><code data-language="plain">pip install Jupyterlab
</code></pre>
<p data-nodeid="47903"><strong data-nodeid="48183">启动</strong></p>
<p data-nodeid="47904">安装完成后，直接在终端执行 Jupyter Notebok 或 Jupyter Lab 命令启动。</p>
<p data-nodeid="47905">不管在 MacOS 系统还是在 Windows 系统，通过以上任意一种方式成功启动后，浏览器都会自动打开 Jupyter Notebook 或 Jupyter Lab 的开发环境，如“Jupyter Notebook &amp; Lab”小节中的图 2。</p>
<h3 data-nodeid="47906">Jupyter Lab 的操作</h3>
<p data-nodeid="47907">Jupyter Lab 是 Jupyter Notebook 的下一代产品，在使用方式上更为灵活、便捷。</p>
<p data-nodeid="47908">我在本讲中介绍的也是 Jupyter Lab。Jupyter Lab 的内容同样适用于 Jupyter Nobebook，它们只是工具的表现形式不一样而已。所以，下面我就省略了 Notebook 的介绍。</p>
<p data-nodeid="47909">我们在命令行或者 Anaconda Navigator 中启动 Jupyter Lab 之后，浏览器会自动打开如下所示的 Jupyter Lab 界面：</p>
<p data-nodeid="47910"><img src="https://s0.lgstatic.com/i/image/M00/71/63/Ciqc1F--ItWAcIEeAAIj7SQv5ok305.png" alt="Drawing 5.png" data-nodeid="48194"></p>
<div data-nodeid="47911"><p style="text-align:center">图 6：Jupyter Lab 界面</p></div>
<p data-nodeid="47912">最左侧显示的是你启动时所在的目录，右侧是你可以使用的一些开发工具。</p>
<p data-nodeid="47913">我们先来看 Notebook 的使用。</p>
<h4 data-nodeid="47914">Notebook</h4>
<p data-nodeid="47915">点击 Notebook 下面的“Python 3”的图标之后，就会自动新建一个 Notebook。</p>
<blockquote data-nodeid="47916">
<p data-nodeid="47917">Jypter Lab 与 Jupyter Notebook 中都会用到这个叫作 Notebook 的编辑工具。</p>
<p data-nodeid="47918">Jupyter Lab 与 Jupyter Notebook 不同的地方是 IDE 的界面以及操作方式，这里讲解用的是 Jupyter Lab 的操作。</p>
</blockquote>
<p data-nodeid="47919">一个 Notebook 的编辑界面主要由 4 个部分组成：菜单栏、工具栏、单元格（Cell）以及内核。如下图所示：</p>
<p data-nodeid="47920"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I3aAStHvAAGEBD0qlGc503.png" alt="Drawing 6.png" data-nodeid="48204"></p>
<div data-nodeid="47921"><p style="text-align:center">图 7：Notebook 的主要编辑界面</p></div>
<p data-nodeid="47922">菜单栏与工具栏这里就不详细介绍了。我们先来看单元格（Cell），然后再介绍内核。</p>
<p data-nodeid="47923"><strong data-nodeid="48209">单元格（Cell）</strong></p>
<p data-nodeid="47924">单元格是我们 Notebook 的主要内容，这里我会介绍两种单元格。</p>
<ul data-nodeid="47925">
<li data-nodeid="47926">
<p data-nodeid="47927"><strong data-nodeid="48215">Code 单元格</strong>：包含可以在内核运行的代码，并且在单元格下方输出运行结果。</p>
</li>
<li data-nodeid="47928">
<p data-nodeid="47929"><strong data-nodeid="48220">Markdown 单元格</strong>：包含运用 Markdown 的文档，常用于文档的说明，也是可以运行的单元格。</p>
</li>
</ul>
<p data-nodeid="47930">从 Code 单元格切换到 Markdown 单元格的切换的快捷键是 m；从 Markdown 单元格切换到 Code 单元格的切换的快捷键是 y。</p>
<p data-nodeid="47931"><strong data-nodeid="48226">切换之前需要先按 Esc，从单元格的编辑状态中退出</strong>。</p>
<p data-nodeid="47932">在工具栏中也可以切换，但是还是快捷键方便些。工具栏的位置在下图中红框的位置：</p>
<p data-nodeid="47933"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I4SAXNiGAADszFKVzDA263.png" alt="Drawing 7.png" data-nodeid="48230"></p>
<div data-nodeid="47934"><p style="text-align:center">图 8：Code 与 Markdown 的工具栏切换位置</p></div>
<p data-nodeid="47935">我们看一个例子。我编辑了下面的 Notebook。第一行是 1 个 Markdown 单元格，是 1 个一级标题，第二行是 1 个 Python 的代码。两行代码都是未运行状态。</p>
<p data-nodeid="47936"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I4-AaBK6AAAVoBvZ4hk416.png" alt="Drawing 8.png" data-nodeid="48234"></p>
<div data-nodeid="47937"><p style="text-align:center">图 9：Notebook 中的代码编辑</p></div>
<p data-nodeid="47938">你注意到左边那个蓝色的竖条了吗？它代表我们所在的单元格。</p>
<p data-nodeid="47939">我们在编辑这个单元格的时候，左边是绿色的竖条。如果我们按 Esc 退出单元格，它就会变为蓝色。</p>
<p data-nodeid="47940">退出单元格后，我们可以通过上下键移动选中的单元格。我们移动到第一行，然后开始运行这两个单元格。</p>
<p data-nodeid="47941">运行单独一个单元格的快捷键 Ctrl+Enter，运行选中单元格并切换到下一个单元格的快捷键是 Shift + Enter。运行结果如下图所示：</p>
<p data-nodeid="47942"><img src="https://s0.lgstatic.com/i/image/M00/71/64/Ciqc1F--I5iANBmTAAAccFQ7alU161.png" alt="Drawing 9.png" data-nodeid="48241"></p>
<div data-nodeid="47943"><p style="text-align:center">图 10：单元格运行结果</p></div>
<p data-nodeid="47944">Markdown 没有左边的“[]”标签，通过这一点你可以区分 Code 单元格与 Markdown 单元格。</p>
<p data-nodeid="47945">“[]”中的数字代表单元格被执行的顺序，例子中“[1]”代表第一个被执行的单元格。</p>
<p data-nodeid="47946">以上就是单元格的内容了。我们接下来看看，单元格中的一些快捷键的使用。</p>
<p data-nodeid="47947"><strong data-nodeid="48258">（1）快捷键</strong></p>
<p data-nodeid="47948">如果你是用 Jupyter 进行开发，掌握单元格的快捷键能让你的开发速度变得更快，下面我列举了几个常用的快捷键：</p>
<ul data-nodeid="47949">
<li data-nodeid="47950">
<p data-nodeid="47951">执行单元格 Ctrl+Enter 或 Shift+Enter；</p>
</li>
<li data-nodeid="47952">
<p data-nodeid="47953">a 在单元格上方插入新的单元格；</p>
</li>
<li data-nodeid="47954">
<p data-nodeid="47955">b 在单元格下方插入新的单元格；</p>
</li>
<li data-nodeid="47956">
<p data-nodeid="47957">x 删除单元格；</p>
</li>
<li data-nodeid="47958">
<p data-nodeid="47959">z 撤销删除的单元格。</p>
</li>
</ul>
<p data-nodeid="47960"><strong data-nodeid="48268">（2）Magic 命令</strong></p>
<p data-nodeid="47961">Jupyter Notebook 的前身是 IPython Notebook，所以 Jupyter 也支持 IPython 的 Magic 命令。IPython 是一个比 Python 自带的 Shell 更加灵活方便的 Shell，它主要活跃于数据科学领域。</p>
<p data-nodeid="47962">Magic 命令分两种：</p>
<ul data-nodeid="47963">
<li data-nodeid="47964">
<p data-nodeid="47965">Line Magics 命令：在命令前面加%，表示只在本行有效</p>
</li>
<li data-nodeid="47966">
<p data-nodeid="47967">Cell Magics 命令：在命令前面加%%，表示在整个 Cell 单元有效。</p>
</li>
</ul>
<p data-nodeid="47968">下面我介绍几个常用的 Magic 命令。</p>
<p data-nodeid="47969"><strong data-nodeid="48278">%lsmagic</strong>：用来查看可以使用的 Magic 命令。</p>
<p data-nodeid="47970"><img src="https://s0.lgstatic.com/i/image/M00/71/64/Ciqc1F--I66AfxFNAAA5qmEu6h8235.png" alt="Drawing 10.png" data-nodeid="48281"></p>
<div data-nodeid="47971"><p style="text-align:center">图 11：%lsmagic 命令</p></div>
<p data-nodeid="47972"><strong data-nodeid="48286">%matplotlib inline</strong>：可以在单元格下面直接打印出 matplotlib 的图标，通常要在 matplotlib 模块引入之前使用；使用这个 Magic 命令之后，可以不用 plt.show()。</p>
<p data-nodeid="47973"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I7WAL9WtAADSQj6wMEE758.png" alt="Drawing 11.png" data-nodeid="48289"></p>
<div data-nodeid="47974"><p style="text-align:center">图 12：%matplotlib inline 命令</p></div>
<p data-nodeid="47975"><strong data-nodeid="48294">%pwd</strong>：查看当前的文件路径。</p>
<p data-nodeid="47976"><strong data-nodeid="48299">%%writefile</strong>：写文件，%%writefile 后面紧跟着文件名，然后下面写文件的内容。</p>
<p data-nodeid="47977"><strong data-nodeid="48304">%run</strong>： 运行一个文件，%run 后面跟着要运行的文件。</p>
<p data-nodeid="47978">请看下面的例子，我们先写一个 temp.py 的文件。运行单元格之后会在当前位置生成一个叫作 temp.py 的文件，然后我们使用%run 来运行它。</p>
<p data-nodeid="47979"><img src="https://s0.lgstatic.com/i/image/M00/71/64/Ciqc1F--I72ADTDOAACGDTfzacM259.png" alt="Drawing 12.png" data-nodeid="48308"></p>
<div data-nodeid="47980"><p style="text-align:center">图 13：命令展示图</p></div>
<p data-nodeid="47981"><strong data-nodeid="48313">%load</strong>：加载文件。使用%load + 文件名可以把指定的文件加载到单元格内。请看下面的例子，我们要把 temp.py 加载到单元格里，首先是执行前，</p>
<p data-nodeid="47982"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I8yADfXjAAAogwvzRZE772.png" alt="Drawing 13.png" data-nodeid="48316"></p>
<div data-nodeid="47983"><p style="text-align:center">图 14：%load 命令执行前</p></div>
<p data-nodeid="47984">运行后，自动加载 temp.py 到本单元格。</p>
<p data-nodeid="47985"><img src="https://s0.lgstatic.com/i/image/M00/71/6F/CgqCHl--I9OAMn4UAABNBcWiRzI223.png" alt="Drawing 14.png" data-nodeid="48320"></p>
<div data-nodeid="47986"><p style="text-align:center">图 15：%load 命令执行后</p></div>
<p data-nodeid="47987"><strong data-nodeid="48324">（3）Markdown 命令</strong></p>
<p data-nodeid="47988">了解了 Magic 命令后，我们再来看 Markdown 命令。Markdown 是一种在 Markdown 单元中用于格式化文本的语言，常用于 Notebook 的文档说明，我们列举了几个常用的命令。</p>
<ul data-nodeid="47989">
<li data-nodeid="47990">
<p data-nodeid="47991"><strong data-nodeid="48330">标题</strong>：通过井号的数目可以决定标题的大小。</p>
</li>
</ul>
<pre class="lang-plain" data-nodeid="47992"><code data-language="plain"># 一级标题：
## 二级标题：
### 三级标题：
#### 四级标题：&nbsp;
##### 五级标题：
</code></pre>
<p data-nodeid="47993">上述 Markdown 命令的执行结果如下：</p>
<p data-nodeid="47994"><img src="https://s0.lgstatic.com/i/image/M00/71/6B/Ciqc1F--M-GARCeVAADhiCTtLk0829.png" alt="图片15.png" data-nodeid="48334"></p>
<div data-nodeid="47995"><p style="text-align:center">图 16：“标题”命令展示图</p></div>
<ul data-nodeid="47996">
<li data-nodeid="47997">
<p data-nodeid="47998"><strong data-nodeid="48339">列表</strong>：分为无序列表与有序列表。</p>
</li>
</ul>
<pre class="lang-plain" data-nodeid="47999"><code data-language="plain">## 无序列表
- 项目 1
- 项目 2
## 有序列表
1. 项目 1 (1. 与项目 1 之间有一个空格)
2. 项目 2
</code></pre>
<p data-nodeid="48000">上述 Markdown 命令的执行结果如下：</p>
<p data-nodeid="48001"><img src="https://s0.lgstatic.com/i/image/M00/71/6B/Ciqc1F--M-2AOVZaAADi54LxFpY147.png" alt="图片16.png" data-nodeid="48343"></p>
<div data-nodeid="48002"><p style="text-align:center">图 17：“列表”命令展示图</p></div>
<ul data-nodeid="48003">
<li data-nodeid="48004">
<p data-nodeid="48005"><strong data-nodeid="48350">换行</strong>：第一种换行方法是在行尾添加<br>
进行换行；第二种则是两行之间空一行。</p>
</li>
<li data-nodeid="48006">
<p data-nodeid="48007"><strong data-nodeid="48359">字体</strong>：可以通过*或者_的数目控制强调的内容，即斜体、加粗以及粗斜体。具体的请看下面的例子。</p>
</li>
</ul>
<pre class="lang-plain" data-nodeid="48008"><code data-language="plain">*斜体*
**加粗** 
***粗斜体***
或者
_斜体_
__加粗__
___粗斜体___
</code></pre>
<p data-nodeid="48009">运行后效果如下所示：</p>
<p data-nodeid="48010"><img src="https://s0.lgstatic.com/i/image/M00/71/6B/Ciqc1F--M_aAY-A8AADP0Q0yTNg149.png" alt="图片17.png" data-nodeid="48363"></p>
<div data-nodeid="48011"><p style="text-align:center">图 18：“字体”命令展示图</p></div>
<p data-nodeid="48012"><strong data-nodeid="48367">（4）调用系统命令</strong></p>
<p data-nodeid="48013">最后，在 Notebook 中还可以调用所在操作系统的命令，只需要在命令前加一个“!”就可以了。例如，在 Linux 系统中查看当前路径：</p>
<pre class="lang-plain" data-nodeid="48014"><code data-language="plain">!pwd
</code></pre>
<p data-nodeid="48015"><strong data-nodeid="48374">内核</strong></p>
<p data-nodeid="48016">上面我介绍了单元格及单元格的常用快捷键和命令，接下来，我们来了解一下内核。我们刚才打开这个 Notebook 的时候其实已经选好了它的内核，即 Python3。</p>
<p data-nodeid="48017">什么是内核呢？</p>
<p data-nodeid="48018">Notebook 背后其实都有一个内核，我们在单元格中写好的代码就是运行在内核中的。运行代码后，内核会将代码中所有的输出都返回给单元格，并在单元格下方显示。</p>
<p data-nodeid="48019">在编辑 Notebook 中的单元格的时候，内核的状态是保持不变的。换句话说，内核的运行状态只与 Notebook 有关，与 Notebook 中的单元格无关。</p>
<p data-nodeid="48020">例如，你在一个单元格中定义了变量、方法或是导入了模块，并运行了它。那么，在其他的单元格中仍然可以使用这些变量、方法以及模块，不会因为我们换了一个单元格，之前的代码就失效了，因为它们都运行在同一个 Notebook 的内核中。</p>
<p data-nodeid="48021">请看下面的例子：</p>
<p data-nodeid="48022"><img src="https://s0.lgstatic.com/i/image/M00/71/70/CgqCHl--JCSAByD0AACis_kkeiA791.png" alt="Drawing 18.png" data-nodeid="48383"></p>
<div data-nodeid="48023"><p style="text-align:center">图 19：内核状态示意图</p></div>
<p data-nodeid="48024">上图中，我们在第一个单元格定义了一个 print_message 函数，然后在下一个单元格中写了一句说明，在最下面一个单元格中依然可以调用了第一个单元格中的 print_message 函数。</p>
<p data-nodeid="48025">我们编辑一个 Notebook 的开发流程肯定是从上至下的，一边分析运行结果一边考虑下面的代码该如何书写。</p>
<p data-nodeid="48026">开发到某一阶段时，可能会发现之前的某些代码写错了，需要回过头重新编辑然后运行代码，这是一件很正常的事情。但这个时候你要注意，每个 Code 单元格是有执行顺序的，这个顺序就是每个单元格前面[]的数字。</p>
<p data-nodeid="48027">如果你想全部重新执行的话，Kernal 菜单中的一些功能会帮到你。</p>
<ul data-nodeid="48028">
<li data-nodeid="48029">
<p data-nodeid="48030">Restart Kernel：重新启动内核，清除所有变量。</p>
</li>
<li data-nodeid="48031">
<p data-nodeid="48032">Restart Kernel and Clear All Outputs：重新启动内核，清除所有变量与输出。</p>
</li>
<li data-nodeid="48033">
<p data-nodeid="48034">Restart Kernel and Run up to Selected Cell：重启内核，运行选中的单元格。</p>
</li>
<li data-nodeid="48035">
<p data-nodeid="48036">Restart Kernel and Run All Cells：重启内核，运行所有单元格。</p>
</li>
</ul>
<p data-nodeid="48037">我们现在使用的是 Python 内核，其实 Jupyter 还支持很多其他的内核，如 Java、C、R 以及 Julia 等编程语言的内核。我们创建 NoteBook 的时候就选择好了使用什么样的内核。</p>
<p data-nodeid="48038"><strong data-nodeid="48403">Notebook 的配置</strong></p>
<p data-nodeid="48039">我们使用 IDE 开发的时候经常会根据实际需要以及个人的喜好，对 IDE 进行一些配置。Notebook 同样支持个性化的配置。</p>
<p data-nodeid="48040">我们可以通过下面这条命令找到 Notebook 的配置文件：</p>
<pre class="lang-plain" data-nodeid="48041"><code data-language="plain">Jupyter notebook --generate-config
</code></pre>
<p data-nodeid="48042">执行完上述命令，Jupyter 就会告诉你生成的配置文件在哪，如果已经有了会问你是否覆盖。配置文件名叫 Jupyter_notebook_config.py。</p>
<p data-nodeid="48043">下面我们就对 Notebook 做一些常用的配置。</p>
<p data-nodeid="48044"><strong data-nodeid="48415">（1）修改启动后的工作路径</strong></p>
<p data-nodeid="48045">默认情况下，在哪里执行 Jupyter Lab 命令，启动后的工作路径就在哪里，如下图左侧显示的位置：</p>
<p data-nodeid="48046"><img src="https://s0.lgstatic.com/i/image/M00/71/64/Ciqc1F--JEeAF_M0AAIj7SQv5ok729.png" alt="Drawing 19.png" data-nodeid="48419"></p>
<div data-nodeid="48047"><p style="text-align:center">图 20：默认工作路径</p></div>
<p data-nodeid="48048">我们可以通过修改下面的变量修改启动的路径：</p>
<pre class="lang-plain" data-nodeid="48049"><code data-language="plain">c.NotebookApp.notebook_dir = '/home/lagou_edu/work'
</code></pre>
<p data-nodeid="48050"><strong data-nodeid="48424">（2）在服务器端启动 Jupyter</strong></p>
<p data-nodeid="48051">讲到这里，我只介绍了在自己本地的机器上启动 Jupyter 的方法：<strong data-nodeid="48430">在自己的电脑上输入 Jupyter Lab 或者 Jupyter Notebook 命令就可以打开 Jupyter</strong>。但是我们经常需要在服务器上开发，也就是在服务器端启动 Jupyter，在本地通过浏览器访问服务器。</p>
<p data-nodeid="48052">如果是这样的话，我们就需要做如下配置了。</p>
<p data-nodeid="48053">首先，通过修改下面的位置，使所有 IP 都可以访问 Jupyter 服务。</p>
<pre class="lang-plain" data-nodeid="48054"><code data-language="plain">c.NotebookApp.ip = '*' # 开启所有的 IP 访问，即可使用远程访问
</code></pre>
<p data-nodeid="48055">然后设定访问的端口号。</p>
<pre class="lang-plain" data-nodeid="48056"><code data-language="plain">c.NotebookApp.port = 8888
</code></pre>
<p data-nodeid="48057">接着关闭启动 Jupyter 后自动弹出浏览器的选项（这一步是可选的）。通常，在服务器上不需要弹出浏览器，因为绝大多数情况下服务器都是没有图形界面的 Linux 系统，根本没有浏览器。</p>
<pre class="lang-plain" data-nodeid="48058"><code data-language="plain">c.NotebookApp.open_browser = False
</code></pre>
<p data-nodeid="48059">最后，设置登录 Jupyter 的密码。如果不设置密码，启动时会生成一段 token，然后登录的时候复制这段 token 就可以了。下面是不设置密码的配置方式：</p>
<pre class="lang-plain" data-nodeid="48060"><code data-language="plain">c.NotebookApp.password = ''
</code></pre>
<p data-nodeid="48061">如果需要设置密码的话，在 Jupyter 5.0 之后可以通过下面的命令进行设置：</p>
<pre class="lang-plain" data-nodeid="48062"><code data-language="plain">Jupyter notebook password
</code></pre>
<p data-nodeid="48063">它会自动帮你修改好密码。</p>
<p data-nodeid="48064">或者你也可以手动设置，在命令行输入 iPython，然后执行下面的命令：</p>
<pre class="lang-plain" data-nodeid="48065"><code data-language="plain">from notebook.auth import passwd
passwd()
</code></pre>
<p data-nodeid="48066">密码设置完成后，会生成一个字符串，需要手动赋值给 c.NotebookApp.password 就可以了。</p>
<h4 data-nodeid="48067">Terminal</h4>
<p data-nodeid="48068">介绍完 Notebook，让我们回到这一节的开始，图 6 的“Other”下还有一个“Terminal”图标。Terminal 也是一个经常被使用到的工具，它是一个命令终端，在这个终端里你可以对操作系统进行操作，就是我们常见的 XShell、iterm2 这样的工具。</p>
<h3 data-nodeid="48069">总结</h3>
<p data-nodeid="48070">到这里，你就完成了 Jupyter 的学习。通过一些简单的练习，你就可以上手进行一些开发或者实验了。</p>
<p data-nodeid="48071">我在这一讲介绍了一些常用的方法，还有很多其他的命令以及功能需要你自己去摸索。正所谓“熟能生巧”，Jupyter 本身没有任何难度，只要你多加使用，很快就可以掌握它，希望它能够在今后的学习与工作中帮到你。</p>
<p data-nodeid="48072">那么，在 Jupyter Lab 的使用上你遇到过什么困难，又是如何解决的呢？欢迎在留言区留言。</p>
<p data-nodeid="48073" class="">下一讲，我将带你了解自然语言处理与计算机视觉常用的预处理方式，这部分内容在实际项目中必不可少。通过对这一部分的学习，你可以更快地融入实际的项目。</p>

---

### 精选评论

##### **升：
> 公司项目开发也用Jupyter吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; jupyter侧重于数据的分析与探索，比如做个实验啥的。如果真的要开发公司的商业应用，还有很多不错的ide，比如pycharm。

