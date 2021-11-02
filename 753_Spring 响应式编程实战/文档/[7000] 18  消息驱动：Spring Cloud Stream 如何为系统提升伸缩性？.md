<p data-nodeid="433" class="">请你回想一下我在“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=753#/detail/pc?id=6983&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="508">01 | 追本溯源：响应式编程究竟是一种什么样的技术体系</a>”中提到的，响应式宣言认为，响应式系统的价值在于提供了即时响应性、可维护性和扩展性，表现的形式是回弹性和弹性，而实现的手段则是消息驱动。</p>
<p data-nodeid="434">今天，我们将讨论与消息驱动相关的话题，并引出 Spring 家族中另一个重要成员，即 Spring Cloud Stream。Spring Cloud Stream 专门用于构建低耦合的事件驱动架构，并提供了响应式编程组件。我将从 Spring Cloud Stream 的基本架构说起，介绍它与主流消息中间件之间的集成关系，并分别给出实现响应式消息发布者和消息消费者的具体方法。</p>
<h3 data-nodeid="435">Spring Cloud Stream 基本架构</h3>
<p data-nodeid="436">Spring Cloud Streams 为异步跨服务消息通信提供了简化的编程模型。Spring Cloud Stream 能够构建具有高度伸缩性的应用程序，而无须处理过于复杂的配置，也无须深入了解特定的消息中间件。</p>
<h4 data-nodeid="437">Spring Cloud Stream 工作流程</h4>
<p data-nodeid="438">Spring Cloud Stream 中有三个角色，即消息的发布者、消费者以及消息通信系统本身，以消息通信系统为中心，整个工作流程表现为一种对称结构，如下图所示。</p>
<p data-nodeid="576" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/3A/E1/CioPOWCBSkeATpnCAAD7HNvxzLk301.png" alt="图片7.png" data-nodeid="580"></p>
<div data-nodeid="577"><p style="text-align:center">Spring Cloud Stream 工作流程图</p></div>


