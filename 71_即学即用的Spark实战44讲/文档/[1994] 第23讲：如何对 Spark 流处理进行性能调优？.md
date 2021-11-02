<p data-nodeid="1013">本课时会从硬件、框架的使用和配置这三个维度介绍性能调优，最后再介绍流处理中出现得最多的一个问题场景：反压。</p>
<p data-nodeid="1014">本课时的主要内容有：</p>
<ul data-nodeid="1015">
<li data-nodeid="1016">
<p data-nodeid="1017">硬件优化</p>
</li>
<li data-nodeid="1018">
<p data-nodeid="1019">使用层面的优化</p>
</li>
<li data-nodeid="1020">
<p data-nodeid="1021">配置优化</p>
</li>
</ul>
<h3 data-nodeid="1022">硬件优化</h3>
<p data-nodeid="1023">Spark Streaming 与 Spark 离线计算相比，I/O 并没有那么密集，整体负载也低于 Spark 离线计算，数据基本存放于 Executor 的内存中。但是，它对于 CPU 的要求相对较高，例如更低的延迟（较小的批次间隔）、大量微批作业的同时提交与处理。但是对基于时间窗口的操作以及对状态进行操作的算子来说，需要在内存中将这部分数据缓存，如果时间窗口跨度较长的话，需要的内存也会比较高，像 updateStateByKey 这种算子，更需要全程追踪状态，这也需要耗费不少内存，因此 Spark Streaming 集群的硬件配置也可参照离线计算型的配置。</p>
<h3 data-nodeid="1024">使用层面的优化</h3>
<p data-nodeid="1025">对于使用层面的下列优化，有些是使用技巧，有些是在某些场景下得到的经验，你可以根据自己的需求选择。</p>
<h4 data-nodeid="1026">1. 批次间隔</h4>
<p data-nodeid="1027">虽然说 Spark Streaming 号称可以达到毫秒级（理论上 50ms）的延迟，<strong data-nodeid="1089">但是在设置批次间隔时，一般不会低于 0.5s，否则大量的作业同时提交会引起负载过高</strong>。这个值可以通过反复实验来得到，我们可以先将该值设置为比较大的值，比如 10s，如果作业很快就完成了，我们可以减小批次间隔，直到 Spark Streaming 在这个时间段内刚好处理完上一批的数据，此时的批次间隔就是比较合适的了。</p>
<h4 data-nodeid="1028">2. 窗口大小与滑动步长</h4>
<p data-nodeid="1029">这两个配置的值同样对性能有巨大影响，<strong data-nodeid="1097">当性能低下时，可以考虑减小窗口大小和增加滑动步长。</strong></p>
<h4 data-nodeid="1030">3. updateStateByKey 与 mapWithState</h4>
<p data-nodeid="3806">在绝大多数情况下，使用 mapWithState 而不用 updateStateByKey，实践证明，前者的延迟表现和能够同时维护的 key 数量都远远优于后者。</p>






