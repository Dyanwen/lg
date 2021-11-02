<p data-nodeid="5031" class="">通过上一课时，我们知道了 Spring Data 对整个数据操作做了很好的封装，其中 Spring Data Common 定义了很多公用的接口和一些相对数据操作的公共实现（如分页排序、结果映射、Autiting 信息、事务等），而 Spring Data JPA 就是 Spring Data Common 的关系数据库的查询实现。</p>


<p data-nodeid="4088">所以本课时我们来了解一下 Spring Data Common 的核心内容——Repository。我将从 Repository 的所有子类着手，带领你逐步掌握 CrudRepository、PageingAndSortingRepository、JpaRepository的使用。</p>
<p data-nodeid="4089">在讲解 Repository 之前，我们先来看看 Spring Data JPA 所依赖的 jar 包关系是什么样的，看下 Spring Data Common 的 jar 依赖关系。</p>
<h3 data-nodeid="4090">Spring Data Common 的依赖关系</h3>
<p data-nodeid="6277">我们通过 Gradle 看一下项目依赖，了解一下 Spring Data Common 的依赖关系。</p>
<p data-nodeid="6278" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/6F/Ciqc1F9i18OABIgzAAGVeUj3uCU674.png" alt="Drawing 0.png" data-nodeid="6282"></p>


<p data-nodeid="4093">通过上图的项目依赖，不难发现，数据库连接用的是 JDBC，连接池用的是 HikariCP，强依赖 Hibernate；Spring Boot Starter Data JPA 依赖 Spring Data JPA；而 Spring Data JPA 依赖 Spring Data Commons。</p>
<p data-nodeid="4094">在这些 jar 依赖关系中，Spring Data Commons 是我们要重点介绍的，因为 Spring Data Commons 是终极依赖。下面我们学习 DB 操作的入口 Repository，为你一一介绍 Repository 的子类。</p>
<h3 data-nodeid="4095">Repository 接口</h3>
<p data-nodeid="4096">Repository 是 Spring Data Common 里面的顶级父类接口，操作 DB 的入口类。首先介绍 Repository 接口的源码、类层次关系和使用实例。</p>
<h4 data-nodeid="4097">查看 Resposiory 源码</h4>
<p data-nodeid="4098">我们查看 Common 里面的 Resposiory 源码，了解一下里面实现了什么。如下所示：</p>
<pre class="lang-java" data-nodeid="4099"><code data-language="java"><span class="hljs-keyword">package</span> org.springframework.data.repository;
<span class="hljs-keyword">import</span> org.springframework.stereotype.Indexed;
<span class="hljs-meta">@Indexed</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="4100" class="">Resposiory 是 Spring Data 里面进行数据库操作顶级的抽象接口，里面什么方法都没有，但是如果任何接口继承它，就能得到一个 Repository，还可以实现 JPA 的一些默认实现方法。Spring 利用 Respository 作为 DAO 操作的 Type，以及利用 Java 动态代理机制就可以实现很多功能，比如为什么接口就能实现 DB 的相关操作？这就是 Spring 框架的高明之处。</p>
<p data-nodeid="4101">Spring 在做动态代理的时候，只要是它的子类或者实现类，再利用 T 类以及 T 类的 主键 ID 类型作为泛型的类型参数，就可以来标记出来、并捕获到要使用的实体类型，就能帮助使用者进行数据库操作。</p>
<h4 data-nodeid="4102">Repository 类层次关系</h4>
<p data-nodeid="4103">下面我们来根据存这个基类 Repository 接口，顺藤摸瓜看看 Spring Data JPA 里面都有什么。</p>
<p data-nodeid="7527"><strong data-nodeid="7532">首先，我们用工具 Intellij Idea，打开类 Repository.class，然后依次导航 → Hierchy 类型，会得到如下图所示的结果：</strong></p>
<p data-nodeid="7528" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7B/CgqCHl9i1-eADg-VAAL1Uy4EvRE891.png" alt="Drawing 1.png" data-nodeid="7535"></p>


