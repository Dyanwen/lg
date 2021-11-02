<p>通过此前 8 个课时的学习，相信你已经了解了 SkyWalking Agent 是通过 Java Agent 的方式随应用程序一起启动，然后通过 Byte Buddy 库动态插入埋点收集 Trace 信息。从本课时开始，我会带你深入研究 SkyWalking Agent 的架构、原理以及具体实现，还将深入分析 Tomcat、Dubbo、MySQL 等常用的插件。</p>
<h3>微内核架构</h3>
<p>SkyWalking Agent 采用了微内核架构（Microkernel Architecture），那什么是微内核架构呢？微内核架构也被称为插件化架构（Plug-in Architecture），是一种面向功能进行拆分的可扩展性架构。在基于产品的应用中通常会使用微内核架构，例如，IDEA、Eclipse 这类 IDE 开发工具，内核都是非常精简的，对 Maven、Gradle 等新功能的支持都是以插件的形式增加的。</p>
<p>如下图所示，微内核架构分为核心系统和插件模块两大部分。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/08/53/Ciqah16FsIuAY29rAADXS1mP1qk235.png" alt=""></p>
<p>在上图展示的微内核架构中，内核功能是比较稳定的，只负责管理插件的生命周期，不会因为系统功能的扩展而不断进行修改。功能上的扩展全部封装到插件之中，插件模块是独立存在的模块，包含特定的功能，能拓展核心系统的功能。通常，不同的插件模块互相之间独立，当然，你可以设计成一个插件依赖于另外一个插件，但应尽量让插件之间的相互依赖关系降低到最小，避免繁杂的依赖带来扩展性问题。</p>
<p>最终所有插件会由内核系统统一接入和管理：</p>
<ul>
<li>首先，内核系统必须知道要加载哪些插件，一般会通过配置文件或是扫描 ClassPath 的方式（例如前文介绍的 SPI 技术）确定待加载的插件；</li>
<li>之后，内核系统还需要了解如何使用这些插件，微内核架构中需要定义一套插件的规范，内核系统会按照统一的方式初始化、启动这些插件；</li>
<li>最后，虽然插件之间完全解耦，但实际开发中总会有一些意想不到的需求会导致插件之间产生依赖或是某些底层插件被复用，此时内核需要提供一套规则，识别插件消息并能正确的在插件之间转发消息，成为插件消息的中转站。</li>
</ul>
<p>由此可见微内核架构的好处：</p>
<ul>
<li>测试成本下降。从软件工程的角度看，微内核架构将变化的部分和不变的部分拆分，降低了测试的成本，符合设计模式中的开放封闭原则。</li>
<li>稳定性。由于每个插件模块相对独立，即使其中一个插件有问题，也可以保证内核系统以及其他插件的稳定性。</li>
<li>可扩展性。在增加新功能或接入新业务的时候，只需要新增相应插件模块即可；在进行历史功能下线时，也只需删除相应插件模块即可。</li>
</ul>
<p>SkyWalking Agent 就是微内核架构的一种落地方式。在前面的课时中我已经介绍了 SkyWalking 中各个模块的功能，其中 apm-agent-core 模块对应微内核架构中的内核系统，apm-sdk-plugin 模块中的各个子模块都是微内核架构中的插件模块。</p>
<h3>SkyWalking Agent 启动流程概述</h3>
<p>此前，在搭建 SkyWalking 源码环境的最后，我们尝试 Debug 了一下 SkyWalking Agent 的源码，其入口是 apm-agent 模块中 SkyWalkingAgent 类的 premain() 方法，其中完成了 Agent 启动的流程：</p>
<ol>
<li>初始化配置信息。该步骤中会加载 agent.config 配置文件，其中会检测 Java Agent 参数以及环境变量是否覆盖了相应配置项。</li>
<li>查找并解析 skywalking-plugin.def 插件文件。</li>
<li>AgentClassLoader 加载插件。</li>
<li>PluginFinder 对插件进行分类管理。</li>
<li>使用 Byte Buddy 库创建 AgentBuilder。这里会根据已加载的插件动态增强目标类，插入埋点逻辑。</li>
<li>使用 JDK SPI 加载并启动 BootService 服务。BootService 接口的实现会在后面的课时中展开详细介绍。</li>
<li>添加一个 JVM 钩子，在 JVM 退出时关闭所有 BootService 服务。</li>
</ol>
<p>SkywalkingAgent.premain() 方法的具体实现如下，其中省略了 try/catch 代码块以及异常处理逻辑：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">premain</span><span class="hljs-params">(String agentArgs, 
 &nbsp; &nbsp; &nbsp; Instrumentation instrumentation)</span> <span class="hljs-keyword">throws</span> PluginException </span>{
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤1、初始化配置信息</span>
 &nbsp; &nbsp;SnifferConfigInitializer.initialize(agentArgs); 
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤2~4、查找并解析skywalking-plugin.def插件文件；</span>
 &nbsp; &nbsp;<span class="hljs-comment">// AgentClassLoader加载插件类并进行实例化；PluginFinder提供插件匹配的功能</span>
 &nbsp; &nbsp;<span class="hljs-keyword">final</span> PluginFinder pluginFinder = <span class="hljs-keyword">new</span> PluginFinder(
 &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> PluginBootstrap().loadPlugins());
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤5、使用 Byte Buddy 库创建 AgentBuilder</span>
 &nbsp; &nbsp;<span class="hljs-keyword">final</span> ByteBuddy byteBuddy = <span class="hljs-keyword">new</span> ByteBuddy()
 &nbsp; &nbsp; &nbsp; .with(TypeValidation.of(Config.Agent.IS_OPEN_DEBUGGING_CLASS));
 &nbsp; &nbsp;<span class="hljs-keyword">new</span> AgentBuilder.Default(byteBuddy)...installOn(instrumentation);
 &nbsp; &nbsp;<span class="hljs-comment">// 这里省略创建 AgentBuilder的具体代码，后面展开详细说</span>
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤6、使用 JDK SPI加载的方式并启动 BootService 服务。</span>
 &nbsp; &nbsp;ServiceManager.INSTANCE.boot();
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤7、添加一个JVM钩子</span>
 &nbsp; &nbsp;Runtime.getRuntime().addShutdownHook(<span class="hljs-keyword">new</span> Thread(<span class="hljs-keyword">new</span> Runnable() {
 &nbsp; &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{ ServiceManager.INSTANCE.shutdown(); }
 &nbsp; &nbsp;}, <span class="hljs-string">"skywalking service shutdown thread"</span>));
}
</code></pre>
<p>了解了 SkyWalking Agent 启动的核心步骤之后，本课时剩余部分将对每个步骤进行深入分析。</p>
<h3>初始化配置</h3>
<p>在启动 demo-webapp 和 demo-provider 两个 demo 应用的时候，需要在 VM options 中指定 agent.confg 配置文件（skywalking_config 参数），agent.config 配置文件中的配置项如下：</p>
<pre><code data-language="java" class="lang-java"># 当前应用的服务名称，通过Skywalking Agent上报的Metrics、Trace数据都会
# 携带该信息进行标识
agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}
</code></pre>
<p>在 &nbsp;SnifferConfigInitializer.initialize() 方法中会将最终的配置信息填充到 Config 的静态字段中，填充过程如下：</p>
<ol>
<li>将 agent.config 文件中全部配置信息填充到 Config 中相应的静态字段中。</li>
<li>解析系统环境变量值，覆盖 Config 中相应的静态字段。</li>
<li>解析 Java Agent 的参数，覆盖 Config 中相应的静态字段。</li>
</ol>
<p>SnifferConfigInitializer.initialize() 方法的具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">initialize</span><span class="hljs-params">(String agentOptions)</span> </span>{
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤1、加载 agent.config配置文件</span>
 &nbsp; &nbsp;InputStreamReader configFileStream = loadConfig();
 &nbsp; &nbsp;Properties properties = <span class="hljs-keyword">new</span> Properties();
 &nbsp; &nbsp;properties.load(configFileStream);
 &nbsp; &nbsp;<span class="hljs-keyword">for</span> (String key : properties.stringPropertyNames()) {
 &nbsp; &nbsp; &nbsp; &nbsp;String value = (String)properties.get(key);
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 按照${配置项名称:默认值}的格式解析各个配置项</span>
 &nbsp; &nbsp; &nbsp; &nbsp;properties.put(key, PropertyPlaceholderHelper.INSTANCE
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;.replacePlaceholders(value, properties));
 &nbsp; &nbsp;}
 &nbsp; &nbsp;<span class="hljs-comment">// 填充 Config中的静态字段</span>
 &nbsp; &nbsp;ConfigInitializer.initialize(properties, Config<span class="hljs-class">.<span class="hljs-keyword">class</span>)</span>;
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤2、解析环境变量，并覆盖 Config中相应的静态字段</span>
 &nbsp; &nbsp;overrideConfigBySystemProp();
 &nbsp; &nbsp;<span class="hljs-comment">// 步骤3、解析 Java Agent参数，并覆盖 Config中相应的静态字段</span>
 &nbsp; &nbsp;overrideConfigByAgentOptions(agentOptions);
 &nbsp; &nbsp;<span class="hljs-comment">// 检测SERVICE_NAME和BACKEND_SERVICE两个配置项，若为空则抛异常(略)</span>
 &nbsp; &nbsp;IS_INIT_COMPLETED = <span class="hljs-keyword">true</span>; <span class="hljs-comment">// 更新初始化标记</span>
}
</code></pre>
<p>步骤 1 中的 loadConfig() 方法会优先根据环境变量（skywalking_config）指定的 agent.config 文件路径加载。若环境变量未指定 skywalking_ config 配置，则到 skywalking-agent.jar 同级的 config 目录下查找 agent.confg 配置文件。</p>
<p>将 agent.config 文件中的配置信息加载到 Properties 对象之后，将使用 PropertyPlaceholderHelper 对配置信息进行解析，将当前的“${配置项名称:默认值}”格式的配置值，替换成其中的默认值，demo-provider 解析结果如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/81/69/Cgq2xl6FsIuAD9SEAAXN2BldzCw554.png" alt=""></p>
<p>完成解析之后，会通过 ConfigInitializer 工具类，将配置信息填充到 Config 中的静态字段中，具体填充规则如下：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/08/53/Ciqah16FsIuAMuWAAAGqpFKOkno592.png" alt=""></p>
<p>在接下来的 overrideConfigBySystemProp() 方法中会遍历环境变量（即 System.getProperties() 集合），如果环境变 是以 "skywalking." 开头的，则认为是 SkyWalking 的配置，同样会填充到 Config 类中，以覆盖 agent.config 中的默认值。</p>
<p>最后的 overrideConfigByAgentOptions() 方法解析的是 Java Agent 的参数，填充 Config 类的规则与前面两步相同，不再重复。</p>
<p>到此为止，SkyWalking Agent 启动所需的全部配置都已经填充到 Config 中，后续使用配置信息时直接访问 Config 中的相应静态字段即可。</p>
<h3>插件加载原理</h3>
<p>完成 Config 类的初始化之后，SkyWalking Agent 开始扫描指定目录下的 SkyWalking Agent 插件 jar 包并进行加载。</p>
<h4>AgentClassLoader</h4>
<p>SkyWalking Agent 加载插件时使用到一个自定义的 ClassLoader —— AgentClassLoader，之所以自定义类加载器，目的是不在应用的 Classpath 中引入 SkyWalking 的插件 jar 包，这样就可以让应用无依赖、无感知的插件。</p>
<h4>并行加载优化</h4>
<p>AgentClassLoader 的静态代码块中会调动 tryRegisterAsParallelCapable() 方法，其中会通过反射方式尝试开启 JDK 的并行加载功能：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">tryRegisterAsParallelCapable</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp;Method[] methods = ClassLoader<span class="hljs-class">.<span class="hljs-keyword">class</span>.<span class="hljs-title">getDeclaredMethods</span>()</span>;
 &nbsp; &nbsp;<span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; methods.length; i++) {
 &nbsp; &nbsp; &nbsp; &nbsp;Method method = methods[i];
 &nbsp; &nbsp; &nbsp; &nbsp;String methodName = method.getName();
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 查找 ClassLoader中的registerAsParallelCapable()静态方法</span>
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> (<span class="hljs-string">"registerAsParallelCapable"</span>.equalsIgnoreCase(methodName)) 
 &nbsp; &nbsp; &nbsp; &nbsp;{
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;method.setAccessible(<span class="hljs-keyword">true</span>);
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;method.invoke(<span class="hljs-keyword">null</span>); <span class="hljs-comment">// 调用registerAsParallelCapable()方法</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span>;
 &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp;}
}
</code></pre>
<p>在使用 ClassLoader 加载一个类的时候，JVM 会进行加锁同步，这也是我们能够利用类加载机制实现单例的原因。在 Java 6 中，ClassLoader.loadClass() 方法是用 synchronized 加锁同步的，需要全局竞争一把锁，效率略低。</p>
<p>在 Java 7 之后提供了两种加锁模式：</p>
<ul>
<li>串行模式下，锁的对象是还是 ClassLoader 本身，和 Java 6 里面的行为一样；</li>
<li>另外一种就是调用&nbsp;registerAsParallelCapable() 方法之后，开启的并行加载模式。在并行模式下加载类时，会按照 classname 去获取锁。ClassLoader.loadClass() 方法中相应的实现片段如下：</li>
</ul>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">protected</span> Class&lt;?&gt; loadClass(String name, <span class="hljs-keyword">boolean</span> resolve)
 &nbsp; &nbsp;<span class="hljs-keyword">throws</span> ClassNotFoundException{
 &nbsp; &nbsp;<span class="hljs-comment">// getClassLoadingLock() 方法会返回加锁的对象</span>
 &nbsp; &nbsp;<span class="hljs-keyword">synchronized</span> (getClassLoadingLock(name)) { 
 &nbsp; &nbsp; &nbsp; ... ... <span class="hljs-comment">// 加载指定类，具体加载细节不展开介绍</span>
 &nbsp; &nbsp;}
}


