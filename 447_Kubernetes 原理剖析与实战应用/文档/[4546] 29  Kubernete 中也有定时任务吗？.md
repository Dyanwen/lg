<p data-nodeid="502">前面我们学习了 Deployment、Statefulset、Daemonset 这些工作负载，它们可以帮助我们在不同的场景下运行<strong data-nodeid="508">长伺型</strong>（Long Running）的服务。</p>



<p data-nodeid="4">但是有一类业务（一次性作业和定时任务）运行完就结束了，不需要长期运行，如果使用上述的那些工作负载就无法满足我们的要求。比如 Pod 运行结束后，会被 Deployment、Statefulset 控制器重启或者创建新的副本替换掉，而这并不是我们期望的行为。</p>
<p data-nodeid="5">所以说，对于这类作业任务，我们需要新的工作负载类型来描述。在 Kubernetes 中，我们分别用 Job 和 Cronjob 来描述一次性任务和定时任务。</p>
<p data-nodeid="6">我们先来看看 Job。</p>
<h3 data-nodeid="7">Job</h3>
<p data-nodeid="8">我通过一个<a href="https://kubernetes.io/examples/controllers/job.yaml" data-nodeid="82">官方的例子</a>来带你了解这个工作负载类型：</p>
<pre class="lang-java" data-nodeid="9"><code data-language="java">apiVersion: batch/v1beta1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: [<span class="hljs-string">"perl"</span>,  <span class="hljs-string">"-Mbignum=bpi"</span>, <span class="hljs-string">"-wle"</span>, <span class="hljs-string">"print bpi(2000)"</span>]
      restartPolicy: Never
  backoffLimit: <span class="hljs-number">4</span>
</code></pre>
<p data-nodeid="10">这个 Job 负责计算 π 到小数点后的 2000 位，并将结果打印出来。</p>
<p data-nodeid="11">我们可以通过 kubectl create 命令将该 Job 创建出来：</p>
<pre class="lang-java" data-nodeid="12"><code data-language="java">$ kubectl create -f https:<span class="hljs-comment">//kubernetes.io/examples/controllers/job.yaml</span>
job.batch/pi created
</code></pre>
<p data-nodeid="13">创建好了以后，我们来看下这个 Job：</p>
<pre class="lang-java" data-nodeid="14"><code data-language="java">kubectl describe jobs/pi
Name:           pi
Namespace:      <span class="hljs-keyword">default</span>
Selector:       controller-uid=<span class="hljs-number">4f</span>8027d0-cac1-<span class="hljs-number">42</span>ea-b5f8-dbb4d9c9f67a
Labels:         controller-uid=<span class="hljs-number">4f</span>8027d0-cac1-<span class="hljs-number">42</span>ea-b5f8-dbb4d9c9f67a
                job-name=pi
Annotations:    &lt;none&gt;
Parallelism:    <span class="hljs-number">1</span>
Completions:    <span class="hljs-number">1</span>
Start Time:     Mon, <span class="hljs-number">02</span> Dec <span class="hljs-number">2020</span> <span class="hljs-number">15</span>:<span class="hljs-number">04</span>:<span class="hljs-number">52</span> +<span class="hljs-number">0200</span>
Completed At:   Mon, <span class="hljs-number">02</span> Dec <span class="hljs-number">2020</span> <span class="hljs-number">15</span>:<span class="hljs-number">06</span>:<span class="hljs-number">39</span> +<span class="hljs-number">0200</span>
Duration:       <span class="hljs-number">65</span>s
Pods Statuses:  <span class="hljs-number">0</span> Running / <span class="hljs-number">1</span> Succeeded / <span class="hljs-number">0</span> Failed
Pod Template:
  Labels:  controller-uid=c9948307-e56d-<span class="hljs-number">4</span>b5d-<span class="hljs-number">8302</span>-ae2d7b7da67c
           job-name=pi
  Containers:
   pi:
    ...
Events:
  Type    Reason            Age   From            Message
  ----    ------            ----  ----            -------
  Normal  SuccessfulCreate  <span class="hljs-number">4</span>m    job-controller  Created pod: pi-jk2k7
