<p data-nodeid="4735" class="">上一讲我们介绍了影响 Node.js 服务性能的一个关键点，也就是 <strong data-nodeid="4871">CPU 的密集型计算</strong>，通过例子，你可以看到只要出现这类请求，基本就会导致服务器瘫痪。那么是否有办法来保护我们的服务呢？比如说我们是否可以丢弃部分 /v1/cpu 的请求，但是可以正常响应 /v1/normal 的用户请求，这就是我们这一讲要介绍的知识点，也就是 CPU 过载保护机制。</p>
<h3 data-nodeid="4736">过载保护</h3>
<p data-nodeid="4737">假设一种场景，我们去银行办事，大家都知道需要拿号排队，银行每 10 分钟处理 1 个人的业务，而每 10 分钟会进来 2 个人，这样每 10 分钟就会积压一个用户，然后偶数进来的用户还需要多等 10 分钟，从而就会导致每个人的等待时长是 ((n + 1) / 2 - 1 + (n + 1) % 2) * 10。</p>
<p data-nodeid="4738">其中变量 n 为第几个进来的用户。随着 n 越大，等待的时间就越长，如果没有及时制止，银行将永远都是饱和状态。长时间饱和工作状态，银行人员将会很辛苦，从而无法更好服务用户。一般情况下，在银行都会有一定的取号上限或者保安会提示无法再服务了，这就是一个<strong data-nodeid="4881">过载的保护</strong>，避免因事务积压，导致系统无法提供更好的服务。</p>
<p data-nodeid="4739">以上是一个简单的例子，接下来我们从技术层面介绍过载保护概念，而由于 Node.js 最大的性能损耗又在于 CPU，因此又需要进一步了解什么是 CPU 的过载保护。</p>
<h4 data-nodeid="4740">1.什么是过载保护</h4>
<p data-nodeid="4741"><strong data-nodeid="4888">这个词最早出现是在电路方面</strong>，在出现短路或者电压承载过大时，会触发电源的过载保护设备，该设备要不熔断、要不跳闸切断电源。</p>
<p data-nodeid="4742">在服务端也是相似的原理，首先我们需要设计一个过载保护的服务，在过载触发时，切断用户服务直接返回报错，在压力恢复时，正常响应用户请求。</p>
<h4 data-nodeid="4743">2.CPU 过载保护</h4>
<p data-nodeid="4744">在 Node.js 中最大的瓶颈在于 CPU，因此我们需要针对 CPU 的过载进行保护。当 CPU 使用率超出一定范围时，进行请求熔断处理，直接报错返回，接下来我们来看下具体的实现原理。</p>
<h3 data-nodeid="4745">实现方案</h3>
<p data-nodeid="4746">在实现方案前，我们需要思考几个关键的问题：</p>
<ul data-nodeid="4747">
<li data-nodeid="4748">
<p data-nodeid="4749">获取当前进程所在的 CPU 使用率的方法；</p>
</li>
<li data-nodeid="4750">
<p data-nodeid="4751">应尽量避免影响服务性能；</p>
</li>
<li data-nodeid="4752">
<p data-nodeid="4753">什么时候触发过载，能否减少误处理情况；</p>
</li>
<li data-nodeid="4754">
<p data-nodeid="4755">请求丢弃方法和优先级；</p>
</li>
</ul>
<p data-nodeid="4756">接下来我们看下这几个部分的实现方法。</p>
<h4 data-nodeid="4757">1.获取 CPU 使用率</h4>
<p data-nodeid="4758">Node.js 进程启动后，都会绑定在单核 CPU 上。假设机器有 2 个 CPU 内核，我们只启动了一个进程，那么在没有其他外在因素影响的情况下，Node.js 即使跑满 CPU，也最多只占用了 50% 的总机器的 CPU 利用率。因此这里我需要获取该进程 CPU 使用率。</p>
<p data-nodeid="4759"><strong data-nodeid="4913">我们需要获取当前进程下的 CPU 使用情况，而不是整体机器的 CPU</strong>，<strong data-nodeid="4914">因此需要使用 PS 这个命令，而不是利用 Node.js 本身的 OS 模块</strong>。这里我们以 Mac 为例子，其他部分你可以参考 <a href="https://github.com/love-flutter/nodejs-column" data-nodeid="4911">GitHub 源码</a>。</p>
<p data-nodeid="4760">首先我们需要使用一个命令：</p>
<pre class="lang-java" data-nodeid="4761"><code data-language="java">$ ps -p ${process.pid} -o pid,rss,vsz,pcpu,comm
</code></pre>
<p data-nodeid="4762">这一命令是<strong data-nodeid="4921">获取当前 Node.js 进程下的进程信息</strong>：</p>
<ul data-nodeid="4763">
<li data-nodeid="4764">
<p data-nodeid="4765"><strong data-nodeid="4926">pid 是进程 ID</strong>；</p>
</li>
<li data-nodeid="4766">
<p data-nodeid="4767"><strong data-nodeid="4931">rss 是实际内存占用</strong>；</p>
</li>
<li data-nodeid="4768">
<p data-nodeid="4769"><strong data-nodeid="4936">vsz 是虚拟内存占用</strong>；</p>
</li>
<li data-nodeid="4770">
<p data-nodeid="4771"><strong data-nodeid="4941">pcpu 是 CPU 使用率</strong>；</p>
</li>
<li data-nodeid="4772">
<p data-nodeid="4773"><strong data-nodeid="4946">comm 是进程执行的指令</strong>。</p>
</li>
</ul>
<p data-nodeid="4774">在 Linux 或者 Mac 系统中可以直接运行以上命令，查看某些进程的信息。</p>
<p data-nodeid="4775">有了命令后，我们需要在 Node.js 中执行修改命令，并获取执行结果，以下代码就是在 Node.js 执行修改命令的方法。</p>
<pre class="lang-javascript" data-nodeid="4776"><code data-language="javascript"><span class="hljs-comment">/**
 * <span class="hljs-doctag">@description </span>使用 ps 命令获取进程信息
 */</span>
