<p data-nodeid="22865">上一讲我讲了 Serverless 架构兴起的必然因素，在这个过程中，我简单提到了 Serverless 的概念，相信你对 Serverless 已经有了初步的认知，这节课我将继续深入剖析到底什么是 Serverless。</p>
<p data-nodeid="22866">有不少刚接触 Serverless 的同学会认为 FaaS 就是 Serverless，也有同学认为 PaaS 也是 Serverless，还有同学说使用 Serverless 就没有服务器了。总的来说，很多同学对 Serverless 到底是什么并没有一个很清晰的认知，概念还比较模糊，所以咱们就用一节课的时间，搞定这个概念。</p>
<p data-nodeid="22867">在我看来，你可以从广义和狭义两个角度入手，这样会更清楚 Serverless 的特点，对它有个整体的认知。<strong data-nodeid="22971">先来看一下广义上的 Serverless 是指什么？</strong></p>
<h3 data-nodeid="22868">广义的 Serverless</h3>
<p data-nodeid="22869"><strong data-nodeid="22980">在我看来，广义的 Serverless 是指：构建和运行软件时不需要关心服务器的一种架构思想。</strong> 虽然 Serverless 翻译过来是 “无服务器”，但这并不代表着应用运行不需要服务器，而是开发者不需要关心服务器。<strong data-nodeid="22981">而基于 Serverless 思想实现的软件架构就是 Serverless 架构。</strong></p>
<p data-nodeid="22870">那与 Serverless 对应的概念就是 Serverful。回顾<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=589#/detail/pc?id=6029" data-nodeid="22987">“01 | 前因后果：Serverless 架构兴起的必然因素是什么？”</a>中的电商网站，我们直接在虚拟机上部署网站的架构，就是 Serverful 的架构，在这种架构下，如果要保证网站持续稳定运行，就需要解决很多问题。</p>
<ul data-nodeid="22871">
<li data-nodeid="22872">
<p data-nodeid="22873"><strong data-nodeid="22993">备份容灾：</strong> 要实现服务器、数据库的备份容灾机制，使一台服务器出故障不影响整个系统。</p>
</li>
<li data-nodeid="22874">
<p data-nodeid="22875"><strong data-nodeid="22998">弹性伸缩：</strong> 系统能根据业务流量大 小等指标，响应式地调整服务规模，实现自动弹性伸缩。</p>
</li>
<li data-nodeid="22876">
<p data-nodeid="22877"><strong data-nodeid="23003">日志监控：</strong> 需要记录详细的日志，方便排查问题和观察系统运行情况，并且实现实时的系统监控和业务监控。</p>
</li>
</ul>
<p data-nodeid="22878">解决这些复杂的问题需要投入大量的人力、物力，小公司几乎无法自己去解决。而对开发者来说，Serverful 的架构开发成本也非常高，原本几行代码就可以搞定一个简单的业务逻辑，但你却得添加庞大的框架，比如 RPC（Remote Procedure Call，远程调用）、缓存等。</p>
<p data-nodeid="22879"><strong data-nodeid="23009">Serverless 就是为了解决这些问题诞生的。</strong> 它可以把底层的硬件、存储等基础资源隐藏起来，由平台统一调度、运维。并将常用的基础技术抽象、封装（比如数据库、消息队列等）以服务的方式提供给开发者。开发者只专注于开发业务逻辑，所有业务无关的基础设施，都交给 Serverless 平台。</p>
<p data-nodeid="22880">总之， Serverless 和 Serverful 的架构有这样几个区别。</p>
<ul data-nodeid="22881">
<li data-nodeid="22882">
<p data-nodeid="22883"><strong data-nodeid="23015">资源分配：</strong> 在 Serverless 架构中，你不用关心应用运行的资源（比如服务配置、磁盘大小）只提供一份代码就行。</p>
</li>
<li data-nodeid="22884">
<p data-nodeid="22885"><strong data-nodeid="23020">计费方式：</strong> 在 Serverless 架构中，计费方式按实际使用量计费（比如函数调用次数、运行时长），不按传统的执行代码所需的资源计费（比如固定 CPU）。计费粒度也精确到了毫秒级，而不是传统的小时级别。</p>
</li>
<li data-nodeid="22886">
<p data-nodeid="22887"><strong data-nodeid="23025">弹性伸缩：</strong> Serverless 架构的弹性伸缩更自动化、更精确，可以快速根据业务并发扩容更多的实例，甚至允许缩容到零实例状态来实现零费用，对用户来说是完全无感知的。而传统架构对服务器（虚拟机）进行扩容，虚拟机的启动速度也比较慢，需要几分钟甚至更久。</p>
</li>
</ul>
<p data-nodeid="22888">所以，一个应用如果是 Serverless 架构的，必须要实现自动弹性伸缩和按量付费，<strong data-nodeid="23030">这也是 Serverless 的核心特点。</strong></p>
<p data-nodeid="22889"><img alt="Lark20201224-184407.jpeg" src="https://s0.lgstatic.com/i/image2/M01/03/EA/Cip5yF_kcS2ABtOCAATEC7Xd7ds88.jpeg" data-nodeid="23033"></p>
<h3 data-nodeid="22890">狭义的 Serverless</h3>
<p data-nodeid="22891">广义的 Serverless 更多是指一种技术理念，狭义的 Serverless 则是现阶段主流的技术实现。之所以说是狭义的，是因为 Serverless 架构正在持续发展中，未来可能有更好的技术方案。</p>
<p data-nodeid="22892"><strong data-nodeid="23039">在我看来，狭义的 Serverless 是 FaaS 和 BaaS 的组合，为什么呢？</strong></p>
<p data-nodeid="22893"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/8B/F0/Ciqc1F_i_RCAJMutAAJC6EJRbDI546.png" data-nodeid="23042"></p>
<div data-nodeid="23187"><p style="text-align:center"><span style="color:#b8b8b8">Serverless 架构图</span></p></div>

