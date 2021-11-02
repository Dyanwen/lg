<p data-nodeid="305914">上一课时，我们知道使用不同的 Namespace，可以实现容器中的进程看不到别的容器的资源，但是有一个问题你是否注意到？容器内的进程仍然可以任意地使用主机的 CPU 、内存等资源，如果某一个容器使用的主机资源过多，可能导致主机的资源竞争，进而影响业务。那如果我们想限制一个容器资源的使用（如 CPU、内存等）应该如何做呢？</p>



<p data-nodeid="305215">这里就需要用到 Linux 内核的另一个核心技术cgroups。那么究竟什么是cgroups？我们应该如何使用cgroups？Docker 又是如何使用cgroups的？下面我带你一一解密。</p>
<p data-nodeid="305216">首先我们来学习下什么是cgroups。</p>
<h3 data-nodeid="305217">cgroups</h3>
<p data-nodeid="305218">cgroups（全称：control groups）是 Linux 内核的一个功能，它可以实现限制进程或者进程组的资源（如 CPU、内存、磁盘 IO 等）。</p>
<blockquote data-nodeid="306376">
<p data-nodeid="306377" class="">在 2006 年，Google 的工程师（ Rohit Seth 和 Paul Menage 为主要发起人） 发起了这个项目，起初项目名称并不是cgroups，而被称为进程容器（process containers）。在 2007 年cgroups代码计划合入Linux 内核，但是当时在 Linux 内核中，容器（container）这个词被广泛使用，并且拥有不同的含义。为了避免命名混乱和歧义，进程容器被重名为cgroups，并在 2008 年成功合入 Linux 2.6.24 版本中。cgroups目前已经成为 systemd、Docker、Linux Containers（LXC） 等技术的基础。</p>
</blockquote>
<h4 data-nodeid="306378">cgroups 功能及核心概念</h4>


