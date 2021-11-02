<h3 data-nodeid="1" class="">背景</h3>
<p data-nodeid="2">通过前面对 SkyWalking Agent 的介绍我们知道，Agent 中的 TraceSegmentServiceClient 上报 TraceSegment 数据的方式是 gRPC（客户端流式发送）。使用客户端流式 gRPC 可以向服务端发送批量的数据，服务端在接收这些数据的时候，可以不必等所有的消息全收齐之后再发送响应，而是在接收到第一条消息的时候就及时响应，这显然比 HTTP 1.1 的交互方式更快地提供了响应。</p>
<p data-nodeid="3">这种上报方式虽然及时，但是在微服务的架构中，依然会面临一些挑战。例如，某一段时间用户请求量突增，整个后端产生的 Trace 上报请求就会增多，若是 OAP 集群无法处理这个尖峰流量，就可能导致整个 OAP 被拖垮。再例如，某些服务进行了扩容，每个后端的服务实例上报 Trace 都是要创建连接的，可能将整个 OAP 集群的对外连接数耗尽。还有可能在通过 gRPC 上报 Trace 数据的过程中网络连接意外断开或是某台 OAP 服务突然宕机，该条 Trace 数据只接收了部分，只能展示出一条断掉的 Trace 链。</p>
<p data-nodeid="4">为了避免上述问题，这里对 Trace 数据的上报方式修改为使用 Kafka 方式进行上报，使用 Kafka 上报有如下好处。</p>
<ol data-nodeid="5">
<li data-nodeid="6">
<p data-nodeid="7"><strong data-nodeid="171">削峰</strong>：Trace 数据会先写入到 Kafka 中，然后由 OAP 服务进行消费，如果出现了尖峰流量，也会先缓存到 Kafka 集群中，这样 OAP 服务不会被突增流量打垮。待尖峰流量过去之后，OAP 服务会将 Kafka 缓存的数据全部消费掉。</p>
</li>
<li data-nodeid="8">
<p data-nodeid="9"><strong data-nodeid="176">扩展性</strong>：当 Trace 数据或是其他 JVM 监控数据增大到 OAP 集群的处理上限之后，我们只需要增加新的 OAP 服务即可。</p>
</li>
<li data-nodeid="10">
<p data-nodeid="11"><strong data-nodeid="181">多副本</strong>：Kafka 中的消息会有多个副本，即使 Kafka 集群中的一台机器或是 OAP 集群的一个实例宕机，也不会导致数据丢失。</p>
</li>
</ol>
<h3 data-nodeid="12">Kafka 基础入门</h3>
<p data-nodeid="13">首先我们先来了解一下 Kafka 的整体架构以及核心概念，如下图所示。</p>
<p data-nodeid="4031"><img src="https://s0.lgstatic.com/i/image/M00/2F/54/CgqCHl8GwXOAJW_TAAENpE35u5w381.png" alt="Drawing 0.png" data-nodeid="4034"></p>


<ul data-nodeid="2400">
<li data-nodeid="2401">
<p data-nodeid="2402"><strong data-nodeid="2418">消息</strong>：Kafka 中最基本的数据单元。消息是一串主要由 key 和 value 构成的字符串，key 和 value 也都是 byte 数组。key 的主要作用是根据一定的策略，将此消息路由到指定的 Partition 中，这样就可以保证包含同一 key 的消息全部写入同一分区中。消息的真正有效负载是 value 部分的数据。为了提高网络和存储的利用率，Producer 会批量发送消息到 Kafka，并在发送之前对消息进行压缩。</p>
</li>
<li data-nodeid="2403">
<p data-nodeid="2404"><strong data-nodeid="2423">Producer</strong>：负责将消息发送到 Kafka 集群，即将消息按照一定的规则推送到 Topic 的Partition 中。这里选择分区的“规则”可以有很多种，例如：根据消息的 key 的 Hash 值选择 Partition ，或按序轮训该 Topic 全部 Partition 的方式。</p>
</li>
<li data-nodeid="2405">
<p data-nodeid="2406"><strong data-nodeid="2428">Broker</strong>：Kafka 集群中一个单独的 Kafka Server 就是一个 Broker。Broker 的主要工作就是接收 Producer 发过来的消息、为其分配 offset 并将消息保存到磁盘中；同时，接收 Consumer 以及其他 Broker 的请求，并根据请求类型进行相应处理并返回响应。</p>
</li>
<li data-nodeid="2407">
<p data-nodeid="2408"><strong data-nodeid="2433">Topic</strong>：Topic 是用于存储消息的逻辑概念，可以看作是一个消息集合。发送到 Kafka 集群的每条消息都存储到一个 Topic 中。每个 Topic 可以有多个生产者向其中推送（push）消息，也可以有任意多个消费者消费其中的消息。</p>
</li>
<li data-nodeid="2409">
<p data-nodeid="2410"><strong data-nodeid="2438">Partition</strong>：每个 Topic 可以划分成一个或多个 Partition，同一 Topic 下的不同分区包含着消息是不同的。每个消息在被添加到 Partition 时，都会被分配一个 offset，它是消息在此分区中的唯一编号，Kafka 通过 offset 保证消息在分区内的顺序，offset 的顺序性不跨分区，即 Kafka 只保证在同一个分区内的消息是有序的；同一 Topic 的多个分区内的消息，Kafka 并不保证其顺序性，如下图所示。</p>
</li>
</ul>




