<p data-nodeid="89861" class="">从这一课时开始我们将进入“Flink CEP 实时预警系统”的学习，本课时先介绍项目的背景、架构设计。</p>


<h3 data-nodeid="90029" class="">背景</h3>

<p data-nodeid="89609">我们在第 11 课时“Flink CEP 复杂事件处理”已经介绍了 Flink CEP 的原理，它是 Flink 提供的复杂事件处理库，也是 Flink 提供的一个非常亮眼的功能，当然更是 Flink 中最难以理解的部分之一。</p>
<p data-nodeid="89610">Complex Event Processing（CEP）允许我们在源源不断的数据中通过自定义的模式（Pattern）检测并且获取需要的数据，还可以对这些数据做对应的处理。Flink 提供了非常丰富的 API 来帮助我们实现非常复杂的模式进行数据匹配。</p>
<h3 data-nodeid="90197" class="">Flink CEP 应用场景</h3>

<p data-nodeid="89612">CEP 在互联网各个行业都有应用，例如金融、物流、电商等行业，具体的作用如下。</p>
<ul data-nodeid="89613">
<li data-nodeid="89614">
<p data-nodeid="89615"><strong data-nodeid="89658">实时监控</strong>：我们需要在大量的订单交易中发现那些虚假交易，在网站的访问日志中寻找那些使用脚本或者工具“爆破”登录的用户，或者在快递运输中发现那些滞留很久没有签收的包裹等。</p>
</li>
<li data-nodeid="89616">
<p data-nodeid="89617"><strong data-nodeid="89663">风险控制</strong>：比如金融行业可以用来进行风险控制和欺诈识别，从交易信息中寻找那些可能存在危险交易和非法交易。</p>
</li>
<li data-nodeid="89618">
<p data-nodeid="89619"><strong data-nodeid="89668">营销广告</strong>：跟踪用户的实时行为，指定对应的推广策略进行推送，提高广告的转化率。</p>
</li>
</ul>
<p data-nodeid="89620">当然了，还要很多其他的场景比如智能交通、物联网行业等，可以应用的场景不胜枚举。</p>
<h3 data-nodeid="90365" class="">Flink CEP 的原理</h3>

<p data-nodeid="89622" class="">如果你对 CEP 的理论基础非常感兴趣，推荐一篇论文“Efﬁcient Pattern Matching over Event Streams”。</p>
<p data-nodeid="89623" class="">Flink CEP 在运行时会将用户提交的代码转化成 NFA Graph，Graph 中包含状态（Flink 中 State 对象），以及连接状态的边（Flink 中 StateTransition 对象）。</p>
<p data-nodeid="89624">Flink 中的每个模式都包含多个状态，我们进行模式匹配的过程就是进行状态转换的过程，在实际应用 Flink CEP 时，首先需要创建一系列的 Pattern，然后利用 NFACompiler 将 Pattern 进行拆分并且创建出 NFA，NFA 包含了 Pattern 中的各个状态和各个状态间转换的表达式。</p>
<p data-nodeid="89625">我们用官网中的一个案例来讲解 Flink CEP 的应用：</p>
<pre class="lang-java" data-nodeid="89626"><code data-language="java">DataStream&lt;Event&gt; input = ... 
Pattern&lt;Event, ?&gt; pattern = Pattern.&lt;Event&gt;begin(<span class="hljs-string">"start"</span>).where( 
        <span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event event)</span> </span>{ 
                <span class="hljs-keyword">return</span> event.getId() == <span class="hljs-number">42</span>; 
            } 
        } 
    ).next(<span class="hljs-string">"middle"</span>).subtype(SubEvent.class).where( 
        <span class="hljs-keyword">new</span> SimpleCondition&lt;SubEvent&gt;() { 
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
<p data-nodeid="89627">在这个案例中可以看到程序结构如下：</p>
<ul data-nodeid="89628">
<li data-nodeid="89629">
<p data-nodeid="89630">第一步，定义一个模式 Pattern，在这里定义了一个这样的模式，即在所有接收到的事件中匹配那些以 ID 等于 42 的事件，然后匹配 volume 大于 10.0 的事件，继续匹配一个 name 等于 end 的事件；</p>
</li>
<li data-nodeid="89631">
<p data-nodeid="89632">第二步，匹配模式并且发出报警，根据定义的 pattern 在输入流上进行匹配，一旦命中我们的模式，就发出一个报警。</p>
</li>
</ul>
<h3 data-nodeid="90859">整体架构</h3>
<p data-nodeid="90860" class=""><img src="https://s0.lgstatic.com/i/image/M00/41/47/CgqCHl801fGAXignAACRQy8N7oY129.png" alt="image (3).png" data-nodeid="90868"></p>



<p data-nodeid="89635">我们在项目中定义特定事件附带各种上下文信息进入 Kafka，Flink 首先会消费这些信息过滤掉不需要的信息，然后会被我们定义好的模式进行处理，接着触发对应的规则；同时把触发规则的数据输出进行存储。</p>
<p data-nodeid="89636">整个项目的设计可以分为下述几个部分：</p>
<ul data-nodeid="89637">
<li data-nodeid="89638">
<p data-nodeid="89639">Flink CEP 源码解析和自定义消息事件</p>
</li>
<li data-nodeid="89640">
<p data-nodeid="89641">自定义 Pattern 和报警规则</p>
</li>
<li data-nodeid="89642">
<p data-nodeid="89643">Flink 调用 CEP 实现报警功能</p>
</li>
</ul>
<h3 data-nodeid="91043" class="">总结</h3>

<p data-nodeid="89645">本节课我们主要讲解了 Flink CEP 的应用场景和基本原理，在实际工作中，如果你的需求涉及从数据流中通过一定的规则识别部分数据，可以考虑使用 CEP。在接下来的课程中我们会分不同的课时来一一讲解这些知识点并进行实现。</p>

---

### 精选评论