<p data-nodeid="22895">为了弄清楚这个问题，你需要对 FaaS 和 BaaS 有更多的了解。在 01 讲，我简单地提了 FaaS 和 BaaS 的概念，接下来我从一个案例入手，带你对其进行更深入的剖析。</p>
<p data-nodeid="22896">假设你要实现一个接口，返回一个数组，你用的是 Node.js，传统的写法如下：</p>
<pre class="lang-javascript" data-nodeid="22897"><code data-language="javascript"><span class="hljs-comment">// index.js</span>
<span class="hljs-keyword">const</span> express = <span class="hljs-built_in">require</span>(<span class="hljs-string">'express'</span>);
<span class="hljs-keyword">const</span> app = express();
<span class="hljs-keyword">const</span> port = process.env.PORT || <span class="hljs-number">3000</span>;
<span class="hljs-comment">// 定义 /list 路由</span>
app.get(<span class="hljs-string">'/list'</span>, (req, res) =&gt; {
  <span class="hljs-keyword">const</span> data = [<span class="hljs-string">'a'</span>, <span class="hljs-string">'b'</span>, <span class="hljs-string">'c'</span>];
  <span class="hljs-comment">// 返回 data 数组</span>
  res.json(data);
});
<span class="hljs-comment">// 监听 3000 端口并启动服务</span>
app.listen(port, () =&gt; {
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">`Express running on http://localhost:<span class="hljs-subst">${port}</span>`</span>);
});
</code></pre>
<p data-nodeid="22898">如果想让这个接口对外提供服务，还要买一台服务器，在服务器上安装 Node.js 运行环境，然后把这段代码上传到服务器上，通过 node index.js 命令运行起来。然后还要购买域名（如 www.example.com），将域名解析到服务器上，再在服务器上安装 Nginx ，编写 Nginx 配置监听服务器的 80 端口，并将域名下的请求转发到 3000 端口。</p>
<p data-nodeid="22899">既然 Serverless 架构可以让你不关心服务器，那怎么用 Serverless 架构去实现这个功能呢？<strong data-nodeid="23050">答案就是 FaaS。</strong></p>
<p data-nodeid="22900">FaaS（Function as a Service）本质上是一个函数运行平台，大多 FaaS 产品都支持 Node.js、Python、Java等编程语言，你可以选择你喜欢的编程语言编写函数并运行。函数运行时，你对底层的服务器是无感知的，FaaS 产品会负责资源的调度和运维，<strong data-nodeid="23056">这是它的特点之一，不用运维</strong>。</p>
<p data-nodeid="22901">另外，FaaS 中的函数也不是持续运行的，而是通过事件进行触发，比如 HTTP 事件、消息事件等，产生事件的源头叫触发器，FaaS 平台会集成这些触发器，我们直接用就行，<strong data-nodeid="23062">这是 FaaS 的第二个特点，事件驱动</strong>。</p>
<p data-nodeid="22902"><strong data-nodeid="23070">FaaS 的第三个特点是按量付费。</strong> FaaS 产品的收费方式，都是按照函数执行次数和执行时消耗的 CPU、内存等资源进行计费的。除此之外，FaaS 在运行函数的时候，会根据并发量自动生成多个函数实例，并且并发理论是没有上限的，<strong data-nodeid="23071">这是它的第四个特点，弹性伸缩。</strong></p>
<p data-nodeid="22903"><img alt="Lark20201223-161544.png" src="https://s0.lgstatic.com/i/image/M00/8B/F0/Ciqc1F_i_SGARHu0AAZMti00uz8578.png" data-nodeid="23074"></p>
<div class="te-preview-highlight" data-nodeid="23832"><p style="text-align:center"><span style="color:#b8b8b8">FaaS 架构图</span></p></div>