<span class="hljs-function"><span class="hljs-keyword">protected</span> Object <span class="hljs-title">getClassLoadingLock</span><span class="hljs-params">(String className)</span> </span>{
 &nbsp; &nbsp;Object lock = <span class="hljs-keyword">this</span>;
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> (parallelLockMap != <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 检测是否开启了并行加载功能</span>
 &nbsp; &nbsp; &nbsp; &nbsp;Object newLock = <span class="hljs-keyword">new</span> Object();
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 若开启了并行加载，则一个className对应一把锁；否则还是只</span>
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 对当前ClassLoader进行加锁</span>
 &nbsp; &nbsp; &nbsp; &nbsp;lock = parallelLockMap.putIfAbsent(className, newLock);
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> (lock == <span class="hljs-keyword">null</span>) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;lock = newLock;
 &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp;}
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> lock;
}
</code></pre>
<h4>AgentClassLoader 核心实现</h4>
<p>在 AgentClassLoader 的构造方法中会初始化其 classpath 字段，该字段指向了 AgentClassLoader 要扫描的目录（skywalking-agent.jar 包同级别的 plugins 目录和 activations 目录），如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span> List&lt;File&gt; classpath; 

<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">AgentClassLoader</span><span class="hljs-params">(ClassLoader parent)</span> </span>{
 &nbsp; &nbsp;<span class="hljs-keyword">super</span>(parent); <span class="hljs-comment">// 双亲委派机制</span>
 &nbsp; &nbsp;<span class="hljs-comment">// 获取 skywalking-agent.jar所在的目录</span>
 &nbsp; &nbsp;File agentDictionary = AgentPackagePath.getPath();
 &nbsp; &nbsp;classpath = <span class="hljs-keyword">new</span> LinkedList&lt;File&gt;();
 &nbsp; &nbsp;<span class="hljs-comment">// 初始化 classpath集合，指向了skywalking-agent.jar包同目录的两个目录</span>
 &nbsp; &nbsp;classpath.add(<span class="hljs-keyword">new</span> File(agentDictionary, <span class="hljs-string">"plugins"</span>));
 &nbsp; &nbsp;classpath.add(<span class="hljs-keyword">new</span> File(agentDictionary, <span class="hljs-string">"activations"</span>));
}
</code></pre>
<p>AgentClassLoader 作为一个类加载器，主要工作还是从其 Classpath 下加载类（或资源文件），对应的就是其 findClass() 方法和 findResource() 方法，这里简单看一下 findClass() 方法的实现：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 在下面的getAllJars()方法中会扫描全部jar文件，并缓存到</span>
<span class="hljs-comment">// allJars字段(List&lt;Jar&gt;类型)中，后续再次扫描时会重用该缓</span>
<span class="hljs-keyword">private</span> List&lt;Jar&gt; allJars;

