<p data-nodeid="28947" class="">在本课时我们主要讲解为什么用完 ThreadLocal 之后都要求调用 remove 方法？</p>
<p data-nodeid="28948">首先，我们要知道这个事情和内存泄漏有关，所以就让我们先来看一下什么是内存泄漏。</p>
<h3 data-nodeid="28949">什么是内存泄漏</h3>
<p data-nodeid="28950">内存泄漏指的是，当某一个对象不再有用的时候，占用的内存却不能被回收，这就叫作<strong data-nodeid="28987">内存泄漏</strong>。</p>
<p data-nodeid="28951">因为通常情况下，如果一个对象不再有用，那么我们的垃圾回收器 GC，就应该把这部分内存给清理掉。这样的话，就可以让这部分内存后续重新分配到其他的地方去使用；否则，如果对象没有用，但一直不能被回收，这样的垃圾对象如果积累的越来越多，则会导致我们可用的内存越来越少，最后发生内存不够用的 OOM 错误。</p>
<p data-nodeid="28952">下面我们来分析一下，在 ThreadLocal 中这样的内存泄漏是如何发生的。</p>
<h4 data-nodeid="28953">Key 的泄漏</h4>
<p data-nodeid="28954">在上一讲中，我们分析了 ThreadLocal 的内部结构，知道了每一个 Thread 都有一个 ThreadLocal.ThreadLocalMap 这样的类型变量，该变量的名字叫作 threadLocals。线程在访问了 ThreadLocal 之后，都会在它的 ThreadLocalMap 里面的 Entry 中去维护该 ThreadLocal 变量与具体实例的映射。</p>
<p data-nodeid="28955">我们可能会在业务代码中执行了 ThreadLocal instance = null 操作，想清理掉这个 ThreadLocal 实例，但是假设我们在 ThreadLocalMap 的 Entry 中强引用了 ThreadLocal 实例，那么，虽然在业务代码中把 ThreadLocal 实例置为了 null，但是在 Thread 类中依然有这个引用链的存在。</p>
<p data-nodeid="28956">GC 在垃圾回收的时候会进行可达性分析，它会发现这个 ThreadLocal 对象依然是可达的，所以对于这个 ThreadLocal 对象不会进行垃圾回收，这样的话就造成了内存泄漏的情况。</p>
<p data-nodeid="28957">JDK 开发者考虑到了这一点，所以 ThreadLocalMap 中的 Entry 继承了 WeakReference 弱引用，代码如下所示：</p>
<pre class="lang-java" data-nodeid="28958"><code data-language="java"><span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Entry</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">WeakReference</span>&lt;<span class="hljs-title">ThreadLocal</span>&lt;?&gt;&gt;&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">/**&nbsp;The&nbsp;value&nbsp;associated&nbsp;with&nbsp;this&nbsp;ThreadLocal.&nbsp;*/</span>
&nbsp;&nbsp;&nbsp;&nbsp;Object&nbsp;value;

&nbsp;&nbsp;&nbsp;&nbsp;Entry(ThreadLocal&lt;?&gt;&nbsp;k,&nbsp;Object&nbsp;v)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">super</span>(k);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value&nbsp;=&nbsp;v;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="28959">可以看到，这个 Entry 是 extends WeakReference。弱引用的特点是，如果这个对象只被弱引用关联，而没有任何强引用关联，那么这个对象就可以被回收，所以弱引用不会阻止 GC。因此，这个弱引用的机制就避免了 ThreadLocal 的内存泄露问题。</p>
<p data-nodeid="28960">这就是为什么 Entry 的 key 要使用弱引用的原因。</p>
<h4 data-nodeid="28961">Value 的泄漏</h4>
<p data-nodeid="28962">可是，如果我们继续研究的话会发现，虽然 ThreadLocalMap 的每个 Entry 都是一个对 key 的弱引用，但是这个 Entry 包含了一个对 value 的强引用，还是刚才那段代码：</p>
<pre class="lang-java" data-nodeid="28963"><code data-language="java"><span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Entry</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">WeakReference</span>&lt;<span class="hljs-title">ThreadLocal</span>&lt;?&gt;&gt;&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">/**&nbsp;The&nbsp;value&nbsp;associated&nbsp;with&nbsp;this&nbsp;ThreadLocal.&nbsp;*/</span>
&nbsp;&nbsp;&nbsp;&nbsp;Object&nbsp;value;


