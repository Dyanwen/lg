<p data-nodeid="7028">你好，我是老蒋，一个在前端摸爬滚打 7 年的老兵，目前是国内某一线大厂的资深软件工程师。</p>
<p data-nodeid="7029">2017 年国内 Serverless 技术刚刚兴起，我就开始使用和推广 Serverless 了，当时，我的团队负责很多前端中后台系统的研发，后端为了方便扩展就把接口设计得很原子化，导致前端渲染一个页面要发几十个请求，前后端联调很痛苦，沟通成本非常高，开发效率也很低。</p>
<p data-nodeid="7030">为了提升前端开发效率和页面加载性能，我们用 Node.js 开发了很多 BFF（Backend for Frontend）应用来做接口的聚合裁剪。可随着接入的系统增多，BFF 应用数量和复杂度日趋上升，不仅消耗了大量机器，还导致前端运维成本急剧增高，很多时候都要去排查服务器的问题。</p>
<p data-nodeid="7031">机缘巧合下，我抱着试一试的心态引入了 Serverless 来实现 BFF 层，结果让前端从运维中解放出来，同时也减少了不必要的机器资源浪费。</p>
<p data-nodeid="7032">从此以后，我便一直在 Serverless 领域深耕和探索，也总结和沉淀了很多关于 Serverless 架构落地的心得，希望通过这门课把这些经验分享给你，让你更加体会到开发的乐趣。</p>
<h3 data-nodeid="7033">Serverless 时代已来临</h3>
<p data-nodeid="7034">可能你会认为 Serverless 是最近两年兴起的技术，实际上，Serverless 概念从 2012 年就提出来了，随后 AWS 在 2014 年推出了第一款 Serverless 产品 Lambda，开启了 Serverless 元年。</p>
<p data-nodeid="7035">此后国外 Serverless 生态迅速发展，诞生了如 <a href="http://serverless.com/" data-nodeid="7078">Serverless Framework</a>、<a href="https://vercel.com/" data-nodeid="7082">Vercel</a> 等很多优秀的产品。只是国内起步比较晚，2017 阿里云和腾讯云才相继发布了国内的 Serverless 产品：函数计算和云函数，这时 Serverless 才开始进入大多数国内开发者的视野，各大厂商开始极力宣传和实践 Serverless，到现在不过短短三年，国内 Serverless 开发进入白热化阶段，<strong data-nodeid="7087">周围不管大厂小厂，都开始用 Serverless 解决各自业务中的效率和成本问题。</strong></p>
<p data-nodeid="7036">比如，阿里这两年的“双十一”都在用 Serverless 提升应用弹性能力、降低服务器成本；Airbnb 在用 Serverless 做实时分析报警；微博在使用 Serverless 进行图片个性化处理；石墨文档也在用 Serverless 实现实时协作……</p>
<h3 data-nodeid="7037">Serverless 带来的机遇</h3>
<p data-nodeid="7038">Serverless 说白了，就是一种不用关心服务器的架构思想，开发者可以不关心除业务代码之外的事情，进而提高业务的迭代效率，使用的云服务也是用多少付多少，从而节省成本。总的来讲，<strong data-nodeid="7095">Serverless 对开发者研发方式的影响是巨大的</strong>。</p>
<p data-nodeid="7039">对前端工程师来说，曾经你部署一个 BFF 或者服务端渲染应用，要购买机器、安装环境，甚至考虑负载均衡、分布式缓存、流量控制等复杂后端问题，而 Serverless 把这些能力封装成服务，让你开箱即用，解决你不会服务器运维的困难。另外，由于 Serverless 把后端变得很简单，甚至前端工程师自己就可以独立完成整个应用开发了，再加上前端工程师离用户最近，所以前端可以成为产品负责人。</p>
<p data-nodeid="7040">而且虽然 Serverless 时代已经来临，但国内 Serverless 生态距离国外还有不小差距，国外关于 Serverless 的开发框架、WebIDE 等技术已经比较成熟，而国内还处于发展初期，这种“差距”对于国内前端工程师来说是个很好的机会。</p>
<p data-nodeid="7041">当然，Serverless 对后端工程师也有很大的影响，虽然关注 Serverless 的前端人数的确比后端要多，但掌握 Serverless 技术，对后端工程师极为必要。因为基于 Serverless，后端工程师不用再关心系统运维，可以专注于业务开发，深入业务细节，使业务快速迭代，帮助业务成功。另一方面，后端工程师也可以深入底层去构建 Serverless 基础设施，让技术为更多开发者服务。</p>
<p data-nodeid="7042">而且就算我不说你也能发现，<strong data-nodeid="7104">不管是前端岗位还是后端岗位，大家更喜欢有 Serverless 开发经验的同学</strong>。腾讯、头条等国内很多大型互联网公司也都在招聘 Serverless 架构师，并且薪资非常高，极大反映出各大厂对 Serverless 技术人才的积极态度。</p>
<p data-nodeid="7043">我作为面试官也认为：如果一个开发者会 Serverless，那说明他很关注前沿技术，未来可以以更高效率、更低成本的方式为公司完成业务创造价值，从而给公司带来产品和技术上的领先。</p>
<p data-nodeid="7044"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/8B/F0/Ciqc1F_i-PWAfISFAAPrRZz2hM0607.png" data-nodeid="7108"></p>
<h3 data-nodeid="7045">学习 Serverless 所面临的挑战</h3>
<p data-nodeid="7046">虽然开发者开始关注、尝试 Serverless，但在学习中也会遇到很多问题。</p>
<p data-nodeid="7047">大部分网络文章对 Serverless 几乎都是概念的介绍，书籍中就算有实践案例也只是简单的 Demo，深度不够。<strong data-nodeid="7116">这让开发者的 Serverless 学习过程变得极其困难，很多知识盲点都要自己探索，无法建立自己的知识体系</strong>。（比如，Serverless 应用开发的全流程是什么？怎样对 Serverless 函数进行本地调试和编写单元测试？）</p>
<p data-nodeid="7048">当然，随着国内云厂商对 Serverless 的支持越来越完善，一些大厂和实践者也开始推出 Serverless 的开发教程，但往往都局限于某个云产品，很容易成为云产品的宣传，不能深入 Serverless 技术本身，不具有普适性。由于 Serverless 本身严重依赖云产品，很容易让开发者被云厂商绑定，比如当开发者选择阿里云之后，就很难迁移到腾讯云，这也是开发者的痛点之一。</p>
<p data-nodeid="7049">另外，虽然所有人都在谈论 Serverless，越来越多的企业和开发者开始尝试使用 Serverless，但都还处于探索阶段，<strong data-nodeid="7123">落地经验输出还非常少，</strong> 更不用说大型 Serverlss 架构项目的落地经验了。这就导致开发者在使用 Serverless 开发复杂业务时缺少经验、容易踩坑，很难将传统应用迁移到 Serverless 架构，开发者需要汲取大厂落地经验，才能更好地使用和发挥出 Serverless 的价值（这正是模块四存在的意义）。</p>
<h3 data-nodeid="7050">课程设计</h3>
<p data-nodeid="7051">我和拉勾教育合作了这门课，希望能脱离具体云产品平台的限制，聚焦在 Serverless 技术本身，循序渐进地从概念到开发基础帮你逐渐理清知识盲区，形成知识体系，再通过真实的落地经验与典型的场景案例，让你真正理解和学会运用 Serverless。</p>
<p data-nodeid="7052">整体上讲，我把课程划分为四个模块。</p>
<ul data-nodeid="7053">
<li data-nodeid="7054">
<p data-nodeid="7055"><strong data-nodeid="7131">概念篇：</strong> 从云计算发展历程的角度，为你深度剖析 Serverless 兴起的必然因素，让你建立对 Serverless 的宏观认知，然后通过一个案例带你理解 Serverless 这个概念。</p>
</li>
<li data-nodeid="7056">
<p data-nodeid="7057"><strong data-nodeid="7136">开发基础篇：</strong> 我会从一个开发者的视角，从应用设计、开发、测试、部署的整个流程，为你解释开发 Serverless 应用的基础知识，帮你理清知识盲点。学完之后，你就可以开发一个简单的 Serverless 应用了。</p>
</li>
<li data-nodeid="7058">
<p data-nodeid="7059"><strong data-nodeid="7141">开发进阶篇：</strong> 鉴于市面上比较缺乏大规模 Serverless 应用的实践经验，我会针对系统迁移难、厂商依赖强、经济成本高、应用不安全等开发者关心的痛点问题，将我在大型 Serverless 架构中总结的相关经验分享给你。学完之后，你就能更好地进行 Serverless 技术选型，基于 Serverless 开发复杂业务。</p>
</li>
<li data-nodeid="7060">
<p data-nodeid="7061"><strong data-nodeid="7146">场景案例篇：</strong> 我会选择一些开发者比较常见且十分典型的真实场景案例（比如基于 Serverless 实现弹性可扩展的 Restful API 系统），带你将前面学习到的理论知识运用于实践，从零开始设计开发一个复杂的 Serverless 应用，深刻感受 Serverless 给开发模式带来的改变，以及开发效率的提升。</p>
</li>
</ul>
<p data-nodeid="7062">为了让你更全面地了解 Serverless，我还特地为你画了一张学习路径图：</p>
<p data-nodeid="7163"><img alt="Lark20201230-111838.png" src="https://s0.lgstatic.com/i/image/M00/8C/5B/Ciqc1F_r8Z6APJcVAA2iRurEGFE180.png" data-nodeid="7166"></p>

