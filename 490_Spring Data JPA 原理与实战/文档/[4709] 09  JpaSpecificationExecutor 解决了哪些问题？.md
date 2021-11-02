<p data-nodeid="943" class="">欢迎来到第二个模块，从这一课时开始，我们就要进入高级用法与实战的学习。在进阶高级开发 / 架构师的路上，我将尽可能把经验都传授给你，帮助你少走弯路。</p>
<p data-nodeid="944">学习完前面 8 个课时，相信作为一名开发人员，你对 JPA 的基本用法已经有了一定的了解。那么从这一课时开始，我们要介绍一些复杂场景的使用，特别是作为一名架构师必须要掌握的内容。</p>
<p data-nodeid="945">我们先来看看除了前几节课我们讲解的 Define Query Method 和 @Query 之外，还有哪些查询方法。首先看一个简单的 QueryByExampleExecutor 用法。</p>
<h3 data-nodeid="946">QueryByExampleExecutor用法</h3>
<p data-nodeid="947">QueryByExampleExecutor（QBE）是一种用户友好的查询技术，具有简单的接口，它允许动态查询创建，并且不需要编写包含字段名称的查询。</p>
<p data-nodeid="948">下面是一个 UML 图，你可以看到 QueryByExampleExecutor 是 JpaRepository 的父接口，也就是 JpaRespository 里面继承了 QueryByExampleExecutor 的所有方法。</p>
<p data-nodeid="949"><img src="https://s0.lgstatic.com/i/image/M00/5D/4B/CgqCHl-EE7WAAfi5AACTjc0iffY586.png" alt="Drawing 0.png" data-nodeid="1098"></p>
<div data-nodeid="950"><p style="text-align:center">图一：Repository 类图</p></div>
<h4 data-nodeid="951">QBE 的基本语法</h4>
<p data-nodeid="952">QBE 的基本语法可以分为下述几种。</p>
<pre class="lang-dart" data-nodeid="953"><code data-language="dart">public <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">QueryByExampleExecutor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{ 
 <span class="hljs-comment">//根据“实体”查询条件，查找一个对象</span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; S findOne(Example&lt;S&gt; example);
<span class="hljs-comment">//根据“实体”查询条件，查找一批对象</span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; <span class="hljs-built_in">Iterable</span>&lt;S&gt; findAll(Example&lt;S&gt; example); 
<span class="hljs-comment">//根据“实体”查询条件，查找一批对象，可以指定排序参数</span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; <span class="hljs-built_in">Iterable</span>&lt;S&gt; findAll(Example&lt;S&gt; example, Sort sort);
 <span class="hljs-comment">//根据“实体”查询条件，查找一批对象，可以指定排序和分页参数 </span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; Page&lt;S&gt; findAll(Example&lt;S&gt; example, Pageable pageable);
