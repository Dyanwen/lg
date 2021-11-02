<p data-nodeid="120223">在上节课我们提到过，Scrapy-Redis 库已经为我们提供了 Scrapy 分布式的队列、调度器、去重等功能，其 GitHub 地址为： <a href="https://github.com/rmax/scrapy-redis" data-nodeid="120227">https://github.com/rmax/scrapy-redis</a>。</p>



<p data-nodeid="119764">本节课我们深入掌握利用 Redis 实现 Scrapy 分布式的方法，并深入了解 Scrapy-Redis 的原理。</p>
<h3 data-nodeid="119765">获取源码</h3>
<p data-nodeid="119766">可以把源码克隆下来，执行如下命令：</p>
<pre class="lang-java" data-nodeid="120680"><code data-language="java">git clone https:<span class="hljs-comment">//github.com/rmax/scrapy-redis.git </span>
</code></pre>


<p data-nodeid="119768">核心源码在 scrapy-redis/src/scrapy_redis 目录下。</p>
<h3 data-nodeid="119769">爬取队列</h3>
<p data-nodeid="119770">我们从爬取队列入手，来看看它的具体实现。源码文件为 queue.py，它包含了三个队列的实现，首先它实现了一个父类 Base，提供一些基本方法和属性，如下所示：</p>
<pre class="lang-dart" data-nodeid="123088"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Base</span>(<span class="hljs-title">object</span>): 
 &nbsp; &nbsp;"""<span class="hljs-title">Per</span>-<span class="hljs-title">spider</span> <span class="hljs-title">base</span> <span class="hljs-title">queue</span> <span class="hljs-title">class</span>""" 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-title">self</span>, <span class="hljs-title">server</span>, <span class="hljs-title">spider</span>, <span class="hljs-title">key</span>, <span class="hljs-title">serializer</span>=<span class="hljs-title">None</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">serializer</span> <span class="hljs-title">is</span> <span class="hljs-title">None</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">serializer</span> = <span class="hljs-title">picklecompat</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">not</span> <span class="hljs-title">hasattr</span>(<span class="hljs-title">serializer</span>, '<span class="hljs-title">loads</span>'): 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">raise</span> <span class="hljs-title">TypeError</span>("<span class="hljs-title">serializer</span> <span class="hljs-title">does</span> <span class="hljs-title">not</span> <span class="hljs-title">implement</span> '<span class="hljs-title">loads</span>' <span class="hljs-title">function</span>: % <span class="hljs-title">r</span>" 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;% <span class="hljs-title">serializer</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">not</span> <span class="hljs-title">hasattr</span>(<span class="hljs-title">serializer</span>, '<span class="hljs-title">dumps</span>'): 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">raise</span> <span class="hljs-title">TypeError</span>("<span class="hljs-title">serializer</span> '% <span class="hljs-title">s</span>' <span class="hljs-title">does</span> <span class="hljs-title">not</span> <span class="hljs-title">implement</span> '<span class="hljs-title">dumps</span>' <span class="hljs-title">function</span>: % <span class="hljs-title">r</span>" 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;% <span class="hljs-title">serializer</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">server</span> = <span class="hljs-title">server</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">spider</span> = <span class="hljs-title">spider</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">key</span> = <span class="hljs-title">key</span> % </span>{<span class="hljs-string">'spider'</span>: spider.name} 
 &nbsp; &nbsp; &nbsp; &nbsp;self.serializer = serializer 
​ 
 &nbsp; &nbsp;def _encode_request(self, request): 
 &nbsp; &nbsp; &nbsp; &nbsp;obj = request_to_dict(request, self.spider) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> self.serializer.dumps(obj) 
​ 
 &nbsp; &nbsp;def _decode_request(self, encoded_request): 
 &nbsp; &nbsp; &nbsp; &nbsp;obj = self.serializer.loads(encoded_request) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> request_from_dict(obj, self.spider) 
​ 
 &nbsp; &nbsp;def __len__(self): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Return the length of the queue"""</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;raise NotImplementedError 
​ 
 &nbsp; &nbsp;def push(self, request): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Push a request"""</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;raise NotImplementedError 
