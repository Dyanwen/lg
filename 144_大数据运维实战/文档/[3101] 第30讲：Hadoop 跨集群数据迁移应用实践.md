<p data-nodeid="187761">我们的 Hadoop 大数据平台已经运行多年，使用的版本是 CDH 5.8，平台上的各个组件（HDFS、Yarn、Hive、Spark）也都是基于这个版本的，但随着对 Hadoop 平台的深入使用，部分组件版本过低，有些新功能无法使用，这迫使我们必须要升级到新的版本。</p>
<p data-nodeid="187762">CDH 5.8 版本的 Hadoop 是基于 Hadoop 2.x 的，此次升级计划从 Hadoop 2.x 版本升级到 3.x 版本，相关依赖组件也一并升级。</p>
<p data-nodeid="187763">大数据平台升级方法常用的有两种：一种是在现有平台基础上升级；第二种是重新构建一套大数据平台，然后将老平台的数据、业务等迁移到新的平台。</p>
<p data-nodeid="187764">由于我们的老 Hadoop 平台硬件也不太给力，在现有平台基础上升级意义不大，所以选择了第二种方法，全新部署基于 Hadoop 3.x 的大数据平台，先将数据导入新的平台，再切换业务到新的平台下，所有业务切换完成后关闭老平台。</p>
<p data-nodeid="188786" class="">所以，本课时重点介绍<strong data-nodeid="188792">如何完成新、旧平台下数据的迁移</strong> 。</p>

<h3 data-nodeid="190624" class="">数据迁移前准备工作</h3>





<p data-nodeid="187767">在迁移数据之前，首先需要安装部署新的大数据平台，接着才能进行数据迁移。那么在迁移之前，需要考虑如下几个问题：</p>
<ul data-nodeid="187768">
<li data-nodeid="187769">
<p data-nodeid="187770">需要迁移的数据量有多少？</p>
</li>
<li data-nodeid="187771">
<p data-nodeid="187772">有什么好的工具可以批量、快速地将数据迁移过去？</p>
</li>
<li data-nodeid="187773">
<p data-nodeid="187774">在迁移数据过程中，要保障不影响老平台的正常运行，可使用的带宽最多有多少？</p>
</li>
<li data-nodeid="187775">
<p data-nodeid="187776">数据迁移完成后，如何校验数据一致性？</p>
</li>
<li data-nodeid="187777">
<p data-nodeid="187778">新平台下 HDFS 数据文件权限，如何和老平台保持一致？</p>
</li>
</ul>
<p data-nodeid="187779">我们这个数据迁移，涉及的数据有 1P 左右，这么大的数据量通过传统的 SCP、FTP 等方式复制肯定不现实，因此需要一个更加高效的数据传输方案；另外，数据每天都在增加，迁移过程也不可能几个小时完成，因此还要考虑数据的增量传输方式。此外，在数据传输过程中，会占用新、老平台大量带宽，如何高效传输数据而不影响老平台的使用，也是个难题。</p>
<p data-nodeid="187780">带着这些问题，我将从数据迁移流程、迁移时间、迁移工具三个方面，来介绍如何制定数据迁移方案。</p>
<h3 data-nodeid="191425" class="">数据迁移流程</h3>


<p data-nodeid="187782">由于在老平台下每天都会生成新的数据，并且要保证有足够的带宽给业务使用，所以需要根据各个业务特点制定迁移步骤，可以通过按业务、分目录、分批迁移的方法来进行。</p>
<p data-nodeid="193024" class="">首先，可以迁移跟业务无关的<strong data-nodeid="193030">冷数据</strong>，这些数据不会有新增，可以马上迁移到新的 HDFS 集群上。</p>


<p data-nodeid="192623" class="">其次，可以迁移历史<strong data-nodeid="192629">老数据</strong>，这些数据量通常会比较大，可以按照年、月等形式，分批、分次进行传输。</p>


<p data-nodeid="193826" class="">然后，可以迁移<strong data-nodeid="193832">业务量较小、数据量变动不多</strong>的这部分数据，比如数据每天变动一次，那么可以在当天将数据迁移完毕，然后将业务一起迁移到新的集群中。</p>


<p data-nodeid="194628" class="">最后，要迁移的是<strong data-nodeid="194634">数据量变化较大、时刻都在产生新数据的业务</strong>，对待这类应用，可以先将大部分数据传输到新的集群；然后将业务同时在新集群上运行一份，这样，新、旧两套集群都在同时运行这个业务了；确定业务在新集群上运行正常后，在旧集群上将新、旧集群并行这段时间的差异数据复制到新的集群；最后，停掉老集群业务即可，这样就实现了新、旧集群的无缝切换。</p>


