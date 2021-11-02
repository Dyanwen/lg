<p data-nodeid="174344" class="">在整个高级篇中，我们主要介绍了 ZooKeeper 服务器以及集群的工作原理等相关知识。本课时我们仍然继续上节课的内容，把 Leader 服务器的另一个关键技术点：“Leader 服务器是如何产生的”进行详细讲解。</p>
<p data-nodeid="174345">下面我们就深入到 ZooKeeper 的底层，来学习一下 Leader 服务器选举的实现方法。</p>
<h3 data-nodeid="174346">Leader 服务器的选举原理</h3>
<p data-nodeid="174347">Leader 服务器的作用是管理 ZooKeeper 集群中的其他服务器。因此，如果是单独一台服务器，不构成集群规模。在 ZooKeeper 服务的运行中不会选举 Leader 服务器，也不会作为 Leader 服务器运行。在前面的课程中我们介绍过，一个 ZooKeeper 服务要想满足集群方式运行，至少需要三台服务器。本课时我们就以三台服务器组成的 ZooKeeper 集群为例，介绍一下 Leader 服务器选举的内部过程和底层实现。</p>
<h4 data-nodeid="174348">服务启动时的 Leader 选举</h4>
<p data-nodeid="174349">Leader 服务器的选举操作主要发生在两种情况下。第一种就是 ZooKeeper 集群服务启动的时候，第二种就是在 ZooKeeper 集群中旧的 Leader 服务器失效时，这时 ZooKeeper 集群需要选举出新的 Leader 服务器。</p>
<p data-nodeid="174350">我们先来介绍在 ZooKeeper 集群服务最初启动的时候，Leader 服务器是如何选举的。在 ZooKeeper 集群启动时，需要在集群中的服务器之间确定一台 Leader 服务器。当 ZooKeeper 集群中的三台服务器启动之后，首先会进行通信检查，如果集群中的服务器之间能够进行通信。集群中的三台机器开始尝试寻找集群中的 Leader 服务器并进行数据同步等操作。如何这时没有搜索到 Leader 服务器，说明集群中不存在 Leader 服务器。这时 ZooKeeper 集群开始发起 Leader 服务器选举。在整个 ZooKeeper 集群中 Leader 选举主要可以分为三大步骤分别是：发起投票、接收投票、统计投票。</p>
<p data-nodeid="174351"><img src="https://s0.lgstatic.com/i/image/M00/26/E5/Ciqc1F7zKQyAK4wsAADnGMCxArI126.png" alt="2.png" data-nodeid="174395"></p>
<h4 data-nodeid="174352">发起投票</h4>
<p data-nodeid="174353">我们先来看一下发起投票的流程，在 ZooKeeper 服务器集群初始化启动的时候，集群中的每一台服务器都会将自己作为 Leader 服务器进行投票。<strong data-nodeid="174402">也就是每次投票时，发送的服务器的 myid（服务器标识符）和 ZXID (集群投票信息标识符)等选票信息字段都指向本机服务器。</strong> 而一个投票信息就是通过这两个字段组成的。以集群中三个服务器 Serverhost1、Serverhost2、Serverhost3 为例，三个服务器的投票内容分别是：Severhost1 的投票是（1，0）、Serverhost2 服务器的投票是（2，0）、Serverhost3 服务器的投票是（3，0）。</p>
<h4 data-nodeid="174354">接收投票</h4>
<p data-nodeid="174355">集群中各个服务器在发起投票的同时，也通过网络接收来自集群中其他服务器的投票信息。</p>
<p data-nodeid="174356">在接收到网络中的投票信息后，服务器内部首先会判断该条投票信息的有效性。检查该条投票信息的时效性，是否是本轮最新的投票，并检查该条投票信息是否是处于 LOOKING 状态的服务器发出的。</p>
<h4 data-nodeid="174357">统计投票</h4>
<p data-nodeid="174358">在接收到投票后，ZooKeeper 集群就该处理和统计投票结果了。对于每条接收到的投票信息，集群中的每一台服务器都会将自己的投票信息与其接收到的 ZooKeeper 集群中的其他投票信息进行对比。主要进行对比的内容是 ZXID，ZXID 数值比较大的投票信息优先作为 Leader 服务器。如果每个投票信息中的 ZXID 相同，就会接着比对投票信息中的 myid 信息字段，选举出 myid 较大的服务器作为 Leader 服务器。</p>
<p data-nodeid="174359">拿上面列举的三个服务器组成的集群例子来说，对于 Serverhost1，服务器的投票信息是（1，0），该服务器接收到的 Serverhost2 服务器的投票信息是（2，0）。在 ZooKeeper 集群服务运行的过程中，首先会对比 ZXID，发现结果相同之后，对比 myid，发现 Serverhost2 服务器的 myid 比较大，于是更新自己的投票信息为（2，0），并重新向 ZooKeeper 集群中的服务器发送新的投票信息。而 Serverhost2 服务器则保留自身的投票信息，并重新向 ZooKeeper 集群服务器中发送投票信息。</p>
<p data-nodeid="174360">而当每轮投票过后，ZooKeeper 服务都会统计集群中服务器的投票结果，判断是否有过半数的机器投出一样的信息。如果存在过半数投票信息指向的服务器，那么该台服务器就被选举为 Leader 服务器。比如上面我们举的例子中，ZooKeeper 集群会选举 Severhost2 服务器作为 Leader 服务器。</p>
<p data-nodeid="174361"><img src="https://s0.lgstatic.com/i/image/M00/26/F1/CgqCHl7zKRuARwOdAACqX-dZDEQ790.png" alt="1.png" data-nodeid="174412"></p>
<p data-nodeid="174362">当 ZooKeeper 集群选举出 Leader 服务器后，ZooKeeper 集群中的服务器就开始更新自己的角色信息，<strong data-nodeid="174417">除被选举成 Leader 的服务器之外，其他集群中的服务器角色变更为 Following。</strong></p>
<h4 data-nodeid="174363">服务运行时的 Leader 选举</h4>
<p data-nodeid="174364">上面我们介绍了 ZooKeeper 集群启动时 Leader 服务器的选举方法。接下来我们再看一下在 ZooKeeper 集群服务的运行过程中，Leader 服务器是如果进行选举的。</p>
<p data-nodeid="174365">在 ZooKeeper 集群服务的运行过程中，Leader 服务器作为处理事物性请求以及管理其他角色服务器，在 ZooKeeper 集群中起到关键的作用。在前面的课程中我们提到过，当 ZooKeeper 集群中的 Leader 服务器发生崩溃时，集群会暂停处理事务性的会话请求，直到 ZooKeeper 集群中选举出新的 Leader 服务器。而整个 ZooKeeper 集群在重新选举 Leader 时也经过了四个过程，分别是变更服务器状态、发起投票、接收投票、统计投票。其中，与初始化启动时 Leader 服务器的选举过程相比，变更状态和发起投票这两个阶段的实现是不同的。下面我们来分别看看这两个阶段。</p>
<h4 data-nodeid="174366">变更状态</h4>
<p data-nodeid="174367">与上面介绍的 ZooKeeper 集群服务器初始化阶段不同。在 ZooKeeper 集群服务运行的过程中，集群中每台服务器的角色已经确定了，当 Leader 服务器崩溃后 ，ZooKeeper 集群中的其他服务器会首先将自身的状态信息变为 LOOKING 状态，该状态表示服务器已经做好选举新 Leader 服务器的准备了，这之后整个 ZooKeeper 集群开始进入选举新的 Leader 服务器过程。</p>
<h4 data-nodeid="174368">发起投票</h4>
<p data-nodeid="174369">ZooKeeper 集群重新选举 Leader 服务器的过程中发起投票的过程与初始化启动时发起投票的过程基本相同。首先每个集群中的服务器都会投票给自己，将投票信息中的 Zxid 和 myid 分别指向本机服务器。</p>
<h3 data-nodeid="174370">底层实现</h3>
<p data-nodeid="174371">到目前为止，我们已经对 ZooKeeper 集群中 Leader 服务器的选举过程做了详细的介绍。接下来我们再深入 ZooKeeper 底层，来看一下底层实现的关键步骤。</p>
<p data-nodeid="174372">之前我们介绍过，ZooKeeper 中实现的选举算法有三种，而在目前的 ZooKeeper 3.6 版本后，只支持 “快速选举” 这一种算法。而在代码层面的实现中，QuorumCnxManager 作为核心的实现类，用来管理 Leader 服务器与 Follow 服务器的 TCP 通信，以及消息的接收与发送等功能。在 QuorumCnxManager 中，主要定义了 ConcurrentHashMap&lt;Long, SendWorker&gt; 类型的 senderWorkerMap 数据字段，用来管理每一个通信的服务器。</p>
<pre class="lang-java" data-nodeid="174373"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">QuorumCnxManager</span> </span>{
  <span class="hljs-keyword">final</span> ConcurrentHashMap&lt;Long, SendWorker&gt; senderWorkerMap;
