<p data-nodeid="47644">在该模块的几讲中，我们都是先有问题，然后再定位分析解决问题，那么是否有方法能够在出现问题之前检测问题呢？那就需要用到本讲要介绍的 clinicjs 工具，在上线之前通过自动化的方式来发现问题。</p>
<h3 data-nodeid="47645">clinicjs 是什么</h3>
<h4 data-nodeid="47646">介绍</h4>
<p data-nodeid="47647">clinicjs 是一个 Node.js 的第三方工具，<a href="https://clinicjs.org/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="47754">clinicjs 官网</a>介绍了其核心的目的是，<strong data-nodeid="47760">帮助诊断和查明 Node.js 性能问题的工具</strong>。那么它具体能帮助我们定位查询哪些性能问题呢？</p>
<ul data-nodeid="47648">
<li data-nodeid="47649">
<p data-nodeid="47650"><strong data-nodeid="47765">CPU 异常问题</strong>，当 CPU 存在密集计算占用时可以检测出来。</p>
</li>
<li data-nodeid="47651">
<p data-nodeid="47652"><strong data-nodeid="47770">事件循环延迟问题</strong>，当主线程存在过载时，事件循环的执行时间存在延迟，可以检测出来。</p>
</li>
<li data-nodeid="47653">
<p data-nodeid="47654"><strong data-nodeid="47775">内存泄漏问题</strong>。</p>
</li>
<li data-nodeid="47655">
<p data-nodeid="47656"><strong data-nodeid="47780">句柄泄漏问题</strong>。</p>
</li>
</ul>
<p data-nodeid="47657">以上几个问题也是我们非常关注的几个点。</p>
<p data-nodeid="47658">我们先使用 npm 来安装该工具，如下所示：</p>
<pre class="lang-java" data-nodeid="47659"><code data-language="java">npm install -g clinic
clinic --help
</code></pre>
<p data-nodeid="47660">安装完成后，我们需要把《12 | 性能分析：性能影响的关键路径以及优化策略》中的 wrk 也安装了，因为其需要 wrk 来进行压测，都安装完成后，需要着重掌握以下三个命令。</p>
<p data-nodeid="47661">你可以先把我们的服务器启动起来，然后尝试在我们项目根目录去运行下面的命令 1 。</p>
<pre class="lang-java" data-nodeid="47662"><code data-language="java">clinic doctor --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/local-cache/no"</span> -- node app.js
</code></pre>
<p data-nodeid="47663"><strong data-nodeid="47789">这个命令 1 是指对服务的 local-cache/no 进行性能全局分析</strong>。</p>
<p data-nodeid="47664">我们再来看下命令 2，如下所示：</p>
<pre class="lang-java" data-nodeid="47665"><code data-language="java">clinic flame --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/local-cache/no"</span> -- node app.js
</code></pre>
<p data-nodeid="47666"><strong data-nodeid="47799">命令 2 是当全局分析出现事件延迟或者高 CPU 占用时，就需要使用该工具进行具体深入分析</strong>，关于如何应用，你可以参考<a href="https://clinicjs.org/documentation/flame/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="47797">clinic flame 官网</a>说明。</p>
<p data-nodeid="47667">以下是命令 3：</p>
<pre class="lang-java" data-nodeid="47668"><code data-language="java">clinic bubbleprof --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/local-cache/no"</span> -- node app.js
</code></pre>
<p data-nodeid="47669"><strong data-nodeid="47809">命令 3 是当全局分析出现 I/O 问题时，我们就需要使用该工具进一步分析</strong>，关于如何应用，你可以参考<a href="https://clinicjs.org/documentation/doctor/06-fixing-io-problem/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="47807"> clinic bubbleprof 官网</a>说明。</p>
<h4 data-nodeid="47670">示例分析</h4>
<p data-nodeid="47671">我们来看几个例子，以下例子都是来自 clinicjs 测试的结果。</p>
<p data-nodeid="47672">我们先来看图 1 所示的结果。</p>
<p data-nodeid="49622" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/90/Cgp9HWB37a2AKhPmAAJeXJvA1Tg457.png" alt="Drawing 0.png" data-nodeid="49626"></p>
<div data-nodeid="49623"><p style="text-align:center">图 1 clinicjs 无异常检测结果</p></div>



