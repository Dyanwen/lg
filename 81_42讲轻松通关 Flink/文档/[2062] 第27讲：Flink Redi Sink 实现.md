<p data-nodeid="6456">我们在第 12 课时“Flink 常用的 Source 和 Connector”中提过 Flink 提供了比较丰富的用来连接第三方的连接器，可以在<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/connectors/" data-nodeid="6511">官网</a>中找到 Flink 支持的各种各样的连接器。</p>
<p data-nodeid="6457">此外，Flink 还会基于 Apache Bahir 发布一些 Connector，其中就有我们非常熟悉的 Redis。很多人在 Flink 项目中访问 Redis 的方法都是自己进行实现的，我们也可以使用 Bahir 实现的 Redis 连接器。</p>
<p data-nodeid="6458">事实上，使用 Redis Sink 常用的方法有很多，比如自定义 Sink 函数、依赖开源的 Redis Sink 实现等。</p>
<p data-nodeid="6459">下面我们就分别介绍常用的 Redis Sink 实现。</p>
<h3 data-nodeid="7031" class="">自定义 Redis Sink</h3>

<blockquote data-nodeid="7488">
<p data-nodeid="7489">REmote DIctionary Server（Redis）是一个由 Salvatore Sanfilippo 写的 key-value 存储系统。Redis 是一个开源的使用 ANSI C 语言编写、遵守 BSD 协议、支持网络、可基于内存亦可持久化的日志型、Key-Value 数据库，并提供多种语言的 API。</p>
<p data-nodeid="7490" class="">它通常被称为数据结构服务器，因为值（value）可以是字符串（String）、哈希（Hash）、列表（List）、集合（Sets）和有序集合（Sorted Sets）等类型。</p>
</blockquote>


<p data-nodeid="6465">如果你对 Redis 不熟悉，可以参考<a href="https://redis.io/" data-nodeid="6522">官网</a>上的说明。 点击<a href="http://download.redis.io/releases/" data-nodeid="6526">这里</a>下载一个稳定版本的 Redis，我在本地安装的是 2.8.5 版本。使用下面命令进行安装：</p>
<pre class="lang-java" data-nodeid="8055"><code data-language="java">wget http:<span class="hljs-comment">//download.redis.io/releases/redis-2.8.5.tar.gz</span>
tar xzf redis-<span class="hljs-number">2.8</span>.<span class="hljs-number">5.</span>tar.gz
cd redis-<span class="hljs-number">2.8</span>.<span class="hljs-number">5</span>
make
</code></pre>



<p data-nodeid="6467">然后进入 src 目录，就可以在本地启动一个 Redis 单机实例。</p>
<pre class="lang-java" data-nodeid="10780"><code data-language="java">src/redis-server
</code></pre>
<p data-nodeid="10781" class=""><img src="https://s0.lgstatic.com/i/image/M00/33/53/Ciqc1F8P95WAMtj9AAEeDYpGXyo212.png" alt="image (4).png" data-nodeid="10788"></p>





<p data-nodeid="8929">我们可以使用 Redis 自带的交互命令来测试 Redis 实例，如下图所示：</p>
<p data-nodeid="8930" class=""><img src="https://s0.lgstatic.com/i/image/M00/33/5E/CgqCHl8P92iAflKfAABAnOh-e38133.png" alt="image (5).png" data-nodeid="8938"></p>


<p data-nodeid="6472">从上图可以看到，我们通过命令可以向 Redis 中设置数据。</p>
<p data-nodeid="6473">然后在使用 Redis 时需要新增新的依赖，你可以点击<a href="https://mvnrepository.com/artifact/redis.clients/jedis" data-nodeid="6540">这里</a>根据 Redis 版本找到对应的 Redis 依赖：</p>
<pre class="lang-xml" data-nodeid="9623"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>redis.clients<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>jedis<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
   <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.8.2<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>


