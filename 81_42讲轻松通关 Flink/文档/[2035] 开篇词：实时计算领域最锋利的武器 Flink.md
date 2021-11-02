<p style="line-height: 1.75em; text-align: justify;"></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">你好，欢迎来到 Flink 专栏，我是王知无，目前在某一线互联网公司从事数据平台架构和研发工作多年，算是整个大数据开发领域的老兵了。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我最早从 Release 版本开始关注 Flink，可以说是国内第一批钻研 Flink 的开发者，后来基于 Flink 开发过实时计算业务应用、实时数据仓库以及监控报警系统，在这个过程中积累了大量宝贵的生产实践经验。</span></p> 
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">面试是开发者永远绕不过去的坎</span></p></h3> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">由于项目需要，我在工作中面试过很多 Flink 开发工程师，并且发现了一些普遍性问题，比如：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">对常用的&nbsp;Flink <strong>核心概念和原理掌握不牢</strong>，一旦参与到实战业务中必将寸步难行，一面直接被刷掉；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">能够通过简历筛选的人基本都有实时流计算开发的经验，可以从容应对典型场景下的问题，但<strong>对于非典型但常见的业务场景问题就会支支吾吾</strong>，无从应答； </span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">有些面试者自称参与过实时计算平台的架构设计、开发、发布和运维等全流程的工作，但稍微追问就会发现他<strong>在项目中的参与度其实很低</strong>，暴露出在上一家公司只是开发团队的一个“小透明”；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们现在招聘其实是偏向招有相关经验并熟悉底层原理的人，曾经有面试者能熟练回答在项目中是如何应用 Flink 的，但是<strong>不知道底层源码级别的实现</strong>。</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">上面列举的这四个问题看似不同，但本质上都是在全方位考察你对技术原理的理解深度，以及在实际工作中解决问题的能力。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">当然还有一类人，他们具备深厚的理论基础和丰富的实战经验，却往往因为缺乏面试经验，依然屡屡与大厂擦肩而过。很多开发者在学习完一个框架后，可以熟练地开发和排查问题，但是在面试的过程中却无法逻辑清晰地表述自己的观点。想象一下，当你在面试中被问到以下三个问题：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Flink 如何实现 Exactly-once 语义？</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Flink 时间类型的分类和各自的实现原理？</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Flink 如何处理数据乱序和延迟？</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">你将如何作答？面试官满意的答案究竟长什么样？上述问题的答案，你都可以在这个专栏中找到。</span></p> 
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">想进大厂，必须掌握 Flink 技术</span></p></h3> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">随着大数据时代的发展、海量数据的实时处理和多样业务的数据计算需求激增，传统的批处理方式和早期的流式处理框架也有自身的局限性，难以在延迟性、吞吐量、容错能力，以及使用便捷性等方面满足业务日益苛刻的要求。在这种形势下，Flink 以其独特的天然<strong>流式计算特性</strong>和更为先进的架构设计，极大地改善了以前的流式处理框架所存在的问题。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">越来越多的国内公司开始用 Flink 来做实时数据处理，其中阿里巴巴率先将 Flink 技术在全集团推广使用，比如&nbsp;Flink SQL 与 Hive 生态的集成、拥抱 AI 等；腾讯、百度、字节跳动、滴滴、华为等众多互联网公司也已经将 Flink 作为未来技术重要的发力点。在未来 3 ~ 5 年，Flink 必将发展成为企业内部主流的数据处理框架，成为开发者进入大厂的“敲门砖”。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><img src="https://s0.lgstatic.com/i/image3/M01/88/E1/Cgq2xl6WhwOACFw5AAMl_djFzLY091.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">反观国外，在 2019 年 Flink 已经成为 <span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">Apache 基金会和 GitHub 社区</span></span><strong><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">最为活跃的项目之一</span></strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">。在全球范围内，越来越多的企业</span>都在迫切地进行技术迭代和更新，无论是更新传统的实时计算业务，还是实时数据仓库的搭建，Flink 都是最佳之选。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">毫无疑问，</span><strong><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(51, 51, 51);">Flink 已经成为大数据开发、有实时数据需求的 Java 后端开发、数据仓库、数据挖掘等岗位必须掌握的技术。</span></strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">目前一名具有 3~5 年经验的 Flink 研发工程师，其薪资普遍在 30K 左右，而如果你是公司大数据实时计算领域的核心开发人员，在大数据实时计算领域有深厚的造诣，那么薪资还会更高。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><img src="https://s0.lgstatic.com/i/image3/M01/02/9C/CgoCgV6WhwOADuDlAAMSWEHrk60986.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">然而从目前的市场状况来看，<strong>熟练掌握 Flink 的开发者仍然供不应求，</strong>大数据领域几乎 100% 的招聘 JD 上都要求开发者掌握 Flink。因此，熟练掌握 Flink 也会为求职中的开发者提供更大的议价空间。</span></p> 
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">你的困惑，我来解答</span></p></h3> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">回顾过往，我在学习 Flink 的过程中也不是一帆风顺的，最初没有任何中文文档，比如在处理消息乱序问题时，在面对 Flink 复杂的窗口设计和水印生成时，不得不在晦涩的英文文档、社区邮件列表和源码中寻找答案。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">目前市面上的 Flink 资源依然较少，而且由于 Flink 更新迭代过快、文档更新不及时，让我们在学习和实践过程中仍然面临诸多难点和各种问题：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">当开发者面对新增 API 的使用时，官网找不到答案；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Flink 的一些概念难以理解，文档又全部是英文的，进一步增加了理解难度；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Flink 在生产实践中也会遇到大量的问题，任何参数和 API 的不正确使用都会导致灾难性后果，虽然其中有些问题只有在大数据量、高并发条件下才会产生，比如数据倾斜、反压、多流 join 等，但是这部分正是我们学习进阶和面试大厂必须掌握的。</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">因此，真正掌握生产环境下的问题处理技能，才能称得上是掌握了 Flink。<strong>在 Flink 实践应用中由于 Flink 社区发展时间较短，版本迭代频繁，很多开发者不得不在摸索中前进，出现问题没有可以借鉴的经验，使得开发者束手无策。</strong>所以我在设计专栏时，把 Flink 相关的基础理论与实战案例相结合，基于 Flink 最新的版本，从基础概念入手，通过大量的实战代码演练，带你进入真实的生产环境，学习如何解决当下企业内部真实面临的生产实践问题，获得大厂一线生产实践的宝贵经验。</span></p> 
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">课程设计</span></p></h3> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">专栏共划分 </span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>5</strong></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"> 部分，合计</span> <span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>42 </strong></span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">篇。</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>入门篇：</strong>以讲解 Flink 的应用场景、编程模型，以及常用 API 的原理和使用为主，借此让你对 Flink 技术有一个更全面的认识。</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>基础篇：</strong>重点讲解 Flink 的核心概念及其原理，比如流批一体、计算资源、State、重启策略、并行度、窗口、时间、水印、CEP 等内容，希望你能对 Flink 设计思想、架构模型有更深刻的认识。这些概念及其原理尤为重要，将是你进入生产环境进行开发和调优的基础，必须掌握牢固。</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>进阶篇：</strong>是这个专栏的核心内容，我将帮助你拓展技术深度，深入到实际的生产环境中，探讨 Flink 的高可用配置，讲解排查反压和数据倾斜、资源配置、监控作业的方法，以及如何使用 Flink 高效地去重和建设实时数据仓库。</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>实战篇：</strong>结合市面上应用较广的一些场景设计了多个实战项目，比如实时数据仓库、实时大屏、错误日志报警等，帮助你掌握 Flink 在不同场景中的使用，打破公司业务场景和应用的局限性。这里也会用到在基础篇和进阶篇中学习的知识，带你巩固所学，达到即学即用的效果。</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><strong>面试篇：</strong>假如你已经掌握了 Flink 的理论知识和实战应用，但还不够，还有最后一个关卡等待你去突破，那就是面试。本模块将从基础、进阶、源码、方案设计上，结合我的实践经历讲解面试中常见的问题，帮助你提前掌握面试的要点，透过题目领会面试技巧，真正成为企业正在寻找的那个人。</span></p></li> 
</ul> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在这个专栏里，我希望将自己多年的大数据开发经验、经历的典型问题，以及遇到的经验教训梳理总结，给你带来一场独家定制的分享。核心目标就是，<strong>帮助你构建一套完整的 Flink 技术知识体系，打牢基础、完成进阶、稳赢实战、通关面试</strong>。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">通过本专栏的学习，你将获得：</span></p> 
<ul> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">原理和实践结合的学习体验，知其然也知其所以然；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">大量的生产实践问题及其解决方案；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">多个实战项目，涵盖了目前各大公司经典的需求和解决方案；</span></p></li> 
 <li><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">面试指南，面试环节是你拿到 Offer 的最终保障。</span></p></li> 
