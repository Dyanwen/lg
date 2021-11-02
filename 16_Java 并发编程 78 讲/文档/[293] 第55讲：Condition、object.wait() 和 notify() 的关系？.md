<p data-nodeid="465" class="">本课时我们主要介绍 Condition、Object 的 wait() 和 notify()&nbsp;的关系。</p>
<p data-nodeid="466">下面先讲一下 Condition 这个接口，来看看它的作用、如何使用，以及需要注意的点有哪些。</p>
<h3 data-nodeid="467">Condition接口</h3>
<h4 data-nodeid="468">作用</h4>
<p data-nodeid="581" class="">我们假设线程 1 需要等待某些条件满足后，才能继续运行，这个条件会根据业务场景不同，有不同的可能性，比如等待某个时间点到达或者等待某些任务处理完毕。在这种情况下，我们就可以执行 Condition 的 await 方法，一旦执行了该方法，这个线程就会进入 WAITING 状态。</p>

<p data-nodeid="1043" class="te-preview-highlight">通常会有另外一个线程，我们把它称作线程 2，它去达成对应的条件，直到这个条件达成之后，那么，线程 2 调用 Condition 的 signal 方法 [或 signalAll 方法]，代表“<strong data-nodeid="1053">这个条件已经达成了，之前等待这个条件的线程现在可以苏醒了</strong>”。这个时候，JVM 就会找到等待该 Condition 的线程，并予以唤醒，根据调用的是 signal 方法或 signalAll 方法，会唤醒 1 个或所有的线程。于是，线程 1 在此时就会被唤醒，然后它的线程状态又会回到 Runnable 可执行状态。</p>


