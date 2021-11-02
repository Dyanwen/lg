<p data-nodeid="1185" class=""><a href="https://prometheus.io/" data-nodeid="1274">Prometheus</a> 和 <a href="https://grafana.com/" data-nodeid="1278">Grafana</a> 可以说是 Kubernetes 监控解决方案中最知名的两个。Prometheus 负责收集、存储、查询数据，而 Grafana 负责将 Prometheus 中的数据进行可视化展示，当然 Grafana 还支持其他平台，比如 <a href="https://grafana.com/docs/grafana/latest/datasources/elasticsearch/" data-nodeid="1282">ElasticSearch</a>、<a href="https://grafana.com/docs/grafana/latest/datasources/influxdb/" data-nodeid="1286">InfluxDB</a>、<a href="https://grafana.com/docs/grafana/latest/datasources/graphite/" data-nodeid="1290">Graphite</a> 等。<a href="https://www.cncf.io/blog/2020/04/24/prometheus-and-grafana-the-perfect-combo/" data-nodeid="1294">CNCF 博客</a>也将这两者称为黄金组合，目前一些公有云提供的托管式 Kubernetes （Managed Kubernetes） 都已经默认安装了 Prometheus 和 Grafana。</p>
<p data-nodeid="1186"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OrgyAPH_iAAFeouU_wAY811.png" alt="Drawing 0.png" data-nodeid="1298"></p>
<p data-nodeid="1187">我们今天就来学习如何在 Kubernetes 集群中搭建 Prometheus 和 Grafana，使之帮我们监控 Kubernetes 集群。</p>
<h3 data-nodeid="1188">通过 Helm 一键安装 Prometheus 和 Grafana</h3>
<p data-nodeid="1189">还记得之前讲过的 Helm 吗？我们今天就通过 Helm 来安装相关的 Charts，一键式搭建 Prometheus 和 Grafana。如果对 Helm 的一些概念和术语还不太清楚，你可以回到第 12 讲复习一下。</p>
<p data-nodeid="1190">首先我们先通过<code data-backticks="1" data-nodeid="1303">helm repo add</code>添加一个 Helm repo，见如下命令：</p>
<pre class="lang-shell" data-nodeid="1191"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm repo add prometheus-community https://prometheus-community.github.io/helm-charts</span>
"prometheus-community" has been added to your repositories
</code></pre>
<p data-nodeid="1192">我们从这个刚刚添加的<code data-backticks="1" data-nodeid="1306">prometheus-community</code>中安装 Prometheus 的 Chart。</p>
<p data-nodeid="1193">然后你通过<code data-backticks="1" data-nodeid="1309">helm repo list</code>就可以看到当前已添加的所有 repo 列表：</p>
<pre class="lang-shell" data-nodeid="1194"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm repo list</span>
NAME                    URL
prometheus-community    https://prometheus-community.github.io/helm-charts
</code></pre>
<p data-nodeid="1195">如果你已经添加了这个 repo，那么可以运行<code data-backticks="1" data-nodeid="1312">helm repo update</code>来更新内容：</p>
<pre class="lang-shell" data-nodeid="1196"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm repo update</span>
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "bitnami" chart repository
Update Complete. ⎈Happy Helming!⎈
</code></pre>
<p data-nodeid="1197">正如我们之前讲 Helm 提到过的，Helm 的这些 repo 非常类似于我们熟悉的 YUM 源、debian 源。</p>
<p data-nodeid="1198">回到正题上，添加了这个 helm repo 了以后，我们就可以开始安装 Prometheus 的 Chart 了。通常来说，使用 Helm 安装 Chart 的最佳实践是单独创建一个 namespace （命名空间），方便区分、隔离和管理。这里我们就创建一个名为<code data-backticks="1" data-nodeid="1316">monitor</code>的 namespace:</p>
<pre class="lang-shell" data-nodeid="1199"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create ns monitor</span>
namespace/monitor created
</code></pre>
<p data-nodeid="1200">创建好了 namespace，我们直接运行如下<code data-backticks="1" data-nodeid="1319">helm install</code>命令进行安装:</p>
<pre class="lang-shell" data-nodeid="1201"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> helm install prometheus-stack prometheus-community/kube-prometheus-stack -n monitor</span>
NAME: prometheus-stack
LAST DEPLOYED: Mon Oct 19 11:07:42 2020
NAMESPACE: monitor
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace monitor get pods -l "release=prometheus-stack"
Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create &amp; configure Alertmanager and Prometheus instances using the Operator.
</code></pre>
<p data-nodeid="1202">下面我来解释下上述命令中的参数含义。</p>
<p data-nodeid="1203">其中<code data-backticks="1" data-nodeid="1323">prometheus-stack</code>是这个 helm release 的名字，当然你在实际使用时候可以换成其他名字。从这个名字的后缀<code data-backticks="1" data-nodeid="1325">stack</code>就可以看出来，<code data-backticks="1" data-nodeid="1327">prometheus-community/kube-prometheus-stack</code>这个 Chart 其实包含了几个组件，类似于我们之前听说过的 <a href="https://www.elastic.co/what-is/elk-stack" data-nodeid="1331">ELKStack</a> 等。除了安装 Prometheus 相关的组件外，该 Chart 的 <a href="https://github.com/prometheus-community/helm-charts/blob/main/charts/kube-prometheus-stack/requirements.yaml" data-nodeid="1335">requirements.yaml</a> 中还定义了三个依赖组件</p>
<ul data-nodeid="1204">
<li data-nodeid="1205">
<p data-nodeid="1206"><a href="https://github.com/helm/charts/tree/master/stable/kube-state-metrics" data-nodeid="1339">stable/kube-state-metrics</a>；</p>
</li>
<li data-nodeid="1207">
<p data-nodeid="1208"><a href="https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-node-exporter" data-nodeid="1343">stable/prometheus-node-exporter</a>；</p>
</li>
<li data-nodeid="1209">
<p data-nodeid="1210"><a href="https://github.com/grafana/helm-charts/tree/main/charts/grafana" data-nodeid="1347">grafana/grafana</a>。</p>
</li>
</ul>
<p data-nodeid="1211">从这个 YAML 文件可以看到这个 Chart 不仅一起安装了 Grafana，还安装了我们之前第 15 讲提到的<code data-backticks="1" data-nodeid="1350">kube-state-metrics</code>和<code data-backticks="1" data-nodeid="1352">prometheus-node-exporter</code>组件。</p>
<p data-nodeid="1212">接着看<code data-backticks="1" data-nodeid="1355">prometheus-community/kube-prometheus-stack</code>，它就是<code data-backticks="1" data-nodeid="1357">prometheus-community</code>repo里面名为<code data-backticks="1" data-nodeid="1359">kube-prometheus-stack</code>的 Chart。</p>
<p data-nodeid="1213">最后看<code data-backticks="1" data-nodeid="1362">monitor</code>，它是我们刚创建的新命名空间，在部署的时候使用。</p>
<p data-nodeid="1214">当然如果你不想安装这些依赖，也可以通过<a href="https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#multiple-releases" data-nodeid="1367">文档中的提及方法</a>更改（override）这些默认设置。</p>
<p data-nodeid="1215">现在我们运行刚才<code data-backticks="1" data-nodeid="1370">helm install</code>输出结果中的命令，来查看我们相关 Pod 的状态：</p>
<pre class="lang-shell" data-nodeid="1216"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl --namespace monitor get pods -l <span class="hljs-string">"release=prometheus-stack"</span></span>
NAME                                                   READY   STATUS    RESTARTS   AGE
prometheus-stack-kube-prom-operator-6998d5c5b7-kgv8b   2/2     Running   0          2m41s
prometheus-stack-prometheus-node-exporter-l7pr9        1/1     Running   0          2m41s
</code></pre>
<p data-nodeid="1217">这里有一个名为prometheus-stack-kube-prom-operator-6998d5c5b7-kgv8b的 Pod，它是 <a href="https://github.com/prometheus-operator/prometheus-operator" data-nodeid="1375">prometheus-operator</a> 的一个实例，主要用于为我们创建、管理和维护 Prometheus 集群，其具体架构图如下。</p>
<p data-nodeid="1218"><img src="https://s0.lgstatic.com/i/image/M00/61/09/Ciqc1F-OrkWAXJsQAAEHWw6G6rM837.png" alt="Drawing 1.png" data-nodeid="1379"></p>
<p data-nodeid="1219">（<a href="https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/images/architecture.png%EF%BC%89" data-nodeid="1383">https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/user-guides/images/architecture.png）</a></p>
<p data-nodeid="1220">Operator 的工作流程相对复杂一些，我会在后面单独行介绍，在此先略过。有兴趣的话可以查看<a href="https://coreos.com/operators/prometheus/docs/latest/user-guides/getting-started.html" data-nodeid="1387">官方文档</a>简单了解下。</p>
<p data-nodeid="1221">另外一个叫prometheus-stack-prometheus-node-exporter-l7pr9的 Pod 是一个 <a href="https://github.com/prometheus/node_exporter" data-nodeid="1392">node-exporter</a>，主要来采集 kubelet 所在节点上的 metrics。</p>
<p data-nodeid="1222">这两个 Pod 都运行成功，我们再来看看<code data-backticks="1" data-nodeid="1395">monitor</code>这个 namespace 下面其他依赖组件的状态：</p>
<pre class="lang-shell" data-nodeid="1223"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get all -n monitor</span>
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/alertmanager-prometheus-stack-kube-prom-alertmanager-0   2/2     Running   0          11m
pod/prometheus-prometheus-stack-kube-prom-prometheus-0       3/3     Running   1          11m
pod/prometheus-stack-grafana-5b6dd6b5fb-rtp6z                2/2     Running   0          11m
pod/prometheus-stack-kube-prom-operator-6998d5c5b7-kgv8b     2/2     Running   0          11m
pod/prometheus-stack-kube-state-metrics-c7c69c8c9-bhgjv      1/1     Running   0          11m
pod/prometheus-stack-prometheus-node-exporter-l7pr9          1/1     Running   0          11m
NAME                                                TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
service/alertmanager-operated                       ClusterIP   None             &lt;none&gt;        9093/TCP,9094/TCP,9094/UDP   11m
service/prometheus-operated                         ClusterIP   None             &lt;none&gt;        9090/TCP                     11m
service/prometheus-stack-grafana                    ClusterIP   10.108.122.155   &lt;none&gt;        80/TCP                       11m
service/prometheus-stack-kube-prom-alertmanager     ClusterIP   10.103.37.81     &lt;none&gt;        9093/TCP                     11m
service/prometheus-stack-kube-prom-operator         ClusterIP   10.97.75.165     &lt;none&gt;        8080/TCP,443/TCP             11m
service/prometheus-stack-kube-prom-prometheus       ClusterIP   10.102.82.76     &lt;none&gt;        9090/TCP                     11m
service/prometheus-stack-kube-state-metrics         ClusterIP   10.109.78.8      &lt;none&gt;        8080/TCP                     11m
service/prometheus-stack-prometheus-node-exporter   ClusterIP   10.101.221.185   &lt;none&gt;        9100/TCP                     11m
NAME                                                       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/prometheus-stack-prometheus-node-exporter   1         1         1       1            1           &lt;none&gt;          11m
NAME                                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/prometheus-stack-grafana              1/1     1            1           11m
deployment.apps/prometheus-stack-kube-prom-operator   1/1     1            1           11m
deployment.apps/prometheus-stack-kube-state-metrics   1/1     1            1           11m
NAME                                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/prometheus-stack-grafana-5b6dd6b5fb              1         1         1       11m
replicaset.apps/prometheus-stack-kube-prom-operator-6998d5c5b7   1         1         1       11m
replicaset.apps/prometheus-stack-kube-state-metrics-c7c69c8c9    1         1         1       11m
NAME                                                                    READY   AGE
statefulset.apps/alertmanager-prometheus-stack-kube-prom-alertmanager   1/1     11m
statefulset.apps/prometheus-prometheus-stack-kube-prom-prometheus       1/1
</code></pre>
<p data-nodeid="1224">可以看到全部 Pod 都已经部署成功且运行。</p>
<p data-nodeid="1225">prometheus-operator 以 Deployment 的形式部署，并帮助我们创建了名为 prometheus-prometheus-stack-kube-prom-prometheus 的 StatefulSet，副本数设置为 1。</p>
<p data-nodeid="1226">接下来就可以本地来访问 Prometheus 了，通过如下命令，我们在本地通过 <a href="http://127.0.0.1:9090" data-nodeid="1402">http://127.0.0.1:9090</a> 来访问 Prometheus：</p>
<pre class="lang-shell" data-nodeid="1227"><code data-language="shell">kubectl port-forward -n monitor prometheus-prometheus-stack-kube-prom-prometheus-0 9090
</code></pre>
<p data-nodeid="1228"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OrlyAf9PiAAW6ru2bHZI707.png" alt="Drawing 2.png" data-nodeid="1406"></p>
<p data-nodeid="1229">同样地，我们也可以使用如下命令，在本地通过 <a href="http://127.0.0.1:3000" data-nodeid="1410">http://127.0.0.1:3000</a> 来访问 Grafana：</p>
<pre class="lang-shell" data-nodeid="1230"><code data-language="shell">kubectl port-forward -n monitor prometheus-stack-grafana-5b6dd6b5fb-rtp6z 3000
</code></pre>
<p data-nodeid="1231"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OrmeAXd8PABEbqH6I_pE362.png" alt="Drawing 3.png" data-nodeid="1414"></p>
<p data-nodeid="1232">这里默认用户名是 admin，密码是 prom-operator。当然你也可以通过 Grafana 的配置来拿到，我们来看看如何操作。</p>
<p data-nodeid="1233">下面是刚才 Chart 部署的 grafana Deployment 的配置：</p>
<pre class="lang-dart" data-nodeid="1234"><code data-language="dart">kubectl <span class="hljs-keyword">get</span> deploy -n monitor prometheus-stack-grafana -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus-stack-grafana
  namespace: monitor
  ...
