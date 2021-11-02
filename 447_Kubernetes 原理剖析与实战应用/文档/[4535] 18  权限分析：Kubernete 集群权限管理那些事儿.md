<p data-nodeid="12582">你好，我是正范。</p>


<p data-nodeid="13120" class="">通过前面的课程学习，你已经学会了使用<code data-backticks="1" data-nodeid="13122">kubectl</code>命令行，或者直接发送 REST 请求，以及使用各种语言的 <a href="https://kubernetes.io/docs/reference/using-api/client-libraries/" data-nodeid="13126">client 库</a>来跟 APIServer 进行交互。那么你是否知道在这其中Kubernetes 是如何对这些请求进行认证、授权的呢？这节课，我们就来一探究竟。</p>

<p data-nodeid="14192">任何请求访问 Kubernetes 的 kube-apiserver 时，都要依次经历三个阶段：认证（Authentication，有时简写成 AuthN）、授权（Authorization，有时简写成 AuthZ）和准入控制（Admission Control）。</p>
<p data-nodeid="14193" class=""><img src="https://s0.lgstatic.com/i/image/M00/62/1E/Ciqc1F-RVYyAeaf5AABESlN1pJg327.png" alt="image (2).png" data-nodeid="14201"></p>


<p data-nodeid="14746" class="">(<a href="http://cloudgeekz.com/1045/kubernetes-authentication-and-authorization.html" data-nodeid="14750">(http://cloudgeekz.com/1045/kubernetes-authentication-and-authorization.html)</a>)</p>

<p data-nodeid="11778">其中，认证这个概念比较好理解，就是做身份校验，解决“你是谁？”的问题。</p>
<p data-nodeid="11779"><strong data-nodeid="11880">授权</strong>负责做权限管控，解决“你能干什么？”的问题，给予你能访问 Kubernetes 集群中的哪些资源以及做哪些操作的权利，比如你是否能创建一个 Pod，是否能删除一个 Pod。</p>
<p data-nodeid="11780"><strong data-nodeid="11885">准入控制</strong>，从字面上你可能不太好理解。其实就是由一组控制逻辑级联而成，对对象进行拦截校验、更改等操作。比如你打算在一个名为 test 的 namespace 中创建一个 Pod，如果这个 namespace 不存在，集群要不要自动创建出来？或者直接拒绝掉该请求？这些逻辑都可以通过准入控制进行配置。</p>
<p data-nodeid="11781">当然如果你的 Kubernetes 集群开启了 HTTP 访问，就不再需要认证和授权流程了。显然这种配置相当于敞开大门，风险太大了，所以在生产环境我建议你不要这么配置。</p>
<p data-nodeid="11782">上述三个阶段，任何一个阶段验证失败都会导致该请求被拒绝访问，我们现在深入认识一下每个阶段。</p>
<h3 data-nodeid="15256" class="">认证</h3>

<p data-nodeid="11784">在认证阶段最重要的就是身份（Identity），我们需要从中获取到用户的相关信息。通常，Identity 信息可以通过 User 或者 Group 来定义，但是 Kubernetes 中其实并没有相关定义，所以你无法通过 Kubernetes API 接口进行创建、查询和管理。</p>
<p data-nodeid="11785">Kubernetes 认为 User 这类信息由外部系统来管理，自己并不负责管理和设计，这样可以避免重复定义用户模型，方便第三方用户权限平台进行对接。</p>
<p data-nodeid="11786">所以 Kuberentes 是如何做身份认证的呢？这里我们简单介绍一下 Kubernetes 中几种常见的用户认证方式。</p>
<h4 data-nodeid="15762" class="">x509 证书</h4>

<p data-nodeid="11788">Kubernetes 使用<a href="https://developers.cloudflare.com/access/service-auth/mtls" data-nodeid="11896">mTLS</a>进行身份验证，通过解析请求使用的 客户端（client）证书，将其中的 subject 的通用名称（Common Name）字段（例如"/CN=bob"）作为用户名。Kubernetes 集群各个组件间相互通信，都是基于 x509 证书进行身份校验的。关于证书的创建、查看等，可以参考<a href="https://kubernetes.io/zh/docs/concepts/cluster-administration/certificates/" data-nodeid="11904">官方文档</a>。</p>
<p data-nodeid="11789">比如使用<code data-backticks="1" data-nodeid="11907">openssl</code>命令行工具生成一个证书签名请求（CSR）：</p>
<pre class="lang-shell" data-nodeid="11790"><code data-language="shell">openssl req -new -key zhangsan.pem -out zhangsan-csr.pem -subj "/CN=zhangsan/O=app1/O=app2"
</code></pre>
<p data-nodeid="11791">这条命令会使用用户名zhangsan生成一个 CSR，且它属于 "app1" 和 "app2" 两个用户组，然后我们使用 CA 证书根据这个 CSR 就可以签发出一对证书。当用这对证书去访问 APIServer 时，它拿到的用户就是zhangsan了。</p>
<h4 data-nodeid="16268" class="">Token</h4>

<p data-nodeid="11793">我们接着来看基于 Token 的验证方式，它又可以分为几种类型。</p>
<p data-nodeid="11794">第一种是<strong data-nodeid="11925">用户自己提供的静态 Token</strong>，可以将这些 Token 写到一个 CSV 文件中，其中至少包含 3 列：分别为 Token、用户名和用户 ID。你可以增加一个可选列包含 group 的信息，比如：</p>
<pre class="lang-json" data-nodeid="11795"><code data-language="json">token1,user1,uid1,"group1,group2,group3" 
</code></pre>
<p data-nodeid="11796">APIServer 在启动时会通过参数<code data-backticks="1" data-nodeid="11927">--token-auth-file=xxxx</code>来指定读取的 CSV 文件。不过这种方式，实际环境中基本上不会使用到。毕竟这种静态文件不方便修改，每次修改后还需要重新启动 APIServer。</p>
<p data-nodeid="11797">还有一种是<strong data-nodeid="11938">ServiceAccount Token</strong>。这是 Kubernetes 中使用最为广泛的认证方式之一，主要用来给 Pod 提供访问 APIServer 的权限，通过使用  Volume 的方式挂载到 Pod 中。我们之前讲 Pod、Volume 的时候提到过，你也可以参考<a href="https://kubernetes.io/zh/docs/tasks/configure-pod-container/configure-service-account/" data-nodeid="11936">这份文档</a>重温下如何为 Pod 配置 ServiceAccount。</p>
<p data-nodeid="11798">当你创建一个 ServiceAccount 的时候，kube-controller-manager 会自动帮你创建出一个Secret来保存 Token，比如：</p>
<pre class="lang-shell" data-nodeid="11799"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create sa demo</span>
serviceaccount/demo created
➜  ~ kubectl get sa demo
NAME   SECRETS   AGE
demo   1         6s
➜  ~ kubectl describe sa demo
Name:                demo
Namespace:           default
Labels:              &lt;none&gt;
Annotations:         &lt;none&gt;
Image pull secrets:  &lt;none&gt;
Mountable secrets:   demo-token-fvsjg
Tokens:              demo-token-fvsjg
Events:              &lt;none&gt;
</code></pre>
<p data-nodeid="11800">可以看到这里自动创建了一个名为<code data-backticks="1" data-nodeid="11941">demo-token-fvsjg</code>的 Secret，我们来查看一下其中的内容：</p>
<pre class="lang-shell" data-nodeid="11801"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get secret demo-token-fvsjg</span>
NAME               TYPE                                  DATA   AGE
demo-token-fvsjg   kubernetes.io/service-account-token   3      27s
<span class="hljs-meta">$</span><span class="bash"> kubectl describe secret demo-token-fvsjg</span>
Name:         demo-token-fvsjg
Namespace:    default
Labels:       &lt;none&gt;
Annotations:  kubernetes.io/service-account.name: demo
              kubernetes.io/service-account.uid: f8fe4799-9add-4a36-8c29-a6b2744ba9a2
Type:  kubernetes.io/service-account-token
Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ildhc3plOFE0UXBlRE8xTFdsRThRVXktRF93T0otcW55VXI3R0RvV1IzVjgifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlbW8tdG9rZW4tZnZzamciLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVtbyIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6ImY4ZmU0Nzk5LTlhZGQtNGEzNi04YzI5LWE2YjI3NDRiYTlhMiIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlbW8ifQ.t0sNk8PiRIgMXaauzM8v8eW_AF7J5Su_TpyURaeBQX0BuFDpU-rrTC6h9sjetAQOLa6iLFsrG2tqH04pNo2N6TB73-K-M54veVpP6FGhceLO0Vsz_iTJuOsnkS_6agqfW0UrtaY8_oCk1OZipdbTJYZlmwlb6opcfP1j7bFHCdDbwfbWx8ilHqTaJ5_r7zrdhxsmC5ogtBRDaJfEYmsIBZSgnt_0qMHNj9L47n2mj_wo-aQQ25o0lbO_7RSeVA17kkgbAJ2wzwvv8-i3f7K23DSxab3k8trbuIRt1R3Gl33fv5WIwiz9okYIFfus4pe0MCtjBxxTMa526cdezKS4LQ
ca.crt:     1025 bytes
namespace:  7 bytes
</code></pre>
<p data-nodeid="18278">Data 里面的 token 我们可以到<a href="https://jwt.io/" data-nodeid="18283">https://jwt.io/</a>中 Decode 出来：</p>
<p data-nodeid="18279" class=""><img src="https://s0.lgstatic.com/i/image/M00/62/1E/Ciqc1F-RVduACXUqABUdlGrHMh4648.png" alt="image (3).png" data-nodeid="18291"></p>




<p data-nodeid="11804">从解析出来的 Payload 中可以看到， ServiceAccount 所属的 namespace、name 等信息。</p>
<p data-nodeid="11805">除此之外，还支持<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/authentication/#bootstrap-tokens" data-nodeid="11958">Bootstrap Token</a>、<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/authentication/#openid-connect-tokens" data-nodeid="11962">OpeID Connect Token (OIDC Token)</a>、<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/authentication/#webhook-token-authentication" data-nodeid="11966">Webhook Token</a>等，在此不一一介绍。</p>
<p data-nodeid="11806">值得一提的是，Bootstrap Token 是 kubeadm 引入的方便创建新集群所使用的 Token 类型，通过如下类似的命令就可以将其加入新的节点中：</p>
<pre class="lang-shell" data-nodeid="11807"><code data-language="shell">kubeadm join --discovery-token abcdef.1234567890abcdef
</code></pre>
<p data-nodeid="20328" class="">不管是上面哪种 Token，我们都可以拿来向 APIServer 发送请求，只需要向 HTTP 请求的 Header 中添加一个键为 Authorization ，值为<code data-backticks="1" data-nodeid="20330">Bearer THETOKEN</code>的字段，类似如下：</p>




<pre class="lang-json" data-nodeid="11809"><code data-language="json">Authorization: Bearer this-is-a-very-very-very-long-token 
</code></pre>
<p data-nodeid="20838">还有一种是静态用户密码文件，在 1.16 版本以前，Kubernetes 支持指定一个 CSV 文件，里面包含了用户名以及密码等信息。但是在 1.16 版本开始已经<a href="https://github.com/kubernetes/kubernetes/pull/81152" data-nodeid="20842">被废弃掉了</a>，这里我提出来是想告诉你请不要再使用这种方式了。</p>


<p data-nodeid="11812">我介绍的这些也是使用最多的认证方式，通过认证就来到了授权阶段。</p>
<h3 data-nodeid="21348" class="">授权</h3>

<p data-nodeid="21854" class="">Kubernetes 内部有多种授权模块，比如 <a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/node/" data-nodeid="21858">Node</a>、<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/abac/" data-nodeid="21862">ABAC</a>、<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/rbac/" data-nodeid="21866">RBAC</a>、<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/webhook/" data-nodeid="21870">Webhook</a>。授权阶段会根据从 AuthN 拿到的用户信息，依次按配置的授权次序逐一进行权限验证。任一授权模块验证通过，即允许该请求继续访问。</p>

<p data-nodeid="11815">我们这里简要讲一下 ABAC 和 RBAC 这两种授权模式。</p>
<h4 data-nodeid="22376" class="">ABAC</h4>

<p data-nodeid="11817">ABAC (Attribute Based Access Control) 是一种基于属性的访问控制，可以给 APIServer 指定一个 JSON 文件<code data-backticks="1" data-nodeid="11999">--authorization-policy-file=SOME_FILENAME</code>，该文件描述了一组属性组合策略，比如：</p>
<pre class="lang-json" data-nodeid="11818"><code data-language="json">{<span class="hljs-attr">"apiVersion"</span>: <span class="hljs-string">"abac.authorization.kubernetes.io/v1beta1"</span>, <span class="hljs-attr">"kind"</span>: <span class="hljs-string">"Policy"</span>, <span class="hljs-attr">"spec"</span>: {<span class="hljs-attr">"user"</span>: <span class="hljs-string">"zhangsan"</span>, <span class="hljs-attr">"namespace"</span>: <span class="hljs-string">"*"</span>, <span class="hljs-attr">"resource"</span>: <span class="hljs-string">"pods"</span>, <span class="hljs-attr">"readonly"</span>: <span class="hljs-literal">true</span>}}
</code></pre>
<p data-nodeid="22882">这条策略表示，用户 zhangsan 可以以<strong data-nodeid="22893">只读</strong>的方式读取任何 namespace 中的 Pod。这里有一份<a href="https://raw.githubusercontent.com/kubernetes/kubernetes/master/pkg/auth/authorizer/abac/example_policy_file.jsonl" data-nodeid="22891">示例文件</a>，你可以参考一下。</p>
<p data-nodeid="22883">不过在实际使用中，ABAC 使用得比较少，跟我们上面 AuthN 中提到的静态 Token 一样，不方便修改、更新，每次变更后都需要重启 APIServer。所以在实际使用中，RBAC 最常见，使用更广泛。</p>

<h4 data-nodeid="23399" class="">RBAC</h4>

<p data-nodeid="11821">相对而言，RBAC 使用起来就非常方便了，通过 Kubernetes 的对象就可以直接进行管理，也便于动态调整权限。</p>
<p data-nodeid="11822">RBAC 中引入了<strong data-nodeid="12020">角色</strong>，所有的权限都是围着这个角色进行展开的，每个角色里面定义了可以操作的资源以及操作方式。在 Kubernetes 中有两种角色，一种是 namespace 级别的 Role ，一种是集群级别的 ClusterRole 。</p>
<p data-nodeid="11823">在 Kubernetes 中有些资源是 namespace 级别的，比如 Pod、Service 等，而有些则属于集群级别，比如 Node、PV 等。所以有了 Role 和 ClusterRole 两种角色。</p>
<p data-nodeid="11824">我们来看个例子：</p>
<pre class="lang-shell" data-nodeid="11825"><code data-language="shell">kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # 空字符串""表明使用core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
</code></pre>
<p data-nodeid="23905">这样一个角色表示可以访问 default 命名空间下面的 Pods，并可以对其进行 get、watch 以及 list 操作。</p>
<p data-nodeid="23906">ClusterRole 除了可以定义集群方位的资源外，比如 Node，还可以定义跨 namespace 的资源访问，比如你想访问所有命名空间下面的 Pod，就可以这么定义：</p>

<pre class="lang-shell" data-nodeid="11827"><code data-language="shell">kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
<span class="hljs-meta">  #</span><span class="bash"> 鉴于ClusterRole是集群范围对象，所以这里不需要定义<span class="hljs-string">"namespace"</span>字段</span>
  name: pods-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
</code></pre>
<p data-nodeid="11828">RBAC 中定义这些角色包含了一组权限规则，这些规则纯粹以累加形式组合，没有互斥的概念。<br>
定义好角色，我们就可以做权限绑定了。这里分别有两个 API 对象，RoleBinding 和 ClusterRoleBinding。角色绑定就是把一个权限授予一个或者一组用户，例如：</p>
<pre class="lang-shell" data-nodeid="11829"><code data-language="shell">apiVersion: rbac.authorization.k8s.io/v1
<span class="hljs-meta">#</span><span class="bash"> 此角色绑定使得用户 <span class="hljs-string">"jane"</span> 或者 <span class="hljs-string">"manager"</span>组（Group）能够读取 <span class="hljs-string">"default"</span> 命名空间中的 Pods</span>
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane # 名称区分大小写
  apiGroup: rbac.authorization.k8s.io
- kind: Group
  name: manager # 名称区分大小写
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role # 这里可以是 Role，也可以是 ClusterRole
  name: pods-reader # 这里的名称必须与你想要绑定的 Role 或 ClusterRole 名称一致
  apiGroup: rbac.authorization.k8s.io
</code></pre>
<p data-nodeid="24919">这里定义的 RoleBinding 对象在 default 命名空间中将 pods-reader 角色授予用户 jane和组 manager。 这一授权将允许用户 jane 和 manager 组中的各个用户从default 命名空间中读取到 Pod。</p>
<p data-nodeid="24920">好了，最后就是准入控制了。</p>

<h3 data-nodeid="24413" class="">准入控制</h3>

<p data-nodeid="11832">准入控制可以帮助我们在 APIServer 真正处理对象前做一些校验以及修改的工作。</p>
<p data-nodeid="11833">这里官方列出了<a href="https://kubernetes.io/zh/docs/reference/access-authn-authz/admission-controllers/#%E6%AF%8F%E4%B8%AA%E5%87%86%E5%85%A5%E6%8E%A7%E5%88%B6%E5%99%A8%E7%9A%84%E4%BD%9C%E7%94%A8%E6%98%AF%E4%BB%80%E4%B9%88" data-nodeid="12037">很多准入控制组件</a>，官方已经将使用方法写得很清楚了，你可以根据自己的需要，选择合适的准入控制插件，我就不再赘述了。</p>
<h3 data-nodeid="25427" class="">写在最后</h3>

<p data-nodeid="11835">作为一个企业级的容器管理调度平台，Kubernetes 在 Auth 方面设计得很完善，支持多种后端身份认证授权系统，比如 LDAP (Lightweight Directory Access Protocol)、Webhook 等。AuthN 负责完成对用户的认证，并获取用户的相关信息（比如 Username、Groups 等），而 AuthZ 则会根据 AuthN 返回的用户信息，根据配置的授权模式进行权限校验，来决定是否允许对某个 API 资源的操作请求。</p>
<p data-nodeid="11836">到这里这节课就结束了，如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **其：
> 总算对权限有点浅薄的认识了

