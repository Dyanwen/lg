<p data-nodeid="24091" class="">通过之前课时的内容，相信你已经对 JPA 有了深入的认识了，那么 JPA 还有哪些应用场景呢？这一讲，我们将通过 Spring Data Rest 来聊聊实体和 Respository 的另外一种用法。</p>
<p data-nodeid="24092">首先通过一个 Demo 让你感受一下，怎么快速创建一个 Rest 风格的 Server 服务端。</p>
<h3 data-nodeid="24093">Spring Data Rest Demo</h3>
<p data-nodeid="24094">我们通过以下四个步骤演示一下 Spring Data Rest 的效果。</p>
<p data-nodeid="24095"><strong data-nodeid="24239">第一步：通过 gradle 引入相关的 jar 依赖</strong>，代码如下所示。</p>
<pre class="lang-java" data-nodeid="24096"><code data-language="java">implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-data-jpa'</span>
<span class="hljs-comment">// spring data rest的依赖，由于我们使用的是spring boot，所以只需要添加starter即可</span>
implementation(<span class="hljs-string">"org.springframework.boot:spring-boot-starter-data-rest"</span>)
<span class="hljs-comment">//我们添加swagger方便看得出来，生成了哪些api接口</span>
implementation <span class="hljs-string">'io.springfox:springfox-boot-starter:3.0.0'</span>
<span class="hljs-comment">// swagger 对spring data reset支持需要添加 springfox-data-rest</span>
implementation <span class="hljs-string">'io.springfox:springfox-data-rest:3.0.0'</span>
</code></pre>
<p data-nodeid="24097">添加完依赖之后，我们可以通过 gradle 的依赖视图看一下都用了哪些 jar 包。</p>
<p data-nodeid="24098"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5W-ASHQlAAGMwEhDK_w968.png" alt="Drawing 0.png" data-nodeid="24243"></p>
<p data-nodeid="24099">通过上图可以很清晰地看到 spring-data-rest 的 jar 包引入情况，以及我们依赖的 spring-data-jpa 和 Swagger。</p>
<p data-nodeid="24100"><strong data-nodeid="24249">第二步：在项目里面添加 SpringFoxConfiguration 开启 Swagger</strong>，代码如下所示。</p>
<pre class="lang-java" data-nodeid="24101"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@EnableSwagger2</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringFoxConfiguration</span> </span>{}
</code></pre>
<p data-nodeid="24102"><strong data-nodeid="24254">第三步：通过 application.properties 指定一个 base-path，以方便和我们自己的 api 进行区分</strong>，代码如下所示。</p>
<pre class="lang-java" data-nodeid="24103"><code data-language="java"># 我们可以通过spring data rest里面提供的配置项，指定bast-path
spring.data.rest.base-path=api/rest/v2
</code></pre>
<p data-nodeid="24104"><strong data-nodeid="24259">第四步：直接启动项目，就可以看到效果了，不需要任何额外的 controller 的配置和设置</strong>。</p>
<p data-nodeid="24105">启动成功之后，我们就会发现，里面多了很多 api/rest/v2 等 Rest 风格的 API，并且可以直接使用。如下图所示，不只有我们自己的 Controller，还有 Spring DataRest 自己生成的 API。</p>
<p data-nodeid="24106"><img src="https://s0.lgstatic.com/i/image2/M01/03/CF/Cip5yF_i5XaAfkQUAAOu3Oghurk191.png" alt="Drawing 1.png" data-nodeid="24263"></p>
<p data-nodeid="24107">这时我们打开 Swagger 看一下：<a href="http://127.0.0.1:8087/swagger-ui/" data-nodeid="24267">http://127.0.0.1:8087/swagger-ui/</a></p>
<p data-nodeid="24108"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5X2AaQhNAAIRIf6mhUk512.png" alt="Drawing 2.png" data-nodeid="24270"></p>
<p data-nodeid="24109">由于我们的 Demo 的项目结构是下图所示这样的。</p>
<p data-nodeid="24110"><img src="https://s0.lgstatic.com/i/image2/M01/03/CF/Cip5yF_i5YKAdyZXAACzqZkR3vs585.png" alt="Drawing 3.png" data-nodeid="24274"></p>
<p data-nodeid="24111">你会发现有几个 Repository 会帮我们生成几个对应的 Rest 协议的 API，除了基本的 CRUD，例如 UserInfoRespository 自定义的方法它们也会帮我们展示出来。而 Room 实体我们没有对应的 Repository，所以不会有对应的 Rest 风格 API 生成。</p>
<p data-nodeid="24112">通过这个 Demo 你可以想象一下，如果要做一个 Rest 风格的 Server API 项目，是不是只需要把对应的 Entity 和 Repository 建好，就可以直接拥有了所有的 CRUD 的 API 了？这样可以大大提高我们的开发效率。</p>
<p data-nodeid="24113">下面我们详细看一下 Spring Data Rest 的基本用法。</p>
<h3 data-nodeid="24114">Spring Data Rest 基本用法</h3>
<p data-nodeid="24115">通过 Demo 可以看得出来，Spring Data Rest 的核心功能就是把 Spring Data Resositories 里对外暴露的方法生成对应的 API，如我们上面的 AddressRepository，里面对应的实体是 Address，代码如下。</p>
<pre class="lang-java" data-nodeid="24116"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AddressRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">Address</span>, <span class="hljs-title">Long</span>&gt;</span>{
}
</code></pre>
<p data-nodeid="24117">它帮我们生成的 API 有下图所示的这些。</p>
<p data-nodeid="24118"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5YqAX6zIAAD0ZWEXWfU636.png" alt="Drawing 4.png" data-nodeid="24283"></p>
<p data-nodeid="24119">从 swagger 我们可以看到 Spring Data Rest 的几点用法。</p>
<h4 data-nodeid="24120">语义化的方法</h4>
<p data-nodeid="24121">把实体转化成复数的形式，生成基本的 PATCH、GET、PUT、POST、DELETE 带有语义的 Rest 相应的方法，包括的子资源有如下几个。</p>
<ul data-nodeid="24122">
<li data-nodeid="24123">
<p data-nodeid="24124">GET：返回单个实体</p>
</li>
<li data-nodeid="24125">
<p data-nodeid="24126">PUT：更新资源</p>
</li>
<li data-nodeid="24127">
<p data-nodeid="24128">PATCH：与 PUT 类似，但部分是更新资源状态</p>
</li>
<li data-nodeid="24129">
<p data-nodeid="24130">DELETE：删除暴露的资源</p>
</li>
<li data-nodeid="24131">
<p data-nodeid="24132">POST：从给定的请求正文创建一个新的实体</p>
</li>
</ul>
<h4 data-nodeid="24133">默认的状态码的支持</h4>
<ul data-nodeid="24134">
<li data-nodeid="24135">
<p data-nodeid="24136">200 OK：适用于纯粹的 GET 请求</p>
</li>
<li data-nodeid="24137">
<p data-nodeid="24138">201 Created：针对创建新资源的 POST 和 PUT 请求</p>
</li>
<li data-nodeid="24139">
<p data-nodeid="24140">204 No Content：对于 PUT、PATCH 和 DELETE 请求</p>
</li>
<li data-nodeid="24141">
<p data-nodeid="24142">401 没有认证</p>
</li>
<li data-nodeid="24143">
<p data-nodeid="24144">403 没有权限，拒绝访问</p>
</li>
<li data-nodeid="24145">
<p data-nodeid="24146">404 没有找到对应的资源</p>
</li>
</ul>
<h4 data-nodeid="24147">分页支持</h4>
<p data-nodeid="24148">通过 Swagger，我们可以看到其完全对分页和排序进行支持，完全兼容我们之前讲过的 Spring Data JPA 的分页和排序的参数，如下图所示。</p>
<p data-nodeid="24149"><img src="https://s0.lgstatic.com/i/image2/M01/03/CF/Cip5yF_i5ZKAMXN9AADg4CrauLY452.png" alt="Drawing 5.png" data-nodeid="24303"></p>
<h4 data-nodeid="24150">通过 @RepositoryRestResource 改变资源的 metaData</h4>
<p data-nodeid="24151">代码如下所示。</p>
<pre class="lang-java" data-nodeid="24152"><code data-language="java"><span class="hljs-meta">@RepositoryRestResource(
      exported = true, //资源是否暴露，默认true
      path = "users",//资源暴露的path访问路径，默认实体名字+s
      collectionResourceRel = "userInfo",//资源名字，默认实体名字
      collectionResourceDescription = @Description("用户基本信息资源"),//资源描述
      itemResourceRel = "userDetail",//取资源详情的Item名字
      itemResourceDescription = @Description("用户详情")
)</span>
</code></pre>
<p data-nodeid="24153">我们将其放置在 UserInfoRepository 上面测试一下，代码变更如下。</p>
<pre class="lang-java" data-nodeid="24154"><code data-language="java"><span class="hljs-meta">@RepositoryRestResource(
      exported = true,
      path = "users",
      collectionResourceRel = "userInfo",
      collectionResourceDescription = @Description("用户资源"),
      itemResourceRel = "userDetail",
      itemResourceDescription = @Description("用户详情")
)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserInfoRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">UserInfo</span>, <span class="hljs-title">Long</span>&gt; </span>{}
</code></pre>
<p data-nodeid="24155">这时通过 Swagger 可以看到，url 的 path 上面变成了 users，而 body 里面的资源名字变成了 userInfo，取 itemResource 的 URL 描述变成了 userDetail，如下图所示。</p>
<p data-nodeid="24156"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5ZyADJsUAAGZTJ31fiw252.png" alt="Drawing 6.png" data-nodeid="24310"></p>
<p data-nodeid="24157">@RepositoryRestResource 是使用在 Repository 类上面的全局设置，我们也可以针对具体的 Repsitory 里面的每个方法进行单独设置，这就是另外一个注解：@RestResource。</p>
<h4 data-nodeid="24158">@RestResource 改变 rest 的 SearchPath</h4>
<p data-nodeid="24159">代码如下所示。</p>
<pre class="lang-java" data-nodeid="24160"><code data-language="java"><span class="hljs-meta">@RestResource(
      exported = true,//是否暴露给Search
      path = "findCities",//Search后面的path路径
      rel = "cities"//资源名字
)</span>
</code></pre>
<p data-nodeid="24161">可以将其用于 ***Repository 的方法中和 @Entity 的实体关系上，那么我们在 address 的 findByAddress 方法上面做一个测试，看看会变成什么样，代码如下所示。</p>
<pre class="lang-java" data-nodeid="24162"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AddressRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">Address</span>, <span class="hljs-title">Long</span>&gt;</span>{
    <span class="hljs-meta">@RestResource(
            exported = true,//是否暴露给Search
            path = "findCities",//Search后面的path路径
            rel = "cities"//资源名字
    )</span>
    <span class="hljs-function">Page&lt;Address&gt; <span class="hljs-title">findByAddress</span><span class="hljs-params">(<span class="hljs-meta">@Param("address")</span> String address, Pageable pageable)</span></span>;
}
</code></pre>
<p data-nodeid="24163">我们打开 Swagger 看一下结果，会发现 search 后面的 path 路径被自定义了，如下图所示。</p>
<p data-nodeid="24164"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5aSAGiOEAABI57HiU0g381.png" alt="Drawing 7.png" data-nodeid="24322"></p>
<p data-nodeid="24165">同时这个注解也可以配置在关联关系上，如 @OneToMany 等。如果我们不想某些方法暴露成 RestAPI，就直接添加 @RestResource(exported = false) 这一注解即可，例如一些删除方法等。</p>
<h4 data-nodeid="24166">spring data rest 的配置项支持</h4>
<p data-nodeid="24167">这个可以直接在 application.properties 里面配置，我们在 IDEA 里面输入前缀的时候，就会有如下提示。</p>
<p data-nodeid="24168"><img src="https://s0.lgstatic.com/i/image2/M01/03/CF/Cip5yF_i5auAW0BTAAEFk4n5M4c741.png" alt="Drawing 8.png" data-nodeid="24328"></p>
<p data-nodeid="24169">对应的描述如下表所示。</p>
<p data-nodeid="24388" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8C/0E/CgqCHl_kTcWAS5aJAAC14Y9YPQ4842.png" alt="Lark20201224-161329.png" data-nodeid="24391"></p>