<p data-nodeid="22905">基于 FaaS，实现上面的接口就简单了，你只需要写一个函数，然后把代码部署到 FaaS 产品上，设置 HTTP 触发器就可以了。</p>
<pre class="lang-javascript" data-nodeid="22906"><code data-language="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">handler</span>(<span class="hljs-params">event</span>) </span>{
  <span class="hljs-comment">// data 数组就是接口的返回数据</span>
  <span class="hljs-keyword">const</span> data = [<span class="hljs-string">'a'</span>, <span class="hljs-string">'b'</span>, <span class="hljs-string">'c'</span>];
  <span class="hljs-keyword">return</span> data;
}
</code></pre>
<p data-nodeid="22907">那假设接下来你要实现一个计算 PV 的接口，简单一点你直接在内存中存储 PV，使用 FaaS 你可能会这样写：</p>
<pre class="lang-javascript" data-nodeid="22908"><code data-language="javascript"><span class="hljs-comment">// 使用全局变量存储当前 PV</span>
<span class="hljs-keyword">let</span> pv = <span class="hljs-number">0</span>;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">handler</span>(<span class="hljs-params">event</span>) </span>{
  <span class="hljs-comment">// 用户每访问一次就给 PV 的值加 1</span>
  pv++;
  <span class="hljs-keyword">return</span> pv;
}
</code></pre>
<p data-nodeid="22909">当你把函数部署到 FaaS，然后访问接口时，会发现得到的结果永远都是 1。这是因为 FaaS 每次执行函数时，都会初始化一个新的运行环境，然后从头开始执行整个代码，而不是只执行其中的 handler 方法。执行完毕后，运行环境就会被释放。这样每次函数执行，都是新的运行环境，自然不同函数之间就无法共用 pv 这个变量了。<strong data-nodeid="23082">这是 FaaS 的另一个特点，无状态</strong>。</p>
<p data-nodeid="22910">然而有时候你可能就想要在不同函数中共享内存，怎么办呢？这个问题抽象一下，就是分布式中的状态共享问题。<strong data-nodeid="23087">为了解决这个问题，所以 BaaS（Backend as a Service）也被纳入 Serverless 的一部分。</strong></p>
<p data-nodeid="22911">在 01 讲中，我提到 BaaS 本质上就是把后端功能封装起来，以接口的形式提供服务。BaaS 的涵盖范围比较广，包含任何应用运行所依赖的服务，典型的有数据库、中间件等。在 Serverless 架构中，常见的 BaaS 产品有 AWS DynamoDB、阿里云表格存储、消息中间件等，这些服务都可以通过 API 进行访问。这样在 FaaS 中，你就可以通过接口来使用 BaaS，基于 BaaS 来存储数据、实现函数间状态共享了。</p>
<p data-nodeid="22912">所以要实现 pv 统计功能，你可以用表格存储这个 BaaS 服务，表格存储是一个 NoSQL 数据库，可以通过 API 来访问，伪代码如下：</p>
<pre class="lang-javascript" data-nodeid="22913"><code data-language="javascript"><span class="hljs-keyword">const</span> tablestore = <span class="hljs-built_in">require</span>(<span class="hljs-string">'tablestore'</span>);
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">handler</span>(<span class="hljs-params">event</span>) </span>{
  <span class="hljs-comment">// 从表格存储中获取当前 PV</span>
  <span class="hljs-keyword">let</span> pv = <span class="hljs-keyword">await</span> tablestore.get();
  <span class="hljs-comment">// 用户每访问一次就给 pv 的值加 1</span>
  pv += <span class="hljs-number">1</span>;
  <span class="hljs-comment">// 保存最新 PV</span>
  <span class="hljs-keyword">await</span> tablestore.save(pv);
  <span class="hljs-keyword">return</span> pv;
}
</code></pre>
<p data-nodeid="22914"><strong data-nodeid="23094">基于 FaaS 和 BaaS 的架构，是一种计算和存储分离的架构。</strong> 计算由 FaaS 负责，存储由 BaaS 负责，计算和存储也被分开部署和收费。这使应用的存储不再是应用本身的一部分，而是演变成了独立的云服务，降低了数据丢失的风险。而应用本身也变成了无状态的应用，更容易进行调度和扩缩容。</p>
<p data-nodeid="22915">基于 FaaS 和 BaaS ，你的应用就实现了自动弹性伸缩、按量付费、不用关心服务器，这正是 Serverless 架构的必要因素。<strong data-nodeid="23099">所以说狭义的 Serverless 是 FaaS 和 BaaS 的组合。</strong></p>
<h3 data-nodeid="22916">什么不是 Serverless</h3>
<p data-nodeid="22917">除了广义和狭义上的 Serverless， 我经常被同学们问到另一个问题，那就是 PaaS、Kubernetes 、云原生等技术是不是 Serverless？如果不是，它们与 Serverless 有什么关系？这个问题很常见，你可以根据我刚刚总结的 Serverless 的特点进行判断。</p>
<p data-nodeid="22918">在01讲中我已经提到了，<strong data-nodeid="23107">PaaS （平台即服务）是云计算虚拟机时代的主要形态之一。</strong> 它是指云厂商提供开发工具、依赖库、服务和运行平台等能力，开发者可以依赖这些能力将自己的应用直接部署在云平台上，不用关心底层的计算资源、网络、存储等。虽然与Serverless 很类似，但依旧存在一些区别。</p>
<ul data-nodeid="22919">
<li data-nodeid="22920">
<p data-nodeid="22921"><strong data-nodeid="23112">付费标准：</strong> PaaS 的部署单位是应用，大部分 PaaS 还是将应用部署直接在服务器（虚拟机）上。所以我们对 PaaS 付费，依然是按资源付费，而不是按实际使用量付费。</p>
</li>
<li data-nodeid="22922">
<p data-nodeid="22923"><strong data-nodeid="23117">弹性伸缩：</strong> 有的 PaaS 虽然提供了弹性伸缩能力，但只能针对底层的服务器进行扩缩容。而 Serverless 的弹性伸缩是请求级别的，扩容速度更快，资源利用也更高效。</p>
</li>
</ul>
<p data-nodeid="22924"><strong data-nodeid="23122">Kubernetes 本身也不是 Serverless，只是在概念方面有些类似。</strong> Kubernetes 是一种容器编排技术。在 Kubernetes 中应用运行的基本单位是 Pod（容器组），Pod 是应用及运行环境的集合，所以你也不用关心服务器了。基于 Kubernetes，你能很方便地进行 Pod 的管理，并且实现应用的弹性伸缩。</p>
<p data-nodeid="22925">但从运维的角度来看，主流的 Kubernetes 服务提供商，如 EKS (Amazon Elastic Kubernetes Service) 和 ACK（阿里云容器服务），提供的都是 Kubernetes 集群托管和运维服务，开发者可以方便地管理 Kubernetes 集群中硬件、存储、Pod 等资源，但上层应用的运维和调度还是需要开发者自己进行。</p>
<p data-nodeid="22926">从成本的角度来看，Kubernetes 也无法做到按代码执行次数和实际消耗资源计费，还是和传统的 Serverful 一样，按照资源数量计费。</p>
<p data-nodeid="22927"><strong data-nodeid="23128">所以，Kubernetes 是介于 Serverful 和 Serverless 中间的产物。</strong></p>
<p data-nodeid="22928">而云原生指的是原生为云设计的架构模式，就是应用一开始设计开发就按照在云上运行的方式进行，充分利用云的优势。Serverless 几乎封装了所有的底层资源调度和运维工作，让你更容易使用云计算基础设施，极大简化了基于云服务的编程。</p>
<p data-nodeid="22929"><strong data-nodeid="23133">因此 Serverless 是云原生的一种实现，云原生的另一种实现是 Kubernetes。</strong></p>
<h3 data-nodeid="22930">Serverless 的优缺点</h3>
<p data-nodeid="22931">没有一项技术是十全十美的，Serverless 也一样。了解它的优缺点，可以让你今后更好地进行技术选型，决定是否用 Serverless 进行应用开发。<strong data-nodeid="23140">根据 Serverless 的定义，Serverless 的优点主要有：不用运维、弹性伸缩、节省成本、开发简单、降低风险、易于扩展。</strong> 但它也存在缺点。</p>
<ul data-nodeid="22932">
<li data-nodeid="22933">
<p data-nodeid="22934"><strong data-nodeid="23144">依赖第三方服务</strong></p>
</li>
</ul>
<p data-nodeid="22935">要用 Serverless，就要用云厂商提供的 Serverless 产品，比如 FaaS、BaaS，这样业务就和第三方云厂商绑定了。并且一旦你选择了一个云厂商，要想从一个云移到另一个台，成本很高（因为现在 Serverless 还没有一个统一的标准，云厂商各做各的，Serverless 产品也都不一样）。所以，依赖第三方服务是优点也是缺点。当然，我觉得这是大势所趋，让专业的人做专业的事，可以极大提高生产力。</p>
<ul data-nodeid="22936">
<li data-nodeid="22937">
<p data-nodeid="22938"><strong data-nodeid="23149">底层硬件的多样性</strong></p>
</li>
</ul>
<p data-nodeid="22939">目前 Serverless 的技术实现是 FaaS 和 BaaS。你的应用代码在 FaaS 上运行，但其底层的硬件资源多样，也不确定，云厂商可以灵活地选择服务器来运行你的代码，这就让运行函数的物理环境变得不同，甚至有的函数会运行在不同代的 CPU 上。 如果代码不依赖底层 CPU，那影响可能是不同 CPU 性能有差异；如果代码必须运行在某种类型的 CPU 或 GPU 上，那就需要云厂商提供这种能力了。这其实也暴露了云厂商的目的，就是最大化平衡资源利用效率与成本。当然，如果你不是特别关注底层硬件，影响也不大。</p>
<ul data-nodeid="22940">
<li data-nodeid="22941">
<p data-nodeid="22942"><strong data-nodeid="23154">应用性能瓶颈</strong></p>
</li>
</ul>
<p data-nodeid="22943">基于 Serverless 架构的应用，函数运行前需要现初始化函数运行环境，这个过程需要消耗一定时间。因为函数不是持续“在线”的，而是需要运行的时候才启动（不像传统应用，服务是一直启动的）。</p>
<p data-nodeid="22944">从资源利用率来讲，这种模式可以节省资源，但从应用性能上来讲，这就会降低应用性能，并且还要靠云厂商实现性能优化（让延时只有几毫秒或者几十毫秒，毕竟一个接口最大的耗时是在网络上，可能长达几百毫秒）。但如果你的应用对性能非常敏感，就需要考虑一下怎么去优化应用性能了。</p>
<ul data-nodeid="22945">
<li data-nodeid="22946">
<p data-nodeid="22947"><strong data-nodeid="23160">函数通信效率低</strong></p>
</li>
</ul>
<p data-nodeid="22948">传统的 MVC（Model-View-Controller） 架构模式中，View 层方法调用 Model 层方法，都是在内存中进行的。而在 Serverless 应用中，函数与函数之间就完全独立了。如果两个函数的数据有依赖，需要进行通信、交换数据，就要进行函数与函数之间的调用（调用方式是 HTTP 调用）。相比之前的内存调用，数据交互效率显然低了很多。而这个问题的本质，是 FaaS 还没有比较好的数据通信协议或方案。</p>
<ul data-nodeid="22949">
<li data-nodeid="22950">
<p data-nodeid="22951"><strong data-nodeid="23165">开发调试复杂</strong></p>
</li>
</ul>
<p data-nodeid="22952">Serverless 架构正处于飞速发展的阶段，其开发、调试、部署工具链并不完善（基本是每个云厂商各玩各的）。 另外，应用依赖的第三方云服务也很难进行调试。要想在本地开发调试 Serverless 应用，还是比较复杂。</p>
<p data-nodeid="22953">不过在我看来，虽然 Serverelss 存在缺点，但随着技术的不断成熟，这些问题在未来都能得到解决。</p>
<h3 data-nodeid="22954">总结</h3>
<p data-nodeid="22955">关于“ Serverless 到底是什么？”的文章很多，观点也很多，为了让你学会区分，对这个概念有一个准确的认知，我准备了这样一节课，希望你有所收获。</p>
<p data-nodeid="22956">这一讲我提到了 Serverless 、Serverless 架构和 Serverless 平台，其中，Serverless 是架构思想；基于 Serverless 思想的软件架构，就是 Serverless 架构；Serverless 平台指云厂商的 Serverless 相关的产品。关于今天这节课，我强调这样几个重点：</p>
<ul data-nodeid="22957">
<li data-nodeid="22958">
<p data-nodeid="22959"><strong data-nodeid="23175">广义上来讲，</strong> Serverless 是一种架构思想，即软件构建和运行时不需要关心服务器；</p>
</li>
<li data-nodeid="22960">
<p data-nodeid="22961"><strong data-nodeid="23180">狭义上来讲，</strong> Serverless 是 FaaS 和 BaaS 的组合，是当前主流的技术实现。</p>
</li>
<li data-nodeid="22962">
<p data-nodeid="22963"><strong data-nodeid="23185">Serverless 架构的主要特点是按量付费、弹性伸缩、不用运维，</strong> 这是区分一个架构是否是 Serverless 架构的关键因素。</p>
</li>
</ul>
<p data-nodeid="22964">本节课的作业是：在没学这一讲之前，你对 Serverless 的理解是什么呢？我总结的 6 个优点具体是什么意思？这个作业可以帮你回顾与梳理这一讲的内容，提高你的归纳总结能力。最后感谢你的阅读，我们下一讲见。</p>

