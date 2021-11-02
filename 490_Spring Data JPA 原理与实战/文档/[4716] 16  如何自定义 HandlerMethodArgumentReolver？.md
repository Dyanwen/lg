<p data-nodeid="318986" class="">上一讲我们介绍了 SpringDataWebConfiguration 类的用法，那么这次我们来看一下这个类是如何被加载的，PageableHandlerMethodArgumentResolver 和 SortHandlerMethodArgumentResolver 又是如何生效的，以及如何定义自己的 HandlerMethodArgumentResolvers 类，还有没有其他 Web 场景需要我们自定义呢？</p>


<p data-nodeid="318091">关于上述几个类，你要先在心里有点印象，我们接下来一个一个详细讲解。</p>
<h3 data-nodeid="318092">Page 和 Sort 参数原理</h3>
<p data-nodeid="320168">想要知道分页和排序参数的加载原理，我们可以通过源码发现是 @EnableSpringDataWebSupport 将这个类加载进去的，其关键代码如下图所示：</p>
<p data-nodeid="320169" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/F1/CgqCHl-ifKuAZqvLAAGaihgL6z0625.png" alt="Drawing 0.png" data-nodeid="320173"></p>


<p data-nodeid="318095">其中，@EnableSpringDataWebSupport 注解是上一讲讲解的核心，即 Spring Data JPA 对 Web 支持需要开启的入口，由于我们使用的是 Spring Boot，所以 @EnableSpringDataWebSupport 不需要我们手动去指定。</p>
<p data-nodeid="321354">这是由于 Spring Boot 有自动加载的机制，我们会发现 org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration 类里面引用了 @EnableSpringDataWebSupport 的注解，所以也不需要我们手动去引用了。这里面的关键代码如下图所示：</p>
<p data-nodeid="321355" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/E5/Ciqc1F-ifLGAXeScAACYeXQaXt0313.png" alt="Drawing 1.png" data-nodeid="321359"></p>


<p data-nodeid="322540">而 Spring Boot 的自动加载的核心文件就是 spring.factories 文件，那么我们打开 spring-boot-autoconfigure-2.3.3.jar 包，看一下 spring.factories 文件内容，可以找到 SpringDataWebAutoConfiguration 这个配置类，如下：</p>
<p data-nodeid="322541" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/F1/CgqCHl-ifLiATqQNAADQjUYmp3o182.png" alt="Drawing 2.png" data-nodeid="322545"></p>


<p data-nodeid="323726">所以可以得出结论：只要是 Spring Boot 项目，我们什么都不需要做，它就会天然地让 Spring Data JPA 支持 Web 相关的操作。</p>
<p data-nodeid="323727" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/F1/CgqCHl-ifL2AJfQTAAA5uE86eqs914.png" alt="Drawing 3.png" data-nodeid="323731"></p>


<p data-nodeid="324912">而 PageableHandlerMethodArgumentResolver 和 SortHandlerMethodArgumentResolver 两个类是通过 SpringDataWebConfiguration 加载进去的，所以我们基本可以知道 Spring Data JPA 的 Page 和 Sort 参数是因为 SpringDataWebConfiguration 里面 @Bean 的注入才生效的。</p>
<p data-nodeid="324913" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/E6/Ciqc1F-ifMKAEkZ7AAD0bB-3aYU721.png" alt="Drawing 4.png" data-nodeid="324917"></p>


<p data-nodeid="318104">通过 PageableHandlerMethodArgumentResolver 和 SortHandlerMethodArgumentResolver 这两个类的源码，我们可以分析出它们分别实现了 Spring MVC Web 框架里面的 org.springframework.web.method.support.HandlerMethodArgumentResolver 这个接口，从而对 Request 里面的 Page 和 Sort 的参数做了处理逻辑和解析逻辑。</p>
<p data-nodeid="318105">那么在实际工作中，可能存在特殊情况需要对其进行扩展，比如 Page 的参数可能需要支持多种 Key 的情况，那么我们应该怎么做呢？下面通过 HandlerMethodArgumentResolver 的用法来学习一下。</p>
<h3 data-nodeid="318106">HandlerMethodArgumentResolver 用法</h3>
<h4 data-nodeid="318107">HandlerMethodArgumentResolvers 详解</h4>
<p data-nodeid="318108">熟悉 MVC 的人都知道，HandlerMethodArgumentResolvers 在 Spring MVC 中的主要作用是对 Controller 里面的方法参数做解析，即可以把 Request 里面的值映射到方法的参数中。我们打开此类的源码会发现只有两个方法，如下所示：</p>
<pre class="lang-java" data-nodeid="328041"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">HandlerMethodArgumentResolver</span> </span>{
   <span class="hljs-comment">//检查方法的参数是否支持处理和转化</span>
   <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">supportsParameter</span><span class="hljs-params">(MethodParameter parameter)</span></span>;
   <span class="hljs-comment">//根据reqest上下文，解析方法的参数</span>
   <span class="hljs-function">Object <span class="hljs-title">resolveArgument</span><span class="hljs-params">(MethodParameter parameter, <span class="hljs-meta">@Nullable</span> ModelAndViewContainer mavContainer,
         NativeWebRequest webRequest, <span class="hljs-meta">@Nullable</span> WebDataBinderFactory binderFactory)</span> <span class="hljs-keyword">throws</span> Exception</span>;
}
</code></pre>
<p data-nodeid="329470">此接口的应用场景非常广泛，我们可以看到其子类非常多，如下图所示：</p>
<p data-nodeid="329471" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/F1/CgqCHl-ifNKAUr5ZAAKB24GVNXo607.png" alt="Drawing 5.png" data-nodeid="329475"></p>


