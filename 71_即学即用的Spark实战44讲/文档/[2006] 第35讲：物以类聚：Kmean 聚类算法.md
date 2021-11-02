<p data-nodeid="89582">在开始之前，先来看看上个课时的思考题。在配置分类器时，我们需要设置的参数主要有：</p>
<ul data-nodeid="89583">
<li data-nodeid="89584">
<p data-nodeid="89585">树的个数；</p>
</li>
<li data-nodeid="89586">
<p data-nodeid="89587">树的最大深度；</p>
</li>
<li data-nodeid="89588">
<p data-nodeid="89589">特征子集选取策略；</p>
</li>
<li data-nodeid="89590">
<p data-nodeid="89591">纯度。</p>
</li>
</ul>
<p data-nodeid="89592">在特征子集的选取策略中可以配置信息增益、信息增益率以及基尼系数。</p>
<p data-nodeid="89593">然后我们开始今天的课程。上一课时中介绍了随机森林和决策树分类器，它们都属于监督学习，本节将介绍聚类算法，它属于机器学习的另一种应用——<strong data-nodeid="89656">无监督学习</strong>。</p>
<h3 data-nodeid="89594">物以类聚</h3>
<p data-nodeid="89595">通俗地说，人们习惯将相似的东西归为一类，这就是“物以类聚”。先来看几个例子，用苹果举例，如下图所示。</p>
<p data-nodeid="89596"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/40/52/CgqCHl8yU0CALbDUAABrSRl7RCg892.png" data-nodeid="89661"></p>
<p data-nodeid="89597">一共有 6 个苹果，如果将这些苹果分类，我们可以很自然地将其分为两类，因为前 4 个的大小明显和后面不同。再来看看另外 6 个苹果，如下图所示。</p>
<p data-nodeid="89598"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/40/47/Ciqc1F8yU0eAUStOAAChV1NLmds129.png" data-nodeid="89665"></p>
<p data-nodeid="89599">对这 6 个苹果进行分类，同样很容易分为两类，前 4 个苹果的颜色和后面的明显不同。在上面的两次分类过程中，已经完成了两次聚类，聚类的依据第一次是大小，第二次是颜色。接下来看看最后 6 个苹果，如下图所示。</p>
<p data-nodeid="89600"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/40/47/Ciqc1F8yU0yAHHBqAAC5gQcgG5E448.png" data-nodeid="89669"></p>
<p data-nodeid="89601">现在还能一口气将这 6 个苹果进行分类吗？这 6 个苹果有大有小，有红有绿，确实令人困惑，如果再加上第 2 个和第 5 个苹果很甜、其余苹果很酸等特征，问题就更棘手了。人对于单一维度的数据很敏感，维度一多就有些力不从心，而计算机恰恰擅长处理这些数据，聚类算法就是“物以类聚”这一朴素思想的数学体现。</p>
<p data-nodeid="89602"><strong data-nodeid="89675">聚类</strong>是把相似的对象通过静态分类的方法分成不同的组别或者更多的子集，使得同一个子集中的成员对象都有相似的一些属性，常见的包括在坐标系中更加短的空间距离。一般把聚类归纳为一种非监督式学习，它与分类算法最大的不同在于训练集没有标签。</p>
<h3 data-nodeid="89603"><em data-nodeid="89680">K</em> 均值聚类算法</h3>
<p data-nodeid="89604">到目前为止，我们对聚类的理解可以用下图来表示。聚类的目标就是将相似的物体进行分组，并将其标注，图中的相似性体现在点与点之间的距离。</p>
<p data-nodeid="89605"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/40/47/Ciqc1F8yU1iAa_T7AADWSLFVU3A993.png" data-nodeid="89684"></p>
<p data-nodeid="89606"><em data-nodeid="89766">K</em> 均值算法是一种被广泛使用的直接聚类算法，位列数据挖掘十大算法之二，可见其影响力。 <em data-nodeid="89767">K</em> 均值算法是一种<strong data-nodeid="89768">迭代型聚类算法</strong>，它将一个给定的数据集分为用户指定的 <em data-nodeid="89769">K</em> 个聚簇，速度较快，易于修改。从上面这句话可以得出 <em data-nodeid="89770">K</em> 均值算法的输入对象为数据集 <em data-nodeid="89771">D</em> 和一个非常关键的 <em data-nodeid="89772">K</em> 值， <em data-nodeid="89773">K</em> 值也就是用户认为数据集 <em data-nodeid="89774">D</em> 应该被分为几类，数据集 <em data-nodeid="89775">D</em> 可以被认为是 <em data-nodeid="89776">d</em> 维向量空间中的一些点（ <em data-nodeid="89777">D</em> ={ <i>x<sub>i</sub></i> | <em data-nodeid="89778">i</em> = 1,…, <em data-nodeid="89779">N</em> }，其中 <em data-nodeid="89780">x<sub>i</sub>∈Rd</em> 表示数据集 <em data-nodeid="89781">D</em> 中第 <em data-nodeid="89782">i</em> 个对象）。</p>
<p data-nodeid="89607">在 <em data-nodeid="89821">K</em> 均值算法中，每个聚簇都用一个点来代表，这些聚簇用集合 <em data-nodeid="89822">C</em> ={ <i>c<sub>j</sub></i> | <em data-nodeid="89823">j</em> = 1,…, <em data-nodeid="89824">k</em> } 来表示，这 <em data-nodeid="89825">K</em> 个代表聚簇有时也被称为聚簇均值或聚簇中心。聚类算法通常用相似度的概念对点集进行分组，具体到 <em data-nodeid="89826">K</em> 均值算法，默认的相似度标准为欧氏距离。 <em data-nodeid="89827">K</em> 均值的实质是要最小化一个如下的非负代价函数：</p>
<p data-nodeid="89608"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/40/53/CgqCHl8yU2qAB5lTAABeO6TQn-Y438.png" data-nodeid="89830"></p>
<p data-nodeid="89609">换言之， <em data-nodeid="89854">K</em> 均值的最小化目标是每个点 <i>x<sub>i</sub></i> 和离它最近的聚簇中心 <i>c<sub>j</sub></i> 之间的欧氏距离的平方和，这也是 <em data-nodeid="89855">K</em> 均值的目标函数。</p>
<p data-nodeid="89610"><strong data-nodeid="89864">下面是</strong> K <strong data-nodeid="89865">均值算法的伪代码</strong>：</p>
<p data-nodeid="89611">输入：数据集 D，聚簇数 k</p>
<p data-nodeid="89612">输出：聚簇代表集合 C，聚簇成员向量 m</p>
<p data-nodeid="89613">/<em data-nodeid="89873">初始化聚簇代表 C</em>/</p>
<p data-nodeid="89614">从数据集 D 中随机挑选 k 个数据点3</p>
<p data-nodeid="89615">使用这 k 个数据点构成初始聚簇代表集合 C</p>
<p data-nodeid="89616">repeat</p>
<p data-nodeid="89617">/<em data-nodeid="89882">再分数据</em>/</p>
<p data-nodeid="89618">将 D 中的每个数据点重新分配至与之最近的聚簇均值</p>
<p data-nodeid="89619">更新 m（m<sub>i</sub> 表示D中第 i 个点的聚簇标识）</p>
<p data-nodeid="89620">/<em data-nodeid="89894">重定均值</em>/</p>
<p data-nodeid="89621">更新 C（c<sub>j</sub> 表示第 j 个聚簇均值）</p>
<p data-nodeid="89622">until 目标函数<br>
<img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image/M00/40/48/Ciqc1F8yU5SAc4YdAABABs5NrzY650.png" data-nodeid="89904"><br>
收敛</p>
<p data-nodeid="89623">从上面的伪代码可以看出，算法主要包括两个交替执行的步骤，即再分数据和重定均值，并且是通过随机选取 <em data-nodeid="89912">K</em> 个点来启动算法。</p>
<p data-nodeid="89624"><strong data-nodeid="89917">再分数据</strong>指的是：将每个数据点分配到当前与之最近的那个聚簇中心，同时打破了上次迭代确定的归属关系。这一步会对全部数据进行一个新的划分。</p>
<p data-nodeid="89625"><strong data-nodeid="89926">重定均值</strong>指的是：重新确定每一个聚簇中心，即计算所有分配给该聚簇的数据点的中心（如算术平均值），这也是 <em data-nodeid="89927">K</em> 均值的名称由来。</p>
<p data-nodeid="89626">当满足收敛条件时，算法停止。</p>
<h3 data-nodeid="89627">鸢尾花数据集进行聚类</h3>
<p data-nodeid="89628">本节将介绍用 <em data-nodeid="89939">K</em> 均值算法对鸢尾花数据集（也就是预处理课时中用到的 iris 数据集）进行聚类的过程，<strong data-nodeid="89940">在聚类之前，需要先对特征用 PCA 进行降维，用 Z 分数进行归一化</strong>。由于鸢尾花数据本身带有标签列（Species，一共有 3 种类型，即 setosa 、 versicolor 和 virginica ），因此最后可以用该列来评估聚类效果，代码如下：</p>
<pre class="lang-scala" data-nodeid="89629"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.{<span class="hljs-type">PCA</span>,&nbsp;<span class="hljs-type">VectorAssembler</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.{<span class="hljs-type">Dataset</span>,&nbsp;<span class="hljs-type">Row</span>,&nbsp;<span class="hljs-type">SparkSession</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">StandardScaler</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.types.{<span class="hljs-type">DoubleType</span>,&nbsp;<span class="hljs-type">StringType</span>,&nbsp;<span class="hljs-type">StructField</span>,&nbsp;<span class="hljs-type">StructType</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.<span class="hljs-type">Pipeline</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">MinMaxScaler</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.clustering.<span class="hljs-type">KMeans</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.evaluation.<span class="hljs-type">ClusteringEvaluator</span> 

<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;IRISKmeans&nbsp;</span></span>{ 

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span> 
&nbsp;&nbsp;&nbsp;&nbsp;.builder() 
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"IRISKmeans"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;iris数据集数据结构 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;fields&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-string">"id"</span>,<span class="hljs-string">"SepalLength"</span>,<span class="hljs-string">"SepalWidth"</span>,<span class="hljs-string">"PetalLength"</span>,<span class="hljs-string">"PetalWidth"</span>,<span class="hljs-string">"Species"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;fieldsType&nbsp;=&nbsp;fields.map( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r&nbsp;=&gt;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(r&nbsp;==&nbsp;<span class="hljs-string">"id"</span>||r&nbsp;==&nbsp;<span class="hljs-string">"Species"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<span class="hljs-type">StructField</span>(r,&nbsp;<span class="hljs-type">StringType</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<span class="hljs-type">StructField</span>(r,&nbsp;<span class="hljs-type">DoubleType</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;schema&nbsp;=&nbsp;<span class="hljs-type">StructType</span>(fieldsType) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;featureCols&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-string">"SepalLength"</span>,<span class="hljs-string">"SepalWidth"</span>,<span class="hljs-string">"PetalLength"</span>,<span class="hljs-string">"PetalWidth"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;data&nbsp;=&nbsp;spark.read.schema(schema) 
&nbsp;&nbsp;&nbsp;&nbsp;.option(<span class="hljs-string">"header"</span>,&nbsp;<span class="hljs-literal">false</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.csv(<span class="hljs-string">"data/iris/"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;vectorAssembler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">VectorAssembler</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCols(featureCols) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"features"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pca&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">PCA</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"features"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"pcaFeatures"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;主成分个数，也就是降维后的维数 </span>
&nbsp;&nbsp;&nbsp;&nbsp;.setK(<span class="hljs-number">2</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;scaler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StandardScaler</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"features"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"scaledFeatures"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setWithStd(<span class="hljs-literal">true</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setWithMean(<span class="hljs-literal">false</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置Kmeans参数 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;kmeans&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">KMeans</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setK(<span class="hljs-number">3</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setSeed(<span class="hljs-type">System</span>.currentTimeMillis()) 
&nbsp;&nbsp;&nbsp;&nbsp;.setMaxIter(<span class="hljs-number">10</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pipeline&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setStages(<span class="hljs-type">Array</span>(vectorAssembler,pca,scaler)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;pipeline.fit(data) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;predictions&nbsp;=&nbsp;model.transform(data) 

&nbsp;&nbsp;&nbsp;&nbsp;predictions.show() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;evaluator&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">ClusteringEvaluator</span>() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;silhouette&nbsp;=&nbsp;evaluator.evaluate(predictions) 

&nbsp;&nbsp;&nbsp;&nbsp;println(<span class="hljs-string">s"Silhouette&nbsp;with&nbsp;squared&nbsp;euclidean&nbsp;distance&nbsp;=&nbsp;<span class="hljs-subst">$silhouette</span>"</span>) 


&nbsp;&nbsp;} 

}
</code></pre>
<p data-nodeid="89630">下面是部分的聚类结果：</p>
<p data-nodeid="89631"><img alt="Drawing 6.png" src="https://s0.lgstatic.com/i/image/M00/40/53/CgqCHl8yU6WAUgR_AAJJxj-Boic557.png" data-nodeid="89944"><br>
……</p>
<p data-nodeid="89632"><img alt="Drawing 7.png" src="https://s0.lgstatic.com/i/image/M00/40/48/Ciqc1F8yU6yAKNibAAIV7Wmjm8U735.png" data-nodeid="89949"><br>
……</p>
<p data-nodeid="89633"><img alt="Drawing 8.png" src="https://s0.lgstatic.com/i/image/M00/40/53/CgqCHl8yU7OAcHxQAAIqHMXe320159.png" data-nodeid="89954"></p>
<p data-nodeid="89634">可以看到，聚类结果为 setosa 类型的样本全部被归到类 1，无一例外；versicolor 类型的样本大部分被归到类 2，极少部分被归到类 0；virginica 类型的样本大部分被归到类 0，少部分被归到类 2，结果基本正确。</p>
<h3 data-nodeid="89635">评估聚类结果</h3>
<p data-nodeid="89636">聚类是一种典型的无监督学习，与监督学习很容易评估模型性能不同，无监督学习通常没有一个标准。那么如何才能判断聚类的结果是优质的，这同样是一个值得讨论的问题。我们先来看一份数据，假设一份数据在数据空间中均匀分布，如下图所示：</p>
<p data-nodeid="89637"><img alt="Drawing 9.png" src="https://s0.lgstatic.com/i/image/M00/40/48/Ciqc1F8yU7qAbdSGAAAxoq-vQ68447.png" data-nodeid="89960"></p>
<p data-nodeid="89638">那么可以想象这份数据的聚类结果质量一定不高，数据的均匀分布导致这些聚簇没有任何实际意义。我们还可以通过霍普金斯统计量来检测变量的空间随机性，这里就不展开讲解了。</p>
<p data-nodeid="89639">一般来说，我们可以通过簇间距离和簇内距离来评估聚类质量，簇间距离指的是两个簇中心之间的距离，而簇内距离是指簇内部成员间的距离。 我们可以这样判断一次优质的聚类结果，如下图所示，有较大的簇间距离和较小的簇内距离，也就是说，优质的聚类是簇中的点尽量紧密，而簇和簇之间尽量远离。</p>
<p data-nodeid="89640"><img alt="Drawing 10.png" src="https://s0.lgstatic.com/i/image/M00/40/53/CgqCHl8yU8OAAiEJAAAyGf6HRaI127.png" data-nodeid="89965"></p>
<p data-nodeid="89641"><img alt="Drawing 11.png" src="https://s0.lgstatic.com/i/image/M00/40/53/CgqCHl8yU8mAfAFoAAAphYBFt9I725.png" data-nodeid="89968"></p>
<h3 data-nodeid="89642">小结</h3>
<p data-nodeid="89643">本课时学习了另一类有代表性的无监督学习算法：聚类算法。在编写 Pipeline 时，我们也用到了前面学习的 Transformer 来进行降维和归一化处理，有了前面的基础，相信同学们对预处理会有更深的理解。</p>
<p data-nodeid="89644">最后给大家留个<strong data-nodeid="89976">思考题</strong>：如果不进行归一化处理，那么会对聚类结果有什么影响呢？</p>

---

### 精选评论


