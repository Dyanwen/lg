<p>在本课时我们主要学习如何正确关闭线程池？以及 shutdown() 与 shutdownNow() 方法的区别？首先，我们创建一个线程数固定为 10 的线程池，并且往线程池中提交 100 个任务，如代码所示。</p>
<pre><code data-language="java" class="lang-java">ExecutorService&nbsp;service&nbsp;=&nbsp;Executors.newFixedThreadPool(<span class="hljs-number">10</span>);
&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;<span class="hljs-number">100</span>;&nbsp;i++)&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;service.execute(<span class="hljs-keyword">new</span>&nbsp;Task());
&nbsp;}
</code></pre>
<p>那么如果现在我们想关闭该线程池该如何做呢？本课时主要介绍 5 种在 ThreadPoolExecutor 中涉及关闭线程池的方法，如下所示。</p>
<ul>
<li>void shutdown;</li>
<li>boolean isShutdown;</li>
<li>boolean isTerminated;</li>
<li>boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;</li>
<li>List&lt;Runnable&gt; shutdownNow;</li>
</ul>
<p>下面我们就对这些方法逐一展开。</p>
<h4>shutdown()</h4>
<p>第一种方法叫作 shutdown()，它可以安全地关闭一个线程池，调用 shutdown() 方法之后线程池并不是立刻就被关闭，因为这时线程池中可能还有很多任务正在被执行，或是任务队列中有大量正在等待被执行的任务，调用 shutdown() 方法后线程池会在执行完正在执行的任务和队列中等待的任务后才彻底关闭。但这并不代表 shutdown() 操作是没有任何效果的，调用 shutdown() 方法后如果还有新的任务被提交，线程池则会根据拒绝策略直接拒绝后续新提交的任务。</p>
<h4>isShutdown()</h4>
<p>第二个方法叫作 isShutdown()，它可以返回 true 或者 false 来判断线程池是否已经开始了关闭工作，也就是是否执行了 shutdown 或者 shutdownNow 方法。这里需要注意，如果调用 isShutdown() 方法的返回的结果为 true 并不代表线程池此时已经彻底关闭了，这仅仅代表线程池开始了关闭的流程，也就是说，此时可能线程池中依然有线程在执行任务，队列里也可能有等待被执行的任务。</p>
<h4>isTerminated()</h4>
<p>第三种方法叫作 isTerminated()，这个方法可以检测线程池是否真正“终结”了，这不仅代表线程池已关闭，同时代表线程池中的所有任务都已经都执行完毕了，因为我们刚才说过，调用 shutdown 方法之后，线程池会继续执行里面未完成的任务，不仅包括线程正在执行的任务，还包括正在任务队列中等待的任务。比如此时已经调用了 shutdown 方法，但是有一个线程依然在执行任务，那么此时调用 isShutdown 方法返回的是 true ，而调用 isTerminated 方法返回的便是 false ，因为线程池中还有任务正在在被执行，线程池并没有真正“终结”。直到所有任务都执行完毕了，调用 isTerminated() 方法才会返回 true，这表示线程池已关闭并且线程池内部是空的，所有剩余的任务都执行完毕了。</p>
<h4>awaitTermination()</h4>
<p>第四个方法叫作 awaitTermination()，它本身并不是用来关闭线程池的，而是主要用来判断线程池状态的。比如我们给 awaitTermination 方法传入的参数是 10 秒，那么它就会陷入 10 秒钟的等待，直到发生以下三种情况之一：</p>
<ol>
<li>等待期间（包括进入等待状态之前）线程池已关闭并且所有已提交的任务（包括正在执行的和队列中等待的）都执行完毕，相当于线程池已经“终结”了，方法便会返回 true；</li>
<li>等待超时时间到后，第一种线程池“终结”的情况始终未发生，方法返回 false；</li>
<li>等待期间线程被中断，方法会抛出 InterruptedException 异常。</li>
</ol>
<p>也就是说，调用 awaitTermination 方法后当前线程会尝试等待一段指定的时间，如果在等待时间内，线程池已关闭并且内部的任务都执行完毕了，也就是说线程池真正“终结”了，那么方法就返回 true，否则超时返回 fasle。</p>
<p>我们则可以根据 awaitTermination() 返回的布尔值来判断下一步应该执行的操作。</p>
<h4>shutdownNow()</h4>
<p>最后一个方法是 shutdownNow()，也是 5 种方法里功能最强大的，它与第一种 shutdown 方法不同之处在于名字中多了一个单词 Now，也就是表示立刻关闭的意思。在执行 shutdownNow 方法之后，首先会给所有线程池中的线程发送 interrupt 中断信号，尝试中断这些任务的执行，然后会将任务队列中正在等待的所有任务转移到一个 List 中并返回，我们可以根据返回的任务 List 来进行一些补救的操作，例如记录在案并在后期重试。shutdownNow() 的源码如下所示。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;List&lt;Runnable&gt;&nbsp;<span class="hljs-title">shutdownNow</span><span class="hljs-params">()</span>&nbsp;</span>{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;List&lt;Runnable&gt;&nbsp;tasks;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;ReentrantLock&nbsp;mainLock&nbsp;=&nbsp;<span class="hljs-keyword">this</span>.mainLock;
&nbsp;&nbsp;&nbsp;&nbsp;mainLock.lock();

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;checkShutdownAccess();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;advanceRunState(STOP);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;interruptWorkers();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tasks&nbsp;=&nbsp;drainQueue();
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;mainLock.unlock();
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;tryTerminate();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;tasks;
&nbsp;}
</code></pre>
<p>你可以看到源码中有一行&nbsp;interruptWorkers() 代码，这行代码会让每一个已经启动的线程都中断，这样线程就可以在执行任务期间检测到中断信号并进行相应的处理，提前结束任务。这里需要注意的是，由于 Java 中不推荐强行停止线程的机制的限制，即便我们调用了 shutdownNow 方法，如果被中断的线程对于中断信号不理不睬，那么依然有可能导致任务不会停止。可见我们在开发中落地最佳实践是很重要的，我们自己编写的线程应当具有响应中断信号的能力，正确停止线程的方法在第 2 讲有讲过，应当利用中断信号来协同工作。</p>
<p>在掌握了这 5 种关闭线程池相关的方法之后，我们就可以根据自己的业务需要，选择合适的方法来停止线程池，比如通常我们可以用 shutdown() 方法来关闭，这样可以让已提交的任务都执行完毕，但是如果情况紧急，那我们就可以用 shutdownNow 方法来加快线程池“终结”的速度。</p>

