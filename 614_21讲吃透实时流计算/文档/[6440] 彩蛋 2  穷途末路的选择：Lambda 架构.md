<p data-nodeid="10406">在前面的课时中，我们都是在讨论实时计算的问题。但真实的世界里，很多事情都不尽人愿。有时候，因为算法复杂度过高、数据量过大，我们并不能通过直接的实时计算，获得想要的结果。比如，二度关联图谱计算以及一些复杂的统计学习模型或机器学习模型训练等。在这种情况下，我们该如何制定出一个可以真实落地的系统架构方案呢？</p>
<p data-nodeid="10407">这个时候，我们就需要用到压箱底的 Lambda 架构了。</p>
<h3 data-nodeid="11216" class="">Lambda 架构</h3>

<p data-nodeid="10409">什么是 Lambda 架构呢？下面的图 1 进行了说明。</p>
<p data-nodeid="11864"><img src="https://s0.lgstatic.com/i/image6/M00/2D/98/CioPOWBmuM6AEJQwAAIYIcuicaY318.png" alt="Drawing 0.png" data-nodeid="11868"></p>
<div data-nodeid="11865" class=""><p style="text-align:center">图 1 Lambda 系统架构图</p></div>



<p data-nodeid="10412">从上面的图 1 可以看出，Lambda 架构总体上分为三层：批处理层（batch layer）、快速处理层（speed layer）和服务层（serving layer），其中：</p>
<ul data-nodeid="10413">
<li data-nodeid="10414">
<p data-nodeid="10415">批处理层负责处理主数据集（也就是历史全量数据）；</p>
</li>
<li data-nodeid="10416">
<p data-nodeid="10417">快速处理层负责处理增量数据（也就是新进入系统的数据）；</p>
</li>
<li data-nodeid="10418">
<p data-nodeid="10419">服务层用于将批处理层和快速处理层的结果合并起来，给用户或应用程序提供查询服务。</p>
</li>
</ul>
<p data-nodeid="10420">Lambda 架构是一种架构设计思路，针对每一层的技术组件选型并没有严格限定。我们可以根据自己公司和项目的实际情况，选择相应的技术方案。</p>
<p data-nodeid="10421">对于批处理层，数据存储可以选择 HDFS、S3 等大数据存储系统，而计算工具则可以选择 MapReduce、Hive、Spark 等大数据处理框架。批处理层的计算结果（比如数据库表或者视图），由于需要被服务层或快速处理层快速访问，所以可以存放在诸如 MySQL、HBase 等能够快速响应查询请求的数据库中。</p>
<p data-nodeid="10422">对于快速处理层，这就是各种流计算框架的用武之地了，比如 Flink、Spark Streaming 和 Storm 等。快速处理层由于对性能要求更加严苛，它们的计算结果可以存入像 Redis 这样具有超高性能表现的内存数据库中。不过有时候为了查询方便，也可以将计算结果存放在 MySQL 等传统数据库中，毕竟这些数据库配合缓存一起使用的话，性能也是非常棒的。</p>
<p data-nodeid="10423">对于服务层，当其接收到查询请求时，就可以分别从存储批处理层和快速处理层计算结果的数据库中，取出相应的计算结果并做合并，就能得到最终的查询结果了。</p>
<p data-nodeid="10424">不过，虽然 Lambda 架构实现了间接的实时计算，但它也存在一些问题。其中最主要的就是，对于同一个查询目标，需要分别为批处理层和快速处理层开发不同的算法实现。也就是说，对于相同的逻辑，需要开发两种不同的代码，并使用两种不同的计算框架（比如同时使用 Storm 和 Spark），这对开发、测试和运维，都带来一定的复杂性和额外工作。</p>
<p data-nodeid="10425">所以，Lambda 架构的改进版本，也就是 Kappa 架构应运而生。</p>
<h3 data-nodeid="10426">Kappa 架构</h3>
<p data-nodeid="10427">下面的图 2 展示了 Kappa 架构的工作原理。</p>
<p data-nodeid="12504" class=""><img src="https://s0.lgstatic.com/i/image6/M01/2D/90/Cgp9HWBmuNqAeyzOAAIVhE-o3d0639.png" alt="Drawing 1.png" data-nodeid="12508"></p>
<div data-nodeid="12505"><p style="text-align:center">图 2 Kappa 系统架构图</p></div>



