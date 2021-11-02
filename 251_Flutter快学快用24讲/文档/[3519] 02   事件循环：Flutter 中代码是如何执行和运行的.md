<p data-nodeid="10386" class="">上节课介绍了 Dart 基础数据类型、基础运算符、类以及库与调用。本课时着重通过实践带你掌握 Dart 的运行原理。</p>
<h3 data-nodeid="10387">Dart 单线程</h3>
<p data-nodeid="10388">单线程在流畅性方面有一定安全保障，这点在 JavaScript 中存在类似的机制原理，其核心是分为主线程、微任务和宏任务。主线程执行主业务逻辑，网络 I/O 、本地文件 I/O 、异步事件等相关任务事件，应用事件驱动方式来执行。在 Dart 中同样是单线程执行，其次也包含了两个事件队列，一个是微任务事件队列，一个是事件队列。</p>
<ul data-nodeid="10389">
<li data-nodeid="10390">
<p data-nodeid="10391">微任务队列</p>
</li>
</ul>
<p data-nodeid="10392">微任务队列包含有 Dart 内部的微任务，主要是通过 scheduleMicrotask 来调度。</p>
<ul data-nodeid="10393">
<li data-nodeid="10394">
<p data-nodeid="10395">事件队列</p>
</li>
</ul>
<p data-nodeid="10396">事件队列包含外部事件，例如 I/O 、 Timer ，绘制事件等等。</p>
<h4 data-nodeid="10397">事件循环</h4>
<p data-nodeid="10398">既然 Dart 包含了微任务和事件任务，那么这两个任务之间是如何进行循环执行的呢？我们可以先看下 Dart 执行的逻辑过程（如图 1）：</p>
<ol data-nodeid="10399">
<li data-nodeid="10400">
<p data-nodeid="10401">首先是执行 main 函数，并生产两个相应的微任务和事件任务队列；</p>
</li>
<li data-nodeid="10402">
<p data-nodeid="10403">判断是否存在微任务，有则执行，执行完成后再继续判断是否还存在微任务，无则判断是否存在事件任务；</p>
</li>
<li data-nodeid="10404">
<p data-nodeid="10405">如果没有可执行的微任务，则判断是否存在事件任务，有则执行，无则继续返回判断是否还存在微任务；</p>
</li>
<li data-nodeid="10406">
<p data-nodeid="10407">在微任务和事件任务执行过程中，同样会产生微任务和事件任务，因此需要再次判断是否需要插入微任务队列和事件任务队列。</p>
</li>
</ol>
<p data-nodeid="10408"><img src="https://s0.lgstatic.com/i/image/M00/1D/7D/Ciqc1F7h_D2ARi2aAAJ2G36y8Ng725.png" alt="Drawing 0.png" data-nodeid="10566"></p>
<p data-nodeid="10409">图 1 Dart 事件循环机制</p>
<p data-nodeid="10410">为了验证上面的运行原理，我实现了下面的示例代码，首先 import async 库，然后在 main 函数中首先打印 flow start ，接下来执行一个微任务事件，再执行一个事件任务，最后再打印 flow end 。</p>
<pre class="lang-dart" data-nodeid="10411"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:async'</span>;
<span class="hljs-keyword">void</span> main() {
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow start'</span>); <span class="hljs-comment">// 执行打印开始&nbsp;</span>
	<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-built_in">print</span>(<span class="hljs-string">'event'</span>); <span class="hljs-comment">// 执行事件任务，打印标记</span>
&nbsp; &nbsp;	});
&nbsp; &nbsp;	<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列&nbsp;</span>
	scheduleMicrotask((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'microtask'</span>); <span class="hljs-comment">// 执行微任务，打印标记</span>
&nbsp; &nbsp; });
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow end'</span>); <span class="hljs-comment">// 打印结束标记</span>
}
</code></pre>
<p data-nodeid="10412">使用 Dart 运行如上命令。</p>
<pre class="lang-shell" data-nodeid="10413"><code data-language="shell">dart flow.dart
</code></pre>
<p data-nodeid="10414">代码的实际运行过程如下：</p>
<ul data-nodeid="10415">
<li data-nodeid="10416">
<p data-nodeid="10417">首先主线程逻辑，执行打印 start ；</p>
</li>
<li data-nodeid="10418">
<p data-nodeid="10419">执行 Timer，为事件任务，将其增加到事件任务队列中；</p>
</li>
<li data-nodeid="10420">
<p data-nodeid="10421">执行 scheduleMicrotask，为微任务队列，将其增加到微任务队列中；</p>
</li>
<li data-nodeid="10422">
<p data-nodeid="10423">执行打印 flow end；</p>
</li>
<li data-nodeid="10424">
<p data-nodeid="10425">判断是否存在微任务队列，存在则执行微任务队列，打印 mcrotask；</p>
</li>
<li data-nodeid="10426">
<p data-nodeid="10427">判断是否还存在微任务队列，无则判断是否存在事件任务队列，存在执行事件任务队列，打印 event。</p>
</li>
</ul>
<pre class="lang-sql" data-nodeid="10428"><code data-language="sql">flow <span class="hljs-keyword">start</span>
flow <span class="hljs-keyword">end</span>
microtask
<span class="hljs-keyword">event</span>
</code></pre>
<p data-nodeid="10429">为了更清晰描述，可以我们使用图 2 动画来演示。</p>
<p data-nodeid="10430"><img src="https://s0.lgstatic.com/i/image/M00/1D/8A/CgqCHl7h_RmAFXa9AAnXFc-CvdQ552.gif" alt="flutter-flow-new.gif" data-nodeid="10580"></p>
<p data-nodeid="10431">图 2 Dart 主线程运行逻辑</p>
<p data-nodeid="10432">介绍完 Dart 的运行原理，你可能会产生以下疑问。</p>
<p data-nodeid="10433"><strong data-nodeid="10586">疑问1，为什么事件任务都执行完成了，还需要继续再循环判断是否有微任务？</strong></p>
<p data-nodeid="10434">核心解释是：微任务在执行过程中，也会产生新的事件任务，事件任务在执行过程中也会产生新的微任务。产生的新微任务，按照执行流程，需要根据队列方式插入到任务队列最后。</p>
<p data-nodeid="10435">我们通过代码来看下该过程。下面一段代码， import async 库，第一步打印 start ， 然后执行一个事件任务，在事件任务中打印 event 。接下来增加了一个微任务事件，在微任务事件中打印 microtask in event 。第二步执行微任务事件，在微任务事件中打印 microtask ，并且在其中增加事件任务队列，事件任务队列中打印 event in microtask ，最后再打印 flow end 。</p>
<pre class="lang-dart" data-nodeid="10436"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:async'</span>;
<span class="hljs-keyword">void</span> main() {
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow start'</span>); <span class="hljs-comment">// 执行打印开始</span>
&nbsp;   <span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp;	<span class="hljs-built_in">print</span>(<span class="hljs-string">'event'</span>); <span class="hljs-comment">// 执行事件任务，打印事件任务标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 执行判断为微任务，添加到微任务队列&nbsp;</span>
&nbsp; &nbsp; &nbsp; &nbsp;	scheduleMicrotask((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'microtask in event'</span>); <span class="hljs-comment">// 执行微任务，打印微任务标记</span>
&nbsp; &nbsp; 	});
&nbsp; &nbsp;	});
&nbsp; <span class="hljs-comment">// 执行判断为微任务，添加到微任务队列&nbsp;</span>
	scheduleMicrotask((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'microtask'</span>); <span class="hljs-comment">// 执行微任务，打印微任务执行标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列&nbsp;</span>
&nbsp; &nbsp; &nbsp; &nbsp; Timer.run((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'event in microtask'</span>); <span class="hljs-comment">// 执行事件任务，打印事件任务标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; });
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow end'</span>); <span class="hljs-comment">// 打印结束标记</span>
}
</code></pre>
<p data-nodeid="10437">使用 Dart 运行如上命令。</p>
<pre class="lang-java" data-nodeid="10438"><code data-language="java">dart event_with_microtask.dart
</code></pre>
<p data-nodeid="10439">代码的实际运行过程如下：</p>
<ul data-nodeid="10440">
<li data-nodeid="10441">
<p data-nodeid="10442">首先还是依次执行打印 flow start ；</p>
</li>
<li data-nodeid="10443">
<p data-nodeid="10444">执行 Timer 为事件任务，添加事件任务队列中；</p>
</li>
<li data-nodeid="10445">
<p data-nodeid="10446">执行 scheduleMicrotask 为微任务，添加到微任务队列中；</p>
</li>
<li data-nodeid="10447">
<p data-nodeid="10448">打印 end ；</p>
</li>
<li data-nodeid="10449">
<p data-nodeid="10450">执行微任务队列，打印 microtask ，其中包括了事件任务，将事件任务插入到事件任务中；</p>
</li>
<li data-nodeid="10451">
<p data-nodeid="10452">执行事件任务队列，打印 event ，其中包括了微任务，将微任务插入到微任务队列中；</p>
</li>
<li data-nodeid="10453">
<p data-nodeid="10454">微任务队列存在微任务，执行微任务队列，打印 microtask in event；</p>
</li>
<li data-nodeid="10455">
<p data-nodeid="10456">微任务队列为空，存在事件任务队列，执行事件任务队列，打印 event in microtask；</p>
</li>
</ul>
<p data-nodeid="10457">根据如上的运行过程，我们可以得出以下的一个运行结果，这点可以通过运行 Dart 命令得到实际的验证。</p>
<pre class="lang-sql" data-nodeid="10458"><code data-language="sql">flow <span class="hljs-keyword">start</span>
flow <span class="hljs-keyword">end</span>
microtask
<span class="hljs-keyword">event</span>
microtask <span class="hljs-keyword">in</span> <span class="hljs-keyword">event</span>
<span class="hljs-keyword">event</span> <span class="hljs-keyword">in</span> microtask
</code></pre>
<p data-nodeid="12184">为了更形象来描述，我使用图 3 动画来演示。</p>
<p data-nodeid="12185" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/21/33/Ciqc1F7p3BCAAutpABeCx2dZvOo916.gif" alt="image" data-nodeid="12189"></p>


<p data-nodeid="10461">图 3 多微任务和事件任务执行流程</p>
<p data-nodeid="10462">一句话概括上面的实践运行结果：每次运行完一个事件后，都会判断微任务和事件任务，在两者都存在时，优先执行完微任务，只有微任务队列没有其他的任务了才会执行事件任务。</p>
<p data-nodeid="10463"><strong data-nodeid="10609">疑问2，Dart 运行过程中是否会被事件运行卡住？</strong></p>
<p data-nodeid="10464">答案是会，比如在运行某个微任务，该微任务非常的耗时，会导致其他微任务和事件任务卡住，从而影响到一些实际运行，这里我们可以看如下例子：</p>
<pre class="lang-dart" data-nodeid="10465"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:async'</span>;
<span class="hljs-keyword">void</span> main() {
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow start'</span>);&nbsp; <span class="hljs-comment">// 执行打印开始</span>
&nbsp; <span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">for</span>(<span class="hljs-built_in">int</span> i=<span class="hljs-number">0</span>; i&lt;<span class="hljs-number">1000000000</span>; i++){ <span class="hljs-comment">// 大循环，为了卡住事件任务执行时间，检查是否会卡住其他任务执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span>(i == <span class="hljs-number">1000000</span>){
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; scheduleMicrotask((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'microtask in event'</span>); <span class="hljs-comment">// 执行微任务，打印微任务标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'event'</span>); <span class="hljs-comment">// 执行完事件任务，打印执行完事件任务标记</span>
&nbsp; &nbsp;	});
&nbsp; <span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
	scheduleMicrotask((){&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'microtask'</span>); <span class="hljs-comment">// 执行微任务，打印微任务标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
&nbsp; &nbsp; &nbsp; &nbsp; Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'event in microtask'</span>); <span class="hljs-comment">// 执行事件任务，打印事件任务标记</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; });
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'flow end'</span>); <span class="hljs-comment">// 打印结束标记</span>
}
</code></pre>
<p data-nodeid="10466">上面这段代码和之前的唯一不同点是在执行第一个事件任务的时候，使用了一个大的 for 循环，从运行结果会看到 event in microtask 和 microtask in event 打印的时间会被 event 的执行所 block 住。从结果分析来看 Dart 中事件运行是会被卡住的，因此在日常编程的时候要特别注意，避免因为某个事件任务密集计算，导致较差的用户操作体验。</p>
<h3 data-nodeid="10467">Isolate 多线程</h3>
<p data-nodeid="10468">上面我们介绍了 Dart 是单线程的，这里说的 Dart 的单线程，其实和操作系统的线程概念是存在一定区别的， Dart 的单线程叫作 isolate 线程，<strong data-nodeid="10617">每个 isolate 线程之间是不共享内存的，通过消息机制通信。</strong></p>
<p data-nodeid="10469">我们看个例子，例子是利用 Dart 的 isolate 实现多线程的方式。</p>
<pre class="lang-dart" data-nodeid="10470"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:async'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'dart:isolate'</span>;
Isolate isolate;
<span class="hljs-built_in">String</span> name = <span class="hljs-string">'dart'</span>;
<span class="hljs-keyword">void</span> main() {
	<span class="hljs-comment">// 执行新线程创建函数</span>
&nbsp;	isolateServer();
}
<span class="hljs-comment">/// <span class="markdown">多线程函数</span></span>
<span class="hljs-keyword">void</span> isolateServer()<span class="hljs-keyword">async</span>{
	<span class="hljs-comment">// 创建新的线程，并且执行回调 changName&nbsp;</span>
	<span class="hljs-keyword">final</span> receive = ReceivePort();
	isolate = <span class="hljs-keyword">await</span> Isolate.spawn(changName, receive.sendPort);
	<span class="hljs-comment">// 监听线程返回信息&nbsp;</span>
	receive.listen((data){
		<span class="hljs-built_in">print</span>(<span class="hljs-string">"Myname is <span class="hljs-subst">$data</span>"</span>); <span class="hljs-comment">// 打印线程返回的数据</span>
		<span class="hljs-built_in">print</span>(<span class="hljs-string">"Myname is <span class="hljs-subst">$name</span>"</span>); <span class="hljs-comment">// 打印全局 name 的数据</span>
	});
}
<span class="hljs-comment">/// <span class="markdown">线程回调处理函数</span></span>
<span class="hljs-keyword">void</span> changName(SendPort port){
	name = <span class="hljs-string">'dart isloate'</span>; <span class="hljs-comment">// 修改当前全局 name 属性</span>
	port.send(name); <span class="hljs-comment">// 将当前name发送给监听方</span>
	<span class="hljs-built_in">print</span>(<span class="hljs-string">"Myname is <span class="hljs-subst">$name</span> in isloate"</span>); <span class="hljs-comment">// 打印当前线程中的 name</span>
}
</code></pre>
<p data-nodeid="10471">使用 Dart 运行如上命令。</p>
<pre class="lang-dart" data-nodeid="10472"><code data-language="dart">dart isolate.dart
</code></pre>
<p data-nodeid="10473">以上代码的执行运行流程如下：</p>
<ul data-nodeid="10474">
<li data-nodeid="10475">
<p data-nodeid="10476">import 对应的库；</p>
</li>
<li data-nodeid="10477">
<p data-nodeid="10478">声明两个变量，一个是 isolate 对象，一个是字符串类型的 name；</p>
</li>
<li data-nodeid="10479">
<p data-nodeid="10480">执行 main 函数，main 函数中执行 isolateServer 异步函数；</p>
</li>
<li data-nodeid="10481">
<p data-nodeid="10482">isolateServer 中创建了一个 isolate 线程，创建线程时候，可以传递接受回调的函数 changName；</p>
</li>
<li data-nodeid="10483">
<p data-nodeid="10484">在 changName 中修改当前的全局变量 name ，并且发送消息给到接收的端口，并且打印该线程中的 name 属性；</p>
</li>
<li data-nodeid="10485">
<p data-nodeid="10486">isolateServer 接收消息，接收消息后打印返回的数据和当前 name 变量。</p>
</li>
</ul>
<p data-nodeid="10487">根据如上执行过程，可以得出如下的运行结果。</p>
<pre class="lang-dart" data-nodeid="10488"><code data-language="dart">Myname <span class="hljs-keyword">is</span> dart isolate <span class="hljs-keyword">in</span> isolate
Myname <span class="hljs-keyword">is</span> dart isolate
Myname <span class="hljs-keyword">is</span> dart
</code></pre>
<p data-nodeid="10489">从运行结果中，可以看到新的线程修改了全局的 name，并且通过消息发送返回到主线程中。而主线程的 name 属性并没有因为创建的新线程中的 name 属性的修改而发生改变，这也印证了内存隔离这点。</p>
<h3 data-nodeid="10490">综合示例</h3>
<p data-nodeid="10491">了解完以上知识点后，我再从一个实际的例子进行综合的分析，让你进一步巩固对 Dart 运行原理的掌握。</p>
<p data-nodeid="10492">假设一个项目，需要 2 个团队去完成，团队中包含多项任务。可以分为 2 个高优先级任务（高优先级的其中，会产生 2 个任务，一个是紧急一个是不紧急），和 2 个非高优先级任务（非高优先级的其中，会产生有 2 个任务，一个是紧急一个是不紧急）。其中还有一个是必须依赖其他团队去做的，因为本团队没有那方面的资源，第三方也会产生一个高优先级任务和一个低优先级任务。</p>
<p data-nodeid="10493">根据以上假设，我们可以用表 1 任务划分来表示：</p>
<table data-nodeid="10495">
<thead data-nodeid="10496">
<tr data-nodeid="10497">
<th data-org-content="**主任务**" data-nodeid="10499"><strong data-nodeid="10636">主任务</strong></th>
<th data-org-content="**高优先级任务（微任务）**" data-nodeid="10500"><strong data-nodeid="10640">高优先级任务（微任务）</strong></th>
<th data-org-content="**低优先级任务（事件任务）**" data-nodeid="10501"><strong data-nodeid="10644">低优先级任务（事件任务）</strong></th>
<th data-org-content="**第三方任务（isolate）**" data-nodeid="10502"><strong data-nodeid="10648">第三方任务（isolate）</strong></th>
</tr>
</thead>
<tbody data-nodeid="10507">
<tr data-nodeid="10508">
<td data-org-content="H1" data-nodeid="10509">H1</td>
<td data-org-content="h1-1" data-nodeid="10510">h1-1</td>
<td data-org-content="l1-1" data-nodeid="10511">l1-1</td>
<td data-org-content="否" data-nodeid="10512">否</td>
</tr>
<tr data-nodeid="10513">
<td data-org-content="H2" data-nodeid="10514">H2</td>
<td data-org-content="h2-1" data-nodeid="10515">h2-1</td>
<td data-org-content="l2-1" data-nodeid="10516">l2-1</td>
<td data-org-content="否" data-nodeid="10517">否</td>
</tr>
<tr data-nodeid="10518">
<td data-org-content="L3" data-nodeid="10519">L3</td>
<td data-org-content="h3-1" data-nodeid="10520">h3-1</td>
<td data-org-content="l3-1" data-nodeid="10521">l3-1</td>
<td data-org-content="否" data-nodeid="10522">否</td>
</tr>
<tr data-nodeid="10523">
<td data-org-content="L4" data-nodeid="10524">L4</td>
<td data-org-content="h4-1" data-nodeid="10525">h4-1</td>
<td data-org-content="l4-1" data-nodeid="10526">l4-1</td>
<td data-org-content="否" data-nodeid="10527">否</td>
</tr>
<tr data-nodeid="10528">
<td data-org-content="C5" data-nodeid="10529">C5</td>
<td data-org-content="ch5-1" data-nodeid="10530">ch5-1</td>
<td data-org-content="cl5-1" data-nodeid="10531">cl5-1</td>
<td data-org-content="是" data-nodeid="10532">是</td>
</tr>
</tbody>
</table>
<p data-nodeid="10533">表1 项目任务划分详情</p>
<p data-nodeid="10534">然后我们按照 Dart 语言执行方式去安排这个项目的开发工作，我们看看安排的工作到底会是怎么样执行流程，代码实现方式如下。</p>
<pre class="lang-dart" data-nodeid="10535"><code data-language="dart"><span class="hljs-keyword">import</span> <span class="hljs-string">'dart:async'</span>;
<span class="hljs-keyword">import</span> <span class="hljs-string">'dart:isolate'</span>;
Isolate isolate;
<span class="hljs-keyword">void</span> main() {
	<span class="hljs-built_in">print</span>(<span class="hljs-string">'project start'</span>); <span class="hljs-comment">// 打印项目启动标记</span>
	ctask(); <span class="hljs-comment">// 分配并执行 C 任务</span>
	<span class="hljs-comment">// 大循环，等待</span>
	<span class="hljs-comment">//for(int i=0; i&lt;1000000000; i++){</span>
	<span class="hljs-comment">//}</span>
	<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
	scheduleMicrotask((){
		<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
		scheduleMicrotask((){
			<span class="hljs-built_in">print</span>(<span class="hljs-string">'h1-1 task complete'</span>); <span class="hljs-comment">// 执行微任务，并打印微任务优先级h1-1</span>
		});
		<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
		Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'l1-1 task complete'</span>); <span class="hljs-comment">// 执行事件任务，并打印事件任务优先级l1-1</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'H1 task complete'</span>); <span class="hljs-comment">// 打印H1微任务执行标记</span>
	});
	<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
	scheduleMicrotask((){
		<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
		scheduleMicrotask((){
			<span class="hljs-built_in">print</span>(<span class="hljs-string">'h2-1 task complete'</span>); <span class="hljs-comment">// 执行微任务，并打印微任务优先级h2-1</span>
		});
		<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
		Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'l2-1 task complete'</span>); <span class="hljs-comment">// 执行事件任务，并打印事件任务优先级l2-1</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'H2 task complete'</span>); <span class="hljs-comment">// 打印H2微任务执行标记</span>
	});
	<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){
		<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
		scheduleMicrotask((){
			<span class="hljs-built_in">print</span>(<span class="hljs-string">'h3-1 task complete'</span>); <span class="hljs-comment">// 执行微任务，并打印微任务优先级h3-1</span>
		});
		<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
		Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'l3-1 task complete'</span>); <span class="hljs-comment">// 执行事件任务，并打印事件任务优先级l3-1</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
		<span class="hljs-built_in">print</span>(<span class="hljs-string">'L3 task complete'</span>); <span class="hljs-comment">// 打印L3事件任务执行标记</span>
