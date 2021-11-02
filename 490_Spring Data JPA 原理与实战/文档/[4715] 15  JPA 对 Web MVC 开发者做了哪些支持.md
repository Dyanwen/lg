<p data-nodeid="1114" class="">你好，欢迎学习第 15 课时，今天带你了解 JPA 对 Web MVC 开发者都做了哪些支持。</p>
<p data-nodeid="1115">我们使用 Spring Data JPA 的时候，一般都会用到 Spring MVC，Spring Data 对 Spring MVC 做了很好的支持，体现在以下几个方面：</p>
<ol data-nodeid="1116">
<li data-nodeid="1117">
<p data-nodeid="1118">支持在 Controller 层直接返回实体，而不使用其显式的调用方法；</p>
</li>
<li data-nodeid="1119">
<p data-nodeid="1120">对 MVC 层支持标准的分页和排序功能；</p>
</li>
<li data-nodeid="1121">
<p data-nodeid="1122">扩展的插件支持 Querydsl，可以实现一些通用的查询逻辑。</p>
</li>
</ol>
<p data-nodeid="1123">正常情况下，我们开启 Spring Data 对 Spring Web MVC 支持的时候需要在 @Configuration 的配置文件里面添加 @EnableSpringDataWebSupport 这一注解，如下面这种形式：</p>
<pre class="lang-java" data-nodeid="1124"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-meta">@EnableWebMvc</span>
<span class="hljs-comment">//开启支持Spring Data Web的支持</span>
<span class="hljs-meta">@EnableSpringDataWebSupport</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WebConfiguration</span> </span>{ }
</code></pre>
<p data-nodeid="1125">由于我们用了 Spring Boot，其有自动加载机制，会自动加载 SpringDataWebAutoConfiguration 类，发生如下变化：</p>
<pre class="lang-java" data-nodeid="1126"><code data-language="java"><span class="hljs-meta">@EnableSpringDataWebSupport</span>
<span class="hljs-meta">@ConditionalOnWebApplication(type = Type.SERVLET)</span>
<span class="hljs-meta">@ConditionalOnClass({ PageableHandlerMethodArgumentResolver.class, WebMvcConfigurer.class })</span>
<span class="hljs-meta">@ConditionalOnMissingBean(PageableHandlerMethodArgumentResolver.class)</span>
<span class="hljs-meta">@EnableConfigurationProperties(SpringDataWebProperties.class)</span>
<span class="hljs-meta">@AutoConfigureAfter(RepositoryRestMvcAutoConfiguration.class)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringDataWebAutoConfiguration</span> </span>{
</code></pre>
<p data-nodeid="1127">从类上面可以看出来，@EnableSpringDataWebSupport 会自动开启，所以当我们用 Spring Boot + JPA + MVC 的时候，什么都不需要做，因为 Spring Boot 利用 Spring Data 对 Spring MVC 做了很多 Web 开发的天然支持。支持的组件有 DomainConverter、Page、Sort、Databinding、Dynamic Param 等。</p>
<p data-nodeid="1128">那么我们先来看一下它对 DomainClassConverter 组件的支持。</p>
<h3 data-nodeid="1129">DomainClassConverter 组件</h3>
<p data-nodeid="1130">这个组件的主要作用是帮我们把 Path 中 ID 的变量，或 Request 参数中的变量 ID 的参数值，直接转化成实体对象注册到 Controller 方法的参数里面。怎么理解呢？我们看个例子，就很好懂了。</p>
<h4 data-nodeid="1131">一个例子</h4>
<p data-nodeid="1132">首先，写一个 MVC 的 Controller，分别从 Path 和 Param 变量里面，根据 ID 转化成实体，代码如下：</p>
<pre class="lang-java" data-nodeid="1133"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoController</span> </span>{
   <span class="hljs-comment">/**
    * 从path变量里面获得参数ID的值，然后直接转化成UserInfo实体
    * <span class="hljs-doctag">@param</span> userInfo
    * <span class="hljs-doctag">@return</span>
    */</span>
   <span class="hljs-meta">@GetMapping("/user/{id}")</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromPath</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> UserInfo userInfo)</span> </span>{
      <span class="hljs-keyword">return</span> userInfo;
   }
   <span class="hljs-comment">/**
    * 将request的param中的ID变量值，转化成UserInfo实体
    * <span class="hljs-doctag">@param</span> userInfo
    * <span class="hljs-doctag">@return</span>
    */</span>
   <span class="hljs-meta">@GetMapping("/user")</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromRequestParam</span><span class="hljs-params">(<span class="hljs-meta">@RequestParam("id")</span> UserInfo userInfo)</span> </span>{
      <span class="hljs-keyword">return</span> userInfo;
   }
}
</code></pre>
<p data-nodeid="1134">然后，我们运行起来，看一下结果：</p>
<pre class="lang-java" data-nodeid="1135"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8089/user/1</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span>&nbsp;
Content-Type: application/json
{
&nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
&nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
}
GET http:<span class="hljs-comment">//127.0.0.1:8089/user?id=1</span>
{
&nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
&nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
}
</code></pre>
<p data-nodeid="1136">从结果来看，Controller 里面的 getUserInfoFromRequestParam 方法会自动根据 ID 查询实体对象 UserInfo，然后注入方法的参数里面。那它是怎么实现的呢？我们看一下源码。</p>
<h4 data-nodeid="1137">源码分析</h4>
<p data-nodeid="1138">我们打开 DomainClassConverter 类，里面有个 ToEntityConverter 的内部转化类的 Matches 方法，它会判断参数的类型是不是实体，并且有没有对应的实体 Repositorie 存在。如果不存在，就会直接报错说找不到合适的参数转化器。</p>
<p data-nodeid="1139">DomainClassConverter 里面的关键代码如下：</p>
<pre class="lang-java" data-nodeid="1140"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DomainClassConverter</span>&lt;<span class="hljs-title">T</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ConversionService</span> &amp; <span class="hljs-title">ConverterRegistry</span>&gt;
      <span class="hljs-keyword">implements</span> <span class="hljs-title">ConditionalGenericConverter</span>, <span class="hljs-title">ApplicationContextAware</span> </span>{
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(TypeDescriptor sourceType, TypeDescriptor targetType)</span> </span>{
      <span class="hljs-comment">//判断参数的类型是不是实体</span>
      <span class="hljs-keyword">if</span> (sourceType.isAssignableTo(targetType)) {
         <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
      }
      Class&lt;?&gt; domainType = targetType.getType();
      <span class="hljs-comment">//有没有对应的实体的Repositorie 存在</span>
      <span class="hljs-keyword">if</span> (!repositories.hasRepositoryFor(domainType)) {
         <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
      }
      Optional&lt;RepositoryInformation&gt; repositoryInformation = repositories.getRepositoryInformationFor(domainType);
      <span class="hljs-keyword">return</span> repositoryInformation.map(it -&gt; {
         Class&lt;?&gt; rawIdType = it.getIdType();
         <span class="hljs-keyword">return</span> sourceType.equals(TypeDescriptor.valueOf(rawIdType))
               || conversionService.canConvert(sourceType.getType(), rawIdType);
      }).orElseThrow(
            () -&gt; <span class="hljs-keyword">new</span> IllegalStateException(String.format(<span class="hljs-string">"Couldn't find RepositoryInformation for %s!"</span>, domainType)));
   }
}
......}
</code></pre>
<p data-nodeid="1141">所以，我们上面的例子其实是需要有 UserInfoRepository 的，否则会失败。通过源码我们也可以看到，如果 matches=true，那么就会执行下面的 convert 方法，最终调用 findById 的方法帮我们执行查询动作，如下图所示：</p>
<p data-nodeid="1142"><img src="https://s0.lgstatic.com/i/image/M00/66/CF/Ciqc1F-f3MqAH54fAANYwOjyA38629.png" alt="Drawing 0.png" data-nodeid="1311"></p>
<p data-nodeid="1143">而 DomainClassConverter 是 Spring MVC 自定义 Formatter 的一直种机制，加载进去，可以看到如下界面：</p>
<p data-nodeid="1144"><img src="https://s0.lgstatic.com/i/image/M00/66/CF/Ciqc1F-f3NWAIYBDAAIFxTM8YZQ845.png" alt="Drawing 1.png" data-nodeid="1315"></p>
<p data-nodeid="1145">而 SpringDataWebConfiguration 是因为实现了 WebMvcConfigurer 的 addFormatters 所有加载了自定义参数转化器的功能，所以才有了 DomainClassConverter 组件的支持。关键代码如下：</p>
<pre class="lang-java" data-nodeid="1146"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringDataWebConfiguration</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WebMvcConfigurer</span>, <span class="hljs-title">BeanClassLoaderAware</span> </span>{
......}
</code></pre>
<p data-nodeid="1147">从源码上我们也可以看到，DomainClassConverter 只会根据 ID 来查询实体，很有局限性，没有更加灵活的参数转化功能，不过你也可以根据源码自己进行扩展，我在这就不展示更多了。</p>
<p data-nodeid="1148">下面来看一下JPA 对 Web MVC 分页和排序是如何支持的。</p>
<h3 data-nodeid="1149">Page 和 Sort 的参数支持</h3>
<p data-nodeid="1150">我们还是先通过一个例子来说明。</p>
<h4 data-nodeid="1151">一个实例</h4>
<p data-nodeid="1152">这是一个通过分页和排序参数查询 UserInfo 的实例。</p>
<p data-nodeid="1153">首先，我们新建一个 UserInfoController，里面添加如下两个方法，分别测试分页和排序。</p>
<pre class="lang-java" data-nodeid="1154"><code data-language="java"><span class="hljs-meta">@GetMapping("/users")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Page&lt;UserInfo&gt; <span class="hljs-title">queryByPage</span><span class="hljs-params">(Pageable pageable, UserInfo userInfo)</span> </span>{
   <span class="hljs-keyword">return</span> userInfoRepository.findAll(Example.of(userInfo),pageable);
}
<span class="hljs-meta">@GetMapping("/users/sort")</span>
<span class="hljs-keyword">public</span> HttpEntity&lt;List&lt;UserInfo&gt;&gt; queryBySort(Sort sort) {
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> HttpEntity&lt;&gt;(userInfoRepository.findAll(sort));
}
</code></pre>
<p data-nodeid="1155">其中，queryByPage 方法中，两个参数可以分别接收分页参数和查询条件，我们请求一下，看看效果：</p>
<pre class="lang-java" data-nodeid="1156"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8089/users?size=2&amp;page=0&amp;ages=10&amp;sort=id,desc</span>
</code></pre>
<p data-nodeid="1157">参数里面可以支持分页大小为 2、页码 0、排序（按照 ID 倒序）、参数 ages=10 的所有结果，如下所示：</p>
<pre class="lang-java" data-nodeid="1158"><code data-language="java">{
&nbsp; <span class="hljs-string">"content"</span>: [
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">4</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
&nbsp; &nbsp; },
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">3</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
&nbsp; &nbsp; }
&nbsp; ],
&nbsp; <span class="hljs-string">"pageable"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"sort"</span>: {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
&nbsp; &nbsp; },
&nbsp; &nbsp; <span class="hljs-string">"offset"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; <span class="hljs-string">"pageNumber"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; <span class="hljs-string">"pageSize"</span>: <span class="hljs-number">2</span>,
&nbsp; &nbsp; <span class="hljs-string">"unpaged"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; <span class="hljs-string">"paged"</span>: <span class="hljs-keyword">true</span>
&nbsp; },
&nbsp; <span class="hljs-string">"totalPages"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"totalElements"</span>: <span class="hljs-number">4</span>,
&nbsp; <span class="hljs-string">"last"</span>: <span class="hljs-keyword">false</span>,
&nbsp; <span class="hljs-string">"size"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"number"</span>: <span class="hljs-number">0</span>,
&nbsp; <span class="hljs-string">"numberOfElements"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"sort"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">true</span>,
&nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
&nbsp; },
&nbsp; <span class="hljs-string">"first"</span>: <span class="hljs-keyword">true</span>,
&nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
}
</code></pre>
<p data-nodeid="1159">上面的字段我就不一一介绍了，在第 4 课时（如何利用 Repository 中的方法返回值解决实际问题）我们已经讲过了，只不过现在应用到了 MVC 的 View 层。</p>
<p data-nodeid="1160">因此，我们可以得出结论：Pageable 既支持分页参数，也支持排序参数。并且从下面这行代码可以看出其也可以单独调用 Sort 参数。</p>
<pre class="lang-java" data-nodeid="1161"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8089/users/sort?ages=10&amp;sort=id,desc</span>
</code></pre>
<p data-nodeid="1162">那么它的实现原理是什么呢？</p>
<h4 data-nodeid="1163">原理分析</h4>
<p data-nodeid="1164">和 DomainClassConverter 组件的支持是一样的，由于 SpringDataWebConfiguration 实现了 WebMvcConfigurer 接口，通过 addArgumentResolvers 方法，扩展了 Controller 方法的参数 HandlerMethodArgumentResolver</p>
<p data-nodeid="1165">的解决者，从下面图片中你就可以看出来。</p>
<p data-nodeid="1166"><img src="https://s0.lgstatic.com/i/image/M00/66/DA/CgqCHl-f3OWAMLu1AAJEBkl8MfA213.png" alt="Drawing 2.png" data-nodeid="1334"></p>
<p data-nodeid="1167">我们通过箭头的地方分析一下 SortHandlerMethodArgumentResolver 的类，会看到如下界面：</p>
<p data-nodeid="1168"><img src="https://s0.lgstatic.com/i/image/M00/66/DA/CgqCHl-f3OyAL_EwAANxdzchFbo230.png" alt="Drawing 3.png" data-nodeid="1338"></p>
<p data-nodeid="1169">这个类里面最关键的就是下面两个方法：</p>
<ol data-nodeid="1170">
<li data-nodeid="1171">
<p data-nodeid="1172">supportsParameter，表示只处理类型为 Sort.class 的参数；</p>
</li>
<li data-nodeid="1173">
<p data-nodeid="1174">resolveArgument，可以把请求里面参数的值，转换成该方法里面的参数 Sort 对象。</p>
</li>
</ol>
<p data-nodeid="1175">这里还要提到的是另外一个类：PageHandlerMethodArgumentResolver 类。</p>
<p data-nodeid="1176"><img src="https://s0.lgstatic.com/i/image/M00/66/CF/Ciqc1F-f3PSAXWofAANpCw8f8NY210.png" alt="Drawing 4.png" data-nodeid="1345"></p>
<p data-nodeid="1177">这个类里面也有两个最关键的方法：</p>
<ol data-nodeid="1178">
<li data-nodeid="1179">
<p data-nodeid="1180">supportsParameter，表示我只处理类型是 Pageable.class 的参数；</p>
</li>
<li data-nodeid="1181">
<p data-nodeid="1182">resolveArgument，把请求里面参数的值，转换成该方法里面的参数 Pageable 的实现类 PageRequest。</p>
</li>
</ol>
<p data-nodeid="1183">关于 Web 请求的分页和排序的支持就介绍到这里，那么如果返回的是一个 Projection 的接口，Spring 是怎么处理的呢？我们接着看。</p>
<h3 data-nodeid="1184">Web Databinding Support</h3>
<p data-nodeid="1185">之前我们在 08 课时，讲 Projection 的时候提到过接口，Spring Data JPA 里面，也可以通过 @ProjectedPayload 和 @JsonPath 对接口进行注解支持，不过要注意这与前面所讲的 Jackson 注解的区别在于，此时我们讲的是接口。</p>
<h4 data-nodeid="1186">一个实例</h4>
<p data-nodeid="1187">这里我依然结合一个实例来对这个接口进行讲解，请看下面的步骤。</p>
<p data-nodeid="1188"><strong data-nodeid="1357">第一步：如果要支持 Projection，必须要在 gradle 里面引入 jsonpath 依赖才可以：</strong></p>
<pre class="lang-java" data-nodeid="1189"><code data-language="java">implementation <span class="hljs-string">'com.jayway.jsonpath:json-path'</span>
</code></pre>
<p data-nodeid="1190"><strong data-nodeid="1361">第二步：新建一个 UserInfoInterface 接口类，用来接收接口传递的 json 对象。</strong></p>
<pre class="lang-java" data-nodeid="1191"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.web.JsonPath;
<span class="hljs-keyword">import</span> org.springframework.data.web.ProjectedPayload;
<span class="hljs-meta">@ProjectedPayload</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserInfoInterface</span> </span>{
   <span class="hljs-meta">@JsonPath("$.ages")</span> <span class="hljs-comment">// 第一级参数/JSON里面找ages字段</span>
<span class="hljs-comment">// @JsonPath("$..ages") $..代表任意层级找ages字段</span>
   <span class="hljs-function">Integer <span class="hljs-title">getAges</span><span class="hljs-params">()</span></span>;
   <span class="hljs-meta">@JsonPath("$.telephone")</span> <span class="hljs-comment">//第一级找参数/JSON里面的telephone字段</span>
<span class="hljs-comment">// @JsonPath({ "$.telephone", "$.user.telephone" }) //第一级或者user下面的telephone都可以</span>
   <span class="hljs-function">String <span class="hljs-title">getTelephone</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="1192"><strong data-nodeid="1365">第三步：在 Controller 里面新建一个 post 方法，通过接口获得 RequestBody 参数对象里面的值。</strong></p>
<pre class="lang-java" data-nodeid="1193"><code data-language="java"><span class="hljs-meta">@PostMapping("/users/projected")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfoInterface <span class="hljs-title">saveUserInfo</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> UserInfoInterface userInfoInterface)</span> </span>{
   <span class="hljs-keyword">return</span> userInfoInterface;
}
</code></pre>
<p data-nodeid="1194"><strong data-nodeid="1369">第四步：我们发送一个 get 请求，代码如下：</strong></p>
<pre class="lang-java" data-nodeid="1195"><code data-language="java">POST /users HTTP/<span class="hljs-number">1.1</span>
{<span class="hljs-string">"ages"</span>:<span class="hljs-number">10</span>,<span class="hljs-string">"telephone"</span>:<span class="hljs-string">"123456789"</span>}
</code></pre>
<p data-nodeid="1196">此时可以正常得到如下结果：</p>
<pre class="lang-java" data-nodeid="1197"><code data-language="java">{
&nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
}
</code></pre>
<p data-nodeid="1198">这个响应结果说明了接口可以正常映射。现在你知道用法了，我们再通过源码分析一下其原理。</p>
<h4 data-nodeid="1199">原理分析</h4>
<p data-nodeid="1200">很简单，我们还是直接看 SpringDataWebConfiguration，其中实现的 WebMvcConfigurer 接口里面有个 extendMessageConverters 方法，方法中加了一个 ProjectingJackson2HttpMessageConverter 的类，这个类会把带 ProjectedPayload.class 注解的接口进行 Converter。</p>
<p data-nodeid="1201">我们看一下其中主要的两个方法：</p>
<p data-nodeid="1202">1.加载 ProjectingJackson2HttpMessageConverter，用来做 Projecting 的接口转化。我们通过源码看一下是在哪里被加载进去的，如下：</p>
<p data-nodeid="1203"><img src="https://s0.lgstatic.com/i/image/M00/66/DA/CgqCHl-f3QGAceZhAAMGX5hYHo8045.png" alt="Drawing 5.png" data-nodeid="1378"></p>
<p data-nodeid="1204">2.而 ProjectingJackson2HttpMessageConverter 主要是继承了 MappingJackson2HttpMessageConverter，并且实现了 HttpMessageConverter 的接口里面的两个重要方法，如下图所示：</p>
<p data-nodeid="1205"><img src="https://s0.lgstatic.com/i/image/M00/66/DA/CgqCHl-f3QeAH_xxAAHoIpHyr-Q310.png" alt="Drawing 6.png" data-nodeid="1382"></p>
<p data-nodeid="1206">其中，</p>
<ul data-nodeid="1207">
<li data-nodeid="1208">
<p data-nodeid="1209">canRead 通过判断参数的实体类型里面是否有接口，以及是否有 ProjectedPayload.class 注解后，才进行解析；</p>
</li>
<li data-nodeid="1210">
<p data-nodeid="1211">read 方法负责把 HttpInputMessage 转化成 Projected 的映射代理对象。</p>
</li>
</ul>
<p data-nodeid="1212">现在你知道了 Spring 里面是如何通过 HttpMessageConverter 对 Projected 进行的支持，在使用过程中，希望你针对实际情况多去 Debug。不过这个不常用，你知道一下就可以了。</p>
<p data-nodeid="1213">下面介绍一个通过 QueryDSL 对 Web 请求进行动态参数查询的方法。</p>
<h3 data-nodeid="1214">QueryDSL Web Support</h3>
<p data-nodeid="1215">实际工作中，经常有人会用 Querydsl 做一些复杂查询，方便生成 Rest 的 API 接口，那么这种方法有什么好处，又会暴露什么缺点呢？我们先看一个实例。</p>
<h4 data-nodeid="1216">一个实例</h4>
<p data-nodeid="1217">这是一个通过 QueryDSL 作为请求参数的使用案例，通过它你就可以体验一下 QueryDSL 的用法和使用场景，我们一步一步来看一下。</p>
<p data-nodeid="1218"><strong data-nodeid="1395">第一步：需要 grandle 引入 querydsl 的依赖。</strong></p>
<pre class="lang-java" data-nodeid="1219"><code data-language="java">implementation <span class="hljs-string">'com.querydsl:querydsl-apt'</span>
implementation <span class="hljs-string">'com.querydsl:querydsl-jpa'</span>
annotationProcessor(<span class="hljs-string">"com.querydsl:querydsl-apt:4.3.1:jpa"</span>,
        <span class="hljs-string">"org.hibernate.javax.persistence:hibernate-jpa-2.1-api:1.0.2.Final"</span>,
        <span class="hljs-string">"javax.annotation:javax.annotation-api:1.3.2"</span>,
        <span class="hljs-string">"org.projectlombok:lombok"</span>)
