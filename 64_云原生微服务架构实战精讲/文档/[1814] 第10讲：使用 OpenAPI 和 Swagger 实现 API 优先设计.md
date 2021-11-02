<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从本课时开始，我们将进入到云原生微服务架构应用的实战开发环节，在介绍微服务的具体实现之前，首要的工作是设计和确定每个微服务的</span><strong style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">开放 API</strong><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">。开放 API 在近几年得到了广泛的流行，很多在线服务和政府机构都对外提供了开放 API，其已经成为在线服务的标配功能。开发者可以利用开放 API 开发出各种不同的应用。</span><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">微服务应用中的开放 API 与在线服务的开放 API，虽然存在一定的关联，但作用是不同的。在微服务架构的应用中，微服务之间只能通过进程间的通讯方式来交互，一般使用 REST 或 gRPC。这样的交互方式需要以形式化的方式固定下来，就形成了开放 API，一个微服务的开放 API 对外部使用者屏蔽了服务内部的实现细节，同时也是外部使用者与之交互的唯一方式（当然，这里指的是微服务之间仅通过 API 来进行集成，如果使用异步事件来进行集成的话，这些事件也是交互方式）。由此可以看出，微服务 API 的重要性。从受众的角度来说，微服务API的使用者主要是其他微服务，也就是说，主要是应用内部的使用者，这一点与在线服务的 API 主要面向外部用户是不同的。除了其他微服务之外，应用的 Web 界面和移动客户端也需要使用微服务的 API，不过，它们一般通过 API 网关来使用微服务的 API。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">由于微服务 API 的重要性，我们需要在很早的时候就得进行 API 的设计，也就是 API 优先的策略。</span></p>
<h2 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">API 优先的策略</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">如果你有过开发在线服务 API 的经验，会发现通常是先有实现，再有公开 API，这是因为在设计之前，并没有考虑到公开 API 的需求，而是之后才添加的公开 API。这种做法带来的结果就是，开放的 API 只是反映了当前的实际实现，而不是 API 应该有的样子。API 优先（API First）的设计方式，是把 API 的设计放在具体的实现之前，API 优先强调应该更多地从 API 使用者的角度来考虑 API 的设计。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在写下第一行实现的代码之前，API 的提供者和使用者应该对 API 进行充分的讨论，综合两方面的意见，最终确定 API 的全部细节，并以形式化的格式固定下来，成为 API 规范。在这之后，API 的提供者确保实际的实现满足 API 规范的要求，使用者则根据 API 规范来编写客户端实现。API 规范是提供者和使用者之间的契约，API 优先的策略已经应用在很多在线服务的开发中了。API 设计并实现出来之后，在线服务自身的 Web 界面和移动端应用，和其他第三方应用一样，都使用相同的 API 实现。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">API 优先的策略，在微服务架构的应用实现中，有着更加重要的作用。这里有必要区分两类 API：一类是提供给其他微服务使用的 API，另一类是提供给 Web 界面和移动客户端使用的 API。在第 07 课时中介绍领域驱动设计的时候，我提到过界定的上下文的映射模式中的开放主机服务和公开语言，微服务与界定的上下文是一一对应的。如果把开放主机服务和公共语言结合起来，就得到了微服务的 API，公共语言就是 API 的规范。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">从这里我们可以知道，第一类微服务 API 的目的是进行上下文映射，与第二类 API 的作用存在显著的不同。举例来说，乘客管理微服务提供了管理乘客的功能，包括乘客注册、信息更新和查询等。对于乘客 App 来说，这些功能都需要 API 的支持，其他微服务如果需要获取乘客信息，也必须调用乘客管理微服务的 API。这是为了把乘客这个概念在不同的微服务之间进行映射。</span></p>
<h2 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">API 实现方式</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 API 实现中，首要的一个问题是选择 API 的实现方式。理论上来说，微服务的内部 API 对互操作性的要求不高，可以使用私有格式。不过，为了可以使用服务网格，推荐使用通用的标准格式，下表给出了常见的 API 格式。除了使用较少的 SOAP 之外，一般在 REST 和 gRPC 之间选择。两者的区别在于，REST 使用文本格式，gRPC 使用二进制格式；两者在流行程度、实现难度和性能上存在不同。简单来说，REST 相对更加流行，实现难度较低，但是性能不如 gRPC。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/03/D7/CgoCgV6ZTAOAe8LCAAAmZRAsEQw388.png"></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">本专栏的示例应用的 API 使用 REST 实现，不过会有一个课时专门来介绍 gRPC。下面介绍与 REST API 相关的 OpenAPI 规范。</span></p>
<h2 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OpenAPI 规范</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">为了在 API 提供者和使用者之间更好的沟通，我们需要一种描述 API 的标准格式。对于 REST API 来说，这个标准格式由 OpenAPI 规范来定义。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OpenAPI 规范（OpenAPI Specification，OAS）是由 Linux 基金会旗下的 OpenAPI 倡议（OpenAPI Initiative，OAI）管理的开放 API 的规范。OAI 的目标是创建、演化和推广一种供应商无关的 API 描述格式。OpenAPI 规范基于 Swagger 规范，由 SmartBear 公司捐赠而来。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OpenAPI 文档描述或定义 API，OpenAPI 文档必须满足 OpenAPI 规范。OpenAPI 规范定义了 OpenAPI 文档的内容格式，也就是其中所能包含的对象及其属性。OpenAPI 文档是一个 JSON 对象，可以用 JSON 或 YAML 文件格式来表示。下面对 OpenAPI 文档的格式进行介绍，本课时的代码示例都使用 YAML 格式。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OpenAPI 规范中定义了几种基本类型，分别是 integer、number、string 和 boolean。对于每种基本类型，可以通过 format 字段来指定数据类型的具体格式，比如 string 类型的格式可以是 date、date-time 或 password。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下表中给出了 OpenAPI 文档的根对象中可以出现的字段及其说明，目前 OpenAPI 规范的最新版本是 3.0.3。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><img src="https://s0.lgstatic.com/i/image3/M01/11/06/Ciqah16ZTAOAAwW0AACP-qu5xrk547.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Info 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Info 对象包含了 API 的元数据，可以帮助使用者更好的了解 API 的相关信息。下表给出了 Info 对象中可以包含的字段及其说明。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/8A/1C/Cgq2xl6ZTAOAdpURAABX1Kfh-DM443.png"></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面的代码是 Info 对象的使用示例。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>title:&nbsp;测试服务
description:&nbsp;该服务用来进行简单的测试
termsOfService:&nbsp;http://myapp.com/terms/
contact:
&nbsp;&nbsp;name:&nbsp;管理员
&nbsp;&nbsp;url:&nbsp;http://www.myapp.com/support
&nbsp;&nbsp;email:&nbsp;support@myapp.com
license:
&nbsp;&nbsp;name:&nbsp;Apache&nbsp;2.0
&nbsp;&nbsp;url:&nbsp;https://www.apache.org/licenses/LICENSE-2.0.html
version:&nbsp;2.1.0</pre>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Server 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Server 对象表示 API 的服务器，下表给出了 Server 对象中可以包含的字段及其说明。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><img src="https://s0.lgstatic.com/i/image3/M01/03/D7/CgoCgV6ZTAOABcF1AAA-57NeBGQ901.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp; &nbsp; &nbsp;</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面代码是 Server 对象的使用示例，其中服务器的 URL 中包含了 port 和 basePath 两个参数，port 是枚举类型，可选值是 80 和 8080。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>url:&nbsp;http://test.myapp.com:{port}/{basePath}
description:&nbsp;测试服务器
variables:
&nbsp;&nbsp;port:
&nbsp;&nbsp;&nbsp;&nbsp;enum:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;'80'
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;'8080'
&nbsp;&nbsp;&nbsp;&nbsp;default:&nbsp;'80'
&nbsp;&nbsp;basePath:
&nbsp;&nbsp;&nbsp;&nbsp;default:&nbsp;v2</pre>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Paths 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Paths 对象中的字段是动态的。每个字段表示一个路径，以“/”开头，路径可以是包含变量的字符串模板。字段的值是 PathItem 对象，在该对象中可以使用 summary、description、servers 和 parameters 这样的通用字段，还可以使用 HTTP 方法名称，包括 get、put、post、delete、options、head、patch 和 trace，这些方法名称的字段，定义了对应的路径所支持的 HTTP 方法。</span></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Operation 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在 Paths 对象中，HTTP 方法对应的字段的值的类型是 Operation 对象，表示 HTTP 操作。下表给出了 Operation 对象中可以包含的字段及其说明，在这些字段中，比较常用的是 parameters、requestBody 和 responses。</span></p>
<p style="white-space: normal;"><br></p>
<p style="white-space: normal;"><img src="https://s0.lgstatic.com/i/image3/M01/11/06/Ciqah16ZTAOAC-XoAAC9Yc_U9FY508.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Parameter 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Parameter 对象表示操作的参数。下表给出了 Parameter 对象中可以包含的字段及其说明。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><img src="https://s0.lgstatic.com/i/image3/M01/8A/1C/Cgq2xl6ZTAOANejbAADhE2FZLo4096.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面的代码是 Parameter 对象的使用示例，参数 id 出现在路径中，类型是 string。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>name:&nbsp;id
in:&nbsp;path
description:&nbsp;乘客ID
required:&nbsp;true
schema:
&nbsp;&nbsp;type:&nbsp;string</pre>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">RequestBody 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">RequestBody 对象表示 HTTP 请求的内容，下表给出了 RequestBody 对象中可以包含的字段及其说明。</span></p>
<p style="white-space: normal;"><br></p>
<p style="white-space: normal;"><img src="https://s0.lgstatic.com/i/image3/M01/03/D7/CgoCgV6ZTAOAU9XxAAA4YbmwX3A208.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Responses 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Responses 对象表示 HTTP 请求的响应，该对象中的字段是动态的。字段的名称是 HTTP 响应的状态码，对应的值的类型是 Response 或 Reference 对象。下表给出了 Response 对象中可以包含的字段及其说明。</span></p>
<p style="white-space: normal;"><br></p>
<p style="white-space: normal;"><img src="https://s0.lgstatic.com/i/image3/M01/11/06/Ciqah16ZTAOAHz9LAAB3y1Wy79M022.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;;"><br></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Reference 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在对不同类型的对象描述中，字段的类型可以是 Reference 对象，该对象表示对其他组件的引用，其中只包含一个 $ref 字段来声明引用。引用可以是同一文档中的组件，也可以来自外部文件。在文档内部，可以在 Components 对象中定义不同类型的可复用组件，并由 Reference 对象来引用；文档内部的引用是以 # 开头的对象路径，比如 #/components/schemas/CreateTripRequest。</span></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Schema 对象</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Schema 对象用来描述数据类型的定义，数据类型可以是简单类型、数组或对象类型，通过字段 type 可以指定类型，format 字段表示类型的格式。如果是数组类型，即 type 的值是 array，则需要通过字段 items 来表示数组中元素的类型；如果是对象类型，即 type 的值是 object，则需要通过字段 properties 来表示对象中属性的类型。</span></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">完整文档示例</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面是一个完整的 OpenAPI 文档示例。在 paths 对象中，定义了 3 个操作，操作的请求内容和响应格式的类型定义，都在 Components 对象的 schemas 字段中定义。操作的 requestBody 和 responses 字段都使用 Reference 对象来引用。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>openapi:&nbsp;'3.0.3'
info:
&nbsp;&nbsp;title:&nbsp;行程服务
&nbsp;&nbsp;version:&nbsp;'1.0'
servers:
&nbsp;&nbsp;-&nbsp;url:&nbsp;http://localhost:8501/api/v1
tags:
&nbsp;&nbsp;-&nbsp;name:&nbsp;trip
&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;行程相关
paths:
&nbsp;&nbsp;/:
&nbsp;&nbsp;&nbsp;&nbsp;post:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tags:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;trip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;summary:&nbsp;创建行程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;operationId:&nbsp;createTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;requestBody:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;content:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;application/json:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/CreateTripRequest"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:&nbsp;true&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;responses:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;'201':
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;创建成功
&nbsp;&nbsp;/{tripId}:
&nbsp;&nbsp;&nbsp;&nbsp;get:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tags:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;trip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;summary:&nbsp;获取行程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;operationId:&nbsp;getTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;tripId
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;in:&nbsp;path
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;行程ID
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:&nbsp;true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;responses:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;'200':
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;获取成功&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;content:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;application/json:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/TripVO"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;'404':
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;找不到行程
&nbsp;&nbsp;/{tripId}/accept:
&nbsp;&nbsp;&nbsp;&nbsp;post:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tags:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;trip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;summary:&nbsp;接受行程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;operationId:&nbsp;acceptTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;parameters:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;name:&nbsp;tripId
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;in:&nbsp;path
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;行程ID
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:&nbsp;true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;requestBody:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;content:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;application/json:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;schema:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/AcceptTripRequest"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:&nbsp;true
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;responses:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;'200':
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;description:&nbsp;接受成功
components:
&nbsp;&nbsp;schemas:
&nbsp;&nbsp;&nbsp;&nbsp;CreateTripRequest:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;object
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;properties:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;passengerId:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;startPos:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/PositionVO"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endPos:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/PositionVO"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;passengerId
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;startPos
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;endPos
&nbsp;&nbsp;&nbsp;&nbsp;AcceptTripRequest:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;object
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;properties:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;driverId:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;posLng:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format:&nbsp;double
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;posLat:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format:&nbsp;double
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;driverId
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;posLng
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;posLat
&nbsp;&nbsp;&nbsp;&nbsp;TripVO:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;object
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;properties:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id:&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;passengerId:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;driverId:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;startPos:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/PositionVO"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endPos:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$ref:&nbsp;"#/components/schemas/PositionVO"
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;state:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;PositionVO:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;object
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;properties:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lng:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format:&nbsp;double
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;lat:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;number
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;format:&nbsp;double&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;addressId:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;type:&nbsp;string&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;required:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;lng
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-&nbsp;lat</pre>
<h2 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">OpenAPI 工具</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">我们可以用一些工具来辅助 OpenAPI 规范相关的开发。作为 OpenAPI 规范的前身，Swagger 提供了很多与 OpenAPI 相关的工具。</span></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Swagger 编辑器</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Swagger 编辑器是一个 Web 版的 Swagger 和 OpenAPI 文档编辑器。在编辑器的左侧是编辑器，右侧是 API 文档的预览。Swagger 编辑器提供了很多实用功能，包括语法高亮、快速添加不同类型的对象、生成服务器代码和生成客户端代码等。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">使用 Swagger 编辑器时，可以直接使用</span><a class="ql-link ql-author-16565634" href="https://editor.swagger.io/" target="_blank" rel="noopener noreferrer nofollow" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">在线版本</a><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">，也可以在本地运行，在本地运行最简单的方式是使用 Docker 镜像 swaggerapi/swagger-editor。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面的代码启动了 Swagger 编辑器的 Docker 容器，容器启动之后，通过 localhost:8000 访问即可。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>docker&nbsp;run&nbsp;-d&nbsp;-p&nbsp;8000:8080&nbsp;swaggerapi/swagger-editor</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下图是 Swagger 编辑器的界面。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; text-align: center; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><img src="https://s0.lgstatic.com/i/image3/M01/8A/1C/Cgq2xl6ZTASABJOwAAFARz_LfhM630.png"></span></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Swagger 界面</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">Swagger 界面提供了一种直观的方式来查看 API 文档，并进行交互。通过该界面，可以直接发送 HTTP 请求到 API 服务器，并查看响应结果。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">同样，我们可以用 Docker 来启动 Swagger 界面，如下面的命令所示。容器启动之后，通过 localhost:8010 来访问即可。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>docker&nbsp;run&nbsp;-d&nbsp;-p&nbsp;8010:8080&nbsp;swaggerapi/swagger-ui</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">对于本地的 OpenAPI 文档，可以配置 Docker 镜像来使用该文档。假设当前目录中有 OpenAPI 文档 openapi.yml，则可以使用下面的命令来启动 Docker 镜像来展示该文档。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>docker&nbsp;run&nbsp;-p&nbsp;8010:8080&nbsp;-e&nbsp;SWAGGER_JSON=/api/openapi.yml&nbsp;-v&nbsp;$PWD:/api&nbsp;swaggerapi/swagger-ui</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下图是 Swagger 界面的截图。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><img src="https://s0.lgstatic.com/i/image3/M01/03/D7/CgoCgV6ZTASAXGi3AAH17b7200I115.png" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; text-align: center;"><br></p>
<h3 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">代码生成</span></p></h3>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">通过 OpenAPI 文档，可以利用 Swagger 提供的代码生成工具来自动生成服务器存根代码和客户端。代码生成时可以使用不同的编程语言和框架。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">下面给出了代码生成工具所支持的编程语言和框架。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>aspnetcore,&nbsp;csharp,&nbsp;csharp-dotnet2,&nbsp;go-server,&nbsp;dynamic-html,&nbsp;html,&nbsp;html2,&nbsp;java,&nbsp;jaxrs-cxf-client,
&nbsp;jaxrs-cxf,&nbsp;inflector,&nbsp;jaxrs-cxf-cdi,&nbsp;jaxrs-spec,&nbsp;jaxrs-jersey,&nbsp;jaxrs-di,&nbsp;jaxrs-resteasy-eap,&nbsp;jaxrs-resteasy,&nbsp;micronaut
,&nbsp;spring,&nbsp;nodejs-server,&nbsp;openapi,&nbsp;openapi-yaml,&nbsp;kotlin-client,&nbsp;kotlin-server,&nbsp;php,&nbsp;python,&nbsp;python-flask,&nbsp;r,&nbsp;scala,&nbsp;scal
a-akka-http-server,&nbsp;swift3,&nbsp;swift4,&nbsp;swift5,&nbsp;typescript-angular,&nbsp;javascript</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">代码生成工具是一个 Java 程序，下载之后可以直接运行。在下载 JAR 文件&nbsp;</span><a class="ql-link ql-author-16565634" href="https://repo1.maven.org/maven2/io/swagger/codegen/v3/swagger-codegen-cli/3.0.19/swagger-codegen-cli-3.0.19.jar" target="_blank" rel="noopener noreferrer nofollow" style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">swagger-codegen-cli-3.0.19.jar</a><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">&nbsp;之后，可以使用下面的命令来生成 Java 客户端代码，其中 -i 参数指定输入的 OpenAPI 文档，-l 指定生成的语言，-o 指定输出目录。</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<pre>java&nbsp;-jar&nbsp;swagger-codegen-cli-3.0.19.jar&nbsp;generate&nbsp;-i&nbsp;openapi.yml&nbsp;-l&nbsp;java&nbsp;-o&nbsp;/tmp</pre>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><br></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">除了生成客户端代码之外，还可以生成服务器存根代码。下面代码是生成 NodeJS 服务器存根代码：</span></p>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;"><br></span></p>
<pre>java&nbsp;-jar&nbsp;swagger-codegen-cli-3.0.19.jar&nbsp;generate&nbsp;-i&nbsp;openapi.yml&nbsp;-l&nbsp;nodejs-server&nbsp;-o&nbsp;/tmp</pre>
<h2 style="white-space: normal;"><p><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">总结</span></p></h2>
<p style="margin-top: 0pt; margin-bottom: 0pt; white-space: normal; line-height: 1.7; font-size: 11pt; color: rgb(73, 73, 73);"><span style="color: rgb(63, 63, 63); font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px;">API 优先的策略保证了微服务的 API 在设计时，充分考虑到 API 使用者的需求，使得 API 成为提供者和使用者之间的良好契约。本课时首先介绍了 API 优先的设计策略，然后介绍了 API 的不同实现方式，接着介绍了 REST API 的 OpenAPI 规范，最后介绍了 OpenAPI 的相关工具。</span></p>

---

### 精选评论

##### *军：
> 刚装了edit，貌似也可以测试接口，不知道和ui功能有啥不一样？另外自动生成的格式一般用吗？感觉自动代码太啰嗦了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Swagger的Editor主要是用来编辑OpenAPI规范的，还提供了代码生成的功能，也可以测试。Swagger UI只能展示和测试。你可以理解为Editor是给OpenAPI规范的编写者使用的，而UI是给OpenAPI规范的消费者来使用的。自动生成的代码一般在测试中使用，或者作为API的客户端。客户端都会被直接编译成JAR文件，不需要与源代码打交道。自动生成的服务端代码一般使用较少，因为生成的代码过于繁琐，也难以维护，只有测试时可能会用到。

##### **龙：
> openapi 是一套规范， swagger提供了根据规范文档生成 client、server的代码能力。

