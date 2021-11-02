<p data-nodeid="34478">在开始本课时的内容之前，我们先来回顾一下上个课时的习题：Personalized PageRank 和 PageRank 有什么不同？</p>
<p data-nodeid="34479">个性化 PageRank(Personalized PageRank) 算法继承了经典 PageRank 算法的思想，利用数据模型(图)链接结构来递归地计算各结点的权重，即模拟用户通过点击链接随机访问图中结点的行为 (随机行走模型)计算稳定状态下各结点得到的随机访问概率。个性化 PageRank 与 PageRank 的最大区别在于随机行走中的跳转行为。</p>
<p data-nodeid="35931" class="">接下来，我们就进入本课时的学习。GraphX 内置了 collectNeighborIds 函数，可以得到每个顶点的邻居顶点，也就是 1 度邻居顶点。如果需要得到每个顶点的 2 度或者 <em data-nodeid="35937">n</em> 度邻居顶点，应该如何利用 GraphX 完成这个任务呢？本课时将解答这个问题。</p>


<p data-nodeid="36169" class="">求 <em data-nodeid="36183">n</em> 度邻居顶点要比 1 度邻居顶点更复杂一些，<strong data-nodeid="36184">核心是要模拟出一个带有生命值的消息</strong>，每传播一次，生命值就会相应减 1，那么在生命值为 0 的时候到达的顶点就是我们所求的 <em data-nodeid="36185">n</em> 度邻居顶点。</p>




<p data-nodeid="35143" class="">本课时将用 GraphGenerators 造出一个度分布为正态分布的图，然后实现 vertexProgress、sendMsg 和 mergeMsg 这 3 个关键函数，代码如下：</p>


