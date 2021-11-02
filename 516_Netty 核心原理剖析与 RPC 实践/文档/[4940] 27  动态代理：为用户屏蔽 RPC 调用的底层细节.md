<p data-nodeid="499" class="">动态代理在 RPC 框架的实现中起到了至关重要的作用，它可以帮助用户屏蔽 RPC 调用时底层网络通信、服务发现、负载均衡等具体细节，这些对用户来说并没有什么意义。你在平时项目开发中使用 RPC 框架的时候，只需要调用接口方法，然后就拿到了返回结果，你是否好奇 RPC 框架是如何完成整个调用流程的呢？今天这节课我们就一起来完成 RPC 框架的最后一部分内容：RPC 请求调用和处理，看看如何使用动态代理机制完成这个神奇的操作。</p>
<blockquote data-nodeid="500">
<p data-nodeid="501">源码参考地址：<a href="https://github.com/wangyapu/mini-rpc" data-nodeid="580">mini-rpc</a></p>
</blockquote>
<h3 data-nodeid="502">动态代理基础</h3>
<p data-nodeid="503">为什么需要代理模式呢？代理模式的优势是可以很好地遵循设计模式中的开放封闭原则，对扩展开发，对修改关闭。你不需要关注目标类的实现细节，通过代理模式可以在不修改目标类的情况下，增强目标类功能的行为。Spring AOP 是 Java 动态代理机制的经典运用，我们在项目开发中经常使用 AOP 技术完成一些切面服务，如耗时监控、事务管理、权限校验等，所有操作都是通过切面扩展实现的，不需要对源代码有所侵入。</p>
<p data-nodeid="504">动态代理是一种代理模式，它提供了一种能够在运行时动态构建代理类以及动态调用目标方法的机制。为什么称为动态是因为代理类和被代理对象的关系是在运行时决定的，代理类可以看作是对被代理对象的包装，对目标方法的调用是通过代理类来完成的。所以通过代理模式可以有效地将服务提供者和服务消费者进行解耦，隐藏了 RPC 调用的具体细节，如下图所示。</p>
<p data-nodeid="2030" class=""><img src="https://s0.lgstatic.com/i/image/M00/8D/ED/Ciqc1GABMWuAQoyjAAG3EtPY5lU539.png" alt="图片1.png" data-nodeid="2033"></p>





