<p data-nodeid="837" class="">通过前面的学习，相信你已经见识到了 Kubernetes 的强大能力，它能帮你轻松管理大规模的容器服务，尤其是面对复杂的环境时，比如节点异常、容器异常退出等，Kubernetes 内部的 Service、Deployment 会动态地进行调整，比如增加新的副本、关联新的 Pod 等。</p>
<p data-nodeid="838">当然 Kubernetes 的这种自动伸缩能力可不止于此。</p>
<p data-nodeid="839" class="">我们今天就来看看如果利用这些伸缩能力，帮我们在应对大促这样的大流量活动时，控制业务的资源水位，以提供稳定的服务，避免容器的负载过高被打爆，出现流量下跌、业务抖动等情况，从而引发业务故障。</p>
<h3 data-nodeid="840">Pod 水平自动伸缩（Horizontal Pod Autoscaler，HPA）</h3>
<p data-nodeid="841">一般来说，我们在遇到这种大流量的场景时，映入我们脑海中的一个想法就是水平扩展，即增加一些实例来分担流量压力。像 Deployment 这种支持多副本的工作负载，我们就可以通过调整<code data-backticks="1" data-nodeid="898">spec.replicas</code>来增加或减少副本数，从而改变整体的业务水位满足我们的需求，即整体负载高时就增加一些实例，负载低就适当减少一些实例来节省资源。</p>
<p data-nodeid="842">当然人为不断地调整<code data-backticks="1" data-nodeid="901">spec.replicas</code>的数值显然是不太现实的，<a href="https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/" data-nodeid="905">HPA</a>可以根据应用的 CPU 利用率等水位信息，动态地增加或者减少 Pod 副本数量，帮你自动化地完成这一调和过程。</p>
<p data-nodeid="1886" class="">HPA 大多数是用来自动扩缩（Scale）一些无状态的应用负载，比如 Deployment，或者你自己定义的其他类型的无状态工作负载。当然你也可以用来扩缩有状态的应用负载，比如 StatefulSet。</p>