<span class="hljs-keyword">protected</span> Class&lt;?&gt; findClass(String name) {
 &nbsp; &nbsp;List&lt;Jar&gt; allJars = getAllJars(); &nbsp;<span class="hljs-comment">// 扫描过程比较简单，不再展开介绍</span>
 &nbsp; &nbsp;String path = name.replace(<span class="hljs-string">'.'</span>, <span class="hljs-string">'/'</span>).concat(<span class="hljs-string">".class"</span>);
 &nbsp; &nbsp;<span class="hljs-keyword">for</span> (Jar jar : allJars) { <span class="hljs-comment">// 扫描所有jar包，查找类文件</span>
 &nbsp; &nbsp; &nbsp; &nbsp;JarEntry entry = jar.jarFile.getJarEntry(path);
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> (entry != <span class="hljs-keyword">null</span>) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;URL classFileUrl = <span class="hljs-keyword">new</span> URL(<span class="hljs-string">"jar:file:"</span> + 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;jar.sourceFile.getAbsolutePath() + <span class="hljs-string">"!/"</span> + path);
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">byte</span>[] data = ...;<span class="hljs-comment">// 省略读取".class"文件的逻辑 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;// 加载类文件内容，创建相应的Class对象</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> defineClass(name, data, <span class="hljs-number">0</span>, data.length);
 &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp;} <span class="hljs-comment">// 类查找失败，直接抛出异常</span>
 &nbsp; &nbsp;<span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ClassNotFoundException(<span class="hljs-string">"Can't find "</span> + name);
}
</code></pre>
<p>findResource() 方法会遍历 allJars 集合缓存的全部 jar 包，从中查找指定的资源文件并返回，遍历逻辑与 findClass() 方法类似，不再展开分析。</p>
<p>最后，AgentClassLoader 中有一个 DEFAULT_LOADER 静态字段，记录了 默认的 AgentClassLoader，如下所示，但是注意，AgentClassLoader 并不是单例，后面会看到其他创建 AgentClassLoader 的地方。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">private</span>&nbsp;<span class="hljs-keyword">static</span>&nbsp;AgentClassLoader&nbsp;DEFAULT_LOADER;
</code></pre>
<h3>解析插件定义</h3>
<p>每个 Agent 插件中都会定义一个 skywalking-plugin.def 文件，如下图&nbsp;tomcat-7.x-8.x-plugin 插件所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/81/69/Cgq2xl6FsIyAD1i3AAAs9sJ1s2Y631.png" alt=""></p>
<p>tomcat-7.x-8.x-plugin 插件中 skywalking-plugin.def 文件的内容如下，其中每一行都是一个插件类的定义：</p>
<pre><code data-language="java" class="lang-java">tomcat-<span class="hljs-number">7</span>.x/<span class="hljs-number">8</span>.x=org.apache.skywalking.apm.plugin.tomcat78x.define 
.TomcatInstrumentation

