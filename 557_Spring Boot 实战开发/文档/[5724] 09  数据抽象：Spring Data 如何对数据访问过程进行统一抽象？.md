<p data-nodeid="232595" class="">事实上，JdbcTemplate 是相对偏底层的一个工具类，作为系统开发最重要的基础功能之一，数据访问层组件的开发方式在 Spring Boot 中也得到了进一步简化，并充分发挥了 Spring 家族中另一个重要成员 Spring Data 的作用。</p>
<p data-nodeid="232596">前面我们通过两个课时介绍了 Spring 框架用于访问关系型数据库的 JdbcTemplate 模板类，今天我们将对 Spring Data 框架中所提供的数据访问方式展开讨论。</p>
<p data-nodeid="232597">Spring Data 是 Spring 家族中专门用于数据访问的开源框架，其核心理念是对所有存储媒介支持资源配置从而实现数据访问。我们知道，数据访问需要完成领域对象与存储数据之间的映射，并对外提供访问入口，Spring Data 基于 Repository 架构模式抽象出一套实现该模式的统一数据访问方式。</p>
<p data-nodeid="232598">Spring Data 对数据访问过程的抽象主要体现在两个方面：① 提供了一套 Repository 接口定义及实现；② 实现了各种多样化的查询支持，接下来我们分别看一下。</p>
<h3 data-nodeid="232599">Repository 接口及实现</h3>
<p data-nodeid="232600">Repository 接口是 Spring Data 中对数据访问的最高层抽象，接口定义如下所示：</p>
<pre class="lang-java" data-nodeid="232601"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="232602">在以上代码中，我们看到 Repository 接口只是一个空接口，通过泛型指定了领域实体对象的类型和 ID。在 Spring Data 中，存在一大批 Repository 接口的子接口和实现类，该接口的部分类层结构如下所示：</p>
<p data-nodeid="232603"><img src="https://s0.lgstatic.com/i/image/M00/84/21/CgqCHl_TH_mAIaaLAABBNaldOqE595.png" alt="image (3).png" data-nodeid="232820"></p>
<div data-nodeid="232604"><p style="text-align:center">Repository 接口的部分类层结构图</p></div>
<p data-nodeid="232605">可以看到 CrudRepository 接口是对 Repository 接口的最常见扩展，添加了对领域实体的 CRUD 操作功能，具体定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="232606"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
&nbsp; &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span></span>;
&nbsp; &lt;S extends T&gt; <span class="hljs-function">Iterable&lt;S&gt; <span class="hljs-title">saveAll</span><span class="hljs-params">(Iterable&lt;S&gt; entities)</span></span>;
&nbsp; <span class="hljs-function">Optional&lt;T&gt; <span class="hljs-title">findById</span><span class="hljs-params">(ID id)</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">existsById</span><span class="hljs-params">(ID id)</span></span>;
&nbsp; <span class="hljs-function">Iterable&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span></span>;
&nbsp; <span class="hljs-function">Iterable&lt;T&gt; <span class="hljs-title">findAllById</span><span class="hljs-params">(Iterable&lt;ID&gt; ids)</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">()</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteById</span><span class="hljs-params">(ID id)</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">delete</span><span class="hljs-params">(T entity)</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">(Iterable&lt;? extends T&gt; entities)</span></span>;
&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="232607">这些方法都是自解释的，我们可以看到 CrudRepository 接口提供了保存单个实体、保存集合、根据 id 查找实体、根据 id 判断实体是否存在、查询所有实体、查询实体数量、根据 id 删除实体 、删除一个实体的集合以及删除所有实体等常见操作，我们具体来看下其中几个方法的实现过程。</p>
<p data-nodeid="232608">在实现过程中，我们首先需要关注最基础的 save 方法。通过查看 CrudRepository 的类层结构，我们找到它的一个实现类 SimpleJpaRepository，这个类显然是基于 JPA 规范所实现的针对关系型数据库的数据访问类。</p>
<p data-nodeid="232609">save 方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="232610"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> JpaEntityInformation&lt;T, ?&gt; entityInformation;
<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> EntityManager em;
&nbsp;
<span class="hljs-meta">@Transactional</span>
<span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (entityInformation.isNew(entity)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; em.persist(entity);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> entity;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> em.merge(entity);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="232611">显然，上述 save 方法依赖于 JPA 规范中的 EntityManager，当它发现所传入的实体为一个新对象时，就会调用 EntityManager 的 persist 方法，反之使用该对象进行 merge。关于 JPA 规范以及 EntityManager 我们在下一课时中会详细展开。</p>
<p data-nodeid="232612">我们接着看一下用于根据 id 查询实体的 findOne 方法，如下代码所示：</p>
<pre class="lang-java" data-nodeid="232613"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> T <span class="hljs-title">findOne</span><span class="hljs-params">(ID id)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(id, ID_MUST_NOT_BE_NULL);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Class&lt;T&gt; domainType = getDomainClass();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (metadata == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> em.find(domainType, id);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; LockModeType type = metadata.getLockModeType();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; hints = getQueryHints();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> type == <span class="hljs-keyword">null</span> ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints);
}
</code></pre>
<p data-nodeid="232614">在执行查询过程中，findOne 方法会根据领域实体的类型调用 EntityManager 的 find 方法来查找目标对象。需要注意的是，这里也会用到一些元数据 Metadata，以及涉及改变正常 SQL 执行效果的 Hint 机制的使用。</p>
<h3 data-nodeid="232615">多样化查询支持</h3>
<p data-nodeid="232616">在日常开发过程中，数据查询的操作次数要远高于数据新增、数据删除和数据修改，因此在 Spring Data 中，除了对领域对象提供默认的 CRUD 操作外，我们还需要对查询场景高度抽象。而在现实的业务场景中，最典型的查询操作是 @Query 注解和方法名衍生查询机制。</p>
<h4 data-nodeid="232617">@Query 注解</h4>
<p data-nodeid="232618">我们可以通过 @Query 注解直接在代码中嵌入查询语句和条件，从而提供类似 ORM 框架所具有的强大功能。</p>
<p data-nodeid="232619">下面就是使用 @Query 注解进行查询的典型例子：</p>
<pre class="lang-java" data-nodeid="232620"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">Account</span>, 
	<span class="hljs-title">Long</span>&gt; </span>{
	&nbsp;
&nbsp; <span class="hljs-meta">@Query("select a from Account a where a.userName = ?1")</span> 
&nbsp; <span class="hljs-function">Account <span class="hljs-title">findByUserName</span><span class="hljs-params">(String userName)</span></span>;
}
</code></pre>
<p data-nodeid="232621">注意到这里的 @Query 注解使用的是类似 SQL 语句的语法，它能自动完成领域对象 Account 与数据库数据之间的相互映射。因我们使用的是 JpaRepository，所以这种类似 SQL 语句的语法实际上是一种 JPA 查询语言，也就是所谓的 JPQL（Java Persistence Query Language）。</p>
<p data-nodeid="232622">JPQL 的基本语法如下所示：</p>
<pre class="lang-xml" data-nodeid="232623"><code data-language="xml">SELECT 子句 FROM 子句 
[WHERE 子句] 
[GROUP BY 子句]
[HAVING 子句] 
[ORDER BY 子句]
</code></pre>
<p data-nodeid="232624">JPQL 语句是不是和原生的 SQL 语句非常类似？唯一的区别就是 JPQL FROM 语句后面跟的是对象，而原生 SQL 语句中对应的是数据表中的字段。</p>
<p data-nodeid="232625">介绍完 JPQL 之后，我们再回到 @Query 注解定义，这个注解位于 org.springframework.data.jpa.repository 包中，如下所示：</p>
<pre class="lang-java" data-nodeid="232626"><code data-language="java"><span class="hljs-keyword">package</span> org.springframework.data.jpa.repository;
&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Query {
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">countQuery</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">countProjection</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">nativeQuery</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">countName</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
}
</code></pre>
<p data-nodeid="232627">@Query 注解中最常用的就是 value 属性，在前面示例中 JPQL 语句有使用到 。当然，如果我们将 nativeQuery 设置为 true，那么 value 属性则需要指定具体的原生 SQL 语句。</p>
<p data-nodeid="232628">请注意，在 Spring Data 中存在一批 @Query 注解，分别针对不同的持久化媒介。例如 MongoDB 中存在一个 @Query 注解，但该注解位于 org.springframework.data.mongodb.repository 包中，定义如下：</p>
<pre class="lang-java" data-nodeid="232629"><code data-language="java"><span class="hljs-keyword">package</span> org.springframework.data.mongodb.repository;
&nbsp;
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Query {
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">fields</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">count</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">exists</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">delete</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
}
</code></pre>
<p data-nodeid="232630">与面向 JPA 的 @Query 注解不同的是，MongoDB 中 @Query 注解的 value 值是一串 JSON 字符串，用于指定需要查询的对象条件，这里我们就不具体展开了。</p>
<h4 data-nodeid="232631">方法名衍生查询</h4>
<p data-nodeid="232632">方法名衍生查询也是 Spring Data 的查询特色之一，通过在方法命名上直接使用查询字段和参数，Spring Data 就能自动识别相应的查询条件并组装对应的查询语句。典型的示例如下所示：</p>
<pre class="lang-java" data-nodeid="232633"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AccountRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">Account</span>, 
	<span class="hljs-title">Long</span>&gt; </span>{
	&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function">List&lt;Account&gt; <span class="hljs-title">findByFirstNameAndLastName</span><span class="hljs-params">(String firstName, String 
	lastName)</span></span>;
}
</code></pre>
<p data-nodeid="232634">在上面的例子中，通过 findByFirstNameAndLastname 这样符合普通语义的方法名，并在参数列表中按照方法名中参数的顺序和名称（即第一个参数是 fistName，第二个参数 lastName）传入相应的参数，Spring Data 就能自动组装 SQL 语句从而实现衍生查询。是不是很神奇？</p>
<p data-nodeid="232635"><strong data-nodeid="232846">而想要使用方法名实现衍生查询，我们需要对 Repository 中定义的方法名进行一定约束。</strong></p>
<p data-nodeid="232636">首先我们需要指定一些查询关键字，常见的关键字如下表所示：</p>
<p data-nodeid="233039" class=""><img src="https://s0.lgstatic.com/i/image2/M01/01/5E/Cip5yF_YhK6AcrMVAAQOamtdsF0627.png" alt="Lark20201215-174017.png" data-nodeid="233042"></p>

