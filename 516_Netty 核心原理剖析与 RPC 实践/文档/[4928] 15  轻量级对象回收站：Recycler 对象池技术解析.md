<p data-nodeid="821" class="">前面两节课，我们学习了 Netty 内存池的高性能设计原理，这节课会介绍 Netty 的另一种池化技术：Recycler 对象池。在刚接触到 Netty 对象池这个概念时，你是不是也会有类似的疑问：</p>
<ul data-nodeid="822">
<li data-nodeid="823">
<p data-nodeid="824">对象池和内存池有什么区别？它们有什么联系吗？</p>
</li>
<li data-nodeid="825">
<p data-nodeid="826">实现对象池的方法有很多，Netty 也是自己实现的吗？是如何实现的？</p>
</li>
<li data-nodeid="827">
<p data-nodeid="828">对象池在实践中我们应该怎么使用？</p>
</li>
</ul>
<p data-nodeid="829">带着这些问题，我们进入今天课程的学习吧。</p>
<h3 data-nodeid="830">Recycler 快速上手</h3>
<p data-nodeid="831">我们通过一个例子直观感受下 Recycler 如何使用，假设我们有一个 User 类，需要实现 User 对象的复用，具体实现代码如下：</p>
<pre class="lang-java" data-nodeid="832"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserCache</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Recycler&lt;User&gt; userRecycler = <span class="hljs-keyword">new</span> Recycler&lt;User&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> User <span class="hljs-title">newObject</span><span class="hljs-params">(Handle&lt;User&gt; handle)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> User(handle);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; };
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">private</span> String name;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">private</span> Recycler.Handle&lt;User&gt; handle;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setName</span><span class="hljs-params">(String name)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.name = name;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getName</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> name;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">User</span><span class="hljs-params">(Recycler.Handle&lt;User&gt; handle)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.handle = handle;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">recycle</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handle.recycle(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; User user1 = userRecycler.get(); <span class="hljs-comment">// 1、从对象池获取 User 对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; user1.setName(<span class="hljs-string">"hello"</span>); <span class="hljs-comment">// 2、设置 User 对象的属性</span>
&nbsp; &nbsp; &nbsp; &nbsp; user1.recycle(); <span class="hljs-comment">// 3、回收对象到对象池</span>
&nbsp; &nbsp; &nbsp; &nbsp; User user2 = userRecycler.get(); <span class="hljs-comment">// 4、从对象池获取对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(user2.getName());
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(user1 == user2);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="833">控制台的输出结果如下：</p>
<pre class="lang-js" data-nodeid="834"><code data-language="js">hello
<span class="hljs-literal">true</span>
</code></pre>
<p data-nodeid="835">代码示例中定义了对象池实例 userRecycler，其中实现了 newObject() 方法，如果对象池没有可用的对象，会调用该方法新建对象。此外需要创建 Recycler.Handle 对象与 User 对象进行绑定，这样我们就可以通过 userRecycler.get() 从对象池中获取 User 对象，如果对象不再使用，通过调用 User 类实现的 recycle() 方法即可完成回收对象到对象池。</p>
<p data-nodeid="836">Recycler 的使用方式是不是特别简单，我们可以单独把它当作工具类在项目中使用。</p>
<h3 data-nodeid="837">Recycler 的设计理念</h3>
<p data-nodeid="838">对象池与内存池的都是为了提高 Netty 的并发处理能力，我们知道 Java 中频繁地创建和销毁对象的开销是很大的，所以很多人会将一些通用对象缓存起来，当需要某个对象时，优先从对象池中获取对象实例。通过重用对象，不仅避免频繁地创建和销毁所带来的性能损耗，而且对 JVM GC 是友好的，这就是对象池的作用。</p>
<p data-nodeid="9623">Recycler 是 Netty 提供的自定义实现的轻量级对象回收站，借助 Recycler 可以完成对象的获取和回收。既然 Recycler 是 Netty 自己实现的对象池，那么它是如何设计的呢？首先看下 Recycler 的内部结构，如下图所示：</p>
<p data-nodeid="21868" class=""><img src="https://s0.lgstatic.com/i/image/M00/7D/21/CgqCHl_OCESAR1UWAAaA8pFs2bg205.png" alt="333.png" data-nodeid="21871"></p>





























































<p data-nodeid="841">通过 Recycler 的 UML 图可以看出，一共包含四个核心组件：<strong data-nodeid="942">Stack</strong>、<strong data-nodeid="943">WeakOrderQueue</strong>、<strong data-nodeid="944">Link</strong>、<strong data-nodeid="945">DefaultHandle</strong>，接下来我们逐一进行介绍。</p>
<p data-nodeid="842">首先我们先看下整个 Recycler 的内部结构中各个组件的关系，可以通过下面这幅图进行描述。</p>
<p data-nodeid="22705" class=""><img src="https://s0.lgstatic.com/i/image/M00/7D/22/CgqCHl_OCJmABlSHAATi6fhCKaA360.png" alt="111.png" data-nodeid="22708"></p>


<p data-nodeid="844">第一个核心组件是 Stack，Stack 是整个对象池的顶层数据结构，描述了整个对象池的构造，用于存储当前本线程回收的对象。在多线程的场景下，Netty 为了避免锁竞争问题，每个线程都会持有各自的对象池，内部通过 FastThreadLocal 来实现每个线程的私有化。FastThreadLocal 你可以理解为 Java 里的 ThreadLocal，后续会有专门的课程介绍它。</p>
<p data-nodeid="845">我们有必要先学习下 Stack 的数据结构，先看下 Stack 的源码定义：</p>
<pre class="lang-java" data-nodeid="846"><code data-language="java"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Stack</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Recycler&lt;T&gt; parent; <span class="hljs-comment">// 所属的 Recycler</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> WeakReference&lt;Thread&gt; threadRef; <span class="hljs-comment">// 所属线程的弱引用</span>

&nbsp; &nbsp; <span class="hljs-keyword">final</span> AtomicInteger availableSharedCapacity; <span class="hljs-comment">// 异线程回收对象时，其他线程能保存的被回收对象的最大个数</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> maxDelayedQueues; <span class="hljs-comment">// WeakOrderQueue最大个数</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> maxCapacity; <span class="hljs-comment">// 对象池的最大大小，默认最大为 4k</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> ratioMask; <span class="hljs-comment">// 控制对象的回收比率，默认只回收 1/8 的对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> DefaultHandle&lt;?&gt;[] elements; <span class="hljs-comment">// 存储缓存数据的数组</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> size; <span class="hljs-comment">// 缓存的 DefaultHandle 对象个数</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> handleRecycleCount = -<span class="hljs-number">1</span>;&nbsp;

&nbsp; &nbsp; <span class="hljs-comment">// WeakOrderQueue 链表的三个重要节点</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> WeakOrderQueue cursor, prev;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">volatile</span> WeakOrderQueue head;

&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="847">对应上面 Recycler 的内部结构图，Stack 包用于存储缓存数据的 DefaultHandle 数组，以及维护了 WeakOrderQueue 链表中的三个重要节点，关于 WeakOrderQueue 相关概念我们之后再详细介绍。除此之外，Stack 其他的重要属性我在源码中已经全部以注释的形式标出，大部分已经都非常清楚，其中 availableSharedCapacity 是比较难理解的，每个 Stack 会维护一个 WeakOrderQueue 的链表，每个 WeakOrderQueue 节点会保存非当前线程的其他线程所释放的对象，例如图中 ThreadA 表示当前线程，WeakOrderQueue 的链表存储着 ThreadB、ThreadC 等其他线程释放的对象。availableSharedCapacity 的初始化方式为 new AtomicInteger(max(maxCapacity / maxSharedCapacityFactor, LINK_CAPACITY))，默认大小为 16K，其他线程在回收对象时，最多可以回收 ThreadA 创建的对象个数不能超过 availableSharedCapacity。还有一个疑问就是既然 Stack 是每个线程私有的，为什么 availableSharedCapacity 还需要用 AtomicInteger 呢？因为 ThreadB、ThreadC 等多个线程可能都会创建 ThreadA 的 WeakOrderQueue，存在同时操作 availableSharedCapacity 的情况。</p>
<p data-nodeid="848">第二个要介绍的组件是 WeakOrderQueue，WeakOrderQueue 用于存储其他线程回收到当前线程所分配的对象，并且在合适的时机，Stack 会从异线程的 WeakOrderQueue 中收割对象。如上图所示，ThreadB 回收到 ThreadA 所分配的内存时，就会被放到 ThreadA 的 WeakOrderQueue 当中。</p>
<p data-nodeid="849">第三个组件是 Link，每个 WeakOrderQueue 中都包含一个 Link 链表，回收对象都会被存在 Link 链表中的节点上，每个 Link 节点默认存储 16 个对象，当每个 Link 节点存储满了会创建新的 Link 节点放入链表尾部。</p>
<p data-nodeid="850">第四个组件是 DefaultHandle，DefaultHandle 实例中保存了实际回收的对象，Stack 和 WeakOrderQueue 都使用 DefaultHandle 存储回收的对象。在 Stack 中包含一个 elements 数组，该数组保存的是 DefaultHandle 实例。DefaultHandle 中每个 Link 节点所存储的 16 个对象也是使用 DefaultHandle 表示的。</p>
<p data-nodeid="851">到此为止，我们已经介绍完 Recycler 的内存结构，对 Recycler 有了初步的认识。Recycler 作为一个高性能的对象池，在多线程的场景下，Netty 是如何保证 Recycler 高效地分配和回收对象的呢？接下来我们一起看下 Recycler 对象获取和回收的原理。</p>
<h3 data-nodeid="852">从 Recycler 中获取对象</h3>
<p data-nodeid="853">前面我们介绍了 Recycler 如何使用，从代码示例中可以看出，从对象池中获取对象的入口是在 Recycler#get() 方法，直接定位到源码：</p>
<pre class="lang-java" data-nodeid="854"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> T <span class="hljs-title">get</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (maxCapacityPerThread == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> newObject((Handle&lt;T&gt;) NOOP_HANDLE);
&nbsp; &nbsp; }
&nbsp; &nbsp; Stack&lt;T&gt; stack = threadLocal.get(); <span class="hljs-comment">// 获取当前线程缓存的 Stack</span>
&nbsp; &nbsp; DefaultHandle&lt;T&gt; handle = stack.pop(); <span class="hljs-comment">// 从 Stack 中弹出一个 DefaultHandle 对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (handle == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; handle = stack.newHandle();
&nbsp; &nbsp; &nbsp; &nbsp; handle.value = newObject(handle); <span class="hljs-comment">// 创建的对象并保存到 DefaultHandle</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> (T) handle.value;
}
</code></pre>
<p data-nodeid="855">Recycler#get() 方法的逻辑非常清晰，首先通过 FastThreadLocal 获取当前线程的唯一栈缓存 Stack，然后尝试从栈顶弹出 DefaultHandle 对象实例，如果 Stack 中没有可用的 DefaultHandle 对象实例，那么会调用 newObject 生成一个新的对象，完成 handle 与用户对象和 Stack 的绑定。</p>
<p data-nodeid="856">那么 Stack 是如何从 elements 数组中弹出 DefaultHandle 对象实例的呢？只是从 elements 数组中取出一个实例吗？我们一起跟进下 stack.pop() 的源码：</p>
<pre class="lang-java" data-nodeid="857"><code data-language="java"><span class="hljs-function">DefaultHandle&lt;T&gt; <span class="hljs-title">pop</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> size = <span class="hljs-keyword">this</span>.size;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (size == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 就尝试从其他线程回收的对象中转移一些到 elements 数组当中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!scavenge()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; size = <span class="hljs-keyword">this</span>.size;
&nbsp; &nbsp; }
&nbsp; &nbsp; size --;
&nbsp; &nbsp; DefaultHandle ret = elements[size]; <span class="hljs-comment">// 将实例从栈顶弹出</span>
&nbsp; &nbsp; elements[size] = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (ret.lastRecycledId != ret.recycleId) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"recycled multiple times"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; ret.recycleId = <span class="hljs-number">0</span>;
&nbsp; &nbsp; ret.lastRecycledId = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.size = size;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> ret;
}
</code></pre>
<p data-nodeid="858">如果 Stack 的 elements 数组中有可用的对象实例，直接将对象实例弹出；如果 elements 数组中没有可用的对象实例，会调用 scavenge 方法，scavenge 的作用是从其他线程回收的对象实例中转移一些到 elements 数组当中，也就是说，它会想办法从 WeakOrderQueue 链表中迁移部分对象实例。每个 Stack 会有一个 WeakOrderQueue 链表，每个 WeakOrderQueue 节点都维持了相应异线程回收的对象，那么以什么样的策略从 WeakOrderQueue 链表中迁移对象实例呢？继续跟进 scavenge 的源码：</p>
<pre class="lang-java" data-nodeid="859"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">scavenge</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 尝试从 WeakOrderQueue 中转移对象实例到 Stack 中</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (scavengeSome()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 如果迁移失败，就会重置 cursor 指针到 head 节点</span>
&nbsp; &nbsp; prev = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; cursor = head;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
<span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">scavengeSome</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; WeakOrderQueue prev;
&nbsp; &nbsp; WeakOrderQueue cursor = <span class="hljs-keyword">this</span>.cursor; <span class="hljs-comment">// cursor 指针指向当前 WeakorderQueueu 链表的读取位置</span>
&nbsp; &nbsp; <span class="hljs-comment">// 如果 cursor 指针为 null, 则是第一次从 WeakorderQueueu 链表中获取对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; prev = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; &nbsp; &nbsp; cursor = head;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; prev = <span class="hljs-keyword">this</span>.prev;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> success = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; <span class="hljs-comment">// 不断循环从 WeakOrderQueue 链表中找到一个可用的对象实例</span>
&nbsp; &nbsp; <span class="hljs-keyword">do</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 尝试迁移 WeakOrderQueue 中部分对象实例到 Stack 中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor.transfer(<span class="hljs-keyword">this</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; success = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; WeakOrderQueue next = cursor.next;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor.owner.get() == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果已退出的线程还有数据</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor.hasFinalData()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (;;) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cursor.transfer(<span class="hljs-keyword">this</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; success = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将已退出的线程从 WeakOrderQueue 链表中移除</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (prev != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; prev.setNext(next);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; prev = cursor;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将 cursor 指针指向下一个 WeakOrderQueue</span>
&nbsp; &nbsp; &nbsp; &nbsp; cursor = next;
&nbsp; &nbsp; } <span class="hljs-keyword">while</span> (cursor != <span class="hljs-keyword">null</span> &amp;&amp; !success);
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.prev = prev;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.cursor = cursor;
&nbsp; &nbsp; <span class="hljs-keyword">return</span> success;
}
</code></pre>
<p data-nodeid="860">scavenge 的源码中首先会从 cursor 指针指向的 WeakOrderQueue 节点回收部分对象到 Stack 的 elements 数组中，如果没有回收到数据就会将 cursor 指针移到下一个 WeakOrderQueue，重复执行以上过程直至回到到对象实例为止。具体的流程可以结合下图来理解。</p>
<p data-nodeid="23547"><img src="https://s0.lgstatic.com/i/image/M00/7D/22/CgqCHl_OCMCAAxGsAAQC8Q_DU54209.png" alt="222.png" data-nodeid="23550"><br>
此外，每次移动 cursor 时，都会检查 WeakOrderQueue 对应的线程是否已经退出了，如果线程已经退出，那么线程中的对象实例都会被回收，然后将 WeakOrderQueue 节点从链表中移除。</p>



<p data-nodeid="863">还有一个问题，每次 Stack 从 WeakOrderQueue 链表会回收多少数据呢？我们依然结合上图讲解，每个 WeakOrderQueue 中都包含一个 Link 链表，Netty 每次会回收其中的一个 Link 节点所存储的对象。从图中可以看出，Link 内部会包含一个读指针 readIndex，每个 Link 节点默认存储 16 个对象，读指针到链表尾部就是可以用于回收的对象实例，每次回收对象时，readIndex 都会从上一次记录的位置开始回收。</p>
<p data-nodeid="864">在回收对象实例之前，Netty 会计算出可回收对象的数量，加上 Stack 中已有的对象数量后，如果超过 Stack 的当前容量且小于 Stack 的最大容量，会对 Stack 进行扩容。为了防止回收对象太多导致 Stack 的容量激增，在每次回收时 Netty 会调用 dropHandle 方法控制回收频率，具体源码如下：</p>
<pre class="lang-java" data-nodeid="865"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">dropHandle</span><span class="hljs-params">(DefaultHandle&lt;?&gt; handle)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!handle.hasBeenRecycled) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> ((++handleRecycleCount &amp; ratioMask) != <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Drop the object.</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; handle.hasBeenRecycled = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
</code></pre>
<p data-nodeid="866">dropHandle 方法中主要靠 hasBeenRecycled 和 handleRecycleCount 两个变量控制回收的频率，会从每 8 个未被收回的对象中选取一个进行回收，其他的都被丢弃掉。</p>
<p data-nodeid="867">到此为止，从 Recycler 中获取对象的主流程已经讲完了，简单总结为两点：</p>
<ul data-nodeid="868">
<li data-nodeid="869">
<p data-nodeid="870">当 Stack 中 elements 有数据时，直接从栈顶弹出。</p>
</li>
<li data-nodeid="871">
<p data-nodeid="872">当 Stack 中 elements 没有数据时，尝试从 WeakOrderQueue 中回收一个 Link 包含的对象实例到 Stack 中，然后从栈顶弹出。</p>
</li>
</ul>
<h3 data-nodeid="873">Recycler 对象回收原理</h3>
<p data-nodeid="874">理解了如何从 Recycler 获取对象之后，再学习 Recycler 对象回收的原理就会清晰很多了，同样上文代码示例中定位到对象回收的源码入口 DefaultHandle#recycle()。</p>
<pre class="lang-java" data-nodeid="875"><code data-language="java"><span class="hljs-comment">// DefaultHandle#recycle</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">recycle</span><span class="hljs-params">(Object object)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (object != value) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"object does not belong to handle"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; Stack&lt;?&gt; stack = <span class="hljs-keyword">this</span>.stack;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (lastRecycledId != recycleId || stack == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"recycled already"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; stack.push(<span class="hljs-keyword">this</span>);
}
<span class="hljs-comment">// Stack#push</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">push</span><span class="hljs-params">(DefaultHandle&lt;?&gt; item)</span> </span>{
&nbsp; &nbsp; Thread currentThread = Thread.currentThread();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (threadRef.get() == currentThread) {
&nbsp; &nbsp; &nbsp; &nbsp; pushNow(item);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; pushLater(item, currentThread);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="876">从源码中可以看出，在回收对象时，会向 Stack 中 push 对象，push 会分为同线程回收和异线程回收两种情况，分别对应 pushNow 和 pushLater 两个方法，我们逐一进行分析。</p>
<h4 data-nodeid="877">同线程对象回收</h4>
<p data-nodeid="878">如果是当前线程回收自己分配的对象时，会调用 pushNow 方法：</p>
<pre class="lang-java" data-nodeid="879"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">pushNow</span><span class="hljs-params">(DefaultHandle&lt;?&gt; item)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> ((item.recycleId | item.lastRecycledId) != <span class="hljs-number">0</span>) { <span class="hljs-comment">// 防止被多次回收</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"recycled already"</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; item.recycleId = item.lastRecycledId = OWN_THREAD_ID;
&nbsp; &nbsp; <span class="hljs-keyword">int</span> size = <span class="hljs-keyword">this</span>.size;
&nbsp; &nbsp; <span class="hljs-comment">// 1. 超出最大容量 2. 控制回收速率</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (size &gt;= maxCapacity || dropHandle(item)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (size == elements.length) {
&nbsp; &nbsp; &nbsp; &nbsp; elements = Arrays.copyOf(elements, min(size &lt;&lt; <span class="hljs-number">1</span>, maxCapacity));
&nbsp; &nbsp; }
&nbsp; &nbsp; elements[size] = item;
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.size = size + <span class="hljs-number">1</span>;
}
</code></pre>
<p data-nodeid="880">同线程回收对象的逻辑非常简单，就是直接向 Stack 的 elements 数组中添加数据，对象会被存放在栈顶指针指向的位置。如果超过了 Stack 的最大容量，那么对象会被直接丢弃，同样这里使用了 dropHandle 方法控制对象的回收速率，每 8 个对象会有一个被回收到 Stack 中。</p>
<h4 data-nodeid="881">异线程对象回收</h4>
<p data-nodeid="882">接下来我们分析异线程对象回收的场景，想必你已经猜到，异线程回收对象时，并不会添加到 Stack 中，而是会与 WeakOrderQueue 直接打交道，先看下 pushLater 的源码：</p>
<pre class="lang-java" data-nodeid="883"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">pushLater</span><span class="hljs-params">(DefaultHandle&lt;?&gt; item, Thread thread)</span> </span>{
&nbsp; &nbsp; Map&lt;Stack&lt;?&gt;, WeakOrderQueue&gt; delayedRecycled = DELAYED_RECYCLED.get(); <span class="hljs-comment">// 当前线程帮助其他线程回收对象的缓存</span>
&nbsp; &nbsp; WeakOrderQueue queue = delayedRecycled.get(<span class="hljs-keyword">this</span>); <span class="hljs-comment">// 取出对象绑定的 Stack 对应的 WeakOrderQueue</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (queue == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 最多帮助 2*CPU 核数的线程回收线程</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (delayedRecycled.size() &gt;= maxDelayedQueues) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; delayedRecycled.put(<span class="hljs-keyword">this</span>, WeakOrderQueue.DUMMY); <span class="hljs-comment">// WeakOrderQueue.DUMMY 表示当前线程无法再帮助该 Stack 回收对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 新建 WeakOrderQueue</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> ((queue = WeakOrderQueue.allocate(<span class="hljs-keyword">this</span>, thread)) == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// drop object</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; delayedRecycled.put(<span class="hljs-keyword">this</span>, queue);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (queue == WeakOrderQueue.DUMMY) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// drop object</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; queue.add(item); <span class="hljs-comment">// 添加对象到 WeakOrderQueue 的 Link 链表中</span>
}
</code></pre>
<p data-nodeid="884">pushLater 的实现过程可以总结为两个步骤：<strong data-nodeid="992">获取 WeakOrderQueue</strong>，<strong data-nodeid="993">添加对象到 WeakOrderQueue 中</strong>。</p>
<p data-nodeid="885">首先看下如何获取 WeakOrderQueue 对象。通过 FastThreadLocal 取出当前对象的 DELAYED_RECYCLED 缓存，DELAYED_RECYCLED 存放着当前线程帮助其他线程回收对象的映射关系。假如 item 是 ThreadA 分配的对象，当前线程是 ThreadB，此时 ThreadB 帮助 ThreadA 回收 item，那么 DELAYED_RECYCLED 放入的 key 是 StackA。然后从 delayedRecycled 中取出 StackA 对应的 WeakOrderQueue，如果 WeakOrderQueue 不存在，那么为 StackA 新创建一个 WeakOrderQueue，并将其加入 DELAYED_RECYCLED 缓存。WeakOrderQueue.allocate() 会检查帮助 StackA 回收的对象总数是否超过 2K 个，如果没有超过 2K，会将 StackA 的 head 指针指向新创建的 WeakOrderQueue，否则不再为 StackA 回收对象。</p>
<p data-nodeid="886">当然 ThreadB 不会只帮助 ThreadA 回收对象，它可以帮助其他多个线程回收，所以 DELAYED_RECYCLED 使用的 Map 结构，为了防止 DELAYED_RECYCLED 内存膨胀，Netty 也采取了保护措施，从 delayedRecycled.size() &gt;= maxDelayedQueues 可以看出，每个线程最多帮助 2 倍 CPU 核数的线程回收线程，如果超过了该阈值，假设当前对象绑定的为 StackX，那么将在 Map 中为 StackX 放入一种特殊的 WeakOrderQueue.DUMMY，表示当前线程无法帮助 StackX 回收对象。</p>
<p data-nodeid="887">接下来我们继续分析对象是如何被添加到 WeakOrderQueue 的，直接跟进 queue.add(item) 的源码：</p>
<pre class="lang-java" data-nodeid="888"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">add</span><span class="hljs-params">(DefaultHandle&lt;?&gt; handle)</span> </span>{
&nbsp; &nbsp; handle.lastRecycledId = id;
&nbsp; &nbsp; Link tail = <span class="hljs-keyword">this</span>.tail;
&nbsp; &nbsp; <span class="hljs-keyword">int</span> writeIndex;
&nbsp; &nbsp; <span class="hljs-comment">// 如果链表尾部的 Link 已经写满，那么再新建一个 Link 追加到链表尾部</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> ((writeIndex = tail.get()) == LINK_CAPACITY) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 检查是否超过对应 Stack 可以存放的其他线程帮助回收的最大对象数</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!head.reserveSpace(LINK_CAPACITY)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// Drop it.</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.tail = tail = tail.next = <span class="hljs-keyword">new</span> Link();
&nbsp; &nbsp; &nbsp; &nbsp; writeIndex = tail.get();
&nbsp; &nbsp; }
&nbsp; &nbsp; tail.elements[writeIndex] = handle; <span class="hljs-comment">// 添加对象到 Link 尾部</span>
&nbsp; &nbsp; handle.stack = <span class="hljs-keyword">null</span>; <span class="hljs-comment">// handle 的 stack 属性赋值为 null</span>
&nbsp; &nbsp; tail.lazySet(writeIndex + <span class="hljs-number">1</span>);
}
</code></pre>
<p data-nodeid="889">在向 WeakOrderQueue 写入对象之前，会先判断 Link 链表的 tail 节点是否还有空间存放对象。如果还有空间，直接向 tail Link 尾部写入数据，否则直接丢弃对象。如果 tail Link 已经没有空间，会新建一个 Link 之后再存放对象，新建 Link 之前会检查异线程帮助回收的对象总数超过了 Stack 设置的阈值，如果超过了阈值，那么对象也会被丢弃掉。</p>
<p data-nodeid="890">对象被添加到 Link 之后，handle 的 stack 属性被赋值为 null，而在取出对象的时候，handle 的 stack 属性又再次被赋值回来，为什么这么做呢，岂不是很麻烦？如果 Stack 不再使用，期望被 GC 回收，发现 handle 中还持有 Stack 的引用，那么就无法被 GC 回收，从而造成内存泄漏。</p>
<p data-nodeid="891">到此为止，Recycler 如何回收对象的实现原理就全部分析完了，在多线程的场景下，Netty 考虑的还是非常细致的，Recycler 回收对象时向 WeakOrderQueue 中存放对象，从 Recycler 获取对象时，WeakOrderQueue 中的对象会作为 Stack 的储备，而且有效地解决了跨线程回收的问题，是一个挺新颖别致的设计。</p>
<h3 data-nodeid="892">Recycler 在 Netty 中的应用</h3>
<p data-nodeid="893">Recycler 在 Netty 里面使用也是非常频繁的，我们直接看下 Netty 源码中 newObject 相关的引用，如下图所示。</p>
<p data-nodeid="24803" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/7D/23/CgqCHl_OCNuAIe4BAAy9l8765xw221.png" alt="444.png" data-nodeid="24806"></p>



