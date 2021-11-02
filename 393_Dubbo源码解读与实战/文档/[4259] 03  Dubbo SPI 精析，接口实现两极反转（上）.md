<p data-nodeid="82496">Dubbo 为了更好地达到 OCP 原则（即“对扩展开放，对修改封闭”的原则），采用了“<strong data-nodeid="82502">微内核+插件</strong>”的架构。那什么是微内核架构呢？微内核架构也被称为插件化架构（Plug-in Architecture），这是一种面向功能进行拆分的可扩展性架构。内核功能是比较稳定的，只负责管理插件的生命周期，不会因为系统功能的扩展而不断进行修改。功能上的扩展全部封装到插件之中，插件模块是独立存在的模块，包含特定的功能，能拓展内核系统的功能。</p>


<p data-nodeid="82105">微内核架构中，内核通常采用 Factory、IoC、OSGi 等方式管理插件生命周期，<strong data-nodeid="82171">Dubbo 最终决定采用 SPI 机制来加载插件</strong>，Dubbo SPI 参考 JDK 原生的 SPI 机制，进行了性能优化以及功能增强。因此，在讲解 Dubbo SPI 之前，我们有必要先来介绍一下 JDK SPI 的工作原理。</p>
<h3 data-nodeid="82761" class="">JDK SPI</h3>

<p data-nodeid="82107">SPI（Service Provider Interface）主要是被<strong data-nodeid="82178">框架开发人员</strong>使用的一种技术。例如，使用 Java 语言访问数据库时我们会使用到 java.sql.Driver 接口，不同数据库产品底层的协议不同，提供的 java.sql.Driver 实现也不同，在开发 java.sql.Driver 接口时，开发人员并不清楚用户最终会使用哪个数据库，在这种情况下就可以使用 Java SPI 机制在实际运行过程中，为 java.sql.Driver 接口寻找具体的实现。</p>
<h4 data-nodeid="83021" class="">1. JDK SPI 机制</h4>

<p data-nodeid="82111">当服务的提供者提供了一种接口的实现之后，需要在 Classpath 下的 META-INF/services/ 目录里创建一个以服务接口命名的文件，此文件记录了该 jar 包提供的服务接口的具体实现类。当某个应用引入了该 jar 包且需要使用该服务时，JDK SPI 机制就可以通过查找这个 jar 包的 META-INF/services/ 中的配置文件来获得具体的实现类名，进行实现类的加载和实例化，最终使用该实现类完成业务功能。</p>
<p data-nodeid="83523">下面我们通过一个简单的示例演示下 JDK SPI 的基本使用方式：</p>
<p data-nodeid="83524" class=""><img src="https://s0.lgstatic.com/i/image/M00/3D/03/CgqCHl8o_UCAI01eAABGsg2cqbw825.png" alt="image (1).png" data-nodeid="83532"></p>


<p data-nodeid="82114">首先我们需要创建一个 Log 接口，来模拟日志打印的功能：</p>
<pre class="lang-java" data-nodeid="84584"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Log</span> </span>{ 
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">log</span><span class="hljs-params">(String info)</span></span>; 
} 
</code></pre>




<p data-nodeid="82116">接下来提供两个实现—— Logback 和 Log4j，分别代表两个不同日志框架的实现，如下所示：</p>
<pre class="lang-java" data-nodeid="84847"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Logback</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Log</span> </span>{ 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">log</span><span class="hljs-params">(String info)</span> </span>{ 
        System.out.println(<span class="hljs-string">"Logback:"</span> + info); 
    } 
} 
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Log4j</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Log</span> </span>{ 
    <span class="hljs-meta">@Override</span> 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">log</span><span class="hljs-params">(String info)</span> </span>{ 
        System.out.println(<span class="hljs-string">"Log4j:"</span> + info); 
    } 
} 
</code></pre>

<p data-nodeid="82118">在项目的 resources/META-INF/services 目录下添加一个名为 com.xxx.Log 的文件，这是 JDK SPI 需要读取的配置文件，具体内容如下：</p>
<pre class="lang-java" data-nodeid="85110"><code data-language="java">com.xxx.impl.Log4j 
com.xxx.impl.Logback 
</code></pre>

<p data-nodeid="82120">最后创建 main() 方法，其中会加载上述配置文件，创建全部 Log 接口实现的实例，并执行其 log() 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="85373"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Main</span> </span>{ 
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{ 
        ServiceLoader&lt;Log&gt; serviceLoader =  
                ServiceLoader.load(Log.class); 
        Iterator&lt;Log&gt; iterator = serviceLoader.iterator(); 
        <span class="hljs-keyword">while</span> (iterator.hasNext()) { 
            Log log = iterator.next(); 
            log.log("JDK SPI");  
        } 
    } 
} 
// 输出如下: 
// Log4j:JDK SPI 
// Logback:JDK SPI 
</code></pre>

