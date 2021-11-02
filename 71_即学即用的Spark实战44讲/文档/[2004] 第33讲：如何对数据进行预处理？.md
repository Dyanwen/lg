<p data-nodeid="2329">在开始今天的课程前，我们先来看看上一讲的思考题，如何理解训练好的模型也是一个 Transformer 呢？这说明 Transformer 只是描述了一种映射关系，而拟合数据，也就是训练模型的过程，就是为了得到这种映射关系。</p>
<p data-nodeid="2330">今天的内容是 如何对数据进行预处理。 在机器学习实践中，数据科学家拿到的数据通常是不尽人意的，例如出现存在大量的缺失值、特征的值是不同的量纲、有一些无关的特征、特征的值需要再次处理等情况，这样的数据无法直接训练，因此我们需要对这些数据进行预处理。预处理在机器学习中是非常重要的步骤，如果没有按照正确的方法对数据进行预处理，往往会得到错误的训练结果。下面我先介绍几种常见的预处理方法。</p>
<h3 data-nodeid="2331">数据标准化</h3>
<p data-nodeid="2332">通常，我们直接获得的数据包含了量纲，也就是单位，例如身高 180 cm、体重 75 kg。对于某些算法来说，如果特征的单位不统一，就无法直接进行计算，因此在很多情况下的预处理过程中，数据标准化是必不可少的过程。数据标准化的方法一般有 Z 分数法、最大最小法等。</p>
<h4 data-nodeid="2333">Z 分数法</h4>
<p data-nodeid="2334">这种方法根据原始数据（特征）的均值（Mean）和标准差（Standard Deviation）进行数据标准化，从而将原始数据变换为 Z 分数，转化函数如下：</p>
<p data-nodeid="2335"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHlyAczDfAAAGj85eEFw493.png" data-nodeid="2523"></p>
<p data-nodeid="2336">其中 <em data-nodeid="2533">μ</em> 为所有样本数据的均值， <em data-nodeid="2534">σ</em> 为所有样本数据的标准差。Spark MLlib 内置了 Z 分数标准化转换器 StandardScaler，下面的代码演示了通过 StandardScaler 对 Libsvm 数据集的特征进行标准化的过程：</p>
<pre class="lang-scala" data-nodeid="2337"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">StandardScaler</span> 

<span class="hljs-keyword">val</span>&nbsp;dataFrame&nbsp;=&nbsp;spark.read.format(<span class="hljs-string">"libsvm"</span>).load(<span class="hljs-string">"data/mllib/sample_libsvm_data.txt"</span>) 

<span class="hljs-keyword">val</span>&nbsp;scaler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StandardScaler</span>() 
.setInputCol(<span class="hljs-string">"features"</span>) 
.setOutputCol(<span class="hljs-string">"scaledFeatures"</span>) 
.setWithStd(<span class="hljs-literal">true</span>) 
.setWithMean(<span class="hljs-literal">false</span>) 

<span class="hljs-comment">//&nbsp;计算汇总统计量，生成ScalerModel </span>
<span class="hljs-keyword">val</span>&nbsp;scalerModel&nbsp;=&nbsp;scaler.fit(dataFrame) 

<span class="hljs-comment">//&nbsp;对特征进行标准化 </span>
<span class="hljs-keyword">val</span>&nbsp;scaledData&nbsp;=&nbsp;scalerModel.transform(dataFrame) 
scaledData.show()
</code></pre>
<h4 data-nodeid="2338">最大最小法</h4>
<p data-nodeid="2339">这种方法也称为离差标准化，是对原始数据的线性变换，使结果值映射到 [0 - 1] 之间。转换函数如下：</p>
<p data-nodeid="2340"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/3D/24/CgqCHl8pHnCAUHvxAAAisWhfTiI536.png" data-nodeid="2543"></p>
<p data-nodeid="2341">其中 max 为样本数据的最大值，min 为样本数据的最小值。这种方法的缺陷是当有新数据加入时，可能导致 max 和 min 出现变化，需要重新定义，但这种情况在训练过程中很少见。对于方差特别小的特征，这种方法可以增强其稳定性。Spark MLlib 内置了最大最小转换器 MinMaxScaler。下面的代码演示了通过 MinMaxScaler 对测试数据集的特征进行标准化的过程：</p>
<pre class="lang-scala" data-nodeid="2342"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">MinMaxScaler</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.linalg.<span class="hljs-type">Vectors</span> 

