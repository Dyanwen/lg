<p data-nodeid="1229" class="">前几天在咱们的社群里看到有同学在讨论，说面试的时候被问到容器和镜像的区别，有同学回答说没什么区别，也许是在开玩笑，不过这两者的区别很大。今天，我们就来看看容器的相关知识，比如什么是容器？容器的生命周期，以及容器常用的操作命令。学完之后你可以对比下与镜像的区别。</p>
<h3 data-nodeid="1230">容器（Container）是什么？</h3>
<p data-nodeid="1231">容器是基于镜像创建的可运行实例，并且单独存在，一个镜像可以创建出多个容器。运行容器化环境时，实际上是在容器内部创建该文件系统的读写副本。 这将添加一个容器层，该层允许修改镜像的整个副本。如图 1 所示。</p>
<p data-nodeid="1232"><img src="https://s0.lgstatic.com/i/image/M00/4C/D1/CgqCHl9YmlSAGgF0AABXUH--rM4624.png" alt="image.png" data-nodeid="1340"></p>
<div data-nodeid="1233"><p style="text-align:center">图1 容器组成</p></div>
<p data-nodeid="1234">了解完容器是什么，接下来我们聊一聊容器的生命周期。</p>
<h3 data-nodeid="1235">容器的生命周期</h3>
<p data-nodeid="1236">容器的生命周期是容器可能处于的状态，容器的生命周期分为 5 种。</p>
<ol data-nodeid="1237">
<li data-nodeid="1238">
<p data-nodeid="1239">created：初建状态</p>
</li>
<li data-nodeid="1240">
<p data-nodeid="1241">running：运行状态</p>
</li>
<li data-nodeid="1242">
<p data-nodeid="1243">stopped：停止状态</p>
</li>
<li data-nodeid="1244">
<p data-nodeid="1245">paused： 暂停状态</p>
</li>
<li data-nodeid="1246">
<p data-nodeid="1247">deleted：删除状态</p>
</li>
</ol>
<p data-nodeid="2150">各生命周期之前的转换关系如图所示：</p>
<p data-nodeid="3393" class=""><img src="https://s0.lgstatic.com/i/image/M00/55/BF/CgqCHl9qxcuANmQGAADHS_nfwJE810.png" alt="Lark20200923-114857.png" data-nodeid="3397"></p>
<div data-nodeid="3394"><p style="text-align:center">图2 容器的生命周期</p></div>