<p data-nodeid="4817" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/89/CgqCHl8G-OaAKGrhAABeWbnSWmg382.png" alt="image (3).png" data-nodeid="4824"></p>

<p data-nodeid="27">同一 Topic 的不同 Partition 会分配在不同的 Broker 上。 Partition 是 Kafka 水平扩展性的基础，我们可以通过增加服务器并在其上分配 Partition 的方式，增加 Kafka 的并行处理能力。</p>
<p data-nodeid="28">Partition 在逻辑上对应着一个 Log，当 Producer 将消息写入 Partition 时，实际上是写入到了 Partition 对应的 Log 中。Log 是一个逻辑概念，可以对应到磁盘上的一个文件夹。Log 由多个 Segment 组成，每个 Segment 对应一个日志文件和索引文件。在面对海量数据时，为避免出现超大文件，每个日志文件的大小是有限制的，当超出限制后则会创建新的 Segment，继续对外提供服务。这里要注意，因为 Kafka 采用顺序 IO，所以只向最新的 Segment 追加数据。为了权衡文件大小、索引速度、占用内存大小等多方面因素，索引文件采用稀疏索引的方式，文件大小并不会很大，在运行时会将其内容映射到内存，提高索引速度。</p>
<ul data-nodeid="29">
<li data-nodeid="30">
<p data-nodeid="31"><strong data-nodeid="222">保留策略（Retention Policy）&amp; 日志压缩（Log Compaction）</strong></p>
</li>
</ul>
<p data-nodeid="32">无论消费者是否已经消费了消息，Kafka 都会一直保存这些消息，但并不会像数据库那样长期保存。为了避免磁盘被占满，Kafka 会配置相应的“保留策略”（Retention Policy），以实现周期性的删除陈旧的消息。</p>
<p data-nodeid="33">Kafka 中有两种“保留策略”：一种是根据消息保留的时间，当消息在 Kafka 中保存的时间超过了指定时间，就可以被删除；另一种是根据 Topic 存储的数据大小，当 Topic 所占的日志文件大小大于一个阈值，则可以开始删除最旧的消息。Kafka 会启动一个后台线程，定期检查是否存在可以删除的消息。“保留策略”的配置是非常灵活的，可以有全局的配置，也可以针对 Topic 进行配置覆盖全局配置。</p>
<p data-nodeid="34">除此之外，Kafka 还会进行“日志压缩”（Log Compaction）。在很多场景中，消息的 key 与 value 的值之间的对应关系是不断变化的，就像数据库中的数据会不断被修改一样，消费者只关心 key 对应的最新 value 值。此时，可以开启 Kafka 的日志压缩功能，Kafka 会在后台启动一个线程，定期将相同 key 的消息进行合并，只保留最新的 value 值。日志压缩的工作原理如下图所示，图展示了一次日志压缩过程的简化版本。</p>
<p data-nodeid="6832"><img src="https://s0.lgstatic.com/i/image/M00/2F/49/Ciqc1F8GwaGAJouRAAKTqlJtZJc799.png" alt="Drawing 2.png" data-nodeid="6835"></p>



<ul data-nodeid="36">
<li data-nodeid="37">
<p data-nodeid="38"><strong data-nodeid="233">Replica</strong>：一般情况下，Kafka 对消息进行了冗余备份，每个 Partition 可以有多个 Replica（副本），每个 Replica 中包含的消息是一样的。每个 Partition 的 Replica 集合中，都会选举出一个 Replica 作为 Leader Replica，Kafka 在不同的场景下会采用不同的选举策略。所有的读写请求都由选举出的 Leader Replica 处理，其他都作为 Follower Replica，Follower Replica 仅仅是从 Leader Replica 处把数据拉取到本地之后，同步更新到自己的 Log 中。每个 Partition 至少有一个 Replica，当 Partition 中只有一个 Replica 时，就只有 Leader Replica，没有 Follower Replica。下图展示了一个拥有三个 Replica 的Partition。</p>
</li>
</ul>
<p data-nodeid="7626" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/89/CgqCHl8G-PqAAWyMAABTqAURrAc486.png" alt="image (4).png" data-nodeid="7633"></p>

