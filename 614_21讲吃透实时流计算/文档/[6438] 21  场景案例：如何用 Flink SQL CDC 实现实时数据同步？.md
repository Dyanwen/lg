<p data-nodeid="3">今天我们来看第二个案例，也就是用 Flink SQL CDC 实现实时数据同步。</p>
<p data-nodeid="4">那究竟什么是 Flink SQL CDC 呢？毕竟这是个相对还比较新的技术，可能很多人都还没听说过这个技术，所以我们先从它诞生的业务场景说起。</p>
<h3 data-nodeid="5">业务场景</h3>
<p data-nodeid="6">如果你是一名后端开发的话，相信十有八九遇到过这样的问题，有时候一种数据库满足不了业务的需求，我们需要将相同的数据，存入多种不同的数据库。</p>
<p data-nodeid="7">比如，最开始的时候业务比较简单，数据量也很小，数据只需要保存到 MySQL 中，作为主数据库即可。之后，随着业务的发展，数据量变得越来越大，为了提升查询效率，需要将数据写一份到 Redis 缓存。同时，业务查询也变得越来越复杂，为了提供更加灵活和高效的查询分析方式，需要将数据再写一份到 Elasticsearch 里。</p>
<p data-nodeid="8">面对以上这种情况，你会怎么做呢？一般情况下，我们首先想到的可能就是，改代码！改成类似于下面图 1 这样的方案。</p>
<p data-nodeid="1553" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/90/Cgp9HWBmt66ACLVqAABVxuvt-SA692.png" alt="Drawing 1.png" data-nodeid="1556"></p>



<p data-nodeid="12">在上面的图 1 中，我们直接在业务代码中，将数据写入了几种不同的数据库里，包括 MySQL、Redis 和 Elasticsearch。</p>
<p data-nodeid="13">这种方案看着简单，也很容易实现。但这种做法<strong data-nodeid="129">非常不好</strong>，原因是这样的：</p>
<ol data-nodeid="14">
<li data-nodeid="15">
<p data-nodeid="16">代码严重耦合，每次需要修改业务代码后重新测试和上线才行；</p>
</li>
<li data-nodeid="17">
<p data-nodeid="18">在业务系统中需要多次将数据写入不同数据库，严重影响业务代码性能；</p>
</li>
<li data-nodeid="19">
<p data-nodeid="20">增量同步前，需要先由人工（至少你要写脚本和执行脚本吧）做一次全量同步。</p>
</li>
</ol>
<p data-nodeid="21">一种好的改进方法，则是<strong data-nodeid="138">使用消息中间件</strong>，就像下面的图 2 这样。</p>
<p data-nodeid="2161" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/98/CioPOWBmt7aAP2UtAACBbUI0Ab4887.png" alt="Drawing 3.png" data-nodeid="2164"></p>