<p data-nodeid="305223">cgroups 主要提供了如下功能。</p>
<ul data-nodeid="305224">
<li data-nodeid="305225">
<p data-nodeid="305226">资源限制： 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。</p>
</li>
<li data-nodeid="305227">
<p data-nodeid="305228">优先级控制：不同的组可以有不同的资源（ CPU 、磁盘 IO 等）使用优先级。</p>
</li>
<li data-nodeid="305229">
<p data-nodeid="305230">审计：计算控制组的资源使用情况。</p>
</li>
<li data-nodeid="305231">
<p data-nodeid="305232">控制：控制进程的挂起或恢复。</p>
</li>
</ul>
<p data-nodeid="305233">了解了  cgroups 可以为我们提供什么功能，下面我来看下 cgroups 是如何实现这些功能的。</p>
<p data-nodeid="305234">cgroups功能的实现依赖于三个核心概念：子系统、控制组、层级树。</p>
<ul data-nodeid="305235">
<li data-nodeid="305236">
<p data-nodeid="305237">子系统（subsystem）：是一个内核的组件，一个子系统代表一类资源调度控制器。例如内存子系统可以限制内存的使用量，CPU 子系统可以限制 CPU 的使用时间。</p>
</li>
<li data-nodeid="305238">
<p data-nodeid="305239">控制组（cgroup）：表示一组进程和一组带有参数的子系统的关联关系。例如，一个进程使用了 CPU 子系统来限制 CPU 的使用时间，则这个进程和 CPU 子系统的关联关系称为控制组。</p>
</li>
<li data-nodeid="305240">
<p data-nodeid="305241">层级树（hierarchy）：是由一系列的控制组按照树状结构排列组成的。这种排列方式可以使得控制组拥有父子关系，子控制组默认拥有父控制组的属性，也就是子控制组会继承于父控制组。比如，系统中定义了一个控制组 c1，限制了 CPU 可以使用 1 核，然后另外一个控制组 c2 想实现既限制 CPU 使用 1 核，同时限制内存使用 2G，那么 c2 就可以直接继承 c1，无须重复定义 CPU 限制。</p>
</li>
</ul>
<p data-nodeid="305242">cgroups 的三个核心概念中，子系统是最核心的概念，因为子系统是真正实现某类资源的限制的基础。</p>
<h3 data-nodeid="305243">cgroups 子系统实例</h3>
<p data-nodeid="305244">下面我通过一个实例演示一下在 Linux 上默认都启动了哪些子系统。</p>
<p data-nodeid="305245">我们先通过 mount 命令查看一下当前系统已经挂载的cgroups信息：</p>
<pre class="lang-shell" data-nodeid="305246"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo mount -t cgroup</span>
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,net_prio,net_cls)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,blkio)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,pids)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuacct,cpu)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,perf_event)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,freezer)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,devices)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,cpuset)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,hugetlb)
</code></pre>
<blockquote data-nodeid="305247">
<p data-nodeid="305248">我的操作系统版本为 CentOS7.8，内核为 3.10.0-1127.el7.x86_64 版本，不同内核版本cgroups子系统和使用方式可能略有差异。如果你对cgroups不是很熟悉，请尽量使用与我相同的内核环境操作。</p>
</blockquote>
<p data-nodeid="305249">通过输出，可以看到当前系统已经挂载了我们常用的cgroups子系统，例如 cpu、memory、pids 等我们常用的cgroups子系统。这些子系统中，cpu 和 memory 子系统是容器环境中使用最多的子系统，下面我对这两个子系统做详细介绍。</p>
<h4 data-nodeid="305250">cpu 子系统</h4>
<p data-nodeid="305251">我首先以 cpu 子系统为例，演示一下cgroups如何限制进程的 cpu 使用时间。由于cgroups的操作很多需要用到 root 权限，我们在执行命令前要确保已经切换到了 root 用户，以下命令的执行默认都是使用 root 用户。</p>
<p data-nodeid="305252"><strong data-nodeid="305346">第一步：在 cpu 子系统下创建 cgroup</strong></p>
<p data-nodeid="305253">cgroups的创建很简单，只需要在相应的子系统下创建目录即可。下面我们到 cpu 子系统下创建测试文件夹：</p>
<pre class="lang-shell" data-nodeid="305254"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> mkdir /sys/fs/cgroup/cpu/mydocker</span>
</code></pre>
<p data-nodeid="305255">执行完上述命令后，我们查看一下我们新创建的目录下发生了什么？</p>
<pre class="lang-shell" data-nodeid="305256"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> ls -l /sys/fs/cgroup/cpu/mydocker</span>
total 0
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cgroup.clone_children
--w--w--w-. 1 root root 0 Sep&nbsp; 5 09:19 cgroup.event_control
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cgroup.procs
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.cfs_period_us
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.cfs_quota_us
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.rt_period_us
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.rt_runtime_us
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.shares
-r--r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpu.stat
-r--r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpuacct.stat
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpuacct.usage
-r--r--r--. 1 root root 0 Sep&nbsp; 5 09:19 cpuacct.usage_percpu
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 notify_on_release
-rw-r--r--. 1 root root 0 Sep&nbsp; 5 09:19 tasks
</code></pre>
<p data-nodeid="305257">由上可以看到我们新建的目录下被自动创建了很多文件，其中 cpu.cfs_quota_us 文件代表在某一个阶段限制的 CPU 时间总量，单位为微秒。例如，我们想限制某个进程最多使用 1 核 CPU，就在这个文件里写入 100000（100000 代表限制 1 个核） ，tasks 文件中写入进程的 ID 即可（如果要限制多个进程 ID，在 tasks 文件中用换行符分隔即可）。</p>
<p data-nodeid="305258">此时，我们所需要的 cgroup 就创建好了。对，就是这么简单。</p>
<p data-nodeid="305259"><strong data-nodeid="305358">第二步：创建进程，加入 cgroup</strong></p>
<p data-nodeid="305260">这里为了方便演示，我先把当前运行的 shell 进程加入 cgroup，然后在当前 shell 运行 cpu 耗时任务（这里利用到了继承，子进程会继承父进程的 cgroup）。</p>
<p data-nodeid="305261">使用以下命令将 shell 进程加入 cgroup 中：</p>
<pre class="lang-shell" data-nodeid="305262"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> /sys/fs/cgroup/cpu/mydocker</span>
<span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">echo</span> $$ &gt; tasks  </span>
</code></pre>
<p data-nodeid="305263">查看一下 tasks 文件内容：</p>
<pre class="lang-shell" data-nodeid="305264"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> cat tasks</span>
3485
3543
</code></pre>
<p data-nodeid="306839">其中第一个进程 ID 为当前 shell 的主进程，也就是说，当前 shell 主进程为 3485。</p>
<p data-nodeid="306840"><strong data-nodeid="306845">第三步：执行 CPU 耗时任务，验证 cgroup 是否可以限制 cpu 使用时间</strong></p>

