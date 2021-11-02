<p data-nodeid="6705" class="">Spring Data JPA 的最大特色是利用<strong data-nodeid="6842">方法名定义查询方法</strong>（Defining Query Methods）来做 CRUD 操作，这一课时我将围绕这个内容来详细讲解。</p>
<p data-nodeid="6706">在工作中，你是否经常为方法名的语义、命名规范而发愁？是否要为不同的查询条件写各种的 SQL 语句？是否为同一个实体的查询，写一个超级通用的查询方法或者 SQL？如果其他开发同事不查看你写的 SQL 语句，而直接看方法名的话，却不知道你想查什么而郁闷？</p>
<p data-nodeid="6707">Spring Data JPA 的 Defining Query Methods（DQM）通过方法名和参数，可以很好地解决上面的问题，也能让我们的方法名的语义更加清晰，开发效率也会提升很多。DQM 语法共有 2 种，可以实现上面的那些问题，具体如下：</p>
<ul data-nodeid="6708">
<li data-nodeid="6709">
<p data-nodeid="6710">一种是直接通过方法名就可以实现，这也是本课时会详细介绍的重点内容；</p>
</li>
<li data-nodeid="6711">
<p data-nodeid="6712">另一种是 @Query 手动在方法上定义，这将在第 05 课时“@Query 帮我们解决了什么问题？什么时候应该选择 @Query?”中详细介绍。</p>
</li>
</ul>
<p data-nodeid="6713">下面我将从 6 个方面来详细讲解 Defining Query Methods。先来分析一下“定义查询方法的配置和使用方法”，这个是 Defining Query Methods 中必须要掌握的语法。</p>
<h3 data-nodeid="6714">定义查询方法的配置和使用方法</h3>
<p data-nodeid="6715">若想要实现 CRUD 的操作，常规做法是写一大堆 SQL 语句。但在 JPA 里面，只需要继承 Spring Data Common 里面的任意 Repository 接口或者子接口，然后直接通过方法名就可以实现，神不神奇？来看下面具体的使用步骤。</p>
<p data-nodeid="6716">第 1 步，User 实体的 UserRepository 继承 Spring Data Common 里面的 Repository 接口：</p>
<pre class="lang-java" data-nodeid="6717"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>&gt; </span>{
     <span class="hljs-function">User <span class="hljs-title">findByEmailAddress</span><span class="hljs-params">(String emailAddress)</span></span>;
}
</code></pre>
<p data-nodeid="6718">第 2 步，对于 Service 层就可以直接使用 UserRepository 接口：</p>
<pre class="lang-java" data-nodeid="6719"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserServiceImpl</span></span>{
    <span class="hljs-meta">@Autowired</span>
    UserRepository userRepository;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testJpa</span><span class="hljs-params">()</span> </span>{
        userRepository.deleteAll();
        userRepository.findAll();
        userRepository.findByEmailAddress(<span class="hljs-string">"zjk@126.com"</span>);
    }
</code></pre>
<p data-nodeid="6720">这个时候就可以直接调用 CrudRepository 里面暴露的所有接口方法，以及 UserRepository 里面定义的方法，不需要写任何 SQL 语句，也不需要写任何实现方法。通过上面的两步我们完成了 Defining Query Methods（DQM）的基本使用，下面来看另外一种情况：<strong data-nodeid="6857">选择性暴露方法</strong>。</p>
<p data-nodeid="6721">然而，有时如果不想暴露 CrudRepository 里面的所有方法，那么可以直接继承我们认为需要暴露的那些方法的接口。假如 UserRepository 只想暴露 findOne 和 save，除了这两个方法之外不允许任何的 User 操作，其做法如下。</p>
<p data-nodeid="6722">我们选择性地暴露 CRUD 方法，直接继承Repository（因为这里面没有任何方法），把CrudRepository 里面的 save 和 findOne 方法复制到我们自己的 MyBaseRepository 接口即可，代码如下：</p>
<pre class="lang-java" data-nodeid="6723"><code data-language="java"><span class="hljs-meta">@NoRepositoryBean</span>
<span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">MyBaseRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Serializable</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
    <span class="hljs-function">T <span class="hljs-title">findOne</span><span class="hljs-params">(ID id)</span></span>; 
    <span class="hljs-function">T <span class="hljs-title">save</span><span class="hljs-params">(T entity)</span></span>;
}
<span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">MyBaseRepository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>&gt; </span>{
     <span class="hljs-function">User <span class="hljs-title">findByEmailAddress</span><span class="hljs-params">(String emailAddress)</span></span>;
}
</code></pre>
<p data-nodeid="6724">这样在 Service 层就只有 findOne、save、findByEmailAddress 这 3 个方法可以调用，不会有更多方法了，我们可以对 SimpleJpaRepository 里面任意已经实现的方法做选择性暴露。</p>
<p data-nodeid="6725">综上所述，得出以下 2 点结论：</p>
<ul data-nodeid="6726">
<li data-nodeid="6727">
<p data-nodeid="6728">MyRepository Extends Repository 接口可以实现 Defining Query Methods 的功能；</p>
</li>
<li data-nodeid="6729">
<p data-nodeid="6730">继承其他 Repository 的子接口，或者自定义子接口，可以选择性地暴露 SimpleJpaRepository 里面已经实现的基础公用方法。</p>
</li>
</ul>
<p data-nodeid="6731">在平时的工作中，你可以通过方法名，或者定义方法名上面添加 @Query 注解两种方式来实现 CRUD 的目的，而 Spring 给我们提供了两种切换方式。接下来我们就讲讲“方法的查询策略设置”。</p>
<h3 data-nodeid="6732">方法的查询策略设置</h3>
<p data-nodeid="6733">目前在实际生产中还没有遇到要修改默认策略的情况，但我们必须要知道有这样的配置方法，做到心中有数，这样我们才能知道为什么方法名可以，@Query 也可以。通过 @EnableJpaRepositories 注解来配置方法的查询策略，详细配置方法如下：</p>
<pre class="lang-java" data-nodeid="6734"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)</span>
</code></pre>
<p data-nodeid="6735">其中，QueryLookupStrategy.Key 的值共 3 个，具体如下：</p>
<ul data-nodeid="6736">
<li data-nodeid="6737">
<p data-nodeid="6738"><strong data-nodeid="6872">Create</strong>：直接根据方法名进行创建，规则是根据方法名称的构造进行尝试，一般的方法是从方法名中删除给定的一组已知前缀，并解析该方法的其余部分。如果方法名不符合规则，启动的时候会报异常，这种情况可以理解为，即使配置了 @Query 也是没有用的。</p>
</li>
<li data-nodeid="6739">
<p data-nodeid="6740"><strong data-nodeid="6881">USE_DECLARED_QUERY</strong>：声明方式创建，启动的时候会尝试找到一个声明的查询，如果没有找到将抛出一个异常，可以理解为必须配置 @Query。</p>
</li>
<li data-nodeid="6741">
<p data-nodeid="6742"><strong data-nodeid="6892">CREATE_IF_NOT_FOUND</strong>：这个是默认的，除非有特殊需求，可以理解为这是以上 2 种方式的兼容版。先用声明方式（@Query）进行查找，如果没有找到与方法相匹配的查询，那用 Create 的方法名创建规则创建一个查询；这两者都不满足的情况下，启动就会报错。</p>
</li>
</ul>
<p data-nodeid="6743">以 Spring Boot 项目为例，更改其配置方法如下：</p>
<pre class="lang-java" data-nodeid="6744"><code data-language="java"><span class="hljs-meta">@EnableJpaRepositories(queryLookupStrategy= QueryLookupStrategy.Key.CREATE_IF_NOT_FOUND)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Example1Application</span> </span>{
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
      SpringApplication.run(Example1Application.class, args);
   }
}
</code></pre>
<p data-nodeid="6745">以上就是方法的查询策略设置，很简单。接下来我们再讲讲“Defining Query Method（DQM）语法”，这是可以让方法生效的详细语法。</p>
<h3 data-nodeid="6746">Defining Query Method（DQM）语法</h3>
<p data-nodeid="6747">该语法是：带查询功能的方法名由查询策略（关键字）+ 查询字段 + 一些限制性条件组成，具有语义清晰、功能完整的特性，我们实际工作中 80% 的 API 查询都可以简单实现。</p>
<p data-nodeid="6748">我们来看一个复杂点的例子，这是一个 and 条件更多、distinct or 排序、忽略大小写的例子。下面代码定义了 PersonRepository，我们可以在 service 层直接使用，如下所示：</p>
<pre class="lang-java" data-nodeid="6749"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PersonRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>&gt; </span>{
   <span class="hljs-comment">// and 的查询关系</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByEmailAddressAndLastname</span><span class="hljs-params">(EmailAddress emailAddress, String lastname)</span></span>;
   <span class="hljs-comment">// 包含 distinct 去重，or 的 sql 语法</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findDistinctPeopleByLastnameOrFirstname</span><span class="hljs-params">(String lastname, String firstname)</span></span>;
   <span class="hljs-comment">// 根据 lastname 字段查询忽略大小写</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastnameIgnoreCase</span><span class="hljs-params">(String lastname)</span></span>;
   <span class="hljs-comment">// 根据 lastname 和 firstname 查询 equal 并且忽略大小写</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastnameAndFirstnameAllIgnoreCase</span><span class="hljs-params">(String lastname, String firstname)</span></span>; 
  <span class="hljs-comment">// 对查询结果根据 lastname 排序，正序</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastnameOrderByFirstnameAsc</span><span class="hljs-params">(String lastname)</span></span>;
&nbsp; <span class="hljs-comment">// 对查询结果根据 lastname 排序，倒序</span>
   <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastnameOrderByFirstnameDesc</span><span class="hljs-params">(String lastname)</span></span>;
}
</code></pre>
<p data-nodeid="6750">下面表格是一个我们在上面 DQM 方法语法里常用的关键字列表，方便你快速查阅，并满足在实际代码中更加复杂的场景：</p>
<p data-nodeid="6751"><img src="https://s0.lgstatic.com/i/image/M00/51/31/Ciqc1F9ki9CAPfoLAAMOpmuNPDY563.png" alt="Lark20200918-182821.png" data-nodeid="6901"></p>
<p data-nodeid="6752">综上，总结 3 点经验：</p>
<ul data-nodeid="6753">
<li data-nodeid="6754">
<p data-nodeid="6755">方法名的表达式通常是实体属性连接运算符的组合，如 And、or、Between、LessThan、GreaterThan、Like 等属性连接运算表达式，不同的数据库（NoSQL、MySQL）可能产生的效果不一样，如果遇到问题，我们可以打开 SQL 日志观察。</p>
</li>
<li data-nodeid="6756">
<p data-nodeid="6757">IgnoreCase 可以针对单个属性（如 findByLastnameIgnoreCase(…)），也可以针对查询条件里面所有的实体属性忽略大小写（所有属性必须在 String 情况下，如 findByLastnameAndFirstnameAllIgnoreCase(…)）。</p>
</li>
<li data-nodeid="6758">
<p data-nodeid="6759">OrderBy 可以在某些属性的排序上提供方向（Asc 或 Desc），称为静态排序，也可以通过一个方便的参数 Sort 实现指定字段的动态排序的查询方法（如 repository.findAll(Sort.by(Sort.Direction.ASC, "myField"))）。</p>
</li>
</ul>
<p data-nodeid="6760">我们看到上面的表格虽然大多是 find 开头的方法，除此之外，JPA 还支持read、get、query、stream、count、exists、delete、remove等前缀，如字面意思一样。我们来看看 count、delete、remove 的例子，其他前缀可以举一反三。实例代码如下：</p>
<pre class="lang-java" data-nodeid="6761"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>&gt; </span>{
     <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">countByLastname</span><span class="hljs-params">(String lastname)</span></span>;<span class="hljs-comment">//查询总数</span>
     <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">deleteByLastname</span><span class="hljs-params">(String lastname)</span></span>;<span class="hljs-comment">//根据一个字段进行删除操作，并返回删除行数</span>
     <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">removeByLastname</span><span class="hljs-params">(String lastname)</span></span>;<span class="hljs-comment">//根据Lastname删除一堆User,并返回删除的User</span>
}
</code></pre>
<p data-nodeid="6762">有的时候随着版本的更新，也会有更多的语法支持，或者不同的版本语法可能也不一样，我们通过源码来看一下上面说的几种语法。感兴趣的同学可以到类 org.springframework.data.repository.query.parser.PartTree 查看相关源码的逻辑和处理方法，关键源码如下：</p>
<p data-nodeid="6763"><img src="https://s0.lgstatic.com/i/image/M00/51/31/Ciqc1F9kjAWAfjJiAAc5lsBJToo426.png" alt="Drawing 0.png" data-nodeid="6914"></p>
<p data-nodeid="6764">根据源码我们也可以分析出来，query method 包含其他的表达式，比如 find、count、delete、exist 等关键字在 by 之前通过正则表达式匹配。</p>
<p data-nodeid="6765"><img src="https://s0.lgstatic.com/i/image/M00/51/31/Ciqc1F9kjA6AFsC1AASo47BSWUo068.png" alt="Drawing 1.png" data-nodeid="6918"></p>
<p data-nodeid="6766">由此可知，我们方法中的关键字不是乱填的，是枚举帮我们定义好的。接下来打开枚举类 Type 源码看下，比什么都清楚。</p>
<pre class="lang-java" data-nodeid="6767"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">enum</span> Type {
    BETWEEN(<span class="hljs-number">2</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsBetween"</span>, <span class="hljs-string">"Between"</span>}),
    IS_NOT_NULL(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNotNull"</span>, <span class="hljs-string">"NotNull"</span>}),
    IS_NULL(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNull"</span>, <span class="hljs-string">"Null"</span>}),
    LESS_THAN(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsLessThan"</span>, <span class="hljs-string">"LessThan"</span>}),
    LESS_THAN_EQUAL(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsLessThanEqual"</span>, <span class="hljs-string">"LessThanEqual"</span>}),
    GREATER_THAN(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsGreaterThan"</span>, <span class="hljs-string">"GreaterThan"</span>}),
    GREATER_THAN_EQUAL(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsGreaterThanEqual"</span>, <span class="hljs-string">"GreaterThanEqual"</span>}),
    BEFORE(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsBefore"</span>, <span class="hljs-string">"Before"</span>}),
    AFTER(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsAfter"</span>, <span class="hljs-string">"After"</span>}),
    NOT_LIKE(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNotLike"</span>, <span class="hljs-string">"NotLike"</span>}),
    LIKE(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsLike"</span>, <span class="hljs-string">"Like"</span>}),
    STARTING_WITH(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsStartingWith"</span>, <span class="hljs-string">"StartingWith"</span>, <span class="hljs-string">"StartsWith"</span>}),
    ENDING_WITH(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsEndingWith"</span>, <span class="hljs-string">"EndingWith"</span>, <span class="hljs-string">"EndsWith"</span>}),
    IS_NOT_EMPTY(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNotEmpty"</span>, <span class="hljs-string">"NotEmpty"</span>}),
    IS_EMPTY(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsEmpty"</span>, <span class="hljs-string">"Empty"</span>}),
    NOT_CONTAINING(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNotContaining"</span>, <span class="hljs-string">"NotContaining"</span>, <span class="hljs-string">"NotContains"</span>}),
    CONTAINING(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsContaining"</span>, <span class="hljs-string">"Containing"</span>, <span class="hljs-string">"Contains"</span>}),
    NOT_IN(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNotIn"</span>, <span class="hljs-string">"NotIn"</span>}),
    IN(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsIn"</span>, <span class="hljs-string">"In"</span>}),
    NEAR(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNear"</span>, <span class="hljs-string">"Near"</span>}),
    WITHIN(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsWithin"</span>, <span class="hljs-string">"Within"</span>}),
    REGEX(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"MatchesRegex"</span>, <span class="hljs-string">"Matches"</span>, <span class="hljs-string">"Regex"</span>}),
    EXISTS(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"Exists"</span>}),
    TRUE(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsTrue"</span>, <span class="hljs-string">"True"</span>}),
    FALSE(<span class="hljs-number">0</span>, <span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsFalse"</span>, <span class="hljs-string">"False"</span>}),
    NEGATING_SIMPLE_PROPERTY(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"IsNot"</span>, <span class="hljs-string">"Not"</span>}),
    SIMPLE_PROPERTY(<span class="hljs-keyword">new</span> String[]{<span class="hljs-string">"Is"</span>, <span class="hljs-string">"Equals"</span>});
....}
</code></pre>
<p data-nodeid="6768">看源码就可以知道框架支持了哪些逻辑关键字，比如 NotIn、Like、In、Exists 等，有的时候比查文档和任何人写的博客都准确、还快。好了，上面介绍了方面名的基本表达方式，希望你可以在工作中灵活运用，举一反三。接下来我们讲讲特定类型的参数：Sort 排序和 Pageable 分页，这是分页和排序必备技能。</p>
<h3 data-nodeid="6769">特定类型的参数：Sort 排序和 Pageable 分页</h3>
<p data-nodeid="6770">Spring Data JPA 为了方便我们排序和分页，支持了两个特殊类型的参数：Sort 和 Pageable。</p>
<p data-nodeid="6771">Sort 在查询的时候可以实现动态排序，我们看下其源码：</p>
<pre class="lang-java" data-nodeid="6772"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">Sort</span><span class="hljs-params">(Direction direction, String... properties)</span> </span>{
   <span class="hljs-keyword">this</span>(direction, properties == <span class="hljs-keyword">null</span> ? <span class="hljs-keyword">new</span> ArrayList&lt;&gt;() : Arrays.asList(properties));
}
</code></pre>
<p data-nodeid="6773">Sort 里面决定了我们哪些字段的排序方向（ASC 正序、DESC 倒序）。<br>
Pageable 在查询的时候可以实现分页效果和动态排序双重效果，我们看下 Pageable 的 Structure，如下图所示：</p>
<p data-nodeid="6774"><img src="https://s0.lgstatic.com/i/image/M00/51/31/Ciqc1F9kjB-AVHUCAADjYUn04XE249.png" alt="Drawing 2.png" data-nodeid="6929"></p>
<p data-nodeid="6775">我们发现 Pageable 是一个接口，里面有常见的分页方法排序、当前页、下一行、当前指针、一共多少页、页码、pageSize 等。</p>
<p data-nodeid="6776">在查询方法中如何使用 Pageable 和 Sort 呢？下面代码定义了根据 Lastname 查询 User 的分页和排序的实例，此段代码是在 UserRepository 接口里面定义的方法：</p>
<pre class="lang-java" data-nodeid="6777"><code data-language="java"><span class="hljs-function">Page&lt;User&gt; <span class="hljs-title">findByLastname</span><span class="hljs-params">(String lastname, Pageable pageable)</span></span>;<span class="hljs-comment">//根据分页参数查询User，返回一个带分页结果的Page(下一课时详解)对象（方法一）</span>
<span class="hljs-function">Slice&lt;User&gt; <span class="hljs-title">findByLastname</span><span class="hljs-params">(String lastname, Pageable pageable)</span></span>;<span class="hljs-comment">//我们根据分页参数返回一个Slice的user结果（方法二）</span>
<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastname</span><span class="hljs-params">(String lastname, Sort sort)</span></span>;<span class="hljs-comment">//根据排序结果返回一个List（方法三）</span>
<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByLastname</span><span class="hljs-params">(String lastname, Pageable pageable)</span></span>;<span class="hljs-comment">//根据分页参数返回一个List对象（方法四）</span>
</code></pre>
<p data-nodeid="6778"><strong data-nodeid="6936">方法一</strong>：允许将 org.springframework.data.domain.Pageable 实例传递给查询方法，将分页参数添加到静态定义的查询中，通过 Page 返回的结果得知可用的元素和页面的总数。这种分页查询方法可能是昂贵的（会默认执行一条 count 的 SQL 语句），所以用的时候要考虑一下使用场景。</p>
<p data-nodeid="6779"><strong data-nodeid="6941">方法二</strong>：返回结果是 Slice，因为只知道是否有下一个 Slice 可用，而不知道 count，所以当查询较大的结果集时，只知道数据是足够的，也就是说用在业务场景中时不用关心一共有多少页。</p>
<p data-nodeid="6780"><strong data-nodeid="6946">方法三</strong>：如果只需要排序，需在 org.springframework.data.domain.Sort 参数中添加一个参数，正如上面看到的，只需返回一个 List 也是有可能的。</p>
<p data-nodeid="6781"><strong data-nodeid="6951">方法四</strong>：排序选项也通过 Pageable 实例处理，在这种情况下，Page 将不会创建构建实际实例所需的附加元数据（即不需要计算和查询分页相关数据），而仅仅用来做限制查询给定范围的实体。</p>
<p data-nodeid="6782">那么如何使用呢？我们再来看一下源码，也就是 Pageable 的实现类，如下图所示：</p>
<p data-nodeid="6783"><img src="https://s0.lgstatic.com/i/image/M00/51/3C/CgqCHl9kjDSAFKPVAASB48M-k1w197.png" alt="Drawing 3.png" data-nodeid="6955"></p>
<p data-nodeid="6784">由此可知，我们可以通过 PageRequest 里面提供的几个 of 静态方法（多态），分别构建页码、页面大小、排序等。我们来看下，在使用中的写法，如下所示：</p>
<pre class="lang-java" data-nodeid="6785"><code data-language="java"><span class="hljs-comment">//查询user里面的lastname=jk的第一页，每页大小是20条；并会返回一共有多少页的信息</span>
Page&lt;User&gt; users = userRepository.findByLastname(<span class="hljs-string">"jk"</span>,PageRequest.of(<span class="hljs-number">1</span>, <span class="hljs-number">20</span>));
<span class="hljs-comment">//查询user里面的lastname=jk的第一页的20条数据，不知道一共多少条</span>
Slice&lt;User&gt;&nbsp;users = userRepository.findByLastname(<span class="hljs-string">"jk"</span>,PageRequest.of(<span class="hljs-number">1</span>, <span class="hljs-number">20</span>));
<span class="hljs-comment">//查询出来所有的user里面的lastname=jk的User数据，并按照name正序返回List</span>
List&lt;User&gt;&nbsp;users =&nbsp;userRepository.findByLastname(<span class="hljs-string">"jk"</span>,<span class="hljs-keyword">new</span> Sort(Sort.Direction.ASC, <span class="hljs-string">"name"</span>))
<span class="hljs-comment">//按照createdAt倒序，查询前一百条User数据</span>
List&lt;User&gt;&nbsp;users =&nbsp;userRepository.findByLastname(<span class="hljs-string">"jk"</span>,PageRequest.of(<span class="hljs-number">0</span>, <span class="hljs-number">100</span>, Sort.Direction.DESC, <span class="hljs-string">"createdAt"</span>));
</code></pre>
<p data-nodeid="6786">上面讲解了分页和排序的应用场景，在实际工作中，如果遇到不知道参数怎么传递的情况，可以看一下源码，因为 Java 是类型安全的。接下来讲解“限制查询结果 First 和 Top”，这是分页的另一种表达方式。</p>
<h3 data-nodeid="6787">限制查询结果 First 和 Top</h3>
<p data-nodeid="6788">有的时候我们想直接查询前几条数据，也不需要动态排序，那么就可以简单地在方法名字中使用 First 和 Top 关键字，来限制返回条数。</p>
<p data-nodeid="6789">我们来看看 userRepository 里面可以定义的一些限制返回结果的使用。在查询方法上加限制查询结果的关键字 First 和 Top。</p>
<pre class="lang-java" data-nodeid="6790"><code data-language="java"><span class="hljs-function">User <span class="hljs-title">findFirstByOrderByLastnameAsc</span><span class="hljs-params">()</span></span>;
<span class="hljs-function">User <span class="hljs-title">findTopByOrderByAgeDesc</span><span class="hljs-params">()</span></span>;
<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findDistinctUserTop3ByLastname</span><span class="hljs-params">(String lastname, Pageable pageable)</span></span>;
<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findFirst10ByLastname</span><span class="hljs-params">(String lastname, Sort sort)</span></span>;
<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findTop10ByLastname</span><span class="hljs-params">(String lastname, Pageable pageable)</span></span>;
</code></pre>
<p data-nodeid="6791">其中：</p>
<ul data-nodeid="6792">
<li data-nodeid="6793">
<p data-nodeid="6794">查询方法在使用 First 或 Top 时，数值可以追加到 First 或 Top 后面，指定返回最大结果的大小；</p>
</li>
<li data-nodeid="6795">
<p data-nodeid="6796">如果数字被省略，则假设结果大小为 1；</p>
</li>
<li data-nodeid="6797">
<p data-nodeid="6798">限制表达式也支持 Distinct 关键字；</p>
</li>
<li data-nodeid="6799">
<p data-nodeid="6800">支持将结果包装到 Optional 中（下一课时详解）。</p>
</li>
<li data-nodeid="6801">
<p data-nodeid="6802">如果将 Pageable 作为参数，以 Top 和 First 后面的数字为准，即分页将在限制结果中应用。</p>
</li>
</ul>
<p data-nodeid="6803">First 和 Top 关键字的使用非常简单，可以让我们的方法名语义更加清晰。接下来讲讲 NULL 的情况作了哪些支持。</p>
<h3 data-nodeid="6804"><a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNull.html" data-nodeid="6970">@NonNull</a>、<a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html" data-nodeid="6974">@NonNullApi</a>、<a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/Nullable.html" data-nodeid="6978">@Nullable</a></h3>
<p data-nodeid="6805">从 Spring Data 2.0 开始，JPA 新增了<a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNull.html" data-nodeid="6982">@NonNull</a> <a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/NonNullApi.html" data-nodeid="6986">@NonNullApi</a> <a href="https://docs.spring.io/spring/docs/5.2.8.RELEASE/javadoc-api/org/springframework/lang/Nullable.html" data-nodeid="6990">@Nullable</a>，是对 null 的参数和返回结果做的支持。</p>
<ul data-nodeid="6806">
<li data-nodeid="6807">
<p data-nodeid="6808"><strong data-nodeid="6996">@NonNullApi</strong>：在包级别用于声明参数，以及返回值的默认行为是不接受或产生空值的。</p>
</li>
<li data-nodeid="6809">
<p data-nodeid="6810"><strong data-nodeid="7001">@NonNull</strong>：用于不能为空的参数或返回值（在 @NonNullApi 适用的参数和返回值上不需要）。</p>
</li>
<li data-nodeid="6811">
<p data-nodeid="6812"><strong data-nodeid="7006">@Nullable</strong>：用于可以为空的参数或返回值。</p>
</li>
</ul>
<p data-nodeid="6813">我在自己的 Repository 所在 package 的 package-info.java 类里面做如下声明：</p>
<pre class="lang-java" data-nodeid="6814"><code data-language="java"><span class="hljs-meta">@org</span>.springframework.lang.NonNullApi
<span class="hljs-keyword">package</span> com.myrespository;
</code></pre>
<p data-nodeid="6815">myrespository 下面的 UserRepository 实现如下：</p>
<pre class="lang-java" data-nodeid="6816"><code data-language="java"><span class="hljs-keyword">package</span> com.myrespository;
<span class="hljs-keyword">import</span> org.springframework.lang.Nullable;
<span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>&gt; </span>{
  <span class="hljs-function">User <span class="hljs-title">getByEmailAddress</span><span class="hljs-params">(EmailAddress emailAddress)</span></span>; 
}
</code></pre>
<p data-nodeid="6817">这个时候当 emailAddress 参数为 null 的时候就会抛异常，当返回结果为 null 的时候也会抛异常。因为我们在package 的 package-info.java里面指定了NonNullApi，所有返回结果和参数不能为 Null。</p>
<pre class="lang-java" data-nodeid="6818"><code data-language="java">  <span class="hljs-meta">@Nullable</span>
  <span class="hljs-function">User <span class="hljs-title">findByEmailAddress</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> EmailAddress emailAdress)</span></span>;<span class="hljs-comment">//当我们添加@Nullable 注解之后，参数和返回结果这个时候就都会允许为 null 了；</span>
  <span class="hljs-function">Optional&lt;User&gt; <span class="hljs-title">findOptionalByEmailAddress</span><span class="hljs-params">(EmailAddress emailAddress)</span></span>; <span class="hljs-comment">//返回结果允许为 null,参数不允许为 null 的情况</span>
</code></pre>
<p data-nodeid="6819">以上就是对 Defining Query Methods 的方法名和分页参数整体学习了。</p>
<h3 data-nodeid="6820">给我们的一些思考</h3>
<p data-nodeid="6821">我们学习了 Defining Query Methods 的语法和其所表达的命名规范，在实际工作中，也可以将方法名（非常语义化的 respository 里面所定义方法命名规范）的强制约定规范运用到 controller 和 service 层，这样全部统一后，可以减少很多的沟通成本。</p>
<p data-nodeid="6822">Spring Data Common 里面的 repository 基类，我们是否可以应用推广到 service 层呢？能否也建立一个自己的 baseService？我们来看下面的实战例子：</p>
<pre class="lang-java" data-nodeid="6823"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">BaseService</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
    <span class="hljs-function">Class&lt;T&gt; <span class="hljs-title">getDomainClass</span><span class="hljs-params">()</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">saveAll</span><span class="hljs-params">(Iterable&lt;S&gt; entities)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">(T entity)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteById</span><span class="hljs-params">(ID id)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">(Iterable&lt;? extends T&gt; entities)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteInBatch</span><span class="hljs-params">(Iterable&lt;T&gt; entities)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAllInBatch</span><span class="hljs-params">()</span></span>;
    <span class="hljs-function">T <span class="hljs-title">getOne</span><span class="hljs-params">(ID id)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">Optional&lt;S&gt; <span class="hljs-title">findOne</span><span class="hljs-params">(Example&lt;S&gt; example)</span></span>;
    <span class="hljs-function">Optional&lt;T&gt; <span class="hljs-title">findById</span><span class="hljs-params">(ID id)</span></span>;
    <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span></span>;
    <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Sort sort)</span></span>;
    <span class="hljs-function">Page&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Pageable pageable)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example, Sort sort)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">Page&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example, Pageable pageable)</span></span>;
    <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">findAllById</span><span class="hljs-params">(Iterable&lt;ID&gt; ids)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">()</span></span>;
    &lt;S extends T&gt; <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">(Example&lt;S&gt; example)</span></span>;
    &lt;S extends T&gt; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">exists</span><span class="hljs-params">(Example&lt;S&gt; example)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">existsById</span><span class="hljs-params">(ID id)</span></span>;
    <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">flush</span><span class="hljs-params">()</span></span>;
    &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">saveAndFlush</span><span class="hljs-params">(S entity)</span></span>;
}
</code></pre>
<p data-nodeid="6824">我们模仿JpaRepository接口也自定义了一个自己的BaseService，声明了常用的CRUD操作，上面的代码是生产代码，可以作为参考。当然了我们也可以建立自己的 PagingAndSortingService、ComplexityService、SampleService 等来划分不同的 service接口，供不同目的 Service 子类继承。</p>
<p data-nodeid="6825">我们再来模仿一个&nbsp;SimpleJpaRepository，来实现自己的 BaseService 的实现类。</p>
<pre class="lang-java" data-nodeid="6826"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BaseServiceImpl</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>, <span class="hljs-title">R</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt;&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">BaseService</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> Map&lt;Class, Class&gt; DOMAIN_CLASS_CACHE = <span class="hljs-keyword">new</span> ConcurrentHashMap&lt;&gt;();
    <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> R repository;
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">BaseServiceImpl</span><span class="hljs-params">(R repository)</span> </span>{
        <span class="hljs-keyword">this</span>.repository = repository;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Class&lt;T&gt; <span class="hljs-title">getDomainClass</span><span class="hljs-params">()</span> </span>{
        Class thisClass = getClass();
        Class&lt;T&gt; domainClass = DOMAIN_CLASS_CACHE.get(thisClass);
        <span class="hljs-keyword">if</span> (Objects.isNull(domainClass)) {
            domainClass = GenericsUtils.getGenericClass(thisClass, <span class="hljs-number">0</span>);
            DOMAIN_CLASS_CACHE.putIfAbsent(thisClass, domainClass);
        }
        <span class="hljs-keyword">return</span> domainClass;
    }
    <span class="hljs-function"><span class="hljs-keyword">protected</span> R <span class="hljs-title">getRepository</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> repository;
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span> </span>{
        <span class="hljs-keyword">return</span> repository.save(entity);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">saveAll</span><span class="hljs-params">(Iterable&lt;S&gt; entities)</span> </span>{
        <span class="hljs-keyword">return</span> repository.saveAll(entities);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">(T entity)</span> </span>{
        repository.delete(entity);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteById</span><span class="hljs-params">(ID id)</span> </span>{
        repository.deleteById(id);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span> </span>{
        repository.deleteAll();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">(Iterable&lt;? extends T&gt; entities)</span> </span>{
        repository.deleteAll(entities);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteInBatch</span><span class="hljs-params">(Iterable&lt;T&gt; entities)</span> </span>{
        repository.deleteInBatch(entities);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAllInBatch</span><span class="hljs-params">()</span> </span>{
        repository.deleteAllInBatch();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">getOne</span><span class="hljs-params">(ID id)</span> </span>{
        <span class="hljs-keyword">return</span> repository.getOne(id);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">Optional&lt;S&gt; <span class="hljs-title">findOne</span><span class="hljs-params">(Example&lt;S&gt; example)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findOne(example);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Optional&lt;T&gt; <span class="hljs-title">findById</span><span class="hljs-params">(ID id)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findById(id);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Sort sort)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll(sort);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Page&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Pageable pageable)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll(pageable);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll(example);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">List&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example, Sort sort)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll(example, sort);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">Page&lt;S&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Example&lt;S&gt; example, Pageable pageable)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAll(example, pageable);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;T&gt; <span class="hljs-title">findAllById</span><span class="hljs-params">(Iterable&lt;ID&gt; ids)</span> </span>{
        <span class="hljs-keyword">return</span> repository.findAllById(ids);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> repository.count();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">(Example&lt;S&gt; example)</span> </span>{
        <span class="hljs-keyword">return</span> repository.count(example);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">exists</span><span class="hljs-params">(Example&lt;S&gt; example)</span> </span>{
        <span class="hljs-keyword">return</span> repository.exists(example);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">existsById</span><span class="hljs-params">(ID id)</span> </span>{
        <span class="hljs-keyword">return</span> repository.existsById(id);
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">flush</span><span class="hljs-params">()</span> </span>{
        repository.flush();
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">saveAndFlush</span><span class="hljs-params">(S entity)</span> </span>{
        <span class="hljs-keyword">return</span> repository.saveAndFlush(entity);
    }
}
</code></pre>
<p data-nodeid="12229" class="te-preview-highlight">以上代码就是 BaseService 常用的 CURD 实现代码，我们这里面大部分也是直接调用 Repository 提供的方法。需要注意的是，当继承 BaseServiceImpl 的时候需要传递自己的 Repository，如下面实例代码：</p>









<pre class="lang-java" data-nodeid="6828"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserServiceImpl</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseServiceImpl</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Long</span>, <span class="hljs-title">UserRepository</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">UserService</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">UserServiceImpl</span><span class="hljs-params">(UserRepository repository)</span> </span>{
        <span class="hljs-keyword">super</span>(repository);
    }
    .....
}
</code></pre>
<p data-nodeid="6829">实战思考只是提供一种常见的实现思路，你也可以根据实际情况进行扩展和扩充。</p>
<h3 data-nodeid="6830">总结</h3>
<p data-nodeid="6831">本课时主要讲解了 Defining Query Methods 的语法和参数部分的内容。首先介绍了配置方法，其次讲解了 DQM 语法结构所支持的关键字和特殊参数类型，最后对分页和 Null 做了特殊说明。通过本课时的学习，希望你可以轻松掌握 DQM 的方法名和参数的精髓所在，下一课时将会重点介绍 DQM 的返回结果有哪些支持，及其实现原理和实战应用场景，到时见~</p>
<p data-nodeid="6832">这里留个思考题：如何返回自定义 DTO 而不是 Entity?</p>
<blockquote data-nodeid="6833">
<p data-nodeid="6834">有思想，有方法，有技巧，有源码。如果觉得有帮助，就动动手指分享吧！同时也欢迎你在留言区发表学习感悟，大家一起更好地成长。</p>
<p data-nodeid="6835">学会看源码，逐步从入门到精通，提高学习效率。此种学习方法，可以应用在任何需要学习的框架里面。</p>
<p data-nodeid="6836" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="7028">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **亮：
> 进行分页查询时，相同的参数列表，相同的方法名称，但是不同的返回类型结果，如何放到同一个Repository接口里面呢？想通过修改方法名称来区分，但是不知道怎么修改算规范，自己改了几个启动运行时解析错误；

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是java基本语法，通过@query做

##### **波：
> PageRequest pageRequest = PageRequest.of(pageNum - 1, pageSize, Sort.by(Sort.Direction.DESC, "publishTime"));        Page这种先排序后分页，为啥不好使呢？jpa生成的sql不太对，有遇到过得没？select * from (select row_.*, rownum rownum_ from( select product0_.* from Product product0_order by product0_.publish_time desc) row_ where rownum

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用的什么数据库？数据库驱动是不是没有选择正确

##### **的太阳：
> 我怎样书写 if判断；比如做个查询多个字段，有的字段为空，怎么判断在mybatis中可以在xml中写，jpa怎么操作，jpa可以有xml吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不可以，耐心往后面看吧，后面课程有揭晓

##### **8963：
> 为view单独创建dto，里面保留entity中需要的字段，创建一个transferfrom方法，使得entity和dto的vo类可以相互转换。查询返回结果的时候的时候，把结果transfer，把返回类型Page中的T改为dto的类，最后返回dto的vo类

##### **1145：
> domainClass = GenericsUtils.getGenericClass(thisClass, 0);GenericsUtils，这个类找不到，是你自己写的吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，非常细心的同学，老师将此类放到了：https://github.com/zhangzhenhuajack/spring-boot-guide/blob/master/spring-data/spring-data-jpa/E02_03_04/src/main/GenericsUtils 中。

##### **的太阳：
> 删除怎么写？😂

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 逻辑删除，后面章节会有提到

##### **俊：
> 引导式学习，内容思路，结构设计的挺好，点赞！

##### *奇：
> 请问下课程3里用到的jdk是哪个版本的? 我看到了目前用不到的packageinfo.java

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1.8，这个是用不到的，我只是没有删除

##### *雨：
> 建一个DTO模型，查询的时候，在@Query里面写上“select new mydto() ...”。但是这个方法有明显的缺点，如果我有很多种返回方式，那就得定义许多DTO模型，工作量大。比如model有10个属性，有时候需要返回第1和第2个属性，有时候需要返回第1和第3个属性，组合的种类太多，需要定义很多DTO，而这些定义和业务逻辑无关，还很耗时间。想问问还有哪些可行的方案？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有，说实体你定义的不够抽象和底层；想想数据库是怎么解决的Entity最佳实践是和DB一一映射，不同的展示逻辑放在view层判断

