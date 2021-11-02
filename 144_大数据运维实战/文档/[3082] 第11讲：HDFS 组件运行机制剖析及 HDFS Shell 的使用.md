<h3><strong>HDFS 的基本架构</strong></h3>
<p>Hadoop 中的分布式文件系统 HDFS 为大数据平台提供了统一的存储，它主要由三部分构成，分别是 NameNode、DataNode 和 SecondaryNameNode。如果是 HA 架构，那么还有 StandbyNameNode 和 JournalNode。</p>
<ul>
<li>NameNode（名字节点，或者元数据节点）是 HDFS 的管理节点，专门用来存储元数据信息，所谓元数据指的是除文件内容之外的数据，比如数据块存储位置、文件权限、文件大小等信息。元数据首先保存在内存中，然后定时持久化到硬盘上，在 NameNode 启动时，元数据就会从硬盘加载到内存中。后续的操作，元数据都是在内存中进行读写操作的。</li>
<li>DataNode（数据节点）主要用来存储数据，是 HDFS 中真正存储文件的部分。HDFS 中的文件以块的形式保存在 DataNode 所在服务器的本地磁盘上，同时 DataNode 也维护了 block id 到 DataNode 本地文件的映射关系。</li>
<li>SecondaryNameNode：表示 NameNode 元数据的备份节点，它主要用于备份 NameNode 的元数据，这个备份不是热备份，而是定时异地冷备份。因此，在 NameNode 出问题时，通过 SecondaryNameNode 不能完整恢复元数据信息，可能会丢失一部分数据。基于此，出现了后续的 StandbyNameNode。</li>
<li>StandbyNameNode：表示 NameNode 的备用节点，此节点处于 Standby 状态，不接受用户请求。但它会通过 JournalNode 时刻同步 NameNode 的元数据信息，当 NameNode（处于 Active 状态）发生故障后，ZooKeeper 会及时发现问题，进而执行主、备切换，StandbyNameNode 就变成 Active 状态。这种高可用机制是完全意义上的热备份，可以做到零数据丢失。生产环境下建议配置此热备份模式。</li>
<li>JournalNode：主要用在 NameNode 的 HA 架构中，是实现主备 NameNode 之间共享数据的桥梁。NameNode 将元数据变动信息实时写入 JournalNode 中，然后 StandbyNameNode 实时从 JournalNode 中读出元数据信息，接着应用到自身，保持 NameNode 和 StandbyNameNode 元数据的实时同步。</li>
</ul>
<p>下图是 HDFS 分布式文件系统图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/18/CgqCHl7U4HOAXZEFAAJszKiTB6o267.png" alt="2.png"></p>
<p>由图可知，NameNode、DataNode 和 SecondaryNameNode 构成了分布式文件系统 HDFS。其中 SecondaryNameNode 是可选服务，如果通过 StandbyNameNode 构建 HA 架构的话，那么可以不需要 SecondaryNameNode。</p>
<p>此外 DataNode 是由多个节点组成，每个 DataNode 上都是以数据块的形式存储数据的，而 HDFS Client 是访问 HDFS 的客户端。当一个客户端要请求 HDFS 数据的时候，首先需要和 NameNode 进行交互，然后经过 NameNode 分配需要查询的 DataNode 服务节点。最后，客户端就直接和 DataNode 节点进行交互通信，进而实现数据的读写等操作。</p>
<h3>NameNode 工作机制剖析</h3>
<p>NameNode 是 HDFS 的心脏，它管理和维护着整个 HDFS 文件系统，主要有以下作用：</p>
<ul>
<li>负责接收用户的操作请求；</li>
<li>负责管理文件系统命名空间（NameSpace）、集群配置信息及存储块的复制等；</li>
<li>负责文件目录树的维护以及文件对应 block 列表的维护；</li>
<li>负责管理 block 与 DataNode 之间的关系。</li>
</ul>
<p>这里说一下什么是<strong>命名空间</strong>，在分布式文件系统中，文件是以块的形式分散存储在不同的节点，这些分散在不同节点中的数据块可能属于同一个文件。为了组织众多的文件，可以把文件放到不同的文件夹中，文件夹一级一级的包含，这种组织形式就称为命名空间。命名空间管理着整个 HDFS 中的所有文件，非常重要。</p>
<p>在 HDFS 中，FsImage 和 Edit Log 是 NameNode 两个非常重要的文件。它们存储在 NameNode 节点的本地磁盘上，这就是 NameNode 的元数据信息。其中，FsImage 文件用来记录数据块到文件的映射、目录或文件的结构、属性等信息，里面记录了自最后一次检查点之前 HDFS 文件系统中所有目录和文件的信息。</p>
<p>Edit Log 文件记录了对文件的创建、删除、重命名等操作日志，也就是自最后一次检查点之后所有针对 HDFS 文件系统的操作都会记录在 Edit Log 文件中。例如，在 HDFS 中创建一个文件， NameNode 就会在 Edit Log 中插入一条记录，同样修改文件的副本系数也会在 Edit Log 中插入一条记录。</p>
<p>由此可知，FsImage 和 Edit Log 组成了 NameNode 元数据信息，这些元数据信息的损坏会导致整个集群的失效。因此，可以将元数据配置成支持多个 FsImage 和 Edit Log 的副本，任何 FsImage 和 Edit Log 的更新都会同步到每一份副本中。这个功能可以通过在 HDFS 配置文件 hdfs-site.xml 中添加 dfs.NameNode.name.dir 参数来实现。</p>
<p>当 NameNode 启动时，它先将 Fsimage 中的信息加载到内存形成文件系统镜像，然后把 Edit Log 回放到这个文件系统镜像上。这样，FsImage 就保持了最新的元数据。最后，它还会将这个 FsImage 文件从内存中保存到本地磁盘上，同时删除旧的 Edit Log。因为旧的 Edit Log 的事务都已经作用在 FsImage 上了，这个合并过程称为一个<strong>检查点（checkpoint）</strong>。</p>
<p>检查点跟数据库里面的检查点概念类似，它的出现是为了解决 Edit Log 文件会不断变大的问题。在 NameNode 运行期间，HDFS 的所有更新操作都是直接写到 Edit Log 文件中，一段时间之后，Edit Log 文件会变得很大。虽然这对 NameNode 运行没有什么明显影响，但是，当 NameNode 重启时，NameNode 需要先将 FsImage 里面的所有内容加载到内存上，然后还要一条一条地加载 Edit Log 中的记录。当 Edit Log 文件非常大的时候，就会导致 NameNode 启动过程非常慢，这是生产环境不能容忍的情况。</p>
<p>通过检查点机制可以定期将 Edit Log 合并到 Fsimage 中生成新的 Fsimage。检查点在 NameNode 启动时会触发，除此之外，还有其他几个触发条件：</p>
<ul>
<li>两次 checkpoint 的时间间隔达到阈值，此属性由 dfs.namenode.checkpoint.period 控制，默认是 3600s，也就是一小时触发一次；</li>
<li>新生成的 Edit Log 中积累的事务数量达到了阈值，此属性由 dfs.namenode.checkpoint.txns 控制，默认 1000000，也就是 HDFS 经过 100 万次操作后就要进行 checkpoint 了。</li>
</ul>
<p>这两个参数任意一个得到满足，都会触发 checkpoint。</p>
<p>由于 checkpoint 的过程需要消耗大量的 IO 和 CPU 资源，并且会阻塞 HDFS 的读写操作。所以，该过程不会在 NameNode 节点上触发，在 Hadoop1.x 中由 SecondaryNameNode 来完成。而在 HDFS 的 HA 模式下，checkpoint 则由 StandBy 状态的 NameNode 来实现。</p>
<h3>SecondaryNameNode 工作机制剖析</h3>
<p>SecondaryNameNode 实现有两个功能，即对 NameNode 元数据的备份以及将 Edit Log 合并到 Fsimage 中，这种实现机制比较简单，下面通过一张图来了解这种实现机制，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/0D/Ciqc1F7U4FSAZPkNAAG0gIoCEHE433.png" alt="1.png"></p>
<p>由图可知，SecondaryNameNode 实现对元数据备份，主要分为 5 个步骤：</p>
<ol>
<li>SecondaryNameNode 节点会定期和 NameNode 通信，请求其停止使用 Edit Log，暂时将新的写操作转到一个新的文件 edit.new 上来，这个操作是瞬间完成的；</li>
<li>SecondaryNameNode 通过 HTTP Get 方式从 NameNode 上获取 FsImage 和 Edit Log 文件，并下载到本地目录；</li>
<li>将下载下来的 FsImage 和 Edit Log 加载到内存中，该过程就是 FsImage 和 Edit Log 的合并，合并后会产生一个新的 FsImage 文件，这个过程就是 checkpoint；</li>
<li>合并成功之后，会通过 post 方式将新的 FsImage 文件发送到 NameNode 上；</li>
<li>NameNode 会将新接收到的 FsImage 替换掉旧的，同时用 edit.new 替换 Edit Log，这样 Edit Log 就会变小。</li>
</ol>
<p>SecondaryNameNode 总是周期性的去检查是否符合触发 checkpoint 的条件，检查周期是由 dfs.NameNode.checkpoint.check.period 控制，默认是 60s。如果符合触发条件，那么就执行 checkpoint 操作并合并 Edit Log 到 Fsimage，将合并后的 Fsimage 传到 NameNode 上。</p>
<p>生产环境下，SecondaryNameNode 建议运行在一台独立的服务器上，因为它在执行 FsImage 和 Edit Log 合并过程需要耗费大量 IO 和内存资源。不过，HDFS 支持 HA 功能后，SecondaryNameNode 已经很少使用了。</p>
<h3>NameNode 下的元数据存储</h3>
<p>元数据在 NameNode 中有 3 种存储形式，即内存、Edit Log 文件和 Fsimage 文件。最完整且最新的元数据一定是内存中的这一部分，理解这一点非常重要。那么元数据的目录结构是什么样的呢，打开元数据目录（我这里是 /data1/hadoop/dfs/name/current），可以看到文件或目录，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/15/D2/CgqCHl7UqTmAazCJAACxChYVipc532.png" alt="image3.png"></p>
<p>由图可知，有大量以 edits_ 开头的文件，这些就是 Edit Log 文件。记录在 Edit Log 之中的每一个操作又称为一个事务，每个事务都有一个整数形式的事务 id 作为编号。这些 edits_ 开头的文件名类似 edits_${start_txid}-${end_txid}，并且多个 Edit Log 之间通过 txid 首尾相连。</p>
<p>其中，${start_txid} 表示 edits 的起始事务 id，而 ${end_txid} 表示 edits 的结束事务 id。正在写入的 edits 文件处于 inprogress 状态，其文件名为 edits_inprogress_${start_txid}，已经写入完成的 edits 处于 finalized 状态，名字就变成了 edits_${start_txid}-${end_txid}。</p>
<p>接着，还有两个 Fsimage 文件，该文件的名称类似 fsimage_${end_txid}，此文件在 NameNode 上保存的数目由 dfs.namenode.num.checkpoints.retained 参数控制，超出的会被删除，默认保存 2 个。历史 Fsimage 文件通常只会在元数据损坏的时候用来做恢复用，适当保留几份就够了，太多了不仅没用，反而浪费磁盘空间。</p>
<p>在上图中，可以看到 fsimage 已经到了 fsimage_0000000000000080693，而目前最新的 Edit Log 是 edits_inprogress_0000000000000080712。因此，在启动 NameNode 时，只需要读入从 edits_0000000000000080694 到 edits_inprogress_0000000000000080712 这个区间的数据，就可以完整恢复出 HDFS 的所有数据。</p>
<p>最后，还有两个文件，一个是 VERSION 文件，此文件是一个 Java 格式文件，保存了 HDFS 的版本号信息；另一个是 seen_txid 文件，此文件非常重要，它记录了最后一次 checkpoint 或者 edit 回滚（将 edits_inprogress_xxx 文件回滚成一个新的 Edit Log 文件）之后的 transaction ID，它主要用来检查 NameNode 启动过程中 Edit Log 文件是否有丢失的情况。</p>
<p>上面说到了 edit 文件可以回滚，回滚的意思是生成一个新的文件。</p>
<h3>StandbyNameNode 下 JournalNode 的元数据管理</h3>
<p>JournalNode 只在 HDFS 的 HA 模式下出现，其作为一个守护进程，实现了元数据在主、备节点的共享。JournalNode 的元数据目录在 hdfs-site.xml 文件中通过 dfs.journalnode.edits.dir 参数指定，包含一个 VERSION 文件，多个 edits_xx 文件和一个 edits_inprogress_xxx 文件，还包含一些与 HA 实现相关的文件，这些文件主要是为了防止脑裂，但 JournalNode 中并不包含 fsimage 和 seen_txid 文件。</p>
<p>在 HA 模式下，StandbyNameNode 之所以会周期地让主 NameNode 对 Edit Log 文件进行回滚，间隔周期由 dfs.ha.log-roll.period 指定，默认是 120s，是因为 StandbyNameNode 不会读取 inprogress 的 Edit Log 文件，它只会周期性（dfs.ha.tail-edits.period，默认是 60s）的去检测已经完成的 Edit Log 文件，然后将该 Edit Log 文件通过 JournalNode 读取到内存，进而更新 Fsimage 在内存中的状态。在达到 checkpoint 触发条件后，新的 Fsimage 文件会在 StandbyNameNode 上生成。接着，StandbyNameNode 会删除自己磁盘上保留的陈旧 Fsimage 文件，然后将新的 Fsimage 文件上传给主 NameNode。</p>
<p>注意，在 HA 模式下，客户端请求只能和主 NameNode 进行通信，而主 NameNode 会将写操作信息分别写入本地的 Edit Log 文件中和 JournalNode 中的 Edit Log 文件。StandbyNameNode 只会周期性的去 JournalNode 中读取 Edit Log 文件进行同步，并将生成的新 Fsimage 文件传送给主 NameNode。通过这种机制，可以几乎实时的实现元数据的同步，当主 NameNode 发生宕机后，StandbyNameNode 自动切换为 Active 状态，开始对外提供读、写服务。</p>
<p>你可能发现了，在主 NameNode 元数据目录下会有非常多的 Edit Log 文件，而在 StandbyNameNode 元数据目录下只有一个 edits_inprogress_xxx 文件，这是因为 StandbyNameNode 并不会从 JournalNode 上拉取 Edit Log 文件保存到磁盘。而主 NameNode 下 Edit Log 文件每两分钟（由 dfs.ha.log-roll.period 控制）生成一个，所以我们需要定期清理陈旧的 Edit Log 文件。</p>
<h3>HDFS 读取、写入数据流程解析</h3>
<p>在使用 HDFS 之前，我们需要了解它的几个特点：</p>
<ul>
<li>一次写入，多次读取（不可修改，只可追加）；</li>
<li>文件由<strong>数据块</strong>（Block）组成，数据块大小默认 128MB，若文件不足 128M，则也会单独存成一个块，一个块只能存一个文件的数据，即使一个文件不足 128M，也会占用一个块，块是一个逻辑空间，并不会占磁盘空间；</li>
<li>默认情况下每个块都有三个副本，三个副本会存储到不同的节点上，副本越多，磁盘利用率越低，但数据的安全性越高，可以通过修改 hdfs-site.xml 的 dfs.replication 属性设置副本的个数；</li>
<li>文件按大小被切分成若干个块，存储到不同的节点上，块大小可通过配置文件进行配置或修改。</li>
</ul>
<p>HDFS 文件名也有对应的格式，随便登录一个 DataNode 节点，找到数据块存储路径，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/15/C7/Ciqc1F7UqUOAMiyxAAB6XluvVzs412.png" alt="image4.png"></p>
<p>可以看出，HDFS 数据块文件名组成格式为：</p>
<ul>
<li>blk_，HDFS 的数据块，保存具体的二进制数据；</li>
<li>blk_.meta，数据块的属性信息，比如版本信息、类型信息。</li>
</ul>
<p>这些数据块非常重要，要确保它们的安全。</p>
<p>介绍了 HDFS 的数据块结构后，下面介绍下 HDFS 是如何读取数据的。下图是 HDFS 读取数据的一个流程图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/19/CgqCHl7U4LaAYHtjAAHdezX8IO8904.png" alt="4.png"></p>
<p>从上图可以看出，HDFS 读取文件基本分为 5 个步骤，每个步骤含义如下：</p>
<ol>
<li>客户端向 NameNode 请求读取一个文件，NameNode 通过查询元数据，找到请求文件对应的数据块所在的位置，也就是块文件对应的 DataNode 节点地址；</li>
<li>NameNode 返回自己查询到的元数据信息给客户端；</li>
<li>客户端挑选一台 DataNode（根据就近原则，然后随机原则）服务器，开始请求读取数据；</li>
<li>DataNode 开始传输数据给客户端（从磁盘里面读取数据放入流，以 packet 为单位来做校验）；</li>
<li>客户端以 packet 为单位接收，先在本地缓存，然后写入目标文件。</li>
</ol>
<p>可以看出，HDFS 读取文件的流程非常简单。接着，再来看看 HDFS 是如何写入数据的，这个过程稍微复杂，如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/16/18/CgqCHl7U4IWAbSbsAAIQMoyHI-s874.png" alt="3.png"></p>
<p>从上图可以看出，HDFS 写入文件基本分为 7 个步骤，每个步骤含义如下所示。</p>
<ol>
<li>首先，客户端对 NameNode 发起上传文件的请求。NameNode 接到请求后，马上检查请求文件是否已存在，上级目录是否存在。</li>
<li>NameNode 检查发现，请求的文件如果不存在，就响应客户端请求，可以上传文件。</li>
<li>Client 会首先对文件进行切分。假定 HDFS 的一个 blok 大小是 128M，如果写入的文件大小有 300M，那么此文件就会被切分成 3 个块，分别是两个 128M 和一个 44M。接着，客户端向 NameNode 发起请求第一个 block 该传输到哪些 DataNode 服务器上。</li>
<li>NameNode 返回信息给 Client，告知可以上传到哪些 DataNode 服务节点上。这里假定有三个副本，所以一个块需要上传到三个节点上，这里设定上传到 A、B、C 三个 DataNode 节点。</li>
<li>Client 开始和 DataNode 建立传输通道，首先请求 DataNode A 节点上传数据（本质上是一个 RPC 调用，建立 pipeline），DataNode A 收到请求后，会继续调用 DataNode B；然后 DataNode B 再去调用第三个 DataNode C，将整个 pipeline 建立完成，逐级返回 Client。</li>
<li>Client 开始往 A 上传第一个 block（先从磁盘读取数据放到一个本地内存缓存），以 packet 为单位（一个 packet 为 64KB），DataNode A 收到一个 packet 就会传给 DataNode B，DataNode B 接着传给 DataNode C；DataNode A 每传一个 packet 会放入一个应答队列，等待应答。</li>
<li>当一个 block 传输完成之后，Client 再次请求 NameNode 开始上传第二个 block，上传过程重复上面 4~6 步骤。</li>
</ol>
<p>这样，就完成了 HDFS 写入数据的一个过程。</p>
<h3>HDFS 中的 Shell 操作</h3>
<p>HDFS 上的 Shell 操作命令跟 Linux 下的 Shell 命令基本类似，相关参数也基本相同，所以对 HDFS 上的文件进行操作非常简单，常见的操作有 6 类，即查看、增加、移动、删除、追加、授权。下面分别做简单介绍。</p>
<p>（1）查看 HDFS 文件系统上的文件</p>
<p>要查看 HDFS 上的文件，可以通过如下几个命令实现：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -ls /</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -ls /user</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -cat  /logs/demo10.txt </span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -text  /logs/demo10.tar.gz #通过 -text 可查看压缩文件的内容，支持场景的 zip、gzip、bzip2 等压缩格式</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -du -s /tez #查看/ tez 目录所占空间大小</span>
</code></pre>
<p>（2）增加文件</p>
<p>要添加文件到 HDFS 上，可以通过如下几个命令实现：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -touch   /logs/demo101.txt</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -mkdir   /logs/demo </span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -put /local/demo10.tar.gz   /logs/demo #这里的 /local/demo10.tar.gz 是本地路径</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -copyFromLocal  /localdemo10.tar.gz   /logs/demo #这个实现功能同 put 参数，都是上传本地文件到 HDFS </span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -get /logs/demo/demo10.tar.gz /home/hadoop  #这里的 /home/hadoop 是本地路径</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -copyToLocal /logs/demo/demo10.tar.gz /home/hadoop #这个实现功能同 get 参数，都是下载 HDFS 文件到本地系统磁盘 </span>
hadoop fs -cp /logs/demo/demo10.tar.gz /tmp <span class="hljs-comment">#这个命令中两个路径均为 HDFS 上路径</span>
</code></pre>
<p>（3）移动文件</p>
<p>移动文件可以实现本地磁盘文件移动到 HDFS，HDFS 上各个路径之间的移动、文件改名等操作。看下面两个例子：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -mv /logs/demo/demo10.tar.gz /tmp #这个命令中两个路径均为 HDFS 上路径</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -moveFromLocal /tmp/spark-2.4.5-bin-hadoop2.7.tgz  /logs/demo/ #这个命令中 /tmp/spark-2.4.5-bin-hadoop2.7.tgz 路径为本地路径</span>
</code></pre>
<p>（4）删除文件</p>
<p>无论是文件或者目录，都可以通过如下命令进行删除：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -rm -r  /logs/demo/demo10.tar.gz</span>
</code></pre>
<p>删除文件要特别小心，为了避免误删除，HDFS 上提供了回收站的功能。这样文件删除后，会进入回收站，在超过回收周期后，自动删除。你可以在 core-site.xml 文件中增加 fs.trash.interval 参数，表示进入回收站中的文件多少分钟后会被系统永久删除，可以根据情况设置一个合适的时间周期。<br>
如果你确定某个文件可以删除，不需要进入回收站的话，可以执行如下命令：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -rm -r -skipTrash /tmp/hive_test</span>
</code></pre>
<p>要清空回收站，可执行如下命令：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -expunge</span>
</code></pre>
<p>（5）追加文件内容</p>
<p>HDFS 上的文件不能修改，但可以在文件最后追加内容，可通过如下命令实现：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -appendToFile  aa1.log /logs/aa.log</span>
</code></pre>
<p>这个意思是将本地 aa1.log 文件的内容追加到 HDFS 的 /logs/aa.log 文件的最后。</p>
<p>（6）对文件进行授权</p>
<p>对文件或目录进行授权，主要使用了 Linux 上的两个命令 chown 和 chmod。HDFS 上授权文件的用法如下：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -chown -R hadoop:hadoop /logs/aa.log</span>
[root@yarnserver ~]<span class="hljs-comment"># hadoop fs -chmod 744 /logs/aa.log</span>
</code></pre>
<p>注意，这里的参数含义跟 Linux 命令参数含义相同。</p>
<h3>总结</h3>
<p>本课时主要讲解了 HDFS 的基本架构，以及 Namenode 和 SecondaryNameNode 的工作机制、Namenode 下元数据的管理，以及 HDFS 读、写数据的流程。理解这些原理和架构对于我们排查 Hadoop 集群故障和调优有很大帮助。</p>

---

### 精选评论

##### *科：
> namenode HA模式下，备用namenode不会去拉起fsimage文件。如果新扩容的备用namenode，第一次启用的fsimage怎么来的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第一次启用备用nn，执行一个命令初始化一下即可，拉取元数据是从jn里面来拉取的，启用备用nn后，会自动拉取元数据，并实现实时同步。

##### **宇：
> Namenode挂掉了SecondaryNameNode只是辅助Namenode帮助其恢复数据而不是直接替代它。如果Namenode起不来之后会怎么样？

##### **9134：
> 讲的真好，对hdfs运行机制，组件都有了深刻认识，就差实践了，谢谢😀

##### **用户7103：
> 我看我这面创建文件的命令是 touchz， 是hadoop版本不一样吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; touch和touchz都可以的

##### **波：
> 非常详细

