<p data-nodeid="166520" class="">你好，我是徐鹏，已经在游戏、互联网行业从业 10 多年，目前在趣头条负责边缘网关和 Service Mesh 的研发、落地。我对统一网关层有着丰富的实践经验，工作期间自研的 Service Mesh 技术体系已经承接了数千条调用链路和数百万 QPS 峰值。</p>
<h3 data-nodeid="166521">为什么要学习 Service Mesh？</h3>
<p data-nodeid="166522">近年来，微服务在业内的实践已经从流行走向成熟。<strong data-nodeid="166598">微服务架构使应用程序更易于扩展、更快开发，从而加速创新并缩短新功能的上市时间</strong>。与单体应用相比，微服务能够更好地满足互联网时代业务快速变化的需要。</p>
<p data-nodeid="166523">但是<strong data-nodeid="166604">与其他现存的架构和解决方案一样，微服务架构也不是银弹</strong>，尽管它解决了单体服务的很多问题，却也带来了负载均衡、服务治理、服务注册发现、如何拆分服务等问题。</p>
<p data-nodeid="166524">Service Mesh （服务网格）是一个用于处理服务和服务之间通信的基础设施层，它最重要的变革，就是引入了数据面和控制面的概念：<strong data-nodeid="166610">通过 sidecar 模式将原本在 SDK 中的代码独立出来，用控制面代替配置中心的部分功能，以透明代理的形式提供安全、快速、可靠的服务间通信，同时也能实现微服务所需的基本组件功能</strong>。</p>
<p data-nodeid="166525">实际上，Service Mesh 需要的基础组件和传统的微服务并没有太大的差别，很多公司选择自研控制面的原因，很多就是出于兼容老的微服务的基础组件的考虑，<strong data-nodeid="166615">你可以把 Service Mesh 看作是分布式的微服务代理。</strong></p>
<p data-nodeid="166526">我从 2018 年开始研发并落地 Service Mesh 架构，当时公司处在迅速地从单体服务转向微服务的过程中，而我也有幸主导了第一版微服务框架的开发和落地，但在实际的落地过程中，我们也遇到了一些问题，比如<strong data-nodeid="166621">微服务框架的升级、框架多语言支持</strong>。</p>
<p data-nodeid="166527">后来由于容器技术的火爆，作为 Service Mesh 的代表 Istio 也顺理成章成为业界明星。受到 Istio 的影响，我也在思考是否可以将第一版微服务 API 网关部署在本地，比如<strong data-nodeid="166627">让服务注册发现的功能集成到一个通用组件中</strong>。带着这样的思考，团队很快将 API 网关改造成了 sidecar，并在公司的服务中落地了。</p>
<p data-nodeid="166528">我们根据公司自身的运维环境和业务需求，研发了一套适合自己的 Service Mesh，名叫 Negri。<strong data-nodeid="166633">通过 Negri，可以让服务无侵入地拥有服务注册发现、限流、熔断、降级等功能，并自动集成公司现有的 Trace、Metrics、Log 等基础服务能力</strong>。通过这种方式，我们成功解决了第一版微服务架构的问题。</p>
<p data-nodeid="166529" class=""><img src="https://s0.lgstatic.com/i/image/M00/8A/EF/Ciqc1F_a_omAfAHQAAGjUQGyVFQ565.png" alt="Drawing 0.png" data-nodeid="166636"></p>
<div data-nodeid="166530"><p style="text-align:center">Service Mesh Negri 研发历程图</p></div>
<p data-nodeid="166531">其实，从这样的经历你可以看出来，<strong data-nodeid="166642">任何架构都不是凭空而来，而是切实地解决了某些痛点和业务场景</strong>。</p>
<p data-nodeid="166532" class="">下表是各大公司 Service Mesh 架构的落地情况，你可以清晰地看到：Service Mesh&nbsp;技术越来越火热，受到越来越多开发者的关注。而<strong data-nodeid="166652">Service Mesh 这个架构</strong>之所以这么迅速地被各大公司实施落地，正是<strong data-nodeid="166653">切中了传统微服务架构中诸如升级成本高、中间件演变困难、缺乏统一管控手段、治理功能不全的痛点</strong>，解决了实际的问题。</p>
<p data-nodeid="166763" class=""><img src="https://s0.lgstatic.com/i/image2/M01/03/A4/CgpVE1_gQIiASAOWAAC8FaUNEhw765.png" alt="1.png" data-nodeid="166766"></p>

<p data-nodeid="167888">通过拉勾网的招聘情况你也可以看到，不少大厂在微服务架构师招聘中，提出需要了解 Service Mesh 并且具备实战经验。随着越来越多的公司引入 Service Mesh 架构体系，如果你对这个架构没有根本的、清晰的认识，也很难保持市场竞争力，以及跟上公司技术发展的步伐。</p>
<p data-nodeid="168260" class=""><img src="https://s0.lgstatic.com/i/image/M00/8A/FA/CgqCHl_a_p6APQjcAAERYC2zPow535.png" alt="Drawing 1.png" data-nodeid="168263"><br>
<img src="https://s0.lgstatic.com/i/image/M00/8B/C1/Ciqc1F_gQK2AGo9bAAEAXKTE2GE791.png" alt="2.png" data-nodeid="168267"></p>





