<p data-nodeid="39450" class="">本课时我将为你讲解 ShardingSphere 中的分布式主键实现方式。</p>


<p data-nodeid="38942">在传统数据库软件开发过程中，主键自动生成技术是基本需求。各个数据库对该需求也提供了相应的支持，比如 MySQL 的自增键，Oracle 的自增序列等。而在分片场景下，问题就变得有点复杂，我们不能依靠单个实例上的自增键来实现不同数据节点之间的全局唯一主键，这时分布式主键的需求就应运而生。ShardingSphere 作为一款优秀的分库分表开源软件，同样提供了分布式主键的实现机制，今天，我们就对这一机制的基本原理和实现方式展开讨论。</p>
<h3 data-nodeid="38943">ShardingSphere 中的自动生成键方案</h3>
<p data-nodeid="38944">在介绍 ShardingSphere 提供的具体分布式主键实现方式之前，我们有必要先对框架中抽象的自动生成键 GeneratedKey 方案进行讨论，从而帮助你明确分布式主键的具体使用场景和使用方法。</p>
<h4 data-nodeid="38945">ShardingSphere 中的 GeneratedKey</h4>
<p data-nodeid="38946">GeneratedKey 并不是 ShardingSphere 所创造的概念。如果你熟悉 Mybatis 这种 ORM 框架，对它就不会陌生。事实上，我们在《数据分片：如何实现分库、分表、分库+分表以及强制路由（上）？》中已经介绍了在 Mybatis 中嵌入 GeneratedKey 的实现方法。通常，我们会在 Mybatis 的 Mapper 文件中设置 useGeneratedKeys 和 keyProperty 属性：</p>
<pre class="lang-xml" data-nodeid="38947"><code data-language="xml">&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">insert</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"addEntity"</span> <span class="hljs-attr">useGeneratedKeys</span>=<span class="hljs-string">"true"</span> <span class="hljs-attr">keyProperty</span>=<span class="hljs-string">"recordId"</span> &gt;</span> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; INSERT INTO health_record (user_id, level_id, remark)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; VALUES (#{userId,jdbcType=INTEGER}, #{levelId,jdbcType=INTEGER},  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; #{remark,jdbcType=VARCHAR}) 
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">insert</span>&gt;</span> 
</code></pre>
<p data-nodeid="38948">在执行这个 insert 语句时，返回的对象中自动包含了生成的主键值。当然，这种方式能够生效的前提是对应的数据库本身支持自增长的主键。</p>
<p data-nodeid="40131" class="">当我们使用 ShardingSphere 提供的自动生成键方案时，开发过程以及效果和上面描述的完全一致。在 ShardingSphere 中，同样实现了一个 GeneratedKey 类。<strong data-nodeid="40137">请注意，该类位于 sharding-core-route 工程下</strong>。我们先看该类提供的 getGenerateKey 方法：</p>


<pre class="lang-java" data-nodeid="38950"><code data-language="java">&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Optional&lt;GeneratedKey&gt; <span class="hljs-title">getGenerateKey</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> TableMetas tableMetas, <span class="hljs-keyword">final</span> List&lt;Object&gt; parameters, <span class="hljs-keyword">final</span> InsertStatement insertStatement)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//找到自增长列 </span>
&nbsp;&nbsp;&nbsp;  Optional&lt;String&gt; generateKeyColumnName = shardingRule.findGenerateKeyColumnName(insertStatement.getTable().getTableName()); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!generateKeyColumnName.isPresent()) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> Optional.absent(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断自增长类是否已生成主键值 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Optional.of(containsGenerateKey(tableMetas, insertStatement, generateKeyColumnName.get()) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ? findGeneratedKey(tableMetas, parameters, insertStatement, generateKeyColumnName.get()) : createGeneratedKey(shardingRule, insertStatement, generateKeyColumnName.get())); 
	} 