<p data-nodeid="47675">从图 1 可以看到一个总结性的话，正如第一行 Detected no issue 表明，本次测试没有任何问题，具体可以再看下四个结果：<strong data-nodeid="47822">CPU 占用、内存占用、事件延迟和句柄</strong>。</p>
<p data-nodeid="47676">我们再来看一种异常的情况，如图 2 所示的结果。</p>
<p data-nodeid="50750" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/98/CioPOWB37bmAOY3WAAKiAW23OY4348.png" alt="Drawing 1.png" data-nodeid="50754"></p>
<div data-nodeid="50751"><p style="text-align:center">图 2 clinicjs 异常检测结果</p></div>



<p data-nodeid="47679">上图的第一行很清晰地告诉我们，<strong data-nodeid="47833">存在 CPU 问题、内存问题和事件延迟的情况</strong>，看到以上问题后，我们接下来怎么处理呢？</p>
<p data-nodeid="47680">在 clinicjs 中如果分析出存在 CPU 和事件延迟，可以使用上面我们所介绍的 clinic flame 工具。而 I/O 问题则使用 clinic bubbleprof 来进一步定位。</p>
<h3 data-nodeid="47681">框架中接入自动化</h3>
<p data-nodeid="47682">以上是一个工具的应用，并不是我们本讲的重点，<strong data-nodeid="47841">我们本讲的核心是将该工具作为一个自动化模块接入我们的框架中</strong>。</p>
<h4 data-nodeid="47683">框架中测试的思考</h4>
<p data-nodeid="47684">clinicjs 接入后，使用方法非常简单，我们只需要使用以下命令就可以进行测试检测。</p>
<pre class="lang-java" data-nodeid="47685"><code data-language="java">clinic doctor --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/local-cache/no"</span> -- node app.js
</code></pre>
<p data-nodeid="47686">但是需要思考一下，如果每次新接口上线我们都需要一个个跑一遍，那么岂不是很花费人力，这时就需要思考下有没有办法可以自动化地做接口测试，而我们只需要深入分析有问题的接口就可以了。</p>
<p data-nodeid="47687">有了以上思考后，我们就开始来规划下，如何进行自动化测试。</p>
<h4 data-nodeid="47688">自动化</h4>
<p data-nodeid="47689">首先来整理一个流程图，来分析下我们应该如何实现这个功能。</p>
<p data-nodeid="51874" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/98/CioPOWB37cOAdvHkAAJjWOy1L2Y825.png" alt="Drawing 2.png" data-nodeid="51878"></p>
<div data-nodeid="51875"><p style="text-align:center">图 3 自动化方案流程图</p></div>



<p data-nodeid="47692">我们分析一下以上流程的每个过程：</p>
<ol data-nodeid="47693">
<li data-nodeid="47694">
<p data-nodeid="47695">我们<strong data-nodeid="47858">将需要测试的接口组装成一个配置信息</strong>，其中包含了待测试的请求路径、请求方式以及请求的参数；</p>
</li>
<li data-nodeid="47696">
<p data-nodeid="47697">因为 clinicjs 生成的是一个 html 文件，并且这个 html 文件是一个 JavaScript 执行生成的结果文件，因此需要启动一个本地服务来读取 html 并解析生成具体的 DOM 结构；</p>
</li>
<li data-nodeid="47698">
<p data-nodeid="47699">从数组取出一个待测试的接口数据，来开始测试，根据 clinicjs 的命令，生成具体的执行命令行，然后借助 child process 的 exec 来执行，并获取其执行结果；</p>
</li>
<li data-nodeid="47700">
<p data-nodeid="47701">最终会生成一个运行期间的结果，在输出的最后一行会提供一个生成后的 html 文件路径地址，一般这个文件都会生成在当前目录下的 .clinic 文件夹中；</p>
</li>
<li data-nodeid="47702">
<p data-nodeid="47703">使用一个 html 的解析工具 puppeteer 从本地服务中读取 html 文件，并解析获取其中的测试结果的 DOM 信息；</p>
</li>
<li data-nodeid="47704">
<p data-nodeid="47705">判断 DOM 信息中是否有异常结果，有异常显示异常并记录测试数据，没有则跳过；</p>
</li>
<li data-nodeid="47706">
<p data-nodeid="47707">最终测试完成以后，显示所有测试异常的接口，并把测试信息给到开发者。</p>
</li>
</ol>
<p data-nodeid="47708">根据以上的流程，我们画一个逻辑执行过程来更清晰描述下这个过程。</p>
<p data-nodeid="52994" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/90/Cgp9HWB37cyAPGhrAAEXls5qfQY168.png" alt="Drawing 3.png" data-nodeid="52998"></p>
<div data-nodeid="52995"><p style="text-align:center">图 4 自动化方案逻辑执行图</p></div>