<p data-nodeid="305266">下面，我们使用以下命令制造一个死循环，来提升 cpu 使用率：</p>
<pre class="lang-shell" data-nodeid="305267"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-keyword">while</span> <span class="hljs-literal">true</span>;<span class="hljs-keyword">do</span> <span class="hljs-built_in">echo</span>;<span class="hljs-keyword">done</span>;</span>
</code></pre>
<p data-nodeid="305268">执行完上述命令后，我们新打开一个 shell 窗口，使用 top -p 命令查看当前 cpu 使用率，-p 参数后面跟进程 ID，我这里是 3485。</p>
<pre class="lang-shell" data-nodeid="305269"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> top -p 3485</span>
top - 09:51:35 up 3 days, 22:00,&nbsp; 4 users,&nbsp; load average: 1.59, 0.58, 0.27
Tasks:&nbsp; &nbsp;1 total,&nbsp; &nbsp;0 running,&nbsp; &nbsp;1 sleeping,&nbsp; &nbsp;0 stopped,&nbsp; &nbsp;0 zombie
<span class="hljs-meta">%</span><span class="bash">Cpu(s):&nbsp; 9.7 us,&nbsp; 2.8 sy,&nbsp; 0.0 ni, 87.4 id,&nbsp; 0.0 wa,&nbsp; 0.0 hi,&nbsp; 0.0 si,&nbsp; 0.0 st</span>
KiB Mem : 32779616 total, 31009780 free,&nbsp; &nbsp;495988 used,&nbsp; 1273848 buff/cache
KiB Swap:&nbsp; &nbsp; &nbsp; &nbsp; 0 total,&nbsp; &nbsp; &nbsp; &nbsp; 0 free,&nbsp; &nbsp; &nbsp; &nbsp; 0 used. 31852336 avail Mem

&nbsp; PID USER&nbsp; &nbsp; &nbsp; PR&nbsp; NI&nbsp; &nbsp; VIRT&nbsp; &nbsp; RES&nbsp; &nbsp; SHR S&nbsp; %CPU %MEM&nbsp; &nbsp; &nbsp;TIME+ COMMAND
3485 root&nbsp; &nbsp; &nbsp; 20&nbsp; &nbsp;0&nbsp; 116336&nbsp; &nbsp;2852&nbsp; &nbsp;1688 S&nbsp; 99.7&nbsp; 0.0&nbsp; &nbsp;2:10.71 bash
</code></pre>
<p data-nodeid="305270">通过上面输出可以看到 3485 这个进程被限制到了只能使用 100 % 的 cpu，也就是 1 个核。说明我们使用 cgroup 来限制 cpu 使用时间已经生效。此时，执行 while 循环的命令行窗口可以使用 Ctrl+c 退出循环。</p>
<p data-nodeid="305271">为了进一步证实 cgroup 限制 cpu 的准确性，我们修改 cpu 限制时间为 0.5 核，命令如下：</p>
<pre class="lang-shell" data-nodeid="305272"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> /sys/fs/cgroup/cpu/mydocker</span>
<span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">echo</span> 50000 &gt; cpu.cfs_quota_us</span>
</code></pre>
<p data-nodeid="305273">同样使用上面的命令来制造死循环：</p>
<pre class="lang-shell" data-nodeid="305274"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-keyword">while</span> <span class="hljs-literal">true</span>;<span class="hljs-keyword">do</span> <span class="hljs-built_in">echo</span>;<span class="hljs-keyword">done</span>;</span>
</code></pre>
<p data-nodeid="305275">保持当前窗口，新打开一个 shell 窗口，使用 top -p 参数查看 cpu 使用率：</p>
<pre class="lang-shell" data-nodeid="305276"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> top -p 3485</span>
top - 10:05:25 up 3 days, 22:14,&nbsp; 3 users,&nbsp; load average: 1.02, 0.43, 0.40
Tasks:&nbsp; &nbsp;1 total,&nbsp; &nbsp;1 running,&nbsp; &nbsp;0 sleeping,&nbsp; &nbsp;0 stopped,&nbsp; &nbsp;0 zombie
<span class="hljs-meta">%</span><span class="bash">Cpu(s):&nbsp; 5.0 us,&nbsp; 1.3 sy,&nbsp; 0.0 ni, 93.7 id,&nbsp; 0.0 wa,&nbsp; 0.0 hi,&nbsp; 0.0 si,&nbsp; 0.0 st</span>
KiB Mem : 32779616 total, 31055676 free,&nbsp; &nbsp;450224 used,&nbsp; 1273716 buff/cache
KiB Swap:&nbsp; &nbsp; &nbsp; &nbsp; 0 total,&nbsp; &nbsp; &nbsp; &nbsp; 0 free,&nbsp; &nbsp; &nbsp; &nbsp; 0 used. 31898216 avail Mem

