<p data-nodeid="1405" class="">在 13 和 14 讲中，我们一同了解了 TensorFlow。通过 TensorFlow，我们可以将设计好的理论模型变成实际可用的真正的模型。这一讲，我们将学习一个高效的实验分析助手：TensorBoard。</p>
<h3 data-nodeid="1406">什么是 TensorBoard</h3>
<p data-nodeid="1407">在机器学习中，我们经常要衡量和把握模型的参数、指标等信息，这就需要一种工具，希望它能提供机器学习工作流程期间所需的测量和可视化的信息，于是就有了 TensorBoard。</p>
<p data-nodeid="1408">在构建深度学习的模型时，只要模型开始训练了，很多内部细节是不对外暴露的。模型的参数是多少、变大了还是变小了、当前的准确率是多少，我们都不知道。而 TensorBoard 可以通过 Web 页面提供查看细节与过程的功能，它将模型的细节与过程，通过浏览器可视化的方式进行展现，帮助使用者感知各个参数与指标的变化，把握训练趋势。</p>
<p data-nodeid="1409">在《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=522#/detail/pc?id=4987" data-nodeid="1532">13｜张量、数据流图与概念：初步了解 TensorFlow</a>》中我们学会了如何安装 TensorFlow，TensorFlow 安装好之后，实际上 TensorBoard 也已经被安装了，所以不需要单独安装。</p>
<h3 data-nodeid="1410">TensorBoard 的简单使用</h3>
<p data-nodeid="29879" class="">在 13 讲中，我们还了解了 TensorFlow 1 和 2 版本的不同，并选择了 TensorFlow 2 作为学习的版本；同样的，TensorBoard 也是在 TensorFlow 2 的基础上运行的。</p>










































<p data-nodeid="1412">TensorBoard 一般有两种使用方法：<strong data-nodeid="1545">Model.fit()中调用</strong>，或者<strong data-nodeid="1546">在其他函数中使用</strong>。</p>
<h4 data-nodeid="1413">在 model.fit()中使用 TensorBoard</h4>
<p data-nodeid="1414">在具体看 TensorBoard 可视化界面到底是什么样子之前，我们仍旧使用经典的 MNIST 数据来作为案例，先把模型跑起来。我们先加载数据和构建模型：</p>
<pre class="lang-python" data-nodeid="1415"><code data-language="python"><span class="hljs-keyword">import</span> tensorflow <span class="hljs-keyword">as</span> tf
<span class="hljs-keyword">import</span> tensorboard
<span class="hljs-keyword">import</span> datetime
<span class="hljs-comment"># 加载 mnist 数据</span>
mnist = tf.keras.datasets.mnist
<span class="hljs-comment"># 将数据进行切分，分为训练集和验证集</span>
(x_train, y_train),(x_test, y_test) = mnist.load_data()
<span class="hljs-comment"># 还记得为什么要除以 255 么</span>
x_train, x_test = x_train / <span class="hljs-number">255.0</span>, x_test / <span class="hljs-number">255.0</span>
<span class="hljs-comment"># 构建一个简单的模型，拍平--全连接--dropout--全链接</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create_model</span>():</span>
  <span class="hljs-keyword">return</span> tf.keras.models.Sequential([
    tf.keras.layers.Flatten(input_shape=(<span class="hljs-number">28</span>, <span class="hljs-number">28</span>)),
    tf.keras.layers.Dense(<span class="hljs-number">512</span>, activation=<span class="hljs-string">'relu'</span>),
    tf.keras.layers.Dropout(<span class="hljs-number">0.2</span>),
    tf.keras.layers.Dense(<span class="hljs-number">10</span>, activation=<span class="hljs-string">'softmax'</span>)
  ])
</code></pre>
<p data-nodeid="1416">以上代码，你可以直接在 Jupyter 中使用，也可以在终端使用。</p>
<p data-nodeid="1417">然后我们就可以在代码中加入 TensorBoard 的内容了。首先要构建一个大致的训练框架，包括初始化模型、模型训练等关键步骤：</p>
<pre class="lang-python" data-nodeid="31262"><code data-language="python">model = create_model()
model.compile(optimizer=<span class="hljs-string">'adam'</span>,loss=<span class="hljs-string">'sparse_categorical_crossentropy'</span>, metrics=[<span class="hljs-string">'accuracy'</span>])
<span class="hljs-comment"># 定义日志目录，这里需要注意的是，它必须是启动 Web 应用时指定目录的子目录</span>
log_dir=<span class="hljs-string">"you/log/path"</span>
tensorboard_callback = tf.keras.callbacks.TensorBoard(log_dir=log_dir, histogram_freq=<span class="hljs-number">1</span>)
model.fit(x=x_train, 
          y=y_train, 
          epochs=<span class="hljs-number">5</span>, 
          validation_data=(x_test, y_test), 
          callbacks=[tensorboard_callback])
