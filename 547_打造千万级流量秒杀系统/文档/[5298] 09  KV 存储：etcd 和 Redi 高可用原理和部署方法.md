<p data-nodeid="66026">在第 05 讲秒杀系统领域建模部分，我曾提到秒杀场次信息是聚合根，它聚合了秒杀商品信息和秒杀专题信息。假如我们要从关系型数据库中提取场次详情，意味着需要访问三张表：活动专题表、活动场次表、活动商品表。</p>



<p data-nodeid="65210">这会出现什么问题呢？一旦遇到高并发情况，数据库就会承受很高的访问压力甚至瘫痪。</p>
<p data-nodeid="65211">那有没有办法解决这个问题，提高数据访问的高性能和高可用？有！那就是使用 KV 存储，这也是本讲的主题。</p>
<h3 data-nodeid="65212">什么是 KV 存储</h3>
<p data-nodeid="65213">KV 是 Key-Value 的缩写，KV 存储也叫键值对存储。<strong data-nodeid="65315">简单来说，它是利用 Key 做索引来实现数据的存储、修改、查询和删除功能。</strong></p>
<p data-nodeid="65214">常用的高性能 KV 存储主要有 Redis、Memcached、etcd、Zookeeper 等，其中 Redis 和 Memcached 主要用来缓存业务数据， etcd 和 Zookeeper 主要用来存储元数据。</p>
<p data-nodeid="65215">业务数据比较好理解，就是业务系统业务逻辑处理的数据。比如我们要在 API 服务中将活动信息快速读取出来，就需要用 Redis 做缓存，降低耗时，提升数据的吞吐能力。</p>
<p data-nodeid="65216">那什么是元数据呢？在分布式系统中，元数据就是系统的基本信息，包括系统名称、服务名称、服务配置、节点 IP 等。比如秒杀系统中，秒杀服务调用商品中心和交易中心的时候，就需要从服务注册中心里获取这两个系统的元数据。</p>
<p data-nodeid="65217">以上是对 KV 存储的基本介绍，一般在秒杀系统中，我们会利用 Redis 和 etcd 实现多级缓存和漏斗模型中的部分功能，比如缓存活动信息、存储秒杀 API 服务节点信息、利用分布式锁和消息队列将请求串行化等。</p>
<p data-nodeid="65218">所以，接下来我就为你着重介绍下 KV 存储中 Redis 和 etcd 的高可用原理及其部署方法。</p>
<h3 data-nodeid="65219">Redis 高可用原理及部署方法</h3>
<h4 data-nodeid="65220">Redis 的高可用原理</h4>
<p data-nodeid="65221">你可能会问了，既然 Redis 和 Memcached 都可以缓存业务数据，那为什么会选择 Redis 而不是 Memcached 呢？</p>
<p data-nodeid="65222">其实 Memcached 在性能上要稍微比 Redis 好，但在易用性和可用性上，Redis 要大大超过Memcached 。</p>
<p data-nodeid="65223">先说易用性。Redis 有五种数据类型：list 、set 、string 、hash 、zset。这表示在使用 Redis 存储数据的时候将会更灵活，能节省很多开发成本。而 Memcached 支持的数据类型比较简单，只有 string，无法满足复杂业务场景的需求。</p>
<p data-nodeid="65224">另外，Redis 还支持原子操作和事务，可以确保操作数据时的准确性，使用非常简单。比如在秒杀中扣减和归还库存，就可以用 Redis 的原子操作和事务来保障库存数据的准确性。</p>
<p data-nodeid="65225">在可用性方面呢，Redis 支持 Master-Slave 模式的数据备份，而 Memcached 不支持； 在故障转移方面，Redis 的可用性也比 Memcached 高。</p>
<p data-nodeid="65226">还有，Memcached 是纯内存的，崩溃后里面存储的信息就丢失了，而 Redis 则可以通过命令将内存中的数据保存到磁盘上，让数据持久化。</p>
<p data-nodeid="65227">换句话说，即使 Redis 节点因为宕机导致内存中的数据丢失，它也可以从磁盘恢复数据到内存，这就大大降低了因宕机导致缓存穿透的风险。应用到秒杀系统里就是，假如秒杀活动进行中，缓存活动信息的 Redis 挂了，由于数据做了持久化，就避免了大量请求直接到 DB。</p>
<p data-nodeid="65228">那 Redis 是如何做到高可用的呢？</p>
<p data-nodeid="66566" class=""><strong data-nodeid="66571">它主要通过支持主从模式、哨兵模式、集群模式这三种模式，来满足不同业务特点和可用等级的需求。</strong> 其中，主从模式部署最简单，用得也最多，集群模式比较复杂，但可用性最高。具体怎么部署呢？</p>

