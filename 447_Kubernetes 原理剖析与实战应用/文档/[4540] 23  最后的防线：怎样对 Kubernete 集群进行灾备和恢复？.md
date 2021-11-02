<p data-nodeid="348013">Kubernetes 隐藏了所有容器编排的复杂细节，让我们可以专注在应用本身，而无须过多关注如何去做部署和维护。此外，Kubernetes 还支持多副本，可以保证我们业务的高可用性。而对于集群本身而言，我们一样也要保证其高可用性，你可以参考官方文档：<a href="https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/" data-nodeid="348083">利用 Kubeadm 来创建高可用集群</a>。</p>
<p data-nodeid="348014">但是这些并不足以让我们高枕无忧，因为 Kubernetes 在帮助我们编排调度容器的同时，往往还保存了很多关键数据，比如集群自身关键数据、密钥、业务配置信息、业务数据等。我们在使用 Kubernetes 的时候，非常有必要进行灾备，防止出现操作失误（比如大规模无删除）、自然灾害、磁盘损坏无法修复、网络异常、机房断电等情况导致的数据丢失，严重时甚至会导致整个集群不可用。</p>
<p data-nodeid="348015">所以在使用 Kubernetes 的时候，我们最好做个灾备以方便对集群进行恢复，回滚到早期的一个稳定的状态。</p>
<h3 data-nodeid="348016">Kubernetes 需要备份哪些东西</h3>
<p data-nodeid="348017">在对 Kubernetes 集群做备份之前，我们首先得知道要备份哪些东西。</p>
<p data-nodeid="349319">我们从整个 Kubernetes 的架构为出发点，来看看整个集群的组件。我在《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=447#/detail/pc?id=4519" data-nodeid="349324">02 | 高屋建瓴：Kubernetes 的架构为什么是这样的？</a>》中讲到 Kubernetes 的官方架构图，如下：</p>
<p data-nodeid="349703"><img src="https://s0.lgstatic.com/i/image/M00/6B/F9/Ciqc1F-qTFCAfuayAAHPVgKdC98338.png" alt="Drawing 0.png" data-nodeid="349707"></p>
<div data-nodeid="349704" class=""><p style="text-align:center">图 1：Kubernetes 官方架构图</p></div>





