<p data-nodeid="1095" class="">在本课时我们主要学习常见的 6 种线程池，并详细讲解 Java 8 新增的 ForkJoinPool 线程池，6 种常见的线程池如下。</p>
<ul data-nodeid="1096">
<li data-nodeid="1097">
<p data-nodeid="1098">FixedThreadPool</p>
</li>
<li data-nodeid="1099">
<p data-nodeid="1100">CachedThreadPool</p>
</li>
<li data-nodeid="1101">
<p data-nodeid="1102">ScheduledThreadPool</p>
</li>
<li data-nodeid="1103">
<p data-nodeid="1104">SingleThreadExecutor</p>
</li>
<li data-nodeid="1105">
<p data-nodeid="1106">SingleThreadScheduledExecutor</p>
</li>
<li data-nodeid="1107">
<p data-nodeid="1108">ForkJoinPool</p>
</li>
</ul>
<h3 data-nodeid="1109">FixedThreadPool</h3>
<p data-nodeid="1110">第一种线程池叫作 FixedThreadPool，它的核心线程数和最大线程数是一样的，所以可以把它看作是固定线程数的线程池，它的特点是线程池中的线程数除了初始阶段需要从 0 开始增加外，之后的线程数量就是固定的，就算任务数超过线程数，线程池也不会再创建更多的线程来处理任务，而是会把超出线程处理能力的任务放到任务队列中进行等待。而且就算任务队列满了，到了本该继续增加线程数的时候，由于它的最大线程数和核心线程数是一样的，所以也无法再增加新的线程了。</p>
<p data-nodeid="1111"><img src="https://s0.lgstatic.com/i/image2/M01/AF/A0/CgotOV3kzoeARRniAAAwS8Pup4A734.png" alt="" data-nodeid="1203"></p>
<p data-nodeid="1112">如图所示，线程池有 t0~t9，10 个线程，它们会不停地执行任务，如果某个线程任务执行完了，就会从任务队列中获取新的任务继续执行，期间线程数量不会增加也不会减少，始终保持在 10 个。</p>
<h3 data-nodeid="1113">CachedThreadPool</h3>
<p data-nodeid="1114">第二种线程池是 CachedThreadPool，可以称作可缓存线程池，它的特点在于线程数是几乎可以无限增加的（实际最大可以达到 Integer.MAX_VALUE，为 2^31-1，这个数非常大，所以基本不可能达到），而当线程闲置时还可以对线程进行回收。也就是说该线程池的线程数量不是固定不变的，当然它也有一个用于存储提交任务的队列，但这个队列是 SynchronousQueue，队列的容量为0，实际不存储任何任务，它只负责对任务进行中转和传递，所以效率比较高。</p>
<p data-nodeid="1115">当我们提交一个任务后，线程池会判断已创建的线程中是否有空闲线程，如果有空闲线程则将任务直接指派给空闲线程，如果没有空闲线程，则新建线程去执行任务，这样就做到了动态地新增线程。让我们举个例子，如下方代码所示。</p>
<pre class="lang-java" data-nodeid="1116"><code data-language="java">ExecutorService&nbsp;service&nbsp;=&nbsp;Executors.newCachedThreadPool();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;<span class="hljs-number">1000</span>;&nbsp;i++)&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;service.execute(<span class="hljs-keyword">new</span>&nbsp;Task()&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;}
</code></pre>
<p data-nodeid="1117">使用 for 循环提交 1000 个任务给 CachedThreadPool，假设这些任务处理的时间非常长，会发生什么情况呢？因为 for 循环提交任务的操作是非常快的，但执行任务却比较耗时，就可能导致 1000 个任务都提交完了但第一个任务还没有被执行完，所以此时 CachedThreadPool 就可以动态的伸缩线程数量，随着任务的提交，不停地创建 1000 个线程来执行任务，而当任务执行完之后，假设没有新的任务了，那么大量的闲置线程又会造成内存资源的浪费，这时线程池就会检测线程在 60 秒内有没有可执行任务，如果没有就会被销毁，最终线程数量会减为 0。</p>
<h3 data-nodeid="1118">ScheduledThreadPool</h3>
<p data-nodeid="1119">第三个线程池是 ScheduledThreadPool，它支持定时或周期性执行任务。比如每隔 10 秒钟执行一次任务，而实现这种功能的方法主要有 3 种，如代码所示：</p>
<pre class="lang-java" data-nodeid="1120"><code data-language="java">ScheduledExecutorService&nbsp;service&nbsp;=&nbsp;Executors.newScheduledThreadPool(<span class="hljs-number">10</span>);
&nbsp;
service.schedule(<span class="hljs-keyword">new</span>&nbsp;Task(),&nbsp;<span class="hljs-number">10</span>,&nbsp;TimeUnit.SECONDS);
&nbsp;
service.scheduleAtFixedRate(<span class="hljs-keyword">new</span>&nbsp;Task(),&nbsp;<span class="hljs-number">10</span>,&nbsp;<span class="hljs-number">10</span>,&nbsp;TimeUnit.SECONDS);
&nbsp;
service.scheduleWithFixedDelay(<span class="hljs-keyword">new</span>&nbsp;Task(),&nbsp;<span class="hljs-number">10</span>,&nbsp;<span class="hljs-number">10</span>,&nbsp;TimeUnit.SECONDS);
</code></pre>
<p data-nodeid="1121">那么这 3 种方法有什么区别呢？</p>
<ul data-nodeid="1122">
<li data-nodeid="1123">
<p data-nodeid="1124">第一种方法 schedule 比较简单，表示延迟指定时间后执行一次任务，如果代码中设置参数为 10 秒，也就是 10 秒后执行一次任务后就结束。</p>
</li>
<li data-nodeid="1125">
<p data-nodeid="1126">第二种方法 scheduleAtFixedRate 表示以固定的频率执行任务，它的第二个参数 initialDelay 表示第一次延时时间，第三个参数 period 表示周期，也就是第一次延时后每次延时多长时间执行一次任务。</p>
</li>
<li data-nodeid="1127">
<p data-nodeid="1128">第三种方法&nbsp;scheduleWithFixedDelay 与第二种方法类似，也是周期执行任务，区别在于对周期的定义，之前的 scheduleAtFixedRate 是以任务开始的时间为时间起点开始计时，时间到就开始执行第二次任务，而不管任务需要花多久执行；而 scheduleWithFixedDelay 方法以任务结束的时间为下一次循环的时间起点开始计时。</p>
</li>
</ul>
<p data-nodeid="1129">举个例子，假设某个同学正在熬夜写代码，需要喝咖啡来提神，假设每次喝咖啡都需要花10分钟的时间，如果此时采用第2种方法&nbsp;scheduleAtFixedRate，时间间隔设置为 1 小时，那么他将会在每个整点喝一杯咖啡，以下是时间表：</p>
<ul data-nodeid="1130">
<li data-nodeid="1131">
<p data-nodeid="1132">00:00: 开始喝咖啡</p>
</li>
<li data-nodeid="1133">
<p data-nodeid="1134">00:10: 喝完了</p>
</li>
<li data-nodeid="1135">
<p data-nodeid="1136">01:00: 开始喝咖啡</p>
</li>
<li data-nodeid="1137">
<p data-nodeid="1138">01:10: 喝完了</p>
</li>
<li data-nodeid="1139">
<p data-nodeid="1140">02:00: 开始喝咖啡</p>
</li>
<li data-nodeid="1141">
<p data-nodeid="1142">02:10: 喝完了</p>
</li>
</ul>
<p data-nodeid="1143">但是假设他采用第3种方法&nbsp;scheduleWithFixedDelay，时间间隔同样设置为 1 小时，那么由于每次喝咖啡需要10分钟，而&nbsp;scheduleWithFixedDelay 是以任务完成的时间为时间起点开始计时的，所以第2次喝咖啡的时间将会在1:10，而不是1:00整，以下是时间表：</p>
<ul data-nodeid="1144">
<li data-nodeid="1145">
<p data-nodeid="1146">00:00: 开始喝咖啡</p>
</li>
<li data-nodeid="1147">
<p data-nodeid="1148">00:10:&nbsp;喝完了</p>
</li>
<li data-nodeid="1149">
<p data-nodeid="1150">01:10: 开始喝咖啡</p>
</li>
<li data-nodeid="1151">
<p data-nodeid="1152">01:20: 喝完了</p>
</li>
<li data-nodeid="1153">
<p data-nodeid="1154">02:20: 开始喝咖啡</p>
</li>
<li data-nodeid="1155">
<p data-nodeid="1156">02:30: 喝完了</p>
</li>
</ul>
<h3 data-nodeid="1157">SingleThreadExecutor</h3>
<p data-nodeid="1158">第四种线程池是 SingleThreadExecutor，它会使用唯一的线程去执行任务，原理和 FixedThreadPool 是一样的，只不过这里线程只有一个，如果线程在执行任务的过程中发生异常，线程池也会重新创建一个线程来执行后续的任务。这种线程池由于只有一个线程，所以非常适合用于所有任务都需要按被提交的顺序依次执行的场景，而前几种线程池不一定能够保障任务的执行顺序等于被提交的顺序，因为它们是多线程并行执行的。</p>
<h3 data-nodeid="1159">SingleThreadScheduledExecutor</h3>
<p data-nodeid="1160">第五个线程池是 SingleThreadScheduledExecutor，它实际和第三种 ScheduledThreadPool 线程池非常相似，它只是 ScheduledThreadPool 的一个特例，内部只有一个线程，如源码所示：</p>
<pre class="lang-java" data-nodeid="1161"><code data-language="java"><span class="hljs-keyword">new</span> ScheduledThreadPoolExecutor(<span class="hljs-number">1</span>)
</code></pre>
<p data-nodeid="1162">它只是将 ScheduledThreadPool 的核心线程数设置为了 1。</p>
<p data-nodeid="1163"><img src="https://s0.lgstatic.com/i/image2/M01/AF/80/CgoB5l3kzomAckv5AAAxf6FCPco696.png" alt="" data-nodeid="1239"></p>
<p data-nodeid="1164">总结上述的五种线程池，我们以核心线程数、最大线程数，以及线程存活时间三个维度进行对比，如表格所示。</p>
<p data-nodeid="1165">第一个线程池 FixedThreadPool，它的核心线程数和最大线程数都是由构造函数直接传参的，而且它们的值是相等的，所以最大线程数不会超过核心线程数，也就不需要考虑线程回收的问题，如果没有任务可执行，线程仍会在线程池中存活并等待任务。</p>
<p data-nodeid="1166">第二个线程池 CachedThreadPool 的核心线程数是 0，而它的最大线程数是 Integer 的最大值，线程数一般是达不到这么多的，所以如果任务特别多且耗时的话，CachedThreadPool 就会创建非常多的线程来应对。</p>
<p data-nodeid="1167">同理，你可以课后按照同样的方法来分析后面三种线程池的参数，来加深对知识的理解。</p>
<h3 data-nodeid="1168">ForkJoinPool</h3>
<p data-nodeid="1169"><img src="https://s0.lgstatic.com/i/image2/M01/AF/A0/CgotOV3kzomAflZxAAB99x9-MzI241.png" alt="" data-nodeid="1246"></p>
<p data-nodeid="2189" class="te-preview-highlight">最后，我们来看下第六种线程池 ForkJoinPool，这个线程池是在 JDK 7 加入的，它的名字 ForkJoin 也描述了它的执行机制，主要用法和之前的线程池是相同的，也是把任务交给线程池去执行，线程池中也有任务队列来存放任务。但是 ForkJoinPool 线程池和之前的线程池有两点非常大的不同之处。第一点它非常适合执行可以产生子任务的任务。</p>