tomcat-<span class="hljs-number">7</span>.x/<span class="hljs-number">8</span>.x=org.apache.skywalking.apm.plugin.tomcat78x.define
.ApplicationDispatcherInstrumentation
</code></pre>
<p>PluginResourcesResolver 是 Agent 插件的资源解析器，会通过 AgentClassLoader 中的 findResource() 方法读取所有 Agent 插件中的 skywalking-plugin.def 文件。</p>
<h3>AbstractClassEnhancePluginDefine</h3>
<p>拿到全部插件的 skywalking-plugin.def 文件之后，PluginCfg 会逐行进行解析，转换成 PluginDefine 对象。PluginDefine 中有两个字段：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 插件名称，以 tomcat-7.x-8.x-plugin 插件第一行为例，就是tomcat-7.x/8.x</span>
<span class="hljs-keyword">private</span> String name; 
<span class="hljs-comment">// 插件类，对应上例中的 org.apache.skywalking.apm.plugin.tomcat78x.define</span>
<span class="hljs-comment">// .TomcatInstrumentation</span>
<span class="hljs-keyword">private</span> String defineClass;
</code></pre>
<p>PluginCfg 是通过枚举实现的、单例的工具类，逻辑非常简单，不再展开介绍。</p>
<p>接下来会遍历全部 PluginDefine 对象，通过反射将其中 defineClass 字段中记录的插件类实例化，核心逻辑如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">for</span> (PluginDefine pluginDefine : pluginClassList) {
 &nbsp; &nbsp;<span class="hljs-comment">// 注意，这里使用类加载器是默认的AgentClassLoader实例</span>
 &nbsp; &nbsp;AbstractClassEnhancePluginDefine plugin =
 &nbsp; &nbsp; &nbsp; &nbsp;(AbstractClassEnhancePluginDefine)
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;Class.forName(pluginDefine.getDefineClass(), <span class="hljs-keyword">true</span>,
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;AgentClassLoader.getDefault()).newInstance();
 &nbsp; &nbsp;plugins.add(plugin); <span class="hljs-comment">// 记录AbstractClassEnhancePluginDefine 对象</span>
}
</code></pre>
<p>AbstractClassEnhancePluginDefine 抽象类是所有 Agent 插件类的顶级父类，其中定义了四个核心方法，决定了一个插件类应该增强哪些目标类、应该如何增强、具体插入哪些逻辑，如下所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/81/69/Cgq2xl6FsIyAT0wmAAIua7pnGBg062.png" alt=""></p>
<ul>
<li><strong>enhanceClass() 方法</strong>：返回的 ClassMatch，用于匹配当前插件要增强的目标类。</li>
<li><strong>define() 方法</strong>：插件类增强逻辑的入口，底层会调用下面的 enhance() 方法和 witnessClass() 方法。</li>
<li><strong>enhance() 方法</strong>：真正执行增强逻辑的地方。</li>
<li><strong>witnessClass() 方法</strong>：一个开源组件可能有多个版本，插件会通过该方法识别组件的不同版本，防止对不兼容的版本进行增强。</li>
</ul>
<p>在后续的课时中会详细介绍每个方法的具体功能和实现，你先知道 AbstractClassEnhancePluginDefine 中大致有这四个方法即可。</p>
<h4>ClassMatch</h4>
<p>enhanceClass() 方法决定了一个插件类要增强的目标类，返回值为 ClassMatch 类型对象。ClassMatch 类似于一个过滤器，可以通过多种方式匹配到目标类，ClassMatch 接口的实现如下：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/81/69/Cgq2xl6FsIyAbzszAAFwI9x3bVc197.png" alt=""></p>
<ul>
<li>**NameMatch：**根据其 className 字段（String 类型）匹配目标类的名称。</li>
<li>**IndirectMatch：**子接口中定义了两个方法。</li>
</ul>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// Junction是Byte Buddy中的类，可以通过and、or等操作串联多个ElementMatcher</span>
<span class="hljs-comment">// 进行匹配</span>
ElementMatcher.<span class="hljs-function">Junction <span class="hljs-title">buildJunction</span><span class="hljs-params">()</span></span>; 
<span class="hljs-comment">// 用于检测传入的类型是否匹配该Match</span>
<span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isMatch</span><span class="hljs-params">(TypeDescription typeDescription)</span></span>;
</code></pre>
<ul>
<li><strong>MultiClassNameMatch</strong>：其中会指定一个 matchClassNames 集合，该集合内的类即为目标类。</li>
<li><strong>ClassAnnotationMatch</strong>：根据标注在类上的注解匹配目标类。</li>
<li><strong>MethodAnnotationMatch</strong>：根据标注在方法上的注解匹配目标类。</li>
<li><strong>HierarchyMatch</strong>：根据父类或是接口匹配目标类。</li>
</ul>
<p>这里以 ClassAnnotationMatch 为例展开分析，其中的 annotations 字段（String[] 类型）指定了该 ClassAnnotationMatch 对象需要检查的注解。在 buildJunction() 方法中将为每一个注解创建相应的 Junction 并将它们以 and 形式连接起来并返回，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> ElementMatcher.<span class="hljs-function">Junction <span class="hljs-title">buildJunction</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp;ElementMatcher.Junction junction = <span class="hljs-keyword">null</span>;
 &nbsp; &nbsp;<span class="hljs-keyword">for</span> (String annotation : annotations) { <span class="hljs-comment">// 遍历全部注解</span>
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> (junction == <span class="hljs-keyword">null</span>) { 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 该Junction用于检测类是否标注了指定注解</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;junction = buildEachAnnotation(annotation);
 &nbsp; &nbsp; &nbsp; &nbsp;} <span class="hljs-keyword">else</span> {<span class="hljs-comment">// 使用 and 方式将所有Junction对象连接起来</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;junction = junction.and(buildEachAnnotation(annotation));
 &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp;}
 &nbsp; &nbsp;junction = junction.and(not(isInterface())); <span class="hljs-comment">// 排除接口</span>
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> junction;
}
</code></pre>
<p>isMatch() 方法的实现类似，只有包含所有指定注解的类，才能匹配成功，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isMatch</span><span class="hljs-params">(TypeDescription typeDescription)</span> </span>{
 &nbsp; &nbsp;List&lt;String&gt; annotationList = 
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">new</span> ArrayList&lt;String&gt;(Arrays.asList(annotations));
 &nbsp; &nbsp;<span class="hljs-comment">// 获取该类上的注解</span>
 &nbsp; &nbsp;AnnotationList declaredAnnotations = 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;typeDescription.getDeclaredAnnotations();
 &nbsp; &nbsp;<span class="hljs-comment">// 匹配一个删除一个</span>
 &nbsp; &nbsp;<span class="hljs-keyword">for</span> (AnnotationDescription annotation : declaredAnnotations) {
 &nbsp; &nbsp; &nbsp; &nbsp;annotationList.remove(annotation
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;.getAnnotationType().getActualName());
 &nbsp; &nbsp;}
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> (annotationList.isEmpty()) { <span class="hljs-comment">// 删空了，就匹配成功了</span>
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
 &nbsp; &nbsp;}
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
}
</code></pre>
<p>其他 ClassMatch 接口的实现原理类似，不再展开分析，如果你感兴趣可以看一下代码。</p>
<h3>PluginFinder</h3>
<p>PluginFinder 是 AbstractClassEnhancePluginDefine&nbsp;查找器，可以根据给定的类查找用于增强的 AbstractClassEnhancePluginDefine 集合。</p>
<p>在 PluginFinder&nbsp;的构造函数中会遍历前面课程已经实例化的 AbstractClassEnhancePluginDefine ，并根据&nbsp;enhanceClass() 方法返回的 ClassMatcher 类型进行分类，得到如下两个集合：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 如果返回值为NameMatch类型，则相应 AbstractClassEnhancePluginDefine </span>
<span class="hljs-comment">// 对象会记录到该集合</span>
<span class="hljs-keyword">private</span> Map&lt;String, LinkedList&lt;AbstractClassEnhancePluginDefine&gt;&gt; nameMatchDefine;

