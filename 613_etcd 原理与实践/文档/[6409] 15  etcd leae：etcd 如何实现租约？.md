<p data-nodeid="38089">上一讲我们介绍了 etcd Watch 实现的机制，今天我们继续分析 etcd 的另一个重要特性：Lease 租约。它类似 TTL（Time To Live），用于 etcd 客户端与服务端之间进行活性检测。在到达 TTL 时间之前，etcd 服务端不会删除相关租约上绑定的键值对；超过 TTL 时间，则会删除。因此我们<strong data-nodeid="38170">需要在到达 TTL 时间之前续租，以实现客户端与服务端之间的保活</strong>。</p>
<p data-nodeid="38090">Lease 也是 etcd v2 与 v3 版本之间的重要变化之一。etcd v2 版本并没有 Lease 概念，TTL 直接绑定在 key 上面。每个 TTL、key 创建一个 HTTP/1.x 连接，定时发送续期请求给 etcd Server。etcd v3 则在 v2 的基础上进行了重大升级，每个 Lease 都设置了一个 TTL 时间，<strong data-nodeid="38176">具有相同 TTL 时间的 key 绑定到同一个 Lease</strong>，实现了 Lease 的复用，并且基于 gRPC 协议的通信实现了连接的多路复用。</p>
<p data-nodeid="38091">下面我们就来介绍 etcd Lease 的基本用法以及分析 Lease 实现的原理。</p>
<h3 data-nodeid="38092">如何使用租约</h3>
<p data-nodeid="38093">Lease 意为租约，类似于分布式系统的中的 TTL（Time To Live）。在介绍 Lease 的实现原理之前，我们先通过 etcdctl 命令行工具来熟悉 Lease 的用法。依次执行如下的命令：</p>
<pre class="lang-shell" data-nodeid="38094"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> etcdctl lease grant 1000</span>
lease 694d77aa9e38260f granted with ttl(1000s)
<span class="hljs-meta">$</span><span class="bash"> etcdctl lease timetolive 694d77aa9e38260f</span>
lease 694d77aa9e38260f granted with ttl(1000s), remaining(983s)
<span class="hljs-meta">$</span><span class="bash"> etcdctl put foo bar --lease 694d77aa9e38260f</span>
OK
<span class="hljs-meta">#</span><span class="bash"> 等待过期，再次查看租约信息</span>
<span class="hljs-meta">$</span><span class="bash"> etcdctl lease timetolive 694d77aa9e38260f</span>
lease 694d77aa9e38260f already expired
</code></pre>
<p data-nodeid="38095">如上的命令中，我们首先创建了一个 Lease，TTL 时间为 1000s；接着根据获取到的 LeaseID 查看其存活时间；然后写入一个键值对，并通过<code data-backticks="1" data-nodeid="38181">--lease</code>绑定 Lease；最后一条命令是在 1000s 之后再次查看该 Lease 对应的存活信息。</p>
<p data-nodeid="38096">通过 etcdctl 命令行工具的形式，我们创建了指定 TTL 时间的 Lease，并了解了 Lease 的基本使用。下面我们具体介绍 Lease 的实现。</p>
<h3 data-nodeid="38097">Lease 架构</h3>
<p data-nodeid="38098">Lease 模块对外提供了 Lessor 接口，其中定义了包括 Grant、Revoke、Attach 和 Renew 等常用的方法，lessor 结构体实现了 Lessor 接口。Lease 模块涉及的主要对象和接口，如下图所示：</p>
<p data-nodeid="39491" class=""><img src="https://s0.lgstatic.com/i/image6/M01/1D/0B/Cgp9HWBPGqaAOSvBAAApAR0Pn5s728.png" alt="Drawing 0.png" data-nodeid="39495"></p>
<div data-nodeid="39492"><p style="text-align:center">Lease 模块涉及的主要对象和接口</p></div>