<p data-nodeid="1171">如图所示，我们有一个 Task，这个 Task 可以产生三个子任务，三个子任务并行执行完毕后将结果汇总给 Result，比如说主任务需要执行非常繁重的计算任务，我们就可以把计算拆分成三个部分，这三个部分是互不影响相互独立的，这样就可以利用 CPU 的多核优势，并行计算，然后将结果进行汇总。这里面主要涉及两个步骤，第一步是拆分也就是 Fork，第二步是汇总也就是 Join，到这里你应该已经了解到 ForkJoinPool 线程池名字的由来了。</p>
<p data-nodeid="1172">举个例子，比如面试中经常考到的菲波那切数列，你一定非常熟悉，这个数列的特点就是后一项的结果等于前两项的和，第 0 项是 0，第 1 项是 1，那么第 2 项就是 0+1=1，以此类推。我们在写代码时应该首选效率更高的迭代形式或者更高级的乘方或者矩阵公式法等写法，不过假设我们写成了最初版本的递归形式，伪代码如下所示：</p>
<pre class="lang-java" data-nodeid="1173"><code data-language="java"><span class="hljs-keyword">if</span>&nbsp;(n&nbsp;&lt;=&nbsp;<span class="hljs-number">1</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;n;
&nbsp;}&nbsp;<span class="hljs-keyword">else</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;Fib&nbsp;f1&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Fib(n&nbsp;-&nbsp;<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;Fib&nbsp;f2&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Fib(n&nbsp;-&nbsp;<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;f1.solve();
&nbsp;&nbsp;&nbsp;&nbsp;f2.solve();
&nbsp;&nbsp;&nbsp;&nbsp;number&nbsp;=&nbsp;f1.number&nbsp;+&nbsp;f2.number;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;number;
&nbsp;}
</code></pre>
<p data-nodeid="1174">你可以看到如果 n&lt;=1 则直接返回 n，如果 n&gt;1 ，先将前一项 f1 的值计算出来，然后往前推两项求出 f2 的值，然后将两值相加得到结果，所以我们看到在求和运算中产生了两个子任务。计算 f(4) 的流程如下图所示。</p>
<p data-nodeid="1175"><img src="https://s0.lgstatic.com/i/image2/M01/AF/80/CgoB5l3kzoqAZgXiAACbX2rJCR4889.png" alt="" data-nodeid="1254"></p>
<p data-nodeid="1176">在计算 f(4) 时需要首先计算出 f(2) 和 f(3)，而同理，计算 f(3) 时又需要计算 f(1) 和 f(2)，以此类推。<br>
<img src="https://s0.lgstatic.com/i/image2/M01/AF/A0/CgotOV3kzoqAUlPyAADYOKK1PgM516.png" alt="" data-nodeid="1258"></p>
<p data-nodeid="1177">这是典型的递归问题，对应到我们的 ForkJoin 模式，如图所示，子任务同样会产生子子任务，最后再逐层汇总，得到最终的结果。</p>
<p data-nodeid="1178">ForkJoinPool 线程池有多种方法可以实现任务的分裂和汇总，其中一种用法如下方代码所示。</p>
<pre class="lang-java" data-nodeid="1179"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Fibonacci</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">RecursiveTask</span>&lt;<span class="hljs-title">Integer</span>&gt;&nbsp;</span>{&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">int</span>&nbsp;n;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">Fibonacci</span><span class="hljs-params">(<span class="hljs-keyword">int</span>&nbsp;n)</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.n&nbsp;=&nbsp;n;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;Integer&nbsp;<span class="hljs-title">compute</span><span class="hljs-params">()</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(n&nbsp;&lt;=&nbsp;<span class="hljs-number">1</span>)&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;n;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;Fibonacci&nbsp;f1&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Fibonacci(n&nbsp;-&nbsp;<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;f1.fork();
&nbsp;&nbsp;&nbsp;&nbsp;Fibonacci&nbsp;f2&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Fibonacci(n&nbsp;-&nbsp;<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;f2.fork();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;f1.join()&nbsp;+&nbsp;f2.join();
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;}
</code></pre>
<p data-nodeid="1180">我们看到它首先继承了 RecursiveTask，RecursiveTask 类是对ForkJoinTask 的一个简单的包装，这时我们重写 compute() 方法，当 n&lt;=1 时直接返回，当 n&gt;1 就创建递归任务，也就是 f1 和 f2，然后我们用 fork() 方法分裂任务并分别执行，最后在 return 的时候，使用 join() 方法把结果汇总，这样就实现了任务的分裂和汇总。</p>
<pre class="lang-java" data-nodeid="1181"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;ExecutionException,&nbsp;InterruptedException&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;ForkJoinPool&nbsp;forkJoinPool&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ForkJoinPool();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;<span class="hljs-number">10</span>;&nbsp;i++)&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ForkJoinTask&nbsp;task&nbsp;=&nbsp;forkJoinPool.submit(<span class="hljs-keyword">new</span>&nbsp;Fibonacci(i));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(task.get());
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;}
</code></pre>
<p data-nodeid="1182">上面这段代码将会打印出斐波那契数列的第 0 到 9 项的值：</p>
<pre class="lang-java" data-nodeid="1183"><code data-language="java"><span class="hljs-number">0</span>
<span class="hljs-number">1</span>
<span class="hljs-number">1</span>
<span class="hljs-number">2</span>
<span class="hljs-number">3</span>
<span class="hljs-number">5</span>
<span class="hljs-number">8</span>
<span class="hljs-number">13</span>
<span class="hljs-number">21</span>
<span class="hljs-number">34</span>
</code></pre>
<p data-nodeid="1184">这就是&nbsp;ForkJoinPool 线程池和其他线程池的第一点不同。</p>
<p data-nodeid="1185">我们来看第二点不同，第二点不同之处在于内部结构，之前的线程池所有的线程共用一个队列，但 ForkJoinPool 线程池中每个线程都有自己独立的任务队列，如图所示。</p>
<p data-nodeid="1186"><img src="https://s0.lgstatic.com/i/image2/M01/AF/80/CgoB5l3kzouAdfLfAAARK97hw4g233.png" alt="" data-nodeid="1268"></p>
<p data-nodeid="1187">ForkJoinPool 线程池内部除了有一个共用的任务队列之外，每个线程还有一个对应的双端队列 deque，这时一旦线程中的任务被 Fork 分裂了，分裂出来的子任务放入线程自己的 deque 里，而不是放入公共的任务队列中。如果此时有三个子任务放入线程 t1 的 deque 队列中，对于线程 t1 而言获取任务的成本就降低了，可以直接在自己的任务队列中获取而不必去公共队列中争抢也不会发生阻塞（除了后面会讲到的 steal 情况外），减少了线程间的竞争和切换，是非常高效的。</p>
<p data-nodeid="1188"><img src="https://s0.lgstatic.com/i/image2/M01/B0/57/CgotOV3nFTCAKmNtAAES7A18i8M873.png" alt="" data-nodeid="1271"></p>
<p data-nodeid="1189">我们再考虑一种情况，此时线程有多个，而线程 t1 的任务特别繁重，分裂了数十个子任务，但是 t0 此时却无事可做，它自己的 deque 队列为空，这时为了提高效率，t0 就会想办法帮助 t1 执行任务，这就是“work-stealing”的含义。</p>
<p data-nodeid="1190">双端队列 deque 中，线程 t1 获取任务的逻辑是后进先出，也就是LIFO（Last In Frist Out），而线程 t0 在“steal”偷线程 t1 的 deque 中的任务的逻辑是先进先出，也就是FIFO（Fast In Frist Out），如图所示，图中很好的描述了两个线程使用双端队列分别获取任务的情景。你可以看到，使用 “work-stealing” 算法和双端队列很好地平衡了各线程的负载。</p>
<p data-nodeid="1191"><img src="https://s0.lgstatic.com/i/image2/M01/B0/37/CgoB5l3nFSOAFOkbAABvJKvhTKk938.png" alt="" data-nodeid="1275"></p>
<p data-nodeid="1192">最后，我们用一张全景图来描述 ForkJoinPool 线程池的内部结构，你可以看到 ForkJoinPool 线程池和其他线程池很多地方都是一样的，但重点区别在于它每个线程都有一个自己的双端队列来存储分裂出来的子任务。ForkJoinPool 非常适合用于递归的场景，例如树的遍历、最优路径搜索等场景。</p>

