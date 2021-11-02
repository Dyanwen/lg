<p data-nodeid="156127">在上两节课中，我们已经了解了 Kubernetes 中最关键的对象 Pod，也学习了一些 Pod 的常见用法。</p>
<p data-nodeid="156128">每一个 Pod 都是应用的一个实例，但是通常来说你不会直接在 Kubernetes 中创建和运行单个 Pod。因为 Pod 的生命周期是短暂的，即“用后即焚”。理解这一点很重要，这也是“不可变基础设施”这一理念在 Kubernetes 中的最佳实践。同样，对于你后续进行业务改造和容器化上云也具有指导意义。</p>
<p data-nodeid="156129">当一个 Pod 被创建出来，不管是由你直接创建，还是由其他工作负载控制器（Workload Controller）自动创建，经过调度器调度以后，就永久地“长”在某个节点上了，直到该 Pod 被删除，或者因为资源不够被驱逐，抑或由于对应的节点故障导致宕机等。因此单独地用一个 Pod 来承载业务，是没办法保证高可用、可伸缩、负载均衡等要求，而且 Pod 也无法“自愈”。</p>
<p data-nodeid="156130">这时我们就需要在 Pod 之上做一层抽象，通过多个副本（Replica）来保证可用 Pod 的数量，避免业务不可用。在介绍 Kubernetes 中对这种抽象的实现之前，我们先来看看应用的业务类型。</p>
<h3 data-nodeid="156131">有状态服务 VS 无状态服务</h3>
<p data-nodeid="156132">一般来说，业务的服务类型可分为无状态服务和有状态服务。举个简单的例子，像打网络游戏这类的服务，就是有状态服务，而正常浏览网页这类服务一般都是无状态服务。其实判断两种请求的关键在于，两个来自相同发起者的请求在服务器端是否具备上下文关系。</p>
<ul data-nodeid="156133">
<li data-nodeid="156134">
<p data-nodeid="156135">如果是有状态服务，其请求是状态化的，服务器端需要保存请求的相关信息，这样每个请求都可以默认地使用之前的请求上下文。</p>
</li>
<li data-nodeid="156136">
<p data-nodeid="156137">而无状态服务就不需要这样，每次请求都包含了需要的所有信息，每次请求都和之前的没有任何关系。</p>
</li>
</ul>
<p data-nodeid="156138">有状态服务和无状态服务分别有各自擅长的业务类型和技术优势，在 Kubernetes 中，分别有不同的工作负载控制器来负责承载这两类服务。</p>
<p data-nodeid="156139">本节课，我们先介绍 Kubernetes 中的无状态工作负载。</p>
<h3 data-nodeid="156140">Kubernetes 中的无状态工作负载</h3>
<p data-nodeid="157743" class="">Kubernetes 中各个对象的 metadata 字段都有 <a href="https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/" data-nodeid="157747">label</a>（标签）和 <a href="https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/annotations/" data-nodeid="157751">annotation</a>（注解） 两个对象，可以用来标识一些元数据信息。不论是 label 还是 annotation，都是一组键值对，键值不允许相同。关于各个对象的 API 基本构成，可以参照 04 讲中 API 的定义说明，Kubernetes 的各个对象的 API 都符合这个规范。</p>


