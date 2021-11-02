<p>我们在爬取网站的时候，经常会遇到各种各样类似加密的情形，比如：</p>
<ul>
<li>某个网站的 URL 带有一些看不懂的长串加密参数，想要抓取就必须要懂得这些参数是怎么构造的，否则我们连完整的 URL 都构造不出来，更不用说爬取了。</li>
<li>分析某个网站的 Ajax 接口的时候，可以看到接口的一些参数也是加密的，或者 Request Headers 里面也可能带有一些加密参数，如果不知道这些参数的具体构造逻辑就无法直接用程序来模拟这些 Ajax 请求。</li>
<li>翻看网站的 JavaScript 源代码，可以发现很多压缩了或者看不太懂的字符，比如 JavaScript 文件名被编码，JavaScript 的文件内容被压缩成几行，JavaScript 变量也被修改成单个字符或者一些十六进制的字符，导致我们不好轻易根据 JavaScript 找出某些接口的加密逻辑。</li>
</ul>
<p>这些情况，基本上都是网站为了保护其本身的一些数据不被轻易抓取而采取的一些措施，我们可以把它归为两大类：</p>
<ul>
<li>接口加密技术；</li>
<li>JavaScript 压缩、混淆和加密技术。</li>
</ul>
<p>本课时我们就来了解下这两类技术的实现原理。</p>
<h3>数据保护</h3>
<p>当今大数据时代，数据已经变得越来越重要，网页和 App 现在是主流的数据载体，如果其数据的接口没有设置任何保护措施，在爬虫工程师解决了一些基本的反爬如封 IP、验证码的问题之后，那么数据还是可以被轻松抓取到。</p>
<p>那么，有没有可能在接口或 JavaScript 层面也加上一层防护呢？答案是可以的。</p>
<h4>接口加密技术</h4>
<p>网站运营商首先想到防护措施可能是对某些数据接口进行加密，比如说对某些 URL 的一些参数加上校验码或者把一些 ID 信息进行编码，使其变得难以阅读或构造；或者对某些接口请求加上一些 token、sign 等签名，这样这些请求发送到服务器时，服务器会通过客户端发来的一些请求信息以及双方约定好的秘钥等来对当前的请求进行校验，如果校验通过，才返回对应数据结果。</p>
<p>比如说客户端和服务端约定一种接口校验逻辑，客户端在每次请求服务端接口的时候都会附带一个 sign 参数，这个 sign 参数可能是由当前时间信息、请求的 URL、请求的数据、设备的 ID、双方约定好的秘钥经过一些加密算法构造而成的，客户端会实现这个加密算法构造 sign，然后每次请求服务器的时候附带上这个参数。服务端会根据约定好的算法和请求的数据对 sign 进行校验，如果校验通过，才返回对应的数据，否则拒绝响应。</p>
<h4>JavaScript 压缩、混淆和加密技术</h4>
<p>接口加密技术看起来的确是一个不错的解决方案，但单纯依靠它并不能很好地解决问题。为什么呢？</p>
<p>对于网页来说，其逻辑是依赖于 JavaScript 来实现的，JavaScript 有如下特点：</p>
<ul>
<li>JavaScript 代码运行于客户端，也就是它必须要在用户浏览器端加载并运行。</li>
<li>JavaScript 代码是公开透明的，也就是说浏览器可以直接获取到正在运行的 JavaScript 的源码。</li>
</ul>
<p>由于这两个原因，导致 JavaScript 代码是不安全的，任何人都可以读、分析、复制、盗用，甚至篡改。</p>
<p>所以说，对于上述情形，客户端 JavaScript 对于某些加密的实现是很容易被找到或模拟的，了解了加密逻辑后，模拟参数的构造和请求也就是轻而易举了，所以如果 JavaScript 没有做任何层面的保护的话，接口加密技术基本上对数据起不到什么防护作用。</p>
<p>如果你不想让自己的数据被轻易获取，不想他人了解 JavaScript 逻辑的实现，或者想降低被不怀好意的人甚至是黑客攻击。那么你就需要用到 JavaScript 压缩、混淆和加密技术了。</p>
<p>这里压缩、混淆、加密技术简述如下。</p>
<ul>
<li>代码压缩：即去除 JavaScript 代码中的不必要的空格、换行等内容，使源码都压缩为几行内容，降低代码可读性，当然同时也能提高网站的加载速度。</li>
<li>代码混淆：使用变量替换、字符串阵列化、控制流平坦化、多态变异、僵尸函数、调试保护等手段，使代码变得难以阅读和分析，达到最终保护的目的。但这不影响代码原有功能。是理想、实用的 JavaScript 保护方案。</li>
<li>代码加密：可以通过某种手段将 JavaScript 代码进行加密，转成人无法阅读或者解析的代码，如将代码完全抽象化加密，如 eval 加密。另外还有更强大的加密技术，可以直接将 JavaScript 代码用 C/C++ 实现，JavaScript 调用其编译后形成的文件来执行相应的功能，如 Emscripten 还有 WebAssembly。</li>
</ul>
<p>下面我们对上面的技术分别予以介绍。</p>
<h3>接口加密技术</h3>
<p>数据一般都是通过服务器提供的接口来获取的，网站或 App 可以请求某个数据接口获取到对应的数据，然后再把获取的数据展示出来。</p>
<p>但有些数据是比较宝贵或私密的，这些数据肯定是需要一定层面上的保护。所以不同接口的实现也就对应着不同的安全防护级别，我们这里来总结下。</p>
<h4>完全开放的接口</h4>
<p>有些接口是没有设置任何防护的，谁都可以调用和访问，而且没有任何时空限制和频率限制。任何人只要知道了接口的调用方式就能无限制地调用。</p>
<p>这种接口的安全性是非常非常低的，如果接口的调用方式一旦泄露或被抓包获取到，任何人都可以无限制地对数据进行操作或访问。此时如果接口里面包含一些重要的数据或隐私数据，就能轻易被篡改或窃取了。</p>
<h4>接口参数加密</h4>
<p>为了提升接口的安全性，客户端会和服务端约定一种接口校验方式，一般来说会使用到各种加密和编码算法，如 Base64、Hex 编码，MD5、AES、DES、RSA 等加密。</p>
<p>比如客户端和服务器双方约定一个 sign 用作接口的签名校验，其生成逻辑是客户端将 URL Path 进行 MD5 加密然后拼接上 URL 的某个参数再进行 Base64 编码，最后得到一个字符串 sign，这个 sign 会通过 Request URL 的某个参数或 Request Headers 发送给服务器。服务器接收到请求后，对 URL Path 同样进行 MD5 加密，然后拼接上 URL 的某个参数，也进行 Base64 编码得到了一个 sign，然后比对生成的 sign 和客户端发来的 sign 是否是一致的，如果是一致的，那就返回正确的结果，否则拒绝响应。这就是一个比较简单的接口参数加密的实现。如果有人想要调用这个接口的话，必须要定义好 sign 的生成逻辑，否则是无法正常调用接口的。</p>
<p>以上就是一个基本的接口参数加密逻辑的实现。</p>
<p>当然上面的这个实现思路比较简单，这里还可以增加一些时间戳信息增加时效性判断，或增加一些非对称加密进一步提高加密的复杂程度。但不管怎样，只要客户端和服务器约定好了加密和校验逻辑，任何形式加密算法都是可以的。</p>
<p>这里要实现接口参数加密就需要用到一些加密算法，客户端和服务器肯定也都有对应的 SDK 实现这些加密算法，如 JavaScript 的 crypto-js，Python 的 hashlib、Crypto 等等。</p>
<p>但还是如上文所说，如果是网页的话，客户端实现加密逻辑如果是用 JavaScript 来实现，其源代码对用户是完全可见的，如果没有对 JavaScript 做任何保护的话，是很容易弄清楚客户端加密的流程的。</p>
<p>因此，我们需要对 JavaScript 利用压缩、混淆、加密的方式来对客户端的逻辑进行一定程度上的保护。</p>
<h3>JavaScript 压缩、混淆、加密</h3>
<p>下面我们再来介绍下 JavaScript 的压缩、混淆和加密技术。</p>
<h4>JavaScript 压缩</h4>
<p>这个非常简单，JavaScript 压缩即去除 JavaScript 代码中的不必要的空格、换行等内容或者把一些可能公用的代码进行处理实现共享，最后输出的结果都被压缩为几行内容，代码可读性变得很差，同时也能提高网站加载速度。</p>
<p>如果仅仅是去除空格换行这样的压缩方式，其实几乎是没有任何防护作用的，因为这种压缩方式仅仅是降低了代码的直接可读性。如果我们有一些格式化工具可以轻松将 JavaScript 代码变得易读，比如利用 IDE、在线工具或 Chrome 浏览器都能还原格式化的代码。</p>
<p>目前主流的前端开发技术大多都会利用 Webpack 进行打包，Webpack 会对源代码进行编译和压缩，输出几个打包好的 JavaScript 文件，其中我们可以看到输出的 JavaScript 文件名带有一些不规则字符串，同时文件内容可能只有几行内容，变量名都是一些简单字母表示。这其中就包含 JavaScript 压缩技术，比如一些公共的库输出成 bundle 文件，一些调用逻辑压缩和转义成几行代码，这些都属于 JavaScript 压缩。另外其中也包含了一些很基础的 JavaScript 混淆技术，比如把变量名、方法名替换成一些简单字符，降低代码可读性。</p>
<p>但整体来说，JavaScript 压缩技术只能在很小的程度上起到防护作用，要想真正提高防护效果还得依靠 JavaScript 混淆和加密技术。</p>
<h4>JavaScript 混淆</h4>
<p>JavaScript 混淆完全是在 JavaScript 上面进行的处理，它的目的就是使得 JavaScript 变得难以阅读和分析，大大降低代码可读性，是一种很实用的 JavaScript 保护方案。</p>
<p>JavaScript 混淆技术主要有以下几种：</p>
<ul>
<li>变量混淆</li>
</ul>
<p>将带有含意的变量名、方法名、常量名随机变为无意义的类乱码字符串，降低代码可读性，如转成单个字符或十六进制字符串。</p>
<ul>
<li>字符串混淆</li>
</ul>
<p>将字符串阵列化集中放置、并可进行 MD5 或 Base64 加密存储，使代码中不出现明文字符串，这样可以避免使用全局搜索字符串的方式定位到入口点。</p>
<ul>
<li>属性加密</li>
</ul>
<p>针对 JavaScript 对象的属性进行加密转化，隐藏代码之间的调用关系。</p>
<ul>
<li>控制流平坦化</li>
</ul>
<p>打乱函数原有代码执行流程及函数调用关系，使代码逻变得混乱无序。</p>
<ul>
<li>僵尸代码</li>
</ul>
<p>随机在代码中插入无用的僵尸代码、僵尸函数，进一步使代码混乱。</p>
<ul>
<li>调试保护</li>
</ul>
<p>基于调试器特性，对当前运行环境进行检验，加入一些强制调试 debugger 语句，使其在调试模式下难以顺利执行 JavaScript 代码。</p>
<ul>
<li>多态变异</li>
</ul>
<p>使 JavaScript 代码每次被调用时，将代码自身即立刻自动发生变异，变化为与之前完全不同的代码，即功能完全不变，只是代码形式变异，以此杜绝代码被动态分析调试。</p>
<ul>
<li>锁定域名</li>
</ul>
<p>使 JavaScript 代码只能在指定域名下执行。</p>
<ul>
<li>反格式化</li>
</ul>
<p>如果对 JavaScript 代码进行格式化，则无法执行，导致浏览器假死。</p>
<ul>
<li>特殊编码</li>
</ul>
<p>将 JavaScript 完全编码为人不可读的代码，如表情符号、特殊表示内容等等。</p>
<p>总之，以上方案都是 JavaScript 混淆的实现方式，可以在不同程度上保护 JavaScript 代码。</p>
<p>在前端开发中，现在 JavaScript 混淆主流的实现是 javascript-obfuscator 这个库，利用它我们可以非常方便地实现页面的混淆，它与 Webpack 结合起来，最终可以输出压缩和混淆后的 JavaScript 代码，使得可读性大大降低，难以逆向。</p>
<p>下面我们会介绍下 javascript-obfuscator 对代码混淆的实现，了解了实现，那么自然我们就对混淆的机理有了更加深刻的认识。</p>
<p>javascript-obfuscator 的官网地址为：<a href="https://obfuscator.io/">https://obfuscator.io/</a>，其官方介绍内容如下：</p>
<blockquote>
<p>A free and efficient obfuscator for JavaScript (including ES2017). Make your code harder to copy and prevent people from stealing your work.</p>
</blockquote>
<p>它是支持 ES8 的免费、高效的 JavaScript 混淆库，它可以使得你的 JavaScript 代码经过混淆后难以被复制、盗用，混淆后的代码具有和原来的代码一模一样的功能。</p>
<p>怎么使用呢？首先，我们需要安装好 Node.js，可以使用 npm 命令。</p>
<p>然后新建一个文件夹，比如 js-obfuscate，随后进入该文件夹，初始化工作空间：</p>
<pre><code data-language="java" class="lang-java">npm init
</code></pre>
<p>这里会提示我们输入一些信息，创建一个 package.json 文件，这就完成了项目初始化了。</p>
<p>接下来我们来安装 javascript-obfuscator 这个库：</p>
<pre><code data-language="js" class="lang-js">npm install --save-dev javascript-obfuscator
</code></pre>
<p>接下来我们就可以编写代码来实现混淆了，如新建一个 main.js 文件，内容如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
let x = '1' + 1
console.log('x', x)
`</span>

<span class="hljs-keyword">const</span> options = {
   <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>,
   <span class="hljs-attr">controlFlowFlattening</span>: <span class="hljs-literal">true</span>

}

<span class="hljs-keyword">const</span> obfuscator = <span class="hljs-built_in">require</span>(<span class="hljs-string">'javascript-obfuscator'</span>)
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">obfuscate</span>(<span class="hljs-params">code, options</span>) </span>{
   <span class="hljs-keyword">return</span> obfuscator.obfuscate(code, options).getObfuscatedCode()
}
<span class="hljs-built_in">console</span>.log(obfuscate(code, options))
</code></pre>
<p>在这里我们定义了两个变量，一个是 code，即需要被混淆的代码，另一个是混淆选项，是一个 Object。接下来我们引入了 javascript-obfuscator 库，然后定义了一个方法，传入 code 和 options，来获取混淆后的代码，最后控制台输出混淆后的代码。</p>
<p>代码逻辑比较简单，我们来执行一下代码：</p>
<pre><code>node main.js
</code></pre>
<p>输出结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x53bf = [<span class="hljs-string">'log'</span>];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x1d84fe, _0x3aeda0</span>) </span>{
   <span class="hljs-keyword">var</span> _0x10a5a = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x2f0a52</span>) </span>{
       <span class="hljs-keyword">while</span> (--_0x2f0a52) {
           _0x1d84fe[<span class="hljs-string">'push'</span>](_0x1d84fe[<span class="hljs-string">'shift'</span>]());
      }
  };
   _0x10a5a(++_0x3aeda0);
}(_0x53bf, <span class="hljs-number">0x172</span>));
<span class="hljs-keyword">var</span> _0x480a = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x4341e5, _0x5923b4</span>) </span>{
   _0x4341e5 = _0x4341e5 - <span class="hljs-number">0x0</span>;
   <span class="hljs-keyword">var</span> _0xb3622e = _0x53bf[_0x4341e5];
   <span class="hljs-keyword">return</span> _0xb3622e;
};
<span class="hljs-keyword">let</span> x = <span class="hljs-string">'1'</span> + <span class="hljs-number">0x1</span>;
<span class="hljs-built_in">console</span>[_0x480a(<span class="hljs-string">'0x0'</span>)](<span class="hljs-string">'x'</span>, x);
</code></pre>
<p>看到了吧，这么简单的两行代码，被我们混淆成了这个样子，其实这里我们就是设定了一个“控制流扁平化”的选项。</p>
<p>整体看来，代码的可读性大大降低，也大大加大了 JavaScript 调试的难度。</p>
<p>好，接下来我们来跟着 javascript-obfuscator 走一遍，就能具体知道 JavaScript 混淆到底有多少方法了。</p>
<h5>代码压缩</h5>
<p>这里 javascript-obfuscator 也提供了代码压缩的功能，使用其参数 compact 即可完成 JavaScript 代码的压缩，输出为一行内容。默认是 true，如果定义为 false，则混淆后的代码会分行显示。</p>
<p>示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
let x = '1' + 1
console.log('x', x)
`</span>
<span class="hljs-keyword">const</span> options = {
   <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>
}
</code></pre>
<p>这里我们先把代码压缩 compact 选项设置为 false，运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">let</span> x = <span class="hljs-string">'1'</span> + <span class="hljs-number">0x1</span>;
<span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](<span class="hljs-string">'x'</span>, x);
</code></pre>
<p>如果不设置 compact 或把 compact 设置为 true，结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x151c=[<span class="hljs-string">'log'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x1ce384,_0x20a7c7</span>)</span>{<span class="hljs-keyword">var</span> _0x25fc92=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x188aec</span>)</span>{<span class="hljs-keyword">while</span>(--_0x188aec){_0x1ce384[<span class="hljs-string">'push'</span>](_0x1ce384[<span class="hljs-string">'shift'</span>]());}};_0x25fc92(++_0x20a7c7);}(_0x151c,<span class="hljs-number">0x1b7</span>));<span class="hljs-keyword">var</span> _0x553e=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x259219,_0x241445</span>)</span>{_0x259219=_0x259219<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x56d72d=_0x151c[_0x259219];<span class="hljs-keyword">return</span> _0x56d72d;};<span class="hljs-keyword">let</span> x=<span class="hljs-string">'1'</span>+<span class="hljs-number">0x1</span>;<span class="hljs-built_in">console</span>[_0x553e(<span class="hljs-string">'0x0'</span>)](<span class="hljs-string">'x'</span>,x);
</code></pre>
<p>可以看到单行显示的时候，对变量名进行了进一步的混淆和控制流扁平化操作。</p>
<h5>变量名混淆</h5>
<p>变量名混淆可以通过配置 identifierNamesGenerator 参数实现，我们通过这个参数可以控制变量名混淆的方式，如 hexadecimal 则会替换为 16 进制形式的字符串，在这里我们可以设定如下值：</p>
<ul>
<li>hexadecimal：将变量名替换为 16 进制形式的字符串，如 0xabc123。</li>
<li>mangled：将变量名替换为普通的简写字符，如 a、b、c 等。</li>
</ul>
<p>该参数默认为 hexadecimal。</p>
<p>我们将该参数修改为 mangled 来试一下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
let hello = '1' + 1
console.log('hello', hello)
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">compact</span>: <span class="hljs-literal">true</span>,
  <span class="hljs-attr">identifierNamesGenerator</span>: <span class="hljs-string">'mangled'</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> a=[<span class="hljs-string">'hello'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">c,d</span>)</span>{<span class="hljs-keyword">var</span> e=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">f</span>)</span>{<span class="hljs-keyword">while</span>(--f){c[<span class="hljs-string">'push'</span>](c[<span class="hljs-string">'shift'</span>]());}};e(++d);}(a,<span class="hljs-number">0x9b</span>));<span class="hljs-keyword">var</span> b=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">c,d</span>)</span>{c=c<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> e=a[c];<span class="hljs-keyword">return</span> e;};<span class="hljs-keyword">let</span> hello=<span class="hljs-string">'1'</span>+<span class="hljs-number">0x1</span>;<span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](b(<span class="hljs-string">'0x0'</span>),hello);
