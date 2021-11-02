<p data-nodeid="44086" class="">DevOps 落地实践这个专栏到本课时就结束了。之前的内容介绍了DevOps软件开发的最佳实践和度量指标，涵盖了软件开发全生命周期每个阶段中涉及的关键部分。希望到目前为止，这门课能够让你从整体上认识 DevOps，对精益、敏捷、DevOps 有了更深的理解。我们知道了 DevOps 是从哪里来，也要知道 DevOps 要到哪里去。DevOps 的下一站是哪里呢？让我来介绍一下。</p>
<h3 data-nodeid="44087">DevOps 将无处不在</h3>
<p data-nodeid="44088">现在，DevOps 的发展已经进入了第二个十年，正如 DevOps 之父 Patrick Debois 在 DevOps 十周年的演讲主题“<strong data-nodeid="44176">Dev{*}Ops</strong>”中说的那样，如今 DevOps 已经扩展到开发和运维之外的很多个领域，延伸到了软件研发的全生命周期中的方方面面，比如安全、测试等。DevOps 不再只是关注软件研发过程，而是关注如何消除业务与用户之间的限制，不仅是提供新功能和产品，还着力为用户提供真正价值。</p>
<p data-nodeid="44089">近几年，随着企业数字化转型的趋势不断加速，企业文化、领导力和团队动力也将发生变化，企业对 DevOps 的需求也在不断增加。 未来，任何一个企业都将是数字化企业，产品和服务也都将走向数字化。比如，银行、电商、汽车、制造业等也被定义为软件驱动的企业，通过软件服务为企业赋能，促进企业发展。这些企业将比以往任何时候都需要软件的快速开发和交付，以满足用户不断变化的需求。</p>
<p data-nodeid="44090">企业领导者也越来越重视 DevOps，通过招聘 DevOps 人才加快企业数字化转型。特别是从 2020 年新冠疫情开始，这一趋势更加明显。DevOps 不仅在意识、文化和组织结构上在不断变化，技术上的变化也在推动 DevOps 地发展。比如，云计算、容器化、微服务架构、数据科学与人工智能等，下面介绍一下云原生和数据科学在未来一段时间对 DevOps 的影响。</p>
<h4 data-nodeid="44091">云原生 DevOps</h4>
<p data-nodeid="44092">虚拟化和云计算、基础设施即代码、平台和应用程序容器化都是推动DevOps发展的重要推动力。云原生 DevOps 就是在这样的技术时代背景下产生的。<strong data-nodeid="44185">云原生是一种利用云计算优势构建和运行应用程序的实践</strong>。基于云平台按需、无限扩容的能力，利用容器和 Kubernetes 等云原生技术实现应用程序的自动化和可伸缩性，从而提高业务的交付速度。</p>
<p data-nodeid="44093">这里需要注意的是，云原生应用不一定就是部署到云上的应用。云原生是一种定义应用程序和系统架构的思维方式，跟在哪里运行无关。由于应用程序是基于云的优势进行构建、测试和部署，采用的都是以云为中心的工具和平台，因此称这些DevOps实践为云原生DevOps，下面是构成<strong data-nodeid="44191">云原生 DevOps</strong>的组成部分：</p>
<p data-nodeid="44335" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/00/94/Cgp9HWAaVzuAOcVgAABzJ4Snows222.png" alt="202123-75618.png" data-nodeid="44338"></p>

