<p>在上一课时中，我们说明了原子类和 synchronized 关键字都可以用来保证线程安全，在本课时中，我们首先分别用原子类和 synchronized 关键字来解决一个经典的线程安全问题，给出具体的代码对比，然后再分析它们背后的区别。</p> <h3>代码对比</h3> <p>首先，原始的线程不安全的情况的代码如下所示：
</p><pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Lesson42</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Runnable</span> </span>{

    <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> value = <span class="hljs-number">0</span>;

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
        Runnable runnable = <span class="hljs-keyword">new</span> Lesson42();
        Thread thread1 = <span class="hljs-keyword">new</span> Thread(runnable);
        Thread thread2 = <span class="hljs-keyword">new</span> Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(value);
    }

    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">10000</span>; i++) {
            value++;
        }
    }
}
</code></pre>
<p>在代码中我们新建了一个 value 变量，并且在两个线程中对它进行同时的自加操作，每个线程加 10000 次，然后我们用 join 来确保它们都执行完毕，最后打印出最终的数值。</p> <p>因为 value++ 不是一个原子操作，所以上面这段代码是线程不安全的（具体分析详见第 6 讲），所以代码的运行结果会小于 20000，例如会输出 14611 等各种数字。</p> <p>我们首先给出<strong>方法一</strong>，也就是用原子类来解决这个问题，代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Lesson42Atomic</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Runnable</span> </span>{

    <span class="hljs-keyword">static</span> AtomicInteger atomicInteger = <span class="hljs-keyword">new</span> AtomicInteger();

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
        Runnable runnable = <span class="hljs-keyword">new</span> Lesson42Atomic();
        Thread thread1 = <span class="hljs-keyword">new</span> Thread(runnable);
        Thread thread2 = <span class="hljs-keyword">new</span> Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(atomicInteger.get());
    }

    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">10000</span>; i++) {
            atomicInteger.incrementAndGet();
        }
    }
}
</code></pre>
<p>用原子类之后，我们的计数变量就不再是一个普通的&nbsp;int&nbsp;变量了，而是 AtomicInteger 类型的对象，并且自加操作也变成了 incrementAndGet 法。由于原子类可以确保每一次的自加操作都是具备原子性的，所以这段程序是线程安全的，所以以上程序的运行结果会始终等于 20000。</p> <p>下面我们给出<strong>方法二</strong>，我们用 synchronized 来解决这个问题，代码如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Lesson42Syn</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Runnable</span> </span>{

    <span class="hljs-keyword">static</span> <span class="hljs-keyword">int</span> value = <span class="hljs-number">0</span>;

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
        Runnable runnable = <span class="hljs-keyword">new</span> Lesson42Syn();
        Thread thread1 = <span class="hljs-keyword">new</span> Thread(runnable);
        Thread thread2 = <span class="hljs-keyword">new</span> Thread(runnable);
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        System.out.println(value);
    }

    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; <span class="hljs-number">10000</span>; i++) {
            <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
                value++;
            }
        }
    }
}

</code></pre>
<p>它与最开始的线程不安全的代码的区别在于，在 run 方法中加了&nbsp;synchronized 代码块，就可以非常轻松地解决这个问题，由于 synchronized 可以保证代码块内部的原子性，所以以上程序的运行结果也始终等于 20000，是线程安全的。</p> <h3>方案对比</h3> <p>下面我们就对这两种不同的方案进行分析。</p> <p>第一点，我们来看一下它们背后<strong>原理</strong>的不同。</p> <p>在第 21 课时中我们详细分析了&nbsp;synchronized 背后的 monitor 锁，也就是&nbsp;synchronized 原理，同步方法和同步代码块的背后原理会有少许差异，但总体思想是一致的：在执行同步代码之前，需要首先获取到 monitor 锁，执行完毕后，再释放锁。</p> <p>而我们在第 39 课时中介绍了原子类，它保证线程安全的原理是利用了 CAS 操作。从这一点上看，虽然原子类和&nbsp;synchronized 都能保证线程安全，但是其实现原理是大有不同的。</p> <p>第二点不同是<strong>使用范围</strong>的不同。</p> <p>对于原子类而言，它的使用范围是比较局限的。因为一个原子类仅仅是一个对象，不够灵活。而 synchronized 的使用范围要广泛得多。比如说 synchronized 既可以修饰一个方法，又可以修饰一段代码，相当于可以根据我们的需要，非常灵活地去控制它的应用范围。</p> <p>所以仅有少量的场景，例如计数器等场景，我们可以使用原子类。而在其他更多的场景下，如果原子类不适用，那么我们就可以考虑用 synchronized 来解决这个问题。</p> <p>第三个区别是<strong>粒度</strong>的区别。</p> <p>原子变量的粒度是比较小的，它可以把竞争范围缩小到变量级别。通常情况下，synchronized 锁的粒度都要大于原子变量的粒度。如果我们只把一行代码用 synchronized 给保护起来的话，有一点杀鸡焉用牛刀的感觉。</p> <p>第四点是它们<strong>性能</strong>的区别，同时也是悲观锁和乐观锁的区别。</p> <p>因为 synchronized 是一种典型的悲观锁，而原子类恰恰相反，它利用的是乐观锁。所以，我们在比较 synchronized 和 AtomicInteger 的时候，其实也就相当于比较了悲观锁和乐观锁的区别。</p> <p>从性能上来考虑的话，悲观锁的操作相对来讲是比较重量级的。因为 synchronized 在竞争激烈的情况下，会让拿不到锁的线程阻塞，而原子类是永远不会让线程阻塞的。不过，虽然 synchronized 会让线程阻塞，但是这并不代表它的性能就比原子类差。</p> <p>因为悲观锁的开销是固定的，也是一劳永逸的。随着时间的增加，这种开销并不会线性增长。</p> <p>而乐观锁虽然在短期内的开销不大，但是随着时间的增加，它的开销也是逐步上涨的。</p> <p>所以从性能的角度考虑，它们没有一个孰优孰劣的关系，而是要区分具体的使用场景。在竞争非常激烈的情况下，推荐使用 synchronized；而在竞争不激烈的情况下，使用原子类会得到更好的效果。</p> <p>值得注意的是，synchronized 的性能随着 JDK 的升级，也得到了不断的优化。synchronized 会从无锁升级到偏向锁，再升级到轻量级锁，最后才会升级到让线程阻塞的重量级锁。因此synchronized 在竞争不激烈的情况下，性能也是不错的，不需要“谈虎色变”。</p><p></p>

---

### 精选评论

##### **花大人：
> 写的太好了！！！！

##### **岭：
> 老师，悲观锁和乐观锁的开销指的具体是什么？有什么区别？是线程之间切换还是等待

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 悲观锁开销是一次性的获取锁的开销，可能涉及线程状态转换，而乐观锁的开销是一次次的尝试获取锁。

