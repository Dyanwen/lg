<p data-nodeid="733" class="">你好，我是若地。我曾担任美团点评技术专家，是一名高性能组件发烧友，平时专注于基础架构中间件的研发工作，积累了丰富的分布式架构设计和调优经验。</p>
<p data-nodeid="734">我们知道网络层是架构设计中至关重要的环节，但 Java 的网络编程框架有很多（比如 Java NIO、Mina、Grizzy），为什么我这里只推荐 Netty 呢？</p>
<p data-nodeid="2399" class="">因为 Netty 是目前最流行的一款高性能 Java 网络编程框架，它被广泛使用在中间件、直播、社交、游戏等领域。目前，许多知名的开源软件也都将 Netty 用作网络通信的底层框架，如 Dubbo、RocketMQ、Elasticsearch、HBase 等。</p>




<h3 data-nodeid="736">为什么要学习 Netty？</h3>
<p data-nodeid="737">讲到这里，你可能要问了：如果我的工作中涉及网络编程的内容并不多，那我是否还有必要花精力学习 Netty 呢？</p>
<p data-nodeid="738">其实在互联网大厂（阿里、腾讯、美团等）的中高级 Java 开发面试中，经常会问到涉及 Netty 核心技术原理的问题，比如：</p>
<ol data-nodeid="739">
<li data-nodeid="740">
<p data-nodeid="741">Netty 的高性能表现在哪些方面？对你平时的项目开发有何启发？</p>
</li>
<li data-nodeid="742">
<p data-nodeid="743">Netty 中有哪些重要组件，它们之间有什么联系？</p>
</li>
<li data-nodeid="744">
<p data-nodeid="745">Netty 的内存池、对象池是如何设计的？</p>
</li>
<li data-nodeid="746">
<p data-nodeid="747">针对 Netty 你有哪些印象比较深刻的系统调优案例？</p>
</li>
</ol>
<p data-nodeid="748">这些问题看似简单，但如果你对 Netty 掌握不够深入，回答时就很容易“翻车”。我面试过很多求职者，虽然他们都有一定的 Netty 使用经验，但当深入探讨技术细节及如何解决项目中的实际问题时，就会发现大部分人只是简单使用，并没有深入掌握 Netty 的技术原理。<strong data-nodeid="829">如果你可以学好 Netty，掌握底层原理，一定会成为你求职面试的加分项。</strong></p>
<p data-nodeid="749"><strong data-nodeid="833">而且通过 Netty 的学习，还可以锻炼你的编程思维，对 Java 其他的知识体系起到融会贯通的作用。</strong></p>
<p data-nodeid="750">当年我刚踏入工作，领到的第一个任务是数据采集和上报。我尝试了各种解决方案最后都被主管否掉了，他说“不用那么麻烦，直接使用 Netty 就好了”。于是我一边学习一边完成工作，工作之余还会挤出时间研究 Netty 源码。</p>
<p data-nodeid="751">回想起研究源码的那段日子，虽然很辛苦，但仿佛为我<strong data-nodeid="840">打开了一扇 Java 新世界的大门</strong>，当我理解领悟 Netty 的设计原理之后，对 I/O 模型 、内存管理、线程模型、数据结构等当时理解起来有一定难度的知识，仿佛一瞬间“顿悟”了。而且在我日后再去学习 RocketMQ、Nginx、Redis 等优秀框架时，也明显感觉更加便捷、高效了。</p>
<p data-nodeid="752">因此，如果你想提升自己的技术水平并找到一份满意的工作，学习掌握 Netty 就非常重要。事实上，在平时的开发工作中，<strong data-nodeid="845">Netty 的易用性和可靠性也极大程度上降低了开发者的心智负担。</strong></p>
<p data-nodeid="753">我在学生时代，写过不少网络应用，现在看来，非常冗长。当我熟练掌握 Netty 后，一切问题迎刃而解。Netty 对 Java NIO 进行了高级封装，简化了网络应用的开发过程，我们不再需要花费大量精力关注 Selector、SocketChannel、ServerSocketChannel 等繁杂的 API。</p>
<p data-nodeid="754">当我自己写网络应用时，拆包/粘包、数据编解码、TCP 断线重连等一系列问题都需要考虑到，而现在 Netty 给我们提供了现成的解决方案。此外遇到问题还可以在社区讨论，Netty 的迭代周期短修复问题快，其可靠性和健壮性被越来越多的公司所认可和采纳。</p>
<p data-nodeid="755">不客气地说，<strong data-nodeid="853">正是因为有 Netty 的存在，网络编程领域 Java 才得以与 C++ 并肩而立</strong>。</p>
<p data-nodeid="756">由以上几点出发，我想和你一起学习 Netty，希望在工作和求职的过程中能够为你提供帮助，也可以为你打开学习思路。</p>
<h3 data-nodeid="757">学习目标与困难</h3>
<p data-nodeid="758">那么我们该如何学习 Netty 技术呢？作为初学者，你一定会有很多疑问或遇到一些问题：</p>
<ul data-nodeid="759">
<li data-nodeid="760">
<p data-nodeid="761">缺乏网络相关的基础知识，学习 Netty 往往理解不深刻，始终不得其法；</p>
</li>
<li data-nodeid="762">
<p data-nodeid="763">Netty 知识点非常多，网上资源比较零散，社区文档对初学者也不够友好，如何系统化学习 Netty；</p>
</li>
<li data-nodeid="764">
<p data-nodeid="765">看了这么多 Netty 的基础理论，落到项目开发中却依然毫无头绪；</p>
</li>
<li data-nodeid="766">
<p data-nodeid="767">Netty 源码过于复杂，学习无从下手，抓不住重点，最终半途而废；</p>
</li>
<li data-nodeid="768">
<p data-nodeid="769">工作中缺少实践，仅仅学习理论知识很容易就忘记了。</p>
</li>
</ul>
<p data-nodeid="770">在学习的过程中我也遇到了同样的问题，但幸运的是美团的工作经历让我有了很多实践和解决问题的机会。在这期间，我在系统设计方面不断有新的认知。</p>
<p data-nodeid="771">这里我想分享一些我的学习经验，供你一同学习。学习方法不但适合 Netty，也适合其他技术。希望通过这些经验，可以一同进步。</p>
<ul data-nodeid="916">
<li data-nodeid="917">
<p data-nodeid="918">首先，兴趣是最好的老师，工作之余我一定会分配出至少 10% 的时间去思考和学习新的知识，像 Netty 如此优秀的学习资源当然不能放过。</p>
</li>
<li data-nodeid="919">
<p data-nodeid="920" class="">其次，如果你工作中缺乏项目实战，其实也不必过于担心，可以尝试实现一些 MVP 的原型系统，例如 RPC、IM 即时聊天，HTTP 服务器等。不要觉得这是在浪费时间，实践出真知，在学习 Netty 的同时你也会得到很多收获。</p>
</li>
<li data-nodeid="921">
<p data-nodeid="922">再次，在学习源码之前，首先要让自己成为一个熟练工，掌握基本理论。事实上，不论是学习什么框架，我会先尝试挑战自己。我在心中问自己：“我会如何设计它的架构？”然后再去学习相关的博客、源码等资源，思考作者的设计为什么与自己完全不一样？两者设计的差别在哪里？</p>
</li>
<li data-nodeid="923">
<p data-nodeid="924">最终，反复学习也很重要。有时在汲取新知识的时候会对之前的知识点理解产生新的想法，我会带着疑问去把相关的知识重新学习一遍，打破砂锅问到底，经常收获满满。</p>
</li>
</ul>