<h4 data-nodeid="85902" class="">2. JDK SPI 源码分析</h4>


<p data-nodeid="82125">通过上述示例，我们可以看到 JDK SPI 的入口方法是 ServiceLoader.load()  方法，接下来我们就对其具体实现进行深入分析。</p>
<p data-nodeid="86412">在 ServiceLoader.load() 方法中，首先会尝试获取当前使用的 ClassLoader（获取当前线程绑定的 ClassLoader，查找失败后使用 SystemClassLoader），然后调用 reload() 方法，调用关系如下图所示：</p>
<p data-nodeid="86413" class=""><img src="https://s0.lgstatic.com/i/image/M00/3C/F8/Ciqc1F8o_V6AR93jAABeDIu_Kso211.png" alt="image (2).png" data-nodeid="86421"></p>


<p data-nodeid="82128">在 reload() 方法中，首先会清理 providers 缓存（LinkedHashMap 类型的集合），该缓存用来记录 ServiceLoader 创建的实现对象，其中 Key 为实现类的完整类名，Value 为实现类的对象。之后创建 LazyIterator 迭代器，用于读取 SPI 配置文件并实例化实现类对象。</p>
<p data-nodeid="82129">ServiceLoader.reload() 方法的具体实现，如下所示：</p>
<pre class="lang-java" data-nodeid="86688"><code data-language="java"><span class="hljs-comment">//&nbsp;缓存，用来缓存 ServiceLoader创建的实现对象 </span>
<span class="hljs-keyword">private</span> LinkedHashMap&lt;String,S&gt; providers = <span class="hljs-keyword">new</span> LinkedHashMap&lt;&gt;(); 
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">reload</span><span class="hljs-params">()</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;providers.clear();&nbsp;<span class="hljs-comment">//&nbsp;清空缓存 </span>
    lookupIterator = <span class="hljs-keyword">new</span> LazyIterator(service, loader); <span class="hljs-comment">// 迭代器 </span>
} 
</code></pre>

<p data-nodeid="87213">在前面的示例中，main() 方法中使用的迭代器底层就是调用了 ServiceLoader.LazyIterator 实现的。Iterator 接口有两个关键方法：hasNext() 方法和 next() 方法。这里的 LazyIterator 中的next() 方法最终调用的是其 nextService() 方法，hasNext() 方法最终调用的是 hasNextService() 方法，调用关系如下图所示：</p>
<p data-nodeid="87214" class=""><img src="https://s0.lgstatic.com/i/image/M00/3C/F8/Ciqc1F8o_WmAZSkmAABmcc0uM54214.png" alt="image (3).png" data-nodeid="87222"></p>


<p data-nodeid="82133">首先来看 LazyIterator.hasNextService() 方法，该方法主要<strong data-nodeid="82206">负责查找 META-INF/services 目录下的 SPI 配置文件</strong>，并进行遍历，大致实现如下所示：</p>
<pre class="lang-java" data-nodeid="87497"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String PREFIX = <span class="hljs-string">"META-INF/services/"</span>; 
Enumeration&lt;URL&gt; configs = <span class="hljs-keyword">null</span>; 
Iterator&lt;String&gt; pending = <span class="hljs-keyword">null</span>; 
String nextName = <span class="hljs-keyword">null</span>; 
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">hasNextService</span><span class="hljs-params">()</span> </span>{ 
    <span class="hljs-keyword">if</span> (nextName != <span class="hljs-keyword">null</span>) { 
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>; 
    } 
    <span class="hljs-keyword">if</span> (configs == <span class="hljs-keyword">null</span>) { 
        <span class="hljs-comment">// PREFIX前缀与服务接口的名称拼接起来，就是META-INF目录下定义的SPI配 </span>
        <span class="hljs-comment">//&nbsp;置文件(即示例中的META-INF/services/com.xxx.Log) </span>
        String fullName = PREFIX + service.getName(); 
        <span class="hljs-comment">// 加载配置文件 </span>
        <span class="hljs-keyword">if</span> (loader == <span class="hljs-keyword">null</span>) 
            configs = ClassLoader.getSystemResources(fullName); 
        <span class="hljs-keyword">else</span> 
            configs = loader.getResources(fullName); 
    } 
    <span class="hljs-comment">// 按行SPI遍历配置文件的内容 </span>
    <span class="hljs-keyword">while</span> ((pending == <span class="hljs-keyword">null</span>) || !pending.hasNext()) {  
        <span class="hljs-keyword">if</span> (!configs.hasMoreElements()) { 
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>; 
        } 
        <span class="hljs-comment">// 解析配置文件 </span>
        pending = parse(service, configs.nextElement());  
    } 
    nextName = pending.next(); <span class="hljs-comment">// 更新 nextName字段 </span>
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>; 
}   
</code></pre>

