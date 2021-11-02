<p>你好，欢迎来到第 11 课时，这一课时将介绍 Flink 中提供的一个很重要的功能：复杂事件处理 CEP。</p>
<h4>背景</h4>
<p>Complex Event Processing（CEP）是 Flink 提供的一个非常亮眼的功能，关于 CEP 的解释我们引用维基百科中的一段话：</p>
<blockquote>
<p>CEP,&nbsp;is&nbsp;event&nbsp;processing&nbsp;that&nbsp;combines&nbsp;data&nbsp;from&nbsp;multiple&nbsp;sources&nbsp;to&nbsp;infer&nbsp;events&nbsp;or&nbsp;patterns&nbsp;that&nbsp;suggest&nbsp;more&nbsp;complicated&nbsp;circumstances.&nbsp;The&nbsp;goal&nbsp;of&nbsp;complex&nbsp;event&nbsp;processing&nbsp;is&nbsp;to&nbsp;identify&nbsp;meaningful&nbsp;events&nbsp;(such&nbsp;as&nbsp;opportunities&nbsp;or&nbsp;threats)&nbsp;and&nbsp;respond&nbsp;to&nbsp;them&nbsp;as&nbsp;quickly&nbsp;as&nbsp;possible.</p>
</blockquote>
<p>在我们的实际生产中，随着数据的实时性要求越来越高，实时数据的量也在不断膨胀，在某些业务场景中需要根据连续的实时数据，发现其中有价值的那些事件。</p>
<p>说到底，Flink 的 CEP 到底解决了什么样的问题呢？</p>
<p>比如，我们需要在大量的订单交易中发现那些虚假交易，在网站的访问日志中寻找那些使用脚本或者工具“爆破”登录的用户，或者在快递运输中发现那些滞留很久没有签收的包裹等。</p>
<p>如果你对 CEP 的理论基础非常感兴趣，推荐一篇论文“Efﬁcient Pattern Matching over Event Streams”。</p>
<p>Flink 对 CEP 的支持非常友好，并且支持复杂度非常高的模式匹配，其吞吐和延迟都令人满意。</p>
<h4>程序结构</h4>
<p>Flink CEP 的程序结构主要分为两个步骤：</p>
<ul>
<li>定义模式</li>
<li>匹配结果</li>
</ul>
<p>我们在官网中可以找到一个 Flink 提供的案例：</p>
<pre><code data-language="java" class="lang-java">DataStream&lt;Event&gt; input = ...
Pattern&lt;Event, ?&gt; pattern = Pattern.&lt;Event&gt;begin(<span class="hljs-string">"start"</span>).where(
        <span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event event)</span> </span>{
                <span class="hljs-keyword">return</span> event.getId() == <span class="hljs-number">42</span>;
            }
        }
    ).next(<span class="hljs-string">"middle"</span>).subtype(SubEvent<span class="hljs-class">.<span class="hljs-keyword">class</span>).<span class="hljs-title">where</span>(
        <span class="hljs-title">new</span> <span class="hljs-title">SimpleCondition</span>&lt;<span class="hljs-title">SubEvent</span>&gt;() </span>{
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(SubEvent subEvent)</span> </span>{
                <span class="hljs-keyword">return</span> subEvent.getVolume() &gt;= <span class="hljs-number">10.0</span>;
            }
        }
    ).followedBy(<span class="hljs-string">"end"</span>).where(
         <span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event event)</span> </span>{
                <span class="hljs-keyword">return</span> event.getName().equals(<span class="hljs-string">"end"</span>);
            }
         }
    );
PatternStream&lt;Event&gt; patternStream = CEP.pattern(input, pattern);
DataStream&lt;Alert&gt; result = patternStream.process(
    <span class="hljs-keyword">new</span> PatternProcessFunction&lt;Event, Alert&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processMatch</span><span class="hljs-params">(
                Map&lt;String, List&lt;Event&gt;&gt; pattern,
                Context ctx,
                Collector&lt;Alert&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            out.collect(createAlertFrom(pattern));
        }
    });
