<p>你好，欢迎学习《Java 并发编程核心 78 讲》，我是讲师星星，一线互联网公司资深研发工程师，参与过集团内多个重点项目的设计与开发。</p> 
<h2></h2> 
<h6 style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;"><strong>扎实的理论基础，宝贵的并发实践经验</strong></span></h6> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: center; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOAN9TxAADOl2eK1YA757.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">工作期间，因为业务需要，我所开发和负责的场景大多数都是大流量和高并发的，其中有很多是对&nbsp;Java&nbsp;并发知识的实际应用。学习如逆旅，从小白成长为并发大神</span><span style="color: rgb(63, 63, 63); font-size: 16px; font-family: &quot;Microsoft YaHei&quot;, sans-serif;">，困难重重，既然不能逃避，那么唯有改变对它的态度。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: center; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOABnQDAAIty53kLZs981.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从一开始面对线程池导致的 OOM 问题的不知所措，到后来可以深入剖析 JUC 源码，并精准定位、复现、修复线上的并发问题，再到现在可以应对千万级流量的业务场景，并预判和发现隐藏在其中的线程安全隐患，这期间，我走过一些弯路，踩过一些坑，也积累了很多宝贵的并发经验。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: center; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLOAELhuAACPIXhX2bY626.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">此外，在对并发问题的逐个解决过程中，在系统的设计和实施过程中，我详细研读了大量的国内外经典并发书籍和资料，把涉及的代码一一落实、验证，并应用到业务里，这期间让我逐渐建立起了完善的 Java 并发知识体系。</span></p> 
<h2></h2> 
<h6 style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;"><strong>为什么并发编程这么重要呢</strong></span></h6> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">随着接触和负责的系统越来越复杂，我逐渐发现，无论是对于优秀的系统设计，还是对于程序员的成长提高、职业发展，并发编程都是必须要跨过去的“坎”，而一旦你跨过了这道“坎”，便会豁然开朗，原来一切都如此简单，职业发展也会更上一层楼。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOAEMv7AABnabGYURQ993.png"></span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>并发已经逐渐成为基本技能</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">流量稍大的系统，随着数据和用户量的不断增加，并发量轻松过万，如果不使用并发编程，那么性能很快就会成为瓶颈。而随着近年来服务器 CPU 性能和核心数的不断提高，又给并发编程带来了广阔的施展拳脚的空间。可谓是<strong>有需求，同时又有资源</strong><strong>保障</strong>，兼具<strong>天时地利</strong>。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><br></span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>并发几乎是</strong><strong>&nbsp;</strong><strong>Java</strong><strong>&nbsp;</strong><strong>面试必考的内容</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">而随着互联网进入下半场，好公司对程序员的要求也水涨船高，各大互联网公司的岗位描述中，并发几乎是逃不掉的关键词，我们举几个来自拉勾网的 JD 实例。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOAJbveAAHrokwEb7Y378.png"></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: center;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLOAXz5wAAG5iaGUShs303.png"></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLOALZydAAE1RSJ3cV0452.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: center;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOAU2pxAAGWghflKDM777.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp; &nbsp; &nbsp;&nbsp;&nbsp; &nbsp; &nbsp; &nbsp;</span><br></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLOAOLUXAADh5hjW9Ao521.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em; text-align: center;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLOAe7dXAAGloBkIUlw875.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">你会发现，Java 高级工程师岗位要求中并发编程几乎成为了必须掌握的技能点，而在面经里涉及的并发编程的知识也数不胜数，本专栏各课时涉及的知识点，也正是各大厂 Java 高级工程师面试的高频考题。</span><br></p> 
<h2></h2> 
<h6 style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;">如何学好并发编程</span></h6> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在此邀请你做一个小测试，看看目录里的问题，你能否回答全面？相信你看到问题后大部分会感觉很熟悉，但要组织答案却又模棱两可，不敢太确定，那么接下来就带你了解如何学好 Java 高并发并攻克这些难题。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLSAHP18AACWVfXCugg682.png"></span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>Java</strong><strong>&nbsp;</strong><strong>编程是众多框架的原理和基础</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">无论是 Spring、tomcat 中对线程池的应用、数据库中的乐观锁思想，还是 Log4j2 对阻塞队列的应用等，无不体现着并发编程的思想，并发编程应用广泛，各大框架都和并发编程有着千丝万缕的联系。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">并发编程就像是<strong>地基</strong>，掌握好以后，可以做到<strong>一通百通</strong>。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong><br></strong></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>不过，要想学好并发编程，却不是一件容易的事，你有没有以下的感受？</strong></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>并发的知识太多、太杂了</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: center; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLSABWlnAAAr88J9c9A926.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">常见的并发工具类数不尽数：例如，线程池、各种 Lock、synchronized 关键字、ConcurrentHashMap、CopyOnWriteArrayList、ArrayBlockingQueue、ThreadLocal、原子类、CountDownLatch、Semaphore，等等，而它们的原理又包括 CAS、AQS、Java 内存模型等等。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: center; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLSABkjiAADTiPdaGcM233.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; text-align: justify; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从刚才那一长串的名字中可以看出，并发工具的数量很多，而且功能也不尽相同，不容易完全掌握。确实，并发涉及的知识点太琐碎了，大家或多或少都学习过一些并发的知识，但是总感觉一直学不完，东一榔头西一棒槌，很零散，也不知道尽头在哪里，导致学完以后，真正能记住的内容却很少。而且如果学到并发底层原理，就不只涉及 Java 语言，更涉及 JVM、JMM、操作系统、内存、CPU 指令等，令人一头雾水。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>不容易找到清晰易懂的学习资料</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">在我学习的过程中，我总是有一种感受，那就是较少有资料能够把 Java 并发编程讲得非常清楚，例如我们学习一个工具类，希望了解它的诞生背景、使用场景，用法、注意点，最后理解原理，以及它和其他工具类的联系，这一系列的内容其实都是我们需要掌握的。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">反观现有的网络相关资料，往往水平参差不齐，真伪难辨，而且经常含有错误，如果我们先入为主地接受了错误的观点，那就得不偿失了。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/3E/CgoB5l3DgLSAein_AADNovsebTk325.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我希望本门课程可以把 Java 并发编程的这些复杂、难理解的概念，用通俗易懂、丰富的图示和例子的方式和大家分享出来，不仅知道怎么用，还能知道背后的原理。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">利用<strong>“全局思维+单点突破”</strong>的理念，建立起并发的知识体系，同时又对各种常见的工具类有深刻认识，以后我们的知识就可以从点到线，从线到面，浑然一体。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<h2></h2> 
<h6 style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 18px;"><strong>学习了本门课，你会有以下收获</strong></span></h6> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>你可以建立完整的</strong><strong>&nbsp;</strong><strong>Java</strong><strong>&nbsp;</strong><strong>并发知识网</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">通过这门课程，你可以系统地学习 Java 并发编程知识，而不再是碎片化获取，建立起知识脉络后，每一个工具类在我们心中就不再高高在上，而仅仅是我们并发知识体系中的一块块“拼图”，相信你对并发的理解会更深入一个层次。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="http://s0.lgstatic.com/i/image2/M01/A5/5E/CgotOV3DgLSAGmEWAADo6Lxf6ww652.png"></span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">建立完整的知识网络后，今后即便是遇到新推出的并发工具类，也可以迅速定位到它应处的位置，并且结合已有的知识，很快就能把它掌握。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>你可以掌握常用的并发工具类：</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">课程中包含了实际生产中常用的大多数并发工具类所对应的并发知识，包括线程池、synchronized、Lock 锁，悲观锁和乐观锁、可重入锁、公平锁和非公平锁、读写锁、ConcurrentHashMap、CopyOnWriteArrayList、ThreadLocal、6 种原子类、CAS 原理、线程协作的 CountDownLatch、CyclicBarrier、Semaphore、AQS 框架、Java 内存模型、happens-before 原则、volatile 关键字、线程创建和停止的正确方法、线程的 6 种状态、如何解决死锁等问题。从用法到原理，再到面试常见问题，一次性掌握透彻。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<ul style=""> 
 <li><p style="text-align: justify; line-height: 1.75em;"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>面试中获取</strong><strong>&nbsp;</strong><strong>Offer</strong><strong>&nbsp;</strong><strong>的利</strong><strong>器</strong></span></p></li> 
