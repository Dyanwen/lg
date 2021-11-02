<p data-nodeid="4468" class="">通过前两讲的内容可以知道，Reactor 框架为我们提供了各种操作符，使用这些操作符可以高效地操作 Flux 和 Mono 对象。Reactor 中的操作符可以分成不同的类型，上一讲我们关注转换、过滤和组合类的操作符，而今天我将继续为你介绍剩余的条件、裁剪、工具类的操作符。</p>
<h3 data-nodeid="4469">条件操作符</h3>
<p data-nodeid="4470">所谓条件操作符，本质上就是提供了一个判断的依据来确定是否处理流中的元素。Reactor 中常用的条件操作符有 defaultIfEmpty、takeUntil、takeWhile、skipUntil 和 skipWhile 等，下面我将分别介绍。</p>
<h4 data-nodeid="4471">defaultIfEmpty 操作符</h4>
<p data-nodeid="4472">defaultIfEmpty 操作符针对空数据流提供了一个简单而有用的处理方法。该操作符用来返回来自原始数据流的元素，如果原始数据流中没有元素，则返回一个默认元素。</p>
<p data-nodeid="4473">defaultIfEmpty 操作符在实际开发过程中应用广泛，通常用在对方法返回值的处理上。如下所示的就是在 Controller 层中对 Service 层返回结果的一种常见处理方法。</p>
<pre class="lang-java" data-nodeid="4474"><code data-language="java"><span class="hljs-meta">@GetMapping("/orders/{id}")</span>
<span class="hljs-keyword">public</span> Mono&lt;ResponseEntity&lt;Order&gt;&gt; findOrderById(<span class="hljs-meta">@PathVariable</span> 
	String id) {
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderService.findOrderById(id)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(ResponseEntity::ok)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .defaultIfEmpty(ResponseEntity
	.status(<span class="hljs-number">404</span>).body(<span class="hljs-keyword">null</span>));
}
</code></pre>
<p data-nodeid="4475">可以看到，这里使用 defaultIfEmpty 操作符实现默认返回值。在示例代码所展示的 HTTP 端点中，当找不到指定的数据时，我们可以通过 defaultIfEmpty 方法返回一个空对象以及 404 状态码。</p>
<h4 data-nodeid="4476">takeUntil/takeWhile 操作符</h4>
<p data-nodeid="4477">takeUntil 操作符的基本用法是 takeUntil (Predicate&lt;? super T&gt; predicate)，其中 Predicate 代表一种断言条件，该操作符将从数据流中提取元素直到断言条件返回 true。takeUntil 的示例代码如下所示，我们希望从一个包含 100 个连续元素的序列中获取 1~10 个元素。</p>
<pre class="lang-java" data-nodeid="4478"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">100</span>).takeUntil(i -&gt; i == <span class="hljs-number">10</span>)
	.subscribe(System.out::println);
</code></pre>
<p data-nodeid="4479">类似的，takeWhile 操作符的基本用法是 takeWhile (Predicate&lt;? super T&gt; continuePredicate)，其中 continuePredicate 代表的也是一种断言条件。与 takeUntil 不同的是，takeWhile 会在 continuePredicate 条件返回 true 时才进行元素的提取。takeWhile 的示例代码如下所示，这段代码的执行效果与 takeUntil 的示例代码一致。</p>
<pre class="lang-java" data-nodeid="4480"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">100</span>).takeWhile(i -&gt; i &lt;= <span class="hljs-number">10</span>)
	.subscribe(System.out::println);
</code></pre>
<p data-nodeid="4481">讲到这里，让我们回顾上一讲介绍的第一个转换操作符 buffer。事实上，Reactor 框架中同样也提供了 bufferUntil 和 bufferWhile 操作符来实现数据收集，这两个操作符用到了和 takeUntil/takeWhile 完全一样的断言机制，如下代码演示了 bufferUntil 的使用方法。</p>
<pre class="lang-java" data-nodeid="4482"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">10</span>).bufferUntil(i -&gt; i % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>)
	.subscribe(System.out::println);
</code></pre>
<p data-nodeid="4483">以上代码的执行结果如下所示，这里所设置的断言就是“i % 2 == 0”这一条件。</p>
<pre class="lang-xml" data-nodeid="4484"><code data-language="xml">[1, 2]
[3, 4]
[5, 6]
[7, 8]
[9, 10]
</code></pre>
<p data-nodeid="4485">对应的，bufferWhile 的使用方法和执行结果分别如下所示。</p>
<pre class="lang-java" data-nodeid="4486"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">10</span>).bufferWhile(i -&gt; i % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>)
	.subscribe(System.out::println);
