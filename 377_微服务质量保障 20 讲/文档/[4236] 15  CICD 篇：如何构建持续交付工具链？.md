<p data-nodeid="7013" class="">上一课时，我讲解了如何更好地利用多个测试环境。本课时，我来讲解下如何构建持续交付工具链。</p>
<h3 data-nodeid="7014">持续交付工具链</h3>
<p data-nodeid="7015">持续交付在上一篇文章中已经提到，它是指所有开发人员始终让 Master 分支保持可随时发布的状态，根据实际需要来判断是否进行一键式发布。而工具链（Tool Chain）通常是指一系列工具，它们按照一定的逻辑顺序运行，最终完成一件比较复杂的事情。</p>
<p data-nodeid="7016">因此，持续交付工具链是帮助我们把持续交付进行落地的工具集合或自动化平台，它可以固化产品交付过程中的各个环节，实现自动化地构建、部署、测试、输出报告等工作。如下图所示。</p>
<p data-nodeid="7017"><img src="https://s0.lgstatic.com/i/image/M00/4A/0D/CgqCHl9QjxSAYZYgAAd1WJT6DX0137.png" alt="Drawing 0.png" data-nodeid="7127"></p>
<div data-nodeid="7018"><p style="text-align:center">持续交付示意图</p></div>
<p data-nodeid="7019"><strong data-nodeid="7131">构建持续交付工具链需要考虑哪些内容？</strong></p>
<p data-nodeid="7020">通过上面的描述，不难看出，构建持续交付工具链涉及如下工作。</p>
<ul data-nodeid="7021">
<li data-nodeid="7022">
<p data-nodeid="7023">基础设施盘点：持续交付包含了产品交付过程的方方面面，因此需要盘点清楚在整个公司或项目里，现有的研发基础设施是怎样的，如何做代码和配置的管理，各个环境如何管理，构建和部署在多大程度上实现了自动化，测试阶段是如何流转的，有着怎样的质量目标，各种类型的自动化测试的建设情况，如何感知关键节点的变更和反馈，等等。可见，如果工具链是一座摩天大楼，那么研发基础设施就是它的地基。</p>
</li>
<li data-nodeid="7024">
<p data-nodeid="7025">组织支持：持续交付的建设涉及交付过程中的多个团队共同协同，所以不仅需要各部门管理层的支持，还需要一线员工有强烈的改进意愿。</p>
</li>
<li data-nodeid="7026">
<p data-nodeid="7027">关键过程自动化：针对上述基础设施进行盘点后，需要对其中的每一个环节尽可能地进行自动化管理，引入合适的工具或者自建工具来完成。比如，如果测试过程是纯手工测试，那么就难以在持续交付中发挥作用，因此可以把重复性的手工测试工作工具化或自动化，比如使用 Curl 或 HTTPclient 编写 HTTP 接口的自动化脚本，使用 Selenium 进行端到端测试，等等。在这个过程，要特别注意的是，尽量基于现有的研发基础进行工具化或自动化改造，持续交付涉及的环节太多了，切记不可重复造轮子，尽量和其他团队共同建设，否则太容易和其他团队形成对立的局面，最终拖垮整个工具链的建设。</p>
</li>
<li data-nodeid="7028">
<p data-nodeid="7029">工具的整合：最后需要用持续交付工具对上述工具或自动化设施进行整合，实现“链”的效果。</p>
</li>
</ul>
<p data-nodeid="7030">现阶段持续集成和持续交付思想已经盛行起来，绝大多数的公司和团队能够认识到这种变化的重要性，因此组织方面的支持通常没有太大问题，但需要考虑落地的成本。因此，对于持续交付工具链的建设，可以借鉴常规的产品研发项目，使用小步快跑的方式，以“先有后优”的心态建设。如以下措施。</p>
<ul data-nodeid="7698">
<li data-nodeid="7699">
<p data-nodeid="7700" class="">规模：在落地规模上，先在小范围试点，逐渐成熟了之后再推广到更大的范围。</p>
</li>
<li data-nodeid="7701">
<p data-nodeid="7702">成熟度：持续交付体系的搭建几乎是永无止境的，应先实现框架，再逐步丰富或完善各个环节，使成熟度逐渐改进。</p>
</li>
</ul>