</code></pre>
<p>可以看到这里的变量命名都变成了 a、b 等形式。</p>
<p>如果我们将 identifierNamesGenerator 修改为 hexadecimal 或者不设置，运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x4e98=[<span class="hljs-string">'log'</span>,<span class="hljs-string">'hello'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x4464de,_0x39de6c</span>)</span>{<span class="hljs-keyword">var</span> _0xdffdda=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x6a95d5</span>)</span>{<span class="hljs-keyword">while</span>(--_0x6a95d5){_0x4464de[<span class="hljs-string">'push'</span>](_0x4464de[<span class="hljs-string">'shift'</span>]());}};_0xdffdda(++_0x39de6c);}(_0x4e98,<span class="hljs-number">0xc8</span>));<span class="hljs-keyword">var</span> _0x53cb=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x393bda,_0x8504e7</span>)</span>{_0x393bda=_0x393bda<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x46ab80=_0x4e98[_0x393bda];<span class="hljs-keyword">return</span> _0x46ab80;};<span class="hljs-keyword">let</span> hello=<span class="hljs-string">'1'</span>+<span class="hljs-number">0x1</span>;<span class="hljs-built_in">console</span>[_0x53cb(<span class="hljs-string">'0x0'</span>)](_0x53cb(<span class="hljs-string">'0x1'</span>),hello);
</code></pre>
<p>可以看到选用了 mangled，其代码体积会更小，但 hexadecimal 其可读性会更低。</p>
<p>另外我们还可以通过设置 identifiersPrefix 参数来控制混淆后的变量前缀，示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
let hello = '1' + 1
console.log('hello', hello)
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">identifiersPrefix</span>: <span class="hljs-string">'germey'</span>
}
</code></pre>
<p>运行结果：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> germey_0x3dea=[<span class="hljs-string">'log'</span>,<span class="hljs-string">'hello'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x348ff3,_0x5330e8</span>)</span>{<span class="hljs-keyword">var</span> _0x1568b1=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x4740d8</span>)</span>{<span class="hljs-keyword">while</span>(--_0x4740d8){_0x348ff3[<span class="hljs-string">'push'</span>](_0x348ff3[<span class="hljs-string">'shift'</span>]());}};_0x1568b1(++_0x5330e8);}(germey_0x3dea,<span class="hljs-number">0x94</span>));<span class="hljs-keyword">var</span> germey_0x30e4=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x2e8f7c,_0x1066a8</span>)</span>{_0x2e8f7c=_0x2e8f7c<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x5166ba=germey_0x3dea[_0x2e8f7c];<span class="hljs-keyword">return</span> _0x5166ba;};<span class="hljs-keyword">let</span> hello=<span class="hljs-string">'1'</span>+<span class="hljs-number">0x1</span>;<span class="hljs-built_in">console</span>[germey_0x30e4(<span class="hljs-string">'0x0'</span>)](germey_0x30e4(<span class="hljs-string">'0x1'</span>),hello);
</code></pre>
<p>可以看到混淆后的变量前缀加上了我们自定义的字符串 germey。</p>
<p>另外 renameGlobals 这个参数还可以指定是否混淆全局变量和函数名称，默认为 false。示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
var $ = function(id) {
  return document.getElementById(id);
};
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">renameGlobals</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x4864b0=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x5763be</span>)</span>{<span class="hljs-keyword">return</span> <span class="hljs-built_in">document</span>[<span class="hljs-string">'getElementById'</span>](_0x5763be);};
</code></pre>
<p>可以看到这里我们声明了一个全局变量 $，在 renameGlobals 设置为 true 之后，$ 这个变量也被替换了。如果后文用到了这个 $ 对象，可能就会有找不到定义的错误，因此这个参数可能导致代码执行不通。</p>
<p>如果我们不设置 renameGlobals 或者设置为 false，结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x239a=[<span class="hljs-string">'getElementById'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x3f45a3,_0x583dfa</span>)</span>{<span class="hljs-keyword">var</span> _0x2cade2=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x28479a</span>)</span>{<span class="hljs-keyword">while</span>(--_0x28479a){_0x3f45a3[<span class="hljs-string">'push'</span>](_0x3f45a3[<span class="hljs-string">'shift'</span>]());}};_0x2cade2(++_0x583dfa);}(_0x239a,<span class="hljs-number">0xe1</span>));<span class="hljs-keyword">var</span> _0x3758=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x18659d,_0x50c21d</span>)</span>{_0x18659d=_0x18659d<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x531b8d=_0x239a[_0x18659d];<span class="hljs-keyword">return</span> _0x531b8d;};<span class="hljs-keyword">var</span> $=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x3d8723</span>)</span>{<span class="hljs-keyword">return</span> <span class="hljs-built_in">document</span>[_0x3758(<span class="hljs-string">'0x0'</span>)](_0x3d8723);};
</code></pre>
<p>可以看到，最后还是有 $ 的声明，其全局名称没有被改变。</p>
<h5>字符串混淆</h5>
<p>字符串混淆，即将一个字符串声明放到一个数组里面，使之无法被直接搜索到。我们可以通过控制 stringArray 参数来控制，默认为 true。</p>
<p>我们还可以通过 rotateStringArray 参数来控制数组化后结果的元素顺序，默认为 true。<br>
还可以通过 stringArrayEncoding 参数来控制数组的编码形式，默认不开启编码，如果设置为 true 或 base64，则会使用 Base64 编码，如果设置为 rc4，则使用 RC4 编码。<br>
还可以通过 stringArrayThreshold 来控制启用编码的概率，范围 0 到 1，默认 0.8。</p>
<p>示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
var a = 'hello world'
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">stringArray</span>: <span class="hljs-literal">true</span>,
  <span class="hljs-attr">rotateStringArray</span>: <span class="hljs-literal">true</span>,
  <span class="hljs-attr">stringArrayEncoding</span>: <span class="hljs-literal">true</span>, <span class="hljs-comment">// 'base64' or 'rc4' or false</span>
  <span class="hljs-attr">stringArrayThreshold</span>: <span class="hljs-number">1</span>,
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x4215=[<span class="hljs-string">'aGVsbG8gd29ybGQ='</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x42bf17,_0x4c348f</span>)</span>{<span class="hljs-keyword">var</span> _0x328832=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x355be1</span>)</span>{<span class="hljs-keyword">while</span>(--_0x355be1){_0x42bf17[<span class="hljs-string">'push'</span>](_0x42bf17[<span class="hljs-string">'shift'</span>]());}};_0x328832(++_0x4c348f);}(_0x4215,<span class="hljs-number">0x1da</span>));<span class="hljs-keyword">var</span> _0x5191=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x3cf2ba,_0x1917d8</span>)</span>{_0x3cf2ba=_0x3cf2ba<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x1f93f0=_0x4215[_0x3cf2ba];<span class="hljs-keyword">if</span>(_0x5191[<span class="hljs-string">'LqbVDH'</span>]===<span class="hljs-literal">undefined</span>){(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x5096b2;<span class="hljs-keyword">try</span>{<span class="hljs-keyword">var</span> _0x282db1=<span class="hljs-built_in">Function</span>(<span class="hljs-string">'return\x20(function()\x20'</span>+<span class="hljs-string">'{}.constructor(\x22return\x20this\x22)(\x20)'</span>+<span class="hljs-string">');'</span>);_0x5096b2=_0x282db1();}<span class="hljs-keyword">catch</span>(_0x2acb9c){_0x5096b2=<span class="hljs-built_in">window</span>;}<span class="hljs-keyword">var</span> _0x388c14=<span class="hljs-string">'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/='</span>;_0x5096b2[<span class="hljs-string">'atob'</span>]||(_0x5096b2[<span class="hljs-string">'atob'</span>]=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x4cc27c</span>)</span>{<span class="hljs-keyword">var</span> _0x2af4ae=<span class="hljs-built_in">String</span>(_0x4cc27c)[<span class="hljs-string">'replace'</span>](<span class="hljs-regexp">/=+$/</span>,<span class="hljs-string">''</span>);<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x21400b=<span class="hljs-number">0x0</span>,_0x3f4e2e,_0x5b193b,_0x233381=<span class="hljs-number">0x0</span>,_0x3dccf7=<span class="hljs-string">''</span>;_0x5b193b=_0x2af4ae[<span class="hljs-string">'charAt'</span>](_0x233381++);~_0x5b193b&amp;&amp;(_0x3f4e2e=_0x21400b%<span class="hljs-number">0x4</span>?_0x3f4e2e*<span class="hljs-number">0x40</span>+_0x5b193b:_0x5b193b,_0x21400b++%<span class="hljs-number">0x4</span>)?_0x3dccf7+=<span class="hljs-built_in">String</span>[<span class="hljs-string">'fromCharCode'</span>](<span class="hljs-number">0xff</span>&amp;_0x3f4e2e&gt;&gt;(<span class="hljs-number">-0x2</span>*_0x21400b&amp;<span class="hljs-number">0x6</span>)):<span class="hljs-number">0x0</span>){_0x5b193b=_0x388c14[<span class="hljs-string">'indexOf'</span>](_0x5b193b);}<span class="hljs-keyword">return</span> _0x3dccf7;});}());_0x5191[<span class="hljs-string">'DuIurT'</span>]=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x51888e</span>)</span>{<span class="hljs-keyword">var</span> _0x29801f=atob(_0x51888e);<span class="hljs-keyword">var</span> _0x561e62=[];<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x5dd788=<span class="hljs-number">0x0</span>,_0x1a8b73=_0x29801f[<span class="hljs-string">'length'</span>];_0x5dd788&lt;_0x1a8b73;_0x5dd788++){_0x561e62+=<span class="hljs-string">'%'</span>+(<span class="hljs-string">'00'</span>+_0x29801f[<span class="hljs-string">'charCodeAt'</span>](_0x5dd788)[<span class="hljs-string">'toString'</span>](<span class="hljs-number">0x10</span>))[<span class="hljs-string">'slice'</span>](<span class="hljs-number">-0x2</span>);}<span class="hljs-keyword">return</span> <span class="hljs-built_in">decodeURIComponent</span>(_0x561e62);};_0x5191[<span class="hljs-string">'mgoBRd'</span>]={};_0x5191[<span class="hljs-string">'LqbVDH'</span>]=!![];}<span class="hljs-keyword">var</span> _0x1741f0=_0x5191[<span class="hljs-string">'mgoBRd'</span>][_0x3cf2ba];<span class="hljs-keyword">if</span>(_0x1741f0===<span class="hljs-literal">undefined</span>){_0x1f93f0=_0x5191[<span class="hljs-string">'DuIurT'</span>](_0x1f93f0);_0x5191[<span class="hljs-string">'mgoBRd'</span>][_0x3cf2ba]=_0x1f93f0;}<span class="hljs-keyword">else</span>{_0x1f93f0=_0x1741f0;}<span class="hljs-keyword">return</span> _0x1f93f0;};<span class="hljs-keyword">var</span> a=_0x5191(<span class="hljs-string">'0x0'</span>);
</code></pre>
<p>可以看到这里就把字符串进行了 Base64 编码，我们再也无法通过查找的方式找到字符串的位置了。</p>
<p>如果将 stringArray 设置为 false 的话，输出就是这样：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> a=<span class="hljs-string">'hello\x20world'</span>;
</code></pre>
<p>字符串就仍然是明文显示的，没有被编码。</p>
<p>另外我们还可以使用 unicodeEscapeSequence 这个参数对字符串进行 Unicode 转码，使之更加难以辨认，示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
var a = 'hello world'
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>,
  <span class="hljs-attr">unicodeEscapeSequence</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x5c0d = [<span class="hljs-string">'\x68\x65\x6c\x6c\x6f\x20\x77\x6f\x72\x6c\x64'</span>];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x54cc9c, _0x57a3b2</span>) </span>{
  <span class="hljs-keyword">var</span> _0xf833cf = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x3cd8c6</span>) </span>{
    <span class="hljs-keyword">while</span> (--_0x3cd8c6) {
      _0x54cc9c[<span class="hljs-string">'push'</span>](_0x54cc9c[<span class="hljs-string">'shift'</span>]());
    }
};
_0xf833cf(++_0x57a3b2);
}(_0x5c0d, <span class="hljs-number">0x17d</span>));
<span class="hljs-keyword">var</span> _0x28e8 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x3fd645, _0x2cf5e7</span>) </span>{
  _0x3fd645 = _0x3fd645 - <span class="hljs-number">0x0</span>;
  <span class="hljs-keyword">var</span> _0x298a20 = _0x5c0d[_0x3fd645];
  <span class="hljs-keyword">return</span> _0x298a20;
};
<span class="hljs-keyword">var</span> a = _0x28e8(<span class="hljs-string">'0x0'</span>);
</code></pre>
<p>可以看到，这里字符串被数字化和 Unicode 化，非常难以辨认。</p>
<p>在很多 JavaScript 逆向的过程中，一些关键的字符串可能会作为切入点来查找加密入口。用了这种混淆之后，如果有人想通过全局搜索的方式搜索 hello 这样的字符串找加密入口，也没法搜到了。</p>
<h5>代码自我保护</h5>
<p>我们可以通过设置 selfDefending 参数来开启代码自我保护功能。开启之后，混淆后的 JavaScript 会强制以一行形式显示，如果我们将混淆后的代码进行格式化（美化）或者重命名，该段代码将无法执行。</p>
<p>例如：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
console.log('hello world')
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">selfDefending</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x26da=[<span class="hljs-string">'log'</span>,<span class="hljs-string">'hello\x20world'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x190327,_0x57c2c0</span>)</span>{<span class="hljs-keyword">var</span> _0x577762=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0xc9dabb</span>)</span>{<span class="hljs-keyword">while</span>(--_0xc9dabb){_0x190327[<span class="hljs-string">'push'</span>](_0x190327[<span class="hljs-string">'shift'</span>]());}};<span class="hljs-keyword">var</span> _0x35976e=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x16b3fe={<span class="hljs-string">'data'</span>:{<span class="hljs-string">'key'</span>:<span class="hljs-string">'cookie'</span>,<span class="hljs-string">'value'</span>:<span class="hljs-string">'timeout'</span>},<span class="hljs-string">'setCookie'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x2d52d5,_0x16feda,_0x57cadf,_0x56056f</span>)</span>{_0x56056f=_0x56056f||{};<span class="hljs-keyword">var</span> _0x5b6dc3=_0x16feda+<span class="hljs-string">'='</span>+_0x57cadf;<span class="hljs-keyword">var</span> _0x333ced=<span class="hljs-number">0x0</span>;<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x333ced=<span class="hljs-number">0x0</span>,_0x19ae36=_0x2d52d5[<span class="hljs-string">'length'</span>];_0x333ced&lt;_0x19ae36;_0x333ced++){<span class="hljs-keyword">var</span> _0x409587=_0x2d52d5[_0x333ced];_0x5b6dc3+=<span class="hljs-string">';\x20'</span>+_0x409587;<span class="hljs-keyword">var</span> _0x4aa006=_0x2d52d5[_0x409587];_0x2d52d5[<span class="hljs-string">'push'</span>](_0x4aa006);_0x19ae36=_0x2d52d5[<span class="hljs-string">'length'</span>];<span class="hljs-keyword">if</span>(_0x4aa006!==!![]){_0x5b6dc3+=<span class="hljs-string">'='</span>+_0x4aa006;}}_0x56056f[<span class="hljs-string">'cookie'</span>]=_0x5b6dc3;},<span class="hljs-string">'removeCookie'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span><span class="hljs-string">'dev'</span>;},<span class="hljs-string">'getCookie'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x30c497,_0x51923d</span>)</span>{_0x30c497=_0x30c497||<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x4b7e18</span>)</span>{<span class="hljs-keyword">return</span> _0x4b7e18;};<span class="hljs-keyword">var</span> _0x557e06=_0x30c497(<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'(?:^|;\x20)'</span>+_0x51923d[<span class="hljs-string">'replace'</span>](<span class="hljs-regexp">/([.$?*|{}()[]\/+^])/g</span>,<span class="hljs-string">'$1'</span>)+<span class="hljs-string">'=([^;]*)'</span>));<span class="hljs-keyword">var</span> _0x817646=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0xf3fae7,_0x5d8208</span>)</span>{_0xf3fae7(++_0x5d8208);};_0x817646(_0x577762,_0x57c2c0);<span class="hljs-keyword">return</span> _0x557e06?<span class="hljs-built_in">decodeURIComponent</span>(_0x557e06[<span class="hljs-number">0x1</span>]):<span class="hljs-literal">undefined</span>;}};<span class="hljs-keyword">var</span> _0x4673cd=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x4c6c5c=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x5cw+\x20*\x5c(\x5c)\x20*{\x5cw+\x20*[\x27|\x22].+[\x27|\x22];?\x20*}'</span>);<span class="hljs-keyword">return</span> _0x4c6c5c[<span class="hljs-string">'test'</span>](_0x16b3fe[<span class="hljs-string">'removeCookie'</span>][<span class="hljs-string">'toString'</span>]());};_0x16b3fe[<span class="hljs-string">'updateCookie'</span>]=_0x4673cd;<span class="hljs-keyword">var</span> _0x5baa80=<span class="hljs-string">''</span>;<span class="hljs-keyword">var</span> _0x1faf19=_0x16b3fe[<span class="hljs-string">'updateCookie'</span>]();<span class="hljs-keyword">if</span>(!_0x1faf19){_0x16b3fe[<span class="hljs-string">'setCookie'</span>]([<span class="hljs-string">'*'</span>],<span class="hljs-string">'counter'</span>,<span class="hljs-number">0x1</span>);}<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(_0x1faf19){_0x5baa80=_0x16b3fe[<span class="hljs-string">'getCookie'</span>](<span class="hljs-literal">null</span>,<span class="hljs-string">'counter'</span>);}<span class="hljs-keyword">else</span>{_0x16b3fe[<span class="hljs-string">'removeCookie'</span>]();}};_0x35976e();}(_0x26da,<span class="hljs-number">0x140</span>));<span class="hljs-keyword">var</span> _0x4391=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x1b42d8,_0x57edc8</span>)</span>{_0x1b42d8=_0x1b42d8<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x2fbeca=_0x26da[_0x1b42d8];<span class="hljs-keyword">return</span> _0x2fbeca;};<span class="hljs-keyword">var</span> _0x197926=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x10598f=!![];<span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0xffa3b3,_0x7a40f9</span>)</span>{<span class="hljs-keyword">var</span> _0x48e571=_0x10598f?<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">if</span>(_0x7a40f9){<span class="hljs-keyword">var</span> _0x2194b5=_0x7a40f9[<span class="hljs-string">'apply'</span>](_0xffa3b3,<span class="hljs-built_in">arguments</span>);_0x7a40f9=<span class="hljs-literal">null</span>;<span class="hljs-keyword">return</span> _0x2194b5;}}:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{};_0x10598f=![];<span class="hljs-keyword">return</span> _0x48e571;};}();<span class="hljs-keyword">var</span> _0x2c6fd7=_0x197926(<span class="hljs-keyword">this</span>,<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x4828bb=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span><span class="hljs-string">'\x64\x65\x76'</span>;},_0x35c3bc=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span><span class="hljs-string">'\x77\x69\x6e\x64\x6f\x77'</span>;};<span class="hljs-keyword">var</span> _0x456070=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x4576a4=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x5c\x77\x2b\x20\x2a\x5c\x28\x5c\x29\x20\x2a\x7b\x5c\x77\x2b\x20\x2a\x5b\x27\x7c\x22\x5d\x2e\x2b\x5b\x27\x7c\x22\x5d\x3b\x3f\x20\x2a\x7d'</span>);<span class="hljs-keyword">return</span>!_0x4576a4[<span class="hljs-string">'\x74\x65\x73\x74'</span>](_0x4828bb[<span class="hljs-string">'\x74\x6f\x53\x74\x72\x69\x6e\x67'</span>]());};<span class="hljs-keyword">var</span> _0x3fde69=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0xabb6f4=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x28\x5c\x5c\x5b\x78\x7c\x75\x5d\x28\x5c\x77\x29\x7b\x32\x2c\x34\x7d\x29\x2b'</span>);<span class="hljs-keyword">return</span> _0xabb6f4[<span class="hljs-string">'\x74\x65\x73\x74'</span>](_0x35c3bc[<span class="hljs-string">'\x74\x6f\x53\x74\x72\x69\x6e\x67'</span>]());};<span class="hljs-keyword">var</span> _0x2d9a50=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x58fdb4</span>)</span>{<span class="hljs-keyword">var</span> _0x2a6361=~<span class="hljs-number">-0x1</span>&gt;&gt;<span class="hljs-number">0x1</span>+<span class="hljs-number">0xff</span>%<span class="hljs-number">0x0</span>;<span class="hljs-keyword">if</span>(_0x58fdb4[<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>](<span class="hljs-string">'\x69'</span>===_0x2a6361)){_0xc388c5(_0x58fdb4);}};<span class="hljs-keyword">var</span> _0xc388c5=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x2073d6</span>)</span>{<span class="hljs-keyword">var</span> _0x6bb49f=~<span class="hljs-number">-0x4</span>&gt;&gt;<span class="hljs-number">0x1</span>+<span class="hljs-number">0xff</span>%<span class="hljs-number">0x0</span>;<span class="hljs-keyword">if</span>(_0x2073d6[<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>]((!![]+<span class="hljs-string">''</span>)[<span class="hljs-number">0x3</span>])!==_0x6bb49f){_0x2d9a50(_0x2073d6);}};<span class="hljs-keyword">if</span>(!_0x456070()){<span class="hljs-keyword">if</span>(!_0x3fde69()){_0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\u0435\x78\x4f\x66'</span>);}<span class="hljs-keyword">else</span>{_0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>);}}<span class="hljs-keyword">else</span>{_0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\u0435\x78\x4f\x66'</span>);}});_0x2c6fd7();<span class="hljs-built_in">console</span>[_0x4391(<span class="hljs-string">'0x0'</span>)](_0x4391(<span class="hljs-string">'0x1'</span>));
</code></pre>
<p>如果我们将上述代码放到控制台，它的执行结果和之前是一模一样的，没有任何问题。<br>
如果我们将其进行格式化，会变成如下内容：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x26da = [<span class="hljs-string">'log'</span>, <span class="hljs-string">'hello\x20world'</span>];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x190327, _0x57c2c0</span>) </span>{
    <span class="hljs-keyword">var</span> _0x577762 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0xc9dabb</span>) </span>{
        <span class="hljs-keyword">while</span> (--_0xc9dabb) {
            _0x190327[<span class="hljs-string">'push'</span>](_0x190327[<span class="hljs-string">'shift'</span>]());
        }
    };
    <span class="hljs-keyword">var</span> _0x35976e = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> _0x16b3fe = {
            <span class="hljs-string">'data'</span>: {
                <span class="hljs-string">'key'</span>: <span class="hljs-string">'cookie'</span>,
                <span class="hljs-string">'value'</span>: <span class="hljs-string">'timeout'</span>
            },
            <span class="hljs-string">'setCookie'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x2d52d5, _0x16feda, _0x57cadf, _0x56056f</span>) </span>{
                _0x56056f = _0x56056f || {};
                <span class="hljs-keyword">var</span> _0x5b6dc3 = _0x16feda + <span class="hljs-string">'='</span> + _0x57cadf;
                <span class="hljs-keyword">var</span> _0x333ced = <span class="hljs-number">0x0</span>;
                <span class="hljs-keyword">for</span> (<span class="hljs-keyword">var</span> _0x333ced = <span class="hljs-number">0x0</span>, _0x19ae36 = _0x2d52d5[<span class="hljs-string">'length'</span>]; _0x333ced &lt; _0x19ae36; _0x333ced++) {
                    <span class="hljs-keyword">var</span> _0x409587 = _0x2d52d5[_0x333ced];
                    _0x5b6dc3 += <span class="hljs-string">';\x20'</span> + _0x409587;
                    <span class="hljs-keyword">var</span> _0x4aa006 = _0x2d52d5[_0x409587];
                    _0x2d52d5[<span class="hljs-string">'push'</span>](_0x4aa006);
                    _0x19ae36 = _0x2d52d5[<span class="hljs-string">'length'</span>];
                    <span class="hljs-keyword">if</span> (_0x4aa006 !== !![]) {
                        _0x5b6dc3 += <span class="hljs-string">'='</span> + _0x4aa006;
                    }
                }
                _0x56056f[<span class="hljs-string">'cookie'</span>] = _0x5b6dc3;
            }, <span class="hljs-string">'removeCookie'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
                <span class="hljs-keyword">return</span> <span class="hljs-string">'dev'</span>;
            }, <span class="hljs-string">'getCookie'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x30c497, _0x51923d</span>) </span>{
                _0x30c497 = _0x30c497 || <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x4b7e18</span>) </span>{
                    <span class="hljs-keyword">return</span> _0x4b7e18;
                };
                <span class="hljs-keyword">var</span> _0x557e06 = _0x30c497(<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'(?:^|;\x20)'</span> + _0x51923d[<span class="hljs-string">'replace'</span>](<span class="hljs-regexp">/([.$?*|{}()[]\/+^])/g</span>, <span class="hljs-string">'$1'</span>) + <span class="hljs-string">'=([^;]*)'</span>));
                <span class="hljs-keyword">var</span> _0x817646 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0xf3fae7, _0x5d8208</span>) </span>{
                    _0xf3fae7(++_0x5d8208);
                };
                _0x817646(_0x577762, _0x57c2c0);
                <span class="hljs-keyword">return</span> _0x557e06 ? <span class="hljs-built_in">decodeURIComponent</span>(_0x557e06[<span class="hljs-number">0x1</span>]) : <span class="hljs-literal">undefined</span>;
            }
        };
        <span class="hljs-keyword">var</span> _0x4673cd = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">var</span> _0x4c6c5c = <span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x5cw+\x20*\x5c(\x5c)\x20*{\x5cw+\x20*[\x27|\x22].+[\x27|\x22];?\x20*}'</span>);
            <span class="hljs-keyword">return</span> _0x4c6c5c[<span class="hljs-string">'test'</span>](_0x16b3fe[<span class="hljs-string">'removeCookie'</span>][<span class="hljs-string">'toString'</span>]());
        };
        _0x16b3fe[<span class="hljs-string">'updateCookie'</span>] = _0x4673cd;
        <span class="hljs-keyword">var</span> _0x5baa80 = <span class="hljs-string">''</span>;
        <span class="hljs-keyword">var</span> _0x1faf19 = _0x16b3fe[<span class="hljs-string">'updateCookie'</span>]();
        <span class="hljs-keyword">if</span> (!_0x1faf19) {
            _0x16b3fe[<span class="hljs-string">'setCookie'</span>]([<span class="hljs-string">'*'</span>], <span class="hljs-string">'counter'</span>, <span class="hljs-number">0x1</span>);
        } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (_0x1faf19) {
            _0x5baa80 = _0x16b3fe[<span class="hljs-string">'getCookie'</span>](<span class="hljs-literal">null</span>, <span class="hljs-string">'counter'</span>);
        } <span class="hljs-keyword">else</span> {
            _0x16b3fe[<span class="hljs-string">'removeCookie'</span>]();
        }
    };
    _0x35976e();
}(_0x26da, <span class="hljs-number">0x140</span>));
<span class="hljs-keyword">var</span> _0x4391 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x1b42d8, _0x57edc8</span>) </span>{
    _0x1b42d8 = _0x1b42d8 - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x2fbeca = _0x26da[_0x1b42d8];
    <span class="hljs-keyword">return</span> _0x2fbeca;
};
<span class="hljs-keyword">var</span> _0x197926 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">var</span> _0x10598f = !![];
    <span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0xffa3b3, _0x7a40f9</span>) </span>{
        <span class="hljs-keyword">var</span> _0x48e571 = _0x10598f ? <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">if</span> (_0x7a40f9) {
                <span class="hljs-keyword">var</span> _0x2194b5 = _0x7a40f9[<span class="hljs-string">'apply'</span>](_0xffa3b3, <span class="hljs-built_in">arguments</span>);
                _0x7a40f9 = <span class="hljs-literal">null</span>;
                <span class="hljs-keyword">return</span> _0x2194b5;
            }
        } : <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{};
        _0x10598f = ![];
        <span class="hljs-keyword">return</span> _0x48e571;
    };
}();
<span class="hljs-keyword">var</span> _0x2c6fd7 = _0x197926(<span class="hljs-keyword">this</span>, <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">var</span> _0x4828bb = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-string">'\x64\x65\x76'</span>;
        },
        _0x35c3bc = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">return</span> <span class="hljs-string">'\x77\x69\x6e\x64\x6f\x77'</span>;
        };
    <span class="hljs-keyword">var</span> _0x456070 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> _0x4576a4 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x5c\x77\x2b\x20\x2a\x5c\x28\x5c\x29\x20\x2a\x7b\x5c\x77\x2b\x20\x2a\x5b\x27\x7c\x22\x5d\x2e\x2b\x5b\x27\x7c\x22\x5d\x3b\x3f\x20\x2a\x7d'</span>);
        <span class="hljs-keyword">return</span> !_0x4576a4[<span class="hljs-string">'\x74\x65\x73\x74'</span>](_0x4828bb[<span class="hljs-string">'\x74\x6f\x53\x74\x72\x69\x6e\x67'</span>]());
    };
    <span class="hljs-keyword">var</span> _0x3fde69 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> _0xabb6f4 = <span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'\x28\x5c\x5c\x5b\x78\x7c\x75\x5d\x28\x5c\x77\x29\x7b\x32\x2c\x34\x7d\x29\x2b'</span>);
        <span class="hljs-keyword">return</span> _0xabb6f4[<span class="hljs-string">'\x74\x65\x73\x74'</span>](_0x35c3bc[<span class="hljs-string">'\x74\x6f\x53\x74\x72\x69\x6e\x67'</span>]());
    };
    <span class="hljs-keyword">var</span> _0x2d9a50 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x58fdb4</span>) </span>{
        <span class="hljs-keyword">var</span> _0x2a6361 = ~<span class="hljs-number">-0x1</span> &gt;&gt; <span class="hljs-number">0x1</span> + <span class="hljs-number">0xff</span> % <span class="hljs-number">0x0</span>;
        <span class="hljs-keyword">if</span> (_0x58fdb4[<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>](<span class="hljs-string">'\x69'</span> === _0x2a6361)) {
            _0xc388c5(_0x58fdb4);
        }
    };
    <span class="hljs-keyword">var</span> _0xc388c5 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x2073d6</span>) </span>{
        <span class="hljs-keyword">var</span> _0x6bb49f = ~<span class="hljs-number">-0x4</span> &gt;&gt; <span class="hljs-number">0x1</span> + <span class="hljs-number">0xff</span> % <span class="hljs-number">0x0</span>;
        <span class="hljs-keyword">if</span> (_0x2073d6[<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>]((!![] + <span class="hljs-string">''</span>)[<span class="hljs-number">0x3</span>]) !== _0x6bb49f) {
            _0x2d9a50(_0x2073d6);
        }
    };
    <span class="hljs-keyword">if</span> (!_0x456070()) {
        <span class="hljs-keyword">if</span> (!_0x3fde69()) {
            _0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\u0435\x78\x4f\x66'</span>);
        } <span class="hljs-keyword">else</span> {
            _0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\x65\x78\x4f\x66'</span>);
        }
    } <span class="hljs-keyword">else</span> {
        _0x2d9a50(<span class="hljs-string">'\x69\x6e\x64\u0435\x78\x4f\x66'</span>);
    }
});
_0x2c6fd7();
<span class="hljs-built_in">console</span>[_0x4391(<span class="hljs-string">'0x0'</span>)](_0x4391(<span class="hljs-string">'0x1'</span>));
</code></pre>
<p>如果把这段代码放到浏览器里面，浏览器会直接卡死无法运行。这样如果有人对代码进行了格式化，就无法正常对代码进行运行和调试，从而起到了保护作用。</p>
<h5>控制流平坦化</h5>
<p>控制流平坦化其实就是将代码的执行逻辑混淆，使其变得复杂难读。其基本思想是将一些逻辑处理块都统一加上一个前驱逻辑块，每个逻辑块都由前驱逻辑块进行条件判断和分发，构成一个个闭环逻辑，导致整个执行逻辑十分复杂难读。</p>
<p>我们通过 controlFlowFlattening 变量可以控制是否开启控制流平坦化，示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
(function(){
    function foo () {
        return function () {
            var sum = 1 + 2;
            console.log(1);
            console.log(2);
            console.log(3);
            console.log(4);
            console.log(5);
            console.log(6);
        }
    }

    foo()();
})();
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>,
  <span class="hljs-attr">controlFlowFlattening</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>输出结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0xbaf1 = [
    <span class="hljs-string">'dZwUe'</span>,
    <span class="hljs-string">'log'</span>,
    <span class="hljs-string">'fXqMu'</span>,
    <span class="hljs-string">'0|1|3|4|6|5|2'</span>,
    <span class="hljs-string">'chYMl'</span>,
    <span class="hljs-string">'IZEsA'</span>,
    <span class="hljs-string">'split'</span>
];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x22d342, _0x4f6332</span>) </span>{
    <span class="hljs-keyword">var</span> _0x43ff59 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x5ad417</span>) </span>{
        <span class="hljs-keyword">while</span> (--_0x5ad417) {
            _0x22d342[<span class="hljs-string">'push'</span>](_0x22d342[<span class="hljs-string">'shift'</span>]());
        }
    };
    _0x43ff59(++_0x4f6332);
}(_0xbaf1, <span class="hljs-number">0x192</span>));
<span class="hljs-keyword">var</span> _0x1a69 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x8d64b1, _0x5e07b3</span>) </span>{
    _0x8d64b1 = _0x8d64b1 - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x300bab = _0xbaf1[_0x8d64b1];
    <span class="hljs-keyword">return</span> _0x300bab;
};
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">var</span> _0x19d8ce = {
        <span class="hljs-string">'chYMl'</span>: _0x1a69(<span class="hljs-string">'0x0'</span>),
        <span class="hljs-string">'IZEsA'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x22e521, _0x298a22</span>) </span>{
            <span class="hljs-keyword">return</span> _0x22e521 + _0x298a22;
        },
        <span class="hljs-string">'fXqMu'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x13124b</span>) </span>{
            <span class="hljs-keyword">return</span> _0x13124b();
        }
    };
    <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_0x4e2ee0</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> _0x118a6a = {
            <span class="hljs-string">'LZAQV'</span>: _0x19d8ce[_0x1a69(<span class="hljs-string">'0x1'</span>)],
            <span class="hljs-string">'dZwUe'</span>: <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x362ef3, _0x352709</span>) </span>{
                <span class="hljs-keyword">return</span> _0x19d8ce[_0x1a69(<span class="hljs-string">'0x2'</span>)](_0x362ef3, _0x352709);
            }
        };
        <span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">var</span> _0x4c336d = _0x118a6a[<span class="hljs-string">'LZAQV'</span>][_0x1a69(<span class="hljs-string">'0x3'</span>)](<span class="hljs-string">'|'</span>), _0x2b6466 = <span class="hljs-number">0x0</span>;
            <span class="hljs-keyword">while</span> (!![]) {
                <span class="hljs-keyword">switch</span> (_0x4c336d[_0x2b6466++]) {
                <span class="hljs-keyword">case</span> <span class="hljs-string">'0'</span>:
                    <span class="hljs-keyword">var</span> _0xbfa3fd = _0x118a6a[_0x1a69(<span class="hljs-string">'0x4'</span>)](<span class="hljs-number">0x1</span>, <span class="hljs-number">0x2</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'1'</span>:
                    <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](<span class="hljs-number">0x1</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'2'</span>:
                    <span class="hljs-built_in">console</span>[_0x1a69(<span class="hljs-string">'0x5'</span>)](<span class="hljs-number">0x6</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'3'</span>:
                    <span class="hljs-built_in">console</span>[_0x1a69(<span class="hljs-string">'0x5'</span>)](<span class="hljs-number">0x2</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'4'</span>:
                    <span class="hljs-built_in">console</span>[_0x1a69(<span class="hljs-string">'0x5'</span>)](<span class="hljs-number">0x3</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'5'</span>:
                    <span class="hljs-built_in">console</span>[_0x1a69(<span class="hljs-string">'0x5'</span>)](<span class="hljs-number">0x5</span>);
                    <span class="hljs-keyword">continue</span>;
                <span class="hljs-keyword">case</span> <span class="hljs-string">'6'</span>:
                    <span class="hljs-built_in">console</span>[_0x1a69(<span class="hljs-string">'0x5'</span>)](<span class="hljs-number">0x4</span>);
                    <span class="hljs-keyword">continue</span>;
                }
                <span class="hljs-keyword">break</span>;
            }
        };
    }
    _0x19d8ce[_0x1a69(<span class="hljs-string">'0x6'</span>)](_0x4e2ee0)();
}());
</code></pre>
<p>可以看到，一些连续的执行逻辑被打破，代码被修改为一个 switch 语句，我们很难再一眼看出多条 console.log 语句的执行顺序了。</p>
<p>如果我们将 controlFlowFlattening 设置为 false 或者不设置，运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x552c = [<span class="hljs-string">'log'</span>];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x4c4fa0, _0x59faa0</span>) </span>{
    <span class="hljs-keyword">var</span> _0xa01786 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x409a37</span>) </span>{
        <span class="hljs-keyword">while</span> (--_0x409a37) {
            _0x4c4fa0[<span class="hljs-string">'push'</span>](_0x4c4fa0[<span class="hljs-string">'shift'</span>]());
        }
    };
    _0xa01786(++_0x59faa0);
}(_0x552c, <span class="hljs-number">0x9b</span>));
<span class="hljs-keyword">var</span> _0x4e63 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x75ea1a, _0x50e176</span>) </span>{
    _0x75ea1a = _0x75ea1a - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x59dc94 = _0x552c[_0x75ea1a];
    <span class="hljs-keyword">return</span> _0x59dc94;
};
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_0x507f38</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">var</span> _0x17fb7e = <span class="hljs-number">0x1</span> + <span class="hljs-number">0x2</span>;
            <span class="hljs-built_in">console</span>[_0x4e63(<span class="hljs-string">'0x0'</span>)](<span class="hljs-number">0x1</span>);
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](<span class="hljs-number">0x2</span>);
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](<span class="hljs-number">0x3</span>);
            <span class="hljs-built_in">console</span>[_0x4e63(<span class="hljs-string">'0x0'</span>)](<span class="hljs-number">0x4</span>);
            <span class="hljs-built_in">console</span>[_0x4e63(<span class="hljs-string">'0x0'</span>)](<span class="hljs-number">0x5</span>);
            <span class="hljs-built_in">console</span>[_0x4e63(<span class="hljs-string">'0x0'</span>)](<span class="hljs-number">0x6</span>);
        };
    }
    _0x507f38()();
}());
</code></pre>
<p>可以看到，这里仍然保留了原始的 console.log 执行逻辑。</p>
<p>因此，使用控制流扁平化可以使得执行逻辑更加复杂难读，目前非常多的前端混淆都会加上这个选项。</p>
<p>但启用控制流扁平化之后，代码的执行时间会变长，最长达 1.5 倍之多。</p>
<p>另外我们还能使用 controlFlowFlatteningThreshold 这个参数来控制比例，取值范围是 0 到 1，默认 0.75，如果设置为 0，那相当于 controlFlowFlattening 设置为 false，即不开启控制流扁平化 。</p>
<h5>僵尸代码注入</h5>
<p>僵尸代码即不会被执行的代码或对上下文没有任何影响的代码，注入之后可以对现有的 JavaScript 代码的阅读形成干扰。我们可以使用 deadCodeInjection 参数开启这个选项，默认为 false。<br>
示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
(function(){
    if (true) {
        var foo = function () {
            console.log('abc');
            console.log('cde');
            console.log('efg');
            console.log('hij');
        };

        var bar = function () {
            console.log('klm');
            console.log('nop');
            console.log('qrs');
        };

        var baz = function () {
            console.log('tuv');
            console.log('wxy');
            console.log('z');
        };

        foo();
        bar();
        baz();
    }
})();
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>,
  <span class="hljs-attr">deadCodeInjection</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x5024 = [
    <span class="hljs-string">'zaU'</span>,
    <span class="hljs-string">'log'</span>,
    <span class="hljs-string">'tuv'</span>,
    <span class="hljs-string">'wxy'</span>,
    <span class="hljs-string">'abc'</span>,
    <span class="hljs-string">'cde'</span>,
    <span class="hljs-string">'efg'</span>,
    <span class="hljs-string">'hij'</span>,
    <span class="hljs-string">'QhG'</span>,
    <span class="hljs-string">'TeI'</span>,
    <span class="hljs-string">'klm'</span>,
    <span class="hljs-string">'nop'</span>,
    <span class="hljs-string">'qrs'</span>,
    <span class="hljs-string">'bZd'</span>,
    <span class="hljs-string">'HMx'</span>
];
<span class="hljs-keyword">var</span> _0x4502 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x1254b1, _0x583689</span>) </span>{
    _0x1254b1 = _0x1254b1 - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x529b49 = _0x5024[_0x1254b1];
    <span class="hljs-keyword">return</span> _0x529b49;
};
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">if</span> (!![]) {
        <span class="hljs-keyword">var</span> _0x16c18d = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">if</span> (_0x4502(<span class="hljs-string">'0x0'</span>) !== _0x4502(<span class="hljs-string">'0x0'</span>)) {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x2'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x3'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](<span class="hljs-string">'z'</span>);
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x4'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x5'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x6'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x7'</span>));
            }
        };
        <span class="hljs-keyword">var</span> _0x1f7292 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">if</span> (_0x4502(<span class="hljs-string">'0x8'</span>) === _0x4502(<span class="hljs-string">'0x9'</span>)) {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xa'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xb'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xc'</span>));
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xa'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xb'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0xc'</span>));
            }
        };
        <span class="hljs-keyword">var</span> _0x33b212 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-keyword">if</span> (_0x4502(<span class="hljs-string">'0xd'</span>) !== _0x4502(<span class="hljs-string">'0xe'</span>)) {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x2'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x3'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](<span class="hljs-string">'z'</span>);
            } <span class="hljs-keyword">else</span> {
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x4'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x5'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x6'</span>));
                <span class="hljs-built_in">console</span>[_0x4502(<span class="hljs-string">'0x1'</span>)](_0x4502(<span class="hljs-string">'0x7'</span>));
            }
        };
        _0x16c18d();
        _0x1f7292();
        _0x33b212();
    }
}());
</code></pre>
<p>可见这里增加了一些不会执行到的逻辑区块内容。</p>
<p>如果将 deadCodeInjection 设置为 false 或者不设置，运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x402a = [
    <span class="hljs-string">'qrs'</span>,
    <span class="hljs-string">'wxy'</span>,
    <span class="hljs-string">'log'</span>,
    <span class="hljs-string">'abc'</span>,
    <span class="hljs-string">'cde'</span>,
    <span class="hljs-string">'efg'</span>,
    <span class="hljs-string">'hij'</span>,
    <span class="hljs-string">'nop'</span>
];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x57239e, _0x4747e8</span>) </span>{
    <span class="hljs-keyword">var</span> _0x3998cd = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x34a502</span>) </span>{
        <span class="hljs-keyword">while</span> (--_0x34a502) {
            _0x57239e[<span class="hljs-string">'push'</span>](_0x57239e[<span class="hljs-string">'shift'</span>]());
        }
    };
    _0x3998cd(++_0x4747e8);
}(_0x402a, <span class="hljs-number">0x162</span>));
<span class="hljs-keyword">var</span> _0x5356 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x2f2c10, _0x2878a6</span>) </span>{
    _0x2f2c10 = _0x2f2c10 - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x4cfe02 = _0x402a[_0x2f2c10];
    <span class="hljs-keyword">return</span> _0x4cfe02;
};
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">if</span> (!![]) {
        <span class="hljs-keyword">var</span> _0x60edc1 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](_0x5356(<span class="hljs-string">'0x1'</span>));
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](_0x5356(<span class="hljs-string">'0x2'</span>));
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](_0x5356(<span class="hljs-string">'0x3'</span>));
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](_0x5356(<span class="hljs-string">'0x4'</span>));
        };
        <span class="hljs-keyword">var</span> _0x56405f = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](<span class="hljs-string">'klm'</span>);
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](_0x5356(<span class="hljs-string">'0x5'</span>));
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](_0x5356(<span class="hljs-string">'0x6'</span>));
        };
        <span class="hljs-keyword">var</span> _0x332d12 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](<span class="hljs-string">'tuv'</span>);
            <span class="hljs-built_in">console</span>[_0x5356(<span class="hljs-string">'0x0'</span>)](_0x5356(<span class="hljs-string">'0x7'</span>));
            <span class="hljs-built_in">console</span>[<span class="hljs-string">'log'</span>](<span class="hljs-string">'z'</span>);
        };
        _0x60edc1();
        _0x56405f();
        _0x332d12();
    }
}());
</code></pre>
<p>另外我们还可以通过设置 deadCodeInjectionThreshold 参数来控制僵尸代码注入的比例，取值 0 到 1，默认是 0.4。</p>
<p>僵尸代码可以起到一定的干扰作用，所以在有必要的时候也可以注入。</p>
<h5>对象键名替换</h5>
<p>如果是一个对象，可以使用 transformObjectKeys 来对对象的键值进行替换，示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
(function(){
    var object = {
        foo: 'test1',
        bar: {
            baz: 'test2'
        }
    };
})(); 
`</span>
<span class="hljs-keyword">const</span> options = {
  <span class="hljs-attr">compact</span>: <span class="hljs-literal">false</span>,
  <span class="hljs-attr">transformObjectKeys</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>输出结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x7a5d = [
    <span class="hljs-string">'bar'</span>,
    <span class="hljs-string">'test2'</span>,
    <span class="hljs-string">'test1'</span>
];
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x59fec5, _0x2e4fac</span>) </span>{
    <span class="hljs-keyword">var</span> _0x231e7a = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x46f33e</span>) </span>{
        <span class="hljs-keyword">while</span> (--_0x46f33e) {
            _0x59fec5[<span class="hljs-string">'push'</span>](_0x59fec5[<span class="hljs-string">'shift'</span>]());
        }
    };
    _0x231e7a(++_0x2e4fac);
}(_0x7a5d, <span class="hljs-number">0x167</span>));
<span class="hljs-keyword">var</span> _0x3bc4 = <span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params">_0x309ad3, _0x22d5ac</span>) </span>{
    _0x309ad3 = _0x309ad3 - <span class="hljs-number">0x0</span>;
    <span class="hljs-keyword">var</span> _0x3a034e = _0x7a5d[_0x309ad3];
    <span class="hljs-keyword">return</span> _0x3a034e;
};
(<span class="hljs-function"><span class="hljs-keyword">function</span> (<span class="hljs-params"></span>) </span>{
    <span class="hljs-keyword">var</span> _0x9f1fd1 = {};
    _0x9f1fd1[<span class="hljs-string">'foo'</span>] = _0x3bc4(<span class="hljs-string">'0x0'</span>);
    _0x9f1fd1[_0x3bc4(<span class="hljs-string">'0x1'</span>)] = {};
    _0x9f1fd1[_0x3bc4(<span class="hljs-string">'0x1'</span>)][<span class="hljs-string">'baz'</span>] = _0x3bc4(<span class="hljs-string">'0x2'</span>);
}());
</code></pre>
<p>可以看到，Object 的变量名被替换为了特殊的变量，这也可以起到一定的防护作用。</p>
<h5>禁用控制台输出</h5>
<p>可以使用 disableConsoleOutput 来禁用掉 console.log 输出功能，加大调试难度，示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
console.log('hello world')
`</span>
<span class="hljs-keyword">const</span> options = {
    <span class="hljs-attr">disableConsoleOutput</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x3a39=[<span class="hljs-string">'debug'</span>,<span class="hljs-string">'info'</span>,<span class="hljs-string">'error'</span>,<span class="hljs-string">'exception'</span>,<span class="hljs-string">'trace'</span>,<span class="hljs-string">'hello\x20world'</span>,<span class="hljs-string">'apply'</span>,<span class="hljs-string">'{}.constructor(\x22return\x20this\x22)(\x20)'</span>,<span class="hljs-string">'console'</span>,<span class="hljs-string">'log'</span>,<span class="hljs-string">'warn'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x2a157a,_0x5d9d3b</span>)</span>{<span class="hljs-keyword">var</span> _0x488e2c=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x5bcb73</span>)</span>{<span class="hljs-keyword">while</span>(--_0x5bcb73){_0x2a157a[<span class="hljs-string">'push'</span>](_0x2a157a[<span class="hljs-string">'shift'</span>]());}};_0x488e2c(++_0x5d9d3b);}(_0x3a39,<span class="hljs-number">0x10e</span>));<span class="hljs-keyword">var</span> _0x5bff=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x43bdfc,_0x52e4c6</span>)</span>{_0x43bdfc=_0x43bdfc<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0xb67384=_0x3a39[_0x43bdfc];<span class="hljs-keyword">return</span> _0xb67384;};<span class="hljs-keyword">var</span> _0x349b01=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x1f484b=!![];<span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x5efe0d,_0x33db62</span>)</span>{<span class="hljs-keyword">var</span> _0x20bcd2=_0x1f484b?<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">if</span>(_0x33db62){<span class="hljs-keyword">var</span> _0x77054c=_0x33db62[_0x5bff(<span class="hljs-string">'0x0'</span>)](_0x5efe0d,<span class="hljs-built_in">arguments</span>);_0x33db62=<span class="hljs-literal">null</span>;<span class="hljs-keyword">return</span> _0x77054c;}}:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{};_0x1f484b=![];<span class="hljs-keyword">return</span> _0x20bcd2;};}();<span class="hljs-keyword">var</span> _0x19f538=_0x349b01(<span class="hljs-keyword">this</span>,<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x7ab6e4=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{};<span class="hljs-keyword">var</span> _0x157bff;<span class="hljs-keyword">try</span>{<span class="hljs-keyword">var</span> _0x5e672c=<span class="hljs-built_in">Function</span>(<span class="hljs-string">'return\x20(function()\x20'</span>+_0x5bff(<span class="hljs-string">'0x1'</span>)+<span class="hljs-string">');'</span>);_0x157bff=_0x5e672c();}<span class="hljs-keyword">catch</span>(_0x11028d){_0x157bff=<span class="hljs-built_in">window</span>;}<span class="hljs-keyword">if</span>(!_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)]){_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)]=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x7ab6e4</span>)</span>{<span class="hljs-keyword">var</span> _0x5a8d9e={};_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x3'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x4'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x5'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x6'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x7'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x8'</span>)]=_0x7ab6e4;_0x5a8d9e[_0x5bff(<span class="hljs-string">'0x9'</span>)]=_0x7ab6e4;<span class="hljs-keyword">return</span> _0x5a8d9e;}(_0x7ab6e4);}<span class="hljs-keyword">else</span>{_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x3'</span>)]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x4'</span>)]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][<span class="hljs-string">'debug'</span>]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x6'</span>)]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x7'</span>)]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x8'</span>)]=_0x7ab6e4;_0x157bff[_0x5bff(<span class="hljs-string">'0x2'</span>)][_0x5bff(<span class="hljs-string">'0x9'</span>)]=_0x7ab6e4;}});_0x19f538();<span class="hljs-built_in">console</span>[_0x5bff(<span class="hljs-string">'0x3'</span>)](_0x5bff(<span class="hljs-string">'0xa'</span>));
</code></pre>
<p>此时，我们如果执行这段代码，发现是没有任何输出的，这里实际上就是将 console 的一些功能禁用了，加大了调试难度。</p>
<h5>调试保护</h5>
<p>我们可以使用 debugProtection 来禁用调试模式，进入无限 Debug 模式。另外我们还可以使用 debugProtectionInterval 来启用无限 Debug 的间隔，使得代码在调试过程中会不断进入断点模式，无法顺畅执行。<br>
示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
for (let i = 0; i &lt; 5; i ++) {
    console.log('i', i)
}
`</span>
<span class="hljs-keyword">const</span> options = {
    <span class="hljs-attr">debugProtection</span>: <span class="hljs-literal">true</span>
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x41d0=[<span class="hljs-string">'action'</span>,<span class="hljs-string">'debu'</span>,<span class="hljs-string">'stateObject'</span>,<span class="hljs-string">'function\x20*\x5c(\x20*\x5c)'</span>,<span class="hljs-string">'\x5c+\x5c+\x20*(?:_0x(?:[a-f0-9]){4,6}|(?:\x5cb|\x5cd)[a-z0-9]{1,4}(?:\x5cb|\x5cd))'</span>,<span class="hljs-string">'init'</span>,<span class="hljs-string">'test'</span>,<span class="hljs-string">'chain'</span>,<span class="hljs-string">'input'</span>,<span class="hljs-string">'log'</span>,<span class="hljs-string">'string'</span>,<span class="hljs-string">'constructor'</span>,<span class="hljs-string">'while\x20(true)\x20{}'</span>,<span class="hljs-string">'apply'</span>,<span class="hljs-string">'gger'</span>,<span class="hljs-string">'call'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x69147e,_0x180e03</span>)</span>{<span class="hljs-keyword">var</span> _0x2cc589=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x18d18c</span>)</span>{<span class="hljs-keyword">while</span>(--_0x18d18c){_0x69147e[<span class="hljs-string">'push'</span>](_0x69147e[<span class="hljs-string">'shift'</span>]());}};_0x2cc589(++_0x180e03);}(_0x41d0,<span class="hljs-number">0x153</span>));<span class="hljs-keyword">var</span> _0x16d2=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x3d813e,_0x59f7b2</span>)</span>{_0x3d813e=_0x3d813e<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x228f98=_0x41d0[_0x3d813e];<span class="hljs-keyword">return</span> _0x228f98;};<span class="hljs-keyword">var</span> _0x241eee=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0xeb17=!![];<span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x5caffe,_0x2bb267</span>)</span>{<span class="hljs-keyword">var</span> _0x16e1bf=_0xeb17?<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">if</span>(_0x2bb267){<span class="hljs-keyword">var</span> _0x573619=_0x2bb267[<span class="hljs-string">'apply'</span>](_0x5caffe,<span class="hljs-built_in">arguments</span>);_0x2bb267=<span class="hljs-literal">null</span>;<span class="hljs-keyword">return</span> _0x573619;}}:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{};_0xeb17=![];<span class="hljs-keyword">return</span> _0x16e1bf;};}();(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{_0x241eee(<span class="hljs-keyword">this</span>,<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x5de4a4=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(_0x16d2(<span class="hljs-string">'0x0'</span>));<span class="hljs-keyword">var</span> _0x4a170e=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(_0x16d2(<span class="hljs-string">'0x1'</span>),<span class="hljs-string">'i'</span>);<span class="hljs-keyword">var</span> _0x5351d7=_0x227210(_0x16d2(<span class="hljs-string">'0x2'</span>));<span class="hljs-keyword">if</span>(!_0x5de4a4[_0x16d2(<span class="hljs-string">'0x3'</span>)](_0x5351d7+_0x16d2(<span class="hljs-string">'0x4'</span>))||!_0x4a170e[_0x16d2(<span class="hljs-string">'0x3'</span>)](_0x5351d7+_0x16d2(<span class="hljs-string">'0x5'</span>))){_0x5351d7(<span class="hljs-string">'0'</span>);}<span class="hljs-keyword">else</span>{_0x227210();}})();}());<span class="hljs-keyword">for</span>(<span class="hljs-keyword">let</span> i=<span class="hljs-number">0x0</span>;i&lt;<span class="hljs-number">0x5</span>;i++){<span class="hljs-built_in">console</span>[_0x16d2(<span class="hljs-string">'0x6'</span>)](<span class="hljs-string">'i'</span>,i);}<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_0x227210</span>(<span class="hljs-params">_0x30bc32</span>)</span>{<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">_0x1971c7</span>(<span class="hljs-params">_0x19628c</span>)</span>{<span class="hljs-keyword">if</span>(<span class="hljs-keyword">typeof</span> _0x19628c===_0x16d2(<span class="hljs-string">'0x7'</span>)){<span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x3718f7</span>)</span>{}[_0x16d2(<span class="hljs-string">'0x8'</span>)](_0x16d2(<span class="hljs-string">'0x9'</span>))[_0x16d2(<span class="hljs-string">'0xa'</span>)](<span class="hljs-string">'counter'</span>);}<span class="hljs-keyword">else</span>{<span class="hljs-keyword">if</span>((<span class="hljs-string">''</span>+_0x19628c/_0x19628c)[<span class="hljs-string">'length'</span>]!==<span class="hljs-number">0x1</span>||_0x19628c%<span class="hljs-number">0x14</span>===<span class="hljs-number">0x0</span>){(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span>!![];}[_0x16d2(<span class="hljs-string">'0x8'</span>)](<span class="hljs-string">'debu'</span>+_0x16d2(<span class="hljs-string">'0xb'</span>))[_0x16d2(<span class="hljs-string">'0xc'</span>)](_0x16d2(<span class="hljs-string">'0xd'</span>)));}<span class="hljs-keyword">else</span>{(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span>![];}[_0x16d2(<span class="hljs-string">'0x8'</span>)](_0x16d2(<span class="hljs-string">'0xe'</span>)+_0x16d2(<span class="hljs-string">'0xb'</span>))[_0x16d2(<span class="hljs-string">'0xa'</span>)](_0x16d2(<span class="hljs-string">'0xf'</span>)));}}_0x1971c7(++_0x19628c);}<span class="hljs-keyword">try</span>{<span class="hljs-keyword">if</span>(_0x30bc32){<span class="hljs-keyword">return</span> _0x1971c7;}<span class="hljs-keyword">else</span>{_0x1971c7(<span class="hljs-number">0x0</span>);}}<span class="hljs-keyword">catch</span>(_0x58d434){}}
</code></pre>
<p>如果我们将代码粘贴到控制台，其会不断跳到 debugger 代码的位置，无法顺畅执行。</p>
<h5>域名锁定</h5>
<p>我们可以通过控制 domainLock 来控制 JavaScript 代码只能在特定域名下运行，这样就可以降低被模拟的风险。</p>
<p>示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">const</span> code = <span class="hljs-string">`
console.log('hello world')
`</span>
<span class="hljs-keyword">const</span> options = {
    <span class="hljs-attr">domainLock</span>: [<span class="hljs-string">'cuiqingcai.com'</span>]
}
</code></pre>
<p>运行结果如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> _0x3203=[<span class="hljs-string">'apply'</span>,<span class="hljs-string">'return\x20(function()\x20'</span>,<span class="hljs-string">'{}.constructor(\x22return\x20this\x22)(\x20)'</span>,<span class="hljs-string">'item'</span>,<span class="hljs-string">'attribute'</span>,<span class="hljs-string">'value'</span>,<span class="hljs-string">'replace'</span>,<span class="hljs-string">'length'</span>,<span class="hljs-string">'charCodeAt'</span>,<span class="hljs-string">'log'</span>,<span class="hljs-string">'hello\x20world'</span>];(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x2ed22c,_0x3ad370</span>)</span>{<span class="hljs-keyword">var</span> _0x49dc54=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0x53a786</span>)</span>{<span class="hljs-keyword">while</span>(--_0x53a786){_0x2ed22c[<span class="hljs-string">'push'</span>](_0x2ed22c[<span class="hljs-string">'shift'</span>]());}};_0x49dc54(++_0x3ad370);}(_0x3203,<span class="hljs-number">0x155</span>));<span class="hljs-keyword">var</span> _0x5b38=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0xd7780b,_0x19c0f2</span>)</span>{_0xd7780b=_0xd7780b<span class="hljs-number">-0x0</span>;<span class="hljs-keyword">var</span> _0x2d2f44=_0x3203[_0xd7780b];<span class="hljs-keyword">return</span> _0x2d2f44;};<span class="hljs-keyword">var</span> _0x485919=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x5cf798=!![];<span class="hljs-keyword">return</span> <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">_0xd1fa29,_0x2ed646</span>)</span>{<span class="hljs-keyword">var</span> _0x56abf=_0x5cf798?<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">if</span>(_0x2ed646){<span class="hljs-keyword">var</span> _0x33af63=_0x2ed646[_0x5b38(<span class="hljs-string">'0x0'</span>)](_0xd1fa29,<span class="hljs-built_in">arguments</span>);_0x2ed646=<span class="hljs-literal">null</span>;<span class="hljs-keyword">return</span> _0x33af63;}}:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{};_0x5cf798=![];<span class="hljs-keyword">return</span> _0x56abf;};}();<span class="hljs-keyword">var</span> _0x67dcc8=_0x485919(<span class="hljs-keyword">this</span>,<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">var</span> _0x276a31;<span class="hljs-keyword">try</span>{<span class="hljs-keyword">var</span> _0x5c8be2=<span class="hljs-built_in">Function</span>(_0x5b38(<span class="hljs-string">'0x1'</span>)+_0x5b38(<span class="hljs-string">'0x2'</span>)+<span class="hljs-string">');'</span>);_0x276a31=_0x5c8be2();}<span class="hljs-keyword">catch</span>(_0x5f1c00){_0x276a31=<span class="hljs-built_in">window</span>;}<span class="hljs-keyword">var</span> _0x254a0d=<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">return</span>{<span class="hljs-string">'key'</span>:_0x5b38(<span class="hljs-string">'0x3'</span>),<span class="hljs-string">'value'</span>:_0x5b38(<span class="hljs-string">'0x4'</span>),<span class="hljs-string">'getAttribute'</span>:<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x5cc3c7=<span class="hljs-number">0x0</span>;_0x5cc3c7&lt;<span class="hljs-number">0x3e8</span>;_0x5cc3c7--){<span class="hljs-keyword">var</span> _0x35b30b=_0x5cc3c7&gt;<span class="hljs-number">0x0</span>;<span class="hljs-keyword">switch</span>(_0x35b30b){<span class="hljs-keyword">case</span>!![]:<span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>[_0x5b38(<span class="hljs-string">'0x3'</span>)]+<span class="hljs-string">'_'</span>+<span class="hljs-keyword">this</span>[_0x5b38(<span class="hljs-string">'0x5'</span>)]+<span class="hljs-string">'_'</span>+_0x5cc3c7;<span class="hljs-keyword">default</span>:<span class="hljs-keyword">this</span>[_0x5b38(<span class="hljs-string">'0x3'</span>)]+<span class="hljs-string">'_'</span>+<span class="hljs-keyword">this</span>[_0x5b38(<span class="hljs-string">'0x5'</span>)];}}}()};};<span class="hljs-keyword">var</span> _0x3b375a=<span class="hljs-keyword">new</span> <span class="hljs-built_in">RegExp</span>(<span class="hljs-string">'[QLCIKYkCFzdWpzRAXMhxJOYpTpYWJHPll]'</span>,<span class="hljs-string">'g'</span>);<span class="hljs-keyword">var</span> _0x5a94d2=<span class="hljs-string">'cuQLiqiCInKYkgCFzdWcpzRAaXMi.hcoxmJOYpTpYWJHPll'</span>[_0x5b38(<span class="hljs-string">'0x6'</span>)](_0x3b375a,<span class="hljs-string">''</span>)[<span class="hljs-string">'split'</span>](<span class="hljs-string">';'</span>);<span class="hljs-keyword">var</span> _0x5c0da2;<span class="hljs-keyword">var</span> _0x19ad5d;<span class="hljs-keyword">var</span> _0x5992ca;<span class="hljs-keyword">var</span> _0x40bd39;<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x5cad1 <span class="hljs-keyword">in</span> _0x276a31){<span class="hljs-keyword">if</span>(_0x5cad1[_0x5b38(<span class="hljs-string">'0x7'</span>)]==<span class="hljs-number">0x8</span>&amp;&amp;_0x5cad1[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x7</span>)==<span class="hljs-number">0x74</span>&amp;&amp;_0x5cad1[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x5</span>)==<span class="hljs-number">0x65</span>&amp;&amp;_0x5cad1[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x3</span>)==<span class="hljs-number">0x75</span>&amp;&amp;_0x5cad1[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x0</span>)==<span class="hljs-number">0x64</span>){_0x5c0da2=_0x5cad1;<span class="hljs-keyword">break</span>;}}<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x29551 <span class="hljs-keyword">in</span> _0x276a31[_0x5c0da2]){<span class="hljs-keyword">if</span>(_0x29551[_0x5b38(<span class="hljs-string">'0x7'</span>)]==<span class="hljs-number">0x6</span>&amp;&amp;_0x29551[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x5</span>)==<span class="hljs-number">0x6e</span>&amp;&amp;_0x29551[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x0</span>)==<span class="hljs-number">0x64</span>){_0x19ad5d=_0x29551;<span class="hljs-keyword">break</span>;}}<span class="hljs-keyword">if</span>(!(<span class="hljs-string">'~'</span>&gt;_0x19ad5d)){<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x2b71bd <span class="hljs-keyword">in</span> _0x276a31[_0x5c0da2]){<span class="hljs-keyword">if</span>(_0x2b71bd[_0x5b38(<span class="hljs-string">'0x7'</span>)]==<span class="hljs-number">0x8</span>&amp;&amp;_0x2b71bd[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x7</span>)==<span class="hljs-number">0x6e</span>&amp;&amp;_0x2b71bd[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x0</span>)==<span class="hljs-number">0x6c</span>){_0x5992ca=_0x2b71bd;<span class="hljs-keyword">break</span>;}}<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x397f55 <span class="hljs-keyword">in</span> _0x276a31[_0x5c0da2][_0x5992ca]){<span class="hljs-keyword">if</span>(_0x397f55[<span class="hljs-string">'length'</span>]==<span class="hljs-number">0x8</span>&amp;&amp;_0x397f55[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x7</span>)==<span class="hljs-number">0x65</span>&amp;&amp;_0x397f55[_0x5b38(<span class="hljs-string">'0x8'</span>)](<span class="hljs-number">0x0</span>)==<span class="hljs-number">0x68</span>){_0x40bd39=_0x397f55;<span class="hljs-keyword">break</span>;}}}<span class="hljs-keyword">if</span>(!_0x5c0da2||!_0x276a31[_0x5c0da2]){<span class="hljs-keyword">return</span>;}<span class="hljs-keyword">var</span> _0x5f19be=_0x276a31[_0x5c0da2][_0x19ad5d];<span class="hljs-keyword">var</span> _0x674f76=!!_0x276a31[_0x5c0da2][_0x5992ca]&amp;&amp;_0x276a31[_0x5c0da2][_0x5992ca][_0x40bd39];<span class="hljs-keyword">var</span> _0x5e1b34=_0x5f19be||_0x674f76;<span class="hljs-keyword">if</span>(!_0x5e1b34){<span class="hljs-keyword">return</span>;}<span class="hljs-keyword">var</span> _0x593394=![];<span class="hljs-keyword">for</span>(<span class="hljs-keyword">var</span> _0x479239=<span class="hljs-number">0x0</span>;_0x479239&lt;_0x5a94d2[<span class="hljs-string">'length'</span>];_0x479239++){<span class="hljs-keyword">var</span> _0x19ad5d=_0x5a94d2[_0x479239];<span class="hljs-keyword">var</span> _0x112c24=_0x5e1b34[<span class="hljs-string">'length'</span>]-_0x19ad5d[<span class="hljs-string">'length'</span>];<span class="hljs-keyword">var</span> _0x51731c=_0x5e1b34[<span class="hljs-string">'indexOf'</span>](_0x19ad5d,_0x112c24);<span class="hljs-keyword">var</span> _0x173191=_0x51731c!==<span class="hljs-number">-0x1</span>&amp;&amp;_0x51731c===_0x112c24;<span class="hljs-keyword">if</span>(_0x173191){<span class="hljs-keyword">if</span>(_0x5e1b34[<span class="hljs-string">'length'</span>]==_0x19ad5d[_0x5b38(<span class="hljs-string">'0x7'</span>)]||_0x19ad5d[<span class="hljs-string">'indexOf'</span>](<span class="hljs-string">'.'</span>)===<span class="hljs-number">0x0</span>){_0x593394=!![];}}}<span class="hljs-keyword">if</span>(!_0x593394){data;}<span class="hljs-keyword">else</span>{<span class="hljs-keyword">return</span>;}_0x254a0d();});_0x67dcc8();<span class="hljs-built_in">console</span>[_0x5b38(<span class="hljs-string">'0x9'</span>)](_0x5b38(<span class="hljs-string">'0xa'</span>));
</code></pre>
<p>这段代码就只能在指定域名 cuiqingcai.com 下运行，不能在其他网站运行，不信你可以试试。</p>
<h5>特殊编码</h5>
<p>另外还有一些特殊的工具包，如使用 aaencode、jjencode、jsfuck 等工具对代码进行混淆和编码。</p>
<p>示例如下：</p>
<pre><code data-language="js" class="lang-js"><span class="hljs-keyword">var</span> a = <span class="hljs-number">1</span>
</code></pre>
<p>jsfuck 的结果：</p>
<pre><code data-language="js" class="lang-js">[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]([][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+([][[]]+[])[+[]]+([][[]]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(![]+[])[!+[]+!![]+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+([]+[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[!+[]+!![]]+([]+{})[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][[]]+[])[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]])(!+[]+!![]+!![]+!![]+!![]))[!+[]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]])(!+[]+!![]+!![]+!![])([][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(![]+[])[!+[]+!![]+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+([]+[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[!+[]+!![]]+([]+{})[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][[]]+[])[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]])(!+[]+!![]+!![]+!![]+!![]))[!+[]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]])(!+[]+!![]+!![]+!![]+!![])(([]+{})[+[]])[+[]]+(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+[])+(!+[]+!![]+!![]+!![]+!![]+!![]+[]))+(+{}+[])[+!![]]+(!![]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+([][[]]+[])[+[]]+([][[]]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(![]+[])[!+[]+!![]+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+([]+[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[!+[]+!![]]+([]+{})[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][[]]+[])[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]])(!+[]+!![]+!![]+!![]+!![]))[!+[]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]])(!+[]+!![]+!![]+!![])([][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(![]+[])[!+[]+!![]+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+([]+[][(![]+[])[!+[]+!![]+!![]]+([]+{})[+!![]]+(!![]+[])[+!![]]+(!![]+[])[+[]]][([]+{})[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]]+(![]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+[]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(!![]+[])[+[]]+([]+{})[+!![]]+(!![]+[])[+!![]]]((!![]+[])[+!![]]+([][[]]+[])[!+[]+!![]+!![]]+(!![]+[])[+[]]+([][[]]+[])[+[]]+(!![]+[])[+!![]]+([][[]]+[])[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(![]+[])[!+[]+!![]]+([]+{})[+!![]]+([]+{})[!+[]+!![]+!![]+!![]+!![]]+(+{}+[])[+!![]]+(!![]+[])[+[]]+([][[]]+[])[!+[]+!![]+!![]+!![]+!![]]+([]+{})[+!![]]+([][[]]+[])[+!![]])(!+[]+!![]+!![]+!![]+!![]))[!+[]+!![]+!![]]+([][[]]+[])[!+[]+!![]+!![]])(!+[]+!![]+!![]+!![]+!![])(([]+{})[+[]])[+[]]+(!+[]+!![]+!![]+[])+([][[]]+[])[!+[]+!![]])+([]+{})[!+[]+!![]+!![]+!![]+!![]+!![]+!![]]+(+!![]+[]))(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![])
</code></pre>
<p>aaencode 的结果：</p>
<pre><code data-language="js" class="lang-js">ﾟωﾟﾉ= <span class="hljs-regexp">/｀ｍ´）ﾉ ~┻━┻ /</span> [<span class="hljs-string">'_'</span>]; o=(ﾟｰﾟ) =_=<span class="hljs-number">3</span>; c=(ﾟΘﾟ) =(ﾟｰﾟ)-(ﾟｰﾟ); (ﾟДﾟ) =(ﾟΘﾟ)= (o^_^o)/ (o^_^o);(ﾟДﾟ)={ﾟΘﾟ: <span class="hljs-string">'_'</span> ,ﾟωﾟﾉ : ((ﾟωﾟﾉ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟΘﾟ] ,ﾟｰﾟﾉ :(ﾟωﾟﾉ+ <span class="hljs-string">'_'</span>)[o^_^o -(ﾟΘﾟ)] ,ﾟДﾟﾉ:((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>)[ﾟｰﾟ] }; (ﾟДﾟ) [ﾟΘﾟ] =((ﾟωﾟﾉ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [c^_^o];(ﾟДﾟ) [<span class="hljs-string">'c'</span>] = ((ﾟДﾟ)+<span class="hljs-string">'_'</span>) [ (ﾟｰﾟ)+(ﾟｰﾟ)-(ﾟΘﾟ) ];(ﾟДﾟ) [<span class="hljs-string">'o'</span>] = ((ﾟДﾟ)+<span class="hljs-string">'_'</span>) [ﾟΘﾟ];(ﾟoﾟ)=(ﾟДﾟ) [<span class="hljs-string">'c'</span>]+(ﾟДﾟ) [<span class="hljs-string">'o'</span>]+(ﾟωﾟﾉ +<span class="hljs-string">'_'</span>)[ﾟΘﾟ]+ ((ﾟωﾟﾉ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟｰﾟ] + ((ﾟДﾟ) +<span class="hljs-string">'_'</span>) [(ﾟｰﾟ)+(ﾟｰﾟ)]+ ((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟΘﾟ]+((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [(ﾟｰﾟ) - (ﾟΘﾟ)]+(ﾟДﾟ) [<span class="hljs-string">'c'</span>]+((ﾟДﾟ)+<span class="hljs-string">'_'</span>) [(ﾟｰﾟ)+(ﾟｰﾟ)]+ (ﾟДﾟ) [<span class="hljs-string">'o'</span>]+((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟΘﾟ];(ﾟДﾟ) [<span class="hljs-string">'_'</span>] =(o^_^o) [ﾟoﾟ] [ﾟoﾟ];(ﾟεﾟ)=((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟΘﾟ]+ (ﾟДﾟ) .ﾟДﾟﾉ+((ﾟДﾟ)+<span class="hljs-string">'_'</span>) [(ﾟｰﾟ) + (ﾟｰﾟ)]+((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [o^_^o -ﾟΘﾟ]+((ﾟｰﾟ==<span class="hljs-number">3</span>) +<span class="hljs-string">'_'</span>) [ﾟΘﾟ]+ (ﾟωﾟﾉ +<span class="hljs-string">'_'</span>) [ﾟΘﾟ]; (ﾟｰﾟ)+=(ﾟΘﾟ); (ﾟДﾟ)[ﾟεﾟ]=<span class="hljs-string">'\\'</span>; (ﾟДﾟ).ﾟΘﾟﾉ=(ﾟДﾟ+ ﾟｰﾟ)[o^_^o -(ﾟΘﾟ)];(oﾟｰﾟo)=(ﾟωﾟﾉ +<span class="hljs-string">'_'</span>)[c^_^o];(ﾟДﾟ) [ﾟoﾟ]=<span class="hljs-string">'\"'</span>;(ﾟДﾟ) [<span class="hljs-string">'_'</span>] ( (ﾟДﾟ) [<span class="hljs-string">'_'</span>] (ﾟεﾟ+(ﾟДﾟ)[ﾟoﾟ]+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) +(o^_^o))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ ((o^_^o) +(o^_^o))+ ((o^_^o) - (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟΘﾟ)+ (ﾟｰﾟ)+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ) + (o^_^o))+ ((ﾟｰﾟ) + (ﾟΘﾟ))+ (ﾟДﾟ)[ﾟεﾟ]+(ﾟｰﾟ)+ (c^_^o)+ (ﾟДﾟ)[ﾟεﾟ]+((o^_^o) +(o^_^o))+ (ﾟΘﾟ)+ (ﾟДﾟ)[ﾟoﾟ])(ﾟΘﾟ))((ﾟΘﾟ)+(ﾟДﾟ)[ﾟεﾟ]+((ﾟｰﾟ)+(ﾟΘﾟ))+(ﾟΘﾟ)+(ﾟДﾟ)[ﾟoﾟ]);
</code></pre>
<p>jjencode 的结果：</p>
<pre><code data-language="js" class="lang-js">$=~[];$={<span class="hljs-attr">___</span>:++$,<span class="hljs-attr">$$$$</span>:(![]+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">__$</span>:++$,<span class="hljs-attr">$_$_</span>:(![]+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">_$_</span>:++$,<span class="hljs-attr">$_$$</span>:({}+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">$$_$</span>:($[$]+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">_$$</span>:++$,<span class="hljs-attr">$$$_</span>:(!<span class="hljs-string">""</span>+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">$__</span>:++$,<span class="hljs-attr">$_$</span>:++$,<span class="hljs-attr">$$__</span>:({}+<span class="hljs-string">""</span>)[$],<span class="hljs-attr">$$_</span>:++$,<span class="hljs-attr">$$$</span>:++$,<span class="hljs-attr">$___</span>:++$,<span class="hljs-attr">$__$</span>:++$};$.$_=($.$_=$+<span class="hljs-string">""</span>)[$.$_$]+($._$=$.$_[$.__$])+($.$$=($.$+<span class="hljs-string">""</span>)[$.__$])+((!$)+<span class="hljs-string">""</span>)[$._$$]+($.__=$.$_[$.$$_])+($.$=(!<span class="hljs-string">""</span>+<span class="hljs-string">""</span>)[$.__$])+($._=(!<span class="hljs-string">""</span>+<span class="hljs-string">""</span>)[$._$_])+$.$_[$.$_$]+$.__+$._$+$.$;$.$$=$.$+(!<span class="hljs-string">""</span>+<span class="hljs-string">""</span>)[$._$$]+$.__+$._+$.$+$.$$;$.$=($.___)[$.$_][$.$_];$.$($.$($.$$+<span class="hljs-string">"\""</span>+<span class="hljs-string">"\\"</span>+$.__$+$.$$_+$.$$_+$.$_$_+<span class="hljs-string">"\\"</span>+$.__$+$.$$_+$._$_+<span class="hljs-string">"\\"</span>+$.$__+$.___+$.$_$_+<span class="hljs-string">"\\"</span>+$.$__+$.___+<span class="hljs-string">"=\\"</span>+$.$__+$.___+$.__$+<span class="hljs-string">"\""</span>)())();
</code></pre>
<p>这些混淆方式比较另类，但只需要输入到控制台即可执行，其没有真正达到强力混淆的效果。</p>
<p>以上便是对 JavaScript 混淆方式的介绍和总结。总的来说，经过混淆的 JavaScript 代码其可读性大大降低，同时防护效果也大大增强。</p>
<h4>JavaScript 加密</h4>
<p>不同于 JavaScript 混淆技术，JavaScript 加密技术可以说是对 JavaScript 混淆技术防护的进一步升级，其基本思路是将一些核心逻辑使用诸如 C/C++ 语言来编写，并通过 JavaScript 调用执行，从而起到二进制级别的防护作用。</p>
<p>其加密的方式现在有 Emscripten 和 WebAssembly 等，其中后者越来越成为主流。<br>
下面我们分别来介绍下。</p>
<h5>Emscripten</h5>
<p>现在，许多 3D 游戏都是用 C/C++ 语言写的，如果能将 C / C++ 语言编译成 JavaScript 代码，它们不就能在浏览器里运行了吗？众所周知，JavaScript 的基本语法与 C 语言高度相似。于是，有人开始研究怎么才能实现这个目标，为此专门做了一个编译器项目 Emscripten。这个编译器可以将 C / C++ 代码编译成 JavaScript 代码，但不是普通的 JavaScript，而是一种叫作 asm.js 的 JavaScript 变体。</p>
<p>因此说，某些 JavaScript 的核心功能可以使用 C/C++ 语言实现，然后通过 Emscripten 编译成 asm.js，再由 JavaScript 调用执行，这可以算是一种前端加密技术。</p>
<h5>WebAssembly</h5>
<p>如果你对 JavaScript 比较了解，可能知道还有一种叫作 WebAssembly 的技术，也能将 C/C++ 转成 JavaScript 引擎可以运行的代码。那么它与 asm.js 有何区别呢？</p>
<p>其实两者的功能基本一致，就是转出来的代码不一样：asm.js 是文本，WebAssembly 是二进制字节码，因此运行速度更快、体积更小。从长远来看，WebAssembly 的前景更光明。</p>
<p>WebAssembly 是经过编译器编译之后的字节码，可以从 C/C++ 编译而来，得到的字节码具有和 JavaScript 相同的功能，但它体积更小，而且在语法上完全脱离 JavaScript，同时具有沙盒化的执行环境。</p>
<p>利用 WebAssembly 技术，我们可以将一些核心的功能利用 C/C++ 语言实现，形成浏览器字节码的形式。然后在 JavaScript 中通过类似如下的方式调用：</p>
<pre><code data-language="js" class="lang-js">WebAssembly.compile(<span class="hljs-keyword">new</span> <span class="hljs-built_in">Uint8Array</span>(<span class="hljs-string">`
  00 61 73 6d  01 00 00 00  01 0c 02 60  02 7f 7f 01
  7f 60 01 7f  01 7f 03 03  02 00 01 07  10 02 03 61
  64 64 00 00  06 73 71 75  61 72 65 00  01 0a 13 02
  08 00 20 00  20 01 6a 0f  0b 08 00 20  00 20 00 6c
  0f 0b`</span>.trim().split(<span class="hljs-regexp">/[\s\r\n]+/g</span>).map(<span class="hljs-function"><span class="hljs-params">str</span> =&gt;</span> <span class="hljs-built_in">parseInt</span>(str, <span class="hljs-number">16</span>))
)).then(<span class="hljs-function"><span class="hljs-params">module</span> =&gt;</span> {
  <span class="hljs-keyword">const</span> instance = <span class="hljs-keyword">new</span> WebAssembly.Instance(<span class="hljs-built_in">module</span>)
  <span class="hljs-keyword">const</span> { add, square } = instance.exports
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'2 + 4 ='</span>, add(<span class="hljs-number">2</span>, <span class="hljs-number">4</span>))
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'3^2 ='</span>, square(<span class="hljs-number">3</span>))
  <span class="hljs-built_in">console</span>.log(<span class="hljs-string">'(2 + 5)^2 ='</span>, square(add(<span class="hljs-number">2</span> + <span class="hljs-number">5</span>)))
})
</code></pre>
<p>这种加密方式更加安全，因为作为二进制编码它能起到的防护效果无疑是更好的。如果想要逆向或破解那得需要逆向 WebAssembly，难度也是很大的。</p>
<h3>总结</h3>
<p>以上，我们就介绍了接口加密技术和 JavaScript 的压缩、混淆和加密技术，知己知彼方能百战不殆，了解了原理，我们才能更好地去实现 JavaScript 的逆向。<br>
本节代码：<a href="https://github.com/Python3WebSpider/JavaScriptObfuscate">https://github.com/Python3WebSpider/JavaScriptObfuscate</a></p>
<h3>参考文献</h3>
<ul>
<li><a href="https://www.ruanyifeng.com/blog/2017/09/asmjs_emscripten.html">https://www.ruanyifeng.com/blog/2017/09/asmjs_emscripten.html</a></li>
<li><a href="https://juejin.im/post/5cfcb9d25188257e853fa71c#heading-23">https://juejin.im/post/5cfcb9d25188257e853fa71c#heading-23</a></li>
<li><a href="https://www.jianshu.com/p/326594cbd4fa">https://www.jianshu.com/p/326594cbd4fa</a></li>
<li><a href="https://github.com/javascript-obfuscator/javascript-obfuscator">https://github.com/javascript-obfuscator/javascript-obfuscator</a></li>
<li><a href="https://obfuscator.io/">https://obfuscator.io/</a></li>
<li><a href="https://www.sojson.com/jjencode.html">https://www.sojson.com/jjencode.html</a></li>
<li><a href="http://dean.edwards.name/packer/">http://dean.edwards.name/packer/</a></li>
</ul>

---

### 精选评论

##### Ther：
> 开拓眼界了👍

