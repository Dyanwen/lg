<p data-nodeid="65576">在上一课时中，我们对比分析了 gRPC 和 Thrift ，发现二者都缺少了大量的功能，比如：连接池、服务框架、服务发现、服务治理、分布式链路追踪、埋点和上下文日志等，这些功能才是日常开发和运维最常使用的。而 Go-kit 提供了上述的功能，并且还可以和 gRPC 进行结合，所以今天我们就来详细讲解一下 Go-kit 和 gRPC 结合的基本原理和实战案例。</p>
<p data-nodeid="65577">Go-kit 框架可以和 gRPC 结合使用，将 RPC 作为传输层的组件，而自身则提供诸如服务注册和发现、断路器等微服务远程交互的通用功能组件。比如，gRPC 缺乏服务治理的功能，我们就可以通过 Go-kit 结合 gRPC 来弥补这一缺陷。Go-kit 框架抽象的 Endpoint 层设计让开发者可以很容易地封装使用其他微服务组件，比如说服务注册与发现、断路器和负载均衡策略等。</p>
<p data-nodeid="65578">在前面 08 课时中我们已经详细介绍了如何使用 Go-kit 构建简单的 Web 系统，那本课时我们就只对 Go-kit 涉及网络通信相关的模块和功能进行更加深入的介绍：① Go-kit 和 gRPC结合的具体原理；②Go-kit 集成 gRPC 的项目实践。</p>
<h3 data-nodeid="65579">Go-kit 和 gRPC 结合的原理</h3>
<p data-nodeid="65580">在前面的课时中，我们介绍过 Go-kit 是一套强大的微服务开发工具集，用于指导开发人员解决分布式系统开发过程中所遇到的问题，帮助开发人员更专注于业务开发。Go-kit 推荐使用 Transport、Endpoint 和 Service 这 3 层结构来组织项目。若是你对这三者的具体概念还不熟悉的话，可以去复习一下前边的课时。</p>
<p data-nodeid="65581">由于本课时主要涉及 Transport 层和 Endpoint 层，所以这里我们就再重申一下这二者的概念。</p>
<ul data-nodeid="65582">
<li data-nodeid="65583">
<p data-nodeid="65584">Transport 层，主要负责网络传输，例如处理HTTP、gRPC、Thrift等相关的逻辑。</p>
</li>
<li data-nodeid="65585">
<p data-nodeid="65586">Endpoint 层，主要负责 request/response 格式的转换，以及公用拦截器相关的逻辑。作为 Go-kit 的核心，Endpoint 层采用类似洋葱的模型，提供了对日志、限流、熔断、链路追踪和服务监控等方面的扩展能力。</p>
</li>
</ul>
<p data-nodeid="65587"><strong data-nodeid="65660">Go-kit 和 gRPC 结合的关键在于需要将 gRPC 集成到 Go-kit 的 Transport 层</strong>。Go-kit 的 Transport 层用于接收用户网络请求并将其转为 Endpoint 可以处理的对象，然后交由 Endpoint 层执行，最后再将处理结果转为响应对象返回给客户端。为了完成这项工作，Transport 层需要具备两个工具方法：</p>
<ul data-nodeid="65588">
<li data-nodeid="65589">
<p data-nodeid="65590"><strong data-nodeid="65665">解码器</strong>，把用户的请求内容转换为请求对象；</p>
</li>
<li data-nodeid="65591">
<p data-nodeid="65592"><strong data-nodeid="65670">编码器</strong>，把处理结果转换为响应对象。</p>
</li>
</ul>
<p data-nodeid="66969">gRPC 请求的处理过程如下图所示，服务端接收到一个客户端请求后，交由 grpc_transport.Handler 处理，它会调用 DecodeRequestFunc 进行解码，然后交给其 Endpoint 层转换为 Service 层能处理的对象，将返回值通过 EncodeResponseFunc 编码，最后返回给客户端。</p>
<p data-nodeid="67279"><img src="https://s0.lgstatic.com/i/image/M00/47/A5/CgqCHl9InuGAYQa8AABJDed6WN0517.png" alt="Drawing 0.png" data-nodeid="67283"></p>
<div data-nodeid="67582" class=""><p style="text-align:center">Go-kit 过程调用示意图</p></div>






<p data-nodeid="65596" class="">接下来，我们就按照上述的流程，实现通过 Go-kit 进行 gRPC 调用。</p>
<h3 data-nodeid="65597">Go-kit 集成 gRPC 案例实战</h3>
<p data-nodeid="65598">本课时中，我们的案例使用的是上一课时案例中所定义的 proto 文件并生成相应的代码，这里不再赘述。下面我们主要分五个步骤来构建基于 gRPC 的 Go-kit 项目，分别为：定义并构建Service、Endpoint、Middleware、Transport 和组装服务端。</p>
<h4 data-nodeid="70276" class="">1. 定义 Service，提供业务实现</h4>





