<p data-nodeid="203396" class="">在上一课时，我们一起学习了 JDK SPI 的基础使用以及核心原理，不过 Dubbo 并没有直接使用 JDK SPI 机制，而是借鉴其思想，实现了自身的一套 SPI 机制，这就是本课时将重点介绍的内容。</p>
<h3 data-nodeid="203397">Dubbo SPI</h3>
<p data-nodeid="203398">在开始介绍 Dubbo SPI 实现之前，我们先来统一下面两个概念。</p>
<ul data-nodeid="203399">
<li data-nodeid="203400">
<p data-nodeid="203401"><strong data-nodeid="203557">扩展点</strong>：通过 SPI 机制查找并加载实现的接口（又称“扩展接口”）。前文示例中介绍的 Log 接口、com.mysql.cj.jdbc.Driver 接口，都是扩展点。</p>
</li>
<li data-nodeid="203402">
<p data-nodeid="203403"><strong data-nodeid="203562">扩展点实现</strong>：实现了扩展接口的实现类。</p>
</li>
</ul>
<p data-nodeid="203404">通过前面的分析可以发现，JDK SPI 在查找扩展实现类的过程中，需要遍历 SPI 配置文件中定义的所有实现类，该过程中会将这些实现类全部实例化。如果 SPI 配置文件中定义了多个实现类，而我们只需要使用其中一个实现类时，就会生成不必要的对象。例如，org.apache.dubbo.rpc.Protocol 接口有 InjvmProtocol、DubboProtocol、RmiProtocol、HttpProtocol、HessianProtocol、ThriftProtocol 等多个实现，如果使用 JDK SPI，就会加载全部实现类，导致资源的浪费。</p>
<p data-nodeid="203405"><strong data-nodeid="203567">Dubbo SPI 不仅解决了上述资源浪费的问题，还对 SPI 配置文件扩展和修改。</strong></p>
<p data-nodeid="203406">首先，Dubbo 按照 SPI 配置文件的用途，将其分成了三类目录。</p>
<ul data-nodeid="203407">
<li data-nodeid="203408">
<p data-nodeid="203409">META-INF/services/ 目录：该目录下的 SPI 配置文件用来兼容 JDK SPI 。</p>
</li>
<li data-nodeid="203410">
<p data-nodeid="203411">META-INF/dubbo/ 目录：该目录用于存放用户自定义 SPI 配置文件。</p>
</li>
<li data-nodeid="203412">
<p data-nodeid="203413">META-INF/dubbo/internal/ 目录：该目录用于存放 Dubbo 内部使用的 SPI 配置文件。</p>
</li>
</ul>
<p data-nodeid="203414">然后，Dubbo 将 SPI 配置文件改成了 <strong data-nodeid="203577">KV 格式</strong>，例如：</p>
<pre class="lang-java" data-nodeid="203415"><code data-language="java">dubbo=org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol
</code></pre>
<p data-nodeid="203416">其中 key 被称为扩展名（也就是 ExtensionName），当我们在为一个接口查找具体实现类时，可以指定扩展名来选择相应的扩展实现。例如，这里指定扩展名为 dubbo，Dubbo SPI 就知道我们要使用：org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol 这个扩展实现类，只实例化这一个扩展实现即可，无须实例化 SPI 配置文件中的其他扩展实现类。</p>
<p data-nodeid="203417">使用 KV 格式的 SPI 配置文件的另一个好处是：让我们更容易定位到问题。假设我们使用的一个扩展实现类所在的 jar 包没有引入到项目中，那么 Dubbo SPI 在抛出异常的时候，会携带该扩展名信息，而不是简单地提示扩展实现类无法加载。这些更加准确的异常信息降低了排查问题的难度，提高了排查问题的效率。</p>
<p data-nodeid="203418">下面我们正式进入 Dubbo SPI 核心实现的介绍。</p>
<h4 data-nodeid="203419">1. @SPI 注解</h4>
<p data-nodeid="203420">Dubbo 中某个接口被 @SPI注解修饰时，就表示该接口是<strong data-nodeid="203589">扩展接口</strong>，前文示例中的 org.apache.dubbo.rpc.Protocol 接口就是一个扩展接口：</p>
<p data-nodeid="203421"><img src="https://s0.lgstatic.com/i/image/M00/3E/A4/CgqCHl8s936AYuePAABLd6cRz6w646.png" alt="Drawing 0.png" data-nodeid="203592"></p>
<p data-nodeid="203422">@SPI 注解的 value 值指定了默认的扩展名称，例如，在通过 Dubbo SPI 加载 Protocol 接口实现时，如果没有明确指定扩展名，则默认会将 @SPI 注解的 value 值作为扩展名，即加载 dubbo 这个扩展名对应的 org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol 这个扩展实现类，相关的 SPI 配置文件在 dubbo-rpc-dubbo 模块中，如下图所示：</p>
<p data-nodeid="203423"><img src="https://s0.lgstatic.com/i/image/M00/3E/A4/CgqCHl8s94mAaj2mAABcaXHNXqc467.png" alt="Drawing 1.png" data-nodeid="203596"></p>
<p data-nodeid="203424"><strong data-nodeid="203600">那 ExtensionLoader 是如何处理 @SPI 注解的呢？</strong></p>
<p data-nodeid="203425">ExtensionLoader 位于 dubbo-common 模块中的 extension 包中，功能类似于 JDK SPI 中的 java.util.ServiceLoader。Dubbo SPI 的核心逻辑几乎都封装在 ExtensionLoader 之中（其中就包括 @SPI 注解的处理逻辑），其使用方式如下所示：</p>
<pre class="lang-java" data-nodeid="203426"><code data-language="java">Protocol protocol = ExtensionLoader 
   .getExtensionLoader(Protocol.class).getExtension("dubbo");
