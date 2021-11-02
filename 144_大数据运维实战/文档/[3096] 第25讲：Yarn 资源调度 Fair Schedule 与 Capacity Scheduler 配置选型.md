<p data-nodeid="101763" class="">在大数据平台运维中，会经常遇到<strong data-nodeid="101769">集群资源争抢的问题</strong>。因为在公司内部，Hadoop Yarn 集群一般会被多个业务、多个用户同时使用，共享 Yarn 资源。此时，如果不对集群资源做规划和管理的话，那么就会出现 Yarn 的资源被某一个用户提交的 Application（App）占满，而其他用户只能等待；或者也可能会出现集群还有很多剩余资源，但 App 就是无法使用的情况。</p>





<p data-nodeid="99779">如何解决这个问题呢？此时就需要用到 Hadoop 中提供的<strong data-nodeid="99892">资源调度器。</strong></p>
<h3 data-nodeid="104021" class="">Yarn 多用户资源管理策略</h3>




<p data-nodeid="99781">Yarn 提供了可插拔的资源调度算法，用于解决 App 之间资源竞争的问题。在 Yarn 中有三种资源调度器可供选择，即 FIFO Scheduler、Capacity Scheduler、Fair Scheduler，目前使用比较多的是 Fair Scheduler 和 Capacity Scheduler。下面对这三种资源调度器分别进行介绍。</p>
<h4 data-nodeid="107359" class="">1. FIFO Scheduler</h4>






<p data-nodeid="99783">在 Hadoop 1.x 系列版本中，默认使用的调度器是 FIFO，它采用队列方式将每个任务按照时间先后顺序进行服务。比如排在最前面的任务需要若干 Map Task 和 Reduce Task，当发现有空闲的服务器节点时就分配给这个任务，直到任务执行完毕。</p>
<h4 data-nodeid="110069" class="">2. Capacity Scheduler</h4>





<p data-nodeid="99785">在 Hadoop 2.x/3.x 系列版本中，默认使用的调度器是 Capacity Scheduler（容量调度器），这是一种<strong data-nodeid="99922">多用户、多队列</strong>的资源调度器。<strong data-nodeid="99923">每个队列</strong>可以配置资源量，可限制每个用户、每个队列的并发运行作业量，也可限制每个作业使用的内存量；每个用户的作业有优先级，在单个队列中，作业按照先来先服务（实际上是先按照优先级，优先级相同的再按照作业提交时间）的原则进行调度。</p>
<p data-nodeid="99786">容量资源调度器，支持多队列，<strong data-nodeid="99929">但默认情况下只有 root.default 这一个队列</strong>。</p>
<p data-nodeid="99787">当不同用户提交任务时，任务都会在这个队列里按照<strong data-nodeid="99935">先进先出</strong>策略执行调度，很明显，单个队列会大大降低多用户的资源使用率。</p>
<p data-nodeid="99788">因此，要使用容量资源调度，一定要配置多个队列，每个队列可配置一定比率的资源量（CPU、内存）；同时为了防止同一个用户的任务独占队列的所有资源，调度器会对同一个用户提交的任务所占资源量进行限定。</p>
<p data-nodeid="111127">举个简单的例子，下图是容量调度器中配置好的一个队列树：</p>
<p data-nodeid="111128" class=""><img src="https://s0.lgstatic.com/i/image/M00/35/54/Ciqc1F8VSH2APIBaAAAlubO9K7M490.png" alt="Drawing 0.png" data-nodeid="111132"></p>


<p data-nodeid="99791">上图通过队列树方式对 Yarn 集群资源做了一个划分，可以看到，在 root 队列下面定义了两个子队列 dev 和 test，分别占 30% 和 70% 的 Yarn 集群资源；而 dev 队列又被分成了 dev1 和 dev2 两个子队列，分别占用 dev 队列 30% 中的 40% 和 60% 的 Yarn 集群资源。</p>
<p data-nodeid="99792">容量调度除了可以配置队列及其容量外，还可以配置一个用户或任务可以分配的最大资源数量、同时可以配置运行应用的数量、队列的 ACL 认证等。</p>
<p data-nodeid="111665" class=""><strong data-nodeid="111670">如何让任务运行在指定的队列呢？</strong> 有两种方式，一种是直接指定队列名，另一种是通过用户名、用户组和队列名进行对应。注意：对于容量调度器，我们的队列名必须是队列树中的最后一部分，如果使用队列树则不会被识别。例如，在上面配置中，可直接使用 dev1 和 dev2 作为队列名，但如果用 root.dev.dev1 或者 dev.dev2 则都是无效的。</p>