</code></pre>
<p>在这个案例中可以看到程序结构分别是：</p>
<ul>
<li>第一步，定义一个模式 Pattern，在这里定义了一个这样的模式，即在所有接收到的事件中匹配那些以 id 等于 42 的事件，然后匹配 volume 大于 10.0 的事件，继续匹配一个 name 等于 end 的事件；</li>
<li>第二步，匹配模式并且发出报警，根据定义的 pattern 在输入流上进行匹配，一旦命中我们的模式，就发出一个报警。</li>
</ul>
<h3>模式定义</h3>
<p>Flink 支持了非常丰富的模式定义，这些模式也是我们实现复杂业务逻辑的基础。我们把支持的模式简单做了以下分类，完整的模式定义 API 支持<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/libs/cep.html#individual-patterns">可以参考官网资料</a>。</p>
<p><strong>简单模式</strong></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AB/CgqCHl7GInaAMu-kAAF3VZMO0po344.png" alt="1.png"></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AC/CgqCHl7GIn6AdsvlAAI2-XxT_0c215.png" alt="2.png"></p>
<p><strong>联合模式</strong></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A0/Ciqc1F7GIoqAfXZbAAEefGXZgmA893.png" alt="3.png"></p>
<p><strong>匹配后的忽略模式</strong></p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A0/Ciqc1F7GIpqADcwOAACchsP0psk030.png" alt="4.png"></p>
<h4>源码解析</h4>
<p>我们在上面的官网案例中可以发现，Flink CEP 的整个过程是：</p>
<ul>
<li>从一个 Source 作为输入</li>
<li>经过一个 Pattern 算子转换为 PatternStream</li>
<li>经过 select/process 算子转换为 DataStream</li>
</ul>
<p>我们来看一下 select 和 process 算子都做了什么？</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AC/CgqCHl7GIqqAeu1oAAGd-JibEng540.png" alt="image (12).png"></p>
<p>可以看到最终的逻辑都是在 PatternStream 这个类中进行的。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> &lt;R&gt; <span class="hljs-function">SingleOutputStreamOperator&lt;R&gt; <span class="hljs-title">process</span><span class="hljs-params">(
      <span class="hljs-keyword">final</span> PatternProcessFunction&lt;T, R&gt; patternProcessFunction,
      <span class="hljs-keyword">final</span> TypeInformation&lt;R&gt; outTypeInfo)</span> </span>{
   <span class="hljs-keyword">return</span> builder.build(
      outTypeInfo,
      builder.clean(patternProcessFunction));
}
</code></pre>
<p>最终经过 PatternStreamBuilder 的 build 方法生成了一个 SingleOutputStreamOperator，这个类继承了 DataStream。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A0/Ciqc1F7GIraAGh4wAAOFG844ie0933.png" alt="image (13).png"></p>
<p>最终的处理计算逻辑其实都封装在了 CepOperator 这个类中，而在 CepOperator 这个类中的 processElement 方法则是对每一条数据的处理逻辑。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/AC/CgqCHl7GIr6AOcT4AAb08tNwBuY069.png" alt="image (14).png"></p>
<p>同时由于 CepOperator 实现了 Triggerable 接口，所以会执行定时器。所有核心的处理逻辑都在 updateNFA 这个方法中。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A0/Ciqc1F7GIsiAViyeAAaiO9zCL3s307.png" alt="image (15).png"></p>
<p>入口在这里：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processEvent</span><span class="hljs-params">(NFAState nfaState, IN event, <span class="hljs-keyword">long</span> timestamp)</span> <span class="hljs-keyword">throws</span> Exception </span>{
   <span class="hljs-keyword">try</span> (SharedBufferAccessor&lt;IN&gt; sharedBufferAccessor = partialMatches.getAccessor()) {
      Collection&lt;Map&lt;String, List&lt;IN&gt;&gt;&gt; patterns =
         nfa.process(sharedBufferAccessor, nfaState, event, timestamp, afterMatchSkipStrategy, cepTimerService);
      processMatchedSequences(patterns, timestamp);
   }
}
</code></pre>
<p>NFA 的全称为 <strong>非确定有限自动机</strong>，NFA 中包含了模式匹配中的各个状态和状态间的转换。</p>
<p>在 NFA 这个类中的核心方法是：process 和 advanceTime，这两个方法的实现有些复杂，总体来说可以归纳为，每当一条新来的数据进入状态机都会驱动整个状态机进行状态转换。</p>
<h4>实战案例</h4>
<p>我们模拟电商网站用户搜索的数据来作为数据的输入源，然后查找其中重复搜索某一个商品的人，并且发送一条告警消息。</p>
<p>代码如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception</span>{
    <span class="hljs-keyword">final</span> StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env.setParallelism(<span class="hljs-number">1</span>);
    DataStreamSource source = env.fromElements(
            <span class="hljs-comment">//浏览记录</span>
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"外套"</span>, <span class="hljs-number">1L</span>),
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">1L</span>),
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">2L</span>),
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">3L</span>),
            Tuple3.of(<span class="hljs-string">"Ming"</span>, <span class="hljs-string">"衣服"</span>,<span class="hljs-number">1L</span>),
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"鞋子"</span>,<span class="hljs-number">1L</span>),
            Tuple3.of(<span class="hljs-string">"Marry"</span>, <span class="hljs-string">"鞋子"</span>,<span class="hljs-number">2L</span>),
            Tuple3.of(<span class="hljs-string">"LiLei"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">1L</span>),
            Tuple3.of(<span class="hljs-string">"LiLei"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">2L</span>),
            Tuple3.of(<span class="hljs-string">"LiLei"</span>, <span class="hljs-string">"帽子"</span>,<span class="hljs-number">3L</span>)
    );
    <span class="hljs-comment">//定义Pattern,寻找连续搜索帽子的用户</span>
    Pattern&lt;Tuple3&lt;String, String, Long&gt;, Tuple3&lt;String, String, Long&gt;&gt; pattern = Pattern
            .&lt;Tuple3&lt;String, String, Long&gt;&gt;begin(<span class="hljs-string">"start"</span>)
            .where(<span class="hljs-keyword">new</span> SimpleCondition&lt;Tuple3&lt;String, String, Long&gt;&gt;() {
                <span class="hljs-meta">@Override</span>
                <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Tuple3&lt;String, String, Long&gt; value)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                    <span class="hljs-keyword">return</span> value.f1.equals(<span class="hljs-string">"帽子"</span>);
                }
            }) <span class="hljs-comment">//.timesOrMore(3);</span>
            .next(<span class="hljs-string">"middle"</span>)
            .where(<span class="hljs-keyword">new</span> SimpleCondition&lt;Tuple3&lt;String, String, Long&gt;&gt;() {
                <span class="hljs-meta">@Override</span>
                <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Tuple3&lt;String, String, Long&gt; value)</span> <span class="hljs-keyword">throws</span> Exception </span>{
                    <span class="hljs-keyword">return</span> value.f1.equals(<span class="hljs-string">"帽子"</span>);
                }
            });

    KeyedStream keyedStream = source.keyBy(<span class="hljs-number">0</span>);
    PatternStream patternStream = CEP.pattern(keyedStream, pattern);
    SingleOutputStreamOperator matchStream = patternStream.select(<span class="hljs-keyword">new</span> PatternSelectFunction&lt;Tuple3&lt;String, String, Long&gt;, String&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">select</span><span class="hljs-params">(Map&lt;String, List&lt;Tuple3&lt;String, String, Long&gt;&gt;&gt; pattern)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            List&lt;Tuple3&lt;String, String, Long&gt;&gt; middle = pattern.get(<span class="hljs-string">"middle"</span>);
            <span class="hljs-keyword">return</span> middle.get(<span class="hljs-number">0</span>).f0 + <span class="hljs-string">":"</span> + middle.get(<span class="hljs-number">0</span>).f2 + <span class="hljs-string">":"</span> + <span class="hljs-string">"连续搜索两次帽子!"</span>;
        }
    });
    matchStream.printToErr();
    env.execute(<span class="hljs-string">"execute cep"</span>);
}
</code></pre>
<p>上述代码的逻辑我们可以分解如下。</p>
<p>首先定义一个数据源，模拟了一些用户的搜索数据，然后定义了自己的 Pattern。这个模式的特点就是连续两次搜索商品“帽子”，然后进行匹配，发现匹配后输出一条提示信息，直接打印在控制台上。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/0E/A0/Ciqc1F7GIvOANvsCAANo0ngoNL0417.png" alt="image (16).png"></p>
<p>可以看到，提示信息已经打印在了控制台上。</p>
<h4>总结</h4>
<p>这一课时讲解了 Flink CEP 的支持和实现，并且模拟了一下简单的电商搜索场景来实现对连续搜索某一个商品的提示，关于模式匹配还有更加复杂的应用，比如识别网站的用户的爆破登录、运维监控等，建议你结合源码和官网进行深入的学习。</p>
<p><a href="https://github.com/wangzhiwubigdata/quickstart">点击这里下载本课程源码</a></p>

