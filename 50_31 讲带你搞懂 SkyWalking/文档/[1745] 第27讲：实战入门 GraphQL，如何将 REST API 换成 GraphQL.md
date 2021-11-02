<p data-nodeid="106088" class="">从本节开始将深入介绍 query-graphql-plugin 插件，我们会启动 SkyWalking Rocketbot 来查询 Trace 数据和 JVM 监控数据，这些用户查询请求最终都会路由到 query-graphql-plugin 插件中进行处理。</p>
<h3 data-nodeid="106089">GraphQL 简介</h3>
<p data-nodeid="106090">GraphQL 是一个用于 API 的查询语言，是一个使用基于类型系统来执行查询的服务端运行时。GraphQL 并没有和任何特定数据库或者存储引擎绑定。GraphQL 对服务端 API 中的数据提供了一套易于理解的完整描述，使得客户端能够准确地获得它需要的数据，而且没有任何冗余，也让 API 更容易地随着时间推移而演进，还能用于构建强大的开发者工具。使用 GraphQL 开发 API 有如下好处。</p>
<ul data-nodeid="106091">
<li data-nodeid="106092">
<p data-nodeid="106093">可描述：使用 GraphQL，你获取的都是你想要的数据，不多也不会少；</p>
</li>
<li data-nodeid="106094">
<p data-nodeid="106095">分级：GraphQL 天然遵循了对象间的关系，通过一个简单的请求，我们可以获取到一个对象及其相关的对象。</p>
</li>
<li data-nodeid="106096">
<p data-nodeid="106097">强类型：使用 GraphQL 的类型系统，能够清晰、准确的描述数据，这样就能确保从服务器获取的数据和我们查询的一致。</p>
</li>
<li data-nodeid="106098">
<p data-nodeid="106099">跨语言：GraphQL 并不绑定于某一特定的语言。</p>
</li>
<li data-nodeid="106100">
<p data-nodeid="106101">兼容性：GraphQL 不限于某一特定存储平台，GraphQL 可以方便地接入已有的存储、代码、甚至可以连接第三方的 API。</p>
</li>
</ul>
<h3 data-nodeid="106102">GraphQL&nbsp;&nbsp;类型系统</h3>
<p data-nodeid="106103">在一篇文章中完整介绍 GraphQL 本身，以及如何在 Java 服务端使用 GraphQL 几乎是不可能的，其中会涉及很多琐碎的细节需要说明，并且还需要列举一些示例。本课时并不是一篇完整的 GraphQL Java 教程，这里重点介绍 query-graphql-plugin 插件涉及的 GraphQL 知识点。</p>
<p data-nodeid="106104">GraphQL 的类型系统与 Java 的类型系统非常相似，主要有下面 6 种类型：</p>
<ol data-nodeid="106105">
<li data-nodeid="106106">
<p data-nodeid="106107">Scalar，类似于 Java 中的基本变量。</p>
</li>
</ol>
<p data-nodeid="106108"><img src="https://s0.lgstatic.com/i/image/M00/21/B6/CgqCHl7q_OOAPGBqAAPjNVHf5Xk878.png" alt="Drawing 0.png" data-nodeid="106208"></p>
<ol start="2" data-nodeid="106109">
<li data-nodeid="106110">
<p data-nodeid="106111">Object，类似于 Java 中的对象，这里使用 GraphQL 定义一个 Book 类型，它就是 Object 类型，在 GraphQL Java 中对应的是 GraphQLObjectType。</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="106112"><code data-language="java">type Book {
    id: ID # 编号
    name: String #书名
    pageCount: Int #页数
    author: Author # 作者
}
</code></pre>
<ol start="3" data-nodeid="106113">
<li data-nodeid="106114">
<p data-nodeid="106115">Interface，类似于 Java 中的接口，在下面中的示例如下：</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="106116"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ComicCharacter</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;String;
}
</code></pre>
<p data-nodeid="106117">在 GraphQL Java 中对应的类型是 GraphQLInterfaceType。</p>
<ol start="4" data-nodeid="106118">
<li data-nodeid="106119">
<p data-nodeid="106120">Union，在 Java 中没有 Union 类型，但是在 C++ 中有 Union 类型，在 GraphQL Schema 中的示例如下：</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="106121"><code data-language="java">type Cat {
    name: String;
    lives: Int;
}
type Dog {
    name: String;
    bonesOwned: <span class="hljs-keyword">int</span>;
}
union Pet = Cat | Dog
</code></pre>
<p data-nodeid="106122">在 GraphQL Java 中对应的类型是 GraphQLUnionType。在使用 Interface 或 Union 时，如果需要获取对象真实类型，可以通过 TypeResolver 进行判断。</p>
<ol start="5" data-nodeid="106123">
<li data-nodeid="106124">
<p data-nodeid="106125">InputObject，主要用于封装方法参数，GraphQL Schema 中的定义与 Object 类似，主要区别是将 type 关键字换成 input 关键字。GraphQL Java 中对应的类型是 GraphQLInputObjectType。</p>
</li>
<li data-nodeid="106126">
<p data-nodeid="106127">Enum，类似于 Java 中的枚举，不再赘述。</p>
</li>
</ol>
<h3 data-nodeid="106128">GraphQL Java&nbsp;基础入门</h3>
<p data-nodeid="106129">SkyWalking OAP 中的 query-graphql-plugin 插件也使用了 GraphQL 开发其 API，这里就简单介绍如何在 Java 中使用 GraphQL。</p>
<h4 data-nodeid="106130">定义 GraphQL Schema</h4>
<p data-nodeid="106131">首先，我们需要定义一套 GraphQL Schema，似于要使用数据库存储数据之前，需要先建表，之后上层应用才能读写数据库。GraphQL Java 服务端要是响应用户的请求，也需要定义一个描述数据的结构，也就是这里的 GraphQL Schema。一般我们会将 GraphQL Schema 单独放到 resources 目录下，这里以图书信息管理的为例，resources/book.graphql 文件如下所示，其中定义了 Book、Author、Query 三个类型，其中 Book 和 Author 类似于普通的 JavaBean，Query 则类似于 Java 中的接口定义：</p>
<pre class="lang-java" data-nodeid="106132"><code data-language="java">type Book {
    id: ID # 编号
    name: String #书名
    pageCount: Int #页数
    author: Author # 作者
}
type Author {
  id: ID # 作者编号
  firstName: String # 作者姓名
  lastName: String
}
type QueryBook {
    # getById()类似于Java方法，根据Id查询书籍信息
    # id是方法参数，ID是类型，"!"表示非空
    # Book是返回值类型，这里返回的是一个Book对象
    getById(id: ID!): Book
    # 查询Book列表
    list: [Book]
}
</code></pre>
<h4 data-nodeid="106133">加载 GraphQL Schema 文件</h4>
<p data-nodeid="106134">在 GraphQL Java 中加载并解析 GraphQL Schema 文件的方式如下：</p>
<pre class="lang-java" data-nodeid="106135"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">GraphQLProvider</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;GraphQL&nbsp;graphQL;
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> GraphQL <span class="hljs-title">graphQL</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> graphQL;
    }
    <span class="hljs-meta">@PostConstruct</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> IOException </span>{
        <span class="hljs-comment">// 读取 GraphQL Schema文件并创建 GraphQL实例，</span>
        <span class="hljs-comment">// 该GraphQL实例会通过上面的 graphQL()方法暴露给Spring，</span>
        <span class="hljs-comment">// 默认情况下，请求到"/graphql"这个path上的请求都会由该GraphQL实例处理</span>
        URL url = Resources.getResource(<span class="hljs-string">"book.graphqls"</span>);
        String sdl = Resources.toString(url, Charsets.UTF_8);
        GraphQLSchema graphQLSchema = buildSchema(sdl);
        <span class="hljs-keyword">this</span>.graphQL = GraphQL.newGraphQL(graphQLSchema).build();
    }
    <span class="hljs-function"><span class="hljs-keyword">private</span> GraphQLSchema <span class="hljs-title">buildSchema</span><span class="hljs-params">(String sdl)</span> </span>{
        <span class="hljs-comment">// GraphQL Schema文件被解析之后，就是这里的 TypeDefinitionRegistry对象</span>
        TypeDefinitionRegistry typeRegistry = <span class="hljs-keyword">new</span> SchemaParser().parse(sdl);
        <span class="hljs-comment">// 注册DataFetcher，DataFetcher的介绍以及buildWiring()方法实现在后面马上会进行介绍</span>
        RuntimeWiring runtimeWiring = buildWiring();
        SchemaGenerator schemaGenerator = <span class="hljs-keyword">new</span> SchemaGenerator();
        <span class="hljs-comment">// 将GraphQL Schema中定义的与 DataFetcher关联起来</span>
        <span class="hljs-keyword">return</span> schemaGenerator.makeExecutableSchema(typeRegistry, runtimeWiring);
    }
    <span class="hljs-comment">// 这哪是省略buildWiring()方法</span>
}
</code></pre>
<p data-nodeid="106136">这里我们接触到了几个新类型。</p>
<ul data-nodeid="106137">
<li data-nodeid="106138">
<p data-nodeid="106139">GraphQL：它默认将处理"/graphql"这个 path 上的全部请求；</p>
</li>
<li data-nodeid="106140">
<p data-nodeid="106141">TypeDefinitionRegistry：它是 GraphQL Schema 文件的解析结果；</p>
</li>
<li data-nodeid="106142">
<p data-nodeid="106143">SchemaGenerator: 它关联了 TypeDefinitionRegistry 对象和后面要介绍的 DataFetcher 对象，并生成一个 GraphQLSchema 对象；</p>
</li>
<li data-nodeid="106144">
<p data-nodeid="106145">RuntimeWiring：后面介绍的 DataFetcher 对象将注册到 RuntimeWiring 中，具体的注册方式在 buildWiring() 方法中（后面分析）。</p>
</li>
</ul>
<h4 data-nodeid="106146">关联&nbsp;DataFetcher</h4>
<p data-nodeid="106147">DataFetcher 是在 GraphQL Java 服务端中比较重要的概念之一。DataFetcher 的核心功能是：获取查询字段的相应数据。DataFetcher 接口的实现如下：</p>
<pre class="lang-java" data-nodeid="106148"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DataFetcher</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">// DataFetchingEnvironment中记录了很多信息，例如：</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;该&nbsp;DataFetcher对应的字段以及类型、查询的外层对象以及根对象、当前上下文信息等等一系列信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">T&nbsp;<span class="hljs-title">get</span><span class="hljs-params">(DataFetchingEnvironment&nbsp;dataFetchingEnvironment)</span>&nbsp;<span class="hljs-keyword">throws</span>&nbsp;Exception</span>;
}
</code></pre>
<p data-nodeid="106149">在 GraphQLProvider.buildWiring() 方法中，我们为 Query.getById 方法与 Book.author 字段绑定了自定义的 DataFetcher 实现，具体实现如下：</p>
<pre class="lang-java" data-nodeid="106150"><code data-language="java"><span class="hljs-meta">@Autowired</span>
GraphQLDataFetchers&nbsp;graphQLDataFetchers;
<span class="hljs-function"><span class="hljs-keyword">private</span> RuntimeWiring <span class="hljs-title">buildWiring</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> RuntimeWiring.newRuntimeWiring()
            <span class="hljs-comment">// 将Query.getById与getBookByIdDataFetcher()方法返回的DataFetcher实现关联</span>
            .type(newTypeWiring(<span class="hljs-string">"Query"</span>).dataFetcher(<span class="hljs-string">"getById"</span>, 
                            graphQLDataFetchers.getBookByIdDataFetcher())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.dataFetcher(<span class="hljs-string">"list"</span>,&nbsp;graphQLDataFetchers.listDataFetcher()))
            <span class="hljs-comment">// 将Book.author字段与getBookByIdDataFetcher()方法返回的DataFetcher实现关联</span>
            .type(newTypeWiring(<span class="hljs-string">"Book"</span>).dataFetcher(<span class="hljs-string">"author"</span>, 
                            graphQLDataFetchers.getAuthorDataFetcher()))
            .build();
}
</code></pre>
<p data-nodeid="106151">GraphQLDataFetchers 的定义如下，GraphQLDataFetchers 中定义了 books 和 authors 两个集合来模拟数据源，getBookByIdDataFetcher() 方法和 getAuthorDataFetcher() 方法返回的自定义 DataFetcher 实现会分别查询这两个集合返回相应数据：</p>
<pre class="lang-java" data-nodeid="106152"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">GraphQLDataFetchers</span> </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> List&lt;ImmutableMap&lt;String, String&gt;&gt; books = Arrays.asList(
            ImmutableMap.of(<span class="hljs-string">"id"</span>, <span class="hljs-string">"book-1"</span>,<span class="hljs-string">"name"</span>, <span class="hljs-string">"Harry Potter and the Philosopher's Stone"</span>,<span class="hljs-string">"pageCount"</span>, <span class="hljs-string">"223"</span>,<span class="hljs-string">"authorId"</span>, <span class="hljs-string">"author-1"</span>),
    );
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> List&lt;ImmutableMap&lt;String, String&gt;&gt; authors = Arrays.asList(
            ImmutableMap.of(<span class="hljs-string">"id"</span>, <span class="hljs-string">"author-1"</span>,<span class="hljs-string">"firstName"</span>, <span class="hljs-string">"Joanne"</span>,<span class="hljs-string">"lastName"</span>, <span class="hljs-string">"Rowling"</span>),
    );
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;DataFetcher&nbsp;<span class="hljs-title">getBookByIdDataFetcher</span><span class="hljs-params">()</span>&nbsp;</span>{
        <span class="hljs-keyword">return</span> dataFetchingEnvironment -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;获取&nbsp;id参数，然后根据id查找&nbsp;books集合并返回相应的 Book信息</span>
            String bookId = dataFetchingEnvironment.getArgument(<span class="hljs-string">"id"</span>);
            <span class="hljs-keyword">return</span> books.stream().filter(book -&gt; book.get(<span class="hljs-string">"id"</span>).equals(bookId))
                    .findFirst().orElse(<span class="hljs-keyword">null</span>);
        };
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> DataFetcher <span class="hljs-title">getAuthorDataFetcher</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> dataFetchingEnvironment -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;DataFetcher&nbsp;会按照&nbsp;GraphQL&nbsp;Schema定义从外层向内层调用</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;这里可以直接通过&nbsp;DataFetchingEnvironment获取外层&nbsp;DataFetcher查找到的数据(即关联的Book）</span>
            Map&lt;String, String&gt; book = dataFetchingEnvironment.getSource();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;String&nbsp;authorId&nbsp;=&nbsp;book.get(<span class="hljs-string">"authorId"</span>);&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;根据&nbsp;authorId查找作者信息</span>
            <span class="hljs-keyword">return</span> authors.stream().filter(author -&gt; author.get(<span class="hljs-string">"id"</span>).equals(authorId))
                    .findFirst().orElse(<span class="hljs-keyword">null</span>);
        };
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> DataFetcher <span class="hljs-title">listDataFetcher</span><span class="hljs-params">()</span> </span>{
       <span class="hljs-keyword">return</span> dataFetchingEnvironment -&gt; books;
    }
}
</code></pre>
<p data-nodeid="106153">GraphQL Schema 中的每个字段都会关联一个 DataFetcher，如果未通过 RuntimeWiring 方式明确关联的自定义 DataFetcher ，会默认关联 PropertyDataFetcher。PropertyDataFetcher 会有多种方式从外层结果中为关联字段查找正确的值：</p>
<ol data-nodeid="106154">
<li data-nodeid="106155">
<p data-nodeid="106156">如果外层查找结果为&nbsp;null，则直接返回 null，否则执行步骤 2；</p>
</li>
<li data-nodeid="106157">
<p data-nodeid="106158">通过&nbsp;function 从外层查询结果中提起对应字段值，该 function 是在&nbsp;PropertyDataFetcher&nbsp;初始化时指定，若未指定&nbsp;function 则执行步骤 3；</p>
</li>
<li data-nodeid="106159">
<p data-nodeid="106160">如果外层查询结果为 Map，则从该 Map 中直接获取字段值；</p>
</li>
<li data-nodeid="106161">
<p data-nodeid="106162">如果外层查询结果是 Java 对象，则调用相应的 getter 方法获取字段值。</p>
</li>
</ol>
<p data-nodeid="106163">最后我们回顾整个示例项目，其核心逻辑如下图：</p>
<p data-nodeid="106164"><img src="https://s0.lgstatic.com/i/image/M00/21/AB/Ciqc1F7q_W-AJHWvAALYJGT3C0E099.png" alt="Drawing 1.png" data-nodeid="106243"></p>
<h4 data-nodeid="106165">启动</h4>
<p data-nodeid="106166">启动该 Spring 项目之后，可以使用 GraphQL Playground 这个工具访问"/graphql"，并传入查询 Book 的请求，如下图所示：</p>
<p data-nodeid="106167"><img src="https://s0.lgstatic.com/i/image/M00/21/AB/Ciqc1F7q_b2ASlkPAAG_Pnp7t9o721.png" alt="Drawing 3.png" data-nodeid="106252"></p>
<p data-nodeid="106168">如果想查看 Http 请求，可以点击 COPY CURL 按钮获取相应的 curl 命令，如下所示：</p>
<pre class="lang-java" data-nodeid="106169"><code data-language="java">curl&nbsp;<span class="hljs-string">'http://localhost:8080/graphql'</span>&nbsp;-H&nbsp;<span class="hljs-string">'Content-Type:&nbsp;application/json'</span>&nbsp;--data-binary&nbsp;<span class="hljs-string">'{"query":"\n{\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;getById(id:\"book-1\")&nbsp;{\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;id\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pageCount\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;author{\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\tfirstName\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\tlastName\n&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}\n&nbsp;&nbsp;&nbsp;&nbsp;}\n}"}'</span>&nbsp;--compressed
</code></pre>
<p data-nodeid="106170">list 请求如下图所示：</p>
<p data-nodeid="106171"><img src="https://s0.lgstatic.com/i/image/M00/21/AB/Ciqc1F7q_ciALYuxAAKZFgwrgvo996.png" alt="Drawing 4.png" data-nodeid="106257"></p>
<p data-nodeid="106172">到此为止，GraphQL Java&nbsp;Demo 项目的涉及的基础知识就全部介绍完了。</p>
<h3 data-nodeid="106173">GraphQL Java Tools&nbsp;入门</h3>
<p data-nodeid="106174">在使用 GraphQL Java&nbsp;开发服务端 API 的时候，需要手写前文介绍的 DataFetcher 实现、GraphQLSchema 解析逻辑以及 GraphQL 对象的实例化过程。</p>
<p data-nodeid="106175">GraphQL Java Tools&nbsp;可以帮助我们屏蔽底层的&nbsp;GraphQL Java&nbsp;中的复杂概念和重复代码，GraphQL Java Tools&nbsp;能够从 GraphQL Schema 定义（即 .graphqls 文件）中构建出相应的 Java 的 POJO 类型对象，GraphQL Java Tools&nbsp;将读取 classpath 下所有以 .graphqls 为后缀名的文件，然后创建 GraphQL Schema 对象。GraphQL Java Tools&nbsp;也是依赖于&nbsp;GraphQL Java&nbsp;实现的。</p>
<p data-nodeid="106176">这里依然以上面图书管理的 demo 为例来介绍&nbsp;GraphQL Java Tools&nbsp;的使用，前文示例的GraphQL Schema 定义的 Book 、Author 以及 Query 三种类型我们保持不变。Query 接口是 GraphQL 查询的入口，我们可以通过 extend 的方式扩展 Query，如下所示：</p>
<pre class="lang-java" data-nodeid="106177"><code data-language="java">extend&nbsp;type&nbsp;Query{&nbsp;#&nbsp;扩展&nbsp;Query
&nbsp;&nbsp;&nbsp;&nbsp;getAuthorById(id:&nbsp;ID!):&nbsp;Author&nbsp;#&nbsp;根据&nbsp;id查询作者信息
}
</code></pre>
<p data-nodeid="106178">GraphQL Schema 中还可以定义 Mutation 类型作为修改数据的入口，如下所示，启动定义了 createBook 和 createAuthor 两个方法，分别用来新增图书信息和作者信息：</p>
<pre class="lang-java" data-nodeid="106179"><code data-language="java">type Mutation {
&nbsp;&nbsp;&nbsp;&nbsp;createBook(input&nbsp;:&nbsp;BookInput!)&nbsp;:&nbsp;Book!
    createAuthor(firstName:String!, lastName:String!) : ID!
}
input BookInput {  # input 表示入参
    name : String!
    pageCount : String!
    authorId: String!
}
</code></pre>
<p data-nodeid="106180">这里引入了一个 BootInput 类型，将需要传递到服务端的数据封装起来，GraphQL 中返回类型和输入类型（input）是不能共用的，所以加上 input 后缀加以区分。</p>
<p data-nodeid="106181">GraphQL Java Tools 可以将 GraphQL 对象的方法和字段映射到 Java 对象。一般情况下，GraphQL Java Tools 可以通过 POJO 的字段或是相应的 getter 方法完成字段读取，对于复杂字段，则需要定义相应的 Resolver 实现。例如，下面为 GraphQL Schema 定义对应了 Book 和 Author 两个 POJO（以及 BookInput）：</p>
<pre class="lang-java" data-nodeid="106182"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Book</span> </span>{
    <span class="hljs-keyword">private</span> String id;
    <span class="hljs-keyword">private</span> String name;
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> pageCount;
    <span class="hljs-keyword">private</span> String authorId;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;省略 getter/setter&nbsp;方法</span>
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Author</span> </span>{
    <span class="hljs-keyword">private</span> String id;
    <span class="hljs-keyword">private</span> String firstName;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">private</span>&nbsp;String&nbsp;lastName;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;省略&nbsp;getter/setter&nbsp;方法</span>
}
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">BookInput</span>&nbsp;</span>{
    <span class="hljs-keyword">private</span> String name;
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> pageCount;
    <span class="hljs-keyword">private</span> String authorId;
    <span class="hljs-comment">// 省略 getter/setter 方法</span>
}
</code></pre>
<p data-nodeid="106183">很明显，这里定义的 Book.java 中大部分字段都与 GraphQL Schema 中的 Book 一一对应，但是 authorId 字段与 GraphQL Schema 中 Book 的 author 字段是无法直接完成映射的，这里就需要一个 GraphQLResolver 实现来完成该转换，例如下面的 BookResolver 实现：</p>
<pre class="lang-java" data-nodeid="106184"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-class"><span class="hljs-keyword">class</span>&nbsp;<span class="hljs-title">BookResolver</span>&nbsp;<span class="hljs-keyword">implements</span>&nbsp;<span class="hljs-title">GraphQLResolver</span>&lt;<span class="hljs-title">Book</span>&gt;&nbsp;</span>{
<span class="hljs-meta">@Autowired</span>
<span class="hljs-keyword">private</span> AuthorService authorService;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span>&nbsp;Author&nbsp;<span class="hljs-title">author</span><span class="hljs-params">(Book&nbsp;book)</span>&nbsp;</span>{
        <span class="hljs-keyword">return</span> authorService.getAuthorById(book.getAuthorId());
    }
}
</code></pre>
<p data-nodeid="106185">在 GraphQL Java Tools 需要将 Java 对象映射成 GraphQL 对象的时候，首先会尝试使用相应 GraphQLResolver（示例中的 BookResolver）&nbsp;的相应方法完成映射（示例中的 author() 方法），如果在 GraphQLResolver 没有方法才会使用相应的 getter 方法或是直接访问字段。</p>
<p data-nodeid="106186">BookResolver.author() 方法的实现也可以看出，被映射的 Java 对象需要作为参数传入。</p>
<p data-nodeid="106187"><br>
在 GraphQL Schema 中定义的 Query 和 Mutation 是&nbsp;GraphQL&nbsp;查询和修改数据的入口，它们对应的 Resolver 实现需要实现&nbsp;GraphQLQueryResolver 或 GraphQLMutationResolver。例如下面定义的 BookService 以及 AuthorService：</p>
<pre class="lang-java" data-nodeid="106188"><code data-language="java"><span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">interface</span>&nbsp;<span class="hljs-title">BookService</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">GraphQLQueryResolver</span>,&nbsp;<span class="hljs-title">GraphQLMutationResolver</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">Book&nbsp;<span class="hljs-title">getBookById</span><span class="hljs-params">(String&nbsp;id)</span></span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">List&lt;Book&gt;&nbsp;<span class="hljs-title">list</span><span class="hljs-params">()</span></span>;
    <span class="hljs-function">Book <span class="hljs-title">createBook</span><span class="hljs-params">(BookInput input)</span></span>;
}&nbsp;
<span class="hljs-keyword">public</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">interface</span>&nbsp;<span class="hljs-title">AuthorService</span>&nbsp;<span class="hljs-keyword">extends</span>&nbsp;<span class="hljs-title">GraphQLQueryResolver</span>,&nbsp;<span class="hljs-title">GraphQLMutationResolver</span>&nbsp;</span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-function">String&nbsp;<span class="hljs-title">createAuthor</span><span class="hljs-params">(String&nbsp;firstName,&nbsp;String&nbsp;lastName)</span></span>;
    <span class="hljs-function">Author <span class="hljs-title">getAuthorById</span><span class="hljs-params">(String id)</span></span>;
}
</code></pre>
<p data-nodeid="106189">GraphQL Java Tools 会根据方法名将上述&nbsp;GraphQLQueryResolver 或 GraphQLMutationResolver&nbsp;与&nbsp;GraphQL Schema 中的 Query 和 Mutation&nbsp;进行映射。</p>
<p data-nodeid="106190">BookService 和&nbsp;AuthorService 的具体实现如下：</p>
<pre class="lang-java" data-nodeid="106191"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BookServiceImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">BookService</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;使用递增方式生成&nbsp;id后缀</span>
    <span class="hljs-keyword">private</span> AtomicLong idGenerator = <span class="hljs-keyword">new</span> AtomicLong(<span class="hljs-number">0L</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-comment">//&nbsp;这里并没有使用持久化存储，而是使用该&nbsp;List将图书信息保存在内存中</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> List&lt;Book&gt; books = Lists.newCopyOnWriteArrayList();
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Book <span class="hljs-title">getBookById</span><span class="hljs-params">(String id)</span> </span>{
    <span class="hljs-keyword">return</span> books.stream().filter(b -&gt; b.getId().equals(id))
    .findFirst().orElse(<span class="hljs-keyword">null</span>);
}
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Book&gt; <span class="hljs-title">list</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> books;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Book <span class="hljs-title">createBook</span><span class="hljs-params">(BookInput input)</span> </span>{
        String id = <span class="hljs-string">"book-"</span> + idGenerator.getAndIncrement();
        Book book = <span class="hljs-keyword">new</span> Book();
        book.setId(id);
        book.setName(input.getName());
        book.setPageCount(input.getPageCount());
        book.setAuthorId(input.getAuthorId());
        books.add(book);
        <span class="hljs-keyword">return</span> book;
    } 
}
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthorServiceImpl</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AuthorService</span> </span>{
    <span class="hljs-keyword">private</span> AtomicLong idGenerator = <span class="hljs-keyword">new</span> AtomicLong(<span class="hljs-number">0L</span>);
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> List&lt;Author&gt; authors = Lists.newCopyOnWriteArrayList();
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">createAuthor</span><span class="hljs-params">(String firstName, String lastName)</span> </span>{
    String id = <span class="hljs-string">"author-"</span> + idGenerator.getAndIncrement();
    Author author = <span class="hljs-keyword">new</span> Author();
    author.setId(id);
    author.setFirstName(firstName);
    author.setLastName(lastName);
    authors.add(author);
    <span class="hljs-keyword">return</span> id;
}
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Author <span class="hljs-title">getAuthorById</span><span class="hljs-params">(String id)</span> </span>{
    <span class="hljs-keyword">return</span> authors.stream().filter(a -&gt; a.getId().equals(id))
    .findFirst().orElse(<span class="hljs-keyword">null</span>);
    }
}
</code></pre>
<p data-nodeid="107224">最后，我们启动该 Demo 项目，使用 GraphQL Playground 分别请求 Query 和 Mutation 中定义的接口，如下图所示：</p>
<p data-nodeid="108006"><img src="https://s0.lgstatic.com/i/image/M00/21/B7/CgqCHl7q_i-AIWISAAH5fsYHv6s340.png" alt="Drawing 7.png" data-nodeid="108010"></p>
<p data-nodeid="108007" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/21/B7/CgqCHl7q_jWAYBBBAAH-aJOU-Zk683.png" alt="Drawing 8.png" data-nodeid="108013"></p>




<p data-nodeid="107033">到此为止，GraphQL 入门示例分析就结束了。</p>

---

### 精选评论


