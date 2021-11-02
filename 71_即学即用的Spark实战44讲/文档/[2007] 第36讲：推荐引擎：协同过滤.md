<p data-nodeid="56193">在开始今天的课程前，我们先来看看上节课的习题：如果不进行归一化处理，那么会对聚类结果有什么影响呢？ 答案是聚类结果会被数值最大的那个维度所左右。</p>
<p data-nodeid="56194">然后我们进行今天的课程学习。推荐系统是计算广告中的重要组成部分，也是典型的大数据应用。在大数据时代，我们真正需要或者感兴趣的物品和信息淹没在了大量的数据中，使得很难发现它们。为了解决信息检索的问题，在门户网站的早期，主要使用分类目录让用户快速找到所需要的信息，接着门户网站就演变出了搜索引擎。和单纯使用分类目录相比，搜索引擎无疑是更加聪明的，它帮助用户查询给定的信息，并为用户返回结果。分类目录是静态的、覆盖率低的索引，搜索引擎则采取主动并且针对全网的索引策略，直到现在，搜索引擎也是信息检索最主流的方案。在累积了大量的用户行为数据后，<strong data-nodeid="56283">推荐系统</strong>诞生了，它不需要用户主动搜索，系统会将它认为用户感兴趣的信息主动推送给用户，因此推荐系统的核心在于从海量数据中将每个用户感兴趣的信息筛选出来。</p>
<p data-nodeid="56195"><strong data-nodeid="56288">协同过滤算法</strong>是诞生最早并且非常著名的推荐算法，是很多电商平台（如亚马逊、京东等）推荐系统的核心。算法通过对用户的历史行为数据进行分析来发现用户的偏好，然后基于不同的偏好对用户进行分组，并推荐相似的商品，因此协同指的是借鉴与你相似的人的观点进行推荐。协同过滤主要分为 3 类：基于用户的协同过滤、基于商品的协同过滤和基于模型的协同过滤。这也是本课时的主要内容。</p>
<h3 data-nodeid="56196">基于用户的协同过滤</h3>
<p data-nodeid="56197">基于用户的协同过滤的核心是根据用户对商品的偏好，来计算不同用户之间的相似性，并在相似的用户之间进行推荐，如下图所示。</p>
<p data-nodeid="56198"><img alt="3.png" src="https://s0.lgstatic.com/i/image/M00/41/45/Ciqc1F804i2AWCkBAACFXL9YB6I503.png" data-nodeid="56293"></p>
<p data-nodeid="56790">在上图中，用户 1 买了商品 1、2、3、4，而用户 3 的历史数据表明他买了商品 2、3，这样我们就认为用户 1、3 是一类用户。于是，就可以将用户 1 买过而用户 3 没有买过的商品 1、4 推荐给用户 3。</p>

<p data-nodeid="60378">完成这个工作的第一步是根据用户行为的历史数据找到用户偏好的数据。表明用户偏好的方式可以有很多种，如转发、点击和购买等。我们用以下表格整理了判断用户偏好的主要方式及其类型和特征，比如对物品的满意程度评分：1~5 分，满分是 5 分。</p>



<p data-nodeid="56201"><img alt="7.png" src="https://s0.lgstatic.com/i/image/M00/41/50/CgqCHl804kSAFoFvAAKTGF84sG4560.png" data-nodeid="56298"></p>
<p data-nodeid="56202">接着需要将这些行为统统量化为一个值，例如加权平均等，这样，我们会得到一个矩阵，如下表所示。</p>
<p data-nodeid="56203"><img alt="4.png" src="https://s0.lgstatic.com/i/image/M00/41/50/CgqCHl804mqAamImAABcrnEaNUk583.png" data-nodeid="56302"></p>
<p data-nodeid="65180">用户的数量决定了矩阵的行数，商品的数量决定了矩阵的列数。可以想到，在类似于淘宝、京东这样的平台，这个矩阵是非常巨大并且稀疏的。构建好这样的矩阵后，我们就可以利用数学的方法来比较相似性，常见的相似性度量有以下几种方法。</p>