</code></pre>
<p data-nodeid="38951">这段代码的逻辑在于先从 ShardingRule 中找到主键对应的 Column，然后判断是否已经包含主键：如果是则找到该主键，如果不是则生成新的主键。今天，我们的重点是分布式主键的生成，所以我们直接来到 createGeneratedKey 方法：</p>
<pre class="lang-java" data-nodeid="38952"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> GeneratedKey <span class="hljs-title">createGeneratedKey</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ShardingRule shardingRule, <span class="hljs-keyword">final</span> InsertStatement insertStatement, <span class="hljs-keyword">final</span> String generateKeyColumnName)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; GeneratedKey result = <span class="hljs-keyword">new</span> GeneratedKey(generateKeyColumnName, <span class="hljs-keyword">true</span>); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; insertStatement.getValueListCount(); i++) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.getGeneratedValues().add(shardingRule.generateKey(insertStatement.getTable().getTableName())); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result; 
	} 
</code></pre>
<p data-nodeid="38953">在 GeneratedKey 中存在一个类型为 LinkedList 的 generatedValues 变量，用于保存生成的主键，但实际上，生成主键的工作转移到了 ShardingRule 的 generateKey 方法中，我们跳转到 ShardingRule 类并找到这个 generateKey 方法：</p>
<pre class="lang-java" data-nodeid="38954"><code data-language="java">&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> Comparable&lt;?&gt; generateKey(<span class="hljs-keyword">final</span> String logicTableName) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Optional&lt;TableRule&gt; tableRule = findTableRule(logicTableName); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!tableRule.isPresent()) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingConfigurationException(<span class="hljs-string">"Cannot find strategy for generate keys."</span>); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//从TableRule中获取ShardingKeyGenerator并生成分布式主键 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ShardingKeyGenerator shardingKeyGenerator = <span class="hljs-keyword">null</span> == tableRule.get().getShardingKeyGenerator() ? defaultShardingKeyGenerator : tableRule.get().getShardingKeyGenerator(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> shardingKeyGenerator.generateKey(); 
	} 
</code></pre>
<p data-nodeid="38955">首先，根据传入的 logicTableName 找到对应的 TableRule，基于 TableRule 找到其包含的 ShardingKeyGenerator，然后通过 ShardingKeyGenerator 的 generateKey 来生成主键。从设计模式上讲，ShardingRule 也只是一个外观类，真正创建 ShardingKeyGenerator 的过程应该是在 TableRule 中。而这里的 ShardingKeyGenerator 显然就是真正生成分布式主键入口，让我们来看一下。</p>
<h4 data-nodeid="38956">ShardingKeyGenerator</h4>
<p data-nodeid="38957">接下来我们分析 ShardingKeyGenerator 接口，从定义上看，该接口继承了 TypeBasedSPI 接口：</p>
<pre class="lang-java" data-nodeid="38958"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ShardingKeyGenerator</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">TypeBasedSPI</span> </span>{&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; Comparable&lt;?&gt; generateKey(); 
} 
</code></pre>
<p data-nodeid="38959">来到 TableRule 中，在它的一个构造函数中找到了 ShardingKeyGenerator 的创建过程：</p>
<pre class="lang-java" data-nodeid="38960"><code data-language="java">shardingKeyGenerator = containsKeyGeneratorConfiguration(tableRuleConfig) 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ? <span class="hljs-keyword">new</span> ShardingKeyGeneratorServiceLoader().newService(tableRuleConfig.getKeyGeneratorConfig().getType(), tableRuleConfig.getKeyGeneratorConfig().getProperties()) : <span class="hljs-keyword">null</span>; 
</code></pre>
<p data-nodeid="38961">这里有一个 ShardingKeyGeneratorServiceLoader 类，该类定义如下：</p>
<pre class="lang-java" data-nodeid="38962"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingKeyGeneratorServiceLoader</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">TypeBasedSPIServiceLoader</span>&lt;<span class="hljs-title">ShardingKeyGenerator</span>&gt; </span>{ 
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">static</span> { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; NewInstanceServiceLoader.register(ShardingKeyGenerator.class); 
&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ShardingKeyGeneratorServiceLoader</span><span class="hljs-params">()</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(ShardingKeyGenerator.class); 
&nbsp;&nbsp;&nbsp; } 
} 
</code></pre>
<p data-nodeid="38963">回顾上一课时的内容，我们不难理解 ShardingKeyGeneratorServiceLoader 类的作用。ShardingKeyGeneratorServiceLoader 继承了 TypeBasedSPIServiceLoader 类，并在静态方法中通过 NewInstanceServiceLoader 注册了类路径中所有的 ShardingKeyGenerator。然后，ShardingKeyGeneratorServiceLoader 的 newService 方法基于类型参数通过 SPI 创建实例，并赋值 Properties 属性。</p>
<p data-nodeid="38964">通过继承 TypeBasedSPIServiceLoader 类来创建一个新的 ServiceLoader 类，然后在其静态方法中注册相应的 SPI 实现，这是 ShardingSphere 中应用微内核模式的常见做法，很多地方都能看到类似的处理方法。</p>
<p data-nodeid="40804">我们在 sharding-core-common 工程的 META-INF/services 目录中看到了具体的 SPI 定义：</p>
<p data-nodeid="41148"><img src="https://s0.lgstatic.com/i/image/M00/3A/E1/Ciqc1F8iliiAVywgAAByh__z6Bw582.png" alt="1.png" data-nodeid="41152"></p>
<div data-nodeid="41487" class=""><p style="text-align:center">分布式主键 SPI 配置</p> </div>






