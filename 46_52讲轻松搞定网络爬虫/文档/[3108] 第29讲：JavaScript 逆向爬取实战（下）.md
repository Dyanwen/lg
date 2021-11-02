<h3 data-nodeid="33112" class="">详情页加密 id 入口的寻找</h3>
<p data-nodeid="33113">好，我们接着上一课时的内容往下讲，我们观察下上一步的输出结果，我们把结果格式化一下，看看部分结果：</p>
<pre class="lang-js" data-nodeid="33114"><code data-language="js">{
 <span class="hljs-string">'count'</span>: <span class="hljs-number">100</span>,
 <span class="hljs-string">'results'</span>: [
  {
     <span class="hljs-string">'id'</span>: <span class="hljs-number">1</span>,
     <span class="hljs-string">'name'</span>: <span class="hljs-string">'霸王别姬'</span>,
     <span class="hljs-string">'alias'</span>: <span class="hljs-string">'Farewell My Concubine'</span>,
     <span class="hljs-string">'cover'</span>: <span class="hljs-string">'https://p0.meituan.net/movie/ce4da3e03e655b5b88ed31b5cd7896cf62472.jpg@464w_644h_1e_1c'</span>,
     <span class="hljs-string">'categories'</span>: [
       <span class="hljs-string">'剧情'</span>,
       <span class="hljs-string">'爱情'</span>
    ],
     <span class="hljs-string">'published_at'</span>: <span class="hljs-string">'1993-07-26'</span>,
     <span class="hljs-string">'minute'</span>: <span class="hljs-number">171</span>,
     <span class="hljs-string">'score'</span>: <span class="hljs-number">9.5</span>,
     <span class="hljs-string">'regions'</span>: [
       <span class="hljs-string">'中国大陆'</span>,
       <span class="hljs-string">'中国香港'</span>
    ]
  },
   ...
]
}
</code></pre>
<p data-nodeid="33115">这里我们看到有个 id 是 1，另外还有一些其他的字段如电影名称、封面、类别，等等，那么这里面一定有什么信息是用来唯一区分某个电影的。</p>
<p data-nodeid="34099" class="">但是呢，这里我们点击下第一个部电影的信息，可以看到它跳转到了 URL 为 <a href="https://dynamic6.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx" data-nodeid="34103">https://dynamic6.scrape.center/detail/ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx</a> 的页面，可以看到这里 URL 里面有一个加密 id 为 ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx，那么这个和电影的这些信息有什么关系呢？</p>