<h3 data-nodeid="781">Netty 的学习路径</h3>
<p data-nodeid="782">如果现阶段的你：</p>
<ol data-nodeid="783">
<li data-nodeid="784">
<p data-nodeid="785">具备一定的 Java 基础，需要深入学习一款开源框架提升能力和开拓视野；</p>
</li>
<li data-nodeid="786">
<p data-nodeid="787">希望自己在求职面试中增加闪光点，成为精通 Netty 的硬核程序员；</p>
</li>
<li data-nodeid="788">
<p data-nodeid="789">想系统学习 Netty 服务端开发，并希望通过实战来加深理解；</p>
</li>
<li data-nodeid="790">
<p data-nodeid="791">正在从事网络、分布式服务框架等方向的工作，期望自己成为该领域的专家。</p>
</li>
</ol>
<p data-nodeid="792">那么这个课程就是为你量身定做的，课程中我会<strong data-nodeid="887">结合高频的面试题</strong>，从源码出发剖析 Netty 的核心技术原理，同时将这么多年使我受益匪浅的一些<strong data-nodeid="888">编程思想</strong>和<strong data-nodeid="889">实战经验</strong>分享给你，帮助你在工作中学以致用，避免踩坑。</p>
<p data-nodeid="793">在这里我也总结归纳出一份 Netty 核心知识点的思维导图，希望可以帮助你梳理本专栏的整体知识脉络。我会由浅入深地带你建立起完整的 Netty 知识体系，夯实你的 Netty 基础知识、Netty 进阶技能、实战开发经验。</p>
<p data-nodeid="794"><img src="https://s0.lgstatic.com/i/image/M00/60/33/CgqCHl-NAQaABGcDAAZa0pmBs40719.png" alt="image (2).png" data-nodeid="893"></p>
<ul data-nodeid="795">
<li data-nodeid="796">
<p data-nodeid="797"><strong data-nodeid="898">夯实 Netty 基础知识</strong>：第一、二部分介绍 Netty 的全貌，了解 Netty 的发展现状和技术架构，并且逐一讲解了 Netty 的核心组件原理和使用，以及网络通信必不可少的编解码技能，为后面的源码解析和实践环节打下基础。</p>
</li>
<li data-nodeid="798">
<p data-nodeid="799"><strong data-nodeid="903">Netty 进阶技能</strong>：第三部分讲解 Netty 的内存管理，并希望通过对比介绍 Nginx、Redis 两个著名的开源软件，帮你达到举一反三的能力。第四部分结合高频的面试问题，通过多解读剖析 Netty 的核心源码，帮助你快速准确地理解 Netty 高性能的技术原理，对其中的设计思想学以致用。</p>
</li>
<li data-nodeid="800">
<p data-nodeid="801"><strong data-nodeid="908">实战开发经验</strong>：课程最后带你从 0 到 1 打造一个高性能分布式 RPC 框架，并针对 RPC 框架的核心要点，帮助你掌握网络编程的技巧，加深对 Netty 的理解。</p>
</li>
</ul>
<p data-nodeid="802">除了上述内容，你还可以通过本专栏获得一些额外的福利。</p>
<ul data-nodeid="803">
<li data-nodeid="804">
<p data-nodeid="805">万丈高楼平地起，课程会穿插必备的 Linux 网络编程基础知识，助你理解 Netty 时事半功倍。</p>
</li>
<li data-nodeid="806">
<p data-nodeid="807">Netty 源码的调试经验和技巧，从源码中我们可以学习到优秀的设计思想和技巧。</p>
</li>
<li data-nodeid="808">
<p data-nodeid="809">Netty 在实际的项目实践中踩过哪些坑？最佳实践应该是什么？</p>
</li>
<li data-nodeid="810">
<p data-nodeid="811">利用 Netty 如何快速搭建一套高性能的分布式 RPC 框架？我会一步步带你完成这个 MVP 原型。</p>
</li>
<li data-nodeid="812">
<p data-nodeid="813">在技术道路上如何升级打怪？告诉你我是如何学习和打造自己的技术体系的。</p>
</li>
</ul>
<p data-nodeid="814" class="">讲到最后，相信你一定对学习 Netty 满怀激情，那么一起来解锁 Netty 这项技能吧，也欢迎你留言和我一起交流和讨论。希望你能够将 Netty 这门技术融会贯通，让你的开发实践与职业发展走得更加顺利、长远！</p>

