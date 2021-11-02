<p data-nodeid="38643" class="">在第 32 课时中介绍了 REST API 在使用上的局限性。由于请求和响应格式的固定，当 API 的使用者的需求发生改变时，需要 API 提供者先做出修改，API 使用者才能进行所需的改动。这种耦合关系降低了整体的开发效率，对于开放 API 来说，这种情况会更加严重。当 API 的使用者很多时，不同使用者的需求可能会产生冲突。从 API 实现者的角度来说，只能在这些冲突的需求中进行取舍，客观上也造成了部分 API 使用者的困难。Backend For Frontend 模式和 API 版本化可以解决一部分问题，但也使得 API 的维护变得更加复杂。</p>
<p data-nodeid="38644">对于 REST API 的问题，我们需要新的解决方案，<a href="https://graphql.org/" data-nodeid="38751">GraphQL</a> 和 Netflix Falcor 都是可以替代的方案，这两种方案对客户端都提供了更高的要求。REST API 的优势在于对客户端的要求很低，使得它有很强的兼容性，这也是 REST API 流行的一个重要原因。随着 JavaScript 的广泛使用，客户端可以承担更多的职责，这使得 GraphQL 这样的技术有了流行起来的基础。本课时将对 GraphQL 进行基本的介绍，并用 GraphQL 实现乘客管理界面所需的 API。</p>
<h3 data-nodeid="38645">GraphQL</h3>
<p data-nodeid="38646">GraphQL 这个名称的含义是<strong data-nodeid="38759">图查询语言</strong>（Graph Query Language），其中的图与 Netflix Falcor 中的 JSON 图，有着异曲同工之妙。图这种数据结构，表达能力强，适用于各种不同的场景。</p>
<p data-nodeid="38647">GraphQL 是为 API 设计的查询语言，提供了完整的语言来描述 API 所提供的数据的模式（Schema）。模式在 GraphQL 中扮演了重要的作用，类似于 REST API 中的 OpenAPI 规范。有了模式之后，客户端可以方便地查看 API 所提供的查询，以及数据的格式；服务器可以对查询请求进行验证，并根据模式来对查询的执行进行优化。</p>
<p data-nodeid="38648">根据 GraphQL 的模式，客户端发送查询到服务器，服务器验证并执行查询，同时返回相应的结果。查询的结果完全由请求来确定，这就意味着客户端对获取的数据有完全的控制。</p>
<p data-nodeid="38649">GraphQL 使用图来描述实体与实体之间的关系，还可以自动处理实体之间的引用关系。在一个查询中可以包含相互引用的多个实体。</p>
<p data-nodeid="38650">GraphQL 使得 API 的更新变得容易。在 API 的 GraphQL 模式中可以增加新的类型和字段，也可以把已有的字段声明为废弃的。已经废弃的字段不会出现在模式的文档中，可以鼓励使用者使用最新的版本。</p>
<p data-nodeid="38651">GraphQL 非常适用于微服务架构应用的 API 接口，可以充分利用已有微服务的 API。GraphQL 最早由 Facebook 开发，目前有开源的规范和不同平台上的前端和后端的实现，而且已经被 Facebook、GitHub、Pinterest、Airbnb、PayPal、Twitter 等公司采用。</p>
<h4 data-nodeid="38652">查询和修改</h4>
<p data-nodeid="38653">GraphQL 中定义了类型和类型中的字段。在示例应用中，我们可以定义乘客和地址等类型，以及类型中的字段，最简单的查询是选择对象中的字段。如果对象中有嵌套的其他对象，可以同时选择嵌套对象中的字段。</p>
<p data-nodeid="38654">下面是一个 GraphQL 的查询代码示例，其中，passengers 表示查询乘客对象的列表，内嵌的字段 id、name、email 和 mobilePhoneNumber 用来查询乘客对象中的属性；userAddresses 是乘客对象中内嵌的用户地址列表，嵌套的 name 字段用来查询用户地址的名称。</p>
<pre class="lang-java" data-nodeid="38655"><code data-language="java">{
&nbsp; passengers {
&nbsp; &nbsp; id
&nbsp; &nbsp; name
&nbsp; &nbsp; email
&nbsp; &nbsp; mobilePhoneNumber
&nbsp; &nbsp; userAddresses {
&nbsp; &nbsp; &nbsp; name
&nbsp; &nbsp; }
&nbsp; }
}
</code></pre>
<p data-nodeid="38656">该查询的执行结果如下面的代码所示，从中可以看出来，查询结果的格式与查询是完全匹配的。如果从查询中删除掉 passengers 中的 email 和 mobilePhoneNumber，那么对应的查询结果也不会包含这两个字段。</p>
<pre class="lang-json" data-nodeid="38657"><code data-language="json">{
&nbsp; <span class="hljs-attr">"data"</span>: {
&nbsp; &nbsp; <span class="hljs-attr">"passengers"</span>: [
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"id"</span>: <span class="hljs-string">"ae31bb42-540e-4cdc-a088-1bc6e2f9f78d"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"bob"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"email"</span>: <span class="hljs-string">"bob@test.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"mobilePhoneNumber"</span>: <span class="hljs-string">"13400003413"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"userAddresses"</span>: [
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"test"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"new"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; ]
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"id"</span>: <span class="hljs-string">"4d609afe-a193-4c4f-a062-146dd3c6c86b"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"alex"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"email"</span>: <span class="hljs-string">"alex@test.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"mobilePhoneNumber"</span>: <span class="hljs-string">"13455353535"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"userAddresses"</span>: [
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"home"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"office"</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; ]
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; ]
&nbsp; }
}
</code></pre>
<p data-nodeid="38658">在上述的 GraphQL 查询中，我们实际上省略了 query 关键词和查询的名称，query 关键词表示操作的类型。GraphQL 中支持 3 种不同类型的操作，分别是查询（Query）、修改（Mutation）和订阅（Subscription），对应的关键词分别是 query、mutation 和 subscription。操作的名称由客户端提供，作为操作的描述，可以增强查询的可读性。</p>
<p data-nodeid="38659">在操作上可以声明变量，并在执行时提供实际的值，这与编程语言中的函数或方法中的参数是相似的。在下面代码的 GraphQL 查询中，操作的名称是 passengerById，并且有一个类型为 ID 的变量 passengerId，该变量作为字段 passenger 的参数 id 的值。</p>
<pre class="lang-java" data-nodeid="38660"><code data-language="java"><span class="hljs-function">query <span class="hljs-title">passengerById</span><span class="hljs-params">($passengerId: ID!)</span> </span>{
&nbsp; passenger(id: $passengerId) {
&nbsp; &nbsp; name
&nbsp; &nbsp; email
&nbsp; &nbsp; mobilePhoneNumber
&nbsp; }
}
</code></pre>
<p data-nodeid="38661">在执行查询时，需要提供变量的实际值，变量一般以 JSON 的格式传递，如下面的代码所示：</p>
<pre class="lang-json" data-nodeid="38662"><code data-language="json">{
&nbsp; <span class="hljs-attr">"passengerId"</span>: <span class="hljs-string">"ae31bb42-540e-4cdc-a088-1bc6e2f9f78d"</span>
}
</code></pre>
<p data-nodeid="38663">查询的结果如下面的代码所示：</p>
<pre class="lang-json" data-nodeid="38664"><code data-language="json">{
&nbsp; <span class="hljs-attr">"data"</span>: {
&nbsp; &nbsp; <span class="hljs-attr">"passenger"</span>: {
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">"name"</span>: <span class="hljs-string">"bob"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">"email"</span>: <span class="hljs-string">"bob@test.com"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-attr">"mobilePhoneNumber"</span>: <span class="hljs-string">"13400003413"</span>
&nbsp; &nbsp; }
&nbsp; }
}
</code></pre>
<p data-nodeid="38665">在查询时，有些字段的组合可能会重复出现多次。为了复用这些字段的组合，可以使用 GraphQL 中的片段（Fragment）。在下面的代码中，fragment 用来声明片段，on 表示片段对应的对象类型，片段可以直接用在查询中。</p>
<pre class="lang-java" data-nodeid="38666"><code data-language="java"><span class="hljs-function">query <span class="hljs-title">passengerById</span><span class="hljs-params">($passengerId: ID!)</span> </span>{
&nbsp; passenger(id: $passengerId) {
&nbsp; &nbsp; ...passengerFields
&nbsp; }
}
fragment passengerFields on Passenger {
&nbsp; name
&nbsp; email
&nbsp; mobilePhoneNumber
}
</code></pre>
<h4 data-nodeid="38667">模式和类型</h4>
<p data-nodeid="38668">GraphQL 使用语言中性的模式语言来描述数据的结构，每个 GraphQL 服务都通过这个模式语言来定义所开放的数据的类型系统。GraphQL 规范中已经定义了一些内置的类型，每个服务提供者也需要创建自己的类型。</p>
<p data-nodeid="38669">GraphQL 中的类型分成下表中给出的几类，不同的类型使用不同的关键词来创建。</p>
<table data-nodeid="38671">
<thead data-nodeid="38672">
<tr data-nodeid="38673">
<th data-org-content="**分类**" data-nodeid="38675"><strong data-nodeid="38780">分类</strong></th>
<th data-org-content="**关键词**" data-nodeid="38676"><strong data-nodeid="38784">关键词</strong></th>
<th data-org-content="**说明**" data-nodeid="38677"><strong data-nodeid="38788">说明</strong></th>
</tr>
</thead>
<tbody data-nodeid="38681">
<tr data-nodeid="38682">
<td data-org-content="对象类型" data-nodeid="38683">对象类型</td>
<td data-org-content="type" data-nodeid="38684">type</td>
<td data-org-content="服务所提供的对象，对象类型定义了对象中包含的字段及其类型" data-nodeid="38685">服务所提供的对象，对象类型定义了对象中包含的字段及其类型</td>
</tr>
<tr data-nodeid="38686">
<td data-org-content="标量类型" data-nodeid="38687">标量类型</td>
<td data-org-content="scalar" data-nodeid="38688">scalar</td>
<td data-org-content="表示具体的值，不包含字段" data-nodeid="38689">表示具体的值，不包含字段</td>
</tr>
<tr data-nodeid="38690">
<td data-org-content="枚举类型" data-nodeid="38691">枚举类型</td>
<td data-org-content="enum" data-nodeid="38692">enum</td>
<td data-org-content="限定了类型的可选值" data-nodeid="38693">限定了类型的可选值</td>
</tr>
<tr data-nodeid="38694">
<td data-org-content="接口类型" data-nodeid="38695">接口类型</td>
<td data-org-content="interface" data-nodeid="38696">interface</td>
<td data-org-content="定义了对象类型中必须包含的字段" data-nodeid="38697">定义了对象类型中必须包含的字段</td>
</tr>
<tr data-nodeid="38698">
<td data-org-content="联合类型" data-nodeid="38699">联合类型</td>
<td data-org-content="union" data-nodeid="38700">union</td>
<td data-org-content="多个具体类型的联合" data-nodeid="38701">多个具体类型的联合</td>
</tr>
<tr data-nodeid="38702">
<td data-org-content="输入类型" data-nodeid="38703">输入类型</td>
<td data-org-content="input" data-nodeid="38704">input</td>
<td data-org-content="作为参数传递的复杂对象" data-nodeid="38705">作为参数传递的复杂对象</td>
</tr>
</tbody>
</table>
<p data-nodeid="38706">GraphQL 中最基本的类型是对象类型以及其中包含的字段。对象类型通常表示 API 中的不同实体，其中的字段则与实体中的属性相对应。每个字段需要声明名称和类型，字段的类型可以是标量类型、枚举类型或是其他自定义的类型。</p>
<p data-nodeid="38707">GraphQL 中提供了内置的标量类型 Int、Float、String、Boolean 和 ID，也允许不同的实现提供自定义的标量类型。ID 表示唯一的标识符，在使用上类似 String。</p>
<p data-nodeid="38708">GraphQL 中的枚举类型与 Java 中的枚举类型是相似的。下面的代码给出了枚举类型的示例。</p>
<pre class="lang-java" data-nodeid="38709"><code data-language="java"><span class="hljs-keyword">enum</span> TrafficColor {
    RED
    GREEN
    YELLOW
}
</code></pre>
<p data-nodeid="38710">除了对象、标量和枚举类型之外，还可以通过感叹号来声明<strong data-nodeid="38817">非空（Non-Null）类型</strong>，如 String! 表示值不能为 null 的 String 类型。非空类型可以用在字段中声明该字段的值不可能为 null，也可以用在参数声明中，用来声明该参数的实际值不能为 null。</p>
<p data-nodeid="38711">当以方括号来封装某个类型时，就得到了该类型的列表形式，如 [String] 表示 String 列表，而 [String!] 表示元素为非空 String 的列表。</p>
<p data-nodeid="38712">GraphQL 中的接口与 Java 中的接口作用类似，用来声明不同对象类型所共有的字段。联合类型则把多个具体的对象类型组合在一起，其值可以是任何一个对象类型。对于一个联合类型的对象，可以使用 __typename 字段来查看其实际的类型，在查询时，可以使用内联片段来根据不同的类型，选择相应的字段。</p>
<p data-nodeid="38713">GraphQL 中的参数可以使用复杂对象，这些对象的类型通过输入类型来声明，如下面的代码所示：</p>
<pre class="lang-js" data-nodeid="38714"><code data-language="js">input&nbsp;CreateUserAddressRequest&nbsp;{
    <span class="hljs-attr">name</span>:&nbsp;<span class="hljs-built_in">String</span>!,
    addressId:&nbsp;ID!
}
</code></pre>
<h4 data-nodeid="38715">查询执行</h4>
<p data-nodeid="38716">当 GraphQL 的查询发送到服务器时，由服务器负责查询的执行，查询的执行结果的结构与查询本身的结构相匹配。查询在执行时需要依靠类型系统的支持。GraphQL 查询中的每个字段都可以看成是它类型上的一个函数或方法，该函数或方法会返回一个新的类型。</p>
<p data-nodeid="38717">每个类型的每个字段，在服务器上都有一个函数与之对应，称为<strong data-nodeid="38840">解析器（Resolver）</strong>。当需要查询某个字段时，这个字段对应的解析器会被调用，从而返回下一个需要处理的值，这个过程会递归下去，直到解析器返回的是标量类型的值。GraphQL 的查询过程，总是以标量值作为结束点。</p>
<p data-nodeid="38718">如果字段本来就是对象中的属性，那么获取这些字段的解析器的实现非常简单，并不需要开发人员显式提供。大部分的 GraphQL 服务器的实现库，都提供了对这种解析器的支持。如果一个字段没有对应的解析器，则默认为读取对象中同样名称的属性值。</p>
<h3 data-nodeid="38719">实现 GraphQL 服务</h3>
<p data-nodeid="38720">下面介绍如何使用 GraphQL 来实现乘客管理界面的 API。后台实现使用的是 Java 语言，基于 GraphQL 的 Java 实现库 <a href="https://www.graphql-java.com/" data-nodeid="38846">graphql-java</a>，以及相应的 Spring Boot 集成库 <a href="https://github.com/graphql-java-kickstart/graphql-spring-boot" data-nodeid="38850">graphql-spring-boot</a>。在实际的数据获取时，使用的是不同微服务 API 的 Java 客户端。完整的实现请参考 GitHub 上源代码中示例应用的 happyride-passenger-web-api-graphql 模块。下图是 GraphQL 服务的架构示意图。</p>
<p data-nodeid="38721"><img src="https://s0.lgstatic.com/i/image/M00/2E/CF/Ciqc1F8Fpv-AZk1NAABk0f47qTs188.png" alt="image (5).png" data-nodeid="38854"></p>
<h4 data-nodeid="38722">模式</h4>
<p data-nodeid="38723">在实现之前，需要先定义服务的 GraphQL 模式，下面的代码是 API 的 GraphQL 模式文件的内容。在模式中，首先定义了乘客、用户地址、地址和区域等 4 个对象类型，对应于 API 所提供的数据中的实体；接下来的 Query 类型中定义了 API 所提供的查询操作，包括查询乘客列表、查询单个乘客和地址，以及搜索地址；最后的 Mutation 类型中定义了 API 所提供的修改操作，即添加新的用户地址。Query 和 Mutation 类型中定义的字段，是整个 GraphQL 服务的入口。</p>
<pre class="lang-js te-preview-highlight" data-nodeid="44920"><code data-language="js">type&nbsp;Passenger&nbsp;{
    <span class="hljs-attr">id</span>:&nbsp;ID!
    name:&nbsp;<span class="hljs-built_in">String</span>!
    email:&nbsp;<span class="hljs-built_in">String</span>!
    mobilePhoneNumber:&nbsp;<span class="hljs-built_in">String</span>
    <span class="hljs-attr">userAddresses</span>:&nbsp;[UserAddress!]
}
type&nbsp;UserAddress&nbsp;{
    <span class="hljs-attr">id</span>:&nbsp;ID!
    name:&nbsp;<span class="hljs-built_in">String</span>!
    address:&nbsp;Address!
}
type&nbsp;Address&nbsp;{
    <span class="hljs-attr">id</span>:&nbsp;ID!
    areaId:&nbsp;Int!
    addressLine:&nbsp;<span class="hljs-built_in">String</span>!
    lat:&nbsp;Float!
    lng:&nbsp;Float!
    areas:&nbsp;[Area!]
}
type&nbsp;Area&nbsp;{
    <span class="hljs-attr">id</span>:&nbsp;Int!
    level:&nbsp;Int!
    parentCode:&nbsp;Int!
    areaCode:&nbsp;Int!
    zipCode:&nbsp;<span class="hljs-built_in">String</span>!
    cityCode:&nbsp;<span class="hljs-built_in">String</span>!
    name:&nbsp;<span class="hljs-built_in">String</span>!
    shortName:&nbsp;<span class="hljs-built_in">String</span>!
    mergerName:&nbsp;<span class="hljs-built_in">String</span>!
    pinyin:&nbsp;<span class="hljs-built_in">String</span>!
    lat:&nbsp;Float!
    lng:&nbsp;Float!
    ancestors:&nbsp;[Area!]
}
type&nbsp;Query&nbsp;{
    passengers(page:&nbsp;Int&nbsp;=&nbsp;<span class="hljs-number">0</span>,&nbsp;<span class="hljs-attr">size</span>:&nbsp;Int&nbsp;=&nbsp;<span class="hljs-number">10</span>):&nbsp;[Passenger]
    passenger(id:&nbsp;ID!):&nbsp;Passenger
    address(id:&nbsp;ID!,&nbsp;areaLevel:&nbsp;Int&nbsp;=&nbsp;<span class="hljs-number">0</span>):&nbsp;Address
    searchAddress(areaCode:&nbsp;<span class="hljs-built_in">String</span>!,&nbsp;query:&nbsp;<span class="hljs-built_in">String</span>!):&nbsp;[Address]
}
input&nbsp;CreateUserAddressRequest&nbsp;{
    <span class="hljs-attr">name</span>:&nbsp;<span class="hljs-built_in">String</span>!,
    addressId:&nbsp;ID!
}
type&nbsp;Mutation&nbsp;{
    addUserAddress(passengerId:&nbsp;ID!,&nbsp;request:&nbsp;CreateUserAddressRequest!):&nbsp;Passenger
}
</code></pre>