<h3 data-nodeid="7036">持续交付全流程：盘点与改进</h3>
<p data-nodeid="7037">在产品研发交付过程中，不外乎有如下几个方面：代码&amp;配置管理、构建&amp;部署自动化、各种测试、反馈相关等内容。</p>
<h4 data-nodeid="7038">代码&amp;配置管理</h4>
<p data-nodeid="7039">通常来说，代码用成熟的工具管理起来的成本不高，比如常见的代码管理工具 GitHub、Atlassian Stash、GitLab 等。但在配置方面，常常有比较明显的问题，需要将配置进行统一化、自动化地管理。</p>
<h4 data-nodeid="7040">构建&amp;部署自动化</h4>
<p data-nodeid="7041">在构建方面，比较推荐的是 Maven，它的“惯例胜于配置”的原则，使你只要按它指定的方式组织代码，就可以使用一条命令执行所有的构建、部署、测试和发布任务。而且它能自动管理项目间的依赖，这对于构建持续交付来说太友好了。</p>
<p data-nodeid="7042">环境部署则需要能够对测试环境、预发布环境、生产环境的修改用自动化来完成。如果你还在以手工的方式远程登录到这些环境上执行部署工作，那就太 out 了。现阶段将部署完全脚本化已不存在任何技术难度。如果需要部署多台机器，充其量就是先分发到这些机器上，再在这些机器的本地执行部署脚本。</p>
<h4 data-nodeid="7043">各种测试</h4>
<p data-nodeid="7044">在没有持续集成和持续交付建设的团队里，测试环节通常比较滞后，且人工占比较高，这无疑给项目带来比较大的质量风险。因此，在测试环节的一个非常重要的策略是要尽可能地把各种测试过程自动化，且覆盖度和稳定性达到一定要求。</p>
<p data-nodeid="7045">整个测试过程不仅有静态测试，也有动态测试。静态测试中的各类文档评审，比较难以自动化，但静态代码检查通常有现成的工具，比如最流行的 Sonar。动态测试包含了功能测试（微服务架构的分层测试）和非功能测试（性能测试、安全性测试），可以针对这些类型的测试进行自动化的改造。比如使用 Spring Cloud Contract 和 Pact 可以进行微服务的契约测试，使用 Jmeter 以非 GUI 的方式做性能测试、使用 SQLmap 检测诸如 SQL 注入的安全问题。自动化建设的投入一定要遵循“测试金字塔”，努力提升单元测试自动化的比重，同时降低端到端自动化测试用例的比重。</p>
<p data-nodeid="7046">在进行自动化建设时，并不是每种测试都需要自动化，尽量只把执行过程中易出错、烦琐的步骤变成可靠且可重复的自动化步骤。比如，每次测试用户评价都需要先构造一笔真实的订单，那么构造订单和用户评价都属于操作烦琐的步骤。当然对于有些测试内容，人比机器更靠谱，手工测试必不可少，比如体验类、界面类功能等。</p>
<p data-nodeid="7047">另外，在持续发布过程中，还需要在 Staging 和 Prod 环境进行回归。通常，这两个环境因为涉及线上数据库，以自动化的方式在这两个环境写入数据会有比较高的质量风险，因此可以只进行读取操作的自动化，其他内容用手工测试来完成。</p>
<p data-nodeid="7048">在持续交付过程中，对自动化测试有四个关键要求。</p>
<ul data-nodeid="7049">
<li data-nodeid="7050">
<p data-nodeid="7051">速度 ：自动化测试必须快速运行，以便在发生故障时向团队提供快速反馈。应尽可能多地使用单元测试，引入少量的端到端自动化用例，以便发现单元测试可能无法发现的故障。最后，应该少进行 UI 类的测试，因为它们花费很长的编写时间和很长的运行时间，尽管它们有时能够发现其他缺陷。</p>
</li>
<li data-nodeid="7052">
<p data-nodeid="7053">可靠性： 持续交付过程中运行自动化的关键在于自动化需要足够稳定。如果自动化发现了新的缺陷，那么其运行不稳定是良性的。否则，持续交付过程中自动化用例执行的频率很高，不稳定的自动化 case 会消耗掉一个测试人员太多的维护时间。</p>
</li>
<li data-nodeid="7054">
<p data-nodeid="7055">数量：由于每次代码或环境变更都会触发自动化测试，随着用例数不断增多，自动化用例执行的时间也在增加，因此在编写用例时要避免增加无效的 case，避免设置无效的轮询等待时长，尽可能多地捕获相关的异常。</p>
</li>
<li data-nodeid="7056">
<p data-nodeid="7057">维护：保持高稳定性离不开自动化测试用例的维护。</p>
</li>
</ul>
<h4 data-nodeid="7058">反馈相关</h4>
<p data-nodeid="7059">要想能做到快速反馈，就需要关键阶段有结果并对结果进行通知。</p>
<ul data-nodeid="7060">
<li data-nodeid="7061">
<p data-nodeid="7062">结果</p>
</li>
</ul>
<p data-nodeid="7063">结果分为两种：构建或部署结果、测试结果。</p>
<ul data-nodeid="7064">
<li data-nodeid="7065">
<ul data-nodeid="7066">
<li data-nodeid="7067">
<p data-nodeid="7068">构建和部署的结果相对明确，判断是否与构建和成功部署的状态匹配即可。</p>
</li>
<li data-nodeid="7069">
<p data-nodeid="7070">测试结果需要设置预期结果，比如测试通过率、代码覆盖率等，也可以设置不同阶段的测试目标，如代码提交后的测试、研发人员提交测试给测试人员、发布前测试等。不同阶段的测试内容也不尽相同，比如代码提交后的测试，主要是静态代码检查、分支规范检查，而提测时的检查则涉及自动部署检查、冒烟测试自动化、功能自动化测试和非功能测试自动化等内容。</p>
</li>
</ul>
</li>
<li data-nodeid="7071">
<p data-nodeid="7072">通知</p>
</li>
</ul>
<p data-nodeid="7073">通知的方式主要通过 IM 或者邮件的方式进行，尽可能做到每一个关键步骤都有通知。对于不满足质量要求的通知，需要有更强烈的通知，比如标红的文字或者多种方式联合通知。</p>
<ul data-nodeid="7074">
<li data-nodeid="7075">
<p data-nodeid="7076">运营</p>
</li>
</ul>
<p data-nodeid="7077">只是通知还不够，很多时候一些不太好的数据需要持续的运营。比如，部署环境总是失败，阻塞了工具链的执行；提测后自动化用例执行失败较多，需要进一步查看是自动化用例稳定性问题、环境稳定性问题还是代码质量问题；这些内容都需要有量化的数据才有利于改进。</p>
<h3 data-nodeid="7078">工具的整合</h3>
<p data-nodeid="7079">用于持续交付的工具有很多，这里不一一列举，借用<a href="http://www.jamesbowman.me/" data-nodeid="7179">James Bowman</a>的一张图：</p>
<p data-nodeid="7080"><img src="https://s0.lgstatic.com/i/image/M00/4A/0D/CgqCHl9Qj3iAKWw_ABiFJEg8CSU454.png" alt="Drawing 1.png" data-nodeid="7183"></p>
<p data-nodeid="7081">除了图片中的工具，还有很多在持续交付过程中发挥作用的工具：</p>
<ul data-nodeid="7082">
<li data-nodeid="7083">
<p data-nodeid="7084">服务发现和全局配置存储，例如 ZooKeeper 等；</p>
</li>
<li data-nodeid="7085">
<p data-nodeid="7086">安全管理和监视工具，例如 Fortify、Vault 等；</p>
</li>
<li data-nodeid="7087">
<p data-nodeid="7088">静态代码分析工具，例如循环复杂度、覆盖范围、质量、标准等；</p>
</li>
<li data-nodeid="7089">
<p data-nodeid="7090">编程语言，工具和框架，例如编译器、IDE 等；</p>
</li>
<li data-nodeid="7091">
<p data-nodeid="7092">用于测试的模拟工具，例如 Mockito 等；</p>
</li>
<li data-nodeid="7093">
<p data-nodeid="7094">质量管理工具，例如 Jira 等；</p>
</li>
<li data-nodeid="7095">
<p data-nodeid="7096">发布管理工具，例如 LaunchDarkly 等；</p>
</li>
<li data-nodeid="7097">
<p data-nodeid="7098">特定于云供应商的工具和工具链，例如适用于 AWS 的 Cloudformation 和 CodeDeploy 等。</p>
</li>
</ul>
<p data-nodeid="8157" class="">可以对持续交付过程的工具进行整合的工具有很多，最常用的是 Jenkins，其中尤为推崇 Jenkins 2.x 。</p>