<h4 data-nodeid="471">代码案例</h4>
<p data-nodeid="472">我们用一个代码来说明这个问题，如下所示：</p>
<pre class="lang-java" data-nodeid="473"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">ConditionDemo</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;ReentrantLock&nbsp;lock&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ReentrantLock();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;Condition&nbsp;condition&nbsp;=&nbsp;lock.newCondition();

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">method1</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.lock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(Thread.currentThread().getName()+<span class="hljs-string">":条件不满足，开始await"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;condition.await();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(Thread.currentThread().getName()+<span class="hljs-string">":条件满足了，开始执行后续的任务"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.unlock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">method2</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.lock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(Thread.currentThread().getName()+<span class="hljs-string">":需要5秒钟的准备时间"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">5000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(Thread.currentThread().getName()+<span class="hljs-string">":准备工作完成，唤醒其他的线程"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;condition.signal();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.unlock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ConditionDemo&nbsp;conditionDemo&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ConditionDemo();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(<span class="hljs-keyword">new</span>&nbsp;Runnable()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;conditionDemo.method2();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}).start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;conditionDemo.method1();
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="474">在这个代码中，有以下三个方法。</p>
<ul data-nodeid="475">
<li data-nodeid="476">
<p data-nodeid="477"><strong data-nodeid="542">method1</strong>，它代表主线程将要执行的内容，首先获取到锁，打印出“条件不满足，开始 await”，然后调用 condition.await() 方法，直到条件满足之后，则代表这个语句可以继续向下执行了，于是打印出“条件满足了，开始执行后续的任务”，最后会在 finally 中解锁。</p>
</li>
<li data-nodeid="478">
<p data-nodeid="479"><strong data-nodeid="547">method2</strong>，它同样也需要先获得锁，然后打印出“需要 5 秒钟的准备时间”，接着用 sleep 来模拟准备时间；在时间到了之后，则打印出“准备工作完成”，最后调用&nbsp;condition.signal() 方法，把之前已经等待的线程唤醒。</p>
</li>
<li data-nodeid="480">
<p data-nodeid="481"><strong data-nodeid="552">main 方法</strong>，它的主要作用是执行上面这两个方法，它先去实例化我们这个类，然后再用子线程去调用这个类的 method2 方法，接着用主线程去调用 method1 方法。</p>
</li>
</ul>
<p data-nodeid="482">最终这个代码程序运行结果如下所示：</p>
<pre class="lang-java" data-nodeid="483"><code data-language="java">main:条件不满足，开始&nbsp;await
Thread-<span class="hljs-number">0</span>:需要&nbsp;<span class="hljs-number">5</span>&nbsp;秒钟的准备时间
Thread-<span class="hljs-number">0</span>:准备工作完成，唤醒其他的线程
main:条件满足了，开始执行后续的任务
</code></pre>
<p data-nodeid="484">同时也可以看到，打印这行语句它所运行的线程，第一行语句和第四行语句打印的是在 main 线程中，也就是在主线程中去打印的，而第二、第三行是在子线程中打印的。这个代码就模拟了我们前面所描述的场景。</p>
<h4 data-nodeid="485">注意点</h4>
<p data-nodeid="486">下面我们来看一下，在使用 Condition 的时候有哪些注意点。</p>
<ul data-nodeid="487">
<li data-nodeid="488">
<p data-nodeid="489">线程 2 解锁后，线程 1 才能获得锁并继续执行</p>
</li>
</ul>
<p data-nodeid="490">线程 2 对应刚才代码中的子线程，而线程 1 对应主线程。这里需要额外注意，并不是说子线程调用了 signal 之后，主线程就可以立刻被唤醒去执行下面的代码了，而是说在调用了 signal 之后，还需要等待子线程完全退出这个锁，即执行 unlock 之后，这个主线程才有可能去获取到这把锁，并且当获取锁成功之后才能继续执行后面的任务。刚被唤醒的时候主线程还没有拿到锁，是没有办法继续往下执行的。</p>
<ul data-nodeid="491">
<li data-nodeid="492">
<p data-nodeid="493">signalAll() 和 signal() 区别</p>
</li>
</ul>
<p data-nodeid="494">signalAll() 会唤醒所有正在等待的线程，而 signal() 只会唤醒一个线程。</p>
<h3 data-nodeid="495">用 Condition 和 wait/notify 实现简易版阻塞队列</h3>
<p data-nodeid="496">在第 05 讲，讲过如何用 Condition 和 wait/notify 来实现生产者/消费者模式，其中的精髓就在于用 Condition 和 wait/notify 来实现简易版阻塞队列，我们来分别回顾一下这两段代码。</p>
<h4 data-nodeid="497">用 Condition 实现简易版阻塞队列</h4>
<p data-nodeid="498">代码如下所示：</p>
<pre class="lang-java" data-nodeid="499"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">MyBlockingQueueForCondition</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;Queue&nbsp;queue;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;max&nbsp;=&nbsp;<span class="hljs-number">16</span>;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;ReentrantLock&nbsp;lock&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ReentrantLock();
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;Condition&nbsp;notEmpty&nbsp;=&nbsp;lock.newCondition();
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;Condition&nbsp;notFull&nbsp;=&nbsp;lock.newCondition();
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">MyBlockingQueueForCondition</span><span class="hljs-params">(<span class="hljs-keyword">int</span>&nbsp;size)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.max&nbsp;=&nbsp;size;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;queue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedList();
&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">put</span><span class="hljs-params">(Object&nbsp;o)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.lock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(queue.size()&nbsp;==&nbsp;max)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notFull.await();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;queue.add(o);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notEmpty.signalAll();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.unlock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;Object&nbsp;<span class="hljs-title">take</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.lock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(queue.size()&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notEmpty.await();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Object&nbsp;item&nbsp;=&nbsp;queue.remove();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notFull.signalAll();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;item;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.unlock();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="500">在上面的代码中，首先定义了一个队列变量 queue，其最大容量是 16；然后定义了一个 ReentrantLock 类型的 Lock 锁，并在 Lock 锁的基础上创建了两个 Condition，一个是 notEmpty，另一个是 notFull，分别代表队列没有空和没有满的条件；最后，声明了 put 和 take 这两个核心方法。</p>
<h4 data-nodeid="501">用 wait/notify 实现简易版阻塞队列</h4>
<p data-nodeid="502">我们再来看看如何使用 wait/notify 来实现简易版阻塞队列，代码如下：</p>
<pre class="lang-java" data-nodeid="503"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">MyBlockingQueueForWaitNotify</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;maxSize;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;LinkedList&lt;Object&gt;&nbsp;storage;
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">MyBlockingQueueForWaitNotify</span>&nbsp;<span class="hljs-params">(<span class="hljs-keyword">int</span>&nbsp;size)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.maxSize&nbsp;=&nbsp;size;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;storage&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedList&lt;&gt;();
&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">put</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(storage.size()&nbsp;==&nbsp;maxSize)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.wait();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;storage.add(<span class="hljs-keyword">new</span>&nbsp;Object());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.notifyAll();
&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">take</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(storage.size()&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.wait();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(storage.remove());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.notifyAll();
&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="504">如代码所示，最主要的部分仍是 put 与 take 方法。我们先来看 put 方法，该方法被 synchronized 保护，while 检查 List 是否已满，如果不满就往里面放入数据，并通过 notifyAll() 唤醒其他线程。同样，take 方法也被 synchronized 修饰，while 检查 List 是否为空，如果不为空则获取数据并唤醒其他线程。</p>
<p data-nodeid="505">在第 05 讲，有对这两段代码的详细讲解，遗忘的小伙伴可以到前面复习一下。</p>
<h4 data-nodeid="506">Condition 和 wait/notify的关系</h4>
<p data-nodeid="507">对比上面两种实现方式的 put 方法，会发现非常类似，此时让我们把这两段代码同时列在屏幕中，然后进行对比：</p>
<p data-nodeid="508">左：</p>
<pre class="lang-java" data-nodeid="509"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">put</span><span class="hljs-params">(Object&nbsp;o)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;lock.lock();
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(queue.size()&nbsp;==&nbsp;max)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;condition1.await();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;queue.add(o);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;condition2.signalAll();
&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lock.unlock();
&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="510">右：</p>
<pre class="lang-java" data-nodeid="511"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">put</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(storage.size()&nbsp;==&nbsp;maxSize)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.wait();
&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;storage.add(<span class="hljs-keyword">new</span>&nbsp;Object());
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.notifyAll();
}
</code></pre>
<p data-nodeid="512">可以看出，左侧是 Condition 的实现，右侧是 wait/notify 的实现：</p>
<pre class="lang-java" data-nodeid="513"><code data-language="java">lock.lock()&nbsp;对应进入&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;方法
condition.await()&nbsp;对应&nbsp;object.wait()
condition.signalAll()&nbsp;对应&nbsp;object.notifyAll()
lock.unlock()&nbsp;对应退出&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;方法
</code></pre>
<p data-nodeid="514">实际上，如果说 Lock 是用来代替 synchronized 的，那么 Condition 就是用来代替相对应的 Object 的 wait/notify/notifyAll，所以在用法和性质上几乎都一样。</p>
<p data-nodeid="515">Condition 把 Object 的 wait/notify/notifyAll 转化为了一种相应的对象，其实现的效果基本一样，但是把更复杂的用法，变成了更直观可控的对象方法，是一种升级。</p>
<p data-nodeid="516">await 方法会自动释放持有的 Lock 锁，和 Object 的 wait 一样，不需要自己手动释放锁。</p>
<p data-nodeid="517">另外，调用 await 的时候必须持有锁，否则会抛出异常，这一点和 Object 的 wait 一样。</p>
<h3 data-nodeid="518">总结</h3>
<p data-nodeid="519" class="">首先介绍了 Condition 接口的作用，并给出了基本用法；然后讲解了它的几个注意点，复习了之前 Condition 和 wait/notify 实现简易版阻塞队列的代码，并且对这两种方法，不同的实现进行了对比；最后分析了它们之间的关系。</p>

---

### 精选评论

##### **鹏：
> 在Debug模式下，有可能method2的方法会先执行造成死锁，一直等待。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可能会一直等待，但是不属于死锁，因为await会释放锁。

##### **文：
> 在调用condition的await方法后，线程在哪儿排队等待呢，是在condition所在的lock的aqs里的队列里，还是condition自己维护了一个队列呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有Condition自己的。

##### **铁：
> 如果匿名线程先于main线程的 conditionDemo.method1(); 之前执行，也就是conditionDemo.method2(); 先执行，则 method2 先拿到锁，先执行完 method2 的方法块，然后再执行conditionDemo.method1(); 此时 main 线程会一直停留在condition.await(); 解决方法可以在ditionDemo.method2(); 前，匿名线程先 sleep 一秒，保证conditionDemo.method1() 先执行。

##### *军：
> 第一段代码的metod1为什么是主线程？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为conditionDemo.method1()是在写在main方法里的。

##### *露：
> Condition的await在调用时，内部会调用release去释放同步状态，来达到唤醒后继等待线程目的，从而实现释放锁的表象，实际是对于LockSupport的park和unpark的灵活运用。

