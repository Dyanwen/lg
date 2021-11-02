<p data-nodeid="41119">上一讲，我介绍了 DevOps 的概念与 DevOps 的一些思想。DevOps 的思想可以帮助我们缩短上线周期并且提高软件迭代速度，而 CI/CD 则是 DevOps 思想中最重要的部分。</p>



<p data-nodeid="40591">具体来说，CI/CD 是一种通过在应用开发阶段，引入自动化的手段来频繁地构建应用，并且向客户交付应用的方法。它的核心理念是持续开发、持续部署以及持续交付，它还可以让自动化持续交付贯穿于整个应用生命周期，使得开发和运维统一参与协同支持。</p>
<p data-nodeid="40592">下面我们来详细了解下 CI/CD 。</p>
<h3 data-nodeid="40593">什么是 CI/CD</h3>
<h4 data-nodeid="40594">CI 持续集成（Continuous Integration）</h4>
<p data-nodeid="40595">随着软件功能越来越复杂，一个大型项目要想在规定时间内顺利完成，就需要多位开发人员协同开发。但是，如果我们每个人都负责开发自己的代码，然后集中在某一天将代码合并在一起（称为“合并日”）。你会发现，代码可能会有很多冲突和编译问题，而这个处理过程十分烦琐、耗时，并且需要每一位工程师确认代码是否被覆盖，代码是否完整。这种情况显然不是我们想要看到的，这时持续集成（CI）就可以很好地帮助我们解决这个问题。</p>
<p data-nodeid="40596">CI 持续集成要求开发人员频繁地（甚至是每天）将代码提交到共享分支中。一旦开发人员的代码被合并，将会自动触发构建流程来构建应用，并通过触发自动化测试（单元测试或者集成测试）来验证这些代码的提交，确保这些更改没有对应用造成影响。如果发现提交的代码在测试过程中或者构建过程中有问题，则会马上通知研发人员确认，修改代码并重新提交。通过将以往的定期合并代码的方式，改变为频繁提交代码并且自动构建和测试的方式，可以帮助我们<strong data-nodeid="40674">及早地发现问题和解决冲突，减少代码出错。</strong></p>
<p data-nodeid="40597">传统 CI 流程的实现十分复杂，无法做到标准化交付，而当我们的应用容器化后，应用构建的结果就是 Docker 镜像。代码检查完毕没有缺陷后合并入主分支。此时启动构建流程，构建系统会自动将我们的应用打包成 Docker 镜像，并且推送到镜像仓库。</p>
<h4 data-nodeid="40598">CD 持续交付（Continuous Delivery）</h4>
<p data-nodeid="40599">当我们每次完成代码的测试和构建后，我们需要将编译后的镜像快速发布到测试环境，这时我们的持续交付就登场了。持续交付要求我们实现自动化准备测试环境、自动化测试应用、自动化监控代码质量，并且自动化交付生产环境镜像。</p>
<p data-nodeid="40600">在以前，测试环境的构建是非常耗时的，并且很难保证测试环境和研发环境的一致性。但现在，借助于容器技术，我们可以很方便地构建出一个测试环境，并且可以保证开发和测试环境的一致性，这样不仅可以提高测试效率，还可以提高敏捷性。</p>
<p data-nodeid="40601">容器化后的应用交付过程是这样的，我们将测试的环境交由 QA 来维护，当我们确定好本次上线要发布的功能列表时，我们将不同开发人员开发的 feature 分支的代码合并到 release 分支。然后由 QA 来将构建镜像部署到测试环境，结合自动测试和人工测试、自动检测和人工记录，形成完整的测试报告，并且把测试过程中遇到的问题交由开发人员修改，开发修改无误后再次构建测试环境进行测试。测试没有问题后，自动交付生产环境的镜像到镜像仓库。</p>
<h4 data-nodeid="40602">CD 持续部署（Continuous Deployment）</h4>
<p data-nodeid="40603">CD 不仅有持续交付的含义，还代表持续部署。经测试无误打包完生产环境的镜像后，我们需要把镜像部署到生产环境，持续部署是最后阶段，它作为持续交付的延伸，可以自动将生产环境的镜像发布到生产环境中。</p>
<p data-nodeid="40604">部署业务首先需要我们有一个资源池，实现资源自动化供给，而且有的应用还希望有自动伸缩的能力，根据外部流量自动调整容器的副本数，而这一切在容器云中都将变得十分简单。</p>
<p data-nodeid="40605">我们可以想象，如果有客户提出了反馈，我们通过持续部署在几分钟内，就能在更改完代码的情况下，将新的应用版本发布到生产环境中（假设通过了自动化测试），这时我们就可以实现快速迭代，持续接收和整合用户的反馈，将用户体验做到极致。</p>
<p data-nodeid="40606">讲了这么多概念，也许你会感觉比较枯燥乏味。下面我们就动动手，利用一些工具搭建一个 DevOps 环境。</p>
<p data-nodeid="40607">搭建 DevOps 环境的工具非常多，这里我选择的工具为 Jenkins、Docker 和 GitLab。Jenkins 和 Docker 都已经介绍过了，这里我再介绍一下 Gitlab。</p>
<p data-nodeid="40608">Gitlab 是由 Gitlab Inc. 开发的一款基于 Git 的代码托管平台，它的功能和 GitHub 类似，可以帮助我们存储代码。除此之外，GitLab 还具有在线编辑 wiki、issue 跟踪等功能，另外最新版本的 GitLab 还集成了 CI/CD 功能，不过这里我们仅仅使用 GitLab 的代码存储功能， CI/CD 还是交给我们的老牌持续集成工具 Jenkins 来做。</p>
<h3 data-nodeid="40609">Docker+Jenkins+GitLab 搭建 CI/CD 系统</h3>
<p data-nodeid="40610">软件安装环境如下。</p>
<ul data-nodeid="40611">
<li data-nodeid="40612">
<p data-nodeid="40613">操作系统：CentOS 7</p>
</li>
<li data-nodeid="40614">
<p data-nodeid="40615">Jenkins：tls 长期维护版</p>
</li>
<li data-nodeid="40616">
<p data-nodeid="40617">Docker：18.06</p>
</li>
<li data-nodeid="40618">
<p data-nodeid="40619">GitLab：13.3.8-ce.0</p>
</li>
</ul>
<h4 data-nodeid="40620">第一步：安装 Docker</h4>
<p data-nodeid="40621">安装 Docker 的步骤可以在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=455#/detail/pc?id=4572" data-nodeid="40697">第一讲</a>的内容中找到，这里就不再赘述。Docker 环境准备好后，我们就可以利用 Docker 来部署 GitLab 和 Jenkins 了。</p>
<h4 data-nodeid="40622">第二步：安装 GitLab</h4>
<p data-nodeid="40623">GitLab 官方提供了 GitLab 的 Docker 镜像，因此我们只需要执行以下命令就可以快速启动一个 GitLab 服务了。</p>
<pre class="lang-java" data-nodeid="42508"><code data-language="java">$ docker run -d \
--hostname localhost \
-p <span class="hljs-number">8080</span>:<span class="hljs-number">80</span> -p <span class="hljs-number">2222</span>:<span class="hljs-number">22</span> \
--name gitlab \
--restart always \
--volume /tmp/gitlab/config:/etc/gitlab \
--volume /tmp/gitlab/logs:/<span class="hljs-keyword">var</span>/log/gitlab \
--volume /tmp/gitlab/data:/<span class="hljs-keyword">var</span>/opt/gitlab \
gitlab/gitlab-ce:<span class="hljs-number">13.3</span><span class="hljs-number">.8</span>-ce<span class="hljs-number">.0</span>
</code></pre>




