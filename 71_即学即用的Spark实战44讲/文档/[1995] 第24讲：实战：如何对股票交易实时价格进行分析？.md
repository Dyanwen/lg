<p data-nodeid="343" class="">在流处理这个模块中，我们已经学习了 Spark 的两种流处理解决方案，在本课时中，我们将进行一个略微复杂的实践，也是我们本模块的实践环节。</p>


<p data-nodeid="3">在本模块中，我们将对实时股票价格数据进行处理。我们将在本课时中计算一个在股票分析中的比较常见的指标：CCI。</p>
<p data-nodeid="4">实时股票价格数据蕴含着巨大的价值，如何能在交易过程中敏锐地捕捉到机会非常重要，所以这就是一个非常典型的流处理应用场景。下面介绍一个股票交易实时分析应用：计算分钟级 CCI。</p>
<p data-nodeid="5"><strong data-nodeid="37">CCI（Commodity Channel Index），也被称为顺势指标</strong>。它最早用于期货市场的判断，后期才运用于股票市场的研判，并被广泛使用。与大多数单一利用股票的收盘价、开盘价、最高价或最低价而发明出的各种技术分析指标不同，<strong data-nodeid="38">CCI 指标是根据统计学原理，引进价格与固定期间的股价平均区间的偏离程度的概念，强调股价平均绝对偏差在股市技术分析中的重要性</strong>，是一种比较独特的技术指标。CCI 有事件时间区间的概念，很适合用 Structured Streaming 来完成。</p>
<p data-nodeid="601">CCI 有日 CCI、周 CCI、年 CCI 以及分钟 CCI 等很多种类型。本例主要实现的是 30 分钟 CCI（一个周期为 30 分钟），其计算公式为：</p>
<p data-nodeid="602" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/37/CgqCHl79v5uAX9KwAAAkw-PcxfE329.png" alt="image (3).png" data-nodeid="610"></p>


<p data-nodeid="883">其中</p>
<p data-nodeid="884" class=""><img src="https://s0.lgstatic.com/i/image/M00/2B/37/CgqCHl79v6OAVF7lAAAOQzHAHtY193.png" alt="image (4).png" data-nodeid="892"></p>


