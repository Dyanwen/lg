<p style="line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;">本课时我将带你学习 Kubernetes 的相关内容。</span><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 是一个可移植和可扩展的开源平台，用来管理容器化的工作负载和服务，它可以促进声明式的配置和自动化。Kubernetes 是 Google 基于自身运行大规模工作负载的丰富经验开发出来的，2014 年 Google 开源了 Kubernetes 项目，集合社区的智慧来让 Kubernetes 变得更好。Kubernetes 是<strong>目前</strong><strong>容器编排领域</strong><strong>上的</strong><strong>事实标准</strong>。绝大多数云平台都提供了 Kubernetes 的支持，同时，它也是本专栏中云原生应用的部署平台。本课时我将对 Kubernetes 及其重要概念进行介绍。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">为了进行本课时的介绍，你需要有一个可用的Kubernetes集群。在本地开发环境中，可以使用Docker Desktop或是Minikube，也可以使用云平台上安装好的集群。前提要求是可以使用kubectl来访问集群。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 集群</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">部署 Kubernetes 之后将运行一个 Kubernetes 集群。Kubernetes 集群中包含很多不同的组件，分别运行在不同的节点（Node）上，其中，组件分成两类：控制平面（Control Plane）组件和节点组件，而节点可以是物理机器或虚拟主机。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">控制平面组件分类中的组件如下表所示：</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span></span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><img src="https://s0.lgstatic.com/i/image3/M01/04/39/Ciqah1595s2ANX-fAABk4rHlqxA992.png"></span></p>
<p></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br>节点组件如下表所示：</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span></span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><img src="https://s0.lgstatic.com/i/image3/M01/04/39/Ciqah1595vqAHKWMAAA0p9KyiHI337.png"></span></p>
<p></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br>除了这些核心组件之外，还可以安装一些附加组件，如 DNS 服务器、Web 界面、容器资源监控和日志管理等。</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下图给出了 Kubernetes 中各个组件之间的交互关系。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/04/21/Ciqah159xA6AU1_bAAC_2Bqc9uM667.png"></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在实际的集群中，控制平面组件和工作节点组件通常部署在不同的机器上，包含控制平面组件的机器称为<strong>集群的主控节点</strong>（Master），包含节点组件的机器称为<strong>工作节点</strong>（Worker）。生产环境的集群中应该至少有 1 个主控节点和 3 个工作节点。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">API 与对象</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 的 API 采用与 Kubernetes 控制平面组件交互的方式，来管理 Kubernetes 集群。当需要与API 交互时，我们可以使用自带的 kubectl 工具，或使用不同编程语言的客户端库。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 的 API 中定义了很多不同类型的对象，这些对象的使用是声明式的，并且只声明了对象所期望的状态。每个对象中都包含 2 个字段，分别是 spec 和 status，其中 spec 描述了对象的期望状态，而 status 包含的是对象当前的状态。Kubernetes 会确保对象的最终状态与期望的状态保持一致。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在对象的描述中，每个对象都有名称和唯一的标识符，其中有一些字段是通用的。对象的名称由创建者提供，同一类对象的名称不能重复。对象的标识符由 Kubernetes 自动生成，其标签是添加到对象上的名值对，标签主要用来选择满足条件的对象。注解同样以名值对的形式出现，只不过注解的作用是添加附加的信息。从作用上来说，注解的作用类似于 Java 中的注解，很多 Kubernetes 上的框架和第三方库都允许使用自定义的注解来进行配置。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本课时中对相关对象的描述都使用 YAML 格式。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Pod</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 中的 Pod 表示由一个或多个容器组成的分组。Pod 里面的容器共享一些资源，包括存储、网络和每个容器的运行配置。我们可以把 Pod 看成是一个应用运行的逻辑主机，相当于一个物理机器或虚拟主机，其中的容器是紧密耦合的。每个 Pod 都有唯一的 IP 地址，其中的容器共享同一个 IP 地址和端口范围，相互之间可以通过 localhost 访问。Pod 还可以使它的容器访问其定义的共享的存储卷（volume），通过这种方式，可以实现 Pod 容器之间的数据共享。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Pod 是 Kubernetes 中调度和运行的基本单元，但在 Kubernetes 上并不能直接运行容器，而是要把容器放在 Pod 中运行，Pod 再把其中的全部容器调度到同一个集群节点上运行。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">控制器</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">作为基本的运行单元，Pod 缺少相应的管理功能。如果一个 Pod 在运行中出现错误，并不会自动创建新的 Pod 来替代它；如果需要管理 Pod，则应该利用 Kubernetes 提供不同类别的控制器，包括副本集、部署、有状态集和守护程序集。</span></p>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">副本集</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">副本集（ReplicaSet）保证在任何时候都有给定数量的 Pod 副本。当创建副本集时需要指定一个创建 Pod 的模板，同时确定 Pod 的选择器，并给出期望的 Pod 副本数量。在运行时，副本集根据指定的 Pod 选择器来监控当前 Pod 副本的数量，如果副本数量小于期望值，则根据 Pod 模板来创建新的 Pod。一般用标签作为 Pod 的选择器。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件描述了包含 3 个 Pod 副本的副本集，每个 Pod 中只有一个 Nginx 容器，每个 Pod 上都添加了名称为 app、值为 nginx 的标签。副本集使用这个标签作为包含的 Pod 的选择器，在 spec.selector.matchLabels 中指定。spec.containers 指定的是创建 Pod 的模板。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;apps/v1
kind:&nbsp;ReplicaSet
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
spec:
&nbsp;&nbsp;replicas:&nbsp;3
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;nginx:1.17
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;containerPort:&nbsp;80</pre>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">部署</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">部署（Deployment）在副本集上提供了对 Pod 的更新功能，每个部署都有对应的副本集。当部署中创建的 Pod 模板发生变化时，这些 Pod 都需要被更新。在更新时，Kubernetes会创建一个新的副本集来管理新的 Pod，当更新完成后，旧的副本集会被删除。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">部署可以采用不同的策略来更新这些 Pod，最常用的策略是<strong>滚动更新</strong>，它可以保证在更新过程中服务不间断。在进行滚动更新时，Kubernetes首先会创建新的副本集中的 Pod，再删除已有副本集中的 Pod。这个过程是交替进行的：在没有足够数量的新 Pod 运行之前，不会删除已有的 Pod；同样的，在没有足够数量的已有 Pod 被删除之前，不会创建新的 Pod。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件创建了包含 3 个 Pod 副本的部署，它与上一小节中的副本集 YAML 的文件的区别在于，kind 的值是 Deployment。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;apps/v1
kind:&nbsp;Deployment
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
spec:
&nbsp;&nbsp;replicas:&nbsp;3
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;nginx:1.17
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;containerPort:&nbsp;80</pre>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">有状态集</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">副本集中的 Pod 没有标识符，不能进行区分，它唯一保证的是所包含的 Pod 总数。有些应用对运行的实例有启动顺序和唯一性的要求，典型的例子是通过多个实例组成集群的服务，如 Cassandra 和 RabbitMQ 等。这些应用要求实例有固定且唯一的网络标识，才能正确的建立集群，对于这样的应用，应该使用有状态集（StatefulSet）来管理。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">有状态集中的 Pod 都有唯一不变的标识符，这个标识符由有状态集的名称和 Pod 的序号组成。如果有状态集的名称是 cassandra，同时期望的 Pod 副本数量是 3，那么 Pod 的名称分别是 cassandra-0、cassandra-1 和 cassandra-2。有状态集需要一个对应的服务来提供 Pod 的 DNS 名称。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">有状态集在创建和扩展时有特殊的限制，如果一个有状态集期望的 Pod 副本数量是 N，那么有状态集会从 0 开始依次创建这些 Pod，在第 1 个 Pod 正常运行之前，不会创建第 2 个Pod；在删除 Pod 时，则是从第 N 个 Pod 开始反向依次删除。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件创建了一个有状态集及其关联的服务。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;v1
kind:&nbsp;Service
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
spec:
&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;port:&nbsp;80
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;web
&nbsp;&nbsp;clusterIP:&nbsp;None
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
---
apiVersion:&nbsp;apps/v1
kind:&nbsp;StatefulSet
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
spec:
&nbsp;&nbsp;replicas:&nbsp;3
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;serviceName:&nbsp;nginx
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;nginx:1.17
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;containerPort:&nbsp;80
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;web</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在下面的 Pod 列表中你可以看到 Pod 的名称。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>NAME&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;READY&nbsp;&nbsp;&nbsp;STATUS&nbsp;&nbsp;&nbsp;&nbsp;RESTARTS&nbsp;&nbsp;&nbsp;AGE
nginx-0&nbsp;&nbsp;&nbsp;1/1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Running&nbsp;&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;35m
nginx-1&nbsp;&nbsp;&nbsp;1/1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Running&nbsp;&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;35m
nginx-2&nbsp;&nbsp;&nbsp;1/1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Running&nbsp;&nbsp;&nbsp;0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;35m</pre>
<h3><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">守护程序集</span></p></h3>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">守护程序集（DaemonSet）确保在全部或部分的集群节点上运行 Pod，每个节点上最多运行一个 Pod 副本。如果应用需要执行的任务与节点相关，则应该使用守护程序集，比如收集日志和节点性能指标数据的应用，都应该使用守护程序集来部署。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件创建了一个守护程序集。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;apps/v1
kind:&nbsp;DaemonSet
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
spec:
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;nginx:1.17
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;containerPort:&nbsp;80</pre>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">服务</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">应用在运行时经常需要为其他应用提供服务。在使用控制器管理应用的 Pod 时，Pod 的 IP 地址会随着 Pod 的创建和删除而发生变化。因此，我们不能使用 Pod 的 IP 地址作为访问服务的方式。Kubernetes 中的服务（Service）作为一组 Pod 的抽象，定义了如何访问这些 Pod，</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">它实现了服务发现和负载均衡两项重要的功能。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在创建服务时需通过选择器来选择服务所对应的 Pod，应用服务的消费者使用服务的地址来访问 Pod，实际的访问请求会被分发到服务所对应的某个 Pod 上。当 Pod 发生变化时，服务会自动更新所对应的 Pod 列表。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下表给出了服务的几种类型。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span></span></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span><img src="https://s0.lgstatic.com/i/image3/M01/04/39/Ciqah1595zWAa7SbAABbO7aQF0w788.png"></span></p>
<p></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br>下面代码中的 YAML 描述了一个类型为 NodePort 的服务。</span><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;v1
kind:&nbsp;Service
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
spec:
&nbsp;&nbsp;type:&nbsp;NodePort
&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;port:&nbsp;8080
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;targetPort:&nbsp;80
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protocol:&nbsp;TCP
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;http
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在服务创建之后，服务的使用者可以有两种方式来发现服务。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第一种方式是使用 Pod 的环境变量。在 Pod 运行时，每个服务的主机名和端口都会被自动添加到环境变量中，对应的环境变量名称是在服务名称之后添加 _SERVICE_HOST 和 _SERVICE_PORT 后缀，并且把服务名称转换成大写，把破折号替换成下划线。比如，如果服务名称是 cassandra，那么所对应的环境变量名称分别是 CASSANDRA_SERVICE_HOST 和 CASSANDRA_SERVICE_PORT。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">第二种方式是使用 DNS 名称。如果服务的访问者和服务的 Pod 在同一个名称空间中，则直接使用服务的名称进行访问即可；如果在不同的名称空间，则需要使用服务“名称.名称空间”的形式，如 cassandra.demo。</span></p>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">存储卷</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">容器运行时对内部文件所做的修改是瞬时的，当容器停止之后，相关的修改会丢失，如果应用具备第 03 课时中云原生应用的无状态特征，这并不是一个问题，反而是正确的做法。但是应用所依赖的支撑服务，绝大部分是有状态的，比如数据库和消息中间件等。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 中的<strong>卷</strong>（Volume）是存储的抽象表示，解决了数据的持久化问题，卷与使用它的 Pod 具有相同的生命周期。Pod 中的容器可以使用卷共享数据，对于运行在 Pod 中容器里面的应用来说，卷就是文件系统上的一个目录，其中包含了可以访问的文件。Kubernetes 支持不同类型的卷，包括云平台提供的实现和第三方存储服务。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在实际的 Kubernetes 集群中，数据存储通常需要进行统一管理，有专门的服务负责提供存储，而 Pod 是存储的消费者。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes 提供了两个对象，即持久卷和持久卷要求，<strong>持久卷</strong>可以由管理员手动创建，或者由存储服务动态创建，用户则以持久卷要求的形式来声明所需要使用的持久卷的大小和访问模式；<strong>持久卷要求</strong>可以与手动创建的持久卷进行绑定，如果需要动态创建，则由存储服务完成创建之后，再进行绑定。一旦完成绑定，持久卷可以用类似卷的方式来访问。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件描述了一个容量为 1G 的持久卷，使用的存储类是 standard。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;v1
kind:&nbsp;PersistentVolume
metadata:
&nbsp;&nbsp;name:&nbsp;pv
spec:
&nbsp;&nbsp;storageClassName:&nbsp;standard
&nbsp;&nbsp;capacity:
&nbsp;&nbsp;&nbsp;&nbsp;storage:&nbsp;1Gi
&nbsp;&nbsp;accessModes:
&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;ReadWriteOnce
&nbsp;&nbsp;hostPath:
&nbsp;&nbsp;&nbsp;&nbsp;path:&nbsp;"/mnt/data"</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件描述了一个持久卷要求，所请求的空间大小是 1G，刚好可以由上面 YAML 文件中的持久卷来满足。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;v1
kind:&nbsp;PersistentVolumeClaim
metadata:
&nbsp;&nbsp;name:&nbsp;pvc
spec:
&nbsp;&nbsp;storageClassName:&nbsp;standard
&nbsp;&nbsp;accessModes:
&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;ReadWriteOnce
&nbsp;&nbsp;resources:
&nbsp;&nbsp;&nbsp;&nbsp;requests:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;storage:&nbsp;1Gi</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果持久卷是动态创建的，则需要使用存储类来描述不同的存储方式，存储类通常由存储服务提供。在持久卷要求中，声明所需要使用的存储类后，它对应的存储服务会负责创建持久卷。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码的 YAML 文件中，部署中 Pod 里的 volumes 声明中的 persistentVolumeClaim 引用的持久卷要求的名称是pvc，与上面的YAML文件相对应。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;apps/v1
kind:&nbsp;Deployment
metadata:
&nbsp;&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
spec:
&nbsp;&nbsp;replicas:&nbsp;1
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volumes:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;data
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;persistentVolumeClaim:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;claimName:&nbsp;pvc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;nginx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;nginx:1.17
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ports:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;containerPort:&nbsp;80
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volumeMounts:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;mountPath:&nbsp;"/usr/share/nginx/html"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;data</pre>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">任务和定时任务</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">当需要执行一次性任务时，我们可以使用任务对象在创建任务时指定需要运行的 Pod 模板。当需要定期执行任务时，则可以使用定时任务对象，在创建时，需要指定定时任务的 cron 表达式。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件描述了执行一次的任务。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;batch/v1
kind:&nbsp;Job
metadata:
&nbsp;&nbsp;name:&nbsp;job-hostname
spec:
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;busybox
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;busybox
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;command:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;hostname
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;restartPolicy:&nbsp;Never
&nbsp;&nbsp;backoffLimit:&nbsp;1</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件描述了每分钟执行一次的定期任务，定期任务在每次执行时都会创建新的 Job 对象。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;batch/v1beta1
kind:&nbsp;CronJob
metadata:
&nbsp;&nbsp;name:&nbsp;cronjob-date
spec:
&nbsp;&nbsp;schedule:&nbsp;"*&nbsp;*&nbsp;*&nbsp;*&nbsp;*"
&nbsp;&nbsp;jobTemplate:
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;busybox
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;busybox
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;command:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;date
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;restartPolicy:&nbsp;OnFailure</pre>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">配置表</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在第 03 课时介绍云原生应用的时候，我们提到了配置外部化，当它运行在 Kubernetes 上时，应该使用配置表（ConfigMap）来管理配置。配置表可以看成是包含名值对的哈希表。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Spring Boot 应用使用 application.properties 文件来管理配置。下面代码中的 ConfigMap 对象，仅包含一个名为 application.properties 的配置项，其对应值是 application.properties 文件的实际内容。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;v1
kind:&nbsp;ConfigMap
metadata:
&nbsp;&nbsp;name:&nbsp;app-config
data:
&nbsp;&nbsp;application.properties:&nbsp;|-
&nbsp;&nbsp;&nbsp;&nbsp;server.port&nbsp;=&nbsp;9090
&nbsp;&nbsp;&nbsp;&nbsp;spring.application.name&nbsp;=&nbsp;myapp</pre>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码中的 YAML 文件创建了 Spring Boot 应用的部署。在 Pod 模板中，我们从配置表中创建了一个存储卷，配置表中的每个名值对都会被转换成一个文件：文件名是名称，而文件的内容是相应的值。应用容器把该存储卷映射到 /etc/config 目录上，通过环境变量 SPRING_CONFIG_LOCATION 可以让 Spring Boot从/etc/config 目录中加载配置文件，从而应用 application.properties 文件的配置。</span></p>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>apiVersion:&nbsp;apps/v1
kind:&nbsp;Deployment
metadata:
&nbsp;&nbsp;name:&nbsp;demo-app
&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;demo-app
spec:
&nbsp;&nbsp;replicas:&nbsp;1
&nbsp;&nbsp;selector:
&nbsp;&nbsp;&nbsp;&nbsp;matchLabels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;demo-app
&nbsp;&nbsp;template:
&nbsp;&nbsp;&nbsp;&nbsp;metadata:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;labels:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app:&nbsp;demo-app
&nbsp;&nbsp;&nbsp;&nbsp;spec:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;containers:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;app
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;image:&nbsp;localhost:5000/demo-app
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;env:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;SPRING_CONFIG_LOCATION
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value:&nbsp;/etc/config/
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volumeMounts:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;config-volume
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mountPath:&nbsp;/etc/config
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;volumes:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;config-volume
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;configMap:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;app-config</pre>
<h2><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总结</span></p></h2>
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Kubernetes是本专栏所介绍的云原生微服务应用的运行平台。本课时对Kubernetes中的基本概念进行了介绍，包括Kubernetes集群、Pod、副本集、部署、有状态集、守护程序集等，还介绍了服务、存储卷、任务和定时任务、以及配置表的相关内容。通过本课时，可以让你对Kubernetes有基本的了解。</span></p>

---

### 精选评论