<ul data-nodeid="56205">
<li data-nodeid="56206">
<p data-nodeid="56207">欧氏距离：欧氏距离是所有距离测度中最简单的，也是最基本的。数学上，两个 <em data-nodeid="56337">n</em> 维向量 <em data-nodeid="56338">（a<sub>1</sub>,a<sub>2</sub>,…,a<sub>n</sub>）和（b<sub>1</sub>,b<sub>2</sub>,…,b<sub>n</sub>）</em> 之间的欧氏距离公式为：</p>
</li>
</ul>
<p data-nodeid="56208"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/41/49/CgqCHl802b2ATtcfAAASglzDNRo974.png" data-nodeid="56341"></p>
<ul data-nodeid="56209">
<li data-nodeid="56210">
<p data-nodeid="56211">平方欧氏距离：如名称所示，平方欧氏距离的值为欧氏距离的平方，两个 <em data-nodeid="56375">n</em> 维向量 <em data-nodeid="56376">（a<sub>1</sub>,a<sub>2</sub>,…,a<sub>n</sub>）和（b<sub>1</sub>,b<sub>2</sub>,…,b<sub>n</sub>）</em> 之间的平方欧氏距离公式为：</p>
</li>
</ul>
<p data-nodeid="56212"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/41/3E/Ciqc1F802ciAPA1WAAAROI8X3hg461.png" data-nodeid="56379"></p>
<ul data-nodeid="56213">
<li data-nodeid="56214">
<p data-nodeid="56215">曼哈顿距离：两个点之间的曼哈顿距离是它们坐标差的绝对值之和，如下图所示。</p>
</li>
</ul>
<p data-nodeid="56216"><img alt="Drawing 5.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802c6AZd0DAABHdRfFyQ0406.png" data-nodeid="56383"></p>
<p data-nodeid="56217">点（2,2）与点（6,6）之间的欧氏距离为&nbsp;5.65（虚线所示），曼哈顿距离为&nbsp;8（实线所示）。两个 <em data-nodeid="56417">n</em> 维向量 <em data-nodeid="56418">（a<sub>1</sub>,a<sub>2</sub>,…,a<sub>n</sub>）和（b<sub>1</sub>,b<sub>2</sub>,…,b<sub>n</sub>）</em> 之间的曼哈顿距离公式为：</p>
<p data-nodeid="56218"><img alt="Drawing 6.png" src="https://s0.lgstatic.com/i/image/M00/41/3E/Ciqc1F802dSAPpyFAAAOXecxmvk995.png" data-nodeid="56421"></p>
<ul data-nodeid="66380">
<li data-nodeid="66381">
<p data-nodeid="66382">余弦距离：余弦距离是用向量空间中两个向量夹角的余弦值，作为两个个体间差异大小的度量。余弦距离需要将待比较的两个点视为从原点指向它们的向量，向量的夹角为 <em data-nodeid="66388">θ</em>，如下图所示。</p>
</li>
</ul>

