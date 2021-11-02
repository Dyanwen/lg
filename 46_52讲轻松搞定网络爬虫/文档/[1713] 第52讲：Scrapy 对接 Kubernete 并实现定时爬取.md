<p data-nodeid="719">在上一节我们了解了如何制作一个 Scrapy 的 Docker 镜像，本节课我们来介绍下如何将镜像部署到 Kubernetes 上。</p>



<h3 data-nodeid="917" class="">Kubernetes</h3>

<p data-nodeid="5">Kubernetes 是谷歌开发的，用于自动部署，扩展和管理容器化应用程序的开源系统，其稳定性高、扩展性好，功能十分强大。现在业界已经越来越多地采用 Kubernetes 来部署和管理各个项目，</p>
<p data-nodeid="6">如果你还不了解 Kubernetes，可以参考其官方文档来学习一下： <a href="https://kubernetes.io/" data-nodeid="49">https://kubernetes.io/</a>。</p>
<h3 data-nodeid="1115" class="">准备工作</h3>

<p data-nodeid="8">如果我们需要将上一节的镜像部署到 Kubernetes 上，则首先需要我们有一个 Kubernetes 集群，同时需要能使用 kubectl 命令。</p>
<p data-nodeid="9">Kubernetes 集群可以自行配置，也可以使用各种云服务提供的集群，如阿里云、腾讯云、Azure 等，另外还可以使用 Minikube 等来快速搭建，当然也可以使用 Docker 本身提供的 Kubernetes 服务。</p>
<p data-nodeid="1501">比如我这里就直接使用了 Docker Desktop 提供的 Kubernetes 服务，勾选 Enable 直接开启即可。</p>
<p data-nodeid="1502" class=""><img src="https://s0.lgstatic.com/i/image/M00/40/A7/Ciqc1F8zZ0eAS3RkAADg5KBv-5U920.png" alt="image (13).png" data-nodeid="1510"></p>


<p data-nodeid="1715" class="">kubectl 是用来操作 Kubernetes 的命令行工具，可以参考 <a href="https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/" data-nodeid="1719">https://kubernetes.io/zh/docs/tasks/tools/install-kubectl/</a> 来安装。</p>

<p data-nodeid="13">如果以上都安装好了，可以运行下 kubectl 命令验证下能否正常获取节点信息：</p>
<pre class="lang-java" data-nodeid="2540"><code data-language="java">kubectl get nodes 
</code></pre>




<p data-nodeid="15">运行结果类似如下：</p>
<pre class="lang-java" data-nodeid="2745"><code data-language="java">NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; STATUS &nbsp; ROLES &nbsp;  AGE &nbsp; VERSION 
docker-desktop &nbsp; Ready &nbsp;  master &nbsp; <span class="hljs-number">75d</span> &nbsp; v1.<span class="hljs-number">16.6</span>-beta.<span class="hljs-number">0</span> 
</code></pre>

<h3 data-nodeid="2950" class="">部署</h3>

<p data-nodeid="18">要部署的话我们需要先创建一个命名空间 Namespace，这里直接使用 kubectl 命令创建即可，Namespace 的名称这里我们取名为 crawler。</p>
<p data-nodeid="19">创建命令如下：</p>
<pre class="lang-java" data-nodeid="3156"><code data-language="java">kubectl create namespace crawler 
</code></pre>

<p data-nodeid="21">运行结果如下：</p>
<pre class="lang-java" data-nodeid="3361"><code data-language="java">namespace/crawler created 
</code></pre>

<p data-nodeid="23">如果出现上述结果就说明命名空间创建成功了。接下来我们就需要把 Docker 镜像部署到这个 Namespace 下面了。<br>
Kubernetes 里面的资源是用 yaml 文件来定义的，如果要部署一次性任务或者为我们提供服务可以使用 Deployment，更多详情可以参考 Kubernetes 对于 Deployment 的说明： <a href="https://kubernetes.io/docs/concepts/workloads/controllers/deployment/" data-nodeid="74">https://kubernetes.io/docs/concepts/workloads/controllers/deployment/</a>。</p>
<p data-nodeid="24">新建 deployment.yaml 文件如下：</p>
<pre class="lang-yaml" data-nodeid="6641"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span> 
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span> 
<span class="hljs-attr">metadata:</span> 
  <span class="hljs-attr">name:</span> <span class="hljs-string">crawler-quotes</span> 
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">crawler</span> 
  <span class="hljs-attr">labels:</span> 
 &nbsp;  <span class="hljs-attr">app:</span> <span class="hljs-string">crawler-quotes</span> 