</code></pre>


<p data-nodeid="1419">运行完成之后，我们可以在 log 文件夹中看到用时间命名的子文件夹，其中包含了一个以“v2”结尾的日志文件。执行如下命令，就可以看到 TensorBoard 的可视化展示了：</p>
<pre class="lang-shell" data-nodeid="1420"><code data-language="shell">tensorboard --logdir "logs/"
</code></pre>
<p data-nodeid="1421"><strong data-nodeid="1556">注意，--logdir 后面不要使用=的形式，要使用双引号的形式</strong>。</p>
<p data-nodeid="1422">在浏览器中输入<a href="http://localhost:6006" data-nodeid="1560">http://localhost:6006</a>就看到 TensorBoard 的样子了。如下图所示：</p>
<p data-nodeid="1423"><img src="https://s0.lgstatic.com/i/image/M00/80/19/CgqCHl_QcFOARF4aAAEZ_Twu9N8686.png" alt="Drawing 0.png" data-nodeid="1564"></p>
<div data-nodeid="1424"><p style="text-align:center">图 1：TensorBoard</p></div>
<p data-nodeid="1425">我们接下来看看这个页面都有些什么。</p>
<p data-nodeid="31953" class="">最上面的橙色菜单栏中，我们能够看到几个页卡，分别是 scalars、graphs、distributions、histograms 和 time series。这几个页卡提供了不同的功能。</p>

<p data-nodeid="1427">首先是<strong data-nodeid="1572">scalars</strong>。</p>
<p data-nodeid="1428">还记得咱们之前学习到的标量吗？scalar 就可以理解为标量。这里记录的是各个数据指标的变化趋势信息，而数据指标一般也确实都是标量。</p>
<p data-nodeid="1429">在 scalars 的图像中，我们可以看到深浅两种曲线，其中浅色的为未平滑的实际线图，深色的则是平滑之后的线图，我们可以通过左侧的 smoothing 调节平滑的程度。</p>
<p data-nodeid="1430">此外，scalars 还可以实现的交互有：</p>
<ul data-nodeid="1431">
<li data-nodeid="1432">
<p data-nodeid="1433">点击每个图表左下角的蓝色小图标将展开图表；</p>
</li>
<li data-nodeid="1434">
<p data-nodeid="1435">拖动图表上的矩形区域将放大；</p>
</li>
<li data-nodeid="1436">
<p data-nodeid="1437">双击图表将缩小；</p>
</li>
<li data-nodeid="1438">
<p data-nodeid="1439">鼠标悬停在图表上会产生十字线，数据值记录在左侧的运行选择器中。</p>
</li>
</ul>
<p data-nodeid="1440">然后是<strong data-nodeid="1585">graphs</strong>。该页卡记录了模型的结构信息，当我们打开这个页卡的时候就可以看到如下的信息：</p>
<p data-nodeid="1441"><img src="https://s0.lgstatic.com/i/image/M00/80/0E/Ciqc1F_QcGaACVbPAAEdZ1TD0pA988.png" alt="Drawing 1.png" data-nodeid="1588"></p>
<div data-nodeid="1442"><p style="text-align:center">图 2：TensorBoard graphs 页卡</p></div>
<p data-nodeid="1443">TensorBoard 的模型结构可视化可以让使用者非常方便地了解模型的“样子”。但我个人觉得，<strong data-nodeid="1594">graphs 在小模型的情况下确实非常有用</strong>，如果模型结构比较复杂（比如后续实战中用到的图像分类模型），那整个可视化展示的网络就会十分庞大，反倒不如通过代码的方式了解网络结构本身的情况。</p>
<p data-nodeid="38252" class="">最后是 <strong data-nodeid="38262">histograms</strong> 和 <strong data-nodeid="38263">distributions</strong>。在这里你可以看到 activations、gradients 以及 weights 等变量的每一步的分布，越靠前就是越新的步数的结果。后文中我会具体介绍。</p>