</ul> 
<h3><p style="line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">讲师寄语</span></p></h3> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我曾经就职于多家一二线互联网公司，深刻了解到大数据实时计算领域的应用和未来发展趋势。就在前段时间，我身边很多在大数据领域浸润多年的朋友，薪资已经很高了，还在跟我打听 Flink 的学习资源，要补充学习这块的知识。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们知道，2013 年被称为“大数据元年”，发展到今天领域内的红利渐渐削弱，懂大数据的人越来越多，入坑早的开发者大有人在，而且很多已是大数据领域的资深专家。新人入行如果想实现弯道超车，学习和掌握新技术是加速的唯一捷径。Flink 作为出道仅仅一年的大数据框架，发展速度超乎想象，加之阿里巴巴的推波助澜，已经成为大数据和实时开发领域的开发者必须掌握的框架。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们从熟悉到真正掌握一门框架，这个过程并不是一帆风顺的，但只有在实际生产环境中不断地发现问题、解决问题，个人能力也才会出现质的飞跃，不断成为公司和团队的核心骨干。</span></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><br></p> 
<p style="margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); line-height: 1.75em;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">独行者速，众行者远，我在这里，和很多朋友一起陪伴你的学习和成长，欢迎你在留言区分享你的困惑和成长。</span></p>

