<p data-nodeid="4248">在写这个专栏的时候，正好赶上张一山版的《鹿鼎记》热播，鹿鼎记的主线就是寻找四十二章经。因为我这个专栏正好是二十四篇，还经常被爱人开玩笑说在写“二十四章经”。</p>
<p data-nodeid="4249">相对于传统的 IT 架构，互联网的架构在最近十几年变化非常迅速，究其原因还是互联网业务本身的飞速发展。如今中国互联网公司的用户规模已经全球领先，这得益于中国网络基础设施的建设，得益于智能手机的普及。</p>
<p data-nodeid="4250">在这十几年中，中国的互联网行业从最早的 Web1.0 代表的新浪、网易等门户网站，到 Web2.0 代表的社区网站豆瓣网、人人网、开心网，再到如今的移动互联网的 O2O 生活——美团、微信、支付宝，甚至最近大红大紫的短视频抖音、快手等，每一个都代表一个时代。</p>
<p data-nodeid="4251">随着用户规模的增大，互联网业务的迭代速度也越来越快，功能也越来越多。</p>
<p data-nodeid="4252">实际上在 PC 互联网时代，无论是门户网站还是社区网站，受限于电脑的普及程度和学习“高”门槛，用户的规模很难说庞大，竞争也不是特别激烈，所以在 PC 互联网时代我们也可以发现，大多数网站采用的就是<strong data-nodeid="4289">单体架构</strong>的设计，因为完全可以满足业务的迭代速度。</p>
<p data-nodeid="4253">但随着以手机为代表的移动设备的普及，用户接入互联网的门槛一下子降低了很多，导致网民数量迅速增长。互联网厂商可以更加容易地获取用户，厂商之间的竞争也变得更加激烈，这就导致了单体架构无法满足业务迭代的速度需求，<strong data-nodeid="4295">微服务架构</strong>应运而生。</p>
<p data-nodeid="4254">进入移动互联网时代后，产品的制作速度越来越快，从最早的一年一款产品，到后面几个月一款产品，再到几周一款产品。这样的速度，也促使互联网公司逐步从自建机房，到使用云厂商的虚拟机，再到云原生时代。这些都是为适应业务的迅速变化而产生的架构变化，<strong data-nodeid="4301">架构不是一蹴而就的，只有能够赋能业务，架构才能有生命力</strong>。</p>
<h3 data-nodeid="4255">从微服务到 Service Mesh</h3>
<p data-nodeid="4256">在这个专栏中，我们从微服务一直讲解到 Service Mesh，再到最终的展望部分简单讲解了 FaaS，可以说整个微服务的关键技术都涉及了。下面我们再来简单回顾一下整个专栏的内容。</p>
<p data-nodeid="4257">在模块一中，我们讲解了微服务和 Service Mesh 中的核心组件，包括注册中心、负载均衡器、路由器、网关、配置中心等。相信通过模块一的学习，你对微服务的原理已经可以深入掌握。同时，我们也提到了一些学习方法，比如要<strong data-nodeid="4309">关注英语词汇本身的意思</strong>，避免中英文思维差异而产生误解，又比如多去查阅 RFC 文档，以了解设计的本意。</p>
<p data-nodeid="4258">在模块二中，我们结合最流行的 Service Mesh 架构 Istio&amp;Envoy 实际进行了操作演示，进一步了解了 Service Mesh 的运行方式，也分析了 Envoy 才是整个 Service Mesh 架构的核心。对于整个 Service Mesh 数据面和控制面交互的 xDS 协议也进行了深入理解。</p>
<p data-nodeid="4259">在模块三中，我们自己用 Go 语言实现了一个简单的 Service Mesh 原型，包括数据面的代理层 Sidecar 和控制面的 xDS 协议。虽然这些只是最初步的实现，但相信这个章节会让你恍然大悟，原来 Istio 这样“高大上”的架构，在底层原理上也并非遥不可及。</p>
<p data-nodeid="4260">在模块四中，我结合自己的实际经验，分享了一些落地中的问题和困难，也对 Service Mesh 未来发展做了初步的展望：FaaS 和中间件 Mesh 都是美好的未来。</p>
<h3 data-nodeid="4261">Service Mesh 会是微服务演进的终极方向吗？</h3>
<p data-nodeid="4262">任何架构都有其生命周期，Service Mesh 也不例外，它不可能是微服务演进的最终目标。但是我相信 Service Mesh 会作为网络通信层，在未来的架构中继续发光发热，Service Mesh 的控制面和数据面分离的思想，更是会一直存在下去。</p>
<p data-nodeid="4263">在 24 讲中，我已经分享过 Service Mesh 未来的发展，它和 FaaS 的相结合将会是一个非常好的方向。但是目前我们也只是看到只有 Knative 这样做了，而且做的并不是很彻底。</p>
<p data-nodeid="4264">结合我们上一讲提过的这张图，虽然 Knative 充分利用了 Istio&amp;Envoy 的能力，但是也受限于开源的 Service Mesh 方案，没有办法更进一步。</p>
<p data-nodeid="5020" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/19/F1/CioPOWBK13KARAmNAAFOLVdbxMk461.png" alt="Drawing 0.png" data-nodeid="5024"></p>
<div data-nodeid="5021"><p style="text-align:center">Knative 架构图</p></div>