<p data-nodeid="4106">通过该层次结构视图，你就会明白基类 Repository 的用意，由此可知，存储库分为以下 4 个大类。</p>
<ul data-nodeid="4107">
<li data-nodeid="4108">
<p data-nodeid="4109">ReactiveCrudRepository 这条线是响应式编程，主要支持当前 NoSQL 方面的操作，因为这方面大部分操作都是分布式的，所以由此我们可以看出 Spring Data 想统一数据操作的“野心”，即想提供关于所有 Data 方面的操作。目前 Reactive 主要有 Cassandra、MongoDB、Redis 的实现。</p>
</li>
<li data-nodeid="4110">
<p data-nodeid="4111">RxJava2CrudRepository 这条线是为了支持 RxJava 2 做的标准响应式编程的接口。</p>
</li>
<li data-nodeid="4112">
<p data-nodeid="4113">CoroutineCrudRepository 这条继承关系链是为了支持 Kotlin 语法而实现的。</p>
</li>
<li data-nodeid="4114">
<p data-nodeid="4115">CrudRepository 这条继承关系链正是本课时我要详细介绍的 JPA 相关的操作接口，你也可以把我的这种方法应用到另外 3 种继承关系链里面学习。</p>
</li>
</ul>
<p data-nodeid="8780">然后，通过 Intellij Idea，我们也可以打开类 UserRepository.java（第一课时“Spring Data JPA 初识”里面的案例），在此类里面，鼠标右键点击 Show Diagram 显示层次结构图，用图表的方式查看类的关系层次，打开后如下图（Repository 继承关系图）所示：</p>
<p data-nodeid="8781" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/70/Ciqc1F9i2AGAReiKAACJ2nYY8aw248.png" alt="Drawing 2.png" data-nodeid="8785"></p>


<p data-nodeid="4118">在这里简单介绍一下，我们需要掌握和使用到的类如下所示。</p>
<p data-nodeid="4119"><strong data-nodeid="4275">7 个大 Repository 接口：</strong></p>
<ul data-nodeid="4120">
<li data-nodeid="4121">
<p data-nodeid="4122">Repository(org.springframework.data.repository)，没有暴露任何方法；</p>
</li>
<li data-nodeid="4123">
<p data-nodeid="4124">CrudRepository(org.springframework.data.repository)，简单的 Curd 方法；</p>
</li>
<li data-nodeid="4125">
<p data-nodeid="4126">PagingAndSortingRepository(org.springframework.data.repository)，带分页和排序的方法；</p>
</li>
<li data-nodeid="4127">
<p data-nodeid="4128">QueryByExampleExecutor(org.springframework.data.repository.query)，简单 Example 查询；</p>
</li>
<li data-nodeid="4129">
<p data-nodeid="4130">JpaRepository(org.springframework.data.jpa.repository)，JPA 的扩展方法；</p>
</li>
<li data-nodeid="4131">
<p data-nodeid="4132">JpaSpecificationExecutor(org.springframework.data.jpa.repository)，JpaSpecification 扩展查询；</p>
</li>
<li data-nodeid="4133">
<p data-nodeid="4134">QueryDslPredicateExecutor(org.springframework.data.querydsl)，QueryDsl 的封装。</p>
</li>
</ul>
<p data-nodeid="4135"><strong data-nodeid="4286">两大 Repository 实现类：</strong></p>
<ul data-nodeid="4136">
<li data-nodeid="4137">
<p data-nodeid="4138">SimpleJpaRepository(org.springframework.data.jpa.repository.support)，JPA 所有接口的默认实现类；</p>
</li>
<li data-nodeid="4139">
<p data-nodeid="4140">QueryDslJpaRepository(org.springframework.data.jpa.repository.support)，QueryDsl 的实现类。</p>
</li>
</ul>
<p data-nodeid="4141">关于其他的类，后面我也会通过不同方式的讲解，让你一一认识。下面我们再来看一个 Repository 实例。</p>
<h4 data-nodeid="4142">一个 Repository 的实例</h4>
<p data-nodeid="4143">我们通过一个例子，利用 UserRepository 继承 Repository 来实现对 User 的两个查询方法，如下：</p>
<pre class="lang-java" data-nodeid="4144"><code data-language="java"><span class="hljs-keyword">import</span> org.springframework.data.repository.Repository;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Integer</span>&gt; </span>{
	<span class="hljs-comment">//根据名称进行查询用户列表</span>
	<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByName</span><span class="hljs-params">(String name)</span></span>;
	<span class="hljs-comment">// 根据用户的邮箱和名称查询</span>
	<span class="hljs-function">List&lt;User&gt; <span class="hljs-title">findByEmailAndName</span><span class="hljs-params">(String email, String name)</span></span>;
}
</code></pre>
<p data-nodeid="10030">由于 Repository 接口里面没有任何方法，所以此 UserRepository 对外只有两个可用方法，如上面的代码一样。Service 里面只能调用到 findByName 和 findByEmailAndName 两个方法，我们通过 IDEA 的 Structure 也可以看到对外只有两个方法可用，如下所示：</p>
<p data-nodeid="10031" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7B/CgqCHl9i2BCAOCRBAADotul53XM199.png" alt="Drawing 3.png" data-nodeid="10035"></p>


