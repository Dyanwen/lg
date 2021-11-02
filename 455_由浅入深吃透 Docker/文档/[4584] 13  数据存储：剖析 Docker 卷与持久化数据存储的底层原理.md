<p data-nodeid="4879" class="">上一课时我介绍了 Docker 网络实现，为我们的容器插上了网线。这一课时我将介绍 Docker 的卷，为我们的容器插上磁盘，实现容器数据的持久化。</p>
<h3 data-nodeid="4880">为什么容器需要持久化存储</h3>
<p data-nodeid="4881">容器按照业务类型，总体可以分为两类：</p>
<ul data-nodeid="4882">
<li data-nodeid="4883">
<p data-nodeid="4884">无状态的（数据不需要被持久化）</p>
</li>
<li data-nodeid="4885">
<p data-nodeid="4886">有状态的（数据需要被持久化）</p>
</li>
</ul>
<p data-nodeid="4887">显然，容器更擅长无状态应用。因为未持久化数据的容器根目录的生命周期与容器的生命周期一样，容器文件系统的本质是在镜像层上面创建的读写层，运行中的容器对任何文件的修改都存在于该读写层，当容器被删除时，容器中的读写层也会随之消失。</p>
<p data-nodeid="4888">虽然容器希望所有的业务都尽量保持无状态，这样容器就可以开箱即用，并且可以任意调度，但实际业务总是有各种需要数据持久化的场景，比如 MySQL、Kafka 等有状态的业务。因此为了解决有状态业务的需求，Docker 提出了卷（Volume）的概念。</p>
<p data-nodeid="4889">什么是卷？卷的本质是文件或者目录，它可以绕过默认的联合文件系统，直接以文件或目录的形式存在于宿主机上。卷的概念不仅解决了数据持久化的问题，还解决了容器间共享数据的问题。使用卷可以将容器内的目录或文件持久化，当容器重启后保证数据不丢失，例如我们可以使用卷将 MySQL 的目录持久化，实现容器重启数据库数据不丢失。</p>
<p data-nodeid="4890">Docker 提供了卷（Volume）的功能，使用<code data-backticks="1" data-nodeid="4983">docker volume</code>命令可以实现对卷的创建、查看和删除等操作。下面我们来详细了解一下这些命令。</p>
<h3 data-nodeid="4891">Docker 卷的操作</h3>
<h4 data-nodeid="4892">创建数据卷</h4>
<p data-nodeid="4893">使用<code data-backticks="1" data-nodeid="4988">docker volume create</code>命令可以创建一个数据卷。</p>
<p data-nodeid="4894">我们使用以下命令创建一个名为 myvolume 的数据卷：</p>
<pre class="lang-shell" data-nodeid="4895"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume create myvolume</span>
</code></pre>
<p data-nodeid="4896">在这里要说明下，默认情况下 ，Docker 创建的数据卷为 local 模式，仅能提供本主机的容器访问。如果想要实现远程访问，需要借助网络存储来实现。Docker 的 local 存储模式并未提供配额管理，因此在生产环境中需要手动维护磁盘存储空间。</p>
<p data-nodeid="4897">除了使用<code data-backticks="1" data-nodeid="4993">docker volume create</code>的方式创建卷，我们还可以在 Docker 启动时使用 -v 的方式指定容器内需要被持久化的路径，Docker 会自动为我们创建卷，并且绑定到容器中，使用命令如下：</p>
<pre class="lang-shell" data-nodeid="4898"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -d --name=nginx-volume -v /usr/share/nginx/html nginx</span>
</code></pre>
<p data-nodeid="4899">使用以上命令，我们启动了一个 nginx 容器，<code data-backticks="1" data-nodeid="4996">-v</code>参数使得 Docker 自动生成一个卷并且绑定到容器的 /usr/share/nginx/html 目录中。<br>
我们可以使用<code data-backticks="1" data-nodeid="5000">docker volume ls</code>命令来查看下主机上的卷：</p>
<pre class="lang-shell" data-nodeid="4900"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume ls</span>
DRIVER&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; VOLUME NAME
local&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;eaa8a223eb61a2091bf5cd5247c1b28ac287450a086d6eee9632d9d1b9f69171
</code></pre>
<p data-nodeid="4901">可以看到，Docker 自动为我们创建了一个名称为随机 ID 的卷。</p>
<h4 data-nodeid="4902">查看数据卷</h4>
<p data-nodeid="4903">已经创建的数据卷可以使用 docker volume ls 命令查看。</p>
<pre class="lang-shell" data-nodeid="4904"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume ls</span>
DRIVER&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; VOLUME NAME
local&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;myvolume
</code></pre>
<p data-nodeid="4905">通过输出可以看到 myvolume 卷已经创建成功。<br>
如果想要查看某个数据卷的详细信息，可以使用<code data-backticks="1" data-nodeid="5008">docker volume inspect</code>命令。例如，我想查看 myvolume 的详细信息，命令如下：</p>
<pre class="lang-java" data-nodeid="4906"><code data-language="java">$ docker volume inspect myvolume
[
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"CreatedAt"</span>: <span class="hljs-string">"2020-09-08T09:10:50Z"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Driver"</span>: <span class="hljs-string">"local"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Labels"</span>: {},
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Mountpoint"</span>: <span class="hljs-string">"/var/lib/docker/volumes/myvolume/_data"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Name"</span>: <span class="hljs-string">"myvolume"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Options"</span>: {},
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"Scope"</span>: <span class="hljs-string">"local"</span>
&nbsp; &nbsp; }
]
</code></pre>
<p data-nodeid="4907">通过<code data-backticks="1" data-nodeid="5011">docker volume inspect</code>命令可以看到卷的创建日期、命令、挂载路径信息。</p>
<h4 data-nodeid="4908">使用数据卷</h4>
<p data-nodeid="4909">使用<code data-backticks="1" data-nodeid="5015">docker volume</code>创建的卷在容器启动时，添加 --mount 参数指定卷的名称即可使用。</p>
<p data-nodeid="4910">这里我们使用上一步创建的卷来启动一个 nginx 容器，并将 /usr/share/nginx/html 目录与卷关联，命令如下：</p>
<pre class="lang-shell" data-nodeid="4911"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -d --name=nginx --mount <span class="hljs-built_in">source</span>=myvolume,target=/usr/share/nginx/html nginx</span>
</code></pre>
<p data-nodeid="4912">使用 Docker 的卷可以实现指定目录的文件持久化，下面我们进入容器中并且修改 index.html 文件内容，命令如下：</p>
<pre class="lang-shell" data-nodeid="4913"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker <span class="hljs-built_in">exec</span> -it&nbsp; nginx bash</span>
<span class="hljs-meta">#</span><span class="bash"><span class="hljs-comment"># 使用以下内容直接替换 /usr/share/nginx/html/index.html 文件 </span></span>
root@719d3c32e211:/# cat &lt;&lt;EOF &gt;/usr/share/nginx/html/index.html
&lt;!DOCTYPE html&gt;
&lt;html&gt;
&lt;head&gt;
&lt;title&gt;Hello, Docker Volume!&lt;/title&gt;
&lt;style&gt;
 &nbsp; &nbsp;body {
 &nbsp; &nbsp; &nbsp; &nbsp;width: 35em;
 &nbsp; &nbsp; &nbsp; &nbsp;margin: 0 auto;
 &nbsp; &nbsp; &nbsp; &nbsp;font-family: Tahoma, Verdana, Arial, sans-serif;
 &nbsp; &nbsp;}