<span class="hljs-keyword">final</span> ConcurrentHashMap&lt;Long, ArrayBlockingQueue&lt;ByteBuffer&gt;&gt; queueSendMap;
<span class="hljs-keyword">final</span> ConcurrentHashMap&lt;Long, ByteBuffer&gt; lastMessageSent;
}
</code></pre>
<p data-nodeid="174374">而在 QuorumCnxManager 类的内部，定义了 RecvWorker 内部类。该类继承了一个 ZooKeeperThread 类的多线程类。主要负责消息接收。在 ZooKeeper 的实现中，为每一个集群中的通信服务器都分配一个 RecvWorker，负责接收来自其他服务器发送的信息。在 RecvWorker 的 run 函数中，不断通过 queueSendMap 队列读取信息。</p>
<pre class="lang-java" data-nodeid="174438"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SendWorker</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ZooKeeperThread</span> </span>{
  Long sid;
  Socket sock;
  <span class="hljs-keyword">volatile</span> <span class="hljs-keyword">boolean</span> running = <span class="hljs-keyword">true</span>;
  DataInputStream din;
  <span class="hljs-keyword">final</span> SendWorker sw;
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
  threadCnt.incrementAndGet();
  <span class="hljs-keyword">while</span> (running &amp;&amp; !shutdown &amp;&amp; sock != <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">int</span> length = din.readInt();
    <span class="hljs-keyword">if</span> (length &lt;= <span class="hljs-number">0</span> || length &gt; PACKETMAXSIZE) {
        <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IOException(
                <span class="hljs-string">"Received packet with invalid packet: "</span>
                        + length);
    }
    
    <span class="hljs-keyword">byte</span>[] msgArray = <span class="hljs-keyword">new</span> <span class="hljs-keyword">byte</span>[length];
    din.readFully(msgArray, <span class="hljs-number">0</span>, length);
    ByteBuffer message = ByteBuffer.wrap(msgArray);
    addToRecvQueue(<span class="hljs-keyword">new</span> Message(message.duplicate(), sid));
  }
  
  }
}
</code></pre>