<p data-nodeid="11280">这时，我在第 01 课时中“Spring Boot 和 Spring Data JPA 的 Demo 演示”的例子里，提到过的 Controller 中引用 userRepository 的 save 和 findAll 方法就会报错。</p>
<p data-nodeid="11281" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7B/CgqCHl9i2BWAKgsoAADcQgdoISs764.png" alt="Drawing 4.png" data-nodeid="11285"></p>


<p data-nodeid="4149">上面这个实例通过继承 Repository，使 Spring 容器知道 UserRepository 是 DB 操作的类，是我们可以对 User 对象进行 CURD 的操作。这时我们对 Repository 有了一定的掌握，接下来再来看看它的直接子类 CurdRepository 接口都为我们提供了哪些方法。</p>
<h3 data-nodeid="13793" class="">CrudRepository 接口</h3>




<p data-nodeid="15027">下面我们通过 IDEA 工具，看下 CrudRepository 为我们提供的方法有哪些。</p>
<p data-nodeid="15028" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7B/CgqCHl9i2B-AcA4zAADw4REfVrA348.png" alt="Drawing 5.png" data-nodeid="15032"></p>


<p data-nodeid="4153">通过上图，你可以看到其中展示的一些方法，在这里一一说明一下：</p>
<ul data-nodeid="4154">
<li data-nodeid="4155">
<p data-nodeid="4156">count(): long 查询总数返回 long 类型；</p>
</li>
<li data-nodeid="4157">
<p data-nodeid="4158">void delete(T entity) 根据 entity 进行删除；</p>
</li>
<li data-nodeid="4159">
<p data-nodeid="4160">void deleteAll(Iterable&lt;? extends T&gt; entities) 批量删除；</p>
</li>
<li data-nodeid="4161">
<p data-nodeid="4162">void deleteAll() 删除所有；原理可以通过刚才的类关系查看，CrudRepository 的实现方法如下：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="4163"><code data-language="java"><span class="hljs-comment">//SimpleJpaRepository里面的deleteALL方法</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAll</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">for</span> (T element : findAll()) {
      delete(element);
   }
}
</code></pre>
<p data-nodeid="4164">通过源码我们可以看出 SimpleJpaRepository 里面的 deleteAll 是利用 for 循环调用 delete 方法进行删除操作。我们接着看 CrudRepository 提供的方法。</p>
<ul data-nodeid="26513">
<li data-nodeid="26514">
<p data-nodeid="26515">void deleteById(ID id); 根据主键删除，查看源码会发现，其是先查询出来再进行删除；</p>
</li>
<li data-nodeid="26516">
<p data-nodeid="26517">boolean existsById(ID id)  根据主键判断实体是否存在；</p>
</li>
<li data-nodeid="26518">
<p data-nodeid="26519">Iterable&lt;T&gt; findAllById(Iterable<id> ids); 根据主键列表查询实体列表；</id></p>
</li>
<li data-nodeid="26520">
<p data-nodeid="26521">Iterable&lt;T&gt; findAll(); 查询实体的所有列表；</p>
</li>
<li data-nodeid="26522">
<p data-nodeid="26523">Optional&lt;T&gt; findById(ID id); 根据主键查询实体，返回 JDK 1.8 的 Optional，这可以避免 null exception；</p>
</li>
<li data-nodeid="26524">
<p data-nodeid="26525">&lt;S extends T&gt; S save(S entity); 保存实体方法，参数和返回结果可以是实体的子类；</p>
</li>
<li data-nodeid="26526">
<p data-nodeid="26527">saveAll(Iterable&lt;S&gt; entities) : 批量保存，原理和 save方法相同，我们去看实现的话，就是 for 循环调用上面的 save 方法。</p>
</li>
</ul>
<p data-nodeid="27744">上面这些方法是 CrudRepository 对外暴露的常见的 Crud 接口，我们在对数据库进行 Crud 的时候就会运用到，如我们打算对 User 实体进行 Curd 操作，来看一下应该怎么写，如下所示：</p>
<pre class="lang-java" data-nodeid="27745"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="29098">我们通过 UserRepository 继承 CrudRepository，这个时候我们的 UserRepository 就会有 CrudRepository 里面的所有方法，如下图所示：</p>
<p data-nodeid="29099" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2PKAdUmeAASdzFspsBQ747.png" alt="Drawing 6.png" data-nodeid="29103"></p>