<span class="hljs-comment">//根据“实体”查询条件，查找返回符合条件的对象个数</span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; long count(Example&lt;S&gt; example); 
<span class="hljs-comment">//根据“实体”查询条件，判断是否有符合条件的对象</span>
&lt;S <span class="hljs-keyword">extends</span> T&gt; boolean exists(Example&lt;S&gt; example); 
}
</code></pre>
<p data-nodeid="954">你可以看到这几个语法其实差不多，下面我们用 Page<code data-backticks="1" data-nodeid="1102">&lt;S&gt;</code> findAll 写一个分页查询的例子，看一下效果。</p>
<h4 data-nodeid="955">QueryByExampleExecutor 的使用案例</h4>
<p data-nodeid="956">我们还用先前的 User 实体和 UserAddress 实体，并把 User 变丰富一点，这样方便测试。两个实体关键代码如下。</p>
<pre class="lang-java" data-nodeid="957"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString(exclude = "address")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> String email;
   <span class="hljs-meta">@Enumerated(EnumType.STRING)</span>
   <span class="hljs-keyword">private</span> SexEnum sex;
   <span class="hljs-keyword">private</span> Integer age;
   <span class="hljs-keyword">private</span> Instant createDate;
   <span class="hljs-keyword">private</span> Date updateDate;
   <span class="hljs-meta">@OneToMany(mappedBy = "user",fetch = FetchType.EAGER,cascade = {CascadeType.ALL})</span>
   <span class="hljs-keyword">private</span> List&lt;UserAddress&gt; address;
}
<span class="hljs-keyword">enum</span> SexEnum {
   BOY,GIRL
}
<span class="hljs-comment">//User实体我们扩充了一些字段去了不同的类型，方便测试</span>
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString(exclude = "user")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserAddress</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String address;
   <span class="hljs-meta">@ManyToOne(cascade = CascadeType.ALL)</span>
   <span class="hljs-meta">@JsonIgnore</span>
   <span class="hljs-keyword">private</span> User user;
}
<span class="hljs-comment">//UserAddress基本上不变</span>
</code></pre>
<p data-nodeid="958">可以看出两个实体我们加了些字段。UserAddressRepository 继承 JpaRepository，从而也继承了 QueryByExampleExceutor 里面的方法，如下所示。</p>
<pre class="lang-java" data-nodeid="959"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserAddressRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">UserAddress</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="960">那么我们写一个测试用例，来熟悉一下 QBE 的语法，看一下完整的测试用例的写法。</p>
<pre class="lang-java" data-nodeid="961"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> com.fasterxml.jackson.core.JsonProcessingException;
<span class="hljs-keyword">import</span> com.fasterxml.jackson.databind.ObjectMapper;
<span class="hljs-keyword">import</span> com.google.common.collect.Lists;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.BeforeAll;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.TestInstance;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Example;
<span class="hljs-keyword">import</span> org.springframework.data.domain.ExampleMatcher;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Page;
<span class="hljs-keyword">import</span> org.springframework.data.domain.PageRequest;
<span class="hljs-keyword">import</span> org.springframework.test.annotation.Rollback;
<span class="hljs-keyword">import</span> javax.transaction.Transactional;
<span class="hljs-keyword">import</span> java.time.Instant;
<span class="hljs-keyword">import</span> java.util.Date;
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-meta">@TestInstance(TestInstance.Lifecycle.PER_CLASS)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserAddressRepositoryTest</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> UserAddressRepository userAddressRepository;
   <span class="hljs-keyword">private</span> Date now = <span class="hljs-keyword">new</span> Date();
   <span class="hljs-comment">/**
    * 负责添加数据，假设数据库里面已经有的数据
    */</span>
   <span class="hljs-meta">@BeforeAll</span>
   <span class="hljs-meta">@Rollback(false)</span>
   <span class="hljs-meta">@Transactional</span>
   <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
      User user = User.builder()
            .name(<span class="hljs-string">"jack"</span>)
            .email(<span class="hljs-string">"123456@126.com"</span>)
            .sex(SexEnum.BOY)
            .age(<span class="hljs-number">20</span>)
            .createDate(Instant.now())
            .updateDate(now)
            .build();
      userAddressRepository.saveAll(Lists.newArrayList(UserAddress.builder().user(user).address(<span class="hljs-string">"shanghai"</span>).build(),
            UserAddress.builder().user(user).address(<span class="hljs-string">"beijing"</span>).build()));
   }
   <span class="hljs-meta">@Test</span>
   <span class="hljs-meta">@Rollback(false)</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testQBEFromUserAddress</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> JsonProcessingException </span>{
      User request = User.builder()
            .name(<span class="hljs-string">"jack"</span>).age(<span class="hljs-number">20</span>).email(<span class="hljs-string">"12345"</span>)
            .build();
      UserAddress address = UserAddress.builder().address(<span class="hljs-string">"shang"</span>).user(request).build();
      ObjectMapper objectMapper = <span class="hljs-keyword">new</span> ObjectMapper();
<span class="hljs-comment">//    System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(address)); //可以打印出来看看参数是什么</span>
<span class="hljs-comment">//创建匹配器，即如何使用查询条件</span>
      ExampleMatcher exampleMatcher = ExampleMatcher.matching()
            .withMatcher(<span class="hljs-string">"user.email"</span>, ExampleMatcher.GenericPropertyMatchers.startsWith())
            .withMatcher(<span class="hljs-string">"address"</span>, ExampleMatcher.GenericPropertyMatchers.startsWith());
      Page&lt;UserAddress&gt; u = userAddressRepository.findAll(Example.of(address,exampleMatcher), PageRequest.of(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>));
    System.out.println(objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(u));
   }
}
</code></pre>
<p data-nodeid="962">其中，方法 testQBEFromUserAddress 负责测试 QBE，那么假设我们要写 API 的话，前端给我们的查询参数如下。</p>
<pre class="lang-java" data-nodeid="963"><code data-language="java">{
   <span class="hljs-string">"id"</span> : <span class="hljs-keyword">null</span>,
   <span class="hljs-string">"address"</span> : <span class="hljs-string">"shang"</span>,
   <span class="hljs-string">"user"</span> : {
      <span class="hljs-string">"id"</span> : <span class="hljs-keyword">null</span>,
      <span class="hljs-string">"name"</span> : <span class="hljs-string">"jack"</span>,
      <span class="hljs-string">"email"</span> : <span class="hljs-string">"12345"</span>,
      <span class="hljs-string">"sex"</span> : <span class="hljs-keyword">null</span>,
      <span class="hljs-string">"age"</span> : <span class="hljs-number">20</span>,
      <span class="hljs-string">"createDate"</span> : <span class="hljs-keyword">null</span>,
      <span class="hljs-string">"updateDate"</span> : <span class="hljs-keyword">null</span>
   }
}
</code></pre>
<p data-nodeid="964">想要满足 email 前缀匹配、地址前缀匹配的动态查询条件，我们可以跑一下测试用例看一下结果。</p>
<pre class="lang-java" data-nodeid="965"><code data-language="java">Hibernate: select useraddres0_.id as id1_2_, useraddres0_.address as address2_2_, useraddres0_.user_id as user_id3_2_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id=user1_.id where user1_.age=<span class="hljs-number">20</span> and (user1_.email like ? escape ?) and user1_.name=? and (useraddres0_.address like ? escape ?) limit ?
<span class="hljs-number">2020</span>-<span class="hljs-number">09</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">04</span>:<span class="hljs-number">24.391</span> TRACE <span class="hljs-number">62179</span> --- [&nbsp; &nbsp; Test worker] o.h.type.descriptor.sql.BasicBinder&nbsp; &nbsp; &nbsp; : binding parameter [<span class="hljs-number">1</span>] as [VARCHAR] - [<span class="hljs-number">12345</span>%]
<span class="hljs-number">2020</span>-<span class="hljs-number">09</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">04</span>:<span class="hljs-number">24.391</span> TRACE <span class="hljs-number">62179</span> --- [&nbsp; &nbsp; Test worker] o.h.type.descriptor.sql.BasicBinder&nbsp; &nbsp; &nbsp; : binding parameter [<span class="hljs-number">2</span>] as [CHAR] - [\]
<span class="hljs-number">2020</span>-<span class="hljs-number">09</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">04</span>:<span class="hljs-number">24.392</span> TRACE <span class="hljs-number">62179</span> --- [&nbsp; &nbsp; Test worker] o.h.type.descriptor.sql.BasicBinder&nbsp; &nbsp; &nbsp; : binding parameter [<span class="hljs-number">3</span>] as [VARCHAR] - [jack]
<span class="hljs-number">2020</span>-<span class="hljs-number">09</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">04</span>:<span class="hljs-number">24.392</span> TRACE <span class="hljs-number">62179</span> --- [&nbsp; &nbsp; Test worker] o.h.type.descriptor.sql.BasicBinder&nbsp; &nbsp; &nbsp; : binding parameter [<span class="hljs-number">4</span>] as [VARCHAR] - [shang%]
<span class="hljs-number">2020</span>-<span class="hljs-number">09</span>-<span class="hljs-number">20</span> <span class="hljs-number">23</span>:<span class="hljs-number">04</span>:<span class="hljs-number">24.393</span> TRACE <span class="hljs-number">62179</span> --- [&nbsp; &nbsp; Test worker] o.h.type.descriptor.sql.BasicBinder&nbsp; &nbsp; &nbsp; : binding parameter [<span class="hljs-number">5</span>] as [CHAR] - [\]
</code></pre>
<p data-nodeid="966">其中我们可以看到，传进来的参数和最终执行的 SQL，还挺符合我们的预期的，所以我们也能得到正确响应的查询结果，如下图：</p>
<p data-nodeid="967"><img src="https://s0.lgstatic.com/i/image/M00/5D/40/Ciqc1F-EFDCAa50VAADEF8jBllY550.png" alt="Drawing 1.png" data-nodeid="1113"></p>
<p data-nodeid="968">也就是一个地址带一个 User 结果。</p>
<p data-nodeid="969">那么接下来我们分析一下 Example 这个参数，看看它具体的语法是什么。</p>
<h4 data-nodeid="970">Example 语法详解</h4>
<p data-nodeid="971">关于 Example 的语法，我们直接看一下它的源码吧，比较简单。</p>
<pre class="lang-java" data-nodeid="972"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Example</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
   <span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">Example&lt;T&gt; <span class="hljs-title">of</span><span class="hljs-params">(T probe)</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TypedExample&lt;&gt;(probe, ExampleMatcher.matching());
   }
   <span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">Example&lt;T&gt; <span class="hljs-title">of</span><span class="hljs-params">(T probe, ExampleMatcher matcher)</span> </span>{
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TypedExample&lt;&gt;(probe, matcher);
   }
   <span class="hljs-comment">//实体参数</span>
   <span class="hljs-function">T <span class="hljs-title">getProbe</span><span class="hljs-params">()</span></span>;
   <span class="hljs-comment">//匹配器</span>
   <span class="hljs-function">ExampleMatcher <span class="hljs-title">getMatcher</span><span class="hljs-params">()</span></span>;
   <span class="hljs-comment">//回顾一下我们上一课时讲解的类型，这个是返回实体参数的Class Type；</span>
   <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
   <span class="hljs-function"><span class="hljs-keyword">default</span> Class&lt;T&gt; <span class="hljs-title">getProbeType</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> (Class&lt;T&gt;) ProxyUtils.getUserClass(getProbe().getClass());
   }
}
</code></pre>
<p data-nodeid="973">而 TypedExample 这个类不是 public 的，看如下源码。</p>
<pre class="lang-java" data-nodeid="974"><code data-language="java"><span class="hljs-meta">@ToString</span>
<span class="hljs-meta">@EqualsAndHashCode</span>
<span class="hljs-meta">@RequiredArgsConstructor(access = AccessLevel.PACKAGE)</span>
<span class="hljs-meta">@Getter</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TypedExample</span>&lt;<span class="hljs-title">T</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">Example</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-meta">@NonNull</span> T probe;
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> <span class="hljs-meta">@NonNull</span> ExampleMatcher matcher;
}
</code></pre>
<p data-nodeid="975">其中我们发现三个类：Probe、ExampleMatcher 和 Example，分别做如下解释：</p>
<ul data-nodeid="976">
<li data-nodeid="977">
<p data-nodeid="978">Probe：这是具有填充字段的域对象的实际实体类，即查询条件的封装类（又可以理解为查询条件参数），必填。</p>
</li>
<li data-nodeid="979">
<p data-nodeid="980">ExampleMatcher：ExampleMatcher 有关如何匹配特定字段的匹配规则，它可以重复使用在多个实例中，必填。</p>
</li>
<li data-nodeid="981">
<p data-nodeid="982">Example：Example 由 Probe 探针和 ExampleMatcher 组成，它用于创建查询，即组合查询参数和参数的匹配规则。</p>
</li>
</ul>
<p data-nodeid="983">通过 Example 的源码，我们发现想创建 Example 的话，只有两个方法：</p>
<ol data-nodeid="984">
<li data-nodeid="985">
<p data-nodeid="986">static <code data-backticks="1" data-nodeid="1125"> &lt;T&gt;</code> Example<code data-backticks="1" data-nodeid="1127">&lt;T&gt;</code> of(T probe)：需要一个实体参数，即查询的条件。而里面的 ExampleMatcher 采用默认的 ExampleMatcher.matching()； 表示忽略 Null，所有字段采用精准匹配。</p>
</li>
<li data-nodeid="987">
<p data-nodeid="988">static <code data-backticks="1" data-nodeid="1130">&lt;T&gt;</code> Example<code data-backticks="1" data-nodeid="1132">&lt;T&gt;</code> of(T probe, ExampleMatcher matcher)：需要两个参数构建 Example，也就表示了 ExampleMatcher 自由组合规则，正如我们上面的测试用例里面的代码一样。</p>
</li>
</ol>
<p data-nodeid="989">那么现在又遇到个类：ExampleMatcher，我们分析一下它的语法。</p>
<h4 data-nodeid="990">ExampleMatcher 语法分析</h4>
<p data-nodeid="991">我们通过分析 ExampleMatcher 的源码来分析一下其用法。</p>
<p data-nodeid="992">首先打开 Structure 视图，看看里面对外暴露的方法都有哪些。</p>
<p data-nodeid="993"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFFGAHjlEAAOzUKkyjE0156.png" alt="Drawing 2.png" data-nodeid="1140"></p>
<p data-nodeid="994">通过 Structure 视图可以很容易地发现，我们要关心的方法都是这些 public 类型的返回 ExampleMatcher 的方法，那么我们把这些方法搞明白了是不是就可以掌握其详细用法了呢？下面看看它的实现类。</p>
<p data-nodeid="995"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFFeAcXU9AACEfIRngF4284.png" alt="Drawing 3.png" data-nodeid="1144"></p>
<p data-nodeid="996">TypedExampleMatcher 不是 public 类型的，所以我们可以基本上不用看了，主要看一下接口里面给我们暴露了哪些实例化方法。</p>
<h5 data-nodeid="997">初始化 ExampleMatcher 实例的方法</h5>
<p data-nodeid="998">查看初始化 ExampleMatcher 实例的方法时，我们发现只有如下三个。</p>
<p data-nodeid="999">先看一下前两个方法：</p>
<pre class="lang-java" data-nodeid="1000"><code data-language="java"><span class="hljs-comment">//默认matching方法</span>
<span class="hljs-function"><span class="hljs-keyword">static</span> ExampleMatcher <span class="hljs-title">matching</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> matchingAll();
}
<span class="hljs-comment">//matchingAll，默认的方法</span>
<span class="hljs-function"><span class="hljs-keyword">static</span> ExampleMatcher <span class="hljs-title">matchingAll</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TypedExampleMatcher().withMode(MatchMode.ALL);
}
</code></pre>
<p data-nodeid="1001">我们看到上面的两个方法所表达的意思是一样的，只不过一个是默认，一个是方法名上面有语义的。两者采用的都是 MatchMode.ALL 的模式，即 AND 模式，生成的 SQL 为如下形式：</p>
<pre class="lang-java" data-nodeid="1002"><code data-language="java">Hibernate: select useraddres0_.id as id1_2_, useraddres0_.address as address2_2_, useraddres0_.user_id as user_id3_2_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id=user1_.id where user1_.age=<span class="hljs-number">20</span> and user1_.name=? and (user1_.email like ? escape ?) and (useraddres0_.address like ? escape ?) limit ?
</code></pre>
<p data-nodeid="1003">可以看到，这些查询条件之间都是 AND 的关系。</p>
<p data-nodeid="1004">我们再看一下方法三：</p>
<pre class="lang-java" data-nodeid="1005"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">static</span> ExampleMatcher <span class="hljs-title">matchingAny</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> TypedExampleMatcher().withMode(MatchMode.ANY);
}
</code></pre>
<p data-nodeid="1006">第三个方法和前面两个方法的区别在于：第三个 MatchMode.ANY，表示查询条件是 or 的关系，我们看一下 SQL：</p>
<pre class="lang-java" data-nodeid="1007"><code data-language="java">Hibernate: <span class="hljs-function">select <span class="hljs-title">count</span><span class="hljs-params">(useraddres0_.id)</span> as col_0_0_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id</span>=user1_.id where useraddres0_.address like ? escape ? or user1_.age=<span class="hljs-number">20</span> or user1_.email like ? escape ? or user1_.name=?
</code></pre>
<p data-nodeid="1008">以上就是三个初始化 ExampleMatcher 实例的方法，你在运用中需要注意 and 和 or 的关系。<br>
那么，我们再看一下 ExampleMatcher 语法给我们暴露的方法有哪些。</p>
<h5 data-nodeid="1009">ExampleMatcher 语法给我们暴露的方法</h5>
<p data-nodeid="1010"><strong data-nodeid="1160">忽略大小写</strong></p>
<p data-nodeid="1011">关于忽略大小写，我们看下代码：</p>
<pre class="lang-java" data-nodeid="1012"><code data-language="java"><span class="hljs-comment">//默认忽略大小写的方式，默认 False。</span>
<span class="hljs-function">ExampleMatcher <span class="hljs-title">withIgnoreCase</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> defaultIgnoreCase)</span></span>;
<span class="hljs-comment">//提供了一个默认的实现方法，忽略大小写；</span>
<span class="hljs-function"><span class="hljs-keyword">default</span> ExampleMatcher <span class="hljs-title">withIgnoreCase</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> withIgnoreCase(<span class="hljs-keyword">true</span>);
}
<span class="hljs-comment">//哪些属性的paths忽略大小写，可以指定多个参数；</span>
<span class="hljs-function">ExampleMatcher <span class="hljs-title">withIgnoreCase</span><span class="hljs-params">(String... propertyPaths)</span></span>;
</code></pre>
<p data-nodeid="1013"><strong data-nodeid="1165">NULL 值的 property 怎么处理</strong></p>
<p data-nodeid="1014">暴露的 Null 值处理方式如下：</p>
<pre class="lang-java" data-nodeid="1015"><code data-language="java"><span class="hljs-function">ExampleMatcher <span class="hljs-title">withNullHandler</span><span class="hljs-params">(NullHandler nullHandler)</span></span>;
</code></pre>
<p data-nodeid="1016">我们直接看参数 NullHandler枚举值即可，有两个可选值：INCLUDE（包括）、IGNORE（忽略），其中要注意：</p>
<ul data-nodeid="1017">
<li data-nodeid="1018">
<p data-nodeid="1019">标识作为条件的实体对象中，一个属性值（条件值）为 Null 时，是否参与过滤；</p>
</li>
<li data-nodeid="1020">
<p data-nodeid="1021">当该选项值是 INCLUDE 时，表示仍参与过滤，会匹配数据库表中该字段值是 Null 的记录；</p>
</li>
<li data-nodeid="1022">
<p data-nodeid="1023">若为 IGNORE 值，表示不参与过滤。</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="1024"><code data-language="java"><span class="hljs-comment">//提供一个默认实现方法，忽略 NULL 属性；</span>
<span class="hljs-function"><span class="hljs-keyword">default</span> ExampleMatcher <span class="hljs-title">withIgnoreNullValues</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> withNullHandler(NullHandler.IGNORE);
}
<span class="hljs-comment">//把 NULL 属性值作为查询条件</span>
<span class="hljs-function"><span class="hljs-keyword">default</span> ExampleMatcher <span class="hljs-title">withIncludeNullValues</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">return</span> withNullHandler(NullHandler.INCLUDE);
}
</code></pre>
<p data-nodeid="1025">到这里看一下，把 NULL 属性值作为查询条件，会执行什么样的 SQL：</p>
<pre class="lang-java" data-nodeid="1026"><code data-language="java">Hibernate: select useraddres0_.id as id1_2_, useraddres0_.address as address2_2_, useraddres0_.user_id as user_id3_2_ from user_address useraddres0_ inner join user user1_ on useraddres0_.user_id=user1_.<span class="hljs-function">id <span class="hljs-title">where</span> <span class="hljs-params">(user1_.id is <span class="hljs-keyword">null</span>)</span> <span class="hljs-title">and</span> <span class="hljs-params">(user1_.update_date is <span class="hljs-keyword">null</span>)</span> and user1_.age</span>=<span class="hljs-number">20</span> and (user1_.create_date is <span class="hljs-keyword">null</span>) <span class="hljs-function">and <span class="hljs-title">lower</span><span class="hljs-params">(user1_.name)</span></span>=? and (lower(user1_.email) like ? escape ?) and (user1_.sex is <span class="hljs-keyword">null</span>) and (lower(useraddres0_.address) like ? escape ?) and (useraddres0_.id is <span class="hljs-keyword">null</span>) limit ?
</code></pre>
<p data-nodeid="1027">这样就会导致我们一条数据都查不出来了。<br>
<strong data-nodeid="1177">忽略某些 Paths，不参加查询条件</strong></p>
<pre class="lang-java" data-nodeid="1028"><code data-language="java"><span class="hljs-comment">//忽略某些属性列表，不参与查询过滤条件。</span>
<span class="hljs-function">ExampleMatcher <span class="hljs-title">withIgnorePaths</span><span class="hljs-params">(String... ignoredPaths)</span></span>;
</code></pre>
<h5 data-nodeid="1029">字符串字段默认的匹配规则</h5>
<pre class="lang-java" data-nodeid="1030"><code data-language="java"><span class="hljs-function">ExampleMatcher <span class="hljs-title">withStringMatcher</span><span class="hljs-params">(StringMatcher defaultStringMatcher)</span></span>;
</code></pre>
<p data-nodeid="1031">关于默认字符串的匹配方式，枚举类型有 6 个可选值，DEFAULT（默认，效果同 EXACT）、EXACT（相等）、STARTING（开始匹配）、ENDING（结束匹配）、CONTAINING（包含，模糊匹配）、REGEX（正则表达式）。<br>
字符串匹配规则，我们和 JPQL 对应到一起举例，如下表所示：</p>
<p data-nodeid="1032"><img src="https://s0.lgstatic.com/i/image/M00/5D/41/Ciqc1F-EFHuAXsn3AABiCE6_I0I978.png" alt="Drawing 4.png" data-nodeid="1184"></p>
<p data-nodeid="1033">相关代码如下：</p>
<pre class="lang-java" data-nodeid="1034"><code data-language="java"><span class="hljs-function">ExampleMatcher <span class="hljs-title">withMatcher</span><span class="hljs-params">(String propertyPath, GenericPropertyMatcher genericPropertyMatcher)</span></span>;
</code></pre>
<p data-nodeid="1035">这里显示的是指定某些属性的匹配规则，我们看一下 GenericPropertyMatcher 是什么东西，它都提供了哪些方法。</p>
<p data-nodeid="1036">如下图，基本可以看出来都是针对字符串属性提供的匹配规则，也就是可以通过这个方法定制不同属性的 StringMatcher 规则。</p>
<p data-nodeid="1037"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFIiAKEMxAAFkdJSuuX4896.png" alt="Drawing 5.png" data-nodeid="1190"></p>
<p data-nodeid="1038">到这里，语法部分我们就学习完了，下面看一个完整的例子感受一下。</p>
<h4 data-nodeid="1039">完整的例子</h4>
<p data-nodeid="1040">下面是一个关于咱们上面所说的暴露的方法的使用的例子，你可以跟着我的步骤自己动手练习一下。</p>
<pre class="lang-java" data-nodeid="1041"><code data-language="java"><span class="hljs-comment">//创建匹配器，即如何使用查询条件</span>
ExampleMatcher exampleMatcher = ExampleMatcher
      <span class="hljs-comment">//采用默认and的查询方式</span>
      .matchingAll()
      <span class="hljs-comment">//忽略大小写</span>
      .withIgnoreCase()
      <span class="hljs-comment">//忽略所有null值的字段</span>
      .withIgnoreNullValues()
      .withIgnorePaths(<span class="hljs-string">"id"</span>,<span class="hljs-string">"createDate"</span>)
      <span class="hljs-comment">//默认采用精准匹配规则</span>
      .withStringMatcher(ExampleMatcher.StringMatcher.EXACT)
      <span class="hljs-comment">//级联查询，字段user.email采用字符前缀匹配规则</span>
      .withMatcher(<span class="hljs-string">"user.email"</span>, ExampleMatcher.GenericPropertyMatchers.startsWith())
      <span class="hljs-comment">//特殊指定address字段采用后缀匹配</span>
      .withMatcher(<span class="hljs-string">"address"</span>, ExampleMatcher.GenericPropertyMatchers.endsWith());
