<p data-nodeid="15396" class="">我们总是想方设法地提升系统的性能。操作系统层面不能给予处理业务逻辑太多帮助，但对于 I/O 性能，操作系统可以通过底层的优化，帮助应用做到极致。</p>
<p data-nodeid="15397">这一讲我将和你一起讨论 I/O 模型。为了引发你更多的思考，我将同步/异步、阻塞/非阻塞等概念滞后讲解。<strong data-nodeid="15502">我们先回到一个最基本的问题：如果有一台服务器，需要响应大量的请求，操作系统如何去架构以适应这样高并发的诉求</strong>。</p>
<p data-nodeid="15398">说到架构，就离不开操作系统提供给应用程序的系统调用。我们今天要介绍的 select/poll/epoll&nbsp;刚好是操作系统提供给应用的三类处理 I/O 的系统调用。这三类系统调用有非常强的代表性，这一讲我会围绕它们，以及处理并发和 I/O 多路复用，为你讲解操作系统的 I/O 模型。</p>
<h3 data-nodeid="15399">从网卡到操作系统</h3>
<p data-nodeid="15400">为了弄清楚高并发网络场景是如何处理的，我们先来看一个最基本的内容：<strong data-nodeid="15510">当数据到达网卡之后，操作系统会做哪些事情</strong>？</p>
<p data-nodeid="15401">网络数据到达网卡之后，首先需要把数据拷贝到内存。拷贝到内存的工作往往不需要消耗 CPU 资源，而是通过 DMA 模块直接进行内存映射。之所以这样做，是因为网卡没有大量的内存空间，只能做简单的缓冲，所以必须赶紧将它们保存下来。</p>
<p data-nodeid="15402" class="">Linux 中用一个双向链表作为缓冲区，你可以观察下图中的 Buffer，看上去像一个有很多个凹槽的线性结构，每个凹槽（节点）可以存储一个封包，这个封包可以从网络层看（IP 封包），也可以从传输层看（TCP 封包）。操作系统不断地从 Buffer 中取出数据，数据通过一个协议栈，你可以把它理解成很多个协议的集合。协议栈中数据封包找到对应的协议程序处理完之后，就会形成 Socket 文件。</p>
<p data-nodeid="15403" class=""><img src="https://s0.lgstatic.com/i/image2/M01/05/F3/Cip5yGABb8uAECMGAAERrnFoSrI090.png" alt="1111.png" data-nodeid="15515">!</p>
<p data-nodeid="15404" class="te-preview-highlight">如果高并发的请求量级实在太大，有可能把 Buffer 占满，此时，操作系统就会拒绝服务。网络上有一种著名的攻击叫作<strong data-nodeid="15526">拒绝服务攻击</strong>，就是利用的这个原理。<strong data-nodeid="15527">操作系统拒绝服务，实际上是一种保护策略。通过拒绝服务，避免系统内部应用因为并发量太大而雪崩</strong>。</p>
<p data-nodeid="15405">如上图所示，传入网卡的数据被我称为 Frames。一个 Frame 是数据链路层的传输单位（或封包）。现代的网卡通常使用 DMA 技术，将 Frame&nbsp;写入缓冲区（Buffer），然后在触发 CPU 中断交给操作系统处理。操作系统从缓冲区中不断取出 Frame，通过协进栈（具体的协议）进行还原。</p>
<p data-nodeid="15406">在 UNIX 系的操作系统中，一个 Socket 文件内部类似一个双向的管道。因此，非常适用于进程间通信。在网络当中，本质上并没有发生变化。网络中的 Socket 一端连接 Buffer，&nbsp;一端连接应用——也就是进程。网卡的数据会进入 Buffer，Buffer 经过协议栈的处理形成 Socket 结构。通过这样的设计，进程读取 Socket 文件，可以从 Buffer 中对应节点读走数据。</p>
<p data-nodeid="15407">对于 TCP 协议，Socket 文件可以用源端口、目标端口、源 IP、目标 IP 进行区别。不同的 Socket 文件，对应着 Buffer 中的不同节点。进程们读取数据的时候从 Buffer 中读取，写入数据的时候向 Buffer 中写入。通过这样一种结构，无论是读和写，进程都可以快速定位到自己对应的节点。</p>
<p data-nodeid="15408">以上就是我们对操作系统和网络接口交互的一个基本讨论。接下来，我们讨论一下作为一个编程模型的 Socket。</p>
<h3 data-nodeid="15409">Socket 编程模型</h3>
<p data-nodeid="15410">通过前面讲述，我们知道 Socket&nbsp;在操作系统中，有一个非常具体的从 Buffer 到文件的实现。但是对于进程而言，Socket 更多是一种编程的模型。接下来我们讨论作为编程模型的 Socket。</p>
<p data-nodeid="15411"><img src="https://s0.lgstatic.com/i/image/M00/8D/F3/Ciqc1GABP3OAHezqAABndlGAu9c457.png" alt="Lark20210115-150702.png" data-nodeid="15536"></p>
<p data-nodeid="15412">如上图所示，Socket&nbsp;连接了应用和协议，如果应用层的程序想要传输数据，就创建一个 Socket。应用向 Socket 中写入数据，相当于将数据发送给了另一个应用。应用从 Socket 中读取数据，相当于接收另一个应用发送的数据。而具体的操作就是由 Socket 进行封装。具体来说，<strong data-nodeid="15542">对于 UNIX 系的操作系统，是利用 Socket 文件系统，Socket 是一种特殊的文件——每个都是一个双向的管道。一端是应用，一端是缓冲</strong>区。</p>
<p data-nodeid="15413">那么作为一个服务端的应用，如何知道有哪些 Socket 呢？也就是，哪些客户端连接过来了呢？这是就需要一种特殊类型的 Socket，也就是服务端 Socket 文件。</p>
<p data-nodeid="15414"><img src="https://s0.lgstatic.com/i/image/M00/8D/F3/Ciqc1GABP3qADKbBAAB564sk120429.png" alt="Lark20210115-150706.png" data-nodeid="15546"></p>
<p data-nodeid="15415">如上图所示，当有客户端连接服务端时，服务端 Socket 文件中会写入这个客户端 Socket 的文件描述符。进程可以通过 accept() 方法，从服务端 Socket 文件中读出客户端的 Socket 文件描述符，从而拿到客户端的 Socket 文件。</p>
<p data-nodeid="15416">程序员实现一个网络服务器的时候，会先手动去创建一个服务端 Socket 文件。服务端的 Socket 文件依然会存在操作系统内核之中，并且会绑定到某个 IP 地址和端口上。以后凡是发送到这台机器、目标 IP 地址和端口号的连接请求，在形成了客户端 Socket 文件之后，文件的文件描述符都会被写入到服务端的 Socket 文件中。应用只要调用 accept 方法，就可以拿到这些客户端的 Socket 文件描述符，这样服务端的应用就可以方便地知道有哪些客户端连接了进来。</p>
<p data-nodeid="15417">而每个客户端对这个应用而言，都是一个文件描述符。如果需要读取某个客户端的数据，就读取这个客户端对应的 Socket 文件。如果要向某个特定的客户端发送数据，就写入这个客户端的 Socket 文件。</p>
<p data-nodeid="15418">以上就是 Socket 的编程模型。</p>
<h3 data-nodeid="15419">I/O 多路复用</h3>
<p data-nodeid="15420">在上面的讨论当中，进程拿到了它关注的所有 Socket，也称作关注的集合（Intersting Set）。如下图所示，这种过程相当于进程从所有的 Socket 中，筛选出了自己关注的一个子集，但是这时还有一个问题没有解决：<strong data-nodeid="15557">进程如何监听关注集合的状态变化，比如说在有数据进来，如何通知到这个进程</strong>？</p>
<p data-nodeid="15421"><img src="https://s0.lgstatic.com/i/image/M00/8D/F3/Ciqc1GABP4OAdKBcAACAbVkbI0g191.png" alt="Lark20210115-150708.png" data-nodeid="15560"></p>
<p data-nodeid="15422">其实更准确地说，一个线程需要处理所有关注的 Socket 产生的变化，或者说消息。实际上一个线程要处理很多个文件的 I/O。<strong data-nodeid="15566">所有关注的 Socket 状态发生了变化，都由一个线程去处理，构成了 I/O 的多路复用问题</strong>。如下图所示：</p>
<p data-nodeid="15423"><img src="https://s0.lgstatic.com/i/image/M00/8D/F3/Ciqc1GABP4uAW8-dAAB_SubmZ4Q301.png" alt="Lark20210115-150711.png" data-nodeid="15569"></p>
<p data-nodeid="15424">处理 I/O 多路复用的问题，需要操作系统提供内核级别的支持。Linux 下有三种提供 I/O 多路复用的 API，分别是：</p>
<ul data-nodeid="15425">
<li data-nodeid="15426">
<p data-nodeid="15427">select</p>
</li>
<li data-nodeid="15428">
<p data-nodeid="15429">poll</p>
</li>
<li data-nodeid="15430">
<p data-nodeid="15431">epoll</p>
</li>
</ul>
<p data-nodeid="15432">如下图所示，内核了解网络的状态。因此不难知道具体发生了什么消息，比如内核知道某个 Socket 文件状态发生了变化。但是内核如何知道该把哪个消息给哪个进程呢？</p>
<p data-nodeid="15433"><img src="https://s0.lgstatic.com/i/image/M00/8D/F3/Ciqc1GABP5KAVSWVAAFSurtl2bU931.png" alt="Lark20210115-150654.png" data-nodeid="15577"></p>
<p data-nodeid="15434"><strong data-nodeid="15586">一个 Socket 文件，可以由多个进程使用；而一个进程，也可以使用多个 Socket 文件</strong>。进程和 Socket 之间是多对多的关系。<strong data-nodeid="15587">另一方面，一个 Socket 也会有不同的事件类型</strong>。因此操作系统很难判断，将哪样的事件给哪个进程。</p>
<p data-nodeid="15435">这样<strong data-nodeid="15601">在进程内部就需要一个数据结构来描述自己会关注哪些 Socket 文件的哪些事件（读、写、异常等</strong>）。通常有两种考虑方向，<strong data-nodeid="15602">一种是利用线性结构</strong>，比如说数组、链表等，这类结构的查询需要遍历。每次内核产生一种消息，就遍历这个线性结构。看看这个消息是不是进程关注的？<strong data-nodeid="15603">另一种是索引结构</strong>，内核发生了消息可以通过索引结构马上知道这个消息进程关不关注。</p>
<h4 data-nodeid="15436">select()</h4>
<p data-nodeid="15437">select 和 poll 都采用线性结构，select 允许用户传入 3 个集合。如下面这段程序所示：</p>
<pre class="lang-c++" data-nodeid="15438"><code data-language="c++">fd_set read_fd_set, write_fd_set, error_fd_set;
<span class="hljs-keyword">while</span>(<span class="hljs-literal">true</span>) {
  select(..., &amp;read_fd_set, &amp;write_fd_set, &amp;error_fd_set); 
}
</code></pre>
<p data-nodeid="15439"><strong data-nodeid="15616">每次 select 操作会阻塞当前线程，在阻塞期间所有操作系统产生的每个消息，都会通过遍历的手段查看是否在 3 个集合当中</strong>。上面程序<code data-backticks="1" data-nodeid="15610">read_fd_set</code>中放入的是当数据可以读取时进程关心的 Socket；<code data-backticks="1" data-nodeid="15612">write_fd_set</code>是当数据可以写入时进程关心的 Socket；<code data-backticks="1" data-nodeid="15614">error_fd_set</code>是当发生异常时进程关心的 Socket。</p>
<p data-nodeid="15440">**用户程序可以根据不同集合中是否有某个 Socket 判断发生的消息类型，**程序如下所示：</p>
<pre class="lang-c++" data-nodeid="15441"><code data-language="c++">fd_set read_fd_set, write_fd_set, error_fd_set;
<span class="hljs-keyword">while</span>(<span class="hljs-literal">true</span>) {
  select(..., &amp;read_fd_set, &amp;write_fd_set, &amp;error_fd_set); 
  <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; FD_SETSIZE; ++i)
        <span class="hljs-keyword">if</span> (FD_ISSET (i, &amp;read_fd_set)){
          <span class="hljs-comment">// Socket可以读取</span>
        } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(FD_ISSET(i, &amp;write_fd_set)) {
          <span class="hljs-comment">// Socket可以写入</span>
        } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(FD_ISSET(i, &amp;error_fd_set)) {
          <span class="hljs-comment">// Socket发生错误</span>
        } 
}
</code></pre>
<p data-nodeid="15442">上面程序中的 FD_SETSIZE 是一个系统的默认设置，通常是 1024。可以看出，select 模式能够一次处理的文件描述符是有上限的，也就是 FD_SETSIZE。当并发请求过多的时候， select 就无能为力了。但是对单台机器而言，1024 个并发已经是一个非常大的流量了。</p>
<p data-nodeid="15443">接下来我给出一个完整的、用 select 实现的服务端程序供你参考，如下所示：</p>
<pre class="lang-c++" data-nodeid="15444"><code data-language="c++"><span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdio.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;errno.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdlib.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;unistd.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;sys/types.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;sys/Socket.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;netinet/in.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;netdb.h&gt;</span></span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> PORT    5555</span>
<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> MAXMSG  512</span>
<span class="hljs-function"><span class="hljs-keyword">int</span>
<span class="hljs-title">read_from_client</span> <span class="hljs-params">(<span class="hljs-keyword">int</span> filedes)</span>
</span>{
  <span class="hljs-keyword">char</span> <span class="hljs-built_in">buffer</span>[MAXMSG];
  <span class="hljs-keyword">int</span> nbytes;
  nbytes = <span class="hljs-built_in">read</span> (filedes, <span class="hljs-built_in">buffer</span>, MAXMSG);
  <span class="hljs-keyword">if</span> (nbytes &lt; <span class="hljs-number">0</span>)
    {
      <span class="hljs-comment">/* Read error. */</span>
      perror (<span class="hljs-string">"read"</span>);
      <span class="hljs-built_in">exit</span> (EXIT_FAILURE);
    }
  <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (nbytes == <span class="hljs-number">0</span>)
    <span class="hljs-comment">/* End-of-file. */</span>
    <span class="hljs-keyword">return</span> <span class="hljs-number">-1</span>;
  <span class="hljs-keyword">else</span>
    {
      <span class="hljs-comment">/* Data read. */</span>
      <span class="hljs-built_in">fprintf</span> (<span class="hljs-built_in">stderr</span>, <span class="hljs-string">"Server: got message: `%s'\n"</span>, <span class="hljs-built_in">buffer</span>);
      <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
    }
}
<span class="hljs-function"><span class="hljs-keyword">int</span>
<span class="hljs-title">main</span> <span class="hljs-params">(<span class="hljs-keyword">void</span>)</span>
</span>{
  <span class="hljs-function"><span class="hljs-keyword">extern</span> <span class="hljs-keyword">int</span> <span class="hljs-title">make_Socket</span> <span class="hljs-params">(<span class="hljs-keyword">uint16_t</span> port)</span></span>;
  <span class="hljs-keyword">int</span> sock;
  fd_set active_fd_set, read_fd_set;
  <span class="hljs-keyword">int</span> i;
  <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">sockaddr_in</span> <span class="hljs-title">clientname</span>;</span>
  <span class="hljs-keyword">size_t</span> <span class="hljs-built_in">size</span>;
  <span class="hljs-comment">/* Create the Socket and set it up to accept connections. */</span>
  sock = make_Socket (PORT);
  <span class="hljs-keyword">if</span> (<span class="hljs-built_in">listen</span> (sock, <span class="hljs-number">1</span>) &lt; <span class="hljs-number">0</span>)
    {
      perror (<span class="hljs-string">"listen"</span>);
      <span class="hljs-built_in">exit</span> (EXIT_FAILURE);
    }
  <span class="hljs-comment">/* Initialize the set of active Sockets. */</span>
  FD_ZERO (&amp;active_fd_set);
  FD_SET (sock, &amp;active_fd_set);
  <span class="hljs-keyword">while</span> (<span class="hljs-number">1</span>)
    {
      <span class="hljs-comment">/* Block until input arrives on one or more active Sockets. */</span>
      read_fd_set = active_fd_set;
      <span class="hljs-keyword">if</span> (select (FD_SETSIZE, &amp;read_fd_set, <span class="hljs-literal">NULL</span>, <span class="hljs-literal">NULL</span>, <span class="hljs-literal">NULL</span>) &lt; <span class="hljs-number">0</span>)
        {
          perror (<span class="hljs-string">"select"</span>);
          <span class="hljs-built_in">exit</span> (EXIT_FAILURE);
        }
      <span class="hljs-comment">/* Service all the Sockets with input pending. */</span>
      <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; FD_SETSIZE; ++i)
        <span class="hljs-keyword">if</span> (FD_ISSET (i, &amp;read_fd_set))
          {
            <span class="hljs-keyword">if</span> (i == sock)
              {
                <span class="hljs-comment">/* Connection request on original Socket. */</span>
                <span class="hljs-keyword">int</span> <span class="hljs-keyword">new</span>;
                <span class="hljs-built_in">size</span> = <span class="hljs-keyword">sizeof</span> (clientname);
                <span class="hljs-keyword">new</span> = accept (sock,
                              (struct sockaddr *) &amp;clientname,
                              &amp;<span class="hljs-built_in">size</span>);
                <span class="hljs-keyword">if</span> (<span class="hljs-keyword">new</span> &lt; <span class="hljs-number">0</span>)
                  {
                    perror (<span class="hljs-string">"accept"</span>);
                    <span class="hljs-built_in">exit</span> (EXIT_FAILURE);
                  }
                <span class="hljs-built_in">fprintf</span> (<span class="hljs-built_in">stderr</span>,
                         <span class="hljs-string">"Server: connect from host %s, port %hd.\n"</span>,
                         inet_ntoa (clientname.sin_addr),
                         ntohs (clientname.sin_port));
                FD_SET (<span class="hljs-keyword">new</span>, &amp;active_fd_set);
              }
            <span class="hljs-keyword">else</span>
              {
                <span class="hljs-comment">/* Data arriving on an already-connected Socket. */</span>
                <span class="hljs-keyword">if</span> (read_from_client (i) &lt; <span class="hljs-number">0</span>)
                  {
                    <span class="hljs-built_in">close</span> (i);
                    FD_CLR (i, &amp;active_fd_set);
                  }
              }
          }
    }
}
</code></pre>
<h4 data-nodeid="15445">poll()</h4>
<p data-nodeid="15446">从写程序的角度来看，select 并不是一个很好的编程模型。一个好的编程模型应该直达本质，当网络请求发生状态变化的时候，核心是会发生事件。<strong data-nodeid="15635">一个好的编程模型应该是直接抽象成消息：用户不需要用 select 来设置自己的集合，而是可以通过系统的 API 直接拿到对应的消息，从而处理对应的文件描述符</strong>。</p>
<p data-nodeid="15447">比如下面这段伪代码就是一个更好的编程模型，具体的分析如下：</p>
<ul data-nodeid="15448">
<li data-nodeid="15449">
<p data-nodeid="15450">poll 是一个阻塞调用，它将某段时间内操作系统内发生的且进程关注的消息告知用户程序；</p>
</li>
<li data-nodeid="15451">
<p data-nodeid="15452">用户程序通过直接调用 poll 函数拿到消息；</p>
</li>
<li data-nodeid="15453">
<p data-nodeid="15454">poll 函数的第一个参数告知内核 poll 关注哪些 Socket 及消息类型；</p>
</li>
<li data-nodeid="15455">
<p data-nodeid="15456">poll 调用后，经过一段时间的等待（阻塞），就拿到了是一个消息的数组；</p>
</li>
<li data-nodeid="15457">
<p data-nodeid="15458">通过遍历这个数组中的消息，能够知道关联的文件描述符和消息的类型；</p>
</li>
<li data-nodeid="15459">
<p data-nodeid="15460">通过消息类型判断接下来该进行读取还是写入操作；</p>
</li>
<li data-nodeid="15461">
<p data-nodeid="15462">通过文件描述符，可以进行实际地读、写、错误处理。</p>
</li>
</ul>
<pre class="lang-c++" data-nodeid="15463"><code data-language="c++"><span class="hljs-keyword">while</span>(<span class="hljs-literal">true</span>) {
  events = poll(fds, ...)
  <span class="hljs-keyword">for</span>(evt in events) {
    fd = evt.fd;
    type = evt.revents;
    <span class="hljs-keyword">if</span>(type &amp; POLLIN ) {
       <span class="hljs-comment">// 有数据需要读，读取fd中的数据</span>
    } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(type &amp; POLLOUT) {
       <span class="hljs-comment">// 可以写入数据</span>
    } 
    <span class="hljs-keyword">else</span> ...
  }
}
</code></pre>
<p data-nodeid="15464">poll 虽然优化了编程模型，但是从性能角度分析，它和 select 差距不大。因为内核在产生一个消息之后，依然需要遍历 poll 关注的所有文件描述符来确定这条消息是否跟用户程序相关。</p>
<h4 data-nodeid="15465">epoll</h4>
<p data-nodeid="15466">为了解决上述问题，<strong data-nodeid="15651">epoll 通过更好的方案实现了从操作系统订阅消息。epoll 将进程关注的文件描述符存入一棵二叉搜索树，通常是红黑树的实现</strong>。在这棵红黑树当中，Key 是 Socket 的编号，值是这个 Socket 关注的消息。因此，当内核发生了一个事件：比如 Socket 编号 1000 可以读取。这个时候，可以马上从红黑树中找到进程是否关注这个事件。</p>
<p data-nodeid="15467"><strong data-nodeid="15661">另外当有关注的事件发生时，epoll 会先放到一个队列当中。当用户调用</strong><code data-backticks="1" data-nodeid="15655">epoll_wait</code>时候，就会从队列中返回一个消息。epoll 函数本身是一个构造函数，只用来创建红黑树和队列结构。<code data-backticks="1" data-nodeid="15657">epoll_wait</code>调用后，如果队列中没有消息，也可以马上返回。因此<code data-backticks="1" data-nodeid="15659">epoll</code>是一个非阻塞模型。</p>
<p data-nodeid="15468"><strong data-nodeid="15670">总结一下，select/poll 是阻塞模型，epoll 是非阻塞模型</strong>。<strong data-nodeid="15671">当然，并不是说非阻塞模型性能就更好。在多数情况下，epoll 性能更好是因为内部有红黑树的实现</strong>。</p>
<p data-nodeid="15469">最后我再贴一段用 epoll 实现的 Socket 服务给你做参考，这段程序的作者将这段代码放到了 Public Domain，你以后看到公有领域的代码可以放心地使用。</p>
<p data-nodeid="15470">下面这段程序跟之前 select 的原理一致，对于每一个新的客户端连接，都使用 accept 拿到这个连接的文件描述符，并且创建一个客户端的 Socket。然后通过<code data-backticks="1" data-nodeid="15674">epoll_ctl</code>将客户端的文件描述符和关注的消息类型放入 epoll 的红黑树。操作系统每次监测到一个新的消息产生，就会通过红黑树对比这个消息是不是进程关注的（当然这段代码你看不到，因为它在内核程序中）。</p>
<p data-nodeid="15471"><strong data-nodeid="15680">非阻塞模型的核心价值，并不是性能更好。当真的高并发来临的时候，所有的 CPU 资源，所有的网络资源可能都会被用完。这个时候无论是阻塞还是非阻塞，结果都不会相差太大</strong>。（前提是程序没有写错）。</p>
<p data-nodeid="15472"><code data-backticks="1" data-nodeid="15681">epoll</code>有 2 个最大的优势：</p>
<ol data-nodeid="15473">
<li data-nodeid="15474">
<p data-nodeid="15475">内部使用红黑树减少了内核的比较操作；</p>
</li>
<li data-nodeid="15476">
<p data-nodeid="15477">对于程序员而言，非阻塞的模型更容易处理各种各样的情况。程序员习惯了写出每一条语句就可以马上得到结果，这样不容易出 Bug。</p>
</li>
</ol>
<pre class="lang-c++" data-nodeid="15478"><code data-language="c++"><span class="hljs-comment">// Asynchronous Socket server - accepting multiple clients concurrently,</span>
	<span class="hljs-comment">// multiplexing the connections with epoll.</span>
	<span class="hljs-comment">//</span>
	<span class="hljs-comment">// Eli Bendersky [http://eli.thegreenplace.net]</span>
	<span class="hljs-comment">// This code is in the public domain.</span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;assert.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;errno.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdbool.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdint.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdio.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;stdlib.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;string.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;sys/epoll.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;sys/Socket.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;sys/types.h&gt;</span></span>
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">&lt;unistd.h&gt;</span></span>
	
	<span class="hljs-meta">#<span class="hljs-meta-keyword">include</span> <span class="hljs-meta-string">"utils.h"</span></span>
	
	<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> MAXFDS 16 * 1024</span>
	
	<span class="hljs-keyword">typedef</span> <span class="hljs-keyword">enum</span> { INITIAL_ACK, WAIT_FOR_MSG, IN_MSG } ProcessingState;
	
	<span class="hljs-meta">#<span class="hljs-meta-keyword">define</span> SENDBUF_SIZE 1024</span>
	
	<span class="hljs-keyword">typedef</span> <span class="hljs-class"><span class="hljs-keyword">struct</span> {</span>
	  ProcessingState state;
	  <span class="hljs-keyword">uint8_t</span> sendbuf[SENDBUF_SIZE];
	  <span class="hljs-keyword">int</span> sendbuf_end;
	  <span class="hljs-keyword">int</span> sendptr;
	} <span class="hljs-keyword">peer_state_t</span>;
	
	<span class="hljs-comment">// Each peer is globally identified by the file descriptor (fd) it's connected</span>
	<span class="hljs-comment">// on. As long as the peer is connected, the fd is unique to it. When a peer</span>
	<span class="hljs-comment">// disconnects, a new peer may connect and get the same fd. on_peer_connected</span>
	<span class="hljs-comment">// should initialize the state properly to remove any trace of the old peer on</span>
	<span class="hljs-comment">// the same fd.</span>
	<span class="hljs-keyword">peer_state_t</span> global_state[MAXFDS];
	
	<span class="hljs-comment">// Callbacks (on_XXX functions) return this status to the main loop; the status</span>
	<span class="hljs-comment">// instructs the loop about the next steps for the fd for which the callback was</span>
	<span class="hljs-comment">// invoked.</span>
	<span class="hljs-comment">// want_read=true means we want to keep monitoring this fd for reading.</span>
	<span class="hljs-comment">// want_write=true means we want to keep monitoring this fd for writing.</span>
	<span class="hljs-comment">// When both are false it means the fd is no longer needed and can be closed.</span>
	<span class="hljs-keyword">typedef</span> <span class="hljs-class"><span class="hljs-keyword">struct</span> {</span>
	  <span class="hljs-keyword">bool</span> want_read;
	  <span class="hljs-keyword">bool</span> want_write;
	} <span class="hljs-keyword">fd_status_t</span>;
	
	<span class="hljs-comment">// These constants make creating fd_status_t values less verbose.</span>
	<span class="hljs-keyword">const</span> <span class="hljs-keyword">fd_status_t</span> fd_status_R = {.want_read = <span class="hljs-literal">true</span>, .want_write = <span class="hljs-literal">false</span>};
	<span class="hljs-keyword">const</span> <span class="hljs-keyword">fd_status_t</span> fd_status_W = {.want_read = <span class="hljs-literal">false</span>, .want_write = <span class="hljs-literal">true</span>};
	<span class="hljs-keyword">const</span> <span class="hljs-keyword">fd_status_t</span> fd_status_RW = {.want_read = <span class="hljs-literal">true</span>, .want_write = <span class="hljs-literal">true</span>};
	<span class="hljs-keyword">const</span> <span class="hljs-keyword">fd_status_t</span> fd_status_NORW = {.want_read = <span class="hljs-literal">false</span>, .want_write = <span class="hljs-literal">false</span>};
	
	<span class="hljs-function"><span class="hljs-keyword">fd_status_t</span> <span class="hljs-title">on_peer_connected</span><span class="hljs-params">(<span class="hljs-keyword">int</span> sockfd, <span class="hljs-keyword">const</span> struct sockaddr_in* peer_addr,
	                              <span class="hljs-keyword">socklen_t</span> peer_addr_len)</span> </span>{
	  assert(sockfd &lt; MAXFDS);
	  report_peer_connected(peer_addr, peer_addr_len);
	
	  <span class="hljs-comment">// Initialize state to send back a '*' to the peer immediately.</span>
	  <span class="hljs-keyword">peer_state_t</span>* peerstate = &amp;global_state[sockfd];
	  peerstate-&gt;state = INITIAL_ACK;
	  peerstate-&gt;sendbuf[<span class="hljs-number">0</span>] = <span class="hljs-string">'*'</span>;
	  peerstate-&gt;sendptr = <span class="hljs-number">0</span>;
	  peerstate-&gt;sendbuf_end = <span class="hljs-number">1</span>;
	
	  <span class="hljs-comment">// Signal that this Socket is ready for writing now.</span>
	  <span class="hljs-keyword">return</span> fd_status_W;
	}
	
	<span class="hljs-function"><span class="hljs-keyword">fd_status_t</span> <span class="hljs-title">on_peer_ready_recv</span><span class="hljs-params">(<span class="hljs-keyword">int</span> sockfd)</span> </span>{
	  assert(sockfd &lt; MAXFDS);
	  <span class="hljs-keyword">peer_state_t</span>* peerstate = &amp;global_state[sockfd];
	
	  <span class="hljs-keyword">if</span> (peerstate-&gt;state == INITIAL_ACK ||
	      peerstate-&gt;sendptr &lt; peerstate-&gt;sendbuf_end) {
	    <span class="hljs-comment">// Until the initial ACK has been sent to the peer, there's nothing we</span>
	    <span class="hljs-comment">// want to receive. Also, wait until all data staged for sending is sent to</span>
	    <span class="hljs-comment">// receive more data.</span>
	    <span class="hljs-keyword">return</span> fd_status_W;
	  }
	
	  <span class="hljs-keyword">uint8_t</span> buf[<span class="hljs-number">1024</span>];
	  <span class="hljs-keyword">int</span> nbytes = recv(sockfd, buf, <span class="hljs-keyword">sizeof</span> buf, <span class="hljs-number">0</span>);
	  <span class="hljs-keyword">if</span> (nbytes == <span class="hljs-number">0</span>) {
	    <span class="hljs-comment">// The peer disconnected.</span>
	    <span class="hljs-keyword">return</span> fd_status_NORW;
	  } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (nbytes &lt; <span class="hljs-number">0</span>) {
	    <span class="hljs-keyword">if</span> (errno == EAGAIN || errno == EWOULDBLOCK) {
	      <span class="hljs-comment">// The Socket is not *really* ready for recv; wait until it is.</span>
	      <span class="hljs-keyword">return</span> fd_status_R;
	    } <span class="hljs-keyword">else</span> {
	      perror_die(<span class="hljs-string">"recv"</span>);
	    }
	  }
	  <span class="hljs-keyword">bool</span> ready_to_send = <span class="hljs-literal">false</span>;
	  <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; nbytes; ++i) {
	    <span class="hljs-keyword">switch</span> (peerstate-&gt;state) {
	    <span class="hljs-keyword">case</span> INITIAL_ACK:
	      assert(<span class="hljs-number">0</span> &amp;&amp; <span class="hljs-string">"can't reach here"</span>);
	      <span class="hljs-keyword">break</span>;
	    <span class="hljs-keyword">case</span> WAIT_FOR_MSG:
	      <span class="hljs-keyword">if</span> (buf[i] == <span class="hljs-string">'^'</span>) {
	        peerstate-&gt;state = IN_MSG;
	      }
	      <span class="hljs-keyword">break</span>;
	    <span class="hljs-keyword">case</span> IN_MSG:
	      <span class="hljs-keyword">if</span> (buf[i] == <span class="hljs-string">'$'</span>) {
	        peerstate-&gt;state = WAIT_FOR_MSG;
	      } <span class="hljs-keyword">else</span> {
	        assert(peerstate-&gt;sendbuf_end &lt; SENDBUF_SIZE);
	        peerstate-&gt;sendbuf[peerstate-&gt;sendbuf_end++] = buf[i] + <span class="hljs-number">1</span>;
	        ready_to_send = <span class="hljs-literal">true</span>;
	      }
	      <span class="hljs-keyword">break</span>;
	    }
	  }
	  <span class="hljs-comment">// Report reading readiness iff there's nothing to send to the peer as a</span>
	  <span class="hljs-comment">// result of the latest recv.</span>
	  <span class="hljs-keyword">return</span> (<span class="hljs-keyword">fd_status_t</span>){.want_read = !ready_to_send,
	                       .want_write = ready_to_send};
	}
	
	<span class="hljs-function"><span class="hljs-keyword">fd_status_t</span> <span class="hljs-title">on_peer_ready_send</span><span class="hljs-params">(<span class="hljs-keyword">int</span> sockfd)</span> </span>{
	  assert(sockfd &lt; MAXFDS);
	  <span class="hljs-keyword">peer_state_t</span>* peerstate = &amp;global_state[sockfd];
	
	  <span class="hljs-keyword">if</span> (peerstate-&gt;sendptr &gt;= peerstate-&gt;sendbuf_end) {
	    <span class="hljs-comment">// Nothing to send.</span>
	    <span class="hljs-keyword">return</span> fd_status_RW;
	  }
	  <span class="hljs-keyword">int</span> sendlen = peerstate-&gt;sendbuf_end - peerstate-&gt;sendptr;
	  <span class="hljs-keyword">int</span> nsent = send(sockfd, &amp;peerstate-&gt;sendbuf[peerstate-&gt;sendptr], sendlen, <span class="hljs-number">0</span>);
	  <span class="hljs-keyword">if</span> (nsent == <span class="hljs-number">-1</span>) {
	    <span class="hljs-keyword">if</span> (errno == EAGAIN || errno == EWOULDBLOCK) {
	      <span class="hljs-keyword">return</span> fd_status_W;
	    } <span class="hljs-keyword">else</span> {
	      perror_die(<span class="hljs-string">"send"</span>);
	    }
	  }
	  <span class="hljs-keyword">if</span> (nsent &lt; sendlen) {
	    peerstate-&gt;sendptr += nsent;
	    <span class="hljs-keyword">return</span> fd_status_W;
	  } <span class="hljs-keyword">else</span> {
	    <span class="hljs-comment">// Everything was sent successfully; reset the send queue.</span>
	    peerstate-&gt;sendptr = <span class="hljs-number">0</span>;
	    peerstate-&gt;sendbuf_end = <span class="hljs-number">0</span>;
	
	    <span class="hljs-comment">// Special-case state transition in if we were in INITIAL_ACK until now.</span>
	    <span class="hljs-keyword">if</span> (peerstate-&gt;state == INITIAL_ACK) {
	      peerstate-&gt;state = WAIT_FOR_MSG;
	    }
	
	    <span class="hljs-keyword">return</span> fd_status_R;
	  }
	}
	
	<span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">main</span><span class="hljs-params">(<span class="hljs-keyword">int</span> argc, <span class="hljs-keyword">const</span> <span class="hljs-keyword">char</span>** argv)</span> </span>{
	  setvbuf(<span class="hljs-built_in">stdout</span>, <span class="hljs-literal">NULL</span>, _IONBF, <span class="hljs-number">0</span>);
	
	  <span class="hljs-keyword">int</span> portnum = <span class="hljs-number">9090</span>;
	  <span class="hljs-keyword">if</span> (argc &gt;= <span class="hljs-number">2</span>) {
	    portnum = atoi(argv[<span class="hljs-number">1</span>]);
	  }
	  <span class="hljs-built_in">printf</span>(<span class="hljs-string">"Serving on port %d\n"</span>, portnum);
	
	  <span class="hljs-keyword">int</span> listener_sockfd = listen_inet_Socket(portnum);
	  make_Socket_non_blocking(listener_sockfd);
	
	  <span class="hljs-keyword">int</span> epollfd = epoll_create1(<span class="hljs-number">0</span>);
	  <span class="hljs-keyword">if</span> (epollfd &lt; <span class="hljs-number">0</span>) {
	    perror_die(<span class="hljs-string">"epoll_create1"</span>);
	  }
	
	  <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">epoll_event</span> <span class="hljs-title">accept_event</span>;</span>
	  accept_event.data.fd = listener_sockfd;
	  accept_event.events = EPOLLIN;
	  <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_ADD, listener_sockfd, &amp;accept_event) &lt; <span class="hljs-number">0</span>) {
	    perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_ADD"</span>);
	  }
	
	  <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">epoll_event</span>* <span class="hljs-title">events</span> = <span class="hljs-title">calloc</span>(<span class="hljs-title">MAXFDS</span>, <span class="hljs-title">sizeof</span>(<span class="hljs-title">struct</span> <span class="hljs-title">epoll_event</span>));</span>
	  <span class="hljs-keyword">if</span> (events == <span class="hljs-literal">NULL</span>) {
	    die(<span class="hljs-string">"Unable to allocate memory for epoll_events"</span>);
	  }
	
	  <span class="hljs-keyword">while</span> (<span class="hljs-number">1</span>) {
	    <span class="hljs-keyword">int</span> nready = epoll_wait(epollfd, events, MAXFDS, <span class="hljs-number">-1</span>);
	    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; nready; i++) {
	      <span class="hljs-keyword">if</span> (events[i].events &amp; EPOLLERR) {
	        perror_die(<span class="hljs-string">"epoll_wait returned EPOLLERR"</span>);
	      }
	
	      <span class="hljs-keyword">if</span> (events[i].data.fd == listener_sockfd) {
	        <span class="hljs-comment">// The listening Socket is ready; this means a new peer is connecting.</span>
	
	        <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">sockaddr_in</span> <span class="hljs-title">peer_addr</span>;</span>
	        <span class="hljs-keyword">socklen_t</span> peer_addr_len = <span class="hljs-keyword">sizeof</span>(peer_addr);
	        <span class="hljs-keyword">int</span> newsockfd = accept(listener_sockfd, (struct sockaddr*)&amp;peer_addr,
	                               &amp;peer_addr_len);
	        <span class="hljs-keyword">if</span> (newsockfd &lt; <span class="hljs-number">0</span>) {
	          <span class="hljs-keyword">if</span> (errno == EAGAIN || errno == EWOULDBLOCK) {
	            <span class="hljs-comment">// This can happen due to the nonblocking Socket mode; in this</span>
	            <span class="hljs-comment">// case don't do anything, but print a notice (since these events</span>
	            <span class="hljs-comment">// are extremely rare and interesting to observe...)</span>
	            <span class="hljs-built_in">printf</span>(<span class="hljs-string">"accept returned EAGAIN or EWOULDBLOCK\n"</span>);
	          } <span class="hljs-keyword">else</span> {
	            perror_die(<span class="hljs-string">"accept"</span>);
	          }
	        } <span class="hljs-keyword">else</span> {
	          make_Socket_non_blocking(newsockfd);
	          <span class="hljs-keyword">if</span> (newsockfd &gt;= MAXFDS) {
	            die(<span class="hljs-string">"Socket fd (%d) &gt;= MAXFDS (%d)"</span>, newsockfd, MAXFDS);
	          }
	
	          <span class="hljs-keyword">fd_status_t</span> status =
	              on_peer_connected(newsockfd, &amp;peer_addr, peer_addr_len);
	          <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">epoll_event</span> <span class="hljs-title">event</span> = {</span><span class="hljs-number">0</span>};
	          event.data.fd = newsockfd;
	          <span class="hljs-keyword">if</span> (status.want_read) {
	            event.events |= EPOLLIN;
	          }
	          <span class="hljs-keyword">if</span> (status.want_write) {
	            event.events |= EPOLLOUT;
	          }
	
	          <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_ADD, newsockfd, &amp;event) &lt; <span class="hljs-number">0</span>) {
	            perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_ADD"</span>);
	          }
	        }
	      } <span class="hljs-keyword">else</span> {
	        <span class="hljs-comment">// A peer Socket is ready.</span>
	        <span class="hljs-keyword">if</span> (events[i].events &amp; EPOLLIN) {
	          <span class="hljs-comment">// Ready for reading.</span>
	          <span class="hljs-keyword">int</span> fd = events[i].data.fd;
	          <span class="hljs-keyword">fd_status_t</span> status = on_peer_ready_recv(fd);
	          <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">epoll_event</span> <span class="hljs-title">event</span> = {</span><span class="hljs-number">0</span>};
	          event.data.fd = fd;
	          <span class="hljs-keyword">if</span> (status.want_read) {
	            event.events |= EPOLLIN;
	          }
	          <span class="hljs-keyword">if</span> (status.want_write) {
	            event.events |= EPOLLOUT;
	          }
	          <span class="hljs-keyword">if</span> (event.events == <span class="hljs-number">0</span>) {
	            <span class="hljs-built_in">printf</span>(<span class="hljs-string">"Socket %d closing\n"</span>, fd);
	            <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_DEL, fd, <span class="hljs-literal">NULL</span>) &lt; <span class="hljs-number">0</span>) {
	              perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_DEL"</span>);
	            }
	            <span class="hljs-built_in">close</span>(fd);
	          } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_MOD, fd, &amp;event) &lt; <span class="hljs-number">0</span>) {
	            perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_MOD"</span>);
	          }
	        } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (events[i].events &amp; EPOLLOUT) {
	          <span class="hljs-comment">// Ready for writing.</span>
	          <span class="hljs-keyword">int</span> fd = events[i].data.fd;
	          <span class="hljs-keyword">fd_status_t</span> status = on_peer_ready_send(fd);
	          <span class="hljs-class"><span class="hljs-keyword">struct</span> <span class="hljs-title">epoll_event</span> <span class="hljs-title">event</span> = {</span><span class="hljs-number">0</span>};
	          event.data.fd = fd;
	
	          <span class="hljs-keyword">if</span> (status.want_read) {
	            event.events |= EPOLLIN;
	          }
	          <span class="hljs-keyword">if</span> (status.want_write) {
	            event.events |= EPOLLOUT;
	          }
	          <span class="hljs-keyword">if</span> (event.events == <span class="hljs-number">0</span>) {
	            <span class="hljs-built_in">printf</span>(<span class="hljs-string">"Socket %d closing\n"</span>, fd);
	            <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_DEL, fd, <span class="hljs-literal">NULL</span>) &lt; <span class="hljs-number">0</span>) {
	              perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_DEL"</span>);
	            }
	            <span class="hljs-built_in">close</span>(fd);
	          } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (epoll_ctl(epollfd, EPOLL_CTL_MOD, fd, &amp;event) &lt; <span class="hljs-number">0</span>) {
	            perror_die(<span class="hljs-string">"epoll_ctl EPOLL_CTL_MOD"</span>);
	          }
	        }
	      }
	    }
	  }
	
	  <span class="hljs-keyword">return</span> <span class="hljs-number">0</span>;
	}
