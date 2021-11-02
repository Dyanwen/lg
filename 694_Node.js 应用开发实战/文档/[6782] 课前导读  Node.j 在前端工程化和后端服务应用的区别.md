<p data-nodeid="4591">在前端工程师眼里，工程化最重要的就是 Webpack 工具，而 Webpack 核心是基于 Node.js 来运行的，当然还有其他场景比如说 SSR 的实现以及前端的一些工具化场景。这些应用最终目标都是为了<strong data-nodeid="4692">提升前端研发效率</strong>或者<strong data-nodeid="4693">保证研发质量</strong>，其实并没有真正地应用到 Node.js 核心特点，而后端服务应用才是真正地应用 Node.js 异步事件驱动的特性，那么本讲就着重来介绍两者存在的差异，并指导你进行一些转型思考。</p>
<p data-nodeid="4592">由于 Node.js 的编程语言就是 JavaScript，因此很多前端同学用起来也是非常顺手，但是顺手和熟练应用区别可太大了。这有段小插曲，有一次我看到一份非常好的简历，精通 React、熟练应用 Node.js，看到这种简历着实让我心情很愉悦，心想终于找到一个对口的人才了。当小伙进来面试后，我问了一个问题，我说：“你主要用 Node.js 做了哪些事情，这些应用中，你觉得哪些场景真正发挥出了 Node.js 的特性”。他最终没有通过我的面试，其主要原因是只用了 Node.js 做一些工具或者简单的服务端应用，并没有真实地了解 Node.js 的特点以及所适用的场景，因此谈不上熟练应用 Node.js。如果你想在简历中带上，熟练应用 Node.js，那你可以带着这些问题来学习下本讲的知识点。</p>
<p data-nodeid="4593">本讲将会从表格 1 的这几个方面来讲解这两者之间的区别。</p>

<p data-nodeid="5964" class=""><img src="https://s0.lgstatic.com/i/image6/M00/13/1D/CioPOWBB0TyASaasAABtZvrLaXk828.png" alt="image.png" data-nodeid="5967"></p>


<h3 data-nodeid="4630">运行环境</h3>
<ul data-nodeid="4631">
<li data-nodeid="4632">
<p data-nodeid="4633">工程化的大部分情况都是基于当前开发环境，运行在本地开发机器上。</p>
</li>
<li data-nodeid="4634">
<p data-nodeid="4635">而后端服务应用一般运行在远程服务器上。</p>
</li>
</ul>
<p data-nodeid="4636">为什么这两者的差异会导致我们理解或者编程方面的区别呢？</p>
<p data-nodeid="4637">这就好比，你是老板，你需要两个人来帮你分别做一件事情，一个就在你旁边，一个需要出差去其他地方。</p>
<p data-nodeid="4638">在你旁边的，可以关注到他的效率、进展、是否情绪有问题、是否可能会离职，甚至知道他到底为你做了什么贡献价值。</p>
<p data-nodeid="4639">那出差的就没有这么清晰了，所以你需要有一些方法和工具来分析这些问题，因此需要知道出差的地方办公环境或者团队协助是怎么样的，是否能够符合该员工的办公要求；有人反馈员工有问题时，我们还需要远程取证，判断这个举报或者反馈是否真实存在；其次你需要使用一些目标和策略来考证出差的员工是否按照你既定目标在办公，他的工作状态和工作效率是否达到了要求，或者是否超出了你的预期范围等等。</p>
<p data-nodeid="4640">讲了上面的这个例子以后，我们再回过头来思考，Node.js 的服务其实也是一样的：</p>
<ul data-nodeid="4641">
<li data-nodeid="4642">
<p data-nodeid="4643">运行在本地的服务，你可以快速地判断定位、分析、解决问题；</p>
</li>
<li data-nodeid="4644">
<p data-nodeid="4645">但是在远程的服务，你需要利用一些工具来分析判断或者监控其运行情况。</p>
</li>
</ul>
<p data-nodeid="4646">那因为环境上的差异，会引发什么不同点呢：</p>
<ul data-nodeid="4647">
<li data-nodeid="4648">
<p data-nodeid="4649">首先我们需要应用工具将服务发布到远程机器上，这里就涉及<strong data-nodeid="4748">devops 工具</strong>；</p>
</li>
<li data-nodeid="4650">
<p data-nodeid="4651">我们需要保证远程服务的安全与稳定，这就涉及一些进程管理工具，例如我们<strong data-nodeid="4754">常见的 PM2</strong>；</p>
</li>
<li data-nodeid="4652">
<p data-nodeid="4653">我们需要判断远程服务运行是否正常，这就涉及远程服务的<strong data-nodeid="4760">监控和告警机制</strong>；</p>
</li>
<li data-nodeid="4654">
<p data-nodeid="4655">遇到运行问题时，我们需要通过远程日志来定位分析问题，这就涉及<strong data-nodeid="4770">日志打印</strong>和<strong data-nodeid="4771">跟踪染色</strong>。</p>
</li>
</ul>
<h3 data-nodeid="4656">受众群体</h3>
<ul data-nodeid="4657">
<li data-nodeid="4658">
<p data-nodeid="4659">前端工程化一般都是服务于开发者，比如我自己在本地应用 Webpack 打包或者将 ES6、ES7 转为 ES5 语法等，都是基于开发者工具，而这部分用户则是我们开发者自己。</p>
</li>
<li data-nodeid="4660">
<p data-nodeid="4661">而后端服务应用则服务于真实的用户群体，为用户提供各种交互体验方面的数据处理等。</p>
</li>
</ul>
<p data-nodeid="4662">因为两者的差异，工程化侧重于为开发者提升<strong data-nodeid="4784">研发效率</strong>或者<strong data-nodeid="4785">研发质量</strong>。</p>
<p data-nodeid="4663">后端服务应用则必须关注服务的<strong data-nodeid="4803">稳定与安全</strong>。因为都是基于用户发送的内容，用户有时候发送一些非法或者违法的内容。其次需要关注<strong data-nodeid="4804">并发性能</strong>，因此必须充分考量服务器所能承载的最大用户并发数，在并发即将达到阈值时，又需要考量平行<strong data-nodeid="4805">扩容方案</strong>。还有就是为了用户体验，需要充分做好服务的<strong data-nodeid="4806">性能优化</strong>，做到极致的接口响应时间。</p>
<h3 data-nodeid="4664">问题调试</h3>
<ul data-nodeid="4665">
<li data-nodeid="4666">
<p data-nodeid="4667">因为前端工程化在本地运行，你可以随意地 console.log 打印日志进行调试，因为这些影响的也只是个人，或者说即使变成通用的工具，打印一些 console.log 也对工具的影响不大。</p>
</li>
<li data-nodeid="4668">
<p data-nodeid="4669">但是在后端服务应用时，你就需要考虑一些方法来进行问题调试和定位策略了。</p>
</li>
</ul>
<p data-nodeid="4670">你需要在每个业务场景中，思考在哪里进行一些关键逻辑或者数据打印日志信息，这里就需要 Node.js 日志服务模块，而这类日志服务又不能影响性能，因此需要考虑一些<strong data-nodeid="4827">高性能日志打印工具</strong>。其次在服务端运行，你可能会遇到诸如<strong data-nodeid="4828">内存泄漏</strong>、<strong data-nodeid="4829">句柄泄漏</strong>或者<strong data-nodeid="4830">进程异常退出</strong>等问题，因此这里就需要这类工具和方法来分析定位现网问题。</p>
<h3 data-nodeid="4671">关注点</h3>
<p data-nodeid="4672">经过上面的各种对比，最后这点已经非常明确了，即两者的关注点不同，从而导致了两者的差异性。这也是需要你在学习 Node.js 的时候进行一些<strong data-nodeid="4837">思维转变</strong>，切莫停留在前端工程化应用的角度去学习了，而应该全面地学习上面总结的知识点。只有上面的知识点掌握了，我们才能在简历上说，掌握 Node.js 的应用。</p>
<p data-nodeid="4673">其次我们也来回答下上面的面试题，如果是我，我就会从本讲的两个方面去回答，一个是前端工程化的应用，另外一个是后端服务应用。前者着重于开发效率的提升和研发质量的保证，后者则是真正发挥出了 Node.js 的异步驱动特性。因为异步驱动特性，在主线程不被 CPU 密集型所影响时，可以真正发挥出 Node.js 高并发特性，可以作为大部分网络 I/O 较高的后端服务。</p>
<h3 data-nodeid="4674">总结</h3>
<p data-nodeid="4675">上面讨论的这些差异点，是我们本专栏所需要重点介绍的知识，也是你从 Node.js 前端工程化应用扩展到系统化应用，所必须要掌握的技能点。学完本讲后，希望你能够从思维中有所转变，认识差异重新出发。</p>
<p data-nodeid="4676">那除了我提到的关于两者的差异点，你觉得它们还有什么差异呢，欢迎在评论区与我分享。</p>
<p data-nodeid="4677">下一讲我们就正式进入专栏内容了，第 01 讲我将为你讲解 Node.js 中最基础也是最核心的部分：事件循环的原理。</p>
<hr data-nodeid="4678">




