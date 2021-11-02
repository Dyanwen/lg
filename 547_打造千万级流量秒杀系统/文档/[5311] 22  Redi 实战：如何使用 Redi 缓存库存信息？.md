<p data-nodeid="1787" class="">首先问你一个问题：秒杀系统中最重要的数据是什么？是秒杀活动信息吗？不是！是秒杀库存信息。</p>
<p data-nodeid="1788">我在需求分析的时候给你介绍过，秒杀活动之所以吸引人，是因为它用少量库存超低价的方式吸引用户。秒杀库存之所以重要，是因为库存数远小于参与秒杀活动的用户数。如果库存数据没处理好，要么影响用户体验，要么可能导致超售带来损失。所以，秒杀库存数据的高性能访问和高一致性，是秒杀系统保障性能和稳定性的关键。而为了保证这一点，一般采用具有高性能的 Redis ，来缓存库存数据。</p>
<p data-nodeid="1789">那么，具体该如何实现呢？接下来我给你详细介绍下。</p>
<h3 data-nodeid="1790">初始化 Redis</h3>
<p data-nodeid="1791">在使用 Redis 之前，我们需要初始化 Redis。假设你本地的电脑上已经启动了 Redis 服务，并且监听了 6379 端口，那么 Redis 服务的地址就是 127.0.0.1:6379。</p>
<p data-nodeid="1792">由于 Redis 性能很高，而且使用非常简单方便，如果没有密码，可以连接到 Redis 的程序能轻易获取或者修改里面的数据。因此，在一些重要的业务中，通常会给 Redis 设置访问密码。</p>
<p data-nodeid="1793">比如，可以通过<strong data-nodeid="1857">redis-cli config set requirepass abcd</strong>命令，或者在配置文件中添加 requirepass abcd 配置，将 Redis 的密码设置为 abcd。在本地电脑上，配置文件的路径通常是 /etc/redis 目录下的 redis.conf。</p>
<p data-nodeid="1794">这两种方式的区别在于：用命令的方式设置，一旦 Redis 服务重启了，则密码会失效；而用配置文件的方式能确保密码不丢失。</p>
<p data-nodeid="1795">设置完密码后，通过 redis-cli 连接到 Redis 服务后，执行任何 set、get 命令都会报错，提示你需要认证。只有执行了 auth abcd 后，才能正常读写数据。如下图所示：</p>
<p data-nodeid="1796"><img src="https://s0.lgstatic.com/i/image2/M01/08/E4/Cip5yGAM8F2AcZjGAADKLFJ5YQ8652.png" alt="Drawing 0.png" data-nodeid="1862"></p>
<p data-nodeid="1797">知道了 Redis 的地址和密码后，我们就可以在代码中初始化 Redis 客户端了。怎么做呢？</p>
<p data-nodeid="1798">首先，在 seckill 配置文件中添加 Redis 的配置，主要有地址 address 和密码 auth。如下所示：</p>
<pre class="lang-java" data-nodeid="1799"><code data-language="java">[redis]
    address = <span class="hljs-string">"127.0.0.1:6379"</span>
    auth = <span class="hljs-string">"abcd"</span>