<p data-nodeid="56222"><img alt="Drawing 7.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802dqAH3JqAAAnCnpaXvs111.png" data-nodeid="56430"></p>
<p data-nodeid="56223">两个 <em data-nodeid="56464">n</em> 维向量 <em data-nodeid="56465">（a<sub>1</sub>,a<sub>2</sub>,…,a<sub>n</sub>）和（b<sub>1</sub>,b<sub>2</sub>,…,b<sub>n</sub>）</em> 之间的余弦距离公式为：</p>
<p data-nodeid="56224"><em data-nodeid="56473">d</em> =&nbsp;1−cos <em data-nodeid="56474">θ</em></p>
<p data-nodeid="56225">其中</p>
<p data-nodeid="56226"><img alt="Drawing 8.png" src="https://s0.lgstatic.com/i/image/M00/41/3E/Ciqc1F802eOAIGXUAAAitgJcmew313.png" data-nodeid="56478"></p>
<p data-nodeid="56227">余弦距离一般用在文本聚类中。</p>
<p data-nodeid="56228">如果我们选择欧氏距离作为相似性度量，用户两两之间的距离和距离倒数就可以填入下表中，我制作了一个空表格来举例。</p>
<p data-nodeid="56229"><img alt="5.png" src="https://s0.lgstatic.com/i/image/M00/41/50/CgqCHl804oeANWTFAABGQOAQMAA378.png" data-nodeid="56483"></p>
<p data-nodeid="56230">两个用户越不相似，则他们之间的距离越远、距离值越大，这对于相似性比较不够直观，因此我们通常采用距离倒数来表现相似性，也就是说越相似，其值越小。</p>
<p data-nodeid="56231">假设与用户&nbsp;3&nbsp;最相似的用户是用户&nbsp;1，这样我们就可以根据用户 1 的行为来对用户 3 进行推荐，但要注意的是，不要推荐用户 3 购买过的商品。从上面这个过程可以看到，协同过滤的大致步骤是首先进行<strong data-nodeid="56498">收集数据</strong>，其次<strong data-nodeid="56499">计算相似度</strong>，最后<strong data-nodeid="56500">进行推荐</strong>。</p>
<p data-nodeid="73587">基于用户的协同过滤需要累积一定的用户行为数据，因此对新网站不是特别友好。另外，通常来说一个大型的电子商务网站，每个用户的商品占比是极少的，这导致了不同用户之间的购买商品重叠较少，算法可能无法找到与某个用户相似的邻居。这种协同过滤需要耗费大量的计算资源，最坏的情况下，协同过滤的复杂度是 O（m × n），其中 m 是用户数量，n 是商品数量，但由于在实际情况中，大量用户只购买过很少一部分商品，因此实际的计算复杂度趋近于 O（m + n）。</p>






<h3 data-nodeid="56233">基于商品的协同过滤</h3>
<p data-nodeid="74787">基于商品的协同过滤与基于用户的协同过滤很相似，核心是基于相似商品对用户进行推荐。如下图所示，如果用户 1 同时购买了商品 1 和 3，那么认为商品 1 和 3 有很强的相关性，那么在用户 3 购买商品 3 时，会给他推荐商品 1。</p>

<p data-nodeid="56235"><img alt="6.png" src="https://s0.lgstatic.com/i/image/M00/41/50/CgqCHl804pWAEl_3AACNGSxsX2U954.png" data-nodeid="56506"></p>
<p data-nodeid="79587">基于商品的协同过滤的基本步骤依然是：收集数据，计算相似度，进行推荐。在收集数据时，表现形式与表 2 类似，只需将矩阵行列进行转置，如下表所示。</p>




<p data-nodeid="56237"><img alt="1.png" src="https://s0.lgstatic.com/i/image/M00/41/50/CgqCHl804pyAHAGcAAA64gImpCw948.png" data-nodeid="56510"></p>
<p data-nodeid="56238">同理，基于这个矩阵，可以得到商品与商品之间的关联性（欧氏距离），如下表所示。</p>
<p data-nodeid="56239"><img alt="2.png" src="https://s0.lgstatic.com/i/image/M00/41/45/Ciqc1F804qWAZgMmAABD-r4jFpY084.png" data-nodeid="56514"></p>
<p data-nodeid="56240">当用户购买了某个商品后，检索上表的结果，将把和该商品关联性最大的商品推荐给用户。</p>
<p data-nodeid="56241">基于商品的协同过滤相比于基于用户的协同过滤来说，<strong data-nodeid="56521">更加稳定</strong>，初期可以凭经验预设一小部分商品之间的相关性，也能得到不错的效果。</p>
<h3 data-nodeid="56242">两种协同过滤的对比</h3>
<p data-nodeid="81987">基于用户的协同过滤很早就提出了，而基于商品的协同过滤是由亚马逊在&nbsp;2001&nbsp;年左右发表的论文中提出并开始流行。那具体到不同的场景中，我们究竟该如何选择过滤方式呢？下面分别从计算复杂度、适用场景、推荐多样性、算法适应度来进行对比。</p>


