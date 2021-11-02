<p data-nodeid="16424">通过前面的课程，相信你对 Kubernetes 中的对象有了很多了解。Kubernetes 是一个强大的容器调度系统，你可以通过一些声明式的定义，很方便地在 Kubernetes 中部署业务。</p>



<p data-nodeid="17294">现在你一定很想尝试在 Kubernetes 中部署一个稍微复杂的系统，比如下面这个典型的三层架构：前端、后端和数据层。</p>
<p data-nodeid="17295" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/EA/Ciqc1F9sPOKACHGPAADXdRLPJwo750.png" alt="image (7).png" data-nodeid="17303"><br>
（<a href="https://docs.bitnami.com/tutorials/_next/static/images/three-tier-kubernetes-architecture-28861dab09dbb6c2dd6ddb986f3a42d4.png.webp" data-nodeid="17310">https://docs.bitnami.com/tutorials/_next/static/images/three-tier-kubernetes-architecture-28861dab09dbb6c2dd6ddb986f3a42d4.png.webp</a>）</p>




<p data-nodeid="15767">也许你内心里已经开始跃跃欲试，但是转念一想，事情好像没那么简单啊。你感觉到这里面要写一堆的 YAML 文件，每一层架构都至少要写 2 个 YAML 文件，写的时候还需要理解各种对象、注意各种缩进的层级关系、还要设置一堆配置参数等。这对于刚接触 Kubernetes 的人来说是一个很高的门槛，简直无法跨越。同时，对于业务交付的同学，也将是一个大灾难，因为如果缺少任何一个 YAML 文件，都会导致整个系统无法正常工作。</p>
<p data-nodeid="15768">看到这里，你也许会想到能不能通过一种包的方式进行管理呢，类似于 Node.js 通过 npm 来管理包，Debian 系统通过 dpkg 来管理包，而 Python 通过 pip 来管理包。</p>
<p data-nodeid="15769">那么在 Kubernetes 中， 这个答案就是<strong data-nodeid="15852">Helm</strong>。</p>
<p data-nodeid="15770">Helm 降低了使用 Kubernetes 的门槛，你无须从头开始编写应用部署文件，甚至都不需要了解 Kubernetes 中的各个对象以及相应的 YAML 语义。直接通过 Helm 就可以在自己的 Kubernetes 中一键部署需要的应用。</p>
<p data-nodeid="15771">对于开发者而言，通过 Helm 进行打包，管理应用内部的各种依赖关系，极其方便。并且可以借助于版本化的概念，方便自身的迭代和分发。除此之外，Helm 还有一些其他强大的功能，请跟我一起开始今天 Helm 的学习之旅。</p>
<p data-nodeid="15772">作为 CNCF 社区官方的项目，Helm 在今年四月份已经宣布毕业，这进一步彰显了 Helm 的市场接受度、生产环境采用率以及技术影响力。</p>
<p data-nodeid="15773">我们先来理解下 Helm 中几个重要的概念。</p>
<h3 data-nodeid="18198" class="">Helm 中的几个概念</h3>