---

### 精选评论

##### **凡：
> 我感觉还是局限性很多，至少现有的实现是的，对于fass或者bass这种，能支持的函数功能或者说接口功能十分有限，作为一个公司，这些接口还是得自己去开发部署成serverless模式才行，至于云厂商提供的那些函数和接口，也仅仅只适合那些非常公用的，适用性太窄。总得来说这个大的概念是不错的，但是目前云厂商得实现是很局限的，甚至于我觉得这是他们抠概念想出的一个新的卖点新的收费盈利方式，，，，

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 现在基于 FaaS 和 BaaS 的架构方式的确是云厂商在探索 Serverless 阶段的一个实现。就像你说的，要将已有代码迁移到 FaaS 确实有较大的改造成本，主要是 FaaS 本身有一定局限性。我觉得 FaaS 最大的局限在于它是无状态的服务，由于函数无状态，我们就需要使用数据库来存储状态，每次执行就需要访问数据库，就会有大量的网络 I/O，进而降低代码执行的效率。我们大多数应用都是有状态的，比如本地缓存、数据库连接池以及进行通信等，FaaS 的局限就会让我们开发复杂的后端应用变得困难。但换个角度思考，无状态的代码也有很多好处，比如方便开发调试，利于扩容缩容，无状态应用可以很容易地进行复制和重启。我们也可以去思考现有应用架构，是否需要复杂的状态共享、数据通信，能否更轻量化？毕竟越简单越易于维护。我觉得 FaaS 是迈向 Serverless 的一小步，它的主要创新就在于自动弹性伸缩和按资源使用量计费。但目前 FaaS 的局限也使得它不是 Serverless 的终态，我也非常期待能出现一种超越 FaaS 的架构，能够根据用户的计算和数据指标，动态地管理资源的分配，让我们更高效地使用云上无限的计算力和存储资源。