<p data-nodeid="47711">上面的逻辑和流程图基本上是一个过程，<strong data-nodeid="47881">只是这里用函数和模块来表示了</strong>，具体我们可以看下部分代码的实现，如图 5 所示，源码在<a href="https://github.com/love-flutter/nodejs-column?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="47877">GitHub </a>中的 bin/clinic_test.js 文件中。</p>
<p data-nodeid="54110" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/98/CioPOWB37dKAaCSYAALhCgAFm2U761.png" alt="Drawing 4.png" data-nodeid="54114"></p>
<div data-nodeid="54111"><p style="text-align:center">图 5 startTestLink 代码实现</p></div>



<p data-nodeid="47714">在图 5 中的第 57-63 行逻辑中，主要是在组装 clinicjs 的测试命令，最终的命令会变成以下的命令行。</p>
<pre class="lang-java" data-nodeid="47715"><code data-language="java">cd ..
clinic doctor --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/xxxx"</span> -- node app.js
exit
</code></pre>
<p data-nodeid="47716">以上是 GET 方法，<strong data-nodeid="47892">如果是 POST 方法则需要写入 POST 数据，并且需要在命令行中加上 POST 参数</strong>，如下命令示例结果：</p>
<pre class="lang-java" data-nodeid="47717"><code data-language="java">cd ..
clinic doctor --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/xxxx -s ./bin/post_tmp.lua"</span> -- node app.js
exit
</code></pre>
<p data-nodeid="47718">拿到命令以后，再使用 Node.js 的 execSync 方法获取执行结果，请注意这里的执行结果是非常长的日志，但是在最后一行会提示 html 生成的文件地址，在图 5 中的 64 到 67 行就是获取到相应的 html 文件，拿到 html 文件后再向本地服务读取到 html 的内容，最后 parseResult 来分析是否存在性能问题。</p>
<p data-nodeid="47719">接下来我们看下 parseResult 的方法实现，代码如图 6 所示。</p>
<p data-nodeid="55779" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/98/CioPOWB37eWAX1KsAAFcIAG2wp8914.png" alt="Drawing 5.png" data-nodeid="55783"></p>
<div data-nodeid="55780"><p style="text-align:center">图 6 parseResult 代码实现</p></div>





<p data-nodeid="47722"><strong data-nodeid="47903">这里有个小技巧，就是在 Node.js 中可以模拟浏览器去读取一个 html 文件，如果 html 文件不是通过 Javascript 动态生成的，那么你可以直接 fs.readFile 去获取，如果是动态生成的则必须用 puppeteer 模拟浏览器解析 html DOM 结构了</strong>。</p>
<p data-nodeid="47723">拿到 DOM 结构后，就可以进行分析判断是否存在异常了，后面的逻辑就比较简单，都是一些判断方法了。通过以上方式，我们就可以在配置文件中增加一些接口，从而实现自动化的方式，每次我们在线上发布之前，在测试环境或者预发布环境中跑一遍测试就可以提前发现问题了。接下来我们来演示一下这个功能。</p>
<h3 data-nodeid="47724">实践测试</h3>
<p data-nodeid="47725">之前我们的项目中有几个存在 CPU 密集计算的问题，这里我们用这个工具来检测下，看下是否能帮我们定位到问题。我们把配置文件修改为如下内容：</p>
<pre class="lang-javascript" data-nodeid="47726"><code data-language="javascript"><span class="hljs-keyword">const</span> testMapping = [
    {
        <span class="hljs-string">'urlPath'</span> : <span class="hljs-string">'cache/local'</span>,
        <span class="hljs-string">'method'</span> : <span class="hljs-string">'get'</span>,
        <span class="hljs-string">'getParams'</span> : {
        },
        <span class="hljs-string">'postParams'</span> : {
        }
    },
    {
      <span class="hljs-string">'urlPath'</span> : <span class="hljs-string">'local-cache/no'</span>,
      <span class="hljs-string">'method'</span> : <span class="hljs-string">'get'</span>,
      <span class="hljs-string">'getParams'</span> : {
      },
      <span class="hljs-string">'postParams'</span> : {
      }
    },
    {
      <span class="hljs-string">'urlPath'</span> : <span class="hljs-string">'local-cache/yes'</span>,
      <span class="hljs-string">'method'</span> : <span class="hljs-string">'get'</span>,
      <span class="hljs-string">'getParams'</span> : {
      },
      <span class="hljs-string">'postParams'</span> : {
      }
    }
];
</code></pre>
<p data-nodeid="47727">接下来我们在项目根目录执行下面两个命令：</p>
<pre class="lang-java" data-nodeid="47728"><code data-language="java">cd bin
node clinic_test.js&nbsp;
</code></pre>
<p data-nodeid="47729">运行完成后，你将会看到如下结果：</p>
<pre class="lang-java" data-nodeid="47730"><code data-language="java">启动服务开始测试...
开始检测 cache/local 的接口性能问题
该接口无任何异常问题
开始检测 local-cache/no 的接口性能问题
该接口存在异常
具体详情请查看项目根目录下的
./.clinic/<span class="hljs-number">21097</span>.clinic-doctor.html
开始检测 local-cache/yes 的接口性能问题
该接口无任何异常问题
你需要处理以下问题汇总，具体请查看下面详细信息
[{<span class="hljs-string">"resultLink"</span>:<span class="hljs-string">"21097.clinic-doctor.html"</span>,<span class="hljs-string">"url"</span>:<span class="hljs-string">"http://127.0.0.1:3000/local-cache/no"</span>,<span class="hljs-string">"command"</span>:<span class="hljs-string">"clinic doctor --on-port \"wrk http://127.0.0.1:3000/local-cache/no\" -- node app.js"</span>,<span class="hljs-string">"problem"</span>:<span class="hljs-string">"Detected data analysis issue"</span>}]
</code></pre>
<p data-nodeid="47731">从原来的分析来看也是这样的结果，因为我们对 cache/local 和 local-cache/yes 做了缓存优化，所以无任何异常问题，而 local-cache/no 存在性能问题，然后我们打开  .clinic/21097.clinic-doctor.html 这个文件，可以看到如图 7 所示的结果。</p>
<p data-nodeid="56887" class=""><img src="https://s0.lgstatic.com/i/image6/M00/37/90/Cgp9HWB37e-APupPAAJ0RW3kNww550.png" alt="Drawing 6.png" data-nodeid="56891"></p>
<div data-nodeid="56888"><p style="text-align:center">图 7  local-cache/no 检测结果</p></div>