</code></pre>
<p data-nodeid="203427">这里首先来了解一下 ExtensionLoader 中三个核心的静态字段。</p>
<ul data-nodeid="206198">
<li data-nodeid="206199">
<p data-nodeid="206200" class=""><strong data-nodeid="206208">strategies（LoadingStrategy[]类型）:</strong> LoadingStrategy 接口有三个实现（通过 JDK SPI 方式加载的），如下图所示，分别对应前面介绍的三个 Dubbo SPI 配置文件所在的目录，且都继承了 Prioritized 这个优先级接口，默认优先级是</p>
</li>
</ul>




<pre class="lang-java" data-nodeid="203431"><code data-language="java"> DubboInternalLoadingStrategy &gt; DubboLoadingStrategy &gt; ServicesLoadingStrateg
</code></pre>
<p data-nodeid="203432"><img src="https://s0.lgstatic.com/i/image/M00/3E/99/Ciqc1F8s95mANXYKAADUVwBlgxs297.png" alt="Drawing 2.png" data-nodeid="203614"></p>
<ul data-nodeid="203433">
<li data-nodeid="203434">
<p data-nodeid="203435"><strong data-nodeid="203624">EXTENSION_LOADERS（ConcurrentMap&lt;Class, ExtensionLoader&gt;类型）</strong><br>
：Dubbo 中一个扩展接口对应一个 ExtensionLoader 实例，该集合缓存了全部 ExtensionLoader 实例，其中的 Key 为扩展接口，Value 为加载其扩展实现的 ExtensionLoader 实例。</p>
</li>
<li data-nodeid="203436">
<p data-nodeid="203437"><strong data-nodeid="203635">EXTENSION_INSTANCES（ConcurrentMap&lt;Class&lt;?&gt;, Object&gt;类型）</strong>：该集合缓存了扩展实现类与其实例对象的映射关系。在前文示例中，Key 为 Class，Value 为 DubboProtocol 对象。</p>
</li>
</ul>
<p data-nodeid="203438">下面我们再来关注一下 ExtensionLoader 的实例字段。</p>
<ul data-nodeid="203439">
<li data-nodeid="203440">
<p data-nodeid="203441"><strong data-nodeid="203643">type（Class&lt;?&gt;类型）</strong>：当前 ExtensionLoader 实例负责加载扩展接口。</p>
</li>
<li data-nodeid="203442">
<p data-nodeid="203443"><strong data-nodeid="203648">cachedDefaultName（String类型）</strong>：记录了 type 这个扩展接口上 @SPI 注解的 value 值，也就是默认扩展名。</p>
</li>
<li data-nodeid="203444">
<p data-nodeid="203445"><strong data-nodeid="203657">cachedNames（ConcurrentMap&lt;Class&lt;?&gt;, String&gt;类型）</strong>：缓存了该 ExtensionLoader 加载的扩展实现类与扩展名之间的映射关系。</p>
</li>
<li data-nodeid="203446">
<p data-nodeid="203447"><strong data-nodeid="203668">cachedClasses（Holder&lt;Map&lt;String, Class&lt;?&gt;&gt;&gt;类型）</strong>：缓存了该 ExtensionLoader 加载的扩展名与扩展实现类之间的映射关系。cachedNames 集合的反向关系缓存。</p>
</li>
<li data-nodeid="203448">
<p data-nodeid="203449"><strong data-nodeid="203678">cachedInstances（ConcurrentMap&lt;String, Holder<code data-backticks="1" data-nodeid="203673">&lt;Object&gt;</code>&gt;类型）</strong>：缓存了该 ExtensionLoader 加载的扩展名与扩展实现对象之间的映射关系。</p>
</li>
</ul>
<p data-nodeid="203450">ExtensionLoader.getExtensionLoader() 方法会根据扩展接口从 EXTENSION_LOADERS 缓存中查找相应的 ExtensionLoader 实例，核心实现如下：</p>
<pre class="lang-java" data-nodeid="203451"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">ExtensionLoader&lt;T&gt; <span class="hljs-title">getExtensionLoader</span><span class="hljs-params">(Class&lt;T&gt; type)</span> </span>{ 
&nbsp; &nbsp; ExtensionLoader&lt;T&gt; loader =
         (ExtensionLoader&lt;T&gt;) EXTENSION_LOADERS.get(type); 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (loader == <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; EXTENSION_LOADERS.putIfAbsent(type, 
               <span class="hljs-keyword">new</span> ExtensionLoader&lt;T&gt;(type)); 
&nbsp; &nbsp; &nbsp; &nbsp; loader = (ExtensionLoader&lt;T&gt;) EXTENSION_LOADERS.get(type); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> loader; 
}
</code></pre>
<p data-nodeid="203452">得到接口对应的 ExtensionLoader 对象之后会调用其 getExtension() 方法，根据传入的扩展名称从 cachedInstances 缓存中查找扩展实现的实例，最终将其实例化后返回：</p>
<pre class="lang-java" data-nodeid="203453"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">getExtension</span><span class="hljs-params">(String name)</span> </span>{ 
    <span class="hljs-comment">// getOrCreateHolder()方法中封装了查找cachedInstances缓存的逻辑 </span>
    Holder&lt;Object&gt; holder = getOrCreateHolder(name); 
    Object instance = holder.get(); 
    <span class="hljs-keyword">if</span> (instance == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// double-check防止并发问题 </span>
        <span class="hljs-keyword">synchronized</span> (holder) { 
            instance = holder.get(); 
            <span class="hljs-keyword">if</span> (instance == <span class="hljs-keyword">null</span>) { 
                <span class="hljs-comment">// 根据扩展名从SPI配置文件中查找对应的扩展实现类 </span>
                instance = createExtension(name); 
                holder.set(instance); 
            } 
        } 
    } 
    <span class="hljs-keyword">return</span> (T) instance; 
}
</code></pre>
<p data-nodeid="203454">在 createExtension() 方法中完成了 SPI 配置文件的查找以及相应扩展实现类的实例化，同时还实现了自动装配以及自动 Wrapper 包装等功能。其核心流程是这样的：</p>
<ol data-nodeid="203455">
<li data-nodeid="203456">
<p data-nodeid="203457">获取 cachedClasses 缓存，根据扩展名从 cachedClasses 缓存中获取扩展实现类。如果 cachedClasses 未初始化，则会扫描前面介绍的三个 SPI 目录获取查找相应的 SPI 配置文件，然后加载其中的扩展实现类，最后将扩展名和扩展实现类的映射关系记录到 cachedClasses 缓存中。这部分逻辑在 loadExtensionClasses() 和 loadDirectory() 方法中。</p>
</li>
<li data-nodeid="203458">
<p data-nodeid="203459">根据扩展实现类从 EXTENSION_INSTANCES 缓存中查找相应的实例。如果查找失败，会通过反射创建扩展实现对象。</p>
</li>
<li data-nodeid="203460">
<p data-nodeid="203461"><strong data-nodeid="203692">自动装配</strong>扩展实现对象中的属性（即调用其 setter）。这里涉及 ExtensionFactory 以及自动装配的相关内容，本课时后面会进行详细介绍。</p>
</li>
<li data-nodeid="203462">
<p data-nodeid="203463"><strong data-nodeid="203697">自动包装</strong>扩展实现对象。这里涉及 Wrapper 类以及自动包装特性的相关内容，本课时后面会进行详细介绍。</p>
</li>
<li data-nodeid="203464">
<p data-nodeid="203465">如果扩展实现类实现了 Lifecycle 接口，在 initExtension() 方法中会调用 initialize() 方法进行初始化。</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="203466"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> T <span class="hljs-title">createExtension</span><span class="hljs-params">(String name)</span> </span>{ 
&nbsp; &nbsp; Class&lt;?&gt; clazz = getExtensionClasses().get(name); <span class="hljs-comment">// --- 1 </span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (clazz == <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> findException(name); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">try</span> { 
&nbsp; &nbsp; &nbsp; &nbsp; T instance = (T) EXTENSION_INSTANCES.get(clazz); <span class="hljs-comment">// --- 2 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (instance == <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance()); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; instance = (T) EXTENSION_INSTANCES.get(clazz); 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; injectExtension(instance); <span class="hljs-comment">// --- 3 </span>
&nbsp; &nbsp; &nbsp; &nbsp; Set&lt;Class&lt;?&gt;&gt; wrapperClasses = cachedWrapperClasses; <span class="hljs-comment">// --- 4 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (CollectionUtils.isNotEmpty(wrapperClasses)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Class&lt;?&gt; wrapperClass : wrapperClasses) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance)); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; initExtension(instance); <span class="hljs-comment">// ---5</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> instance; 
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) { 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"Extension instance (name: "</span> + name + <span class="hljs-string">", class: "</span> + 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; type + <span class="hljs-string">") couldn't be instantiated: "</span> + t.getMessage(), t); 
&nbsp; &nbsp; } 
}
</code></pre>
<h4 data-nodeid="203467">2. @Adaptive 注解与适配器</h4>
<p data-nodeid="203468">@Adaptive 注解用来实现 Dubbo 的适配器功能，那什么是适配器呢？这里我们通过一个示例进行说明。Dubbo 中的 ExtensionFactory 接口有三个实现类，如下图所示，ExtensionFactory 接口上有 @SPI 注解，AdaptiveExtensionFactory 实现类上有 @Adaptive 注解。</p>
<p data-nodeid="203469"><img src="https://s0.lgstatic.com/i/image/M00/3E/99/Ciqc1F8s-D6AZFtdAAC318rtQ-I710.png" alt="Drawing 3.png" data-nodeid="203705"></p>
<p data-nodeid="203470">AdaptiveExtensionFactory 不实现任何具体的功能，而是用来适配 ExtensionFactory 的 SpiExtensionFactory 和 SpringExtensionFactory 这两种实现。AdaptiveExtensionFactory 会根据运行时的一些状态来选择具体调用 ExtensionFactory 的哪个实现。</p>
<p data-nodeid="203471">@Adaptive 注解还可以加到接口方法之上，Dubbo 会动态生成适配器类。例如，Transporter接口有两个被 @Adaptive 注解修饰的方法：</p>
<pre class="lang-java" data-nodeid="203472"><code data-language="java"><span class="hljs-meta">@SPI("netty")</span> 
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Transporter</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.SERVER_KEY, Constants.TRANSPORTER_KEY})</span> 
&nbsp; &nbsp; <span class="hljs-function">RemotingServer <span class="hljs-title">bind</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> <span class="hljs-keyword">throws</span> RemotingException</span>; 
&nbsp; &nbsp; <span class="hljs-meta">@Adaptive({Constants.CLIENT_KEY, Constants.TRANSPORTER_KEY})</span> 
&nbsp; &nbsp; <span class="hljs-function">Client <span class="hljs-title">connect</span><span class="hljs-params">(URL url, ChannelHandler handler)</span> <span class="hljs-keyword">throws</span> RemotingException</span>; 
}
</code></pre>
<p data-nodeid="203473">Dubbo 会生成一个 Transporter$Adaptive 适配器类，该类继承了 Transporter 接口：</p>
<pre class="lang-java" data-nodeid="203474"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Transporter</span>$<span class="hljs-title">Adaptive</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Transporter</span> </span>{ 
    <span class="hljs-keyword">public</span> org.apache.dubbo.remoting.<span class="hljs-function">Client <span class="hljs-title">connect</span><span class="hljs-params">(URL arg0, ChannelHandler arg1)</span> <span class="hljs-keyword">throws</span> RemotingException </span>{ 
        <span class="hljs-comment">// 必须传递URL参数 </span>
        <span class="hljs-keyword">if</span> (arg0 == <span class="hljs-keyword">null</span>) <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"url == null"</span>); 
        URL url = arg0; 
        <span class="hljs-comment">// 确定扩展名，优先从URL中的client参数获取，其次是transporter参数 </span>
        <span class="hljs-comment">// 这两个参数名称由@Adaptive注解指定，最后是@SPI注解中的默认值 </span>
        String extName = url.getParameter(<span class="hljs-string">"client"</span>,
            url.getParameter(<span class="hljs-string">"transporter"</span>, <span class="hljs-string">"netty"</span>)); 
        <span class="hljs-keyword">if</span> (extName == <span class="hljs-keyword">null</span>) 
            <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"..."</span>); 
        <span class="hljs-comment">// 通过ExtensionLoader加载Transporter接口的指定扩展实现 </span>
        Transporter extension = (Transporter) ExtensionLoader 
              .getExtensionLoader(Transporter.class) 
                    .getExtension(extName); 
        <span class="hljs-keyword">return</span> extension.connect(arg0, arg1); 
    } 
    ... <span class="hljs-comment">// 省略bind()方法 </span>
}
</code></pre>
<p data-nodeid="203475">生成 Transporter$Adaptive 这个类的逻辑位于 ExtensionLoader.createAdaptiveExtensionClass() 方法，若感兴趣你可以看一下相关代码，其中涉及的 javassist 等方面的知识，在后面的课时中我们会进行介绍。</p>
<p data-nodeid="203476">明确了 @Adaptive 注解的作用之后，我们回到 ExtensionLoader.createExtension() 方法，其中在扫描 SPI 配置文件的时候，会调用 loadClass() 方法加载 SPI 配置文件中指定的类，如下图所示：</p>
<p data-nodeid="203477"><img src="https://s0.lgstatic.com/i/image/M00/3E/A5/CgqCHl8s-H2AJE1LAACILXqbtHY819.png" alt="Drawing 4.png" data-nodeid="203713"></p>
<p data-nodeid="203478">loadClass() 方法中会识别加载扩展实现类上的 @Adaptive 注解，将该扩展实现的类型缓存到 cachedAdaptiveClass 这个实例字段上（volatile修饰）：</p>
<pre class="lang-java" data-nodeid="203479"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">loadClass</span><span class="hljs-params">()</span></span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (clazz.isAnnotationPresent(Adaptive.class)) { 
        // 缓存到cachedAdaptiveClass字段 