</code></pre>
<p data-nodeid="1800">由于 Redis 属于基础设施，接下来我们在 infrastructure/stores/redis 目录下的 redis.go 中实现两个函数：<strong data-nodeid="1870">Init 和 GetClient</strong>。</p>
<p data-nodeid="1801">在 Init 函数中，主要是读取配置文件中的 address 和 auth 配置，并作为参数调用 redis.NewClient 函数创建一个 Redis Client 对象，将其保存到一个全局变量 cli 中。这样，就可以在 GetClient 函数中获取 cli 并提供给调用方使用。具体代码如下：</p>
<pre class="lang-go" data-nodeid="1802"><code data-language="go"><span class="hljs-keyword">const</span> Nil = redis.Nil
<span class="hljs-keyword">var</span> cli *redis.Client
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Init</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   addr := viper.GetString(<span class="hljs-string">"redis.address"</span>)
   auth := viper.GetString(<span class="hljs-string">"redis.auth"</span>)
   <span class="hljs-keyword">if</span> addr == <span class="hljs-string">""</span> {
      addr = <span class="hljs-string">"127.0.0.1:6379"</span>
   }
   opt := &amp;redis.Options{
      Network:  <span class="hljs-string">"tcp"</span>,
      Addr:     addr,
      Password: auth,
   }
   cli = redis.NewClient(opt)
   <span class="hljs-keyword">if</span> cli == <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> errors.New(<span class="hljs-string">"init redis client failed"</span>)
   }
   <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">GetClient</span><span class="hljs-params">()</span> *<span class="hljs-title">redis</span>.<span class="hljs-title">Client</span></span> {
   <span class="hljs-keyword">return</span> cli
}
</code></pre>
<p data-nodeid="1803">需要注意的是，redis 客户端库在发现 Key 不存在的时候，会返回 redis.Nil 错误。为了避免在判断错误的时候重复引用客户端库，我在代码中定义了一个 Nil 常量供上游调用方使用，值为 redis.Nil。</p>
<p data-nodeid="1804">接下来，我们就可以在 api 服务的启动函数中初始化 Redis。具体做法是在 interfaces/api/api.go 中调用 redis.Init 函数，并判断是否出错。代码如下所示：</p>
<pre class="lang-go" data-nodeid="1805"><code data-language="go"><span class="hljs-keyword">if</span> err := redis.Init(); err != <span class="hljs-literal">nil</span> {
   <span class="hljs-keyword">return</span> err
}
</code></pre>
<h3 data-nodeid="1806">秒杀库存的 Redis 数据结构</h3>
<p data-nodeid="1807">我们知道，Redis 是个 KV 存储，要用好 Redis，就需要设计好数据在 Redis 中的 Key 和 Value 格式。秒杀库存数据包含三个维度信息：活动场次、商品 ID、库存数量，而 Redis 的 Key 和 Value 只有两个维度。那么，我们该如何用 Redis 保存那三个维度的库存信息呢？</p>
<p data-nodeid="1808">在 Redis 中通常有两种常用的方法保存 3 个维度的信息。</p>
<p data-nodeid="1809"><strong data-nodeid="1881">第一种是采用 HashSet 数据类型。</strong> HashSet 在 Key-Value 的基础上，还多了一个 Field 参数。比如，一场活动中有多个商品，对应着多个库存数据，就可以把这场活动的所有库存信息放到一个 HashSet 里。具体来说，就是用活动场次 ID 做 Key，用商品ID 做 Field，而 Value 是库存数量。</p>
<p data-nodeid="1810">不过，为了避免不同活动系统之间的 ID 冲突，通常需要在 Key 里加个前缀，如 seckill。假如活动 ID 为 100，商品 ID 为 1001 和 1002，库存分别为 8 和 9。那么，我们可以将 Key 构造成 seckill.event.100.stocks，然后用 hmset 命令将 ID 为 1001 和 1002 这两个商品的库存保存到活动 ID 为 100 的 HashSet 中。</p>
<p data-nodeid="1811">接下来，我们就可以用 hgetall 命令获取该 HashSet 中的所有数据，或者通过 hget 命令获取商品 ID 为 1001 的库存。效果如下所示：</p>
<p data-nodeid="1812"><img src="https://s0.lgstatic.com/i/image2/M01/08/E6/CgpVE2AM8HCAQxg3AADTwImteFk399.png" alt="Drawing 1.png" data-nodeid="1886"></p>
<p data-nodeid="1813"><strong data-nodeid="1891">第二种方法是使用最常用的 String 数据类型。</strong> 我们可以将活动 ID 和商品 ID 拼接成 Key 来存储。以前面的活动和商品为例，拼接成 Key 后便是 seckill.event.100.goods.1001.stock 。接下来我们用 set 命令便可将该商品的库存保存到 Redis 中，通过 get 命令便可以取到库存。如下图所示：</p>
<p data-nodeid="1814"><img src="https://s0.lgstatic.com/i/image2/M01/08/E4/Cip5yGAM8HeAaSLGAACE-ciSroA380.png" alt="Drawing 2.png" data-nodeid="1894"></p>
<p data-nodeid="1815">这两种方法各有什么优缺点呢？我们最终选取哪种方式呢？</p>
<p data-nodeid="1816">第一种方法的好处是能一次性取出活动中所有商品的库存信息。这种方式在单节点的 Redis 中没什么问题，在 Redis 集群中它就个很致命的缺点：容易导致热 Key 问题。</p>
<p data-nodeid="1817">具体来说，主要是因为 Redis 集群是按照 Key 来将数据分配到不同节点上的。当采用 HashSet 时，一场活动的数据只会分配到一个节点。在高并发下频繁读写一场活动的库存数据，容易导致存储该活动的 Redis 节点的负载远高于其他节点，从而导致 Redis 集群节点负载不均衡，影响性能和稳定性。</p>
<p data-nodeid="1818">第二种方法虽然解决了第一种方法中的热 Key 问题，但会导致无法通过活动 ID 获取到该活动下的所有商品库存信息。</p>
<p data-nodeid="1819">怎么办呢？Redis 不像 MySQL 那样有表索引，但是，我们可以利用 Set 类型为 String 类型的数据建立一个活动与商品的映射关系。比如，构造一个名为 seckill.event.100.goods 的 Key，然后用 sadd 命令将 ID 为 1001 和 1002 这两个商品与 ID 为 100 的活动建立映射关系，通过 smembers 命令便可以获取到该场活动下的所有商品。如下图所示：</p>
<p data-nodeid="1820"><img src="https://s0.lgstatic.com/i/image2/M01/08/E6/CgpVE2AM8IOARLE4AACayxWrPr4382.png" alt="Drawing 3.png" data-nodeid="1902"></p>
<p data-nodeid="1821">在活动过程中，虽然这个映射关系会频繁使用，但不会被修改，因此我们可以将其缓存到内存中。再加上商品的库存数据分散到各个商品的 Key 中，也就大大降低了热 Key 问题的风险。</p>
<h3 data-nodeid="1822">如何用 Go 操作 Redis 中的秒杀库存？</h3>
<p data-nodeid="1823">在 DDD 中，库存是属于支撑域，用来支撑秒杀活动的。所以，我们需要将操作库存的代码逻辑放到 domain/stock 目录下的 stock.go 中。</p>
<p data-nodeid="1824">由于库存逻辑有多种实现，比如基于内存缓存的库存逻辑、基于 Redis 缓存的库存逻辑等，因此，我们需要将库存抽象为接口类 Stock。它主要包括这几个方法： Set、Get、Sub、Del、EventID、GoodsID，分别用于设置库存值、获取库存值、扣减库存值、删除库存、获取活动 ID、获取商品 ID。具体实现如下：</p>
<pre class="lang-go" data-nodeid="1825"><code data-language="go"><span class="hljs-keyword">type</span> Stock <span class="hljs-keyword">interface</span> {
   <span class="hljs-comment">// 设置库存，并设置过期时间</span>
   Set(val <span class="hljs-keyword">int64</span>, expire <span class="hljs-keyword">int64</span>) error
   <span class="hljs-comment">// 直接返回剩余库存</span>
   Get() (<span class="hljs-keyword">int64</span>, error)
   <span class="hljs-comment">// 尝试扣减一个库存，并返回剩余库存</span>
   Sub() (<span class="hljs-keyword">int64</span>, error)
   <span class="hljs-comment">// 删除库存数据</span>
   Del() error
   <span class="hljs-comment">// 返回活动 ID</span>
   EventID() <span class="hljs-keyword">string</span>
   <span class="hljs-comment">// 返回商品 ID</span>
   GoodsID() <span class="hljs-keyword">string</span>
}
</code></pre>
<p data-nodeid="1826">接下来，我们用 Go 的结构体定义基于 Redis 缓存的库存类 redisStock。它主要包括 eventID、goodsID、key 这三个字段，分别用于保存活动 ID、商品 ID、Redis 缓存中的 Key。并且，我们实现一个 NewRedisStock 函数，参数为活动 ID 和商品 ID，用于创建基于 Redis 的库存对象。代码如下：</p>
<pre class="lang-go" data-nodeid="1827"><code data-language="go"><span class="hljs-keyword">type</span> redisStock <span class="hljs-keyword">struct</span> {
   eventID <span class="hljs-keyword">string</span>
   goodsID <span class="hljs-keyword">string</span>
   key     <span class="hljs-keyword">string</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">NewRedisStock</span><span class="hljs-params">(eventID <span class="hljs-keyword">string</span>, goodsID <span class="hljs-keyword">string</span>)</span> <span class="hljs-params">(Stock, error)</span></span> {
   <span class="hljs-keyword">if</span> eventID == <span class="hljs-string">""</span> || goodsID == <span class="hljs-string">""</span> {
      <span class="hljs-keyword">return</span> <span class="hljs-literal">nil</span>, errors.New(<span class="hljs-string">"invalid event id or goods id"</span>)
   }
   stock := &amp;redisStock{
      eventID: eventID,
      goodsID: goodsID,
      key:     fmt.Sprintf(<span class="hljs-string">"seckill#%s#%s"</span>, eventID, goodsID),
   }
   <span class="hljs-keyword">return</span> stock, <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="1828">之后，我们分别实现 Stock 接口类中定义的方法。在 Set、Get、Sub、Del 方法中，我们调用 Redis 客户端的方法操作库存数据。代码示例如下所示：</p>
<pre class="lang-go" data-nodeid="1829"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">Set</span><span class="hljs-params">(val <span class="hljs-keyword">int64</span>, expiration <span class="hljs-keyword">int64</span>)</span> <span class="hljs-title">error</span></span> {
   cli := redis.GetClient()
   _, err := cli.Set(rs.key, val, time.Duration(expiration)*time.Second).Result()
   <span class="hljs-keyword">return</span> err
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">Sub</span><span class="hljs-params">()</span> <span class="hljs-params">(<span class="hljs-keyword">int64</span>, error)</span></span> {
   cli := redis.GetClient()
   <span class="hljs-keyword">return</span> cli.Decr(rs.key).Result()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">Get</span><span class="hljs-params">()</span> <span class="hljs-params">(<span class="hljs-keyword">int64</span>, error)</span></span> {
   cli := redis.GetClient()
   <span class="hljs-keyword">if</span> val, err := cli.Get(rs.key).Int64(); err != <span class="hljs-literal">nil</span> &amp;&amp; err != redis.Nil {
      <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>, err
   } <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span> val, <span class="hljs-literal">nil</span>
   }
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">Del</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   cli := redis.GetClient()
   <span class="hljs-keyword">return</span> cli.Del(rs.key).Err()
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">EventID</span><span class="hljs-params">()</span> <span class="hljs-title">string</span></span> {
   <span class="hljs-keyword">return</span> rs.eventID
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(rs *redisStock)</span> <span class="hljs-title">GoodsID</span><span class="hljs-params">()</span> <span class="hljs-title">string</span></span> {
   <span class="hljs-keyword">return</span> rs.goodsID
}
</code></pre>
<p data-nodeid="1830">最后，我们实现一个单元测试来测试一下我们实现的代码逻辑，代码在 domain/stock 目录下的 stock_test.go 中，函数为 TestStock。</p>
<p data-nodeid="1831">在单元测试中，我们先调用 redis.Init 初始化 Redis 客户端，然后调用 NewRedisStock 创建一个 Redis 库存对象，并调用 Set 函数将值设置为 10，过期时间设置为 1 秒。接着我们分别调用它的 Get、Sub、Del 方法，并校验返回值是否正确。如果不正确，就输出错误日志并退出测试。代码如下：</p>
<pre class="lang-go" data-nodeid="1832"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">TestStock</span><span class="hljs-params">(t *testing.T)</span></span> {
   <span class="hljs-keyword">var</span> (
      st  Stock
      err error
      val <span class="hljs-keyword">int64</span>
   )
   <span class="hljs-keyword">if</span> err = redis.Init(); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   }
   <span class="hljs-keyword">if</span> st, err = NewRedisStock(<span class="hljs-string">"101"</span>, <span class="hljs-string">"1001"</span>); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   }
   <span class="hljs-keyword">if</span> err = st.Set(<span class="hljs-number">10</span>, <span class="hljs-number">1</span>); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   }
   <span class="hljs-keyword">if</span> val, err = st.Get(); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> val != <span class="hljs-number">10</span> {
      t.Fatal(<span class="hljs-string">"not equal 10"</span>)
   }
   <span class="hljs-keyword">if</span> val, err = st.Sub(); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> val != <span class="hljs-number">9</span> {
      t.Fatal(<span class="hljs-string">"not equal 9"</span>)
   }
   <span class="hljs-keyword">if</span> err = st.Del(); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   }
   <span class="hljs-keyword">if</span> val, err = st.Get(); err != <span class="hljs-literal">nil</span> {
      t.Fatal(err)
   } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> val != <span class="hljs-number">0</span> {
      t.Fatal(<span class="hljs-string">"not equal 0"</span>)
   }
}
</code></pre>
<p data-nodeid="1833">在 Goland 中，点击单元测试函数 TestStock 左边的绿色小箭头，就可以运行单元测试。效果如下：</p>
<p data-nodeid="1834"><img src="https://s0.lgstatic.com/i/image2/M01/08/E4/Cip5yGAM8JOAeVYMAAD9qWhuGeI924.png" alt="Drawing 4.png" data-nodeid="1916"></p>
<p data-nodeid="2257">如果逻辑正常，你将看到 PASS TestStock 这样的日志。如果不正常，你将看到 FAIL TestStock 这样的日志。</p>
<p data-nodeid="2258" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/0A/30/Cip5yGAREI-ATftwAAbcU2Askbs147.png" alt="图片1.png" data-nodeid="2262"></p>

