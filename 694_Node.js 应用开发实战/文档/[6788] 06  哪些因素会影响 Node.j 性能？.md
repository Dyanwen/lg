<p data-nodeid="1197" class="">Node.js 作为后台服务性能是非常关键的一点，而影响 Node.js 的性能不仅仅要考虑其本身的因素，还应该考虑所在服务器的一些因素。前面我们介绍的 Node.js 的事件循环机制和 cluster 模式就是一种 Node.js 潜在的内在因素，而网络 I/O 、磁盘 I/O 以及其他内存、句柄的一些问题则是因为服务器的资源因素导致的性能问题。本讲就详细地分析影响其性能的因素原因，以及部分的优化解决方案。</p>
<h3 data-nodeid="1198">代码逻辑</h3>
<p data-nodeid="1199">影响性能的一个最大的原因就是在写 Node.js 代码时，没有注重性能影响问题，接下来我们就从三个方面来分析下到底哪些代码会出现性能影响。</p>
<h4 data-nodeid="1200">1.CPU 密集型计算</h4>
<p data-nodeid="1496" class="">CPU 负责了程序的运行和业务逻辑的处理，而 CPU 密集型表示的主要是 <strong data-nodeid="1502">CPU 承载了比较复杂的运算</strong>。</p>

<p data-nodeid="1202">在 Node.js 中由于主线程是单线程的（这部分知识点，在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=694#/detail/pc?id=6783" data-nodeid="1306">《01 | 事件循环：高性能到底是如何做到的？》</a>中已经非常详细地讲解了），无论是主线程逻辑，还是回调处理逻辑，最终都是在主线程处理，那么如果该线程一直在处理复杂的计算，其他请求就无法再次进来，也就是单个用户就可以阻塞所有用户的请求。因此保持主线程的通畅是非常关键的。</p>
<p data-nodeid="1203">在 Node.js 中有以下几种情况，会影响到主线程的运行，应该主动避免：</p>
<ul data-nodeid="1204">
<li data-nodeid="1205">
<p data-nodeid="1206"><strong data-nodeid="1313">大的数据循环</strong>，比如没有利用好数据流，一次性处理非常大的数组；</p>
</li>
<li data-nodeid="1207">
<p data-nodeid="1208"><strong data-nodeid="1318">字符串处理转化</strong>，比如加解密、字符串序列化等；</p>
</li>
<li data-nodeid="1209">
<p data-nodeid="1210"><strong data-nodeid="1327">图片</strong>、<strong data-nodeid="1328">视频的计算处理</strong>，比如对图片进行裁剪、缩放或者切割等。</p>
</li>
</ul>
<p data-nodeid="1211">接下来举个实际例子，假设我们现在有 2 个功能：</p>
<ul data-nodeid="1212">
<li data-nodeid="1213">
<p data-nodeid="1214">大数组的循环；</p>
</li>
<li data-nodeid="1215">
<p data-nodeid="1216">一个非常简单的 Hello World 输出。</p>
</li>
</ul>
<p data-nodeid="1217">再来看下因为大计算的逻辑如何影响了其他逻辑的处理，代码如下（请注意这里会应用 MSVC 框架来实现，因此我们只看 Controller 部分逻辑）：</p>
<pre class="lang-javascript" data-nodeid="1218"><code data-language="javascript"><span class="hljs-keyword">const</span> Controller = <span class="hljs-built_in">require</span>(<span class="hljs-string">'../core/controller'</span>);
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Controller</span> </span>{
    <span class="hljs-keyword">constructor</span>(res, req) {
        <span class="hljs-keyword">super</span>(res, req);
    }
    <span class="hljs-comment">/**
     * 复杂运算
     */</span>
    bad() {
        <span class="hljs-keyword">let</span> sum = <span class="hljs-number">0</span>;
        <span class="hljs-keyword">for</span>(<span class="hljs-keyword">let</span> i=<span class="hljs-number">0</span>; i&lt;<span class="hljs-number">10000000000</span>; i++){
            sum = sum + i;
        }

        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resApi(<span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, {<span class="hljs-string">'sum'</span> : sum});
    }
    <span class="hljs-comment">/**
     * 正常请求
     */</span>
    normal() {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resApi(<span class="hljs-literal">true</span>, <span class="hljs-string">'good'</span>, <span class="hljs-string">'hello world'</span>);
    }
}
<span class="hljs-built_in">module</span>.exports = Test;
</code></pre>
<p data-nodeid="1219">这段代码中 bad 是一个复杂的 CPU 计算，而 normal 是一个正常的请求。</p>
<p data-nodeid="1220">接下来我们使用 PM2 将服务运行，也可以直接运行（作为后台进程服务，建议后续都使用 PM2 来运行）。</p>
<pre class="lang-java" data-nodeid="1221"><code data-language="java">$ pm2 start pm2.config.js --env=development
或者
$ node index
</code></pre>
<p data-nodeid="1222">运行成功后，我们在浏览器可以多次访问该地址：</p>
<pre class="lang-java" data-nodeid="1223"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:3000/v1/normal</span>
或者在命令行运行
$ time curl http:<span class="hljs-comment">//127.0.0.1:3000/v1/normal</span>
</code></pre>
<p data-nodeid="1224">不管如何访问，响应速度都是非常快的，那么接下来我们打开浏览器的另外一个窗口，访问如下地址：</p>
<pre class="lang-java" data-nodeid="1225"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:3000/v1/cpu</span>
或者运行 
$ time curl http:<span class="hljs-comment">//127.0.0.1:3000/v1/cpu</span>
</code></pre>
<p data-nodeid="1226">你再切换到 normal 的请求链接，会发现响应时间变得非常慢了。</p>
<p data-nodeid="1227">这样就会因为某些用户的复杂运算，而影响到整个系统的请求处理，如果这种复杂运算占用的 CPU 时间越久，那么就会导致请求堆积，而这就会进一步导致系统处于崩溃状态无法恢复。</p>
<h4 data-nodeid="1228">2.网络 I/O</h4>
<p data-nodeid="1229">网络 I/O 中有 2 种相关的类型，同步阻塞 I/O 和 异步非阻塞 I/O：</p>
<ul data-nodeid="1230">
<li data-nodeid="1231">
<p data-nodeid="1232"><strong data-nodeid="1345">同步阻塞 I/O</strong>的字面意思是发出网络请求后需要等待返回后，再处理其他计算；</p>
</li>
<li data-nodeid="1233">
<p data-nodeid="1234"><strong data-nodeid="1350">异步非阻塞 I/O</strong>就是发起网络 I/O 后，还可以处理其他的计算，这也是为什么 Node.js 在处理网络 I/O 性能较高的原因。</p>
</li>
</ul>
<p data-nodeid="1235">举个例子，假设我们有个功能需要访问一个 API 的数据，Node.js 调用 API 就是一种网络 I/O：</p>
<ul data-nodeid="1236">
<li data-nodeid="1237">
<p data-nodeid="1238">如果该 API 处理慢，那么则所有用户请求都被阻塞了；</p>
</li>
<li data-nodeid="1239">
<p data-nodeid="1240">而如果异步的话，则无须等待处理，可以继续其他的运行。</p>
</li>
</ul>
<p data-nodeid="2101" class="">在 CPU 例子中，我们有一种办法就是将 CPU 密集型计算使用其他进程来处理，那么这里我们可以来做一个简单的测试，启用 2 个服务，一个是使用 <strong data-nodeid="2107">CPU 密集型计算</strong>（保留上面 CPU 的服务、代码不变）、另外一个则是正常请求的，这部分 Controller 代码如下：</p>

