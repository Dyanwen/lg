<p data-nodeid="3641">从上一课时的内容中，我们对 Spring Cloud Stream 的基本架构有了全面的了解。今天，就让我们回到案例，来看看如何使用 Spring Cloud Stream 来完成消息发布者和消费者的构建。</p>
<h3 data-nodeid="3642">设计 SpringHealth 中的消息发布场景</h3>
<p data-nodeid="4824">在《消息驱动：如何理解 Spring 中对消息处理机制的抽象过程？》课时中，我们已经给出了在 SpringHealth 案例系统中应用消息处理机制的一个典型场景。类似 SpringHealth 这样的系统中的用户信息变动并不会太频繁，所以很多时候我们会想到通过缓存系统来存放用户信息。而一旦用户信息发生变化，user-service 可以发送一个事件，给到相关的订阅者并更新缓存信息，如下图所示：</p>
<p data-nodeid="4825" class=""><img src="https://s0.lgstatic.com/i/image/M00/76/EF/CgqCHl_IvYmAAdzPAABKd1VzQCI715.png" alt="Drawing 0.png" data-nodeid="4830"></p>
<div data-nodeid="4826"><p style="text-align:center">用户信息更新场景中的事件驱动架构</p></div>





<p data-nodeid="5349">一般而言，事件在命名上通常采用过去时态以表示该事件所代表的动作已经发生。所以，我们把这里的用户信息变更事件命名为 UserInfoChangedEvent。通常，我们也会建议使用一个独立的事件消费者来订阅这个事件，就像上图中的 consumer-service1 一样。但为了保持 SpringHealth 系统的简单性，我们不想再单独构建一个微服务，而是选择把事件订阅和消费的相关功能同样放在了 intervention-service 中，如下图所示：</p>
<p data-nodeid="5619"><img src="https://s0.lgstatic.com/i/image/M00/76/E4/Ciqc1F_IvgSAQ-k8AABA--dRdWc848.png" alt="Drawing 1.png" data-nodeid="5623"></p>
<div data-nodeid="5620" class=""><p style="text-align:center">简化之后的用户信息更新场景处理流程</p></div>





<p data-nodeid="6394">接下来我们关注于上图中的事件发布者 user-service。在 user-service 中需要设计并实现使用 Spring Cloud Stream 发布消息的各个组件，包括 Source、Channel 和 Binder。我们围绕 UserInfoChangedEvent 事件给出 user-service 内部的整个实现流程，如下图所示：</p>
<p data-nodeid="6659" class=""><img src="https://s0.lgstatic.com/i/image/M00/76/EF/CgqCHl_IvjSAX_W6AAHB35Qu21g693.png" alt="3.png" data-nodeid="6663"></p>
<div data-nodeid="6660"><p style="text-align:center">user-service 消息发布实现流程</p></div>







