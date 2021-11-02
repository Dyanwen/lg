<p data-nodeid="3796" class="">近几年，国内外越来越多人开始提倡 DevSecOps 理念，很多企业也逐步从 SDL 切换到 DevSecOps，以便研发出更安全的系统。</p>
<p data-nodeid="3797">此外，DevSecOps 的应用也非常广泛，适用于各种研发领域，并不仅限于本套课程所讲的 Web 领域。所以，本讲就向你介绍下 SDL、DevSecOps 相关理论与实践，并对其进行出对比，方便你能够清楚地理解它们之点的差异和价值。</p>
<h3 data-nodeid="3798">什么是 SDL？</h3>
<p data-nodeid="3799">SDL（安全开发生命周期）是一个满足安全合规要求的同时，兼顾开发成本的软件开发过程。它由微软提出，旨在帮助开发人员构建更加安全的软件。</p>
<p data-nodeid="3800">SDL 的核心理念就是将<strong data-nodeid="3889">安全</strong>集成到软件开发的每一个阶段，从需求（要求）、设计、编码（实施）、测试、发布的每一阶段都应该加入相应的安全工作，以提升软件安全质量。</p>
<p data-nodeid="3801"><img src="https://s0.lgstatic.com/i/image6/M01/04/AC/Cgp9HWAuBCGAH1M5AAFGTV0f1u8829.png" alt="图片1.png" data-nodeid="3892"></p>
<div data-nodeid="3802"><p style="text-align:center">SDL（安全开发生命周期）</p></div>
<p data-nodeid="3803">如上图所示，早期对员工进行的安全培训，就已经是 SDL 工作的一部分了；同时遵循整个研发流程，即使是软件发布后，也应该建立一套完善的安全响应流程，以应对发布后可能出现的安全事件。</p>
<p data-nodeid="3804">不过，SDL 也有不足，那就是它更强调安全人员的工作，而并未关注运维问题。同时，现在企业越来越强调敏捷快速地开发产品，按 SDL 的流程看，是比较难适应这种快速迭代的研发流程的，从而促使了 DevSecOps 的诞生。</p>
<h3 data-nodeid="3805">什么是 DevSecOps？</h3>
<p data-nodeid="3806">提到 DevSecOps，就不得不先说下 DevOps 与开发模式。</p>
<h4 data-nodeid="3807">1. DevOps</h4>
<p data-nodeid="3808">DevOps 是“开发”与“运维”两词的缩写，它是一套最佳实践方法论，旨在软件的开发生命周期中，促进 IT 专业人员（包括产品、开发、测试、运维等工作人员）之间的协作和交流，最终实践<strong data-nodeid="3913">持续集成</strong>（CI，集成各个开发团队成员工作，以及时地发现错误）、<strong data-nodeid="3914">持续部署</strong>（CD，保证快速且经常地发布）、<strong data-nodeid="3915">持续反馈</strong>（收集相关反馈帮助优化产品）的目标。</p>
<p data-nodeid="3809"><img src="https://s0.lgstatic.com/i/image6/M01/04/A9/CioPOWAuBDCAcBMmAAeRQL4-nwo108.png" alt="图片2.png" data-nodeid="3918"></p>
<div data-nodeid="3810"><p style="text-align:center">DevOps 过程</p></div>
<h4 data-nodeid="3811">2. DevOps 与其他开发模式的不同</h4>
<p data-nodeid="3812">开发模式重点介绍下瀑布式开发、敏捷开发与 DevOps 的对比，通过下图就可以很直观地看出三者之间的差异。</p>
<p data-nodeid="3813"><img src="https://s0.lgstatic.com/i/image6/M00/04/AC/Cgp9HWAuBD-AdNggAADeU2LQqws391.png" alt="图片3.png" data-nodeid="3925"></p>
<div data-nodeid="3814"><p style="text-align:center">各种开发模型对比</p></div>
<p data-nodeid="3815">可以看到从瀑布式开发到敏捷开发，再从敏捷开发到 DevOps，可以看到各个阶段的切换速度越来越快，且以前的运维部署工作都是放到最后的。而 DevOps 则结合敏捷开发思想，将部署工作也敏捷起来，更强调自动化工具的实现与应用，以帮助实现软件的快速迭代。</p>
<h4 data-nodeid="3816">3.DevSecOps 强调安全</h4>
<p data-nodeid="3817"><strong data-nodeid="3932">DevSecOps 正是在 DevOps的 CI/CD 过程中嵌入安全工作，整合开发、安全、运维等各项工作</strong>，强调安全是整个 IT 团队（开发、安全、运维等工作人员）的责任，而不仅仅是安全人员的工作，且需要贯穿整个研发生命周期的每一个环节，如下图所示。</p>
<p data-nodeid="3818"><img src="https://s0.lgstatic.com/i/image6/M01/04/AC/Cgp9HWAuBE2AdTREAAUyA7ehkaA210.png" alt="图片4.png" data-nodeid="3935"></p>
<div data-nodeid="3819"><p style="text-align:center">DevSecOps 流程</p></div>
<h3 data-nodeid="3820">SDL 与 DevSecOps 的对比</h3>
<p data-nodeid="3821">SDL 与 DevSecOps 并不冲突，一些安全工作是相同的，<strong data-nodeid="3942">只是 DevSecOps 更进一步强调自动化融入流程，安全责任属于每个人</strong>，自建更适合自己企业的安全文化。</p>
<p data-nodeid="3822">下面我整理出 SDL 与 DevSecOps 的一些对比，以帮助你更好地理解它们。</p>
<p data-nodeid="3823"><img src="https://s0.lgstatic.com/i/image6/M00/04/AC/Cgp9HWAuBHSAHqHOAADU4OCrhvM183.png" alt="图片9.png" data-nodeid="3946"></p>
<h3 data-nodeid="3824">DevSecOps 工具链及其建设实践</h3>
<p data-nodeid="3825">如下图所示，Gartner 曾给出一套 DevSecOps 工具链，从计划、创建，到发布、预防，再到预测、适应，共包括了 10 个环节，接下来我们来看看每个环节中的实践建议。</p>
<p data-nodeid="3826"><img src="https://s0.lgstatic.com/i/image6/M01/04/AC/Cgp9HWAuBIWAHtwaAAUVTKLJymM985.png" alt="图片5.png" data-nodeid="3951"></p>
<div data-nodeid="3827"><p style="text-align:center">DevSecOps 工具链</p></div>
<h4 data-nodeid="3828">1.计划（Plan）</h4>
<ul data-nodeid="3829">
<li data-nodeid="3830">
<p data-nodeid="3831">在研发的早期，安全人员可以先给相关人员做<strong data-nodeid="3958">安全培训</strong>，开设一些安全相关课程，在公司内部不定期地开展培训；</p>
</li>
<li data-nodeid="3832">
<p data-nodeid="3833">同时安全人员可制定相应的公司级<strong data-nodeid="3964">安全规范</strong>，包括各种语言的代码安全规范、服务器安全加固规范等等，以引导相关人员在将需求设计阶段就将安全考虑在内，并在后续的研发和运维中起参考作用，可以有效地避免一些常见的安全问题；</p>
</li>
<li data-nodeid="3834">
<p data-nodeid="3835">同时，可邀请安全团队对需求做<strong data-nodeid="3970">安全评估</strong>，包括威胁建模等相关工作，以尽早地发现安全问题，避免开发后再修改，浪费开发资源。</p>
</li>
</ul>
<h4 data-nodeid="3836">2.创建（Create）</h4>
<p data-nodeid="3837">在前面的漏洞攻防部分，修复漏洞时曾介绍过一些<strong data-nodeid="3977">安全开发库</strong>，这些也是此阶段的应用实践部分。</p>
<p data-nodeid="3838">还有一些现在流行的 IDE 集成开发环境中的代码<strong data-nodeid="3987">安全检测插件。</strong> 如下图所示，比如最近陌陌开源的 Java 代码安全审计插件 <a href="https://github.com/momosecurity/momo-code-sec-inspector-java" data-nodeid="3985">Momo Code Sec Inspector</a>，用于检测 Java 代码漏洞，并提供一些常见漏洞的修复代码自动生成功能；除了自动化，也可以在企业内部推广代码评审，通过评审后再合并代码。</p>
<p data-nodeid="3839"><img src="https://s0.lgstatic.com/i/image6/M01/04/A9/CioPOWAuBQ6AH64LAAWKuEUqGec581.png" alt="图片6.png" data-nodeid="3990"></p>
<div data-nodeid="3840"><p style="text-align:center">Momo Code Sec Inspector（Java）</p></div>
<h4 data-nodeid="3841">3.验证（Verify）</h4>
<p data-nodeid="3842">验证测试是安全检测中不可缺少的部分，目前已经衍生出 SAST（静态应用程序安全测试）、DAST（动态应用程序安全测试）、IAST（交互式应用程序安全测试）、SCA（软件成分分析）等安全测试技术。</p>
<blockquote data-nodeid="3843">
<p data-nodeid="3844">SAST、DAST 和 IAST 在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=602#/detail/pc?id=6281" data-nodeid="3998">《08 | SQL注入：漏洞的检测与防御》</a>中已经介绍过，此处不再赘述。</p>
</blockquote>
<p data-nodeid="3845">这里聊下 SCA，它主要用于被利用的各类组件、库代码的安全问题，如果是一些存在已知漏洞的代码被引用，就会影响产品的正常运行。所以在使用前，需要对其进行检测，有时这部分检测也会放在 SAST 中，因为它也涉及静态代码分析。</p>
<h4 data-nodeid="3846">4.预发布（Preprod）</h4>
<p data-nodeid="3847">在前面的验证环节中使用的方法也会应用在预发布中，比如将 SAST、DAST、IAST、SCA 等技术应用到集成测试中；同时也会应用混沌工程去人为制造各种故障，以验证系统在应对故障的情况下，能否稳定持续地提供正常服务；以及使用 Fuzzing 模糊测试技术去构造一些随机数据，发送给程序进行解析处理，通过检测是否触发异常（比如内存崩溃）来判断是否存在安全漏洞。</p>
<p data-nodeid="3848">一些加固壳等安全加固方案也是提高代码安全的一种方式，提高逆向破解成本，在一定程度上也能防止遭受安全攻击。</p>
<h4 data-nodeid="3849">5.发布（Release）</h4>
<p data-nodeid="3850">此阶段主要是应用软件签名来识别软件是由哪家企业开发的，同时区别一些重打包的篡改软件，这能在一定程度上区分软件的可靠性。</p>
<h4 data-nodeid="3851">6.预防（Prevent）</h4>
<p data-nodeid="3852">采用签名验证、完整性检查等手段来验证应用和数据是否被篡改过，防止由于判断失误从而做出不符合预期的行为；同时采取“纵深防御”的思路，设置多层安全措施和机制，防止单层防御被直接突破，可有效提高攻击成本，这是安全建设工作中常用的一种手段。</p>
<h4 data-nodeid="3853">7.检测（Detect）</h4>
<p data-nodeid="3854">运行时应用自我保护（RASP，Runtime ApplicationSelf-Protection）在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=602#/detail/pc?id=6281" data-nodeid="4014">《08 | SQL注入：漏洞的检测与防御》</a>中介绍过，它通过实时监控、检测与阻断安全攻击，使得应用程序具备自我保护能力。</p>
<blockquote data-nodeid="3855">
<p data-nodeid="3856">国内比较知名的开源项目就是百度的 OpenRASP。</p>
</blockquote>
<p data-nodeid="3857">Gartner 还创建一个新词叫 UEBA（User and Entity Behavior Analytics，用户和实体行为分析），即通过对用户和系统实体在数据层面的异常行为，利用机器学习的方法来发现网络安全、IT办公安全、内外部的业务安全等风险，如数据泄露、入侵、内部滥用等的安全问题。像一些业务安全中常应用到的风险控制策略，也正属于此。</p>
<p data-nodeid="3858">产品上线后，对其进行漏洞扫描，所以一些 SAST、DAST、IAST 技术又可以派上用场了，包括 Web 漏洞扫描器、移动应用漏洞扫描系统、主机入侵检测系统、流量安全分析系统等工具都属于此阶段。</p>
<p data-nodeid="3859">国内流行的红蓝军对抗的演习、渗透测试也是对系统进行有效安全检测的方式，能够避免闭门造车，以及其他过于主观的判断，只有经得起安全考验的系统才是安全的系统。</p>
<h4 data-nodeid="3860">8.响应（Respond）</h4>
<p data-nodeid="3861">RASP 既属于检测环节，也属于响应环节，因为它既有检测能力，也有阻断能力。</p>
<p data-nodeid="3862">**安全编排自动化响应（SOAR）**是 Gartner 提出的概念，用于帮助快速实现安全应急响应工作，通过采集各种检测数据，对其进行分析，然后自动化定义、排序和驱动执行应对安全事件的响应工作。</p>
<p data-nodeid="3863">如下图所示，开源的 SOAR 有<a href="https://github.com/zbnio/zbn" data-nodeid="4031">织布鸟</a>、<a href="https://github.com/nsacyber/WALKOFF" data-nodeid="4035">Walkoff</a>。</p>
<p data-nodeid="3864"><img src="https://s0.lgstatic.com/i/image6/M00/04/A9/CioPOWAuBKWAMCPPAAEy9c7nh_Y846.png" alt="图片7.png" data-nodeid="4039"></p>
<div data-nodeid="3865"><p style="text-align:center">织布鸟 SOAR 系统</p></div>
<p data-nodeid="3866"><img src="https://s0.lgstatic.com/i/image6/M00/04/A9/CioPOWAuBLKAMaRdAAFXXPl7Ogg107.png" alt="图片8.png" data-nodeid="4042"></p>
<div data-nodeid="3867"><p style="text-align:center">Walkoff SOAR</p></div>
<p data-nodeid="3868">WAF、DDOS 防御系统等传统的安全防御系统在对一些恶意请求时，能够做出拦截阻断的响应，发挥安全防御作用。</p>
<p data-nodeid="3869">这几年国内流行建立<strong data-nodeid="4049">SRC 应急响应平台</strong>，通过奖励机制去收集产品漏洞，帮助企业完善自身的检测与防御系统，同时提高产品的安全性。这种奖励外部的响应方法非常有效，企业建议专门的应急响应团队也有助于第一时间响应安全事件，降低或消除事件带来的负面影响。</p>
<h4 data-nodeid="3870">9.预测（Predict）</h4>
<p data-nodeid="3871">通过监控外部情报，收集与整理出相关安全威胁信息，帮助内部一些检测与防御系统增加和优化规则。比如外部曝光一款主流应用的高危漏洞，我们就可能及时感知到，并进行漏洞分析，从而添加相应的检测与拦截规则，去提前做好防御工作。</p>
<h4 data-nodeid="3872">10.适应（Adapt）</h4>
<p data-nodeid="3873">基于前面的威胁情报、外部报告漏洞、红蓝军对抗等诸多手段去发现内部安全与检测系统的不足，不断地复盘优化检测与防御规则，优化应急响应流程，从而提升自身安全风险的管控能力。</p>
<h3 data-nodeid="3874">总结</h3>
<p data-nodeid="3875">本节课重点介绍 SDL 与 DevSecOps 的相关概念与技术，并做了对比，然后详细介绍了 DevSecOps 各个环节所涉及的相关技术和工具。</p>
<p data-nodeid="3876">综上可以看出，想建立一套完整的 DevSecOps 实践流程，其所涉及的工作范围、流程、工具都非常繁多，非一人之力所能完成，甚至不是单纯一两个小团队能够完成的，本身它就是提倡多工种协同建设安全工作，而不是安全仅由安全人员来背锅。</p>
<p data-nodeid="5408">只有不断地实践、探索、优化 DevSecOps 模式，不断地改进相关工具链，不断地提供更加自动化、更加便捷的安全能力，才能够提高整体的安全能力，做好安全风险管控工作。</p>
<p data-nodeid="5409"><img src="https://s0.lgstatic.com/i/image6/M01/04/AC/CioPOWAuCxOAcc6YAAVgr8YKbVY942.png" alt="2021218-143636.png" data-nodeid="5413"></p>



<hr data-nodeid="3878">
<p data-nodeid="4063"><a href="https://wj.qq.com/s2/8059116/3881/" data-nodeid="4066">课程评价入口，挑选 5 名小伙伴赠送小礼品～</a></p>

---

### 精选评论


