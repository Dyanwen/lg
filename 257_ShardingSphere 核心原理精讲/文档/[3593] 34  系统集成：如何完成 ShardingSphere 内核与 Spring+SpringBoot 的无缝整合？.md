<p data-nodeid="2045" class="">今天，我们将进入整个课程中最后一个模块——系统集成模块的介绍。这里所谓的系统集成，指的就是 ShardingSphere 和 Spring 框架的集成。</p>
<p data-nodeid="2046">到目前为止，ShardingSphere 实现了<strong data-nodeid="2122">两种系统集成机制</strong>：一种是命名空间（namespace）机制，即通过扩展 Spring Schema 来实现与 Spring 框架的集成；而另一种则是通过编写自定义的 starter 组件来完成与 Spring Boot 的集成。本课时我将分别讲解这两种系统集成机制。</p>
<p data-nodeid="2047">基于系统集成模块，无论开发人员采用哪一种 Spring 框架，对于使用 ShardingSphere 而言都是零学习成本。</p>
<h3 data-nodeid="2048">基于命名空间集成 Spring</h3>
<p data-nodeid="2049">从扩展性的角度讲，基于 XML Schema 的扩展机制也是非常常见和实用的一种方法。在 Spring 中，允许我们自己定义 XML 的结构，并且可以用自己的 Bean 解析器进行解析。通过对 Spring Schema 的扩展，ShardingSphere 可以完成与 Spring 框架的有效集成。</p>
<h4 data-nodeid="2050">1.基于命名空间集成 Spring 的通用开发流程</h4>
<p data-nodeid="2051">基于命名空间机制实现与 Spring 的整合，开发上通常采用的是固定的一个流程，包括如下所示的五大步骤：</p>
<p data-nodeid="2052"><img src="https://s0.lgstatic.com/i/image/M00/5A/27/Ciqc1F90KhSANChgAACMwNPdi-8838.png" alt="image (10).png" data-nodeid="2130"></p>
<p data-nodeid="2053">这些步骤包括：编写业务对象、编写 XSD 文件、编写 BeanDefinitionParser 实现类、编写 NamespaceHandler 实现类，以及编写 spring.handlers 和 spring.schemas 配置文件，我们来看看 ShardingSphere 中实现这些步骤的具体做法。</p>
<h4 data-nodeid="2054">2.ShardingSphere 集成 Spring</h4>
<p data-nodeid="2055">ShardingSphere 中存在两个以“spring-namespace”结尾的代码工程，即 sharding-jdbc-spring-namespace 和 sharding-jdbc-orchestration-spring-namespace，显然后者关注的是编排治理相关功能的集成，相对比较简单。再因为命名空间机制的实现过程也基本一致，因此，我们以 sharding-jdbc-spring-namespace 工程为例展开讨论。</p>
<p data-nodeid="2056">而在 sharding-jdbc-spring-namespace 工程中，又包含了对普通分片、读写分离和数据脱敏这三块核心功能的集成内容，它们的实现也都是采用了类似的方式，因此我们也不会重复进行说明，这里就以普通分片为例进行介绍。</p>
<p data-nodeid="2057">首先，我们发现了一个专门用于与 Spring 进行集成的 SpringShardingDataSource 类，这个类就是业务对象类，如下所示：</p>
<pre class="lang-java" data-nodeid="2058"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringShardingDataSource</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ShardingDataSource</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SpringShardingDataSource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Map&lt;String, DataSource&gt; dataSourceMap, <span class="hljs-keyword">final</span> ShardingRuleConfiguration shardingRuleConfiguration, <span class="hljs-keyword">final</span> Properties props)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>(dataSourceMap, <span class="hljs-keyword">new</span> ShardingRule(shardingRuleConfiguration, dataSourceMap.keySet()), props);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2059">可以看到这个 SpringShardingDataSource 类实际上只是对 ShardingDataSource 的一种简单封装，没有包含任何实际操作。</p>
<p data-nodeid="2060">然后，我们来看配置项标签的定义类，这种类是一种简单的工具类，其作用就是定义标签的名称。在命名上，ShardingSphere 中的这些类都以“BeanDefinitionParserTag”结尾，例如如下所示的 ShardingDataSourceBeanDefinitionParserTag：</p>
<pre class="lang-xml" data-nodeid="2061"><code data-language="xml">public final class ShardingDataSourceBeanDefinitionParserTag {
&nbsp;&nbsp;&nbsp; public static final String ROOT_TAG = "data-source";
&nbsp;&nbsp;&nbsp; public static final String SHARDING_RULE_CONFIG_TAG = sharding-rule";
&nbsp;&nbsp;&nbsp; public static final String PROPS_TAG = "props";
	public static final String DATA_SOURCE_NAMES_TAG = "data-source-names";
	public static final String DEFAULT_DATA_SOURCE_NAME_TAG = "default-data-source-name";
&nbsp;&nbsp;&nbsp; public static final String TABLE_RULES_TAG = "table-rules"; 
&nbsp;&nbsp;&nbsp; …
}
</code></pre>
<p data-nodeid="2062">这里定义了一批 Tag 和一批 Attribute，我们不做 一 一 展开。可以对照如下所示的基于 XML 的配置示例来对这些定义的配置项进行理解：</p>
<pre class="lang-xml" data-nodeid="2063"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">sharding:data-source</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"shardingDataSource"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:sharding-rule</span> <span class="hljs-attr">data-source-names</span>=<span class="hljs-string">"ds0,ds1"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:table-rules</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:table-rule</span> …/&gt;</span>
	            <span class="hljs-tag">&lt;<span class="hljs-name">sharding:table-rule</span> …/&gt;</span>
	            …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">sharding:table-rules</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">sharding:sharding-rule</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">sharding:data-source</span>&gt;</span>
