<p data-nodeid="25634" class="">本课时我们主要分析一下在 Thread 中多个 ThreadLocal 是怎么存储的。</p>
<h3 data-nodeid="25635">Thread、 ThreadLocal 及 ThreadLocalMap 三者之间的关系</h3>
<p data-nodeid="25636">在讲解本课时之前，先要搞清楚 Thread、 ThreadLocal 及 ThreadLocalMap 三者之间的关系。我们用最直观、最容易理解的图画的方式来看看它们三者的关系：<br>
<img src="https://s0.lgstatic.com/i/image3/M01/67/E8/Cgq2xl5M5a6ADeCKAABC52ZxZCk238.png" alt="" data-nodeid="25680"></p>
<p data-nodeid="25637">我们看到最左下角的 Thread 1，这是一个线程，它的箭头指向了 &nbsp;ThreadLocalMap 1，其要表达的意思是，每个 Thread 对象中都持有一个 ThreadLocalMap 类型的成员变量，在这里 Thread 1 所拥有的成员变量就是 ThreadLocalMap 1。</p>
<p data-nodeid="25638">而这个 ThreadLocalMap 自身类似于是一个 Map，里面会有一个个 key value 形式的键值对。那么我们就来看一下它的 key 和 value 分别是什么。可以看到这个表格的左侧是 ThreadLocal 1、ThreadLocal 2…… ThreadLocal n，能看出这里的 key 就是 ThreadLocal 的引用。</p>
<p data-nodeid="25639">而在表格的右侧是一个一个的 value，这就是我们希望 ThreadLocal 存储的内容，例如 user 对象等。</p>
<p data-nodeid="25640">这里需要重点看到它们的数量对应关系：一个 Thread 里面只有一个ThreadLocalMap ，而在一个 ThreadLocalMap 里面却可以有很多的 ThreadLocal，每一个 ThreadLocal 都对应一个 value。因为一个 Thread 是可以调用多个 ThreadLocal 的，所以 Thread 内部就采用了 ThreadLocalMap 这样 Map 的数据结构来存放 ThreadLocal 和 value。</p>
<p data-nodeid="25641">通过这张图片，我们就可以搞清楚 Thread、 ThreadLocal 及 ThreadLocalMap 三者在宏观上的关系了。</p>
<h3 data-nodeid="25642">源码分析</h3>
<p data-nodeid="25643">知道了它们的关系之后，我们再来进行源码分析，来进一步地看到它们内部的实现。</p>
<h4 data-nodeid="25644">get 方法</h4>
<p data-nodeid="25645">首先我们来看一下 get 方法，源码如下所示：</p>
<pre class="lang-java" data-nodeid="25646"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;T&nbsp;<span class="hljs-title">get</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//获取到当前线程</span>
&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;t&nbsp;=&nbsp;Thread.currentThread();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//获取到当前线程内的&nbsp;ThreadLocalMap&nbsp;对象，每个线程内都有一个&nbsp;ThreadLocalMap&nbsp;对象</span>
&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocalMap&nbsp;map&nbsp;=&nbsp;getMap(t);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(map&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//获取&nbsp;ThreadLocalMap&nbsp;中的&nbsp;Entry&nbsp;对象并拿到&nbsp;Value</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocalMap.Entry&nbsp;e&nbsp;=&nbsp;map.getEntry(<span class="hljs-keyword">this</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(e&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@SuppressWarnings("unchecked")</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;T&nbsp;result&nbsp;=&nbsp;(T)e.value;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//如果线程内之前没创建过&nbsp;ThreadLocalMap，就创建</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;setInitialValue();
}
</code></pre>
<p data-nodeid="25647">这是 ThreadLocal 的 get 方法，可以看出它利用了 Thread.currentThread 来获取当前线程的引用，并且把这个引用传入到了 getMap 方法里面，来拿到当前线程的 ThreadLocalMap。</p>
<p data-nodeid="25648">然后就是一个 if ( map != null ) 条件语句，那我们先来看看 if (map == null) 的情况，如果 map == null，则说明之前这个线程中没有创建过 ThreadLocalMap，于是就去调用 setInitialValue 来创建；如果 map != null，我们就应该通过 this 这个引用（也就是当前的 ThreadLocal 对象的引用）来获取它所对应的 Entry，同时再通过这个 Entry 拿到里面的 value，最终作为结果返回。</p>
<p data-nodeid="25649">值得注意的是，这里的 ThreadLocalMap 是保存在线程 Thread 类中的，而不是保存在 ThreadLocal 中的。</p>
<h4 data-nodeid="25650">getMap 方法</h4>
<p data-nodeid="25651">下面我们来看一下 getMap 方法，源码如下所示：</p>
<pre class="lang-java" data-nodeid="25652"><code data-language="java"><span class="hljs-function">ThreadLocalMap&nbsp;<span class="hljs-title">getMap</span><span class="hljs-params">(Thread&nbsp;t)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;t.threadLocals;
}
</code></pre>
<p data-nodeid="25653">可以看到，这个方法很清楚地表明了 Thread 和 ThreadLocalMap 的关系，可以看出 ThreadLocalMap 是线程的一个成员变量。这个方法的作用就是获取到当前线程内的 ThreadLocalMap 对象，每个线程都有 ThreadLocalMap 对象，而这个对象的名字就叫作 threadLocals，初始值为 null，代码如下：</p>
<pre class="lang-java" data-nodeid="25654"><code data-language="java">ThreadLocal.ThreadLocalMap&nbsp;threadLocals&nbsp;=&nbsp;<span class="hljs-keyword">null</span>;
</code></pre>
<h4 data-nodeid="25655">set 方法</h4>
<p data-nodeid="25656">下面我们再来看一下 set 方法，源码如下所示：</p>
<pre class="lang-java" data-nodeid="25657"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">set</span><span class="hljs-params">(T&nbsp;value)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;Thread&nbsp;t&nbsp;=&nbsp;Thread.currentThread();
&nbsp;&nbsp;&nbsp;&nbsp;ThreadLocalMap&nbsp;map&nbsp;=&nbsp;getMap(t);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;(map&nbsp;!=&nbsp;<span class="hljs-keyword">null</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;map.set(<span class="hljs-keyword">this</span>,&nbsp;value);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;createMap(t,&nbsp;value);
}
</code></pre>
<p data-nodeid="25658">set 方法的作用是把我们想要存储的 value 给保存进去。可以看出，首先，它还是需要获取到当前线程的引用，并且利用这个引用来获取到&nbsp;ThreadLocalMap ；然后，如果 map == null 则去创建这个 map，而当 map != null 的时候就利用 map.set 方法，把 value 给 set 进去。</p>
<p data-nodeid="25659">可以看出，map.set(this, value) &nbsp;传入的这两个参数中，第一个参数是 this，就是当前 ThreadLocal 的引用，这也再次体现了，在&nbsp;ThreadLocalMap 中，它的 key 的类型是&nbsp;ThreadLocal；而第二个参数就是我们所传入的 value，这样一来就可以把这个键值对保存到&nbsp;ThreadLocalMap 中去了。</p>
<h4 data-nodeid="25660">ThreadLocalMap 类，也就是 Thread.threadLocals</h4>
<p data-nodeid="25661">下面我们来看一下&nbsp;ThreadLocalMap 这个类，下面这段代码截取自定义在 ThreadLocal 类中的 ThreadLocalMap 类：</p>
<pre class="lang-java" data-nodeid="25662"><code data-language="java"><span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">ThreadLocalMap</span>&nbsp;</span>{