annotationProcessor(<span class="hljs-string">"org.springframework.boot:spring-boot-starter-data-jpa"</span>)
annotationProcessor <span class="hljs-string">'org.projectlombok:lombok'</span>
</code></pre>
<p data-nodeid="1220"><strong data-nodeid="1399">第二步：UserInfoRepository 继承 QuerydslPredicateExecutor 接口，就可以实现 QueryDSL 的查询方法了，代码如下：</strong></p>
<pre class="lang-java" data-nodeid="1221"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserInfoRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">UserInfo</span>, <span class="hljs-title">Long</span>&gt;, <span class="hljs-title">QuerydslPredicateExecutor</span>&lt;<span class="hljs-title">UserInfo</span>&gt; </span>{}
</code></pre>
<p data-nodeid="1222"><strong data-nodeid="1403">第三步：Controller 里面直接利用 @QuerydslPredicate 注解接收 Predicate predicate 参数。</strong></p>
<pre class="lang-java" data-nodeid="1223"><code data-language="java"><span class="hljs-meta">@GetMapping(value = "user/dsl")</span>
<span class="hljs-function">Page&lt;UserInfo&gt; <span class="hljs-title">queryByDsl</span><span class="hljs-params">(<span class="hljs-meta">@QuerydslPredicate(root = UserInfo.class)</span> com.querydsl.core.types.Predicate predicate, Pageable pageable)</span> </span>{
<span class="hljs-comment">//这里面我用的userInfoRepository里面的QuerydslPredicateExecutor里面的方法</span>
   <span class="hljs-keyword">return</span> userInfoRepository.findAll(predicate, pageable);
}
</code></pre>
<p data-nodeid="1224"><strong data-nodeid="1409">第四步：直接请求我们的 user / dsl 即可，这里利用 queryDsl 的语法 ，使 &amp;ages=10 作为我们的请求参数。</strong></p>
<pre class="lang-java" data-nodeid="1225"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8089/user/dsl?size=2&amp;page=0&amp;ages=10&amp;sort=id%2Cdesc&amp;ages=10</span>
Content-Type: application/json
{
&nbsp; <span class="hljs-string">"content"</span>: [
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">2</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
&nbsp; &nbsp; },
&nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-string">"123456789"</span>
&nbsp; &nbsp; }
&nbsp; ],
&nbsp; <span class="hljs-string">"pageable"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"sort"</span>: {
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
&nbsp; &nbsp; },
&nbsp; &nbsp; <span class="hljs-string">"offset"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; <span class="hljs-string">"pageNumber"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; <span class="hljs-string">"pageSize"</span>: <span class="hljs-number">2</span>,
&nbsp; &nbsp; <span class="hljs-string">"unpaged"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; <span class="hljs-string">"paged"</span>: <span class="hljs-keyword">true</span>
&nbsp; },
&nbsp; <span class="hljs-string">"totalPages"</span>: <span class="hljs-number">1</span>,
&nbsp; <span class="hljs-string">"totalElements"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"last"</span>: <span class="hljs-keyword">true</span>,
&nbsp; <span class="hljs-string">"size"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"number"</span>: <span class="hljs-number">0</span>,
&nbsp; <span class="hljs-string">"sort"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">true</span>,
&nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">false</span>,
&nbsp; &nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
&nbsp; },
&nbsp; <span class="hljs-string">"numberOfElements"</span>: <span class="hljs-number">2</span>,
&nbsp; <span class="hljs-string">"first"</span>: <span class="hljs-keyword">true</span>,
&nbsp; <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
}
Response code: <span class="hljs-number">200</span>; Time: <span class="hljs-number">721</span>ms; Content length: <span class="hljs-number">425</span> bytes
</code></pre>
<p data-nodeid="1226">现在我们可以得出结论：QuerysDSL 可以帮我们省去创建 Predicate 的过程，简化了操作流程。但是它依然存在一些局限性，比如多了一些模糊查询、范围查询、大小查询，它对这些方面的支持不是特别友好。可能未来会更新、优化，不过在这里你只要关注一下就可以了。</p>
<p data-nodeid="1227">此外，你还要注意这里讲解的 QuerysDSL 的参数处理方式与第 10 课时“JpaSpecificationExecutor 实战应用场景”讲的参数处理方式的区别，你可以自己感受一下，看看哪个使用起来更加方便。</p>
<h4 data-nodeid="1228">原理分析</h4>
<p data-nodeid="1229">QueryDSL 也是主要利用自定义 Spring MVC 的 HandlerMethodArgumentResolver 实现类，根据请求的参数字段，转化成 Controller 里面所需要的参数，请看一下源码。</p>
<pre class="lang-java" data-nodeid="1230"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">QuerydslPredicateArgumentResolver</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">HandlerMethodArgumentResolver</span> </span>{
....
<span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">resolveArgument</span><span class="hljs-params">(MethodParameter parameter, <span class="hljs-meta">@Nullable</span> ModelAndViewContainer mavContainer,
      NativeWebRequest webRequest, <span class="hljs-meta">@Nullable</span> WebDataBinderFactory binderFactory)</span> <span class="hljs-keyword">throws</span> Exception </span>{
      .....<span class="hljs-comment">//你有兴趣的话可以在下图关键节点打个断点看看效果，我就不多说了</span>
</code></pre>
<p data-nodeid="1231"><img src="https://s0.lgstatic.com/i/image/M00/66/DA/CgqCHl-f3SCAX2bZAANpM9TDbyc579.png" alt="Drawing 7.png" data-nodeid="1416"></p>
<p data-nodeid="1232">在实际开发中，关于 insert 和 update 的接口我们是“逃不掉”的，但不是每次的字段都会全部传递过来，那这个时候我们应该怎么做呢？这就涉及了上述实例里面的两个注解 @DynamicUpdate 和 @DynamicInsert，下面来详细介绍一下。</p>
<h3 data-nodeid="1233">@DynamicUpdate &amp; @DynamicInsert 详解</h3>
<h4 data-nodeid="1234">通过语法快速了解</h4>
<p data-nodeid="1235">@DynamicInsert：这个注解表示 insert 的时候，会动态生产 insert SQL 语句，其生成 SQL 的规则是：<strong data-nodeid="1427">只有非空的字段才能生成 SQL</strong>。代码如下：</p>
<pre class="lang-java" data-nodeid="1236"><code data-language="java"><span class="hljs-meta">@Target( TYPE )</span>
<span class="hljs-meta">@Retention( RUNTIME )</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> DynamicInsert {
   <span class="hljs-comment">//默认是true，如果设置成false，就表示空的字段也会生成sql语句；</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>
<p data-nodeid="1237">这个注解主要是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 insert SQL 的 Columns 只包含非空的字段；如果实体中不加这个注解，默认的情况是空的，字段也会作为 insert 语句里面的 Columns。</p>
<p data-nodeid="1238">@DynamicUpdate：和 insert 是一个意思，只不过这个注解指的是在 update 的时候，会动态产生 update SQL 语句，生成 SQL 的规则是：<strong data-nodeid="1434">只有非空的字段才会生成到 update SQL 的 Columns 里面</strong>。请看代码：</p>
<pre class="lang-java" data-nodeid="1485"><code data-language="java"><span class="hljs-meta">@Target( TYPE )</span>
<span class="hljs-meta">@Retention( RUNTIME )</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> DynamicUpdate {
   <span class="hljs-comment">//和insert里面一个意思，默认true，如果设置成false和不添加这个注解的效果一样</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>

<p data-nodeid="3722">和上一个注解的原理类似，这个注解也是用在 @Entity 的实体中，如果加上这个注解，就表示生成的 update SQL 的 Columns 只包含改变的字段；如果不加这个注解，默认的情况是所有的字段也会作为 update 语句里面的 Columns。</p>
<p data-nodeid="17942" class="">这样做的目的是提高 sql 的执行效率，默认更新所有字段，这样会导致一些到索引的字段也会更新，这样 sql 的执行效率就比较低了。需要注意的是：这种生效的前提是 select-before-update 的触发机制。</p>






















<p data-nodeid="1241">这是什么意思呢？我们看个案例感受一下。</p>
<h4 data-nodeid="1242" class="">案例</h4>
<p data-nodeid="1243">第一步：为了方便测试，我们修改一下 User 实体：加上 @DynamicInsert 和 @DynamicUpdate 注解。</p>
<pre class="lang-java" data-nodeid="1244"><code data-language="java"><span class="hljs-meta">@DynamicInsert</span>
<span class="hljs-meta">@DynamicUpdate</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> String email;
   <span class="hljs-meta">@Enumerated(EnumType.STRING)</span>
   <span class="hljs-keyword">private</span> SexEnum sex;
   <span class="hljs-keyword">private</span> Integer age;
......}<span class="hljs-comment">//其他不变的信息省略</span>
</code></pre>
<p data-nodeid="1245">第二步：UserInfo 实体还保持不变，即没有加上 @DynamicInsert 和 @DynamicUpdate 注解。</p>
<pre class="lang-java" data-nodeid="1246"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-keyword">private</span> String telephone;
}
</code></pre>
<p data-nodeid="1247">第三步：我们在 UserController 里面添加如下方法，用来测试新增和更新 User。</p>
<pre class="lang-java" data-nodeid="1248"><code data-language="java"><span class="hljs-meta">@PostMapping("/user")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">saveUser</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> User user)</span> </span>{
   <span class="hljs-keyword">return</span> userRepository.save(user);
}
</code></pre>
<p data-nodeid="1249">第四步：在 UserInfoController 里面添加如下方法，用来测试新增和更新 UserInfo。</p>
<pre class="lang-java" data-nodeid="1250"><code data-language="java"><span class="hljs-meta">@PostMapping("/user/info")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">saveUserInfo</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> UserInfo userInfo)</span> </span>{
   <span class="hljs-keyword">return</span> userInfoRepository.save(userInfo);
}
</code></pre>
<p data-nodeid="1251">第五步：测试一下 UserController的post 的 user 情况，看一下 insert 的情况。</p>
<pre class="lang-java" data-nodeid="35016"><code data-language="java">#### 通过post测试insert
POST /user HTTP/1.1
Host: 127.0.0.1:8089
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 56d8dc02-7f3e-7b95-7ff1-572a4bb7d102
{"age":10,"name":"jack"}
</code></pre>

