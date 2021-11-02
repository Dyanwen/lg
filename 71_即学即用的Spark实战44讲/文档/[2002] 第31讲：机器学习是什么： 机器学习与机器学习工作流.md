<p data-nodeid="6493">在开始本课时的学习之前，我们先来讲解上个课时的思考题：成环的路径会使消息一直传递下去，所以需要在发送消息时对消息最初出发的顶点和当前顶点进行校验。</p>



<p data-nodeid="5603">下面我们进入本课时的学习，整个模块 6 主要学习 Spark 的机器学习套件 MLlib。MLlib 在功能上与 Scikit-Learn 等机器学习库非常类似，但计算引擎采用的是 Spark，<strong data-nodeid="5691">即所有计算过程均实现了分布式</strong>，这也是它和其他机器学习库最大的不同。</p>
<p data-nodeid="5604">但在学习 MLlib 的时候，你大可不必关注其分布式细节，这是 MLlib 组件与其他组件很不一样的地方，这里不用考虑 GraphX、Structured Streaming 中的关键抽象、分布式计算框架，而只需关注那些机器学习任务本身的东西，如参数、模型、工作流、测试、算法调优等。本课时的主要内容有：</p>
<ul data-nodeid="5605">
<li data-nodeid="5606">
<p data-nodeid="5607">机器学习</p>
</li>
<li data-nodeid="5608">
<p data-nodeid="5609">典型的机器学习工作流</p>
</li>
<li data-nodeid="5610">
<p data-nodeid="5611">机器学习任务的学习类型</p>
</li>
</ul>
<h3 data-nodeid="5612">机器学习</h3>
<p data-nodeid="5613">在本课时中，我们将试着从计算机科学、 统计学和数据分析的角度来定义机器学习 。机器学习是计算机科学的一个分支，为计算机提供了无须明确编程的学习能力（Arthur Samuel，1959）。这个研究领域是从人工智能的模式识别和计算学习理论的研究中演化而来的。</p>
<p data-nodeid="5614">更具体地说，机器学习探讨了启发式学习和基于数据进行预测的算法研究和构建。这种算法通过样本输入构建模型，通过制订数据驱动的预测来代替严格的静态程序代码。</p>
<p data-nodeid="5615">来看看卡耐基梅隆大学的 Tom M. Mitchell 教授对机器学习的定义，他从计算机科学的角度解释了机器学习的真正意义：</p>
<p data-nodeid="5616">对于某类任务 T 和性能度量 P， <strong data-nodeid="5705">如果一个计算机程序在 T 上以 P 衡量的性能随着经验 E 而自我完善，那么就称这个计算机程序从经验 E 中学习</strong> 。</p>
<p data-nodeid="5617">基于该定义，我们得出计算机程序或机器能够：</p>
<ul data-nodeid="5618">
<li data-nodeid="5619">
<p data-nodeid="5620">从历史数据中学习；</p>
</li>
<li data-nodeid="5621">
<p data-nodeid="5622">通过经验而获得提升；</p>
</li>
<li data-nodeid="5623">
<p data-nodeid="5624">交互式地增强可用于预测问题结果的模型；</p>
</li>
<li data-nodeid="5625">
<p data-nodeid="5626">典型的机器学习任务是概念学习、预测建模、聚类以及寻找有用的模式。最终目标是提高学习的自动化程度，从而不再需要人为地干预，或尽可能地降低人为干预的水平。</p>
</li>
</ul>
<h3 data-nodeid="5627">典型的机器学习工作流</h3>
<p data-nodeid="7179"><strong data-nodeid="7185">一个典型的机器学习应用程序涉及从输入、处理到输出这几个步骤</strong> ，从而形成一个科学的工作流程，如下图所示：</p>
<p data-nodeid="7180" class=""><img src="https://s0.lgstatic.com/i/image/M00/39/86/Ciqc1F8f3hyAGNzsAAFyUK2GB9c136.png" alt="5.png" data-nodeid="7188"></p>


<p data-nodeid="5630">具体步骤如下：</p>
<ol data-nodeid="5631">
<li data-nodeid="5632">
<p data-nodeid="5633">加载样本数据。</p>
</li>
<li data-nodeid="5634">
<p data-nodeid="5635">将数据解析为算法所需的格式。</p>
</li>
<li data-nodeid="5636">
<p data-nodeid="5637">预处理数据并处理缺失值。</p>
</li>
<li data-nodeid="5638">
<p data-nodeid="5639">将数据分成两组：一组用于构建模型（训练数据集），另一组用于测试模型（测试数据集）。</p>
</li>
<li data-nodeid="5640">
<p data-nodeid="5641">运行算法来构建或训练你的 ML 模型。</p>
</li>
<li data-nodeid="5642">
<p data-nodeid="5643">用训练数据进行预测并观察结果。</p>
</li>
<li data-nodeid="5644">
<p data-nodeid="5645">使用测试数据测试和评估模型，或者使用第 3 个数据集（称为验证数据集）运用交叉验证技术验证模型。</p>
</li>
<li data-nodeid="5646">
<p data-nodeid="5647">调整模型以获得更好的性能和准确性。</p>
</li>
<li data-nodeid="5648">
<p data-nodeid="5649">调整模型扩展性，以便将来能够处理大量的数据集。</p>
</li>
<li data-nodeid="5650">
<p data-nodeid="5651">部署模型。</p>
</li>
</ol>
<p data-nodeid="5652">在步骤 4 中，实验数据集是随机分割的，通常分为一个训练数据集和一个被称为采样的测试数据集。训练数据集用于训练模型，而测试数据集用于最终评估最佳模型的性能。更好的做法是尽可能多地使用训练数据集以提高泛化性能。另一方面，建议只使用一次测试数据集，以便在计算预测误差和相关度量时避免过度拟合的问题。</p>
<h3 data-nodeid="5653">机器学习任务的学习类型</h3>
<p data-nodeid="7873">根据学习系统学习反馈的本质，机器学习任务通常被分为以下 3 类，即监督学习、无监督学习以及增强学习，如下图所示：</p>
<p data-nodeid="7874" class=""><img src="https://s0.lgstatic.com/i/image/M00/39/86/Ciqc1F8f3i6Aec4xAAEgqHp1dio379.png" alt="1.png" data-nodeid="7878"></p>