<pre class="lang-javascript" data-nodeid="1242"><code data-language="javascript"><span class="hljs-keyword">const</span> rp = <span class="hljs-built_in">require</span>(<span class="hljs-string">'request-promise'</span>);
<span class="hljs-keyword">const</span> Controller = <span class="hljs-built_in">require</span>(<span class="hljs-string">'../core/controller'</span>);
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Test</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Controller</span> </span>{
    <span class="hljs-keyword">constructor</span>(res, req) {
        <span class="hljs-keyword">super</span>(res, req);
    }
    <span class="hljs-comment">/**
     * 复杂运算，使用网络 I/O 调用
     */</span>
    <span class="hljs-keyword">async</span> bad() {
        <span class="hljs-keyword">let</span> result = <span class="hljs-keyword">await</span> rp.get(<span class="hljs-string">'http://127.0.0.1:3000/v1/cpu'</span>);

        <span class="hljs-keyword">let</span> sumData = <span class="hljs-built_in">JSON</span>.parse(result);
        <span class="hljs-keyword">let</span> sum = sumData &amp;&amp; sumData.data ? sumData.data.sum : <span class="hljs-literal">false</span>;
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resApi(<span class="hljs-literal">true</span>, <span class="hljs-string">'success'</span>, {<span class="hljs-string">'sum'</span> : sum});
    }
    <span class="hljs-comment">/**
     * 正常请求
     */</span>
    normal() {
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.resApi(<span class="hljs-literal">true</span>, <span class="hljs-string">'good'</span>, <span class="hljs-string">'hello world io'</span>);
    }
}
<span class="hljs-built_in">module</span>.exports = Test;
</code></pre>
<p data-nodeid="1243">这部分代码和上面 CPU 代码例子唯一不同在于<strong data-nodeid="1365">bad 函数中复杂的运算使用了网络 I/O</strong>，这样就不会影响 normal 的请求了。</p>
<p data-nodeid="1244">接下来我们将 CPU 部分的服务启动，然后在浏览器再次访问。</p>
<pre class="lang-java" data-nodeid="1245"><code data-language="java">http:<span class="hljs-comment">//127.0.0.1:4000/v1/cpu</span>
http:<span class="hljs-comment">//127.0.0.1:4000/v1/normal</span>
</code></pre>
<p data-nodeid="1246">你会发现虽然 /v1/cpu 很慢，但是并不影响 /v1/normal 的请求，这就是我们上面介绍到的为什么 Node.js 适合网络 I/O 密集型的服务的原因了。</p>
<p data-nodeid="4518" class="">从上面的例子中，看到了网络 I/O 其实是 Node.js 的优势，虽然不影响主线程的处理，但是对于 <a href="http://127.0.0.1:4000/v1/cpu" data-nodeid="4522">http://127.0.0.1:4000/v1/cpu</a> 这个请求，如果要提升性能，我们应该关注什么呢？</p>