<p data-nodeid="4267">你可以试想一下，作为 Route（Istio Gateway）是否有存在的必要，而 Activator 是否又有必要呢？</p>
<p data-nodeid="4268">我们已经知道，经过 Sidecar 的代理后，微服务的内部通信都是以<strong data-nodeid="4335">服务发现</strong>的方式进行的，也就是说，内部流量根本没有必要经过 Route 这个 Gateway，而是直接通过 Pod to Pod 的方式通信。如果我们改造一下 Sidecar，<strong data-nodeid="4336">让 Sidecar 可以接管 Route 的功能</strong>，当没有发现服务节点的时候，将流量转发到 Activator；发现节点的情况将流量转发到服务的 Pods，这样不就可以省掉 Route 这层了吗？</p>
<p data-nodeid="4269">Knative 之所以没有这么做，是因为<strong data-nodeid="4342">Envoy 并不支持这样的工作</strong>。传统的 Service Mesh 数据面没有为 Faas 改造 Sidecar，在没有节点的情况下会直接返回 500 错误。</p>
<p data-nodeid="4270">我们进一步思考，Activator 这个组件的功能是否也可以集成到 Sidecar 中？Activator 和 Route 作为 Knative 的两个核心组件，存在一个最大的问题——它们都是中心化的。也就是说，这样的方式<strong data-nodeid="4348">一旦挂掉，将会影响整个集群的调用</strong>。</p>
<p data-nodeid="4271">Activator 的功能就是在服务 Pod 为 0 的时候，通知 Autoscaler 组件进行服务扩容，这样的操作 Sidecar 自然也可以接管。所以整个架构就变成了，Sidecar 在发现服务没有节点的时候，直接通知 Autoscaler 组件进行扩容操作。此时先阻塞住请求，等待服务发现节点的时候，再将请求转发过去，这样我们就可以利用 Service Mesh 的能力让 FaaS 的架构变得更加稳定可靠，也更容易落地。</p>
<p data-nodeid="4272">你也可以思考一下，我在上一讲中提到的 OpenFaaS 异步模式，是否也可以用上面的方式进行改造，省去 Gateway。</p>
<h3 data-nodeid="4273">写在最后的话</h3>
<p data-nodeid="4274">到这里整个专栏就全部结束了，很高兴能将自己微薄的知识分享给你，也希望你在学习的过程中有所收获。我是一个非常较真，凡事喜欢刨根问底的人，所以在这个专栏中也会偏重原理部分。当然，这些内容我也在整个课程中以最直白的讲解方式分享给你，尽量避免所谓的“高大上”的词汇，而是结合我自己理解，用更易理解、更加准确的词汇去表述，也希望你喜欢这样的讲解风格。</p>
<p data-nodeid="4275">“人生相遇，自是有时。送君千里，终须一别。”很高兴和你一同度过了 3 个月的学习时光，对于专栏内容有任何疑问，也欢迎你在留言区给我留言。</p>
<p data-nodeid="4276">最后，我想邀请你参与对本专栏的评价，你的每一个观点对我们来说都是最重要的。<a href="https://wj.qq.com/s2/8177891/91b1/" data-nodeid="4357">点击链接，即可参与评价，还有机会获得惊喜奖品！</a></p>
<p data-nodeid="4277">本专栏到此结束，衷心希望各位读者一切顺利。</p>

---

### 精选评论

##### **华：
> 感谢分享

##### **军：
> 感谢分享

