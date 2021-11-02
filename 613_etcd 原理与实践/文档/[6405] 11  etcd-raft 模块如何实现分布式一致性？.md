<p data-nodeid="52008">上一讲我们介绍了etcd 读写操作的底层实现，但关于 etcd 集群如何实现分布式数据一致性并没有详细介绍。在分布式环境中，常用<strong data-nodeid="52235">数据复制</strong>来避免单点故障，实现多副本，提高服务的高可用性以及系统的吞吐量。</p>
<p data-nodeid="52009">etcd 集群中的多个节点不可避免地会出现相互之间数据不一致的情况。但不管是同步复制、异步复制还是半同步复制，都会存在可用性或者一致性的问题。我们一般会使用<strong data-nodeid="52241">共识算法</strong>来解决多个节点数据一致性的问题，常见的共识算法有 Paxos和Raft。ZooKeeper 使用的是 ZAB 协议，etcd 使用的共识算法就是Raft。</p>
<p data-nodeid="52010">etcd-raft 模块是 etcd中解决分布式一致性的模块，这一讲我们就结合源码具体分析这部分内容。</p>
<h3 data-nodeid="52011">etcdraft 对外提供的接口</h3>
<p data-nodeid="52012">raft 库对外提供了一个 Node 接口，由 raft/node.go 中的 node结构体实现，Node 接口需要实现的函数包括：Tick、Propose、Ready、Step 等。</p>
<p data-nodeid="52013">我们重点需要了解 Ready 接口，该接口将返回类型为 Ready 的 channel，该通道表示当前时间点的channel。应用层需要关注该 channel，当发生变更时，其中的数据也将会进行相应的操作。其他的函数对应的功能如下：</p>
<ul data-nodeid="52014">
<li data-nodeid="52015">
<p data-nodeid="52016">Tick：时钟，触发选举或者发送心跳；</p>
</li>
<li data-nodeid="52017">
<p data-nodeid="52018">Propose：通过 channel 向 raft StateMachine 提交一个 Op，提交的是本地 MsgProp 类型的消息；</p>
</li>
<li data-nodeid="52019">
<p data-nodeid="52020">Step：节点收到 Peer 节点发送的 Msg 时会通过该接口提交给 raft 状态机，Step 接口通过 recvc channel向raft StateMachine 传递 Msg；</p>
</li>
</ul>
<p data-nodeid="52021">然后是 raft 算法的实现，node 结构体实现了 Node 接口，其定义如下：</p>
<pre class="lang-go" data-nodeid="52022"><code data-language="go"><span class="hljs-keyword">type</span> node <span class="hljs-keyword">struct</span> {
propc      <span class="hljs-keyword">chan</span> msgWithResult
recvc      <span class="hljs-keyword">chan</span> pb.Message
confc      <span class="hljs-keyword">chan</span> pb.ConfChangeV2
confstatec <span class="hljs-keyword">chan</span> pb.ConfState
readyc     <span class="hljs-keyword">chan</span> Ready
advancec   <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
tickc <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
done       <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
stop       <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
status     <span class="hljs-keyword">chan</span> <span class="hljs-keyword">chan</span> Status
rn *RawNode
}
</code></pre>
<p data-nodeid="52023">这个结构体会在后面经常用到。</p>
<p data-nodeid="52024">在 raft/raft.go 中还有两个核心数据结构：</p>
<ul data-nodeid="52025">
<li data-nodeid="52026">
<p data-nodeid="52027">Config，封装了与 raft 算法相关的配置参数，公开用于外部调用。</p>
</li>
<li data-nodeid="52028">
<p data-nodeid="52029">raft，具体实现 raft 算法的结构体。</p>
</li>
</ul>
<h3 data-nodeid="52030">节点状态</h3>
<p data-nodeid="52031">下面我们来看看 raft StateMachine 的状态机转换，实际上就是 raft 算法中各种角色的转换。每个 raft 节点，可能具有以下三种状态中的一种。</p>
<ul data-nodeid="52032">
<li data-nodeid="52033">
<p data-nodeid="52034"><strong data-nodeid="52260">Candidate</strong>：候选人状态，该状态意味着将进行一次新的选举。</p>
</li>
<li data-nodeid="52035">
<p data-nodeid="52036"><strong data-nodeid="52265">Follower</strong>：跟随者状态，该状态意味着选举结束。</p>
</li>
<li data-nodeid="52037">
<p data-nodeid="52038"><strong data-nodeid="52270">Leader</strong>：领导者状态，选举出来的节点，所有数据提交都必须先提交到 Leader 上。</p>
</li>
</ul>
<p data-nodeid="52039">每一个状态都有其对应的状态机，每次收到一条提交的数据时，都会根据其不同的状态将消息输入到不同状态的状态机中。同时，在进行 tick 操作时，每种状态对应的处理函数也是不一样的。</p>
<p data-nodeid="52040">因此 raft 结构体中将不同的状态及其不同的处理函数，独立出来几个成员变量：</p>
<ul data-nodeid="52041">
<li data-nodeid="52042">
<p data-nodeid="52043">state，保存当前节点状态；</p>
</li>
<li data-nodeid="52044">
<p data-nodeid="52045">tick 函数，每个状态对应的 tick 函数不同；</p>
</li>
<li data-nodeid="52046">
<p data-nodeid="52047">step，状态机函数，同样每个状态对应的状态机也不相同。</p>
</li>
</ul>
<h3 data-nodeid="52048">状态转换</h3>
<p data-nodeid="52049">我们接着看 etcd raft 状态转换。etcd-raft StateMachine 封装在 raft机构体中，其状态转换如下图：</p>
<p data-nodeid="55328" class=""><img src="https://s0.lgstatic.com/i/image6/M01/0F/10/Cgp9HWA9EIqAVw9QAABJGq2-dl4448.png" alt="Drawing 0.png" data-nodeid="55332"></p>
<div data-nodeid="55329"><p style="text-align:center">etcd raft 状态转换示意图</p></div>