<p data-nodeid="844">我们来看下面这张图，它描述了 HPA 通过动态地调整<code data-backticks="1" data-nodeid="909">Deployment</code>的副本数，从而控制 Pod 的数量。</p>
<p data-nodeid="845"><img src="https://s0.lgstatic.com/i/image/M00/5E/E4/CgqCHl-H4QiAJarCAAQ5OqhzE0c569.png" alt="Drawing 0.png" data-nodeid="913"></p>
<p data-nodeid="846">（<a href="https://github.com/kubernetes/website/blob/master/static/images/docs/horizontal-pod-autoscaler.png" data-nodeid="917">https://github.com/kubernetes/website/blob/master/static/images/docs/horizontal-pod-autoscaler.png</a>）</p>
<p data-nodeid="847">在使用 HPA 的时候，你需要提前部署好 <a href="https://github.com/kubernetes-sigs/metrics-server" data-nodeid="922">metrics-server</a>，可以通过 <code data-backticks="1" data-nodeid="924">kubectl apply -f</code>一键部署完成（如果你想了解更多关于 metrics-server 的部署，可以参考<a href="https://github.com/kubernetes-sigs/metrics-server#deployment" data-nodeid="928">官方文档</a>）。</p>
<pre class="lang-java" data-nodeid="848"><code data-language="java">$ kubectl apply -f https:<span class="hljs-comment">//github.com/kubernetes-sigs/metrics-server/releases/download/v0.3.7/components.yaml</span>
clusterrole.rbac.authorization.k8s.io/system:aggregated-metrics-reader created
clusterrolebinding.rbac.authorization.k8s.io/metrics-server:system:auth-delegator created
rolebinding.rbac.authorization.k8s.io/metrics-server-auth-reader created
apiservice.apiregistration.k8s.io/v1beta1.metrics.k8s.io created
serviceaccount/metrics-server created
deployment.apps/metrics-server created
service/metrics-server created
clusterrole.rbac.authorization.k8s.io/system:metrics-server created
clusterrolebinding.rbac.authorization.k8s.io/system:metrics-server created
</code></pre>
<p data-nodeid="849">我们来查看下<code data-backticks="1" data-nodeid="931">metrics-server</code>对应的 Deployment 的状态，如下图它已经处于 Ready 状态。</p>
<pre class="lang-java" data-nodeid="850"><code data-language="java">kubectl get deploy -n kube-system metrics-server
NAME &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; READY &nbsp; UP-TO-DATE &nbsp; AVAILABLE &nbsp; AGE
metrics-server &nbsp; <span class="hljs-number">1</span>/<span class="hljs-number">1</span> &nbsp; &nbsp; <span class="hljs-number">1</span> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">1</span> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">26</span>s
</code></pre>
<p data-nodeid="851">你可以通过<code data-backticks="1" data-nodeid="934">kubectl logs</code>来查看对应 Pod 的日志，来确定其是否正常工作。如果有异常，可以参考<a href="https://github.com/kubernetes-sigs/metrics-server#configuration" data-nodeid="938">这里</a>调整部署参数。</p>
<p data-nodeid="852">现在我们来看看如何来使用 HPA。</p>
<p data-nodeid="853">首先我们部署一个 Deployment，其 YAML 配置如下（你可以保存为<code data-backticks="1" data-nodeid="942">nginx-deploy-hpa.yaml</code>文件）：</p>
<pre class="lang-js" data-nodeid="854"><code data-language="js">apiVersion: apps/v1
kind: Deployment
metadata:
name: nginx-deployment
namespace: demo
spec:
selector:
matchLabels:
app: nginx
usage: hpa
replicas: 1
template:
metadata:
labels:
app: nginx
usage: hpa
spec:
containers:
- name: nginx
image: nginx:1.19.2
ports:
- containerPort: 80
resources:
requests: # 这里我们把quota设置得小一点，方便做压力测试
memory: "64Mi"
cpu: "250m"
limits:
memory: "128Mi"
cpu: "500m"
</code></pre>
<p data-nodeid="855">下面我们通过 kubectl 来创建这个 Deployment：</p>
<pre class="lang-shell" data-nodeid="856"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create -f nginx-deploy-hpa.yaml</span>
deployment.apps/nginx-deployment created
</code></pre>
<p data-nodeid="857">现在我们来查看一下该 Deployment 的状态，通过下面几条命令可以看到它已经 ready ：</p>
<pre class="lang-shell" data-nodeid="858"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get deploy -n demo</span>
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           4s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo</span>
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6d4b885966-zngnd   1/1     Running   0          8s
</code></pre>
<p data-nodeid="859">可以看到，我们创建的 Deployment 运行成功。<br>
现在我们来创建一个 Service ，来关联这个 Deployment，我们后续就可以通过这个 Service 来访问 Deployment 的各个 Pod 实例了。通过如下命令，我们就可以快速创建一个对应的 Service 出来，当然你也可以通过 YAML 文件来创建。</p>
<pre class="lang-shell" data-nodeid="860"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl expose deployment/nginx-deployment -n demo</span>
service/nginx-deployment exposed
</code></pre>
<p data-nodeid="861">我们用<code data-backticks="1" data-nodeid="950">kubectl get</code>来查看一下这个 Service 和对应的 Endpoints 对象：</p>
<pre class="lang-shell" data-nodeid="862"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get svc -n demo</span>
NAME               TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
nginx-deployment   ClusterIP   10.109.93.199   &lt;none&gt;        80/TCP    4s
<span class="hljs-meta">$</span><span class="bash"> kubectl get endpoints -n demo</span>
NAME               ENDPOINTS                     AGE
nginx-deployment   10.1.0.163:80,10.1.0.164:80   12s
</code></pre>
<p data-nodeid="863">后面我们就可以通过访问这个 Service 的<code data-backticks="1" data-nodeid="953">10.109.93.199</code>地址来访问该服务。</p>
<p data-nodeid="864">现在我们来创建一个 HPA 对象。通过kubectl autoscale的命令，我们就可以将 Deployment 的副本数控制在 1~10 之间，CPU 利用率保持在 50% 以下，即当该 Deployment 所关联的 Pod 的平均 CPU 利用率超过 50% 时，就增加副本数，直到小于该阈值。当平均 CPU 利用率低于 50% 时，就减少副本数：</p>
<pre class="lang-shell" data-nodeid="865"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl autoscale deploy nginx-deployment -n demo --cpu-percent=50 --min=1 --max=10</span>
horizontalpodautoscaler.autoscaling/nginx-deployment autoscaled
</code></pre>
<p data-nodeid="866">我们来查看一下刚才创建出来的 HPA 对象：</p>
<pre class="lang-shell" data-nodeid="867"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get hpa -n demo</span>
NAME               REFERENCE                     TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   0%/50%          1         10        0          6s
</code></pre>
<p data-nodeid="868">在 kube-controller-manager 中有对应的 HPAController 负责算出合适的副本数，它会根据 metrics-server 上报的对应 Pod metrics 进行计算，具体的<a href="https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details" data-nodeid="962">算法细节</a>可以通过官方文档来了解。</p>
<p data-nodeid="869">好了，到现在，一切准备就绪，我们开始见证 HPA 的魔力！</p>
<p data-nodeid="870">我们现在新开两个终端，分别运行命令<code data-backticks="1" data-nodeid="966">kubectl get deploy -n demo -w</code>和<code data-backticks="1" data-nodeid="968">kubectl get hpa -n demo -w</code>来观察其状态变化。</p>
<p data-nodeid="2726" class="">现在创建一个 Pod 来增加上面 Nginx 服务的访问压力。这里我们用压测工具ApacheBench 进行压力测试看看：</p>


