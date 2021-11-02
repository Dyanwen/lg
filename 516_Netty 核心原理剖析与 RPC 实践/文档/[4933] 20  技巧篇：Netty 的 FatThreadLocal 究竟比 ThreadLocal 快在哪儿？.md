<p data-nodeid="957" class="">在前面几篇源码解析的课程中，我们都有在源码中发现 FastThreadLocal 的身影。顾名思义，Netty 作为高性能的网络通信框架，FastThreadLocal 是比 JDK 自身的 ThreadLocal 性能更高的通信框架。FastThreadLocal 到底比 ThreadLocal 快在哪里呢？这节课我们就一起来探索 FastThreadLocal 高性能的奥秘。</p>
<blockquote data-nodeid="958">
<p data-nodeid="959">说明：本文参考的 Netty 源码版本为 4.1.42.Final。</p>
</blockquote>
<h3 data-nodeid="960">JDK ThreadLocal 基本原理</h3>
<p data-nodeid="961">JDK ThreadLocal 不仅是高频的面试知识点，而且在日常工作中也是常用一种工具，所以首先我们先学习下 Java 原生的 ThreadLocal 的实现原理，可以帮助我们更好地对比和理解 Netty 的 FastThreadLocal。</p>
<p data-nodeid="962">如果你需要变量在多线程之间隔离，或者在同线程内的类和方法中共享，那么 ThreadLocal 大显身手的时候就到了。ThreadLocal 可以理解为线程本地变量，它是 Java 并发编程中非常重要的一个类。ThreadLocal 为变量在每个线程中都创建了一个副本，该副本只能被当前线程访问，多线程之间是隔离的，变量不能在多线程之间共享。这样每个线程修改变量副本时，不会对其他线程产生影响。</p>
<p data-nodeid="963">接下来我们通过一个例子看下 ThreadLocal 如何使用：</p>
<pre class="lang-java" data-nodeid="964"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ThreadLocalTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;String&gt; THREAD_NAME_LOCAL = ThreadLocal.withInitial(() -&gt; Thread.currentThread().getName());
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;TradeOrder&gt; TRADE_THREAD_LOCAL = <span class="hljs-keyword">new</span> ThreadLocal&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">2</span>; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> tradeId = i;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> Thread(() -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TradeOrder tradeOrder = <span class="hljs-keyword">new</span> TradeOrder(tradeId, tradeId % <span class="hljs-number">2</span> == <span class="hljs-number">0</span> ? <span class="hljs-string">"已支付"</span> : <span class="hljs-string">"未支付"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TRADE_THREAD_LOCAL.set(tradeOrder);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"threadName: "</span> + THREAD_NAME_LOCAL.get());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"tradeOrder info："</span> + TRADE_THREAD_LOCAL.get());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }, <span class="hljs-string">"thread-"</span> + i).start();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TradeOrder</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> id;
&nbsp; &nbsp; &nbsp; &nbsp; String status;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">TradeOrder</span><span class="hljs-params">(<span class="hljs-keyword">int</span> id, String status)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.id = id;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.status = status;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"id="</span> + id + <span class="hljs-string">", status="</span> + status;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="965">在上述示例中，构造了 THREAD_NAME_LOCAL 和 TRADE_THREAD_LOCAL 两个 ThreadLocal 变量，分别用于记录当前线程名称和订单交易信息。ThreadLocal 是可以支持泛型的，THREAD_NAME_LOCAL 和 TRADE_THREAD_LOCAL 存放 String 类型和 TradeOrder 对象类型的数据，你可以通过 set()/get() 方法设置和读取 ThreadLocal 实例。一起看下示例代码的运行结果：</p>
<pre class="lang-java" data-nodeid="966"><code data-language="java">threadName: thread-<span class="hljs-number">0</span>
threadName: thread-<span class="hljs-number">1</span>
tradeOrder info：id=<span class="hljs-number">1</span>, status=未支付
tradeOrder info：id=<span class="hljs-number">0</span>, status=已支付
</code></pre>
<p data-nodeid="967">可以看出 thread-1 和 thread-2 虽然操作的是同一个 ThreadLocal 对象，但是它们取到了不同的线程名称和订单交易信息。那么一个线程内如何存在多个 ThreadLocal 对象，每个 ThreadLocal 对象是如何存储和检索的呢？</p>
<p data-nodeid="968">接下来我们看看 ThreadLocal 的实现原理。既然多线程访问 ThreadLocal 变量时都会有自己独立的实例副本，那么很容易想到的方案就是在 ThreadLocal 中维护一个 Map，记录线程与实例之间的映射关系。当新增线程和销毁线程时都需要更新 Map 中的映射关系，因为会存在多线程并发修改，所以需要保证 Map 是线程安全的。那么 JDK 的 ThreadLocal 是这么实现的吗？答案是 NO。因为在高并发的场景并发修改 Map 需要加锁，势必会降低性能。JDK 为了避免加锁，采用了相反的设计思路。以 Thread 入手，在 Thread 中维护一个 Map，记录 ThreadLocal 与实例之间的映射关系，这样在同一个线程内，Map 就不需要加锁了。示例代码中线程 Thread 和 ThreadLocal 的关系可以用以下这幅图表示。</p>
<p data-nodeid="969"><img src="https://s0.lgstatic.com/i/image2/M01/04/2D/CgpVE1_qwuqAN-08AAkfe67UOIA904.png" alt="Drawing 0.png" data-nodeid="1092"></p>
<p data-nodeid="970">那么在 Thread 内部，维护映射关系的 Map 是如何实现的呢？从源码中可以发现 Thread 使用的是 ThreadLocal 的内部类 ThreadLocalMap，所以 Thread、ThreadLocal 和 ThreadLocalMap 之间的关系可以用下图表示：</p>
<p data-nodeid="971"><img src="https://s0.lgstatic.com/i/image/M00/8C/49/Ciqc1F_qwvCAauqfAAI07PytbZY507.png" alt="Drawing 1.png" data-nodeid="1096"></p>
<p data-nodeid="972">为了更加深入理解 ThreadLocal，了解 ThreadLocalMap 的内部实现是非常有必要的。ThreadLocalMap 其实与 HashMap 的数据结构类似，但是 ThreadLocalMap 不具备通用性，它是为 ThreadLocal 量身定制的。</p>
<p data-nodeid="973">ThreadLocalMap 是一种使用线性探测法实现的哈希表，底层采用数组存储数据。如下图所示，ThreadLocalMap 会初始化一个长度为 16 的 Entry 数组，每个 Entry 对象用于保存 key-value 键值对。与 HashMap 不同的是，Entry 的 key 就是 ThreadLocal 对象本身，value 就是用户具体需要存储的值。</p>
<p data-nodeid="974"><img src="https://s0.lgstatic.com/i/image/M00/8C/49/Ciqc1F_qwveAHxDEAASdhRTxzyk624.png" alt="Drawing 2.png" data-nodeid="1101"></p>
<p data-nodeid="975">当调用 ThreadLocal.set() 添加 Entry 对象时，是如何解决 Hash 冲突的呢？这就需要我们了解线性探测法的实现原理。每个 ThreadLocal 在初始化时都会有一个 Hash 值为 threadLocalHashCode，每增加一个 ThreadLocal， Hash 值就会固定增加一个魔术 HASH_INCREMENT = 0x61c88647。为什么取 0x61c88647 这个魔数呢？实验证明，通过 0x61c88647 累加生成的 threadLocalHashCode 与 2 的幂取模，得到的结果可以较为均匀地分布在长度为 2 的幂大小的数组中。有了 threadLocalHashCode 的基础，下面我们通过下面的表格来具体讲解线性探测法是如何实现的。</p>
<p data-nodeid="1687" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8C/55/CgqCHl_qyZGABbMMAACKf1C8HLE741.png" alt="图片2.png" data-nodeid="1690"></p>


