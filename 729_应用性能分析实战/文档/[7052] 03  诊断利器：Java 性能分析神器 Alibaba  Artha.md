<p data-nodeid="2237" class="">Arthas 是阿里开源的 Java 诊断工具，可以让开发者快速定位故障问题，那么为什么用“神器”来形容 Arthas 呢？</p>
<p data-nodeid="2238">Arthas 由阿里早期的开源诊断工具 Greys 演进而来，你可以将 Arthas 看作是Greys的升级产品。由于 Arthas 提供了更加丰富的工具，在 2018 年底发布，就受到了很多开发者的青睐，并且也得到了来自 Java 官方的赞许。</p>
<p data-nodeid="2239">Arthas 的核心应用场景如下。</p>
<ul data-nodeid="2240">
<li data-nodeid="2241">
<p data-nodeid="2242">场景 1：这个类从哪个 jar 包加载的？为什么会报各种类相关的 Exception？</p>
</li>
<li data-nodeid="2243">
<p data-nodeid="2244">场景 2：我改的代码为什么没有执行？难道是我没 commit？分支搞错了？</p>
</li>
<li data-nodeid="2245">
<p data-nodeid="2246">场景 3：遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？</p>
</li>
<li data-nodeid="2247">
<p data-nodeid="2248">场景 4：线上遇到某个用户的数据处理有问题，但线上同样无法 debug，线下无法重现！</p>
</li>
<li data-nodeid="2249">
<p data-nodeid="2250">场景 5：是否有一个全局视角来查看系统的运行状况？</p>
</li>
<li data-nodeid="2251">
<p data-nodeid="2252">场景 6：有什么办法可以监控到 JVM 的实时运行状态？</p>
</li>
<li data-nodeid="2253">
<p data-nodeid="2254">场景 7：怎么快速定位应用的热点，生成火焰图？</p>
</li>
</ul>
<p data-nodeid="2255"><strong data-nodeid="2409">总结起来就是，Arthas 不仅能解决开发者无法对线上服务进行 Remote Debug 的难题，还能以全局的视角进行在线监控诊断和热修复，轻松化解让人束手无策的难题。可以说，企业部署了 Arthas，就如同拥有了在线定位问题的神器。</strong></p>
<p data-nodeid="2256">接下来，我会先介绍 Arthas 的学习路径，并结合具体场景，让你逐步认识 Arthas 的所有工具，并能在已知的场景中使用相应的工具，对线上问题打出组合拳。最后，我会分享我的落地经验，让 Arthas 提供的能力最大化。</p>
<h3 data-nodeid="2257">学习路径：命令+场景+原理</h3>
<p data-nodeid="2258">Arthas 目前已经集成了 41 个在线命令，那么如何快速掌握这么多的命令呢？我建议的学习路径是：命令+场景+原理，你可以按照这三个方向，来持续迭代自己掌握 Arthas 的能力。</p>
<p data-nodeid="2259"><img src="https://s0.lgstatic.com/i/image6/M01/2D/B7/Cgp9HWBm5BaAPOoWAABxIX9uEqs906.png" alt="Drawing 0.png" data-nodeid="2415"></p>
<h4 data-nodeid="2260">1.通过实操，熟悉命令</h4>
<p data-nodeid="2261">这 41 个命令，都是开发者结合实际的场景、分析背后的原理，总结出的好用的命令。你可以先根据<a href="https://arthas.aliyun.com/doc/arthas-tutorials.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2420">官网的在线教程</a>，按照四个分类：基础命令、系统命令、类命令和增强命令，将这些<strong data-nodeid="2426">命令</strong>通通实操一遍，对 Arthas 命令有个全局的感性认知。</p>
<blockquote data-nodeid="2262">
<p data-nodeid="2263">如果你是新手，可以不用着急学习下面的内容，可以根据 Arthas 的在线教程，先动手实践一下。</p>
</blockquote>
<h4 data-nodeid="2264">2.深入场景，进行诊断</h4>
<p data-nodeid="2265">掌握好“命令”后，你可以根据实际场景，持续梳理每个命令在场景中发挥的作用，对 Arthas 命令有更理性、全面的认识。</p>
<p data-nodeid="2266">当我们遇到线上问题时，我们会通过命令对线上的问题剥茧抽丝，不断深入场景的每一个细节。在这个过程中，你会发现，虽然每个命令设计得非常简单，但随着我们不断地加深对它的理解，并能将它们灵活组合时，便能达到强大效果。</p>
<p data-nodeid="2267">下面便是我对 Arthas 官网上七大场景的总结，这七个场景也最能体现 Arthas 产品的价值。</p>
<p data-nodeid="4489" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/D6/CioPOWBm-0GAZC_MAATTkxNsFlc217.png" alt="图片1.png" data-nodeid="4492"></p>