<p data-nodeid="65602">首先定义了 UserService 结构，它有一个名为 CheckPassword 的 grpc_transport.Handler 的方法。这个方法会调用 grpc_transport.Handler 的 ServeGRPC 方法来将请求交由 Go-kit 处理。</p>
<pre class="lang-java" data-nodeid="68778"><code data-language="java"><span class="hljs-comment">// 定义接口</span>
type UserService <span class="hljs-class"><span class="hljs-keyword">interface</span> </span>{
 &nbsp; &nbsp;CheckPassword(ctx context.Context, username string, password string) (bool, error);
}
​
type UserServiceImpl struct {}
​
func (s UserServiceImpl) CheckPassword(ctx context.Context, username string, password string) (bool, error) {
  <span class="hljs-keyword">if</span> username == <span class="hljs-string">"admin"</span> &amp;&amp; password == <span class="hljs-string">"admin"</span> {
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>, nil
  }
  <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>, nil
}
</code></pre>




<p data-nodeid="65604">和上一课时中 UserService 的实现对比，我们会发现 UserService 不再以 LoginRequest 和 LoginResponse 作为输入参数和输出对象，这是因为 Go-kit 的分层理念，认为 Request 和 Response 代表请求和响应，应该是 Transport 层和 Endpoint 层处理的概念，而 Service 层只需要处理最为纯粹的业务逻辑即可，不需要感知或了解请求和响应这些概念。</p>
<h4 data-nodeid="71719" class="">2. 定义 Endpoint，提供参数转换能力</h4>





<p data-nodeid="65608">接下来我们需要建立对应的 Endpoint。它应该是将请求转发给上述的 UserService 处理，然后定义编解码函数 DecodeRequest 和 EncodeResponse，具体代码如下所示：</p>
<pre class="lang-java" data-nodeid="72836"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">MakeUserEndpoint</span><span class="hljs-params">(svc UserService)</span> endpoint.Endpoint </span>{
  <span class="hljs-keyword">return</span> func(ctx context.Context, form <span class="hljs-class"><span class="hljs-keyword">interface</span></span>{}) (result <span class="hljs-class"><span class="hljs-keyword">interface</span></span>{}, err error) {
    req := form.(LoginForm)
    ret, err := svc.CheckPassword(ctx,req.Username, req.Password)
    <span class="hljs-keyword">return</span> LoginResult{Ret: ret, Err: err}, nil
  }
}
​
type LoginForm struct {
  Username &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; string `json:<span class="hljs-string">"username"</span>`
  Password &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; string `json:<span class="hljs-string">"password"</span>`
}
​
type LoginResult struct {
  Ret bool `json:<span class="hljs-string">"ret"</span>`
  Err &nbsp;error &nbsp;`json:<span class="hljs-string">"err"</span>`
}
​
</code></pre>




<p data-nodeid="65610">同样的，Endpoint 层也未直接处理 LoginRequest 和 LoginResponse，而是直接处理 LoginForm 和 LoginResult，使用 DecodeLoginRequest 函数将 LoginRequest 转换成 LoginForm，然后使用 EncodeLoginResponse 将 LoginResult 转换为 LoginResponse。转换函数的具体定义如下所示（这两个函数实际也可以在 Transport 层，为了讲解思路流畅，才放在本处）：</p>
<pre class="lang-java" data-nodeid="73952"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">DecodeLoginRequest</span><span class="hljs-params">(ctx context.Context, r interface{})</span> <span class="hljs-params">(interface{}, error)</span> </span>{
  req := r.(*pb.LoginRequest)
  <span class="hljs-keyword">return</span> LoginForm{
    Username: &nbsp; &nbsp; &nbsp;req.Username,
    Password: &nbsp; &nbsp; &nbsp;req.Password,
  }, nil
}
​
​
<span class="hljs-function">func <span class="hljs-title">EncodeLoginResponse</span><span class="hljs-params">(_ context.Context, r interface{})</span> <span class="hljs-params">(interface{}, error)</span> </span>{
  resp := r.(LoginResult)
 &nbsp;retStr := <span class="hljs-string">"fail"</span>
  <span class="hljs-keyword">if</span> resp.Ret {
    retStr = <span class="hljs-string">"success"</span>
  }
 &nbsp;errStr := <span class="hljs-string">""</span>
 &nbsp;<span class="hljs-keyword">if</span> resp.Err != nil {
 &nbsp; &nbsp;errStr = resp.Err.Error()
  }
  <span class="hljs-keyword">return</span> &amp;pb.LoginResponse{
    Ret: retStr,
    Err: errStr,
  }, nil
}
</code></pre>




