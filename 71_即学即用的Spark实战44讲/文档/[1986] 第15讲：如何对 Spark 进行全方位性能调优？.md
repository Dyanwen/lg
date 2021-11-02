<p>通过前面的课程学习和练习，我相信你们已经能够比较熟练地编写 Spark 作业，但如何在生产环境中让 Spark 作业稳定且快速地运行是另外一个问题，本课时将回答这个问题。</p>
<p>让 Spark 作业在海量数据面前稳定且快速地运行，这就需要对 Spark 进行性能调优。调优 Spark 是一个持续的过程，随着你对 Spark、数据本身、业务场景愈发了解，调优的思路也会更加多样，这是一个持续累积的过程。能够有针对性地对 Spark 作业进行调优是一名有经验的大数据工程师的必备技能。本课时将会从硬件、资源管理平台与使用方式 3 个维度介绍如何对 Spark 进行性能调优。在介绍调优之前，我们先来看看如何查看 Spark 的作业日志。</p>
<h3>日志收集</h3>
<p>如果作业执行报错或者速度异常，通常需要查看 Spark 作业日志，Spark 日志通常是排错的唯一根据，更是作业调优的好帮手。查看日志的时候，需要注意的是 Spark 作业是一个分布式执行的过程，所以日志也是分布式的，联想到 Spark 的架构，Spark 的日志也分为两个级别：</p>
<ul>
<li>Driver</li>
<li>Executor</li>
</ul>
<p>一般来说，小错误通常可以从 Driver 日志中定位，但是复杂一点的问题，还是要从 Executor 的执行情况来判断。如果我们选取 yarn-client 的模式执行，日志会输出到客户端，我们直接查看即可，非常方便。我们可以用下面这种方式收集，如下：</p>
<pre><code data-language="java" class="lang-java">nohup ./bin/spark-submit \
--<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">org</span>.<span class="hljs-title">apache</span>.<span class="hljs-title">spark</span>.<span class="hljs-title">examples</span>.<span class="hljs-title">SparkPi</span> \
--<span class="hljs-title">master</span> <span class="hljs-title">yarn</span> \
--<span class="hljs-title">deploy</span>-<span class="hljs-title">mode</span> <span class="hljs-title">client</span> \
--<span class="hljs-title">executor</span>-<span class="hljs-title">memory</span> 20<span class="hljs-title">G</span> \
--<span class="hljs-title">num</span>-<span class="hljs-title">executors</span> 50 \
/<span class="hljs-title">path</span>/<span class="hljs-title">to</span>/<span class="hljs-title">examples</span>.<span class="hljs-title">jar</span> \
</span></code></pre>
<p>1000&nbsp; &gt;&gt;&nbsp; o&nbsp; &amp;<br>
其中 nohup 和 &amp; 表示后台执行，&gt;&gt; o 表示将日志输出到文件 o 中。</p>
<p>查看 Executor 的日志需要先将散落在各个节点（Container）的日志收集汇总成一个文件。以 YARN 平台为例：</p>
<pre><code data-language="java" class="lang-java">yarn logs -applicationId application_1552880376963_0002&nbsp;&gt;&gt; o
application_1552880376963_0002&nbsp;是 Spark 作业 id，当汇聚为一个文件后，我们就可以对其进行查看了。打开文件，我们发现这份日志是这样组织的：
container_0
-----------------------------------------------------
WARN.....
ERROR.....
......（日志内容）
 
