<p data-nodeid="1013">Ingress 是 Kubernetes 集群为了让外部可以访问，引入的一种资源类型。一般 Ingress 为常见的 Nginx 这样的反向代理服务器提供具体功能，诸如负载均衡、SSL 证书卸载，以及基于名称的虚拟主机功能。这些功能是反向代理服务器的基本功能，</p>
<p data-nodeid="1014"><strong data-nodeid="1152">Ingress 可以理解为入口网关</strong>，而 Egress 和 Ingress 的功能相仿，只是流量的代理流向不同，<strong data-nodeid="1153">Egress 负责出口流量的代理</strong>。</p>
<p data-nodeid="1015">在讲解 Istio 的 Ingress 之前，我们还是先来看一下传统的 Kubernetes 是如何做 Ingress 代理的。</p>
<h3 data-nodeid="1016">为什么需要 Ingress</h3>
<p data-nodeid="1017">这里我们需要讲到一些 Kubernetes 的基本知识和概念，你可以把 Kubernetes 集群简单理解为一个内部网络，我们想要在集群外直接访问 Kubernetes 的 Pod IP 是不通的，只能通过一些代理手段<strong data-nodeid="1165">做一次路由转发才可以访问</strong>。一般来说有两种常用的方式访问 Kubernetes 集群内部的网络，分别是<strong data-nodeid="1166">NodePort 模式和 Ingress 模式</strong>。</p>
<p data-nodeid="1018">为了更好地理解今天要学习的内容，我们先简单聊一聊 Kubernetes 集群中的一些名词。</p>
<p data-nodeid="1019">Pod：Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署计算单元，每个 Pod 都有独立的 IP，Pod 内部由多个Container（容器）组成。</p>
<p data-nodeid="1020">Service：将运行在一组 Pods 上的应用程序公开为网络服务的抽象方法。Service 是一个抽象的概念，等同于微服务中的 Service 概念。</p>
<p data-nodeid="1021">Node：Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中执行工作负载。节点可以是一个虚拟机或者物理机器，主要取决于所在的集群配置。</p>
<p data-nodeid="1022">ClusterIP：Kubernetes 为了让 Service 可以进行内部访问，为每个 Service 提供了一个唯一 IP，通过 ClusterIP 可以访问 Service 对应的一组 Pod。</p>
<h4 data-nodeid="1023">NodePort 模式</h4>
<p data-nodeid="1024">刚刚我们提到过，集群外部是无法直接访问集群内部的资源的，在 Kubernetes 集群内部，我们可以<strong data-nodeid="1178">通过 Service 的域名访问服务的 Pod</strong>。下面是一个简单的 YAML 配置，这个配置可以让集群外部通过 NodePort 的方式访问集群内部的资源：</p>
<pre class="lang-yaml" data-nodeid="1025"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Service</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">my-service</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">type:</span> <span class="hljs-string">NodePort</span>
  <span class="hljs-attr">selector:</span>
    <span class="hljs-attr">app:</span> <span class="hljs-string">MyApp</span>
  <span class="hljs-attr">ports:</span>
      <span class="hljs-comment"># 默认情况下，为了方便起见，`targetPort` 被设置为与 `port` 字段相同的值</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
      <span class="hljs-attr">targetPort:</span> <span class="hljs-number">80</span>
      <span class="hljs-comment"># 可选字段</span>
      <span class="hljs-comment"># 默认情况下，为了方便起见，Kubernetes 控制平面会从某个范围内分配一个端口号（默认：30000-32767）</span>
      <span class="hljs-attr">nodePort:</span> <span class="hljs-number">30007</span>
