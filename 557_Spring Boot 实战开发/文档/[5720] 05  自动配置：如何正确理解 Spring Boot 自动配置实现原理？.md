<p data-nodeid="258352" class="">通过前面几个课时内容的介绍，相信你对 Spring Boot 中的配置体系已经有了全面的了解。Spring Boot 中的配置体系是一套强大而复杂的体系，其中最基础、最核心的要数自动配置（AutoConfiguration）机制了。今天我们将围绕这个话题详细展开讨论，看看 Spring Boot 如何实现自动配置。那我们就先从 @SpringBootApplication 注解开始讲起。</p>
<h3 data-nodeid="258353">@SpringBootApplication 注解</h3>
<p data-nodeid="258354">@SpringBootApplication 注解位于 spring-boot-autoconfigure 工程的 org.springframework.boot.autoconfigure 包中，定义如下：</p>
<pre class="lang-java" data-nodeid="258355"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Inherited</span>
<span class="hljs-meta">@SpringBootConfiguration</span>
<span class="hljs-meta">@EnableAutoConfiguration</span>
<span class="hljs-meta">@ComponentScan(excludeFilters = {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> SpringBootApplication {
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor(annotation = EnableAutoConfiguration.class)</span>
&nbsp;&nbsp;&nbsp; Class&lt;?&gt;[] exclude() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor(annotation = EnableAutoConfiguration.class)</span>
&nbsp;&nbsp;&nbsp; String[] excludeName() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")</span>
&nbsp;&nbsp;&nbsp; String[] scanBasePackages() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")</span>
&nbsp;&nbsp;&nbsp; Class&lt;?&gt;[] scanBasePackageClasses() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="258356">相较一般的注解，@SpringBootApplication 注解显得有点复杂。我们可以通过 exclude 和 excludeName 属性来配置不需要实现自动装配的类或类名，也可以通过 scanBasePackages 和 scanBasePackageClasses 属性来配置需要进行扫描的包路径和类路径。</p>
<p data-nodeid="258357">注意到 @SpringBootApplication 注解实际上是一个组合注解，它由三个注解组合而成，分别是 @SpringBootConfiguration、@EnableAutoConfiguration 和 @ComponentScan。</p>
<ul data-nodeid="258358">
<li data-nodeid="258359">
<p data-nodeid="258360"><strong data-nodeid="258464">@ComponentScan 注解</strong></p>
</li>
</ul>
<p data-nodeid="258361">@ComponentScan 注解不是 Spring Boot 引入的新注解，而是属于 Spring 容器管理的内容。@ComponentScan 注解就是扫描基于 @Component 等注解所标注的类所在包下的所有需要注入的类，并把相关 Bean 定义批量加载到容器中。显然，Spring Boot 应用程序中同样需要这个功能。</p>
<ul data-nodeid="258362">
<li data-nodeid="258363">
<p data-nodeid="258364"><strong data-nodeid="258469">@SpringBootConfiguration 注解</strong></p>
</li>
</ul>
<p data-nodeid="258365">@SpringBootConfiguration 注解比较简单，事实上它是一个空注解，只是使用了 Spring 中的 @Configuration 注解。@Configuration 注解比较常见，提供了 JavaConfig 配置类实现。</p>
<ul data-nodeid="258366">
<li data-nodeid="258367">
<p data-nodeid="258368"><strong data-nodeid="258474">@EnableAutoConfiguration 注解</strong></p>
</li>
</ul>
<p data-nodeid="258369">@EnableAutoConfiguration 注解是我们需要重点剖析的对象，下面进行重点展开。该注解的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="258370"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Inherited</span>
<span class="hljs-meta">@AutoConfigurationPackage</span>
<span class="hljs-meta">@Import(AutoConfigurationImportSelector.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> EnableAutoConfiguration {
&nbsp;
&nbsp;&nbsp;&nbsp; String ENABLED_OVERRIDE_PROPERTY = <span class="hljs-string">"spring.boot.enableautoconfiguration"</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; Class&lt;?&gt;[] exclude() <span class="hljs-keyword">default</span> {};
&nbsp;
&nbsp;&nbsp;&nbsp; String[] excludeName() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="258371">这里我们关注两个新注解，@AutoConfigurationPackage 和 @Import(AutoConfigurationImportSelector.class)。</p>
<h4 data-nodeid="258372">@AutoConfigurationPackage 注解</h4>
<p data-nodeid="258373">@AutoConfigurationPackage 注解定义如下：</p>
<pre class="lang-java" data-nodeid="258374"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Inherited</span>
<span class="hljs-meta">@Import(AutoConfigurationPackages.Registrar.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> AutoConfigurationPackage {
}
</code></pre>
<p data-nodeid="258375">从命名上讲，在这个注解中我们对该注解所在包下的类进行自动配置，而在实现方式上用到了 Spring 中的 @Import 注解。在使用 Spring Boot 时，@Import 也是一个非常常见的注解，可以用来动态创建 Bean。为了便于理解后续内容，这里有必要对 @Import 注解的运行机制做一些展开，该注解定义如下：</p>
<pre class="lang-java" data-nodeid="258376"><code data-language="java"><span class="hljs-meta">@Target(ElementType.TYPE)</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Import {
&nbsp;&nbsp;&nbsp; Class&lt;?&gt;[] value();
}
</code></pre>
<p data-nodeid="258377">在 @Import 注解的属性中可以设置需要引入的类名，例如 @AutoConfigurationPackage 注解上的 @Import(AutoConfigurationPackages.Registrar.class)。根据该类的不同类型，Spring 容器针对 @Import 注解有以下四种处理方式：</p>
<ul data-nodeid="258378">
<li data-nodeid="258379">
<p data-nodeid="258380">如果该类实现了 ImportSelector 接口，Spring 容器就会实例化该类，并且调用其 selectImports 方法；</p>
</li>
<li data-nodeid="258381">
<p data-nodeid="258382">如果该类实现了 DeferredImportSelector 接口，则 Spring 容器也会实例化该类并调用其 selectImports方法。DeferredImportSelector 继承了 ImportSelector，区别在于 DeferredImportSelector 实例的 selectImports 方法调用时机晚于 ImportSelector 的实例，要等到 @Configuration 注解中相关的业务全部都处理完了才会调用；</p>
</li>
<li data-nodeid="258383">
<p data-nodeid="258384">如果该类实现了 ImportBeanDefinitionRegistrar 接口，Spring 容器就会实例化该类，并且调用其 registerBeanDefinitions 方法；</p>
</li>
<li data-nodeid="258385">
<p data-nodeid="258386">如果该类没有实现上述三种接口中的任何一个，Spring 容器就会直接实例化该类。</p>
</li>
</ul>
<p data-nodeid="258387">有了对 @Import 注解的基本理解，我们再来看 AutoConfigurationPackages.Registrar 类，定义如下：</p>
<pre class="lang-java" data-nodeid="258388"><code data-language="java"><span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Registrar</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ImportBeanDefinitionRegistrar</span>, <span class="hljs-title">DeterminableImports</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerBeanDefinitions</span><span class="hljs-params">(AnnotationMetadata metadata,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BeanDefinitionRegistry registry)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; register(registry, <span class="hljs-keyword">new</span> PackageImport(metadata).getPackageName());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Set&lt;Object&gt; <span class="hljs-title">determineImports</span><span class="hljs-params">(AnnotationMetadata metadata)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Collections.singleton(<span class="hljs-keyword">new</span> PackageImport(metadata));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="258389">可以看到这个 Registrar 类实现了前面第三种情况中提到的 ImportBeanDefinitionRegistrar 接口并重写了 registerBeanDefinitions 方法，该方法中调用 AutoConfigurationPackages 自身的 register 方法：</p>
<pre class="lang-java" data-nodeid="258390"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">register</span><span class="hljs-params">(BeanDefinitionRegistry registry, String... packageNames)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (registry.containsBeanDefinition(BEAN)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; BeanDefinition beanDefinition = registry.getBeanDefinition(BEAN);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ConstructorArgumentValues constructorArguments = beanDefinition
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getConstructorArgumentValues();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; constructorArguments.addIndexedArgumentValue(<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; addBasePackages(constructorArguments, packageNames));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; GenericBeanDefinition beanDefinition = <span class="hljs-keyword">new</span> GenericBeanDefinition();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; beanDefinition.setBeanClass(BasePackages.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; beanDefinition.getConstructorArgumentValues().addIndexedArgumentValue(<span class="hljs-number">0</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; packageNames);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; beanDefinition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; registry.registerBeanDefinition(BEAN, beanDefinition);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="258391">这个方法的逻辑是先判断整个 Bean 有没有被注册，如果已经注册则获取 Bean 的定义，通过 Bean 获取构造函数的参数并添加参数值；如果没有，则创建一个新的 Bean 的定义，设置 Bean 的类型为 AutoConfigurationPackages 类型并进行 Bean 的注册。</p>
<h4 data-nodeid="258392">AutoConfigurationImportSelector</h4>
<p data-nodeid="258393">然后我们再来看 @EnableAutoConfiguration 注解中的 @Import(AutoConfigurationImportSelector.class) 部分，首先我们明确 AutoConfigurationImportSelector 类实现了 @Import 注解第二种情况中的 DeferredImportSelector 接口，所以会执行如下所示的 selectImports 方法：</p>
<pre class="lang-java" data-nodeid="258394"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-keyword">public</span> String[] selectImports(AnnotationMetadata annotationMetadata) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!isEnabled(annotationMetadata)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> NO_IMPORTS;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .loadMetadata(<span class="hljs-keyword">this</span>.beanClassLoader);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AnnotationAttributes attributes = getAttributes(annotationMetadata);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 configurations 集合</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;String&gt; configurations = getCandidateConfigurations(annotationMetadata,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; attributes);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; configurations = removeDuplicates(configurations);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Set&lt;String&gt; exclusions = getExclusions(annotationMetadata, attributes);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; checkExcludedClasses(configurations, exclusions);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; configurations.removeAll(exclusions);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; configurations = filter(configurations, autoConfigurationMetadata);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; fireAutoConfigurationImportEvents(configurations, exclusions);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> StringUtils.toStringArray(configurations);
}
</code></pre>
<p data-nodeid="258395">这段代码的核心是通过 getCandidateConfigurations 方法获取 configurations 集合并进行过滤。getCandidateConfigurations 方法如下所示：</p>
<pre class="lang-java" data-nodeid="258396"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> List&lt;String&gt; <span class="hljs-title">getCandidateConfigurations</span><span class="hljs-params">(AnnotationMetadata metadata,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AnnotationAttributes attributes)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;String&gt; configurations = SpringFactoriesLoader.loadFactoryNames(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getSpringFactoriesLoaderFactoryClass(), getBeanClassLoader());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notEmpty(configurations,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"No auto configuration classes found in META-INF/spring.factories. If you "</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; + <span class="hljs-string">"are using a custom packaging, make sure that file is correct."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> configurations;
}
</code></pre>
<p data-nodeid="258397">这段代码中可以先关注 Assert 校验，该校验是一个非空校验，会提示 “在 META-INF/spring.factories 中没有找到自动配置类” 这个异常信息。看到这里，不得不提到 JDK 中的 SPI 机制，因为无论从 SpringFactoriesLoader 这个类的命名上，还是 META-INF/spring.factories 这个文件目录，两者之间都存在很大的相通性。关于 JDK 中的 SPI 机制，我们在 05 讲的后续内容中马上就会介绍到。</p>
<p data-nodeid="258398">从类名上看，AutoConfigurationImportSelector 类是一种选择器，负责从各种配置项中找到需要导入的具体配置类。该类的结构如下图所示：</p>
<p data-nodeid="259314"><img src="https://s0.lgstatic.com/i/image/M00/73/9F/Ciqc1F_GIU2AGFv2AACsHHV_6h0534.png" alt="图片6.png" data-nodeid="259317"></p>

<div data-nodeid="258922"><p style="text-align:center">AutoConfigurationImportSelector 类层结构图</p></div>




<p data-nodeid="258401">显然，AutoConfigurationImportSelector 所依赖的最关键组件就是 SpringFactoriesLoader，下面我们对其进行具体展开。</p>
<h3 data-nodeid="258402">SPI 机制和 SpringFactoriesLoader</h3>
<p data-nodeid="258403">要想理解 SpringFactoriesLoader 类，我们首先需要了解 JDK 中 SPI（Service Provider Interface，服务提供者接口）机制。</p>
<h4 data-nodeid="258404">JDK 中的 SPI 机制</h4>
<p data-nodeid="258405">JDK 提供了用于服务查找的一个工具类 java.util.ServiceLoader 来实现 SPI 机制。当服务提供者提供了服务接口的一种实现之后，我们可以在 jar 包的 META-INF/services/ 目录下创建一个以服务接口命名的文件，该文件里配置着一组 Key-Value，用于指定服务接口与实现该服务接口具体实现类的映射关系。而当外部程序装配这个 jar 包时，就能通过该 jar 包 META-INF/services/ 目录中的配置文件找到具体的实现类名，并装载实例化，从而完成模块的注入。SPI 提供了一种约定，基于该约定就能很好地找到服务接口的实现类，而不需要在代码里硬编码指定。JDK 中 SPI 机制开发流程如下图所示：</p>
<p data-nodeid="260280"><img src="https://s0.lgstatic.com/i/image/M00/73/AA/CgqCHl_GIVmABagiAAEbqB5E-U0604.png" alt="图片7.png" data-nodeid="260283"></p>

<div data-nodeid="259888"><p style="text-align:center">JDK 中 SPI 机制开发流程图</p></div>




<h4 data-nodeid="258408">SpringFactoriesLoader</h4>
<p data-nodeid="258409">SpringFactoriesLoader 类似这种 SPI 机制，只不过以服务接口命名的文件是放在 META-INF/spring.factories 文件夹下，对应的 Key 为 EnableAutoConfiguration。SpringFactoriesLoader 会查找所有 META-INF/spring.factories 文件夹中的配置文件，并把 Key 为 EnableAutoConfiguration 所对应的配置项通过反射实例化为配置类并加载到容器中。这一点我们可以在 SpringFactoriesLoader 的 loadSpringFactories 方法中进行印证：</p>
<pre class="lang-java" data-nodeid="258410"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Map&lt;String, List&lt;String&gt;&gt; loadSpringFactories(<span class="hljs-meta">@Nullable</span> ClassLoader classLoader) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MultiValueMap&lt;String, String&gt; result = cache.get(classLoader);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (result != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Enumeration&lt;URL&gt; urls = (classLoader != <span class="hljs-keyword">null</span> ?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result = <span class="hljs-keyword">new</span> LinkedMultiValueMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (urls.hasMoreElements()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; URL url = urls.nextElement();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UrlResource resource = <span class="hljs-keyword">new</span> UrlResource(url);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Properties properties = PropertiesLoaderUtils.loadProperties(resource);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;?, ?&gt; entry : properties.entrySet()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String factoryClassName = ((String) entry.getKey()).trim();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (String factoryName : StringUtils.commaDelimitedListToStringArray((String) entry.getValue())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.add(factoryClassName, factoryName.trim());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cache.put(classLoader, result);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (IOException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"Unable to load factories from location ["</span> +
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; FACTORIES_RESOURCE_LOCATION + <span class="hljs-string">"]"</span>, ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="258411">以下就是 spring-boot-autoconfigure 工程中所使用的 spring.factories 配置文件片段，可以看到 EnableAutoConfiguration 项中包含了各式各样的配置项，这些配置项在 Spring Boot 启动过程中都能够通过 SpringFactoriesLoader 加载到运行时环境，从而实现自动化配置：</p>
<pre class="lang-xml" data-nodeid="258412"><code data-language="xml"># Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
…
</code></pre>
<p data-nodeid="258413">以上就是 Spring Boot 中基于 @SpringBootApplication 注解实现自动配置的基本过程和原理。当然，@SpringBootApplication 注解也可以基于外部配置文件加载配置信息。基于约定优于配置思想，Spring Boot 在加载外部配置文件的过程中大量使用了默认配置。</p>
<h3 data-nodeid="258414">@ConditionalOn 系列条件注解</h3>
<p data-nodeid="258415">Spring Boot 默认提供了 100 多个 AutoConfiguration 类，显然我们不可能会全部引入。所以在自动装配时，系统会去类路径下寻找是否有对应的配置类。如果有对应的配置类，则按条件进行判断，决定是否需要装配。这里就引出了在阅读 Spring Boot 代码时经常会碰到的另一批注解，即 @ConditionalOn 系列条件注解。</p>
<h4 data-nodeid="258416">@ConditionalOn 系列条件注解的示例</h4>
<p data-nodeid="258417">我们先通过一个简单的示例来了解 @ConditionalOn 系列条件注解的使用方式，例如以下代码就是这类注解的一种典型应用，该代码位于 Spring Cloud Config 的客户端代码工程 spring-cloud-config-client 中：</p>
<pre class="lang-java" data-nodeid="258418"><code data-language="java"><span class="hljs-meta">@Bean</span>&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@ConditionalOnMissingBean(ConfigServicePropertySourceLocator.class)</span>
<span class="hljs-meta">@ConditionalOnProperty(value = "spring.cloud.config.enabled", matchIfMissing = true)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> ConfigServicePropertySourceLocator <span class="hljs-title">configServicePropertySource</span><span class="hljs-params">(ConfigClientProperties properties)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ConfigServicePropertySourceLocator locator = <span class="hljs-keyword">new</span> ConfigServicePropertySourceLocator(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; properties);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> locator;
}
</code></pre>
<p data-nodeid="258419">可以看到，这里运用了两个 @ConditionalOn 注解，一个是 @ConditionalOnMissingBean，一个是 @ConditionalOnProperty。再比如在 Spring Cloud Config 的服务器端代码工程 spring-cloud-config-server 中，存在如下 ConfigServerAutoConfiguration 自动配置类：</p>
<pre class="lang-java" data-nodeid="258420"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@ConditionalOnBean(ConfigServerConfiguration.Marker.class)</span>
<span class="hljs-meta">@EnableConfigurationProperties(ConfigServerProperties.class)</span>
<span class="hljs-meta">@Import({ EnvironmentRepositoryConfiguration.class, CompositeConfiguration.class, ResourceRepositoryConfiguration.class,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ConfigServerEncryptionConfiguration.class, ConfigServerMvcConfiguration.class })</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ConfigServerAutoConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="258421">这里我们运用了 @ConditionalOnBean 注解。实际上，Spring Boot 中提供了一系列的条件注解，常见的包括：</p>
<ul data-nodeid="258422">
<li data-nodeid="258423">
<p data-nodeid="258424">@ConditionalOnProperty：只有当所提供的属性属于 true 时才会实例化 Bean</p>
</li>
<li data-nodeid="258425">
<p data-nodeid="258426">@ConditionalOnBean：只有在当前上下文中存在某个对象时才会实例化 Bean</p>
</li>
<li data-nodeid="258427">
<p data-nodeid="258428">@ConditionalOnClass：只有当某个 Class 位于类路径上时才会实例化 Bean</p>
</li>
<li data-nodeid="258429">
<p data-nodeid="258430">@ConditionalOnExpression：只有当表达式为 true 的时候才会实例化 Bean</p>
</li>
<li data-nodeid="258431">
<p data-nodeid="258432">@ConditionalOnMissingBean：只有在当前上下文中不存在某个对象时才会实例化 Bean</p>
</li>
<li data-nodeid="258433">
<p data-nodeid="258434">@ConditionalOnMissingClass：只有当某个 Class 在类路径上不存在的时候才会实例化 Bean</p>
</li>
<li data-nodeid="258435">
<p data-nodeid="258436">@ConditionalOnNotWebApplication：只有当不是 Web 应用时才会实例化 Bean</p>
</li>
</ul>
<p data-nodeid="258437">当然 Spring Boot 还提供了一些不大常用的 @ConditionalOnXXX 注解，这些注解都定义在 org.springframework.boot.autoconfigure.condition 包中。</p>
<p data-nodeid="258438">显然上述 ConfigServicePropertySourceLocator 类中只有在 "spring.cloud.config.enabled" 属性为 true（通过 matchIfMissing 配置项表示默认即为 true）以及类路径上不存在 ConfigServicePropertySourceLocator 时才会进行实例化。而 ConfigServerAutoConfiguration 只有在类路径上存在 ConfigServerConfiguration.Marker 类时才会进行实例化，这是一种常用的自动配置控制技巧。</p>
<h4 data-nodeid="258439">@ConditionalOn 系列条件注解的实现原理</h4>
<p data-nodeid="258440">@ConditionalOn 系列条件注解非常多，我们无意对所有这些组件进行展开。事实上这些注解的实现原理也大致相同，我们只需要深入了解其中一个就能做到触类旁通。这里我们挑选 @ConditionalOnClass 注解进行展开，该注解定义如下：</p>
<pre class="lang-java" data-nodeid="258441"><code data-language="java"><span class="hljs-meta">@Target({ ElementType.TYPE, ElementType.METHOD })</span>
<span class="hljs-meta">@Retention(RetentionPolicy.RUNTIME)</span>
<span class="hljs-meta">@Documented</span>
<span class="hljs-meta">@Conditional(OnClassCondition.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> ConditionalOnClass {
&nbsp; Class&lt;?&gt;[] value() <span class="hljs-keyword">default</span> {};
&nbsp; String[] name() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="258442">可以看到， @ConditionalOnClass 注解本身带有两个属性，一个 Class 类型的 value，一个 String 类型的 name，所以我们可以采用这两种方式中的任意一种来使用该注解。同时 ConditionalOnClass 注解本身还带了一个 @Conditional(OnClassCondition.class) 注解。所以， ConditionalOnClass 注解的判断条件其实就包含在 OnClassCondition 这个类中。</p>
<p data-nodeid="258443">OnClassCondition 是 SpringBootCondition 的子类，而 SpringBootCondition 又实现了Condition 接口。Condition 接口只有一个 matches 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="258444"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Condition</span> </span>{
<span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(ConditionContext context, AnnotatedTypeMetadata metadata)</span></span>;
}
</code></pre>
<p data-nodeid="258445">SpringBootCondition 中的 matches 方法实现如下：</p>
<pre class="lang-java" data-nodeid="258446"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(ConditionContext context,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AnnotatedTypeMetadata metadata)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String classOrMethodName = getClassOrMethodName(metadata);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ConditionOutcome outcome = getMatchOutcome(context, metadata);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logOutcome(classOrMethodName, outcome);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; recordEvaluation(context, classOrMethodName, outcome);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> outcome.isMatch();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略其他方法</span>
}
</code></pre>
<p data-nodeid="258447">这里的 getClassOrMethodName 方法获取被添加了@ConditionalOnClass 注解的类或者方法的名称，而 getMatchOutcome 方法用于获取匹配的输出。我们看到 getMatchOutcome 方法实际上是一个抽象方法，需要交由 SpringBootCondition 的各个子类完成实现，这里的子类就是 OnClassCondition 类。在理解 OnClassCondition 时，我们需要明白在 Spring Boot 中，@ConditionalOnClass 或者 @ConditionalOnMissingClass 注解对应的条件类都是 OnClassCondition，所以在 OnClassCondition 的 getMatchOutcome 中会同时处理两种情况。这里我们挑选处理 @ConditionalOnClass 注解的代码，核心逻辑如下所示：</p>
<pre class="lang-java" data-nodeid="258448"><code data-language="java">ClassLoader classLoader = context.getClassLoader();
ConditionMessage matchMessage = ConditionMessage.empty();
List&lt;String&gt; onClasses = getCandidates(metadata, ConditionalOnClass.class);
<span class="hljs-keyword">if</span> (onClasses != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;String&gt; missing = getMatches(onClasses, MatchType.MISSING, classLoader);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!missing.isEmpty()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ConditionOutcome
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .noMatch(ConditionMessage.forCondition(ConditionalOnClass.class)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .didNotFind("required class", "required classes")
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .items(Style.QUOTE, missing));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; matchMessage = matchMessage.andCondition(ConditionalOnClass.class)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .found("required class", "required classes").items(Style.QUOTE, getMatches(onClasses, MatchType.PRESENT, classLoader));
}
</code></pre>
<p data-nodeid="258449">这里有两个方法值得注意，一个是 getCandidates 方法，一个是 getMatches 方法。首先通过 getCandidates 方法获取了 ConditionalOnClass 的 name 属性和 value 属性。然后通过 getMatches 方法将这些属性值进行比对，得到这些属性所指定的但在类加载器中不存在的类。如果发现类加载器中应该存在但事实上又不存在的类，则返回一个匹配失败的 Condition；反之，如果类加载器中存在对应类的话，则把匹配信息进行记录并返回一个 ConditionOutcome。</p>
<h3 data-nodeid="258450">从源码解析到日常开发</h3>
<p data-nodeid="258451">在今天的内容中，我们接触到了 Spring Boot 开发过程中非常核心的话题，即自动配置。自动配置是理解 Spring Boot 应用程序构建和运行的关键要素。当我们尝试去理解一个基于 Spring Boot 开发的工具或框架时，今天的内容能帮助你快速切入该工具或框架的实现原理。同时，在日常开发过程中，诸如 SPI 机制和 @ConditionalOn 系列条件注解也都可以直接应用到我们自身的系统设计和开发中，从而提供高扩展性的架构实现方案。</p>
<h3 data-nodeid="258452">小结与预告</h3>
<p data-nodeid="258453">可以说，自动配置是 Spring Boot 最核心和最基本的功能，而 @SpringBootApplication 注解又是 Spring Boot 应用程序的入口。本课时从 @SpringBootApplication 注解入手，详细分析了自动配置机制的实现过程。涉及的知识点比较多，包含 JDK 中的 SPI 机制，以及 @ConditionalOn 系列条件注解，需要你进行分析和掌握。</p>
<p data-nodeid="258454">这里给你留一道思考题：在 Spring Boot 中，如何基于 JDK 中的 SPI 机制完成对配置类的自动加载？欢迎你在留言区与我交流、互动。</p>
<p data-nodeid="258455" class="">介绍完配置体系之后，06 讲我们将进入一个全新的主题，即介绍 Spring Boot 中数据访问层的构建方式。</p>

---

### 精选评论

##### **伟：
> 同时 ConditionalOnClass 注解本身还带了一个 @Conditional(OnClassCondition.class) 
注解。所以， ConditionalOnClass 注解的判断条件其实就包含在 OnClassCondition 这个类中这个所以，是怎么看出来的呢？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一个注解定义上标注了另一个注解，就说明有一种依赖关系，从依赖关系可以推导出具体的逻辑位于哪一个实现类中。

##### **伟：
> AutoConfigurationImportSelector 类层结构图，这个图没看明白，这是类层结构图？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 只是表明了各个类之间的简单依赖关系，不算严格意义上的类层结构图。

##### **旭：
> 这些注解是怎么被springboot识别出来的呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是 Spring Boot 内部的处理机制了，原理上就是传统 Spring 中注解编写和实现的那一套。

##### *淦：
> "在 Spring Boot 中，如何基于 JDK 中的 SPI 机制完成对配置类的自动加载？", 想问下老师这里的配置类具体指的是被@Configuration注解修饰的类吗?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是，是放在META-INF/services目录下所指定的配置类。

