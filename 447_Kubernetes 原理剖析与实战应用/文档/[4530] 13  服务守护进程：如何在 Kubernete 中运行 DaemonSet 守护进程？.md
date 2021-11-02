<p data-nodeid="35778" class="">通过前面课程的学习，我们对 Kubernetes 中一些常见工作负载已经有所了解。比如无状态工作负载 Dployment 可以帮助我们运行指定数目的服务副本，并维护其状态，而对于有状态服务来说，我们同样可以采用 StatefulSet 来做到这一点。</p>
<p data-nodeid="35779">但是，在实际使用的时候，有些场景，比如监控各个节点的状态，使用 Deployment 或者 StatefulSet 都无法满足我们的需求，因为这个时候我们可能会有以下这些需求。</p>
<ol data-nodeid="35780">
<li data-nodeid="35781">
<p data-nodeid="35782">希望每个节点上都可以运行一个副本，且只运行一个副本。虽然通过调整 spec.replicas 的数值，可以使之等于节点数目，再配合一些调度策略（我们后面讲调度原理的时候会深入解释）可以实现这一点。但是如果节点数目发生了变化呢？</p>
</li>
<li data-nodeid="35783">
<p data-nodeid="35784">希望在新节点上也快速拉起副本。比如集群扩容，这个时候会有一些新节点加入进来，如何立即感知到这些节点，并在上面部署新的副本。</p>
</li>
<li data-nodeid="35785">
<p data-nodeid="35786">希望节点下线的时候，对应的 Pod 也可以被删除。</p>
</li>
<li data-nodeid="35787">
<p data-nodeid="35788">……</p>
</li>
</ol>
<p data-nodeid="35789">Kubernetes 提供的 DaemonSet 就可以完美地解决上述问题，其主要目的就是可以在集群内的每个节点上（或者指定的一堆节点上）都只运行一个副本，即 Pod 和 Node 是一一对应的关系。DaemonSet 会结合节点的情况来帮助你管理这些 Pod，见下面的拓扑结构：</p>
<p data-nodeid="36967"><img src="https://s0.lgstatic.com/i/image/M00/5B/9B/Ciqc1F9_0FqACmcBAABg43Rbbow934.png" alt="Lark20201009-105149.png" data-nodeid="36971"></p>
<p data-nodeid="36968">今天我们就来学习一下 DaemonSet，先来看看其主要的使用场景。</p>