<p data-nodeid="38101">除此之外，lessor 还启动了两个异步 goroutine：RevokeExpiredLease 和 CheckpointScheduledLease，分别用于撤销过期的租约和更新 Lease 的剩余到期时间。</p>
<p data-nodeid="38102">下图是客户端创建一个指定 TTL 的租约流程，当 etcd 服务端的 gRPC Server 接收到创建 Lease 的请求后，Raft 模块首先进行日志同步；接着 MVCC 调用 Lease 模块的 Grant 接口，保存对应的日志条目到 ItemMap 结构中，接着将租约信息存到 boltdb；最后将 LeaseID 返回给客户端，Lease 创建成功。</p>
<p data-nodeid="40287" class=""><img src="https://s0.lgstatic.com/i/image6/M01/1D/08/CioPOWBPGq6AbSSHAAAmYqC-FLA568.png" alt="Drawing 1.png" data-nodeid="40291"></p>
<div data-nodeid="40288"><p style="text-align:center">客户端创建一个指定 TTL 租约流程图</p></div>



<p data-nodeid="38105">那么 Lease 与键值对是如何绑定的呢？</p>
<p data-nodeid="38106">客户端根据返回的 LeaseID，在执行写入和更新操作时，可以绑定该 LeaseID。如上面示例的命令行工具 etcdctl 指定<code data-backticks="1" data-nodeid="38198">--lease</code>参数，MVCC 会调用 Lease 模块 Lessor 接口中的 Attach 方法，将 key 关联到 Lease 的 key 内存集合 ItemSet 中，以完成键值对与 Lease 租约的绑定。</p>
<h3 data-nodeid="38107">实现细节</h3>
<p data-nodeid="38108">我们继续来看 etcd Lease 实现涉及的主要接口和结构体。</p>
<h4 data-nodeid="38109">Lessor 接口</h4>
<p data-nodeid="38110">Lessor 接口是 Lease 模块对外提供功能的核心接口，定义了包括<strong data-nodeid="38208">创建、绑定和延长租约</strong>等常用方法：</p>
<pre class="lang-go" data-nodeid="38111"><code data-language="go"><span class="hljs-comment">// 位于 lease/lessor.go:82</span>
<span class="hljs-keyword">type</span> Lessor <span class="hljs-keyword">interface</span> {
    <span class="hljs-comment">//...省略部分</span>
    <span class="hljs-comment">// 将 lessor 设置为 Primary，这个与 raft 会出现网络分区有关</span>
    Promote(extend time.Duration)
	<span class="hljs-comment">// Grant 创建了一个在指定时间过期的 Lease 对象</span>
	Grant(id LeaseID, ttl <span class="hljs-keyword">int64</span>) (*Lease, error)
	<span class="hljs-comment">// Revoke 撤销指定 LeaseID，绑定到其上的键值对将会被移除，如果该 LeaseID 对应的 Lease 不存在，则会返回错误</span>
	Revoke(id LeaseID) error
	<span class="hljs-comment">// Attach 绑定给定的 LeaseItem 到 LeaseID，如果该租约不存在，将会返回错误</span>
	Attach(id LeaseID, items []LeaseItem) error
	<span class="hljs-comment">// GetLease 返回 LeaseItem 对应的 LeaseID</span>
	GetLease(item LeaseItem) LeaseID
	<span class="hljs-comment">// Detach 将 LeaseItem 从给定的 LeaseID 解绑。如果租约不存在，则会返回错误</span>
	Detach(id LeaseID, items []LeaseItem) error
	<span class="hljs-comment">// Renew 刷新指定 LeaseID，结果将会返回刷新后的 TTL</span>
	Renew(id LeaseID) (<span class="hljs-keyword">int64</span>, error)
	<span class="hljs-comment">// Lookup 查找指定的 LeaseID，返回对应的 Lease</span>
	Lookup(id LeaseID) *Lease
	<span class="hljs-comment">// Leases 方法列出所有的 Leases</span>
	Leases() []*Lease
	<span class="hljs-comment">// ExpiredLeasesC 用于返回接收过期 Lease 的 channel</span>
	ExpiredLeasesC() &lt;-<span class="hljs-keyword">chan</span> []*Lease
}
</code></pre>
<p data-nodeid="38112">Lessor 接口定义了很多方法，租约相关的方法都在这里面。常用的方法有：</p>
<ul data-nodeid="38113">
<li data-nodeid="38114">
<p data-nodeid="38115">Grant 创建一个在指定时间过期的 Lease 对象；</p>
</li>
<li data-nodeid="38116">
<p data-nodeid="38117">Revoke 撤销指定 LeaseID，绑定到其上的键值对将会被移除；</p>
</li>
<li data-nodeid="38118">
<p data-nodeid="38119">Attach 绑定给定的 leaseItem 到 LeaseID；</p>
</li>
<li data-nodeid="38120">
<p data-nodeid="38121">Renew 刷新指定 LeaseID，结果将会返回刷新后的 TTL。</p>
</li>
</ul>
<h4 data-nodeid="38122">Lease 与 lessor 结构体</h4>
<p data-nodeid="38123">下面我们来看租约相关的 Lease 结构体：</p>
<pre class="lang-go" data-nodeid="38124"><code data-language="go"><span class="hljs-comment">// 位于 lease/lessor.go:800</span>
<span class="hljs-keyword">type</span> Lease <span class="hljs-keyword">struct</span> {
	ID           LeaseID
	ttl          <span class="hljs-keyword">int64</span> <span class="hljs-comment">// 存活时间，单位秒</span>
	remainingttl <span class="hljs-keyword">int64</span> <span class="hljs-comment">// 剩余的存活时间，如果为 0，则被认为是未设置，这种情况下该值与 TTL 相等</span>
	<span class="hljs-comment">// expiry 的并发锁</span>
	expiryMu sync.RWMutex
	<span class="hljs-comment">// expiry 是 Lease 过期的时间，当expiry.IsZero() 为 true 时，则没有过期时间</span>
	expiry time.Time
	<span class="hljs-comment">// ItemSet 并发锁</span>
	mu      sync.RWMutex
	itemSet <span class="hljs-keyword">map</span>[LeaseItem]<span class="hljs-keyword">struct</span>{}
	revokec <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
}
</code></pre>
<p data-nodeid="38125">租约 Lease 的定义中包含了 LeaseID、TTL、过期时间等属性。其中<strong data-nodeid="38221">LeaseID 在获取 Lease 的时候生成</strong>。</p>
<p data-nodeid="38126">lessor 实现了 Lessor 接口，我们继续来看 lessor 结构体的定义。lessor 是对租约的封装，其中对外暴露出一系列操作租约的方法，比如创建、绑定和延长租约的方法：</p>
<pre class="lang-go" data-nodeid="38127"><code data-language="go"><span class="hljs-keyword">type</span> lessor <span class="hljs-keyword">struct</span> {
	mu sync.RWMutex
	demotec <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
	leaseMap             <span class="hljs-keyword">map</span>[LeaseID]*Lease
	leaseExpiredNotifier *LeaseExpiredNotifier
	leaseCheckpointHeap  LeaseQueue
	itemMap              <span class="hljs-keyword">map</span>[LeaseItem]LeaseID
	<span class="hljs-comment">// 当 Lease 过期，lessor 将会通过 RangeDeleter 删除相应范围内的 keys</span>
	rd RangeDeleter
	cp Checkpointer
	<span class="hljs-comment">// backend 目前只会保存 LeaseID 和 expiry。LeaseItem 通过遍历 kv 中的所有键来恢复</span>
	b backend.Backend
	<span class="hljs-comment">// minLeasettl 是最小的 TTL 时间</span>
	minLeasettl <span class="hljs-keyword">int64</span>
	expiredC <span class="hljs-keyword">chan</span> []*Lease
	<span class="hljs-comment">// stopC 用来表示 lessor 应该被停止的 channel</span>
	stopC <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
	<span class="hljs-comment">// doneC 用来表示 lessor 已经停止的 channel</span>
	doneC <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
	lg *zap.Logger
	checkpointInterval time.Duration
	expiredLeaseRetryInterval time.Duration
}
</code></pre>
<p data-nodeid="38128">lessor 实现了 Lessor 接口，lessor 中维护了三个数据结构：LeaseMap、ItemMap 和 LeaseExpiredNotifier。</p>
<ul data-nodeid="38129">
<li data-nodeid="38130">
<p data-nodeid="38131">leaseMap 是一个 map 结构，其定义为 map[LeaseID]*Lease，用于根据 LeaseID 快速查询对应的 Lease；</p>
</li>
<li data-nodeid="38132">
<p data-nodeid="38133">ItemMap 同样是一个 map 结构，其定义为<br>
map[LeaseItem]LeaseID，用于根据 LeaseItem 快速查找 LeaseID，从而找到对应的 Lease；</p>
</li>
<li data-nodeid="38134">
<p data-nodeid="38135">LeaseExpiredNotifier 是对 LeaseQueue 的一层封装，使得快要到期的租约保持在队头。</p>
</li>
</ul>
<p data-nodeid="38136">其中 LeaseQueue 是一个优先级队列，每次插入都会根据<strong data-nodeid="38251">过期时间</strong>插入到合适的位置。优先级队列，普遍都是用堆来实现，etcd Lease 的实现基于<strong data-nodeid="38252">最小堆</strong>，比较的依据是<strong data-nodeid="38253">Lease 失效的时间</strong>。我们每次从最小堆里判断堆顶元素是否失效，失效就 Pop 出来并保存到 expiredC 的 channel 中。etcd Server 会定期从 channel 读取过期的 LeaseID，之后发起 revoke 请求。</p>
<p data-nodeid="38137">那么集群中的其他 etcd 节点是如何删除过期节点的呢？</p>
<p data-nodeid="38138">通过 Raft 日志将 revoke 请求发送给其他节点，集群中的其他节点收到 revoke 请求后，首先获取 Lease 绑定的键值对，接着删除 boltdb 中的 key 和存储的 Lease 信息，以及 LeaseMap 中的 Lease 对象。</p>
<h3 data-nodeid="38139">核心方法解析</h3>
<p data-nodeid="38140">Lessor 接口中有几个常用的核心方法，包括<strong data-nodeid="38262">Grant 申请租约、Attach 绑定租约以及 Revoke 撤销租约</strong>等。下面我们具体介绍这几个方法的实现。</p>
<h4 data-nodeid="38141">Grant 申请租约</h4>
<p data-nodeid="38142">客户端要想申请一个租约 Lease，需要调用 Lessor 对外暴露的 Grant 方法。Grant 用于申请租约，并在指定的 TTL 时长之后失效。具体实现如下：</p>
<pre class="lang-go" data-nodeid="38143"><code data-language="go"><span class="hljs-comment">// 位于 lease/lessor.go:258</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(le *lessor)</span> <span class="hljs-title">Grant</span><span class="hljs-params">(id LeaseID, ttl <span class="hljs-keyword">int64</span>)</span> <span class="hljs-params">(*Lease, error)</span></span> {
  <span class="hljs-comment">// TTL 不能大于 MaxLeasettl</span>
	<span class="hljs-keyword">if</span> ttl &gt; MaxLeasettl {
		<span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>, ErrLeasettlTooLarge
	}
	<span class="hljs-comment">// 构建 Lease 对象</span>
	l := &amp;Lease{
		ID:      id,
		ttl:     ttl,
		itemSet: <span class="hljs-built_in">make</span>(<span class="hljs-keyword">map</span>[LeaseItem]<span class="hljs-keyword">struct</span>{}),
		revokec: <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}),
	}
	le.mu.Lock()
	<span class="hljs-keyword">defer</span> le.mu.Unlock()
    <span class="hljs-comment">// 查找内存 LeaseMap 中是否有 LeaseID 对应的 Lease</span>
	<span class="hljs-keyword">if</span> _, ok := le.leaseMap[id]; ok {
		<span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>, ErrLeaseExists
	}
	<span class="hljs-keyword">if</span> l.ttl &lt; le.minLeasettl {
		l.ttl = le.minLeasettl
	}
	<span class="hljs-keyword">if</span> le.isPrimary() {
		l.refresh(<span class="hljs-number">0</span>)
	} <span class="hljs-keyword">else</span> {
		l.forever()
	}
    <span class="hljs-comment">// 将 l 存放到 LeaseMap 和 LeaseExpiredNotifier&nbsp;</span>
	le.leaseMap[id] = l
	item := &amp;LeaseWithTime{id: l.ID, time: l.expiry.UnixNano()}
	le.leaseExpiredNotifier.RegisterOrUpdate(item)
	l.persistTo(le.b)
	leaseTotalttls.Observe(<span class="hljs-keyword">float64</span>(l.ttl))
	leaseGranted.Inc()
	<span class="hljs-keyword">if</span> le.isPrimary() {
		le.scheduleCheckpointIfNeeded(l)
	}
	<span class="hljs-keyword">return</span> l, <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="38144">可以看到，当 Grant 一个租约 Lease 时，Lease 被同时存放到 LeaseMap 和 LeaseExpiredNotifier 中。在队列头，有一个 goroutine  revokeExpiredLeases 定期检查队头的租约是否过期，如果过期就放入 expiredChan 中。只有当发起 revoke 操作之后，才会从队列中删除。</p>