&nbsp; &nbsp; &nbsp; &nbsp; cacheAdaptiveClass(clazz, overridden);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> ... <span class="hljs-comment">// 省略其他分支 </span>
}
</code></pre>
<p data-nodeid="203480">我们可以通过 ExtensionLoader.getAdaptiveExtension() 方法获取适配器实例，并将该实例缓存到 cachedAdaptiveInstance 字段（Holder类型）中，核心流程如下：</p>
<ul data-nodeid="203481">
<li data-nodeid="203482">
<p data-nodeid="203483">首先，检查 cachedAdaptiveInstance 字段中是否已缓存了适配器实例，如果已缓存，则直接返回该实例即可。</p>
</li>
<li data-nodeid="203484">
<p data-nodeid="203485">然后，调用 getExtensionClasses() 方法，其中就会触发前文介绍的 loadClass() 方法，完成 cachedAdaptiveClass 字段的填充。</p>
</li>
<li data-nodeid="203486">
<p data-nodeid="203487">如果存在 @Adaptive 注解修饰的扩展实现类，该类就是适配器类，通过 newInstance() 将其实例化即可。如果不存在 @Adaptive 注解修饰的扩展实现类，就需要通过 createAdaptiveExtensionClass() 方法扫描扩展接口中方法上的 @Adaptive 注解，动态生成适配器类，然后实例化。</p>
</li>
<li data-nodeid="203488">
<p data-nodeid="203489">接下来，调用 injectExtension() 方法进行自动装配，就能得到一个完整的适配器实例。</p>
</li>
<li data-nodeid="203490">
<p data-nodeid="203491">最后，将适配器实例缓存到 cachedAdaptiveInstance 字段，然后返回适配器实例。</p>
</li>
</ul>
<p data-nodeid="203492">getAdaptiveExtension() 方法的流程涉及多个方法，这里不再粘贴代码，感兴趣的同学可以参考上述流程分析相应源码。</p>
<p data-nodeid="203493">此外，我们还可以通过 API 方式（addExtension() 方法）设置 cachedAdaptiveClass 这个字段，指定适配器类型（这个方法你知道即可）。</p>
<p data-nodeid="203494">总之，适配器什么实际工作都不用做，就是根据参数和状态选择其他实现来完成工作。 。</p>
<h4 data-nodeid="203495">3. 自动包装特性</h4>
<p data-nodeid="203496">Dubbo 中的一个扩展接口可能有多个扩展实现类，这些扩展实现类可能会包含一些相同的逻辑，如果在每个实现类中都写一遍，那么这些重复代码就会变得很难维护。Dubbo 提供的自动包装特性，就可以解决这个问题。 Dubbo 将多个扩展实现类的公共逻辑，抽象到 Wrapper 类中，Wrapper 类与普通的扩展实现类一样，也实现了扩展接口，在获取真正的扩展实现对象时，在其外面包装一层 Wrapper 对象，你可以理解成一层装饰器。</p>
<p data-nodeid="203497">了解了 Wrapper 类的基本功能，我们回到 ExtensionLoader.loadClass() 方法中，可以看到：</p>
<pre class="lang-java" data-nodeid="203498"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">loadClass</span><span class="hljs-params">()</span></span>{ 
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略前面对@Adaptive注解的处理 </span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (isWrapperClass(clazz)) { <span class="hljs-comment">// ---1 </span>
&nbsp; &nbsp; &nbsp; &nbsp; cacheWrapperClass(clazz); <span class="hljs-comment">// ---2 </span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> ... <span class="hljs-comment">// 省略其他分支</span>
}
</code></pre>
<ol data-nodeid="203499">
<li data-nodeid="203500">
<p data-nodeid="203501">在 isWrapperClass() 方法中，会判断该扩展实现类是否包含拷贝构造函数（即构造函数只有一个参数且为扩展接口类型），如果包含，则为 Wrapper 类，这就是判断 Wrapper 类的标准。</p>
</li>
<li data-nodeid="203502">
<p data-nodeid="203503">将 Wrapper 类记录到 cachedWrapperClasses（Set&lt;Class&lt;?&gt;&gt;类型）这个实例字段中进行缓存。</p>
</li>
</ol>
<p data-nodeid="203504">前面在介绍 createExtension() 方法时的 4 处，有下面这段代码，其中会遍历全部 Wrapper 类并一层层包装到真正的扩展实例对象外层：</p>
<pre class="lang-java" data-nodeid="203505"><code data-language="java">Set&lt;Class&lt;?&gt;&gt; wrapperClasses = cachedWrapperClasses;
<span class="hljs-keyword">if</span> (CollectionUtils.isNotEmpty(wrapperClasses)) { 
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Class&lt;?&gt; wrapperClass : wrapperClasses) { 
&nbsp; &nbsp; &nbsp; &nbsp; instance = injectExtension((T) wrapperClass 
            .getConstructor(type).newInstance(instance)); 
&nbsp; &nbsp; } 
}
</code></pre>
<h4 data-nodeid="203506">4. 自动装配特性</h4>
<p data-nodeid="203507">在 createExtension() 方法中我们看到，Dubbo SPI 在拿到扩展实现类的对象（以及 Wrapper 类的对象）之后，还会调用 injectExtension() 方法扫描其全部 setter 方法，并根据 setter 方法的名称以及参数的类型，加载相应的扩展实现，然后调用相应的 setter 方法填充属性，这就实现了 Dubbo SPI 的自动装配特性。简单来说，自动装配属性就是在加载一个扩展点的时候，将其依赖的扩展点一并加载，并进行装配。</p>
<p data-nodeid="203508">下面简单看一下 injectExtension() 方法的具体实现：</p>
<pre class="lang-java" data-nodeid="203509"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> T <span class="hljs-title">injectExtension</span><span class="hljs-params">(T instance)</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (objectFactory == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 检测objectFactory字段 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> instance; 
&nbsp; &nbsp; } 