<p data-nodeid="348021">从上图可以看出，整个 Kubernetes 集群可分为Master 节点（左侧）和 Node 节点（右侧）。</p>
<p data-nodeid="348022">在 Master 节点上，我们运行着 Etcd 集群以及 Kubernetes 控制面的几大组件，比如 kube-apiserver、kube-controller-manager、kube-scheduler 和 cloud-controller-manager（可选）等。</p>
<p data-nodeid="348023">在这些组件中，除了 Etcd，其他都是无状态的服务。只要保证 Etcd 的数据正常，其他几个组件不管出现什么问题，我们都可以通过重启或者新建实例来解决，并不会受到任何影响。因此我们<strong data-nodeid="348105">只需要备份 Etcd 中的数据</strong>。</p>
<p data-nodeid="348024">看完了 Master 节点，我们再来看看 Node 节点。</p>
<p data-nodeid="348025">Node 节点上运行着&nbsp;kubelet、kube-proxy 等服务。Kubelet 负责维护各个容器实例，以及容器使用到的存储。为了保证数据的持久化存储，对于关键业务的关键数据，我都建议通过我在《<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=447#/detail/pc?id=4527" data-nodeid="348110">10 | 存储管理：怎样对业务数据进行持久化存储？</a>》中提到的 PV（Persistent Volume）来保存和使用。鉴于这一点，我们<strong data-nodeid="348116">还需要对 PV 进行备份</strong>。</p>
<p data-nodeid="348026">如果是节点出现了问题，我们可以向集群中增加新的节点，替换掉有问题的节点。</p>
<p data-nodeid="348027">看完 Kubernetes 的官方架构图之后，下面我们就来看看该如何备份 Etcd 中的数据和 PV。</p>
<h4 data-nodeid="348028">对 Etcd 数据进行备份及恢复</h4>
<p data-nodeid="348029">Etcd 官方也提供了<a href="https://etcd.io/docs/v3.4.0/op-guide/recovery/" data-nodeid="348123">备份的文档</a>，你有兴趣可以阅读一下。我在这里总结了一些实际操作，以便你后续可以借鉴并进行手动的备份和恢复。命令行里面的一些证书路径以及 endpoint 地址需要根据自己的集群参数进行更改。实际操作代码如下：</p>
<pre class="lang-shell" data-nodeid="348030"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> 0. 数据备份</span>
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--key=/etc/kubernetes/pki/etcd/peer.key \
--cert=/etc/kubernetes/pki/etcd/peer.crt \
snapshot save ./new.snapshot.db
<span class="hljs-meta">#</span><span class="bash"> 1. 查看 etcd 集群的节点</span>
ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \ 
--cacert=/etc/kubernetes/pki/etcd/ca.crt \ 
--cert=/etc/kubernetes/pki/etcd/peer.crt \ 
--key=/etc/kubernetes/pki/etcd/peer.key \
member list
<span class="hljs-meta">#</span><span class="bash"> 2. 停止所有节点上的 etcd！（注意是所有！！）</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 如果是 static pod，可以听过如下的命令进行 stop</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 如果是 systemd 管理的，可以通过 systemctl stop etcd</span></span>
mv /etc/kubernetes/manifests/etcd.yaml /etc/kubernetes/
<span class="hljs-meta">#</span><span class="bash"> 3. 数据清理</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 依次在每个节点上，移除 etcd 数据</span></span>
rm -rf /var/lib/etcd
<span class="hljs-meta">#</span><span class="bash"> 4. 数据恢复</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 依次在每个节点上，恢复 etcd 旧数据</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 里面的 name，initial-advertise-peer-urls，initial-cluster=controlplane</span></span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 等参数，可以从 etcd pod 的 yaml 文件中获取到。</span></span>
ETCDCTL_API=3 etcdctl snapshot restore ./old.snapshot.db \
--data-dir=/var/lib/etcd \
--name=controlplane \
--initial-advertise-peer-urls=https://172.17.0.18:2380 \
--initial-cluster=controlplane=https://172.17.0.18:2380
<span class="hljs-meta">#</span><span class="bash"> 5. 恢复 etcd 服务</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 依次在每个节点上，拉起 etcd 服务</span></span>
mv /etc/kubernetes/etcd.yaml /etc/kubernetes/manifests/
systemctl restart kubelet
</code></pre>
<p data-nodeid="348031">上述这些备份，都需要手动运行命令行进行操作。如果你的 Etcd 集群是运行在 Kubernetes 集群中的，你可以通过以下的定时 Job (CronJob) 来帮你自动化、周期性（如下的 YAML 文件中会每分钟对 Etcd 进行一次备份）地备份 Etcd 的数据。关于 CronJob 部分的内容，我们在后面单独章节会进行介绍。自动备份代码如下：</p>
<pre class="lang-yaml" data-nodeid="348032"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">batch/v1beta1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">CronJob</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">backup</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">kube-system</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-comment"># activeDeadlineSeconds: 100</span>
  <span class="hljs-attr">schedule:</span> <span class="hljs-string">"*/1 * * * *"</span>
  <span class="hljs-attr">jobTemplate:</span>
    <span class="hljs-attr">spec:</span>
      <span class="hljs-attr">template:</span>
        <span class="hljs-attr">spec:</span>
          <span class="hljs-attr">containers:</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">backup</span>
            <span class="hljs-comment"># Same image as in /etc/kubernetes/manifests/etcd.yaml</span>
            <span class="hljs-attr">image:</span> <span class="hljs-string">k8s.gcr.io/etcd:3.2.24</span>
            <span class="hljs-attr">env:</span>
            <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">ETCDCTL_API</span>
              <span class="hljs-attr">value:</span> <span class="hljs-string">"3"</span>
            <span class="hljs-attr">command:</span> <span class="hljs-string">["/bin/sh"]</span>
            <span class="hljs-attr">args:</span> <span class="hljs-string">["-c",</span> <span class="hljs-string">"etcdctl --endpoints=https://127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key snapshot save /backup/etcd-snapshot-$(date +%Y-%m-%d_%H:%M:%S_%Z).db"</span><span class="hljs-string">]</span>
            <span class="hljs-attr">volumeMounts:</span>
            <span class="hljs-bullet">-</span> <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/etc/kubernetes/pki/etcd</span>
              <span class="hljs-attr">name:</span> <span class="hljs-string">etcd-certs</span>
              <span class="hljs-attr">readOnly:</span> <span class="hljs-literal">true</span>
            <span class="hljs-bullet">-</span> <span class="hljs-attr">mountPath:</span> <span class="hljs-string">/backup</span>
              <span class="hljs-attr">name:</span> <span class="hljs-string">backup</span>
          <span class="hljs-attr">restartPolicy:</span> <span class="hljs-string">OnFailure</span>
          <span class="hljs-attr">hostNetwork:</span> <span class="hljs-literal">true</span>
          <span class="hljs-attr">volumes:</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">etcd-certs</span>
            <span class="hljs-attr">hostPath:</span>
              <span class="hljs-attr">path:</span> <span class="hljs-string">/etc/kubernetes/pki/etcd</span>
              <span class="hljs-attr">type:</span> <span class="hljs-string">DirectoryOrCreate</span>
          <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">backup</span>
            <span class="hljs-attr">hostPath:</span>
              <span class="hljs-attr">path:</span> <span class="hljs-string">/data/backup</span>
              <span class="hljs-attr">type:</span> <span class="hljs-string">DirectoryOrCreate</span>
