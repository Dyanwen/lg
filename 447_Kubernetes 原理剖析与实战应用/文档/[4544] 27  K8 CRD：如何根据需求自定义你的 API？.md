<p data-nodeid="2948">你好，我是正范。</p>


<p data-nodeid="2587">随着使用的深入，你会发现 Kubernetes 中内置的对象定义，比如 Deployment、StatefulSet、Configmap，可能已经不能满足你的需求了。你很希望在 Kubernetes 定义一些自己的对象，一来可以通过 kube-apiserver 提供统一的访问入口，二来可以像其他内置对象一样，通过 kubectl 命令管理这些自定义的对象。</p>
<p data-nodeid="2588">Kubernetes 中提供了两种自定义对象的方式，一种是聚合 API，另一种是 CRD。</p>
<p data-nodeid="2589">我们今天就来看看。</p>
<h3 data-nodeid="2590">聚合 API</h3>
<p data-nodeid="2591">聚合 API（Aggregation API，AA）是自 Kubernetes v1.7 版本就引入的功能，主要目的是方便用户将自己定义的 API 注册到 kube-apiserver 中，并且可以像使用其他内置的 API 一样，通过 APIServer 的 URL 就可以访问和操作。</p>
<p data-nodeid="2592">在使用 API 聚合之前，我们还需要通过一些参数在 kube-apiserver 中启用这个功能：</p>
<pre class="lang-shell" data-nodeid="2593"><code data-language="shell">--requestheader-client-ca-file=&lt;path to aggregator CA cert&gt;
--requestheader-allowed-names=front-proxy-client
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--proxy-client-cert-file=&lt;path to aggregator proxy cert&gt;
--proxy-client-key-file=&lt;path to aggregator proxy key&gt;
</code></pre>
<p data-nodeid="2594">Kubernetes 在 kube-apiserver 中引入了一个 API 聚合层（API Aggregation Layer），可以将访问请求转发到后端用户自己的扩展 APIServer 中。你可以参考这个<a href="https://kubernetes.io/zh/docs/tasks/extend-kubernetes/setup-extension-api-server/" data-nodeid="2643">文档</a>学习如何安装一个扩展的 APIServer。当然，扩展的 APIServer 需要你自己修改并实现相关的代码逻辑。</p>
<ul data-nodeid="2595">
<li data-nodeid="2596">
<p data-nodeid="2597">Kubernetes apiserver 会判断 --requestheader-client-ca-file 指定的 CA 证书中的 CN 是否是 --requestheader-allowed-names 提供的列表名称之一。如果不是，则该请求被拒绝。如果名称允许，则请求会被转发。</p>
</li>
<li data-nodeid="2598">
<p data-nodeid="2599">接着 Kubernetes apiserver 将使用由 --proxy-client-*-file 指定的文件来访问用户的扩展 APIServer。</p>
</li>
</ul>
<p data-nodeid="3420">下图就是用户发起请求后一个完整的身份认证流程，你可以阅读<a href="https://kubernetes.io/zh/docs/tasks/extend-kubernetes/configure-aggregation-layer/#%E8%BA%AB%E4%BB%BD%E8%AE%A4%E8%AF%81%E6%B5%81%E7%A8%8B" data-nodeid="3425">官方文档</a>来详细了解步骤 。</p>
<p data-nodeid="3670"><img src="https://s0.lgstatic.com/i/image/M00/71/C8/CgqCHl-_cp6AR0muAAIPqi0uDMk927.png" alt="Drawing 0.png" data-nodeid="3674"></p>
<div data-nodeid="3671" class=""><p style="text-align:center">图 1：一个完整的身份认证流程</p></div>





<p data-nodeid="2603">配置好了上述参数后， 为了让 kube-apiserver 知道我们自定义的 API，我们需要创建一个 APIService 的对象，比如：</p>
<pre class="lang-yaml" data-nodeid="2604"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apiregistration.k8s.io/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">APIService</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">v1alpha1.wardle.example.com</span> <span class="hljs-comment"># 对象名称</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">insecureSkipTLSVerify:</span> <span class="hljs-literal">true</span>
  <span class="hljs-attr">group:</span> <span class="hljs-string">wardle.example.com</span> <span class="hljs-comment"># 扩展 Apiserver 的 API group 名称</span>
  <span class="hljs-attr">groupPriorityMinimum:</span> <span class="hljs-number">1000</span> <span class="hljs-comment"># APIService 对对应 group 的优先级</span>
  <span class="hljs-attr">versionPriority:</span> <span class="hljs-number">15</span> <span class="hljs-comment"># 优先考虑 version 在 group 中的排序</span>
  <span class="hljs-attr">service:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">myapi</span> <span class="hljs-comment"># 扩展 Apiserver 服务的 name</span>
    <span class="hljs-attr">namespace:</span> <span class="hljs-string">wardle</span> <span class="hljs-comment"># 扩展 Apiserver 服务的 namespace </span>
  <span class="hljs-attr">version:</span> <span class="hljs-string">v1alpha1</span> <span class="hljs-comment"># 扩展 Apiserver 的 API version</span>
