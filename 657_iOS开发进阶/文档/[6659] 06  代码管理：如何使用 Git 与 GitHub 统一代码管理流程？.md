<p data-nodeid="808" class="">在软件开发当中，代码管理一直是其中重要的一环，每当软件出现问题，我们就需要查看源码，及时发现其中的漏洞加以修复。并且，由于分工不同，软件开发需要多个人共同完成，如何保证每个人编写的代码符合要求，能够相互配合，也是一个重要的问题。所以，一个非常实用的代码管理工具和一套统一的代码管理流程在开发当中必不可少。</p>
<p data-nodeid="809">而在这方面，StackOverflow 曾调查发现，有超过 87% 的开发者使用 Git，有超过 82% 的开发者使用 GitHub 来进行代码托管和开发协作。可以说，熟练使用 Git 和 GitHub 已经成为开发者的基本技能，而如何结合它们来规范代码管理也是 iOS 开发工程化实践当中的基础。</p>
<h3 data-nodeid="810">Git 分支管理</h3>
<p data-nodeid="811">现代的软件开发活动通常需要多人参与。为了保证不同开发者可以同时贡献到同一个代码库，Git 提供了分支（Branch）来支持并行开发。不同团队有不同的 Git 分支管理方式，根据我们团队多年的经验，经过不断的完善，最终形成出一套简单并十分有效的 Git 分支管理规范，你可以参考下。这套规范是怎样的呢？</p>
<p data-nodeid="812">具体来说，我把所有的 Git 分支分成三类：<strong data-nodeid="890">主分支，功能分支和发布分支</strong>，让它们各自承担不同的功能。</p>
<p data-nodeid="813">其中主分支也称为<code data-backticks="1" data-nodeid="892">master</code>或者<code data-backticks="1" data-nodeid="894">main</code>分支， 是 Git 代码仓库的默认分支。在 Xcode 12 以后，新建项目时也会默认生成命名为<code data-backticks="1" data-nodeid="896">main</code>的主分支。</p>
<p data-nodeid="814">主分支在软件开发中非常重要，它是我们 App 的唯一的信息源（Single source of truth），不论是编译出不同版本的 App 还是排查问题，都需要用到主分支的代码。并且，团队同事的代码，最终也必须汇总到这个主分支中，且不能出错。所以，<strong data-nodeid="902">所谓的统一代码管理流程，就是制定其他分支的代码如何合并到主分支的流程。</strong></p>
<p data-nodeid="815"><strong data-nodeid="907">功能分支</strong>是我们在开发过程中建立的临时分支，它可以用来保存一次开发活动的状态。根据不同的开发活动，我把功能分支分成几个小类。</p>
<ol data-nodeid="816">
<li data-nodeid="817">
<p data-nodeid="818"><code data-backticks="1" data-nodeid="908">feature</code>分支，当开发一个新功能的时候，我会为每一个功能建立一个叫作<code data-backticks="1" data-nodeid="910">feature</code>分支，当整个功能完成后就<del data-nodeid="916">可以</del>合并到主分支里面，并把该分支从 Git 代码仓库中删除掉。</p>
</li>
<li data-nodeid="819">
<p data-nodeid="820"><code data-backticks="1" data-nodeid="917">bugfix</code>分支，当发现 Bug 的时候，会专门建立一个<code data-backticks="1" data-nodeid="919">bugfix</code>分支，修改 Bug 后把它合并到主分支里面去。</p>
</li>
<li data-nodeid="821">
<p data-nodeid="822"><code data-backticks="1" data-nodeid="921">spike</code>分支，当我们探索或研究一些新技术（如 App Clips 功能）的时候，会建立一个叫作<code data-backticks="1" data-nodeid="923">spike</code>的分支。在得出结论以后，我才决定是否把该分支合并到主分支。如果探索失败，我就不会把相关的<code data-backticks="1" data-nodeid="925">spike</code>分支合并到主分支了。</p>
</li>
</ol>
<p data-nodeid="823">在新建一个功能分支的时候，我们都遵循一定的命名规范，一般会把功能的描述作为分支的名称。例如当我们要开发一个点赞功能时，会把该分支可以命名为<code data-backticks="1" data-nodeid="928">feature/add-like-button-to-moments-screen</code>。又例如当我们需要修改用户头像的一个 UI Bug 时，会把分支命名为<code data-backticks="1" data-nodeid="930">bugfix/fix-avatar-ui-bug</code>。</p>
<p data-nodeid="824">有那么多功能分支，在并行开发过程中我们该如何管理它们呢？</p>
<p data-nodeid="825">来看下面这幅。</p>
<p data-nodeid="826"><img src="https://s0.lgstatic.com/i/image6/M00/16/11/CioPOWBF-duAKS0tAAC-6ZH1k9w147.png" alt="Drawing 0.png" data-nodeid="936"></p>
<p data-nodeid="827">举例来说，我有一个同事要开发点赞功能，他就从主分支最新的<code data-backticks="1" data-nodeid="938">MC1</code>commit 签出（checkout）并新建<code data-backticks="1" data-nodeid="940">feature/add-like-button-to-moments-screen</code>来进行点赞功能的开发。与此同时，另外一个同事发现了一个用户头像的 UI Bug，她也从<code data-backticks="1" data-nodeid="942">MC1</code>commit 签出并新建了<code data-backticks="1" data-nodeid="944">bugfix/fix-avatar-ui-bug</code>来修改 Bug。</p>
<p data-nodeid="828">当头像的 Bug 修改完成后，开发组长把<code data-backticks="1" data-nodeid="947">BC2</code>commit 合并到主分支里面。在此之后点赞功能也开发完毕，开发组长又把该功能分支上的<code data-backticks="1" data-nodeid="949">FC4</code>commit 合并到主分支里面。这两个分支彼此独立，而且互不影响。</p>
<p data-nodeid="829">除了主分支和功能分支以外，我们在发布 App 的时候还使用到发布分支。</p>
<p data-nodeid="830"><strong data-nodeid="958">发布分支</strong>一般命名为<code data-backticks="1" data-nodeid="956">release</code>。每次当我们发布 App 之前都会把主分支的最新代码合并到发布分支去。因此发布分支会一直保存 App 发布版的源码记录。</p>
<p data-nodeid="831">有了发布分支以后，一旦发生严重的线上事故，例如出现引起高崩溃率的 Bug 时，我们可以马上在发布分支上进行修复。一般的做法是从发布分支上签出一个功能分支，例如当修复点赞按钮引起的崩溃时，我们可以建立一个叫作<code data-backticks="1" data-nodeid="960">bugfix/fix-like-button-crash</code>的功能分支，在修复该崩溃以后马上合并到发布分支，并提交到 App Store ，更新线上的 App。</p>
<p data-nodeid="832"><img src="https://s0.lgstatic.com/i/image6/M00/16/14/Cgp9HWBF-eaARQ3lAACY5uUYZq0336.png" alt="Drawing 1.png" data-nodeid="964"></p>
<p data-nodeid="833">由上图可见，当我们把主分支最新的<code data-backticks="1" data-nodeid="966">MC1</code>commit 合并到<code data-backticks="1" data-nodeid="968">release</code>分支以后，提交了一个版本号为<code data-backticks="1" data-nodeid="970">V2.0</code>的 App 到 App Store。当我们通过查看崩溃报告，得知新上线的点缀功能引起很多崩溃时，可以采取如下措施：</p>
<ol data-nodeid="834">
<li data-nodeid="835">
<p data-nodeid="836">马上从<code data-backticks="1" data-nodeid="973">release</code>分支的<code data-backticks="1" data-nodeid="975">V2.0</code>commit 签出<code data-backticks="1" data-nodeid="977">bugfix/fix-like-button-crash</code>分支并修复该崩溃；</p>
</li>
<li data-nodeid="837">
<p data-nodeid="838">把<code data-backticks="1" data-nodeid="980">BC2</code>commit 合并回<code data-backticks="1" data-nodeid="982">release</code>分支，并立刻提交版本号为<code data-backticks="1" data-nodeid="984">V2.1</code>的 App 到 App Store；</p>
</li>
<li data-nodeid="839">
<p data-nodeid="840">把包含了该修复的<code data-backticks="1" data-nodeid="987">V2.1</code>commit 合并到<code data-backticks="1" data-nodeid="989">main</code>分支，保证主分支统一管理所有代码的更新状态。</p>
</li>
</ol>
<p data-nodeid="841">有了上面讲的三大分支，以及它们签出和合并的流程，我们就定义了一部分的代码管理规范——知道什么时候要使用哪个分支进行开发或者发布。但是你可能已经察觉到，当我们把功能分支合并到主分支的时候并没有进行任何的审查，万一有人不小心把 Bug 合并到主分支怎么办？</p>
<p data-nodeid="842">我们在这方面就吃过很多大的亏，有人把没有经过验证的代码直接 Push 到主分支，使得 App 的崩溃率提升了 10% 以上。为了保证主分支受控，我们就需要引入一个代码管理流程来管控主分支的合并过程。目前管控这一流程最有效的办法是使用<strong data-nodeid="997">Pull Request</strong>，众多代码托管和协作平台（如 GitLab，BitBucket 等）都支持 Pull Request 功能。下面我就以 GitHub Pull Request 为例子来看看它是如何工作的。</p>
<h3 data-nodeid="843">GitHub Pull Request 流程</h3>
<p data-nodeid="844"><strong data-nodeid="1003">Pull Request</strong>我把它简称为 PR。PR 是一种团队协助的机制，在 GitLab 也叫作 Merge Request。当一个开发者完成一个功能的开发时，可以通过 PR 来通知团队其他成员进行代码审查和讨论。在协商并得到共识后可以通过 PR 把功能代码合并到主分支中。</p>
<p data-nodeid="845">一套完善的 PR 流程能有效降低沟通成本，提高代码质量，以及提升项目的自动化和工程化程度。</p>
<p data-nodeid="846">下面我们一起看一套完整的，并经过我们多年实践证明过的 PR 流程。</p>
<p data-nodeid="847"><img src="https://s0.lgstatic.com/i/image6/M00/16/14/Cgp9HWBF-fKAVu_QAAKUNfBQLr8841.png" alt="Drawing 2.png" data-nodeid="1008"></p>
<p data-nodeid="848">这套流程分成六步。</p>
<p data-nodeid="849">第一步，当我们要开发一个新功能或者修改一个 Bug 时，从主分支的签出并建立一个功能分支。这里需要注意的是，为了减少合并时出现的冲突，我们需要从主分支最新的一个 commit 签出。</p>
<p data-nodeid="850">第二步，我们可以在功能分支上持续开发并多次提交 commit。因为我们是在独立的功能分支上进行开发，所有的变动都不会影响到主分支，所以可以放心修改所需的代码。</p>
<p data-nodeid="851">第三步，当我们完成一个功能的开发以后，就可以提交一个 PR 了。为了避免合并冲突，我建议在提交 PR 前先 rebase 主分支的 Git 历史。同时为了方便其他成员审查代码和参与讨论，我们在提交 PR 的时候需要清楚地描述所完成的功能，并把注意 事项，UI 前后变动的区别，测试步骤等等一同写到 PR 描述文档里面。</p>
<p data-nodeid="852">在我们 Moments App 项目中，为了方便开发者编写 PR 描述文档，我们建立了一个模板文件<code data-backticks="1" data-nodeid="1014">pull_request_template.md</code>。当我们提交 PR 的时候，GitHub 会自动读取并准备好描述文档的模板，我们只需要填写相关内容即可。</p>
<p data-nodeid="853"><img src="https://s0.lgstatic.com/i/image6/M00/1A/11/Cgp9HWBK9gSAXnguAAMUTTFvSAg277.png" alt="图片1.png" data-nodeid="1018"></p>
<p data-nodeid="854">你可以到<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.github/pull_request_template.md" data-nodeid="1022">拉钩的代码仓库</a>查看该模板文件。</p>
<p data-nodeid="855">在提交 PR 的时候，我们还可以加上代码审查人（Reviewer）来通知他/她审查代码。同时也可以加上分类的标签（Label）来方便管理所有的 PR，例如使用<code data-backticks="1" data-nodeid="1025">enhancement</code>表示功能开发，使用<code data-backticks="1" data-nodeid="1027">bug</code>表示 Bug 修改。如果你的项目由多团队同时开发，我们还会为每个产品团队都建立一个标签，这样就能清楚地知道这个 PR 来自那个产品团队了。</p>
<p data-nodeid="856"><img src="https://s0.lgstatic.com/i/image6/M00/1A/0E/CioPOWBK9fKAaGtgAAHLCg2avjU907.png" alt="图片2.png" data-nodeid="1031"></p>
<p data-nodeid="857">第四步，一旦 PR 提交以后，其他成员就会收到通知消息，他们可以进行代码审查，并把反馈意见留言到 RP 里面。提交者可以根据留言来修改代码和提交新的 commit。当所有留言都修正和完善以后，可以再次通知审查人进行进一步的审查。</p>
<p data-nodeid="858">在项目之初，这个过程可能需要来回好几遍。但随着团队的不断磨合，这个过程会越来越快。因为代码审查人一般由项目中最资深开发者组成，他们的留言能帮助团队新成员迅速熟悉项目相关的技术和背景知识。同时能保证代码风格的一致性和提高整个项目的代码质量。</p>
<p data-nodeid="859">根据我们的经验，在代码审查上的投资所得到的好处，远高于所花的时间成本，我建议你或者你所在的团队在项目初期多投入 PR 审查工作。</p>
<p data-nodeid="860">第五步，大家通过协商得到共识以后，审查人就可以批准（approve）该 PR。这表示该 PR 可以随时合并到主分支了。</p>
<p data-nodeid="861">第六步，当 PR 得到批准以后，提交人可以把 PR 合并到主分支里面去。我建议在合并之前先 rebase 主分支上的最新 commit，这样能保证合并过程没有冲突，并使得 Git 的历史更加简洁。</p>
<p data-nodeid="862">上面就是我们的统一的代码管理流程。为了进一步保证少犯错误，我们可以通过修改 GitHub 上的配置来为每一步操作做自动检查，以保证所有成员都必须严格遵循一致的代码管理流程。</p>
<h3 data-nodeid="863">GitHub 配置</h3>
<p data-nodeid="864">首先，我们可以把主分支保护起来，不允许任何人直接 Push 到主分支。</p>
<p data-nodeid="865"><img src="https://s0.lgstatic.com/i/image6/M01/1A/11/Cgp9HWBK9eKAGESRAAJW2j80OLg852.png" alt="图片3.png" data-nodeid="1042"></p>
<p data-nodeid="866">然后要求所有 PR 在合并之前都必须经过一个或以上的代码审查人批准。审查人的数量可以根据团队的情况进行调整。</p>
<p data-nodeid="867"><img src="https://s0.lgstatic.com/i/image6/M01/1A/10/CioPOWBK9pGAWYgdAANH9SPivxA932.png" alt="图片7.png" data-nodeid="1046"></p>
<p data-nodeid="868">当 Github 检查到有某些条件不完全符合时，就不允许我们合并该 PR。</p>
<p data-nodeid="869"><img src="https://s0.lgstatic.com/i/image6/M01/1A/0F/CioPOWBK9lyAEzO1AAOXuc4tDlg494.png" alt="图片6.png" data-nodeid="1050"></p>
<p data-nodeid="870">比如，因为我们的 Moments 项目配置了所有的 PR 都必须有一个或以上的代码审查人批准后才能合并。上图可以见 GitHub PR 页面上的<strong data-nodeid="1056">Merge pull request</strong>按钮是失效的，并提示 “Merging can be performed automatically with 1 approving review” （需要一个代码审查通过后才能合并）。</p>
<h3 data-nodeid="871">总结</h3>
<p data-nodeid="2727">在这一讲我介绍了 Git 的分支管理和 GitHub 的 Pull Request 流程。根据多年的项目经验，我给出了一套完整的统一代码管理流程，其重点是把 Git 分支分成三类，主分支、功能分支和发布分支，然后严格按照 GitHub Pull Request 流程来把代码合并到主分支里面。</p>
<p data-nodeid="3281"><img src="https://s0.lgstatic.com/i/image6/M00/1A/14/Cgp9HWBK-BSAHpViAAapoc7YA9U941.png" alt="思维导图+二维码.png" data-nodeid="3285"></p>
<p data-nodeid="3282">有了这个规范，开发者就能严格遵循这个流程贡献代码，从而保证主分支在管控状态，同时也为项目的自动化和工程化打下基础。</p>