</ul> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">本课程的各小节，都是从高频常考的面试问题出发，首先给出对应的参考解答，然后引申出背后所关联的知识。不但能够让你回答好面试官的问题，而且还可以在面试问题的基础上，做进一步的升华，让面试官眼前一亮。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">我还会和你分享面试经验和技巧，如何把面试官往我们的思路上“引导”，最终帮助你拿到心仪的Offer，向更高阶的岗位迈进。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">可以说并发编程是成为 Java 高级、资深工程师的必经之路。现在几乎所有的程序都或多或少的需要用到并发和多线程，如果你平时只能接触到 CRUD 的项目，想要进一步提高技术水平；或者是长期一线，只是不断地把业务逻辑“翻译”成代码；想要跳槽加薪，面试却屡屡碰壁，那么学习并发将会帮助你突破“瓶颈”，进阶到下一个层级。</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">希望这个专栏可以让 Java 并发编程这个非常难啃的老大难问题，变得“平易近人”、“通俗易懂”、“一点就通”，希望可以让你体会到“哦，原来如此简单！”的感觉，体会到久违的学习的快乐。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: justify; line-height: 1.75em;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><br></span></p>
<blockquote style="margin-top: 0pt; margin-bottom: 0pt; font-size: 11pt; text-align: justify; line-height: 1.75em; color: rgb(73, 73, 73);">
 本专栏在写作时参考了前辈们的优秀作品，具体的参考点已统一整理至专栏最后一小节的《参考文献》，内有详细说明，在此对前辈们表示敬意和感谢。