<p data-nodeid="25">在上面的图 2 中，首先由业务系统将数据写入 Kafka，然后由 Flink 从 Kafka 将消息读取出来，最后再写入不同的数据库。</p>
<p data-nodeid="26">这种做法最大的优点在于<strong data-nodeid="152">将业务系统和写入数据库的逻辑隔离开，降低了代码的耦合度</strong>，并且在业务中只需要写一次数据到 Kafka 即可，提升了业务系统的性能。</p>
<p data-nodeid="27">但它还是有几个缺点：</p>
<ol data-nodeid="28">
<li data-nodeid="29">
<p data-nodeid="30">还是需要修改代码，比如在写数据的地方埋点写入 Kafka；</p>
</li>
<li data-nodeid="31">
<p data-nodeid="32">增加了系统的复杂性，因为需要维护 Kafka，并且要开发写入数据库的代码。当需要写入的数据库和表比较多时，这种复杂性就更加严重了；</p>
</li>
<li data-nodeid="33">
<p data-nodeid="34">增量同步前，同样需要先由人工做一次全量同步。</p>
</li>
</ol>
<p data-nodeid="35">虽然有以上这些缺点，但不管怎样，这种方法在思路上都是值得遵循的。因为它可以解耦，并且性能会更好。特别是当我们有了 Flink 这种神器后，可以直接通过 Flink 从 Kafka 里读取出数据，然后高效地使用流计算技术写入各种数据库。</p>
<p data-nodeid="36">面对以上图 2 方案的缺点，咱们的 Flink 神器当然没有熟视无睹。所以，它基于 CDC 技术的思路，推出了 Flink CDC 实现。</p>
<p data-nodeid="37">CDC（Change Data Capture，变化数据捕获）正如其名，是一种捕获数据变化的技术。比如在 MySQL 做同步时，我们设置 MySQL<strong data-nodeid="168">从数据库</strong>（secondary database）来跟随<strong data-nodeid="169">主数据库</strong>（primary database）的 binlog 日志，从而将主数据库的所有数据变化同步到从数据库中，这其实就是一个 CDC 的使用场景。</p>
<p data-nodeid="38">而 Flink CDC 就是一种使用 Flink 流计算框架，来实现 CDC 功能的技术。比如，我们可以通过 Flink CDC 技术，先将主数据库的全量数据同步到另外的数据库中，然后再跟随主数据库的 binlog 日志，将所有增量的数据也实时同步到从数据库中。</p>
<p data-nodeid="39">由于 Flink CDC 将全量同步和增量同步的操作封装到了一起，并且因为 Flink 还支持 SQL 语句，所以最终我们只需要写几行简单的 SQL，就能轻松解决将同一份数据写入多种不同数据库的问题。</p>
<p data-nodeid="40">下面的图 3 就展示了使用 Flink CDC 进行数据同步的方案。</p>
<p data-nodeid="2757" class=""><img src="https://s0.lgstatic.com/i/image6/M00/2D/98/CioPOWBmt8CACcZ1AAB-UuW3dDg051.png" alt="Drawing 5.png" data-nodeid="2760"></p>



