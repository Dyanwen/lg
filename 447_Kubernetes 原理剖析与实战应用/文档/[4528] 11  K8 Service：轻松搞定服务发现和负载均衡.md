<p data-nodeid="12217">经过前面几节课的学习，我们已经可以发布高可用的业务了，通过 PV 持久化地保存数据，通过 Deployment或Statefulset 这类工作负载来管理多实例，从而保证服务的高可用。</p>



<p data-nodeid="11644">想一想，这个时候如果有别的应用来访问我们的服务的话，该怎么办呢？直接访问后端的 Pod IP 吗？不，这里我们还需要做服务发现（Service Discovery）。</p>
<h3 data-nodeid="11645">为什么需要服务发现？</h3>
<p data-nodeid="11646">传统的应用部署，服务实例的网络位置是固定的，即在给定的机器上进行部署，这个时候的服务地址一般是机器的 IP 加上某个特定的端口号。</p>
<p data-nodeid="11647">但是在 Kubernetes 中，这是完全不同的。业务都是通过 Pod 来承载的，每个 Pod 的生命周期又很短暂，用后即焚，IP 地址也都是随机分配，动态变化的。而且，我们还经常会遇到一些高并发的流量进来，这时候往往需要快速扩容，服务的实例数也会随之动态调整。因此我们在这里就不能用传统的基于 IP 的方式去访问某个服务了。这个对于所有云上的系统，以及微服务应用体系，都是一个大难题。这时我们就需要做服务发现来确定服务的访问地址。</p>
<p data-nodeid="11648">今天我们就来聊聊 Kubernetes 中的服务发现 —— Service。</p>
<h3 data-nodeid="11649">Kubernetes 中的 Service</h3>
<p data-nodeid="12969">在之前的课程中，我们知道 Deployment、StatefulSet 这类工作负载都是通过 labelSelector 来管理一组 Pod 的。那么 Kubernetes 中的 Service 也采用了同样的做法，如下图。</p>
<p data-nodeid="12970" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/EA/Ciqc1F9sPFSAMnmPAAO5U6ZAsB0716.png" alt="image (4).png" data-nodeid="12978"><br>
（<a href="https://platform9.com/wp-content/uploads/2019/05/kubernetes-service-discovery.jpg" data-nodeid="12983">https://platform9.com/wp-content/uploads/2019/05/kubernetes-service-discovery.jpg</a>）</p>




<p data-nodeid="11653">这样一个 Service 会选择集群所有 label 中带有<code data-backticks="1" data-nodeid="11730">app=nginx</code>和<code data-backticks="1" data-nodeid="11732">env=prod</code>的 Pod。</p>
<p data-nodeid="11654">我们来看看这样的一个 Service 是如何定义的：</p>
<pre class="lang-yaml" data-nodeid="11655"><code data-language="yaml"><span class="hljs-string">$</span> <span class="hljs-string">cat</span> <span class="hljs-string">nginx-svc.yaml</span>
<span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Service</span>
<span class="hljs-attr">metadata:</span>  
  <span class="hljs-attr">name:</span> <span class="hljs-string">nginx-prod-svc-demo</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span> <span class="hljs-comment"># service 是 namespace 级别的对象</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>     &nbsp; <span class="hljs-comment"># Pod选择器</span>
    <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
    <span class="hljs-attr">env:</span> <span class="hljs-string">prod</span>
  <span class="hljs-attr">type:</span> <span class="hljs-string">ClusterIP</span> <span class="hljs-comment"># service 的类型</span>
  <span class="hljs-attr">ports:</span>  
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">http</span> 
    <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>       <span class="hljs-comment"># service 的端口号</span>
    <span class="hljs-attr">targetPort:</span> <span class="hljs-number">80</span> <span class="hljs-comment"># 对应到 Pod 上的端口号</span>
    <span class="hljs-attr">protocol:</span> <span class="hljs-string">TCP</span>  <span class="hljs-comment"># 还支持 udp，http 等</span>
