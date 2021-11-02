<p data-nodeid="93097">从 Pipeline 的角度来说，上一个课时我们主要学习的是 Transformer，从本课时开始将进入 Estimator 的学习，也就是机器学习算法的学习。从本课时开始，我们将从 MLlib 实现的算法中按照算法类型：分类（有监督）、聚类（无监督）、推荐算法，各选取一个算法进行学习，并且<strong data-nodeid="93191">每个课时后还会有基于真实数据的实践训练</strong>，这是本模块与其他模块不同的地方。</p>
<p data-nodeid="93098">上节课的思考题可以看成是本课时的预习，在下面的内容中，我们会通过案例解答这个问题。</p>
<p data-nodeid="93099">分类器是机器学习最常见的应用，MLlib 中也内置了许多分类模型，而其中支持分布式计算最好的也是最常用的方法就是随机森林算法，它的基础是决策树算法。本课时，我将介绍决策树算法和随机森林算法，以及用 Spark MLlib 的随机森林分类器实现根据身体监控数据判断人体状态。</p>
<h3 data-nodeid="93100">决策树</h3>
<p data-nodeid="93101">决策树是一种机器学习的方法，它通过一种树形结构对样本进行分类，每个非叶子结点代表一次判断，每个叶子结点代表的是分类结果。它是一种<strong data-nodeid="93200">典型的监督学习</strong>，需要一定量的样本，常见的决策树构造树算法有 C4.5 与 ID3。</p>
<p data-nodeid="93102">下面先来看一个例子。下表中的内容是信贷审批常见的场景：根据信息判断客户是否会逾期。它的样本一共有 4 个特征，最后得出“是否逾期”的结果，这是一个二分类场景。</p>
<p data-nodeid="93103"><img alt="2.png" src="https://s0.lgstatic.com/i/image/M00/3E/30/CgqCHl8rs1yAeh-3AADxNFeYbKs151.png" data-nodeid="93204"></p>
<p data-nodeid="93104">根据这些样本，我们可以构造出一棵这样的树，如下图所示。</p>
<p data-nodeid="93105"><img alt="1.png" src="https://s0.lgstatic.com/i/image/M00/3E/25/Ciqc1F8rs22AZdTQAAFhkujn7lI795.png" data-nodeid="93208"></p>
<p data-nodeid="93106">构造一棵决策树需要从样本中学习结点分裂的时机，并判断阈值，也就是上图中的 A、B、C、D、E，这个过程被称为特征选择。通过这样的方法，我们可以达到对样本分类的效果。</p>
<p data-nodeid="93107">下面我们先来简要介绍一下决策树构造算法的整个过程。决策树构造算法通常是先选择一个最优特征，将样本分为若干个子集，如果子集已经被正确分类，那么就构造叶子结点，将样本分到叶子结点中；如果某个子集没有被正确分类，那么就对这个子集继续进行选择特征。决策树构造是一个递归过程，递归停止条件是所有样本被基本正确分类，这样就构造出了一棵决策树。</p>
<p data-nodeid="93108">从这个过程中可以看出，决策树通常对训练数据有良好的表现，但对新样本却未必如此，容易出现过拟合，也就是说模型能将训练数据很好地正确分类，而对测试数据和真实数据来说，分类结果却不尽如人意 。因此我们需要对已经生成的树进行剪枝，从而提升模型的泛化能力。如果特征过多，在一开始构造的时候，我们也会对特征进行选择，只留下足够有区分度的特征。决策树的生成对应模型的局部选择，而剪枝对应模型的全局选择。</p>
<p data-nodeid="93109">从上述过程可以看出，构造决策树主要包含<strong data-nodeid="93225">特征选择</strong>、<strong data-nodeid="93226">决策树生成</strong>与<strong data-nodeid="93227">决策树剪枝</strong>。下面我们将来详细介绍。</p>
<h4 data-nodeid="93110">特征选择</h4>
<p data-nodeid="93111">本节主要介绍两种特征选择的方式：<strong data-nodeid="93238">信息增益</strong>与<strong data-nodeid="93239">信息增益率</strong>。在介绍这两种方式前，先来看看熵的概念：熵是表示随机变量的不确定性的度量。</p>
<p data-nodeid="93112"><strong data-nodeid="93302">定义</strong>：假设随机变量 <em data-nodeid="93303">X</em> 的可能取值有 <i>x<sub>1</sub> , x<sub>2</sub> , …, x<sub>n</sub></i> ，对于每一个可能的取值 <i>x<sub>i</sub></i> ，其概率 <em data-nodeid="93304">P（X</em> = <em data-nodeid="93305">x<sub>i</sub>）</em> = <em data-nodeid="93306">p<sub>i</sub> （</em> <em data-nodeid="93307">i</em> = 1, 2, …, <em data-nodeid="93308">n）</em> ，则随机变量 <em data-nodeid="93309">X</em> 的熵为：</p>
<p data-nodeid="93113"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/3E/36/CgqCHl8ruuSATGy3AAAmmKUzEis127.png" data-nodeid="93312"></p>
<p data-nodeid="93114">对于样本集合 <em data-nodeid="93359">D</em> 来说，随机变量 <em data-nodeid="93360">X</em> 是样本的类别，即假设样本有 <em data-nodeid="93361">k</em> 个类别，每个类别的概率是<br>
<img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/3E/36/CgqCHl8ruwWASQ_iAAANoL6crhU032.png" data-nodeid="93329"><br>
，其中 | <i>C<sub>k</sub></i> | 表示类别 <em data-nodeid="93362">k</em> 的样本个数，| <em data-nodeid="93363">D</em> | 表示样本总数，对于样本集合 <em data-nodeid="93364">D</em> 来说，熵（经验熵）为：</p>
<p data-nodeid="93115"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/3E/37/CgqCHl8ruxOAHrq8AABK5ZnXoS8805.png" data-nodeid="93367"></p>
<p data-nodeid="93116">而条件熵的概念为：设有随机变量（ <em data-nodeid="93377">X</em> , <em data-nodeid="93378">Y)</em>，其联合概率分布为：</p>
<p data-nodeid="93117"><img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image/M00/3E/37/CgqCHl8ruxqAOg4aAABq0voqPeg077.png" data-nodeid="93381"></p>
<p data-nodeid="93118">条件熵 <em data-nodeid="93435">H</em>（<em data-nodeid="93436">Y</em> | *X）*表示在已知随机变量 <em data-nodeid="93437">X</em> 的条件下，随机变量 <em data-nodeid="93438">Y</em> 的不确定性。在随机变量 <em data-nodeid="93439">X</em> 给定的条件下，随机变量 <em data-nodeid="93440">Y</em> 的条件熵 <em data-nodeid="93441">H（Y</em> | <em data-nodeid="93442">X）</em>，定义为 <em data-nodeid="93443">X</em> 给定条件下 <em data-nodeid="93444">Y</em> 的条件概率分布的熵对 <em data-nodeid="93445">X</em> 的数学期望：</p>
<p data-nodeid="93119"><img alt="Drawing 6.png" src="https://s0.lgstatic.com/i/image/M00/3E/37/CgqCHl8ruyKAcbBLAABJvNn0hIA847.png" data-nodeid="93448"></p>
<p data-nodeid="93120">当熵和条件熵中的概率由数据估计得到时，所对应的熵与条件熵分别称为经验熵与经验条件熵。从经验熵与经验条件熵可以得到<strong data-nodeid="93454">信息增益的定义</strong>：</p>
<p data-nodeid="93121">特征 <em data-nodeid="93492">A</em> 对训练数据集 <em data-nodeid="93493">D</em> 的信息增益 <em data-nodeid="93494">g</em> （<em data-nodeid="93495">D</em> , <em data-nodeid="93496">A）</em> 的定义，是集合 <em data-nodeid="93497">D</em> 的经验熵 <em data-nodeid="93498">H（D)</em> 与特征 <em data-nodeid="93499">A</em> 给定条件下 <em data-nodeid="93500">D</em> 的经验熵与条件熵之差：</p>
<p data-nodeid="93122"><img alt="Drawing 7.png" src="https://s0.lgstatic.com/i/image/M00/3E/37/CgqCHl8ruyqAQoMqAABDo99VV2A633.png" data-nodeid="93503"></p>
<p data-nodeid="93877">信息增益通常用来选择特征，经验熵 <em data-nodeid="93921">H（D）</em> 表示的是对数据集 <em data-nodeid="93922">D</em> 进行分类的不确定性。而经验条件熵 <em data-nodeid="93923">H（D</em> | <em data-nodeid="93924">A）</em> 表示在特征 <em data-nodeid="93925">A</em> 给定的条件下，对数据集 <em data-nodeid="93926">D</em> 进行分类的不确定性，那么信息增益就表示由于特征 <em data-nodeid="93927">A</em> 而使得对数据集 <em data-nodeid="93928">D</em> 的分类的不确定性的减少程度。显然，对于数据集 <em data-nodeid="93929">D</em> 而言，信息增益依赖于特征，不同特征往往具有不同的信息增益。信息增益大的特征具有更强的分类能力。根据信息增益准则，选择特征的方法是：对训练数据集（或子集） <em data-nodeid="93930">D</em>，计算其每个特征的信息增益，并选择信息增益最大的特征。</p>