<p data-nodeid="27748">这里我们需要注意一下 save 和 deleteById 的实现逻辑，分别看看一下这两种方法是怎么实现的：</p>
<pre class="lang-java" data-nodeid="27749"><code data-language="java"><span class="hljs-comment">//新增或者保存</span>
<span class="hljs-keyword">public</span> &lt;S extends T&gt; <span class="hljs-function">S <span class="hljs-title">save</span><span class="hljs-params">(S entity)</span> </span>{
   <span class="hljs-keyword">if</span> (entityInformation.isNew(entity)) {
      em.persist(entity);
      <span class="hljs-keyword">return</span> entity;
   } <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">return</span> em.merge(entity);
   }
}
<span class="hljs-comment">//删除</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteById</span><span class="hljs-params">(ID id)</span> </span>{
   Assert.notNull(id, ID_MUST_NOT_BE_NULL);
   delete(findById(id).orElseThrow(() -&gt; <span class="hljs-keyword">new</span> EmptyResultDataAccessException(
         String.format(<span class="hljs-string">"No %s entity with id %s exists!"</span>, entityInformation.getJavaType(), id), <span class="hljs-number">1</span>)));
}
</code></pre>
<p data-nodeid="27750">你会发现在进行&nbsp;Update、Delete、Insert 等操作之前，我们看上面的源码，会通过 findById 先查询一下实体对象的 ID，然后再去对查询出来的实体对象进行保存操作。而如果在 Delete 的时候，查询到的对象不存在，则直接抛异常。</p>
<p data-nodeid="27751" class="">我在这里特别强调了一下 Delete 和 Save 方法，是因为在实际工作中，看到有的同事画蛇添足：自己在做 Save 的时候先去 Find 一下，其实是没有必要的，Spring JPA 底层都考虑到了。这里其实是想告诉你，当我们用任何第三方方法的时候，最好先查一下其源码和逻辑或者 API，然后再写出优雅的代码。</p>
<p data-nodeid="27752">关于 entityInformation.isNew（entity），在这里简单说一下，如果当传递的参数里面没有 ID，则直接 insert；若当传递的参数里面有 ID，则会触发 select 查询。此方法会去看一下数据库里面是否存在此记录，若存在，则 update，否则 insert。后面第 14 课时讲乐观锁实现机制的时候会有详细介绍。</p>
<h3 data-nodeid="27753">PagingAndSortingRepository 接口</h3>
<p data-nodeid="27754">上面我们介绍完了 Crud 的基本操作，发现没有分页和排序方法，那么接下来讲讲 PagingAndSortingRepository 接口，该接口也是 Repository 接口的子类，主要用于分页查询和排序查询。我们先来看看 PagingAndSortingRepository 的源码，了解一下都有哪些方法。</p>
<h4 data-nodeid="27755">PagingAndSortingRepository 的源码</h4>
<p data-nodeid="27756">PagingAndSortingRepository 源码发现有两个方法，分别是用于分页和排序的时候使用的，如下所示：</p>
<pre class="lang-java" data-nodeid="27757"><code data-language="java"><span class="hljs-keyword">package</span> org.springframework.data.repository;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Page;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Pageable;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Sort;
<span class="hljs-meta">@NoRepositoryBean</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PagingAndSortingRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; </span>{
	<span class="hljs-function">Iterable&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Sort sort)</span></span>; （<span class="hljs-number">1</span>）
	<span class="hljs-function">Page&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(Pageable pageable)</span></span>; （<span class="hljs-number">2</span>）
}
</code></pre>
<p data-nodeid="29724">其中，第一个方法 findAll 参数是 Sort，是根据排序参数，实现不同的排序规则获取所有的对象的集合；第二个方法 findAll 参数是 Pageable，是根据分页和排序进行查询，并用 Page 对返回结果进行封装。而 Pageable 对象包含 Page 和 Sort 对象。</p>
<p data-nodeid="29725">通过开篇讲到的【Repository 继承关系图】和上面介绍的一大堆源码可以看到，PagingAndSortingRepository 继承了 CrudRepository，进而拥有了父类的方法，并且增加了分页和排序等对查询结果进行限制的通用的方法。</p>