&nbsp; PID USER&nbsp; &nbsp; &nbsp; PR&nbsp; NI&nbsp; &nbsp; VIRT&nbsp; &nbsp; RES&nbsp; &nbsp; SHR S&nbsp; %CPU %MEM&nbsp; &nbsp; &nbsp;TIME+ COMMAND
&nbsp;3485 root&nbsp; &nbsp; &nbsp; 20&nbsp; &nbsp;0&nbsp; 115544&nbsp; &nbsp;2116&nbsp; &nbsp;1664 R&nbsp; 50.0&nbsp; 0.0&nbsp; &nbsp;0:23.39 bash
</code></pre>
<p data-nodeid="305277">通过上面输出可以看到，此时 cpu 使用率已经被限制到了 50%，即 0.5 个核。<br>
验证完 cgroup 限制 cpu，我们使用相似的方法来验证 cgroup 对内存的限制。</p>
<h4 data-nodeid="305278">memroy 子系统</h4>
<p data-nodeid="305279"><strong data-nodeid="305381">第一步：在 memory 子系统下创建 cgroup</strong></p>
<pre class="lang-shell" data-nodeid="305280"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> mkdir /sys/fs/cgroup/memory/mydocker</span>
</code></pre>
<p data-nodeid="305281">同样，我们查看一下新创建的目录下发生了什么？</p>
<pre class="lang-java" data-nodeid="311728"><code data-language="java">total <span class="hljs-number">0</span>
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> cgroup.clone_children
--w--w--w-. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> cgroup.event_control
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> cgroup.procs
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.failcnt
--w-------. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.force_empty
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.failcnt
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.limit_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.max_usage_in_bytes
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.slabinfo
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.tcp.failcnt
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.tcp.limit_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.tcp.max_usage_in_bytes
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.tcp.usage_in_bytes
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.kmem.usage_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.limit_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.max_usage_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.memsw.failcnt
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.memsw.limit_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.memsw.max_usage_in_bytes
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.memsw.usage_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.move_charge_at_immigrate
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.numa_stat
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.oom_control
----------. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.pressure_level
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.soft_limit_in_bytes
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.stat
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.swappiness
-r--r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.usage_in_bytes
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> memory.use_hierarchy
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> notify_on_release
-rw-r--r--. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">5</span> <span class="hljs-number">10</span>:<span class="hljs-number">18</span> tasks
</code></pre>
<p data-nodeid="312281">其中 memory.limit_in_bytes 文件代表内存使用总量，单位为 byte。</p>
<p data-nodeid="312282">例如，这里我希望对内存使用限制为 1G，则向 memory.limit_in_bytes 文件写入 1073741824，命令如下：</p>

<pre class="lang-shell" data-nodeid="311730"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> /sys/fs/cgroup/memory/mydocker</span>
<span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">echo</span> 1073741824 &gt; memory.limit_in_bytes</span>
</code></pre>
<p data-nodeid="311731"><strong data-nodeid="311776">第二步：创建进程，加入 cgroup</strong><br>
同样把当前 shell 进程 ID 写入 tasks 文件内：</p>
<pre class="lang-shell" data-nodeid="311732"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> /sys/fs/cgroup/memory/mydocker</span>
<span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">echo</span> $$ &gt; tasks </span>
</code></pre>
<p data-nodeid="311733"><strong data-nodeid="311786">第三步，执行内存测试工具，申请内存</strong><br>
这里我们需要借助一下工具 memtester，memtester 的安装这里不再详细介绍了。具体安装方式可以参考<a href="https://wilhelmguo.cn/blog/post/william/CentOS7-%E5%AE%89%E8%A3%85%E5%86%85%E5%AD%98%E6%B5%8B%E8%AF%95%E5%B7%A5%E5%85%B7-memtester" data-nodeid="311784">这里</a>。</p>
<p data-nodeid="311734">安装好 memtester 后，我们执行以下命令：</p>
<pre class="lang-shell" data-nodeid="311735"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> memtester 1500M 1</span>
memtester version 4.2.2 (64-bit)
Copyright (C) 2010 Charles Cazabon.
Licensed under the GNU General Public License version 2 (only).