<p data-nodeid="3652">在 user-service 中，势必会存在一个对用户信息的修改操作，这个修改操作会上图中的触发 UserInfoChangedEvent 事件，然后该事件将被构建成一个消息并通过 UserInfoChangedSource 进行发送。UserInfoChangedSource 就是一种 Spring Cloud Stream 中的具体 Source 实现。然后 UserInfoChangedSource 使用默认的名为“output”的 Channel 进行消息发布。在案例中，我们将同时演示 Kafka 和 RabbitMQ，所以 Binder 组件分别封装了这两个消息中间件。</p>
<h3 data-nodeid="3653">实现消息发布者</h3>
<p data-nodeid="3654">站在消息处理的角度讲，这个消息发布流程并不复杂，主要的实现过程是如何使用 Spring Cloud Stream 完成 Source 组件的创建、Binder 组件的配置以及如何与 user-service 进行集成，让我们一起来看一下。</p>
<h4 data-nodeid="3655">使用 @EnableBinding 注解</h4>
<p data-nodeid="3656">无论是消息发布者还是消息消费者，首先都需要引入 spring-cloud-stream 依赖，如下所示：</p>
<pre class="lang-xml" data-nodeid="3657"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-stream<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="3658">而在 SpringHealth 案例中，如果我们使用 Kafka 作为我们的消息中间件系统，那么也需要引入 spring-cloud-starter-stream-kafka 依赖，如下所示。</p>
<pre class="lang-xml" data-nodeid="3659"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-stream-kafka<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="3660">对应的，RabbitMQ 就需要引入 spring-cloud-starter-stream-rabbit 依赖，如下所示：</p>
<pre class="lang-xml" data-nodeid="3661"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-stream-rabbit<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="3662">对于消息发布者而言，它在 Spring Cloud Stream 体系中扮演着 Source 的角色，<a href="mailto:%E6%89%80%E4%BB%A5%E6%88%91%E4%BB%AC%E9%9C%80%E8%A6%81%E5%9C%A8user-service%E7%9A%84Bootstrap%E7%B1%BB%E4%B8%AD%E6%B7%BB%E5%8A%A0@EnableBinding(Source.class)" data-nodeid="3734">所以我们需要在 user-service 的 Bootstrap 类中标明这个 Spring</a>Boot 应用程序是一个 Source 组件。调整之后的 UserApplication 类如下所示。</p>
<pre class="lang-java" data-nodeid="3663"><code data-language="java"><span class="hljs-meta">@SpringCloudApplication</span>
<span class="hljs-meta">@EnableBinding(Source.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserApplication</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; SpringApplication.run(UserApplication.class, args);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3664">可以看到，我们在原有 UserApplication 上我们添加了一个 @EnableBinding(Source.class) 注解，该注解的作用就是告诉 Spring Cloud Stream 这个 Spring Boot 应用程序是一个消息发布者，需要绑定到消息中间件，实现两者之间的连接。@EnableBinding 注解定义比较简单，如下所示：</p>
<pre class="lang-java" data-nodeid="3665"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> EnableBinding {
&nbsp;&nbsp;&nbsp; Class&lt;?&gt;[] value() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="3666">我们可以使用一个或者多个接口作为该注解的参数。在上面的代码中，我们使用了 Source 接口，表示与消息中间件绑定的是一个消息发布者。在下一课时中，我们在介绍消息消费者时同样也会使用到这个 @EnableBinding 注解。</p>
<h4 data-nodeid="3667">定义 Event</h4>
<p data-nodeid="3668">接下来，需要给出 UserInfoChangedEvent 的定义。对于事件的定义也存在一些通用的做法，事件类型、事件所对应的操作、事件对应的业务领域对象等是一个完整事件定义所必需的元素。因此，我们将 UserInfoChangedEvent 定义如下：</p>
<pre class="lang-java" data-nodeid="3669"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoChangedEvent</span></span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//事件类型</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String type;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//事件所对应的操作</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String operation;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//事件对应的领域模型</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> User user;
}
</code></pre>
<p data-nodeid="3670">定义完事件的数据结构之后，接下来我们就需要通过 Source 接口来具体实现消息的发布。</p>
<h4 data-nodeid="3671">创建 Source</h4>
<p data-nodeid="3672">在 Spring Cloud Stream 中，Source 是一个接口，包含了一个发送消息的 MessageChannel，让我们简单回顾一下该接口的定义，如下所示：</p>
<pre class="lang-java" data-nodeid="3673"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Source</span> </span>{
&nbsp;&nbsp;&nbsp; String OUTPUT = <span class="hljs-string">"output"</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Output(Source.OUTPUT)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">MessageChannel <span class="hljs-title">output</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="3674">使用这个接口的方式也很简单，我们只需要在业务代码中直接进行注入即可，就像在使用一个普通的 Javabean 一样。完整的 UserInfoChangedSource 类如下所示：</p>
<pre class="lang-java" data-nodeid="3675"><code data-language="java"><span class="hljs-keyword">import</span> org.springframework.cloud.stream.messaging.Source;
<span class="hljs-keyword">import</span> org.springframework.messaging.support.MessageBuilder;
…
	&nbsp;
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoChangedSource</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Source source;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Logger logger = LoggerFactory.getLogger(UserInfoChangedSource.class);
&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">UserInfoChangedSource</span><span class="hljs-params">(Source source)</span></span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.source = source;
&nbsp;&nbsp;&nbsp; }
&nbsp;
	<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishUserInfoChangedEvent</span><span class="hljs-params">(UserInfoOperation operation, User user)</span></span>{
	&nbsp;
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Sending message for UserId: {}"</span>, user.getId());
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserInfoChangedEvent change =&nbsp; <span class="hljs-keyword">new</span> UserInfoChangedEvent(
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;UserInfoChangedEvent.class.getTypeName(),
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;operation.toString(),
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;user);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; source.output().send(MessageBuilder.withPayload(change).build());
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishUserInfoAddedEvent</span><span class="hljs-params">(User user)</span> </span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; publishUserInfoChangedEvent(UserInfoOperation.ADD, user);
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishUserInfoUpdatedEvent</span><span class="hljs-params">(User user)</span> </span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; publishUserInfoChangedEvent(UserInfoOperation.UPDATE, user);
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">publishUserInfoDeletedEvent</span><span class="hljs-params">(User user)</span> </span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; publishUserInfoChangedEvent(UserInfoOperation.DELETE, user);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3676">可以看到我们创建了一个 publishUserInfoChangedEvent 方法，在该方法中，我们首先构建了 UserInfoChangedEvent 事件并通过 Spring Messaging 模块所提供的 MessageBuilder 工具类将它转换为消息中间件所能发送的Message对象。然后，我们调用 Source 接口的 output() 方法将事件发送出去，这里的 output() 方法背后使用的就是一个具体的 MessageChannel。</p>
<h4 data-nodeid="3677">配置 Binder</h4>
<p data-nodeid="3678">为了通过 UserInfoChangedSource 将代表 UserInfoChangedEvent 的消息发送到正确的地址，我们需要在 application.yml 配置文件中配置 Binder 信息。Binder 信息中存在一些通用的配置项，例如如果要想把消息发布到消息中间件，就需要知道消息所发送的通道或者说目的地 Destination，以及序列化方式，如下所示：</p>
<pre class="lang-xml" data-nodeid="3679"><code data-language="xml">spring:
&nbsp; cloud:
&nbsp;&nbsp;&nbsp; stream:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bindings:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; destination:&nbsp; userInfoChangedTopic
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; content-type: application/json
</code></pre>
<p data-nodeid="3680">另一方面，因为 Binder 完成了与具体消息中间件的整合过程，所以需要针对特定的消息中间件来提供专门的配置项。我们先来看在使用 Kafka 的场景下 Binder 的配置方法，相关配置项如下所示：</p>
<pre class="lang-xml" data-nodeid="3681"><code data-language="xml">spring:
&nbsp; cloud:
&nbsp;&nbsp;&nbsp; stream:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bindings:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; destination:&nbsp; userInfoChangedTopic
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; content-type: application/json
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; kafka:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; binder:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; zk-nodes: localhost
	&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; brokers: localhost
</code></pre>
<p data-nodeid="3682">在以上配置项中，除了前面介绍的通用配置型之外，因为 Kafka 的运行依赖于 Zookeeper，所以“kafka”配置段使用 Kafka 作为消息中间件平台，并将其 Zookeeper 地址以及 Kafka 自身的地址都指向了本地。</p>
<p data-nodeid="3683">相比 Kafka，RabbitMQ 的配置稍微复杂一点，如下所示：</p>
<pre class="lang-xml" data-nodeid="3684"><code data-language="xml">spring:
&nbsp; cloud:
&nbsp;&nbsp;&nbsp; stream:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; bindings:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; default:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; content-type: application/json
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; binder: rabbitmq
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; destination: userInfoChangedExchange
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; contentType: application/json&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; binders:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rabbitmq:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; type: rabbit
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; environment:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; spring:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rabbitmq:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; host: 127.0.0.1
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; port: 5672
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; username: guest
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; password: guest
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; virtual-host: /
</code></pre>
<p data-nodeid="3685">在以上配置项中，我们设置了 destination为userInfoChangedExchange 后会在 RabbitMQ  中创建一个名为“userInfoChangedExchange”的交换器，并把 Spring Cloud Stream 的消息输出通道绑定到该交换器。同时，我们在 bindings 配置段中指定了一个“default”子配置段，用于指定默认所使用的 binder。在这个示例中，我们将这个默认 binder 命名为“rabbitmq”并在“binders”配置段中指定了运行 RabbitMQ 的相关参数。请注意 RabbitMQ 和 Kafka 这两款消息中间件在配置方式上各个配置项的层级以及内容上的差别。</p>
<h4 data-nodeid="3686">集成服务</h4>
<p data-nodeid="3687">最后，我们要做的事情就是在 user-service 中集成消息发布功能。在前一版本的 UserService 类的基础之上，我们添加对 UserInfoChangedSource 的使用过程，如下所示：</p>
<pre class="lang-java" data-nodeid="3688"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserService</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserRepository userRepository;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserInfoChangedSource userInfoChangedSource;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">getUserById</span><span class="hljs-params">(Long userId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> userRepository.findById(userId).orElse(<span class="hljs-keyword">null</span>); 
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">getUserByUserName</span><span class="hljs-params">(String userName)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> userRepository.findUserByUserName(userName);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addUser</span><span class="hljs-params">(User user)</span></span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userRepository.save(user);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userInfoChangedSource.publishUserInfoAddedEvent(user);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">updateUser</span><span class="hljs-params">(User user)</span></span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userRepository.save(user);
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userInfoChangedSource.publishUserInfoUpdatedEvent(user);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteUser</span><span class="hljs-params">(User user)</span></span>{
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userRepository.delete(user);
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; userInfoChangedSource.publishUserInfoDeletedEvent(user);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3689">可以看到，我们在增加、修改和删除用户操作时都添加了发布用户信息变更事件的机制。注意到在 UserService 中我们并没有构建具体的 UserInfoChangedEvent 事件，而是把这部分操作放在了 UserInfoChangedSource中，目的也是为了降低各个层次之间的依赖关系，并封装对事件的统一操作。</p>
<p data-nodeid="3690">至此，完整的消息发布者实现完毕。接下来，我们来看看消息消费场景应该如何进行设计。</p>
<h3 data-nodeid="3691">设计 SpringHealth 中的消息消费场景</h3>
<p data-nodeid="3692">我们继续讨论 SpringHealth 案例，根据整个消息交互流程，intervention-service 就是 UserInfoChangedEvent 事件的消费者。作为该事件的消费者，intervention-service 需要把变更后的用户信息更新到缓存中。</p>
<p data-nodeid="7428">在 Spring Cloud Stream 中，负责消费消息的是 Sink 组件，因此，我们同样围绕 UserInfoChangedEvent 事件给出 intervention-service 内部的整个实现流程，如下图所示：</p>
<p data-nodeid="7429" class=""><img src="https://s0.lgstatic.com/i/image/M00/76/F0/CgqCHl_IvlKADfCXAAA8UAK4iIs978.png" alt="Drawing 3.png" data-nodeid="7434"></p>
<div data-nodeid="7430"><p style="text-align:center">intervention-service 消息消费实现流程</p></div>





<p data-nodeid="3696">在上图中，UserInfoChangedEvent 事件通过消息中间件发送到 Spring Cloud Stream 中，Spring Cloud Stream 通过 Sink 获取消息并交由 UserInfoChangedSink 实现具体的消费逻辑。可以想象在这个 UserInfoChangedSink 中会负责实现缓存相关的处理逻辑。</p>
<p data-nodeid="3697">让我们把消息消费过程与 intervention-service 中的业务流程串联起来。我们知道在 intervention-service 中存在 UserServiceClient 类，其核心方法 getUserByUserName 如下所示：</p>
<pre class="lang-java" data-nodeid="3698"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> UserMapper <span class="hljs-title">getUserByUserName</span><span class="hljs-params">(String userName)</span></span>{
&nbsp;&nbsp;&nbsp;  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResponseEntity&lt;UserMapper&gt; restExchange =
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; restTemplate.exchange(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"http://zuulservice:5555/springhealth/user/users/username/{userName}"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; HttpMethod.GET,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">null</span>, UserMapper.class, userName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserMapper user = restExchange.getBody();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user;
}
</code></pre>
<p data-nodeid="3699">这里我们直接通过调用 user-service 远程获取 User 信息。我们知道用户账户信息变更是一个低频事件，而每次通过 UserServiceClient 实现远程调用的成本很高且没有必要。现在我们可以通过 Spring Cloud Stream 获取用户信息更新的消息了，UserServiceClient 就有了优化的空间。基本思路就是缓存用户信息，并通过消息触发缓存更新，然后我们先从缓存中获取用户信息，只有在缓存中找不到对应的用户信息时才会发起远程调用。下图展示了采用这一设计思想之后的流程图：</p>
<p data-nodeid="8216" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/76/F0/CgqCHl_IvmGADKwmAAFJvHum5Z8651.png" alt="5.png" data-nodeid="8220"></p>
<div data-nodeid="8217"><p style="text-align:center">用户账户更新流程图</p></div>




<p data-nodeid="3701">在上图中，我们看到 user-service 异步发送的 UserInfoChangedEvent 事件会被消费，该消息的处理器 UserInfoChangedSink 所消费，然后 UserInfoChangedSink 将更新后的用户账户信息进行缓存以供 intervertion-service 使用。显然，UserInfoChangedSink 是整个流程的关键。至于如何实现这个 UserInfoChangedSink，我们放在下一课时中进行详细展开并给出代码示例。</p>
<h3 data-nodeid="3702">小结与预告</h3>
<p data-nodeid="3703">今天，我们基于用户信息更新这一特定业务场景，介绍了使用 Spring Cloud Stream 来完成对 SpringHealth 系统中消息发布消费流程的建模，并提供了针对消息发布者的实现过程。可以看到，只要理解了 Spring Cloud Stream 的基本架构，使用该框架发送消息的开发更多的是配置工作。</p>
<p data-nodeid="3704">这里给你留一道思考题：在 Spring Cloud Stream 配置不同的 Binder 时，有哪些公共配置项，又有哪些是针对具体消息中间件的特定配置项？</p>
<p data-nodeid="3705">下一课时将继续讨论基于 Spring Cloud Stream 的开发过程，我们关注于消息消费者的实现，以及自定义消息通道、消费者分组以及消息分区等高级主题的实现方式。</p>

---

### 精选评论

##### **旭：
> 不管是rabbitMq 还是 Kafka，还需要单独部署消息中间件环境(rabbitMq/kafka)吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，需要启动专门的服务器，Spring Cloud Stream只是一个集成性的框架，而不是消息服务器

