<p data-nodeid="10286" class="">很多开源框架之所以能够流行起来，是因为它们解决了领域内的一些通用问题。但在实际使用这些开源框架的时候，我们都是要解决通用问题中的一个特例问题，所以这时我们就需要使用一种方式来控制开源框架的行为，这就是开源框架提供各种各样配置的核心原因之一。</p>
<p data-nodeid="10287">现在控制开源框架行为主流的配置方式就是 XML 配置方式和注解方式。在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6373" data-nodeid="10419">《02 | 订单系统持久层示例分析，20 分钟带你快速上手 MyBatis》</a>这一讲中我们介绍过，MyBatis 有两方面的 XML 配置，<strong data-nodeid="10425">一个是 mybatis-config.xml 配置文件中的整体配置，另一个是 Mapper.xml 配置文件中的 SQL 语句</strong>。当然，MyBatis 中也有注解，前面的课程中也多少有涉及，其核心实现与 XML 配置基本类似，所以这一讲我们就重点分析 XML 配置的初始化过程，注解相关的内容就留给你自己分析了。</p>
<p data-nodeid="10288">在初始化的过程中，MyBatis 会读取 mybatis-config.xml 这个全局配置文件以及所有的 Mapper 映射配置文件，同时还会加载这两个配置文件中指定的类，解析类中的相关注解，最终将解析得到的信息转换成配置对象。<strong data-nodeid="10431">完成配置加载之后，MyBatis 就会根据得到的配置对象初始化各个模块</strong>。</p>
<p data-nodeid="10289">MyBatis 在加载配置文件、创建配置对象的时候，会使用到经典设计模式中的<strong data-nodeid="10437">构造者模式</strong>，所以下面我们就来先介绍一下构造者模式的知识点。</p>
<h3 data-nodeid="10290">构造者模式</h3>
<p data-nodeid="10291">构造者模式最核心的思想就是将创建复杂对象的过程与复杂对象本身进行拆分。通俗来讲，构造者模式是<strong data-nodeid="10444">将复杂对象的创建过程分解成了多个简单步骤，在创建复杂对象的时候，只需要了解复杂对象的基本属性即可，而不需要关心复杂对象的内部构造过程</strong>。这样的话，使用方只需要关心这个复杂对象要什么数据，而不再关心内部细节。</p>
<p data-nodeid="10292">构造者模式的类图如下所示：</p>
<p data-nodeid="10713" class=""><img src="https://s0.lgstatic.com/i/image6/M01/08/8C/Cgp9HWA01CyAP_FuAAGR6B2VRBg565.png" alt="2021223-18655.png" data-nodeid="10717"></p>
<div data-nodeid="10714"><p style="text-align:center">构造者模式类图</p></div>


