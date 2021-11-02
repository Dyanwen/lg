<p>本课时主要讲“手动模式构建双 NameNode + Yarn 的 Hadoop 集群”的内容。</p>
<h3>双 NameNode 实现原理与应用架构</h3>
<p>前面铺垫了那么多，现在是时候开始进入 Hadoop 的内容了，学习大数据运维，首先从安装、部署入手，这是大数据运维的基础，本课时将重点讲述如何构建企业级大数据应用平台。</p>
<h4>1. 什么是双 NameNode</h4>
<p>在分布式文件系统 HDFS 中，NameNode 是 master 角色，当 NameNode 出现故障后，整个 HDFS 将不可用，所以保证 NameNode 的稳定性至关重要。在 Hadoop1.x 版本中，HDFS 只支持一个 NameNode，为了保证稳定性，只能靠 SecondaryNameNode 来实现，而 SecondaryNameNode 不能做到热备，而且恢复的数据也不是最新的元数据。基于此，从 Hadoop2.x 版本开始，HDFS 开始支持多个 NameNode，这样不但可以实现 HDFS 的高可用，而且还可以横行扩容 HDFS 的存储规模。</p>
<p>在实际的企业应用中，使用最多的是双 NameNode 架构，也就是一个 NameNode 处于 <strong>Active（活跃） 状态</strong>，另一个 NameNode 处于 <strong>Standby（备用）状态</strong>，通过这种机制，实现 NameNode 的<strong>双机热备高可用功能</strong>。</p>
<h4>2. 双 NameNode 的运行原理</h4>
<p>在高可用的 NameNode 体系结构中，只有 Active 状态的 NameNode 是正常工作的，Standby 状态的 NameNode 处于随时待命状态，它时刻去同步 Active 状态 NameNode 的元数据。一旦 Active 状态的 NameNode 不能工作，可以通过手工或者自动切换方式将 Standby 状态的 NameNode 转变为 Active 状态，保持 NameNode 持续工作。这就是<strong>两个高可靠的 NameNode 的实现机制</strong>。</p>
<p>NameNode 主、备之间的切换可以通过手动或者自动方式来实现，作为线上大数据环境，都是通过自动方式来实现切换的，为保证自动切换，NameNode 使用 ZooKeeper 集群进行仲裁选举。基本的思路是 HDFS 集群中的两个 NameNode 都在 ZooKeeper 中注册，当 Active 状态的 NameNode 出故障时，ZooKeeper 能马上检测到这种情况，它会自动把 Standby 状态切换为 Active 状态。</p>
<p>ZooKeeper（ZK）集群作为一个高可靠系统，能够为集群协作数据提供监控，并将数据的更改随时反馈给客户端。HDFS 的热备功能依赖 ZK 提供的两个特性：错误监测、活动节点选举。HDFS 通过 ZK 实现高可用的机制如下。</p>
<p>每个 NameNode 都会在 ZK 中注册并且持久化一个 session 标识，一旦 NameNode 失效了，那么 session 也将过期，而 ZK 也会通知其他的 NameNode 发起一个失败切换。ZK 提供了一个简单的机制来保证只有一个 NameNode 是活动的，那就是独占锁，如果当前的活动 NameNode 失效了，那么另一个 NameNode 将获取 ZK 中的<strong>独占锁</strong>，表明自己是活动的节点。</p>
<p>ZKFailoverController（ZKFC）是 ZK 集群的客户端，用来监控 NN 的状态信息，每个运行 NameNode 的节点必须要运行一个 ZKFC。ZKFC 提供以下功能：</p>
<ul>
<li>健康检查，ZKFC 定期对本地的 NN 发起 health-check 的命令，如果 NN 正确返回，那么 NN 被认为是 OK 的，否则被认为是失效节点；</li>
<li>session管理，当本地 NN 是健康的时候，ZKFC 将会在 ZK 中持有一个 session，如果本地 NN 又正好是 Active，那么 ZKFC 将持有一个短暂的节点作为锁，一旦本地 NN 失效了，那么这个节点就会被自动删除；</li>
<li>基础选举，如果本地 NN 是健康的，并且 ZKFC 发现没有其他 NN 持有这个独占锁，那么它将试图去获取该锁，一旦成功，那么它就开始执行 Failover，然后变成 Active 状态的 NN 节点；Failover 的过程分两步，首先对之前的 NameNode 执行隔离（如果需要的话），然后将本地 NameNode 切换到 Active 状态。</li>
</ul>
<h4>3. 双 NameNode 架构中元数据一致性如何保证</h4>
<p>聪明的你可能要问了，两个 NameNode 架构之间的元数据是如何共享的呢？</p>
<p>从 Hadoop2.x 版本后，HDFS 采用了一种全新的元数据共享机制，即通过 Quorum Journal Node（JournalNode）集群或者 network File System（NFS）进行数据共享。NFS 是操作系统层面的，而 JournalNode 是 Hadoop 层面的，成熟可靠、使用简单方便，所以，这里我们采用 JournalNode 集群进行元数据共享。</p>
<p>JournalNode 集群以及与 NameNode 之间如何共享元数据，可参照下图所示。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/07/4E/CgqCHl65FrmAJkEYAADp0NN0xGw607.png" alt="01.png"></p>
<p>由图可知，JournalNode 集群可以几乎实时的去 NameNode 上拉取元数据，然后保存元数据到 JournalNode 集群；同时，处于 standby 状态的 NameNode 也会实时的去 JournalNode 集群上同步 JNS 数据，通过这种方式，就实现了两个 NameNode 之间的数据同步。</p>
<p>那么，JournalNode 集群内部是如何实现的呢？</p>
<p>两个 NameNode 为了数据同步，会通过一组称作 JournalNodes 的独立进程进行相互通信。当 Active 状态的 NameNode 元数据有任何修改时，会告知大部分的 JournalNodes 进程。同时，Standby 状态的 NameNode 也会读取 JNs 中的变更信息，并且一直监控 EditLog （事务日志）的变化，并把变化应用于自己的命名空间。Standby 可以确保在集群出错时，元数据状态已经完全同步了。</p>
<p>下图是 JournalNode 集群的内部运行架构图。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/07/4E/CgqCHl65FsmAReOgAACOMp8vfYQ648.png" alt="02.png"></p>
<p>由图可知，JN1、JN2、JN3 等是 JournalNode 集群的节点，QJM（Qurom Journal Manager）的基本原理是用 2N+1 台 JournalNode 存储 EditLog，每次写数据操作有 N/2+1 个节点返回成功，那么本次写操作才算成功，保证数据高可用。当然这个算法所能容忍的是最多有 N 台机器挂掉，如果多于 N 台挂掉，算法就会失效。</p>
<p>ANN 表示处于 Archive 状态的 NameNode，SNN 表示处于 Standbye 状态的 NameNode，QJM 从 ANN 读取数据写入 EditLog 中，然后 SNN 从 EditLog 中读取数据，进而应用到自身。</p>
<h4>4. 双 NameNode 高可用 Hadoop 集群架构</h4>
<p>作为 Hadoop 的第二个版本，Hadoop2.x 最大的变化是 NameNode 可实现高可用，以及计算资源管理器 Yarn。本课时我们将重点介绍下如何构建一个线上高可用的 Hadoop 集群系统，这里有两个重点，一是 NameNode 高可用的构建，二是资源管理器 Yarn 的实现，通过 Yarn 实现真正的分布式计算和多种计算框架的融合。</p>
<p>下图是一个高可用的 Hadoop 集群运行原理图。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/07/55/Ciqc1F65G7aAKwckAADqfdUc2EA969.png" alt="03.png"></p>
<p>此架构主要解决了两个问题，一是 NameNode 元数据同步问题，二是主备 NameNode 切换问题，由图可知，解决主、备 NameNode 元数据同步是通过 JournalNode 集群来完成的，而解决主、备 NameNode 切换可通过 ZooKeeper 来完成。</p>
<p>ZooKeeper 是一个独立的集群，在两个 NameNode 上还需要启动一个 failoverController（zkfc）进程，该进程作为 ZooKeeper 集群的客户端存在，通过 zkfc 可以实现与 ZooKeeper 集群的交互和状态监测。</p>
<h3>双 NameNode + Yarn 构建 HDFS 高可用 Hadoop 集群过程</h3>
<h4>1. 部署前主机、软件功能、磁盘存储规划</h4>
<p>双 NameNode 的 Hadoop 集群环境涉及到的角色有 Namenode、datanode、resourcemanager、nodemanager、historyserver、ZooKeeper、JournalNode 和 zkfc，这些角色可以单独运行在一台服务器上，也可以将某些角色合并在一起运行在一台机器上。</p>
<p>一般情况下，NameNode 服务要独立部署，这样两个 NameNode 就需要两台服务器，而 datanode 和 nodemanager 服务建议部署在一台服务器上，resourcemanager 服务跟 NameNode 类似，也建议独立部署在一台服务器上，而 historyserver 一般和 resourcemanager 服务放在一起。ZooKeeper 和 JournalNode 服务是基于集群架构的，因此至少需要 3 个集群节点，即需要 3 台服务器，不过 ZooKeeper 和 JournalNode 集群可以放在一起，共享 3 台服务器资源。最后，zkfc 是对 NameNode 进行资源仲裁的，所以它必须和 NameNode 服务运行在一起，这样 zkfc 就不需要占用独立的服务器了。</p>
<p>本着节约成本、优化资源、合理配置的原则，下面的部署通过 5 台独立的服务器来实现，操作系统均采用 Centos7.7 版本，每个服务器主机名、IP 地址以及功能角色如下表所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/07/55/CgqCHl65G9OAUprTAADj0vKtWPY235.png" alt="图片1.png"></p>
<p>由表可知，namenodemaster 和 yarnserver 是作为 NameNode 的主、备两个节点，同时 yarnserver 还充当了 ResourceManager 和 JobHistoryServer 的角色。如果服务器资源充足，可以将 ResourceManager 和 JobHistoryServer 服务放在一台独立的机器上。</p>
<p>此外，slave001、slave002 和 slave003 三台主机上部署了 ZooKeeper 集群、JournalNode 集群，还充当了 DataNode 和 NodeManager 的角色。</p>
<p>在软件部署上，每个软件采用的版本如下表所示：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/07/56/CgqCHl65G-SAIyUlAABHEvoc-ZI303.png" alt="图片2.png"></p>
<p>最后，还需要考虑<strong>磁盘存储规划</strong>，HDFS 文件系统的数据块都存储在本地的每个 datanode 节点上。因此，每个 datanode 节点要有大容量的磁盘，磁盘类型可以是普通的机械硬盘，有条件的话 SSD 硬盘最好，单块硬盘推荐 4T 或者 8T，这些硬盘无需做 RAID，单盘使用即可，因为 HDFS 本身已经有了副本容错机制。</p>
<p>在本课时介绍的环境中，我的每个 datanode 节点均有 2 块大容量硬盘用来存储 HDFS 数据块。此外，NameNode 节点所在的主机要存储 HDFS 的元数据信息，这些元数据控制着整个 HDFS 集群中的存储与读写，一旦丢失，那么 HDFS 将丢失数据甚至无法使用，所以保证 NameNode 节点 HDFS 元数据安全性至关重要。</p>
<p>在 NameNode 节点建议配置 4 块大小一样的磁盘，每两块做一个 raid1，共做两组 raid1，然后将元数据镜像存储在这两个 raid1 上。</p>
<h4>2. 自动化安装系统基础环境</h4>
<p>在进行 Hadoop 集群部署之前，先需要对系统的基础环境进行配置， 包含主机名、本地解析 hosts 文件、ansible 管理机到集群节点建立 ssh 信任、系统参数修改（ulimit 资源配置、关闭 selinux）、创建 Hadoop 用户五个方面。 这五个方面可以通过 ansible playbook 脚本自动化完成，下面依次介绍。</p>
<p>（1）建立管理机到集群节点 ssh 无密码登录</p>
<p>在大数据运维中，操作系统安装基本是自动化完成的，系统安装完成后，所有主机的密码也是相同的，为了自动化运维方便，需要将 ansilbe 管理机与 Hadoop 集群所有节点之间建立单向无密码登录权限。</p>
<p>首先，修改 ansible 的配置文件 hosts（本例为 /etc/ansible/hosts）添加主机组信息，内容如下：</p>
<pre><code data-language="java" class="lang-java">[hadoophosts]
<span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.31</span>   hostname=namenodemaster
<span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.41</span>   hostname=yarnserver
<span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.70</span>   hostname=slave001 myid=<span class="hljs-number">1</span>
<span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.103</span>  hostname=slave002 myid=<span class="hljs-number">2</span>
<span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.169</span>  hostname=slave003 myid=<span class="hljs-number">3</span>
</code></pre>
<p>此主机组中，前面是 IP 地址，后面是每个主机的主机名，通过定义 hostname 变量，可实现通过 ansible 自动修改主机名。</p>
<p>接着，创建 /etc/ansible/roles/vars/main.yml 文件，内容如下：</p>
<pre><code data-language="java" class="lang-java">zk1_hostname: <span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.70</span>
zk2_hostname: <span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.103</span>
zk3_hostname: <span class="hljs-number">172.16</span><span class="hljs-number">.213</span><span class="hljs-number">.169</span>
AnsibleDir: /etc/ansible
BigdataDir: /opt/bigdata
hadoopconfigfile: /etc/hadoop
</code></pre>
<p>这里面定义了 6 个角色变量，在后面 playbook 中会用到。最后，编写 playbook 脚本，内容如下：</p>
<pre><code data-language="SQL" class="lang-SQL">- hosts: hadoophosts
  gather_facts: no
  roles:
   - roles
  tasks:
   - name: close ssh yes/no <span class="hljs-keyword">check</span>
     lineinfile: <span class="hljs-keyword">path</span>=/etc/ssh/ssh_config regexp=<span class="hljs-string">'(.*)StrictHostKeyChecking(.*)'</span> line=<span class="hljs-string">"StrictHostKeyCheck
