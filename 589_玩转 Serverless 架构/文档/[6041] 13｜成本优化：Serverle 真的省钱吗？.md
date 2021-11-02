<p data-nodeid="33633" class="">今天这一讲我想和你讨论一下 Serverless 应用的成本优化。</p>


<p data-nodeid="32810">我们一直说 Serverless 可以节省成本，各大云厂商也是这样宣传的。可能一些同学会有疑惑：Serverless 真的省钱吗？到底有多省钱呢？之前有同学给了我一个数据：同规格（1G内存）情况下，函数一小时的费用比云服务器一小时的费用大约要高 2.5 倍，所以 Serverless 云计算要比传统的 Serverful 云计算贵得多。那么实际情况是什么样呢？</p>
<p data-nodeid="32811">为了解答你 Serverless 成本方面的疑惑，所以我准备了今天的内容，希望通过今天的学习，你能学会分析、优化和控制 Serverless 应用的成本（尤其对技术 Leader 来说，在进行技术选型时，除了考虑技术架构、稳定性等，更需要考虑成本）。</p>
<h3 data-nodeid="32812">Serverless 应用的成本分析</h3>
<p data-nodeid="32813">为什么有的同学会说：函数一小时的费用比云服务器一小时的费用大约要高 2.5 倍呢？这个数据是怎么来的呢？要解答这个问题，你要先了解 FaaS 和云服务器的计费逻辑。</p>
<p data-nodeid="32814">目前大多数 FaaS 都是按照函数执行次数和函数执行消耗的内存来收费的。</p>
<ul data-nodeid="32815">
<li data-nodeid="32816">
<p data-nodeid="32817">执行次数：1.3元/百万次。</p>
</li>
<li data-nodeid="32818">
<p data-nodeid="32819">执行时间（按量付费）：0.0110592 元 / 1 GB-秒。表示函数按照 100GB 内存运行 1s，费用是 0.003167元（目前 AWS Lambda、阿里云函数计算等也都实现了 1 毫秒粒度的计费周期）。</p>
</li>
</ul>
<p data-nodeid="32820">FaaS 平台的计费模式和收费标准最早是 AWS Lambda 提出来的，各大云厂商的 FaaS 平台也都参考了 Lambda 的计费规则，这也是为什么大多数 FaaS 平台都是同样的计费标准。按照这个计费标准，我们来计算一下函数持续运行一个小时的费用。</p>
<p data-nodeid="32821">假设函数在一小时内只运行一次，消耗内存是 1GB，函数运行时间为 1 小时，费用为：</p>
<pre class="lang-java" data-nodeid="32822"><code data-language="java"><span class="hljs-number">0.00000133</span> + <span class="hljs-number">60</span> * <span class="hljs-number">60</span> * <span class="hljs-number">0.000110592</span> = <span class="hljs-number">0.398</span> 元
</code></pre>
<p data-nodeid="32823">而使用按量付费的云服务器，以阿里云 ECS 为例，型号为 ecs.s6-c1m1.small 的 1核 1G 的服务器 1 小时的费用为 0.157 元。<br>
这么来看，函数的费用确实大约是服务器的 2.5 倍，但这里面存在两个误区。</p>
<ul data-nodeid="35308">
<li data-nodeid="35309">
<p data-nodeid="35310"><strong data-nodeid="35318">Serverless 的收费包含了应用的系统管理功能：</strong> 包括自动弹性伸缩、监控、日志、可用性、备份容灾能力等，单个函数的能力其实完全超过了单个云服务器实例的功能。对于云服务器，我们通常很难 100% 利用其 CPU 和内存等资源，并且一旦资源利用率过高，还会影响应用的性能和稳定性。而 Serverless 函数则可以完全利用分配的 CPU 和内存。</p>
</li>
<li data-nodeid="35311">
<p data-nodeid="35312"><strong data-nodeid="35323">Serverless 的函数并不是持续运行的：</strong> 前面计算费用时，为了简化计算方式，我们假设函数会持续运行一个小时，但在实际情况中，函数每次只会运行几十毫秒到几百毫秒，不运行时不收费，所以实际产生的费用可能要便宜得多。而云服务器的按量付费，是按照你购买了之后的持有资源的时间长度计费，比如你购买云服务器后持有了一个月（即没有释放该服务器），在这一个月内，就算你不在这台服务器上运行任何应用，云厂商依旧会收取一个月的费用。</p>
</li>
</ul>
<p data-nodeid="35313" class=""><strong data-nodeid="35328">接下来我们再看一个更接近真实的例子：</strong> 假设你的网站每天处理 10 万个请求，每个请求处理时间大约为 100ms，每个函数需要 1G 内存，则一个月的费用为：</p>




