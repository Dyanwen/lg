<p data-nodeid="29063">之前的课程我们都在介绍 ZooKeeper 框架能够实现的功能，而无论是什么程序，其本质就是对数据的操作。比如 MySQl 数据库操作的是数据表，Redis 数据库操作的是存储在内存中的 Key-Value 值。不同的数据格式和存储方式对系统运行的效率和处理能力都有很大影响。本课时就来学习，在 ZooKeeper 程序运行期间，都会处理哪些数据，以及他们的存储格式和存储位置。</p>



<p data-nodeid="28178">ZooKeeper 服务提供了创建节点、添加 Watcher 监控机制、集群服务等丰富的功能。这些功能服务的实现，离不开底层数据的支持。从数据存储地点角度讲，ZooKeeper 服务产生的数据可以分为内存数据和磁盘数据。而从数据的种类和作用上来说，又可以分为事务日志数据和数据快照数据。</p>
<h3 data-nodeid="28179">内存数据</h3>
<p data-nodeid="28180">首先，我们介绍一下什么是内存数据。在专栏的基础篇中，主要讲解了通过 ZooKeeper 数据节点的特性，来实现一些像发布订阅这样的功能。而这些数据节点实际上就是 ZooKeeper 在服务运行过程中所操作的数据。</p>
<p data-nodeid="28181">我在基础篇中提到过，ZooKeeper 的数据模型可以看作一棵<strong data-nodeid="28225">树形结构</strong>，而数据节点就是这棵树上的叶子节点。从数据存储的角度看，ZooKeeper 的数据模型是存储在内存中的。我们可以把 ZooKeeper 的数据模型看作是存储在内存中的数据库，而这个数据库不但存储数据的节点信息，还存储每个数据节点的 ACL 权限信息以及 stat 状态信息等。</p>
<p data-nodeid="28182">而在底层实现中，ZooKeeper  数据模型是通过 DataTree 类来定义的。如下面的代码所示，DataTree 类定义了一个 ZooKeeper 数据的内存结构。DataTree 的内部定义类 nodes 节点类型、root 根节点信息、子节点的 WatchManager 监控信息等数据模型中的相关信息。可以说，一个 DataTree 类定义了 ZooKeeper 内存数据的逻辑结构。</p>
<pre class="lang-java" data-nodeid="29345"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataTree</span> </span>{
  <span class="hljs-keyword">private</span> DataNode root
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> WatchManager dataWatches
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> WatchManager childWatches
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String rootZookeeper = <span class="hljs-string">"/"</span>;
}
</code></pre>


<h3 data-nodeid="28184">事务日志</h3>
<p data-nodeid="28185">在介绍 ZooKeeper 集群服务的时候，我们介绍过，为了整个 ZooKeeper 集群中数据的一致性，Leader 服务器会向 ZooKeeper 集群中的其他角色服务发送数据同步信息，在接收到数据同步信息后， ZooKeeper 集群中的 Follow 和 Observer 服务器就会进行数据同步。而这两种角色服务器所接收到的信息就是 Leader 服务器的事务日志。在接收到事务日志后，并在本地服务器上执行。这种数据同步的方式，避免了直接使用实际的业务数据，减少了网络传输的开销，提升了整个 ZooKeeper 集群的执行性能。</p>
<p data-nodeid="28186">在我们启动一个 ZooKeeper 服务器之前，首先要创建一个 zoo.cfg 文件并进行相关配置，其中有一项配置就是 dataLogDir 。在这项配置中，我们会指定该台 ZooKeeper 服务器事务日志的存放位置。</p>
<p data-nodeid="28187">在 ZooKeeper 服务的底层实现中，是通过 FileTxnLog 类来实现事务日志的底层操作的。如下图代码所示，在 FileTxnLog 类中定义了一些属性字段，分别是：</p>
<ul data-nodeid="29897">
<li data-nodeid="29898">
<p data-nodeid="29899">preAllocSize：可存储的日志文件大小。如用户不进行特殊设置，默认的大小为 65536*1024 字节。</p>
</li>
<li data-nodeid="29900">
<p data-nodeid="29901">TXNLOG_MAGIC：设置日志文件的魔数信息为ZKLG。</p>
</li>
<li data-nodeid="29902">
<p data-nodeid="29903">VERSION：设置日志文件的版本信息。</p>
</li>
<li data-nodeid="29904">
<p data-nodeid="29905">lastZxidSeen：最后一次更新日志得到的 ZXID。</p>
</li>
</ul>
<p data-nodeid="29906" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/DF/Ciqc1F8IC-uAcS1bAABJoZ4awKg473.png" alt="image (11).png" data-nodeid="29921"></p>