<p data-nodeid="78758" class="te-preview-highlight">这时，我们发送一个 post 请求，只带 ages 和 name 字段，而并没有带上 User 实体里面的其他字段，看一下生成的 sql 是什么样的。</p>




























<pre class="lang-java" data-nodeid="35763"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user</span> <span class="hljs-params">(create_time, last_modified_time, version, age, name, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?, ?)</span>
</span></code></pre>

<p data-nodeid="1255">这时你会发现，除了 BaseEntity 里面的一些基础字段，而其他字段并没有生成到 insert 语句里面。</p>
<p data-nodeid="1256">第六步：我们再测试一下 user 的 update 情况。</p>
<pre class="lang-java" data-nodeid="37257"><code data-language="java">#### 还是发生post请求，带上ID和version执行update操作
POST /user HTTP/1.1
Host: 127.0.0.1:8089
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 56d8dc02-7f3e-7b95-7ff1-572a4bb7d102

{name":"jack1","id":1,"version":0}
</code></pre>


<p data-nodeid="44882">此时你会看到，update 和 insert 的区别有两点：</p>
<ol data-nodeid="45660">
<li data-nodeid="45661">
<p data-nodeid="45662" class="">去掉了 age 字段，修改了 name 字段的值；</p>
</li>
<li data-nodeid="45663">
<p data-nodeid="45664">当 Entity 里面有 version 字段的时候，我们再带上 version 和 id 就会显示为 update。</p>
</li>
</ol>











<p data-nodeid="38005" class="">再看一下调用完之后的 sql：用一条 select 查询一下实体是否存在，代码如下：</p>

<pre class="lang-java" data-nodeid="1259"><code data-language="java">Hibernate: select user0_.id as id1_1_0_, user0_.create_time as create_t2_1_0_, user0_.create_user_id as create_u3_1_0_, user0_.last_modified_time as last_mod4_1_0_, user0_.last_modified_user_id as last_mod5_1_0_, user0_.version as version6_1_0_, user0_.age as age7_1_0_, user0_.deleted as deleted8_1_0_, user0_.email as email9_1_0_, user0_.name as name10_1_0_, user0_.sex as sex11_1_0_ from user user0_ where user0_.id=?
</code></pre>
<p data-nodeid="46431" class="">其中一条 update 动态更新了我们传递的那些值，只更新有变化的字段，而包括了 null 的字段也更新了，如 age 字段中我们传递的是 null，所以 update 的 sql 打印如下：</p>

<pre class="lang-java" data-nodeid="47197"><code data-language="java">Hibernate: update user set last_modified_time=?, version=?, name=?，age=? where id=? and version=?
</code></pre>

<p data-nodeid="1262">第七步：那么我们再看一下 UserInfo 的 insert 方法。</p>
<pre class="lang-java" data-nodeid="1263"><code data-language="java">#### insert
POST /user/info HTTP/1.1
Host: 127.0.0.1:8089
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 56d8dc02-7f3e-7b95-7ff1-572a4bb7d102
{"ages":10}
</code></pre>
<p data-nodeid="76352" class="">发送一个 post 的 insert 操作，我们看一下 sql：</p>





<pre class="lang-java" data-nodeid="1265"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user_info</span> <span class="hljs-params">(create_time, create_user_id, last_modified_time, last_modified_user_id, version, ages, telephone, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?, ?, ?, ?)</span>
</span></code></pre>
<p data-nodeid="1266">你会发现，无论你有没有传递值，每个字段都做了 insert，没有传递的话会用 null 代替。<br>
第八步：我们再看一下 UserInfo 的 update 方法。</p>
<pre class="lang-java" data-nodeid="1267"><code data-language="java">#### update
POST /user/info HTTP/1.1
Host: 127.0.0.1:8089
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 56d8dc02-7f3e-7b95-7ff1-572a4bb7d102
{"ages":10,"id":1,"version":0}
</code></pre>
<p data-nodeid="1268">还是发送一个 post 的 update 操作，原理一样，也是带上 ID 和 version 即可。我们看一下 SQL：</p>
<pre class="lang-java" data-nodeid="1269"><code data-language="java">Hibernate: update user_info set create_time=?, create_user_id=?, last_modified_time=?, last_modified_user_id=?, version=?, ages=?, telephone=? where id=? and version=?
</code></pre>
<p data-nodeid="47962">通过 update 的 SQL 可以看出，即使我们传递了 ages 的值，虽然没有变化，它也会把我们所有字段进行更新，包括未传递的 telephone 会更新成 null。</p>
<p data-nodeid="47963">通过上面的两个例子你应该能弄清楚 @DynamicInsert 和 @DynamicUpdate 注解是做什么的了，我们在写 API 的时候就要考虑一下是否需要对 null 的字段进行操作，因为 JPA 是不知道字段为 null 的时候，是想更新还是不想更新，所以默认 JPA 会比较实例对象里面的所有包括 null 的字段，发现有变化也会更新。</p>
<p data-nodeid="51052">而当我们做 API 开发的时候，有些场景是不期望更新未传递的字段的，例如如果我们没有传递某些字段而不期望 server 更新，那么我们该怎么做呢？</p>
<h4 data-nodeid="54152">只更新非 Null 的字段</h4>
<p data-nodeid="55712">第一步：新增一个 PropertyUtils 工具类，用来复制字段的属性值，代码如下：</p>
<pre class="lang-java" data-nodeid="58448"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.util;

<span class="hljs-keyword">import</span> com.google.common.collect.Sets;
<span class="hljs-keyword">import</span> org.springframework.beans.BeanUtils;
<span class="hljs-keyword">import</span> org.springframework.beans.BeanWrapper;
<span class="hljs-keyword">import</span> org.springframework.beans.BeanWrapperImpl;

<span class="hljs-keyword">import</span> java.util.Set;

<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PropertyUtils</span> </span>{

    <span class="hljs-comment">/**
     * 只copy非null字段
     *
     * <span class="hljs-doctag">@param</span> source
     * <span class="hljs-doctag">@param</span> dest
     */</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">copyNotNullProperty</span><span class="hljs-params">(Object source, Object dest)</span> </span>{
        <span class="hljs-comment">//利用spring提供的工具类忽略为null的字段</span>
        BeanUtils.copyProperties(source, dest, getNullPropertyNames(source));
    }

    <span class="hljs-comment">/**
     * get property name that value is null
     *
     * <span class="hljs-doctag">@param</span> source
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> String[] getNullPropertyNames(Object source) {
        <span class="hljs-keyword">final</span> BeanWrapper src = <span class="hljs-keyword">new</span> BeanWrapperImpl(source);
        java.beans.PropertyDescriptor[] pds = src.getPropertyDescriptors();

        Set&lt;String&gt; emptyNames = Sets.newHashSet();
        <span class="hljs-keyword">for</span> (java.beans.PropertyDescriptor pd : pds) {
            Object srcValue = src.getPropertyValue(pd.getName());
            <span class="hljs-keyword">if</span> (srcValue == <span class="hljs-keyword">null</span>) {
                emptyNames.add(pd.getName());
            }
        }
        String[] result = <span class="hljs-keyword">new</span> String[emptyNames.size()];
        <span class="hljs-keyword">return</span> emptyNames.toArray(result);
    }
}
</code></pre>
<p data-nodeid="60811" class="">第二步：我们的 User 实体保持不变，类里面还加上 @DynamicUpdate 注解，新增一个 Controller 方法，代码如下：</p>
<pre class="lang-java" data-nodeid="62778"><code data-language="java"><span class="hljs-comment">/**
 * <span class="hljs-doctag">@param</span> user
 * <span class="hljs-doctag">@return</span>
 */</span>
<span class="hljs-meta">@PostMapping("/user/notnull")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">saveUserNotNullProperties</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> User user)</span> </span>{
   <span class="hljs-comment">//数据库里面取出最新的数据，当然了这一步严谨一点可以根据id和version来取数据，如果没取到可以报乐观锁异常</span>
   User userSrc = userRepository.findById(user.getId()).get();
   <span class="hljs-comment">//将不是null的字段copy到userSrc里面，我们只更新传递了不是null的字段</span>
   PropertyUtils.copyNotNullProperty(user,userSrc);
   <span class="hljs-keyword">return</span> userRepository.save(userSrc);
}
</code></pre>
<p data-nodeid="65159" class="">第三步：调用 API，触发更新操作：</p>
<pre class="lang-java" data-nodeid="67141"><code data-language="java">POST http:<span class="hljs-comment">//127.0.0.1:8089/user HTTP/1.1</span>
Host: <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8089</span>
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: <span class="hljs-number">56</span>d8dc02-<span class="hljs-number">7f</span>3e-<span class="hljs-number">7</span>b95-<span class="hljs-number">7f</span>f1-<span class="hljs-number">572</span>a4bb7d102