<h4 data-nodeid="65230">Redis 的部署方法</h4>
<p data-nodeid="65231"><strong data-nodeid="65339">主从模式</strong></p>
<p data-nodeid="65232">主从模式比较简单，可部署多个节点，其中一个作为 Master 节点，剩下的作为 Slave 节点。它的示意图如下：</p>
<p data-nodeid="67112" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/25/Ciqc1F_QgPOAaL8TAAC5EiNlvo4795.png" alt="Drawing 1.png" data-nodeid="67115"></p>



<p data-nodeid="65236">在主从模式里，客户端同时连接 Master 节点和 Slave 节点，写操作通过 Master 节点执行，并将结果同步给 Slave 节点，读操作通过 Slave 节点执行。假如 Master 节点挂了，不影响读操作，我们可以通过手动修改配置将某个 Slave 节点提升为 Master 节点，重新提供写能力。</p>
<p data-nodeid="65237">主从模式最大的优点是部署简单，最少两个节点便可以构成主从模式，并且可以通过读写分离避免读和写同时不可用。不过，一旦 Master 节点出现故障，主从节点就无法自动切换，直接导致 SLA 下降。所以，<strong data-nodeid="65353">主从模式一般适合业务发展初期，并发量低，运维成本低的情况。</strong></p>
<p data-nodeid="65238">后来，为了避免主从无法自动切换的问题，又出现了一种新模式——哨兵模式，它是主从模式的升级。</p>
<p data-nodeid="65239"><strong data-nodeid="65358">哨兵模式</strong></p>
<p data-nodeid="65240">哨兵模式是如何部署的呢？请看下图：</p>
<p data-nodeid="68184" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/26/Ciqc1F_QgQCAHUvXAAD-y-SanS4837.png" alt="Drawing 3.png" data-nodeid="68187"></p>



<p data-nodeid="65244">相比主从模式，哨兵模式新增了独立部署的节点——哨兵节点（Sentinel）。这些节点虽然不参与数据处理，但它们会像哨兵一样负责监控 Master 和 Slave 的状态及拓扑关系，并把主从关系信息提供给客户端。客户端在连接 Redis 的时候，会先连接哨兵节点，获取 Master 和 Slave 信息，然后再连接 Master 和 Slave。</p>
<p data-nodeid="65245">在三种模式当中，哨兵模式是 Redis 官方推荐的高可用模式，那它是如何做到高可用的呢？</p>
<p data-nodeid="65246">首先，哨兵集群会通过 Ping 和 Info 请求监控所有 Redis 节点的状态，并且哨兵集群内会选举出 Leader 节点。当 Master 节点挂了后，哨兵集群内部会进行投票，如果超过半数节点确认 Master 节点挂了，则会由哨兵集群的 Leader 节点选择一个 Slave 节点进行主从切换。最后，哨兵集群会将新的主从信息通知给客户端，让客户端连接到新的 Master 和 Slave 节点上。</p>
<p data-nodeid="68704" class=""><strong data-nodeid="68709">哨兵模式适合读请求远多于写请求的业务场景，比如在秒杀系统中用来缓存活动信息。</strong> 如果写请求较多，当集群 Slave 节点数量多了后，Master 节点同步数据的压力会非常大。怎么办呢？为了解决写请求较多的业务场景， Redis 又提供了集群模式。</p>