</blockquote> 
<p><br style="white-space: normal;"></p> 
<p><br></p>

---

### 精选评论

##### **强：
> 在学校学了java两年了，基本都是些crud操作，一直想进入IO和高并发学习，奈何没有找到一门适合自己的课程，今天看了老师的章节导学以及脉络梳理，这不正是我想要的吗哈哈，开心哟，希望学完有所收获填补上高并发编程的知识空缺，加油！！！

##### *俊：
> 这门课程讲得真好，知识点讲得很有层次感，从浅入深，并且有代码演示。从听课的感觉来讲，我觉得老师非常负责任。知识点方面，很实在，在平时工作中都会遇到。

##### **圳：
> 前一阵子面试多线程被问的云里雾里，感觉自己还需要系统的学习一下，老师讲的比较好。底层原理和实现讲的也通俗易懂。

##### **syan：
> 曾国藩 在《曾国藩家书》中说：“凡人做一事，便须全副精神注在此一事，首尾不懈，不可见异思迁，做这样，想那样，坐这山，望那山。人而无恒，终身一无所成。”

因此，学习在于坚持。目前，我一直在学习拉勾教育的并发编程，老师站在很高的高度上给我们总结，让我构建出了完整的知识体系，收益匪浅。比如：为什么面试官喜欢问hashmap?

又到了面试的高峰期。在面试中，面试官经常喜欢问hashmap的底层原理，甚至可以直接问上半个小时。这是因为，hashmap可以引申出很多问题，可以很好地考察候选人的基础是否扎实。

首先，hashmap会涉及到哈希算法的原理，包括哈希冲突的处理方案，开放寻址法和链表法，哈希函数，冲突解决方案这些内容，就是一个知识领域。

另一方面，hashmap还会涉及到链表和红黑树。这也是两个知识领域，无论是链表的插入删除，还是红黑树的调整，都是可以问的问题。包括哈希表，红黑树和链表，其实代表了O(1)，O(log n)，到O(n)的不同检索能力。在数据结构和检索效率方面，也可以追问许多问题。

