<p data-nodeid="1013"><img alt="Drawing 0.png" src="https://s0.lgstatic.com/i/image/M00/68/87/CgqCHl-j3qaAVQfkACNvx19akQc735.png" data-nodeid="1087"><br>
（<a href="https://rancher.com/blog/2019/2019-01-17-101-more-kubernetes-security-best-practices/%EF%BC%89" data-nodeid="1092">https://rancher.com/blog/2019/2019-01-17-101-more-kubernetes-security-best-practices/）</a></p>
<p data-nodeid="1014">你好，我是正范。</p>
<p data-nodeid="1015">Kubernetes 作为一个分布式集群的管理工具，提供了非常强大的可扩展能力，可以帮助你管理容器，实现业务的高可用性和弹性能力，保障业务的规模。现在也有越来越多的企业正在逐步将核心应用部署到 Kubernetes 集群中。</p>
<p data-nodeid="1016">但是当业务规模扩大，集群的承载能力变大的时候，Kubernetes 平台自身的安全性就不得不考虑起来。</p>
<p data-nodeid="1017">那么 Kubernetes 平台自身身的安全问题如何解决？我们又该采取什么的策略来保证我们业务应用的安全部署？</p>
<h3 data-nodeid="1018">Kubernetes 的安全性</h3>
<p data-nodeid="1019"><a href="https://thenewstack.io/a-security-checklist-for-cloud-native-kubernetes-environments/" data-nodeid="1100">A Security Checklist for Cloud Native Kubernetes Environments</a> 这篇文章对 Kubernetes 的安全性总结得非常好，将它 Kubernetes 的安全性归纳为了以下四个方面： Infrastructure（基础设施）、Kubernetes 集群自身、Containers（容器）及其运行时和 Applications（业务应用）。</p>
<p data-nodeid="1020"><img alt="Drawing 1.png" src="https://s0.lgstatic.com/i/image/M00/68/7C/Ciqc1F-j3r-AS8V0AAHzD-VlJ8M782.png" data-nodeid="1104"></p>
<p data-nodeid="1021">(<a href="https://thenewstack.io/a-security-checklist-for-cloud-native-kubernetes-environments/" data-nodeid="1108">https://thenewstack.io/a-security-checklist-for-cloud-native-kubernetes-environments/)</a>)</p>
<p data-nodeid="1022">我建议你详细阅读一下这篇文档，我们这里只是做一些简短的总结介绍。</p>
<p data-nodeid="1023">Infrastructure，即基础设施层。正所谓“万丈高楼平地起”，基础设施的安全性是最基础的，也是最重要和关键的，却常常被忽略。这里基础设施，主要包括网络、存储、物理机、操作系统，等等。</p>
<p data-nodeid="1024">Kubernetes 其实对用户屏蔽了底层的基础架构，所以我们在初期规划和设计网络的时候，要提前做好规划，比如同时支持基于第 2 层 VLAN 的分段和基于第 3 层 VXLAN 的分段，以隔离不同租户或不同应用程序之间的流量。</p>
<p data-nodeid="1025">如果你的 Kubernetes 集群搭建在云上，社区也给出了<a href="https://rancher.com/blog/2019/2019-01-17-101-more-kubernetes-security-best-practices/" data-nodeid="1116">云上 Kubernetes 集群的 9 大安全最佳实践</a>供你参考，如果你使用了一些 Cloud Provider 也可以参考这篇文档。</p>
<h3 data-nodeid="1026">最佳实践</h3>
<p data-nodeid="1027">在底下基础设施安全的情况下，我们下一步要增强安全性的就是 Kubernetes 集群自身了。我们主要详细来看看 Kubernetes 集群及业务层安全性的十个最佳实践。</p>
<h4 data-nodeid="1028">1. 集群版本更新及 CVE 漏洞</h4>
<p data-nodeid="1029">首先，也是最重要的，你要时刻关注社区 Kubernetes 的版本更新，以及披露的 <a href="https://www.cvedetails.com/vulnerability-list/vendor_id-15867/product_id-34016/Kubernetes-Kubernetes.html" data-nodeid="1126">CVE 漏洞</a>，及时地把 CVE 的修复方案变更到你的集群中去。</p>
<p data-nodeid="1030">同时你需要保证跟社区的版本不要太脱节，跟社区保持 1 到 2 个大版本的差异。</p>
<h4 data-nodeid="1031">2. 保护好 Etcd 集群</h4>
<p data-nodeid="1032">Etcd 中保存着整个 Kubernetes 集群中最重要的数据，比如 Pod 信息、Secret、Token 等。一旦这些数据遭到攻击，造成的影响面非常巨大。我们必须确保 Etcd 集群的安全。</p>
<p data-nodeid="1033">对于部署 Etcd 集群的各个节点，我们应该被授予最小的访问权限，同时还要尽量避免这些节点被用作其他用途。由于 Etcd 对数据的读写要求很高，这里磁盘最好是 SSD 类型。</p>
<p data-nodeid="1034">Etcd 集群要配置<a href="https://www.baidu.com/s?ie=utf-8&amp;f=8&amp;rsv_bp=1&amp;rsv_idx=1&amp;tn=baidu&amp;wd=%E5%8F%8C%E5%90%91tls&amp;fenlei=256&amp;rsv_pq=c8a0d88c0000e037&amp;rsv_t=d667Pudr2AHhkSCaZ6OpWuO3TlaXRotfANdu6vCun6oZS%2F4Agn%2Fj1iqc3Ac&amp;rqlang=cn&amp;rsv_enter=1&amp;rsv_dl=tb&amp;rsv_sug3=26&amp;rsv_sug1=20&amp;rsv_sug7=101&amp;rsv_sug2=0&amp;rsv_btype=i&amp;inputT=3772&amp;rsv_sug4=3772" data-nodeid="1137">双向 TLS 认证（mTLS）</a>，用于 Etcd 各个节点之间的通信。同时 APIServer 对 Etcd 集群的访问最好也要基于 mTLS。通过 Kubeadm 搭建出来的集群，默认已经采取这种配置方式。</p>
<h4 data-nodeid="1035">3. 限制对 Kubernetes APIServer 的访问</h4>
<p data-nodeid="1036">APIServer 是整个 Kubernetes 的大脑，及流量入口，所有的数据都在此进行交互。Kubernetes 的安全机制也都是围绕着保护 APIServer 进行设计的，正如我们第 18 讲介绍的认证（Authentication）、鉴权（Authorization）和准入控制（Admission Control），这三大机制保护了 APIServer 的安全。</p>
<p data-nodeid="1037">显而易见，APIServer 也必须得使用 TLS 进行通信，尽量不要开启不安全的 HTTP 方式，尤其是在云上的环境中，切记一定要关闭，你可以通过<code data-nodeid="1144" data-backticks="1">--insecure-port=0</code>参数来关闭。</p>
<p data-nodeid="1038">同时要避免使用 AlwaysAllow 这种鉴权模式，这种模式会允许所有请求。一般来说，我建议这么配置鉴权模式，即<code data-nodeid="1147" data-backticks="1">--authorization-mode=RBAC,Node</code>。RBAC（基于角色的访问控制）会将传入的用户/组与一组绑定到角色的权限进行匹配。这些权限将 HTTP 请求的动作（GET，POST，DELETE）和 Kubernetes 内部各种资源对象，比如 Pod、Service 或者 Node，在命名空间或者集群范围内有机地结合起来，你可以回顾我们第 18 讲的内容，这里不再赘述。</p>
<h4 data-nodeid="1039">4. 减少集群的端口暴露</h4>
<p data-nodeid="4819">当然你还需要尽可能减少集群的端口暴露。如下是 Kubernetes 各组件需要的端口，在使用的时候，可以限制只允许固定节点对这些端口的访问。</p>
<p class="te-preview-highlight" data-nodeid="4820"><img alt="图片1.png" src="https://s0.lgstatic.com/i/image/M00/68/86/Ciqc1F-j-c-AbItzAAERT0Op8K0385.png" data-nodeid="4824"></p>











<h4 data-nodeid="1043">5. 限制对 Kubelet 的访问</h4>
<p data-nodeid="1044">Kubelet 上运行着我们的业务容器，同时还暴露了 10250 端口，可以用来查询容器，支持容器 exec，获取容器日志等功能。因此在生产级别的集群，我们要启用 <a href="https://kubernetes.io/zh/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/" data-nodeid="1162">Kubelet 身份验证和授权</a>，同时关闭匿名访问<code data-nodeid="1164" data-backticks="1">--anonymous-auth=false</code>。</p>
<h4 data-nodeid="1045">6. 开启审计能力</h4>
<p data-nodeid="1046">APIServer 支持对所有的请求进行<a href="https://kubernetes.io/zh/docs/tasks/debug-application-cluster/audit/" data-nodeid="1172">审计</a>（Audit），你可以通过<code data-nodeid="1174" data-backticks="1">--audit-log-path</code>来指定用来写入审计事件的日志文件路径，默认是不指定的。通过这个开启审计能力，再结合 RBAC，我们可以对整个集群的请求访问进行详细的监控和分析。</p>
<p data-nodeid="1047">这个审计功能提供了与安全相关的，按时间顺序排列的记录集，每一个请求都会有对应的审计事件，记录着：</p>
<ul data-nodeid="1048">
<li data-nodeid="1049">
<p data-nodeid="1050">发生了什么？</p>
</li>
<li data-nodeid="1051">
<p data-nodeid="1052">什么时候发生的？</p>
</li>
<li data-nodeid="1053">
<p data-nodeid="1054">谁触发的？</p>
</li>
<li data-nodeid="1055">
<p data-nodeid="1056">活动发生在哪个（些）对象上？</p>
</li>
<li data-nodeid="1057">
<p data-nodeid="1058">在哪观察到的？</p>
</li>
<li data-nodeid="1059">
<p data-nodeid="1060">它从哪触发的？</p>
</li>
<li data-nodeid="1061">
<p data-nodeid="1062">活动的后续处理行为是什么？</p>
</li>
</ul>
<p data-nodeid="1063">通过这些审计事件，集群管理员可以很容易地分析出集群内的安全风险，以采取应对策略。</p>
<h4 data-nodeid="1064">7. 通过 namespace 进行隔离</h4>
<p data-nodeid="1065">通过 namespace 来隔离工作负载和数据，比如区分不用的用户，不同的业务场景，以及一些关键的业务。你可以通过对这些 namespace 内的资源设置一些 RBAC 的规则，来进一步增强安全性。</p>
<p data-nodeid="1066"><img alt="Drawing 3.png" src="https://s0.lgstatic.com/i/image/M00/68/7C/Ciqc1F-j3vOAFJcHAAF4VRsEQdc795.png" data-nodeid="1191"></p>
<h4 data-nodeid="1067">8. 使用网络策略进行限制</h4>
<p data-nodeid="1068">Kubernetes 提供了基于 namespace 的<a href="https://kubernetes.io/zh/docs/tasks/administer-cluster/declare-network-policy/" data-nodeid="1198">网络策略</a>（Network Policy），可以允许和限制不同 namespace 中的 Pod 互访。</p>
<p data-nodeid="1069">网络策略通过<a href="https://kubernetes.io/zh/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/" data-nodeid="1203">网络插件</a>来实现，所以你要使用这种能力，要选择支持 NetworkPolicy 的网络解决方案，比如 <a href="https://kubernetes.io/zh/docs/tasks/administer-cluster/network-policy-provider/calico-network-policy/" data-nodeid="1207">Calico</a>、<a href="https://kubernetes.io/zh/docs/tasks/administer-cluster/network-policy-provider/cilium-network-policy/" data-nodeid="1211">Cilium</a>、<a href="https://kubernetes.io/zh/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/" data-nodeid="1215">Weave</a> 等。</p>
<h4 data-nodeid="1070">9. 容器镜像安全</h4>
<p data-nodeid="1071">容器的运行依赖容器的镜像，保证镜像安全是最基本的。要避免使用一些未知来源的镜像，同时自己构建镜像的时候，要使用安全的、确知的、官方的基础镜像。</p>
<p data-nodeid="1072">同时，要定期地对镜像进行漏洞扫描，你可以使用 <a href="https://github.com/aquasecurity/harbor-scanner-aqua" data-nodeid="1224">harbor-scanner-aqua</a>、<a href="https://github.com/quay/clair" data-nodeid="1228">Clair</a> 等工具对你的镜像中心、镜像进行扫描，来查找已知的漏洞。</p>
<p data-nodeid="1073">除此之外，我们还要限制特权容器，避免在容器镜像中使用 root 用户，防止特权升级。</p>
<h4 data-nodeid="1074">10. 应用程序的安全性</h4>
<p data-nodeid="1075">应用程序自身的安全性也很重要。如果你的应用程序要暴露给集群外部，可以使用<a href="https://kubernetes.io/zh/docs/concepts/services-networking/ingress-controllers/" data-nodeid="1237">入口控制器</a>（Ingress Controller），比如 <a href="https://git.k8s.io/ingress-nginx/README.md" data-nodeid="1241">Nginx</a>。</p>
<p data-nodeid="1076">应用容器也要使用 TLS 进行通信，包括跟入口控制器之间的通信。不需要定期扫描源代码及依赖项，查找是否有新漏洞，还要持续测试你的应用程序，查看是否会受到攻击，比如 SQL 注入，DDoS 攻击等。</p>
<h3 data-nodeid="1077">使用 kube-bench 对 Kubernetes 集群的安全性进行检测</h3>
<p data-nodeid="1078">这里我要跟你介绍一个开源工具<a href="https://github.com/aquasecurity/kube-bench" data-nodeid="1248">kube-bench</a>，它能帮助你排查 95% Kubernetes 集群的配置风险。这里我推荐你使用 kube-bench 对你的Kubernetes 集群进行一次全面的安全性检测，并根据反馈采取安全加固措施。</p>
<p data-nodeid="1079"><img alt="Drawing 4.png" src="https://s0.lgstatic.com/i/image/M00/68/87/CgqCHl-j3xiAe4TMAAFscw-4NYk411.png" data-nodeid="1252"></p>
<p data-nodeid="1080">（<a href="https://github.com/aquasecurity/kube-bench%EF%BC%89" data-nodeid="1256">https://github.com/aquasecurity/kube-bench）</a></p>
<p data-nodeid="1081">这个工具使用起来也非常方便快捷，你可以参考<a href="https://github.com/aquasecurity/kube-bench" data-nodeid="1260">官方文档</a>进行操作。</p>
<h3 data-nodeid="1082">写在最后</h3>
<p data-nodeid="1083">在云原生时代，拥有强大的安全非常重要，我们不能掉以轻心。在一开始我们就应该尽量把安全性纳入开发周期，将安全风险消灭在萌芽状态。</p>
<p data-nodeid="1084">好的，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **强：
> "应用容器也要使用 TLS 进行通信，包括跟入口控制器之间的通信。不需要定期扫描源代码及依赖项，查找是否有新漏洞，还要持续测试你的应用程序，查看是否会受到攻击，比如 SQL 注入，DDoS 攻击等。"不需要 还是 需要?

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 已修正 ，感谢您的指正，“不需要”——>“不仅需要”