&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Entry</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">WeakReference</span>&lt;<span class="hljs-title">ThreadLocal</span>&lt;?&gt;&gt;&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">/**&nbsp;The&nbsp;value&nbsp;associated&nbsp;with&nbsp;this&nbsp;ThreadLocal.&nbsp;*/</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Object&nbsp;value;


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Entry(ThreadLocal&lt;?&gt;&nbsp;k,&nbsp;Object&nbsp;v)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">super</span>(k);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value&nbsp;=&nbsp;v;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;Entry[]&nbsp;table;
<span class="hljs-comment">//...</span>
}
</code></pre>
<p data-nodeid="25663">ThreadLocalMap 类是每个线程 Thread 类里面的一个成员变量，其中最重要的就是截取出的这段代码中的 Entry 内部类。在 ThreadLocalMap 中会有一个 Entry 类型的数组，名字叫 table。我们可以把 Entry 理解为一个 map，其键值对为：</p>
<ul data-nodeid="25664">
<li data-nodeid="25665">
<p data-nodeid="25666">键，当前的 ThreadLocal；</p>
</li>
<li data-nodeid="25667">
<p data-nodeid="25668">值，实际需要存储的变量，比如 user 用户对象或者 simpleDateFormat 对象等。</p>
</li>
</ul>
<p data-nodeid="25669">ThreadLocalMap 既然类似于 Map，所以就和 HashMap 一样，也会有包括 set、get、rehash、resize 等一系列标准操作。但是，虽然思路和 HashMap 是类似的，但是具体实现会有一些不同。</p>
<p data-nodeid="25670">比如其中一个不同点就是，我们知道 HashMap 在面对 hash 冲突的时候，采用的是拉链法。它会先把对象 hash 到一个对应的格子中，如果有冲突就用链表的形式往下链，如下图所示：</p>
<p data-nodeid="25671"><img src="https://s0.lgstatic.com/i/image3/M01/67/E8/CgpOIF5M5mqAPY_GAABqhQqH5zw536.png" alt="" data-nodeid="25714"></p>
<p data-nodeid="25672">但是 ThreadLocalMap 解决 hash 冲突的方式是不一样的，它采用的是线性探测法。如果发生冲突，并不会用链表的形式往下链，而是会继续寻找下一个空的格子。这是 ThreadLocalMap 和 HashMap 在处理冲突时不一样的点。</p>
<p data-nodeid="25673">以上就是本节课的内容。</p>
<p data-nodeid="25890">在本节课中，我们主要分析了 Thread、 ThreadLocal 和 ThreadLocalMap 这三个非常重要的类的关系。用图画的方式表明了它们之间的关系：一个 Thread 有一个 ThreadLocalMap，而 ThreadLocalMap 的 key 就是一个个的 ThreadLocal，它们就是用这样的关系来存储并维护内容的。之后我们对于 ThreadLocal 的一些重要方法进行了源码分析。</p>
<blockquote data-nodeid="28502">
<p data-nodeid="28503" class="te-preview-highlight">注：第一张图片来自网络，未能找到原始出处，原作者若看到，欢迎联系，将进行标注。</p>
</blockquote>

---

### 精选评论

##### **翔：
> 老师，提个问题哈，既然key都是this，而map的key是不允许重复的，那一个线程中多个value怎么存？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一个线程如果要对应多个value，需要用多个ThreadLocal

##### **施：
> ThreadLocalMap的Entry是弱引用，会发生使用的时候已被回收吗讲师回复： 不会的。怎么不会呢？那为什么是弱引用？这个能违反JVM吗？不会的原因是？（我不懂，望指教）

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不违反JVM。当还有强引用的时候，不会被回收，只有弱引用的时候，会被回收。

##### **强：
> ThreadLocalMap的Entry是弱引用，会发生使用的时候已被回收吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的。

##### *罗：
> 线性探测时找不到空格子了怎么处理

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 会进行扩容。

