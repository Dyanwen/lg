<p data-nodeid="517">在开始今天的课程前，我们先来看看上个课时的习题：推荐引擎如何冷启动？冷启动是一个比较复杂也比较常见的问题。这里简单介绍下，可以选用的方法有用户登录自选标签，利用用户的注册信息分类并进行相应的推荐等。</p>
<p data-nodeid="518">而当我们 完成训练之后我们还需要对模型进行评估，根据评估的结果，有可能需要对模型超参进行优化，以达到理想的效果。本课时的主要内容有：</p>
<ul data-nodeid="519">
<li data-nodeid="520">
<p data-nodeid="521">模型评估</p>
</li>
<li data-nodeid="522">
<p data-nodeid="523">交叉验证与超参调优</p>
</li>
</ul>
<p data-nodeid="524">如果你是用 MLlib 来进行模型构建，那么本节内容的使用频率会大大高于本模块的其他内容。</p>
<h3 data-nodeid="525">模型评估</h3>
<p data-nodeid="646">评估模型的质量，通常可以用一些评价方法说明，如精确率、召回率、AUC 、ROC 、PRC 等。我们先从混淆矩阵讲起，如果模型是一个二分类模型，当预测的概率值大于某个阈值，我们就将其归为类 1，那么我们可以从测试集的结果中得到如下表所示的矩阵。</p>

<p data-nodeid="527"><img alt="1.png" src="https://s0.lgstatic.com/i/image/M00/43/72/CgqCHl87ncyAUK8bAABpsM2lFeI981.png" data-nodeid="582"></p>
<p data-nodeid="1686">上表中 True Positive 是本身为类 1，预测为类 1 的数目，称为真阳性；False Positive 是本身为类 0，预测为类 1 的数目，称为伪阳性；False Negative 是本身为类 1，预测为类 0 的数目，称为伪阴性；True Negative 是本身为类 0，预测为类 0 的数目，称为真阴性。根据这 4 个值，可以计算出精确率、召回率和 F 值。</p>




<ul data-nodeid="3316">
<li data-nodeid="3317">
<p data-nodeid="3318">精确率 P= TP / (TP + FP)。精确率表示在预测为类 1 的样本中，实际为类 1 的样本比例。有一些模型对精确率要求很高，例如垃圾邮件监测，宁愿放过垃圾邮件也不愿误判一封正常邮件。</p>
</li>
<li data-nodeid="3319">
<p data-nodeid="3320">召回率 R= TP / (TP + FN)。召回率表示在实际为类 1 的样本中，预测为类 1 的样本比例。有一些模型对召回率的要求很高，例如肿瘤监测，宁愿误判也不愿错过。</p>
</li>
<li data-nodeid="3321">
<p data-nodeid="3322">准确率 = (TP + TN) / ALL。准确率表示正确预测的样本比例。</p>
</li>
<li data-nodeid="3323">
<p data-nodeid="3324">F=2/(1/P+1/R)=2PR/(P+R)。F 值是精确率与召回率的几何平均值，是对模型的一种综合评估，一个值比一条曲线来得直观。</p>
</li>
<li data-nodeid="3325">
<p data-nodeid="3326">ROC 与 PRC。ROC 曲线的横轴 FPR（False Positive Rate，伪阳性率）= FP/(FP + TN)；纵轴 TPR（True Positive Rate，真阳性率）= TP/(TP + FN)，曲线上的每个点代表不同的阈值所带来不同分类结果，阈值变化越细，ROC 曲线越平滑。如果存在一个理想的阈值，高于该阈值的都为类&nbsp;1，反之为类 2，那么该模型就能完美区分两个类，此时横轴值为 0，纵轴值为 1，所以该 ROC 曲线经过&nbsp;(0,&nbsp;1)&nbsp;点，曲线下面积为 1。通常我们用 ROC 曲线下面积（Area Under Curve，AUC）来评估模型质量，最大值为 1，越大说明模型效果越好，如下图所示中&nbsp;3&nbsp;条曲线分别代表了&nbsp;3&nbsp;个模型表现。</p>
</li>
</ul>