<h4 data-nodeid="1032">4. mapPartition 与 map</h4>
<p data-nodeid="1033">在与外部数据库交互，如写操作时，使用 mapPartition 而不要使用 map 算子，mapPartition 会在处理每个分区时连接一次数据库，而不像 map 每条数据连接一次数据库，性能优势明显。</p>
<h4 data-nodeid="1034">5. reduceByKey/aggregateByKey 与 groupByKey</h4>
<p data-nodeid="1035">前者的聚合性能要明显优于后者，因此尽量使用 reduceByKey/aggregateByKey。</p>
<h4 data-nodeid="1036">6. 反函数</h4>
<p data-nodeid="1037">在前面的课时中，我们介绍了 <strong data-nodeid="1118">reduceByKeyAndWindow 的反函数重载版本，这对于跨度很大的时间窗口，且滑动窗口与上一个时间窗口有较大重合部分的场景来说尤其有用</strong>。这里解释下反函数的由来，从滑动时间窗口的原理上来说，如果：</p>
<p data-nodeid="1038"><img alt="image (7).png" src="https://s0.lgstatic.com/i/image/M00/35/7F/Ciqc1F8VdpqASoUUAAAlFfFzIpE914.png" data-nodeid="1121"><br>
公式 1</p>
<p data-nodeid="1039">则：</p>
<p data-nodeid="1040"><img alt="image (8).png" src="https://s0.lgstatic.com/i/image/M00/35/8A/CgqCHl8VdqKAUAZ_AAAo_pY8SIU333.png" data-nodeid="1127"><br>
公式 2</p>
<p data-nodeid="1041">如果把 <em data-nodeid="1145">S</em>* <i>重叠部分的计算结果 看成自变量，<em data-nodeid="1146">S</em></i> *该滑动窗口的处理结果 看成因变量，那么就可以认为公式 2 是公式 1 的反函数。反函数的主要作用是避免重复计算。</p>
<h4 data-nodeid="1042">7. 序列化</h4>
<p data-nodeid="1043">采用 Kyro 进行序列化，可以改善 GC。</p>
<h4 data-nodeid="1044">8. 数据处理的并行程度</h4>
<p data-nodeid="1045">可以通过增大计算的并行度来提升性能，如 reduceByKey、join 等，如果不指定，并行度为配置项 spark.default.parallelism 的值，如果遇到数据倾斜还可以使用 repartition。</p>
<h4 data-nodeid="1046">9. filter 与 coalesce</h4>
<p data-nodeid="1047">与离线计算相似，在 filter 算子作用后，会产生大量零碎的分区，不利于计算，可以在后面接 coalesce 或者 repartition 算子将其进行合并或者重分。</p>
<h4 data-nodeid="1048">10. 将 Checkpoint 存储到 Alluxio</h4>
<p data-nodeid="1049">使用 Alluxio 作为 Spark Streaming 的 Checkpoint 存储介质，这有助于提高读写 Checkpoint 的性能。</p>
<h4 data-nodeid="1050">11. 资源调度</h4>
<p data-nodeid="1051">如果使用统一资源管理平台，那么批处理作业与流处理作业有可能会运行在同一个节点的不同容器中。如果批处理作业负载较高，就会对流处理作业造成较大影响，建议分离部署。<strong data-nodeid="1170">如果从提高资源利用率的角度出发，确实需要部署在一个集群，那么建议采用 Hadoop 2.6 以后引入的新特性：基于标签的调度（Label based scheduling），使流处理计算作业得到稳定且独立的计算资源。</strong></p>
<h4 data-nodeid="1052">12. 缓存数据与清除数据</h4>
<p data-nodeid="1053">与 Spark 离线计算一样，需要重复计算的数据需要用 cache 算子进行缓存。但是，这些缓存会不断占用内存，可以设置 spark.streaming.unpersist 为 true，让 Spark 来决定哪些数据需要缓存，否则需要手动控制，这样通常性能开销还会大一点。</p>
<h3 data-nodeid="1054">配置优化</h3>
<p data-nodeid="1055">配置方面的优化具体如下</p>
<h4 data-nodeid="1056">1. JVM GC</h4>
<p data-nodeid="1057">在 Executor 的堆足够大（大概 30GB 以上）时，使用 G1 GC 代替 CMS GC，否则采用 Parallel GC，如下所示：</p>
<p data-nodeid="1058">--conf&nbsp;"spark.executor.extraJavaOptions=-XX:+UseG1GC"</p>
<p data-nodeid="1059">--conf&nbsp;"spark.executor.extraJavaOptions=-XX:+UseParallelGC"</p>
<h4 data-nodeid="1060">2. spark.streaming.blockInterval</h4>
<p data-nodeid="1061">该参数设置了 Receiver 的接收块间隔时间，默认为 200ms。对于大多数 Receiver，接收的数据在存储到 Spark 的 Executor 之前，会先聚合成块的形式，每个块就是一个分区，也就是说，每个批次间隔的数据中，块的数量决定了后面类似 map 算子所处理的任务数，这也影响了数据处理的并行程度。一个批次的数据块的数量（分区数）的计算公式为：batch interval /spark.streaming.blockInterval，分子为我们设置的批次间隔，假设为 2s，那么每个批次会有 2000/200=10 个数据块。如果这个数字低于节点的 CPU 核数，说明没有充分发挥 CPU 的能力，那么可以考虑降低 spark.streaming.blockInterval 的值，但是一般也不推荐低于 50 ms。</p>
<h4 data-nodeid="1062">3. 反压</h4>
<p data-nodeid="1063">反压在流处理场景里面比较常见，是每个流处理框架必须考虑的问题。<strong data-nodeid="1205">反压的实质是，当每批数据处理时间大于批次间隔时间时，长久以往，数据会在 Executor 中的内存中迅速累积，内存会很快溢出</strong>，如果设定持久化存储基本为硬盘，则会出现大量磁盘 I/O，增加延迟。</p>
<p data-nodeid="1064">防止反压的关键是做好流量控制，如果一味地限制 Receiver 接收数据的速度，会降低整个集群的资源利用率。Spark Streaming 在 1.5 之后引入了反压机制，可以通过 spark.streaming. backpressure.enabled 来开启，开启后系统会根据每一批次作业调度与完成的情况让系统按照处理数据的速率来接收数据。实际上，就是限制 Receiver 接收数据的速度，上限由 spark.streaming. receiver.maxRate 设置，如果以 Kafka Direct 方式接收的话，上限由 spark.streaming.kafka.maxRatePerPartition 来配置。开启反压机制后，资源利用率肯定会有所下降，因此 spark.streaming.backpressure.enabled 默认关闭。</p>
<p data-nodeid="1065">Spark Streaming 是利用 PID（proportional-integral-derivative）算法来确定新的数据接收速率的，开启反压机制后的速率公式为（单位：条/秒）：</p>
<p data-nodeid="1066"><img alt="image (9).png" src="https://s0.lgstatic.com/i/image/M00/35/7F/Ciqc1F8VdriAJ7D_AAAfkkBnAEQ809.png" data-nodeid="1210"></p>
<p data-nodeid="1067">其中，<em data-nodeid="1252">V</em>newRate 为下一批次的接收速率；<em data-nodeid="1253">V</em>latestRate 为在上一批次中所确定当前批次的接收速率；<em data-nodeid="1254">V</em>error 为 <em data-nodeid="1255">V</em>latestRate 减去当前批次的实际处理速率；<em data-nodeid="1256">V</em>historicalError 为当前批次等待调度的时间乘以当前批次的处理速率再除以批次间隔；<em data-nodeid="1257">d</em>Error 为<em data-nodeid="1258">V</em>error 减去上一批次的 Verror 的差，除以当前批次完成的时间，减去上一批次完成的时间的结果。<em data-nodeid="1259">K</em>proportional、<em data-nodeid="1260">K</em>integral、<em data-nodeid="1261">K</em>derivative 为 PID 算法的 3 个重要的调适参数。</p>
<h3 data-nodeid="1068">小结</h3>
<p data-nodeid="1069">与 Spark 批处理调优一样，流处理调优也是一个与业务紧密结合的问题，不光需要对原理、参数、配置非常熟悉，还需要大量的实践。</p>
<p data-nodeid="1070">最后给你留一个思考题：</p>
<p data-nodeid="1071">在很多场景中，为了提高资源利用率，很多时候，集群中既长驻着流处理作业也跑着批处理作业，这样做有什么不好的影响呢？</p>

---

### 精选评论

##### **生：
> 集群中既长驻着流处理作业也跑着批处理作业 会导致资源的竞争，万一流处理作业资源紧张，导致任务失败