</code></pre>
<p data-nodeid="15">在这段代码中，有几点需要特别注意下。</p>
<ol data-nodeid="16">
<li data-nodeid="17">
<p data-nodeid="18">系统自动给 Job 添加了 Selector，即 controller-uid=4f8027d0-cac1-42ea-b5f8-dbb4d9c9f67a，后面的这个 uid 就是指该 Job 自己的 uid。</p>
</li>
<li data-nodeid="19">
<p data-nodeid="20">Job 上会自动被加上了 Label，即 controller-uid=4f8027d0-cac1-42ea-b5f8-dbb4d9c9f67a 和 job-name=pi。</p>
</li>
<li data-nodeid="21">
<p data-nodeid="22">Job 中 spec.podTemplate 中也被加上了 Label，即 controller-uid=4f8027d0-cac1-42ea-b5f8-dbb4d9c9f67a 和 job-name=pi。这样 Job 就可以通过这些 Label 和 Pod 关联起来，从而控制 Pod 的创建、删除等操作。</p>
</li>
</ol>
<p data-nodeid="23">我们可以通过这些 Label 来找到对应的 Pod。你可以直接使用 Job 的名字，这个最简洁最方便：</p>
<pre class="lang-java" data-nodeid="24"><code data-language="java">kubectl get pods --selector=job-name=pi
</code></pre>
<p data-nodeid="25">或者你也可以选择使用 Job 的 uid：</p>
<pre class="lang-java" data-nodeid="26"><code data-language="java">kubectl get pods --selector=controller-uid=<span class="hljs-number">4f</span>8027d0-cac1-<span class="hljs-number">42</span>ea-b5f8-dbb4d9c9f67a
</code></pre>
<p data-nodeid="27">我们可以看到，由 Job 创建出来的 Pod 已经运行结束，为 Completed 状态。</p>
<pre class="lang-java" data-nodeid="28"><code data-language="java">NAME       READY    STATUS       RESTARTS    AGE
pi-jk2k7   <span class="hljs-number">0</span>/<span class="hljs-number">1</span>      Completed    <span class="hljs-number">0</span>           <span class="hljs-number">2</span>m
</code></pre>
<p data-nodeid="29">如果说 Pod 运行过程中异常退出了，那就会根据的 Job 中 PodTemplate 定义的重启策略（restart policy）来操作。对于 Job 来说，我们当然不希望一直重启，因此这里的 restartPolicy 只能为 Never 或者 OnFailure。</p>
<p data-nodeid="30">如果说创建出来的 Pod 一直由于某些原因，导致运行不成功，怎么办呢？这个时候<strong data-nodeid="100">Job 控制器会根据 spec.backoffLimit 中定义的数值来限制 Pod 失败的次数</strong>。默认值是 6，我们在例子中设置为 4。达到这个次数以后，Job controller 便不会再新建 Pod，并直接停止运行这个 Job，将其标记为 Failure。</p>
<p data-nodeid="31">Job 还支持创建多个 Pod 并发地运行，我们来看另一个官方的例子“<a href="https://kubernetes.io/zh/docs/tasks/job/fine-parallel-processing-work-queue/" data-nodeid="104">使用工作队列进行精细的并行处理</a>”：</p>
<pre class="lang-java" data-nodeid="32"><code data-language="java">apiVersion: batch/v1
kind: Job
metadata:
  name: job-wq-<span class="hljs-number">2</span>
spec:
  parallelism: <span class="hljs-number">2</span>
  template:
    metadata:
      name: job-wq-<span class="hljs-number">2</span>
    spec:
      containers:
      - name: c
        image: gcr.io/myproject/job-wq-<span class="hljs-number">2</span>
      restartPolicy: OnFailure
</code></pre>
<p data-nodeid="33">在 Job 的定义中通过 spec.parallelism 字段，我们可以指定并发运行的 Pod 数目。我们这里指定为 2，也就是会创建 2 个同时运行的 Pod。</p>
<p data-nodeid="34">例子中的 2 个 Pod 都会不停地从队列中获取数据进行处理，直到队列为空后退出，运行结束。</p>
<p data-nodeid="35">Job 还支持其他的工作模式，比如通过模板渲染 Job 来支持批量任务的处理等等。你可以参照<a href="https://kubernetes.io/zh/docs/concepts/workloads/controllers/job/#job-patterns" data-nodeid="111">官方文档</a>来解锁 Job 更多地使用场景，并动手学习实践。</p>
<p data-nodeid="36">通常来说，Job 运行结束，即状态为 Completed 或 Failure 时，我们并不需要在系统中继续保留该对象，尤其是这种对象较多的时候，会给 kube-apiserver 的 cache 以及系统访问带来很大的压力。</p>
<p data-nodeid="37">这个时候我们就可以使用<a href="https://kubernetes.io/zh/docs/concepts/workloads/controllers/ttlafterfinished/" data-nodeid="117">TTL 控制器</a>提供 的 TTL 能力了。</p>
<p data-nodeid="38">我们只需要在 Job  的&nbsp;spec.ttlSecondsAfterFinished&nbsp;字段设置一下，就可以让该控制器帮我们自动清理掉已经结束的资源，包括 Job 本身及其关联的 Pod 对象。</p>
<p data-nodeid="39">我们来看下面这个例子：</p>
<pre class="lang-java" data-nodeid="40"><code data-language="java">apiVersion: batch/v1
kind: Job
metadata:
  name: pi-with-ttl
spec:
  ttlSecondsAfterFinished: <span class="hljs-number">100</span>
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: [<span class="hljs-string">"perl"</span>,  <span class="hljs-string">"-Mbignum=bpi"</span>, <span class="hljs-string">"-wle"</span>, <span class="hljs-string">"print bpi(2000)"</span>]
      restartPolicy: Never