<p data-nodeid="93124">信息增益率是对信息增益的改进，特征 <em data-nodeid="93598">A</em> 对训练数据集 <em data-nodeid="93599">D</em> 的信息增益率 <em data-nodeid="93600">g<sub>R</sub>（D</em> , <em data-nodeid="93601">A）</em> 的定义，是其信息增益 <em data-nodeid="93602">g（D</em> , <em data-nodeid="93603">A）</em> 与训练数据集 <em data-nodeid="93604">D</em> 的经验熵 <em data-nodeid="93605">H</em> （<em data-nodeid="93606">D）</em> 之比，如下图所示：</p>
<p data-nodeid="93125"><img alt="Drawing 8.png" src="https://s0.lgstatic.com/i/image/M00/3E/2B/Ciqc1F8ruzGAPP1YAABNXeHlZfw189.png" data-nodeid="93609"></p>
<p data-nodeid="93126">除了信息增益与信息增益率之外，还可以用基尼系数来完成特征选择。</p>
<h4 data-nodeid="93127">决策树生成</h4>
<p data-nodeid="93128">决策树生成算法与特征选择方法相对应，选用信息增益进行特征选择的是 ID3 算法，选择信息增益比进行特征选择的是&nbsp;C4.5&nbsp;算法，选择基尼系数来完成特征选择的是分类回归树（CART）算法。本节将介绍 C4.5 与 ID3 算法。</p>
<p data-nodeid="93129"><strong data-nodeid="93616">ID3 算法如下：</strong></p>
<p data-nodeid="93130">简单来说，ID3 算法会在决策树各个结点上应用信息增益准则选择特征，递归地构建决策树。</p>
<p data-nodeid="93131">给定训练数据集 <em data-nodeid="93631">D</em>，特征集 <em data-nodeid="93632">S</em>，阈值 <em data-nodeid="93633">ϵ</em> ：</p>
<ol data-nodeid="116575">
<li data-nodeid="116576">
<p data-nodeid="116577">若 <em data-nodeid="116611">D</em> 中所有实例属于同一类 <i>C<sub>k</sub></i>，则 <em data-nodeid="116612">T</em> 为单结点树，并将类 <i>C<sub>k</sub></i> 作为该结点的类标记，返回 T；</p>
</li>
<li data-nodeid="116578">
<p data-nodeid="116579">若 <em data-nodeid="116637">S</em> =&nbsp;Æ，则 <em data-nodeid="116638">T</em> 为单结点树，并将 <em data-nodeid="116639">D</em> 中实例数最大的类 <i>C<sub>k</sub></i> 作为该结点的类标记，返回 <em data-nodeid="116640">T</em>；</p>
</li>
<li data-nodeid="116580">
<p data-nodeid="116581">否则，计算 <em data-nodeid="116657">S</em> 中各特征对 <em data-nodeid="116658">D</em> 的信息增益，选择信息增益最大的特征 <i>S<sub>g</sub></i>；</p>
</li>
<li data-nodeid="116582">
<p data-nodeid="116583">如果 <i>S<sub>g</sub></i> 的信息增益小于阈值 <em data-nodeid="116690">ϵ</em>，则置 <em data-nodeid="116691">T</em> 为单结点树，并将 <em data-nodeid="116692">D</em> 中实例数最大的类 <i>C<sub>k</sub></i> 作为该结点的类标记，返回 <em data-nodeid="116693">T</em>；</p>
</li>
<li data-nodeid="116584">
<p data-nodeid="116585">否则，对 <i>S<sub>g</sub></i> 的每一个可能值 <i>a<sub>i</sub></i>，将 <em data-nodeid="116735">D</em> 分割为若干个非空子集 <i>D<sub>i</sub></i>，将 <i>D<sub>i</sub></i> 中实例数最大的类作为标记，构建子结点，由结点及其子结点构成树 <em data-nodeid="116736">T</em>，返回 <em data-nodeid="116737">T</em>；</p>
</li>
<li data-nodeid="116586">
<p data-nodeid="116587">对第 <em data-nodeid="116771">i</em> 个子结点，以 <i>D<sub>i</sub></i> 为训练集，以 <i>S -S<sub>g</sub></i> 为特征集，递归调用第 1～5 步，得到子树 <i>T<sub>i</sub></i>，返回 <i>T<sub>i</sub></i>。</p>
</li>
</ol>