<h4 data-nodeid="38725">查询</h4>
<p data-nodeid="38726">为了实现查询操作，需要提供 Query 类型的解析器。下面代码中的 Query 类实现了 GraphQLQueryResolver 接口，其中的每个方法都对应查询模式中的一个字段。以获取乘客列表的 passengers 方法为例，它的参数 page 和 size 对应于字段的同名参数，而返回值类型 List 则与字段的类型 [Passenger] 相对应。在 passengers 方法的实现中，使用了乘客管理服务提供的 Java 客户端 PassengerApi 对象来调用 API 并获取结果。Query 类中的其他方法的实现，也采用类似的方式，与查询模式中的其他字段相对应。<br>
<br></p>
<pre class="lang-java" data-nodeid="38727"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Query</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">GraphQLQueryResolver</span>&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;PassengerApi&nbsp;passengerApi;
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;AddressApi&nbsp;addressApi;
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;List&lt;Passenger&gt;&nbsp;<span class="hljs-title">passengers</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;page,&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;size)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;ApiException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">this</span>.passengerApi.listPassengers(page,&nbsp;size).stream()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.map(ServiceApiHelper::fromPassengerVO)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.collect(Collectors.toList());
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;Passenger&nbsp;<span class="hljs-title">passenger</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;String&nbsp;id)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;ApiException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;Optional.ofNullable(<span class="hljs-keyword">this</span>.passengerApi.getPassenger(id))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.map(ServiceApiHelper::fromPassengerVO)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.orElse(<span class="hljs-keyword">null</span>);
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;AddressVO&nbsp;<span class="hljs-title">address</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;String&nbsp;id,&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;areaLevel)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;io.vividcode.happyride.addressservice.client.ApiException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">this</span>.addressApi.getAddress(id,&nbsp;areaLevel);
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;List&lt;AddressVO&gt;&nbsp;<span class="hljs-title">searchAddress</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;String&nbsp;areaCode,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;String&nbsp;query)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;io.vividcode.happyride.addressservice.client.ApiException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;<span class="hljs-keyword">this</span>.addressApi.searchAddress(Long.valueOf(areaCode),&nbsp;query);
&nbsp;&nbsp;}
}
</code></pre>
<h4 data-nodeid="38728">修改</h4>
<p data-nodeid="38729">对于模式中的修改操作，需要提供 Mutation 类型的解析器。下面代码中的 Mutation 类实现了 GraphQLMutationResolver 接口，其中的 addUserAddress 方法对应于修改模式中的同名字段。</p>
<pre class="lang-java" data-nodeid="38730"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">Mutation</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">GraphQLMutationResolver</span>&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;PassengerApi&nbsp;passengerApi;
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;Passenger&nbsp;<span class="hljs-title">addUserAddress</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;String&nbsp;passengerId,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;CreateUserAddressRequest&nbsp;request)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">throws</span>&nbsp;ApiException&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;ServiceApiHelper
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.fromPassengerVO(<span class="hljs-keyword">this</span>.passengerApi.createAddress(passengerId,&nbsp;request));
&nbsp;&nbsp;}
}
</code></pre>
<h4 data-nodeid="38731">自定义解析器</h4>
<p data-nodeid="38732">在查询模式的解析器中，字段对应的方法直接返回了 Passenger 和 AddressVO 对象。对于 GraphQL模式中的 Passenger 和 Address 类型中的字段，如果对应的 Java 对象中有同名的属性，那么 GraphQL 服务器可以自动进行解析。比如，Passenger 类型中的 name 和 email 字段，会直接解析成对应的 Java 中的 Passenger 对象中的 name 和 email 属性。</p>
<p data-nodeid="38733">在 GraphQL 模式中，UserAddress 类型中的 address 字段的类型是 Address，而乘客管理服务 API 只提供了地址的标识符，需要调用地址管理服务的 API 才能获取到实际的地址信息。在这种情况下，需要使用自定义的解析器来获取 address 字段的值。</p>
<p data-nodeid="38734">下面的代码是 Address 对象类型对应的 Java 类，其中的 getAddress 方法用来解析 address 字段，该方法的 DataFetchingEnvironment 类型的参数表示的是获取数据时的上下文环境。当 GraphQL 服务器执行该字段的查询时，会提供 DataFetchingEnvironment 接口的实现对象。</p>
<p data-nodeid="38735">DataFetchingEnvironment 接口的 getContext 方法可以获取到与本次查询相关的上下文对象。从该上下文对象中获取到包含了 DataLoader 对象的 DataLoaderRegistry 对象，再查找到对应的 DataLoader 对象来进行实际的地址获取操作。DataLoader 是 GraphQL 服务器实现中获取数据的通用接口。需要注意的是，getAddress 方法返回的是 CompletableFuture<code data-backticks="1" data-nodeid="38872">&lt;AddressVO&gt;</code>对象，表示这是一个异步获取操作。</p>
<pre class="lang-java" data-nodeid="38736"><code data-language="java"><span class="hljs-meta">@Data</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">UserAddress</span>&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;String&nbsp;id;
&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;String&nbsp;name;
&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;String&nbsp;addressId;
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;CompletableFuture&lt;AddressVO&gt;&nbsp;<span class="hljs-title">getAddress</span><span class="hljs-params">(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;DataFetchingEnvironment&nbsp;environment)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;GraphQLContext&nbsp;context&nbsp;=&nbsp;environment.getContext();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;context.getDataLoaderRegistry()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.map(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;registry&nbsp;-&gt;&nbsp;registry.
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&lt;String,&nbsp;AddressVO&gt;getDataLoader(USER_ADDRESS_DATA_LOADER)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.load(<span class="hljs-keyword">this</span>.addressId))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.orElse(CompletableFuture.completedFuture(<span class="hljs-keyword">null</span>));
&nbsp;&nbsp;}
</code></pre>
<p data-nodeid="38737">下面代码中的 UserAddressLoader 类是获取地址操作的 BatchLoader 接口的实现。数据加载是异步完成的，同时也是批量进行的。这里通过 AddressApi 的异步调用方法 getAddressesAsync 来调用 API。</p>
<pre class="lang-java" data-nodeid="38738"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">UserAddressLoader</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">BatchLoader</span>&lt;<span class="hljs-title">String</span>,&nbsp;<span class="hljs-title">AddressVO</span>&gt;&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;AddressApi&nbsp;addressApi;
&nbsp;&nbsp;<span class="hljs-meta">@SneakyThrows(ApiException.class)</span>
&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;<span class="hljs-keyword">public</span>&nbsp;CompletionStage&lt;List&lt;AddressVO&gt;&gt;&nbsp;load(<span class="hljs-keyword">final</span>&nbsp;List&lt;String&gt;&nbsp;keys)&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;CompletableFuture&lt;List&lt;AddressVO&gt;&gt;&nbsp;future&nbsp;=&nbsp;<span class="hljs-keyword">new</span>&nbsp;CompletableFuture&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">this</span>.addressApi.getAddressesAsync(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;AddressBatchRequest(keys),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">new</span>&nbsp;ApiCallback&lt;List&lt;AddressVO&gt;&gt;()&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">onFailure</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;ApiException&nbsp;e,&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;statusCode,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;Map&lt;String,&nbsp;List&lt;String&gt;&gt;&nbsp;responseHeaders)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;future.completeExceptionally(e);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">onSuccess</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;List&lt;AddressVO&gt;&nbsp;result,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">int</span>&nbsp;statusCode,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;Map&lt;String,&nbsp;List&lt;String&gt;&gt;&nbsp;responseHeaders)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;future.complete(result);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">onUploadProgress</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;bytesWritten,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;contentLength,&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;done)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-keyword">void</span>&nbsp;<span class="hljs-title">onDownloadProgress</span><span class="hljs-params">(<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;bytesRead,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">long</span>&nbsp;contentLength,&nbsp;<span class="hljs-keyword">final</span>&nbsp;<span class="hljs-keyword">boolean</span>&nbsp;done)</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;future;
&nbsp;&nbsp;}
}
</code></pre>
<h4 data-nodeid="38739">Spring 配置</h4>
<p data-nodeid="38740">为了启动 GraphQL 服务，需要通过 Spring 的配置来创建 GraphQLSchema 类型的 bean，如下面的代码所示。首先使用 SchemaParser 来解析 GraphQL 模式文件，然后设置查询和修改的两个解析器，最后创建出 GraphQLSchema 对象。其他的相关工作，由 Spring Boot 的自动配置功能来完成。</p>
<pre class="lang-java" data-nodeid="38741"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">SchemaConfig</span>&nbsp;</span>{
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;Query&nbsp;query;
&nbsp;&nbsp;<span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;Mutation&nbsp;mutation;
&nbsp;&nbsp;<span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;GraphQLSchema&nbsp;<span class="hljs-title">graphQLSchema</span><span class="hljs-params">()</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span>&nbsp;SchemaParser.newParser()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.file(<span class="hljs-string">"passenger-api.graphqls"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.resolvers(<span class="hljs-keyword">this</span>.query,&nbsp;<span class="hljs-keyword">this</span>.mutation)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.build()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.makeExecutableSchema();
&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="38742">在启动 Spring Boot 应用之后，通过路径 /graphql 可以访问 GraphQL 服务。除此之外，如果添加了对 GraphQL 的工具 GraphiQL 的依赖，还可以通过路径 /graphiql 来访问该工具。很多工具都提供了对 GraphQL 的支持，可以在开发中使用，包括 Postman 和 Insomnia。</p>
<p data-nodeid="38743">下图是使用 Insomnia 查询 GraphQL 时的截图。</p>
<p data-nodeid="38744"><img src="https://s0.lgstatic.com/i/image/M00/2E/CF/Ciqc1F8Fp3yAHFszAAMallAEM2s704.png" alt="insomnia.png" data-nodeid="38881"></p>
<h3 data-nodeid="38745">总结</h3>
<p data-nodeid="38746">作为一个新的开放 API 的方式，GraphQL 释放了客户端的查询能力，已经得到了广泛的流行。通过本课时的学习，你可以了解 GraphQL 中的基本概念，包括查询和修改，以及如何编写 GraphQL 模式，还可以掌握如何使用 Java 来实现基于 Spring Boot 的 GraphQL 服务器。</p>

---

### 精选评论

##### **舒：
> 程序包org.openapitools.client.model不存在，这个怎么处理啊

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 能具体说明一些在哪里出现的错误吗？我查了一下，这个包是OpenAPI代码生成工具所默认使用的模型包的名称，在实例的代码中并没有用到。如果是你自己的代码，可以通过Maven插件的 modelPackage 选项来设置一个自定义的包名。

##### *军：
> graphql和falcor如果两选一时如何选？基于什么选择？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一般优先选择GraphQL，Falcor是Netflix自己的产品，使用的范围比较窄；GraphQL有自己的规范，还有大量的第三方库的支持，社区和文档也非常齐全，长期来看是更值得投资的技术。

