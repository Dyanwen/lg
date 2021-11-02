<p data-nodeid="4971" class="">前面我们已经知道 Hadoop 的两大核心：</p>
<ul data-nodeid="4972">
<li data-nodeid="4973">
<p data-nodeid="4974"><strong data-nodeid="5069">分布式文件存储系统</strong>，这个我们已经介绍过了；</p>
</li>
<li data-nodeid="4975">
<p data-nodeid="4976"><strong data-nodeid="5074">分布式计算框架</strong>，作为最早的分布式计算框架 MapReduce，就是我们这节课的主角。</p>
</li>
</ul>
<p data-nodeid="4977">回顾一下我在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=615#/detail/pc?id=6517" data-nodeid="5080">《05 | 大数据开发必备工具——Hadoop》</a>中介绍的关于 MapReduce 的例子，如果我们要进行<strong data-nodeid="5086">人口普查</strong>，一定不会说是从黑龙江数到海南，再从上海数到乌鲁木齐，把整个中国的人都数一遍，哪怕他是全世界数数最快的人，估计一辈子也数不完。</p>
<p data-nodeid="4978">实际上，我们是先按照地区进行划分，把省的任务划分成若干个市的任务，把市的任务划分成若干个区县的任务，甚至划分到每个小区、每栋楼，作为最小的单位。给每个单位安排一个负责人，每个负责人只需要负责自己片区的人数统计，然后再由负责人逐级向上汇总，把所有人的统计结果汇总起来，就能得到全国的人口普查数据。如果说其中一个负责人有事情，不能完成，就可以换一个人继续这个地区的统计，而不会明显地影响全国的人口普查进度，这样的任务，哪怕是小学刚毕业的人都可以很好地完成自己片区的统计，这种分而治之的思想也就是<strong data-nodeid="5092">MapReduce 思想的根源</strong>。</p>
<h3 data-nodeid="4979">MapReduce 的架构</h3>
<p data-nodeid="4980">如果理解了上面的小例子，你也就理解了 MapReduce 的基本理念。MapReduce 即是采用了这种<strong data-nodeid="5099">分而治之</strong>的处理数据模式，将要进行的数据处理任务分成 Map（映射）和 Reduce（化简）这两个处理过程。</p>
<p data-nodeid="4981">在 Map 操作中进行数据的读取和预处理，把国家划分成地区，然后一个负责人负责一个片区，这就是 Map。统计之后负责人一起汇总结果，这就是 Reduce。</p>
<p data-nodeid="4982">这样做最大的好处就是可以把<strong data-nodeid="5106">大规模数据分布到普通性能的服务器中进行预处理</strong>，然后将处理后的结果重新进行整合，从而得到需要的结果。当然，要让这么多机器共同完成一项任务，还有很多细节的问题需要处理。</p>
<p data-nodeid="4983">除了计算本身，MapReduce 还解决了协调这些集群中的服务器的问题，比如在若干台机器中执行运算的顺序、计算压力的分配、操作的原子性等等，如果让我们自己去编写一套程序那将是一个非常复杂的系统。但是 MapReduce 解决了这些问题，我们只需要关注如何把问题拆解成 Map 部分和 Reduce 部分就可以了。</p>
<p data-nodeid="4984">跟 HDFS 一样的，<strong data-nodeid="5113">MapReduce 也是采用主从结构的架构</strong>，架构图如下所示：</p>
<p data-nodeid="4985"><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB70aAc2EnAAIQeeCayAg590.png" alt="图片1.png" data-nodeid="5116"></p>
<h4 data-nodeid="4986">1.Job</h4>
<p data-nodeid="4987">Job 即是作业，一个 MapReduce 作业是用户提交的最小单位，比如说我们在下面即将动手运行的 WordCount 运算就可以称为一个作业。</p>
<h4 data-nodeid="4988">2.Client</h4>
<p data-nodeid="4989">Client 就是客户端，是用户访问 MapReduce 的接口，通过 Client 把编写好的 MapReduce 程序提交到 JobTracker 上。</p>
<h4 data-nodeid="4990">3.JobTracker</h4>
<p data-nodeid="4991">在 MapReduce 中，为了实现分布式的管理架构，使用了领导和随从的模式。而 JobTracker 就是这个体系中的领导。一个 MapReduce 集群只有一个JobTracker，这个节点负责下发作业，同时，它要收听来自很多 TaskTracker 的状态信息，从而决定如何分配工作。在这个集群中，JobTracker“既当爹又当妈”，而且只有一个，存在单点故障的可能，必须给它安排一个好点、稳定点的机器，不然如果它罢工了，将导致集群所有正在运行的任务全部失败。</p>
<h4 data-nodeid="4992">4.TaskTracker</h4>
<p data-nodeid="4993">TaskTracker 在集群中则扮演了随从的角色，主要负责执行 JopTracker 分配的任务，同时 TaskTracker 通过 Heartbeat（心跳信息）汇报当前的状态，JobTracker 根据这些状态再进行任务的分配。随从是具体负责工作的，人多力量大，一个集群自然可以有多个 TaskTracker。在一个实际的节点上，会有一个 TaskTracker ，还有我们在 HDFS 环节介绍的 DataNode，这样，一个节点既是计算部分又是存储部分，在进行运算时，可以优先进行本机数据本机计算，从而提升效率。</p>
<h4 data-nodeid="4994">5.Task</h4>
<p data-nodeid="4995">Task 称为任务，是在 MapReduce 实际计算时的最小单位，Task 有两种：MapTask 和ReduceTask，由 TaskTracker 进行启动。</p>
<h4 data-nodeid="4996">6.Split</h4>
<p data-nodeid="4997">除了在图中出现的概念，还有一个 Split，称为一个划分，这是 MapReduce 处理的元数据信息，Split 会记录实际要处理的数据所在的位置、大小等等。在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=615#/detail/pc?id=6519" data-nodeid="5133">《07 | 专为解决大数据存储问题而产生的 HDFS》</a>中，我们介绍了数据块的概念，Split 实际的数据就存储在 HDFS 的数据块中，通常我们会把 Split 的大小设置成与数据块大小一致，每次需要处理的数据存放在一个位置，这样可以减少不必要的网络传输，节约资源，提升计算速度。</p>
<p data-nodeid="4998">了解完这么多概念，我们再回顾一下整个处理流程：</p>
<ul data-nodeid="4999">
<li data-nodeid="5000">
<p data-nodeid="5001">我们编写一个 MapReduce 程序，这可以称为一个 Job，在这个程序中往往包含较多的 Map 操作，和较少的 Reduce 操作；</p>
</li>
<li data-nodeid="5002">
<p data-nodeid="5003">通过 Client 把 Job 提交到 JobTracker 上，JobTracker 会根据当前集群中 TaskTracker 的状态，分配这些操作到具体的 TaskTracker上；</p>
</li>
<li data-nodeid="5004">
<p data-nodeid="5005">TaskTracker 启动相应的 MapTask 或者 ReduceTask 来执行运算。</p>
</li>
</ul>
<p data-nodeid="5006">在具体的运算中，MapTask 读取 Split 的数据，根据事先写好的程序对其中的每一条数据执行运算，比如说后面的单词计数代码，我们输入的是若干文本文件，Map 操作处理每一行文本，并为一行中的单词进行计数，然后把结果保存下来。而在 ReduceTask 中，获取之前 Map 的预处理结果，并对数据进行分组，比如说相同单词分为一组，然后进行加和运算。对于 Reduce 的输出结果，会存储三个副本。</p>
<h3 data-nodeid="5007">MapReduce 的特点</h3>
<h4 data-nodeid="5008">1.简化了分布式程序的编写</h4>
<p data-nodeid="5009">就像我们前面介绍的，MapReduce 解决了很多分布式计算的底层问题，使得开发者不用关心复杂的调度和通信问题，只把精力放在如何实现 MapReduce 过程就可以了。</p>
<h4 data-nodeid="5010">2.可扩展性强</h4>
<p data-nodeid="5011">这是大数据处理框架的通用特点，只需要使用普通的机器就可以解决大规模数据处理的问题，当资源不足时只需要增加普通的机器节点就可以解决问题。</p>
<h4 data-nodeid="5012">3.容错性强</h4>
<p data-nodeid="5013">在集群中，避免不了各种节点的故障问题，但是 MapReduce 会自动地处理这些问题，屏蔽故障节点，使用正常的节点重新处理问题，这些都不需要使用者关心，直到任务处理完毕。</p>
<h3 data-nodeid="5014">MapReduce 的硬伤</h3>
<p data-nodeid="5015">虽然说 MapReduce 有非常多的好处，但是它的硬伤同样明显。</p>
<h4 data-nodeid="5016">1.学习成本高</h4>
<p data-nodeid="5017">虽然说 MapReduce 已经处理了大量的分布式问题，但是仅仅是统计一些数据也要完全理解 MapReduce 的思想，学习编写一个 MapReduce 程序，这对于很多人是非常不友好的。我们前面讲到的 Hive 就是为了减轻这种问题而产生的，但是 MapReduce 仍然是非常底层的技术，而且不是所有的算法都可以使用 MapReduce 来实现，很多功能无法使用这种并行的方式来解决。</p>
<h4 data-nodeid="5018">2.时间成本过高</h4>
<p data-nodeid="5019">MapReduce 是纯粹的<strong data-nodeid="5157">批处理模式</strong>，也就是说所有的数据都是事先已经存储好的，MapReduce 只是对这些数据进行批量的处理，这对于互联网中大量的流式数据无法给到很好的支持，如果我们想要处理今天的数据，要等到今天的数据都已经存储好再进行计算，而不能随来随算。同时， MapReduce 的所有中间结果都是存放在磁盘中的，网络传输这些数据和磁盘读写消耗了大量的时间。</p>
<h3 data-nodeid="5020">动手环节</h3>
<p data-nodeid="5021">首先需要注意的是，我们的试验是在 Windows 10 中进行的，如果你使用的不是管理员账户，在运行 Hadoop 命令时没有创建符号表的权限，这会导致我们执行例子程序出错，可行的解决方式有两种：</p>
<ol data-nodeid="5022">
<li data-nodeid="5023">
<p data-nodeid="5024">因为默认管理员可以创建符号表，可以使用管理员命令行启动 Hadoop 的应用；</p>
</li>
<li data-nodeid="5025">
<p data-nodeid="5026">使用 win 键 + R 调出运行，输入 gpedit.msc，在【计算机配置】-【Windows 设置】-【安全设置】-【本地策略】-【用户权限分配】-【创建符号链接】中加入你使用的用户组，然后重启系统使之生效。</p>
</li>
</ol>
<p data-nodeid="5027"><strong data-nodeid="5165">修改配置文件</strong></p>
<p data-nodeid="5028">在 D:\hadoop-3.2.1\hadoop-3.2.1\bin\ 目录下创建一个 input 文件夹，并创建 3 个 txt 文件，在每个 txt 文件中随便写一些内容，我这里写入：</p>
<ul data-nodeid="5029">
<li data-nodeid="5030">
<p data-nodeid="5031">f1.txt：hello&nbsp;world</p>
</li>
<li data-nodeid="5032">
<p data-nodeid="5033">f2.txt：hello&nbsp;hadoop</p>
</li>
<li data-nodeid="5034">
<p data-nodeid="5035">f3.txt：hello&nbsp;mapreduce</p>
</li>
</ul>
<p data-nodeid="5036"><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB712AZiDjAAH-wJvR-eY684.png" alt="图片2.png" data-nodeid="5180"></p>
<p data-nodeid="5037">创建好之后，照着我们之前在 07 节中讲的方式启动 Hadoop，然后在 HDFS 上创建 input 路径：</p>
<pre class="lang-java" data-nodeid="5038"><code data-language="java">hadoop&nbsp; fs -mkdir /input
</code></pre>
<p data-nodeid="5039">把刚才新建的三个文件从本地存储到 HDFS 中去：</p>
<pre class="lang-java" data-nodeid="5040"><code data-language="java">hadoop fs -put input/f1.txt /input
hadoop fs -put input/f2.txt /input
hadoop fs -put input/f3.txt /input
</code></pre>
<p data-nodeid="5041">然后查看是否已经上传成功：</p>
<pre class="lang-java" data-nodeid="5042"><code data-language="java">hadoop fs -ls /input
</code></pre>
<p data-nodeid="5043"><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB726AQXroAAPFdthYS3M284.png" alt="图片3.png" data-nodeid="5186"></p>
<p data-nodeid="5044">我们也可以通过前面介绍的监控页面（<a href="http://localhost:9870/dfshealth.html#tab-overview" data-nodeid="5190">http://localhost:9870/dfshealth.html#tab-overview</a>）来查看 HDFS 上的文件，通过最右侧 Utilities 下面的Browse&nbsp;the&nbsp;file&nbsp;system 即可。</p>
<p data-nodeid="5045"><img src="https://s0.lgstatic.com/i/image6/M00/13/40/Cgp9HWBB73mAdIZUAAIhKFLU5UM538.png" alt="图片4.png" data-nodeid="5194"></p>
<p data-nodeid="5046">接下来我们使用命令来执行 WordCount 程序：</p>
<pre class="lang-java" data-nodeid="5047"><code data-language="java">hadoop jar ../share/hadoop/mapreduce/hadoop-mapreduce-examples-<span class="hljs-number">3.2</span><span class="hljs-number">.1</span>.jar wordcount /input /output
</code></pre>
<p data-nodeid="5048">注意，jar 后面的第一个参数是要执行的 jar 包路径，第二个参数是选择 jar 包中的程序，第三个参数是 HDFS 上的输入目录，最后一个参数是输出目录，由于 WordCount 的代码中会去创建输出路径，所以我们不需要提前创建好。</p>
<p data-nodeid="5049">如果执行成功，我们可以通过输出的日志看到 MapReduce 过程执行完成，如下图所示：</p>
<p data-nodeid="5050"><img src="https://s0.lgstatic.com/i/image6/M00/13/3D/CioPOWBB74WAPIiRAAC491VixDU630.png" alt="Drawing 4.png" data-nodeid="5200"></p>
<p data-nodeid="5051">执行成功之后，我们查看 output 路径下的结果：</p>
<pre class="lang-java" data-nodeid="5052"><code data-language="java">hadoop fs -cat /output/part-r-<span class="hljs-number">00000</span>
</code></pre>
<p data-nodeid="5053">你可以看到如下结果：</p>
<pre class="lang-java" data-nodeid="5054"><code data-language="java">hadoop&nbsp;&nbsp;<span class="hljs-number">1</span>
hello&nbsp;&nbsp;&nbsp;<span class="hljs-number">3</span>
mapreduce&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-number">1</span>
world&nbsp;&nbsp;&nbsp;<span class="hljs-number">1</span>
</code></pre>
<p data-nodeid="5055">这一结果已经对我们文本中出现的单词数量进行了统计。</p>
<h3 data-nodeid="5056">总结</h3>
<p data-nodeid="5057">这一讲，我们介绍了 Hadoop 中的另一个核心模块 MapReduce，并且动手运行了一个示例程序。MapReduce 的思想虽然非常强大，但是从 MapReduce 的硬伤可以看出来，MapReduce 已经不太适合互联网的需求，在实际的应用中，现在 MapReduce 已经被更加强大计算框架所替代，比如 Spark 和 Flink。</p>
<p data-nodeid="5058">在这一讲的学习和动手过程中，有任何问题都欢迎在交流区与我讨论。</p>
<p data-nodeid="5059">在接下来的课程中，我们将继续介绍分布式计算框架 Spark 与 Flink，看看它们比起 MapReduce 做了什么样的改进，到时见。</p>
<hr data-nodeid="5060">
<p data-nodeid="5061"><a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="5212"><img src="https://s0.lgstatic.com/i/image6/M00/00/6D/Cgp9HWAaHaOAI85HAAUCrlmIuEw966.png" alt="Drawing 2.png" data-nodeid="5211"></a></p>
<p data-nodeid="5062"><strong data-nodeid="5216">《大数据开发高薪训练营》</strong></p>
<p data-nodeid="5063" class="te-preview-highlight">PB 级企业大数据项目实战 + 拉勾硬核内推，5 个月全面掌握大数据核心技能。<a href="https://shenceyun.lagou.com/r/rJs" data-nodeid="5220">点击链接</a>，全面赋能！</p>

---

### 精选评论

##### **瑶：
> 老师你好，请问PPT课件可以在什么地方获取

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 关注【拉勾教育公众号】回复【大数据基础】，即可领取本课程专属课件。

##### **前：
> 老师：你好，在上文中讲到，一个 MapReduce 集群只有一个JobTracker，如果在进群中同时并发运行多个job的话，只有一个JobTracker是怎么来协调管理实现的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你好，只有一个JobTracker更方便协调管理，JobTracker只负责调度，不负责具体的运算，它会根据具体的运算节点能力也就是TaskTracker上报的状态和Job的情况进行分派。具体的实现逻辑可以参考相关书目，在结束语中我会放一些参考书目供大家深入学习。