<p data-nodeid="65248"><strong data-nodeid="65377">集群模式</strong></p>
<p data-nodeid="65249">集群模式具体是怎么部署的呢？请看下图：</p>
<p data-nodeid="69228" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/31/CgqCHl_QgQyAT1M3AACihlPFlYc982.png" alt="Drawing 5.png" data-nodeid="69231"></p>



<p data-nodeid="65253">为了避免单一节点负载过高导致不稳定，集群模式采用一致性哈希算法将 Key 分布到各个节点上。其中，每个 Master 节点后跟若干个 Slave 节点，用于出现故障时做主备切换。</p>
<p data-nodeid="65254">在这种模式下，Master 节点可以同时处理读和写请求。具体来说，客户端可以连接任意 Master 节点，集群内部会按照不同 key 将请求转发到不同的 Master 节点。当然，为了减少 Master 内部转发请求的压力，也可以选择在客户端连接所有 Master 节点，直接在客户端将请求哈希到对应的 Master 节点。</p>
<p data-nodeid="65255">集群模式是如何实现高可用的呢？集群内部节点之间会互相定时探测对方是否存活，如果多数节点判断某个节点挂了，则会将其踢出集群，然后从 Slave 节点中选举出一个节点替补挂掉的 Master 节点。</p>
<p data-nodeid="65256">虽然集群模式避免了 Master 单节点的问题，但集群内同步数据时会占用一定的带宽。所以，只有在写操作比较多的情况下人们才使用集群模式，其他大多数情况，使用哨兵模式都能满足需求。</p>
<p data-nodeid="65257">以上便是 Redis 常用的几种部署方法，不知道你是否掌握了呢？聊完 Redis，接下来我为你介绍下 etcd。</p>
<p data-nodeid="69738" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/26/Ciqc1F_QgRWAGBltAANgSUoQQRo882.png" alt="Drawing 6.png" data-nodeid="69741"></p>

<h3 data-nodeid="65259">etcd 高可用原理及部署方法</h3>
<p data-nodeid="65260">etcd 和 Zookeeper 都可以存储集群元数据，以保障集群核心信息的高可用和高性能访问。其中**etcd 用的是 Raft 协议， Zookeeper 用的是 Paxos 协议，这两种分布式协议都满足 CAP 理论中的 CP。**也就是说，它们都满足一致性（Consistency）和分区容错（Partition tolerance）的要求。</p>
<p data-nodeid="65261">但相比 Zookeeper ，这几年 etcd 在业界的认可度越来越高，多款重量级开源软件（如，Kubernetes、Traefik、Casbin）都使用了它。那么，人们为何会青睐 etcd 呢？</p>
<h4 data-nodeid="65262">etcd 的高可用原理</h4>
<p data-nodeid="65263">和 Zookeeper 相比，etcd 有很大优势。</p>
<p data-nodeid="65264">首先，etcd 支持 HTTPS 访问、划分命名空间、 RBAC 权限控制，可以有效保证数据安全。</p>
<p data-nodeid="65265">特别是在易用性方面，etcd 是用 Golang 实现的，简单配置后就可以运行。同时，它还提供了 HTTP 接口和 gRPC 接口，非常方便客户端使用。另外，etcd 还支持订阅某个 Key 下的数据变更，假如这个数据被修改了，客户端能实时收到通知并获取最新的数据。</p>
<p data-nodeid="65266">可用性方面，etcd 会将数据写入磁盘，不会因为节点宕机丢失数据。即便 etcd 集群当中有的节点宕机了，凭借 Raft 协议，也能保证集群稳定运行。</p>
<p data-nodeid="65267">你可能会问了，什么是 Raft 协议，etcd 具体是如何用 Raft 协议做集群管理的呢？</p>
<p data-nodeid="70760" class="">简单来说，Raft 协议是 etcd 中保证分布式系统强一致性的算法。Raft 协议维护了集群节点的角色状态——<strong data-nodeid="70766">Leader（领导者）、Follower（跟随者）、Candidate（候选者）</strong>。</p>