<ul data-nodeid="156142">
<li data-nodeid="156143">
<p data-nodeid="156144">annotation 主要用来记录一些非识别的信息，并不用于标识和选择对象。</p>
</li>
<li data-nodeid="156145">
<p data-nodeid="156146">label 主要用来标识一些有意义且和对象密切相关的信息，用来支持labelSelector（标签选择器）以及一些查询操作，还有选择对象。</p>
</li>
</ul>
<p data-nodeid="156147">为了让这种抽象的对象可以跟 Pod 关联起来，Kubernetes 使用了labelSelector来跟 label 进行选择匹配，从而达到这种<strong data-nodeid="156231">松耦合</strong>的关联效果。</p>
<pre class="lang-shell" data-nodeid="156148"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -l label1=value1,label2=value2 -n my-namespace</span>
</code></pre>
<p data-nodeid="156149">比如，我们就可以通过上述命令，查询出 my-namespace 这个命名空间下面，带有标签<code data-backticks="1" data-nodeid="156233">label1=value1</code>和<code data-backticks="1" data-nodeid="156235">label2=value2</code>的 pod。label 中的键值对在匹配的时候是“<strong data-nodeid="156241">且</strong>”的关系。</p>
<h4 data-nodeid="156150">ReplicationController</h4>
<p data-nodeid="156151">Kubernetes 中有一系列的工作负载可以用来部署无状态服务。在最初，Kubernetes 中使用了ReplicationController来做 Pod 的副本控制，即确保该服务的 Pod 数量维持在特定的数量。为了简洁并便于使用，ReplicationController通常缩写为“rc”，并作为 kubectl 命令的快捷方式，例如：</p>
<pre class="lang-shell" data-nodeid="156152"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get rc -n my-namespace</span>
</code></pre>
<p data-nodeid="156153">如果副本数少于预定的值，则创建新的 Pod。如果副本数大于预定的值，就删除多余的副本。因此即使你的业务应用只需要一个 Pod，你也可以使用 rc 来自动帮你维护和创建 Pod。</p>
<h4 data-nodeid="156154">ReplicaSet</h4>
<p data-nodeid="156155">随后社区开发了下一代的 Pod 控制器 ReplicaSet（可简写为 rs） 用来替代 ReplicaController。虽然 ReplicaController 目前依然可以使用，但是社区已经不推荐继续使用了。这两者的功能和目的完全相同，但是 ReplicaSet 具备更强大的<a href="https://kubernetes.io/zh/docs/concepts/overview/working-with-objects/labels/#%E5%9F%BA%E4%BA%8E%E9%9B%86%E5%90%88-%E7%9A%84%E9%9C%80%E6%B1%82" data-nodeid="156249">基于集合的标签选择器</a>，这样你可以通过一组值来进行标签匹配选择。目前支持三种操作符：<code data-backticks="1" data-nodeid="156251">in</code>、<code data-backticks="1" data-nodeid="156253">notin</code>和<code data-backticks="1" data-nodeid="156255">exists</code>。</p>
<p data-nodeid="156156">例如，你可以用<code data-backticks="1" data-nodeid="156258">environment in (production, qa)</code>来匹配 label 中带有<code data-backticks="1" data-nodeid="156260">environment=production</code>或<code data-backticks="1" data-nodeid="156262">environment=qa</code>的 Pod。</p>
<p data-nodeid="156157">同样你也可以使用<code data-backticks="1" data-nodeid="156265">tier notin (frontend,backend)</code>来匹配 label 中不带有<code data-backticks="1" data-nodeid="156267">tier=frontend</code>或<code data-backticks="1" data-nodeid="156269">tier=backend</code>的 Pod。</p>
<p data-nodeid="156158">或者你可以用 partition来匹配 label 中带有 partition 这个 key 的 Pod。</p>
<p data-nodeid="156159">了解了标签选择器，我们就可以通过如下的 kubectl 命令查找 Pod：</p>
<pre class="lang-dart" data-nodeid="160951"><code data-language="dart">kubectl <span class="hljs-keyword">get</span> pods -l environment=production,tier=frontend
</code></pre>







<p data-nodeid="156161">或者使用：</p>
<pre class="lang-dart" data-nodeid="161408"><code data-language="dart">kubectl <span class="hljs-keyword">get</span> pods -l <span class="hljs-string">'environment in (production),tier in (frontend)'</span>
</code></pre>

<p data-nodeid="156163">虽然 Replicaset 可以独立使用，但是为了能够更好地协调 Pod 的创建、删除以及更新等操作，我们都是直接<strong data-nodeid="156283">使用更高级的 Deployment</strong>来管理 Replicaset，社区也是一直这么定位和推荐的。比如一些业务升级的场景，使用单一的 ReplicaController 或者 Replicaset 是无法实现滚动升级的诉求，至少需要定义两个该对象才能实现，而且这两个对象使用的标签选择器中的 label 至少要有一个不相同。通过不断地对这两个对象的副本进行增减，也可以称为<strong data-nodeid="156284">调和</strong>（Reconcile），才可以完成滚动升级。这样使用起来不方便，也增加了用户的使用门槛，极大地降低了业务发布的效率。</p>
<h4 data-nodeid="156164">Deployment</h4>
<p data-nodeid="162313">通过 Deployment，我们就不需要再关心和操作 ReplicaSet 了。</p>
<p data-nodeid="162787" class=""><img src="https://s0.lgstatic.com/i/image/M00/4D/56/CgqCHl9Z5ayANEsBAAPzg_vpPBs686.png" alt="image (3).png" data-nodeid="162794"><br>
（<a href="https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/07751442-deployment.png" data-nodeid="162799">https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/07751442-deployment.png</a>）</p>