&nbsp; &nbsp; });
	
	<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){
		<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
		scheduleMicrotask((){
			<span class="hljs-built_in">print</span>(<span class="hljs-string">'h4-1 task complete'</span>); <span class="hljs-comment">// 执行微任务，并打印微任务优先级h4-1</span>
		});
		<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
		Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; 	<span class="hljs-built_in">print</span>(<span class="hljs-string">'l4-1 task complete'</span>); <span class="hljs-comment">// 执行事件任务，并打印事件任务优先级l4-1</span>
&nbsp; &nbsp; &nbsp; &nbsp; });
		<span class="hljs-built_in">print</span>(<span class="hljs-string">'L4 task complete'</span>); <span class="hljs-comment">// 打印L4事件任务执行标记</span>
&nbsp; &nbsp; });
}
<span class="hljs-comment">/// <span class="markdown">C 任务具体代码，创建新的线程，并监听线程返回数据&nbsp;</span></span>
<span class="hljs-keyword">void</span> ctask()<span class="hljs-keyword">async</span>{
	<span class="hljs-keyword">final</span> receive = ReceivePort();
	isolate = <span class="hljs-keyword">await</span> Isolate.spawn(doCtask, receive.sendPort);
	receive.listen((data){
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(data);
	});
}
<span class="hljs-comment">/// <span class="markdown">创建的新线程，具体执行的任务代码</span></span>
<span class="hljs-keyword">void</span> doCtask(SendPort port){
	<span class="hljs-comment">// 执行判断为微任务，添加到微任务队列</span>
	scheduleMicrotask((){
		<span class="hljs-built_in">print</span>(<span class="hljs-string">'ch5-1 task complete'</span>); <span class="hljs-comment">// 执行微任务，并打印微任务优先级ch5-1&nbsp;</span>
	});
	<span class="hljs-comment">// 执行判断为事件任务，添加到事件任务队列</span>
	Timer.run((){
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-built_in">print</span>(<span class="hljs-string">'cl5-1 task complete'</span>); <span class="hljs-comment">// 打印cl5-1事件任务执行标记</span>
&nbsp; &nbsp; });
	port.send(<span class="hljs-string">'C1 task complete'</span>); <span class="hljs-comment">// 打印 C 任务执行标记</span>
}
</code></pre>
<p data-nodeid="10536">使用 Dart 运行如上命令。</p>
<pre class="lang-plain" data-nodeid="10537"><code data-language="plain">dart isolate.dart
</code></pre>
<p data-nodeid="10538">我们先来看下，上面代码的运行结果。</p>
<pre class="lang-sql" data-nodeid="10539"><code data-language="sql">project <span class="hljs-keyword">start</span>
H1 task <span class="hljs-keyword">complete</span>
H2 task <span class="hljs-keyword">complete</span>
h1<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
h2<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
L3 task <span class="hljs-keyword">complete</span>
h3<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
L4 task <span class="hljs-keyword">complete</span>
h4<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l1<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l2<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l3<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l4<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
ch5<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
cl5<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
C1 task <span class="hljs-keyword">complete</span>
</code></pre>
<p data-nodeid="10540">H 和 L 的运行原理，希望你用上面我所讲到的知识点，去一步步分析，可以像我们图 2 或者图 3 的方法，画两个队列，然后逐步去分析。<br>
上面的运行结果中，非 C 任务的运行原理留给你自己去分析，这里我着重介绍下为什么 C 的任务一直在最后才完成。</p>
<p data-nodeid="10541">由于 C 任务是由其他线程执行，因此这里存在一定的时间去创建线程。创建线程完成后，才会进行回调，回调后才会将相应的回调事件插入到事件任务队列中。因此 C1 task complete 会在最后的一个事件任务中执行。而 ch5-1 task complete 和 cl5-1 task complete 由于需要等线程创建完成才能执行，因此执行也在后面。为了验证上面的结论，我们在 ctask() 后面增加一段耗 CPU 计算的代码，让新的线程执行快于当前的主线程。</p>
<pre class="lang-dart" data-nodeid="10542"><code data-language="dart">    <span class="hljs-built_in">print</span>(<span class="hljs-string">'project start'</span>);
	ctask();
	<span class="hljs-keyword">for</span>(<span class="hljs-built_in">int</span> i=<span class="hljs-number">0</span>; i&lt;<span class="hljs-number">1000000000</span>; i++){
	}
