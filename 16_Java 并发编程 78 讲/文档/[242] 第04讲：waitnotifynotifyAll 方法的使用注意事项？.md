<p>本课时我们主要学习 wait/notify/notifyAll&nbsp;方法的使用注意事项。</p>
<p>我们主要从三个问题入手：</p>
<ol>
<li>为什么 wait&nbsp;方法必须在 synchronized&nbsp;保护的同步代码中使用？</li>
<li>为什么 wait/notify/notifyAll 被定义在 Object 类中，而 sleep 定义在 Thread 类中？</li>
<li>wait/notify 和 sleep 方法的异同？</li>
</ol>
<h3>为什么 wait 必须在 synchronized&nbsp;保护的同步代码中使用？</h3>
<p>首先，我们来看第一个问题，为什么 wait&nbsp;方法必须在 synchronized&nbsp;保护的同步代码中使用？</p>
<p>我们先来看看 wait 方法的源码注释是怎么写的。</p>
<p>“wait method should always be used in a loop:</p>
<pre><code data-language="java" class="lang-java">&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;(obj)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(condition&nbsp;does&nbsp;not&nbsp;hold)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;obj.wait();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...&nbsp;<span class="hljs-comment">//&nbsp;Perform&nbsp;action&nbsp;appropriate&nbsp;to&nbsp;condition</span>
}
</code></pre>
<p>This method should only be called by a thread that is the owner of this object's monitor.”</p>
<p>英文部分的意思是说，在使用 wait 方法时，必须把 wait 方法写在 synchronized&nbsp;保护的 while 代码块中，并始终判断执行条件是否满足，如果满足就往下继续执行，如果不满足就执行 wait 方法，而在执行 wait 方法之前，必须先持有对象的 monitor 锁，也就是通常所说的&nbsp;synchronized&nbsp;锁。那么设计成这样有什么好处呢？</p>
<p>我们逆向思考这个问题，如果不要求 wait 方法放在 synchronized&nbsp;保护的同步代码中使用，而是可以随意调用，那么就有可能写出这样的代码。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">BlockingQueue</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;Queue&lt;String&gt;&nbsp;buffer&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LinkedList&lt;String&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">give</span><span class="hljs-params">(String&nbsp;data)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buffer.add(data);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notify();&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Since&nbsp;someone&nbsp;may&nbsp;be&nbsp;waiting&nbsp;in&nbsp;take</span>
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">take</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(buffer.isEmpty())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wait();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;buffer.remove();
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>在代码中可以看到有两个方法，give 方法负责往 buffer 中添加数据，添加完之后执行 notify&nbsp;方法来唤醒之前等待的线程，而&nbsp;take 方法负责检查整个 buffer 是否为空，如果为空就进入等待，如果不为空就取出一个数据，这是典型的生产者消费者的思想。</p>
<p>但是这段代码并没有受 synchronized&nbsp;保护，于是便有可能发生以下场景：</p>
<ol>
<li>首先，消费者线程调用 take 方法并判断 buffer.isEmpty&nbsp;方法是否返回&nbsp;true，若为 true&nbsp;代表buffer是空的，则线程希望进入等待，但是在线程调用 wait 方法之前，就被调度器暂停了，所以此时还没来得及执行 wait 方法。</li>
<li>此时生产者开始运行，执行了整个 give 方法，它往 buffer 中添加了数据，并执行了 notify&nbsp;方法，但&nbsp;notify&nbsp;并没有任何效果，因为消费者线程的&nbsp;wait&nbsp;方法没来得及执行，所以没有线程在等待被唤醒。</li>
<li>此时，刚才被调度器暂停的消费者线程回来继续执行&nbsp;wait&nbsp;方法并进入了等待。</li>
</ol>
<p>虽然刚才消费者判断了&nbsp;buffer.isEmpty 条件，但真正执行 wait&nbsp;方法时，之前的 buffer.isEmpty 的结果已经过期了，不再符合最新的场景了，因为这里的“判断-执行”不是一个原子操作，它在中间被打断了，是线程不安全的。</p>
<p>假设这时没有更多的生产者进行生产，消费者便有可能陷入无穷无尽的等待，因为它错过了刚才 give 方法内的 notify&nbsp;的唤醒。</p>
<p>我们看到正是因为 wait&nbsp;方法所在的 take 方法没有被 synchronized&nbsp;保护，所以它的 while 判断和 wait 方法无法构成原子操作，那么此时整个程序就很容易出错。</p>
<p>我们把代码改写成源码注释所要求的被&nbsp;synchronized&nbsp;保护的同步代码块的形式，代码如下。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">give</span><span class="hljs-params">(String&nbsp;data)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;(<span class="hljs-keyword">this</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buffer.add(data);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;notify();
&nbsp;&nbsp;}
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;String&nbsp;<span class="hljs-title">take</span><span class="hljs-params">()</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;(<span class="hljs-keyword">this</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">while</span>&nbsp;(buffer.isEmpty())&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;wait();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;buffer.remove();
&nbsp;&nbsp;}
}
</code></pre>
<p>这样就可以确保 notify&nbsp;方法永远不会在&nbsp;buffer.isEmpty 和 wait 方法之间被调用，提升了程序的安全性。</p>
<p>另外，wait 方法会释放 monitor 锁，这也要求我们必须首先进入到 synchronized 内持有这把锁。</p>
<p>这里还存在一个“虚假唤醒”（spurious wakeup）的问题，线程可能在既没有被notify/notifyAll，也没有被中断或者超时的情况下被唤醒，这种唤醒是我们不希望看到的。虽然在实际生产中，虚假唤醒发生的概率很小，但是程序依然需要保证在发生虚假唤醒的时候的正确性，所以就需要采用while循环的结构。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">while</span>&nbsp;(condition&nbsp;does&nbsp;not&nbsp;hold)
&nbsp;&nbsp;&nbsp;&nbsp;obj.wait();
</code></pre>
<p>这样即便被虚假唤醒了，也会再次检查while里面的条件，如果不满足条件，就会继续wait，也就消除了虚假唤醒的风险。</p>
<h3>为什么 wait/notify/notifyAll 被定义在 Object 类中，而 sleep 定义在 Thread 类中？</h3>
<p>我们来看第二个问题，为什么 wait/notify/notifyAll&nbsp;方法被定义在 Object 类中？而 sleep&nbsp;方法定义在 Thread 类中？主要有两点原因：</p>
<ol>
<li>因为 Java 中每个对象都有一把称之为 monitor 监视器的锁，由于每个对象都可以上锁，这就要求在对象头中有一个用来保存锁信息的位置。这个锁是对象级别的，而非线程级别的，wait/notify/notifyAll 也都是锁级别的操作，它们的锁属于对象，所以把它们定义在 Object 类中是最合适，因为 Object 类是所有对象的父类。</li>
<li>因为如果把 wait/notify/notifyAll&nbsp;方法定义在 Thread 类中，会带来很大的局限性，比如一个线程可能持有多把锁，以便实现相互配合的复杂逻辑，假设此时 wait&nbsp;方法定义在 Thread 类中，如何实现让一个线程持有多把锁呢？又如何明确线程等待的是哪把锁呢？既然我们是让当前线程去等待某个对象的锁，自然应该通过操作对象来实现，而不是操作线程。</li>
</ol>
<h3>wait/notify 和 sleep 方法的异同？</h3>
<p>第三个问题是对比 wait/notify 和 sleep&nbsp;方法的异同，主要对比 wait 和 sleep 方法，我们先说相同点：</p>
<ol>
<li>它们都可以让线程阻塞。</li>
<li>它们都可以响应&nbsp;interrupt&nbsp;中断：在等待的过程中如果收到中断信号，都可以进行响应，并抛出 InterruptedException 异常。</li>
</ol>
<p>但是它们也有很多的不同点：</p>
<ol>
<li>wait 方法必须在 synchronized&nbsp;保护的代码中使用，而 sleep&nbsp;方法并没有这个要求。</li>
<li>在同步代码中执行 sleep 方法时，并不会释放 monitor 锁，但执行 wait 方法时会主动释放 monitor 锁。</li>
<li>sleep 方法中会要求必须定义一个时间，时间到期后会主动恢复，而对于没有参数的&nbsp;wait 方法而言，意味着永久等待，直到被中断或被唤醒才能恢复，它并不会主动恢复。</li>
<li>wait/notify 是 Object 类的方法，而 sleep 是 Thread 类的方法。</li>
</ol>
<p>以上就是关于 wait/notify 与 sleep 的异同点。</p>
<p>好了，本课时的内容就全部讲完了，下一课时我将讲解“有哪几种实现生产者-消费者模式的方法？<span class="size" style="font-size:12pt">”记得按时来听课啊，下一课时见。</span></p>

