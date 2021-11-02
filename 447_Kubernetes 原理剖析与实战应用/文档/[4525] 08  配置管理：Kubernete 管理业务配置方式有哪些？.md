<p data-nodeid="120123">通过前面几节课的学习，我们已经对 Kubernetes 中的 Pod 以及一些业务负载有所了解。你可以根据课程中提供的示例，自己动手尝试在集群中实践起来。</p>



<p data-nodeid="119457">在使用过程中，我们常常需要对 Pod 进行一些配置管理，比如参数配置文件怎么使用，敏感数据怎么保存传递，等等。有些人可能会觉得，为什么不把这些配置（不限于参数、配置文件、密钥等）打包到镜像中去啊？乍一听，好像有点可行，但是这种做法“硬伤”太多。</p>
<ul data-nodeid="119458">
<li data-nodeid="119459">
<p data-nodeid="119460">有些不变的配置是可以打包到镜像中的，那可变的配置呢？</p>
</li>
<li data-nodeid="119461">
<p data-nodeid="119462">信息泄漏，很容易引发安全风险，尤其是一些敏感信息，比如密码、密钥等。</p>
</li>
<li data-nodeid="119463">
<p data-nodeid="119464">每次配置更新后，都要重新打包一次，升级应用。镜像版本过多，也给镜像管理和镜像中心存储带来很大的负担。</p>
</li>
<li data-nodeid="119465">
<p data-nodeid="119466">定制化太严重，可扩展能力差，且不容易复用。</p>
</li>
</ul>
<p data-nodeid="119467">所以这里的一个最佳实践就是将配置信息和容器镜像进行解耦，以“不变应万变”。在 Kubernetes 中，一般有 ConfigMap 和 Secret 两种对象，可以用来做配置管理。</p>
<h3 data-nodeid="120563" class="">ConfigMap</h3>

<p data-nodeid="121439">首先我们来讲一下 ConfigMap 这个对象，它主要用来保存一些非敏感数据，可以用作环境变量、命令行参数或者挂载到存储卷中。</p>
<p data-nodeid="121440" class=""><img src="https://s0.lgstatic.com/i/image/M00/4F/8C/Ciqc1F9gkFGAAFxaAAB80T5b-WU361.png" alt="image (2).png" data-nodeid="121448"><br>
（<a href="https://matthewpalmer.net/kubernetes-app-developer/articles/configmap-diagram.gif" data-nodeid="121453">https://matthewpalmer.net/kubernetes-app-developer/articles/configmap-diagram.gif</a>）</p>




<p data-nodeid="119472">ConfigMap 通过键值对来存储信息，是个 namespace 级别的资源。在 kubectl 使用时，我们可以简写成 cm。</p>
<p data-nodeid="119473">我们来看一下两个 ConfigMap 的 API 定义：</p>
<pre class="lang-shell" data-nodeid="119474"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat cm-demo-mix.yaml</span>
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-demo-mix # 对象名字
  namespace: demo # 所在的命名空间
data: # 这是跟其他对象不太一样的地方，其他对象这里都是spec
<span class="hljs-meta">  #</span><span class="bash"> 每一个键都映射到一个简单的值</span>
  player_initial_lives: "3" # 注意这里的值如果数字的话，必须用字符串来表示
  ui_properties_file_name: "user-interface.properties"
<span class="hljs-meta">  #</span><span class="bash"> 也可以来保存多行的文本</span>
  game.properties: |
    enemy.types=aliens,monsters
    player.maximum-lives=5
  user-interface.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
<span class="hljs-meta">$</span><span class="bash"> cat cm-demo-all-env.yaml</span>
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-demo-all-env
  namespace: demo
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
</code></pre>
<p data-nodeid="121901">可见，我们通过 ConfigMap 既可以存储简单的键值对，也能存储多行的文本。</p>
<p data-nodeid="121902">现在我们来创建这两个 ConfigMap：</p>

