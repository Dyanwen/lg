<p data-nodeid="35730" class="">上一讲我向你介绍了 TensorFlow 中一些最基础的知识。这一讲，我会继续介绍 TensorFlow 的知识，不过会通过搭建一个手写数字识别的 CNN 训练代码的方式来进行。</p>


<h3 data-nodeid="34205">TensorFlow 2 的 API</h3>
<p data-nodeid="34206">我们先来看看 TensorFlow 中，几个比较常用的 API：tf.compact、tf.data、tf.image、tf.nn、tf.keras、tf.lite、tf.math。其余的 API 你可以学完之后去这个<a href="https://www.tensorflow.org/versions/r2.0/api_docs/python/tf" data-nodeid="34339">网址</a>中查看。</p>
<p data-nodeid="34207"><strong data-nodeid="34344">tf.compact</strong></p>
<p data-nodeid="34208">因为从 TensorFlow 1 到 TensorFlow 2 的升级中，API 发生了很大的变化，TensorFlow 团队为了保证 TensorFlow 1 的代码能够运行在 TensorFlow 2 上，为我们提供了迁移工具。TensorFlow 在1.13 的版本之后都会自动安装这个迁移工具，你可以通过以下命令将 Tensorflow 1 的代码升级到 Tensorflow 2。</p>
<pre class="lang-js" data-nodeid="37761"><code data-language="js">tf_upgrade_v2 \
&nbsp; --intree my_project/ \
&nbsp; --outtree my_project_v2/ \
&nbsp; --reportfile report.txt
</code></pre>


<p data-nodeid="39792" class="">脚本在运行的时候会将 TensorFlow 1 中的一些函数转移到 Tensorflow2 中的 tf.compat.v1 模块中，这就使得 TensorFlow 1 的代码仍然可以在 TensorFlow 2 中使用。但是官方有两个建议，一是这些替换最好自己人工校对一下，二是尽快切换到 TensorFlow 2 的 API 中。</p>