<p data-nodeid="895">其中比较常用的有 PooledHeapByteBuf 和 PooledDirectByteBuf，分别对应的堆内存和堆外内存的池化实现。例如我们在使用 PooledDirectByteBuf 的时候，并不是每次都去创建新的对象实例，而是从对象池中获取预先分配好的对象实例，不再使用 PooledDirectByteBuf 时，被回收归还到对象池中。</p>
<p data-nodeid="896">此外，可以看到内存池的 MemoryRegionCache 也有使用到对象池，MemoryRegionCache 中保存着一个队列，队列中每个 Entry 节点用于保存内存块，Entry 节点在 Netty 中就是以对象池的形式进行分配和释放，在这里我就不展开了，建议你翻阅下源码，学习下 Entry 节点是何时被分配和释放的，从而加深下对 Recycler 对象池的理解。</p>
<h3 data-nodeid="897">总结</h3>
<p data-nodeid="898">最后，简单总结下对象池几个重要的知识点：</p>
<ul data-nodeid="899">
<li data-nodeid="900">
<p data-nodeid="901">对象池有两个重要的组成部分：Stack 和 WeakOrderQueue。</p>
</li>
<li data-nodeid="902">
<p data-nodeid="903">从 Recycler 获取对象时，优先从 Stack 中查找，如果 Stack 没有可用对象，会尝试从 WeakOrderQueue 迁移部分对象到 Stack 中。</p>
</li>
<li data-nodeid="904">
<p data-nodeid="905">Recycler 回收对象时，分为同线程对象回收和异线程对象回收两种情况，同线程回收直接向 Stack 中添加对象，异线程回收向 WeakOrderQueue 中的 Link 添加对象。</p>
</li>
<li data-nodeid="906">
<p data-nodeid="907">对象回收都会控制回收速率，每 8 个对象会回收一个，其他的全部丢弃。</p>
</li>
</ul>
<p data-nodeid="908" class="">学完内存池、对象池的设计之后，相信你已经有很大的收获，同时也感受到学好数据结构是多么重要。为了避免依赖，Netty 并没有借助第三方库实现对象池，而是采用了独特的思路自己实现了一个轻量级的对象池，其中优秀的设计思路在开发中是非常值得借鉴的。如果你已经理解了 Recycler，你可以直接在项目中当成工具类使用它，在一些高并发的场景下能够较好地提升应用的性能。</p>

---

### 精选评论

##### 无：
> 2020-12-07 daka

##### **胜：
> 学到了，WeakOrderQueue的使用。一直不明白弱引用的使用场景

##### **威：
> 老师讲得很深入，学到了很多

##### **增：
> 为什么栈顶对象就是我想获取复用的呢，回收时各个handle绑定的都是不同的对象吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 每种类型的 Stack 都使用 FastThreadLocal 隔离的。