---

### 精选评论

##### **先：
> 学习大数据最好是十年前，其次是现在。

##### *柳：
> <span style="font-size: 16.0125px;">1元他不香么？非常期待</span>

##### *刚：
> 一起学习，提升Flink技术水平。

##### **文：
> 开始学习

##### *鑫：
> 来自大数据技术与架构，加油

##### **蜗牛：
> 不错不错，是否有技术讨论群呀？拉一下我呗～😀

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 有的啊，可以关注 拉勾教育 公众号入群哦

##### **威：
> 我是老师的小迷弟，God Of Bigdata

##### **用户5737：
> 加油

##### **也：
> 学习提升一下

##### **飞：
> 有没有学习交流群可以拉一下？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 请在课程页点击“互动”，微信识别二维码，添加运营小姐姐，拉你进群，耐心等待，运营小姐姐加的人很多，会按顺序拉群的

##### **yyan 117：
> 讲的很不错👍

##### **3470：
> 课件哪里下载？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注拉勾教育公众号 咨询小助手获取课件

##### **田：
> 跟着老师混，爱了爱了

##### **营：
> 打卡

##### **东：
> 课程更新频率是多久，有固定更新时间吗😀

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 每周一、周四更细哦~

##### **7651：
> 独行者速，众行者远<div><br></div><div>都不要走一起学习到天亮，请问有没有官方学习群？</div>

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注 拉勾教育 公众号，找小助手入群

##### *强：
> 开篇很赞，一语中的，非常诚恳，期待更新

##### **雄：
> 太棒了，正是我想要学习的课程！

##### **诚：
> 学习Flink，加油。

##### **林：
> 实时计算，需要实时统计赠险产品已领的到最高份数，进行限购，防止超卖，项目组正准备接入大数据实时数据统计呢，Flink正合此景。

##### *鹏：
> 老师好，我想咨询下，没有学习过spark，目前在大数据岗位，做的是数据集市的工作，学习flink会不会很吃力呢，或者有什么建议可以让我学的更容易理解一点！请指点！多谢多谢！！！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 简直易如反掌