<p data-nodeid="328044">其中几个类的作用如下：</p>
<ul data-nodeid="328045">
<li data-nodeid="328046">
<p data-nodeid="328047">PathVariableMapMethodArgumentResolver 专门解析 @PathVariable 里面的值；</p>
</li>
<li data-nodeid="328048">
<p data-nodeid="328049">RequestResponseBodyMethodProcessor 专门解析带 @RequestBody 注解的方法参数的值；</p>
</li>
<li data-nodeid="328050">
<p data-nodeid="328051">RequestParamMethodArgumentResolver 专门解析 @RequestParam 的注解参数的值，当方法的参数中没有任何注解的时候，默认是 @RequestParam；</p>
</li>
<li data-nodeid="328052">
<p data-nodeid="328053">以及我们上一讲提到的 PageableHandlerMethodArgumentResolver 和 SortHandlerMethodArgumentResolver。</p>
</li>
</ul>
<p data-nodeid="328054">到这里你会发现，我们上一讲还讲解了 HttpMessageConverter，那么它和 HandlerMethodArgumentResolvers 是什么关系呢？我们接着看。</p>
<h4 data-nodeid="328055">HandlerMethodArgumentResolvers 与 HttpMessageConverter 的关系</h4>
<p data-nodeid="330656">我们打开 RequestResponseBodyMethodProcessor 就会发现，这个类中主要处理的是，方法里面带 @RequestBody 注解的参数，如下图所示：</p>
<p data-nodeid="330657" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/E6/Ciqc1F-ifNqALsgiAAPVRBCs4Q4327.png" alt="Drawing 6.png" data-nodeid="330661"></p>


<p data-nodeid="328058">而其中的 readWithMessageConverters(webRequest, parameter, parameter.getNestedGenericParameterType()) 方法，如果我们点进去继续观察，发现里面会根据 Http 请求的 MediaType，来选择不同的 HttpMessageConverter 进行转化。</p>
<p data-nodeid="328059">所以到这里你可以很清楚 HandlerMethodArgumentResolvers 与 HttpMessageConverter 的关系了，即不同的 HttpMessageConverter 都是由 RequestResponseBodyMethodProcessor 进行调用的。</p>
<p data-nodeid="328060">那么调用关系我们知道了，如此多的 HttpMessageConverter 之间是通过什么顺序执行的呢？</p>
<h4 data-nodeid="328061">HttpMessageConverter 的执行顺序</h4>
<p data-nodeid="328062">当我们自定义 HandlerMethodArgumentResolver 时，通过下面的方法加载进去。</p>
<pre class="lang-java" data-nodeid="333659"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addArgumentResolvers</span><span class="hljs-params">(List&lt;HandlerMethodArgumentResolver&gt; resolvers)</span> </span>{
   resolvers.add(myPageableHandlerMethodArgumentResolver);
}
</code></pre>
<p data-nodeid="335664">在 List<code data-backticks="1" data-nodeid="335667">&lt;HandlerMethodArgumentResolver&gt;</code> 里面自定义的 resolver 的优先级是最高的，也就是会优先执行 HandlerMethodArgumentResolver 之后，才会按照顺序执行系统里面自带的那一批 HttpMessageConverter，按照 List 的循环顺序一个一个执行。</p>
<p data-nodeid="335665">Spring 里面有个执行效率问题，就是一旦一次执行找到了需要的 HandlerMethodArgumentResolver 的时候，利用 Spring 中的缓存机制，执行过程中就不会再遍历 List<code data-backticks="1" data-nodeid="335670">&lt;HandlerMethodArgumentResolver&gt;</code> 了，而是直接用上次找到的 HandlerMethodArgumentResolver，这样提升了执行效率。</p>