<pre class="lang-dart" data-nodeid="872"><code data-language="dart">$ kubectl run demo-benchmark --image httpd:<span class="hljs-number">2.4</span><span class="hljs-number">.46</span>-alpine -n demo -it sh
/usr/local/apache2 # ab -n <span class="hljs-number">50000</span> -c <span class="hljs-number">500</span> -s <span class="hljs-number">60</span> http:<span class="hljs-comment">//10.109.93.199/</span>
This <span class="hljs-keyword">is</span> ApacheBench, Version <span class="hljs-number">2.3</span> &lt;$Revision: <span class="hljs-number">1879490</span> $&gt;
Copyright <span class="hljs-number">1996</span> Adam Twiss, Zeus Technology Ltd, http:<span class="hljs-comment">//www.zeustech.net/</span>
Licensed to The Apache Software Foundation, http:<span class="hljs-comment">//www.apache.org/</span>
Benchmarking <span class="hljs-number">10.109</span><span class="hljs-number">.93</span><span class="hljs-number">.199</span> (be patient)
Completed <span class="hljs-number">5000</span> requests
Completed <span class="hljs-number">10000</span> requests
Completed <span class="hljs-number">15000</span> requests
Completed <span class="hljs-number">20000</span> requests
Completed <span class="hljs-number">25000</span> requests
Completed <span class="hljs-number">30000</span> requests
Completed <span class="hljs-number">35000</span> requests
Completed <span class="hljs-number">40000</span> requests
Completed <span class="hljs-number">45000</span> requests
Completed <span class="hljs-number">50000</span> requests
Finished <span class="hljs-number">50000</span> requests

Server Software:        nginx/<span class="hljs-number">1.19</span><span class="hljs-number">.2</span>
Server Hostname:        <span class="hljs-number">10.109</span><span class="hljs-number">.93</span><span class="hljs-number">.199</span>
Server Port:            <span class="hljs-number">80</span>
Document Path:          /
Document Length:        <span class="hljs-number">612</span> bytes
Concurrency Level:      <span class="hljs-number">500</span>
Time taken <span class="hljs-keyword">for</span> tests:   <span class="hljs-number">74.783</span> seconds
Complete requests:      <span class="hljs-number">50000</span>
Failed requests:        <span class="hljs-number">2</span>
   (Connect: <span class="hljs-number">0</span>, Receive: <span class="hljs-number">0</span>, Length: <span class="hljs-number">1</span>, Exceptions: <span class="hljs-number">1</span>)