---

### 精选评论

##### **生：
> 干货满满，一言千金，绝无废话

##### **平：
> 亲测被synchronized修饰的代码块执行过程中也会被调度，中间可能让出cpu，只不过不存在线程安全问题

##### **谦：
> <div>你好,我有一个比较混淆的点是针对原子性和原子操作的?</div>原子性: 是描述动作要么被完整执行 要么都不执行的,完整执行并不意味这一口气做完,仍旧受线程调度机制影响<div>原子操作: 不会受线程调度机制而中断,也就是不会出现线程上下文切换</div><div>这两个描述是否准确?</div><div>如果说synchronized不受线程调度机制影响的话,那单核处理器上一旦加synchronized,同时同步块内执行死循环,由另一个线程更新一个volatile变量达到一定的值再去break死循环的话,是不是意味死循环不会退出</div><div><br></div><div><br></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，即便是单核处理器，synchronized执行期间也是可以进行线程切换的，受到线程调度机制的控制。

##### **栓：
> 很棒很棒！！

##### *启：
> 今天总算继续学习了，很棒

##### *木：
> 这次 我想要的知识 牛逼

##### **狗：
> 看完本届课程还有个疑问想问下老师，调用了sleep()方法的线程如何唤醒呢？是否只能等待倒计时时间结束自动恢复，有没有其他方法可以打断休眠？😨

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以使用中断interrupt来提前唤醒线程。

