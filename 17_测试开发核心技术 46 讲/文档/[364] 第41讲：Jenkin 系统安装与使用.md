<p>本课时我们开始进入 Jenkins 搭建与配置章节。</p>
<p>首先，我们来看一下 Jenkins 是如何进行部署的，进入 Jenkins 官方网站（Jenkins.io），可以看到 Jenkins 是一个开源的领先的持续集成平台，可以帮你完成构建、部署和自动化工作。</p>
<p>我们可以打开官方文档链接，点开文档你可以看到它包括了入门使用相关内容，包括怎样进行安装，使用，pipeline、blue ocean 以及相关的各种配置。</p>
<p>然后点击 installing jenkins 就可以找到相关的官方说明，比如机器的配置建议，通常建议是 50G 的硬盘加 1G 的内存，但如果你的公司使用场景比较复杂，建议你可以配置高一些，比如至少是双核 CPU，内存 4~8G。选用更好的配置能够对将来 Jenkins 的运行速度有很好的帮助。</p>
<p>安装 Jenkins，第一个办法是官方推荐使用的 Docker，我个人也建议你使用 Docker 来进行部署。关于 Docker 的具体部署，你可以看一下官方文档，下面有对应命令、细节等内容。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/Ciqc1F6-byKALv2EAAR1VSL27rM786.png" alt="image (3).png"></p>
<p>接下来，我来教你几个简单的安装方法。</p>
<h4>Docker 部署</h4>
<p>docker 部署的方式可参考我图中提到的命令，你也可以用本地的文件进行影射，但是直接使用 -v 映射本地目录会提示错误，需要配置权限才可以，所以我推荐你用 docker 文件卷这个办法来进行安装。</p>
<p>解决了上面的问题就可以开始执行了。执行之后，开始启动 docker，你可以使用 docker  logs -f  看一下系统的启动过程，它会实时打印 Jenkins 日志。我启动的端口是 8080，同时也开放了一个 50000 的端口，8080 是 Jenkins 对外服务的端口，而 50000 端口是 Jenkins slave 连接主服务器时需要的。</p>
<h4>手动部署</h4>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/Ciqc1F6-bzmAauztAAHn5CtWGbQ563.png" alt="image (4).png"></p>
<p>如果我不想使用 docker，可以自己独立下载 war 包，它在我刚才给你看到的官网中的最后一个下载地址，点开发现是一个通用的 Java 的 war 包，把它下载下来，放到你的 tomcat 或 jetty 下面就可以了。你可以很简单地在本地用 java-jar 直接启动，非常便捷。</p>
<p>整体上我建议你使用 Docker 进行部署，然后使用 Nginx 做反向代理，配置域名、认证等，这样整个公司便都可以使用。</p>
<h4>自定义 Jenkins</h4>
<p>等jenkins启动完成之后，这里可以看到一个页面，Jenkins 第一次安装时，会给你一个管理员密码，这个密码也是需要的，你需要把它从docker的log中找到并复制到web界面里。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/CgqCHl6-b0SARPOEAAFPx1Xi2IY790.png" alt="image (5).png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/Ciqc1F6-b02AMJb7AAGfo97c2I8514.png" alt="image (6).png"></p>
<p>接下来会进入第二步，就是自定义 Jenkins。</p>
<p>选第一个选项系统会按照一些推荐插件，你也可以对插件列表进行自定义。注意，如果你是在国内，第一次安装推荐的插件时，因为需要连接海外服务器，会非常慢，所以可能安装起来比较耗时。如果想更快地安装，你可以选择什么插件都不装，先启动起来，然后配置插件的代理地址，配完之后再去安装插件，就会快很多。两个办法取决于你的实际情况。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/Ciqc1F6-b1yARPpFAAGQtQWZq-0515.png" alt="image (7).png"></p>
<p>配完之后就到了这个界面，在这个界面里，最后安装完成之后就会进入 Jenkins 的首界面，这就是它的整个安装流程。</p>
<h4>案例演示</h4>
<p>既然 Jenkins 已经搭建完成，我就可以创建 job 了，这里取名为 demo，但有很多插件都没有装，目前只有一个 freestyle project，我们选它就可以了。点击 ok，选完之后就进入项目界面，比如霍格沃兹测试学院演练项目，我们随便创建一个，然后会重点分这样几个区域：上面的是关于项目的描述以及项目的基础配置。</p>
<p>下面是源代码控制，你只有装了git 才会有 git 相关配置，所以这里我们需要安装插件。第三个是构建，如何构建，我希望设置为周期构建，以及根据代码变进行构建。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0A/C5/Ciqc1F6-b2WAe1vsAABljoYJtk4105.png" alt="image (8).png"></p>
<p>构建之后我们需要将文件进行存档，然后点击 save 就可以了。</p>
<p>这就是一个简单的项目，我可以先触发一下 build now，它就可以运行起来。点击进来你可以看到它的执行结果，在 console output 里面，我所配置的命令现在就已经完成了，你可以看到里面有 demo，有 hello from hogwarts，这就是这个项目的基本配置。</p>

---

### 精选评论