<p data-nodeid="40">一般情况下，同一 Partition 的多个 Replica 会被分配到不同的 Broker 上，这样，当 Leader 所在的 Broker 宕机之后，可以重新选举新的 Leader，继续对外提供服务。</p>
<ul data-nodeid="41">
<li data-nodeid="42">
<p data-nodeid="43"><strong data-nodeid="242">ISR 集合</strong>：ISR（In-Sync Replica）集合表示的是目前“可用”（alive）且消息量与 Leader 相差不多的副本集合，这是整个副本集合的一个子集。“可用”和“相差不多”都是很模糊的描述，其实际含义是ISR集合中的副本必须满足下面两个条件：</p>
</li>
</ul>
<ol data-nodeid="44">
<li data-nodeid="45">
<p data-nodeid="46">副本所在节点必须维持着与ZooKeeper的连接。</p>
</li>
<li data-nodeid="47">
<p data-nodeid="48">副本最后一条消息的 offset 与 Leader 副本的最后一条消息的 offset 之间的差值不能超出指定的阈值。</p>
</li>
</ol>
<p data-nodeid="49">每个分区中的 Leader Replica 都会维护此分区的 ISR 集合。写请求首先是由 Leader Replica 处理，之后 Follower&nbsp;Replica 会从 Leader Replica 上拉取写入的消息，这个过程会有一定的延迟，导致 Follower&nbsp;Replica 中保存的消息略少于 Leader Replica，只要未超出阈值都是可以容忍的。如果一个 Follower Replica 出现异常，比如：宕机、发生长时间 GC 而导致 Kafka 僵死或是网络断开连接导致长时间没有拉取消息进行同步，就会违反上面的两个条件，从而被 Leader Replica 踢出 ISR 集合。当 Follower Replica 从异常中恢复之后，会继续与 Leader Replica 进行同步，当 Follower Replica “追上” Leader Replica 的时候（即最后一条消息的 offset 的差值小于指定阈值），此 Follower Replica 会被 Leader Replica 重新加入 ISR 集合中。</p>
<ul data-nodeid="50">
<li data-nodeid="51">
<p data-nodeid="52"><strong data-nodeid="252">HW&amp;LEO</strong>：HW（HighWatermark）和 LEO 与上面的 ISR 集合紧密相关。HW 标记了一个特殊的 offset ，当消费者处理消息的时候，只能拉取到 HW 之前的消息，HW 之后的消息对消费者来说是不可见的。与 ISR 集合类似，HW 也是由 Leader&nbsp; Replica 管理的。当 ISR 集合中全部的 Follower&nbsp;Replica 都拉取 HW 指定消息进行同步后，Leader&nbsp;Replica 会递增 HW 的值。Kafka 官方网站的将 HW 之前的消息的状态称为“commit”，其含义是这些消息在多个 Replica 中同时存在，即使此时 Leader&nbsp;Replica 损坏，也不会出现数据丢失。</p>
</li>
</ul>
<p data-nodeid="53">LEO（Log End&nbsp;offset）是所有的 Replica 都会有的一个 offset 标记，它指向追加到当前 Replica 的最后一个消息的 offset 。当 Producer 向 Leader&nbsp;Replica 追加消息的时候， Leader&nbsp;Replica 的 LEO 标记会递增；当 Follower Replica 成功从 Leader Replica 拉取消息并更新到本地的时候，Follower&nbsp;Replica 的 LEO 就会增加。</p>
<p data-nodeid="54">为了让你更好地理解 HW 和 LEO 之间的关系，下面通过一个示例进行分析，下图中展示了针对 offset 为 11 的消息，ISR 集合、HW 与 LEO 是如何协调工作。</p>
<p data-nodeid="10088"><img src="https://s0.lgstatic.com/i/image/M00/2F/7D/Ciqc1F8G-Q6ACSq7AABvhBoNdlo220.png" alt="image (5).png" data-nodeid="10095"></p>

<p data-nodeid="9261">①&nbsp;Producer 向此 Partition 推送消息。</p>



<p data-nodeid="57">②&nbsp;Leader&nbsp;Replica 将消息追加到 Log 中，并递增其 LEO。</p>
<p data-nodeid="58">③&nbsp;Follower&nbsp;Replica 从 Leader&nbsp;Replica 拉取消息进行同步。</p>
<p data-nodeid="59">④&nbsp;Follower&nbsp;Replica 将拉取到的消息更新到本地 Log 中，并递增其 LEO 。</p>
<p data-nodeid="60">⑤&nbsp;当 ISR 集合中所有 Replica 都完成了对 offset =11 的消息的同步，Leader&nbsp;Replica 会递增 HW。</p>
<p data-nodeid="61">在 ①~⑤ 步完成之后，offset=11 的消息就对 Consumer 可见了。</p>
<p data-nodeid="62">了解了 Replica 复制原理之后，请你考虑一下，为什么 Kafka 要这么设计？在分布式存储中，冗余备份是常见的一种设计，常用的方案有同步复制和异步复制：</p>
<ul data-nodeid="63">
<li data-nodeid="64">
<ul data-nodeid="65">
<li data-nodeid="66">
<p data-nodeid="67">同步复制要求所有能工作的 Follower&nbsp;Replica 都复制完，这条消息才会被认为提交成功。一旦有一个 Follower&nbsp;Replica 出现故障，就会导致 HW 无法完成递增，消息就无法提交，消费者获取不到消息。这种情况下，故障的 Follower&nbsp;Replica 会拖慢整个系统的性能，甚至导致整个系统不可用。</p>
</li>
<li data-nodeid="68">
<p data-nodeid="69">异步复制中，Leader&nbsp;Replica 收到生产者推送的消息后，就认为此消息提交成功。 Follower&nbsp;Replica 则异步地从 Leader&nbsp;Replica 同步消息。这种设计虽然避免了同步复制的问题，但同样也存在一定的风险，现在假设所有 Follower&nbsp;Replica 的同步速度都比较慢，它们保存的消息量都远远落后于 Leader&nbsp;Replica，如下图所示。</p>
</li>
</ul>
</li>
</ul>
<p data-nodeid="12574"><img src="https://s0.lgstatic.com/i/image/M00/2F/7E/Ciqc1F8G-RyAAeDAAAAnkFKrwaI521.png" alt="image (6).png" data-nodeid="12581"></p>