<p data-nodeid="24207">Spring Data Rest 的常见用法我们介绍完了，之前还讲过 Spring Data JPA 对 Jackson 的支持，它在 Spring Data Rest 里面完全适用，下面来看一下。</p>
<h3 data-nodeid="24208">返回结果对 Jackson 的支持</h3>
<p data-nodeid="24209">通过 jackson 的注解，可以改变 rest api 的属性的名字，或者忽略具体的某个属性。我们在 address 的实体里面，改变一下属性 city 的名字，同时忽略 address 属性，代码会变成如下所示的样子。</p>
<pre class="lang-java" data-nodeid="24210"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@SuperBuilder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString(exclude = "userInfo")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Address</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-meta">@JsonProperty("myCity")</span> <span class="hljs-comment">//改变JSON响应的属性名字</span>
   <span class="hljs-keyword">private</span> String city;
   <span class="hljs-meta">@JsonIgnore</span> <span class="hljs-comment">//JSON解析的时候忽略某个属性</span>
   <span class="hljs-keyword">private</span> String address;
}
</code></pre>
<p data-nodeid="24211">我们通过 Swagger 里面的 Description 可以看到，当前的资源的描述发生了变化，字段名变成了 myCity，address 属性没有了，具体如下图所示。</p>
<p data-nodeid="24212"><img src="https://s0.lgstatic.com/i/image2/M01/03/D1/CgpVE1_i5bSAA7YZAAE7YXeoJQY773.png" alt="Drawing 9.png" data-nodeid="24362"></p>
<p data-nodeid="24213">Spring Data Rest 返回 ResponseBody 的原理和接收 RequestBody 的原理都是基于 JSON 格式的，我们之前讲的 Jackson 的所有注解语法同样适用。</p>
<p data-nodeid="24214">那么介绍了这么多，到底 Spring Data Rest 和 Spring Data JPA 是什么关系呢？我们来总结一下。</p>
<h3 data-nodeid="24215">Spring Data Rest 和 Spring Data JPA 的关系</h3>
<p data-nodeid="24216">大概有如下几点。</p>
<ol data-nodeid="24217">
<li data-nodeid="24218">
<p data-nodeid="24219">Spring Data JPA 基于 JPA 协议提供了一套标准的 Repository 的操作统一接口，方法名和 @Query 都是有固定语法和约定的规则的。</p>
</li>
<li data-nodeid="24220">
<p data-nodeid="24221">Spring Data Rest 利用 JPA 的约定和语法，利用 Java 反射、动态代理等机制，很容易可以生成一套标准的 rest 风格的 API 协议操作。</p>
</li>
<li data-nodeid="24222">
<p data-nodeid="24223">也就是说 JPA 制定协议和标准，Spring Data Rest 基于这套协议生成 rest 风格的 Controller。</p>
</li>
</ol>
<h3 data-nodeid="24224">总结</h3>
<p data-nodeid="24225">由于篇幅有限，SpringDataRest 本身的原理和实现方式一个课时是介绍不完的，虽然这一讲的内容不多，但其精髓都在这里了。</p>
<p data-nodeid="24226">我想表达的重点是 JPA 的应用领域其实有很多，我的讲解就是想帮你打开思路，在写一些基于实体的框架时就可以参考 Spring Data Rest 的做法。例如<a href="https://jsonapi.org/format/" data-nodeid="24375">yahoo 团队设计的 JSONAPI 协议</a>，以及<a href="https://github.com/yahoo/elide/blob/master/translations/zh/README.md" data-nodeid="24379">Elide 的实现</a>，也是基于 JPA 的实体注解来实现的。</p>
<p data-nodeid="24227">甚至 Spring 在研究的 graph QL，也可以基于约定的实体来做很多事情。所以当你把 JPA “玩得很溜”的时候，就可以大大提升开发效率。</p>
<p data-nodeid="24228">最后欢迎你在留言区发表自己的看法，希望我们可以一起讨论。下一讲我们来聊聊如何通过 spring boot test 提高开发效率，到时见。</p>
<blockquote data-nodeid="24229">
<p data-nodeid="24230" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="24387">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论