<p data-nodeid="27759">PagingAndSortingRepository 和 CrudRepository 都是 Spring Data Common 的标准接口，那么实现类是什么呢？如果我们采用 JPA，那对应的实现类就是 Spring Data JPA 的 jar 包里面的 SimpleJpaRepository。如果是其他 NoSQL的 实现如 MongoDB，那实现就在 Spring Data MongoDB 的 jar 里面的 MongoRepositoryImpl。</p>
<p data-nodeid="27760">关于 PagingAndSortingRepository 源码的介绍到这里，下面我们看看怎么使用这两个方法。</p>
<h4 data-nodeid="27761">PagingAndSortingRepository 使用案例</h4>
<p data-nodeid="27762">第一步：我们定一个 UserRepository 类来继承 PagingAndSortingRepository 接口，实现对 User 的分页和排序操作，实现源码如下：</p>
<pre class="lang-java" data-nodeid="27763"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.repository.PagingAndSortingRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">PagingAndSortingRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="27764">第二步：我们利用 UserRepository 直接继承 PagingAndSortingRepository 即可，而 Controller 里面就可以有如下用法了：</p>
<pre class="lang-java" data-nodeid="27765"><code data-language="java"><span class="hljs-comment">/**
 * 验证排序和分页查询方法，Pageable的默认实现类：PageRequest
 * <span class="hljs-doctag">@return</span>
 */</span>