spec:
  ...
  template:
    ...
    spec:
      containers:
      ...
      - env:
        - name: GF_SECURITY_ADMIN_USER
          valueFrom:
            secretKeyRef:
              key: admin-user
              name: prometheus-stack-grafana
        - name: GF_SECURITY_ADMIN_PASSWORD
          valueFrom:
            secretKeyRef:
              key: admin-password
              name: prometheus-stack-grafana
        image: grafana/grafana:<span class="hljs-number">7.2</span><span class="hljs-number">.0</span>
        ...
      ...
status:
  ...
</code></pre>
<p data-nodeid="1235">可以看到环境变量<code data-backticks="1" data-nodeid="1418">GF_SECURITY_ADMIN_USER</code>和<code data-backticks="1" data-nodeid="1420">GF_SECURITY_ADMIN_PASSWORD</code>就是 grafana 的登录用户名和密码，具体的值来自 Secret<code data-backticks="1" data-nodeid="1422">prometheus-stack-grafana</code>。</p>
<p data-nodeid="1236">我们接着看这个 Secret：</p>
<pre class="lang-shell" data-nodeid="1237"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get secret prometheus-stack-grafana -n monitor -o jsonpath=<span class="hljs-string">'{.data}'</span></span>
map[admin-password:cHJvbS1vcGVyYXRvcg== admin-user:YWRtaW4= ldap-toml:]
</code></pre>
<p data-nodeid="1238">分别通过 base64 对其进行解码就可以得到用户名和密码：</p>
<pre class="lang-shell" data-nodeid="1239"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get secret prometheus-stack-grafana -n monitor -o jsonpath=<span class="hljs-string">'{.data.admin-user}'</span> | base64 --decode</span>
admin
<span class="hljs-meta">$</span><span class="bash"> kubectl get secret prometheus-stack-grafana -n monitor -o jsonpath=<span class="hljs-string">'{.data.admin-password}'</span> | base64 --decode</span>
prom-operator
</code></pre>
<p data-nodeid="1240">使用上述用户名和密码登录，进来后就可以看到 grafana 的主页面了。</p>
<p data-nodeid="1241"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OroiAZBPUABgqqawSub0949.png" alt="Drawing 4.png" data-nodeid="1429"></p>
<p data-nodeid="2075" class="">按照上述图示点进来，你就可以看到已经配置好的各个 Dashboard：</p>


