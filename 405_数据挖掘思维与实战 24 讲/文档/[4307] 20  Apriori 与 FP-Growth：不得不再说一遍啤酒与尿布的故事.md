<p data-nodeid="11343" class="">这一课时，我们进入第四种数据挖掘算法——关联分析的学习。关联分析是一种无监督学习，它的目标就是从大数据中找出那些经常一起出现的东西，不管是商品还是其他什么 item，然后靠这些结果总结出关联规则以用于后续的商业目的或者其他项目需求。</p>
<h3 data-nodeid="11344">一个例子</h3>
<p data-nodeid="11345">不管你在哪一个数据挖掘课堂上，几乎都会听到这样一个“都市传说”：在一个大型超市中，数据分析人员整理了一整年的购物篮数据，来分析大家都买过什么样的东西。就在对购物篮的数据进行分析的时候，分析人员惊奇地发现，与“尿不湿”出现在一个购物小票上最频繁的商品竟然是啤酒。</p>
<p data-nodeid="11346">这个结果背后的原因，是女人嘱托丈夫去超市给孩子买尿不湿，而丈夫通常会顺便买上一些自己喜欢的啤酒。超市发现了这个神奇的组合，于是毫不犹豫地把尿布和啤酒摆在了一起进行销售，以起到互相促进的作用。</p>
<p data-nodeid="11347">关于这个故事是否真实发生过，我们保持怀疑的态度，但是这个故事确实反映了数据挖掘在商业运作中的价值。同时，购物篮分析也早已经是大型商超必备的技术手段。那么，如何发现这些潜在的价值，就用到了我们这一课时要讲的关联分析算法。</p>
<h3 data-nodeid="11348">算法原理</h3>
<p data-nodeid="11349">要了解算法原理，首先我们需要对关联分析中的一些概念进行一下解释。</p>
<p data-nodeid="11350">为了方便说明，我在这里编造了十条购物小票数据（如有雷同，纯属巧合），如下表所示。</p>
<div data-nodeid="11351"><p style="text-align:center">表1  购物小票数据</p></div>
<table data-nodeid="11353">
<thead data-nodeid="11354">
<tr data-nodeid="11355">
<th data-org-content="**序号**" data-nodeid="11357"><strong data-nodeid="11559">序号</strong></th>
<th data-org-content="**购物小票**" data-nodeid="11358"><strong data-nodeid="11563">购物小票</strong></th>
</tr>
</thead>
<tbody data-nodeid="11361">
<tr data-nodeid="11362">
<td data-org-content="1" data-nodeid="11363">1</td>
<td data-org-content="尿布，啤酒，奶粉，洋葱" data-nodeid="11364">尿布，啤酒，奶粉，洋葱</td>
</tr>
<tr data-nodeid="11365">
<td data-org-content="2" data-nodeid="11366">2</td>
<td data-org-content="尿布，啤酒，奶粉，洋葱" data-nodeid="11367">尿布，啤酒，奶粉，洋葱</td>
</tr>
<tr data-nodeid="11368">
<td data-org-content="3" data-nodeid="11369">3</td>
<td data-org-content="尿布，啤酒，苹果，洋葱" data-nodeid="11370">尿布，啤酒，苹果，洋葱</td>
</tr>
<tr data-nodeid="11371">
<td data-org-content="4" data-nodeid="11372">4</td>
<td data-org-content="尿布，啤酒，苹果" data-nodeid="11373">尿布，啤酒，苹果</td>
</tr>
<tr data-nodeid="11374">
<td data-org-content="5" data-nodeid="11375">5</td>
<td data-org-content="尿布，啤酒，奶粉" data-nodeid="11376">尿布，啤酒，奶粉</td>
</tr>
<tr data-nodeid="11377">
<td data-org-content="6" data-nodeid="11378">6</td>
<td data-org-content="尿布，啤酒，奶粉" data-nodeid="11379">尿布，啤酒，奶粉</td>
</tr>
<tr data-nodeid="11380">
<td data-org-content="7" data-nodeid="11381">7</td>
<td data-org-content="尿布，啤酒，苹果" data-nodeid="11382">尿布，啤酒，苹果</td>
</tr>
<tr data-nodeid="11383">
<td data-org-content="8" data-nodeid="11384">8</td>
<td data-org-content="尿布，啤酒，苹果" data-nodeid="11385">尿布，啤酒，苹果</td>
</tr>
<tr data-nodeid="11386">
<td data-org-content="9" data-nodeid="11387">9</td>
<td data-org-content="尿布，奶粉，洋葱" data-nodeid="11388">尿布，奶粉，洋葱</td>
</tr>
<tr data-nodeid="11389">
<td data-org-content="10" data-nodeid="11390">10</td>
<td data-org-content="奶粉，洋葱" data-nodeid="11391">奶粉，洋葱</td>
</tr>
</tbody>
</table>
<p data-nodeid="11392"><strong data-nodeid="11588">项集（Item&nbsp;Set）：</strong> 第一个要介绍的概念叫项集，中文名称有点拗口，还是看英文比较容易理解。项集可以是单个的项，也可以是一系列项目的合集。在我们的例子中，项目就是啤酒、尿布等商品，一个小票上的内容就可以看作一个项集，通过关联分析得到的经常一起出现的啤酒和尿布可以称为一个“频繁项集”。</p>
<p data-nodeid="11393"><strong data-nodeid="11593">关联规则：</strong> 根据频繁项集挖掘出的结果，例如 {尿布}→{啤酒}，规则的左侧称为先导，右侧称为后继。</p>
<p data-nodeid="11394"><strong data-nodeid="11598">支持度：</strong> 支持度就是一个项集在数据中出现的比例。在我们的 10 条数据中，{尿布} 出现了 9 次，那它的支持度就是 0.9；{啤酒} 出现了 8 次，啤酒的支持度就是 0.8。{尿布，啤酒} 的支持度是 8/10=0.8，以此类推。支持度还可以用来判定一条规则是否还需要继续进行挖掘，如果支持度已经很低，再加入新的项肯定会更低，挖掘的意义不大。</p>
<p data-nodeid="11395"><img src="https://s0.lgstatic.com/i/image/M00/59/EE/CgqCHl9y3iuAEKNhAAAW0lZ7m_s551.png" alt="1.png" data-nodeid="11601"></p>
<p data-nodeid="11396"><strong data-nodeid="11606">置信度：</strong> 置信度指的是在一条规则中，出现先导也出现后继的比例。我们可以用公式来看一下，</p>
<p data-nodeid="11397">置信度表示的是一条规则的可靠程度。</p>
<p data-nodeid="11398"><img src="https://s0.lgstatic.com/i/image/M00/59/E3/Ciqc1F9y3jiAJ3b7AAAaFiqi3c0554.png" alt="2.png" data-nodeid="11610"></p>
<p data-nodeid="11399"><strong data-nodeid="11615">提升度：</strong> 在置信度中，只考虑了规则中的先导与后继同时发生的情况，而对于后继单独发生的情况没有加以考虑。所以又有人提出了一个提升度，用来衡量先导和后继的独立性。比如在前面我们算出“尿布→啤酒”的置信度为 0.89，这说明买了“尿布”的人里有 89% 会买“啤酒”，这看起来已经很高了。但是如果在没有买“尿布”的购物小票中，购买“啤酒”的概率仍然为 0.89，那其实购买“尿布”和购买“啤酒”并没有什么关系。所以，提升度的计算公式如下：</p>
<p data-nodeid="11400"><img src="https://s0.lgstatic.com/i/image/M00/59/EE/CgqCHl9y3kSAZT01AAAaVQmw6Jw591.png" alt="3.png" data-nodeid="11618"></p>
<p data-nodeid="11401">如果提升度大于 1，说明是有提升的。</p>
<p data-nodeid="11402"><strong data-nodeid="11624">确信度：</strong> 最后一个指标是确信度，确信度指的是对于一条规则，不发生先导而发生后继的概率与这条规则错误的概率比值。公式如下：</p>
<p data-nodeid="11403"><img src="https://s0.lgstatic.com/i/image/M00/59/EE/CgqCHl9y3k6AIxI2AAAYT6dsokk718.png" alt="4.png" data-nodeid="11627"></p>
<p data-nodeid="11404">上面的结果为 1.82，这说明该条规则是真的概率比它只是偶然发生的概率高 82%。</p>
<p data-nodeid="11405">一口气看了这么多的概念和指标，下面终于进入我们的正题环节，看看在关联挖掘中的算法原理。</p>
<h4 data-nodeid="11406">Apriori</h4>
<p data-nodeid="11407">关联挖掘的目标已经很明确了，而关联挖掘的步骤也就只有两个：第一步是找出频繁项集，第二步是从频繁项集中提取规则。Apriori 算法的核心就是：如果某个项集是频繁项集，那么它的全部子集也都是频繁项集。</p>
<p data-nodeid="11408">为了方便了解算法的效果，我预先画出了数据集中所有可能存在的项集关系，就如下图显示：</p>
<p data-nodeid="11409"><img src="https://s0.lgstatic.com/i/image/M00/59/6E/CgqCHl9xgbCAbSHyAADAlRv3T18240.png" alt="Drawing 4.png" data-nodeid="11635"></p>
<p data-nodeid="11410">首先，需要设定一个最小支持度阈值，假设我们设定为 0.5，那么高于 0.5 的就认为是频繁项集。然后，我们计算出所有单个商品的支持度，如下表所示：</p>
<div data-nodeid="11411"><p style="text-align:center">表2  一阶项集支持度</p></div>
<table data-nodeid="11413">
<thead data-nodeid="11414">
<tr data-nodeid="11415">
<th data-org-content="**项集**" data-nodeid="11417"><strong data-nodeid="11640">项集</strong></th>
<th data-org-content="**支持度**" data-nodeid="11418"><strong data-nodeid="11644">支持度</strong></th>
</tr>
</thead>
<tbody data-nodeid="11421">
<tr data-nodeid="11422">
<td data-org-content="尿布" data-nodeid="11423">尿布</td>
<td data-org-content="0.9" data-nodeid="11424">0.9</td>
</tr>
<tr data-nodeid="11425">
<td data-org-content="啤酒" data-nodeid="11426">啤酒</td>
<td data-org-content="0.8" data-nodeid="11427">0.8</td>
</tr>
<tr data-nodeid="11428">
<td data-org-content="奶粉" data-nodeid="11429">奶粉</td>
<td data-org-content="0.6" data-nodeid="11430">0.6</td>
</tr>
<tr data-nodeid="11431">
<td data-org-content="洋葱" data-nodeid="11432">洋葱</td>
<td data-org-content="0.5" data-nodeid="11433">0.5</td>
</tr>
<tr data-nodeid="11434">
<td data-org-content="苹果" data-nodeid="11435">苹果</td>
<td data-org-content="0.4" data-nodeid="11436">0.4</td>
</tr>
</tbody>
</table>
<p data-nodeid="11437">从这里可以看出，“苹果”的支持度达不到阈值，于是把它删掉。因此，所有跟“苹果”相关的父集也都是低频的，在后续的计算也不会涉及了，就是下图标绿色的这些。</p>
<p data-nodeid="11438"><img src="https://s0.lgstatic.com/i/image/M00/59/6E/CgqCHl9xgeKAclXiAAF2xSzo8xk775.png" alt="Drawing 5.png" data-nodeid="11658"></p>
<p data-nodeid="11439">接下来我们再计算二阶的项集支持度。</p>
<div data-nodeid="11440"><p style="text-align:center">表3  二阶项集支持度</p></div>
<table data-nodeid="11442">
<thead data-nodeid="11443">
<tr data-nodeid="11444">
<th data-org-content="**项集**" data-nodeid="11446"><strong data-nodeid="11663">项集</strong></th>
<th data-org-content="**支持度**" data-nodeid="11447"><strong data-nodeid="11667">支持度</strong></th>
</tr>
</thead>
<tbody data-nodeid="11450">
<tr data-nodeid="11451">
<td data-org-content="尿布，啤酒" data-nodeid="11452">尿布，啤酒</td>
<td data-org-content="0.8" data-nodeid="11453">0.8</td>
</tr>
<tr data-nodeid="11454">
<td data-org-content="尿布，奶粉" data-nodeid="11455">尿布，奶粉</td>
<td data-org-content="0.5" data-nodeid="11456">0.5</td>
</tr>
<tr data-nodeid="11457">
<td data-org-content="尿布，洋葱" data-nodeid="11458">尿布，洋葱</td>
<td data-org-content="0.4" data-nodeid="11459">0.4</td>
</tr>
<tr data-nodeid="11460">
<td data-org-content="啤酒，洋葱" data-nodeid="11461">啤酒，洋葱</td>
<td data-org-content="0.3" data-nodeid="11462">0.3</td>
</tr>
<tr data-nodeid="11463">
<td data-org-content="啤酒，奶粉" data-nodeid="11464">啤酒，奶粉</td>
<td data-org-content="0.4" data-nodeid="11465">0.4</td>
</tr>
<tr data-nodeid="11466">
<td data-org-content="洋葱，奶粉" data-nodeid="11467">洋葱，奶粉</td>
<td data-org-content="0.4" data-nodeid="11468">0.4</td>
</tr>
</tbody>
</table>
<p data-nodeid="11469">到了这一步，后四个项集的支持度已经达不到我们的阈值 0.5，于是也删掉。</p>
<p data-nodeid="11470">接下来再计算三阶的项集支持度，这个时候发现已经没有可用的三阶项集了，所以算法计算结束。这时候我们得到了三组频繁项集 {尿布，啤酒}、{尿布，洋葱}、{尿布，奶粉}。接下来从这三个频繁项集，我们可以<strong data-nodeid="11686">得到三个关联关系：尿布→啤酒，尿布→洋葱，尿布→奶粉</strong>。根据前面的公式，分别计算这三个关系的置信度、提升度和确信度。</p>
<div data-nodeid="11471"><p style="text-align:center">表4  三个关联关系的置信度、提升度和确信度</p></div>
<table data-nodeid="11473">
<thead data-nodeid="11474">
<tr data-nodeid="11475">
<th data-org-content="**关系**" data-nodeid="11477"><strong data-nodeid="11690">关系</strong></th>
<th data-org-content="**支持度**" data-nodeid="11478"><strong data-nodeid="11694">支持度</strong></th>
<th data-org-content="**置信度**" data-nodeid="11479"><strong data-nodeid="11698">置信度</strong></th>
<th data-org-content="**提升度**" data-nodeid="11480"><strong data-nodeid="11702">提升度</strong></th>
<th data-org-content="**确信度**" data-nodeid="11481"><strong data-nodeid="11706">确信度</strong></th>
</tr>
</thead>
<tbody data-nodeid="11487">
<tr data-nodeid="11488">
<td data-org-content="尿布" data-nodeid="11489">尿布</td>
<td data-org-content="0.9" data-nodeid="11490">0.9</td>
<td data-org-content="|||" data-nodeid="11491">|||</td>
<td data-nodeid="11492"></td>
<td data-nodeid="11493"></td>
</tr>
<tr data-nodeid="11494">
<td data-org-content="啤酒" data-nodeid="11495">啤酒</td>
<td data-org-content="0.8" data-nodeid="11496">0.8</td>
<td data-org-content="|||" data-nodeid="11497">|||</td>
<td data-nodeid="11498"></td>
<td data-nodeid="11499"></td>
</tr>
<tr data-nodeid="11500">
<td data-org-content="洋葱" data-nodeid="11501">洋葱</td>
<td data-org-content="0.5" data-nodeid="11502">0.5</td>
<td data-org-content="|||" data-nodeid="11503">|||</td>
<td data-nodeid="11504"></td>
<td data-nodeid="11505"></td>
</tr>
<tr data-nodeid="11506">
<td data-org-content="奶粉" data-nodeid="11507">奶粉</td>
<td data-org-content="0.6" data-nodeid="11508">0.6</td>
<td data-org-content="|||" data-nodeid="11509">|||</td>
<td data-nodeid="11510"></td>
<td data-nodeid="11511"></td>
</tr>
<tr data-nodeid="11512">
<td data-org-content="尿布→啤酒" data-nodeid="11513">尿布→啤酒</td>
<td data-org-content="0.8" data-nodeid="11514">0.8</td>
<td data-org-content="0.89" data-nodeid="11515">0.89</td>
<td data-org-content="1.1" data-nodeid="11516">1.1</td>
<td data-org-content="1.82" data-nodeid="11517">1.82</td>
</tr>
<tr data-nodeid="11518">
<td data-org-content="尿布→奶粉" data-nodeid="11519">尿布→奶粉</td>
<td data-org-content="0.5" data-nodeid="11520">0.5</td>
<td data-org-content="0.55" data-nodeid="11521">0.55</td>
<td data-org-content="0.926" data-nodeid="11522">0.926</td>
<td data-org-content="0.89" data-nodeid="11523">0.89</td>
</tr>
</tbody>
</table>
<p data-nodeid="11524">到了这里，再根据我们的需求设定阈值来筛选最终需要留下来的规则。</p>
<h4 data-nodeid="11525">FP-Growth（Frequent&nbsp;Pattern&nbsp;Growth）</h4>
<p data-nodeid="11526">根据上面的 Apriori 计算过程，我们可以知道 Apriori 计算的过程中，会使用排列组合的方式列举出所有可能的项集，每一次计算都需要重新读取整个数据集，从而计算本轮次的项集支持度。所以 Apriori 会耗费大量的计算资源，这时候就有了一个更高效的算法——FP-Growth 算法。</p>
<p data-nodeid="11527">Apriori 算法一开始需要对所有的规则进行枚举，然后再进行计算，而 FP-Growth 则是首先使用数据生成一棵 FP-Growth 树，然后再根据这棵树来生成频繁项集。下面我们来看一下如何构建 FP 树。</p>
<p data-nodeid="11528">仍然使用上面编的购物小票数据，并对每一个小票里的项按统一的顺序进行排序，设置一个空集作为根节点。我们把第一条数据录入这个树中，每一个单项作为上一个节点的叶子节点，旁边的数据代表该路径访问的次数。下图就是我们录入第一条数据后的结果：</p>
<p data-nodeid="11529"><img src="https://s0.lgstatic.com/i/image/M00/59/6E/CgqCHl9xgf-AFIGbAAA51sNwDnE073.png" alt="Drawing 6.png" data-nodeid="11736"></p>
<p data-nodeid="11530">第二条数据与第一条一样，所以只有数字的变化，节点不会发生变化。</p>
<p data-nodeid="11531">接着输入第三条数据。</p>
<p data-nodeid="11532">输入完第二、三条数据之后的结果如下，其中由第一个洋葱和第二个洋葱直接画了一条虚线，标识它们是同一项。</p>
<p data-nodeid="11533"><img src="https://s0.lgstatic.com/i/image/M00/59/63/Ciqc1F9xggWAI0JRAABFAWcB1c0239.png" alt="Drawing 7.png" data-nodeid="11742"></p>
<p data-nodeid="11534">接下来我们把所有的数据都插入到这棵树上。</p>
<p data-nodeid="11535"><img src="https://s0.lgstatic.com/i/image/M00/59/63/Ciqc1F9xggmAUKZ6AABvvu1fW4M677.png" alt="Drawing 8.png" data-nodeid="11746"></p>
<p data-nodeid="11536">这个时候我们得到的是完整的 FP 树，接下来我们要从这里面去寻找频繁项集。当然首先也要设定一个最小频度的阈值，然后从叶子节点开始，也就是最低频的节点。如果频度高于阈值那么就收录所有以该项结尾的项集，否则就向上继续检索。</p>
<p data-nodeid="11537">至于后面的计算关联规则的方法跟上面就没有什么区别了，这里不再赘述。下面我们尝试使用代码来实现关联规则的发现。</p>
<h3 data-nodeid="11538">尝试动手</h3>
<p data-nodeid="15113" class="">在我们之前使用的 sklearn 算法工具包中没有 apriori 算法，所以这次我们需要先安装一个 efficient-apriori 算法包。而我们这次所使用的数据集，也就是我在上面提到的这个“啤酒尿布”数据。这个算法包使用起来也极其简单。</p>