<p data-nodeid="874">思考题：</p>
<blockquote data-nodeid="875">
<p data-nodeid="876">请问你是使用怎样的流程来进行代码管理的？在上述 PR 流程的第一步是从主分支签出一个功能分支，而不是使用 Fork，为什么这样做？</p>
</blockquote>
<p data-nodeid="877">可以把你的答案写得留言区哦，下一讲我将介绍如何统一设计规范，提高沟通效率。</p>
<p data-nodeid="878">源码地址：</p>
<blockquote data-nodeid="879">
<p data-nodeid="880" class="">pull_request_template.md<br>
<a href="https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.github/pull_request_template.md" data-nodeid="1076">https://github.com/lagoueduCol/iOS-linyongjian/blob/main/.github/pull_request_template.md</a></p>
</blockquote>

---

### 精选评论

##### **1127：
> 配合着commit规范食用更佳

##### *虎：
> 同时还可以 rebase 主分支代码，降低 PR 的冲突几率，保持更清晰的提交记录。如果要是 fork 想要达到以上的目的，操作相对复杂些。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 谢谢你分享你的经验，如果你没有创建和 push 分支的权限，确实是需要 Fork，但独立的 Fork 有个缺点是其他成员不容易 push 到你的 Fork 里面。想象一下你提交了一个 PR ，然后放假了，其他人就很难接手你的工作。虽然现在 GitHub 允许代码审核人员修改你的 Fork 和 Branch，但在一个公司里面，大家使用同一个 Repo 能简化很多工作，一个 Branch 可以多人同时开发或者换结对编程的引导者。你说的在提交前需要 rebase Fork 也是对的，其实也不难，就是繁琐些。我们需要在 git remote 设一个 upstream 来指向原先的 Repo，不定时把 upstream 同步到 Fork 的主分支。我们做事的原则是在保证质量的基础上尽量减少无必要的额外工作，所以在条件允许的情况下所有团队都使用同一个 Repo 开发，我们可以在分支命名上加上团队名字。例如 payment-team/feature/add-apple-pay 表示支付团队加入 苹果支付 功能。最后，再次感谢你的分享。

