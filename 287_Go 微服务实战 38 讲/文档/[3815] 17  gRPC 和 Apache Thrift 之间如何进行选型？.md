<p data-nodeid="797" class="">在上一课时中，我们已经对 Go 语言原生 RPC 的使用和具体实现原理进行了详细讲解，并指出其缺少超时熔断、链接管理和服务注册发现等功能，达不到生产环境“开箱即用”的水准，不过官方已经不再为其扩充新功能了，而是推荐使用 gRPC。</p>
<p data-nodeid="798">其实，除了 gRPC 外，Facebook 开源的 Thrift 框架也是业界较为流行的 RPC 方案，比如 HBase 就是使用它来提供 API 支持的。</p>
<p data-nodeid="799">本课时我们将会首先简要介绍下 gRPC 的特性和使用案例，然后再介绍 Thrift，最后再对比这二者之间的异同点，给出你选择的依据。</p>
<h3 data-nodeid="800">gRPC 简介和使用</h3>
<p data-nodeid="801">gRPC 是由 Google 开源的高性能 RPC 框架。自 2015 年发布以来，gRPC 日益成熟，并成为跨语言 RPC 通信中最流行也最受欢迎的选择之一。gRPC 拥有很多特性，其中最引人注目的有以下几个方面：</p>
<ul data-nodeid="993">
<li data-nodeid="994">
<p data-nodeid="995"><strong data-nodeid="1012">内置流式 RPC 支持</strong>。这意味着你可以使用同一 RPC 框架来处理普通的 RPC 调用和分块进行的数据传输调用，这在很大程度上统一了网络相关的基础代码并简化了逻辑。</p>
</li>
<li data-nodeid="996">
<p data-nodeid="997"><strong data-nodeid="1017">内置拦截器的支持</strong>。gRPC 提供了一种向多个服务端点添加通用功能的强大方法，这使得你可以轻松使用拦截器对所有接口进行共享的运行状况检查和身份验证。</p>
</li>
<li data-nodeid="998">
<p data-nodeid="999"><strong data-nodeid="1022">内置流量控制和 TLS 支持</strong>。gRPC 是基于 HTTP/2 协议构建的，具有很多强大的特性，其中很多特性以前是必须在 Netty 上自行实现的。这使得客户端的实现更简单，并且可以轻松实现更多语言的绑定。</p>
</li>
<li data-nodeid="1000">
<p data-nodeid="1001"><strong data-nodeid="1027">基于 ProtoBuf 进行数据序列化</strong>。ProtoBuf 是由 Google 开源的数据序列化协议，用于将数据进行序列化，在数据存储和通信协议等方面有较大规模的应用和成熟案例。gRPC 直接使用成熟的 ProtoBuf 来定义服务、接口和数据类型，其序列化性能、稳定性和兼容性得到保障。</p>
</li>
<li data-nodeid="1002">
<p data-nodeid="1003"><strong data-nodeid="1032">底层基于 HTTP/2 标准设计</strong>。gRPC 正是基于 HTTP/2 才得以实现更多强大功能，如双向流、多复用请求和头部压缩等，从而可以节省带宽、降低 TCP 连接次数和提高 CPU 利用率等。同时，基于 HTTP/2 标准的 gRPC 还提高了云端服务和 Web 应用的性能，使得 gRPC 既能够在客户端应用，也能够在服务器端应用，从而实现客户端和服务器端的通信以及简化通信系统的构建。</p>
</li>
<li data-nodeid="1004">
<p data-nodeid="1005" class=""><strong data-nodeid="1037">优秀的社区支持</strong>。作为一个开源项目，gRPC 拥有良好的社区支持和维护，发展迅速，并且 gRPC 的文档也很丰富，这些对用户都很有帮助。</p>
</li>
<li data-nodeid="1006">
<p data-nodeid="1007"><strong data-nodeid="1042">提供多种语言支持</strong>。gRPC 支持多种语言，如 C、C++、Go 、Python、Ruby、Java 、PHP 、C# 和 Node.js 等，并且能够基于 ProtoBuf 定义自动生成相应的客户端和服务端代码。目前已提供了 Java 语言版本的 gRPC-Java 和 Go 语言版本的 gRPC-Go。</p>
</li>
</ul>