<pre class="lang-dart" data-nodeid="11540"><code data-language="dart"><span class="hljs-string">'''记得安装包
pip install efficient-apriori
'''</span>
from efficient_apriori <span class="hljs-keyword">import</span> apriori
# 设置数据集
data = [(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>,<span class="hljs-string">'洋葱'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>,<span class="hljs-string">'洋葱'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'苹果'</span>,<span class="hljs-string">'洋葱'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'苹果'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'苹果'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'苹果'</span>),
(<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'洋葱'</span>),
(<span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'洋葱'</span>)
]
# 挖掘频繁项集和规则
itemsets, rules = apriori(data, min_support=<span class="hljs-number">0.4</span>, min_confidence=<span class="hljs-number">1</span>)
<span class="hljs-built_in">print</span>(itemsets)
<span class="hljs-built_in">print</span>(rules)
#输出结果
{<span class="hljs-number">1</span>: {(<span class="hljs-string">'奶粉'</span>,): <span class="hljs-number">6</span>, (<span class="hljs-string">'洋葱'</span>,): <span class="hljs-number">5</span>, (<span class="hljs-string">'尿布'</span>,): <span class="hljs-number">9</span>, (<span class="hljs-string">'啤酒'</span>,): <span class="hljs-number">8</span>, (<span class="hljs-string">'苹果'</span>,): <span class="hljs-number">4</span>}, <span class="hljs-number">2</span>: {(<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>): <span class="hljs-number">4</span>, (<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'尿布'</span>): <span class="hljs-number">8</span>, (<span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'尿布'</span>): <span class="hljs-number">5</span>, (<span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'洋葱'</span>): <span class="hljs-number">4</span>, (<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'洋葱'</span>): <span class="hljs-number">4</span>, (<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'苹果'</span>): <span class="hljs-number">4</span>, (<span class="hljs-string">'尿布'</span>, <span class="hljs-string">'苹果'</span>): <span class="hljs-number">4</span>}, <span class="hljs-number">3</span>: {(<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'尿布'</span>): <span class="hljs-number">4</span>, (<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'尿布'</span>, <span class="hljs-string">'苹果'</span>): <span class="hljs-number">4</span>}}
[{啤酒} -&gt; {尿布}, {苹果} -&gt; {啤酒}, {苹果} -&gt; {尿布}, {啤酒, 奶粉} -&gt; {尿布}, {尿布, 苹果} -&gt; {啤酒}, {啤酒, 苹果} -&gt; {尿布}, {苹果} -&gt; {啤酒, 尿布}]
#把min_support设置成<span class="hljs-number">0.5</span>输出结果
{<span class="hljs-number">1</span>: {(<span class="hljs-string">'尿布'</span>,): <span class="hljs-number">9</span>, (<span class="hljs-string">'奶粉'</span>,): <span class="hljs-number">6</span>, (<span class="hljs-string">'啤酒'</span>,): <span class="hljs-number">8</span>, (<span class="hljs-string">'洋葱'</span>,): <span class="hljs-number">5</span>}, <span class="hljs-number">2</span>: {(<span class="hljs-string">'啤酒'</span>, <span class="hljs-string">'尿布'</span>): <span class="hljs-number">8</span>, (<span class="hljs-string">'奶粉'</span>, <span class="hljs-string">'尿布'</span>): <span class="hljs-number">5</span>}}
[{啤酒} -&gt; {尿布}]
</code></pre>
<p data-nodeid="11541">通过上面的代码，我们就成功使用了 apriori 算法，对我们自己编造的数据集完成了关联关系挖掘，当在设置最小支持度为 0.4 的时候，我们找到了 7 条规则；而我们把最小支持度改为 0.5 以后，只剩下 {啤酒} -&gt; {尿布} 这一条规则了。你学会了吗？</p>
<h3 data-nodeid="11542">总结</h3>
<p data-nodeid="11543">这节课里，我们介绍了两种关联关系挖掘的方法，其中 Apriori 使用了穷举的方式，而 FP-Growth 使用了树形结构来提高速度。关联关系挖掘通常使用的算法都非常简单，或者我们可以把关联关系挖掘转化成分类问题、聚类问题来解决都是可以的。</p>
<p data-nodeid="11544">在这节课中，我们还介绍了关联关系的评估指标，不管是用什么算法来挖掘的关联关系，都可以使用这些指标来进行评估。</p>
<p data-nodeid="11545">下一课时，我们会进入关联关系挖掘的实践课程，看看如何使用关联关系挖掘来解决业务中的问题。</p>
<blockquote data-nodeid="11546">
<p data-nodeid="11547" class="">点击下方链接查看源代码（不定时更新）以及相关工具：<br>
<a href="https://github.com/icegomic/GomicDatamining/tree/master/LagouCodes" data-nodeid="11760">https://github.com/icegomic/GomicDatamining/tree/master/LagouCodes</a></p>
</blockquote>

---

### 精选评论