<p data-nodeid="10430">从上面的图 2 可以看出，Kappa 架构相比 Lambda 架构的最大改进，就是将批处理层也用快速处理层的流计算技术所取代。这样一来，批处理层和快速处理层<strong data-nodeid="10487">使用相同的流计算逻辑，并有更统一的计算框架</strong>，从而降低了开发、测试和运维的成本。</p>
<p data-nodeid="10431">另外，由于 Kappa 架构完全使用“流计算”来处理数据，这就让我们在“存储”方面也可以作出调整。我们不必再像在 Lambda 架构中，将离线数据转储到 HDFS、S3 这样的“块数据”存储系统。而只需要将数据按照“流”的方式，存储在 Kafka 这样的“流数据”存储系统里即可。这既减少了数据存储的空间，也避免了不必要的数据转储，同时还降低了系统的复杂程度。</p>
<p data-nodeid="10432">所以说，在 Flink 和 Spark Streaming 等新一代<strong data-nodeid="10502">流批一体计算框架</strong>，以及诸如 Kafka 和 Pulsar 等新一代<strong data-nodeid="10503">流式大数据存储系统</strong>的<strong data-nodeid="10504">双重加持</strong>下，使用 Kappa 架构处理大数据，已经成为一种非常自然的选择。</p>
<h3 data-nodeid="10433">使用 Flink 实现 Kappa 架构</h3>
<p data-nodeid="10434">正所谓光说不练假把式，下面我们就使用 Flink 来演示如何实现 Kappa 架构。</p>
<p data-nodeid="10435">假设现在我们需要统计“最近 3 天每种商品的销售量”。根据 Kappa 架构的思路，我们将这个计算任务，分为离线处理层和快速处理层。</p>
<p data-nodeid="10436">其中离线处理层的实现如下（<a href="https://github.com/alain898/realtime_stream_computing_course/blob/main/course_bonus02/src/main/java/com/alain898/course/realtimestreaming/course_bonus02/example/BatchLayer.java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="10511">完整代码参考这里</a>）：</p>
<pre class="lang-java" data-nodeid="10437"><code data-language="java">DataStream counts = stream
&nbsp; &nbsp; <span class="hljs-comment">// 将字符串的数据解析为JSON对象</span>
&nbsp; &nbsp; .map(<span class="hljs-keyword">new</span> MapFunction&lt;String, Event&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Event <span class="hljs-title">map</span><span class="hljs-params">(String s)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> JSONObject.parseObject(s, Event.class);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; })
&nbsp; &nbsp; // 提取出每个事件中的商品，转化为商品计数事件
&nbsp; &nbsp; .map(<span class="hljs-keyword">new</span> MapFunction&lt;Event, CountedEvent&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> CountedEvent <span class="hljs-title">map</span><span class="hljs-params">(Event event)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CountedEvent(event.product, <span class="hljs-number">1</span>, event.timestamp);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; })
&nbsp; &nbsp; .assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> EventTimestampPeriodicWatermarks())
&nbsp; &nbsp; .keyBy(<span class="hljs-string">"product"</span>)
&nbsp; &nbsp; <span class="hljs-comment">// 对于批处理层，使用滑动窗口SlidingEventTimeWindows</span>
.timeWindow(Time.days(<span class="hljs-number">3</span>), Time.minutes(<span class="hljs-number">30</span>))
<span class="hljs-comment">// 最后是批处理窗口内的聚合计算</span>
&nbsp; &nbsp; .reduce((e1, e2) -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; CountedEvent countedEvent = <span class="hljs-keyword">new</span> CountedEvent();
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.product = e1.product;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.timestamp = e1.timestamp;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.count = e1.count + e2.count;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.minTimestamp = Math.min(e1.minTimestamp, e2.minTimestamp);
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.maxTimestamp = Math.max(e1.maxTimestamp, e2.maxTimestamp);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> countedEvent;
&nbsp; &nbsp; });
</code></pre>
<p data-nodeid="12825">在上面的代码中，我们采用了长度为 3 天，步长为 30 分钟的滑动窗口。也就是说，每三十分钟会计算一次三天内各个商品的销售量。</p>
<p data-nodeid="12826">接下来是快速处理层的实现（<a href="https://github.com/alain898/realtime_stream_computing_course/blob/main/course_bonus02/src/main/java/com/alain898/course/realtimestreaming/course_bonus02/example/FastLayer.java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="12831">完整代码参考这里</a>）：</p>