<span class="hljs-keyword">async</span> _getPs() {
    <span class="hljs-comment">// 命令行</span>
    <span class="hljs-keyword">const</span> cmd = <span class="hljs-string">`ps -p <span class="hljs-subst">${process.pid}</span> -o pid,rss,vsz,pcpu,comm`</span>;
    <span class="hljs-comment">// 获取执行结果</span>
    <span class="hljs-keyword">const</span> { stdout, stderr } = <span class="hljs-keyword">await</span> exec(cmd);
    <span class="hljs-keyword">if</span>(stderr) { <span class="hljs-comment">// 异常情况</span>
      <span class="hljs-built_in">console</span>.log(stderr);
      <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
    }
    <span class="hljs-keyword">return</span> stdout;
}
</code></pre>
<p data-nodeid="4777"><strong data-nodeid="4953">在上面代码中 exec 是一个经过 util.promisify 处理的方法，而不是 Node.js 原生模块的 exec 方法</strong>，处理逻辑如下：</p>
<pre class="lang-javascript" data-nodeid="4778"><code data-language="javascript"><span class="hljs-keyword">const</span> util = <span class="hljs-built_in">require</span>(<span class="hljs-string">'util'</span>);
<span class="hljs-keyword">const</span> exec = util.promisify(<span class="hljs-built_in">require</span>(<span class="hljs-string">'child_process'</span>).exec);
</code></pre>
<p data-nodeid="4779">获取到进程信息后，我们需要将进程信息转化为相应的数据对象，具体方法如下：</p>
<pre class="lang-javascript te-preview-highlight" data-nodeid="5143"><code data-language="javascript"><span class="hljs-comment">/**
 * <span class="hljs-doctag">@description </span>获取进程信息
 */</span>