<pre class="lang-java" data-nodeid="32830"><code data-language="java">(<span class="hljs-number">0.00000133</span> * <span class="hljs-number">100000</span>  + <span class="hljs-number">0.1</span> * <span class="hljs-number">0.000110592</span> * <span class="hljs-number">100000</span>) * <span class="hljs-number">30</span> = <span class="hljs-number">37.16</span> 元
</code></pre>
<p data-nodeid="35881">而阿里云一台 1核 1G 的云服务器每个月费用为 53 元，为了保证服务的可用性，一般至少还需要准备 2 台云服务器，费用为 106 元，<strong data-nodeid="35887">所以 Serverless 的价格要比传统 Serverful 便宜很多。</strong></p>
<p data-nodeid="35882">当应用变得更复杂，使用 Serverless 的成本是随着请求量的大小线性增加的。而 Serverful 的架构，则是不断增加新的服务器，并且还需要为流量峰值准备资源，成本可能成倍增加。</p>

<p data-nodeid="36441" class=""><strong data-nodeid="36446">我之前也做过一个调研：</strong> 大型企业中除 20% 的核心应用外，剩余 80% 的都是中长尾应用，这些应用平均 QPS 远远低于 1， 甚至一天也没有几个用户访问，但传统的 Serverful 架构中又不得不为这些应用准备至少 2 台服务器，使用 Serverless 则可以极大节省这部分成本。另外，很多 Serverless 平台都有一定的免费额度，对于很多小应用来说，使用 Serverless 几乎是完全免费的。</p>

