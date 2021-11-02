<p data-nodeid="96432">在上一课时中，我们详细讲解了 Flink CEP 中 Pattern 的分类，需要根据实际生产环境来选择单个模式、组合模式或者模式组。</p>



<p data-nodeid="96147">在前面的课程中我们提到的三种典型场景下，分别根据业务需要实现了 Pattern 的定义，也可以根据自定义的 Pattern 检测到异常事件。那么接下来就需要根据检测到的异常事件发送告警，这一课将从这三种场景入手，来讲解完整的代码实现逻辑。</p>
<h3 data-nodeid="96148">连续登录场景</h3>
<p data-nodeid="96149">在这个场景中，我们需要找出那些 5 秒钟内连续登录失败的账号，然后禁止用户，再次尝试登录需要等待 1 分钟。</p>
<p data-nodeid="96150">我们定义的 Pattern 规则如下：</p>
<pre class="lang-java" data-nodeid="96151"><code data-language="java">Pattern.&lt;LogInEvent&gt;begin(<span class="hljs-string">"start"</span>).where(<span class="hljs-keyword">new</span> IterativeCondition&lt;LogInEvent&gt;() {
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
<p data-nodeid="96152">从登录消息 LogInEvent 可以得到用户登录是否成功，当检测到 5 秒钟内用户连续两次登录失败，则会发出告警消息，提示用户 1 分钟以后再试，或者这时候就需要前端输入验证码才能继续尝试。</p>
<p data-nodeid="96153">首先我们模拟读取 Kafka 中的消息事件：</p>
<pre class="lang-java" data-nodeid="96154"><code data-language="java">DataStream&lt;LogInEvent&gt; source = env.fromElements(
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">1L</span>, <span class="hljs-string">"fail"</span>, <span class="hljs-number">1597905234000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">1L</span>, <span class="hljs-string">"success"</span>, <span class="hljs-number">1597905235000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">2L</span>, <span class="hljs-string">"fail"</span>, <span class="hljs-number">1597905236000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">2L</span>, <span class="hljs-string">"fail"</span>, <span class="hljs-number">1597905237000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">2L</span>, <span class="hljs-string">"fail"</span>, <span class="hljs-number">1597905238000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">3L</span>, <span class="hljs-string">"fail"</span>, <span class="hljs-number">1597905239000L</span>),
        <span class="hljs-keyword">new</span> LogInEvent(<span class="hljs-number">3L</span>, <span class="hljs-string">"success"</span>, <span class="hljs-number">1597905240000L</span>)
).assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> BoundedOutOfOrdernessGenerator()).keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;LogInEvent, Object&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">getKey</span><span class="hljs-params">(LogInEvent value)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> value.getUserId();
    }
});
</code></pre>
<p data-nodeid="96155">我们模拟了用户的登录信息，其中可以看到 ID 为 2 的用户连续三次登录失败。<br>
时间戳和水印提取器的代码如下：</p>
<pre class="lang-java" data-nodeid="96156"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BoundedOutOfOrdernessGenerator</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AssignerWithPeriodicWatermarks</span>&lt;<span class="hljs-title">LogInEvent</span>&gt;</span>{
        <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> maxOutOfOrderness = <span class="hljs-number">5000L</span>;
        <span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> currentTimeStamp;
        <span class="hljs-meta">@Nullable</span>
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> Watermark <span class="hljs-title">getCurrentWatermark</span><span class="hljs-params">()</span> </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Watermark(currentTimeStamp - maxOutOfOrderness);
        }
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">extractTimestamp</span><span class="hljs-params">(LogInEvent element, <span class="hljs-keyword">long</span> previousElementTimestamp)</span> </span>{
            Long timeStamp = element.getTimeStamp();
            currentTimeStamp = Math.max(timeStamp, currentTimeStamp);
            System.err.println(element.toString() + <span class="hljs-string">",EventTime:"</span> + timeStamp + <span class="hljs-string">",watermark:"</span> + (currentTimeStamp - maxOutOfOrderness));
            <span class="hljs-keyword">return</span> timeStamp;
        }
    }