pagesize is 4096
pagesizemask is 0xfffffffffffff000
want 1500MB (1572864000 bytes)
got&nbsp; 1500MB (1572864000 bytes), trying mlock ...Killed
</code></pre>
<p data-nodeid="312751">该命令会申请 1500 M 内存，并且做内存测试。由于上面我们对当前 shell 进程内存限制为 1 G，当 memtester 使用的内存达到 1G 时，cgroup 便将 memtester 杀死。</p>
<p data-nodeid="312752">上面最后一行的输出结果表示 memtester 想要 1500 M 内存，但是由于 cgroup 限制，达到了内存使用上限，被杀死了，与我们的预期一致。</p>

<p data-nodeid="311737">我们可以使用以下命令，降低一下内存申请，将内存申请调整为 500M：</p>
<pre class="lang-shell" data-nodeid="311738"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> memtester 500M 1</span>
memtester version 4.2.2 (64-bit)
Copyright (C) 2010 Charles Cazabon.
Licensed under the GNU General Public License version 2 (only).

pagesize is 4096
pagesizemask is 0xfffffffffffff000
want 500MB (524288000 bytes)
got&nbsp; 500MB (524288000 bytes), trying mlock ...locked.
Loop 1/1:
&nbsp; Stuck Address&nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Random Value&nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Compare XOR&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Compare SUB&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Compare MUL&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Compare DIV&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Compare OR&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Compare AND&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: ok
&nbsp; Sequential Increment: ok
&nbsp; Solid Bits&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Block Sequential&nbsp; &nbsp; : ok
&nbsp; Checkerboard&nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Bit Spread&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Bit Flip&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Walking Ones&nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; Walking Zeroes&nbsp; &nbsp; &nbsp; : ok
&nbsp; 8-bit Writes&nbsp; &nbsp; &nbsp; &nbsp; : ok
&nbsp; 16-bit Writes&nbsp; &nbsp; &nbsp; &nbsp;: ok

Done.
</code></pre>
<p data-nodeid="313213" class="">这里可以看到，此时 memtester 已经成功申请到 500M 内存并且正常完成了内存测试。<br>
到此，我们讲解了cgroups的 cpu 和 memroy 子系统，如果你想了解更多的cgroups的知识和使用，可以参考 <a href="https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/resource_management_guide/chap-introduction_to_control_groups" data-nodeid="313219">Red Hat 官网</a>。</p>

<h4 data-nodeid="315513" class="">删除 cgroups</h4>