<p data-nodeid="33117">这里，如果你仔细观察其实是可以比较容易地找出规律来的，但是这总归是观察出来的，如果遇到一些观察不出规律的那就不好处理了。所以还是需要靠技巧去找到它真正加密的位置。</p>
<p data-nodeid="33118">这时候我们该怎么办呢？首先为我们分析一下，这个加密 id 到底是什么生成的。</p>
<p data-nodeid="33119">我们在点击详情页的时候就看到它访问的 URL 里面就带上了 ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx 这个加密 id 了，而且不同的详情页的加密 id 是不同的，这说明这个加密 id 的构造依赖于列表页 Ajax 的返回结果，所以可以确定这个加密 id 的生成是发生在 Ajax 请求完成后或者点击详情页的一瞬间。</p>
<p data-nodeid="33120">为了进一步确定是发生在何时，我们看看页面源码，可以看到在没有点击之前，详情页链接的 href 里面就已经带有加密 id 了，如图所示。</p>
<p data-nodeid="33121"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/CgqCHl6qlJCAdQd6AAk_ZX1vrAI092.png" alt="image (25).png" data-nodeid="33263"></p>
<p data-nodeid="33122">由此我们可以确定，这个加密 id 是在 Ajax 请求完成之后生成的，而且肯定也是由 JavaScript 生成的了。</p>
<p data-nodeid="33123">那怎么再去查找 Ajax 完成之后的事件呢？是否应该去找 Ajax 完成之后的事件呢？<br>
可以是可以，你可以试试，我们可以看到在 Sources 面板的右侧，有一个 Event Listener Breakpoints，这里有一个 XHR 的监听，包括发起时、成功后、发生错误时的一些监听，这里我们勾选上 readystatechange 事件，代表 Ajax 得到响应时的事件，其他的断点可以都删除了，然后刷新下页面看下，如图所示。</p>
<p data-nodeid="33124"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlKSAQ_UFAAh49gbNW9s441.png" alt="image (26).png" data-nodeid="33270"></p>
<p data-nodeid="33125">这里我们可以看到就停在了 Ajax 得到响应时的位置了。</p>
<p data-nodeid="33126">那我们怎么才能弄清楚这个 id 是怎么加密的呢？可以选择一个断点一个断点地找下去，但估计找的过程会崩溃掉，因为这里可能会逐渐调用到页面 UI 渲染的一些底层实现，甚至可能即使找到了也不知道具体找到哪里去了。</p>
<p data-nodeid="33127">那怎么办呢？这里我们再介绍一种定位的方法，那就是 Hook。</p>
<p data-nodeid="33128">Hook 技术中文又叫作钩子技术，它就是在程序运行的过程中，对其中的某个方法进行重写，在原有的方法前后加入我们自定义的代码。相当于在系统没有调用该函数之前，钩子程序就先捕获该消息，可以先得到控制权，这时钩子函数便可以加工处理（改变）该函数的执行行为。</p>
<p data-nodeid="33129">通俗点来说呢，比如我要 Hook 一个方法 a，可以先临时用一个变量存一下，把它存成 _a，然后呢，我再重新声明一个方法 a，里面添加自己的逻辑，比如加点调试语句、输出语句等等，然后再调用 _a，这里调用的 _a 就是之前的 a。</p>
<p data-nodeid="33130">这样就相当于新的方法 a 里面混入了我们自己定义的逻辑，同时又把原来的方法 a 也执行了一遍。所以这不会影响原有的执行逻辑和运行效果，但是我们通过这种改写便可以顺利在原来的 a 方法前后加上了我们自己的逻辑，这就是 Hook。</p>
<p data-nodeid="33131">那么，我们这里怎么用 Hook 的方式来找到加密 id 的加密入口点呢？</p>
<p data-nodeid="33132">想一下，这个加密 id 是一个 Base64 编码的字符串，那么生成过程中想必就调用了 JavaScript 的 Base64 编码的方法，这个方法名叫作 btoa，这个 btoa 方法可以将参数转化成 Base64 编码。当然 Base64 也有其他的实现方式，比如利用 crypto-js 这个库实现的，这个可能底层调用的就不是 btoa 方法了。</p>
<p data-nodeid="33133">所以，其实现在并不确定是不是调用的 btoa 方法实现的 Base64 编码，那就先试试吧。要实现 Hook，其实关键在于将原来的方法改写，这里我们其实就是 Hook btoa 这个方法了，btoa 这个方法属于 window 对象，我们将 window 对象的 btoa 方法进行改写即可。</p>
<p data-nodeid="33134">改写的逻辑如下：</p>
<pre class="lang-java" data-nodeid="33135"><code data-language="java">(function () {
   <span class="hljs-string">'use strict'</span>
   <span class="hljs-function">function <span class="hljs-title">hook</span><span class="hljs-params">(object, attr)</span> </span>{
       <span class="hljs-keyword">var</span> func = object[attr]
       object[attr] = function () {
           console.log(<span class="hljs-string">'hooked'</span>, object, attr, arguments)
           <span class="hljs-keyword">var</span> ret = func.apply(object, arguments)
           debugger
           console.log(<span class="hljs-string">'result'</span>, ret)
           <span class="hljs-keyword">return</span> ret
      }
  }
   hook(window, <span class="hljs-string">'btoa'</span>)
})()
</code></pre>
<p data-nodeid="33136">我们定义了一个 hook 方法，传入 object 和 attr 参数，意思就是 Hook object 对象的 attr 参数。例如我们如果想 Hook 一个 alert 方法，那就把 object 设置为 window，把 attr 设置为 alert 字符串。这里我们想要 Hook Base64 的编码方法，那么这里就只需要 Hook window 对象的 btoa 方法就好了。</p>
<p data-nodeid="33137">我们来看下，首先是 var func = object[attr]，相当于先把它赋值为一个变量，我们调用 func 方法就可以实现和原来相同的功能。接着，我们再直接改写这个方法的定义，直接改写 object[attr]，将其改写成一个新的方法，在新的方法中，通过 func.apply 方法又重新调用了原来的方法。</p>
<p data-nodeid="33138">这样我们就可以保证，前后方法的执行效果是不受什么影响的，之前这个方法该干啥就还是干啥的。但是和之前不同的是，我们自定义方法之后，现在可以在 func 方法执行的前后，再加入自己的代码，如 console.log 将信息输出到控制台，如 debugger 进入断点等等。</p>
<p data-nodeid="33139">这个过程中，我们先临时保存下来了 func 方法，然后定义一个新的方法，接管程序控制权，在其中自定义我们想要的实现，同时在新的方法里面再重新调回 func 方法，保证前后结果是不受影响的。所以，我们达到了在不影响原有方法效果的前提下，可以实现在方法的前后实现自定义的功能，就是 Hook 的完整实现过程。</p>
<p data-nodeid="33140">最后，我们调用 hook 方法，传入 window 对象和 btoa 字符串即可。</p>
<p data-nodeid="33141">那这样，怎么去注入这个代码呢？这里我们介绍三种注入方法。</p>
<ul data-nodeid="33142">
<li data-nodeid="33143">
<p data-nodeid="33144">直接控制台注入；</p>
</li>
<li data-nodeid="33145">
<p data-nodeid="33146">复写 JavaScript 代码；</p>
</li>
<li data-nodeid="33147">
<p data-nodeid="33148">Tampermonkey 注入。</p>
</li>
</ul>
<h4 data-nodeid="33149">控制台注入</h4>
<p data-nodeid="33150">对于我们这个场景，控制台注入其实就够了，我们先来介绍这个方法。</p>
<p data-nodeid="33151">其实控制台注入很简单，就是直接在控制台输入这行代码运行，如图所示。</p>
<p data-nodeid="33152"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlPqALm_6AAY-ro6hjwQ572.png" alt="image (27).png" data-nodeid="33309"></p>
<p data-nodeid="33153">执行完这段代码之后，相当于我们就已经把 window 的 btoa 方法改写了，可以控制台调用下 btoa 方法试试，如：</p>
<pre class="lang-js" data-nodeid="33154"><code data-language="js">btoa(<span class="hljs-string">'germey'</span>)
</code></pre>
<p data-nodeid="33155">回车之后就可以看到它进入了我们自定义的 debugger 的位置停下了，如图所示。</p>
<p data-nodeid="33156"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlQiALo85AAgcGmKilPk416.png" alt="image (28).png" data-nodeid="33314"></p>
<p data-nodeid="33157">我们把断点向下执行，点击 Resume 按钮，然后看看控制台的输出，可以看到也输出了一些对应的结果，如被 Hook 的对象，Hook 的属性，调用的参数，调用后的结果等，如图所示。</p>
<p data-nodeid="33158"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlRSAdfGaAAacrG5N8q4214.png" alt="image (29).png" data-nodeid="33318"></p>
<p data-nodeid="33159">这里我们就可以看到，我们通过 Hook 的方式改写了 btoa 方法，使其每次在调用的时候都能停到一个断点，同时还能输出对应的结果。</p>
<p data-nodeid="33160">接下来我们看下怎么用 Hook 找到对应的加密 id 的加密入口？</p>
<p data-nodeid="33161">由于此时我们是在控制台直接输入的 Hook 代码，所以页面一旦刷新就无效了，但由于我们这个网站是 SPA 式的页面，所以在点击详情页的时候页面是不会整个刷新的，所以这段代码依然还会生效。但是如果不是 SPA 式的页面，即每次访问都需要刷新页面的网站，这种注入方式就不生效了。</p>
<p data-nodeid="33162">好，那我们的目的是为了 Hook 列表页 Ajax 加载完成后的加密 id 的 Base64 编码的过程，那怎么在不刷新页面的情况下再次复现这个操作呢？很简单，点下一页就好了。</p>
<p data-nodeid="33163">这时候我们可以点击第 2 页的按钮，可以看到它确实再次停到了 Hook 方法的 debugger 处，由于列表页的 Ajax 和加密 id 都会带有 Base64 编码的操作，因此它每一个都能 Hook 到，通过观察对应的 Arguments 或当前网站的行为或者观察栈信息，我们就能大体知道现在走到了哪个位置了，从而进一步通过栈的调用信息找到调用 Base64 编码的位置。</p>
<p data-nodeid="33164">我们可以根据调用栈的信息来观察这些变量是在哪一层发生变化的，比如最后的这一层，我们可以很明显看到它执行了 Base64 编码，编码前的结果是：</p>
<pre class="lang-java" data-nodeid="33165"><code data-language="java">ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs\*-!i-0-mb1
</code></pre>
<p data-nodeid="33166">编码后的结果是：</p>
<pre class="lang-java" data-nodeid="33167"><code data-language="java">ZWYzNCN0ZXVxMGJ0dWEjKC01N3cxcTVvNS0takA5OHh5Z2ltbHlmeHMqLSFpLTAtbWIx
</code></pre>
<p data-nodeid="33168">如图所示。</p>
<p data-nodeid="33169"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlTOAXTlNAAbm2aEXYuA791.png" alt="image (30).png" data-nodeid="33329"></p>
<p data-nodeid="33170">这里很明显。</p>
<p data-nodeid="33171">那么核心问题就来了，编码前的结果 ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs*-!i-0-mb1又是怎么来的呢？我们展开栈的调用信息，一层层看看这个字符串的变化情况。如果不变那就看下一层，如果变了那就停下来仔细看看。</p>
<p data-nodeid="33172">最后我们可以在第五层找到它的变化过程，如图所示。</p>
<p data-nodeid="33173"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlVGAO62xAAmWfTuXTnI776.png" alt="image (31).png" data-nodeid="33339"></p>
<p data-nodeid="33174">那这里我们就一目了然了，看到了 _0x135c4d 是一个写死的字符串 ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs*-!i-0-mb，然后和传入的这个 _0x565f18 拼接起来就形成了最后的字符串。</p>
<p data-nodeid="33175">那这个 _0x565f18 又是怎么来的呢？再往下追一层，那就一目了然了，其实就是 Ajax 返回结果的单个电影信息的 id。</p>
<p data-nodeid="33176">所以，这个加密逻辑的就清楚了，其实非常简单，就是 ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs*-!i-0-mb1 加上电影 id，然后 Base64 编码即可。</p>
<p data-nodeid="33177">到此，我们就成功用 Hook 的方式找到加密的 id 生成逻辑了。</p>
<p data-nodeid="33178">但是想想有什么不太科学的地方吗？刚才其实也说了，我们的 Hook 代码是在控制台手动输入的，一旦刷新页面就不生效了，这的确是个问题。而且它必须是在页面加载完了才注入的，所以它并不能在一开始就生效。</p>
<p data-nodeid="33179">下面我们再介绍几种 Hook 注入方式</p>
<h4 data-nodeid="33180">重写 JavaScript</h4>
<p data-nodeid="33181">我们可以借助于 Chrome 浏览器的 Overrides 功能实现某些 JavaScript 文件的重写和保存，它会在本地生成一个 JavaScript 文件副本，以后每次刷新的时候会使用副本的内容。</p>
<p data-nodeid="33182">这里我们需要切换到 Sources 选项卡的 Overrides 选项卡，然后选择一个文件夹，比如这里我自定了一个文件夹名字叫作 modify，如图所示。</p>
<p data-nodeid="33183"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/Ciqc1F6qlX6ADDfeAAZ2FWeRVsQ406.png" alt="image (32).png" data-nodeid="33365"></p>
<p data-nodeid="33184">然后我们随便选一个 JavaScript 脚本，后面贴上这段注入脚本，如图所示。</p>
<p data-nodeid="33185"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/CgqCHl6qlYmAK5yrAAh8B0rKWe4441.png" alt="image (33).png" data-nodeid="33369"></p>
<p data-nodeid="33186">保存文件。此时可能提示页面崩溃，但是不用担心，重新刷新页面就好了，这时候我们就发现现在浏览器加载的 JavaScript 文件就是我们修改过后的了，文件的下方会有一个标识符，如图所示。</p>
<p data-nodeid="33187"><img src="https://s0.lgstatic.com/i/image/M00/00/DF/CgqCHl6qlZSADgU4AAZ9p5grU3A458.png" alt="image (34).png" data-nodeid="33373"></p>
<p data-nodeid="33188">同时我们还注意到这时候它就直接进入了断点模式，成功 Hook 到了 btoa 这个方法了。其实 Overrides 的这个功能非常有用，有了它我们可以持久化保存我们任意修改的 JavaScript 代码，所以我们想在哪里改都可以了，甚至可以直接修改 JavaScript 的原始执行逻辑也都是可以的。</p>
<h4 data-nodeid="33189">Tampermonkey 注入</h4>
<p data-nodeid="33190">如果我们不想用 Overrides 的方式改写 JavaScript 的方式注入的话，还可以借助于浏览器插件来实现注入，这里推荐的浏览器插件叫作 Tampermonkey，中文叫作“油猴”。它是一款浏览器插件，支持 Chrome。利用它我们可以在浏览器加载页面时自动执行某些 JavaScript 脚本。由于执行的是 JavaScript，所以我们几乎可以在网页中完成任何我们想实现的效果，如自动爬虫、自动修改页面、自动响应事件等等。</p>
<p data-nodeid="33191">首先我们需要安装 Tampermonkey，这里我们使用的浏览器是 Chrome。直接在 Chrome 应用商店或者在 Tampermonkey 的官网 <a href="https://www.tampermonkey.net/" data-nodeid="33380">https://www.tampermonkey.net/</a> 下载安装即可。</p>
<p data-nodeid="33192">安装完成之后，在 Chrome 浏览器的右上角会出现 Tampermonkey 的图标，这就代表安装成功了。</p>
<p data-nodeid="33193"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/Ciqc1F6qlciAUhX0AAARZPlxmoI615.png" alt="image (35).png" data-nodeid="33385"></p>
<p data-nodeid="33194">我们也可以自己编写脚本来实现想要的功能。编写脚本难不难呢？其实就是写 JavaScript 代码，只要懂一些 JavaScript 的语法就好了。另外除了懂 JavaScript 语法，我们还需要遵循脚本的一些写作规范，这其中就包括一些参数的设置。</p>
<p data-nodeid="33195">下面我们就简单实现一个小的脚本，实现某个功能。</p>
<p data-nodeid="33196">首先我们可以点击 Tampermonkey 插件图标，点击“管理面板”按钮，打开脚本管理页面。</p>
<p data-nodeid="33197"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/CgqCHl6qldmABWZBAABrf5xHC2s424.png" alt="image (36).png" data-nodeid="33391"></p>
<p data-nodeid="33198">界面类似显示如下图所示。</p>
<p data-nodeid="33199"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/Ciqc1F6qleOAMwp5AAC3g20XxJI627.png" alt="image (37).png" data-nodeid="33395"></p>
<p data-nodeid="33200">在这里显示了我们已经有的一些 Tampermonkey 脚本，包括我们自行创建的，也包括从第三方网站下载安装的。</p>
<p data-nodeid="33201">另外这里也提供了编辑、调试、删除等管理功能，我们可以方便地对脚本进行管理。接下来我们来创建一个新的脚本来试试，点击左侧的“+”号，会显示如图所示的页面。</p>
<p data-nodeid="33202"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/CgqCHl6qlfeAR-XCAAFU2XQaKcs123.png" alt="image (38).png" data-nodeid="33400"></p>
<p data-nodeid="33203">初始化的代码如下：</p>
<pre class="lang-js" data-nodeid="33204"><code data-language="js"><span class="hljs-comment">// ==UserScript==</span>
<span class="hljs-comment">// @name         New Userscript</span>
<span class="hljs-comment">// @namespace   http://tampermonkey.net/</span>
<span class="hljs-comment">// @version     0.1</span>
<span class="hljs-comment">// @description try to take over the world!</span>
<span class="hljs-comment">// @author       You</span>
<span class="hljs-comment">// @match       https://www.tampermonkey.net/documentation.php?ext=dhdg</span>
<span class="hljs-comment">// @grant       none</span>
<span class="hljs-comment">// ==/UserScript==</span>

