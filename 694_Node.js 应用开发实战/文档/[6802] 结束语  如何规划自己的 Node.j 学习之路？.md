<p data-nodeid="881" class="">到今天为止，你已经学完了本专栏的全部内容，最后我想和你分享一个稍微轻松的话题：如何最小路径学习一门新技术？希望你能有所收获。</p>
<p data-nodeid="882">首先我们要了解清楚什么是“最小路径”，我自己理解是能够从本质出发，剖析出这门技术的所有需要掌握的知识点，然后提炼出核心的技术实现，只针对这部分核心的知识进行深入学习，并扩充自己基础技术框架的技能点，从而达到掌握这门技术的目的的学习路径。</p>
<p data-nodeid="883">我在面试候选人时，经常会问这样一个问题：“<strong data-nodeid="991">如果你要学习一门新的技术，比如 Node.js，你该怎么学？</strong>”</p>
<p data-nodeid="884">大部分的候选人给出的答案都很雷同：</p>
<ol data-nodeid="885">
<li data-nodeid="886">
<p data-nodeid="887">首先了解这是什么技术（查看一些技术类文章博客）；</p>
</li>
<li data-nodeid="888">
<p data-nodeid="889">查看官方文档，走一下简单的 demo ，以及文档中的部分 API ；</p>
</li>
<li data-nodeid="890">
<p data-nodeid="891">接着去实践一个测试项目（也称为踩坑记），熟悉这门技术；</p>
</li>
<li data-nodeid="892">
<p data-nodeid="893">最后去了解深层的源码实现。</p>
</li>
</ol>
<p data-nodeid="894">以上也是我一开始学习 Node.js 的路径，而当我学习了 PHP、Java、React、Flutter 以及 Swift 后，我发现技术都有共通的地方。我希望你意识到这一点，并在面试时，能向面试官展现你的学习技巧。</p>
<p data-nodeid="895">你要先把技术分为三类。</p>
<ul data-nodeid="896">
<li data-nodeid="897">
<p data-nodeid="898">编程语言类： PHP、Java、Go、Kotlin、JavaScript、Dart、Object-C、Swift……</p>
</li>
<li data-nodeid="899">
<p data-nodeid="900">技术框架类：React、Node.js……</p>
</li>
<li data-nodeid="901">
<p data-nodeid="902">技术服务类：Redis、Kafka、MQ……</p>
</li>
</ul>
<p data-nodeid="903">接下来，我会带你了解前两类的学习方法，前两者需要学习的成本和时间都比较多，而技术服务类的一般掌握应用起来周期比较短。</p>
<h3 data-nodeid="904">学习路径</h3>
<p data-nodeid="905">对于语言和技术框架，你要了解这样几个本质的知识点：</p>
<ul data-nodeid="906">
<li data-nodeid="907">
<p data-nodeid="908">编程语言的本质；</p>
</li>
<li data-nodeid="909">
<p data-nodeid="910">前端/后台的本质；</p>
</li>
<li data-nodeid="911">
<p data-nodeid="912">程序员的本质。</p>
</li>
</ul>
<h4 data-nodeid="913">编程语言的本质</h4>
<p data-nodeid="914">你应该知道，编程语言的本质就是把可描述的自然语言转为机器可运算解读的二进制，那么在这个过程中你要对编程语言有足够的认识：</p>
<ul data-nodeid="915">
<li data-nodeid="916">
<p data-nodeid="917">语言转化为可运算的二进制应该掌握哪些基础；</p>
</li>
<li data-nodeid="918">
<p data-nodeid="919">编程语言是如何去执行的；</p>
</li>
<li data-nodeid="920">
<p data-nodeid="921">这门语言的特点和规范是什么；</p>
</li>
</ul>
<p data-nodeid="922">我所提及的基础，就是编程语言的数据类型、流程控制、运算方式、异常处理、并行串行、面向对象以及面向函数等。</p>
<p data-nodeid="923">举个例子，假如你想学 Dart 语言，那么你就要先了解它包含的基础数据类型，基础数据类型的申明，基础类型的应用场景。接下来就是流程控制，流程控制对于各种编程语言来说，都是非常相似的，比如说 if...else 、for...in 、switch……对于这类的知识点尽量学习其与众不同的部分，不用面面俱到，只需要了解其差异点即可。运算方式的话，每种语言也都比较相同；而并行串行，每种语言的处理方式都会不同，你需要深入了解，最后的面向对象、面向函数，每门语言也基本相似。</p>
<p data-nodeid="924">所以，关于“基础”每门语言都是相通的，只要你有编程基础，就可以去学习并且掌握。</p>
<p data-nodeid="925">关于“如何执行”，主要考虑是解释型语言还是脚本类语言？多进程还是单进程？多线程还是单线程、底层的代码机制是如何执行的？</p>
<p data-nodeid="926">举个例子，如果你想学习 Go 语言，那么你应该了解它是多线程，并且其包含了一个协程（ goroutine ），所谓的一个 M 比 N 的模型（GMP模型），那么 M 比 N 应该如何去调度执行呢？通过探究“如何执行”的问题，你就会去深入了解和学习这门技术的深层次知识，而不是掌握其命令行运行方式。</p>
<p data-nodeid="927">关于“语言特点和规范”，每一门语言都不同，比如你熟悉的 JavaScript，它是弱类型的语言，知道这一点后，你就要注意弱类型使用时要对其进行严格的校验，避免异常。语言的规范就是我们的根基，每一种语言的规范都有所不同，不管是变量、常量、函数、类命名还是使用上都存在不同，而这也是学习一门语言必须要掌握的知识。</p>
<p data-nodeid="928">相信你只要掌握了这三个方面，就可以掌握编程语言的技术了。</p>
<h4 data-nodeid="929">前端的本质</h4>
<p data-nodeid="930">我自己经过了长时间的分析和提炼，总结了前端的本质：</p>
<ol data-nodeid="931">
<li data-nodeid="932">
<p data-nodeid="933">视图布局；</p>
</li>
<li data-nodeid="934">
<p data-nodeid="935">视图交互；</p>
</li>
<li data-nodeid="936">
<p data-nodeid="937">数据流转；</p>
</li>
<li data-nodeid="938">
<p data-nodeid="939">用户体验；</p>
</li>
</ol>
<p data-nodeid="940">关于视觉布局，你应该去学习这门技术视图包含哪些基础组件，比如说列表、Form 表单、图片、动画等；另外，布局是用什么方式进行的，比如静态、流式、响应式以及弹性等。以上是前端比较通用的技术，只要你掌握了这些内容，学习其他语言是非常轻松的。</p>
<p data-nodeid="941">而对前端来说，视图交互无非是事件类型（事件生成者）、事件响应（事件消费者）以及事件流（事件传递者）三个事件对象。</p>
<ul data-nodeid="942">
<li data-nodeid="943">
<p data-nodeid="944"><strong data-nodeid="1032">事件类型包含：</strong> 点击事件、滑动事件、拖拽事件、输入事件……而前端这类事件都是共通的，因为人机交互的方式也就这些。</p>
</li>
<li data-nodeid="945">
<p data-nodeid="946"><strong data-nodeid="1037">事件响应包含：</strong> 路由跳转、数据处理转化、数据提交、事件转发……路由跳转对于前端来说是比较核心的部分，但是原理相似，主要是 URL 、Deep link 、Scheme 、Universal link 、 内部的 Action 机制……</p>
</li>
<li data-nodeid="947">
<p data-nodeid="948"><strong data-nodeid="1042">事件转发包含：</strong> 冒泡机制、事件捕获和事件委托，对前端来说都是非常相似的技术点。</p>
</li>
</ul>
<p data-nodeid="949">关于数据流转，会涉及数据的网络模块、传输协议、存储和管理、本地存储技术、数据的动态响应等。</p>
<ul data-nodeid="950">
<li data-nodeid="951">
<p data-nodeid="952"><strong data-nodeid="1048">网络模块：</strong> 每个技术都有该模块，并且都会有一些不同的处理原则，比如说同步返回还是异步获取。</p>
</li>
<li data-nodeid="953">
<p data-nodeid="954"><strong data-nodeid="1053">传输协议：</strong> 无非就是 JSON、XML 和 Protobuff ，当然也存在自定义的消息二进制数据。</p>
</li>
<li data-nodeid="955">
<p data-nodeid="956"><strong data-nodeid="1058">本地存储技术：</strong> 前端的各类技术中都有所不同，也分为私密的和公开的，那么在应用时也应该区分开来。</p>
</li>
<li data-nodeid="957">
<p data-nodeid="958"><strong data-nodeid="1063">数据响应：</strong> 目前就包含我们比较熟悉的 MVVM 的 VM 模块。</p>
</li>
</ul>
<p data-nodeid="959">而前端最重要的目标就是用户体验，所以用户体验是任何前端技术的核心部分，你要考虑“用户体验的指标怎么去衡量？”不过每一种技术衡量的指标都相似，比如首屏渲染时长、FPS、事件响应延时……</p>
<p data-nodeid="960">那怎么去获取这些用户体验的性能指标，并且上报监控呢？这是你需要了解的，比如 Web 有 performance 对象。其次，使用什么工具化方式来调试以及优化用户体验指标的数据，也是你需要了解和掌握的部分，比如前端有 Chrome 、Flutter 有 DevTools 。</p>
<p data-nodeid="961">以上就是前端的本质，掌握这些本质的知识以后，你学习一门新的技术往往是比较容易的，比如说现在都是人操作交互，假设以后靠神经传输交互，那么视图交互还是没有变，变的主要是交互方式。</p>
<h4 data-nodeid="962">程序员的本质</h4>
<p data-nodeid="963">程序员的本质我认为是高质量、高效率、高扩展。我认为，如果你没有追求这三个方面，你还是不够努力。那么为了做好这三方面，你要掌握哪些技术点呢？</p>
<ul data-nodeid="964">
<li data-nodeid="965">
<p data-nodeid="966"><strong data-nodeid="1073">高质量：</strong> 包括单元测试、自动化测试、性能压测工具、性能检测工具、异常检测上报、异常告警、安全扫描、安全保护策略……</p>
</li>
</ul>
<p data-nodeid="967">有了以上知识，你再套用一下新的技术，假设你要学习 iOS 技术开发，那么就应该去学习其单元测试的方法（XCTest）、UI 自动化测试工具（Appium）、性能压测工具（instrument）、性能检测工具（instrument）、异常检测上报和告警（firebase 平台）、安全扫描和安全保护策略（passionfruit 和其他云平台服务）。你也可以把它套用在 Web 技术和 Flutter 上……</p>
<ul data-nodeid="968">
<li data-nodeid="969">
<p data-nodeid="970"><strong data-nodeid="1079">高效率：</strong> 包括工程化（指工程化工具比如 webpack ，包体积优化、打包时间优化方案）；和工具化（比如前端技术中的多国家地区多语言、埋点自动化、IDE……）。</p>
</li>
<li data-nodeid="971">
<p data-nodeid="972"><strong data-nodeid="1084">高扩展：</strong> 包括组件化、框架（生命周期、分层结构、原理实现）、设计模式、架构模式（MVC、MVVM、KVO、MVP）、库包管理……相信你应该熟悉这些基础。</p>
</li>
</ul>
<p data-nodeid="1531">当你学习新技术时，只需要扩充这部分的知识体系就足够了。为了帮助你梳理内容，建立整个框架，我整理了一张思维导图，如图 1 所示。</p>
<p data-nodeid="1532" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/3E/52/CioPOWCY3O2AfqpLAAQW5ovFSIo859.png" alt="图片2.png" data-nodeid="1537"></p>
<div data-nodeid="1533"><p style="text-align:center">图 1 最小路径学习新技术思维导图</p></div>