<p data-nodeid="15775">在 Helm 中，有三个非常核心的概念—— Chart、Config 和 Release。</p>
<p data-nodeid="15776"><strong data-nodeid="15867">Chart 可以理解成应用的安装包</strong>，<strong data-nodeid="15868">通常包含了一组我们在 Kubernetes 要部署的 YAML 文件</strong>。一个 Chart 就是一个目录，我们可以将这个目录进行压缩打包，比如打包成 some-chart-version.tgz 类型的压缩文件，方便传输和存储。</p>
<p data-nodeid="15777">每个这样的 Chart 包内都必须有一个 Chart.yaml 文件，类似这样：</p>
<pre class="lang-yaml" data-nodeid="15778"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">redis</span>
<span class="hljs-attr">version:</span> <span class="hljs-number">11.0</span><span class="hljs-number">.0</span>
<span class="hljs-attr">appVersion:</span> <span class="hljs-number">6.0</span><span class="hljs-number">.8</span>
<span class="hljs-attr">description:</span> <span class="hljs-string">Open</span> <span class="hljs-string">source,</span> <span class="hljs-string">advanced</span> <span class="hljs-string">key-value</span> <span class="hljs-string">store.</span> <span class="hljs-string">It</span> <span class="hljs-string">is</span> <span class="hljs-string">often</span> <span class="hljs-string">referred</span> <span class="hljs-string">to</span> <span class="hljs-string">as</span> <span class="hljs-string">a</span> <span class="hljs-string">data</span> <span class="hljs-string">structure</span> <span class="hljs-string">server</span> <span class="hljs-string">since</span> <span class="hljs-string">keys</span> <span class="hljs-string">can</span> <span class="hljs-string">contain</span> <span class="hljs-string">strings,</span> <span class="hljs-string">hashes,</span> <span class="hljs-string">lists,</span> <span class="hljs-string">sets</span> <span class="hljs-string">and</span> <span class="hljs-string">sorted</span> <span class="hljs-string">sets.</span>
<span class="hljs-attr">keywords:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">redis</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">keyvalue</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">database</span>
<span class="hljs-attr">home:</span> <span class="hljs-string">https://github.com/bitnami/charts/tree/master/bitnami/redis</span>
<span class="hljs-attr">icon:</span> <span class="hljs-string">https://bitnami.com/assets/stacks/redis/img/redis-stack-220x234.png</span>
<span class="hljs-attr">sources:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">https://github.com/bitnami/bitnami-docker-redis</span>
  <span class="hljs-bullet">-</span> <span class="hljs-string">http://redis.io/</span>
<span class="hljs-attr">maintainers:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">Bitnami</span>
    <span class="hljs-attr">email:</span> <span class="hljs-string">containers@bitnami.com</span>
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">desaintmartin</span>
    <span class="hljs-attr">email:</span> <span class="hljs-string">cedric@desaintmartin.fr</span>
<span class="hljs-attr">engine:</span> <span class="hljs-string">gotpl</span>
<span class="hljs-attr">annotations:</span>
  <span class="hljs-attr">category:</span> <span class="hljs-string">Database</span>

</code></pre>
<p data-nodeid="18642">这个 Chart.yaml 主要用来描述该 Chart 的名字、版本号，以及关键字等信息。</p>
<p data-nodeid="18643">有了这个 Chart，我们就可以在 Kubernetes集群中部署了。每一次的安装部署，我们称为一个 Release。在同一个 Kubernetes 集群中，可以有多个 Release。你可以将 Release 理解成是 Chart 包部署后的一个 Chart（应用）实例。</p>

<blockquote data-nodeid="15780">
<p data-nodeid="15781">注： 这个 Release 和我们通常概念中理解的“版本”有些差异。</p>
</blockquote>
<p data-nodeid="15782">同时，为了能够让 Chart 实现参数可配置，即 Config，<strong data-nodeid="15883">我们在每个 Chart 包内还有一个 values.yaml 文件</strong>，<strong data-nodeid="15884">用来记录可配置的参数和其默认值</strong>。在每个 Release 中，我们也可以指定自己的 values.yaml 文件用来覆盖默认的配置。</p>
<p data-nodeid="15783">此外，我们还需要统一存放这些 Chart，即 Repository（仓库）。这个概念和我们以前的 yum repo 是一样的。本质上来说，<strong data-nodeid="15898">Repository 就是一个 Web 服务器</strong>，<strong data-nodeid="15899">不仅保存了一系列的 Chart 软件包让用户下载安装使用</strong>，<strong data-nodeid="15900">还有对应的清单文件以供查询</strong>。</p>
<p data-nodeid="15784">通过 Helm，我们可以同时使用和管理多个不同的 Repository，就像我们可以给 yum 配置多个源一样</p>
<p data-nodeid="15785">掌握了这些概念，我们现在再拉远视角，整体来看一下。</p>
<h3 data-nodeid="19088" class="">Helm 的安装及架构组成</h3>

<p data-nodeid="15787">首先，Helm 的安装非常简单，你可以通过如下的命令安装最新版本的 Helm 可执行文件：</p>
<pre class="lang-shell" data-nodeid="15788"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash</span>
<span class="hljs-meta">  %</span><span class="bash"> Total    % Received % Xferd  Average Speed   Time    Time     Time  Current</span>
                                 Dload  Upload   Total   Spent    Left  Speed