<span class="hljs-comment">// 如果是其他类型返回值，则相应 AbstractClassEnhancePluginDefine </span>
<span class="hljs-comment">// 对象会记录到该集合</span>
<span class="hljs-keyword">private</span> List&lt;AbstractClassEnhancePluginDefine&gt; signatureMatchDefine;
</code></pre>
<p>find() 方法是 PluginFinder 对外暴露的查询方法，其中会先后遍历 nameMatchDefine 集合和 signatureMatchDefine 集合，通过 ClassMatch.isMatch() 方法确定所有的匹配插件。find() 方法的实现并不复杂，不再展开介绍。</p>
<h3>AgentBuilder</h3>
<p>前面已经分析了 Skywalking Agent 启动过程中加载配置信息、初始化 Config 类、查找 skywalking-pluing.def 文件、初始化 AbstractClassEnhancePluginDefine 对象等步骤。现在开始介绍 Byte Buddy 如何使用加载到的插件类增强目标方法。</p>
<p>在 SkywalkingAgent.premain() 方法中的步骤 5 中，首先会创建 ByteBuddy 对象，正如前面 Byte Buddy 基础课时中提到的，它是 Byte Buddy 的基础对象之一：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-comment">// 步骤5、通过Byte Buddy API创建Agent</span>
<span class="hljs-keyword">final</span> ByteBuddy byteBuddy = <span class="hljs-keyword">new</span> ByteBuddy()
 &nbsp; .with(TypeValidation.of(Config.Agent.IS_OPEN_DEBUGGING_CLASS));