(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
   <span class="hljs-string">'use strict'</span>;

   <span class="hljs-comment">// Your code here...</span>
})();
</code></pre>
<p data-nodeid="33205">这里最上面是一些注释，但这些注释是非常有用的，这部分内容叫作 UserScript Header ，我们可以在里面配置一些脚本的信息，如名称、版本、描述、生效站点等等。</p>
<p data-nodeid="33206">在 UserScript Header 下方是 JavaScript 函数和调用的代码，其中 use strict 标明代码使用 JavaScript 的严格模式，在严格模式下可以消除 Javascript 语法的一些不合理、不严谨之处，减少一些怪异行为，如不能直接使用未声明的变量，这样可以保证代码的运行安全，同时提高编译器的效率，提高运行速度。在下方 // Your code here... 这里我们就可以编写自己的代码了。</p>
<p data-nodeid="33207">我们可以将脚本改写为如下内容：</p>
<pre class="lang-js" data-nodeid="35414"><code data-language="js"><span class="hljs-comment">// ==UserScript==</span>
<span class="hljs-comment">// @name         HookBase64</span>
<span class="hljs-comment">// @namespace   https://scrape.center/</span>
<span class="hljs-comment">// @version     0.1</span>
<span class="hljs-comment">// @description Hook Base64 encode function</span>
<span class="hljs-comment">// @author       Germey</span>
<span class="hljs-comment">// @match       https://dynamic6.scrape.center/</span>
<span class="hljs-comment">// @grant       none</span>
<span class="hljs-comment">// @run-at     document-start</span>
<span class="hljs-comment">// ==/UserScript==</span>
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
   <span class="hljs-string">'use strict'</span>
   <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">hook</span>(<span class="hljs-params">object, attr</span>) </span>{
       <span class="hljs-keyword">var</span> func = object[attr]
       <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'func'</span>, func)
       object[attr] = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
           <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'hooked'</span>, object, attr)
           <span class="hljs-keyword">var</span> ret = func.apply(object, <span class="hljs-built_in">arguments</span>)
           <span class="hljs-keyword">debugger</span>
           <span class="hljs-keyword">return</span> ret
      }
  }
   hook(<span class="hljs-built_in">window</span>, <span class="hljs-string">'btoa'</span>)
})()
</code></pre>


