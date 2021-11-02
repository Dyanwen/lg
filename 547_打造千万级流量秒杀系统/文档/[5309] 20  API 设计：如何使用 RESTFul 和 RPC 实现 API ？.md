<p data-nodeid="829" class="">在 Web 系统中，前端通过 HTTP 请求给后端传递参数有四种方式，可以将参数放在请求路径、Query 参数、HTTP 协议头、HTTP 协议体中。而放在协议体中的参数又有很多格式，比如 json、form 表单等。当然，前端也可能选择其他协议，比如 Websocket、gRPC-Web 等，具体的参数形式跟 HTTP 又不一样。</p>
<p data-nodeid="830">面对这么多种技术实现方式，当我们要设计一套 Web 系统时，该如何选择呢？应当遵守什么样的规范呢？这就是我接下来要给你详细介绍的。</p>
<h3 data-nodeid="831">RESTful 解决什么问题？</h3>
<p data-nodeid="832">首先，我们来看下三个时间点：</p>
<ul data-nodeid="833">
<li data-nodeid="834">
<p data-nodeid="835">1991 年 HTTP 0.9 诞生</p>
</li>
<li data-nodeid="836">
<p data-nodeid="837">1996 年 5 月 HTTP 1.0 诞生</p>
</li>
<li data-nodeid="838">
<p data-nodeid="839">1997 年 1 月 HTTP 1.1 诞生</p>
</li>
</ul>
<p data-nodeid="840">在 HTTP 1.0 出现以前，也就是 HTTP 0.9 时代，HTTP 协议只支持 GET 请求，所有参数只能通过 URL 传递。比如一个获取动态网页的请求可能是这样的 GET /index.php?page=hello&amp;user=1234&amp;action=view 。</p>
<p data-nodeid="841">在 HTTP 1.0 出现后，开始有了 POST 请求和 HEAD 请求，支持从 HTTP 协议头和协议体传参数。但 HTTP 1.0 并没有解决每次请求都需要建立新连接的问题，然后 HTTP 1.1 很快就出现了。</p>
<p data-nodeid="842">HTTP 1.1 除了能保持连接外，还新增了多种方法，如 PUT、DELETE、PATCH、OPTIONS。虽然功能更强大，体验更丰富了，却带来了新的问题：这些方法该如何选择呢？比如我要上传数据，到底是用 POST 、PUT 还是 PATCH 呢？这无疑增加了选择成本。</p>
<p data-nodeid="843">随后，REST（ Representational State Transfer，表现层状态转移） 出现了，简单来说它就是一组架构约束条件和原则，而符合 REST 规则的设计便是 RESTful。</p>
<p data-nodeid="844">因为 HTTP 协议本身是无状态的，但后端数据是有状态的，如何用无状态的协议来操作有状态的数据就比较有挑战。比如，当只用 GET 请求的时候，你无法从请求参数上直观判断到底是对数据做什么操作，因为大家对 GET 请求参数命名没有统一规范。在前面的例子中，action=view 也有可能被定义成 a=v。</p>
<p data-nodeid="845">RESTful 是如何解决这个问题的呢？它充分利用了各种 HTTP 请求方法的特点，来表达对数据的具体操作方式。比如当你要新增数据时，应当用 POST 方法；要整个替换数据时，应当用 PUT 方法；仅仅是替换数据的部分字段时，应当用 PATCH 方法；如果是删除数据，应当用 DELETE 方法。这样一来，对数据的操作就清晰明了。</p>
<h3 data-nodeid="846">RPC 解决什么问题？</h3>
<p data-nodeid="847">RPC 的全称叫 Remote Procedure Call，也就是远程过程调用。它和 REST 都是客户端向服务端发起请求的技术手段。</p>
<p data-nodeid="848">前面提到了 RESTful 能解决很多问题，但为何还需要 RPC 呢？</p>
<p data-nodeid="849">首先，前面提到了从 HTTP 1.0 开始，HTTP 协议增加了很多功能。在 HTTP2 出现前，HTTP 协议内容都是文本格式，功能越来越多导致协议内容变得越来越臃肿。在一些高性能场景下，特别是系统内部服务之间频繁请求的时候，臃肿的协议会带来不少网络开销，影响服务性能。而 RPC 在底层协议可以选择直接使用 TCP 这种长连接，协议内容可以做到比较精简。RPC 还可以对数据进行压缩后再传输，性能要比 HTTP 好很多。</p>
<p data-nodeid="850">其次，RESTful 只是一种约定，而不是一种强制规范，你可以遵守，也可以不遵守。这就导致不同技术水平的人，实现出来的接口风格和行为不一致。</p>
<p data-nodeid="851">比如，新手程序员在用 HTTP 协议实现 API 的时候，可能不小心给客户端多返回一些额外的数据，客户端也不一定会报错，但却存在数据泄漏的风险。而 RPC 通常利用像 protobuf 这种 IDL （Interactive Data Language ，交互式数据语言）来定义接口规范，并生成客户端和服务端的框架代码。</p>
<p data-nodeid="852">其中，IDL的作用是定义客户端和服务端之间的数据交互方式，利用生成的框架代码，在双方开发代码时形成强约束，是契约式编程的一种体现。在使用 RPC 的时候，服务端只能返回 IDL 中定义的数据，否则使用框架代码时会报错。</p>
<p data-nodeid="853"><strong data-nodeid="943">目前，主流跨平台 RPC 协议主要有 Thrift 和 gRPC</strong>。Thrift 是由 Facebook 开源的，底层主要基于 TCP 传输数据。而 gRPC 是由谷歌开源的，底层基于 HTTP2 协议。gRPC 由于支持丰富的中间件以及双向流模式，具有很强的易用性，应用越来越广泛。比如，ETCD 就提供了 gRPC 接口。</p>
<h3 data-nodeid="854">秒杀 API 设计</h3>
<p data-nodeid="855">**秒杀系统中的 API 主要有两大块：API 服务、Admin 服务。**接下来，我将给你介绍下 RESTful 和 gRPC 在这两个服务中的应用。</p>
<h4 data-nodeid="856">API 服务 API 设计</h4>
<p data-nodeid="857">根据我们之前做的需求分析，秒杀 API 服务主要给用户提供活动信息和抢购商品的功能。那么，秒杀 API 服务的 API 应当分为活动信息和抢购这两组，我们分别用 event 和 shop 来表示。</p>
<p data-nodeid="858">在 event 功能中，我们需要提供三个接口给前端：</p>
<p data-nodeid="859">第一个是 /event/list ，采用 GET 方法，用于获取所有正在进行或者即将进行的活动列表；</p>
<p data-nodeid="860">第二个是 /event/info，采用 GET 方法，用于查询某个商品当前的秒杀活动信息，参数是商品 ID；</p>
<p data-nodeid="861">第三个是 /event/subscribe，采用 POST 方法，用于订阅某商品的活动开始通知，参数是活动场次 ID 、商品 ID、设备 ID。</p>
<p data-nodeid="862">在 shop 功能中，我们只需要提供一个抢购接口即可：/shop/cart/add，具体可采用 PUT 方法，也就是将商品加到购物车，参数为商品 ID。这里之所以用 PUT 方法，是因为用户的购物车是一直存在的，用 PUT 方法是遵循 RESTful。</p>
<p data-nodeid="863">这些接口，都会返回一些相同的信息，比如：错误号、提示信息、用户是否已登录、数据等。它们可以用 Go 语言中的结构体定义，如下所示：</p>
<pre class="lang-go" data-nodeid="864"><code data-language="go"><span class="hljs-keyword">type</span> Response <span class="hljs-keyword">struct</span> {
   Code        <span class="hljs-keyword">int</span>         <span class="hljs-string">`json:"code"`</span>         <span class="hljs-comment">// 业务错误码</span>
   Data        <span class="hljs-keyword">interface</span>{} <span class="hljs-string">`json:"data"`</span>         <span class="hljs-comment">// 数据</span>
   Msg         <span class="hljs-keyword">string</span>      <span class="hljs-string">`json:"msg"`</span>          <span class="hljs-comment">// 提示信息</span>
}
</code></pre>
<p data-nodeid="865">需要注意的是，为了支持返回多种类型的数据，需要将结构体中的 Data 字段定义为 interface 类型。另外，登录状态 Login-Status 将放到 HTTP Header 中返回给前端。为了让 admin 也能复用这个结构体定义，我们需要将它放在 infrastructure/utils/response.go 里。</p>
<p data-nodeid="866">接下来，我们在 application/api 中定义 Event 和 Shop 这两个应用，并定义与前面接口对应的处理函数，代码如下：</p>
<pre class="lang-go" data-nodeid="867"><code data-language="go"><span class="hljs-keyword">type</span> Event <span class="hljs-keyword">struct</span>{
<span class="hljs-keyword">type</span> Shop <span class="hljs-keyword">struct</span>{
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(e *Event)</span> <span class="hljs-title">List</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event list"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(e *Event)</span> <span class="hljs-title">Info</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event info"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(e *Event)</span> <span class="hljs-title">Subscribe</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event subscribe"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *Shop)</span> <span class="hljs-title">AddCart</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"shop add cart"</span>)
   ctx.JSON(status, resp)
}
</code></pre>
<p data-nodeid="868">接下来，我们实现一个 initRouters 函数，将前面实现的接口函数注册到 Gin 框架的路由中。代码在 interfaces/api 目录下的 routers.go 文件中，如下所示：</p>
<pre class="lang-go" data-nodeid="869"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">initRouters</span><span class="hljs-params">(g *gin.Engine)</span></span> {
   logrus.Info(<span class="hljs-string">"init api routers"</span>)
   event := g.Group(<span class="hljs-string">"/event"</span>)
   eventApp := api.Event{}
   event.GET(<span class="hljs-string">"/list"</span>, eventApp.List)
   event.GET(<span class="hljs-string">"/info"</span>, eventApp.Info)
   event.POST(<span class="hljs-string">"/subscribe"</span>, eventApp.Subscribe)

   shop := g.Group(<span class="hljs-string">"/shop"</span>)
   shopApp := api.Shop{}
   shop.PUT(<span class="hljs-string">"/cart/add"</span>, shopApp.AddCart)
}
</code></pre>
<p data-nodeid="870">然后我们就可以在 api.go 文件的 Run 函数中调用 initRouters 函数，以便将路由注册到 Gin 框架中。</p>
<p data-nodeid="871">除了需要实现给浏览器请求的 API 接口外，我们还需要实现一个给 admin 请求的 RPC 服务，用于同步活动配置，主要提供专题、场次的上线和下线功能。代码在 application/api/rpc 目录下的 event.proto， 定义如下：</p>
<pre class="lang-java" data-nodeid="872"><code data-language="java">syntax = <span class="hljs-string">"proto3"</span>;
<span class="hljs-keyword">package</span> rpc;
message Goods {
  int32 id = <span class="hljs-number">1</span>;
  string desc = <span class="hljs-number">2</span>;
  string img = <span class="hljs-number">3</span>;
  string price = <span class="hljs-number">4</span>;
  string event_price = <span class="hljs-number">5</span>;
  int32 event_stock = <span class="hljs-number">6</span>;
}
message Event {
  int32 id = <span class="hljs-number">1</span>;
  int32 topic_id = <span class="hljs-number">2</span>;
  int64 start_time = <span class="hljs-number">3</span>;
  int64 end_time = <span class="hljs-number">4</span>;
  int32 limit = <span class="hljs-number">5</span>;
  repeated Goods goods_list = <span class="hljs-number">6</span>;
}
message Topic {
  int32 id = <span class="hljs-number">1</span>;
  string title = <span class="hljs-number">2</span>;
  string desc = <span class="hljs-number">3</span>;
  string banner = <span class="hljs-number">4</span>;
  int64 start_time = <span class="hljs-number">5</span>;
  int64 end_time = <span class="hljs-number">6</span>;
}
message Response {
  int32 code = <span class="hljs-number">1</span>;
  string msg = <span class="hljs-number">2</span>;
}
service EventRPC {
  <span class="hljs-function">rpc <span class="hljs-title">EventOnline</span><span class="hljs-params">(Event)</span> <span class="hljs-title">returns</span><span class="hljs-params">(Response)</span></span>;
  <span class="hljs-function">rpc <span class="hljs-title">EventOffline</span><span class="hljs-params">(Event)</span> <span class="hljs-title">returns</span><span class="hljs-params">(Response)</span></span>;
  <span class="hljs-function">rpc <span class="hljs-title">TopicOnline</span><span class="hljs-params">(Topic)</span> <span class="hljs-title">returns</span><span class="hljs-params">(Response)</span></span>;
  <span class="hljs-function">rpc <span class="hljs-title">TopicOffline</span><span class="hljs-params">(Topic)</span> <span class="hljs-title">returns</span><span class="hljs-params">(Response)</span></span>;
}
</code></pre>
<p data-nodeid="873">你可以看到，我在该 protobuf 文件中定义了一个名为 EventRPC 的 RPC 服务，以及对应的参数和返回值类型，这些将会在后面生成服务端和客户端的 Go 代码。<br>
我们要如何使用这个 protobuf 文件呢？让我们在 Makefile 中添加编译 protobuf 的指令 proto，如下所示：</p>
<pre class="lang-java" data-nodeid="874"><code data-language="java">proto: application/api/rpc/event.proto
   protoc --go_out=plugins=grpc:./ application/api/rpc/event.proto