<p data-nodeid="38968">可以看到，这里有两个 ShardingKeyGenerator，分别是 SnowflakeShardingKeyGenerator 和 UUIDShardingKeyGenerator，它们都位于org.apache.shardingsphere.core.strategy.keygen 包下。</p>
<h3 data-nodeid="38969">ShardingSphere 中的分布式主键实现方案</h3>
<p data-nodeid="38970">在 ShardingSphere 中，ShardingKeyGenerator 接口存在一批实现类。除了前面提到的 SnowflakeShardingKeyGenerator 和UUIDShardingKeyGenerator，还实现了 LeafSegmentKeyGenerator 和 LeafSnowflakeKeyGenerator 类，但这两个类的实现过程有些特殊，我们一会再具体展开。</p>
<h4 data-nodeid="38971">UUIDShardingKeyGenerator</h4>
<p data-nodeid="38972">我们先来看最简单的 ShardingKeyGenerator，即 UUIDShardingKeyGenerator。UUIDShardingKeyGenerator 的实现非常容易理解，直接采用 UUID.randomUUID() 的方式产生分布式主键：</p>
<pre class="lang-java" data-nodeid="38973"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UUIDShardingKeyGenerator</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ShardingKeyGenerator</span> </span>{ 
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Properties properties = <span class="hljs-keyword">new</span> Properties(); 
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getType</span><span class="hljs-params">()</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"UUID"</span>; 
&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> Comparable&lt;?&gt; generateKey() { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> UUID.randomUUID().toString().replaceAll(<span class="hljs-string">"-"</span>, <span class="hljs-string">""</span>); 
&nbsp;&nbsp;&nbsp; } 
} 
</code></pre>
<h4 data-nodeid="38974">SnowflakeShardingKeyGenerator</h4>
<p data-nodeid="42150">再来看 SnowFlake（雪花）算法，SnowFlake 是 ShardingSphere 默认的分布式主键生成策略。它是 Twitter 开源的分布式 ID 生成算法，其核心思想是使用一个 64bit 的 long 型数字作为全局唯一 ID，且 ID 引入了时间戳，基本上能够保持自增。SnowFlake 算法在分布式系统中的应用十分广泛，SnowFlake 算法中 64bit 的详细结构存在一定的规范：</p>
<p data-nodeid="42835"><img src="https://s0.lgstatic.com/i/image/M00/3A/ED/CgqCHl8ilkuAHxUeAAHYgqa5Z0Q435.png" alt="2.png" data-nodeid="42839"></p>
<div data-nodeid="42836" class=""><p style="text-align:center">64bit 的 ID 结构图</p> </div>
<p></p>





