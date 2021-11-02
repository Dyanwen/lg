<p data-nodeid="15453" class="">这一讲我将带领你学习 SkyWalking 的线程监控设计。SkyWalking 使用字节码增强技术实现了监控，通常的场景下在应用服务的启动命令中，增加探针属性就能完成接入 SkyWalking 的 APM 监控。</p>


<p data-nodeid="14561">这样简单的部署方案最大化地提升了接入效率，但也让开发人员越来越忽略对 SkyWalking 的学习。久而久之，当接入 SkyWalking 出现问题时，大家解决问题的能力就越来越低了。我处理的最多的咨询问题就是：多线程怎么串联链路，日志里面的全链路 ID 怎么打印不出来？</p>
<p data-nodeid="14562">这些问题的本质原因便是<strong data-nodeid="14656">没有掌握好对线程的监控</strong>。所以，今天我会讲述 SkyWalking 对 Java 微服务的线程监控模型，带你从监控视角回顾线程模型，探秘 SkyWalking 实现微服务监控的原理。</p>
<p data-nodeid="14563">很多有关 Java 微服务监控的分享，都会提到“线程监控模型”这个词。听起来高大上，但是通过本节课深入浅出的学习后，你会发现 SkyWalking 的线程监控方案非常清晰易懂。相信通过本课程的学习，你一定会一通百通，对各大 APM 是如何实现 Java 微服务监控有清晰深刻的理解。</p>
<h3 data-nodeid="17826" class="">工作线程、任务线程、线程池是如何配合的？</h3>




<p data-nodeid="14565">你在学习本节课程以及本专栏其他课程时，会发现我经常用了两个关键词来描述线程，那就是<strong data-nodeid="14671">工作线程</strong>和<strong data-nodeid="14672">任务线程</strong>。</p>
<p data-nodeid="14566">这两个词来自 Java 并发框架包（java.util.concurrent）中的线程池执行器类（ThreadPoolExecutor），其中当前正在执行 runWorker 方法的线程是工作线程；在 runWorker 不停循环，从等待执行队列中获取的对象为任务线程。</p>
<blockquote data-nodeid="14567">
<p data-nodeid="14568">这些译名你不必纠结，方便记忆就行。</p>
</blockquote>
<p data-nodeid="14569">接下来，我们看下工作线程和任务线程是如何配合，并在线程池中完成工作的。</p>
<p data-nodeid="14570">下文的场景完全是基于线程池使用线程的场景，这是因为在应用服务的进程中，所有的线程都必须有管理策略，否则线程数量就会与请求量挂钩。当流量高峰时，进程会被瞬间拖垮。</p>
<p data-nodeid="14571"><strong data-nodeid="14681">最好的线程管理策略就是使用线程池</strong>，线程池执行过程如下所示。</p>
<p data-nodeid="18414" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3B/17/CioPOWCCfBaAax6SAAKau0ImkMw950.png" alt="Drawing 0.png" data-nodeid="18417"></p>

