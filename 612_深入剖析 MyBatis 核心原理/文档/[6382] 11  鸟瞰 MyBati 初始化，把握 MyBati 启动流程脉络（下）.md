<p data-nodeid="439621" class="">在上一讲，我们深入分析了MyBatis 初始化过程中对 mybatis-config.xml 全局配置文件的解析，详细介绍了其中每个标签的解析流程以及涉及的经典设计模式——构造者模式。这一讲我们就紧接着上一讲的内容，继续介绍 MyBatis 初始化流程，重点介绍Mapper.xml 配置文件的解析以及 SQL 语句的处理逻辑。</p>
<h3 data-nodeid="439622">Mapper.xml 映射文件解析全流程</h3>
<p data-nodeid="439623">在上一讲分析 mybatis-config.xml 配置文件解析流程的时候我们看到，在 mybatis-config.xml 配置文件中可以定义多个 <code data-backticks="1" data-nodeid="439813">&lt;mapper&gt;</code> 标签指定 Mapper 配置文件的地址，<strong data-nodeid="439819">MyBatis 会为每个 Mapper.xml 映射文件创建一个 XMLMapperBuilder 实例完成解析</strong>。</p>
<p data-nodeid="439624">与 XMLConfigBuilder 类似，XMLMapperBuilder也是具体构造者的角色，继承了 BaseBuilder 这个抽象类，解析 Mapper.xml 映射文件的入口是 XMLMapperBuilder.parse() 方法，其核心步骤如下：</p>
<ul data-nodeid="439625">
<li data-nodeid="439626">
<p data-nodeid="439627">执行 configurationElement() 方法解析整个Mapper.xml 映射文件的内容；</p>
</li>
<li data-nodeid="439628">
<p data-nodeid="439629">获取当前 Mapper.xml 映射文件指定的 Mapper 接口，并进行注册；</p>
</li>
<li data-nodeid="439630">
<p data-nodeid="439631">处理 configurationElement() 方法中解析失败的 <code data-backticks="1" data-nodeid="439824">&lt;resultMap&gt;</code> 标签；</p>
</li>
<li data-nodeid="439632">
<p data-nodeid="439633">处理 configurationElement() 方法中解析失败的 <code data-backticks="1" data-nodeid="439827">&lt;cache-ref&gt;</code> 标签；</p>
</li>
<li data-nodeid="439634">
<p data-nodeid="439635">处理 configurationElement() 方法中解析失败的SQL 语句标签。</p>
</li>
</ul>
<p data-nodeid="439636">可以清晰地看到，<strong data-nodeid="439835">configurationElement() 方法才是真正解析 Mapper.xml 映射文件的地方</strong>，其中定义了处理 Mapper.xml 映射文件的核心流程：</p>
<ul data-nodeid="439637">
<li data-nodeid="439638">
<p data-nodeid="439639">获取 <code data-backticks="1" data-nodeid="439837">&lt;mapper&gt;</code> 标签中的 namespace 属性，同时会进行多种边界检查；</p>
</li>
<li data-nodeid="439640">
<p data-nodeid="439641">解析 <code data-backticks="1" data-nodeid="439840">&lt;cache&gt;</code> 标签；</p>
</li>
<li data-nodeid="439642">
<p data-nodeid="439643">解析 <code data-backticks="1" data-nodeid="439843">&lt;cache-ref&gt;</code> 标签；</p>
</li>
<li data-nodeid="439644">
<p data-nodeid="439645">解析 <code data-backticks="1" data-nodeid="439846">&lt;resultMap&gt;</code> 标签；</p>
</li>
<li data-nodeid="439646">
<p data-nodeid="439647">解析 <code data-backticks="1" data-nodeid="439849">&lt;sql&gt;</code> 标签；</p>
</li>
<li data-nodeid="439648">
<p data-nodeid="439649">解析 <code data-backticks="1" data-nodeid="439852">&lt;select&gt;</code>、<code data-backticks="1" data-nodeid="439854">&lt;insert&gt;</code>、<code data-backticks="1" data-nodeid="439856">&lt;update&gt;</code>、<code data-backticks="1" data-nodeid="439858">&lt;delete&gt;</code> 等 SQL 标签。</p>
</li>
</ul>
<p data-nodeid="439650">下面我们就按照顺序逐一介绍这些方法的核心实现。</p>
<h4 data-nodeid="439651">1. 处理 <code data-backticks="1" data-nodeid="439864">&lt;cache&gt;</code> 标签</h4>
<p data-nodeid="439652">我们知道 Cache 接口及其实现是MyBatis 一级缓存和二级缓存的基础，其中，一级缓存是默认开启的，而二级缓存默认情况下并没有开启，如有需要，<strong data-nodeid="439873">可以通过<code data-backticks="1" data-nodeid="439869">&lt;cache&gt;</code>标签为指定的namespace 开启二级缓存</strong>。</p>
<p data-nodeid="439653">XMLMapperBuilder 中解析 <code data-backticks="1" data-nodeid="439875">&lt;cache&gt;</code> 标签的<strong data-nodeid="439881">核心逻辑位于 cacheElement() 方法</strong>之中，其具体步骤如下：</p>
<ul data-nodeid="439654">
<li data-nodeid="439655">
<p data-nodeid="439656">获取 <code data-backticks="1" data-nodeid="439883">&lt;cache&gt;</code> 标签中的各项属性（type、flushInterval、size 等属性）；</p>
</li>
<li data-nodeid="439657">
<p data-nodeid="439658">读取 <code data-backticks="1" data-nodeid="439886">&lt;cache&gt;</code> 标签下的子标签信息，这些信息将用于初始化二级缓存；</p>
</li>
<li data-nodeid="439659">
<p data-nodeid="439660">MapperBuilderAssistant 会根据上述配置信息，创建一个全新的Cache 对象并添加到 Configuration.caches 集合中保存。</p>
</li>
</ul>
<p data-nodeid="439661">也就是说，解析 <code data-backticks="1" data-nodeid="439890">&lt;cache&gt;</code> 标签得到的所有信息将会传给 MapperBuilderAssistant 完成 Cache 对象的创建，创建好的Cache 对象会添加到 Configuration.caches 集合中，<strong data-nodeid="439898">这个 caches 字段是一个StrictMap<code data-backticks="1" data-nodeid="439894">&lt;Cache&gt;</code> 类型的集合</strong>，其中的 Key是Cache 对象的唯一标识，默认值是Mapper.xml 映射文件的namespace，Value 才是真正的二级缓存对应的 Cache 对象。</p>
<p data-nodeid="439662">这里我们简单介绍一下 StrictMap的特性。</p>
<p data-nodeid="439663">StrictMap 继承了 HashMap，并且覆盖了 HashMap 的一些行为，例如，相较于 HashMap 的 put() 方法，StrictMap 的 put() 方法有如下几点不同：</p>
<ul data-nodeid="439664">
<li data-nodeid="439665">
<p data-nodeid="439666">如果检测到重复 Key 的写入，会直接抛出异常；</p>
</li>
<li data-nodeid="439667">
<p data-nodeid="439668">在没有重复 Key的情况下，会正常写入 KV 数据，与此同时，还会根据 Key产生一个 shortKey，shortKey 与完整 Key 指向同一个 Value 值；</p>
</li>
<li data-nodeid="439669">
<p data-nodeid="439670">如果 shortKey 已经存在，则将 value 修改成 Ambiguity 对象，Ambiguity 对象表示这个 shortKey 存在二义性，后续通过 StrictMap的get() 方法获取该 shortKey 的时候，会抛出异常。</p>
</li>
</ul>
<p data-nodeid="439671">了解了 StrictMap 这个集合类的特性之后，我们回到MapperBuilderAssistant 这个类继续分析，在它的 useNewCache() 方法中，会根据前面解析得到的配置信息，通过 CacheBuilder 创建 Cache 对象。</p>
<p data-nodeid="439672">通过名字你就能猜测到 CacheBuilder 是 Cache 的构造者，<strong data-nodeid="439910">CacheBuilder 中最核心的方法是build() 方法，其中会根据传入的配置信息创建底层存储数据的 Cache 对象以及相关的 Cache 装饰器</strong>，具体实现如下：</p>
<pre class="lang-java" data-nodeid="439673"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Cache <span class="hljs-title">build</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 将implementation默认值设置为PerpetualCache，在decorators集合中默认添加LruCache装饰器，</span>
&nbsp; &nbsp; <span class="hljs-comment">// 都是在setDefaultImplementations()方法中完成的</span>
&nbsp; &nbsp; setDefaultImplementations();
&nbsp; &nbsp; <span class="hljs-comment">// 通过反射，初始化implementation指定类型的对象</span>
&nbsp; &nbsp; Cache cache = newBaseCacheInstance(implementation, id);
&nbsp; &nbsp; <span class="hljs-comment">// 创建Cache关联的MetaObject对象，并根据properties设置Cache中的各个字段</span>
&nbsp; &nbsp; setCacheProperties(cache);
&nbsp; &nbsp; <span class="hljs-comment">// 根据上面创建的Cache对象类型，决定是否添加装饰器</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (PerpetualCache.class.equals(cache.getClass())) {
&nbsp; &nbsp; &nbsp; &nbsp; // 如果是PerpetualCache类型，则为其添加decorators集合中指定的装饰器
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Class&lt;? extends Cache&gt; decorator : decorators) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; // 通过反射创建Cache装饰器
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cache = newCacheDecoratorInstance(decorator, cache);
            // 依赖MetaObject将properties中配置信息设置到Cache的各个属性中，同时调用Cache的initialize()方法完成初始化
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; setCacheProperties(cache);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; // 根据readWrite、blocking、clearInterval等配置，
&nbsp; &nbsp; &nbsp; &nbsp; // 添加SerializedCache、ScheduledCache等装饰器
&nbsp; &nbsp; &nbsp; &nbsp; cache = setStandardDecorators(cache);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!LoggingCache.class.isAssignableFrom(cache.getClass())) {
&nbsp; &nbsp; &nbsp; &nbsp; // 如果不是PerpetualCache类型，就是其他自定义类型的Cache，则添加一个LoggingCache装饰器
&nbsp; &nbsp; &nbsp; &nbsp; cache = <span class="hljs-keyword">new</span> LoggingCache(cache);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> cache;
}
</code></pre>
<h4 data-nodeid="439674">2. 处理<code data-backticks="1" data-nodeid="439914">&lt;cache-ref&gt;</code>标签</h4>
<p data-nodeid="439675">通过上述介绍我们知道，可以通过 <code data-backticks="1" data-nodeid="439917">&lt;cache&gt;</code> 标签为每个 namespace 开启二级缓存，同时还会将 namespace 与关联的二级缓存 Cache对象记录到 Configuration.caches 集合中，也就是说二级缓存是 namespace 级别的。但是，在有的场景中，我们会需要在多个 namespace 共享同一个二级缓存，也就是<strong data-nodeid="439923">共享同一个 Cache 对象</strong>。</p>
<p data-nodeid="439676">为了解决这个需求，MyBatis提供了 <code data-backticks="1" data-nodeid="439925">&lt;cache-ref&gt; </code> 标签来引用另一个 namespace 的二级缓存。cacheRefElement() 方法是处理 <code data-backticks="1" data-nodeid="439927">&lt;cache-ref&gt;</code> 标签的核心逻辑所在，在 Configuration 中维护了一个 cacheRefMap 字段（HashMap&lt;String,String&gt; 类型），其中的 Key 是 <code data-backticks="1" data-nodeid="439931">&lt;cache-ref&gt;</code> 标签所属的namespace 标识，Value 值是 <code data-backticks="1" data-nodeid="439933">&lt;cache-ref&gt;</code> 标签引用的 namespace 值，这样的话，就可以将两个namespace 关联起来了，即这两个 namespace 共用一个 Cache对象。</p>
<p data-nodeid="439677">这里会使用到一个叫 CacheRefResolver 的 Cache 引用解析器。<strong data-nodeid="439944">CacheRefResolver 中记录了被引用的 namespace以及当前 namespace 关联的MapperBuilderAssistant 对象</strong>。前面在解析 <code data-backticks="1" data-nodeid="439940">&lt;cache&gt;</code>标签的时候我们介绍过，MapperBuilderAssistant 会在 useNewCache() 方法中通过 CacheBuilder 创建新的 Cache 对象，并记录到 currentCache 字段。而这里解析 <code data-backticks="1" data-nodeid="439942">&lt;cache-ref&gt;</code> 标签的时候，MapperBuilderAssistant 会通过 useCacheRef() 方法从 Configuration.caches 集合中，根据被引用的namespace 查找共享的 Cache 对象来初始化 currentCache，而不再创建新的Cache 对象，从而实现二级缓存的共享。</p>
<h4 data-nodeid="439678">3. 处理<code data-backticks="1" data-nodeid="439948">&lt;resultMap&gt;</code>标签</h4>
<p data-nodeid="439679">有关系型数据库使用经验的同学应该知道，select 语句执行得到的结果集实际上是一张二维表，而 Java 是一门面向对象的程序设计语言，在使用 JDBC 的时候，我们需要手动写代码将select 语句的结果集转换成 Java 对象，这是一项重复性很大的操作。</p>
<p data-nodeid="439680"><strong data-nodeid="439956">为了将 Java 开发者从这种重复性的工作中解脱出来，MyBatis 提供了 <code data-backticks="1" data-nodeid="439953">&lt;resultMap&gt;</code> 标签来定义结果集与 Java 对象之间的映射规则。</strong></p>
<p data-nodeid="439681">首先，<code data-backticks="1" data-nodeid="439958">&lt;resultMap&gt;</code> 标签下的每一个子标签，例如，<code data-backticks="1" data-nodeid="439960">&lt;column&gt;</code>、<code data-backticks="1" data-nodeid="439962">&lt;id&gt;</code> 等，都被解析一个 ResultMapping 对象，其中维护了数据库表中一个列与对应 Java 类中一个属性之间的映射关系。</p>
<p data-nodeid="439682">下面是 ResultMapping 中核心字段的含义。</p>
<ul data-nodeid="439683">
<li data-nodeid="439684">
<p data-nodeid="439685">column（String 类型）：当前标签中指定的 column 属性值，指向的是数据库表中的一个列名（或是别名）。</p>
</li>
<li data-nodeid="439686">
<p data-nodeid="439687">property（String 类型）：当前标签中指定的 property 属性值，指向的是与 column 列对应的属性名称。</p>
</li>
<li data-nodeid="439688">
<p data-nodeid="439689">javaType（Class&lt;?&gt;&nbsp;类型）、jdbcType（JdbcType 类型）：当前标签指定的 javaType 属性值和 jdbcType 属性值，指定了 property 字段的 Java 类型以及对应列的 JDBC 类型。</p>
</li>
<li data-nodeid="439690">
<p data-nodeid="439691">typeHandler（TypeHandler&lt;?&gt;&nbsp;类型）：当前标签的 typeHandler 属性值，这里指定的 TypeHandler 会覆盖默认的类型处理器。</p>
</li>
<li data-nodeid="439692">
<p data-nodeid="439693">nestedResultMapId（String类型）：当前标签的 resultMap 属性值，通过该属性我们可以引用另一个 <code data-backticks="1" data-nodeid="439974">&lt;resultMap&gt;</code> 标签的id，然后由这个被引用的<code data-backticks="1" data-nodeid="439976">&lt;resultMap&gt;</code> 标签映射结果集中的一部分列。这样，我们就可以将一个查询结果集映射成多个对象，同时确定这些对象之间的关联关系。</p>
</li>
<li data-nodeid="439694">
<p data-nodeid="439695">nestedQueryId（String&nbsp;类型）：当前标签的select 属性，我们可以通过该属性引用另一个 <code data-backticks="1" data-nodeid="439979">&lt;select&gt;</code> 标签中的select 语句定义，它会将当前列的值作为参数传入这个 select 语句。由于当前结果集可能查询出多行数据，那么可能就会导致 select 属性指定的 SQL 语句会执行多次，也就是著名的 N+1 问题。</p>
</li>
<li data-nodeid="439696">
<p data-nodeid="439697">columnPrefix（String 类型）：当前标签的 columnPrefix 属性值，记录了表中列名的公共前缀。</p>
</li>
<li data-nodeid="439698">
<p data-nodeid="439699">resultSet（String 类型）：当前标签的 resultSet 属性值。</p>
</li>
<li data-nodeid="439700">
<p data-nodeid="439701">lazy（boolean 类型）：当前标签的fetchType 属性，表示是否延迟加载当前标签对应的列。</p>
</li>
</ul>
<p data-nodeid="439702">介绍完 ResultMapping 对象（即<code data-backticks="1" data-nodeid="439985">&lt;resultMap&gt;</code> 标签下各个子标签的解析结果）之后，我们再来看<code data-backticks="1" data-nodeid="439987">&lt;resultMap&gt;</code> 标签如何被解析。整个 <code data-backticks="1" data-nodeid="439989">&lt;resultMap&gt;</code> 标签最终会被解析成 ResultMap 对象，它与 ResultMapping 之间的映射关系如下图所示：</p>
<p data-nodeid="439703"><img src="https://s0.lgstatic.com/i/image6/M00/0D/AC/CioPOWA7kqSASvnUAAPk5cQ7q3c025.png" alt="图片1.png" data-nodeid="439993"></p>
<div data-nodeid="439704"><p style="text-align:center">ResultMap 结构图</p></div>
<p data-nodeid="439705">通过上图我们可以看出，ResultMap 中有四个集合与 ResultMapping 紧密相连。</p>
<ul data-nodeid="439706">
<li data-nodeid="439707">
<p data-nodeid="439708">resultMappings 集合，维护了整个<code data-backticks="1" data-nodeid="439996">&lt;resultMap&gt;</code> 标签解析之后得到的全部映射关系，也就是全部 ResultMapping 对象。</p>
</li>
<li data-nodeid="439709">
<p data-nodeid="439710">idResultMappings 集合，维护了与唯一标识相关的映射，例如，<code data-backticks="1" data-nodeid="439999">&lt;id&gt;</code> 标签、<code data-backticks="1" data-nodeid="440001">&lt;constructor&gt;</code> 标签下的 <code data-backticks="1" data-nodeid="440003">&lt;idArg&gt;</code> 子标签解析得到的 ResultMapping 对象。如果没有定义 <code data-backticks="1" data-nodeid="440005">&lt;id&gt;</code> 等唯一性标签，则由 resultMappings 集合中全部映射关系来确定一条记录的唯一性，即 idResultMappings 集合与 resulMappings 集合相同。</p>
</li>
<li data-nodeid="439711">
<p data-nodeid="439712">constructorResultMappings 集合，维护了 <code data-backticks="1" data-nodeid="440008">&lt;constructor&gt;</code> 标签下全部子标签定义的映射关系。</p>
</li>
<li data-nodeid="439713">
<p data-nodeid="439714">propertyResultMappings 集合，维护了不带 Constructor 标志的映射关系。</p>
</li>
</ul>
<p data-nodeid="439715">除了上述四个 ResultMapping 集合，ResultMap 中还维护了下列核心字段。</p>
<ul data-nodeid="439716">
<li data-nodeid="439717">
<p data-nodeid="439718">id（String 类型）：当前 <code data-backticks="1" data-nodeid="440013">&lt;resultMap&gt;</code> 标签的 id 属性值。</p>
</li>
<li data-nodeid="439719">
<p data-nodeid="439720">type（Class 类型）：当前 <code data-backticks="1" data-nodeid="440016">&lt;resultMap&gt;</code> 的 type 属性值。</p>
</li>
<li data-nodeid="439721">
<p data-nodeid="439722">mappedColumns（Set<code data-backticks="1" data-nodeid="440019">&lt;String&gt;</code> 类型）：维护了所有映射关系中涉及的 column 属性值，也就是所有的列名（或别名）。</p>
</li>
<li data-nodeid="439723">
<p data-nodeid="439724">hasNestedResultMaps（boolean 类型）：当前 <code data-backticks="1" data-nodeid="440022">&lt;resultMap&gt;</code> 标签是否嵌套了其他 <code data-backticks="1" data-nodeid="440024">&lt;resultMap&gt;</code> 标签，即这个映射关系中指定了 resultMap属性，且未指定 resultSet 属性。</p>
</li>
<li data-nodeid="439725">
<p data-nodeid="439726">hasNestedQueries（boolean 类型）：当前 <code data-backticks="1" data-nodeid="440027">&lt;resultMap&gt;</code> 标签是否含有嵌套查询。也就是说，这个映射关系中是否指定了 select 属性。</p>
</li>
<li data-nodeid="439727">
<p data-nodeid="439728">autoMapping（Boolean 类型）：当前 ResultMap 是否开启自动映射的功能。</p>
</li>
<li data-nodeid="439729">
<p data-nodeid="439730">discriminator（Discriminator 类型）：对应 <code data-backticks="1" data-nodeid="440031">&lt;discriminator&gt;</code> 标签。</p>
</li>
</ul>
<p data-nodeid="439731">接下来我们开始深入分析 <code data-backticks="1" data-nodeid="440034">&lt;resultMap&gt;</code> 标签解析的流程。XMLMapperBuilder的resultMapElements() 方法负责解析 Mapper 配置文件中的全部 <code data-backticks="1" data-nodeid="440036">&lt;resultMap&gt;</code> 标签，其中会通过 resultMapElement() 方法解析单个 <code data-backticks="1" data-nodeid="440038">&lt;resultMap&gt;</code> 标签。</p>
<p data-nodeid="439732">下面是 resultMapElement() 方法解析 <code data-backticks="1" data-nodeid="440041">&lt;resultMap&gt; </code> 标签的核心流程。</p>
<ul data-nodeid="439733">
<li data-nodeid="439734">
<p data-nodeid="439735">获取 <code data-backticks="1" data-nodeid="440044">&lt;resultMap&gt;</code> 标签的type 属性值，这个值表示结果集将被映射成 type 指定类型的对象。如果没有指定 type 属性的话，会找其他属性值，优先级依次是：type、ofType、resultType、javaType。在这一步中会确定映射得到的对象类型，这里支持别名转换。</p>
</li>
<li data-nodeid="439736">
<p data-nodeid="439737">解析<code data-backticks="1" data-nodeid="440047">&lt;resultMap&gt;</code>标签下的各个子标签，每个子标签都会生成一个ResultMapping 对象，这个 ResultMapping 对象会被添加到resultMappings 集合（List<code data-backticks="1" data-nodeid="440049">&lt;ResultMapping&gt;</code> 类型）中暂存。这里会涉及 <code data-backticks="1" data-nodeid="440051">&lt;id&gt;</code>、<code data-backticks="1" data-nodeid="440053">&lt;result&gt;</code>、<code data-backticks="1" data-nodeid="440055">&lt;association&gt;</code>、<code data-backticks="1" data-nodeid="440057">&lt;collection&gt;</code>、<code data-backticks="1" data-nodeid="440059">&lt;discriminator&gt;</code> 等子标签的解析。</p>
</li>
<li data-nodeid="439738">
<p data-nodeid="439739">获取 <code data-backticks="1" data-nodeid="440062">&lt;resultMap&gt;</code> 标签的id 属性，默认值会拼装所有父标签的id、value 或 property 属性值。</p>
</li>
<li data-nodeid="439740">
<p data-nodeid="439741">获取 <code data-backticks="1" data-nodeid="440065">&lt;resultMap&gt;</code> 标签的extends、autoMapping 等属性。</p>
</li>
<li data-nodeid="439742">
<p data-nodeid="439743">创建 ResultMapResolver 对象，ResultMapResolver 会根据上面解析到的ResultMappings 集合以及 <code data-backticks="1" data-nodeid="440068">&lt;resultMap&gt;</code> 标签的属性构造 ResultMap 对象，并将其添加到 Configuration.resultMaps 集合（StrictMap 类型）中。</p>
</li>
</ul>
<h5 data-nodeid="439744">（1）解析 <code data-backticks="1" data-nodeid="440071">&lt;id&gt;</code>、<code data-backticks="1" data-nodeid="440073">&lt;result&gt;</code>、<code data-backticks="1" data-nodeid="440075">&lt;constructor&gt;</code>标签</h5>
<p data-nodeid="439745">在 resultMapElement() 方法中获取到 id 属性和 type 属性值之后，会调用 buildResultMappingFromContext() 方法解析上述标签得到 ResultMapping 对象，其核心逻辑如下：</p>
<ul data-nodeid="439746">
<li data-nodeid="439747">
<p data-nodeid="439748">获取当前标签的property的属性值作为目标属性名称（如果 <code data-backticks="1" data-nodeid="440079">&lt;constructor&gt;</code> 标签使用的是 name 属性）；</p>
</li>
<li data-nodeid="439749">
<p data-nodeid="439750">获取 column、javaType、typeHandler、jdbcType、select 等一系列属性，与获取 property 属性的方式类似；</p>
</li>
<li data-nodeid="439751">
<p data-nodeid="439752">根据上面解析到的信息，调用 MapperBuilderAssistant.buildResultMapping() 方法创建 ResultMapping 对象。</p>
</li>
</ul>
<p data-nodeid="439753">正如 resultMapElement() 方法核心步骤描述的那样，经过解析得到 ResultMapping 对象集合之后，会记录到resultMappings 这个临时集合中，然后由 ResultMapResolver 调用 MapperBuilderAssistant.addResultMap() 方法创建 ResultMap 对象，将resultMappings 集合中的全部 ResultMapping 对象添加到其中，然后将ResultMap 对象记录到 Configuration.resultMaps 集合中。</p>
<p data-nodeid="439754">下面是 MapperBuilderAssistant.addResultMap() 的具体实现：</p>
<pre class="lang-java" data-nodeid="439755"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> ResultMap <span class="hljs-title">addResultMap</span><span class="hljs-params">(
&nbsp; &nbsp; &nbsp; &nbsp; String id,
&nbsp; &nbsp; &nbsp; &nbsp; Class&lt;?&gt; type,
&nbsp; &nbsp; &nbsp; &nbsp; String extend,
&nbsp; &nbsp; &nbsp; &nbsp; Discriminator discriminator,
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ResultMapping&gt; resultMappings,
&nbsp; &nbsp; &nbsp; &nbsp; Boolean autoMapping)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// ResultMap的完整id是"namespace.id"的格式</span>
&nbsp; &nbsp; id = applyCurrentNamespace(id, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 获取被继承的ResultMap的完整id，也就是父ResultMap对象的完整id</span>
&nbsp; &nbsp; extend = applyCurrentNamespace(extend, <span class="hljs-keyword">true</span>);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (extend != <span class="hljs-keyword">null</span>) {&nbsp; <span class="hljs-comment">// 针对extend属性的处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 检测Configuration.resultMaps集合中是否存在被继承的ResultMap对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!configuration.hasResultMap(extend)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IncompleteElementException(<span class="hljs-string">"Could not find a parent resultmap with id '"</span> + extend + <span class="hljs-string">"'"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取需要被继承的ResultMap对象，也就是父ResultMap对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; ResultMap resultMap = configuration.getResultMap(extend);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取父ResultMap对象中记录的ResultMapping集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; List&lt;ResultMapping&gt; extendedResultMappings = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(resultMap.getResultMappings());
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 删除需要覆盖的ResultMapping集合</span>
&nbsp; &nbsp; &nbsp; &nbsp; extendedResultMappings.removeAll(resultMappings);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果当前&lt;resultMap&gt;标签中定义了&lt;constructor&gt;标签，则不需要使用父ResultMap中记录</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 的相应&lt;constructor&gt;标签，这里会将其对应的ResultMapping对象删除</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> declaresConstructor = <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (ResultMapping resultMapping : resultMappings) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; declaresConstructor = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (declaresConstructor) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; extendedResultMappings.removeIf(resultMapping -&gt; resultMapping.getFlags().contains(ResultFlag.CONSTRUCTOR));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 添加需要被继承下来的ResultMapping对象记录到resultMappings集合中</span>
&nbsp; &nbsp; &nbsp; &nbsp; resultMappings.addAll(extendedResultMappings);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 创建ResultMap对象，并添加到Configuration.resultMaps集合中保存</span>
&nbsp; &nbsp; ResultMap resultMap = <span class="hljs-keyword">new</span> ResultMap.Builder(configuration, id, type, resultMappings, autoMapping)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .discriminator(discriminator)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .build();
&nbsp; &nbsp; configuration.addResultMap(resultMap);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> resultMap;
}
</code></pre>
<p data-nodeid="439756">至于 <code data-backticks="1" data-nodeid="440086">&lt;constructor&gt; </code> 标签的流程，是由XMLMapperBuilder 中的processConstructorElement() 方法实现，其中会先获取 <code data-backticks="1" data-nodeid="440088">&lt;constructor&gt;</code> 标签的全部子标签，然后为每个标签添加 CONSTRUCTOR 标志（为每个<code data-backticks="1" data-nodeid="440090">&lt;idArg&gt;</code> 标签添加额外的ID标志），最后通过 buildResultMappingFromContext()方法创建 ResultMapping对象并记录到 resultMappings 集合中暂存，这些 ResultMapping 对象最终也会添加到前面介绍的ResultMap 对象。</p>
<h5 data-nodeid="439757">（2）解析 <code data-backticks="1" data-nodeid="440093">&lt;association&gt;</code> 和 <code data-backticks="1" data-nodeid="440095">&lt;collection&gt;</code>标签</h5>
<p data-nodeid="439758">接下来，我们来介绍解析 <code data-backticks="1" data-nodeid="440098">&lt;association&gt;</code> 和 <code data-backticks="1" data-nodeid="440100">&lt;collection&gt;</code>标签的核心流程，两者解析的过程基本一致。前面介绍的 buildResultMappingFromContext() 方法不仅完成了 <code data-backticks="1" data-nodeid="440102">&lt;id&gt;</code>、<code data-backticks="1" data-nodeid="440104">&lt;result&gt;</code> 等标签的解析，还完成了 <code data-backticks="1" data-nodeid="440106">&lt;association&gt;</code> 和 <code data-backticks="1" data-nodeid="440108">&lt;collection&gt;</code> 标签的解析，其中相关的代码片段如下：</p>
<pre class="lang-java" data-nodeid="439759"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> ResultMapping <span class="hljs-title">buildResultMappingFromContext</span><span class="hljs-params">(XNode context, Class&lt;?&gt; resultType, List&lt;ResultFlag&gt; flags)</span> </span>{
&nbsp; &nbsp; ... <span class="hljs-comment">// &lt;association&gt;标签中其他属性的解析与&lt;result&gt;、&lt;id&gt;标签类似，这里不再展开</span>
&nbsp; &nbsp; <span class="hljs-comment">// 如果&lt;association&gt;标签没有指定resultMap属性，那么就是匿名嵌套映射，需要通过</span>
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp; processNestedResultMappings()方法解析该匿名的嵌套映射</span>
&nbsp; &nbsp; String nestedResultMap = context.getStringAttribute(<span class="hljs-string">"resultMap"</span>, () -&gt;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processNestedResultMappings(context, Collections.emptyList(), resultType));
&nbsp; &nbsp; ... <span class="hljs-comment">// &lt;association&gt;标签中其他属性的解析与&lt;result&gt;、&lt;id&gt;标签类似，这里不再展开</span>
&nbsp; &nbsp; <span class="hljs-comment">// 根据上面解析到的属性值，创建ResultMapping对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> builderAssistant.buildResultMapping(resultType, property, column, javaTypeClass, jdbcTypeEnum, nestedSelect, nestedResultMap, notNullColumn, columnPrefix, typeHandlerClass, flags, resultSet, foreignColumn, lazy);
}
</code></pre>
<p data-nodeid="439760">这里的 processNestedResultMappings() 方法会递归执行resultMapElement() 方法解析 <code data-backticks="1" data-nodeid="440111">&lt;association&gt;</code> 标签和 <code data-backticks="1" data-nodeid="440113">&lt;collection&gt;</code> 标签指定的匿名嵌套映射，得到一个完整的ResultMap 对象，并添加到Configuration.resultMaps集合中。</p>
<h5 data-nodeid="439761">（3）解析 <code data-backticks="1" data-nodeid="440116">&lt;discriminator&gt;</code> 标签</h5>
<p data-nodeid="439762">最后一个要介绍的是 <code data-backticks="1" data-nodeid="440119">&lt;discriminator&gt;</code> 标签的解析过程，我们将 <code data-backticks="1" data-nodeid="440121">&lt;discriminator&gt;</code> 标签与 <code data-backticks="1" data-nodeid="440123">&lt;case&gt;</code> 标签配合使用，根据结果集中某列的值改变映射行为。从 resultMapElement() 方法的逻辑我们可以看出，<code data-backticks="1" data-nodeid="440125">&lt;discriminator&gt;</code> 标签是由 processDiscriminatorElement() 方法专门进行解析的，具体实现如下：</p>
<pre class="lang-java" data-nodeid="439763"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Discriminator <span class="hljs-title">processDiscriminatorElement</span><span class="hljs-params">(XNode context, Class&lt;?&gt; resultType, List&lt;ResultMapping&gt; resultMappings)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 从&lt;discriminator&gt;标签中解析column、javaType、jdbcType、typeHandler四个属性的逻辑非常简单，这里将这部分代码省略</span>
&nbsp; &nbsp; Map&lt;String, String&gt; discriminatorMap = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-comment">// 解析&lt;discriminator&gt;标签的&lt;case&gt;子标签</span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (XNode caseChild : context.getChildren()) {
&nbsp; &nbsp; &nbsp; &nbsp; String value = caseChild.getStringAttribute(<span class="hljs-string">"value"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 通过前面介绍的processNestedResultMappings()方法，解析&lt;case&gt;标签，</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建相应的嵌套ResultMap对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; String resultMap = caseChild.getStringAttribute(<span class="hljs-string">"resultMap"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; processNestedResultMappings(caseChild, resultMappings, resultType));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 记录该列值与对应选择的ResultMap的Id</span>
&nbsp; &nbsp; &nbsp; &nbsp; discriminatorMap.put(value, resultMap);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 创建Discriminator对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> builderAssistant.buildDiscriminator(resultType, column, javaTypeClass, jdbcTypeEnum, typeHandlerClass, discriminatorMap);
}
</code></pre>
<h3 data-nodeid="439764">SQL 语句解析全流程</h3>
<p data-nodeid="439765">在 Mapper.xml 映射文件中，除了上面介绍的标签之外，还有一类比较重要的标签，那就是 <code data-backticks="1" data-nodeid="440129">&lt;select&gt;</code>、<code data-backticks="1" data-nodeid="440131">&lt;insert&gt;</code>、<code data-backticks="1" data-nodeid="440133">&lt;delete&gt;</code>、<code data-backticks="1" data-nodeid="440135">&lt;update&gt;</code> 等 SQL 语句标签。虽然定义在 Mapper.xml 映射文件中，但是<strong data-nodeid="440141">这些标签是由 XMLStatementBuilder 进行解析的</strong>，而不再由 XMLMapperBuilder 来完成解析。</p>
<p data-nodeid="439766">在开始介绍 XMLStatementBuilder 解析 SQL 语句标签的具体实现之前，我们先来了解一下 MyBatis 在内存中是如何表示这些 SQL 语句标签的。在内存中，MyBatis 使用 SqlSource 接口来表示解析之后的 SQL 语句，其中的 SQL 语句只是一个中间态，可能包含动态 SQL 标签或占位符等信息，无法直接使用。SqlSource 接口的定义如下：</p>
<pre class="lang-java" data-nodeid="439767"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SqlSource</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 根据Mapper文件或注解描述的SQL语句，以及传入的实参，返回可执行的SQL</span>
&nbsp; &nbsp; <span class="hljs-function">BoundSql <span class="hljs-title">getBoundSql</span><span class="hljs-params">(Object parameterObject)</span></span>;
}
</code></pre>
<p data-nodeid="439768">MyBatis 在内存中使用 MappedStatement 对象表示上述 SQL 标签。在 MappedStatement 中的 sqlSource 字段记录了 SQL 标签中定义的 SQL 语句，sqlCommandType 字段记录了 SQL 语句的类型（INSERT、UPDATE、DELETE、SELECT 或 FLUSH 类型）。</p>
<p data-nodeid="439769">介绍完表示 SQL 标签的基础类之后，我们来分析 XMLStatementBuilder 解析 SQL 标签的入口方法—— parseStatementNode() 方法，在该方法中首先会根据 id 属性和 databaseId 属性决定加载匹配的 SQL 标签，然后解析其中的<code data-backticks="1" data-nodeid="440145">&lt;include&gt;</code> 标签和 <code data-backticks="1" data-nodeid="440147">&lt;selectKey&gt;</code> 标签，相关的代码片段如下：</p>
<pre class="lang-java" data-nodeid="439770"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">parseStatementNode</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL标签的id以及databaseId属性</span>
&nbsp; &nbsp; String id = context.getStringAttribute(<span class="hljs-string">"id"</span>);
&nbsp; &nbsp; String databaseId = context.getStringAttribute(<span class="hljs-string">"databaseId"</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 若databaseId属性值与当前使用的数据库不匹配，则不加载该SQL标签</span>
&nbsp; &nbsp; <span class="hljs-comment">// 若存在相同id且databaseId不为空的SQL标签，则不再加载该SQL标签</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!databaseIdMatchesCurrent(id, databaseId, <span class="hljs-keyword">this</span>.requiredDatabaseId)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 根据SQL标签的名称决定其SqlCommandType</span>
&nbsp; &nbsp; String nodeName = context.getNode().getNodeName();
&nbsp; &nbsp; SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL标签的属性值，例如，fetchSize、timeout、parameterType、parameterMap、</span>
&nbsp; &nbsp; <span class="hljs-comment">// resultMap、resultType、lang、resultSetType、flushCache、useCache等。</span>
&nbsp; &nbsp; <span class="hljs-comment">// 这些属性的具体含义在MyBatis官方文档中已经有比较详细的介绍了，这里不再赘述</span>
&nbsp; &nbsp; ... ...
&nbsp; &nbsp; <span class="hljs-comment">// 在解析SQL语句之前，先处理其中的&lt;include&gt;标签</span>
&nbsp; &nbsp; XMLIncludeTransformer includeParser = <span class="hljs-keyword">new</span> XMLIncludeTransformer(configuration, builderAssistant);
&nbsp; &nbsp; includeParser.applyIncludes(context.getNode());
&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL标签的parameterType、lang两个属性</span>
&nbsp; &nbsp; ... ...
&nbsp; &nbsp; <span class="hljs-comment">// 解析&lt;selectKey&gt;标签</span>
&nbsp; &nbsp; processSelectKeyNodes(id, parameterTypeClass, langDriver);

&nbsp; &nbsp; <span class="hljs-comment">// 暂时省略后面的逻辑</span>
&nbsp; &nbsp; ...
}
</code></pre>
<h4 data-nodeid="439771">1. 处理 <code data-backticks="1" data-nodeid="440152">&lt;include&gt;</code> 标签</h4>
<p data-nodeid="439772">在实际应用中，我们会在<code data-backticks="1" data-nodeid="440155">&lt;sql&gt;</code> 标签中定义一些能够被重用的SQL 片段，在 XMLMapperBuilder.sqlElement() 方法中会根据当前使用的 DatabaseId 匹配 <code data-backticks="1" data-nodeid="440157">&lt;sql&gt;</code> 标签，只有匹配的 SQL 片段才会被加载到内存。</p>
<p data-nodeid="439773">在解析 SQL 标签之前，MyBatis 会先将 <code data-backticks="1" data-nodeid="440160">&lt;include&gt;</code> 标签转换成对应的 SQL 片段（即定义在 <code data-backticks="1" data-nodeid="440162">&lt;sql&gt;</code> 标签内的文本），这个转换过程是在 XMLIncludeTransformer.applyIncludes() 方法中实现的（其中不仅包含了 <code data-backticks="1" data-nodeid="440164">&lt;include&gt;</code> 标签的处理，还包含了“${}”占位符的处理）。</p>
<p data-nodeid="439774">针对 <code data-backticks="1" data-nodeid="440167">&lt;include&gt;</code> 标签的处理如下：</p>
<ul data-nodeid="439775">
<li data-nodeid="439776">
<p data-nodeid="439777">查找 refid 属性指向的 <code data-backticks="1" data-nodeid="440170">&lt;sql&gt;</code> 标签，得到其对应的 Node 对象；</p>
</li>
<li data-nodeid="439778">
<p data-nodeid="439779">解析 <code data-backticks="1" data-nodeid="440173">&lt;include&gt;</code> 标签下的 <code data-backticks="1" data-nodeid="440175">&lt;property&gt;</code> 标签，将得到的键值对添加到 variablesContext 集合（Properties 类型）中，并形成新的 Properties 对象返回，用于替换占位符；</p>
</li>
<li data-nodeid="439780">
<p data-nodeid="439781">递归执行 applyIncludes()方法，因为在 <code data-backticks="1" data-nodeid="440178">&lt;sql&gt;</code> 标签的定义中可能会使用 <code data-backticks="1" data-nodeid="440180">&lt;include&gt;</code> 引用其他 SQL 片段，在 applyIncludes()方法递归的过程中，如果遇到“${}”占位符，则使用 variablesContext 集合中的键值对进行替换；</p>
</li>
<li data-nodeid="439782">
<p data-nodeid="439783">最后，将 <code data-backticks="1" data-nodeid="440183">&lt;include&gt;</code> 标签替换成 <code data-backticks="1" data-nodeid="440185">&lt;sql&gt;</code> 标签的内容。</p>
</li>
</ul>
<p data-nodeid="439784">通过上面逻辑可以看出，<code data-backticks="1" data-nodeid="440188">&lt;include&gt;</code> 标签和 <code data-backticks="1" data-nodeid="440190">&lt;sql&gt;</code> 标签是可以嵌套多层的，此时就会涉及 applyIncludes()方法的递归，同时可以配合“${}”占位符，实现 SQL 片段模板化，更大程度地提高 SQL 片段的重用率。</p>
<h4 data-nodeid="439785">2. 处理 <code data-backticks="1" data-nodeid="440195">&lt;selectKey&gt;</code> 标签</h4>
<p data-nodeid="439786">在有的数据库表设计场景中，我们会添加一个自增 ID 字段作为主键，例如，用户 ID、订单 ID 或者这个自增 ID 本身并没有什么业务含义，只是一个唯一标识而已。在某些业务逻辑里面，我们希望在执行 insert 语句的时候返回这个自增 ID 值，<code data-backticks="1" data-nodeid="440198">&lt;selectKey&gt;</code> 标签就可以实现自增 ID 的获取。<code data-backticks="1" data-nodeid="440200">&lt;selectKey&gt;</code> 标签不仅可以获取自增 ID，还可以指定其他 SQL 语句，从其他表或执行数据库的函数获取字段值。</p>
<p data-nodeid="439787"><strong data-nodeid="440210">parseSelectKeyNode() 方法是解析 <code data-backticks="1" data-nodeid="440204">&lt;selectKey&gt;</code> 标签的核心所在</strong>，其中会解析 <code data-backticks="1" data-nodeid="440208">&lt;selectKey&gt;</code> 标签的各个属性，并根据这些属性值将其中的 SQL 语句解析成 MappedStatement 对象，具体实现如下：</p>
<pre class="lang-java" data-nodeid="439788"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">parseSelectKeyNode</span><span class="hljs-params">(String id, XNode nodeToHandle, Class&lt;?&gt; parameterTypeClass, LanguageDriver langDriver, String databaseId)</span> </span>{
&nbsp; &nbsp; ... <span class="hljs-comment">// 解析&lt;selectKey&gt;标签的resultType、statementType、keyProperty等属性</span>

