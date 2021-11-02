<p data-nodeid="25798" class="">本节课是 Spark 课程的彩蛋，教你如何成为 Spark Contributor，即为 Spark 贡献代码的人。</p>


<p data-nodeid="25518">你一定知道，GitHub 是全球最大的社会化编程及代码托管网站，随着 GitHub 的出现，软件开发者才真正拥有了源代码，世界各地的人都可以比从前更加容易地获得源代码，将其自由更改并加以公开。如今，世界上众多的程序员都在通过 GitHub 公开源代码，越来越多的项目也选择 GitHub 进行托管，其中有 Linux 这种史诗怪兽级项目，也不乏 Akka 这种精致小巧的项目。</p>
<p data-nodeid="25519">Apache Spark 也将代码托管在 GitHub 上，这样就能利用 GitHub 强大的多人协作能力，轻而易举地使世界各地的程序员共同为 Spark 进行开发。GitHub 独有的 Pull Request 机制，能够使不同背景、不同地域、互不相识的程序员产生化学反应，不得不说是一件很奇妙的事情。</p>
<p data-nodeid="25520">我们从 GitHub 上面可以看到 Spark Contributor 中有很多中国人的面孔，这非常值得高兴，我也曾经通过向 Spark 贡献代码成为 Spark Contributor 的一员。在大数据时代，中国很有希望实现弯道超车。本节将介绍如何通过 GitHub 的 Pull Request 功能，向 Spark 贡献代码。</p>
<h3 data-nodeid="25521">如何向 Spark 贡献代码</h3>
<p data-nodeid="25522">具体步骤如下：</p>
<h4 data-nodeid="25523">将 Spark 项目克隆（即 Fork）到自己的账号下</h4>
<p data-nodeid="26160">首先，我们要把 Spark 项目克隆（即 Fork）到自己的账号下。你需要注册一个 GitHub 账号，这样在 GitHub 下就拥有了自己的域名。然后搜索 Spark 项目，点击 Fork。你的账号（域名）下就会有一份 Spark 的代码仓库。这是属于你自己的代码仓库，可以任意修改，如下图所示。</p>
<p data-nodeid="26161" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbueAZ8LVAABAJ68Lpoc394.png" alt="Drawing 0.png" data-nodeid="26165"></p>


<h4 data-nodeid="25526">创建一个用于修改的分支</h4>
<p data-nodeid="25527">在创建之前，我们已经在自己的代码仓库上有了一份 Spark 代码的镜像，这时我们当然可以在这份镜像的 master 分支上直接进行修改并发起 Pull Request。但更好的做法是用自己的 master 分支去追踪 Spark 项目的 master 分支（因为 master 分支会随着 Pull Request 不断变化），并根据当前的 master 分支创建一个用于修改的分支。在该分支上进行修改，修改完成后，再发起 Pull Request。</p>
<p data-nodeid="26526">我们创建了 mybranch 分支，如下图所示。</p>
<p data-nodeid="26527" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbu-ATUugAABae51Pbgg918.png" alt="Drawing 1.png" data-nodeid="26531"></p>


<h4 data-nodeid="25530">将当前的代码仓库切换到 mybranch 分支</h4>
<p data-nodeid="26892">现在我们准备对当前最新分支进行修改，所以将当前的代码仓库切换到待修改的分支。如下图所示。</p>
<p data-nodeid="26893" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbvWAWfPdAAAP1iFFXK4291.png" alt="Drawing 2.png" data-nodeid="26897"></p>


<h4 data-nodeid="25533">修改代码</h4>
<p data-nodeid="27258">选定一个文件，点击修改按钮，进行修改，如下图所示。</p>
<p data-nodeid="27259" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbvuARekFAAA9RmSRhHE670.png" alt="Drawing 3.png" data-nodeid="27263"></p>


<p data-nodeid="27624">修改完成后，点击 Commit，提交到 mybranch 分支，如下图所示。</p>
<p data-nodeid="27625" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbwGAQ__8AACKMvG6lwI149.png" alt="Drawing 4.png" data-nodeid="27629"></p>


<h4 data-nodeid="25538">处理冲突，提交 Pull Request</h4>
<p data-nodeid="25539">修改完代码后，就可以准备提交一个 Pull Request，在之前还需要考虑一个问题：在我们修改代码的这段时间，Spark 的 master 分支是否已经接受了别人 Pull Request 而发生了变化？如果是的话，需要先处理冲突，再进行提交，具体步骤如下。</p>
<p data-nodeid="27990">首先，我们通过 Pull Request 选项卡新建一个 Pull Request，如下图所示。</p>
<p data-nodeid="27991" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbweALVImAABBAswO3wI769.png" alt="Drawing 5.png" data-nodeid="27995"></p>


<p data-nodeid="28356">这时候会出现你所修改的版本与你要提交的 Spark 版本的对比页面，如下图所示。</p>
<p data-nodeid="28357" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbw2AXjaXAABf6cM_Mv4422.png" alt="Drawing 6.png" data-nodeid="28361"></p>


<p data-nodeid="28722">在页面的后半部分，会显示两个版本之间的差异。如下图所示。</p>
<p data-nodeid="28723" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/76/CgqCHl9gbxSAAQw4AAEIVttVko8492.png" alt="Drawing 7.png" data-nodeid="28727"></p>


<p data-nodeid="29090">这时，如果你发现 Spark 的 master 版本已经接受了别人的 Pull Request ，同你当初修改的版本发生了变化，就需要进行修改以防止出现冲突。处理完毕后，就可以完成创建了，如下图所示。</p>
<p data-nodeid="29091" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/6B/Ciqc1F9gbxqADf23AABEEaapEeA969.png" alt="Drawing 8.png" data-nodeid="29095"></p>



<p data-nodeid="25548">我们可以在新建 Pull Request 页面将关联的 Jira issue 编号作为题目（如果有的话），便于 Committer 审阅，另外在正文中尽可能地加上对该 Pull Request 的描述，最好写上关联 Jira issue 的链接，Jira 会自动将该 Pull Request 进行关联。如果 Spark Committer 认可了你的 Pull Request，那么具有写权限的 Committer 就会将你的 Pull Request 合并到 Spark 的 master 分支上。</p>
<p data-nodeid="25549">完成了这 5 步，就完成了向 Spark 贡献代码的任务。此外，在学习 Spark 的过程中，也可以将 GitHub 作为源码阅读、心得记录、代码演练的环境。用好 GitHub，将对开源软件的学习起到重要作用。</p>
<p data-nodeid="25550">此外，上述贡献代码的步骤，都可以用 Git 客户端以命令行的形式在本地完成。这需要先熟悉 Git shell 操作，你可以自己尝试一下，Git 客户端可以在 GitHub 官网下载。</p>
<p data-nodeid="25551">本课程的彩蛋就介绍完了。最后，我还是邀请你参与对本专栏的评价，你的每一个观点对我们来说都是最重要的。<a href="https://wj.qq.com/s2/7164306/3a46/" data-nodeid="25608">点击链接，即可参与评价</a>，还有机会获得惊喜奖品！</p>

---

### 精选评论