---

### 精选评论

##### **明：
> ScheduledThreadPool&nbsp;&nbsp;&nbsp;&nbsp; 执行周期性任务,假设某次出现了异常,周期任务还会继续吗<br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 查看ScheduledExecutorService接口里的scheduleWithFixedDelay方法的jdk文档，有描述如下：

If any execution of the task encounters an exception, subsequent executions are suppressed.
如果在任务的执行中遇到异常，后续执行被取消。

 ScheduledThreadPoolExecutor的最优实践：将所有执行代码用try-catch包裹。

##### **生：
> 最有良心的作者，收下我的膝盖

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 谢谢您的支持

##### **正：
> 通俗易懂，点赞

##### SummerPan：
> SingleThreadScheduledExecutor的maxPoolSize没有设置成1或者同corePoolSize,意味着多个任务时也会创建超过一个线程吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的，因为它的队列是无界队列，不会满，不会轮到创建最大线程数量个线程的步骤。

##### **峰：
> 666

##### **0220：
> “<span style="font-size: 16.0125px;">线程 t1 的任务特别繁重，分裂了数十个子任务，但是 t0 此时却无事可做，它自己的 deque 队列为空</span>”<div>作者你好，请问一下，为什么会出现这种情况？</div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 每个线程被分到的任务可以是不一样的，有的任务重，有的任务轻。