<p data-nodeid="32833">除了我们假设的例子，你也可以从一些公开的案例中发现，国内外不管是大厂小厂使用 Serverless 都能极大降低成本，例如：</p>
<ul data-nodeid="32834">
<li data-nodeid="32835">
<p data-nodeid="32836">Financial Engines 使用 AWS Lambda 和 Serverless 将成本降低 90％；</p>
</li>
<li data-nodeid="32837">
<p data-nodeid="32838">微博使用阿里云函数计算进行个性化图片处理，日处理数十亿次请求，综合成本下降 35%；</p>
</li>
<li data-nodeid="32839">
<p data-nodeid="32840">世纪联华 2020 年双 11 基于阿里云函数计算开发大促会场服务端渲染、线上商品秒杀、优惠券定点发放等业务，整体成本减少 40% 以上。</p>
</li>
</ul>
<p data-nodeid="32841">整体而言，使用 Serverless 你不用再为空闲服务器付费，也不用担心不可预测的负载，所以 Serverless 能大幅降低成本。就我个人经验而言，当大型企业中服务器利用率低于 20% 时，使用 Serverless 更能节省很大一笔费用。当你准备选择 Serverless 架构时，你可能也需要根据业务调用量来预估成本，为了方便计算，你也可以使用一些云厂商提供的或开源的 Serverless 成本计算器，比如下面这些：</p>
<ul data-nodeid="32842">
<li data-nodeid="32843">
<p data-nodeid="32844"><a href="http://tools.functioncompute.com/" data-nodeid="32973">阿里云函数计算成本计算器</a>；</p>
</li>
<li data-nodeid="32845">
<p data-nodeid="32846"><a href="https://aws.amazon.com/cn/lambda/pricing/" data-nodeid="32977">AWS Lambda 成本计算器</a>；</p>
</li>
<li data-nodeid="32847">
<p data-nodeid="32848"><a href="https://cost-calculator.bref.sh/" data-nodeid="32981">Serverless costs calculator</a>。</p>
</li>
</ul>
<p data-nodeid="32849">可能对一些企业来说，Serverless 这种用多少花多少的收费模式会带来缺点：很难预测具体会产生多少费用。这和传统的管理预算的方式很不一样，企业通常以年为单位进行预算审批，因此需要知道下一年 Serverless 究竟会带来多少费用。这是一个合理的顾虑，所以云厂商通常也提供了预付费模式来解决这个问题，这就跟预付费购买云服务器一样。同时我也相信，随着越来越多的开发者和企业使用 Serverless，他们将可以根据历史数据来推测未来 Serverless 可能产生的费用。</p>
<p data-nodeid="32850">现在看来，Serverless 虽然已经够省钱了，那还能不能更省钱呢？毕竟对成本来说，我们永远只希望它更低，当然没问题，<strong data-nodeid="32988">接下来我们来学习如何优化 Serverless 的成本。</strong></p>
<h3 data-nodeid="32851">Serverless 应用的成本优化</h3>
<p data-nodeid="32852">由于 Serverless 应用是按照代码执行次数、执行时间进行收费，所以我们可以总结出下面这个公式：</p>
<pre class="lang-java" data-nodeid="32853"><code data-language="java">性能 = 时间 = 成本
</code></pre>
<p data-nodeid="32854">基于这个公式不难看出，对于一个 Serverless 应用来说，性能越好，函数运行时间也就越短，成本就越低。因此我根据自己的实践，总结出了一些常见的成本优化方案，供你参考。</p>
<ul data-nodeid="32855">
<li data-nodeid="32856">
<p data-nodeid="32857"><strong data-nodeid="32995">为函数设置超时时间</strong></p>
</li>
</ul>
<p data-nodeid="32858">第一点就是为函数设置超时时间，这样才能避免函数因为异常而无限制地运行下去，导致成本的上升。例如，通常我们会为提供 API 服务的函数设置 10 秒左右的超时时间。如果一个请求 10 秒以上才能返回结果，那说明这个函数需要拆分或优化了。</p>
<p data-nodeid="32859">而且在 12 讲我也提到了，为函数设置超时时间，可以降低 DDoS 攻击的风险，避免恶意攻击导致函数持续运行，造成资产损失。</p>
<ul data-nodeid="32860">
<li data-nodeid="32861">
<p data-nodeid="32862"><strong data-nodeid="33001">为函数分配合适的内存</strong></p>
</li>
</ul>
<p data-nodeid="32863">函数收费是按照 GB 每秒来收费的，所以内存越高，费用越高，但通常内存越高，函数运行速度又更快。</p>
<p data-nodeid="32864">假设一个函数以 128 MB 运行需要 200ms，则需要 0.0000040948 元；但如果函数以 1024MB 运行，只需要 20ms 就能执行完毕，则只需要 0.00000247784 元，比前者要节省一半的成本，所以你要为函数分配合适的内存，以此提升函数性能，节省成本。</p>
<p data-nodeid="32865">不过在成本和性能之间找到最佳结合点通常很棘手，你需要具体分析每个函数，可以以不同内存自动执行函数，并使用链路追踪持续观察函数的执行耗时情况，进而找到函数最佳内存。</p>
<ul data-nodeid="32866">
<li data-nodeid="32867">
<p data-nodeid="32868"><strong data-nodeid="33008">减少函数的冷启动耗时</strong></p>
</li>
</ul>
<p data-nodeid="32869">函数的冷启动会直接增加函数的执行时间，并且这部分时间是你的业务逻辑之外的耗时，所以减少函数的冷启动时间，可以帮你节省一大笔费用。关于如何减少冷启动，我在 09 讲中也提到了，主要有函数预热、选择冷启动耗时短的编程语言、减小代码体积、执行上下文重用等方案，如果你不记得了可以回顾一下。</p>
<ul data-nodeid="32870">
<li data-nodeid="32871">
<p data-nodeid="32872"><strong data-nodeid="33013">减少外部慢 API 调用</strong></p>
</li>
</ul>
<p data-nodeid="32873">通常调用外部 API 会涉及网络请求，如果外部 API 比较慢，函数执行过程就会阻塞，导致函数执行耗时增加，进而增加成本，所以尽量减少外部慢 API 的调用。但很多时候我们的函数也必须依赖外部 API，比如依赖第三方 OAuth 进行身份认证，这时就需要就近选择 OAuth 服务，比如在北京调用北京地域的 OAuth 服务接口，这样能大幅降低跨地域间的网络请求。另一方面，也可以适当对网络请求进行缓存，进而提升函数性能。</p>
<ul data-nodeid="32874">
<li data-nodeid="32875">
<p data-nodeid="32876"><strong data-nodeid="33018">为函数实例设置并发</strong></p>
</li>
</ul>
<p data-nodeid="32877">为函数实例设置并发，不仅能提升函数性能，还能节省函数成本。因为单个函数实例可以支持更多请求，多个请求共用一个函数实例，执行时间是从第一个请求开始，到最后一个请求结束为止，这样并发处理多个请求就能大幅降低成本。</p>
<p data-nodeid="39218" class=""><img src="https://s0.lgstatic.com/i/image/M00/94/41/CgqCHmAXusuAPUuzAAZNxvjF9MA299.png" alt="Drawing 0.png" data-nodeid="39222"></p>
<div data-nodeid="39219"><p style="text-align:center">单实例单并发</p></div>




