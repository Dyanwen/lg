<p>在本课时我们主要学习哪些场景需要额外注意线程安全问题，在这里总结了四种场景。</p>
<h3>访问共享变量或资源</h3>
<p>第一种场景是访问共享变量或共享资源的时候，典型的场景有访问共享对象的属性，访问 static 静态变量，访问共享的缓存，等等。因为这些信息不仅会被一个线程访问到，还有可能被多个线程同时访问，那么就有可能在并发读写的情况下发生线程安全问题。比如我们上一课时讲过的多线程同时 i++ 的例子：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">/**
&nbsp;*&nbsp;描述：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;共享的变量或资源带来的线程安全问题
&nbsp;*/</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">ThreadNotSafe1</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;i;

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Runnable&nbsp;r&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Runnable()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;j&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;j&nbsp;&lt;&nbsp;<span class="hljs-number">10000</span>;&nbsp;j++)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;i++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;thread1&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(r);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;thread2&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Thread(r);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread1.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread2.start();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread1.join();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;thread2.join();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(i);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p>如代码所示，两个线程同时对 i 进行 i++ 操作，最后的输出可能是 15875 等小于20000的数，而不是我们期待的20000，这便是非常典型的共享变量带来的线程安全问题。</p>
<h3>依赖时序的操作</h3>
<p>第二个需要我们注意的场景是依赖时序的操作，如果我们操作的正确性是依赖时序的，而在多线程的情况下又不能保障执行的顺序和我们预想的一致，这个时候就会发生线程安全问题，如下面的代码所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">if</span>&nbsp;(map.containsKey(key))&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;map.remove(obj)
}
</code></pre>
<p>代码中首先检查 map 中有没有 key 对应的元素，如果有则继续执行 remove 操作。此时，这个组合操作就是危险的，因为它是先检查后操作，而执行过程中可能会被打断。如果此时有两个线程同时进入 if() 语句，然后它们都检查到存在 key 对应的元素，于是都希望执行下面的 remove 操作，随后一个线程率先把 obj 给删除了，而另外一个线程它刚已经检查过存在 key 对应的元素，if 条件成立，所以它也会继续执行删除 obj 的操作，但实际上，集合中的 obj 已经被前面的线程删除了，这种情况下就可能导致线程安全问题。</p>
<p>类似的情况还有很多，比如我们先检查 x=1，如果 x=1 就修改 x 的值，代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">if</span>&nbsp;(x&nbsp;==&nbsp;<span class="hljs-number">1</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;x&nbsp;=&nbsp;<span class="hljs-number">7</span>&nbsp;*&nbsp;x;
}
</code></pre>
<p>这样类似的场景都是同样的道理，“检查与执行”并非原子性操作，在中间可能被打断，而检查之后的结果也可能在执行时已经过期、无效，换句话说，获得正确结果取决于幸运的时序。这种情况下，我们就需要对它进行加锁等保护措施来保障操作的原子性。</p>
<h3>不同数据之间存在绑定关系</h3>
<p>第三种需要我们注意的线程安全场景是不同数据之间存在相互绑定关系的情况。有时候，我们的不同数据之间是成组出现的，存在着相互对应或绑定的关系，最典型的就是 IP 和端口号。有时候我们更换了 IP，往往需要同时更换端口号，如果没有把这两个操作绑定在一起，就有可能出现单独更换了 IP 或端口号的情况，而此时信息如果已经对外发布，信息获取方就有可能获取一个错误的 IP 与端口绑定情况，这时就发生了线程安全问题。在这种情况下，我们也同样需要保障操作的原子性。</p>
<h3>对方没有声明自己是线程安全的</h3>
<p>第四种值得注意的场景是在我们使用其他类时，如果对方没有声明自己是线程安全的，那么这种情况下对其他类进行多线程的并发操作，就有可能会发生线程安全问题。举个例子，比如说我们定义了 ArrayList，它本身并不是线程安全的，如果此时多个线程同时对 ArrayList 进行并发读/写，那么就有可能会产生线程安全问题，造成数据出错，而这个责任并不在 ArrayList，因为它本身并不是并发安全的，正如源码注释所写的：</p>
<pre><code data-language="java" class="lang-java">Note&nbsp;that&nbsp;<span class="hljs-keyword">this</span>&nbsp;implementation&nbsp;is&nbsp;not&nbsp;<span class="hljs-keyword">synchronized</span>.&nbsp;If&nbsp;multiple&nbsp;threads
access&nbsp;an&nbsp;ArrayList&nbsp;instance&nbsp;concurrently,&nbsp;and&nbsp;at&nbsp;least&nbsp;one&nbsp;of&nbsp;the&nbsp;threads
modifies&nbsp;the&nbsp;list&nbsp;structurally,&nbsp;it&nbsp;must&nbsp;be&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;externally.
</code></pre>
<p>这段话的意思是说，如果我们把 ArrayList 用在了多线程的场景，需要在外部手动用 synchronized 等方式保证并发安全。</p>
<p>所以 ArrayList 默认不适合并发读写，是我们错误地使用了它，导致了线程安全问题。所以，我们在使用其他类时如果会涉及并发场景，那么一定要首先确认清楚，对方是否支持并发操作，以上就是四种需要我们额外注意线程安全问题的场景，分别是访问共享变量或资源，依赖时序的操作，不同数据之间存在绑定关系，以及对方没有声明自己是线程安全的。</p>

---

### 精选评论

##### **杰：
> 老师课课精华

##### *展：
> 我理解，从本质上说，可能发生线程安全的场景，就是使用了某个共享的对象或变量，即场景一。场景二和三都是对共享资源的不安全操作引起的

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油~

##### *豪：
> 线程安全可能会出现场景，①共享变量或者共享资源。②依赖时序的操作。 ③不同数据之间存在绑定关系。 ④没有声明自己是线程安全的工具类。坚持。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 同学加油~

##### *桃：
> 共享资源变量：需要保证可见性和原子性。对共享资源的操作要多线成之间可见，修改操作必须是原子操作；依赖时序：则需要保证有序性；绑定关系：应该属性原子性问题，将存在绑定关系的变量的修改操作变为原子操作，可以通过 synchronized 或者加锁的方式来实现；将不是线程安全的类变为线程安全：对变量的读取和修改都需要加互斥锁，才能够保证其线程安全，但是这种锁的粗粒度都比较大，性能应该比较低

##### **鑫：
> 如果采用的数据结构本身不支持并发的话，但又因为特殊需要必须使用特定数据结构，怎么去设计方案优化并发呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以使用加锁等其他方法辅助。

##### **滔：
> 1)  访问共享变量或缓存时：如两个线程同时对全局变量进行i++操作;2）依赖时序的操作：如先检查变量是否满足条件再操作;3）不同数据的绑定关系：如IP和端口号，需要绑定在一起成为一个原子操作;4)  访问了线程不安全的对象：如ArrayList。

##### 杨：
> 访问共享变量或资源，依赖时序的操作，不同数据之间存在绑定关系，以及对方没有声明自己是线程安全的

##### *博：
> 依赖时序的操作单例的双重校验 解决的就是这个问题

##### *桃：
> 共享变量资源，依赖时序的操作，不同数据之间的绑定关系，对方没有声明自己是现场安全的

##### **墨影：
> 1. 访问共享变量或资源 - 可见性2. 依赖时序的操作 - 有序性3. 不同数据之间存在绑定关系 - 原子性

