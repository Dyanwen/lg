<p data-nodeid="553" class="">你好，我是正范。在上一讲，我们学习了如何通过一个 YAML 文件来定义一个 CRD，即扩展 API。这种扩展 API 跟 Kubernetes 内置的其他 API 同等地位，都可以通过 kubectl 或者 REST 接口访问，在使用过程中不会有任何差异。</p>
<p data-nodeid="554">但只是定义一个 CRD 并没有什么作用。虽说 kube-apiserver 会将其数据存放到 etcd 中，并暴露出相应的 REST 接口，然而并不涉及该对象的核心处理逻辑。</p>
<p data-nodeid="555">如何对这些 CRD 定义的对象进行一些逻辑处理，需要由用户自己来定义和实现，也就是通过控制器来实现。对此，我们有个专门的名字：Operator。</p>
<h3 data-nodeid="556">什么是 Kubernetes Operator</h3>
<p data-nodeid="557">你可能对 Operator 这个名字比较陌生。这个名字最早由 <a href="https://coreos.com/operators/" data-nodeid="620">CoreOS</a> 在 2016 年提出来，我们来看看他们给出的定义：</p>
<blockquote data-nodeid="558">
<p data-nodeid="559">An operator is a method of packaging, deploying and managing a Kubernetes application. A Kubernetes application is an application that is both deployed on Kubernetes and managed using the Kubernetes APIs and kubectl tooling.</p>
<p data-nodeid="560">To be able to make the most of Kubernetes, you need a set of cohensive APIs to extend in order to service and manage your applications that run on Kubernetes. You can think of Operators as the runtime that manages this type of application on Kubernetes.</p>
</blockquote>
<p data-nodeid="561">总结一下，所谓的 Kubernetes Operator 其实就是借助 Kubernetes 的控制器模式，配合一些自定义的 API，完成对某一类应用的操作，比如资源创建、资源删除。</p>
<p data-nodeid="562">这里对 Kubernetes 的控制器模式做个简要说明。<strong data-nodeid="630">Kubernetes 通过声明式 API 来定义对象，各个控制器负责实时查看对应对象的状态，确保达到定义的期望状态</strong>。这就是 Kubernetes 的控制器模式。</p>
<p data-nodeid="563"><img src="https://s0.lgstatic.com/i/image/M00/74/A3/Ciqc1F_HAmyACHLHAAHSt7ZcZoY464.png" alt="Drawing 0.png" data-nodeid="633"></p>
<p data-nodeid="564">kube-controller-manager 就是由这样一组控制器组成的。我们以 StatefulSet 为例来简单说明下控制器的具体逻辑。</p>
<p data-nodeid="565">假设你声明了一个 StatefulSet，并将其副本数设置为 2。kube-controller-manager 中以 goroutine 方式运行的 StatefulSet 控制器在观察 kube-apiserver 的时候，发现了这个新创建的对象，它会先创建一个 index 为 0 的 Pod ，并实时观察这个 Pod 的状态，待其状态变为 Running 后，再创建 index 为 1 的 Pod。后续该控制器会一直观察并维护这些 Pod 的状态，保证 StatefulSet 的有效副本数始终为 2。</p>
<p data-nodeid="566">所以我们在声明完成 CRD 之后，也需要创建一个控制器，即 Operator，来完成对应的控制逻辑。</p>
<p data-nodeid="567">了解了 Operator 的概念和控制器模式后，我们来看看 Operator 是如何工作的。</p>
<h3 data-nodeid="568">Kubernetes Operator 是如何工作的</h3>
<p data-nodeid="569">Operator 工作的时候采用上述的控制器模式，会持续地观察 Kubernetes 中的自定义对象，即 CR（Custom Resource）。我们通过 CRD 来定义一个对象，CR 则是 CRD 实例化的对象。</p>
<p data-nodeid="570"><img src="https://s0.lgstatic.com/i/image/M00/74/A3/Ciqc1F_HAnWAZUN3AAGfGj4K8Gw651.png" alt="Drawing 1.png" data-nodeid="642"></p>
<p data-nodeid="571">Operator 会持续跟踪这些 CR 的变化事件，比如 ADD、UPDATE、DELETE，然后采取一系列操作，使其达到期望的状态。</p>
<p data-nodeid="572">那么具体的代码层面，整个逻辑又如何实现呢？</p>
<p data-nodeid="2643" class="">下面就是 Operator 代码层面的工作流程图：</p>








<p data-nodeid="574"><img src="https://s0.lgstatic.com/i/image/M00/74/ED/CgqCHl_HLoWAHjvEAAPol71Pgh8456.png" alt="image.png" data-nodeid="648"></p>
<p data-nodeid="7106" class="te-preview-highlight">如上图所示，上半部分是一个 Informer，它的机制就是不断地 list/watch kube-apiserver 中特定的资源，比如你只关心 Pod，那么就只 list/watch Pod。Informer 主要有两个方法：一个是 ListFunc，一个是 WatchFunc。</p>


<ul data-nodeid="6823">
<li data-nodeid="6824">
<p data-nodeid="6825" class="">ListFunc 可以把某类资源的所有资源都列出来，当然你可以指定某个命名空间。</p>
</li>
<li data-nodeid="6826">
<p data-nodeid="6827">WatchFunc 则会和 apiserver 建立一个长链接，一旦有一个对象更新或者新对象创建，apiserver 就会反向推送回来，告诉 Informer 有一个新的对象创建或者更新等操作。</p>
</li>
</ul>



<p data-nodeid="5701" class="">当 Informer 接收到了这些操作，就会调用对应的函数（比如 AddFunc、UpdateFunc 和 DeleteFunc），并将其按照 key 值的格式放到一个先入先出的队列中。</p>