</code></pre>
<p data-nodeid="1026">NodePort 的默认端口范围是 30000-32767，但是一般不建议应用程序使用这个端口范围，以免引发冲突。</p>
<p data-nodeid="1027">如上示例，集群外部首先访问 NodePort 的端口，示例中端口为 30007，<strong data-nodeid="1185">通过 IPVS 一系列的劫持路由操作</strong>，将流量路由到集群内部的 my-service 服务的端口 80，这个端口是 ClusterIP 暴露出来的端口，在 Kubernetes 集群内部可以通过 ClusterIP:Port 的方式访问 my-service 服务。</p>
<p data-nodeid="1028">至此，集群外部如何访问集群内部的整个流程就串起来了，而下面的 TargetPort 是服务自身的端口，接下来的 Kubernetes 集群内部会通<strong data-nodeid="1191">过服务发现的方式</strong>，将流量转发到服务对应的 Pod IP 的 TargetPort 上面。</p>
<p data-nodeid="1029">到这里，NodePort 的访问方式我们就讲完了。通过原理的学习你可以了解到，NodePort 的方式存在一些问题，比如<strong data-nodeid="1197">配置方式不灵活</strong>，需要前端的入口层将 upstream 的地址配置为 NodePort 的地址和端口，如果 Kubernetes 集群的 Node 节点所在的机器发生扩容或者缩容，都需要相应做出调整。</p>
<p data-nodeid="1030">另外，通过几台特定的 Node 机器做数据转发，一旦流量增加，可能会影响该机器宿主机的稳定性。如果集群对外暴露多个服务，维护的难度也随之增加，上面提到的扩缩容问题，会被数倍放大：<strong data-nodeid="1203">一台 Node 机器的变动，需要修改大量的入口服务配置</strong>。</p>
<p data-nodeid="1031">那么，有没有更好的方式解决 NodePort 的问题呢？接下来我们来看 Ingress 模式。</p>
<h4 data-nodeid="4865" class="">Ingress 模式</h4>




<p data-nodeid="1033">客户端通过 Ingress 上定义的路由规则，转发到特定的 Service 上面，比如通过设置 path、header、host 等路由规则来决定具体转发到哪个 Service。</p>
<p data-nodeid="6250" class=""><img src="https://s0.lgstatic.com/i/image6/M00/04/29/CioPOWAikmiAcbONAACHq-N5-0s752.png" alt="Drawing 0.png" data-nodeid="6254"></p>
<div data-nodeid="6251"><p style="text-align:center">Ingress 工作原理图</p></div>



<p data-nodeid="1036">通过下面的配置你可以看到，Ingress 是一种全新的资源类型，与所有其他 Kubernetes 资源一样，Ingress 需要使用 apiVersion、kind 和 metadata 字段。</p>
<pre class="lang-java" data-nodeid="1037"><code data-language="java">apiVersion: networking.Kubernetes.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="1038">我们来看一下， Ingress 资源类型特有的几个字段的含义。</p>
<ul data-nodeid="1039">
<li data-nodeid="1040">
<p data-nodeid="1041">host：可选择和域名匹配的 host，比如如果设置 host 字段 foo.bar.com，则需要通过 foo.bar.com 的域名访问。如果这个例子中没有设置 host 字段，则表明任何域名过来的请求都可以匹配。</p>
</li>
<li data-nodeid="1042">
<p data-nodeid="1043">http.paths.path：用于请求 path 的匹配，比如这个例子通过 pathType:Prefix 前缀匹配的方式，所有 path 中前缀为 /testpath 的路径，都会被匹配到；而其他没有匹配到的 path 就会返回 404。</p>
</li>
<li data-nodeid="1044">
<p data-nodeid="1045">http.paths.backend：其中 name 字段表示服务名，这个服务名和 Kubernetes 中的Service 名称相匹配；port 则是 Service 的端口号，流量会通过 clusterIP:port 的方式被转发到特定服务。</p>
</li>
</ul>
<h4 data-nodeid="1046">IngressClass 资源类型</h4>
<p data-nodeid="1047">在 Kubernetes1.18 版本之前，IngressClass 是通过 Ingress 中的一个 kubernetes.io/ingress.class 注解来指定的，下面我们来看一下从 1.18 开始如何配置 Ingress Class。</p>
<p data-nodeid="1048">通过在 Ingress 资源类型中<strong data-nodeid="1226">设置 IngressClassName</strong>来指定特定的 IngressClass 配置，IngressClass 的配置如下，其中包含了 ingress-controller 的名称，在这里 ingress-controller 名称为 example.com/ingress-controller：</p>
<pre class="lang-js" data-nodeid="6945"><code data-language="js">apiVersion: networking.k8s.io/v1
<span class="hljs-attr">kind</span>: IngressClass
<span class="hljs-attr">metadata</span>:
  name: external-lb
<span class="hljs-attr">spec</span>:
  controller: example.com/ingress-controller
  <span class="hljs-attr">parameters</span>:
    apiGroup: k8s.example.com
    <span class="hljs-attr">kind</span>: IngressParameters
    <span class="hljs-attr">name</span>: external-lb
</code></pre>

