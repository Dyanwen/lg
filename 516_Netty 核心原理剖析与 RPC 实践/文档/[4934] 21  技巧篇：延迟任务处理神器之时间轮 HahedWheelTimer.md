<p data-nodeid="1425" class="">Netty 中有很多场景依赖定时任务实现，比较典型的有客户端连接的超时控制、通信双方连接的心跳检测等场景。在学习 Netty Reactor 线程模型时，我们知道 NioEventLoop 不仅负责处理 I/O 事件，而且兼顾执行任务队列中的任务，其中就包括定时任务。为了实现高性能的定时任务调度，Netty 引入了时间轮算法驱动定时任务的执行。时间轮到底是什么呢？为什么 Netty 一定要用时间轮来处理定时任务呢？JDK 原生的实现方案不能满足要求吗？本节课我将一步步为你深入剖析时间轮的原理以及 Netty 中是如何实现时间轮算法的。</p>
<blockquote data-nodeid="1426">
<p data-nodeid="1427">说明：本文参考的 Netty 源码版本为 4.1.42.Final。</p>
</blockquote>
<h3 data-nodeid="1428">定时任务的基础知识</h3>
<p data-nodeid="1429">首先，我们先了解下什么是定时任务？定时器有非常多的使用场景，大家在平时工作中应该经常遇到，例如生成月统计报表、财务对账、会员积分结算、邮件推送等，都是定时器的使用场景。定时器一般有三种表现形式：按固定周期定时执行、延迟一定时间后执行、指定某个时刻执行。</p>
<p data-nodeid="1430">定时器的本质是设计一种数据结构，能够存储和调度任务集合，而且 deadline 越近的任务拥有更高的优先级。那么定时器如何知道一个任务是否到期了呢？定时器需要通过轮询的方式来实现，每隔一个时间片去检查任务是否到期。</p>
<p data-nodeid="1431">所以定时器的内部结构一般需要一个任务队列和一个异步轮询线程，并且能够提供三种基本操作：</p>
<ul data-nodeid="1432">
<li data-nodeid="1433">
<p data-nodeid="1434">Schedule 新增任务至任务集合；</p>
</li>
<li data-nodeid="1435">
<p data-nodeid="1436">Cancel 取消某个任务；</p>
</li>
<li data-nodeid="1437">
<p data-nodeid="1438">Run 执行到期的任务。</p>
</li>
</ul>
<p data-nodeid="1439">JDK 原生提供了三种常用的定时器实现方式，分别为 Timer、DelayedQueue 和 ScheduledThreadPoolExecutor。下面我们逐一对它们进行介绍。</p>
<h4 data-nodeid="1440">Timer</h4>
<p data-nodeid="1441">Timer 属于 JDK 比较早期版本的实现，它可以实现固定周期的任务，以及延迟任务。Timer 会起动一个异步线程去执行到期的任务，任务可以只被调度执行一次，也可以周期性反复执行多次。我们先来看下 Timer 是如何使用的，示例代码如下。</p>
<pre class="lang-java" data-nodeid="1442"><code data-language="java">Timer timer = <span class="hljs-keyword">new</span> Timer();
timer.scheduleAtFixedRate(<span class="hljs-keyword">new</span> TimerTask() {
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// do something</span>
&nbsp; &nbsp; }
}, <span class="hljs-number">10000</span>, <span class="hljs-number">1000</span>);&nbsp; <span class="hljs-comment">// 10s 后调度一个周期为 1s 的定时任务</span>
</code></pre>
<p data-nodeid="1443">可以看出，任务是由 TimerTask 类实现，TimerTask 是实现了 Runnable 接口的抽象类，Timer 负责调度和执行 TimerTask。接下来我们看下 Timer 的内部构造。</p>
<pre class="lang-java" data-nodeid="1444"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Timer</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> TaskQueue queue = <span class="hljs-keyword">new</span> TaskQueue();
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> TimerThread thread = <span class="hljs-keyword">new</span> TimerThread(queue);

