<p data-nodeid="149767" class="">上一讲我们一同了解了图像算法中常用的卷积神经网络（CNN）。CNN 是图像算法方面中最基础和核心的环节，通过上一讲的学习，你可以知晓算法是如何感知图像的内容的。这节课我们来学习循环神经网络和长短期记忆网络。</p>




<h3 data-nodeid="147726">循环神经网络</h3>
<p data-nodeid="147727">我们先来了解循环神经网络（Recurrent Neural Network，以下简称 RNN）。</p>
<p data-nodeid="147728">回忆一下咱们之前学到的 CNN 和全连接，不难发现模型的输入数据之间是没有关联的，比如上节课提到的图像分类，每次输入的图片与图片之间就没有任何关系，上一张图片的内容不会影响到下一张图片的结果。但在自然语言处理领域，这似乎就成了一个短板。</p>
<p data-nodeid="147729">我举一个例子：我是一名中国人，一年中最重要的节日是_______。</p>
<p data-nodeid="147730">我们需要在空缺的部分填上一个词，这个词最有可能的是春节，而不是圣诞节（因为我是中国人），也不是中秋节（不是最重要的节日），更不可能是火车（根本就不搭边）。这个类似完形填空的问题就是一个典型的语言模型问题，即给定一串内容或者问题，预测接下来可能的内容或者结果，但我之前介绍的模型对于这样的一个序列问题似乎不是很好解决。</p>
<p data-nodeid="147731">之前人们比较常用的办法就是首先将文本进行分词，也就是划分成一个一个的词语序列，即：我｜是｜一名｜中国人｜ ，｜一年｜中｜最｜重要｜的｜节日｜是。</p>
<p data-nodeid="147732">人们通过大量的语料（也就是学习数据），将每个词语和其他词语共同出现的概率进行计算，当出现“是”词语的时候，将最有可能出现的词语作为最终的填充结果。</p>
<p data-nodeid="147733">不难想象，这样做的效果肯定不好。那怎么办呢？增加词的计算长度。比如“节日｜是”，甚至更长的“最｜重要｜的｜节日｜是”，效果就稍微好一点了。但是这又导致了另一个问题：如果句子的长度是任意的，那在句子比较长的情况下，计算量会急剧增大，比如为了限定节日为中国的，需要计算的长度就会很长，所以在真正使用的时候很不方便，准确性也不能保证。</p>
<p data-nodeid="147734">这个时候，RNN 就要上场了，它是<strong data-nodeid="147842">一类用于处理序列数据的神经网络</strong>。这里有 2 个关键点：</p>
<ol data-nodeid="147735">
<li data-nodeid="147736">
<p data-nodeid="147737">RNN 是一类网络；</p>
</li>
<li data-nodeid="147738">
<p data-nodeid="147739">处理序列数据。</p>
</li>
</ol>
<p data-nodeid="147740">什么是序列数据？以常见的时间序列数据（time series data）为例，它是在不同时间上收集到的数据，用于所描述现象随时间变化的情况。这类数据反映了某一事物、现象等随时间的变化状态或程度，比如股票走势、天气情况、化学反应情况等。</p>
<p data-nodeid="151387">我们来看 RNN 的结构。我们之前学习的网络结构，比较典型的是输入层—隐藏层—输出层，RNN 也差不多，但有少许的不同，如下图：</p>
<p data-nodeid="152207"><img src="https://s0.lgstatic.com/i/image/M00/6E/63/Ciqc1F-ySqeAe7L5AAB8Kd8pVco390.png" alt="Drawing 0.png" data-nodeid="152211"></p>
<div data-nodeid="152208" class=""><p style="text-align:center">图 1：RNN 结构图</p></div>





<p data-nodeid="147744">上图是一个抽象的 RNN 基本单元结构。自底向上的三个蓝色的节点很显然，分别是输入层、隐藏层和输出层。U 和 V 分别是连接两个层的权重矩阵。如果不考虑右边的棕色环路的话，就是一个典型的全连接的网络。</p>
<p data-nodeid="154631">那么这个环路是干吗的呢？我们不妨先将其展开：</p>
<p data-nodeid="155445" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySrWASJQVAAEaIAJEhFU880.png" alt="Drawing 1.png" data-nodeid="155449"></p>
<div data-nodeid="155446"><p style="text-align:center">图 2：RNN 内部结构展开图</p></div>





<p data-nodeid="156646">现在看上去是不是明晰多了？ 在 t 时刻，网络接受输入 X<sub>t</sub> 和来自 t-1 时刻的隐藏层状态 S<sub>t-1</sub>，并产生一个 t 时刻的隐藏层状态 S<sub>t</sub>，以及 t 时刻的输出 O<sub>t</sub>。其公式化的表示为：</p>
<p data-nodeid="156647" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/63/Ciqc1F-ySsKAPKZhAACEozEroIE529.png" alt="Drawing 3.png" data-nodeid="156667"></p>