<p data-nodeid="11739">此时 Leader&nbsp;Replica 所在的 Broker 突然宕机，则会重新选举新的 Leader&nbsp;Replica，而新的 Leader&nbsp;Replica 中没有原来 Leader&nbsp;Replica 的消息，这就出现了消息的丢失，而有些 Consumer 则可能消费了这些丢失的消息，后续服务状态变得不可控。</p>



<p data-nodeid="72">Kafka 权衡了同步复制和异步复制两种策略，通过引入了 ISR 集合，巧妙地解决了上面两种方案存在的缺陷：首先，当 Follower&nbsp;Replica 的延迟过高时，会将 Leader&nbsp;Replica 被踢出 ISR 集合，消息依然可以快速提交，Producer 也可以快速得到响应，避免高延时的 Follower&nbsp;Replica 影响整个 Kafka 集群的性能。当 Leader&nbsp;Replica 所在的 Broker 突然宕机的时候，会优先将 ISR 集合中 Follower&nbsp;Replica 选举为 Leader&nbsp;Replica，新 Leader&nbsp; Replica 中包含了 HW 之前的全部消息，这就避免了消息的丢失。值得注意是，Follower&nbsp; Replica 可以批量地从 Leader&nbsp;Replica 复制消息，这就加快了网络 I/O，Follower Replica 在更新消息时是批量写磁盘，加速了磁盘的 I/O，极大减少了 Follower 与 Leader 的差距。</p>
<ul data-nodeid="15084">
<li data-nodeid="15085">
<p data-nodeid="15086"><strong data-nodeid="15096">Cluster&amp;Controller</strong>：多个 Broker 可以做成一个 Cluster（集群）对外提供服务，每个 Cluster 当中会选举出一个 Broker 来担任 Controller，Controller 是 Kafka 集群的指挥中心，而其他 Broker 则听从 Controller 指挥实现相应的功能。Controller 负责管理分区的状态、管理每个分区的 Replica 状态、监听 Zookeeper 中数据的变化等工作。Controller 也是一主多从的实现，所有 Broker 都会监听 Controller Leader 的状态，当 Leader Controller 出现故障时则重新选举新的 Controller Leader。</p>
</li>
<li data-nodeid="15087" class="">
<p data-nodeid="15088"><strong data-nodeid="15101">Consumer</strong>：从 Topic 中拉取消息，并对消息进行消费。某个消费者消费到 Partition 的哪个位置（offset）的相关信息，是 Consumer 自己维护的。在下图中，三个消费者同时消费同一个 Partition，各自管理自己的消费位置。</p>
</li>
</ul>
<p data-nodeid="15089"><img src="https://s0.lgstatic.com/i/image/M00/2F/7E/Ciqc1F8G-SeAbrU5AAAzthf0-to945.png" alt="image (7).png" data-nodeid="15108"></p>


<p data-nodeid="14241">这样设计非常巧妙，避免了 Kafka Server 端维护消费者消费位置的开销，尤其是在消费数量较多的情况下。另一方面，如果是由 Kafka Server 端管理每个 Consumer 消费状态，一旦 Kafka Server 端出现延或是消费状态丢失时，将会影响大量的 Consumer。同时，这一设计也提高了 Consumer 的灵活性，Consumer 可以按照自己需要的顺序和模式拉取消息进行消费。例如：Consumer 可以通过修改其消费的位置实现针对某些特殊 key 的消息进行反复消费，或是跳过某些消息的需求。</p>



<ul data-nodeid="80">
<li data-nodeid="81">
<p data-nodeid="82"><strong data-nodeid="294">Consumer Group</strong>：在 Kafka 中，多个 Consumer 可以组成一个 Consumer Group，一个Consumer 只能属于一个 Consumer Group。Consumer Group 保证其订阅的 Topic 的每个Partition 只被分配给此 Consumer Group 中的一个消费者处理。如果不同 Consumer Group 订阅了同一 Topic，Consumer Group 彼此之间不会干扰。这样，如果要实现一个消息可以被多个 Consumer 同时消费（“广播”）的效果，则将每个 Consumer 放入单独的一个 Consumer Group；如果要实现一个消息只被一个 Consumer 消费（“独占”）的效果，则将所有的 Consumer 放入一个 Consumer Group 中。在 Kafka 官网的介绍中，将 Consumer Group 称为“逻辑上的订阅者”（logical subscriber），从这个角度看，是有一定道理的。</p>
</li>
</ul>
<p data-nodeid="83">下图展示了一个 Consumer Group 中消费者与 Partition 之间的对应关系，其中，Consumer1 和 Consumer2 分别消费 Partition0 和 Partition1，而 Partition2 和 Partition3 分配给了 Consumer3 进行处。</p>
<p data-nodeid="17635"><img src="https://s0.lgstatic.com/i/image/M00/2F/7E/Ciqc1F8G-TOAJ2cnAABuLTyumCs642.png" alt="image (8).png" data-nodeid="17642"></p>

<p data-nodeid="16784">Consumer Group 除了实现“独占”和“广播”模式的消息处理外，Kafka 还通过 Consumer Group 实现了消费者的水平扩展和故障转移。在上图中，当 Consumer3 的处理能力不足以处理两个 Partition 中的数据时，可以通过向 Consumer Group 中添加消费者的方式，触发Rebalance 操作重新分配 Partition 与 Consumer 的对应关系，从而实现水平扩展。如下图所示，添加 Consumer4 之后，Consumer3 只消费 Partition3 中的消息，Partition4 中的消息则由 Consumer4 来消费。</p>