</code></pre>
<p>Config.Agent.IS_OPEN_DEBUGGING_CLASS&nbsp;在&nbsp;agent.config&nbsp;文件中对应的配置项是：</p>
<pre><code data-language="java" class="lang-java">agent.is_open_debugging_class
</code></pre>
<p>如果将其配置为&nbsp;true，则会将动态生成的类输出到&nbsp;debugging&nbsp;目录中。</p>
<p>接下来创建 AgentBuilder 对象，AgentBuilder 是 Byte Buddy 库专门用来支持 Java Agent 的一个 API，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">new</span> AgentBuilder.Default(byteBuddy) <span class="hljs-comment">// 设置使用的ByteBuddy对象</span>
.ignore(nameStartsWith(<span class="hljs-string">"net.bytebuddy."</span>)<span class="hljs-comment">// 不会拦截下列包中的类</span>
 &nbsp; &nbsp; &nbsp; .or(nameStartsWith(<span class="hljs-string">"org.slf4j."</span>))
 &nbsp; &nbsp; &nbsp; .or(nameStartsWith(<span class="hljs-string">"org.apache.logging."</span>))
 &nbsp; &nbsp; &nbsp; .or(nameStartsWith(<span class="hljs-string">"org.groovy."</span>))
 &nbsp; &nbsp; &nbsp; .or(nameContains(<span class="hljs-string">"javassist"</span>))
 &nbsp; &nbsp; &nbsp; .or(nameContains(<span class="hljs-string">".asm."</span>))
 &nbsp; &nbsp; &nbsp; .or(nameStartsWith(<span class="hljs-string">"sun.reflect"</span>))
 &nbsp; &nbsp; &nbsp; .or(allSkyWalkingAgentExcludeToolkit()) <span class="hljs-comment">// 处理 Skywalking 的类</span>
 &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// synthetic类和方法是由编译器生成的，这种类也需要忽略</span>
 &nbsp; &nbsp; &nbsp; .or(ElementMatchers.&lt;TypeDescription&gt;isSynthetic()))