</code></pre>
<p data-nodeid="11656">现在我们先来看如下一个 Deployment的定义：</p>
<pre class="lang-yaml" data-nodeid="11657"><code data-language="yaml"><span class="hljs-string">$</span> <span class="hljs-string">cat</span> <span class="hljs-string">nginx-deploy.yaml</span>
<span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span>
<span class="hljs-attr">metadata:</span>
&nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">nginx-prod-deploy</span>
&nbsp; <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span>
&nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; <span class="hljs-attr">env:</span> <span class="hljs-string">prod</span>
<span class="hljs-attr">spec:</span>
&nbsp; <span class="hljs-attr">replicas:</span> <span class="hljs-number">3</span>
&nbsp; <span class="hljs-attr">selector:</span>
&nbsp; &nbsp; <span class="hljs-attr">matchLabels:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">env:</span> <span class="hljs-string">prod</span>
&nbsp; <span class="hljs-attr">template:</span>
&nbsp; &nbsp; <span class="hljs-attr">metadata:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">env:</span> <span class="hljs-string">prod</span>
&nbsp; &nbsp; <span class="hljs-attr">spec:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">containers:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">image:</span> <span class="hljs-string">nginx:1.14.2</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">ports:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="11658">我们创建好这个 Deployment后，查看其 Pod 状态：</p>
<pre class="lang-shell" data-nodeid="11659"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get deploy -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; READY&nbsp; &nbsp;UP-TO-DATE&nbsp; &nbsp;AVAILABLE&nbsp; &nbsp;AGE
nginx-prod-deploy&nbsp; &nbsp;3/3&nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;5s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -o wide</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE&nbsp; &nbsp;IP&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NODE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;NOMINATED NODE&nbsp; &nbsp;READINESS GATES
nginx-prod-deploy-6fb6fbb77d-h2gn4&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 87s&nbsp; &nbsp;10.1.0.31&nbsp; &nbsp;docker-desktop&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
nginx-prod-deploy-6fb6fbb77d-r78k9&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 87s&nbsp; &nbsp;10.1.0.29&nbsp; &nbsp;docker-desktop&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
nginx-prod-deploy-6fb6fbb77d-xm8tp&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 87s&nbsp; &nbsp;10.1.0.30&nbsp; &nbsp;docker-desktop&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
</code></pre>
<p data-nodeid="11660">我们再来创建下上面定义的 Service：</p>
<pre class="lang-shell" data-nodeid="11661"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create -f nginx-svc.yaml</span>
service/nginx-prod-svc-demo created
<span class="hljs-meta">$</span><span class="bash"> kubectl get svc -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TYPE&nbsp; &nbsp; &nbsp; &nbsp; CLUSTER-IP&nbsp; &nbsp; &nbsp; &nbsp;EXTERNAL-IP&nbsp; &nbsp;PORT(S)&nbsp; &nbsp;AGE
nginx-prod-svc-demo&nbsp; &nbsp;ClusterIP&nbsp; &nbsp;10.111.193.186&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP&nbsp; &nbsp; 6s
可以看到，这个 Service 分配到了一个地址为10.111.193.186的 Cluster IP，这是一个虚拟 IP（VIP）地址，集群内所有的 Pod 和 Node 都可以通过这个虚拟 IP 地址加端口的方式来访问该 Service。这个 Service 会根据标签选择器，把匹配到的 Pod 的 IP 地址都挂载到后端。我们使用`kubectl describe`来看看这个 Service：
<span class="hljs-meta">$</span><span class="bash"> kubectl describe svc -n demo nginx-prod-svc-demo</span>
Name:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; nginx-prod-svc-demo
Namespace:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;demo
Labels:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;none&gt;
Annotations:&nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
Selector:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; app=nginx,env=prod
Type:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ClusterIP
IP:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 10.111.193.186
Port:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; http&nbsp; 80/TCP
TargetPort:&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP
Endpoints:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;10.1.0.29:80,10.1.0.30:80,10.1.0.31:80
Session Affinity:&nbsp; None
Events:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;none&gt;
</code></pre>
<p data-nodeid="13369">可以看到，这时候 Service 关联的 Endpoints 里面有三个 IP 地址，和我们上面看到的 Pod IP 地址完全吻合。</p>
<p data-nodeid="13370">我们试着来缩容 Deployment 的副本数，再来看看 Service 关联的 Pod IP 地址有什么变化：</p>