<p data-nodeid="540"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/43/73/CgqCHl87npmASyfKAAET6Rbivy8093.png" data-nodeid="591"></p>
<p data-nodeid="541">还有一种曲线叫精确召回率曲线（Precision Recall Curve，PRC），横轴是召回率，纵轴是精确率，与 ROC 越左凸效果越好不同，PRC 是越右凸效果越好，同样是曲线下面积越大越好。如下图所示，最上面的一条（AUC = 0.79）比最下面的一条（AUC = 0.39）效果好。PRC 可以在给定精确率或者召回率的情况下，去比较召回率或者精确率。</p>
<p data-nodeid="542"><img alt="Drawing 2.png" src="https://s0.lgstatic.com/i/image/M00/43/68/Ciqc1F87nqCAQwL9AAGtxOauXzc349.png" data-nodeid="595"></p>
<p data-nodeid="3590">Spark MLlib 提供了模型评估的 API，我们在得到了测试结果后，可以使用如下代码对模型进行评估：</p>

<pre class="lang-scala" data-nodeid="544"><code data-language="scala"><span class="hljs-comment">//&nbsp;逻辑回归模型得到的测试结果 </span>
<span class="hljs-keyword">val</span>&nbsp;result&nbsp;=&nbsp;model.transform(test) 
<span class="hljs-keyword">val</span>&nbsp;evaluator&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;org.apache.spark.ml.evaluation.<span class="hljs-type">RegressionEvaluator</span> 
<span class="hljs-comment">//&nbsp;指定模型评估指标 </span>
<span class="hljs-keyword">var</span>&nbsp;evaluatorParamMap&nbsp;=&nbsp;<span class="hljs-type">ParamMap</span>(evaluator.metricName-&gt;&nbsp;<span class="hljs-string">"areaUnderROC"</span>) 
<span class="hljs-keyword">var</span>&nbsp;aucTraining&nbsp;=&nbsp;evaluator.evaluate(result,&nbsp;evaluatorParamMap)
</code></pre>
<h3 data-nodeid="545">交叉验证与超参调优</h3>
<p data-nodeid="546">在机器学习中，我们都会将数据分为两份，一份作为训练集，另一份作为测试集，这就是交叉验证的思想。但在样本量不足的情况下，我们经常采用 <em data-nodeid="603">k</em> 折交叉验证法来充分测试，以期得到更好的效果。</p>
<p data-nodeid="547"><em data-nodeid="612">k</em> 折交叉验证法将全量数据分为 <em data-nodeid="613">k</em> 份，每次选一份作为测试集，剩下的作为训练集，如下图所示是一个10 折交叉验证。</p>
<p data-nodeid="548"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/43/73/CgqCHl87nqqAVvieAADgeC3PtEM025.png" data-nodeid="616"></p>
<p data-nodeid="549"><em data-nodeid="621">k</em> 折交叉验证通常和超参调优一起使用，超参是在开始训练过程之前设置值的参数，而不是通过训练得到的参数。例如，随机森林模型的超参数有：</p>
<ul data-nodeid="550">
<li data-nodeid="551">
<p data-nodeid="552">树的个数；</p>
</li>
<li data-nodeid="553">
<p data-nodeid="554">树的最大深度；</p>
</li>
<li data-nodeid="555">
<p data-nodeid="556">特征子集选取策略；</p>
</li>
<li data-nodeid="557">
<p data-nodeid="558">每个特征分裂时，最大划分数量；</p>
</li>
<li data-nodeid="559">
<p data-nodeid="560">纯度。</p>
</li>
</ul>
<p data-nodeid="561">调整这些参数对模型性能有着巨大影响。通常，没有特别好的方法来指定这些参数，经验通常会很有用。我们只能通过不停地实验各种参数组合来测试模型性能，这样做的前提是，一定要保证组合非常全面。Spark MLlib 强大的运算能力能让我们快速测试大量的组合。</p>
<p data-nodeid="562">还是以随机森林模型为例，下面的代码可以让我们进行参数组合的测试：</p>
<pre class="lang-scala" data-nodeid="563"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;paramGrid&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">ParamGridBuilder</span>() 
.addGrid(rf.numTrees,&nbsp;<span class="hljs-number">3</span>&nbsp;::&nbsp;<span class="hljs-number">5</span>&nbsp;::&nbsp;<span class="hljs-number">10</span>&nbsp;::&nbsp;<span class="hljs-type">Nil</span>) 
.addGrid(rf.featureSubsetStrategy,&nbsp;<span class="hljs-string">"auto"</span>&nbsp;::&nbsp;<span class="hljs-string">"all"</span>&nbsp;::&nbsp;<span class="hljs-type">Nil</span>) 
.addGrid(rf.impurity,&nbsp;<span class="hljs-string">"gini"</span>&nbsp;::&nbsp;<span class="hljs-string">"entropy"</span>&nbsp;::&nbsp;<span class="hljs-type">Nil</span>) 
.addGrid(rf.maxBins,&nbsp;<span class="hljs-number">2</span>&nbsp;::&nbsp;<span class="hljs-number">5</span>&nbsp;::&nbsp;<span class="hljs-type">Nil</span>) 
.addGrid(rf.maxDepth,&nbsp;<span class="hljs-number">3</span>&nbsp;::&nbsp;<span class="hljs-number">5</span>&nbsp;::&nbsp;<span class="hljs-type">Nil</span>) 
.build()
</code></pre>
<p data-nodeid="564">这里实际上我们定义了一个超参空间，空间的维度与超参的个数一样，所以 5 维空间中的每一个点代表了一个参数组合。下面我们创建一个交叉验证器，并定义 <em data-nodeid="634">k</em> = 5：</p>
<pre class="lang-scala" data-nodeid="565"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;crossValidator&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">CrossValidator</span>() 
.setEstimator(<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>().setStages(<span class="hljs-type">Array</span>(labelIndexer,&nbsp;featureIndexer,&nbsp;rf))) 
.setEstimatorParamMaps(paramGrid) 
.setNumFolds(<span class="hljs-number">5</span>) 
.setEvaluator(evaluator)
</code></pre>
<p data-nodeid="566">并通过训练测试所有的参数组合从而得到最好的一组参数组合：</p>
<pre class="lang-scala" data-nodeid="567"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;crossValidatorModel&nbsp;=&nbsp;crossValidator.fit(data) 
<span class="hljs-comment">//&nbsp;得到效果最好的模型 </span>
<span class="hljs-keyword">val</span>&nbsp;bestModel&nbsp;=&nbsp;crossValidatorModel.bestModel 