<span class="hljs-attr">spec:</span> 
  <span class="hljs-attr">replicas:</span> <span class="hljs-number">1</span> 
  <span class="hljs-attr">selector:</span> 
 &nbsp;  <span class="hljs-attr">matchLabels:</span> 
 &nbsp; &nbsp;  <span class="hljs-attr">app:</span> <span class="hljs-string">crawler-quotes</span> 
  <span class="hljs-attr">template:</span> 
 &nbsp;  <span class="hljs-attr">metadata:</span> 
 &nbsp; &nbsp;  <span class="hljs-attr">labels:</span> 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-attr">app:</span> <span class="hljs-string">crawler-quotes</span> 
 &nbsp;  <span class="hljs-attr">spec:</span> 
 &nbsp; &nbsp;  <span class="hljs-attr">containers:</span> 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">crawler-quotes</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-attr">image:</span> <span class="hljs-string">germey/quotes</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-attr">env:</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">MONGO_URI</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-attr">value:</span> <span class="hljs-string">&lt;mongo&gt;</span> 
</code></pre>
















<p data-nodeid="7159">这里我们就可以按照 Deployment 的规范声明一个 yaml 文件了，指定 namespace 为 crawler，并指定 container 的 image 为我们已经 Push 到 Docker Hub 的镜像 germey/quotes，另外通过 env 指定了环境变量，注意这里需要将 <code data-backticks="1" data-nodeid="7162">&lt;mongo&gt;</code> 替换成一个有效的 MongoDB 连接字符串，如一个远程 MongoDB 服务。</p>
<p data-nodeid="7160">接下来我们只需要使用 kubectl 命令即可应用该部署：</p>



<pre class="lang-java" data-nodeid="6846"><code data-language="java">kubectl apply -f deployment.yaml 
</code></pre>

<p data-nodeid="28">运行完毕之后会提示类似如下结果：</p>
<pre class="lang-java" data-nodeid="7369"><code data-language="java">deployment.apps/crawler-quotes created 
</code></pre>

<p data-nodeid="7574">这样就说明部署成功了。如果 MongoDB 服务能够正常连接的话，这个爬虫就会运行并将结果存储到 MongoDB 中。</p>
<p data-nodeid="7575">另外我们还可以通过命令行或者 Kubernetes 的 Dashboard 查看部署任务的运行状态。</p>

<p data-nodeid="31">如果我们想爬虫定时运行的话，可以借助于 Kubernetes 提供的 cronjob 来将爬虫配置为定时任务，其运行模式就类似于 crontab 命令一样，详细用法可以参考： <a href="https://kubernetes.io/zh/docs/tasks/job/automated-tasks-with-cron-jobs/" data-nodeid="89">https://kubernetes.io/zh/docs/tasks/job/automated-tasks-with-cron-jobs/</a>。</p>
<p data-nodeid="32">可以新建 cronjob.yaml，内容如下：</p>
<pre class="lang-java" data-nodeid="7782"><code data-language="java">apiVersion: batch/v1beta1 
kind: CronJob 
metadata: 
  name: crawler-quotes 
  namespace: crawler 
spec: 
  schedule: <span class="hljs-string">"0 */1 * * *"</span> 
  jobTemplate: 
 &nbsp;  spec: 
 &nbsp; &nbsp;  template: 
 &nbsp; &nbsp; &nbsp;  spec: 
 &nbsp; &nbsp; &nbsp; &nbsp;  restartPolicy: OnFailure 
 &nbsp; &nbsp; &nbsp; &nbsp;  containers: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  - name: crawler-quotes 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  image: germey/quotes 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  env: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  - name: MONGO_URI 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  value: &lt;mongo&gt; 
</code></pre>

<p data-nodeid="34">注意到这里 kind 我们不再使用 Deployment，而是改成了 CronJob，代表定时任务。 <code data-backticks="1" data-nodeid="93">spec.schedule</code> 里面定义了 crontab 格式的定时任务配置，这里代表每小时运行一次。其他的配置基本一致，同样注意这里需要将 <code data-backticks="1" data-nodeid="95">&lt;mongo&gt;</code> 替换成一个有效的 MongoDB 连接字符串，如一个远程 MongoDB 服务。<br>
接下来我们只需要使用 kubectl 命令即可应用该部署：</p>
<pre class="lang-java" data-nodeid="7987"><code data-language="java">kubectl apply -f cronjob.yaml 
</code></pre>

<p data-nodeid="36">运行完毕之后会提示类似如下结果：</p>
<pre class="lang-java" data-nodeid="8192"><code data-language="java">cronjob.batch/crawler-quotes created 
</code></pre>

<p data-nodeid="38">出现这样的结果这就说明部署成功了，这样这个爬虫就会每小时运行一次，并将数据存储到指定的 MongoDB 数据库中。</p>
<h3 data-nodeid="513" class="">总结</h3>


<p data-nodeid="40">以上我们就简单介绍了下 Kubernetes 部署爬虫的基本操作，Kubernetes 非常复杂，需要学习的内容很多，我们这一节介绍的只是冰山一角，还有更多的内容等待你去探索。</p>

---

### 精选评论

##### **6707：
> 完结撒花！

##### *戴：
> 终于看完了，佩服大佬

