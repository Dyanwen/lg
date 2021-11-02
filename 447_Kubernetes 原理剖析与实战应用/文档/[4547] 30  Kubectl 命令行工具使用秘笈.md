<p data-nodeid="81559">在本课程的最后一讲，我来为你介绍一些 kubectl 使用过程中的小技巧。kubectl 是我们日常操纵整个 Kubernetes 的利器，操作方便，功能强大。接下来，我会向你介绍常用的七个功能。</p>
<h3 data-nodeid="81560">自动补全</h3>
<p data-nodeid="81561">我们可以通过如下命令进行命令行的自动补全，方便我们使用。</p>
<ul data-nodeid="81562">
<li data-nodeid="81563">
<p data-nodeid="81564">如果你使用的是 bash，可以通过如下命令：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="81565"><code data-language="java">source &lt;(kubectl completion bash) #你需要先安装 bash-completion
echo "source &lt;(kubectl completion bash)" &gt;&gt; ~/.bashrc #这样就不需要每次都 source 一下了
</code></pre>
<ul data-nodeid="81566">
<li data-nodeid="81567">
<p data-nodeid="81568">如果你使用的是 zsh，也有可用的命令：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="81569"><code data-language="java">source &lt;(kubectl completion zsh)
echo "[[ $commands[kubectl] ]] &amp;&amp; source &lt;(kubectl completion zsh)" &gt;&gt; ~/.zshrc #这样就不需要每次都 source 一下了
</code></pre>
<h3 data-nodeid="81570">命令行帮助</h3>
<p data-nodeid="81571">遇到任何关于命令行使用的问题，都可以通过“kubectl -h”命令来查看有哪些子命令、包含哪些参数、使用例子等等。</p>
<p data-nodeid="81572">这条命令也是我使用比较多的，可以帮助你更快地熟悉和了解 kubectl 的使用。</p>
<h3 data-nodeid="81573">集群管理</h3>
<p data-nodeid="81574">我们可以通过“kubectl cluster-info”命令查看整个集群信息，比如 apiserver 暴露的地址、dns 的地址、metrics-server 的地址。</p>
<p data-nodeid="81575">还可以通过“kubectl version”命令查看到 kubectl 以及 apiserver 的版本。毕竟 apiserver 的版本对整个集群至关重要，决定了各个 api 的版本、feature gate、准入控制等等，</p>
<p data-nodeid="81576">而各个 kubelet 节点的版本，你可以通过“kubectl get node”命令查看。</p>
<p data-nodeid="81577">你可以通过这些版本号了解到整个集群的版本信息，对集群的维护和升级很有帮助。</p>
<h3 data-nodeid="81578">资源查询</h3>
<p data-nodeid="81579">通过 kubectl 命令来查询集群中的资源是我们日常使用频率最高的。</p>
<p data-nodeid="81580">你可以通过 kubectl get 查询到某类资源对象，代码如下：</p>
<pre class="lang-java" data-nodeid="81581"><code data-language="java">kubectl get [资源]
</code></pre>
<p data-nodeid="81582">假如我们想要查看集群中的所有节点，可以在代码的“资源”处输入“nodes”，如下所示：</p>
<pre class="lang-java" data-nodeid="81583"><code data-language="java">kubectl get nodes
</code></pre>
<p data-nodeid="81584">这里的资源名，我们可以使用资源名称的单数，比如 node；也可以使用其复数，比如 nodes；还可以使用其缩写名。</p>
<p data-nodeid="81585">对于集群中定义的资源信息，比如资源名、对应的缩写、是否是 namespace 级别的资源，你可以通过“kubectl api-resources”命令获取。</p>
<p data-nodeid="81586">如果我们想要查询某个资源对象，我们同样可以通过“kubectl get”命令，只不过要在原先的资源名后面加上“/对象名”。如下所示：</p>
<pre class="lang-java" data-nodeid="81587"><code data-language="java">kubectl get [资源]/[对象名]
</code></pre>
<p data-nodeid="81588">还是以 node 为例，我想查询 node01 节点，就可以通过“kubectl get node/node01”命令完成。当然，不使用“/”也是允许的，代码如下所示：</p>
<pre class="lang-java" data-nodeid="81589"><code data-language="java">kubectl get [资源] [对象名]
</code></pre>
<p data-nodeid="81590">如果你想看到关于这个对象更详细的信息，你可以“kubectl describe”一下，即：</p>
<pre class="lang-java" data-nodeid="81591"><code data-language="java">kubectl describe [资源]/[对象名]
</code></pre>
<p data-nodeid="81592">对于 namesppace 级别的资源，我们只需要在上述命令后面加上“-n [命名空间]”或“--namespace [命名空间]”就可以了。代码如下所示：</p>
<pre class="lang-java" data-nodeid="81593"><code data-language="java">kubectl get [资源]/[对象名] -n [命名空间]
</code></pre>
<p data-nodeid="81594">比如：</p>
<pre class="lang-java" data-nodeid="81595"><code data-language="java">kubectl get pod pod-example -n demo
</code></pre>
<p data-nodeid="81596">如果你没有指定 namespace 的话，默认是名为 default 的命名空间。</p>
<p data-nodeid="81597">此外，如果你想要查看所有命名空间下的某类资源，可以在“资源”后面加上“--all-namespaces”。代码如下：</p>
<pre class="lang-java" data-nodeid="81598"><code data-language="java">kubectl get [资源] --all-namespaces
</code></pre>
<p data-nodeid="81599">比如：</p>
<pre class="lang-java" data-nodeid="81600"><code data-language="java">kubectl get pod --all-namespaces
</code></pre>
<h3 data-nodeid="81601">资源创建、更改、删除</h3>
<p data-nodeid="81602">你还可以通过 kubectl create 进行资源创建，代码如下：</p>
<pre class="lang-java" data-nodeid="81603"><code data-language="java">kubectl create -f demo.yaml
</code></pre>
<p data-nodeid="81604">通过在 yaml 文件中定义各种资源及对象，我们通过这条命令将其在集群中创建出来。</p>
<p data-nodeid="81605">当然，kubectl create 还提供了一些子命令，方便通过命令行直接创建资源对象。你可以通过“kubectl create -h”查看其支持的子命令。</p>
<p data-nodeid="81606">假如我们想要在命名空间 demo 下创建一个名为 sa-demo 的 ServiceAccount 对象，我们可以通过如下命令进行：</p>
<pre class="lang-java" data-nodeid="81607"><code data-language="java">kubectl create sa sa-demo -n demo
</code></pre>
<p data-nodeid="81608">通常来说，从零开始写一个 yaml 文件很难，一般我们都是找一些资源的 yaml 例子拿来自己修改下。我并不推荐自己一点点去写 yaml，效率低下，而且还会出现缩进的问题。</p>
<p data-nodeid="81609">我推荐<strong data-nodeid="81691">通过 kubectl create 的这些命令来解决</strong>。通过命令行参数，我们可以让 kubectl 帮我们自动生成一些 yaml 文件，比如你可以通过下面的命令拿到了一个 deployment 的 yaml 文件，然后就可以对这个文件进一步地修改以达到你的期望定义。</p>
<pre class="lang-java" data-nodeid="81610"><code data-language="java">kubectl create deploy my-deployment -n demo --image busybox --dry-run server -o yaml &gt; my-deployment.yaml
</code></pre>
<p data-nodeid="81611">这儿主要是用到了 dry-run 的能力。</p>
<p data-nodeid="81612">你还可以通过 kubectl edit 直接修改这些资源：</p>
<pre class="lang-java" data-nodeid="81613"><code data-language="java">kubectl edit [资源]/[对象名] -n [命名空间]
</code></pre>
<p data-nodeid="81614">如果是集群级别的资源对象，那么代码中就不用加“-n”了。</p>
<p data-nodeid="81615">或者你也可以通过修改 yaml 文件，然后 apply 到集群中：</p>
<pre class="lang-java" data-nodeid="81616"><code data-language="java">kubectl apply -f demo.yaml
</code></pre>
<p data-nodeid="81617">当然，这条命令还能被用来创建对象，如果对象已经存在就会对它进行更新。</p>
<p data-nodeid="81618">对于资源对象的删除，可以直接通过 kubectl delete 进行：</p>
<pre class="lang-java" data-nodeid="81619"><code data-language="java">kubectl delete [资源]/[对象名] [-n [命名空间]]
</code></pre>
<p data-nodeid="81620">比如：</p>
<pre class="lang-java" data-nodeid="81621"><code data-language="java">kubectl delete pod/pod-demo -n demo
</code></pre>
<p data-nodeid="81622">如果你是通过 yaml 文件创建的某些资源对象，比如 flannel；yaml 文件中包含了很多对象，一个个删除太麻烦，也容易遗漏，你就可以通过如下命令删除：</p>
<pre class="lang-java" data-nodeid="81623"><code data-language="java">kubectl delete -f flannel.yaml
</code></pre>
<h3 data-nodeid="81624">日志</h3>
<p data-nodeid="81625">如果 pod 内只有一个容器，你可以通过“kubectl logs [pod 名] -n [命名空间]”命令查看该容器的日志。</p>
<p data-nodeid="81626">如果有多个容器，就需要在“pod 名”后插入“-c [容器名]”，如下所示：</p>
<pre class="lang-java" data-nodeid="81627"><code data-language="java">kubectl logs [pod 名] -c [容器名] -n [命名空间]
</code></pre>
<p data-nodeid="81628">你还可以通过“-f”参数来实时查看容器最新的日志。更多参数，就需要你自己来探索了。</p>
<h3 data-nodeid="81629">快速创建一个 Pod</h3>
<p data-nodeid="81630">我们可以通过如下命令快速创建一个 Pod，在我们做环境 debug 的时候非常方便：</p>
<pre class="lang-java" data-nodeid="81631"><code data-language="java">kubectl run [pod 名] -n [命名空间] --image [镜像] [.....]
</code></pre>
<p data-nodeid="81632">比如：</p>
<pre class="lang-java" data-nodeid="81633"><code data-language="java">kubectl run pod1 -n debug-pod --image network-debug -- bash
</code></pre>
<p data-nodeid="81634">其他参数，你可以通过 “kubectl run -h” 查看；也可以直接 exec 到容器里查看，代码如下：</p>
<pre class="lang-java" data-nodeid="81635"><code data-language="java">kubectl exec -it [pod 名] -n [命名空间] bash
</code></pre>
<p data-nodeid="81636">同样，如果有多个容器的话，需要知道一个容器名：</p>
<pre class="lang-java" data-nodeid="81637"><code data-language="java">kubectl exec -it [pod 名] -c [容器名] -n [命名空间] bash
</code></pre>
<h3 data-nodeid="81638">写在最后</h3>
<p data-nodeid="81639">kubectl 支持 JSONPath 模版，在<a href="https://kubernetes.io/zh/docs/reference/kubectl/jsonpath" data-nodeid="81725">官方文档</a>中有详细的说明和例子，这里就不再重复了。</p>
<p data-nodeid="81640">kubectl 能力非常强大，随着你不断地使用，你会发现更多 kubectl 中的小技巧，也会慢慢积累一些自己的使用技巧。以上我只是列举了一些常用的命令操作，你可以点击链接查看<a href="https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/" data-nodeid="81730">官方的 kubectl 小抄</a>来学习。</p>
<p data-nodeid="81641">本文中的例子，只使用到了部分参数，你可以通过 “-h”来查看其具体支持的各个参数。</p>
<p data-nodeid="81642">如果你对本节课有什么想法或者疑问，欢迎在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **生：
> 学习加复习

##### **东：
> 写得真心不错