<p data-nodeid="93145"><strong data-nodeid="93822">C4.5 算法</strong>与 ID3 算法非常类似，只是把用到信息增益的地方换成了信息增益比。</p>
<h4 data-nodeid="93146">剪枝</h4>
<p data-nodeid="93147">通常决策树在训练数据上表现很好，但是在测试数据上就不尽如人意，这就是模型过拟合。决策树剪枝主要分为预剪枝和后剪枝。<strong data-nodeid="93833">预剪枝</strong>是在构造决策树的同时进行剪枝，通常作为停止条件，即设定一个熵的阈值，就算可以继续降低熵，也停止创建分支。而<strong data-nodeid="93834">通常我们说的剪枝是指后剪枝</strong>，后剪枝通常有以下两种做法：</p>
<ul data-nodeid="93148">
<li data-nodeid="93149">
<p data-nodeid="93150"><strong data-nodeid="93839">应用交叉验证的思想</strong>，若局部剪枝能够使得模型在测试集上的错误率降低，则进行局部剪枝。</p>
</li>
<li data-nodeid="93151">
<p data-nodeid="93152"><strong data-nodeid="93844">应用正则化的思想</strong>，综合考虑不确定性和模型复杂度来定出一个新的损失，用该损失作为一个结点是否应该局部剪枝的标准。这种做法的核心是定义新的代价函数，通常会采用树的结构复杂度与模型预测误差之和作为代价衡量。</p>
</li>
</ul>
<p data-nodeid="93153">在 ID3、C4.5 中我们会应用前者做法，而在分类回归树中，我们会采取后者做法。</p>
<h3 data-nodeid="93154">随机森林</h3>
<p data-nodeid="93155">在决策树的基础上了解随机森林的原理相对容易。随机森林就是通过集成学习的思想将多棵树集成的一种算法，它的基本单元是决策树，属于机器学习的一大分支——集成学习（Ensemble Learning）方法。集成学习是通过构建多个弱分类器，并按一定规则组合起来的分类系统，常常比单一分类器具有显著优越的泛化性能，常见集成学习算法有随机森林、AdaBoost、XgBoost、梯度提升树等，在风险建模、疾病预测等领域应用相当广泛。</p>
<p data-nodeid="93156">随机森林将 N 棵决策树集成，每一棵决策树都是一个分类器，相当于每个分类器都会对结果进行投票，随机森林会综合所有的分类结果，并将票数最高的分类结果作为最终结果输出。可以想到，在随机森林中，每一棵决策树的生成是算法的关键。<strong data-nodeid="93853">每棵树的生成规则如下</strong>：</p>
<ol data-nodeid="119905">
<li data-nodeid="119906">
<p data-nodeid="119907">如果训练集大小为 N，对每棵树而言，随机且有放回地从训练集中抽取 N 个训练样本（这种采样方式称为 Bootstrap Sample 方法），作为该树的训练集。</p>
</li>
<li data-nodeid="119908">
<p data-nodeid="119909">如果每个样本的特征维度为 M，指定一个常数 m&lt;&lt;M，随机地从 M 个特征中选取 m 个特征子集，每次树进行分裂时，从这 m 个特征中选择最优的。</p>
</li>
<li data-nodeid="119910">
<p data-nodeid="119911">每棵树都尽最大限度地生长，并且没有剪枝过程。</p>
</li>
</ol>