<p data-nodeid="82135">在 hasNextService() 方法中完成 SPI 配置文件的解析之后，再来看 LazyIterator.nextService() 方法，该方法<strong data-nodeid="82212">负责实例化 hasNextService() 方法读取到的实现类</strong>，其中会将实例化的对象放到 providers 集合中缓存起来，核心实现如下所示：</p>
<pre class="lang-java" data-nodeid="87772"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> S <span class="hljs-title">nextService</span><span class="hljs-params">()</span> </span>{ 
    String cn = nextName; 
    nextName = <span class="hljs-keyword">null</span>; 
&nbsp; &nbsp; <span class="hljs-comment">// 加载 nextName字段指定的类 </span>
    Class&lt;?&gt; c = Class.forName(cn, <span class="hljs-keyword">false</span>, loader); 
    <span class="hljs-keyword">if</span> (!service.isAssignableFrom(c)) { <span class="hljs-comment">// 检测类型 </span>
        fail(service, <span class="hljs-string">"Provider "</span> + cn  + <span class="hljs-string">" not a subtype"</span>); 
    } 
    S p = service.cast(c.newInstance()); <span class="hljs-comment">// 创建实现类的对象 </span>
    providers.put(cn, p); <span class="hljs-comment">// 将实现类名称以及相应实例对象添加到缓存 </span>
    <span class="hljs-keyword">return</span> p; 
} 
</code></pre>

<p data-nodeid="82137">以上就是在 main() 方法中使用的迭代器的底层实现。最后，我们再来看一下 main() 方法中使用ServiceLoader.iterator() 方法拿到的迭代器是如何实现的，这个迭代器是依赖 LazyIterator 实现的一个匿名内部类，核心实现如下：</p>
<pre class="lang-java" data-nodeid="88047"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Iterator&lt;S&gt; <span class="hljs-title">iterator</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Iterator&lt;S&gt;() { 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// knownProviders用来迭代providers缓存 </span>
&nbsp; &nbsp; &nbsp; &nbsp; Iterator&lt;Map.Entry&lt;String,S&gt;&gt; knownProviders 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; = providers.entrySet().iterator(); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">hasNext</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 先走查询缓存，缓存查询失败，再通过LazyIterator加载 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (knownProviders.hasNext())&nbsp; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> lookupIterator.hasNext(); 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> S <span class="hljs-title">next</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 先走查询缓存，缓存查询失败，再通过 LazyIterator加载 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (knownProviders.hasNext()) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> knownProviders.next().getValue(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> lookupIterator.next(); 
&nbsp; &nbsp; &nbsp; &nbsp; } 
        <span class="hljs-comment">// 省略remove()方法 </span>
&nbsp; &nbsp; }; 
} 
</code></pre>

<h4 data-nodeid="88878" class="">3. JDK SPI 在 JDBC 中的应用</h4>



<p data-nodeid="82142">了解了 JDK SPI 实现的原理之后，我们再来看实践中 JDBC 是如何使用 JDK SPI 机制加载不同数据库厂商的实现类。</p>
<p data-nodeid="82143">JDK 中只定义了一个 java.sql.Driver 接口，具体的实现是由不同数据库厂商来提供的。这里我们就以 MySQL 提供的 JDBC 实现包为例进行分析。</p>
<p data-nodeid="82144">在 mysql-connector-java-*.jar 包中的 META-INF/services 目录下，有一个 java.sql.Driver 文件中只有一行内容，如下所示：</p>
<pre class="lang-java" data-nodeid="89150"><code data-language="java">com.mysql.cj.jdbc.Driver 
</code></pre>

<p data-nodeid="82146">在使用 mysql-connector-java-*.jar 包连接 MySQL 数据库的时候，我们会用到如下语句创建数据库连接：</p>
<pre class="lang-java" data-nodeid="89421"><code data-language="java">String url = <span class="hljs-string">"jdbc:xxx://xxx:xxx/xxx"</span>; 
Connection conn = DriverManager.getConnection(url, username, pwd); 
</code></pre>