<pre class="lang-shell" data-nodeid="11663"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl scale --replicas=2 deploy -n demo nginx-prod-deploy</span>
deployment.apps/nginx-prod-deploy scaled
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -o wide</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE&nbsp; &nbsp;IP&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NODE&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;NOMINATED NODE&nbsp; &nbsp;READINESS GATES
nginx-prod-deploy-6fb6fbb77d-r78k9&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 11m&nbsp; &nbsp;10.1.0.29&nbsp; &nbsp;docker-desktop&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
nginx-prod-deploy-6fb6fbb77d-xm8tp&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 11m&nbsp; &nbsp;10.1.0.30&nbsp; &nbsp;docker-desktop&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
<span class="hljs-meta">$</span><span class="bash"> kubectl describe svc -n demo nginx-prod-svc-demo</span>
Name:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; nginx-prod-svc-demo
Namespace:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;demo
Labels:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;none&gt;
Annotations:&nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
Selector:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; app=nginx,env=prod
Type:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ClusterIP
IP:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 10.111.193.186
Port:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; http&nbsp; 80/TCP
TargetPort:&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP
Endpoints:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;10.1.0.29:80,10.1.0.30:80
Session Affinity:&nbsp; None
Events:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &lt;none&gt;
</code></pre>
<p data-nodeid="11664">可见当 Pod 的生命周期发生变化时，比如缩容或者异常退出，Service 会自动把有问题的 Pod 从后端地址中摘除。这样实现的好处在于，我们可以始终通过一个虚拟的稳定 IP 地址来访问服务，而不用关心其后端真正实例的变化。<br>
Kubernetes 中 Service 一共有四种类型，除了上面讲的 ClusterIP，还有 NodePort、LoadBalancer 和 ExternalName。</p>
<p data-nodeid="11665">其中 LoadBalancer 在云上用的较多，使用的时候需要跟各家云厂商做适配，比如部署对应的 cloud-controller-manager。有兴趣的话，可以查看<a href="https://kubernetes.io/zh/docs/concepts/services-networking/service/#loadbalancer" data-nodeid="11747">这个文档</a>，看看如何在云上使用。LoadBalancer主要用于做外部的服务发现，即暴露给集群外部的访问。</p>
<p data-nodeid="11666">ExternalName 类型的 Service 在实际中使用的频率不是特别高，但是对于某些特殊场景还是有一些用途的。比如在云上或者内部已经运行着一个应用服务，但是暂时没有运行在 Kubernetes 中，如果想让在 Kubernetes 集群中的 Pod 访问该服务，这时当然可以直接使用它的域名地址，也可以通过 ExternalName 类型的 Service 来解决。这样就可以直接访问 Kubernetes 内部的 Service 了。</p>
<p data-nodeid="11667">这样一来方便后续服务迁移到 Kubernetes 中，二来也方便随时切换到备份的服务上，而不用更改 Pod 内的任何配置。由于使用频率并不高，我们不做重点介绍，有兴趣可以参考这篇<a href="https://kubernetes.io/zh/docs/concepts/services-networking/service/#externalname" data-nodeid="11753">文档</a>。</p>
<p data-nodeid="11668">我们最后来看下另外一种 NodePort 类型的 Service：</p>
<pre class="lang-yaml" data-nodeid="11669"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Service</span>
<span class="hljs-attr">metadata:</span>  
  <span class="hljs-attr">name:</span> <span class="hljs-string">my-nodeport-service</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">selector:</span>    
    <span class="hljs-attr">app:</span> <span class="hljs-string">my-app</span>
  <span class="hljs-attr">type:</span> <span class="hljs-string">NodePort</span> <span class="hljs-comment"># 这里设置类型为 NodePort</span>
  <span class="hljs-attr">ports:</span>  
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">http</span>
    <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
    <span class="hljs-attr">targetPort:</span> <span class="hljs-number">80</span>
    <span class="hljs-attr">nodePort:</span> <span class="hljs-number">30000</span>
    <span class="hljs-attr">protocol:</span> <span class="hljs-string">TCP</span>
</code></pre>
<p data-nodeid="14142">顾名思义，这种类型的 Service 通过任一 Node 节点的 IP 地址，再加上端口号就可以访问 Service 后端负载了。我们看下面这个流量图，方便理解。</p>
<p data-nodeid="14143" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/F5/CgqCHl9sPGuAbYFBAAR9dQ-LWLw022.png" alt="image (5).png" data-nodeid="14151"><br>
（<a href="https://miro.medium.com/max/1680/1" data-nodeid="14156">https://miro.medium.com/max/1680/1</a>*CdyUtG-8CfGu2oFC5s0KwA.png）</p>