##### 魏：
> 讲的太好了，感谢作者

##### *豪：
> 加油

##### **滔：
> FixedThreadPool： 固定线程池。核心线程数由构造参数传入，最大线程数=核心线程数。CachedThreadPool: 缓存线程池。核心线程数为0，最大线程数为  2^31-1 。 队列的容量为0 （ SynchronousQueue ）。ScheduledThreadPool: 定时线程池。可延迟x秒，可延迟x秒，按y周期执行（起始点有开始或结束）。SingleThreadPool：单一线程池。和FixedThreadPool差不多，区别在于只有一个核心线程数。SingleThreadScheduledPool: 单一定时线程池。和ScheduledThreadPool差不多，区别在于内部只有一个线程。ForkJoinPool（java8才有）：拆分汇总线程池。特点1：任务可以再次分裂子任务，并且可以汇总子任务的数据。特点2：每个任务都有一个自己的阻塞队列，而非共同拥有一个线程池阻塞队列。常用于递归场景（树的遍历，最有路径搜索）。

##### **皓：
> forkjoinpool和后面的Add类的cell数组在设计思想上有点像。

##### **润：
> 最后那个例子如果能加个存储变成动态规划就完美了😀

##### *桃：
> FixedThreadPool 核心和最大线程数一样CacheThreadPool 核心线程数为零，最大线程数为2^31-1ScheduledThreadPool 周期性的执行任务SingleThreadExecutor 只有一个线程执行任务SingleScheduleThreafPool 核心线程为1的定时任务线程ForkJoinPool

##### zhangsan：
> 老师你好，看了上面得表格,maxPoolSize值为1、corePoolSize或Integer.MAX_VALUE。那什么场景下会出现其他值呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 自己定义线程池时，可以对该值进行其他值的设置。