<p data-nodeid="147751">其中 g 和 f 是各自节点的激活函数。这里面需要注意的一点是，对于每一个时间 t，U、V、W 都是同一个，这非常类似上一节课讲到的权值共享。RNN 的权值共享主要出于两方面的考虑：</p>
<ol data-nodeid="147752">
<li data-nodeid="147753">
<p data-nodeid="147754">减少参数量，也减少计算量。</p>
</li>
<li data-nodeid="147755">
<p data-nodeid="147756">RNN 接受的输入是可变长的，如果不进行权值共享，那每个 W 都不同，我们无法提前预知需要多少个 W，实现上的计算就会非常困难。</p>
</li>
</ol>
<p data-nodeid="147757">以上我说说到的是典型的 RNN 的结构，实际上 RNN 有很多种变体。</p>
<p data-nodeid="147758">我们还是看一个完形填空的问题：我的圆珠笔坏了，我想_____一支新的笔。</p>
<p data-nodeid="158262">你发现问题了吗？在这个例子中我们要填写的内容不只取决于前面的字词，还跟后面的内容相关。如果只看前面的，填写“吃饭”或者“放学”都是可以的，但从后面的内容来看，就限定为了“买”“换”等相关的词语。刚才介绍的 RNN 结构此时就不能解决问题了。于是便有了双向的 RNN（BiRNN）。其抽象表示如下：</p>
<p data-nodeid="159070"><img src="https://s0.lgstatic.com/i/image/M00/6E/63/Ciqc1F-ySsyAPBSNAAEUzZqTsmw815.png" alt="Drawing 4.png" data-nodeid="159074"></p>
<div data-nodeid="159071" class=""><p style="text-align:center">图 3：BiRNN 抽象表示图</p></div>





<p data-nodeid="160655">相比于 RNN，BiRNN 维持了两个方向的状态。正向计算和反向计算不共享权重，也就是说 U、V、W 分别有两个，以对应不同的方向。其公式化的表示就变成了如下的形式：</p>
<p data-nodeid="160656" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-yStWAVfhpAADYYz7YpXY006.png" alt="Drawing 6.png" data-nodeid="160660"></p>



<blockquote data-nodeid="147765">
<p data-nodeid="147766">需要注意：S'在 t 时间接受的隐藏层状态不是来自 t-1，而是来自 t+1。</p>
</blockquote>
<p data-nodeid="147767">在咱们介绍的 RNN 和 BiRNN 结构中，隐藏层只有一层，但在实际的使用中，也经常会增加隐藏层的数量，由此又会得到深度循环神经网络等一系列的变体，这些变体能够捕获和关联更多的前后信息以提升效果。</p>
<h3 data-nodeid="147768">RNN 的梯度消失与爆炸</h3>
<p data-nodeid="147769">相对于全连接的方式，RNN 能够更好地处理序列相关的问题，但正是因为 RNN 需要考虑的内容是变长的，所以就会带来梯度相关的问题。</p>
<p data-nodeid="162225">根据在《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=522#/detail/pc?id=4979" data-nodeid="162230">05 | 前馈网络与反向传播：模型的自我学习（下）</a>》中介绍的方式，尝试求 RNN 的梯度。我们首先明确函数关系，如下所示：</p>
<p data-nodeid="162226" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySt-AKb4BAACTZzPSMEs415.png" alt="Drawing 8.png" data-nodeid="162234"></p>



<p data-nodeid="163791">求梯度实际上是求 W、V、U 的偏导数。我们以 L 对 W 在 t 时刻求偏导数为例，推导过程如下：</p>
<p data-nodeid="163792" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/63/Ciqc1F-ySuWAHM_HAADP2k2SS70385.png" alt="Drawing 9.png" data-nodeid="163796"></p>