</code></pre>
<p data-nodeid="2064">然后，我们在 sharding-jdbc-spring-namespace 代码工程的 META-INF/namespace 文件夹下找到了对应的 sharding.xsd 文件，其基本结构如下所示：</p>
<pre class="lang-xml" data-nodeid="2065"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">xsd:schema</span> <span class="hljs-attr">xmlns</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/sharding"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">xmlns:xsd</span>=<span class="hljs-string">"http://www.w3.org/2001/XMLSchema"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">xmlns:beans</span>=<span class="hljs-string">"http://www.springframework.org/schema/beans"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">xmlns:encrypt</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/encrypt"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">targetNamespace</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/sharding"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">elementFormDefault</span>=<span class="hljs-string">"qualified"</span> <span class="hljs-attr">xmlns:xsi</span>=<span class="hljs-string">"http://www.w3.org/2001/XMLSchema-instance"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-attr">xsi:schemaLocation</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/encrypt http://shardingsphere.apache.org/schema/shardingsphere/encrypt/encrypt.xsd"</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:import</span> <span class="hljs-attr">namespace</span>=<span class="hljs-string">"http://www.springframework.org/schema/beans"</span> <span class="hljs-attr">schemaLocation</span>=<span class="hljs-string">"http://www.springframework.org/schema/beans/spring-beans.xsd"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:import</span> <span class="hljs-attr">namespace</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/encrypt"</span> <span class="hljs-attr">schemaLocation</span>=<span class="hljs-string">"http://shardingsphere.apache.org/schema/shardingsphere/encrypt/encrypt.xsd"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:element</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"data-source"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:complexType</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:all</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:element</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"sharding-rule"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:element</span> <span class="hljs-attr">ref</span>=<span class="hljs-string">"props"</span> <span class="hljs-attr">minOccurs</span>=<span class="hljs-string">"0"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">xsd:all</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">xsd:attribute</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"id"</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"xsd:string"</span> <span class="hljs-attr">use</span>=<span class="hljs-string">"required"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">xsd:complexType</span>&gt;</span>
	<span class="hljs-tag">&lt;/<span class="hljs-name">xsd:element</span>&gt;</span>
	…