Page&lt;UserAddress&gt; u = userAddressRepository.findAll(Example.of(address,exampleMatcher), PageRequest.of(<span class="hljs-number">0</span>,<span class="hljs-number">2</span>));
</code></pre>
<p data-nodeid="1042">这时候可能会有同学问了，我是怎么知道默认值的呢？我们直接看类的构造方法就可以了，如下所示：</p>
<p data-nodeid="1043"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFJGAOa8BAAGc6Bk2F3g271.png" alt="Drawing 6.png" data-nodeid="1197"></p>
<p data-nodeid="1044">从源码中我们可以看到，实现类的构造方法只有一个，就是“赋值默认”的方式。下面我整理了一些在使用这个语法时需要考虑的细节。</p>
<h4 data-nodeid="1045">ExampleExceutor 使用中需要考虑的因素</h4>
<ol data-nodeid="1046">
<li data-nodeid="1047">
<p data-nodeid="1048">Null 值的处理：当某个条件值为 Null 时，是应当忽略这个过滤条件，还是应当去匹配数据库表中该字段值是 Null 的记录呢？</p>
</li>
<li data-nodeid="1049">
<p data-nodeid="1050">忽略某些属性值：一个实体对象，有许多个属性，是否每个属性都参与过滤？是否可以忽略某些属性？</p>
</li>
<li data-nodeid="1051">
<p data-nodeid="1052">不同的过滤方式：同样是作为 String 值，可能“姓名”希望精确匹配，“地址”希望模糊匹配，如何做到？</p>
</li>
</ol>
<p data-nodeid="1053">那么接下来我们分析一下源码看看其原理，说了这么半天，它到底和 JpaSpecificationExecutor 什么关系呢？我们接着看。</p>
<h3 data-nodeid="1054">QueryByExampleExecutor 源码分析</h3>
<p data-nodeid="1055">怎么分析源码也很简单，我们看一下上面的我们 findAll 的方法调用之处。</p>
<p data-nodeid="1056"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFJmAO7ItAABKDcL98Uc576.png" alt="Drawing 7.png" data-nodeid="1208"></p>
<p data-nodeid="1057">从而找到 findAll 方法的实现类，如下所示：</p>
<p data-nodeid="1058"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFKCAOUw0AAaMM8yZ64k573.png" alt="Drawing 8.png" data-nodeid="1212"></p>
<p data-nodeid="1059">通过 Debug 断点我们可以看到，我们刚才组合出来的 Example 对象，这个时候被封装成了 ExampleSpecification 对象，那么我们接着往下看方法里面的关键内容。</p>
<pre class="lang-java" data-nodeid="1060"><code data-language="java">TypedQuery&lt;S&gt; query = getQuery(<span class="hljs-keyword">new</span> ExampleSpecification&lt;&gt;(example, escapeCharacter), probeType, pageable);
</code></pre>
<p data-nodeid="1061">getQuery 方法是创建 Query 的关键，因为它里面做了条件的转化逻辑。那么我们再看一下参数 ExampleSpecification 的源码，发现它是接口 Specification 的实现类，并且是非公开的实现类，可以通过接口对外暴露 and、or、not、where 等组合条件的查询条件。</p>
<p data-nodeid="1062"><img src="https://s0.lgstatic.com/i/image/M00/5D/41/Ciqc1F-EFKeAOHpRAASLh36FrZI858.png" alt="Drawing 9.png" data-nodeid="1217"></p>
<p data-nodeid="1063">我们接着看上面的 getQuery 方法的实现，可以看到接收的参数是 Specification<code data-backticks="1" data-nodeid="1219">&lt;S&gt;</code>接口，所以不用关心实现类是什么。</p>
<p data-nodeid="1256"><img src="https://s0.lgstatic.com/i/image/M00/5D/41/Ciqc1F-EFK2AWfUgAAEJccNGWh4199.png" alt="Drawing 10.png" data-nodeid="1260"></p>
<p data-nodeid="1257">我们接着再看这个断点的 getQuery 方法：</p>