&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">Timer</span><span class="hljs-params">(String name)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; thread.setName(name);
&nbsp; &nbsp; &nbsp; &nbsp; thread.start();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1445">TaskQueue 是由数组结构实现的小根堆，deadline 最近的任务位于堆顶端，queue[1] 始终是最优先被执行的任务。所以使用小根堆的数据结构，Run 操作时间复杂度 O(1)，新增 Schedule 和取消 Cancel 操作的时间复杂度都是 O(logn)。</p>
<p data-nodeid="1446">Timer 内部启动了一个 TimerThread 异步线程，不论有多少任务被加入数组，始终都是由 TimerThread 负责处理。TimerThread 会定时轮询 TaskQueue 中的任务，如果堆顶的任务的 deadline 已到，那么执行任务；如果是周期性任务，执行完成后重新计算下一次任务的 deadline，并再次放入小根堆；如果是单次执行的任务，执行结束后会从 TaskQueue 中删除。</p>
<h4 data-nodeid="1447">DelayedQueue</h4>
<p data-nodeid="1448">DelayedQueue 是 JDK 中一种可以延迟获取对象的阻塞队列，其内部是采用优先级队列 PriorityQueue 存储对象。DelayQueue 中的每个对象都必须实现 Delayed 接口，并重写 compareTo 和 getDelay 方法。DelayedQueue 的使用方法如下：</p>
<pre class="lang-java" data-nodeid="1449"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DelayQueueTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; BlockingQueue&lt;SampleTask&gt; delayQueue = <span class="hljs-keyword">new</span> DelayQueue&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> now = System.currentTimeMillis();
&nbsp; &nbsp; &nbsp; &nbsp; delayQueue.put(<span class="hljs-keyword">new</span> SampleTask(now + <span class="hljs-number">1000</span>));
&nbsp; &nbsp; &nbsp; &nbsp; delayQueue.put(<span class="hljs-keyword">new</span> SampleTask(now + <span class="hljs-number">2000</span>));
&nbsp; &nbsp; &nbsp; &nbsp; delayQueue.put(<span class="hljs-keyword">new</span> SampleTask(now + <span class="hljs-number">3000</span>));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">3</span>; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-keyword">new</span> Date(delayQueue.take().getTime()));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SampleTask</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Delayed</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> time;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SampleTask</span><span class="hljs-params">(<span class="hljs-keyword">long</span> time)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.time = time;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">getTime</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> time;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">compareTo</span><span class="hljs-params">(Delayed o)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Long.compare(<span class="hljs-keyword">this</span>.getDelay(TimeUnit.MILLISECONDS), o.getDelay(TimeUnit.MILLISECONDS));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">getDelay</span><span class="hljs-params">(TimeUnit unit)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> unit.convert(time - System.currentTimeMillis(), TimeUnit.MILLISECONDS);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1450">DelayQueue 提供了 put() 和 take() 的阻塞方法，可以向队列中添加对象和取出对象。对象被添加到 DelayQueue 后，会根据 compareTo() 方法进行优先级排序。getDelay() 方法用于计算消息延迟的剩余时间，只有 getDelay &lt;=0 时，该对象才能从 DelayQueue 中取出。</p>
<p data-nodeid="1451">DelayQueue 在日常开发中最常用的场景就是实现重试机制。例如，接口调用失败或者请求超时后，可以将当前请求对象放入 DelayQueue，通过一个异步线程 take() 取出对象然后继续进行重试。如果还是请求失败，继续放回 DelayQueue。为了限制重试的频率，可以设置重试的最大次数以及采用指数退避算法设置对象的 deadline，如 2s、4s、8s、16s ……以此类推。</p>
<p data-nodeid="1452">相比于 Timer，DelayQueue 只实现了任务管理的功能，需要与异步线程配合使用。DelayQueue 使用优先级队列实现任务的优先级排序，新增 Schedule 和取消 Cancel 操作的时间复杂度也是 O(logn)。</p>
<h4 data-nodeid="1453">ScheduledThreadPoolExecutor</h4>
<p data-nodeid="1454">上文中介绍的 Timer 其实目前并不推荐用户使用，它是存在不少设计缺陷的。</p>
<ul data-nodeid="1455">
<li data-nodeid="1456">
<p data-nodeid="1457">Timer 是单线程模式。如果某个 TimerTask 执行时间很久，会影响其他任务的调度。</p>
</li>
<li data-nodeid="1458">
<p data-nodeid="1459">Timer 的任务调度是基于系统绝对时间的，如果系统时间不正确，可能会出现问题。</p>
</li>
<li data-nodeid="1460">
<p data-nodeid="1461">TimerTask 如果执行出现异常，Timer 并不会捕获，会导致线程终止，其他任务永远不会执行。</p>
</li>
</ul>
<p data-nodeid="1462">为了解决 Timer 的设计缺陷，JDK 提供了功能更加丰富的 ScheduledThreadPoolExecutor。ScheduledThreadPoolExecutor 提供了周期执行任务和延迟执行任务的特性，下面通过一个例子先看下 ScheduledThreadPoolExecutor 如何使用。</p>
<pre class="lang-java" data-nodeid="1463"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ScheduledExecutorServiceTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; ScheduledExecutorService executor = Executors.newScheduledThreadPool(<span class="hljs-number">5</span>);
&nbsp; &nbsp; &nbsp; &nbsp; executor.scheduleAtFixedRate(() -&gt; System.out.println(<span class="hljs-string">"Hello World"</span>), <span class="hljs-number">1000</span>, <span class="hljs-number">2000</span>, TimeUnit.MILLISECONDS); <span class="hljs-comment">// 1s 延迟后开始执行任务，每 2s 重复执行一次</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1464">ScheduledThreadPoolExecutor 继承于 ThreadPoolExecutor，因此它具备线程池异步处理任务的能力。线程池主要负责管理创建和管理线程，并从自身的阻塞队列中不断获取任务执行。线程池有两个重要的角色，分别是任务和阻塞队列。ScheduledThreadPoolExecutor 在 ThreadPoolExecutor 的基础上，重新设计了任务 ScheduledFutureTask 和阻塞队列 DelayedWorkQueue。ScheduledFutureTask 继承于 FutureTask，并重写了 run() 方法，使其具备周期执行任务的能力。DelayedWorkQueue 内部是优先级队列，deadline 最近的任务在队列头部。对于周期执行的任务，在执行完会重新设置时间，并再次放入队列中。ScheduledThreadPoolExecutor 的实现原理可以用下图表示。</p>
<p data-nodeid="1465"><img src="https://s0.lgstatic.com/i/image2/M01/04/10/CgpVE1_okGKAVhdBAAKC36HWpJQ727.png" alt="图片11.png" data-nodeid="1619"><br>
以上我们简单介绍了 JDK 三种实现定时器的方式。可以说它们的实现思路非常类似，都离不开<strong data-nodeid="1634">任务</strong>、<strong data-nodeid="1635">任务管理</strong>、<strong data-nodeid="1636">任务调度</strong>三个角色。三种定时器新增和取消任务的时间复杂度都是 O(logn)，面对海量任务插入和删除的场景，这三种定时器都会遇到比较严重的性能瓶颈。因此，对于性能要求较高的场景，我们一般都会采用时间轮算法。那么时间轮又是如何解决海量任务插入和删除的呢？我们继续向下分析。</p>
<h3 data-nodeid="1466">时间轮原理分析</h3>
<p data-nodeid="1467">技术有时就源于生活，例如排队买票可以想到队列，公司的组织关系可以理解为树等，而时间轮算法的设计思想就来源于钟表。如下图所示，时间轮可以理解为一种环形结构，像钟表一样被分为多个 slot 槽位。每个 slot 代表一个时间段，每个 slot 中可以存放多个任务，使用的是链表结构保存该时间段到期的所有任务。时间轮通过一个时针随着时间一个个 slot 转动，并执行 slot 中的所有到期任务。</p>
<p data-nodeid="1468"><img src="https://s0.lgstatic.com/i/image2/M01/04/10/CgpVE1_okKiAGl0gAAMLshtTq-M933.png" alt="图片22.png" data-nodeid="1641"></p>
<p data-nodeid="1469">任务是如何添加到时间轮当中的呢？可以根据任务的到期时间进行取模，然后将任务分布到不同的 slot 中。如上图所示，时间轮被划分为 8 个 slot，每个 slot 代表 1s，当前时针指向 2。假如现在需要调度一个 3s 后执行的任务，应该加入 2+3=5 的 slot 中；如果需要调度一个 12s 以后的任务，需要等待时针完整走完一圈 round 零 4 个 slot，需要放入第 (2+12)%8=6 个 slot。</p>
<p data-nodeid="1470">那么当时针走到第 6 个 slot 时，怎么区分每个任务是否需要立即执行，还是需要等待下一圈 round，甚至更久时间之后执行呢？所以我们需要把 round 信息保存在任务中。例如图中第 6 个 slot 的链表中包含 3 个任务，第一个任务 round=0，需要立即执行；第二个任务 round=1，需要等待 1*8=8s 后执行；第三个任务 round=2，需要等待 2*8=8s 后执行。所以当时针转动到对应 slot 时，只执行 round=0 的任务，slot 中其余任务的 round 应当减 1，等待下一个 round 之后执行。</p>
<p data-nodeid="1471">上面介绍了时间轮算法的基本理论，可以看出时间轮有点类似 HashMap，如果多个任务如果对应同一个 slot，处理冲突的方法采用的是拉链法。在任务数量比较多的场景下，适当增加时间轮的 slot 数量，可以减少时针转动时遍历的任务个数。</p>
<p data-nodeid="1472">时间轮定时器最大的优势就是，任务的新增和取消都是 O(1) 时间复杂度，而且只需要一个线程就可以驱动时间轮进行工作。HashedWheelTimer 是 Netty 中时间轮算法的实现类，下面我就结合 HashedWheelTimer 的源码详细分析时间轮算法的实现原理。</p>
<h3 data-nodeid="1473">Netty HashedWheelTimer 源码解析</h3>
<p data-nodeid="1474">在开始学习 HashedWheelTimer 的源码之前，需要了解 HashedWheelTimer 接口定义以及相关组件，才能更好地使用 HashedWheelTimer。</p>
<h4 data-nodeid="1475">接口定义</h4>
<p data-nodeid="1476">HashedWheelTimer 实现了接口 io.netty.util.Timer，Timer 接口是我们研究 HashedWheelTimer 一个很好的切入口。一起看下 Timer 接口的定义：</p>
<pre class="lang-java" data-nodeid="1477"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Timer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function">Timeout <span class="hljs-title">newTimeout</span><span class="hljs-params">(TimerTask task, <span class="hljs-keyword">long</span> delay, TimeUnit unit)</span></span>;
&nbsp; &nbsp; <span class="hljs-function">Set&lt;Timeout&gt; <span class="hljs-title">stop</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="1478">Timer 接口提供了两个方法，分别是创建任务 newTimeout() 和停止所有未执行任务 stop()。从方法的定义可以看出，Timer 可以认为是上层的时间轮调度器，通过 newTimeout() 方法可以提交一个任务 TimerTask，并返回一个 Timeout。TimerTask 和 Timeout 是两个接口类，它们有什么作用呢？我们分别看下 TimerTask 和 Timeout 的接口定义：</p>
<pre class="lang-java" data-nodeid="1479"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">TimerTask</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(Timeout timeout)</span> <span class="hljs-keyword">throws</span> Exception</span>;
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Timeout</span> </span>{
&nbsp; &nbsp; <span class="hljs-function">Timer <span class="hljs-title">timer</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-function">TimerTask <span class="hljs-title">task</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isExpired</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isCancelled</span><span class="hljs-params">()</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">cancel</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="1480">Timeout 持有 Timer 和 TimerTask 的引用，而且通过 Timeout 接口可以执行取消任务的操作。Timer、Timeout 和 TimerTask 之间的关系如下图所示：</p>
<p data-nodeid="1481"><img src="https://s0.lgstatic.com/i/image2/M01/04/10/CgpVE1_okNGAJA8SAAJSJkBDij0471.png" alt="图片1.png" data-nodeid="1658"><br>
清楚 HashedWheelTimer 的接口定义以及相关组件的概念之后，接下来我们就可以开始使用它了。</p>
<h4 data-nodeid="1482">快速上手</h4>
<p data-nodeid="1483">通过下面这个简单的例子，我们看下 HashedWheelTimer 是如何使用的。</p>
<pre class="lang-java" data-nodeid="1484"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HashedWheelTimerTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Timer timer = <span class="hljs-keyword">new</span> HashedWheelTimer();
&nbsp; &nbsp; &nbsp; &nbsp; Timeout timeout1 = timer.newTimeout(<span class="hljs-keyword">new</span> TimerTask() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(Timeout timeout)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"timeout1: "</span> + <span class="hljs-keyword">new</span> Date());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }, <span class="hljs-number">10</span>, TimeUnit.SECONDS);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!timeout1.isExpired()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; timeout1.cancel();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; timer.newTimeout(<span class="hljs-keyword">new</span> TimerTask() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(Timeout timeout)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"timeout2: "</span> + <span class="hljs-keyword">new</span> Date());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Thread.sleep(<span class="hljs-number">5000</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }, <span class="hljs-number">1</span>, TimeUnit.SECONDS);
&nbsp; &nbsp; &nbsp; &nbsp; timer.newTimeout(<span class="hljs-keyword">new</span> TimerTask() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(Timeout timeout)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"timeout3: "</span> + <span class="hljs-keyword">new</span> Date());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }, <span class="hljs-number">3</span>, TimeUnit.SECONDS);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1485">代码运行结果如下：</p>
<pre class="lang-java" data-nodeid="1486"><code data-language="java">timeout2: Mon Nov <span class="hljs-number">09</span> <span class="hljs-number">19</span>:<span class="hljs-number">57</span>:<span class="hljs-number">04</span> CST <span class="hljs-number">2020</span>
timeout3: Mon Nov <span class="hljs-number">09</span> <span class="hljs-number">19</span>:<span class="hljs-number">57</span>:<span class="hljs-number">09</span> CST <span class="hljs-number">2020</span>
</code></pre>
<p data-nodeid="1487">简单的几行代码，基本展示了 HashedWheelTimer 的大部分用法。示例中我们通过 newTimeout() 启动了三个 TimerTask，timeout1 由于被取消了，所以并没有执行。timeout2 和 timeout3 分别应该在 1s 和 3s 后执行。然而从结果输出看并不是，timeout2 和 timeout3 的打印时间相差了 5s，这是由于 timeout2 阻塞了 5s 造成的。由此可以看出，时间轮中的任务执行是<strong data-nodeid="1669">串行</strong>的，当一个任务执行的时间过长，会影响后续任务的调度和执行，很可能产生任务堆积的情况。</p>
<p data-nodeid="1488">至此，对 HashedWheelTimer 的基本使用方法已经有了初步了解，下面我们开始深入研究 HashedWheelTimer 的实现原理。</p>
<h4 data-nodeid="1489">内部结构</h4>
<p data-nodeid="1490">我们先从 HashedWheelTimer 的构造函数看起，结合上文中介绍的时间轮算法，一起梳理出 HashedWheelTimer 的内部实现结构。</p>
<pre class="lang-java" data-nodeid="1491"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">HashedWheelTimer</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; ThreadFactory threadFactory,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> tickDuration,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; TimeUnit unit,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> ticksPerWheel,&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> leakDetection,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> maxPendingTimeouts)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>