<p data-nodeid="147775">可以发现，L 关于 W 的偏导数会随着序列的长度而产生长期依赖。</p>
<blockquote data-nodeid="147776">
<p data-nodeid="147777">长期依赖是指当前系统的状态，可能受很长时间之前系统状态的影响，是RNN中无法解决的一个问题。</p>
</blockquote>
<p data-nodeid="147778">此外，别忘了还有激活函数 f 的存在。RNN 一般会使用 tanh 函数作为它的激活函数，而 tanh 的导数在 0~1 之间。如此一来，如果 W 也是在 0~1 之间，随着 t 的增大，梯度计算中连续相乘就会变得很长，很多个在 0~1 之间的数相乘会逐渐接近 0。梯度接近 0 则意味着梯度消失了；反之如果 W 很大，则梯度也会变得非常大，进而产生梯度爆炸，这是一个很严重的问题。</p>
<p data-nodeid="147779">那我们怎么解决这个问题呢？</p>
<p data-nodeid="147780">从上面的表述来看，问题出现在连续相乘的环节。那我们是不是可以把这个环节优化一下，不要让梯度消失或者爆炸就好了？这就是长短期记忆网络要做的事情。</p>
<h3 data-nodeid="147781">长短期记忆网络</h3>
<p data-nodeid="147782">我们接下来看长短期记忆网络（Long Short-Term Memory，以下简称 LSTM）。</p>
<p data-nodeid="147783">刚才提到 RNN 的梯度问题，其本质原因就是模型“记忆”的序列太长了，不管真实序列有多长都会一股脑地记忆和学习，从合理性的角度来看这并不是一个很好的方案。</p>
<p data-nodeid="147784">如果我们能让 RNN 在接受上一时刻的状态和当前时刻的输入时，有选择地记忆和删除一部分内容（或者说信息），问题就可以解决了，比如有一句话提及刚才吃了苹果，那么在此之前说的吃香蕉的内容就没那么重要，删除就好了。</p>
<p data-nodeid="165353">在各种博客和技术文档中，都有很多种 LSTM 的表现形式，每一种都有其特点。为了便于理解，我按照下图的形式绘制了 LSTM 的结构图：</p>
<p data-nodeid="165354" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/63/Ciqc1F-ySu6ADJ-7AAFAFHa4bpU298.png" alt="Drawing 10.png" data-nodeid="165358"></p>


<blockquote data-nodeid="147787">
<p data-nodeid="147788">RNN 只维持 1 个传递状态，LSTM 则需要维持 2 个传输状态。C<sub>t-1</sub>表示上一时刻的细胞状态（cell state），h<sub>t-1</sub>则表示上一时刻的隐藏状态（hidden state）。</p>
</blockquote>
<p data-nodeid="147789">LSTM 独特的地方在于它内部使用了 3 个逻辑门来控制细胞的状态，分别是<strong data-nodeid="147977">遗忘门</strong>、<strong data-nodeid="147978">输入门</strong>和<strong data-nodeid="147979">输出门</strong>，并对应了<strong data-nodeid="147980">忘记</strong>、<strong data-nodeid="147981">选择</strong>、<strong data-nodeid="147982">更新</strong>、<strong data-nodeid="147983">输出</strong>这 4 个不同的阶段，从而有选择性地保留或删除信息。我们来具体看一下。</p>
<ul data-nodeid="166905">
<li data-nodeid="166906">
<p data-nodeid="166907"><strong data-nodeid="166929">忘记阶段</strong>：刚才说过，对于上一时刻的状态我们如果能够选择性地记忆就好了。LSTM 中就使用了 Z<sub>f</sub>这个逻辑门来实现相应的功能，比如我们阅读一篇小说，我们会更倾向于忘记景色描写而不是人物对话，因为它并没有太多用途。这个逻辑门实际上是一个 Sigmoid 单元，我们称为<strong data-nodeid="166930">遗忘门</strong>。Sigmoid 可以将输入映射在 0～1 之间，得到的值再与 C<sub>t-1</sub>相乘，这样就实现了对上一时刻状态 C<sub>t-1</sub>的控制，即哪些信息保留或者删除多少。遗忘门的公式化表示为：</p>
</li>
</ul>
<p data-nodeid="166908" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySwiAOFQ2AAAxNHY_Ia4190.png" alt="Drawing 12.png" data-nodeid="166933"></p>



<ul data-nodeid="168464">
<li data-nodeid="168465">
<p data-nodeid="168466"><strong data-nodeid="168488">选择阶段</strong>：忘记阶段用来选择性保留或者删除上一时刻的内容，那当前时刻的输入呢？我们也需要类似的处理单元来进行选择，或者说决定给细胞状态添加哪些新的信息。这个阶段包括 2 个环节：首先是利用 h<sub>t-1</sub>和 x<sub>t</sub>通过 1 个 Sigmoid 单元决定更新哪些信息，然后利用 h<sub>t-1</sub>和 x<sub>t</sub>通过 1 个 tanh 层得到新的候选细胞信息，这些信息会根据计算的结果更新到细胞中。这个过程就是输入门，公式化表示为：</p>
</li>
</ul>
<p data-nodeid="168467" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySw-AE--yAABVEd4UPXQ146.png" alt="Drawing 14.png" data-nodeid="168491"></p>



<ul data-nodeid="170008">
<li data-nodeid="170009">
<p data-nodeid="170010"><strong data-nodeid="170016">更新阶段</strong>：经过选择阶段我们确定了当前时刻输入的信息哪些需要留下，接下来就要对细胞状态 C 进行更新了。这个环节实际上就是把前 2 个环节得到的结果与对应的信息相乘后再加起来，其公式化表示如下：</p>
</li>
</ul>
<p data-nodeid="170011" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySxaAOz5uAAAtBr8uRgk536.png" alt="Drawing 16.png" data-nodeid="170019"></p>