<p data-nodeid="582">key 值的命名规则就是 “namespace/name”，name 是对应的资源的名字，比如在 default 的 namespace 中创建一个 foo 类型的资源 example-foo，那么它的 key 值就是 “default/example-foo”。</p>
<p data-nodeid="583">我们一般会给 Operator 设置多个 Worker，并行地从上面的队列中拿到对象去操作处理。工作完成之后，就把这个 key 丢掉，表示已经处理完成。但如果处理过程中有错误，则可以把这个 key 重新放回到队列中，后续再重新处理。</p>
<p data-nodeid="584">看得出来，上述的流程其实还是有些复杂的，尤其是对初学者有一定的门槛。幸好社区提供了一些脚手架，方便我们快速地构建自己的 Operator。</p>
<h3 data-nodeid="585">构建一个自己的 Kubernetes Operator</h3>
<p data-nodeid="586">目前社区有一些可以用于创建 Kubernetes Operator 的开源项目，例如：<a href="https://github.com/operator-framework/operator-sdk" data-nodeid="660">Operator SDK</a>、<a href="https://github.com/kubernetes-sigs/kubebuilder" data-nodeid="664">Kubebuilder</a>、<a href="https://github.com/kudobuilder/kudo" data-nodeid="668">KUDO</a>。</p>
<p data-nodeid="587">有了这些脚手架，我们就可以快速创建出 Operator 的框架。这里我以 kubebuilder 为例。</p>
<p data-nodeid="4311" class="">我们可以先通过如下命令安装 kustomize：</p>






<pre data-nodeid="589"><code>curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh" | bash
mv kustomize /usr/local/bin/
</code></pre>
<p data-nodeid="590">再安装 kubebuilder。你可以选择通过源码安装：</p>
<pre class="lang-shell" data-nodeid="591"><code data-language="shell">git clone https://github.com/kubernetes-sigs/kubebuilder
cd kubebuilder
git checkout v2.3.1
make build
cp bin/kubebuilder $GOPATH/bin
</code></pre>
<p data-nodeid="592">如果你本地有些代码拉不下来，可以用 proxy：</p>
<pre class="lang-shell" data-nodeid="593"><code data-language="shell">export GOPROXY=https://goproxy.cn
</code></pre>
<p data-nodeid="594">也可以直接下载二进制文件：</p>
<pre class="lang-shell" data-nodeid="595"><code data-language="shell">os=$(go env GOOS)
arch=$(go env GOARCH)
<span class="hljs-meta">#</span><span class="bash"> download kubebuilder and extract it to tmp</span>
curl -sL https://go.kubebuilder.io/dl/2.3.1/${os}/${arch} | tar -xz -C /tmp/
sudo mv /tmp/kubebuilder_2.3.1_${os}_${arch} /usr/local/bin/kubebuilder
</code></pre>
<p data-nodeid="596">安装好以后，我们就可以通过 kubebuilder 来创建 Operator 了：</p>
<pre class="lang-shell" data-nodeid="597"><code data-language="shell">cd $GOPATH/src
mkdir -p github.com/zhangsan/operator-demo
cd github.com/zhangsan/operator-demo
kubebuilder init --domain abc.com --license apache2 --owner "zhangsan"
kubebuilder create api --group foo --version v1 --kind Bar
</code></pre>
<p data-nodeid="598">通过上面 kubebuilder 的命令，我们会在当前目录创建一个 Operator 的框架，并声明了一个 Bar 类型的 API。</p>
<p data-nodeid="599">你通过 make manifests 即可生成所需要的 yaml 文件，包括 CRD、RBAC等。</p>
<p data-nodeid="600">通过如下的命令即可安装 CRD、RBAC等对象：</p>
<pre class="lang-shell" data-nodeid="601"><code data-language="shell">make install # 安装CRD
</code></pre>
<p data-nodeid="602">然后我们就可以看到创建的CRD了：</p>
<pre class="lang-shell" data-nodeid="603"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> kubectl get crd</span>
NAME               AGE
bars.foo.abc.com   2m
</code></pre>
<p data-nodeid="604">再来创建一个 Bar 类型的对象：</p>
<pre class="lang-shell" data-nodeid="605"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> kubectl apply -f config/samples/</span>
<span class="hljs-meta">#</span><span class="bash"> kubectl get bar </span>
NAME         AGE
bar-sample   25s
</code></pre>
<p data-nodeid="606">在本地开发阶段，我们可以通过 make run 命令，在本地运行。运行起来以后，你可以从输出日志中看到我们刚创建的 default 命名空间下的 bar-sample，即 key 为 “default/bar-sample”。</p>
<p data-nodeid="607">我们在开发的时候，只需要修改 “api/v1/bar_types.go”和“controllers/bar_controller.go”这两个文件即可。这两个文件中有注释，会告诉你新增的对象定义和具体逻辑写哪里。这里你也可以参考 kubebuilder 的文档。</p>
<p data-nodeid="608">你开发完成之后，就可以构建一个镜像出来，方便部署：</p>
<pre class="lang-shell" data-nodeid="609"><code data-language="shell">make docker-build docker-push IMG=zhangsan/operator-demo
make deploy
</code></pre>
<h3 data-nodeid="610">写在最后</h3>
<p data-nodeid="611">到这里，你就完成了对扩展 Kubernetes API 的学习。这一讲的难点不在于 Operator 本身，而是要学会理解它的行为。</p>
<p data-nodeid="612" class="">如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **刚：
> 光定义一个控制器，算crd吗？