<pre class="lang-shell" data-nodeid="119476"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create -f cm-demo-mix.yaml</span>
configmap/cm-demo-mix created
<span class="hljs-meta">$</span><span class="bash"> kubectl create -f cm-demo-all-env.yaml</span>
configmap/cm-demo-all-env created
</code></pre>
<p data-nodeid="122351">创建 ConfigMap，你也可以通过<code data-backticks="1" data-nodeid="122354">kubectl create cm</code>基于<a href="https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-directories" data-nodeid="122358">目录</a>、<a href="https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-files" data-nodeid="122362">文件</a>或者<a href="https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/#create-configmaps-from-literal-values" data-nodeid="122366">字面值</a>来创建，详细可参考这个<a href="https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-pod-configmap/#%E4%BD%BF%E7%94%A8-kubectl-create-configmap-%E5%88%9B%E5%BB%BA-configmap" data-nodeid="122370">官方文档</a>。</p>
<p data-nodeid="122352">创建成功后，我们可以通过如下方式来查看创建出来的对象。</p>

<pre class="lang-shell" data-nodeid="119478"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get cm -n demo</span>
NAME              DATA   AGE
cm-demo-all-env   2      30s
cm-demo-mix       4      2s
<span class="hljs-meta">$</span><span class="bash"> kubectl describe cm cm-demo-all-env -n demo</span>
Name:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cm-demo-all-env
Namespace:&nbsp; &nbsp; demo
Labels:&nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
Annotations:&nbsp; &lt;none&gt;

Data
====
SPECIAL_LEVEL:
----
very
SPECIAL_TYPE:
----
charm
Events:&nbsp; &lt;none&gt;
<span class="hljs-meta">$</span><span class="bash"> kubectl describe cm cm-demo-mix -n demo</span>
Name:&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;cm-demo-mix
Namespace:&nbsp; &nbsp; demo
Labels:&nbsp; &nbsp; &nbsp; &nbsp;&lt;none&gt;
Annotations:&nbsp; &lt;none&gt;

Data
====
user-interface.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true

game.properties:
----
enemy.types=aliens,monsters
player.maximum-lives=5