<h4 data-nodeid="38145">Attach 绑定租约</h4>
<p data-nodeid="38146">Attach 用于绑定键值对与指定的 LeaseID。当租约过期，且没有续期的情况下，该 Lease 上绑定的键值对会被自动移除。</p>
<pre class="lang-go" data-nodeid="38147"><code data-language="go"><span class="hljs-comment">// 位于 lease/lessor.go:518</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(le *lessor)</span> <span class="hljs-title">Attach</span><span class="hljs-params">(id LeaseID, items []LeaseItem)</span> <span class="hljs-title">error</span></span> {
	le.mu.Lock()
	<span class="hljs-keyword">defer</span> le.mu.Unlock()
  <span class="hljs-comment">// 从 LeaseMap 取出 LeaseID 对应的 lease</span>
	l := le.leaseMap[id]
	<span class="hljs-keyword">if</span> l == <span class="hljs-literal">nil</span> {
		<span class="hljs-keyword">return</span> ErrLeaseNotFound
	}
	l.mu.Lock()
	<span class="hljs-keyword">for</span> _, it := <span class="hljs-keyword">range</span> items {
		l.itemSet[it] = <span class="hljs-keyword">struct</span>{}{}
		le.itemMap[it] = id
	}
	l.mu.Unlock()
	<span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="38148">绑定键值对时，Attach 首先用 LeaseID 去 leaseMap 中查询租约是否存在。如果在 LeaseMap 中找不到给定的 LeaseID，将会返回错误；如果对应的租约存在，则会将 Item 保存到对应的租约下，随后将 Item 和 LeaseID 保存在 ItemMap 中。</p>