<span class="hljs-keyword">async</span> _getProcessInfo() {
    <span class="hljs-keyword">let</span> pidInfo, cpuInfo;

    <span class="hljs-keyword">if</span> (platform === <span class="hljs-string">'win32'</span>) { <span class="hljs-comment">// windows 平台</span>
      pidInfo = <span class="hljs-keyword">await</span> <span class="hljs-keyword">this</span>._getWmic();
    } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 其他平台 linux &amp; mac</span>
      pidInfo = <span class="hljs-keyword">await</span> <span class="hljs-keyword">this</span>._getPs();
    }
    cpuInfo = <span class="hljs-keyword">await</span> <span class="hljs-keyword">this</span>._parseInOs(pidInfo);

    <span class="hljs-keyword">if</span>(!cpuInfo) { <span class="hljs-comment">// 异常处理</span>
      <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
    }
    <span class="hljs-comment">/// 命令行数据，字段解析处理</span>
    <span class="hljs-keyword">const</span> pid = <span class="hljs-built_in">parseInt</span>(cpuInfo.pid, <span class="hljs-number">10</span>);
    <span class="hljs-keyword">const</span> name = cpuInfo.name.substr(cpuInfo.name.lastIndexOf(<span class="hljs-string">'/'</span>) + <span class="hljs-number">1</span>);
    <span class="hljs-keyword">const</span> cpu = <span class="hljs-built_in">parseFloat</span>(cpuInfo.cpu);
    <span class="hljs-keyword">const</span> mem = {
    <span class="hljs-attr">private</span>: <span class="hljs-built_in">parseInt</span>(cpuInfo.pmem, <span class="hljs-number">10</span>),
      <span class="hljs-attr">virtual</span>: <span class="hljs-built_in">parseInt</span>(cpuInfo.vmem, <span class="hljs-number">10</span>),
      <span class="hljs-attr">usage</span>: cpuInfo.pmem / totalmem * <span class="hljs-number">100</span>
    };

    <span class="hljs-keyword">return</span> {
      pid, name, cpu, mem
    }
}
</code></pre>