<p data-nodeid="52052">raft 状态转换的接口都在 raft.go 中，其定义如下：</p>
<pre class="lang-go" data-nodeid="52053"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(r *raft)</span> <span class="hljs-title">becomeFollower</span><span class="hljs-params">(term <span class="hljs-keyword">uint64</span>, lead <span class="hljs-keyword">uint64</span>)</span></span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(r *raft)</span> <span class="hljs-title">becomePreCandidate</span><span class="hljs-params">()</span></span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(r *raft)</span> <span class="hljs-title">becomeCandidate</span><span class="hljs-params">()</span></span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(r *raft)</span> <span class="hljs-title">becomeLeader</span><span class="hljs-params">()</span></span>
</code></pre>
<p data-nodeid="52054">raft 在不同的状态下，是如何驱动 raft StateMachine 状态机运转的呢？答案是etcd 将 raft 相关的所有处理都抽象为了 Msg，通过 Step 接口处理：</p>
<pre class="lang-go" data-nodeid="52055"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(r *raft)</span> <span class="hljs-title">Step</span><span class="hljs-params">(m pb.Message)</span> <span class="hljs-title">error</span></span> {<span class="hljs-string">``</span>
r.step(r, m)
}
</code></pre>
<p data-nodeid="52056">这里的step是一个<strong data-nodeid="52291">回调函数</strong>，根据不同的状态会设置不同的回调函数来驱动 raft，这个回调函数 stepFunc 就是在<code data-backticks="1" data-nodeid="52289">becomeXX()</code>函数完成的设置：</p>
<pre class="lang-java" data-nodeid="52057"><code data-language="java">type raft struct {
...
step stepFunc
}
</code></pre>
<p data-nodeid="52058">step 回调函数有如下几个值，注意其中 stepCandidate 会处理 PreCandidate 和 Candidate 两种状态：</p>
<pre class="lang-java" data-nodeid="52059"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">stepFollower</span><span class="hljs-params">(r *raft, m pb.Message)</span> error
func <span class="hljs-title">stepCandidate</span><span class="hljs-params">(r *raft, m pb.Message)</span> error
func <span class="hljs-title">stepLeader</span><span class="hljs-params">(r *raft, m pb.Message)</span> error
</span></code></pre>
<p data-nodeid="52060">这几个函数的实现其实就是对各种 Msg 进行处理，这里就不详细展开了。我们来看一下 raft 消息的类型及其定义。</p>
<h4 data-nodeid="52061">raft 消息</h4>
<p data-nodeid="52062"><strong data-nodeid="52299">raft 算法本质上是一个大的状态机</strong>，任何的操作例如选举、提交数据等，最后都被封装成一个消息结构体，输入到 raft 算法库的状态机中。</p>
<p data-nodeid="52063">在 raft/raftpb/raft.proto 文件中，定义了 raft 算法中传输消息的结构体。raft 算法其实由好几个协议组成，etcd-raft 将其统一定义在了 Message 结构体之中，以下总结了该结构体的成员用途：</p>
<pre class="lang-go" data-nodeid="52064"><code data-language="go"><span class="hljs-comment">// 位于 raft/raftpb/raft.pb.go:295</span>
<span class="hljs-keyword">type</span> Message <span class="hljs-keyword">struct</span> {
Type             MessageType <span class="hljs-string">`protobuf:"varint,1,opt,name=type,enum=raftpb.MessageType" json:"type"`</span> <span class="hljs-comment">// 消息类型</span>
To               <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,2,opt,name=to" json:"to"`</span> <span class="hljs-comment">// 消息接收者的节点ID</span>
From             <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,3,opt,name=from" json:"from"`</span> <span class="hljs-comment">// 消息发送者的节点 ID</span>
Term             <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,4,opt,name=term" json:"term"`</span> <span class="hljs-comment">// 任期 ID</span>
LogTerm          <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,5,opt,name=logTerm" json:"logTerm"`</span> <span class="hljs-comment">// 日志所处的任期 ID</span>
Index            <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,6,opt,name=index" json:"index"`</span> <span class="hljs-comment">// 日志索引 ID，用于节点向 Leader 汇报自己已经commit的日志数据 ID</span>
Entries          []Entry     <span class="hljs-string">`protobuf:"bytes,7,rep,name=entries" json:"entries"`</span> <span class="hljs-comment">// 日志条目数组</span>
Commit           <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,8,opt,name=commit" json:"commit"`</span> <span class="hljs-comment">// 提交日志索引</span>
Snapshot         Snapshot    <span class="hljs-string">`protobuf:"bytes,9,opt,name=snapshot" json:"snapshot"`</span> <span class="hljs-comment">// 快照数据</span>
Reject           <span class="hljs-keyword">bool</span>        <span class="hljs-string">`protobuf:"varint,10,opt,name=reject" json:"reject"`</span> <span class="hljs-comment">// 是否拒绝</span>
RejectHint       <span class="hljs-keyword">uint64</span>      <span class="hljs-string">`protobuf:"varint,11,opt,name=rejectHint" json:"rejectHint"`</span> <span class="hljs-comment">// 拒绝同步日志请求时返回的当前节点日志 ID，用于被拒绝方快速定位到下一次合适的同步日志位置</span>
Context          []<span class="hljs-keyword">byte</span>      <span class="hljs-string">`protobuf:"bytes,12,opt,name=context" json:"context,omitempty"`</span> <span class="hljs-comment">// 上下文数据</span>
XXX_unrecognized []<span class="hljs-keyword">byte</span>      <span class="hljs-string">`json:"-"`</span>
}
</code></pre>
<p data-nodeid="52065">Message结构体相关的数据类型为 MessageType，MessageType 有 19 种。当然，并不是所有的消息类型都会用到上面定义的Message结构体中的所有字段，因此其中有些字段是Optinal的。</p>
<p data-nodeid="52066">我将其中常用的协议（即不同的消息类型）的用途总结成如下的表格：</p>
<p data-nodeid="56277" class=""><img src="https://s0.lgstatic.com/i/image6/M01/0F/10/Cgp9HWA9EJ2ABd9rAAO37xDchBs024.png" alt="Drawing 1.png" data-nodeid="56280"></p>