<p data-nodeid="33209">这时候启动脚本，重新刷新页面，可以发现也可以成功 Hook 住 btoa 方法，如图所示。</p>
<p data-nodeid="33210"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/Ciqc1F6qlkWAXng9AAXpKOp2pIg630.png" alt="image (39).png" data-nodeid="33408"></p>
<p data-nodeid="33211">然后我们再顺着找调用逻辑就好啦。</p>
<p data-nodeid="33212">以上，我们就成功通过 Hook 的方式找到加密 id 的实现了。</p>
<h3 data-nodeid="33213">详情页 Ajax 的 token 寻找</h3>
<p data-nodeid="33214">现在我们已经找到详情页的加密 id 了，但是还差一步，其 Ajax 请求也有一个 token，如图所示。</p>
<p data-nodeid="33215"><img src="https://s0.lgstatic.com/i/image/M00/00/E0/CgqCHl6qllaAX0VEAAcFtYFfiyc075.png" alt="image (40).png" data-nodeid="33415"></p>
<p data-nodeid="33216">其实这个 token 和详情页的 token 构造逻辑是一样的了。</p>
<p data-nodeid="33217">这里就不再展开说了，可以运用上文的几种找入口的方法来找到对应的加密逻辑。</p>
<h3 data-nodeid="33218">Python 实现详情页爬取</h3>
<p data-nodeid="33219">现在我们已经成功把详情页的加密 id 和 Ajax 请求的 token 找出来了，下一步就能使用 Python 完成爬取了，这里我就只实现第一页的爬取了，代码示例如下：</p>
<pre class="lang-js te-preview-highlight" data-nodeid="36724"><code data-language="js"><span class="hljs-keyword">import</span> hashlib
<span class="hljs-keyword">import</span> time
<span class="hljs-keyword">import</span> base64
<span class="hljs-keyword">from</span> typing <span class="hljs-keyword">import</span> List, Any
<span class="hljs-keyword">import</span> requests