&nbsp; &nbsp; <span class="hljs-comment">// 通过LanguageDriver解析&lt;selectKey&gt;标签中的SQL语句，得到对应的SqlSource对象</span>
&nbsp; &nbsp; SqlSource sqlSource = langDriver.createSqlSource(configuration, nodeToHandle, parameterTypeClass);
&nbsp; &nbsp; SqlCommandType sqlCommandType = SqlCommandType.SELECT;
&nbsp; &nbsp; <span class="hljs-comment">// 创建MappedStatement对象</span>
&nbsp; &nbsp; builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; resultSetTypeEnum, flushCache, useCache, resultOrdered,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; keyGenerator, keyProperty, keyColumn, databaseId, langDriver, <span class="hljs-keyword">null</span>);
&nbsp; &nbsp; id = builderAssistant.applyCurrentNamespace(id, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 创建&lt;selectKey&gt;标签对应的KeyGenerator对象，这个KeyGenerator对象会添加到Configuration.keyGenerators集合中</span>
&nbsp; &nbsp; MappedStatement keyStatement = configuration.getMappedStatement(id, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; configuration.addKeyGenerator(id, <span class="hljs-keyword">new</span> SelectKeyGenerator(keyStatement, executeBefore));
}
</code></pre>
<h4 data-nodeid="439789">3. 处理 SQL 语句</h4>
<p data-nodeid="439790">经过 <code data-backticks="1" data-nodeid="440215">&lt;include&gt;</code> 标签和 <code data-backticks="1" data-nodeid="440217">&lt;selectKey&gt;</code> 标签的处理流程之后，XMLStatementBuilder 中的 parseStatementNode()方法接下来就要开始处理 SQL 语句了，相关的代码片段之前被省略了，这里我们详细分析一下：</p>
<pre class="lang-java" data-nodeid="439791"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">parseStatementNode</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 前面是解析&lt;selectKey&gt;和&lt;include&gt;标签的逻辑，这里不再展示</span>
&nbsp; &nbsp; <span class="hljs-comment">// 当执行到这里的时候，&lt;selectKey&gt;和&lt;include&gt;标签已经被解析完毕，并删除掉了</span>
&nbsp; &nbsp; <span class="hljs-comment">// 下面是解析SQL语句的逻辑，也是parseStatementNode()方法的核心</span>
&nbsp; &nbsp; <span class="hljs-comment">// 通过LanguageDriver.createSqlSource()方法创建SqlSource对象</span>
&nbsp; &nbsp; SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);