<p data-nodeid="14573">整个流程非常简单地分为两部分：</p>
<ul data-nodeid="14574">
<li data-nodeid="14575">
<p data-nodeid="14576">在主线程中因为有异步处理功能，所以创建任务线程并提交到阻塞队列；</p>
</li>
<li data-nodeid="14577">
<p data-nodeid="14578">工作线程会持续从队列中拿取任务线程并执行。</p>
</li>
</ul>
<p data-nodeid="14579">上图中的<strong data-nodeid="14705">主线程</strong>，是其他<strong data-nodeid="14706">线程池</strong>中<strong data-nodeid="14707">工作线程</strong>在执行的<strong data-nodeid="14708">任务线程</strong>，你可以理解为这是个“套娃”的思想。以上这个过程非常重要，它将贯穿这一课时，需要你深刻理解。</p>
<blockquote data-nodeid="14580">
<p data-nodeid="14581">其中线程池如同蓄水池一样，起到了节约资源、提升效率的作用，这种“池化思想”你可以到《Java 性能优化实战》专栏中的<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=356&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=4186&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14712">《09 | 案例分析：池化对象的应用场景》</a>中去学习了解。</p>
</blockquote>
<h3 data-nodeid="14582">从监控视角重新认识线程</h3>
<p data-nodeid="14583">如果你还没理解也没关系，接下来我们通过案例再去深刻理解它。以 Spring Boot 微服务为例，来看下 SkyWalking 是如何实现 Web 服务器的监控的。</p>
<h4 data-nodeid="14584">1.工作线程</h4>
<p data-nodeid="14585">在 Spring Boot 微服务的场景中，主线程是在 Tomcat 容器的线程池中正在响应用户请求时执行的业务代码的任务线程。</p>
<p data-nodeid="14586">SkyWalking 在 Tomcat 进行转发请求的代码块进行监控。在同步模式下，整体的监控类同于 Spring AOP 面向切面的设计思想，在执行方法的前后进行监控数据的收集，也就是对任务线程的执行过程进行监控。</p>
<h4 data-nodeid="14587">2.任务线程</h4>
<p data-nodeid="14588">为了实现在执行任务线程的过程中，每处收集的监控信息可以不被开发人员感知即可实现关联，SkyWalking 使用了 Java 线程提供的线程本地变量，也就是 ThreadLocal 能力来实现。</p>
<p data-nodeid="14589">在执行任务线程开始之初，当栈针触及第一个 SkyWalking 监控埋点后，会在 ThreadLocal 创建当前任务线程的跟踪信息，重要的属性如下。</p>
<ul data-nodeid="14590">
<li data-nodeid="14591">
<p data-nodeid="14592"><strong data-nodeid="14735">任务线程类型</strong><br>
根据第一个监控埋点的类型，将任务线程分为是要追踪<strong data-nodeid="14736">跨进程链路</strong>还是**跨线程链路。**对于跨进程链路，又可以分为上游是人还是机器。以 SpringMvc 追踪为例，第一个监控埋点需要识别 HTTP Header 中的标记信息。通过标记属性的有无，将上游识别为机器或是人的调用。</p>
</li>
</ul>
<blockquote data-nodeid="14593">
<p data-nodeid="14594">至于跨线程链路，我会在《14 | 互通有无：如何设计跨语言的 APM 交互协议？》中讲解。</p>
</blockquote>
<ul data-nodeid="14595">
<li data-nodeid="14596">
<p data-nodeid="14597"><strong data-nodeid="14743">全链路跟踪标识</strong><br>
若 HTTP Header 没有链路标识信息，就将追踪的链路渲染为人为调用，并创建全链路标识绑定到 ThreadLocal 信息中；反之，识别 HTTP Header 的链路标识，解析出全链路跟踪标识，并追加到 ThreadLocal 中的全链路标识信息。</p>
</li>
</ul>
<p data-nodeid="14598">当任务线程开始执行后，SkyWalking 监控埋点会有以下两类：跨进程监控埋点和跨线程监控埋点。</p>
<ul data-nodeid="14599">
<li data-nodeid="14600">
<p data-nodeid="14601">对于<strong data-nodeid="14750">跨进程监控</strong>，需要找到面向传输的请求头部属性，将 ThreadLocal 中的全链路标识放入其中。</p>
</li>
<li data-nodeid="14602">
<p data-nodeid="14603">对于<strong data-nodeid="14756">跨线程监控</strong>，那就很简单了，在主线程创建任务线程时，扩展任务线程对象的属性。将主线程的 ThreadLocal 中的链路标识放到扩展属性中，在任务线程执行时，识别这个属性完成串联。</p>
</li>
</ul>
<p data-nodeid="14604">最后，就是任务线程执行接近结束时，监控任务线程的结束，依赖于首个监控方法的退出时机，此时需要清理 SkyWalking 所有监控的 ThreadLocal 信息。这个环节最关键，也最容易出问题。通过上文我们知道，SkyWalking 的无侵入监控线程模型使用 ThreadLocal，这些存在于 ThreadLocal 中的属性开发人员没有感知，也无法控制。</p>
<p data-nodeid="14605"><strong data-nodeid="14761">所以在线程复用的时候，若在任务线程结束的时候没有清理干净，那接下来执行的任务线程：轻则内存泄漏，无法串联链路；重则内存溢出，整个进程被不停的 Long GC 夯住。</strong></p>
<p data-nodeid="14606">和你讲了那么多，也不如一个典型的问题案例更能让你体会到：<strong data-nodeid="14766">线程追踪模型设计的重要性，以及错误配置会带来的严重后果。</strong></p>
<h3 data-nodeid="14607">最常见、最灾难的错误用法</h3>
<p data-nodeid="14608">案例的起因是这样的：公司内部有个赔付系统，在核心赔付用户操作完成后，会异步做一些功能类似发短信通知的事情，赔付系统的业务方很想将这些核心操作的主线程与其引起的异步线程串联起来。</p>
<p data-nodeid="14609">通过之前的学习，我们知道 SkyWalking 使用与 Dapper 相同的标记方案实现了分布式链路追踪。在实现跨线程跟踪时，我们需要将<strong data-nodeid="14774">主线程的链路标记传递给任务线程</strong>。实现传递的方式多种，在 SkyWalking 涉及跨线程组件中，最常见的方式在主线程创建任务线程时，将链路的标记信息传递给任务线程。在任务线程执行时，识别并绑定父线程传递过来链路标记信息，开始追踪任务线程的执行过程。</p>
<p data-nodeid="14610">理论很清晰，但落地时还是很容易出问题。以上面我们说的赔付异步短信通知的场景为例，一线开发的代码这样的:</p>
<pre class="lang-java" data-nodeid="14611"><code data-language="java">ThreadPoolTaskExecutor executor = <span class="hljs-keyword">new</span> ThreadPoolTaskExecutor();
executor.setThreadFactory(r -&gt; <span class="hljs-keyword">new</span> Thread(r, <span class="hljs-string">"rename"</span>));
executor.initialize();
executor.execute(<span class="hljs-keyword">new</span> Runnable() {
	<span class="hljs-meta">@Override</span> <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">()</span> </span>{
		<span class="hljs-comment">//短信通知</span>
	}
});
</code></pre>
<p data-nodeid="14612">创建一个线程池，修改线程名称方便定位问题，然后将通知的代码块放到线程池内执行，就达到了像短信通知这样非重要的功能不影响赔付用户的主线功能的目的。<br>
但这样的代码，因为没有将主线程的链路标记传递给子线程，所以会出现<strong data-nodeid="14783">断链</strong>，无法完成父子线程的串联。SkyWalking 提供了子线程包装类 RunnableWrapper。通过子线程包装类，我们可以轻易实现跨线程串联。</p>
<p data-nodeid="14613">那问题来了：上面代码中，哪个线程是任务线程呢？如果使用错误会怎么样呢？</p>
<h4 data-nodeid="14614">1.如何找寻任务线程？</h4>
<p data-nodeid="14615">对于第一个问题：哪个线程是任务线程呢？这个问题不难。根据我们上文对线程池的认识，我们都知道线程池提交的线程就是任务线程。所以需要在提交任务线程时，通过子线程包装类将短信通知的线程包装起来即可，完成跨进程链路串联。</p>
<h4 data-nodeid="14616">2.找错任务线程会怎样？</h4>
<p data-nodeid="14617">对于第二个问题：如果使用错误会怎么样呢？也就是如果用线程包装类的包装方法，包装了工作线程会怎么样呢？</p>
<p data-nodeid="14618">我们再一次回顾一下这张图，并再次理解工作线程和任务线程的区别。工作线程有两种状态：执行任务线程与等待任务线程。这两个状态循环交替着：当执行任务线程时，工作线程承载着任务线程执行。可以说，这时两种线程可以看作一个整体。</p>
<p data-nodeid="19004" class=""><img src="https://s0.lgstatic.com/i/image6/M00/3B/0F/Cgp9HWCCfCmAQ6J-AAKau0ImkMw629.png" alt="Drawing 1.png" data-nodeid="19007"></p>