100 11213  100 11213    0     0  12109      0 --:--:-- --:--:-- --:--:-- 12096
Downloading https://get.helm.sh/helm-v3.3.1-darwin-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
Password:
helm installed into /usr/local/bin/helm
</code></pre>
<p data-nodeid="15789">当然你也可以去<a href="https://helm.sh/docs/intro/install/" data-nodeid="15908">官方的文档</a>里选择合适自己的安装方式。<br>
安装完成后，我们来查看下 Helm 的版本：</p>
<pre class="lang-shell" data-nodeid="15790"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm version</span>
version.BuildInfo{Version:"v3.3.1", GitCommit:"249e5215cde0c3fa72e27eb7a30e8d55c9696144", GitTreeState:"clean", GoVersion:"go1.14.7"}
</code></pre>
<p data-nodeid="15791">可以看到目前我们安装的版本是<code data-backticks="1" data-nodeid="15913">v3.3.1</code>，这是 Helm 3 的一个版本。<br>
其实在 Helm 3 之前，还有 Helm 2，目前还有不少人在继续使用 Helm 2 的版本。现在我们来简单了解下 Helm 2 和 Helm 3 。</p>
<h4 data-nodeid="19976" class="">Helm 2</h4>


<p data-nodeid="20860">Helm 2 于 2015 年在 KubeCon 上亮相。其架构如下图所示，是个常见的 CS 架构，由 Helm Client 和 Tiller 两部分组成。</p>
<p data-nodeid="20861" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/F6/CgqCHl9sPQ2AWA7kAAKDpOtvIDk020.png" alt="image (8).png" data-nodeid="20869"><br>
（<a href="https://medium.com/dwarves-foundation/kubernetes-helm-101-78f70eeb0d1" data-nodeid="20874">https://medium.com/dwarves-foundation/kubernetes-helm-101-78f70eeb0d1</a>）</p>




<p data-nodeid="15796">Tiller 是 Helm 的服务端， 用于接收来自 Helm Client 的请求，主要负责部署 Helm Chart、管理 Release（比如升级、删除、回滚等操作），以及在 Kubernetes 集群中创建应用。TIller 通常部署在 Kubernetes 中，直接通过<code data-backticks="1" data-nodeid="15928">helm init</code>就可以在 Kuberentes 集群中拉起服务。</p>
<p data-nodeid="15797">Helm Client 主要负责跟用户进行交互，通过命令行就可以完成 Chart 的安装、升级、删除等操作。Helm Client 可以从公有或者私有的 Helm 仓库中拉取 Chart 包，通过用户指定的 Config，就可以直接传给后端的 Tiller 服务，使之在集群内进行部署。</p>
<p data-nodeid="15798">这里，我们可以看到 Tiller 在 Helm 2 和 Kubernetes 集群中间其实起到了一个中转的作用，因此 Tiller 实际上是代替用户在 Kubernetes 集群中执行操作，所以 Tiller 在运行中往往需要一个很高的权限。这里 Tiller 存在着两大安全风险：</p>
<ol data-nodeid="15799">
<li data-nodeid="15800">
<p data-nodeid="15801">要对外暴露自身的端口；</p>
</li>
<li data-nodeid="15802">
<p data-nodeid="15803">自身运行过程中，跟 Kubernetes 交互需要很高的权限，这样才可以在 Kuberentes 中创建、删除各种各样的资源。</p>
</li>
</ol>
<p data-nodeid="15804">当然了，之所以有这种架构设计是由于在 Helm 2 开发的时候，Kubernetes 的 RBAC 体系还没有建立完成。Kubernetes 社区直到 2017 年 10 月份 v1.8 版本才默认采用了 RBAC 权限体系。随着 Kubernetes 的各项功能日益完善，主要是 RBAC 能力，Tiller 这个组件存在的必要性进一步降低，社区也在一直想着怎么去解决 TIller 的安全问题。</p>
<p data-nodeid="15805">我们再来看看 Helm 3。</p>
<h4 data-nodeid="21326" class="">Helm 3</h4>

