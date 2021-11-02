<p>在本课时中我们主要讲解阻塞队列包含哪些常用的方法，以及 add，offer，put 等方法的区别。</p>
<p>在阻塞队列中有很多方法，而且它们都非常相似，所以非常有必要对这些类似的方法进行辨析，所以本课时会用分类的方式，和你一起，把阻塞队列中常见的方法进行梳理和讲解。</p>
<p>我们把 BlockingQueue 中最常用的和添加、删除相关的 8 个方法列出来，并且把它们分为三组，每组方法都和添加、移除元素相关。</p>
<p>这三组方法由于功能很类似，所以比较容易混淆。它们的区别仅在于特殊情况：当队列满了无法添加元素，或者是队列空了无法移除元素时，不同组的方法对于这种特殊情况会有不同的处理方式：</p>
<ol>
<li>抛出异常：add、remove、element</li>
<li>返回结果但不抛出异常：offer、poll、peek</li>
<li>阻塞：put、take</li>
</ol>
<h3>第一组：add、remove、element</h3>
<h4>add 方法</h4>
<p>add 方法是往队列里添加一个元素，如果队列满了，就会抛出异常来提示队列已满。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">addTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;BlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.add(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.add(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.add(<span class="hljs-number">1</span>);
}
</code></pre>
<p>在这段代码中，我们创建了一个容量为 2 的 BlockingQueue，并且尝试往里面放 3 个值，超过了容量上限，那么在添加第三个值的时候就会得到异常：</p>
<pre><code data-language="java" class="lang-java">Exception in thread <span class="hljs-string">"main"</span> java.lang.IllegalStateException:Queue full
</code></pre>
<h4>remove 方法</h4>
<p>remove 方法的作用是删除元素，如果我们删除的队列是空的，由于里面什么都没有，所以也无法删除任何元素，那么 remove 方法就会抛出异常。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">removeTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.add(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.add(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.remove();
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.remove();
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.remove();
}
</code></pre>
<p>在这段代码中，我们往一个容量为 2 的 BlockingQueue 里放入 2 个元素，并且删除 3 个元素。在删除前面两个元素的时候会正常执行，因为里面依然有元素存在，但是在删除第三个元素时，由于队列里面已经空了，所以便会抛出异常：</p>
<pre><code data-language="java" class="lang-java">Exception in thread <span class="hljs-string">"main"</span> java.util.NoSuchElementException
</code></pre>
<h4>element 方法</h4>
<p>element 方法是返回队列的头部节点，但是并不删除。和 remove 方法一样，如果我们用这个方法去操作一个空队列，想获取队列的头结点，可是由于队列是空的，我们什么都获取不到，会抛出和前面 remove 方法一样的异常：NoSuchElementException。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">elementTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.element();
}
</code></pre>
<p>我们新建了一个容量为 2 的 ArrayBlockingQueue，直接调用 element 方法，由于之前没有往里面添加元素，默认为空，那么会得到异常：</p>
<pre><code data-language="java" class="lang-java">Exception in thread <span class="hljs-string">"main"</span> java.util.NoSuchElementException
</code></pre>
<h3>第二组：offer、poll、peek</h3>
<p>实际上我们通常并不想看到第一组方法抛出的异常，这时我们可以优先采用第二组方法。第二组方法相比于第一组而言要友好一些，当发现队列满了无法添加，或者队列为空无法删除的时候，第二组方法会给一个提示，而不是抛出一个异常。</p>
<h4>offer 方法</h4>
<p>offer 方法用来插入一个元素，并用返回值来提示插入是否成功。如果添加成功会返回 true，而如果队列已经满了，此时继续调用 offer 方法的话，它不会抛出异常，只会返回一个错误提示：false。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">offerTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.offer(<span class="hljs-number">1</span>));
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.offer(<span class="hljs-number">1</span>));
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.offer(<span class="hljs-number">1</span>));
}
</code></pre>
<p>我们创建了一个容量为 2 的 ArrayBlockingQueue，并且调用了三次 offer方法尝试添加，每次都把返回值打印出来，运行结果如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">true</span>
<span class="hljs-keyword">true</span>
<span class="hljs-keyword">false</span>
</code></pre>
<p>可以看出，前面两次添加成功了，但是第三次添加的时候，已经超过了队列的最大容量，所以会返回 false，表明添加失败。</p>
<h4>poll 方法</h4>
<p>poll 方法和第一组的 remove 方法是对应的，作用也是移除并返回队列的头节点。但是如果当队列里面是空的，没有任何东西可以移除的时候，便会返回 null 作为提示。正因如此，我们是不允许往队列中插入 null 的，否则我们没有办法区分返回的 null 是一个提示还是一个真正的元素。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">pollTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">3</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.offer(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.offer(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;blockingQueue.offer(<span class="hljs-number">3</span>);
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.poll());
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.poll());
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.poll());
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.poll());
}
</code></pre>
<p>在这个代码中我们创建了一个容量为 3 的 ArrayBlockingQueue，并且先往里面放入 3 个元素，然后四次调用 poll 方法，运行结果如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-number">1</span>
<span class="hljs-number">2</span>
<span class="hljs-number">3</span>
<span class="hljs-keyword">null</span>
</code></pre>
<p>前面三次 poll 都运行成功了，并且返回了元素内容 1、2、3，是先进先出的顺序。第四次的 poll 方法返回 null，代表此时已经没有元素可以移除了。</p>
<h4>peek 方法</h4>
<p>peek 方法和第一组的 element 方法是对应的，意思是返回队列的头元素但并不删除。如果队列里面是空的，它便会返回 null 作为提示。示例代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">peekTest</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;ArrayBlockingQueue&lt;Integer&gt;&nbsp;blockingQueue&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;ArrayBlockingQueue&lt;Integer&gt;(<span class="hljs-number">2</span>);
&nbsp;&nbsp;&nbsp;&nbsp;System.out.println(blockingQueue.peek());
}
</code></pre>
<p>运行结果：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">null</span>
</code></pre>
<p>我们新建了一个空的 ArrayBlockingQueue，然后直接调用 peek，返回结果 null，代表此时并没有东西可以取出。</p>
<h4>带超时时间的 offer 和 poll</h4>
<p>第二组还有一些额外值得讲解的内容，offer 和 poll 都有带超时时间的重载方法。</p>
<pre><code data-language="java" class="lang-java">offer(E&nbsp;e,&nbsp;<span class="hljs-keyword">long</span>&nbsp;timeout,&nbsp;TimeUnit&nbsp;unit)
</code></pre>
<p>它有三个参数，分别是元素、超时时长和时间单位。通常情况下，这个方法会插入成功并返回 true；如果队列满了导致插入不成功，在调用带超时时间重载方法的 offer 的时候，则会等待指定的超时时间，如果时间到了依然没有插入成功，就会返回 false。</p>
<pre><code data-language="java" class="lang-java">poll(<span class="hljs-keyword">long</span>&nbsp;timeout,&nbsp;TimeUnit&nbsp;unit)
</code></pre>
<p>带时间参数的 poll 方法和 offer 类似：如果能够移除，便会立刻返回这个节点的内容；如果队列是空的就会进行等待，等待时间正是我们指定的时间，直到超时时间到了，如果队列里依然没有元素可供移除，便会返回 null 作为提示。</p>
<h3>第三组：put、take</h3>
<p>第三组是我们比较熟悉的、阻塞队列最大特色的 put 和 take 方法，我们复习一下 34 课时里对于 put 和 take 方法的讲解。</p>
<h4>put 方法</h4>
<p>put 方法的作用是插入元素。通常在队列没满的时候是正常的插入，但是如果队列已满就无法继续插入，这时它既不会立刻返回 false 也不会抛出异常，而是让插入的线程陷入阻塞状态，直到队列里有了空闲空间，此时队列就会让之前的线程解除阻塞状态，并把刚才那个元素添加进去。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/62/7E/Cgq2xl4lhcOAYPonAAB1UtAAltk655.png" alt=""></p>
<h4>take 方法</h4>
<p>take 方法的作用是获取并移除队列的头结点。通常在队列里有数据的时候会正常取出数据并删除；但是如果执行 take 的时候队列里无数据，则阻塞，直到队列里有数据；一旦队列里有数据了，就会立刻解除阻塞状态，并且取到数据。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/62/7E/Cgq2xl4lhdWAWOz8AABp-t8dt_8107.png" alt=""></p>
<h3>总结</h3>
<p>以上就是本课时的内容，本课时我们讲解了阻塞队列中常见的方法并且把它们分为了三组，每一组都有各自的特点。第一组的特点是在无法正常执行的情况下抛出异常；第二组的特点是在无法正常执行的情况下不抛出异常，但会用返回值提示运行失败；第三组的特点是在遇到特殊情况时让线程陷入阻塞状态，等到可以运行再继续执行。</p>
<p>我们用表格把上面 8 种方法总结如下：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/62/7E/CgpOIF4lheGALDjnAAHFyzrSvqU109.png" alt=""><br>
有了这个表格之后，我们就可以非常清晰地理清这 8 个方法之间的关系了，课后你可以仔细对比表格以加深印象。</p>

---

### 精选评论

##### **雄：
> 课程真的不错，顺带说一句，万丈高楼平地起

##### **才：
> 为什么在BlockingQueue的源码中没有发现element方法呢？而在Queue中发现了这个方法，而且在BlockingQueue的源码中有10个方法，而不是你所说的8个方法，没有peek和element方法，反而在Queue中发现了有这两个方法，对不上呀，我猜想是不是Queue中的6个方法和BlockingQueue方法中的take和put方法？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; BlockingQueue继承了Queue。

##### *京：
> ArrayBlockingQueue 继承了AbstractQueue抽象类，AbstractQueue实现了Queue接口中的element 方法 ，然鹅ArrayBlockingQueue 并没有element 方法

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; ArrayBlockingQueue有element方法，但是你在ArrayBlockingQueue类里看不到，因为ArrayBlockingQueue已经继承了AbstractQueue，里面已经有了。

##### **寿：
> 透彻，多刷几遍。

##### Edward：
> 看了DelayedWorkQueue基于siftDown的remove(Object x)方法，返回true和false。还以为和上述不一致，仔细一看，是remove()会抛异常，并在AbstractQueue中提供了实现。把最小堆复习了下...