<h3 data-nodeid="976">专栏体会</h3>
<p data-nodeid="977">之所以分享我的学习路径，是希望你在学习课程时，掌握学习后台编程的本质，在课程设计中，我都是在根据学习路径和你一起学习。以我们本专栏学习的 Node.js 为例，我们首先学习了 Node.js 单线程的事件循环原理，这部分就是编程语言的本质中所需要掌握的部分，然后在后台的本质中，用户体验以及性能是非常关键的点，所以我们用了大量的时间在讲性能的问题和优化策略。</p>
<p data-nodeid="978">再比如程序员的本质，我们学习了框架以及原理，介绍了安全、安全检测和防护方法以及工具化、自动化方案……</p>
<h3 data-nodeid="979">直击本质</h3>
<p data-nodeid="980">上面的学习路径是我近期的一个思考，也是我学习 Flutter 这门技术的一个缩影，而灵感来源于《直击本质》这本书。你在学习一门新的前端或者后台技术时，首先要去探索该技术方向的本质特征，在此基础上思考要掌握哪些基础要素。只要把这些基础技术要点掌握后，我相信当遇到一门新的技术或者新的方向时，只需要在原来的技术面之上，拓展基础技术点就可以了，那样学习速度会比别人快很多。</p>
<p data-nodeid="981">那么假设让你学习 Flutter 这个新的前端技术时，你应该如何去学习掌握好这门技术呢？相信看到这里以后你心中一定有答案了，欢迎分享你的想法。</p>
<p data-nodeid="982">最后，我为你准备了一份结课问卷，希望你对本课程的内容进行评价，以便我及时优化课程内容。</p>
<p data-nodeid="983" class=""><a href="https://wj.qq.com/s2/8431305/6925" data-nodeid="1098">点击链接，即可参与课程评价</a>。</p>

---

### 精选评论