<span class="hljs-keyword">val</span>&nbsp;dataFrame&nbsp;=&nbsp;spark.createDataFrame(<span class="hljs-type">Seq</span>( 
&nbsp;&nbsp;(<span class="hljs-number">0</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">1.0</span>,&nbsp;<span class="hljs-number">0.1</span>,&nbsp;<span class="hljs-number">-1.0</span>)), 
&nbsp;&nbsp;(<span class="hljs-number">1</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">2.0</span>,&nbsp;<span class="hljs-number">1.1</span>,&nbsp;<span class="hljs-number">1.0</span>)), 
&nbsp;&nbsp;(<span class="hljs-number">2</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">3.0</span>,&nbsp;<span class="hljs-number">10.1</span>,&nbsp;<span class="hljs-number">3.0</span>)) 
)).toDF(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"features"</span>) 

<span class="hljs-keyword">val</span>&nbsp;scaler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">MinMaxScaler</span>() 
.setInputCol(<span class="hljs-string">"features"</span>) 
.setOutputCol(<span class="hljs-string">"scaledFeatures"</span>) 

<span class="hljs-comment">//&nbsp;计算汇总统计量，生成MinMaxScalerModel </span>
<span class="hljs-keyword">val</span>&nbsp;scalerModel&nbsp;=&nbsp;scaler.fit(dataFrame) 

<span class="hljs-comment">//&nbsp;rescale&nbsp;each&nbsp;feature&nbsp;to&nbsp;range&nbsp;[min,&nbsp;max]. </span>
<span class="hljs-keyword">val</span>&nbsp;scaledData&nbsp;=&nbsp;scalerModel.transform(dataFrame) 
println(<span class="hljs-string">s"Features&nbsp;scaled&nbsp;to&nbsp;range:&nbsp;[<span class="hljs-subst">${scaler.getMin}</span>,&nbsp;<span class="hljs-subst">${scaler.getMax}</span>]"</span>) 
scaledData.select(<span class="hljs-string">"features"</span>,&nbsp;<span class="hljs-string">"scaledFeatures"</span>).show()
</code></pre>
<h4 data-nodeid="2343"><em data-nodeid="2549">p</em> 范数法</h4>
<p data-nodeid="2344"><em data-nodeid="2566">p</em> 范数法指的是计算样本的 <em data-nodeid="2567">p</em> 范数，用该样本除以该样本的 <em data-nodeid="2568">p</em> 范数，得到的值就是标准化的结果。 <em data-nodeid="2569">p</em> 范数的计算公式如下：</p>
<p data-nodeid="2345"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHoKAVk6gAAAMcm1E_XI378.png" data-nodeid="2572"></p>
<p data-nodeid="2346">当 <em data-nodeid="2594">p</em> =1 时， <em data-nodeid="2595">p</em> 范数也叫 L1 范数，此时 L1 等于样本的所有特征值的绝对值相加。当 <em data-nodeid="2596">p</em> =2 时， <em data-nodeid="2597">p</em> 范数也叫 L2 范数，此时 L2 等于样本 <em data-nodeid="2598">x</em> 距离向量空间的原点的欧氏距离：</p>
<p data-nodeid="2347"><img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHoyAH6qCAAAKPd-gKDk980.png" data-nodeid="2601"></p>
<p data-nodeid="2348">归一化的结果为：</p>
<p data-nodeid="2349"><img alt="Drawing 6.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHpSAOctmAAAIfKPjFb4327.png" data-nodeid="2605"></p>
<p data-nodeid="2350">Spark MLlib 内置了 <em data-nodeid="2611">p</em> 范数标准化转化器 Normalizer，下面的代码演示了通过 Normalizer 对测试数据集的特征进行标准化的过程：</p>
<pre class="lang-scala" data-nodeid="2351"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">Normalizer</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.linalg.<span class="hljs-type">Vectors</span> 

<span class="hljs-keyword">val</span>&nbsp;dataFrame&nbsp;=&nbsp;spark.createDataFrame(<span class="hljs-type">Seq</span>( 
(<span class="hljs-number">0</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">1.0</span>,&nbsp;<span class="hljs-number">0.5</span>,&nbsp;<span class="hljs-number">-1.0</span>)), 
(<span class="hljs-number">1</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">2.0</span>,&nbsp;<span class="hljs-number">1.0</span>,&nbsp;<span class="hljs-number">1.0</span>)), 
(<span class="hljs-number">2</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">4.0</span>,&nbsp;<span class="hljs-number">10.0</span>,&nbsp;<span class="hljs-number">2.0</span>)) 
)).toDF(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"features"</span>) 

<span class="hljs-comment">//&nbsp;设置p&nbsp;=&nbsp;1 </span>
<span class="hljs-keyword">val</span>&nbsp;normalizer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Normalizer</span>() 
.setInputCol(<span class="hljs-string">"features"</span>) 
.setOutputCol(<span class="hljs-string">"normFeatures"</span>) 
.setP(<span class="hljs-number">1.0</span>) 

<span class="hljs-keyword">val</span>&nbsp;l1NormData&nbsp;=&nbsp;normalizer.transform(dataFrame) 
l1NormData.show() 

<span class="hljs-comment">//&nbsp;设置p&nbsp;=&nbsp;-∞ </span>
<span class="hljs-keyword">val</span>&nbsp;lInfNormData&nbsp;=&nbsp;normalizer.transform(dataFrame,&nbsp;normalizer.p&nbsp;-&gt;&nbsp;<span class="hljs-type">Double</span>.<span class="hljs-type">PositiveInfinity</span>) 
println(<span class="hljs-string">"Normalized&nbsp;using&nbsp;L^inf&nbsp;norm"</span>) 
lInfNormData.show()
</code></pre>
<h3 data-nodeid="2352">缺失值处理</h3>
<p data-nodeid="2353">缺失值的处理需要根据数据的具体情况来定：如果特征值是连续型，通常用中位数来填充；如果特征值是标签型，通常用众数来补齐；某些情况下，还可以用一个显著区别于已有样本中该特征的值来补齐。Spark 并没有提供预置的缺失值处理的 Transformer，这通常需要自己实现，在后面的课时中，我们会实现一个对缺失值处理的自定义 Transformer。</p>
<h3 data-nodeid="2354">特征抽取</h3>
<p data-nodeid="2355">特征抽取（Feature Extraction）指的是按照某种映射关系生成原有特征的一个特征子集，而前面提到的特征选择（Feature Selection）指的是根据某种规则对原有特征筛选出一个特征子集。特征选择和特征抽取有相同之处，它们都试图减少数据集中的特征数目，但具体方法不同，特征抽取主要是通过特征间的关系来操作，如组合不同特征得到新的特征，这样就改变了原来的特征空间；而特征选择是从原始特征数据集中选择出子集，两者是一种包含关系，没有更改原始的特征空间，如下图所示。</p>
<p data-nodeid="2356"><img alt="Drawing 7.png" src="https://s0.lgstatic.com/i/image/M00/3D/24/CgqCHl8pHqCATQtNAAAsMz3U2wU664.png" data-nodeid="2618"></p>
<p data-nodeid="2357">Spark&nbsp;也提供了很多种特征抽取的方法，常见的如主成分分析、广泛应用于文本的 Word2Vector 等。</p>
<h4 data-nodeid="2358">主成分分析</h4>
<p data-nodeid="2359">如果在样本中无关的特征太多，就会影响模式的发现，我们需要用降维技术从样本中生成用来代表原有特征的一个特征子集。</p>
<p data-nodeid="2360">在了解主成分分析（PCA）之前，需要先了解协方差的概念， <em data-nodeid="2631">X</em> 特征与 <em data-nodeid="2632">Y</em> 特征之间的协方差为：</p>
<p data-nodeid="2361"><img alt="Drawing 8.png" src="https://s0.lgstatic.com/i/image/M00/3D/24/CgqCHl8pHqmAXpqaAAALYIFlgZY287.png" data-nodeid="2635"></p>
<p data-nodeid="2362">如果协方差为正，说明 <em data-nodeid="2673">X</em> 和 <em data-nodeid="2674">Y</em> 是正相关关系；如果协方差为负，则说明是负相关关系；当协方差为 0 时， <em data-nodeid="2675">X</em> 和 <em data-nodeid="2676">Y</em> 相互独立。如果样本集 <em data-nodeid="2677">D</em> 有 <em data-nodeid="2678">n</em> 维特征，那么两两之间的协方差就可以组成一个 <em data-nodeid="2679">n</em> × <em data-nodeid="2680">n</em> 的矩阵，如下是 <em data-nodeid="2681">n</em> =3 的情况：</p>
<p data-nodeid="2363"><img alt="Drawing 9.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHreAQBtCAAATNVzupVs430.png" data-nodeid="2684"></p>
<p data-nodeid="2364">然后需要对这个矩阵进行特征值分解，得到特征值和特征向量，再取出最大的 <em data-nodeid="2730">m</em> （ <em data-nodeid="2731">m</em> &lt; <em data-nodeid="2732">n</em> ） 个特征值对应的特征向量（w<sub>1</sub>、w<sub>2</sub>、…、 w<sub>m</sub>），从而组成特征向量矩阵 <strong data-nodeid="2733">W</strong>，对每个样本 <i>x<sub>i</sub></i> 执行如下操作，得到降维后的样本 <i>z<sub>i</sub></i>，如下：</p>
<p data-nodeid="2365"><img alt="Drawing 10.png" src="https://s0.lgstatic.com/i/image/M00/3D/24/CgqCHl8pHsWAFsvxAAAEt-j1_VQ537.png" data-nodeid="2736"></p>
<p data-nodeid="2366">降维后的数据集为：</p>
<p data-nodeid="2367"><img alt="Drawing 11.png" src="https://s0.lgstatic.com/i/image/M00/3D/24/CgqCHl8pHs6ARdiqAAAHmI3k1_k058.png" data-nodeid="2740"></p>
<p data-nodeid="2368">下面以数据挖掘领域著名的鸢尾花数据集（IRIS： https://pan.baidu.com/s/1CPB8NQEN5crGC3MQ_eVutA 密码： yey1）为例，来展示用 PCA 实现降维操作的过程。</p>
<p data-nodeid="2369">鸢尾花数据集是常用的分类数据集，包含 150 个样本。样本总共分为 3 种类别、各 50 条，每个样本有 4 个维度，分别是花萼长度、花萼宽度、花瓣长度和花瓣宽度。操作过程的代码如下：</p>
<pre class="lang-scala" data-nodeid="16186"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.{<span class="hljs-type">PCA</span>,&nbsp;<span class="hljs-type">VectorAssembler</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.{<span class="hljs-type">Dataset</span>,&nbsp;<span class="hljs-type">Row</span>,&nbsp;<span class="hljs-type">SparkSession</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">StandardScaler</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.types.{<span class="hljs-type">DoubleType</span>,&nbsp;<span class="hljs-type">StringType</span>,&nbsp;<span class="hljs-type">StructField</span>,&nbsp;<span class="hljs-type">StructType</span>} 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.<span class="hljs-type">Pipeline</span> 

<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;IRISPCA&nbsp;</span></span>{ 

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span> 
&nbsp;&nbsp;&nbsp;&nbsp;.builder() 
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"IRISPCA"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;数据结构为花萼长度、花萼宽度、花瓣长度、花瓣宽度 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;fields&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-string">"id"</span>,<span class="hljs-string">"Species"</span>,<span class="hljs-string">"SepalLength"</span>,<span class="hljs-string">"SepalWidth"</span>,<span class="hljs-string">"PetalLength"</span>,<span class="hljs-string">"PetalWidth"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;fieldsType&nbsp;=&nbsp;fields.map( 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r&nbsp;=&gt;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(r&nbsp;==&nbsp;<span class="hljs-string">"id"</span>||r&nbsp;==&nbsp;<span class="hljs-string">"Species"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<span class="hljs-type">StructField</span>(r,&nbsp;<span class="hljs-type">StringType</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<span class="hljs-type">StructField</span>(r,&nbsp;<span class="hljs-type">DoubleType</span>)} 
&nbsp;&nbsp;&nbsp;&nbsp;) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;schema&nbsp;=&nbsp;<span class="hljs-type">StructType</span>(fieldsType) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;featureCols&nbsp;=&nbsp;<span class="hljs-type">Array</span>(<span class="hljs-string">"SepalLength"</span>,<span class="hljs-string">"SepalWidth"</span>,<span class="hljs-string">"PetalLength"</span>,<span class="hljs-string">"PetalWidth"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;data=spark.read.schema(schema).csv(<span class="hljs-string">"data/iris"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;vectorAssembler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">VectorAssembler</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCols(featureCols) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"features"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;vectorData=vectorAssembler.transform(data) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;特征标准化 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;standardScaler&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">StandardScaler</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"features"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"scaledFeatures"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setWithMean(<span class="hljs-literal">true</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setWithStd(<span class="hljs-literal">false</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.fit(vectorData) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pca&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">PCA</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setInputCol(<span class="hljs-string">"scaledFeatures"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setOutputCol(<span class="hljs-string">"pcaFeatures"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;主成分个数，也就是降维后的维数 </span>
&nbsp;&nbsp;&nbsp;&nbsp;.setK(<span class="hljs-number">2</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pipeline&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setStages(<span class="hljs-type">Array</span>(vectorAssembler,standardScaler,pca))

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;pipeline.fit(data) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;对特征进行PCA降维 </span>
&nbsp;&nbsp;&nbsp;&nbsp;model.transform(data).select(<span class="hljs-string">"Species"</span>,&nbsp;<span class="hljs-string">"pcaFeatures"</span>).show(<span class="hljs-number">100</span>,&nbsp;<span class="hljs-literal">false</span>) 

&nbsp;&nbsp;} 

}
</code></pre>
<p data-nodeid="16187">通过操作，可以发现降维后的数据只有两个维度，如下，这样就实现了我们的降维过程：</p>
<p data-nodeid="16188"><img alt="Drawing 12.png" src="https://s0.lgstatic.com/i/image/M00/3D/25/CgqCHl8pHtmAA0eIAADQjIqfWM0637.png" data-nodeid="16334"></p>
<h4 data-nodeid="16189">Word2Vector</h4>
<p data-nodeid="16190">在自然语言处理领域，训练集通常为纯文本，这样的数据是无法直接训练的，前面提到的 TF-IDF 就是一种生成词向量的方式。但是 TF-IDF 的缺点在于单纯以“词频”衡量一个词的重要性，不够全面，忽略了上下文信息，例如“阿里巴巴成立达摩院”与“人工智能应用有望加速落地”字面无任何相似之处，但它们之间有很强的关联，这用 TF-IDF 却无法体现。</p>
<p data-nodeid="16191">而 Word2Vec 可以解决这个问题。 Word2Vec 最先出现在谷歌公司在 2013 年发表的论文“Efficient Estimation of Word Representation in Vector Space”中，作者是 Mikolov。Word2Vec 的基本思想是采用一个 3 层的神经网络将每个词映射成 <em data-nodeid="16342">n</em> 维的实数向量，为接下来的聚类或者比较相似性等操作做准备。这个 3 层神经网络实际是在对语言模型进行建模，在建模的同时也获得了单词在向量空间上的表示，即词向量，也就是说这个词向量是建模过程的中间产物，而这个中间产物才是 Word2Vec 的真正目标。</p>
<p data-nodeid="16192">Word2Vec 采用了：CBOW 与 Skip-gram，前者可以根据上下文预测下一个词，后者可以根据当前词预测上下文。以 CBOW 为例，如下图所示。</p>
<p data-nodeid="16193"><img alt="Drawing 13.png" src="https://s0.lgstatic.com/i/image/M00/3D/19/Ciqc1F8pHuSAFsjJAAB5_1HQCOo265.png" data-nodeid="16346"></p>
<p data-nodeid="16194">我们选择一个固定的窗口作为语境（上下文）：t - 2&nbsp;— t + 2，输入层是 4 个 <em data-nodeid="16356">n</em> 维的词向量（初始为随机值）；隐藏层做的操作是累计求和操作，隐藏层包含 <em data-nodeid="16357">n</em> 个结点；输出层是一棵巨大的二叉树，构建这棵二叉树的算法就是霍夫曼树，它的叶子结点代表了语料中的 M 个词语，语料中有多少个词，就有多少个叶子结点。</p>
<p data-nodeid="16195">假设左子树为 1，右子树为 0，这样每个叶子结点都有唯一的编码。最后输出的时候，CBOW 采用了层次 Softmax 算法，隐藏层的每个结点都与树的每个结点相连，即霍夫曼树上的每个结点都会有 M 条边，每条边都有权重，我们需要达到的是在输入的上下文一定的情况下，预测词 W 的概率最大。以 010110 为例，由霍夫曼树的定义可知树有 5 层，我们希望在根结点，词向量与根结点相连，也就是在第一层，希望经过回归运算后第一位等于 0 的概率尽量等于 1 。依次类推，在第二层，希望第二位的值等于 1 的概率尽可能等于&nbsp;1。这样一直下去，路径上所有的权重乘积就是预测词在当前上下文的概率 <em data-nodeid="16375">P</em> ( <em data-nodeid="16376">wt</em> )，而在语料中我们可以得到当前上下文的残差 1- <em data-nodeid="16377">P</em> ( <em data-nodeid="16378">wt</em> )，这样的话，就可以使用梯度下降来学习参数了。</p>
<p data-nodeid="16196">由于不需要标注，Word2Vec 本质上是一种无监督学习，对自然语言处理有兴趣的读者不妨仔细阅读谷歌公司的那篇论文。</p>
<p data-nodeid="16197">Spark MLlib 内置了 Word2Vec 转换器 Word2Vec，在下面的例子中对文档使用了 Word2Vec 转换器：</p>
<pre class="lang-scala" data-nodeid="16198"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">Word2Vec</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.linalg.<span class="hljs-type">Vector</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.<span class="hljs-type">Row</span> 

<span class="hljs-comment">//&nbsp;每一行输入数据都是来源于某句话或是某个文档 </span>
<span class="hljs-keyword">val</span>&nbsp;documentDF&nbsp;=&nbsp;spark.createDataFrame(<span class="hljs-type">Seq</span>( 
<span class="hljs-string">"Hi&nbsp;I&nbsp;heard&nbsp;about&nbsp;Spark"</span>.split(<span class="hljs-string">"&nbsp;"</span>), 
<span class="hljs-string">"I&nbsp;wish&nbsp;Java&nbsp;could&nbsp;use&nbsp;case&nbsp;classes"</span>.split(<span class="hljs-string">"&nbsp;"</span>), 
<span class="hljs-string">"Logistic&nbsp;regression&nbsp;models&nbsp;are&nbsp;neat"</span>.split(<span class="hljs-string">"&nbsp;"</span>) 
).map(<span class="hljs-type">Tuple1</span>.apply)).toDF(<span class="hljs-string">"text"</span>) 

<span class="hljs-comment">//&nbsp;设置word2Vec参数 </span>
<span class="hljs-keyword">val</span>&nbsp;word2Vec&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Word2Vec</span>() 
.setInputCol(<span class="hljs-string">"text"</span>) 
.setOutputCol(<span class="hljs-string">"result"</span>) 
.setVectorSize(<span class="hljs-number">3</span>) 
.setMinCount(<span class="hljs-number">0</span>) 

<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;word2Vec.fit(documentDF) 

<span class="hljs-keyword">val</span>&nbsp;result&nbsp;=&nbsp;model.transform(documentDF) 
result.collect().foreach&nbsp;{&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-type">Row</span>(text:&nbsp;<span class="hljs-type">Seq</span>[_],&nbsp;features:&nbsp;<span class="hljs-type">Vector</span>)&nbsp;=&gt; 
println(<span class="hljs-string">s"Text:&nbsp;[<span class="hljs-subst">${text.mkString(",&nbsp;")}</span>]&nbsp;=&gt;&nbsp;\nVector:&nbsp;<span class="hljs-subst">$features</span>\n"</span>)&nbsp;}
</code></pre>
<h3 data-nodeid="16199">特征选择</h3>
<p data-nodeid="16200">特征选择的目标通常是提高预测准确性、提升训练性能，以便能够更好地解释模型。特征选择的一种很重要的思想就是对每一维的特征打分，这样就能选出最重要的特征了。基于这种思想的方法有：卡方检验、信息增益以及相关系数。相关系数主要量化的是任意两个特征是否存在线性相关，信息增益会在下一节介绍，本小节主要介绍卡方检验。</p>
<p data-nodeid="16201">卡方检验是以卡方分布为基础的一种常用假设检验方法，它的原假设是观察频数与期望频数没有差别。该检验的基本思想是：首先假设原假设成立，基于此前提计算出 x<sup>2</sup> 值，它表示观察值与理论值之间的偏离程度。根据卡方分布及自由度可以确定在原假设成立的条件下，获得当前统计量及更极端情况的概率 P 。如果 P 值很小，说明观察值与理论值偏离程度太大，应当拒绝无效假设，表示比较资料之间有显著差异；否则就不能拒绝无效假设，尚不能认为样本所代表的实际情况和理论假设有差别。</p>
<p data-nodeid="16202">假设样本的某一个特征，它的取值为 A 和 B 两个组，而样本的类别有 0 和 1 两类。对样本进行统计，可以得到如下表所示的统计表。</p>
<table data-nodeid="16204">
<thead data-nodeid="16205">
<tr data-nodeid="16206">
<th data-nodeid="16208" data-org-content="组别">组别</th>
<th data-nodeid="16209" data-org-content="0">0</th>
<th data-nodeid="16210" data-org-content="1">1</th>
<th data-nodeid="16211" data-org-content="合计">合计</th>
</tr>
</thead>
<tbody data-nodeid="16216">
<tr data-nodeid="16217">
<td data-nodeid="16218" data-org-content="*A*"><em data-nodeid="16396">A</em></td>
<td data-nodeid="16219" data-org-content="19">19</td>
<td data-nodeid="16220" data-org-content="24">24</td>
<td data-nodeid="16221" data-org-content="43">43</td>
</tr>
<tr data-nodeid="16222">
<td data-nodeid="16223" data-org-content="*B*"><em data-nodeid="16403">B</em></td>
<td data-nodeid="16224" data-org-content="34">34</td>
<td data-nodeid="16225" data-org-content="10">10</td>
<td data-nodeid="16226" data-org-content="44">44</td>
</tr>
<tr data-nodeid="16227">
<td data-nodeid="16228" data-org-content="合计">合计</td>
<td data-nodeid="16229" data-org-content="53">53</td>
<td data-nodeid="16230" data-org-content="34">34</td>
<td data-nodeid="16231" data-org-content="87">87</td>
</tr>
</tbody>
</table>
<p data-nodeid="16232">从上表中，我们可看出 <em data-nodeid="16420">A</em> 和 <em data-nodeid="16421">B</em> 组对分类结果有很大的影响，但这不排除抽样的影响。首先假设该特征有结果是独立无关的，随机取一个样本，属于类 0 的概率为 (19 + 34)/(19 + 34 + 24 + 10)= 60.9%。接下来，我们需要根据上表得到一个理论值表，如下表所示。</p>
<table data-nodeid="16234">
<thead data-nodeid="16235">
<tr data-nodeid="16236">
<th data-nodeid="16238" data-org-content="组别">组别</th>
<th data-nodeid="16239" data-org-content="0">0</th>
<th data-nodeid="16240" data-org-content="1">1</th>
<th data-nodeid="16241" data-org-content="合计">合计</th>
</tr>
</thead>
<tbody data-nodeid="16246">
<tr data-nodeid="16247">
<td data-nodeid="16248" data-org-content="*A*"><em data-nodeid="16429">A</em></td>
<td data-nodeid="16249" data-org-content="43 × 0.609 = 26.2">43 × 0.609 = 26.2</td>
<td data-nodeid="16250" data-org-content="43 × 0.391 = 16.8">43 × 0.391 = 16.8</td>
<td data-nodeid="16251" data-org-content="43">43</td>
</tr>
<tr data-nodeid="16252">
<td data-nodeid="16253" data-org-content="*B*"><em data-nodeid="16436">B</em></td>
<td data-nodeid="16254" data-org-content="44 × 0.609 = 26.8">44 × 0.609 = 26.8</td>
<td data-nodeid="16255" data-org-content="44 × 0.391 = 17.2">44 × 0.391 = 17.2</td>
<td data-nodeid="16256" data-org-content="44">44</td>
</tr>
</tbody>
</table>
<p data-nodeid="16257">如果两个变量是独立无关的，那么上表中的理论值与实际值的差别会非常小。</p>
<p data-nodeid="16258">卡方值的计算公式为：</p>
<p data-nodeid="16259"><img alt="Drawing 15.png" src="https://s0.lgstatic.com/i/image/M00/3D/25/CgqCHl8pHyqAAHYvAAAI87-YsrM071.png" data-nodeid="16444"></p>
<p data-nodeid="16260">其中 <em data-nodeid="16454">A</em> 为实际值，也就是第一个表中的 4 个数据， <em data-nodeid="16455">T</em> 为理论值，也就是上表中给出的 4 个数据，计算后得到卡方值为 10.01。得到该值后，需要在给定的置信水平下查得卡方分布临界值，如下表所示。</p>
<table data-nodeid="16262">
<thead data-nodeid="16263">
<tr data-nodeid="16264">
<th data-nodeid="16266"></th>
<th data-nodeid="16267" data-org-content="*P*"><em data-nodeid="16459">P</em></th>
<th data-nodeid="16268"></th>
<th data-nodeid="16269"></th>
<th data-nodeid="16270"></th>
<th data-nodeid="16271"></th>
<th data-nodeid="16272"></th>
<th data-nodeid="16273"></th>
<th data-nodeid="16274"></th>
<th data-nodeid="16275"></th>
<th data-nodeid="16276"></th>
<th data-nodeid="16277"></th>
<th data-nodeid="16278"></th>
<th data-nodeid="16279"></th>
</tr>
</thead>
<tbody data-nodeid="16294">
<tr data-nodeid="16295">
<td data-nodeid="16296" data-org-content="*n* ¢"><em data-nodeid="16464">n</em> ¢</td>
<td data-nodeid="16297" data-org-content="0.995">0.995</td>
<td data-nodeid="16298" data-org-content="0.99">0.99</td>
<td data-nodeid="16299" data-org-content="0.975">0.975</td>
<td data-nodeid="16300" data-org-content="0.95">0.95</td>
<td data-nodeid="16301" data-org-content="0.9">0.9</td>
<td data-nodeid="16302" data-org-content="0.75">0.75</td>
<td data-nodeid="16303" data-org-content="0.5">0.5</td>
<td data-nodeid="16304" data-org-content="0.25">0.25</td>
<td data-nodeid="16305" data-org-content="0.1">0.1</td>
<td data-nodeid="16306" data-org-content="0.05">0.05</td>
<td data-nodeid="16307" data-org-content="0.025">0.025</td>
<td data-nodeid="16308" data-org-content="0.01">0.01</td>
<td data-nodeid="16309" data-org-content="0.005">0.005</td>
</tr>
<tr data-nodeid="16310">
<td data-nodeid="16311" data-org-content="1">1</td>
<td data-nodeid="16312" data-org-content="…">…</td>
<td data-nodeid="16313" data-org-content="…">…</td>
<td data-nodeid="16314" data-org-content="…">…</td>
<td data-nodeid="16315" data-org-content="…">…</td>
<td data-nodeid="16316" data-org-content="0.02">0.02</td>
<td data-nodeid="16317" data-org-content="0.1">0.1</td>
<td data-nodeid="16318" data-org-content="0.45">0.45</td>
<td data-nodeid="16319" data-org-content="1.32">1.32</td>
<td data-nodeid="16320" data-org-content="2.71">2.71</td>
<td data-nodeid="16321" data-org-content="3.94">3.94</td>
<td data-nodeid="16322" data-org-content="5.02">5.02</td>
<td data-nodeid="16323" data-org-content="5.63">5.63</td>
<td data-nodeid="16324" data-org-content="7.88">7.88</td>
</tr>
</tbody>
</table>
<p data-nodeid="16325">显然 10.01 &gt; 7.88，也就是说该特征与分类结果无关的概率小于 0.5%，换言之，我们应该保留这个特征。</p>
<p data-nodeid="16326">Spark MLlib 内置了卡方检验组件 ChiSqSelector，下面的例子中对某个测试数据集采用了卡方检验：</p>
<pre class="lang-scala" data-nodeid="16327"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.feature.<span class="hljs-type">ChiSqSelector</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.linalg.<span class="hljs-type">Vectors</span> 

<span class="hljs-keyword">val</span>&nbsp;data&nbsp;=&nbsp;<span class="hljs-type">Seq</span>( 
&nbsp;&nbsp;(<span class="hljs-number">7</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">0.0</span>,&nbsp;<span class="hljs-number">0.0</span>,&nbsp;<span class="hljs-number">18.0</span>,&nbsp;<span class="hljs-number">1.0</span>),&nbsp;<span class="hljs-number">1.0</span>), 
&nbsp;&nbsp;(<span class="hljs-number">8</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">0.0</span>,&nbsp;<span class="hljs-number">1.0</span>,&nbsp;<span class="hljs-number">12.0</span>,&nbsp;<span class="hljs-number">0.0</span>),&nbsp;<span class="hljs-number">0.0</span>), 
&nbsp;&nbsp;(<span class="hljs-number">9</span>,&nbsp;<span class="hljs-type">Vectors</span>.dense(<span class="hljs-number">1.0</span>,&nbsp;<span class="hljs-number">0.0</span>,&nbsp;<span class="hljs-number">15.0</span>,&nbsp;<span class="hljs-number">0.1</span>),&nbsp;<span class="hljs-number">0.0</span>) 
) 

<span class="hljs-keyword">val</span>&nbsp;df&nbsp;=&nbsp;spark.createDataset(data).toDF(<span class="hljs-string">"id"</span>,&nbsp;<span class="hljs-string">"features"</span>,&nbsp;<span class="hljs-string">"clicked"</span>) 

<span class="hljs-comment">//&nbsp;配置卡方检验参数 </span>
<span class="hljs-keyword">val</span>&nbsp;selector&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">ChiSqSelector</span>() 
.setNumTopFeatures(<span class="hljs-number">3</span>) 
.setFeaturesCol(<span class="hljs-string">"features"</span>) 
.setLabelCol(<span class="hljs-string">"clicked"</span>) 
.setOutputCol(<span class="hljs-string">"selectedFeatures"</span>) 

<span class="hljs-keyword">val</span>&nbsp;result&nbsp;=&nbsp;selector.fit(df).transform(df) 

println(<span class="hljs-string">s"ChiSqSelector&nbsp;output&nbsp;with&nbsp;top&nbsp;<span class="hljs-subst">${selector.getNumTopFeatures}</span>&nbsp;features&nbsp;selected"</span>) 
result.show()
</code></pre>
<h3 data-nodeid="16328">小结</h3>
<p data-nodeid="16329">本课时主要介绍了数据预处理的方法，结合 Pipeline API 可以发现，编程已经完全不是预处理的重点，如何根据手上的数据集和目标选择预处理方法才是最重要的。老实说，MLlib 并没有提供非常多的预处理组件，也就是前面课时学习的 Transformer，好在它提供了自定义Transformer 的接口。</p>
<p data-nodeid="16330">最后给大家留一个思考题：实现一个任意逻辑的自定义 Transformer。</p>

---

### 精选评论