<blockquote data-nodeid="1445">
<p data-nodeid="1446">time serie 暂时用不上，这里就不做过多讲解了。</p>
</blockquote>
<p data-nodeid="1447">在 TensorBoard 中有很多可供选择的页卡，一般 scalars 使用较多， 因为它可以直观地展示数据指标的变化趋势。下面我们来看看 TensorBoard 其他的使用方法。</p>
<h4 data-nodeid="1448">更加灵活地使用 TensorBoard</h4>
<p data-nodeid="1449">在 model.fit()中使用 TensorBoard 的时候，可定制化的东西还是挺少的，没有那么灵活。因此，我们可以通过 tf.summary()方法指定需要 TensorBoard 展示的参数。这个方法非常简单，甚至可以脱离模型训练本身来进行，也就是说：<strong data-nodeid="1614">只要给了数据，就能记录</strong>。我们不妨举一个简单的例子。</p>
<pre class="lang-python" data-nodeid="44482"><code data-language="python"><span class="hljs-keyword">import</span> tensorflow <span class="hljs-keyword">as</span> tf
<span class="hljs-keyword">import</span> random
<span class="hljs-keyword">import</span> tensorboard
<span class="hljs-keyword">import</span> datetime
<span class="hljs-comment"># 这里，我们首先生成一个 tf.summary 的记录文件。</span>
log_dir=<span class="hljs-string">"./logs/"</span> + datetime.datetime.now().strftime(<span class="hljs-string">"%Y%m%d-%H%M%S"</span>)
summary_writer = tf.summary.create_file_writer(log_dir)
<span class="hljs-comment"># 假设我们有一些二维坐标点，我们以函数 y=0.1x^2-4x+1 为例，此外，为了引入类似模型波动的效果，我们在 y 上乘一个 0.9～1 的随机数。</span>
xs = list(range(<span class="hljs-number">100</span>))
ys = [(<span class="hljs-number">0.1</span> * x * x - <span class="hljs-number">4</span> * x + <span class="hljs-number">1</span>) * random.randint(<span class="hljs-number">90</span>,<span class="hljs-number">100</span>)/<span class="hljs-number">100</span> <span class="hljs-keyword">for</span> x <span class="hljs-keyword">in</span> xs]
<span class="hljs-comment"># 任意一个二维坐标点，我们只需要使用 tf.summary.scalar，就可以记录该信息。</span>
<span class="hljs-keyword">for</span> i, j <span class="hljs-keyword">in</span> zip(xs, ys):
&nbsp; &nbsp; <span class="hljs-keyword">with</span> summary_writer.as_default():
&nbsp; &nbsp; &nbsp; &nbsp; tf.summary.scalar(<span class="hljs-string">'fake_index'</span>, j, step=i)
summary_writer.close()
</code></pre>