&lt;/style&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;Hello, Docker Volume!&lt;/h1&gt;
&lt;/body&gt;
&lt;/html&gt;
EOF
</code></pre>
<p data-nodeid="4914">此时我们使用<code data-backticks="1" data-nodeid="5020">docker rm</code>命令将运行中的 nginx 容器彻底删除。</p>
<pre class="lang-shell" data-nodeid="4915"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker rm -f nginx</span>
</code></pre>
<p data-nodeid="4916">旧的 nginx 容器删除后，我们再使用<code data-backticks="1" data-nodeid="5023">docker run</code>命令启动一个新的容器，并且挂载 myvolume 卷，命令如下。</p>
<pre class="lang-shell" data-nodeid="4917"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -d --name=nginx --mount <span class="hljs-built_in">source</span>=myvolume,target=/usr/share/nginx/html nginx</span>
</code></pre>
<p data-nodeid="4918">新容器启动后，我们进入容器查看一下 index.html 文件内容：</p>
<pre class="lang-js" data-nodeid="4919"><code data-language="js">$ docker exec -it nginx bash
root@<span class="hljs-number">7</span>ffac645f431:<span class="hljs-regexp">/# cat /u</span>sr/share/nginx/html/index.html
&lt;!DOCTYPE html&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Hello, Docker Volume!<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">style</span>&gt;</span><span class="css">
&nbsp; &nbsp; <span class="hljs-selector-tag">body</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attribute">width</span>: <span class="hljs-number">35em</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attribute">margin</span>: <span class="hljs-number">0</span> auto;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attribute">font-family</span>: Tahoma, Verdana, Arial, sans-serif;
&nbsp; &nbsp; }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">style</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Hello, Docker Volume!<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span>
</code></pre>
<p data-nodeid="4920">可以看到，此时 index.html 文件内容依旧为我们之前写入的内容。可见，使用 Docker 卷后我们的数据并没有随着容器的删除而消失。</p>
<h4 data-nodeid="4921">删除数据卷</h4>
<p data-nodeid="4922">容器的删除并不会自动删除已经创建的数据卷，因此不再使用的数据卷需要我们手动删除，删除的命令为 docker volume rm 。例如，我们想要删除上面创建 myvolume 数据卷，可以使用以下命令：</p>
<pre class="lang-shell" data-nodeid="4923"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume rm myvolume</span>
</code></pre>
<p data-nodeid="4924">这里需要注意，正在被使用中的数据卷无法删除，如果你想要删除正在使用中的数据卷，需要先删除所有关联的容器。</p>
<p data-nodeid="4925">有时候，两个容器之间会有共享数据的需求，很典型的一个场景就是容器内产生的日志需要一个专门的日志采集程序去采集日志内容，例如我需要使用 Filebeat (一种日志采集工具)采集 nginx 容器内的日志，我就需要使用卷来共享一个日志目录，从而使得 Filebeat 和 nginx 容器都可以访问到这个目录，这时就需要用到容器之间共享数据卷的方式。</p>
<h4 data-nodeid="4926">容器与容器之间数据共享</h4>
<p data-nodeid="4927">那如何实现容器与容器之间数据共享呢？下面我举例说明。</p>
<p data-nodeid="4928">首先使用<code data-backticks="1" data-nodeid="5034">docker volume create</code>命令创建一个共享日志的数据卷。</p>
<pre class="lang-shell" data-nodeid="4929"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume create log-vol</span>
</code></pre>
<p data-nodeid="4930">启动一个生产日志的容器（下面用 producer 窗口来表示）：</p>
<pre class="lang-shell" data-nodeid="4931"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run --mount <span class="hljs-built_in">source</span>=log-vol,target=/tmp/<span class="hljs-built_in">log</span> --name=log-producer -it busybox</span>
</code></pre>
<p data-nodeid="4932">然后新打开一个命令行窗口，启动一个消费者容器（下面用 consumer 窗口来表示）：</p>
<pre class="lang-js" data-nodeid="4933"><code data-language="js">docker run -it --name consumer --volumes-<span class="hljs-keyword">from</span> log-producer&nbsp; busybox
</code></pre>
<p data-nodeid="4934">使用<code data-backticks="1" data-nodeid="5039">volumes-from</code>参数可以在启动新的容器时来挂载已经存在的容器的卷，<code data-backticks="1" data-nodeid="5041">volumes-from</code>参数后面跟已经启动的容器名称。<br>
下面我们切换到 producer 窗口，使用以下命令创建一个 mylog.log 文件并写入 "Hello，My log." 的内容：</p>
<pre class="lang-shell" data-nodeid="4935"><code data-language="shell">/ # cat &lt;&lt;EOF &gt;/tmp/log/mylog.log
Hello, My log.
EOF
</code></pre>
<p data-nodeid="4936">然后我们切换到 consumer 窗口，查看一下相关内容：</p>
<pre class="lang-shell" data-nodeid="4937"><code data-language="shell">/ # cat /tmp/log/mylog.log
Hello, My log.
</code></pre>
<p data-nodeid="4938">可以看到我们从 producer 容器写入的文件内容会自动出现在 consumer 容器中，证明我们成功实现了两个容器间的数据共享。</p>
<p data-nodeid="4939">总结一下，我们首先使用 docker volume create 命令创建了 log-vol 卷来作为共享目录，log-producer 容器向该卷写入数据，consumer 容器从该卷读取数据。这就像主机上的两个进程，一个向主机目录写数据，一个从主机目录读数据，利用主机的目录，实现了容器之间的数据共享。</p>
<h4 data-nodeid="4940">主机与容器之间数据共享</h4>
<p data-nodeid="4941">Docker 卷的目录默认在 /var/lib/docker 下，当我们想把主机的其他目录映射到容器内时，就需要用到主机与容器之间数据共享的方式了，例如我想把 MySQL 容器中的 /var/lib/mysql 目录映射到主机的 /var/lib/mysql 目录中，我们就可以使用主机与容器之间数据共享的方式来实现。</p>
<p data-nodeid="4942">要实现主机与容器之间数据共享，其实很简单，只需要我们在启动容器的时候添加<code data-backticks="1" data-nodeid="5055">-v</code>参数即可, 使用格式为：<code data-backticks="1" data-nodeid="5057">-v HOST_PATH:CONTIANAER_PATH</code>。</p>
<p data-nodeid="4943">例如，我想挂载主机的 /data 目录到容器中的 /usr/local/data 中，可以使用以下命令来启动容器：</p>
<pre class="lang-shell" data-nodeid="4944"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker run -v /data:/usr/<span class="hljs-built_in">local</span>/data -it busybox</span>
</code></pre>
<p data-nodeid="4945">容器启动后，便可以在容器内的 /usr/local/data 访问到主机 /data 目录的内容了，并且容器重启后，/data 目录下的数据也不会丢失。</p>
<p data-nodeid="4946">以上就是 Docker 卷的操作，关键命令我帮你总结如下：</p>
<p data-nodeid="4947"><img src="https://s0.lgstatic.com/i/image/M00/5C/50/Ciqc1F-BW1SAQEkaAACOwJuMTHI950.png" alt="Lark20201010-145710.png" data-nodeid="5064"></p>
<p data-nodeid="4948">那你了解完卷的相关操作后，你有没有想过 Docker 的卷是怎么实现的呢？接下来我们就看看卷的实现原理。</p>
<h3 data-nodeid="4949">Docker 卷的实现原理</h3>
<p data-nodeid="4950">在了解 Docker 卷的原理之前，我们先来回顾一下镜像和容器的文件系统原理。</p>
<blockquote data-nodeid="4951">
<p data-nodeid="4952"><strong data-nodeid="5072">镜像和容器的文件系统原理：</strong> 镜像是由多层文件系统组成的，当我们想要启动一个容器时，Docker 会在镜像上层创建一个可读写层，容器中的文件都工作在这个读写层中，当容器删除时，与容器相关的工作文件将全部丢失。</p>
</blockquote>
<p data-nodeid="4953">Docker 容器的文件系统不是一个真正的文件系统，而是通过联合文件系统实现的一个伪文件系统，而 Docker 卷则是直接利用主机的某个文件或者目录，它可以绕过联合文件系统，直接挂载主机上的文件或目录到容器中，这就是它的工作原理。</p>
<p data-nodeid="4954">下面，我们通过一个实例来说明卷的工作原理。首先，我们创建一个名称为 volume-data 的卷：</p>
<pre class="lang-shell" data-nodeid="4955"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> docker volume create volume-data</span>
</code></pre>
<p data-nodeid="4956">我们使用 ls 命令查看一下 /var/lib/docker/volumes 目录下的内容：</p>
<pre class="lang-java" data-nodeid="4957"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/volumes
drwxr-xr-x. <span class="hljs-number">3</span> root root&nbsp; &nbsp; <span class="hljs-number">19</span> Sep&nbsp; <span class="hljs-number">8</span> <span class="hljs-number">10</span>:<span class="hljs-number">59</span> volume-data
</code></pre>
<p data-nodeid="4958">然后再看下 volume-data 目录下有什么内容：</p>
<pre class="lang-java" data-nodeid="4959"><code data-language="java">$ sudo ls -l /<span class="hljs-keyword">var</span>/lib/docker/volumes/volume-data
total <span class="hljs-number">0</span>
drwxr-xr-x. <span class="hljs-number">2</span> root root <span class="hljs-number">6</span> Sep&nbsp; <span class="hljs-number">8</span> <span class="hljs-number">10</span>:<span class="hljs-number">59</span> _data
</code></pre>
<p data-nodeid="4960">可以看到我们创建的卷出现在了 /var/lib/docker/volumes 目录下，并且 volume-data 目录下还创建了一个 _data 目录。</p>
<p data-nodeid="4961">实际上，在我们创建 Docker 卷时，Docker 会把卷的数据全部放在 /var/lib/docker/volumes 目录下，并且在每个对应的卷的目录下创建一个 _data 目录，然后把 _data 目录绑定到容器中。因此我们在容器中挂载卷的目录下操作文件，实际上是在操作主机上的 _data 目录。为了证实我的说法，我们来实际演示下。</p>
<p data-nodeid="4962">首先，我们启动一个容器，并且绑定 volume-data 卷到容器内的 /data 目录下：</p>
<pre class="lang-shell" data-nodeid="4963"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> &nbsp;docker run -it --mount <span class="hljs-built_in">source</span>=volume-data,target=/data busybox</span>
/ #
</code></pre>
<p data-nodeid="4964">我们进入到容器的 /data 目录，创建一个 data.log 文件:</p>
<pre class="lang-shell" data-nodeid="4965"><code data-language="shell">/ # cd data/
/data # touch data.log
</code></pre>
<p data-nodeid="4966">然后我们新打开一个命令行窗口，查看一下主机上的文件内容：</p>
<pre class="lang-shell" data-nodeid="4967"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> &nbsp;sudo ls -l /var/lib/docker/volumes/volume-data/_data</span>
total 0
-rw-r--r--. 1 root root 0 Sep&nbsp; 8 11:15 data.log
</code></pre>
<p data-nodeid="4968">可以看到主机上的 _data 目录下也出现了 data.log 文件。这说明，在容器内操作卷挂载的目录就是直接操作主机上的 _data 目录，符合我上面的说法。</p>
<p data-nodeid="4969">综上，<strong data-nodeid="5101">Docker 卷的实现原理是在主机的 /var/lib/docker/volumes 目录下，根据卷的名称创建相应的目录，然后在每个卷的目录下创建 _data 目录，在容器启动时如果使用 --mount 参数，Docker 会把主机上的目录直接映射到容器的指定目录下，实现数据持久化。</strong></p>
<h3 data-nodeid="4970">结语</h3>
<p data-nodeid="4971">到此，相信你已经了解了 Docker 使用卷做持久化存储的必要性，也了解 Docker 卷的常用操作，并且对卷的实现原理也有了较清晰的认识。</p>
<p data-nodeid="4972">那么，你知道 Docker 如何使用卷来挂载 NFS 类型的持久化存储到容器内吗？思考后，把你的想法写在留言区。</p>
<p data-nodeid="4973" class="te-preview-highlight">下一课时，我将讲解 Docker 文件存储驱动 AUFS 的系统原理及生产环境的最佳配置。</p>