<p data-nodeid="52156">上表列出了消息的类型对应的功能、消息接收者的节点 ID 和消息发送者的节点 ID。在收到消息之后，根据消息类型检索此表，可以帮助我们理解 raft 算法的操作。</p>
<h3 data-nodeid="52157">选举流程</h3>
<p data-nodeid="52158">raft 一致性算法实现的关键有 Leader 选举、日志复制和安全性限制。Leader 故障后集群能快速选出新 Leader，集群只有Leader 能写入日志， Leader 负责复制日志到 Follower 节点，并强制 Follower 节点与自己保持相同。</p>
<p data-nodeid="52159">raft 算法的第一步是<strong data-nodeid="52386">选举出 Leader</strong>，即使在 Leader 出现故障后也需要快速选出新 Leader，下面我们来梳理一下选举的流程。</p>
<h4 data-nodeid="67532" class="">发起选举</h4>




<p data-nodeid="52161">发起选举对节点的状态有限制，很显然只有在<strong data-nodeid="52396">Candidate 或者 Follower 状态</strong>下的节点才有可能发起一个选举流程，而这两种状态的节点，其对应的tick 函数为 raft.tickElection 函数，用来发起选举和选举超时控制。发起选举的流程如下。</p>
<ul data-nodeid="57550">
<li data-nodeid="57551">
<p data-nodeid="57552">节点启动时都以Follower 启动，同时随机生成自己的选举超时时间。</p>
</li>
<li data-nodeid="57553">
<p data-nodeid="57554">在Follower的tickElection 函数中，当选举超时，节点向自己发送 MsgHup 消息。</p>
</li>
<li data-nodeid="57555">
<p data-nodeid="57556">在状态机函数 raft.Step函数中，收到 MsgHup 消息之后，节点首先判断当前有没有 apply 的配置变更消息，如果有就忽略该消息。</p>
</li>
<li data-nodeid="57557">
<p data-nodeid="57558">否则进入 campaign 函数中进行选举：首先将任期号增加 1，然后广播给其他节点选举消息，带上的其他字段，包括：节点当前的最后一条日志索引（Index 字段）、最后一条日志对应的任期号（LogTerm 字段）、选举任期号（Term 字段，即前面已经进行 +1 之后的任期号）、Context 字段（目的是告知这一次是否是 Leader 转让类需要强制进行选举的消息）。</p>
</li>
<li data-nodeid="57559">
<p data-nodeid="57560" class="">如果在一个选举超时期间内，发起新的选举流程的节点，得到了超过半数的节点投票，那么状态就切换到 Leader 状态。<strong data-nodeid="57570">成为 Leader的同时，Leader 将发送一条 dummy 的 append 消息</strong>，目的是提交该节点上在此任期之前的值。</p>
</li>
</ul>