<p data-nodeid="20193"><img src="https://s0.lgstatic.com/i/image/M00/2F/89/CgqCHl8G-T2AAoDPAAB37LzFH3w280.png" alt="image (9).png" data-nodeid="20200"></p>

<p data-nodeid="19334">下面来看 Consumer 出现故障的场景，当 Consumer4 宕机时，Consumer Group 会自动重新分配 Partition，如下图所示，由 Consumer3 接管 Consumer4 对应的 Partition 继续处理。</p>



<p data-nodeid="22763"><img src="https://s0.lgstatic.com/i/image/M00/2F/7E/Ciqc1F8G-UuASSRJAABvDdSbF40361.png" alt="image (10).png" data-nodeid="22771"></p>
<p data-nodeid="22764">注意，Consumer Group 中消费者的数量并不是越多越好，当消费者数量超过 Partition 的数量时，会导致有 Consumer 分配不到 Partition，从而造成 Consumer 的浪费。</p>




<p data-nodeid="90">介绍完 Kafka 的核心概念后，我们通过下图进行总结，并从更高的视角审视 Kafka 集群的完整架构。</p>
<p data-nodeid="91"><img src="https://s0.lgstatic.com/i/image/M00/2F/4D/Ciqc1F8GxJWAWTtSAAKDvoKBlPU986.png" alt="Drawing 10.png" data-nodeid="311"></p>
<p data-nodeid="92">在上图中，Producer 会根据业务逻辑产生消息，之后根据路由规则将消息发送到指定的分区的 Leader&nbsp;Replica 所在的 Broker 上。在 Kafka 服务端接收到消息后，会将消息追加到 Leader&nbsp;Replica 的 Log 中保存，之后 Follower&nbsp;Replica 会与 Leader&nbsp;Replica 进行同步，当 ISR 集合中所有 Replica 都完成了此消息的同步之后，则 Leader&nbsp;Replica 的 HW 会增加，并向 Producer 返回响应。</p>
<p data-nodeid="93">当 Consumer 加入 Consumer Group 时，会触发 Rebalance 操作将 Partition 分配给不同的 Consumer 进行消费。随后，Consumer 会确定其消费的位置，并向 Kafka 集群发送拉取消息的请求， Leader&nbsp;Replica 会验证请求的 offset 以及其他相关信息，然后批量返回消息。</p>
<h3 data-nodeid="94">Kafka 环境搭建</h3>
<h4 data-nodeid="95">ZooKeeper</h4>
<p data-nodeid="96">Kafka 集群有一些元数据和选举操作会依赖 ZooKeeper，这里需要先启动 ZooKeeper 集群，前文搭建 Demo 示例（demo-webapp 和 demo-provider）时，已经搭建好了 ZooKeeper 环境，这里直接启动就好了，不再重复。</p>
<h4 data-nodeid="97">Scala 环境</h4>
<p data-nodeid="98">Kafka 是使用 Scala 语言编写的，Scala 是一种现代多范式编程语言，集成了面向对象和函数式编程的特性。Scala 语言需要运行在 Java 虚拟机之上，前面我们已经说明了 JDK8 的安装流程，不再赘述。这里使用 Scala 2.13 版本，首先从官网（<a href="https://www.scala-lang.org/download/" data-nodeid="321">https://www.scala-lang.org/download/</a>）下载 Scala 安装包并执行如下命令解压：</p>
<pre class="lang-java" data-nodeid="99"><code data-language="java">tar -zxf&nbsp;scala-<span class="hljs-number">2.13</span>.<span class="hljs-number">1.</span>tgz
</code></pre>
<p data-nodeid="100">然后编辑 .bash_profile 文件添加 $SCALA_HONME ，如下所示：</p>
<pre class="lang-java" data-nodeid="101"><code data-language="java">export SCALA_HOME=/Users/xxx/scala-<span class="hljs-number">2.13</span>.<span class="hljs-number">1</span>
export PATH=$PATH:$JAVA_HOME:$SCALA_HOME/bin
</code></pre>
<p data-nodeid="102">编辑完成后，保存并关闭 .bash_profile 文件，执行 source 命令：</p>
<pre class="lang-java" data-nodeid="103"><code data-language="java">source .bash_profile
</code></pre>
<p data-nodeid="104">最后执行 scala -version 命令，看到如下输出即安装成功：</p>
<pre class="lang-java" data-nodeid="105"><code data-language="java">scala -version
Scala code runner version <span class="hljs-number">2.13</span>.<span class="hljs-number">1</span> -- Copyright <span class="hljs-number">2002</span>-<span class="hljs-number">2019</span>, LAMP/EPFL and Lightbend, Inc.
</code></pre>
<h4 data-nodeid="106">安装 Kafka</h4>
<p data-nodeid="107">首先从 kafka 官网（<a href="http://kafka.apache.org/downloads.html" data-nodeid="336">http://kafka.apache.org/downloads.html</a>）下载 Kafka 的二进制安装包，目前最新版本是 2.4.0，我们选择在 Scala 2.13 上打包出的二进制包，如下图所示：</p>
<p data-nodeid="108"><img src="https://s0.lgstatic.com/i/image/M00/2F/4D/Ciqc1F8GxLqAHJWSAAGLMsqgETA207.png" alt="Drawing 11.png" data-nodeid="340"></p>
<p data-nodeid="109">下载完毕之后，执行如下命令解压缩：</p>
<pre class="lang-java" data-nodeid="110"><code data-language="java">tar -zxf kafka_2.<span class="hljs-number">13</span>-<span class="hljs-number">2.4</span>.<span class="hljs-number">0.</span>tgz
</code></pre>
<p data-nodeid="111">进入解压后的目录 /Users/xxx/kafka_2.13-2.4.0，创建一个空目录 logs 作为存储 Log 文件的目录。</p>
<p data-nodeid="112">然后打开 ./config/server.properties 文件，将其中的 log.dirs 这一项指向上面创建的 logs 目录，如下所示：</p>
<pre class="lang-sql" data-nodeid="113"><code data-language="sql">vim ./config/server.properties&nbsp;
<span class="hljs-comment"># A comma separated list of directories under which to store log files</span>
log.dirs=/Users/xxx/kafka_2.13-2.4.0/logs
</code></pre>
<p data-nodeid="114">最后执行如下命令即可启动 Kafka，启动过程中关注一下日志，不报错即可：</p>
<pre class="lang-java" data-nodeid="115"><code data-language="java">./bin/kafka-server-start.sh ./config/server.properties
</code></pre>
<h4 data-nodeid="116">验证</h4>
<p data-nodeid="117">这里通过 Kafka 自带的命令行 Producer 和 Consumer 验证 Kafka 是否搭建成功。首先需要创建一个名为“test”的 Topic：</p>
<pre class="lang-dart" data-nodeid="118"><code data-language="dart">./bin/kafka-topics.sh --create --zookeeper localhost:<span class="hljs-number">2181</span> \
   --replication-factor <span class="hljs-number">1</span> --partitions <span class="hljs-number">1</span> --topic test