</code></pre>
<h4 data-nodeid="348033">对 PV 的数据进行备份</h4>
<p data-nodeid="348034">对于 PV 来讲，备份就比较麻烦了。Kubernetes 自身不提供存储能力，它依赖各个存储插件对存储进行管理和使用。因此对于存储的备份操作，尤其是 PV 的备份操作，我们需要依赖各个云提供商的 API 来做 snapshot。</p>
<p data-nodeid="348035">但是上述对于 Etcd 和 PV 的备份操作并不是很方便，我推荐你通过<a href="https://github.com/vmware-tanzu/velero" data-nodeid="348131">Velero</a>来备份 Kubernetes。Velero 功能强大，但是操作起来很简单，它可以帮你做到以下 3 点：</p>
<ol data-nodeid="348036">
<li data-nodeid="348037">
<p data-nodeid="348038">对 Kubernets 集群做备份和恢复。</p>
</li>
<li data-nodeid="348039">
<p data-nodeid="348040">对集群进行迁移。</p>
</li>
<li data-nodeid="348041">
<p data-nodeid="348042">对集群的配置和对象进行复制，比如复制到其他的开发和测试集群中去。</p>
</li>
</ol>
<p data-nodeid="348043">而且 Velero 还提供针对单个 Namespace 进行备份的能力，如果你只想备份某些关键的业务和数据，这是一个十分方便的功能。</p>
<p data-nodeid="348044">说了这么多，下满我们来看看 Velero 是如何备份 Kubernetes 的。</p>
<h3 data-nodeid="348045">使用 Velero 对 Kubernetes 进行备份</h3>
<p data-nodeid="350808">这是 Velero 的架构图：</p>
<p data-nodeid="350809" class=""><img src="https://s0.lgstatic.com/i/image/M00/6C/04/CgqCHl-qTK6ADCLwAAFaThL2Fxk754.png" alt="Drawing 1.png" data-nodeid="350814"></p>
<div data-nodeid="350810"><p style="text-align:center">图 2：Velero 架构图</p></div>





