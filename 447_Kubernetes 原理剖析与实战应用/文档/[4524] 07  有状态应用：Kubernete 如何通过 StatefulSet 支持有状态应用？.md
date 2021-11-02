<p data-nodeid="841" class="">在上一节课中，我们学习了 Kubernetes 中的无状态工作负载，并上手实践了 Deployment 对象，相信现在你已经慢慢喜欢上 Kubernetes 了。</p>
<p data-nodeid="842">那么本节课，我们来一起看看Kubernetes 中的另外一种工作负载 StatefulSet。从名字就可以看出，这个工作负载主要用于有状态的服务发布。关于有状态服务和无状态服务，你可以参考上一节课的内容。</p>
<p data-nodeid="843">这节课，我们从一个具体的例子来逐渐了解、认识 StatefulSet。在 kubectl 命令行中，我们一般将 StatefulSet 简写为 sts。在部署一个 StatefulSet 的时候，有个前置依赖对象，即 Service（服务）。这个对象在 StatefulSet 中的作用，我们在下文中会一一道来。另外，关于这个对象的详细介绍和其他作用，我们会在后面的课程中单独讲解。在此，你可以先暂时略过对 Service 的感知。我们先看如下一个 Service：</p>
<pre class="lang-shell" data-nodeid="844"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat nginx-svc.yaml</span>
apiVersion: v1
kind: Service
metadata:
  name: nginx-demo
  namespace: demo
  labels:
    app: nginx
spec:
  clusterIP: None
  ports:
  - port: 80
    name: web
  selector:
    app: nginx
