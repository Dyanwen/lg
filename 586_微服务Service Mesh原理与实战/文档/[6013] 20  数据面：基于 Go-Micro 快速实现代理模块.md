<p data-nodeid="693" class="">今天我要和你分享的内容是：如何自研 Service Mesh 中的数据面，这里我们选择用 Go-Micro 框架实现 Sidecar 中最核心的代理模块。</p>
<p data-nodeid="694">在本讲中，我们选用了 Go 语言实现数据面。在常见的开源数据面中，Envoy 使用 C++ 实现，Linkerd 早期版本使用 Java，新版本使用 Rust 实现，而 MOSN 和 Maesh 都使用 Go 语言实现。从这些技术选型可以看出，Go 语言在数据面的实现中占有一席之地，主要是<strong data-nodeid="765">较高的性能和较少的内存占用</strong>，比较适合 Sidecar 的应用场景。</p>
<p data-nodeid="695">为了快速实现，我们使用了 Go-micro 框架，这个框架有非常好的抽象层，支持 MUCP、gRPC、HTTP 等协议的原生代理，后期也可以根据需求扩展协议层。</p>
<p data-nodeid="696">本文讲解的代码地址在：<a href="https://github.com/beck917/easymesh" data-nodeid="770">https://github.com/beck917/easymesh</a>，你可以配合代码阅读本文。</p>
<h3 data-nodeid="697">环境安装&amp;框架搭建</h3>
<p data-nodeid="698">首先进入 Go 语言官网，根据环境下载合适的版本<a href="https://golang.google.cn/dl/" data-nodeid="778">https://golang.google.cn/dl/</a>，因为 Go-Micro 的旧版本依赖问题，这里我们选择 Go 1.14 版本。</p>
<p data-nodeid="699"><img src="https://s0.lgstatic.com/i/image6/M00/07/7C/Cgp9HWAzh5aAdgqCAAGppXYgA2M543.png" alt="Drawing 0.png" data-nodeid="782"></p>
<p data-nodeid="700">代码中我们并没有使用最新的 v3 版本，因为 Go-Micro 框架的母公司转向云平台的开发模式，Go-Micro 的 v3 版本转移到了个人项目下，但这个版本还有诸多问题，缺少以前的很多特性，所以这里还是采用了 v1 版本。</p>
<h3 data-nodeid="701">代码解析</h3>
<p data-nodeid="702">首先我们看一下 main 方法，这里创建了两个代理服务器，分别是监听 8082 端口的出流量代理和监听 8083 端口的 8083 服务：</p>
<pre class="lang-java" data-nodeid="703"><code data-language="java"><span class="hljs-function">func&nbsp;<span class="hljs-title">main</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;http.HandleFunc(<span class="hljs-string">"/"</span>,&nbsp;indexHandler)
&nbsp;&nbsp;&nbsp;&nbsp;go&nbsp;http.ListenAndServe(<span class="hljs-string">":6060"</span>,&nbsp;nil)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;start&nbsp;the&nbsp;client&nbsp;proxy</span>
&nbsp;&nbsp;&nbsp;&nbsp;go&nbsp;proxy.Run(&amp;proxy.ServerOptions{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Name:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Client&nbsp;Listener&nbsp;Proxy"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Address:&nbsp;&nbsp;<span class="hljs-string">":8082"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Endpoint:&nbsp;<span class="hljs-string">"http://127.0.0.1:8083"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Protocol:&nbsp;<span class="hljs-string">"http"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;time.Sleep(<span class="hljs-number">1</span>&nbsp;*&nbsp;time.Second)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;start&nbsp;the&nbsp;server&nbsp;proxy</span>
&nbsp;&nbsp;&nbsp;&nbsp;proxy.Run(&amp;proxy.ServerOptions{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Name:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Server&nbsp;Listener&nbsp;Proxy"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Address:&nbsp;&nbsp;<span class="hljs-string">":8083"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Endpoint:&nbsp;<span class="hljs-string">"http://127.0.0.1:6060"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Protocol:&nbsp;<span class="hljs-string">"http"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;})
}
</code></pre>
<p data-nodeid="704">下面解释下 proxy.Run 方法的几个参数。</p>
<ul data-nodeid="705">
<li data-nodeid="706">
<p data-nodeid="707">Name：代理服务器唯一标识。</p>
</li>
<li data-nodeid="708">
<p data-nodeid="709">Address：代理服务器监听地址。</p>
</li>
<li data-nodeid="710">
<p data-nodeid="711">Endpoint：代理服务器流量转发的地址，可以看到这里 Client Proxy 将 Endpoint 设置为 http://127.0.0.1:8083，意思是将流量转发到 Server Proxy，而 Server Proxy 将流量转向了本地的 6060 端口，也就是服务端口，这样就形成了一个完成的 Mesh 链路。</p>
</li>
<li data-nodeid="712">
<p data-nodeid="713">Protocol：代理的协议，这里是 HTTP。</p>
</li>
</ul>
<p data-nodeid="714">下面我们进入 Run 方法看一下核心代码：</p>
<pre class="lang-java" data-nodeid="715"><code data-language="java">&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;new&nbsp;proxy</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;p&nbsp;proxy.Proxy
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">var</span>&nbsp;s&nbsp;server.Server
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;set&nbsp;endpoint</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">if</span>&nbsp;<span class="hljs-title">len</span><span class="hljs-params">(Endpoint)</span>&nbsp;&gt;&nbsp;0&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">switch</span>&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;strings.HasPrefix(Endpoint,&nbsp;<span class="hljs-string">"grpc://"</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ep&nbsp;:=&nbsp;strings.TrimPrefix(Endpoint,&nbsp;<span class="hljs-string">"grpc://"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;popts&nbsp;=&nbsp;append(popts,&nbsp;proxy.WithEndpoint(ep))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;grpc.NewProxy(popts...)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">case</span>&nbsp;strings.HasPrefix(Endpoint,&nbsp;<span class="hljs-string">"http://"</span>):
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;<span class="hljs-doctag">TODO:</span>&nbsp;strip&nbsp;prefix?</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;popts&nbsp;=&nbsp;append(popts,&nbsp;proxy.WithEndpoint(Endpoint))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;http.NewProxy(popts...)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;s&nbsp;=&nbsp;smucp.NewServer(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server.WrapHandler(ratelimiter.NewHandlerWrapper(<span class="hljs-number">10</span>)),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">default</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;<span class="hljs-doctag">TODO:</span>&nbsp;strip&nbsp;prefix?</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;popts&nbsp;=&nbsp;append(popts,&nbsp;proxy.WithEndpoint(Endpoint))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;p&nbsp;=&nbsp;mucp.NewProxy(popts...)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="716">Run()方法根据传入的参数 Endpoint 会选择对应的协议，这里我传入的是 HTTP 协议的Endpoint，会创建一个 HTTP 的 Proxy。</p>
<p data-nodeid="717">这个 HTTP 的 Proxy 跟踪你可以参照 <a href="https://github.com/asim/go-micro" data-nodeid="796">Go-Micro 的代码</a>，也很简单，其实所有的 Proxy 都会通过 ServeRequest(ctx context.Context, req server.Request, rsp server.Response) 这个方法，这个方法对 Server 处理过后的 HTTP 进行转发操作。</p>
<pre class="lang-java" data-nodeid="718"><code data-language="java">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;get&nbsp;data</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;body,&nbsp;err&nbsp;:=&nbsp;req.Read()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;err&nbsp;==&nbsp;io.EOF&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;nil
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;err&nbsp;!=&nbsp;nil&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;err
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;get&nbsp;the&nbsp;header</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hdr&nbsp;:=&nbsp;req.Header()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;get&nbsp;method</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;method&nbsp;:=&nbsp;getMethod(hdr)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;get&nbsp;endpoint</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endpoint&nbsp;:=&nbsp;getEndpoint(hdr)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;send&nbsp;to&nbsp;backend</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hreq,&nbsp;err&nbsp;:=&nbsp;http.NewRequest(method,&nbsp;endpoint,&nbsp;bytes.NewReader(body))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;err&nbsp;!=&nbsp;nil&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;errors.InternalServerError(req.Service(),&nbsp;err.Error())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;set&nbsp;the&nbsp;headers</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">for</span>&nbsp;k,&nbsp;v&nbsp;:=&nbsp;range&nbsp;hdr&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hreq.Header.Set(k,&nbsp;v)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;make&nbsp;the&nbsp;call</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;hrsp,&nbsp;err&nbsp;:=&nbsp;http.DefaultClient.Do(hreq)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;err&nbsp;!=&nbsp;nil&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;errors.InternalServerError(req.Service(),&nbsp;err.Error())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="719">这里先读取了 Server 获取到的请求 header 和 body，然后建立了 HTTP 客户端进行流量转发。你可以看到，这里的转发相对简单，因为相对于自定义协议，<strong data-nodeid="803">HTTP Golang 提供了现成的客户端</strong>，这里直接使用即可。</p>
<p data-nodeid="720">Go-Micro 的 HTTP 的代理实现相对简单，但是你要注意：这里的代码并没有进行优化和客户端的超时处理，无法直接用于生产环境。如果想用于生产环境，还需要一些改造，比如自己建立 HTTP Client，而不是直接用 http.DefaultClient。另外，hreq 对象也要携带 context，以确保可以<strong data-nodeid="809">控制超时</strong>，返回 504 Gateway Timetout 的错误。</p>
<p data-nodeid="721">创建完 Proxy 后，下面的代码基本上和 Go-Micro 创建一个新的 Server 没有什么区别了。只是这里的 Muxer 这个 router 将会直接绕过 Server 的 router 层，直接将流量转向 Proxy 层，因为 Server 的 router 其实已经没有用了，相当于被 Proxy 劫持了，毕竟这里我们不需要路由到 Server 的具体方法。</p>
<pre class="lang-java" data-nodeid="722"><code data-language="java">&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;new&nbsp;service</span>
&nbsp;&nbsp;&nbsp;&nbsp;service&nbsp;:=&nbsp;micro.NewService(srvOpts...)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;create&nbsp;a&nbsp;new&nbsp;proxy&nbsp;muxer&nbsp;which&nbsp;includes&nbsp;the&nbsp;debug&nbsp;handler</span>
&nbsp;&nbsp;&nbsp;&nbsp;muxer&nbsp;:=&nbsp;mux.New(Name,&nbsp;p)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;set&nbsp;the&nbsp;router</span>
&nbsp;&nbsp;&nbsp;&nbsp;service.Server().Init(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;server.WithRouter(muxer),
&nbsp;&nbsp;&nbsp;&nbsp;)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;Run&nbsp;internal&nbsp;service</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;err&nbsp;:=&nbsp;service.Run();&nbsp;err&nbsp;!=&nbsp;nil&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;log.Fatal(err)
&nbsp;&nbsp;&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="723">进入 mux 的代码也可以了解到，这里直接调用了proxy.ServeRequest()：</p>
<pre class="lang-java" data-nodeid="724"><code data-language="java">func&nbsp;(s&nbsp;*Server)&nbsp;ServeRequest(ctx&nbsp;context.Context,&nbsp;req&nbsp;server.Request,&nbsp;rsp&nbsp;server.Response)&nbsp;error&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span>&nbsp;req.Service()&nbsp;==&nbsp;s.Name&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;server.DefaultRouter.ServeRequest(ctx,&nbsp;req,&nbsp;rsp)
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;s.Proxy.ServeRequest(ctx,&nbsp;req,&nbsp;rsp)
}
</code></pre>
<p data-nodeid="725">我们回顾一下，刚才在创建 Proxy 的时候，同时创建了一个 Server 对象，这个 Server 同时创建了一个 Wrapper，Wrapper 其实就是 Go-Micro 的 Middleware 层。这里我们创建了一个限流的中间件，后面在讲解控制面的时候，还要创建路由和负载均衡中间件。</p>
<p data-nodeid="726">我们先通过一个简单的限流中间件学习一下这部分内容：</p>
<pre class="lang-java" data-nodeid="727"><code data-language="java">s = smucp.NewServer(
                server.WrapHandler(ratelimiter.NewHandlerWrapper(<span class="hljs-number">10</span>)),
            )
</code></pre>
<p data-nodeid="728">RateLimit 中间件使用了 Uber 的库实现，达到限流值会产生阻塞，而不是返回错误。<strong data-nodeid="819">在生产中，还是建议使用返回错误的方式实现限流器</strong>，因为产生阻塞可能会进一步恶化并发情况。具体代码如下：</p>
<pre class="lang-java" data-nodeid="729"><code data-language="java"><span class="hljs-comment">//&nbsp;NewHandlerWrapper&nbsp;creates&nbsp;a&nbsp;blocking&nbsp;server&nbsp;side&nbsp;rate&nbsp;limiter</span>
<span class="hljs-function">func&nbsp;<span class="hljs-title">NewHandlerWrapper</span><span class="hljs-params">(rate&nbsp;<span class="hljs-keyword">int</span>)</span>&nbsp;server.HandlerWrapper&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;r&nbsp;:=&nbsp;ratelimit.New(rate)
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;func(h&nbsp;server.HandlerFunc)&nbsp;server.HandlerFunc&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;func(ctx&nbsp;context.Context,&nbsp;req&nbsp;server.Request,&nbsp;rsp&nbsp;<span class="hljs-class"><span class="hljs-keyword">interface</span></span>{})&nbsp;error&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;r.Take()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;h(ctx,&nbsp;req,&nbsp;rsp)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="730">至此，一个简单的 Sidecar Proxy 的代码实现部分就讲解完了。下面我们编译并启动这个简单的 Sidecar ，来看一下实际的运行效果。</p>
<p data-nodeid="731">对代码进行编译并启动：</p>
<pre class="lang-java" data-nodeid="732"><code data-language="java">go run -mod=vendor .\main.go
</code></pre>
<p data-nodeid="733">可以看到下面的启动信息：</p>
<pre class="lang-java" data-nodeid="734"><code data-language="java"><span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">44.486504</span> I | [proxy] Proxy [http] serving endpoint: http:<span class="hljs-comment">//127.0.0.1:8083</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">44.490502</span> I | [proxy] Transport [http] Listening on [::]:<span class="hljs-number">8082</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">44.502494</span> I | [proxy] Broker [http] Connected to [::]:<span class="hljs-number">14393</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">44.695376</span> I | [proxy] Registry [mdns] Registering node: Client Listener Proxy-<span class="hljs-number">5512f</span>579-<span class="hljs-number">7</span>eed-<span class="hljs-number">4</span>ef6-<span class="hljs-number">8</span>c80-<span class="hljs-number">4342650</span>afc2d
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">45.497904</span> I | [proxy] Proxy [http] serving endpoint: http:<span class="hljs-comment">//127.0.0.1:6060</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">45.498903</span> I | [proxy] Transport [http] Listening on [::]:<span class="hljs-number">8083</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">45.585850</span> I | [proxy] Broker [http] Connected to [::]:<span class="hljs-number">14393</span>
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">45.758753</span> I | [proxy] Registry [mdns] Registering node: Server Listener Proxy-<span class="hljs-number">5512f</span>579-<span class="hljs-number">7</span>eed-<span class="hljs-number">4</span>ef6-<span class="hljs-number">8</span>c80-<span class="hljs-number">4342650</span>afc2d
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">46.606504</span> I | [proxy] Registry [mdns] Deregistering node: Client Listener Proxy-<span class="hljs-number">5512f</span>579-<span class="hljs-number">7</span>eed-<span class="hljs-number">4</span>ef6-<span class="hljs-number">8</span>c80-<span class="hljs-number">4342650</span>afc2d
<span class="hljs-number">2021</span>-<span class="hljs-number">02</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">49</span>:<span class="hljs-number">46.609502</span> I | [proxy] Registry [mdns] Deregistering node: Server Listener Proxy-<span class="hljs-number">5512f</span>579-<span class="hljs-number">7</span>eed-<span class="hljs-number">4</span>ef6-<span class="hljs-number">8</span>c80-<span class="hljs-number">4342650</span>afc2d
</code></pre>
<p data-nodeid="735">这里可以看到同时启动了两个端口 8082 和 8083，分别是 Sidecar 处理出流量和入流量的端口，同时有输出了两个 Server 代理的目的地。</p>
<p data-nodeid="736">我们先来请求 8083 端口，这个端口会将流量路由到本地的 6060 端口。为了调试，我们的 Sidecar 代码在 6060 端口启动了一个 HTTP 服务器，请求这个 Server 会返回 Hello World 的输出：</p>
<pre class="lang-java" data-nodeid="737"><code data-language="java">curl <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8083</span>/ -i

StatusCode&nbsp; &nbsp; &nbsp; &nbsp; : <span class="hljs-number">200</span>
StatusDescription : OK
Content&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: hello world
RawContent&nbsp; &nbsp; &nbsp; &nbsp; : HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; User-Agent: Mozilla/<span class="hljs-number">5.0</span> (Windows NT; Windows NT <span class="hljs-number">10.0</span>; zh-CN) WindowsPowerShell/<span class="hljs-number">5.1</span><span class="hljs-number">.18362</span><span class="hljs-number">.1171</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Length: <span class="hljs-number">11</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Type: text/plain; charset=utf-<span class="hljs-number">8</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Date: Sun, <span class="hljs-number">21</span> Feb <span class="hljs-number">2021</span> <span class="hljs-number">18</span>:<span class="hljs-number">2</span>...
</code></pre>
<p data-nodeid="738">可以看到这里返回了正常的请求，返回的内容为 Hello World，说明 8083 端口正常处理了代理的请求。为了验证内容返回是否正常，这里我们直接请求 6060 端口，查看返回的信息：</p>
<pre class="lang-java" data-nodeid="739"><code data-language="java">curl <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">6060</span>/ -i

StatusCode&nbsp; &nbsp; &nbsp; &nbsp; : <span class="hljs-number">200</span>
StatusDescription : OK
Content&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: hello world
RawContent&nbsp; &nbsp; &nbsp; &nbsp; : HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Length: <span class="hljs-number">11</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Type: text/plain; charset=utf-<span class="hljs-number">8</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Date: Sun, <span class="hljs-number">21</span> Feb <span class="hljs-number">2021</span> <span class="hljs-number">18</span>:<span class="hljs-number">27</span>:<span class="hljs-number">35</span> GMT
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; hello world
</code></pre>
<p data-nodeid="740">可以看到输出了相同的信息，表示 Sidecar 确实进行了正确的转发。</p>
<p data-nodeid="2279">下面我们再请求 8082 端口，模拟一下完整的 Mesh 过程。整个过程如下图所示：</p>
<p data-nodeid="2990" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/08/FF/CioPOWA1hs6AX1jDAAB6NkWYyhA014.png" alt="图片1.png" data-nodeid="2994"></p>
<div data-nodeid="2991"><p style="text-align:center">完整 Mesh 过程示意图</p></div>








<p data-nodeid="744">请求8082端口：</p>
<pre class="lang-java" data-nodeid="745"><code data-language="java">curl <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8082</span>/

StatusCode&nbsp; &nbsp; &nbsp; &nbsp; : <span class="hljs-number">200</span>
StatusDescription : OK
Content&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;: hello world
RawContent&nbsp; &nbsp; &nbsp; &nbsp; : HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> OK
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Accept-Encoding: gzip
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Connection: Keep-Alive
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; User-Agent: Mozilla/<span class="hljs-number">5.0</span> (Windows NT; Windows NT <span class="hljs-number">10.0</span>; zh-CN) WindowsPowerShell/<span class="hljs-number">5.1</span><span class="hljs-number">.18362</span><span class="hljs-number">.1171</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Length: <span class="hljs-number">11</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Content-Type: text/pl...
</code></pre>
<p data-nodeid="746">这里同样返回了正确的信息。结合上面的图片和代码解析可以发现，这里的流量经由 8082，被转发到了 8083 端口，经历了一个完整的 Mesh 代理过程。从客户端的正向代理，到服务端的反向代理，通过对出流量和入流量进行代理，完整掌控了整个微服务链路的流量。最后将流量转发到了真实的 Service，也就是 6060 端口启动的服务。</p>
<p data-nodeid="747">到这里，利用 Go-Micro 实现一个简单的 Sidecar 就完成了。你可以看到，Sidecar 本身就是一个代理程序，而利用 Go-Micro 实现一个 Sidecar 原型也并不复杂，但要做一个生产环境可用的 Sidecar 还有很多工作要完善，比如一些<strong data-nodeid="846">错误的处理、多种协议的支持、支持复杂的路由和负载均衡策略</strong>，以及<strong data-nodeid="847">限流熔断</strong>等基本的服务治理功能。另外，在复杂的线上流量环境保证<strong data-nodeid="848">Sidecar 自身的健壮性和稳定性</strong>，也不是一件容易的事情。</p>
<p data-nodeid="748">实际上，在 Go 语言中还有另外一种实现 HTTP 代理更简单、更稳定的方法，这里我们也简单做一下介绍。至于为什么没有着重介绍这种方法，而选择采用 Go-Micro 来实现，是因为 Go-Micro 方便扩展多种协议，以及统一的中间件层。</p>
<p data-nodeid="749">下面我们结合代码简单介绍一下 Go 原生的代理实现：</p>
<pre class="lang-java" data-nodeid="750"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">package</span> main
<span class="hljs-title">import</span> <span class="hljs-params">(
        <span class="hljs-string">"log"</span>
        <span class="hljs-string">"math/rand"</span>
        <span class="hljs-string">"net/http"</span>
        <span class="hljs-string">"net/http/httputil"</span>
        <span class="hljs-string">"net/url"</span>
)</span>
func <span class="hljs-title">NewMultipleHostsReverseProxy</span><span class="hljs-params">(targets []*url.URL)</span> *httputil.ReverseProxy </span>{
        director := func(req *http.Request) {
                target := targets[rand.Int()%len(targets)]
                req.URL.Scheme = target.Scheme
                req.URL.Host = target.Host
                req.URL.Path = target.Path
        }
        <span class="hljs-keyword">return</span> &amp;httputil.ReverseProxy{Director: director}
}
<span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
        proxy := NewMultipleHostsReverseProxy([]*url.URL{
                {
                        Scheme: <span class="hljs-string">"http"</span>,
                        Host:   <span class="hljs-string">"localhost:6060"</span>,
                },
        })
        log.Fatal(http.ListenAndServe(<span class="hljs-string">":9090"</span>, proxy))
}
</code></pre>
<p data-nodeid="751">可以看到，Golang 本身提供了 ReverseProxy 的对象。直接创建这个对象，并传入 HTTP Server，就可以实现一个简单的 HTTP Proxy 功能。对于 HTTP 服务来说，这个 Proxy 实现比 Go-Micro 的实现相对高效，它并没有创建 HTTP Client，而是直接利用了更底层的 HTTP Transport 进行流量的转发，有兴趣的同学也可以直接去查看 Golang 的源码（<a href="https://github.com/golang/go/blob/master/src/net/http/httputil/reverseproxy.go" data-nodeid="854">https://github.com/golang/go/blob/master/src/net/http/httputil/reverseproxy.go</a>）。</p>
<p data-nodeid="752">整个 Sidecar 的代码实现和讲解到这里就结束了，下面我们做一个简单的总结。</p>
<h3 data-nodeid="753">总结</h3>
<p data-nodeid="754">今天我们主要了解了 Sidecar 的代码实现，通过代码层的讲解，相信你已经了解 Sidecar 的底层原理和 Proxy 的实现原理。此外，我们通过手动运行 Demo 程序，清晰展示了整个 Mesh 数据面的运行过程，并利用这个最小化的运行环境，深入了解了 Mesh 数据面的架构。相信较于 Istio 的实战部分，今天的内容会帮助你对整个 Mesh 数据面有一个更清晰的认知。</p>
<p data-nodeid="1215">本讲内容总结如下：</p>
<p data-nodeid="1925" class=""><img src="https://s0.lgstatic.com/i/image6/M00/09/02/Cgp9HWA1hrWAMQuCAAGBGu1j9ok772.png" alt="金句.png" data-nodeid="1928"></p>






<p data-nodeid="757">结合今天内容的讲解，如果让你研发一个 Mesh 的数据面，你会用什么语言和框架呢。欢迎在留言区和我分享你的观点。</p>
<p data-nodeid="758" class="">今天的内容到这里就结束了，下一讲我们开始讲解控制面：实现 xDS 配置管理。下一讲我会实现 Service Mesh 的控制面，并将本讲实现的 Sidecar 和控制面进行通信，用于展示完整的 Service Mesh 过程。我们下一讲再见。</p>

---

### 精选评论


