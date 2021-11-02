<p data-nodeid="87842" class="">在前面 44 课时的内容中，我们学习了 Spark 的计算模型、编程接口和架构。在本课程的最后，我将从这几个方面对本课程进行总结并展望，帮你巩固之前学的内容，如果能引起你的深入思考就更好了。</p>
<p data-nodeid="87843">本课时的主要内容为：</p>
<ul data-nodeid="87844">
<li data-nodeid="87845">
<p data-nodeid="87846">统一的计算模型</p>
</li>
<li data-nodeid="87847">
<p data-nodeid="87848">统一的编程接口</p>
</li>
<li data-nodeid="87849">
<p data-nodeid="87850">统一的架构</p>
</li>
</ul>
<h3 data-nodeid="87851">统一的计算模型</h3>
<p data-nodeid="87852">就计算模型而言，“统一”有两层意思：第一层是计算模型统一了批处理与流处理的编程接口，第二层是统一了不同的计算引擎。第一个“统一”我们在“第 21 课时|统一批处理与流处理：Dataflow”中已经学到了，你可以通过对以下几个维度对数据流进行解构：</p>
<ul data-nodeid="87853">
<li data-nodeid="87854">
<p data-nodeid="87855">计算什么（What）。</p>
</li>
<li data-nodeid="87856">
<p data-nodeid="87857">根据事件时间，哪些数据会参与计算（Where）。</p>
</li>
<li data-nodeid="87858">
<p data-nodeid="87859">什么时候触发计算（When）。</p>
</li>
<li data-nodeid="87860">
<p data-nodeid="87861">早期的计算结果如何被修正（How）。</p>
</li>
</ul>
<p data-nodeid="87862">这样便可以统一地满足不同计算场景的需求。</p>
<p data-nodeid="87863">第二个“统一”也很好理解，目前主流的大数据处理引擎均选用 Dataflow 模型作为自己的设计蓝本，这造成的影响是，在未来的一段时间内，Spark 与 Flink 将会变得越来越像（即使目前两者还有差距）。<strong data-nodeid="87912">从某种层面上来说，Dataflow 是计算模型的计算模型。</strong></p>
<h3 data-nodeid="87864">统一的编程接口</h3>
<p data-nodeid="87865">虽然说 Flink 与 Spark 均采用了 Dataflow 思想，但两者的编程接口差异还是比较大。那有没有一种技术能够统一不同计算引擎的编程接口呢？答案是肯定的。</p>
<p data-nodeid="88161">Google 在 2016 年宣布其内部项目 Beam 进入 Apache 孵化器。2017 年 1 月，Apache 基金会宣布 Beam 成功毕业，成为 Apache 顶级项目。Apache Beam 的理论基础脱胎于 Google Dataflow 的论文<a href="https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf" data-nodeid="88166">（</a><a href="https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf" data-nodeid="88169">The Dataflow Model: A Practical Approach to Balancing Correctness, Latency, and Cost in Massive-Scale, Unbounded, Out-of-Order Data Processing</a><a href="https://www.vldb.org/pvldb/vol8/p1792-Akidau.pdf" data-nodeid="88172">）</a>，但不一样的是，Google Beam 对用户暴露 Dataflow 模型（Beam 模型）的编程接口，并兼容多种语言（Java、Python、Go 等），其下层可以适配多个计算引擎。也就是说，Beam 是一套统一的编程模型，<strong data-nodeid="88178">它可以在任何引擎上运行批处理和流处理作业</strong>，对应的开源实现是一个 SDK（Software Development Kit，软件开发工具包），如下图所示。</p>
<p data-nodeid="88492"><img src="https://s0.lgstatic.com/i/image/M00/50/B2/Ciqc1F9jL9eABs9WAABSB58-zS8281.png" alt="Lark20200917-174159.png" data-nodeid="88495"></p>