.type(pluginFinder.buildMatch())<span class="hljs-comment">// 拦截</span>
.transform(<span class="hljs-keyword">new</span> Transformer(pluginFinder)) <span class="hljs-comment">// 设置Transform</span>
.with(<span class="hljs-keyword">new</span> Listener()) <span class="hljs-comment">// 设置Listener</span>
.installOn(instrumentation)
</code></pre>
<p>简单解释一下这里使用到的 AgentBuilder 的方法：</p>
<ul>
<li><strong>ignore() 方法</strong>：忽略指定包中的类，对这些类不会进行拦截增强。</li>
<li><strong>type() 方法</strong>：在类加载时根据传入的 ElementMatcher 进行拦截，拦截到的目标类将会被 transform() 方法中指定的 Transformer 进行增强。</li>
<li><strong>transform() 方法</strong>：这里指定的 Transformer 会对前面拦截到的类进行增强。</li>
<li><strong>with() 方法</strong>：添加一个 Listener 用来监听 AgentBuilder 触发的事件。</li>
</ul>
<p>首先， PluginFInder.buildMatch() 方法返回的 ElementMatcher 对象会将全部插件的匹配规则（即插件的 enhanceClass() 方法返回的 ClassMatch）用 OR 的方式连接起来，这样，所有插件能匹配到的所有类都会交给 Transformer 处理。</p>
<p>再来看 with() 方法中添加的监听器 —— SkywalkingAgent.Listener，它继承了 AgentBuilder.Listener 接口，当监听到 Transformation 事件时，会根据 IS_OPEN_DEBUGGING_CLASS 配置决定是否将增强之后的类持久化成 class 文件保存到指定的 log 目录中。注意，该操作是需要加锁的，会影响系统的性能，一般只在测试环境中开启，在生产环境中不会开启。</p>
<p>最后来看 Skywalking.Transformer，它实现了 AgentBuilder.Transformer 接口，其 transform() 方法是插件增强目标类的入口。Skywalking.Transformer 会通过 PluginFinder 查找目标类匹配的插件（即 AbstractClassEnhancePluginDefine 对象），然后交由 AbstractClassEnhancePluginDefine 完成增强，核心实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> DynamicType.Builder&lt;?&gt; transform(DynamicType.Builder&lt;?&gt;builder,
 &nbsp; &nbsp;TypeDescription typeDescription, <span class="hljs-comment">// 被拦截的目标类</span>
 &nbsp; &nbsp;ClassLoader classLoader, &nbsp;<span class="hljs-comment">// 加载目标类的ClassLoader</span>
 &nbsp; &nbsp;JavaModule <span class="hljs-keyword">module</span>) {
 &nbsp; &nbsp;<span class="hljs-comment">// 从PluginFinder中查找匹配该目标类的插件，PluginFinder的查找逻辑不再重复</span>
 &nbsp; &nbsp;List&lt;AbstractClassEnhancePluginDefine&gt; pluginDefines =
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; pluginFinder.find(typeDescription);
 &nbsp; &nbsp;<span class="hljs-keyword">if</span> (pluginDefines.size() &gt;<span class="hljs-number">0</span>){ 
 &nbsp; &nbsp; &nbsp; &nbsp;DynamicType.Builder&lt;?&gt;newBuilder = builder;
 &nbsp; &nbsp; &nbsp; &nbsp;EnhanceContext context = <span class="hljs-keyword">new</span> EnhanceContext();
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">for</span> (AbstractClassEnhancePluginDefinedefine : pluginDefines) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// AbstractClassEnhancePluginDefine.define()方法是插件入口，</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 在其中完成了对目标类的增强</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DynamicType.Builder&lt;?&gt;possibleNewBuilder = 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; define.define(typeDescription, 
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;newBuilder, classLoader,context);
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">if</span> (possibleNewBuilder != <span class="hljs-keyword">null</span>) {
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 注意这里，如果匹配了多个插件，会被增强多次</span>
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;newBuilder = possibleNewBuilder;
 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp;}
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-keyword">return</span> newBuilder;
 &nbsp; &nbsp;}
 &nbsp; &nbsp;<span class="hljs-keyword">return</span> builder;
}
</code></pre>
<p>这里需要注意：如果一个类被多个插件匹配会被增强多次，当你打开 IS_OPEN_DEBUGGING_CLASS 配置项时，会看到对应的多个 class 文件。</p>
<h3>加载 BootService</h3>
<p>SkyWalking Agent &nbsp;启动的最后一步是使用前面介绍的 JDK SPI 技术加载 BootService 接口的所有实现类，BootService 接口中定义了 SkyWalking Agent 核心服务的行为，其定义如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">BootService</span> </span>{
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">prepare</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable</span>;
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">boot</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable</span>;
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">onComplete</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable</span>;
 &nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">shutdown</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable</span>;
}
</code></pre>
<p>ServiceManager 是 BootService 实例的管理器，主要负责管理 BootService 实例的生命周期。</p>
<p>ServiceManager 是个单例，底层维护了一个 bootedServices 集合（Map&lt;Class, BootService&gt; 类型），记录了每个 BootService 实现对应的实例。boot() 方法是 ServiceManager 的核心方法，它首先通过 load() 方法实例化全部 BootService 接口实现，如下所示：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">load</span><span class="hljs-params">(List&lt;BootService&gt; allServices)</span> </span>{ 
 &nbsp; &nbsp;<span class="hljs-comment">// 很明显使用了 JDK SPI 技术加载并实例化 META-INF/services下的全部 </span>
 &nbsp; &nbsp;<span class="hljs-comment">// BootService接口实现</span>
 &nbsp; &nbsp;Iterator&lt;BootService&gt; iterator = ServiceLoader.load(
 &nbsp; &nbsp; &nbsp; &nbsp;BootService<span class="hljs-class">.<span class="hljs-keyword">class</span>,<span class="hljs-title">AgentClassLoader</span>.<span class="hljs-title">getDefault</span>()).<span class="hljs-title">iterator</span>()</span>;
 &nbsp; &nbsp;<span class="hljs-keyword">while</span> (iterator.hasNext()) {
 &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 记录到方法参数传入的 allServices集合中</span>
 &nbsp; &nbsp; &nbsp; &nbsp;allServices.add(iterator.next()); 
 &nbsp; &nbsp;}
}
</code></pre>
<p>在 apm-agent-core 模块的 resource/META-INF.services/org.apache.skywalking.apm.agent.core.boot.BootService 文件中，记录了 ServiceManager 要加载的 BootService 接口实现类，如下所示，这些类在后面的课时中会逐个详细介绍其具体功能：</p>
<pre><code data-language="java" class="lang-java">org.apache.skywalking.apm.agent.core.remote.TraceSegmentServiceClient
org.apache.skywalking.apm.agent.core.context.ContextManager
org.apache.skywalking.apm.agent.core.sampling.SamplingService
org.apache.skywalking.apm.agent.core.remote.GRPCChannelManager
org.apache.skywalking.apm.agent.core.jvm.JVMService
org.apache.skywalking.apm.agent.core.remote.ServiceAndEndpointRegisterClient
org.apache.skywalking.apm.agent.core.context.ContextManagerExtendService
</code></pre>
<p>加载完上述 BootService 实现类型之后，ServiceManager 会针对 BootService 上的 @DefaultImplementor 和 @OverrideImplementor 注解进行处理：</p>
<ul>
<li>@DefaultImplementor 注解用于标识 BootService 接口的默认实现。</li>
<li>@OverrideImplementor 注解用于覆盖默认 BootService 实现，通过其 value 字段指定要覆盖的默认实现。</li>
</ul>
<p>BootService 的覆盖逻辑如下图所示：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/08/53/Ciqah16FsIyAKF5JAAFCMa4LIHU953.png" alt=""></p>
<p>确定完要使用的 BootService 实现之后，ServiceManager 将统一初始化 bootServices 集合中的 BootService 实现，同样是在 ServiceManager.boot() 方法中，会逐个调用 BootService 实现的 prepare()、startup()、onComplete() 方法，具体实现如下：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">boot</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp;bootedServices = loadAllServices(); 
 &nbsp; &nbsp;prepare(); <span class="hljs-comment">// 调用全部BootService对象的prepare()方法</span>
 &nbsp; &nbsp;startup(); <span class="hljs-comment">// 调用全部BootService对象的boot()方法</span>
 &nbsp; &nbsp;onComplete(); <span class="hljs-comment">// 调用全部BootService对象的onComplete()方法</span>
}
</code></pre>
<p>在 Skywalking Agent 启动流程的最后，会添加一个 JVM 退出钩子，并通过 ServiceManager.shutdown() 方法，关闭前文启动的全部 BootService 服务。</p>
<p>SkywalkingAgent.premain() 方法中相关的代码片段如下：</p>
<pre><code data-language="java" class="lang-java">Runtime.getRuntime().addShutdownHook(<span class="hljs-keyword">new</span> Thread(<span class="hljs-keyword">new</span> Runnable() {
 &nbsp; &nbsp;<span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
 &nbsp; &nbsp; &nbsp; &nbsp;ServiceManager.INSTANCE.shutdown(); 
 &nbsp; &nbsp;}
}, <span class="hljs-string">"skywalking service shutdown thread"</span>));
</code></pre>
<p>总结</p>
<p>本课时重点介绍了 SkyWalking Agent 启动核心流程的实现，深入分析了 Skywalking Agent 配置信息的初始化、插件加载原理、AgentBuilder 如何与插件类配合增强目标类、BootService 的加载流程。本课时是整个 Skywalking Agent 的框架性流程介绍，在后续的课时中将详细介绍 AbstractClassEnhancePluginDefine 以及 BootService 接口的实现。</p>

---

### 精选评论

##### **桂：
> 有点复杂

##### JackLi：
> 之前看徐老师的书，本次看徐老师出了课程就参与了

