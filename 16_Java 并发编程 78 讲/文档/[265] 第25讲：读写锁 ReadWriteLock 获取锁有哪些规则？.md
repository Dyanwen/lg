<p data-nodeid="932" class="">在本课时我们主要讲解读写锁 ReadWriteLock 获取锁有哪些规则呢？</p>
<p data-nodeid="933">在没有读写锁之前，我们假设使用普通的 ReentrantLock，那么虽然我们保证了线程安全，但是也浪费了一定的资源，因为如果多个读操作同时进行，其实并没有线程安全问题，我们可以允许让多个读操作并行，以便提高程序效率。</p>
<p data-nodeid="934">但是写操作不是线程安全的，如果多个线程同时写，或者在写的同时进行读操作，便会造成线程安全问题。</p>
<p data-nodeid="935">我们的读写锁就解决了这样的问题，它设定了一套规则，既可以保证多个线程同时读的效率，同时又可以保证有写入操作时的线程安全。</p>
<p data-nodeid="936">整体思路是它有两把锁，第 1 把锁是写锁，获得写锁之后，既可以读数据又可以修改数据，而第 2 把锁是读锁，获得读锁之后，只能查看数据，不能修改数据。读锁可以被多个线程同时持有，所以多个线程可以同时查看数据。</p>
<p data-nodeid="937">在读的地方合理使用读锁，在写的地方合理使用写锁，灵活控制，可以提高程序的执行效率。</p>
<h3 data-nodeid="938">读写锁的获取规则</h3>
<p data-nodeid="939">我们在使用读写锁时遵守下面的获取规则：</p>
<ol data-nodeid="2103">
<li data-nodeid="2104">
<p data-nodeid="2105">当一个线程已经占有了读锁，那么其他线程如果想要申请读锁，可以申请成功；</p>
</li>
<li data-nodeid="2106">
<p data-nodeid="2107" class="">当一个线程已经占有了读锁，而且有其他线程想要申请获取写锁的话，是不能申请成功的，因为读写互斥；</p>
</li>
<li data-nodeid="2108">
<p data-nodeid="2109" class="te-preview-highlight">当一个线程已经占有了写锁，那么此时其他线程无论是想申请读锁还是写锁，都无法申请成功。</p>
</li>
</ol>














<p data-nodeid="947">所以我们用一句话总结：要么是一个或多个线程同时有读锁，要么是一个线程有写锁，但是两者不会同时出现。也可以总结为：读读共享、其他都互斥（写写互斥、读写互斥、写读互斥）。</p>
<h3 data-nodeid="948">使用案例</h3>
<p data-nodeid="949">下面我们举个例子来应用读写锁，ReentrantReadWriteLock 是 ReadWriteLock 的实现类，最主要的有两个方法：readLock() 和 writeLock() 用来获取读锁和写锁。</p>
<p data-nodeid="950">代码如下：</p>
<pre class="lang-java" data-nodeid="951"><code data-language="java"><span class="hljs-comment">/**
 * 描述：     演示读写锁用法
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReadWriteLockDemo</span> </span>{

    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ReentrantReadWriteLock reentrantReadWriteLock = <span class="hljs-keyword">new</span> ReentrantReadWriteLock(
            <span class="hljs-keyword">false</span>);
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ReentrantReadWriteLock.ReadLock readLock = reentrantReadWriteLock
            .readLock();
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> ReentrantReadWriteLock.WriteLock writeLock = reentrantReadWriteLock
            .writeLock();

    <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">read</span><span class="hljs-params">()</span> </span>{
        readLock.lock();
        <span class="hljs-keyword">try</span> {
            System.out.println(Thread.currentThread().getName() + <span class="hljs-string">"得到读锁，正在读取"</span>);
            Thread.sleep(<span class="hljs-number">500</span>);
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        } <span class="hljs-keyword">finally</span> {
            System.out.println(Thread.currentThread().getName() + <span class="hljs-string">"释放读锁"</span>);
            readLock.unlock();
        }
    }

    <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">write</span><span class="hljs-params">()</span> </span>{
        writeLock.lock();
        <span class="hljs-keyword">try</span> {
            System.out.println(Thread.currentThread().getName() + <span class="hljs-string">"得到写锁，正在写入"</span>);
            Thread.sleep(<span class="hljs-number">500</span>);
        } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
        } <span class="hljs-keyword">finally</span> {
            System.out.println(Thread.currentThread().getName() + <span class="hljs-string">"释放写锁"</span>);
            writeLock.unlock();
        }
    }

    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> InterruptedException </span>{
        <span class="hljs-keyword">new</span> Thread(() -&gt; read()).start();
        <span class="hljs-keyword">new</span> Thread(() -&gt; read()).start();
        <span class="hljs-keyword">new</span> Thread(() -&gt; write()).start();
        <span class="hljs-keyword">new</span> Thread(() -&gt; write()).start();
    }
}
</code></pre>
<p data-nodeid="952">程序的运行结果是：</p>
<pre class="lang-java" data-nodeid="953"><code data-language="java">Thread-<span class="hljs-number">0</span>得到读锁，正在读取
Thread-<span class="hljs-number">1</span>得到读锁，正在读取
Thread-<span class="hljs-number">0</span>释放读锁
Thread-<span class="hljs-number">1</span>释放读锁
Thread-<span class="hljs-number">2</span>得到写锁，正在写入
Thread-<span class="hljs-number">2</span>释放写锁
Thread-<span class="hljs-number">3</span>得到写锁，正在写入
Thread-<span class="hljs-number">3</span>释放写锁
</code></pre>
<p data-nodeid="954">可以看出，读锁可以同时被多个线程获得，而写锁不能。</p>
<h3 data-nodeid="955">读写锁适用场合</h3>
<p data-nodeid="956" class="">最后我们来看下读写锁的适用场合，相比于 ReentrantLock 适用于一般场合，ReadWriteLock 适用于读多写少的情况，合理使用可以进一步提高并发效率。</p>

---

### 精选评论

##### **6167：
> 对于为什么要对读加锁有点晕了，望老师解惑

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 读本身是线程安全的，加读锁，主要是为了让写锁感知到，在有人读取的时候，不要同时写入。

##### *帅：
> 读数据不会有线程安全问题为什么要加锁呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 加读锁，不是因为读取有安全问题，而是为了让程序感知到，此时正在读，不要同时来写入。

##### **强：
> 我的理解：如果都是"读"这一操作，本身没有线程安全问题；但是既存在对共享变量的读，又存在写操作得话，假如读操作不加锁，读的过程中，共享变量是允许其他线程修改的，那么就可能发生问题，读到的不是原期望值。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，理解的很对

##### **宏：
> 其实我的想法和留言一样，为什么要对读加锁，mysql中如果不是事务的读也是不用加锁的吧。读就读副本不行吗。读操作排队后面有写操作的化，读出来以后就不是最新的了，不如直接让大家直接读当前的快照性能更快了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用读写锁可以保证读到的最新的数据，你说的是CopyOnWrite思想，这两种思想适用于不同的业务场景。

##### **猛：
> 学习知识要善于思考，思考，在思考。

##### *广：
> 对于楼上说为什么需要加读锁，可以这样理解。假如存在两个变量a和b，都可以被并发修改，并且a和b之间有一定的约束关系（比如b=2*a），修改之后也满足这个关系。假如没有读锁，执行完a++之后，b+=2还没执行完，就可能读到a和b的值，此时a和b不满足约束关系，显然违反了我们设定的逻辑，加了读写锁锁定a和b的读取就不会出现这个问题

##### **诚：
> 读加锁，主要是读写互斥呀😀