<p data-nodeid="1251">通过<code data-backticks="1" data-nodeid="1354">docker create</code>命令生成的容器状态为初建状态，初建状态通过<code data-backticks="1" data-nodeid="1356">docker start</code>命令可以转化为运行状态，运行状态的容器可以通过<code data-backticks="1" data-nodeid="1358">docker stop</code>命令转化为停止状态，处于停止状态的容器可以通过<code data-backticks="1" data-nodeid="1360">docker start</code>转化为运行状态，运行状态的容器也可以通过<code data-backticks="1" data-nodeid="1362">docker pause</code>命令转化为暂停状态，处于暂停状态的容器可以通过<code data-backticks="1" data-nodeid="1364">docker unpause</code>转化为运行状态 。处于初建状态、运行状态、停止状态、暂停状态的容器都可以直接删除。</p>
<p data-nodeid="1252">下面我通过实际操作和命令来讲解容器各生命周期间的转换关系。</p>
<h3 data-nodeid="1253">容器的操作</h3>
<p data-nodeid="1254">容器的操作可以分为五个步骤：创建并启动容器、终止容器、进入容器、删除容器、导入和导出容器。下面我们逐一来看。</p>
<h4 data-nodeid="1255">（1）创建并启动容器</h4>
<p data-nodeid="1256">容器十分轻量，用户可以随时创建和删除它。我们可以使用<code data-backticks="1" data-nodeid="1371">docker create</code>命令来创建容器，例如：</p>
<pre class="lang-java" data-nodeid="1257"><code data-language="java">$ docker create -it --name=busybox busybox
Unable to find image <span class="hljs-string">'busybox:latest'</span> locally
latest: Pulling from library/busybox
<span class="hljs-number">61</span>c5ed1cbdf8: Pull complete
Digest: sha256:<span class="hljs-number">4f</span>47c01fa91355af2865ac10fef5bf6ec9c7f42ad2321377c21e844427972977
Status: Downloaded newer image <span class="hljs-keyword">for</span> busybox:latest
<span class="hljs-number">2</span>c2e919c2d6dad1f1712c65b3b8425ea656050bd5a0b4722f8b01526d5959ec6
$ docker ps -a| grep busybox
<span class="hljs-number">2</span>c2e919c2d6d&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"sh"</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">34</span> seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Created&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox
</code></pre>
<p data-nodeid="1258">如果使用<code data-backticks="1" data-nodeid="1374">docker create</code>命令创建的容器处于停止状态，我们可以使用<code data-backticks="1" data-nodeid="1376">docker start</code>命令来启动它，如下所示。</p>
<pre class="lang-java" data-nodeid="1259"><code data-language="java">$ docker start busybox
$ docker ps
CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES
d6f3d364fad3&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"sh"</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">16</span> seconds ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up <span class="hljs-number">8</span> seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;busybox
</code></pre>
<p data-nodeid="1260">这时候我们可以看到容器已经处于启动状态了。<br>
容器启动有两种方式：</p>
<ol data-nodeid="1261">
<li data-nodeid="1262">
<p data-nodeid="1263">使用<code data-backticks="1" data-nodeid="1382">docker start</code>命令基于已经创建好的容器直接启动 。</p>
</li>
<li data-nodeid="1264">
<p data-nodeid="1265">使用<code data-backticks="1" data-nodeid="1385">docker run</code>命令直接基于镜像新建一个容器并启动，相当于先执行<code data-backticks="1" data-nodeid="1387">docker create</code>命令从镜像创建容器，然后再执行<code data-backticks="1" data-nodeid="1389">docker start</code>命令启动容器。</p>
</li>
</ol>
<p data-nodeid="1266">使用<code data-backticks="1" data-nodeid="1392">docker run</code>的命令如下:</p>
<pre class="lang-java" data-nodeid="1267"><code data-language="java">$ docker run -it --name=busybox busybox
</code></pre>
<p data-nodeid="1268">当使用<code data-backticks="1" data-nodeid="1395">docker run</code>创建并启动容器时，Docker 后台执行的流程为：</p>
<ul data-nodeid="1269">
<li data-nodeid="1270">
<p data-nodeid="1271">Docker 会检查本地是否存在 busybox 镜像，如果镜像不存在则从 Docker Hub 拉取 busybox 镜像；</p>
</li>
<li data-nodeid="1272">
<p data-nodeid="1273">使用 busybox 镜像创建并启动一个容器；</p>
</li>
<li data-nodeid="1274">
<p data-nodeid="1275">分配文件系统，并且在镜像只读层外创建一个读写层；</p>
</li>
<li data-nodeid="1276">
<p data-nodeid="1277">从 Docker IP 池中分配一个 IP 给容器；</p>
</li>
<li data-nodeid="1278">
<p data-nodeid="1279">执行用户的启动命令运行镜像。</p>
</li>
</ul>
<p data-nodeid="1280">上述命令中， -t 参数的作用是分配一个伪终端，-i 参数则可以终端的 STDIN 打开，同时使用 -it 参数可以让我们进入交互模式。 在交互模式下，用户可以通过所创建的终端来输入命令，例如：</p>
<pre class="lang-java" data-nodeid="1281"><code data-language="java">$ ps aux
PID&nbsp;&nbsp; USER&nbsp;&nbsp;&nbsp;&nbsp; TIME&nbsp; COMMAND
&nbsp;&nbsp;&nbsp; <span class="hljs-number">1</span> root&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;<span class="hljs-number">0</span>:<span class="hljs-number">00</span> sh
&nbsp;&nbsp;&nbsp; <span class="hljs-number">6</span> root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">0</span>:<span class="hljs-number">00</span> ps aux
</code></pre>
<p data-nodeid="1282">我们可以看到容器的 1 号进程为 sh 命令，在容器内部并不能看到主机上的进程信息，因为容器内部和主机是完全隔离的。同时由于 sh 是 1 号进程，意味着如果通过 exit 退出 sh，那么容器也会退出。所以对于容器来说，<strong data-nodeid="1407">杀死容器中的主进程，则容器也会被杀死。</strong></p>
<h4 data-nodeid="1283">（2）终止容器</h4>
<p data-nodeid="1284">容器启动后，如果我们想停止运行中的容器，可以使用<code data-backticks="1" data-nodeid="1410">docker stop</code>命令。命令格式为 docker stop [-t|--time[=10]]。该命令首先会向运行中的容器发送 SIGTERM 信号，如果容器内 1 号进程接受并能够处理 SIGTERM，则等待 1 号进程处理完毕后退出，如果等待一段时间后，容器仍然没有退出，则会发送 SIGKILL 强制终止容器。</p>
<pre class="lang-java" data-nodeid="1285"><code data-language="java">$ docker stop busybox
busybox
</code></pre>
<p data-nodeid="1286">如果你想查看停止状态的容器信息，你可以使用 docker ps -a 命令。</p>
<pre class="lang-java" data-nodeid="1287"><code data-language="java">$ docker ps -a
CONTAINERID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp; PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES
<span class="hljs-number">28d</span>477d3737a&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"sh"</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">26</span> <span class="hljs-function">minutes ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-title">Exited</span> <span class="hljs-params">(<span class="hljs-number">137</span>)</span> About a minute ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox
</span></code></pre>
<p data-nodeid="1288">处于终止状态的容器也可以通过<code data-backticks="1" data-nodeid="1426">docker start</code>命令来重新启动。</p>
<pre class="lang-java" data-nodeid="1289"><code data-language="java">$ docker start busybox
busybox
$ docker ps
CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES
<span class="hljs-number">28d</span>477d3737a&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"sh"</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;<span class="hljs-number">30</span> minutes ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up <span class="hljs-number">25</span> seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox
</code></pre>
<p data-nodeid="1290">此外，<code data-backticks="1" data-nodeid="1429">docker restart</code>命令会将一个运行中的容器终止，并且重新启动它。</p>
<pre class="lang-java" data-nodeid="1291"><code data-language="java">$ docker restart busybox
busybox
$ docker ps
CONTAINER ID&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; IMAGE&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; COMMAND&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATED&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; STATUS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;PORTS&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NAMES
<span class="hljs-number">28d</span>477d3737a&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"sh"</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-number">32</span> minutes ago&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Up <span class="hljs-number">3</span> seconds&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; busybox
</code></pre>
<h4 data-nodeid="1292">（3）进入容器</h4>
<p data-nodeid="1293">处于运行状态的容器可以通过<code data-backticks="1" data-nodeid="1433">docker attach</code>、<code data-backticks="1" data-nodeid="1435">docker exec</code>、<code data-backticks="1" data-nodeid="1437">nsenter</code>等多种方式进入容器。</p>
<ul data-nodeid="1294">
<li data-nodeid="1295">
<p data-nodeid="1296"><strong data-nodeid="1447">使用</strong><code data-backticks="1" data-nodeid="1442">docker attach</code>命令<strong data-nodeid="1448">进入容器</strong></p>
</li>
</ul>
<p data-nodeid="1297">使用 docker attach ，进入我们上一步创建好的容器，如下所示。</p>
<pre class="lang-java" data-nodeid="1298"><code data-language="java">$ docker attach busybox
/ # ps aux
PID&nbsp;&nbsp; USER&nbsp;&nbsp;&nbsp;&nbsp; TIME&nbsp; COMMAND
&nbsp;&nbsp;&nbsp; 1 root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0:00 sh
&nbsp;&nbsp;&nbsp; 7 root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0:00 ps aux
/ #
</code></pre>
<p data-nodeid="1299">注意：当我们同时使用<code data-backticks="1" data-nodeid="1451">docker attach</code>命令同时在多个终端运行时，所有的终端窗口将同步显示相同内容，当某个命令行窗口的命令阻塞时，其他命令行窗口同样也无法操作。<br>
由于<code data-backticks="1" data-nodeid="1455">docker attach</code>命令不够灵活，因此我们一般不会使用<code data-backticks="1" data-nodeid="1457">docker attach</code>进入容器。下面我介绍一个更加灵活的进入容器的方式<code data-backticks="1" data-nodeid="1459">docker exec</code></p>
<ul data-nodeid="1300">
<li data-nodeid="1301">
<p data-nodeid="1302"><strong data-nodeid="1463">使用 docker exec 命令进入容器</strong></p>
</li>
</ul>
<p data-nodeid="1303">Docker 从 1.3 版本开始，提供了一个更加方便地进入容器的命令<code data-backticks="1" data-nodeid="1465">docker exec</code>，我们可以通过<code data-backticks="1" data-nodeid="1467">docker exec -it CONTAINER</code>的方式进入到一个已经运行中的容器，如下所示。</p>
<pre class="lang-java" data-nodeid="1304"><code data-language="java">$ docker exec -it busybox sh
/ # ps aux
PID&nbsp;&nbsp; USER&nbsp;&nbsp;&nbsp;&nbsp; TIME&nbsp; COMMAND
&nbsp;&nbsp;&nbsp; 1 root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0:00 sh
&nbsp;&nbsp;&nbsp; 7 root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0:00 sh
&nbsp;&nbsp; 12 root&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 0:00 ps aux
</code></pre>
<p data-nodeid="1305">我们进入容器后，可以看到容器内有两个<code data-backticks="1" data-nodeid="1470">sh</code>进程，这是因为以<code data-backticks="1" data-nodeid="1472">exec</code>的方式进入容器，会单独启动一个 sh 进程，每个窗口都是独立且互不干扰的，也是使用最多的一种方式。</p>
<h4 data-nodeid="1306">（4）删除容器</h4>
<p data-nodeid="1307">我们已经掌握了用 Docker 命令创建、启动和终止容器。那如何删除处于终止状态或者运行中的容器呢？删除容器命令的使用方式如下：<code data-backticks="1" data-nodeid="1476">docker rm [OPTIONS] CONTAINER [CONTAINER...]</code>。</p>
<p data-nodeid="1308">如果要删除一个停止状态的容器，可以使用<code data-backticks="1" data-nodeid="1479">docker rm</code>命令删除。</p>
<pre class="lang-java" data-nodeid="1309"><code data-language="java">docker rm busybox
</code></pre>
<p data-nodeid="1310">如果要删除正在运行中的容器，必须添加 -f (或 --force) 参数， Docker 会发送 SIGKILL 信号强制终止正在运行的容器。</p>
<pre class="lang-java" data-nodeid="1311"><code data-language="java">docker rm -f busybox
</code></pre>
<h4 data-nodeid="1312">（5）导出导入容器</h4>
<ul data-nodeid="1313">
<li data-nodeid="1314">
<p data-nodeid="1315"><strong data-nodeid="1486">导出容器</strong></p>
</li>
</ul>
<p data-nodeid="1316">我们可以使用<code data-backticks="1" data-nodeid="1488">docker export CONTAINER</code>命令导出一个容器到文件，不管此时该容器是否处于运行中的状态。导出容器前我们先进入容器，创建一个文件，过程如下。</p>
<p data-nodeid="1317">首先进入容器创建文件</p>
<pre class="lang-java" data-nodeid="1318"><code data-language="java">docker exec -it busybox sh
cd /tmp &amp;&amp; touch test
</code></pre>
<p data-nodeid="1319">然后执行导出命令</p>
<pre class="lang-java" data-nodeid="1320"><code data-language="java">docker export busybox &gt; busybox.tar
</code></pre>
<p data-nodeid="1321">执行以上命令后会在当前文件夹下生成 busybox.tar 文件，我们可以将该文件拷贝到其他机器上，通过导入命令实现容器的迁移。</p>
<ul data-nodeid="1322">
<li data-nodeid="1323">
<p data-nodeid="1324"><strong data-nodeid="1496">导入容器</strong></p>
</li>
</ul>
<p data-nodeid="1325">通过<code data-backticks="1" data-nodeid="1498">docker export</code>命令导出的文件，可以使用<code data-backticks="1" data-nodeid="1500">docker import</code>命令导入，执行完<code data-backticks="1" data-nodeid="1502">docker import</code>后会变为本地镜像，最后再使用<code data-backticks="1" data-nodeid="1504">docker run</code>命令启动该镜像，这样我们就实现了容器的迁移。</p>
<p data-nodeid="1326">导入容器的命令格式为 docker import [OPTIONS] file|URL [REPOSITORY[:TAG]]。接下来我们一步步将上一步导出的镜像文件导入到其他机器的 Docker 中并启动它。</p>
<p data-nodeid="1327">首先，使用<code data-backticks="1" data-nodeid="1521">docker import</code>命令导入上一步导出的容器</p>
<pre class="lang-java" data-nodeid="1328"><code data-language="java">docker <span class="hljs-keyword">import</span> busybox.tar busybox:test
</code></pre>
<p data-nodeid="1329">此时，busybox.tar 被导入成为新的镜像，镜像名称为 busybox:test 。下面，我们使用<code data-backticks="1" data-nodeid="1524">docker run</code>命令启动并进入容器，查看上一步创建的临时文件</p>
<pre class="lang-java" data-nodeid="1330"><code data-language="java">docker run -it busybox:test sh
/ # ls /tmp/
test
</code></pre>
<p data-nodeid="1331">可以看到我们之前在 /tmp 目录下创建的 test 文件也被迁移过来了。这样我们就通过<code data-backticks="1" data-nodeid="1527">docker export</code>和<code data-backticks="1" data-nodeid="1529">docker import</code>命令配合实现了容器的迁移。</p>
<h3 data-nodeid="1332">结语</h3>
<p data-nodeid="1333">到此，我相信你已经了解了容器的基本概念和组成，并已经熟练掌握了容器各个生命周期操作和管理。那容器与镜像的区别，你应该也很清楚了。镜像包含了容器运行所需要的文件系统结构和内容，是静态的只读文件，而容器则是在镜像的只读层上创建了可写层，并且容器中的进程属于运行状态，容器是真正的应用载体。</p>
<p data-nodeid="1334" class="">那你知道为什么容器的文件系统要设计成写时复制(如图 1 所示)，而不是每一个容器都单独拷贝一份镜像文件吗？思考后，可以把你的想法写在留言区。</p>

