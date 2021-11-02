<p data-nodeid="80407">上一课时我们使用了 3 种方法进行了 PV 和 UV 的计算，分别是全窗口内存统计、使用分组和过期数据剔除、使用 BitMap / 布隆过滤器。到此为止我们已经讲了从数据清洗到水印、窗口设计，PV 和 UV 的计算，接下来需要把结果写入不同的目标库供前端查询使用。</p>
<p data-nodeid="80408">下面我们分别讲解 Flink 和 Redis/MySQL/HBase 是如何整合实现 Flink Sink 的。</p>
<h3 data-nodeid="80822" class="">Flink Redis Sink</h3>

<p data-nodeid="80410">我们在第 27 课时，详细讲解过 Flink 使用 Redis 作为 Sink 的设计和实现，分别使用自定义 Redis Sink、开源的 Redis Connector 实现了写入 Redis。</p>
<p data-nodeid="80411">在这里我们直接使用开源的 Redis 实现，首先新增 Maven 依赖如下：</p>
<pre class="lang-xml" data-nodeid="80412"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-connector-redis_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.1.5<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span> 
</code></pre>
<p data-nodeid="80413">可以通过实现 RedisMapper 来自定义 Redis Sink，在这里我们使用 Redis 中的 HASH 作为存储结构，Redis 中的 HASH 相当于 Java 语言里面的 HashMap：</p>
<pre class="lang-java" data-nodeid="80414"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyRedisSink</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RedisMapper</span>&lt;<span class="hljs-title">Tuple3</span>&lt;<span class="hljs-title">String</span>,<span class="hljs-title">String</span>, <span class="hljs-title">Integer</span>&gt;&gt;</span>{ 
    <span class="hljs-comment">/** 
     * 设置redis数据类型 
     */</span> 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> RedisCommandDescription <span class="hljs-title">getCommandDescription</span><span class="hljs-params">()</span> </span>{ 
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> RedisCommandDescription(RedisCommand.HSET,<span class="hljs-string">"flink_pv_uv"</span>); 
    } 
    <span class="hljs-comment">//指定key </span>
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKeyFromData</span><span class="hljs-params">(Tuple3&lt;String, String, Integer&gt; data)</span> </span>{ 
        <span class="hljs-keyword">return</span> data.f1; 
    } 
    <span class="hljs-comment">//指定value </span>
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getValueFromData</span><span class="hljs-params">(Tuple3&lt;String, String, Integer&gt; data)</span> </span>{ 
        <span class="hljs-keyword">return</span> data.f2.toString(); 
    } 
} 
</code></pre>
<p data-nodeid="80415">上面实现了 RedisMapper 并覆写了其中的 getCommandDescription、getKeyFromData、getValueFromData 3 种方法，其中 getCommandDescription 定义了存储到 Redis 中的数据格式。这里我们定义的 RedisCommand 为 HSET，使用 Redis 中的 HASH 作为数据结构；getKeyFromData 定义了 HASH 的 Key；getValueFromData 定义了 HASH 的值。</p>
<p data-nodeid="80416">然后我们直接调用 addSink 函数即可：</p>
<pre class="lang-java" data-nodeid="80417"><code data-language="java">... 
userClickSingleOutputStreamOperator 
            .keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;UserClick, String&gt;() { 
                <span class="hljs-meta">@Override</span> 
                <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKey</span><span class="hljs-params">(UserClick value)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                    <span class="hljs-keyword">return</span> value.getUserId(); 
                } 
            }) 
            .window(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
            .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
            .evictor(TimeEvictor.of(Time.seconds(<span class="hljs-number">0</span>), <span class="hljs-keyword">true</span>)) 
            .process(<span class="hljs-keyword">new</span> MyProcessWindowFunction()) 
            .addSink(<span class="hljs-keyword">new</span> RedisSink&lt;&gt;(conf,<span class="hljs-keyword">new</span> MyRedisSink())); 
... 
</code></pre>
<p data-nodeid="80418">接下来讲解 Flink 和 MySQL 是如何整合实现 Flink Sink 的？</p>
<h3 data-nodeid="80986" class="">Flink MySQL Sink</h3>

<p data-nodeid="80420">Flink 在最新版本 1.11 中支持了新的 JDBC Connector，我们可以直接在 Maven 中新增依赖：</p>
<pre class="lang-xml" data-nodeid="80421"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span> 
  <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
  <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-connector-jdbc_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
  <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.11.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span> 
</code></pre>
<p data-nodeid="80422">可以直接使用 JdbcSink 如下：</p>
<pre class="lang-java" data-nodeid="80423"><code data-language="java">String driverClass = <span class="hljs-string">"com.mysql.jdbc.Driver"</span>; 
String dbUrl = <span class="hljs-string">"jdbc:mysql://127.0.0.1:3306/test"</span>; 
String userNmae = <span class="hljs-string">"root"</span>; 
String passWord = <span class="hljs-string">"123456"</span>; 

userClickSingleOutputStreamOperator 
        .keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;UserClick, String&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKey</span><span class="hljs-params">(UserClick value)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                <span class="hljs-keyword">return</span> value.getUserId(); 
            } 
        }) 
        .window(TumblingProcessingTimeWindows.of(Time.days(<span class="hljs-number">1</span>), Time.hours(-<span class="hljs-number">8</span>))) 
        .trigger(ContinuousProcessingTimeTrigger.of(Time.seconds(<span class="hljs-number">20</span>))) 
        .evictor(TimeEvictor.of(Time.seconds(<span class="hljs-number">0</span>), <span class="hljs-keyword">true</span>)) 
        .process(<span class="hljs-keyword">new</span> MyProcessWindowFunction()) 
        .addSink( 
                JdbcSink.sink( 
                        <span class="hljs-string">"replace into pvuv_result (type,value) values (?,?)"</span>, 
                        (ps, value) -&gt; { 
                            ps.setString(<span class="hljs-number">1</span>, value.f1); 
                            ps.setInt(<span class="hljs-number">2</span>,value.f2); 
                        }, 
                        <span class="hljs-keyword">new</span> JdbcConnectionOptions.JdbcConnectionOptionsBuilder() 
                                .withUrl(dbUrl) 
                                .withDriverName(driverClass) 
                                .withUsername(userNmae) 
                                .withPassword(passWord) 
                                .build()) 
                ); 