<p data-nodeid="10">这个公式为给定 30 分钟内的最高价、最低价和收盘价的平均值，SMA 是 N 个周期的 pt 的移动平均值，MD 是 pt 的平均离差，本例中 N = 3。</p>
<p data-nodeid="11">再来看看数据，目前股票的实时数据来源渠道有很多，如 Wind、大智慧等，都提供了自己的接口。假定数据已经被实时拉取并灌入到消息队列中，在这个过程中，有可能会出现数据晚到和乱序的现象。为了结果的准确，在处理时需要考虑这些情况，来看一条数据样例：</p>
<blockquote data-nodeid="12">
<p data-nodeid="13">000002.SZ, 1502126681, 22.71, 21.54, 22.32, 22.17</p>
</blockquote>
<p data-nodeid="14">其中每个字段分别是股票代码、事件时间戳、现价、买入价、卖出价、成交均价。</p>
<p data-nodeid="15">下面我们来看看 CCI 的计算方式，SMA、MD 都需要 pt 序列计算得到，而从公式可以看到计算 pt 需要先得到 30 分钟内的最高价、最低价和收盘价。当 pt 序列按照时间被保存到数据库后，那么计算 CCI 就非常容易了，一个应用定期进行查询并计算即可，<strong data-nodeid="55">所以计算 CCI 的核心是计算 pt。</strong></p>
<p data-nodeid="16">下面的代码采用 Structured Streaming 对数据流进行处理，得到最高价、最低价和收盘价并求其均值，从而得到 pt 序列。其中，我们需要开发一个求收盘价的 UDAF，然后还要开发一个输出到 HBase 的 Sink，正好可以把我们前面学到的知识用上。</p>
<pre class="lang-scala" data-nodeid="17"><code data-language="scala">	<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">SparkSession</span>
	<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
	<span class="hljs-keyword">import</span> org.apache.spark.sql.types.<span class="hljs-type">TimestampType</span>
	<span class="hljs-keyword">import</span> org.apache.spark.sql.streaming.<span class="hljs-type">Trigger</span>
	<span class="hljs-keyword">import</span> java.sql.<span class="hljs-type">Timestamp</span>
	
	<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">StockCCICompute</span> </span>{
	&nbsp; 
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">main</span></span>(args: <span class="hljs-type">Array</span>[<span class="hljs-type">String</span>]): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> spark = <span class="hljs-type">SparkSession</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .builder
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .appName(<span class="hljs-string">"StockCCICompute"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getOrCreate()
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//分别设置window长度、容忍最大晚到时间和触发间隔</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> windowDuration = <span class="hljs-string">"30 minutes"</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> waterThreshold = <span class="hljs-string">"5 minutes"</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> triggerTime = <span class="hljs-string">"1 minutes"</span>
	
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">import</span> spark.implicits._
	
	&nbsp;&nbsp;&nbsp; spark.readStream
	&nbsp;&nbsp;&nbsp; .format(<span class="hljs-string">"kafka"</span>)
	&nbsp;&nbsp;&nbsp; .option(<span class="hljs-string">"kafka.bootstrap.servers"</span>, <span class="hljs-string">"broker1:port1,broker2:port2"</span>)
	&nbsp;&nbsp;&nbsp; .option(<span class="hljs-string">"subscribe"</span>, <span class="hljs-string">"stock"</span>)
	&nbsp;&nbsp;&nbsp; .load()
	&nbsp;&nbsp;&nbsp; .selectExpr(<span class="hljs-string">"CAST(key AS STRING)"</span>, <span class="hljs-string">"CAST(value AS STRING)"</span>)
	&nbsp;&nbsp;&nbsp; .as[(<span class="hljs-type">String</span>, <span class="hljs-type">String</span>)]
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//解析数据</span>
	&nbsp;&nbsp;&nbsp; .map(f =&gt; {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> companyNo = f._1
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> infos = f._2.split(<span class="hljs-string">","</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; (f._1,infos(<span class="hljs-number">0</span>),infos(<span class="hljs-number">1</span>),infos(<span class="hljs-number">2</span>),infos(<span class="hljs-number">3</span>),infos(<span class="hljs-number">4</span>))
	&nbsp;&nbsp;&nbsp; })
	&nbsp;&nbsp;&nbsp; .toDF(<span class="hljs-string">"companyno"</span>,<span class="hljs-string">"timestamp"</span>,<span class="hljs-string">"price"</span>,<span class="hljs-string">"bidprice"</span>,<span class="hljs-string">"sellpirce"</span>,<span class="hljs-string">"avgprice"</span>)
	&nbsp;&nbsp;&nbsp; .selectExpr(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(companyno AS STRING)"</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(timestamp AS TIMESTAMP[DF1]&nbsp;)"</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(price AS DOUBLE)"</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(bidprice AS DOUBLE)"</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(sellpirce AS DOUBLE)"</span>,
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"CAST(avgprice AS DOUBLE)"</span>)
	&nbsp;&nbsp;&nbsp; .as[(<span class="hljs-type">String</span>,<span class="hljs-type">Timestamp</span>,<span class="hljs-type">Double</span>,<span class="hljs-type">Double</span>,<span class="hljs-type">Double</span>,<span class="hljs-type">Double</span>)]
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设定水位</span>
	&nbsp;&nbsp;&nbsp; .withWatermark(<span class="hljs-string">"timestamp"</span>, waterThreshold)
	&nbsp;&nbsp;&nbsp; .groupBy(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; window($<span class="hljs-string">"timestamp"</span>, 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; windowDuration), 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; $<span class="hljs-string">"companyno"</span>)
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//求出最高价、最低价和收盘价，其中收盘价需要自己开发UDAF</span>
	&nbsp;&nbsp;&nbsp; .agg(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; max(col(<span class="hljs-string">"price"</span>)).as(<span class="hljs-string">"max_price"</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; min(col(<span class="hljs-string">"price"</span>)).as(<span class="hljs-string">"min_price"</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">ClosePriceUDAF</span>(col(<span class="hljs-string">"price"</span>).as(<span class="hljs-string">"latest_price"</span>)))
	&nbsp;&nbsp;&nbsp; .writeStream
	&nbsp;&nbsp;&nbsp; .outputMode(<span class="hljs-string">"append"</span>)
	&nbsp;&nbsp;&nbsp; .trigger(<span class="hljs-type">Trigger</span>.<span class="hljs-type">ProcessingTime</span>(triggerTime))
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//输出到HBase中</span>
	&nbsp;&nbsp;&nbsp; .foreach(<span class="hljs-type">HBaseWriter</span>)
	&nbsp;&nbsp;&nbsp; .start()
	&nbsp;&nbsp;&nbsp; .awaitTermination()
	
	&nbsp; }
	}
</code></pre>
<p data-nodeid="18">代码中选取了 append 模式，所以分析应用不用处理结果发生变化的情况。另外代码风格特意采用了 Dataflow 的数据管道式，本例中的数据处理的逻辑是完全可以应用于批处理的。开发的 UDAF 目的是求出收盘价，也就是窗口内时间戳最大的那一条，代码如下：</p>
<pre class="lang-scala" data-nodeid="19"><code data-language="scala">	<span class="hljs-keyword">import</span> org.apache.spark.sql.expressions._
	<span class="hljs-keyword">import</span> org.apache.spark.sql.types._
	<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Row</span>
	<span class="hljs-keyword">import</span> org.apache.spark.sql.functions._
	<span class="hljs-keyword">import</span> java.sql.<span class="hljs-type">Timestamp</span>
	
	<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">ClosePriceUDAF</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDefinedAggregateFunction</span> </span>{
	
	&nbsp; <span class="hljs-comment">//指定输入的类型</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">inputSchema</span></span>: <span class="hljs-type">StructType</span> 
	&nbsp;&nbsp;&nbsp; = <span class="hljs-type">StructType</span>(<span class="hljs-type">Array</span>(<span class="hljs-type">StructField</span>[<span class="hljs-type">DF1</span>]&nbsp;(<span class="hljs-string">"price"</span>, <span class="hljs-type">DoubleType</span>, <span class="hljs-literal">true</span>)))
	&nbsp; 
	&nbsp; <span class="hljs-comment">//中间结果只需要两个字段：价格、时间</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">bufferSchema</span></span>: <span class="hljs-type">StructType</span> 
	&nbsp;&nbsp;&nbsp; = <span class="hljs-type">StructType</span>(
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">Array</span>(<span class="hljs-type">StructField</span>(<span class="hljs-string">"latestprice"</span>, <span class="hljs-type">DoubleType</span>, <span class="hljs-literal">true</span>),
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-type">StructField</span>(<span class="hljs-string">"timestamp"</span>, <span class="hljs-type">TimestampType</span>, <span class="hljs-literal">true</span>)))
	
	&nbsp; <span class="hljs-comment">//指定最后输出的类型</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dataType</span></span>: <span class="hljs-type">DataType</span> = <span class="hljs-type">DoubleType</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">deterministic</span></span>: <span class="hljs-type">Boolean</span> = <span class="hljs-literal">true</span>
	&nbsp; 
	&nbsp; <span class="hljs-comment">//初始化中间结果</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">initialize</span></span>(buffer: <span class="hljs-type">MutableAggregationBuffer</span>): <span class="hljs-type">Unit</span> 
	&nbsp;&nbsp;&nbsp; = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer(<span class="hljs-number">0</span>) = <span class="hljs-number">0</span>D
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer(<span class="hljs-number">1</span>) = <span class="hljs-number">0</span>L
	&nbsp;&nbsp;&nbsp; }
	&nbsp; 
	&nbsp; <span class="hljs-comment">//更新时间晚的价格为中间</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">update</span></span>(buffer: <span class="hljs-type">MutableAggregationBuffer</span>, input: <span class="hljs-type">Row</span>): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> priceNow = input.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"price"</span>)
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> timestampNow = input.getAs[<span class="hljs-type">Timestamp</span>](<span class="hljs-string">"timestamp"</span>)
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> timestampBuf = buffer.getAs[<span class="hljs-type">Timestamp</span>](<span class="hljs-string">"timestamp"</span>)
	&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(timestampNow.after(timestampBuf)){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer(<span class="hljs-number">0</span>) = priceNow
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer(<span class="hljs-number">1</span>) = timestampNow
	&nbsp;&nbsp;&nbsp; }
	&nbsp; }
	&nbsp;
	&nbsp; <span class="hljs-comment">//合并中间结果</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">merge</span></span>(buffer1: <span class="hljs-type">MutableAggregationBuffer</span>, buffer2: <span class="hljs-type">Row</span>): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> buffer1Timestamp = buffer1.getAs[<span class="hljs-type">Timestamp</span>](<span class="hljs-string">"timestamp"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> buffer2Timestamp = buffer2.getAs[<span class="hljs-type">Timestamp</span>](<span class="hljs-string">"timestamp"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span>(buffer2Timestamp.after(buffer1Timestamp)){
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer1(<span class="hljs-number">0</span>) = buffer2.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"price"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; buffer1(<span class="hljs-number">1</span>) = buffer2Timestamp
	&nbsp;&nbsp;&nbsp;&nbsp; }
	&nbsp; }
	&nbsp; 
	&nbsp; <span class="hljs-comment">//返回最后的结果</span>
	&nbsp; <span class="hljs-keyword">override</span> <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">evaluate</span></span>(buffer: <span class="hljs-type">Row</span>): <span class="hljs-type">Any</span> = buffer.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"price"</span>)
	}