<h4 data-nodeid="38149">Revoke 撤销租约</h4>
<p data-nodeid="38150">Revoke 方法用于撤销指定 LeaseID 的租约，同时绑定到该 Lease 上的键值都会被移除。</p>
<pre class="lang-go" data-nodeid="38151"><code data-language="go"><span class="hljs-comment">// 位于 lease/lessor.go:311</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(le *lessor)</span> <span class="hljs-title">Revoke</span><span class="hljs-params">(id LeaseID)</span> <span class="hljs-title">error</span></span> {
	le.mu.Lock()
	l := le.leaseMap[id]
	<span class="hljs-keyword">if</span> l == <span class="hljs-literal">nil</span> {
		le.mu.Unlock()
		<span class="hljs-keyword">return</span> ErrLeaseNotFound
	}
	<span class="hljs-keyword">defer</span> <span class="hljs-built_in">close</span>(l.revokec)
	<span class="hljs-comment">// 释放锁</span>
	le.mu.Unlock()
	<span class="hljs-keyword">if</span> le.rd == <span class="hljs-literal">nil</span> {
		<span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
	}
	txn := le.rd()
	<span class="hljs-comment">// 对键值进行排序，使得所有的成员保持删除键值对的顺序一致</span>
	keys := l.Keys()
	sort.StringSlice(keys).Sort()
	<span class="hljs-keyword">for</span> _, key := <span class="hljs-keyword">range</span> keys {
		txn.DeleteRange([]<span class="hljs-keyword">byte</span>(key), <span class="hljs-literal">nil</span>)
	}
	le.mu.Lock()
	<span class="hljs-keyword">defer</span> le.mu.Unlock()
	<span class="hljs-built_in">delete</span>(le.leaseMap, l.ID)
	<span class="hljs-comment">// 键值删除操作需要在一个事务中进行</span>
	le.b.BatchTx().UnsafeDelete(leaseBucketName, int64ToBytes(<span class="hljs-keyword">int64</span>(l.ID)))
	txn.End()
	leaseRevoked.Inc()
	<span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="38152">想要实现 Revoke 方法，首先要根据 LeaseID 从 LeaseMap 中找到对应的 Lease 并从 LeaseMap 中删除，然后从 Lease 中找到绑定的 Key，并从 Backend 中将 KeyValue 删除。</p>
