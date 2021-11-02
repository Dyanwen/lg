<p data-nodeid="781" class="">14 讲我们介绍了基于 Kafka 和 KafkaTemplate 实现消息发送和消费，并重构了 SpringCSS 案例系统中的 account-service 和 customer-service 服务。今天，我们继续介绍 ActiveMQ，并基于 JmsTemplate 模板工具类为 SpringCSS 案例添加对应的消息通信机制。</p>
<h3 data-nodeid="782">JMS 规范与 ActiveMQ</h3>
<p data-nodeid="783">JMS（Java Messaging Service）是一种 Java 消息服务，它基于消息传递语义，提供了一整套经过抽象的公共 API。目前，业界也存在一批 JMS 规范的实现框架，其中具备代表性的是 ActiveMQ。</p>
<h4 data-nodeid="784">JMS 规范</h4>
<p data-nodeid="1353">JMS 规范提供了一批核心接口供开发人员使用，而这些接口构成了客户端的 API 体系，如下图所示：</p>
<p data-nodeid="1354" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/04/A0/CgpVE1_0G4iAFaHdAADIv5Y696k445.png" alt="图片2.png" data-nodeid="1359"></p>
<div data-nodeid="1355"><p style="text-align:center">JMS 规范中的核心 API</p></div>




<p data-nodeid="788">上图中可以看到，我们可以通过 ConnectionFactory 创建 Connection，作为客户端的 MessageProducer 和 MessageConsumer 通过 Connection 提供的会话（Session）与服务器进行交互，而交互的媒介就是各种经过封装、包含目标地址（Destination）的消息。</p>
<p data-nodeid="789"><strong data-nodeid="890">JMS 的消息由两大部分组成，即消息头（Header）和消息体（Payload）。</strong></p>
<p data-nodeid="790"><strong data-nodeid="899">消息体</strong>只包含具体的业务数据，而<strong data-nodeid="900">消息头</strong>包含了 JMS 规范定义的通用属性，比如消息的唯一标识 MessageId、目标地址 Destination、接收消息的时间 Timestamp、有效期 Expiration、优先级 Priority、持久化模式 DeliveryMode 等都是常见的通用属性，这些通用属性构成了消息通信的基础元数据（Meta Data），由消息通信系统默认设置。</p>
<p data-nodeid="791">JMS 规范中的点对点模型表现为队列（Queue），队列为消息通信提供了一对一顺序发送和消费的机制。点对点模型 API 在通用 API 基础上，专门区分生产者 QueueSender 和消费者 QueueReceiver。</p>
<p data-nodeid="792">而 Topic 是 JMS 规范中对发布-订阅模型的抽象，JMS 同样提供了专门的 TopicPublisher 和 TopicSubscriber。</p>
<p data-nodeid="793">对于 Topic 而言，因多个消费者存在同时消费一条消息的情况，所以消息有副本的概念。相较点对点模型，发布-订阅模型通常用于更新、事件、通知等非响应式请求场景。在这些场景中，消费者和生产者之间是透明的，消费者可以通过配置文件进行静态管理，也可以在运行过程中动态被创建，同时还支持取消订阅操作。</p>
<h4 data-nodeid="794">ActiveMQ</h4>
<p data-nodeid="795"><strong data-nodeid="908">JMS 规范存在 ActiveMQ、WMQ、TIBCO 等多种第三方实现方式，其中较主流的是 ActiveMQ。</strong></p>
<p data-nodeid="796">针对 ActiveMQ，目前有两个实现项目可供选择，一个是经典的 5.x 版本，另一个是下一代的 Artemis，关于这两者之间的关系，我们可以简单地认为 Artemis 是 ActiveMQ 的未来版本，代表 ActiveMQ 的发展趋势。因此，本课程我们将使用 Artemis 演示消息通信机制。</p>
<p data-nodeid="797">如果我们想启动 Artemis 服务，首先需要通过如下所示的命名创建一个服务实例。</p>
<pre class="lang-xml" data-nodeid="798"><code data-language="xml">artemis.cmd create D:\artemis --user springcss --password springcss_password
</code></pre>
<p data-nodeid="799">然后，执行如下命令，我们就可以正常启动这个 Artemis 服务实例了。</p>
<pre class="lang-xml" data-nodeid="800"><code data-language="xml">D:\artemis \bin\artemis run
</code></pre>
<p data-nodeid="801">Spring 提供了对 JMS 规范及各种实现的友好集成，通过直接配置 Queue 或 Topic，我们就可以使用 JmsTemplate 提供的各种方法简化对 Artemis 的操作了。</p>
<h3 data-nodeid="802">使用 JmsTemplate 集成 ActiveMQ</h3>
<p data-nodeid="803">如果我们想基于 Artemis 使用 JmsTemplate，首先需要在 Spring Boot 应用程序中添加对 spring-boot-starter-artemis 的依赖，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="804"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-artemis<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="805">在讨论如何使用 JmsTemplate 实现消息发送和消费之前，我们先来分析消息生产者和消费者的工作模式。</p>
<p data-nodeid="806">通常，生产者行为模式单一，而消费者根据消费方式的不同有一些特定的分类，比如常见的有推送型消费者（Push Consumer）和拉取型消费者（Pull Consumer）。</p>
<p data-nodeid="807"><strong data-nodeid="920">推送型方式指的是应用系统向消费者对象注册一个 Listener 接口并通过回调 Listener 接口方法实现消息消费，而在拉取方式下应用系统通常主动调用消费者的拉取消息方法消费消息，主动权由应用系统控制。</strong></p>
<p data-nodeid="808">在消息通信的两种基本模型中，发布-订阅模型支持生产者/消费者之间的一对多关系，属于一种典型的推送消费者实现机制；而点对点模型中有且仅有一个消费者，他们主要通过基于间隔性拉取的轮询（Polling）方式进行消息消费。</p>
<p data-nodeid="809">14 讲我们提到 Kafka 中消费消息的方式是一种典型的推送型消费者，所以 KafkaTemplate 只提供了发送消息的方法而没有提供实现消费消息的方法。而 JmsTemplate 则不同，它同时支持推送型消费和拉取型消费，接下来我们一起看下如何使用JmsTemplate 发送消息。</p>
<h4 data-nodeid="810">使用 JmsTemplate 发送消息</h4>
<p data-nodeid="811">JmsTemplate 中存在一批 send 方法用来实现消息发送，如下代码所示：</p>
<pre class="lang-java" data-nodeid="812"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(MessageCreator messageCreator)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Destination destination, <span class="hljs-keyword">final</span> MessageCreator messageCreator)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">send</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String destinationName, <span class="hljs-keyword">final</span> MessageCreator messageCreator)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
</code></pre>
<p data-nodeid="813">这些方法一方面指定了目标 Destination，另一方面提供了一个用于创建消息对象的 MessageCreator 接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="814"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">MessageCreator</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Message <span class="hljs-title">createMessage</span><span class="hljs-params">(Session session)</span> <span class="hljs-keyword">throws</span> JMSException</span>;
}
</code></pre>
<p data-nodeid="815">通过 send 方法发送消息的典型实现方式如下代码所示：</p>
<pre class="lang-java" data-nodeid="816"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendDemoObject</span><span class="hljs-params">(DemoObject demoObject)</span> </span>{ 
    jmsTemplate.send(<span class="hljs-string">"demo.queue"</span>, <span class="hljs-keyword">new</span> MessageCreator() { 
        <span class="hljs-meta">@Override</span> 
        <span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">createMessage</span><span class="hljs-params">(Session session)</span> 
        <span class="hljs-keyword">throws</span> JMSException </span>{ 
	        <span class="hljs-keyword">return</span> session.createObjectMessage(demoObject); 
	    } 
}
</code></pre>
<p data-nodeid="817">与 KakfaTemplate 不同，JmsTemplate 还提供了一组更为简便的方法实现消息发送，即 convertAndSend 方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="818"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">(Destination destination, <span class="hljs-keyword">final</span> Object message)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
</code></pre>
<p data-nodeid="819">通过 convertAndSend 方法，我们可以直接传入任意业务对象，且该方法能自动将业务对象转换为消息对象并进行消息发送，具体的示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="820"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendDemoObject</span><span class="hljs-params">(DemoObject demoObject)</span> </span>{ 
	jmsTemplate.convertAndSend(<span class="hljs-string">"demo.queue"</span>, demoObject); 
}
</code></pre>
<p data-nodeid="821">在以上代码中，我们注意到 convertAndSend 方法还存在一批重载方法，它包含了消息后处理功能，如下代码所示：</p>
<pre class="lang-java" data-nodeid="822"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">convertAndSend</span><span class="hljs-params">( Destination destination, <span class="hljs-keyword">final</span> Object message, <span class="hljs-keyword">final</span> MessagePostProcessor postProcessor)</span><span class="hljs-keyword">throws</span> JmsException </span>{
}
</code></pre>
<p data-nodeid="823">上述方法中的 MessagePostProcessor 就是一种消息后处理器，它用来在构建消息过程中添加自定义的消息属性，它的一种典型的使用方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="824"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">sendDemoObject</span><span class="hljs-params">(DemoObject demoObject)</span> </span>{ 
	jmsTemplate.convertAndSend(<span class="hljs-string">"demo.queue"</span>, demoObject, <span class="hljs-keyword">new</span>                 MessagePostProcessor() { 
        <span class="hljs-meta">@Override</span> 
        <span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">postProcessMessage</span><span class="hljs-params">(Message message)</span> <span class="hljs-keyword">throws</span>       JMSException </span>{ 
	        <span class="hljs-comment">//针对 Message 的处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> message;
	    } 
});
</code></pre>
<p data-nodeid="825">使用 JmsTemplate 的最后一步就是在配置文件中添加配置项，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="826"><code data-language="xml">spring:
&nbsp; artemis:
&nbsp;&nbsp;&nbsp; host: localhost
&nbsp;&nbsp;&nbsp; port: 61616
&nbsp;&nbsp;&nbsp; user: springcss
&nbsp;&nbsp;&nbsp; password: springcss_password
</code></pre>
<hr data-nodeid="827">
<p data-nodeid="828">这里我们指定了 artemis 服务器的地址、端口、用户名和密码等信息。同时，我们也可以在配置文件中指定 Destination 信息，具体配置方式如下代码所示：</p>
<pre class="lang-xml" data-nodeid="829"><code data-language="xml">spring:
&nbsp; jms:
&nbsp;&nbsp;&nbsp; template:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; default-destination: springcss.account.queue
</code></pre>
<h4 data-nodeid="830">使用 JmsTemplate 消费消息</h4>
<p data-nodeid="831">基于前面的讨论，我们知道 JmsTemplate 同时支持推送型消费和拉取型消费两种消费类型。我们先来看一下如何实现拉取型消费模式。</p>
<p data-nodeid="832">在 JmsTemplate 中提供了一批 receive 方法供我们从 artemis 中拉取消息，如下代码所示：</p>
<pre class="lang-java" data-nodeid="833"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">receive</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">receive</span><span class="hljs-params">(Destination destination)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> Message <span class="hljs-title">receive</span><span class="hljs-params">(String destinationName)</span> <span class="hljs-keyword">throws</span> JmsException </span>{
}
</code></pre>
<p data-nodeid="834">到这一步我们需要注意一点：调用上述方法时，当前线程会发生阻塞，直到一条新消息的到来。针对阻塞场景，这时 receive 方法的使用方式如下代码所示：</p>
<pre class="lang-java" data-nodeid="835"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DemoEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{
	Message message = jmsTemplate.receive(“demo.queue”); 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (DemoEvent) messageConverter.fromMessage(message);
}
</code></pre>
<p data-nodeid="836">这里我们使用了一个 messageConverter 对象将消息转化为领域对象。</p>
<p data-nodeid="837">在使用 JmsTemplate 时，我们可以使用 Spring 提供的 MappingJackson2MessageConverter、MarshallingMessageConverter、MessagingMessageConverter，以及 SimpleMessageConverter 实现消息转换，一般系统默认使用 SimpleMessageConverter。而在日常开发过程中，我们通常会使用 MappingJackson2MessageConverter 来完成 JSON 字符串与对象之间的转换。</p>
<p data-nodeid="838">同时，JmsTemplate 还提供了一组更为高阶的 receiveAndConvert 方法完成消息的接收和转换，如下代码所示：</p>
<pre class="lang-java" data-nodeid="839"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">receiveAndConvert</span><span class="hljs-params">(Destination destination)</span> <span class="hljs-keyword">throws</span> JmsException </span>{

}
</code></pre>
<p data-nodeid="840">顾名思义，receiveAndConvert 方法能在接收消息后完成对消息对象的自动转换，使得接收消息的代码更为简单，如下代码所示：</p>
<pre class="lang-java" data-nodeid="841"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> DemoEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{ 
	<span class="hljs-keyword">return</span> (DemoEvent)jmsTemplate.receiveAndConvert(<span class="hljs-string">"demo.queue"</span>); 
}
</code></pre>
<p data-nodeid="842">当然，在消费者端，我们同样需要指定与发送者端完全一致的 MessageConverter 和 Destination 来分别实现消息转换和设置消息目的地。</p>
<p data-nodeid="843">介绍完拉模式，接下来我们介绍推模式下的消息消费方法，实现方法也很简单，如下代码所示：</p>
<pre class="lang-java" data-nodeid="844"><code data-language="java"><span class="hljs-meta">@JmsListener(queues = “demo.queue”)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerEvent</span><span class="hljs-params">(DemoEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//TODO：添加消息处理逻辑</span>
}
</code></pre>
<p data-nodeid="845">在推模式下，开发人员只需要在 @JmsListener 注解中指定目标队列，就能自动接收来自该队列的消息。</p>
<h3 data-nodeid="846">在 SpringCSS 案例中集成 ActiveMQ</h3>
<p data-nodeid="847">ActiveMQ 是本专栏中使用到的第二款消息中间件，因为每款消息中间件都需要设置一些配置信息，所以我们有必要回到 SpringCSS 案例系统，先对配置信息的管理做一些优化。</p>
<h4 data-nodeid="848">实现 account-service 消息生产者</h4>
<p data-nodeid="849">首先，我们来回顾下《多维配置：如何使用 Spring Boot 中的配置体系？》的内容介绍，在 Spring Boot 中，我们可以通过 Profile 有效管理针对不同场景和环境的配置信息。</p>
<p data-nodeid="850">而在 SpringCSS 案例中，Kafka、ActiveMQ 及 16 讲将要介绍的 RabbitMQ 都是消息中间件，在案例系统运行过程中，我们需要选择其中一种中间件演示消息发送和接收到过程，这样我们就需要针对不同的中间件设置不同的 Profile 了。</p>
<p data-nodeid="851">在 account-service 中，我们可以根据 Profile 构建如下所示的配置文件体系。</p>
<p data-nodeid="852"><img src="https://s0.lgstatic.com/i/image2/M01/03/CB/Cip5yF_ivRuAdYV0AAANYpYvrk4834.png" alt="Drawing 1.png" data-nodeid="952"></p>
<div data-nodeid="853"><p style="text-align:center">account-service 中的配置文件</p></div>
<p data-nodeid="854">从以上图中可以看到：根据三种不同的中间件，我们分别提供了三个配置文件。以其中的 application-activemq.yml 为例，其包含的配置项如下代码所示：</p>
<pre class="lang-java" data-nodeid="855"><code data-language="java">spring:
&nbsp; jms:
&nbsp;&nbsp;&nbsp; template:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">default</span>-destination: springcss.account.queue
&nbsp; artemis:
&nbsp;&nbsp;&nbsp; host: localhost
&nbsp;&nbsp;&nbsp; port: <span class="hljs-number">61616</span>
&nbsp;&nbsp;&nbsp; user: springcss
&nbsp;&nbsp;&nbsp; password: springcss_password
&nbsp;&nbsp;&nbsp; embedded:
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; enabled: <span class="hljs-keyword">false</span>
</code></pre>
<p data-nodeid="856">在主配置文件 application.yml 中，我们可以将当前可用的 Profile 设置为 activemq，如下代码所示：</p>
<pre class="lang-java" data-nodeid="857"><code data-language="java">spring:
&nbsp; profiles:
&nbsp;&nbsp;&nbsp; active: activemq
</code></pre>
<p data-nodeid="858">介绍完配置信息的优化管理方案，我们再来看看实现消息发送的 ActiveMQAccountChangedPublisher 类，如下代码所示：</p>
<pre class="lang-java" data-nodeid="859"><code data-language="java"><span class="hljs-meta">@Component("activeMQAccountChangedPublisher")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActiveMQAccountChangedPublisher</span></span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> JmsTemplate jmsTemplate;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jmsTemplate.convertAndSend(AccountChannels.SPRINGCSS_ACCOUNT_QUEUE, event, <span class="hljs-keyword">this</span>::addEventSource);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Message <span class="hljs-title">addEventSource</span><span class="hljs-params">(Message message)</span> <span class="hljs-keyword">throws</span> JMSException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; message.setStringProperty(<span class="hljs-string">"EVENT_SYSTEM"</span>, <span class="hljs-string">"SpringCSS"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> message;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="860">以上代码中，我们基于 JmsTemplate 的 convertAndSend 方法完成了消息的发送。同时，我们注意到：这里也使用了另一种实现 MessagePostProcessor 的方法，即 lambda 语法，你可以参考这种语法简化代码的组织方式。</p>
<p data-nodeid="861">另一方面，在案例中，我们希望使用 MappingJackson2MessageConverter 完成对消息的转换。因此，我们可以在 account-service 中添加一个 ActiveMQMessagingConfig 初始化具体的 MappingJackson2MessageConverter 对象，如下代码所示：</p>
<pre class="lang-java" data-nodeid="862"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActiveMQMessagingConfig</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> MappingJackson2MessageConverter <span class="hljs-title">activeMQMessageConverter</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MappingJackson2MessageConverter messageConverter = <span class="hljs-keyword">new</span> MappingJackson2MessageConverter();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; messageConverter.setTypeIdPropertyName(<span class="hljs-string">"_typeId"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Class&lt;?&gt;&gt; typeIdMappings = <span class="hljs-keyword">new</span> HashMap&lt;String, Class&lt;?&gt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; typeIdMappings.put(<span class="hljs-string">"accountChangedEvent"</span>, AccountChangedEvent.class);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; messageConverter.setTypeIdMappings(typeIdMappings);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> messageConverter;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="863">上述代码的核心作用是定义了 typeId 到 Class 的 Map，这样做的目的在于为消息的转换提供了灵活性。比如我们可以在 account-service 中发送了一个 Id 为“accountChangedEvent”且类型为 AccountChangedEvent 的业务对象，而在消费该消息的场景中，我们只需要指定同一个 Id，对应的消息就可以自动转化为另一种业务对象（不一定是这里使用到的 AccountChangedEvent）。</p>
<h4 data-nodeid="864">实现 customer-service 消息消费者</h4>
<p data-nodeid="865">我们先回到 customer-service 服务，看看如何消费来自 account-service 的消息，如下代码所示：</p>
<pre class="lang-java" data-nodeid="866"><code data-language="java"><span class="hljs-meta">@Component("activeMQAccountChangedReceiver")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActiveMQAccountChangedReceiver</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> JmsTemplate jmsTemplate;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> AccountChangedEvent <span class="hljs-title">receiveEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (AccountChangedEvent) jmsTemplate.receiveAndConvert(AccountChannels.SPRINGCSS_ACCOUNT_QUEUE);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="867">这里，我们只是简单通过 JmsTemplate 的 receiveAndConvert 方法拉取来自 ActiveMQ 的消息。</p>
<p data-nodeid="868"><strong data-nodeid="965">请注意，因为 receiveAndConvert 方法的执行过程是阻塞性的拉取行为，所以我们可以实现一个新的 Controller 专门测试该方法的有效性，如下代码所示：</strong></p>
<pre class="lang-java" data-nodeid="869"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value="messagereceive")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MessageReceiveController</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> ActiveMQAccountChangedReceiver accountChangedReceiver; 

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@RequestMapping(value = "", method = RequestMethod.GET)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">receiveAccountChangedEvent</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; accountChangedReceiver.receiveAccountChangedEvent();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="870">一旦我们访问了这个端点，系统就会拉取 ActiveMQ 中目前尚未消费的消息。如果 ActiveMQ 没有待消费的消息，这个方法就会阻塞，且一直处于等待状态，直到新消息的到来。</p>
<p data-nodeid="871">如果你想使用消息推送的方式来消费消息，实现过程更加简单，如下代码所示：</p>
<pre class="lang-java" data-nodeid="872"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-meta">@JmsListener(destination = AccountChannels.SPRINGCSS_ACCOUNT_QUEUE)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handlerAccountChangedEvent</span><span class="hljs-params">(AccountChangedEvent event)</span> </span>{ 
&nbsp;&nbsp;&nbsp; AccountMessage account = event.getAccountMessage();
&nbsp;&nbsp;&nbsp; String operation = event.getOperation(); 
&nbsp;
	System.out.print(accountMessage.getId() + <span class="hljs-string">":"</span> + accountMessage.getAccountCode() + <span class="hljs-string">":"</span> + accountMessage.getAccountName());
}
</code></pre>
<p data-nodeid="873">从以上代码中可以看到，我们可以直接通过 @JmsListener 注解消费从 ActiveMQ 推送过来的消息。这里我们只是把消息打印了出来，你可以根据实际需要对消息进行任何形式的处理。</p>
<h3 data-nodeid="874">小结与预告</h3>
<p data-nodeid="875">本节课我们继续介绍基于 JMS 规范的 ActiveMQ 消息中间件，并使用 Spring Boot 提供的 JmsTemplate 完成了消息的发送和消费。同样，我们也将 ActiveMQ 的使用过程集成到 SpringCSS 案例中，并基于 Spring Boot 的配置体系对配置信息的管理过程做了优化。16 讲我们将继续介绍本课程中最后一款要引入的消息中间件 RabbitMQ 及 Spring Boot 中提供的模板工具类 RabbitTemplate。</p>
<p data-nodeid="876">这里给你留一道思考题：使用 JmsTemplate 时，如何分别实现基于推模式和拉模式的消息消费？</p>
<p data-nodeid="877" class="">学完本节课后，你有什么问题欢迎在留言区与我互动、交流。另外，如果你觉得本专栏有价值，欢迎转发给有需要的朋友哦~</p>

---

### 精选评论

##### **飞：
> java.lang.NullPointerException: null	at com.springcss.customer.message.AbstractAccountChangedReceiver.handleEvent(AbstractAccountChangedReceiver.java:24) ~[classes/:na]	at com.springcss.customer.message.AbstractAccountChangedReceiver.receiveAccountChangedEvent(AbstractAccountChangedReceiver.java:20) ~[classes/:na]	at com.springcss.customer.controller.MessageReceiveController.receiveAccountChangedEvent(MessageReceiveController.java:22) ~[classes/:na]	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:na]	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62) ~[na:na]	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:na]	at java.base/java.lang.reflect.Method.invoke(Method.java:566) ~[na:na]老师，请问Account项目添加账户成功，Customer项目控制台也打印了消息内容，但调用http://localhost:8083/messagereceive为什么报错呢？谢谢！Whitelabel Error PageThis application has no explicit mapping for /error, so you are seeing this as a fallback.Thu Jan 07 09:32:13 CST 2021There was an unexpected error (type=Internal Server Error, status=500).No message available

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 看到是个空指针，估计是某个地方没赋值，可以再验证一下数据赋值上是否完整。