##### **生：
> 举的那个例子在没有被synchronized保护的场景，那个take方法，在调用wait()方法之前被调度器暂停了，没有调用wait()方法的情况下执行的，那如果没有被调度器暂停呢？会不会就正常运行了一段时间呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是可能正常运行一段时间的。

##### **滔：
> 1.为什么 wait 方法必须在 synchronized 保护的同步代码中使用？ 1）拿生产者消费者举例，如果消费者未加 synchronized 保护，while 判断和 wait 方法无法构成原子操作，可能生产者先notify，消费者再wait，后续如无生产者再notify，消费者便有可能陷入无穷无尽的等待。2）程序需采用while循环来应对“虚假唤醒”（非notify/notifyAll，中断或超时的唤醒），即便被虚假唤醒了，也会再次检查while里面的条件。2.为什么 wait/notify/notifyAll 被定义在 Object 类中，而 sleep 定义在 Thread 类中？1）wait/notify/notifyAll 也都是锁级别的操作，它们的锁属于对象，所以把它们定义在 Object 类中是最合适，因为 Object 类是所有对象的父类。2）因为可能在Thread中持有多个对象锁，做复杂的业务操作，如果定义wait定义在Thread中，那么是不满足的。3.wait/notify 和 sleep 方法的异同？相同点：1）都可以使线程阻塞；2）都可以响应interrupt 中断，并抛出InterruptedException 异常不同点：1）wait必须写在 synchronized 保护的代码中，sleep不用；2）在同步代码中，执行sleep方法时不会释放monitor锁，执行wait方法时会自动释放monitor锁；3）sleep方法需要设置到期时间，到期自动恢复，wait方法不能设置时间，不会主动恢复，除非被中断或者唤醒；4）wait/notify 是属于Object类的，sleep是属于Thread类的；

##### **0686：
> ”但是在线程调用 wait 方法之前，就被调度器暂停了“ 这个调度器是CPU吗还是？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 线程调度，由操作系统负责。

##### **松 2年半java求职：
> 调度器暂停是指CPU调度器把 ，学到了

##### **光：
> 虚假唤醒应该是当条件满足被唤醒后会重新竞争mintor锁，等竞争到的时候，条件可能已经不满足了。所以在while循环里判断是吧？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 虚假唤醒特指没有实际唤醒动作，但是却被唤醒的情况。

##### **狗：
> 老师，这节课程，有个很大的疑惑：说到wait/notify 和 sleep 方法的异同？这个问题时，有这么一句话：“但执行 wait 方法时会主动释放 monitor 锁”,但是下一句话又说：“而对于没有参数的 wait 方法而言，意味着永久等待，直到被中断或被唤醒才能恢复，它并不会主动恢复。”那么这个wait()方法到底是主动还是不会主动释放锁呢？😰😅😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 会主动释放锁。