&nbsp;&nbsp;&nbsp;&nbsp;Entry(ThreadLocal&lt;?&gt;&nbsp;k,&nbsp;Object&nbsp;v)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">super</span>(k);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value&nbsp;=&nbsp;v;
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="28964">可以看到，value = v 这行代码就代表了强引用的发生。</p>
<p data-nodeid="28965">正常情况下，当线程终止，key 所对应的 value 是可以被正常垃圾回收的，因为没有任何强引用存在了。但是有时线程的生命周期是很长的，如果线程迟迟不会终止，那么可能 ThreadLocal 以及它所对应的 value 早就不再有用了。在这种情况下，我们应该保证它们都能够被正常的回收。</p>
<p data-nodeid="28966">为了更好地分析这个问题，我们用下面这张图来看一下具体的引用链路（实线代表强引用，虚线代表弱引用）：</p>
<p data-nodeid="28967"><img src="https://s0.lgstatic.com/i/image3/M01/68/C4/Cgq2xl5Pld-AHFhJAADLtGXmSxc833.png" alt="" data-nodeid="29003"></p>
<p data-nodeid="28968">可以看到，左侧是引用栈，栈里面有一个 ThreadLocal 的引用和一个线程的引用，右侧是我们的堆，在堆中是对象的实例。</p>
<p data-nodeid="28969">我们重点看一下下面这条链路：Thread Ref → Current Thread → ThreadLocalMap → Entry → Value → 可能泄漏的value实例。</p>
<p data-nodeid="28970">这条链路是随着线程的存在而一直存在的，如果线程执行耗时任务而不停止，那么当垃圾回收进行可达性分析的时候，这个 Value 就是可达的，所以不会被回收。但是与此同时可能我们已经完成了业务逻辑处理，不再需要这个 Value 了，此时也就发生了内存泄漏问题。</p>
<p data-nodeid="28971">JDK 同样也考虑到了这个问题，在执行 ThreadLocal 的 set、remove、rehash 等方法时，它都会扫描 key 为 null 的 Entry，如果发现某个 Entry 的 key 为 null，则代表它所对应的 value 也没有作用了，所以它就会把对应的 value 置为 null，这样，value 对象就可以被正常回收了。</p>
<p data-nodeid="28972">但是假设 ThreadLocal 已经不被使用了，那么实际上 set、remove、rehash 方法也不会被调用，与此同时，如果这个线程又一直存活、不终止的话，那么刚才的那个调用链就一直存在，也就导致了 value 的内存泄漏。</p>
<h4 data-nodeid="28973">如何避免内存泄露</h4>
<p data-nodeid="28974">分析完这个问题之后，该如何解决呢？解决方法就是我们本课时的标题：调用 ThreadLocal 的 remove 方法。调用这个方法就可以删除对应的 value 对象，可以避免内存泄漏。</p>
<p data-nodeid="28975">我们来看一下 remove 方法的源码：</p>
<pre class="lang-java" data-nodeid="28976"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">remove</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocalMap&nbsp;m&nbsp;=&nbsp;getMap(Thread.currentThread());
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(m&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;m.remove(<span class="hljs-keyword">this</span>);
}
</code></pre>
<p data-nodeid="28977">可以看出，它是先获取到 ThreadLocalMap 这个引用的，并且调用了它的 remove 方法。这里的 remove 方法可以把 key 所对应的 value 给清理掉，这样一来，value 就可以被 GC 回收了。</p>
<p data-nodeid="29152">所以，在使用完了 ThreadLocal 之后，我们应该手动去调用它的 remove 方法，目的是防止内存泄漏的发生。</p>
<blockquote data-nodeid="30146">
<p data-nodeid="30147" class="te-preview-highlight">注：第一张图片和引用链相关内容，参考自<a href="https://blog.csdn.net/zhongxiangbo/article/details/70859181" data-nodeid="30151">https://blog.csdn.net/zhongxiangbo/article/details/70859181</a>，但未能找到更原始的出处，原作者若看到，欢迎联系，将进行标注。</p>
</blockquote>

---

### 精选评论

##### *锋：
> 很清楚，比以前看的清楚太多了

##### *海：
> 有个问题，如果threadLocal在使用的过程中key被垃圾回收了，后面如果再用到threadLocal是不是会出问题？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 除非主动置为null，否则ThreadLocal会有强引用，不会自动回收。

##### **明：
> 老师,您讲假如 ThreadLocalMap中的entry对ThreadLocal是强引用,这时把ThreadLocal设置为null,因为引用链的存在,无法回收ThreadLocal对象,所以设置entry对ThreadLocal弱引用,这样就可以进行垃圾回收,但是,您又讲了entry是对value强引用的,可以利用set方法,将ThreadLocal为null所对应的Value设置为null,从而可以进行回收,为什么这里对强引用Value设置为null就可以直接回收了?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这样的话，value就没有强引用了。

##### **洲：
> 老师讲的很好，学习到了很多

##### **龙：
> 明白了

##### **金：
> 写的太好了。

##### *翼：
> 真的挺感谢老师的，感觉一定要买这个专栏！发现底下一些同学的存在的问题也是我思考的一些问题，也有一些结论了比如说ThreadLocal的key在使用中被回收，后续使用会不会有问题；首先要认知到:ThreadLocalMap是一个键值对的map，那么我们要获取对应的值，那就一定要使用对应的ThreadLocal来进行获取。那在我们的程序中，就必须强引用这个ThreadLocal，才能取得对应的value值。如果我们主程序没有对应的ThreadLocal，那么也同时代表着我们不在使用该键值；就可以被GC进行回收了！还有就是为什么ThreadLocal会导致内存泄漏的问题：首先ThreadLocal是解决并发问题的一种解决方式，其核心思想是将共享资源私有化的思想，从根本上解决线程问题。因为没有共享资源，自然没有多个线程对其进行竞争。但同时也将该资源的作用域提升至整个线程的生命周期。当前我们较为广泛的使用线程池，里面对线程进行复用，那么线程的生命周期就不单单是处理完该任务就进行释放了，从而导致了严重的内存泄漏问题。