<p data-nodeid="93164">随机森林中所谓“随机”的含义，就是模型在这里引入了随机性（随机抽取训练集、随机抽取特征），两个随机性的引入对随机森林的分类性能至关重要。由于它们的引入，使得随机森林不容易陷入过拟合，并且具有很好的抗噪能力。随机森林的特点有：</p>
<ul data-nodeid="93165">
<li data-nodeid="93166">
<p data-nodeid="93167">在当前所有算法中，具有极佳的准确率，在国内外最近几年的数据挖掘大赛中，随机森林取得了令人瞩目的成绩。</p>
</li>
<li data-nodeid="93168">
<p data-nodeid="93169">能够高效地运行在大数据集上，很容易可以看出，随机森林是非常容易分布式的。</p>
</li>
<li data-nodeid="93170">
<p data-nodeid="93171">能够处理具有高维特征的输入样本。</p>
</li>
<li data-nodeid="93172">
<p data-nodeid="93173">能够评估各个特征在分类问题上的重要性。</p>
</li>
<li data-nodeid="93174">
<p data-nodeid="93175">在生成过程中，能够获取到内部生成误差的一种无偏估计。</p>
</li>
<li data-nodeid="93176">
<p data-nodeid="93177">对于缺省值问题也能够获得很好的结果。</p>
</li>
</ul>
<h3 data-nodeid="93178">人体状态监测器</h3>
<p data-nodeid="121478">下面，我们将用真实的数据（数据集下载链接： <a href="https://pan.baidu.com/s/1rUxHKl119qGDlV6KeTtkLA" data-nodeid="121482">https://pan.baidu.com/s/1rUxHKl119qGDlV6KeTtkLA</a>提取码：ev1d）拟合出一个随机森林分类器。案例的内容是通过身体监测数据来判断人体状态，比如监测走路、骑行、跑步、看电视。数据集包括时间戳、心跳、活动标签和 3 个传感器，传感器分别佩戴在手上、胸部、踝关节处，每个传感器有 17 个检测指标 （温度、3D 加速度、陀螺仪和磁强计数据、方位数据） 。数据集共计 54 个属性，3850505 条样本，包含了 18 种人体活动。代码如下：</p>