<p data-nodeid="34211"><strong data-nodeid="34350">tf.data</strong></p>
<p data-nodeid="34212">与数据读取相关的模块，我会在《<strong data-nodeid="34356">17 | 图像分类：实现你的第一个图像分类实战项目</strong>》中介绍。</p>
<p data-nodeid="34213"><strong data-nodeid="34360">tf.image</strong></p>
<p data-nodeid="34214">这个模块中，包含了与图像相关的处理，可以用来帮助我们做数据增强，或者图像的一些变化。</p>
<p data-nodeid="34215">例如，剪切、旋转、色彩饱和度增强等。</p>
<p data-nodeid="34216"><strong data-nodeid="34366">tf.nn</strong></p>
<p data-nodeid="34217">这里封装了一些神经网络最基础的操作。例如，卷积操作、池化操作、dropout、激活函数。</p>
<p data-nodeid="34218"><strong data-nodeid="34371">tf.keras</strong></p>
<p data-nodeid="34219">在 TensorFlow 2 以后，Keras 作为 TensorFlow 唯一 的高级API。Keras 是一个有前端与后端的机器学习库，后端可以在 TensorFlow、Theano 等平台上运行，前端有一个统一的 API，其特点是高度模块化，代码简单易读，这点在前文已经介绍过了。</p>
<p data-nodeid="34220">tf.keras 是被 TensorFlow 重新编写过的一套 Keras API，作为 TensorFlow 的一个子模块，它的后端只能是 TensorFlow。</p>
<p data-nodeid="34221"><strong data-nodeid="34377">tf.lite</strong></p>
<p data-nodeid="34222">你还记得我说过，TensorFlow 可以把模型部署到移动终端吗？我们只需要将我们训练好的模型转换为 tflite 格式即可。与 tflite 模型相关的 API 就在这个模块中。</p>
<p data-nodeid="34223"><strong data-nodeid="34382">tf.math</strong></p>
<p data-nodeid="34224">和它的名字一样，这里面包含了很多数学方法。</p>
<p data-nodeid="34225">当我们编写训练代码时，如果从 API 的封装程度上来说，TensorFlow 可以分为低级 API 与高级 API。低级 API 就是利用 tf.nn 中的 API 进行搭建，这里面的 API 偏向底层，较为灵活；高级 API 是 tf.keras 中的 API，这一部分的 API 经过封装，用起来更加容易一些，代码也更为易读。</p>
<p data-nodeid="34226">这一讲我先介绍“如何利用低级 API 构建网络与训练代码”，在《<strong data-nodeid="34390">17 | 图像分类：实现你的第一个图像分类实战项目</strong>》中我会向你介绍高级 API 的使用。</p>
<h3 data-nodeid="34227">利用低级 API 搭建手写数字识别网络</h3>
<p data-nodeid="34228">我们来构建一个卷积神经网络对手写数字进行分类，使用的数据依然是 MNIST。在着手训练之前，我们先来回忆一下，训练一个模型有哪些必不可少的要素。</p>
<ul data-nodeid="34229">
<li data-nodeid="34230">
<p data-nodeid="34231">数据：主要是训练集与评估集，用来训练与评估我们的模型。</p>
</li>
<li data-nodeid="34232">
<p data-nodeid="34233">网络结构：也就是我们模型的主体。</p>
</li>
<li data-nodeid="34234">
<p data-nodeid="34235">损失函数：更新模型参数的核心。</p>
</li>
<li data-nodeid="34236">
<p data-nodeid="34237">优化方法：更新模型参数的方法。</p>
</li>
</ul>
<p data-nodeid="34238">我们就从以上 4 个部分来了解，如何使用 TensorFlow 低级 API 构建一个模型。</p>
<h4 data-nodeid="34239">数据</h4>
<p data-nodeid="34240">MNIST 数据集已经被封装在 TensorFlow 中了，我们通过下面的代码可以将数据加载进来并查看：</p>
<pre class="lang-js" data-nodeid="41823"><code data-language="js">(X_train, Y_train), (X_test, Y_test) = mnist.load_data()
print(<span class="hljs-string">'train set:'</span>, X_train.shape, X_train.dtype, Y_train.shape, Y_train.dtype)
print(<span class="hljs-string">'test set:'</span>, X_test.shape, X_test.dtype, Y_test.shape,  Y_test.dtype)
<span class="hljs-attr">output</span>:
train set: (<span class="hljs-number">60000</span>, <span class="hljs-number">28</span>, <span class="hljs-number">28</span>) uint8 (<span class="hljs-number">60000</span>,) uint8
test set: (<span class="hljs-number">10000</span>, <span class="hljs-number">28</span>, <span class="hljs-number">28</span>) uint8 (<span class="hljs-number">10000</span>,) uint8
</code></pre>


<p data-nodeid="34242">原始数据是 uint8 类型，我们将它们转换为 float32 类型，并做归一化处理。</p>
<pre class="lang-js" data-nodeid="43853"><code data-language="js">X_train= np.array(X_train, np.float32)
X_test = np.array(X_test, np.float32)
# 归一化
X_train, X_test = X_train / 255, X_test / 255
</code></pre>


<p data-nodeid="34244">然后，我们将数据用 tf.data 封装。这里我们设定训练 2 个 Epoch，batchsize 大小为 128。如何使用 tf.data API，我会在《17 | 图像分类：实现你的第一个图像分类实战项目》中介绍。</p>
<pre class="lang-js" data-nodeid="45883"><code data-language="js">batch_size = 128
epoch = 2
# 使用 tf.data API 对数据进行随机排序和批处理
train_set = tf.data.Dataset.from_tensor_slices((X_train, Y_train))
train_set = train_set.repeat(epoch).shuffle(5000).batch(batch_size).prefetch(batch_size)
</code></pre>


<p data-nodeid="34246">到这里，数据读取就算完成了。</p>
<h4 data-nodeid="34247">网络结构</h4>
<p data-nodeid="34248">我们要构建下面这样的一个卷积神经网络。它是由 1 个卷积层与 1 个全连接层组成的：</p>
<p data-nodeid="46898" class=""><img src="https://s0.lgstatic.com/i/image/M00/7D/1C/CgqCHl_OBLKAOr9HAANDi58OqB8017.png" alt="Drawing 0.png" data-nodeid="46901"></p>