##### **俊：
> 对于函数之间通讯，目前已经有方案了，基于事件总线event bus，也有了对应的标准，各大厂商现在都在去实现

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 事件总线的确是目前比较通用的函数通信的方案，不过它本质上还是生产者消费-消费者的设计模式，这就需要消费者知道生产者的数据什么时候可用，另外还可能存在一定网络延迟。我觉得最优的通信方案是能够达到毫秒级的延迟，能够保证可靠的送到，能够高效率地通讯。传统基于虚拟机，代码可以运行在同一台实例上，进程可以通过共享数据的方式来实现通信，如果需要多个虚拟机实例之间通信，还可以在本地对消息进行聚合。而函数实例由于无法在本地共享、聚合数据，通信效率会低很多，最终也会产生更多的消息数量。

##### **俩事：
> 讲述Serverless无状态时，“执行完毕后，运行环境就会被释放。”这个不严谨吧，并没有立即被释放，如果连续触发的话，pv仍然会依次增加。环境会保存一定的时间，所以，有一个预热的优化。问题：怎么去了解环境会持续多久后销毁呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好问题。本节课中为了方便理解，就说了 “执行完毕后，运行环境就会被释放”，但这句话确实不严谨，原因就是你说的，FaaS 平台通常会将执行环境保留一段时间，提升函数启动的性能，使之后函数都是热启动的。
对于公有云上的 FaaS 平台，要了解运行环境多久后被销毁，就只能去看它们的相关产品文档了，通常可能是几分钟到几十分钟。由于这种不确定存在，所以我们编写代码时，就不能依赖于上一次执行结果，需要假设“函数执行完毕后运行环境会被释放”。

