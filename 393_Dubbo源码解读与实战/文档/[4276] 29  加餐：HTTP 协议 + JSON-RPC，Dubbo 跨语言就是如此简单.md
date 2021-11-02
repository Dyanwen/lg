<p data-nodeid="5023">在前面课时介绍 Protocol 和 Invoker 实现时，我们重点介绍了 AbstractProtocol 以及 DubboInvoker 实现。其实，Protocol 还有一个实现分支是 AbstractProxyProtocol，如下图所示：</p>
<p data-nodeid="5359"><img src="https://s0.lgstatic.com/i/image/M00/67/57/CgqCHl-hFBOAU2UWAAFM9gWuXAk914.png" alt="Lark20201103-162545.png" data-nodeid="5363"></p>
<div data-nodeid="5360" class=""><p style="text-align:center">AbstractProxyProtocol 继承关系图</p></div>





<p data-nodeid="3828">从图中我们可以看到：gRPC、HTTP、WebService、Hessian、Thrift 等协议对应的 Protocol 实现，都是继承自 AbstractProxyProtocol 抽象类。</p>
<p data-nodeid="3829">目前互联网的技术栈百花齐放，很多公司会使用 Node.js、Python、Rails、Go 等语言来开发 一些 Web 端应用，同时又有很多服务会使用 Java 技术栈实现，这就出现了大量的跨语言调用的需求。Dubbo 作为一个 RPC 框架，自然也希望能实现这种跨语言的调用，目前 Dubbo 中使用“HTTP 协议 + JSON-RPC”的方式来达到这一目的，其中 HTTP 协议和 JSON 都是天然跨语言的标准，在各种语言中都有成熟的类库。</p>
<p data-nodeid="3830">下面我们就重点来分析 Dubbo 对 HTTP 协议的支持。首先，我会介绍 JSON-RPC 的基础，并通过一个示例，帮助你快速入门，然后介绍 Dubbo 中 HttpProtocol 的具体实现，也就是如何将 HTTP 协议与 JSON-RPC 结合使用，实现跨语言调用的效果。</p>
<h3 data-nodeid="3831">JSON-RPC</h3>
<p data-nodeid="3832">Dubbo 中支持的 HTTP 协议实际上使用的是 JSON-RPC 协议。</p>
<p data-nodeid="3833"><strong data-nodeid="3930">JSON-RPC 是基于 JSON 的跨语言远程调用协议</strong>。Dubbo 中的 dubbo-rpc-xml、dubbo-rpc-webservice 等模块支持的 XML-RPC、WebService 等协议与 JSON-RPC 一样，都是基于文本的协议，只不过 JSON 的格式比 XML、WebService 等格式更加简洁、紧凑。与 Dubbo 协议、Hessian 协议等二进制协议相比，JSON-RPC 更便于调试和实现，可见 JSON-RPC 协议还是一款非常优秀的远程调用协议。</p>
<p data-nodeid="3834">在 Java 体系中，有很多成熟的 JSON-RPC 框架，例如 jsonrpc4j、jpoxy 等，其中，jsonrpc4j 本身体积小巧，使用方便，既可以独立使用，也可以与 Spring 无缝集合，非常适合基于 Spring 的项目。</p>
<p data-nodeid="3835">下面我们先来看看 JSON-RPC 协议中请求的基本格式：</p>
<pre class="lang-java" data-nodeid="3836"><code data-language="java">{
&nbsp; &nbsp; <span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>，
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"method"</span>:<span class="hljs-string">"sayHello"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"params"</span>:[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"Dubbo json-rpc"</span>
&nbsp;&nbsp;&nbsp;&nbsp;]
}
</code></pre>
<p data-nodeid="3837">JSON-RPC<strong data-nodeid="3938">请求</strong>中各个字段的含义如下：</p>
<ul data-nodeid="3838">
<li data-nodeid="3839">
<p data-nodeid="3840">id 字段，用于唯一标识一次远程调用。</p>
</li>
<li data-nodeid="3841">
<p data-nodeid="3842">method 字段，指定了调用的方法名。</p>
</li>
<li data-nodeid="3843">
<p data-nodeid="3844">params 数组，表示方法传入的参数，如果方法无参数传入，则传入空数组。</p>
</li>
</ul>
<p data-nodeid="3845">在 JSON-RPC 的服务端收到调用请求之后，会查找到相应的方法并进行调用，然后将方法的返回值整理成如下格式，返回给客户端：</p>
<pre class="lang-java" data-nodeid="3846"><code data-language="java">{
&nbsp; &nbsp; <span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>，
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"result"</span>:<span class="hljs-string">"Hello&nbsp;Dubbo json-rpc"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-string">"error"</span>:<span class="hljs-keyword">null</span>
}
</code></pre>
<p data-nodeid="3847">JSON-RPC<strong data-nodeid="3948">响应</strong>中各个字段的含义如下：</p>
<ul data-nodeid="3848">
<li data-nodeid="3849">
<p data-nodeid="3850">id 字段，用于唯一标识一次远程调用，该值与请求中的 id 字段值保持一致。</p>
</li>
<li data-nodeid="3851">
<p data-nodeid="3852">result 字段，记录了方法的返回值，若无返回值，则返回空；若调用错误，返回 null。</p>
</li>
<li data-nodeid="3853">
<p data-nodeid="3854">error 字段，表示调用发生异常时的异常信息，方法执行无异常时该字段为 null。</p>
</li>
</ul>
<h3 data-nodeid="3855">jsonrpc4j 基础使用</h3>
<p data-nodeid="3856">Dubbo 使用 jsonrpc4j 库来实现 JSON-RPC 协议，下面我们使用 jsonrpc4j 编写一个简单的 JSON-RPC 服务端示例程序和客户端示例程序，并通过这两个示例程序说明 jsonrpc4j 最基本的使用方式。</p>
<p data-nodeid="3857">首先，我们需要创建服务端和客户端都需要的 domain 类以及服务接口。我们先来创建一个 User 类，作为最基础的数据对象：</p>
<pre class="lang-java" data-nodeid="3858"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> userId;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> String name;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> age;
    <span class="hljs-comment">// 省略上述字段的getter/setter方法以及toString()方法</span>
}
</code></pre>
<p data-nodeid="3859">接下来创建一个 UserService 接口作为服务接口，其中定义了 5 个方法，分别用来创建 User、查询 User 以及相关信息、删除 User：</p>
<pre class="lang-java" data-nodeid="3860"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserService</span> </span>{
&nbsp; &nbsp; <span class="hljs-function">User <span class="hljs-title">createUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId, String name, <span class="hljs-keyword">int</span> age)</span></span>; 
&nbsp; &nbsp; <span class="hljs-function">User <span class="hljs-title">getUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span></span>;
&nbsp; &nbsp; <span class="hljs-function">String <span class="hljs-title">getUserName</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">getUserId</span><span class="hljs-params">(String name)</span></span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="3861">UserServiceImpl 是 UserService 接口的实现类，其中使用一个 ArrayList 集合管理 User 对象，具体实现如下：</p>
<pre class="lang-java" data-nodeid="3862"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserServiceImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserService</span> </span>{
    <span class="hljs-comment">// 管理所有User对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">private</span> List&lt;User&gt; users = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;(); 
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">createUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId, String name, <span class="hljs-keyword">int</span> age)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"createUser method"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; User user = <span class="hljs-keyword">new</span> User();
&nbsp; &nbsp; &nbsp; &nbsp; user.setUserId(userId);
&nbsp; &nbsp; &nbsp; &nbsp; user.setName(name);
&nbsp; &nbsp; &nbsp; &nbsp; user.setAge(age);
&nbsp; &nbsp; &nbsp; &nbsp; users.add(user); <span class="hljs-comment">// 创建User对象并添加到users集合中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> user;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">getUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"getUser method"</span>);
        <span class="hljs-comment">// 根据userId从users集合中查询对应的User对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> users.stream().filter(u -&gt; u.getUserId() == userId).findAny().get();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUserName</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"getUserName method"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据userId从users集合中查询对应的User对象之后，获取该User的name</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> getUser(userId).getName();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getUserId</span><span class="hljs-params">(String name)</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"getUserId method"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 根据name从users集合中查询对应的User对象，然后获取该User的id</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> users.stream().filter(u -&gt; u.getName().equals(name)).findAny().get().getUserId();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(<span class="hljs-string">"deleteAll"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; users.clear(); <span class="hljs-comment">// 清空users集合</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3863">整个用户管理业务的核心大致如此。下面我们来看服务端如何将 UserService 与 JSON-RPC 关联起来。</p>
<p data-nodeid="3864">首先，我们创建 RpcServlet 类，它是 HttpServlet 的子类，并覆盖了 HttpServlet 的 service() 方法。我们知道，HttpServlet 在收到 GET 和 POST 请求的时候，最终会调用其 service() 方法进行处理；HttpServlet 还会将 HTTP 请求和响应封装成 HttpServletRequest 和 HttpServletResponse 传入 service() 方法之中。这里的 RpcServlet 实现之中会创建一个 JsonRpcServer，并在 service() 方法中将 HTTP 请求委托给 JsonRpcServer 进行处理：</p>
<pre class="lang-java" data-nodeid="3865"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RpcServlet</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">HttpServlet</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> JsonRpcServer rpcServer = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">RpcServlet</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>();
&nbsp; &nbsp; &nbsp; &nbsp;&nbsp;<span class="hljs-comment">// JsonRpcServer会按照json-rpc请求，调用UserServiceImpl中的方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; rpcServer = <span class="hljs-keyword">new</span> JsonRpcServer(<span class="hljs-keyword">new</span> UserServiceImpl(), UserService.class);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">service</span><span class="hljs-params">(HttpServletRequest request,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;HttpServletResponse response)</span> <span class="hljs-keyword">throws</span> ServletException, IOException </span>{
&nbsp; &nbsp; &nbsp; &nbsp; rpcServer.handle(request, response);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3866">最后，我们创建一个 JsonRpcServer 作为服务端的入口类，在其 main() 方法中会启动 Jetty 作为 Web 容器，具体实现如下：</p>
<pre class="lang-java" data-nodeid="3867"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JsonRpcServer</span> </span>{
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 服务器的监听端口</span>
&nbsp; &nbsp; &nbsp; &nbsp; Server server = <span class="hljs-keyword">new</span> Server(<span class="hljs-number">9999</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 关联一个已经存在的上下文</span>
&nbsp; &nbsp; &nbsp; &nbsp; WebAppContext context = <span class="hljs-keyword">new</span> WebAppContext();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 设置描述符位置</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.setDescriptor(<span class="hljs-string">"/dubbo-demo/json-rpc-demo/src/main/webapp/WEB-INF/web.xml"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 设置Web内容上下文路径</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.setResourceBase(<span class="hljs-string">"/dubbo-demo/json-rpc-demo/src/main/webapp"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 设置上下文路径</span>
&nbsp; &nbsp; &nbsp; &nbsp; context.setContextPath(<span class="hljs-string">"/"</span>);
&nbsp; &nbsp; &nbsp; &nbsp; context.setParentLoaderPriority(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; &nbsp; &nbsp; server.setHandler(context);
&nbsp; &nbsp; &nbsp; &nbsp; server.start();
&nbsp; &nbsp; &nbsp; &nbsp; server.join();
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3868">这里使用到的 web.xml 配置文件如下：</p>
<pre class="lang-java" data-nodeid="3869"><code data-language="java">&lt;?xml version="1.0" encoding="UTF-8"?&gt;
&lt;web-app
&nbsp; &nbsp; &nbsp; &nbsp; xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
&nbsp; &nbsp; &nbsp; &nbsp; xmlns="http://xmlns.jcp.org/xml/ns/javaee"
&nbsp; &nbsp; &nbsp; &nbsp; xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
&nbsp; &nbsp; &nbsp; &nbsp; version="3.1"&gt;
&nbsp; &nbsp; &lt;servlet&gt;
&nbsp; &nbsp; &nbsp; &nbsp; &lt;servlet-name&gt;RpcServlet&lt;/servlet-name&gt;
&nbsp; &nbsp; &nbsp; &nbsp; &lt;servlet-class&gt;com.demo.RpcServlet&lt;/servlet-class&gt;
&nbsp; &nbsp; &lt;/servlet&gt;
&nbsp; &nbsp; &lt;servlet-mapping&gt;
&nbsp; &nbsp; &nbsp; &nbsp; &lt;servlet-name&gt;RpcServlet&lt;/servlet-name&gt;
&nbsp; &nbsp; &nbsp; &nbsp; &lt;url-pattern&gt;/rpc&lt;/url-pattern&gt;
&nbsp; &nbsp; &lt;/servlet-mapping&gt;
&lt;/web-app&gt;
</code></pre>
<p data-nodeid="3870"><strong data-nodeid="3965">完成服务端的编写之后，下面我们再继续编写 JSON-RPC 的客户端</strong>。在 JsonRpcClient 中会创建 JsonRpcHttpClient，并通过 JsonRpcHttpClient 请求服务端：</p>
<pre class="lang-java" data-nodeid="3871"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JsonRpcClient</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> JsonRpcHttpClient rpcHttpClient;
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
        <span class="hljs-comment">// 创建JsonRpcHttpClient</span>
&nbsp; &nbsp; &nbsp; &nbsp; rpcHttpClient = <span class="hljs-keyword">new</span> JsonRpcHttpClient(<span class="hljs-keyword">new</span> URL(<span class="hljs-string">"http://127.0.0.1:9999/rpc"</span>));
&nbsp; &nbsp; &nbsp; &nbsp; JsonRpcClient jsonRpcClient = <span class="hljs-keyword">new</span> JsonRpcClient();
&nbsp; &nbsp; &nbsp; &nbsp; jsonRpcClient.deleteAll(); <span class="hljs-comment">// 调用deleteAll()方法删除全部User</span>
        <span class="hljs-comment">// 调用createUser()方法创建User</span>
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(jsonRpcClient.createUser(<span class="hljs-number">1</span>, <span class="hljs-string">"testName"</span>, <span class="hljs-number">30</span>));
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">//&nbsp;调用getUser()、getUserName()、getUserId()方法进行查询</span>
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(jsonRpcClient.getUser(<span class="hljs-number">1</span>));
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(jsonRpcClient.getUserName(<span class="hljs-number">1</span>));
&nbsp; &nbsp; &nbsp; &nbsp; System.out.println(jsonRpcClient.getUserId(<span class="hljs-string">"testName"</span>));
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
        <span class="hljs-comment">// 调用服务端的deleteAll()方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; rpcHttpClient.invoke(<span class="hljs-string">"deleteAll"</span>, <span class="hljs-keyword">null</span>); 
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">createUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId, String name, <span class="hljs-keyword">int</span> age)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Object[] params = <span class="hljs-keyword">new</span> Object[]{userId, name, age};
        <span class="hljs-comment">// 调用服务端的createUser()方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> rpcHttpClient.invoke(<span class="hljs-string">"createUser"</span>, params, User.class);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">getUser</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Integer[] params = <span class="hljs-keyword">new</span> Integer[]{userId};
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用服务端的getUser()方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> rpcHttpClient.invoke(<span class="hljs-string">"getUser"</span>, params, User.class);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUserName</span><span class="hljs-params">(<span class="hljs-keyword">int</span> userId)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; Integer[] params = <span class="hljs-keyword">new</span> Integer[]{userId};
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用服务端的getUserName()方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> rpcHttpClient.invoke(<span class="hljs-string">"getUserName"</span>, params, String.class);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">getUserId</span><span class="hljs-params">(String name)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; String[] params = <span class="hljs-keyword">new</span> String[]{name};
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 调用服务端的getUserId()方法</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> rpcHttpClient.invoke(<span class="hljs-string">"getUserId"</span>, params, Integer.class);
&nbsp; &nbsp; }
}
<span class="hljs-comment">// 输出：</span>
<span class="hljs-comment">// User{userId=1, name='testName', age=30}</span>
<span class="hljs-comment">// User{userId=1, name='testName', age=30}</span>
<span class="hljs-comment">// testName</span>
<span class="hljs-comment">// 1</span>
</code></pre>
<h3 data-nodeid="5690">AbstractProxyProtocol</h3>


<p data-nodeid="3874">在 AbstractProxyProtocol 的 export() 方法中，首先会根据 URL 检查 exporterMap 缓存，如果查询失败，则会调用 ProxyFactory.getProxy() 方法将 Invoker 封装成业务接口的代理类，然后通过子类实现的 doExport() 方法启动底层的 ProxyProtocolServer，并初始化 serverMap 集合。具体实现如下：</p>
<pre class="lang-java" data-nodeid="3875"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">Exporter&lt;T&gt; <span class="hljs-title">export</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Invoker&lt;T&gt; invoker)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
    <span class="hljs-comment">// 首先查询exporterMap集合</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> String uri = serviceKey(invoker.getUrl());
&nbsp; &nbsp; Exporter&lt;T&gt; exporter = (Exporter&lt;T&gt;) exporterMap.get(uri);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (exporter != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (Objects.equals(exporter.getInvoker().getUrl(), invoker.getUrl())) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> exporter;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp;&nbsp;<span class="hljs-comment">// 通过ProxyFactory创建代理类，将Invoker封装成业务接口的代理类</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Runnable runnable = doExport(proxyFactory.getProxy(invoker, <span class="hljs-keyword">true</span>), invoker.getInterface(), invoker.getUrl());
&nbsp; &nbsp; <span class="hljs-comment">// doExport()方法返回的Runnable是一个回调，其中会销毁底层的Server，将会在unexport()方法中调用该Runnable</span>
&nbsp; &nbsp; exporter = <span class="hljs-keyword">new</span> AbstractExporter&lt;T&gt;(invoker) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">unexport</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>.unexport();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; exporterMap.remove(uri);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (runnable != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; runnable.run();
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; };
&nbsp; &nbsp; exporterMap.put(uri, exporter);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> exporter;
}
</code></pre>
<p data-nodeid="6654">在 HttpProtocol 的 doExport() 方法中，与前面介绍的 DubboProtocol 的实现类似，也要启动一个 RemotingServer。为了适配各种 HTTP 服务器，例如，Tomcat、Jetty 等，Dubbo 在 Transporter 层抽象出了一个 HttpServer 的接口。</p>
<p data-nodeid="6655" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/4C/Ciqc1F-hFD-ATNiiAABhOjw9PMM486.png" alt="Drawing 1.png" data-nodeid="6660"></p>
<div data-nodeid="6656"><p style="text-align:center">dubbo-remoting-http 模块位置</p></div>





<p data-nodeid="7299">dubbo-remoting-http 模块的入口是 HttpBinder 接口，它被 @SPI 注解修饰，是一个扩展接口，有三个扩展实现，默认使用的是 JettyHttpBinder 实现，如下图所示：</p>
<p data-nodeid="7629"><img src="https://s0.lgstatic.com/i/image/M00/67/4C/Ciqc1F-hFEaAUtnPAABBFL3GfzE890.png" alt="Drawing 2.png" data-nodeid="7633"></p>
<div data-nodeid="7630" class=""><p style="text-align:center">JettyHttpBinder 继承关系图</p></div>





<p data-nodeid="8584">HttpBinder 接口中的 bind() 方法被 @Adaptive 注解修饰，会根据 URL 的 server 参数选择相应的 HttpBinder 扩展实现，不同 HttpBinder 实现返回相应的 HttpServer 实现。HttpServer 的继承关系如下图所示：</p>
<p data-nodeid="8585" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/57/CgqCHl-hFFSAApv5AABUwFas2rw795.png" alt="Drawing 3.png" data-nodeid="8590"></p>
<div data-nodeid="8586"><p style="text-align:center">HttpServer 继承关系图</p></div>





<p data-nodeid="3885">这里我们以 JettyHttpServer 为例简单介绍 HttpServer 的实现，在 JettyHttpServer 中会初始化 Jetty Server，其中会配置 Jetty Server 使用到的线程池以及处理请求 Handler：</p>
<pre class="lang-java" data-nodeid="3886"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">JettyHttpServer</span><span class="hljs-params">(URL url, <span class="hljs-keyword">final</span> HttpHandler handler)</span> </span>{
    <span class="hljs-comment">// 初始化AbstractHttpServer中的url字段和handler字段</span>
&nbsp; &nbsp; <span class="hljs-keyword">super</span>(url, handler); 
&nbsp; &nbsp; <span class="hljs-keyword">this</span>.url = url;
    DispatcherServlet.addHttpHandler( <span class="hljs-comment">// 添加HttpHandler</span>
        url.getParameter(Constants.BIND_PORT_KEY, 
        url.getPort()), handler);
    <span class="hljs-comment">// 创建线程池</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> threads = url.getParameter(THREADS_KEY, DEFAULT_THREADS);
&nbsp; &nbsp; QueuedThreadPool threadPool = <span class="hljs-keyword">new</span> QueuedThreadPool();
&nbsp; &nbsp; threadPool.setDaemon(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; threadPool.setMaxThreads(threads);
&nbsp; &nbsp; threadPool.setMinThreads(threads);
    <span class="hljs-comment">// 创建Jetty Server</span>
&nbsp; &nbsp; server = <span class="hljs-keyword">new</span> Server(threadPool);
    <span class="hljs-comment">// 创建ServerConnector，并指定绑定的ip和port</span>
&nbsp; &nbsp; ServerConnector connector = <span class="hljs-keyword">new</span> ServerConnector(server);
&nbsp; &nbsp; String bindIp = url.getParameter(Constants.BIND_IP_KEY, url.getHost());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (!url.isAnyHost() &amp;&amp; NetUtils.isValidLocalHost(bindIp)) {
&nbsp; &nbsp; &nbsp; &nbsp; connector.setHost(bindIp);
&nbsp; &nbsp; }
&nbsp; &nbsp; connector.setPort(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()));
&nbsp; &nbsp; server.addConnector(connector);
    <span class="hljs-comment">// 创建ServletHandler并与Jetty Server关联，由DispatcherServlet处理全部的请求</span>
&nbsp; &nbsp; ServletHandler servletHandler = <span class="hljs-keyword">new</span> ServletHandler();
&nbsp; &nbsp; ServletHolder servletHolder = servletHandler.addServletWithMapping(DispatcherServlet.class, <span class="hljs-string">"/*"</span>);
&nbsp; &nbsp; servletHolder.setInitOrder(<span class="hljs-number">2</span>);
    <span class="hljs-comment">// 创建ServletContextHandler并与Jetty Server关联</span>
&nbsp; &nbsp; ServletContextHandler context = <span class="hljs-keyword">new</span> ServletContextHandler(server, <span class="hljs-string">"/"</span>, ServletContextHandler.SESSIONS);
&nbsp; &nbsp; context.setServletHandler(servletHandler);
&nbsp; &nbsp;ServletManager.getInstance().addServletContext(url.getParameter(Constants.BIND_PORT_KEY, url.getPort()), context.getServletContext());
&nbsp; &nbsp; server.start();
}
</code></pre>
<p data-nodeid="3887">我们可以看到 JettyHttpServer 收到的全部请求将委托给 DispatcherServlet 这个 HttpServlet 实现，而 DispatcherServlet 的 service() 方法会把请求委托给对应接端口的 HttpHandler 处理：</p>
<pre class="lang-java" data-nodeid="3888"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">service</span><span class="hljs-params">(HttpServletRequest request, HttpServletResponse response)</span> <span class="hljs-keyword">throws</span> ServletException, IOException </span>{
    <span class="hljs-comment">// 从HANDLERS集合中查询端口对应的HttpHandler对象</span>
&nbsp; &nbsp; HttpHandler handler = HANDLERS.get(request.getLocalPort());
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (handler == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 端口没有对应的HttpHandler实现</span>
&nbsp; &nbsp; &nbsp; &nbsp; response.sendError(HttpServletResponse.SC_NOT_FOUND, <span class="hljs-string">"Service not found."</span>);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> { <span class="hljs-comment">// 将请求委托给HttpHandler对象处理</span>
&nbsp; &nbsp; &nbsp; &nbsp; handler.handle(request, response);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3889">了解了 Dubbo 对 HttpServer 的抽象以及 JettyHttpServer 的核心之后，我们回到 HttpProtocol 中的 doExport() 方法继续分析。</p>
<p data-nodeid="9221">在 HttpProtocol.doExport() 方法中会通过 HttpBinder 创建前面介绍的 HttpServer 对象，并记录到 serverMap 中用来接收 HTTP 请求。这里初始化 HttpServer 以及处理请求用到的 HttpHandler 是 HttpProtocol 中的内部类，在其他使用 HTTP 协议作为基础的 RPC 协议实现中也有类似的 HttpHandler 实现类，如下图所示：</p>
<p data-nodeid="9547"><img src="https://s0.lgstatic.com/i/image/M00/67/57/CgqCHl-hFGCARUTkAABNZnY-dJg331.png" alt="Drawing 4.png" data-nodeid="9551"></p>
<div data-nodeid="9548" class=""><p style="text-align:center">HttpHandler 继承关系图</p></div>





<p data-nodeid="3893">在 HttpProtocol.InternalHandler 中的 handle() 实现中，会将请求委托给 skeletonMap 集合中记录的 JsonRpcServer 对象进行处理：</p>
<pre class="lang-java" data-nodeid="3894"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handle</span><span class="hljs-params">(HttpServletRequest request, HttpServletResponse response)</span> <span class="hljs-keyword">throws</span> ServletException </span>{
&nbsp; &nbsp; String uri = request.getRequestURI();
&nbsp; &nbsp; JsonRpcServer skeleton = skeletonMap.get(uri);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cors) { ... <span class="hljs-comment">// 处理跨域问题 }</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (request.getMethod().equalsIgnoreCase(<span class="hljs-string">"OPTIONS"</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; response.setStatus(<span class="hljs-number">200</span>); <span class="hljs-comment">// 处理OPTIONS请求</span>
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (request.getMethod().equalsIgnoreCase(<span class="hljs-string">"POST"</span>)) {
        <span class="hljs-comment">// 只处理POST请求</span>
        RpcContext.getContext().setRemoteAddress(
            request.getRemoteAddr(), request.getRemotePort());
  &nbsp; &nbsp; &nbsp; skeleton.handle(request.getInputStream(),   response.getOutputStream());
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {<span class="hljs-comment">// 其他Method类型的请求，例如，GET请求，直接返回500</span>
&nbsp; &nbsp; &nbsp; &nbsp; response.setStatus(<span class="hljs-number">500</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3895">skeletonMap 集合中的 JsonRpcServer 是与 HttpServer 对象一同在 doExport() 方法中初始化的。最后，我们来看 HttpProtocol.doExport() 方法的实现：</p>
<pre class="lang-java" data-nodeid="3896"><code data-language="java"><span class="hljs-keyword">protected</span> &lt;T&gt; <span class="hljs-function">Runnable <span class="hljs-title">doExport</span><span class="hljs-params">(<span class="hljs-keyword">final</span> T impl, Class&lt;T&gt; type, URL url)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; String addr = getAddr(url);
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;先查询serverMap缓存</span>
&nbsp; &nbsp; ProtocolServer protocolServer = serverMap.get(addr);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (protocolServer == <span class="hljs-keyword">null</span>) { <span class="hljs-comment">// 查询缓存失败</span>
        <span class="hljs-comment">// 创建HttpServer,注意，传入的HttpHandler实现是InternalHandler</span>
&nbsp; &nbsp; &nbsp; &nbsp; RemotingServer remotingServer = httpBinder.bind(url, <span class="hljs-keyword">new</span> InternalHandler(url.getParameter(<span class="hljs-string">"cors"</span>, <span class="hljs-keyword">false</span>)));
&nbsp; &nbsp; &nbsp; &nbsp; serverMap.put(addr, <span class="hljs-keyword">new</span> ProxyProtocolServer(remotingServer));
&nbsp; &nbsp; }
    <span class="hljs-comment">// 创建JsonRpcServer对象，并将URL与JsonRpcServer的映射关系记录到skeletonMap集合中</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> String path = url.getAbsolutePath();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> String genericPath = path + <span class="hljs-string">"/"</span> + GENERIC_KEY;
&nbsp; &nbsp; JsonRpcServer skeleton = <span class="hljs-keyword">new</span> JsonRpcServer(impl, type);
&nbsp; &nbsp; JsonRpcServer genericServer = <span class="hljs-keyword">new</span> JsonRpcServer(impl, GenericService.class);
&nbsp; &nbsp; skeletonMap.put(path, skeleton);
&nbsp; &nbsp; skeletonMap.put(genericPath, genericServer);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> () -&gt; {<span class="hljs-comment">// 返回Runnable回调，在Exporter中的unexport()方法中执行</span>
&nbsp; &nbsp; &nbsp; &nbsp; skeletonMap.remove(path);
&nbsp; &nbsp; &nbsp; &nbsp; skeletonMap.remove(genericPath);
&nbsp; &nbsp; };
}
</code></pre>
<p data-nodeid="3897">介绍完 HttpProtocol 暴露服务的相关实现之后，下面我们再来看 HttpProtocol 中引用服务相关的方法实现，即 protocolBindinRefer() 方法实现。该方法首先通过 doRefer() 方法创建业务接口的代理，这里会使用到 jsonrpc4j 库中的 JsonProxyFactoryBean 与 Spring 进行集成，在其 afterPropertiesSet() 方法中会创建 JsonRpcHttpClient 对象：</p>
<pre class="lang-java" data-nodeid="3898"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">afterPropertiesSet</span><span class="hljs-params">()</span> </span>{
    ... ... <span class="hljs-comment">// 省略ObjectMapper等对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
        <span class="hljs-comment">// 创建JsonRpcHttpClient，用于后续发送json-rpc请求</span>
&nbsp; &nbsp; &nbsp; &nbsp; jsonRpcHttpClient = <span class="hljs-keyword">new</span> JsonRpcHttpClient(objectMapper, <span class="hljs-keyword">new</span> URL(getServiceUrl()), extraHttpHeaders);
&nbsp; &nbsp; &nbsp; &nbsp; jsonRpcHttpClient.setRequestListener(requestListener);
&nbsp; &nbsp; &nbsp; &nbsp; jsonRpcHttpClient.setSslContext(sslContext);
&nbsp; &nbsp; &nbsp; &nbsp; jsonRpcHttpClient.setHostNameVerifier(hostNameVerifier);
&nbsp; &nbsp; } <span class="hljs-keyword">catch</span> (MalformedURLException mue) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RuntimeException(mue);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="3899">下面来看 doRefer() 方法的具体实现：</p>
<pre class="lang-java" data-nodeid="3900"><code data-language="java"><span class="hljs-keyword">protected</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">doRefer</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Class&lt;T&gt; serviceType, URL url)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> String generic = url.getParameter(GENERIC_KEY);
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">boolean</span> isGeneric = ProtocolUtils.isGeneric(generic) || serviceType.equals(GenericService.class);
&nbsp; &nbsp; JsonProxyFactoryBean jsonProxyFactoryBean = <span class="hljs-keyword">new</span> JsonProxyFactoryBean();
&nbsp; &nbsp; ... <span class="hljs-comment">// 省略其他初始化逻辑</span>
&nbsp; &nbsp; jsonProxyFactoryBean.afterPropertiesSet();
    <span class="hljs-comment">// 返回的是serviceType类型的代理对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> (T) jsonProxyFactoryBean.getObject(); 
}
</code></pre>
<p data-nodeid="3901">在 AbstractProxyProtocol.protocolBindingRefer() 方法中，会通过 ProxyFactory.getInvoker() 方法将 doRefer() 方法返回的代理对象转换成 Invoker 对象，并记录到 Invokers 集合中，具体实现如下：</p>
<pre class="lang-java" data-nodeid="3902"><code data-language="java"><span class="hljs-keyword">protected</span> &lt;T&gt; <span class="hljs-function">Invoker&lt;T&gt; <span class="hljs-title">protocolBindingRefer</span><span class="hljs-params">(<span class="hljs-keyword">final</span> Class&lt;T&gt; type, <span class="hljs-keyword">final</span> URL url)</span> <span class="hljs-keyword">throws</span> RpcException </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Invoker&lt;T&gt; target = proxyFactory.getInvoker(doRefer(type, url), type, url);
&nbsp; &nbsp; Invoker&lt;T&gt; invoker = <span class="hljs-keyword">new</span> AbstractInvoker&lt;T&gt;(type, url) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> Result <span class="hljs-title">doInvoke</span><span class="hljs-params">(Invocation invocation)</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Result result = target.invoke(invocation);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略处理异常的逻辑</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> result;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; };
&nbsp; &nbsp; invokers.add(invoker); <span class="hljs-comment">// 将Invoker添加到invokers集合中</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> invoker;
}
</code></pre>
<h3 data-nodeid="3903">总结</h3>
<p data-nodeid="3904">本课时重点介绍了在 Dubbo 中如何通过“HTTP 协议 + JSON-RPC”的方案实现跨语言调用。首先我们介绍了 JSON-RPC 中请求和响应的基本格式，以及其实现库 jsonrpc4j 的基本使用；接下来我们还详细介绍了 Dubbo 中 AbstractProxyProtocol、HttpProtocol 等核心类，剖析了 Dubbo 中“HTTP 协议 + JSON-RPC”方案的落地实现。</p>
<p data-nodeid="9868" class="">下一课时，我们将介绍 Dubbo 中 Filter 接口相关的实现。</p>

---

### 精选评论

##### **豹：
> 老师好，除了Dubbo 协议，是不是其它的协议都不需要考虑编解码、序列化、线程模型等？Dubbo 2.6.x 版本非Dubbo 协议使用的是Spring 的东西，感觉新版本改动有点大。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1、还是需要考虑这几方面的，毕竟要了解编码、序列化、线程模型，才能更好的处理问题。以Http+JSON-RPC这个为例，其实我们也是理解了它的序列化方式就是JSON方式。
2、Dubbo 2.6.x 用的是 Spring HttpInvoker，和 2.7 版本完全不是一个东西。给一篇参考文章吧，可以简单了解一下废弃2.6.x 方案的背景[https://lexburner.github.io/dubbo-http-protocol/]