<p data-nodeid="174376">除了接收信息的功能外，QuorumCnxManager 内还定义了一个 SendWorker 内部类用来向集群中的其他服务器发送投票信息。如下面的代码所示。在 SendWorker 类中，不会立刻将投票信息发送到 ZooKeeper 集群中，而是将投票信息首先插入到 pollSendQueue 队列，之后通过 send 函数进行发送。</p>
<pre class="lang-java" data-nodeid="174377"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SendWorker</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ZooKeeperThread</span> </span>{
  Long sid;
  Socket sock;
  RecvWorker recvWorker;
  <span class="hljs-keyword">volatile</span> <span class="hljs-keyword">boolean</span> running = <span class="hljs-keyword">true</span>;
  DataOutputStream dout;
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">while</span> (running &amp;&amp; !shutdown &amp;&amp; sock != <span class="hljs-keyword">null</span>) {
    ByteBuffer b = <span class="hljs-keyword">null</span>;
    <span class="hljs-keyword">try</span> {
        ArrayBlockingQueue&lt;ByteBuffer&gt; bq = queueSendMap
                .get(sid);
        <span class="hljs-keyword">if</span> (bq != <span class="hljs-keyword">null</span>) {
            b = pollSendQueue(bq, <span class="hljs-number">1000</span>, TimeUnit.MILLISECONDS);
        } <span class="hljs-keyword">else</span> {
            LOG.error(<span class="hljs-string">"No queue of incoming messages for "</span> +
                      <span class="hljs-string">"server "</span> + sid);
            <span class="hljs-keyword">break</span>;
        }
        <span class="hljs-keyword">if</span>(b != <span class="hljs-keyword">null</span>){
            lastMessageSent.put(sid, b);
            send(b);
        }
    } <span class="hljs-keyword">catch</span> (InterruptedException e) {
        LOG.warn(<span class="hljs-string">"Interrupted while waiting for message on queue"</span>,
                e);
    }
}
  }
}
</code></pre>
<p data-nodeid="174378">实现了投票信息的发送与接收后，接下来我们就来看看如何处理投票结果。在 ZooKeeper 的底层，是通过 FastLeaderElection 类实现的。如下面的代码所示，在 FastLeaderElection 的内部，定义了最大通信间隔 maxNotificationInterval、服务器等待时间 finalizeWait 等属性配置。</p>
<pre class="lang-java" data-nodeid="174379"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FastLeaderElection</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Election</span> </span>{
  <span class="hljs-keyword">final</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> maxNotificationInterval = <span class="hljs-number">60000</span>;
&nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> IGNOREVALUE = -<span class="hljs-number">1</span>
&nbsp; QuorumCnxManager manager;
}
</code></pre>
<p data-nodeid="174380">在 ZooKeeper 底层通过 getVote 函数来设置本机的投票内容，如下图面的代码所示，在 getVote 中通过 proposedLeader 服务器信息、proposedZxid 服务器 ZXID、proposedEpoch 投票轮次等信息封装投票信息。</p>
<pre class="lang-java" data-nodeid="174381"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">synchronized</span> <span class="hljs-keyword">public</span> Vote <span class="hljs-title">getVote</span><span class="hljs-params">()</span></span>{
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Vote(proposedLeader, proposedZxid, proposedEpoch);
 }