&nbsp; &nbsp; <span class="hljs-keyword">for</span> (Method method : instance.getClass().getMethods()) { 
&nbsp; &nbsp; &nbsp; &nbsp; ... <span class="hljs-comment">// 如果不是setter方法，忽略该方法(略) </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (method.getAnnotation(DisableInject.class) != <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>; // 如果方法上明确标注了<span class="hljs-meta">@DisableInject</span>注解，忽略该方法 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据setter方法的参数，确定扩展接口 </span>
&nbsp; &nbsp; &nbsp; &nbsp; Class&lt;?&gt; pt = method.getParameterTypes()[<span class="hljs-number">0</span>]; 
&nbsp; &nbsp; &nbsp; &nbsp; ... <span class="hljs-comment">// 如果参数为简单类型，忽略该setter方法(略) </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据setter方法的名称确定属性名称 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String property = getSetterProperty(method); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 加载并实例化扩展实现类 </span>
&nbsp; &nbsp; &nbsp; &nbsp; Object object = objectFactory.getExtension(pt, property); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (object != <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; method.invoke(instance, object); <span class="hljs-comment">// 调用setter方法进行装配 </span>
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> instance; 
}
</code></pre>
<p data-nodeid="203510">injectExtension() 方法实现的自动装配依赖了 ExtensionFactory（即 objectFactory 字段），前面我们提到过 ExtensionFactory 有 SpringExtensionFactory 和 SpiExtensionFactory 两个真正的实现（还有一个实现是 AdaptiveExtensionFactory 是适配器）。下面我们分别介绍下这两个真正的实现。</p>
<p data-nodeid="203511"><strong data-nodeid="203746">第一个，SpiExtensionFactory。</strong> 根据扩展接口获取相应的适配器，没有到属性名称：</p>
<pre class="lang-java" data-nodeid="203512"><code data-language="java"><span class="hljs-meta">@Override</span> 
<span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">getExtension</span><span class="hljs-params">(Class&lt;T&gt; type, String name)</span> </span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (type.isInterface() &amp;&amp; type.isAnnotationPresent(SPI.class)) { 
&nbsp; &nbsp; &nbsp; &nbsp; // 查找type对应的ExtensionLoader实例 
&nbsp; &nbsp; &nbsp; &nbsp; ExtensionLoader&lt;T&gt; loader = ExtensionLoader 
          .getExtensionLoader(type); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!loader.getSupportedExtensions().isEmpty()) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> loader.getAdaptiveExtension(); <span class="hljs-comment">// 获取适配器实现 </span>
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>; 
}
</code></pre>
<p data-nodeid="203513"><strong data-nodeid="203751">第二个，SpringExtensionFactory。</strong> 将属性名称作为 Spring Bean 的名称，从 Spring 容器中获取 Bean：</p>
<pre class="lang-java" data-nodeid="203514"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">getExtension</span><span class="hljs-params">(Class&lt;T&gt; type, String name)</span> </span>{ 
&nbsp; &nbsp; ... <span class="hljs-comment">// 检查:type必须为接口且必须包含@SPI注解(略) </span>
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (ApplicationContext context : CONTEXTS) { 
        <span class="hljs-comment">// 从Spring容器中查找Bean </span>
&nbsp; &nbsp; &nbsp; &nbsp; T bean = BeanFactoryUtils.getOptionalBean(context,name,type); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (bean != <span class="hljs-keyword">null</span>) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> bean; 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>; 
}
</code></pre>
<h4 data-nodeid="203515">5. @Activate注解与自动激活特性</h4>
<p data-nodeid="203516">这里以 Dubbo 中的 Filter 为例说明自动激活特性的含义，org.apache.dubbo.rpc.Filter 接口有非常多的扩展实现类，在一个场景中可能需要某几个 Filter 扩展实现类协同工作，而另一个场景中可能需要另外几个实现类一起工作。这样，就需要一套配置来指定当前场景中哪些 Filter 实现是可用的，这就是 @Activate 注解要做的事情。</p>
<p data-nodeid="203517">@Activate 注解标注在扩展实现类上，有 group、value 以及 order 三个属性。</p>
<ul data-nodeid="203518">
<li data-nodeid="203519">
<p data-nodeid="203520">group 属性：修饰的实现类是在 Provider 端被激活还是在 Consumer 端被激活。</p>
</li>
<li data-nodeid="203521">
<p data-nodeid="203522">value 属性：修饰的实现类只在 URL 参数中出现指定的 key 时才会被激活。</p>
</li>
<li data-nodeid="203523">
<p data-nodeid="203524">order 属性：用来确定扩展实现类的排序。</p>
</li>
</ul>
<p data-nodeid="203525">我们先来看 loadClass() 方法对 @Activate 的扫描，其中会将包含 @Activate 注解的实现类缓存到 cachedActivates 这个实例字段（Map&lt;String, Object&gt;类型，Key为扩展名，Value为 @Activate 注解）：</p>
<pre class="lang-java" data-nodeid="203526"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">loadClass</span><span class="hljs-params">()</span></span>{ 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (clazz.isAnnotationPresent(Adaptive.class)) { 
        // 处理<span class="hljs-meta">@Adaptive</span>注解 
