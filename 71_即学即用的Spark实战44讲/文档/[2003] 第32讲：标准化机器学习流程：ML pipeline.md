<p data-nodeid="413">在开始今天的课程前，我们先来讲解一下上节课的课后思考题。深度学习与机器学习的不同之处在于：</p>
<ul data-nodeid="414">
<li data-nodeid="415">
<p data-nodeid="416">数据量大小。深度学习通常需要更多的样本才能达到更好的效果，所以通常深度学习的训练时间更长。</p>
</li>
<li data-nodeid="417">
<p data-nodeid="418">硬件区别。深度学习算法通常涉及大量浮点运算，样本量也巨大，而 GPU 天然的海量流处理器架构非常适合并行计算，所以一般复杂的深度学习应用通常需要 GPU 的硬件架构。</p>
</li>
<li data-nodeid="419">
<p data-nodeid="420">特征选择。一般机器学习解决问题时，都需要专家指定或者先验知识来确定特征，如信用模型，这些特征在很大程度上影响了模型的准确性。</p>
</li>
<li data-nodeid="421">
<p data-nodeid="422">解决问题的方法。当使用传统机器学习方法解决问题时，经常采取化整为零、分别解决、再合并结果求解的策略。而深度学习是端到端的模型，输入训练数据，再直接输出最终结果，让深度神经网络自己学习如何提取关键特征，比如对一张有着多个目标的照片进行目标检测，需要识别出目标的类别，并指出图中所在位置。典型机器学习方法会将这个问题分为两步：目标检测与目标识别。首先，使用边框检测技术，扫描全图找到所有可能的对象，对这些对象使用目标识别算法，如支持向量机，识别出相关物体。深度学习方法则按照端到端的方式处理这个问题，比如通过卷积神经网络就能够实现目标的定位与识别，也就是将原始图像输入到卷积神经网络模型中，再直接输出图像中目标的位置和类别。</p>
</li>
<li data-nodeid="423">
<p data-nodeid="424">可解释性。同神经网络算法一样，深度学习模型很难进行解释，这也使深度学习算法无法应用于很多要求模型可解释的场景，如金融等。</p>
</li>
</ul>
<p data-nodeid="425">接下来我们开始讲解今天的内容，标准化机器学习流程：ML Pipeline。Spark MLlib 是 Spark 机器学习套件， 它的目的是成为大数据机器学习的最佳实践。为了简化机器学习过程并使其可扩展，Spark ML API 引入了 Pipelines API（管道），这类似于 Python 机器学习库 Scikit-Learn 中的 Pipeline，它采用了一系列 API 定义并标准化了上一课时中我们学习的机器学习工作流，包含了数据收集、预处理、特征抽取、特征选择、模型拟合、模型验证、模型评估等一系列阶段。例如，对文档进行分类时，也许会包含分词、特征抽取、训练分类模型，以及调优等过程。大多数机器学习库不是为分布式计算而设计的，也不提供 Pipeline 的创建与调优，而这就是 Spark ML PipeLines 要做的。</p>
<p data-nodeid="426">Spark ML Pipelines 就是对分布式机器学习过程进行模块化地抽象，这样可以使多个算法合并成一个 Pipeline 或者使工作流变得更加容易，下面是 Pipelines API 的关键概念：</p>
<ul data-nodeid="427">
<li data-nodeid="428">
<p data-nodeid="429">DataFrame：DataFrame 与 Spark SQL 中用到的 DataFrame 一样，是 Spark 的基础数据结构，贯穿了整个 Pipeline。它可以存储文本、特征向量、训练集以及测试集。除了常见的类型，DataFrame 还支持 Spark MLlib 特有的 Vector 类型。</p>
</li>
<li data-nodeid="430">
<p data-nodeid="431">Transformer：Transformer 对应了数据转换的过程，它接收一个 DataFrame，在它的作用下，会生成一个新的 DataFrame。在机器学习中，在涉及特征转换的过程中经常会用到它。Transformer也可以用于使训练完成后的模型将特征数据集（测试集）转换为带有预测结果的数据集的场景。Transformer 必须实现 transform() 方法。</p>
</li>
<li data-nodeid="432">
<p data-nodeid="433">Estimator：从上面&nbsp;Transformer&nbsp;的定义中可以得知， <strong data-nodeid="488">训练完成好的模型也是一个 Transformer</strong> ，所以 Estimator 包含了一个可以让数据集拟合出一个 Transformer 的算法。 Estimator 必须实现 fit() 方法。</p>
</li>
<li data-nodeid="434">
<p data-nodeid="435">Pipeline：一个 Pipeline 可以将多个 Transformer 和 Estimator 组装成一个特定的机器学习工作流。</p>
</li>
<li data-nodeid="436">
<p data-nodeid="437">Parameter：所有的 Estimator 和 Transformer 共用一套通用的 API 来指定参数。</p>
</li>
</ul>
<p data-nodeid="438">文档分类是一个在自然语言处理中非常常见的应用，如垃圾邮件监测、情感分析等。下面，我们将通过一个文档分类的例子来让读者对 Spark 的 Pipeline 有一个感性的理解。简单来说，任何文档分类应用都需要以下 4 步：</p>
<ol data-nodeid="439">
<li data-nodeid="440">
<p data-nodeid="441">将文档分词。</p>
</li>
<li data-nodeid="442">
<p data-nodeid="443">将分词的结果转换为词向量。</p>
</li>
<li data-nodeid="444">
<p data-nodeid="445">学习模型。</p>
</li>
<li data-nodeid="446">
<p data-nodeid="447">预测（是否为垃圾邮件或者正负情感）。</p>
</li>
</ol>
<p data-nodeid="448">比如在垃圾邮件监测中，我们需要通过邮件正文甄别出哪些是垃圾邮件。垃圾邮件的正文一般会是一段文字，如：“代开各种发票，手续费极低，请联系我。”但这样一段文字是无法直接应用于 Estimator 的，需要将其转换为特征向量。一般做法是用一个词典构建一个向量空间，其中每一个维度都是一个词，出现过的为 1 ，未出现的为 0 ，再根据文档中出现的词语的频数，用 TF-IDF 算法为词维度赋予权重。这样的话，每个文档就能被转换为一个等长的特征向量，如下：</p>
<p data-nodeid="449">(0, 0, …, 0.27, 0, 0, …, 0.1, 0) ，接着就可以用它来拟合模型并输出测试结果。</p>
<p data-nodeid="450">我们用一个流程图来表示整个过程，如下图所示： Tokenizer 和 HashingTF 为 Transformer，作用分别是分词和计算权重，训练出的模型也是 Transformer，用来生成测试结果；Estimator 采用的是逻辑回归算法（LR）；DS0-DS3 都是不同阶段输出的数据，这就是一个完整意义上的 Pipeline。</p>
<p data-nodeid="451"><img alt="1.png" src="https://s0.lgstatic.com/i/image/M00/3A/CE/Ciqc1F8igOiAGSXYAAEv2-wuZAs564.png" data-nodeid="501"></p>
<p data-nodeid="452">下面用代码实现整个 Pipeline，如下：</p>
<pre class="lang-scala" data-nodeid="994"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.{<span class="hljs-type">Pipeline</span>,&nbsp;<span class="hljs-type">PipelineModel</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.classification.<span class="hljs-type">LogisticRegression</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.{<span class="hljs-type">HashingTF</span>,&nbsp;<span class="hljs-type">Tokenizer</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.linalg.<span class="hljs-type">Vector</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.<span class="hljs-type">Row</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.<span class="hljs-type">SparkSession</span> 

<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;PipelineExample</span></span>{ 

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>&nbsp;]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span> 
&nbsp;&nbsp;&nbsp;&nbsp;.builder 
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"PipelineExample"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate() 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">import</span>&nbsp;spark.implicits._ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;准备训练数据，其中最后一列，就是该文档的标签，即是否为垃圾邮件 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;training&nbsp;=&nbsp;spark.createDataFrame(<span class="hljs-type">Seq</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">0</span>L,&nbsp;<span class="hljs-string">"a&nbsp;b&nbsp;c&nbsp;d&nbsp;e&nbsp;spark"</span>,&nbsp;<span class="hljs-number">1.0</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">1</span>L,&nbsp;<span class="hljs-string">"b&nbsp;d"</span>,&nbsp;<span class="hljs-number">0.0</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">2</span>L,&nbsp;<span class="hljs-string">"spark&nbsp;f&nbsp;g&nbsp;h"</span>,&nbsp;<span class="hljs-number">1.0</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">3</span>L,&nbsp;<span class="hljs-string">"hadoop&nbsp;mapreduce"</span>,&nbsp;<span class="hljs-number">0.0</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;)).toDF(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"text"</span>,&nbsp;<span class="hljs-string">"label"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;配置整个Pipeline,&nbsp;由3个组件组成：tokenizer（Transformer）、hashingTF（Transformer） </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;和&nbsp;lr（Estimator） </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;tokenizer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Tokenizer</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"text"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"words"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;hashingTF&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">HashingTF</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setNumFeatures(<span class="hljs-number">1000</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(tokenizer.getOutputCol) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"features"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;lr&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">LogisticRegression</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setMaxIter(<span class="hljs-number">10</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setRegParam(<span class="hljs-number">0.001</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pipeline&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setStages(<span class="hljs-type">Array</span>(tokenizer,&nbsp;hashingTF,&nbsp;lr)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;拟合模型，得到结果 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;pipeline.fit(training) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将模型持久化 </span>
&nbsp;&nbsp;&nbsp;&nbsp;model.write.overwrite().save(<span class="hljs-string">"/tmp/spark-logistic-regression-model"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;将Pipeline持久化 </span>
&nbsp;&nbsp;&nbsp;&nbsp;pipeline.write.overwrite().save(<span class="hljs-string">"/tmp/unfit-lr-model"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;加载模型 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;sameModel&nbsp;=&nbsp;<span class="hljs-type">PipelineModel</span>.load(<span class="hljs-string">"/tmp/spark-logistic-regression-model"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;准备无标签的测试集 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;test&nbsp;=&nbsp;spark.createDataFrame(<span class="hljs-type">Seq</span>( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">4</span>L,&nbsp;<span class="hljs-string">"spark&nbsp;i&nbsp;j&nbsp;k"</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">5</span>L,&nbsp;<span class="hljs-string">"l&nbsp;m&nbsp;n"</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">6</span>L,&nbsp;<span class="hljs-string">"spark&nbsp;hadoop&nbsp;spark"</span>), 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;(<span class="hljs-number">7</span>L,&nbsp;<span class="hljs-string">"apache&nbsp;hadoop"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;)).toDF(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"text"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;用模型预测测试集，得到预测结果（标签） </span>
&nbsp;&nbsp;&nbsp;&nbsp;sameModel.transform(test) 
&nbsp;&nbsp;&nbsp;&nbsp;.select(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"text"</span>,&nbsp;<span class="hljs-string">"probability"</span>,&nbsp;<span class="hljs-string">"prediction"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.collect() 
&nbsp;&nbsp;&nbsp;&nbsp;.foreach&nbsp;{&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-type">Row</span>(id:&nbsp;<span class="hljs-type">Long</span>,&nbsp;text:&nbsp;<span class="hljs-type">String</span>,&nbsp;prob:&nbsp;<span class="hljs-type">Vector</span>,&nbsp;prediction:&nbsp;<span class="hljs-type">Double</span>)&nbsp;=&gt; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;println(<span class="hljs-string">s"(<span class="hljs-subst">$id</span>,&nbsp;<span class="hljs-subst">$text</span>)&nbsp;--&gt;&nbsp;prob=<span class="hljs-subst">$prob</span>,&nbsp;prediction=<span class="hljs-subst">$prediction</span>"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;} 
&nbsp;&nbsp;} 
}
</code></pre>
<p data-nodeid="995">这样就用 Spark 完整实现了一个机器学习的流程。从上面的代码中可以看出，这样的结构非常有利于复用 Transformer 与 Estimator 组件。</p>
<p data-nodeid="996">Spark MLlib（ML API）的算法包主要分为以下几个部分：</p>
<ul data-nodeid="997">
<li data-nodeid="998">
<p data-nodeid="999">特征抽取、转换与选择；</p>
</li>
<li data-nodeid="1000">
<p data-nodeid="1001">分类和回归；</p>
</li>
<li data-nodeid="1002">
<p data-nodeid="1003">聚类；</p>
</li>
<li data-nodeid="1004">
<p data-nodeid="1005">协同过滤；</p>
</li>
<li data-nodeid="1006">
<p data-nodeid="1007">频繁项集挖掘。</p>
</li>
</ul>
<p data-nodeid="1008">其中每一类都有若干种算法的实现，用户可以利用 Pipeline 按需进行切换，下面我们将根据这几个类别，分别实现一些真实数据的案例，让读者可以直接上手应用。</p>
<p data-nodeid="1009">此外，在上面的代码中，我们用 Pipeline API 将模型序列化成文件，这样的好处在于可以将模型看成一个黑盒，非常方便模型上线，而不用在上线应用时再去对模型进行硬编码，这类似于 Python 的 Pickle 库的用法。</p>
<h3 data-nodeid="1010">小结</h3>
<p data-nodeid="1011">在 Spark 的早期版本中，并没有 ML pipeline API，这导致代码难以维护、可读性较差、也一定程度上影响了 MLlib 的流行。而 Pipeline API 的引入抽象了机器学习流程，让整个代码变得简洁优美，另外这种抽象也利于与第三方库进行结合，如 Tensorflow、XgBoost 等等。</p>
<p data-nodeid="1012">如何理解本课时内容中的这句话：最后给大家留一个思考题，</p>
<p data-nodeid="1013">训练完成好的模型也是一个 Transformer。</p>

---

### 精选评论

##### **升：
> 这里的实现方式怎么都没有Python？数据分析师很多都不会用java的，理解起来也很有难度

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有一些还是有的哈，另外一些如Graphx是因为本身就没有Python API