# 输出下面的一行，即为创建成功
Created topic test.
</code></pre>
<p data-nodeid="119">接下来启动命令行 Producer，并输入一条消息“This is a test Message”，以回车结束，如下所示：</p>
<pre class="lang-java" data-nodeid="120"><code data-language="java">./bin/kafka-console-producer.sh --broker-list localhost:<span class="hljs-number">9092</span> \
 --topic test
&gt;This is a test Message
</code></pre>
<p data-nodeid="121">最后启动命令行 Consumer，可以接收到前面输入的消息，如下所示，即表示 Kafka 安装并启动成功：</p>
<pre class="lang-java" data-nodeid="122"><code data-language="java">./bin/kafka-console-consumer.sh --bootstrap-server localhost:<span class="hljs-number">9092</span> \
 --topic test --from-beginning
 &gt;This is a test Message
</code></pre>
<h3 data-nodeid="123">Agent 改造</h3>
<p data-nodeid="124">SkyWalking Agent 在 TraceSegment 结束的时候，会通过 TraceSegmentServiceClient 将 TraceSegment 序列化并发送给后端 OAP。这里我们对其进行改造，将单一的 gRPC 上报方式修改成可配置的上报方式，可配置的方式有 gRPC 调用或是 Kafka 方式，修改后的结构如下图所示：</p>
<p data-nodeid="125"><img src="https://s0.lgstatic.com/i/image/M00/2F/58/CgqCHl8GxP2AWGQxAACxh34qQEw194.png" alt="Drawing 12.png" data-nodeid="355"></p>
<p data-nodeid="126">SegmentReportStrategy 接口中定义了发送 TraceSegment 数据的 report() 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="127"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SegmentReportStrategy</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">GRPCChannelListener</span></span>{
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">report</span><span class="hljs-params">(List&lt;TraceSegment&gt; data)</span></span>;
}
</code></pre>
<p data-nodeid="128">在 AbstractSegmentReportStrategy 抽象类的 report() 方法中会根据当前发送请求打印日志信息（与 TraceSegmentServiceClient.printUplinkStatus() 方法类似），然后将请求委托给抽象方法 doReport() ，该方法由子类 KafkaSegmentReport 和 GrpcSegmentReporter 具体实现。</p>
<p data-nodeid="129">GrpcSegmentReportor 使用 gRPC 方式上报 TraceSegment 数据，具体逻辑与 TraceSegmentServiceClient 原有的 gRPC 上报方式相同，不再展开介绍。</p>
<p data-nodeid="130">再来看 KafkaSegmentReporter ，要使用 Kafka 方式上报，我们先要引入 Kafka Client 的依赖，如下所示：</p>
<pre class="lang-js" data-nodeid="131"><code data-language="js">&lt;dependency&gt;
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.kafka<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span></span>
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>kafka-clients<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span></span>
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.4.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span></span>
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="132">KafkaSegmentReporter 的大致逻辑是将序列化后的 UpstreamSegment 数据封装成一条消息，然后通过 Kafka Client 发送到指定的 Topic 中。在其构造函数中会出初始化 KafkaProducer 对象，具体实现如下：</p>
<pre class="lang-js" data-nodeid="133"><code data-language="js">public KafkaSegmentReporter(String topic) {
    if (!StringUtil.isEmpty(topic)) {
        this.topic = topic; // 默认 topic为 "sw_segment_topic"
    }
    Properties props = new Properties();
    // Kafka服务端的主机名和端口号，关于 Kafka集群的配置可以写到 agent.config
    //&nbsp;配置文件中，然后通过 Config读取，这里为了演示简单，直接硬编码了
    props.put("bootstrap.servers", "localhost:9092");
    // UpstreamSegmentSerializer用来将UpstreamSegment对象序列化成字节数组
    props.put("value.serializer", "org.apache.skywalking.apm.agent
          .core.remote.UpstreamSegmentSerializer");
    producer = new KafkaProducer&lt;&gt;(props); // 生产者的核心类
}
</code></pre>
<p data-nodeid="134">KafkaProducer 是 Kafka Producer 的核心对象，它是线程安全的。在 doReport()&nbsp;方法实现中会将 UpstreamSegment 封装成 ProducerRecord 消息发送出去，发送之前会使用上面指定的 UpstreamSegmentSerializer 将 UpstreamSegment 序列化成字节数组。 doReport() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="135"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">doReport</span><span class="hljs-params">(List&lt;TraceSegment&gt; data)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (TraceSegment segment : data) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将 TraceSegment封装成 UpstreamSegment对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; UpstreamSegment upstreamSegment = segment.transform();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 只添加了消息 value，并未指定消息的 key</span>
&nbsp; &nbsp; &nbsp; &nbsp; ProducerRecord&lt;Object, UpstreamSegment&gt; record =&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> ProducerRecord&lt;&gt;(topic, upstreamSegment);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 发送消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; producer.send(record, (recordMetadata, e) -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (e != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 该回调用来监听发送过程中出现的异常</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; segmentUplinkedCounter += data.size();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; segmentAbandonedCounter += data.size();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="136">完成 SegmentReportStrategy 接口及其实现类之后，我们需要修改 TraceSegmentServiceClient，让其在 prepare() 方法中根据配置选择上报方式：</p>
<pre class="lang-java" data-nodeid="137"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">prepare</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
    ServiceManager.INSTANCE.findService(GRPCChannelManager.class)
        .addChannelListener(<span class="hljs-keyword">this</span>);
    <span class="hljs-keyword">if</span> (Config.Report.strategy == Strategy.GRPC) {
        segmentReportStrategy = <span class="hljs-keyword">new</span> GrpcSegmentReporter();
    } <span class="hljs-keyword">else</span> {
        segmentReportStrategy = <span class="hljs-keyword">new</span> 
             KafkaSegmentReporter(Config.Report.topic);
    }
}
</code></pre>
<p data-nodeid="138">在从 DataCarrier 中消费 TraceSegment 的时候，只需委托给当前 SegmentReportStrategy 对象即可，TraceSegmentServiceClient.consume() 方法的修改如下：</p>
<pre class="lang-java" data-nodeid="139"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">consume</span><span class="hljs-params">(List&lt;TraceSegment&gt; data)</span> </span>{
    segmentReportStrategy.report(data);
}
</code></pre>
<p data-nodeid="140">最后，我们在 demo-webapp、demo-provider 使用的 agent.config 配置文件的末尾添加如下配置，将它们切换为 Kafka 方式上报：</p>
<pre class="lang-java" data-nodeid="141"><code data-language="java">report.strategy = ${SW_LOGGING_LEVEL:KAFKA}
</code></pre>
<p data-nodeid="142">相应的在 Config 中需要添加相应的 Report 内部类来读取该配置：</p>
<pre class="lang-java" data-nodeid="143"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Report</span></span>{
    <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Strategy strategy = Strategy.GRPC;
}
</code></pre>
<h3 data-nodeid="144">trace-receiver-plugin 改造</h3>
<p data-nodeid="145">trace-receiver-plugin 插件本身使用 TraceSegmentReportServiceHandler 处理 gRPC 方式上报的 UpstreamSegment 数据，相关的逻辑无须做任何修改。</p>
<p data-nodeid="146">为了处理 Kafka 上报方式 ，我们先要引入 Kafka Client 的依赖，如下所示：</p>
<pre class="lang-js" data-nodeid="147"><code data-language="js">&lt;dependency&gt;
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.kafka<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span></span>
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>kafka-clients<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span></span>
    <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.4.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span></span>
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="148">之后我们添加一个 TraceSegmentReportServiceConsumer 类，在其构造函数中会初始化 Kafka Consumer 对象，如下所示（Kafka 集群的其他配置信息也可以配置化，这里为了方便直接硬编码了）：</p>
<pre class="lang-java" data-nodeid="149"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">TraceSegmentReportServiceConsumer</span><span class="hljs-params">(SegmentParseV2.Producer segmentProducer, String topic)</span> </span>{
    Properties props = <span class="hljs-keyword">new</span> Properties();
    props.put(<span class="hljs-string">"bootstrap.servers"</span>, <span class="hljs-string">"localhost:9092"</span>); <span class="hljs-comment">// Broker的地址</span>
    props.put(<span class="hljs-string">"group.id"</span>, <span class="hljs-string">"sw_trace"</span>); <span class="hljs-comment">// 所属Consumer Group的Id</span>
    props.put(<span class="hljs-string">"enable.auto.commit"</span>, <span class="hljs-string">"true"</span>); <span class="hljs-comment">// 自动提交offset</span>
    <span class="hljs-comment">// 自动提交offset的时间间隔</span>
    props.put(<span class="hljs-string">"auto.commit.interval.ms"</span>, <span class="hljs-string">"1000"</span>);
    props.put(<span class="hljs-string">"session.timeout.ms"</span>, <span class="hljs-string">"30000"</span>);
    <span class="hljs-comment">// value使用的反序列化器</span>
    props.put(<span class="hljs-string">"value.deserializer"</span>,<span class="hljs-string">"org.apache.skywalking.oap.server    .receiver.trace.provider.handler.kafka.UpstreamSegmentDeserializer"</span>);
    <span class="hljs-keyword">this</span>.consumer = <span class="hljs-keyword">new</span> KafkaConsumer&lt;&gt;(props);
    <span class="hljs-keyword">this</span>.segmentProducer = segmentProducer;
    <span class="hljs-keyword">this</span>.topic = topic;
    <span class="hljs-comment">// 负责消费的线程</span>
    <span class="hljs-keyword">this</span>.consumerExecutor = 
         Executors.newSingleThreadScheduledExecutor();
}
</code></pre>
<p data-nodeid="150">在 TraceSegmentReportServiceConsumer.start() 方法中会启动任务，调用 cosume() 方法消费指定的 Kafka Topic（默认为 sw_segment_topic），具体实现如下：</p>
<pre class="lang-java" data-nodeid="151"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">consume</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; consumer.subscribe(Arrays.asList(topic)); <span class="hljs-comment">// 订阅Topic</span>
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (<span class="hljs-keyword">true</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 从 Kafka集群拉取消息，每次poll()可以拉取多个消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; ConsumerRecords&lt;String, UpstreamSegment&gt; records =
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; consumer.poll(<span class="hljs-number">100</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 消费消息</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (ConsumerRecord&lt;String, UpstreamSegment&gt; record:records){
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; segmentProducer.send(record.value(), SegmentSource.Agent);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="152">为了在 trace-receiver-plugin 插件启动时将 TraceSegmentReportServiceConsumer 一并启动，需要在 TraceModuleProvider.start() 方法中添加如下代码：</p>
<pre class="lang-java" data-nodeid="153"><code data-language="java">String reportStrategy = moduleConfig.getReportStrategy();
<span class="hljs-keyword">if</span>(!StringUtil.isEmpty(reportStrategy) &amp;&amp; 
         <span class="hljs-string">"kafka"</span>.equals(reportStrategy.toLowerCase())){
    segmentReportServiceConsumer = <span class="hljs-keyword">new</span> 
        TraceSegmentReportServiceConsumer(segmentProducerV2,
              moduleConfig.getKafkaTopic());
    segmentReportServiceConsumer.start(); 
}
</code></pre>
<p data-nodeid="154">最后，要在 application.yml 配置文件以及 TraceServiceModuleConfig 中添加相应的配置项，如下所示：</p>
<pre class="lang-dart" data-nodeid="155"><code data-language="dart">public <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TraceServiceModuleConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ModuleConfig</span> </span>{
&nbsp; &nbsp; ... ... <span class="hljs-comment">// 省略其他已有字段</span>
&nbsp; &nbsp; <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Getter</span> private <span class="hljs-built_in">String</span> reportStrategy = <span class="hljs-string">"kafka"</span>;
&nbsp; &nbsp; <span class="hljs-meta">@Setter</span> <span class="hljs-meta">@Getter</span> private <span class="hljs-built_in">String</span> kafkaTopic = <span class="hljs-string">"sw_segment_topic"</span>;
}
receiver-trace:
  <span class="hljs-keyword">default</span>:
    # 省略已有的配置信息
    reportStrategy: ${SW_REPORT_STRATEGY:kafka}
    kafkaTopic: ${SW_KAFKA_TOPIC:sw_segment_topic}
</code></pre>
<h3 data-nodeid="156">验证</h3>
<p data-nodeid="157">为了验证上述的改造是否成功，我们将改造后的 Agent 切换成 Kafka 上报模式，打开 trace-receiver-plugin 插件接收 Kafka 上报 Trace 的功能，同时还可以开启一个命令行 Kafka Consumer。</p>
<p data-nodeid="158">还有就是要从 apm-sdk-plugin 模块中暂时删除 apm-kafka-v1-plugin-6.2.0 模块，该插件会拦截 Kafka Client 来生成 Trace，前文没有对该模块进行修改，会导致死循环生成 TraceSegment 的问题。这个问题属于如何让 SkyWalking 自己监控自己的问题，留给你自己思考一下如何解决。</p>
<p data-nodeid="159">完成上述操作之后，可以请求 <a href="http://localhost:8000/hello/xxx" data-nodeid="383">http://localhost:8000/hello/xxx</a> ，此时 demo-provider 和 demo-provider 都会分别生成两条 TraceSegment 并通过 Kafka 方式上报。在 Kafka 的命令行 Consumer 中可以看到如下输出：</p>
<p data-nodeid="160"><img src="https://s0.lgstatic.com/i/image/M00/2F/59/CgqCHl8GxVeAbpX5AAFAyMPwgYA338.png" alt="Drawing 13.png" data-nodeid="387"></p>
<p data-nodeid="161">在 SkyWalking Rocketbot UI 中可以查找到相应的完整 Trace 信息，如下图所示，即表示上述改造成果：</p>
<p data-nodeid="162"><img src="https://s0.lgstatic.com/i/image/M00/2F/4E/Ciqc1F8GxV-AWrfaAACzTk0fGVU367.png" alt="Drawing 14.png" data-nodeid="391"></p>

---

### 精选评论