<p data-nodeid="817"><img src="https://s0.lgstatic.com/i/image/M00/46/C9/CgqCHl9GF-WAbunfAADemgWlVgo940.png" alt="grpc_language.png" data-nodeid="922"></p>
<div data-nodeid="818"><p style="text-align:center">gRPC 调用示意图</p></div>
<p data-nodeid="819">结合上面的 gRPC 调用示意图，我们可以看到，一个 C++ 语言的服务器可以通过 gRPC 分别与 Ruby 语言开发的桌面客户端和 Java 语言开发的 Android 客户端进行交互。</p>
<p data-nodeid="820">下面，我们来讲解一下 gRPC 的使用过程。gRPC 过程调用时，服务端和客户端需要依赖共同的 proto 文件。proto 文件可以定义远程调用的接口、方法名、参数和返回值等。通过 proto 文件可以自动生成相应的客户端和服务端 RPC 代码。借助这些代码，客户端可以十分方便地发送 RPC 请求，并且服务端也可以很简单地建立 RPC 服务器、处理 RPC 请求并且将返回值作为响应发送给客户端。</p>
<p data-nodeid="821">gRPC 使用一般分为三大步骤：①定义和编译 proto 文件；②客户端发送 RPC 请求；③服务端建立 RPC 服务。</p>
<h4 data-nodeid="822">1. 定义和编译 proto 文件</h4>
<p data-nodeid="823">首先，我们要定义一个 proto 文件，其具体语法可查看 <a href="https://www.google.com/search?q=https%3A%2F%2F+developers.google.com%2Fprotocol-buffers%2Fdocs%2Fproto3&amp;rlz=1C1GCEU_zh-CNHK904HK904&amp;oq=https%3A%2F%2F+developers.google.com%2Fprotocol-buffers%2Fdocs%2Fproto3&amp;aqs=chrome.0.69i59j69i58.447j0j4&amp;sourceid=chrome&amp;ie=UTF-8" data-nodeid="932">Protobuf3 语言指南</a>。在该文件中，我们定义了两个参数结果，分别是 LoginRequest 和 LoginResponse，同时还有一个服务结构 UserService，代码如下：</p>
<pre class="lang-java" data-nodeid="824"><code data-language="java">syntax = <span class="hljs-string">"proto3"</span>;
<span class="hljs-keyword">package</span> pb;

service UserService{
<span class="hljs-function">rpc <span class="hljs-title">CheckPassword</span><span class="hljs-params">(LoginRequest)</span> <span class="hljs-title">returns</span> <span class="hljs-params">(LoginResponse)</span> </span>{}
}

message LoginRequest {
string Username = <span class="hljs-number">1</span>;
string Password = <span class="hljs-number">2</span>;
}

message LoginResponse {
string Ret = <span class="hljs-number">1</span>;
string err = <span class="hljs-number">2</span>;
}
</code></pre>
<p data-nodeid="825">UserService 有一个 CheckPassword 方法，并定义了该方法对应的输入参数和返回值，这些值也都定义在 proto 文件中。</p>
<p data-nodeid="826">接下来我们使用 protoc 编译工具编译这个 proto 文件，生成服务端和客户端的代码，如下：</p>
<pre class="lang-js" data-nodeid="827"><code data-language="js">protoc --go_out=plugins=grpc:. pb/user.proto
</code></pre>
<p data-nodeid="828">使用 protoc 生成 Go 语言版本的客户端和服务端代码后，开发人员就可以在业务代码中直接调用这些 API，并在服务器端实现相应的接口。然后，运行 gRPC 服务端代码并将实现的服务进行注册，来处理客户端的调用。gRPC 框架会接收网络传入请求，解析请求数据，执行相应服务方法并将方法结果编码成响应通过网络传递给客户端。</p>
<p data-nodeid="829">客户端的本地定义方法，其方法名、参数和返回值与服务端定义的方法相同。客户端可以直接调用这些方法，将调用的参数设置到对应的参数消息类型中，gRPC 生成的客户端代码会将请求转换为网络消息发送到服务端，然后服务端解析请求并处理。</p>
<h4 data-nodeid="830">2. 客户端发送 RPC 请求</h4>
<p data-nodeid="831">它首先调用 grpc.Dial 建立网络连接，然后使用 protoc 编译生成的 pb.NewUserServiceClient 函数创建 gRPC 客户端，最后再调用客户端的 CheckPassword 函数进行 RPC 调用，代码如下所示：</p>
<pre class="lang-java" data-nodeid="832"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
serviceAddress := <span class="hljs-string">"127.0.0.1:1234"</span>
conn, err := grpc.Dial(serviceAddress, grpc.WithInsecure())
<span class="hljs-keyword">if</span> err != nil {
panic(<span class="hljs-string">"connect error"</span>)
}
defer conn.Close()