</code></pre>
<p data-nodeid="20">最后为了保证结果的正确性，需要实现自定义 Writer。这也是选取 HBase 的原因，因为插入到 HBase 的操作天然就具有幂等性（重复 Put 会覆盖之前的值），所以可以实现端到端的恰好一次的消息送达的效果，代码如下：</p>
<pre class="lang-scala" data-nodeid="21"><code data-language="scala">	<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">ForeachWriter</span>
	<span class="hljs-keyword">import</span> org.apache.spark.sql.<span class="hljs-type">Row</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.<span class="hljs-type">HBaseConfiguration</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.client.<span class="hljs-type">ConnectionFactory</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.client.<span class="hljs-type">Connection</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.<span class="hljs-type">TableName</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.client[<span class="hljs-type">DF1</span>]&nbsp;.<span class="hljs-type">Put</span>
	<span class="hljs-keyword">import</span> org.apache.hadoop.hbase.util.<span class="hljs-type">Bytes</span>
	
	<span class="hljs-class"><span class="hljs-keyword">object</span> <span class="hljs-title">HBaseWriter</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ForeachWriter</span>[<span class="hljs-type">Row</span>] </span>{
	&nbsp; 
	&nbsp; <span class="hljs-keyword">var</span> conn: <span class="hljs-type">Connection</span>&nbsp; = <span class="hljs-literal">null</span>
	&nbsp; 
	&nbsp; <span class="hljs-comment">//初始化HBase连接</span>
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">open</span></span>(partitionId: <span class="hljs-type">Long</span>, version: <span class="hljs-type">Long</span>): <span class="hljs-type">Boolean</span> = {
	&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> conf = <span class="hljs-type">HBaseConfiguration</span>.create()
	&nbsp;&nbsp;&nbsp;&nbsp; conn = <span class="hljs-type">ConnectionFactory</span>.createConnection(conf)
	
	&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-literal">true</span>
	&nbsp; }
	&nbsp; 
	&nbsp; <span class="hljs-comment">//获取结果表中的字段，并以窗口标识为行键，插入HBase中</span>
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">process</span></span>(row: <span class="hljs-type">Row</span>): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//window字段作为rowkey供分析应用查询</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> window = row.getAs[<span class="hljs-type">String</span>](<span class="hljs-string">"window"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> maxPrice = row.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"max_price"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> minPrice = row.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"min_price"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> latestPrice = row.getAs[<span class="hljs-type">Double</span>](<span class="hljs-string">"latest_price"</span>)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> table = conn.getTable(<span class="hljs-type">TableName</span>.valueOf(<span class="hljs-string">"CCI"</span>))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">val</span> put = <span class="hljs-keyword">new</span> <span class="hljs-type">Put</span>(<span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"window"</span>))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//列族为cf，列名分别为max_price、min_price、latest_price</span>
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; put.addColumn(<span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"cf"</span>), <span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"max_price"</span>), <span class="hljs-type">Bytes</span>.toBytes(maxPrice))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; put.addColumn(<span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"cf"</span>), <span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"min_price"</span>), <span class="hljs-type">Bytes</span>.toBytes(minPrice))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; put.addColumn(<span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"cf"</span>), <span class="hljs-type">Bytes</span>.toBytes(<span class="hljs-string">"latest_price"</span>), <span class="hljs-type">Bytes</span>.toBytes(latestPrice))
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; table.put(put)
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; table.close()
	&nbsp; }&nbsp; 
	&nbsp; 
	&nbsp; <span class="hljs-comment">//关闭连接</span>
	&nbsp; <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">close</span></span>(errorOrNull: <span class="hljs-type">Throwable</span>): <span class="hljs-type">Unit</span> = {
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; conn.close()
	&nbsp; }
	
	}