<p data-nodeid="15807">Helm 3 是在 Helm 2 之上的一次大更改，于 2019 年 11 月份正式推出。相比较于 Helm 2， Helm 3 简单了很多，移除了 Tiller，只剩下一个 Helm Client。在使用中，你只需要这一个二进制可执行文件即可，使用你当前用的 kubeconfig 文件，即可在 Kubernetes 集群中部署 Chart、管理 Release。</p>
<p data-nodeid="15808">至此，Helm 2 开始要退出历史的舞台了。目前 Helm 社区官方也逐步开始暂停维护 Helm 2，最终的 Bugfix 修复截止到今年的 8 月份，版本号是  v2.16.10，而安全相关的 patch 会继续支持，直到 今年的 11 月份。</p>
<p data-nodeid="15809">如果你还在使用 Helm 2，建议你尽快切换到 Helm 3 上来。Helm 社区也提供了相关的插件，帮助你从 Helm 2 迁移到 Helm 3 中，可以参考这篇 Helm 的<a href="https://helm.sh/docs/topics/v2_v3_migration/" data-nodeid="15942">官方迁移文档</a>，这里我就不再赘述了。</p>
<p data-nodeid="15810">通过这个版本迭代的过程，你应该也清楚了 Helm 的原理，对它更加熟悉了，接下来我们一起动手实践看看。</p>
<h3 data-nodeid="21778" class="">如何创建和部署 Helm Chart</h3>

<p data-nodeid="15812">创建一个 Chart 很简单，通过<code data-backticks="1" data-nodeid="15947">helm create</code>的命令就可以创建一个 Chart 模板出来：</p>
<pre class="lang-shell" data-nodeid="15813"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm create hello-world</span>
Creating hello-world
</code></pre>
<p data-nodeid="15814">我们现在来看看这个 Chart 的目录里面有什么：</p>
<pre class="lang-shell" data-nodeid="15815"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> tree ./hello-world</span>
./hello-world
├── Chart.yaml
├── charts
├── templates
│&nbsp; &nbsp;├── NOTES.txt
│&nbsp; &nbsp;├── _helpers.tpl
│&nbsp; &nbsp;├── deployment.yaml
│&nbsp; &nbsp;├── hpa.yaml
│&nbsp; &nbsp;├── ingress.yaml
│&nbsp; &nbsp;├── service.yaml
│&nbsp; &nbsp;├── serviceaccount.yaml
│&nbsp; &nbsp;└── tests
│&nbsp; &nbsp; &nbsp; &nbsp;└── test-connection.yaml
└── values.yaml

3 directories, 10 files
</code></pre>
<p data-nodeid="22230">这里面包含了我们上面讲的 Chart.yaml 和 values.yaml。</p>
<p data-nodeid="22231">先看看这里 Chart.yaml 的内容：</p>

<pre class="lang-yaml" data-nodeid="15817"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v2</span>
<span class="hljs-attr">name:</span> <span class="hljs-string">hello-world</span>
<span class="hljs-attr">description:</span> <span class="hljs-string">A</span> <span class="hljs-string">Helm</span> <span class="hljs-string">chart</span> <span class="hljs-string">for</span> <span class="hljs-string">Kubernetes</span>

<span class="hljs-comment"># A chart can be either an 'application' or a 'library' chart.</span>
<span class="hljs-comment">#</span>
<span class="hljs-comment"># Application charts are a collection of templates that can be packaged into versioned archives</span>
<span class="hljs-comment"># to be deployed.</span>
<span class="hljs-comment">#</span>
<span class="hljs-comment"># Library charts provide useful utilities or functions for the chart developer. They're included as</span>
<span class="hljs-comment"># a dependency of application charts to inject those utilities and functions into the rendering</span>
<span class="hljs-comment"># pipeline. Library charts do not define any templates and therefore cannot be deployed.</span>
<span class="hljs-attr">type:</span> <span class="hljs-string">application</span>

<span class="hljs-comment"># This is the chart version. This version number should be incremented each time you make changes</span>
<span class="hljs-comment"># to the chart and its templates, including the app version.</span>
<span class="hljs-comment"># Versions are expected to follow Semantic Versioning (https://semver.org/)</span>
<span class="hljs-attr">version:</span> <span class="hljs-number">0.1</span><span class="hljs-number">.0</span>