<p data-nodeid="38663" class=""><img src="https://s0.lgstatic.com/i/image/M00/94/41/CgqCHmAXutWATIu3AAes0bVKSe4361.png" alt="Drawing 1.png" data-nodeid="38667"></p>
<div data-nodeid="38664"><p style="text-align:center">单实例多并发</p></div>



<p data-nodeid="32882">如图所示，单实例单并发情况下，T1-T2，T3-T4 之间都会计费，并且计费周期是两个函数的执行时长。而在单实例多并发的情况下，只计算 T1-T4 的整体时间，并且一个实例可以同时处理多个请求，最终按执行时间最长的请求计算执行耗时。假设单函数实例并发为 10，理论上可以节省 10 倍成本。</p>
<ul data-nodeid="32883">
<li data-nodeid="32884">
<p data-nodeid="32885"><strong data-nodeid="33032">选择合适的计费方式</strong></p>
</li>
</ul>
<p data-nodeid="32886">目前绝大部分 FaaS 平台都支持按量付费和预付费，你可以根据应用特点选择合适的付费方式。</p>
<p data-nodeid="32887">例如在生产环境中，如果应用流量一直很高且比较平稳，对延迟也比较敏感，通常你可以使用预留模式，预留一定函数实例，这样就能极大减少冷启动，从而降低成本。而日常测试或离线处理数据时，函数可能是临时大量执行，这时就可以使用按量付费，在不需要执行函数时就不会消耗任何资源，这样就能保持较高的资源利用率，进而降低成本。</p>
<p data-nodeid="40047" class=""><img src="https://s0.lgstatic.com/i/image2/M01/0C/2E/Cip5yGAXuuKAZuKXAAbyIr7ir5M253.png" alt="Drawing 2.png" data-nodeid="40051"></p>
<div data-nodeid="40048"><p style="text-align:center">预留资源</p></div>



<p data-nodeid="32890">如图所示，这是预留资源的情况，T1 时刻用户创建预留资源，这时 FaaS 平台开始创建函数实例，同时开始计费，之后所有请求都会使用预留的函数实例，T2 时刻用户释放预留实例，计费结束。</p>
<p data-nodeid="32891">总的来说，在 Serverless 架构中，我们可以通过提升应用性能来降低成本，而在传统 Serverful 架构中，性能和成本却几乎没有关系。这也是 Serverless 的一大特点。</p>
<p data-nodeid="32892">虽然我们已经掌握了一些 Serverless 架构中的成本优化方案，但有时候我们还是很难理解云厂商提供的账单，因为里面经常会有一些预期之外的费用。传统我们可以通过预付费的方式购买 1000 台云服务器，这样能准确知道要付多少钱。但基于 Serverless 架构，账单变得动态起来，费用难以预测。这时我们就需要一些控制成本的方案。</p>
<h3 data-nodeid="40600">Serverless 应用的成本控制</h3>