<pre class="lang-java" data-nodeid="10439"><code data-language="java">DataStream counts = stream
&nbsp; &nbsp; <span class="hljs-comment">// 将字符串的数据解析为JSON对象</span>
&nbsp; &nbsp; .map(<span class="hljs-keyword">new</span> MapFunction&lt;String, Event&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Event <span class="hljs-title">map</span><span class="hljs-params">(String s)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> JSONObject.parseObject(s, Event.class);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; })
&nbsp; &nbsp; // 提取出每个事件中的商品，转化为商品计数事件
&nbsp; &nbsp; .map(<span class="hljs-keyword">new</span> MapFunction&lt;Event, CountedEvent&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> CountedEvent <span class="hljs-title">map</span><span class="hljs-params">(Event event)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CountedEvent(event.product, <span class="hljs-number">1</span>, event.timestamp);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; })
&nbsp; &nbsp; .assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> EventTimestampPeriodicWatermarks())
&nbsp; &nbsp; .keyBy(x -&gt; x.product)
&nbsp; &nbsp; <span class="hljs-comment">// 对于批处理层，使用翻转窗口TumblingEventTimeWindows</span>
&nbsp; &nbsp; .window(TumblingEventTimeWindows.of(Time.seconds(<span class="hljs-number">15</span>)))
<span class="hljs-comment">// 最后是批处理窗口内的聚合计算</span>
&nbsp; &nbsp; .reduce((e1, e2) -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; CountedEvent countedEvent = <span class="hljs-keyword">new</span> CountedEvent();
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.product = e1.product;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.timestamp = e1.timestamp;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.count = e1.count + e2.count;
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.minTimestamp = Math.min(e1.minTimestamp, e2.minTimestamp);
&nbsp; &nbsp; &nbsp; &nbsp; countedEvent.maxTimestamp = Math.max(e1.maxTimestamp, e2.maxTimestamp);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> countedEvent;
&nbsp; &nbsp; });
</code></pre>
<p data-nodeid="13149">在上面的代码中，我们采用了长度为 15 秒的翻滚窗口。也就是说，每 15 秒钟会计算一次 15 秒内各个商品的销售量。</p>
<p data-nodeid="13150">从上面两部分的代码中，我们就可以体会到 Kappa 架构的优势所在了。因为，在上面<strong data-nodeid="13165">批处理层</strong>和<strong data-nodeid="13166">快速处理层</strong>的实现中，<strong data-nodeid="13167">除了两个窗口的类型不一样以外，其他的代码完全相同</strong>！是不是非常惊艳？！</p>

<p data-nodeid="10441">接下来，在批处理层和快速处理层各自计算出结果后，需要将计算结果存入数据库，具体如下（<a href="https://github.com/alain898/realtime_stream_computing_course/blob/main/course_bonus02/src/main/java/com/alain898/course/realtimestreaming/course_bonus02/example/JdbcWriter.java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="10541">完整代码参考这里</a>）：</p>
<pre class="lang-java" data-nodeid="10442"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JdbcWriter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichSinkFunction</span>&lt;<span class="hljs-title">CountedEvent</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 将每个窗口内的计算结果保存到数据库中</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String inset_sql = <span class="hljs-string">"INSERT INTO table_counts(id,start,end,product,v_count,layer) VALUES(?,?,?,?,?,?) "</span> +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"ON DUPLICATE KEY UPDATE start=?,end=?,product=?,v_count=?,layer=?;"</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> slideMS = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> slideNumberInWindow = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String layer = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">JdbcWriter</span><span class="hljs-params">(<span class="hljs-keyword">long</span> slideMS, <span class="hljs-keyword">long</span> slideNumberInWindow, String layer)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.slideMS = slideMS;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.slideNumberInWindow = slideNumberInWindow;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.layer = layer;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(CountedEvent value, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 通过对滑动或翻滚的步长取整，以对齐时间窗口，从而方便后续合并离线部分和实时部分的计算结果</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> start = value.minTimestamp / slideMS;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> end = value.minTimestamp / slideMS + slideNumberInWindow;
&nbsp; &nbsp; &nbsp; &nbsp; String product = value.product;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> v_count = value.count;
&nbsp; &nbsp; &nbsp; &nbsp; String layer = <span class="hljs-keyword">this</span>.layer;
&nbsp; &nbsp; &nbsp; &nbsp; String id = DigestUtils.md5Hex(Joiner.on(<span class="hljs-string">"&amp;"</span>).join(Lists.newArrayList(start, end, product, layer)));
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setString(<span class="hljs-number">1</span>, id);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setLong(<span class="hljs-number">2</span>, start);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setLong(<span class="hljs-number">3</span>, end);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setString(<span class="hljs-number">4</span>, product);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setInt(<span class="hljs-number">5</span>, v_count);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setString(<span class="hljs-number">6</span>, layer);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setLong(<span class="hljs-number">7</span>, start);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setLong(<span class="hljs-number">8</span>, end);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setString(<span class="hljs-number">9</span>, product);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setInt(<span class="hljs-number">10</span>, v_count);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.setString(<span class="hljs-number">11</span>, layer);
&nbsp; &nbsp; &nbsp; &nbsp; preparedStatement.executeUpdate();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="13484">在上面的代码中，我们将批处理层和快速处理层的结果都存入了数据库。</p>
<p data-nodeid="13485" class="">最后，服务层只需要使用一条简单的 SQL 语句，就能将批处理层和快速处理层的计算结果合并起来，具体如下（<a href="https://github.com/alain898/realtime_stream_computing_course/blob/main/course_bonus02/src/main/java/com/alain898/course/realtimestreaming/course_bonus02/example/ServerLayer.java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="13490">完整代码参考这里</a>）：</p>