&nbsp; &nbsp; &nbsp; &nbsp; cacheAdaptiveClass(clazz, overridden); 
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (isWrapperClass(clazz)) { <span class="hljs-comment">// 处理Wrapper类 </span>
&nbsp; &nbsp; &nbsp; &nbsp; cacheWrapperClass(clazz); 
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 处理真正的扩展实现类 </span>
&nbsp; &nbsp; &nbsp; &nbsp; clazz.getConstructor(); <span class="hljs-comment">// 扩展实现类必须有无参构造函数 </span>
&nbsp; &nbsp; &nbsp; &nbsp; ...<span class="hljs-comment">// 兜底:SPI配置文件中未指定扩展名称，则用类的简单名称作为扩展名(略) </span>
&nbsp; &nbsp; &nbsp; &nbsp; String[] names = NAME_SEPARATOR.split(name); 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (ArrayUtils.isNotEmpty(names)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将包含@Activate注解的实现类缓存到cachedActivates集合中 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cacheActivateClass(clazz, names[<span class="hljs-number">0</span>]); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (String n : names) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 在cachedNames集合中缓存实现类-&gt;扩展名的映射 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; cacheName(clazz, n);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 在cachedClasses集合中缓存扩展名-&gt;实现类的映射 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; saveInExtensionClass(extensionClasses, clazz, n, 
                     overridden); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; } 
}
</code></pre>
<p data-nodeid="203527">使用 cachedActivates 这个集合的地方是 getActivateExtension() 方法。首先来关注 getActivateExtension() 方法的参数：url 中包含了配置信息，values 是配置中指定的扩展名，group 为 Provider 或 Consumer。下面是 getActivateExtension() 方法的核心逻辑：</p>
<ol data-nodeid="203528">
<li data-nodeid="203529">
<p data-nodeid="203530">首先，获取默认激活的扩展集合。默认激活的扩展实现类有几个条件：①在 cachedActivates 集合中存在；②@Activate 注解指定的 group 属性与当前 group 匹配；③扩展名没有出现在 values 中（即未在配置中明确指定，也未在配置中明确指定删除）；④URL 中出现了 @Activate 注解中指定的 Key。</p>
</li>
<li data-nodeid="203531">
<p data-nodeid="203532">然后，按照 @Activate 注解中的 order 属性对默认激活的扩展集合进行排序。</p>
</li>
<li data-nodeid="203533">
<p data-nodeid="203534">最后，按序添加自定义扩展实现类的对象。</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="203535"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;T&gt; <span class="hljs-title">getActivateExtension</span><span class="hljs-params">(URL url, String[] values, 
         String group)</span> </span>{ 
&nbsp; &nbsp; List&lt;T&gt; activateExtensions = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(); 
&nbsp; &nbsp; <span class="hljs-comment">// values配置就是扩展名 </span>
&nbsp; &nbsp; List&lt;String&gt; names = values == <span class="hljs-keyword">null</span> ?
            <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(<span class="hljs-number">0</span>) : asList(values); 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!names.contains(REMOVE_VALUE_PREFIX + DEFAULT_KEY)) {<span class="hljs-comment">// ---1 </span>
&nbsp; &nbsp; &nbsp; &nbsp; getExtensionClasses(); <span class="hljs-comment">// 触发cachedActivates等缓存字段的加载 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Map.Entry&lt;String, Object&gt; entry :
                  cachedActivates.entrySet()) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String name = entry.getKey(); <span class="hljs-comment">// 扩展名 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Object activate = entry.getValue(); <span class="hljs-comment">// @Activate注解 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; String[] activateGroup, activateValue; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (activate <span class="hljs-keyword">instanceof</span> Activate) { <span class="hljs-comment">// @Activate注解中的配置 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; activateGroup = ((Activate) activate).group(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; activateValue = ((Activate) activate).value(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">continue</span>; 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isMatchGroup(group, activateGroup) <span class="hljs-comment">// 匹配group </span>
                    <span class="hljs-comment">// 没有出现在values配置中的，即为默认激活的扩展实现 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &amp;&amp; !names.contains(name)
                    <span class="hljs-comment">// 通过"-"明确指定不激活该扩展实现 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &amp;&amp; !names.contains(REMOVE_VALUE_PREFIX + name)
                    <span class="hljs-comment">// 检测URL中是否出现了指定的Key </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &amp;&amp; isActive(activateValue, url)) { 
                <span class="hljs-comment">// 加载扩展实现的实例对象，这些都是激活的 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; activateExtensions.add(getExtension(name)); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 排序 --- 2 </span>