<p data-nodeid="82148"><strong data-nodeid="82227">DriverManager 是 JDK 提供的数据库驱动管理器</strong>，其中的代码片段，如下所示：</p>
<pre class="lang-java" data-nodeid="89692"><code data-language="java"><span class="hljs-keyword">static</span> { 
    loadInitialDrivers(); 
    println(<span class="hljs-string">"JDBC DriverManager initialized"</span>); 
} 
</code></pre>

<p data-nodeid="82150">在调用 getConnection() 方法的时候，DriverManager 类会被 Java 虚拟机加载、解析并触发 static 代码块的执行；在 loadInitialDrivers() 方法中通过 JDK SPI 扫描 Classpath 下  java.sql.Driver 接口实现类并实例化，核心实现如下所示：</p>
<pre class="lang-java" data-nodeid="89963"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">loadInitialDrivers</span><span class="hljs-params">()</span> </span>{ 
&nbsp; &nbsp; String drivers = System.getProperty(<span class="hljs-string">"jdbc.drivers"</span>) 
    <span class="hljs-comment">// 使用 JDK SPI机制加载所有 java.sql.Driver实现类 </span>
&nbsp; &nbsp; ServiceLoader&lt;Driver&gt; loadedDrivers =  
           ServiceLoader.load(Driver.class); 
&nbsp; &nbsp; Iterator&lt;Driver&gt; driversIterator = loadedDrivers.iterator(); 
&nbsp; &nbsp; <span class="hljs-keyword">while</span>(driversIterator.hasNext()) { 
&nbsp; &nbsp; &nbsp; &nbsp; driversIterator.next(); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; String[] driversList = drivers.split(":"); 
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (String aDriver : driversList) { // 初始化Driver实现类 
&nbsp; &nbsp; &nbsp; &nbsp; Class.forName(aDriver, <span class="hljs-keyword">true</span>, 
            ClassLoader.getSystemClassLoader()); 
&nbsp; &nbsp; } 
} 
</code></pre>

<p data-nodeid="82152">在 MySQL 提供的 com.mysql.cj.jdbc.Driver 实现类中，同样有一段 static 静态代码块，这段代码会创建一个 com.mysql.cj.jdbc.Driver 对象并注册到 DriverManager.registeredDrivers 集合中（CopyOnWriteArrayList 类型），如下所示：</p>
<pre class="lang-java" data-nodeid="90234"><code data-language="java"><span class="hljs-keyword">static</span> { 
   java.sql.DriverManager.registerDriver(<span class="hljs-keyword">new</span> Driver()); 
} 
</code></pre>

<p data-nodeid="82154">在 getConnection() 方法中，DriverManager 从该 registeredDrivers 集合中获取对应的 Driver 对象创建 Connection，核心实现如下所示：</p>
<pre class="lang-java" data-nodeid="90505"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Connection <span class="hljs-title">getConnection</span><span class="hljs-params">(String url, java.util.Properties info, Class&lt;?&gt; caller)</span> <span class="hljs-keyword">throws</span> SQLException </span>{ 
&nbsp; &nbsp; <span class="hljs-comment">// 省略 try/catch代码块以及权限处理逻辑 </span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span>(DriverInfo aDriver : registeredDrivers) { 
&nbsp; &nbsp; &nbsp; &nbsp; Connection con = aDriver.driver.connect(url, info); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> con; 
&nbsp; &nbsp; } 
} 
</code></pre>

<h3 data-nodeid="90776" class="">总结</h3>

<p data-nodeid="82157">本文我们通过一个示例入手，介绍了 JDK 提供的 SPI 机制的基本使用，然后深入分析了 JDK SPI 的核心原理和底层实现，对其源码进行了深入剖析，最后我们以 MySQL 提供的 JDBC 实现为例，分析了 JDK SPI 在实践中的使用方式。</p>
<p data-nodeid="82158">JDK SPI 机制虽然简单易用，但是也存在一些小瑕疵，你可以先思考一下，在下一课时剖析 Dubbo SPI 机制的时候，我会为你解答该问题。</p>

---

### 精选评论

##### **安：
> 如果要新增一种驱动，是不是就得手动在配置文件加上一个，删除也需要手动在配置文件中去掉

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

##### **博：
> 最底层还是Class.forName

##### **东：
> jdk spi 默认加载所有spi接口的实现类

##### **健：
> 老师应该再说下spi设计破坏双亲委派的原理

##### **春：
> providers为啥会用 LinkedHashMap呢？其他的Map不行吗？

##### **青：
> 精辟

##### *阳：
> 很不错

##### ✔：
> 做了这么久的码农，没往下翻过这些代码，惭愧，多谢老师领路

##### **光：
> 讲的已经很清楚了