<p data-nodeid="11672">NodePort 类型的 Service 创建好了以后，Kubernetes 会在每个 Node 节点上开个端口，比如这里的 30000 端口。这个时候我们可以访问任何一个 Node 的 IP 地址，通过 30000 端口即可访问该服务。</p>
<p data-nodeid="11673">那么如果在集群内部，该如何访问这些 Service 呢？</p>
<h3 data-nodeid="11674">集群内如何访问 Service？</h3>
<p data-nodeid="11675">一般来说，在 Kubernetes 集群内，我们有两种方式可以访问到一个 Service。</p>
<ol data-nodeid="11676">
<li data-nodeid="11677">
<p data-nodeid="11678">如果该 Service 有 ClusterIP，我们就可以直接用这个虚拟 IP 去访问。比如我们上面创建的 nginx-prod-svc-demo 这个 Service，我们通过<code data-backticks="1" data-nodeid="11772">kubectl get svc nginx-prod-svc-demo -n dmeo</code>或<code data-backticks="1" data-nodeid="11774">kubectl get svc nginx-prod-svc-demo -n dmeo</code>就可以看到其 Cluster IP 为  10.111.193.186，端口号为 80。那么我们通过  http(s)://10.111.193.186:80 就可以访问到该服务。</p>
</li>
<li data-nodeid="11679">
<p data-nodeid="11680">当然我们也可以使用该 Service 的域名，依赖于集群内部的 DNS 即可访问。还是以上面的例子做说明，同 namespace 下的 Pod 可以直接通过 nginx-prod-svc-demo 这个 Service 名去访问。如果是不同 namespace 下的 Pod 则需要加上该 Service 所在的 namespace 名，即<code data-backticks="1" data-nodeid="11777">nginx-prod-svc-demo.demo</code>去访问。</p>
</li>
</ol>
<p data-nodeid="11681">如果在某个 namespace 下，Service 先于 Pod 创建出来，那么 kubelet 在创建 Pod 的时候，会自动把这些 namespace 相同的 Service 访问信息当作环境变量注入  Pod 中，即<code data-backticks="1" data-nodeid="11780">{SVCNAME}_SERVICE_HOST</code>和<code data-backticks="1" data-nodeid="11782">{SVCNAME}_SERVICE_PORT</code>。这里<code data-backticks="1" data-nodeid="11784">SVCNAME</code>对应是各个 Service 的大写名称，名字中的横线会被自动转换成下划线。比如：</p>
<pre class="lang-shell" data-nodeid="11682"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> env</span>
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=nginx-prod-deploy2-68d8fb9586-4m5hr
NGINX_PROD_SVC_DEMO_SERVICE_PORT_HTTP=80
NGINX_PROD_SVC_DEMO_SERVICE_HOST=10.111.193.186
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
NGINX_PROD_SVC_DEMO_SERVICE_PORT=80
NGINX_PROD_SVC_DEMO_PORT=tcp://10.111.193.186:80
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NGINX_PROD_SVC_DEMO_PORT_80_TCP_ADDR=10.111.193.186
NGINX_PROD_SVC_DEMO_PORT_80_TCP_PORT=80
NGINX_PROD_SVC_DEMO_PORT_80_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
NGINX_PROD_SVC_DEMO_PORT_80_TCP=tcp://10.111.193.186:80
...
</code></pre>
<p data-nodeid="11683">知道了这两种访问方式，我们就可以在启动 Pod 的时候，通过注入环境变量、启动参数或者挂载配置文件等方式，来指定要访问的 Service 信息。如果是同 namespace 的 Pod，可以直接从自己的环境变量中知道同 namespace 下的其他 Service 的访问方式。</p>
<p data-nodeid="11684">那么这样通过该 Service 进行访问时，Kubernetes 又是如何实现负载均衡的呢，即将流量打到后端挂载的各个 Pod 上面去？</p>
<h3 data-nodeid="11685">集群内部的负载均衡如何实现？</h3>
<p data-nodeid="11686">这一切都是通过 kube-proxy 来实现的。所有的节点上都会运行着一个 kube-proxy的服务，主要监听 Kubernetes 中的 Service 和 Endpoints。当 Service 或 Endpoints 发生变化时，就会调用相应的接口创建对应的规则出来，常用模式主要是 iptables 模式和 IPVS 模式。iptables 模式比较简单，使用起来也方便。而 IPVS 支持更高的吞吐量以及复杂的负载均衡策略，你可以通过<a href="https://kubernetes.io/zh/docs/concepts/services-networking/service/#proxy-mode-ipvs" data-nodeid="11792">官方文档</a>了解更多 IPVS 模式的工作原理。</p>
<p data-nodeid="14941">目前 kube-proxy 默认的工作方式是 iptables 模式，我们来通过如下一个 iptables 模式的例子来看一下实际访问链路是什么样的。</p>
<p data-nodeid="14942" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/F5/CgqCHl9sPH6AMn4jAA32mDhWECM868.png" alt="image (6).png" data-nodeid="14950"><br>
（<a href="https://d33wubrfki0l68.cloudfront.net/27b2978647a8d7bdc2a96b213f0c0d3242ef9ce0/e8c9b/images/docs/services-iptables-overview.svg" data-nodeid="14955">https://d33wubrfki0l68.cloudfront.net/27b2978647a8d7bdc2a96b213f0c0d3242ef9ce0/e8c9b/images/docs/services-iptables-overview.svg</a>）</p>