<p data-nodeid="4781">在上面代码中，一开始需要根据平台的不同，<strong data-nodeid="4960">调用不同的命令来获取进程信息</strong>。其他基本上都是一些字符串的处理，没有什么特殊的逻辑。</p>
<p data-nodeid="4782">以上就是一个获取当前进程的相关信息的方法，其中的 usage 就是 CPU 相关的信息，由于还是涉及非常多的逻辑处理和计算，因此我们需要思考如何简化方式，减少对主线程 CPU 性能损耗。</p>
<h4 data-nodeid="4783">2.性能影响</h4>
<p data-nodeid="4784">由于在 Node.js 就只有一个主线程，因此<strong data-nodeid="4968">必须严格减少框架在主线程的占用时间，控制框架基础模块的性能损耗，从而将主线程资源更多服务于业务，增强业务并发处理能力</strong>。为了满足这点，我们需要做两件事情：</p>
<ul data-nodeid="4785">
<li data-nodeid="4786">
<p data-nodeid="4787"><strong data-nodeid="4973">只处理需要的数据</strong>，因此在第一步获取 CPU 使用率的基础上，我们需要缩减一些字段，只获取 CPU 信息即可；</p>
</li>
<li data-nodeid="4788">
<p data-nodeid="4789"><strong data-nodeid="4978">定时落地 CPU 信息到内存中</strong>，而非根据用户访问来实时计算。</p>
</li>
</ul>
<p data-nodeid="4790">在第一点上，我们把原来获取的 pid、rss、vsz、comm 全部去掉，只留下 pcpu，然后将逻辑优化。第二点则需要定时设置内存中的 CPU 使用率，这部分代码如下：</p>
<pre class="lang-javascript" data-nodeid="4791"><code data-language="javascript"><span class="hljs-keyword">async</span> check(maxOverloadNum =<span class="hljs-number">30</span>, maxCpuPercentage=<span class="hljs-number">80</span>) {
     <span class="hljs-comment">/// 定时处理逻辑</span>
     setInterval(<span class="hljs-keyword">async</span> () =&gt; {
        <span class="hljs-keyword">try</span> {
            <span class="hljs-keyword">const</span> cpuInfo = <span class="hljs-keyword">await</span> <span class="hljs-keyword">this</span>._getProcessInfo();
            <span class="hljs-keyword">if</span>(!cpuInfo) { <span class="hljs-comment">// 异常不处理</span>
                <span class="hljs-keyword">return</span>;
            }
            <span class="hljs-keyword">if</span>(cpuInfo &gt; maxCpuPercentage) {
                overloadTimes++;
            } <span class="hljs-keyword">else</span> {
                overloadTimes = <span class="hljs-number">0</span>;
                <span class="hljs-keyword">return</span> isOverload = <span class="hljs-literal">false</span>;
            }
            <span class="hljs-keyword">if</span>(overloadTimes &gt; maxOverloadNum){
                isOverload = <span class="hljs-literal">true</span>;
            }
        } <span class="hljs-keyword">catch</span>(err){
            <span class="hljs-built_in">console</span>.log(err);
            <span class="hljs-keyword">return</span>;
        }
    }, <span class="hljs-number">2000</span>);
}
</code></pre>
<p data-nodeid="4792">上面代码中使用了 <strong data-nodeid="4985">setInterval</strong> 来实现，每秒执行一次。在代码中的两个参数 maxOverloadNum 和 maxCpuPercentage：</p>
<ul data-nodeid="4793">
<li data-nodeid="4794">
<p data-nodeid="4795">maxOverloadNum 表示最大持续超出负载次数，当大于该值时才会判断为超出负载了；</p>
</li>
<li data-nodeid="4796">
<p data-nodeid="4797">maxCpuPercentage 表示单次 CPU 使用率是否大于该分位值，大于则记录一次超载次数。</p>
</li>
</ul>
<p data-nodeid="4798">最后我们再看下应用的地方，如下所示，整个代码在 <a href="https://github.com/love-flutter/nodejs-column" data-nodeid="4991">GitHub 项目</a>的 index.js 文件中。</p>
<pre class="lang-javascript" data-nodeid="4799"><code data-language="javascript">cpuOverload.check().then().catch(<span class="hljs-function"><span class="hljs-params">err</span> =&gt;</span> {
<span class="hljs-built_in">console</span>.log(err)
});
</code></pre>
<p data-nodeid="4800">上面代码主要是调用 <strong data-nodeid="4998">check 方法</strong>，并且用来捕获异常，避免引起服务器崩溃。</p>
<h4 data-nodeid="4801">3.概率丢弃</h4>
<p data-nodeid="4802">在获取 CPU 值以后，我们可以根据当前 CPU 的情况进行一些丢弃处理，但是应尽量避免出现<strong data-nodeid="5009">误处理</strong>的情况。比如当前 CPU 某个时刻出现了过高，但是立马恢复了，这种情况下我们是不能进行丢弃请求的，<strong data-nodeid="5010">只有当 CPU 长期处于一个高负载情况下才能进行请求丢弃</strong>。</p>
<p data-nodeid="4803">即使要丢请求，也需要根据概率来丢弃，而不是每个请求都丢弃，我们需要根据三个变量：</p>
<ul data-nodeid="4804">
<li data-nodeid="4805">
<p data-nodeid="4806"><strong data-nodeid="5018">overloadTimes</strong>，用 o 表示，指 CPU 过载持续次数，该值越高则丢弃概率越大，设定取值范围为 0 ~ 10；</p>
</li>
<li data-nodeid="4807">
<p data-nodeid="4808"><strong data-nodeid="5025">currentCpuPercentage</strong>，用 c 表示，指 CPU 当前负载越高，占用率越大则丢弃概率越大，这里设定范围为 0 ~ 10，10 代表是最大值 100% ；</p>
</li>
<li data-nodeid="4809">
<p data-nodeid="4810"><strong data-nodeid="5032">baseProbability</strong>，用 b 表示，是负载最大时的丢弃概率，取值范围为 0 ~ 1。</p>
</li>
</ul>
<p data-nodeid="4811">虽然都是<strong data-nodeid="5038">正向反馈</strong>，但是三者对结果影响是不同的：</p>
<ul data-nodeid="4812">
<li data-nodeid="4813">
<p data-nodeid="4814"><strong data-nodeid="5043">overloadTimes 可以看作是直线型</strong>，但是影响系数为 0.1；</p>
</li>
<li data-nodeid="4815">
<p data-nodeid="4816"><strong data-nodeid="5048">baseProbability 我们也可以看作是直线型</strong>；</p>
</li>
<li data-nodeid="4817">
<p data-nodeid="4818">而 <strong data-nodeid="5054">currentCpuPercentage 则是一个指数型增长模型</strong>。</p>
</li>
</ul>
<p data-nodeid="4819">可以得出一个简单的算法公式，如下所示：</p>
<pre class="lang-java" data-nodeid="4820"><code data-language="java">P = (<span class="hljs-number">0.1</span> * o) * Math.exp(c) / (<span class="hljs-number">10</span> * Math.exp(<span class="hljs-number">10</span>)) * b
</code></pre>
<p data-nodeid="4821">其中 o 取最大值 100，c 取最大值 10，b 为固定值，这里假设为 0.7，那么求出来的最大概率是 0.7 ；那么在 o 为 30，c 为 90 的概率则是 0.19 ，因此会丢弃 19% 的用户请求。</p>
<p data-nodeid="4822">接下来我们先实现该 P 概率公式，代码如下：</p>
<pre class="lang-javascript" data-nodeid="4823"><code data-language="javascript"><span class="hljs-comment">/**
 * <span class="hljs-doctag">@description </span>获取丢弃概率
 */</span>
