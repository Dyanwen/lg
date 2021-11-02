<p data-nodeid="573" class="">欢迎来到多级缓存实战模块！</p>
<p data-nodeid="574">首先请你思考这样一个业务场景：一个有 10 个节点的集群，当你修改了某一项配置后，你希望能立即同步给这 10 个节点，并更新到各节点的缓存中，该怎么做？</p>
<p data-nodeid="575">一种方法是你获取到这 10 个节点的地址信息，然后逐个调用接口同步给它们。但这会带来新的问题：你如何准确获取这 10 个节点的地址信息，又如何将配置稳定可靠地同步给它们？如果集群部署到多个可用区，且是用容器部署的，你拿到的地址不一定是最新的、完全可用的地址，最后很可能因部分节点同步失败而导致节点数据不一致。</p>
<p data-nodeid="576">另一种方法是让这 10 个节点自己来取。那怎么让它们知道配置变更了呢？它们又该去哪儿获取最新配置呢？</p>
<p data-nodeid="577">把这个问题抽象下就是：A 系统修改了数据，需要即时同步给 B、C、D 系统的所有节点。显然，这是非常经典的数据同步问题。想要解决，我们可以引入一个分布式系统协调器，如 etcd来处理。</p>
<p data-nodeid="578">在秒杀系统中，我们使用 etcd 存储集群信息和活动信息，以此解决分布式系统中的数据同步问题。除此之外，在多级缓存中我们还有一种为 Redis 缓存可用性兜底的手段。具体是怎么做的呢？</p>
<h3 data-nodeid="579">etcd 初始化</h3>
<p data-nodeid="580">在使用 etcd 存储集群信息和活动信息前，我们需要先对 etcd 进行初始化。具体来说，在 infrastructure/stores/etcd 目录下的 etcd.go 中，我实现了 Init 和 GetClient 这两个函数。其中，在 Init 函数里，获取 etcd 相关的配置，并初始化 etcd 客户端，用 sync.Once 确保只初始化一次；在 GetClient 函数里，返回 Init 函数中初始化好的客户端。具体代码如下：</p>
<pre class="lang-go" data-nodeid="581"><code data-language="go"><span class="hljs-keyword">var</span> etcdCli *etcd.Client
<span class="hljs-keyword">var</span> etcdOnce = &amp;sync.Once{}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Init</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   <span class="hljs-keyword">var</span> err error
   etcdOnce.Do(
      <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
         endpoints := viper.GetStringSlice(<span class="hljs-string">"etcd.endpoints"</span>)
         username := viper.GetString(<span class="hljs-string">"etcd.username"</span>)
         password := viper.GetString(<span class="hljs-string">"etcd.password"</span>)
         cfg := etcd.Config{
            Endpoints:   endpoints,
            DialTimeout: time.Second,
            Username:    username,
            Password:    password,
         }
         etcdCli, err = etcd.New(cfg)
      })
   <span class="hljs-keyword">return</span> err
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">GetClient</span><span class="hljs-params">()</span> *<span class="hljs-title">etcd</span>.<span class="hljs-title">Client</span></span> {
   <span class="hljs-keyword">return</span> etcdCli
}
</code></pre>
<p data-nodeid="582">接下来，我在 cmd/root.go 中调用 etcd.Init ，初始化 etcd 客户端。代码如下：</p>
<pre class="lang-java" data-nodeid="583"><code data-language="java"><span class="hljs-keyword">if</span> err := etcd.Init(); err != nil {
   panic(err)
}
</code></pre>
<h3 data-nodeid="584">存储集群信息</h3>
<p data-nodeid="585">既然要用 etcd 存储集群信息，那么集群信息都有哪些呢？</p>
<p data-nodeid="586">总的来说，秒杀系统的集群信息主要是秒杀服务节点信息，比如服务地址、版本号、协议类型等。在秒杀系统集群内，主要是将秒杀 API 服务的节点信息提供给 Admin 服务，用于调用 API 服务的 RPC 接口同步配置；在秒杀系统集群外使用节点信息，主要通过服务发现的方式，将节点暴露给监控系统或者其他微服务调用。</p>
<p data-nodeid="587">在 etcd 中，我们可以将节点信息存放到 /seckill/nodes 这个 key 下。以 IP 为 10.10.11.12 的节点为例，我们用 addr 字段保存服务地址，用 proto 保存协议类型，用 version 字段保存版本号。由于节点信息只会在节点启动时变更，我们用 json 格式来存放，具体的数据示例如下：</p>
<pre class="lang-java" data-nodeid="588"><code data-language="java">{
  <span class="hljs-string">"addr"</span>: <span class="hljs-string">"10.10.11.12:8080"</span>,
  <span class="hljs-string">"proto"</span>: <span class="hljs-string">"http"</span>,
  <span class="hljs-string">"version"</span>: <span class="hljs-string">"v1.0"</span>
}
</code></pre>
<p data-nodeid="589">我们可以用 Go 结构体来定义一个新类型 Node，用于处理 json 格式的节点信息，字段名跟前面提到的 json 格式保持一致。如下所示：</p>
<pre class="lang-go" data-nodeid="590"><code data-language="go"><span class="hljs-keyword">type</span> Node <span class="hljs-keyword">struct</span> {
   Addr    <span class="hljs-keyword">string</span> <span class="hljs-string">`json:"addr"`</span>
   Version <span class="hljs-keyword">string</span> <span class="hljs-string">`json:"version"`</span>
   Proto   <span class="hljs-keyword">string</span> <span class="hljs-string">`json:"proto"`</span>
}
</code></pre>
<p data-nodeid="591"><strong data-nodeid="657">节点信息的管理需要使用三个函数：Register、Deregister 和 Discover，它们分别用于服务节点的注册、注销和发现。</strong></p>
<p data-nodeid="592">具体怎么实现呢？</p>
<p data-nodeid="593">第一步，我定义了个 cluster 结构体，用于保存集群管理过程中需要的所有信息，包括锁、etcd 客户端、节点列表等，然后在 Init 函数中初始化它。具体代码如下所示：</p>
<pre class="lang-go" data-nodeid="594"><code data-language="go"><span class="hljs-keyword">type</span> cluster <span class="hljs-keyword">struct</span> {
   sync.RWMutex
   cli     *etcdv3.Client
   service <span class="hljs-keyword">string</span>
   once    *sync.Once
   deregCh <span class="hljs-keyword">map</span>[<span class="hljs-keyword">string</span>]<span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}
   nodes   <span class="hljs-keyword">map</span>[<span class="hljs-keyword">string</span>]*Node
   v       *viper.Viper
}
<span class="hljs-keyword">var</span> defaultCluster *cluster
<span class="hljs-keyword">var</span> once = &amp;sync.Once{
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Init</span><span class="hljs-params">(service <span class="hljs-keyword">string</span>)</span></span> {
   once.Do(<span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      defaultCluster = &amp;cluster{
         cli:     etcd.GetClient(),
         service: service,
         once:    &amp;sync.Once{},
         deregCh: <span class="hljs-built_in">make</span>(<span class="hljs-keyword">map</span>[<span class="hljs-keyword">string</span>]<span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{}),
         nodes:   <span class="hljs-built_in">make</span>(<span class="hljs-keyword">map</span>[<span class="hljs-keyword">string</span>]*Node),
      }
   })
}
</code></pre>
<p data-nodeid="595">第二步，我实现了 Register、Deregister 和 Discover 这三个函数。</p>
<p data-nodeid="596">具体来说，我在 Register 函数中将节点信息注册到 etcd，设置一个有效期，并定期更新，确保节点信息是最新的。在 Deregister 函数中将注册到 etcd 的节点信息删除。在 Discover 函数中，监控节点变更，并及时更新本地内存缓存中的节点信息，将最新的节点信息返回给调用者。代码如下：</p>
<pre class="lang-go" data-nodeid="597"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Register</span><span class="hljs-params">(node *Node, ttl <span class="hljs-keyword">int</span>)</span> <span class="hljs-title">error</span></span> {
   <span class="hljs-keyword">const</span> minTTL = <span class="hljs-number">2</span>
   c := defaultCluster
   key := c.makeKey(node)
   <span class="hljs-keyword">if</span> ttl &lt; minTTL {
      ttl = minTTL
   }
   <span class="hljs-keyword">var</span> errCh = <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> error)
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      kv := etcdv3.NewKV(c.cli)
      closeCh := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> <span class="hljs-keyword">struct</span>{})
      lease := etcdv3.NewLease(c.cli)
      val, _ := json.Marshal(node)
      <span class="hljs-keyword">var</span> curLeaseId etcdv3.LeaseID = <span class="hljs-number">0</span>
      ticker := time.NewTicker(time.Duration(ttl/<span class="hljs-number">2</span>) * time.Second)
      register := <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
         <span class="hljs-keyword">if</span> curLeaseId == <span class="hljs-number">0</span> {
            leaseResp, err := lease.Grant(context.TODO(), <span class="hljs-keyword">int64</span>(ttl))
            <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
               <span class="hljs-keyword">return</span> err
            }
            <span class="hljs-keyword">if</span> _, err := kv.Put(context.TODO(), key, <span class="hljs-keyword">string</span>(val), etcdv3.WithLease(leaseResp.ID)); err != <span class="hljs-literal">nil</span> {
               <span class="hljs-keyword">return</span> err
            }
            curLeaseId = leaseResp.ID
         } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">if</span> _, err := lease.KeepAliveOnce(context.TODO(), curLeaseId); err == rpctypes.ErrLeaseNotFound {
               curLeaseId = <span class="hljs-number">0</span>
            }
         }
         <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
      }
      <span class="hljs-keyword">if</span> err := register(); err != <span class="hljs-literal">nil</span> {
         logrus.Error(<span class="hljs-string">"register node failed, error:"</span>, err)
         errCh &lt;- err
      }
      <span class="hljs-built_in">close</span>(errCh)
      <span class="hljs-keyword">for</span> {
         <span class="hljs-keyword">select</span> {
         <span class="hljs-keyword">case</span> &lt;-ticker.C:
            <span class="hljs-keyword">if</span> err := register(); err != <span class="hljs-literal">nil</span> {
               logrus.Error(<span class="hljs-string">"register node failed, error:"</span>, err)
               <span class="hljs-built_in">panic</span>(err)
            }
         <span class="hljs-keyword">case</span> &lt;-closeCh:
            ticker.Stop()
            <span class="hljs-keyword">return</span>
         }
      }
   }()
   err := &lt;-errCh
   <span class="hljs-keyword">return</span> err
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Deregister</span><span class="hljs-params">(node *Node)</span> <span class="hljs-title">error</span></span> {
   c := defaultCluster
   c.Lock()
   <span class="hljs-keyword">defer</span> c.Unlock()
   key := c.makeKey(node)
   <span class="hljs-keyword">if</span> ch, ok := c.deregCh[key]; ok {
      <span class="hljs-built_in">close</span>(ch)
      <span class="hljs-built_in">delete</span>(c.deregCh, key)
   }
   _, err := c.cli.Delete(context.Background(), key, etcdv3.WithPrefix())
   <span class="hljs-keyword">return</span> err
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Discover</span><span class="hljs-params">()</span> <span class="hljs-params">(output []*Node, err error)</span></span> {
   c := defaultCluster
   key := fmt.Sprintf(<span class="hljs-string">"/%s/nodes/"</span>, c.service)
   c.once.Do(<span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      <span class="hljs-keyword">var</span> resp *etcdv3.GetResponse
      resp, err = c.cli.Get(context.Background(), key, etcdv3.WithPrefix())
      <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
         <span class="hljs-keyword">return</span>
      }
      <span class="hljs-keyword">for</span> _, kv := <span class="hljs-keyword">range</span> resp.Kvs {
         k := <span class="hljs-keyword">string</span>(kv.Key)
         <span class="hljs-keyword">if</span> <span class="hljs-built_in">len</span>(k) &gt; <span class="hljs-built_in">len</span>(key) {
            <span class="hljs-keyword">var</span> node *Node
            json.Unmarshal(kv.Value, &amp;node)
            <span class="hljs-keyword">if</span> node != <span class="hljs-literal">nil</span> {
               c.Lock()
               c.nodes[k] = node
               c.Unlock()
            }
         }
      }
      watchCh := c.cli.Watch(context.Background(), key, etcdv3.WithPrefix())
      <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
         <span class="hljs-keyword">for</span> {
            <span class="hljs-keyword">select</span> {
            <span class="hljs-keyword">case</span> resp := &lt;-watchCh:
               <span class="hljs-keyword">for</span> _, evt := <span class="hljs-keyword">range</span> resp.Events {
                  k := <span class="hljs-keyword">string</span>(evt.Kv.Key)
                  <span class="hljs-keyword">if</span> <span class="hljs-built_in">len</span>(k) &lt;= <span class="hljs-built_in">len</span>(key) {
                     <span class="hljs-keyword">continue</span>
                  }
                  <span class="hljs-keyword">switch</span> evt.Type {
                  <span class="hljs-keyword">case</span> etcdv3.EventTypePut:
                     <span class="hljs-keyword">var</span> node *Node
                     json.Unmarshal(evt.Kv.Value, &amp;node)
                     <span class="hljs-keyword">if</span> node != <span class="hljs-literal">nil</span> {
                        c.Lock()
                        c.nodes[k] = node
                        c.Unlock()
                     }
                  <span class="hljs-keyword">case</span> etcdv3.EventTypeDelete:
                     c.Lock()
                     <span class="hljs-keyword">if</span> _, ok := c.nodes[k]; ok {
                        <span class="hljs-built_in">delete</span>(c.nodes, k)
                     }
                     c.Unlock()
                  }
               }
            }
         }
      }()
   })
   <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>, err
   }
   c.RLock()
   <span class="hljs-keyword">for</span> _, node := <span class="hljs-keyword">range</span> c.nodes {
      output = <span class="hljs-built_in">append</span>(output, node)
   }
   c.RUnlock()
   <span class="hljs-keyword">return</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(c *cluster)</span> <span class="hljs-title">makeKey</span><span class="hljs-params">(node *Node)</span> <span class="hljs-title">string</span></span> {
   id := strings.Replace(node.Addr, <span class="hljs-string">"."</span>, <span class="hljs-string">"-"</span>, <span class="hljs-number">-1</span>)
   id = strings.Replace(id, <span class="hljs-string">":"</span>, <span class="hljs-string">"-"</span>, <span class="hljs-number">-1</span>)
   <span class="hljs-keyword">return</span> fmt.Sprintf(<span class="hljs-string">"/%s/nodes/%s"</span>, c.service, id)
}
</code></pre>
<p data-nodeid="598">第三步，修改 interfaces/rpc 目录下的 rpc.go，在 Run 函数中加上注册节点信息的代码，其中初始化一个变量 node，如下所示：</p>
<pre class="lang-go" data-nodeid="599"><code data-language="go"><span class="hljs-comment">//初始化集群</span>
cluster.Init(<span class="hljs-string">"seckill"</span>)
<span class="hljs-keyword">var</span> addr <span class="hljs-keyword">string</span>
<span class="hljs-keyword">if</span> addr, err = utils.Extract(bind); err == <span class="hljs-literal">nil</span> {
   <span class="hljs-comment">//注册节点信息</span>
   version := viper.GetString(<span class="hljs-string">"api.version"</span>)
   <span class="hljs-keyword">if</span> version == <span class="hljs-string">""</span> {
      version = <span class="hljs-string">"v0.1"</span>
   }
   once.Do(<span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      node = &amp;cluster.Node{
         Addr:    addr,
         Version: version,
         Proto:   <span class="hljs-string">"gRPC"</span>,
      }
      err = cluster.Register(node, <span class="hljs-number">6</span>)
   })
}
<span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
   <span class="hljs-keyword">return</span> err
}
</code></pre>
<p data-nodeid="600">另外，我还在 Exit 函数中加上了注销节点的代码，主要是调用 cluster.Deregister 函数将节点信息从 etcd 中删除。</p>
<p data-nodeid="601">第四步，在 Run 函数里初始化集群，并加上一段代码用于输出获取到的节点信息。代码如下：</p>
<pre class="lang-go" data-nodeid="602"><code data-language="go">cluster.Init(<span class="hljs-string">"seckill"</span>)
<span class="hljs-keyword">if</span> nodes, err := cluster.Discover(); err == <span class="hljs-literal">nil</span> {
   log, _ := json.Marshal(nodes)
   logrus.Info(<span class="hljs-string">"discover nodes "</span>, <span class="hljs-keyword">string</span>(log))
} <span class="hljs-keyword">else</span> {
   logrus.Error(<span class="hljs-string">"discover nodes error:"</span>, err)
}
</code></pre>
<p data-nodeid="603">注意，这个 Run 函数是位于 interfaces/admin 目录下的 admin.go 里面哦。</p>
<p data-nodeid="604">第五步，编译成功后，我们先启动 api，然后启动 admin。可以看到 admin 中输出了节点信息日志，说明 admin 服务成功发现了 api 服务的节点。此时，使用 etcdctl 也能获取到节点信息。</p>
<p data-nodeid="605">为了验证节点注册功能是否正常工作，我用 pkill -9 seckill 命令将 api 和 admin 都杀掉，使用 etcdctl 前两次还能获取到节点信息，但后面就获取不到了。就是因为杀掉 api 服务后，服务停止了更新过期时间，导致 etcd 将过期的节点信息删除。这也就证明了服务节点注册功能是正常工作的。</p>
<p data-nodeid="606">运行效果如下图所示：</p>
<p data-nodeid="607"><img src="https://s0.lgstatic.com/i/image/M00/8F/73/CgqCHmAH4OWATNp8AARKiB3EUIA085.png" alt="Drawing 0.png" data-nodeid="671"></p>
<h3 data-nodeid="608">存储集群配置</h3>
<p data-nodeid="609">秒杀集群配置主要有日志等级、限流器速度、熔断条件等。它们存放到 etcd 中 /seckill/config 这个 key 下，其中的 logLevel 保存日志等级， rateLimit 保存中间层和底层限流器的速度， circuitBreaker 保存熔断条件中的 cpu 使用率和请求时延。具体配置示例如下：</p>
<pre class="lang-java" data-nodeid="610"><code data-language="java">{
  <span class="hljs-string">"logLevel"</span>:<span class="hljs-string">"info"</span>,
  <span class="hljs-string">"rateLimit"</span>: {
    <span class="hljs-string">"middle"</span>: <span class="hljs-number">100000</span>,
    <span class="hljs-string">"low"</span>: <span class="hljs-number">10000</span>,
  }
  <span class="hljs-string">"circuitBreaker"</span>:{
    <span class="hljs-string">"cpu"</span>: <span class="hljs-number">80</span>,
    <span class="hljs-string">"latency"</span>: <span class="hljs-number">1000</span>,
  }
}
</code></pre>
<p data-nodeid="611">对应到 Go 代码中，大致步骤如下：</p>
<ol data-nodeid="612">
<li data-nodeid="613">
<p data-nodeid="614">将上面的 json 配置用一个 Config 结构体保存，字段名跟 json 中的保持一致，然后把它放在 infrastructure/cluster 目录下的 cluster.go 中；</p>
</li>
<li data-nodeid="615">
<p data-nodeid="616">实现一个 WatchClusterConfig 函数，主要是监控并处理 etcd 中集群配置的 Delete 和 Put 事件，并及时更新内存缓存中的配置数据；</p>
</li>
<li data-nodeid="617">
<p data-nodeid="618">实现一个 GetClusterConfig 函数，用于将缓存中的配置提供给其他模块使用。</p>
</li>
</ol>
<p data-nodeid="619">注意，在更新和读取缓存中配置数据的时候，都需要加锁，以免获取到的配置数据不是最新的。</p>
<p data-nodeid="620">代码如下所示：</p>
<pre class="lang-go" data-nodeid="621"><code data-language="go"><span class="hljs-keyword">type</span> Config <span class="hljs-keyword">struct</span> {
   LogLevel  <span class="hljs-keyword">string</span> <span class="hljs-string">`json:"logLevel"`</span>
   RateLimit <span class="hljs-keyword">struct</span> {
      Middle <span class="hljs-keyword">int</span> <span class="hljs-string">`json:"middle"`</span>
      Low    <span class="hljs-keyword">int</span> <span class="hljs-string">`json:"low"`</span>
   } <span class="hljs-string">`json:"rateLimit"`</span>
   CircuitBreaker <span class="hljs-keyword">struct</span> {
      Cpu     <span class="hljs-keyword">int</span> <span class="hljs-string">`json:"cpu"`</span>
      Latency <span class="hljs-keyword">int</span> <span class="hljs-string">`json:"latency"`</span>
   } <span class="hljs-string">`json:"circuitBreaker"`</span>
}
<span class="hljs-keyword">var</span> configLock = &amp;sync.RWMutex{}
<span class="hljs-keyword">var</span> config = &amp;Config{}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">WatchClusterConfig</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   cli := etcd.GetClient()
   key := <span class="hljs-string">"/seckill/config"</span>
   resp, err := cli.Get(context.Background(), key)
   <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> err
   }
   update := <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(kv *mvccpb.KeyValue)</span> <span class="hljs-params">(<span class="hljs-keyword">bool</span>, error)</span></span> {
      <span class="hljs-keyword">if</span> <span class="hljs-keyword">string</span>(kv.Key) == key {
         <span class="hljs-keyword">var</span> tmpConfig *Config
         err = json.Unmarshal(kv.Value, &amp;tmpConfig)
         <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
            logrus.Error(<span class="hljs-string">"update cluster config failed, error:"</span>, err)
            <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>, err
         }
         configLock.Lock()
         *config = *tmpConfig
         logrus.Info(<span class="hljs-string">"update cluster config "</span>, *config)
         configLock.Unlock()
         <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>, <span class="hljs-literal">nil</span>
      }
      <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>, <span class="hljs-literal">nil</span>
   }
   <span class="hljs-keyword">for</span> _, kv := <span class="hljs-keyword">range</span> resp.Kvs {
      <span class="hljs-keyword">if</span> ok, err := update(kv); ok {
         <span class="hljs-keyword">break</span>
      } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
         <span class="hljs-keyword">return</span> err
      }
   }
   <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
      watchCh := cli.Watch(context.Background(), key)
      <span class="hljs-keyword">for</span> resp := <span class="hljs-keyword">range</span> watchCh {
         <span class="hljs-keyword">for</span> _, evt := <span class="hljs-keyword">range</span> resp.Events {
            <span class="hljs-keyword">if</span> evt.Type == etcdv3.EventTypePut {
               <span class="hljs-keyword">if</span> ok, err := update(evt.Kv); ok {
                  <span class="hljs-keyword">break</span>
               } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
                  <span class="hljs-keyword">break</span>
               }
            }
         }
      }
   }()
   <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">GetClusterConfig</span><span class="hljs-params">()</span> <span class="hljs-title">Config</span></span> {
   configLock.RLock()
   <span class="hljs-keyword">defer</span> configLock.RUnlock()
   <span class="hljs-keyword">return</span> *config
}
</code></pre>
<p data-nodeid="622">接下来，为了能同时启动两个 api 服务，我先将 config/seckill.toml 配置文件拷贝一份，新的配置文件为 config/seckill1.toml，将里面的端口号和 pid 文件修改成与老的配置不一样，比如端口号改成 8083 和 8084，pid 文件改成 .pid1。</p>
<p data-nodeid="623">现在我们用这两份配置文件启动两个 api 服务，然后使用下面的 etcdctl 命令将配置数据写入到 etcd 。</p>
<pre class="lang-powershell" data-nodeid="624"><code data-language="powershell">ETCDCTL_API=<span class="hljs-number">3</span> etcdctl put /seckill/config <span class="hljs-string">'{"logLevel":"info","rateLimit":{"middle":100000,"low":10000},"circuitBreaker":{"cpu":80,"latency":1000}}'</span>
</code></pre>
<p data-nodeid="625">此时我们就能在两个运行 api 服务的命令行终端看到 update cluster config 这样的日志，而且它们的时间戳一样，说明它们几乎能同时接收到配置变更事件，并在日志中输出最新的配置。</p>
<p data-nodeid="626">效果如下图所示：</p>
<p data-nodeid="1014"><img src="https://s0.lgstatic.com/i/image/M00/8F/68/Ciqc1GAH4PaAC-tXAALovDi42Kc380.png" alt="Drawing 1.png" data-nodeid="1018"></p>
<p data-nodeid="1015" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/07/A8/Cip5yGAI5D-AdpWlAAlo2hWmewc294.png" alt="图片1.png" data-nodeid="1021"></p>