<p data-nodeid="336852">如果想要了解更多的 Resolver，你可以看下图这个类，我不一一细说了。</p>
<p data-nodeid="336853" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/F1/CgqCHl-ifQaAX5YQAAN2flcVp0c284.png" alt="Drawing 7.png" data-nodeid="336857"></p>


<p data-nodeid="333663">那么了解了这么多，能否举个实战的例子呢？</p>
<h3 data-nodeid="333664">自定义 HandlerMethodArgumentResolver 实战</h3>
<p data-nodeid="333665">在实际的工作中，你可能会遇到对老项目进行改版的工作，如果要我们把旧的 API 接口改造成 JPA 的技术实现，那么可能会出现需要新、老参数的问题。假设在实际场景中，我们 Page 的参数是 page[number]，而 page size 的参数是 page[size]，看看应该怎么做。</p>
<p data-nodeid="333666"><strong data-nodeid="333755">第一步：新建 MyPageableHandlerMethodArgumentResolver。</strong></p>
<p data-nodeid="333667">这个类的作用有两个：</p>
<ol data-nodeid="333668">
<li data-nodeid="333669">
<p data-nodeid="333670">用来兼容 ?page[size]=2&amp;page[number]=0 的参数情况；</p>
</li>
<li data-nodeid="333671">
<p data-nodeid="333672">支持 JPA 新的参数形式 ?size=2&amp;page=0。</p>
</li>
</ol>
<p data-nodeid="333673">我们通过自定义的 MyPageableHandlerMethodArgumentResolver 来实现这个需求，请看下面这段代码。</p>
<pre class="lang-java" data-nodeid="339684"><code data-language="java"><span class="hljs-comment">/**
 * 通过<span class="hljs-doctag">@Component</span>把此类加载到Spring的容器里面去 
 */</span>
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyPageableHandlerMethodArgumentResolver</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">PageableHandlerMethodArgumentResolver</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">HandlerMethodArgumentResolver</span> </span>{
   <span class="hljs-comment">//我们假设sort的参数没有发生变化，采用PageableHandlerMethodArgumentResolver里面的写法</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> SortHandlerMethodArgumentResolver DEFAULT_SORT_RESOLVER = <span class="hljs-keyword">new</span> SortHandlerMethodArgumentResolver();
   <span class="hljs-comment">//给定两个默认值</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Integer DEFAULT_PAGE = <span class="hljs-number">0</span>;
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Integer DEFAULT_SIZE = <span class="hljs-number">10</span>;
   <span class="hljs-comment">//兼容新版，引入JPA的分页参数</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String JPA_PAGE_PARAMETER = <span class="hljs-string">"page"</span>;
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String JPA_SIZE_PARAMETER = <span class="hljs-string">"size"</span>;
   <span class="hljs-comment">//兼容原来老的分页参数</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String DEFAULT_PAGE_PARAMETER = <span class="hljs-string">"page[number]"</span>;
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String DEFAULT_SIZE_PARAMETER = <span class="hljs-string">"page[size]"</span>;
   <span class="hljs-keyword">private</span> SortArgumentResolver sortResolver;
   <span class="hljs-comment">//模仿PageableHandlerMethodArgumentResolver里面的构造方法</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">MyPageableHandlerMethodArgumentResolver</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> SortArgumentResolver sortResolver)</span> </span>{
      <span class="hljs-keyword">this</span>.sortResolver = sortResolver == <span class="hljs-keyword">null</span> ? DEFAULT_SORT_RESOLVER : sortResolver;
   }
   
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supportsParameter</span><span class="hljs-params">(MethodParameter parameter)</span> </span>{
<span class="hljs-comment">//    假设用我们自己的类MyPageRequest接收参数</span>
      <span class="hljs-keyword">return</span> MyPageRequest.class.equals(parameter.getParameterType());
      <span class="hljs-comment">//同时我们也可以支持通过Spring Data JPA里面的Pageable参数进行接收，两种效果是一样的</span>
<span class="hljs-comment">//    return Pageable.class.equals(parameter.getParameterType());</span>
   }
   <span class="hljs-comment">/**
    * 参数封装逻辑page和sort，JPA参数的优先级高于page[number]和page[size]参数
    */</span>
    <span class="hljs-comment">//public Pageable resolveArgument(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory) { //这种是Pageable的方式</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> MyPageRequest <span class="hljs-title">resolveArgument</span><span class="hljs-params">(MethodParameter parameter, ModelAndViewContainer mavContainer, NativeWebRequest webRequest, WebDataBinderFactory binderFactory)</span> </span>{
      String jpaPageString = webRequest.getParameter(JPA_PAGE_PARAMETER);
      String jpaSizeString = webRequest.getParameter(JPA_SIZE_PARAMETER);
      <span class="hljs-comment">//我们分别取参数里面page、sort和 page[number]、page[size]的值</span>
      String pageString = webRequest.getParameter(DEFAULT_PAGE_PARAMETER);
      String sizeString = webRequest.getParameter(DEFAULT_SIZE_PARAMETER);
      <span class="hljs-comment">//当两个都有值时候的优先级，及其默认值的逻辑</span>
      Integer page = jpaPageString != <span class="hljs-keyword">null</span> ? Integer.valueOf(jpaPageString) : pageString != <span class="hljs-keyword">null</span> ? Integer.valueOf(pageString) : DEFAULT_PAGE;
      <span class="hljs-comment">//在这里同时可以计算 page+1的逻辑;如：page=page+1;</span>
      Integer size = jpaSizeString != <span class="hljs-keyword">null</span> ? Integer.valueOf(jpaSizeString) : sizeString != <span class="hljs-keyword">null</span> ? Integer.valueOf(sizeString) : DEFAULT_SIZE;
       <span class="hljs-comment">//我们假设，sort排序的取值方法先不发生改变</span>
      Sort sort = sortResolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