---

### 精选评论

##### *雨：
> 每个容器单独拷贝一份镜像文件会占用较多的磁盘空间。假设我有3个程序都用到了jdk镜像，那该jdk镜像就得复制3份。如果改成写时复制的话，只需要一份jdk镜像，新的内容写在顶层即可。比如用docker images查看镜像的时候，把所有镜像的SIZE加起来可能有十几个G，但是到目录下查看，实际上只有几个G，这是因为很多镜像复用了同一个底层的镜像，起到了节省磁盘空间的作用。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 赞

##### TUTU：
> 讲解下容器迁移的save 是不是更好点

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; save 和 load 操作是针对镜像的，export 和 import 是针对容器的操作

##### **彬：
> 最重要的是学到了容器迁移

##### *波：
> docker状态应该是没有deleted的吧，只有created，running，paused和excited

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; deleted 是一个虚状态，表示被删除了，其实删了就看不见了。说没有 deleted 状态也没问题

##### TUTU：
> 容器迁移，原来这么简单

##### **辉：
> docker run -it --name=busybox busybox　　docker run --name=busybox busybox　老师，请教一下，上面两条命令创建的容器，第一个可以在stop状态下使用start命令进入运行状态，第二个start后还是停止状态，请问这是为什么？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 容器要想进入运行状态有两种方法，一种使用 -it 参数交互式运行，另外就是使用 -d 参数后台运行

