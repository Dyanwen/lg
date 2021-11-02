<p data-nodeid="43693" class="">15 讲我们介绍了基于 ActiveMQ 和 JmsTemplate 实现消息发送和消费，并重构了 SpringCSS 案例系统中的 account-service 和 customer-service 服务。</p>
<p data-nodeid="43694">今天，我们将介绍另一款主流的消息中间件 RabbitMQ，并基于 RabbitTemplate 模板工具类为 SpringCSS 案例添加对应的消息通信机制。</p>
<h3 data-nodeid="43695">AMQP 规范与 RabbitMQ</h3>
<p data-nodeid="43696">AMQP（Advanced Message Queuing Protocol）是一个提供统一消息服务的应用层标准高级消息队列规范。和 JMS 规范一样，AMQP 描述了一套模块化的组件及组件之间进行连接的标准规则，用于明确客户端与服务器交互的语义。而业界也存在一批实现 AMQP 规范的框架，其中极具代表性的是 RabbitMQ。</p>
<h4 data-nodeid="43697">AMQP 规范</h4>
<p data-nodeid="43698">在 AMQP 规范中存在三个核心组件，分别是交换器（Exchange）、消息队列（Queue）和绑定（Binding）。其中交换器用于接收应用程序发送的消息，并根据一定的规则将这些消息路由发送到消息队列中；消息队列用于存储消息，直到这些消息被消费者安全处理完毕；而绑定定义了交换器和消息队列之间的关联，为它们提供了路由规则。</p>
<p data-nodeid="43699">在 AMQP 规范中并没有明确指明类似 JMS 中一对一的点对点模型和一对多的发布-订阅模型，不过通过控制 Exchange 与 Queue 之间的路由规则，我们可以很容易地模拟 Topic 这种典型消息中间件的概念。</p>
<p data-nodeid="43700">如果存在多个 Queue，Exchange 如何知道把消息发送到哪个 Queue 中呢？</p>
<p data-nodeid="43701">通过 Binding 规则设置路由信息即可。在与多个 Queue 关联之后，Exchange 中会存在一个路由表，这个表中维护着每个 Queue 存储消息的限制条件。</p>
<p data-nodeid="43702">消息中包含一个路由键（Routing Key），它由消息发送者产生，并提供给 Exchange 路由这条消息的标准。而 Exchange 会检查 Routing Key，并结合路由算法决定将消息路由发送到哪个 Queue 中。</p>
<p data-nodeid="43703">通过下面 Exchange 与 Queue 之间的路由关系图，我们可以看到一条来自生产者的消息通过 Exchange 中的路由算法可以发送给一个或多个 Queue，从而实现点对点和发布订阅功能。</p>
<p data-nodeid="43860" class=""><img src="https://s0.lgstatic.com/i/image/M00/8D/16/CgqCHl_4DTKAbAiPAADqglSjwTc554.png" alt="图片7.png" data-nodeid="43864"></p>
<div data-nodeid="43861"><p style="text-align:center">AMQP 路由关系图</p></div>


<p data-nodeid="43706">上图中，不同的路由算法存在不同的 Exchange 类型，而 AMQP 规范中指定了直接式交换器（Direct Exchange）、广播式交换器（Fanout Exchange）、主题式交换器（Topic Exchange）和消息头式交换器（Header Exchange）这几种 Exchange 类型，不过这一讲我们将重点介绍直接式交换器。</p>
<p data-nodeid="44540">通过精确匹配消息的 Routing Key，直接式交换器可以将消息路由发送到零个或多个队列中，如下图所示：</p>
<p data-nodeid="44541"><img src="https://s0.lgstatic.com/i/image/M00/8D/0B/Ciqc1F_4DTqAJ5kHAADKl5uWm0U797.png" alt="图片8.png" data-nodeid="44545"></p>

<div data-nodeid="44200"><p style="text-align:center">Direct Exchange 示意图</p></div>