</code></pre>
<p data-nodeid="875">通过执行 make proto，我们就可以在 application/api/rpc 目录下编译出一个 event.pb.go 文件。这个文件里面就是 RPC 接口的 Go 定义，包括服务端和客户端的定义，它便是秒杀 Admin 服务与 API 服务通信的契约。具体的定义如下：</p>
<pre class="lang-go" data-nodeid="876"><code data-language="go"><span class="hljs-comment">// For semantics around ctx use and closing/ending streaming RPCs, please refer to https://godoc.org/google.golang.org/grpc#ClientConn.NewStream.</span>
<span class="hljs-keyword">type</span> EventRPCClient <span class="hljs-keyword">interface</span> {
   EventOnline(ctx context.Context, in *Event, opts ...grpc.CallOption) (*Response, error)
   EventOffline(ctx context.Context, in *Event, opts ...grpc.CallOption) (*Response, error)
   TopicOnline(ctx context.Context, in *Topic, opts ...grpc.CallOption) (*Response, error)
   TopicOffline(ctx context.Context, in *Topic, opts ...grpc.CallOption) (*Response, error)
}
<span class="hljs-comment">// EventRPCServer is the server API for EventRPC service.</span>
<span class="hljs-keyword">type</span> EventRPCServer <span class="hljs-keyword">interface</span> {
   EventOnline(context.Context, *Event) (*Response, error)
   EventOffline(context.Context, *Event) (*Response, error)
   TopicOnline(context.Context, *Topic) (*Response, error)
   TopicOffline(context.Context, *Topic) (*Response, error)
}
</code></pre>
<p data-nodeid="877">然后，我们还需要按照 event.pb.go 的定义实现 RPC 服务。具体代码在 application/api/rpc.go 中，如下所示：</p>
<pre class="lang-go" data-nodeid="878"><code data-language="go"><span class="hljs-keyword">type</span> EventRPCServer <span class="hljs-keyword">struct</span> {
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *EventRPCServer)</span> <span class="hljs-title">EventOnline</span><span class="hljs-params">(ctx context.Context, evt *rpc.Event)</span> <span class="hljs-params">(*rpc.Response, error)</span></span> {
   logrus.Info(<span class="hljs-string">"event online "</span>, evt)
   resp := &amp;rpc.Response{}
   <span class="hljs-keyword">return</span> resp, <span class="hljs-literal">nil</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *EventRPCServer)</span> <span class="hljs-title">EventOffline</span><span class="hljs-params">(ctx context.Context, evt *rpc.Event)</span> <span class="hljs-params">(*rpc.Response, error)</span></span> {
   logrus.Info(<span class="hljs-string">"event offline "</span>, evt)
   resp := &amp;rpc.Response{}
   <span class="hljs-keyword">return</span> resp, <span class="hljs-literal">nil</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *EventRPCServer)</span> <span class="hljs-title">TopicOnline</span><span class="hljs-params">(ctx context.Context, t *rpc.Topic)</span> <span class="hljs-params">(*rpc.Response, error)</span></span> {
   logrus.Info(<span class="hljs-string">"topic online "</span>, t)
   resp := &amp;rpc.Response{}
   <span class="hljs-keyword">return</span> resp, <span class="hljs-literal">nil</span>
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(s *EventRPCServer)</span> <span class="hljs-title">TopicOffline</span><span class="hljs-params">(ctx context.Context, t *rpc.Topic)</span> <span class="hljs-params">(*rpc.Response, error)</span></span> {
   logrus.Info(<span class="hljs-string">"topic offline "</span>, t)
   resp := &amp;rpc.Response{}
   <span class="hljs-keyword">return</span> resp, <span class="hljs-literal">nil</span>
}
</code></pre>
<p data-nodeid="879">同样地，我们也要像 API 服务那样提供启动和退出相关的函数，代码在 interfaces/rpc 目录下的 rpc.go 文件中，如下所示：</p>
<pre class="lang-go" data-nodeid="880"><code data-language="go"><span class="hljs-keyword">var</span> grpcS *grpc.Server
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Run</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   bind := viper.GetString(<span class="hljs-string">"api.rpc"</span>)
   logrus.Info(<span class="hljs-string">"run RPC server on "</span>, bind)
   lis, err := utils.Listen(<span class="hljs-string">"tcp"</span>, bind)
   <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> err
   }
   grpcS = grpc.NewServer()
   eventRPC := &amp;api.EventRPCServer{}
   rpc.RegisterEventRPCServer(grpcS, eventRPC)
   <span class="hljs-comment">// 支持 gRPC reflection，方便调试</span>
   reflection.Register(grpcS)
   <span class="hljs-keyword">return</span> grpcS.Serve(lis)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Exit</span><span class="hljs-params">()</span></span> {
   grpcS.GracefulStop()
   logrus.Info(<span class="hljs-string">"rpc server exit"</span>)
}
</code></pre>
<p data-nodeid="881">接下来，我们还需要在启动命令中加上启动 RPC 服务的代码。具体代码你可以看代码仓库中的 <a href="https://github.com/lagoueduCol/MiaoSha-Yiletian/blob/main/cmd/api.go" data-nodeid="973">cmd/api.go</a>。</p>
<p data-nodeid="882">最后，我们编译出可执行程序并运行，效果如下：</p>
<p data-nodeid="883"><img src="https://s0.lgstatic.com/i/image2/M01/05/DF/CgpVE2ABOgyABtV7AAHfW3MYaP4035.png" alt="Drawing 0.png" data-nodeid="978"></p>
<p data-nodeid="884">你可以看到，当请求 /event/list 接口时，服务输出了 "event list" 日志。使用 grpcurl 工具请求 EventRPC 的 EventOnline 接口时，服务输出了 "event online" 日志。</p>
<h4 data-nodeid="885">Admin 服务 API 设计</h4>
<p data-nodeid="886">首先，我们回顾下需求分析时介绍的管理后台功能需求，主要有这几块：专题管理、场次管理、商品管理。每个模块分别有：列表、创建、查看、修改、删除等功能。其中，专题和场次还有上线、下线这两个功能。那么，我们需要实现的接口清单如下所示：</p>
<pre class="lang-powershell" data-nodeid="887"><code data-language="powershell"><span class="hljs-comment"># 创建专题</span>
POST /topic
<span class="hljs-comment"># 获取专题列表</span>
GET /topic?page=<span class="hljs-number">1</span>&amp;size=<span class="hljs-number">10</span>
<span class="hljs-comment"># 查看某个专题</span>
GET /topic/{id}
<span class="hljs-comment"># 修改某个专题</span>
PUT /topic/{id}
<span class="hljs-comment"># 上线/下线某个专题</span>
PUT /topic/{id}/{status}
<span class="hljs-comment"># 删除某个专题</span>
DELETE /topic/{id}
<span class="hljs-comment"># 创建场次</span>
POST /event
<span class="hljs-comment"># 获取场次列表</span>
GET /eventpage=<span class="hljs-number">1</span>&amp;size=<span class="hljs-number">10</span>
<span class="hljs-comment"># 查看某个场次</span>
GET /event/{id}
<span class="hljs-comment"># 修改某个场次</span>
PUT /event/{id}
<span class="hljs-comment"># 上线/下线某个场次</span>
PUT /event/{id}/{status}
<span class="hljs-comment"># 删除某个场次</span>
DELETE /event/{id}
<span class="hljs-comment"># 创建商品</span>
POST /goods
<span class="hljs-comment"># 获取商品列表</span>
GET /goods?page=<span class="hljs-number">1</span>&amp;size=<span class="hljs-number">10</span>
<span class="hljs-comment"># 查看某个商品</span>
GET /goods/{id}
<span class="hljs-comment"># 修改某个商品</span>
PUT /goods/{id}
<span class="hljs-comment"># 删除某个商品</span>
DELETE /goods/{id}
</code></pre>
<p data-nodeid="888">由于接口比较多，我们在 application/admin 目录下新建三个文件，分别是 topic.go、event.go、goods.go，用于实现 admin 的 API 代码。</p>
<p data-nodeid="889">topic.go 中定义了 Topic 这个应用，它包含 4 个方法，分别是<strong data-nodeid="996">Post、Get、Put、Delete</strong>，代码如下：</p>
<pre class="lang-go" data-nodeid="890"><code data-language="go"><span class="hljs-keyword">type</span> Topic <span class="hljs-keyword">struct</span>{
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Topic)</span> <span class="hljs-title">Post</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"topic post"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Topic)</span> <span class="hljs-title">Get</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"topic get"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Topic)</span> <span class="hljs-title">Put</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"topic put"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Topic)</span> <span class="hljs-title">Delete</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"topic delete"</span>)
   ctx.JSON(status, resp)
}
</code></pre>
<p data-nodeid="891">event.go 中定义了 Event 这个应用，它的 4 个方法命名跟 topic 的一样，代码如下：</p>
<pre class="lang-go" data-nodeid="892"><code data-language="go"><span class="hljs-keyword">type</span> Event <span class="hljs-keyword">struct</span>{}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Event)</span> <span class="hljs-title">Post</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event post"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Event)</span> <span class="hljs-title">Get</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event get"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Event)</span> <span class="hljs-title">Put</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event put"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Event)</span> <span class="hljs-title">Delete</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"event delete"</span>)
   ctx.JSON(status, resp)
}
</code></pre>
<p data-nodeid="893">goods.go 中主要是定义了 Goods 这个应用，代码如下：</p>
<pre class="lang-go" data-nodeid="894"><code data-language="go"><span class="hljs-keyword">type</span> Goods <span class="hljs-keyword">struct</span>{}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Goods)</span> <span class="hljs-title">Post</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"goods post"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Goods)</span> <span class="hljs-title">Get</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"goods get"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Goods)</span> <span class="hljs-title">Put</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"goods put"</span>)
   ctx.JSON(status, resp)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-params">(t *Goods)</span> <span class="hljs-title">Delete</span><span class="hljs-params">(ctx *gin.Context)</span></span> {
   resp := &amp;utils.Response{
      Code: <span class="hljs-number">0</span>,
      Data: <span class="hljs-literal">nil</span>,
      Msg:  <span class="hljs-string">"ok"</span>,
   }
   status := http.StatusOK
   logrus.Info(<span class="hljs-string">"goods delete"</span>)
   ctx.JSON(status, resp)
}
</code></pre>
<p data-nodeid="895">接下来，我们需要参考前面 api 命令的实现，将应用关联到路由并注入框架中，也就是在 interfaces/admin/routers.go 中实现 initRouters 函数。代码如下：</p>
<pre class="lang-go" data-nodeid="896"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">initRouters</span><span class="hljs-params">(g *gin.Engine)</span></span> {
   topic := g.Group(<span class="hljs-string">"/topic"</span>)
   topicApp := admin.Topic{}
   topic.POST(<span class="hljs-string">"/"</span>, topicApp.Post)
   topic.GET(<span class="hljs-string">"/"</span>, topicApp.Get)
   topic.GET(<span class="hljs-string">"/:id"</span>, topicApp.Get)
   topic.PUT(<span class="hljs-string">"/:id"</span>, topicApp.Put)
   topic.PUT(<span class="hljs-string">"/:id/:status"</span>, topicApp.Put)
   topic.DELETE(<span class="hljs-string">"/:id"</span>, topicApp.Delete)

   event := g.Group(<span class="hljs-string">"/event"</span>)
   eventApp := admin.Event{}
   event.POST(<span class="hljs-string">"/"</span>, eventApp.Post)
   event.GET(<span class="hljs-string">"/"</span>, eventApp.Get)
   event.GET(<span class="hljs-string">"/:id"</span>, eventApp.Get)
   event.PUT(<span class="hljs-string">"/:id"</span>, eventApp.Put)
   event.PUT(<span class="hljs-string">"/:id/:status"</span>, eventApp.Put)
   event.DELETE(<span class="hljs-string">"/:id"</span>, eventApp.Delete)

   goods := g.Group(<span class="hljs-string">"/goods"</span>)
   goodsApp := admin.Goods{}
   goods.POST(<span class="hljs-string">"/"</span>, goodsApp.Post)
   goods.GET(<span class="hljs-string">"/"</span>, goodsApp.Get)
   goods.GET(<span class="hljs-string">"/:id"</span>, goodsApp.Get)
   goods.PUT(<span class="hljs-string">"/:id"</span>, goodsApp.Put)
   goods.DELETE(<span class="hljs-string">"/:id"</span>, goodsApp.Delete)
}
</code></pre>
<p data-nodeid="897">除此之外，我们还需要启动 admin 服务，以及处理服务退出时关闭连接的问题。这里我将 interfaces/api 目录下的 api.go 做了适当调整，把关于接口重用和程序更新的代码放到了 infrastructure/utils 目录下的 proc.go 和 listen.go 中，以便给 admin 复用。最终，我们在 interfaces/admin 目录下实现的 admin.go 代码如下：</p>
<pre class="lang-go" data-nodeid="898"><code data-language="go"><span class="hljs-keyword">var</span> lis net.Listener
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Run</span><span class="hljs-params">()</span> <span class="hljs-title">error</span></span> {
   <span class="hljs-keyword">var</span> err error
   bind := viper.GetString(<span class="hljs-string">"admin.bind"</span>)
   lis, err = utils.Listen(<span class="hljs-string">"tcp"</span>, bind)
   <span class="hljs-keyword">if</span> err != <span class="hljs-literal">nil</span> {
      <span class="hljs-keyword">return</span> err
   }
   g := gin.New()
   <span class="hljs-comment">// 更新程序，给老版本发送信号</span>
   <span class="hljs-keyword">go</span> utils.UpdateProc(<span class="hljs-string">"admin"</span>)
   <span class="hljs-comment">// 初始化路由</span>
   initRouters(g)
   <span class="hljs-comment">// 运行服务</span>
   <span class="hljs-keyword">return</span> g.RunListener(lis)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">Exit</span><span class="hljs-params">()</span></span> {
   lis.Close()
   <span class="hljs-comment">// <span class="hljs-doctag">TODO:</span> 等待请求处理完</span>
   <span class="hljs-comment">// time.Sleep(10 * time.Second)</span>
}
</code></pre>
<p data-nodeid="899">接下来，我们就可以在 cmd/admin.go 中解析命令并启动 admin 了。代码如下：</p>
<pre class="lang-go" data-nodeid="900"><code data-language="go"><span class="hljs-keyword">var</span> adminCmd = &amp;cobra.Command{
   Use:   <span class="hljs-string">"admin"</span>,
   Short: <span class="hljs-string">"Seckill admin server."</span>,
   Long:  <span class="hljs-string">`Seckill admin server.`</span>,
   Run: <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">(cmd *cobra.Command, args []<span class="hljs-keyword">string</span>)</span></span> {
      onExit := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> error)
      <span class="hljs-keyword">go</span> <span class="hljs-function"><span class="hljs-keyword">func</span><span class="hljs-params">()</span></span> {
         <span class="hljs-keyword">if</span> err := admin.Run(); err != <span class="hljs-literal">nil</span> {
            logrus.Error(err)
            onExit &lt;- err
         }
         <span class="hljs-built_in">close</span>(onExit)
      }()
      onSignal := <span class="hljs-built_in">make</span>(<span class="hljs-keyword">chan</span> os.Signal)
      signal.Notify(onSignal, syscall.SIGINT, syscall.SIGTERM)
      <span class="hljs-keyword">select</span> {
      <span class="hljs-keyword">case</span> sig := &lt;-onSignal:
         logrus.Info(<span class="hljs-string">"exit by signal "</span>, sig)
         admin.Exit()
      <span class="hljs-keyword">case</span> err := &lt;-onExit:
         logrus.Info(<span class="hljs-string">"exit by error "</span>, err)
      }
   },
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">init</span><span class="hljs-params">()</span></span> {
   rootCmd.AddCommand(adminCmd)
}
</code></pre>
<p data-nodeid="901">最终编译出来的程序运行效果如下：</p>
<p data-nodeid="902"><img src="https://s0.lgstatic.com/i/image2/M01/05/DF/CgpVE2ABOiWAATWXAAMRlcrJwes939.png" alt="Drawing 1.png" data-nodeid="1005"></p>
<p data-nodeid="903">你可以看到，我在命令行下同时运行了 admin 和 api 服务，并且分别给它们发送请求都能正常接收。</p>
<p data-nodeid="1468">到这里，秒杀系统的 API 就设计完毕，并实现了最基础的框架。你可以看到，在整个过程中，我并不是一次性将某个功能开发完毕，而是从外到内逐渐深入，并随时验证功能正确性。这个过程叫<strong data-nodeid="1475">渐进式开发</strong>，是软件开发中常用的方法。</p>
<p data-nodeid="1469" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/05/E8/Cip5yGABVDWAGn1vAAobGj3vHUg526.png" alt="1.png" data-nodeid="1478"></p>