<p data-nodeid="44">从上面的图 3 可以看出，相比前面图 2 的方案，这里不需要再引入单独的 Kafka，也不需要我们修改业务系统的代码，只需要引入一个 Flink SQL CDC 工具，就能够同时实现数据的全量同步和增量同步。并且，由于 Flink SQL CDC 封装得很好，我们只需要写一些 SQL 就可以了。</p>
<p data-nodeid="45">Flink CDC 如此神奇，那它究竟是怎样实现的呢？接下来，我们就来看下 Flink CDC 的实现原理。</p>
<h3 data-nodeid="46">实现原理</h3>
<p data-nodeid="47">我们以使用 Flink CDC 从 MySQL 中同步数据的情景，来讲解下 Flink CDC 的工作原理。</p>
<p data-nodeid="48">一般来说，Flink CDC 同步数据需要两个步骤。</p>
<ul data-nodeid="49">
<li data-nodeid="50">
<p data-nodeid="51"><strong data-nodeid="188">第一步是将源数据库的数据全量同步到目标数据库；</strong></p>
</li>
<li data-nodeid="52">
<p data-nodeid="53"><strong data-nodeid="192">第二步是跟随源数据库的 binlog 日志，将源数据库的所有变动，以增量数据的方式同步到目标数据库。</strong></p>
</li>
</ul>
<p data-nodeid="54">我们先来看将源数据库的数据<strong data-nodeid="198">全量同步</strong>到目标数据库的过程。Flink CDC 将这个过程称之为“快照”（sanpshot），具体步骤是这样的。</p>
<ol data-nodeid="55">
<li data-nodeid="56">
<p data-nodeid="57">Flink CDC 会获取一个全局读锁（global read lock），从而阻塞其他客户端往数据库写入数据。不用担心这个锁定时间会很长，因为它马上就会在第 5 步中被释放掉。</p>
</li>
<li data-nodeid="58">
<p data-nodeid="59">启动一个可重复读语义（repeatable read semantics）的事务，从而确保后续在该事务内的所有“读”操作都是在“一致的快照”（consistent snapshot）中进行。这一步中“可重复读语义”以及后续步骤只涉及“读”操作是非常关键的。因为只有在“可重复读语义”且不存在“写”操作的情况下， MySQL 的“可重复读语义”级别事务才不会存在“幻读”现象（<a href="https://stackoverflow.com/questions/47041215/innodb-mysql-select-query-locking?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="203">具体原因可以参考这个问题的两个回答</a>），这样才能保证后续做“扫描”和读取 binlog 位置时，它们的数据和时间点是对得上的。这就是“一致的快照”的含义，它保证了同步到目标数据库中的数据是完整的，并且和源数据库中的数据是完全相同的，既不会多一条，也不会少一条。</p>
</li>
<li data-nodeid="60">
<p data-nodeid="61">读取当前 binlog 的位置。</p>
</li>
<li data-nodeid="62">
<p data-nodeid="63">读取 Flink CDC 配置指定的数据库和表定义(schema)。</p>
</li>
<li data-nodeid="64">
<p data-nodeid="65">释放步骤 1 中的全局读锁。这个时候其他的客户端就可以继续往数据库中写入数据了。从步骤 1 到步骤 5，Flink CDC 并没有做非常耗时的任务，所以全局锁定的时间很短，这样对业务运行的影响可以尽量降至最小。</p>
</li>
<li data-nodeid="66">
<p data-nodeid="67">将步骤 4 读取的数据库和表定义，作用到目标数据库上。</p>
</li>
<li data-nodeid="68">
<p data-nodeid="69">对数据库里的表进行全表扫描，将读取出的每条记录，都发送到目标数据库。</p>
</li>
<li data-nodeid="70">
<p data-nodeid="71">完成全表扫描后，提交（commit）步骤 2 时启动的可重复读语义事务。</p>
</li>
<li data-nodeid="72">
<p data-nodeid="73">将步骤 3 读取的 binlog 位置记录下来，表明本次数据全量同步过程（也就是“快照”）成功完成。后续做增量同步时，如果发现没有这个 binlog 位置记录，就意味着数据全量同步过程是失败的，可以重新再做一次步骤 1 到步骤 9，直到全量同步成功为止。</p>
</li>
</ol>
<p data-nodeid="74">可以看出，数据全量同步的过程还是比较复杂的，但好在 Flink CDC 的<a href="https://github.com/ververica/flink-cdc-connectors/wiki/MySQL-CDC-Connector?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="215">flink-connector-mysql-cdc 连接器插件</a>已经为我们实现了这个过程，所以我们直接使用它就好了。</p>
<p data-nodeid="75">完成数据全量同步后，后面的<strong data-nodeid="226">增量同步</strong>过程就相对简单了，直接跟随源数据库的 binlog 日志，然后将每次的数据变更同步到目标数据库即可。增量同步过程中，Flink 自己会<strong data-nodeid="227">周期性地执行 checkpoint 操作</strong>，从而记录下当时增量同步到的 binlog 位置。</p>
<p data-nodeid="76">这样，如果 Flink CDC 作业（job）因为发生故障而重启的话，也能够从最近一次 checkpoint 处，<strong data-nodeid="233">恢复出故障发生前的状态</strong>，从而继续执行之前的过程。</p>
<p data-nodeid="77">最后，再配合<strong data-nodeid="243">写入目标数据库时是幂等性的操作</strong>。这样，就保证了 Flink CDC 的整个数据同步过程，<strong data-nodeid="244">能够达到 exactly once 级别的数据处理可靠性</strong>。是不是非常惊艳！</p>
<p data-nodeid="78">以上就是 Flink CDC 的工作原理。接下来，我们就具体展示下，如何使用 Flink CDC 技术，进行实时数据同步。</p>
<h3 data-nodeid="79">具体实现</h3>
<p data-nodeid="80">我们这里以将 MySQL 里的数据实时同步到 Elasticsearch 为例。Flink CDC 的具体实现方式有两种，一种是基于 DataStream（<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/connectors/elasticsearch.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="250">参见链接</a>）的方式，另一种是基于 Table &amp; SQL（<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/elasticsearch.html?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="256">参见链接</a>）的方式。我们说的 Flink SQL CDC 就是指基于 Table &amp; SQL 方式的 Flink CDC 实现。</p>
<p data-nodeid="81">由于 Flink SQL 在经过解析之后，最终也会转化为基于 DataStream 的底层代码。所以我先演示直接使用 DataStream 的方式，然后再演示使用 SQL 的方式，这样更加有助于理解。</p>
<h4 data-nodeid="82">基于 DataStream 的方式</h4>
<p data-nodeid="83">我们先来看基于 DataStream 的方式。具体代码如下（<a href="https://github.com/alain898/realtime_stream_computing_course/tree/main/course21?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="265">完整代码和操作说明参看这里</a>）。</p>
<pre class="lang-java" data-nodeid="84"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FlinkCdcDemo</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 源数据库，下面是以 MySQL 作为源数据库的配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; SourceFunction&lt;String&gt; sourceFunction = MySQLSource.&lt;String&gt;builder()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .hostname(<span class="hljs-string">"127.0.0.1"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .port(<span class="hljs-number">3306</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .databaseList(<span class="hljs-string">"db001"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .username(<span class="hljs-string">"root"</span>)&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 测试用，生产不要用root账号！</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .password(<span class="hljs-string">"123456"</span>)&nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 测试用，生产不要用这种简单密码！</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .deserializer(<span class="hljs-keyword">new</span> StringDebeziumDeserializationSchema())
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 目标数据库，下面是以 Elasticsearch 作为目标数据库的配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;HttpHost&gt; httpHosts = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; httpHosts.add(<span class="hljs-keyword">new</span> HttpHost(<span class="hljs-string">"127.0.0.1"</span>, <span class="hljs-number">9200</span>, <span class="hljs-string">"http"</span>));
&nbsp; &nbsp; &nbsp; &nbsp; ElasticsearchSink.Builder&lt;String&gt; esSinkBuilder = <span class="hljs-keyword">new</span> ElasticsearchSink.Builder&lt;&gt;(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; httpHosts,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> ElasticsearchSinkFunction&lt;String&gt;() {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> IndexRequest <span class="hljs-title">createIndexRequest</span><span class="hljs-params">(String element)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Map&lt;String, String&gt; json = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
                        <span class="hljs-comment">// 这里直接将数据 element 表示为字符串</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; json.put(<span class="hljs-string">"data"</span>, element);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Requests.indexRequest()
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .index(<span class="hljs-string">"table001"</span>)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .source(json);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">process</span><span class="hljs-params">(String element, RuntimeContext ctx, RequestIndexer indexer)</span> </span>{
                        <span class="hljs-comment">// 这里就是将数据同步到目标数据库 Elasticsearch</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; indexer.add(createIndexRequest(element));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; );
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 实验时配置逐条插入，生产为了提升性能的话，可以改为批量插入</span>
&nbsp; &nbsp; &nbsp; &nbsp; esSinkBuilder.setBulkFlushMaxActions(<span class="hljs-number">1</span>);
&nbsp; &nbsp; &nbsp; &nbsp; StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
&nbsp; &nbsp; &nbsp; &nbsp; env
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addSource(sourceFunction)&nbsp; <span class="hljs-comment">// 设置源数据库</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .addSink(esSinkBuilder.build())&nbsp; <span class="hljs-comment">// 设置目标数据库</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .setParallelism(<span class="hljs-number">1</span>); <span class="hljs-comment">// 设置并行度为1，以保持消息顺序</span>
&nbsp; &nbsp; &nbsp; &nbsp; env.execute(<span class="hljs-string">"FlinkCdcDemo"</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3341">在上面的代码中，我们首先使用 MySQLSource 类配置了一个 MySQL 源数据库。然后，我们再使用 ElasticsearchSink 类配置了一个 Elasticsearch 目标数据库。最后，我们使用 addSource 和 addSink 函数，将源数据库和目标数据库之间的数据同步链路打通。这样，我们就实现了基于 DataStream 的 Flink CDC 实时数据同步功能。</p>
<p data-nodeid="3342">可以看到，上面的代码还是非常简单的。主要的原因在于 MySQLSource 这个 CDC Connector 为我们封装了所有的复杂操作，这些复杂操作就包括我们在实现原理部分讲到的“全量同步”和“增量同步”的实现细节。</p>

<p data-nodeid="86">不过，上面的代码并不完美，它主要有两个问题：</p>
<ol data-nodeid="87">
<li data-nodeid="88">
<p data-nodeid="89">一是，同步数据时，写入 Elasticsearch 的数据是字符串，而不是经过解析的各个独立字段。这样就会导致很多没有用的字段也保存到了 Elasticsearch ，并且后续查询的效率会非常低。</p>
</li>
<li data-nodeid="90">
<p data-nodeid="91">二是，还是需要写一些代码。虽然上面的代码并不复杂，但是毕竟还是没有 SQL 方便。</p>
</li>
</ol>
<p data-nodeid="92">为了解决以上两个问题，接下来我们就使用 Table &amp; SQL&nbsp;的方式来实现 Flink CDC 功能，这就是 Flink SQL CDC。你会发现，我们真的就只需要写几行 SQL 语句，就能轻松解决上面两个问题。</p>
<h4 data-nodeid="93">基于 Table &amp; SQL&nbsp;方式</h4>
<p data-nodeid="94">下面就是基于 Table &amp; SQL&nbsp;方式实现 Flink CDC 功能的代码（<a href="https://github.com/alain898/realtime_stream_computing_course/tree/main/course21?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="284">完整代码和操作说明参看这里</a>）。</p>
<pre class="lang-sql" data-nodeid="95"><code data-language="sql"><span class="hljs-comment">-- 在 Flink SQL Client 里执行以下 SQL。</span>
<span class="hljs-comment">-- 创建源数据库</span>
<span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">TABLE</span> sourceTable (
&nbsp; <span class="hljs-keyword">id</span> <span class="hljs-built_in">INT</span>,
&nbsp; <span class="hljs-keyword">name</span> <span class="hljs-keyword">STRING</span>,
&nbsp; counts <span class="hljs-built_in">INT</span>,
&nbsp; description <span class="hljs-keyword">STRING</span>
) <span class="hljs-keyword">WITH</span> (
&nbsp;<span class="hljs-string">'connector'</span> = <span class="hljs-string">'mysql-cdc'</span>,
&nbsp;<span class="hljs-string">'hostname'</span> = <span class="hljs-string">'192.168.1.7'</span>,
&nbsp;<span class="hljs-string">'port'</span> = <span class="hljs-string">'3306'</span>,
&nbsp;<span class="hljs-string">'username'</span> = <span class="hljs-string">'root'</span>,
&nbsp;<span class="hljs-string">'password'</span> = <span class="hljs-string">'123456'</span>,
&nbsp;<span class="hljs-string">'database-name'</span> = <span class="hljs-string">'db001'</span>,
&nbsp;<span class="hljs-string">'table-name'</span> = <span class="hljs-string">'table001'</span>
);
<span class="hljs-comment">-- 创建目标数据库</span>
<span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">TABLE</span> sinkTable (
&nbsp; <span class="hljs-keyword">id</span> <span class="hljs-built_in">INT</span>,
&nbsp; <span class="hljs-keyword">name</span> <span class="hljs-keyword">STRING</span>,
&nbsp; counts <span class="hljs-built_in">INT</span>
) <span class="hljs-keyword">WITH</span> (
&nbsp; <span class="hljs-string">'connector'</span> = <span class="hljs-string">'elasticsearch-7'</span>,
&nbsp; <span class="hljs-string">'hosts'</span> = <span class="hljs-string">'http://192.168.1.7:9200'</span>,
&nbsp; <span class="hljs-string">'index'</span> = <span class="hljs-string">'table001'</span>
);
<span class="hljs-comment">-- 启动 Flink SQL CDC 作业</span>
<span class="hljs-keyword">insert</span> <span class="hljs-keyword">into</span> sinkTable <span class="hljs-keyword">select</span> <span class="hljs-keyword">id</span>, <span class="hljs-keyword">name</span>, counts <span class="hljs-keyword">from</span> sourceTable;
</code></pre>
<p data-nodeid="3925">看到没！只需要简简单单三个 SQL 语句，就实现了 Flink CDC 的功能。</p>
<p data-nodeid="3926">其中，CREATE TABLE sourceTable 是在配置一个 MySQL 源数据库，CREATE TABLE sinkTable 是在配置一个 Elasticsearch 目标数据库。而 insert into select from 则是指定了需要从源数据库向目标数据库<strong data-nodeid="3933">同步哪些字段</strong>，并且它会触发启动这个 Flink CDC 作业。</p>

<p data-nodeid="97">当启动 Flink CDC 作业后，如果我们向 MySQL 写入数据，你就可以看到数据从 MySQL 同步到 Elasticsearch 的效果了。</p>
<p data-nodeid="98">下面的图 4 就是 Flink CDC 数据同步的效果图。</p>
<p data-nodeid="5100"><img src="https://s0.lgstatic.com/i/image6/M01/2D/90/Cgp9HWBmt-WAB1cbABNhEi6mLVE369.png" alt="Drawing 6.png" data-nodeid="5104"></p>
<div data-nodeid="5101" class=""><p style="text-align:center">图 4 使用 Flink CDC 实时同步数据的效果图</p></div>



<p data-nodeid="101">从上面的图 4 可以看出，左边源数据库 MySQL 里的数据和右边目标数据库 Elasticsearch 里的数据是完全对应的。并且，同步到 Elasticsearch 里数据的字段，也是和我们在 insert into select from 语句里指定的字段是完全一致的。你看，Flink SQL CDC 实现实时数据同步的效果是不是很不错！</p>
<p data-nodeid="102">最后还需要说明下的是，这里我为了专注于讲解 Flink CDC 的工作原理本身，就使用了相对简单的 SQL 语句。其实，Flink SQL CDC 是可以使用一些更加复杂的 SQL 语句，来实现更加丰富的数据同步功能的。比如，使用 GROUP BY 分组和使用 Window 进行窗口计算等。对于这些更完整和更复杂的 Flink SQL 语句说明，你可以参考<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/sql/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="304">这里的官方文档</a>。</p>
<h3 data-nodeid="103">小结</h3>
<p data-nodeid="104">总的来说，相比 DataStream 的方式，Flink SQL CDC 使用起来会更加方便些。但这两种方式我们都需要掌握，因为目前 Flink SQL CDC 还不算非常成熟，一些 Flink SQL 暂时不支持的功能和插件，还是需要我们自己基于 DataStream 在底层实现。</p>
<p data-nodeid="105">你的工作中有没有可以使用到 Flink CDC，或者用 Flink CDC 进行改造的场景呢？可以将你的想法或问题写在留言区。</p>
<p data-nodeid="106">下面是本课时的知识脑图。</p>
<p data-nodeid="5683" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/2D/90/Cgp9HWBmt_uANir5AAfXEAo3ILg905.png" alt="Drawing 7.png" data-nodeid="5686"></p>

---

### 精选评论

##### **1605：
> 想到一个场景，数据库同步到数据仓库可以用Flink cdc

##### **谦：
> 使用flinkcdc如果源表数据量过大，同步全量数据的时候会查询超时，这个有什么解决办法吗？可以根据指定字段分片吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 根据你说的情况，我不能确定你说的是哪种超时。如果是MySQL本身的查询超时的话，可能是max_execution_time参数设置的太短了，可以在全表扫描阶段临时改大些，等全表扫描完成后再改回来。如果是Flink checkpoint超时的话，可以将checkpoint的时间间隔设置长一些，失败容忍次数也可以调高些，同时配置失败重启策略为fixed-delay，将失败重启尝试次数设置得大一些。另外，是可以通过Flink SQL中的PARTITIONED BY来指定分片字段的。比如，如果是sink到文件系统里，那么每个partition就会对应一个单独的目录。

