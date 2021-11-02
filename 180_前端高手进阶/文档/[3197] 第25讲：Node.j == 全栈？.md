<p data-nodeid="24471">提到 Node.js，相信大部分前端工程师都会想到基于它来开发服务端，只需要掌握 JavaScript 一门语言就可以成为全栈工程师，但其实 Node.js 的意义并不仅于此。</p>
<p data-nodeid="24472">很多高级语言，执行权限都可以触及操作系统，而运行在浏览器端的 JavaScript 则例外，浏览器为其创建的沙箱环境，把前端工程师封闭在一个编程世界的象牙塔里。不过 Node.js 的出现则弥补了这个缺憾，前端工程师也可以触达计算机世界的底层。</p>
<p data-nodeid="24473">所以 Node.js 对于前端工程师的意义不仅在于提供了全栈开发能力，更重要的是为前端工程师打开了一扇通向计算机底层世界的大门。这一课时我们通过分析 Node.js 的实现原理来打开这扇大门。</p>
<h3 data-nodeid="24474">Node.js 源码结构</h3>
<p data-nodeid="26505">Node.js 源码仓库的 /deps 目录下有十几个依赖，其中既有 C 语言编写的模块（如 libuv、V8）也有 JavaScript 语言编写的模块（如 acorn、acorn-plugins），如下图所示。</p>
<p data-nodeid="27093"><img src="https://s0.lgstatic.com/i/image/M00/46/40/Ciqc1F9EvvmAC2x9AAAVgzc1Izg188.png" alt="image.png" data-nodeid="27097"></p>
<div data-nodeid="27676" class=""><p style="text-align:center">Node.js 的依赖模块</p></div>






<ul data-nodeid="24478">
<li data-nodeid="24479">
<p data-nodeid="24480">acorn：前面的课程中已经提过，用 JavaScript 编写的轻量级 JavaScript 解析器。</p>
</li>
<li data-nodeid="24481">
<p data-nodeid="24482">acorn-plugins：acorn 的扩展模块，让 acorn 支持 ES6 特性解析，比如类声明。</p>
</li>
<li data-nodeid="24483">
<p data-nodeid="24484">brotli：C 语言编写的 Brotli 压缩算法。</p>
</li>
<li data-nodeid="24485">
<p data-nodeid="24486">cares：应该写为“c-ares”，C 语言编写的用来处理异步 DNS 请求。</p>
</li>
<li data-nodeid="24487">
<p data-nodeid="24488">histogram：C 语言编写，实现柱状图生成功能。</p>
</li>
<li data-nodeid="24489">
<p data-nodeid="24490">icu-small：C 语言编写，为 Node.js 定制的 ICU（International Components for Unicode）库，包括一些用来操作 Unicode 的函数。</p>
</li>
<li data-nodeid="24491">
<p data-nodeid="24492">llhttp：C 语言编写，轻量级的 http 解析器。</p>
</li>
<li data-nodeid="24493">
<p data-nodeid="24494">nghttp2/nghttp3/ngtcp2：处理 HTTP/2、HTTP/3、TCP/2 协议。</p>
</li>
<li data-nodeid="24495">
<p data-nodeid="24496">node-inspect：让 Node.js 程序支持 CLI debug 调试模式。</p>
</li>
<li data-nodeid="24497">
<p data-nodeid="24498">npm：JavaScript 编写的 Node.js 模块管理器。</p>
</li>
<li data-nodeid="24499">
<p data-nodeid="24500">openssl：C 语言编写，加密相关的模块，在 tls 和 crypto 模块中都有使用。</p>
</li>
<li data-nodeid="24501">
<p data-nodeid="24502">uv：C 语言编写，采用非阻塞型的 I/O 操作，为 Node.js 提供了访问系统资源的能力。</p>
</li>
<li data-nodeid="24503">
<p data-nodeid="24504">uvwasi：C 语编写，实现 WASI 系统调用 API。</p>
</li>
<li data-nodeid="24505">
<p data-nodeid="24506">v8：C 语言编写，JavaScript 引擎。</p>
</li>
<li data-nodeid="24507">
<p data-nodeid="24508">zlib：用于快速压缩，Node.js 使用 zlib 创建同步、异步和数据流压缩、解压接口。</p>
</li>
</ul>
<p data-nodeid="24509">其中最重要的是v8 和uv两个目录对应的模块。</p>
<p data-nodeid="24510">在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=180#/detail/pc?id=3180" data-nodeid="24599">09 | 为什么代码没有按照编写顺序执行</a>”中我们详细分析过 V8 的工作原理，V8 本身并没有异步运行的能力，而是借助浏览器的其他线程实现的。但在 Node.js 中，异步实现主要依赖于 libuv，下面我们来重点分析 libuv 的实现原理。</p>
<h3 data-nodeid="24511">什么是 libuv</h3>
<p data-nodeid="28819">libuv 是一个用 C 编写的支持多平台的异步 I/O 库，主要解决 I/O 操作容易引起阻塞的问题。最开始是专门为 Node.js 使用而开发的，但后来也被 Luvit、Julia、pyuv 等其他模块使用。下图是 libuv 的结构图。</p>
<p data-nodeid="29976"><img src="https://s0.lgstatic.com/i/image/M00/46/4C/CgqCHl9EvxCAf_yRAAD8dPwXfWE007.png" alt="1.png" data-nodeid="29980"></p>
<div data-nodeid="29977" class=""><p style="text-align:center">libuv 结构图</p></div>
<p></p>