<div data-nodeid="47916" class=""><p style="text-align:center">图 1：卷积神经网络</p></div>

<p data-nodeid="34251">第一层为卷积层，是由 32 个 大小为3x3 的卷积核构成，我们可以使用 tf.nn.conv2d 来实现。</p>
<p data-nodeid="34252"><strong data-nodeid="34413">tf.nn.conv2d</strong></p>
<p data-nodeid="34253">我们来看一下 tf.nn.conv2d 的详细内容。</p>
<pre class="lang-js" data-nodeid="49942"><code data-language="js">tf.nn.conv2d(
&nbsp; &nbsp; input, filters, strides, padding, data_format=<span class="hljs-string">'NHWC'</span>, dilations=None, name=None
)
</code></pre>


<p data-nodeid="34255"><strong data-nodeid="34437">input</strong>：卷积操作的输入，必须是 half, bfloat16, float32, float64 类型。这也是为什么我们要将训练数据转换为 float32 类型。输入有 4 个维度：[batch_size, in_height, in_width, n_channels]。维度的顺序由 data_format 决定。在我们的例子中，Layer1 输入数据的维度为[128, 28, 28, 1]。</p>
<p data-nodeid="34256"><strong data-nodeid="34458">filters</strong>：卷积层中的卷积核，必须与输入数据有相同的数据类型。它也是一个 4 维的 tensor，[filter_height, filter_width, in_channels, out_channels]。在我们的例子中，Layer1 的卷积为[3, 3, 1, 28]。</p>
<p data-nodeid="34257"><strong data-nodeid="34463">strides</strong>：卷积移动的步长，是一个整数或者是整数类型的列表。列表的长度为 1，2 或者 4。如果只给 1 个值，那么默认是在 H 与 W 维度上使用相同的步长。N 与 C 的步长必须为 1。</p>
<p data-nodeid="34258"><strong data-nodeid="34536">padding</strong>：可使用'SAME'与'VALID'算法，也可以用一个列表指定各个维度 padding 的数目。当 data_format 是'NHWC'时，列表的内容依次为：[0, 0], [pad_top,pad_bottom], [pad_left, pad_right], [0, 0]。当 data_format 是'NCHW'时，列表的内容依次为：[0, 0], [0, 0],[pad_top, pad_bottom], [pad_left, pad_right]。</p>
<p data-nodeid="34259"><strong data-nodeid="34541">dilations</strong>：卷积扩张因子，可以是一个整数或者一个整形的列表。默认是 1，如果设置大于 1,则卷积的时候会跳过设定值-1 个元素进行卷积。</p>
<p data-nodeid="34260">我们来详细地看一下 padding 参数，用得比较多的还是直接指定的 same 或者 vaild。下面我介绍一下两者的区别。</p>
<blockquote data-nodeid="34261">
<p data-nodeid="34262">输入 tensor 的形状我们用**[batch, in_height, in_width, in_channels]<strong data-nodeid="34568">表示，输出的形状用</strong>[batch, out_height, out_width, out_channels]**表示。</p>
</blockquote>
<p data-nodeid="34263">当 padding 为 SAME 时，out_height 与 out_width 的计算方式为:</p>
<pre class="lang-js" data-nodeid="51968"><code data-language="js">out_height = ceil(in_height / stride_h)
out_width = ceil(out_width / stride_w)
</code></pre>


<p data-nodeid="34265">当 padding 为 VALID 时，out_height 与 out_width 的计算方式为:</p>
<pre class="lang-js" data-nodeid="56020"><code data-language="js">out_height = ceil((in_height - filter_height + <span class="hljs-number">1</span> )/ stride_h)
out_width = ceil((out_width - filter_width + <span class="hljs-number">1</span> ) / stride_w)
</code></pre>