<p data-nodeid="74503">这样做的好处有两点：一是 LoginRequest 和 LoginResponse 是通过 gRPC 生成的，属于 Transport 层，Endpoint 层不需要感知，后续如果技术选型变化了，需要将 gRPC 替换成 Thrift 时就可以只处理 Transport 层的变化，让变更最小化（如下图）；二是后端业务处理时的属性类型和返回给前端的数据属性类型不一定完全一样，比如上述代码中 LoginResult 中的 Ret 是 bool 类型，而返回给前端的 LoginResponse 中 Ret 是 string 类型，从而实现兼容性。</p>
<p data-nodeid="75076"><img src="https://s0.lgstatic.com/i/image/M00/47/9A/Ciqc1F9InySAYF-DAAAPcyoNePo946.png" alt="Drawing 1.png" data-nodeid="75080"></p>
<div data-nodeid="75077" class=""><p style="text-align:center">Go-kit 分层示意图</p></div>
<p></p>





<p data-nodeid="65615">如上图所示，Service 在最内层，Endpoint 在中间，Transport在最外侧，所以 Transport 是最容易进行变更的一层，越往内层逻辑应该越贴近领域逻辑。</p>
<h4 data-nodeid="76751" class="">3. 定义 Middleware，提供限流和日志中间件</h4>






<p data-nodeid="65619">如上文所说，Endpoint 层可以添加诸如日志、限流、熔断、链路追踪和服务监控等能力，这里我们就以限流为例，讲述如何为 Endpoint 添加额外能力。</p>
<pre class="lang-java" data-nodeid="77820"><code data-language="java"><span class="hljs-keyword">var</span> ErrLimitExceed = errors.New(<span class="hljs-string">"Rate limit exceed!"</span>)
​
<span class="hljs-comment">// 使用x/time/rate创建限流中间件</span>
<span class="hljs-function">func <span class="hljs-title">NewTokenBucketLimitterWithBuildIn</span><span class="hljs-params">(bkt *rate.Limiter)</span> endpoint.Middleware </span>{
  <span class="hljs-keyword">return</span> func(next endpoint.Endpoint) endpoint.Endpoint {
    <span class="hljs-keyword">return</span> func(ctx context.Context, request <span class="hljs-class"><span class="hljs-keyword">interface</span></span>{}) (response <span class="hljs-class"><span class="hljs-keyword">interface</span></span>{}, err error) {
 &nbsp; &nbsp; &nbsp;<span class="hljs-comment">// 如果超过流量，就直接返回限流异常</span>
      <span class="hljs-keyword">if</span> !bkt.Allow() {
        <span class="hljs-keyword">return</span> nil, ErrLimitExceed
      }
      <span class="hljs-keyword">return</span> next(ctx, request)
    }
  }
}
​
<span class="hljs-comment">// 使用时的代码实例</span>
ratebucket := rate.NewLimiter(rate.Every(time.Second * <span class="hljs-number">100</span>), <span class="hljs-number">1000</span>)
endpoint = user.NewTokenBucketLimitterWithBuildIn(ratebucket)(endpoint)
</code></pre>




<p data-nodeid="65621">上述代码使用了 x/time/rate 来进行限流，具体则使用了<strong data-nodeid="65713">令牌桶限流策略</strong>，其中 NewLimiter 函数会生成 Limiter 限流器，有两个参数，一个表示每秒生成多少令牌，另一个表示允许缓存多少令牌。</p>
<p data-nodeid="65622">当请求通过 Endpoint 时，就会被该 Middleware 拦截，然后调用 Limiter 的 Allow 函数，如果当前还存有令牌，则消耗一枚令牌，放行请求，返回 true；如果不存在，则阻拦请求，返回 false。有关令牌桶的限流策略，如果你感兴趣的话，可以自行搜索学习。</p>
<p data-nodeid="65623">除了限流外，Endpoint 的 Middleware 还可以和 Hystrix 结合提供熔断能力，和 ZipkinTracer 结合提供服务链路追踪能力、自定义接口调用统计指标或打印日志。</p>
<p data-nodeid="65624">由此可以看出 Endpoint 的 Middleware 确实是 Go-kit 的核心，众多服务治理相关中间件的集成都使用该层进行封装，提供统一的类似于切面的能力供开发者使用，免去了开发者处理架构集成的烦恼。</p>
<h4 data-nodeid="79422" class="">4. 定义 Transport，提供网络传输能力</h4>






