<h4>背景</h4>
<p>先来看几个常见的概念：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/7B/Ciqah16f29OAE86BAAEmniG3sQE414.png" alt="测试1.png"></p>
<p>上面写了一个 flask demo，这段代码定义了一个 session_handle 方法，我们给它一个 url，即“/session”。首先读取请求，比如我们请求会发一个参数，先通过一个 get 请求发过来，服务器收到之后，然后把几个 session 写进去，之后创建一个服务器的响应，把 session 的内容全都打印出来；接着我们再写一段代码，去给服务器设置一个 cookie，为了表示这是个 cookie，在 cookie 前面加了一个标记 “cookie_”，加上我传的一个参数的名字，这样就可以通过一个 get 请求让服务器既设置 session 又设置 cookie。</p>
<p>我们来看一下对于这样的一个代码，它的响应是什么？完整的代码在这个地方，也就是在服务器里面：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/06/4C/CgoCgV6f2-eADKSNAAI7pBcLbbY475.png" alt="测试2.png"></p>
<p>有这样的代码，我们只要请求 session 就可以了，基于这个情况，继续进行对比。</p>
<p>首先，我们发起 session 请求，来看看 session 请求是怎么做的，先发起一个命令，在脚本里面以刚才的请求为例，把 get 变成 session，即 session 那段代码的意思是说如果传入一个 a 等于 1、b 等于 2，当服务器收到请求之后，那么它会在 session 里面加入 a 等于 1、b 等于 2，cookie 里面加入 cookie a=1、cookie b=2，我们通过一个内容改出来两个 session。所以，根据 route 可设置对应的 session 和 cookie，为了区分这两个文件，我们在所有的 cookie 后面加了 _ 作为后缀，同样以 curl 命令发起一个请求把它存到 session 里面，然后把它复制到项目里，此时我们再拿到这个文件，去使用 get 与 session。</p>
<h4>cookie 与 session 的区别</h4>
<p>我们看下这两个文件的区别：</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/7B/Ciqah16f2_6Aa4YEAAbbYYDnx6g884.png" alt="测试3.png"></p>
<p>可以看到，首先都是 get，但 url 不一样，第二个 url 是带有 session 处理功能的；再就是返回的内容不一样，内容长度也不一样，这都是正常的，什么时候出现变化呢？从 set-cookie 开始，可以看到当传入 a=1、b=2 时，服务器设置了 cookie a=1、cookie b =2，设置完后，服务器在响应头里面会增加 set-cookie 的字段；还有一个 Vary Cookie，这是另外一个字段，我们可以先忽略它；再往下还有一个 set-cookie 叫 session，这是一个复杂的字符串标记，从这里可以看到，当设置了 session 存储之后，如果设置了 cookie，那么会通过浏览器原样将 set-cookie 值发给你的浏览器，为什么会出现这样的机制呢？</p>
<p>是因为浏览器遵从一个约定，一个功能完备的浏览器它支持 cookie，就是说，服务器如果写一个 set-cookie，浏览器就会被 cookie 存下来，那么只需要告诉我 cookie 的格式和属性就可以了；然后再看 session，我们刚才设置了一堆数据，session 也设置了，session 传过来之后会发现，它没有 cookie 的前缀，因为刚才加的是 cookie 特有的，所以它也是个 cookie，只不过是服务端自动生成的一个指令，从而指定的 cookie，它不是我们手工设置的，是写死的一个东西，即 session=“……”后面是它的执行内容。</p>
<p>你可以看到它们的区别，cookie 的核心是它会通过一个独立 cookie 字段传下来，而无论我们设置多少 session，它只会传下来一个 cookie，用一个类似于加密串的东西，来代表存储了它的数据。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/7B/Ciqah16f3A6AKYweAAY8z01smgA390.png" alt="测试4.png"></p>
<p>那么数据到底存哪了？是加密串里还是服务端？你可以用命令再次执行，比如我们把数据存的非常长，那这个数据会不会发生变化？</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/06/4C/CgoCgV6f3BaAATJPAAMf80atzeA754.png" alt="测试5.png"></p>
<p>现在我写了一个 session2，然后把它复制过来，去服务器中看看它与 session 的区别，可以看到，首先由于 cookie 已经设置了，所以这个时候它会发生变化。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/7B/Ciqah16f3COAStenAAbwnGtoVp8331.png" alt="测试6.png"></p>
<p>再看这个字段，可以看到它设置的 session 大小并没有发生变化，这说明不同的用户访问的时候，session 是不一样的。但是它的数据并没有存到加密串里，是因为它的长度不一样，存的大小东西也不一样，但现在是一模一样，说明数据都存到了服务区，它只不过是把一个加密的关联数据放到这儿了，用来代表关联关系而已。</p>
<p>那么经过这两个体系，我们已经知道了，session 的数据全存到了服务端，它会通过 cookie 把它关联的加密内容都存进去，这种通常也叫<strong>基于 cookie 的 session</strong>。这种 session 是数据的一套管理机制，它可以用 cookie 或者其他形式进行通讯，所以说我们这次使用中央基金访问的时候，很多的网站开发框架就会默认使用 cookie 存储 session 的关联数据。有了这个实例，你可能已经了解 cookie 与 session 的区别，cookie 就是你写什么它就存到你的浏览器里面；session 是把所有的数据都存到服务端，把 session 关联的一个字符串，以 cookie 的形式传达到浏览器。</p>
<h4>cookie 机制是什么</h4>
<p>由于无痕模式的浏览器没有历史记录，所以它是没有 cookie 的。我们换一种方式，把请求换成一次 session，再发一次请求。浏览器跟 curl 命令不太一样，浏览器支持 cookie，可以看到，服务器会返回 cookie_a、cookie_b，以及 session 字段。当拿到消息之后，浏览器里其实就多了两个 cookie、一个 session（一共有 3 个 cookie）。当我们下一次再去访问服务器的时候，哪怕不输入这个网址了，只输入一个首页，甚至首页报错了，在 Headers 信息里面，只要是发送给网站的，浏览器都会自动追加一个 cookie 字段，里面包含了所有的 cookie。我们回到 request，点开这个链接，会发现 cookie 全被带上了，这样其实也很浪费带宽。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/06/4D/CgoCgV6f3FCAJOhrAALJLghJNSc769.png" alt="测试7.png"></p>
<p>所以根据用户的数据，我们通常都会存到 session 里面，只把一个标记放到客户端，而 session 唯一的 ID 其实是存在 cookie 内的，只要 cookie 带着，那么在服务端根据 cookie 的值，再去取 session 里面存在的数据就可以了。</p>
<p>这是两套机制，session 的 ID 只不过是以 cookie 的形态存到了客户端，cookie 里面的数据可以发送给服务端，服务端也可以正常接受。</p>
<p>所以大多数服务器更多的是存 session 的 ID，或者是存储一些本地离线也能访问的资料，通常不会放太多敏感信息。因为数据库给的数据也是可以篡改的，只是 session ID 无法更改，一旦改了就会失效，是因为它有一个加密机制。</p>
<p>我们了解了 cookie 的作用，就知道 session 是依托在 cookie 机制上的，但并不只是依托于 cookie，就算没有了cookie 机制，它仍然可以有其他办法来运行。</p>
<p>接下来我们做一个总结，首先浏览器接受服务端的 Set-Cookie 指令，会把 cookie 放到浏览器里面，每一个网站所保存的 cookie 值，基本上只作用于自己的网站，当然也有个例。session 会把数据都会存储到服务端，只是把关联数据的加密串或者关联的 ID，放到 cookie 中来进行了解。</p>
<h4>token 是什么</h4>
<p>token 的应用场景比如有开放认证协议、企业微信、GitHub、GitLab 等相关认证。下面以企业微信为例，打开<a href="https://work.weixin.qq.com/api/doc/90000/90135/91039">官网</a>，找到服务端的 API，它里面有个内容，即获取 token。接下来展示如何获取 token？</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/06/4D/CgoCgV6f3HWAFMFwAAQTS6oVITo864.png" alt="测试8.png"></p>
<p>获取 token 是调用企业微信 API 接口的第 1 步，调查接口都需要使用 token，这相当于创立了一个登录的凭证。从这个意义上看，token 有点类似于我们古代出城用的令牌，要想获取 token，需要你拿到企业 ID 的加密内容，也就是有个认证，然后可以在后台进行配置。拿着 corpid 和 corpsecret 这两个去申请，发起请求之后，服务器就会生成一个 token，不过它会过期，比如两小时之后，此时我会加一个标记，过期后再次申请就可以了，这样就避免了 cookie 长期被使用。</p>
<p>所以当需要过期机制的时候，通常先要申请 token，申请之后，接下来你所有的网络请求，比如创建成员、企业微信的接口、读取成员等，大量的接口创建部门都用到了 token。</p>
<h4>token 的应用场景和机制</h4>
<p>回到服务端 API 的上一页，可以看到 token 的应用场景，即利用你自身的信息获取 token，获取之后，你可以在后续的相关请求中，使用它来获得系统的一些认证，从而进行各种请求。这个 token 是放在 query 里的，可以看到它 get 的是 query 参数的形态提供，这是需要过期机制的 token，所以 token 需要有一个申请过程。</p>
<p>还有一些是不太需要过期的，以 GitHub 为例，当我们访问该网站的时候，在 GitHub 后台有一个 Settings，它里面有一个开发者设置，该设置里面有一个 Personal access token，它也会有一个 token 机制，那么 token 是用来做什么的？当你要以 API 的形式进行访问时，需要有一个认证，也就是需要知道你是谁，在生成认证的时候，会发现它这儿需要你填一个东西，比如我们这次填的是 hogwards2020。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/7C/Ciqah16f3IWAJW4RAAL-uCQbX4g904.png" alt="测试9.png"></p>
<p>再往下会有一个权限和认证机制，比如 token 是否能访问这几个接口，写完之后保存并点击一下，否则会丢失，这样就可以生成一个新的 token 了。由于 GitHub 足够保密，所以不需要有过期机制，这个 token 可以长期使用。类似这种就不需要再去动态申请了，只需要在客户端的后台有一个地方去生成 token 就可以了。</p>
<p>以上就是 GitHub 的 token 机制，有了这个机制，你就可以通过它的各种请求，把 token 放到这个地方，然后请求 GitHub 各个接口。以上就是 token 的两个案例。</p>
<h4>session 与 token 的区别</h4>
<p>有了这两个场景，我们再来看一下 session 与 token 的区别。首先回忆一下，token 是用户发起请求时附带的请求字段，用于验证用户的身份和权，也就是 token 代表了你是谁？你有什么样的权限？这就是个令牌。</p>
<p>而 session 的数据存到了服务端，它只是存了一个 session 的 ID 到浏览器里，浏览器通过 cookie，但 session 不一定要通过 cookie，也可以通过其他的内容，比如 querry 和 Headers 也可以进行存储，只要能够标记用户，代表用户的某一次访问就可以了。</p>
<p>所以主要区别在于，token 主要用于验证身份和权限，而 session 用来管理与用户相关的一些数据。一个是认证权限或身份权限，一个是管理用户相关的数据，比如你是男还是女，你的名字叫什么？最近一些推荐的数据是什么？</p>
<p>sessionid，在 Android 跨端应用的时候可能会有些变化。举个例子， Android 的原生系统是不支持 cookie 的，很多 App 写的过程中没有 cookie 这个机制，因为 App 不是浏览器，不支持服务端的 cookie 机制，除非你里面加了一个 webview，对于原生的这种不支持 cookie 的机制，会有这些解决方案。</p>
<p>首先要识别一个人的身份，我们可以通过 token 找出这个用户是谁，怎么去代表用户打开 App 之后的访问呢？光一个 token，有的时候它可以标记出来一个身份，如果每次更新 token，那么它其实已经接近于 session 的作用了。</p>
<p>通常一个 App 会这样做，在登录之后，给你一个 token，代表你是谁，打开 App 之后需要用一个东西代表你的一系列操作，并且这些操作要保持连贯。所以我会把 sessionid 再存到 HTTP 请求里。因为移动原生不支持 cookie，它没有 cookie 机制，这个时候 sessionid 怎么进行保存呢？</p>
<p>通常在我的移动端发送请求时，某一些客户端的 SDK 会发一个原生的 HTTP 请求，在请求的过程中，它会从服务端接受对应的一些 sessionid，并记录下来，以后再次发起请求的时候，会把这些 sessionid 带到 HTTP 请求的 Header 和 query 字段里。可以试着去抓你公司的 App 数据包，会发现它的请求里面其实就带了对应的 token 或者 session ID，有的时候两个都有，有一些不提供 token，只通过 sessionid 也可以知道这个用户是谁。具体需要根据公司的做法。</p>
<p>通常 token 是跨端的，既可以在移动端，也可以用浏览器或命令行，还可以在多个场景下验证用户身份与权限。</p>
<p>而 session 记录的是用户相关的数据，通常跟用户的访问会话有关。所以 sessionid 会在一些具体的业务场景里面，比如登录之后做的一系列操作、浏览器的操作、 Android 端登录之后的操作，我都会再存一个 sessionid 到 HTTP 请求里去。能支持公开就公开，不支持公开就存到 Header 或 query，这些办法都可以解决。区别点就在于：token 是个字符串，验证用户身份权限；而 session 代表的是与用户相关的一些数据，更多的是为了获取服务端的一些数据。</p>
<p>到这里，cookie 与 session、session 与 token 这几个概念分别以具体的例子来分析的，不知道你是否还有相关的疑问，如果有，欢迎加入社群一起讨论。</p>

---

### 精选评论