<p data-nodeid="34267">我们可以做一下实验，从例子中来看 SAME 与 VALID 的区别：</p>
<pre class="lang-js" data-nodeid="58046"><code data-language="js"># MNIST 输入的 tensor
input = tf.Variable(tf.random.normal([1, 28, 28, 1]))
# 3x3 的卷积
filter = tf.Variable(tf.random.normal([3, 3, 1, 32]))
print(input.shape)
output = tf.nn.conv2d(input, filter, strides=[1, 3, 3, 1], padding='SAME')
print(output.shape)
output:
(1, 28, 28, 1)
(1, 10, 10, 32)
filter = tf.Variable(tf.random.normal([3, 3, 1, 32]))
print(input.shape)
output = tf.nn.conv2d(input, filter, strides=[1, 3, 3, 1], padding='VALID')
print(output.shape)
output:
(1, 28, 28, 1)
(1, 9, 9, 32)
</code></pre>


<p data-nodeid="34269">为了让卷积操作在我们的代码里看起来更简单一点，我们把 tf.nn.conv2d 包装一下，让它包含偏移项与激活函数，代码如下：</p>
<pre class="lang-js" data-nodeid="60072"><code data-language="js">def conv(x, W, b, strides=1,activation=tf.nn.relu):
&nbsp; &nbsp; # Conv2D 包装器, 带有偏置和 relu 激活
&nbsp; &nbsp; x = tf.nn.conv2d(x, W, strides=[1, strides, strides, 1], padding='SAME')
&nbsp; &nbsp; x = tf.nn.bias_add(x, b)
&nbsp; &nbsp; return activation(x)
</code></pre>


<p data-nodeid="34271">在 Layer1 的输出之后我们接一个 Max Pooling 操作进行降采样，我们使用 tf.nn.max_pool 在实现。</p>
<p data-nodeid="34272"><strong data-nodeid="34589">tf.nn.max_pool</strong></p>
<pre class="lang-js" data-nodeid="62098"><code data-language="js">tf.nn.max_pool(
&nbsp; &nbsp; input, ksize, strides, padding, data_format=None, name=None
)
</code></pre>


<p data-nodeid="34274">我们来看看各个参数的含义。</p>
<p data-nodeid="34275"><strong data-nodeid="34595">input</strong>：输入的 tensor。</p>
<p data-nodeid="34276"><strong data-nodeid="34600">ksize</strong>：是一个整形或者列表，指定 Pooling 的尺寸。</p>
<p data-nodeid="34277"><strong data-nodeid="34605">strides</strong>：是一个整形或者列表。Pooling 移动时的步长度。</p>
<p data-nodeid="34278"><strong data-nodeid="34618">padding</strong>：只可以是'VALID'或者'SAME'。</p>
<p data-nodeid="34279">为了代码的易读性，我们再封装一个 maxpooling 函数：</p>
<pre class="lang-js" data-nodeid="64124"><code data-language="js">def maxpooling(x, k=<span class="hljs-number">2</span>):
&nbsp; &nbsp; <span class="hljs-keyword">return</span> tf.nn.max_pool(x, ksize=[<span class="hljs-number">1</span>, k, k, <span class="hljs-number">1</span>], strides=[<span class="hljs-number">1</span>, k, k, <span class="hljs-number">1</span>], padding=<span class="hljs-string">'SAME'</span>)
</code></pre>


<p data-nodeid="34281"><strong data-nodeid="34623">全连接层</strong></p>
<p data-nodeid="34282">做完这些，我们就来到了图 1 最后的一个部分：全连接层。全连接层在低级 API 中没有直接的 API，需要我们自己实现。</p>
<p data-nodeid="65137" class=""><img src="https://s0.lgstatic.com/i/image/M00/7D/11/Ciqc1F_OBNqAN3LVAALl1acWu-w634.png" alt="Drawing 1.png" data-nodeid="65140"></p>

