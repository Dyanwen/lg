<p data-nodeid="66220" class=""><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">你好，我是邴越，在一线互联网公司从事分布式开发工作多年，一直关注分布式理论和新技术的发展。</span></span></span></p>
<p data-nodeid="66221"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">互联网发展到今天，用户数量越来越多，产生的数据规模也越来越大，应用系统必须支持高并发访问和海量数据处理的需求。</span></span></span></p>
<p data-nodeid="66222"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">对比集中式架构，分布式系统由于具有可扩展性，可以动态扩展服务和存储节点，使用廉价的机器构建高性能的服务，更适合如今的互联网业务。分布式系统技术已经成为微服务架构、大数据、云计算等技术领域的基石，在电商、互联网金融、支付等众多业务中，都离不开分布式技术的有效运用。</span></span></span></p>
<p data-nodeid="66223"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">掌握分布式技能的后端工程师越来越抢手，不止业务部门、中间件和基础架构等部门也在大规模抢人。分布式技术的应用越来越广泛，各大公司的相关岗位要求也越来越高，然而在</span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">面试和工作中，我们却看到各种各样的问题：</span></span></span></p>
<ul data-nodeid="66224">
<li data-nodeid="66225">
<p data-nodeid="66226"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">面试时，可以回答概念性的问题，但问到实质性问题时就懵了，由于缺少相关经验而卡住；</span></span></span></p>
</li>
<li data-nodeid="66227">
<p data-nodeid="66228"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">工作中对常用分布式技术的原理一知半解，在典型场景下可以应付，但稍微变更业务场景或业务目标后，就开始毫无头绪；</span></span></span></p>
</li>
<li data-nodeid="66229">
<p data-nodeid="66230"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">系统设计中，没有全面平衡各个设计点，关注了收益，却没考虑到风险，比如增加了缓存，却带来了数据不一致，增加了消息队列，却因为不合理的重试导致服务异常。</span></span></span></p>
</li>
</ul>
<p data-nodeid="66231"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">总结来说，这往往是从业者没有在实际的分布式业务场景中实践过，或者对分步式技术缺乏体系化的认知，或者对一些原理和底层的内容未曾深入研究，导致可以解决常见问题，而没有系统化的解决思路。</span></span></span></p>
<p data-nodeid="66232"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">因此，<strong data-nodeid="66378">我梳理了一套分布式技术的方法论，希望可以帮助你快速而体系化地补齐分布式知识</strong>。此外，一路走来，我在分布式系统设计中踩过的坑，在开发实践中看到和经历过的一些典型问题，也将在这里一并分享给你，希望能够帮到更多开发者，并减轻你学习分布式的畏难心理。</span></span></span></p>
<h3 data-nodeid="66233"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式是工程师进阶的必经之路</span></span></span></h3>
<p data-nodeid="66234"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">经常听到一些开发人员，在工作之余感叹自己职业发展的困惑与焦虑，比如每天写业务代码，如何摆脱 CRUD Boy 的标签，去提升技术能力？一直在传统企业工作，怎么才能加入 BAT 等大公司？所有的机遇都是在充分准备后才能获得的，这些问题的关键，就是你在技术上的持续精进。</span></span></span></p>
<p data-nodeid="66235"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><img src="https://s0.lgstatic.com/i/image3/M01/07/63/Ciqah16ERqmAE-qtAAJ5c5hckiA055.png" alt="" data-nodeid="66397"></span></span></span></p>
<p data-nodeid="71236" class="te-preview-highlight"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="71256">如果想在技术线上深耕和谋求发展，成为高级工程师、资深工程师或者架构师，</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="71257">掌握分布式系统知识已经成为了必要的一环</strong>。不管是目前流行的 SOA 架构，还是蓬勃发展的微服务和 Serverless 架构，都是在分布式的基础上构建的，业务开发中的框架选型、注册中心，以及服务拆分之后面临的分布式事务问题、分布式锁，也都是分布式系统所关注的。</span></span></span></p>