</code></pre>
<p data-nodeid="2605">通过上述 YAML 文件，我们就在 kube-apiserver 中创建一个 API 对象组，其对应的组（Group）为 wardle.example.com，版本号为 v1alpha1。创建成功后，我们就可以通过 /apis/wardle.example.com/v1alpha1 这个路径访问了。至于这个对象组里提供了哪些对象，就需要我们在扩展的 APIServer 中声明出来了。</p>
<p data-nodeid="2606">那 kube-apiserver 又是怎么知道将 /apis/wardle.example.com/v1alpha1 这个路径的所有请求转发到后端的扩展 APIServer 中的呢？我们注意到上面的 APIService 定义中，还有一个 spec.service 字段，就是在这里，我们指定了扩展 APIServer 的服务名，也就是说 kube-apiserver 会将 /apis/wardle.example.com/v1alpha1 这个路径的所有访问通过 API 聚合层转发到后端的服务 myapi.wardle.svc 上。</p>
<p data-nodeid="2607">扩展 APIServer 的具体代码设计及逻辑，你可以参考<a href="https://github.com/kubernetes/sample-apiserver" data-nodeid="2664">sample-apiserver</a>或者使用<a href="https://github.com/kubernetes-incubator/apiserver-builder" data-nodeid="2668">apiserver-builder</a>。</p>
<p data-nodeid="2608">这种聚合 API 的方式对代码要求很高，但支持对 API 行为进行完全的掌控，比如你可以自己定义数据如何存储、API 各个版本的切换、各个对象的逻辑控制。</p>
<p data-nodeid="2609">如果你只想简简单单地在 Kubernetes 中定义个对象，可以直接通过下面要介绍的 CRD 定义。</p>
<h3 data-nodeid="2610">CRD</h3>
<p data-nodeid="2611">CRD（CustomResourceDefinitions）在 v1.7 刚引入进来的时候，其实是 ThirdPartyResources（TPR）的升级版本，而 TPR 在 v1.8 的版本被剔除了。CRD 目前使用非常广泛，各个周边项目都在使用它，比如 Ingress、Rancher。</p>
<p data-nodeid="2612">我们来看一下官方的一个例子，通过如下的 YAML 文件，我们可以创建一个 API：</p>
<pre class="lang-yaml" data-nodeid="2613"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apiextensions.k8s.io/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">CustomResourceDefinition</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-comment"># 名字必须与下面的 spec 字段匹配，并且格式为 '&lt;名称的复数形式&gt;.&lt;组名&gt;'</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">crontabs.stable.example.com</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-comment"># 组名称，用于 REST API: /apis/&lt;组&gt;/&lt;版本&gt;</span>
  <span class="hljs-attr">group:</span> <span class="hljs-string">stable.example.com</span>
  <span class="hljs-comment"># 列举此 CustomResourceDefinition 所支持的版本</span>
  <span class="hljs-attr">versions:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">v1</span>
      <span class="hljs-comment"># 每个版本都可以通过 served 标志来独立启用或禁止</span>
      <span class="hljs-attr">served:</span> <span class="hljs-literal">true</span>
      <span class="hljs-comment"># 其中一个且只有一个版本必需被标记为存储版本</span>
      <span class="hljs-attr">storage:</span> <span class="hljs-literal">true</span>
      <span class="hljs-attr">schema:</span>
        <span class="hljs-attr">openAPIV3Schema:</span>
          <span class="hljs-attr">type:</span> <span class="hljs-string">object</span>
          <span class="hljs-attr">properties:</span>
            <span class="hljs-attr">spec:</span>
              <span class="hljs-attr">type:</span> <span class="hljs-string">object</span>
              <span class="hljs-attr">properties:</span>
                <span class="hljs-attr">cronSpec:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">string</span>
                <span class="hljs-attr">image:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">string</span>
                <span class="hljs-attr">replicas:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">integer</span>
  <span class="hljs-comment"># 可以是 Namespaced 或 Cluster</span>
  <span class="hljs-attr">scope:</span> <span class="hljs-string">Namespaced</span>
  <span class="hljs-attr">names:</span>
    <span class="hljs-comment"># 名称的复数形式，用于 URL：/apis/&lt;组&gt;/&lt;版本&gt;/&lt;名称的复数形式&gt;</span>
    <span class="hljs-attr">plural:</span> <span class="hljs-string">crontabs</span>
    <span class="hljs-comment"># 名称的单数形式，作为命令行使用时和显示时的别名</span>
    <span class="hljs-attr">singular:</span> <span class="hljs-string">crontab</span>
    <span class="hljs-comment"># kind 通常是单数形式的驼峰编码（CamelCased）形式。你的资源清单会使用这一形式。</span>
    <span class="hljs-attr">kind:</span> <span class="hljs-string">CronTab</span>
    <span class="hljs-comment"># shortNames 允许你在命令行使用较短的字符串来匹配资源</span>
    <span class="hljs-attr">shortNames:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">ct</span>