<p data-nodeid="6475">我们自定义一个 RedisSink 函数，继承 RichSinkFunction，重写其中的方法：</p>
<pre class="lang-java" data-nodeid="6476"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SelfRedisSink</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RichSinkFunction</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">transient</span> Jedis jedis;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">open</span><span class="hljs-params">(Configuration config)</span> </span>{
        jedis = <span class="hljs-keyword">new</span> Jedis(<span class="hljs-string">"localhost"</span>, <span class="hljs-number">6379</span>);
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">invoke</span><span class="hljs-params">(Tuple2&lt;String, String&gt; value, Context context)</span> <span class="hljs-keyword">throws</span> Exception </span>{
        <span class="hljs-keyword">if</span> (!jedis.isConnected()) {
            jedis.connect();
        }
        jedis.set(value.f0, value.f1);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">close</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
        jedis.close();
    }
}
</code></pre>
<p data-nodeid="6477">上述代码，我们继承了 RichSinkFunction 并且实现了其中的 open、invoke 和 close 方法。其中 open 用于新建 Redis 客户端；invoke 函数将数据存储到 Redis 中，我们在这里将数据以字符串的形式存储到 Redis 中；close 函数用于使用完毕后关闭 Redis 客户端。</p>
<p data-nodeid="6478">我们使用自定义 Redis Sink 的优点在于可以根据业务需求灵活处理，可以方便地使用 Jedis 提供的各种能力。</p>
<h3 data-nodeid="9852" class="">使用开源 Redis Connector</h3>

<p data-nodeid="6480">常用的开源 Connector 有两个，分别是 Flink 和 Bahir 提供的实现，其内部都是使用了 Java Redis 客户端&nbsp;Jedis&nbsp;实现了 Redis Sink。我们分别看一下它们的使用方法。</p>
<h4 data-nodeid="6481">第一种：Flink 提供的依赖包</h4>
<p data-nodeid="6482">新增以下依赖：</p>
<pre class="lang-xml" data-nodeid="6483"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.flink<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>flink-connector-redis_2.11<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>1.1.5<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="6484">我们可以通过实现 RedisMapper 来自定义 Redis Sink：</p>
<pre class="lang-java" data-nodeid="6485"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedisSink</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RedisMapper</span>&lt;<span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>&gt;&gt;</span>{
    <span class="hljs-comment">/**
     * 设置 Redis 数据类型
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> RedisCommandDescription <span class="hljs-title">getCommandDescription</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> RedisCommandDescription(RedisCommand.SET);
    }
    <span class="hljs-comment">/**
     * 设置Key
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKeyFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f0;
    }
    <span class="hljs-comment">/**
     * 设置value
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getValueFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f1;
    }
}
</code></pre>
<p data-nodeid="6486">我们自定义的 RedisSink 实现了 RedisMapper 并覆写了其中的 getCommandDescription、getKeyFromData、getValueFromData。其中 getCommandDescription 定义了存储到 Redis 中的数据格式，这里我们定义的 RedisCommand 为 SET，将数据以字符串的形式存储；getKeyFromData 定义了 SET 的 Key，getValueFromData 定义了 SET 的值。</p>
<p data-nodeid="6487">完整的使用方式为：</p>
<pre class="lang-java" data-nodeid="6488"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception</span>{
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    DataStream&lt;Tuple2&lt;String, String&gt;&gt; stream = env.fromElements(<span class="hljs-string">"Flink"</span>,<span class="hljs-string">"Spark"</span>,<span class="hljs-string">"Storm"</span>).map(<span class="hljs-keyword">new</span> MapFunction&lt;String, Tuple2&lt;String, String&gt;&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> Tuple2&lt;String, String&gt; <span class="hljs-title">map</span><span class="hljs-params">(String s)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Tuple2&lt;&gt;(s, s);
        }
    });
    FlinkJedisPoolConfig conf = <span class="hljs-keyword">new</span> FlinkJedisPoolConfig.Builder().setHost(<span class="hljs-string">"localhost"</span>).setPort(<span class="hljs-number">6379</span>).build();
    stream.addSink(<span class="hljs-keyword">new</span> RedisSink&lt;&gt;(conf, <span class="hljs-keyword">new</span> RedisSink01()));
    env.execute(<span class="hljs-string">"redis sink01"</span>);
}
</code></pre>
<p data-nodeid="10302">我们可以在 Redis 的控制台中查询数据，如下图所示：</p>
<p data-nodeid="10303" class=""><img src="https://s0.lgstatic.com/i/image/M00/33/5E/CgqCHl8P94eAdc6BAABVXKIV0Sk143.png" alt="image (10).png" data-nodeid="10311"></p>


<p data-nodeid="6491">由图可知，我们设置了 Redis 中的数据。</p>
<h4 data-nodeid="6492">第二种：Bahir 提供的依赖包</h4>
<p data-nodeid="6493">新增以下依赖：</p>
<pre class="lang-java" data-nodeid="6494"><code data-language="java">&lt;dependency&gt;
   &lt;groupId&gt;org.apache.bahir&lt;/groupId&gt;
   &lt;artifactId&gt;flink-connector-redis_2.11&lt;/artifactId&gt;
   &lt;version&gt;1.0&lt;/version&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="6495">目前该开源的 Bahir 实现最新版本是 1.1-snapshot，在这里使用的是 1.0 稳定版本。同样，我们也可以通过实现 RedisMapper 来自定义 Redis Sink：</p>
<pre class="lang-java" data-nodeid="6496"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedisSink02</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">RedisMapper</span>&lt;<span class="hljs-title">Tuple2</span>&lt;<span class="hljs-title">String</span>, <span class="hljs-title">String</span>&gt;&gt; </span>{
    <span class="hljs-comment">/**
     * 设置 Redis 数据类型
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> RedisCommandDescription <span class="hljs-title">getCommandDescription</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> RedisCommandDescription(RedisCommand.SET);
    }
    <span class="hljs-comment">/**
     * 设置 Key
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getKeyFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f0;
    }
    <span class="hljs-comment">/**
     * 设置 value
     */</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getValueFromData</span><span class="hljs-params">(Tuple2&lt;String, String&gt; data)</span> </span>{
        <span class="hljs-keyword">return</span> data.f1;
    }
}
</code></pre>
<p data-nodeid="6497">和第一种方式一样，完整的使用方式如下代码所示：</p>
<pre class="lang-java" data-nodeid="6498"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Exception</span>{
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    DataStream&lt;Tuple2&lt;String, String&gt;&gt; stream = env.fromElements(<span class="hljs-string">"Flink"</span>,<span class="hljs-string">"Spark"</span>,<span class="hljs-string">"Storm"</span>).map(<span class="hljs-keyword">new</span> MapFunction&lt;String, Tuple2&lt;String, String&gt;&gt;() {
        <span class="hljs-meta">@Override</span>
        <span class="hljs-function"><span class="hljs-keyword">public</span> Tuple2&lt;String, String&gt; <span class="hljs-title">map</span><span class="hljs-params">(String s)</span> <span class="hljs-keyword">throws</span> Exception </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Tuple2&lt;&gt;(s, s+<span class="hljs-string">"_sink2"</span>);
        }
    });
    FlinkJedisPoolConfig conf = <span class="hljs-keyword">new</span> FlinkJedisPoolConfig.Builder().setHost(<span class="hljs-string">"localhost"</span>).setPort(<span class="hljs-number">6379</span>).build();
    stream.addSink(<span class="hljs-keyword">new</span> RedisSink&lt;&gt;(conf, <span class="hljs-keyword">new</span> RedisSink02()));
    env.execute(<span class="hljs-string">"redis sink02"</span>);
}
</code></pre>
<p data-nodeid="11277">我们可以在 Redis 的控制台中查询数据，如下图所示：</p>
<p data-nodeid="11278" class=""><img src="https://s0.lgstatic.com/i/image/M00/33/53/Ciqc1F8P96WAPoW3AABHf0rerEo843.png" alt="image (11).png" data-nodeid="11286"></p>