player_initial_lives:
----
3
ui_properties_file_name:
----
user-interface.properties
Events:&nbsp; &lt;none&gt;
</code></pre>
<p data-nodeid="119479">下面我们看看怎么和 Pod 结合起来使用。在使用的时候，有几个地方需要特别注意：</p>
<ul data-nodeid="119480">
<li data-nodeid="119481">
<p data-nodeid="119482"><strong data-nodeid="119589">Pod 必须和 ConfigMap 在同一个 namespace 下面；</strong></p>
</li>
<li data-nodeid="119483">
<p data-nodeid="119484"><strong data-nodeid="119593">在创建 Pod 之前，请务必保证 ConfigMap 已经存在，否则 Pod 创建会报错。</strong></p>
</li>
</ul>
<pre class="lang-shell" data-nodeid="119485"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat cm-demo-pod.yaml</span>
apiVersion: v1
kind: Pod
metadata:
&nbsp; name: cm-demo-pod
&nbsp; namespace: demo
spec:
&nbsp; containers:
&nbsp; &nbsp; - name: demo
&nbsp; &nbsp; &nbsp; image: busybox:1.28
&nbsp; &nbsp; &nbsp; command:
&nbsp; &nbsp; &nbsp; &nbsp; - "bin/sh"
&nbsp; &nbsp; &nbsp; &nbsp; - "-c"
&nbsp; &nbsp; &nbsp; &nbsp; - "echo PLAYER_INITIAL_LIVES=$PLAYER_INITIAL_LIVES &amp;&amp; sleep 10000"
&nbsp; &nbsp; &nbsp; env:
&nbsp; &nbsp; &nbsp; &nbsp; # 定义环境变量
&nbsp; &nbsp; &nbsp; &nbsp; - name: PLAYER_INITIAL_LIVES # 请注意这里和 ConfigMap 中的键名是不一样的
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; valueFrom:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configMapKeyRef:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; name: cm-demo-mix&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;# 这个值来自 ConfigMap
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; key: player_initial_lives # 需要取值的键
&nbsp; &nbsp; &nbsp; &nbsp; - name: UI_PROPERTIES_FILE_NAME
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; valueFrom:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configMapKeyRef:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; name: cm-demo-mix
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; key: ui_properties_file_name
&nbsp; &nbsp; &nbsp; envFrom:&nbsp; # 可以将 configmap 中的所有键值对都通过环境变量注入容器中
&nbsp; &nbsp; &nbsp; &nbsp; - configMapRef:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; name: cm-demo-all-env
&nbsp; &nbsp; &nbsp; volumeMounts:
&nbsp; &nbsp; &nbsp; - name: full-config # 这里是下面定义的 volume 名字
&nbsp; &nbsp; &nbsp; &nbsp; mountPath: "/config" # 挂载的目标路径
&nbsp; &nbsp; &nbsp; &nbsp; readOnly: true
&nbsp; &nbsp; &nbsp; - name: part-config
&nbsp; &nbsp; &nbsp; &nbsp; mountPath: /etc/game/
&nbsp; &nbsp; &nbsp; &nbsp; readOnly: true
&nbsp; volumes: # 您可以在 Pod 级别设置卷，然后将其挂载到 Pod 内的容器中
&nbsp; &nbsp; - name: full-config # 这是 volume 的名字
&nbsp; &nbsp; &nbsp; configMap:
&nbsp; &nbsp; &nbsp; &nbsp; name: cm-demo-mix # 提供你想要挂载的 ConfigMap 的名字
&nbsp; &nbsp; - name: part-config
&nbsp; &nbsp; &nbsp; configMap:
&nbsp; &nbsp; &nbsp; &nbsp; name: cm-demo-mix
&nbsp; &nbsp; &nbsp; &nbsp; items: # 我们也可以只挂载部分的配置
&nbsp; &nbsp; &nbsp; &nbsp; - key: game.properties
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; path: properties
</code></pre>
<p data-nodeid="119486">在上面的这个例子中，几乎囊括了 ConfigMap 的几大使用场景：</p>
<ul data-nodeid="119487">
<li data-nodeid="119488">
<p data-nodeid="119489">命令行参数；</p>
</li>
<li data-nodeid="119490">
<p data-nodeid="119491">环境变量，可以只注入部分变量，也可以全部注入；</p>
</li>
<li data-nodeid="119492">
<p data-nodeid="119493">挂载文件，可以是单个文件，也可以是所有键值对，用每个键值作为文件名。</p>
</li>
</ul>
<p data-nodeid="119494">我们接着来创建：</p>
<pre class="lang-shell" data-nodeid="119495"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create -f cm-demo-pod.yaml</span>
pod/cm-demo-pod created
</code></pre>
<p data-nodeid="119496">创建成功后，我们 exec 到容器中看看：</p>
<pre class="lang-shell" data-nodeid="119497"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl <span class="hljs-built_in">exec</span> -it cm-demo-pod -n demo sh</span>
kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.
/ # env
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
UI_PROPERTIES_FILE_NAME=user-interface.properties
HOSTNAME=cm-demo-pod
SHLVL=1
HOME=/root
SPECIAL_LEVEL=very
TERM=xterm
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
PLAYER_INITIAL_LIVES=3
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
SPECIAL_TYPE=charm
/ # ls /config/
game.properties&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ui_properties_file_name
player_initial_lives&nbsp; &nbsp; &nbsp; &nbsp;user-interface.properties
/ # ls -alh /config/
total 12
drwxrwxrwx&nbsp; &nbsp; 3 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; 4.0K Aug 27 09:54 .
drwxr-xr-x&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; 4.0K Aug 27 09:54 ..
drwxr-xr-x&nbsp; &nbsp; 2 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; 4.0K Aug 27 09:54 ..2020_08_27_09_54_31.007551221
lrwxrwxrwx&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 31 Aug 27 09:54 ..data -&gt; ..2020_08_27_09_54_31.007551221
lrwxrwxrwx&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 22 Aug 27 09:54 game.properties -&gt; ..data/game.properties
lrwxrwxrwx&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 27 Aug 27 09:54 player_initial_lives -&gt; ..data/player_initial_lives
lrwxrwxrwx&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 30 Aug 27 09:54 ui_properties_file_name -&gt; ..data/ui_properties_file_name
lrwxrwxrwx&nbsp; &nbsp; 1 root&nbsp; &nbsp; &nbsp;root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 32 Aug 27 09:54 user-interface.properties -&gt; ..data/user-interface.properties
/ # cat /config/game.properties
enemy.types=aliens,monsters
player.maximum-lives=5
/ # cat /etc/game/properties
enemy.types=aliens,monsters
player.maximum-lives=5
</code></pre>
<p data-nodeid="119498">可以看到，环境变量都已经正确注入，对应的文件和目录也都挂载进来了。<br>
在上面<code data-backticks="1" data-nodeid="119603">ls -alh /config/</code>后，我们看到挂载的文件中存在软链接，都指向了<code data-backticks="1" data-nodeid="119605">..data</code>目录下的文件。这样做的好处，是 kubelet 会定期同步检查已经挂载的 ConfigMap 是否是最新的，如果更新了，就是创建一个新的文件夹存放最新的内容，并同步修改<code data-backticks="1" data-nodeid="119607">..data</code>指向的软链接。</p>
<p data-nodeid="119499">一般我们只把一些非敏感的数据保存到 ConfigMap 中，敏感的数据就要保存到 Secret 中了。</p>
<h3 data-nodeid="122819" class="">Secret</h3>