</code></pre>
<p data-nodeid="10543">在运行代码后，你将看到这样的结果：</p>
<pre class="lang-sql" data-nodeid="10544"><code data-language="sql">project <span class="hljs-keyword">start</span>
ch5<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
cl5<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
H1 task <span class="hljs-keyword">complete</span>
H2 task <span class="hljs-keyword">complete</span>
h1<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
h2<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
C1 task <span class="hljs-keyword">complete</span>
L3 task <span class="hljs-keyword">complete</span>
h3<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
L4 task <span class="hljs-keyword">complete</span>
h4<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l1<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l2<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l3<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
l4<span class="hljs-number">-1</span> task <span class="hljs-keyword">complete</span>
</code></pre>
<p data-nodeid="10545">首先就输出了 C 线程中的微任务和事件任务，C 任务完成后，向主线程的事件任务中插入事件任务。由于主线程还没有运行结束，接下来运行后会产生微任务和事件任务，由于 C 回调的事件任务最先插入，因此在事件任务中最先执行，但是会慢于微任务事件的执行。</p>
<h3 data-nodeid="10546">总结</h3>
<p data-nodeid="10547">本课时首先介绍了 Dart 中单线程两个概念微任务事件队列和事件任务队列，并通过实践代码运行来介绍 Dart 事件循环方式。其次介绍了在 Dart 中应用 isolate 实现多线程的方式。最后使用一个实际的例子，来练习掌握 Dart 运行原理。在综合例子里还涉及了多线程中微任务和事件任务的调度方式。</p>
<p data-nodeid="10548">学完本课时，你需要掌握其单线程中微任务队列和事件任务队列的调度方式，其次知道线程创建需要处理时间，以及线程事件执行完成后的回调是一个事件任务，这样就可以掌握其整体的运行原理了。如果你还有其他困惑，可以在下方留言或加入学习交流群。</p>
<p data-nodeid="10549">以上就是本课时的主要内容，下一课时，我将用“三步法”带你掌握 Flutter ，并开始你的第一个应用，这也是我们即将开始实际的代码编程的第一步。</p>
<p data-nodeid="10550">点击这里下载本课时源码，Flutter 专栏，源码地址：<a href="https://github.com/love-flutter/flutter-column" data-nodeid="10686">https://github.com/love-flutter/flutter-column</a></p>