userClient := pb.NewUserServiceClient(conn)
userReq := &amp;pb.LoginRequest{Username: <span class="hljs-string">""</span>, Password: <span class="hljs-string">""</span>}
reply, _ := userClient.CheckPassword(context.Background(), userReq)
fmt.Printf(<span class="hljs-string">"UserService CheckPassword : %s"</span>, reply.Ret)

}
</code></pre>
<h4 data-nodeid="833">3. 服务端建立 RPC 服务</h4>
<p data-nodeid="834">它首先需要调用 grpc.NewServer() 来建立 RPC 的服务端，然后将 UserService 注册到 RPC 服务端上，UserService 中实现了 CheckPassword 方法，代码如下：</p>
<pre class="lang-java" data-nodeid="835"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{
flag.Parse()

lis, err := net.Listen(<span class="hljs-string">"tcp"</span>,<span class="hljs-string">"127.0.0.1:1234"</span>)
<span class="hljs-keyword">if</span> err != nil {
log.Fatalf(<span class="hljs-string">"failed to listen: %v"</span>, err)
}

grpcServer := grpc.NewServer()
userService := <span class="hljs-keyword">new</span>(user_service.UserService)
pb.RegisterUserServiceServer(grpcServer, userService)
grpcServer.Serve(lis)
}
</code></pre>
<p data-nodeid="836">最后我们再来看下 UserService 的具体代码实现：①定义 UserService 结构体，②实现 CheckPassword 方法。</p>
<pre class="lang-java" data-nodeid="837"><code data-language="java">type UserService struct{}

func (s * UserService) CheckPassword(ctx context.Context, req pb.LoginRequest) (pb.LoginResponse, error) {
<span class="hljs-keyword">if</span> req.Username == <span class="hljs-string">"admin"</span> &amp;&amp; req.Password == <span class="hljs-string">"admin"</span> {
response := pb.LoginResponse{Ret: <span class="hljs-string">"success"</span>}
<span class="hljs-keyword">return</span> &amp;response, nil
}

response := pb.LoginResponse{Ret: <span class="hljs-string">"fail"</span>}
<span class="hljs-keyword">return</span> &amp;response, nil
}
</code></pre>
<p data-nodeid="838">如上代码所示，UserService 的 CheckPassword 实现起来都很简单，CheckPassword 方法就是判断用户名和密码是否都是 admin，如果是则检查成功，否则即为失败。</p>
<h3 data-nodeid="839">Thrift 简介</h3>
<p data-nodeid="840">Thrift 是由 Facebook 开源的跨平台、支持多语言的成熟 RPC 框架，它通过定义中间语言（IDL） 自动生成 RPC 客户端与服务端通信代码，从而可以在 C++、Java、Python、PHP 和 Go 等多种编程语言间构建无缝结合的、高效的 RPC 通信服务。Thrift 通过中间语言来定义 RPC 的接口和数据类型，然后通过编译器生成不同语言的代码并由生成的代码负责 RPC 协议层和传输层的实现。</p>
<p data-nodeid="841">下面我们同样来看一下 Thrift 的具体使用方法。</p>
<h4 data-nodeid="842">1. 定义和编译 Thrift 文件</h4>
<p data-nodeid="843">不同于 gRPC 使用 Protobuf 的方法，Thrift 使用自己的中间语言 thrift 来定义接口，不过二者极为类似，比如下面的代码，也是定义了一个拥有检查用户名密码接口的 user 服务。</p>
<pre class="lang-java" data-nodeid="844"><code data-language="java">namespace go user

struct LoginRequest {
<span class="hljs-number">1</span>: string username;
<span class="hljs-number">2</span>: string password;
}

struct LoginResponse {
<span class="hljs-number">1</span>: string msg;
}

service User {
<span class="hljs-function">LoginResponse <span class="hljs-title">checkPassword</span><span class="hljs-params">(<span class="hljs-number">1</span>: LoginRequest req)</span></span>;
}
</code></pre>
<p data-nodeid="1829" class="">然后，我们使用 thrift 工具将上述定义编译，生成对应的 Go 代码。</p>