<span class="hljs-comment">//    如果使用Pageable参数接收值，我们也可以不用自定义MyPageRequest对象，直接返回PageRequest;</span>
<span class="hljs-comment">//    return PageRequest.of(page,size,sort);</span>
      <span class="hljs-comment">//将page和size计算出来的记过封装到我们自定义的MyPageRequest类里面去</span>
      MyPageRequest myPageRequest = <span class="hljs-keyword">new</span> MyPageRequest(page, size,sort);
      <span class="hljs-comment">//返回controller里面的参数需要的对象；</span>
      <span class="hljs-keyword">return</span> myPageRequest;
   }
}
</code></pre>
<p data-nodeid="339685">你可以通过代码里面的注释仔细看一下其中的逻辑，其实这个类并不复杂，就是取 Request 的 Page 相关的参数，封装到对象中返回给 Controller 的方法参数里面。其中 MyPageRequest 不是必需的，我只是为了给你演示不同的做法。</p>
<p data-nodeid="339686"><strong data-nodeid="339744">第二步：新建 MyPageRequest。</strong></p>
<pre class="lang-java" data-nodeid="342636"><code data-language="java"><span class="hljs-comment">/**
 * 继承父类，可以省掉很多计算page和index的逻辑
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyPageRequest</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">PageRequest</span> </span>{
   <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-title">MyPageRequest</span><span class="hljs-params">(<span class="hljs-keyword">int</span> page, <span class="hljs-keyword">int</span> size, Sort sort)</span> </span>{
      <span class="hljs-keyword">super</span>(page, size, sort);
   }
}
</code></pre>
<p data-nodeid="346284">此类，我们用来接收 Page 相关的参数值，也不是必需的。</p>
<p data-nodeid="346285"><strong data-nodeid="346290">第三步：implements WebMvcConfigurer 加载 myPageableHandlerMethodArgumentResolver。</strong></p>

<pre class="lang-java" data-nodeid="345556"><code data-language="java"><span class="hljs-comment">/**
 * 实现WebMvcConfigurer
 */</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyWebMvcConfigurer</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WebMvcConfigurer</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> MyPageableHandlerMethodArgumentResolver myPageableHandlerMethodArgumentResolver;
   <span class="hljs-comment">/**
    * 覆盖这个方法，把我们自定义的myPageableHandlerMethodArgumentResolver加载到原始的mvc的resolvers里面去
    * <span class="hljs-doctag">@param</span> resolvers
    */</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addArgumentResolvers</span><span class="hljs-params">(List&lt;HandlerMethodArgumentResolver&gt; resolvers)</span> </span>{
      resolvers.add(myPageableHandlerMethodArgumentResolver);
   }
}
</code></pre>
<p data-nodeid="345557">这里我利用 Spring MVC 的机制加载我们自定义的 myPageableHandlerMethodArgumentResolver，由于自定义的优先级是最高的，所以用 MyPageRequest.class</p>
<p data-nodeid="345558">和 Pageable.class 都是可以的。</p>
<p data-nodeid="345559"><strong data-nodeid="345612">第四步：我们看下 Controller 里面的写法。</strong></p>
<pre class="lang-plain" data-nodeid="345560"><code data-language="plain">//用Pageable这种方式也是可以的
@GetMapping("/users")
public Page&lt;UserInfo&gt; queryByPage(Pageable pageable, UserInfo userInfo) {
   return userInfoRepository.findAll(Example.of(userInfo),pageable);
}
//用MyPageRequest进行接收
@GetMapping("/users/mypage")
public Page&lt;UserInfo&gt; queryByMyPage(MyPageRequest pageable, UserInfo userInfo) {
   return userInfoRepository.findAll(Example.of(userInfo),pageable);
}
</code></pre>
<p data-nodeid="345561">你可以看到，这里利用 Pageable 和 MyPageRequest 两种方式都是可以的。<br>
<strong data-nodeid="345618">第五步：启动项目测试一下。</strong></p>
<p data-nodeid="345562">我们依次可以测试下面两种情况，发现都是可以正常工作的。</p>
<pre class="lang-plain" data-nodeid="345563"><code data-language="plain">GET http://127.0.0.1:8089/users?page[size]=2&amp;page[number]=0&amp;ages=10&amp;sort=id,desc
###
GET http://127.0.0.1:8089/users?size=2&amp;page=0&amp;ages=10&amp;sort=id,desc
###
GET http://127.0.0.1:8089/users/mypage?page[size]=2&amp;page[number]=0&amp;ages=10&amp;sort=id,desc
###
GET http://127.0.0.1:8089/users/mypage?size=2&amp;page=0&amp;ages=10&amp;sort=id,desc
</code></pre>
<p data-nodeid="346885">其中，你应该可以注意到，我演示的 Controller 方法里面有多个参数的，每个参数都各司其职，找到自己对应的 HandlerMethodArgumentResolver，这正是 Spring MVC 框架的优雅之处。</p>
<p data-nodeid="346886">那么除了上面的 Demo，自定义 HandlerMethodArgumentResolver 对我们的实际工作还有什么建议呢？</p>