<p data-nodeid="156168">Deployment、ReplicaSet 和 Pod 这三者之间的关系见上图。通过 Deployment，我们可以管理多个 label 各不相同的 ReplicaSet，每个 ReplicaSet 负责保证对应数目的 Pod 在运行。</p>
<p data-nodeid="156169">我们来看一个定义 Deployment 的例子：</p>
<pre class="lang-yaml" data-nodeid="156170"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span>
<span class="hljs-attr">metadata:</span>
&nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">nginx-deployment-demo</span>
&nbsp; <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span>
&nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
<span class="hljs-attr">spec:</span>
&nbsp; <span class="hljs-attr">replicas:</span> <span class="hljs-number">3</span>
&nbsp; <span class="hljs-attr">selector:</span>
&nbsp; &nbsp; <span class="hljs-attr">matchLabels:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; <span class="hljs-attr">template:</span>
&nbsp; &nbsp; <span class="hljs-attr">metadata:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">version:</span> <span class="hljs-string">v1</span>
&nbsp; &nbsp; <span class="hljs-attr">spec:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">containers:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">image:</span> <span class="hljs-string">nginx:1.14.2</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">ports:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="156171">我们这里定义了副本数<code data-backticks="1" data-nodeid="156298">spec.replicas</code>为 3，同时在<code data-backticks="1" data-nodeid="156300">spec.selector.matchLabels</code>中设置了<code data-backticks="1" data-nodeid="156302">app=nginx</code>的 label，用于匹配<code data-backticks="1" data-nodeid="156304">spec.template.metadata.labels</code>的 label。我们还增加了<code data-backticks="1" data-nodeid="156306">version=v1</code>的 label 用于后续滚动升级做比较。</p>
<p data-nodeid="156172">我们将上述内容保存到<code data-backticks="1" data-nodeid="156309">deploy-demo.yaml</code>文件中。</p>
<p data-nodeid="156173">注意，<code data-backticks="1" data-nodeid="156312">spec.selector.matchLabels</code>中写的 label 一定要能匹配得了<code data-backticks="1" data-nodeid="156314">spec.template.metadata.labels</code>中的 label。否则该 Deployment 无法关联创建出来的 ReplicaSet。</p>
<p data-nodeid="156174">然后我们通过下面这些命令，便可以在命名空间 demo 中创建名为 nginx-deployment-demo 的 Deployment。</p>
<pre class="lang-shell" data-nodeid="156175"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl create ns demo</span>
<span class="hljs-meta">$</span><span class="bash"> kubectl create -f deploy-demo.yaml</span>
deployment.apps/nginx-deployment-demo created
</code></pre>
<p data-nodeid="156176">创建完成后，我们可以查看自动创建出来的 rs。</p>
<pre class="lang-shell" data-nodeid="156177"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get rs -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DESIRED&nbsp; &nbsp;CURRENT&nbsp; &nbsp;READY&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9&nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp;5s
</code></pre>
<p data-nodeid="156178">接下来，我们可以通过如下的几个命令不断地查询系统，看看对应的 Pod 是否运行成功。</p>
<pre class="lang-shell" data-nodeid="156179"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
NAME                                     READY   STATUS              RESTARTS   AGE
nginx-deployment-demo-5d65f98bd9-7w5gp   0/1     ContainerCreating   0          30s
nginx-deployment-demo-5d65f98bd9-d78fx   0/1     ContainerCreating   0          30s
nginx-deployment-demo-5d65f98bd9-ssstk   0/1     ContainerCreating   0          30s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
NAME                                     READY   STATUS    RESTARTS   AGE
nginx-deployment-demo-5d65f98bd9-7w5gp   1/1     Running   0          63s
nginx-deployment-demo-5d65f98bd9-d78fx   1/1     Running   0          63s
nginx-deployment-demo-5d65f98bd9-ssstk   1/1     Running   0          63s
</code></pre>
<p data-nodeid="156180">我们可以看到，从 Deployment 到这里，最后的 Pod 已经创建成功了。</p>
<p data-nodeid="156181">现在我们试着做下镜像更新看看。更改<code data-backticks="1" data-nodeid="156321">spec.template.metadata.labels</code>中的<code data-backticks="1" data-nodeid="156323">version=v1</code>为<code data-backticks="1" data-nodeid="156325">version=v2</code>，同时更新镜像<code data-backticks="1" data-nodeid="156327">nginx:1.14.2</code>为<code data-backticks="1" data-nodeid="156329">nginx:1.19.2</code>。</p>
<p data-nodeid="156182">你可以直接通过下述命令来直接更改：</p>
<pre class="lang-shell" data-nodeid="156183"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl edit deploy nginx-deployment-demo -n demo </span>
</code></pre>
<p data-nodeid="156184">也可以更改<code data-backticks="1" data-nodeid="156333">deploy-demo.yaml</code>这个文件：</p>
<pre class="lang-yaml" data-nodeid="156185"><code data-language="yaml"><span class="hljs-attr">apiVersion:</span> <span class="hljs-string">apps/v1</span>
<span class="hljs-attr">kind:</span> <span class="hljs-string">Deployment</span>
<span class="hljs-attr">metadata:</span>
&nbsp; <span class="hljs-attr">name:</span> <span class="hljs-string">nginx-deployment-demo</span>
&nbsp; <span class="hljs-attr">namespace:</span> <span class="hljs-string">demo</span>
&nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
<span class="hljs-attr">spec:</span>
&nbsp; <span class="hljs-attr">replicas:</span> <span class="hljs-number">3</span>
&nbsp; <span class="hljs-attr">selector:</span>
&nbsp; &nbsp; <span class="hljs-attr">matchLabels:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; <span class="hljs-attr">template:</span>
&nbsp; &nbsp; <span class="hljs-attr">metadata:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">labels:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">app:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">version:</span> <span class="hljs-string">v2</span>
&nbsp; &nbsp; <span class="hljs-attr">spec:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">containers:</span>
&nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">name:</span> <span class="hljs-string">nginx</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">image:</span> <span class="hljs-string">nginx:1.19.2</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">ports:</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-bullet">-</span> <span class="hljs-attr">containerPort:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="156186">然后运行这些命令：</p>
<pre class="lang-shell" data-nodeid="156187"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl apply -f deploy-demo.yaml</span>
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/nginx-deployment-demo configured
</code></pre>
<p data-nodeid="156188">这个时候，我们来看看 ReplicaSet 有什么变化：</p>
<pre class="lang-shell" data-nodeid="156189"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get rs -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DESIRED&nbsp; &nbsp;CURRENT&nbsp; &nbsp;READY&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9&nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp;4m10s
nginx-deployment-demo-7594578db7&nbsp; &nbsp;1&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;1&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp;3s
</code></pre>
<p data-nodeid="156190">可以看到，这个时候自动创建了一个新的 rs 出来：<code data-backticks="1" data-nodeid="156338">nginx-deployment-demo-7594578db7</code>。一般 Deployment 的默认更新策略是 RollingUpdate，即先创建一个新的 Pod，并待其成功运行以后，再删除旧的。</p>
<p data-nodeid="156191">还有一个更新策略是Recreate，即先删除现有的 Pod，再创建新的。关于 Deployment，我们还可以设置最大不可用（<code data-backticks="1" data-nodeid="156341">maxUnavailable</code>）和最大增量（<code data-backticks="1" data-nodeid="156343">maxSurge</code>），来更精确地控制滚动更新操作，具体配置可参照<a href="https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/#%E6%BB%9A%E5%8A%A8%E6%9B%B4%E6%96%B0-deployment" data-nodeid="156347">这个文档</a>。</p>
<p data-nodeid="156192">我建议你使用默认的策略来保证可用性。后面 Deployment 控制器会不断对这两个 rs 进行调和，直到新的 rs 副本数为 3，老的 rs 副本数为 0。</p>
<p data-nodeid="156193">我们可以通过如下命令，能够观察到 Deployment 的调和过程。</p>
<pre class="lang-shell" data-nodeid="156194"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9-7w5gp&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m15s
nginx-deployment-demo-5d65f98bd9-d78fx&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m15s
nginx-deployment-demo-5d65f98bd9-ssstk&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m15s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9-7w5gp&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m22s
nginx-deployment-demo-5d65f98bd9-d78fx&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m22s
nginx-deployment-demo-5d65f98bd9-ssstk&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m22s
nginx-deployment-demo-7594578db7-zk8jq&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 15s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v2</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-7594578db7-zk8jq&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 19s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v2</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-7594578db7-4g4fk&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 1s
nginx-deployment-demo-7594578db7-zk8jq&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 31s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9-7w5gp&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m40s
nginx-deployment-demo-5d65f98bd9-d78fx&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m40s
nginx-deployment-demo-5d65f98bd9-ssstk&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m40s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v2</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-7594578db7-4g4fk&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 5s
nginx-deployment-demo-7594578db7-ftzmf&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;ContainerCreating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 2s
nginx-deployment-demo-7594578db7-zk8jq&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 35s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; &nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9-7w5gp&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m52s
nginx-deployment-demo-5d65f98bd9-ssstk&nbsp; &nbsp;0/1&nbsp; &nbsp; &nbsp;Terminating&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 4m52s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v2</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;READY&nbsp; &nbsp;STATUS&nbsp; &nbsp; RESTARTS&nbsp; &nbsp;AGE
nginx-deployment-demo-7594578db7-4g4fk&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 17s
nginx-deployment-demo-7594578db7-ftzmf&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 14s
nginx-deployment-demo-7594578db7-zk8jq&nbsp; &nbsp;1/1&nbsp; &nbsp; &nbsp;Running&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 47s
<span class="hljs-meta">$</span><span class="bash"> kubectl get rs -n demo</span>
NAME&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DESIRED&nbsp; &nbsp;CURRENT&nbsp; &nbsp;READY&nbsp; &nbsp;AGE
nginx-deployment-demo-5d65f98bd9&nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;0&nbsp; &nbsp; &nbsp; &nbsp;5m5s
nginx-deployment-demo-7594578db7&nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;3&nbsp; &nbsp; &nbsp; &nbsp;58s
<span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx,version=v1</span>
No resources found in demo namespace.
</code></pre>
<p data-nodeid="156195">或者可以通过 kubectl 的 watch 功能：</p>
<pre class="lang-shell" data-nodeid="156196"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> kubectl get pod -n demo -l app=nginx -w</span>
</code></pre>
<p data-nodeid="156197">至此，我们完成了 Deployment 的升级过程。</p>
<h3 data-nodeid="156198">写在最后</h3>
<p data-nodeid="156199">Kubernetes 中这些高阶的抽象对象，都是通过标签选择器来控制 Pod 的，包括我们下一节课要讲的有状态服务控制器。通过这些标签选择器，我们也可以通过 kubectl 命令行方便地查询一些对象。</p>
<p data-nodeid="156200">有了 Deployment 这个高级对象，我们可以很方便地完成无状态服务的发布、更新升级，无须多余的人工参与，就能保证业务的高可用性。这也是 Kubernetes 迷人之处——声明式 API。</p>
<p data-nodeid="156201">如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **伟：
> 不明白文中提到的工作负载是什么意思？所以这节看的一头雾水，老师，能解释下吗

##### **堃：
> 用yaml创建deploy的时候也会同时创建rs吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这部分的 rs 是有 kube-controller-manager 自己来创建和管理的，我们不用直接操作。

##### **6113：
> 还有，当副本扩容时，副本数+1了，那如何确保这个副本是可以正常提供服务的呢

##### **6113：
> 当副本伸缩时，具体是谁在操作容器？是如何操作的呢？

##### **锋：
> 老师好，请问除了修改镜像，还有什么配置项改动会触发滚动更新呢？

