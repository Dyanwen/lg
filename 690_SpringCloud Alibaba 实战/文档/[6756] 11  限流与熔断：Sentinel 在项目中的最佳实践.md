<p data-nodeid="1781" class="">上一讲我们讲解了微服务的雪崩效应与如何基于 Sentinel 实现初步微服务限流，掌握了部署 Sentinel Dashboard与配置 Sentinel Core 客户端的技巧。本讲咱们继续 Sentinel 这个话题，将更有针对性的讲解 Sentinel 底层的细节与限流、熔断的各种配置方式。</p>
<p data-nodeid="1782">本讲咱们主要学习三方面内容：</p>
<ul data-nodeid="1783">
<li data-nodeid="1784">
<p data-nodeid="1785">Sentinel 通信与降级背后的技术原理；</p>
</li>
<li data-nodeid="1786">
<p data-nodeid="1787">Sentinel 限流降级的规则配置；</p>
</li>
<li data-nodeid="1788">
<p data-nodeid="1789">Sentinel 熔断降级的规则配置。</p>
</li>
</ul>
<p data-nodeid="1790">下面咱们先开始第一部分。</p>
<h3 data-nodeid="1791">Sentinel Dashboard通信与降级原理</h3>
<p data-nodeid="1792">Sentinel Dashboard 是Sentinel的控制端，是新的限流与熔断规则的创建者。当内置在微服务内的 Sentinel Core（客户端）接收到新的限流、熔断规则后，微服务便会自动启用的相应的保护措施。</p>
<p data-nodeid="1793">按执行流程，Sentinel 的执行流程分为三个阶段：</p>
<ol data-nodeid="1794">
<li data-nodeid="1795">
<p data-nodeid="1796">Sentinel Core 与 Sentinel Dashboard 建立连接；</p>
</li>
<li data-nodeid="1797">
<p data-nodeid="1798">Sentinel Dashboard 向 Sentinel Core 下发新的保护规则；</p>
</li>
<li data-nodeid="1799">
<p data-nodeid="1800">Sentinel Core 应用新的保护规则，实施限流、熔断等动作。</p>
</li>
</ol>
<p data-nodeid="1801"><strong data-nodeid="2010">第一步，建立连接。</strong></p>
<p data-nodeid="1802">Sentine Core 在初始化的时候，通过 application.yml 参数中指定的 Dashboard 的 IP地址，会主动向 dashboard 发起连接的请求。</p>
<pre class="lang-yaml" data-nodeid="1803"><code data-language="yaml"><span class="hljs-comment">#Sentinel Dashboard通信地址</span>
<span class="hljs-attr">spring:</span> 
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">sentinel:</span> 
      <span class="hljs-attr">transport:</span>
        <span class="hljs-attr">dashboard:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.10</span><span class="hljs-string">:9100</span>
</code></pre>
<p data-nodeid="2209">该请求是以心跳包的方式定时向 Dashboard 发送，包含 Sentinel Core 的 AppName、IP、端口信息。这里有个重要细节：Sentinel Core为了能够持续接收到来自 Dashboard的数据，会在微服务实例设备上监听 8719 端口，在心跳包上报时也是上报这个 8719 端口，而非微服务本身的 80 端口。</p>
<p data-nodeid="3072" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/25/CioPOWBarxyAfxg4AAEytt3cAkM947.png" alt="图片1.png" data-nodeid="3076"></p>
<div data-nodeid="3073"><p style="text-align:center">Sentinel Core 向 Dashboard 建立连接</p></div>




<p data-nodeid="1806" class="">在 Sentinel Dashboard 接收到心跳包后，来自 Sentinel Core的AppName、IP、端口信息会被封装为 MachineInfo 对象放入 ConcurrentHashMap 保存在 JVM的内存中，以备后续使用。</p>
<p data-nodeid="1807"><strong data-nodeid="2021">第二步，推送新规则。</strong></p>
<p data-nodeid="1808">如果在 Dashboard 页面中设置了新的保护规则，会先从当前的 MachineInfo 中提取符合要求的微服务实例信息，之后通过 Dashboard内置的 transport 模块将新规则打包推送到微服务实例的 Sentinel Core，Sentinel Core收 到新规则在微服务应用中对本地规则进行更新，这些新规则会保存在微服务实例的 JVM 内存中。</p>
<p data-nodeid="3933" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/28/Cgp9HWBaryuANUd3AAFKoecEXLU156.png" alt="图片22.png" data-nodeid="3937"></p>
<div data-nodeid="3934"><p style="text-align:center">Sentinel Dashboard 向Sentinel Core推送新规则</p></div>