<h3 data-nodeid="345565">实际工作的建议</h3>
<p data-nodeid="345566">自定义 HandlerMethodArgumentResolver 到底对我们的实际工作起到哪些作用呢？分为下述几个场景。</p>
<h4 data-nodeid="345567">场景一</h4>
<p data-nodeid="345568">当我们在 Controller 里面处理某些参数时，重复的步骤非常多，那么我们就可以考虑写一下自己的框架，来处理请求里面的参数，而 Controller 里面的代码就会变得非常优雅，不需要关心其他框架代码，只要知道方法的参数有值就可以了。</p>
<h4 data-nodeid="345569">场景二</h4>
<p data-nodeid="345570" class="">再举个例子，在实际工作中需要注意的是，默认 JPA 里面的 Page 是从 0 开始，而我们可能有些老的代码也要维护，因为老的代码大多数的 Page 都会从 1 开始。如果我们不自定义 HandlerMethodArgumentResolver，那么在用到分页时，每个 Controller 的方法里面都需要关心这个逻辑。那么这个时候你就应该想到上面列举的自定义 MyPageableHandlerMethodArgumentResolver 的 resolveArgument 方法的实现，使用这种方法我们只需要在里面修改 Page 的计算逻辑即可。</p>
<h4 data-nodeid="345571" class="">场景三</h4>
<p data-nodeid="345572" class="">再举个例子，在实际的工作中，还经常会遇到“取当前用户”的应用场景。此时，普通做法是，当使用到当前用户的 UserInfo 时，每次都需要根据请求 header 的 token 取到用户信息，伪代码如下所示：</p>
<pre class="lang-java" data-nodeid="349544"><code data-language="java"><span class="hljs-meta">@PostMapping("user/info")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfo</span><span class="hljs-params">(<span class="hljs-meta">@RequestHeader</span> String token)</span> </span>{
    <span class="hljs-comment">// 伪代码</span>
    Long userId = redisTemplate.get(token);
    UserInfo useInfo = userInfoRepository.getById(userId);
    <span class="hljs-keyword">return</span> userInfo;
}
</code></pre>
<p data-nodeid="349545">如果我们使用<code data-backticks="1" data-nodeid="349579">HandlerMethodArgumentResolver</code>接口来实现，代码就会变得优雅许多。伪代码如下：</p>
<pre class="lang-java" data-nodeid="352277"><code data-language="java"><span class="hljs-comment">// 1. 实现HandlerMethodArgumentResolver接口</span>
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoArgumentResolver</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">HandlerMethodArgumentResolver</span> </span>{
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> RedisTemplate redisTemplate;<span class="hljs-comment">//伪代码，假设我们token是放在redis里面的</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> UserInfoRepository userInfoRepository;
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">UserInfoArgumentResolver</span><span class="hljs-params">(RedisTemplate redisTemplate, UserInfoRepository userInfoRepository)</span> </span>{
      <span class="hljs-keyword">this</span>.redisTemplate = redisTemplate;<span class="hljs-comment">//伪代码，假设我们token是放在redis里面的</span>
      <span class="hljs-keyword">this</span>.userInfoRepository = userInfoRepository;
   }
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supportsParameter</span><span class="hljs-params">(MethodParameter parameter)</span> </span>{
      <span class="hljs-keyword">return</span> UserInfo.class.isAssignableFrom(parameter.getParameterType());
   }
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">resolveArgument</span><span class="hljs-params">(MethodParameter parameter, ModelAndViewContainer mavContainer,
                          NativeWebRequest webRequest, WebDataBinderFactory binderFactory)</span> <span class="hljs-keyword">throws</span> Exception </span>{
      HttpServletRequest nativeRequest = (HttpServletRequest) webRequest.getNativeRequest();
      String token = nativeRequest.getHeader(<span class="hljs-string">"token"</span>);
      Long userId = (Long) redisTemplate.opsForValue().get(token);<span class="hljs-comment">//伪代码，假设我们token是放在redis里面的</span>
      UserInfo useInfo = userInfoRepository.getOne(userId);
      <span class="hljs-keyword">return</span> useInfo;
   }
}
<span class="hljs-comment">//2. 我们只需要在MyWebMvcConfigurer里面把userInfoArgumentResolver添加进去即可，关键代码如下：</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyWebMvcConfigurer</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WebMvcConfigurer</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> MyPageableHandlerMethodArgumentResolver myPageableHandlerMethodArgumentResolver;
<span class="hljs-meta">@Autowired</span>
<span class="hljs-keyword">private</span> UserInfoArgumentResolver userInfoArgumentResolver;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addArgumentResolvers</span><span class="hljs-params">(List&lt;HandlerMethodArgumentResolver&gt; resolvers)</span> </span>{
   resolvers.add(myPageableHandlerMethodArgumentResolver);
   <span class="hljs-comment">//我们只需要把userInfoArgumentResolver加入resolvers中即可</span>
   resolvers.add(userInfoArgumentResolver);
}
}
<span class="hljs-comment">// 3. 在Controller中使用</span>
<span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoController</span> </span>{
  <span class="hljs-comment">//获得当前用户的信息</span>
  <span class="hljs-meta">@GetMapping("user/info")</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfo</span><span class="hljs-params">(UserInfo userInfo)</span> </span>{
     <span class="hljs-keyword">return</span> userInfo;
  }
  <span class="hljs-comment">//给当前用户 say hello</span>
  <span class="hljs-meta">@PostMapping("sayHello")</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">sayHello</span><span class="hljs-params">(UserInfo userInfo)</span> </span>{
    <span class="hljs-keyword">return</span> <span class="hljs-string">"hello "</span> + userInfo.getTelephone();
  }
}
</code></pre>
<p data-nodeid="352278">上述代码可以看到，在 Contoller 里面可以完全省掉根据 token 从 redis 取当前用户信息的过程，优化了操作流程。</p>
<h4 data-nodeid="352279">场景四</h4>
<p data-nodeid="353545">有的时候我们也会更改 Pageable 的默认值和参数的名字，也可以在 application.properties 的文件里面通过如下的 Key 值对自定义进行配置，如下图所示：</p>
<p data-nodeid="353546" class=""><img src="https://s0.lgstatic.com/i/image/M00/67/E6/Ciqc1F-ifTSAY0xeAAIfdBh0SkQ798.png" alt="Drawing 8.png" data-nodeid="353550"></p>