<p data-nodeid="47734">如果遇到了 CPU 占用和事件延迟这类问题就使用 clinic flame 来进一步分析，我们从刚才的运行结果中，取出测试方法，把 doctor 修改为 flame 即可，如下命令所示（请注意要回到项目根目录去运行，不要在当前 bin 目录）。</p>
<pre class="lang-java" data-nodeid="47735"><code data-language="java">clinic flame --on-port <span class="hljs-string">"wrk http://127.0.0.1:3000/local-cache/no"</span> -- node app.js
</code></pre>
<p data-nodeid="47736">成功运行后，打开相应的 html 文件，可以看到如图 8 所示的结果。</p>
<p data-nodeid="57991" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/37/90/Cgp9HWB37fiAX8tvAAJ_iWJXzkU653.png" alt="Drawing 7.png" data-nodeid="57995"></p>
<div data-nodeid="57992"><p style="text-align:center">图 8  clinic flame 运行结果</p></div>



<p data-nodeid="47739">在图 8 中我们可以非常清晰地看到提示，具体在 localCache.js 中的第 20 行占用的问题，你都不用去详细分析就可以轻松得到结论。</p>
<p data-nodeid="47740">为了使用方便，我们将此工具集成到了 package 中，只需要运行下面命令就可以了。</p>
<pre class="lang-java" data-nodeid="47741"><code data-language="java">npm run clinic-test
</code></pre>
<p data-nodeid="47742">以上就完成了该工具的自动化方法。</p>
<h3 data-nodeid="47743">总结</h3>
<p data-nodeid="47744">你可以想象一下如果没有该工具，我们每次开发完的功能都像是一次冒险，心里很忐忑，但是有了 clinicjs 工具以后，我们再也不需要担心这个事情了，在发布之前就可以预先发现这些问题，我希望如果应用 Node.js 做后台开发时，它都应该作为一个必备的工具。</p>
<p data-nodeid="47745">我们本讲实现的自动化工具，还可以继续抽离细化，希望你可以根据自己的想法生成一个比较通用的工具，如果有人抽离出来后，请给出 GitHub 地址，让大家可以一起来使用，一起来维护。</p>
<p data-nodeid="47746">下一讲我们将进入实战模块，基本把当前 Node.js 的相关知识都介绍完了，我们也开始进行一些项目尝试。</p>

---

### 精选评论

##### *舒：
> 抽象整个工具还是有点难度的，加油吧，骚年。也期望老师尽快更新接下来的五讲，看完你的专栏，准备尝试 BFF 在业务落地了