<pre class="lang-java" data-nodeid="846"><code data-language="java">thrift -r --gen go user.thrift
</code></pre>
<h4 data-nodeid="847">2. 客户端发送 RPC 请求</h4>
<p data-nodeid="848">接下来，我们使用 Thrift 相关的代码库来实现客户端，如下面的示例代码所示，和上面 gRPC 的代码对比起来，你可以明显发现 Thrift 需要配置的功能项更多、更复杂。至于 transport、protocol 等配置，我们后面讲解服务端代码实现后再具体介绍。</p>
<pre class="lang-java" data-nodeid="849"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span>  </span>{
tSocket, err := thrift.NewTSocket(net.JoinHostPort(HOST, PORT))
<span class="hljs-keyword">if</span> err != nil {
log.Fatalln(<span class="hljs-string">"tSocket error:"</span>, err)
}
transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
transport := transportFactory.GetTransport(tSocket)
protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()
client := user.NewUserClientFactory(transport, protocolFactory)

<span class="hljs-keyword">if</span> err := transport.Open(); err != nil {
log.Fatalln(<span class="hljs-string">"Error opening:"</span>, HOST + <span class="hljs-string">":"</span> + PORT)
}
defer transport.Close()

req := user.LoginRequest{Username:<span class="hljs-string">"admin"</span>, Password: <span class="hljs-string">"admin"</span>}
res, err := client.CheckPassword(&amp;req)
fmt.Println(res.Msg)
}

</code></pre>
<h4 data-nodeid="850">3. 服务端建立 RPC 服务</h4>
<p data-nodeid="851">与客户端类似，服务端建立 RPC 服务时需要选择跟客户端一致的网络传输和序列化协议配置，然后调用 NewTSimpleServer4 函数使用具体接口实现类（代码中的 UserService），再结合网络传输和序列化协议配置来共同建立 RPC 服务。</p>
<pre class="lang-java" data-nodeid="852"><code data-language="java"><span class="hljs-function">func <span class="hljs-title">main</span><span class="hljs-params">()</span> </span>{

handler := &amp;UserService{} <span class="hljs-comment">// 类似上边的gRPC，是一个实现生成代码中接口的函数的结构体</span>
processor := user.NewUserProcessor(handler)
serverTransport, err := thrift.NewTServerSocket(HOST + <span class="hljs-string">":"</span> + PORT)
<span class="hljs-keyword">if</span> err != nil {
log.Fatalln(<span class="hljs-string">"Error:"</span>, err)
}
transportFactory := thrift.NewTFramedTransportFactory(thrift.NewTTransportFactory())
protocolFactory := thrift.NewTBinaryProtocolFactoryDefault()

server := thrift.NewTSimpleServer4(processor, serverTransport, transportFactory, protocolFactory)
fmt.Println(<span class="hljs-string">"Running at:"</span>, HOST + <span class="hljs-string">":"</span> + PORT)
server.Serve()
}
</code></pre>
<p data-nodeid="853">如上述示例代码，Thrift 可以让用户选择客户端和服务端之间进行 RPC 网络传输和序列化协议，对于服务端，还提供了建立不同网络处理模型的服务端能力。</p>
<p data-nodeid="854">对于通信协议（TProtocol），Thrift 提供了基于文本和二进制传输协议，可选的协议有：二进制编码协议（TBinaryProtocol）、压缩的二进制编码协议（TCompactProtocol）、JSON 格式的编码协议（TJSONProtocol）和用于调试的可读编码协议（TDebugProtocol）。上面示例代码中我们使用的是默认的二进制协议，也就是 TBinaryProtocol。</p>
<p data-nodeid="855">对于传输方式（TTransport），Thrift 提供了丰富的传输方式，可选的传输方式有：最常见的阻塞式 I/O 的 TSocket、HTTP 协议传输的 THttpTransport、以 frame 为单位进行非阻塞传输的 TFramedTransport 和以内存进行传输的 TMemoryTransport 等。</p>
<p data-nodeid="3405" class="">对于服务端模型（TServer），Thrift 目前提供了：单线程服务器端使用标准的阻塞式 I/O 的 TServer、多线程服务器端使用标准的阻塞式 I/O 的 TThreadedServer 和多线程网络模型使用配有线程池的阻塞式 I/O 的 TThreadPoolServer 等。</p>




