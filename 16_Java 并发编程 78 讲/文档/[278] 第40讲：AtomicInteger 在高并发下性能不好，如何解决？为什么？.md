<p data-nodeid="20863" class="">本课时我们主要讲解&nbsp;AtomicInteger 在高并发下性能不好，如何解决？以及为什么会出现这种情况？</p>
<p data-nodeid="20864">我们知道在&nbsp;JDK1.5 中新增了并发情况下使用的 Integer/Long&nbsp;所对应的原子类 AtomicInteger 和 AtomicLong。</p>
<p data-nodeid="20865">在并发的场景下，如果我们需要实现计数器，可以利用 AtomicInteger 和 AtomicLong，这样一来，就可以避免加锁和复杂的代码逻辑，有了它们之后，我们只需要执行对应的封装好的方法，例如对这两个变量进行原子的增操作或原子的减操作，就可以满足大部分业务场景的需求。</p>
<p data-nodeid="20866">不过，虽然它们很好用，但是如果你的业务场景是并发量很大的，那么你也会发现，这两个原子类实际上会有较大的性能问题，这是为什么呢？就让我们从一个例子看起。</p>
<h3 data-nodeid="20867">AtomicLong&nbsp;存在的问题</h3>
<p data-nodeid="20868">首先我们来看一段代码：</p>
<pre class="lang-java" data-nodeid="20869"><code data-language="java"><span class="hljs-comment">/**
*&nbsp;描述：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在16个线程下使用AtomicLong
*/</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">AtomicLongDemo</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AtomicLong&nbsp;counter&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;AtomicLong(<span class="hljs-number">0</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ExecutorService&nbsp;service&nbsp;=&nbsp;Executors.newFixedThreadPool(<span class="hljs-number">16</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;<span class="hljs-number">100</span>;&nbsp;i++)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;service.submit(<span class="hljs-keyword">new</span>&nbsp;Task(counter));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">2000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(counter.get());
&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Task</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">Runnable</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">final</span>&nbsp;AtomicLong&nbsp;counter;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">Task</span><span class="hljs-params">(AtomicLong&nbsp;counter)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.counter&nbsp;=&nbsp;counter;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counter.incrementAndGet();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="20870">在这段代码中可以看出，我们新建了一个原始值为 0 的&nbsp;AtomicLong。然后，有一个线程数为 16 的线程池，并且往这个线程池中添加了 100 次相同的一个任务。</p>
<p data-nodeid="20871">那我们往下看这个任务是什么。在下面的 Task 类中可以看到，这个任务实际上就是每一次去调用 AtomicLong 的 incrementAndGet 方法，相当于一次自加操作。这样一来，整个类的作用就是把这个原子类从 0 开始，添加 100 个任务，每个任务自加一次。</p>
<p data-nodeid="20872">这段代码的运行结果毫无疑问是 100，虽然是多线程并发访问，但是 AtomicLong 依然可以保证 incrementAndGet 操作的原子性，所以不会发生线程安全问题。</p>
<p data-nodeid="20873">不过如果我们深入一步去看内部情景的话，你可能会感到意外。我们把模型简化成只有两个线程在同时工作的并发场景，因为两个线程和更多个线程本质上是一样的。如图所示：</p>
<p data-nodeid="20874"><img src="https://s0.lgstatic.com/i/image6/M00/44/55/Cgp9HWC-6OWAOb6NAAC1SVqjdK8226.png" alt="1.png" data-nodeid="20912"></p>
<p data-nodeid="20875">我们可以看到在这个图中，每一个线程是运行在自己的 core 中的，并且它们都有一个本地内存是自己独用的。在本地内存下方，有两个&nbsp;CPU&nbsp;核心共用的共享内存。</p>
<p data-nodeid="20876">对于 AtomicLong 内部的 value 属性而言，也就是保存当前 AtomicLong 数值的属性，它是被 volatile 修饰的，所以它需要保证自身可见性。</p>
<p data-nodeid="20877">这样一来，每一次它的数值有变化的时候，它都需要进行 flush 和 refresh。比如说，如果开始时，ctr 的数值为 0 的话，那么如图所示，一旦&nbsp;&nbsp;core 1 把它改成 1 的话，它首先会在左侧把这个 1 的最新结果给 flush 到下方的共享内存。然后，再到右侧去往上 refresh 到核心 2 的本地内存。这样一来，对于核心 2 而言，它才能感知到这次变化。</p>
<p data-nodeid="20878">由于竞争很激烈，这样的 flush 和 refresh 操作耗费了很多资源，而且 CAS 也会经常失败。</p>
<h3 data-nodeid="20879">LongAdder&nbsp;带来的改进和原理</h3>
<p data-nodeid="20880">在 JDK&nbsp;8 中又新增了 LongAdder 这个类，这是一个针对 Long 类型的操作工具类。那么既然已经有了 AtomicLong，为何又要新增 LongAdder 这么一个类呢？</p>
<p data-nodeid="20881">我们同样是用一个例子来说明。下面这个例子和刚才的例子很相似，只不过我们把工具类从 &nbsp;AtomicLong 变成了 LongAdder。其他的不同之处还在于最终打印结果的时候，调用的方法从原来的 get 变成了现在的 sum 方法。而其他的逻辑都一样。</p>
<p data-nodeid="20882">我们来看一下使用&nbsp;LongAdder&nbsp;的代码示例：</p>
<pre class="lang-java" data-nodeid="20883"><code data-language="java"><span class="hljs-comment">/**
*&nbsp;描述：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在16个线程下使用LongAdder
*/</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">LongAdderDemo</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">main</span><span class="hljs-params">(String[]&nbsp;args)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;InterruptedException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LongAdder&nbsp;counter&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;LongAdder();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ExecutorService&nbsp;service&nbsp;=&nbsp;Executors.newFixedThreadPool(<span class="hljs-number">16</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;<span class="hljs-number">100</span>;&nbsp;i++)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;service.submit(<span class="hljs-keyword">new</span>&nbsp;Task(counter));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Thread.sleep(<span class="hljs-number">2000</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(counter.sum());
&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Task</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">Runnable</span>&nbsp;</span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">final</span>&nbsp;LongAdder&nbsp;counter;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-title">Task</span><span class="hljs-params">(LongAdder&nbsp;counter)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.counter&nbsp;=&nbsp;counter;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">run</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counter.increment();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="20884">代码的运行结果同样是 100，但是运行速度比刚才&nbsp;AtomicLong&nbsp;的实现要快。下面我们解释一下，为什么高并发下 LongAdder 比 AtomicLong 效率更高。</p>
<p data-nodeid="20885">因为 LongAdder 引入了分段累加的概念，内部一共有两个参数参与计数：第一个叫作 base，它是一个变量，第二个是 Cell[]&nbsp;，是一个数组。</p>
<p data-nodeid="20886">其中的 base&nbsp;是用在竞争不激烈的情况下的，可以直接把累加结果改到 base 变量上。</p>
<p data-nodeid="20887">那么，当竞争激烈的时候，就要用到我们的&nbsp;Cell[]&nbsp;数组了。一旦竞争激烈，各个线程会分散累加到自己所对应的那个 Cell[] 数组的某一个对象中，而不会大家共用同一个。</p>
<p data-nodeid="20888">这样一来，LongAdder 会把不同线程对应到不同的 Cell 上进行修改，降低了冲突的概率，这是一种分段的理念，提高了并发性，这就和 Java 7 的 ConcurrentHashMap 的 16 个 Segment 的思想类似。</p>
<p data-nodeid="20889">竞争激烈的时候，LongAdder 会通过计算出每个线程的 hash 值来给线程分配到不同的 Cell 上去，每个 Cell 相当于是一个独立的计数器，这样一来就不会和其他的计数器干扰，Cell 之间并不存在竞争关系，所以在自加的过程中，就大大减少了刚才的&nbsp;flush&nbsp;和&nbsp;refresh，以及降低了冲突的概率，这就是为什么&nbsp;LongAdder 的吞吐量比 AtomicLong 大的原因，本质是空间换时间，因为它有多个计数器同时在工作，所以占用的内存也要相对更大一些。</p>
<p data-nodeid="20890">那么 LongAdder 最终是如何实现多线程计数的呢？答案就在最后一步的求和 sum 方法，执行 LongAdder.sum() 的时候，会把各个线程里的 Cell 累计求和，并加上 base，形成最终的总和。代码如下：</p>
<pre class="lang-java" data-nodeid="20891"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;<span class="hljs-title">sum</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;Cell[]&nbsp;as&nbsp;=&nbsp;cells;&nbsp;Cell&nbsp;a;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">long</span>&nbsp;sum&nbsp;=&nbsp;base;
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(as&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;(<span class="hljs-keyword">int</span>&nbsp;i&nbsp;=&nbsp;<span class="hljs-number">0</span>;&nbsp;i&nbsp;&lt;&nbsp;as.length;&nbsp;++i)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;((a&nbsp;=&nbsp;as[i])&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sum&nbsp;+=&nbsp;a.value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;sum;
}
</code></pre>
<p data-nodeid="20892">在这个 sum 方法中可以看到，思路非常清晰。先取 base 的值，然后遍历所有 Cell，把每个 Cell 的值都加上去，形成最终的总和。由于在统计的时候并没有进行加锁操作，所以这里得出的 sum 不一定是完全准确的，因为有可能在计算 sum 的过程中 Cell 的值被修改了。</p>
<p data-nodeid="20893">那么我们已经了解了，为什么 AtomicLong 或者说 AtomicInteger 它在高并发下性能不好，也同时看到了性能更好的 LongAdder。下面我们就分析一下，对它们应该如何选择。</p>
<h3 data-nodeid="20894">如何选择</h3>
<p data-nodeid="20895">在低竞争的情况下，AtomicLong 和 LongAdder 这两个类具有相似的特征，吞吐量也是相似的，因为竞争不高。但是在竞争激烈的情况下，LongAdder 的预期吞吐量要高得多，经过试验，LongAdder 的吞吐量大约是 AtomicLong 的十倍，不过凡事总要付出代价，LongAdder 在保证高效的同时，也需要消耗更多的空间。</p>
<h3 data-nodeid="20896">AtomicLong&nbsp;可否被&nbsp;LongAdder&nbsp;替代？</h3>
<p data-nodeid="20897" class="">那么我们就要考虑了，有了更高效的&nbsp;LongAdder，那&nbsp;AtomicLong&nbsp;可否不使用了呢？是否凡是用到&nbsp;AtomicLong&nbsp;的地方，都可以用&nbsp;LongAdder&nbsp;替换掉呢？答案是不是的，这需要区分场景。</p>
<p data-nodeid="20898" class="">LongAdder 只提供了 add、increment 等简单的方法，适合的是统计求和计数的场景，场景比较单一，而 AtomicLong 还具有 compareAndSet 等高级方法，可以应对除了加减之外的更复杂的需要 CAS 的场景。</p>
<p data-nodeid="21113">结论：如果我们的场景仅仅是需要用到加和减操作的话，那么可以直接使用更高效的 LongAdder，但如果我们需要利用 CAS 比如 compareAndSet 等操作的话，就需要使用 AtomicLong 来完成。</p>
<blockquote data-nodeid="25110">
<p data-nodeid="25111" class="te-preview-highlight">“老师，LongAdder 既然最后在相加的时候可能不准确，那不也是线程不安全的么，为什么还要使用呢？”</p>
<p data-nodeid="25112" class="">答案在这里：<a href="https://www.cnblogs.com/thisiswhy/p/13176237.html" data-nodeid="25117">https://www.cnblogs.com/thisiswhy/p/13176237.html</a></p>
</blockquote>

---

### 精选评论

##### ccarlos：
> 学到干货了😀

##### *飞：
> 老师LongAdder既然最后在相加的时候可能不准确，那不也是线程不安全的么，为什么还要使用呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当在多线程的情况下对一个共享数据进行更新（写）操作，比如实现一些统计信息类的需求，LongAdder 的表现比它的老大哥 AtomicLong 表现的更好。在并发不高的时候，两个类都差不多。但是高并发时 LongAdder 的吞吐量明显高一点，它也占用更多的空间。这是一种空间换时间的思想。

因为它在多线程并发情况下，没有一个准确的返回值，所以当你需要根据返回值去搞事情的时候，你就要仔细思考思考，这个返回值你是要精准的，还是大概的统计类的数据就行。

比如说，如果你是用来做序号生成器，所以你需要一个准确的返回值，那么还是用 AtomicLong 更加合适。

如果你是用来做计数器，这种写多读少的场景。比如接口访问次数的统计类需求，不需要时时刻刻的返回一个准确的值，那就上 LongAdder 吧。

总之，AtomicLong 是可以保证每次都有准确值，而 LongAdder 是可以保证最终数据是准确的。高并发的场景下 LongAdder 的写性能比 AtomicLong 高。

##### *林：
> 学到了，清晰明了

##### *晨：
> 简洁精炼！

##### **林：
> LongAdder专注累加，功能单一，但是性能整体做到极致，最终一致性，不是实时一致性。如果需要时时刻刻准确，还是用AtomicInteger吧。另外LongAccumulate提供更多的思路，值得一看。

##### **豪：
> LongAdder性能好，但是不安全，高并发下也不选用的吧