<p data-nodeid="44095">基于云原生构建和部署的应用程序具有以下特点：</p>
<ul data-nodeid="44096">
<li data-nodeid="44097">
<p data-nodeid="44098"><strong data-nodeid="44200">微服务架构</strong>：云原生应用利用微服务架构，每个应用程序都是可以彼此独立运行的小型服务单元。每个应用都归属于不同的团队，并且由各自团队根据团队的发布计划进行开发、部署和升级。</p>
</li>
<li data-nodeid="44099">
<p data-nodeid="44100"><strong data-nodeid="44205">容器化</strong>：云原生应用都是以容易化部署，容器化为应用程序提供了硬件隔离的运行环境，使它更易移植。基于容器的应用程序使得部署和卸载都更加方便。</p>
</li>
<li data-nodeid="44101">
<p data-nodeid="44102"><strong data-nodeid="44210">持续交付模型</strong>：云原生应用基于持续交付模型进行自动化构建、测试和发布。可以采用滚动部署策略，在不影响最终用户的前提下进行部署。持续交付模型采用 DevOps 原则，促进软件开发人员和运维人员之间的协作，快速、频繁且持续地进行软件的交付。</p>
</li>
</ul>
<p data-nodeid="44103">云原生 DevOps 的核心是一种提高业务交付速度的方法，也是一种构建团队的方法。基于云原生技术，我们可以实现以下几点。</p>
<ul data-nodeid="44104">
<li data-nodeid="44105">
<p data-nodeid="44106"><strong data-nodeid="44216">可扩展性</strong>：充分利用平台的动态扩缩容特性，拥抱故障而不是尝试阻止故障。</p>
</li>
<li data-nodeid="44107">
<p data-nodeid="44108"><strong data-nodeid="44221">敏捷性</strong>：允许应用程序快速部署和快速迭代。</p>
</li>
<li data-nodeid="44109">
<p data-nodeid="44110"><strong data-nodeid="44226">可运维性</strong>：从应用程序内部添加对应用程序生命周期的控制，而不是依赖于外部的监控系统。</p>
</li>
<li data-nodeid="44111">
<p data-nodeid="44112"><strong data-nodeid="44231">可观测性</strong>：应用程序需提供接口和信息以帮助了解应用程序的状态问题。</p>
</li>
</ul>
<h4 data-nodeid="44113">OAM 下一代 DevOps</h4>
<p data-nodeid="44114">OAM 被认为是下一代 DevOps，是由微软和阿里云共同创建的开放应用程序模型（Open Application Model，OAM）。OAM 是用于描述应用程序的规范，以便将应用程序描述与应用程序部署的基础结构进行分离。在DevOps实践中，不断促进开发和运维之间的协作，而 OAM 规范也是为了促进这一点。OAM 的目标是“<strong data-nodeid="44238">使简单的应用程序变得容易，复杂的应用程序可管理</strong>”。下图是 OAM 的模型图，说明了模型中不同的角色以及各自的职责。</p>
<p data-nodeid="44115"><img src="https://s0.lgstatic.com/i/image6/M00/00/89/Cgp9HWAaSGmAJWGkAAFCHWYyTSw756.png" alt="Drawing 1.png" data-nodeid="44241"></p>
<p data-nodeid="44116">在 OAM 中，应用程序由两个概念组成。</p>
<ul data-nodeid="44117">
<li data-nodeid="44118">
<p data-nodeid="44119"><strong data-nodeid="44247">组件</strong>：组件可能是服务，如 MySQL 数据库或 Web 服务器。开发人员编写将他们打包为组件的代码，然后编写描述该组件与其他微服务之间关系的清单。组件使得开发人员能够构建可重用的模块，这些模块封装了围绕安全性和可扩展性部署的最佳实践。另外，还将组件的实现与这些组件如何在分布式应用程序体系结构中组合的描述分开。</p>
</li>
<li data-nodeid="44120">
<p data-nodeid="44121"><strong data-nodeid="44252">特征集</strong>：这些特征描述了应用程序环境的特征，包括对应用程序的运维很重要但可以在不同环境中以不同方式实现的负载均衡、自动扩缩容等。比如，使用软件负载均衡器与硬件负载均衡器，从应用程序开发人员的角度来看是一样的，但对于运维人员来说，是完全不同的。特征使得关注点分离，从而使应用程序在部署该特征的任意地方执行。</p>
</li>
</ul>
<p data-nodeid="44122">OAM 是一个与平台无关、与供应商无关的规范，该规范是根据 Open Web Foundation 协议开发的。目前基于 Kubernetes 构建的 Rudr、Kubevela 已经实现了 OAM 的规范，相信在不久的将来会更加成熟。</p>
<h3 data-nodeid="44123">AI 与 DevOps</h3>
<p data-nodeid="44124">AI 与 DevOps 这两者是相辅相成，相互促进的关系，一方面 DevOps 实践加速了 AI 的模型训练，目前已经出现了针对人工智能、机器学习方面的 DevOps 工具。另一方面 AI 也使得DevOps 更加智能，更加准确。根据 Gartner 统计，到 2023 年，将有 40% 的 DevOps 团队将在 AIOps 平台中集成人工智能。下面分别从这两个方面介绍一下。</p>
<h4 data-nodeid="44125">MLOps</h4>
<p data-nodeid="44126">MLOps 是在人工智能领域应用 DevOps 的实践。它是关于如何更好地管理数据科学家和运维人员，以便有效的开发、部署和监控模型。人工智能领域的关键部分是对模型和数据的管理，目前针对这两部分都有对应的开源工具。</p>
<p data-nodeid="44127"><strong data-nodeid="44263">1. CML</strong></p>
<p data-nodeid="44128">CML，持续机器学习（Continuous Machine Learning）是一个开放源代码库，用于在机器学习项目中实施持续集成和持续交付。使用 CML 可以自动化一部分开发流程，包括模型训练和评估，在整个项目历史中比较机器学习的实验结果，以及监控数据集的变化。</p>
<p data-nodeid="44129">CML为机器学习模型的构建提供了以下功能。</p>
<ul data-nodeid="44130">
<li data-nodeid="44131">
<p data-nodeid="44132"><strong data-nodeid="44270">数据科学 GitFlow</strong>：使用 GitLab 或 GitHub 管理 ML 实验，跟踪谁训练了 ML 模型、谁修改了数据以及何时执行的。使用 DVC 进行编码数据和模型存储，而不是Git仓库。</p>
</li>
<li data-nodeid="44133">
<p data-nodeid="44134"><strong data-nodeid="44275">自动生成机器学习实验的报告</strong>：在每个 Git Pull Request 时自动生成机器学习报告，即机器学习的持续集成。每次生成的报告可以帮助团队做出以数据为依据的决策。</p>
</li>
<li data-nodeid="44135">
<p data-nodeid="44136"><strong data-nodeid="44280">不需要依赖其他服务</strong>：使用 GitHub 或 GitLab 即可构建 ML 平台，不需要数据、服务或复杂的设置。</p>
</li>
</ul>
<p data-nodeid="44137"><strong data-nodeid="44286">2. DVC</strong></p>
<p data-nodeid="44138">DVC，数据版本控制（Data Version Control）是一个用于管理ML数据和模型的工具，它利用了已经熟悉的现有工具集，如 Git，CI/CD 等。DVC 分为以下几个功能组件。</p>
<ul data-nodeid="44139">
<li data-nodeid="44140">
<p data-nodeid="44141"><strong data-nodeid="44292">数据版本控制</strong>：是 DVC 的基础层，用于存储数据集和机器学习的模型。仓库只用于存储数据，可以认为是“Git for Data”。通过将数据存储在共享库中，可以更有效的与他人共享。</p>
</li>
<li data-nodeid="44142">
<p data-nodeid="44143"><strong data-nodeid="44297">数据访问</strong>：用于从项目外部访问数据以及如何从另一个DVC项目导入数据。可以帮助将ML模型的特定版本下载到部署服务器或将模型导入到另一个项目中。</p>
</li>
<li data-nodeid="44144">
<p data-nodeid="44145"><strong data-nodeid="44302">数据管道</strong>：用于构建模型和其他数据制品，并提供了一种类似构建流水线的方式可靠、重复的执行。</p>
</li>
<li data-nodeid="44146">
<p data-nodeid="44147"><strong data-nodeid="44307">数据实验</strong>：可以在 DVC 中通过参数、指标或图形完成数据实验地执行、捕获和浏览，可以认为是“Git for Machine Learning”。</p>
</li>
</ul>
<h4 data-nodeid="44148">AIOps</h4>
<p data-nodeid="44149">AIOps 仍然是未来的趋势，DevOps 团队正在加快将 AI 和机器学习技术集成到DevOps平台中，以更高的准确性、质量和可靠性加快软件开发全生命周期的每个阶段。将AI应用到DevOps中的方式有以下几种。</p>
<ul data-nodeid="44150">
<li data-nodeid="44151">
<p data-nodeid="44152"><strong data-nodeid="44314">AI 监测代码错误和自动化建议以改善代码质量</strong>。</p>
</li>
</ul>
<p data-nodeid="44153">阿里云效平台研发发的 PRECFIX，通过分析海量的历史代码提交数据，提取潜在的“代码缺陷对”，利用聚类算法将相似度较高的修复方式总结出来，得到缺陷和修复模板。</p>
<ul data-nodeid="44154">
<li data-nodeid="44155">
<p data-nodeid="44156"><strong data-nodeid="44320">基于给定代码库的属性自动生成和运行测试用例来提高软件质量</strong>。</p>
</li>
</ul>
<p data-nodeid="44157">在任何 DevOps 团队中，创建和维护测试用例都是一项耗时操作，每次代码更新都要同时修改测试用例。基于AI的测试用例生成和执行方式，在提高测试用例一致性和准确性的同时，也节省了DevOps团队的宝贵时间。</p>
<ul data-nodeid="44158">
<li data-nodeid="44159">
<p data-nodeid="44160"><strong data-nodeid="44326">使用 AI 技术发现和分析应用程序监控的数据，提高监控系统的准确性</strong>。</p>
</li>
</ul>
<p data-nodeid="44161">应用程序在运行时出现异常的根因分析是一项复杂并且烦琐的操作，导致目前的大多数监控系统的告警只是停留在表象，并未给出具体的原因和修复建议。基于 AI 的异常根因分析，能够在出现异常时，给出导致问题的根本原因以及修复的建议。</p>
<p data-nodeid="44162">DevOps 团队都面临着加快研发进度，提高代码质量的挑战，经过几年的发展，基于现有的技术水平很难进一步提高。AI 通过对历史数据的分析、学习，可以帮助 DevOps 团队在研发效能领域更上一层楼。</p>
<p data-nodeid="44163">这个课时是本专栏的最后一讲，主要介绍了一下 DevOps 未来发展的方向和趋势，说是未来，其实已经在路上。技术的发展促进了各行各业的技术变革，作为近几年软件工程领域非常重要的实践，DevOps 随着技术的发展也在不断创新。</p>
<p data-nodeid="44164">如今，我们身处在这样一个乌卡（VUCA）时代，突如其来的新冠疫情彻底改变了我们的工作和生活方式。只有拥抱变化、迭代进化才能应对这个复杂多变的时代。不管时代如何变化，技术如何更新，DevOps 的思想是不变的，DevOps 没有终点，一直都在路上，也希望在探索 DevOps的道路上有你！</p>
<p data-nodeid="44165">如果有时间，也请填写一下下面这份问卷，对我的课程给予一个反馈，谢谢！</p>
<p data-nodeid="44166" class=""><a href="https://wj.qq.com/s2/7971419/479f/" data-nodeid="44334">https://wj.qq.com/s2/7971419/479f/</a></p>

---

### 精选评论

##### *彦：
> 所以最后测测自己的课程留存率？顺便说一句，我怎么感觉留言表情输出的翻页坏了，算不算是技术债务😓

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，这个应该算是线上问题，你发现了一个线上的大BUG，是问题 不是债务。