<p data-nodeid="1451">tf.summary()声明记录的位置以及记录的数据，只需要 2 句核心代码，就可以完成数据指标的追踪。我们来看看这 100 个二维坐标在 Web 页面的显示情况：</p>
<p data-nodeid="1452"><img src="https://s0.lgstatic.com/i/image/M00/80/19/CgqCHl_QcHKAO3ziAAFuKH9aOEY579.png" alt="Drawing 2.png" data-nodeid="1618"></p>
<div data-nodeid="1453"><p style="text-align:center">图 3：100 个二维坐标在 Web 页面的显示情况</p></div>
<p data-nodeid="1454">我们可以在每一个 epoch，甚至每一个 step 完成之后，记录模型的参数信息、性能信息等内容。</p>
<h3 data-nodeid="1455">TensorFlow Summary API 的使用</h3>
<p data-nodeid="1456">在使用 summary API 之前，我们需要先声明 SummaryWriter 的实例：</p>
<pre class="lang-python" data-nodeid="1457"><code data-language="python">tf.summary.create_file_writer(logdir, max_queue=<span class="hljs-number">10</span>, flush_millis=<span class="hljs-literal">None</span>, filename_suffix=<span class="hljs-literal">None</span>, name=<span class="hljs-literal">None</span>)
</code></pre>
<p data-nodeid="1458">以下是代码中的注解。</p>
<ul data-nodeid="1459">
<li data-nodeid="1460">
<p data-nodeid="1461"><strong data-nodeid="1627">logdir</strong>为我们要保存 log 文件的路径。</p>
</li>
<li data-nodeid="1462">
<p data-nodeid="1463"><strong data-nodeid="1638">max_queue</strong>代表最多在缓存中暂存 max_queue 个数据。默认为 10，在实际使用的时候，默认值差不多就够了。如果超过 max_queue 个，flush 会更新到日志文件中并清空缓存。</p>
</li>
<li data-nodeid="1464">
<p data-nodeid="1465"><strong data-nodeid="1647">flush_millis</strong>表示至少 flush_millis 毫秒内进行一次 flush。</p>
</li>
<li data-nodeid="1466">
<p data-nodeid="1467"><strong data-nodeid="1654">filename_suffix</strong>是日志文件的后缀，默认为.v2。</p>
</li>
<li data-nodeid="1468">
<p data-nodeid="1469"><strong data-nodeid="1659">name</strong>则代表本操作的名称。</p>
</li>
</ul>
<p data-nodeid="1470">SummaryWriter 作为一个类，其内部自然也有一些预先声明好的函数，一般常用的有 set_as_default、flush 和 close。</p>
<ul data-nodeid="1471">
<li data-nodeid="1472">
<p data-nodeid="1473"><strong data-nodeid="1673">set_as_default</strong>：该函数会设定默认的 summary writer，也就是以后出现的所有 summary 写入操作，都默认使用本个 summary writer 实例进行记录。</p>
</li>
<li data-nodeid="1474">
<p data-nodeid="1475"><strong data-nodeid="1678">flush</strong>：强制写入操作。log 的生成不是严格实时的，有时候我们需要尽快写入的话，就要用到该函数。</p>
</li>
<li data-nodeid="1476">
<p data-nodeid="1477"><strong data-nodeid="1683">close</strong>：写文件的常规操作，但是不能忘了哟。</p>
</li>
</ul>
<p data-nodeid="1478">刚才我初步介绍了 summary 的使用，现在是它的各个常用方法。</p>
<h4 data-nodeid="1479">tf.summary.scalar</h4>
<p data-nodeid="1480">现在你已经知道，该方法是适用于记录标量数据信息的。其具体参数如下：</p>
<pre class="lang-python" data-nodeid="1481"><code data-language="python">tf.summary.scalar(tags, values, step)
</code></pre>
<p data-nodeid="1482">其中：</p>
<ul data-nodeid="1483">
<li data-nodeid="1484">
<p data-nodeid="1485">tags 代表记录的节点的名字；</p>
</li>
<li data-nodeid="1486">
<p data-nodeid="1487">values 代表节点的值，也就是纵坐标；</p>
</li>
<li data-nodeid="1488">
<p data-nodeid="1489">step 代表训练的步数。</p>
</li>
</ul>
<p data-nodeid="1490">在实际的使用中，values 和 step 的使用非常灵活，你可以用 step 记录 epoch，也可以记录 batch。</p>
<h4 data-nodeid="1491">tf.summary.histogram</h4>
<p data-nodeid="1492">tf.summary.histogram()将输入的张量压缩成了一个由宽度和数量组成的直方图数据结构。为了简化问题，我们假设有一个数组（实际是张量）：[1,3, 2.8, 2.9, 3.3, 4.1]，则 TensorBoard 会将这个数组分成多个块：小于 2 的数据为一个块，2 和 3 之间的数据一个块，3 和 4 之间的数据一个块等等。这个数组就是模型训练中我们要记录的信息（比如 loss）。</p>
<p data-nodeid="1493">在实际的训练中，每一个训练的 step（或者 epoch）就会有一个这样的张量需要被“分块”，我们再结合刚才的图片来具体看看：</p>
<p data-nodeid="1494"><img src="https://s0.lgstatic.com/i/image/M00/80/19/CgqCHl_QcICANAhRAAHteGiEBK0162.png" alt="Drawing 3.png" data-nodeid="1701"></p>
<div data-nodeid="1495"><p style="text-align:center">图 4：histograms 和 distributions 页卡</p></div>
<p data-nodeid="1496">在 dense_1/bias_0 表格中，数据从上到下一共有 5 个切片。回顾一下代码可以知道，纵轴实际上代表了 5 个训练的 step。</p>
<blockquote data-nodeid="1497">
<p data-nodeid="1498">注意一下纵轴的坐标，分别是 0-5（这里只显示了 1 和 3）。</p>
</blockquote>
<p data-nodeid="1499">在 histogram 中，横轴表示要记录的信息的值，纵轴表示 step 或者记录的切片的计数，每个切片显示一个直方图，切片按步骤（步数或时间）排列：旧的切片较暗，新的切片颜色较浅。</p>
<p data-nodeid="1500">我们将鼠标放到表格上，会发现变成了如下的样子：</p>
<p data-nodeid="1501"><img src="https://s0.lgstatic.com/i/image/M00/80/19/CgqCHl_QcIiABV-AAACdbtCSv4A129.png" alt="Drawing 4.png" data-nodeid="1712"></p>
<div data-nodeid="1502"><p style="text-align:center">图 5：histogram 表格</p></div>
<p data-nodeid="1503">带黑色线条的直方图表示你此刻正在观察的切片，它对应了第 2 个 step，在此时有 635 个参数集中在以 0.0648 为中心的块中。</p>
<p data-nodeid="1504">其实，histogram 有两种模式：OVERLAY 和 OFFSET。咱们刚才学习的是 OFFSET，那么如何打开 OVERLAY 模式呢？给你留个小作业。</p>
<h4 data-nodeid="1505">tf.summary.image 与 tf.summary.text</h4>
<p data-nodeid="1506">如果只能记录参数信息，那么 TensorBoard 未免有些单调了。它还可以记录训练样本的内容信息，比如文本、图片。</p>
<p data-nodeid="1507">图片的记录，我们以刚才的 MNIST 为例子：</p>
<pre class="lang-python" data-nodeid="1508"><code data-language="python">(x_train, y_train),(x_test, y_test) = mnist.load_data()
log_dir=<span class="hljs-string">"./logs/"</span> + datetime.datetime.now().strftime(<span class="hljs-string">"%Y%m%d-%H%M%S"</span>)
summary_writer = tf.summary.create_file_writer(log_dir)
img = np.reshape(x_train[<span class="hljs-number">0</span>], (<span class="hljs-number">-1</span>, <span class="hljs-number">28</span>, <span class="hljs-number">28</span>, <span class="hljs-number">1</span>))
tf.summary.image(<span class="hljs-string">"Training image data"</span>, img, step=i)
</code></pre>
<p data-nodeid="1509">很简单， 我们打开 TensorBoard 之后就可以看到记录的图片了：</p>
<p data-nodeid="1510"><img src="https://s0.lgstatic.com/i/image/M00/80/0E/Ciqc1F_QcJOAJyOyAACjdT-BDSI231.png" alt="Drawing 5.png" data-nodeid="1721"></p>
<div data-nodeid="1511"><p style="text-align:center">图 6：TensorBoard 记录的图片</p></div>
<p data-nodeid="1512">同样， 文本可以用如下的方式：</p>
<pre class="lang-python" data-nodeid="1513"><code data-language="python">text = <span class="hljs-string">'tenboard is so cool!'</span>
<span class="hljs-keyword">with</span> summary_writer.as_default():
&nbsp; &nbsp; tf.summary.text(<span class="hljs-string">'fake_text'</span>, tf.convert_to_tensor(text), step=<span class="hljs-number">1</span>)
</code></pre>
<p data-nodeid="1514">你可能会问，记录 scalar 和 histogram 可以理解，因为它们能够展示参数的实时和历史变化，那记录 image 和 text 有什么用呢？</p>
<p data-nodeid="1515">其实，在模型的训练和预测过程中，咱们经常会遇到一些模型不太好学习的顽固分子（badcase），或是某些评估指标很低的类别的数据，它们都可以用这个 API 来记录和追踪，这总比跑完实验之后再一条一条去查来得方便和快捷。</p>
<h4 data-nodeid="1516">tf.summary 的其他 API</h4>
<p data-nodeid="1517">tf.summary 中还有一些其他的 API，我对它们做一个简单的介绍。</p>
<p data-nodeid="45173" class="te-preview-highlight"><strong data-nodeid="45178">tf.summary.audio</strong>：展示训练过程中记录的音频。在本门课程中，不涉及音视频相关的内容，所以如果你有兴趣，可以查阅资料来学习 TensorBoard 是如何记录音频内容的。</p>

