<p data-nodeid="3636" class="">设计模式的运用是面试过程中常考的，学习设计模式切勿死记硬背，结合优秀项目的源码去理解设计模式的使用会事半功倍。Netty 源码中运用了大量的设计模式，常见的设计模式在 Netty 源码中都有所体现。本节课我们便一起梳理 Netty 源码中所包含的设计模式，希望能帮助你更深入地了解 Netty 的设计精髓，并可以结合 Netty 源码向面试官讲述你对设计模式的理解。</p>
<h3 data-nodeid="3637">单例模式</h3>
<p data-nodeid="3638">单例模式是最常见的设计模式，它可以保证全局只有一个实例，避免线程安全问题。单例模式有很多种实现方法，其中我比较推荐三种最佳实践：<strong data-nodeid="3737">双重检验锁</strong>、<strong data-nodeid="3738">静态内部类方式</strong>、<strong data-nodeid="3739">饿汉方式</strong>和<strong data-nodeid="3740">枚举方式</strong>，其中双重检验锁和静态内部类方式属于懒汉式单例，饿汉方式和枚举方式属于饿汉式单例。</p>
<h4 data-nodeid="3639">双重检验锁</h4>
<p data-nodeid="3640">在多线程环境下，为了提高实例初始化的性能，不是每次获取实例时在方法上加锁，而是当实例未创建时才会加锁，如下所示：</p>
<pre class="lang-java te-preview-highlight" data-nodeid="4228"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingletonTest</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">volatile</span> <span class="hljs-keyword">static</span> SingletonTest instance;
    
    <span class="hljs-function"><span class="hljs-keyword">public</span> SingletonTest <span class="hljs-title">getInstance</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">if</span> (instance == <span class="hljs-keyword">null</span>) {
            <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
                <span class="hljs-keyword">if</span> (instance == <span class="hljs-keyword">null</span>) {
                    instance = <span class="hljs-keyword">new</span> SingletonTest();
                }
            }
        }
        <span class="hljs-keyword">return</span> instance;
    }
}

</code></pre>