<p data-nodeid="4572"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="4580"><img src="https://s0.lgstatic.com/i/image6/M00/12/FA/CioPOWBBrAKAAod-AASyC72ZqWw233.png" alt="Drawing 2.png" data-nodeid="4579"></a></p>
<p data-nodeid="4573"><strong data-nodeid="4584">《大前端高薪训练营》</strong></p>
<p data-nodeid="4574" class="">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="4588">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **6400：
> 分析得很具体，作为服务千万用户产品的前端工程师，node.js使用也仅仅停留在2方面，一个是与wwbpack结合做工程化，另一个是开发脚手架提高研发效率。涉及到日志、监控相关的虽然使用，但研究不多。node.js使用不多主要有两个原因：第一是不知道node.js使用好能对业务有什么价值；第二是服务端相关的知识需要花时间梳理，但不知道哪些知识是最重要的，比如多线程多进程，阻塞非阻塞IO，CPU逻辑核数与进程数的关系

##### **凌：
> 看完这篇导读真是觉得改变了一些，目前也在基于egg的框架上写服务端的业务，却从来没了解过node的相关知识和特性，真的仅仅是因为同为js，也只是像用js那样用了。最多也就是调用了框架里封装好的东西，真正对于node的特性一个没用过。希望跟着讲师学完，自己能有进一步的提升😁

##### **生：
> 看完这篇感觉干货满满的~

##### *金：
> 觉得node.js除了前端工程化和后端服务应用外，还有跨平台应用的，这样node.js服务环境和对象就变为在本地服务于用户。

##### **火的鱼：
> node一般不敢轻易用服务端吧，一般都是用java多

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 目前使用java较多，像导读里面讲的，Node.js 也是可行的，我在项目中也是实践过使用 Node.js 的，实际效果并不会比java差，并且对于前端同学来说还是有一定优势。

##### *伟：
> 我想问下老师，deno 和 node 的优缺点

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢引导这方面的讨论，我暂时对 deno 了解不多，光从网上资料给大家解释可能意义不大，还需要进一步去应用，并且对比非农，我尽量去深入研究一下，后续在 GitHub 上或者在专栏的后记中补充这部分的分析结论。