<h3 data-nodeid="166564">学习 Service Mesh 的痛点</h3>
<p data-nodeid="166565">实际上 Service Mesh 可以看作微服务架构的一种演进形式，所以要想从根本上理解整个 Service Mesh 架构，必须首先了解微服务架构，尤其是两者的基础组件，都是可以通用的。</p>
<p data-nodeid="166566">Service Mesh 一个最重要的变革，就是引入了数据面和控制面的概念。了解了这两个概念，可以说你就对 Service Mesh 有了一个大致印象。另外，Service Mesh 本身涉及的名词非常多，特别是我们在看一些开源组件时，经常会因为对一些名词不够理解而陷入困境。</p>
<p data-nodeid="166567">这门课程中，我会<strong data-nodeid="166705">针对这些基础内容进行讲解，带你拨开迷雾，深入感知 Service Mesh</strong>。</p>
<p data-nodeid="166568">现在市面上有很多介绍 Service Mesh 的文章，大多数是讲为什么演进到 Service Mesh，或是只要提到 Service Mesh 我就给你讲 Istio，说好的实战实则是 Istio 的“使用手册”，对于 Service Mesh 背后的原理、如何才能真正有效地落地并未太多提及。甚至你看了大量文章、学了很多课程好像还是没太明白 Service Mesh 整体是什么。</p>
<p data-nodeid="166569">我也希望从这个角度出发，<strong data-nodeid="166712">从底层原理展开讲一讲 Service Mesh 的全貌，并结合自己在落地实践中的一些经验，更多聚焦在为什么、怎么做上，和你一起探讨如何才能更好地让系统演进到 Service Mesh 架构</strong>。</p>
<h3 data-nodeid="166570">课程设计</h3>
<p data-nodeid="166571">很多人可能难以理解 Service Mesh 相对于传统的微服务体系的优势是什么、为什么要从微服务演进到 Service Mesh 体系，针对这个问题我会从微服务开始讲解，循序渐进地介绍架构的演进过程。从介绍微服务和 Service Mesh 的基础知识、核心组件出发，帮助你形成对 Service Mesh 全方位的了解。</p>
<ul data-nodeid="166572">
<li data-nodeid="166573">
<p data-nodeid="166574"><strong data-nodeid="166723">模块一：微服务和 Service Mesh 核心组件</strong>。这个模块我会讲解微服务和 Service Mesh 的中的核心组件，比如注册中心、负载均衡器、路由器、配置中心等，带你<strong data-nodeid="166724">从原理上理解各个组件的作用，如何使用组件，以及如何研发此类型组件</strong>，让你在未来的工作中游刃有余。</p>
</li>
<li data-nodeid="166575">
<p data-nodeid="166576"><strong data-nodeid="166733">模块二： Service Mesh 实战。</strong> 我将从最流行的<strong data-nodeid="166734">Istio + Envoy 架构</strong>入手，带你理解控制面和数据面，动手实战 Service Mesh。现在市面上很多课程或者书籍都是基于 Istio 1.5 以前的版本进行学习，但 Istio 在 1.5 版本之后已经发生了巨大变化，我会带你从最新技术出发，“重新认识”相关架构。</p>
</li>
<li data-nodeid="166577">
<p data-nodeid="166578"><strong data-nodeid="166739">模块三：自己动手用 Go 实现 Service Mesh。</strong> 通过最简化的代码级演示，让你能够在源码级别，结合已有的知识，更清晰地认识 Service Mesh 的底层架构。通过自己动手实战，才能加强对原理的理解。</p>
</li>
<li data-nodeid="166579">
<p data-nodeid="166580"><strong data-nodeid="166744">模块四： Service Mesh 落地和展望。</strong> 这一模块我将主要讲解 Service Mesh 落地中常见的问题和困难，并带你展望未来，通过自己的实践经验帮助大家思考如何更好地落地 Service Mesh。学完之后，希望你能够在面对公司 Service Mesh 落地时，做到心中有数、操作自如。</p>
</li>
</ul>
<p data-nodeid="167133" class=""><img src="https://s0.lgstatic.com/i/image/M00/8B/CC/CgqCHl_gQKOAS1mEAAFjDwopSa8857.png" alt="3.png" data-nodeid="167137"></p>
<div data-nodeid="167134"><p style="text-align:center">Service Mesh 学习结构图</p></div>


<p data-nodeid="166583">另外在这个课程中，我会更多地讲解为什么要这么设计，也希望你能带着问题来思考更本质的原因。</p>
<h3 data-nodeid="166584">讲师寄语</h3>
<p data-nodeid="166585" class="">虽然技术的发展日新月异，但是能够解决公司实际问题的技术才是最有价值的，Service Mesh 就是这样的技术。</p>
<p data-nodeid="169004">技术是在实践中总结并得到升华的，很多时候我们作为研发者也需要经常思考，如何从业务中提炼出更多的基础设施，让业务开发更高效。我也希望你通过本次课程，不仅能够掌握 Service Mesh，也能够通过了解最前沿的技术方案具备举一反三的能力，有效定位公司实际面临的问题，探索出新的技术演进方向。</p>
<p data-nodeid="169005" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8B/CD/CgqCHl_gQNSAK9iZAAFML6ohmkk219.png" alt="4.png" data-nodeid="169009"></p>



<p data-nodeid="166587">多说不如行动，我们专栏中见。</p>
<hr data-nodeid="166588">
<p data-nodeid="166589"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="166759"><img src="https://s0.lgstatic.com/i/image/M00/8B/BD/Ciqc1F_gEFiAcnCNAAhXSgFweBY589.png" alt="java_高薪训练营.png" data-nodeid="166758"></a></p>
<p data-nodeid="166590" class=""><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="166762">拉勾背书内推 + 硬核实战技术干货，帮助每位 Java 工程师达到阿里 P7 技术能力。点此链接，快来领取！</a></p>

---

### 精选评论

##### *悦：
> 加油坚持下去一定会有收获

##### *楠：
> 架构师前来报道

##### *鹏：
> 感觉特别好

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 那就一定要坚持学习呀~

##### J：
> 留言表示支持