<h3 data-nodeid="1253">小结</h3>




<p data-nodeid="906">这一讲我主要给你介绍了 RESTful 和 RPC 解决的问题，以及通过渐进式开发来设计并搭建好秒杀系统的框架。通过代码实战，希望你能掌握 RESTful、RPC 以及渐进式开发的技巧，并能应用到实际工作中。</p>
<p data-nodeid="907">前后端的技术日益发展，它们之间的交互方式也一直是个需要持续关注的事情。作为开发人员或者架构师，需要学会从众多技术中选出适合自己业务的技术，以便兼顾开发效率、维护成本、用户体验。</p>
<p data-nodeid="908">接下来你可以思考下：如果将秒杀 API 服务的 HTTP 接口也用 RPC 实现，应该怎么设计呢？</p>
<p data-nodeid="909">好了，这一讲就到这里了，下一讲我将给你介绍“如何使用 etcd 存储配置信息”。到时见！</p>
<p data-nodeid="910">源码地址：<a href="https://github.com/lagoueduCol/MiaoSha-Yiletian" data-nodeid="1021">https://github.com/lagoueduCol/MiaoSha-Yiletian</a></p>
<hr data-nodeid="911">
<p data-nodeid="912"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1026"><img src="https://s0.lgstatic.com/i/image/M00/80/32/CgqCHl_QgX2AHJo_ACRP1TPc6yM423.png" alt="Drawing 15.png" data-nodeid="1025"></a></p>
<p data-nodeid="913"><strong data-nodeid="1030">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="914" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="1034">点击链接，提升自己</a>！</p>

---

### 精选评论


