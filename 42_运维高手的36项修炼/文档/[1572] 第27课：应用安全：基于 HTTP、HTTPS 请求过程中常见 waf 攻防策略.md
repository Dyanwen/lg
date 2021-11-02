<p>上个课时讲解的 iptables 安全主要是4 层协议规则。除了 4 层安全外，7 层的安全攻击也是我们必须掌握的。那么在本课时我们基于 HTTP、HTTPS 请求过程中常见的安全防护问题来讲解 7 层应用安全。</p>
<h3>常见的 7 层安全类攻击疑难点</h3>
<p>对于 7 层安全攻击行为，作为运维工程师可能会遇到哪些痛点？</p>
<ol>
<li>无法了解代码的逻辑，这种攻击更多贴合代码或者业务逻辑漏洞，由于我们很少能了解代码，所以很难定位到代码层次的具体问题。</li>
<li>黑客的攻击行为多样，7 层的攻击行为会非常多样，所以就运维来说我们可能需要提前储备多种防护技能。</li>
</ol>
<h3>常见 HTTP、HTTPS 攻击行为</h3>
<p>本课时我们主要讲解 HTTP、HTTPS 协议的攻击行为，常见种类如下：</p>
<ol>
<li>暴力破解，这里主要对弱口令漏洞来进行攻击；</li>
<li>HTTP 内容劫持；</li>
<li>CSRF 跨站请求伪造；</li>
<li>SQL 注入；</li>
<li>XSS 跨站脚本攻击，等等。</li>
</ol>
<p>这都属于常见的 HTTP、HTTPS 请求类的攻击行为，我们将重点讲解其中的一些类型，并讲解一个运维工程师该如何去进行防范？</p>
<h3>暴力破解及其防护策略</h3>
<p>首先我们讲的是暴力破解攻击类型。暴力破解是指攻击者通过系统组合的所有可能性（网站登录时的账户、密码）尝试破解用户的账户名、密码等敏感信息。攻击者会经常使用自动化脚本或工具攻击，拿出正确的用户名和密码。它的特点有两个，一是会持续很长时间，因为要不断尝试多种用户名和密码组合；二是访问的次数会很多，因为访问的时间越长，那么它顺利破解可能性就会越大。 如果我们的用户名和密码口令设置得越弱，也就越容易被破解。</p>
<p>常见的暴力破解工具有以下几种：hydra、htpwdScan、Playload，你可以在网上查询下，其实都是通过海量的用户密码不断尝试后台访问。</p>
<p>对于这种攻击，我们应该从如下两个维度去防范：</p>
<p>运维工程师维度，需要对用户的后台，也就是我们常见的既有用户名密码的管理后台，或者是一些后台的登录接口添加一些安全策略，如：基于访问控制添加一个白名单访问，只允许某一个 IP 地址进行访问，这就大大减少了直接暴露后台接口，对所有公网都可以降低访问的隐患。</p>
<p>或者要限制用户的访问频率，不能让用户频繁地尝试登录后台，所以可以对这种攻击的过程进行频次的限制，这都是我们常会做的一些防范策略。</p>
<p>对于开发人员维度而言，首先需要把管理后台的密码设置得更加复杂，大小写字母加特殊字符加数字多种组合，让你的密码长度更长，这样尝试的时间周期就会更长，而且也越难进行破解。</p>
<p>第二就是设置一些密码校验失败的策略，比如同一个 IP，在单位时间限定其访问校验次数，如果超出了次数，就把基于 IP 或一些相关信息的后续访问限制，不让它频繁进行访问了，这也是可以在后台程序里实现的。</p>
<p>第三就是不使用用户密码的校验行为，加入一些更复杂的校验方式，如电子口令卡、动态密码等，这些都是非常有助于保护网站后台访问的，不至于把整个后台沦落到黑客手中。以上就是开发人员常常可以去做的一些防护的策略。</p>
<p>接下来为你举一个例子，我们以 Nginx 的设置为例，如果 Nginx 作为你公司服务架构的代理网关，那么在 Nginx 来做一些防范策略，具体包含： Nginx 实现管理后台的白名单控制，限流配置。</p>
<pre><code data-language="java" class="lang-java">limit_req_zone $binary_remote_addr zone=limiter:<span class="hljs-number">10</span>m rate=<span class="hljs-number">20</span>r/s;
server {
&nbsp; &nbsp; ....
&nbsp; &nbsp; location /admin {
&nbsp; &nbsp; &nbsp; proxy_pass http:<span class="hljs-comment">//127.0.0.1:8080;</span>
&nbsp; &nbsp; &nbsp; limit_req zone=limiter burst=<span class="hljs-number">1</span> nodelay;
&nbsp; &nbsp; }
&nbsp; &nbsp; location / {
&nbsp; &nbsp; &nbsp; proxy_pass http:<span class="hljs-comment">//127.0.0.1:9090;</span>
&nbsp; &nbsp; }
}
</code></pre>
<p>我们来看一下是如何来做的。首先前面这一段配置是 Nginx 的限流配置，里面用到的是 Nginx 的 limit_req_zone $binary_remote_addr zone=limite，也就是限流的配置模块limit，此配置中定义了基于什么样的方式来进行限流，这里是基于客户的原 IP 地址形式来进行限流。同时定义限制速率的大小 20s/个。这样当我的客户端访问后台这个 URL+/admin 路径的网站，每秒最大只能允许 20 次单用户 IP 进行访问，如果超过会开启限频，从而实现不被频繁进行恶意访问。</p>
<p>第二个方式就是基于控制访问的白名单来进行限制。只允许符合条件的 IP 访问，其他的一概不允许，也就是加入了一个访问控制的配置模块。allow 一个网段，允许内网是 10.1.1.0/16 的网段来访问 /admin 路径，其他的客户端 IP 过来访问一概拒绝。这就是基于 Nginx 的配置来进行访问白名单的控制。</p>
<pre><code data-language="java" class="lang-java">server {
&nbsp; &nbsp; ....
&nbsp; &nbsp; location /admin {
&nbsp; &nbsp; &nbsp; allow <span class="hljs-number">10.1</span><span class="hljs-number">.1</span><span class="hljs-number">.0</span>/<span class="hljs-number">16</span>; &nbsp; &nbsp; &nbsp; deny&nbsp; all;
&nbsp; &nbsp; &nbsp; proxy_pass http:<span class="hljs-comment">//127.0.0.1:8080;</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; location / {
&nbsp; &nbsp; &nbsp; proxy_pass http:<span class="hljs-comment">//127.0.0.1:9090;</span>
&nbsp; &nbsp; }
}
</code></pre>
<p><strong>值得注意的一点是，这样的一种模式都是基于客户端的 4 层 IP，也就是基于直接访问的客户端的 IP 地址进行限制的。</strong> 那么在课时 2 我讲过 4 层的限制和 7 层限制会存在一些差异，如果我们想要做到更高层次，进行更精确的用户行为限制，就需要结合考虑 HTTP 请求里面的一些特殊头信息，比如 x-forward-for，或者是 cookie 或其他的自定义变量来设计并实现。</p>
<h3>HTTP 内容劫持及防护策略</h3>
<p>接下来讲解第 2 个常见的应用 HTTP 协议的攻击：HTTP 内容劫持。它是指用户正常发起的一个 HTTP 请求的访问，由于服务端返回的内容被第三方（通常可以是运营商）劫持了这些数据包，并且修改了 HTTP 返回的数据，这样就可能导致用户收不到预期的返回结果。所以当用户受到这样的攻击时，就会导致用户的正常访问受到影响，另外也有可能导致用户数据发生泄漏，甚至其他更严重的安全影响。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/14/76/Ciqc1F7Q6YiASMiTAAB4a-LuU8A861.png" alt="1.png"></p>
<p>那么怎么来解决呢？我们可以通过使用 HTTPS 协议来代替 HTTP，采用 HTTPS 协议会在 HTTP 协议的基础上加了一层 TLS 防护，使得传输的数据进行加密， 劫持端无法分析和篡改内容。所以 HTTPS 这种方式可以保护我们的数据不被劫持，值得特殊注意的它是无法解决域名劫持的问题，对于基于域名劫持行为我们还需要考虑其他一些访问策略。</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/14/76/Ciqc1F7Q6Z6AEwfFAAD8F-GMrW4529.png" alt="2.png"></p>
<h3>CSRF 跨站请求伪造及其防护策略</h3>
<p>第 3 个就是 CSRF 跨站请求伪造 。我们首先来讲一下什么是 CSRF 跨站请求伪造行为。这里我画了一张图：</p>
<p><img src="https://s0.lgstatic.com/i/image/M00/14/82/CgqCHl7Q6f-AQensAAE6SboBoZM422.png" alt="3.png"></p>
<p>左边是客户端，右边的上端是受信任的网站 A（www.aaa.com），下面是一个危害的网站B（www.bbb. com）。我们来讲一讲这三者之间的关系，并且结合这张图的访问过程来讲一讲 CSRF 这种伪造请求是如何攻击并且导致用户受到影响。</p>
<p>这张图中，用户首先通过浏览器访问了他所信任的站点 A（Step1），登录站点 A 时，它会返回给用户通过验证，因为它是受信任的站点，所以通常浏览器也会保留这个 cookie 信息（Step2）。如果用户下次再来登录新站点 A，就直接通过传递 cookie 的方式去访问站点里的数据和其他页面内容。</p>
<p>恰巧这个时候有黑客给用户发送了一个引导链接，也就是一些钓鱼网站链接，让用户主动打开，并且去访问站点 B（Step3），这时用户在本地已经保存好站点 A 的 cookie 信息，是在 Cookie 的生效周期之内访问有危害的网站 B（Step4），如果网站 B 此时再给用户返回的页面内容里面有一些是让用户去请求网站 A 的恶意代码就会导致用户以为是直接访问网站 B，但实际上网站 B 返回的是一些恶意的网站请求内容，并让用户去访问受信任的站点 A（Step5）。这个时候因为信任站点 A 里面已经验证了用户的 cookie 数据，并且可以在周期之内，所以它就会成功地让用户通过信任站点 A 所设置的 cookie 信息校验。</p>
<p>这时用户就可能根据网站 B 的返回恶意代码，在不知不觉的情况下成功地请求了网站 A。如果是在这样的一种攻击情况下，假设网站 A 是银行的后台网站，控制现金的流水，网站 B 实现成功利用 A 网站登录留存认证信息，访问到了 A 网站中受信任的数据，甚至可以拿用户的一些权限来操作一些危害行为。所以这就实现了一个跨站的请求，看似让用户请求了网站 B，实际上是请求了他的受信任站点 A，而导致信任站点 A 被攻击。</p>
<p>那么我们来讲一讲应该如何去防范这样的攻击行为：</p>
<p>第一点，需要验证 HTTP Referer 字段。前面的课程里面我们有讲过 HTTP Referer 是什么，它记录了用户本次访问的 URL 地址的上一次 URL 信息是，这个信息可以溯源出它的上一级来源 URL 地址。这个时候我们可以在网站 A 里面添加一个 Referer 的头的控制。如果是拿 Nginx 进行配置，它是这样来进行配置：</p>
<pre><code data-language="java" class="lang-java">valid_referers none blocked server_names www.aaa.<span class="hljs-function">com
<span class="hljs-title">if</span> <span class="hljs-params">($invalid_referer)</span> </span>{
<span class="hljs-keyword">return</span> <span class="hljs-number">403</span>;
}
</code></pre>
<p>这里会加一个 valid_referer，就是可信的 Referer 头，它包含了正常的网站域名（www.aaa.com）等等。不满足 Referer 头信息，或是域名非指定则返回 403，这样就通过用户请求的 Referer 头的信息，有效地控制这种跨站请求行为。</p>
<p>除此之外的常见防护策略，第一个就是在请求地址中添加 token 并验证，这当然是增加验证的安全机制。还可以通过用户端的浏览器来进行安全加固，如 Chrome 浏览器，那么浏览器就可以启用 SameSite cookie 设置，避免发生跨站安全攻击的产生。</p>
<h3>SQL 注入及其防护策略</h3>
<p>最后要讲解的就是 HTTP 里常见的另外一种攻击方式：SQL 注入 ，SQL 注入你应该并不陌生，它是通过将恶意 SQL 查询，或者是添加语句插入到网站应用输入的参数之中，一旦输入的参数匹配成功，就会实现后台执行这样的 SQL，并返回一些我们不希望正常返回的数据内容。举个例子，我们来看一下 SQL 具体是如何来执行的：</p>
<pre><code data-language="java" class="lang-java">$query=SELECT first_name, last_name FROM users WHERE user_id = ’$id';
</code></pre>
<p>这里我拿了一段页面代码，假设这个页面代码的内容里发起一个查询，要查询 user 这张表格，然后基于 user_id 进行查询，那么 user_id 里面它是要拿到一个变量：$id，$id 通常是页面的输入框里面用户所填入的.<br>
正常情况下，id 在页面端需要传输一些用户 id 的数值类的数据，这时如果黑客加以篡改，把 id 信息传入特殊字符：</p>
<pre><code data-language="java" class="lang-java">1' union select database(),user()#
</code></pre>
<p>实际上它的执行效果就会完全不一样了，这个执行语句就会变成 这样子的：</p>
<pre><code data-language="java" class="lang-java">SELECT first_name, last_name FROM users WHERE user_id = <span class="hljs-string">'1'</span> <span class="hljs-function">union select <span class="hljs-title">database</span><span class="hljs-params">()</span>,<span class="hljs-title">user</span><span class="hljs-params">()</span>#`
</span></code></pre>
<p>如果在 SQL 里面来执行这样的一段语句时，# 后面的内容会被注释掉，我们可以理解为这一段内容会被注释掉，不去做执行。那么屏蔽了这段内容以后，实际上执行的什么呢？实际会改为执行前面的一段内容 SELECT first_name, last_name FROM users WHERE user_id = '1' union select database(),user()，会把我们关联查询 select database user 表里面的信息，这些敏感信息内容整体取出来，执行成功以后就会把信息返回给客户端了。</p>
<p>作为网站的管理人员，或者是安全工程师，这是我们所不希望看到的攻击行为，那么该怎么办呢？</p>
<p>通常来说，这种攻击首先需要从研发这里重点考虑，加强代码安全逻辑，对参数严格检查。或者我们可以尝试使用更加安全的一些开发框架，如 ORM 这种开发框架，接口被完全封装在&nbsp;ORM&nbsp;机制内部，不对外暴露，通过将它转化成 SQL 语句的方式去进行查询，这样能得到有效解决。</p>
<p>除了研发层需要考虑以外，运维层通常也可以考虑一些安全类的产品，比如我们可以购买一些应用类的安全防火墙服务（包含：硬件、软件类型的），这些都可以帮助我们来解决这样的问题，或者可以直接建一套基于 SQL 的敏感词过滤的入口网关系统。</p>
<p>如：Nginx 结合 LUA 这个常见语言自行开发，把 SQL 在 HTTP 协议里面发起的请求类的关键词进行过滤，比如 select、from、 Information 、Schema，tables 等关键词。这些关键词恰恰是正常网站请求里很少会携带的字符参数，我们可以基于这样的一些字符来整体做屏蔽。如果是在请求过程中发现这样的一些关键词请求，直接就在入口网关层就进行拒绝返回一个 403，这样就可以通过自建的一套敏感词入口网关的防火墙服务来防止你的后台网站被 SQL 攻击。</p>

---

### 精选评论