<pre class="lang-sql" data-nodeid="10444"><code data-language="sql"><span class="hljs-keyword">SELECT</span> product, <span class="hljs-keyword">sum</span>(v_count) <span class="hljs-keyword">as</span> s_count <span class="hljs-keyword">from</span>
(
	<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> table_counts <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">start</span>=? <span class="hljs-keyword">AND</span> <span class="hljs-keyword">end</span>=? <span class="hljs-keyword">AND</span> layer=<span class="hljs-string">'batch'</span>
	<span class="hljs-keyword">UNION</span>
	<span class="hljs-keyword">SELECT</span> * <span class="hljs-keyword">FROM</span> table_counts <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">start</span>&gt;=? <span class="hljs-keyword">AND</span> <span class="hljs-keyword">end</span>&lt;=? <span class="hljs-keyword">AND</span> layer=<span class="hljs-string">'fast'</span>
) <span class="hljs-keyword">as</span> union_table <span class="hljs-keyword">GROUP</span> <span class="hljs-keyword">BY</span> product;
</code></pre>
<p data-nodeid="10445">在上面的代码中，我们使用 UNION 操作，将批处理层和快速处理层的结果合并起来。然后，在这个合并的表上，通过分组聚合计算，就能非常方便地计算出“最近 3 天每种商品的销售量”这个计算目标了。</p>
<h3 data-nodeid="10446">小结</h3>
<p data-nodeid="10447">今天，我们讨论了采用 Lambda 架构和 Kappa 架构，间接实现实时计算的方法。</p>
<p data-nodeid="10448">总的来说，Lambda 架构是一种通用的架构思想，它告诉我们，<strong data-nodeid="10558">当不能直接做到实时计算时，不妨尝试采用离线和实时相结合的折中计算方案</strong>。而从 Lambda 架构上改进而来的 Kappa 架构，通过“流”来统一“编程界面”，降低了系统、开发和运维的复杂程度。</p>
<p data-nodeid="10449">但是，这并不意味着 Kappa 架构就能够取代 Lambda 架构了。</p>
<p data-nodeid="10450">因为，在实际项目开发过程中，并不是所有的任务都适合用流计算的方式来完成。目前为止，采用批处理方式实现的算法，比采用流处理方式实现的算法，不管是在丰富度、成熟度、还是可用第三方工具库方面，都要优越很多。</p>
<p data-nodeid="10451">另外，是选择将离线计算和实时计算框架统一起来，还是将数据人员（他们已经有很多好用且熟悉的数据分析工具，比如 R、Python、Spark 等）和开发人员各自的生产力和创造力发挥出来，还有待商榷。</p>
<p data-nodeid="10452">所以，我们还是需要根据具体的业务场景、已有技术积累、团队研发能力等多方面因素，设计出最终能够真实落地的方案。</p>
<p data-nodeid="10453">最后，你在实际工作中有没有碰到过，不能够直接实现实时计算的场景呢？如果使用 Lambda 架构或 Kappa 架构的话，你会怎么做？可以将你的想法或问题写在留言区。</p>
<p data-nodeid="10454">下面是本课时的知识脑图。</p>
<p data-nodeid="13808" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/2D/98/CioPOWBmuPiAemNQAAhVcWpIr6g122.png" alt="Drawing 2.png" data-nodeid="13811"></p>

---

### 精选评论