<p data-nodeid="441">在上图中，充当消息发布者的服务 A 根据业务需要产生消息发送的需求，Spring Cloud Stream 中的 Source 组件是真正生成消息的组件，然后消息通过 Channel 传送到 Binder，这里的 Binder 是一个抽象组件，通过 Binder，Channel 可以与特定的消息中间件进行通信。在 Spring Cloud Stream 中，目前已经内置集成的消息中间件实现工具包括 RabbitMQ 和 Kafka。</p>
<p data-nodeid="442">另一方面，消息消费者则同样通过 Binder 从消息中间件中获取消息，消息将通过 Channel 流转到 Sink 组件。这里的 Sink 组件是服务级别的，即类似上图中服务 B 的不同服务可能会实现不同的 Sink 组件，分别对消息进行不同业务上的处理。</p>
<h4 data-nodeid="443">Spring Cloud Stream 核心组件</h4>
<p data-nodeid="444">在 Spring Cloud Stream 工作流程图中，我们不难看出其具备四个核心组件，分别是 Binder、Channel、Source 和 Sink，其中 Binder 和 Channel 成对出现，而 Source 和 Sink 分别面向消息的发布者和消费者。</p>
<ul data-nodeid="445">
<li data-nodeid="446">
<p data-nodeid="447">Binder</p>
</li>
</ul>
<p data-nodeid="448">Binder 是 Spring Cloud Stream 的一个核心概念，它充当了服务与消息中间件之间的桥梁。通过 Binder，我们可以很方便地连接 RabbitMQ、Kafka 等消息中间件。同时，Binder 组件也为我们提供了消费者分组和消息分区等特性，关于这些特性我会在“20 | 消息消费：如何选择可用的高级开发技巧？”中详细介绍。Binder 的核心价值就在于我们可以直接使用这些特性，而不需要了解其背后的各种消息中间件在实现上的差异。</p>
<ul data-nodeid="449">
<li data-nodeid="450">
<p data-nodeid="451">Channel</p>
</li>
</ul>
<p data-nodeid="452">Channel 即通道，是对队列（Queue）的一种抽象。我们知道在消息中间件中，队列的作用就是实现存储转发的媒介，消息发布者所生成的消息都将保存在队列中并由消息消费者进行消费。通道的名称对应的就是队列的名称，但是作为一种抽象和封装，各个消息中间件所特有的队列概念并不会直接暴露在业务代码中，而是通过通道来对队列进行配置。</p>
<ul data-nodeid="453">
<li data-nodeid="454">
<p data-nodeid="455">Source 和 Sink</p>
</li>
</ul>
<p data-nodeid="456">我们可以把 Source 和 Sink 简单理解为输出和输入，但还是要明确这里输入输出的参照对象是 Spring Cloud Stream 自身，即从 Spring Cloud Stream 发布消息的组件就是 Source，而通过 Spring Cloud Stream 接收消息的就是 Sink。</p>
<p data-nodeid="457">在 Spring Cloud Stream 中，表面上 Source 组件是使用一个 POJO 对象来作为需要发布的消息，通过将该对象进行序列化（默认的序列化方式是 JSON）然后发布到通道中。另一方面，Sink 组件监听通道并等待消息的到来，一旦有可用消息，Sink 将该消息反序列化为一个 POJO 对象并用于处理业务逻辑。而在内部，Spring Cloud Stream 在实现这一过程中需要借助 Spring 家族中的底层消息处理机制。</p>
<h4 data-nodeid="458">Spring Cloud Stream 与 Spring 消息处理机制</h4>
<p data-nodeid="459">在了解了 Spring Cloud Stream 的基本流程和核心组件之后，我们来看一下该框架背后的实现机制。事实上，Spring Cloud Streams 模块构建在 Spring 家族中的 Spring Messaging 模块之上，而后者是与外部服务和异步消息通信进行集成的基本抽象。下面我将为你简要介绍 Spring 中的底层消息通信机制，方便你在使用 Spring Cloud Stream 时对其背后的实现原理有更好的理解。</p>
<p data-nodeid="460">Spring Messaging 把通道抽象成两种基本的表现形式，即支持轮询的 PollableChannel 和实现发布/订阅模式的 SubscribableChannel，这两个通道都继承自具有消息发送功能的 MessageChannel，通道相关的定义如下所示。</p>
<pre class="lang-java" data-nodeid="461"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">MessageChannel</span> </span>{
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">send</span><span class="hljs-params">(Message message)</span></span>;
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">send</span><span class="hljs-params">(Message message, <span class="hljs-keyword">long</span> timeout)</span></span>;
}
	&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PollableChannel</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">MessageChannel</span> </span>{
	&nbsp;
	&nbsp;&nbsp;&nbsp; Message&lt;?&gt; receive();
	&nbsp;
	&nbsp;&nbsp;&nbsp; Message&lt;?&gt; receive(<span class="hljs-keyword">long</span> timeout);
}
	&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SubscribableChannel</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">MessageChannel</span> </span>{
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">subscribe</span><span class="hljs-params">(MessageHandler handler)</span></span>;
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">unsubscribe</span><span class="hljs-params">(MessageHandler handler)</span></span>;
}
</code></pre>
<p data-nodeid="462">我们注意到对于 PollableChannel 而言才有 receive() 的概念，代表这是通过轮询操作主动获取消息的过程，而 SubscribableChannel 则是通过注册回调处理器 MessageHandler 来实现事件响应。</p>
<p data-nodeid="463">结合上述消息通道的相关概念，我们就不难理解 Spring Cloud Stream 中关于 Source 和 Sink 的定义。Source 和 Sink 都是接口，其中 Source 接口的定义如下，通过 MessageChannel 来发送消息。注意这里的 @Output 注解定义的就是一个输出通道。</p>
<pre class="lang-java" data-nodeid="464"><code data-language="java"><span class="hljs-keyword">import</span> org.springframework.cloud.stream.annotation.Output;
<span class="hljs-keyword">import</span> org.springframework.messaging.MessageChannel;
	&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Source</span> </span>{
&nbsp;
&nbsp; String OUTPUT = <span class="hljs-string">"output"</span>;
&nbsp;
&nbsp; <span class="hljs-meta">@Output(Source.OUTPUT)</span>
&nbsp; <span class="hljs-function">MessageChannel <span class="hljs-title">output</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="465">类似的，Sink 接口定义如下，通过 Spring Messaging 中的 SubscribableChannel 来实现消息接收。显然，这里的 @Input 注解定义了一个输入通道，请看代码。</p>
<pre class="lang-java" data-nodeid="466"><code data-language="java"><span class="hljs-keyword">import</span> org.springframework.cloud.stream.annotation.Input;
<span class="hljs-keyword">import</span> org.springframework.messaging.SubscribableChannel;
	&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Sink</span></span>{
&nbsp;
&nbsp; String INPUT = <span class="hljs-string">"input"</span>;
&nbsp;
&nbsp; <span class="hljs-meta">@Input(Source.INPUT)</span>
&nbsp; <span class="hljs-function">SubscribableChannel <span class="hljs-title">input</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="467">@Input 注解和 @Output 注解可以使用通道名称作为参数，如果没有名称，它们会使用注解所对应的方法名字作为参数，也就是默认情况下分别使用“input”和“output”作为通道名称。从这个角度讲，一个应用程序中的 Input 和 Output 通道数量是不限制的，我们只需要对这些通道通过 @Input 和 @Output 注解进行定义即可。</p>
<p data-nodeid="468">例如在如下的接口中，我们定义了 SpringCssChannel 接口并声明了两个 Input 通道和一个 Output 通道，表明该服务会向外部的一个通道发送消息，并从外部的两个通道中接收消息。</p>
<pre class="lang-java" data-nodeid="469"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SpringCssChannel</span></span>{
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Input</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-function">SubscribableChannel <span class="hljs-title">input1</span><span class="hljs-params">()</span></span>;
	&nbsp;
	    <span class="hljs-meta">@Input</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-function">SubscribableChannel <span class="hljs-title">input2</span><span class="hljs-params">()</span></span>;
	&nbsp;
	&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Output</span>
	&nbsp;&nbsp;&nbsp; <span class="hljs-function">MessageChannel <span class="hljs-title">output1</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="470">上述接口定义中直接使用了 Spring Messaging 中的 SubscribableChannel 和 MessageChannel 接口，Spring Cloud Stream 对 Spring Messaging 提供了原生支持，我们可以使用 Spring Messaging 提供的 API 直接操作消息发布和接收的过程，但因为这些 API 偏底层且过于复杂，不适合直接面向应用程序开发，通常我们不需要也不建议使用它们。</p>
<h3 data-nodeid="471">Reactive Spring Cloud Stream 组件</h3>
<p data-nodeid="472">Spring Cloud Stream 2.x 版本还引入了基于响应式编程模型的 Reactive Spring Cloud Stream 组件，该组件提供了对响应式流的支持，从而把传入和传出的消息作为连续数据流进行处理。</p>
<p data-nodeid="473">接下来，我将在 Spring Cloud Stream 的基础上引入 Reactive Spring Cloud Stream 来实现响应式消息通信系统，首先需要在项目中添加如下 Maven 依赖。</p>
<pre class="lang-java" data-nodeid="474"><code data-language="java">&lt;dependency&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &lt;groupId&gt;org.springframework.cloud&lt;/groupId&gt;
	&nbsp; &lt;artifactId&gt;spring-cloud-stream-reactive&lt;/artifactId&gt;
&lt;/dependency&gt;
</code></pre>
<p data-nodeid="475">与 Spring Cloud Stream 一样，在 Reactive Spring Cloud Stream 中同样提供了响应式 Source 组件和 Sink 组件，它们在使用方式上与传统的 Source 组件和 Sink 组件有一定区别，这点对于响应式 Source 组件而言尤为明显。</p>
<h4 data-nodeid="476">响应式 Source 组件</h4>
<p data-nodeid="477">响应式 Spring Cloud Stream 支持通过 @StreamEmitter 注解来实现响应式 Source 组件。通过 @StreamEmitter 注解，我们可以把一个传统的 Source 组件转变成响应式组件。</p>
<p data-nodeid="478">@StreamEmitter 是一个方法级别的注解，通过该注解可以把方法转变成一个 Emitter。我们在使用 @StreamEmitter 注解时只能与 @Output 注解进行组合，因为 @StreamEmitter 注解的作用就是生产消息。</p>
<p data-nodeid="479">@StreamEmitter 注解的使用方法非常多样，例如我们可以构建如下所示的 ReactiveSourceApplication 类。这里 emit() 方法的作用是每秒发射一个 "Hello World" 字符串到一个 Reactor Flux 对象，而该 Flux 对象则会被发送到 Source 组件默认的“output”通道。</p>
<pre class="lang-java" data-nodeid="480"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableBinding(Source.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveSourceApplication</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@StreamEmitter</span>
&nbsp; <span class="hljs-meta">@Output(Source.OUTPUT)</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;String&gt; <span class="hljs-title">emit</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Flux.interval(Duration.ofSeconds(<span class="hljs-number">1</span>)).map(l -&gt; <span class="hljs-string">"Hello World"</span>);
&nbsp; }
}
</code></pre>
<p data-nodeid="481">如下代码演示了另一种使用 @StreamEmitter 注解的方式。你可以注意到，这里的 emit() 方法不是直接返回一个 Flux 对象，而是使用 FluxSender 工具类发送 Flux 对象到 Source 组件。</p>
<pre class="lang-java" data-nodeid="482"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableBinding(Source.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveSourceApplication</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@StreamEmitter</span>
&nbsp; <span class="hljs-meta">@Output(Source.OUTPUT)</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">emit</span><span class="hljs-params">(FluxSender output)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.send(Flux.interval(Duration.ofSeconds(<span class="hljs-number">1</span>)).map(l -&gt; <span class="hljs-string">"Hello World"</span>));
&nbsp; }
}
</code></pre>
<p data-nodeid="483">上述代码中我们也可以把 @Output(Source.OUTPUT) 注解从方法名移到方法参数上，两者效果完全一致，如下所示。</p>
<pre class="lang-java" data-nodeid="484"><code data-language="java"><span class="hljs-meta">@StreamEmitter</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">emit</span><span class="hljs-params">(<span class="hljs-meta">@Output(Source.OUTPUT)</span> FluxSender output)</span> </span>{
      output.send(Flux.interval(Duration.ofSeconds(<span class="hljs-number">1</span>)).map(l -&gt; <span class="hljs-string">"Hello World"</span>));
}
</code></pre>
<h4 data-nodeid="485">响应式 Sink 组件</h4>
<p data-nodeid="486">有了前面的基础，就不难理解构建响应式 Sink 的方法。我们可以使用 @StreamListener 注解来实现消息的消费。示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="487"><code data-language="java"><span class="hljs-meta">@EnableBinding(Sink.class)</span>
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SinkApplication</span> </span>{
&nbsp;
&nbsp; <span class="hljs-meta">@StreamListener</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Flux&lt;String&gt; <span class="hljs-title">receive</span><span class="hljs-params">(<span class="hljs-meta">@Input(Sink.INPUT)</span> Flux&lt;String&gt; input)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> input.map(s -&gt; s.toUpperCase());
&nbsp; }
}
</code></pre>
<p data-nodeid="488">@StreamListener 并不是一个新的注解，在传统的 Spring Cloud Stream 中就已经存在了。将 @StreamListener 注解添加到某个方法上，就可以使之接收由通道传入的事件。如下代码展示了另一种使用 @StreamListener 注解的方法，我们直接在该注解中指定它的 target 为 Sink.INPUT，并在 loggerSink() 方法中传入 Flux 对象。</p>
<pre class="lang-java" data-nodeid="489"><code data-language="java"><span class="hljs-meta">@EnableBinding(Sink.class)</span>
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveSinkApplication</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Logger logger = LoggerFactory.getLogger(SinkApplication.class);
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@StreamListener(target = Sink.INPUT)</span>
&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">loggerSink</span><span class="hljs-params">(Flux&lt;String&gt; inputs)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; inputs.map(String::toUpperCase)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;.subscribe(input -&gt; logger.info(<span class="hljs-string">"Received: {}"</span>, input));
    }
}
</code></pre>
<h4 data-nodeid="490">Processor 组件</h4>
<p data-nodeid="491">在 Spring Cloud Stream 中还存在 Processor 组件，可以把该组件理解为是一种集成 Source 和 Sink 的双向通道，Processor 接口定义如下所示。</p>
<pre class="lang-java" data-nodeid="492"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Processor</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Source</span>, <span class="hljs-title">Sink</span> </span>{
&nbsp;
}
</code></pre>
<p data-nodeid="493">Processor 可用于同时具备 Input 通道和 Output 通道的应用程序，使用 Processor 的示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="494"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableBinding(Processor.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ReactiveSourceApplication</span> </span>{
&nbsp;
 <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">receive</span><span class="hljs-params">(<span class="hljs-meta">@Input(Processor.INPUT)</span> Flux&lt;String&gt; input, <span class="hljs-meta">@Output(Processor.OUTPUT)</span> FluxSender output)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; output.send(input.map(s -&gt; s.toUpperCase()));
&nbsp; }
}
</code></pre>
<p data-nodeid="495">上述代码中，我们一方面从 Processor.INPUT 通道中获取 Flux 对象。同时，也通过 Processor.OUTPUT 通道对外发送消息。</p>
<p data-nodeid="496">好了，关于 Reactive Spring Cloud Stream 组件就介绍到这，你可以根据我的演示，自己简单操作一下，以便更了解这部分内容。</p>
<h3 data-nodeid="497">小结与预告</h3>
<p data-nodeid="498">Spring Cloud Stream 是 Spring Cloud 中针对消息处理的一款平台型框架，该框架的<strong data-nodeid="568">核心优势在于在内部集成了主流消息中间件，而对外则提供了统一的 API 接入层</strong>。而 Reactive Spring Cloud Stream 是 Spring Cloud Stream 的响应式版本，基于响应式流完成对消息通信过程的处理。这一讲我们对 Reactive Spring Cloud Stream 进行了讨论，并重点分析了它所具备的响应式编程组件。</p>
<p data-nodeid="499">最后给你留一道思考题：在 Reactive Spring Cloud Stream 中，发送消息和消费消息分别可以使用什么注解？</p>
<p data-nodeid="500">在明确了 Reactive Spring Cloud Stream 的基本架构之后，在接下来的两讲中，我将结合 ReactiveSpringCSS 案例为你介绍如何使用它来实现响应式消息发布者和消费者，我们到时候见。</p>
<blockquote data-nodeid="501">
<p data-nodeid="502" class="">点击链接，获取课程相关代码 ↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="575">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### *瑾：
> 老师，在自定义订阅者，为什么建议继承BaseSubscriber，而不直接使用Subscriber？可以分别解释下区别和好处吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 直接实现Subscriber接口难度较大，需要实现很多底层方法，也有点属于重复造轮子。BaseSubscriber已经包含了很多既有功能，开发人员只需要关注业务上的处理即可

