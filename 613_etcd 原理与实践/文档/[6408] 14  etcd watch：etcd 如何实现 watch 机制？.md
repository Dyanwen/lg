<p data-nodeid="829" class="">etcd v2 和 v3 版本之间发生的其中一个重要变化就是 watch 机制的优化。etcd v2 watch 机制采用的是基于 HTTP/1.x 协议的<strong data-nodeid="911">客户端轮询机制</strong>，历史版本则通过滑动窗口存储。在大量的客户端连接场景或集群规模较大的场景下，etcd 服务端的扩展性和稳定性都无法保证。etcd v3 在此基础上进行优化，满足了 Kubernetes Pods 部署和状态管理等业务场景诉求。</p>
<p data-nodeid="830">这一讲我们就来介绍 watch 的用法，包括如何通过 etcdctl 命令行工具及 clientv3 客户端实现键值对的监控。在了解基本用法的基础上，我们再来重点介绍 etcd watch 实现的原理和细节。</p>
<h3 data-nodeid="831">watch 的用法</h3>
<p data-nodeid="832">在具体讲解 watch 的实现方式之前，我们先来体验一下如何使用 watch。</p>
<h4 data-nodeid="833">etcdctl 命令行工具</h4>
<p data-nodeid="834">通过 etcdctl 命令行工具实现键值对的监测：</p>
<pre class="lang-shell" data-nodeid="835"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> etcdctl put hello aoho</span>
<span class="hljs-meta">$</span><span class="bash"> etcdctl put hello boho</span>
<span class="hljs-meta">$</span><span class="bash"> etcdctl watch hello -w=json --rev=1</span>
{
	"Header": {
		"cluster_id": 14841639068965178418,
		"member_id": 10276657743932975437,
		"revision": 4,
		"raft_term": 4
	},
	"Events": [{
		"kv": {
			"key": "aGVsbG8=",
			"create_revision": 3,
			"mod_revision": 3,
			"version": 1,
			"value": "YW9obw=="
		}
	}, {
		"kv": {
			"key": "aGVsbG8=",
			"create_revision": 3,
			"mod_revision": 4,
			"version": 2,
			"value": "Ym9obw=="
		}
	}],
	"CompactRevision": 0,
	"Canceled": false,
	"Created": false
}
</code></pre>
<p data-nodeid="836">依次在命令行中输入上面三条命令，前面两条依次更新 hello 对应的值，第三条命令监测键为 hello 的变化，并指定版本号从 1 开始。最后的结果是输出了两条 watch 事件。</p>
<p data-nodeid="837">接着，我们在另一个命令行继续输入如下的更新命令：</p>
<pre class="lang-shell" data-nodeid="838"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> etcdctl put hello coho</span>
</code></pre>
<p data-nodeid="839">可以看到前一个命令行输出了如下的内容：</p>
<pre class="lang-shell" data-nodeid="840"><code data-language="shell">{
	"Header": {
		"cluster_id": 14841639068965178418,
		"member_id": 10276657743932975437,
		"revision": 5,
		"raft_term": 4
	},
	"Events": [{
		"kv": {
			"key": "aGVsbG8=",
			"create_revision": 3,
			"mod_revision": 5,
			"version": 3,
			"value": "Y29obw=="
		}
	}],
	"CompactRevision": 0,
	"Canceled": false,
	"Created": false
}
</code></pre>
<p data-nodeid="841">命令行输出的事件表明，键<code data-backticks="1" data-nodeid="921">hello</code>对应的键值对发生了更新，并输出了事件的详细信息。上述内容就是通过 etcdctl 命令行工具实现 watch 指定的键值对功能的全过程。</p>
<h4 data-nodeid="842">clientv3 客户端</h4>
<p data-nodeid="843">下面我们继续来看在 clientv3 中如何实现 watch 功能。</p>
<p data-nodeid="844">etcd 的 MVCC 模块对外提供了两种访问键值对的实现方式，一种是键值存储 kvstore，另一种是 watchableStore，它们都实现了 KV 接口。clientv3 中很简洁地封装了 watch 客户端与服务端交互的细节，基于 watchableStore 即可实现 watch 功能，客户端使用的代码如下：</p>
<pre class="lang-go" data-nodeid="845"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">testWatch</span><span class="hljs-params">()</span></span> {
    s := newWatchableStore()
    w := s.NewWatchStream()
    w.Watch(start_key: hello, end_key: <span class="hljs-literal">nil</span>)
    <span class="hljs-keyword">for</span> {
        consume := &lt;- w.Chan()
    }
}
</code></pre>
<p data-nodeid="846">在上述实现中，我们调用了 watchableStore。为了实现 watch 监测，我们创建了一个 watchStream，watchStream 监听的 key 为 hello，之后我们就可以消费<code data-backticks="1" data-nodeid="927">w.Chan()</code>返回的 channel。key 为 hello 的任何变化，都会通过这个 channel 发送给客户端。</p>
<p data-nodeid="847"><img src="https://s0.lgstatic.com/i/image6/M00/19/19/Cgp9HWBJu_WAfdpGAAAef8_AVBQ682.png" alt="Drawing 0.png" data-nodeid="931"></p>
<p data-nodeid="848">结合这张图，我们可以看到：watchStream 实现了在大量 KV 的变化事件中，<strong data-nodeid="937">过滤出当前所指定监听的 key，并将键值对的变更事件输出</strong>。</p>
<h3 data-nodeid="849">watchableStore 存储</h3>
<p data-nodeid="1447" class="te-preview-highlight">在第 10 讲<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=613#/detail/pc?id=6404" data-nodeid="1451">“etcd 存储：如何实现键值对的读写操作？”</a>中我们已经介绍过 kvstore，这里我们具体介绍一下 watchableStore 的实现。</p>