##### **用户3471：
> 请问如果只使用一个主分支打包，那么并行提测但是非同时上线的需求如何提测，多个需求不能同时合并的主分支，

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好问题，我们可以使用功能开关，请看 《第 09 讲| 开关组件：如何使用功能开关，支持产品快速迭代》

##### **泽：
> 老师，这个PR 前先 rebase 主分支的 Git 历史，是什么意思？ 是通过 git rebase -i 调整提交嘛

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这样主分支的 Git 的历史就变成一条直线，没有很多分支。

##### **泽：
> 老师，怎么把控一个commit的提交，因为有时候好几个小功能交叉着， 即提交颗粒度是否频繁一点呢？之后再通过rebase整理提交记录

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是和你团队和个人习惯有关，我偏向于很小的 commit，在提交 PR 时有必要再 rebase。我一个 commit 的原则是，下一步如果做错啥随时回去重写来，小一点比较容易频繁迭代，甚至换人接着干。

##### **飞：
> 你好，我想问下，就是在功能分支或bug分支 开发完成之后、给测试在哪个分支打包？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 测试应该拿主分支进行测试，我们会在第四部分会讲述如何打包和分发。一切都是由 CI 自动完成的。测试人员只需要下载 Moments Beta app 就可以了。

##### **彬：
> 想问下大厂的commit规范是怎样的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我们一般把每个 Commit 只做一个小功能，当然是完整的而且编译可以通过的。因为我们要求代码库可以随时回滚到任意的 Commit 去。在 Commit 的 Message 一般写上所做卡的 ID，然后加上是动词+名词的格式，例如"Add Colors to DesignKit"

##### *虎：
> 请问你是使用怎样的流程来进行代码管理的？在上述 PR 流程的第一步是从主分支签出一个功能分支，而不是使用 Fork，为什么这样做？因为你有权限创建分支，如果没有权限创建分支，必然需要 fork 后，在 PR