<p data-nodeid="14620">但是，如果把监控放在工作线程就会出现严重问题。因为线程的<strong data-nodeid="14802">监控</strong>是监控<strong data-nodeid="14803">任务线程执行的生命周期</strong>，而工作线程的生命周期是常驻于进程的。随着线程池策略启动，工作线程就进入了无限等待任务线程和执行任务线程的状态，最后会在应用服务优雅关闭时才停止。</p>
<p data-nodeid="14621">在 SkyWalking 的监控线程模型下，若监控了工作线程，就会出现以下情况：</p>
<ul data-nodeid="14622">
<li data-nodeid="14623">
<p data-nodeid="14624">工作线程在启动后，就会开启监控，在执行任务线程时会与主线程进行关联；</p>
</li>
<li data-nodeid="14625">
<p data-nodeid="14626">在结束任务线程时，由于工作线程并没有退出，所以监控信息不会上报；</p>
</li>
<li data-nodeid="14627">
<p data-nodeid="14628">当复用工作线程，再次执行任务线程时，会继续与新的（创建本次任务线程的）主线程进行关联。</p>
</li>
</ul>
<p data-nodeid="14629">那最终的结果就是：每一个工作线程内部的监控信息由于一直无法上报而无法得到释放，占用的内存空间会直线上升。且这些被占用的内存由于不能匹配内存垃圾回收算法，JVM 进程在运行一段时间到达临界值后，会频繁执行垃圾回收，但通过垃圾回收日志会发现收效甚微。</p>
<p data-nodeid="14630"><strong data-nodeid="14812">当然这是最终分析问题的结论，那我是怎么在测试环境少流量这一情况下，发现这个问题的呢？</strong></p>
<p data-nodeid="14631">首先是赔付业务方在开发完串联跨线程功能后，发现之前有的“断链”的子线程监控在发布后，SkyWalking 展示端不能显示这些监控信息了。</p>
<blockquote data-nodeid="14632">
<p data-nodeid="14633">SkyWalking 的展示端根据存储中的链路数据所见即所得。如果展示端没有，那问题肯定就是没有进行链路数据上报。</p>
</blockquote>
<p data-nodeid="14634">那我们第一个做法就是<strong data-nodeid="14820">回滚对比</strong>，回滚后“断链”的子线程监控信息又可以收集了。而且展示端还展示了一个结束时间与回滚时间一致，有着多个发送短信监控据合在一起的监控代码段，且与多个赔付主线程发送了关联。</p>
<p data-nodeid="14635"><strong data-nodeid="14824">那问题就很明确了，主要解决这两个问题：为什么会出现任务线程监控数据打包的问题？以及为什么上传时间与回滚时间有关系？</strong></p>
<p data-nodeid="14636">通过发送短信代码块 Debug，我们发现第一次请求发生时，监控数据的开始时间并非与请求时间一致；在第二次请求时，发现线程中包含第一次请求的链路信息。</p>
<p data-nodeid="14637">再次整理思路，第一次请求监控数据的开始时间过早，证明监控的线程模型的起始点并非任务线程的起始，很可能是工作线程的起始；第二次请求的监控数据中数据包含第一次请求的数据，证明很可能发生了内存泄漏，任务线程退出时没有清除监控信息。</p>
<p data-nodeid="14638">最后综上所述，监控的线程模型与 SkyWalking 设计有出入，就是任务线程没有被监控，而工作线程被监控了。通过 Debug 调整栈针位置，很快证明了这一点，并得出与上面相同的结论。</p>
<blockquote data-nodeid="14639">
<p data-nodeid="14640">案例现场和解决过程我已经写到了<a href="https://github.com/apache/skywalking/blob/master/docs/en/FAQ/Memory-leak-enhance-Worker-thread.md?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14831">SkyWalking 官方的 FAQ 文档</a>，如果你需要可以查看详情。</p>
</blockquote>
<p data-nodeid="14641">通过上面的问题，我们得知：开发人员还是很需要对 SkyWalking 线程监控模型有更高维度的认知的，不然很容易造成者内存泄漏或内存溢出等严重问题。</p>
<h3 data-nodeid="14642">小结与思考</h3>
<p data-nodeid="14643">本节课，我带你从监控视角温习了线程池。线程池有两个很重要的角色：通过管理策略创建的<strong data-nodeid="14844">工作线程</strong>，和其他主线程放到阻塞队列的<strong data-nodeid="14845">任务线程</strong>。如下图所示，我们第三次再回顾一下这张图。</p>
<p data-nodeid="19594" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/3B/0F/Cgp9HWCCfDaAEbuYAAKau0ImkMw690.png" alt="Drawing 2.png" data-nodeid="19597"></p>

<p data-nodeid="14645">因为任务线程才是真正执行业务代码的载体，所以 SkyWalking 的<strong data-nodeid="14854">线程监控模型就是监控任务线程的生命周期</strong>。</p>
<p data-nodeid="14646">之后我们讲述了 1.任务线程生命周期中的开始监控  2.任务线程执行中的监控  3.结束监控。以及在这三个重要监控环节中，SkyWalking 是如何实现无侵入监控的。最后我们又结合了一个线程池监控实例，认识到线程监控模型并正确配置监控的重要性。</p>
<p data-nodeid="14647">那么你在工作中使用 SkyWalking 时，遇到过内存泄漏问题吗？你又是怎么解决的？欢迎你在评论区分享你的案例故事，期待与你讨论。</p>

---

### 精选评论

##### **峰：
> SkyWalking对响应编程怎么解决断连问题的呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 监控执行业务代码块的线程生命周期即可