<p data-nodeid="11690">当你通过 Service 的域名去访问时，会先通过 CoreDNS 解析出 Service 对应的 Cluster IP，即虚拟 IP。然后请求到达宿主机的网络后，就会被kube-proxy所配置的 iptables 规则所拦截，之后请求会被转发到每一个实际的后端 Pod 上面去，这样就实现了负载均衡。</p>
<h3 data-nodeid="11691">Headless Service</h3>
<p data-nodeid="11692">如果我们在定义 Service 的时候，将spec.clusterIP设置为 None，这个时候创建出来的 Service 并不会分配到一个 Cluster IP，此时它就被称为Headless Service。</p>
<p data-nodeid="11693">现在我们来通过一个例子来看看 Headless Service 有什么特殊的地方。我们在上面的 Service 基础上，增加了spec.clusterIP为None，并命名为nginx-prod-demo-headless-svc：</p>
<pre class="lang-yaml" data-nodeid="11694"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Service</span>
<span class="hljs-attr">metadata:</span>  
  <span class="hljs-attr">name:</span> <span class="hljs-string">nginx-prod-demo-headless-svc</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span> 
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">clusterIP:</span> <span class="hljs-string">None</span>
  <span class="hljs-attr">selector:</span> 
    <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
    <span class="hljs-attr">env:</span> <span class="hljs-string">prod</span>
  <span class="hljs-attr">type:</span> <span class="hljs-string">ClusterIP</span>
  <span class="hljs-attr">ports:</span>  
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">http</span> 
    <span class="hljs-attr">port:</span> <span class="hljs-number">80</span> 
    <span class="hljs-attr">targetPort:</span> <span class="hljs-number">80</span> 
    <span class="hljs-attr">protocol:</span> <span class="hljs-string">TCP</span> 
</code></pre>
<p data-nodeid="11695">通过 kubectl 创建成功后，我们现在<code data-backticks="1" data-nodeid="11808">kubectl get</code>一下看看：</p>
<pre class="lang-shell" data-nodeid="11696"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get svc -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;TYPE&nbsp; &nbsp; &nbsp; &nbsp; CLUSTER-IP&nbsp; &nbsp; &nbsp; &nbsp;EXTERNAL-IP&nbsp; &nbsp;PORT(S)&nbsp; &nbsp;AGE
nginx-prod-demo-headless-svc&nbsp; &nbsp;ClusterIP&nbsp; &nbsp;None&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP&nbsp; &nbsp; 4s
nginx-prod-svc-demo&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ClusterIP&nbsp; &nbsp;10.111.193.186&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP&nbsp; &nbsp; 3d5h
</code></pre>
<p data-nodeid="15357">可以看到这个叫 nginx-prod-demo-headless-svc 的 Service 并没有分配到一个 ClusterIP，符合预期，毕竟我们已经设置了 spec.clusterIP 为 None。</p>
<p data-nodeid="15358">我们来创建一个 Pod，看看 DNS 记录有没有什么差别。 Pod 的 yaml 文件如下：</p>

<pre class="lang-yaml" data-nodeid="11698"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Pod</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">headless-svc-test-pod</span>
  <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">containers:</span>
  <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">dns-test</span>
    <span class="hljs-attr">image:</span> <span class="hljs-string">busybox:1.28</span>
    <span class="hljs-attr">command:</span> [<span class="hljs-string">'sh'</span>, <span class="hljs-string">'-c'</span>, <span class="hljs-string">'echo The app is running! &amp;&amp; sleep 3600'</span>]