</code></pre>
<p data-nodeid="80424">JDBC Sink 可以保证 "at-least-once" 语义保障，可通过实现“有则更新、无则写入”来实现写入 MySQL 的幂等性来实现 "exactly-once" 语义。</p>
<p data-nodeid="80425">当然我们也可以自定义 MySQL Sink，直接继承 RichSinkFunction ：</p>
<pre class="lang-java" data-nodeid="80426"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyMysqlSink</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichSinkFunction</span>&lt;<span class="hljs-title">Person</span>&gt; </span>{ 
    <span class="hljs-keyword">private</span> PreparedStatement ps = <span class="hljs-keyword">null</span>; 
    <span class="hljs-keyword">private</span> Connection connection = <span class="hljs-keyword">null</span>; 
    String driver = <span class="hljs-string">"com.mysql.jdbc.Driver"</span>; 
    String url = <span class="hljs-string">"jdbc:mysql://127.0.0.1:3306/test"</span>; 
    String username = <span class="hljs-string">"root"</span>; 
    String password = <span class="hljs-string">"123456"</span>; 
    <span class="hljs-comment">// 初始化方法 </span>
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.open(parameters); 
        <span class="hljs-comment">// 获取连接 </span>
        connection = getConn(); 
        connection.setAutoCommit(<span class="hljs-keyword">false</span>); 
    } 
    <span class="hljs-function"><span class="hljs-keyword">private</span> Connection <span class="hljs-title">getConn</span><span class="hljs-params">()</span> </span>{ 
        <span class="hljs-keyword">try</span> { 
            Class.forName(driver); 
            connection = DriverManager.getConnection(url, username, password); 
        } <span class="hljs-keyword">catch</span> (Exception e) { 
            e.printStackTrace(); 
        } 
        <span class="hljs-keyword">return</span> connection; 
    } 
    <span class="hljs-comment">//每一个元素的插入，都会调用一次 </span>
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(Tuple3&lt;String,String,Integer&gt; data, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        ps.prepareStatement(<span class="hljs-string">"replace into pvuv_result (type,value) values (?,?)"</span>) 
        ps.setString(<span class="hljs-number">1</span>,data.f1); 
        ps.setInt(<span class="hljs-number">2</span>,data.f2); 
        ps.execute(); 
        connection.commit(); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.close(); 
        <span class="hljs-keyword">if</span>(connection != <span class="hljs-keyword">null</span>){ 
            connection.close(); 
        } 
        <span class="hljs-keyword">if</span> (ps != <span class="hljs-keyword">null</span>){ 
            ps.close(); 
        } 
    } 
} 
</code></pre>
<p data-nodeid="80427">我们通过重写 open、invoke、close 方法，数据写入 MySQL 时会首先调用 open 方法新建连接，然后调用 invoke 方法写入 MySQL，最后执行 close 方法关闭当前连接。</p>
<p data-nodeid="80428">最后来讲讲 Flink 和 HBase 是如何整合实现 Flink Sink 的？</p>
<h3 data-nodeid="81150" class="">Flink HBase Sink</h3>