<p data-nodeid="6501">开源实现的 Redis Connector 使用非常方便，但是有些功能缺失，例如，无法使用一些 Jedis 中的高级功能如设置过期时间等。</p>
<p data-nodeid="6502">所以我们在实际生产中可以根据自己业务需要使用不同的实现。</p>
<h3 data-nodeid="11543" class="">总结</h3>

<p data-nodeid="6504">这一课时我们使用自定义 Redis Sink、开源的 Redis Connector 实现了写入 Redis，Redis Sink 的实现可以根据业务需要进行实现，Redis 以其极高的写入读取性能，是我们经常使用的 Sink 之一。学完本课时的内容后，你将掌握如何定义 Redis Sink 及其实现。</p>

---

### 精选评论

##### **6513：
> 老师 我是用springboot框架写的，从redis充获取到数据融合后缓存到内存，目前遇到一个问题在idea 中调试 flink可以获取到缓存的数据，但是提交到flink环境后就无法获取到缓存的数据，显示缓存为空，您能帮忙分析下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在编写Flink代码时，尽量避免使用spring之类的框架，因为没有必要。只要依赖Flink必要的包和一些工具类即可。

##### **涛：
> 老师，maven库找不到这个依赖，工程中一直无法引入

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你可以在阿里云的镜像库中找到：https://maven.aliyun.com/mvn/search

