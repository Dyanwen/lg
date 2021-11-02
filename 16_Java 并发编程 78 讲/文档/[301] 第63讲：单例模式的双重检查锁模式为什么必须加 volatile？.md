<p data-nodeid="31503" class="">本课时我们主要讲解单例模式的双重检查锁模式为什么必须加 volatile？</p>
<h3 data-nodeid="31504">什么是单例模式</h3>
<p data-nodeid="31505">单例模式指的是，保证一个类只有一个实例，并且提供一个可以全局访问的入口。</p>
<h4 data-nodeid="31506">为什么需要使用单例模式</h4>
<p data-nodeid="31507">那么我们为什么需要单例呢？其中**一个理由，那就是为了节省内存、节省计算。**因为在很多情况下，我们只需要一个实例就够了，如果出现更多的实例，反而纯属浪费。</p>
<p data-nodeid="31508">下面我们举一个例子来说明这个情况，以一个初始化比较耗时的类来说，代码如下所示：</p>
<pre class="lang-java" data-nodeid="31509"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">ExpensiveResource</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">ExpensiveResource</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;field1&nbsp;=&nbsp;<span class="hljs-comment">//&nbsp;查询数据库</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;field2&nbsp;=&nbsp;<span class="hljs-comment">//&nbsp;然后对查到的数据做大量计算</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;field3&nbsp;=&nbsp;<span class="hljs-comment">//&nbsp;加密、压缩等耗时操作</span>
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="31510">这个类在构造的时候，需要查询数据库并对查到的数据做大量计算，所以在第一次构造时，我们花了很多时间来初始化这个对象。但是假设数据库里的数据是不变的，我们就可以把这个对象保存在内存中，那么以后开发的时候就可以直接用这同一个实例了，不需要再次构建新实例。如果每次都重新生成新的实例，则会造成更多的浪费，实在没有必要。</p>
<p data-nodeid="31511">接下来看看需要单例的第二个理由，那就是为了保证结果的正确。**比如我们需要一个全局的计数器，用来统计人数，如果有多个实例，反而会造成混乱。</p>
<p data-nodeid="31512">另外呢，就是为了方便管理。**很多工具类，我们只需要一个实例，那么我们通过统一的入口，比如通过&nbsp;getInstance 方法去获取这个单例是很方便的，太多实例不但没有帮助，反而会让人眼花缭乱。</p>
<p data-nodeid="31513">一般单例模式的类结构如下图所示：有一个私有的 Singleton 类型的 singleton 对象；同时构造方法也是私有的，为了防止他人调用构造函数来生成实例；另外还会有一个 public 的 getInstance 方法，可通过这个方法获取到单例。</p>
<p data-nodeid="31514"><img src="https://s0.lgstatic.com/i/image3/M01/05/B6/Ciqah16BpV-AG9iPAAAf42nvy5s798.png" alt="" data-nodeid="31567"></p>
<h4 data-nodeid="31515">双重检查锁模式的写法</h4>
<p data-nodeid="31516">单例模式有多种写法，我们重点介绍一下和 volatile 强相关的双重检查锁模式的写法，代码如下所示：</p>
<pre class="lang-java" data-nodeid="31517"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Singleton</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">volatile</span>&nbsp;Singleton&nbsp;singleton;

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-title">Singleton</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;}

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;Singleton&nbsp;<span class="hljs-title">getInstance</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(singleton&nbsp;==&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">synchronized</span>&nbsp;(Singleton.class)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(singleton&nbsp;==&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;singleton&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;Singleton();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;singleton;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="31518">在这里我将重点讲解&nbsp;getInstance&nbsp;方法，方法中首先进行了一次 if (singleton == null) 的检查，然后是 synchronized 同步块，然后又是一次 if (singleton == null) 的检查，最后是 singleton&nbsp;= new Singleton() 来生成实例。</p>
<p data-nodeid="31519">我们进行了两次 if (singleton == null) 检查，这就是“双重检查锁”这个名字的由来。这种写法是可以保证线程安全的，假设有两个线程同时到达 synchronized 语句块，那么实例化代码只会由其中先抢到锁的线程执行一次，而后抢到锁的线程会在第二个 if 判断中发现 singleton 不为 null，所以跳过创建实例的语句。再后面的其他线程再来调用 getInstance 方法时，只需判断第一次的&nbsp;if (singleton == null) ，然后会跳过整个 if 块，直接 return 实例化后的对象。</p>
<p data-nodeid="31520">这种写法的优点是不仅线程安全，而且延迟加载、效率也更高。</p>
<p data-nodeid="31521"><strong data-nodeid="31576">讲到这里就涉及到了一个常见的问题，面试官可能会问你，“为什么要 double-check？去掉任何一次的 check 行不行？”</strong></p>
<p data-nodeid="31522">我们先来看第二次的 check，这时你需要考虑这样一种情况，有两个线程同时调用&nbsp;getInstance&nbsp;方法，由于&nbsp;singleton&nbsp;是空的&nbsp;，因此两个线程都可以通过第一重的 if 判断；然后由于锁机制的存在，会有一个线程先进入同步语句，并进入第二重 if 判断 ，而另外的一个线程就会在外面等待。</p>
<p data-nodeid="31523">不过，当第一个线程执行完&nbsp;new Singleton()&nbsp;语句后，就会退出 synchronized 保护的区域，这时如果没有第二重&nbsp;if (singleton == null) 判断的话，那么第二个线程也会创建一个实例，此时就破坏了单例，这肯定是不行的。</p>
<p data-nodeid="31524">而对于第一个 check 而言，如果去掉它，那么所有线程都会串行执行，效率低下，所以两个 check 都是需要保留的。</p>
<h4 data-nodeid="31525">在双重检查锁模式中为什么需要使用 volatile 关键字</h4>
<p data-nodeid="31526">相信细心的你可能看到了，我们在双重检查锁模式中，给 singleton 这个对象加了 volatile 关键字，那**为什么要用 volatile 呢？**主要就在于 singleton = new Singleton() ，它并非是一个原子操作，事实上，在&nbsp;JVM&nbsp;中上述语句至少做了以下这&nbsp;3&nbsp;件事：</p>
<p data-nodeid="31527"><img src="https://s0.lgstatic.com/i/image3/M01/7E/CC/Cgq2xl6BpWCAMBaVAACFIdffjfM852.png" alt="" data-nodeid="31589"></p>
<ul data-nodeid="31528">
<li data-nodeid="31529">
<p data-nodeid="31530">第一步是给&nbsp;singleton&nbsp;分配内存空间；</p>
</li>
<li data-nodeid="31531">
<p data-nodeid="31532">然后第二步开始调用&nbsp;Singleton&nbsp;的构造函数等，来初始化 singleton；</p>
</li>
<li data-nodeid="31533">
<p data-nodeid="31534">最后第三步，将 singleton 对象指向分配的内存空间（执行完这步&nbsp;singleton&nbsp;就不是&nbsp;null&nbsp;了）。</p>
</li>
</ul>
<p data-nodeid="31535">这里需要留意一下 1-2-3 的顺序，因为存在指令重排序的优化，也就是说第2 步和第 3 步的顺序是不能保证的，最终的执行顺序，可能是 1-2-3，也有可能是 1-3-2。</p>
<p data-nodeid="31536">如果是 1-3-2，那么在第 3 步执行完以后，singleton&nbsp;就不是&nbsp;null&nbsp;了，可是这时第 2 步并没有执行，singleton 对象未完成初始化，它的属性的值可能不是我们所预期的值。假设此时线程 2 进入 getInstance 方法，由于&nbsp;singleton&nbsp;已经不是&nbsp;null&nbsp;了，所以会通过第一重检查并直接返回，但其实这时的 singleton 并没有完成初始化，所以使用这个实例的时候会报错，详细流程如下图所示：</p>
<p data-nodeid="31537"><img src="https://s0.lgstatic.com/i/image3/M01/7E/CC/Cgq2xl6BpWCAB6QQAAEKacFd0CE542.png" alt="" data-nodeid="31596"></p>
<p data-nodeid="31538">线程 1 首先执行新建实例的第一步，也就是分配单例对象的内存空间，由于线程 1 被重排序，所以执行了新建实例的第三步，也就是把 singleton 指向之前分配出来的内存地址，在这第三步执行之后，singleton 对象便不再是 null。</p>
<p data-nodeid="31539">这时线程 2 进入&nbsp;getInstance&nbsp;方法，判断 singleton 对象不是 null，紧接着线程 2 就返回 singleton 对象并使用，由于没有初始化，所以报错了。最后，线程 1 “姗姗来迟”，才开始执行新建实例的第二步——初始化对象，可是这时的初始化已经晚了，因为前面已经报错了。</p>
<p data-nodeid="31540">使用了 volatile 之后，相当于是表明了该字段的更新可能是在其他线程中发生的，因此应确保在读取另一个线程写入的值时，可以顺利执行接下来所需的操作。在 JDK 5 以及后续版本所使用的 JMM 中，在使用了 volatile 后，会一定程度禁止相关语句的重排序，从而避免了上述由于重排序所导致的读取到不完整对象的问题的发生。</p>
<p data-nodeid="31541">到这里关于“为什么要用 volatile” 的问题就讲完了，使用 volatile 的意义主要在于它可以防止避免拿到没完成初始化的对象，从而保证了线程安全。</p>
<h3 data-nodeid="31542">总结</h3>
<p data-nodeid="31807">在本课时中我们首先介绍了什么是单例模式，以及为什么需要使用单例模式，然后介绍了双重检查锁模式这种写法，以及面对这种写法时为什么需要 double-check，为什么需要用 volatile？最主要的是为了保证线程安全。</p>
<blockquote data-nodeid="34414">
<p data-nodeid="34415" class="te-preview-highlight">参考：<br>
小宝马的爸爸 - 梦想的家园《单例模式（Singleton）》：<a href="https://www.cnblogs.com/BoyXiao/archive/2010/05/07/1729376.html" data-nodeid="34421">https://www.cnblogs.com/BoyXiao/archive/2010/05/07/1729376.html</a><br>
Jark's Blog《如何正确地写出单例模式》：<a href="http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/" data-nodeid="34428">http://wuchong.me/blog/2014/08/28/how-to-correctly-write-singleton-pattern/</a><br>
Hollis Chuang《为什么我墙裂建议大家使用枚举来实现单例》：<a href="https://www.hollischuang.com/archives/2498" data-nodeid="34433">https://www.hollischuang.com/archives/2498</a><br>
Hollis Chuang《深度分析Java的枚举类型—-枚举的线程安全性及序列化问题》：<a href="https://www.hollischuang.com/archives/197" data-nodeid="34438">https://www.hollischuang.com/archives/197</a></p>
</blockquote>

---

### 精选评论

##### happyboy：
> 老师，我这里有一点点不明白，就是`<span style="font-size: 16.0125px;">&nbsp;singleton = new Singleton()</span>`会拆分成3条语句，但他毕竟在synchronized同步代码块中，为什么这三条语句不是全部执行完毕之后就会退出，而是只执行了前两步就退出，另一个线程就进来了呢，那synchronized的意义在哪里呢？是不是我对synchronized的理解不对呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你的问题很好哈，注意时机，第一个线程执行了前两步后，并没有退出synchronized，就在此时，第二个线程进来了，而不是第一个线程已经退出了synchronized代码块，就会发生课程里讲的线程安全问题。

##### **峰：
> 我理解用Volatile修饰是为了保证变量的可见性吧，以保证后续的线程能读取到最新的变量值

##### **林：
> 老师，synchronized同步块可以禁止指令重排序，是指同步块的内容相对于同步块之外的代码可以保证不会重排序，但其实同步块里面还是会重排序嘛

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，是会的。

##### *达：
> Sync代码块不是也能禁止重排序吗 ？为什么代码块中的实例化对象居然还会发生重排序问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 重排序发生在：第二个线程并没有进入到锁保护的同步块，只是进入了第一层if判断。第一个线程退出synchronized之前，无法保证可见性。

##### *杰：
> 老师，第二步调用构造函数初始化single，是不是可以理解为对分配的内存空间进行赋值操作？让内存空间有数据

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以滴

##### **明：
> 老师讲的好！

##### *池：
> 老师，synchronized 可以保证可见性，singleton = new Singleton() 执行对于第二个线程应该是可见的，第二个线程应该能看到new Singleton()的对象。不知道我理解哪里出现问题了，请老师指教

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第二个线程并没有进入到锁保护的同步块，只是进入了第一层if判断。第一个线程退出synchronized之前，无法保证可见性。

##### **男：
> 如果第一个线程正在执行syn块内，此时另一个线程刚到第一个if判断，是有可见性问题，那等第一个线程执行完成，第二个线程进入第二个if判断，会有问题吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第一个线程如果在syn块内，就不能保证其他线程没有可见性问题。执行完后，可以保证可见性。

##### *豪：
> 老师，new Singleton()发生在synchronized中，为什么第一个线程三条指令还没执行完第二个线程就能进来？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第二个线程并没有进入synchronized，第二个线程在第一层if时会判断!=null然后跳过synchronized。