</code></pre>
<p data-nodeid="845">上面这段 yaml 的意思是，在 demo 这个命名空间中，创建一个名为 nginx-demo 的服务，这个服务暴露了 80 端口，可以访问带有<code data-backticks="1" data-nodeid="916">app=nginx</code>这个 label 的 Pod。</p>
<p data-nodeid="846">我们现在利用上面这段 yaml 在集群中创建出一个 Service：</p>
<pre class="lang-shell" data-nodeid="847"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create ns demo</span>
<span class="hljs-meta">$</span><span class="bash"> kubectl create -f nginx-svc.yaml</span>
service/nginx-demo created
<span class="hljs-meta">$</span><span class="bash"> kubectl get svc -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;TYPE&nbsp; &nbsp; &nbsp; &nbsp; CLUSTER-IP&nbsp; &nbsp; &nbsp; &nbsp;EXTERNAL-IP&nbsp; &nbsp;PORT(S)&nbsp; &nbsp;AGE
nginx-demo&nbsp; &nbsp;ClusterIP&nbsp; &nbsp;None        &nbsp; &nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; 80/TCP&nbsp; &nbsp; 5s
</code></pre>
<p data-nodeid="848">创建好了这个前置依赖的 Service，下面我们就可以开始创建真正的 StatefulSet 对象，可参照如下的 yaml 文件：</p>
<pre class="lang-shell" data-nodeid="849"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat web-sts.yaml</span>
apiVersion: apps/v1
kind: StatefulSet
metadata:
&nbsp; name: web-demo
&nbsp; namespace: demo
spec:
&nbsp; serviceName: "nginx-demo"
&nbsp; replicas: 2
&nbsp; selector:
&nbsp; &nbsp; matchLabels:
&nbsp; &nbsp; &nbsp; app: nginx
&nbsp; template:
&nbsp; &nbsp; metadata:
&nbsp; &nbsp; &nbsp; labels:
&nbsp; &nbsp; &nbsp; &nbsp; app: nginx
&nbsp; &nbsp; spec:
&nbsp; &nbsp; &nbsp; containers:
&nbsp; &nbsp; &nbsp; - name: nginx
&nbsp; &nbsp; &nbsp; &nbsp; image: nginx:1.19.2-alpine
&nbsp; &nbsp; &nbsp; &nbsp; ports:
&nbsp; &nbsp; &nbsp; &nbsp; - containerPort: 80
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; name: web
<span class="hljs-meta">$</span><span class="bash"> kubectl create -f web-sts.yaml</span>
<span class="hljs-meta">$</span><span class="bash"> kubectl get sts -n demo</span>
NAME       READY   AGE
web-demo   0/2     9s
</code></pre>
<p data-nodeid="850">可以看到，到这里我已经将名为web-demo的StatefulSet部署完成了。</p>
<p data-nodeid="851">下面我们来一点点探索 StatefulSet 的秘密，看看它有哪些特性，为何可以保障服务有状态的运行。</p>
<h3 data-nodeid="852">StatefulSet 的特性</h3>
<p data-nodeid="853">通过 kubectl 的<strong data-nodeid="930">watch</strong>功能（命令行加参数<code data-backticks="1" data-nodeid="928">-w</code>），我们可以观察到 Pod 状态的一步步变化。</p>
<pre class="lang-shell" data-nodeid="854"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -w</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
web-demo-0&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 18s
web-demo-0&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 20s
web-demo-1&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-1&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-1&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-1&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 2s
</code></pre>
<p data-nodeid="855"><strong data-nodeid="937">通过 StatefulSet 创建出来的 Pod 名字有一定的规律</strong>，即<code data-backticks="1" data-nodeid="935">$(statefulset名称)-$(序号)</code>，比如这个例子中的web-demo-0、web-demo-1。</p>
<p data-nodeid="856">这里面还有个有意思的点，web-demo-0 这个 Pod 比 web-demo-1 优先创建，而且在 web-demo-0 变为 Running 状态以后，才被创建出来。为了证实这个猜想，我们在一个终端窗口观察 StatefulSet 的 Pod：</p>
<pre class="lang-shell" data-nodeid="857"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -w -l app=nginx</span>
</code></pre>
<p data-nodeid="858">我们再开一个终端端口来 watch 这个 namespace 中的 event：</p>
<pre class="lang-shell" data-nodeid="859"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get event -n demo -w</span>
</code></pre>
<p data-nodeid="860">现在我们试着改变这个 StatefulSet 的副本数，将它改成 5：</p>
<pre class="lang-shell" data-nodeid="861"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl scale sts web-demo -n demo --replicas=5</span>
statefulset.apps/web-demo scaled
</code></pre>
<p data-nodeid="862">此时我们观察到另外两个终端端口的输出：</p>
<pre class="lang-shell" data-nodeid="863"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -w</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
web-demo-0&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 20m
web-demo-1&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 20m
web-demo-2&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-2&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-2&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-2&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 2s
web-demo-3&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-3&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-3&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-3&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3s
web-demo-4&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-4&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Pending&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-4&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 0s
web-demo-4&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3s
</code></pre>
<p data-nodeid="864">我们再一次看到了 StatefulSet 管理的 Pod 按照 2、3、4 的顺序依次创建，名称有规律，跟上一节通过 Deployment 创建的随机 Pod 名有很大的区别。</p>
<p data-nodeid="865">通过观察对应的 event 信息，也可以再次证实我们的猜想。</p>
<pre class="lang-js" data-nodeid="866"><code data-language="js">$ kubectl get event -n demo -w
LAST SEEN&nbsp; &nbsp;TYPE&nbsp; &nbsp; &nbsp;REASON&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;OBJECT&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;MESSAGE
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Scheduled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully assigned demo/web-demo<span class="hljs-number">-0</span> to kraken
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Pulling&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Pulling image <span class="hljs-string">"nginx:1.19.2-alpine"</span>
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Pulled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pod/web-demo<span class="hljs-number">-0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully pulled image <span class="hljs-string">"nginx:1.19.2-alpine"</span>
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Created&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Created container nginx
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Started&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Started container nginx
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Scheduled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully assigned demo/web-demo<span class="hljs-number">-1</span> to kraken
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Pulled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pod/web-demo<span class="hljs-number">-1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Container image <span class="hljs-string">"nginx:1.19.2-alpine"</span> already present on machine
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Created&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Created container nginx
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;Started&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-1</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Started container nginx
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;SuccessfulCreate&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;create Pod web-demo<span class="hljs-number">-0</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">20</span>m&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Normal&nbsp; &nbsp;SuccessfulCreate&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;create Pod web-demo<span class="hljs-number">-1</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulCreate&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;create Pod web-demo<span class="hljs-number">-2</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Scheduled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully assigned demo/web-demo<span class="hljs-number">-2</span> to kraken
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Pulled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pod/web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Container image <span class="hljs-string">"nginx:1.19.2-alpine"</span> already present on machine
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Created&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Created container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Started&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Started container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulCreate&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;create Pod web-demo<span class="hljs-number">-3</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Scheduled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully assigned demo/web-demo<span class="hljs-number">-3</span> to kraken
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Pulled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pod/web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Container image <span class="hljs-string">"nginx:1.19.2-alpine"</span> already present on machine
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Created&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Created container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Started&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Started container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulCreate&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;create Pod web-demo<span class="hljs-number">-4</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Scheduled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Successfully assigned demo/web-demo<span class="hljs-number">-4</span> to kraken
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Pulled&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;pod/web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Container image <span class="hljs-string">"nginx:1.19.2-alpine"</span> already present on machine
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Created&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Created container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Started&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Started container nginx
</code></pre>
<p data-nodeid="867">现在我们试着进行一次缩容：</p>
<pre class="lang-shell" data-nodeid="1621"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl scale sts web-demo -n demo --replicas=2</span>
statefulset.apps/web-demo scaled
</code></pre>
<p data-nodeid="1622">此时观察另外两个终端窗口，分别如下：</p>
<pre class="lang-js" data-nodeid="1623"><code data-language="js">web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">11</span>m
web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">11</span>m
web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">11</span>m
web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">11</span>m
web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp;<span class="hljs-number">1</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp;<span class="hljs-number">0</span>/<span class="hljs-number">1</span>&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">12</span>m
</code></pre>
<pre class="lang-js" data-nodeid="1624"><code data-language="js"><span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulDelete&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;<span class="hljs-keyword">delete</span> Pod web-demo<span class="hljs-number">-4</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Killing&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-4</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Stopping container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Killing&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-3</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Stopping container nginx
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulDelete&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;<span class="hljs-keyword">delete</span> Pod web-demo<span class="hljs-number">-3</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;SuccessfulDelete&nbsp; &nbsp;statefulset/web-demo&nbsp; &nbsp;<span class="hljs-keyword">delete</span> Pod web-demo<span class="hljs-number">-2</span> <span class="hljs-keyword">in</span> StatefulSet web-demo successful
<span class="hljs-number">0</span>s&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Normal&nbsp; &nbsp;Killing&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pod/web-demo<span class="hljs-number">-2</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Stopping container nginx
</code></pre>
<p data-nodeid="1625">可以看到，在缩容的时候，StatefulSet 关联的 Pod 按着 4、3、2 的顺序依次删除。</p>
<p data-nodeid="1626">可见，<strong data-nodeid="1680">对于一个拥有 N 个副本的 StatefulSet 来说</strong>，<strong data-nodeid="1681">Pod 在部署时按照 {0 …… N-1} 的序号顺序创建的</strong>，<strong data-nodeid="1682">而删除的时候按照逆序逐个删除</strong>，这便是我想说的第一个特性。</p>
<p data-nodeid="1627">接着我们来看，<strong data-nodeid="1688">StatefulSet 创建出来的 Pod 都具有固定的、且确切的主机名</strong>，比如：</p>
<pre class="lang-shell" data-nodeid="1628"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> <span class="hljs-keyword">for</span> i <span class="hljs-keyword">in</span> 0 1; <span class="hljs-keyword">do</span> kubectl <span class="hljs-built_in">exec</span> web-demo-<span class="hljs-variable">$i</span> -n demo -- sh -c <span class="hljs-string">'hostname'</span>; <span class="hljs-keyword">done</span></span>
web-demo-0
web-demo-1
</code></pre>
<p data-nodeid="1629">我们再看看上面 StatefulSet 的 API 对象定义，有没有发现跟我们上一节中 Deployment 的定义极其相似，主要的差异在于<code data-backticks="1" data-nodeid="1690">spec.serviceName</code>这个字段。它很重要，StatefulSet 根据这个字段，为每个 Pod 创建一个 DNS 域名，这个<strong data-nodeid="1698">域名的格式</strong>为<code data-backticks="1" data-nodeid="1696">$(podname).(headless service name)</code>，下面我们通过例子来看一下。</p>
<p data-nodeid="1630">当前 Pod 和 IP 之间的对应关系如下：</p>
<pre class="lang-shell" data-nodeid="1631"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx -o wide</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE&nbsp; &nbsp; &nbsp;IP&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NODE&nbsp; &nbsp; &nbsp;NOMINATED NODE&nbsp; &nbsp;READINESS GATES
web-demo-0&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3h17m&nbsp; &nbsp;10.244.0.39&nbsp; &nbsp;kraken&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
web-demo-1&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 3h17m&nbsp; &nbsp;10.244.0.40&nbsp; &nbsp;kraken&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
</code></pre>
<p data-nodeid="1632">Podweb-demo-0 的IP 地址是 10.244.0.39，web-demo-1的 IP 地址是 10.244.0.40。这里我们通过<code data-backticks="1" data-nodeid="1701">kubectl run</code>在同一个命名空间<code data-backticks="1" data-nodeid="1703">demo</code>中创建一个名为 dns-test 的 Pod，同时 attach 到容器中，类似于<code data-backticks="1" data-nodeid="1705">docker run -it --rm</code>这个命令。<br>
我么在容器中运行 nslookup 来查询它们在集群内部的 DNS 地址，如下所示：</p>
<pre class="lang-shell" data-nodeid="1633"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl run -it --rm --image busybox:1.28 dns-test -n demo</span>
If you don't see a command prompt, try pressing enter.
/ # nslookup web-demo-0.nginx-demo
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:&nbsp; &nbsp; &nbsp; web-demo-0.nginx-demo
Address 1: 10.244.0.39 web-demo-0.nginx-demo.demo.svc.cluster.local
/ # nslookup web-demo-1.nginx-demo
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
Name:&nbsp; &nbsp; &nbsp; web-demo-1.nginx-demo
Address 1: 10.244.0.40 web-demo-1.nginx-demo.demo.svc.cluster.local
</code></pre>
<p data-nodeid="1634">可以看到，每个 Pod 都有一个对应的 <a href="https://baike.baidu.com/item/A%E8%AE%B0%E5%BD%95/1188077?fr=aladdin" data-nodeid="1712">A 记录</a>。<br>
我们现在删除一下这些 Pod，看看会有什么变化：</p>
<pre class="lang-shell" data-nodeid="1635"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl delete pod -l app=nginx -n demo</span>
pod "web-demo-0" deleted
pod "web-demo-1" deleted
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -l app=nginx -n demo -o wide</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE&nbsp; &nbsp;IP&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NODE&nbsp; &nbsp; &nbsp;NOMINATED NODE&nbsp; &nbsp;READINESS GATES
web-demo-0&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 15s&nbsp; &nbsp;10.244.0.50&nbsp; &nbsp;kraken&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
web-demo-1&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 13s&nbsp; &nbsp;10.244.0.51&nbsp; &nbsp;kraken&nbsp; &nbsp;&lt;none&gt;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
</code></pre>
<p data-nodeid="1636">删除成功后，可以发现 StatefulSet 立即生成了新的 Pod，但是 Pod 名称维持不变。唯一变化的就是 IP 发生了改变。</p>
<p data-nodeid="1637">我们再来看看 DNS 记录：</p>
<pre class="lang-shell" data-nodeid="1638"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl run -it --rm --image busybox:1.28 dns-test -n demo</span>
If you don't see a command prompt, try pressing enter.
/ # nslookup web-demo-0.nginx-demo
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:&nbsp; &nbsp; &nbsp; web-demo-0.nginx-demo
Address 1: 10.244.0.50 web-demo-0.nginx-demo.demo.svc.cluster.local
/ # nslookup web-demo-1.nginx-demo
Server:&nbsp; &nbsp; 10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:&nbsp; &nbsp; &nbsp; web-demo-1.nginx-demo
Address 1: 10.244.0.51 web-demo-1.nginx-demo.demo.svc.cluster.local
</code></pre>
<p data-nodeid="1639">可以看出，<strong data-nodeid="1735">DNS 记录中 Pod 的域名没有发生变化</strong>，<strong data-nodeid="1736">仅仅 IP 地址发生了更换</strong>。因此当 Pod 所在的节点发生故障导致 Pod 飘移到其他节点上，或者 Pod 因故障被删除重建，Pod 的 IP 都会发生变化，但是 Pod 的域名不会有任何变化，这也就意味着<strong data-nodeid="1737">服务间可以通过不变的 Pod 域名来保障通信稳定</strong>，<strong data-nodeid="1738">而不必依赖 Pod IP</strong>。</p>
<p data-nodeid="1640">有了<code data-backticks="1" data-nodeid="1740">spec.serviceName</code>这个字段，保证了 StatefulSet 关联的 Pod 可以有稳定的网络身份标识，即 Pod 的序号、主机名、DNS 记录名称等。</p>
<p data-nodeid="1641">最后一个我想说的是，对于有状态的服务来说，每个副本都会用到持久化存储，且各自使用的数据是不一样的。</p>
<p data-nodeid="1642">StatefulSet 通过 PersistentVolumeClaim（PVC）可以保证 Pod 的存储卷之间一一对应的绑定关系。同时，删除 StatefulSet 关联的 Pod 时，不会删除其关联的 PVC。</p>
<p data-nodeid="1643">我们会在后续网络存储的章节中来专门介绍，再次先略过。</p>
<h3 data-nodeid="1644">如何更新升级 StatefulSet</h3>
<p data-nodeid="1645">那么，如果想对一个 StatefulSet 进行升级，该怎么办呢？</p>
<p data-nodeid="1646">在 StatefulSet 中，支持两种更新升级策略，即 RollingUpdate 和 OnDelete。</p>
<p data-nodeid="1647">RollingUpdate策略是<strong data-nodeid="1755">默认的更新策略</strong>。可以实现 Pod 的滚动升级，跟我们上一节课中 Deployment 介绍的RollingUpdate策略一样。比如我们这个时候做了镜像更新操作，那么整个的升级过程大致如下，先逆序删除所有的 Pod，然后依次用新镜像创建新的 Pod 出来。这里你可以通过<code data-backticks="1" data-nodeid="1753">kubectl get pod -n demo -w -l app=nginx</code>来动手观察下。</p>
<p data-nodeid="1648">同时使用 RollingUpdate 更新策略还支持通过 partition 参数来分段更新一个 StatefulSet。所有序号大于或者等于 partition 的Pod 都将被更新。你这里也可以手动更新 StatefulSet 的配置来实验下。</p>
<p data-nodeid="1649">当你把更新策略设置为 OnDelete 时，我们就必须手动先删除 Pod，才能触发新的 Pod 更新。</p>
<h3 data-nodeid="1650">写在最后</h3>
<p data-nodeid="1651">现在我们就总结下 StatefulSet 的特点：</p>
<ul data-nodeid="1652">
<li data-nodeid="1653">
<p data-nodeid="1654">具备固定的网络标记，比如主机名，域名等；</p>
</li>
<li data-nodeid="1655">
<p data-nodeid="1656">支持持久化存储，而且最好能够跟实例一一绑定；</p>
</li>
<li data-nodeid="1657">
<p data-nodeid="1658">可以按照顺序来部署和扩展；</p>
</li>
<li data-nodeid="1659">
<p data-nodeid="1660">可以按照顺序进行终止和删除操作；</p>
</li>
<li data-nodeid="1661">
<p data-nodeid="1662">在进行滚动升级的时候，也会按照一定顺序。</p>
</li>
</ul>
<p data-nodeid="1663">借助 StatefulSet 的这些能力，我们就可以去部署一些有状态服务，比如 MySQL、ZooKeeper、MongoDB 等。你可以跟着这个<a href="https://kubernetes.io/zh/docs/tutorials/stateful-application/zookeeper/" data-nodeid="1768">教程</a>在 Kubernetes 中搭建一个 ZooKeeper 集群。</p>
<p data-nodeid="1664">到这里这节课就结束了，下节课我们就来学习配置管理。如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **强：
> kubectl run -it --rm --image busybox:1.28.3 dns-test -n demo #busybox 版本一定要用">1.28.3