<h3 data-nodeid="7064">讲师寄语</h3>
<p data-nodeid="7065"><strong data-nodeid="7160">虽然 Serverless 技术还比较新，但我相信它终会成为技术架构的主流，因为成本和效率永远是开发者和企业最关心的两大问题</strong>。</p>
<p data-nodeid="7066">技术新意味着你有更多的机会，越早学习、实践就越能占领技术先机。希望你能通过这门课，构建自己的 Serverless 知识体系，并将知识灵活地运用到实际工作中去解决业务问题。</p>
<p data-nodeid="7067">最后，学习是相互的，你在学习过程中有任何收获或感想，以及任何关于 Serverless 落地实践的困难与经验，都欢迎在评论区留言，与我沟通交流。</p>

---

### 精选评论

##### *琴：
> 如何才能不被“云厂商绑架”？又如何抹平各大云厂商的差异？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 云厂商绑定一方面是 API 绑定，API 是连接业务代码和 Serverless 平台的桥梁，但不同云厂商的 API 定义不同，这就会导致开发者从一个云迁移到另一个云变得困难。要解决这个问题，就需要将业务代码和云厂商的 API 分离，降低业务与云厂商 API 的耦合度，架构设计上可以采用六边形架构，例如专门实现一个 API 适配器层，用来调用云厂商的 API，这样在不同云之间迁移时，就只需要更新适配器。当然适配器是一件非常繁琐的工作，需要细致了解不同云的 API，你也可以选择一些 Serverless 框架来实现，比如 Serverless Framework。
另一方面是云服务的绑定，尤其是开发 Serverless 应用，更需要依赖云服务。为了避免云服务的绑定，在技术选型时可以选择业界有公共标准的云服务，比如 MySQL、Redis、Kafka，这样在使用不同云厂商的云服务甚至自建这些服务时，都可以平滑迁移。对于 Serverless 应用，在这方面另一个比较棘手的问题是基于云服务的触发器事件格式不同，解决这个问题的方法与 API 适配类似，你可以将业务逻辑和触发器对应的入口函数分离，为不同云厂商的触发器实现适配器。目前云原生基金会也在积极推进 CloudEvents 标准，如果之后各个云厂商都遵循该标准定义事件数据，则 Serverless 平台方面的厂商绑定问题就会减轻很多。

##### **云：
> 作者的想法赞，市场上 serverless 文章容易让入局者误解必须和云厂商绑定的误区，让学习门槛增高

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢支持，互相学习