&nbsp; &nbsp; wheel = createWheel(ticksPerWheel); <span class="hljs-comment">// 创建时间轮的环形数组结构</span>
&nbsp; &nbsp; mask = wheel.length - <span class="hljs-number">1</span>; <span class="hljs-comment">// 用于快速取模的掩码</span>
&nbsp; &nbsp; <span class="hljs-keyword">long</span> duration = unit.toNanos(tickDuration); <span class="hljs-comment">// 转换成纳秒处理</span>
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; workerThread = threadFactory.newThread(worker); <span class="hljs-comment">// 创建工作线程</span>
&nbsp; &nbsp; leak = leakDetection || !workerThread.isDaemon() ? leakDetector.track(<span class="hljs-keyword">this</span>) : <span class="hljs-keyword">null</span>; <span class="hljs-comment">// 是否开启内存泄漏检测</span>
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.maxPendingTimeouts = maxPendingTimeouts; <span class="hljs-comment">// 最大允许等待任务数，HashedWheelTimer 中任务超出该阈值时会抛出异常</span>
&nbsp; &nbsp; <span class="hljs-comment">// 如果 HashedWheelTimer 的实例数超过 64，会打印错误日志</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (INSTANCE_COUNTER.incrementAndGet() &gt; INSTANCE_COUNT_LIMIT &amp;&amp;
&nbsp; &nbsp; &nbsp; &nbsp; WARNED_TOO_MANY_INSTANCES.compareAndSet(<span class="hljs-keyword">false</span>, <span class="hljs-keyword">true</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; reportTooManyInstances();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1492">HashedWheelTimer 的构造函数清晰地列举出了几个核心属性：</p>
<ul data-nodeid="1493">
<li data-nodeid="1494">
<p data-nodeid="1495"><strong data-nodeid="1678">threadFactory</strong>，线程池，但是只创建了一个线程；</p>
</li>
<li data-nodeid="1496">
<p data-nodeid="1497"><strong data-nodeid="1683">tickDuration</strong>，时针每次 tick 的时间，相当于时针间隔多久走到下一个 slot；</p>
</li>
<li data-nodeid="1498">
<p data-nodeid="1499"><strong data-nodeid="1688">unit</strong>，表示 tickDuration 的时间单位；</p>
</li>
<li data-nodeid="1500">
<p data-nodeid="1501"><strong data-nodeid="1693">ticksPerWheel</strong>，时间轮上一共有多少个 slot，默认 512 个。分配的 slot 越多，占用的内存空间就越大；</p>
</li>
<li data-nodeid="1502">
<p data-nodeid="1503"><strong data-nodeid="1698">leakDetection</strong>，是否开启内存泄漏检测；</p>
</li>
<li data-nodeid="1504">
<p data-nodeid="1505"><strong data-nodeid="1703">maxPendingTimeouts</strong>，最大允许等待任务数。</p>
</li>
</ul>
<p data-nodeid="1506">下面我们看下 HashedWheelTimer 是如何创建出来的，我们直接跟进 createWheel() 方法的源码：</p>
<pre class="lang-java" data-nodeid="1507"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> HashedWheelBucket[] createWheel(<span class="hljs-keyword">int</span> ticksPerWheel) {
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; ticksPerWheel = normalizeTicksPerWheel(ticksPerWheel);
&nbsp; &nbsp; HashedWheelBucket[] wheel = <span class="hljs-keyword">new</span> HashedWheelBucket[ticksPerWheel];
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; wheel.length; i ++) {
&nbsp; &nbsp; &nbsp; &nbsp; wheel[i] = <span class="hljs-keyword">new</span> HashedWheelBucket();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> wheel;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> <span class="hljs-title">normalizeTicksPerWheel</span><span class="hljs-params">(<span class="hljs-keyword">int</span> ticksPerWheel)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> normalizedTicksPerWheel = <span class="hljs-number">1</span>;
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (normalizedTicksPerWheel &lt; ticksPerWheel) {
&nbsp; &nbsp; &nbsp; &nbsp; normalizedTicksPerWheel &lt;&lt;= <span class="hljs-number">1</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> normalizedTicksPerWheel;
}
<span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HashedWheelBucket</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> HashedWheelTimeout head;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> HashedWheelTimeout tail;
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="1508">时间轮的创建就是为了创建 HashedWheelBucket 数组，每个 HashedWheelBucket 表示时间轮中一个 slot。从 HashedWheelBucket 的结构定义可以看出，HashedWheelBucket 内部是一个双向链表结构，双向链表的每个节点持有一个 HashedWheelTimeout 对象，HashedWheelTimeout 代表一个定时任务。每个 HashedWheelBucket 都包含双向链表 head 和 tail 两个 HashedWheelTimeout 节点，这样就可以实现不同方向进行链表遍历。关于 HashedWheelBucket 和 HashedWheelTimeout 的具体功能下文再继续介绍。</p>
<p data-nodeid="1509">因为时间轮需要使用 &amp; 做取模运算，所以数组的长度需要是 2 的次幂。normalizeTicksPerWheel() 方法的作用就是找到不小于 ticksPerWheel 的最小 2 次幂，这个方法实现的并不好，可以参考 JDK HashMap 扩容 tableSizeFor 的实现进行性能优化，如下所示。当然 normalizeTicksPerWheel() 只是在初始化的时候使用，所以并无影响。</p>
<pre class="lang-java" data-nodeid="1510"><code data-language="java"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> MAXIMUM_CAPACITY = <span class="hljs-number">1</span> &lt;&lt; <span class="hljs-number">30</span>;
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> <span class="hljs-title">normalizeTicksPerWheel</span><span class="hljs-params">(<span class="hljs-keyword">int</span> ticksPerWheel)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> n = ticksPerWheel - <span class="hljs-number">1</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">1</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">2</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">4</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">8</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">16</span>;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> (n &lt; <span class="hljs-number">0</span>) ? <span class="hljs-number">1</span> : (n &gt;= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + <span class="hljs-number">1</span>;
}
</code></pre>
<p data-nodeid="1511">HashedWheelTimer 初始化的主要工作我们已经介绍完了，其内部结构与上文中介绍的时间轮算法类似，如下图所示。</p>
<p data-nodeid="1512"><img src="https://s0.lgstatic.com/i/image/M00/8C/37/CgqCHl_okUGATnXpAAPdCRAt-n0348.png" alt="图片2.png" data-nodeid="1712"></p>
<p data-nodeid="1513">接下来我们围绕定时器的三种基本操作，分析下 HashedWheelTimer 是如何实现添加任务、执行任务和取消任务的。</p>
<h4 data-nodeid="1514">添加任务</h4>
<p data-nodeid="1515">HashedWheelTimer 初始化完成后，如何向 HashedWheelTimer 添加任务呢？我们自然想到 HashedWheelTimer 提供的 newTimeout() 方法。</p>
<pre class="lang-java" data-nodeid="1516"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Timeout <span class="hljs-title">newTimeout</span><span class="hljs-params">(TimerTask task, <span class="hljs-keyword">long</span> delay, TimeUnit unit)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; <span class="hljs-keyword">long</span> pendingTimeoutsCount = pendingTimeouts.incrementAndGet();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (maxPendingTimeouts &gt; <span class="hljs-number">0</span> &amp;&amp; pendingTimeoutsCount &gt; maxPendingTimeouts) {
&nbsp; &nbsp; &nbsp; &nbsp; pendingTimeouts.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RejectedExecutionException(<span class="hljs-string">"Number of pending timeouts ("</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + pendingTimeoutsCount + <span class="hljs-string">") is greater than or equal to maximum allowed pending "</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; + <span class="hljs-string">"timeouts ("</span> + maxPendingTimeouts + <span class="hljs-string">")"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; start(); <span class="hljs-comment">// 1. 如果 worker 线程没有启动，需要启动</span>
&nbsp; &nbsp; <span class="hljs-keyword">long</span> deadline = System.nanoTime() + unit.toNanos(delay) - startTime; <span class="hljs-comment">// 计算任务的 deadline</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (delay &gt; <span class="hljs-number">0</span> &amp;&amp; deadline &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; deadline = Long.MAX_VALUE;
&nbsp; &nbsp; }
&nbsp; &nbsp; HashedWheelTimeout timeout = <span class="hljs-keyword">new</span> HashedWheelTimeout(<span class="hljs-keyword">this</span>, task, deadline); <span class="hljs-comment">//&nbsp; 2. 创建定时任务</span>
&nbsp; &nbsp; timeouts.add(timeout); <span class="hljs-comment">// 3. 添加任务到 Mpsc Queue</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> timeout;
}
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Queue&lt;HashedWheelTimeout&gt; timeouts = PlatformDependent.newMpscQueue();
</code></pre>
<p data-nodeid="1517">newTimeout() 方法主要做了三件事，分别为启动工作线程，创建定时任务，并把任务添加到 Mpsc Queue。HashedWheelTimer 的工作线程采用了懒启动的方式，不需要用户显示调用。这样做的好处是在时间轮中没有任务时，可以避免工作线程空转而造成性能损耗。先看下启动工作线程 start() 的源码：</p>
<pre class="lang-java" data-nodeid="1518"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">start</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">switch</span> (WORKER_STATE_UPDATER.get(<span class="hljs-keyword">this</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> WORKER_STATE_INIT:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (WORKER_STATE_UPDATER.compareAndSet(<span class="hljs-keyword">this</span>, WORKER_STATE_INIT, WORKER_STATE_STARTED)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; workerThread.start();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> WORKER_STATE_STARTED:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> WORKER_STATE_SHUTDOWN:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"cannot be started once stopped"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">default</span>:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> Error(<span class="hljs-string">"Invalid WorkerState"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (startTime == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; startTimeInitialized.await();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (InterruptedException ignore) {
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1519">工作线程的启动之前，会通过 CAS 操作获取工作线程的状态，如果已经启动，则直接跳过。如果没有启动，再次通过 CAS 操作更改工作线程状态，然后启动工作线程。启动的过程是直接调用的 Thread#start() 方法，我们暂且先不关注工作线程具体做了什么，下文再继续分析。</p>
<p data-nodeid="1520">回到 newTimeout() 的主流程，接下来的逻辑就非常简单了。根据用户传入的任务延迟时间，可以计算出任务的 deadline，然后创建定时任务 HashedWheelTimeout 对象，最终把 HashedWheelTimeout 添加到 Mpsc Queue 中。看到这里，你会不会有个疑问，为什么不是将 HashedWheelTimeout 直接添加到时间轮中呢？而是先添加到 Mpsc Queue？Mpsc Queue 可以理解为多生产者单消费者的线程安全队列，下节课我们会对 Mpsc Queue 详细分析，在这里就不做展开了。可以猜到 HashedWheelTimer 是想借助 Mpsc Queue 保证多线程向时间轮添加任务的线程安全性。</p>
<p data-nodeid="1521">那么什么时候任务才会被加入时间轮并执行呢？此时还没有太多信息，接下来我们只能工作线程 Worker 里寻找问题的答案。</p>
<h4 data-nodeid="1522">工作线程 Worker</h4>
<p data-nodeid="1523">工作线程 Worker 是时间轮的核心引擎，随着时针的转动，到期任务的处理都由 Worker 处理完成。下面我们定位到 Worker 的 run() 方法一探究竟。</p>
<pre class="lang-java" data-nodeid="1524"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Worker</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Runnable</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Set&lt;Timeout&gt; unprocessedTimeouts = <span class="hljs-keyword">new</span> HashSet&lt;Timeout&gt;(); <span class="hljs-comment">// 未处理任务列表</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> tick;
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; startTime = System.nanoTime();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (startTime == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; startTime = <span class="hljs-number">1</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; startTimeInitialized.countDown();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> deadline = waitForNextTick(); <span class="hljs-comment">// 1. 计算下次 tick 的时间, 然后sleep 到下次 tick</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (deadline &gt; <span class="hljs-number">0</span>) { <span class="hljs-comment">// 可能因为溢出或者线程中断，造成 deadline &lt;= 0</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> idx = (<span class="hljs-keyword">int</span>) (tick &amp; mask); <span class="hljs-comment">// 2. 获取当前 tick 在 HashedWheelBucket 数组中对应的下标</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processCancelledTasks(); <span class="hljs-comment">// 3. 移除被取消的任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; HashedWheelBucket bucket =
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; wheel[idx];
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; transferTimeoutsToBuckets(); <span class="hljs-comment">// 4. 从 Mpsc Queue 中取出任务加入对应的 slot 中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; bucket.expireTimeouts(deadline); <span class="hljs-comment">// 5. 执行到期的任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; tick++;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">while</span> (WORKER_STATE_UPDATER.get(HashedWheelTimer.<span class="hljs-keyword">this</span>) == WORKER_STATE_STARTED);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 时间轮退出后，取出 slot 中未执行且未被取消的任务，并加入未处理任务列表，以便 stop() 方法返回</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (HashedWheelBucket bucket: wheel) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; bucket.clearTimeouts(unprocessedTimeouts);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将还没来得及添加到 slot 中的任务取出，如果任务未取消则加入未处理任务列表，以便 stop() 方法返回</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; HashedWheelTimeout timeout = timeouts.poll();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!timeout.isCancelled()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; unprocessedTimeouts.add(timeout);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; processCancelledTasks();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1525">工作线程 Worker 的核心执行流程是代码中的 do-while 循环，只要 Worker 处于 STARTED 状态，就会执行 do-while 循环，我们把该过程拆分成为以下几个步骤，逐一分析。</p>
<ul data-nodeid="1526">
<li data-nodeid="1527">
<p data-nodeid="1528">通过 waitForNextTick() 方法计算出时针到下一次 tick 的时间间隔，然后 sleep 到下一次 tick。</p>
</li>
<li data-nodeid="1529">
<p data-nodeid="1530">通过位运算获取当前 tick 在 HashedWheelBucket 数组中对应的下标</p>
</li>
<li data-nodeid="1531">
<p data-nodeid="1532">移除被取消的任务。</p>
</li>
<li data-nodeid="1533">
<p data-nodeid="1534">从 Mpsc Queue 中取出任务加入对应的 HashedWheelBucket 中。</p>
</li>
<li data-nodeid="1535">
<p data-nodeid="1536">执行当前 HashedWheelBucket 中的到期任务。</p>
</li>
</ul>
<p data-nodeid="1537">首先看下 waitForNextTick() 方法是如何计算等待时间的，源码如下：</p>
<pre class="lang-java" data-nodeid="1538"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> <span class="hljs-title">waitForNextTick</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">long</span> deadline = tickDuration * (tick + <span class="hljs-number">1</span>);
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> currentTime = System.nanoTime() - startTime;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> sleepTimeMs = (deadline - currentTime + <span class="hljs-number">999999</span>) / <span class="hljs-number">1000000</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (sleepTimeMs &lt;= <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (currentTime == Long.MIN_VALUE) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> -Long.MAX_VALUE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> currentTime;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (PlatformDependent.isWindows()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; sleepTimeMs = sleepTimeMs / <span class="hljs-number">10</span> * <span class="hljs-number">10</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Thread.sleep(sleepTimeMs);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (InterruptedException ignored) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (WORKER_STATE_UPDATER.get(HashedWheelTimer.<span class="hljs-keyword">this</span>) == WORKER_STATE_SHUTDOWN) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Long.MIN_VALUE;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1539">根据 tickDuration 可以推算出下一次 tick 的 deadline，deadline 减去当前时间就可以得到需要 sleep 的等待时间。所以 tickDuration 的值越小，时间的精准度也就越高，同时 Worker 的繁忙程度越高。如果 tickDuration 设置过小，为了防止系统会频繁地 sleep 再唤醒，会保证 Worker 至少 sleep 的时间为 1ms 以上。</p>
<p data-nodeid="1540">Worker 从 sleep 状态唤醒后，接下来会执行第二步流程，通过按位与的操作计算出当前 tick 在 HashedWheelBucket 数组中对应的下标。按位与比普通的取模运算效率要快很多，前提是时间轮中的数组长度是 2 的次幂，掩码 mask 为 2 的次幂减 1，这样才能达到与取模一样的效果。</p>
<p data-nodeid="1541">接下来 Worker 会调用 processCancelledTasks() 方法处理被取消的任务，所有取消的任务都会加入 cancelledTimeouts 队列中，Worker 会从队列中取出任务，然后将其从对应的 HashedWheelBucket 中删除，删除操作为基本的链表操作。processCancelledTasks() 的源码比较简单，我们在此就不展开了。</p>
<p data-nodeid="1542">之前我们还留了一个疑问，Mpsc Queue 中的任务什么时候加入时间轮的呢？答案就在 transferTimeoutsToBuckets() 方法中。</p>
<pre class="lang-java" data-nodeid="1543"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">transferTimeoutsToBuckets</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 每次时针 tick 最多只处理 100000 个任务，以防阻塞 Worker 线程</span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">100000</span>; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; HashedWheelTimeout timeout = timeouts.poll();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout.state() == HashedWheelTimeout.ST_CANCELLED) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> calculated = timeout.deadline / tickDuration; <span class="hljs-comment">// 计算任务需要经过多少个 tick</span>
&nbsp; &nbsp; &nbsp; &nbsp; timeout.remainingRounds = (calculated - tick) / wheel.length; <span class="hljs-comment">// 计算任务需要在时间轮中经历的圈数 remainingRounds</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> ticks = Math.max(calculated, tick); <span class="hljs-comment">// 如果任务在 timeouts 队列里已经过了执行时间, 那么会加入当前 HashedWheelBucket 中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> stopIndex = (<span class="hljs-keyword">int</span>) (ticks &amp; mask);
&nbsp; &nbsp; &nbsp; &nbsp; HashedWheelBucket bucket = wheel[stopIndex];
&nbsp; &nbsp; &nbsp; &nbsp; bucket.addTimeout(timeout);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1544">transferTimeoutsToBuckets() 的主要工作就是从 Mpsc Queue 中取出任务，然后添加到时间轮对应的 HashedWheelBucket 中。每次时针 tick 最多只处理 100000 个任务，一方面避免取任务的操作耗时过长，另一方面为了防止执行太多任务造成 Worker 线程阻塞。</p>
<p data-nodeid="1545">根据用户设置的任务 deadline，可以计算出任务需要经过多少次 tick 才能开始执行以及需要在时间轮中转动圈数 remainingRounds，remainingRounds 会记录在 HashedWheelTimeout 中，在执行任务的时候 remainingRounds 会被使用到。因为时间轮中的任务并不能够保证及时执行，假如有一个任务执行的时间特别长，那么任务在 timeouts 队列里已经过了执行时间，也没有关系，Worker 会将这些任务直接加入当前HashedWheelBucket 中，所以过期的任务并不会被遗漏。</p>
<p data-nodeid="1546">任务被添加到时间轮之后，重新再回到 Worker#run() 的主流程，接下来就是执行当前 HashedWheelBucket 中的到期任务，跟进 HashedWheelBucket#expireTimeouts() 方法的源码：</p>
<pre class="lang-java" data-nodeid="1547"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">expireTimeouts</span><span class="hljs-params">(<span class="hljs-keyword">long</span> deadline)</span> </span>{
&nbsp; &nbsp; HashedWheelTimeout timeout = head;
&nbsp; &nbsp; <span class="hljs-keyword">while</span> (timeout != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; HashedWheelTimeout next = timeout.next;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout.remainingRounds &lt;= <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; next = remove(timeout);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (timeout.deadline &lt;= deadline) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; timeout.expire(); <span class="hljs-comment">// 执行任务</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(String.format(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"timeout.deadline (%d) &gt; deadline (%d)"</span>, timeout.deadline, deadline));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (timeout.isCancelled()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; next = remove(timeout);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; timeout.remainingRounds --; <span class="hljs-comment">// 未到执行时间，remainingRounds 减 1</span>
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; timeout = next;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1548">执行任务的操作比较简单，就是从头开始遍历 HashedWheelBucket 中的双向链表。如果 remainingRounds &lt;=0，则调用 expire() 方法执行任务，timeout.expire() 内部就是调用了 TimerTask 的 run() 方法。如果任务已经被取消，直接从链表中移除。否则表示任务的执行时间还没到，remainingRounds 减 1，等待下一圈即可。</p>
<p data-nodeid="1549">至此，工作线程 Worker 的核心逻辑 do-while 循环我们已经讲完了。当时间轮退出后，Worker 还会执行一些后置的收尾工作。Worker 会从每个 HashedWheelBucket 取出未执行且未取消的任务，以及还来得及添加到 HashedWheelBucket 中的任务，然后加入未处理任务列表，以便 stop() 方法统一处理。</p>
<h4 data-nodeid="1550">停止时间轮</h4>
<p data-nodeid="1551">回到 Timer 接口两个方法，newTimeout() 上文已经分析完了，接下来我们就以 stop() 方法为入口，看下时间轮停止都做了哪些工作。</p>
<pre class="lang-java" data-nodeid="1552"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Set&lt;Timeout&gt; <span class="hljs-title">stop</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// Worker 线程无法停止时间轮</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (Thread.currentThread() == workerThread) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; HashedWheelTimer.class.getSimpleName() +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ".stop() cannot be called from " +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TimerTask.class.getSimpleName());
&nbsp; &nbsp; }
&nbsp; &nbsp; // 尝试通过 CAS 操作将工作线程的状态更新为 SHUTDOWN 状态
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!WORKER_STATE_UPDATER.compareAndSet(<span class="hljs-keyword">this</span>, WORKER_STATE_STARTED, WORKER_STATE_SHUTDOWN)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (WORKER_STATE_UPDATER.getAndSet(<span class="hljs-keyword">this</span>, WORKER_STATE_SHUTDOWN) != WORKER_STATE_SHUTDOWN) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; INSTANCE_COUNTER.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (leak != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> closed = leak.close(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">assert</span> closed;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Collections.emptySet();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> interrupted = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">while</span> (workerThread.isAlive()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; workerThread.interrupt(); <span class="hljs-comment">// 中断 Worker 线程</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; workerThread.join(<span class="hljs-number">100</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (InterruptedException ignored) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; interrupted = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (interrupted) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Thread.currentThread().interrupt();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; INSTANCE_COUNTER.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (leak != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> closed = leak.close(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">assert</span> closed;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> worker.unprocessedTimeouts(); <span class="hljs-comment">// 返回未处理任务的列表</span>
}
</code></pre>
<p data-nodeid="1553">如果当前线程是 Worker 线程，它是不能发起停止时间轮的操作的，是为了防止有定时任务发起停止时间轮的恶意操作。停止时间轮主要做了三件事，首先尝试通过 CAS 操作将工作线程的状态更新为 SHUTDOWN 状态，然后中断工作线程 Worker，最后将未处理的任务列表返回给上层。</p>
<p data-nodeid="1554">到此为止，HashedWheelTimer 的实现原理我们已经分析完了。再来回顾一下 HashedWheelTimer 的几个核心成员。</p>
<ul data-nodeid="1555">
<li data-nodeid="1556">
<p data-nodeid="1557"><strong data-nodeid="1748">HashedWheelTimeout</strong>，任务的封装类，包含任务的到期时间 deadline、需要经历的圈数 remainingRounds 等属性。</p>
</li>
<li data-nodeid="1558">
<p data-nodeid="1559"><strong data-nodeid="1753">HashedWheelBucket</strong>，相当于时间轮的每个 slot，内部采用双向链表保存了当前需要执行的 HashedWheelTimeout 列表。</p>
</li>
<li data-nodeid="1560">
<p data-nodeid="1561"><strong data-nodeid="1758">Worker</strong>，HashedWheelTimer 的核心工作引擎，负责处理定时任务。</p>
</li>
</ul>
<h3 data-nodeid="1562">时间轮进阶应用</h3>
<p data-nodeid="1563">Netty 中的时间轮是通过固定的时间间隔 tickDuration 进行推动的，如果长时间没有到期任务，那么会存在时间轮空推进的现象，从而造成一定的性能损耗。此外，如果任务的到期时间跨度很大，例如 A 任务 1s 后执行，B 任务 6 小时之后执行，也会造成空推进的问题。</p>
<p data-nodeid="1564">那么上述问题有没有什么解决方案呢？在研究 Kafka 的时候，Kafka 也有时间轮的应用，它的实现思路与 Netty 是存在区别的。因为 Kafka 面对的应用场景是更加严苛的，可能会存在各种时间粒度的定时任务，那么 Kafka 是否有解决时间跨度问题呢？我们接下来就简单介绍下 Kafka 的优化思路。</p>
<p data-nodeid="1565">Kafka 时间轮的内部结构与 Netty 类似，如下图所示。Kafka 的时间轮也是采用环形数组存储定时任务，数组中的每个 slot 代表一个 Bucket，每个 Bucket 保存了定时任务列表 TimerTaskList，TimerTaskList 同样采用双向链表的结构实现，链表的每个节点代表真正的定时任务 TimerTaskEntry。</p>
<p data-nodeid="1566"><img src="https://s0.lgstatic.com/i/image2/M01/04/10/CgpVE1_okZyASBNiAANiUk0T6tc832.png" alt="图片3.png" data-nodeid="1765"></p>
<p data-nodeid="1567">为了解决空推进的问题，Kafka 借助 JDK 的 DelayQueue 来负责推进时间轮。DelayQueue 保存了时间轮中的每个 Bucket，并且根据 Bucket 的到期时间进行排序，最近的到期时间被放在 DelayQueue 的队头。Kafka 中会有一个线程来读取 DelayQueue 中的任务列表，如果时间没有到，那么 DelayQueue 会一直处于阻塞状态，从而解决空推荐的问题。这时候你可能会问，DelayQueue 插入和删除的性能不是并不好吗？其实 Kafka 采用的是一种权衡的策略，把 DelayQueue 用在了合适的地方。DelayQueue 只存放了 Bucket，Bucket 的数量并不多，相比空推进带来的影响是利大于弊的。</p>
<p data-nodeid="1568">为了解决任务时间跨度很大的问题，Kafka 引入了层级时间轮，如下图所示。当任务的 deadline 超出当前所在层的时间轮表示范围时，就会尝试将任务添加到上一层时间轮中，跟钟表的时针、分针、秒针的转动规则是同一个道理。</p>
<p data-nodeid="1569"><img src="https://s0.lgstatic.com/i/image/M00/8C/2C/Ciqc1F_okdOAION1AALmn-njKt8140.png" alt="图片4.png" data-nodeid="1770"></p>
<p data-nodeid="1570">从图中可以看出，第一层时间轮每个时间格为 1ms，整个时间轮的跨度为 20ms；第二层时间轮每个时间格为 20ms，整个时间轮跨度为 400ms；第三层时间轮每个时间格为 400ms，整个时间轮跨度为 8000ms。每一层时间轮都有自己的指针，每层时间轮走完一圈后，上层时间轮也会相应推进一格。</p>
<p data-nodeid="2495" class="te-preview-highlight">假设现在有一个任务到期时间是 450ms 之后，应该放在第三层时间轮的第一格。随着时间的流逝，当指针指向该时间格时，发现任务到期时间还有 50ms，这里就涉及时间轮降级的操作，它会将任务重新提交到时间轮中。此时发现第一层时间轮整体跨度不够，需要放在第二层时间轮中第三格。当时间再经历 40ms 之后，该任务又会触发一次降级操作，放入到第一层时间轮，最后等到 10ms 后执行任务。</p>


<p data-nodeid="1572">由此可见，Kafka 的层级时间轮的时间粒度更好控制，可以应对更加复杂的定时任务处理场景，适用的范围更广。</p>
<h3 data-nodeid="1573">总结</h3>
<p data-nodeid="1574">HashedWheelTimer 的源码通俗易懂，其设计思想值得我们借鉴。在平时开发中如果有类似的任务处理机制，你可以尝试套用 HashedWheelTimer 的工作模式。</p>
<p data-nodeid="1575">HashedWheelTimer 并不是十全十美的，使用的时候需要清楚它存在的问题：</p>
<ul data-nodeid="1576">
<li data-nodeid="1577">
<p data-nodeid="1578">如果长时间没有到期任务，那么会存在时间轮空推进的现象。</p>
</li>
<li data-nodeid="1579">
<p data-nodeid="1580">只适用于处理耗时较短的任务，由于 Worker 是单线程的，如果一个任务执行的时间过长，会造成 Worker 线程阻塞。</p>
</li>
<li data-nodeid="1581">
<p data-nodeid="1582">相比传统定时器的实现方式，内存占用较大。</p>
</li>
</ul>
<p data-nodeid="1583" class="">感谢你认真学习，我们下节课继续加油。</p>

---

### 精选评论

##### **华：
> 老师讲得非常清楚，棒！

##### **靖：
> 讲的很不错，自己跟一遍源码收获很多！

##### **辉：
> 请问老师图是用什么工具画的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Sketch

##### **成：
> 收获满满😁

##### **威：
> 旁征博引，老师讲得真好😀

##### **杰：
> 开眼界了，666