<p data-nodeid="1065"><img src="https://s0.lgstatic.com/i/image/M00/5D/41/Ciqc1F-EFLOAfcLIAAEtDgDfmQU527.png" alt="Drawing 11.png" data-nodeid="1228"></p>
<p data-nodeid="1066">里面有一段代码会调用 applySpecificationToCriteria 生成 root，并由 Root 作为参数生成 Query，从而交给 EM（EntityManager）进行查询。</p>
<p data-nodeid="1067">我们再来看一下关键的 applySpecificationToCriteria 方法。</p>
<p data-nodeid="1068"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFLyAJh4iAAFuOV3pYzA214.png" alt="Drawing 12.png" data-nodeid="1233"></p>
<p data-nodeid="1069">根据 Specification 调用 toPredicate 方法，生成 Predicate，从而实现查询需求。</p>
<p data-nodeid="1070">现在我们已经对 QueryByExampleExecutor 的用法和实现原理基本掌握了，我们再来看一个十分相似的接口：JpaSpecificationExecutor 是干什么用的。</p>
<h3 data-nodeid="1071">JpaSpecificationExecutor 接口结构</h3>
<p data-nodeid="1072">正如我们开篇提到的【图一：Repository 类图】，JpaSpecificationExecutor 是 JPA 里面的另一个接口分支。我们先来看看它的基本语法。</p>
<p data-nodeid="1073"><img src="https://s0.lgstatic.com/i/image/M00/5D/4C/CgqCHl-EFMSABjEBAAEBE-nLmV0807.png" alt="Drawing 13.png" data-nodeid="1240"></p>
<p data-nodeid="1074">我们通过查看 JpaSpecificationExecutor 的 Structure 图会发现，方法就有这么几个，细心的同学这个时候会发现它的参数 Specification，正是我们分析 QueryByExampleExecutor 的原理时候使用的 Specification。</p>
<p data-nodeid="1075">那么 JpaSpecificationExecutor 帮我们解决了哪些问题呢？</p>
<h3 data-nodeid="1076">JpaSpecificationExecutor 解决了哪些问题</h3>
<ol data-nodeid="1077">
<li data-nodeid="1078">
<p data-nodeid="1079">我们通过 QueryByExampleExecutor 的使用方法和原理分析，不难发现，JpaSpecificationExecutor 的查询条件 Specification 十分灵活，可以帮我们解决动态查询条件的问题，正如 QueryByExampleExecutor 的用法一样；</p>
</li>
<li data-nodeid="1080">
<p data-nodeid="1081">它提供的 Criteria API 的使用封装，可以用于动态生成 Query 来满足我们业务中的各种复杂场景；</p>
</li>
<li data-nodeid="1082">
<p data-nodeid="1083">既然QueryByExampleExecutor 能利用 Specification 封装成框架，我们是不是也可以利用 JpaSpecificationExecutor 封装成框架呢？这样就学会了举一反三。</p>
</li>
</ol>
<h3 data-nodeid="1084">总结</h3>
<p data-nodeid="1085">本课时我们通过分析QueryByExampleExecutor 的详细用法和实现原理，知道了 Specification 的应用场景，那么下一课时我会详细介绍JpaSpecificationExecutor 的用法和实现原理。</p>
<p data-nodeid="1086">另外本课时也提供了一种学习框架的思路，就是怎么通过源码来详细掌握语法。</p>
<p data-nodeid="1087">总之，保持一颗好奇心，不断深挖，你才能掌握得更加全面。本节课就到这里了，如果你觉得有帮助，欢迎你留言讨论和分享，我们下一课时再见。</p>
<blockquote data-nodeid="1088">
<p data-nodeid="1089" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="1255">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **兴：
> 老师，能不能增加一些关于JpaSpecificationExecutor实际应用场景实战方面的内容，在实际应用经常不知道在什么情况下使用JpaSpecificationExecutor，什么情况下使用@Query来自己编写SQL？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 写框架的时候用JpaSpecificationExecutor，正常业务建议全部使用@Query来自己编写JPQL

##### **博：
> 老师 可以多讲一下">JpaSpecificationExecutor的具体使用吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 正如我说的，只有写框架的时候基本才需要，例如我在文章里面举例的MySpecification

##### **7519：
> 想问问老师如果User 和 Role是多对多的，现在关联查询，两边都用了上了查询条件，查询出来的结果好像不对，List() { @Override query, CriteriaBuilder criteriaBuilder) { if(!Constants.ALL.equals(userName)) { } if(!Constants.ALL.equals(roleName)) { } }结果是先关联查出List 然后根据这个userId去查出所有的Role，也就是Role里的查询条件就不起作用了，想问问老师这种问题该怎么解决，十分感谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 把sql打印出来，看一下，然后debug一下，一步一步检查