<h3 data-nodeid="66237"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">想要高薪 Offer，必须掌握分布式</span></span></span></h3>
<p data-nodeid="66238"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">要想进入大公司并拿到高薪 Offer，分布式技术也是一个很好的敲门砖。大型互联网公司每天都要面对海量的业务请求，处理各种复杂的系统问题是工作常态，所以需要应聘人员掌握常用的分布式技术，并在面试过程中重点考察你对分布式系统的理解和经验水平。</span></span></span></p>
<p data-nodeid="66239"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">针对</span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66464">高级岗位</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">，除了掌握在分布式环境下进行开发的能力，你还需要了解其中的原理、机制，以便能够快速定位线上问题；而对于<strong data-nodeid="66465">架构师</strong>来说，你还需要具备独立设计分布式系统的能力，这就需要了解高并发、高可用的相关知识了。</span></span></span></p>
<p data-nodeid="66240"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66484">在</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66485">拉勾网上搜索后端工程师的招聘岗位，可以看到很多岗位都要求掌握缓存、分布式服务、消息队列等分布式组件应用，部分岗位还要求在高并发等分布式设计方向有一定的积累。</strong></span></span></span></p>
<p data-nodeid="66241"><img src="https://s0.lgstatic.com/i/image3/M01/08/6A/Ciqah16FzVKAUHomABFJwSsmtFg192.png" alt="" data-nodeid="66487"></p>
<p data-nodeid="66242"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">结合拉勾对海量招聘启事的大数据分析，我们也总结出了后端开发者在面试中要求</span></span></span><strong data-nodeid="66511"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">掌握的分布式技能点</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">，同时也把它们融入到了课程设计中：</span></span></span></p>
<ul data-nodeid="66243">
<li data-nodeid="66244">
<p data-nodeid="66245"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式系统理论和设计；</span></span></span></p>
</li>
<li data-nodeid="66246">
<p data-nodeid="66247"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式事务和一致性；</span></span></span></p>
</li>
<li data-nodeid="66248">
<p data-nodeid="66249"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式服务及微服务架构；</span></span></span></p>
</li>
<li data-nodeid="66250">
<p data-nodeid="66251"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式缓存和常见 NoSQL 应用；</span></span></span></p>
</li>
<li data-nodeid="66252">
<p data-nodeid="66253"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式下数据库的拆分，比如读写分离、分库分表；</span></span></span></p>
</li>
<li data-nodeid="66254">
<p data-nodeid="66255"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">消息中间件的应用，常见组件的选型；</span></span></span></p>
</li>
<li data-nodeid="66256">
<p data-nodeid="66257"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">合理应用分布式技术，实现系统的高可用。</span></span></span></p>
</li>
</ul>
<p data-nodeid="66258"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><img src="https://s0.lgstatic.com/i/image3/M01/80/79/Cgq2xl6ERqmAEq1kAAENcRtXEvU094.png" alt="" data-nodeid="66565"></span></span></span></p>
<h3 data-nodeid="66259"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">难点不难，给你学得会的分布式课程</span></span></span></h3>
<p data-nodeid="66260"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式系统在工作和面试中如此重要，但是掌握起来并不容易。</span></span></span></p>
<ul data-nodeid="66261">
<li data-nodeid="66262">
<p data-nodeid="66263"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66607">理论众多、难以入手。</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式系统不仅涉及一致性、事务等众多的理论知识，还包括非常多的复杂算法，比如 Paxos 和 Zab 算法，如果<strong data-nodeid="66608">没有一个明确的抓手</strong>，<strong data-nodeid="66609">学习起来会很吃力</strong>。</span></span></span></p>
</li>
<li data-nodeid="66264">
<p data-nodeid="66265"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66630">领域庞杂、关联技术栈多。</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式系统涉及很多领域，比如 RPC 服务调用、分库分表，这些不同的领域需要了解和掌握不同的技术栈。因此我的建议是，要想快速提升分布式技术能力，那么<strong data-nodeid="66631">需要明确哪些才是你日常工作中最迫切需要的</strong>，从实践中开始体验和学习，积累经验。要知道，分布式不是一堆理论的堆砌，而是和日常开发息息相关。</span></span></span></p>
</li>
<li data-nodeid="66266">
<p data-nodeid="66267"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66661">工作特点，接触不到分布式</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66662">。</strong></span></span></span><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">鉴于现在一些软件开发公司，或者传统公司的 IT 部门，还在使用集中式系统架构，所以部分开发者平时在工作中很少接触分布式系统，因此，我在这个课程中，将会侧重<strong data-nodeid="66663">讲解很多实际场景的实践内容</strong>，以帮助你更有效地掌握分布式。</span></span></span></p>
</li>
</ul>
<p data-nodeid="66268"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">工作多年，我从一个初入行的新人，一步步晋升一线互联网公司的核心业务负责人，我深知分布式知识的重要性和学习痛点，为了让你在短时间内能够快速掌握分布式知识，我对这门课程进行了精心设计。</span></span></span></p>
<p data-nodeid="66269"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66680">（1）知识体系化，快速学习</strong></span></span></span></p>
<p data-nodeid="66270"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">碎片化知识很难有效学习，体系化的学习才是重点。分布式系统知识足够庞杂，本课程从理论开始，一步一步落地到实践中，帮助你快速构建知识框架，让你对分布式技术有个总体的认知。</span></span></span></p>
<p data-nodeid="66271"><img src="https://s0.lgstatic.com/i/image3/M01/07/63/Ciqah16ERqmAGJjpAACXHV15Oyg347.png" alt="" data-nodeid="66689"></p>
<p data-nodeid="66272"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66699">（2）选取最常用的知识点</strong></span></span></span></p>
<p data-nodeid="66273"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式系统博大精深，但并不是每个人都在做基础架构研发，也不是每一项技术都能直接落地，因而本课程选取了在工程开发中最常用的技术栈，比如在分布式服务模块中选取了网关、注册中心、容器化等内容来讲解，在数据库模块中选择了读写分离、分库分表等内容，这些都是在开发中打交道最多的知识点。</span></span></span></p>
<p data-nodeid="66274"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66716">（3）拒绝空谈理论，结合实际业务场景</strong></span></span></span></p>
<p data-nodeid="66275"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">技术是为业务服务的，再高深的技术都要落地，我们的课程内容不是干巴巴的讲理论，而是结合了实际业务场景，带着问题去讲解，让你能够在实际的场景中理解并应用，达到事半功倍的效果。</span></span></span></p>
<p data-nodeid="66276"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px"><strong data-nodeid="66733">（4）面试真题解析，帮你赢取高薪 Offer</strong></span></span></span></p>
<p data-nodeid="66277"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">为了帮助你更好地准备面试，每个模块后面都附上了一个“<strong data-nodeid="66745">加餐</strong>”内容，并梳理出了面试中经常出现的考点，以及高频面试真题。虽然是加餐，但是内容绝对有料。</span></span></span></p>
<p data-nodeid="66278"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">当然，快速通关面试只是我们的目标之一，我更希望你在这个课程中，真正学有所得，将知识和经验融入到个人能力中，做一些长期主义的事情。</span></span></span></p>
<h3 data-nodeid="66279"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">课程设计</span></span></span></h3>
<p data-nodeid="66280"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">本课程分为 7 个模块，共 45 讲。我将从实际工作和面试出发，从分布式理论开始带你建立知识框架，然后逐个攻破分布式技术的各个核心技术领域。为了让你更清晰地了解本课程中的所有知识点，我还准备了一份思维导图：</span></span></span></p>
<p data-nodeid="66281"><img src="https://s0.lgstatic.com/i/image3/M01/80/79/Cgq2xl6ERqmAdmMXAAMdZN_Jn7I815.png" alt="" data-nodeid="66768"></p>
<ul data-nodeid="66282">
<li data-nodeid="66283">
<p data-nodeid="66284"><strong data-nodeid="66785"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式基础：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">扎实的理论是进一步学习分布式知识的钥匙，这一模块将详解分布式的概念，包括 CAP 和 Base 理论、各种数据一致性模型，以及两阶段和三阶段提交协议等。</span></span></span></p>
</li>
<li data-nodeid="66285">
<p data-nodeid="66286"><strong data-nodeid="66802"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式事务：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">在电商、金融等业务中都涉及资金往来，事务非常重要，那么分布式事务如何解决、分布式锁如何实现、……，这一模块将会解答。</span></span></span></p>
</li>
<li data-nodeid="66287">
<p data-nodeid="66288"><strong data-nodeid="66819"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式服务：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式服务是微服务架构的必要条件，这一模块将讲解如何解决服务拆分后的一系列问题，比如 RPC、网关、注册中心等。</span></span></span></p>
</li>
<li data-nodeid="66289">
<p data-nodeid="66290"><strong data-nodeid="66836"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式存储：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">系统架构拆分以后，存储层面的拆分同样重要，数据库层涉及读写分离、分库分表等，这一模块我们来一起来探究这些技术的原理，以及如何在业务中落地。</span></span></span></p>
</li>
<li data-nodeid="66291">
<p data-nodeid="66292"><strong data-nodeid="66853"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">消息队列：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">消息中间件是分布式系统架构的整合剂，这一模块将分享消息队列使用的常见问题，比如重复消费、消息时序等。</span></span></span></p>
</li>
<li data-nodeid="66293">
<p data-nodeid="66294"><strong data-nodeid="66870"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式缓存：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">&nbsp;缓存的高性能在分布式系统中发挥了更加重要的作用，那么分布式缓存有哪些分类，以及有哪些经典问题，这一模块我们来一起探究。</span></span></span></p>
</li>
<li data-nodeid="66295">
<p data-nodeid="66296"><strong data-nodeid="66887"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">分布式高可用：</span></span></span></strong><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">高可用是工程师始终追求的目标，最后这个模块，我将会为你分享在分布式系统中如何保障系统可用性，如何做好系统监控和限流降级。</span></span></span></p>
</li>
</ul>
<h3 data-nodeid="66297"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">写在最后</span></span></span></h3>
<p data-nodeid="66298"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">这个课程，我从面试出发，利用贴近工作实战的内容来为你梳理知识体系，希望无论是对分布式技术感兴趣，还是正在准备面试的工程师，都能够轻松掌握分布式技术。专栏中贴近实战经验和方法论，也一定会让从事分布式开发的你，找到答案或得到启发。当然，如果你是即将面临求职季的学生，如果能了解一些分布式知识，相信你一定能在校园招聘中得到面试官的更多青睐。</span></span></span></p>
<p data-nodeid="66299"><span style="color:#3f3f3f"><span class="font" style="font-family:微软雅黑, &quot;Microsoft YaHei&quot;"><span class="size" style="font-size:16px">对于用户来说，学习专栏是自我提升的方式；对于作者来说，打磨一个好的专栏，高质量地输出是另一种方式的提高。希望我们在这个课程结束时，都能给自己一份满意的答卷。</span></span></span></p>
<hr data-nodeid="66300">
<p data-nodeid="66301"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="66913"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="66912"></a></p>
<p data-nodeid="66302"><strong data-nodeid="66917">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="66303" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="66921">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **领：
> 一直感觉分布式离自己太过遥远，里面的概念以及原理太过复杂。从今天开始，跟着老师，拿下分布式这个知识点