<h4 data-nodeid="56244">计算复杂度</h4>
<p data-nodeid="56245">一般来说，大型电子商务平台的用户数量往往大大超过商品的数量，商品数量也会相对稳定，因此计算商品之间的相似度不但计算量较小，同时也不必频繁更新。但这只是针对购物平台来说，对于新闻、博客、微博等内容平台来说恰恰相反，用户的数量是一定的，而内容则是海量的，也是频繁更新的，因此从计算复杂度来说，需要<strong data-nodeid="56530">根据具体场景具体分析</strong>。</p>
<h4 data-nodeid="56246">适用场景</h4>
<p data-nodeid="56247">在类似于京东这样的购物网站，如果采取基于用户的协同过滤或许有些莫名其妙， 比如向某位用户推荐了一本书，给出的解释是和你相似的某位用户也在看这本书，但这两位用户很可能不认识，但如果根据用户的浏览记录推荐便令人信服。因此在<strong data-nodeid="56549">非社交类的平台</strong>上，<strong data-nodeid="56550">基于商品的协同</strong>过滤通常效果更好，而在<strong data-nodeid="56551">社交类平台</strong>，<strong data-nodeid="56552">选择基于用户的协同过滤</strong>似乎更合理。</p>
<h4 data-nodeid="56248">性能</h4>
<p data-nodeid="56249">推荐引擎的性能主要通过<strong data-nodeid="56563">精度</strong>、<strong data-nodeid="56564">多样性</strong>两个指标评估。研究发现在相同的数据集上同时使用基于用户和基于商品的协同过滤算法，推荐列表中只有 50% 是一样的，其余的则完全不同，但是这两个算法都有相似的精度，因此这两种协同过滤很适合互为补充。</p>
<p data-nodeid="56250">单从一个用户的角度来讲，推荐系统的多样性指的是给定一个用户，看推荐列表中的商品是否多样化，也就是要比较推荐物品之间的两两相似度。这样说来，基于商品的协同过滤显然不如基于用户的协同过滤好，因为基于商品的协同过滤就是因为相似才推荐的。</p>
<p data-nodeid="56251">而从系统的角度来说，推荐系统的多样性指的是能够给所有用户提供丰富的选择。在这种情况下，基于商品的协同过滤要远远好于基于用户的协同过滤，因为基于用户的协同过滤推荐的总是最热门的。</p>
<p data-nodeid="56252">从上面的分析可以知道，这两种推荐方法都有其合理性，但都不是最完美的。<strong data-nodeid="56572">最好的选择是将它们结合使用</strong>，当采用基于商品的协同过滤导致了系统对个人推荐的多样性不足时，可以通过加入基于用户的协同过滤增加个人推荐的多样性，从而提高精度。而当因为采用基于用户的协同过滤而使系统的整体多样性不足时，可以通过加入基于商品的协同过滤增加整体的多样性，也可以提高推荐的精度。</p>
<h4 data-nodeid="56253">算法适应度</h4>
<p data-nodeid="56254">前面都是从推荐引擎本身来进行考虑，而算法适应度则是站在用户的角度来比较这两种不同推荐的效果。作为一个用户，如果采用基于用户的协同过滤算法进行推荐，而与这位用户有着相同喜好的用户的数量又很少，可想而知这样的效果是很差的，因为基于用户的协同过滤中有个很重要的假设就是该用户的爱好和与他相似的用户的爱好相似，所以基于用户的协同过滤的算法适应度是和与这位用户有共同爱好的用户数量成正比的。</p>
<p data-nodeid="56255">而基于商品的协同过滤也有一个假设：用户会喜欢与他以前喜欢的东西相似的东西，那么我们可以计算一个用户喜欢的物品的自相似度。一个用户喜欢的商品的自相似度大，就说明他喜欢的东西都是比较相似的，也就是说他比较满足这个基本假设，那么他对基于商品的协同过滤的适应度自然比较好；反之，如果喜欢的商品自相似度小，就说明这个用户的喜好习惯并不满足这个基本假设，那么对于这种用户，用基于商品的协同过滤做出好的推荐的可能性非常低。</p>
<h3 data-nodeid="56256">基于模型的协同过滤</h3>
<p data-nodeid="56257">Netflix 公司曾经举办过一次竞赛，提供了 1998 年 10 月到 2005 年 12 月间 480 189 个用户对 17 770 部电影做出的共 103 297 638 条评分记录。每条记录的取值为 1～5 的整数，分值越高代表用户对电影的评价越高，显然，如果每位用户都对所有电影都做了评价，则评分记录的总数将达到 853&nbsp;2958&nbsp;530 条，但是在实际的数据集中，评价记录只有该数字的 1.2% 。这也很好理解，一个用户一辈子可能也只能看几百部电影。在这个竞赛中，目标是根据已有的用户评分来对那些缺失的记录进行预测。在协同过滤中，可以采用<strong data-nodeid="56582">矩阵分解</strong>来完成这个任务。</p>
<p data-nodeid="83187">矩阵分解的基本思想是把原始用户商品矩阵 <em data-nodeid="83201">R</em> 分解为两个小规模矩阵 <em data-nodeid="83202">U</em> 和 <em data-nodeid="83203">V</em>，即</p>