<p data-nodeid="43913">这个启动过程可能需要几分钟的时间。当服务启动后我们就可以通过 <a href="http://localhost:8080" data-nodeid="43918">http://localhost:8080</a> 访问到我们的 GitLab 服务了。</p>
<p data-nodeid="44271"><img src="https://s0.lgstatic.com/i/image/M00/6F/40/CgqCHl-05ceAOEtOAAC7xTjpRgo536.png" alt="Drawing 0.png" data-nodeid="44275"></p>
<div data-nodeid="44272" class=""><p style="text-align:center">图1 GitLab 设置密码界面</p></div>







<p data-nodeid="45298">第一次登陆，GitLab 会要求我们设置管理员密码，我们输入管理员密码后点击确认即可，之后 GitLab 会自动跳转到登录页面。</p>
<p data-nodeid="45299" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/40/CgqCHl-05eKAftEiAACO_hts6R8497.png" alt="Drawing 1.png" data-nodeid="45304"></p>
<div data-nodeid="45300"><p style="text-align:center">图2 GitLab 登录界面</p></div>





<p data-nodeid="40630">然后输入默认管理员用户名：admin@example.com，密码为我们上一步设置的密码。点击登录即可登录到系统中，至此，GitLab 已经安装成功。</p>
<h4 data-nodeid="40631">第三步：安装 Jenkins</h4>
<p data-nodeid="40632">Jenkins 官方提供了 Jenkins 的 Docker 镜像，我们使用 Jenkins  镜像就可以一键启动一个 Jenkins 服务。命令如下：</p>
<pre class="lang-dart" data-nodeid="52850"><code data-language="dart"># docker run -d --name=jenkins \
-p <span class="hljs-number">8888</span>:<span class="hljs-number">8080</span> \
-u root \
--restart always \
-v /<span class="hljs-keyword">var</span>/run/docker.sock:/<span class="hljs-keyword">var</span>/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
-v /tmp/jenkins_home:/<span class="hljs-keyword">var</span>/jenkins_home \
jenkins/jenkins:lts
</code></pre>