Total transferred:      <span class="hljs-number">42250000</span> bytes
HTML transferred:       <span class="hljs-number">30600000</span> bytes
Requests per second:    <span class="hljs-number">668.60</span> [#/sec] (mean)
Time per request:       <span class="hljs-number">747.830</span> [ms] (mean)
Time per request:       <span class="hljs-number">1.496</span> [ms] (mean, across all concurrent requests)
Transfer rate:          <span class="hljs-number">551.73</span> [Kbytes/sec] received
Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        <span class="hljs-number">0</span>   <span class="hljs-number">95</span> <span class="hljs-number">347.7</span>     <span class="hljs-number">10</span>    <span class="hljs-number">3134</span>
Processing:     <span class="hljs-number">9</span>   <span class="hljs-number">49</span> <span class="hljs-number">295.1</span>     <span class="hljs-number">33</span>   <span class="hljs-number">60048</span>
Waiting:        <span class="hljs-number">5</span>   <span class="hljs-number">44</span> <span class="hljs-number">122.8</span>     <span class="hljs-number">30</span>    <span class="hljs-number">3393</span>
Total:         <span class="hljs-number">19</span>  <span class="hljs-number">144</span> <span class="hljs-number">477.2</span>     <span class="hljs-number">50</span>   <span class="hljs-number">60048</span>
Percentage of the requests served within a certain time (ms)
  <span class="hljs-number">50</span>%     <span class="hljs-number">50</span>
  <span class="hljs-number">66</span>%     <span class="hljs-number">61</span>
  <span class="hljs-number">75</span>%     <span class="hljs-number">65</span>
  <span class="hljs-number">80</span>%     <span class="hljs-number">69</span>
  <span class="hljs-number">90</span>%     <span class="hljs-number">83</span>
  <span class="hljs-number">95</span>%   <span class="hljs-number">1071</span>
  <span class="hljs-number">98</span>%   <span class="hljs-number">1143</span>
  <span class="hljs-number">99</span>%   <span class="hljs-number">1946</span>
 <span class="hljs-number">100</span>%  <span class="hljs-number">60048</span> (longest request)
</code></pre>
<p data-nodeid="873">在上述 ab 命令运行的同时，我们可以切回之前打开的两个终端窗口看看输出信息：</p>
<pre class="lang-dart" data-nodeid="874"><code data-language="dart">$ kubectl <span class="hljs-keyword">get</span> hpa -n demo -w
NAME               REFERENCE                     TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">0</span>%/<span class="hljs-number">50</span>%    <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">1</span>          <span class="hljs-number">48</span>m
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">125</span>%/<span class="hljs-number">50</span>%   <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">1</span>         <span class="hljs-number">50</span>m
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">125</span>%/<span class="hljs-number">50</span>%   <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">3</span>         <span class="hljs-number">50</span>m
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">0</span>%/<span class="hljs-number">50</span>%     <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">3</span>         <span class="hljs-number">51</span>m
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">0</span>%/<span class="hljs-number">50</span>%     <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">3</span>         <span class="hljs-number">56</span>m
nginx-deployment   Deployment/nginx-deployment   <span class="hljs-number">0</span>%/<span class="hljs-number">50</span>%     <span class="hljs-number">1</span>         <span class="hljs-number">10</span>        <span class="hljs-number">1</span>         <span class="hljs-number">56</span>m
$ kubectl <span class="hljs-keyword">get</span> deploy -n demo -w
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">1</span>     <span class="hljs-number">1</span>            <span class="hljs-number">1</span>           <span class="hljs-number">48</span>m
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">3</span>     <span class="hljs-number">1</span>            <span class="hljs-number">1</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">3</span>     <span class="hljs-number">1</span>            <span class="hljs-number">1</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">3</span>     <span class="hljs-number">1</span>            <span class="hljs-number">1</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">3</span>     <span class="hljs-number">3</span>            <span class="hljs-number">1</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">2</span>/<span class="hljs-number">3</span>     <span class="hljs-number">3</span>            <span class="hljs-number">2</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">3</span>/<span class="hljs-number">3</span>     <span class="hljs-number">3</span>            <span class="hljs-number">3</span>           <span class="hljs-number">50</span>m
nginx-deployment   <span class="hljs-number">3</span>/<span class="hljs-number">1</span>     <span class="hljs-number">3</span>            <span class="hljs-number">3</span>           <span class="hljs-number">56</span>m
nginx-deployment   <span class="hljs-number">3</span>/<span class="hljs-number">1</span>     <span class="hljs-number">3</span>            <span class="hljs-number">3</span>           <span class="hljs-number">56</span>m
nginx-deployment   <span class="hljs-number">1</span>/<span class="hljs-number">1</span>     <span class="hljs-number">1</span>            <span class="hljs-number">1</span>           <span class="hljs-number">56</span>m
</code></pre>
<p data-nodeid="875">可以看到，随着访问压力的增加，Pod 的平均利用率也直线上升，一度达到了 125%，超过我们的阈值 50%。这个时候，Deployment 的副本数被调整到了 3，随之 2 个新 Pod 被拉起，负载很快降到了 50% 以下。而后随着压测结束，HPA 又将 Deployment 调整为了 1，维持在低水位。<br>
我们现在回过头来看看 metrics-server 创建的对象PodMetrics：</p>
<pre class="lang-shell" data-nodeid="876"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get podmetrics -n demo</span>
NAME                                AGE
nginx-deployment-6d4b885966-zngnd   0s
nginx-deployment-6d4b885966-lgwd9   0s
nginx-deployment-6d4b885966-hhk7v   0s
<span class="hljs-meta">$</span><span class="bash"> kubectl get podmetrics -n demo nginx-deployment-6d4b885966-zngnd -o yaml</span>
apiVersion: metrics.k8s.io/v1beta1
containers:
- name: nginx
  usage:
    cpu: "0"
    memory: 5524Ki
kind: PodMetrics
metadata:
  creationTimestamp: "2020-10-13T09:38:15Z"
  name: nginx-deployment-6d4b885966-zngnd
  namespace: demo
  selfLink: /apis/metrics.k8s.io/v1beta1/namespaces/demo/pods/nginx-deployment-6d4b885966-zngnd
timestamp: "2020-10-13T09:37:55Z"
</code></pre>
<p data-nodeid="877">HPAController 就是通过这些 PodMetrics 来计算平均的 CPU 使用率，从而确定 spec.replicas 的新数值。<br>
除了 CPU 以外，HPA 还支持其他自定义度量指标，有兴趣可以参考<a href="https://kubernetes.io/zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/#%E5%9F%BA%E4%BA%8E%E5%A4%9A%E9%A1%B9%E5%BA%A6%E9%87%8F%E6%8C%87%E6%A0%87%E5%92%8C%E8%87%AA%E5%AE%9A%E4%B9%89%E5%BA%A6%E9%87%8F%E6%8C%87%E6%A0%87%E8%87%AA%E5%8A%A8%E6%89%A9%E7%BC%A9" data-nodeid="980">官方文档</a>。</p>
<p data-nodeid="878">HPA 能够自适应地伸缩 Pod 的数目，但是如果集群中资源不够了怎么办？比如节点紧张无法支撑新的 Pod 运行？</p>
<p data-nodeid="879">这个时候我们就可以添加新的节点资源到集群中，那么有没有类似 HPA 的做法可以自动化地扩容集群的节点资源？</p>
<p data-nodeid="880">当然是有的！我们来看看 Cluster Autoscaler。</p>
<h3 data-nodeid="881">Cluster Autoscaler</h3>
<p data-nodeid="882"><a href="https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler" data-nodeid="988">Cluster Autoscaler</a>（下文统称 CA）目前对接了<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/alicloud/README.md" data-nodeid="992">阿里云</a>、<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/aws/README.md" data-nodeid="996">AWS</a>、<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/azure/README.md" data-nodeid="1000">Azure</a>、<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/baiducloud/README.md" data-nodeid="1004">百度云</a>、<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/huaweicloud/README.md" data-nodeid="1008">华为云</a>、<a href="https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/cloudprovider/magnum/README.md" data-nodeid="1012">Openstack</a>等云厂商，你可以参照各个厂商的部署要求，进行部署。在部署的时候，请注意 CA 和 Kubernetes 版本要对应，<a href="https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#releases" data-nodeid="1016">最好两者版本一样</a>。</p>
<p data-nodeid="883">这里描述了 CA 和 HPA 一起使用的情形。两者一般可以配合起来一起使用。</p>
<p data-nodeid="884"><img src="https://s0.lgstatic.com/i/image/M00/5E/DA/Ciqc1F-H5LCAEEBiAAB2SWxVmmg879.png" alt="Drawing 1.png" data-nodeid="1021"></p>
<p data-nodeid="885">（<a href="https://blogs.tensult.com/2019/08/20/cluster-autoscalerca-and-horizontal-pod-autoscalerhpa-on-kubernetes/" data-nodeid="1025">https://blogs.tensult.com/2019/08/20/cluster-autoscalerca-and-horizontal-pod-autoscalerhpa-on-kubernetes/</a>）</p>
<p data-nodeid="886">我们来看看 CA 是如何工作的。CA 主要用来监听（watch）集群中未被调度的 Pod （即 Pod 暂时由于某些调度策略、抑或资源不满足，导致无法被成功调度），然后确定是否可以通过增加节点资源来解决无法调度的问题。</p>
<p data-nodeid="4406" class="">如果可以的话，就会调用对应的 cloud provider 接口，向集群中增加新的节点。当然 CA 在创建新的节点资源前，也会尝试是否可以将正在运行的一部分 Pod “挤压”到某些节点上，从而让这些未被调度的 Pod 可以被调度，如果可行的话，CA 会将这些 Pod 进行驱逐。这些被驱逐的 Pod 会被重新调度（reschedule）到其他的节点上。我们会在后续调度章节来深入讨论有关 Pod 驱逐的话题。</p>




<p data-nodeid="888">目前 CA 仅支持<a href="https://github.com/kubernetes/autoscaler/tree/master/cluster-autoscaler#deployment" data-nodeid="1032">部分云厂商</a>，如果不满足你的需求，你可以考虑基于<a href="https://github.com/kubernetes/autoscaler" data-nodeid="1036">社区的代码</a>进行二次开发，链接中有详细说明我就不赘述了。</p>
<h3 data-nodeid="889">写在最后</h3>
<p data-nodeid="890" class="">除了 HPA 和 CA 以外，还有 <a href="https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler" data-nodeid="1042">Vertical Pod Autoscaler (VPA)</a>可以帮我们确定 Pod 中合适的 CPU 和 Memory 区间，有兴趣可以了解下。限于篇幅，本节不多做介绍。在实际使用的时候，注意千万不要同时使用 HPA 和 VPA，以免造成异常。</p>
<p data-nodeid="891">使用 HPA 的时候，也尽量对 Deployment 这类对象进行操作，避免对 ReplicaSet 操作。毕竟 ReplicaSet 由 Deployment 管理着，一旦 Deployment 更新了，旧的ReplicaSet 会被新的 ReplicaSet 替换掉。</p>
<p data-nodeid="892" class="">这节课到这里就结束了，如果你有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### *伦：
> 我用dashboard可以吗