<h4 data-nodeid="43709">RabbitMQ 基本架构</h4>
<p data-nodeid="43710">RabbitMQ 使用 Erlang 语言开发的 AMQP 规范标准实现框架，而 ConnectionFactory、Connection、Channel 是 RabbitMQ 对外提供的 API 中最基本的对象，都需要遵循 AMQP 规范的建议。其中，Channel 是应用程序与 RabbitMQ 交互过程中最重要的一个接口，因为我们大部分的业务操作需要通过 Channel 接口完成，如定义 Queue、定义 Exchange、绑定 Queue 与 Exchange、发布消息等。</p>
<p data-nodeid="43711">如果想启动 RabbitMQ，我们只需要运行 rabbitmq-server.sh 文件即可。不过，因为 RabbitMQ 依赖于 Erlang，所以首先我们需要确保安装上 Erlang 环境。</p>
<p data-nodeid="43712">接下来，我们一起看下如何使用 Spring 框架所提供的 RabbitTemplate 模板工具类集成 RabbitMQ。</p>
<h3 data-nodeid="43713">使用 RabbitTemplate 集成 RabbitMQ</h3>
<p data-nodeid="43714">如果想使用 RabbitTemplate 集成 RabbitMQ，首先我们需要在 Spring Boot 应用程序中添加对 spring-boot-starter-amqp 的依赖，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="43715"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-amqp<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<h4 data-nodeid="43716">使用 RabbitTemplate 发送消息</h4>
<p data-nodeid="43717">和其他模板类一样，RabbitTemplate 也提供了一批 send 方法用来发送消息，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43718"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(Message message)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;send(<span class="hljs-keyword">this</span>.exchange, <span class="hljs-keyword">this</span>.routingKey, message);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(String routingKey, Message message)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;send(<span class="hljs-keyword">this</span>.exchange, routingKey, message);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String exchange, <span class="hljs-keyword">final</span> String routingKey, <span class="hljs-keyword">final</span> Message message)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;send(exchange, routingKey, message, <span class="hljs-keyword">null</span>);
}
</code></pre>
<p data-nodeid="43719">在这里可以看到，我们指定了消息发送的 Exchange 及用于消息路由的路由键 RoutingKey。因为这些 send 方法发送的是原生消息对象，所以在与业务代码进行集成时，我们需要将业务对象转换为 Message 对象，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="43720"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendDemoObject</span><span class="hljs-params">(DemoObject demoObject)</span> </span>{ 
    MessageConverter converter =               rabbitTemplate.getMessageConverter(); 
    MessageProperties props = <span class="hljs-keyword">new</span> MessageProperties(); 
    Message message = converter.toMessage(demoObject, props); 
    rabbitTemplate.send(<span class="hljs-string">"demo.queue"</span>, message); 
}
</code></pre>
<p data-nodeid="43721">如果我们不想在业务代码中嵌入 Message 等原生消息对象，还可以使用 RabbitTemplate 的 convertAndSend 方法组进行实现，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43722"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(Object object)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; convertAndSend(<span class="hljs-keyword">this</span>.exchange, <span class="hljs-keyword">this</span>.routingKey, object, (CorrelationData) <span class="hljs-keyword">null</span>);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">correlationConvertAndSend</span><span class="hljs-params">(Object object, CorrelationData correlationData)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; convertAndSend(<span class="hljs-keyword">this</span>.exchange, <span class="hljs-keyword">this</span>.routingKey, object, correlationData);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(String routingKey, <span class="hljs-keyword">final</span> Object object)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; convertAndSend(<span class="hljs-keyword">this</span>.exchange, routingKey, object, (CorrelationData) <span class="hljs-keyword">null</span>);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(String routingKey, <span class="hljs-keyword">final</span> Object object, CorrelationData correlationData)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; convertAndSend(<span class="hljs-keyword">this</span>.exchange, routingKey, object, correlationData);
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(String exchange, String routingKey, <span class="hljs-keyword">final</span> Object object)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; convertAndSend(exchange, routingKey, object, (CorrelationData) <span class="hljs-keyword">null</span>);
}
</code></pre>
<p data-nodeid="43723">上述 convertAndSend 方法组在内部就完成了业务对象向原生消息对象的自动转换过程，因此，我们可以使用如下所示的代码来简化消息发送过程。</p>
<pre class="lang-java" data-nodeid="43724"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendDemoObject</span><span class="hljs-params">(DemoObject demoObject)</span> </span>{ 
	rabbitTemplate.convertAndSend(<span class="hljs-string">"demo.queue"</span>, demoObject); 
}
</code></pre>
<p data-nodeid="43725">当然，有时候我们需要在消息发送的过程中为消息添加一些属性，这就不可避免需要操作原生 Message 对象，而 RabbitTemplate 也提供了一组 convertAndSend 重载方法应对这种场景，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43726"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(String exchange, String routingKey, <span class="hljs-keyword">final</span> Object message,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">final</span> MessagePostProcessor messagePostProcessor, CorrelationData correlationData)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Message messageToSend = convertMessageIfNecessary(message);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; messageToSend = messagePostProcessor.postProcessMessage(messageToSend, correlationData);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; send(exchange, routingKey, messageToSend, correlationData);
}
</code></pre>
<p data-nodeid="43727">注意这里，我们使用了一个 MessagePostProcessor 类对所生成的消息进行后处理，MessagePostProcessor 的使用方式如下代码所示：</p>
<pre class="lang-java" data-nodeid="43728"><code data-language="java">rabbitTemplate.convertAndSend(“demo.queue”, event, <span class="hljs-keyword">new</span> MessagePostProcessor() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">postProcessMessage</span><span class="hljs-params">(Message message)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//针对 Message 的处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> message;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
});
</code></pre>
<p data-nodeid="43729">使用 RabbitTemplate 的最后一步是在配置文件中添加配置项，在配置时我们需要指定 RabbitMQ 服务器的地址、端口、用户名和密码等信息，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="43730"><code data-language="xml">spring:
&nbsp; rabbitmq:
&nbsp;&nbsp;&nbsp; host: 127.0.0.1
&nbsp;&nbsp;&nbsp; port: 5672
&nbsp;&nbsp;&nbsp; username: guest
&nbsp;&nbsp;&nbsp; password: guest
&nbsp;&nbsp;&nbsp; virtual-host: DemoHost
</code></pre>
<p data-nodeid="43731"><strong data-nodeid="43819">注意，出于对多租户和安全因素的考虑，AMQP 还提出了虚拟主机（Virtual Host）概念，因此这里出现了一个 virtual-host 配置项。</strong></p>
<p data-nodeid="45221">Virtual Host 类似于权限控制组，内部可以包含若干个 Exchange 和 Queue。多个不同用户使用同一个 RabbitMQ 服务器提供的服务时，我们可以将其划分为多个 Virtual Host，并在自己的 Virtual Host 中创建相应组件，如下图所示：</p>
<p data-nodeid="45222"><img src="https://s0.lgstatic.com/i/image/M00/8D/16/CgqCHl_4DUiAGJuEAAC1QELaeNI351.png" alt="图片9.png" data-nodeid="45226"></p>