ing no"</span>
   - <span class="hljs-keyword">name</span>: <span class="hljs-keyword">delete</span> /root/.ssh/
     <span class="hljs-keyword">file</span>: <span class="hljs-keyword">path</span>=/root/.ssh/ state=absent
   - <span class="hljs-keyword">name</span>: <span class="hljs-keyword">create</span> .ssh <span class="hljs-keyword">directory</span>
     <span class="hljs-keyword">file</span>: dest=/root/.ssh <span class="hljs-keyword">mode</span>=<span class="hljs-number">0600</span> state=<span class="hljs-keyword">directory</span>
   - <span class="hljs-keyword">name</span>: generating <span class="hljs-keyword">local</span> <span class="hljs-keyword">public</span>/<span class="hljs-keyword">private</span> rsa <span class="hljs-keyword">key</span> pair
     local_action: shell ssh-keygen -t rsa -b <span class="hljs-number">2048</span> -N <span class="hljs-string">''</span> -y -f /root/.ssh/id_rsa
   - <span class="hljs-keyword">name</span>: <span class="hljs-keyword">view</span> id_rsa.pub
     local_action: shell cat /root/.ssh/id_rsa.pub
     <span class="hljs-keyword">register</span>: sshinfo
   - set_fact: sshpub={{sshinfo.stdout}}
   - <span class="hljs-keyword">name</span>: <span class="hljs-keyword">add</span> ssh <span class="hljs-built_in">record</span>
     local_action: shell echo {{sshpub}} &gt; {{AnsibleDir}}/<span class="hljs-keyword">roles</span>/templates/authorized_keys.j2
   - <span class="hljs-keyword">name</span>: copy authorized_keys.j2 <span class="hljs-keyword">to</span> <span class="hljs-keyword">all</span>
     <span class="hljs-keyword">template</span>: src={{AnsibleDir}}/<span class="hljs-keyword">roles</span>/templates/authorized_keys.j2 dest=/root/.ssh/authorized_keys <span class="hljs-keyword">mode</span>