<p data-nodeid="352282">关于 Spring MVC 和 Spring Data 相关的参数处理，你通过了解上面的内容并动手操作一下，基本上就可以掌握了。但是实际工作肯定不会这么简单，还会遇到 WebMvcConfigurer 里面其他方法的需求，我顺带给你介绍一下。</p>
<h3 data-nodeid="352283">思路拓展</h3>
<h4 data-nodeid="352284">WebMvcConfigurer 介绍</h4>
<p data-nodeid="352285">当我们做 Spring 的 MVC 开发的时候，可能会通过实现 WebMvcConfigurer 去做一些公用的业务逻辑，下面我列举几个常见的方法，方便你了解。</p>
<pre class="lang-java" data-nodeid="356134"><code data-language="java"> <span class="hljs-comment">/* 拦截器配置 */</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">addInterceptors</span><span class="hljs-params">(InterceptorRegistry var1)</span></span>;
<span class="hljs-comment">/* 视图跳转控制器 */</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">addViewControllers</span><span class="hljs-params">(ViewControllerRegistry registry)</span></span>;
<span class="hljs-comment">/**
  *静态资源处理
**/</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">addResourceHandlers</span><span class="hljs-params">(ResourceHandlerRegistry registry)</span></span>;
<span class="hljs-comment">/* 默认静态资源处理器 */</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">configureDefaultServletHandling</span><span class="hljs-params">(DefaultServletHandlerConfigurer configurer)</span></span>;
<span class="hljs-comment">/**
  *这里配置视图解析器
 **/</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">configureViewResolvers</span><span class="hljs-params">(ViewResolverRegistry registry)</span></span>;