&nbsp; &nbsp; &nbsp; &nbsp; activateExtensions.sort(ActivateComparator.COMPARATOR); 
&nbsp; &nbsp; } 

&nbsp; &nbsp; List&lt;T&gt; loadedExtensions = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(); 
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; names.size(); i++) { <span class="hljs-comment">// ---3 </span>
&nbsp; &nbsp; &nbsp; &nbsp; String name = names.get(i); 
        <span class="hljs-comment">// 通过"-"开头的配置明确指定不激活的扩展实现，直接就忽略了 </span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!name.startsWith(REMOVE_VALUE_PREFIX) 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &amp;&amp; !names.contains(REMOVE_VALUE_PREFIX + name)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (DEFAULT_KEY.equals(name)) { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (!loadedExtensions.isEmpty()) { 
                    <span class="hljs-comment">// 按照顺序，将自定义的扩展添加到默认扩展集合前面 </span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; activateExtensions.addAll(<span class="hljs-number">0</span>, loadedExtensions); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; loadedExtensions.clear(); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> { 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; loadedExtensions.add(getExtension(name)); 
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; &nbsp; &nbsp; } 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!loadedExtensions.isEmpty()) { 
        <span class="hljs-comment">// 按照顺序，将自定义的扩展添加到默认扩展集合后面 </span>
&nbsp; &nbsp; &nbsp; &nbsp; activateExtensions.addAll(loadedExtensions); 
&nbsp; &nbsp; } 
&nbsp; &nbsp; <span class="hljs-keyword">return</span> activateExtensions; 
}
</code></pre>
<p data-nodeid="203536">最后举个简单的例子说明上述处理流程，假设 cachedActivates 集合缓存的扩展实现如下表所示：</p>
<p data-nodeid="203537"><img src="https://s0.lgstatic.com/i/image/M00/3E/CB/CgqCHl8tNGCAIw8fAACXC_dle_g809.png" alt="11.png" data-nodeid="203770"></p>
<p data-nodeid="203538">在 Provider 端调用 getActivateExtension() 方法时传入的 values 配置为 "demoFilter3、-demoFilter2、default、demoFilter1"，那么根据上面的逻辑：</p>
<ol data-nodeid="203539">
<li data-nodeid="203540">
<p data-nodeid="203541">得到默认激活的扩展实实现集合中有 [ demoFilter4, demoFilter6 ]；</p>
</li>
<li data-nodeid="203542">
<p data-nodeid="203543">排序后为 [ demoFilter6, demoFilter4 ]；</p>
</li>
<li data-nodeid="203544">
<p data-nodeid="203545">按序添加自定义扩展实例之后得到 [ demoFilter3, demoFilter6, demoFilter4, demoFilter1 ]。</p>
</li>
</ol>
<h3 data-nodeid="203546">总结</h3>
<p data-nodeid="203547">本课时我们深入全面地讲解了 Dubbo SPI 的核心实现：首先介绍了 @SPI 注解的底层实现，这是 Dubbo SPI 最核心的基础；然后介绍了 @Adaptive 注解与动态生成适配器类的核心原理和实现；最后分析了 Dubbo SPI 中的自动包装和自动装配特性，以及 @Activate 注解的原理。</p>
<p data-nodeid="203548">Dubbo SPI 是 Dubbo 框架实现扩展机制的核心，希望你仔细研究其实现，为后续源码分析过程打下基础。</p>
<p data-nodeid="203549" class="">也欢迎你在留言区分享你的学习心得和实践经验。</p>

---

### 精选评论

##### **安：
> 这节课有点难

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 同学，要加油哦~

##### **斌：
> 扩展点下的扩展实现类 分为三种：第一种为普通的扩展实现类，第二种为adpative类型的扩展实现类用于根据url和状态来选择合适的扩展实现，第三种 为 wrapper类，也是继承了扩展类接口，并有一个拷贝构造函数参数类型为扩展类类型，用来实现AOP功能。通过extensionFactory实现依赖注入其他扩展实现包括spring的bean。总结了下这应该就是dubbo spi的核心

##### **杰：
> 看懂spi代码花了一天 可能是水平太差了

##### *镇：
> injectExtension（）方法进行扩展点实现类相互依赖自动注入 老师在什么情况下会需要实现类 相互依赖注入尼，

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在通过SPI创建一个对象的时候，如果这个对象包含了其他的可以注册的字段，都会通过这里注入

##### *镇：
> 老师例子中的demoFilter3为什么会排在第一位置尼

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 示例中传入的values集合，明确指定了demoFilter3在第一位

##### *震：
> 内容有点儿多，感觉消化不了，再看一遍~