<blockquote data-nodeid="40634">
<p data-nodeid="40635">这里，我将 docker.sock 和 docker 二进制挂载到了 Jenkins 容器中，是为了让 Jenkins 可以直接调用 docker 命令来构建应用镜像。</p>
</blockquote>
<p data-nodeid="40636">Jenkins 的默认密码会在容器启动后打印在容器的日志中，我们可以通过以下命令找到 Jenkins 的默认密码，星号之间的类似于一串 UUID 的随机串就是我们的密码。</p>
<pre class="lang-dart" data-nodeid="51478"><code data-language="dart">$ docker logs -f jenkins
unning from: /usr/share/jenkins/jenkins.war
webroot: EnvVars.masterEnvVars.<span class="hljs-keyword">get</span>(<span class="hljs-string">"JENKINS_HOME"</span>)
<span class="hljs-number">2020</span><span class="hljs-number">-10</span><span class="hljs-number">-31</span> <span class="hljs-number">16</span>:<span class="hljs-number">13</span>:<span class="hljs-number">06.472</span>+<span class="hljs-number">0000</span> [id=<span class="hljs-number">1</span>]	INFO	org.eclipse.jetty.util.log.Log#initialized: Logging initialized @<span class="hljs-number">292</span>ms to org.eclipse.jetty.util.log.JavaUtilLog
<span class="hljs-number">2020</span><span class="hljs-number">-10</span><span class="hljs-number">-31</span> <span class="hljs-number">16</span>:<span class="hljs-number">13</span>:<span class="hljs-number">06.581</span>+<span class="hljs-number">0000</span> [id=<span class="hljs-number">1</span>]	INFO	winstone.Logger#logInternal: Beginning extraction from war file
<span class="hljs-number">2020</span><span class="hljs-number">-10</span><span class="hljs-number">-31</span> <span class="hljs-number">16</span>:<span class="hljs-number">13</span>:<span class="hljs-number">08.369</span>+<span class="hljs-number">0000</span> [id=<span class="hljs-number">1</span>]	WARNING	o.e.j.s.handler.ContextHandler#setContextPath: Empty contextPath
... 省略部分启动日志
Jenkins initial setup <span class="hljs-keyword">is</span> required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup <span class="hljs-keyword">is</span> required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

fb3499944e4845bba9d4b7d9eb4e3932

This may also be found at: /<span class="hljs-keyword">var</span>/jenkins_home/secrets/initialAdminPassword
*************************************************************
*************************************************************
*************************************************************
This may also be found at: /<span class="hljs-keyword">var</span>/jenkins_home/secrets/initialAdminPassword
<span class="hljs-number">2020</span><span class="hljs-number">-10</span><span class="hljs-number">-31</span> <span class="hljs-number">16</span>:<span class="hljs-number">17</span>:<span class="hljs-number">07.577</span>+<span class="hljs-number">0000</span> [id=<span class="hljs-number">28</span>]	INFO	jenkins.InitReactorRunner$<span class="hljs-number">1</span>#onAttained: Completed initialization
<span class="hljs-number">2020</span><span class="hljs-number">-10</span><span class="hljs-number">-31</span> <span class="hljs-number">16</span>:<span class="hljs-number">17</span>:<span class="hljs-number">07.589</span>+<span class="hljs-number">0000</span> [id=<span class="hljs-number">21</span>]	INFO	hudson.WebAppMain$<span class="hljs-number">3</span>#run: Jenkins <span class="hljs-keyword">is</span> fully up and running
</code></pre>