</code></pre>
<p data-nodeid="11699">该 Pod 创建出来后，我们通过<code data-backticks="1" data-nodeid="11814">kubectl exec</code>进入 Pod 中，运行如下两条 nslookup 查询命令，依次查看两个 Service 对应的 DNS 记录：</p>
<pre class="lang-shell" data-nodeid="11700"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl <span class="hljs-built_in">exec</span> -it -n demo headless-svc-test-pod sh</span>
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
/ # nslookup nginx-prod-demo-headless-svc
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:&nbsp; &nbsp; &nbsp; nginx-prod-demo-headless-svc
Address 1: 10.1.0.32 10-1-0-32.nginx-prod-demo-headless-svc.demo.svc.cluster.local
Address 2: 10.1.0.33 10-1-0-33.nginx-prod-demo-headless-svc.demo.svc.cluster.local
/ # nslookup nginx-prod-svc-demo
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:&nbsp; &nbsp; &nbsp; nginx-prod-svc-demo
Address 1: 10.111.193.186 nginx-prod-svc-demo.demo.svc.cluster.local
</code></pre>
<p data-nodeid="11701">我们可以看到正常 Service<code data-backticks="1" data-nodeid="11817">nginx-prod-svc-demo</code>对应的 DNS 记录的是与虚拟 IP<code data-backticks="1" data-nodeid="11819">10.111.193.166</code>有关的记录，而 Headless Service<code data-backticks="1" data-nodeid="11821">nginx-prod-demo-headless-svc</code>则解析到所有后端的 Pod 的地址。<br>
总结下， Headless Service 主要有如下两种场景。</p>
<ol data-nodeid="11702">
<li data-nodeid="11703">
<p data-nodeid="11704">用户可以自己选择要连接哪个 Pod，通过查询 Service 的 DNS 记录来获取后端真实负载的 IP 地址，自主选择要连接哪个 IP；</p>
</li>
<li data-nodeid="11705">
<p data-nodeid="11706">可用于部署有状态服务。回顾下，我们在 StatefulSet 那节课也有 Headless Service 例子，每个 StatefulSet 管理的 Pod 都有一个单独的 DNS 记录，且域名保持不变，即<code data-backticks="1" data-nodeid="11827">&lt;PodName&gt;.&lt;ServiceName&gt;.&lt;NamespaceName&gt;.svc.cluster.local</code>。这样 Statefulset 中的各个 Pod 就可以直接通过 Pod 名字解决相互间身份以及访问问题。</p>
</li>
</ol>
<h3 data-nodeid="11707">写在最后</h3>
<p data-nodeid="11708">Service 是 Kubernetes 很重要的对象，主要负责为各种工作负载暴露服务，方便各个服务之间互访。通过对一组 Pod 提供统一入口，Service 极大地方便了用户使用，用户只需要与 Service 打交道即可，而不用过多地关心后端实例的变动，比如扩缩容、容器异常、节点宕机，等等。</p>
<p data-nodeid="11709">正是因为有了 Service 的支持，你在 Kubernetes 部署业务会非常方便，这是相比较于 Docker Swarm 以及 Mesos Marathon 巨大的技术优势，可以说，它是 Kubernetes 是运行大规模微服务的最佳载体。</p>
<p data-nodeid="11710">好的，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### *星：
> 请教下老师，服务A使用nodeport访问，服务B使用host模式，如何访问，宿主的ip不是固定的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; nodeport的话，可以通过任一宿主机ip+port 的方式来访问。所以服务A 和服务B可以通过任一宿主机ip来访问。

##### *悦：
> apiVersion: v1kind: Podmetadata: name: headless-svc-test-pod namespace: demospec: containers: - name: dns-test image: busybox:1.28 sleep 3600']这个怎么和headless server 匹配麻烦老师说下

##### *悦：
> apiVersion: v1kind: Servicemetadata:  name: nginx-prod-demo-headless-svcspec: clusterIP: None app: nginx env: prod type: ClusterIP——pod怎么匹配的 pod没有labels标签怎么匹配

##### **彬：
> 老师，你好，请教一下，配置clusterip时，通过coredns解析到虚拟IP，请求是怎么转发到宿主机上的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这部分官方文档讲解的比较清楚，还是图。https://kubernetes.io/zh/docs/concepts/services-networking/service/#proxy-mode-iptables