<p data-nodeid="2320" class="">比如在“场景 3：遇到问题无法在线上 debug，难道只能通过加日志再重新发布吗？”中，通过 4 个命令的组合，便实现了热发布功能。Arthas 设计的理念和<a href="https://en.wikipedia.org/wiki/Unix_philosophy?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2660">Unix 哲学</a>很像，每个命令只专注做好一个任务，然后充分发挥开发者的能力，通过不断地使用这些命令排查问题，在实践场景中积累实战经验。之后，再遇到相似场景时，便能快速通过一系列组合的命令，诊断出问题所在了。</p>
<p data-nodeid="2321" class="">但是，我们也要注意，积累的经验越多，越容易让我们犯“经验主义”的错误。就是遇到问题以经验先入为主，在排查问题时无法放空自己，让排查问题的能力提升出现瓶颈。</p>
<p data-nodeid="2322">所以，这时就需要深入掌握原理。只有掌握了原理，你才能以更高更全面的视角，深入场景，进行问题的诊断。避免之前知识碎片化，无法形成知识面的情况。</p>
<h4 data-nodeid="2323">3.掌握原理，提升效率</h4>
<p data-nodeid="2324">首先我们需要掌握应用服务相关依赖的原理，比如框架原理、JVM 原理。理解这些依赖原理后，再来理解 Arthas 的原理，包括 Arthas 每个命令是如何工作的。</p>
<p data-nodeid="2325">Arthas 的原理，我们可以通过在线文档和源码，并根据实际场景来针对性深入。</p>
<p data-nodeid="2326">接下来，我以最新支持的“场景 7：怎么快速定位应用的热点，生成火焰图？”为例，通过以下两个建议，向你讲解如何快速掌握 profiler 命令的原理。</p>
<p data-nodeid="2327"><strong data-nodeid="2671">建议 1 精读设计，略读源码</strong></p>
<p data-nodeid="2328">因为 Arthas（甚至所有的 APM 产品）技术栈相对应用服务技术栈，较为小众、底层，学习源码近似于在读另一门语言。就像应用服务，其开发语言是汉字，书写源码时用的也是普通话，非常通俗易懂；而 APM 底层技术栈虽然使用的也是汉字，但由于受众很少，源码阅读起来就像是文言文那般深奥、复杂了。</p>
<p data-nodeid="2329"><strong data-nodeid="2676">所以不正确地苛求源码只会浪费时间，与其如此，我们更加该去学习“文言文转换的白话文”，也就是精读设计。</strong></p>
<p data-nodeid="2330">接下来，我以 Arthas 的 profiler 命令，制作 CPU 维度的火焰图为例，来讲述如何掌握 Arthas 每个命令的学习过程。</p>
<p data-nodeid="2331">火焰图的制作使用<a href="https://github.com/jvm-profiling-tools/async-profiler?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2681">async-profiler</a>工具来完成。故此，就要学习操作系统的 perf 命令。操作系统的 perf 命令，需要定时不断采集 CPU 的正在执行的函数名和调用栈。由于 CPU 是机器运算的核心，所以采集到的数据是海量级别的。</p>
<p data-nodeid="2332">所以，我们需要一种形式将这些海量数据呈现出来。从而我们就设计了一张图，X 轴表示调用函数被采集的情况（按照函数字母排序），Y 轴表示详细描述了调用栈。整个绘制出来的图案，加上暖色后看起来很像一副<a href="https://github.com/brendangregg/FlameGraph?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="2686">火焰图</a>（火焰图的颜色没有特殊含义）。</p>
<p data-nodeid="2333"><img src="https://s0.lgstatic.com/i/image6/M01/2D/B8/Cgp9HWBm5DKAU6qGAAXfqh87K-I401.png" alt="Drawing 1.png" data-nodeid="2690"></p>
<p data-nodeid="2334">在这个过程中，你会发现我们并没有学习 Arthas profiler 实现火焰图源码，而是通过对火焰图的产品设计进行了深入学习，就达到了掌握火焰图和 Arthas profiler 命令的效果，可见精读设计，略读源码的效果。</p>
<p data-nodeid="2335"><strong data-nodeid="2695">建议 2 原理之上，用好工具</strong></p>
<p data-nodeid="2336">工具的目的是提升工作效率，所以每个开发者都逃避不了被问到：你是如何使用 Arthas 的火焰图快速分析出应用服务性能问题的呢？</p>
<p data-nodeid="2337">我给出的答案是，非预期的火焰图的<strong data-nodeid="2702">屏顶</strong>，就是应用服务性能问题的所在。</p>
<p data-nodeid="2338">以下面购买服务的 CPU 火焰图为例：</p>
<p data-nodeid="2339"><img src="https://s0.lgstatic.com/i/image6/M00/2D/C0/CioPOWBm5D6AelwSAAC2dlxp9uc971.png" alt="Drawing 2.png" data-nodeid="2706"></p>
<p data-nodeid="2340">非预期的屏顶出现在 failAccountError() 和 failNotEnoughMoney() 两个函数上，接下来我可以通过 tt 和 watch 命令详细观察性能问题的原因。</p>
<blockquote data-nodeid="2341">
<p data-nodeid="2342">这里我只是以“场景 7 的火焰图”为例，至于其他场景及其命令，也可以按照这两点建议进行原理探索。</p>
</blockquote>
<p data-nodeid="2343">Arthas 的学习路径讲完了，你会发现：<strong data-nodeid="2713">使用 Arthas 命令进行问题分析，再掌握场景背后的原理，就是一个从命令到场景，再到原理，三者认知不断螺旋上升的过程。</strong></p>
<h3 data-nodeid="2344">两套部署方案</h3>
<p data-nodeid="2345">既然 Arthas 对诊断线上问题这么有帮助，那应该如何部署 Arthas 呢？</p>
<p data-nodeid="2346">我的理解是：工欲善其事必先利其器，相比较其他 APM 产品，Arthas 更专注于线上问题的及时诊断。线上问题的诊断越快越好，所以你需要提前将 Arthas 部署好，当线上问题出现时，确保 Arthas 能开箱即用。</p>
<p data-nodeid="2347">我这里有两套部署方案，你可以根据自身团队的情况，针对各个方案的优势进行部署：</p>
<p data-nodeid="5061" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/2D/D6/CioPOWBm-2iAFQfSAAHJSv_DCB0568.png" alt="图片2.png" data-nodeid="5064"></p>