<p data-nodeid="179336" class="te-preview-highlight">我们可以到这个公式的平衡美：z<sub>f</sub> 和 z<sub>i</sub> 分别控制了上个阶段和当前阶段要保留多少内容，c<sub>t-1</sub> 和 z<sub>i</sub> 则是上个阶段和当前阶段的内容本身，所以是一个非常灵活的控制方式。</p>






<ul data-nodeid="175459">
<li data-nodeid="175460">
<p data-nodeid="175461" class=""><strong data-nodeid="175494">输出阶段</strong>：更新完细胞的状态，就到了最终的输出环节。这个过程中还是需要 h<sub>t-1</sub> 和 x<sub>t</sub> 的参与，这 2 个信息经过一个叫<strong data-nodeid="175495">输出门</strong>的 Sigmoid 逻辑单元后，与经过 tanh 后被缩放到-1～1 之间的细胞状态 C<sub>t</sub> 信息相乘，就得到了当前时刻的隐藏状态 h<sub>t</sub>。得到 h<sub>t</sub> 之后，就能得到当前时刻的输出 y 了。h<sub>t</sub> 的计算过程如下：</p>
</li>
</ul>





<p data-nodeid="171523" class=""><img src="https://s0.lgstatic.com/i/image/M00/6E/6E/CgqCHl-ySx6AT5LTAAAq615iunk235.png" alt="Drawing 18.png" data-nodeid="171560"></p>



<p data-nodeid="147811">LSTM 在过去的一段时间里都有着广泛的应用，比如音乐创作、股票价格预测等与时间序列相关的问题，并且在 NLP 问题上也表现良好。但正因为它侧重时间序列以及其本身的结构特点，LSTM 也有着非常明显的缺点，我列举了其中 3 点，如下：</p>
<ol data-nodeid="147812">
<li data-nodeid="147813">
<p data-nodeid="147814"><strong data-nodeid="148112">并行化困难</strong>。LSTM 的本质是一个递归的训练过程，随着实际问题的愈发复杂，这个缺点就会越来越致命。</p>
</li>
<li data-nodeid="147815">
<p data-nodeid="147816"><strong data-nodeid="148117">梯度消失</strong>。LSTM 虽然在一定程度上缓解了 RNN 的问题，但是对于长序列的情况，仍有可能会出现梯度消失。</p>
</li>
<li data-nodeid="147817">
<p data-nodeid="147818"><strong data-nodeid="148122">LSTM 在计算的时候需要的资源较多</strong>。</p>
</li>
</ol>
<p data-nodeid="147819">这些缺点意味着 LSTM 在未来的应用可能会逐步减少，同时，在不同的问题上已经出现了更好的解决方案，例如在 NLP 问题中被广泛采用的 Transformer 框架（我将在实战环节《<strong data-nodeid="148128">20 | 文本分类：技术背景与经典网络结构介绍（上）</strong>》中具体介绍）。</p>
<h3 data-nodeid="147820">总结</h3>
<p data-nodeid="147821">恭喜你完成了 RNN 及 LSTM 的学习，这一讲，我从概念上带你了解了它们的原理和特点。随着算法的发展，将会有越来越多的新的解决方案来不断地提升深度学习的表现。</p>
<p data-nodeid="147822">那么你对于 LSTM 的缺点还有什么看法呢？欢迎在留言区留言。</p>
<p data-nodeid="147823">刚才说到的 LSTM 在音乐创作其实上也有着一定的应用。在接下来的两节课中，我会通过自编码器和生成式对抗网络来带你了解深度学习在自我表达、艺术创作等领域上的一些基础知识与概念。</p>

---

### 精选评论

##### *炜：
> 请问一下，LSTM的Ot呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好好看看文章哟，RNN里面的叫O，lstm里面的叫C

##### **烁：
> 老师，z0，是什么意义？是不是承担着细胞状态和真实性的映射。细胞状态是不是类似于隐马尔可夫模型中的观测值。如果是，是不是就是一个标签的作用？lstm中，标签是什么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; z0有讲，好好看看。lstm我们一般使用它的隐藏状态或者最终状态，可能是一组向量，然后再接一个全联接做最终的分类。

##### **威：
> 想请教老师，在分类任务中，sigmoid和softmax起得作用都很明确，LSTM这里的应用不太明白

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 序列问题，lstm用得多，比如要用一段时间内的连续数据的情况。

##### **威：
> LSTM这块内容看得很迷糊，看不懂为什么sigmoid和tanh组合后，就能预测到真实值，以及为什么不是使用其它激活函数

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 激活函数的使用，不是必须用某一种的，而是说，在不同的场合和问题下，我们更倾向于时候某种激活函数，具体可以回到cnn的章节回顾一下。