</code></pre>
<p data-nodeid="174382">在完成投票信息的封装以及投票信息的接收和发送后。一个 ZooKeeper 集群中，Leader 服务器选举底层实现的关键步骤就已经介绍完了。 Leader 节点的底层实现过程的逻辑相对来说比较简单，基本分为封装投票信息、发送投票、接收投票等。</p>
<h3 data-nodeid="174383">结束</h3>
<p data-nodeid="174384">通过本课时的学习，我们就 ZooKeeper 服务端在集群环境下，如何选举出 Leader 服务器做了一个比较详细的介绍。我们知道 Leader 选举一般发生在 ZooKeeper 集群服务初始化和集群中旧的 Leader 服务器崩溃时。Leader 选举保证了 ZooKeeper 集群运行的可靠性。当旧的 Leader 服务器发生崩溃时，需要重新选举出新的 Leader 服务器以保证集群服务的稳定性。</p>
<p data-nodeid="174385" class="">在这个过程中我们思考一个问题，那就是之前崩溃的 Leader 服务器是否会参与本次投票，以及是否能被重新选举为 Leader 服务器。这主要取决于在选举过程中旧的 Leader 服务器的运行状态。如果该服务器可以正常运行且可以和集群中其他服务器通信，那么该服务器也会参与新的 Leader 服务器的选举，在满足条件的情况下该台服务器也会再次被选举为新的 Leader 服务器。</p>

---

### 精选评论

##### *潇：
> 在统计投票过程中，您说serverhost2作为leader服务器，还是没有太明白，serverhost3服务器在选举过程中没有参与进来吗，希望解答一下，这一块有点模糊，谢谢

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; Leader 选举投票采用轮次方式，就是一轮如果没有满足多数原则就会进行下一轮，serverhost3 也做为选举服务器
在接收到网络中其他服务器的投票信息后会进行对比，判断是否使用接收到的投票信息更新自己的投票，并发送最新的投票信息到
网络中，这样在每个服务器中进行循环操作，直到满足多数原则就选举出Leader 服务器

##### **生：
> RecvWorker 这个类的代码在哪呢，贴错了吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; RecvWorker 这个作为变量，并没有贴出它的实现。主要讲的是同步数据的过程，并不能把所有的代码都贴出来，还是要学员自己搭建一个源码环境，然后结合课程来学习