<span class="hljs-tag">&lt;/<span class="hljs-name">xsd:schema</span>&gt;</span>
</code></pre>
<p data-nodeid="2066">可以看到对于“data-source”这个 element 而言，包含了“sharding-rule”和“props”这两个子 element，其中“props”不是必需的。同时，“data-source”还可以包含一个“id”属性，而这个属性则是必填的，我们在前面的配置示例中已经看到了这一点。而对于“sharding-rule”而言，则可以有很多内嵌的属性，sharding.xsd 文件中对这些属性都做了定义。</p>
<p data-nodeid="2067">同时，我们应该注意到的是，sharding.xsd 中通过使用 xsd:import 标签还引入了两个 namespace，一个是 Spring 中的<a href="http://www.springframework.org/schema/beans" data-nodeid="2144">http://www.springframework.org/schema/beans</a>，另一个则是 ShardingSphere 自身的http://shardingsphere.apache.org/schema/shardingsphere/encrypt，这个命名空间的定义位于与 <a href="http://shardingsphere.apache.org/schema/shardingsphere/encrypt" data-nodeid="2148">sharding.xsd 同目录下的 encrypt.xsd</a><a href="http://shardingsphere.apache.org/schema/shardingsphere/encrypt" data-nodeid="2151">文件中</a>。</p>
<p data-nodeid="2068">有了业务对象类，以及 XSD 文件的定义，接下来我们就来看看 NamespaceHandler 实现类 ShardingNamespaceHandler，如下所示：</p>
<pre class="lang-java" data-nodeid="2069"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingNamespaceHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">NamespaceHandlerSupport</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; registerBeanDefinitionParser(ShardingDataSourceBeanDefinitionParserTag.ROOT_TAG, <span class="hljs-keyword">new</span> ShardingDataSourceBeanDefinitionParser());
&nbsp;
	registerBeanDefinitionParser(ShardingStrategyBeanDefinitionParserTag.STANDARD_STRATEGY_ROOT_TAG, <span class="hljs-keyword">new</span> ShardingStrategyBeanDefinitionParser());