<p data-nodeid="119501">我们可以用 Secret 来保存一些敏感的数据信息，比如密码、密钥、token 等。在使用的时候， 跟 ConfigMap 的用法基本保持一致，都可以用来作为环境变量或者文件挂载。</p>
<p data-nodeid="119502">Kubernetes 自身也有一些内置的 Secret，主要用来保存访问 APIServer 的 service account token，我们放到后面权限部分一起讲解，在此先略过。</p>
<p data-nodeid="119503">除此之外，还可以用来保存私有镜像中心的身份信息，这样 kubelet 可以拉取到镜像。</p>
<blockquote data-nodeid="119504">
<p data-nodeid="119505">注： 如果你使用的是 Docker，也可以提前在目标机器上运行<code data-backticks="1" data-nodeid="119615">docker login yourprivateregistry.com</code>来保存你的有效登录信息。Docker 一般会将私有仓库的密钥保存在<code data-backticks="1" data-nodeid="119617">$HOME/.docker/config.json</code>文件中，将该文件分发到所有节点即可。</p>
</blockquote>
<p data-nodeid="119506">我们看看如何通过 kubectl 来创建 secret，通过命令行 help 可以看到 kubectl 能够创建多种类型的 Secret。</p>
<pre class="lang-shell" data-nodeid="119507"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create secret  -h</span>
 Create a secret using specified subcommand.
Available Commands:
   docker-registry Create a secret for use with a Docker registry
   generic         Create a secret from a local file, directory or literal value
   tls             Create a TLS secret
Usage:
   kubectl create secret [flags] [options]
Use "kubectl  --help" for more information about a given command.
 Use "kubectl options" for a list of global command-line options (applies to all commands).
</code></pre>
<p data-nodeid="119508">我们先来创建一个 Secret 来保存访问私有容器仓库的身份信息：</p>
<pre class="lang-shell" data-nodeid="119509"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create secret -n demo docker-registry regcred \</span>
   --docker-server=yourprivateregistry.com \
   --docker-username=allen \
   --docker-password=mypassw0rd \
--docker-email=allen@example.com
 secret/regcred created
<span class="hljs-meta"> $</span><span class="bash"> kubectl get secret -n demo regcred</span>
 NAME      TYPE                             DATA   AGE
 regcred   kubernetes.io/dockerconfigjson   1      28s