</code></pre>
<p data-nodeid="96157">我们调用 Pattern.CEP 方法将 Pattern 和 Stream 结合在一起，在匹配到事件后先在控制台打印，并且向外发送。</p>
<pre class="lang-java" data-nodeid="96158"><code data-language="java">PatternStream&lt;LogInEvent&gt; patternStream = CEP.pattern(source, pattern);
SingleOutputStreamOperator&lt;AlertEvent&gt; process = patternStream.process(<span class="hljs-keyword">new</span> PatternProcessFunction&lt;LogInEvent, AlertEvent&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processMatch</span><span class="hljs-params">(Map&lt;String, List&lt;LogInEvent&gt;&gt; match, Context ctx, Collector&lt;AlertEvent&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        List&lt;LogInEvent&gt; start = match.get(<span class="hljs-string">"start"</span>);
        List&lt;LogInEvent&gt; next = match.get(<span class="hljs-string">"next"</span>);
        System.err.println(<span class="hljs-string">"start:"</span> + start + <span class="hljs-string">",next:"</span> + next);

        out.collect(<span class="hljs-keyword">new</span> AlertEvent(String.valueOf(start.get(<span class="hljs-number">0</span>).getUserId()), <span class="hljs-string">"出现连续登陆失败"</span>));
    }
});
process.printToErr();
env.execute(<span class="hljs-string">"execute cep"</span>);
</code></pre>
<p data-nodeid="96794">我们右键运行，查看结果，如下图所示：</p>
<p data-nodeid="96795" class=""><img src="https://s0.lgstatic.com/i/image/M00/45/CC/Ciqc1F9DfmuARRpfAAMOQ71OEHc817.png" alt="Drawing 0.png" data-nodeid="96799"></p>