<p data-nodeid="24516">我用黄色线框将图中模块分为了两部分，分别代表了两种不同的异步实现方式。</p>
<p data-nodeid="24517">左边部分为网络 I/O 模块，在不同平台下有不同的实现机制，Linux 系统下通过 epoll 实现，OSX 和其他 BSD 系统采用 KQueue，SunOS 系统采用 Event ports，Windows 系统采用的是IOCP。由于涉及操作系统底层 API，理解起来比较复杂，这里就不多介绍了，对这些实现机制比较感兴趣的同学可以查阅这篇文章“<a href="https://cloud.tencent.com/developer/article/1373483" data-nodeid="24614">各种 IO 复用模式之 select、poll、epoll、kqueue、iocp 分析</a>”。</p>
<p data-nodeid="24518">右边部分包括文件 I/O 模块、DNS 模块和用户代码，通过线程池来实现异步操作。文件 I/O 与网络 I/O 不同，libuv 没有依赖于系统底层的 API，而是在全局线程池中执行阻塞的文件 I/O 操作。</p>
<h3 data-nodeid="24519">libuv 中的事件轮询</h3>
<p data-nodeid="32242">下图是 libuv 官网给出的事件轮询工作流程图，我们结合代码来一起分析。</p>
<p data-nodeid="32243" class=""><img src="https://s0.lgstatic.com/i/image/M00/46/4C/CgqCHl9EvySAMrSYAADR1tJd-r8402.png" alt="image (1).png" data-nodeid="32252"></p>
<div data-nodeid="32244"><p style="text-align:center">libuv 事件轮询</p></div>






<p data-nodeid="24523">libuv 事件循环的核心代码是在 uv_run() 函数中实现的，下面是 Unix 系统下的部分核心代码。虽然是用 C 语言编写的，但和 JavaScript 一样都是高级语言，所以理解起来也不算太困难。最大的区别可能是星号和箭头，星号我们可以直接忽略。例如，函数参数中 uv_loop_t* loop 可以理解为 uv_loop_t 类型的变量 loop。箭头“→”可以理解为点号“.”，例如，loop→stop_flag 可以理解为 loop.stop_flag。</p>
<pre class="lang-c++" data-nodeid="24524"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/unix/core.c</span>
<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">uv_run</span><span class="hljs-params">(<span class="hljs-keyword">uv_loop_t</span>* <span class="hljs-built_in">loop</span>, uv_run_mode mode)</span> </span>{
  ...
  r = uv__loop_alive(<span class="hljs-built_in">loop</span>);
  <span class="hljs-keyword">if</span> (!r)
