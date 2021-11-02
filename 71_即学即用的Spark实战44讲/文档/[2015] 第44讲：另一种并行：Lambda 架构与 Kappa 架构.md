<p data-nodeid="193622" class="">在上个课时中，我提出了一个问题：数据仓库如何实时获取更新的数据，并将结果融合到最新的报表中，也就是提供统一的查询方式呢？本课时我会介绍一种新的架构模式 Lambda，它能很好地解决上一课时的问题。</p>
<p data-nodeid="193623">本课时的主要内容有：</p>
<ul data-nodeid="193624">
<li data-nodeid="193625">
<p data-nodeid="193626">Lambda 架构</p>
</li>
<li data-nodeid="193627">
<p data-nodeid="193628">Kappa 架构</p>
</li>
</ul>
<h3 data-nodeid="193629">Lambda 架构</h3>
<p data-nodeid="193630">在前面的项目中，我们对全量数据进行了一次性处理，也就是批处理。但在真实环境下，数据经常实时更新，一般来说会想到运用流处理技术。但事实上，这两种技术都不能完全满足我们的需求。</p>
<p data-nodeid="193631">流处理通常应对的是低吞吐、低延迟的场景，而批处理应对的则是高吞吐、高延迟的场景。对于 Hadoop 架构下的数据仓库来说，虽然能够对全量数据进行批量处理，但处理完的数据并不能真实反映外部系统的情况，会有一定时延。而流处理平台虽然能对当前时间的窗口数据进行很好的处理，但得到的结果只能反映当前视图。</p>
<p data-nodeid="193632">那是否有一种架构能够将这两种数据处理方式的优点相结合，以达到我们的要求呢？答案就是下面介绍的 Lambda 架构。</p>
<p data-nodeid="194142">Lambda 架构是由 Storm 的创始人 Nathan Marz 提出的一种实时大数据系统方法论，它借鉴了很多函数式编程思想。在 Lambda 架构中，首先将整个系统分为 3 层，即批处理层（Batch Layer）、速度层（Speed Layer）与服务层（Serving Layer），如下图所示。其中，服务层分别依赖速度层与批处理层，而批处理层与速度层之间没有依赖。</p>
<p data-nodeid="194143" class=""><img src="https://s0.lgstatic.com/i/image/M00/4D/6B/CgqCHl9aCKOABvhsAABo2DMsAgY779.png" alt="Lark20200910-190544.png" data-nodeid="194147"></p>