<p data-nodeid="857">整个 Thrift 框架中，可供用户选择和配置的项目如下图所示，由此可见，Thrift 具备丰富的配置项，可以为开发者提供尽可能多的选择。</p>
<p data-nodeid="858"><img src="https://s0.lgstatic.com/i/image/M00/46/BF/Ciqc1F9GGH6AYzO_AACRpMJ8K94764.png" alt="thrift_structure.png" data-nodeid="973"></p>
<div data-nodeid="859"><p style="text-align:center">Thrift 框架示意图</p></div>
<h3 data-nodeid="860">gRPC 和 Thrift 的区别和选择</h3>
<p data-nodeid="861">Thrift 是 RPC 领域的老牌开源项目，而 gRPC 后来者居上，逐渐超越了 Thrift，二者目前在社区欢迎度和使用度上的对比可以通过 StackShare 网站查看，截至 2020 年 7 月的数据如下图所示：</p>
<p data-nodeid="862"><img src="https://s0.lgstatic.com/i/image/M00/46/BF/Ciqc1F9GGIiAJCt0AAHADfLpvVw269.jpg" alt="gprc_vs_thrift.jpg" data-nodeid="982"></p>
<div data-nodeid="863"><p style="text-align:center">Thrift 和 gRPC在社区的欢迎度和使用度对比</p></div>
<p data-nodeid="864">可以看出，gRPC 拥有更加良好的生态环境和社区规模，而且更多的公司开始将自身技术栈迁移到 gRPC，比如 Dropbox。那为什么会出现这种“后来者反而居上”的情况呢？</p>
<ul data-nodeid="865">
<li data-nodeid="866">
<p data-nodeid="867">首先，在工程性方面，gRPC 比 Thrift 拥有更加良好的文档并且代码更容易上手，gRPC 编译生成的代码量远小于 Thrift 生成的代码，这些优势相信你在实践上述案例时就能发现。</p>
</li>
<li data-nodeid="868">
<p data-nodeid="869">其次，在功能方面，Thrift 不支持流式编程，不支持大批量流式读写数据的能力，这对很多大数据项目至关重要，比如开源的分布式内存文件系统 Alluxio 就因此从 Thrift 迁移到 gRPC。</p>
</li>
<li data-nodeid="870">
<p data-nodeid="871">最后，在性能方面，gRPC 已经从刚开始的被吊打，逐渐缩小与 Thrift 之间的差距。目前根据 GitHub 上他人压测的效果，gRPC 和 Thrift 已经不存在数量级上的性能差距，而且 gRPC 可以使用流式 stream 能力来提升性能。可以说，二者都不会成为你链路性能优化上的瓶颈。</p>
</li>
</ul>
<p data-nodeid="872">综上所述，从成熟度上来讲，因为 Thrift 的起源要早于 gRPC，所以目前使用的范围要大于 gRPC，在 HBase、Hadoop 和 Cassandra 等许多开源组件中都得到了广泛应用，而且 Thrift 支持的语言要比 gRPC 更多，多达 25 种语言，所以如果遇到 gRPC 不支持的语言场景，我建议你选择 Thrift。</p>
<p data-nodeid="873">但 gRPC 作为 Google 开源的后起之秀，因为采用了 HTTP/2 作为通信协议、ProtoBuf 作为数据序列化格式和支持流式处理，在移动端设备应用等对传输带宽比较敏感的场景下具有很大的优势，而且开发文档和代码示例丰富，社区活跃度高，根据 ProtoBuf 文件生成的代码要比 Thrift 更加简洁，更容易上手，所以如果是 gRPC 支持开发语言的场景，我建议你还是采用 gRPC 比较好。</p>
<h3 data-nodeid="874">小结</h3>
<p data-nodeid="875">在本课时我们先分别介绍了 gRPC 和 Thrift 的整体概念和示例，然后又对二者进行了分析和对比，这都可以为你以后对 RPC 框架进行选型提供依据。后续我们会讲解如何将 gRPC 集成到微服务架构中。</p>
<p data-nodeid="876">但是对比二者，你会发现它们也都缺少了大量的功能，比如：连接池、服务框架、服务发现、服务治理、分布式链路追踪、埋点和上下文日志等，而这些功能才是日常开发和运维最常使用的。不过本课程的后面我们都会一一讲述上述功能，为你拼接出最为详细和完整的 Go 微服务架构全貌。</p>
<p data-nodeid="877" class="">关于本课时，你有什么经验或想法呢？欢迎你在留言区和我分享。</p>

---

### 精选评论

##### JSON：
> 老师，问个问题，目前遇到这样一个问题：go写的grpc服务(无界面的)部署到线上后，测试没法做回归测试，目前针对这快，有什么好的解决方案吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 业界可以使用jenkins执行对应的测试脚本来进行定期回归测试