_setProbability() {
     <span class="hljs-keyword">let</span> o = overloadTimes &gt;= <span class="hljs-number">100</span> ? <span class="hljs-number">100</span> : overloadTimes;
     <span class="hljs-keyword">let</span> c = currentCpuPercentage &gt;= <span class="hljs-number">100</span> ? <span class="hljs-number">10</span> : currentCpuPercentage/<span class="hljs-number">10</span>;
     currentProbability = ((<span class="hljs-number">0.1</span> * o) * <span class="hljs-built_in">Math</span>.exp(c) / maxValue * <span class="hljs-keyword">this</span>.baseProbability).toFixed(<span class="hljs-number">4</span>);
}
</code></pre>
<p data-nodeid="4824">为了性能考虑，我们会将上面的 10 * Math.exp(10) 作为一个 const 值，避免重复计算，其次这个方法是在 check 函数中调用，2 秒处理一次，避免过多计算影响 CPU 性能。然后我们再来实现一个<strong data-nodeid="5065">获取随机数</strong>的方法，代码如下：</p>
<pre class="lang-javascript" data-nodeid="4825"><code data-language="javascript"><span class="hljs-comment">/**
 * <span class="hljs-doctag">@description </span>获取一个概率值
 */</span>
_getRandomNum(){
    <span class="hljs-keyword">return</span> <span class="hljs-built_in">Math</span>.random();
}
</code></pre>
<p data-nodeid="4826">最后我们在 isAvailable 函数中判断当前的随机数是否大于等于概率值，如果小于概率值则丢弃该请求，大于则认为允许请求继续访问，如下代码所示：</p>
<pre class="lang-javascript" data-nodeid="4827"><code data-language="javascript">isAvailable(path, uuid) {
    <span class="hljs-keyword">if</span>(isOverload) {
      <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>._getRandomNum() &lt;= <span class="hljs-keyword">this</span>._getProbability()) {
          <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
      }
      <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
}
</code></pre>
<p data-nodeid="4828">以上就是判断是否需要丢弃的逻辑。在某些情况下，我们需要做一定的优化，避免一些重要的请求无法触达用户，因此还需要做一些优化级和同一个 uuid 进行优化的策略。</p>
<h4 data-nodeid="4829">4.优先级处理</h4>
<p data-nodeid="4830">这里我们需要考虑 2 个点：</p>
<ul data-nodeid="4831">
<li data-nodeid="4832">
<p data-nodeid="4833"><strong data-nodeid="5074">优先级问题</strong>，因为有些核心的请求我们不希望用户在访问时出现丢弃的情况，比如支付或者其他核心重要的流程；</p>
</li>
<li data-nodeid="4834">
<p data-nodeid="4835">其次对于一个用户，我们允许了该用户访问其中一个接口，那么其他接口在短时间内应该也允许请求，不然会导致有些接口响应成功，有些失败，那么用户还是无法正常使用。</p>
</li>
</ul>
<p data-nodeid="4836"><strong data-nodeid="5079">优先级的实现</strong></p>
<p data-nodeid="4837">优先级实现最简单的方式，就是接受一个<strong data-nodeid="5085">白名单参数</strong>，如果设置了则会在白名单中的请求通过处理，无须校验，如果不在才会进行检查，代码实现如下：</p>
<pre class="lang-javascript" data-nodeid="4838"><code data-language="javascript">isAvailable(path, uuid) {
    <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>.whiteList.includes(path)) {
        <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
    }
    <span class="hljs-keyword">if</span>(isOverload) {
        <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>._getRandomNum() &lt;= currentProbability) {
            <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
        }
        <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
}
</code></pre>
<p data-nodeid="4839"><strong data-nodeid="5089">uuid 处理</strong></p>
<p data-nodeid="4840">这部分稍微复杂一些，首先我们需要考虑<strong data-nodeid="5099">时效性</strong>，如果存储没有时效会导致存储数据过大，从而引起内存异常问题，其次应该考虑使用<strong data-nodeid="5100">共享内存 Redis 方式</strong>，因为有可能是多机器部署。这里为了简单化，会使用本地内存的方式，但是也需要考虑上限，超过上限剔除第一个元素，代码实现如下：</p>
<pre class="lang-javascript" data-nodeid="4841"><code data-language="javascript">isAvailable(path, uuid) {
    <span class="hljs-keyword">if</span>(path &amp;&amp; <span class="hljs-keyword">this</span>.whiteList.includes(path)) { <span class="hljs-comment">// 判断是否在白名单内</span>
        <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
    }
    <span class="hljs-keyword">if</span>(uuid &amp;&amp; canAccessList.includes(uuid)){ <span class="hljs-comment">// 判断是否已经放行过</span>
        <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
    }
    <span class="hljs-keyword">if</span>(isOverload) {
         <span class="hljs-keyword">if</span>(<span class="hljs-keyword">this</span>._getRandomNum() &lt;= currentProbability) {
            removeCount++;
            <span class="hljs-keyword">return</span> <span class="hljs-literal">false</span>;
          }
    }
    <span class="hljs-keyword">if</span>(uuid) { <span class="hljs-comment">// 需要将 uuid 加入放行数组</span>
        <span class="hljs-keyword">if</span>(canAccessList.length &gt; maxUser){
            canAccessList.shift()
        }
        canAccessList.push(uuid);
    }
    <span class="hljs-keyword">return</span> <span class="hljs-literal">true</span>;
}
</code></pre>
<p data-nodeid="4842">以上就实现这个过载模块了，重点要注意的是获取 CPU 使用率的方法、减少性能影响、概率丢弃和优先级处理。接下来我们就实践应用一下，首先我们可以对比下性能影响，在没有应用和应用之后两者的空转性能对比。</p>
<h3 data-nodeid="4843">实践应用</h3>
<p data-nodeid="4844">在下一讲中我们会将 MSVC 框架转化为 Koa 框架接入，这里我们还是以最原始的框架为基础来接入 MSVC。</p>
<h4 data-nodeid="4845">1.接入 MSVC</h4>
<p data-nodeid="4846">首先我们需要在入口文件初始化过载保护模块，并且调用 check 方法，定时获取 CPU 信息，代码如下:</p>
<pre class="lang-javascript" data-nodeid="4847"><code data-language="javascript"><span class="hljs-keyword">const</span> cpuOverload = <span class="hljs-keyword">new</span> (<span class="hljs-built_in">require</span>(<span class="hljs-string">'./util/cpuOverload'</span>))();
<span class="hljs-comment">/**
 * 处理 cpu 信息采集
 */</span>