<p data-nodeid="851"><strong data-nodeid="948">watchableStore 负责了注册、管理以及触发 Watcher 的功能</strong>。我们先来看一下这个结构体的各个字段：</p>
<pre class="lang-go" data-nodeid="852"><code data-language="go"><span class="hljs-comment">// 位于 mvcc/watchable_store.go:47</span>
<span class="hljs-keyword">type</span> watchableStore <span class="hljs-keyword">struct</span> {
	*store
	<span class="hljs-comment">// 同步读写锁</span>
	mu sync.RWMutex
	<span class="hljs-comment">// 被阻塞在 watch channel 中的 watcherBatch</span>
	victims []watcherBatch
	victimc <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
	<span class="hljs-comment">// 未同步的 watchers</span>
	unsynced watcherGroup
	<span class="hljs-comment">// 已同步的 watchers</span>
	synced watcherGroup
	stopc <span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
	wg    sync.WaitGroup
}
</code></pre>
<p data-nodeid="853">watchableStore 组合了 store 结构体的字段和方法，除此之外，还有两个 watcherGroup 类型的字段，watcherGroup 管理多个 watcher，并能够根据 key 快速找到监听该 key 的一个或多个 watcher。</p>
<ul data-nodeid="854">
<li data-nodeid="855">
<p data-nodeid="856">unsynced 表示 watcher 监听的数据还未同步完成。当创建的 watcher 指定的版本号小于 etcd server 最新的版本号时，会将 watcher 保存到 unsynced watcherGroup。</p>
</li>
<li data-nodeid="857">
<p data-nodeid="858">synced 表示 watcher 监听的数据都已经同步完毕，在等待新的变更。如果创建的 watcher 未指定版本号或指定的版本号大于当前最新的版本号，它将会保存到 synced watcherGroup 中。</p>
</li>
</ul>
<p data-nodeid="859">根据 watchableStore 的定义，我们可以结合下图描述前文示例 watch 监听的过程。</p>
<p data-nodeid="860"><img src="https://s0.lgstatic.com/i/image6/M01/19/17/CioPOWBJvWKAN2GEAAArY0rVWO4011.png" alt="Drawing 1.png" data-nodeid="955"></p>
<div data-nodeid="861"><p style="text-align:center">watch 监听流程</p></div>
<p data-nodeid="862">watchableStore 收到了所有 key 的变更后，将这些 key 交给 synced（watchGroup），synced 使用了 map 和 ADT（红黑树），能够快速地从所有 key 中找到监听的 key，将这些 key 发送给对应的 watcher，这些 watcher 再通过 chan 将变更信息发送出去。</p>
<p data-nodeid="863">在查找监听 key 对应的事件时，如果只监听一个 key：</p>
<pre class="lang-java" data-nodeid="864"><code data-language="java">watch(start_key: foo, end_key: nil)
</code></pre>
<p data-nodeid="865">则对应的存储为<code data-backticks="1" data-nodeid="959">map[key]*watcher</code>。这样可以根据 key 快速找到对应的 watcher。但是 watch 可以监听一组范围的 key，这种情况应该如何处理呢？</p>
<pre class="lang-java" data-nodeid="866"><code data-language="java">watch(start_key: hello1, end_key: hello3)
</code></pre>
<p data-nodeid="867">上面的代码监听了从 hello1→hello3 之间的所有 key，这些 key 的数量不固定，比如：key=hello11 也处于监听范围。这种情况就无法再使用 map 了，因此 etcd 用 ADT 结构来存储一个范围内的 key。</p>
<p data-nodeid="868">watcherGroup 是由一系列范围 watcher 组织起来的 watchers。在找到对应的 watcher 后，调用 watcher 的 send() 方法，将变更的事件发送出去。</p>
<h3 data-nodeid="869">syncWatchers 同步监听</h3>
<p data-nodeid="870">在初始化一个新的 watchableStore 时，etcd 会创建一个用于同步 watcherGroup 的 goroutine，会在 syncWatchersLoop 函数中<strong data-nodeid="969">每隔 100ms 调用一次 syncWatchers 方法</strong>，将所有未通知的事件通知给所有的监听者：</p>
<pre class="lang-go" data-nodeid="871"><code data-language="go"><span class="hljs-comment">// 位于 mvcc/watchable_store.go:334</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *watchableStore)</span> <span class="hljs-title">syncWatchers</span><span class="hljs-params">()</span> <span class="hljs-title">int</span></span> {
  <span class="hljs-comment">//...</span>
	<span class="hljs-comment">// 为了从 unsynced watchers 中找到未同步的键值对，我们需要查询最小的版本号，利用最小的版本号查询 backend 存储中的键值对</span>
	curRev := s.store.currentRev
	compactionRev := s.store.compactMainRev
	wg, minRev := s.unsynced.choose(maxWatchersPerSync, curRev, compactionRev)
	minBytes, maxBytes := newRevBytes(), newRevBytes()
	<span class="hljs-comment">// UnsafeRange 方法返回了键值对。在 boltdb 中存储的 key 都是版本号，而 value 为在 backend 中存储的键值对</span>
	tx := s.store.b.ReadTx()
	tx.RLock()
	revs, vs := tx.UnsafeRange(keyBucketName, minBytes, maxBytes, <span class="hljs-number">0</span>)
	<span class="hljs-keyword">var</span> evs []mvccpb.Event
  <span class="hljs-comment">// 转换成事件</span>
	evs = kvsToEvents(s.store.lg, wg, revs, vs)
	<span class="hljs-keyword">var</span> victims watcherBatch
	wb := newWatcherBatch(wg, evs)
	<span class="hljs-keyword">for</span> w := <span class="hljs-keyword">range</span> wg.watchers {
		w.minRev = curRev + <span class="hljs-number">1</span>
    <span class="hljs-comment">//...</span>
		<span class="hljs-keyword">if</span> eb.moreRev != <span class="hljs-number">0</span> {
			w.minRev = eb.moreRev
		}
    <span class="hljs-comment">// 通过 send 将事件和 watcherGroup 发送到每一个 watcher 对应的 channel 中</span>
		<span class="hljs-keyword">if</span> w.send(WatchResponse{WatchID: w.id, Events: eb.evs, Revision: curRev}) {
			pendingEventsGauge.Add(<span class="hljs-keyword">float64</span>(<span class="hljs-built_in">len</span>(eb.evs)))
		} <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 异常情况处理</span>
			<span class="hljs-keyword">if</span> victims == <span class="hljs-literal">nil</span> {
				victims = <span class="hljs-built_in">make</span>(watcherBatch)
			}
			w.victim = <span class="hljs-literal">true</span>
		}
    <span class="hljs-comment">//...</span>
		s.unsynced.<span class="hljs-built_in">delete</span>(w)
	}
  <span class="hljs-comment">//...</span>
}
</code></pre>
<p data-nodeid="872">简化后的 syncWatchers 方法中有三个核心步骤，首先是根据当前的版本从未同步的 watcherGroup 中选出一些待处理的任务，然后从 BoltDB 中获取当前版本范围内的数据变更，并将它们转换成事件，事件和 watcherGroup 在打包之后会通过 send 方法发送到每一个 watcher 对应的 channel 中。</p>
<p data-nodeid="873"><img src="https://s0.lgstatic.com/i/image6/M01/19/17/CioPOWBJvXaAJROAAAAlohW0T4M993.png" alt="Drawing 2.png" data-nodeid="973"></p>
<div data-nodeid="874"><p style="text-align:center">syncWatchers 方法调用流程图</p></div>
<h3 data-nodeid="875">客户端监听事件</h3>
<p data-nodeid="876">客户端监听键值对时，调用的正是<code data-backticks="1" data-nodeid="976">Watch</code>方法，<code data-backticks="1" data-nodeid="978">Watch</code>在 stream 中创建一个新的 watcher，并返回对应的 WatchID。</p>
<pre class="lang-go" data-nodeid="877"><code data-language="go"><span class="hljs-comment">// 位于 mvcc/watcher.go:108</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(ws *watchStream)</span> <span class="hljs-title">Watch</span><span class="hljs-params">(id WatchID, key, end []<span class="hljs-keyword">byte</span>, startRev <span class="hljs-keyword">int64</span>, fcs ...FilterFunc)</span> <span class="hljs-params">(WatchID, error)</span></span> {
	<span class="hljs-comment">// 防止出现 key &gt;= end 的错误 range</span>
	<span class="hljs-keyword">if</span> <span class="hljs-built_in">len</span>(end) != <span class="hljs-number">0</span> &amp;&amp; bytes.Compare(key, end) != <span class="hljs-number">-1</span> {
		<span class="hljs-keyword">return</span> <span class="hljs-number">-1</span>, ErrEmptyWatcherRange
	}
	ws.mu.Lock()
	<span class="hljs-keyword">defer</span> ws.mu.Unlock()
	<span class="hljs-keyword">if</span> ws.closed {
		<span class="hljs-keyword">return</span> <span class="hljs-number">-1</span>, ErrEmptyWatcherRange
	}
	<span class="hljs-keyword">if</span> id == AutoWatchID {
		<span class="hljs-keyword">for</span> ws.watchers[ws.nextID] != <span class="hljs-literal">nil</span> {
			ws.nextID++
		}
		id = ws.nextID
		ws.nextID++
	} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> _, ok := ws.watchers[id]; ok {
		<span class="hljs-keyword">return</span> <span class="hljs-number">-1</span>, ErrWatcherDuplicateID
	}
	w, c := ws.watchable.watch(key, end, startRev, id, ws.ch, fcs...)
	ws.cancels[id] = c
	ws.watchers[id] = w
	<span class="hljs-keyword">return</span> id, <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="878">AutoWatchID 是 WatchStream 中传递的观察者 ID。当用户没有提供可用的 ID 时，如果又传递该值，etcd 将自动分配一个 ID。<strong data-nodeid="987">如果传递的 ID 已经存在，则会返回 ErrWatcherDuplicateID 错误</strong>。watchable_store.go 中的 watch 实现是监听的具体实现，实现代码如下：</p>