INDEX_URL = <span class="hljs-string">'https://dynamic6.scrape.center/api/movie?limit={limit}&amp;offset={offset}&amp;token={token}'</span>
DETAIL_URL = <span class="hljs-string">'https://dynamic6.scrape.center/api/movie/{id}?token={token}'</span>
LIMIT = <span class="hljs-number">10</span>
OFFSET = <span class="hljs-number">0</span>
SECRET = <span class="hljs-string">'ef34#teuq0btua#(-57w1q5o5--j@98xygimlyfxs*-!i-0-mb'</span>

def get_token(args: List[Any]):
   timestamp = str(int(time.time()))
   args.append(timestamp)
   sign = hashlib.sha1(<span class="hljs-string">','</span>.join(args).encode(<span class="hljs-string">'utf-8'</span>)).hexdigest()
   <span class="hljs-keyword">return</span> base64.b64encode(<span class="hljs-string">','</span>.join([sign, timestamp]).encode(<span class="hljs-string">'utf-8'</span>)).decode(<span class="hljs-string">'utf-8'</span>) 

args = [<span class="hljs-string">'/api/movie'</span>]
token = get_token(args=args)
index_url = INDEX_URL.format(limit=LIMIT, offset=OFFSET, token=token)
response = requests.get(index_url)
print(<span class="hljs-string">'response'</span>, response.json())