​ 
 &nbsp; &nbsp;def pop(self, timeout=<span class="hljs-number">0</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Pop a request"""</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;raise NotImplementedError 
​ 
 &nbsp; &nbsp;def clear(self): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Clear queue/stack"""</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;self.server.delete(self.key) 
</code></pre>








<p data-nodeid="123389">首先看一下 _encode_request 和 _decode_request 方法，因为我们需要把一个  Request 对象存储到数据库中，但数据库无法直接存储对象，所以需要将 Request 序列转化成字符串再存储，而这两个方法分别是序列化和反序列化的操作，利用 pickle 库来实现，一般在调用 push 将 Request 存入数据库时会调用 _encode_request 方法进行序列化，在调用 pop 取出 Request 的时候会调用 _decode_request 进行反序列化。</p>
<p data-nodeid="126432" class="">在父类中 __len__、push 和 pop 方法都是未实现的，会直接抛出 NotImplementedError，因此是不能直接使用这个类的，必须实现一个子类来重写这三个方法，而不同的子类就会有不同的实现，也就有着不同的功能。</p>



<p data-nodeid="119773">接下来我们就需要定义一些子类来继承 Base 类，并重写这几个方法，那在源码中就有三个子类的实现，它们分别是 FifoQueue、PriorityQueue、LifoQueue，我们分别来看下它们的实现原理。</p>
<p data-nodeid="119774">首先是 FifoQueue：</p>
<pre class="lang-dart" data-nodeid="125821"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FifoQueue</span>(<span class="hljs-title">Base</span>): 
 &nbsp; &nbsp;"""<span class="hljs-title">Per</span>-<span class="hljs-title">spider</span> <span class="hljs-title">FIFO</span> <span class="hljs-title">queue</span>""" 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__len__</span>(<span class="hljs-title">self</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Return</span> <span class="hljs-title">the</span> <span class="hljs-title">length</span> <span class="hljs-title">of</span> <span class="hljs-title">the</span> <span class="hljs-title">queue</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">llen</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">push</span>(<span class="hljs-title">self</span>, <span class="hljs-title">request</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Push</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">lpush</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, <span class="hljs-title">self</span>.<span class="hljs-title">_encode_request</span>(<span class="hljs-title">request</span>)) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">pop</span>(<span class="hljs-title">self</span>, <span class="hljs-title">timeout</span>=0): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Pop</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">timeout</span> &gt; 0: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">brpop</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, <span class="hljs-title">timeout</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">isinstance</span>(<span class="hljs-title">data</span>, <span class="hljs-title">tuple</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">data</span>[1] 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">else</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">rpop</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">data</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">_decode_request</span>(<span class="hljs-title">data</span>) 
</span></code></pre>








<p data-nodeid="127848">可以看到这个类继承了 Base 类，并重写了 __len__、push、pop 这三个方法，在这三个方法中都是对 server 对象的操作，而 server 对象就是一个 Redis 连接对象，我们可以直接调用其操作 Redis 的方法对数据库进行操作，可以看到这里的操作方法有 llen、lpush、rpop 等，这就代表此爬取队列是使用的 Redis 的列表，序列化后的 Request 会被存入列表中，就是列表的其中一个元素，__len__ 方法是获取列表的长度，push 方法中调用了 lpush 操作，这代表从列表左侧存入数据，pop 方法中调用了 rpop 操作，这代表从列表右侧取出数据。</p>
<p data-nodeid="127849">所以 Request 在列表中的存取顺序是左侧进、右侧出，所以这是有序的进出，即先进先出，英文叫作 First Input First Output，也被简称为 FIFO，而此类的名称就叫作 FifoQueue。</p>





<p data-nodeid="119777">另外还有一个与之相反的实现类，叫作 LifoQueue，实现如下：</p>
<pre class="lang-dart" data-nodeid="130269"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LifoQueue</span>(<span class="hljs-title">Base</span>): 
 &nbsp; &nbsp;"""<span class="hljs-title">Per</span>-<span class="hljs-title">spider</span> <span class="hljs-title">LIFO</span> <span class="hljs-title">queue</span>.""" 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__len__</span>(<span class="hljs-title">self</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Return</span> <span class="hljs-title">the</span> <span class="hljs-title">length</span> <span class="hljs-title">of</span> <span class="hljs-title">the</span> <span class="hljs-title">stack</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">llen</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">push</span>(<span class="hljs-title">self</span>, <span class="hljs-title">request</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Push</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">lpush</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, <span class="hljs-title">self</span>.<span class="hljs-title">_encode_request</span>(<span class="hljs-title">request</span>)) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">pop</span>(<span class="hljs-title">self</span>, <span class="hljs-title">timeout</span>=0): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Pop</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">timeout</span> &gt; 0: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">blpop</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, <span class="hljs-title">timeout</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">isinstance</span>(<span class="hljs-title">data</span>, <span class="hljs-title">tuple</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">data</span>[1] 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">else</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">lpop</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>) 
​ 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">data</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">_decode_request</span>(<span class="hljs-title">data</span>) 
</span></code></pre>








<p data-nodeid="130570">与 FifoQueue 不同的就是它的 pop 方法，在这里使用的是 lpop 操作，也就是从左侧出，而 push 方法依然是使用的 lpush 操作，是从左侧入。那么这样达到的效果就是先进后出、后进先出，英文叫作 Last In First Out，简称为 LIFO，而此类名称就叫作 LifoQueue。同时这个存取方式类似栈的操作，所以其实也可以称作 StackQueue。</p>
<p data-nodeid="130571">另外在源码中还有一个子类实现，叫作 PriorityQueue，顾名思义，它叫作优先级队列，实现如下：</p>

<pre class="lang-dart" data-nodeid="131777"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PriorityQueue</span>(<span class="hljs-title">Base</span>): 
 &nbsp; &nbsp;"""<span class="hljs-title">Per</span>-<span class="hljs-title">spider</span> <span class="hljs-title">priority</span> <span class="hljs-title">queue</span> <span class="hljs-title">abstraction</span> <span class="hljs-title">using</span> <span class="hljs-title">redis</span>' <span class="hljs-title">sorted</span> <span class="hljs-title">set</span>""" 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__len__</span>(<span class="hljs-title">self</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Return</span> <span class="hljs-title">the</span> <span class="hljs-title">length</span> <span class="hljs-title">of</span> <span class="hljs-title">the</span> <span class="hljs-title">queue</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">zcard</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">push</span>(<span class="hljs-title">self</span>, <span class="hljs-title">request</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Push</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span>""" 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">data</span> = <span class="hljs-title">self</span>.<span class="hljs-title">_encode_request</span>(<span class="hljs-title">request</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">score</span> = -<span class="hljs-title">request</span>.<span class="hljs-title">priority</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">execute_command</span>('<span class="hljs-title">ZADD</span>', <span class="hljs-title">self</span>.<span class="hljs-title">key</span>, <span class="hljs-title">score</span>, <span class="hljs-title">data</span>) 
​ 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">pop</span>(<span class="hljs-title">self</span>, <span class="hljs-title">timeout</span>=0): 
 &nbsp; &nbsp; &nbsp; &nbsp;""" 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Pop</span> <span class="hljs-title">a</span> <span class="hljs-title">request</span> 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">timeout</span> <span class="hljs-title">not</span> <span class="hljs-title">support</span> <span class="hljs-title">in</span> <span class="hljs-title">this</span> <span class="hljs-title">queue</span> <span class="hljs-title">class</span> 
 &nbsp; &nbsp; &nbsp;  """ 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">pipe</span> = <span class="hljs-title">self</span>.<span class="hljs-title">server</span>.<span class="hljs-title">pipeline</span>() 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">pipe</span>.<span class="hljs-title">multi</span>() 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">pipe</span>.<span class="hljs-title">zrange</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, 0, 0).<span class="hljs-title">zremrangebyrank</span>(<span class="hljs-title">self</span>.<span class="hljs-title">key</span>, 0, 0) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">results</span>, <span class="hljs-title">count</span> = <span class="hljs-title">pipe</span>.<span class="hljs-title">execute</span>() 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">if</span> <span class="hljs-title">results</span>: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">return</span> <span class="hljs-title">self</span>.<span class="hljs-title">_decode_request</span>(<span class="hljs-title">results</span>[0]) 
</span></code></pre>




<p data-nodeid="133340">在这里我们可以看到 __len__、push、pop 方法中使用了 server 对象的 zcard、zadd、zrange 操作，可以知道这里使用的存储结果是有序集合 Sorted Set，在这个集合中每个元素都可以设置一个分数，那么这个分数就代表优先级。</p>
<p data-nodeid="133341">在 __len__ 方法里调用了 zcard 操作，返回的就是有序集合的大小，也就是爬取队列的长度，在 push 方法中调用了 zadd 操作，就是向集合中添加元素，这里的分数指定成 Request 的优先级的相反数，因为分数低的会排在集合的前面，所以这里高优先级的 Request 就会存在集合的最前面。pop 方法是首先调用了 zrange 操作取出了集合的第一个元素，因为最高优先级的 Request 会存在集合最前面，所以第一个元素就是最高优先级的 Request，然后再调用 zremrangebyrank 操作将这个元素删除，这样就完成了取出并删除的操作。</p>





<p data-nodeid="119782">此队列是默认使用的队列，也就是爬取队列默认是使用有序集合来存储的。</p>
<h3 data-nodeid="119783">去重过滤</h3>
<p data-nodeid="119784">前面说过 Scrapy 的去重是利用集合来实现的，而在 Scrapy 分布式中的去重就需要利用共享的集合，那么这里使用的就是 Redis 中的集合数据结构。我们来看看去重类是怎样实现的，源码文件是 dupefilter.py，其内实现了一个 RFPDupeFilter 类，如下所示：</p>
<pre class="lang-dart" data-nodeid="133654"><code data-language="dart"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RFPDupeFilter</span>(<span class="hljs-title">BaseDupeFilter</span>): 
 &nbsp; &nbsp;"""<span class="hljs-title">Redis</span>-<span class="hljs-title">based</span> <span class="hljs-title">request</span> <span class="hljs-title">duplicates</span> <span class="hljs-title">filter</span>. 
 &nbsp;  <span class="hljs-title">This</span> <span class="hljs-title">class</span> <span class="hljs-title">can</span> <span class="hljs-title">also</span> <span class="hljs-title">be</span> <span class="hljs-title">used</span> <span class="hljs-title">with</span> <span class="hljs-title">default</span> <span class="hljs-title">Scrapy</span>'<span class="hljs-title">s</span> <span class="hljs-title">scheduler</span>. 
 &nbsp;  """ 
 &nbsp; &nbsp;<span class="hljs-title">logger</span> = <span class="hljs-title">logger</span> 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">__init__</span>(<span class="hljs-title">self</span>, <span class="hljs-title">server</span>, <span class="hljs-title">key</span>, <span class="hljs-title">debug</span>=<span class="hljs-title">False</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Initialize</span> <span class="hljs-title">the</span> <span class="hljs-title">duplicates</span> <span class="hljs-title">filter</span>. 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Parameters</span> 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">server</span> : <span class="hljs-title">redis</span>.<span class="hljs-title">StrictRedis</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-title">The</span> <span class="hljs-title">redis</span> <span class="hljs-title">server</span> <span class="hljs-title">instance</span>. 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">key</span> : <span class="hljs-title">str</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Redis</span> <span class="hljs-title">key</span> <span class="hljs-title">Where</span> <span class="hljs-title">to</span> <span class="hljs-title">store</span> <span class="hljs-title">fingerprints</span>. 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">debug</span> : <span class="hljs-title">bool</span>, <span class="hljs-title">optional</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Whether</span> <span class="hljs-title">to</span> <span class="hljs-title">log</span> <span class="hljs-title">filtered</span> <span class="hljs-title">requests</span>. 
 &nbsp; &nbsp; &nbsp;  """ 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">server</span> = <span class="hljs-title">server</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">key</span> = <span class="hljs-title">key</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">debug</span> = <span class="hljs-title">debug</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">self</span>.<span class="hljs-title">logdupes</span> = <span class="hljs-title">True</span> 
​ 
 &nbsp; &nbsp;@<span class="hljs-title">classmethod</span> 
 &nbsp; &nbsp;<span class="hljs-title">def</span> <span class="hljs-title">from_settings</span>(<span class="hljs-title">cls</span>, <span class="hljs-title">settings</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;"""<span class="hljs-title">Returns</span> <span class="hljs-title">an</span> <span class="hljs-title">instance</span> <span class="hljs-title">from</span> <span class="hljs-title">given</span> <span class="hljs-title">settings</span>. 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">This</span> <span class="hljs-title">uses</span> <span class="hljs-title">by</span> <span class="hljs-title">default</span> <span class="hljs-title">the</span> <span class="hljs-title">key</span> ``<span class="hljs-title">dupefilter</span>:&lt;<span class="hljs-title">timestamp</span>&gt;``. <span class="hljs-title">When</span> <span class="hljs-title">using</span> <span class="hljs-title">the</span> 
 &nbsp; &nbsp; &nbsp;  ``<span class="hljs-title">scrapy_redis</span>.<span class="hljs-title">scheduler</span>.<span class="hljs-title">Scheduler</span>`` <span class="hljs-title">class</span>, <span class="hljs-title">this</span> <span class="hljs-title">method</span> <span class="hljs-title">is</span> <span class="hljs-title">not</span> <span class="hljs-title">used</span> <span class="hljs-title">as</span> 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">it</span> <span class="hljs-title">needs</span> <span class="hljs-title">to</span> <span class="hljs-title">pass</span> <span class="hljs-title">the</span> <span class="hljs-title">spider</span> <span class="hljs-title">name</span> <span class="hljs-title">in</span> <span class="hljs-title">the</span> <span class="hljs-title">key</span>. 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Parameters</span> 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">settings</span> : <span class="hljs-title">scrapy</span>.<span class="hljs-title">settings</span>.<span class="hljs-title">Settings</span> 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">Returns</span> 
 &nbsp; &nbsp; &nbsp;  ------- 
 &nbsp; &nbsp; &nbsp;  <span class="hljs-title">RFPDupeFilter</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  <span class="hljs-title">A</span> <span class="hljs-title">RFPDupeFilter</span> <span class="hljs-title">instance</span>. 
 &nbsp; &nbsp; &nbsp;  """ 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">server</span> = <span class="hljs-title">get_redis_from_settings</span>(<span class="hljs-title">settings</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-title">key</span> = <span class="hljs-title">defaults</span>.<span class="hljs-title">DUPEFILTER_KEY</span> % </span>{<span class="hljs-string">'timestamp'</span>: <span class="hljs-built_in">int</span>(time.time())} 
 &nbsp; &nbsp; &nbsp; &nbsp;debug = settings.getbool(<span class="hljs-string">'DUPEFILTER_DEBUG'</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> cls(server, key=key, debug=debug) 
​ 
 &nbsp; &nbsp;<span class="hljs-meta">@classmethod</span> 
 &nbsp; &nbsp;def from_crawler(cls, crawler): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Returns instance from crawler. 
 &nbsp; &nbsp; &nbsp;  Parameters 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  crawler : scrapy.crawler.Crawler 
 &nbsp; &nbsp; &nbsp;  Returns 
 &nbsp; &nbsp; &nbsp;  ------- 
 &nbsp; &nbsp; &nbsp;  RFPDupeFilter 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;  Instance of RFPDupeFilter. 
 &nbsp; &nbsp; &nbsp;  """</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> cls.from_settings(crawler.settings) 
​ 
 &nbsp; &nbsp;def request_seen(self, request): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Returns True if request was already seen. 
 &nbsp; &nbsp; &nbsp;  Parameters 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  request : scrapy.http.Request 
 &nbsp; &nbsp; &nbsp;  Returns 
 &nbsp; &nbsp; &nbsp;  ------- 
 &nbsp; &nbsp; &nbsp;  bool 
 &nbsp; &nbsp; &nbsp;  """</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;fp = self.request_fingerprint(request) 
 &nbsp; &nbsp; &nbsp; &nbsp;added = self.server.sadd(self.key, fp) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> added == <span class="hljs-number">0</span> 
​ 
 &nbsp; &nbsp;def request_fingerprint(self, request): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Returns a fingerprint for a given request. 
 &nbsp; &nbsp; &nbsp;  Parameters 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  request : scrapy.http.Request 
​ 
 &nbsp; &nbsp; &nbsp;  Returns 
 &nbsp; &nbsp; &nbsp;  ------- 
 &nbsp; &nbsp; &nbsp;  str 
​ 
 &nbsp; &nbsp; &nbsp;  """</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> request_fingerprint(request) 
​ 
 &nbsp; &nbsp;def close(self, reason=<span class="hljs-string">''</span>): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Delete data on close. Called by Scrapy's scheduler. 
 &nbsp; &nbsp; &nbsp;  Parameters 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  reason : str, optional 
 &nbsp; &nbsp; &nbsp;  """</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;self.clear() 
​ 
 &nbsp; &nbsp;def clear(self): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Clears fingerprints data."""</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;self.server.delete(self.key) 
​ 
 &nbsp; &nbsp;def log(self, request, spider): 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"""Logs given request. 
 &nbsp; &nbsp; &nbsp;  Parameters 
 &nbsp; &nbsp; &nbsp;  ---------- 
 &nbsp; &nbsp; &nbsp;  request : scrapy.http.Request 
 &nbsp; &nbsp; &nbsp;  spider : scrapy.spiders.Spider 
 &nbsp; &nbsp; &nbsp;  """</span> 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> self.debug: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;msg = <span class="hljs-string">"Filtered duplicate request: %(request) s"</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;self.logger.debug(msg, {<span class="hljs-string">'request'</span>: request}, extra={<span class="hljs-string">'spider'</span>: spider}) 
 &nbsp; &nbsp; &nbsp; &nbsp;elif self.logdupes: 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;msg = (<span class="hljs-string">"Filtered duplicate request %(request) s"</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"- no more duplicates will be shown"</span> 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"(see DUPEFILTER_DEBUG to show all duplicates)"</span>) 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;self.logger.debug(msg, {<span class="hljs-string">'request'</span>: request}, extra={<span class="hljs-string">'spider'</span>: spider}) 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;self.logdupes = False 
</code></pre>

<p data-nodeid="133955">这里同样实现了一个 request_seen 方法，和 Scrapy 中的 request_seen 方法实现极其类似。不过这里集合使用的是 server 对象的 sadd 操作，也就是集合不再是一个简单数据结构了，而是直接换成了数据库的存储方式。</p>
<p data-nodeid="133956">鉴别重复的方式还是使用指纹，指纹同样是依靠 request_fingerprint 方法来获取的。获取指纹之后就直接向集合添加指纹，如果添加成功，说明这个指纹原本不存在于集合中，返回值 1。代码中最后的返回结果是判定添加结果是否为 0，如果刚才的返回值为 1，那这个判定结果就是 False，也就是不重复，否则判定为重复。</p>

<p data-nodeid="119787">这样我们就成功利用 Redis 的集合完成了指纹的记录和重复的验证。</p>
<h3 data-nodeid="119788">调度器</h3>
<p data-nodeid="119789">Scrapy-Redis 还帮我们实现了配合 Queue、DupeFilter 使用的调度器 Scheduler，源文件名称是 scheduler.py。我们可以指定一些配置，如 SCHEDULER_FLUSH_ON_START 即是否在爬取开始的时候清空爬取队列，SCHEDULER_PERSIST 即是否在爬取结束后保持爬取队列不清除。我们可以在 settings.py 里自由配置，而此调度器很好地实现了对接。</p>
<p data-nodeid="119790">接下来我们看看两个核心的存取方法，实现如下所示：</p>
<pre class="lang-dart" data-nodeid="134265"><code data-language="dart">def enqueue_request(self, request): 
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> not request.dont_filter and self.df.request_seen(request): 
 &nbsp; &nbsp; &nbsp; &nbsp;self.df.log(request, self.spider) 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> False 
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> self.stats: 
 &nbsp; &nbsp; &nbsp; &nbsp;self.stats.inc_value(<span class="hljs-string">'scheduler/enqueued/redis'</span>, spider=self.spider) 
 &nbsp; &nbsp;self.queue.push(request) 
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> True 
​ 
def next_request(self): 
 &nbsp; &nbsp;block_pop_timeout = self.idle_before_close 
 &nbsp; &nbsp;request = self.queue.pop(block_pop_timeout) 
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> request and self.stats: 
 &nbsp; &nbsp; &nbsp; &nbsp;self.stats.inc_value(<span class="hljs-string">'scheduler/dequeued/redis'</span>, spider=self.spider) 
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> request 
</code></pre>

<p data-nodeid="119792">enqueue_request 可以向队列中添加 Request，核心操作就是调用 Queue 的 push 操作，还有一些统计和日志操作。next_request 就是从队列中取 Request，核心操作就是调用 Queue 的 pop 操作，此时如果队列中还有 Request，则 Request 会直接取出来，爬取继续，否则如果队列为空，爬取则会重新开始。</p>
<h3 data-nodeid="119793">总结</h3>
<p data-nodeid="119794">那么到现在为止我们就把之前所说的三个分布式的问题解决了，总结如下：</p>
<ul data-nodeid="119795">
<li data-nodeid="119796">
<p data-nodeid="119797">爬取队列的实现，在这里提供了三种队列，使用了 Redis 的列表或有序集合来维护。</p>
</li>
<li data-nodeid="119798">
<p data-nodeid="119799">去重的实现，使用了 Redis 的集合来保存 Request 的指纹以提供重复过滤。</p>
</li>
<li data-nodeid="119800">
<p data-nodeid="119801">中断后重新爬取的实现，中断后 Redis 的队列没有清空，再次启动时调度器的 next_request 会从队列中取到下一个 Request，继续爬取。</p>
</li>
</ul>
<h3 data-nodeid="119802">结语</h3>
<p data-nodeid="119803">以上内容便是 Scrapy-Redis 的核心源码解析。Scrapy-Redis 中还提供了 Spider、Item Pipeline 的实现，不过它们并不是必须使用的。</p>
<p data-nodeid="119804">在下一节，我们会将 Scrapy-Redis 集成到之前所实现的 Scrapy 项目中，实现多台主机协同爬取。</p>

---

### 精选评论