<p data-nodeid="32895">为什么 Serverless 账单难以预测呢？原因如下：</p>
<ul data-nodeid="32896">
<li data-nodeid="32897">
<p data-nodeid="32898">应用是弹性的，很难预测函数到底要执行多少次；</p>
</li>
<li data-nodeid="32899">
<p data-nodeid="32900">应用通常不仅仅包含 FaaS 的费用，还有其他组合使用的云产品的费用，而这些费用可能比 FaaS 的成本还高。</p>
</li>
</ul>
<p data-nodeid="32901">比如你可能使用 API 网关和 FaaS 的组合来实现 Restful API，你可能还需要使用 AWS CloudWatch Logs 或阿里云日志服务等云产品来存储函数日志，你可能也需要使用 DynamoDB 或表格存储来保存数据。而在前面的成本分析中，之所以没有把 FaaS 之外的成本包含在内，是因为基于 Serverful 的架构，你也可能需要使用到这些产品。当然可能有的企业有能力去开发自己的 API 网关或日志服务，而不使用云产品，因此能节省这些云产品的开支，但取而代之的是自己维护这些服务的人力和机器成本，不过很多使用 Serverless 的开发者和企业一开始可能会忽略掉这些成本。</p>
<p data-nodeid="32902">总的来说，Serverelss 应用的成本，除了 FaaS 之外，还需要考虑事件源的成本和相关服务成本，比如 API 网关、消息触发器以及各种 BaaS 服务。所以进行 Serverless 应用的成本控制时，也需要把这些考虑进去。</p>
<p data-nodeid="32903">那么如何控制成本呢？你可以从两个角度进行：</p>
<ul data-nodeid="32904">
<li data-nodeid="32905">
<p data-nodeid="32906"><strong data-nodeid="33052">成本预测</strong></p>
</li>
</ul>
<p data-nodeid="32907">成本预测主要就是根据以往的数据预测未来的成本。预测是一种简单但实用的方案。通常一个应用的费用是有一定周期性的，周期可能是月也可能是季度。</p>
<p data-nodeid="32908">下面是一个简单的计算公式：</p>
<pre class="lang-java" data-nodeid="32909"><code data-language="java">月末费用 = 当前费用 *（每月 / 今天的天数）
</code></pre>
<p data-nodeid="32910">例如，如果一个月中有 30 天，今天是第 15 天，总费用为 1000 元，那么到月末的估计费用为 2000 元。当然，你也可以根据上月的消费情况来预测当月每天的消费情况。</p>
<ul data-nodeid="32911">
<li data-nodeid="32912">
<p data-nodeid="32913"><strong data-nodeid="33059">成本监控</strong></p>
</li>
</ul>
<p data-nodeid="32914">就像我们使用监控来观测程序是否正常运行一样，Serverless 应用的成本监控也非常重要。**我曾经就遇到过这样的情况：**一个同学的函数使用了 API 网关和表格存储等服务，由于函数被恶意攻击最终产生了大量费用，直到月底收到账单通知才发现。这就是缺乏成本监控导致的。</p>
<p data-nodeid="32915">成本一方面是由 Serverless 函数运行时产生的，另一方面是由函数依赖的云服务产生的。对于前者，你可以设置函数调用次数的监控，对于后者，就需要查看每个云产品的使用及费用信息。</p>
<p data-nodeid="41148" class="te-preview-highlight">如果你具备一定的开发能力，则可以基于 Serverless 开发一个简单的成本监控程序，定时拉取函数运行日志以及账单数据，分析一段期间内的函数费用情况并进行报警，进而达到成本监控的目的。此外你也可以使用一些第三方的成本分析和成本监控平台，比如 <a href="https://dashbird.io/" data-nodeid="41152">dashbird</a>、<a href="https://epsagon.com/" data-nodeid="41156">epsagon</a>，不过遗憾的是这些产品都只适用于 AWS、Azure 等国外 Serverless 平台，而国内关于 Serverelss 应用成本分析的相关产品几乎没有。</p>

<h3 data-nodeid="32917">总结</h3>
<p data-nodeid="32918">这一讲，我从成本分析、成本优化和成本控制三个角度，详细讲解了 Serverelss 应用成本方面的相关知识。整体而言，基于 Serverless 架构，我们不用再为闲置的服务器付费，只为实际使用的资源付费就可以了。</p>
<p data-nodeid="32919">同时你还可以通过提高 Serverelss 应用的性能，进一步优化成本。由于 Serverless 应用通常依赖 FaaS 之外的触发器、数据源和 BaaS 服务，所以在分析和控制 Serverelss 应用的成本时，也需要关注这些云服务的成本。与 Serverless 应用的安全防护一样，目前国外已经有一些优秀的 Serverless 成本分析相关的产品，而国内还是一片空白，这对我们来说也是一个机会。</p>
<p data-nodeid="32920">关于这一讲，我想要强调以下几点：</p>
<ul data-nodeid="32921">
<li data-nodeid="32922">
<p data-nodeid="32923">Serverless 应用的成本包括 FaaS 中函数执行的成本，以及函数所依赖的触发器、数据源和 BaaS 服务的成本；</p>
</li>
<li data-nodeid="32924">
<p data-nodeid="32925">Serverless 中函数按照执行次数和执行时间进行收费，因此能大幅降低成本；</p>
</li>
<li data-nodeid="32926">
<p data-nodeid="32927">我们可以通过提升 Serverless 函数的性能来优化成本。</p>
</li>
</ul>
<p data-nodeid="32928">那么除了今天的内容，你还知道哪些成本优化的方案呢？欢迎在评论区留言。我们下一讲见。</p>

---

### 精选评论

##### **俊：
> 函数的并发数，是依据什么进行设置的？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 根据业务需求进行设置。比如你的应用是 I/O 密集型的，就适合设置较大的并发，因为因为 I/O 操作大部分时间都是在等待响应。如果是 CPU 密集型的任务，则不太适合多并发，会导致多个请求争抢实例资源。此外并发也有上限，函数计算上限是 100。