<p data-nodeid="10295">从图中，我们可以看到构造者模式的四个核心组件。</p>
<ul data-nodeid="10296">
<li data-nodeid="10297">
<p data-nodeid="10298">Product 接口：复杂对象的接口，定义了要创建的目标对象的行为。</p>
</li>
<li data-nodeid="10299">
<p data-nodeid="10300">ProductImpl 类：Product 接口的实现，它真正要创建的复杂对象，其中实现了我们需要的复杂业务逻辑。</p>
</li>
<li data-nodeid="10301">
<p data-nodeid="10302">Builder 接口：定义了构造 Product 对象的每一步行为。</p>
</li>
<li data-nodeid="10303">
<p data-nodeid="10304">BuilderImpl 类：Builder 接口的具体实现，其中具体实现了构造一个 Product 的每一个步骤，例如上图中的 setPart1()、setPart2() 等方法，都是用来构造 ProductImpl 对象的各个部分。在完成整个 Product 对象的构造之后，我们会通过 build() 方法返回这个构造好的 Product 对象。</p>
</li>
</ul>
<p data-nodeid="10305">使用构造者模式一般有两个目的。第一个目的是<strong data-nodeid="10463">将使用方与复杂对象的内部细节隔离，从而实现解耦的效果</strong>。使用方提供的所有信息，都是由 Builder 这个“中间商”接收的，然后由 Builder 消化这些信息并构造出一个完整可用的 Product 对象。第二个目的是<strong data-nodeid="10464">简化复杂对象的构造过程</strong>。在很多场景中，复杂对象可能有很多默认属性，这时我们就可以将这些默认属性封装到 Builder 中，这样就可以简化创建复杂对象所需的信息。</p>
<p data-nodeid="10306">通过构建者模式的类图我们还可以看出，<strong data-nodeid="10470">每个 BuilderImpl 实现都是能够独立创建出对应的 ProductImpl 对象</strong>，那么在程序需要扩展的时候，我们只需要添加新的 BuilderImpl 和 ProductImpl，就能实现功能的扩展，这完全符合“开放-封闭原则”。</p>
<h3 data-nodeid="10307">mybatis-config.xml 解析全流程</h3>
<p data-nodeid="10308">介绍完构造者模式相关的知识点之后，下面我们正式开始介绍 MyBatis 的初始化过程。</p>
<p data-nodeid="10309"><strong data-nodeid="10477">MyBatis 初始化的第一个步骤就是加载和解析 mybatis-config.xml 这个全局配置文件</strong>，入口是 XMLConfigBuilder 这个 Builder 对象，它由 SqlSessionFactoryBuilder.build() 方法创建。XMLConfigBuilder 会解析 mybatis-config.xml 配置文件得到对应的 Configuration 全局配置对象，然后 SqlSessionFactoryBuilder 会根据得到的 Configuration 全局配置对象创建一个 DefaultSqlSessionFactory 对象返回给上层使用。</p>
<p data-nodeid="10310">这里<strong data-nodeid="10483">创建的 XMLConfigBuilder 对象的核心功能就是解析 mybatis-config.xml 配置文件</strong>。XMLConfigBuilder 有一部分能力继承自 BaseBuilder 抽象类，具体继承关系如下图所示：</p>
<p data-nodeid="11572" class=""><img src="https://s0.lgstatic.com/i/image6/M01/08/8C/Cgp9HWA01DeAFFn1AAEKQNyimxk937.png" alt="2021223-1877.png" data-nodeid="11576"></p>
<div data-nodeid="11573"><p style="text-align:center">BaseBuilder 继承关系图</p></div>