<p data-nodeid="38978">在上图中，我们把 64bit 分成了四个部分：</p>
<ul data-nodeid="38979">
<li data-nodeid="38980">
<p data-nodeid="38981">符号位</p>
</li>
</ul>
<p data-nodeid="38982">第一个部分即第一个 bit，值为 0，没有实际意义。</p>
<ul data-nodeid="38983">
<li data-nodeid="38984">
<p data-nodeid="38985">时间戳位</p>
</li>
</ul>
<p data-nodeid="38986">第二个部分是 41 个 bit，表示的是时间戳。41 位的时间戳可以容纳的毫秒数是 2 的 41 次幂，一年所使用的毫秒数是365 * 24 * 60 * 60 * 1000，即 69.73 年。 <strong data-nodeid="39080">也就是说，ShardingSphere 的 SnowFlake 算法的时间纪元从 2016 年 11 月 1 日零点开始，可以使用到 2086 年</strong> ，相信能满足绝大部分系统的要求。</p>
<ul data-nodeid="38987">
<li data-nodeid="38988">
<p data-nodeid="38989">工作进程位</p>
</li>
</ul>
<p data-nodeid="38990">第三个部分是 10 个 bit，表示工作进程位，其中前 5 个 bit 代表机房 id，后 5 个 bit 代表机器id。</p>
<ul data-nodeid="38991">
<li data-nodeid="38992">
<p data-nodeid="38993">序列号位</p>
</li>
</ul>
<p data-nodeid="38994">第四个部分是 12 个 bit，表示序号，也就是某个机房某台机器上在一毫秒内同时生成的 ID 序号。如果在这个毫秒内生成的数量超过 4096（即 2 的 12 次幂），那么生成器会等待下个毫秒继续生成。</p>
<p data-nodeid="43511" class="">因为 SnowFlake 算法依赖于时间戳，所以还需要考虑时钟回拨这种场景。<strong data-nodeid="43517">所谓时钟回拨，是指服务器因为时间同步，导致某一部分机器的时钟回到了过去的时间点</strong>。显然，时间戳的回滚会导致生成一个已经使用过的 ID，因此默认分布式主键生成器提供了一个最大容忍的时钟回拨毫秒数。如果时钟回拨的时间超过最大容忍的毫秒数阈值，则程序报错；如果在可容忍的范围内，默认分布式主键生成器会等待时钟同步到最后一次主键生成的时间后再继续工作。ShardingSphere 中最大容忍的时钟回拨毫秒数的默认值为 0，可通过属性设置。</p>


<p data-nodeid="38996">了解了 SnowFlake 算法的基本概念之后，我们来看 SnowflakeShardingKeyGenerator 类的具体实现。首先在 SnowflakeShardingKeyGenerator 类中存在一批常量的定义，用于维护 SnowFlake 算法中各个 bit 之间的关系，同时还存在一个 TimeService 用于获取当前的时间戳。而 SnowflakeShardingKeyGenerator 的核心方法 generateKey 负责生成具体的 ID，我们这里给出详细的代码，并为每行代码都添加注释：</p>
<pre class="lang-java" data-nodeid="38997"><code data-language="java">    <span class="hljs-meta">@Override</span> 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">synchronized</span> Comparable&lt;?&gt; generateKey() { 
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取当前时间戳 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">long</span> currentMilliseconds = timeService.getCurrentMillis(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//如果出现了时钟回拨，则抛出异常或进行时钟等待 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (waitTolerateTimeDifferenceIfNeed(currentMilliseconds)) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; currentMilliseconds = timeService.getCurrentMillis(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//如果上次的生成时间与本次的是同一毫秒 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (lastMilliseconds == currentMilliseconds) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  <span class="hljs-comment">//这个位运算保证始终就是在4096这个范围内，避免你自己传递的sequence超过了4096这个范围 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-number">0L</span> == (sequence = (sequence + <span class="hljs-number">1</span>) &amp; SEQUENCE_MASK)) { 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-comment">//如果位运算结果为0，则需要等待下一个毫秒继续生成 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; currentMilliseconds = waitUntilNextTime(currentMilliseconds); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {<span class="hljs-comment">//如果不是，则生成新的sequence </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; vibrateSequenceOffset(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sequence = sequenceOffset; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lastMilliseconds = currentMilliseconds; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//先将当前时间戳左移放到完成41个bit，然后将工作进程为左移到10个bit，再将序号为放到最后的12个bit </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//最后拼接起来成一个64 bit的二进制数字 </span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ((currentMilliseconds - EPOCH) &lt;&lt; TIMESTAMP_LEFT_SHIFT_BITS) | (getWorkerId() &lt;&lt; WORKER_ID_LEFT_SHIFT_BITS) | sequence; 
	} 