<h3 data-nodeid="35792">DaemonSet 的使用场景</h3>
<p data-nodeid="35793">跟 Deployment 和 StatefulSet 一样，DaemonSet 也是一种工作负载，可以管理一些 Pod。 通常来说，主要有以下的用法：</p>
<ul data-nodeid="35794">
<li data-nodeid="35795">
<p data-nodeid="35796">监控数据收集，比如可以将节点信息收集上报给 Prometheus；</p>
</li>
<li data-nodeid="35797">
<p data-nodeid="35798">日志的收集、轮转和清理；</p>
</li>
<li data-nodeid="35799">
<p data-nodeid="35800">监控节点状态，比如运行 node-problem-detector 来监测节点的状态，并上报给 APIServer；</p>
</li>
<li data-nodeid="35801">
<p data-nodeid="35802">负责在每个节点上网络、存储等组件的运行，比如 glusterd、ceph、flannel 等；</p>
</li>
</ul>
<p data-nodeid="35803">现在我们来尝试部署一个 DaemonSet。</p>
<h3 data-nodeid="35804">部署你的第一个 DaemonSet</h3>
<p data-nodeid="35805">这里是一个 DaemonSet 的 YAML 文件：</p>
<pre class="lang-c" data-nodeid="35806"><code data-language="c">apiVersion: apps/v1 # 这个地方已经不是 extension/v1beta1 了，在<span class="hljs-number">1.16</span>版本已经废弃了，请使用 apps/v1
kind: DaemonSet # 这个是类型名
metadata:
  name: fluentd-elasticsearch # 对象名
  <span class="hljs-keyword">namespace</span>: kube-system # 所属的命名空间
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  <span class="hljs-keyword">template</span>:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2<span class="hljs-number">.5</span><span class="hljs-number">.2</span>
        volumeMounts:
        - name: varlog
          mountPath: /var/<span class="hljs-built_in">log</span>
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: <span class="hljs-literal">true</span>
      restartPolicy: Always
      volumes:
      - name: varlog
        hostPath:
          path: /var/<span class="hljs-built_in">log</span>
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
</code></pre>
<p data-nodeid="35807">在这个 YAML 文件里，我们有一个地方需要特别注意，就 <code data-backticks="1" data-nodeid="35893">restartPolicy</code>这个字段，它是缺省字段，默认值是 <code data-backticks="1" data-nodeid="35895">Always</code>。而且如果你想显式地去设置，你也只能设置为 <code data-backticks="1" data-nodeid="35897">Always</code>。</p>
<p data-nodeid="35808">其他的配置和写法，跟我们之前了解的 Deployment 和 StatefulSet 是类似的。</p>
<p data-nodeid="35809">我们将上面 YAML 文件保存到本地的 <code data-backticks="1" data-nodeid="35901">fluentd-elasticsearch-ds.yaml</code> 中，然后用 <code data-backticks="1" data-nodeid="35903">kubectl apply</code>创建出来：</p>
<pre class="lang-c" data-nodeid="35810"><code data-language="c">$ kubectl apply -f fluentd-elasticsearch-ds.yaml
daemonset.apps/fluentd-elasticsearch created
</code></pre>
<p data-nodeid="35811">创建好后，我们来查看这个 DaemonSet 关联 Pod 的情况：</p>
<pre class="lang-java" data-nodeid="35812"><code data-language="java">$ kubectl get pod -n kube-system -l name=fluentd-elasticsearch
NAME                          READY   STATUS    RESTARTS   AGE
fluentd-elasticsearch-m9zjb   <span class="hljs-number">1</span>/<span class="hljs-number">1</span>     Running   <span class="hljs-number">0</span>          <span class="hljs-number">85</span>s
</code></pre>
<p data-nodeid="35813">可以看到，集群中只有一个 Pod 被创建了出来。我们再来看看集群中有多少个节点：</p>
<pre class="lang-java" data-nodeid="35814"><code data-language="java">$ kubectl get node
NAME             STATUS   ROLES    AGE   VERSION
docker-desktop   Ready    master   <span class="hljs-number">22d</span>   v1.<span class="hljs-number">16.6</span>-beta.<span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="35815">由于目前集群中就只有一个节点，所以 Kubernetes 只为这一个节点生成了 Pod。我们来查看下该 DaemonSet 的整体状态：</p>
<pre class="lang-java" data-nodeid="35816"><code data-language="java">$ kubectl get ds -n kube-system fluentd-elasticsearch
NAME                    DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd-elasticsearch   <span class="hljs-number">1</span>         <span class="hljs-number">1</span>         <span class="hljs-number">0</span>       <span class="hljs-number">1</span>            <span class="hljs-number">0</span>           &lt;none&gt;          <span class="hljs-number">18</span>
</code></pre>
<p data-nodeid="35817">在 kubectl 使用的时候，我们常常将<code data-backticks="1" data-nodeid="35909">DaemonSet </code> 缩写成 ds。这里输出列表的含义如下：<br>
DESIRED 代表该 DaemonSet 期望要创建的 Pod 个数，即我们需要几个，它跟节点数目息息相关；</p>
<ul data-nodeid="35818">
<li data-nodeid="35819">
<p data-nodeid="35820"><code data-backticks="1" data-nodeid="35913">CURRENT</code>代表当前已经存在的 Pod 个数；</p>
</li>
<li data-nodeid="35821">
<p data-nodeid="35822"><code data-backticks="1" data-nodeid="35915">READY</code> 代表目前已就绪的 Pod 个数；</p>
</li>
<li data-nodeid="35823">
<p data-nodeid="35824"><code data-backticks="1" data-nodeid="35917">UP-TO-DATE</code> 代表最新创建的个数；</p>
</li>
<li data-nodeid="35825">
<p data-nodeid="35826"><code data-backticks="1" data-nodeid="35919">AVAILABLE </code> 代表目前可用的 Pod个数；</p>
</li>
<li data-nodeid="35827">
<p data-nodeid="35828"><code data-backticks="1" data-nodeid="35921">NODE SELECTOR</code>表示节点选择标签，这个在 DaemonSet 中非常有用。有时候我们只希望在部分节点上运行一些 Pod，比如我们只节点上带有 app=logging-node 的节点上运行一些服务，就可以通过这个标签选择器来实现。</p>
</li>
</ul>
<h4 data-nodeid="35829">限定 DaemonSet 运行的节点</h4>
<p data-nodeid="35830">现在我们来看看如何限定一个 DaemonSet，让其只在某些节点上运行，比如只在带有 <code data-backticks="1" data-nodeid="35925">app=logging-node </code> 的节点上运行，可以看这张图：</p>
<p data-nodeid="35831"><img src="https://s0.lgstatic.com/i/image/M00/59/81/Ciqc1F9xnpWAKFwRAAB-Lq1l0YU648.png" alt="1.png" data-nodeid="35929"></p>
<p data-nodeid="35832">此时，我们就可以通过 DaemonSet 的 selector 来实现：</p>
<pre class="lang-c" data-nodeid="35833"><code data-language="c">apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: my-ds
  <span class="hljs-keyword">namespace</span>: demo
  Labels:
    key: value