=<span class="hljs-number">0600</span>
     tags:
     - <span class="hljs-keyword">install</span> ssh
</code></pre>
<p>将此playbook脚本命名为sshk.yml，然后在命令行执行如下命令完成ssh单向信任：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># pwd</span>
/etc/ansible
[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  sshk.yml -k</span>
</code></pre>
<p>注意，这里添加了一个“-k”参数，因为现在管理机和 Hadoop 节点之间还没有建立 ssh 信任，所以需要指定此参数手动输入密码，但这个操作仅需执行一次，后面的操作就可以无需输入密码了。</p>
<p>（2）自动修改主机名</p>
<p>紧接上面的 ansible 配置环境，要实现批量自动修改主机名，执行如下 playbook 脚本即可：</p>
<pre><code data-language="SQL" class="lang-SQL">- hosts: hadoophosts
  remote_user: root
  tasks:
  - name: <span class="hljs-keyword">change</span> <span class="hljs-keyword">name</span>
    shell: <span class="hljs-string">"echo {{hostname}} &gt; /etc/hostname"</span>
  - <span class="hljs-keyword">name</span>:
    shell: hostname {{hostname|quote}}
</code></pre>
<p>将此 playbook 脚本命名为 hostname.yml，然后在命令行执行如下命令完成主机名修改：</p>
<pre><code data-language="SQL" class="lang-SQL"> [root@server239 ansible]<span class="hljs-comment"># ansible-playbook  hostname.yml</span>
</code></pre>
<p>（3）自动构建本地解析hosts文件</p>
<p>紧接上面的 ansible 配置环境，要自动构建本地解析 hosts 文件，可通过如下 playbook 脚本实现：</p>
<pre><code data-language="java" class="lang-java">- hosts: hadoophosts
  remote_user: root
  roles:
  - roles
  tasks:
   - name: add localhost
     local_action: shell echo <span class="hljs-string">"127.0.0.1   localhost"</span> &gt; {{AnsibleDir}}/roles/templates/hosts.j2
     run_once: <span class="hljs-keyword">true</span>
   - set_fact: ipaddress={{inventory_hostname}}
   - set_fact: hostname={{hostname}}
   - name: add host record
     local_action: shell echo {{ipaddress}} {{hostname}} &gt;&gt; {{AnsibleDir}}/roles/templates/hosts.j2
   - name: copy hosts.j2 to all host
     template: src={{AnsibleDir}}/roles/templates/hosts.j2 dest=/etc/hosts
</code></pre>
<p>将 playbook 脚本命名为 hosts.yml，然后在命令行执行如下命令，完成构建本地解析 hosts 文件并分发集群每个节点：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># pwd</span>
/etc/ansible
[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  hosts.yml</span>
</code></pre>
<p>（4）自动修改优化系统参数</p>
<p>紧接上面的 ansible 配置环境，系统参数优化主要有关闭 selinux、关闭防火墙 firewalld、iptables、添加 ulimit 资源限制、添加时间同步服务器等，要实现自动优化系统参数，可通过如下 playbook 脚本实现：</p>
<pre><code data-language="java" class="lang-java">- hosts: hadoophosts
  remote_user: root
  gather_facts: <span class="hljs-keyword">false</span>
  tasks:
   - name: selinux disabled
     lineinfile: dest=/etc/selinux/config regexp=<span class="hljs-string">'SELINUX=(.*)'</span> line=<span class="hljs-string">'SELINUX=disabled'</span>
   - name:
     lineinfile: dest=/etc/security/limits.conf line=<span class="hljs-string">"{{item.value}}"</span>
     with_items:
     - {value: <span class="hljs-string">"*         soft    nofile         655360"</span>}
     - {value: <span class="hljs-string">"*         hard    nofile         655360"</span>}
   - name: disabled iptables and firewalld
     shell: systemctl stop firewalld&amp;&amp;systemctl disable firewalld&amp;&amp;iptables –F
   - name: cron ntpdate
     cron: name=ntpdate minute=*/<span class="hljs-number">5</span> user=root job=<span class="hljs-string">"source /etc/profile;/usr/sbin/ntpdate -u 172.16.213.154;/sbin/hwclock -w"</span>
</code></pre>
<p>playbook 脚本依次执行了关闭 selinux、添加用户资源配置、关闭防火墙和增加时间同步服务器，其中 172.16.213.154 是我内网的时间同步服务器，如果没有这个时间服务器，也可以使用外网时间同步时钟，但要<strong>保证机器能够访问互联网</strong>。</p>
<p>将 playbook 脚本命名为 os.yml，然后在命令行执行如下命令，完成优化系统参数：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  os.yml</span>
</code></pre>
<p>（5）自动化批量创建 Hadoop 用户</p>
<p>Hadoop 用户作为集群的管理员用户，需要在每个集群节点进行创建，此用户不需要密码，仅创建一个用户即可，后面所有服务的启动，均是通过 Hadoop 用户来完成的。如下 playbook 脚本可自动完成创建用户的工作，脚本内容如下：</p>
<pre><code data-language="java" class="lang-java">- name: create user
  hosts: hadoophosts
  remote_user: root
  gather_facts: <span class="hljs-keyword">true</span>
  vars:
    user1: hadoop
  tasks:
   - name: start createuser
     user: name=<span class="hljs-string">"{{user1}}"</span>
</code></pre>
<p>将 playbook 脚本命名为 adduser.yml，然后在命令行执行如下命令完成用户创建：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  adduser.yml</span>
</code></pre>
<h4>3. 自动化安装 JDK、ZooKeeper 及 Hadoop</h4>
<p>整个 Hadoop 集群的安装部署需要三个步骤， 即安装 JDK 并设置 Java 环境变量、ZooKeeper 集群的安装部署以及 Hadoop 集群的安装和部署。</p>
<p>软件的部署一般分为安装和配置，若通过自动化工具来进行部署的话，一般是将软件下载好，然后修改配置文件，最后将程序和配置进行打包压缩，这样，一个自动化部署程序就包装好了。将包装好的程序放在 ansible 管理机对应的目录下，进行调用即可。</p>
<p>这里我们将 JDK、ZooKeeper 和 Hadoop 都安装在服务器的 /opt/bigdata 目录下。先从最简单的 JDK 安装部署开始，仍然采用编写 ansible-playbook 脚本的方式进行，编写好的自动化安装 JDK 脚本内容如下：</p>
<pre><code data-language="java" class="lang-java">- hosts: hadoophosts
  remote_user: root
  roles:
  - roles
  tasks:
   - name: mkdir jdk directory
     file: path={{BigdataDir}} state=directory mode=<span class="hljs-number">0755</span>
   - name: copy and unzip jdk
     unarchive: src={{AnsibleDir}}/roles/files/jdk.tar.gz dest={{BigdataDir}}
   - name: chmod bin
     file: dest={{BigdataDir}}/jdk/bin mode=<span class="hljs-number">0755</span> recurse=yes
   - name: set jdk env
     lineinfile: dest=/home/hadoop/.bash_profile line=<span class="hljs-string">"{{item.value}}"</span> state=present
     with_items:
     - {value: <span class="hljs-string">"export JAVA_HOME={{BigdataDir}}/jdk"</span>}
     - {value: <span class="hljs-string">"export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar"</span>}
     - {value: <span class="hljs-string">"export PATH=$JAVA_HOME/bin:$PATH"</span>}
</code></pre>
<p>此脚本仍然使用了角色变量 BigdataDir 和 AnsibleDir，其中，jdk.tar.gz 是包装好的 JDK 安装程序，只需要拷贝到集群每个节点解压即可完成安装。</p>
<p>将 playbook 脚本命名为 jdk.yml，然后在命令行执行如下命令完成 JDK 安装：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  jdk.yml</span>
</code></pre>
<p>接着，编写自动化安装 ZooKeeper 集群的脚本，内容如下：</p>
<pre><code data-language="java" class="lang-java"> - hosts: hadoophosts
  remote_user: root
  roles:
  - roles
  tasks:
   - name: mkdir directory <span class="hljs-keyword">for</span> bigdata data
     file: dest={{BigdataDir}} mode=<span class="hljs-number">0755</span> state=directory
   - name: install zookeeper
     unarchive: src={{AnsibleDir}}/roles/files/zookeeper.tar.gz dest={{BigdataDir}}
   - name: install configuration file <span class="hljs-keyword">for</span> zookeeper
     template: src={{AnsibleDir}}/roles/templates/zoo.cfg.j2 dest={{BigdataDir}}/zookeeper/current/conf/zoo.cfg
   - name: create data and log directory
     file: dest={{BigdataDir}}/zookeeper/current/{{item}} mode=<span class="hljs-number">0755</span> state=directory
     with_items:
     - dataLogDir
     - data
   - name: add myid file
     shell: echo {{ myid }} &gt; {{BigdataDir}}/zookeeper/current/data/myid
   - name: chown hadoop <span class="hljs-keyword">for</span> zk directory
     file: dest={{BigdataDir}}/zookeeper owner=hadoop group=hadoop state=directory recurse=yes
</code></pre>
<p>此脚本引用了一个模板文件 zoo.cfg.j2，它位于管理机上 /etc/ansible/roles/templates/ 路径下，此文件内容如下：</p>
<pre><code data-language="java" class="lang-java">tickTime=<span class="hljs-number">2000</span>
initLimit=<span class="hljs-number">20</span>
syncLimit=<span class="hljs-number">10</span>
dataDir={{BigdataDir}}/zookeeper/current/data
dataLogDir={{BigdataDir}}/zookeeper/current/dataLogDir
clientPort=<span class="hljs-number">2181</span>
quorumListenOnAllIPs=<span class="hljs-keyword">true</span>
server<span class="hljs-number">.1</span>={{zk1_hostname}}:<span class="hljs-number">2888</span>:<span class="hljs-number">3888</span>
server<span class="hljs-number">.2</span>={{zk2_hostname}}:<span class="hljs-number">2888</span>:<span class="hljs-number">3888</span>
server<span class="hljs-number">.3</span>={{zk3_hostname}}:<span class="hljs-number">2888</span>:<span class="hljs-number">3888</span>
</code></pre>
<p>在这个模板文件中，也引用了几个角色变量 BigdataDir、zk1_hostname、zk2_hostname 和 zk3_hostname，这些变量都在 roles 文件夹中 vars 子文件夹下的 main.yml 文件中定义。</p>
<p>将 playbook 脚本命名为 zk.yml，然后在命令行执行如下命令，完成 ZooKeeper 的自动化安装与配置：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  zk.yml</span>
</code></pre>
<p>最后，重点内容来了，即编写自动化安装 Hadoop 脚本，我们采用 Hadoop3.2.1 版本，下载二进制安装包进行，其实就是将包装好的 Hadoop 安装程序打包成 hadoop.tar.gz 这种压缩格式，然后从管理机自动拷贝到集群每个节点，playbook 文件内容如下：</p>
<pre><code data-language="java" class="lang-java"> - hosts: hadoophosts
  remote_user: root
  roles:
  - roles
  tasks:
   - name: create hadoop user
     user: name=hadoop state=present
   - name: mkdir directory <span class="hljs-keyword">for</span> bigdata directory
     file: dest={{BigdataDir}} mode=<span class="hljs-number">0755</span> state=directory
   - name: mkdir directory <span class="hljs-keyword">for</span> bigdata configfiles
     file: dest={{hadoopconfigfile}} mode=<span class="hljs-number">0755</span> state=directory
   - name: install hadoop
     unarchive: src={{AnsibleDir}}/roles/files/hadoop.tar.gz dest={{BigdataDir}}
   - name: chown hadoop configfiles directory
     file: dest={{BigdataDir}}/hadoop owner=hadoop group=hadoop state=directory
   - name: install configuration file <span class="hljs-keyword">for</span> hadoop
     unarchive: src={{AnsibleDir}}/roles/files/conf.tar.gz dest={{hadoopconfigfile}}
   - name: chown hadoop configfiles directory
     file: dest={{hadoopconfigfile}}/conf owner=hadoop group=hadoop state=directory
   - name: set hadoop env
     lineinfile: dest=/home/hadoop/.bash_profile insertafter=<span class="hljs-string">"{{item.position}}"</span> line=<span class="hljs-string">"{{item.value}}"</span> st
ate=present
     with_items:
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_HOME={{BigdataDir}}/hadoop/current"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_MAPRED_HOME=${HADOOP_HOME}"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_COMMON_HOME=${HADOOP_HOME}"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_HDFS_HOME=${HADOOP_HOME}"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_YARN_HOME=${HADOOP_HOME}"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HTTPFS_CATALINA_HOME=${HADOOP_HOME}/share/hadoop/httpfs/tomcat"</span>}
     - {position: EOF, value: <span class="hljs-string">"export CATALINA_BASE=${HTTPFS_CATALINA_HOME}"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HADOOP_CONF_DIR={{hadoopconfigfile}}/conf"</span>}
     - {position: EOF, value: <span class="hljs-string">"export HTTPFS_CONFIG={{hadoopconfigfile}}/conf"</span>}
     - {position: EOF, value: <span class="hljs-string">"export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin"</span>}
   - name: enforce env
     shell: source /home/hadoop/.bash_profile
</code></pre>
<p>将此 playbook 脚本命名为 hadoop.yml，然后在命令行执行如下命令，完成 Hadoop 的自动化安装与配置：</p>
<pre><code data-language="SQL" class="lang-SQL">[root@server239 ansible]<span class="hljs-comment"># ansible-playbook  hadoop.yml</span>
</code></pre>

---

### 精选评论

##### *峰：
> (｡˘•㉨•˘｡)心疼..，写了这么多，点个赞，等学会基层使用再回来看😗

##### **超：
> 请问执行sshk.yml报错是什么原因<div><span style="font-size: 16.0125px;">"ssh-keygen -t rsa -b 2048 -N '' -y -f /root/.ssh/id_rsa", "stderr": "/root/.ssh/id_rsa: No such file or directory"</span><div><br></div></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你有已经存在的密钥文件吗，-f /root/.ssh/id_rsa就是指定存在的私钥文件，如果没有这个私钥文件，那么去掉-y这个参数即可。

##### 李：
> 老师您好，请教个问题，假如不在乎空间损耗的话，datanode的硬盘可以做raid吗？建议用哪种raid？因为之前用过swift，官方建议不做raid，会影响性能，但实际生产中每次坏盘处理起来都很麻烦，踢盘、同步等，拔盘时可能还会宕机😅

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; hdfs内部通过副本，就保障了数据安全性，所以物理磁盘一般无需做raid，如果要做的话，双盘做raid1，多组几组raid1即可，当然多块磁盘做raid5也行，不过这些做的话，多少会影响写磁盘性能。所以一般不用做，因为大量读写的话，无论磁盘是否做raid，磁盘都会故障的，做了raid，也就是直接插拔，hdfs上无需做配置修改而已。

##### **生：
> 目前主要讲解的是基于五个节点的部署方法。但我有个疑问。如果资源足够的情况下。能否选择多namenode节点机制（3/4/5/10?）,节点数量（10/50/100/1000/10000）的时候，选取怎么分配资源会让集群效率最高？还有一个问题。就是在集群资源够用的情况下节点数量（10/50/100/1000/10000）的时候，怎么分配hadoop环境的角色呢？Namenode、datanode、Rresourcemanager、Nodemanager、Historyserver、Zookeeper、JournalNode、zkfc，spark等角色，对于不同资源条件下，各个角色的节点数量分配有什么规则没有？还是说弄成不同的集群？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 管理节点，主要是保证高可用，存储和计算节点是大数据平台的主要节点，要根据计算量来定，不够的话，随时扩展就是了，在某些极端情况下，可能会出现比如namenode节点资源不足情况，这可能是因为元数据过大导致的，此时可以通过namenode的联邦机制实现，其实就是多个高可用的namenode组成一个更大的namenode联邦。

##### **全：
> 老师请问ansible可以安装在这5台服务器中的一个吗，如果不行是不是需要6台服务器

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，ansible是独立的，跟hadoop，ansible主需要安装一个管理端即可。

##### **明：
> 自动化安装 Hadoop 脚本 中的conf.tar.gz是哪个

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个就是haodop的配置文件啊，配置文件你修改完成后，打包成conf.tar.gz即可

##### **一首诗：
> 老师好，文中没有提怎样自动化设置Hdoop集群中各节点的免密登录，安装集群前不是要配置ssh免密登录 ？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 设置免密码是ansible和hadoop各个节点之间的，后面对hadoop服务都是通过ansible来进行管理的。

##### **强：
> 我在官网下载的Hadoop3.2.1版本，解压后没有share/hadoop/httpfs/tomcat 这个目录呀，老师用的那个版本呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，hadoop3下没有这个路径，这个是环境变量配置的那个路径，hadoop3下已经将httpfs集成好了，可以去掉这个配置

##### **军：
> 那jn宕机了怎么办&nbsp;

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; jn是三个节点的集群啊，宕机一个不影响的。

##### **翼：
> shell&nbsp; 命令，提示找不到命令

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是环境变量没有配置正常，找不到相关命令，请检查环境变量配置。

##### **波：
> {{ myid }} 在哪定义了？？？？？？？？？？？？？？？？？？？？？？？<br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; myid在/etc/ansible/hosts中进行定义，已经添加上去了。