<p data-nodeid="10313">BaseBuilder 抽象类扮演了构造者模式中 Builder 接口的角色，下面我们先来看 BaseBuilder 中各个字段的定义。</p>
<ul data-nodeid="10314">
<li data-nodeid="10315">
<p data-nodeid="10316">configuration（Configuration 类型）：MyBatis 的初始化过程就是围绕 Configuration 对象展开的，我们可以认为 Configuration 是一个单例对象，MyBatis 初始化解析到的全部配置信息都会记录到 Configuration 对象中。</p>
</li>
<li data-nodeid="10317">
<p data-nodeid="10318">typeAliasRegistry（TypeAliasRegistry 类型）：别名注册中心。比如，<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6373" data-nodeid="10492">02 讲的订单系统示例</a>中，我们在 mybatis-config.xml 配置文件中，使用 标签为很多类定义了别名。</p>
</li>
<li data-nodeid="10319">
<p data-nodeid="10320">typeHandlerRegistry（TypeHandlerRegistry 类型）：TypeHandler 注册中心。除了定义别名之外，我们在 mybatis-config.xml 配置文件中，还可以使用 <code data-backticks="1" data-nodeid="10495">&lt;typeHandlers&gt;</code> 标签添加自定义 TypeHandler 实现，实现数据库类型与 Java 类型的自定义转换，这些自定义的 TypeHandler 都会记录在这个 TypeHandlerRegistry 对象中。</p>
</li>
</ul>
<p data-nodeid="10321">除了关联 Configuration 对象之外，BaseBuilder 还提供了另外两个基本能力：</p>
<ul data-nodeid="10322">
<li data-nodeid="10323">
<p data-nodeid="10324"><strong data-nodeid="10502">解析别名</strong>，核心逻辑是在 resolveAlias() 方法中实现的，主要依赖于 TypeAliasRegistry 对象；</p>
</li>
<li data-nodeid="10325">
<p data-nodeid="10326"><strong data-nodeid="10507">解析 TypeHandler</strong>，核心逻辑是在 resolveTypeHandler() 方法中实现的，主要依赖于 TypeHandlerRegistry 对象。</p>
</li>
</ul>
<p data-nodeid="10327">了解了 BaseBuilder 提供的基础能力之后，我们回到 XMLConfigBuilder 这个 Builder 实现类，看看它是如何解析 mybatis-config.xml 配置文件的。</p>
<p data-nodeid="10328">首先我们来了解一下 XMLConfigBuilder 的核心字段。</p>
<ul data-nodeid="10329">
<li data-nodeid="10330">
<p data-nodeid="10331">parsed（boolean 类型）：状态标识字段，记录当前 XMLConfigBuilder 对象是否已经成功解析完 mybatis-config.xml 配置文件。</p>
</li>
<li data-nodeid="10332">
<p data-nodeid="10333">parser（XPathParser 类型）：XPathParser 对象是一个 XML 解析器，这里的 parser 对象就是用来解析 mybatis-config.xml 配置文件的。</p>
</li>
<li data-nodeid="10334">
<p data-nodeid="10335">environment（String 类型）： 标签定义的环境名称。</p>
</li>
<li data-nodeid="10336">
<p data-nodeid="10337">localReflectorFactory（ReflectorFactory 类型）：ReflectorFactory 接口的核心功能是实现对 Reflector 对象的创建和缓存。</p>
</li>
</ul>
<p data-nodeid="10338">在 SqlSessionFactoryBuilder.build() 方法中也可以看到，XMLConfigBuilder.parse() 方法触发了 mybatis-config.xml 配置文件的解析，<strong data-nodeid="10519">其中的 parseConfiguration() 方法定义了解析 mybatis-config.xml 配置文件的完整流程</strong>，核心步骤如下：</p>
<ul data-nodeid="10339">
<li data-nodeid="10340">
<p data-nodeid="10341">解析 <code data-backticks="1" data-nodeid="10521">&lt;properties&gt;</code> 标签；</p>
</li>
<li data-nodeid="10342">
<p data-nodeid="10343">解析 <code data-backticks="1" data-nodeid="10524">&lt;settings&gt;</code> 标签；</p>
</li>
<li data-nodeid="10344">
<p data-nodeid="10345">处理日志相关组件；</p>
</li>
<li data-nodeid="10346">
<p data-nodeid="10347">解析 <code data-backticks="1" data-nodeid="10528">&lt;typeAliases&gt;</code> 标签；</p>
</li>
<li data-nodeid="10348">
<p data-nodeid="10349">解析 <code data-backticks="1" data-nodeid="10531">&lt;plugins&gt;</code> 标签；</p>
</li>
<li data-nodeid="10350">
<p data-nodeid="10351">解析 <code data-backticks="1" data-nodeid="10534">&lt;objectFactory&gt;</code> 标签；</p>
</li>
<li data-nodeid="10352">
<p data-nodeid="10353">解析 <code data-backticks="1" data-nodeid="10537">&lt;objectWrapperFactory&gt;</code> 标签；</p>
</li>
<li data-nodeid="10354">
<p data-nodeid="10355">解析 <code data-backticks="1" data-nodeid="10540">&lt;reflectorFactory&gt;</code> 标签；</p>
</li>
<li data-nodeid="10356">
<p data-nodeid="10357">解析 <code data-backticks="1" data-nodeid="10543">&lt;environments&gt;</code> 标签；</p>
</li>
<li data-nodeid="10358">
<p data-nodeid="10359">解析 <code data-backticks="1" data-nodeid="10546">&lt;databaseIdProvider&gt;</code> 标签；</p>
</li>
<li data-nodeid="10360">
<p data-nodeid="10361">解析 <code data-backticks="1" data-nodeid="10549">&lt;typeHandlers&gt;</code> 标签；</p>
</li>
<li data-nodeid="10362">
<p data-nodeid="10363">解析 <code data-backticks="1" data-nodeid="10552">&lt;mappers&gt;</code> 标签。</p>
</li>
</ul>
<p data-nodeid="10364">从 parseConfiguration()方法中，我们可以清晰地看到 XMLConfigBuilder 对 mybatis-config.xml 配置文件中各类标签的解析方法，下面我们就逐一介绍这些方法的核心实现。</p>
<h4 data-nodeid="10365">1. 处理<code data-backticks="1" data-nodeid="10558">&lt;properties&gt;</code>标签</h4>
<p data-nodeid="10366">我们可以通过 <code data-backticks="1" data-nodeid="10561">&lt;properties&gt;</code> 标签定义 KV 信息供 MyBatis 使用，propertiesElement() 方法的核心逻辑就是解析 mybatis-config.xml 配置文件中的 <code data-backticks="1" data-nodeid="10563">&lt;properties&gt;</code> 标签。</p>
<p data-nodeid="10367">从 <code data-backticks="1" data-nodeid="10566">&lt;properties&gt;</code> 标签中解析出来的 KV 信息会被记录到一个 Properties 对象（也就是 Configuration 全局配置对象的 variables 字段），在后续解析其他标签的时候，MyBatis 会使用这个 Properties 对象中记录的 KV 信息替换匹配的占位符。</p>
<h4 data-nodeid="10368">2. 处理<code data-backticks="1" data-nodeid="10571">&lt;settings&gt;</code>标签</h4>
<p data-nodeid="10369">MyBatis 中有很多全局性的配置，例如，是否使用二级缓存、是否开启懒加载功能等，这些都是通过 mybatis-config.xml 配置文件中的 <code data-backticks="1" data-nodeid="10574">&lt;settings&gt;</code> 标签进行配置的。</p>
<p data-nodeid="10370">XMLConfigBuilder.settingsAsProperties() 方法的核心逻辑就是解析 <code data-backticks="1" data-nodeid="10577">&lt;settings&gt;</code> 标签，并将解析得到的配置信息记录到 Configuration 这个全局配置对象的同名属性中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="10371"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Properties <span class="hljs-title">settingsAsProperties</span><span class="hljs-params">(XNode context)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (context == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Properties();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 处理&lt;settings&gt;标签的所有子标签，也就是&lt;setting&gt;标签，将其name属性和value属性</span>
&nbsp; &nbsp; <span class="hljs-comment">// 整理到Properties对象中保存</span>
&nbsp; &nbsp; Properties props = context.getChildrenAsProperties();
&nbsp; &nbsp; <span class="hljs-comment">// 创建Configuration对应的MetaClass对象</span>
&nbsp; &nbsp; MetaClass metaConfig = MetaClass.forClass(Configuration.class, localReflectorFactory);
&nbsp; &nbsp; // 检测Configuration对象中是否包含每个配置项的setter方法
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Object key : props.keySet()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!metaConfig.hasSetter(String.valueOf(key))) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BuilderException(<span class="hljs-string">"The setting "</span> + key + <span class="hljs-string">" is not known.&nbsp; Make sure you spelled it correctly (case sensitive)."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> props;
}
</code></pre>
<h4 data-nodeid="10372">3. 处理<code data-backticks="1" data-nodeid="10582">&lt;typeAliases&gt;</code>和<code data-backticks="1" data-nodeid="10584">&lt;typeHandlers&gt;</code>标签</h4>
<p data-nodeid="10373">XMLConfigBuilder 中提供了 typeAliasesElement() 方法和 typeHandlerElement() 方法，分别用来负责处理 <code data-backticks="1" data-nodeid="10587">&lt;typeAliases&gt;</code> 标签和 <code data-backticks="1" data-nodeid="10589">&lt;typeHandlers&gt;</code> 标签，解析得到的别名信息和 TypeHandler 信息就会分别记录到 TypeAliasRegistry 和 TypeHandlerRegistry（前面介绍 BaseHandler 的时候，我们已经简单介绍过这两者了）。</p>
<p data-nodeid="10374">下面我们以 typeHandlerElement() 方法为例来分析一下这个过程：</p>
<pre class="lang-java" data-nodeid="10375"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">typeHandlerElement</span><span class="hljs-params">(XNode parent)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (parent != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (XNode child : parent.getChildren()) { <span class="hljs-comment">// 处理全部&lt;typeHandler&gt;子标签</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-string">"package"</span>.equals(child.getName())) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果指定了package属性，则扫描指定包中所有的类，</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 并解析@MappedTypes注解，完成TypeHandler的注册</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String typeHandlerPackage = child.getStringAttribute(<span class="hljs-string">"name"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; typeHandlerRegistry.register(typeHandlerPackage);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果没有指定package属性，则尝试获取javaType、jdbcType、handler三个属性</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String javaTypeName = child.getStringAttribute(<span class="hljs-string">"javaType"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String jdbcTypeName = child.getStringAttribute(<span class="hljs-string">"jdbcType"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String handlerTypeName = child.getStringAttribute(<span class="hljs-string">"handler"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据属性确定TypeHandler类型以及它能够处理的数据库类型和Java类型</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Class&lt;?&gt; javaTypeClass = resolveClass(javaTypeName);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; JdbcType jdbcType = resolveJdbcType(jdbcTypeName);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Class&lt;?&gt; typeHandlerClass = resolveClass(handlerTypeName);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用TypeHandlerRegistry.register()方法注册TypeHandler</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (javaTypeClass != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (jdbcType == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; typeHandlerRegistry.register(javaTypeClass, typeHandlerClass);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; typeHandlerRegistry.register(javaTypeClass, jdbcType, typeHandlerClass);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; typeHandlerRegistry.register(typeHandlerClass);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="10376">4. 处理<code data-backticks="1" data-nodeid="10595">&lt;plugins&gt;</code>标签</h4>
<p data-nodeid="10377">我们知道 MyBatis 是一个非常易于扩展的持久层框架，而<strong data-nodeid="10602">插件就是 MyBatis 提供的一种重要扩展机制</strong>。</p>
<p data-nodeid="10378">我们可以自定义一个实现了 Interceptor 接口的插件来扩展 MyBatis 的行为，或是拦截 MyBatis 的一些默认行为。插件的工作机制我们会在后面的课时中详细分析，这里我们重点来看 MyBatis 初始化过程中插件配置的加载，也就是 XMLConfigBuilder 中的 pluginElement()方法，该方法的核心就是解析 <code data-backticks="1" data-nodeid="10604">&lt;plugins&gt;</code> 标签中配置的自定义插件，具体实现如下：</p>
<pre class="lang-java" data-nodeid="10379"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">pluginElement</span><span class="hljs-params">(XNode parent)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (parent != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 遍历全部的&lt;plugin&gt;子标签</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (XNode child : parent.getChildren()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取每个&lt;plugin&gt;标签中的interceptor属性</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String interceptor = child.getStringAttribute(<span class="hljs-string">"interceptor"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取&lt;plugin&gt;标签下的其他配置信息</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Properties properties = child.getChildrenAsProperties();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 初始化interceptor属性指定的自定义插件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).getDeclaredConstructor().newInstance();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 初始化插件的配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; interceptorInstance.setProperties(properties);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将Interceptor对象添加到Configuration的插件链中保存，等待后续使用</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configuration.addInterceptor(interceptorInstance);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="10380">5. 处理<code data-backticks="1" data-nodeid="10609">&lt;objectFactory&gt;</code>标签</h4>
<p data-nodeid="10381">在前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6375" data-nodeid="10616">《04 | MyBatis 反射工具箱：带你领略不一样的反射设计思路》</a>中我们提到过，MyBatis 支持自定义 ObjectFactory 实现类和 ObjectWrapperFactory。XMLConfigBuilder 中的 objectFactoryElement() 方法就实现了加载自定义 ObjectFactory 实现类的功能，其核心逻辑就是解析 <code data-backticks="1" data-nodeid="10618">&lt;objectFactory&gt;</code> 标签中配置的自定义 ObjectFactory 实现类，并完成相关的实例化操作，相关的代码实现如下：</p>
<pre class="lang-java" data-nodeid="10382"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">objectFactoryElement</span><span class="hljs-params">(XNode context)</span> <span class="hljs-keyword">throws</span> Exception </span>{
<span class="hljs-keyword">if</span> (context != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; <span class="hljs-comment">// 获取&lt;objectFactory&gt;标签的type属性</span>
&nbsp; &nbsp; String type = context.getStringAttribute(<span class="hljs-string">"type"</span>);
&nbsp; &nbsp; <span class="hljs-comment">// 根据type属性值，初始化自定义的ObjectFactory实现</span>
&nbsp; &nbsp; ObjectFactory factory = (ObjectFactory) resolveClass(type).getDeclaredConstructor().newInstance();
&nbsp; &nbsp; <span class="hljs-comment">// 初始化ObjectFactory对象的配置</span>
&nbsp; &nbsp; Properties properties = context.getChildrenAsProperties();
&nbsp; &nbsp; factory.setProperties(properties);
&nbsp; &nbsp; <span class="hljs-comment">// 将ObjectFactory对象记录到Configuration这个全局配置对象中</span>
&nbsp; &nbsp; configuration.setObjectFactory(factory);
}
</code></pre>
<p data-nodeid="10383">除了 <code data-backticks="1" data-nodeid="10621">&lt;objectFactory&gt;</code> 标签之外，我们还可以通过 <code data-backticks="1" data-nodeid="10623">&lt;objectWrapperFactory&gt;</code> 标签和 <code data-backticks="1" data-nodeid="10625">&lt;reflectorFactory&gt;</code> 标签配置自定义的 ObjectWrapperFactory 实现类和 ReflectorFactory 实现类，这两个标签的解析分别对应 objectWrapperFactoryElement() 方法和 reflectorFactoryElement() 方法，两者实现与 objectFactoryElement() 方法实现类似，这里就不再展示，你若感兴趣的话可以参考<a href="https://github.com/xxxlxy2008/mybatis" data-nodeid="10629">源码</a>进行学习。</p>
<h4 data-nodeid="10384">6. 处理<code data-backticks="1" data-nodeid="10634">&lt;environments&gt;</code>标签</h4>
<p data-nodeid="10385">在 MyBatis 中，我们可以通过 <code data-backticks="1" data-nodeid="10637">&lt;environment&gt;</code> 标签为不同的环境添加不同的配置，例如，线上环境、预上线环境、测试环境等，<strong data-nodeid="10645">每个 <code data-backticks="1" data-nodeid="10641">&lt;environment&gt;</code> 标签只会对应一种特定的环境配置</strong>。</p>
<p data-nodeid="10386">environmentsElement() 方法中实现了 XMLConfigBuilder 处理 <code data-backticks="1" data-nodeid="10647">&lt;environments&gt;</code> 标签的核心逻辑，它会根据 XMLConfigBuilder.environment 字段值，拿到正确的 <code data-backticks="1" data-nodeid="10649">&lt;environment&gt;</code> 标签，然后解析这个环境中使用的 TransactionFactory、DataSource 等核心对象，也就知道了 MyBatis 要请求哪个数据库、如何管理事务等信息。</p>
<p data-nodeid="10387">下面是 environmentsElement() 方法的核心逻辑：</p>
<pre class="lang-java" data-nodeid="10388"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">environmentsElement</span><span class="hljs-params">(XNode context)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (context != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (environment == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 未指定使用的环境id，默认获取default值&nbsp;</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; environment = context.getStringAttribute(<span class="hljs-string">"default"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取&lt;environment&gt;标签下的所有配置</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (XNode child : context.getChildren()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取环境id</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String id = child.getStringAttribute(<span class="hljs-string">"id"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isSpecifiedEnvironment(id)) {&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取&lt;transactionManager&gt;、&lt;dataSource&gt;等标签，并进行解析，其中会根据配置信息初始化相应的TransactionFactory对象和DataSource对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; TransactionFactory txFactory = transactionManagerElement(child.evalNode(<span class="hljs-string">"transactionManager"</span>));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DataSourceFactory dsFactory = dataSourceElement(child.evalNode(<span class="hljs-string">"dataSource"</span>));
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; DataSource dataSource = dsFactory.getDataSource();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建Environment对象，并关联创建好的TransactionFactory和DataSource</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Environment.Builder environmentBuilder = <span class="hljs-keyword">new</span> Environment.Builder(id)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .transactionFactory(txFactory)
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .dataSource(dataSource);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将Environment对象记录到Configuration中，等待后续使用</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configuration.setEnvironment(environmentBuilder.build());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="10389">7. 处理<code data-backticks="1" data-nodeid="10655">&lt;databaseIdProvider&gt;</code>标签</h4>
<p data-nodeid="10390">通过前面课时的介绍可知，在 MyBatis 中编写的都是原生的 SQL 语句，而很多数据库产品都会有一些 SQL 方言，这些方言与标准 SQL 不兼容。</p>
<p data-nodeid="10391">在 mybatis-config.xml 配置文件中，我们可以通过 <code data-backticks="1" data-nodeid="10659">&lt;databaseIdProvider&gt;</code> 标签定义需要支持的全部数据库的 DatabaseId，在后续编写 Mapper 映射配置文件的时候，就可以为同一个业务场景定义不同的 SQL 语句（带有不同的 DataSourceId），来支持不同的数据库，这里就是靠 DatabaseId 来确定哪个 SQL 语句支持哪个数据库的。</p>
<p data-nodeid="10392">databaseIdProviderElement() 方法是 XMLConfigBuilder 处理 <code data-backticks="1" data-nodeid="10662">&lt;databaseIdProvider&gt;</code> 标签的地方，其中的<strong data-nodeid="10668">核心就是获取 DatabaseId 值</strong>，具体实现如下：</p>
<pre class="lang-java" data-nodeid="10393"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">databaseIdProviderElement</span><span class="hljs-params">(XNode context)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; DatabaseIdProvider databaseIdProvider = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (context != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 获取type属性值</span>
&nbsp; &nbsp; &nbsp; &nbsp; String type = context.getStringAttribute(<span class="hljs-string">"type"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-string">"VENDOR"</span>.equals(type)) { <span class="hljs-comment">// 兼容操作</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; type = <span class="hljs-string">"DB_VENDOR"</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 初始化DatabaseIdProvider</span>
&nbsp; &nbsp; &nbsp; &nbsp; Properties properties = context.getChildrenAsProperties();
&nbsp; &nbsp; &nbsp; &nbsp; databaseIdProvider = (DatabaseIdProvider) resolveClass(type).getDeclaredConstructor().newInstance();
&nbsp; &nbsp; &nbsp; &nbsp; databaseIdProvider.setProperties(properties);
&nbsp; &nbsp; }
&nbsp; &nbsp; Environment environment = configuration.getEnvironment();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (environment != <span class="hljs-keyword">null</span> &amp;&amp; databaseIdProvider != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 通过DataSource获取DatabaseId，并保存到Configuration中，等待后续使用</span>
&nbsp; &nbsp; &nbsp; &nbsp; String databaseId = databaseIdProvider.getDatabaseId(environment.getDataSource());
&nbsp; &nbsp; &nbsp; &nbsp; configuration.setDatabaseId(databaseId);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="10394">可以看到，解析<code data-backticks="1" data-nodeid="10670">&lt;databaseIdProvider&gt;</code> 标签之后会得到一个 DatabaseIdProvider 对象，其核心方法是 getDatabaseId() 方法，主要是根据前面解析得到的 DataSource 对象来生成 DatabaseId。DatabaseIdProvider 的继承关系如下图所示：</p>
<p data-nodeid="12431" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/08/89/CioPOWA01E6AM0S_AAFq9Ci2CSc510.png" alt="2021223-1874.png" data-nodeid="12435"></p>
<div data-nodeid="12432"><p style="text-align:center">DatabaseIdProvider 继承关系图</p></div>


<p data-nodeid="10397">从继承关系图中可以看出，DefaultDatabaseIdProvider 是个空实现，而且已被标记为过时了，所以这里我们就重点来看 VendorDatabaseIdProvider 实现。</p>
<p data-nodeid="10398">在 getDatabaseId() 方法中，VendorDatabaseIdProvider 首先会从 DataSource 中拿到数据库的名称，然后根据 <code data-backticks="1" data-nodeid="10677">&lt;databaseIdProvider&gt; </code> 标签配置和 DataSource 返回的数据库名称，确定最终的 DatabaseId 标识，具体实现如下：</p>
<pre class="lang-java" data-nodeid="10399"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getDatabaseId</span><span class="hljs-params">(DataSource dataSource)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略边界检查和异常处理</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> getDatabaseName(dataSource);
}
<span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">getDatabaseName</span><span class="hljs-params">(DataSource dataSource)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 从数据库连接中，获取数据库名称</span>
&nbsp; &nbsp; String productName = getDatabaseProductName(dataSource);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.properties != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据&lt;databaseIdProvider&gt;标签配置，查找自定义数据库名称</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;Object, Object&gt; property : properties.entrySet()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (productName.contains((String) property.getKey())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> (String) property.getValue(); <span class="hljs-comment">// 返回配置的value</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> productName;
}
</code></pre>
<h4 data-nodeid="10400">8. 处理<code data-backticks="1" data-nodeid="10682">&lt;mappers&gt;</code>标签</h4>
<p data-nodeid="10401">除了 mybatis-config.xml 这个全局配置文件之外，MyBatis 初始化的时候还会加载 <code data-backticks="1" data-nodeid="10685">&lt;mappers&gt;</code> 标签下定义的 Mapper 映射文件。<code data-backticks="1" data-nodeid="10687">&lt;mappers&gt;</code> 标签中会指定 Mapper.xml 映射文件的位置，通过解析 <code data-backticks="1" data-nodeid="10689">&lt;mappers&gt; </code> 标签，MyBatis 就能够知道去哪里加载这些 Mapper.xml 文件了。</p>
<p data-nodeid="10402">mapperElement() 方法就是 XMLConfigBuilder 处理 <code data-backticks="1" data-nodeid="10692">&lt;mappers&gt;</code> 标签的具体实现，其中会初始化 XMLMapperBuilder 对象来加载各个 Mapper.xml 映射文件。同时，还会扫描 Mapper 映射文件相应的 Mapper 接口，处理其中的注解并将 Mapper 接口注册到 MapperRegistry 中。</p>
<p data-nodeid="10403">mapperElement() 方法的具体实现如下：</p>
<pre class="lang-java" data-nodeid="10404"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">mapperElement</span><span class="hljs-params">(XNode parent)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (parent != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (XNode child : parent.getChildren()) { <span class="hljs-comment">// 遍历每个子标签</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (<span class="hljs-string">"package"</span>.equals(child.getName())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果指定了&lt;package&gt;子标签，则会扫描指定包内全部Java类型</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String mapperPackage = child.getStringAttribute(<span class="hljs-string">"name"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configuration.addMappers(mapperPackage);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 解析&lt;mapper&gt;子标签，这里会获取resource、url、class三个属性，这三个属性互斥</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String resource = child.getStringAttribute(<span class="hljs-string">"resource"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String url = child.getStringAttribute(<span class="hljs-string">"url"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String mapperClass = child.getStringAttribute(<span class="hljs-string">"class"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果&lt;mapper&gt;子标签指定了resource或是url属性，都会创建XMLMapperBuilder对象，</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 然后使用这个XMLMapperBuilder实例解析指定的Mapper.xml配置文件</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (resource != <span class="hljs-keyword">null</span> &amp;&amp; url == <span class="hljs-keyword">null</span> &amp;&amp; mapperClass == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ErrorContext.instance().resource(resource);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; InputStream inputStream = Resources.getResourceAsStream(resource);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; XMLMapperBuilder mapperParser = <span class="hljs-keyword">new</span> XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mapperParser.parse();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (resource == <span class="hljs-keyword">null</span> &amp;&amp; url != <span class="hljs-keyword">null</span> &amp;&amp; mapperClass == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ErrorContext.instance().resource(url);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; InputStream inputStream = Resources.getUrlAsStream(url);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; XMLMapperBuilder mapperParser = <span class="hljs-keyword">new</span> XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; mapperParser.parse();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (resource == <span class="hljs-keyword">null</span> &amp;&amp; url == <span class="hljs-keyword">null</span> &amp;&amp; mapperClass != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 如果&lt;mapper&gt;子标签指定了class属性，则向MapperRegistry注册class属性指定的Mapper接口</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Class&lt;?&gt; mapperInterface = Resources.classForName(mapperClass);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; configuration.addMapper(mapperInterface);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BuilderException(<span class="hljs-string">"A mapper element may only specify a url, resource or class, but not more than one."</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<h3 data-nodeid="10405">总结</h3>
<p data-nodeid="10406">这一讲我们重点介绍了 MyBatis 初始化过程中对 mybatis-config.xml 全局配置文件的解析，深入分析了 mybatis-config.xml 配置文件中所有标签的解析流程，让你进一步了解这些配置加载的原理。同时，我们还介绍了构造者模式这一经典设计模式，它是整个 MyBatis 初始化逻辑的基础思想。</p>
<p data-nodeid="10407">关于这一讲的内容，若你有什么问题，期待在留言区与我分享和交流。</p>
<p data-nodeid="10408">在下一讲，我们将继续介绍 MyBatis 初始化流程的内容，重点讲解 Mapper.xml 配置文件的加载以及 SQL 语句的相关处理，记得按时来听课。</p>
<hr data-nodeid="10409">
<p data-nodeid="10410"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="10703"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="10702"></a></p>
<p data-nodeid="10411"><strong data-nodeid="10707">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="10412" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="10711">点击链接，提升自己</a>！</p>

---

### 精选评论

##### **辉：
> 老师，那注解也是这个逻辑么？@select

##### **生：
> 老师我想问一下，mybatisplus select (@i:=@i+1) as rowNum, pma.* from ()pma ,(select @i:=0) as it 在mapper里面报错

##### **举：
> 牛呀牛呀~😊