</code></pre>
<p data-nodeid="41">该 Job&nbsp;在运行结束 100 秒之后就被自动清理删除了，包括创建出来的 Pod。</p>
<p data-nodeid="42">目前这种 TTL 的能力还处于 Alpha 阶段，如果你要使用的话，需要手动开启 TTLAfterFinished&nbsp;这个 feature gate，具体可以参考&nbsp;TTL 控制器的<a href="https://kubernetes.io/zh/docs/concepts/workloads/controllers/ttlafterfinished/" data-nodeid="125">文档</a>学习如何打开和使用这个功能。</p>
<p data-nodeid="43">我们再来看看 CronJob。</p>
<h3 data-nodeid="44">CronJob</h3>
<p data-nodeid="45">从名字就可以看出来，这个工作负载是用于定时任务的，比如每隔 1 分钟执行 1 次任务。</p>
<p data-nodeid="46">我们来看一个官方的 CronJob 的例子，这个例子比较简单明了：</p>
<pre class="lang-java" data-nodeid="47"><code data-language="java">apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: hello # cronjob 名字
spec:
  schedule: "*/1 * * * *" # job执行的周期，通过 cron 格式来标明
  jobTemplate: # job模板
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            args:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
</code></pre>
<p data-nodeid="48">CronJob 通过 spec.schedule 字段来标明 Job 被创建和执行的周期，该字段段用<a href="https://en.wikipedia.org/wiki/Cron" data-nodeid="134">Cron</a>格式编写。Cron 的基本格式为：</p>
<pre class="lang-java" data-nodeid="49"><code data-language="java">&lt;分钟&gt; &lt;小时&gt; &lt;日&gt; &lt;月&gt; &lt;星期&gt;
</code></pre>
<p data-nodeid="50">其中分钟的值从 0 到 59，小时的值从 0 到 23，日的值从 1 到 31，月的值从 1 到 12，星期的值从 0 到 6，0 表示星期日。</p>
<p data-nodeid="51" class="te-preview-highlight">Cron 还支持“*<strong data-nodeid="144">,-/</strong>”等字符，其中 * 是个通配符，可以匹配任何值；/ 则表示起始时间触发，然后每隔一个固定时间触发一次。例如我们如果在分钟中设置 10/20，则表示第一次触发在第 10 分钟，接下来每隔 20 分钟触发一次，也就是第 30 分钟、第 50 分钟等依次往后的时间点触发一次。</p>
<p data-nodeid="52">所以我们例子中的"*/1 * * * *"表示每隔一分钟触发一次新 Job 的执行。</p>
<p data-nodeid="53">例子中的 spec.jobTemplate 指定了 Job 的模板，CronJob 控制器会根据 Cron 设置的时间触发新的 Job 创建。因此，我们修改 CronJob 的 spec，只会影响新 Job 的spec 配置，对于已经创建的 Job spec 不会有任何影响。</p>
<p data-nodeid="54">CronJob 会帮助我们管理 Job，比如自动清理运行完的 Job；也就是说，由 CronJob 管理的 Job，我们不需要去配置上文提到的 TTL 自动清理，CronJob 控制器会自动帮我们清理。CronJob 通过 spec.successfulJobsHistoryLimit 和 spec.failedJobsHistoryLimit 来限制保留已完成的 Job 数量，确保不会有大量的 Job 残留在系统中。默认值分别为 3 和 1。</p>
<p data-nodeid="55">当然在 CronJob 被触发，创建新的 Job 的时候，还会出现一种情形：上一次触发的 Job 还未执行完成。如果这个时候触发了另一个新 Job 的创建，势必会导致任务重叠。此时就需要你结合自己的业务来考虑这种行为对业务的影响了。</p>
<p data-nodeid="56">你可以在 spec.concurrentPolicy 中配置：</p>
<ul data-nodeid="57">
<li data-nodeid="58">
<p data-nodeid="59">设置为 Allow，这也是默认的值，允许并发任务的执行；</p>
</li>
<li data-nodeid="60">
<p data-nodeid="61">设置为 Forbid，不允许并发任务执行；</p>
</li>
<li data-nodeid="62">
<p data-nodeid="63">设置为 Replace，用新的 Job 来替换当前正在运行的老的 Job。</p>
</li>
</ul>
<h3 data-nodeid="64">写在最后</h3>
<p data-nodeid="65">这一讲，我带你了解了如何在 Kubernetes 中设置定时任务。所有 Cronjob 中的 schedule 字段中的时间都是基于 kube-controller-manager 的时区。我们在搭建环境的时候，最好将各个节点上的时间进行同步，这样可以避免很多奇奇怪怪的问题。</p>
<p data-nodeid="66">如果你对本节课有什么想法或者疑问，欢迎你在留言区留言，我们一起讨论。</p>

---

### 精选评论

##### **5661：
> 老师，k8s弃用docker，操作层面会有哪些影响，应该不会影响部署和拉起docker制作的镜像吧