</code></pre>
<p data-nodeid="1041">Structured Streaming 用窗口起点--窗口终点作为 window 字段的值，也就是窗口唯一标识。在入库时，该值作为行键方便应用查询。在数据入库后，分析应用可以用窗口标识进行查询，例如用 12:00-12:30、12:30-13:00、13:00-13:30 这三个值分别发起三次查询，从而得到这些窗口的最高价、最低价和收盘价，再分别计算出 pt，最后就能得到当前周期的 CCI。</p>
<p data-nodeid="1042">如果你看到这里，就会发现这个应用的开发过程以及它所需要的组件还是比较复杂的，除了上面的开发过程，我们还需要开发一个后端查询应用才能计算出 CCI，这也是流处理通常是属于数据工程领域而非数据科学领域。</p>

<p data-nodeid="1193" class="te-preview-highlight"><strong data-nodeid="1197">最后值得注意的是，从代码 spark.readStream 开始到最后处理过程的完成，其实是一行代码，这一行代码稍加改动也可直接用于批处理，这也是 Spark 统一编程接口的一种体现。</strong></p>

<p data-nodeid="205" class="">本课时的内容就到这里，下个课时我们将进入下一个模块的学习，我将为你讲解什么是图：图模式，图相关技术与使用场景。</p>

---

### 精选评论

##### **生：
> 那个自定义函数怎么会只有一个参数？应该是这样？ClosePriceUDAF(col("price")as "latest_price",col("timestamp"))

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，应该是两个参数，UDFA中的代码的输入数据结构中应该加上时间戳