---

### 精选评论

##### **诚：
> 如果只是想中断线程池某个任务呢？这个要怎么操作？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以用Future的cancel方法。

##### **光：
> 正常停止线程的本质，是让线程任务的run方法执行完，对应到线程池的线程，就是Worker类的run方法执行完。这个run方法执行完的前提是getTask()返回的是null。所以可以去看ThreadPoolExecutor类的getTask()方法，当线程池状态是运行时，调用阻塞队列的take()方法，该方法在队列为空时阻塞，即线程在这种情况下不会结束，当线程池状态是关闭时，调用阻塞队列的poll()方法获取任务，该方法在队列为空时返回null，不会阻塞，也就是会结束线程。

##### **林：
> 在实际项目中,模块全局单例线程池可以不关闭吧?关闭之后再次调用,任务提交后并不执行呀,只执行第一次调用时提交的任务啊!

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以不关闭。

##### **光：
> 老师讲的真细致！赞👍🏻

##### **来：
> 什么情况下需要关闭,线程池不是一直存在的吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 线程池不关闭的话，里面的线程会一直存在。在使用完线程池后，可以选择关闭。

##### *杰：
> 想问下&nbsp;<span style="font-size: 16.0125px;">isShutdown&nbsp;isTerminated 这2个方法只是获取状态 还是由关闭的功能？？ 看源码好像并没有关闭功能 只是做了一些判断</span>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，没有关闭，只是判断。

##### **滔：
> shutdown()：调用此方法，线程池不会马上关闭，会等线程运行完 ，并且阻塞的任务运行完再关闭。isShutdown()：判断线程池是否被标记关闭。调用了shutdown方法后，此方法会返回true。isTerminated()：判断线程池中是否已关闭并且阻塞的任务都已执行完 。awaitTermination()：判断线程池终结状态。等待周期内，线程池终结会返回true。超过等待时间，线程池未终结会返回false。等待周期内，线程被中断会抛出InterruptedException异常。shutdownNow()：表示立即关闭线程池。会向所有线程发送中断信号，并停止线程。将等待中的任务转移到list中，以后可做补救措施。

