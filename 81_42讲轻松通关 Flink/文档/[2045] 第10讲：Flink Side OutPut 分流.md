<p>这一课时将介绍 Flink 中提供的一个很重要的功能：旁路分流器。</p>
<h3>分流场景</h3>
<p>我们在生产实践中经常会遇到这样的场景，需把输入源按照需要进行拆分，比如我期望把订单流按照金额大小进行拆分，或者把用户访问日志按照访问者的地理位置进行拆分等。面对这样的需求该如何操作呢？</p>
<h3>分流的方法</h3>
<p>通常来说针对不同的场景，有以下三种办法进行流的拆分。</p>
<h4>Filter 分流</h4>
<p><img src="https://s0.lgstatic.com/i/image/M00/0B/F6/CgqCHl7CAy6ADUaXAACSFUbdpuA911.png" alt="image (9).png"></p>
<p>Filter 方法我们在第 04 课时中（Flink 常用的 DataSet 和 DataStream API）讲过，这个算子用来根据用户输入的条件进行过滤，每个元素都会被 filter() 函数处理，如果 filter() 函数返回 true 则保留，否则丢弃。那么用在分流的场景，我们可以做多次 filter，把我们需要的不同数据生成不同的流。</p>
<p>来看下面的例子：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
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


    DataStreamSource&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; items = env.fromCollection(data);

    SingleOutputStreamOperator&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; zeroStream = items.filter((FilterFunction&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt;) value -&gt; value.f0 == <span class="hljs-number">0</span>);
    SingleOutputStreamOperator&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; oneStream = items.filter((FilterFunction&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt;) value -&gt; value.f0 == <span class="hljs-number">1</span>);

    zeroStream.print();
    oneStream.printToErr();


    <span class="hljs-comment">//打印结果</span>
    String jobName = <span class="hljs-string">"user defined streaming source"</span>;
    env.execute(jobName);
}
</code></pre>
<p>在上面的例子中我们使用 filter 算子将原始流进行了拆分，输入数据第一个元素为 0 的数据和第一个元素为 1 的数据分别被写入到了 zeroStream 和 oneStream 中，然后把两个流进行了打印。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0B/EB/Ciqc1F7CA2WAYbshAAKj494h86s723.png" alt="image (10).png"></p>
<p>可以看到 zeroStream 和 oneStream 分别被打印出来。</p>
<p>Filter 的弊端是显而易见的，为了得到我们需要的流数据，需要多次遍历原始流，这样无形中浪费了我们集群的资源。</p>
<h4>Split 分流</h4>
<p>Split 也是 Flink 提供给我们将流进行切分的方法，需要在 split 算子中定义 OutputSelector，然后重写其中的 select 方法，将不同类型的数据进行标记，最后对返回的 SplitStream 使用 select 方法将对应的数据选择出来。</p>
<p>我们来看下面的例子：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
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


    DataStreamSource&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; items = env.fromCollection(data);


    SplitStream&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; splitStream = items.split(<span class="hljs-keyword">new</span> OutputSelector&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> Iterable&lt;String&gt; <span class="hljs-title">select</span><span class="hljs-params">(Tuple3&lt;Integer, Integer, Integer&gt; value)</span> </span>{
            List&lt;String&gt; tags = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
            <span class="hljs-keyword">if</span> (value.f0 == <span class="hljs-number">0</span>) {
                tags.add(<span class="hljs-string">"zeroStream"</span>);
            } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (value.f0 == <span class="hljs-number">1</span>) {
                tags.add(<span class="hljs-string">"oneStream"</span>);
            }
            <span class="hljs-keyword">return</span> tags;
        }
    });

    splitStream.select(<span class="hljs-string">"zeroStream"</span>).print();
    splitStream.select(<span class="hljs-string">"oneStream"</span>).printToErr();

    <span class="hljs-comment">//打印结果</span>
    String jobName = <span class="hljs-string">"user defined streaming source"</span>;
    env.execute(jobName);
}
</code></pre>
<p>同样，我们把来源的数据使用 split 算子进行了切分，并且打印出结果。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0B/F6/CgqCHl7CA4aAbUSJAAG1LWNB3qw627.png" alt="image (11).png"></p>
<p>但是要注意，使用 split 算子切分过的流，是不能进行二次切分的，假如把上述切分出来的 zeroStream 和 oneStream 流再次调用 split 切分，控制台会抛出以下异常。</p>
<pre><code data-language="java" class="lang-java">Exception in thread <span class="hljs-string">"main"</span> java.lang.IllegalStateException: Consecutive multiple splits are not supported. Splits are deprecated. Please use side-outputs.
</code></pre>
<p>这是什么原因呢？我们在源码中可以看到注释，该方式已经废弃并且建议使用最新的 SideOutPut 进行分流操作。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0B/F7/CgqCHl7CA6OAJ-JDAAIrh1JSAEo033.png" alt="image (12).png"></p>
<h4>SideOutPut 分流</h4>
<p>SideOutPut 是 Flink 框架为我们提供的最新的也是最为推荐的分流方法，在使用 SideOutPut 时，需要按照以下步骤进行：</p>
<ul>
<li>定义 OutputTag</li>
<li>调用特定函数进行数据拆分
<ul>
<li>ProcessFunction</li>
<li>KeyedProcessFunction</li>
<li>CoProcessFunction</li>
<li>KeyedCoProcessFunction</li>
<li>ProcessWindowFunction</li>
<li>ProcessAllWindowFunction</li>
</ul>
</li>
</ul>
<p>在这里我们使用 ProcessFunction 来讲解如何使用 SideOutPut：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception </span>{

    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
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


    DataStreamSource&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; items = env.fromCollection(data);

    OutputTag&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; zeroStream = <span class="hljs-keyword">new</span> OutputTag&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt;(<span class="hljs-string">"zeroStream"</span>) {};
    OutputTag&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt; oneStream = <span class="hljs-keyword">new</span> OutputTag&lt;Tuple3&lt;Integer,Integer,Integer&gt;&gt;(<span class="hljs-string">"oneStream"</span>) {};


    SingleOutputStreamOperator&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; processStream= items.process(<span class="hljs-keyword">new</span> ProcessFunction&lt;Tuple3&lt;Integer, Integer, Integer&gt;, Tuple3&lt;Integer, Integer, Integer&gt;&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processElement</span><span class="hljs-params">(Tuple3&lt;Integer, Integer, Integer&gt; value, Context ctx, Collector&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{

            <span class="hljs-keyword">if</span> (value.f0 == <span class="hljs-number">0</span>) {
                ctx.output(zeroStream, value);
            } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (value.f0 == <span class="hljs-number">1</span>) {
                ctx.output(oneStream, value);
            }
        }
    });

    DataStream&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; zeroSideOutput = processStream.getSideOutput(zeroStream);
    DataStream&lt;Tuple3&lt;Integer, Integer, Integer&gt;&gt; oneSideOutput = processStream.getSideOutput(oneStream);

    zeroSideOutput.print();
    oneSideOutput.printToErr();


    <span class="hljs-comment">//打印结果</span>
    String jobName = <span class="hljs-string">"user defined streaming source"</span>;
    env.execute(jobName);
}
</code></pre>
<p>可以看到，我们将流进行了拆分，并且成功打印出了结果。这里要注意，Flink 最新提供的 SideOutPut 方式拆分流是<strong>可以多次进行拆分</strong>的，无需担心会爆出异常。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0B/F8/CgqCHl7CBMKAGHoUAAM-5UL5geg132.png" alt="image (13).png"></p>
<h3>总结</h3>
<p>这一课时我们讲解了 Flink 的一个小的知识点，是我们生产实践中经常遇到的场景，Flink 在最新的版本中也推荐我们使用 SideOutPut 进行流的拆分。</p>
<p><a href="https://github.com/wangzhiwubigdata/quickstart">点击这里下载本课程源码</a>。</p>

---

### 精选评论