---

### 精选评论

##### **9792：
> -V参数就是映射主机目录到容器中，这和通过卷的方式有什么区别呢？感觉差不多呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; -v 参数可以挂载主机任意目录，卷的 Docker 对目录的封装，只需要指定卷的名称即可使用

##### **用户9307：
> 我觉得--mount和-v区别是，--mount在卷被删除之后，volumes目录下的对应目录也被删除了，而-v是不会被删除的。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是一个区别， volumes 是一个存储概念的封装，不仅支持 local 还支持 nfs 等网络存储

##### *冲：
> docker run -v /data:/usr/local/data -it busybox感觉这个命令就超级好,为啥还要 创建卷,将卷映射到主机目录?感觉后者好麻烦呀

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 卷作为一种统一封装，不仅可以支持本地目录，也可以支持 NFS 等网络文件系统。如果你的需求仅仅需要本地目录，使用 -v 挂载也完全可以的。

##### **琴：
> 老师请问下为什么使用$ docker volume create volume-data在容器里创建的卷没有通过挂载就能在主机上看到呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; docker volume 的实现实际上就是主机上的特定目录，通常在/var/lib/docker/volumes 目录下

##### *冲：
> 100个容器100个mysql,磁盘空间满了，我怎么知道是哪个容器的mysql满了？并且怎么知道该容器的mysql数据库文件在什么地方？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不同的 mysql 使用不同的 volume 挂载或目录挂载即可，通过监控这些数据目录就可以定位到磁盘满的容器。

##### **7547：
> 问：Docker 如何使用卷来挂载 NFS 类型的持久化存储到容器内答：在主机上挂载NFS共享，并将其作为主机卷传递到容器中mount server:/dir /path/to/mount/pointdocker run -v /path/to/mount/point:/path/to/mount/point

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，很完美的自问自答

##### **卫：
> docker的-v参数和--mount参数有什么区别吗？我测试的时候因为版本号是1.13的，所以不支持--mount，我就直接用-v指定myvolume卷名到容器挂载目录了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有本质区别，mount 使用已经创建的卷来挂载到主机中，-v 是把指定的目录直接映射到主机中。

##### **4562：
> 容器间共享是不是也可以让多个容器都挂载同一个主机目录就行了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，容器间共享目录可以挂载主机相同的目录

##### **平：
> 这个在我的centos7.6.1810 ,Docker version 20.10.2 上运行异常docker run -it --name consumer --volumes-from log-producer busyboxdocker: Error response from daemon: No such container: log-producer.命令行得参数跟顺序有关系嘛？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 需要先执行上一步的命令，docker run --mount source=log-vol,target=/tmp/log --name=log-producer -it busybox  ，启动 log-producer 容器

##### **升：
> 打卡，写得很好，多谢啦！！！