<p data-nodeid="311741">上面创建的cgroups如果不想使用了，直接删除创建的文件夹即可。</p>
<p data-nodeid="311742">例如我想删除内存下的 mydocker 目录，使用以下命令即可：</p>
<pre class="lang-shell" data-nodeid="311743"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> rmdir /sys/fs/cgroup/memory/mydocker/</span>
</code></pre>
<p data-nodeid="311744">学习了cgroups的使用方式，下面我带你了解一下 Docker 是如何使用cgroups的。</p>
<h3 data-nodeid="311745">Docker 是如何使用cgroups的？</h3>
<p data-nodeid="311746">首先，我们使用以下命令创建一个 nginx 容器：</p>
<pre class="lang-shell" data-nodeid="311747"><code data-language="shell">docker run -it -m=1g nginx
</code></pre>
<p data-nodeid="311748">上述命令创建并启动了一个 nginx 容器，并且限制内存为 1G。然后我们进入cgroups内存子系统的目录，使用 ls 命令查看一下该目录下的内容：</p>
<pre class="lang-shell" data-nodeid="311749"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> ls -l /sys/fs/cgroup/memory</span>
total 0
-rw-r--r--.&nbsp; 1 root root 0 Sep&nbsp; 1 11:50 cgroup.clone_children
--w--w--w-.&nbsp; 1 root root 0 Sep&nbsp; 1 11:50 cgroup.event_control
-rw-r--r--.&nbsp; 1 root root 0 Sep&nbsp; 1 11:50 cgroup.procs
-r--r--r--.&nbsp; 1 root root 0 Sep&nbsp; 1 11:50 cgroup.sane_behavior
drwxr-xr-x.&nbsp; 3 root root 0 Sep&nbsp; 5 10:50 docker
... 省略部分输出
</code></pre>
<p data-nodeid="311750">通过上面输出可以看到，该目录下有一个 docker 目录，该目录正是 Docker 在内存子系统下创建的。我们进入到 docker 目录下查看一下相关内容：</p>
<pre class="lang-shell" data-nodeid="311751"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> /sys/fs/cgroup/memory/docker</span>
<span class="hljs-meta">#</span><span class="bash"> ls -l</span>
total 0
drwxr-xr-x. 2 root root 0 Sep&nbsp; 5 10:49 cb5c5391177b44ad87636bf3840ecdda83529e51b76a6406d6742f56a2535d5e
-rw-r--r--. 1 root root 0 Sep&nbsp; 4 10:40 cgroup.clone_children
--w--w--w-. 1 root root 0 Sep&nbsp; 4 10:40 cgroup.event_control
-rw-r--r--. 1 root root 0 Sep&nbsp; 4 10:40 cgroup.procs
... 省略部分输出
-rw-r--r--. 1 root root 0 Sep&nbsp; 4 10:40 tasks
</code></pre>
<p data-nodeid="311752">可以看到 docker 的目录下有一个一串随机 ID 的目录，该目录即为我们上面创建的 nginx 容器的 ID。然后我们进入该目录，查看一下该容器的 memory.limit_in_bytes 文件的内容。</p>
<pre class="lang-shell" data-nodeid="311753"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> <span class="hljs-built_in">cd</span> cb5c5391177b44ad87636bf3840ecdda83529e51b76a6406d6742f56a2535d5e</span>
<span class="hljs-meta">#</span><span class="bash"> cat memory.limit_in_bytes</span>
1073741824
</code></pre>
<p data-nodeid="311754">可以看到内存限制值正好为 1G。<br>
事实上，Docker 创建容器时，Docker  会根据启动容器的参数，在对应的 cgroups 子系统下创建以容器 ID 为名称的目录, 然后根据容器启动时设置的资源限制参数, 修改对应的 cgroups 子系统资源限制文件, 从而达到资源限制的效果。</p>
<h3 data-nodeid="311755">小结</h3>
<p data-nodeid="311756">本课时我们讲解了什么是 cgroups，以及 cgroups 可以为我们提供哪些核心功能。其实 cgroups 不仅可以实现资源的限制，还可以为我们统计资源的使用情况，容器监控系统的数据来源也是 cgroups 提供的。</p>
<p data-nodeid="311757">另外，请注意 cgroups 虽然可以实现资源的限制，但是不能保证资源的使用。例如，cgroups 限制某个容器最多使用 1 核 CPU，但不保证总是能使用到 1 核 CPU，当 CPU 资源发生竞争时，可能会导致实际使用的 CPU 资源产生竞争。</p>
<p data-nodeid="315967">那么，你知道cgroups还有哪些子系统吗？思考后，把你的想法写在留言区。</p>

---

### 精选评论

##### **辉：
> 删除 cgroups不能在关联的shell进程中进行，需要另外shell

##### **彬：
> mac 系统的这个目录是什么 /sys/fs/cgroup/memory/docker

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个目录在虚拟机里哈，从主机上无法看到的

##### *瑞：
> 设置了docker的CPU使用数，但是在容器内使用top还是主机的cpu数量，有什么办法和设置的一样吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以使用 LXCFS 实现，参考文章：https://developer.aliyun.com/article/566208

##### **平：
> blkio: 这个子系统为块设备设定输入/输出限制，比如物理设备（磁盘，固态硬盘，USB等等）.cpu: 这个子系统使用调度程序提供对任务访问 .cpuacct:这个子系统自动生成报告.cpuset :这个子系统为CPU（在多核系统）和内存节点。devices:中的任务访问设备。freezer中的任务。memory:这个子系统设定中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告。net_cls:这个子系统使用等级识别符（classid）标记网络数据包，可允许中生成的数据包。ns :名称空间子系统。

##### *冲：
> 我的cgroup没做限制cpu,死循环也只能达到11% 的cpu使用率

##### **明：
> cpu -- 这个子系统使用调度程序提供对 CPU 的 cgroup 任务访问.memory中任务使用的内存限制，并自动生成由那些任务使用的内存资源报告.freezer中的任务.cpuset -- 这个子系统为 cgroup 中的任务分配独立 CPU（在多核系统）和内存节点cpuacct -- 这个子系统自动生成 cgroup 中任务所使用的 CPU 报告