spec:
  selector: # 通过这个 selector，我们就可以让 daemonset pod 只在指定的节点上运行
    matchLabels:
      app: logging-node
  <span class="hljs-keyword">template</span>:
    metadata:
      labels:
        name: my-daemonset-container
    ...
</code></pre>
<p data-nodeid="35834">这样一个 DaemonSet 就会只匹配集群中所有带有标签 <code data-backticks="1" data-nodeid="35932">app=logging-node</code> 的节点，并分别在这些节点上运行一个 Pod。</p>
<p data-nodeid="35835">如果集群中节点的标签发生了变化，这个时候<code data-backticks="1" data-nodeid="35935">DaemonSetsController</code>会立刻为新匹配上的节点创建 Pod，同时删除不匹配的节点上的 Pod。</p>
<p data-nodeid="35836">知道了这些我们再来看如何调度。</p>
<h4 data-nodeid="35837">DaemonSet 的 Pod 是如何被调度的</h4>
<p data-nodeid="35838">早期 Kubernetes 是通过<code data-backticks="1" data-nodeid="35940">DaemonSetsController</code>（在 kube- controller-manager 组件内以 goroutine 方式运行）调度 DaemonSet 管理的 Pod，这些 Pod 在创建的时候，就在 Pod 的 spec 中提前指定了节点名称，即 <code data-backticks="1" data-nodeid="35942">spec.nodeName</code>。这些 Pod 由于指定了节点，所以不会经过默认调度器进行调度，这就导致了一些问题。</p>
<ul data-nodeid="35839">
<li data-nodeid="35840">
<p data-nodeid="35841">不一致的 Pod 行为：其他的 Pod 都是通过默认调度器进行调度的，初始状态都是 Pending（等待调度），而 DaemonSet 的这些 Pod 的起始状态却不是 Pending。</p>
</li>
<li data-nodeid="35842">
<p data-nodeid="35843"><code data-backticks="1" data-nodeid="35945">DaemonSetsController</code> 并不会感知到节点的资源变化；</p>
</li>
<li data-nodeid="35844">
<p data-nodeid="35845">默认调度器的一些高级特性需要在 、DaemonSetsController 中二次实现。</p>
</li>
<li data-nodeid="35846">
<p data-nodeid="35847">多组件负责调度会导致 Pod 抢占等功能实现起来非常困难；</p>
</li>
<li data-nodeid="35848">
<p data-nodeid="35849">...</p>
</li>
</ul>
<p data-nodeid="35850">如果你有兴趣，你可以看看设计文档 <a href="https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/schedule-DS-pod-by-scheduler.md" data-nodeid="35953">Schedule DaemonSet Pods by default scheduler, not DaemonSet controller</a> ，它详细介绍了<code data-backticks="1" data-nodeid="35955">DaemonSetsController </code> 调度时遇到的各种问题，并给出了详细的解决方案。</p>
<p data-nodeid="35851">简单来说，DaemonSet Pod 依然由 <code data-backticks="1" data-nodeid="35958">DaemonSetsController</code> 进行创建，但是不预先指定<code data-backticks="1" data-nodeid="35960">spec.nodeName</code>了，而通过节点的亲和性，交由默认调度器进行调度。</p>
<p data-nodeid="35852">我们回过头来看看上面<code data-backticks="1" data-nodeid="35963">fluentd-elasticsearch</code>这个 DaemonSet 创建的 Pod：</p>
<pre class="lang-c" data-nodeid="35853"><code data-language="c">$ kubectl get pod -n kube-system fluentd-elasticsearch-m9zjb -o yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: <span class="hljs-string">"2020-09-25T12:01:31Z"</span>
  generateName: fluentd-elasticsearch-
  labels:
    controller-revision-hash: <span class="hljs-number">5b</span>5b9c8855
    name: fluentd-elasticsearch
    pod-<span class="hljs-keyword">template</span>-generation: <span class="hljs-string">"1"</span>
  name: fluentd-elasticsearch-m9zjb
  <span class="hljs-keyword">namespace</span>: kube-system
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: <span class="hljs-literal">true</span>
    controller: <span class="hljs-literal">true</span>
    kind: DaemonSet
    name: fluentd-elasticsearch
    uid: <span class="hljs-number">33</span>dc29aa<span class="hljs-number">-60b</span>0<span class="hljs-number">-4486</span><span class="hljs-number">-8645</span><span class="hljs-number">-731</span>daa85f25d
  ...