<h3 data-nodeid="2388" class="">小结与思考</h3>
<p data-nodeid="2389" class="">今天的课程，我带你学习了 Java 在线诊断工具 Arthas。</p>
<p data-nodeid="2390" class="">Arthas 目前有非常多的在线诊断命令，我们要对问题场景进行剥茧抽丝，不断深入细节定位问题，然后总结什么样的场景，应该是用什么样的命令，深入学习每个命令背后的原理设计。</p>
<p data-nodeid="2391" class="">希望你在使用 Arthas 命令定位线上问题时，分析过程不要专注某一个点，抱有尝试着改下就能解决的侥幸心理；而是要以一种科学的方法来寻找和定位问题，用数据来指导问题定位的方向。</p>
<p data-nodeid="2392">那么，你用 Arthas 定位过什么那些场景呢？请把你的场景特点，使用哪些命令、事中事后的总结与思考写在评论区，期待与你讨论。</p>
<p data-nodeid="2796" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/D6/CioPOWBm-x6AH9BYAAE8EwU18Vg656.png" alt="应用性能分析实战金句-03.png" data-nodeid="2799"></p>

---

### 精选评论

##### *奇：
> 老师好，能不能详细的介绍下Arthas的用法？

##### **1451：
> 老师，我在使用火焰图的时候，没有在图里找到我关注的那个java方法，我可以确定这个方法是有执行的，我同时在用jmeter发请求，这是什么原因呢，我用的还是arthas提供的demo

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 基于采样技术而非埋点技术，所以在一些场景下，记录不到是正常的

##### **飞：
> 老师，请问“非预期的火焰图的屏顶”是什么意思？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以简单的形象理解为：在层峦叠嶂的山峰中，你认为山峰不够“尖”，那就是平顶

##### chengl：
> 老师好，方案1无入侵的方式， 有没有具体的架构细节和操作流程步骤呢 或者文章，想学习下。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 通用无侵入方案Arthas已经实现，在其官网搜Arthas WebConsole即可

##### **用户5069：
> 老师，您对XPocket这个产品怎么看待呢？

