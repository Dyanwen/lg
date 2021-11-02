<p data-nodeid="41849">在上一课时提过，PatternStream 是 Flink CEP 对模式匹配后流的抽象和定义，它把 DataStream 和 Pattern 组合到一起，并且基于 PatternStream 提供了一系列的方法，比如 select、process 等。</p>
<p data-nodeid="41850">Flink CEP 的核心在于模式匹配，对于不同模式匹配特性的支持，往往决定相应的 CEP 框架是否能够得到广泛应用。那么 Flink CEP 对模式提供了哪些支持呢？</p>
<h3 data-nodeid="42725" class="">Pattern 分类</h3>

<p data-nodeid="41852">Flink CEP 提供了 Pattern API 用于对输入流数据进行复杂事件规则的定义，用来提取符合规则的事件序列。</p>
<p data-nodeid="41853">Flink 中的 Pattern 分为单个模式、组合模式、模式组  3 类。</p>
<h4 data-nodeid="42969" class="">单个模式</h4>

<p data-nodeid="41855">复杂规则中的每一个单独的模式定义，就是个体模式。我们既可以定义一个给定事件出现的次数（<strong data-nodeid="41924">量词</strong>），也可以定义一个条件来决定一个进来的事件是否被接受进入这个模式（<strong data-nodeid="41925">条件</strong>）。</p>
<p data-nodeid="41856">例如，我们对一个命名为 start 的模式，可以定义如下量词：</p>
<pre class="lang-java" data-nodeid="41857"><code data-language="java"><span class="hljs-comment">// 期望出现4次 </span>
start.times(<span class="hljs-number">4</span>); 
<span class="hljs-comment">// 期望出现0或者4次 </span>
start.times(<span class="hljs-number">4</span>).optional(); 
<span class="hljs-comment">// 期望出现2、3或者4次 </span>
start.times(<span class="hljs-number">2</span>, <span class="hljs-number">4</span>); 
<span class="hljs-comment">// 期望出现2、3或者4次，并且尽可能地重复次数多 </span>
start.times(<span class="hljs-number">2</span>, <span class="hljs-number">4</span>).greedy(); 
<span class="hljs-comment">// 期望出现0、2、3或者4次 </span>
start.times(<span class="hljs-number">2</span>, <span class="hljs-number">4</span>).optional(); 
<span class="hljs-comment">// 期望出现0、2、3或者4次，并且尽可能地重复次数多 </span>
start.times(<span class="hljs-number">2</span>, <span class="hljs-number">4</span>).optional().greedy(); 
<span class="hljs-comment">// 期望出现1到多次 </span>
start.oneOrMore(); 
<span class="hljs-comment">// 期望出现1到多次，并且尽可能地重复次数多 </span>
start.oneOrMore().greedy(); 
<span class="hljs-comment">// 期望出现0到多次 </span>
start.oneOrMore().optional(); 
<span class="hljs-comment">// 期望出现0到多次，并且尽可能地重复次数多 </span>
start.oneOrMore().optional().greedy(); 
<span class="hljs-comment">// 期望出现2到多次 </span>
start.timesOrMore(<span class="hljs-number">2</span>); 
<span class="hljs-comment">// 期望出现2到多次，并且尽可能地重复次数多 </span>
start.timesOrMore(<span class="hljs-number">2</span>).greedy(); 
<span class="hljs-comment">// 期望出现0、2或多次 </span>
start.timesOrMore(<span class="hljs-number">2</span>).optional(); 
<span class="hljs-comment">// 期望出现0、2或多次，并且尽可能地重复次数多 </span>
start.timesOrMore(<span class="hljs-number">2</span>).optional().greedy();  
</code></pre>
<p data-nodeid="41858">我们还可以定义需要的匹配条件：</p>
<pre class="lang-java" data-nodeid="41859"><code data-language="java">start.where(<span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event value)</span> </span>{ 
        <span class="hljs-keyword">return</span> value.getName().startsWith(<span class="hljs-string">"foo"</span>); 
    } 
}); 
</code></pre>
<p data-nodeid="41860">上面代码展示了定义一个以“foo”开头的事件。</p>
<p data-nodeid="41861">当然我们还可以把条件组合到一起，例如，可以通过依次调用 where 来组合条件：</p>
<pre class="lang-java" data-nodeid="41862"><code data-language="java">pattern.where(<span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event value)</span> </span>{ 
        <span class="hljs-keyword">return</span> ... <span class="hljs-comment">// 一些判断条件 </span>
    } 
}).or(<span class="hljs-keyword">new</span> SimpleCondition&lt;Event&gt;() { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(Event value)</span> </span>{ 
        <span class="hljs-keyword">return</span> ... <span class="hljs-comment">// 一些判断条件 </span>
    } 
}); 
</code></pre>
<h4 data-nodeid="43213" class="">组合模式</h4>