</code></pre>
<pre class="lang-xml" data-nodeid="4487"><code data-language="xml">[2]
[4]
[6]
[8]
[10]
</code></pre>
<h4 data-nodeid="4488">skipUntil/skipWhile 操作符</h4>
<p data-nodeid="4489">与 takeUntil 相对应，skipUntil 操作符的基本用法是 skipUntil (Predicate&lt;? super T&gt; predicate)。skipUntil 将丢弃原始数据流中的元素直到 Predicate 返回 true。</p>
<p data-nodeid="4490">同样，与 takeWhile 相对应，skipWhile 操作符的基本用法是 skipWhile (Predicate&lt;? super T&gt; continuePredicate)，当 continuePredicate 返回 true 时才进行元素的丢弃。这两个操作符都很简单，就不具体展开讨论了。</p>
<p data-nodeid="4491">下面来说说裁剪操作符。</p>
<h3 data-nodeid="4492">裁剪操作符</h3>
<p data-nodeid="4493">裁剪操作符通常用于统计流中的元素数量，或者检查元素是否具有一定的属性。在 Reactor 中，常用的裁剪操作符有 any 、concat、count 和 reduce 等。</p>
<h4 data-nodeid="4494">any 操作符</h4>
<p data-nodeid="4495">any 操作符用于检查是否至少有一个元素具有所指定的属性，示例代码如下。</p>
<pre class="lang-java" data-nodeid="4496"><code data-language="java">Flux.just(<span class="hljs-number">3</span>, <span class="hljs-number">5</span>, <span class="hljs-number">7</span>, <span class="hljs-number">9</span>, <span class="hljs-number">11</span>, <span class="hljs-number">15</span>, <span class="hljs-number">16</span>, <span class="hljs-number">17</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .any(e -&gt; e % <span class="hljs-number">2</span> == <span class="hljs-number">0</span>)
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; .subscribe(isExisted -&gt; System.out.println(isExisted));
</code></pre>
<p data-nodeid="4497">在这个 Flux 流中存在一个元素 16 可以被 2 除尽，所以控制台将输出“true”。</p>
<p data-nodeid="4498">类似的，Reactor 中还存在一个 all 操作符，用来检查流中元素是否都满足同一属性，示例代码如下所示。</p>
<pre class="lang-java" data-nodeid="4499"><code data-language="java">Flux.just(<span class="hljs-string">"abc"</span>, <span class="hljs-string">"ela"</span>, <span class="hljs-string">"ade"</span>, <span class="hljs-string">"pqa"</span>, <span class="hljs-string">"kang"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .all(a -&gt; a.contains(<span class="hljs-string">"a"</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .subscribe(isAllContained -&gt; System.out.println(isAllContained));
</code></pre>
<p data-nodeid="4500">显然，在这个 Flux 流中所有元素都包含了字符“a”，所以控制台也将输出“true”。</p>
<h4 data-nodeid="4501">concat 操作符</h4>
<p data-nodeid="4502">concat 操作符用来合并来自不同 Flux 的数据。与上一讲中所介绍的 merge 操作符不同，这种合并采用的是顺序的方式，所以严格意义上并不是一种合并操作，所以我们把它归到裁剪操作符类别中。</p>
<p data-nodeid="4503">例如，如果执行下面这段代码，我们将在控制台中依次看到 1 到 10 这 10 个数字。</p>
<pre class="lang-java" data-nodeid="4504"><code data-language="java">Flux.concat(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">3</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Flux.range(<span class="hljs-number">4</span>, <span class="hljs-number">2</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Flux.range(<span class="hljs-number">6</span>, <span class="hljs-number">5</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ).subscribe(System.out::println);
};
</code></pre>
<h4 data-nodeid="4505">reduce 操作符</h4>
<p data-nodeid="4506">裁剪操作符中最经典的就是这个 reduce 操作符。reduce 操作符对来自 Flux 序列中的所有元素进行累积操作并得到一个 Mono 序列，该 Mono 序列中包含了最终的计算结果。reduce 操作符示意图如下所示。</p>
<p data-nodeid="4507"><img src="https://s0.lgstatic.com/i/image6/M00/2C/98/CioPOWBlYDaAMg1pAAPcwZ2XS_I628.png" alt="Drawing 1.png" data-nodeid="4599"></p>
<div data-nodeid="4508"><p style="text-align:center">reduce 操作符示意图（来自 Reactor 官网）</p></div>
<p data-nodeid="4509">在上图中，具体的累积计算很简单，我们也可以通过一个 BiFunction 来实现任何自定义的复杂计算逻辑。reduce 操作符的示例代码如下所示，这里的 BiFunction 就是一个求和函数，用来对 1 到 10 的数字进行求和，运行结果为 55。</p>
<pre class="lang-java" data-nodeid="4510"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">10</span>).reduce((x, y) -&gt; x + y)
	.subscribe(System.out::println);
</code></pre>
<p data-nodeid="4511">与 reduce 操作符类似的还有一个 reduceWith 操作符，用来在 reduce 操作时指定一个初始值。reduceWith 操作符的代码示例如下所示，我们使用 5 来初始化求和过程，显然得到的结果将是 60。</p>
<pre class="lang-java" data-nodeid="4512"><code data-language="java">Flux.range(<span class="hljs-number">1</span>, <span class="hljs-number">10</span>).reduceWith(() -&gt; <span class="hljs-number">5</span>, (x, y) -&gt; x + y)
	.subscribe(System.out::println);
</code></pre>
<p data-nodeid="4513">以上就是三种裁剪操作符的介绍，应该很好理解，下面我们来看看工具操作符。</p>
<h3 data-nodeid="4514">工具操作符</h3>
<p data-nodeid="4515">Reactor 中常用的工具操作符有 subscribe、timeout、block、log 和 debug 等。</p>
<h4 data-nodeid="4516">subscribe 操作符</h4>
<p data-nodeid="4517">说起 subscribe 操作符，我已经在“06 | 流式操作：如何使用 Flux 和 Mono 高效构建响应式数据流”中讲到订阅响应式流时介绍过很多，这里再带你回顾一下通过该操作符订阅序列的最通用方式，如下所示。</p>
<pre class="lang-java" data-nodeid="4518"><code data-language="java"><span class="hljs-comment">//订阅序列的最通用方式，可以为我们的Subscriber实现提供所需的任意行为</span>
subscribe(Subscriber&lt;T&gt; subscriber);
</code></pre>
<p data-nodeid="4519">基于这种方式，如果默认的 subscribe() 方法没有提供所需的功能，我们可以实现自己的 Subscriber。一般而言，我们总是可以直接实现响应式流规范所提供的 Subscriber 接口，并将其订阅到流。实现一个自定义的 Subscriber 并没有想象中那么困难，这里我给你演示一个简单的实现示例。</p>
<pre class="lang-java" data-nodeid="4520"><code data-language="java">Subscriber&lt;String&gt; subscriber = <span class="hljs-keyword">new</span> Subscriber&lt;String&gt;() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">volatile</span> Subscription subscription; 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onSubscribe</span><span class="hljs-params">(Subscription s)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; subscription = s;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"initialization"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; subscription.request(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onNext</span><span class="hljs-params">(String s)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"onNext:"</span> + s);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; subscription.request(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onComplete</span><span class="hljs-params">()</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"onComplete"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">onError</span><span class="hljs-params">(Throwable t)</span> </span>{ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"onError:"</span> + t.getMessage());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
};
</code></pre>
<p data-nodeid="4521">在这个自定义 Subscriber 实现中，我们首先持有对订阅令牌 Subscription 的引用。由于订阅和数据处理可能发生在不同的线程中，因此我们使用 volatile 关键字来确保所有线程都具有对 Subscription 实例的正确引用。</p>
<p data-nodeid="4522">当订阅到达时，我们会通过 onSubscribe 回调通知 Subscriber。在这里，我们保存订阅令牌并初始化请求。</p>
<p data-nodeid="4523">你应该注意到，在 onNext 回调中，我们打印接收到的数据并请求下一个元素。在这种情况下，我们执行 subscription.request(1) 方法，也就是说使用简单的拉模型来管理背压。</p>
<p data-nodeid="4524">剩下的 onComplete 和 onError 方法我们都只是打印了一下日志。</p>
<p data-nodeid="4525">现在，让我们通过 subscribe() 方法来使用这个 Subscriber，如下所示。</p>
<pre class="lang-java" data-nodeid="4526"><code data-language="java">Flux&lt;String&gt; flux = Flux.just(<span class="hljs-string">"12"</span>, <span class="hljs-string">"23"</span>, <span class="hljs-string">"34"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; flux.subscribe(subscriber);
</code></pre>
<p data-nodeid="4527">上述代码应该产生以下控制台输出的结果。</p>
<pre class="lang-xml" data-nodeid="4528"><code data-language="xml">initialization
onNext:12
onNext:23
onNext:34
onComplete
</code></pre>
<p data-nodeid="4529">前面构建的自定义 Subscriber 虽然能够正常运作，但因为过于偏底层，因此并不推荐你使用。我们推荐的方法是扩展 Project Reactor 提供的 BaseSubscriber 类。在这种情况下，订阅者可能如下所示。</p>
<pre class="lang-java" data-nodeid="4530"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MySubscriber</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseSubscriber</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hookOnSubscribe</span><span class="hljs-params">(Subscription subscription)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"initialization"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; request(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">hookOnNext</span><span class="hljs-params">(T value)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.println(<span class="hljs-string">"onNext:"</span> + value);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; request(<span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="4531">可以看到这里使用了两个钩子方法：hookOnSubscribe(Subscription) 和 hookOnNext(T)。和这两个方法一起，我们可以重载诸如 hookOnError(Throwable)、hookOnCancel()、hookOnComplete() 等方法。</p>
<h4 data-nodeid="4532">timeout 操作符</h4>
<p data-nodeid="4533">timeout 操作符非常简单，保持原始的流发布者，当特定时间段内没有产生任何事件时，将生成一个异常。</p>
<h4 data-nodeid="4534">block 操作符</h4>
<p data-nodeid="4535">顾名思义，block 操作符在接收到下一个元素之前会一直阻塞。block 操作符常用来把响应式数据流转换为传统数据流。例如，使用如下方法将分别把 Flux 数据流和 Mono 数据流转变成普通的 List<code data-backticks="1" data-nodeid="4622">&lt;Order&gt;</code> 对象和单个的 Order 对象，我们同样可以设置 block 操作的等待时间。</p>
<pre class="lang-java" data-nodeid="4536"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Order&gt; <span class="hljs-title">getAllOrders</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderservice.getAllOrders()
	.block(Duration.ofSecond(<span class="hljs-number">5</span>));
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">getOrderById</span><span class="hljs-params">(Long orderId)</span> </span>{
&nbsp; <span class="hljs-keyword">return</span> orderservice.getOrderById(orderId)
	.block(Duration.ofSecond(<span class="hljs-number">2</span>));
}
</code></pre>
<h4 data-nodeid="4537">log 操作符</h4>
<p data-nodeid="4538">Reactor 中专门提供了针对日志的工具操作符 log，它会观察所有的数据并使用日志工具进行跟踪。我们可以通过如下代码演示 log 操作符的使用方法，在 Flux.just() 方法后直接添加 log() 函数。</p>
<pre class="lang-java" data-nodeid="4539"><code data-language="java">Flux.just(<span class="hljs-number">1</span>, <span class="hljs-number">2</span>).log().subscribe(System.out::println);
</code></pre>
<p data-nodeid="4540">以上代码的执行结果如下所示（为了显示简洁，部分内容和格式做了调整）。通常，我们也可以在 log() 方法中添加参数来指定日志分类的名称。</p>
<pre class="lang-xml" data-nodeid="4541"><code data-language="xml">Info: | onSubscribe([Synchronous Fuseable] FluxArray.ArraySubscription)
Info: | request(unbounded)
Info: | onNext(1)
1
Info: | onNext(2)
2
Info: | onComplete()
</code></pre>
<h4 data-nodeid="4542">debug 操作符</h4>
<p data-nodeid="4543">在“01 | 追本溯源：响应式编程究竟是一种什么样的技术体系”中，我们已经提到基于回调和异步的实现方式比较难以调整。响应式编程也是一样，这也是它与传统编程方式之间一个很大的差异点。</p>
<p data-nodeid="4544">为此，Reactor 框架的设计者也考虑到了普通开发人员的诉求，并开发了专门用于 debug 的操作符。要想启动调试模式，我们需要在程序开始的地方添加如下代码。</p>
<pre class="lang-java" data-nodeid="4545"><code data-language="java">Hooks.onOperator(providedHook -&gt; 
	providedHook.operatorStacktrace())
</code></pre>
<p data-nodeid="4546">现在，所有的操作符在执行时都会保存与执行过程相关的附加信息。而当系统出现异常时，这些附加信息就相当于系统异常堆栈信息的一部分，方便开发人员进行问题的分析和排查。</p>
<p data-nodeid="4547">上述做法是全局性的，如果你只想观察某个特定的流，那么就可以使用检查点（checkpoint）这一调试功能。例如以下代码演示了如何通过检查点来捕获 0 被用作除数的场景，我们在代码中添加了一个名为“debug”的检查点。</p>
<pre class="lang-java" data-nodeid="4548"><code data-language="java">Mono.just(<span class="hljs-number">0</span>).map(x -&gt; <span class="hljs-number">1</span> / x)
	.checkpoint(<span class="hljs-string">"debug"</span>).subscribe(System.out::println);
</code></pre>
<p data-nodeid="4549">以上代码的执行结果如下所示。</p>
<pre class="lang-java" data-nodeid="4550"><code data-language="java">Exception in thread <span class="hljs-string">"main"</span> reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zero
	Caused by: java.lang.ArithmeticException: / by zero
	…
&nbsp;
Assembly trace from producer [reactor.core.publisher.MonoMap] :
&nbsp;&nbsp;&nbsp; reactor.core.publisher.Mono.map(Mono.java:<span class="hljs-number">2029</span>)
&nbsp;&nbsp;&nbsp; com.jianxiang.reactor.demo.Debug.main(Debug.java:<span class="hljs-number">10</span>)
<span class="hljs-function">Error has been observed by the following <span class="hljs-title">operator</span><span class="hljs-params">(s)</span>:
&nbsp;&nbsp;&nbsp; |_&nbsp; Mono.<span class="hljs-title">map</span><span class="hljs-params">(Debug.java:<span class="hljs-number">10</span>)</span>
&nbsp;&nbsp;&nbsp; |_&nbsp; Mono.<span class="hljs-title">checkpoint</span><span class="hljs-params">(Debug.java:<span class="hljs-number">10</span>)</span>
&nbsp;
&nbsp;&nbsp;&nbsp; Suppressed: reactor.core.publisher.FluxOnAssembly$AssemblySnapshotException: zero
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; at reactor.core.publisher.MonoOnAssembly.&lt;init&gt;<span class="hljs-params">(MonoOnAssembly.java:<span class="hljs-number">55</span>)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; at reactor.core.publisher.Mono.<span class="hljs-title">checkpoint</span><span class="hljs-params">(Mono.java:<span class="hljs-number">1304</span>)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ... 1 more
</span></code></pre>
<p data-nodeid="4551">可以看到，这个检查点信息会包含在异常堆栈中。根据需要在系统的关键位置上添加自定义的检查点，也是我们日常开发过程中的一种最佳实践。</p>
<h3 data-nodeid="4552">小结与预告</h3>
<p data-nodeid="4553">好了，这一讲内容就介绍到这。承接上一讲的 Reactor 框架所提供的操作符，这一讲我分别就条件操作符、裁剪操作符以及各种工具操作符展开了详细的说明。在日常开发过程中，这些操作符都比较常见，能够加速我们开发响应式系统的开发过程。</p>
<p data-nodeid="4554">这里依然给你留一道思考题：在 Reactor 中，如何自己实现一个 Subscriber？</p>
<p data-nodeid="4555">那么介绍完 Spring 内置的 Reactor 框架之后，从下一讲开始，我们要讨论在 Spring 中使用这一框架来实现响应式组件的具体过程，首先要说的就是全新的 WebFlux 组件。下一讲，我们将详细分析 WebFlux 与传统 WebMVC 之间的区别，希望会带给你新的思路，我们到时见。</p>
<blockquote data-nodeid="4556">
<p data-nodeid="4557" class="te-preview-highlight">点击链接，获取课程相关代码↓↓↓<br>
<a href="https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git?fileGuid=oD5pMrGWYzgDig8d" data-nodeid="4644">https://github.com/lagoueduCol/ReactiveProgramming-jianxiang.git</a></p>
</blockquote>

---

### 精选评论

##### **风：
> 博主，能否帮助大家理解一下它的实现，因为如果只是说这些个api的使用的话，看看文档也就会了，个人觉得，reactiveStream难的是理解，不是这些api的使用。从一个简单的数据流开始，订阅，推，拉，背压，什么情况下切换了线程。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这门课还是比较面向入门的，通过案例来理解框架的应用方式，至于框架的实现原理规划上不会讲得太多

##### **敏：
> 这些调试手段，生产环境发布时，是不是就要去掉才好

##### **明：
> 老师的react是哪个版本的？我的只有Hooks.onOperatorDebug()方法，没有Hooks.onOperator，而且加了checkpoint后异常输出也没有具体发生在哪个操作符，我的是3.4.4版本的。reactor.core.Exceptions$ErrorCallbackNotImplemented: java.lang.ArithmeticException: / by zeroCaused by: java.lang.ArithmeticException: / by zero	at react.FluxDemo3.lambda$main$7(FluxDemo3.java:93)	Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException:Error has been observed at the following site(s):	|_ checkpoint ⇢ debugStack trace:		at react.FluxDemo3.lambda$main$7(FluxDemo3.java:93)		at reactor.core.publisher.FluxMapFuseable$MapFuseableSubscriber.onNext(FluxMapFuseable.java:113)

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我用的Spring Boot是2.1.13.RELEASE，对应的Reactor版本好像是3.3.X版本的

