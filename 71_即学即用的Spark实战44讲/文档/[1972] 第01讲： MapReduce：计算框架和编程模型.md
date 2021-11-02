<p style="line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">你好，我是你的 Spark 老师范东来，今天是本课程基础模块的第一节课，我们来聊聊一个比较基础也比较重要的内容 MapReduce，说它基础，是因为它诞生的时间实在是太久远了，并不是什么新东西，说它重要则是因为基于它的提出衍生出很多重要的技术，比如我们关心的 Spark。</span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">今天的内容主要有以下几点：</span></p>
<ol>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Google 的三驾马车；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 编程模型与 MapReduce 计算框架；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">并发与并行；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如何理解分布式计算框架的编程接口与背后的工程实现。</span></p></li>
</ol>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Google 的三驾马车</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">USNew 把计算机科学分为 4 个领域：人工智能、编程语言、系统以及理论。其中的系统领域有两大顶级会议，一个是 ODSI（USENIX conference on Operating Systems Design and Implementation），另一个是 SOSP（ACM Symposium on Operating Systems Principles），这两个会议在业界的分量非常重，如果把近几十年关于这两个会议的重要论文收录到一本书，就可以看作是操作系统和分布式系统的一本教科书。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从 2003 年到 2006 年，Google 分别在 ODSI 与 SOSP 发表了 3 篇论文，引起了业界对于分布式系统的广泛讨论，这三篇论文分别是：</span></p>
<ul>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">SOSP2003<span style="color: rgb(43, 43, 43); font-size: 12pt; font-family: &quot;Microsoft YaHei&quot;, sans-serif;">：</span><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">The Google File System；</span></span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">ODSI2004：<span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">MapReduce: Simplifed Data Processing on Large Clusters；</span></span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">ODSI2006：<span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">Bigtable: A Distributed Storage System for Structured Data。</span></span></p></li>
</ul>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 2006 年，Google 首席执行官施密特提出了云计算这个词语，Google 的这 3 篇论文也被称为 Google 的三驾马车，代表 Google 大数据处理的基石、云计算的基础。不过值得注意的是，虽然 Google 作为业界领军者经常会将自己的技术开源出来，但是客观地讲，Google 开源出来的技术并不是内部使用的最新技术，中间甚至会有代差，这也侧面反映出 Google 的技术实力。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第 1 篇论文主要讨论分布式文件系统，第 2 篇论文主要讨论的分布式计算框架，第 3 篇论文则主要讨论分布式数据存储。这 3 篇论文揭开了分布式系统神秘的面纱，为大数据处理技术做出了重要的贡献。 有了这 3 篇论文的理论基础与后续的一系列文章，再加上开源社区强大的实践能力，Hadoop、HBase、Spark 等很快走上了台前，大数据技术开始呈现出一个百花齐放的状态。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 编程模型与 MapReduce 计算框架</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在发表的第 2 篇文章中，Google 很明确地表示 MapReduce 是其实现的一个分布式计算框架，其编程模型名为 MapReduce。开源社区基于这篇论文的内容，照猫画虎地实现了一个分布式计算框架，也叫作 MapReduce。但一些书籍和网上的资料在提到 MapReduce 的时候并未说明，容易造成困惑。其实 Google 拿编程模型的名字直接作为计算框架的名字这种例子还有很多，比如 Google Dataflow。而 MapReduce 有两个含义，一般来说，在说到计算框架时，我们指的是开源社区的 MapReduce 计算框架，但随着新一代计算框架如 Spark、Flink 的崛起，开源社区的 MapReduce 计算框架在生产环境中使用得越来越少，逐渐退出舞台。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 的第二个含义是一种编程模型，这种编程模型来源于古老的函数式编程思想，在 Lisp 等比较老的语言中也有相应的实现，并随着计算机 CPU 单核性能以及核心数量的飞速提升在分布式计算中焕发出新的生机。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 模型将数据处理方式抽象为 map 和 reduce，其中 map 也叫映射，顾名思义，它表现的是数据的一对一映射，通常完成数据转换的工作，如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/85/BF/Cgq2xl6OuuWAMmCdAABmTwpHHM4748.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">reduce 被称为归约，它表示另外一种映射方式，通常完成聚合的工作，如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/85/BF/Cgq2xl6OuuyASdPvAAC2zb5yaHo183.png"></span></p>
<p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">圆角框可以看成是一个集合，里面的方框可以看成某条要处理的数据，箭头表示映射的方式和要执行的自定义函数，运用 MapReduce 编程思想，我们可以实现以下内容：</span></p>
<ol>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">将数据集（输入数据）抽象成集合；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">将数据处理过程用 map 与 reduce 进行表示；</span></p></li>
 <li><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在自定义函数中实现自己的逻辑。</span></p></li>