spec:
  affinity:
    nodeAffinity: <span class="hljs-meta"># daemonset 就是利用了 nodeAffinity 的能力</span>
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchFields:
          - key: metadata.name
            <span class="hljs-keyword">operator</span>: In
            values:
            - docker-desktop
  containers:
  - image: quay.io/fluentd_elasticsearch/fluentd:v2<span class="hljs-number">.5</span><span class="hljs-number">.2</span>
    imagePullPolicy: IfNotPresent
    name: fluentd-elasticsearch
    ...
  ...
</code></pre>
<p data-nodeid="35854"><code data-backticks="1" data-nodeid="35965">DaemonSetsController</code> 创建的这个 Pod，自动添加了 <code data-backticks="1" data-nodeid="35967">spec.affinity.nodeAffinity</code>指定节点的名称，替换以前<code data-backticks="1" data-nodeid="35969">spec.nodeName</code> 的方式。</p>
<p data-nodeid="35855">除了这个<code data-backticks="1" data-nodeid="35972">nodeAffinity</code>之外，<code data-backticks="1" data-nodeid="35974">DaemonSetsController</code>还会自动加一些 Toleration 到 Pod 中。有兴趣可以查看<a href="https://kubernetes.io/zh/docs/concepts/workloads/controllers/daemonset/#taint-and-toleration" data-nodeid="35978">这个清单</a>。我们在后续调度器章节，将统一介绍这些 Affinity 和 Toleration 的用法。</p>
<p data-nodeid="35856">接下来，我们看看它的升级方法。</p>
<h3 data-nodeid="35857">如何升级一个 Daemonset</h3>
<p data-nodeid="35858">升级一个 DaemonSet 其实非常简单，你可以通过<code data-backticks="1" data-nodeid="35983">kubectl edit</code> 直接编辑对应的 DaemonSet 对象：</p>
<pre class="lang-java" data-nodeid="35859"><code data-language="java">kubectl edit ds -n kube-system fluentd-elasticsearch
</code></pre>
<p data-nodeid="35860">也可以直接在<code data-backticks="1" data-nodeid="35986">fluentd-elasticsearch-ds.yaml</code> 中修改，然后使用<code data-backticks="1" data-nodeid="35988">kubectl apply </code> ：</p>
<pre data-nodeid="35861"><code>$ kubectl apply -f fluentd-elasticsearch-ds.yaml
</code></pre>
<p data-nodeid="35862">那么修改后，DaemonSet 的这些 Pod 又是如何更新的呢？</p>
<p data-nodeid="35863">DaemonSet 中定义了两种更新策略。</p>
<p data-nodeid="35864"><strong data-nodeid="36001">第一种是 OnDelete</strong>，顾名思义，当指定这种策略时，我们只有先手动删除原有的 Pod 才会触发新的 DaemonSet Pod 的创建。否则，不论你怎么修改 DaemonSet ，都不会触发新的 Pod 生成。<br>
<strong data-nodeid="36002">第二种是 RollingUpdate</strong>，这是默认的更新策略，使用这种策略可以实现滚动更新，同时你还可以更精细地控制更新的范围，比如通过 maxUnavailable 为 1 来控制更新的速度（你也可以理解为更新时设置的步长），这表示每次只会先删除 1 个 Pod，待其重新创建并Ready 后，再更新同样的方法更新其他的 Pod。在更新期间，每个节点上最多只能有 DaemonSet 的一个 Pod。</p>
<p data-nodeid="35865">我给你举个例子，下面这个 YAML 是一个使用了<code data-backticks="1" data-nodeid="36004">RollingUpdate</code>更新策略的 DaemonSet ：</p>
<pre class="lang-c" data-nodeid="35866"><code data-language="c">apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  <span class="hljs-keyword">namespace</span>: kube-system
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  updateStrategy: # 这里指定了更新策略
    type: RollingUpdate # 进行滚动更新
    rollingUpdate:
      maxUnavailable: <span class="hljs-number">1</span> # 这是默认的值
  <span class="hljs-keyword">template</span>:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2<span class="hljs-number">.5</span><span class="hljs-number">.2</span>
        volumeMounts:
        - name: varlog
          mountPath: /var/<span class="hljs-built_in">log</span>
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: <span class="hljs-literal">true</span>
      terminationGracePeriodSeconds: <span class="hljs-number">30</span>
      volumes:
      - name: varlog
        hostPath:
          path: /var/<span class="hljs-built_in">log</span>
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
</code></pre>
<h3 data-nodeid="35867">写在最后</h3>
<p data-nodeid="35868">从业务状态的角度来看，Kubernetes 使用 Deployment 和 StatefulSet 来分别支持无状态服务和有状态服务。而 DaemonSet 则从另外的一个角度来为集群中的所有节点提供基础服务，比如网络、存储等。</p>
<p data-nodeid="35869">通过 DaemonSet，我们可以确保在所有满足条件的 Node 上只运行一个 Pod 实例，通常使用于日志收集、监控、网络等场景。Kubernetes 的组件 kube-prox 有时也可以借助 DaemonSet 拉起，这样每个节点上就会只运行一个 kube-proxy 的实例。Kubeadm 拉起的集群就是这样部署 kube-proxy 的。</p>
<p data-nodeid="35870">同时 DaemonSet 也帮助我们解决了集群中节点动态变化时业务实例的部署和运维能力，比如扩容、缩容、节点宕机等场景。</p>
<p data-nodeid="35871" class="">好的，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **伟：
> 希望每个节点上都可以运行一个副本，且只运行一个副本。其中的 节点，指的是什么