##### **城：
> 老师，为什么先用docker create命令，再用docker start 命令启动容器，可以启动成功；而直接使用docker run 命令启动，容器会自动退出呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 具体的启动命名可以提供一下么？

##### *宇：
> 打包了，打不进去，open busybox.tar: no such file or directory

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 请确认执行 docker import 的目录是否有 busybox.tar 这个文件

##### **0588：
> 复用和节省空间，基础镜像标准化

##### **选：
> export 和 import 是不是和commit 的功能一样.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以实现类似的功能，但是本质是不一样的哈

##### **通：
> 为了节省磁盘空间， docker每个镜像都是有多个镜像层组成的， 并且是只读的， 启动时在最上层增加一个可读写层， 这样数据会保存到对应的volume中

##### **伟：
> 用docker run或者docker start命令，启动容器后，容器立马就退出了，怎么解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 启动命令是什么？容器本质是进程，启动需要一个不能退出的命令作为主进程。

##### Simon：
> 写时复制，是不是容器创建哪怕对镜像没有写的操作也会创建一个读写副本在容器层，然后这个副本不写它相当于只读镜像的软链，写的话就会实际占用磁盘空间？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有写的操作就不会创建任何文件，需要读取的话都是直接读取的镜像中的文件。只有修改了文件才会复制文件到镜像层。

##### **洋：
> 老师在我们社群哇？可以可以，老师讲的太好了！😁😁

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 郭少老师有看到社群的问题喔，谢谢支持，奥利给~

##### **盼：
> 期待实操+理论，原理性质多提提

##### **强：
> 想问一下老师，使用docker exec -it busybox sh，或 docker attach 进入容器后，若想从容器出来，使用什么命令好呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Ctrl+D 可以退出当前容器交互窗口哈

##### **6342：
> docker load命令是来做什么用的？跟dockerimport命令有什么不同呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; load 和 save  是搭配使用的，主要用来直接操作镜像，而 import 是和 export 搭配使用的，主要是用来操作容器的。具体可以参考这里 https://www.cnblogs.com/Cherry-Linux/p/8025777.html

##### **用户1280：
> 每个容器单独拷贝一份这不浪费空间嘛

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，镜像设计成分层结构可以节省很多存储空间

##### *豆：
> 一方面保护镜像文件，另一方面可以共享相同的镜像层？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，镜像分层主要是为了镜像层之间的复用