<h3 data-nodeid="2100">小结</h3>




<p data-nodeid="1837">这一讲我主要给你介绍了秒杀系统如何使用 Redis 存储库存数据，以及如何解决 Redis 热 Key 问题。</p>
<p data-nodeid="1838">在使用 Redis 的时候，始终要记住，Redis 不像 MySQL 那种关系型数据库，Redis 中所有数据关系需要你自己维护。你需要根据需求来决定：是否设置过期时间，过期时间多久。对于无过期时间的数据要建立映射关系，以便方便管理和清理无用数据，避免 Redis 中数据膨胀，导致资源浪费。</p>
<p data-nodeid="1839">现在你可以思考一下：秒杀系统中，库存数据的过期时间应当如何设置呢？</p>
<p data-nodeid="1840">好了，这一讲就到这里了，下一讲我将给你介绍“如何使用内存缓存提升数据命中率”。到时见！</p>
<p data-nodeid="1841">源码地址：<a href="https://github.com/lagoueduCol/MiaoSha-Yiletian" data-nodeid="1926">https://github.com/lagoueduCol/MiaoSha-Yiletian</a></p>
<hr data-nodeid="1842">
<p data-nodeid="1843"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1931"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="1930"></a></p>
<p data-nodeid="1844"><strong data-nodeid="1935">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="1845" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1939">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **5183：
> redis代码没有更新

 ###### &nbsp;&nbsp;&nbsp; 官方客服回复：