&nbsp; &nbsp; uv__update_time(<span class="hljs-built_in">loop</span>);
&nbsp; <span class="hljs-keyword">while</span> (r != <span class="hljs-number">0</span> &amp;&amp; <span class="hljs-built_in">loop</span>-&gt;stop_flag == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; uv__update_time(<span class="hljs-built_in">loop</span>);
&nbsp; &nbsp; uv__run_timers(<span class="hljs-built_in">loop</span>);
&nbsp; &nbsp; ran_pending = uv__run_pending(<span class="hljs-built_in">loop</span>);
&nbsp; &nbsp; uv__run_idle(<span class="hljs-built_in">loop</span>);
&nbsp; &nbsp; uv__run_prepare(<span class="hljs-built_in">loop</span>);
    ...
&nbsp; &nbsp; uv__io_poll(<span class="hljs-built_in">loop</span>, timeout);
&nbsp; &nbsp; uv__run_check(<span class="hljs-built_in">loop</span>);
&nbsp; &nbsp; uv__run_closing_handles(<span class="hljs-built_in">loop</span>);
    ...
&nbsp; }
  ...
}
</code></pre>
<h4 data-nodeid="24525">uv__loop_alive</h4>
<p data-nodeid="24526">这个函数用于判断事件轮询是否要继续进行，如果 loop 对象中不存在活跃的任务则返回 0 并退出循环。</p>
<p data-nodeid="24527">在 C 语言中这个“任务”有个专业的称呼，即“句柄”，可以理解为指向任务的变量。句柄又可以分为两类：request 和 handle，分别代表短生命周期句柄和长生命周期句柄。具体代码如下：</p>
<pre class="lang-c++" data-nodeid="24528"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/unix/core.c</span>
<span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> <span class="hljs-title">uv__loop_alive</span><span class="hljs-params">(<span class="hljs-keyword">const</span> <span class="hljs-keyword">uv_loop_t</span>* <span class="hljs-built_in">loop</span>)</span> </span>{
&nbsp; <span class="hljs-keyword">return</span> uv__has_active_handles(<span class="hljs-built_in">loop</span>) ||
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;uv__has_active_reqs(<span class="hljs-built_in">loop</span>) ||
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-built_in">loop</span>-&gt;closing_handles != <span class="hljs-literal">NULL</span>;
}
</code></pre>
<h4 data-nodeid="24529">uv__update_time</h4>
<p data-nodeid="24530">为了减少与时间相关的系统调用次数，同构这个函数来缓存当前系统时间，精度很高，可以达到纳秒级别，但单位还是毫秒。</p>
<p data-nodeid="24531">具体源码如下：</p>
<pre class="lang-c++" data-nodeid="24532"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/unix/internal.h</span>
UV_UNUSED(<span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> uv__update_time(<span class="hljs-keyword">uv_loop_t</span>* <span class="hljs-built_in">loop</span>)) {
&nbsp; <span class="hljs-built_in">loop</span>-&gt;time = uv__hrtime(UV_CLOCK_FAST) / <span class="hljs-number">1000000</span>;
}
</code></pre>
<h4 data-nodeid="24533">uv__run_timers</h4>
<p data-nodeid="24534">执行 setTimeout() 和 setInterval() 中到达时间阈值的回调函数。这个执行过程是通过 for 循环遍历实现的，从下面的代码中也可以看到，定时器回调是存储于一个最小堆结构的数据中的，当这个最小堆为空或者还未到达时间阈值时退出循环。</p>
<p data-nodeid="24535">在执行定时器回调函数前先移除该定时器，如果设置了 repeat，需再次加到最小堆里，然后执行定时器回调。</p>
<p data-nodeid="24536">具体代码如下：</p>
<pre class="lang-c++" data-nodeid="24537"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/timer.c</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">uv__run_timers</span><span class="hljs-params">(<span class="hljs-keyword">uv_loop_t</span>* <span class="hljs-built_in">loop</span>)</span> </span>{
&nbsp; <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">heap_node</span>* <span class="hljs-title">heap_node</span>;</span>
&nbsp; <span class="hljs-keyword">uv_timer_t</span>* handle;
&nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; heap_node = heap_min(timer_heap(<span class="hljs-built_in">loop</span>));
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (heap_node == <span class="hljs-literal">NULL</span>)
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; handle = container_of(heap_node, <span class="hljs-keyword">uv_timer_t</span>, heap_node);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (handle-&gt;timeout &gt; <span class="hljs-built_in">loop</span>-&gt;time)
&nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; uv_timer_stop(handle);
&nbsp; &nbsp; uv_timer_again(handle);
&nbsp; &nbsp; handle-&gt;timer_cb(handle);
&nbsp; }
}
</code></pre>
<h4 data-nodeid="24538">uv__run_pending</h4>
<p data-nodeid="24539">遍历所有存储在 pending_queue 中的 I/O 回调函数，当 pending_queue 为空时返回 0；否则在执行完 pending_queue 中的回调函数后返回 1。</p>
<p data-nodeid="24540">代码如下：</p>
<pre class="lang-c++" data-nodeid="24541"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/unix/core.c</span>
<span class="hljs-function"><span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;<span class="hljs-title">uv__run_pending</span><span class="hljs-params">(<span class="hljs-keyword">uv_loop_t</span>*&nbsp;<span class="hljs-built_in">loop</span>)</span>&nbsp;</span>{
&nbsp;&nbsp;QUEUE*&nbsp;q;
&nbsp;&nbsp;QUEUE&nbsp;pq;
&nbsp;&nbsp;<span class="hljs-keyword">uv__io_t</span>*&nbsp;w;
&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(QUEUE_EMPTY(&amp;<span class="hljs-built_in">loop</span>-&gt;pending_queue))
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;QUEUE_MOVE(&amp;<span class="hljs-built_in">loop</span>-&gt;pending_queue,&nbsp;&amp;pq);
&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(!QUEUE_EMPTY(&amp;pq))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;q&nbsp;=&nbsp;QUEUE_HEAD(&amp;pq);
&nbsp;&nbsp;&nbsp;&nbsp;QUEUE_REMOVE(q);
&nbsp;&nbsp;&nbsp;&nbsp;QUEUE_INIT(q);
&nbsp;&nbsp;&nbsp;&nbsp;w&nbsp;=&nbsp;QUEUE_DATA(q,&nbsp;<span class="hljs-keyword">uv__io_t</span>,&nbsp;pending_queue);
&nbsp;&nbsp;&nbsp;&nbsp;w-&gt;cb(<span class="hljs-built_in">loop</span>,&nbsp;w,&nbsp;POLLOUT);
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-number">1</span>;
}
</code></pre>
<h4 data-nodeid="24542">uv__run_idle / uv__run_prepare / uv__run_check</h4>
<p data-nodeid="24543">这 3 个函数都是通过一个宏函数 UV_LOOP_WATCHER_DEFINE 进行定义的，宏函数可以理解为代码模板，或者说用来定义函数的函数。3 次调用宏函数并分别传入 name 参数值 prepare、check、idle，同时定义了 uv__run_idle、uv__run_prepare、uv__run_check 3 个函数。</p>
<p data-nodeid="24544">所以说它们的执行逻辑是一致的，都是按照先进先出原则循环遍历并取出队列 loop-&gt;name##_handles 中的对象，然后执行对应的回调函数。</p>
<pre class="lang-c++" data-nodeid="24545"><code data-language="c++"><span class="hljs-comment">// deps/uv/src/unix/loop-watcher.c</span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> UV_LOOP_WATCHER_DEFINE(name, type)&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; void uv__run_##name(uv_loop_t* loop) {&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; uv_##name##_t* h;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;\
&nbsp; &nbsp; QUEUE queue;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; QUEUE* q;&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;\
&nbsp; &nbsp; QUEUE_MOVE(&amp;loop-&gt;name##_handles, &amp;queue);&nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; while (!QUEUE_EMPTY(&amp;queue)) {&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; &nbsp; q = QUEUE_HEAD(&amp;queue);&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;\
&nbsp; &nbsp; &nbsp; h = QUEUE_DATA(q, uv_##name##_t, queue);&nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; &nbsp; QUEUE_REMOVE(q);&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; &nbsp; QUEUE_INSERT_TAIL(&amp;loop-&gt;name##_handles, q);&nbsp; \
&nbsp; &nbsp; &nbsp; h-&gt;name##_cb(h);&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; \
&nbsp; &nbsp; }&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;\
&nbsp; }&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;\
</span>
UV_LOOP_WATCHER_DEFINE(<span class="hljs-built_in">prepare</span>, PREPARE)
UV_LOOP_WATCHER_DEFINE(check, CHECK)
UV_LOOP_WATCHER_DEFINE(idle, IDLE)
</code></pre>
<h4 data-nodeid="24546">uv__io_poll</h4>
<p data-nodeid="24547">uv__io_poll&nbsp;主要是用来轮询 I/O 操作。具体实现根据操作系统的不同会有所区别，我们以 Linux 系统为例进行分析。</p>
<p data-nodeid="24548">uv__io_poll 函数源码较多，核心为两段循环代码，部分代码如下：</p>
<pre class="lang-c++" data-nodeid="24549"><code data-language="c++"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">uv__io_poll</span><span class="hljs-params">(<span class="hljs-keyword">uv_loop_t</span>* <span class="hljs-built_in">loop</span>, <span class="hljs-keyword">int</span> timeout)</span> </span>{
    <span class="hljs-keyword">while</span> (!QUEUE_EMPTY(&amp;<span class="hljs-built_in">loop</span>-&gt;watcher_queue)) {
     &nbsp;q&nbsp;=&nbsp;QUEUE_HEAD(&amp;<span class="hljs-built_in">loop</span>-&gt;watcher_queue);
  &nbsp;&nbsp;&nbsp;&nbsp;QUEUE_REMOVE(q);
  &nbsp;&nbsp;&nbsp;&nbsp;QUEUE_INIT(q);
  &nbsp;&nbsp;&nbsp;&nbsp;w&nbsp;=&nbsp;QUEUE_DATA(q,&nbsp;<span class="hljs-keyword">uv__io_t</span>,&nbsp;watcher_queue);
      <span class="hljs-comment">// 设置当前感兴趣的事件</span>
  &nbsp;&nbsp;&nbsp;&nbsp;e.events&nbsp;=&nbsp;w-&gt;pevents;
      <span class="hljs-comment">// 设置事件对象的文件描述符</span>
  &nbsp;&nbsp;&nbsp;&nbsp;e.data.fd&nbsp;=&nbsp;w-&gt;fd;
  &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(w-&gt;events&nbsp;==&nbsp;<span class="hljs-number">0</span>)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;op&nbsp;=&nbsp;EPOLL_CTL_ADD;
  &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;op&nbsp;=&nbsp;EPOLL_CTL_MOD;
      <span class="hljs-comment">// 修改 epoll 事件</span>
  &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(epoll_ctl(<span class="hljs-built_in">loop</span>-&gt;backend_fd,&nbsp;op,&nbsp;w-&gt;fd,&nbsp;&amp;e))&nbsp;{
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(errno&nbsp;!=&nbsp;EEXIST)
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">abort</span>();
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(epoll_ctl(<span class="hljs-built_in">loop</span>-&gt;backend_fd,&nbsp;EPOLL_CTL_MOD,&nbsp;w-&gt;fd,&nbsp;&amp;e))
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">abort</span>();
  &nbsp;&nbsp;&nbsp;&nbsp;}  
  &nbsp;&nbsp;&nbsp;&nbsp;w-&gt;events&nbsp;=&nbsp;w-&gt;pevents;
    }
    <span class="hljs-keyword">for</span> (;;) {
      <span class="hljs-keyword">for</span>&nbsp;(i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;nfds;&nbsp;i++)&nbsp;{
      &nbsp;&nbsp;pe&nbsp;=&nbsp;events&nbsp;+&nbsp;i;
      &nbsp;&nbsp;fd&nbsp;=&nbsp;pe-&gt;data.fd;
      &nbsp;&nbsp;w&nbsp;=&nbsp;<span class="hljs-built_in">loop</span>-&gt;watchers[fd];
      &nbsp;&nbsp;pe-&gt;events&nbsp;&amp;=&nbsp;w-&gt;pevents&nbsp;|&nbsp;POLLERR&nbsp;|&nbsp;POLLHUP;
      &nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(pe-&gt;events&nbsp;==&nbsp;POLLERR&nbsp;||&nbsp;pe-&gt;events&nbsp;==&nbsp;POLLHUP)
      &nbsp;&nbsp;&nbsp;&nbsp;pe-&gt;events&nbsp;|=&nbsp;w-&gt;pevents&nbsp;&amp;&nbsp;(POLLIN&nbsp;|&nbsp;POLLOUT&nbsp;|&nbsp;UV__POLLRDHUP&nbsp;|&nbsp;UV__POLLPRI);
      &nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(pe-&gt;events&nbsp;!=&nbsp;<span class="hljs-number">0</span>)&nbsp;{
          <span class="hljs-comment">// 感兴趣事件触发，标记信号 </span>
      &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(w&nbsp;==&nbsp;&amp;<span class="hljs-built_in">loop</span>-&gt;signal_io_watcher)
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;have_signals&nbsp;=&nbsp;<span class="hljs-number">1</span>;
      &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>
            <span class="hljs-comment">// 直接执行回调</span>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;w-&gt;cb(<span class="hljs-built_in">loop</span>,&nbsp;w,&nbsp;pe-&gt;events);
      &nbsp;&nbsp;&nbsp;&nbsp;nevents++;
      &nbsp;&nbsp;}
      }
      <span class="hljs-comment">// 有信号发生时触发回调</span>
     &nbsp;<span class="hljs-keyword">if</span>&nbsp;(have_signals&nbsp;!=&nbsp;<span class="hljs-number">0</span>)
&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-built_in">loop</span>-&gt;signal_io_watcher.cb(<span class="hljs-built_in">loop</span>,&nbsp;&amp;<span class="hljs-built_in">loop</span>-&gt;signal_io_watcher,&nbsp;POLLIN);
   }
   ...
}
</code></pre>
<p data-nodeid="24550">在 while 循环中，遍历观察者队列 watcher_queue，并把事件和文件描述符取出来赋值给事件对象 e，然后调用 epoll_ctl 函数来注册或修改 epoll 事件。</p>
<p data-nodeid="24551">在 for 循环中，会先将 epoll 中等待的文件描述符取出赋值给 nfds，然后再遍历 nfds，执行回调函数。</p>
<h4 data-nodeid="24552">uv__run_closing_handles</h4>
<p data-nodeid="24553">遍历等待关闭的队列，关闭 stream、tcp、udp 等 handle，然后调用 handle 对应的 close_cb。代码如下：</p>
<pre class="lang-c++" data-nodeid="24554"><code data-language="c++"><span class="hljs-function"><span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">uv__run_closing_handles</span><span class="hljs-params">(<span class="hljs-keyword">uv_loop_t</span>*&nbsp;<span class="hljs-built_in">loop</span>)</span>&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">uv_handle_t</span>*&nbsp;p;
&nbsp;&nbsp;<span class="hljs-keyword">uv_handle_t</span>*&nbsp;q;
&nbsp;&nbsp;p&nbsp;=&nbsp;<span class="hljs-built_in">loop</span>-&gt;closing_handles;
&nbsp;&nbsp;<span class="hljs-built_in">loop</span>-&gt;closing_handles&nbsp;=&nbsp;<span class="hljs-literal">NULL</span>;
&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(p)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;q&nbsp;=&nbsp;p-&gt;next_closing;
&nbsp;&nbsp;&nbsp;&nbsp;uv__finish_close(p);
&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;q;
&nbsp;&nbsp;}
}
</code></pre>
<h3 data-nodeid="24555">process.nextTick 和 Promise</h3>
<p data-nodeid="24556">虽然 process.nextTick 和 Promise 都是异步 API，但并不属于事件轮询的一部分，它们都有各自的任务队列，在事件轮询的每个步骤完成后执行。所以当我们使用这两个异步 API 的时候要注意，如果在传入的回调函数中执行长任务或递归，则会导致事件轮询被阻塞，从而“饿死”I/O 操作。</p>
<p data-nodeid="24557">下面的代码就是通过递归调用 prcoess.nextTick 而导致 fs.readFile 的回调函数无法执行的例子。</p>
<pre class="lang-c++" data-nodeid="24558"><code data-language="c++">fs.readFile('config.json', (err, data) =&gt; {
  ...
})
const traverse = () =&gt; {
&nbsp; &nbsp;process.nextTick(traverse)
}
</code></pre>
<p data-nodeid="24559">要解决这个问题，可以使用 setImmediate 来替代，因为 setImmediate 会在事件轮询中执行回调函数队列。</p>
<p data-nodeid="24560">在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=180#/detail/pc?id=3180" data-nodeid="24748">09</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=180#/detail/pc?id=3180" data-nodeid="24751">|</a><a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=180#/detail/pc?id=3180" data-nodeid="24754">为什么代码没有按照编写顺序执行？</a>”中提到过，process.nextTick 任务队列优先级比 Promise 任务队列更高，具体的原因可以参看下面的代码：</p>
<pre class="lang-c++" data-nodeid="24561"><code data-language="c++"><span class="hljs-comment">// lib/internal/process/task_queues.js</span>
<span class="hljs-function">function&nbsp;<span class="hljs-title">processTicksAndRejections</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;let&nbsp;tock;
&nbsp;&nbsp;<span class="hljs-keyword">do</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(tock&nbsp;=&nbsp;<span class="hljs-built_in">queue</span>.shift())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;asyncId&nbsp;=&nbsp;tock[async_id_symbol];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;emitBefore(asyncId,&nbsp;tock[trigger_async_id_symbol],&nbsp;tock);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;callback&nbsp;=&nbsp;tock.callback;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(tock.args&nbsp;===&nbsp;undefined)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;callback();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span>&nbsp;args&nbsp;=&nbsp;tock.args;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">switch</span>&nbsp;(args.length)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-number">1</span>:&nbsp;callback(args[<span class="hljs-number">0</span>]);&nbsp;<span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-number">2</span>:&nbsp;callback(args[<span class="hljs-number">0</span>],&nbsp;args[<span class="hljs-number">1</span>]);&nbsp;<span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-number">3</span>:&nbsp;callback(args[<span class="hljs-number">0</span>],&nbsp;args[<span class="hljs-number">1</span>],&nbsp;args[<span class="hljs-number">2</span>]);&nbsp;<span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;<span class="hljs-number">4</span>:&nbsp;callback(args[<span class="hljs-number">0</span>],&nbsp;args[<span class="hljs-number">1</span>],&nbsp;args[<span class="hljs-number">2</span>],&nbsp;args[<span class="hljs-number">3</span>]);&nbsp;<span class="hljs-keyword">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">default</span>:&nbsp;callback(...args);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;finally&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(destroyHooksExist())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;emitDestroy(asyncId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;emitAfter(asyncId);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;runMicrotasks();
&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">while</span>&nbsp;(!<span class="hljs-built_in">queue</span>.isEmpty()&nbsp;||&nbsp;processPromiseRejections());
&nbsp;&nbsp;setHasTickScheduled(<span class="hljs-literal">false</span>);
&nbsp;&nbsp;setHasRejectionToWarn(<span class="hljs-literal">false</span>);
}
</code></pre>
<p data-nodeid="24562">从 processTicksAndRejections() 函数中可以看出，首先通过 while 循环取出 queue 队列的回调函数，而这个 queue 队列中的回调函数就是通过 process.nextTick来添加的。当 while 循环结束后才调用 runMicrotasks() 函数执行 Promise 的回调函数。</p>
<h3 data-nodeid="24563">总结</h3>
<p data-nodeid="24564">这一课时我们主要分析了 Node.js 的核心依赖 libuv。libuv 的结构可以分两部分，一部分是网络 I/O，底层实现会根据不同操作系统依赖不同的系统 API，另一部分是文件 I/O、DNS、用户代码，这一部分采用线程池来处理。</p>
<p data-nodeid="24565">libuv 处理异步操作的核心机制是事件轮询，事件轮询分成若干步骤，大致操作是遍历并执行队列中的回调函数。</p>
<p data-nodeid="24566">最后提到处理异步的 API process.nextTick 和 Promise 不属于事件轮询，使用不当则会导致事件轮询阻塞，其中一种解决方式就是使用 setImmediate 来替代。</p>
<p data-nodeid="32827">最后布置一道思考题：尝试着阅读一下 libuv 的源码，看看能不能找出 setTimeout 对应的底层实现原理，然后把你的发现写在留言区和大家一起分享交流。</p>

---

### 精选评论

##### **贤：
> 通过递归调用 prcoess.nextTick 而导致 fs.readFile 的回调函数无法执行的例子这个例子代码，不知道为何会阻塞。看traverse都没有执行的样子

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为会在 nextTick 队列中不断执行 traverse 函数，可简单的理解为  function traverse() {traverse()}。