<h4 data-nodeid="8225" class="">1. 监督学习</h4>

<p data-nodeid="5659">监督学习的目标是：学习将输入映射到与现实世界相一致的输出的一般规则，你可以理解为基于过去的数据进行预测 。例如，垃圾邮件过滤数据集通常包含垃圾邮件以及非垃圾邮件，因此，能够知道训练集中的数据是垃圾邮件还是正常邮件。我们有机会利用这些信息来训练模型，以便对新来的邮件进行分类。</p>
<p data-nodeid="8903">算法找到所需的模式后，可以使用这些模式对未标记的测试数据进行预测。这是最常见的机器学习任务类型，MLlib&nbsp;也不例外，其中大部分算法都是监督学习，如朴素贝叶斯、逻辑回归、随机森林等，监督学习的数据处理流程大致如下图所示。</p>
<p data-nodeid="8904" class=""><img src="https://s0.lgstatic.com/i/image/M00/39/91/CgqCHl8f3kGAer8MAAEdGQuqmfA650.png" alt="2.png" data-nodeid="8908"></p>


<p data-nodeid="5662">从图中可以看出，经过数据预处理后，数据会被分为两部分，一部分为测试集，另一部分为训练集。通过学习算法，可以由训练集得到我们所需的模型，模型会用测试集进行验证，工程师会根据验证的情况对模型进行调优，实现一个数据驱动的优化过程。</p>
<h4 data-nodeid="9597" class="">2. 无监督学习</h4>


<p data-nodeid="10267">在无监督学习中，数据没有相关的标签，也就是说无法区分训练集与测试集。因此，我们需要在算法上加上标签，如下图所示。因此，标签必须从数据集中推断出来，这意味着无监督学习算法的目标是通过描述结构，以某种结构化的方式对数据进行预处理。</p>
<p data-nodeid="10268" class=""><img src="https://s0.lgstatic.com/i/image/M00/39/86/Ciqc1F8f3mKAGgNvAADCMxdG_20446.png" alt="3.png" data-nodeid="10272"></p>


<p data-nodeid="5668">为了克服无监督学习中的这个障碍，通常使用聚类技术，基于某些相似性度量来对未标记样本进行分组。因此，无监督学习任务会涉及挖掘隐藏的模式、特征学习等。聚类是智能地对数据集中的元素进行分类的过程。总体思路是，同一个类中的两个元素比属于不同类中的元素彼此更为“接近”。 “接近”的定义可以有很多种，在后面的课时会详细讨论。</p>
<p data-nodeid="5669">无监督的例子包括聚类、频繁模式挖掘以及降维等。MLlib 也提供了 <em data-nodeid="5755">k</em> 均值聚类、潜在狄利克雷分布（Latent Dirichlet Allocation）、主成分分析（Principal Component Analysis）、奇异值分解（Singular value decomposition）等聚类与降维算法。</p>
<h4 data-nodeid="10953" class="">3. 增强学习</h4>


<p data-nodeid="5673">和我们从过去的经验中学习一样，多年来积极的赞美和负面的批评都有助于塑造出今天的我们。通过与朋友、家人，甚至陌生人互动，我们可以了解什么让人开心，什么让人难过。当你执行某个操作时，你有时会立即得到奖励。例如，在附近找到购物中心时可能会产生即时的满足感，但也有些时候，奖励不会马上兑现，比如长途跋涉去寻找某个地方。这些都与增强学习密切相关。</p>
<p data-nodeid="11615">因此， 增强学习是一种模型从一系列行为中学习的技术 。数据集或样本的复杂性对于需要 学习目标函数的增强学习非常重要 。此外，为了达到最终目标，每条数据都需要做到一点，即在保证与外部环境相互作用的同时，应确保奖励函数的最大化，如下图所示：</p>
<p data-nodeid="11616" class=""><img src="https://s0.lgstatic.com/i/image/M00/39/86/Ciqc1F8f3nKAJYBCAAErmWvG_r4694.png" alt="4.png" data-nodeid="11620"></p>


<p data-nodeid="5676">从图中也可以看出， <strong data-nodeid="5767">增强学习与监督学习最大的不同在于其训练集中包含一个尝试的过程</strong> ，会试图从环境中获得评价或者反馈。如围棋这种博弈类游戏，会有两个代理互相用已有的模型制订策略，并根据最后的结果修正自己模型的过程。谷歌公司的 AlphaGo 就是深度学习与增强学习相结合的一个很好的例子。</p>
<p data-nodeid="5677">总而言之，增强学习在 <strong data-nodeid="5773">行动—评价</strong> 的环境中获得知识，改进行动方案以适应环境，并在物联网环境、路线问题、股市交易、机器人等场景重得到了广泛应用。</p>
<h3 data-nodeid="5678">总结</h3>
<p data-nodeid="5679">由于机器学习涉及很多概念，本课时的主要目的是理清概念，为后面的学习打好基础。在后面学习机器学习算法的课时中，每一个课时后都会有一个基于真实数据的实践。</p>
<p data-nodeid="5680">最后给大家留一个思考题：</p>
<p data-nodeid="6137">机器学习和深度学习的区别是什么？</p>

---

### 精选评论