<p data-nodeid="52173">在上述流程中，之所以每个节点随机选择自己的超时时间，是为了<strong data-nodeid="52416">避免有两个节点同时进行选举</strong>，此时没有任何一个节点会赢得半数以上的投票，从而导致这一轮选举失败，继续进行下一轮选举。在第三步，判断是否有 apply 配置变更消息，其原因在于，当<strong data-nodeid="52417">有配置更新的情况下不能进行选举操作</strong>，即要保证每一次集群成员变化时只能变化一个，不能多个集群成员的状态同时发生变化。</p>
<h4 data-nodeid="62595" class="">参与选举</h4>








<p data-nodeid="52175">当收到任期号大于当前节点任期号的消息，且该消息类型如果是选举类的消息（类型为 prevote 或者 vote）时，节点会做出以下判断。</p>
<ul data-nodeid="52176">
<li data-nodeid="52177">
<p data-nodeid="52178">首先判断该消息是否为强制要求进行选举的类型（context 为 campaignTransfer，表示进行 Leader转让）。</p>
</li>
<li data-nodeid="52179">
<p data-nodeid="52180">判断当前是否在租约期内，满足的条件包括：checkQuorum 为 true、当前节点保存的 Leader 不为空、没有到选举超时。</p>
</li>
</ul>
<p data-nodeid="52181">如果不是强制要求选举，且在租约期内，就忽略该选举消息，这样做是为了避免出现那些分裂集群的节点，频繁发起新的选举请求。</p>
<ul data-nodeid="52182">
<li data-nodeid="52183">
<p data-nodeid="52184">如果不是忽略选举消息的情况，除非是 prevote 类的选举消息，否则在收到其他消息的情况下，该节点都切换为 Follower 状态。</p>
</li>
<li data-nodeid="52185">
<p data-nodeid="52186">此时需要针对投票类型中带来的其他字段进行处理，同时满足日志新旧的判断和参与选举的条件。</p>
</li>
</ul>
<p data-nodeid="52187">只有在同时满足以上两个条件的情况下，才能同意该节点的选举，否则都会被拒绝。这种做法可以保证最后选出来的新 Leader 节点，其日志都是最新的。</p>
<h3 data-nodeid="52188">日志复制</h3>
<p data-nodeid="52189">选举好 Leader 后，Leader在收到 put 提案时，如何将提案复制给其他 Follower 呢？</p>
<p data-nodeid="52190">我们回顾一下前面讲的 etcd 读写请求的处理流程。并结合下图说明日志复制的流程：</p>
<p data-nodeid="63832" class=""><img src="https://s0.lgstatic.com/i/image6/M01/0F/0D/CioPOWA9ELSAZR1_AACDc7CHoj4319.png" alt="Drawing 2.png" data-nodeid="63836"></p>
<div data-nodeid="63833"><p style="text-align:center">日志复制的流程图 ①</p></div>