<p data-nodeid="41864">我们把很多单个模式组合起来，就形成了组合模式。Flink CEP 支持事件之间如下形式的连续策略：</p>
<ul data-nodeid="41865">
<li data-nodeid="41866">
<p data-nodeid="41867"><strong data-nodeid="41936">严格连续</strong>，期望所有匹配的事件严格的一个接一个出现，中间没有任何不匹配的事件；</p>
</li>
<li data-nodeid="41868">
<p data-nodeid="41869"><strong data-nodeid="41940">松散连续</strong></p>
</li>
<li data-nodeid="41870">
<p data-nodeid="41871"><strong data-nodeid="41944">不确定的松散连续</strong></p>
</li>
</ul>
<p data-nodeid="41872">我们直接参考官网给出的案例：</p>
<pre class="lang-java" data-nodeid="41873"><code data-language="java"><span class="hljs-comment">// 严格连续 </span>
Pattern&lt;Event, ?&gt; strict = start.next(<span class="hljs-string">"middle"</span>).where(...); 
<span class="hljs-comment">// 松散连续 </span>
Pattern&lt;Event, ?&gt; relaxed = start.followedBy(<span class="hljs-string">"middle"</span>).where(...); 
<span class="hljs-comment">// 不确定的松散连续 </span>
Pattern&lt;Event, ?&gt; nonDetermin = start.followedByAny(<span class="hljs-string">"middle"</span>).where(...); 
<span class="hljs-comment">// 严格连续的NOT模式 </span>
Pattern&lt;Event, ?&gt; strictNot = start.notNext(<span class="hljs-string">"not"</span>).where(...); 
<span class="hljs-comment">// 松散连续的NOT模式 </span>
Pattern&lt;Event, ?&gt; relaxedNot = start.notFollowedBy(<span class="hljs-string">"not"</span>).where(...); 
</code></pre>
<p data-nodeid="41874">对于上述的<strong data-nodeid="41955">松散连续</strong>和<strong data-nodeid="41956">不确定的松散连续</strong>这里举个例子来说明一下，例如，我们定义了模式“a b”，对于一个事件序列“a,c,b1,b2”，那么会产生下面的结果：</p>
<ul data-nodeid="41875">
<li data-nodeid="41876">
<p data-nodeid="41877">“a”和“b”之间严格连续则返回：&nbsp;{}&nbsp;（没有匹配）</p>
</li>
<li data-nodeid="41878">
<p data-nodeid="41879">“a”和“b”之间松散连续则返回：&nbsp;{a b1}</p>
</li>
<li data-nodeid="41880">
<p data-nodeid="41881">“a”和“b”之间不确定的松散连续则返回：&nbsp;{a b1}、{a b2}</p>
</li>
</ul>
<h4 data-nodeid="43457" class="">模式组</h4>

<p data-nodeid="41883">将一个模式作为条件嵌套在单个模式里，就是模式组。我们举例如下：</p>
<pre class="lang-java" data-nodeid="41884"><code data-language="java">Pattern&lt;Event, ?&gt; start = Pattern.begin( 
    Pattern.&lt;Event&gt;begin(<span class="hljs-string">"start"</span>).where(...).followedBy(<span class="hljs-string">"start_middle"</span>).where(...) 
); 
<span class="hljs-comment">// 严格连续 </span>
Pattern&lt;Event, ?&gt; strict = start.next( 
    Pattern.&lt;Event&gt;begin(<span class="hljs-string">"next_start"</span>).where(...).followedBy(<span class="hljs-string">"next_middle"</span>).where(...) 
).times(<span class="hljs-number">3</span>); 
<span class="hljs-comment">// 松散连续 </span>
Pattern&lt;Event, ?&gt; relaxed = start.followedBy( 
    Pattern.&lt;Event&gt;begin(<span class="hljs-string">"followedby_start"</span>).where(...).followedBy(<span class="hljs-string">"followedby_middle"</span>).where(...) 
).oneOrMore(); 
<span class="hljs-comment">// 不确定松散连续 </span>
Pattern&lt;Event, ?&gt; nonDetermin = start.followedByAny( 
    Pattern.&lt;Event&gt;begin(<span class="hljs-string">"followedbyany_start"</span>).where(...).followedBy(<span class="hljs-string">"followedbyany_middle"</span>).where(...) 
).optional(); 
</code></pre>
<h3 data-nodeid="43701" class="">自定义 Pattern</h3>