<pre class="lang-scala" data-nodeid="34483"><code data-language="scala">	<span class="hljs-keyword">import</span> org.apache.spark.graphx.<span class="hljs-type">EdgeTriplet</span>
	<span class="hljs-keyword">import</span> org.apache.spark.graphx.<span class="hljs-type">EdgeDirection</span>
	<span class="hljs-keyword">import</span> org.apache.spark.<span class="hljs-type">SparkConf</span>
	<span class="hljs-keyword">import</span> org.apache.spark.<span class="hljs-type">SparkContext</span>
	<span class="hljs-keyword">import</span> org.apache.spark.graphx.util.<span class="hljs-type">GraphGenerators</span>
	<span class="hljs-keyword">import</span> org.apache.spark.graphx.<span class="hljs-type">VertexId</span>
	<span class="hljs-keyword">import</span> org.apache.spark.graphx.<span class="hljs-type">PartitionStrategy</span>
	<span class="hljs-keyword">import</span> org.apache.spark.graphx.<span class="hljs-type">Graph</span>.graphToGraphOps
	<span class="hljs-keyword">import</span> scala.<span class="hljs-type">Iterator</span>
	&nbsp;
	<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">NDegreeNeighbor</span> </span>{
	&nbsp; 
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 顶点个数</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> vextexNum = args(<span class="hljs-number">0</span>).toInt
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 期望</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> u = args(<span class="hljs-number">1</span>).toDouble
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 方差</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> sigma = args(<span class="hljs-number">2</span>).toDouble
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 边分区数</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> eParts = args(<span class="hljs-number">3</span>).toInt
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 结果输出</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> outputPath = args(<span class="hljs-number">4</span>)
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 分区数</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> numParts = args(<span class="hljs-number">5</span>).toInt
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 分区策略</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> partStra = args(<span class="hljs-number">6</span>).toInt
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// n</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> n = args(<span class="hljs-number">7</span>).toInt
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> conf = <span class="hljs-keyword">new</span> <span class="hljs-type">SparkConf</span>()
	&nbsp;&nbsp;&nbsp; conf.setAppName(<span class="hljs-string">"NDegreeNeighbor"</span>)
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> sc = <span class="hljs-keyword">new</span> <span class="hljs-type">SparkContext</span>(conf)
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 生成度分布为对数正态分布的图，并将顶点属性初始化</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> g = <span class="hljs-type">GraphGenerators</span>
	&nbsp;&nbsp;&nbsp; .logNormalGraph(sc, vextexNum, eParts, u, sigma, <span class="hljs-type">System</span>.currentTimeMillis())
	&nbsp;&nbsp;&nbsp; .mapVertices[(<span class="hljs-type">List</span>[<span class="hljs-type">Long</span>],<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)],<span class="hljs-type">Int</span>)]((x,y) =&gt; (<span class="hljs-type">List</span>(),<span class="hljs-type">List</span>(),<span class="hljs-number">0</span>))
	&nbsp;&nbsp;&nbsp; .partitionBy(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(partStra == <span class="hljs-number">0</span>) 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">PartitionStrategy</span>.<span class="hljs-type">EdgePartition1D</span> 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (partStra == <span class="hljs-number">1</span>) 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">PartitionStrategy</span>.<span class="hljs-type">EdgePartition2D</span> 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (partStra == <span class="hljs-number">2</span>) 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">PartitionStrategy</span>.<span class="hljs-type">CanonicalRandomVertexCut</span> 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">PartitionStrategy</span>.<span class="hljs-type">RandomVertexCut</span>,numParts)
	&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 执行Pregel并保存结果</span>
	&nbsp;&nbsp;&nbsp;&nbsp; g.pregel[<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)]](<span class="hljs-type">List</span>((<span class="hljs-number">-1</span>L,<span class="hljs-number">-1</span>)), n, <span class="hljs-type">EdgeDirection</span>.<span class="hljs-type">Out</span>)(vertexProgress, sendMsg, mergeMsg)
	&nbsp;&nbsp;&nbsp;&nbsp; .vertices
	&nbsp;&nbsp;&nbsp;&nbsp; .filter(x =&gt; <span class="hljs-keyword">if</span>(x._2._1 == <span class="hljs-type">List</span>()) <span class="hljs-literal">false</span> <span class="hljs-keyword">else</span> <span class="hljs-literal">true</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; .saveAsTextFile(outputPath)
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">vertexProgress</span></span>(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; id:<span class="hljs-type">VertexId</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; attr:(<span class="hljs-type">List</span>[<span class="hljs-type">Long</span>],<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)],<span class="hljs-type">Int</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; msgSum:<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)])
	&nbsp;&nbsp;&nbsp;&nbsp; :(<span class="hljs-type">List</span>[<span class="hljs-type">Long</span>],<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)],<span class="hljs-type">Int</span>) = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 迭代次数</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> iteCount = attr._3
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 保存邻居顶点Id的集合</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> neighbor = attr._1
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 消息存储（集合）</span>
	&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">val</span> msgStore = attr._2
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 新的迭代次数</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> newIteCount = iteCount + <span class="hljs-number">1</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 如果是第一次迭代</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(iteCount == <span class="hljs-number">0</span>){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (<span class="hljs-type">List</span>(),<span class="hljs-type">List</span>(),newIteCount)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 如果是第n + 1次迭代，那么将生命值为1的消息保留</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(newIteCount == n + <span class="hljs-number">1</span>){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> newNeighbor = msgSum.par.filter(x =&gt; <span class="hljs-keyword">if</span>(x._2 == <span class="hljs-number">1</span>) <span class="hljs-literal">true</span> <span class="hljs-keyword">else</span> <span class="hljs-literal">false</span>).map(_._1).toList
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (newNeighbor,<span class="hljs-type">List</span>(),newIteCount)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 将传过来的消息保存，生命值-1，以备下一次发送消息，再滤掉不可能跳5次的消息</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<span class="hljs-keyword">else</span>{
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> newMsgStore:<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)] = msgSum.map(x =&gt; (x._1,x._2 - <span class="hljs-number">1</span>)).++(msgStore)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(x =&gt; (x._1,x._2 - <span class="hljs-number">1</span>))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (<span class="hljs-type">List</span>(),newMsgStore,newIteCount)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
	&nbsp;&nbsp;&nbsp; }
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">sendMsg</span></span>(edgeTriplet: <span class="hljs-type">EdgeTriplet</span>[(<span class="hljs-type">List</span>[<span class="hljs-type">Long</span>],<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)],<span class="hljs-type">Int</span>), <span class="hljs-type">Int</span>]):<span class="hljs-type">Iterator</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)])] = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> oldMsg = edgeTriplet.srcAttr._2
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> iteCount = edgeTriplet.srcAttr._3
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">// 最开始发消息，初始化生命值为n</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(iteCount == <span class="hljs-number">1</span>){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">Iterator</span>((edgeTriplet.dstId,<span class="hljs-type">List</span>((edgeTriplet.srcId,n))))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }<span class="hljs-keyword">else</span>{
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;<span class="hljs-type">Iterator</span>((edgeTriplet.dstId,oldMsg.par.filter(x =&gt; <span class="hljs-keyword">if</span>(x._2 + iteCount == n + <span class="hljs-number">1</span>) <span class="hljs-literal">true</span> <span class="hljs-keyword">else</span> <span class="hljs-literal">false</span>).toList))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
	&nbsp;&nbsp;&nbsp; }
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">mergeMsg</span></span>(a:<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)],b:<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)]):<span class="hljs-type">List</span>[(<span class="hljs-type">Long</span>,<span class="hljs-type">Int</span>)] = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; a.++(b)
	&nbsp;&nbsp;&nbsp; }&nbsp;&nbsp;&nbsp; 
	&nbsp; }
	}
</code></pre>
<p data-nodeid="35696">其中，u、sigma 参数用来声明生成图的顶点数的期望与标准差（对数正态分布）。另外，核心的数据结构是消息的数据结构与顶点属性的数据结构，它们分别是 List[(Long,Int)] 和 (List[Long],List[(Long,Int)],Int)，前者的消息是顶点 Id 以及消息生命值组成的元组。后者元组中的第一个元素是保存当前顶点的 <em data-nodeid="35715">n</em> 度邻居顶点 Id 的集合，用于结果输出；第二个元素用于存储发送过来的消息，第三个元素是当前迭代次数。</p>
<p data-nodeid="35697">在本例中，没有考虑成环的情况，你可以试着实现一下这种情况，这也是本课时留给你的思考题。</p>





<p data-nodeid="34644">其实求顶点的 n 度邻居这个需求并不常见，之所以举这个例子，是因为它很好地体现了 Pregel API 的核心用法：消息传递。你可以仔细理解这个带有生命值的消息的抽象，如果有更好的方法，欢迎与我讨论。</p>

---

### 精选评论