</code></pre>
<h3 data-nodeid="15479">重新思考：I/O 模型</h3>
<p data-nodeid="15480">在上面的模型当中，select/poll 是阻塞（Blocking）模型，epoll 是非阻塞（Non-Blocking）模型。<strong data-nodeid="15691">阻塞和非阻塞强调的是线程的状态</strong>，所以阻塞就是触发了线程的阻塞状态，线程阻塞了就停止执行，并且切换到其他线程去执行，直到触发中断再回来。</p>
<p data-nodeid="15481">还有一组概念是同步（Synchrounous）和异步（Asynchrounous），select/poll/epoll 三者都是同步调用。</p>
<p data-nodeid="15482">**同步强调的是顺序，**所谓同步调用，就是可以确定程序执行的顺序的调用。比如说执行一个调用，知道调用返回之前下一行代码不会执行。这种顺序是确定的情况，就是同步。</p>
<p data-nodeid="15483">而异步调用则恰恰相反，<strong data-nodeid="15704">异步调用不明确执行顺序</strong>。比如说一个回调函数，不知道何时会回来。异步调用会加大程序员的负担，因为我们习惯顺序地思考程序。因此，我们还会发明像协程的 yield 、迭代器等将异步程序转为同步程序。</p>
<p data-nodeid="15484">由此可见，<strong data-nodeid="15710">非阻塞不一定是异步，阻塞也未必就是同步</strong>。比如一个带有回调函数的方法，阻塞了线程 100 毫秒，又提供了回调函数，那这个方法是异步阻塞。例如下面的伪代码：</p>
<pre class="lang-java" data-nodeid="15485"><code data-language="java">asleep(<span class="hljs-number">100</span>ms, () -&gt; {
  <span class="hljs-comment">// 100ms 或更多后到这里</span>
  <span class="hljs-comment">// ...do some thing</span>
})
<span class="hljs-comment">// 100 ms 后到这里</span>
</code></pre>
<h3 data-nodeid="15486">总结</h3>
<p data-nodeid="15487">总结下，操作系统给大家提供各种各样的 API，是希望满足各种各样程序架构的诉求。但总体诉求其实是一致的：希望程序员写的单机代码，能够在多线程甚至分布式的环境下执行。这样你就不需要再去学习复杂的并发控制算法。从这个角度去看，非阻塞加上同步的编程模型确实省去了我们编程过程当中的很多思考。</p>
<p data-nodeid="15488">但可惜的是，至少在今天这个时代，<strong data-nodeid="15722">多线程、并发编程依然是程序员们的必修课</strong>。因此你在思考 I/O 模型的时候，还是需要结合自己的业务特性及系统自身的架构特点，进行选择。<strong data-nodeid="15723">I/O 模型并不是选择效率，而是选择编程的手段</strong>。试想一个所有资源都跑满了的服务器，并不会因为是异步或者非阻塞模型就获得更高的吞吐量。</p>
<p data-nodeid="15489"><strong data-nodeid="15728">那么通过以上的学习，你现在可以尝试来回答本讲关联的面试题目：select/poll/epoll 有什么区别</strong>？</p>
<p data-nodeid="15490">【<strong data-nodeid="15734">解析</strong>】这三者都是处理 I/O 多路复用的编程手段。select/poll 模型是一种阻塞模型，epoll 是非阻塞模型。select/poll 内部使用线性结构存储进程关注的 Socket 集合，因此每次内核要判断某个消息是否发送给 select/poll 需要遍历进程关注的 Socket 集合。</p>
<p data-nodeid="15491">而 epoll 不同，epoll 内部使用二叉搜索树（红黑树），用 Socket 编号作为索引，用关注的事件类型作为值，这样内核可以在非常快的速度下就判断某个消息是否需要发送给使用 epoll 的线程。</p>
<h3 data-nodeid="15492">思考题</h3>
<p data-nodeid="15493"><strong data-nodeid="15741">最后我再给你出一道需要查资料的思考题：如果用 epoll 架构一个Web 服务器应该是一个怎样的架构</strong>？</p>
<p data-nodeid="15494">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>
<p data-nodeid="15495" class="">这一讲就到这里，发现求知的乐趣，我是林䭽。感谢你学习本次课程，下一讲我们将学习 36 | 公私钥体系和网络安全：什么是中间人攻击？再见！</p>

---

### 精选评论

##### *敏：
> 老师 socket编程模型讲的很好，但是看完还是对select poll epoll迷迷糊糊，希望能有个流程图对比

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 这个提议好！小编和林老师沟通了，近期会结合大家提出的疑问，优化和补充已更新的内容哈，比心：）

##### **乐：
> 期待流程图

##### **男：
> 创建socket_fd，设置端口服用，绑定bond，监听listen，epoll创建，将fd添加到epoll，循环等待有没有事件发生，若有epoll_wait返回，若客户端有新的连接，accept新的连接fd，将新的连接fd添加到epoll上

##### *明：
> 用epoll思想实现web服务，其实就是netty了

##### **生：
> 讲的很好 期待后边的网络课程

##### *彭：
> 阻塞 非阻塞 同步 异步 比较难理解