<p data-nodeid="34284">请看上图，经过 maxpooling 之后的 feature map 的大小为（14, 14, 32）。为了后面的全连接计算，我们需要将它 reshape 到[-1, 14<em data-nodeid="34639">14</em>32]。第一个维度是-1，因为这里的大小要自动适配 batch_size 的大小。代码如下：</p>
<pre class="lang-plain" data-nodeid="34285"><code data-language="plain"># conv1 是 maxpooling 的输出，一会儿可以看整个网络的定义
fc1 = tf.reshape(conv1, [-1, fc1_w.get_shape().as_list()[0]])
</code></pre>
<p data-nodeid="34286">接下来就是我在《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=522#/detail/pc?id=4976" data-nodeid="34643">02 | 从神经元说起：结构篇</a>》中介绍的人工神经网络（在 CNN 中叫作全连接层）。我们的第一层全连接层有 1024 个神经元，所以有 1024 个输出。</p>
<p data-nodeid="34287">权重的形状为（14 * 14 * 32, 1024），偏移项是一个长度为 1024 的向量。</p>
<p data-nodeid="34288">第二个全连接层的输入是第一层的输出，所以第二层全连接层权重的形状为（1024, 10），偏移项是一个长度为 10 的向量。</p>
<p data-nodeid="34289">最后，因为是一个多分类问题，所以以 softmax 返回。请看下面的代码：</p>
<pre class="lang-js" data-nodeid="67166"><code data-language="js"># 全连接层， 输出形状： [batch_size, 1024]
fc1 = tf.add(tf.matmul(fc1, fc1_w), fc1_b)
# 将 ReLU 应用于 fc1 输出以获得非线性
fc1 = tf.nn.relu(fc1)
# 全连接层，输出形状 [batch_size, num_classes]
out = tf.add(tf.matmul(fc1, fc2_w), fc2_b)
return tf.nn.softmax(out)
</code></pre>


<p data-nodeid="34291">整个网络结构，可以参考下面的代码：</p>
<pre class="lang-js" data-nodeid="69192"><code data-language="js">def net(x):
&nbsp; &nbsp; # 输入形状：[batch_size, 28, 28, 1]
&nbsp; &nbsp; x = tf.reshape(x, [-1, 28, 28, 1])
&nbsp; &nbsp; # 输出形状：[batch_size, 28, 28 ,32]
&nbsp; &nbsp; conv1 = conv(x, conv_w, conv_b)
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; # maxpooling 输出形状：[batch_size, 14, 14, 32]
&nbsp; &nbsp; conv1 = maxpooling(conv1, k=2)
&nbsp; &nbsp; # 对 conv1 进行 reshape， 输出形状：[batch_size, 14*14*32]
&nbsp; &nbsp; fc1 = tf.reshape(conv1, [-1, fc1_w.get_shape().as_list()[0]])
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; # 全连接层， 输出形状： [batch_size, 1024]
&nbsp; &nbsp; fc1 = tf.add(tf.matmul(fc1, fc1_w), fc1_b)
&nbsp; &nbsp; &nbsp;# 将 ReLU 应用于 fc1 输出以获得非线性
&nbsp; &nbsp; fc1 = tf.nn.relu(fc1)
&nbsp; &nbsp; # 全连接层，输出形状 [batch_size, num_classes]
&nbsp; &nbsp; out = tf.add(tf.matmul(fc1, fc2_w), fc2_b)
&nbsp; &nbsp; return tf.nn.softmax(out)
</code></pre>


<p data-nodeid="34293">使用低级 API 的时候，需要注意一点：<strong data-nodeid="34658">我们使用 tf.nn.conv2d、全连接层或者其他 API 中，如果需要参数（卷积核、权重、偏移项等），需要我们自己手动创建</strong>。所以在我们定义网络之前，需要先使用变量定义好我们的参数。请看下面的代码：</p>
<pre class="lang-js" data-nodeid="71218"><code data-language="js">num_classes = 10
# 随机值生成器初始化权重
random_normal = tf.initializers.RandomNormal()
# 第一层卷积层的权重：3 * 3 卷积，1 个输入，32 个卷积核
conv_w = tf.Variable(random_normal([3, 3, 1, 32]))
# 第一层卷积层的偏移
conv_b = tf.Variable(tf.zeros([32]))
# fc1：14*14*32 个输入，1024 个神经元
# 第一个全连接层的权重
fc1_w = tf.Variable(random_normal([14 * 14 * 32, 1024]))
# 第一个全连接层的权重
fc1_b&nbsp; = tf.Variable(tf.zeros([1024]))
# fc2: 1024 个输入，10 个神经元
# 第二个全连接层的权重
fc2_w = tf.Variable(random_normal([1024, num_classes]))
# 第二个全连接层的权重
fc2_b&nbsp; = tf.Variable(tf.zeros([10]))
</code></pre>