<span class="hljs-meta">@GetMapping(path = "/page")</span>
<span class="hljs-meta">@ResponseBody</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Page&lt;User&gt; <span class="hljs-title">getAllUserByPage</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> userRepository.findAll(
         PageRequest.of(<span class="hljs-number">1</span>, <span class="hljs-number">20</span>,Sort.by(<span class="hljs-keyword">new</span> Sort.Order(Sort.Direction.ASC,<span class="hljs-string">"name"</span>))));
}
<span class="hljs-comment">/**
 * 排序查询方法，使用Sort对象
 * <span class="hljs-doctag">@return</span>
 */</span>
<span class="hljs-meta">@GetMapping(path = "/sort")</span>
<span class="hljs-meta">@ResponseBody</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Iterable&lt;User&gt; <span class="hljs-title">getAllUsersWithSort</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> userRepository.findAll(Sort.by(<span class="hljs-keyword">new</span> Sort.Order(Sort.Direction.ASC,<span class="hljs-string">"name"</span>)));
}
</code></pre>
<p data-nodeid="27766">到这里，你已经实现了对实体 User 的 DB 操作，那么以上内容我们学习了 CURD 和分页排序的基本操作，下面看看 JpaRepsitory 的接口为我们提供了哪些方法。</p>
<h3 data-nodeid="27767">JpaRepository 接口</h3>
<p data-nodeid="30960">到这里可以进入到分水岭了，上面的那些都是 Spring Data 为了兼容 NoSQL 而进行的一些抽象封装，而从 JpaRepository 开始是对关系型数据库进行抽象封装。从类图可以看出来它继承 PagingAndSortingRepository 类，也就继承了其所有方法，并且其实现类也是 SimpleJpaRepository。从类图上还可以看出 JpaRepository 继承和拥有了 QueryByExampleExecutor 的相关方法，我们先来看一下 JpaRepository 有哪些方法。一样的道理，我们直接看它的源码，看 Structure 即可，如下图所示：</p>
<p data-nodeid="30961" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2RKAfFGjAAGKTsMkBdw667.png" alt="Drawing 7.png" data-nodeid="30965"></p>


<p data-nodeid="27770">涉及 QueryByExample 的部分我们在 11 课时“JpaRepository 如何自定义”再详细介绍，而 JpaRepository 里面重点新增了批量删除，优化了批量删除的性能，类似于之前 SQL 的 batch 操作，并不是像上面的 deleteAll 来 for 循环删除。其中 flush() 和 saveAndFlush() 提供了手动刷新 session，把对象的值立即更新到数据库里面的机制。</p>
<p data-nodeid="27771">我们都知道 JPA 是 由 Hibernate 实现的，所以有 session 一级缓存的机制，当调用 save() 方法的时候，数据库里面是不会立即变化的，其原理我将在 21 课时“Persistence Context 所表达的核心概念是什么”再详细讲解。JpaRepository 的使用方式也一样，直接继承 JpaRepository 即可。</p>
<p data-nodeid="27772">我们看一个 Demo，用 UserRepository 直接继承 JpaRepository，来实现 JPA 的相关方法，如下所示：</p>
<pre class="lang-java" data-nodeid="27773"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="27774">这样 controller 里面就可以直接调用 JpaRepository 及其父接口里面的所有方法了。</p>
<p data-nodeid="27775">那么以上就是我们对 Repository 及其他子接口的使用案例，在应用时，你需要注意不同的接口有不同的方法，根据业务场景继承不同的接口即可。下面我们接着学习 Repository 的实现类 SimpleJpaRepository。</p>
<h3 data-nodeid="27776">Repository 的实现类 SimpleJpaRepository</h3>
<p data-nodeid="27777">关系数据库的所有 Repository 接口的实现类就是 SimpleJpaRepository，如果有些业务场景需要进行扩展了，可以继续继承此类，如 QueryDsl 的扩展（虽然不推荐使用了，但我们可以参考它的做法，自定义自己的 SimpleJpaRepository），如果能将此类里面的实现方法看透了，基本上 JPA 中的 API 就能掌握大部分内容。</p>
<p data-nodeid="32198">我们可以通过 Debug 视图看一下动态代理过程，如下面【类的继承关系图】所示：</p>
<p data-nodeid="32199" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2SCAeilgAAZs6DPtWQM598.png" alt="Drawing 8.png" data-nodeid="32203"></p>