<p data-nodeid="71275" class="">其中，Leader 节点主要维护整个集群的运行状态，它负责通过 Raft 协议将写操作同步给所有节点，特别是通过心跳机制与 Follower 通信。而 Follower 节点主要负责把写请求和自身运行状态上报给 Leader，如果 Leader 任期结束或者心跳超时，Follower 会变成 Candidate 并发起选举。</p>

<p data-nodeid="65270">这里的 Candidate 就是选举 Leader 过程中的候选者，当某个 Candidate 获得一半以上的票数， Candidate 就会成为 Leader，承担起维护集群运行的责任。其他 Candidate 自动转变成 Follower，继续发挥自己的作用。</p>
<p data-nodeid="65271">除了选举出 Leader 维护集群信息外，etcd 对于每个 Key 的每次修改都会生成一个自增 ID 用于版本控制，并且会带上任期 ID， 确保集群各节点收到的数据版本一致并且是最新的。</p>
<p data-nodeid="65272">以上便是 etcd 的高可用原理。可以说，正是因为 etcd 的这些优点，越来越多的系统开始用它保存关键数据。比如，秒杀系统经常用它保存各节点信息，以便控制消费 MQ 的服务数量。还有些业务系统的配置数据，也会通过 etcd 实时同步给业务系统的各节点，比如，秒杀管理后台会使用 etcd  将秒杀活动的配置数据实时同步给秒杀 API 服务各节点。</p>
<h4 data-nodeid="65273">etcd 的部署方法</h4>
<p data-nodeid="65274">我们先看下它的架构图。</p>
<p data-nodeid="71781" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/27/Ciqc1F_QgVaAIuZXAADca-AE4gI784.png" alt="Drawing 8.png" data-nodeid="71784"></p>



<p data-nodeid="65278">通常，etcd 会监听两个端口，默认是 2379 端口和 2380 端口。其中，2380 端口用于集群内部通信，主要涉及集群间数据同步、心跳、选举等。2379 端口用于与客户端通信，比如接收客户端发起的读/写数据请求。</p>
<p data-nodeid="65279">etcd 节点在部署的时候有两种运行模式：<strong data-nodeid="65432">集群模式和代理模式。</strong></p>
<p data-nodeid="65280">当 etcd 节点以集群模式运行时，它会加入已有集群中，作为集群的一部分。也就是说，后续的心跳、数据同步、选举等它都会参与。</p>
<p data-nodeid="65281">集群模式中的节点数一般采用奇数个。为什么呢？因为假如同时有两个 Candidate 发起选举，如果是偶数节点的话，可能存在两个 Candidate 获得相同票数。</p>
<p data-nodeid="65282">这会导致什么问题？如果两个 Candidate 票数一样，就需要再次发起选举，而再次发起选举还是有一定概率出现票数一样，这会导致选举耗时较多，影响稳定性。所以，采用奇数个节点，能有效降低票数一样的概率，提升选举的效率。另外，使用奇数节点来部署，也能让 etcd 很好地处理分区容错问题。</p>
<p data-nodeid="72277" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgV6AOHqnAACBfLob2wk159.png" alt="Drawing 10.png" data-nodeid="72280"></p>



<p data-nodeid="65286">当某个 etcd 节点以代理模式运行时，该节点负责将接收到的请求转发给 etcd 集群节点。目前 etcd 接口有 v2 和 v3 两个版本，其中 v2 是 HTTP 接口，v3 是 gRPC 接口。需要注意的是，代理模式只支持转发 v2 版本的请求，也就是只支持转发 HTTP 请求。</p>
<p data-nodeid="72761" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgWWAFwz1AAB0BCWTmWA124.png" alt="Drawing 12.png" data-nodeid="72764"></p>