<p data-nodeid="34295">到这里我们就完成了整个网络结构的搭建，我们来看看损失函数是如何设定的。</p>
<h4 data-nodeid="34296">损失函数</h4>
<p data-nodeid="34297">低级 API 中没有直接可以用的损失函数，虽然官方的 API 中有 tf.losses 模块，但是点进去会自动跳转到 tf.keras.losses 中。</p>
<p data-nodeid="34298">在这个问题中我们使用交叉熵损失函数，具体定义可以回顾《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=522#/detail/pc?id=4978" data-nodeid="34665">04 | 函数与优化方法：模型的自我学习（上）</a>》中的内容。因为交叉熵损失比较简单，所以我们可以自己手写一个损失函数。</p>
<p data-nodeid="34299">我们模型返回的是一个 10 个维度 tensor，tensor 的内容是经过 softmax 后获得的概率。但是我们的真实 label 是具体的数字，所以我们先试用 tf.one_hot 对真实标签进行一个次 one_hot 编码。代码如下：</p>
<pre class="lang-js" data-nodeid="73244"><code data-language="js"># 交叉熵损失函数
def cross_entropy(y_pred, y_true):
&nbsp; &nbsp; y_true = tf.one_hot(y_true, depth=num_classes)
&nbsp; &nbsp; # 计算交叉熵
&nbsp; &nbsp; return tf.math.reduce_mean(-tf.math.reduce_sum(y_true * tf.math.log(y_pred)))
</code></pre>


<p data-nodeid="34301">有几个常用的数学方法：tf.math.reduce_sum、tf.math.reduce_mean、tf.math.reduce_max 和 tf.math.reduce_min，它们分别是沿着某一个轴求合、求均值、求最大值以及求最小值。</p>
<p data-nodeid="34302">最后，就来到了优化函数。</p>
<h4 data-nodeid="34303">优化函数</h4>
<p data-nodeid="34304">与 tf.losses 相同，在 API 中你可以看到 tf.optimizers，但是你点进去之后会自动跳转到 tf.keras.optimizers。</p>
<p data-nodeid="34305">优化方法我们选择我们熟知的 SGD。SGD 函数如下：</p>
<pre class="lang-js" data-nodeid="75270"><code data-language="js">tf.keras.optimizers.SGD(
&nbsp; &nbsp; learning_rate=<span class="hljs-number">0.01</span>, momentum=<span class="hljs-number">0.0</span>, nesterov=False, name=<span class="hljs-string">'SGD'</span>, **kwargs
)
</code></pre>


<p data-nodeid="34307">当 momentum=0 时，按照下面的公式来更新权重 w：</p>
<pre class="lang-js" data-nodeid="77296"><code data-language="js">w = w - learning_rate * gradient
</code></pre>


<p data-nodeid="34309">当 momentum&gt;0 时，按照下面的公式来更新权重 w：</p>
<pre class="lang-js" data-nodeid="79322"><code data-language="js">velocity = momentum * velocity - learning_rate * g
w = w * velocity
</code></pre>


<p data-nodeid="34311">初始速度 velocity 为 0。</p>
<p data-nodeid="34312">当 nesterov=False 时，权重的更新方式变为：</p>
<pre class="lang-js" data-nodeid="81348"><code data-language="js">velocity = momentum * velocity - learning_rate * g
w = w + momentum * velocity - learning_rate * g
</code></pre>


<p data-nodeid="34314">在我们的代码中，只设定学习率，一般设定为 0.001，但这也不是绝对的，其余的用默认值。</p>
<pre class="lang-js" data-nodeid="85400"><code data-language="js">optimizer = tf.keras.optimizers.SGD(learning_rate)
</code></pre>