<p data-nodeid="1519"><strong data-nodeid="1736">tf.summary.distribution</strong>：该函数的功能主要是用来记录 weights 分布信息的，与之前提到的 tf.summary.histogram 的功能有一定的相似之处。</p>
<p data-nodeid="1520"><strong data-nodeid="1743">tf.summary.merge</strong>：merge_all 可以将所有 summary 全部保存到磁盘，以便 TensorBoard 显示。</p>
<h3 data-nodeid="1521">总结</h3>
<p data-nodeid="1522">TensorBoard 是一个非常高效简便的实验记录和辅助工具，本讲中我介绍了它的基础使用方法，这些足够支持你日常的大部分使用场景和需求了。</p>
<p data-nodeid="1523">从第 01 讲到这里，咱们深度学习实战开发的基础知识做了相对全面的学习，包括数学基础、深度学习的概念与结构、辅助工具、开发工具等。至此，咱们已经有了足够的理论知识，来进入后面的课程了。那可是咱们的大戏：基于真实项目实战开发属于自己的深度学习模型。你准备好了吗？</p>
<p data-nodeid="1524" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/19/CgqCHl_QcLWARb8VAAUTv_jNlYg328.png" alt="槐树—深度学习.png" data-nodeid="1749"></p>

---

### 精选评论