<ul data-nodeid="52193">
<li data-nodeid="52194">
<p data-nodeid="52195">收到客户端请求之后，etcd Server 的 KVServer 模块会向 raft 模块提交一个类型为 MsgProp 的提案消息。</p>
</li>
<li data-nodeid="52196">
<p data-nodeid="52197">Leader节点在本地添加一条日志，其对应的命令为<code data-backticks="1" data-nodeid="52438">put foo bar</code>。此步骤只是添加一条日志，并没有提交，<strong data-nodeid="52444">两个索引值还指向上一条日志</strong>。</p>
</li>
<li data-nodeid="52198">
<p data-nodeid="52199">Leader 节点向集群中其他节点广播 AppendEntries 消息，带上 put 命令。</p>
</li>
</ul>
<p data-nodeid="52200">第二步中，两个索引值分别为 committedIndex和appliedIndex，图中有标识。committedIndex 存储最后一条提交日志的索引，而 appliedIndex 存储的是最后一条应用到状态机中的日志索引值。两个数值满足<strong data-nodeid="52451">committedIndex 大于等于 appliedIndex</strong>，这是因为一条日志只有被提交了才能应用到状态机中。</p>
<p data-nodeid="52201">接下来我们看看 Leader 如何将日志数据复制到 Follower 节点。</p>
<p data-nodeid="68142" class=""><img src="https://s0.lgstatic.com/i/image6/M00/0F/0E/CioPOWA9ENOAQLFKAAB3Zn9Qj4Q619.png" alt="Drawing 3.png" data-nodeid="68146"></p>
<div data-nodeid="68143"><p style="text-align:center">日志复制的流程图 ②</p></div>





<ul data-nodeid="52204">
<li data-nodeid="52205">
<p data-nodeid="52206">Follower 节点收到 AppendEntries 请求后，与 Leader 节点一样，在本地添加一条新的日志，此时日志也没有提交。</p>
</li>
<li data-nodeid="52207">
<p data-nodeid="52208">添加成功日志后，Follower 节点向 Leader 节点应答 AppendEntries 消息。</p>
</li>
<li data-nodeid="52209">
<p data-nodeid="52210">Leader 节点汇总 Follower 节点的应答。当Leader 节点收到半数以上节点的 AppendEntries 请求的应答消息时，表明 <code data-backticks="1" data-nodeid="52460">put foo bar</code> 命令成功复制，可以进行日志提交。</p>
</li>
<li data-nodeid="52211">
<p data-nodeid="52212">Leader 修改本地 committed 日志的索引，指向最新的存储<code data-backticks="1" data-nodeid="52463">put foo bar</code>的日志，因为还没有应用该命令到状态机中，所以 appliedIndex 还是保持着上一次的值。</p>
</li>
</ul>
<p data-nodeid="52213">当这个命令提交完成之后，命令就可以提交给应用层了。</p>
<ul data-nodeid="52214">
<li data-nodeid="52215">
<p data-nodeid="52216">此时修改 appliedIndex的值，与 committedIndex 的值相等。</p>
</li>
<li data-nodeid="52217">
<p data-nodeid="52218">Leader 节点在后续发送给 Follower 的 AppendEntries 请求中，总会带上最新的 committedIndex 索引值。</p>
</li>
<li data-nodeid="52219">
<p data-nodeid="52220">Follower 收到AppendEntries 后会修改本地日志的 committedIndex 索引。</p>
</li>
</ul>
<p data-nodeid="52221">至此，日志复制的过程在集群的多个节点之间就完成了。</p>
<h3 data-nodeid="52222">小结</h3>
<p data-nodeid="52223">这一讲我们主要介绍了 etcd-raft 模块实现分布式一致性的原理，并且通过 raftexample 了解了 raft 模块的使用方式和过程。接着重点介绍了选举流程和日志复制的过程。</p>
<p data-nodeid="52224">本讲内容总结如下：</p>
<p data-nodeid="68755" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/0F/0E/CioPOWA9ENyARGJxAAG3ZxiXwp0220.png" alt="Drawing 4.png" data-nodeid="68758"></p>

<p data-nodeid="52226">除此之外，etcd 还有安全性限制，以保证日志选举和日志复制的正确性，比如 raft 算法中，并不是所有节点都能成为 Leader。一个节点要想成为 Leader，需要得到集群中半数以上节点的投票，而一个节点会投票给另一个节点，其中一个充分条件是：<strong data-nodeid="52481">进行选举的节点，其日志需要比本节点的日志更新</strong>。此外还有判断日志的新旧以及提交前面任期的日志条目等措施。</p>
<p data-nodeid="52227">学习完这一讲，我给大家留一个问题，哪些情况下会出现选举超时且没有任何一个节点成为 Leader？欢迎你在留言区和我分享你的观点。下一讲，我们将介绍 etcd 存储多版本控制 MVCC 如何实现。</p>

---

### 精选评论

##### **7162：
> 怎么解决写写操作丢数据？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请求具体是哪种异常的丢数据？ etcd 的 WAL 、Snapshot 以及事务机制可以保证大部分场景的数据一致性。假设出现日志条目没有持久化到db文件的情况，在etcd 启动时，会根据 WAL 文件重放，不会丢失数据。