<p data-nodeid="1811"><strong data-nodeid="2029">第三步，处理请求。</strong></p>
<p data-nodeid="1812">Sentinel Core 为服务限流、熔断提供了核心拦截器 SentinelWebInterceptor，这个拦截器默认对所有请求 /** 进行拦截，然后开始请求的链式处理流程，在对于每一个处理请求的节点被称为 Slot（槽），通过多个槽的连接形成处理链，在请求的流转过程中，如果有任何一个 Slot 验证未通过，都会产生 BlockException，请求处理链便会中断，并返回“Blocked by sentinel" 异常信息。</p>
<p data-nodeid="4794" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/25/CioPOWBarziAOzV2AAFkVrbLros829.png" alt="图片2.png" data-nodeid="4798"></p>
<div data-nodeid="4795"><p style="text-align:center">SentinelWebInterceptor 实施请求拦截与保护</p></div>


<p data-nodeid="1815">那这些 Slot 都有什么作用呢？我们需要了解一下，默认 Slot 有7 个，前 3 个 Slot为前置处理，用于收集、统计、分析必要的数据；后 4 个为规则校验 Slot，从Dashboard 推送的新规则保存在“规则池”中，然后对应 Slot 进行读取并校验当前请求是否允许放行，允许放行则送入下一个 Slot 直到最终被 RestController 进行业务处理，不允许放行则直接抛出 BlockException 返回响应。</p>
<p data-nodeid="1816">以下是每一个 Slot 的具体职责：</p>
<ul data-nodeid="1817">
<li data-nodeid="1818">
<p data-nodeid="1819">NodeSelectorSlot 负责收集资源的路径，并将这些资源的调用路径，以树状结构存储起来，用于根据调用路径来限流降级；</p>
</li>
<li data-nodeid="1820">
<p data-nodeid="1821">ClusterBuilderSlot 则用于存储资源的统计信息以及调用者信息，例如该资源的 RT（运行时间）, QPS, thread count（线程总数）等，这些信息将用作为多维度限流，降级的依据；</p>
</li>
<li data-nodeid="1822">
<p data-nodeid="1823">StatistcSlot 则用于记录，统计不同维度的runtime 信息；</p>
</li>
<li data-nodeid="1824">
<p data-nodeid="1825">SystemSlot 则通过系统的状态，例如CPU、内存的情况，来控制总的入口流量；</p>
</li>
<li data-nodeid="1826">
<p data-nodeid="1827">AuthoritySlot 则根据黑白名单，来做黑白名单控制；</p>
</li>
<li data-nodeid="1828">
<p data-nodeid="1829">FlowSlot 则用于根据预设的限流规则，以及前面 slot 统计的状态，来进行限流；</p>
</li>
<li data-nodeid="1830">
<p data-nodeid="1831">DegradeSlot 则通过统计信息，以及预设的规则，来做熔断降级。</p>
</li>
</ul>
<p data-nodeid="1832">到这里我们理解了 Sentinel 通信与降级背后的执行过程，下面咱们学习如何有效配置 Sentinel 的限流策略。</p>
<h3 data-nodeid="1833">Sentinel 限流降级的规则配置</h3>
<h4 data-nodeid="1834">滑动窗口算法</h4>
<p data-nodeid="1835">实现限流降级的核心是如何统计单位时间某个接口的访问量，常见的算法有计数器算法、令牌桶算法、漏桶算法、滑动窗口算法。Sentinel 采用滑动窗口算法来统计访问量。</p>
<p data-nodeid="1836">滑动窗口算法并不复杂，咱们举例说明：某应用限流控制 1 分钟最多允许 600 次访问。采用滑动窗口算法是将每 1 分钟拆分为 6（变量）个等份时间段，每个时间段为 10 秒，6 个时间段为 1 组在下图用红色边框区域标出，而这个红色边框区域就是滑动窗口。当每产生 1 个访问在对应时间段的计数器自增加 1，当滑动窗口内所有时间段的计数器总和超过 600，后面新的访问将被限流直接拒绝。同时每过 10 秒，滑动窗口向右移动，前面的过期时间段计数器将被作废。</p>
<p data-nodeid="1837">总结下，滑动窗口算法的理念是将整段时间均分后独立计数再汇总统计，滑动窗口算法被广泛应用在各种流控场景中，请你理解它的实现过程。</p>
<p data-nodeid="5655" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/25/CioPOWBar0iAFl_9AASda6g75YE021.png" alt="图片4.png" data-nodeid="5659"></p>
<div data-nodeid="5656"><p style="text-align:center">滑动窗口算法</p></div>


<h4 data-nodeid="1840">基于 Sentinel Dashboard 的限流设置</h4>
<p data-nodeid="1841">在 Sentinel Dashboard 中“簇点链路”,找到需要限流的 URI，点击“+流控”进入流控设置。小提示，sentinel-dashboard 基于懒加载模式，如果在簇点链路没有找到对应的 URI，需要先访问下这个功能的功能后对应的 URI 便会出现。</p>
<p data-nodeid="6516" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/28/Cgp9HWBar1OAG6JAAAKivJHK_-k419.png" alt="图片5.png" data-nodeid="6520"></p>
<div data-nodeid="6517"><p style="text-align:center">流控设置界面</p></div>


<p data-nodeid="1844">流控规则项目说明主要有以下几点。</p>
<ul data-nodeid="1845">
<li data-nodeid="1846">
<p data-nodeid="1847">资源名：要流控的 URI，在 Sentinel 中 URI 被称为“资源”；</p>
</li>
<li data-nodeid="1848">
<p data-nodeid="1849">针对来源：默认 default 代表所有来源，可以针对某个微服务或者调用者单独设置；</p>
</li>
<li data-nodeid="1850">
<p data-nodeid="1851">阈值类型：是按每秒访问数量（QPS）还是并发数（线程数）进行流控；</p>
</li>
<li data-nodeid="1852">
<p data-nodeid="1853">单机阈值：具体限流的数值是多少。</p>
</li>
</ul>
<p data-nodeid="8247" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/28/Cgp9HWBar16Abbz2AAOQnjLspDY532.png" alt="图片6.png" data-nodeid="8251"></p>
<div data-nodeid="8248"><p style="text-align:center">默认流控规则</p></div>




<p data-nodeid="1856">点击对话框中的“高级选项”，就会出现更为详细的设置项。</p>
<p data-nodeid="1857">其中流控模式是指采用什么方式进行流量控制。Sentinel支持三种模式：直接、关联、链路，下面咱们分别讲解这三种模式。</p>
<ul data-nodeid="1858">
<li data-nodeid="1859">
<p data-nodeid="1860"><strong data-nodeid="2075">直接模式：</strong></p>
</li>
</ul>
<p data-nodeid="1861">以下图为例，当 List 接口 QPS 超过 1个时限流，浏览器会出现“Blocked by Sentinel”。</p>
<p data-nodeid="9108" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/28/Cgp9HWBar4CARQwVAAEmI2EpwUs844.png" alt="图片7.png" data-nodeid="9112"></p>
<div data-nodeid="9109"><p style="text-align:center">流控模式-直接</p></div>


<ul data-nodeid="1864">
<li data-nodeid="1865">
<p data-nodeid="1866"><strong data-nodeid="2083">关联模式：</strong></p>
</li>
</ul>
<p data-nodeid="1867">如下图所示，当同 List 接口关联的update 接口 QPS 超过 1 时，再次访问List 接口便会响应“Blocked by Sentinel”。</p>
<p data-nodeid="10839" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/25/CioPOWBar4qAaeVSAAEgPLPCgYU751.png" alt="图片8.png" data-nodeid="10843"></p>
<div data-nodeid="10840"><p style="text-align:center">流控模式-关联</p></div>




<ul data-nodeid="1870">
<li data-nodeid="1871">
<p data-nodeid="1872"><strong data-nodeid="2091">链路模式：</strong></p>
</li>
</ul>
<p data-nodeid="1873">链路模式相对复杂，我们举例说明，现在某公司开发了一个单机的电商系统，为了满足完成“下订单”的业务，程序代码会依次执行<strong data-nodeid="2097">订单创建方法-&gt;减少库存方法-&gt;微信支付方法-&gt;短信发送</strong>方法。方法像链条一样从前向后依次执行，这种执行的链条被称为调用链路，而链路模式限流就是为此而生。</p>
<p data-nodeid="1874">以下图为例，在某个微服务中 List 接口，会被 Check 接口调用。在另一个业务，List 接口也会被 Scan 接口调用。</p>
<p data-nodeid="11700" class=""><img src="https://s0.lgstatic.com/i/image6/M00/26/28/Cgp9HWBar5qAQ8kxAAB6_MghZFM405.png" alt="图片99.png" data-nodeid="11704"></p>
<div data-nodeid="11701"><p style="text-align:center">调用链路</p></div>


<p data-nodeid="1877">但如果按下图配置，将入口资源设为“/check”，则只会针对 check 接口的调用链路生效。当访问 check 接口的QPS 超过 1 时，List 接口就会被限流。而另一条链路从 scan 接口到List 接口的链路则不会受到任何影响。链路模式与关联模式最大的区别是 check 接口与 List 接口必须是在同一个调用链路中才会限流，而关联模式是任意两个资源只要设置关联就可以进行限流。</p>
<p data-nodeid="12561" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar6SAU6wQAAFdYOUQcEQ336.png" alt="图片9.png" data-nodeid="12565"></p>
<div data-nodeid="12562"><p style="text-align:center">流控模式-链路</p></div>


<p data-nodeid="1880">讲完了直接、关联、链路三种流控模式，下面咱们聊一聊高级选项中的“流控效果”。</p>
<p data-nodeid="1881">流控效果是指在达到流控条件后，对当前请求如何进行处理。流控效果有三种：<strong data-nodeid="2120">快速失败</strong>、<strong data-nodeid="2121">Warm UP（预热）</strong>、<strong data-nodeid="2122">排队等待</strong>。</p>
<ul data-nodeid="1882">
<li data-nodeid="1883">
<p data-nodeid="1884"><strong data-nodeid="2126">快速失败：</strong></p>
</li>
</ul>
<p data-nodeid="1885">快速失败是指流浪当过限流阈值后，直接返回响应并抛出 BlockException，快速失败是最常用的处理形式。如下图所示，当 List 接口每秒 QPS 超过 1 时，可以直接抛出“Blocked By Sentinel”异常。</p>
<p data-nodeid="13422" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/29/Cgp9HWBar7CAPY7AAAGALPdbIwo406.png" alt="图片11.png" data-nodeid="13426"></p>
<div data-nodeid="13423"><p style="text-align:center">流控效果-快速失败</p></div>


<ul data-nodeid="1888">
<li data-nodeid="1889">
<p data-nodeid="1890"><strong data-nodeid="2134">Warm Up（预热）：</strong></p>
</li>
</ul>
<p data-nodeid="1891">Warm Up 用于应对瞬时大并发流量冲击。当遇到突发大流量 Warm Up 会缓慢拉升阈值限制，预防系统瞬时崩溃，这期间超出阈值的访问处于队列等待状态，并不会立即抛出 BlockException。</p>
<p data-nodeid="1892">如下图所示，List 接口平时单机阈值 QPS 处于低水位：默认为 1000/3 (冷加载因子)≈333，当瞬时大流量进来，10 秒钟内将 QPS 阈值逐渐拉升至 1000，为系统留出缓冲时间，预防突发性系统崩溃。</p>
<p data-nodeid="14283" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar76AYECLAAKiRTkN46w430.png" alt="图片12.png" data-nodeid="14287"></p>
<div data-nodeid="14284"><p style="text-align:center">流控效果-Warm Up</p></div>


<ul data-nodeid="1895">
<li data-nodeid="1896">
<p data-nodeid="1897"><strong data-nodeid="2143">排队等待：</strong></p>
</li>
</ul>
<p data-nodeid="1898">排队等待是采用匀速放行的方式对请求进行处理。如下所示，假设现在有100个请求瞬间进入，那么会出现以下几种情况：</p>
<ol data-nodeid="1899">
<li data-nodeid="1900">
<p data-nodeid="1901">单机 QPS 阈值=1，代表每 1 秒只能放行 1 个请求，其他请求队列等待，共需 100 秒处理完毕；</p>
</li>
<li data-nodeid="1902">
<p data-nodeid="1903">单机 QPS 阈值=4，代表 250 毫秒匀速放行 1 个请求，其他请求队列等待，共需 25 秒处理完毕；</p>
</li>
<li data-nodeid="1904">
<p data-nodeid="1905">单机 QPS 阈值=200，代表 5 毫秒匀速放行一个请求，其他请求队列等待，共需 0.5 秒处理完毕；</p>
</li>
<li data-nodeid="1906">
<p data-nodeid="1907">如果某一个请求在队列中处于等待状态超过 2000 毫秒，则直接抛出 BlockException。</p>
</li>
</ol>
<p data-nodeid="1908">注意，匀速队列只支持 QPS 模式，且单机阈值不得大于 1000。</p>
<p data-nodeid="15144" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar8mAA3X_AAFbifYRYio573.png" alt="图片13.png" data-nodeid="15148"></p>
<div data-nodeid="15145"><p style="text-align:center">流控效果-排队等待</p></div>


<p data-nodeid="1911">讲到这，我为你讲解了从滑动窗口统计流量到 Sentinel Dashboard 如何进行流控配置。下面咱们再来讲解 Sentinel的熔断降级策略。</p>
<h3 data-nodeid="1912">Sentinel 熔断降级的规则配置</h3>
<h4 data-nodeid="1913">什么是熔断？</h4>
<p data-nodeid="1914">先说现实中的股市熔断机制。2016 年 1 月 4 日，A 股遇到史上首次熔断，沪指开盘后跌幅超过 5%，直接引发熔断。三家股市交易所暂停交易15分钟，但恢复交易之后股市继续下跌，三大股市交易所暂停交易至闭市。通过现象可以看<strong data-nodeid="2161">出熔断是一种保护机制</strong>，当事物的状态达到某种“不可接受”的情况时，便会触发“熔断”。在股市中，熔断条件就是大盘跌幅超过 5%，而熔断的措施便是强制停止交易 15 分钟，之后尝试恢复交易，如仍出现继续下跌，便会再次触发熔断直接闭市。但假设 15分钟后，大盘出现回涨，便认为事故解除继续正常交易。这是现实生活中的熔断，如果放在软件中也是一样的。</p>
<p data-nodeid="1915">微服务的熔断是指在某个服务接口在执行过程中频繁出现故障的情况，我们便认为这种状态是“不可接受”的，立即对当前接口实施熔断。在规定的时间内，所有送达该接口的请求都将直接抛出 BlockException，在熔断期过后新的请求进入看接口是否恢复正常，恢复正常则继续运行，仍出现故障则再次熔断一段时间，以此往复直到服务接口恢复。</p>
<p data-nodeid="1916">下图清晰的说明了 Sentinel的熔断过程：</p>
<ol data-nodeid="1917">
<li data-nodeid="1918">
<p data-nodeid="1919">设置熔断的触发条件，当某接口超过20%的请求访问出现故障，便启动熔断；</p>
</li>
<li data-nodeid="1920">
<p data-nodeid="1921">在熔断状态下，若干秒内所有该接口的请求访问都会直接抛出BlockException拒绝访问。</p>
</li>
<li data-nodeid="1922">
<p data-nodeid="1923">熔断器过后，下一次请求重新访问接口，当前接口为“半开状态”，后续处理以下分两种情况。</p>
</li>
</ol>
<ul data-nodeid="1924">
<li data-nodeid="1925">
<p data-nodeid="1926">当前请求被有效处理，接口恢复到正常状态。</p>
</li>
<li data-nodeid="1927">
<p data-nodeid="1928">当前请求访问出现故障，接口继续熔断。</p>
</li>
</ul>
<p data-nodeid="16005" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar9OAT7iIAADpMfE3-dw738.png" alt="图片14.png" data-nodeid="16009"></p>
<div data-nodeid="16006"><p style="text-align:center">Sentinel 熔断机制</p></div>


<h4 data-nodeid="1931"><strong data-nodeid="2175">基于SentinelDashboard的熔断设置</strong></h4>
<p data-nodeid="1932">Sentinel Dashboard可以设置三种不同的熔断模式：慢调用比例、异常比例、异常数，下面我们分别讲解：</p>
<ul data-nodeid="1933">
<li data-nodeid="1934">
<p data-nodeid="1935">慢调用比例是指当接口在1秒内“慢处理”数量超过一定比例，则触发熔断。</p>
</li>
</ul>
<p data-nodeid="16866" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar92AAT0jAAHYZ6NDKjQ113.png" alt="图片15.png" data-nodeid="16870"></p>
<div data-nodeid="16867"><p style="text-align:center">熔断模式-慢调用比例</p></div>


<p data-nodeid="1938">结合上图的设置，介绍下“慢调用比例”熔断规则。</p>
<p data-nodeid="17727" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/25/CioPOWBar-qACeUjAALQjoYzEvE265.png" alt="图片18.png" data-nodeid="17730"></p>

<ul data-nodeid="1952">
<li data-nodeid="1953">
<p data-nodeid="1954">异常比例是指 1 秒内按接口调用产生异常的比例（异常调用数/总数量）触发熔断。</p>
</li>
</ul>
<p data-nodeid="18561" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/29/Cgp9HWBar_SAOTw9AAFxRASYnnE809.png" alt="图片16.png" data-nodeid="18565"></p>
<div data-nodeid="18562"><p style="text-align:center">熔断模式-异常比例</p></div>


<p data-nodeid="1957">结合上图的设置，介绍下“异常比例”熔断规则。</p>
<p data-nodeid="19396" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/26/CioPOWBar_6AXJmYAAK3pfImZs4903.png" alt="图片19.png" data-nodeid="19399"></p>

<ul data-nodeid="1971">
<li data-nodeid="1972">
<p data-nodeid="1973">异常数是指在 1 分钟内异常的数量超过阈值则触发熔断。</p>
</li>
</ul>
<p data-nodeid="20204" class=""><img src="https://s0.lgstatic.com/i/image6/M01/26/29/Cgp9HWBasAmAO9LnAAFiqbTOxTs071.png" alt="图片17.png" data-nodeid="20208"></p>
<div data-nodeid="20205"><p style="text-align:center">熔断模式-异常数</p></div>


<p data-nodeid="1976">结合上图的设置，介绍下“异常数”熔断规则。</p>
<p data-nodeid="21013" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/26/26/CioPOWBasBOAY3wPAALTmvi2q7s202.png" alt="图片20.png" data-nodeid="21016"></p>

<p data-nodeid="1990">以上就是三种熔断模式的介绍，熔断相对流控配置比较简单，只需要设置熔断检查开启条件与触发熔断条件即可。讲到这关于限流与熔断的配置暂时告一段落，下面对本讲内容进行下总结。</p>
<h3 data-nodeid="1991">小结与预告</h3>
<p data-nodeid="1992">本讲咱们介绍了三部分内容，第一部分讲解了 Sentinel Dashboard 与 Sentinel Core的通信机制与执行原理，了解 Sentinel Core 是通过拦截器实现了限流与熔断；第二部分讲解了滑动窗口算法与 Dashboard 流控配置的每一种情况；第三部分讲解了熔断机制与 Dashboard 的熔断配置。</p>
<p data-nodeid="1993">这里留一个讨论话题：假如你遇到像春运 12306 热门车次购票这种高并发场景，为了保证应用的稳定和用户的体验，我们要采取哪些措施呢？可以把你的经验写在评论中，咱们一起探讨。</p>
<p data-nodeid="1994" class="">下一讲，将进入生产集群保护这个话题，看 Sentinel 是如何对整个服务集群实施保护的。</p>

---

### 精选评论

##### **筑：
> 假如你遇到像春运 12306 热门车次购票这种高并发场景，为了保证应用的稳定和用户的体验，可采用消息队列、熔断、降级等策略

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没错，Sentinel就是干这事的。MQ也有类似的作用

##### *超：
> 讲熔断就讲熔断，为啥要扯上股票？伤心得不行😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没事，兄弟，我这也一片绿油油~~~不说啦，工头又喊我搬砖了

##### **伟：
> 那我们的应该依据什么来设置这些限流和熔断的参数呢？如果规避因为设置不合理导致不该限流、熔断的却进行了限流熔断呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个是需要评估的，但也有通行的策略。首先对服务进行压测，了解服务的性能指标。
开始限流熔断条件放宽一些，观察系统负载情况，之后不断收窄，保证服务器负载基线不超过60%即可

##### Null：
> 同时每过 10 秒，滑动窗口向右移动，前面的过期时间段计数器将被作废。按照这个逻辑，窗口永远都不会满的，每过10秒就滑动，窗口永远都是空的，因为窗口第一个时间段也是10秒。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不是这样的，https://blog.csdn.net/king0406/article/details/103129786
细节你可以参考下这个Java实现