<p data-nodeid="56259"><img alt="Drawing 10.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802gaAD0m1AAAES2ciqeQ235.png" data-nodeid="56601"></p>
<p data-nodeid="56260">矩阵分解的结果就可以用来生成新的用户评分矩阵，也就是那些用户没有打分的电影的预测分值，从而将最高分值的电影推荐给对应的用户。矩阵分解模型有非常自然的解释，我们可以假设有若干个维度来对电影进行评价，如剧情、视觉效果、声音效果、音乐和剪辑等，每一个维度都有其分值。假设所有用户对电影的评价 <em data-nodeid="56665">R</em> 都是对这些维度进行加权线性组合得到，即： <em data-nodeid="56666">R</em> = <i>w<sub>1</sub></i> × 剧情 + <i>w<sub>2</sub></i> × 视觉效果 + <i>w<sub>3</sub></i> × 声音效果 + <i>w<sub>4</sub></i> × 音乐 + <i>w<sub>5</sub></i> × 剪辑 ，那么矩阵 <em data-nodeid="56667">V</em> 的每一列都对应了一部电影在上述 5 个维度的分值，矩阵 <em data-nodeid="56668">U</em> 中的每一列则对应了一位用户对这 5 个维度所赋予的权重，所以 <em data-nodeid="56669">V</em><sup>T</sup> <em data-nodeid="56670">U</em> 矩阵就对应了所有用户对所有电影的评分，如下图所示：</p>
<p data-nodeid="56261"><img alt="Drawing 11.png" src="https://s0.lgstatic.com/i/image/M00/41/3E/Ciqc1F802g6AOA3HAAAhNWeokKY835.png" data-nodeid="56673"></p>
<p data-nodeid="56262"><em data-nodeid="56693">V</em> 中的每一列被称为电影特征向量， <em data-nodeid="56694">V</em> 也被称为电影特征矩阵。 <em data-nodeid="56695">U</em> 中的每一列被称为用户特征向量，其中用户商品矩阵的每个元素（代表了某个用户对某个电影的打分） <i>r<sub>ui</sub></i> 为：</p>
<p data-nodeid="56263"><img alt="Drawing 12.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802hWAO15gAAANCZL2gkA918.png" data-nodeid="56698"></p>
<p data-nodeid="56264">其中 <em data-nodeid="56745">μ</em> 是评分集合 <em data-nodeid="56746">R</em> 中所有评分的均值， <i>b<sub>u</sub></i> 是用户 <em data-nodeid="56747">u</em> 的偏置， <i>b<sub>i</sub></i> 是商品的偏置。模型参数 <i>b<sub>i</sub>、b<sub>u</sub>、q<sub>i</sub>、p<sub>u</sub></i> 可以通过对下面这个损失函数分别求偏导，不断逼近到误差和的最小值，从而得到最优的参数组合：</p>
<p data-nodeid="56265"><img alt="Drawing 13.png" src="https://s0.lgstatic.com/i/image/M00/41/3E/Ciqc1F802huAUl52AAAyq0PrPE4761.png" data-nodeid="56750"></p>
<p data-nodeid="56266">损失函数的第一部分<br>
<img alt="Drawing 14.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802iaAOE6uAAATGJeyD2Q891.png" data-nodeid="56755"><br>
表示误差和；</p>
<p data-nodeid="56267">第二部分<br>
<img alt="Drawing 15.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802jSAaDkNAAAPRLowHRo811.png" data-nodeid="56762"><br>
表示正则化项。</p>
<p data-nodeid="56268">这种方式把协同过滤转化为一个有监督学习的问题，被称为基于模型的协同过滤。用户矩阵 <em data-nodeid="56774">U</em> 描述的是每个用户对于每个特征的偏好，我们可以理解为品味，这是完全主观的。而特征矩阵 <em data-nodeid="56775">V</em> 对商品来说，可以理解为电影的风格，在这个场景中是客观的，这种方式有点类似于对品味和风格进行建模。得到了每个用户的品味和每个电影的风格后，我们就能按照下面的公式计算出所有用户对于所有商品的评分（补全数据集中缺失的部分），从而进行推荐：</p>
<p data-nodeid="56269"><img alt="Drawing 16.png" src="https://s0.lgstatic.com/i/image/M00/41/4A/CgqCHl802j2ATBnQAAAGBAMv9KA236.png" data-nodeid="56778"></p>
<p data-nodeid="85602">Spark MLlib 中采用了基于模型的协同过滤。Spark MLlib 将传统协同过滤中用户商品矩阵中的值称为显式反馈，也就是用户对商品的明确评分，例如电影评分，而那些商品特征信息，如购买、转发、评论等称为隐式反馈。Spark MLlib 的矩阵分解模型是奇异值分解（SVD），采用交替最小二乘法（ALS）最小化误差。Spark 也提供了 SVD 的改良算法：SVD++算法的  GraphX 版本实现，SVD++ 在模型中加入了隐式反馈的影响。</p>