<span class="hljs-comment">/* 配置内容裁决的一些选项*/</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">configureContentNegotiation</span><span class="hljs-params">(ContentNegotiationConfigurer configurer)</span></span>;
<span class="hljs-comment">/** 解决跨域问题 **/</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">addCorsMappings</span><span class="hljs-params">(CorsRegistry registry)</span> </span>;
<span class="hljs-comment">/** 添加都会contoller的Return的结果的处理 **/</span>
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">addReturnValueHandlers</span><span class="hljs-params">(List&lt;HandlerMethodReturnValueHandler&gt; handlers)</span>；
</span></code></pre>
<p data-nodeid="356135">当我们实现 Restful 风格的 API 协议时，会经常看到其对 json 响应结果进行了统一的封装，我们也可以采用 HandlerMethodReturnValueHandler 来实现，再来看一个例子。</p>
<h4 data-nodeid="356136">用 Result 对 JSON 的返回结果进行统一封装</h4>
<p data-nodeid="356137">下面通过五个步骤来实现一个通过自定义注解，利用<strong data-nodeid="356164">HandlerMethodReturnValueHandler 实现 JSON 结果封装</strong>的例子。</p>
<p data-nodeid="356138"><strong data-nodeid="356169">第一步：我们自定义一个注解 @WarpWithData</strong>，表示此注解包装的返回结果用 Data 进行包装，代码如下：</p>
<pre class="lang-plain" data-nodeid="356139"><code data-language="plain">@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
/**
 * 自定义一个注解对返回结果进行包装
 */
public @interface WarpWithData {
}
</code></pre>
<p data-nodeid="356140"><strong data-nodeid="356174">第二步：自定义 MyWarpWithDataHandlerMethodReturnValueHandler，并继承 RequestResponseBodyMethodProcessor 来实现 HandlerMethodReturnValueHandler 接口</strong>，用来处理 Data 包装的结果，代码如下：</p>
<pre class="lang-java" data-nodeid="356797"><code data-language="java"><span class="hljs-comment">//自定义自己的return的处理类，我们直接继承RequestResponseBodyMethodProcessor，这样父类里面的方法我们直接使用就可以了</span>
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyWarpWithDataHandlerMethodReturnValueHandler</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">RequestResponseBodyMethodProcessor</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">HandlerMethodReturnValueHandler</span> </span>{
   <span class="hljs-comment">//参考父类RequestResponseBodyMethodProcessor的做法</span>
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">MyWarpWithDataHandlerMethodReturnValueHandler</span><span class="hljs-params">(List&lt;HttpMessageConverter&lt;?&gt;&gt; converters)</span> </span>{
      <span class="hljs-keyword">super</span>(converters);
   }
   <span class="hljs-comment">//只处理需要包装的注解的方法</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supportsReturnType</span><span class="hljs-params">(MethodParameter returnType)</span> </span>{
      <span class="hljs-keyword">return</span> returnType.hasMethodAnnotation(WarpWithData.class);
   }
   <span class="hljs-comment">//将返回结果包装一层Data</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">handleReturnValue</span><span class="hljs-params">(Object returnValue, MethodParameter methodParameter, ModelAndViewContainer modelAndViewContainer, NativeWebRequest nativeWebRequest)</span> <span class="hljs-keyword">throws</span> IOException, HttpMediaTypeNotAcceptableException </span>{
      Map&lt;String,Object&gt; res = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
      res.put(<span class="hljs-string">"data"</span>,returnValue);
      <span class="hljs-keyword">super</span>.handleReturnValue(res,methodParameter,modelAndViewContainer,nativeWebRequest);
   }
}
</code></pre>
<p data-nodeid="356798"><strong data-nodeid="356817">第三步：在 MyWebMvcConfigurer 里面直接把 myWarpWithDataHandlerMethodReturnValueHandler 加入 handlers 里面即可</strong>，也是通过覆盖父类 WebMvcConfigurer 里面的 addReturnValueHandlers 方法完成的，关键代码如下：</p>
<pre class="lang-java" data-nodeid="357435"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MyWebMvcConfigurer</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">WebMvcConfigurer</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> MyWarpWithDataHandlerMethodReturnValueHandler myWarpWithDataHandlerMethodReturnValueHandler;
   <span class="hljs-comment">//把我们自定义的myWarpWithDataHandlerMethodReturnValueHandler加入handlers里面即可</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addReturnValueHandlers</span><span class="hljs-params">(List&lt;HandlerMethodReturnValueHandler&gt; handlers)</span> </span>{
      handlers.add(myWarpWithDataHandlerMethodReturnValueHandler);
   }
   
  <span class="hljs-meta">@Autowired</span>
  <span class="hljs-keyword">private</span> RequestMappingHandlerAdapter requestMappingHandlerAdapter;
  <span class="hljs-comment">//由于HandlerMethodReturnValueHandler处理的优先级问题，我们通过如下方法，把我们自定义的myWarpWithDataHandlerMethodReturnValueHandler放到第一个；</span>
  <span class="hljs-meta">@PostConstruct</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
     List&lt;HandlerMethodReturnValueHandler&gt; returnValueHandlers = Lists.newArrayList(myWarpWithDataHandlerMethodReturnValueHandler);