<ul data-nodeid="1248">
<li data-nodeid="1249">
<p data-nodeid="1250"><strong data-nodeid="1377">通道复用</strong>，比如我们现在每次访问 :4000/v1/cpu 时都会发起一个 TCP 到 :3000/v1/cpu，如果能够通道复用，减少 TCP 握手，那么就可以提升该接口的性能，或者将某些内部服务使用 UDP 来实现。</p>
</li>
<li data-nodeid="1251">
<p data-nodeid="1252"><strong data-nodeid="1382">增加缓存</strong>，对于相同响应的返回数据，增加缓存处理，避免不必要的计算，对于上面的计算，我们完全可以缓存计算结果，这样来减少网络 I/O。</p>
</li>
<li data-nodeid="1253">
<p data-nodeid="1254"><strong data-nodeid="1387">长链接链接池</strong>，有一些网络 I/O 是长链接的形式，比如 MySQL、Mamcached 或者 Redis，为了避免排队使用长链接的问题，可以使用链接池，而由于 Redis 和 Node.js 是单线程非阻塞处理，因此可以不用链接池。</p>
</li>
</ul>
<p data-nodeid="1255">网络 I/O 一般不影响主线程逻辑，往往<strong data-nodeid="1393">网络 I/O 请求的服务反而是瓶颈端</strong>，从而影响 Node.js 中涉及该网络服务的请求。其次网络 I/O 堆积较多会侧面影响：</p>
<ul data-nodeid="1256">
<li data-nodeid="1257">
<p data-nodeid="1258">服务器本身的网络模块问题；</p>
</li>
<li data-nodeid="1259">
<p data-nodeid="1260">Node.js 性能，导致其他服务接口受影响。</p>
</li>
</ul>
<p data-nodeid="1261">因此在网络 I/O 瓶颈时需要考虑修改服务器网络相关的配置。</p>
<h4 data-nodeid="1262">3.磁盘 I/O</h4>
<p data-nodeid="1263">和网络 I/O 相似，在一般情况下磁盘 I/O 是不会影响到主线程性能的，这里就不举例子了，因为磁盘 I/O 也是异步其他线程处理。这里所说的也不是主线程的问题，而是涉及磁盘 I/O 的服务请求。因为服务器的<strong data-nodeid="1403">磁盘性能</strong>是一定的，如果在高并发情况下，磁盘 I/O 压力较大，从而导致磁盘 I/O 的服务性能下降。</p>
<p data-nodeid="1264">在实际开发过程中，最常见的磁盘 I/O 场景，那就是<strong data-nodeid="1409">日志模块</strong>，因为日志是需要写文件，从而会有频繁的日志写入。和网络 I/O 相似，磁盘 I/O 也会从侧面的影响机器性能，导致 Node.js 服务性能受影响。</p>
<p data-nodeid="1265">在上面 3 种情况中，只有<strong data-nodeid="1425">CPU 密集计算会真正影响到 Node.js 服务性能</strong>，<strong data-nodeid="1426">而网络 I/O 和磁盘 I/O 都是直接影响服务器性能</strong>，<strong data-nodeid="1427">从而侧面影响到 Node.js 服务性能</strong>，一般这时候就需要调整服务器配置或者做一些队列优化方式来提升服务器性能，这点我们在《12 | 性能分析：性能影响的关键路径以及优化策略》将会介绍。</p>
<h3 data-nodeid="1266">集群服务</h3>
<p data-nodeid="1267"><strong data-nodeid="1433">后台服务一般都有集群的概念</strong>，无论是多机器部署，还是单机器（Node.js cluster 模式），具体我们画一个集群的架构例子，如图 1 所示。在进程分发的主节点 Nginx 和 Master 都可能会存在性能影响因素点，本讲核心是介绍 Node.js，因此我们主要看 cluster 模式的性能影响问题。</p>
<p data-nodeid="1268"><img src="https://s0.lgstatic.com/i/image6/M00/1D/E1/CioPOWBQK0GAZFfaAABsROVQ92Y096.png" alt="Drawing 1.png" data-nodeid="1436"></p>
<h4 data-nodeid="1269">1.多进程 cluster 模式</h4>
<p data-nodeid="1270">在上一讲中我们详细地介绍了 cluster 模式，在实际应用过程中这种模式也是存在性能瓶颈问题的。我们在上一讲中讲到的 cluster 模式，如图 2 所示。</p>
<p data-nodeid="1271"><img src="https://s0.lgstatic.com/i/image6/M00/1D/E1/CioPOWBQK0mAAVVQAAB3dogpY-k967.png" alt="Drawing 3.png" data-nodeid="1441"></p>
<p data-nodeid="1272">你会发现这种模式的主进程也就是上一讲中的<strong data-nodeid="1447">master 进程</strong>会存在瓶颈，因为所有的请求都必须经过 master 进程进行分发，同时接收处理 worker 进程的返回。</p>
<p data-nodeid="1273">在实际开发过程中，遇到一个问题，由于我们所用机器是一个 96 核以上的服务器，因此启用了比较多的 worker 进程，而主进程只有一个，从而在单机高并发时（2 万以上的每秒并发请求）会导致 master 进程处理瓶颈，这样就影响到了服务性能，并且这时候你会发现 worker 进程的 CPU 并没有任何压力。</p>
<p data-nodeid="1274">以上这点非常重要，在生产环境下一般很难发现这类问题，不过你应该有一个这样的概念：大概在 2 万以上的并发时，master 进程会存在性能瓶颈。</p>
<h3 data-nodeid="1275">其他相关</h3>
<p data-nodeid="1276">对于 Node.js 后台服务，我们不仅仅要考虑其本身的性能影响，更应该考虑它对服务器资源竞争产生的性能影响。比如<strong data-nodeid="1456">无节制地使用服务器的内存或者句柄</strong>，都会导致服务器的异常，而服务器的异常则从侧面影响到 Node.js 本身的性能。</p>
<h4 data-nodeid="1277">1.内存限制</h4>
<p data-nodeid="1278">在 32 位服务器上 Node.js 的内存限制是 0.7 G，而在 64 位服务器上则是 1.4 G，而这个限制主要是因为 Node.js 的垃圾回收线程在超过限制内存时，回收时长循环会大于 1s，从而会影响性能问题。</p>
<p data-nodeid="1279">现网我们一般会<strong data-nodeid="1464">启用多个进程</strong>，如果每个进程损耗 1.4 G，那么加起来可能超出了服务器内存上限，从而导致服务器瘫痪。其次如果内存不会超出服务器上限，而是在达到一定上限时，也就是我们上面说的 0.7 G和 1.4 G，会导致服务器重启，从而会导致接口请求失败的问题。</p>
<h4 data-nodeid="1280">2.句柄限制</h4>
<p data-nodeid="5122" class="te-preview-highlight">句柄可以简单理解为一个 <strong data-nodeid="5128">ID 索引</strong>，通过这个索引可以访问到其他的资源，比如说文件句柄、网络 I/O 操作句柄等等，而一般服务器句柄都有上限。当 Node.js 没有控制好句柄，比如说无限的打开文件并未关闭，就会出现句柄泄漏问题，而这样会导致服务器异常，从而影响 Node.js 服务。</p>