<p data-nodeid="506">接下来我们一起探讨下动态代理的实现原理，以及常用的 JDK 动态代理、Cglib 动态代理是如何使用的。</p>
<h4 data-nodeid="507">JDK 动态代理</h4>
<p data-nodeid="508" class="te-preview-highlight">JDK 动态代理实现依赖 java.lang.reflect 包中的两个核心类：<strong data-nodeid="598">InvocationHandler 接口</strong>和<strong data-nodeid="599">Proxy 类</strong>。</p>
<ul data-nodeid="509">
<li data-nodeid="510">
<p data-nodeid="511"><strong data-nodeid="603">InvocationHandler 接口</strong></p>
</li>
</ul>
<p data-nodeid="512">JDK 动态代理所代理的对象必须实现一个或者多个接口，生成的代理类也是接口的实现类，然后通过 JDK 动态代理是通过反射调用的方式代理类中的方法，不能代理接口中不存在的方法。每一个动态代理对象必须提供 InvocationHandler 接口的实现类，InvocationHandler 接口中只有一个 invoke() 方法。当我们使用代理对象调用某个方法的时候，最终都会被转发到 invoke() 方法执行具体的逻辑。invoke() 方法的定义如下：</p>
<pre class="lang-java" data-nodeid="513"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">InvocationHandler</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">invoke</span><span class="hljs-params">(Object proxy, Method method, Object[] args)</span> <span class="hljs-keyword">throws</span> Throwable</span>;
}
</code></pre>
<p data-nodeid="514">其中 proxy 参数表示需要代理的对象，method 参数表示代理对象被调用的方法，args 参数为被调用方法所需的参数。</p>
<ul data-nodeid="515">
<li data-nodeid="516">
<p data-nodeid="517"><strong data-nodeid="609">Proxy 类</strong></p>
</li>
</ul>
<p data-nodeid="518">Proxy 类可以理解为动态创建代理类的工厂类，它提供了一组静态方法和接口用于动态生成对象和代理类。通常我们只需要使用 newProxyInstance() 方法，方法定义如下所示。</p>
<pre class="lang-java" data-nodeid="519"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Object <span class="hljs-title">newProxyInstance</span><span class="hljs-params">(ClassLoader loader, Class&lt;?&gt;[] interfaces, InvocationHandler h)</span> </span>{
&nbsp; &nbsp; Objects.requireNonNull(h);
&nbsp; &nbsp; Class&lt;?&gt; caller = System.getSecurityManager() == <span class="hljs-keyword">null</span> ? <span class="hljs-keyword">null</span> : Reflection.getCallerClass();
&nbsp; &nbsp; Constructor&lt;?&gt; cons = getProxyConstructor(caller, loader, interfaces);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> newProxyInstance(caller, cons, h);
}
</code></pre>
<p data-nodeid="520">其中 loader 参数表示需要装载的类加载器 ClassLoader，interfaces 参数表示代理类实现的接口列表，然后你还需要提供一个 InvocationHandler 接口类型的处理器，所有动态代理类的方法调用都会交由该处理器进行处理，这是动态代理的核心所在。</p>
<p data-nodeid="521">下面我们用一个简单的例子模拟数据库操作的事务管理，从而学习 JDK 动态代理的具体使用方式。首先我们定义数据库表 User 的接口以及实现类：</p>
<pre class="lang-java" data-nodeid="522"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDao</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">insert</span><span class="hljs-params">()</span></span>;
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserDaoImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDao</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">insert</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"insert user success."</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="523">接下来我们实现一个事务管理的工具类，在数据库操作执行前后执行事务操作，代码如下所示：</p>
<pre class="lang-java" data-nodeid="524"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TransactionProxy</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> Object target;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">TransactionProxy</span><span class="hljs-params">(Object target)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.target = target;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">genProxyInstance</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> Proxy.newProxyInstance(target.getClass().getClassLoader(),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; target.getClass().getInterfaces(),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; (proxy, method, args) -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"start transaction"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Object result = method.invoke(target, args);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"submit transaction"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> result;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="525">在 genProxyInstance() 方法中我们最主要的是实现 InvocationHandler 接口，在真实对象方法执行方法调用的前后可以扩展自定义行为，以此来增强目标类的功能。为了便于理解，上述例子中我们只简单打印了控制台日志，可以通过测试类看看 JDK 动态代理的实际效果：</p>
<pre class="lang-java" data-nodeid="526"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TransactionProxyTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Test</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testProxy</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; UserDao userDao = <span class="hljs-keyword">new</span> UserDaoImpl();
&nbsp; &nbsp; &nbsp; &nbsp; UserDao proxyInstance = (UserDao) <span class="hljs-keyword">new</span> TransactionProxy(userDao).genProxyInstance();
&nbsp; &nbsp; &nbsp; &nbsp; proxyInstance.insert();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="527">程序运行结果如下：</p>
<pre class="lang-java" data-nodeid="528"><code data-language="java">start transaction
insert user success.
submit transaction
</code></pre>
<h4 data-nodeid="529">Cglib 动态代理</h4>
<p data-nodeid="530">Cglib 动态代理是基于 ASM 字节码生成框架实现的第三方工具类库，相比于 JDK 动态代理，Cglib 动态代理更加灵活，它是通过字节码技术生成的代理类，所以代理类的类型是不受限制的。使用 Cglib 代理的目标类无须实现任何接口，可以做到对目标类零侵入。</p>
<p data-nodeid="531">Cglib 动态代理是对指定类以字节码的方式生成一个子类，并重写其中的方法，以此来实现动态代理。因为 Cglib 动态代理创建的是目标类的子类，所以目标类必须要有无参构造函数，而且目标类不要用 final 进行修饰。</p>
<p data-nodeid="532">在我们使用 Cglib 动态代理之前，需要引入相关的 Maven 依赖，如下所示。如果你的项目中已经引入了 spring-core 的依赖，则已经包含了 Cglib 的相关依赖，无须再次引入。</p>
<pre class="lang-xml" data-nodeid="533"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>cglib<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>cglib<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp; &nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>3.3.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="534">下面我们还是使用上述数据库事务管理的例子，从而学习 JDK 动态代理的具体使用方式。UserDao 接口和实现类保持不变，TransactionProxy 需要重新实现，代码如下所示：</p>
<pre class="lang-java" data-nodeid="535"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CglibTransactionProxy</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">MethodInterceptor</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> Object target;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CglibTransactionProxy</span><span class="hljs-params">(Object target)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.target = target;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">genProxyInstance</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Enhancer enhancer = <span class="hljs-keyword">new</span> Enhancer();
&nbsp; &nbsp; &nbsp; &nbsp; enhancer.setSuperclass(target.getClass());
&nbsp; &nbsp; &nbsp; &nbsp; enhancer.setCallback(<span class="hljs-keyword">this</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> enhancer.create();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">intercept</span><span class="hljs-params">(Object object, Method method, Object[] args, MethodProxy methodProxy)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"start transaction"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; Object result = methodProxy.invokeSuper(object, args);
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"submit transaction"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> result;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="536">Cglib 动态代理的实现需要依赖两个核心组件：MethodInterceptor 接口和 Enhancer 类，类似于 JDK 动态代理中的<strong data-nodeid="630">InvocationHandler 接口</strong>和<strong data-nodeid="631">Proxy 类</strong>。</p>
<ul data-nodeid="537">
<li data-nodeid="538">
<p data-nodeid="539"><strong data-nodeid="635">MethodInterceptor 接口</strong></p>
</li>
</ul>
<p data-nodeid="540">MethodInterceptor 接口只有 intercept() 一个方法，所有被代理类的方法执行最终都会转移到 intercept() 方法中进行行为增强，真实方法的执行逻辑则通过 Method 或者 MethodProxy 对象进行调用。</p>
<ul data-nodeid="541">
<li data-nodeid="542">
<p data-nodeid="543"><strong data-nodeid="640">Enhancer 类</strong></p>
</li>
</ul>
<p data-nodeid="544">Enhancer 类是 Cglib 中的一个字节码增强器，它为我们对代理类进行扩展时提供了极大的便利。Enhancer 类的本质是在运行时动态为代理类生成一个子类，并且拦截代理类中的所有方法。我们可以通过 Enhancer 设置 Callback 接口对代理类方法执行的前后执行一些自定义行为，其中 MethodInterceptor 接口是我们最常用的 Callback 操作。</p>
<p data-nodeid="545">Cglib 动态代理的测试类与 JDK 动态代理测试类大同小异，程序输出结果也是一样的。测试类代码如下所示：</p>
<pre class="lang-java" data-nodeid="546"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CglibTransactionProxyTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; UserDao userDao = <span class="hljs-keyword">new</span> UserDaoImpl();
&nbsp; &nbsp; &nbsp; &nbsp; UserDao proxyInstance = (UserDao) <span class="hljs-keyword">new</span> CglibTransactionProxy(userDao).genProxyInstance();
&nbsp; &nbsp; &nbsp; &nbsp; proxyInstance.insert();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="547">学习完动态代理的基础后，我们接下来实现 RPC 框架中的请求调用和处理就易如反掌啦，首先我们先从服务消费者如何通过动态代理发起 RPC 请求入手。</p>
<h3 data-nodeid="548">服务消费者动态代理实现</h3>
<p data-nodeid="549">在《服务发布与订阅：搭建生产者和消费者的基础框架》课程中，我们讲解了 @RpcReference 注解的实现过程。通过一个自定义的 RpcReferenceBean 完成了所有执行方法的拦截，RpcReferenceBean 中 init() 方法是当时留下的 TODO 内容，这里就是代理对象的创建入口，代理对象创建如下所示.</p>
<pre class="lang-java" data-nodeid="550"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RpcReferenceBean</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">FactoryBean</span>&lt;<span class="hljs-title">Object</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>

&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; RegistryService registryService = RegistryFactory.getInstance(<span class="hljs-keyword">this</span>.registryAddr, RegistryType.valueOf(<span class="hljs-keyword">this</span>.registryType));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.object = Proxy.newProxyInstance(
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; interfaceClass.getClassLoader(),
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> Class&lt;?&gt;[]{interfaceClass},
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">new</span> RpcInvokerProxy(serviceVersion, timeout, registryService));
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="551">RpcInvokerProxy 处理器是实现动态代理逻辑的核心所在，其中包含 RPC 调用时底层网络通信、服务发现、负载均衡等具体细节，我们详细看下如何实现 RpcInvokerProxy 处理器，代码如下所示：</p>
<pre class="lang-java" data-nodeid="552"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RpcInvokerProxy</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">InvocationHandler</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String serviceVersion;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> timeout;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> RegistryService registryService;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">RpcInvokerProxy</span><span class="hljs-params">(String serviceVersion, <span class="hljs-keyword">long</span> timeout, RegistryService registryService)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.serviceVersion = serviceVersion;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.timeout = timeout;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.registryService = registryService;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">invoke</span><span class="hljs-params">(Object proxy, Method method, Object[] args)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 构造 RPC 协议对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; MiniRpcProtocol&lt;MiniRpcRequest&gt; protocol = <span class="hljs-keyword">new</span> MiniRpcProtocol&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; MsgHeader header = <span class="hljs-keyword">new</span> MsgHeader();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">long</span> requestId = MiniRpcRequestHolder.REQUEST_ID_GEN.incrementAndGet();
&nbsp; &nbsp; &nbsp; &nbsp; header.setMagic(ProtocolConstants.MAGIC);
&nbsp; &nbsp; &nbsp; &nbsp; header.setVersion(ProtocolConstants.VERSION);
&nbsp; &nbsp; &nbsp; &nbsp; header.setRequestId(requestId);
&nbsp; &nbsp; &nbsp; &nbsp; header.setSerialization((<span class="hljs-keyword">byte</span>) SerializationTypeEnum.HESSIAN.getType());
&nbsp; &nbsp; &nbsp; &nbsp; header.setMsgType((<span class="hljs-keyword">byte</span>) MsgType.REQUEST.getType());
&nbsp; &nbsp; &nbsp; &nbsp; header.setStatus((<span class="hljs-keyword">byte</span>) <span class="hljs-number">0x1</span>);
&nbsp; &nbsp; &nbsp; &nbsp; protocol.setHeader(header);
&nbsp; &nbsp; &nbsp; &nbsp; MiniRpcRequest request = <span class="hljs-keyword">new</span> MiniRpcRequest();
&nbsp; &nbsp; &nbsp; &nbsp; request.setServiceVersion(<span class="hljs-keyword">this</span>.serviceVersion);
&nbsp; &nbsp; &nbsp; &nbsp; request.setClassName(method.getDeclaringClass().getName());
&nbsp; &nbsp; &nbsp; &nbsp; request.setMethodName(method.getName());
&nbsp; &nbsp; &nbsp; &nbsp; request.setParameterTypes(method.getParameterTypes());
&nbsp; &nbsp; &nbsp; &nbsp; request.setParams(args);
&nbsp; &nbsp; &nbsp; &nbsp; protocol.setBody(request);
&nbsp; &nbsp; &nbsp; &nbsp; RpcConsumer rpcConsumer = <span class="hljs-keyword">new</span> RpcConsumer();
&nbsp; &nbsp; &nbsp; &nbsp; MiniRpcFuture&lt;MiniRpcResponse&gt; future = <span class="hljs-keyword">new</span> MiniRpcFuture&lt;&gt;(<span class="hljs-keyword">new</span> DefaultPromise&lt;&gt;(<span class="hljs-keyword">new</span> DefaultEventLoop()), timeout);
&nbsp; &nbsp; &nbsp; &nbsp; MiniRpcRequestHolder.REQUEST_MAP.put(requestId, future);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 发起 RPC 远程调用</span>
&nbsp; &nbsp; &nbsp; &nbsp; rpcConsumer.sendRequest(protocol, <span class="hljs-keyword">this</span>.registryService);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 等待 RPC 调用执行结果</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> future.getPromise().get(future.getTimeout(), TimeUnit.MILLISECONDS).getData();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="553">RpcInvokerProxy 处理器必须要实现 InvocationHandler 接口的 invoke() 方法，被代理的 RPC 接口在执行方法调用时，都会转发到 invoke() 方法上。invoke() 方法的核心流程主要分为三步：构造 RPC 协议对象、发起 RPC 远程调用、等待 RPC 调用执行结果。</p>
<p data-nodeid="554">RPC 协议对象的构建，只要根据用户配置的接口参数对 MiniRpcProtocol 类的属性依次赋值即可。构建完MiniRpcProtocol 协议对象后，就可以对远端服务节点发起 RPC 调用了，所以 sendRequest() 方法是我们需要重点实现的内容。</p>
<pre class="lang-java" data-nodeid="555"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendRequest</span><span class="hljs-params">(MiniRpcProtocol&lt;MiniRpcRequest&gt; protocol, RegistryService registryService)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; MiniRpcRequest request = protocol.getBody();
&nbsp; &nbsp; Object[] params = request.getParams();
&nbsp; &nbsp; String serviceKey = RpcServiceHelper.buildServiceKey(request.getClassName(), request.getServiceVersion());
&nbsp; &nbsp; <span class="hljs-keyword">int</span> invokerHashCode = params.length &gt; <span class="hljs-number">0</span> ? params[<span class="hljs-number">0</span>].hashCode() : serviceKey.hashCode();
&nbsp; &nbsp; ServiceMeta serviceMetadata = registryService.discovery(serviceKey, invokerHashCode);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (serviceMetadata != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; ChannelFuture future = bootstrap.connect(serviceMetadata.getServiceAddr(), serviceMetadata.getServicePort()).sync();
&nbsp; &nbsp; &nbsp; &nbsp; future.addListener((ChannelFutureListener) arg0 -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (future.isSuccess()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; log.info(<span class="hljs-string">"connect rpc server {} on port {} success."</span>, serviceMetadata.getServiceAddr(), serviceMetadata.getServicePort());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; log.error(<span class="hljs-string">"connect rpc server {} on port {} failed."</span>, serviceMetadata.getServiceAddr(), serviceMetadata.getServicePort());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; future.cause().printStackTrace();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; eventLoopGroup.shutdownGracefully();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; future.channel().writeAndFlush(protocol);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="556">发起 RPC 调用之前，我们需要找到最合适的服务节点，直接调用注册中心服务 RegistryService 的 discovery() 方法即可，默认是采用一致性 Hash 算法实现的服务发现。这里有一个小技巧，为了尽可能使所有服务节点收到的请求流量更加均匀，需要为 discovery() 提供一个 invokerHashCode，一般可以采用 RPC 服务接口参数列表中第一个参数的 hashCode 作为参考依据。找到服务节点地址后，接下来通过 Netty 建立 TCP 连接，然后调用 writeAndFlush() 方法将数据发送到远端服务节点。</p>
<p data-nodeid="557">再次回到 invoke() 方法的主流程，发送 RPC 远程调用后如何等待调用结果返回呢？在《远程通信：通信协议设计以及编解码的实现》课程中，我们介绍了如何使用 Netty 提供的 Promise 工具来实现 RPC 请求的同步等待，Promise 模式本质是一种异步编程模型，我们可以先拿到一个查看任务执行结果的凭证，不必等待任务执行完毕，当我们需要获取任务执行结果时，再使用凭证提供的相关接口进行获取。</p>
<p data-nodeid="558">当服务提供者收到 RPC 请求后，又应该如何执行真实的方法调用呢？接下来我们继续看下服务提供者如何处理 RPC 请求。</p>
<h3 data-nodeid="559">服务提供者反射调用实现</h3>
<p data-nodeid="560">在《远程通信：通信协议设计以及编解码的实现》课程中，我们已经介绍了服务提供者的 Handler 处理器，RPC 请求数据经过 MiniRpcDecoder 解码成 MiniRpcProtocol 对象后，再交由 RpcRequestHandler 执行 RPC 请求调用。一起先来回顾下 RpcRequestHandler 中 channelRead0() 方法的处理逻辑：<br>
<br></p>
<pre class="lang-java" data-nodeid="561"><code data-language="java"><span class="hljs-meta">@Slf4j</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RpcRequestHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">SimpleChannelInboundHandler</span>&lt;<span class="hljs-title">MiniRpcProtocol</span>&lt;<span class="hljs-title">MiniRpcRequest</span>&gt;&gt; </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Map&lt;String, Object&gt; rpcServiceMap;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">RpcRequestHandler</span><span class="hljs-params">(Map&lt;String, Object&gt; rpcServiceMap)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.rpcServiceMap = rpcServiceMap;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">channelRead0</span><span class="hljs-params">(ChannelHandlerContext ctx, MiniRpcProtocol&lt;MiniRpcRequest&gt; protocol)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; RpcRequestProcessor.submitRequest(() -&gt; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MiniRpcProtocol&lt;MiniRpcResponse&gt; resProtocol = <span class="hljs-keyword">new</span> MiniRpcProtocol&lt;&gt;();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MiniRpcResponse response = <span class="hljs-keyword">new</span> MiniRpcResponse();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; MsgHeader header = protocol.getHeader();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; header.setMsgType((<span class="hljs-keyword">byte</span>) MsgType.RESPONSE.getType());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Object result = handle(protocol.getBody());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; response.setData(result);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; header.setStatus((<span class="hljs-keyword">byte</span>) MsgStatus.SUCCESS.getCode());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; resProtocol.setHeader(header);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; resProtocol.setBody(response);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable throwable) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; header.setStatus((<span class="hljs-keyword">byte</span>) MsgStatus.FAIL.getCode());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; response.setMessage(throwable.toString());
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; log.error(<span class="hljs-string">"process request {} error"</span>, header.getRequestId(), throwable);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ctx.writeAndFlush(resProtocol);
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="562">因为 RPC 请求调用是比较耗时的，推荐的做法是将 RPC 请求提交到自定义的业务线程池中执行。其中 handle() 方法是真正执行 RPC 调用的地方，是我们这节课需要实现的内容，handle() 方法的实现如下所示：</p>
<pre class="lang-java" data-nodeid="563"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Object <span class="hljs-title">handle</span><span class="hljs-params">(MiniRpcRequest request)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; String serviceKey = RpcServiceHelper.buildServiceKey(request.getClassName(), request.getServiceVersion());
&nbsp; &nbsp; Object serviceBean = rpcServiceMap.get(serviceKey);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (serviceBean == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RuntimeException(String.format(<span class="hljs-string">"service not exist: %s:%s"</span>, request.getClassName(), request.getMethodName()));
&nbsp; &nbsp; }
&nbsp; &nbsp; Class&lt;?&gt; serviceClass = serviceBean.getClass();
&nbsp; &nbsp; String methodName = request.getMethodName();
&nbsp; &nbsp; Class&lt;?&gt;[] parameterTypes = request.getParameterTypes();
&nbsp; &nbsp; Object[] parameters = request.getParams();
&nbsp; &nbsp; FastClass fastClass = FastClass.create(serviceClass);
&nbsp; &nbsp; <span class="hljs-keyword">int</span> methodIndex = fastClass.getIndex(methodName, parameterTypes);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> fastClass.invoke(methodIndex, serviceBean, parameters);
}
</code></pre>
<p data-nodeid="564">rpcServiceMap 中存放着服务提供者所有对外发布的服务接口，我们可以通过服务名和服务版本找到对应的服务接口。通过服务接口、方法名、方法参数列表、参数类型列表，我们一般可以使用反射的方式执行方法调用。为了加速服务接口调用的性能，我们采用 Cglib 提供的 FastClass 机制直接调用方法，Cglib 中 MethodProxy 对象就是采用了 FastClass 机制，它可以和 Method 对象完成同样的事情，但是相比于反射性能更高。</p>
<p data-nodeid="565">FastClass 机制并没有采用反射的方式调用被代理的方法，而是运行时动态生成一个新的 FastClass 子类，向子类中写入直接调用目标方法的逻辑。同时该子类会为代理类分配一个 int 类型的 index 索引，FastClass 即可通过 index 索引定位到需要调用的方法。</p>
<p data-nodeid="566">至此，整个 RPC 框架的原型我们已经实现完毕。你可以在本地先启动 Zookeeper 服务器，然后启动 rpc-provider、rpc-consumer 两个模块，通过 HTTP 请求发起测试，如下所示：</p>
<pre class="lang-java" data-nodeid="567"><code data-language="java">$ curl http:<span class="hljs-comment">//localhost:8080/hello</span>
hellomini rpc
</code></pre>
<h3 data-nodeid="568">总结</h3>
<p data-nodeid="569">本节课我们介绍了动态代理的基本原理，并使用动态代理技术完成了 RPC 请求的调用和处理。动态代理技术是 RPC 框架的核心技术之一，也是很重要的一个性能优化点。选择哪种动态代理技术需要根据场景有的放矢，实践出真知，在技术选型时还是要做好性能测试。例如，在 JDK 1.8 版本之后 JDK 动态代理在运行多次之后比 Cglib 的速度更快了，但是它还是有使用的局限性；虽然 Javassist 字节码生成的性能相比 JDK 动态代理和 Cglib 动态代理更好，但是 Javassist 在生成动态代理类上性能较慢的。</p>
<p data-nodeid="570">留两个课后任务：</p>
<ul data-nodeid="571">
<li data-nodeid="572">
<p data-nodeid="573">Dubbo 框架默认使用 Javassist 实现动态代理功能，你可以将 JDK 动态代理的方式替换为 Javassist 的实现方式。</p>
</li>
<li data-nodeid="574">
<p data-nodeid="575" class="">服务消费者每次发起 RPC 调用时都建立了一次 TCP 连接，你知道怎么优化吗？</p>
</li>
</ul>

---

### 精选评论

##### Q：
> 这里每发一个请求，都创建一个channel，搞个channel管理池统一管理好维护更好些吧

##### **霖：
> 第二个问题，当然是考虑用连接池啦

