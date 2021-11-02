<p data-nodeid="457" class="">在本课时我们主要学习如何正确停止一个线程？以及为什么用 volatile 标记位的停止方法是错误的？</p>
<p data-nodeid="458" class="">首先，我们来复习如何启动一个线程，想要启动线程需要调用 Thread 类的 start() 方法，并在 run() 方法中定义需要执行的任务。启动一个线程非常简单，但如果想要正确停止它就没那么容易了。</p>
<h3 data-nodeid="459">原理介绍</h3>
<p data-nodeid="460" class="">通常情况下，我们不会手动停止一个线程，而是允许线程运行到结束，然后让它自然停止。但是依然会有许多特殊的情况需要我们提前停止线程，比如：用户突然关闭程序，或程序运行出错重启等。</p>
<p data-nodeid="461">在这种情况下，即将停止的线程在很多业务场景下仍然很有价值。尤其是我们想写一个健壮性很好，能够安全应对各种场景的程序时，正确停止线程就显得格外重要。但是Java 并没有提供简单易用，能够直接安全停止线程的能力。</p>
<h3 data-nodeid="462">为什么不强制停止？而是通知、协作</h3>
<p data-nodeid="463">对于 Java 而言，最正确的停止线程的方式是使用&nbsp;interrupt。但 interrupt 仅仅起到通知被停止线程的作用。而对于被停止的线程而言，它拥有完全的自主权，它既可以选择立即停止，也可以选择一段时间后停止，也可以选择压根不停止。那么为什么 Java 不提供强制停止线程的能力呢？</p>
<p data-nodeid="464">事实上，Java 希望程序间能够相互通知、相互协作地管理线程，因为如果不了解对方正在做的工作，贸然强制停止线程就可能会造成一些安全的问题，为了避免造成问题就需要给对方一定的时间来整理收尾工作。比如：线程正在写入一个文件，这时收到终止信号，它就需要根据自身业务判断，是选择立即停止，还是将整个文件写入成功后停止，而如果选择立即停止就可能造成数据不完整，不管是中断命令发起者，还是接收者都不希望数据出现问题。</p>
<h3 data-nodeid="465">如何用 interrupt 停止线程</h3>
<pre class="lang-java" data-nodeid="571"><code data-language="java"><span class="hljs-keyword">while</span>&nbsp;(!Thread.currentThread().isInterrupted()&nbsp;&amp;&amp;&nbsp;more&nbsp;work&nbsp;to&nbsp;<span class="hljs-keyword">do</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">do</span>&nbsp;more&nbsp;work
}
</code></pre>
<p data-nodeid="572">明白 Java 停止线程的设计原则之后，我们看看如何用代码实现停止线程的逻辑。我们一旦调用某个线程的 interrupt() 之后，这个线程的中断标记位就会被设置成 true。每个线程都有这样的标记位，当线程执行时，应该定期检查这个标记位，如果标记位被设置成 true，就说明有程序想终止该线程。回到源码，可以看到在 while 循环体判断语句中，首先通过 Thread.currentThread().isInterrupt() 判断线程是否被中断，随后检查是否还有工作要做。&amp;&amp; 逻辑表示只有当两个判断条件同时满足的情况下，才会去执行下面的工作。</p>
<p data-nodeid="573">我们再看看具体例子。</p>
<pre class="lang-java" data-nodeid="574"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">StopThread</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">Runnable</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">int</span>&nbsp;count&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(!Thread.currentThread().isInterrupted()&nbsp;&amp;&amp;&nbsp;count&nbsp;&lt;&nbsp;<span class="hljs-number">1000</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"count&nbsp;=&nbsp;"</span>&nbsp;+&nbsp;count++);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;thread&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(<span class="hljs-keyword">new</span>&nbsp;StopThread());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">5</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread.interrupt();
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="575">在 StopThread 类的 run() 方法中，首先判断线程是否被中断，然后判断 count 值是否小于 1000。这个线程的工作内容很简单，就是打印 0~999 的数字，每打印一个数字 count 值加 1，可以看到，线程会在每次循环开始之前，检查是否被中断了。接下来在 main 函数中会启动该线程，然后休眠 5 毫秒后立刻中断线程，该线程会检测到中断信号，于是在还没打印完1000个数的时候就会停下来，这种就属于通过 interrupt 正确停止线程的情况。</p>
<h3 data-nodeid="576">sleep 期间能否感受到中断</h3>
<pre class="lang-java" data-nodeid="577"><code data-language="java">Runnable&nbsp;runnable&nbsp;=&nbsp;()&nbsp;-&gt;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">int</span>&nbsp;num&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(!Thread.currentThread().isInterrupted()&nbsp;&amp;&amp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;num&nbsp;&lt;=&nbsp;<span class="hljs-number">1000</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(num);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;num++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;}
};
</code></pre>
<p data-nodeid="578">那么我们考虑一种特殊情况，改写上面的代码，如果线程在执行任务期间有休眠需求，也就是每打印一个数字，就进入一次&nbsp;sleep&nbsp;，而此时将 Thread.sleep()&nbsp;的休眠时间设置为 1000 秒钟。</p>
<pre class="lang-java" data-nodeid="579"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StopDuringSleep</span> </span>{
 
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
 &nbsp; &nbsp; &nbsp; &nbsp;Runnable runnable = () -&gt; {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">int</span> num = <span class="hljs-number">0</span>;
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">try</span> {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">while</span> (!Thread.currentThread().isInterrupted() &amp;&amp; num &lt;= <span class="hljs-number">1000</span>) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;System.out.println(num);
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;num++;
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Thread.sleep(<span class="hljs-number">1000000</span>);
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;} <span class="hljs-keyword">catch</span> (InterruptedException e) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;e.printStackTrace();
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp;};
 &nbsp; &nbsp; &nbsp; &nbsp;Thread thread = <span class="hljs-keyword">new</span> Thread(runnable);
 &nbsp; &nbsp; &nbsp; &nbsp;thread.start();
 &nbsp; &nbsp; &nbsp; &nbsp;Thread.sleep(<span class="hljs-number">5</span>);
 &nbsp; &nbsp; &nbsp; &nbsp;thread.interrupt();
 &nbsp; &nbsp;}
}
</code></pre>
<p data-nodeid="580">主线程休眠 5 毫秒后，通知子线程中断，此时子线程仍在执行 sleep 语句，处于休眠中。那么就需要考虑一点，在休眠中的线程是否能够感受到中断通知呢？是否需要等到休眠结束后才能中断线程呢？如果是这样，就会带来严重的问题，因为响应中断太不及时了。正因为如此，Java 设计者在设计之初就考虑到了这一点。</p>
<p data-nodeid="581">如果 sleep、wait 等可以让线程进入阻塞的方法使线程休眠了，而处于休眠中的线程被中断，那么线程是可以感受到中断信号的，并且会抛出一个 InterruptedException 异常，同时清除中断信号，将中断标记位设置成 false。这样一来就不用担心长时间休眠中线程感受不到中断了，因为即便线程还在休眠，仍然能够响应中断通知，并抛出异常。</p>
<h3 data-nodeid="582">两种最佳处理方式</h3>
<p data-nodeid="583">在实际开发中肯定是团队协作的，不同的人负责编写不同的方法，然后相互调用来实现整个业务的逻辑。那么如果我们负责编写的方法需要被别人调用，同时我们的方法内调用了 sleep 或者 wait 等能响应中断的方法时，仅仅&nbsp;catch 住异常是不够的。</p>
<pre class="lang-java" data-nodeid="584"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">subTas</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;在这里不处理该异常是非常不好的</span>
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="585">我们可以在方法中使用 try/catch 或在方法签名中声明 throws &nbsp;InterruptedException。</p>
<h3 data-nodeid="586">方法签名抛异常，run() 强制 try/catch</h3>
<p data-nodeid="587">我们先来看下 try/catch 的处理逻辑。如上面的代码所示，catch 语句块里代码是空的，它并没有进行任何处理。假设线程执行到这个方法，并且正在 sleep，此时有线程发送&nbsp;interrupt 通知试图中断线程，就会立即抛出异常，并清除中断信号。抛出的异常被 catch 语句块捕捉。</p>
<p data-nodeid="588">但是，捕捉到异常的 catch 没有进行任何处理逻辑，相当于把中断信号给隐藏了，这样做是非常不合理的，那么究竟应该怎么处理呢？首先，可以选择在方法签名中抛出异常。</p>
<pre class="lang-java" data-nodeid="589"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">subTask2</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1000</span>);
}
</code></pre>
<p data-nodeid="590">正如代码所示，要求每一个方法的调用方有义务去处理异常。调用方要不使用 try/catch&nbsp;并在 catch 中正确处理异常，要不将异常声明到方法签名中。如果每层逻辑都遵守规范，便可以将中断信号层层传递到顶层，最终让 run() 方法可以捕获到异常。而对于 run() 方法而言，它本身没有抛出 checkedException 的能力，只能通过 try/catch 来处理异常。层层传递异常的逻辑保障了异常不会被遗漏，而对 run() 方法而言，就可以根据不同的业务逻辑来进行相应的处理。</p>
<h3 data-nodeid="591">再次中断</h3>
<pre class="lang-java" data-nodeid="592"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">reInterrupt</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">2000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.currentThread().interrupt();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="593">除了刚才推荐的将异常声明到方法签名中的方式外，还可以在 catch 语句中再次中断线程。如代码所示，需要在 catch 语句块中调用 Thread.currentThread().interrupt() 函数。因为如果线程在休眠期间被中断，那么会自动清除中断信号。如果这时手动添加中断信号，中断信号依然可以被捕捉到。这样后续执行的方法依然可以检测到这里发生过中断，可以做出相应的处理，整个线程可以正常退出。</p>
<p data-nodeid="594">我们需要注意，我们在实际开发中不能盲目吞掉中断，如果不在方法签名中声明，也不在&nbsp;catch&nbsp;语句块中再次恢复中断，而是在 catch 中不作处理，我们称这种行为是“屏蔽了中断请求”。如果我们盲目地屏蔽了中断请求，会导致中断信号被完全忽略，最终导致线程无法正确停止。</p>
<h3 data-nodeid="595">为什么用 volatile 标记位的停止方法是错误的</h3>
<p data-nodeid="596">下面我们来看一看本课时的第二个问题，为什么用 volatile 标记位的停止方法是错误的？</p>
<h4 data-nodeid="597">错误的停止方法</h4>
<p data-nodeid="598">首先，我们来看几种停止线程的错误方法。比如 stop()，suspend() 和 resume()，这些方法已经被 Java 直接标记为&nbsp;@Deprecated。如果再调用这些方法，IDE 会友好地提示，我们不应该再使用它们了。但为什么它们不能使用了呢？是因为 stop() 会直接把线程停止，这样就没有给线程足够的时间来处理想要在停止前保存数据的逻辑，任务戛然而止，会导致出现数据完整性等问题。</p>
<p data-nodeid="599">而对于 suspend() 和 resume() 而言，它们的问题在于如果线程调用 suspend()，它并不会释放锁，就开始进入休眠，但此时有可能仍持有锁，这样就容易导致死锁问题，因为这把锁在线程被 resume() 之前，是不会被释放的。</p>
<p data-nodeid="600">假设线程 A 调用了 suspend() 方法让线程 B 挂起，线程 B 进入休眠，而线程 B 又刚好持有一把锁，此时假设线程 A 想访问线程 B&nbsp;持有的锁，但由于线程 B 并没有释放锁就进入休眠了，所以对于线程 A&nbsp;而言，此时拿不到锁，也会陷入阻塞，那么线程 A 和线程 B 就都无法继续向下执行。</p>
<p data-nodeid="601">正是因为有这样的风险，所以 suspend() 和 resume() 组合使用的方法也被废弃了。那么接下来我们来看看，为什么用 volatile 标记位的停止方法也是错误的？</p>
<h4 data-nodeid="602">volatile 修饰标记位适用的场景</h4>
<pre class="lang-java" data-nodeid="603"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">VolatileCanStop</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">Runnable</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;canceled&nbsp;=&nbsp;<span class="hljs-keyword">false</span>;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">int</span>&nbsp;num&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(!canceled&nbsp;&amp;&amp;&nbsp;num&nbsp;&lt;=&nbsp;<span class="hljs-number">1000000</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(num&nbsp;%&nbsp;<span class="hljs-number">10</span>&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(num&nbsp;+&nbsp;<span class="hljs-string">"是10的倍数。"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;num++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;VolatileCanStop&nbsp;r&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;VolatileCanStop();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;thread&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(r);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">3000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r.canceled&nbsp;=&nbsp;<span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="604">什么场景下 volatile 修饰标记位可以让线程正常停止呢？如代码所示，声明了一个叫作 VolatileStopThread 的类， 它实现了 Runnable 接口，然后在 run() 中进行 while 循环，在循环体中又进行了两层判断，首先判断&nbsp;canceled&nbsp;变量的值，canceled&nbsp;变量是一个被 volatile 修饰的初始值为 false 的布尔值，当该值变为 true 时，while 跳出循环，while 的第二个判断条件是 num 值小于1000000（一百万），在while 循环体里，只要是 10 的倍数就打印出来，然后 num++。</p>
<p data-nodeid="605">接下来，首先启动线程，然后经过 3 秒钟的时间，把用 volatile 修饰的布尔值的标记位设置成 true，这样，正在运行的线程就会在下一次 while 循环中判断出&nbsp;canceled&nbsp;的值已经变成 true 了，这样就不再满足 while 的判断条件，跳出整个 while 循环，线程就停止了，这种情况是演示 volatile 修饰的标记位可以正常工作的情况，但是如果我们说某个方法是正确的，那么它应该不仅仅是在一种情况下适用，而在其他情况下也应该是适用的。</p>
<h4 data-nodeid="606">volatile 修饰标记位不适用的场景</h4>
<p data-nodeid="607">接下来我们就用一个生产者/消费者模式的案例来演示为什么说 &nbsp;volatile 标记位的停止方法是不完美的。</p>
<pre class="lang-java" data-nodeid="608"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Producer</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">Runnable</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;canceled&nbsp;=&nbsp;<span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;BlockingQueue&nbsp;storage;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">Producer</span><span class="hljs-params">(BlockingQueue&nbsp;storage)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.storage&nbsp;=&nbsp;storage;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">int</span>&nbsp;num&nbsp;=&nbsp;<span class="hljs-number">0</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">try</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(num&nbsp;&lt;=&nbsp;<span class="hljs-number">100000</span>&nbsp;&amp;&amp;&nbsp;!canceled)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(num&nbsp;%&nbsp;<span class="hljs-number">50</span>&nbsp;==&nbsp;<span class="hljs-number">0</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;storage.put(num);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(num&nbsp;+&nbsp;<span class="hljs-string">"是50的倍数,被放到仓库中了。"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;num++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">catch</span>&nbsp;(InterruptedException&nbsp;e)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;e.printStackTrace();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="hljs-keyword">finally</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"生产者结束运行"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="609">首先，声明了一个生产者 Producer，通过 volatile 标记的初始值为 false 的布尔值 canceled 来停止线程。而在 run() 方法中，while 的判断语句是 num 是否小于 100000 及&nbsp;canceled 是否被标记。while 循环体中判断 num 如果是 50 的倍数就放到&nbsp;storage 仓库中，storage 是生产者与消费者之间进行通信的存储器，当 num 大于 100000 或被通知停止时，会跳出 while 循环并执行 finally 语句块，告诉大家“生产者结束运行”。</p>
<pre class="lang-java" data-nodeid="610"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Consumer</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;BlockingQueue&nbsp;storage;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">Consumer</span><span class="hljs-params">(BlockingQueue&nbsp;storage)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.storage&nbsp;=&nbsp;storage;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;<span class="hljs-title">needMoreNums</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(Math.random()&nbsp;&gt;&nbsp;<span class="hljs-number">0.97</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="611">而对于消费者 Consumer，它与生产者共用同一个仓库 storage，并且在方法内通过 needMoreNums() 方法判断是否需要继续使用更多的数字，刚才生产者生产了一些 50 的倍数供消费者使用，消费者是否继续使用数字的判断条件是产生一个随机数并与 0.97 进行比较，大于 0.97&nbsp;就不再继续使用数字。</p>
<pre class="lang-java" data-nodeid="612"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&nbsp;storage&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayBlockingQueue(<span class="hljs-number">8</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Producer&nbsp;producer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Producer(storage);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;producerThread&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(producer);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;producerThread.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">500</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Consumer&nbsp;consumer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Consumer(storage);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(consumer.needMoreNums())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(consumer.storage.take()&nbsp;+&nbsp;<span class="hljs-string">"被消费了"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">100</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(<span class="hljs-string">"消费者不需要更多数据了。"</span>);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//一旦消费不需要更多数据了，我们应该让生产者也停下来，但是实际情况却停不下来</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;producer.canceled&nbsp;=&nbsp;<span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(producer.canceled);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="613">下面来看下 main 函数，首先创建了生产者/消费者共用的仓库&nbsp;BlockingQueue storage，仓库容量是 8，并且建立生产者并将生产者放入线程后启动线程，启动后进行 500 毫秒的休眠，休眠时间保障生产者有足够的时间把仓库塞满，而仓库达到容量后就不会再继续往里塞，这时生产者会阻塞，500 毫秒后消费者也被创建出来，并判断是否需要使用更多的数字，然后每次消费后休眠 100 毫秒，这样的业务逻辑是有可能出现在实际生产中的。</p>
<p data-nodeid="614">当消费者不再需要数据，就会将 canceled 的标记位设置为 true，理论上此时生产者会跳出 while 循环，并打印输出“生产者运行结束”。</p>
<p data-nodeid="615">然而结果却不是我们想象的那样，尽管已经把 canceled 设置成 true，但生产者仍然没有停止，这是因为在这种情况下，生产者在执行 storage.put(num)&nbsp;时发生阻塞，在它被叫醒之前是没有办法进入下一次循环判断&nbsp;canceled&nbsp;的值的，所以在这种情况下用 volatile 是没有办法让生产者停下来的，相反如果用 interrupt 语句来中断，即使生产者处于阻塞状态，仍然能够感受到中断信号，并做响应处理。</p>
<h3 data-nodeid="616">总结</h3>
<p data-nodeid="617">好了，本课时的内容就全部讲完了，我们来总结下学到了什么，首先学习了如何正确停止线程，其次是掌握了为什么说&nbsp;volatile 修饰标记位停止方法是错误的。</p>
<p data-nodeid="618">如果我们在面试中被问到“你知不知道如何正确停止线程”这样的问题，我想你一定可以完美地回答了，首先，从原理上讲应该用 interrupt 来请求中断，而不是强制停止，因为这样可以避免数据错乱，也可以让线程有时间结束收尾工作。</p>
<p data-nodeid="619">如果我们是子方法的编写者，遇到了 interruptedException，应该如何处理呢？</p>
<p data-nodeid="620">我们可以把异常声明在方法中，以便顶层方法可以感知捕获到异常，或者也可以在&nbsp;catch&nbsp;中再次声明中断，这样下次循环也可以感知中断，所以要想正确停止线程就要求我们停止方，被停止方，子方法的编写者相互配合，大家都按照一定的规范来编写代码，就可以正确地停止线程了。</p>
<p data-nodeid="621">最后我们再来看下有哪些方法是不够好的，比如说已经被舍弃的 stop()、suspend() 和 resume()，它们由于有很大的安全风险比如死锁风险而被舍弃，而 volatile 这种方法在某些特殊的情况下，比如线程被长时间阻塞的情况，就无法及时感受中断，所以 volatile 是不够全面的停止线程的方法。</p>

---

### 精选评论

##### **强：
> 1.正确停止线程的方法： thread.interrupt()，通知线程中断； 2.线程内逻辑需配合响应中断：1）正常执行循环中使用 Thread.currentThread().isInterrupted()判断中断标识; 2)若含有sleep()等Waiting操作,会唤醒线程，抛出interruptedException，抛出后中断标识会重置。对于中断异常，要么正确处理，重新设置中断标识；要么在方法上声明抛出异常以便调用方处理。3.为什么用 volatile 标记位的停止方法是错误的例如 生产-消费模式，含有阻塞put操作时，volatile 标记变量改变也无法唤醒阻塞中的生产者线程。4.stop()、suspend() 和 resume()是已过期方法，有很大安全风险，它们强行停止线程，有可能造成线程持有的锁或资源没有释放。

##### **生：
> 用interrupte替代volatile也是不行吧，block阻塞线程，没有地方的notify线程，producer线程也不可能继续运行下去

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 消费者consumer.storage.take()时，内部会触发唤醒。
方法源码：
    public E take() throws InterruptedException {
        final ReentrantLock lock = this.lock;
        lock.lockInterruptibly();
        try {
            while (count == 0)
                notEmpty.await();
            return dequeue();
        } finally {
            lock.unlock();
        }
    }

##### **用户3114：
> 精选留言的前两条矛盾了吧。。既然存在消费者直接结束不唤醒生产者，那么换成interrupt也是无法正常结束的。(在阻塞队列这个案例里，interrupt和vole都无法百分百正常停止线程)。求讲师回复，想知道答案。

##### **飞：
> BlokingQueue的put方法生成数据时，如果队列满的会被阻塞，但是当我们调用take方法时会去消费掉队列中的数据同时也会唤醒阻塞队列的啊，那也会继续去判断canceled是否为true啊

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 存在消费者已运行结束，不执行take方法的可能性。

##### *露：
> 学到了

##### **顺：
> 老师，有个疑问，在生产者的demo中，把while循环的判断条件改为只对canceld进行判断，把对num的判断放到循环体里面，这样改变canceld的值也是可以让线程停止的吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不行的，没有本质区别，canceled值虽然变了，但是生产者无法感知到，它处于阻塞状态。

##### akakour：
> 简单的说，用volatile做线程死循环判断条件的原理是，当volatile变量改变的时候，线程在下一次再入循环体的时候能读到所以跳出循环结束线程，这就有个前提：“能读到”，即线程不能堵塞在循环体内部，本例中，线程就堵塞在循环体内部，从而不能“读到”。所以，最优解是interrupt响应中断指令，如果非要用volatile，就需要考虑清楚循环体是否有堵塞的情况。

##### lemonwan：
> 细细的品读，还是有蛮多收获。

##### *策：
> volatile这个关键字本身特性根本就没有用于线程终止的功能，如果要是通过它来做标记位，只能说是线程自己正常停止

##### **永：
> 这一讲是讲如何正确停止线程的，不知道跟voilate关键字有什么关系，感觉这两个知识点老师讲混了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 使用voilate关键字是一种停止线程的方式，但是这种方式有缺陷。

##### **嵩：
> 学到了！！

##### **禹：
> 老师 有个问题，如果线程中没有没有while(true){} 代码，只是执行一个复杂的逻辑，但是中间可能需要停止线程不再继续执行，该如何操作。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 在适当时机检查中断状态。

##### **才：
> 队列put方法时，线程应该是被阻塞了，这时可能是wait或挂起的状态，然后interrupt线程时，会触发异常。而不是唤醒线程

##### **街牛牛：
> 总结：1.使用interuptor中断线程，线程在阻塞情况下还能感受到中断请求2.调用suspend暂停线程 线程还是会持有锁，等到调用resume方法唤醒后完成线程操作才会释放锁，导致死锁问题

##### **杰：
> 中断是个好东西

##### **燕：
> main方法的while循环中，如果不加sleep，则有很大的概率线程会正常结束。因为消费者消费了数据，生产者立刻生产，而由于程序速度执行很快，有一定概率这时候Cancled已经被执行为true了，那么这时候就能正常结束，打印“生产者运行结束”。如果加了sleep，被消费者消费了的数据，生产者很快又生产完了，这时候生产者进入阻塞状态，无法进入while循环，程序也就不会结束，一直阻塞。

##### *川：
> 老师你好。对于线程的中断有个疑问。为什么线程被调用了interrupt方法以后在进入异常块之前。虚拟机要将中断标志位清除。老师有空的时候能解答一下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 因为通常情况下，一旦检测到中断，就会去执行别的逻辑，代表此次中断已经使用过了，清除后就可以感受到下次中断。

##### **柔：
> Thread.sleep是让哪个线程停止啊 是当前正在运行的线程还是main函数的线程呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是执行Thread.sleep方法的线程，可能是主线程也可能是子线程。

##### **凯：
> 学习心得：1. 通过interrupt打断线程，在线程中通过isinterrupt()来判断是否继续执行业务逻辑还是结束2.当线程block住时，线程还是可以收到来自interrupt的打断信号，如果并抛出interrupt异常我的疑惑：基于上面的2，我知道阻塞的线程会接收到来自interrupt的打断信号，但是如果我们的线程并未抛出interrupt异常并且也没有try/catch，那么此时是否还是阻塞的呢？(在相关的解答中我看到消费者consumer.storage.take()时，内部会触发唤醒，这实际上是take()源码抛出了InterruptedException)，那么这是不是意味着我们去实现某一个操作时每次都要抛出这个异常呢？

##### **豆奶：
> 首先通过 Thread.currentThread().isInterrupt() 判断线程是否被中断 Thread.currentThread().isInterrupted()