<p data-nodeid="65628">下面，我们来具体看一下 Transport 层的实现。我们要给出 proto 文件中 UserServiceServer 的具体实现，也就是下述代码中的 grpcServer 结构体，它实现了 CheckPassword 方法，其中调用了其成员变量 checkPassword 的 ServeGRPC 方法。</p>
<pre class="lang-java" data-nodeid="79680"><code data-language="java">​
type grpcServer struct {
  checkPassword grpc.Handler
}
​
func (s *grpcServer) CheckPassword(ctx context.Context, r *pb.LoginRequest) (*pb.LoginResponse, error) {
  _, resp, err := s.checkPassword.ServeGRPC(ctx, r)
  <span class="hljs-keyword">if</span> err != nil {
    <span class="hljs-keyword">return</span> nil, err
  }
  <span class="hljs-keyword">return</span> resp.(*pb.LoginResponse), nil
}
​
​
<span class="hljs-function">func <span class="hljs-title">NewUserServer</span><span class="hljs-params">(ctx context.Context, endpoints Endpoints)</span> pb.UserServiceServer </span>{
  <span class="hljs-keyword">return</span> &amp;grpcServer{
    checkPassword: grpc.NewServer(
      endpoints.UserEndpoint,
      DecodeLoginRequest,
      EncodeLoginResponse,
    ),
  }
}
</code></pre>

<p data-nodeid="65630">checkPassword 的类型是 grpc.Handler，在 NewUserServer 方法中我们可以看到调用 grpc.NewServer 将其创建出来，需要传入 Endpoint 和编解码函数。这也正对应上文原理解析中 Go-kit 过程调用示意图的内容。</p>
<h4 data-nodeid="80968" class="">5. 启动服务端，注册RPC服务</h4>





<p data-nodeid="65634">我们来将上述部分组合起来，建立真正的网络服务端，并注册对应的 RPC 服务。具体代码如下所示：</p>
<pre class="lang-java" data-nodeid="81216"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
  flag.Parse()
​
  ctx := context.Background()
  <span class="hljs-comment">// 建立 service</span>
  <span class="hljs-keyword">var</span> svc user.UserService
  svc = user.UserServiceImpl{}
  <span class="hljs-comment">// 建立 endpoint</span>
  endpoint := user.MakeUserEndpoint(svc)
 &nbsp;<span class="hljs-comment">// 构造限流中间件</span>
  ratebucket := rate.NewLimiter(rate.Every(time.Second * <span class="hljs-number">1</span>), <span class="hljs-number">100</span>)
  endpoint = user.NewTokenBucketLimitterWithBuildIn(ratebucket)(endpoint)
 &nbsp;
  endpts := user.Endpoints{
    UserEndpoint: &nbsp; &nbsp; &nbsp;endpoint,
  }
  <span class="hljs-comment">// 使用 transport 构造 UserServiceServer</span>
  handler := user.NewUserServer(ctx, endpts)
  <span class="hljs-comment">// 监听端口，建立 gRPC 网络服务器，注册 RPC 服务</span>
  ls, _ := net.Listen(<span class="hljs-string">"tcp"</span>, <span class="hljs-string">"127.0.0.1:8080"</span>)
  gRPCServer := grpc.NewServer()
  pb.RegisterUserServiceServer(gRPCServer, handler)
  gRPCServer.Serve(ls)
​
}
​
</code></pre>

<p data-nodeid="65636">在 main 函数中，首先创建 UserService；然后调用 MakeUserEndpoint 函数创建 Endpoint，并使用限流中间件封装；接着调用 user 的 NewUserServer 方法，传入对应的 Endpoint，得到对应的 gRPC 处理器（Handler）；最后监听网络端口，创建 gRPC 服务端，并注册对应的处理器，即可启动 gRPC 服务端。</p>
<p data-nodeid="65637">至此，我们就构造了一个基于 gRPC 来提供 RPC 通信能力的 Go-kit 项目。</p>
<h3 data-nodeid="65638" class="">小结</h3>
<p data-nodeid="65639">相信你学完本课时后，会发现 Go-kit 可以和 gRPC 框架完美无缝结合，不仅能获得 gRPC 的高性能，还可以获得 Go-kit 作为微服务框架提供的构建微服务项目实例的便捷性，比如使用 Endpoint 的 Middleware 层来获得日志、限流、熔断、链路追踪和服务监控等方面的扩展能力，可以说是一举两得。</p>
<p data-nodeid="65640">最后给你留个思考题：如果要将 gRPC 替换为 Thrift，需要对上述代码做哪些改动呢？欢迎你在留言区积极发言和讨论。</p>

---

### 精选评论

##### **龙：
> 老师好，能发下代码地址么，谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 详见 https://github.com/longjoy/micro-go-course/tree/master/section18

##### *子：
> 请问课程有把项目源码打包到github上吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 详见 https://github.com/longjoy/micro-go-course/tree/master/section18