container_1
-----------------------------------------------------
......（日志内容）
......
</code></pre>
<p>这也非常好理解，本来就是从各个 Container 中收集并拼接的，但是这种方式给我们定位造成了一定障碍。<strong>阅读这样的日志，最重要的是找到最开始报错的那一句日志，因为一旦作业报错，几乎会造成所有 Container 报错，但大部分错误日志都对定位原因没有什么帮助。所以拿到这份日志要做的第一件事是利用时间戳和 ERROR 标记定位最初的错误日志。这种方式通常可以直接解决一半以上的报错问题。</strong></p>
<h3>硬件配置与资源管理平台</h3>
<p>构建 Spark 集群的硬件只需普通的商用 PC Server 即可，由于 Spark 作业对内存需求巨大，建议配置高性能 CPU、大内存的服务器，以下是建议配置：</p>
<p>内存：256G</p>
<p>CPU：Intel E5-2640v4</p>
<p>硬盘：3T * 8</p>
<p>该 CPU 是双路 6 核心，且具有超线程技术，所以一个 CPU 相当于有 2 * 6 * 2 = 24 核心。对于交换机的选择，通常，如果在生产环境使用，那么无论集群规模大小，都应该直接考虑万兆交换机，对于上千的集群，还需要多台交换机进行堆叠才能满足需求。</p>
<p>Spark 基于资源管理平台运行，该平台对于 Spark 来说就像一个资源池一样，资源池的大小取决于每个物理节点有多少资源供资源管理平台调度。一般来说，每台节点应预留 20% 的资源保证操作系统与其他服务稳定运行，对于前面提到的机器配置，加入资源池的内存为 200G，CPU 为 20 核。假设使用 YARN 作为资源管理平台，相关配置如下：</p>
<pre><code data-language="java" class="lang-java">yarn.nodemanager.resource.memory-mb = <span class="hljs-number">200</span>G
yarn.nodemanager.resource.cpu-vcores = <span class="hljs-number">20</span>
</code></pre>
<p>假设 YARN 集群中有 10 个 NodeManager 节点，那么总共的资源池大小为 2000G、200 核。在 Spark 作业运行时，用户可以通过集群监控页面来查看集群 CPU 使用率，如果发现 CPU 使用率一直维持在偏低的水平，可以尝试将 yarn.nodemanager.resource.cpu-vcores 改大。内存与 CPU 资源设置应该维持一个固定的比例，如 1:5，这样在提交作业时，也按照这个比例来申请资源，可以提高集群整体资源利用率。</p>
<p>一般来说，YARN 集群中会运行各种各样的作业，这样资源利用率会比较高，但是也经常造成 Spark 作业在需要时申请不到资源，这时可以采取 YARN 的新特性：基于标签的调度，在某些节点上打上相应的标签，来实现部分资源的隔离。</p>
<p>这部分内容对于工程师与分析师来说，一般接触不到，属于大数据运维工程师职责的范畴，但是对于调优来说非常重要，有必要了解。</p>
<h3>参数调优与应用调优</h3>
<p>本课时主要从使用层面来介绍调优，其中会涉及参数调优、应用调优甚至代码调优。</p>
<h4>1. 提高作业并行度</h4>
<p>在作业并行程度不高的情况下，最有效的方式就是提高作业并行程度。在 Spark 作业划分中，一个 Executor 只能同时执行一个 Task ，一个计算任务的输入是一个分区（partition），所以改变并行程度只有一个办法，就是提高同时运行 Executor 的个数。通常集群的资源总量是一定的，这样 Executor 数量增加，必然会导致单个 Executor 所分得的资源减少，这样的话，在每个分区不变的情况下，有可能会引起性能方面的问题，所以，我们可以增大分区数来降低每个分区的大小，从而避免这个问题。</p>
<p>RDD 一开始的分区数与该份数据在 HDFS 上的数据块数量一致，后面我们可以通过 coalesce 与 repartition 算子进行重分区，这其实改变的是 Map 端的分区数，如果想改变 Reduce 端的分区数，有两个办法，一个是修改配置 spark.default.parallelism，该配置设定所有 Reduce 端的分区数，会对所有 Shuffle 过程生效，此外还可以直接在算子中将分区数作为参数传入，绝大多数算子都有分区数参数的重载版本，如 groupByKey(600) 等。在 Shuffle 过程中，Shuffle 相关的算子会构建一个哈希表，Reduce 任务有时会因为这个表过大而造成内存溢出，这时就可以试着增大并行程度。</p>
<h4>2. 提高 Shuffle 性能</h4>
<p>Shuffle 是 Spark 作业中关键的一环，也是性能调优的重点，先来看看 Spark 参数中与 Shuffle 性能有关的有哪些：</p>
<pre><code data-language="java" class="lang-java">spark.shuffle.file.buffer
spark.reducer.maxSizeInFlight
spark.shuffle.compress
</code></pre>
<p>根据课时 11 的内容，第 1 个配置是 Map 端输出的中间结果的缓冲区大小，默认 32K，第二个配置是 Map 端输出的中间结果的文件大小，默认为 48M，该文件还会与其他文件进行合并。第三个配置是 Map 端输出是否开启压缩，默认开启。缓冲区当然越大，写入性能越高，所以有条件可以增大缓冲区大小，可以提升 Shuffle Write 的性能，但该参数实际消耗的内存为 C * spark.shuffle.file.buffer，其中 C 为执行该任务的核数。在 Shuffle Read 的过程中，Reduce Task 所在的 Executor 会按照 spark.reducer.maxSizeInFlight 的设置大小去拉取文件，这需要创建内存缓冲区来接收，在内存足够大的情况下，可以考虑提高 spark.reducer.maxSizeInFlight 的值来提升 Shuffle Read 的效率。spark.shuffle.compress 配置项默认为 true，表示会对 Map 端输出进行压缩。</p>
<p>Spark Shuffle 会将中间结果写入到 spark.local.dir 配置的目录下，可以将该目录配置多路磁盘目录，以提升写入性能。</p>
<h4>3. 内存管理</h4>
<p>Spark 作业中内存主要有两个用途：计算和存储。计算是指在 Shuffle，连接，排序和聚合等操作中用于执行计算任务的内存，而存储指的是用于跨集群缓存和传播数据的内存。在 Spark 中，这两块共享一个统一的内存区域（M），如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/C8/Ciqc1F7WFsmAQOFQAABUiQJ1H3s108.png" alt="图片1.png"></p>
<p>用计算内存时，存储部分可以获取所有可用内存，反之亦然。如有必要，计算内存也可以将数据从存储区移出，但会在总存储内存使用量下降到特定阈值（R）时才执行。换句话说，R 决定了 M 内的一个分区，在这个分区中，数据不会被移出。由于实际情况的复杂性，存储区一般不会去占用计算区。</p>
<p>这样设计是为了对那些不使用缓存的作业可以尽可能地使用全部内存；而需要使用缓存的作业也会有一个区域始终用来缓存数据，这样用户就可以不需要知道其背后复杂原理，自己根据实际内存需求来调节 M 与 R 的值，以达到最好效果。下面是决定 M 与 R 的两个配置：</p>
<ul>
<li>spark.memory.fraction，该配置表示 M 占 JVM 堆空间的比例，默认为 0.6，剩下 0.4 用于存储用户数据结构、Spark 中的内部元数据并防止在应对稀疏数据和异常大的数据时出现 OutOfMemory 的错误；</li>
<li>spark.memory.storageFraction，该配置表示 R 占 M 的比例，默认为 0.5，这部分缓存的数据不会被移出。</li>
</ul>
<p>上面两个默认值基本满足绝大多数作业的使用，在特殊情况可以考虑设置 spark.memory.fraction 的值以适配 JVM 老年代的空间大小，默认 JVM 老年代在不经过设置的情况下占 JVM 的 2/3，所以这个值是合理的。</p>
<p>Spark Executor 除了堆内存以外，还有非堆内存空间，这部分通过参数spark.yarn.executor.memoryoverhead 进行配置，最小为 384MB，默认为 Executor 内存的 10%。所以整个Executor JVM 消耗的内存为:</p>
<pre><code data-language="java" class="lang-java">spark.yarn.executor.memoryoverhead + spark.executor.memory
</code></pre>
<p>其中：</p>
<pre><code data-language="java" class="lang-java">M = spark.executor.memory * spark.memory.fraction
R = M * spark.memory.storageFraction
</code></pre>
<p>此外，Spark 还有可能会用到堆外内存 O：</p>
<pre><code data-language="java" class="lang-java">O = spark.memory.offHeap.size
</code></pre>
<p>所以整个 Spark 的内存管理布局如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/C8/Ciqc1F7WFuGAMzJVAABYCZ_TNcw607.png" alt="图片2.png"></p>
<p>用户需要知道每个部分的大小应如何调节，这样才能针对场景进行调优。这其实是 Spark 实现的一种比较简化且粗粒度的内存调节方案。如果用户想要更精细地调整内存的每个区域，就需要在参数中 spark.executor.extraClassPath 配置 Java 选项了，这种方式只针对富有经验的工程师，对于普通用户来说不太友好。</p>
<h4>4. 序列化</h4>
<p>序列化是以时间换空间的一种内存取舍方式，其根本原因还是内存比较吃紧，我们可以优先选择对象数组或者基本类型而不是那些集合类型来实现自己的数据结构，fastutil 包提供了与 Java 标准兼容的集合类型。除此之外，还应该避免使用大量小对象与指针嵌套的结构。我们可以考虑使用数据 ID 或者枚举对象来代替字符串键。如果内存小于 32GB，可以设置 Java 选项 -XX:+UseCompressedOops 来压缩指针为 4 字节，以上是需要用到序列化之前可以做的调优工作，以节省内存。</p>
<p>对于大对象来说，可以使用 RDD 的 persist 算子并选取 MEMORY_ONLY_SER 级别进行存储，更好的方式则是以序列化的方式进行存储。这相当于用时间换空间，因为反序列化时会造成访问时间过慢，如果想用序列化的方式存储数据，推荐使用 Kyro 格式，它比原生的 Java 序列化框架性能优秀（官方介绍，性能提升 10 倍）。Spark 2.0 已经开始用 Kyro 序列化 shuffle 中传输的字符串等基础类型数据了。</p>
<p>要想使用 Kyro 序列化库，要将需要序列化的类在 Kyro 中注册方可使用。使用步骤如下。</p>
<ol>
<li>编写一个注册器，实现 KyroRegister 接口，在 Kyro 中注册你需要使用的类：</li>
</ol>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">YourKryoRegistrator</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">KryoRegistrator</span> </span>{
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerClasses</span><span class="hljs-params">(Kryo kryo)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">////在Kryo序列化库中注册自定义的类</span>
	kryo.register(YourClass<span class="hljs-class">.<span class="hljs-keyword">class</span>, <span class="hljs-title">new</span> <span class="hljs-title">FieldSerializer</span>(<span class="hljs-title">kryo</span>, <span class="hljs-title">YourClass</span>.<span class="hljs-title">class</span>))</span>;
	}
}
</code></pre>
<ol start="2">
<li>设置序列化工具并配置注册器：</li>
</ol>
<pre><code data-language="java" class="lang-java">……
spark.config(<span class="hljs-string">"spark.serializer"</span>, <span class="hljs-string">"org.apache.spark.serializer.KryoSerializer"</span>)
spark.config(<span class="hljs-string">"spark.kryo.registrator"</span>, YourKryoRegistrator<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getName</span>())
</span></code></pre>
<h4>5.&nbsp;JVM垃圾回收（GC）调优</h4>
<p>通常来说，那种只读取 RDD 一次，然后对其进行各种操作的作业不太会引起 GC 问题。当 Java 需要将老对象释放，为新对象腾出空间时，需要追踪所有 Java 对象，然后在其中找出没有使用的那些对象。GC 的成本与 Java 对象数量成正比，所以使用较少对象的数据结构会大大减轻 GC 压力，如直接使用整型数组，而不选用链表。<strong>通常在出现 GC 问题的时候，序列化缓存是首先应该尝试的方法。</strong></p>
<p>由于执行计算任务需要的内存和缓存 RDD 的内存互相干扰，GC 也可能成为问题。这可以控制分配给 RDD 缓存空间来缓解这个问题。</p>
<p>GC 调优的第 1 步是搞清楚 GC 的频率和花费的时间，这可以通过添加 Java 选项来完成：</p>
<pre><code data-language="java" class="lang-java">-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps
</code></pre>
<p>在 Spark 运行时，一旦发生 GC，就会被记录到日志里。</p>
<p>为了进一步调优 JVM，先来看看 JVM 如何管理内存。Java 的堆空间被划分为 2 个区域：年轻代、老年代，顾名思义，年轻代会保存一些短生命周期对象，而老年代会保存长生命周期对象。年轻代又被划分为 3 个区域：一个 Eden 区，两个 Supervisor 区，如下图所示。简单来说，GC 过程是这样的：当 Eden 区被填满后，会触发 minor GC，然后 Eden 区和 Supvisor1 区还存活的对象被复制到 Supervisor2 区，如果某个对象太老或者 Supervisor2 区已满，则会将对象复制到老年代中，当老年代快满了，则会触发 full GC。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/C8/Ciqc1F7WFv6AUG-CAABmaFyaNF8801.png" alt="图片3.png"></p>
<p>在 Spark 中，GC 调优的目的是确保只有长生命周期的对象才会保存到老年代中，年轻代有充足的空间来存储短生命周期对象。这会有助于避免执行 full GC 来回收任务执行期间生成的临时对象，有以下几个办法：</p>
<ul>
<li>通过收集到的 GC 统计信息来检查是否有过多 GC，如果任务在完成之前多次触发 full GC，则意味着没有足够的内存可用于执行任务；</li>
<li>如果 minor GC 次数过多，但并没有 major GC，可以为 Eden 区分配更多的内存来缓解，可以将 Eden 区大小预估为每个任务需要的内存空间，如果 Eden 区的大小为 E，则可以使用选项 -Xmn = 4 / 3 * E 来设置年轻代大小，增加的 1/3 为 Supervisor 区；</li>
<li>如果通过收集到的 GC 统计信息，发现老年代快满了，可以通过 spark.memory.fraction 来减少用于缓存的内存空间；少缓存一点总比执行缓慢好，也可以考虑减少年轻代的大小，这可以通过设置 -Xmn 来实现，也可以设置 JVM 的 NewRatio 参数，该参数表示老年代与年轻代之间的比值，许多 JVM 默认为 2（2:1），这意味着老年代占堆空间的 2/3。该比例应该要大于 spark.memory.fraction 所设置的比例；</li>
<li>在某些 GC 是瓶颈的情况下，可以通过 -XX:+UseG1GC 开启 G1GC，可以提高 GC 性能，对较大的堆，可能需要增加 G1 区大小，使用 -XX:G1HeapRegionSize=n 来进行设置；</li>
<li>如果任务从 HDFS 读取数据，则可以以此来估计任务使用的内存量，解压缩块大小通常是 HDFS 数据块大小（假设为 256MB）的 2~3 倍，如果希望有足够 3~4 个任务内存空间，则 Eden 区大小为 4 * 3 * 256MB；每当我们对 Java 选项做出调整后，要通过监控工具来查看 GC 花费的时间和频率是否有变化。可以通过在作业中设置 spark.executor.extraJavaOptions选项来指定执行程序的 GC 选项以及 JVM 内存各个区域的精确大小，但不能设置 JVM 堆大小，该项只能通过 --executor-memory 或者 spark.executor.memory 来进行设置。</li>
</ul>
<h4>6. 将经常被使用的数据进行缓存</h4>
<p>如果某份数据经常会被使用，可以尝试用 cache 算子将其缓存，有时效果极好。</p>
<h4>7. 使用广播变量避免 Hash 连接操作</h4>
<p>在进行连接操作时，可以尝试将小表通过广播变量进行广播，从而避免 Shuffle，这种方式也被称为 Map 端连接。</p>
<h4>8. 聚合 filter 算子产生的大量小分区数据</h4>
<p>在使用 filter 算子后，通常数据会被打碎成很多个小分区，这会影响后面的执行操作，可以先对后面的数据用 coalesce 算子进行一次合并。</p>
<h4>9. 根据场景选用高性能算子</h4>
<p>很多算子都能达到相同的效果，但是性能差异却比较大，例如在聚合操作时，选择 reduceByKey 无疑比 groupByKey 更好；在 map 函数初始化性能消耗太大或者单条记录很大时，mapPartition 算子比 map 算子表现更好；在去重时，distinct 算子比 groupBy 算子表现更好。</p>
<h4>10. 数据倾斜</h4>
<p><strong>数据倾斜是数据处理作业中的一个非常常见也是非常难以处理的一个问题。</strong> 正常情况下，数据通常都会出现数据倾斜的问题，只是情况轻重有别而已。数据倾斜的症状是大量数据集中到一个或者几个任务里，导致这几个任务会严重拖慢整个作业的执行速度，严重时甚至会导致整个作业执行失败。如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/D5/CgqCHl7WGCWAT_qZAAGa1IyRwSw294.png" alt="Lark20200602-171237.png"></p>
<p>可以看到 Task A 处理了绝大多数数据，其他任务执行完成后，需要等待此任务执行完成，作业才算完成。对于这种情况，可以采取以下几种办法处理：</p>
<ul>
<li>过滤掉脏数据</li>
</ul>
<p>很多情况下，数据倾斜通常是由脏数据引起的，这个时候需要将脏数据过滤。</p>
<ul>
<li>提高作业的并行度</li>
</ul>
<p>这种方式从根本上仍然不能消除数据倾斜，只是尽可能地将数据分散到多个任务中去，这种方案只能提升作业的执行速度，但是不能解决数据倾斜的问题。</p>
<ul>
<li>广播变量</li>
</ul>
<p>可以将小表进行广播，避免了 Shuffle 的过程，这样就使计算相对均匀地分布在每个 Map 任务中，但是对于数据倾斜严重的情况，还是会出现作业执行缓慢的情况。</p>
<ul>
<li>将不均匀的数据进行单独处理</li>
</ul>
<p>在连接操作的时候，可以先从大表中将集中分布的连接键找出来，与小表单独处理，再与剩余数据连接的结果做合并。处理方法为：如果大表的数据存在数据倾斜，而小表不存在这种情况，可以将大表中存在倾斜的数据提取出来，并将小表中对应的数据提取出来，这时可以将小表中的数据扩充 n 倍，而大表中的每条数据则打上一个 n 以内的随机数作为新键，而小表中的数据则根据扩容批次作为新键，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/C8/Ciqc1F7WFxWADnk5AAFNpUVH_hE246.png" alt="图片4.png"></p>
<p>这种方式可以将倾斜的数据打散，从而避免数据倾斜。</p>
<p>对于那种分组统计的任务，可以通过两阶段聚合的方案来解决，首先将数据打上一个随机的键值，并根据键的哈希值进行分发，将数据均匀的分散到多个任务中去，然后在每个任务中按照真实的键值做局部聚合，最后再按照真实的键值分发一次，得到最后的结果，如下图所示，这样，最后一次分发的数据已经是聚合过后的数据，就不会出现数据倾斜的情况。这种方法虽然能够解决数据倾斜的问题但只适合聚合计算的场景。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/C8/Ciqc1F7WFyGAf-GtAAL8itCZco0430.png" alt="图片5.png"></p>
<h3>小结</h3>
<p>本课时介绍了如何从几个方面对 Spark 作业进行调优。调优之前，看作业日志是基本功，这个没有什么捷径，只能多看。调优这个话题是一个很个性化的问题。对于离线计算任务，时间当然很重要，但不一定是最重要的。通常来说，对于实时处理，时间通常是唯一优化目标，如果执行时间有优化的空间，当然会不遗余力地进行优化。但是，对于离线计算任务，如 Spark 作业，作业执行时间并没有那么重要。通常，这类作业都是在夜深人静的晚上执行，1 小时与 90 分钟真的差异就那么大吗，不一定，所以对于离线计算作业来说，作业执行时间并不是最重要的，开发效率同样重要。</p>
<p>这里给你留一个思考题：通过 Spark Web UI 查看作业的执行情况。</p>
<p>未来的趋势，一定是 Spark 越来越智能，越来越简单，Spark 希望开发人员专注于业务，而对框架则无须过多关注。</p>

---

### 精选评论

##### *宁：
> 一个 Executor 只能同时执行一个 Task，这句话应该不对吧？一个 Executor 能同时执行多个Task吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是的，一个Excutor同时只能执行一个Task

##### **成：
> RDD 一开始的分区数与该份数据在 HDFS 上的数据块数量一致，后面我们可以通过 coalesce 与 repartition 算子进行重分区，这其实改变的是 Map 端的分区数，如果想改变 Reduce 端的分区数，有两个办法，一个是修改配置 spark.default.parallelism，该配置设定所有 Reduce 端的分区数，会对所有 Shuffle 过程生效，此外还可以直接在算子中将分区数作为参数传入，绝大多数算子都有分区数参数的重载版本，如 groupByKey(600) 等。在 Shuffle 过程中，Shuffle 相关的算子会构建一个哈希表，Reduce 任务有时会因为这个表过大而造成内存溢出，这时就可以试着增大并行程度。DataSet的api好像都不能直接在算子中将分区数作为参数传入，且使用Dataset api的时候参数应该这么设置：spark.sql.shuffle.partitions

##### **用户1827：
> 这节不错

##### *壮：
> 真心不错，讲得实在太好了