<p data-nodeid="87868">目前，Beam 支持的计算引擎有 Apex、Flink、Spark、Google Dataflow（这里的 Google Dataflow 指的是 Google 自己实现 Dataflow 模型的计算引擎，可以在 Google Cloud 使用）、Gearpump 和 samza 等，如下图所示。</p>
<p data-nodeid="87869"><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gb3OAKxf0AAA2eQRmQQg503.png" alt="Drawing 1.png" data-nodeid="87937"></p>
<p data-nodeid="87870">虽然 Dataflow 提出了一种统一流处理和批处理的编程接口，但是实现它的计算引擎却各有不同，比如 Spark 或 Flink，这也使得如果要切换计算框架，全部代码都必须重写。而 Beam 统一了大数据处理的计算框架的编程接口，<strong data-nodeid="87942">从某种层面上来说，Beam API 是编程接口的编程接口。</strong></p>
<p data-nodeid="87871">Google 开源 Beam 对用户和 Beam 本身来说都是好事，用户的代码更简洁、更具移植性，Beam 社区也会越来越活跃。但 Google 并不是慈善组织，对于 Beam 来说，在功能支持方面，最好的当然是 Google 自己的 Dataflow 计算引擎，随着用户代码越来越具备可移植性，用户也越来越容易且更愿意将数据处理应用部署到 Google Cloud 上，这也是 Google 开源 Beam 的一个目的：希望用 Beam 为 Google Cloud 抢占更多的云计算市场份额。</p>
<h3 data-nodeid="87872">统一的架构</h3>
<p data-nodeid="87873">在架构设计中，我们通常采用分层设计的方式进行规划，一般将大数据平台分为三层：计算层，资源管理与调度层，以及存储层，如下图所示。</p>
<p data-nodeid="88949">在我们熟悉的 Hadoop 中，这三层通常为：</p>
<p data-nodeid="89265"><img src="https://s0.lgstatic.com/i/image/M00/50/BD/CgqCHl9jL-qAHEh7AAA7B7KqYWU300.png" alt="Lark20200917-174208.png" data-nodeid="89268"></p>





<p data-nodeid="87876">结合前面的内容，可以发现 Beam 统一了计算层，对于用户来说，无论是什么计算引擎，只要使用 Beam API，就没什么不同。<strong data-nodeid="87954">从某种层面上来说，Beam API 是大数据编程接口的编程接口。</strong></p>
<p data-nodeid="87877">而资源管理与调度系统就比较有趣了。YARN 虽然比较常用，但它有个缺点，目前只能调度大数据计算作业，不能调度 Web 服务和数据库服务之类的计算需求。也就是说，在资源层面，YARN 并没有做到统一，而目前最火的资源管理与调度系统 Kubernets（简称 K8s）则能很好地调度这些资源。K8s 是 Google 10 多年大规模容器管理技术 Borg 的开源版本。在 Spark 2.4.4 中，Spark 添加了 Spark on Kubernets 的支持，如下图：</p>
<p data-nodeid="87878"><img src="https://s0.lgstatic.com/i/image/M00/4F/6B/Ciqc1F9gb4KAW7eHAABZ9dYwV58197.png" alt="Drawing 3.png" data-nodeid="87958"></p>
<p data-nodeid="87879">这样一来，在资源管理与调度层面，理论上，我们就可以通过 K8s 调度任意类型的计算作业。注意，我在这里强调了“理论上”，也就是说在实际中并不一定有必要这样做。另外，就算 Spark 不支持 Spark on K8s，我们也可以在 K8s 中部署 YARN 来达到目的。<strong data-nodeid="87963">从某种层面上来说，K8s 可以看成是资源管理与调度系统的资源管理与调度系统。</strong></p>
<p data-nodeid="87880">最后再来看看存储层。存储层通常为 HDFS，这也很好理解，HDFS 是 Hadoop 的标配。试想一个情况，如果在数据中心里有两个 Hadoop 集群，还有一些其他文件系统，如 Ceph 等，那么有没有办法在文件系统，也就是存储层进行统一呢？答案是有的，我们先来看一种新的技术——Alluxio。</p>
<p data-nodeid="87881">Alluxio 的前身是 Tachyon，同样由伯克利 AMP 实验室开源，是一个提供内存级别读写速度的虚拟分布式存储，是 BDAS 的重要组成部分。它已经在业界大规模使用，如 Palantir、巴克莱银行、百度、阿里云、携程等。组成 Alluxio 的文件系统有三层，如下图所示。</p>
<p data-nodeid="90199"><img src="https://s0.lgstatic.com/i/image/M00/50/B2/Ciqc1F9jL_aAf0I8AABHqStNd4E590.png" alt="Lark20200917-174228.png" data-nodeid="90202"></p>