<h4 data-nodeid="114309" class="">3. Fair Scheduler</h4>





<p data-nodeid="99795">Fair Scheduler（公平调度器）支持<strong data-nodeid="99961">多用户、多分组</strong>管理，每个分组可以配置资源量，也可限制每个用户和每个分组中并发运行的作业数量；每个用户的作业有优先级，优先级越高分配的资源就越多。公平调度器的主要目标是实现 Yarn 上运行的任务能公平的分配到资源。</p>
<p data-nodeid="99796">Fair Scheduler 将整个 Yarn 的可用资源划分成多个队列资源池，每个队列中可以配置最小和最大的可用资源（内存和 CPU）、最大可同时运行 Application 数量、权重，以及可以提交和管理 Application 的用户等。</p>
<p data-nodeid="115339">资源池以及用户的对应关系如下图所示：</p>
<p data-nodeid="115340" class=""><img src="https://s0.lgstatic.com/i/image/M00/35/5F/CgqCHl8VSJmAU1DfAADOTE-gHkM880.png" alt="1.png" data-nodeid="115344"></p>


<p data-nodeid="99799">在上图中，假设整个 Yarn 集群可用的 CPU 资源为 100vCPU，可用的内存资源为 100GB。现在为三个业务线各自划分一个队列，分别是 Queue1、Queue2 和 Queue3，每个队列可用的资源均为 20vCPU 和 20GB 内存，最后还规划了一个 default 队列，用于运行其他用户和业务提交的任务。可用资源为 40vCPU 和 40GB 内存，这样，四个队列将整个 Yarn 集群资源刚好分配完毕。</p>
<p data-nodeid="99800">在执行任务的时候，可以显性地指定任务运行的队列，但更多情况下不指定队列，而是通过用户名作为队列名称来提交任务，即用户 user1 提交的任务被分配到队列 Queue1 中，用户 user2 提交的任务被分配到资源池 Queue2 中。注意，这里的 user1 和 user2 是配置的固定用户，除了这些用户外，其他未指定的用户提交的任务将会被分配到 default 队列中。这里的用户名，就是提交 App 所使用的 Linux/Unix 的系统用户名。</p>
<p data-nodeid="99801">除了可以通过用户名作为队列名，在用户比较多的时候，还可以使用用户组，将同一类用户放到一个用户组下，然后将这个用户组配置到资源调度策略中。</p>
<p data-nodeid="99802">接下来，向你介绍 Fair Scheduler 调度的配置。</p>
<h3 data-nodeid="117420" class="">Fair Scheduler 调度的配置</h3>




<p data-nodeid="99804">要启用公平调度器，首先需要配置 yarn-site.xml 文件，添加如下设置：</p>
<pre class="lang-js" data-nodeid="118704"><code data-language="js">&lt;property&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.resourcemanager.scheduler.class<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span></span>
&lt;/property&gt;
</code></pre>



<p data-nodeid="99806">公平调度器的配置文件路径位于 HADOOP_CONF_DIR下 的 fair-scheduler.xml 文件中，这个路径可以通过配置 yarn-site.xml 文件，添加如下内容来实现：</p>
<pre class="lang-js" data-nodeid="119730"><code data-language="js">    &lt;property&gt;
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.scheduler.fair.allocation.file<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span></span>
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>/etc/hadoop/conf/fair-scheduler.xml<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span></span>
    &lt;/property&gt;
</code></pre>