</code></pre>
<p data-nodeid="2614">我们可以像创建其他对象一样，通过 kubectl create 命令创建。创建好了以后，一个类型为 CronTab 的对象就在 kube-apiserver 中注册好了，你可以通过如下的 REST 接口访问，比如 LIST 命名空间 ns1 下的 CronTab 对象，可以通过这个 URL“/apis/stable.example.com/v1/namespaces/ns1/crontabs/”访问。这种接口跟 Kubernetes 内置的其他对象的接口风格是一模一样的。</p>
<p data-nodeid="2615">声明好了 CronTab，下面我们就来看看如何创建一个 CronTab 类型的对象。我们来看看官方给的一个例子：</p>
<pre class="lang-yaml" data-nodeid="2616"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">"stable.example.com/v1"</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">CronTab</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">my-new-cron-object</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">cronSpec:</span> <span class="hljs-string">"* * * * */5"</span>
  <span class="hljs-attr">image:</span> <span class="hljs-string">my-awesome-cron-image</span>
</code></pre>
<p data-nodeid="2617">通过 kubectl create 创建 my-new-cron-object 后，就可以通过 kubectl get 查看，并使用 kubectl 管理你的 CronTab 对象了。例如：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="4622"><code data-language="java">kubectl get crontab
NAME                 AGE
my-<span class="hljs-keyword">new</span>-cron-object   <span class="hljs-number">6</span>s
</code></pre>




<p data-nodeid="2619">这里的资源名是大小写不敏感的，我们在这里可以使用缩写 kubectl get ct，也可以使用 kubectl get crontabs。</p>
<p data-nodeid="2620">同时原生内置的 API 对象一样，这些 CRD 不仅可以通过 kubectl 来创建、查看、修改，删除等操作，还可以给其配置 RBAC 规则。</p>
<p data-nodeid="2621">我们还能开发自定义的控制器，来感知和操作这些自定义的 API。这一部分我会在下一讲中介绍。</p>
<p data-nodeid="2622">我们在创建 CRD 的时候，还可以一起定义 /status 和 /scale 子资源，如下所示：</p>
<pre class="lang-yaml" data-nodeid="2623"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apiextensions.k8s.io/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">CustomResourceDefinition</span>
<span class="hljs-attr">metadata:</span>
  <span class="hljs-attr">name:</span> <span class="hljs-string">crontabs.stable.example.com</span>