<p data-nodeid="977">为了便于理解，我们采用一组简单的数据模拟 ThreadLocal.set() 的过程是如何解决 Hash 冲突的。</p>
<ol data-nodeid="978">
<li data-nodeid="979">
<p data-nodeid="980">threadLocalHashCode = 4，threadLocalHashCode &amp; 15 = 4；此时数据应该放在数组下标为 4 的位置。下标 4 的位置正好没有数据，可以存放。</p>
</li>
<li data-nodeid="981">
<p data-nodeid="982">threadLocalHashCode = 19，threadLocalHashCode &amp; 15 = 4；但是下标 4 的位置已经有数据了，如果当前需要添加的 Entry 与下标 4 位置已存在的 Entry 两者的 key 相同，那么该位置 Entry 的 value 将被覆盖为新的值。我们假设 key 都是不相同的，所以此时需要向后移动一位，下标 5 的位置没有冲突，可以存放。</p>
</li>
<li data-nodeid="983">
<p data-nodeid="984">threadLocalHashCode = 33，threadLocalHashCode &amp; 15 = 3；下标 3 的位置已经有数据，向后移一位，下标 4 位置还是有数据，继续向后查找，发现下标 6 没有数据，可以存放。</p>
</li>
</ol>
<p data-nodeid="985">ThreadLocal.get() 的过程也是类似的，也是根据 threadLocalHashCode 的值定位到数组下标，然后判断当前位置 Entry 对象与待查询 Entry 对象的 key 是否相同，如果不同，继续向下查找。由此可见，ThreadLocal.set()/get() 方法在数据密集时很容易出现 Hash 冲突，需要 O(n) 时间复杂度解决冲突问题，效率较低。</p>
<p data-nodeid="986">下面我们再聊聊 ThreadLocalMap 中 Entry 的设计原理。Entry 继承自弱引用类 WeakReference，Entry 的 key 是弱引用，value 是强引用。在 JVM 垃圾回收时，只要发现了弱引用的对象，不管内存是否充足，都会被回收。那么为什么 Entry 的 key 要设计成弱引用呢？我们试想下，如果 key 都是强引用，当 ThreadLocal 不再使用时，然而 ThreadLocalMap 中还是存在对 ThreadLocal 的强引用，那么 GC 是无法回收的，从而造成内存泄漏。</p>
<p data-nodeid="987">虽然 Entry 的 key 设计成了弱引用，但是当 ThreadLocal 不再使用被 GC 回收后，ThreadLocalMap 中可能出现 Entry 的 key 为 NULL，那么 Entry 的 value 一直会强引用数据而得不到释放，只能等待线程销毁。那么应该如何避免 ThreadLocalMap 内存泄漏呢？ThreadLocal 已经帮助我们做了一定的保护措施，在执行 ThreadLocal.set()/get() 方法时，ThreadLocal 会清除 ThreadLocalMap 中 key 为 NULL 的 Entry 对象，让它还能够被 GC 回收。除此之外，当线程中某个 ThreadLocal 对象不再使用时，立即调用 remove() 方法删除 Entry 对象。如果是在异常的场景中，记得在 finally 代码块中进行清理，保持良好的编码意识。</p>
<p data-nodeid="988">关于 JDK 的 ThreadLocal 的基本原理我们已经介绍完了，既然 ThreadLocal 已经非常成熟，而且在日常开发中也被广泛使用，Netty 为什么还要自己实现一个 FastThreadLocal 呢？性能真的比 ThreadLocal 高很多吗？我们接下来一起一探究竟。</p>
<h3 data-nodeid="989">FastThreadLocal 为什么快</h3>
<p data-nodeid="990">FastThreadLocal 的实现与 ThreadLocal 非常类似，Netty 为 FastThreadLocal 量身打造了 FastThreadLocalThread 和 InternalThreadLocalMap 两个重要的类。下面我们看下这两个类是如何实现的。</p>
<p data-nodeid="991">FastThreadLocalThread 是对 Thread 类的一层包装，每个线程对应一个 InternalThreadLocalMap 实例。只有 FastThreadLocal 和 FastThreadLocalThread 组合使用时，才能发挥 FastThreadLocal 的性能优势。首先看下 FastThreadLocalThread 的源码定义：</p>
<pre class="lang-java" data-nodeid="992"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FastThreadLocalThread</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Thread</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> InternalThreadLocalMap threadLocalMap;
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="993">可以看出 FastThreadLocalThread 主要扩展了 InternalThreadLocalMap 字段，我们可以猜测到 FastThreadLocalThread 主要使用 InternalThreadLocalMap 存储数据，而不再是使用 Thread 中的 ThreadLocalMap。所以想知道 FastThreadLocalThread 高性能的奥秘，必须要了解 InternalThreadLocalMap 的设计原理。</p>
<p data-nodeid="994">上文中我们讲到了 ThreadLocal 的一个重要缺点，就是 ThreadLocalMap 采用线性探测法解决 Hash 冲突性能较慢，那么 InternalThreadLocalMap 又是如何优化的呢？首先一起看下 InternalThreadLocalMap 的内部构造。</p>
<pre class="lang-java" data-nodeid="995"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">InternalThreadLocalMap</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UnpaddedInternalThreadLocalMap</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> DEFAULT_ARRAY_LIST_INITIAL_CAPACITY = <span class="hljs-number">8</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> STRING_BUILDER_INITIAL_SIZE;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> STRING_BUILDER_MAX_SIZE;
&nbsp; &nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Object UNSET = <span class="hljs-keyword">new</span> Object();
&nbsp; &nbsp; <span class="hljs-keyword">private</span> BitSet cleanerFlags;