<p data-nodeid="54903" class="">之后，我们通过访问 <a href="http://localhost:8888" data-nodeid="54907">http://localhost:8888</a> 就可以访问到 Jenkins 服务了。</p>


<p data-nodeid="54211" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05giAJYDhAADCpDZzl2M065.png" alt="Drawing 2.png" data-nodeid="54220"></p>
<div data-nodeid="54212"><p style="text-align:center">图3 Jenkins 登录界面</p></div>





<p data-nodeid="55578">然后将日志中的密码粘贴到密码框即可，之后 Jenkins 会自动初始化，我们根据引导，安装推荐的插件即可。</p>
<p data-nodeid="55579" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05hSAHd7VAAEpFeu4qfY218.png" alt="Drawing 3.png" data-nodeid="55584"></p>
<div data-nodeid="55580"><p style="text-align:center">图4 Jenkins 引导页面</p></div>





<p data-nodeid="56589">选择好安装推荐的插件后，Jenkins 会自动开始初始化一些常用插件。界面如下：</p>
<p data-nodeid="56590" class=""><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05h-AUSiFAAEAxHFCt30058.png" alt="Drawing 4.png" data-nodeid="56595"></p>
<div data-nodeid="56591"><p style="text-align:center">图5 Jenkins 插件初始化</p></div>





<p data-nodeid="57262">插件初始化完后，创建管理员账户和密码，输入用户名、密码和邮箱等信息，然后点击保存并完成即可。</p>
<p data-nodeid="58280"><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05iaANJXNAABXftlfsvQ115.png" alt="Drawing 5.png" data-nodeid="58284"></p>
<div data-nodeid="58281" class=""><p style="text-align:center">图6 Jenkins 创建管理员</p></div>





<p data-nodeid="57936">这里，确认 Jenkins 地址，我们就可以进入到 Jenkins 主页了。</p>
<p data-nodeid="58619"><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05i2AR2lRAADHULl7Ysk145.png" alt="Drawing 6.png" data-nodeid="58623"></p>
<div data-nodeid="58620" class=""><p style="text-align:center">图7 Jenkins 主页</p></div>





<p data-nodeid="59602">然后在系统管理 -&gt; 插件管理 -&gt; 可选插件处，搜索 GitLab 和 Docker ，分别安装相关插件即可，以便我们的 Jenkins 服务和 GitLab 以及 Docker 可以更好地交互。</p>
<p data-nodeid="60097" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05j2AfmTUAAGVJGSmG1g804.png" alt="Drawing 7.png" data-nodeid="60101"><br>
<img src="https://s0.lgstatic.com/i/image/M00/6F/35/Ciqc1F-05kKAMr27AAGh4SQTNsU299.png" alt="Drawing 8.png" data-nodeid="60105"></p>
<div data-nodeid="60098"><p style="text-align:center">图8 Jenkins 插件安装</p></div>










<p data-nodeid="40656">等待插件安装完成， 重启 Jenkins ，我们的 Jenkins 环境就准备完成了。</p>
<p data-nodeid="40657">现在，我们的 Docker+Jenkins+GitLab 环境已经准备完成，后面只需要把我们的代码推送到 GitLab 中，并做相关的配置即可实现推送代码自动构建镜像和发布。</p>
<h3 data-nodeid="40658">结语</h3>
<p data-nodeid="40659">Docker 的出现解决了 CI/CD 流程中的各种问题，Docker 交付的镜像不仅包含应用程序，也包含了应用程序的运行环境，这很好地解决了开发和线上环境不一致问题。同时 Docker 的出现也极大地提升了 CI/CD 的构建效率，我们仅仅需要编写一个 Dockerfile 并将 Dockerfile 提交到我们的代码仓库即可快速构建出我们的应用，最后，当我们构建好 Docker 镜像后 Docker 可以帮助我们快速发布及更新应用。</p>
<p data-nodeid="40660">那么，你知道 Docker 还可以帮助我们解决 CI/CD 流程中的哪些问题吗？</p>
<p data-nodeid="40661">下一讲，我将为你讲解 CI/CD 实战，利用我们准备好的环境自动构建和发布应用。</p>

---

### 精选评论

##### *锋：
> 我们用了docker和所谓的CI/CD，同样需要面对代码合并问题。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 多人开发代码合并问题确实是无法避免的，只能做好项目规范，大家够遵守规范，这样可以尽量减轻代码合并的工作量