<p data-nodeid="65290">不过，由于 etcd v3 接口在性能、安全、稳定性等方面要比 v2 接口优秀很多，新项目倾向于使用 v3 接口，老项目也逐渐从 v2 接口迁移到 v3 接口。也就是说，代理模式以后可能逐渐被淘汰掉。</p>
<p data-nodeid="65291">关于 etcd 集群的具体搭建，etcd 官网有比较详细的教程和测试环境，感兴趣的话你可以上官网体验下。</p>
<p data-nodeid="73233" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgWyAe5PLAAJm2YHnKNQ087.png" alt="Drawing 13.png" data-nodeid="73236"></p>

<h3 data-nodeid="65293">小结</h3>
<p data-nodeid="65294">这一讲我们了解了常用 KV 存储中的 Redis 和 etcd 的高可用原理和部署方法，并且重点介绍了它们各种部署方式以及应用场景，不知道你是否掌握了吗？</p>
<p data-nodeid="65295">前面我也提到了，在秒杀系统中，我们将利用 Redis 和 etcd 实现多级缓存和漏斗模型中的部分功能，具体的实现方法我会在模块七详细介绍。</p>
<p data-nodeid="65296">请你思考下：假如秒杀系统有 5 个节点，只让其中一个节点能消费消息队列，如何使用 etcd 来实现这一功能呢？</p>
<p data-nodeid="65297">可以把你的想法写在下面的留言区哦。</p>
<p data-nodeid="65298">这一讲就介绍到这里，下一讲我将为你介绍如何防范重放攻击和 XSS 注入，到时见！</p>
<p data-nodeid="73705" class=""><img src="https://s0.lgstatic.com/i/image/M00/80/27/Ciqc1F_QgXSAVwDoAAO2dZB__aM380.png" alt="Drawing 14.png" data-nodeid="73708"></p>

<hr data-nodeid="65300">
<p data-nodeid="75592" class="te-preview-highlight"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="75597"><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgX2AHJo_ACRP1TPc6yM423.png" alt="Drawing 15.png" data-nodeid="75596"></a></p>





<p data-nodeid="65303" class=""><strong data-nodeid="65475">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="76060" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="76064">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **8214：
> etcd与zk的主要区别是啥，都是cp/都支持选举同步数据吧，看着没啥区别，偏向etcd的原因主要是？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 文中有相应解释，etcd 使用简单，它支持 grpc 和 http 两种协议访问。etcd 使用的 RAFT 协议比 zk 用的 ZAB 协议要容易理解。存储结构上 etcd v3是简单的 kv 结构，而 zk 是树形结构，访问数据时要遍历，所以 etcd 性能比 zk 好。

##### **辉：
> 老师，哨兵模式脑裂的问题怎么解决呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 首先，redis 哨兵部署时采用奇数个节点，能降低脑裂概率。其次，redis 集群中的数据本来就无法做到强一致性，对一致性要求较高的数据不适合放到 redis 集群中。对一致性要求不高的数据，脑裂了影响较小。

##### *明：
> 目前我们公司使用etcd做了基于雪花算法的分布式id和分布式锁的实现，目前业界这种场景多么，有没有建议或者注意的地方，希望老师指点一下

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 雪花算法在大型分布式系统中比较常见，比如订单系统中生成订单 ID、IM 系统中生成消息 ID。需要注意的是，雪花算法生成的 ID 是由几部分生成的，需要评估好业务规模，合理分配好 ID 中个部分的比特位数。

##### *强：
> 老师，思考题答案能解答一下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; etcd 支持分布式锁。可以每个节点去抢一个分布式锁，抢到锁的节点负责消费队列。要注意的是要给锁设置过期时间，每个节点内部需要有一个定时任务一直去抢锁，这样是为了消费队列的节点挂了后，其他节点能抢到锁并继续消费队列。

##### *铁：
> Etcd是第一次听说，学习了

##### *庚：
> 用etcd锁选主