<p data-nodeid="28198">定义了事务日志操作的相关指标参数后，在 FileTxnLog 类中调用 static 静态代码块，来将这些配置参数进行初始化。比如读取 preAllocSize 参数分配给日志文件的空间大小等操作。</p>
<pre class="lang-java" data-nodeid="29532"><code data-language="java"><span class="hljs-keyword">static</span> {
    LOG = LoggerFactory.getLogger(FileTxnLog.class);
    String size = System.getProperty("zookeeper.preAllocSize");
    <span class="hljs-keyword">if</span> (size != <span class="hljs-keyword">null</span>) {
        <span class="hljs-keyword">try</span> {
            preAllocSize = Long.parseLong(size) * <span class="hljs-number">1024</span>;
        } <span class="hljs-keyword">catch</span> (NumberFormatException e) {
            LOG.warn(size + <span class="hljs-string">" is not a valid value for preAllocSize"</span>);
        }
    }
    Long fsyncWarningThreshold;
    <span class="hljs-keyword">if</span> ((fsyncWarningThreshold = Long.getLong(<span class="hljs-string">"zookeeper.fsync.warningthresholdms"</span>)) == <span class="hljs-keyword">null</span>)
        fsyncWarningThreshold = Long.getLong(<span class="hljs-string">"fsync.warningthresholdms"</span>, <span class="hljs-number">1000</span>);
    fsyncWarningThresholdMS = fsyncWarningThreshold;
</code></pre>

<p data-nodeid="28200">经过参数定义和日志文件的初始化创建后，在 ZooKeeper  服务器的 dataDir 路径下就生成了一个用于存储事务性操作的日志文件。我们知道在 ZooKeeper 服务运行过程中，会不断地接收和处理来自客户端的事务性会话请求，这就要求每次在处理事务性请求的时候，都要记录这些信息到事务日志中。</p>
<p data-nodeid="28201">如下面的代码所示，在 FileTxnLog 类中，实现记录事务操作的核心方法是 append。从方法的命名中可以看出，ZooKeeper 采用末尾追加的方式来维护新的事务日志数据到日志文件中。append 方法首先会解析事务请求的头信息，并根据解析出来的 zxid 字段作为事务日志的文件名，之后设置日志的文件头信息 magic、version、dbid 以及日志文件的大小 。</p>
<pre class="lang-java" data-nodeid="30116"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">append</span><span class="hljs-params">(TxnHeader hdr, Record txn)</span>
    <span class="hljs-keyword">throws</span> IOException
</span>{
    <span class="hljs-keyword">if</span> (hdr == <span class="hljs-keyword">null</span>) {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
    }
    <span class="hljs-keyword">if</span> (hdr.getZxid() &lt;= lastZxidSeen) {
        LOG.warn(<span class="hljs-string">"Current zxid "</span> + hdr.getZxid()
                + <span class="hljs-string">" is &lt;= "</span> + lastZxidSeen + <span class="hljs-string">" for "</span>
                + hdr.getType());
    } <span class="hljs-keyword">else</span> {
        lastZxidSeen = hdr.getZxid();
    }
    <span class="hljs-keyword">if</span> (logStream==<span class="hljs-keyword">null</span>) {
       <span class="hljs-keyword">if</span>(LOG.isInfoEnabled()){
            LOG.info(<span class="hljs-string">"Creating new log file: log."</span> +
                    Long.toHexString(hdr.getZxid()));
       }
       logFileWrite = <span class="hljs-keyword">new</span> File(logDir, (<span class="hljs-string">"log."</span> +
               Long.toHexString(hdr.getZxid())));
       fos = <span class="hljs-keyword">new</span> FileOutputStream(logFileWrite);
       logStream=<span class="hljs-keyword">new</span> BufferedOutputStream(fos);
       oa = BinaryOutputArchive.getArchive(logStream);
       FileHeader fhdr = <span class="hljs-keyword">new</span> FileHeader(TXNLOG_MAGIC,VERSION, dbId);
       fhdr.serialize(oa, <span class="hljs-string">"fileheader"</span>);
       <span class="hljs-comment">// Make sure that the magic number is written before padding.</span>
       logStream.flush();
       currentSize = fos.getChannel().position();
       streamsToFlush.add(fos);
    }
    padFile(fos);
    <span class="hljs-keyword">byte</span>[] buf = Util.marshallTxnEntry(hdr, txn);
    <span class="hljs-keyword">if</span> (buf == <span class="hljs-keyword">null</span> || buf.length == <span class="hljs-number">0</span>) {
        <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IOException(<span class="hljs-string">"Faulty serialization for header "</span> +
                <span class="hljs-string">"and txn"</span>);
    }
    Checksum crc = makeChecksumAlgorithm();
    crc.update(buf, <span class="hljs-number">0</span>, buf.length);
    oa.writeLong(crc.getValue(), <span class="hljs-string">"txnEntryCRC"</span>);
    Util.writeTxnBytes(oa, buf);
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
</code></pre>

<p data-nodeid="28203">从对事务日志的底底层代码分析中可以看出，在 datadir 配置参数路径下存放着 ZooKeeper 服务器所有的事务日志，所有事务日志的命名方法都是“log.+ 该条事务会话的 zxid”。</p>
<h3 data-nodeid="28204">数据快照</h3>
<p data-nodeid="28205">最后，我们来介绍 ZooKeeper 服务运行过程中产生的最后一个数据文件，即事务快照。</p>
<p data-nodeid="28206">说到快照，可能很多技术人员都不陌生。一个快照可以看作是当前系统或软件服务运行状态和数据的副本。在 ZooKeeper 中，数据快照的作用是将内存数据结构存储到本地磁盘中。因此，从设计的角度说，数据快照与内存数据的逻辑结构一样，都使用 DataTree 结构。在 ZooKeeper 服务运行的过程中，数据快照每间隔一段时间，就会把 ZooKeeper 内存中的数据存储到磁盘中，快照文件是间隔一段时间后对内存数据的备份。</p>
<p data-nodeid="28207">因此，与内存数据相比，快照文件的数据具有滞后性。而与上面介绍的事务日志文件一样，在创建数据快照文件时，也是使用 zxid 作为文件名称。</p>
<p data-nodeid="28208">在代码层面，ZooKeeper 通过 FileTxnSnapLog 类来实现数据快照的相关功能。如下图所示，在FileTxnSnapLog 类的内部，最核心的方法是 save 方法，在 save 方法的内部，首先会创建数据快照文件，之后调用 FileSnap 类对内存数据进行序列化，并写入到快照文件中。</p>
<pre class="lang-java" data-nodeid="30311"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">save</span><span class="hljs-params">(DataTree dataTree,
                 ConcurrentHashMap&lt;Long, Integer&gt; sessionsWithTimeouts,
                 <span class="hljs-keyword">boolean</span> syncSnap)</span>
    <span class="hljs-keyword">throws</span> IOException </span>{
    <span class="hljs-keyword">long</span> lastZxid = dataTree.lastProcessedZxid;
    File snapshotFile = <span class="hljs-keyword">new</span> File(snapDir, Util.makeSnapshotName(lastZxid));
    LOG.info(<span class="hljs-string">"Snapshotting: 0x{} to {}"</span>, Long.toHexString(lastZxid),
            snapshotFile);
    snapLog.serialize(dataTree, sessionsWithTimeouts, snapshotFile, syncSnap);
}
</code></pre>

<h3 data-nodeid="28210">总结</h3>
<p data-nodeid="30719" class="te-preview-highlight">通过本课时的学习，我们知道在 ZooKeeper 服务的运行过程中，<strong data-nodeid="30733">会涉及内存数据</strong>、<strong data-nodeid="30734">事务日志</strong>、<strong data-nodeid="30735">数据快照这三种数据文件</strong>。从存储位置上来说，事务日志和数据快照一样，都存储在本地磁盘上；而从业务角度来讲，内存数据就是我们创建数据节点、添加监控等请求时直接操作的数据。事务日志数据主要用于记录本地事务性会话操作，用于 ZooKeeper 集群服务器之间的数据同步。事务快照则是将内存数据持久化到本地磁盘。</p>


<p data-nodeid="28862" class="">这里要注意的一点是，<strong data-nodeid="28868">数据快照是每间隔一段时间才把内存数据存储到本地磁盘，因此数据并不会一直与内存数据保持一致</strong>。在单台 ZooKeeper 服务器运行过程中因为异常而关闭时，可能会出现数据丢失等情况。</p>

---

### 精选评论


