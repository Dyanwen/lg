<p>在本课时中，我将讲解&nbsp;CAS 的应用场景，什么时候会用到 CAS？</p> <h3>并发容器</h3> <p>Doug&nbsp;Lea 大神在 <strong>JUC</strong> 包中大量使用了&nbsp;CAS&nbsp;技术，该技术既能保证安全性，又不需要使用互斥锁，能大大提升工具类的性能。下面我将通过两个例子来展示 CAS 在并发容器中的使用情况。</p> <h4>案例一：ConcurrentHashMap</h4> <p>先来看看并发容器 ConcurrentHashMap 的例子，我们截取部分 putVal 方法的代码，如下所示：
</p><pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">final</span> V <span class="hljs-title">putVal</span><span class="hljs-params">(K key, V value, <span class="hljs-keyword">boolean</span> onlyIfAbsent)</span> </span>{
    <span class="hljs-keyword">if</span> (key == <span class="hljs-keyword">null</span> || value == <span class="hljs-keyword">null</span>) <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NullPointerException();
    <span class="hljs-keyword">int</span> hash = spread(key.hashCode());
    <span class="hljs-keyword">int</span> binCount = <span class="hljs-number">0</span>;
    <span class="hljs-keyword">for</span> (Node&lt;K,V&gt;[] tab = table;;) {
        Node&lt;K,V&gt; f; <span class="hljs-keyword">int</span> n, i, fh;
        <span class="hljs-keyword">if</span> (tab == <span class="hljs-keyword">null</span> || (n = tab.length) == <span class="hljs-number">0</span>)
            tab = initTable();
        <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> ((f = tabAt(tab, i = (n - <span class="hljs-number">1</span>) &amp; hash)) == <span class="hljs-keyword">null</span>) {
            <span class="hljs-keyword">if</span> (casTabAt(tab, i, <span class="hljs-keyword">null</span>,
                         <span class="hljs-keyword">new</span> Node&lt;K,V&gt;(hash, key, value, <span class="hljs-keyword">null</span>)))
                <span class="hljs-keyword">break</span>;                   <span class="hljs-comment">// no lock when adding to empty bin</span>
        }
    <span class="hljs-comment">//以下部分省略</span>
    ...
}
</code></pre>
<p>在第 10 行，有一个醒目的方法，它就是 “casTabAt”，这个方法名就带有 “CAS”，可以猜测它一定是和 CAS 密不可分了，下面给出 casTabAt 方法的代码实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> &lt;K,V&gt; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">casTabAt</span><span class="hljs-params">(Node&lt;K,V&gt;[] tab, <span class="hljs-keyword">int</span> i,
                                    Node&lt;K,V&gt; c, Node&lt;K,V&gt; v)</span> </span>{
    <span class="hljs-keyword">return</span> U.compareAndSwapObject(tab, ((<span class="hljs-keyword">long</span>)i &lt;&lt; ASHIFT) + ABASE, c, v);
}
</code></pre>
<p>该方法里面只有一行代码，即调用变量 U 的 <strong>compareAndSwapObject</strong> 的方法，那么，这个变量 U 是什么类型的呢？U 的定义是：</p> <pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">final</span>&nbsp;sun.misc.Unsafe&nbsp;U </code></pre> <p>可以看出，U 是 <strong>Unsafe</strong> 类型的，Unsafe 类包含 compareAndSwapInt、compareAndSwapLong、compareAndSwapObject 等和 CAS 密切相关的 native 层的方法，其底层正是利用 CPU 对 CAS 指令的支持实现的。</p> <p>上面介绍的 casTabAt 方法，不仅被用在了 ConcurrentHashMap 的 putVal 方法中，还被用在了 merge、compute、computeIfAbsent、transfer 等重要的方法中，所以 ConcurrentHashMap 对于 CAS 的应用是比较广泛的。</p> <h4>案例二：ConcurrentLinkedQueue</h4> <p>接下来，我们来看并发容器的第二个案例。非阻塞并发队列 ConcurrentLinkedQueue 的 offer 方法里也有 CAS 的身影，offer 方法的代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">offer</span><span class="hljs-params">(E e)</span> </span>{
    checkNotNull(e);
    <span class="hljs-keyword">final</span> Node&lt;E&gt; newNode = <span class="hljs-keyword">new</span> Node&lt;E&gt;(e);

    <span class="hljs-keyword">for</span> (Node&lt;E&gt; t = tail, p = t;;) {
        Node&lt;E&gt; q = p.next;
        <span class="hljs-keyword">if</span> (q == <span class="hljs-keyword">null</span>) {
            <span class="hljs-keyword">if</span> (p.casNext(<span class="hljs-keyword">null</span>, newNode)) {
                <span class="hljs-keyword">if</span> (p != t) 
                    casTail(t, newNode); 
                <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
            }
        }
        <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (p == q)
            p = (t != (t = tail)) ? t : head;
        <span class="hljs-keyword">else</span>
            p = (p != t &amp;&amp; t != (t = tail)) ? t : q;
    }
}
</code></pre>
<p>可以看出，在 offer 方法中，有一个 for 循环，这是一个死循环，在第 8 行有一个与 CAS 相关的方法，是 <strong>casNext</strong> 方法，用于更新节点。那么如果执行 p 的 casNext 方法失败的话，casNext 会返回 false，那么显然代码会继续在 for 循环中进行下一次的尝试。所以在这里也可以很明显的看出 ConcurrentLinkedQueue 的 offer 方法使用到了 CAS。</p> <p>以上就是 CAS 在并发容器中应用的两个例子，我们再来看一看 CAS 在数据库中有哪些应用。</p> <h3>数据库</h3> <p>在我们的数据库中，也存在对乐观锁和 CAS 思想的应用。在更新数据时，我们可以利用&nbsp;version&nbsp;字段在数据库中实现乐观锁和 CAS 操作，而在获取和修改数据时都不需要加悲观锁。</p> <p><strong>具体思路</strong>如下：当我们获取完数据，并计算完毕，准备更新数据时，会检查现在的版本号与之前获取数据时的版本号是否一致，如果一致就说明在计算期间数据没有被更新过，可以直接更新本次数据；如果版本号不一致，则说明计算期间已经有其他线程修改过这个数据了，那就可以选择重新获取数据，重新计算，然后再次尝试更新数据。</p> <p>假设取出数据的时候&nbsp;version&nbsp;版本为&nbsp;1，相应的&nbsp;SQL&nbsp;语句示例如下所示：</p> <pre><code data-language="java" class="lang-java">UPDATE&nbsp;student &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SET &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name&nbsp;=&nbsp;‘小王’, &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;version&nbsp;=&nbsp;<span class="hljs-number">2</span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;WHERE&nbsp;&nbsp;id&nbsp;=&nbsp;<span class="hljs-number">10</span> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AND&nbsp;version&nbsp;=&nbsp;<span class="hljs-number">1</span> </code></pre> <p>这样一来就可以用&nbsp;CAS 的思想去实现本次的更新操作，它会先去比较&nbsp;version&nbsp;是不是最开始获取到的 1，如果和初始值相同才去进行 name 字段的修改，同时也要把 version 的值加一。</p> <h3>原子类</h3> <p>在原子类中，例如&nbsp;AtomicInteger，也使用了&nbsp;CAS，原子类的内容我们在第&nbsp;39&nbsp;课时中已经具体分析过了，现在我们复习一下和&nbsp;CAS&nbsp;相关的重点内容，也就是&nbsp;AtomicInteger&nbsp;的&nbsp;getAndAdd&nbsp;方法，该方法代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getAndAdd</span><span class="hljs-params">(<span class="hljs-keyword">int</span> delta)</span> </span>{    
    <span class="hljs-keyword">return</span> unsafe.getAndAddInt(<span class="hljs-keyword">this</span>, valueOffset, delta);
}
</code></pre>
<p>从上面的三行代码中可以看到，return&nbsp;的内容是 Unsafe 的 getAndAddInt 方法的执行结果，接下来我们来看一下 getAndAddInt 方法的具体实现，代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getAndAddInt</span><span class="hljs-params">(Object var1, <span class="hljs-keyword">long</span> var2, <span class="hljs-keyword">int</span> var4)</span> </span>{
    <span class="hljs-keyword">int</span> var5;
    <span class="hljs-keyword">do</span> {
        var5 = <span class="hljs-keyword">this</span>.getIntVolatile(var1, var2);
    } <span class="hljs-keyword">while</span>(!<span class="hljs-keyword">this</span>.compareAndSwapInt(var1, var2, var5, var5 + var4));
    <span class="hljs-keyword">return</span> var5;
}
</code></pre>
<p>在这里，我们看到上述方法中有对&nbsp;var5 的赋值，调用了&nbsp;unsafe&nbsp;的 getIntVolatile(var1, var2) 方法，这是一个&nbsp;native&nbsp;方法，作用是获取变量&nbsp;var1&nbsp;中偏移量 var2 处的值。这里传入&nbsp;var1&nbsp;的是 AtomicInteger 对象的引用，而&nbsp;var2&nbsp;就是 AtomicInteger&nbsp;里面所存储的数值（也就是 value）的偏移量&nbsp;valueOffset，所以此时得到的 var5 实际上代表当前时刻下的原子类中存储的数值。</p> <p>接下来重点来了，我们看到有一个&nbsp;<strong>compareAndSwapInt</strong>&nbsp;方法，这里会传入多个参数，分别是&nbsp;var1、var2、 var5、var5 + var4，其实它们代表&nbsp;object、offset、expectedValue&nbsp;和&nbsp;newValue。</p> <ul> <li>第一个参数 object 就是将要修改的对象，传入的是 this，也就是 atomicInteger 这个对象本身；</li> <li>第二个参数是 offset，也就是偏移量，借助它就可以获取到 value 的数值；</li> <li>第三个参数 expectedValue，代表“期望值”，传入的是刚才获取到的 var5；</li> <li>而最后一个参数 newValue 是希望修改为的新值 ，等于之前取到的数值 var5 再加上 var4，而 var4 就是我们之前所传入的 delta，delta 就是我们希望原子类所改变的数值，比如可以传入 +1，也可以传入 -1。</li> </ul> <p>所以 compareAndSwapInt&nbsp;方法的作用就是，判断如果现在原子类里 value 的值和之前获取到的 var5 相等的话，那么就把计算出来的 var5 + var4 给更新上去，所以说这行代码就实现了&nbsp;CAS&nbsp;的过程。</p> <p>一旦 CAS 操作成功，就会退出这个 while 循环，但是也有可能操作失败。如果操作失败就意味着在获取到 var5 之后，并且在 CAS 操作之前，value 的数值已经发生变化了，证明有其他线程修改过这个变量。</p> <p>这样一来，就会再次执行循环体里面的代码，重新获取 var5 的值，也就是获取最新的原子变量的数值，并且再次利用 CAS 去尝试更新，直到更新成功为止，所以这是一个死循环。</p> <p>我们总结一下，Unsafe 的 getAndAddInt 方法是通过<strong>循环 + CAS</strong> 的方式来实现的，在此过程中，它会通过 compareAndSwapInt 方法来尝试更新 value 的值，如果更新失败就重新获取，然后再次尝试更新，直到更新成功。</p> <h3>总结</h3> <p>在本课时中，我们讲解了&nbsp;CAS&nbsp;的应用场景。在<strong>并发容器</strong>、<strong>数据库</strong>以及<strong>原子类</strong>中都有很多和 CAS 相关的代码，所以 CAS 有着广泛的应用场景，你需要清楚的了解什么情况下会用到 CAS。</p><p></p>

---

### 精选评论