<pre class="lang-scala" data-nodeid="93180"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.<span class="hljs-type">Pipeline</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.classification.{<span class="hljs-type">RandomForestClassificationModel</span>,&nbsp;<span class="hljs-type">RandomForestClassifier</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.evaluation.<span class="hljs-type">MulticlassClassificationEvaluator</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.{<span class="hljs-type">Dataset</span>,&nbsp;<span class="hljs-type">Row</span>,&nbsp;<span class="hljs-type">SparkSession</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.rdd.<span class="hljs-type">RDD</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.mllib.regression.<span class="hljs-type">LabeledPoint</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.mllib.linalg.<span class="hljs-type">Vectors</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.{<span class="hljs-type">IndexToString</span>,&nbsp;<span class="hljs-type">StringIndexer</span>,&nbsp;<span class="hljs-type">VectorAssembler</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.types._ 
<span class="hljs-keyword">import</span>&nbsp;scala.collection.mutable 

<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;RandomForestBodyDetection&nbsp;</span></span>{ 

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span> 
&nbsp;&nbsp;&nbsp;&nbsp;.builder() 
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"RandomForestBodyDetection"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.enableHiveSupport() 
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">import</span>&nbsp;spark.implicits._ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;读取数据集 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;dataFiles&nbsp;=&nbsp;spark.read.textFile(<span class="hljs-string">"data/bodydetect"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;rawData&nbsp;=&nbsp;dataFiles.map(r=&gt;r.toString().split(<span class="hljs-string">"&nbsp;"</span>)).rdd.map(row&nbsp;=&gt;&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;list&nbsp;=&nbsp;mutable.<span class="hljs-type">ArrayBuffer</span>[<span class="hljs-type">Any</span>]()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(i&nbsp;&lt;-&nbsp;row.toSeq)&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;list.append(i) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">Row</span>.fromSeq(list.map(v=&gt;<span class="hljs-keyword">if</span>&nbsp;(v.toString.toUpperCase&nbsp;==&nbsp;<span class="hljs-string">"NAN"</span>)&nbsp;<span class="hljs-type">Double</span>.<span class="hljs-type">NaN</span>&nbsp;<span class="hljs-keyword">else</span>&nbsp;v.toString.toDouble)) 
&nbsp;&nbsp;&nbsp;&nbsp;}) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;schema&nbsp;=&nbsp;<span class="hljs-type">StructType</span>(<span class="hljs-type">Array</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"timestamp"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"activityId"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hr"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_temp"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel1X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel1Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel1Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel2X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel2Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_accel2Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_gyroX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_gyroY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_gyroZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_magnetX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_magnetY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_magnetZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_orientX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_orientY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_orientZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"hand_orientD"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_temp"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel1X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel1Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel1Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel2X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel2Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_accel2Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_gyroX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_gyroY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_gyroZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_magnetX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_magnetY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_magnetZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_orientX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_orientY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_orientZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"chest_orientD"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_temp"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel1X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel1Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel1Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel2X"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel2Y"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_accel2Z"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_gyroX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_gyroY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_gyroZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_magnetX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_magnetY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_magnetZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_orientX"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_orientY"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_orientZ"</span>,&nbsp;<span class="hljs-type">DoubleType</span>),&nbsp;<span class="hljs-type">StructField</span>(<span class="hljs-string">"ankle_orientD"</span>,&nbsp;<span class="hljs-type">DoubleType</span>))) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;df&nbsp;=&nbsp;spark.createDataFrame(rawData,schema) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;数据集的列名，sensor_name&nbsp;表示某个传感器的某个指标数据，例如，手上的传感器的温度指标为&nbsp;hand_temp </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;allColumnNames&nbsp;=&nbsp;<span class="hljs-type">Array</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"timestamp"</span>,&nbsp;<span class="hljs-string">"activityId"</span>,&nbsp;<span class="hljs-string">"hr"</span>)&nbsp;++&nbsp;<span class="hljs-type">Array</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"hand"</span>,&nbsp;<span class="hljs-string">"chest"</span>,&nbsp;<span class="hljs-string">"ankle"</span>).flatMap(sensor&nbsp;=&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">Array</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"temp"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"accel1X"</span>,&nbsp;<span class="hljs-string">"accel1Y"</span>,&nbsp;<span class="hljs-string">"accel1Z"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"accel2X"</span>,&nbsp;<span class="hljs-string">"accel2Y"</span>,&nbsp;<span class="hljs-string">"accel2Z"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"gyroX"</span>,&nbsp;<span class="hljs-string">"gyroY"</span>,&nbsp;<span class="hljs-string">"gyroZ"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"magnetX"</span>,&nbsp;<span class="hljs-string">"magnetY"</span>,&nbsp;<span class="hljs-string">"magnetZ"</span>, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"orientX"</span>,&nbsp;<span class="hljs-string">"orientY"</span>,&nbsp;<span class="hljs-string">"orientZ"</span>,&nbsp;<span class="hljs-string">"orientD"</span>).map(name&nbsp;=&gt;&nbsp;<span class="hljs-string">s"<span class="hljs-subst">${sensor}</span>_<span class="hljs-subst">${name}</span>"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;数据集中不需要的列、时间戳和方位数据，分别表示手、胸部、踝关节上传感器的第一个方位指标 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;ignoredColumns&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-number">0</span>,&nbsp;<span class="hljs-number">16</span>,&nbsp;<span class="hljs-number">17</span>,&nbsp;<span class="hljs-number">18</span>,&nbsp;<span class="hljs-number">19</span>,&nbsp;<span class="hljs-number">33</span>,&nbsp;<span class="hljs-number">34</span>,&nbsp;<span class="hljs-number">35</span>,&nbsp;<span class="hljs-number">36</span>,&nbsp;<span class="hljs-number">50</span>,&nbsp;<span class="hljs-number">51</span>,&nbsp;<span class="hljs-number">52</span>,&nbsp;<span class="hljs-number">53</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;inputColNames&nbsp;=&nbsp;ignoredColumns.map(l&nbsp;=&gt;&nbsp;allColumnNames(l)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;columnNames&nbsp;=&nbsp;allColumnNames. 
&nbsp;&nbsp;&nbsp;&nbsp;filter&nbsp;{&nbsp;!inputColNames.contains(_)&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;滤掉不需要的列，并填充缺失值 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;typeTransformer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">FillMissingValueTranformer</span>().setInputCols(inputColNames) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;构造标签列 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;labelIndexer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StringIndexer</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"activityId"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"indexedLabel"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.fit(df) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;构造特征列 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;vectorAssembler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">VectorAssembler</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCols(columnNames) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"featureVector"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;配置分类器 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;rfClassifier&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">RandomForestClassifier</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setLabelCol(<span class="hljs-string">"indexedLabel"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setFeaturesCol(<span class="hljs-string">"featureVector"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setFeatureSubsetStrategy(<span class="hljs-string">"auto"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setNumTrees(<span class="hljs-number">350</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setMaxBins(<span class="hljs-number">30</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setMaxDepth(<span class="hljs-number">30</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setImpurity(<span class="hljs-string">"entropy"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setCacheNodeIds(<span class="hljs-literal">true</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;labelConverter&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">IndexToString</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"prediction"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"predictedLabel"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setLabels(labelIndexer.labels) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;<span class="hljs-type">Array</span>(trainingData,&nbsp;testData)&nbsp;=&nbsp;df.randomSplit(<span class="hljs-type">Array</span>(<span class="hljs-number">0.8</span>,&nbsp;<span class="hljs-number">0.2</span>)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;构建整个Pipeline </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pipeline&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>().setStages( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">Array</span>(typeTransformer, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labelIndexer,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vectorAssembler,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rfClassifier,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labelConverter)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;pipeline.fit(trainingData) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;predictionResultDF&nbsp;=&nbsp;model.transform(testData) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;展示结果 </span>
&nbsp;&nbsp;&nbsp;&nbsp;predictionResultDF.select( 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"hr"</span>,&nbsp;<span class="hljs-string">"hand_temp"</span>,&nbsp;<span class="hljs-string">"hand_accel1X"</span>,&nbsp;<span class="hljs-string">"hand_accel1Y"</span>,&nbsp;<span class="hljs-string">"hand_accel1Z"</span>,&nbsp;<span class="hljs-string">"hand_accel2X"</span>,&nbsp;<span class="hljs-string">"hand_accel2Y"</span>,&nbsp;<span class="hljs-string">"hand_accel2Z"</span>,&nbsp;<span class="hljs-string">"hand_gyroX"</span>,&nbsp;<span class="hljs-string">"hand_gyroY"</span>,&nbsp;<span class="hljs-string">"hand_gyroZ"</span>,&nbsp;<span class="hljs-string">"hand_magnetX"</span>,&nbsp;<span class="hljs-string">"hand_magnetY"</span>,&nbsp;<span class="hljs-string">"hand_magnetZ"</span>,&nbsp;<span class="hljs-string">"chest_temp"</span>,&nbsp;<span class="hljs-string">"chest_accel1X"</span>,&nbsp;<span class="hljs-string">"chest_accel1Y"</span>,&nbsp;<span class="hljs-string">"chest_accel1Z"</span>,&nbsp;<span class="hljs-string">"chest_accel2X"</span>,&nbsp;<span class="hljs-string">"chest_accel2Y"</span>,&nbsp;<span class="hljs-string">"chest_accel2Z"</span>,&nbsp;<span class="hljs-string">"chest_gyroX"</span>,&nbsp;<span class="hljs-string">"chest_gyroY"</span>,&nbsp;<span class="hljs-string">"chest_gyroZ"</span>,<span class="hljs-string">"chest_magnetX"</span>,&nbsp;<span class="hljs-string">"chest_magnetY"</span>,&nbsp;<span class="hljs-string">"chest_magnetZ"</span>,&nbsp;<span class="hljs-string">"ankle_temp"</span>,&nbsp;<span class="hljs-string">"ankle_accel1X"</span>,&nbsp;<span class="hljs-string">"ankle_accel1Y"</span>,&nbsp;<span class="hljs-string">"ankle_accel1Z"</span>,&nbsp;<span class="hljs-string">"ankle_accel2X"</span>,<span class="hljs-string">"ankle_accel2Y"</span>,&nbsp;<span class="hljs-string">"ankle_accel2Z"</span>,&nbsp;<span class="hljs-string">"ankle_gyroX"</span>,&nbsp;<span class="hljs-string">"ankle_gyroY"</span>,&nbsp;<span class="hljs-string">"ankle_gyroZ"</span>,&nbsp;<span class="hljs-string">"ankle_magnetX"</span>,&nbsp;<span class="hljs-string">"ankle_magnetY"</span>,&nbsp;<span class="hljs-string">"ankle_magnetZ"</span>,&nbsp;<span class="hljs-string">"indexedLabel"</span>,&nbsp;<span class="hljs-string">"predictedLabel"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.show(<span class="hljs-number">20</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;evaluator&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">MulticlassClassificationEvaluator</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setLabelCol(<span class="hljs-string">"indexedLabel"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setPredictionCol(<span class="hljs-string">"prediction"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setMetricName(<span class="hljs-string">"accuracy"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;predictionAccuracy&nbsp;=&nbsp;evaluator.evaluate(predictionResultDF) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;模型性能 </span>
&nbsp;&nbsp;&nbsp;&nbsp;println(<span class="hljs-string">"Testing&nbsp;Error&nbsp;=&nbsp;"</span>&nbsp;+&nbsp;(<span class="hljs-number">1.0</span>&nbsp;-&nbsp;predictionAccuracy)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;randomForestModel&nbsp;=&nbsp;model.stages(<span class="hljs-number">2</span>).asInstanceOf[<span class="hljs-type">RandomForestClassificationModel</span>] 

&nbsp;&nbsp;&nbsp;&nbsp;println(<span class="hljs-string">"Trained&nbsp;Random&nbsp;Forest&nbsp;Model&nbsp;is:\n"</span>&nbsp;+&nbsp;randomForestModel.toDebugString) 
&nbsp;&nbsp;} 

}
</code></pre>
<p data-nodeid="93181">在处理流程中，用自定义的&nbsp;Transformer&nbsp;过滤掉了不需要的数据，并通过填充缺失值的方式对数据进行预处理。自定义的 Transformer 代码如下：</p>
<pre class="lang-scala" data-nodeid="93182"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.<span class="hljs-type">Transformer</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.param.{<span class="hljs-type">Param</span>,&nbsp;<span class="hljs-type">ParamMap</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.util.{<span class="hljs-type">DefaultParamsWritable</span>,&nbsp;<span class="hljs-type">Identifiable</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.types.{<span class="hljs-type">BooleanType</span>,&nbsp;<span class="hljs-type">NumericType</span>,&nbsp;<span class="hljs-type">StructType</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.{<span class="hljs-type">DataFrame</span>,&nbsp;<span class="hljs-type">Dataset</span>,&nbsp;<span class="hljs-type">Row</span>} 

<span class="hljs-comment">//&nbsp;继承基类Transformer </span>
<span class="hljs-class"><span class="hljs-keyword">class</span><span class="hljs-title">&nbsp;FillMissingValueTranformer&nbsp;extends&nbsp;Transformer</span> </span>
{ 
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;uid:&nbsp;<span class="hljs-type">String</span>&nbsp;=&nbsp;<span class="hljs-type">Identifiable</span>.randomUID(<span class="hljs-string">"MissingValueTransformer"</span>) 


&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">val</span>&nbsp;inputCols&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Param</span>[<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]](<span class="hljs-keyword">this</span>,&nbsp;<span class="hljs-string">"inputCol"</span>,&nbsp;<span class="hljs-string">"The&nbsp;input&nbsp;column"</span>) 

&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">override</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;transformSchema</span></span>(schema:&nbsp;<span class="hljs-type">StructType</span>):&nbsp;<span class="hljs-type">StructType</span>&nbsp;=&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;检查输入和输出是否符合要求,&nbsp;比如数据类型 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;返回处理之后的schema </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;inputColNames&nbsp;=&nbsp;$(inputCols) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;incorrectColumns&nbsp;=&nbsp;inputColNames.flatMap&nbsp;{&nbsp;name&nbsp;=&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema(name).dataType&nbsp;<span class="hljs-keyword">match</span>&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;_:&nbsp;<span class="hljs-type">NumericType</span>&nbsp;|&nbsp;<span class="hljs-type">BooleanType</span>&nbsp;=&gt;&nbsp;<span class="hljs-type">None</span> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;other&nbsp;=&gt;&nbsp;<span class="hljs-type">Some</span>(<span class="hljs-string">s"Data&nbsp;type&nbsp;<span class="hljs-subst">$other</span>&nbsp;of&nbsp;column&nbsp;<span class="hljs-subst">$name</span>&nbsp;is&nbsp;not&nbsp;supported."</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(incorrectColumns.nonEmpty)&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throw</span>&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">IllegalArgumentException</span>(incorrectColumns.mkString(<span class="hljs-string">"\n"</span>)) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">StructType</span>(schema.fields) 
&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;setInputCols</span></span>(value:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]):&nbsp;<span class="hljs-keyword">this</span>.<span class="hljs-keyword">type</span>&nbsp;=&nbsp;set(inputCols,&nbsp;value) 

&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">override</span>&nbsp;<span class="hljs-keyword">def</span>&nbsp;transform(dataset:&nbsp;Dataset[_]):&nbsp;DataFrame&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;inputColNames&nbsp;=&nbsp;$(inputCols) 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;rawdata=dataset 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(i&lt;-inputColNames)&nbsp;{rawdata=rawdata.drop(i)} 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;allColumnNames&nbsp;=&nbsp;dataset.columns 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;过滤掉不需要的列名 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;columnNames&nbsp;=&nbsp;allColumnNames.filter&nbsp;{&nbsp;!inputColNames.contains(_)&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;心率的空值填充为<span class="hljs-number">60</span>，其他属性的空值填充为<span class="hljs-number">0</span> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;imputedValues:<span class="hljs-type">Map</span>[<span class="hljs-type">String</span>,<span class="hljs-type">Double</span>]=<span class="hljs-type">Map</span>() 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(colname&lt;-columnNames){ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>(colname==<span class="hljs-string">"hr"</span>){imputedValues&nbsp;+=&nbsp;(colname-&gt;<span class="hljs-number">60.0</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>&nbsp;{imputedValues&nbsp;+=&nbsp;(colname-&gt;<span class="hljs-number">0.0</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;processdata=rawdata.na.drop(<span class="hljs-number">26</span>,columnNames).na.fill(imputedValues) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;processdata.toDF() 
&nbsp;&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">override</span>&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;copy</span></span>(extra:&nbsp;<span class="hljs-type">ParamMap</span>):&nbsp;<span class="hljs-type">Transformer</span>&nbsp;=&nbsp;defaultCopy(extra) 

}
</code></pre>
<h3 data-nodeid="93183">小结</h3>
<p data-nodeid="93184">本课时，我们学习了决策树算法与随机森林算法，随机森林算法也曾经是各种数据科学竞赛中的明星算法，属于集成学习的一种。此外，可以发现随机森林算法对于分布式计算来说是很好的方法，这也是 MLlib 将其实现的原因。在本课时中，我们还实现了一个自定义 Transformer，也是对上一节课的复习。</p>
<p data-nodeid="93185">最后给大家留一个思考题： 在配置分类器时，我们一共设置了多少个参数，每个参数有什么用？</p>

---

### 精选评论