<h4 data-nodeid="38153">调用 Lessor API</h4>
<p data-nodeid="38154">上面我们介绍了 Lessor 接口中几个常用方法的实现。下面我们将基于上面三个接口，通过调用 Lessor API 创建 Lease 租约，将键值对绑定到租约上，到达 TTL 时间后主动将对应的键值对删除，实现代码如下：</p>
<pre class="lang-go" data-nodeid="38155"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">testLease</span><span class="hljs-params">()</span></span> {
    le := newLessor()    <span class="hljs-comment">// 创建一个 Lessor</span>
    le.Promote(<span class="hljs-number">0</span>)        
    Go <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {   <span class="hljs-comment">// 开启一个协程，接收过期的 key，主动删除</span>
        <span class="hljs-keyword">for</span> {  
           expireLease := &lt;-le.ExpiredLeasesC()  
           <span class="hljs-keyword">for</span> _, v := <span class="hljs-keyword">range</span> expireLease {  
              le.Revoke(v.ID)    <span class="hljs-comment">// 通过租约 ID 删除租约，删除租约时会从 backend 中删除绑定的 key</span>
           }  
        }
    }()
    ttl = <span class="hljs-number">5</span>         
    lease := le.Grant(id, ttl)   <span class="hljs-comment">// 申请一个租约</span>
    le.Attach(lease, <span class="hljs-string">"foo"</span>)      <span class="hljs-comment">// 将租约绑定在"foo"上</span>
    time.Sleep(<span class="hljs-number">10</span> * time.Second)
}
</code></pre>
<p data-nodeid="38156">上述代码展示了如何使用 Lessor 实现键值对申请、绑定和撤销租约操作。首先申请了一个过期时间设置为 5s 的 Lease；接着将 key<code data-backticks="1" data-nodeid="38275">foo</code>绑定到该 Lease 上，为了方便看到结果，阻塞 10s。</p>
<p data-nodeid="38157">同时有一点需要你注意，我们这里直接调用了 Lessor 对外提供的接口，<strong data-nodeid="38282">Lessor 不会主动删除过期的租约，而是将过期的 Lease 通过一个 channel 发送出来，由使用者主动删除</strong>。clientv3 包中定义好了 Lease 相关的实现，基于客户端 API 进行调用会更加简单。</p>
<h3 data-nodeid="38158">小结</h3>
<p data-nodeid="38159">这一讲我们主要介绍了 etcd Lease 的实现，首先通过 etcdctl 命令行工具介绍了客户端如何使用 Lease 的使用方法；接着介绍了 Lease 实现的主要架构，描述了 Lease 申请、绑定以及过期撤销的过程；随后介绍了 Lease 实现涉及的主要接口、结构体；最后介绍了 Lessor 对外提供的常见方法，包括：Grant 申请租约、Attach 绑定租约以及 Revoke 撤销租约，并通过一个测试用例介绍了如何直接使用 Lessor 对外提供的方法。</p>
<p data-nodeid="38160">本讲内容总结如下：</p>
<p data-nodeid="40686" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/1D/0B/Cgp9HWBPGsqAJAozAAE8kcN24Nw326.png" alt="Drawing 2.png" data-nodeid="40689"></p>

<p data-nodeid="38162">学习完这一讲，我想给大家留一个问题，你知道在 etcd 重启之后，Lease 与键值对的绑定关系是如何重建的吗？欢迎你在留言区和我分享自己的想法。下一讲，我们将从整体来梳理 etcd 启动的过程。</p>

---

### 精选评论