&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">InternalThreadLocalMap</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>(newIndexedVariableTable());
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Object[] newIndexedVariableTable() {
&nbsp; &nbsp; &nbsp; &nbsp; Object[] array = <span class="hljs-keyword">new</span> Object[<span class="hljs-number">32</span>];
&nbsp; &nbsp; &nbsp; &nbsp; Arrays.fill(array, UNSET);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> array;
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> <span class="hljs-title">nextVariableIndex</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> index = nextIndex.getAndIncrement();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (index &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; nextIndex.decrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"too many thread-local indexed variables"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> index;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UnpaddedInternalThreadLocalMap</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ThreadLocal&lt;InternalThreadLocalMap&gt; slowThreadLocalMap = <span class="hljs-keyword">new</span> ThreadLocal&lt;InternalThreadLocalMap&gt;();
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> AtomicInteger nextIndex = <span class="hljs-keyword">new</span> AtomicInteger();

&nbsp; &nbsp; Object[] indexedVariables;
&nbsp; &nbsp; UnpaddedInternalThreadLocalMap(Object[] indexedVariables) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.indexedVariables = indexedVariables;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="996">从 InternalThreadLocalMap 内部实现来看，与 ThreadLocalMap 一样都是采用数组的存储方式。但是 InternalThreadLocalMap 并没有使用线性探测法来解决 Hash 冲突，而是在 FastThreadLocal 初始化的时候分配一个数组索引 index，index 的值采用原子类 AtomicInteger 保证顺序递增，通过调用 InternalThreadLocalMap.nextVariableIndex() 方法获得。然后在读写数据的时候通过数组下标 index 直接定位到 FastThreadLocal 的位置，时间复杂度为 O(1)。如果数组下标递增到非常大，那么数组也会比较大，所以 FastThreadLocal 是通过空间换时间的思想提升读写性能。下面通过一幅图描述 InternalThreadLocalMap、index 和 FastThreadLocal 之间的关系。</p>
<p data-nodeid="997"><img src="https://s0.lgstatic.com/i/image/M00/8C/49/Ciqc1F_qw1KAUXO0AAMZJ_Hk4dQ099.png" alt="Drawing 3.png" data-nodeid="1130"></p>
<p data-nodeid="998">通过上面 FastThreadLocal 的内部结构图，我们对比下与 ThreadLocal 有哪些区别呢？FastThreadLocal 使用 Object 数组替代了 Entry 数组，Object[0] 存储的是一个Set&lt;FastThreadLocal&lt;?&gt;&gt; 集合，从数组下标 1 开始都是直接存储的 value 数据，不再采用 ThreadLocal 的键值对形式进行存储。</p>
<p data-nodeid="999">假设现在我们有一批数据需要添加到数组中，分别为 value1、value2、value3、value4，对应的 FastThreadLocal 在初始化的时候生成的数组索引分别为 1、2、3、4。如下图所示。</p>
<p data-nodeid="1000"><img src="https://s0.lgstatic.com/i/image/M00/8C/49/Ciqc1F_qw1qAYzsdAAEbsTk70Is389.png" alt="Drawing 4.png" data-nodeid="1143"></p>
<p data-nodeid="1001">至此，我们已经对 FastThreadLocal 有了一个基本的认识，下面我们结合具体的源码分析 FastThreadLocal 的实现原理。</p>
<h3 data-nodeid="1002">FastThreadLocal 源码分析</h3>
<p data-nodeid="1003">在讲解源码之前，我们回过头看下上文中的 ThreadLocal 示例，如果把示例中 ThreadLocal 替换成 FastThread，应当如何使用呢？</p>
<pre class="lang-java" data-nodeid="1004"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FastThreadLocalTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> FastThreadLocal&lt;String&gt; THREAD_NAME_LOCAL = <span class="hljs-keyword">new</span> FastThreadLocal&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> FastThreadLocal&lt;TradeOrder&gt; TRADE_THREAD_LOCAL = <span class="hljs-keyword">new</span> FastThreadLocal&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">2</span>; i++) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> tradeId = i;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String threadName = <span class="hljs-string">"thread-"</span> + i;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> FastThreadLocalThread(() -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; THREAD_NAME_LOCAL.set(threadName);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TradeOrder tradeOrder = <span class="hljs-keyword">new</span> TradeOrder(tradeId, tradeId % <span class="hljs-number">2</span> == <span class="hljs-number">0</span> ? <span class="hljs-string">"已支付"</span> : <span class="hljs-string">"未支付"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TRADE_THREAD_LOCAL.set(tradeOrder);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"threadName: "</span> + THREAD_NAME_LOCAL.get());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"tradeOrder info："</span> + TRADE_THREAD_LOCAL.get());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }, threadName).start();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1005">可以看出，FastThreadLocal 的使用方法几乎和 ThreadLocal 保持一致，只需要把代码中 Thread、ThreadLocal 替换为 FastThreadLocalThread 和 FastThreadLocal 即可，Netty 在易用性方面做得相当棒。下面我们重点对示例中用得到 FastThreadLocal.set()/get() 方法做深入分析。</p>
<p data-nodeid="1006">首先看下 FastThreadLocal.set() 的源码：</p>
<pre class="lang-java" data-nodeid="1007"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">set</span><span class="hljs-params">(V value)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (value != InternalThreadLocalMap.UNSET) { <span class="hljs-comment">// 1. value 是否为缺省值</span>
&nbsp; &nbsp; &nbsp; &nbsp; InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.get(); <span class="hljs-comment">// 2. 获取当前线程的 InternalThreadLocalMap</span>
&nbsp; &nbsp; &nbsp; &nbsp; setKnownNotUnset(threadLocalMap, value); <span class="hljs-comment">// 3. 将 InternalThreadLocalMap 中数据替换为新的 value</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; remove();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1008">FastThreadLocal.set() 方法虽然入口只有几行代码，但是内部逻辑是相当复杂的。我们首先还是抓住代码主干，一步步进行拆解分析。set() 的过程主要分为三步：</p>
<ol data-nodeid="1009">
<li data-nodeid="1010">
<p data-nodeid="1011">判断 value 是否为缺省值，如果等于缺省值，那么直接调用 remove() 方法。这里我们还不知道缺省值和 remove() 之间的联系是什么，我们暂且把 remove() 放在最后分析。</p>
</li>
<li data-nodeid="1012">
<p data-nodeid="1013">如果 value 不等于缺省值，接下来会获取当前线程的 InternalThreadLocalMap。</p>
</li>
<li data-nodeid="1014">
<p data-nodeid="1015">然后将 InternalThreadLocalMap 中对应数据替换为新的 value。</p>
</li>
</ol>
<p data-nodeid="1016">首先我们看下 InternalThreadLocalMap.get() 方法，源码如下：</p>
<pre class="lang-java" data-nodeid="1017"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> InternalThreadLocalMap <span class="hljs-title">get</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; Thread thread = Thread.currentThread();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (thread <span class="hljs-keyword">instanceof</span> FastThreadLocalThread) { <span class="hljs-comment">// 当前线程是否为 FastThreadLocalThread 类型</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> fastGet((FastThreadLocalThread) thread);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> slowGet();
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> InternalThreadLocalMap <span class="hljs-title">fastGet</span><span class="hljs-params">(FastThreadLocalThread thread)</span> </span>{
&nbsp; &nbsp; InternalThreadLocalMap threadLocalMap = thread.threadLocalMap(); <span class="hljs-comment">// 获取 FastThreadLocalThread 的 threadLocalMap 属性</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadLocalMap == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; thread.setThreadLocalMap(threadLocalMap = <span class="hljs-keyword">new</span> InternalThreadLocalMap());
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> threadLocalMap;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> InternalThreadLocalMap <span class="hljs-title">slowGet</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; ThreadLocal&lt;InternalThreadLocalMap&gt; slowThreadLocalMap = UnpaddedInternalThreadLocalMap.slowThreadLocalMap;&nbsp;
&nbsp; &nbsp; InternalThreadLocalMap ret = slowThreadLocalMap.get(); <span class="hljs-comment">// 从 JDK 原生 ThreadLocal 中获取 InternalThreadLocalMap</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ret == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; ret = <span class="hljs-keyword">new</span> InternalThreadLocalMap();
&nbsp; &nbsp; &nbsp; &nbsp; slowThreadLocalMap.set(ret);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> ret;
}
</code></pre>
<p data-nodeid="1018">InternalThreadLocalMap.get() 逻辑很简单，为了帮助你更好地理解，下面使用一幅图描述 InternalThreadLocalMap 的获取方式。</p>
<p data-nodeid="1019"><img src="https://s0.lgstatic.com/i/image/M00/8C/49/Ciqc1F_qw2WAV1UtAAWTkglpnjs396.png" alt="Drawing 5.png" data-nodeid="1157"></p>
<p data-nodeid="1020">如果当前线程是 FastThreadLocalThread 类型，那么直接通过 fastGet() 方法获取 FastThreadLocalThread 的 threadLocalMap 属性即可。如果此时 InternalThreadLocalMap 不存在，直接创建一个返回。关于 InternalThreadLocalMap 的初始化在上文中已经介绍过，它会初始化一个长度为 32 的 Object 数组，数组中填充着 32 个缺省对象 UNSET 的引用。</p>
<p data-nodeid="1021">那么 slowGet() 又是什么作用呢？从代码分支来看，slowGet() 是针对非 FastThreadLocalThread 类型的线程发起调用时的一种兜底方案。如果当前线程不是 FastThreadLocalThread，内部是没有 InternalThreadLocalMap 属性的，Netty 在 UnpaddedInternalThreadLocalMap 中保存了一个 JDK 原生的 ThreadLocal，ThreadLocal 中存放着 InternalThreadLocalMap，此时获取 InternalThreadLocalMap 就退化成 JDK 原生的 ThreadLocal 获取。</p>
<p data-nodeid="1022">获取 InternalThreadLocalMap 的过程已经讲完了，下面看下 setKnownNotUnset() 如何将数据添加到 InternalThreadLocalMap 的。</p>
<pre class="lang-java" data-nodeid="1023"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setKnownNotUnset</span><span class="hljs-params">(InternalThreadLocalMap threadLocalMap, V value)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadLocalMap.setIndexedVariable(index, value)) { <span class="hljs-comment">// 1. 找到数组下标 index 位置，设置新的 value</span>
&nbsp; &nbsp; &nbsp; &nbsp; addToVariablesToRemove(threadLocalMap, <span class="hljs-keyword">this</span>); <span class="hljs-comment">// 2. 将 FastThreadLocal 对象保存到待清理的 Set 中</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1024">setKnownNotUnset() 主要做了两件事：</p>
<ol data-nodeid="1025">
<li data-nodeid="1026">
<p data-nodeid="1027">找到数组下标 index 位置，设置新的 value。</p>
</li>
<li data-nodeid="1028">
<p data-nodeid="1029">将 FastThreadLocal 对象保存到待清理的 Set 中。</p>
</li>
</ol>
<p data-nodeid="1030">首先我们看下第一步 threadLocalMap.setIndexedVariable() 的源码实现：</p>
<pre class="lang-java" data-nodeid="1031"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">setIndexedVariable</span><span class="hljs-params">(<span class="hljs-keyword">int</span> index, Object value)</span> </span>{
&nbsp; &nbsp; Object[] lookup = indexedVariables;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (index &lt; lookup.length) {
&nbsp; &nbsp; &nbsp; &nbsp; Object oldValue = lookup[index];&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; lookup[index] = value; <span class="hljs-comment">// 直接将数组 index 位置设置为 value，时间复杂度为 O(1)</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> oldValue == UNSET;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; expandIndexedVariableTableAndSet(index, value); <span class="hljs-comment">// 容量不够，先扩容再设置值</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1032">indexedVariables 就是 InternalThreadLocalMap 中用于存放数据的数组，如果数组容量大于 FastThreadLocal 的 index 索引，那么直接找到数组下标 index 位置将新 value 设置进去，事件复杂度为 O(1)。在设置新的 value 之前，会将之前 index 位置的元素取出，如果旧的元素还是 UNSET 缺省对象，那么返回成功。</p>
<p data-nodeid="1033">如果数组容量不够了怎么办呢？InternalThreadLocalMap 会自动扩容，然后再设置 value。接下来看看 expandIndexedVariableTableAndSet() 的扩容逻辑：</p>
<pre class="lang-java" data-nodeid="1034"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">expandIndexedVariableTableAndSet</span><span class="hljs-params">(<span class="hljs-keyword">int</span> index, Object value)</span> </span>{
&nbsp; &nbsp; Object[] oldArray = indexedVariables;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> oldCapacity = oldArray.length;
&nbsp; &nbsp; <span class="hljs-keyword">int</span> newCapacity = index;
&nbsp; &nbsp; newCapacity |= newCapacity &gt;&gt;&gt;&nbsp; <span class="hljs-number">1</span>;
&nbsp; &nbsp; newCapacity |= newCapacity &gt;&gt;&gt;&nbsp; <span class="hljs-number">2</span>;
&nbsp; &nbsp; newCapacity |= newCapacity &gt;&gt;&gt;&nbsp; <span class="hljs-number">4</span>;
&nbsp; &nbsp; newCapacity |= newCapacity &gt;&gt;&gt;&nbsp; <span class="hljs-number">8</span>;
&nbsp; &nbsp; newCapacity |= newCapacity &gt;&gt;&gt; <span class="hljs-number">16</span>;
&nbsp; &nbsp; newCapacity ++;
&nbsp; &nbsp; Object[] newArray = Arrays.copyOf(oldArray, newCapacity);
&nbsp; &nbsp; Arrays.fill(newArray, oldCapacity, newArray.length, UNSET);
&nbsp; &nbsp; newArray[index] = value;
&nbsp; &nbsp; indexedVariables = newArray;
}
</code></pre>
<p data-nodeid="1035">上述代码的位移操作是不是似曾相识？我们去翻阅下 JDK HashMap 中扩容的源码，其中有这么一段代码：</p>
<pre class="lang-java" data-nodeid="1036"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> <span class="hljs-title">tableSizeFor</span><span class="hljs-params">(<span class="hljs-keyword">int</span> cap)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> n = cap - <span class="hljs-number">1</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">1</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">2</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">4</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">8</span>;
&nbsp; &nbsp; n |= n &gt;&gt;&gt; <span class="hljs-number">16</span>;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> (n &lt; <span class="hljs-number">0</span>) ? <span class="hljs-number">1</span> : (n &gt;= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + <span class="hljs-number">1</span>;
}
</code></pre>
<p data-nodeid="1037">可以看出 InternalThreadLocalMap 实现数组扩容几乎和 HashMap 完全是一模一样的，所以多读源码还是可以给我们很多启发的。InternalThreadLocalMap 以 index 为基准进行扩容，将数组扩容后的容量向上取整为 2 的次幂。然后将原数组内容拷贝到新的数组中，空余部分填充缺省对象 UNSET，最终把新数组赋值给 indexedVariables。</p>
<p data-nodeid="1038">为什么 InternalThreadLocalMap 以 index 为基准进行扩容，而不是原数组长度呢？假设现在初始化了 70 个 FastThreadLocal，但是这些 FastThreadLocal 从来没有调用过 set() 方法，此时数组还是默认长度 32。当第 index = 70 的 FastThreadLocal 调用 set() 方法时，如果按原数组容量 32 进行扩容 2 倍后，还是无法填充 index = 70 的数据。所以使用 index 为基准进行扩容可以解决这个问题，但是如果 FastThreadLocal 特别多，数组的长度也是非常大的。</p>
<p data-nodeid="1039">回到 setKnownNotUnset() 的主流程，向 InternalThreadLocalMap 添加完数据之后，接下就是将 FastThreadLocal 对象保存到待清理的 Set 中。我们继续看下 addToVariablesToRemove() 是如何实现的。</p>
<pre class="lang-java" data-nodeid="1040"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addToVariablesToRemove</span><span class="hljs-params">(InternalThreadLocalMap threadLocalMap, FastThreadLocal&lt;?&gt; variable)</span> </span>{
&nbsp; &nbsp; Object v = threadLocalMap.indexedVariable(variablesToRemoveIndex); <span class="hljs-comment">// 获取数组下标为 0 的元素</span>
&nbsp; &nbsp; Set&lt;FastThreadLocal&lt;?&gt;&gt; variablesToRemove;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (v == InternalThreadLocalMap.UNSET || v == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; variablesToRemove = Collections.newSetFromMap(<span class="hljs-keyword">new</span> IdentityHashMap&lt;FastThreadLocal&lt;?&gt;, Boolean&gt;()); <span class="hljs-comment">// 创建 FastThreadLocal 类型的 Set 集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; threadLocalMap.setIndexedVariable(variablesToRemoveIndex, variablesToRemove); <span class="hljs-comment">// 将 Set 集合填充到数组下标 0 的位置</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; variablesToRemove = (Set&lt;FastThreadLocal&lt;?&gt;&gt;) v; <span class="hljs-comment">// 如果不是 UNSET，Set 集合已存在，直接强转获得 Set 集合</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; variablesToRemove.add(variable); <span class="hljs-comment">// 将 FastThreadLocal 添加到 Set 集合中</span>
}
</code></pre>
<p data-nodeid="1041">variablesToRemoveIndex 是采用 static final 修饰的变量，在 FastThreadLocal 初始化时 variablesToRemoveIndex 被赋值为 0。InternalThreadLocalMap 首先会找到数组下标为 0 的元素，如果该元素是缺省对象 UNSET 或者不存在，那么会创建一个 FastThreadLocal 类型的 Set 集合，然后把 Set 集合填充到数组下标 0 的位置。如果数组第一个元素不是缺省对象 UNSET，说明 Set 集合已经被填充，直接强转获得 Set 集合即可。这就解释了 InternalThreadLocalMap 的 value 数据为什么是从下标为 1 的位置开始存储了，因为 0 的位置已经被 Set 集合占用了。</p>
<p data-nodeid="1042">为什么 InternalThreadLocalMap 要在数组下标为 0 的位置存放一个 FastThreadLocal 类型的 Set 集合呢？这时候我们回过头看下 remove() 方法。</p>
<pre class="lang-java" data-nodeid="1043"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">remove</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; remove(InternalThreadLocalMap.getIfSet());
}
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> InternalThreadLocalMap <span class="hljs-title">getIfSet</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; Thread thread = Thread.currentThread();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (thread <span class="hljs-keyword">instanceof</span> FastThreadLocalThread) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> ((FastThreadLocalThread) thread).threadLocalMap();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> slowThreadLocalMap.get();
}
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">remove</span><span class="hljs-params">(InternalThreadLocalMap threadLocalMap)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadLocalMap == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; Object v = threadLocalMap.removeIndexedVariable(index); <span class="hljs-comment">// 删除数组下标 index 位置对应的 value</span>
&nbsp; &nbsp; removeFromVariablesToRemove(threadLocalMap, <span class="hljs-keyword">this</span>); <span class="hljs-comment">// 从数组下标 0 的位置取出 Set 集合，并删除当前 FastThreadLocal</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (v != InternalThreadLocalMap.UNSET) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; onRemoval((V) v); <span class="hljs-comment">// 空方法，用户可以继承实现</span>
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; PlatformDependent.throwException(e);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="1044">在执行 remove 操作之前，会调用 InternalThreadLocalMap.getIfSet() 获取当前 InternalThreadLocalMap。有了之前的基础，理解 getIfSet() 方法就非常简单了，如果是 FastThreadLocalThread 类型，直接取 FastThreadLocalThread 中 threadLocalMap 属性。如果是普通线程 Thread，从 ThreadLocal 类型的 slowThreadLocalMap 中获取。</p>
<p data-nodeid="1045"><br>
找到 InternalThreadLocalMap 之后，InternalThreadLocalMap 会从数组中定位到下标 index 位置的元素，并将 index 位置的元素覆盖为缺省对象 UNSET。接下来就需要清理当前的 FastThreadLocal 对象，此时 Set 集合就派上了用场，InternalThreadLocalMap 会取出数组下标 0 位置的 Set 集合，然后删除当前 FastThreadLocal。最后 onRemoval() 方法起到什么作用呢？Netty 只是留了一处扩展，并没有实现，用户需要在删除的时候做一些后置操作，可以继承 FastThreadLocal 实现该方法。</p>
<p data-nodeid="1046">至此，FastThreadLocal.set() 的完成过程已经讲完了，接下来我们继续 FastThreadLocal.get() 方法的实现就易如反掌拉。FastThreadLocal.get() 的源码实现如下：</p>
<pre class="lang-java" data-nodeid="1047"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> V <span class="hljs-title">get</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; InternalThreadLocalMap threadLocalMap = InternalThreadLocalMap.get();
&nbsp; &nbsp; Object v = threadLocalMap.indexedVariable(index); <span class="hljs-comment">// 从数组中取出 index 位置的元素</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (v != InternalThreadLocalMap.UNSET) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> (V) v;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> initialize(threadLocalMap); <span class="hljs-comment">// 如果获取到的数组元素是缺省对象，执行初始化操作</span>
}
<span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">indexedVariable</span><span class="hljs-params">(<span class="hljs-keyword">int</span> index)</span> </span>{
&nbsp; &nbsp; Object[] lookup = indexedVariables;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> index &lt; lookup.length? lookup[index] : UNSET;
}
<span class="hljs-function"><span class="hljs-keyword">private</span> V <span class="hljs-title">initialize</span><span class="hljs-params">(InternalThreadLocalMap threadLocalMap)</span> </span>{
&nbsp; &nbsp; V v = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; v = initialValue();
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; &nbsp; &nbsp; PlatformDependent.throwException(e);
&nbsp; &nbsp; }
&nbsp; &nbsp; threadLocalMap.setIndexedVariable(index, v);
&nbsp; &nbsp; addToVariablesToRemove(threadLocalMap, <span class="hljs-keyword">this</span>);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> v;
}
</code></pre>
<p data-nodeid="1048">首先根据当前线程是否是 FastThreadLocalThread 类型找到 InternalThreadLocalMap，然后取出从数组下标 index 的元素，如果 index 位置的元素不是缺省对象 UNSET，说明该位置已经填充过数据，直接取出返回即可。如果 index 位置的元素是缺省对象 UNSET，那么需要执行初始化操作。可以看到，initialize() 方法会调用用户重写的 initialValue 方法构造需要存储的对象数据，如下所示。</p>
<pre class="lang-java" data-nodeid="1049"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> FastThreadLocal&lt;String&gt; threadLocal = <span class="hljs-keyword">new</span> FastThreadLocal&lt;String&gt;() {
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> String <span class="hljs-title">initialValue</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"hello world"</span>;
&nbsp; &nbsp; }
};
</code></pre>
<p data-nodeid="1050">构造完用户对象数据之后，接下来就会将它填充到数组 index 的位置，然后再把当前 FastThreadLocal 对象保存到待清理的 Set 中。整个过程我们在分析 FastThreadLocal.set() 时都已经介绍过，就不再赘述了。</p>
<p data-nodeid="1051">到此为止，FastThreadLocal 最核心的两个方法 set()/get() 我们已经分析完了。下面有两个问题我们再深入思考下。</p>
<ol data-nodeid="1052">
<li data-nodeid="1053">
<p data-nodeid="1054">FastThreadLocal 真的一定比 ThreadLocal 快吗？答案是不一定的，只有使用FastThreadLocalThread 类型的线程才会更快，如果是普通线程反而会更慢。</p>
</li>
<li data-nodeid="1055">
<p data-nodeid="1056">FastThreadLocal 会浪费很大的空间吗？虽然 FastThreadLocal 采用的空间换时间的思路，但是在 FastThreadLocal 设计之初就认为不会存在特别多的 FastThreadLocal 对象，而且在数据中没有使用的元素只是存放了同一个缺省对象的引用，并不会占用太多内存空间。</p>
</li>
</ol>
<h3 data-nodeid="1057">总结</h3>
<p data-nodeid="1058">本节课我们对比介绍了 ThreadLocal 和 FastThreadLocal，简单总结下 FastThreadLocal 的优势。</p>
<ul data-nodeid="1059">
<li data-nodeid="1060">
<p data-nodeid="1061"><strong data-nodeid="1189">高效查找</strong>。FastThreadLocal 在定位数据的时候可以直接根据数组下标 index 获取，时间复杂度 O(1)。而 JDK 原生的 ThreadLocal 在数据较多时哈希表很容易发生 Hash 冲突，线性探测法在解决 Hash 冲突时需要不停地向下寻找，效率较低。此外，FastThreadLocal 相比 ThreadLocal 数据扩容更加简单高效，FastThreadLocal 以 index 为基准向上取整到 2 的次幂作为扩容后容量，然后把原数据拷贝到新数组。而 ThreadLocal 由于采用的哈希表，所以在扩容后需要再做一轮 rehash。</p>
</li>
<li data-nodeid="1062">
<p data-nodeid="1063"><strong data-nodeid="1194">安全性更高</strong>。JDK 原生的 ThreadLocal 使用不当可能造成内存泄漏，只能等待线程销毁。在使用线程池的场景下，ThreadLocal 只能通过主动检测的方式防止内存泄漏，从而造成了一定的开销。然而 FastThreadLocal 不仅提供了 remove() 主动清除对象的方法，而且在线程池场景中 Netty 还封装了 FastThreadLocalRunnable，FastThreadLocalRunnable 最后会执行 FastThreadLocal.removeAll() 将 Set 集合中所有 FastThreadLocal 对象都清理掉，</p>
</li>
</ul>
<p data-nodeid="1064" class="">FastThreadLocal 体现了 Netty 在高性能方面精益求精的设计精神，FastThreadLocal 仅仅是其中的冰山一角，下节课我们继续探索 Netty 中其他高效的数据结构技巧。</p>

---

### 精选评论

##### **红：
> InternalThreadLocalMap 中初始化数组中第0个元素的set作用是?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Set 集合是为了保存 FastThreadLocal对象，我认为 Set 集合设计的好处有几点：1. 删除 FastThreadLocal 留扩展接口。2. 提高 removeAll 的删除效率，不需要去遍历膨胀的数组。3. 可以更好地做内存泄露的管理。

##### *迪：
> 有点不明白，既然threadLocalMap中的entry是个弱引用对象，为啥key都失去引用了gc还不回收，还能存在你说的null key，value有值的情况,如果是这样的话那弱引用的意义在哪？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果 key 都是强引用，当 ThreadLocal 不再使用时，然而 ThreadLocalMap 中还是存在对 ThreadLocal 的强引用，那么 GC 是无法回收的，从而造成内存泄漏。弱引用可以避免长期存活的线程（例如线程池）导致 ThreadLocal 无法回收造成的内存泄漏。

##### **杰：
> 666