<p data-nodeid="34316">神经网络依赖反向传播求梯度来更新网络参数，所以接下来还剩计算梯度。</p>
<p data-nodeid="34317">TensorFlow 为我们提供了自动微分的机制，这就是我们之前讲过的，为什么要使用框架的原因，除非你喜欢自己写程序进行求导。</p>
<p data-nodeid="34318"><strong data-nodeid="34695">自动微分</strong></p>
<p data-nodeid="34319">TensorFlow 提供了一个 tf.GradientTape API，使用 tf.GradientTape 可以为我们自动计算某一计算关系中因变量相对于某些自变量的梯度值。tf.GradientTape 会自动记录前向传播的过程，并且利用反向传播自动计算出梯度值。</p>
<p data-nodeid="34320">用 tf.GradientTap 记录下某些操作后，就可以使用 GradientTap.gradient（target, sources）计算梯度了。这个 target 通常是损失函数。</p>
<p data-nodeid="34321">我们看一下下面这个例子：</p>
<pre class="lang-js" data-nodeid="87426"><code data-language="js">x = tf.Variable(3.0)
with tf.GradientTape() as tape:
&nbsp; y = x**2
# dy = 2x * dx
dy_dx = tape.gradient(y, x)
dy_dx.numpy()
</code></pre>


<p data-nodeid="34323">例子中，先在 tf.GradientTape 中记录与 y 相关的操作，然后利用 GradientTap.gradienty（y, x）计算 y 在 x=3 时的梯度值。</p>
<p data-nodeid="34324">计算完梯度之后，我们需要使用优化器的 apply_gradients 更新参数。在 TensorFlow 2 中无论是计算梯度，还是应用梯度都需要显式的指定。</p>
<p data-nodeid="34325">我们将梯度计算相关的部分应用到我们的模型训练代码中，如下所示：</p>
<pre class="lang-js te-preview-highlight" data-nodeid="89452"><code data-language="js">def run_optimization(x, y):
&nbsp; &nbsp; # 将计算封装在 GradientTape 中以实现自动微分
&nbsp; &nbsp; with tf.GradientTape() as g:
&nbsp; &nbsp; &nbsp; &nbsp; pred = net(x)
&nbsp; &nbsp; &nbsp; &nbsp; loss = cross_entropy(pred, y)
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; # 要更新的变量，就是网络中需要训练的参数
&nbsp; &nbsp; trainable_variables = [conv_w, conv_b, fc1_w, fc1_b, fc2_w, fc2_b]
&nbsp; &nbsp; # 计算梯度
&nbsp; &nbsp; gradients = g.gradient(loss, trainable_variables)
&nbsp; &nbsp;&nbsp;
&nbsp; &nbsp; # 按 gradients 更新参数
&nbsp; &nbsp; optimizer.apply_gradients(zip(gradients, trainable_variables))
</code></pre>


<p data-nodeid="34327">接下来，我们只需要不断读取数据调用 run_optimization 函数来计算梯度、更新参数就可以了。</p>
<p data-nodeid="34328">到这里，我们就算利用低级 API 搭建了一个手写数字识别网络。</p>
<h3 data-nodeid="34329">总结</h3>
<p data-nodeid="34330">这一讲我向你介绍了如何使用 Tensorlfow 低级 API 来搭建网络并训练网络，在接下来的实战章节中我还会继续向你介绍 Tensorlfow 相关的知识。</p>
<p data-nodeid="34331">以上，我们搭建了一个包含全连接层的手写数字识别网络，你可以把网络修改成不使用全连接层的网络吗？欢迎在留言区以及在交流群中分享你的问题和回答。</p>
<p data-nodeid="34332">下一讲，我将带你了解 Tensorboard，它可以帮助我们在实验中分析与发现问题。</p>

---

### 精选评论

##### **强：
> 有源码吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 本章节的代码组合起来就已经是核心代码了。此外，手写识别在咱们课程中已经反复提及了哟，现在的你已经完全能够写出属于自己的手写识别代码了～试一试。