<p data-nodeid="9070" class="">jenkins 1.0 虽然也可以实现自动化构建，但 Jenkins 2.x 的精髓是<strong data-nodeid="9076">Pipeline as Code</strong>，它能将 project 中的配置信息以 steps 的方式放在一个脚本里，将原本独立运行于单个或者多个节点的任务连接起来，实现单个任务难以完成的复杂流程，形成流水式发布。</p>


<p data-nodeid="7101">Jenkins Pipeline 脚本示例：</p>
<pre class="lang-groovy" data-nodeid="7102"><code data-language="groovy">Jenkinsfile (Declarative Pipeline)
pipeline { 
    agent any 
    options {
        skipStagesAfterUnstable()
    }
    stages {
        stage(<span class="hljs-string">'Build'</span>) { 
            steps { 
                sh <span class="hljs-string">'make'</span> 
            }
        }
        stage(<span class="hljs-string">'Test'</span>){
            steps {
                sh <span class="hljs-string">'make check'</span>
                junit <span class="hljs-string">'reports/**/*.xml'</span> 
            }
        }
        stage(<span class="hljs-string">'Deploy'</span>) {
            steps {
                sh <span class="hljs-string">'make publish'</span>
            }
        }
    }
}
</code></pre>
<p data-nodeid="7103">Jenkins Pipeline 可以轻松实现如下所示的持续交付效果：</p>
<p data-nodeid="7104"><img src="https://s0.lgstatic.com/i/image/M00/4A/02/Ciqc1F9Qj4uADoy1AAHSlw_w0bw987.png" alt="Drawing 2.png" data-nodeid="7204"></p>
<h3 data-nodeid="7105">总结</h3>
<p data-nodeid="7106">本节课我首先讲解了持续交付工具链，它是帮助我们把持续交付进行落地的工具集合或自动化平台，用于固化产品交付过程中的各个环节，实现自动化地构建、部署、测试、输出报告等工作。然后讲解了构建持续交付工具链需要进行基础设施盘点、组织支持、关键过程自动化、工具的整合。</p>
<p data-nodeid="7107">接着我讲解了持续交付全流程中的关键过程，如下所述。</p>
<ul data-nodeid="7108">
<li data-nodeid="7109">
<p data-nodeid="7110">代码&amp;配置管理：代码比较容易用成熟的工具管理，配置也需要进行统一化、自动化地管理。</p>
</li>
<li data-nodeid="7111">
<p data-nodeid="7112">构建&amp;部署自动化：推荐使用 Maven，因为按其指定的方式组织代码后，可以实现一条命令搞定所有的构建、部署、测试和发布。另外环境部署实现脚本化也非难事。</p>
</li>
<li data-nodeid="7113">
<p data-nodeid="7114">各种测试：尽可能地将整个测试过程中的各种测试类型进行自动化或工具化，如使用 Sonar 进行静态代码扫描，使用 Spring Cloud Contract 和 Pact 可以进行微服务的契约测试，使用 Jmeter 以非 GUI 的方式做性能测试、使用 SQLmap 可以检测诸如 SQL 注入的安全问题，等等。</p>
</li>
<li data-nodeid="7115">
<p data-nodeid="7116">反馈相关：要想能做到快速反馈，就需要关键阶段有结果并对结果进行通知。其中，结果包含构建结果、部署结果和各种测试结果。通知则一般以IM或邮件方式体现。在此之外，还应对交付过程中数据进行运营和改进。</p>
</li>
</ul>
<p data-nodeid="9529" class="">最后列举出了可用于持续交付的工具，并且推荐使用 Jenkins 2.x 版本进行这些工具的整合。</p>

<p data-nodeid="7118">你所负责的业务或项目，持续交付工具链是怎样建设的，实现了怎样的效果？请写在留言区里。</p>
<blockquote data-nodeid="7119">
<p data-nodeid="7120" class="">相关链接：<br>
CI 自动化测试是 DevOps 的基础<br>
https://www.kubernetes.org.cn/7279.html<br>
https://insights.thoughtworks.cn/devops-and-continuous-delivery/<br>
持续交付工具链<br>
Continuous delivery tool landscape：<br>
<a href="http://www.jamesbowman.me/post/continuous-delivery-tool-landscape/" data-nodeid="7232">http://www.jamesbowman.me/post/continuous-delivery-tool-landscape/</a><br>
Jenkins：<br>
<a href="https://jenkins.io/doc/book/pipeline/" data-nodeid="7238">https://jenkins.io/doc/book/pipeline/</a></p>
</blockquote>

---

### 精选评论