result = response.json()
<span class="hljs-keyword">for</span> item <span class="hljs-keyword">in</span> result[<span class="hljs-string">'results'</span>]:
   id = item[<span class="hljs-string">'id'</span>]
   encrypt_id = base64.b64encode((SECRET + str(id)).encode(<span class="hljs-string">'utf-8'</span>)).decode(<span class="hljs-string">'utf-8'</span>)
   args = [f<span class="hljs-string">'/api/movie/{encrypt_id}'</span>]
   token = get_token(args=args)
   detail_url = DETAIL_URL.format(id=encrypt_id, token=token)
   response = requests.get(detail_url)
   print(<span class="hljs-string">'response'</span>, response.json())
</code></pre>


<p data-nodeid="33221">这里模拟了详情页的加密 id 和 token 的构造过程，然后请求了详情页的 Ajax 接口，这样我们就可以爬取到详情页的内容了。</p>
<h3 data-nodeid="33222">总结</h3>
<p data-nodeid="33223">本课时内容很多，一步步介绍了整个网站的 JavaScript 逆向过程，其中的技巧有：</p>
<ul data-nodeid="33224">
<li data-nodeid="33225">
<p data-nodeid="33226">全局搜索查找入口</p>
</li>
<li data-nodeid="33227">
<p data-nodeid="33228">代码格式化</p>
</li>
<li data-nodeid="33229">
<p data-nodeid="33230">XHR 断点</p>
</li>
<li data-nodeid="33231">
<p data-nodeid="33232">变量监听</p>
</li>
<li data-nodeid="33233">
<p data-nodeid="33234">断点设置和跳过</p>
</li>
<li data-nodeid="33235">
<p data-nodeid="33236">栈查看</p>
</li>
<li data-nodeid="33237">
<p data-nodeid="33238">Hook 原理</p>
</li>
<li data-nodeid="33239">
<p data-nodeid="33240">Hook 注入</p>
</li>
<li data-nodeid="33241">
<p data-nodeid="33242">Overrides 功能</p>
</li>
<li data-nodeid="33243">
<p data-nodeid="33244">Tampermonkey 插件</p>
</li>
<li data-nodeid="33245">
<p data-nodeid="33246">Python 模拟实现</p>
</li>
</ul>
<p data-nodeid="33247">掌握了这些技巧我们就能更加得心应手地实现 JavaScript 逆向分析。</p>
<p data-nodeid="33248" class="">本节代码：<a href="https://github.com/Python3WebSpider/ScrapeDynamic6" data-nodeid="33438">https://github.com/Python3WebSpider/ScrapeDynamic6</a></p>

---

### 精选评论

##### **兵：
> 崔老师 你这两讲真的太牛逼了，真的很实在的干货，我们以前在处理某网站的简历的时候 也遇到一些id&nbsp; 的问题 长知识了 ，希望崔老师能够加一些selenium被后台检测后的反反爬技术

##### **鹏：
> 这个是真的肝，我的小心脏啊😇

##### **兵：
> 老师更新课程了

##### **4717：
> 学到点皮毛了