&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL标签中配置的resultSets、keyProperty、keyColumn等属性，以及前面解析&lt;selectKey&gt;标签得到的KeyGenerator对象等，</span>
&nbsp; &nbsp; <span class="hljs-comment">// 这些信息将会填充到MappedStatement对象中</span>

&nbsp; &nbsp; <span class="hljs-comment">// 根据上述属性信息创建MappedStatement对象，并添加到Configuration.mappedStatements集合中保存</span>
&nbsp; &nbsp; builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; resultSetTypeEnum, flushCache, useCache, resultOrdered,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
}
</code></pre>
<p data-nodeid="439792">这里解析 SQL 语句<strong data-nodeid="440224">使用的是 LanguageDriver 接口</strong>，其核心实现是 XMLLanguageDriver，继承关系如下图所示：</p>
<p data-nodeid="439793"><img src="https://s0.lgstatic.com/i/image6/M00/0D/B0/Cgp9HWA7ksyAUvrwAADwoAT3J5M370.png" alt="图片2.png" data-nodeid="440227"></p>
<div data-nodeid="439794"><p style="text-align:center">LanguageDriver 继承关系图</p></div>
<p data-nodeid="439795">在 createSqlSource() 方法中，XMLLanguageDriver 会依赖 XMLScriptBuilder 创建 SqlSource 对象，XMLScriptBuilder 首先会判断 SQL 语句是否为动态SQL，判断的核心逻辑在 parseDynamicTags()方法中，核心实现如下：</p>
<pre class="lang-java" data-nodeid="439796"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> MixedSqlNode <span class="hljs-title">parseDynamicTags</span><span class="hljs-params">(XNode node)</span> </span>{
&nbsp; &nbsp; List&lt;SqlNode&gt; contents = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(); <span class="hljs-comment">// 解析后的SqlNode结果集合</span>
&nbsp; &nbsp; NodeList children = node.getNode().getChildNodes();
&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL标签下的所有节点，包括标签节点和文本节点</span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; children.getLength(); i++) {
&nbsp; &nbsp; &nbsp; &nbsp; XNode child = node.newXNode(children.item(i));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (child.getNode().getNodeType() == Node.CDATA_SECTION_NODE ||
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; child.getNode().getNodeType() == Node.TEXT_NODE) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 处理文本节点，也就是SQL语句</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String data = child.getStringBody(<span class="hljs-string">""</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TextSqlNode textSqlNode = <span class="hljs-keyword">new</span> TextSqlNode(data);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 解析SQL语句，如果含有未解析的"${}"占位符，则为动态SQL</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (textSqlNode.isDynamic()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; contents.add(textSqlNode);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; isDynamic = <span class="hljs-keyword">true</span>; <span class="hljs-comment">// 标记为动态SQL语句</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; contents.add(<span class="hljs-keyword">new</span> StaticTextSqlNode(data));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (child.getNode().getNodeType() == Node.ELEMENT_NODE) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果解析到一个子标签，那么一定是动态SQL</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 这里会根据不同的标签，获取不同的NodeHandler，然后由NodeHandler进行后续解析</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String nodeName = child.getNode().getNodeName();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; NodeHandler handler = nodeHandlerMap.get(nodeName);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BuilderException(<span class="hljs-string">"Unknown element &lt;"</span> + nodeName + <span class="hljs-string">"&gt; in SQL statement."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 处理动态SQL语句，并将解析得到的SqlNode对象记录到contents集合中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; handler.handleNode(child, contents);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; isDynamic = <span class="hljs-keyword">true</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
    <span class="hljs-comment">// 解析后的SqlNode集合将会被封装成MixedSqlNode返回</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> MixedSqlNode(contents);
}
</code></pre>
<p data-nodeid="439797">这里使用 SqlNode 接口来表示一条 SQL 语句的不同部分，其中，TextSqlNode 表示的是SQL 语句的文本（可能包含“${}”占位符），StaticTextSqlNode 表示的是不包含占位符的SQL 语句文本。</p>
<p data-nodeid="439798">另外一个新接口是NodeHandler，它有很多实现类，如下图所示：</p>
<p data-nodeid="439799"><img src="https://s0.lgstatic.com/i/image6/M00/0D/B0/Cgp9HWA7kvSAHP1yAAEyhRwHGEE543.png" alt="图片3.png" data-nodeid="440233"></p>
<div data-nodeid="439800"><p style="text-align:center">NodeHandler 继承关系图</p></div>
<p data-nodeid="439801"><strong data-nodeid="440240">NodeHandler接口负责解析动态 SQL 内的标签</strong>，生成相应的 SqlNode 对象，通过 NodeHandler 实现类的名称，我们就可以大概猜测到其解析的标签名称。以 IfHandler 为例，它解析的就是 <code data-backticks="1" data-nodeid="440238">&lt;if&gt;</code> 标签，其核心实现如下：</p>
<pre class="lang-java" data-nodeid="439802"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">IfHandler</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">NodeHandler</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleNode</span><span class="hljs-params">(XNode nodeToHandle, List&lt;SqlNode&gt; targetContents)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 通过parseDynamicTags()方法，解析&lt;if&gt;标签下嵌套的动态SQL</span>
&nbsp; &nbsp; &nbsp; &nbsp; MixedSqlNode mixedSqlNode = parseDynamicTags(nodeToHandle);
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 获取&lt;if&gt;标签判断分支的条件</span>
&nbsp; &nbsp; &nbsp; &nbsp; String test = nodeToHandle.getStringAttribute(<span class="hljs-string">"test"</span>);
        <span class="hljs-comment">// 创建IfNode对象(也是SqlNode接口的实现)，并将其保存下来</span>
&nbsp; &nbsp; &nbsp; &nbsp; IfSqlNode ifSqlNode = <span class="hljs-keyword">new</span> IfSqlNode(mixedSqlNode, test);
&nbsp; &nbsp; &nbsp; &nbsp; targetContents.add(ifSqlNode);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="439803">完成了对 SQL 语句的解析，得到了相应的 MixedSqlNode对象之后，XMLScriptBuilder 会根据 SQL 语句的类型生成不同的 SqlSource 实现：</p>
<pre class="lang-java" data-nodeid="439804"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> SqlSource <span class="hljs-title">parseScriptNode</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 对SQL语句进行解析</span>
&nbsp; &nbsp; MixedSqlNode rootSqlNode = parseDynamicTags(context);
&nbsp; &nbsp; SqlSource sqlSource;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isDynamic) { <span class="hljs-comment">// 根据该SQL是否为动态SQL，创建不同的SqlSource实现</span>
&nbsp; &nbsp; &nbsp; &nbsp; sqlSource = <span class="hljs-keyword">new</span> DynamicSqlSource(configuration, rootSqlNode);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; sqlSource = <span class="hljs-keyword">new</span> RawSqlSource(configuration, rootSqlNode, parameterType);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> sqlSource;
}
</code></pre>
<h3 data-nodeid="439805">总结</h3>
<p data-nodeid="439806">这一讲我们重点介绍了 MyBatis 在初始化过程中对 Mapper.xml 映射文件的解析。</p>
<p data-nodeid="439807">首先，我们着重介绍了 Mapper.xml 映射文件中对 <code data-backticks="1" data-nodeid="440245">&lt;cache&gt;</code> 标签、<code data-backticks="1" data-nodeid="440247">&lt;cache-ref&gt;</code> 标签以及 <code data-backticks="1" data-nodeid="440249">&lt;resultMap&gt;</code> 标签（包括它的各个子标签）的解析流程，让我们知道 MyBatis是如何正确理解二级缓存的配置信息以及我们定义的各种映射规则。</p>
<p data-nodeid="439808">然后，我们详细分析了 MyBatis 对 Mapper.xml 映射文件中 SQL 语句标签的解析，其中涉及 <code data-backticks="1" data-nodeid="440252">&lt;include&gt;</code>、<code data-backticks="1" data-nodeid="440254">&lt;selectKey&gt;</code> 等标签的处理逻辑。</p>
<p data-nodeid="442169">在解析 SQL 语句的过程中，涉及了动态 SQL 语句的解析，不过这一讲只是让你找到了这一逻辑的入口，在下一讲，我们就会深入讲解 MyBatis 动态 SQL 的设计思想和解析流程，记得按时来听课。</p>
<hr data-nodeid="442170">
<p data-nodeid="442171"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="442179"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="442178"></a></p>
<p data-nodeid="442172"><strong data-nodeid="442183">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="442173" class="te-preview-highlight">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="442187">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **红：
> 难度越来越大了 加油冲

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一定坚持打卡哦！

##### *鑫：
> 属实有点硬核啊