</code></pre>
<p data-nodeid="119510">这里我们可以看到，创建出来的 Secret 类型是<code data-backticks="1" data-nodeid="119622">kubernetes.io/dockerconfigjson</code>：</p>
<pre class="lang-shell" data-nodeid="119511"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl describe secret -n demo regcred</span>
Name:         regcred
Namespace:    demo
Labels:       &lt;none&gt;
Annotations:  &lt;none&gt;
Type:  kubernetes.io/dockerconfigjson
Data
====
.dockerconfigjson:  144 bytes
</code></pre>
<p data-nodeid="119512">为了防止 Secret 中的内容被泄漏，<code data-backticks="1" data-nodeid="119625">kubectl get</code>和<code data-backticks="1" data-nodeid="119627">kubectl describe</code>会避免直接显示密码的内容。但是我们可以通过拿到完整的 Secret 对象来进一步查看其数据：</p>
<pre class="lang-shell" data-nodeid="119513"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get secret -n demo regcred -o yaml</span>
apiVersion: v1
data: # 跟 configmap 一样，这块用于保存数据信息
  .dockerconfigjson: eyJhdXRocyI6eyJ5b3VycHJpdmF0ZXJlZ2lzdHJ5LmNvbSI6eyJ1c2VybmFtZSI6ImFsbGVuIiwicGFzc3dvcmQiOiJteXBhc3N3MHJkIiwiZW1haWwiOiJhbGxlbkBleGFtcGxlLmNvbSIsImF1dGgiOiJZV3hzWlc0NmJYbHdZWE56ZHpCeVpBPT0ifX19
kind: Secret
metadata:
  creationTimestamp: "2020-08-27T12:18:35Z"
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:data:
        .: {}
        f:.dockerconfigjson: {}
      f:type: {}
    manager: kubectl
    operation: Update
    time: "2020-08-27T12:18:35Z"
  name: regcred
  namespace: demo
  resourceVersion: "1419452"
  selfLink: /api/v1/namespaces/demo/secrets/regcred
  uid: 6d34123e-4d79-406b-9556-409cfb4db2e7
type: kubernetes.io/dockerconfigjson
</code></pre>
<p data-nodeid="119514">这里我们发现<code data-backticks="1" data-nodeid="119630">.dockerconfigjson</code>是一段乱码，我们用 base64 解压试试看：</p>
<pre class="lang-shell" data-nodeid="119515"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get secret regcred -n demo --output=<span class="hljs-string">"jsonpath={.data.\.dockerconfigjson}"</span> | base64 --decode</span>
{"auths":{"yourprivateregistry.com":{"username":"allen","password":"mypassw0rd","email":"allen@example.com","auth":"YWxsZW46bXlwYXNzdzByZA=="}}}
</code></pre>
<p data-nodeid="119516">这实际上跟我们通过 docker login 后的<code data-backticks="1" data-nodeid="119633">~/.docker/config.json</code>中的内容一样。<br>
至此，我们发现 Secret 和 ConfigMap 在数据保存上的最大不同。<strong data-nodeid="119641">Secret 保存的数据都是通过 base64 加密后的数据</strong>。</p>
<p data-nodeid="119517">我们平时使用较为广泛的还有另外一种<code data-backticks="1" data-nodeid="119643">Opaque</code>类型的 Secret：</p>
<pre class="lang-shell" data-nodeid="119518"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat secret-demo.yaml</span>
apiVersion: v1
kind: Secret
metadata:
  name: dev-db-secret
  namespace: demo
type: Opaque
data: # 这里的值都是 base64 加密后的
  password: UyFCXCpkJHpEc2I9
  username: ZGV2dXNlcg==
</code></pre>
<p data-nodeid="119519">或者我们也可以通过如下等价的 kubectl 命令来创建出来：</p>
<pre class="lang-shell" data-nodeid="119520"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create secret generic dev-db-secret -n demo \</span>
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='
</code></pre>
<p data-nodeid="119521">或通过文件来创建对象，比如：</p>
<pre class="lang-shell" data-nodeid="119522"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> <span class="hljs-built_in">echo</span> -n <span class="hljs-string">'username=devuser'</span> &gt; ./db_secret.txt</span>
<span class="hljs-meta">$</span><span class="bash"> <span class="hljs-built_in">echo</span> -n <span class="hljs-string">'password=S!B\*d$zDsb='</span> &gt;&gt; ./db_secret.txt</span>
<span class="hljs-meta">$</span><span class="bash"> kubectl create secret generic dev-db-secret -n demo \</span>
  --from-file=./db_secret.txt