<p data-nodeid="1282">以上这两点我们都需要有一定的工具检测方法，<strong data-nodeid="1477">在服务上限之前进行检测</strong>，其次也需要有一定的定位的方法，在出现现网异常时，能够定位出具体的问题，这也就是我们接下来会涉及的知识点。</p>
<h3 data-nodeid="1283">总结</h3>
<p data-nodeid="1284">本讲介绍了影响 Node.js 后台服务的代码因素、cluster 模式因素以及其他服务器资源相关的因素。学完本讲后，要了解哪些直接影响性能，哪些是从侧面也就是因为服务器资源问题导致的性能下降原因。本讲中的几个知识点，也是面试中常见的面试题，希望你能牢记这些知识点，这些也是我们后续章节中经常会被提及的一些知识。</p>
<p data-nodeid="1285">那这一讲学完，你有什么心得或者问题吗？欢迎在评论区与我讨论。</p>
<p data-nodeid="1286">下一讲将会为你讲解“CPU 过载保护设计：如何在服务层面确保系统稳定”。</p>
<hr data-nodeid="1287">
<p data-nodeid="1288"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1486"><img src="https://s0.lgstatic.com/i/image6/M00/12/FA/CioPOWBBrAKAAod-AASyC72ZqWw233.png" alt="Drawing 2.png" data-nodeid="1485"></a></p>
<p data-nodeid="1289"><strong data-nodeid="1490">《大前端高薪训练营》</strong></p>
<p data-nodeid="1290" class="">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="1494">点击链接</a>，快来领取！</p>

---

### 精选评论

##### **杰：
> 虽然还是有点迷糊具体实践，但是总归是有个大体概念了，就是通过多进程来解决单线程的缺陷

##### *舒：
> 看完这章节，我觉得我做BFF更有信心了

##### **田：
> 666