---

### 精选评论

##### **伟：
> 先微任务再事件任务，dart线程不共享变量，线程间通过消息机制传递信息。

##### **成：
> 等待实战

##### **玲慧：
> 既然线程之间的内存是不共享的，多线程章节例子中的name是怎么做到的？name是在主线程中定义，但是在声明的线程中并没有定义而是直接赋值了，这块怎么理解？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 很好的问题。在main函数外是一个全局变量，全局变量在任何线程都可以访问到，如果你在main函数中去声明一个变量就无法在另外一个线程访问到了。

##### **亮：
> 开发中async配合关键字await是不是相当于isolate加receive？它们都是开劈一个新线程吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是，async/await 不是新的线程，async/await 只是将异步回调的写法修改了同步写法，实际上还是异步等待执行，并非多线程处理。

##### **云：
> 老师，C的运行结果跟上面的不一样

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 具体是指哪部分不一样呢？如果有问题你把结果放到 github 的 issue 中，我会去 github 的 issue 上看下，并且回复你。

##### **的阿科：
> 1 微队列和事件队列，微队列优先于事件队列执行，微队列执行完才会去执行事件队列，如果事件中又有微任务产生，会在任务队列执行完后再去执行微任务2 一个单线程就是一个isolate,两个线程间不共享内存，但任务队列是共享的3 线程创建是需要时间的，创建出来可能主线程都执行完了4 线程事件的回调是一个事件任务，放在事件队列中😀