<div data-nodeid="232757" class=""><p style="text-align:center">方法名衍生查询中查询关键字列表</p></div>
<p data-nodeid="232758">有了这些查询关键字后，在方法命名上我们还需要指定查询字段和一些限制性条件。例如，在前面的示例中，我们只是基于“fistName”和“lastName”这两个字段做查询。</p>
<p data-nodeid="232759">事实上，我们可以查询的内容非常多，下表列出了更多的方法名衍生查询示例，你可以参考下。</p>
<p data-nodeid="233501" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/01/5E/Cip5yF_YhLiAbg0pAAEzy-P0ZVA978.png" alt="Lark20201215-174023.png" data-nodeid="233504"></p>

<div data-nodeid="232791"><p style="text-align:center">方法名衍生查询示例</p></div>
<p data-nodeid="232792">在 Spring Data 中，方法名衍生查询的功能非常强大，上表中罗列的这些也只是全部功能中的一小部分而已。</p>
<p data-nodeid="232793">讲到这里，你可能会问一个问题：如果我们在一个 Repository 中同时指定了 @Query 注解和方法名衍生查询，那么 Spring Data 会具体执行哪一个呢？要想回答这个问题，就需要我们对查询策略有一定的了解。</p>
<p data-nodeid="232794">在 Spring Data 中，查询策略定义在 QueryLookupStrategy 中，如下代码所示：</p>
<pre class="lang-java" data-nodeid="232795"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">QueryLookupStrategy</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">enum</span> Key {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CREATE, USE_DECLARED_QUERY, CREATE_IF_NOT_FOUND;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> Key <span class="hljs-title">create</span><span class="hljs-params">(String xml)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!StringUtils.hasText(xml)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> valueOf(xml.toUpperCase(Locale.US).replace(<span class="hljs-string">"-"</span>, <span class="hljs-string">"_"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">RepositoryQuery <span class="hljs-title">resolveQuery</span><span class="hljs-params">(Method method, RepositoryMetadata metadata, ProjectionFactory factory, NamedQueries namedQueries)</span></span>;
}
</code></pre>
<p data-nodeid="232796">从以上代码中，我们看到 QueryLookupStrategy 分为三种，即 CREATE、USE_DECLARED_QUERY 和 CREATE_IF_NOT_FOUND。</p>
<p data-nodeid="232797">这里的 CREATE 策略指的是直接根据方法名创建的查询策略，也就是使用前面介绍的方法名衍生查询。</p>
<p data-nodeid="232798">而 USE_DECLARED_QUERY 指的是声明方式，主要使用 @Query 注解，如果没有 @Query 注解系统就会抛出异常。</p>
<p data-nodeid="232799">而最后一种 CREATE_IF_NOT_FOUND 可以理解为是 @Query 注解和方法名衍生查询两者的兼容版。<strong data-nodeid="233011">请注意，Spring Data 默认使用的是 CREATE_IF_NOT_FOUND 策略，也就是说系统会先查找 @Query 注解，如果查到没有，会再去找与方法名相匹配的查询。</strong></p>
<h3 data-nodeid="232800">Spring Data 中的组件</h3>
<p data-nodeid="232801"><strong data-nodeid="233016">Spring Data 支持对多种数据存储媒介进行数据访问，表现为提供了一系列默认的 Repository，包括针对关系型数据库的 JPA/JDBC Repository，针对 MongoDB、Neo4j、Redis 等 NoSQL 对应的 Repository，支持 Hadoop 的大数据访问的 Repository，甚至包括 Spring Batch 和 Spring Integration 在内的系统集成的 Repository。</strong></p>
<p data-nodeid="232802">在 Spring Data 的官方网站<a href="https://spring.io/projects/spring-data" data-nodeid="233020">https://spring.io/projects/spring-data</a> 中，列出了其提供的所有组件，如下图所示：</p>
<p data-nodeid="232803"><img src="https://s0.lgstatic.com/i/image/M00/84/22/CgqCHl_TICWAOAMHAAAkcFfMwis206.png" alt="image (4).png" data-nodeid="233024"></p>
<div data-nodeid="232804"><p style="text-align:center">Spring Data 所提供的组件列表（来自 Spring Data 官网）</p></div>
<p data-nodeid="232805">根据官网介绍，Spring Data 中的组件可以分成四大类：核心模块（Main modules）、社区模块（Community modules）、关联模块（Related modules）和正在孵化的模块（Modules in Incubation）。例如，前面介绍的 Respository 和多样化查询功能就在核心模块 Spring Data Commons 组件中。</p>
<p data-nodeid="232806"><strong data-nodeid="233034">这里，我特别想强调下的是正在孵化的模块，它目前只包含一个组件，即 Spring Data R2DBC。</strong> R2DBC 是<a href="https://github.com/r2dbc/" data-nodeid="233032">Reactive Relational Database Connectivity</a> 的简写，代表响应式关系型数据库连接，相当于是响应式数据访问领域的 JDBC 规范。</p>
<h3 data-nodeid="232807">小结与预告</h3>
<p data-nodeid="232808">数据访问是一切应用系统的基础，Spring Boot 作为一款集成性的开发框架，专门提供了 Spring Data 组件实现对数据访问过程进行抽象。基于 Repository 架构模式，Spring Data 为开发人员提供了一系列用于完成 CRUD 操作的工具方法，尤其是对最常用的查询操作专门进行了提炼和设计，使得开发过程更简单、高效。</p>
<p data-nodeid="232809">这里给你留一道思考题：在使用 Spring Data 时，针对查询操作可以使用哪些高效的实现方法？</p>
<p data-nodeid="232810" class="">在今天内容的基础上，下一课时我们将基于 Spring Data 框架中的 Spring Data JPA 来访问关系型数据库，并结合 SpringCSS 案例完成对现有实现方式的重构。</p>

---

### 精选评论

##### *龙：
> 是mybaties的替代品吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不算替代吧，只是另一种实现策略，而且是 Java 领域的 ORM 标准规范。

##### *谨：
> Spring data jpa 能够解决动态拼接参数么,像后台管理很多这样的需求

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的

##### **7093：
> spring data 可以支持多表关联查询么？单表查询有时候无法满足业务需求？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以支持的，后面会有代码示例。