</code></pre>
<p data-nodeid="119523">有时候为了方便，你也可以使用<code data-backticks="1" data-nodeid="119648">stringData</code>，这样可以避免自己事先手动用 base64 进行加密。</p>
<pre class="lang-shell" data-nodeid="131091"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat secret-demo-stringdata.yaml</span>
apiVersion: v1
kind: Secret
metadata:
  name: dev-db-secret
  namespace: demo
type: Opaque
stringData:
  password: devuser
  username: S!B\*d$zDsb=
</code></pre>
<p data-nodeid="131092">下面我们在 Pod 中使用 Secret：</p>
<pre class="lang-shell" data-nodeid="133524"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> cat pod-secret.yaml</span>
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
  namespace: demo
spec:
  containers:
    - name: demo-container
      image: busybox:1.28
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - secretRef:
          name: dev-db-secret
  restartPolicy: Never
<span class="hljs-meta">$</span><span class="bash"> kubectl create -f pod-secret.yaml</span>
pod/secret-test-pod created
</code></pre>
<p data-nodeid="133525">创建成功后，我们来查看下：</p>
<pre class="lang-shell" data-nodeid="135942"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo secret-test-pod</span>
NAME              READY   STATUS      RESTARTS   AGE
secret-test-pod   0/1     Completed   0          14s
<span class="hljs-meta">$</span><span class="bash"> kubectl logs -f -n demo secret-test-pod</span>
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=secret-test-pod
SHLVL=1
username=devuser
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
password=S!B\*d$zDsb=
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
</code></pre>
<p data-nodeid="135943">我们可以在日志中看到命令<code data-backticks="1" data-nodeid="135955">env</code>的输出，看到环境变量<code data-backticks="1" data-nodeid="135957">username</code>和<code data-backticks="1" data-nodeid="135959">password</code>已经正确注入。类似地，我们也可以将 Secret 作为 Volume 挂载到 Pod 内，你<del data-nodeid="135965">大家</del>可以课后实践一下。</p>
<h3 data-nodeid="136425" class="">写在最后</h3>

<p data-nodeid="135945">ConfigMap 和 Secret 是 Kubernetes 常用的保存配置数据的对象，你可以根据需要选择合适的对象存储数据。通过 Volume 方式挂载到 Pod 内的，kubelet 都会定期进行更新。但是通过环境变量注入到容器中，这样无法感知到 ConfigMap 或 Secret 的内容更新。</p>
<p data-nodeid="135946">目前如何让 Pod 内的业务感知到 ConfigMap 或 Secret 的变化，还是一个待解决的问题。但是我们还是有一些 Workaround 的。</p>
<ul data-nodeid="135947">
<li data-nodeid="135948">
<p data-nodeid="135949">如果业务自身支持 reload 配置的话，比如<code data-backticks="1" data-nodeid="135970">nginx -s reload</code>，可以通过 inotify 感知到文件更新，或者直接定期进行 reload（这里可以配合我们的 readinessProbe 一起使用）。</p>
</li>
<li data-nodeid="135950">
<p data-nodeid="135951">如果我们的业务没有这个能力，考虑到不可变基础设施的思想，我们是不是可以采用滚动升级的方式进行？没错，这是一个非常好的方法。目前有个开源工具<a href="https://github.com/stakater/Reloader" data-nodeid="135975">Reloader</a>，它就是采用这种方式，通过 watch ConfigMap 和 Secret，一旦发现对象更新，就自动触发对 Deployment 或 StatefulSet 等工作负载对象进行滚动升级。具体使用方法，可以参考项目的文档说明。</p>
</li>
</ul>
<p data-nodeid="135952">对于这个问题，社区其实也一直在讨论比较好的解法，我们可以拭目以待。</p>
<p data-nodeid="135953">好的，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **某：
> 对于 ConfigMap 或者 Secret 的“热更新”，我目前的方法比较原始。在 Deployment 对象中增加一个ConfigMap 或 Secret 发生变化，则修改这个注解的值，从而触发工作负载对象的滚动升级。

##### TUTU：
> 所以这里的一个最佳实践就是将配置信息和容器镜像进行解耦，以“不变应万变”，这句话印象深刻，对于一个不对容器不怎么精通的入门者，get到了。