<p data-nodeid="80430">HBase 也是我们经常使用的存储系统之一。</p>
<blockquote data-nodeid="80431">
<p data-nodeid="80432">HBase 是一个分布式的、面向列的开源数据库，该技术来源于 Fay Chang 所撰写的 Google 论文“Bigtable：一个结构化数据的分布式存储系统”。就像 Bigtable 利用了 Google 文件系统（File System）所提供的分布式数据存储一样，HBase 在 Hadoop 之上提供了类似于 Bigtable 的能力。HBase 是 Apache 的 Hadoop 项目的子项目。HBase 不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库；另一个不同的是 HBase 基于列的而不是基于行的模式。</p>
</blockquote>
<p data-nodeid="80433">如果你对 HBase 不了解，可以参考官网给出的 <a href="http://hbase.apache.org/book.html#quickstart" data-nodeid="80477">快速入门</a>。</p>
<p data-nodeid="80434">Flink 没有提供直接连接 HBase 的连接器，我们通过继承 RichSinkFunction 来实现 HBase Sink。</p>
<p data-nodeid="80435">首先，我们在 Maven 中新增以下依赖：</p>
<pre class="lang-xml" data-nodeid="80436"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.hbase<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>hbase-client<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.2.6.1<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span> 
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.hadoop<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>hadoop-common<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span> 
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.7.5<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span> 
</code></pre>
<p data-nodeid="80437">接下来通过继承 RichSinkFunction 覆写其中的 open、invoke、close 方法。代码如下：</p>
<pre class="lang-java" data-nodeid="80438"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyHbaseSink</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichSinkFunction</span>&lt;<span class="hljs-title">Tuple3</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>, <span class="hljs-title">Integer</span>&gt;&gt; </span>{ 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> Connection connection; 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.open(parameters); 
        org.apache.hadoop.conf.Configuration conf = HBaseConfiguration.create(); 
        conf.set(HConstants.ZOOKEEPER_QUORUM, <span class="hljs-string">"localhost:2181"</span>); 
        connection = ConnectionFactory.createConnection(conf); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(Tuple3&lt;String, String, Integer&gt; value, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        String tableName = <span class="hljs-string">"database:pvuv_result"</span>; 
        String family = <span class="hljs-string">"f"</span>; 
        Table table = connection.getTable(TableName.valueOf(tableName)); 
        Put put = <span class="hljs-keyword">new</span> Put(value.f0.getBytes()); 
        put.addColumn(Bytes.toBytes(family),Bytes.toBytes(value.f1),Bytes.toBytes(value.f2)); 
        table.put(put); 
        table.close(); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.close(); 
        connection.close(); 
    } 
} 
</code></pre>
<p data-nodeid="80439">因为我们的程序是每 20 秒计算一次，并且输出，所以在写入 HBase 时没有使用批量方式。在实际的业务中，如果你的输出写入 HBase 频繁，那么推荐使用批量提交的方式。我们只需要稍微修改一下代码实现即可：</p>
<pre class="lang-java" data-nodeid="80440"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyHbaseSink</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichSinkFunction</span>&lt;<span class="hljs-title">Tuple3</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>, <span class="hljs-title">Integer</span>&gt;&gt; </span>{ 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> Connection connection; 
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> List&lt;Put&gt; puts = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(<span class="hljs-number">100</span>); 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration parameters)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.open(parameters); 
        org.apache.hadoop.conf.Configuration conf = HBaseConfiguration.create(); 
        conf.set(HConstants.ZOOKEEPER_QUORUM, <span class="hljs-string">"localhost:2181"</span>); 
        connection = ConnectionFactory.createConnection(conf); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(Tuple3&lt;String, String, Integer&gt; value, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        String tableName = <span class="hljs-string">"database:pvuv_result"</span>; 
        String family = <span class="hljs-string">"f"</span>; 
        Table table = connection.getTable(TableName.valueOf(tableName)); 
        Put put = <span class="hljs-keyword">new</span> Put(value.f0.getBytes()); 
        put.addColumn(Bytes.toBytes(family),Bytes.toBytes(value.f1),Bytes.toBytes(value.f2)); 
        puts.add(put); 
        <span class="hljs-keyword">if</span>(puts.size() == <span class="hljs-number">100</span>){ 
            table.put(puts); 
            puts.clear(); 
        } 
        table.close(); 
    } 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">super</span>.close(); 
        connection.close(); 
    } 
} 
</code></pre>
<p data-nodeid="80441">我们定义了一个容量为 100 的 List<put>，每 100 条数据批量提交一次，可以大大提高写入效率。</put></p>
<h3 data-nodeid="81314" class="">总结</h3>

<p data-nodeid="81634">这节课我们学习了 Flink 计算 PV、UV后的结果分别写入 Redis、MySQL 和 HBase。我们在实际业务中可以选择使用不同的目标库，你可以在本文中找到对应的实现根据实际情况进行修改来使用。</p>

---

### 精选评论

##### **民：
> 老师，您好。请问代码中的MyHbaseSink当puts链表中有缓存数据，但应用程序异常退出了，请问对业务是否有影响呢，puts中的数据会自动重算吗(假如设置了CheckPoint)？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有影响。内存中的数据会丢失，需要重置消费位点。这也是实时计算很大的痛点，需要你在下游做好幂等，不要因为再次消费导致数据不一致。