</ol>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">这样就可以实现从输入数据到结果数据的处理流程（映射）了。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">并发与并行</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">一般来说，底层的东西越简单，那么上层的东西变化就越复杂，对于 MapReduce 编程模型来说，map 与 reduce 的组合加上用户定义函数，对于业务的表现力是非常强的。这里举一个分组聚合的例子，如下图所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/0C/A9/Ciqah16OuviABEodAADcW42Rou4249.png"></span></p>
<p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">map 端的用户自定义函数与 map 算子对原始数据人名进行了转换，生成了组标签：性别，reduce 端的自定义函数与 reduce 算子对数据按照标签进行了聚合（汇总）。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 认为，再复杂的数据处理流程也无非是这两种映射方式的组合，例如 map + map + reduce，或者 reduce 后面接 map，等等，在我展示出的这张图里你可以看到相对复杂的一种组合形式：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/85/BF/Cgq2xl6Ouv6ATQMfAAC8oOgBQqk428.png"></span></p>
<p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">很多支持函数式编程的语言，对于语言本身自带的集合数据结构，都会提供 map、reduce 算子。现在，我们可以很容易的将第一个圆角方框想象成一个数十条数据的集合，它是内存中的集合变量，那么要实现上图中的变换，对于计算机来说，难度并不大，就算数据量再大些，我们也可以考虑将不同方框和计算流程交给同一台计算机的 CPU 不同的核心进行计算，这就是我们说的并行和并发。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如何理解分布式计算框架的编程接口与背后的工程实现</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">现在你可以想象下，随着数据集继续增大，要处理的数据（上图中开始的集合）超过了计算内存的大小，那么就算是逻辑非常简单的流程，也要考虑中间结果的存储。比如计算过程涉及到硬盘和内存之前的数据交换等等之类的工程实现的问题，虽然在这个过程中上面 3 步并没有发生变化，但是背后实现的系统复杂度大大提高了。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们可以再发挥想象，将上图中的圆角框想象成一个极其巨大的数据集，而方框想象成大数据集的一部分，我们会发现，对于从输入数据到结果数据的映射需求来说，前面 3 步仍然适用，只是这个集合变得非常大。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">但是由于数据量的急剧扩大，相比于刚才的第 2 种情况，背后工程实现的复杂度会成倍增加，当整个数据集的容量和计算量达到 1 台计算机能处理的极限的时候，我们就会想办法把图中方框所代表的数据集分别交给不同的计算机来完成，那么如何调度计算机，如何实现 reduce 过程中不同计算机之间的数据传输等问题，就是 Spark 基于 MapReduce 编程模型的分布式实现，这也是我们常常所说的分布式计算。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从上图可以看出，在 reduce 过程中，会涉及到数据在不同计算机之间进行传输，这也是 MapReduce 模型下的分布式实现的一个关键点，后面我们会讲到 Spark 是如何做的。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">看到这里，你可能对分布式运算有一个感性的认识，以小见大，函数式语言本身就提供了类似于 map、reduce 的操作，如下图第 1、2 行代码：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/85/B2/Cgq2xl6Or3WAAtCtAAARYYA8XK4196.png"></span></p>
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="text-align:center;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/0C/9C/Ciqah16Or3WAXcJqAAAY1xzPv8Y404.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">1、2 行是函数式编程语言 Scala 对于集合的处理，3、4 行是 Spark 对集合的处理，逻辑同样是对集合元素都加 1 再过滤掉小于等于 1 的元素并求和。对于 Spark 来说，处理几十 GB到几十 TB 的数据集，第2行代码或者说第4行代码同样适用，只是 list 变得比较特殊，它不是只存在于一台计算机的内存里，而是存在于多台计算机的磁盘和内存上。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">现在，我们可以这样理解基于 MapReduce 编程模型的分布式计算框架，其编程接口与普通函数式语言的数据处理并没有什么不同（甚至可以说完全一样），但是背后的工程实现千差万别，而像 Spark、MapReduce 这样的框架，它们的目标都是尽力为用户提供尽可能简单的编程接口以及高效地工程实践。从这个角度上来讲，我们可以把 Spark 看成是一种分布式计算编程语言，它的终极目标是希望达到这样一种体验：让用户处理海量数据集时与处理内存中的集合变量并没有什么不同。</span></p>
<p style="text-indent: 29.3333px;line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">MapReduce 这种思想或者编程模型已经出现几十年了，不变的是思想，变得是使用场景和实现方法。我相信未来一定会有效率优于 Spark 的计算框架出现，就像 Spark 优于普通的编程语言一样。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总结</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时的主要目的是在深入讲解 Spark 之前，对 Spark 之前的技术、范式、抽象进行一个简单的讲解，为后面的学习打下基础。</span></p>

---

### 精选评论

##### Cobra：
> 1元也这么香，内容质量真心不错😍

##### **超：
> 老师能不能把一周两更，改为四更、五更、或者六更、七更，迫不及待呀😘

##### *健：
> 求更新的快一点，我这条咸鱼求知若渴

##### **天：
> 着急想看下一讲，付全款可以全看嘛😂😂😂

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 同学，周二周四更哦~

##### *凯：
> 老师加油，spark得学习之路就靠老师来指点了

##### *尖：
> map还是可以起到filter和flatmap的作用的。

##### **松：
> 让用户处理海量数据集时与处理内存中的集合变量并没有什么不同。这句话境界真高

##### **4566：
> 希望更新快着点

##### **天空：
> map负责数据转换，reduce负责数据聚合

##### zwei：
> 期待更新

##### **路：
> 1元也是很香的嘛，哈哈哈，期待ing

##### **亮：
> 每篇文章都要精读好几遍，每一遍都有不同的收获，每一句都很精炼

##### **里的火：
> 不错，对原理讲解的很好

##### **诚：
> 求更新！！！求更新！！！求更新！！！

##### **生：
> 相当不错，按时听课，希望学完能摸到门。

##### **1421：
> 讲解简单明了，很不错呢😀

##### *稳：
> 老师讲的很好🍻

##### **3698：
> 期待更新中

##### **2709：
> 很清晰，质量不错

##### **3215：
> 写的真好，期待更新😀

##### **2572：
> 讲的真心很好！谢谢！

##### *强：
> 期待更新😀

##### **生：
> 等老师更新吧 我对知识付费这事 其实还是蛮认可

##### **忠：
> 确实不错

##### **谱：
> 小白学习，非常喜欢大数据

##### **强：
> 香

