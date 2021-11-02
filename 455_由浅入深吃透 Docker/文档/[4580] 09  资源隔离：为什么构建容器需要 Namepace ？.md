<p data-nodeid="989" class="">我们知道， Docker 是使用 Linux 的 Namespace 技术实现各种资源隔离的。那么究竟什么是 Namespace，各种 Namespace 都有什么作用，为什么 Docker 需要 Namespace呢？下面我带你一一揭秘。</p>
<p data-nodeid="990">首先我们来了解一下什么是 Namespace。</p>
<h3 data-nodeid="991">什么是 Namespace？</h3>
<p data-nodeid="992">下面是 Namespace 的维基百科定义：</p>
<blockquote data-nodeid="993">
<p data-nodeid="994">Namespace 是 Linux 内核的一项功能，该功能对内核资源进行分区，以使一组进程看到一组资源，而另一组进程看到另一组资源。Namespace 的工作方式通过为一组资源和进程设置相同的 Namespace 而起作用，但是这些 Namespace 引用了不同的资源。资源可能存在于多个 Namespace 中。这些资源可以是进程 ID、主机名、用户 ID、文件名、与网络访问相关的名称和进程间通信。</p>
</blockquote>
<p data-nodeid="995">简单来说，Namespace 是 Linux 内核的一个特性，该特性可以实现在同一主机系统中，对进程 ID、主机名、用户 ID、文件名、网络和进程间通信等资源的隔离。Docker 利用 Linux 内核的 Namespace 特性，实现了每个容器的资源相互隔离，从而保证容器内部只能访问到自己 Namespace 的资源。</p>
<p data-nodeid="996">最新的 Linux 5.6 内核中提供了 8 种类型的 Namespace：</p>
<table data-nodeid="998">
<thead data-nodeid="999">
<tr data-nodeid="1000">
<th data-org-content="Namespace 名称" data-nodeid="1002">Namespace 名称</th>
<th data-org-content="作用" data-nodeid="1003">作用</th>
<th data-org-content="内核版本" data-nodeid="1004">内核版本</th>
</tr>
</thead>
<tbody data-nodeid="1008">
<tr data-nodeid="1009">
<td data-org-content="Mount（mnt）" data-nodeid="1010">Mount（mnt）</td>
<td data-org-content="隔离挂载点" data-nodeid="1011">隔离挂载点</td>
<td data-org-content="2.4.19" data-nodeid="1012">2.4.19</td>
</tr>
<tr data-nodeid="1013">
<td data-org-content="Process ID (pid)" data-nodeid="1014">Process ID (pid)</td>
<td data-org-content="隔离进程 ID" data-nodeid="1015">隔离进程 ID</td>
<td data-org-content="2.6.24" data-nodeid="1016">2.6.24</td>
</tr>
<tr data-nodeid="1017">
<td data-org-content="Network (net)" data-nodeid="1018">Network (net)</td>
<td data-org-content="隔离网络设备，端口号等" data-nodeid="1019">隔离网络设备，端口号等</td>
<td data-org-content="2.6.29" data-nodeid="1020">2.6.29</td>
</tr>
<tr data-nodeid="1021">
<td data-org-content="Interprocess Communication (ipc)" data-nodeid="1022">Interprocess Communication (ipc)</td>
<td data-org-content="隔离 System V IPC 和 POSIX message queues" data-nodeid="1023">隔离 System V IPC 和 POSIX message queues</td>
<td data-org-content="2.6.19" data-nodeid="1024">2.6.19</td>
</tr>
<tr data-nodeid="1025">
<td data-org-content="UTS Namespace(uts)" data-nodeid="1026">UTS Namespace(uts)</td>
<td data-org-content="隔离主机名和域名" data-nodeid="1027">隔离主机名和域名</td>
<td data-org-content="2.6.19" data-nodeid="1028">2.6.19</td>
</tr>
<tr data-nodeid="1029">
<td data-org-content="User Namespace (user)" data-nodeid="1030">User Namespace (user)</td>
<td data-org-content="隔离用户和用户组" data-nodeid="1031">隔离用户和用户组</td>
<td data-org-content="3.8" data-nodeid="1032">3.8</td>
</tr>
<tr data-nodeid="1033">
<td data-org-content="Control group (cgroup) Namespace" data-nodeid="1034">Control group (cgroup) Namespace</td>
<td data-org-content="隔离 Cgroups 根目录" data-nodeid="1035">隔离 Cgroups 根目录</td>
<td data-org-content="4.6" data-nodeid="1036">4.6</td>
</tr>
<tr data-nodeid="1037">
<td data-org-content="Time Namespace" data-nodeid="1038">Time Namespace</td>
<td data-org-content="隔离系统时间" data-nodeid="1039">隔离系统时间</td>
<td data-org-content="5.6" data-nodeid="1040">5.6</td>
</tr>
</tbody>
</table>
<p data-nodeid="1041">虽然 Linux 内核提供了8种 Namespace，但是最新版本的 Docker 只使用了其中的前6 种，分别为Mount Namespace、PID Namespace、Net Namespace、IPC Namespace、UTS Namespace、User Namespace。</p>
<p data-nodeid="1042">下面，我们详细了解下 Docker 使用的 6 种 Namespace的作用分别是什么。</p>
<h3 data-nodeid="1043">各种 Namespace 的作用？</h3>
<h4 data-nodeid="1044">（1）Mount Namespace</h4>
<p data-nodeid="1045">Mount Namespace 是 Linux 内核实现的第一个 Namespace，从内核的 2.4.19 版本开始加入。它可以用来隔离不同的进程或进程组看到的挂载点。通俗地说，就是可以实现在不同的进程中看到不同的挂载目录。使用 Mount Namespace 可以实现容器内只能看到自己的挂载信息，在容器内的挂载操作不会影响主机的挂载目录。</p>
<p data-nodeid="1046">下面我们通过一个实例来演示下 Mount Namespace。在演示之前，我们先来认识一个命令行工具 unshare。unshare 是 util-linux 工具包中的一个工具，CentOS 7 系统默认已经集成了该工具，<strong data-nodeid="1173">使用 unshare 命令可以实现创建并访问不同类型的 Namespace</strong>。</p>
<p data-nodeid="1047">首先我们使用以下命令创建一个 bash 进程并且新建一个 Mount Namespace：</p>
<pre class="lang-shell" data-nodeid="1048"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo unshare --mount --fork /bin/bash</span>
[root@centos7 centos]#
</code></pre>
<p data-nodeid="1049">执行完上述命令后，这时我们已经在主机上创建了一个新的 Mount Namespace，并且当前命令行窗口加入了新创建的 Mount Namespace。下面我通过一个例子来验证下，在独立的 Mount Namespace 内创建挂载目录是不影响主机的挂载目录的。</p>
<p data-nodeid="1050">首先在 /tmp 目录下创建一个目录。</p>
<pre class="lang-dart" data-nodeid="1051"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# mkdir /tmp/tmpfs
</code></pre>
<p data-nodeid="1052">创建好目录后使用 mount 命令挂载一个 tmpfs 类型的目录。命令如下：</p>
<pre class="lang-dart" data-nodeid="1053"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# mount -t tmpfs -o size=<span class="hljs-number">20</span>m tmpfs /tmp/tmpfs
</code></pre>
<p data-nodeid="1054">然后使用 df 命令查看一下已经挂载的目录信息：</p>
<pre class="lang-dart" data-nodeid="1055"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# df -h
Filesystem&nbsp; &nbsp; &nbsp; Size&nbsp; Used Avail Use% Mounted <span class="hljs-keyword">on</span>
/dev/vda1&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">1.4</span>G&nbsp; <span class="hljs-number">499</span>G&nbsp; &nbsp;<span class="hljs-number">1</span>% /
devtmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /dev
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /dev/shm
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /sys/fs/cgroup
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">57</span>M&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">1</span>% /run
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">3.2</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; <span class="hljs-number">3.2</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /run/user/<span class="hljs-number">1000</span>
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">20</span>M&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">20</span>M&nbsp; &nbsp;<span class="hljs-number">0</span>% /tmp/tmpfs
</code></pre>
<p data-nodeid="1056">可以看到 /tmp/tmpfs 目录已经被正确挂载。为了验证主机上并没有挂载此目录，我们新打开一个命令行窗口，同样执行 df 命令查看主机的挂载信息：</p>
<pre class="lang-java" data-nodeid="1057"><code data-language="java">[centos<span class="hljs-meta">@centos7</span> ~]$ df -h
Filesystem&nbsp; &nbsp; &nbsp; Size&nbsp; Used Avail Use% Mounted on
devtmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /dev
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /dev/shm
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">57</span>M&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">1</span>% /run
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">16</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; &nbsp;<span class="hljs-number">16</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /sys/fs/cgroup
/dev/vda1&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">500</span>G&nbsp; <span class="hljs-number">1.4</span>G&nbsp; <span class="hljs-number">499</span>G&nbsp; &nbsp;<span class="hljs-number">1</span>% /
tmpfs&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">3.2</span>G&nbsp; &nbsp; &nbsp;<span class="hljs-number">0</span>&nbsp; <span class="hljs-number">3.2</span>G&nbsp; &nbsp;<span class="hljs-number">0</span>% /run/user/<span class="hljs-number">1000</span>
</code></pre>
<p data-nodeid="1058">通过上面输出可以看到主机上并没有挂载 /tmp/tmpfs，可见我们独立的 Mount Namespace 中执行 mount 操作并不会影响主机。</p>
<p data-nodeid="1059">为了进一步验证我们的想法，我们继续在当前命令行窗口查看一下当前进程的 Namespace 信息，命令如下：</p>
<pre class="lang-dart" data-nodeid="1060"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# ls -l /proc/self/ns/
total <span class="hljs-number">0</span>
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> ipc -&gt; ipc:[<span class="hljs-number">4026531839</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> mnt -&gt; mnt:[<span class="hljs-number">4026532239</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> net -&gt; net:[<span class="hljs-number">4026531956</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> pid -&gt; pid:[<span class="hljs-number">4026531836</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> user -&gt; user:[<span class="hljs-number">4026531837</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> root root <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> uts -&gt; uts:[<span class="hljs-number">4026531838</span>]
</code></pre>
<p data-nodeid="1061">然后新打开一个命令行窗口，使用相同的命令查看一下主机上的 Namespace 信息：</p>
<pre class="lang-java" data-nodeid="1062"><code data-language="java">[centos<span class="hljs-meta">@centos7</span> ~]$ ls -l /proc/self/ns/
total <span class="hljs-number">0</span>
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> ipc -&gt; ipc:[<span class="hljs-number">4026531839</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> mnt -&gt; mnt:[<span class="hljs-number">4026531840</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> net -&gt; net:[<span class="hljs-number">4026531956</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> pid -&gt; pid:[<span class="hljs-number">4026531836</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> user -&gt; user:[<span class="hljs-number">4026531837</span>]
lrwxrwxrwx. <span class="hljs-number">1</span> centos centos <span class="hljs-number">0</span> Sep&nbsp; <span class="hljs-number">4</span> <span class="hljs-number">08</span>:<span class="hljs-number">20</span> uts -&gt; uts:[<span class="hljs-number">4026531838</span>]
</code></pre>
<p data-nodeid="1063">通过对比两次命令的输出结果，我们可以看到，除了 Mount Namespace 的 ID 值不一样外，其他Namespace 的 ID 值均一致。</p>
<p data-nodeid="1064">通过以上结果我们可以得出结论，<strong data-nodeid="1188">使用 unshare 命令可以新建 Mount Namespace，并且在新建的 Mount Namespace 内 mount 是和外部完全隔离的。</strong></p>
<h4 data-nodeid="1065">（2）PID Namespace</h4>
<p data-nodeid="1066">PID Namespace 的作用是用来隔离进程。在不同的 PID Namespace 中，进程可以拥有相同的 PID 号，利用 PID Namespace 可以实现每个容器的主进程为 1 号进程，而容器内的进程在主机上却拥有不同的PID。例如一个进程在主机上 PID 为 122，使用 PID Namespace 可以实现该进程在容器内看到的 PID 为 1。</p>
<p data-nodeid="1067">下面我们通过一个实例来演示下 PID Namespace的作用。首先我们使用以下命令创建一个 bash 进程，并且新建一个 PID Namespace：</p>
<pre class="lang-shell" data-nodeid="1068"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo unshare --pid --fork --mount-proc /bin/bash</span>
[root@centos7 centos]#
</code></pre>
<p data-nodeid="1069">执行完上述命令后，我们在主机上创建了一个新的 PID Namespace，并且当前命令行窗口加入了新创建的 PID Namespace。在当前的命令行窗口使用 ps aux 命令查看一下进程信息：</p>
<pre class="lang-dart" data-nodeid="1070"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# ps aux
USER&nbsp; &nbsp; &nbsp; &nbsp;PID %CPU %MEM&nbsp; &nbsp; VSZ&nbsp; &nbsp;RSS TTY&nbsp; &nbsp; &nbsp; STAT START&nbsp; &nbsp;TIME COMMAND
root&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">1</span>&nbsp; <span class="hljs-number">0.0</span>&nbsp; <span class="hljs-number">0.0</span> <span class="hljs-number">115544</span>&nbsp; <span class="hljs-number">2004</span> pts/<span class="hljs-number">0</span>&nbsp; &nbsp; S&nbsp; &nbsp; <span class="hljs-number">10</span>:<span class="hljs-number">57</span>&nbsp; &nbsp;<span class="hljs-number">0</span>:<span class="hljs-number">00</span> bash
root&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">10</span>&nbsp; <span class="hljs-number">0.0</span>&nbsp; <span class="hljs-number">0.0</span> <span class="hljs-number">155444</span>&nbsp; <span class="hljs-number">1764</span> pts/<span class="hljs-number">0</span>&nbsp; &nbsp; R+&nbsp; &nbsp;<span class="hljs-number">10</span>:<span class="hljs-number">59</span>&nbsp; &nbsp;<span class="hljs-number">0</span>:<span class="hljs-number">00</span> ps aux
</code></pre>
<p data-nodeid="1071">通过上述命令输出结果可以看到当前 Namespace 下 bash 为 1 号进程，而且我们也看不到主机上的其他进程信息。</p>
<h4 data-nodeid="1072">（3）UTS Namespace</h4>
<p data-nodeid="1073">UTS Namespace 主要是用来隔离主机名的，它允许每个 UTS Namespace 拥有一个独立的主机名。例如我们的主机名称为 docker，使用 UTS Namespace 可以实现在容器内的主机名称为 lagoudocker 或者其他任意自定义主机名。</p>
<p data-nodeid="1074">同样我们通过一个实例来验证下 UTS Namespace 的作用，首先我们使用 unshare 命令来创建一个 UTS Namespace：</p>
<pre class="lang-shell" data-nodeid="1236"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo unshare --uts --fork /bin/bash</span>
[root@centos7 centos]#
</code></pre>
<p data-nodeid="1237">创建好 UTS Namespace 后，当前命令行窗口已经处于一个独立的 UTS Namespace 中，下面我们使用 hostname 命令（hostname 可以用来查看主机名称）设置一下主机名：</p>
<pre class="lang-shell" data-nodeid="1238"><code data-language="shell">root@centos7 centos]# hostname -b lagoudocker
</code></pre>
<p data-nodeid="1239">然后再查看一下主机名：</p>
<pre class="lang-java" data-nodeid="1240"><code data-language="java">[root@centos7 centos]# hostname
lagoudocker
</code></pre>
<p data-nodeid="1241">通过上面命令的输出，我们可以看到当前UTS Namespace 内的主机名已经被修改为 lagoudocker。然后我们新打开一个命令行窗口，使用相同的命令查看一下主机的 hostname：</p>
<pre class="lang-java" data-nodeid="1242"><code data-language="java">[centos<span class="hljs-meta">@centos7</span> ~]$ hostname
centos7
</code></pre>
<p data-nodeid="1243">可以看到主机的名称仍然为 centos7，并没有被修改。由此，可以验证 UTS Namespace 可以用来隔离主机名。</p>
<h4 data-nodeid="1244">（4）IPC Namespace</h4>
<p data-nodeid="1245">IPC Namespace 主要是用来隔离进程间通信的。例如 PID Namespace 和 IPC Namespace 一起使用可以实现同一 IPC Namespace 内的进程彼此可以通信，不同 IPC Namespace 的进程却不能通信。</p>
<p data-nodeid="1246">同样我们通过一个实例来验证下IPC Namespace的作用，首先我们使用 unshare 命令来创建一个 IPC Namespace：</p>
<pre class="lang-shell" data-nodeid="1247"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo unshare --ipc --fork /bin/bash</span>
[root@centos7 centos]#
</code></pre>
<p data-nodeid="1248">下面我们需要借助两个命令来实现对 IPC Namespace 的验证。</p>
<ul data-nodeid="1249">
<li data-nodeid="1250">
<p data-nodeid="1251">ipcs -q 命令：用来查看系统间通信队列列表。</p>
</li>
<li data-nodeid="1252">
<p data-nodeid="1253">ipcmk -Q 命令：用来创建系统间通信队列。</p>
</li>
</ul>
<p data-nodeid="1254">我们首先使用 ipcs -q 命令查看一下当前 IPC Namespace 下的系统通信队列列表：</p>
<pre class="lang-java" data-nodeid="1255"><code data-language="java">[centos<span class="hljs-meta">@centos7</span> ~]$ ipcs -q

------ Message Queues --------
key&nbsp; &nbsp; &nbsp; &nbsp; msqid&nbsp; &nbsp; &nbsp; owner&nbsp; &nbsp; &nbsp; perms&nbsp; &nbsp; &nbsp; used-bytes&nbsp; &nbsp;messages
</code></pre>
<p data-nodeid="1256">由上可以看到当前无任何系统通信队列，然后我们使用 ipcmk -Q 命令创建一个系统通信队列：</p>
<pre class="lang-dart" data-nodeid="1257"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# ipcmk -Q
Message queue id: <span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="1258">再次使用 ipcs -q 命令查看当前 IPC Namespace 下的系统通信队列列表：</p>
<pre class="lang-dart" data-nodeid="1259"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# ipcs -q

------ Message Queues --------
key&nbsp; &nbsp; &nbsp; &nbsp; msqid&nbsp; &nbsp; &nbsp; owner&nbsp; &nbsp; &nbsp; perms&nbsp; &nbsp; &nbsp; used-bytes&nbsp; &nbsp;messages
<span class="hljs-number">0x73682a32</span> <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; root&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-number">644</span>&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-number">0</span>
</code></pre>
<p data-nodeid="1260">可以看到我们已经成功创建了一个系统通信队列。然后我们新打开一个命令行窗口，使用ipcs -q 命令查看一下主机的系统通信队列：</p>
<pre class="lang-java" data-nodeid="1261"><code data-language="java">[centos<span class="hljs-meta">@centos7</span> ~]$ ipcs -q

------ Message Queues --------
key&nbsp; &nbsp; &nbsp; &nbsp; msqid&nbsp; &nbsp; &nbsp; owner&nbsp; &nbsp; &nbsp; perms&nbsp; &nbsp; &nbsp; used-bytes&nbsp; &nbsp;messages
</code></pre>
<p data-nodeid="1262">通过上面的实验，可以发现，在单独的 IPC Namespace 内创建的系统通信队列在主机上无法看到。即 IPC Namespace 实现了系统通信队列的隔离。</p>
<h4 data-nodeid="1263">（5）User Namespace</h4>
<p data-nodeid="1264">User Namespace 主要是用来隔离用户和用户组的。一个比较典型的应用场景就是在主机上以非 root 用户运行的进程可以在一个单独的 User Namespace 中映射成 root 用户。使用 User Namespace 可以实现进程在容器内拥有 root 权限，而在主机上却只是普通用户。</p>
<p data-nodeid="1265">User Namesapce 的创建是可以不使用 root 权限的。下面我们以普通用户的身份创建一个 User Namespace，命令如下：</p>
<pre class="lang-dart" data-nodeid="1266"><code data-language="dart">[centos<span class="hljs-meta">@centos</span>7 ~]$ unshare --user -r /bin/bash
[root<span class="hljs-meta">@centos</span>7 ~]#
</code></pre>
<blockquote data-nodeid="1267">
<p data-nodeid="1268">CentOS7 默认允许创建的 User Namespace 为 0，如果执行上述命令失败（ unshare 命令返回的错误为 unshare: unshare failed: Invalid argument ），需要使用以下命令修改系统允许创建的&nbsp;User Namespace&nbsp;数量，命令为：echo 65535 &gt; /proc/sys/user/max_user_namespaces，然后再次尝试创建 User Namespace。</p>
</blockquote>
<p data-nodeid="1269">然后执行 id 命令查看一下当前的用户信息：</p>
<pre class="lang-dart" data-nodeid="1270"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 ~]# id
uid=<span class="hljs-number">0</span>(root) gid=<span class="hljs-number">0</span>(root) groups=<span class="hljs-number">0</span>(root),<span class="hljs-number">65534</span>(nfsnobody) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
</code></pre>
<p data-nodeid="1271">通过上面的输出可以看到我们在新的 User Namespace 内已经是 root 用户了。下面我们使用只有主机 root 用户才可以执行的 reboot 命令来验证一下，在当前命令行窗口执行 reboot 命令：</p>
<pre class="lang-dart" data-nodeid="1272"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 ~]# reboot
Failed to open /dev/initctl: Permission denied
Failed to talk to init daemon.
</code></pre>
<p data-nodeid="1273">可以看到，我们在新创建的 User Namespace 内虽然是 root 用户，但是并没有权限执行 reboot 命令。这说明在隔离的 User Namespace 中，并不能获取到主机的 root 权限，也就是说 User Namespace 实现了用户和用户组的隔离。</p>
<h4 data-nodeid="1274">（6）Net Namespace</h4>
<p data-nodeid="1275">Net Namespace 是用来隔离网络设备、IP 地址和端口等信息的。Net Namespace 可以让每个进程拥有自己独立的 IP 地址，端口和网卡信息。例如主机 IP 地址为 172.16.4.1 ，容器内可以设置独立的 IP 地址为 192.168.1.1。</p>
<p data-nodeid="1276">同样用实例验证，我们首先使用 ip a 命令查看一下主机上的网络信息：</p>
<pre class="lang-java" data-nodeid="1277"><code data-language="java">$ ip a
<span class="hljs-number">1</span>: lo: &lt;LOOPBACK,UP,LOWER_UP&gt; mtu <span class="hljs-number">65536</span> qdisc noqueue state UNKNOWN group <span class="hljs-keyword">default</span> qlen <span class="hljs-number">1000</span>
&nbsp; &nbsp; link/loopback <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span> brd <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>
&nbsp; &nbsp; inet <span class="hljs-number">127.0</span>.<span class="hljs-number">0.1</span>/<span class="hljs-number">8</span> scope host lo
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft forever preferred_lft forever
&nbsp; &nbsp; inet6 ::<span class="hljs-number">1</span>/<span class="hljs-number">128</span> scope host
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft forever preferred_lft forever
<span class="hljs-number">2</span>: eth0: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu <span class="hljs-number">1500</span> qdisc pfifo_fast state UP group <span class="hljs-keyword">default</span> qlen <span class="hljs-number">1000</span>
&nbsp; &nbsp; link/ether <span class="hljs-number">02</span>:<span class="hljs-number">11</span>:b0:<span class="hljs-number">14</span>:<span class="hljs-number">01</span>:<span class="hljs-number">0</span>c brd ff:ff:ff:ff:ff:ff
&nbsp; &nbsp; inet <span class="hljs-number">172.20</span>.<span class="hljs-number">1.11</span>/<span class="hljs-number">24</span> brd <span class="hljs-number">172.20</span>.<span class="hljs-number">1.255</span> scope global dynamic eth0
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft <span class="hljs-number">86063337</span>sec preferred_lft <span class="hljs-number">86063337</span>sec
&nbsp; &nbsp; inet6 fe80::<span class="hljs-number">11</span>:b0ff:fe14:<span class="hljs-number">10</span>c/<span class="hljs-number">64</span> scope link
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft forever preferred_lft forever
<span class="hljs-number">3</span>: docker0: &lt;NO-CARRIER,BROADCAST,MULTICAST,UP&gt; mtu <span class="hljs-number">1500</span> qdisc noqueue state DOWN group <span class="hljs-keyword">default</span>
&nbsp; &nbsp; link/ether <span class="hljs-number">02</span>:<span class="hljs-number">42</span>:<span class="hljs-number">82</span>:<span class="hljs-number">8d</span>:a0:df brd ff:ff:ff:ff:ff:ff
&nbsp; &nbsp; inet <span class="hljs-number">172.17</span>.<span class="hljs-number">0.1</span>/<span class="hljs-number">16</span> scope global docker0
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft forever preferred_lft forever
&nbsp; &nbsp; inet6 fe80::<span class="hljs-number">42</span>:<span class="hljs-number">82f</span>f:fe8d:a0df/<span class="hljs-number">64</span> scope link
&nbsp; &nbsp; &nbsp; &nbsp;valid_lft forever preferred_lft forever
</code></pre>
<p data-nodeid="1278">然后我们使用以下命令创建一个 Net Namespace：</p>
<pre class="lang-shell" data-nodeid="1279"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> sudo unshare --net --fork /bin/bash</span>
[root@centos7 centos]#
</code></pre>
<p data-nodeid="1280">同样的我们使用 ip a 命令查看一下网络信息：</p>
<pre class="lang-dart" data-nodeid="1281"><code data-language="dart">[root<span class="hljs-meta">@centos</span>7 centos]# ip a
<span class="hljs-number">1</span>: lo: &lt;LOOPBACK&gt; mtu <span class="hljs-number">65536</span> qdisc noop state DOWN group <span class="hljs-keyword">default</span> qlen <span class="hljs-number">1000</span>
&nbsp; &nbsp; link/loopback <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span> brd <span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>:<span class="hljs-number">00</span>
</code></pre>
<p data-nodeid="1282">可以看到，宿主机上有 lo、eth0、docker0 等网络设备，而我们新建的 Net Namespace 内则与主机上的网络设备不同。</p>
<h3 data-nodeid="1283">为什么 Docker 需要 Namespace？</h3>
<p data-nodeid="1284">Linux 内核从 2002 年 2.4.19 版本开始加入了 Mount Namespace，而直到内核 3.8 版本加入了 User Namespace 才为容器提供了足够的支持功能。</p>
<p data-nodeid="1285">当 Docker 新建一个容器时， 它会创建这六种 Namespace，然后将容器中的进程加入这些 Namespace 之中，使得 Docker 容器中的进程只能看到当前 Namespace 中的系统资源。</p>
<p data-nodeid="1286">正是由于 Docker 使用了 Linux 的这些 Namespace 技术，才实现了 Docker 容器的隔离，可以说没有 Namespace，就没有 Docker 容器。</p>
<h3 data-nodeid="1287">小结</h3>
<p data-nodeid="1288">到此，相信你已经了解了什么是 Namespace。Namespace 是 Linux 内核的一个特性，该特性可以实现在同一主机系统中对进程 ID、主机名、用户 ID、文件名、网络和进程间通信等资源的隔离。Docker 正是结合了这六种 Namespace 的功能，才诞生了 Docker 容器。</p>
<p data-nodeid="1289">最后，试想下，当我们使用 docker run --net=host 命令启动容器时，容器是否和主机共享同一个 Net Namespace？思考后，可以把你的想法写在留言区。</p>

---

### 精选评论

##### *欢：
> net=host 应该与主机共享IP namespace了吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，共享主机网络

##### **4939：
> 如果启动容器的时候使用host模式，那么这个容器将不会获得一个独立的Network Namespace，而是和宿主机共用一个Network Namespace。容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。

##### *冲：
> 创建了一个隔离的挂载点，可是怎么使用呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以使用 nsenter 命令来进入隔离的挂载点，从而模拟容器的实现，实现应用的隔离。

##### *冲：
> mnt namespace,创建了挂载点，那么主机插上优盘后，两个namespace的挂载点都可以看来优盘的内容吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有把优盘内容挂载到非主机的namespace中，默认是无法看见优盘里的内容的

##### **将：
> 添加-net=host之后，容器与主机共享同一个Net Namespace，一般不建议这么设置

##### *庚：
> 还有nsenter可以玩玩

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，nsenter 也可以尝试下

##### **4409：
> 我猜 docker 会创建一个与主机相同的 net namespace，然后独立运行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 确实是共享的主机的 net namespace 哈，可以启动一个 net=host 的容器使用 ls -l /proc/self/ns/ 命令验证下，可以发现 net namespace id 和主机上是一样的

##### *军：
> 选用主机模式应该使用了共同的namespace

##### *晴：
> 这种方式启动容器，并不会创建Net Namespace，而是和宿主共享。所以，Docker不建议使用这种

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你的理解非常正确。确实是共享的主机的 net namespace。