cpuOverload.check().then().catch(<span class="hljs-function"><span class="hljs-params">err</span> =&gt;</span> {
    <span class="hljs-built_in">console</span>.log(err)
});
</code></pre>
<p data-nodeid="4848">接下来在请求转发处，先进行判断，在进入业务之前就进行拦截处理，代码如下图 1 所示：</p>
<p data-nodeid="4849"><img src="https://s0.lgstatic.com/i/image6/M00/1D/E4/Cgp9HWBQK5GADhMxAAHtP-9awms474.png" alt="Drawing 0.png" data-nodeid="5109"></p>
<div data-nodeid="4850"><p style="text-align:center">图 1 增加 CPU 过载处理代码图</p></div>
<p data-nodeid="4851">使用起来比较简单，接下来我们就来看看实际性能对比。</p>
<h4 data-nodeid="4852">2.性能分析对比</h4>
<p data-nodeid="4853">我们对移除 CPU 过载保护代码和加上过载保护逻辑后的压测数据，使用压测工具进行压测，这里你只需要了解 WRK 即可，具体压测工具我们还会在《12 | 性能分析：性能影响的关键路径以及优化策略》中详细介绍。最后我们可以得到如下表格 1 所示的结果。</p>
<p data-nodeid="4854"><img src="https://s0.lgstatic.com/i/image6/M01/1D/E1/CioPOWBQK6KACYDqAACIqA12oSE255.png" alt="Drawing 1.png" data-nodeid="5117"></p>
<p data-nodeid="4855">上面的测试数据是在持续时长为 20 秒、CPU 占用大于 98、丢弃概率为 80% 时的测试数据，可以看出，整体上两者并没有多大差距（由于是本机器测试，会有部分误差），那么如果我们将 CPU 占用修改为 80 时，我们可以看下 1000 并发时压测数据，如下所示：</p>
<pre class="lang-java" data-nodeid="4856"><code data-language="java">&nbsp;<span class="hljs-number">10</span> threads and <span class="hljs-number">1000</span> connections
&nbsp; Thread Stats&nbsp; &nbsp;Avg&nbsp; &nbsp; &nbsp; Stdev&nbsp; &nbsp; &nbsp;Max&nbsp; &nbsp;+/- Stdev
&nbsp; &nbsp; Latency&nbsp; &nbsp; <span class="hljs-number">71.31</span>ms&nbsp; &nbsp; <span class="hljs-number">4.95</span>ms <span class="hljs-number">189.60</span>ms&nbsp; &nbsp;<span class="hljs-number">90.88</span>%
&nbsp; &nbsp; Req/Sec&nbsp; &nbsp; &nbsp;<span class="hljs-number">1.40</span>k&nbsp; &nbsp;<span class="hljs-number">171.05</span>&nbsp; &nbsp; &nbsp;<span class="hljs-number">2.25</span>k&nbsp; &nbsp; <span class="hljs-number">80.83</span>%
&nbsp; <span class="hljs-number">416766</span> requests in <span class="hljs-number">30.04</span>s, <span class="hljs-number">72.26</span>MB read
&nbsp; Socket errors: connect <span class="hljs-number">0</span>, read <span class="hljs-number">3990</span>, write <span class="hljs-number">0</span>, timeout <span class="hljs-number">0</span>
&nbsp; Non-<span class="hljs-number">2</span>xx or <span class="hljs-number">3</span>xx responses: <span class="hljs-number">12779</span>
Requests/sec:&nbsp; <span class="hljs-number">13874.51</span>
</code></pre>
<p data-nodeid="4857">你可以看到结果中平均耗时减少了，从原来的 76.96 变成了 71.31，其次增加了 503 的返回量，原来是 0 现在是 12779，在 scoket 超时方面还是基本一致的。因此在实际情况，我们需要根据业务以及机器的配置来选择这几个参数的配置，具体的关系就是我上面所提到的。<strong data-nodeid="5124">随着并发越来越高，如果没有负载保护用户的处理时长会越来越长，但是有了负载保护就可以避免雪崩现象，从而保护服务器可以正常地提供服务</strong>。</p>
<h3 data-nodeid="4858">总结</h3>
<p data-nodeid="4859">本讲首先介绍了什么是过载保护和什么是 CPU 过载保护，接下来实践教学了如何去实现一个 CPU 过载保护模块，最后实践接入 MSVC 框架，并且与基础框架进行了对比分析。学完本讲后，要掌握 CPU 过载保护的设计，同时从这个过程中，掌握在 Node.js 中应注重的代码设计原则。</p>
<p data-nodeid="4860">学完本讲后，你可以再思考下，setInterval 中的 2000 ms 是否可以进行调整，这个值的调整会有哪些影响，这部分希望你可以动手验证下效果，有任何问题，都可以在留言区与我交流。</p>
<p data-nodeid="4861">下一讲我们将会讲解在 I/O 方面应该注意哪些要点，到时见！</p>
<hr data-nodeid="4862">
<p data-nodeid="4863"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="5133"><img src="https://s0.lgstatic.com/i/image6/M00/12/FA/CioPOWBBrAKAAod-AASyC72ZqWw233.png" alt="Drawing 2.png" data-nodeid="5132"></a></p>
<p data-nodeid="4864"><strong data-nodeid="5137">《大前端高薪训练营》</strong></p>
<p data-nodeid="4865" class="">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="5141">点击链接</a>，快来领取！</p>