##### *心：
> midwayjs怎样

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Midway 是对 egg.js 的上层封装。我觉得 Midway 的主要优势是 TypeScript、依赖注入，以及无缝使用 Egg.js 生态，对于复杂应用，TypeScript 可以提升代码的可读性和可维护性以及稳定性，依赖注入可以降低代码耦合度、提升开发效率。

此外 Midway 团队还开源了 Middway FaaS 这个 Serverless 框架，可以用来开发 Serverless 应用，目前支持阿里云和腾讯云，这也是目前国内做的比较好的 Serverless 框架。

##### **X：
> 私有化部署的serverless也不需要企业自己维护吗？比如版本升级，自有存储高可用之类的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果是企业自建的私有化 Serverless 平台，则需要企业自己维护。此外，私有化部署 Serverless 也可以使用一些云厂商提供的私有化版本，比如 Azure Functions 就支持私有化部署，这样就不需要企业自己维护，而是由云厂商维护。

##### **用户7688：
> 谢谢大神😊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢信任，互相学习

##### **用户4908：
> 老师能补充一些开源的serverless解决方案么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 现在开源的 Serverless 解决方案有很多，比如 OpenWhisk、Fission、Kubeless、OpenFaaS、Fn 等。

OpenWhisk 起源于2016年2月，是由 IMB 开发的，IBM 的 Cloud Functions 就是基于OpenWhisk 实现的。后来 IMB 将 OpenWhisk 捐赠给了 Apache 基金会。OpenWhisk可以运行在物理机、虚拟机和 kubernetes 等多种不同的基础架构上。 OpenWhisk 是用 Scale 编写的。