<h4 data-nodeid="3642">静态内部类方式</h4>
<p data-nodeid="3643">静态内部类方式实现单例巧妙地利用了 Java 类加载机制，保证其在多线程环境下的线程安全性。当一个类被加载时，其静态内部类是不会被同时加载的，只有第一次被调用时才会初始化，而且我们不能通过反射的方式获取内部的属性。由此可见，静态内部类方式实现单例更加安全，可以防止被反射入侵。具体实现方式如下：</p>
<pre class="lang-java" data-nodeid="3644"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingletonTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">SingletonTest</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Singleton <span class="hljs-title">getInstance</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> SingletonInstance.instance;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingletonInstance</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Singleton instance = <span class="hljs-keyword">new</span> Singleton();
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="3645">饿汉方式</h4>
<p data-nodeid="3646">饿汉式实现单例非常简单，类加载的时候就创建出实例。饿汉方式使用私有构造函数实现全局单个实例的初始化，并使用 public static final 加以修饰，实现延迟加载和保证线程安全性。实现方式如下所示：</p>
<pre class="lang-java" data-nodeid="3647"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SingletonTest</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Singleton instance = <span class="hljs-keyword">new</span> Singleton();
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">Singleton</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Singleton <span class="hljs-title">getInstance</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> instance;
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="3648">枚举方式</h4>
<p data-nodeid="3649">枚举方式是一种天然的单例实现，在项目开发中枚举方式是非常推荐使用的。它能够保证序列化和反序列化过程中实例的唯一性，而且不用担心线程安全问题。枚举方式实现单例如下所示：</p>
<pre class="lang-java" data-nodeid="3650"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">enum</span> SingletonTest {
&nbsp; &nbsp; SERVICE_A {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hello</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"hello, service A"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; },
&nbsp; &nbsp; SERVICE_B {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hello</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"hello, service B"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; };
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hello</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="3651">在《源码篇：解密 Netty Reactor 的线程模型》课程中，我们介绍了 NioEventLoop 的核心原理。NioEventLoop 通过核心方法 select() 不断轮询注册的 I/O 事件，Netty 提供了选择策略 SelectStrategy 对象，它用于控制 select 循环行为，包含 CONTINUE、SELECT、BUSY_WAIT 三种策略。SelectStrategy 对象的默认实现就是使用的饿汉式单例，源码如下：</p>
<pre class="lang-java" data-nodeid="3652"><code data-language="java"><span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultSelectStrategy</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">SelectStrategy</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> SelectStrategy INSTANCE = <span class="hljs-keyword">new</span> DefaultSelectStrategy();
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">DefaultSelectStrategy</span><span class="hljs-params">()</span> </span>{ }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">calculateStrategy</span><span class="hljs-params">(IntSupplier selectSupplier, <span class="hljs-keyword">boolean</span> hasTasks)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> hasTasks ? selectSupplier.get() : SelectStrategy.SELECT;
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3653">此外 Netty 中还有不少饿汉方式实现单例的实践，例如 MqttEncoder、ReadTimeoutException 等。</p>
<h3 data-nodeid="3654">工厂方法模式</h3>
<p data-nodeid="3655">工厂模式封装了对象创建的过程，使用者不需要关心对象创建的细节。在需要生成复杂对象的场景下，都可以使用工厂模式实现。工厂模式分为三种：<strong data-nodeid="3767">简单工厂模式</strong>、<strong data-nodeid="3768">工厂方法模式</strong>和<strong data-nodeid="3769">抽象工厂模式</strong>。</p>
<ul data-nodeid="3656">
<li data-nodeid="3657">
<p data-nodeid="3658"><strong data-nodeid="3774">简单工厂模式</strong>。定义一个工厂类，根据参数类型返回不同类型的实例。适用于对象实例类型不多的场景，如果对象实例类型太多，每增加一种类型就要在工厂类中增加相应的创建逻辑，这是违背开放封闭原则的。</p>
</li>
<li data-nodeid="3659">
<p data-nodeid="3660"><strong data-nodeid="3779">工厂方法模式</strong>。简单工厂模式的升级版，不再是提供一个统一的工厂类来创建所有对象的实例，而是每种类型的对象实例都对应不同的工厂类，每个具体的工厂类只能创建一个类型的对象实例。</p>
</li>
<li data-nodeid="3661">
<p data-nodeid="3662"><strong data-nodeid="3784">抽象工厂模式</strong>。较少使用，适用于创建多个产品的场景。如果按照工厂方法模式的实现思路，需要在具体工厂类中实现多个工厂方法，是非常不友好的。抽象工厂模式就是把这些工厂方法单独剥离到抽象工厂类中，然后创建工厂对象并通过组合的方式来获取工厂方法。</p>
</li>
</ul>
<p data-nodeid="3663">Netty 中使用的就是工厂方法模式，这也是项目开发中最常用的一种工厂模式。工厂方法模式如何使用呢？我们先来看个简单的例子：</p>
<pre class="lang-java" data-nodeid="3664"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TSLAFactory</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">CarFactory</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Car <span class="hljs-title">createCar</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TSLA();
&nbsp; &nbsp; }
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BMWFactory</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">CarFactory</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Car <span class="hljs-title">createCar</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> BMW();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3665">Netty 在创建 Channel 的时候使用的就是工厂方法模式，因为服务端和客户端的 Channel 是不一样的。Netty 将反射和工厂方法模式结合在一起，只使用一个工厂类，然后根据传入的 Class 参数来构建出对应的 Channel，不需要再为每一种 Channel 类型创建一个工厂类。具体源码实现如下：</p>
<pre class="lang-java" data-nodeid="3666"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReflectiveChannelFactory</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Channel</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">ChannelFactory</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Constructor&lt;? extends T&gt; constructor;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ReflectiveChannelFactory</span><span class="hljs-params">(Class&lt;? extends T&gt; clazz)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; ObjectUtil.checkNotNull(clazz, <span class="hljs-string">"clazz"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.constructor = clazz.getConstructor();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (NoSuchMethodException e) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"Class "</span> + StringUtil.simpleClassName(clazz) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">" does not have a public non-arg constructor"</span>, e);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">newChannel</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> constructor.newInstance();
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (Throwable t) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ChannelException(<span class="hljs-string">"Unable to create Channel from class "</span> + constructor.getDeclaringClass(), t);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> StringUtil.simpleClassName(ReflectiveChannelFactory.class) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; '(' + StringUtil.simpleClassName(constructor.getDeclaringClass()) + ".class)";
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3667">虽然通过反射技术可以有效地减少工厂类的数据量，但是反射相比直接创建工厂类有性能损失，所以对于性能敏感的场景，应当谨慎使用反射。</p>
<h3 data-nodeid="3668">责任链模式</h3>
<p data-nodeid="3669">想必学完本专栏的前面课程后，责任链模式大家应该再熟悉不过了，自然而然联想到 ChannlPipeline 和 ChannelHandler。ChannlPipeline 内部是由一组 ChannelHandler 实例组成的，内部通过双向链表将不同的 ChannelHandler 链接在一起，如下图所示。</p>
<p data-nodeid="3670"><img src="https://s0.lgstatic.com/i/image2/M01/08/2F/Cip5yGAKfLmANJ_0AAZUvQP4FxQ293.png" alt="Drawing 0.png" data-nodeid="3792"></p>
<p data-nodeid="3671">对于 Netty 中责任链模式的实现，也遵循了责任链模式的四个基本要素：</p>
<h4 data-nodeid="3672">责任处理器接口</h4>
<p data-nodeid="3673">ChannelHandler 对应的就是责任处理器接口，ChannelHandler 有两个重要的子接口：ChannelInboundHandler和ChannelOutboundHandler，分别拦截入站和出站的各种 I/O 事件。</p>
<h4 data-nodeid="3674">动态创建责任链，添加、删除责任处理器</h4>
<p data-nodeid="3675">ChannelPipeline 负责创建责任链，其内部采用双向链表实现，ChannelPipeline 的内部结构定义如下所示：</p>
<pre class="lang-java" data-nodeid="3676"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultChannelPipeline</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">ChannelPipeline</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> InternalLogger logger = InternalLoggerFactory.getInstance(DefaultChannelPipeline.class);
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String HEAD_NAME = generateName0(HeadContext.class);
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String TAIL_NAME = generateName0(TailContext.class);
&nbsp; &nbsp; // 省略其他代码
&nbsp; &nbsp; <span class="hljs-keyword">final</span> AbstractChannelHandlerContext head; // 头结点
&nbsp; &nbsp; <span class="hljs-keyword">final</span> AbstractChannelHandlerContext tail; // 尾节点
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> Channel channel;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> ChannelFuture succeededFuture;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> VoidChannelPromise voidPromise;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> touch = ResourceLeakDetector.isEnabled();