<p data-nodeid="27780">你会发现 UserRepository 的实现类是 Spring 启动的时候，利用 Java 动态代理机制帮我们生成的实现类，而真正的实现类就是 SimpleJpaRepository。</p>
<p data-nodeid="27781">通过上面【类的继承关系图】也可以知道 SimpleJpaRepository 是 Repository 接口、CrudRepository 接口、PagingAndSortingRepository 接口、JpaRepository 接口的实现。其中，SimpleJpaRepository 的部分源码如下：</p>
<pre class="lang-java" data-nodeid="27782"><code data-language="java"><span class="hljs-meta">@Repository</span>
<span class="hljs-meta">@Transactional(readOnly = true)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SimpleJpaRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">T</span>, <span class="hljs-title">ID</span>&gt;, <span class="hljs-title">JpaSpecificationExecutor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String ID_MUST_NOT_BE_NULL = <span class="hljs-string">"The given id must not be null!"</span>;
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> JpaEntityInformation&lt;T, ?&gt; entityInformation;
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> EntityManager em;
	<span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> PersistenceProvider provider;
	<span class="hljs-keyword">private</span> <span class="hljs-meta">@Nullable</span> CrudMethodMetadata metadata;
	......
	<span class="hljs-meta">@Transactional</span>
	<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">deleteAllInBatch</span><span class="hljs-params">()</span> </span>{
		em.createQuery(getDeleteAllQueryString()).executeUpdate();
	}
	......
</code></pre>
<p data-nodeid="27783">通过此类的源码，我们可以挺清晰地看出 SimpleJpaRepository 的实现机制，是通过 EntityManger 进行实体的操作，而 JpaEntityInforMation 里面存在实体的相关信息和 Crud 方法的元数据等。</p>
<p data-nodeid="33436">上面我们讲到利用 Java 动态代理机制帮我们生成的实现类，那么关于动态代理的实现，我们可以在 RepositoryFactorySupport 设置一个断点，启动的时候，在我们的断点处就会发现 UserRepository 的接口会被动态代理成 SimpleJapRepository 的实现，如下图所示：</p>
<p data-nodeid="33437" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2S2AC9AXAAWAo3HVeSY110.png" alt="Drawing 9.png" data-nodeid="33441"></p>


<p data-nodeid="27786">这里需要注意的是每一个 Repository 的子类，都会通过这里的动态代理生成实现类，在实际工作中 debug 看源码的时候，希望上面介绍的内容可以帮助到你。</p>
<h3 data-nodeid="27787">Repository 接口给我的启发</h3>
<p data-nodeid="27788">在接触了 Repository 的源码之后，我在工作中遇到过一些类似需要抽象接口和写动态代理的情况，所以对于 Repository 的源码，我受到了一些启发：</p>
<p data-nodeid="27789">第一，上面的 7 个大 Repository 接口，我们在使用的时候可以根据实际场景，来继承不同的接口，从而选择暴露不同的 Spring Data Common 给我们提供的已有接口。这其实利用了 Java 语言的 interface 特性，在这里可以好好理解一下 interface 的妙用。</p>
<p data-nodeid="27790">第二，利用源码也可以很好地理解一下 Spring 中动态代理的作用，可以利用这种思想，在改善 MyBatis 的时候使用。</p>
<h3 data-nodeid="27791">总结</h3>
<p data-nodeid="27792">本课时到这里就结束了，这一课时我讲解了 Repository 接口、CrudRepository 接口、PagingAndSortingRepository 接口、JpaRepository 接口的用法，通过源码我们知道了接口里面的方法有哪些、怎么实现的，也知道了 Spring 的动态代理机制是怎么运用到 UserRepository 接口的。</p>
<p data-nodeid="27793">通过这一课时，相信你对 Repository 的基本用法，以及接口暴露的方法和使用方法都有了一定的了解，下节课我会讲解除了 Repository 的接口里面定义的方法之外，还可以在我们的 UserRepository 里面实现哪些方法，又会有哪些动态实现机制呢？我们到时见。</p>
<p data-nodeid="34674">补充一个TIPS：课程中的案例是依赖 lombok 插件的，如下图所示：</p>
<p data-nodeid="34675" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2TWAJZg_AABb1DeHmt4363.png" alt="Drawing 10.png" data-nodeid="34679"></p>