<p data-nodeid="96161">可以看到报警事件的触发，其中 AlertEvent 是我们定义的报警事件。</p>
<pre class="lang-java" data-nodeid="96162"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AlertEvent</span> </span>{
    <span class="hljs-keyword">private</span> String id;
    <span class="hljs-keyword">private</span> String message;
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getId</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> id;
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setId</span><span class="hljs-params">(String id)</span> </span>{
        <span class="hljs-keyword">this</span>.id = id;
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getMessage</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> message;
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setMessage</span><span class="hljs-params">(String message)</span> </span>{
        <span class="hljs-keyword">this</span>.message = message;
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AlertEvent</span><span class="hljs-params">(String id, String message)</span> </span>{
        <span class="hljs-keyword">this</span>.id = id;
        <span class="hljs-keyword">this</span>.message = message;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-string">"AlertEvent{"</span> +
                <span class="hljs-string">"id='"</span> + id + <span class="hljs-string">'\''</span> +
                <span class="hljs-string">", message='"</span> + message + <span class="hljs-string">'\''</span> +
                <span class="hljs-string">'}'</span>;
    }
}
</code></pre>
<p data-nodeid="97160">对于 PatternProcessFunction 中的 processMatch 方法，第一个参数 Map&lt;String, List<loginevent>&gt; match，可以在源码中看到以下的注释：</loginevent></p>
<p data-nodeid="97161" class=""><img src="https://s0.lgstatic.com/i/image/M00/45/CC/Ciqc1F9DfnaAbHW-AAGdZ1czPP8127.png" alt="Drawing 1.png" data-nodeid="97169"></p>


<p data-nodeid="96165">match 这个 Map 的 Key 则是我们在 Pattern 中定义的 start 和 next。</p>
<h3 data-nodeid="96166">交易活跃用户</h3>
<p data-nodeid="96167">在这个场景下，我们需要找出那些 24 小时内至少 5 次有效交易的账户。</p>
<p data-nodeid="96168">在上一课时中我们的定义的规则如下：</p>
<pre class="lang-java" data-nodeid="96169"><code data-language="java">Pattern.&lt;TransactionEvent&gt;begin(<span class="hljs-string">"start"</span>).where(
        <span class="hljs-keyword">new</span> SimpleCondition&lt;TransactionEvent&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">filter</span><span class="hljs-params">(TransactionEvent transactionEvent)</span> </span>{
                <span class="hljs-keyword">return</span> transactionEvent.getAmount() &gt; <span class="hljs-number">0</span>;
            }
        }
).timesOrMore(<span class="hljs-number">5</span>)
 .within(Time.hours(<span class="hljs-number">24</span>));
</code></pre>
<p data-nodeid="96170">首先，我们模拟读取 Kafka 中的消息事件：</p>
<pre class="lang-java" data-nodeid="96171"><code data-language="java">DataStream&lt;TransactionEvent&gt; source = env.fromElements(
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">0.0D</span>, <span class="hljs-number">1597905234000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">100.0D</span>, <span class="hljs-number">1597905235000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">200.0D</span>, <span class="hljs-number">1597905236000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">300.0D</span>, <span class="hljs-number">1597905237000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">400.0D</span>, <span class="hljs-number">1597905238000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"100XX"</span>, <span class="hljs-number">500.0D</span>, <span class="hljs-number">1597905239000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"101XX"</span>, <span class="hljs-number">0.0D</span>, <span class="hljs-number">1597905240000L</span>),
        <span class="hljs-keyword">new</span> TransactionEvent(<span class="hljs-string">"101XX"</span>, <span class="hljs-number">100.0D</span>, <span class="hljs-number">1597905241000L</span>)
).assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> BoundedOutOfOrdernessGenerator()).keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;TransactionEvent, Object&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">getKey</span><span class="hljs-params">(TransactionEvent value)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> value.getAccout();
    }
});
</code></pre>
<p data-nodeid="96172">然后调用 Pattern.CEP 方法将 Pattern 和 Stream 结合在一起，我们在匹配到事件后先在控制台打印，并且向外发送。</p>
<pre class="lang-java" data-nodeid="96173"><code data-language="java">PatternStream&lt;TransactionEvent&gt; patternStream = CEP.pattern(source, pattern);
    SingleOutputStreamOperator&lt;AlertEvent&gt; process = patternStream.process(<span class="hljs-keyword">new</span> PatternProcessFunction&lt;TransactionEvent, AlertEvent&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">processMatch</span><span class="hljs-params">(Map&lt;String, List&lt;TransactionEvent&gt;&gt; match, Context ctx, Collector&lt;AlertEvent&gt; out)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            List&lt;TransactionEvent&gt; start = match.get(<span class="hljs-string">"start"</span>);
            List&lt;TransactionEvent&gt; next = match.get(<span class="hljs-string">"next"</span>);
            System.err.println(<span class="hljs-string">"start:"</span> + start + <span class="hljs-string">",next:"</span> + next);
            out.collect(<span class="hljs-keyword">new</span> AlertEvent(start.get(<span class="hljs-number">0</span>).getAccout(), <span class="hljs-string">"连续有效交易！"</span>));
        }
    });
    process.printToErr();
    env.execute(<span class="hljs-string">"execute cep"</span>);
}
</code></pre>
<p data-nodeid="97530">最后右键运行查看结果，如下图所示：</p>
<p data-nodeid="97531" class=""><img src="https://s0.lgstatic.com/i/image/M00/45/D8/CgqCHl9DfoKAJoaeAAMWpTwyZAU138.png" alt="Drawing 2.png" data-nodeid="97535"></p>