<h3 data-nodeid="866">小结</h3>




<p data-nodeid="629">这一讲我主要介绍了如何用 etcd 存储集群信息和集群配置。实际上，集群信息和配置管理有很多种方法，比如你还可以用 consul 来实现配置管理、服务注册和发现，那可能会比用 etcd 更简单些。</p>
<p data-nodeid="630">我之所以在这里用 etcd ，主要是为了给你介绍集群配置管理、服务注册和发现的底层代码原理。当你真正学会了自己去实现一套类似的代码后，你在使用其他方案时将会更容易理解它们的原理。</p>
<p data-nodeid="631">另外，还需要注意一点，我在这里用的 etcd 版本是 v3，在代码和命令行上与 v2 有些差别。</p>
<p data-nodeid="632"><strong data-nodeid="695">思考题：</strong> 如果用 consul 来实现集群信息和配置管理，该如何做呢？</p>
<p data-nodeid="633">期待你在留言区回答。</p>
<p data-nodeid="634">好了，这一讲就到这里了，下一讲我将给你介绍“如何使用 Redis 缓存库存信息”。到时见！</p>
<p data-nodeid="635">源码地址：<a href="https://github.com/lagoueduCol/MiaoSha-Yiletian" data-nodeid="701">https://github.com/lagoueduCol/MiaoSha-Yiletian</a></p>
<hr data-nodeid="636">
<p data-nodeid="637"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="706"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="705"></a></p>
<p data-nodeid="638"><strong data-nodeid="710">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="639" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="714">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **耀：
> 老师我很喜欢你的课程，每次都能学到很多知识，谢谢老师。

##### **0835：
> 老师，问一下，这些配置信息，可以用redis来做存储吗？不是redis的性能更好一些吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Redis性能虽然好，但无法保证数据一致和可靠。ETCD是会将数据落磁盘的，会有WAL日志，基于WAL之上用RAFT确保数据一致，不担心数据丢失。