<p data-nodeid="99808">若没有这个配置文件，调度器会在用户提交第一个应用时为其自动创建一个队列，队列的名字就是用户名，所有的任务都会被分配到 default 队列中。</p>
<p data-nodeid="99809">接下来重点看看 fair-scheduler.xml 文件如何编写，此文件中定义队列的层次是通过嵌套元素实现的。所有的队列都是 root 队列的孩子，下面是一个定义好的公平调度策略：</p>
<pre class="lang-js" data-nodeid="120756"><code data-language="js">&lt;?xml version=<span class="hljs-string">"1.0"</span>?&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">allocations</span>&gt;</span>  
        <span class="hljs-comment">&lt;!-- users max running apps --&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">userMaxAppsDefault</span>&gt;</span>10<span class="hljs-tag">&lt;/<span class="hljs-name">userMaxAppsDefault</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">queue</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"root"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">aclSubmitApps</span>&gt;</span> <span class="hljs-tag">&lt;/<span class="hljs-name">aclSubmitApps</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">aclAdministerApps</span>&gt;</span> <span class="hljs-tag">&lt;/<span class="hljs-name">aclAdministerApps</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">queue</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"default"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">minResources</span>&gt;</span>12000mb,5vcores<span class="hljs-tag">&lt;/<span class="hljs-name">minResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxResources</span>&gt;</span>100000mb,50vcores<span class="hljs-tag">&lt;/<span class="hljs-name">maxResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxRunningApps</span>&gt;</span>22<span class="hljs-tag">&lt;/<span class="hljs-name">maxRunningApps</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">schedulingMode</span>&gt;</span>fair<span class="hljs-tag">&lt;/<span class="hljs-name">schedulingMode</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">weight</span>&gt;</span>1<span class="hljs-tag">&lt;/<span class="hljs-name">weight</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">aclSubmitApps</span>&gt;</span>*<span class="hljs-tag">&lt;/<span class="hljs-name">aclSubmitApps</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">queue</span>&gt;</span>
       
        <span class="hljs-tag">&lt;<span class="hljs-name">queue</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"dev_group"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">minResources</span>&gt;</span>115000mb,50vcores<span class="hljs-tag">&lt;/<span class="hljs-name">minResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxResources</span>&gt;</span>500000mb,150vcores<span class="hljs-tag">&lt;/<span class="hljs-name">maxResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxRunningApps</span>&gt;</span>181<span class="hljs-tag">&lt;/<span class="hljs-name">maxRunningApps</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">schedulingMode</span>&gt;</span>fair<span class="hljs-tag">&lt;/<span class="hljs-name">schedulingMode</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">weight</span>&gt;</span>5<span class="hljs-tag">&lt;/<span class="hljs-name">weight</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">aclSubmitApps</span>&gt;</span> dev_group<span class="hljs-tag">&lt;/<span class="hljs-name">aclSubmitApps</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">aclAdministerApps</span>&gt;</span>hadoop dev_group<span class="hljs-tag">&lt;/<span class="hljs-name">aclAdministerApps</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">queue</span>&gt;</span>
                                                                                                         
                                          
        <span class="hljs-tag">&lt;<span class="hljs-name">queue</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"test_group"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">minResources</span>&gt;</span>23000mb,10vcores<span class="hljs-tag">&lt;/<span class="hljs-name">minResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxResources</span>&gt;</span>300000mb,100vcores<span class="hljs-tag">&lt;/<span class="hljs-name">maxResources</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">maxRunningApps</span>&gt;</span>22<span class="hljs-tag">&lt;/<span class="hljs-name">maxRunningApps</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">schedulingMode</span>&gt;</span>fair<span class="hljs-tag">&lt;/<span class="hljs-name">schedulingMode</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">weight</span>&gt;</span>4<span class="hljs-tag">&lt;/<span class="hljs-name">weight</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">aclSubmitApps</span>&gt;</span> test_group<span class="hljs-tag">&lt;/<span class="hljs-name">aclSubmitApps</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">aclAdministerApps</span>&gt;</span>hadoop test_group<span class="hljs-tag">&lt;/<span class="hljs-name">aclAdministerApps</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">queue</span>&gt;</span>
                                                      
<span class="hljs-tag">&lt;/<span class="hljs-name">queue</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">queuePlacementPolicy</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">rule</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"user"</span> <span class="hljs-attr">create</span>=<span class="hljs-string">"false"</span> /&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">rule</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"primaryGroup"</span> <span class="hljs-attr">create</span>=<span class="hljs-string">"false"</span> /&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">rule</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"secondaryGroupExistingQueue"</span> <span class="hljs-attr">create</span>=<span class="hljs-string">"false"</span> /&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">rule</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"default"</span> <span class="hljs-attr">queue</span>=<span class="hljs-string">"default"</span> /&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">queuePlacementPolicy</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">allocations</span>&gt;</span></span>
</code></pre>


