<p data-nodeid="1265" class="">从这一讲开始，我将带你学习本专栏的第五模块，在这个模块中，你将学到我们项目中最常用的编码操作，也就是编写 RESTful API 和 RPC 服务。在实际开发项目中，你编写的这些服务可以被其他服务使用，这样就组成了微服务的架构；也可以被前端调用，这样就可以前后端分离。</p>
<p data-nodeid="1266">今天我就先来为你介绍什么是 RESTful API，以及 Go 语言是如何玩转 RESTful API 的。</p>
<h3 data-nodeid="1267">什么是 RESTful API</h3>
<p data-nodeid="1268">RESTful API 是一套规范，它可以规范我们如何对服务器上的资源进行操作。在了解 RESTful API 之前，我先为你介绍下 HTTP Method，因为 RESTful API 和它是密不可分的。</p>
<p data-nodeid="1269">说起 HTTP Method，最常见的就是<strong data-nodeid="1438">POST</strong>和<strong data-nodeid="1439">GET</strong>，其实最早在 HTTP 0.9 版本中，只有一个<strong data-nodeid="1440">GET</strong>方法，该方法是一个<strong data-nodeid="1441">幂等方法</strong>，用于获取服务器上的资源，也就是我们在浏览器中直接输入网址回车请求的方法。</p>
<p data-nodeid="1270">在 HTTP 1.0 版本中又增加了<strong data-nodeid="1451">HEAD</strong>和<strong data-nodeid="1452">POST</strong>方法，其中常用的是 POST 方法，一般用于给服务端提交一个资源，导致服务器的资源发生变化。</p>
<p data-nodeid="1271">随着网络越来越复杂，发现这两个方法是不够用的，就继续新增了方法。所以在 HTTP1.1 版本的时候，一口气增加到了 9 个，新增的方法有 HEAD、OPTIONS、PUT、DELETE、TRACE、PATCH 和 CONNECT。下面我为你一一介绍它们的作用。</p>
<ol data-nodeid="1272">
<li data-nodeid="1273">
<p data-nodeid="1274">GET 方法可请求一个指定资源的表示形式，使用 GET 的请求应该只被用于获取数据。</p>
</li>
<li data-nodeid="1275">
<p data-nodeid="1276">HEAD 方法用于请求一个与 GET 请求的响应相同的响应，但没有响应体。</p>
</li>
<li data-nodeid="1277">
<p data-nodeid="1278">POST 方法用于将实体提交到指定的资源，通常导致服务器上的状态变化或副作用。</p>
</li>
<li data-nodeid="1279">
<p data-nodeid="1280">PUT 方法用于请求有效载荷替换目标资源的所有当前表示。</p>
</li>
<li data-nodeid="1281">
<p data-nodeid="1282">DELETE 方法用于删除指定的资源。</p>
</li>
<li data-nodeid="1283">
<p data-nodeid="1284">CONNECT 方法用于建立一个到由目标资源标识的服务器的隧道。</p>
</li>
<li data-nodeid="1285">
<p data-nodeid="1286">OPTIONS 方法用于描述目标资源的通信选项。</p>
</li>
<li data-nodeid="1287">
<p data-nodeid="1288">TRACE 方法用于沿着到目标资源的路径执行一个消息环回测试。</p>
</li>
<li data-nodeid="1289">
<p data-nodeid="1290">PATCH 方法用于对资源应用部分修改。</p>
</li>
</ol>
<p data-nodeid="1291">从以上每个方法的介绍可以看到，HTTP 规范针对每个方法都给出了明确的定义，所以我们使用的时候也要尽可能地<strong data-nodeid="1468">遵循这些定义</strong>，这样我们在开发中才可以更好地协作。</p>
<p data-nodeid="1292">理解了这些 HTTP 方法，就可以更好地理解 RESTful API 规范了，因为 RESTful API 规范就是基于这些 HTTP 方法规范我们对服务器资源的操作，同时规范了 URL 的样式和 HTTP Status Code。</p>
<p data-nodeid="1293">在 RESTful API 中，使用的主要是以下五种 HTTP 方法：</p>
<ol data-nodeid="1294">
<li data-nodeid="1295">
<p data-nodeid="1296">GET，表示读取服务器上的资源；</p>
</li>
<li data-nodeid="1297">
<p data-nodeid="1298">POST，表示在服务器上创建资源；</p>
</li>
<li data-nodeid="1299">
<p data-nodeid="1300">PUT，表示更新或者替换服务器上的资源；</p>
</li>
<li data-nodeid="1301">
<p data-nodeid="1302">DELETE，表示删除服务器上的资源；</p>
</li>
<li data-nodeid="1303">
<p data-nodeid="1304">PATCH，表示更新 / 修改资源的一部分。</p>
</li>
</ol>
<p data-nodeid="1305">以上 HTTP 方法在 RESTful API 规范中是一个操作，操作的就是服务器的资源，服务器的资源通过特定的 URL 表示。</p>
<p data-nodeid="1306">现在我们通过一些示例让你更好地理解 RESTful API，如下所示：</p>
<pre class="lang-java" data-nodeid="1307"><code data-language="java">HTTP GET https:<span class="hljs-comment">//www.flysnow.org/users</span>
HTTP GET https:<span class="hljs-comment">//www.flysnow.org/users/123</span>
</code></pre>
<p data-nodeid="1308">以上是两个 GET 方法的示例：</p>
<ul data-nodeid="1309">
<li data-nodeid="1310">
<p data-nodeid="1311">第一个表示获取所有用户的信息；</p>
</li>
<li data-nodeid="1312">
<p data-nodeid="1313">第二个表示获取 ID 为 123 用户的信息。</p>
</li>
</ul>
<p data-nodeid="1314">下面再看一个 POST 方法的示例，如下所示：</p>
<pre class="lang-java" data-nodeid="1315"><code data-language="java">HTTP POST https:<span class="hljs-comment">//www.flysnow.org/users</span>
</code></pre>
<p data-nodeid="1316">这个示例表示创建一个用户，通过 POST 方法给服务器提供创建这个用户所需的全部信息。</p>
<blockquote data-nodeid="1317">
<p data-nodeid="1318">注意：这里 users 是个复数。</p>
</blockquote>
<p data-nodeid="1319">现在你已经知道了如何创建一个用户，那么如果要更新某个特定的用户怎么做呢？其实也非常简单，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="1320"><code data-language="java">HTTP PUT https:<span class="hljs-comment">//www.flysnow.org/users/123</span>
</code></pre>
<p data-nodeid="1321">这表示要更新 / 替换 ID 为 123 的这个用户，在更新的时候，会通过 PUT 方法提供更新这个用户需要的全部用户信息。这里 PUT 方法和 POST 方法不太一样的是，从 URL 上看，PUT 方法操作的是单个资源，比如这里 ID 为 123 的用户。</p>
<blockquote data-nodeid="1322">
<p data-nodeid="1323">小提示：如果要更新一个用户的部分信息，使用 PATCH 方法更恰当。</p>
</blockquote>
<p data-nodeid="1324">看到这里，相信你已经知道了如何删除一个用户，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="1325"><code data-language="java">HTTP DELETE https:<span class="hljs-comment">//www.flysnow.org/users/123</span>
</code></pre>
<p data-nodeid="1326">DELETE 方法的使用和 PUT 方法一样，也是操作单个资源，这里是删除 ID 为 123 的这个用户。</p>
<h3 data-nodeid="1327">一个简单的 RESTful API</h3>
<p data-nodeid="1328">相信你已经非常了解什么是 RESTful API 了，现在开始，我会带你通过一个使用 Golang 实现 RESTful API 风格的示例，加深 RESTful API 的理解。</p>
<p data-nodeid="1329">Go 语言的一个很大的优势，就是可以很容易地开发出网络后台服务，而且性能快、效率高。在开发后端 HTTP 网络应用服务的时候，我们需要处理很多 HTTP 的请求访问，比如常见的RESTful API 服务，就要处理很多 HTTP 请求，然后把处理的信息返回给使用者。对于这类需求，Golang 提供了内置的 net/http 包帮我们处理这些 HTTP 请求，让我们可以比较方便地开发一个 HTTP 服务。</p>
<p data-nodeid="1330">下面我们来看一个简单的 HTTP 服务的 Go 语言实现，代码如下所示：</p>
<p data-nodeid="1331"><em data-nodeid="1496">ch21/main.go</em></p>
<pre class="lang-go" data-nodeid="1332"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   http.HandleFunc(<span class="hljs-string">"/users"</span>,handleUsers)
   http.ListenAndServe(<span class="hljs-string">":8080"</span>, <span class="hljs-literal">nil</span>)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">handleUsers</span><span class="hljs-params">(w http.ResponseWriter, r *http.Request)</span></span>{
   fmt.Fprintln(w,<span class="hljs-string">"ID:1,Name:张三"</span>)
   fmt.Fprintln(w,<span class="hljs-string">"ID:2,Name:李四"</span>)
   fmt.Fprintln(w,<span class="hljs-string">"ID:3,Name:王五"</span>)
}
</code></pre>
<p data-nodeid="1333">这个示例运行后，你在浏览器中输入 http://localhost:8080/users, 就可以看到如下内容信息：</p>
<pre class="lang-java" data-nodeid="1334"><code data-language="java">ID:<span class="hljs-number">1</span>,Name:张三
ID:<span class="hljs-number">2</span>,Name:李四
ID:<span class="hljs-number">3</span>,Name:王五
</code></pre>
<p data-nodeid="1335">也就是获取所有的用户信息，但是这并不是一个 RESTful API，因为使用者不仅可以通过 HTTP GET 方法获得所有的用户信息，还可以通过 POST、DELETE、PUT 等 HTTP 方法获得所有的用户信息，这显然不符合 RESTful API 的规范。</p>
<p data-nodeid="1336">现在我对以上示例进行修改，使它符合 RESTful API 的规范，修改后的示例代码如下所示：</p>
<p data-nodeid="1337"><em data-nodeid="1503">ch20/main.go</em></p>
<pre class="lang-go" data-nodeid="1338"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">handleUsers</span><span class="hljs-params">(w http.ResponseWriter, r *http.Request)</span></span>{
   <span class="hljs-keyword">switch</span> r.Method {
   <span class="hljs-keyword">case</span> <span class="hljs-string">"GET"</span>:
      w.WriteHeader(http.StatusOK)
      fmt.Fprintln(w,<span class="hljs-string">"ID:1,Name:张三"</span>)
      fmt.Fprintln(w,<span class="hljs-string">"ID:2,Name:李四"</span>)
      fmt.Fprintln(w,<span class="hljs-string">"ID:3,Name:王五"</span>)
   <span class="hljs-keyword">default</span>:
      w.WriteHeader(http.StatusNotFound)
      fmt.Fprintln(w,<span class="hljs-string">"not found"</span>)
   }
}
</code></pre>
<p data-nodeid="1339">这里我只修改了 handleUsers 函数，在该函数中增加了只在使用 GET 方法时，才获得所有用户的信息，其他情况返回 not found。</p>
<p data-nodeid="1340">现在再运行这个示例，会发现只能通过 HTTP GET 方法进行访问了，使用其他方法会提示 not found。</p>
<h3 data-nodeid="1341">RESTful JSON API</h3>
<p data-nodeid="1342">在项目中最常见的是使用 JSON 格式传输信息，也就是我们提供的 RESTful API 要返回 JSON 内容给使用者。</p>
<p data-nodeid="1343">同样用上面的示例，我把它改造成可以返回 JSON 内容的方式，示例代码如下所示：</p>
<p data-nodeid="1344"><em data-nodeid="1512">ch20/main.go</em></p>
<pre class="lang-go" data-nodeid="1345"><code data-language="go"><span class="hljs-comment">//数据源，类似MySQL中的数据</span>
<span class="hljs-keyword">var</span> users = []User{
   {ID: <span class="hljs-number">1</span>,Name: <span class="hljs-string">"张三"</span>},
   {ID: <span class="hljs-number">2</span>,Name: <span class="hljs-string">"李四"</span>},
   {ID: <span class="hljs-number">3</span>,Name: <span class="hljs-string">"王五"</span>},
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">handleUsers</span><span class="hljs-params">(w http.ResponseWriter, r *http.Request)</span></span>{
   <span class="hljs-keyword">switch</span> r.Method {
   <span class="hljs-keyword">case</span> <span class="hljs-string">"GET"</span>:
      users,err:=json.Marshal(users)
      <span class="hljs-keyword">if</span> err!=<span class="hljs-literal">nil</span> {
         w.WriteHeader(http.StatusInternalServerError)
         fmt.Fprint(w,<span class="hljs-string">"{\"message\": \""</span>+err.Error()+<span class="hljs-string">"\"}"</span>)
      }<span class="hljs-keyword">else</span> {
         w.WriteHeader(http.StatusOK)
         w.Write(users)
      }
   <span class="hljs-keyword">default</span>:
      w.WriteHeader(http.StatusNotFound)
      fmt.Fprint(w,<span class="hljs-string">"{\"message\": \"not found\"}"</span>)
   }
}
<span class="hljs-comment">//用户</span>
<span class="hljs-keyword">type</span> User <span class="hljs-keyword">struct</span> {
   ID <span class="hljs-keyword">int</span>
   Name <span class="hljs-keyword">string</span>
}
</code></pre>
<p data-nodeid="1346">从以上代码可以看到，这次的改造主要是新建了一个 User 结构体，并且使用 users 这个切片存储所有的用户，然后在 handleUsers 函数中把它转化为一个 JSON 数组返回。这样，就实现了基于 JSON 数据格式的 RESTful API。</p>
<p data-nodeid="1347">运行这个示例，在浏览器中输入 http://localhost:8080/users，可以看到如下信息：</p>
<pre class="lang-java" data-nodeid="1348"><code data-language="java">[{<span class="hljs-string">"ID"</span>:<span class="hljs-number">1</span>,<span class="hljs-string">"Name"</span>:<span class="hljs-string">"张三"</span>},{<span class="hljs-string">"ID"</span>:<span class="hljs-number">2</span>,<span class="hljs-string">"Name"</span>:<span class="hljs-string">"李四"</span>},{<span class="hljs-string">"ID"</span>:<span class="hljs-number">3</span>,<span class="hljs-string">"Name"</span>:<span class="hljs-string">"王五"</span>}]
</code></pre>
<p data-nodeid="1349">这已经是 JSON 格式的用户信息，包含了所有用户。</p>
<h3 data-nodeid="1350">Gin 框架</h3>
<p data-nodeid="1351">虽然 Go 语言自带的 net/http 包，可以比较容易地创建 HTTP 服务，但是它也有很多不足：</p>
<ul data-nodeid="1352">
<li data-nodeid="1353">
<p data-nodeid="1354">不能单独地对请求方法（POST、GET 等）注册特定的处理函数；</p>
</li>
<li data-nodeid="1355">
<p data-nodeid="1356">不支持 Path 变量参数；</p>
</li>
<li data-nodeid="1357">
<p data-nodeid="1358">不能自动对 Path 进行校准；</p>
</li>
<li data-nodeid="1359">
<p data-nodeid="1360">性能一般；</p>
</li>
<li data-nodeid="1361">
<p data-nodeid="1362">扩展性不足；</p>
</li>
<li data-nodeid="1363">
<p data-nodeid="1364">……</p>
</li>
</ul>
<p data-nodeid="1365">基于以上这些不足，出现了很多 Golang Web 框架，如 Mux，Gin、Fiber 等，今天我要为你介绍的就是这款使用最多的 Gin 框架。</p>
<h4 data-nodeid="1366">引入 Gin 框架</h4>
<p data-nodeid="1367">Gin 框架是一个在 Github 上开源的 Web 框架，封装了很多 Web 开发需要的通用功能，并且性能也非常高，可以让我们很容易地写出 RESTful API。</p>
<p data-nodeid="1368">Gin 框架其实是一个模块，也就是 Go Mod，所以采用 Go Mod 的方法引入即可。我在第 18讲的时候详细介绍过如何引入第三方的模块，这里再复习一下。</p>
<p data-nodeid="1369">首先需要下载安装 Gin 框架，安装代码如下：</p>
<pre class="lang-shell" data-nodeid="1370"><code data-language="shell"><span class="hljs-meta">$</span><span class="bash"> go get -u github.com/gin-gonic/gin</span>
</code></pre>
<p data-nodeid="1371">然后就可以在 Go 语言代码中导入使用了，导入代码如下：</p>
<pre class="lang-go" data-nodeid="1372"><code data-language="go"><span class="hljs-keyword">import</span> <span class="hljs-string">"github.com/gin-gonic/gin"</span>
</code></pre>
<p data-nodeid="1373">通过以上安装和导入这两个步骤，就可以在你的 Go 语言项目中使用 Gin 框架了。</p>
<h4 data-nodeid="1374">使用 Gin 框架</h4>
<p data-nodeid="1375">现在，已经引入了 Gin 框架，下面我就是用 Gin 框架重写上面的示例，修改的代码如下所示：</p>
<p data-nodeid="1376"><em data-nodeid="1536">ch21/main.go</em></p>
<pre class="lang-go" data-nodeid="1377"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   r:=gin.Default()
   r.GET(<span class="hljs-string">"/users"</span>, listUser)
   r.Run(<span class="hljs-string">":8080"</span>)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">listUser</span><span class="hljs-params">(c *gin.Context)</span></span>  {
   c.JSON(<span class="hljs-number">200</span>,users)
}
</code></pre>
<p data-nodeid="1378">相比 net/http 包，Gin 框架的代码非常简单，通过它的 GET 方法就可以创建一个只处理 HTTP GET 方法的服务，而且输出 JSON 格式的数据也非常简单，使用 c.JSON 方法即可。</p>
<p data-nodeid="1379">最后通过 Run 方法启动 HTTP 服务，监听在 8080 端口。现在运行这个 Gin 示例，在浏览器中输入 http://localhost:8080/users，看到的信息和通过 net/http 包实现的效果是一样的。</p>
<h4 data-nodeid="1380">获取特定的用户</h4>
<p data-nodeid="1381">现在你已经掌握了如何使用 Gin 框架创建一个简单的 RESTful API，并且可以返回所有的用户信息，那么如何获取特定用户的信息呢？</p>
<p data-nodeid="1382">我们知道，如果要获得特定用户的信息，需要使用的是 GET 方法，并且 URL 格式如下所示：</p>
<pre class="lang-java" data-nodeid="1383"><code data-language="java">http:<span class="hljs-comment">//localhost:8080/users/2</span>
</code></pre>
<p data-nodeid="1384">以上示例中的 2 是用户的 ID，也就是通过 ID 来获取特定的用户。</p>
<p data-nodeid="1385">下面我通过 Gin 框架 Path 路径参数来实现这个功能，示例代码如下：</p>
<p data-nodeid="1386"><em data-nodeid="1547">ch21/main.go</em></p>
<pre class="lang-go" data-nodeid="1387"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-comment">//省略没有改动的代码</span>
   r.GET(<span class="hljs-string">"/users/:id"</span>, getUser)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">getUser</span><span class="hljs-params">(c *gin.Context)</span></span> {
   id := c.Param(<span class="hljs-string">"id"</span>)
   <span class="hljs-keyword">var</span> user User
   found := <span class="hljs-literal">false</span>
   <span class="hljs-comment">//类似于数据库的SQL查询</span>
   <span class="hljs-keyword">for</span> _, u := <span class="hljs-keyword">range</span> users {
      <span class="hljs-keyword">if</span> strings.EqualFold(id, strconv.Itoa(u.ID)) {
         user = u
         found = <span class="hljs-literal">true</span>
         <span class="hljs-keyword">break</span>
      }
   }
   <span class="hljs-keyword">if</span> found {
      c.JSON(<span class="hljs-number">200</span>, user)
   } <span class="hljs-keyword">else</span> {
      c.JSON(<span class="hljs-number">404</span>, gin.H{
         <span class="hljs-string">"message"</span>: <span class="hljs-string">"用户不存在"</span>,
      })
   }
}
</code></pre>
<p data-nodeid="1388">在 Gin 框架中，路径中使用冒号表示 Path 路径参数，比如示例中的 :id，然后在 getUser 函数中可以通过 c.Param("id") 获取需要查询用户的 ID 值。</p>
<blockquote data-nodeid="1389">
<p data-nodeid="1390">小提示：Param 方法的参数要和 Path 路径参数中的一致，比如示例中都是 ID。</p>
</blockquote>
<p data-nodeid="1391">现在运行这个示例，通过浏览器访问 <a href="http://localhost:8080/users/2" data-nodeid="1557">http://localhost:8080/users/2</a>，就可以获得 ID 为 2 的用户，输出信息如下所示：</p>
<pre class="lang-java" data-nodeid="1392"><code data-language="java">{<span class="hljs-string">"ID"</span>:<span class="hljs-number">2</span>,<span class="hljs-string">"Name"</span>:<span class="hljs-string">"李四"</span>}
</code></pre>
<p data-nodeid="1393">可以看到，已经正确的获取到了 ID 为 2 的用户，他的名字叫李四。</p>
<p data-nodeid="1394">假如我们访问一个不存在的 ID，会得到什么结果呢？比如 99，示例如下所示：</p>
<pre class="lang-java" data-nodeid="1395"><code data-language="java">➜ curl http:<span class="hljs-comment">//localhost:8080/users/99</span>
{<span class="hljs-string">"message"</span>:<span class="hljs-string">"用户不存在"</span>}%
</code></pre>
<p data-nodeid="1396">从以上示例输出可以看到，返回了『用户不存在』的信息，和我们代码中处理的逻辑一样。</p>
<h4 data-nodeid="1397">新增一个用户</h4>
<p data-nodeid="1398">现在你已经可以使用 Gin 获取所有用户，还可以获取特定的用户，那么你也应该知道如何新增一个用户了，现在我通过 Gin 实现如何新增一个用户，看和你想的方案是否相似。</p>
<p data-nodeid="1399">根据 RESTful API 规范，实现新增使用的是 POST 方法，并且 URL 的格式为 http://localhost:8080/users ，向这个 URL 发送数据，就可以新增一个用户，然后返回创建的用户信息。</p>
<p data-nodeid="1400">现在我使用 Gin 框架实现新增一个用户，示例代码如下：</p>
<pre class="lang-go" data-nodeid="1401"><code data-language="go"><span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">main</span><span class="hljs-params">()</span></span> {
   <span class="hljs-comment">//省略没有改动的代码</span>
   r.POST(<span class="hljs-string">"/users"</span>, createUser)
}
<span class="hljs-function"><span class="hljs-keyword">func</span> <span class="hljs-title">createUser</span><span class="hljs-params">(c *gin.Context)</span></span> {
   name := c.DefaultPostForm(<span class="hljs-string">"name"</span>, <span class="hljs-string">""</span>)
   <span class="hljs-keyword">if</span> name != <span class="hljs-string">""</span> {
      u := User{ID: <span class="hljs-built_in">len</span>(users) + <span class="hljs-number">1</span>, Name: name}
      users = <span class="hljs-built_in">append</span>(users, u)
      c.JSON(http.StatusCreated,u)
   } <span class="hljs-keyword">else</span> {
      c.JSON(http.StatusOK, gin.H{
         <span class="hljs-string">"message"</span>: <span class="hljs-string">"请输入用户名称"</span>,
      })
   }
}
</code></pre>
<p data-nodeid="1402">以上新增用户的主要逻辑是获取客户端上传的 name 值，然后生成一个 User 用户，最后把它存储到 users 集合中，达到新增用户的目的。</p>
<p data-nodeid="1403">在这个示例中，使用 POST 方法来新增用户，所以只能通过 POST 方法才能新增用户成功。</p>
<p data-nodeid="1404">现在运行这个示例，然后通过如下命令发送一个新增用户的请求，查看结果：</p>
<pre class="lang-java" data-nodeid="1405"><code data-language="java">➜ curl -X POST -d <span class="hljs-string">'name=飞雪'</span> http:<span class="hljs-comment">//localhost:8080/users</span>
{<span class="hljs-string">"ID"</span>:<span class="hljs-number">4</span>,<span class="hljs-string">"Name"</span>:<span class="hljs-string">"飞雪"</span>}
</code></pre>
<p data-nodeid="1406">可以看到新增用户成功，并且返回了新增的用户，还有分配的 ID。</p>
<h3 data-nodeid="1407">总结</h3>
<p data-nodeid="1408">Go 语言已经给我们提供了比较强大的 SDK，让我们可以很容易地开发网络服务的应用，而借助第三方的 Web 框架，可以让这件事情更容易、更高效。比如这篇文章介绍的 Gin 框架，就可以很容易让我们开发出 RESTful API，更多关于 Gin 框架的使用可以参考 <a href="https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&amp;album_id=1362784031968149504&amp;__biz=MzI3MjU4Njk3Ng==#wechat_redirect" data-nodeid="1574">Golang Gin 实战</a>系列文章。</p>
<p data-nodeid="1581" class="te-preview-highlight">在我们做项目开发的时候，要善于借助已经有的轮子，让自己的开发更有效率，也更容易实现。<br>
<img src="https://s0.lgstatic.com/i/image/M00/8C/DA/Ciqc1F_1dACARBqrAAVSvK3wokw352.png" alt="go语言金句.png" data-nodeid="1586"><br>
在我们做项目开发的时候，会有增、删、改和查，现在增和查你已经学会了，那么就给你留 2 个作业，任选其中 1 个即可，它们是：</p>


<ol data-nodeid="1411">
<li data-nodeid="1412">
<p data-nodeid="1413">修改一个用户的名字；</p>
</li>
<li data-nodeid="1414">
<p data-nodeid="1415">删除一个用户。</p>
</li>
</ol>
<p data-nodeid="1416" class="">下一讲，也就是本专栏的最后一讲，我将为你介绍如何使用 Go 语言实现 RPC 服务，记得来听课哦。</p>

---

### 精选评论

##### **伟：
> 坚持打卡。加油！我相信自己会成功的。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油

##### *轩：
> 又有了更深刻的认识

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 加油，马上就结束了，到时候再回过头来想一想，一定学有所获的。

##### *磊：
> 写的很不错，每期都仔细看完

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 学会一定会有收获的，加油