<h3 data-nodeid="56271">Movielens 电影推荐系统</h3>
<p data-nodeid="86802">本课时将利用 Spark MLlib 实现一个推荐系统，如以下代码所示，使用的数据为 Movielens。Movielens 是一个关于电影评分的数据集，里面包含了多个用户对多部电影的评级数据，也包括电影元数据信息和用户属性信息。</p>

<pre class="lang-scala" data-nodeid="56273"><code data-language="scala"><span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.<span class="hljs-type">Pipeline</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.evaluation.<span class="hljs-type">RegressionEvaluator</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.ml.recommendation.<span class="hljs-type">ALS</span> 
<span class="hljs-keyword">import</span>&nbsp;org.apache.spark.sql.{<span class="hljs-type">Row</span>,&nbsp;<span class="hljs-type">SparkSession</span>} 

<span class="hljs-class"><span class="hljs-keyword">object</span><span class="hljs-title">&nbsp;CollaborativeFilteringMovieLens&nbsp;</span></span>{ 

<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span><span class="hljs-title">&nbsp;Rating</span>(<span class="hljs-params">userId:&nbsp;<span class="hljs-type">Int</span>,&nbsp;movieId:&nbsp;<span class="hljs-type">Int</span>,&nbsp;rating:&nbsp;<span class="hljs-type">Float</span>,&nbsp;timestamp:&nbsp;<span class="hljs-type">Long</span></span>) </span>

&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;main</span></span>(args:&nbsp;<span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]):&nbsp;<span class="hljs-type">Unit</span>&nbsp;=&nbsp;{ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;spark&nbsp;=&nbsp;<span class="hljs-type">SparkSession</span> 
&nbsp;&nbsp;&nbsp;&nbsp;.builder() 
&nbsp;&nbsp;&nbsp;&nbsp;.master(<span class="hljs-string">"local[2]"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.appName(<span class="hljs-string">"CollaborativeFilteringMovieLens"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.enableHiveSupport() 
&nbsp;&nbsp;&nbsp;&nbsp;.getOrCreate() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">import</span>&nbsp;spark.implicits._ 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">def</span><span class="hljs-title">&nbsp;parseRating</span></span>(str:&nbsp;<span class="hljs-type">String</span>):&nbsp;<span class="hljs-type">Rating</span>&nbsp;=&nbsp;{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;fields&nbsp;=&nbsp;str.split(<span class="hljs-string">"::"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;assert(fields.size&nbsp;==&nbsp;<span class="hljs-number">4</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-type">Rating</span>(fields(<span class="hljs-number">0</span>).toInt,&nbsp;fields(<span class="hljs-number">1</span>).toInt,&nbsp;fields(<span class="hljs-number">2</span>).toFloat,&nbsp;fields(<span class="hljs-number">3</span>).toLong) 
&nbsp;&nbsp;&nbsp;&nbsp;} 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;ratings&nbsp;=&nbsp;spark.read.textFile(<span class="hljs-string">"data/mllib/rating"</span>).map(parseRating).toDF() 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;<span class="hljs-type">Array</span>(training,&nbsp;test)&nbsp;=&nbsp;ratings.randomSplit(<span class="hljs-type">Array</span>(<span class="hljs-number">0.8</span>,&nbsp;<span class="hljs-number">0.2</span>)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置ALS算法参数 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;als&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">ALS</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setMaxIter(<span class="hljs-number">5</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setRegParam(<span class="hljs-number">0.01</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setUserCol(<span class="hljs-string">"userId"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setItemCol(<span class="hljs-string">"movieId"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setRatingCol(<span class="hljs-string">"rating"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setColdStartStrategy(<span class="hljs-string">"drop"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;pipeline&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">Pipeline</span>().setStages(<span class="hljs-type">Array</span>(als)) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;model&nbsp;=&nbsp;pipeline.fit(training) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;predictions&nbsp;=&nbsp;model.transform(test) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;设置模型评价指标 </span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;evaluator&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;<span class="hljs-type">RegressionEvaluator</span>() 
&nbsp;&nbsp;&nbsp;&nbsp;.setMetricName(<span class="hljs-string">"rmse"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setLabelCol(<span class="hljs-string">"rating"</span>) 
&nbsp;&nbsp;&nbsp;&nbsp;.setPredictionCol(<span class="hljs-string">"prediction"</span>) 

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;rmse&nbsp;=&nbsp;evaluator.evaluate(predictions) 

&nbsp;&nbsp;&nbsp;&nbsp;println(<span class="hljs-string">s"Root-mean-square&nbsp;error&nbsp;=&nbsp;<span class="hljs-subst">$rmse</span>"</span>) 

&nbsp;&nbsp;} 
}
</code></pre>
<h3 data-nodeid="56274">总结</h3>
<p data-nodeid="56275">本课时从原理出发，以协同过滤算法的进化为主线学习了推荐引擎。推荐引擎也可以算是最早以及最成功的大数据应用之一。</p>
<p data-nodeid="56276">最后给大家留一个<strong data-nodeid="56789">思考题</strong>：推荐引擎如何冷启动？</p>

---

### 精选评论