<span class="hljs-keyword">val</span>&nbsp;bestPipelineModel&nbsp;=crossValidatorModel.bestModel.asInstanceOf[<span class="hljs-type">PipelineModel</span>] 
<span class="hljs-keyword">val</span>&nbsp;stages&nbsp;=&nbsp;bestPipelineModel.stages 

<span class="hljs-comment">//&nbsp;得到最佳的模型超参 </span>
<span class="hljs-keyword">val</span>&nbsp;rfStage&nbsp;=&nbsp;stages(stages.length<span class="hljs-number">-1</span>).asInstanceOf[<span class="hljs-type">RandomForestClassificationModel</span>] 
<span class="hljs-keyword">val</span>&nbsp;numTrees&nbsp;=&nbsp;rfStage.getNumTrees 
<span class="hljs-keyword">val</span>&nbsp;featureSubsetStrategy&nbsp;=&nbsp;rfStage.getFeatureSubsetStrategy 
<span class="hljs-keyword">val</span>&nbsp;impurity&nbsp;=&nbsp;rfStage.getImpurity 
<span class="hljs-keyword">val</span>&nbsp;maxBins&nbsp;=&nbsp;rfStage.getMaxBins 
<span class="hljs-keyword">val</span>&nbsp;maxDepth&nbsp;=&nbsp;rfStage.getMaxDepth
</code></pre>
<p data-nodeid="3850">CrossValidator 继承了 Estimator 并实现了 fit() 方法。在 fit() 方法调用后，整个 Pipeline 会执行多次，包含从数据预处理到训练模型。在这个例子中，超参一共有 5 个，并进行了 5 折实验，需要执行 3 × 2 × 2 × 2 × 2 × 5 = 240 次 Pipeline，因此每多一个超参数，计算量通常都会大大增长，得益于 Spark 的并行计算能力，计算时间可以大大缩短。</p>

<h3 data-nodeid="569">总结</h3>
<p data-nodeid="570">MLlib 不光为数据预处理与训练模型提供了解决方案，还为模型评估与调优提供了接口。如开头所述，不要看本课时的篇幅较短，但实际上使用率会很高。</p>
<p data-nodeid="571">本课时学完，Spark 所有组件就已经学完。其实大家不难发现，从模块三开始，使用场景是越来越窄的，批处理 -&gt; 流处理 -&gt; 图挖掘 -&gt; 机器学习，所以本模块对于数据科学家来说在某些情况下是很有用的，但是对于数据工程师来说，使用场景并没有那么多。</p>
<p data-nodeid="572">最后给大家留一个<strong data-nodeid="645">思考题</strong>：树的最大深度对随机森林模型效果有什么影响呢？</p>

---

### 精选评论