{
  <span class="hljs-string">"name"</span>: <span class="hljs-string">"jack1"</span>,
  <span class="hljs-string">"version"</span>: <span class="hljs-number">1</span>,
  <span class="hljs-string">"id"</span>:<span class="hljs-string">"1"</span>
}
</code></pre>
<p data-nodeid="69538">发送一个更新请求，和上面的更新请求一样，还是 age 不传递，值传递改变了的 name 属性，我们再看一下 sql 的变化，代码如下：</p>
<pre class="lang-java" data-nodeid="71535"><code data-language="java">update user set last_modified_time=?, version=?, name=? where id=? and version=?
</code></pre>
<p data-nodeid="72340" class="">你会发现，这个时候未传递的 age 字段就不会更新了。实际工作中你也可以将 Controller 里面的逻辑放到 BaseService 里面，提供一个公共的 updateOnlyNotNull 的方法，以便和默认的 save 方法作区分。</p>
<p data-nodeid="72341" class="">我们既然做了 MVC，一定也免不了要对系统进行监控，那么怎么看监控指标呢？</p>

























































<h3 data-nodeid="1273">Spring Data 对系统监控做了哪些支持？</h3>
<p data-nodeid="1274">对数据层面的系统进行监控，这里我主要为你介绍两个方法。</p>
<p data-nodeid="1275"><strong data-nodeid="1462">方法一：/actuator/health 的支持，里面会检查 DB 的状态。</strong></p>
<p data-nodeid="1276"><img src="https://s0.lgstatic.com/i/image/M00/66/CF/Ciqc1F-f3TCAM1YCAACeuNm2X7E973.png" alt="Drawing 8.png" data-nodeid="1465"></p>
<p data-nodeid="1277"><strong data-nodeid="1469">方法二：/actuator/prometheus 里面会包含一些 Hibernate 和 Datasource 的 metric。</strong></p>
<p data-nodeid="1278"><img src="https://s0.lgstatic.com/i/image/M00/66/CF/Ciqc1F-f3TaACqNbAAIajUOGQMQ163.png" alt="Drawing 9.png" data-nodeid="1472"></p>
<p data-nodeid="1279">这个方法在我们做 granfan 图表的时候会很有用，不过需要注意的是：</p>
<p data-nodeid="1280">1.开启 prometheus 需要 gradle 额外引入下面这个包：</p>
<pre class="lang-java" data-nodeid="1281"><code data-language="java">implementation <span class="hljs-string">'io.micrometer:micrometer-registry-prometheus'</span>
</code></pre>
<p data-nodeid="1282">2.开启 Hibernate 的 statistics 需要配置如下操作：</p>
<pre class="lang-java" data-nodeid="1283"><code data-language="java">spring.jpa.properties.hibernate.generate_statistics=<span class="hljs-keyword">true</span>
management.endpoint.prometheus.enabled=<span class="hljs-keyword">true</span>
management.metrics.export.prometheus.enabled=<span class="hljs-keyword">true</span>
</code></pre>
<h3 data-nodeid="1284">总结</h3>
<p data-nodeid="1285">到这里，本课时的内容就告一段落了。通过上面的讲解，你会发现 Spring Data 为我们做了不少支持 MVC 的工作，帮助我们提升了很多开发效率；并且通过原理分析，你也知道了自定义 HttpMessageConverter 和 HandlerMethodArgumentResolver 的方法。</p>
<p data-nodeid="1286">我根据自身经验总结了上面的几个常见的 Web MVC 相关的 Case，当然了也可能有我没有想到的地方，欢迎你补充留言。</p>
<p data-nodeid="1287">最后如果你觉得有帮助就动动手指分享吧。下一课时，我们学习如何自定义 HandlerMethodArgumentResolvers，用来把请求参数结构化地传递到 Controller 的参数里面。到时见。</p>
<blockquote data-nodeid="1288">
<p data-nodeid="1289" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="1484">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *扬：
> 还真是只有非空的字段才能生成sql

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 多实践哦

##### *谨：
> 你好,请问下在使用@DynamicUpdate时会将BaseEntiy里的字段置空这种情况如何解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我在测试的时候没有发现这种情况呀，建议简化代码先排除是不是jpa清理的？

##### 韩：
> 请确认一下@DynamicUpdate这个注解，我实践后是只有更新的字段才会在update set里，并不是非空字段会在update set里。而且我看到java doc中解释的也是这个意思For updating, should this entity use dynamic sql generation where only changed columns get referenced in the prepared sql statement?Note, for re-attachment of detached entities this is not possible without select-before-update being enabled.

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 确实之前存在一些bug，我整体调整了一下，你可以看看。