<p data-nodeid="348049">Velero 由两部分组成：</p>
<ul data-nodeid="348050">
<li data-nodeid="348051">
<p data-nodeid="348052"><strong data-nodeid="348149">一个命令行客户端</strong>，你可以运行在本地，通过命令行完成对 Etcd 以及 PV 的备份操作；你也可以像使用 kubectl 操作 Kubernetes 资源一样备份 Kubernetes。</p>
</li>
<li data-nodeid="348053">
<p data-nodeid="348054"><strong data-nodeid="348154">一个运行在 kubernetes 集群中的服务</strong>（BackupController），负责执行具体的备份和恢复操作。</p>
</li>
</ul>
<p data-nodeid="348055">我们来看看具体使用时的流程：</p>
<ol data-nodeid="348056">
<li data-nodeid="348057">
<p data-nodeid="348058">通过本地 Velero 客户端发送备份命令，比如图中的<code data-backticks="1" data-nodeid="348157">velero backup create test-project-s2i --include-namespaces test</code>，这条命令会向 APIServer 中创建一个 Backup 对象。</p>
</li>
<li data-nodeid="348059">
<p data-nodeid="348060">BackupController 会去监测并验证这个 Backup 对象的合法性，比如参数的定义。</p>
</li>
<li data-nodeid="348061">
<p data-nodeid="348062">BackupController 通过向 APIServer 查询相关数据开始备份工作。</p>
</li>
<li data-nodeid="348063">
<p data-nodeid="348064">BackupController 将查询到的数据备份到远端的对象存储中。</p>
</li>
</ol>
<p data-nodeid="348065">Velero 在 Kubernetes 集群中创建了很多 CRD （Custome Resource Definition）以及相关的控制器，通过这些进行备份恢复等操作。因此，对集群的备份和恢复，实质上是对这些相关 CRD 的操作。BackupController 会根据 CRD 来确定该进行何种操作。我会在《<strong data-nodeid="348167">27 | K8s CRD：如何根据需求自定义你的 API？</strong>》中专门介绍 CRD 的使用。</p>
<p data-nodeid="348066">Velero 支持两种关于后端存储的 CRD，分别是 BackupStorageLocation 和 VolumeSnapshotLocation。</p>
<ul data-nodeid="348067">
<li data-nodeid="348068">
<p data-nodeid="348069">BackupStorageLocation 主要用来定义 Kubernetes 集群资源的数据存放位置，也就是集群对象数据，而不是 PVC 和 PV 的数据。你可以从这个<a href="https://velero.io/docs/main/supported-providers/" data-nodeid="348172">支持列表</a>里面找到目前官方和第三方支持的后端存储服务，主要是以支持 S3 兼容的存储为主，比如 AWS S3、阿里云 OSS、Minio 等。</p>
</li>
<li data-nodeid="348070">
<p data-nodeid="348071">VolumeSnapshotLocation 主要用来给 PV 做快照，快照功能通常由 Amazon EBS Volumes、Azure Managed Disks、Google Persistent Disks 等云厂商提供，你可以根据需要选择使用各个云厂商的服务。或者你使用专门的备份工具<a href="https://github.com/restic/restic" data-nodeid="348177">Restic</a>，把 PV 数据备份到<a href="https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv" data-nodeid="348181">Azure Files</a>、阿里云 OSS 中去。阿里云目前已经提供了<a href="https://github.com/AliyunContainerService/velero-plugin" data-nodeid="348185">基于 Velero 的插件</a>。</p>
</li>
</ul>
<p data-nodeid="348072">除此之外，BackupController 在工作过程中，还会创建其他的 CRD，主要用于内部的逻辑处理。你可以参考阿里云的<a href="https://developer.aliyun.com/article/726863" data-nodeid="348190">文档</a>进一步学习。</p>
<p data-nodeid="348073">如果你没有阿里云的 OSS，或者集群是线下的内部集群，你也可以自行搭建 Minio，作为对象存储服务来代替阿里云的 OSS。你可以参考官方的<a href="https://velero.io/docs/main/contributions/minio/" data-nodeid="348195">文档</a>进行详细的安装配置工作。</p>
<h3 data-nodeid="348074">写在最后</h3>
<p data-nodeid="348075">在分布式的世界里，我们很难保证万无一失。当你在 Kubernetes 集群中部署越来越多的业务的时候，对集群和数据的灾备是非常有必要的。在今年 7 月份，我们常用的代码托管平台 Github 就发生了 Kubernetes 故障 ，导致了持续 4 个半小时的严重故障。所以，我建议对于关键性的业务数据，要记得时常备份。</p>
<p data-nodeid="348076">那么，你对于 Kubernetes 的备份还有哪些想要了解的呢？欢迎在留言区留言。</p>

---

### 精选评论

##### **8761：
> Etcd集群本身也是有主从关系的，而且可能会发生漂移，那使用crontab进行备份的时候是不是该先找出master后再进行备份？

##### **佳：
> 老师我有个问题 就是之前备份的数据能被恢复到不同的网段吗？我今天做实验发生了一个问题，就是由于路由器被更换，所有虚拟机的网段都被换了，从原来192.168.10.0/24变成了192.168.50.0/24，导致现在所有pod都无法操作了……请问一下这种情况要如何处理呢？谢谢

##### *航：
> K8s etcd数据何时建议使用恢复,恢复前需要注意什么