<span class="hljs-attr">spec:</span>
  <span class="hljs-attr">group:</span> <span class="hljs-string">stable.example.com</span>
  <span class="hljs-attr">versions:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">v1</span>
      <span class="hljs-attr">served:</span> <span class="hljs-literal">true</span>
      <span class="hljs-attr">storage:</span> <span class="hljs-literal">true</span>
      <span class="hljs-attr">schema:</span>
        <span class="hljs-attr">openAPIV3Schema:</span>
          <span class="hljs-attr">type:</span> <span class="hljs-string">object</span>
          <span class="hljs-attr">properties:</span>
            <span class="hljs-attr">spec:</span>
              <span class="hljs-attr">type:</span> <span class="hljs-string">object</span>
              <span class="hljs-attr">properties:</span>
                <span class="hljs-attr">cronSpec:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">string</span>
                <span class="hljs-attr">image:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">string</span>
                <span class="hljs-attr">replicas:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">integer</span>
            <span class="hljs-attr">status:</span>
              <span class="hljs-attr">type:</span> <span class="hljs-string">object</span>
              <span class="hljs-attr">properties:</span>
                <span class="hljs-attr">replicas:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">integer</span>
                <span class="hljs-attr">labelSelector:</span>
                  <span class="hljs-attr">type:</span> <span class="hljs-string">string</span>
      <span class="hljs-comment"># subresources 描述定制资源的子资源</span>
      <span class="hljs-attr">subresources:</span>
        <span class="hljs-comment"># status 启用 status 子资源</span>
        <span class="hljs-attr">status:</span> <span class="hljs-string">{}</span>
        <span class="hljs-comment"># scale 启用 scale 子资源</span>
        <span class="hljs-attr">scale:</span>
          <span class="hljs-comment"># specReplicasPath 定义定制资源中对应 scale.spec.replicas 的 JSON 路径</span>
          <span class="hljs-attr">specReplicasPath:</span> <span class="hljs-string">.spec.replicas</span>
          <span class="hljs-comment"># statusReplicasPath 定义定制资源中对应 scale.status.replicas 的 JSON 路径 </span>
          <span class="hljs-attr">statusReplicasPath:</span> <span class="hljs-string">.status.replicas</span>
          <span class="hljs-comment"># labelSelectorPath  定义定制资源中对应 scale.status.selector 的 JSON 路径 </span>
          <span class="hljs-attr">labelSelectorPath:</span> <span class="hljs-string">.status.labelSelector</span>
  <span class="hljs-attr">scope:</span> <span class="hljs-string">Namespaced</span>
  <span class="hljs-attr">names:</span>
    <span class="hljs-attr">plural:</span> <span class="hljs-string">crontabs</span>
    <span class="hljs-attr">singular:</span> <span class="hljs-string">crontab</span>
    <span class="hljs-attr">kind:</span> <span class="hljs-string">CronTab</span>
    <span class="hljs-attr">shortNames:</span>
    <span class="hljs-bullet">-</span> <span class="hljs-string">ct</span>
</code></pre>
<p data-nodeid="2624">这些子资源的定义，可以帮助我们在更新对象的时候，只更改期望的部分，比如只更新 status 部分，避免误更新 spec 部分。</p>
<p data-nodeid="2625">同时 CRD 定义的时候还支持<a href="https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#validation" data-nodeid="2686">合法性检查</a>、<a href="https://kubernetes.io/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/#efaulting" data-nodeid="2690">设置默认值</a>等操作，你可以参照文档来实践。</p>
<h3 data-nodeid="2626">写在最后</h3>
<p data-nodeid="2627">当然，你也可以选择在其他地方定义新的 API，你可以参考这份<a href="https://kubernetes.io/zh/docs/concepts/extend-kubernetes/api-extension/custom-resources/#%E6%88%91%E6%98%AF%E5%90%A6%E5%BA%94%E8%AF%A5%E5%90%91%E6%88%91%E7%9A%84-kubernetes-%E9%9B%86%E7%BE%A4%E6%B7%BB%E5%8A%A0%E5%AE%9A%E5%88%B6%E8%B5%84%E6%BA%90" data-nodeid="2696">文档</a>确定是否需要在 Kubernetes 中定义 API，还是让你的 API 独立运行。</p>
<p data-nodeid="2628">你可以通过这个<a href="https://kubernetes.io/zh/docs/concepts/extend-kubernetes/api-extension/custom-resources/#advanced-features-and-flexibility" data-nodeid="2701">网址</a>中的内容来比较 CRD 和 聚合 API 的功能差异。简单来说，CRD 更为易用，而聚合 API 更为灵活。</p>
<p data-nodeid="2629">Kubernetes 提供这两种选项以满足不同用户的需求，这样就既不会牺牲易用性也不会牺牲灵活性。</p>
<p data-nodeid="2630">对于聚合 API 和 CRD，你还有什么问题的话，欢迎在留言区留言。</p>
<p data-nodeid="2631">下一讲，我将介绍如何通过 Operator 扩展 Kubernetes API。</p>

---

### 精选评论