<p data-nodeid="89884">最下面一层是底层文件系统（UFS），中间层是由从节点本地的存储资源组成的分层缓存层（Tired Cache），最上面是 Alluxio 角色节点（Alluxio Cluster）。对于一个 Alluxio 来说，底层文件系统可以是 HDFS、Amazon S3、Ceph 等，而分层缓存层则是作为 Alluxio 这个文件系统的高速副本缓存存在，用户可以按照访问速度依次设置为内存、固态硬盘和机械硬盘。Alluxio 客户端请求 UFS 中的某个目录的文件时，发现这份数据在分层缓存层中有副本，则直接从缓存将数据返回。所以，从这个角度上来说，Alluxio 没有自己的文件系统，它只是“借用”其他文件系统并进行统一封装，这也是其定义中“虚拟”的意义所在。除此之外，“虚拟”还指的是 Alluxio 最重要的一个特性：统一命名空间。</p>



<p data-nodeid="91272">Alluxio 统一命名空间主要是由 Alluxio 提供的挂载 API 实现的，有了命名空间，才能通过 Alluxio 真正地访问多种数据源。默认情况下，Alluxio 的命名空间是挂载到由 alluxio.underfs.address 配置指定的目录下，该目录将作为 Alluxio 的主存储。此外，用户可以使用挂载 API 将其他存储系统的路径挂载到 Alluxio 的目录下，如下图所示。</p>
<p data-nodeid="91273"><img src="https://s0.lgstatic.com/i/image/M00/50/BD/CgqCHl9jMAGAQZtKAAFRgsWcjEI437.png" alt="Lark20200917-174231.png" data-nodeid="91277"></p>




<p data-nodeid="90502">如此，Alluxio 就融合了不同介质的数据源，并提供了统一访问的 URL：alluxio://host:port/…。从这一个角度上说，Alluxio 是一个整合了多种不同存储介质的分层存储，我们可以将数据按照用途存到 Alluxio 中的不同目录（介质），以达到读写的最佳效率。</p>


<p data-nodeid="87887">看到这里，你应该能够想到，通过 Alluxio 的挂载功能，你可以将任意的文件系统挂载到 Alluxio 的底层文件系统下，<strong data-nodeid="87980">从某种层面上来说，Alluxio 是文件系统的文件系统</strong>。</p>
<h3 data-nodeid="87888">总结</h3>
<p data-nodeid="87889">在了解了 Dataflow、Beam、K8s 以及 Alluxio 这些技术后不难发现，统一的主旋律贯穿始终：Dataflow 是对批处理和流处理的统一，Apache Beam 是对计算框架（大数据编程）的统一，K8s 是对任意计算类型资源的统一，Allxuio 是对任意文件系统的统一。我们可以进一步想象，对于整个数据中心来说，是不是可以用 K8s 和 Alluxio 作为一个统一的大数据操作系统对整个数据中心进行抽象，这样一来，整个数据中心不就可以看成是一台普通服务器了吗？</p>
<p data-nodeid="87890">借用《三个火枪手》中的一句话作为本课时与整个 Spark 课程的结束：</p>
<p data-nodeid="87891">“all for one，one for all。”</p>
<p data-nodeid="87892">人人为我，我为人人。</p>
<p data-nodeid="87893" class="">相信本课程一定能帮助你在 Spark 领域有所突破。最后，我还是邀请你参与对本专栏的评价，你的每一个观点对我们来说都是最重要的。<a href="https://wj.qq.com/s2/7164306/3a46/" data-nodeid="87989">点击链接，即可参与评价</a>，还有机会获得惊喜奖品！</p>

---

### 精选评论