##### **用户7409：
> <div>作为一名研究生，跟着导师研究区块链知识，从而了解到分布式等相关知识。从Paxos、PBFT等经典算法到PoW等区块链共识算法，一直都是处于理论研究阶段，苦于没有很好的实践方法。看到这个专栏，立马选择报名，希望能真正学会更多的分布式技术。</div>

##### Kee：
> 分布式：基础，事务，服务，存储，缓存，高可用，消息队列！ 我来了！加油💪

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油呀

##### **亮：
> 我要进军bat，加油！不负自己

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油呀！

##### **安：
> 不断学习

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 给你点赞！

##### **慧：
> 期待

##### *铮：
> 大数据也是必备分布式系统性知识。学得越多，理解的越透彻

##### **天：
> 打卡

##### **3128：
> 挺好的

##### **东：
> 课程中代码实现是用什么语言写的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你好，是用 Java 语言来写哦~

##### *星：
> 差不多东西都用过，也掌握了很多，但是还是想再看看，巩固一下

##### **传：
> 可否把思维导图发下，没法下载，谢谢

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注拉勾教育公众号 咨询小助手获取

##### **6112：
> 有学习交流的群吗

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 关注 拉勾教育 公众号，找小助手入群哦

##### **书的皮卡丘：
> 冲冲冲

##### **周：
> 赞一个

##### **荞：
> 啥时候能更新

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 每周二、周四更新

##### tomgs：
> 期待中

##### *帅：
> 期待

##### **东：
> 有建群吗？老师

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 可以关注 拉勾教育 公众号咨询小助手加入学习群