并且，hashmap不支持排序，那么如果我们想要支持排序（比如实现一个LRU缓存），应该如何改造？这也是更高阶一些的追问方式。

此外，对于工程实现，hashmap是否是线程安全的，这也是可以考量的一个地方。它能体现候选者是否具有多线程开发的经验。

你会发现，小小的一个hashmap，竟然可以问出这么多问题，它能检测出候选人的基础知识和工程经验。

因此，站在巨人肩膀上学习，就是捷径。

##### **新：
> 拉勾教育在我的学习生涯中给我很大帮助，就拿Java并发面试78讲这门课来说吧，并发编程是一门很啃的骨头，光锁就有自旋锁，偏向锁，轻量级锁，膨胀锁，Lock锁，读写锁，栅栏锁，门栓锁，信号量等等，要想深入挖掘还会牵涉到Hotspot源码，自己学是很耗时的，而且很可能费力不讨好，在课程中，老师详细且正确地辨析了各个锁的概念、用法且不流于表面，对底层机理也做了深入剖析。
在经典的HashMap 的各种并发问题，底层机理都有深入分析，在讲解ConcurrentHashMap时还贴心的讲解了不同jdk版本的差异。对晦涩的Java内存模型在课程中也进行了详细地梳理。
整个课程对Java并发编程的脉络梳理的非常清晰，对我帮助极大。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 非常感谢，我们也会在后续呈现更好的内容

##### **飞：
> 课程很好，有理有据，用事实说话；希望面试的时候可以硬钢面试官，拿到大Offer

##### **志：
> 老师的讲课非常好，本人由于是自学编程，从未涉及多线程，层级多次尝试都没有搞明白，看了老师的课之后茅塞顿开，讲的概念，代码，原理、总结很到位。课程知识很系统，非常值得推荐。

##### *扬：
> 老师讲的真的特别清楚，之前自己学的时候比较散，现在一次性来看这个视频，感觉所有知识点都得到了好的梳理，瞬间觉得神清气爽。

##### *一：
> 课程十分生动有趣，自己之前了解过并发，看书也是看的一知半解，但是看过这门课程后，反过来再来看之前看过的书和理解以前那些一知半解的知识后，一切都是豁然开朗。

##### 魏：
> 我看过葛一鸣的《Java高并发程序设计》、《Java编程思想》以及各个博客，他们讲解的并发内容太官方难懂，我觉得都不及这篇78讲通俗易懂，感觉好的钱太值了，谢谢作者！

##### **文：
> 老师讲的很好很详细，很适合对并发有初步了解的同学进行深入学习，好评

##### **4179：
> 我一直对大流量、高并发的业务场景比较好奇，一直有疑问，身边的人都是用的现成框架，说是框架已经设计好了多线程这块，我还是不明白怎么处理这些业务场景。希望读了这个教程有所收获！坚持✊

##### allen：
> 推荐一下相关的书籍、网站呗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 《Java并发编程实战》

##### **彪：
> 666

##### **义：
> 来二刷

##### *尉：
> 大佬厉害

##### **建：
> 挺全面

##### **鸣：
> 本来想火钳刘明的，看来是得火候刘明了。光看简介和专栏的目录结构就感觉很厉害。毕业一年，自己零零散散的学过多线程和并发知识。谢谢作者给一个更加系统的学习并发知识，希望看完之后，我也能构建自己的并发知识体系。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 小编看好你哦~  加油

##### **航：
> <span style="font-size: 16.7811px;">常见的并发工具类数不尽数：例如，线程池、各种 Lock、synchronized 关键字、ConcurrentHashMap、CopyOnWriteArrayList、ArrayBlockingQueue、ThreadLocal、原子类、CountDownLatch、Semaphore，等等，而它们的原理又包括 CAS、AQS、Java 内存模型等等</span>

##### *俊：
> 开篇，走起