&nbsp;
	…
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2070">可以看到这里也是直接使用了 registerBeanDefinitionParser 方法来完成标签项与具体的 BeanDefinitionParser 类之间的对应关系。我们来看这里的 ShardingDataSourceBeanDefinitionParser，其核心的 parseInternal 方法如下所示：</p>
<pre class="lang-java" data-nodeid="2071"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> AbstractBeanDefinition <span class="hljs-title">parseInternal</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Element element, <span class="hljs-keyword">final</span> ParserContext parserContext)</span> </span>{
	&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//构建针对 SpringShardingDataSource 的 BeanDefinitionBuilder</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BeanDefinitionBuilder factory = BeanDefinitionBuilder.rootBeanDefinition(SpringShardingDataSource.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//解析构造函数中的 DataSource 参数</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; factory.addConstructorArgValue(parseDataSources(element));
<span class="hljs-comment">//解析构造函数中 ShardingRuleConfiguration 参数&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; factory.addConstructorArgValue(parseShardingRuleConfiguration(element));</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//解析构造函数中 Properties 参数</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; factory.addConstructorArgValue(parseProperties(element, parserContext));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; factory.setDestroyMethodName(<span class="hljs-string">"close"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> factory.getBeanDefinition();
}
</code></pre>
<p data-nodeid="2072">这里，我们自己定义了一个 BeanDefinitionBuilder 并将其绑定到前面定义的业务对象类 SpringShardingDataSource。然后，我们通过三个 addConstructorArgValue 方法的调用，分别为 SpringShardingDataSource 构造函数中所需的 dataSourceMap、shardingRuleConfiguration 以及 props 参数进行赋值。</p>
<p data-nodeid="2073">我们再来进一步看一下上述方法中的 parseDataSources 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="2074"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Map&lt;String, RuntimeBeanReference&gt; <span class="hljs-title">parseDataSources</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Element element)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Element shardingRuleElement = DomUtils.getChildElementByTagName(element, ShardingDataSourceBeanDefinitionParserTag.SHARDING_RULE_CONFIG_TAG);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;String&gt; dataSources = Splitter.on(<span class="hljs-string">","</span>).trimResults().splitToList(shardingRuleElement.getAttribute(ShardingDataSourceBeanDefinitionParserTag.DATA_SOURCE_NAMES_TAG));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, RuntimeBeanReference&gt; result = <span class="hljs-keyword">new</span> ManagedMap&lt;&gt;(dataSources.size());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (String each : dataSources) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(each, <span class="hljs-keyword">new</span> RuntimeBeanReference(each));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
}
</code></pre>
<p data-nodeid="2075">基于前面介绍的配置示例，我们理解这段代码的作用是获取所配置的“ds0,ds1”字符串，并对其进行拆分，然后基于每个代表具体 DataSource 的名称构建 RuntimeBeanReference 对象并进行返回，这样就可以把在 Spring 容器中定义的其他 Bean 加载到 BeanDefinitionBuilder 中。</p>
<p data-nodeid="2076">关于 ShardingDataSourceBeanDefinitionParser 中其他 parse 方法的使用，大家可以通过阅读对应的代码进行理解，处理方式都是非常类似的，就不再重复展开。</p>
<p data-nodeid="2077">最后，我们需要在 META-INF 目录下提供spring.schemas 文件，如下所示：</p>
<pre class="lang-xml" data-nodeid="2078"><code data-language="xml">http\://shardingsphere.apache.org/schema/shardingsphere/sharding/sharding.xsd=META-INF/namespace/sharding.xsd
http\://shardingsphere.apache.org/schema/shardingsphere/masterslave/master-slave.xsd=META-INF/namespace/master-slave.xsd
http\://shardingsphere.apache.org/schema/shardingsphere/encrypt/encrypt.xsd=META-INF/namespace/encrypt.xsd
</code></pre>
<p data-nodeid="2079">同样，spring.handlers 的内容如下所示：</p>
<pre class="lang-xml" data-nodeid="2080"><code data-language="xml">http\://shardingsphere.apache.org/schema/shardingsphere/sharding=org.apache.shardingsphere.shardingjdbc.spring.namespace.handler.ShardingNamespaceHandler
http\://shardingsphere.apache.org/schema/shardingsphere/masterslave=org.apache.shardingsphere.shardingjdbc.spring.namespace.handler.MasterSlaveNamespaceHandler
http\://shardingsphere.apache.org/schema/shardingsphere/encrypt=org.apache.shardingsphere.shardingjdbc.spring.namespace.handler.EncryptNamespaceHandler
</code></pre>
<p data-nodeid="2081">至此，我们对 ShardingSphere 中基于命名空间机制与 Spring 进行系统集成的实现过程介绍完毕。</p>
<p data-nodeid="2082">接下来，我们来看 ShardingSphere 中实现一个自定义 spring-boot-starter 的过程。</p>
<h3 data-nodeid="2083">基于自定义 starter 集成 Spring Boot</h3>
<p data-nodeid="2084">与基于命名空间的实现方式一样，ShardingSphere 提供了 sharding-jdbc-spring-boot-starter 和 sharding-jdbc-orchestration-spring-boot-starter 这两个 starter 工程。篇幅关系，我们同样只关注于 sharding-jdbc-spring-boot-starter 工程。</p>
<p data-nodeid="2085">对于 Spring Boot 工程，我们首先来关注 META-INF 文件夹下的 spring.factories 文件。Spring Boot 中提供了一个 SpringFactoriesLoader 类，该类的运行机制类似于 <strong data-nodeid="2172">“13 | 微内核架构：ShardingSphere如何实现系统的扩展性？”</strong> 中所介绍的 SPI 机制，只不过以服务接口命名的文件是放在 META-INF/spring.factories 文件夹下，对应的 Key 为 EnableAutoConfiguration。SpringFactoriesLoader 会查找所有 META-INF/spring.factories 目录下的配置文件，并把 Key 为 EnableAutoConfiguration 所对应的配置项通过反射实例化为配置类并加载到容器。在 sharding-jdbc-spring-boot-starter 工程中，该文件内容如下所示：</p>
<pre class="lang-xml" data-nodeid="2086"><code data-language="xml">org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.apache.shardingsphere.shardingjdbc.spring.boot.SpringBootConfiguration
</code></pre>
<p data-nodeid="2087">现在这里的 EnableAutoConfiguration 配置项指向了 SpringBootConfiguration 类。也就是说，这个类在 Spring Boot 启动过程中都能够通过 SpringFactoriesLoader 被加载到运行时环境中。</p>
<h4 data-nodeid="2088">1.SpringBootConfiguration 中的注解</h4>
<p data-nodeid="2089">接下来，我们就来到这个 SpringBootConfiguration，首先关注于加在该类上的各种注解，如下所示：</p>
<pre class="lang-java" data-nodeid="2090"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@ComponentScan("org.apache.shardingsphere.spring.boot.converter")</span>
<span class="hljs-meta">@EnableConfigurationProperties({
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringBootShardingRuleConfigurationProperties.class, 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringBootMasterSlaveRuleConfigurationProperties.class, SpringBootEncryptRuleConfigurationProperties.class, SpringBootPropertiesConfigurationProperties.class})</span>
<span class="hljs-meta">@ConditionalOnProperty(prefix = "spring.shardingsphere", name = "enabled", havingValue = "true", matchIfMissing = true)</span>
<span class="hljs-meta">@AutoConfigureBefore(DataSourceAutoConfiguration.class)</span>
<span class="hljs-meta">@RequiredArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringBootConfiguration</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">EnvironmentAware</span>
</span></code></pre>
<p data-nodeid="2091">首先，我们看到了一个 @Configuration 注解。这个注解不是 Spring Boot 引入的新注解，而是属于 Spring 容器管理的内容。该注解表明这个类是一个配置类，可以启动组件扫描，用来将带有 @Bean 注解的实体进行实例化 bean。</p>
<p data-nodeid="2092">然后，我们又看到了一个同样属于 Spring 容器管理范畴的老注解，即 @ComponentScan 注解。@ComponentScan 注解就是扫描基于 @Component 等注解所标注的类所在包下的所有需要注入的类，并把相关 Bean 定义批量加载到IoC容器中。</p>
<p data-nodeid="2093">显然，Spring Boot 应用程序中同样需要这个功能。注意到，这里需要进行扫描的包路径位于另一个代码工程 sharding-spring-boot-util 的 org.apache.shardingsphere.spring.boot.converter 包中。</p>
<p data-nodeid="2094">然后，我们看到了一个 @EnableConfigurationProperties 注解，该注解的作用就是使添加了 @ConfigurationProperties 注解的类生效。在 Spring Boot 中，如果一个类只使用了 @ConfigurationProperties 注解，然后该类没有在扫描路径下或者没有使用 @Component 等注解，就会导致无法被扫描为 bean，那么就必须在配置类上使用 @EnableConfigurationProperties 注解去指定这个类，才能使 @ConfigurationProperties 生效，并作为一个 bean 添加进 spring 容器中。这里的 @EnableConfigurationProperties 注解包含了四个具体的 ConfigurationProperties。以 SpringBootShardingRuleConfigurationProperties 为例，该类的定义如下所示，可以看到，这里直接继承了 sharding-core-common 代码工程中的 YamlShardingRuleConfiguration：</p>
<pre class="lang-java" data-nodeid="2095"><code data-language="java"><span class="hljs-meta">@ConfigurationProperties(prefix = "spring.shardingsphere.sharding")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringBootShardingRuleConfigurationProperties</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">YamlShardingRuleConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="2096">SpringBootConfiguration 上的下一个注解是 @ConditionalOnProperty，该注解的作用在于只有当所提供的属性属于 true 时才会实例化 Bean。</p>
<p data-nodeid="2097">最后一个与自动加载相关的注解是 @AutoConfigureBefore，如果该注解用在类名上，其作用是标识在加载当前类之前需要加载注解中所设置的配置类。基于这一点，我们明确在加载 SpringBootConfiguration 类之前，Spring Boot 会先加载 DataSourceAutoConfiguration。这一步的作用与我们后面要看到的创建各种 DataSource 相关。</p>
<h4 data-nodeid="2098">2.SpringBootConfiguration 中的功能</h4>
<p data-nodeid="2099">介绍完这些注解之后，我们来看一下 SpringBootConfiguration 类所提供的功能。</p>
<p data-nodeid="2100">我们知道对于 ShardingSphere 而言，其对外的入口实际上就是各种 DataSource，因此 SpringBootConfiguration 中提供了一批创建不同 DataSource 的入口方法，例如如下所示的 shardingDataSource 方法：</p>
<pre class="lang-java" data-nodeid="2101"><code data-language="java"><span class="hljs-meta">@Bean</span>
<span class="hljs-meta">@Conditional(ShardingRuleCondition.class)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">shardingDataSource</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ShardingDataSourceFactory.createDataSource(dataSourceMap, <span class="hljs-keyword">new</span> ShardingRuleConfigurationYamlSwapper().swap(shardingRule), props.getProps());
}
</code></pre>
<p data-nodeid="2102">该方法上添加了两个注解，一个是常见的 @Bean，另一个则是 @Conditional 注解，该注解的作用是只有满足指定条件的情况下才能加载这个 Bean。我们看到 @Conditional 注解中设置了一个 ShardingRuleCondition，该类如下所示：</p>
<pre class="lang-java" data-nodeid="2103"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShardingRuleCondition</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">SpringBootCondition</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> ConditionOutcome <span class="hljs-title">getMatchOutcome</span><span class="hljs-params">(<span class="hljs-keyword">final</span> ConditionContext conditionContext, <span class="hljs-keyword">final</span> AnnotatedTypeMetadata annotatedTypeMetadata)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">boolean</span> isMasterSlaveRule = <span class="hljs-keyword">new</span> MasterSlaveRuleCondition().getMatchOutcome(conditionContext, annotatedTypeMetadata).isMatch();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">boolean</span> isEncryptRule = <span class="hljs-keyword">new</span> EncryptRuleCondition().getMatchOutcome(conditionContext, annotatedTypeMetadata).isMatch();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> isMasterSlaveRule || isEncryptRule ? ConditionOutcome.noMatch(<span class="hljs-string">"Have found master-slave or encrypt rule in environment"</span>) : ConditionOutcome.match();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2104">可以看到 ShardingRuleCondition 是一个标准的 SpringBootCondition，实现了 getMatchOutcome 抽象方法。我们知道 SpringBootCondition 的作用就是代表一种用于注册类或加载 Bean 的条件。ShardingRuleCondition 类的实现上分别调用了 MasterSlaveRuleCondition 和 EncryptRuleCondition 来判断是否满足这两个 SpringBootCondition。显然，对于 ShardingRuleCondition 而言，只有在两个条件都不满足的情况下才应该被加载。对于 masterSlaveDataSource 和 encryptDataSource 这两个方法而言，处理逻辑也类似，不做赘述。</p>
<p data-nodeid="2105">最后，我们注意到 SpringBootConfiguration 还实现了 Spring 的 EnvironmentAware 接口。在 Spring Boot 中，当一个类实现了 EnvironmentAware 接口并重写了其中的 setEnvironment 方法之后，在代码工程启动时就可以获得 application.properties 配置文件中各个配置项的属性值。SpringBootConfiguration 中所重写的 setEnvironment 方法如下所示：</p>
<pre class="lang-java" data-nodeid="2106"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">setEnvironment</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Environment environment)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String prefix = <span class="hljs-string">"spring.shardingsphere.datasource."</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (String each : getDataSourceNames(environment, prefix)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; dataSourceMap.put(each, getDataSource(environment, prefix, each));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> ReflectiveOperationException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"Can't find datasource type!"</span>, ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (<span class="hljs-keyword">final</span> NamingException namingEx) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ShardingException(<span class="hljs-string">"Can't find JNDI datasource!"</span>, namingEx);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="2107">这里的代码逻辑是获取“spring.shardingsphere.datasource.name”或“spring.shardingsphere.datasource.names”配置项，然后根据该配置项中所指定的 DataSource 信息构建新的 DataSource 并加载到 dataSourceMap 这个 LinkedHashMap。这点我们可以结合课程案例中的配置项来加深理解：</p>
<pre class="lang-java" data-nodeid="2108"><code data-language="java">spring.shardingsphere.datasource.names=ds0,ds1
&nbsp;
spring.shardingsphere.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds0.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span></span>=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds0.url=jdbc:mysql:<span class="hljs-comment">//localhost/ds0</span>
spring.shardingsphere.datasource.ds0.username=root
spring.shardingsphere.datasource.ds0.password=root
&nbsp;
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds1.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span></span>=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds1.url=jdbc:mysql:<span class="hljs-comment">//localhost/ds1</span>
spring.shardingsphere.datasource.ds1.username=root
spring.shardingsphere.datasource.ds1.password=root
</code></pre>
<p data-nodeid="2109">至此，整个 SpringBootConfiguration 的实现过程介绍完毕。</p>
<h3 data-nodeid="2110">从源码解析到日常开发</h3>
<p data-nodeid="2111">今天所介绍的关于 ShardingSphere 集成 Spring 的实现方法可以直接导入到日常开发过程中。如果我们需要实现一个自定义的框架或工具类，从面向开发人员的角度讲，最好能与 Spring 等主流的开发框架进行集成，以便提供最低的学习和维护成本。与 Spring 框架的集成过程都有固定的开发步骤，我们按照今天课时中所介绍的内容，就可以模仿 ShardingSphere 中的做法自己实现这些步骤。</p>
<h3 data-nodeid="2112">小结与预告</h3>
<p data-nodeid="2113">本课时是 ShardingSphere 源码解析的最后一部分内容，我们围绕如何集成 Spring 框架这一主题对 ShardingSphere 的具体实现方法做了展开。ShardingSphere 在这方面提供了一种可以直接进行参考的模版式的实现方法，包括基于命名空间的 Spring 集成以及基于 starter的Spring Boot 集成方法。</p>
<p data-nodeid="2114">这里给你留一道思考题：在 ShardingSphere 集成 Spring Boot 时，SpringBootConfiguration 类上的注解有哪些，分别起到了什么作用？</p>
<p data-nodeid="2502">讲完 ShardingSphere 源码解析部分内容之后，下一课时是整个课程的最后一讲，我们将对 ShardingSphere 进行总结，并对它的后续发展进行展望。</p>
<p data-nodeid="2503" class=""><a href="https://wj.qq.com/s2/7238084/d702/" data-nodeid="2508">课程评价入口，挑选 5 名小伙伴赠送小礼品~</a></p>

---

### 精选评论

##### **超：
> 请教老师 https://github.com/apache/shardingsphere/issues/9067    这个问题有遇到过么？该怎么解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有碰到哦

##### *闰：
> 老师，请问我使用最新的4.1.1版本连接oracle数据库启动很久都不能启动起来

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不大明确具体是什么问题，请同学描述得再详细些