&nbsp; &nbsp; // 省略其他代码
}
</code></pre>
<p data-nodeid="3677">ChannelPipeline 提供了一系列 add 和 remove 相关接口用于动态添加和删除 ChannelHandler 处理器，如下所示：</p>
<p data-nodeid="3678"><img src="https://s0.lgstatic.com/i/image2/M01/08/31/CgpVE2AKfMSAehuyAA0_w5_fB-c556.png" alt="Drawing 1.png" data-nodeid="3801"></p>
<h4 data-nodeid="3679">上下文</h4>
<p data-nodeid="3680">从 ChannelPipeline 内部结构定义可以看出，ChannelHandlerContext 负责保存责任链节点上下文信息。ChannelHandlerContext 是对 ChannelHandler 的封装，每个 ChannelHandler 都对应一个 ChannelHandlerContext，实际上 ChannelPipeline 维护的是与 ChannelHandlerContext 的关系。</p>
<h4 data-nodeid="3681">责任传播和终止机制</h4>
<p data-nodeid="3682">ChannelHandlerContext 提供了 fire 系列的方法用于事件传播，如下所示：</p>
<p data-nodeid="3683"><img src="https://s0.lgstatic.com/i/image2/M01/08/30/Cip5yGAKfM6AHTnGAA43hnlCx54417.png" alt="Drawing 2.png" data-nodeid="3808"></p>
<p data-nodeid="3684">以 ChannelInboundHandlerAdapter 的 channelRead 方法为例，ChannelHandlerContext 会默认调用 fireChannelRead 方法将事件默认传递到下一个处理器。如果我们重写了 ChannelInboundHandlerAdapter 的 channelRead 方法，并且没有调用 fireChannelRead 进行事件传播，那么表示此次事件传播已终止。</p>
<h3 data-nodeid="3685">观察者模式</h3>
<p data-nodeid="3686">观察者模式有两个角色：观察者和被观察。被观察者发布消息，观察者订阅消息，没有订阅的观察者是收不到消息的。首先我们通过一个简单的例子看下观察者模式的是如何实现的。</p>
<pre class="lang-java" data-nodeid="3687"><code data-language="java"><span class="hljs-comment">// 被观察者</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Observable</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">registerObserver</span><span class="hljs-params">(Observer observer)</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">removeObserver</span><span class="hljs-params">(Observer observer)</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">notifyObservers</span><span class="hljs-params">(String message)</span></span>;
}
<span class="hljs-comment">// 观察者</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Observer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">notify</span><span class="hljs-params">(String message)</span></span>;
}
<span class="hljs-comment">// 默认被观察者实现</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultObservable</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Observable</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;Observer&gt; observers = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">registerObserver</span><span class="hljs-params">(Observer observer)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; observers.add(observer);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeObserver</span><span class="hljs-params">(Observer observer)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; observers.remove(observer);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">notifyObservers</span><span class="hljs-params">(String message)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span> (Observer observer : observers) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; observer.notify(message);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3688">Netty 中观察者模式的运用非常多，但是并没有以上示例代码这么直观，我们平时经常使用的ChannelFuture#addListener 接口就是观察者模式的实现。我们先来看下 ChannelFuture 使用的示例：</p>
<pre class="lang-java" data-nodeid="3689"><code data-language="java">ChannelFuture channelFuture = channel.writeAndFlush(object);
channelFuture.addListener(future -&gt; {
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (future.isSuccess()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// do something</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// do something</span>
&nbsp; &nbsp; }
});
</code></pre>
<p data-nodeid="3690">addListener 方法会将添加监听器添加到 ChannelFuture 当中，并在 ChannelFuture 执行完毕的时候立刻通知已经注册的监听器。所以 ChannelFuture 是被观察者，addListener 方法用于添加观察者。</p>
<h3 data-nodeid="3691">建造者模式</h3>
<p data-nodeid="3692">建造者模式非常简单，通过链式调用来设置对象的属性，在对象属性繁多的场景下非常有用。建造者模式的优势就是可以像搭积木一样自由选择需要的属性，并不是强绑定的。对于使用者来说，必须清楚需要设置哪些属性，在不同场景下可能需要的属性也是不一样的。</p>
<p data-nodeid="3693">Netty 中 ServerBootStrap 和 Bootstrap 引导器是最经典的建造者模式实现，在构建过程中需要设置非常多的参数，例如配置线程池 EventLoopGroup、设置 Channel 类型、注册 ChannelHandler、设置 Channel 参数、端口绑定等。ServerBootStrap 引导器的具体使用可以参考《引导器作用：客户端和服务端启动都要做些什么？》课程，在此我就不多作赘述了。</p>
<h3 data-nodeid="3694">策略模式</h3>
<p data-nodeid="3695">策略模式针对同一个问题提供多种策略的处理方式，这些策略之间可以相互替换，在一定程度上提高了系统的灵活性。策略模式非常符合开闭原则，使用者在不修改现有系统的情况下选择不同的策略，而且便于扩展增加新的策略。</p>
<p data-nodeid="3696">Netty 在多处地方使用了策略模式，例如 EventExecutorChooser 提供了不同的策略选择 NioEventLoop，newChooser() 方法会根据线程池的大小是否是 2 的幂次，以此来动态的选择取模运算的方式，从而提高性能。EventExecutorChooser 源码实现如下所示：</p>
<pre class="lang-java" data-nodeid="3697"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DefaultEventExecutorChooserFactory</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">EventExecutorChooserFactory</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> DefaultEventExecutorChooserFactory INSTANCE = <span class="hljs-keyword">new</span> DefaultEventExecutorChooserFactory();
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">DefaultEventExecutorChooserFactory</span><span class="hljs-params">()</span> </span>{ }
&nbsp; &nbsp; <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> EventExecutorChooser <span class="hljs-title">newChooser</span><span class="hljs-params">(EventExecutor[] executors)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (isPowerOfTwo(executors.length)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> PowerOfTwoEventExecutorChooser(executors);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> GenericEventExecutorChooser(executors);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<h3 data-nodeid="3698">装饰者模式</h3>
<p data-nodeid="3699">装饰器模式是对被装饰类的功能增强，在不修改被装饰类的前提下，能够为被装饰类添加新的功能特性。当我们需要为一个类扩展功能时会使用装饰器模式，但是该模式的缺点是需要增加额外的代码。我们先通过一个简单的例子学习下装饰器模式应当如何使用，如下所示：</p>
<pre class="lang-java" data-nodeid="3700"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Shape</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">draw</span><span class="hljs-params">()</span></span>;
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Circle</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Shape</span> </span>{
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">draw</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.print(<span class="hljs-string">"draw a circle."</span>);
&nbsp; &nbsp; }
}
<span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ShapeDecorator</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Shape</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">protected</span> Shape shapeDecorated;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">ShapeDecorator</span><span class="hljs-params">(Shape shapeDecorated)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.shapeDecorated = shapeDecorated;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">draw</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; shapeDecorated.draw();
&nbsp; &nbsp; }
}
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">FillReadColorShapeDecorator</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ShapeDecorator</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">FillReadColorShapeDecorator</span><span class="hljs-params">(Shape shapeDecorated)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>(shapeDecorated);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">draw</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; shapeDecorated.draw();
&nbsp; &nbsp; &nbsp; &nbsp; fillColor();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">fillColor</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"Fill Read Color."</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3701">我们创建了一个 Shape 接口的抽象装饰类 ShapeDecorator，并维护 Shape 原始对象，FillReadColorShapeDecorator 是用于装饰 ShapeDecorator 的实体类，它不对 draw() 方法做任何修改，而是直接调用 Shape 对象原有的 draw() 方法，然后再调用 fillColor() 方法进行颜色填充。</p>
<p data-nodeid="3702">下面我们再来看一下 Netty 中 WrappedByteBuf 是如何装饰 ByteBuf 的，源码如下所示：</p>
<pre class="lang-java" data-nodeid="3703"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WrappedByteBuf</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ByteBuf</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">protected</span> <span class="hljs-keyword">final</span> ByteBuf buf;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">WrappedByteBuf</span><span class="hljs-params">(ByteBuf buf)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (buf == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> NullPointerException(<span class="hljs-string">"buf"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">this</span>.buf = buf;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">hasMemoryAddress</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> buf.hasMemoryAddress();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> <span class="hljs-title">memoryAddress</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> buf.memoryAddress();
&nbsp; &nbsp; }

&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="3704">WrappedByteBuf 是所有 ByteBuf 装饰器的基类，它并没有什么特别的，也是在构造函数里传入了原始的 ByteBuf 实例作为被装饰者。WrappedByteBuf 有两个子类 UnreleasableByteBuf 和 SimpleLeakAwareByteBuf，它们是真正实现对 ByteBuf 的功能增强，例如 UnreleasableByteBuf 类的 release() 方法是直接返回 false 表示不可被释放，源码实现如下所示。</p>
<pre class="lang-java" data-nodeid="3705"><code data-language="java"><span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UnreleasableByteBuf</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WrappedByteBuf</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> SwappedByteBuf swappedBuf;
&nbsp; &nbsp; UnreleasableByteBuf(ByteBuf buf) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>(buf <span class="hljs-keyword">instanceof</span> UnreleasableByteBuf ? buf.unwrap() : buf);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">release</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
}
</code></pre>
<p data-nodeid="3706">不知道你会不会有一个疑问，装饰器模式和代理模式都是实现目标类增强，他们有什么区别吗？装饰器模式和代理模式的实现确实是非常相似的，都需要维护原始的目标对象，装饰器模式侧重于为目标类增加新的功能，代理模式更侧重于在现有功能的基础上进行扩展。</p>
<h3 data-nodeid="3707">总结</h3>
<p data-nodeid="3708">学习设计模式切勿死记硬背，不仅要吸收设计模式的思想，还要理解为什么使用该设计模式。锻炼代码设计能力比较好的办法就是读优秀框架的源码，Netty 就是一个非常丰富的学习资源。我们需要了解源码中设计模式的使用场景，不断吸收消化，并能够做到在项目开发中学以致用。</p>
<p data-nodeid="3709">本节课所介绍的设计模式在 Netty 中并不能涵盖所有，还有很多等待我们去挖掘。留一个课后任务，学习以下设计模式在 Netty 中的运用。</p>
<ul data-nodeid="3710">
<li data-nodeid="3711">
<p data-nodeid="3712">模板方法模式：ServerBootStrap 和 Bootstrap 的 init 过程实现；</p>
</li>
<li data-nodeid="3713">
<p data-nodeid="3714">迭代器模式：CompositeByteBuf；</p>
</li>
<li data-nodeid="3715">
<p data-nodeid="3716">适配器模式：ScheduledFutureTask。</p>
</li>
</ul>
<p data-nodeid="3717" class="">如果你还有更多设计模式在 Netty 中的巧妙运用，欢迎留言分享。</p>

---

### 精选评论

##### **耀：
> 单例模式是最常见的设计模式，它可以保证全局只有一个实例，避免线程安全问题。避免线程安全是指？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 多线程并发安全问题