> &nbsp;&nbsp;&nbsp; 已经更新了的，在源码仓库 https://github.com/lagoueduCol/MiaoSha-Yiletian 的 infrastructure/stores/redis 和 domain/stock 目录里面，老师也在文章里面提示了具体的代码位置~

##### *佳：
> 为什么hashset不能避免集群中的热点key问题，而String能避免呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; redis集群是根据 key 来分片的，无法根据 hashset的 field 来分片。如果 hashset 里有成千上万个 field，每个 field 访问量都很高，会导致 hashset 数据量大，请求量大，会把某个 redis 实例的带宽打满。如果将hashset 的 field 都拆出来，拆分成多个 string ，比较均衡地分布在多个 redis 实例中，就能有效避免 redis 集群中负载不均衡的问题

##### *佳：
> 您好，我看这里redis缓存的秒杀相关的数据是不是有点少了1、秒杀商品详情 这也应该要缓存起来吧，比如用String这种数据结构，key是秒杀商品id2、秒杀场次列表 比如List这种结构key是当前租户，value是所有场次3、秒杀场次下的商品列表 List这种结构，key是具体场次，value是商品列表另外有个问题：比如 缓存了商品列表，是否还需要缓存单个单个的商品详情呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这些都需要缓存。但商品相关的缓存逻辑比较简单，因为它在活动过程中是只读。而库存是同时涉及读写操作的，实现难度比商品大。