<p data-nodeid="99811">下面介绍这个配置中的几个配置项的含义：</p>
<table data-nodeid="99813">
<thead data-nodeid="99814">
<tr data-nodeid="99815">
<th data-org-content="**配置项**" data-nodeid="99817"><strong data-nodeid="99987">配置项</strong></th>
<th data-org-content="**含义**" data-nodeid="99818"><strong data-nodeid="99991">含义</strong></th>
</tr>
</thead>
<tbody data-nodeid="99821">
<tr data-nodeid="99822">
<td data-org-content="userMaxAppsDefault" data-nodeid="99823">userMaxAppsDefault</td>
<td data-org-content="默认的用户最多可同时运行多少个应用程序" data-nodeid="99824">默认的用户最多可同时运行多少个应用程序</td>
</tr>
<tr data-nodeid="99825">
<td data-org-content="minResources" data-nodeid="99826">minResources</td>
<td data-org-content="设置最少资源保证量，设置格式为“X mb, Y vcores”，当一个队列的最少资源保证量未满足时，它将优先于其他同级队列获得资源" data-nodeid="99827">设置最少资源保证量，设置格式为“X mb, Y vcores”，当一个队列的最少资源保证量未满足时，它将优先于其他同级队列获得资源</td>
</tr>
<tr data-nodeid="99828">
<td data-org-content="maxResources" data-nodeid="99829">maxResources</td>
<td data-org-content="设置最多可以使用的资源量，fair scheduler 会保证每个队列使用的资源量不会超过该队列的最多可使用资源量" data-nodeid="99830">设置最多可以使用的资源量，fair scheduler 会保证每个队列使用的资源量不会超过该队列的最多可使用资源量</td>
</tr>
<tr data-nodeid="99831">
<td data-org-content="maxRunningApps" data-nodeid="99832">maxRunningApps</td>
<td data-org-content="设置最多同时运行的应用程序数" data-nodeid="99833">设置最多同时运行的应用程序数</td>
</tr>
<tr data-nodeid="99834">
<td data-org-content="schedulingMode" data-nodeid="99835">schedulingMode</td>
<td data-org-content="设置队列采用的调度模式，可以是 fifo、fair 或者 drf" data-nodeid="99836">设置队列采用的调度模式，可以是 fifo、fair 或者 drf</td>
</tr>
<tr data-nodeid="99837">
<td data-org-content="weight" data-nodeid="99838">weight</td>
<td data-org-content="设置队列的权重，权重越高，可获取的资源就越多" data-nodeid="99839">设置队列的权重，权重越高，可获取的资源就越多</td>
</tr>
<tr data-nodeid="99840">
<td data-org-content="aclSubmitApps" data-nodeid="99841">aclSubmitApps</td>
<td data-org-content="表示可向队列中提交应用程序的用户和组列表，默认情况下为“*”，表示任何用户和组均可以向该队列提交应用程序" data-nodeid="99842">表示可向队列中提交应用程序的用户和组列表，默认情况下为“*”，表示任何用户和组均可以向该队列提交应用程序</td>
</tr>
</tbody>
</table>
<p data-nodeid="99843">再来看一下队列执行规则列表（Queue Placement Policy），Fair 调度器采用了一套基于规则的配置来确定应用应该放到哪个队列中。在上面的例子中，我定义了一个规则列表，总共有四个规则，其中的每个规则会被逐个尝试，直到匹配成功。</p>
<p data-nodeid="99844">例如，第一个规则是 user，表示将提交任务的用户名作为队列名，然后将任务放到这个队列中执行；第二个规则 primaryGroup，表示将提交任务的用户所属的主组作为队列名；第三个规则 secondaryGroupExistingQueue 表示将提交任务的用户所属的附属组作为队列名；最后一个规则 default，表示当前面所有规则都不满足时，用户提交的任务会放到 default 队列中。</p>
<p data-nodeid="99845">除了上面的规则之外，还可以在 yarn-site.xml 文件添加如下配置：</p>
<pre class="lang-js" data-nodeid="121782"><code data-language="js">&lt;property&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.scheduler.fair.user-as-default-queue<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">description</span>&gt;</span>default is True<span class="hljs-tag">&lt;/<span class="hljs-name">description</span>&gt;</span></span>
&lt;/property&gt;
</code></pre>


<p data-nodeid="99847">此配置值默认为 true，表示当任务中未指定队列名时，将以用户名作为队列名，这个配置就实现了根据用户名自动分配队列；如果设置为 false，那么所有任务会被放入 default 队列，而不是放到基于用户名的队列中。</p>
<p data-nodeid="99848">另外，我们还可以在 yarn-site.xml 文件添加如下配置：</p>
<pre class="lang-js" data-nodeid="122808"><code data-language="js">&lt;property&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.scheduler.fair.allow-undeclared-pools<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>false<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span></span>
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">description</span>&gt;</span>default is True<span class="hljs-tag">&lt;/<span class="hljs-name">description</span>&gt;</span></span>
&lt;/property&gt;
</code></pre>