<p data-nodeid="35912">并开启 annotation processing。</p>
<p data-nodeid="35913" class=""><img src="https://s0.lgstatic.com/i/image/M00/50/7C/CgqCHl9i2TuAT9DcAACj394zaUc082.png" alt="Drawing 11.png" data-nodeid="35917"></p>


<blockquote data-nodeid="27798">
<p data-nodeid="27799">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="27865">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>







<s>
</s><s extends="" t=""><s>
</s></s><s extends="" t=""><s>
</s></s><s extends="" t=""><s>
</s></s><s extends="" t=""><s>
</s></s><s extends="" t=""><s>
</s></s><s extends="" t=""><s>








































































































































































</s></s>

---

### 精选评论

##### **伟：
> 看到有的同事画蛇添足：自己在做 Save 的时候先去 Find 一下，其实是没有必要的没有看到源码里有提前find的操作啊

##### **生：
> Spring data Jpa crudReposity 的deleteById 当不存在id数据时直接抛异常感觉不是很优雅

##### *冉：
> 这一节课还是看文章比较好一些，可以根据自己的节奏慢慢看，看懂一部分再看下一部分。还有这节课很重要对你理解JPA有很大的作用，可以多看几遍。

##### *文：
> 请问JPA如何做批量的save操作

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; saveAll();同时设置一下spring.jpa.properties.hibernate.jdbc.batch_size=200

##### **俊：
> 关系数据库的所有 Repository 接口的实现类就是 SimpleJpaRepository。还是得看源码啊，博主这帖子让人领会到源码的学习意义。

##### *深：
> 这钱，没白花

##### **9720：
> 为啥自己写代码总是没有写过这种需要抽象接口和写动态代理的代码呢，天天都是CRUD这种，求教：实际上是自己封装组件的时候用到这些代码么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当自己写框架时，就需要了解得多了

##### **仙：
> 之前以为对Repository已经比较了解了，没想到还是学到了不少新知识，赞！

##### **5755：
> 对于刚接触Repository思想的，源码核心部分点睛不错，期待源码讲解深入一点，比如在哪里扫描Repository的，调用链路等

##### *迪：
> save方法，id存在的时候不给某个字段赋值为什么直接给处理成空了？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 后面章节也会详细介绍，可以自己先查一下DynamicUpdate的概念

##### **喆：
> 讲的是真好，很用心

##### **钰：
> 动态sql咋实现 mybatis的if标签

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 接着往后看，后面课时有介绍

##### July：
> 更新太慢了，感兴趣的都在后面😅

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 一周两篇，别急哦，把每一讲都学扎实了，才能更好地运用。

##### **5833：
> 期待下一讲

##### *彬：
> 如果能顺便说一下再分布式微服务中直接使用jpa分页会出现问题就更好了，还有就是能细致的讲解一下JpaRespositoyFactoryBean接口原理，以及开发中如果扩展这个接口实现我们自己业务需求的功能就更好了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1. 分布式和分页有什么关系？会遇到问题？可以详细说一下。2. 如何自定义看一下后面的章节是否有帮助