<p data-nodeid="187787">为了不影响老集群业务的正常运行，在复制数据时，可以选择业务不繁忙的时段（比如凌晨）集中复制数据。</p>
<h3 data-nodeid="196214" class="">数据迁移时间</h3>




<p data-nodeid="187789">如果要迁移的数据较多，可采用分批次、逐个迁移的方式。</p>
<p data-nodeid="187790">对于历史数据，可设置一个较小的传输带宽持续进行传输；而对于每天都有新增的数据，为了减小对线上业务的影响，尽量选择老集群低负载运行的时间段来进行数据迁移，比如可以写个定时脚本，在凌晨或者集群负载最低的时候开启数据迁移任务。</p>
<p data-nodeid="187791">由于新、旧集群同时运行，所以对于数据的迁移完成时间没有严格要求，但选择数据迁移的时机、迁移的顺序却至关重要，需要严格考虑。</p>
<h3 data-nodeid="197771" class="">数据迁移工具 Distcp</h3>




<p data-nodeid="198155" class="">数据迁移工具推荐使用 Hadoop 自带的数据迁移工具 <strong data-nodeid="198161">Distcp</strong>，此工具主要用于 Hadoop 集群内部和集群之间的数据复制。它使用 Map/Reduce 实现文件分发、错误处理和恢复，它把文件和目录的列表作为 map 任务的输入，通过这种模式，可以实现数据的分布式复制，极大提高了数据传输的速率。</p>

<p data-nodeid="187794">Distcp 使用虽然方便，但是 Hadoop 新、旧集群的版本差别太大，比如老 Hadoop 为 Hadoop 1.x，新 Hadoop 为 Hadoop 3.x 时，那么它们之间传输数据，可能会出现问题，这个需要注意。</p>
<p data-nodeid="187795">在完成了所有准备工作后，可先尝试进行小批量的迁移，例如，可以先进行 100G 的数据迁移、500G 的数据迁移、1T 的数据迁移，以此来评估数据迁移速率和迁移过程中可能遇到的问题。</p>
<p data-nodeid="198544" class=""><strong data-nodeid="198549">Distcp 命令的本质是一个 MapReduce 任务</strong>，它只有 Map 阶段，没有 Reduce 阶段，具备分布式执行的特性。它在 Map 任务中从老集群读取数据，然后写入新集群，以此来完成数据迁移。</p>

<p data-nodeid="187797" class="">此工具使用很简单，只需要执行如下命令即可开始数据复制：</p>
<p data-nodeid="198932" class=""><strong data-nodeid="198936">Hadoop Distcp  源 HDFS 文件路径目标 HDFS 文件路径。</strong></p>

<p data-nodeid="187799">例如，下面的操作是将 bigdata1 集群上的 /data 目录复制到 bigdata2 集群中：</p>
<pre class="lang-java" data-nodeid="200468"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> ~]$ hadoop distcp hdfs:<span class="hljs-comment">//bigdata1:8020/data hdfs://bigdata2:8020/ </span>
</code></pre>




<p data-nodeid="187801">此工具常用的几个选项如下：</p>
<ul data-nodeid="201261">
<li data-nodeid="201262">
<p data-nodeid="201263">-m &lt;num_maps&gt;，用来指定复制数据时 map 的数目，请注意，并不是 map 数越多吞吐量越大；</p>
</li>
<li data-nodeid="201264">
<p data-nodeid="201265">-i，表示忽略失败；</p>
</li>
<li data-nodeid="201266">
<p data-nodeid="201267">-log <code data-backticks="1" data-nodeid="201284">&lt;logdir&gt;</code>，用来开启日志记录功能，到指定的文件中；</p>
</li>
<li data-nodeid="201268">
<p data-nodeid="201269">-update，表示当目标 HDFS 上的文件不存在或文件不一致时，才会从源集群复制；</p>
</li>
<li data-nodeid="201270">
<p data-nodeid="201271">-overwrite，表示覆盖目标 HDFS 上相同的文件；</p>
</li>
<li data-nodeid="201272">
<p data-nodeid="201273">-filter，表示过滤不需要复制的文件；</p>
</li>
<li data-nodeid="201274">
<p data-nodeid="201275">-delete，表示以源 HDFS 为准，也就是如果目标 HDFS 上存在此文件，但源 HDFS 上不存在此文件时，则删除这个文件。</p>
</li>
</ul>
<p data-nodeid="201276" class=""><strong data-nodeid="201294">注意</strong>：当使用 Distcp 工具时，可以在老 Dadoop 集群上执行，也可以在新 Hadoop 上执行，如果在老 Hadoop 集群，可执行如下命令：</p>



