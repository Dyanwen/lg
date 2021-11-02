<p data-nodeid="985" class="">本课时我们主要介绍 Flink 的 DataSet 和 DataStream 的 API，并模拟了实时计算的场景，详细讲解了 DataStream 常用的 API 的使用。</p>
<h3 data-nodeid="986">说好的流批一体呢</h3>
<h4 data-nodeid="987">现状</h4>
<p data-nodeid="988">在前面的课程中，曾经提到过，Flink 很重要的一个特点是“流批一体”，然而事实上 Flink 并没有完全做到所谓的“流批一体”，即编写一套代码，可以同时支持流式计算场景和批量计算的场景。目前截止 1.10 版本依然采用了 DataSet 和 DataStream 两套 API 来适配不同的应用场景。</p>
<h4 data-nodeid="1231" class="te-preview-highlight">DataSet 和 DataStream 的区别和联系</h4>

<p data-nodeid="990">在官网或者其他网站上，都可以找到目前 Flink 支持两套 API 和一些应用场景，但大都缺少了“为什么”这样的思考。</p>
<p data-nodeid="991">Apache Flink 在诞生之初的设计哲学是：<strong data-nodeid="1083">用同一个引擎支持多种形式的计算，包括批处理、流处理和机器学习等</strong>。尤其是在流式计算方面，<strong data-nodeid="1084">Flink 实现了计算引擎级别的流批一体</strong>。那么对于普通开发者而言，如果使用原生的 Flink ，直接的感受还是要编写两套代码。</p>
<p data-nodeid="992">整体架构如下图所示：</p>
<p data-nodeid="993"><img src="https://s0.lgstatic.com/i/image3/M01/14/67/Ciqah16hTJCARnCYAALXFI10sJU200.png" alt="image.png" data-nodeid="1088"><br>
在 Flink 的源代码中，我们可以在 flink-java 这个模块中找到所有关于 DataSet 的核心类，DataStream 的核心实现类则在 flink-streaming-java 这个模块。</p>
<p data-nodeid="994"><img src="https://s0.lgstatic.com/i/image3/M01/07/39/CgoCgV6hTRuAdaYYAAfiA9_tU84430.png" alt="image (1).png" data-nodeid="1093"></p>
<p data-nodeid="995"><img src="https://s0.lgstatic.com/i/image3/M01/14/68/Ciqah16hTSOAaofrAAd_Hyp6Zuw422.png" alt="image (2).png" data-nodeid="1096"></p>
<p data-nodeid="996">在上述两张图中，我们分别打开 DataSet 和 DataStream 这两个类，可以发现，二者支持的 API 都非常丰富且十分类似，比如常用的 map、filter、join 等常见的 transformation 函数。</p>
<p data-nodeid="997">我们在前面的课时中讲过 Flink 的编程模型，对于 DataSet 而言，Source 部分来源于文件、表或者 Java 集合；而 DataStream 的 Source 部分则一般是消息中间件比如 Kafka 等。</p>
<p data-nodeid="998">由于 Flink DataSet 和 DataStream API 的高度相似，并且 Flink 在实时计算领域中应用的更为广泛。所以下面我们详细讲解 DataStream API 的使用。</p>
<h3 data-nodeid="999">DataStream</h3>
<p data-nodeid="1000">我们先来回顾一下 Flink 的编程模型，在之前的课时中提到过，Flink 程序的基础构建模块是<strong data-nodeid="1122">流</strong>（Streams）和<strong data-nodeid="1123">转换</strong>（Transformations），每一个数据流起始于一个或多个 <strong data-nodeid="1124">Source</strong>，并终止于一个或多个 <strong data-nodeid="1125">Sink</strong>。数据流类似于<strong data-nodeid="1126">有向无环图</strong>（DAG）。</p>
<p data-nodeid="1001"><img src="https://s0.lgstatic.com/i/image3/M01/14/68/Ciqah16hTWOAYItJAADWU6-1xbw110.png" alt="image (3).png" data-nodeid="1129"></p>
<p data-nodeid="1002">在第 02 课时中模仿了一个流式计算环境，我们选择监听一个本地的 Socket 端口，并且使用 Flink 中的滚动窗口，每 5 秒打印一次计算结果。</p>
<h4 data-nodeid="1003">自定义实时数据源</h4>
<p data-nodeid="1004">在本课时中，我们利用 Flink 提供的自定义 Source 功能来实现一个自定义的实时数据源，具体实现如下：</p>
<pre class="lang-java" data-nodeid="1005"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyStreamingSource</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SourceFunction</span>&lt;<span class="hljs-title">MyStreamingSource</span>.<span class="hljs-title">Item</span>&gt; </span>{

    <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> isRunning = <span class="hljs-keyword">true</span>;

    <span class="hljs-comment">/**
     * 重写run方法产生一个源源不断的数据发送源
     * <span class="hljs-doctag">@param</span> ctx
     * <span class="hljs-doctag">@throws</span> Exception
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(SourceContext&lt;Item&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">while</span>(isRunning){
            Item item = generateItem();
            ctx.collect(item);

            <span class="hljs-comment">//每秒产生一条数据</span>
            Thread.sleep(<span class="hljs-number">1000</span>);
        }
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">cancel</span><span class="hljs-params">()</span> </span>{
        isRunning = <span class="hljs-keyword">false</span>;
    }

    <span class="hljs-comment">//随机产生一条商品数据</span>
    <span class="hljs-function"><span class="hljs-keyword">private</span> Item <span class="hljs-title">generateItem</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">int</span> i = <span class="hljs-keyword">new</span> Random().nextInt(<span class="hljs-number">100</span>);

        Item item = <span class="hljs-keyword">new</span> Item();
        item.setName(<span class="hljs-string">"name"</span> + i);
        item.setId(i);
        <span class="hljs-keyword">return</span> item;
    }

    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Item</span></span>{
        <span class="hljs-keyword">private</span> String name;
        <span class="hljs-keyword">private</span> Integer id;

        Item() {
        }

        <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getName</span><span class="hljs-params">()</span> </span>{
            <span class="hljs-keyword">return</span> name;
        }

        <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">setName</span><span class="hljs-params">(String name)</span> </span>{
            <span class="hljs-keyword">this</span>.name = name;
        }

        <span class="hljs-function"><span class="hljs-keyword">private</span> Integer <span class="hljs-title">getId</span><span class="hljs-params">()</span> </span>{
            <span class="hljs-keyword">return</span> id;
        }

        <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">setId</span><span class="hljs-params">(Integer id)</span> </span>{
            <span class="hljs-keyword">this</span>.id = id;
        }

        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-string">"Item{"</span> +
                    <span class="hljs-string">"name='"</span> + name + <span class="hljs-string">'\''</span> +
                    <span class="hljs-string">", id="</span> + id +
                    <span class="hljs-string">'}'</span>;
        }
    }
}


<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StreamingDemo</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        <span class="hljs-comment">//获取数据源</span>
        DataStreamSource&lt;MyStreamingSource.Item&gt; text = 
        <span class="hljs-comment">//注意：并行度设置为1,我们会在后面的课程中详细讲解并行度</span>
        env.addSource(<span class="hljs-keyword">new</span> MyStreamingSource()).setParallelism(<span class="hljs-number">1</span>); 
        DataStream&lt;MyStreamingSource.Item&gt; item = text.map(
                (MapFunction&lt;MyStreamingSource.Item, MyStreamingSource.Item&gt;) value -&gt; value);

        <span class="hljs-comment">//打印结果</span>
        item.print().setParallelism(<span class="hljs-number">1</span>);
        String jobName = <span class="hljs-string">"user defined streaming source"</span>;
        env.execute(jobName);
    }

}
</code></pre>
<p data-nodeid="1006">在自定义的数据源中，实现了 Flink 中的 SourceFunction 接口，同时实现了其中的 run 方法，在 run 方法中每隔一秒钟随机发送一个自定义的 Item。</p>
<p data-nodeid="1007">可以直接运行 main 方法来进行测试：</p>
<p data-nodeid="1008"><img src="https://s0.lgstatic.com/i/image3/M01/14/69/Ciqah16hTgOAZCaLAAcArI5AjtQ208.png" alt="image (4).png" data-nodeid="1137"></p>
<p data-nodeid="1009">可以在控制台中看到，已经有源源不断地数据开始输出。下面我们就用自定义的实时数据源来演示 DataStream API 的使用。</p>
<h4 data-nodeid="1010">Map</h4>
<p data-nodeid="1011">Map 接受一个元素作为输入，并且根据开发者自定义的逻辑处理后输出。</p>
<p data-nodeid="1012"><img src="https://s0.lgstatic.com/i/image3/M01/14/69/Ciqah16hThSAdYzhAADDHstaa9E625.png" alt="image (5).png" data-nodeid="1143"></p>
<pre class="lang-java" data-nodeid="1013"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StreamingDemo</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        <span class="hljs-comment">//获取数据源</span>
        DataStreamSource&lt;MyStreamingSource.Item&gt; items = env.addSource(<span class="hljs-keyword">new</span> MyStreamingSource()).setParallelism(<span class="hljs-number">1</span>); 
        <span class="hljs-comment">//Map</span>
        SingleOutputStreamOperator&lt;Object&gt; mapItems = items.map(<span class="hljs-keyword">new</span> MapFunction&lt;MyStreamingSource.Item, Object&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">map</span><span class="hljs-params">(MyStreamingSource.Item item)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                <span class="hljs-keyword">return</span> item.getName();
            }
        });
        <span class="hljs-comment">//打印结果</span>
        mapItems.print().setParallelism(<span class="hljs-number">1</span>);
        String jobName = <span class="hljs-string">"user defined streaming source"</span>;
        env.execute(jobName);
    }
}
</code></pre>
<p data-nodeid="1014">我们只取出每个 Item 的 name 字段进行打印。</p>
<p data-nodeid="1015"><img src="https://s0.lgstatic.com/i/image3/M01/07/3A/CgoCgV6hTiuABREkAARA23HrkOQ888.png" alt="image (6).png" data-nodeid="1147"></p>
<p data-nodeid="1016">注意，<strong data-nodeid="1153">Map 算子是最常用的算子之一</strong>，官网中的表述是对一个 DataStream 进行映射，每次进行转换都会调用 MapFunction 函数。从源 DataStream 到目标 DataStream 的转换过程中，返回的是 SingleOutputStreamOperator。当然了，我们也可以在重写的 map 函数中使用 lambda 表达式。</p>
<pre class="lang-java" data-nodeid="1017"><code data-language="java">SingleOutputStreamOperator&lt;Object&gt; mapItems = items.map(
      item -&gt; item.getName()
);
</code></pre>
<p data-nodeid="1018">甚至，还可以自定义自己的 Map 函数。通过重写 MapFunction 或 RichMapFunction 来自定义自己的 map 函数。</p>
<pre class="lang-java" data-nodeid="1019"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">StreamingDemo</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        <span class="hljs-comment">//获取数据源</span>
        DataStreamSource&lt;MyStreamingSource.Item&gt; items = env.addSource(<span class="hljs-keyword">new</span> MyStreamingSource()).setParallelism(<span class="hljs-number">1</span>);
        SingleOutputStreamOperator&lt;String&gt; mapItems = items.map(<span class="hljs-keyword">new</span> MyMapFunction());
        <span class="hljs-comment">//打印结果</span>
        mapItems.print().setParallelism(<span class="hljs-number">1</span>);
        String jobName = <span class="hljs-string">"user defined streaming source"</span>;
        env.execute(jobName);
    }

    <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyMapFunction</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichMapFunction</span>&lt;<span class="hljs-title">MyStreamingSource</span>.<span class="hljs-title">Item</span>,<span class="hljs-title">String</span>&gt; </span>{

        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">map</span><span class="hljs-params">(MyStreamingSource.Item item)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            <span class="hljs-keyword">return</span> item.getName();
        }
    }
}
</code></pre>
<p data-nodeid="1020">此外，在 RichMapFunction 中还提供了 open、close 等函数方法，重写这些方法还能实现更为复杂的功能，比如获取累加器、计数器等。</p>
<h4 data-nodeid="1021">FlatMap</h4>
<p data-nodeid="1022">FlatMap 接受一个元素，返回零到多个元素。FlatMap 和 Map 有些类似，但是当返回值是列表的时候，FlatMap 会将列表“平铺”，也就是以单个元素的形式进行输出。</p>
<pre class="lang-java" data-nodeid="1023"><code data-language="java">SingleOutputStreamOperator&lt;Object&gt; flatMapItems = items.flatMap(<span class="hljs-keyword">new</span> FlatMapFunction&lt;MyStreamingSource.Item, Object&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flatMap</span><span class="hljs-params">(MyStreamingSource.Item item, Collector&lt;Object&gt; collector)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        String name = item.getName();
        collector.collect(name);
    }
});
</code></pre>
<p data-nodeid="1024">上面的程序会把名字逐个输出。我们也可以在 FlatMap 中实现更为复杂的逻辑，比如过滤掉一些我们不需要的数据等。</p>
<h4 data-nodeid="1025">Filter</h4>
<p data-nodeid="1026">顾名思义，Fliter 的意思就是过滤掉不需要的数据，每个元素都会被 filter 函数处理，如果 filter 函数返回 true 则保留，否则丢弃。</p>
<p data-nodeid="1027"><img src="https://s0.lgstatic.com/i/image3/M01/14/6A/Ciqah16hTtiAPWhJAADGVua1-cc867.png" alt="image (7).png" data-nodeid="1163"></p>
<p data-nodeid="1028">例如，我们只保留 id 为偶数的那些 item。</p>
<pre class="lang-java" data-nodeid="1029"><code data-language="java">SingleOutputStreamOperator&lt;MyStreamingSource.Item&gt; filterItems = items.filter(<span class="hljs-keyword">new</span> FilterFunction&lt;MyStreamingSource.Item&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(MyStreamingSource.Item item)</span> <span class="hljs-keyword">throws</span> Exception </span>{

        <span class="hljs-keyword">return</span> item.getId() % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>;
    }
});
</code></pre>
<p data-nodeid="1030"><img src="https://s0.lgstatic.com/i/image3/M01/07/3B/CgoCgV6hTuWALqJwAATDpDg9dpY638.png" alt="image (8).png" data-nodeid="1167"></p>
<p data-nodeid="1031">当然，我们也可以在 filter 中使用 lambda 表达式：</p>
<pre class="lang-java" data-nodeid="1032"><code data-language="java">SingleOutputStreamOperator&lt;MyStreamingSource.Item&gt; filterItems = items.filter( 
    item -&gt; item.getId() % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>
);
</code></pre>
<h4 data-nodeid="1033">KeyBy</h4>
<p data-nodeid="1034">在介绍 KeyBy 函数之前，需要你理解一个概念：<strong data-nodeid="1175">KeyedStream</strong>。 在实际业务中，我们经常会需要根据数据的某种属性或者单纯某个字段进行分组，然后对不同的组进行不同的处理。举个例子，当我们需要描述一个用户画像时，则需要根据用户的不同行为事件进行加权；再比如，我们在监控双十一的交易大盘时，则需要按照商品的品类进行分组，分别计算销售额。</p>
<p data-nodeid="1035"><img src="https://s0.lgstatic.com/i/image3/M01/07/3B/CgoCgV6hTzyAUKHxAAF12IHd3bQ582.png" alt="image (9).png" data-nodeid="1178"></p>
<p data-nodeid="1036">我们在使用 KeyBy 函数时会把 DataStream 转换成为 KeyedStream，事实上 KeyedStream 继承了 DataStream，KeyedStream 中的元素会根据用户传入的参数进行分组。</p>
<p data-nodeid="1037">我们在第 02 课时中讲解的 WordCount 程序，曾经使用过 KeyBy：</p>
<pre class="lang-java" data-nodeid="1038"><code data-language="java">    <span class="hljs-comment">// 将接收的数据进行拆分，分组，窗口计算并且进行聚合输出</span>
        DataStream&lt;WordWithCount&gt; windowCounts = text
                .flatMap(<span class="hljs-keyword">new</span> FlatMapFunction&lt;String, WordWithCount&gt;() {
                    <span class="hljs-meta">@Override</span>
                    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flatMap</span><span class="hljs-params">(String value, Collector&lt;WordWithCount&gt; out)</span> </span>{
                        <span class="hljs-keyword">for</span> (String word : value.split(<span class="hljs-string">"\\s"</span>)) {
                            out.collect(<span class="hljs-keyword">new</span> WordWithCount(word, <span class="hljs-number">1L</span>));
                        }
                    }
                })
                .keyBy(<span class="hljs-string">"word"</span>)
                .timeWindow(Time.seconds(<span class="hljs-number">5</span>), Time.seconds
                ....
</code></pre>
<p data-nodeid="1039">在生产环境中使用 KeyBy 函数时要十分注意！该函数会把数据按照用户指定的 key 进行分组，那么相同分组的数据会被分发到一个 subtask 上进行处理，在大数据量和 key 分布不均匀的时非常容易出现数据倾斜和反压，导致任务失败。</p>
<p data-nodeid="1040"><img src="https://s0.lgstatic.com/i/image3/M01/14/6B/Ciqah16hT2CAUq4YAAIFumFqfTg398.png" alt="image (10).png" data-nodeid="1184"></p>
<p data-nodeid="1041">常见的解决方式是把所有<strong data-nodeid="1190">数据加上随机前后缀</strong>，这些我们会在后面的课时中进行深入讲解。</p>
<h4 data-nodeid="1042">Aggregations</h4>
<p data-nodeid="1043">Aggregations 为聚合函数的总称，常见的聚合函数包括但不限于 sum、max、min 等。Aggregations 也需要指定一个 key 进行聚合，官网给出了几个常见的例子：</p>
<pre class="lang-java" data-nodeid="1044"><code data-language="java">keyedStream.sum(<span class="hljs-number">0</span>);
keyedStream.sum(<span class="hljs-string">"key"</span>);
keyedStream.min(<span class="hljs-number">0</span>);
keyedStream.min(<span class="hljs-string">"key"</span>);
keyedStream.max(<span class="hljs-number">0</span>);
keyedStream.max(<span class="hljs-string">"key"</span>);
keyedStream.minBy(<span class="hljs-number">0</span>);
keyedStream.minBy(<span class="hljs-string">"key"</span>);
keyedStream.maxBy(<span class="hljs-number">0</span>);
keyedStream.maxBy(<span class="hljs-string">"key"</span>);
</code></pre>
<p data-nodeid="1045">在上面的这几个函数中，max、min、sum 会分别返回最大值、最小值和汇总值；而 minBy 和 maxBy 则会把最小或者最大的元素全部返回。</p>
<p data-nodeid="1046">我们拿 max 和 maxBy 举例说明：</p>
<pre class="lang-java" data-nodeid="1047"><code data-language="java">StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
<span class="hljs-comment">//获取数据源</span>
List data = <span class="hljs-keyword">new</span> ArrayList&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt;();
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">0</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">1</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>,<span class="hljs-number">2</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">3</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">5</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">9</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">11</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">13</span>));

DataStreamSource&lt;MyStreamingSource.Item&gt; items = env.fromCollection(data);
items.keyBy(<span class="hljs-number">0</span>).max(<span class="hljs-number">2</span>).printToErr();

<span class="hljs-comment">//打印结果</span>
String jobName = <span class="hljs-string">"user defined streaming source"</span>;
env.execute(jobName);
</code></pre>
<p data-nodeid="1048">我们直接运行程序，会发现奇怪的一幕：</p>
<p data-nodeid="1049"><img src="https://s0.lgstatic.com/i/image3/M01/07/3C/CgoCgV6hT9SATGmCAATvGBf2FXg156.png" alt="image (11).png" data-nodeid="1198"></p>
<p data-nodeid="1050">从上图中可以看到，我们希望按照 Tuple3 的第一个元素进行聚合，并且按照第三个元素取最大值。结果如我们所料，的确是按照第三个元素大小依次进行的打印，但是结果却出现了一个这样的元素 (0,1,2)，这在我们的源数据中并不存在。</p>
<p data-nodeid="1051">我们在 Flink 官网中的文档可以发现：</p>
<blockquote data-nodeid="1052">
<p data-nodeid="1053">The difference between min and minBy is that min returns the minimum value, whereas minBy returns the element that has the minimum value in this field (same for max and maxBy).</p>
</blockquote>
<p data-nodeid="1054">文档中说：<strong data-nodeid="1207">min 和 minBy 的区别在于，min 会返回我们制定字段的最大值，minBy 会返回对应的元素（max 和 maxBy 同理）</strong>。</p>
<p data-nodeid="1055">网上很多资料也这么写：min 和 minBy 的区别在于 min 返回最小的值，而 minBy 返回最小值的key，严格来说这是不正确的。</p>
<p data-nodeid="1056">min 和 minBy 都会返回整个元素，只是 min 会根据用户指定的字段取最小值，并且把这个值保存在对应的位置，而对于其他的字段，并不能保证其数值正确。max 和 maxBy 同理。</p>
<p data-nodeid="1057">事实上，对于 Aggregations 函数，Flink 帮助我们封装了状态数据，这些状态数据不会被清理，所以在实际生产环境中应该<strong data-nodeid="1215">尽量避免在一个无限流上使用 Aggregations</strong>。而且，对于同一个 keyedStream ，只能调用一次 Aggregation 函数。</p>
<h4 data-nodeid="1058">Reduce</h4>
<p data-nodeid="1059">Reduce 函数的原理是，会在每一个分组的 keyedStream 上生效，它会按照用户自定义的聚合逻辑进行分组聚合。<br>
<img src="https://s0.lgstatic.com/i/image3/M01/07/3C/CgoCgV6hUAyAIaCwAAGkybBDznc114.png" alt="image (12).png" data-nodeid="1221"></p>
<p data-nodeid="1060">例如：</p>
<pre class="lang-java" data-nodeid="1061"><code data-language="java">List data = <span class="hljs-keyword">new</span> ArrayList&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt;();
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">0</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">1</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>,<span class="hljs-number">2</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">3</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">5</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">9</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">11</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">13</span>));

DataStreamSource&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; items = env.fromCollection(data);
<span class="hljs-comment">//items.keyBy(0).max(2).printToErr();</span>

SingleOutputStreamOperator&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; reduce = items.keyBy(<span class="hljs-number">0</span>).reduce(<span class="hljs-keyword">new</span> ReduceFunction&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Tuple3&lt;Integer,Integer,Integer&gt; <span class="hljs-title">reduce</span><span class="hljs-params">(Tuple3&lt;Integer, Integer, Integer&gt; t1, Tuple3&lt;Integer, Integer, Integer&gt; t2)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        Tuple3&lt;Integer,Integer,Integer&gt; newTuple = <span class="hljs-keyword">new</span> Tuple3&lt;&gt;();

        newTuple.setFields(<span class="hljs-number">0</span>,<span class="hljs-number">0</span>,(Integer)t1.getField(<span class="hljs-number">2</span>) + (Integer) t2.getField(<span class="hljs-number">2</span>));
        <span class="hljs-keyword">return</span> newTuple;
    }
});

reduce.printToErr().setParallelism(<span class="hljs-number">1</span>);
</code></pre>
<p data-nodeid="1062">我们对下面的元素按照第一个元素进行分组，第三个元素分别求和，并且把第一个和第二个元素都置为 0：</p>
<pre class="lang-java" data-nodeid="1063"><code data-language="java">data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">0</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">1</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>,<span class="hljs-number">2</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">0</span>,<span class="hljs-number">1</span>,<span class="hljs-number">3</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">5</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">9</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">11</span>));
data.add(<span class="hljs-keyword">new</span> Tuple3&lt;&gt;(<span class="hljs-number">1</span>,<span class="hljs-number">2</span>,<span class="hljs-number">13</span>));
</code></pre>
<p data-nodeid="1064">那么最终会得到：(0,0,6) 和 (0,0,38)。</p>
<h3 data-nodeid="1065">总结</h3>
<p data-nodeid="1066">这一课时介绍了常用的 API 操作，事实上 DataStream 的 API 远远不止这些，我们在看官方文档的时候要动手去操作验证一下，更为高级的 API 将会在实战课中用到的时候着重进行讲解。</p>
<p data-nodeid="1067" class=""><a href="https://github.com/wangzhiwubigdata/quickstart" data-nodeid="1229">点击这里下载本课程源码</a>。</p>

---

### 精选评论

##### *尖：
> flink的拓扑结构可以有环

##### **保：
> 抛砖引玉

##### *鑫：
> 老师 当我把代码改为 <font color="#a9b7c6" face="Consolas"><span style="font-size: 18px;">items.keyBy(1).maxBy(2).printToErr();&nbsp; 发现结果连keyBy都不能保证正确了，这是啥情况呢</span></font>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 分组的时候跟keyby没关系，只看你用的max还是maxby

##### **宾：
> 这个filter是不是也可以用旁路输出替代？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的。可以灵活根据实际需要进行使用。

##### **成：
> 老师,您好! 下载完源码之后，pom.xml报错，包导不下来，求助！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 跟本地网络有关，目前的这些包都是Maven仓库有的。你可以把本地仓库先清空一下，重新导入。

##### *凡：
> reduce我也是返回(0,1,0)(0,0,1)(0,0,3)(0,0,6)(1,2,5)(0,0,14)(0,0,25)(0,0,38)我理解第一个前面由于没有计算的用默认的，后面就是按对应的计算方式(0,1,0)为第一次计算，第二次就是（0,1,0）(0,1,1)前两位置0，第三位相加（0,0,1）,然后（0,0,1）与（0,2,2）计算为(0,0,3)依此类推，不知道是不是这样算的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，根据指定keyby字段进行聚合

##### **风：
> 怎么感觉像是Java stream的分布式版本。。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 分布式的框架是个Java 中的一些API的设计非常像

##### **平：
> (0,1,0)(0,0,1)(0,0,3)(0,0,6)(1,2,5)(0,0,14)(0,0,25)(0,0,38)这是我运行出来的结果

##### **5200：
> 生产环境无界流不建议使用聚合函数，那一些聚合的需求，应该怎么处理？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不建议的是那些状态无限增长的聚合，实际应用中一般会配合窗口使用。使得状态不会无限制扩张。

##### *璇：
> <span style="font-size: 15.372px;">老师您好，从kafka读取数据到source表，使用event-time。如果时间戳需要通过两个字段拼接而成（比如DATE+TIME），该如何实现？没查到相关的例子和语法。</span>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; assignTimestampsAndWatermarks会覆写extractTimestamp，这时候可以处理

##### **1633：
> 老师，你的课程对我学习flink很有帮助，谢谢！关于&nbsp;max 和 maxBy 的举例中<div><pre style="color: rgb(0, 0, 0); font-family: Menlo; font-size: 9pt;"><span style="color:#808080;font-style:italic;">DataStreamSource&lt;MyStreamingSource.Item&gt; items = env.fromCollection(data);</span></pre><pre style="color: rgb(0, 0, 0); font-family: Menlo; font-size: 9pt;"><span style="color:#808080;font-style:italic;">这里会报incompatible types，需要写成</span></pre><pre style="color: rgb(0, 0, 0); font-family: Menlo; font-size: 9pt;"><pre style="font-family: Menlo; font-size: 9pt;">DataStreamSource&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; items = env.fromCollection(data);</pre><pre style="font-family: Menlo; font-size: 9pt;">老师，请看看是不是这样，或者可能你代码里是做了转化的，只是没有贴出来。</pre></pre></div>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