<p data-nodeid="41886">上一课时定义了几个不同的消息源，我们分别根据需求自定义匹配模式。</p>
<ul data-nodeid="41887">
<li data-nodeid="41888">
<p data-nodeid="41889">第一个场景，连续登录场景</p>
</li>
</ul>
<p data-nodeid="41890">在这个场景中，我们需要找出那些 5 秒钟内连续登录失败的账号，然后禁止用户再次尝试登录需要等待 1 分钟。</p>
<pre class="lang-java" data-nodeid="41891"><code data-language="java">Pattern.&lt;LogInEvent&gt;begin(<span class="hljs-string">"start"</span>).where(<span class="hljs-keyword">new</span> IterativeCondition&lt;LogInEvent&gt;() { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(LogInEvent value, Context&lt;LogInEvent&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">return</span> value.getIsSuccess().equals(<span class="hljs-string">"fail"</span>); 
    } 
}).next(<span class="hljs-string">"next"</span>).where(<span class="hljs-keyword">new</span> IterativeCondition&lt;LogInEvent&gt;() { 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(LogInEvent value, Context&lt;LogInEvent&gt; ctx)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
        <span class="hljs-keyword">return</span> value.getIsSuccess().equals(<span class="hljs-string">"fail"</span>); 
    } 
}).within(Time.seconds(<span class="hljs-number">5</span>)); 
</code></pre>
<ul data-nodeid="41892">
<li data-nodeid="41893">
<p data-nodeid="41894">第二个场景，超时未支付</p>
</li>
</ul>
<p data-nodeid="41895">在这个场景中，我们需要找出那些下单后 10 分钟内没有支付的订单。</p>
<pre class="lang-java" data-nodeid="41896"><code data-language="java">Pattern.&lt;PayEvent&gt; 
        begin(<span class="hljs-string">"begin"</span>) 
        .where(<span class="hljs-keyword">new</span> IterativeCondition&lt;PayEvent&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(PayEvent payEvent, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                <span class="hljs-keyword">return</span> payEvent.getAction().equals(<span class="hljs-string">"create"</span>); 
            } 
        }) 
        .next(<span class="hljs-string">"next"</span>) 
        .where(<span class="hljs-keyword">new</span> IterativeCondition&lt;PayEvent&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(PayEvent payEvent, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{ 
                <span class="hljs-keyword">return</span> payEvent.getAction().equals(<span class="hljs-string">"pay"</span>); 
            } 
        }) 
        .within(Time.seconds(<span class="hljs-number">600</span>)); 
OutputTag&lt;PayEvent&gt; orderTiemoutOutput = <span class="hljs-keyword">new</span> OutputTag&lt;PayEvent&gt;(<span class="hljs-string">"orderTimeout"</span>) {}; 
</code></pre>
<p data-nodeid="41897">在这里我们使用了侧输出流，并且将正常的订单流和超时未支付的超时流分开：</p>
<pre class="lang-java" data-nodeid="41898"><code data-language="java">SingleOutputStreamOperator selectResult = patternStream.select(orderTiemoutOutput, 
        (PatternTimeoutFunction&lt;PayEvent, ResultPayEvent&gt;) (map, l) -&gt; <span class="hljs-keyword">new</span> ResultPayEvent(map.get(<span class="hljs-string">"begin"</span>).get(<span class="hljs-number">0</span>).getUserId(), <span class="hljs-string">"timeout"</span>), 
        (PatternSelectFunction&lt;PayEvent, ResultPayEvent&gt;) map -&gt; <span class="hljs-keyword">new</span> ResultPayEvent(map.get(<span class="hljs-string">"next"</span>).get(<span class="hljs-number">0</span>).getUserId(), <span class="hljs-string">"success"</span>) 
); 
DataStream timeOutSideOutputStream = selectResult.getSideOutput(orderTiemoutOutput) 
</code></pre>
<ul data-nodeid="41899">
<li data-nodeid="41900">
<p data-nodeid="41901">第三个场景，找出交易活跃用户</p>
</li>
</ul>
<p data-nodeid="41902">在这个场景下，我们需要找出那些 24 小时内至少 5 次有效交易的账户。</p>
<pre class="lang-java" data-nodeid="41903"><code data-language="java">Pattern.&lt;TransactionEvent&gt;begin(<span class="hljs-string">"start"</span>).where( 
        <span class="hljs-keyword">new</span> SimpleCondition&lt;TransactionEvent&gt;() { 
            <span class="hljs-meta">@Override</span> 
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(TransactionEvent transactionEvent)</span> </span>{ 
                <span class="hljs-keyword">return</span> transactionEvent.getAmount() &gt; <span class="hljs-number">0</span>; 
            } 
        } 
).timesOrMore(<span class="hljs-number">5</span>) 
 .within(Time.hours(<span class="hljs-number">24</span>)); 
</code></pre>
<h3 data-nodeid="43945" class="">总结</h3>

<p data-nodeid="42231">本一课时我们讲解了 Flink CEP 的模式匹配种类，并且基于上一课时的三个场景自定义了 Pattern，我们在实际生产中可以根据需求定义更为复杂的模式。</p>

---

### 精选评论