<h4 data-nodeid="193635">批处理层</h4>
<p data-nodeid="193636">Lambda 架构的批处理层会对数据进行一次性处理。从数学层面来说，会通过下面的等式来对所有数据进行分析：</p>
<pre class="lang-js" data-nodeid="193637"><code data-language="js">query = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">all data</span>)
</span></code></pre>
<p data-nodeid="193638">但等式若要成立却很困难，很难有一种技术可以直接对全量数据进行处理并马上返回结果，那么<strong data-nodeid="193705">退而求其次的方案是</strong>：基于数据仓库的思路对全量数据进行批处理，然后对批处理的结果视图进行查询，也就是说新的等式应该如下所示。</p>
<pre class="lang-js" data-nodeid="193639"><code data-language="js">batch view = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">all data</span>)
<span class="hljs-title">query</span> = <span class="hljs-title">function</span>(<span class="hljs-params">batch view</span>)
</span></code></pre>
<p data-nodeid="193640">这么说或许有些抽象，但用传统数据仓库与数据集市的星形模型可以很好地解释，如下图所示。</p>
<p data-nodeid="193641"><img src="https://s0.lgstatic.com/i/image/M00/4D/6A/CgqCHl9aB-KAcKsOAAD4Y7Hpl_8590.png" alt="Lark20200910-190009.png" data-nodeid="193709"></p>
<p data-nodeid="193642">星形模型的维表与事实表就是我们所说的批处理视图（在项目中对应我们生成的 ORC 格式的数据立方体），而处理过程可以视作某种函数。在进行查询的时候，无须扫描数据仓库中的所有数据，只需对星形模型所代表的数据方体进行上卷、下钻、切片等操作。比如，如果需要查询某个商品在某个时间段，以及某个地区的销量，只需要在数据集市中用按照时间维度、地区维度以及商品维度进行聚合即可。</p>
<p data-nodeid="193643">但是我们要看到<strong data-nodeid="193716">这种方式的缺陷</strong>：以这种方式进行处理的时候，在这段时间内收集到的数据不会被包含到批处理视图中，所以查询结果的时效性会很差。此外，数据仓库中的数据被称为主数据集，它表示到目前为止所有可用的数据。而到现在为止，批处理的工作就是基于主数据集生成批处理视图，从而忽略了上述时间内收集到的数据。</p>
<h4 data-nodeid="193644">服务层</h4>
<p data-nodeid="193645">既然批处理的工作只是生成批处理视图，那么我们还需要将其加载并提供查询服务，这就是服务层的工作。服务层是一个数据库，它会保存批处理视图，当新一批次的批处理完成后，新的批处理视图会替换旧的批处理视图，于是新的批处理视图已经包含了新数据的处理结果。</p>
<p data-nodeid="193646">服务层的数据库<strong data-nodeid="193724">面对的场景很特殊</strong>，在该场景下，只有读查询而完全没有随机写操作，但包含批量写入（在加载新的批处理视图时的一次性操作）。这样一来，服务层的数据库就避免了随机写所带来的各种问题，如一致性等。一般来说，要想保证整个架构的良好扩展性，每个组件都需要有很好的扩展性，所以服务层数据库一般选择分布式数据库，如 HBase。</p>
<h4 data-nodeid="193647">速度层</h4>
<p data-nodeid="193648">当批处理层计算出批处理视图并加载到服务层后，这段时间产生的新数据是没有体现在查询结果中的。于是就需要速度层来对这部分数据进行流处理。</p>
<p data-nodeid="193649">也就是说，速度层负责处理两次批处理之间产生的新数据，但是流处理的模式与批处理不同，它并不是只做一次计算，而是持续不断地处理。所以速度层做的是增量计算，与服务层的批处理视图没有随机写入不同，速度层的流处理视图包含大量随机写入操作，并混合着部分的读操作。</p>
<p data-nodeid="193650">流处理的工作就是处理新数据并更新至实时视图。在下一次批处理完成后，这个间隔产生的实时视图就可以完全丢弃，因为最新的批处理视图已经包含了这部分的结果，而速度层则又从零开始构建实时视图，如此循环往复。从使用场景上来看，实时视图的数据库比批处理视图的数据库的复杂程度要高很多。</p>
<p data-nodeid="193651">我们也可以用下面的等式来概括速度层：</p>
<pre class="lang-js" data-nodeid="193652"><code data-language="js">realtime view = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">realtime view,new data</span>)
</span></code></pre>
<p data-nodeid="193653">从等式中可以看出，新的实时视图需要老实时视图作为输入，这就是平时常说的增量计算，也是速度层很大一部分读操作的来源。</p>
<h4 data-nodeid="193654">Lambda 架构</h4>
<p data-nodeid="193655">通过前面三小节的内容，我们已经了解了批处理层、服务层与速度层。可以用如下的数学公式来总结整个操作过程：</p>
<pre class="lang-js" data-nodeid="193656"><code data-language="js">batch view = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">all data</span>)
<span class="hljs-title">realtime</span> <span class="hljs-title">view</span> = <span class="hljs-title">function</span>(<span class="hljs-params">realtime view,new data</span>)
<span class="hljs-title">query</span> = <span class="hljs-title">function</span>(<span class="hljs-params">batch view, realtime view</span>)
</span></code></pre>
<p data-nodeid="193657">这也解答了本节开头的一个问题：如何实时地对全量数据进行分析？答案是，需要对批处理视图和实时视图进行查询并合并，才能得到正确的、实时的结果。这也是Lambda 架构的操作流程，如下图所示。</p>
<p data-nodeid="193658"><img src="https://s0.lgstatic.com/i/image/M00/4D/6A/CgqCHl9aB_WAR0OQAAHh5Y4cgKA078.png" alt="Lark20200910-190005.png" data-nodeid="193736"></p>
<p data-nodeid="193659">这个架构有个优点，一旦批处理层重算生成了新的批处理视图，当前实时视图的结果立刻可以丢弃。也就是说，如果当前的实时视图有什么问题的话，只需要丢弃掉当前的实时视图，并在几小时内（甚至更短的时间），整个系统就可以重新回到正常状态。</p>
<p data-nodeid="193660">整个系统的结果的正确性建立在算法能很好地支持增量计算的基础上。如果算法无法支持增量计算，那么速度层只能采取近似处理，而由于批处理层可以实现精确算法，整个系统仍然具有最终准确性。可以说，Lambda 架构在性能和准确性上做出了很好的权衡，每隔一段时间，批处理层会纠正速度层的错误，速度层的不准确只是暂时的，但这个特性可以给你极大的灵活性。</p>
<p data-nodeid="193661">Nathan Marz 创造 Lambda 架构这个术语来描述一种可扩展、容错数据处理的通用模式，在这个模式中，最重要的是速度层与批处理层。有趣的是，虽然这个架构为什么被命名为 Lambda（λ）我们并不清楚，但从λ 的形状也可以得到解释。λ 的形状似乎暗示了批处理层与速度层通过服务层进行合并的过程，如下图所示，这多少与 Lambda 架构本身有一些契合。<strong data-nodeid="193743">从某种层面上来说，Lambda 架构也体现了一种并行计算的思路。</strong></p>
<p data-nodeid="193662"><img src="https://s0.lgstatic.com/i/image/M00/4D/5F/Ciqc1F9aB_yAadPhAAAWu5-QmPM618.png" alt="Drawing 3.png" data-nodeid="193746"></p>
<h4 data-nodeid="193663">Lambda 架构的原则</h4>
<p data-nodeid="193664">Nathan Marz 总结了 Lambda 架构的原则，主要有 3 条。</p>
<ul data-nodeid="193665">
<li data-nodeid="193666">
<p data-nodeid="193667"><strong data-nodeid="193753">容错性</strong>：Lambda 架构的每个组件都应该具有很好的容错性，在任何情况下，都不会存在数据丢失的情况，此外，对于那些数据处理的错误，也要能很好地恢复。</p>
</li>
<li data-nodeid="193668">
<p data-nodeid="193669"><strong data-nodeid="193758">数据不可变</strong>：这一点其实是 Lambda 的灵魂。Lambda 架构之所有能够如上述般架构，最重要的原因是数据不可变，这样的系统具有很小的复杂性，也更易于管理。它只允许查询和插入数据，不允许删除与更改数据。</p>
</li>
<li data-nodeid="193670">
<p data-nodeid="193671"><strong data-nodeid="193763">重算</strong>：重算可以使架构充满灵活性，由于主数据集总是可用的，因此总可以重新定义计算来满足新需求，这意味着主数据集并没有和什么模式绑定在一起。</p>
</li>
</ul>
<h3 data-nodeid="193672">Kappa 架构</h3>
<p data-nodeid="193673">前面已经介绍了 Lambda 架构的理论和实现，它的优点有容错性、可扩展性，低延迟读取等，但它的最大缺点在于需要维护两套业务逻辑相同的代码。而当时还是 LinkedIn 的高级技术主管 Jay&nbsp;Kreps 提出了一种类 Lambda 架构的新架构 Kappa 架构，可以改善 Lambda 架构的不足。</p>
<p data-nodeid="193674">Kappa 架构与 Lambda 架构很相似，但它的批处理层被删除，只保留速度层。这样做主要是想避免不得不从头计算一个批处理视图的过程，而是尝试将几乎所有的计算都放在速度层。通过这样的设计，它<strong data-nodeid="193771">避免了 Lambda 架构的最大问题</strong>：必须对同样的业务需求进行两次实现（批处理层与速度层）。</p>
<p data-nodeid="193675">下图是Kappa 和 Lambda 架构的并排比较。你可以清楚地看到，在 Kappa 架构中唯一缺少的部分是批处理层。</p>
<p data-nodeid="193676"><img src="https://s0.lgstatic.com/i/image/M00/4D/5F/Ciqc1F9aCAaAY0BcAACwzWL9f_Y295.png" alt="Drawing 4.png" data-nodeid="193775"></p>
<p data-nodeid="193677">与 Lambda 架构相比，Kappa 架构只会在必要时才对历史数据进行重复计算，而不会定时计算，所有计算都由流计算引擎完成，所以代码只用维护一份。平时数据通常保存在消息队列中，计算的时候从中读取即可，需要一天就读一天，需要全量就读全量，读取后利用流处理系统进行处理，然后输出到服务层，新旧版本输出的结果互不相关，如下图所示。</p>
<p data-nodeid="193678"><img src="https://s0.lgstatic.com/i/image/M00/4D/5F/Ciqc1F9aCA6Adv0HAAElTPp3uJs815.png" alt="Lark20200910-185956.png" data-nodeid="193779"></p>
<p data-nodeid="193679">目前 Kappa 架构<strong data-nodeid="193785">最大的缺点在于</strong>：由于只使用流处理系统作为计算引擎，所以它的数据处理能力有限，无法对全量的历史数据进行很好的分析。</p>
<p data-nodeid="193680">那么在实际情况中，该如何对Lambda 架构和 Kappa 架构进行选择呢？在下面的表格中，我详细对比了两种架构的数据处理能力、机器开销等，你可以根据需求进行选择。</p>
<p data-nodeid="193681"><img src="https://s0.lgstatic.com/i/image/M00/4D/5F/Ciqc1F9aCBaAGp6bAAEzPhtEdm4797.png" alt="Lark20200910-190002.png" data-nodeid="193789"></p>
<h3 data-nodeid="193682">总结</h3>
<p data-nodeid="193683">总体而言，Lambda 架构是在批处理、流处理技术共同作用下的实时大数据架构，它在某种程度上解决了我们上一课时的问题：如何实时获取数据进行处理，并提供统一的查询方式。值得一提的是搜索引擎就是典型的 Lambda 架构，很多开源大数据分析平台也采用了 Lambda 架构，如 Druid 等。而这一课时介绍的两种常见的实时大数据架构，也算是对整个实践模块的升华。</p>
<p data-nodeid="193684">到目前为止，所有的课程就结束了。但别急着走，我还会更新一个彩蛋：“如何成为&nbsp;Spark&nbsp;Contributor”，向你介绍如何向Spark贡献代码。此外还会在结束语中介绍一些业界比较新颖的架构和理念，也是对整个课程的总结。</p>
<p data-nodeid="193685" class="">最后，我邀请你参与对本专栏的评价，你的每一个观点对我们来说都是最重要的。<a href="https://wj.qq.com/s2/7164306/3a46/" data-nodeid="193796">点击链接，即可参与评价</a>，还有机会获得惊喜奖品！</p>

---

### 精选评论