<div data-nodeid="44881"><p style="text-align:center">添加了 Virtual Host 的 AMQP 模型</p></div>


<h4 data-nodeid="43734">使用 RabbitTemplate 消费消息</h4>
<p data-nodeid="43735">和 JmsTemplate 一样，使用 RabbitTemplate 消费消息时，我们也可以使用推模式和拉模式。</p>
<p data-nodeid="43736">在拉模式下，使用 RabbitTemplate 的典型示例如下代码所示：</p>
<pre class="lang-java" data-nodeid="43737"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DemoEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (DemoEvent) rabbitTemplate.receiveAndConvert(“demo.queue”);
}
</code></pre>
<p data-nodeid="43738">这里，我们使用了 RabbitTemplate 中的 receiveAndConvert 方法，该方法可以从一个指定的 Queue 中拉取消息，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43739"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">receiveAndConvert</span><span class="hljs-params">(String queueName)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> receiveAndConvert(queueName, <span class="hljs-keyword">this</span>.receiveTimeout);
}
</code></pre>
<p data-nodeid="43740">这里请注意，内部的 receiveAndConvert 方法中出现了第二个参数 receiveTimeout，这个参数的默认值是 0，意味着即使调用 receiveAndConvert 时队列中没有消息，该方法也会立即返回一个空对象，而不会等待下一个消息的到来，这点与 15 讲介绍的 JmsTemplate 存在本质性的区别。</p>
<p data-nodeid="43741">如果我们想实现与 JmsTemplate 一样的阻塞等待，设置好 receiveTimeout 参数即可，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43742"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DemoEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{ 
	<span class="hljs-keyword">return</span> (DemoEvent)rabbitTemplate.receiveAndConvert(<span class="hljs-string">"demo.queue"</span>, <span class="hljs-number">2000</span>ms); 
}
</code></pre>
<p data-nodeid="43743">如果不想每次方法调用都指定 receiveTimeout，我们可以在配置文件中通过添加配置项的方式设置 RabbitTemplate 级别的时间，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="43744"><code data-language="xml">spring:
&nbsp; rabbitmq:
&nbsp;&nbsp;&nbsp; template:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; receive-timeout: 2000
</code></pre>
<p data-nodeid="43745">当然，RabbitTemplate 也提供了一组支持接收原生消息的 receive 方法，但我们还是建议使用 receiveAndConvert 方法实现拉模式下的消息消费。</p>
<p data-nodeid="43746">介绍完拉模式，接下来我们介绍推模式，它的实现方法也很简单，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43747"><code data-language="java"><span class="hljs-meta">@RabbitListener(queues = “demo.queue”)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerEvent</span><span class="hljs-params">(DemoEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//TODO：添加消息处理逻辑</span>
}
</code></pre>
<p data-nodeid="43748">开发人员在 @RabbitListener 中指定目标队列即可自动接收来自该队列的消息，这种实现方式与 15 讲中介绍的 @JmsListener 完全一致。</p>
<h3 data-nodeid="43749">在 SpringCSS 案例中集成 RabbitMQ</h3>
<p data-nodeid="43750">因为这三种模板工具类的使用方式非常类似，都可以用来提取公共代码形成统一的接口和抽象类，所以作为介绍消息中间件的最后一讲，我们想对 SpringCSS 案例中的三种模板工具类的集成方式进行抽象。</p>
<h4 data-nodeid="43751">实现 account-service 消息生产者</h4>
<p data-nodeid="43752">在消息生产者的 account-service 中，我们提取了如下所示的 AccountChangedPublisher 作为消息发布的统一接口。</p>
<pre class="lang-java" data-nodeid="43753"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountChangedPublisher</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">publishAccountChangedEvent</span><span class="hljs-params">(Account account, String operation)</span></span>;
}
</code></pre>
<p data-nodeid="43754">请注意，这是一个面向业务的接口，没有使用用于消息通信的 AccountChangedEvent 对象。</p>
<p data-nodeid="43755">而我们将在 AccountChangedPublisher 接口的实现类 AbstractAccountChangedPublisher 中完成对 AccountChangedEvent 对象的构建，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43756"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AbstractAccountChangedPublisher</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AccountChangedPublisher</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishAccountChangedEvent</span><span class="hljs-params">(Account account, String operation)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountMessage accountMessage = <span class="hljs-keyword">new</span> AccountMessage(account.getId(), account.getAccountCode(), account.getAccountName());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountChangedEvent event = <span class="hljs-keyword">new</span> AccountChangedEvent(AccountChangedEvent.class.getTypeName(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operation.toString(), accountMessage);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; publishEvent(event);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishEvent</span><span class="hljs-params">(AccountChangedEvent event)</span></span>;
}
</code></pre>
<p data-nodeid="43757">AbstractAccountChangedPublisher 是一个抽象类，我们基于传入的业务对象构建了一个消息对象 AccountChangedEvent，并通过 publishEvent 抽象方法发送消息。</p>
<p data-nodeid="43758">针对不同的消息中间件，我们需要分别实现对应的 publishEvent 方法。以 Kafka 为例，我们重构了原有代码并提供了如下所示的 KafkaAccountChangedPublisher 实现类。</p>
<pre class="lang-java" data-nodeid="43759"><code data-language="java"><span class="hljs-meta">@Component("kafkaAccountChangedPublisher")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KafkaAccountChangedPublisher</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractAccountChangedPublisher</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> KafkaTemplate&lt;String, AccountChangedEvent&gt; kafkaTemplate;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kafkaTemplate.send(AccountChannels.SPRINGCSS_ACCOUNT_TOPIC, event);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="43760">对 RabbitMQ 而言，RabbitMQAccountChangedPublisher 的实现方式也是类似，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43761"><code data-language="java"><span class="hljs-meta">@Component("rabbitMQAccountChangedPublisher")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RabbitMQAccountChangedPublisher</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractAccountChangedPublisher</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> RabbitTemplate rabbitTemplate;

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rabbitTemplate.convertAndSend(AccountChannels.SPRINGCSS_ACCOUNT_QUEUE, event, <span class="hljs-keyword">new</span> MessagePostProcessor() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">postProcessMessage</span><span class="hljs-params">(Message message)</span> <span class="hljs-keyword">throws</span> AmqpException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MessageProperties props = message.getMessageProperties();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; props.setHeader(<span class="hljs-string">"EVENT_SYSTEM"</span>, <span class="hljs-string">"SpringCSS"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> message;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="43762">对于 RabbitMQ 而言，在使用 RabbitMQAccountChangedPublisher 发送消息之前，我们需要先初始化 Exchange、Queue，以及两者之间的 Binding 关系，因此我们实现了如下所示的 RabbitMQMessagingConfig 配置类。</p>
<pre class="lang-java" data-nodeid="43763"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RabbitMQMessagingConfig</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String SPRINGCSS_ACCOUNT_DIRECT_EXCHANGE = <span class="hljs-string">"springcss.account.exchange"</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String SPRINGCSS_ACCOUNT_ROUTING = <span class="hljs-string">"springcss.account.routing"</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Queue <span class="hljs-title">SpringCssDirectQueue</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Queue(AccountChannels.SPRINGCSS_ACCOUNT_QUEUE, <span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> DirectExchange <span class="hljs-title">SpringCssDirectExchange</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DirectExchange(SPRINGCSS_ACCOUNT_DIRECT_EXCHANGE, <span class="hljs-keyword">true</span>, <span class="hljs-keyword">false</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Binding <span class="hljs-title">bindingDirect</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> BindingBuilder.bind(SpringCssDirectQueue()).to(SpringCssDirectExchange())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .with(SPRINGCSS_ACCOUNT_ROUTING);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Jackson2JsonMessageConverter <span class="hljs-title">rabbitMQMessageConverter</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Jackson2JsonMessageConverter();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="43764">上述代码中初始化了一个 DirectExchange、一个 Queue ，并设置了两者之间的绑定关系，同时我们还初始化了一个 Jackson2JsonMessageConverter 用于在消息发送过程中将消息转化为序列化对象，以便在网络上进行传输。</p>
<h4 data-nodeid="43765">实现 customer-service 消息消费者</h4>
<p data-nodeid="43766">现在，回到 customer-service 服务，我们先看看提取用于接收消息的统一化接口 AccountChangedReceiver，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43767"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountChangedReceiver</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//Pull 模式下的消息接收方法</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">receiveAccountChangedEvent</span><span class="hljs-params">()</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//Push 模式下的消息接收方法</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">handlerAccountChangedEvent</span><span class="hljs-params">(AccountChangedEvent event)</span></span>;
}
</code></pre>
<p data-nodeid="43768">AccountChangedReceiver 分别定义了拉取模式和推送模式下的消息接收方法，同样我们也提取了一个抽象实现类 AbstractAccountChangedReceiver，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43769"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AbstractAccountChangedReceiver</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AccountChangedReceiver</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; LocalAccountRepository localAccountRepository;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">receiveAccountChangedEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountChangedEvent event = receiveEvent();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; handleEvent(event);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; AccountMessage account = event.getAccountMessage();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String operation = event.getOperation();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; operateAccount(account, operation);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">operateAccount</span><span class="hljs-params">(AccountMessage accountMessage, String operation)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.print(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountMessage.getId() + <span class="hljs-string">":"</span> + accountMessage.getAccountCode() + <span class="hljs-string">":"</span> + accountMessage.getAccountName());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LocalAccount localAccount = <span class="hljs-keyword">new</span> LocalAccount(accountMessage.getId(), accountMessage.getAccountCode(),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountMessage.getAccountName());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (operation.equals(<span class="hljs-string">"ADD"</span>) || operation.equals(<span class="hljs-string">"UPDATE"</span>)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; localAccountRepository.save(localAccount);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; localAccountRepository.delete(localAccount);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> AccountChangedEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="43770">这里实现了 AccountChangedReceiver 接口的 receiveAccountChangedEvent 方法，并定义了一个 receiveEvent 抽象方法接收来自不同消息中间件的 AccountChangedEvent 消息。一旦 receiveAccountChangedEvent 方法获取了消息，我们将根据其中的 Account 对象及对应的操作更新本地数据库。</p>
<p data-nodeid="43771">接下来我们看看 AbstractAccountChangedReceiver 中的一个实现类 RabbitMQAccountChangedReceiver，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43772"><code data-language="java"><span class="hljs-meta">@Component("rabbitMQAccountChangedReceiver")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RabbitMQAccountChangedReceiver</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractAccountChangedReceiver</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> RabbitTemplate rabbitTemplate;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> AccountChangedEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (AccountChangedEvent) rabbitTemplate.receiveAndConvert(AccountChannels.SPRINGCSS_ACCOUNT_QUEUE);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@RabbitListener(queues = AccountChannels.SPRINGCSS_ACCOUNT_QUEUE)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerAccountChangedEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.handleEvent(event);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="43773">上述 RabbitMQAccountChangedReceiver 同时实现了 AbstractAccountChangedReceiver 的 receiveEvent 抽象方法及 AccountChangedReceiver 接口中的 handlerAccountChangedEvent 方法。其中 receiveEvent 方法用于主动拉取消息，而 handlerAccountChangedEvent 方法用于接受推动过来的消息，在该方法上我们添加了 @RabbitListener 注解。</p>
<p data-nodeid="43774">接着我们来看下同样继承了 AbstractAccountChangedReceiver 抽象类的 KafkaAccountChangedListener 类，如下代码所示：</p>
<pre class="lang-java" data-nodeid="43775"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">KafkaAccountChangedListener</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractAccountChangedReceiver</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@KafkaListener(topics = AccountChannels.SPRINGCSS_ACCOUNT_TOPIC)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerAccountChangedEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">super</span>.handleEvent(event);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> AccountChangedEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="43776">我们知道 Kafka 只能通过推送方式获取消息，所以它只实现了 handlerAccountChangedEvent 方法，而 receiveEvent 方法为空。</p>
<h3 data-nodeid="43777">小结与预告</h3>
<p data-nodeid="43778">这一讲，我们学习了最后一款消息中间件 RabbitMQ，并使用 Spring Boot 提供的 RabbitTemplate 完成了消息的发送和消费。同时，基于三种消息中间件的对接方式，我们提取了它们之间的共同点，并抽取了对应的接口和抽象类，重构了 SpringCSS 系统的实现过程。</p>
<p data-nodeid="43779">Spring 为我们提供了一整套完整的安全解决方案，17 讲我们将对这套解决方案展开讨论。</p>
<p data-nodeid="43780">这里给你留一道思考题：在使用 RabbitMQ 时，如何完成 Exchange、Queue ，以及它们之间绑定关系的初始化？欢迎你在留言区与我交流、互动。</p>
<p data-nodeid="43781" class="">另外，如果你觉得本专栏很有价值，欢迎转发给更多好友哦~</p>

---

### 精选评论