##### *晋：
> 根据运行结果来看，也就是说在其他线程中无法修改另外一线程中变量的值？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 理解是对的，因为内存隔离的特点，单个线程中的变量只会在当前线程内存中，其他线程师无法直接修改到当前内存的变量。

##### **珍：
> 还是不明白async await如何实现，我只知道MessageLoop和定时器。老师讲解一下呗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; async 的作用就是标记可异步处理返回的函数。await 的原理就是将await后面的代码（该执行函数中await后面的所有代码）作为一个Future也就是事件任务插入到事件任务队列中。每遇到一个await就执行类似的操作，这样说你是否能够理解。

##### **星：
> 怎么理解dart 是单线程呢？后面又说可以用Isolate实现多线程，我可以是单CUP，同时只能有一条线程再跑吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这里所说的单线程是指 Flutter 的主线程，如果你了解 JavaScript / Node.js 的话也是一样是单线程处理。主线程单线程处理，但是在主线程中可以使用 isolate 来新建其他线程，处理某些功能模块。这里所说的单线程，并不是说整个运行过程是只有一个线程在处理，记住是主线程，毕竟在 Flutter 中并非也只有一个线程，除了主线程，还会存在 UI 线程、GPU 线程和 IO 线程等。

##### **阳：
> async await 没有讲？执行顺序是怎么样的？<br>

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 如果了解Promise的话，就比较好理解，Flutter中的Future类似，而 async/await的作用就是将原来嵌套或者then语法转化为同步写法。async/await 创建的是一个事件任务，因此整体流程按照事件任务来执行。

##### **清：
> 你好，请问下await读取文件是怎么处理的，我理解的是读取文件是Flutter的IO线程，那主线程里面是怎么样的机制去检测文件读取完毕了？await读取文件的时候是不是加了任务到event queue？能够帮忙解答下吗，一直找不到这块相关的资料，非常感谢！

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 文件读取的确是一个I/O线程，通过事件驱动的方式，当文件I/O执行完成后，通过调用回调函数来触发主线程逻辑。回调处理函数是一个事件任务队列，也就是在执行完成后，不会立马调用回调函数，而是将回调函数放入事件任务队列。

##### **榜：
> 代码对齐看着太难受了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 习惯成自然，可能需要点时间来适应。

##### **帅：
> 第一次接触dart语言，希望能在语法上讲解稍微细一点

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 详细的可以看下官网https://dart.dev/里面的内容文档，部分语法可以在应用中去掌握。建议先掌握一些基本的，然后在实践中去学习应用。

##### **泽：
> future.也是把任务加到event queue吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，理解正确哈，因此在微任务队列是优先于这个执行的。但是这里要注意的是future的then是一个函数，因此在执行到future的时候，会立马执行起then函数，而不是将then放到事件任务队列中。

##### *亮：
> 学习 加油

