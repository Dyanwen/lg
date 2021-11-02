<p data-nodeid="20909" class="">在上一课时，我们详细了解了微服务下的测试分层实践，也讲解了微服务给测试带来的挑战。你会发现，其中最重要的一条挑战，便是微服务独立开发、独立部署这一特性。由于各个微服务都是独立开发和部署，增大了微服务联调测试时的难度。</p>
<p data-nodeid="20910">在实践中，大部分微服务被拆分到不同的小型开发和测试团队。而各个团队由于各自的KPI导向不同，势必会产生对同一个Task， 两个团队设定有不同的优先级。这样就导致了<strong data-nodeid="21036">开发节奏不一致， 联调测试变得更加困难了</strong>。</p>
<p data-nodeid="20911">那么，对于相互有依赖的微服务，当我方已经接近完成，而对方尚未开始或仍在进行的情况下，我方该如何进行测试就成了一个不得不解决的问题。<strong data-nodeid="21041">这也是本讲我们要解决的问题：如何搭建 Mock Server 破除环境依赖。</strong></p>
<p data-nodeid="20912">下图是本讲的知识脑图，可供你学习参考。</p>
<p data-nodeid="20913"><img src="https://s0.lgstatic.com/i/image/M00/71/5D/Ciqc1F--FzGAXZ6WAAMdoZK9qV4023.png" alt="图片1.png" data-nodeid="21045"></p>
<h3 data-nodeid="20914">什么是 Mock？</h3>
<p data-nodeid="20915">Mock 是模拟的意思。在测试中，通常表述为：对测试过程中<strong data-nodeid="21060">不容易构造</strong>或者<strong data-nodeid="21061">不容易获取</strong>的对象，用一个<strong data-nodeid="21062">虚拟</strong>的对象来进行模拟的一个过程。</p>
<p data-nodeid="20916">那么哪些对象不容易构造？哪些对象不容易获取呢？</p>
<ul data-nodeid="20917">
<li data-nodeid="20918">
<p data-nodeid="20919">拿微服务举例，在一个调用链条上，微服务 A 依赖 B 服务才能提供服务，而微服务 B 依赖 C 服务， 微服务 C 依赖 D 服务.....在这样的情况下，把每个依赖的服务都构造完毕再开始测试，就变得不太现实。这种情况我们称之为<strong data-nodeid="21069">不容易构造</strong>。</p>
</li>
<li data-nodeid="20920">
<p data-nodeid="20921">又比如，假设我们的服务依赖银行的接口提供资金的查询。在测试中， 银行不可能无条件或者随意提供接口给我们调用。那么，在我们开发完毕但是要依赖对方才能开始测试时， 我们称这种情况为<strong data-nodeid="21075">不容易获取</strong>。</p>
</li>
</ul>
<p data-nodeid="20922">无论是哪种情况，使用 Mock 的一大前提条件是：我们仅关注测试对象自身内部逻辑是否正确，而<strong data-nodeid="21081">不关心其依赖对象逻辑的正确性</strong>。</p>
<h3 data-nodeid="20923">Mock Server 是什么</h3>
<p data-nodeid="20924">了解了什么是 Mock，理解 Mock Server 就比较容易了。简而言之，能够提供 Mock 功能的服务就叫作 Mock Server。Mock Server 通过模仿真实的服务器，提供对来自客户端请求的真实响应。</p>
<p data-nodeid="20925"><strong data-nodeid="21087">那么 Mock Serve 如何模仿真实的服务器呢？</strong></p>
<p data-nodeid="20926">一般情况下，搭建 Mock Server 前，需要了解将要 Mock 的服务，都能提供哪些功能？对外提供功能时，又以哪种格式提供服务？例如，以接口方式提供服务，接口的种类、接口的定义，以及接口输出的参数等信息。</p>
<p data-nodeid="20927">了解了这些，Mock Server 就可以根据请求的不同，直接静态地返回符合业务规范的接口，也可以在 Mock Server 内部经过简单计算，动态返回符合业务规范的接口。</p>
<p data-nodeid="20928">在实际工作中，Mock Server 通常以 Mock API Server 的形式存在，也就是我们一般以接口的形式对外提供服务，Mock Server 搭建在本地或者远程均可以对外提供服务。</p>
<h3 data-nodeid="20929">Mock Server 的常用场景</h3>
<p data-nodeid="20930">最常见的 Mock Server 的使用场景如下：</p>
<ul data-nodeid="20931">
<li data-nodeid="20932">
<p data-nodeid="20933">前后端联调使用，通过事先约定接口规范，使前端可以不依赖后端服务<strong data-nodeid="21098">独立开展工作</strong>，这也是开发最常用的功能。</p>
</li>
<li data-nodeid="20934">
<p data-nodeid="20935">使用 Mock Server<strong data-nodeid="21104">屏蔽无关的真实服务</strong>，从而专注于要测试的服务本身。仅仅测试需要测试的服务，其他不在我负责范围的服务使用 Mock。</p>
</li>
<li data-nodeid="20936">
<p data-nodeid="20937">供测试工程师使用，在测试环境<strong data-nodeid="21110">避免调用第三方收费服务</strong>。比如，企查查等服务是收费的，在测试环境就可以不调用，以节省费用。</p>
</li>
<li data-nodeid="20938">
<p data-nodeid="20939"><strong data-nodeid="21115">破除第三方依赖</strong>。比如，本公司业务流程的某一个步骤需要获取第三方服务的正确返回才能继续进行，那么在测试中就可以用 Mock Server，直接模拟外部 API 的响应来断言系统的正确行为。</p>
</li>
</ul>
<p data-nodeid="20940">以上四条基本可以概括 Mock Server 绝大多数的使用情况。</p>
<p data-nodeid="20941">可以看到，前两条主要是开发之间在使用，那么这个 Mock Server 通常是开发之间协调提供；或者是前端开发根据 API 接口规范，直接写 Hard Code 的响应供自己调用；或者是后端直接提供一个返回值给前端调用，基于成本和时间考虑，这个返回值通常也是 Hard Code 的，这一块不在我们今天的讨论范畴。</p>
<p data-nodeid="20942"><strong data-nodeid="21121">而后两条就都是跟测试密切相关了，也是我们今天需要关注的。</strong></p>
<h3 data-nodeid="20943">Mock Server 搭建</h3>
<p data-nodeid="20944">Mock Server 的搭建有两种方式，分别是借助第三方工具直接提供 Mock Server，以及自主编码实现 Mock Server。下面我来分别介绍下这两种方式。</p>
<h4 data-nodeid="20945">1.借助第三方工具直接提供 Mock Server</h4>
<p data-nodeid="20946">可以直接提供 Mock Server 功能的第三方工具很多，这里我选择使用<strong data-nodeid="21130">Postman</strong>的 Mock 功能。 Postman 提供了三种方式创建 Mock Server，我们直接选择第一种，并以Postman官方给的例子来看下如何不写代码创建 Mock Server。</p>
<p data-nodeid="20947">（1）打开 Postman， 点击"+New"&nbsp;button。</p>
<p data-nodeid="20948"><img src="https://s0.lgstatic.com/i/image/M00/71/68/CgqCHl--Fx6ARgtQAAJ9oo1jikM473.png" alt="图片2.png" data-nodeid="21138"></p>
<p data-nodeid="20949">（2）在弹出来的"Create New"选项中点击&nbsp;Mock Server&nbsp;。</p>
<p data-nodeid="20950"><img src="https://s0.lgstatic.com/i/image/M00/71/5D/Ciqc1F--FzyAMKe3AAJtlj-EnM4677.png" alt="图片3.png" data-nodeid="21146"></p>
<p data-nodeid="20951">（3）Postman支持"Create a new API"或者"Use collection from this workspace"两种方式来创建 Mock Server。</p>
<p data-nodeid="20952">简单起见，我们选择“Create a new API”。在下图中我们选择请求方法，可以是 GET、POST、UPDATE，也可以是 DELETE，也就是我们常说的增删查改。然后输入请求路径，需要返回的 HTTP 响应码，以及响应的 Body，可以模拟多个 API 接口。全部设置好后点击下一步。</p>
<p data-nodeid="20953"><img src="https://s0.lgstatic.com/i/image/M00/71/69/CgqCHl--F0WATe6SAAIT0U9K5xI725.png" alt="图片4.png" data-nodeid="21159"></p>
<p data-nodeid="20954">（4）然后，你将看到下图 4 个需要配置的地方。</p>
<p data-nodeid="20955"><img src="https://s0.lgstatic.com/i/image/M00/71/5D/Ciqc1F--F02AUHoXAAI1JkHHCBM654.png" alt="图片5.png" data-nodeid="21163"></p>
<ul data-nodeid="20956">
<li data-nodeid="20957">
<p data-nodeid="20958">输入 Mock Server 的名称。</p>
</li>
<li data-nodeid="20959">
<p data-nodeid="20960">选择一个环境（可选），通常我们的测试环境有好几个，你可以配置使用不同的测试环境。</p>
</li>
<li data-nodeid="20961">
<p data-nodeid="20962">是否要将 Mock Server 设为私有。</p>
</li>
<li data-nodeid="20963">
<p data-nodeid="20964">是否将 Mock Server 的 URL 保存为环境变量。</p>
</li>
</ul>
<p data-nodeid="20965">等你都配置好后，单击下一步继续。</p>
<p data-nodeid="20966">（5）当你看到如下界面，说明配置成功。此时你的简易版 Mock Server 就生成了。记录下生成的 URL，然后在你的测试中调用相应的 URL 地址即可。</p>
<p data-nodeid="20967"><img src="https://s0.lgstatic.com/i/image/M00/71/5D/Ciqc1F--F1WAG2HEAAIVgTJq3fU557.png" alt="图片6.png" data-nodeid="21172"></p>
<p data-nodeid="20968">在本例中，我在第（3）步设置了 echo 这个接口，它是个 GET 请求，你就可以直接在浏览器输入 http://mock-server-url/echo 这样的方式来访问，需要替换这里 mock-server-url 为图中的地址。</p>
<p data-nodeid="20969">如果是 POST 请求，你也可以自定义参数，Request Body 等。</p>
<h4 data-nodeid="20970">2.自主编码实现 Mock Server（Flask）</h4>
<p data-nodeid="20971">使用第三方工具创建 Mock Server 比较简单，但是由于严重依赖于第三方工具，在实际工作中，一般用作开发完成后的第一轮手工测试。而<strong data-nodeid="21181">业务上线后，在测试框架中使用时</strong>，我们还是倾向于根据业务规则自主编码实现 Mock Server。</p>
<p data-nodeid="20972">当前，Github 上有很多成熟的 Mock Server 可供我们使用，根据编程语言的不同，最常见的有如下几个：</p>
<ul data-nodeid="20973">
<li data-nodeid="20974">
<p data-nodeid="20975"><a href="https://github.com/mock-server" data-nodeid="21185">Java - Mock Server</a></p>
</li>
<li data-nodeid="20976">
<p data-nodeid="20977"><a href="https://github.com/getsentry/responses" data-nodeid="21188">Python - responses</a></p>
</li>
<li data-nodeid="20978">
<p data-nodeid="20979"><a href="https://github.com/easy-mock/easy-mock" data-nodeid="21191">JavaScript - easy Mock</a></p>
</li>
</ul>
<p data-nodeid="20980">这些 Mock Server 的搭建非常简单，按照步骤操作即可，我就不再赘述。</p>
<p data-nodeid="20981">下面我讲下 Mock Server 的另外一个普遍搭建过程，即使用<strong data-nodeid="21198">Flask</strong>来充当 Mock Server。</p>
<blockquote data-nodeid="20982">
<p data-nodeid="20983">Flask 是一个微 Web 框架，使用 Python 语言编写。使用它可以快速完成功能丰富的中小型网站或 Web 服务的实现。</p>
</blockquote>
<p data-nodeid="20984">（1）首先你要保证系统已经安装好 Flask，并确保你的机器有 Python 运行环境。</p>
<pre class="lang-python" data-nodeid="20985"><code data-language="python">pip install flask
</code></pre>
<p data-nodeid="20986">（2）创建一个 Python 文件，比如叫 easyMock.py.，代码如下：</p>
<p data-nodeid="20987"><img src="https://s0.lgstatic.com/i/image/M00/70/B7/CgqCHl-7kfKAPZUaAAG9yxU5oGo024.png" alt="Drawing 5.png" data-nodeid="21204"></p>
<p data-nodeid="20988">这段代码实现了这一功能：访问 <a href="http://127.0.0.1:5000" data-nodeid="21208">http://127.0.0.1:5000</a>，直接返回“hello world”。</p>
<p data-nodeid="20989">直接使用 GET 方式访问<strong data-nodeid="21215">http://127.0.0.1:5000/mock</strong>，会出现 404 错误。</p>
<p data-nodeid="20990">如果使用 POST 方式，假设提交的数据中包括“name=kevin”这个键值对，则返回如下结果：</p>
<pre class="lang-java" data-nodeid="20991"><code data-language="java">{<span class="hljs-string">"status"</span>: <span class="hljs-number">200</span>, <span class="hljs-string">"message"</span>: <span class="hljs-string">"True"</span>, <span class="hljs-string">"response"</span>: {<span class="hljs-string">"orderID"</span>: <span class="hljs-number">100</span>}}
</code></pre>
<p data-nodeid="20992">如果你提交的数据中不包括“name=kevin”， 则返回如下结果：</p>
<pre class="lang-java" data-nodeid="20993"><code data-language="java">{<span class="hljs-string">"status"</span>: <span class="hljs-number">400</span>, <span class="hljs-string">"message"</span>: <span class="hljs-string">"False"</span>, <span class="hljs-string">"response"</span>: {}}
</code></pre>
<p data-nodeid="20994">如果代码在运行过程中发生了错误，则返回如下结果：</p>
<pre class="lang-java" data-nodeid="20995"><code data-language="java">{<span class="hljs-string">"status"</span>: <span class="hljs-number">500</span>, <span class="hljs-string">"message"</span>: <span class="hljs-string">"Server Error"</span>, <span class="hljs-string">"response"</span>: {}}
</code></pre>
<p data-nodeid="20996">这其实就是一个最简单的Mock Server。<br>
（3）启动这个 Flask 服务。</p>
<p data-nodeid="20997">打开命令行工具，在你的 Terimal 里运行以下命令行，以启动这个 Mock Server。</p>
<pre class="lang-python" data-nodeid="20998"><code data-language="python">python easyMock.py
</code></pre>
<p data-nodeid="20999">（4）测试 Mock Server。<br>
首先安装 curl。</p>
<blockquote data-nodeid="21000">
<p data-nodeid="21001">curl 是一个利用 URL 语法在命令行方式下工作的文件传输工具。由于它支持 HTTP 协议及其请求方法，故也可以用来发送 HTTP 请求。</p>
</blockquote>
<pre class="lang-python" data-nodeid="21002"><code data-language="python"><span class="hljs-comment"># curl的安装和配置，根据操作系统的不同，步骤也不同。</span>
<span class="hljs-comment"># 如果你使用pip， 可以直接以如下方式安装。 </span>
pip install curl
<span class="hljs-comment"># 如果你发现在你的操作系统下，上述安装方式不起作用，你可以直接在搜索引擎中搜索相关的安装方式。</span>
</code></pre>
<p data-nodeid="21003">curl 常用的语法如下：</p>
<pre class="lang-python" data-nodeid="21004"><code data-language="python"><span class="hljs-comment"># 直接发送GET请求</span>
$ curl https://www.helloqa.com
<span class="hljs-comment"># 添加HTTP请求头访问</span>
$ curl -H <span class="hljs-string">"Content-type: application/json"</span> https://www.helloqa.com
<span class="hljs-comment"># 指定HTTP请求</span>
<span class="hljs-comment"># -X 表示请求方法</span>
<span class="hljs-comment"># -d 表示发送 POST 请求的数据体</span>
$ curl -X POST  -d <span class="hljs-string">'iTesting=Good'</span> https://www.helloqa.com
</code></pre>
<p data-nodeid="21005">最后，我们通过 curl 发送 HTTP 请求，来验证下搭建的 Mock Server 是否功能正确：</p>
<pre class="lang-python" data-nodeid="21006"><code data-language="python"><span class="hljs-comment"># 通过curl直接调用，返回500</span>
curl -H <span class="hljs-string">"Content-type: application/json"</span> -X POST -d <span class="hljs-string">'{"name":"kevin"}'</span> http://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">5000</span>/mock
<span class="hljs-comment"># 返回400</span>
curl -d<span class="hljs-string">'name=kevin＆'</span>-X POST http://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">5000</span>/mock
<span class="hljs-comment"># 返回200</span>
curl -d<span class="hljs-string">'name=kevin'</span> -X POST http://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">5000</span>/mock
</code></pre>
<p data-nodeid="21007">可以看到，根据我的输入不同，Mock Server 返回了期望的结果。</p>
<p data-nodeid="21008">至此，你的 Mock Server 已经搭建完毕。之后在你的测试代码里，涉及调用第三方应用的情况，你就可以直接转而调用 Mock Server 来继续你的测试了。当然，你的 Mock Server 实现要考虑第三方应用的业务逻辑和输出结果的格式、参数以及数据等方面。</p>
<p data-nodeid="21009">不知道你有没有注意到，Mock Server 无论是上述哪种方式的创建，都需要一点点工作量，而且都有如下弊端：</p>
<ul data-nodeid="21010">
<li data-nodeid="21011">
<p data-nodeid="21012">你无法向真正的服务器发送请求，你的所有请求都发送至 Mock Server。</p>
</li>
<li data-nodeid="21013">
<p data-nodeid="21014">在真实服务器可以提供工作，或由 Mock Server 向真实服务器之间进行切换时，可能由于<strong data-nodeid="21238">人为原因</strong>导致错误。比如，有的地方你替换了真实服务器，有点地方你仍调用 Mock Server。</p>
</li>
</ul>
<p data-nodeid="21015">那么，有没有办法可以实现：我直接向真实的服务器发送请求，同时我要求真实的服务器根据我的需要，来返回 Mock 数据或者真实的服务器响应数据呢？</p>
<p data-nodeid="21016">当然有了，利用新一代前端自动化测试框架 Cypress 可以不写代码便能完成如上请求。Cypress 是新一代端到端测试神器，被誉为 Selenium/WebDriver 杀手和 Web 端自动化测试技术的未来。</p>
<blockquote data-nodeid="21017">
<p data-nodeid="21018">关于如何利用 Cypress 搭建 MockServer，实现更高效的 Mock，大家可以参考下我今年出版的新书《前端自动化测试框架 -- Cypress 从入门到精通》。</p>
</blockquote>
<h3 data-nodeid="21019">总结</h3>
<p data-nodeid="21020">下面我来总结下本章学习的内容。本章我先是从 Mock 的定义出发，讲解了 Mock Server 的含义、常用场景，以及 Mock Server 在我们开发测试中的重要性，接着又采用两种方式实现了Mock Server，分别是：</p>
<ul data-nodeid="21021">
<li data-nodeid="21022">
<p data-nodeid="21023">使用 API 接口工具 Postman。它的优点是无须编程，缺点是生成是 Server URL 地址无法更改。</p>
</li>
<li data-nodeid="21024">
<p data-nodeid="21025">使用 Python 的 Flask 框架编程。它的优点是可以把 Mock Server 集成至自己的测试框架中，缺点是需要一定的编程能力。</p>
</li>
</ul>
<p data-nodeid="21026">Mock Server 作为破除环境依赖的利器，能够大大提升我们的测试效率，希望通过这些内容，让你彻底掌握 Mock Server。如果你希望掌握更前沿的 Mock 技术，也可以去了解 Cypress 框架，让自己更上一层楼。</p>
<p data-nodeid="24332" class="">我是蔡超，我们下节课再见。</p>