<p data-nodeid="1050"><strong data-nodeid="1230">ingress-controller</strong></p>
<p data-nodeid="1051">ingress-controller 并不是随着 Kubernetes 集群一起启动的，下面我们先来看一下如何在 Minukube 环境中使用 Nginx ingress-controller。</p>
<p data-nodeid="1052">启动 Minukube 集群，需要先删除已有的 Minukube 集群：</p>
<pre class="lang-java" data-nodeid="1053"><code data-language="java">minikube delete
</code></pre>
<p data-nodeid="1054">然后通过虚拟机方式启动，因为 Ingress 的 Addon 无法在 Docker 模式使用：</p>
<pre class="lang-java" data-nodeid="1055"><code data-language="java">minikube start --kubernetes-version=v1<span class="hljs-number">.19</span><span class="hljs-number">.2</span> --vm=<span class="hljs-keyword">true</span>
</code></pre>
<p data-nodeid="1056">启动 ingress-controller ：</p>
<pre class="lang-java" data-nodeid="1057"><code data-language="java">minikube addons enable ingress
</code></pre>
<p data-nodeid="1058">部署 helloapp：</p>
<pre class="lang-java" data-nodeid="1059"><code data-language="java">kubectl create deployment web --image=gcr.io/google-samples/hello-app:<span class="hljs-number">1.0</span>
</code></pre>
<p data-nodeid="1060">创建 Ingress 资源：</p>
<pre class="lang-java" data-nodeid="1061"><code data-language="java">kubectl apply -f https:<span class="hljs-comment">//k8s.io/examples/service/networking/example-ingress.yaml</span>
</code></pre>
<p data-nodeid="1062">通过 minikube iP 命令获取 Ip 地址，并修改 /etc/hosts，将其添加在文件结尾，代码中 127.0.0.1 就是 minikube ip 获取的 IP：</p>
<pre class="lang-java" data-nodeid="1063"><code data-language="java"><span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span> hello-world.info
</code></pre>
<p data-nodeid="1064">通过 cURL 或者浏览器访问 hello-world.info：</p>
<pre class="lang-java" data-nodeid="1065"><code data-language="java">curl hello-world.info
</code></pre>
<p data-nodeid="1066">至此，Kubernetes 原生的 Ingress 就讲完了。Ingress 解决了 NodePod 配置不方便的问题，但通过 YAML 的方式控制 Ingress 依然是一件麻烦事，另外 Ingress 内部依然是使用 ClusterIP 的方式来访问 Service，而这样的方式是通过 IPVS 四层转发做到的。</p>
<p data-nodeid="1067">在前面的章节中，我们也提到了四层负载均衡存在流量不均衡的问题，那么在 Ingress 中应该如何解决呢？下面我们来看一下 Service Mesh 世界中的 Istio 是如何做 Ingress 的。</p>
<h4 data-nodeid="1068">Istio Gateway</h4>
<p data-nodeid="1069">Istio 采用了一种新的模型——Istio Gateway 来代替 Kubernetes 中的 Ingress 资源类型。<strong data-nodeid="1247">Gateway 允许外部流量访问内部服务</strong>，得益于 Istio 强大的控制面配置，Gateway 资源类型的配置非常简单，只需要配置流量转发即可。</p>
<p data-nodeid="1070">先删除已有的 Minukube 集群：</p>
<pre class="lang-java" data-nodeid="1071"><code data-language="java">minikube delete
</code></pre>
<p data-nodeid="1072">首先启动 Minikube，并启动 Tunnel：</p>
<pre class="lang-java" data-nodeid="1073"><code data-language="java">minikube start --kubernetes-version=v1<span class="hljs-number">.19</span><span class="hljs-number">.2</span> --driver=docker
minikube tunnel
</code></pre>
<p data-nodeid="1074">这样外部就可以通过 Minikube IP 访问集群内部的资源了。</p>
<p data-nodeid="1075">下面进入 Istio 目录，部署一个测试服务：</p>
<pre class="lang-java" data-nodeid="1076"><code data-language="java">kubectl apply -f samples/httpbin/httpbin.yaml
</code></pre>
<p data-nodeid="1077">通过命令查看 Pod 是否成功启动，根据机器配置不同，这里的 Pod 启动可能需要一定的时间，请耐心等待 pod 启动成功：</p>
<pre class="lang-java" data-nodeid="1078"><code data-language="java">$ kubectl get pods
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp;RESTARTS&nbsp; &nbsp;AGE
details-v1-<span class="hljs-number">79</span>c697d759-gt9th&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">7</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
httpbin-<span class="hljs-number">74f</span>b669cc6-jzs87&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Init:<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">7</span>s
productpage-v1-<span class="hljs-number">65576</span>bb7bf-wjkjm&nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">7</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
ratings-v1-<span class="hljs-number">7</span>d99676f7f-h9blt&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">6</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
reviews-v1-<span class="hljs-number">987</span>d495c-pmnb9&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">7</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
reviews-v2-<span class="hljs-number">6</span>c5bf657cf-qd2kr&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">7</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
reviews-v3-<span class="hljs-number">5f</span>7b9f4f77-gnv4c&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">2</span>/<span class="hljs-number">2</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; <span class="hljs-number">7</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
</code></pre>
<p data-nodeid="1079">创建 Istio Gateway，注意：<strong data-nodeid="1258">这里设置了 hosts 为 httpbin.example.com</strong>，也就是只有 host 为httpbin.example.com 才能正确访问 httpbin 的服务：</p>
<pre data-nodeid="1080"><code>kubectl apply -f - &lt;&lt;EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: httpbin-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "httpbin.example.com"
EOF
</code></pre>
<p data-nodeid="1081">将 httpbin 服务暴露给 Istio Gateway，其中 destination 中的 host 字段为 Istio Service 配置中的服务名，Envoy 会通过<strong data-nodeid="1264">服务发现</strong>的方式将流量路由到 httpbin 对应的 Pod IP：</p>
<pre class="lang-java" data-nodeid="1082"><code data-language="java">kubectl apply -f - &lt;&lt;EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: httpbin
spec:
  hosts:
  - <span class="hljs-string">"httpbin.example.com"</span>
  gateways:
  - httpbin-gateway
  http:
  - match:
    - uri:
        prefix: /status
    - uri:
        prefix: /delay
    route:
    - destination:
        port:
          number: <span class="hljs-number">8000</span>
        host: httpbin