<span class="hljs-comment">//取出原始列表，重新覆盖进去；</span>
        returnValueHandlers.addAll(requestMappingHandlerAdapter.getReturnValueHandlers());
     requestMappingHandlerAdapter.setReturnValueHandlers(returnValueHandlers);
  }
}
</code></pre>
<p data-nodeid="358688">这里需要注意的是，我们利用 @PostConstruct 调整了一下 HandlerMethodReturnValueHandler 加载的优先级，使其生效。</p>
<p data-nodeid="358689"><strong data-nodeid="358695">第四步：Controller 方法中直接加上 @WarpWithData 注解</strong>，关键代码如下：</p>

<pre class="lang-java" data-nodeid="358066"><code data-language="java"><span class="hljs-meta">@GetMapping("/user/{id}")</span>
<span class="hljs-meta">@WarpWithData</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromPath</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> Long id)</span> </span>{
   <span class="hljs-keyword">return</span> userInfoRepository.getOne(id);
}
</code></pre>
<p data-nodeid="358067"><strong data-nodeid="358081">第五步：我们测试一下。</strong></p>
<pre class="lang-java" data-nodeid="359290"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8089/user/1</span>
</code></pre>
<p data-nodeid="359291">就会得到如下结果，你会发现我们的 JSON 结果多了一个 Data 的包装。</p>
<pre class="lang-java" data-nodeid="359906"><code data-language="java">{
&nbsp; <span class="hljs-string">"data"</span>: {
&nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
&nbsp; &nbsp; <span class="hljs-string">"version"</span>: <span class="hljs-number">0</span>,
&nbsp; &nbsp; <span class="hljs-string">"createUserId"</span>: <span class="hljs-keyword">null</span>,
&nbsp; &nbsp; <span class="hljs-string">"createTime"</span>: <span class="hljs-string">"2020-10-23T00:23:10.185Z"</span>,
&nbsp; &nbsp; <span class="hljs-string">"lastModifiedUserId"</span>: <span class="hljs-keyword">null</span>,
&nbsp; &nbsp; <span class="hljs-string">"lastModifiedTime"</span>: <span class="hljs-string">"2020-10-23T00:23:10.185Z"</span>,
&nbsp; &nbsp; <span class="hljs-string">"ages"</span>: <span class="hljs-number">10</span>,
&nbsp; &nbsp; <span class="hljs-string">"telephone"</span>: <span class="hljs-keyword">null</span>,
&nbsp; &nbsp; <span class="hljs-string">"hibernateLazyInitializer"</span>: {}
&nbsp; }
}
</code></pre>
<p data-nodeid="359907">我们通过五个步骤，利用 Spring MVC 的扩展机制，实现了对返回结果的格式统一处理。不知道你是否掌握了这种方法，希望你可以多多实践，将它运用得更好。</p>
<h3 data-nodeid="359908">总结</h3>
<p data-nodeid="359909">以上就是这一讲的内容了。在这一讲中，我通过原理分析、语法讲解、实战经验分享，帮助你掌握了 HandlerMethodArgumentResolvers 的详细用法，并为你扩展了学习思路，了解了 HandlerMethodReturnValueHandler 的用法。</p>
<p data-nodeid="359910">其实 Spring MVC 肯定远不止这些，这里我只介绍了一些和 Spring Data 相关的知识点。你在工作和学习中，要时刻保持好奇心和挖掘精神，不断地探究不理解的知识点。</p>
<p data-nodeid="359911">最后，如果你觉得有帮助就动动手指分享吧，也欢迎你在评论区留言，一起讨论、进步。下一讲我们将学习数据源相关的知识。到时见~</p>
<blockquote data-nodeid="359912">
<p data-nodeid="359913">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="359924">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *信：
> Spring mvc框架还是蛮优雅的