</code></pre>
<p data-nodeid="38998">可以看到这里综合考虑了时钟回拨、同一个毫秒内请求等设计要素，从而完成了 SnowFlake 算法的具体实现。</p>
<h4 data-nodeid="38999">LeafSegmentKeyGenerator 和 LeafSnowflakeKeyGenerator</h4>
<p data-nodeid="39000">事实上，如果实现类似 SnowflakeShardingKeyGenerator 这样的 ShardingKeyGenerator 是比较困难的，而且也属于重复造轮子。因此，尽管 ShardingSphere 在 4.X 版本中也提供了 LeafSegmentKeyGenerator 和 LeafSnowflakeKeyGenerator 这两个 ShardingKeyGenerator 的完整实现类。但在正在开发的 5.X 版本中，这两个实现类被移除了。</p>
<p data-nodeid="39001">目前，ShardingSphere 专门提供了 OpenSharding 这个代码仓库来存放新版本的 LeafSegmentKeyGenerator 和 LeafSnowflakeKeyGenerator。新版本的实现类直接采用了第三方美团提供的 Leaf 开源实现。</p>
<p data-nodeid="39002">Leaf 提供两种生成 ID 的方式，一种是号段（Segment）模式，一种是前面介绍的 Snowflake 模式。无论使用哪种模式，我们都需要提供一个 leaf.properties 文件，并设置对应的配置项。无论是使用哪种方式，应用程序都需要设置一个 leaf.key：</p>
<pre class="lang-xml" data-nodeid="39003"><code data-language="xml"># for keyGenerator key 
leaf.key=sstest 
&nbsp; 
# for LeafSnowflake 
leaf.zk.list=localhost:2181 
</code></pre>
<p data-nodeid="39004">如果使用号段模式，需要依赖于一张数据库表来存储运行时数据，因此需要在 leaf.properties 文件中添加数据库的相关配置：</p>
<pre class="lang-plain" data-nodeid="39005"><code data-language="plain"># for LeafSegment 
leaf.jdbc.url=jdbc:mysql://127.0.0.1:3306/test?serverTimezone=UTC&amp;useSSL=false 
leaf.jdbc.username=root 
leaf.jdbc.password=123456 
</code></pre>
<p data-nodeid="39006">基于这些配置，我们就可以创建对应的 DataSource，并进一步创建用于生成分布式 ID 的 IDGen 实现类，这里创建的是基于号段模式的 SegmentIDGenImpl 实现类：</p>
<pre class="lang-java" data-nodeid="39007"><code data-language="java"><span class="hljs-comment">//通过DruidDataSource构建数据源并设置属性 </span>
DruidDataSource dataSource = <span class="hljs-keyword">new</span> DruidDataSource(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSource.setUrl(properties.getProperty(LeafPropertiesConstant.LEAF_JDBC_URL)); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSource.setUsername(properties.getProperty(LeafPropertiesConstant.LEAF_JDBC_USERNAME)); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSource.setPassword(properties.getProperty(LeafPropertiesConstant.LEAF_JDBC_PASSWORD)); 
dataSource.init(); 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  
<span class="hljs-comment">//构建数据库访问Dao组件 </span>
IDAllocDao dao = <span class="hljs-keyword">new</span> IDAllocDaoImpl(dataSource); 
<span class="hljs-comment">//创建IDGen实现类 </span>
<span class="hljs-keyword">this</span>.idGen = <span class="hljs-keyword">new</span> SegmentIDGenImpl(); 
<span class="hljs-comment">//将Dao组件绑定到IDGen实现类 </span>
&nbsp;((SegmentIDGenImpl) <span class="hljs-keyword">this</span>.idGen).setDao(dao); 
<span class="hljs-keyword">this</span>.idGen.init(); 
<span class="hljs-keyword">this</span>.dataSource = dataSource; 
</code></pre>
<p data-nodeid="39008">一旦我们成功创建了 IDGen 实现类，可以通过该类来生成目标 ID，LeafSegmentKeyGenerator 类中包含了所有的实现细节：</p>
<pre class="lang-java" data-nodeid="39009"><code data-language="java">Result result = <span class="hljs-keyword">this</span>.idGen.get(properties.getProperty(LeafPropertiesConstant.LEAF_KEY)); 
<span class="hljs-keyword">return</span> result.getId(); 
</code></pre>
<p data-nodeid="39010">介绍完 LeafSegmentKeyGenerator 之后，我们再来看 LeafSnowflakeKeyGenerator。LeafSnowflakeKeyGenerator 的实现依赖于分布式协调框架 Zookeeper，所以在配置文件中需要指定 Zookeeper 的目标地址：</p>
<pre class="lang-xml" data-nodeid="39011"><code data-language="xml"># for LeafSnowflake 
leaf.zk.list=localhost:2181 
</code></pre>
<p data-nodeid="39012">创建用于 LeafSnowflake 的 IDGen 实现类 SnowflakeIDGenImpl 相对比较简单，我们直接在构造函数中设置 Zookeeper 地址就可以了：</p>
<pre class="lang-xml" data-nodeid="39013"><code data-language="xml">IDGen idGen = new SnowflakeIDGenImpl(properties.getProperty(LeafPropertiesConstant.LEAF_ZK_LIST), 8089); 
</code></pre>
<p data-nodeid="39014">同样，通过 IDGen 获取模板 ID 的方式是一致的：</p>
<hr data-nodeid="39015">
<pre class="lang-xml" data-nodeid="39016"><code data-language="xml">idGen.get(properties.getProperty(LeafPropertiesConstant.LEAF_KEY)).getId(); 
</code></pre>
<p data-nodeid="39017">显然，基于 Leaf 框架实现号段模式和 Snowflake 模式下的分布式 ID 生成方式非常简单，Leaf 框架为我们屏蔽了内部实现的复杂性。</p>
<h3 data-nodeid="39018">从源码解析到日常开发</h3>
<p data-nodeid="39019">相比 ShardingSphere 中其他架构设计上的思想和实现方案，分布式主键非常独立，所以今天介绍的各种分布式主键的实现方式完全可以直接套用到日常开发过程中。无论是 ShardingSphere 自身实现的 SnowflakeShardingKeyGenerator，还是基于第三方框架实现的 LeafSegmentKeyGenerator 和 LeafSnowflakeKeyGenerator，都为我们使用分布式主键提供了直接的解决方案。当然，我们也可以在这些实现方案的基础上，进一步挖掘同类型的其他方案。</p>
<h3 data-nodeid="39020">总结</h3>
<p data-nodeid="39021">在分布式系统的开发过程中，分布式主键是一种基础需求。而对于与数据库相关的操作而言，我们往往需要将分布式主键与数据库的主键自动生成机制关联起来。在今天的课程中，我们就从 ShardingSphere 的自动生成键方案说起，引出了分布式主键的各种实现方案。这其中包括最简单的 UUID，也包括经典的雪花算法，以及雪花算法的改进方案 LeafSegment 和 LeafSnowflake 算法。</p>
<p data-nodeid="39022">这里给你留一道思考题：ShardingSphere 中如何分别实现基于号段的 Leaf 以及基于 Snowflake 的 Leaf 来生成分布式 ID？</p>
<p data-nodeid="39023">从下一课时开始，我们将进入到 ShardingSphere 分片引擎实现原理的讲解过程中，我将首先为你介绍解析引擎的执行流程，记得按时来听课。</p>

---

### 精选评论

##### **0619：
> 如果业务系统需要自定义一套生成规则，只有基于spi对应实现嘛？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; ShardingSphere是一个可插拔的框架，推荐基于SPI的方式实现定制化需求

##### **川：
> 写的好！

##### **杰：
> 打卡