Fisson 起源于 2016年 11 月，是  Platform9 公司的 Soam Vasani 及少数几位工程师开源的 Serverless 解决方案。Fisson 运行在 kubernetes 集群上。通常基于 Docker 的 FaaS 冷启动要 200 毫秒左右，而 Fisson 通过预热容器、容器重用等技术，实现了低于 100毫秒的冷启动。Fisson 是用 Go 编写的。

Kubeless 起源于 2016 年 8 月，是纯开源的软件。和 Fisson 一样，kubeless 也是运行在运行 kubernetes 的，kubernetes 使用了很多 Kubernetes原生的组件，如 Service、 Ingress、 HPA（ Horizontal Pod Autoscaler）等。Kubeless 是用Go 编写的。

OpenFaaS 起源于 2016 年 12 月，最初是由一个 Alex Ellis 编写的，主要特点是简单易用，并且支持  Kubernetes 和 Docker Swarm。OpenFaaS 也是用Go 编写的。

##### **凡：
> 还有，所谓的不需要维护，也仅仅是云厂商提供的函数和接口不需要维护，自己开发实现的依然得维护。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 传统开发模式中，我们需要维护服务器、网络以及数据库等，需要根据业务流量进行资源扩缩。但基于 FaaS 和 BaaS，底层计算存储等资源都不需要我们关心了，我们只需要维护业务代码。

##### lin yuehuang：
> 所以如果是在政府的内网环境里，serverless就无用武之地了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在政府的内网环境里可以部署私有 Serverless 平台。比如 Azure Functions 就同时支持公有云和私有云，你可以在私有数据中心部署 Azure Functions。除此你也可以使用一些开源解决方案来部署私有 Serverless 平台。