<span class="hljs-comment"># This is the version number of the application being deployed. This version number should be</span>
<span class="hljs-comment"># incremented each time you make changes to the application. Versions are not expected to</span>
<span class="hljs-comment"># follow Semantic Versioning. They should reflect the version the application is using.</span>
<span class="hljs-attr">appVersion:</span> <span class="hljs-number">1.16</span><span class="hljs-number">.0</span>
</code></pre>
<p data-nodeid="15818">可以看到，这里 Chart 的名字就是我们 Chart 所在目录的名字。你在使用的时候，就可以在这个文件里面添加 keyword，以及 annotation，方便后续查找和分类。<br>
values.yaml 里面包含了一些可配置的参数，你可以根据自己的需要进行添加，或者修改默认值等。</p>
<p data-nodeid="15819">这个目录下面有个<code data-backticks="1" data-nodeid="15957">charts</code>文件夹，主要用来放置一些子 Chart 包，也可以是 tar 包、 Chart 目录。你可以根据自己的场景，决定需不需要放置子 Chart 包。</p>
<p data-nodeid="15820">这个目录里面还有个<code data-backticks="1" data-nodeid="15960">templates</code>文件夹，这个文件夹里面的文件是一些模板文件，Helm 会根据<code data-backticks="1" data-nodeid="15962">values.yaml</code>的参数进行渲染，然后在 Kubernetes 集群中创建出来。</p>
<p data-nodeid="15821">你可以根据自己的需要，在这个<code data-backticks="1" data-nodeid="15965">templates</code>目录下，添加和更改 yaml 文件。</p>
<p data-nodeid="15822">你这样创建好了一个 Chart 包后，如果想要更改其中一些参数，可以将它们放到其他的文件里面，比如 myvalues.yaml 文件。</p>
<p data-nodeid="15823">然后通过<code data-backticks="1" data-nodeid="15969">helm install</code>命令，你就可以在 Kubernetes 集群里面部署 hello-world 这个 Chart 包：</p>
<pre class="lang-shell" data-nodeid="15824"><code data-language="shell">helm install -f myvalues.yaml hello-world ./hello-world
</code></pre>
<p data-nodeid="15825">或者，你可以通过添加 Repository 直接使用别人的 Chart，使用<code data-backticks="1" data-nodeid="15972">helm search</code>来查找自己想要的 Chart：</p>
<pre class="lang-shell" data-nodeid="24536"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm repo add brigade https://brigadecore.github.io/charts</span>
"brigade" has been added to your repositories
<span class="hljs-meta">$</span><span class="bash"> helm search repo brigade</span>
NAME                          CHART VERSION APP VERSION DESCRIPTION
brigade/brigade               1.3.2         v1.2.1      Brigade provides event-driven scripting of Kube...
brigade/brigade-github-app    0.4.1         v0.2.1      The Brigade GitHub App, an advanced gateway for...
brigade/brigade-github-oauth  0.2.0         v0.20.0     The legacy OAuth GitHub Gateway for Brigade
brigade/brigade-k8s-gateway   0.1.0                     A Helm chart for Kubernetes
brigade/brigade-project       1.0.0         v1.0.0      Create a Brigade project
brigade/kashti                0.4.0         v0.4.0      A Helm chart for Kubernetes
</code></pre>
<p data-nodeid="24537">更多 Helm 的使用命令，可以参考官方的<a href="https://helm.sh/docs/intro/using_helm/" data-nodeid="24544">使用文档</a>，使用起来非常简单就不占用篇幅了。</p>
<h3 data-nodeid="24999" class="">写在最后</h3>

<p data-nodeid="24539">目前 Helm 是 CNCF 基金会旗下已经“毕业”的独立的项目。它简化了 Kubernetes 应用的部署和管理，大大提高了效率，越来越多的人在生产环境中使用 Helm 来部署和管理应用，所以我在这里用一个课时来专门讲解它的原理和使用，想让你在使用 Kubernetes 时如虎添翼。</p>
<p data-nodeid="24540">如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论