---

### 精选评论

##### *勾：
> 通过 Netty 的学习，可以锻炼你的编程思维，对 Java 其他的知识体系起到融会贯通的作用。我们知道，Java 学习不是一朝一夕完成的。Java 是不断发展的，一旦停止 Java 学习，必然意味着技术的后退。想要精炼 Java，为自己加薪嘛？点击链接https://shenceyun.lagou.com/t/Mka ，学习 Java，让你在 Java 精进的道路上永为人前不落后。

##### Certan：
> 终于有这么一门介绍 Netty 的课了！最近还正想着怎么找地方系统学一下。Netty入门容易，想提高太难了。看了下老师给的大纲感觉还可以，mark一下跟进看看。

##### **百代：
> Netty用处挺多的，也是Java的重要组件。课程内容看着还可以，先买了。

##### *然：
> 终于有人来讲Netty了，特别有用一组件，可惜很多人没发现它的价值.....这次要好好学习这个了，根本没替代品！

##### *宁：
> 感觉这门课很实用 希望可以坚持学完

##### **将：
> 紧跟若大佬，拿下Netty这份技能包

##### *帅：
> 此生无悔学Netty

##### **英：
> 二刷

##### **桂：
> Flag：两周学习完，后续持续学习思考。

##### **飞：
> 打卡

##### *明：
> 新年flag, 跟着大佬搞定netty

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 新年新气象，加油学习~

##### **扬：
> 这个专栏是不是市面上第一个netty音频专栏课。。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; Netty 目前课程较少，抓紧机会好好学习哦~

##### **杰：
> 打卡

##### **鑫：
> 向大佬学习。

##### Salvation：
> 打卡😁

##### **2963：
> 准备跟着学习

##### **良：
> 写的不错

##### **森：
> Netty学习目标：当前比较熟悉Netty的使用和常用组件，下一阶段目标为1.掌握Netty重要组件原理：EventLoop、ByteBuf、ChannelPipeline、ChannelHandler2.掌握Netty的设计思想和设计哲学，设计模式，并学以致用，mvp3.Netty的调优调试经验4.侧面掌握Linux网络编程的内容、编程思想与实战方法

##### **航：
> 2020.11.09打卡

##### *琪：
> 老师好，请问下你们是怎么通过netty实现的数据采集和上报的，我们正好也有这个需求，有方案文档吗？谢谢！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题有点广泛，首先你需要学会网络协议怎么设计，然后再学习 Netty 如何传输数据即可，第二章课程我们会讲到。关于文档的话推荐你看下开源组件的协议是如何设计的，例如 Dubbo、RocketMQ

##### *磊：
> 收下这netty这个生日礼物。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 生日快乐~让知识随着年龄一起不断成长！

##### **成：
> 紧跟大佬脚步,通过本次学习，从小白进阶到高手，最后到精通。

##### **仙：
> 打卡，正好在学习Netty

##### **升：
> 打卡

##### **微世：
> 打卡😁😁😁

##### *宇：
> 打卡