<p data-nodeid="99850">此配置表示是否允许创建未定义的队列，默认值为 true，表示 Yarn 将会自动创建任务中指定的未定义过的队列名。设置成 false 后，用户就无法创建队列了，该任务会被分配到 default 队列中。</p>
<p data-nodeid="99851">最后，再来说下<strong data-nodeid="100019">资源抢占</strong>，当一个任务提交到一个繁忙集群中的空队列时，任务并不会马上执行，而是暂时阻塞，直到正在运行的任务释放系统资源，才开始执行。为了使提交的任务执行时间更具预测性（可以设置等待的超时时间），Fair 调度器支持抢占。</p>
<p data-nodeid="99852">抢占就是允许调度器杀掉占用超过其应占资源份额队列的 containers，这些 containers 资源释放后可，被分配到应该享有这些份额资源的队列中。需要注意，抢占会降低集群的执行效率，因为被终止的 containers 需要被重新执行。</p>
<p data-nodeid="99853">要启用抢占模式，可以在 yarn-site.xml 文件中添加如下配置：</p>
<pre class="lang-js" data-nodeid="123834"><code data-language="js">    &lt;property&gt;
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">name</span>&gt;</span>yarn.scheduler.fair.preemption<span class="hljs-tag">&lt;/<span class="hljs-name">name</span>&gt;</span></span>
      <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">value</span>&gt;</span>true<span class="hljs-tag">&lt;/<span class="hljs-name">value</span>&gt;</span></span>
    &lt;/property&gt;
</code></pre>


<p data-nodeid="99855">可以设置此参数为 true 来启用抢占功能。此外，还需要在 fair-scheduler.xml 文件中添加一个参数用来控制抢占的过期时间，参数设置如下：</p>
<pre class="lang-js" data-nodeid="124860"><code data-language="js">&lt;fairSharePreemptionTimeout&gt;<span class="hljs-number">60</span>&lt;/fairSharePreemptionTimeout&gt;
</code></pre>


<p data-nodeid="99857">此参数用来设置某个队列的超时时间，如果队列在指定的时间内未获得最小的资源保障，调度器就会抢占 container。</p>
<p data-nodeid="99858">还可以在 fair-scheduler.xml 文件中添加全局配置参数，内容如下：</p>
<pre class="lang-js" data-nodeid="125886"><code data-language="js">&lt;defaultFairSharePreemptionTimeout&gt;<span class="hljs-number">60</span>&lt;/defaultFairSharePreemptionTimeout&gt;
</code></pre>


<p data-nodeid="99860">此参数用来配置所有队列的超时时间。</p>
<p data-nodeid="99861">这里需要注意，在 fair-scheduler.xml 配置中，添加了用户和用户组，这里的用户和用户组的对应关系，需要维护在 ResourceManager 上，ResourceManager 在分配资源池时候，是从 ResourceManager 所在的操作系统上读取用户和用户组的对应关系的，否则就会被分配到default 队列中。而客户端机器上的用户对应的用户组无关紧要。</p>
<p data-nodeid="99862">在 fair-scheduler.xml 第一次添加、配置完成后，需要重启 Yarn 集群才能生效，而后面再对 fair-scheduler.xml 进行修改用户或者调整资源池配额后，无须重启 yarn 集群，只需执行下面的命令刷新即可生效：</p>
<pre class="lang-js" data-nodeid="126912"><code data-language="js">[hadoop@yarnserver ~]$ yarn rmadmin -refreshQueues
[hadoop@yarnserver ~]$ yarn rmadmin -refreshUserToGroupsMappings
</code></pre>


<p data-nodeid="99864">动态更新只支持修改资源池配额，如果是新增或减少资源池，则还需要重启 Yarn 集群。</p>
<h3 data-nodeid="128964" class="">容量调度与公平调度对比与选型</h3>