<pre class="lang-java" data-nodeid="201677"><code data-language="java">[hadoop<span class="hljs-meta">@namenodemaster</span> ~]$ hadoop distcp /logs hdfs:<span class="hljs-comment">//172.16.218.29:8020/data/hadoop </span>
</code></pre>

<p data-nodeid="187819">这个操作是在老集群上通过 Hadoop 用户，将老集群的 HDFS 上 /logs 目录，迁移到新集群的 /data/hadoop 路径下。注意新集群的 NameNode 节点 IP 是 172.16.218.29，8020 是默认端口。</p>
<p data-nodeid="187820">由于此命令是通过老集群的 Hadoop 用户执行的，因此，要确保新集群中 /data/hadoop 目录有 Hadoop 用户的读写权限，否则执行会报错。</p>
<p data-nodeid="202434">此命令执行后，可以在老集群的 Yarn 界面下发现有一个 Distcp 任务在运行，如下图所示：</p>
<p data-nodeid="202435" class=""><img src="https://s0.lgstatic.com/i/image/M00/3E/C0/CgqCHl8tHTiAcSEkAAB_dGYJO0c193.png" alt="image1 (1).png" data-nodeid="202443"></p>


<p data-nodeid="187823">从图中可以看出，此 Distcp 其实执行的就是一个 MAPREDUCE 任务。很明显，如果在老集群执行 Distcp 命令，就会占据老集群的 yarn 资源，这样会影响老集群下业务的正常运行，因此， 在实际迁移环境中，一般都在新的集群下执行 Distcp 命令，执行方式如下：</p>
<pre class="lang-java" data-nodeid="202834"><code data-language="java">[hadoop<span class="hljs-meta">@newmaster</span> ~]$ hadoop distcp hdfs:<span class="hljs-comment">//172.16.216.99:8020/logs  /data/hadoop </span>
</code></pre>

<p data-nodeid="187825">此命令执行后，也会在新集群 Yarn 中运行一个 MAPREDUCE 任务，在使用 Distcp 时，必须注意以下事项：</p>
<ul data-nodeid="187826">
<li data-nodeid="187827">
<p data-nodeid="187828">数据源集群的所有节点，必须和目标集群所有节点能互相通信；</p>
</li>
<li data-nodeid="187829">
<p data-nodeid="187830">指定的目标 HDFS 路径必须存在，否则报错；</p>
</li>
<li data-nodeid="187831">
<p data-nodeid="187832">Distcp 命令中，可以使用主机名，也可以使用 IP 地址。</p>
</li>
</ul>
<p data-nodeid="204428" class="">在使用 Distcp 时，还有很多小技巧，例如，为了传输速度，可以使用 Distcp 的 “-m <code data-backticks="1" data-nodeid="204430">&lt;arg&gt;</code>” 参数来设置 map 任务的最大数量（默认 20），以提高并发性。<strong data-nodeid="204438">但需要注意</strong>：这里要结合最大网络传输速率来设置；如果对传输带宽要限制，还可以使用“-bandwidth <code data-backticks="1" data-nodeid="204436">&lt;arg&gt;</code>”参数来控制单个 Map 任务的最大带宽，单位是 MB。</p>




<p data-nodeid="187834">在数据迁移过程中，最主要的就是文件的权限，如果传输过程中权限丢失，那么重新配置权限将非常麻烦。为此，Distcp 也提供了一个参数 -p 来在新集群里保留文件的权限、属性等信息（主要是复制、块大小、用户、组、权限、校验和类型、ACL、XATTR、时间戳）。而如果要增量迁移数据，则可以使用 Distcp 的 update 参数，这样就会忽略新集群中已经存在的文件，提升数据迁移效率。</p>
<h3 data-nodeid="187835">小结</h3>
<p data-nodeid="187836">本课时主要介绍了在 Hadoop 版本升级过程中，如何在两个 Hadoop 集群环境下高效、快速地进行迁移数据，重点介绍了 Distcp 这个工具的使用，此工具主要用于在两个 Hadoop 集群之间进行数据复制。掌握了此工具的使用，不但可以做好数据的快速迁移，还可以通过它对 HDFS 集群数据进行跨机房的异地备份，最大限度地保证数据的安全性。</p>
<p data-nodeid="187837">至此，这门课程到这里就讲完了，如果你觉得课程不错，从中有所收获的话，不要忘了推荐给身边的朋友哦。前路漫漫，一起加油。</p>

---

### 精选评论

##### **强：
> 标题不一样了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是这样的，这一课时主要是介绍 Hadoop 跨集群数据迁移应用实践，而扩缩容、数据倾斜处理方式前面的课时中都陆续介绍过了~标题名称已经修改了，感谢反馈，祝学习愉快