EOF
</code></pre>
<p data-nodeid="7636" class="">接下来，我们通过 cURL 访问特定的 URL Path 就可以访问该服务了，这里需要注意的是：<strong data-nodeid="7646">需要设置 host 为 httpbin.example.com</strong>。因为在前面的 Gateway 配置中，我们绑定了 host，当然你也可以通过<strong data-nodeid="7647">修改 /etc/hosts</strong> 来绑定域名为本地地址：</p>

<pre class="lang-java" data-nodeid="1084"><code data-language="java">curl -I -HHost:httpbin.example.com http:<span class="hljs-comment">//127.0.0.1/status/200</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
server: istio-envoy
date: Mon, <span class="hljs-number">08</span> Feb <span class="hljs-number">2021</span> <span class="hljs-number">04</span>:<span class="hljs-number">42</span>:<span class="hljs-number">54</span> GMT
content-type: text/html; charset=utf-<span class="hljs-number">8</span>
access-control-allow-origin: *
access-control-allow-credentials: <span class="hljs-keyword">true</span>
content-length: <span class="hljs-number">0</span>
x-envoy-upstream-service-time: <span class="hljs-number">327</span>
</code></pre>
<p data-nodeid="1085">如果没有匹配到路由规则，则会返回404：</p>
<pre class="lang-java" data-nodeid="1086"><code data-language="java">$ curl -I -HHost:httpbin.example.com http:<span class="hljs-comment">//127.0.0.1/1status/200</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">404</span> Not Found
date: Mon, <span class="hljs-number">08</span> Feb <span class="hljs-number">2021</span> <span class="hljs-number">04</span>:<span class="hljs-number">45</span>:<span class="hljs-number">50</span> GMT
server: istio-envoy
transfer-encoding: chunked
</code></pre>
<p data-nodeid="1087">通过命令查看 Envoy 日志，这里的 httpbin-74fb669cc6-jzs87 是通过上面 kubectl get pods 命令获取到的 Pod 名称，需要加上 -c container 的名称来查看 Envoy 的日志：</p>
<pre class="lang-java" data-nodeid="1088"><code data-language="java">&nbsp;kubectl logs&nbsp; httpbin-<span class="hljs-number">74f</span>b669cc6-jzs87 -c istio-proxy
</code></pre>
<p data-nodeid="1089">这里我列了几条刚才访问产生的 Envoy 日志，你可以看到有正确的 200 日志和错误的 404 日志：</p>
<pre class="lang-java" data-nodeid="1090"><code data-language="java"><span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">29</span>:<span class="hljs-number">17.701261</span>Z	info	Envoy proxy is ready
[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">42</span>:<span class="hljs-number">54.644</span>Z] <span class="hljs-string">"HEAD /status/200 HTTP/1.1"</span> <span class="hljs-number">200</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">225</span> <span class="hljs-number">199</span> <span class="hljs-string">"172.17.0.2"</span> <span class="hljs-string">"curl/7.54.0"</span> <span class="hljs-string">"be1d93ab-32b3-9882-8c34-87dd39ddf6ab"</span> <span class="hljs-string">"httpbin.example.com"</span> <span class="hljs-string">"127.0.0.1:80"</span> inbound|<span class="hljs-number">8000</span>|http|httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">34580</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.16</span>:<span class="hljs-number">80</span> <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span>:<span class="hljs-number">0</span> outbound_<span class="hljs-number">.8000</span>_._.httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-keyword">default</span>
[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">45</span>:<span class="hljs-number">14.862</span>Z] <span class="hljs-string">"HEAD /status2/200 HTTP/1.1"</span> <span class="hljs-number">404</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">351</span> <span class="hljs-number">337</span> <span class="hljs-string">"172.17.0.2"</span> <span class="hljs-string">"curl/7.54.0"</span> <span class="hljs-string">"b433ee7d-6f89-9854-bee7-eeed33884003"</span> <span class="hljs-string">"httpbin.example.com"</span> <span class="hljs-string">"127.0.0.1:80"</span> inbound|<span class="hljs-number">8000</span>|http|httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">36896</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.16</span>:<span class="hljs-number">80</span> <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span>:<span class="hljs-number">0</span> outbound_<span class="hljs-number">.8000</span>_._.httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-keyword">default</span>
[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">45</span>:<span class="hljs-number">57.688</span>Z] <span class="hljs-string">"HEAD /status/500 HTTP/1.1"</span> <span class="hljs-number">500</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">14</span> <span class="hljs-number">11</span> <span class="hljs-string">"172.17.0.2"</span> <span class="hljs-string">"curl/7.54.0"</span> <span class="hljs-string">"5ca2dbf4-473f-9dd0-97a0-397a98f2805c"</span> <span class="hljs-string">"httpbin.example.com"</span> <span class="hljs-string">"127.0.0.1:80"</span> inbound|<span class="hljs-number">8000</span>|http|httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">37604</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.16</span>:<span class="hljs-number">80</span> <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span>:<span class="hljs-number">0</span> outbound_<span class="hljs-number">.8000</span>_._.httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-keyword">default</span>
[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">46</span>:<span class="hljs-number">04.233</span>Z] <span class="hljs-string">"HEAD /status/400 HTTP/1.1"</span> <span class="hljs-number">400</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">3</span> <span class="hljs-number">2</span> <span class="hljs-string">"172.17.0.2"</span> <span class="hljs-string">"curl/7.54.0"</span> <span class="hljs-string">"4c6e9a80-74f4-98bb-ba3e-8140a4a16099"</span> <span class="hljs-string">"httpbin.example.com"</span> <span class="hljs-string">"127.0.0.1:80"</span> inbound|<span class="hljs-number">8000</span>|http|httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">37722</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.16</span>:<span class="hljs-number">80</span> <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span>:<span class="hljs-number">0</span> outbound_<span class="hljs-number">.8000</span>_._.httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-keyword">default</span>
[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T04:<span class="hljs-number">53</span>:<span class="hljs-number">33.695</span>Z] <span class="hljs-string">"HEAD /status/500 HTTP/1.1"</span> <span class="hljs-number">500</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">4</span> <span class="hljs-number">2</span> <span class="hljs-string">"172.17.0.2"</span> <span class="hljs-string">"curl/7.54.0"</span> <span class="hljs-string">"6865f4bf-c407-9fa2-a6e2-f14668a9ba63"</span> <span class="hljs-string">"httpbin.example.com"</span> <span class="hljs-string">"127.0.0.1:80"</span> inbound|<span class="hljs-number">8000</span>|http|httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">45104</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.16</span>:<span class="hljs-number">80</span> <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.2</span>:<span class="hljs-number">0</span> outbound_<span class="hljs-number">.8000</span>_._.httpbin.<span class="hljs-keyword">default</span>.svc.cluster.local <span class="hljs-keyword">default</span>
</code></pre>
<p data-nodeid="1091">最后，可以通过命令，清除 httpbin 服务和相关的 Istio Gateway 配置：</p>
<pre class="lang-java" data-nodeid="1092"><code data-language="java">$ kubectl delete gateway httpbin-gateway
$ kubectl delete virtualservice httpbin
$ kubectl delete --ignore-not-found=<span class="hljs-keyword">true</span> -f samples/httpbin/httpbin.yaml
</code></pre>
<p data-nodeid="1093">至此，Istio 的外部访问方式——Gateway 资源类型，就学习完成了，Gateway 类型利用 Envoy 的强大功能，可以实现路由层丰富的配置。这些功能和网格内部提供的功能、配置方式一样，包括<strong data-nodeid="1293">熔断、丰富的负载均衡策略、服务发现、金丝雀发布</strong>等，通过<strong data-nodeid="1294">服务发现</strong>的方式你也可以<strong data-nodeid="1295">解决 Kubernetes Ingress 四层路由的缺陷</strong>。</p>
<p data-nodeid="1094">接下来，我们继续学习 Egress 出口流量控制。</p>
<h3 data-nodeid="1095">Egress 出口流量</h3>
<p data-nodeid="1096">Egress 出口流量是云原生引入的新的设计模式和架构，在传统的 Web 架构中，很少有出口网关的概念。通过 Egress 架构的学习，可以拓展我们在微服务治理中的思维方式，在学习 Egress 过程中，我也会讲到如何通过 Egress 架构赋能传统微服务架构。</p>
<p data-nodeid="1097">在学习 Istio  Egress 以前，我们先来了解下 Kubernetes 中的 Egress 。</p>
<h4 data-nodeid="1098">Kubernetes 中的 Egress</h4>
<p data-nodeid="1099">相较于 Kubernetes 中 Ingress 的强大功能，Kubernetes 中的 Egress 就显得比较弱了，Kubernetes 的 Egress 并没有引入像 Nginx 这样的七层负载均衡器，只是<strong data-nodeid="1306">在 IP 地址或端口层面（OSI 第 3 层或第 4 层）控制网络流量</strong>。</p>
<p data-nodeid="1100">下面我们通过一个简单的配置进行学习下。</p>
<p data-nodeid="1101">通过 Network Policy 的资源，可以配置在三层或者四层网络的 Ingress 或者 Egress 策略。这里的配置设置了一些 IP 和端口的黑白名单：</p>
<pre class="lang-java" data-nodeid="1102"><code data-language="java">apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: <span class="hljs-keyword">default</span>
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: <span class="hljs-number">172.17</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>/<span class="hljs-number">16</span>
        except:
        - <span class="hljs-number">172.17</span><span class="hljs-number">.1</span><span class="hljs-number">.0</span>/<span class="hljs-number">24</span>
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: <span class="hljs-number">6379</span>
  egress:
  - to:
    - ipBlock:
        cidr: <span class="hljs-number">10.0</span><span class="hljs-number">.0</span><span class="hljs-number">.0</span>/<span class="hljs-number">24</span>
    ports:
    - protocol: TCP
      port: <span class="hljs-number">5978</span>
</code></pre>
<h4 data-nodeid="1103">Istio Egress</h4>
<p data-nodeid="1104">Istio Egress 和 Kubernetes 中的 Egress 不同，<strong data-nodeid="1319">Istio的 Egress 本质上是一个 Envoy Proxy</strong>，通过 Envoy 强大的<strong data-nodeid="1320">七层代理功能</strong>，提供丰富的路由策略，而不局限于简单的四层网络 IP 端口黑白名单的配置。</p>
<p data-nodeid="1105">通过下面的架构图，你可以看到每个服务的本地 Proxy 可以通过连接到 Egress  Gateway 访问外部服务，达到精准的安全策略控制。</p>
<p data-nodeid="9027" class=""><img src="https://s0.lgstatic.com/i/image6/M00/04/2D/Cgp9HWAikpmAQQN3AAP80pyzQ1o891.png" alt="Drawing 1.png" data-nodeid="9031"></p>
<div data-nodeid="9028"><p style="text-align:center">Istio Egress 工作原理示意图</p></div>



<p data-nodeid="1108">下面我们通过一个简单的例子，来学习 Istio  Egress。</p>
<p data-nodeid="1109">首先创建一个新的服务 Sleep：</p>
<pre class="lang-java" data-nodeid="1110"><code data-language="java">$ kubectl apply -f samples/sleep/sleep.yaml
</code></pre>
<p data-nodeid="1111">设置 Pod 的环境变量：</p>
<pre class="lang-java" data-nodeid="1112"><code data-language="java">export SOURCE_POD=$(kubectl get pod -l app=sleep -o jsonpath={.items..metadata.name})
</code></pre>
<p data-nodeid="1113">在设置策略之前，我们先尝试从 Pod 内部访问外部服务：</p>
<pre class="lang-java" data-nodeid="1114"><code data-language="java">kubectl exec -it $SOURCE_POD -c sleep -- curl -I https:<span class="hljs-comment">//www.douban.com | grep&nbsp; "HTTP/"; kubectl exec -it $SOURCE_POD -c sleep -- curl -I https://edition.cnn.com | grep "HTTP/"</span>
</code></pre>
<p data-nodeid="1115">可以得到以下结果，表明访问外部正常：</p>
<pre class="lang-java" data-nodeid="1116"><code data-language="java">HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
HTTP/<span class="hljs-number">2</span> <span class="hljs-number">200</span>
</code></pre>
<p data-nodeid="1117">通过下述命令，查看 Istio Egress Gateway 是否部署：</p>
<pre class="lang-java" data-nodeid="1118"><code data-language="java">$ kubectl get pod -l istio=egressgateway -n istio-system
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
istio-egressgateway-<span class="hljs-number">8556f</span>8c8dc-<span class="hljs-number">4</span>tkn7&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;<span class="hljs-number">5</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">95</span>d
</code></pre>
<p data-nodeid="1119">创建一个 ServiceEntry，允许流量直接访问一个外部服务：</p>
<pre class="lang-java" data-nodeid="1120"><code data-language="java">kubectl apply -f - &lt;&lt;EOF
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cnn
spec:
  hosts:
  - edition.cnn.com
  ports:
  - number: <span class="hljs-number">80</span>
    name: http-port
    protocol: HTTP
  - number: <span class="hljs-number">443</span>
    name: https
    protocol: HTTPS
  resolution: DNS
EOF
</code></pre>
<p data-nodeid="1121">为 edition.cnn.com 端口 80 创建  Egress  Gateway，并为指向 Egress Gateway 的流量创建一个 Destination Rule：</p>
<pre class="lang-java" data-nodeid="1122"><code data-language="java">$ kubectl apply -f - &lt;&lt;EOF
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: istio-egressgateway
spec:
  selector:
    istio: egressgateway
  servers:
  - port:
      number: <span class="hljs-number">80</span>
      name: http
      protocol: HTTP
    hosts:
    - edition.cnn.com
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: egressgateway-<span class="hljs-keyword">for</span>-cnn
spec:
  host: istio-egressgateway.istio-system.svc.cluster.local
  subsets:
  - name: cnn
EOF
</code></pre>
<p data-nodeid="1123">定义一个 VirtualService，将流量从 Sidecar 引导至 Egress Gateway，再从 Egress Gateway 引导至外部服务：</p>
<pre class="lang-java" data-nodeid="1124"><code data-language="java">$ kubectl apply -f - &lt;&lt;EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: direct-cnn-through-egress-gateway
spec:
  hosts:
  - edition.cnn.com
  gateways:
  - istio-egressgateway
  - mesh
  http:
  - match:
    - gateways:
      - mesh
      port: <span class="hljs-number">80</span>
    route:
    - destination:
        host: istio-egressgateway.istio-system.svc.cluster.local
        subset: cnn
        port:
          number: <span class="hljs-number">80</span>
      weight: <span class="hljs-number">100</span>
  - match:
    - gateways:
      - istio-egressgateway
      port: <span class="hljs-number">80</span>
    route:
    - destination:
        host: edition.cnn.com
        port:
          number: <span class="hljs-number">80</span>
      weight: <span class="hljs-number">100</span>
EOF
</code></pre>
<p data-nodeid="1125">访问外部第三方服务：</p>
<pre class="lang-java" data-nodeid="1126"><code data-language="java">$ kubectl exec -it $SOURCE_POD -c sleep -- curl -sL -o /dev/<span class="hljs-keyword">null</span> -D - http:<span class="hljs-comment">//edition.cnn.com/politics</span>
</code></pre>
<p data-nodeid="1127">查看 istio-egressgateway 日志：</p>
<pre class="lang-java" data-nodeid="1128"><code data-language="java">kubectl logs -l istio=egressgateway -c istio-proxy -n istio-system | tail
</code></pre>
<p data-nodeid="1129">可以看到如下日志，表明流量经过了 Egress Gateway：</p>
<pre class="lang-java" data-nodeid="1130"><code data-language="java">[<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">08</span>T06:<span class="hljs-number">02</span>:<span class="hljs-number">46.098</span>Z] <span class="hljs-string">"GET /politics HTTP/2"</span> <span class="hljs-number">301</span> - <span class="hljs-string">"-"</span> <span class="hljs-string">"-"</span> <span class="hljs-number">0</span> <span class="hljs-number">0</span> <span class="hljs-number">162</span> <span class="hljs-number">147</span> <span class="hljs-string">"172.18.0.17"</span> <span class="hljs-string">"curl/7.69.1"</span> <span class="hljs-string">"57d13492-b521-961b-bd93-90ab3f0e295e"</span> <span class="hljs-string">"edition.cnn.com"</span> <span class="hljs-string">"151.101.129.67:80"</span> outbound|<span class="hljs-number">80</span>||edition.cnn.com <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.4</span>:<span class="hljs-number">41758</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.4</span>:<span class="hljs-number">8080</span> <span class="hljs-number">172.18</span><span class="hljs-number">.0</span><span class="hljs-number">.17</span>:<span class="hljs-number">32816</span> - -
</code></pre>
<p data-nodeid="1131">清除服务相关配置：</p>
<pre class="lang-java" data-nodeid="1132"><code data-language="java">$ kubectl delete gateway istio-egressgateway
$ kubectl delete serviceentry cnn
$ kubectl delete virtualservice direct-cnn-through-egress-gateway
$ kubectl delete destinationrule egressgateway-<span class="hljs-keyword">for</span>-cnn
</code></pre>
<p data-nodeid="1133">至此，Istio Egress Gateway 就配置完成了，通过这部分的学习，我们可以看到 Istio Egress Gateway 的强大配置，通过 Egress Gateway 我们可以<strong data-nodeid="1344">对外部流量进行权限控制和精准的路由匹配访问</strong>。</p>
<p data-nodeid="1134">在实际工作的项目中，我借鉴了 Istio Egress Gateway 的思想，<strong data-nodeid="1354">将所有外部第三方服务的访问流量都转发到 Egress Gateway</strong>，经由 Egress Gateway 访问出去，通过这样的方式可以大大<strong data-nodeid="1355">降低外部服务访问的延时，维持 HTTP 的 KeepAlive 长连接</strong>。因为如果服务的机器数量过多，访问外部频率又不是很高，与外部服务的连接就很容易断掉，不得不重新建连，而现在大多数外部服务都是 HTTPS 的，需要消耗过多的 SSL 握手时间。</p>
<h3 data-nodeid="1135">总结</h3>
<p data-nodeid="1136">这一讲我主要介绍了 Ingress 和 Egress：入口流量和出口流量控制。通过今天的学习，相信你已经了解了 Kubernetes和 Istio中 Ingress 和 Egress 的基本概念和区别。</p>
<p data-nodeid="1137">本讲内容总结如下：</p>
<p data-nodeid="9720" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/04/2D/Cgp9HWAikquAESGWAAGAzMGoHMY346.png" alt="Drawing 2.png" data-nodeid="9723"></p>

<p data-nodeid="1139">今天我们学习了一种全新的设计模式：Egress，Egress 最基本的功能是做出口流量的权限控制。其实通过 Envoy 的强大七层代理实现的 Egress Gateway 可以实现很多功能，在实际项目中我就通过 Egress Gateway 提升了外部服务访问的性能，那么在你的经验里 Egress Gateway 还有哪些应用场景呢? 欢迎在留言区和我分享你的观点。</p>
<p data-nodeid="1140">今天内容到这里就结束了，下一讲我会讲解如何在 Istio 中完成金丝雀发布：金丝雀部署和版本控制。我们下一讲再见！</p>

---

### 精选评论

##### **杰克：
> Egress应该还能做缓存,共享IP？就是传统网络中的代理服务器的能力