---

### 精选评论

##### **卓：
> next、fellowby的区别是 next可以不连续，fellowby必须连续吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; next 的意思是说上一步符合条件的元素之后紧挨着的元素；而 followedBy 并不要求一定是挨着的元素。这两者分别称为严格近邻和非严格近邻。简单的说，next要求必须连续，followedBy可以不连续。

##### **兵：
> flink cep是固化的，如何才能做到cep规则热部署动态实时生效呢？求指导

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; FlinkCEP已经支持热部署了，你可以查一下FLINK-7129动态变更模式这个issue

##### **0278：
> 老师实现连续五次匹配出帽子，该怎么使用times方法呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp;  可以直接使用times方法，times(5) 代表匹配5次

##### *帅：
> 如果需求改为在30分钟内连续搜索两次帽子的话，应该要怎么实现CEP?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; CEP对时间的支持很丰富，可以直接在where后面接within(Time.minutes(30))，即代表30分钟内发生事件

##### *帅：
> 匹配后的忽略模式，表格中的结果一样吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 应该是一样的。

##### **军：
> hello&nbsp; 老师，麻烦问一下我的flink的CEP依赖不能下载<font color="#000000" face="Menlo"><span style="font-size: 12px;">，这个知道是什么问题吗？</span></font>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 大概率是网络问题，都是在maven中央仓库下载的