<p data-nodeid="25014">更多关于测试框架、Mock Server、PostMan 使用的知识，请关注公众号 iTesting 查看。</p>
<hr data-nodeid="25697">
<p data-nodeid="25698" class="te-preview-highlight"><a href="https://wj.qq.com/s2/7506053/9b01" data-nodeid="25701">课程评价入口，挑选 5 名小伙伴赠送小礼品～</a></p>

---

### 精选评论

##### *影：
> 老师，你好，如果我要测试的服务A调用了其它系统（称为B）,要使用mock来测试服务A是不是必须在部署之前将服务A中调用服务B的地方换成用来模拟B的mock接口C才行？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的。如果你测试的是功能部分， 比如你测试A，A依赖B， 但是B没有开发完。 开发会给你一个用了Mock 的A服务。会告诉你他先Mock了。你需要在B开发完后再测试一下。
但是比如你在做单元测试，这个功能的实现里，包括有对第三方的接口调用。 那么你就可以把这个接口换成你的Mock server地址来使用了。

##### **0321：
> 老师好，如果我指写了几个接口的mock 服务，有没有办法让服务器在运行到这几个接口的时候转发到我的mock 服务器。类似hosts 转发？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 写个配置函数吧，去读一个配置文件，文件里是真实server:mock的配置文件，然后根据需要映射返回。简单点就直接在测试需要用到的地方的setup里加上就行了。通用点就是，如果单纯是接口，可以把get方法和post方法再包装一层，把这个配置函数放进去。在你的测试代码里仅调用包装后的get和post就行了。

##### **青：
> curl -d'name=kevin' -X POST http://127.0.0.1:5000/mock Current Speed100提交的参数是对的，不知道为什么还是报500错误呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Server启动了吗？ Mock Server创建好后，需要启动下。然后你再仔细检查下代码和文中的是否一致。

##### **0676：
> 老师，你好，能不能讲一下接口之间的参数依赖(比如接口A需要以接口B的某个返回值为入参)在自动化框架中是如何实现的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 写个函数再包一层。 例如 你需要调用A和B是来实现功能。你可以写个函数类似下面这样：
def test_func():
	 prar_to_b = A()
	 B(prar_to_b)