<p data-nodeid="1243"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OrpKAG4VoAAs8370oK_k082.png" alt="Drawing 5.png" data-nodeid="1433"></p>
<p data-nodeid="1244">这些都是<code data-backticks="1" data-nodeid="1435">prometheus-community/kube-prometheus-stack</code>这个 Chart 预先配置好的，基本上包括我们对 Kubernetes 的各项监控大盘，你可以随意点击几个 Dashboard 进行了解。</p>
<p data-nodeid="1245"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-OrtSASkD9ABXvtUrLlxI610.png" alt="Drawing 6.png" data-nodeid="1439"></p>
<p data-nodeid="1246">你可以参考<a href="https://grafana.com/docs/grafana/latest/dashboards/" data-nodeid="1443">官方文档</a>学习 Dashboard 的创建和数据展示能力。</p>
<h3 data-nodeid="1247">生产环境中一些重要的关注指标</h3>
<h4 data-nodeid="1248">集群状态、性能以及各个 API 对象</h4>
<p data-nodeid="1249">kube-state-metrics 可以帮助我们汇聚 Kubernetes 集群中的各大信息，比如 Pod 数量，APIServer 访问请求数，Pod 调度性能，等等。你可以通过如下命令可本地访问<a href="http://127.0.0.1:8080/metrics" data-nodeid="1450">http://127.0.0.1:8080/metrics</a>拿到相关的 metrics。</p>
<pre class="lang-shell" data-nodeid="1250"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl port-forward -n monitor prometheus-stack-kube-state-metrics-c7c69c8c9-bhgjv 8080</span>
</code></pre>
<p data-nodeid="1251"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-Oru-AevJ5AB_ZvYnlO60668.png" alt="Drawing 7.png" data-nodeid="1454"></p>
<h4 data-nodeid="1252">各节点的监控指标</h4>
<p data-nodeid="1253">各个节点承载着 Pod 的运行，因此对各个节点的监控至关重要，比如节点的 CPU 使用率、内存使用率、平均工作负载等。你可以通过上面 Chart 预配置好的 Node dashboard 来查看：</p>
<p data-nodeid="1254"><img src="https://s0.lgstatic.com/i/image/M00/61/09/Ciqc1F-OrvaARKZ2ABNH_bHh8sg135.png" alt="Drawing 8.png" data-nodeid="1459"></p>
<p data-nodeid="1255">你可以通过切换节点来查看不同节点的状态，或者单独创建一个 Dashboard，添加一些指标，诸如：</p>
<ul data-nodeid="1256">
<li data-nodeid="1257">
<p data-nodeid="1258"><code data-backticks="1" data-nodeid="1461">kube_node_status_capacity_cpu_cores</code>代表节点 CPU 容量；</p>
</li>
<li data-nodeid="1259">
<p data-nodeid="1260"><code data-backticks="1" data-nodeid="1463">kube_node_status_capacity_memory_bytes</code>代表节点的 Memory容量；</p>
</li>
<li data-nodeid="1261">
<p data-nodeid="1262"><code data-backticks="1" data-nodeid="1465">kubelet_running_container_count</code>代表节点上运行的容器数量；</p>
</li>
<li data-nodeid="1263">
<p data-nodeid="1264">……</p>
</li>
</ul>
<h3 data-nodeid="1265">设置 Alert</h3>
<p data-nodeid="1266">除了监控指标的收集和展示以外，我们还需要设置报警（Alert）。你可以通过设置合适的 Alert rules，在触发的时候通过钉钉、邮件、Slack、PagerDuty 等多种方式通知你。</p>
<p data-nodeid="1267"><img src="https://s0.lgstatic.com/i/image/M00/61/14/CgqCHl-Orv6ACQ9XAAqhwgoMegs538.png" alt="Drawing 9.png" data-nodeid="1472"></p>
<p data-nodeid="1268">在此不做太多介绍，可以通过<a href="https://grafana.com/docs/grafana/latest/alerting/create-alerts/" data-nodeid="1476">官方文档</a>来设置你的 Alert。</p>
<h3 data-nodeid="1269">写在最后</h3>
<p data-nodeid="1270">在这一小节，我们介绍了快速搭建基于 Prometheus + Grafana 的 Kubernetes 监控体系。当然安装的方式有很多，今天这里只写了最方便、最快捷的方式。Chart 里预先配置了多个 Dashboard，方便你开箱即用。</p>
<p data-nodeid="1271" class="">如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### *星：
> 老师，请问下，内网中如何使用helm安装prome，有指导链接吗？