<pre class="lang-go" data-nodeid="879"><code data-language="go"><span class="hljs-comment">// 位于 mvcc/watchable_store.go:120</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *watchableStore)</span> <span class="hljs-title">watch</span><span class="hljs-params">(key, end []<span class="hljs-keyword">byte</span>, startRev <span class="hljs-keyword">int64</span>, id WatchID, ch <span class="hljs-keyword">chan</span>&lt;- WatchResponse, fcs ...FilterFunc)</span> <span class="hljs-params">(*watcher, cancelFunc)</span></span> {
	<span class="hljs-comment">// 构建 watcher</span>
	wa := &amp;watcher{
		key:    key,
		end:    end,
		minRev: startRev,
		id:     id,
		ch:     ch,
		fcs:    fcs,
	}
	s.mu.Lock()
	s.revMu.RLock()
	synced := startRev &gt; s.store.currentRev || startRev == <span class="hljs-number">0</span>
	<span class="hljs-keyword">if</span> synced {
		wa.minRev = s.store.currentRev + <span class="hljs-number">1</span>
		<span class="hljs-keyword">if</span> startRev &gt; wa.minRev {
			wa.minRev = startRev
		}
	}
	<span class="hljs-keyword">if</span> synced {
		s.synced.add(wa)
	} <span class="hljs-keyword">else</span> {
		slowWatcherGauge.Inc()
		s.unsynced.add(wa)
	}
	s.revMu.RUnlock()
	s.mu.Unlock()
	<span class="hljs-comment">// prometheus 的指标增加</span>
	watcherGauge.Inc()
	<span class="hljs-keyword">return</span> wa, <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> { s.cancelWatcher(wa) }
}
</code></pre>
<p data-nodeid="880">对 watchableStore 进行操作之前，需要加锁。如果 etcd 收到客户端的 watch 请求中携带了 revision 参数，则<strong data-nodeid="993">比较请求的 revision 和 store 当前的 revision</strong>，如果大于当前 revision，则放入 synced 组中，否则放入 unsynced 组。</p>
<h3 data-nodeid="881">服务端处理监听</h3>
<p data-nodeid="882">当 etcd 服务启动时，会在服务端运行一个用于处理监听事件的 watchServer gRPC 服务，客户端的 watch 请求最终都会被转发到 Watch 函数处理：</p>
<pre class="lang-go" data-nodeid="883"><code data-language="go"><span class="hljs-comment">// 位于 etcdserver/api/v3rpc/watch.go:140</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(ws *watchServer)</span> <span class="hljs-title">Watch</span><span class="hljs-params">(stream pb.Watch_WatchServer)</span> <span class="hljs-params">(err error)</span></span> {
	sws := serverWatchStream{
    <span class="hljs-comment">// 构建 serverWatchStream</span>
	}
	sws.wg.Add(<span class="hljs-number">1</span>)
	<span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
		sws.sendLoop()
		sws.wg.Done()
	}()
	errc := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> error, <span class="hljs-number">1</span>)
  <span class="hljs-comment">// 理想情况下，recvLoop 将会使用 sws.wg 通知操作的完成，但是当  stream.Context().Done() 关闭时，由于使用了不同的 ctx，stream 的接口有可能一直阻塞，调用 sws.close() 会发生死锁</span>
	<span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
		<span class="hljs-keyword">if</span> rerr := sws.recvLoop(); rerr != <span class="hljs-literal">nil</span> {
			<span class="hljs-keyword">if</span> isClientCtxErr(stream.Context().Err(), rerr) {
        <span class="hljs-comment">// 错误处理</span>
			}
			errc &lt;- rerr
		}
	}()
	<span class="hljs-keyword">select</span> {
	<span class="hljs-keyword">case</span> err = &lt;-errc:
		<span class="hljs-built_in">close</span>(sws.ctrlStream)
	<span class="hljs-keyword">case</span> &lt;-stream.Context().Done():
		err = stream.Context().Err()
		<span class="hljs-keyword">if</span> err == context.Canceled {
			err = rpctypes.ErrGRPCNoLeader
		}
	}
	sws.<span class="hljs-built_in">close</span>()
	<span class="hljs-keyword">return</span> err
}
</code></pre>
<p data-nodeid="884">如果出现了更新或者删除操作，相应的事件就会被发送到 watchStream 的通道中。客户端可以通过 Watch 功能监听某一个 Key 或者一个范围的变动，在每一次客户端调用服务端时都会创建两个 goroutine，其中一个协程 sendLoop 负责向监听者发送数据变动的事件，另一个协程 recvLoop 负责处理客户端发来的事件。</p>
<p data-nodeid="885">sendLoop 会通过<strong data-nodeid="1006">select 关键字</strong>来监听多个 channel 中的数据，将接收到的数据封装成 pb.WatchResponse 结构，并通过 gRPC 流发送给客户端；recvLoop 方法调用了 MVCC 模块暴露出的<strong data-nodeid="1007">watchStream.Watch 方法</strong>，该方法会返回一个可以用于取消监听事件的 watchID；当 gRPC 流已经结束或者出现错误时，当前的循环就会返回，两个 goroutine 也都会结束。</p>
<h3 data-nodeid="886">异常流程处理</h3>
<p data-nodeid="887">我们来考虑一下异常流程的处理。消息都是通过 channel 发送出去，但如果消费者消费速度慢，channel 中的消息形成堆积，但是空间有限，满了之后应该怎么办呢？带着这个问题，首先我们来看 channel 的默认容量：</p>
<pre class="lang-go" data-nodeid="888"><code data-language="go"><span class="hljs-keyword">var</span> (
	<span class="hljs-comment">// chanBufLen 是发送 watch 事件的 buffered channel 长度</span>
   chanBufLen = <span class="hljs-number">1024</span>
	<span class="hljs-comment">// maxWatchersPerSync 是每次 sync 时 watchers 的数量</span>
	maxWatchersPerSync = <span class="hljs-number">512</span>
)
</code></pre>
<p data-nodeid="889">在实现中设置的 channel 的长度是 1024。channel 一旦满了，etcd 并不会丢弃 watch 事件，而是会进行如下的操作：</p>
<pre class="lang-go" data-nodeid="890"><code data-language="go"><span class="hljs-comment">// 位于 mvcc/watchable_store.go:438</span>
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *watchableStore)</span> <span class="hljs-title">notify</span><span class="hljs-params">(rev <span class="hljs-keyword">int64</span>, evs []mvccpb.Event)</span></span> {
	<span class="hljs-keyword">var</span> victim watcherBatch
	<span class="hljs-keyword">for</span> w, eb := <span class="hljs-keyword">range</span> newWatcherBatch(&amp;s.synced, evs) {
		<span class="hljs-keyword">if</span> eb.revs != <span class="hljs-number">1</span> {
      <span class="hljs-comment">// 异常</span>
		}
		<span class="hljs-keyword">if</span> w.send(WatchResponse{WatchID: w.id, Events: eb.evs, Revision: rev}) {
			pendingEventsGauge.Add(<span class="hljs-keyword">float64</span>(<span class="hljs-built_in">len</span>(eb.evs)))
		} <span class="hljs-keyword">else</span> {
			<span class="hljs-comment">// 将 slow watchers 移动到 victims</span>
			w.minRev = rev + <span class="hljs-number">1</span>
			<span class="hljs-keyword">if</span> victim == <span class="hljs-literal">nil</span> {
				victim = <span class="hljs-built_in">make</span>(watcherBatch)
			}
			w.victim = <span class="hljs-literal">true</span>
			victim[w] = eb
			s.synced.<span class="hljs-built_in">delete</span>(w)
			slowWatcherGauge.Inc()
		}
	}
	s.addVictim(victim)
}
</code></pre>
<p data-nodeid="891">从 notify 的实现中可以知道，此 watcher 将会从 synced watcherGroup 中删除，和事件列表保存到一个名为 victim 的 watcherBatch 结构中。watcher 会记录当前的 Revision，并将自身标记为<strong data-nodeid="1016">受损</strong>，变更操作也会被保存到 watchableStore 的 victims 中。我使用如下的示例来描述上述过程：</p>
<p data-nodeid="892">channel 已满的情况下，有一个写操作写入 foo = bar。监听 foo 的 watcher 将从 synced 中移除，同时 foo=bar 也被保存到 victims 中。</p>
<p data-nodeid="893"><img src="https://s0.lgstatic.com/i/image6/M01/19/1A/Cgp9HWBJvY6AFDZcAAAvIEpMXI4453.png" alt="Drawing 3.png" data-nodeid="1020"></p>
<div data-nodeid="894"><p style="text-align:center">channel 已满时的处理流程</p></div>
<p data-nodeid="895">接下来该 watcher 不会记录对 foo 的任何变更。那么这些变更消息怎么处理呢？</p>
<p data-nodeid="896">我们知道在 channel 队列满时，变更的 Event 就会放入 victims 中。在 etcd 启动的时候，WatchableKV 模块启动了 syncWatchersLoop 和 syncVictimsLoop 两个异步协程，这两个协程用于处理不同场景下发送事件。</p>
<pre class="lang-java" data-nodeid="897"><code data-language="java"><span class="hljs-comment">// 位于 mvcc/watchable_store.go:246</span>
<span class="hljs-comment">// syncVictimsLoop 清除堆积的 Event</span>
func (s *watchableStore) syncVictimsLoop() {
	defer s.wg.Done()
	<span class="hljs-keyword">for</span> {
		<span class="hljs-keyword">for</span> s.moveVictims() != <span class="hljs-number">0</span> {
			<span class="hljs-comment">//更新所有的 victim watchers</span>
		}
		s.mu.RLock()
		isEmpty := len(s.victims) == <span class="hljs-number">0</span>
		s.mu.RUnlock()
		<span class="hljs-keyword">var</span> tickc &lt;-chan time.Time
		<span class="hljs-keyword">if</span> !isEmpty {
			tickc = time.After(<span class="hljs-number">10</span> * time.Millisecond)
		}
		select {
		<span class="hljs-keyword">case</span> &lt;-tickc:
		<span class="hljs-keyword">case</span> &lt;-s.victimc:
		<span class="hljs-keyword">case</span> &lt;-s.stopc:
			<span class="hljs-keyword">return</span>
		}
	}
}
</code></pre>
<p data-nodeid="898">syncVictimsLoop 则负责堆积的事件推送，尝试清除堆积的 Event。它会不断尝试让 watcher 发送这个 Event，一旦队列不满，watcher 将这个 Event 发出后，该 watcher 就被划入了 unsycned 中，同时不再是 victim 状态。</p>
<p data-nodeid="899">至此，syncWatchersLoop 协程就开始起作用。由于该 watcher 在 victim 状态已经落后了很多消息。为了保持同步，协程会根据 watcher 保存的 Revision，查出 victim 状态之后所有的消息，将关于 foo 的消息全部给到 watcher，当 watcher 将这些消息都发送出去后，watcher 就由 unsynced 变成 synced。</p>
<h3 data-nodeid="900">小结</h3>
<p data-nodeid="901">watch 可以用来监听一个或一组 key，key 的任何变化都会发出事件消息。某种意义上讲，etcd 也是一种发布订阅模式。</p>
<p data-nodeid="902">这一讲我们通过介绍 watch 的用法，引入对 etcd watch 机制实现的分析和讲解。watchableStore 负责了注册、管理以及触发 Watcher 的功能。watchableStore 将 watcher 划分为 synced 、unsynced 以及异常状态下的 victim 三类。在 etcd 启动时，WatchableKV 模块启动了 syncWatchersLoop 和 syncVictimsLoop 异步 goroutine，用以负责不同场景下的事件推送，并提供了事件重试机制，保证事件都能发送出去给到客户端。</p>
<p data-nodeid="903">本讲内容总结如下：</p>
<p data-nodeid="904"><img src="https://s0.lgstatic.com/i/image6/M01/19/1A/Cgp9HWBJvZyAFPLNAAIuCsjxWZQ162.png" alt="Drawing 4.png" data-nodeid="1031"></p>
<p data-nodeid="905" class="">刚刚我们说 etcd 也实现了发布订阅模式，那么它和消息中间件 Kafka 有什么异同，是否能够替换呢？欢迎你在留言区和我交流自己的想法。下一讲，我们将继续介绍 etcd Lease 租约的实现原理。</p>

---

### 精选评论