---

### 精选评论

##### *皇：
> 很想知道那个丢失概率的计算公式是怎么得出来的，或者是有参考资料和依据介绍一下吗？想举一反三

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有非常明确的资料来说明，我是根据自己一些经验依据来分析得出的。首先概率丢弃我觉得应该是没有问题的。
首先找到因变量，概率会随着哪些因素调整
找到因变量与结果的关系，首先分为正向和逆向，其次就是影响程度，可以近视的按照几种关系来判断，一种是直线型（但是有影响因素），一种是指数型，一种是 log(a)x类型，a分为大于1和小于1。从这些关系去摸索，找出合适的指标来分析，当然不可能完全准确，只能先假设后验证，需要多进行实验然后对比数据效果。
根据这种方法，我之前也做过一个直播推荐的算法的，正常的算法是根据多个指标，比如说礼物数、当前房间人数、总房间进出房人数，这些按照产品重要性，添加系数就可以了，但是产品提了另外一个需求，希望增加 PGC 的权重，比如一个 PGC 一开播就先给到一定的基础分排在前面。这时候你就考虑时间衰减的算法了，随着时间变化，这个基础分应该不断的降低，直至降低到0，这时候你就可以找出各种因变量，然后考虑其可能的涉及变化曲线，然后再调参数，一直调到满意为止。
很大可能是需要很长时间去调整的，也可能给不到最满意的答案，但是在这种不需要非常精准的时候，近视能解决问题也是可以的。

##### **宇：
> 大佬，实际应用中，真的需要在node层面，增加这种过载保护的策略吗？还是换成，在几个node服务之前，增加nginx服务，有策略的抛弃一些请求，或者路由到更多的其他服务上？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 实际应用过程中，看真实的架构情况，比如说你们服务经过 nginx 的那么可以直接在 nginx 处理，这种是有域名的。另外一种情况是，你们使用 Node.js 来作为网关，或者你们是内部服务调用（没有域名），没有经过 nginx，那么这些情况下都是需要 Node.js 来加一层这种保护机制的。

##### **强：
> 评论1楼：这种丢弃请求的肯定是最后的选择呀！你可以负载均衡，但是还是能过载的，还是要做过载处理，动态扩容。反正吧这个还是要站在用户和钱角度去想方案的