<h3 data-nodeid="96176">超时未支付</h3>
<p data-nodeid="96177">在这个场景中，我们需要找出那些下单后 10 分钟内没有支付的订单。</p>
<p data-nodeid="96178">在上一课时中我们的定义的规则如下：</p>
<pre class="lang-java" data-nodeid="96179"><code data-language="java">Pattern.&lt;PayEvent&gt;
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
<p data-nodeid="96180">首先我们模拟读取 Kafka 中的消息事件：</p>
<pre class="lang-java" data-nodeid="96181"><code data-language="java">DataStream&lt;PayEvent&gt; source = env.fromElements(
        <span class="hljs-keyword">new</span> PayEvent(<span class="hljs-number">1L</span>, <span class="hljs-string">"create"</span>, <span class="hljs-number">1597905234000L</span>),
        <span class="hljs-keyword">new</span> PayEvent(<span class="hljs-number">1L</span>, <span class="hljs-string">"pay"</span>, <span class="hljs-number">1597905235000L</span>),
        <span class="hljs-keyword">new</span> PayEvent(<span class="hljs-number">2L</span>, <span class="hljs-string">"create"</span>, <span class="hljs-number">1597905236000L</span>),
        <span class="hljs-keyword">new</span> PayEvent(<span class="hljs-number">2L</span>, <span class="hljs-string">"pay"</span>, <span class="hljs-number">1597905237000L</span>),
        <span class="hljs-keyword">new</span> PayEvent(<span class="hljs-number">3L</span>, <span class="hljs-string">"create"</span>, <span class="hljs-number">1597905239000L</span>)

).assignTimestampsAndWatermarks(<span class="hljs-keyword">new</span> BoundedOutOfOrdernessGenerator()).keyBy(<span class="hljs-keyword">new</span> KeySelector&lt;PayEvent, Object&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">getKey</span><span class="hljs-params">(PayEvent value)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> value.getUserId();
    }
});
</code></pre>
<p data-nodeid="96182">然后对匹配的结果进行分流，select 在这里有三个参数，第一个是超时消息的侧输出 Tag，第二个参数是超时消息的处理逻辑，第三个参数是正常的订单消息。</p>
<pre class="lang-java" data-nodeid="96183"><code data-language="java">OutputTag&lt;PayEvent&gt; orderTimeoutOutput = <span class="hljs-keyword">new</span> OutputTag&lt;PayEvent&gt;(<span class="hljs-string">"orderTimeout"</span>) {};
Pattern&lt;PayEvent, PayEvent&gt; pattern = Pattern.&lt;PayEvent&gt;
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
PatternStream&lt;PayEvent&gt; patternStream = CEP.pattern(source, pattern);
SingleOutputStreamOperator&lt;PayEvent&gt; result = patternStream.select(orderTimeoutOutput, <span class="hljs-keyword">new</span> PatternTimeoutFunction&lt;PayEvent, PayEvent&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> PayEvent <span class="hljs-title">timeout</span><span class="hljs-params">(Map&lt;String, List&lt;PayEvent&gt;&gt; map, <span class="hljs-keyword">long</span> l)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> map.get(<span class="hljs-string">"begin"</span>).get(<span class="hljs-number">0</span>);
    }
}, <span class="hljs-keyword">new</span> PatternSelectFunction&lt;PayEvent, PayEvent&gt;() {
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> PayEvent <span class="hljs-title">select</span><span class="hljs-params">(Map&lt;String, List&lt;PayEvent&gt;&gt; map)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">return</span> map.get(<span class="hljs-string">"next"</span>).get(<span class="hljs-number">0</span>);
    }
});
DataStream&lt;PayEvent&gt; sideOutput = result.getSideOutput(orderTimeoutOutput);
sideOutput.printToErr();
env.execute(<span class="hljs-string">"execute cep"</span>);
</code></pre>
<p data-nodeid="98262">最后右键运行查看结果，如下图所示：</p>
<p data-nodeid="98263" class=""><img src="https://s0.lgstatic.com/i/image/M00/45/CC/Ciqc1F9DfpmAYN7DAANS_nOGo3g194.png" alt="Drawing 3.png" data-nodeid="98267"></p>




<p data-nodeid="96186">到此为止，我们三种场景的完整代码实现就完成了。</p>
<h3 data-nodeid="96187">总结</h3>
<p data-nodeid="98452">这一课时我们分别对连续登录、交易活跃用户、超时未支付三种业务场景进行了完整的代码实现，我们实际业务场景中可以根据本节课的内容灵活处理。</p>

---

### 精选评论