<h4 data-nodeid="99866">1. 相同</h4>
<p data-nodeid="99867">容量调度和公平调度实现的功能基本一致，例如，它们都支持多用户、多队列，即都适用于多用户共享集群的应用环境。同时，单个队列均支持优先级和 FIFO 调度方式，还支持资源共享，即某个队列中的资源有剩余时，可共享给其他缺资源的队列。</p>
<h4 data-nodeid="99868">2. 不同</h4>
<ul data-nodeid="99869">
<li data-nodeid="99870">
<p data-nodeid="99871"><strong data-nodeid="100039">核心调度策略不同</strong></p>
</li>
</ul>
<p data-nodeid="99872">容量调度器的调度策略是，先选择资源利用率低的队列，然后在队列中同时考虑 FIFO 和内存因素；而公平调度器仅考虑公平，而公平是通过任务缺额体现的，调度器每次选择缺额最大的任务（队列的资源量，任务的优先级等仅用于计算任务缺额）。</p>
<ul data-nodeid="99873">
<li data-nodeid="99874">
<p data-nodeid="99875"><strong data-nodeid="100044">对特殊任务的处理不同</strong></p>
</li>
</ul>
<p data-nodeid="99876">容量调度器调度任务时会考虑作业的内存限制，为了满足某些特殊任务的特殊内存需求，可能会为该任务分配多个 slot；而公平调度器对这种特殊的任务无能为力，只能杀掉这种任务。</p>
<p data-nodeid="99877">因此，具体选用哪种调度算法，可根据实际应用需求而定。<strong data-nodeid="100050">一个基本的经验是，小型 Yarn 集群（100 个节点以内），可考虑使用公平调度器，而大型 Yarn 集群（超过 100 个节点）可采用容量调度器效果会更好。</strong></p>
<h3 data-nodeid="99878">小结</h3>
<p data-nodeid="99879">本课时主要介绍了 Yarn 集群中常用的两个资源调度器：<strong data-nodeid="100057">容量调度和公平调度</strong>。通过该课时的学习，我们了解到，在多个用户同时使用 Yarn 集群的时候，合理地设置调度器可以有效利用集群资源，并减少资源争抢，使集群资源利用率达到最大化。</p>

---

### 精选评论

##### **5241：
> 老师您好，谢谢您提供了这么好的课程，关于capacity scheduler即容量调度器有个问题咨询您下。经常看到介绍capacity scheduler时一般会说“当某个队列的资源空闲时，可以将它的剩余资源共享给其他队列”，即所谓弹性。我举个例子，我有A、B两个队列，其中B队列又有B1和B2两个子队列，假设B队列的yarn.scheduler.capacity..maximum-capacity参数设为60%，如果此时A队列没有任务，那么系统最多也是将60%的资源给B队列是吗？即一个队列占有的资源是不能突破maximum-capacity的，哪怕此时别的队列都是idle的？不知道我理解的是否正确，请老师指点，谢谢。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 队列资源是可以抢占的，前提是要开启抢占，所谓抢占就是其他队列空闲，而我的队列资源不够，是可以强占空闲队列资源的，至于抢占多少，是根据你需要的资源而定，不是全部抢占，而此时如果之前空闲队列有任务要运行的话，他也会抢占回来他自己应得的资源，这种情况下，如果你的队列正在运行任务，可能导致运行失败。Yarn优先选择优先级低的Container作为资源抢占对象，且不会立刻杀死Container，而是将释放资源的任务留给应用程序自己：ResourceManager将待杀死的Container列表发送给对应的ApplicationMaster，以期望它采取一定的机制自行释放这些Container占用的资源，比如先进行一些状态保存工作后，再将对应的Container杀死，以避免计算浪费，如果一段时间后，ApplicationMaster尚未主动杀死这些Container，则ResourceManager再强制杀死这些Container。通过将yarn.scheduler.fair.preemption设置为true，可以全面启用抢占功能。有两个相关的抢占超时设置，一个用于最小共享（minimum share preemption timeout），另一个用于公平共享（fair share preemption timeout），两者设定时间均为秒级。默认情况下，两个超时参数均不设置。所有为了允许抢占容器，需要至少设置其中一个超时参数。

##### **7324：
> 对小型 Yarn 集群 更适合公平调度器 ，而yarn集群规模变大时 适合容量调度器 ，这句话不理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是一个使用的经验，小型集群，可能机器配置资源不高，最好是公平调度原则，资源平均分配，而大集群的话，使用容量调度会更好，因为主机较多，资源较多，这其实是跟两种调度策略有关，因为容量调度会尽量让任务在一个节点跑满，主要此节点有资源，不追求公平分配资源。

