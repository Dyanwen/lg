<p data-nodeid="72810" class="">通过上一课时，我们了解到 JpaSpecificationExecutor 给我们提供了动态查询或者写框架的一种思路，那么这节课我们来看一下 JpaSpecificationExecutor 的详细用法和原理，及其实战应用场景中如何实现自己的框架。</p>
<p data-nodeid="72811">在开始讲解之前，请先思考几个问题：</p>
<ol data-nodeid="72812">
<li data-nodeid="72813">
<p data-nodeid="72814">JpaSpecificationExecutor 如何创建？</p>
</li>
<li data-nodeid="72815">
<p data-nodeid="72816">它的使用方法有哪些？</p>
</li>
<li data-nodeid="72817">
<p data-nodeid="72818">toPredicate 方法如何实现？</p>
</li>
</ol>
<p data-nodeid="72819">带着这些问题，我们开始探索。先看一个例子感受一下 JpaSpecificationExecutor 具体的用法。</p>
<h3 data-nodeid="72820">JpaSpecificationExecutor 使用案例</h3>
<p data-nodeid="72821">我们假设一个后台管理页面根据 name 模糊查询、sex 精准查询、age 范围查询、时间区间查询、address 的 in 查询这样一个场景，来查询 user 信息，我们看看这个例子应该怎么写。</p>
<p data-nodeid="72822"><strong data-nodeid="72946">第一步：创建 User 和 UserAddress 两个实体。</strong> 代码如下：</p>
<pre class="lang-java" data-nodeid="72823"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> com.fasterxml.jackson.annotation.JsonIgnore;
<span class="hljs-keyword">import</span> lombok.*;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-keyword">import</span> java.io.Serializable;
<span class="hljs-keyword">import</span> java.time.Instant;
<span class="hljs-keyword">import</span> java.util.Date;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-comment">/**
* 用户基本信息表
**/</span>
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString(exclude = "addresses")</span>
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
   <span class="hljs-meta">@OneToMany(mappedBy = "user")</span>
   <span class="hljs-meta">@JsonIgnore</span>
   <span class="hljs-keyword">private</span> List&lt;UserAddress&gt; addresses;
}
<span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">SexEnum</span> </span>{
   BOY,GIRL
}
<span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> lombok.*;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-comment">/**
 * 用户地址表
 */</span>
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
   <span class="hljs-keyword">private</span> User user;
}
</code></pre>
<p data-nodeid="72824"><strong data-nodeid="72950">第二步：创建 UserRepository 继承 JpaSpecificationExecutor 接口。</strong></p>
<pre class="lang-java" data-nodeid="72825"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaSpecificationExecutor;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt;, <span class="hljs-title">JpaSpecificationExecutor</span>&lt;<span class="hljs-title">User</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="72826"><strong data-nodeid="72954">第三步：创建一个测试用例进行测试。</strong></p>
<pre class="lang-java" data-nodeid="72827"><code data-language="java"><span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-meta">@TestInstance(TestInstance.Lifecycle.PER_CLASS)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserJpeTest</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository userRepository;
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserAddressRepository userAddressRepository;
    <span class="hljs-keyword">private</span> Date now = <span class="hljs-keyword">new</span> Date();
    <span class="hljs-comment">/**
     * 提前创建一些数据
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
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testSPE</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-comment">//模拟请求参数</span>
        User userQuery = User.builder()
                .name(<span class="hljs-string">"jack"</span>)
                .email(<span class="hljs-string">"123456@126.com"</span>)
                .sex(SexEnum.BOY)
                .age(<span class="hljs-number">20</span>)
                .addresses(Lists.newArrayList(UserAddress.builder().address(<span class="hljs-string">"shanghai"</span>).build()))
                .build();
                <span class="hljs-comment">//假设的时间范围参数</span>
        Instant beginCreateDate = Instant.now().plus(-<span class="hljs-number">2</span>, ChronoUnit.HOURS);
        Instant endCreateDate = Instant.now().plus(<span class="hljs-number">1</span>, ChronoUnit.HOURS);
        <span class="hljs-comment">//利用Specification进行查询</span>
        Page&lt;User&gt; users = userRepository.findAll(<span class="hljs-keyword">new</span> Specification&lt;User&gt;() {
            <span class="hljs-meta">@Override</span>
            <span class="hljs-function"><span class="hljs-keyword">public</span> Predicate <span class="hljs-title">toPredicate</span><span class="hljs-params">(Root&lt;User&gt; root, CriteriaQuery&lt;?&gt; query, CriteriaBuilder cb)</span> </span>{
                List&lt;Predicate&gt; ps = <span class="hljs-keyword">new</span> ArrayList&lt;Predicate&gt;();
                <span class="hljs-keyword">if</span> (StringUtils.isNotBlank(userQuery.getName())) {
                    <span class="hljs-comment">//我们模仿一下like查询，根据name模糊查询</span>
                    ps.add(cb.like(root.get(<span class="hljs-string">"name"</span>),<span class="hljs-string">"%"</span> +userQuery.getName()+<span class="hljs-string">"%"</span>));
                }
                <span class="hljs-keyword">if</span> (userQuery.getSex()!=<span class="hljs-keyword">null</span>){
                    <span class="hljs-comment">//equal查询条件，这里需要注意，直接传递的是枚举</span>
                    ps.add(cb.equal(root.get(<span class="hljs-string">"sex"</span>),userQuery.getSex()));
                }
                <span class="hljs-keyword">if</span> (userQuery.getAge()!=<span class="hljs-keyword">null</span>){
                    <span class="hljs-comment">//greaterThan大于等于查询条件</span>
                    ps.add(cb.greaterThan(root.get(<span class="hljs-string">"age"</span>),userQuery.getAge()));
                }
                <span class="hljs-keyword">if</span> (beginCreateDate!=<span class="hljs-keyword">null</span>&amp;&amp;endCreateDate!=<span class="hljs-keyword">null</span>){
                    <span class="hljs-comment">//根据时间区间去查询创建</span>
                    ps.add(cb.between(root.get(<span class="hljs-string">"createDate"</span>),beginCreateDate,endCreateDate));
                }
                <span class="hljs-keyword">if</span> (!ObjectUtils.isEmpty(userQuery.getAddresses())) {
                    <span class="hljs-comment">//联表查询，利用root的join方法，根据关联关系表里面的字段进行查询。</span>
                    ps.add(cb.in(root.join(<span class="hljs-string">"addresses"</span>).get(<span class="hljs-string">"address"</span>)).value(userQuery.getAddresses().stream().map(a-&gt;a.getAddress()).collect(Collectors.toList())));
                }
                <span class="hljs-keyword">return</span> query.where(ps.toArray(<span class="hljs-keyword">new</span> Predicate[ps.size()])).getRestriction();
            }
        }, PageRequest.of(<span class="hljs-number">0</span>, <span class="hljs-number">2</span>));
        System.out.println(users);
    }
}
</code></pre>
<p data-nodeid="72828">我们看一下执行结果。</p>
<pre class="lang-java" data-nodeid="72829"><code data-language="java">Hibernate: select user0_.id as id1_1_, user0_.age as age2_1_, user0_.create_date as create_d3_1_, user0_.email as email4_1_, user0_.name as name5_1_, user0_.sex as sex6_1_, user0_.update_date as update_d7_1_ from user user0_ inner join user_address addresses1_ on user0_.id=addresses1_.<span class="hljs-function">user_id <span class="hljs-title">where</span> <span class="hljs-params">(user0_.name like ?)</span> and user0_.sex</span>=? and user0_.age&gt;<span class="hljs-number">20</span> and (user0_.create_date between ? and ?) and (addresses1_.<span class="hljs-function">address <span class="hljs-title">in</span> <span class="hljs-params">(?)</span>) limit ?
</span></code></pre>
<p data-nodeid="72830">此 SQL 的参数如下：</p>
<p data-nodeid="72831"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw4GAVwjvAACBlHuTFN0497.png" alt="Drawing 0.png" data-nodeid="72959"></p>
<p data-nodeid="72832">此 SQL 就是查询 User inner Join user_address 之后组合成的查询 SQL，基本符合我们的预期，即不同的查询条件。我们通过这个例子大概知道了 JpaSpecificationExecutor 的用法，那么它具体是什么呢？</p>
<h3 data-nodeid="72833">JpaSpecificationExecutor 语法详解</h3>
<p data-nodeid="72834">我们依然通过看 JpaSpecificationExecutor 的源码，来了解一下它的几个使用方法，如下所示：</p>
<pre class="lang-java" data-nodeid="72835"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">JpaSpecificationExecutor</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
   <span class="hljs-comment">//根据 Specification 条件查询单个对象，要注意的是，如果条件能查出来多个会报错</span>
   <span class="hljs-function">T <span class="hljs-title">findOne</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec)</span></span>;
   <span class="hljs-comment">//根据 Specification 条件，查询 List 结果</span>
   <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec)</span></span>;
   <span class="hljs-comment">//根据 Specification 条件，分页查询</span>
   <span class="hljs-function">Page&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec, Pageable pageable)</span></span>;
   <span class="hljs-comment">//根据 Specification 条件，带排序的查询结果</span>
   <span class="hljs-function">List&lt;T&gt; <span class="hljs-title">findAll</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec, Sort sort)</span></span>;
   <span class="hljs-comment">//根据 Specification 条件，查询数量</span>
   <span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">count</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec)</span></span>;
}
</code></pre>
<p data-nodeid="72836">其返回结果和 Pageable、Sort，我们在前面课时都有介绍过，这里我们重点关注一下 Specification。看一下 Specification 接口的代码。</p>
<p data-nodeid="72837"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw4uAGaD2AADHMlVN_q0440.png" alt="Drawing 1.png" data-nodeid="72968"></p>
<p data-nodeid="72838">通过看其源码就会发现里面提供的方法很简单。其中，下面一段代码表示组合的 and 关系的查询条件。</p>
<pre class="lang-java" data-nodeid="72839"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">default</span> Specification&lt;T&gt; <span class="hljs-title">and</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; other)</span> </span>{
   <span class="hljs-keyword">return</span> composed(<span class="hljs-keyword">this</span>, other, (builder, left, rhs) -&gt; builder.and(left, rhs));
}
</code></pre>
<p data-nodeid="72840">下面是静态方法，创建 where 后面的 Predicate 集合。</p>
<pre class="lang-java" data-nodeid="72841"><code data-language="java"><span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">Specification&lt;T&gt; <span class="hljs-title">where</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec)</span>
</span></code></pre>
<p data-nodeid="72842">下面是默认方法，创建 or 条件的查询参数。</p>
<pre class="lang-java" data-nodeid="72843"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">default</span> Specification&lt;T&gt; <span class="hljs-title">or</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; other)</span>
</span></code></pre>
<p data-nodeid="72844">这是静态方法，创建 Not 的查询条件。</p>
<pre class="lang-java" data-nodeid="72845"><code data-language="java"><span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">Specification&lt;T&gt; <span class="hljs-title">not</span><span class="hljs-params">(<span class="hljs-meta">@Nullable</span> Specification&lt;T&gt; spec)</span>
</span></code></pre>
<p data-nodeid="72846">上面这几个方法比较简单，我就不一一细说了，我们主要看一下需要实现的方法：toPredicate。</p>
<pre class="lang-java" data-nodeid="72847"><code data-language="java"><span class="hljs-function">Predicate <span class="hljs-title">toPredicate</span><span class="hljs-params">(Root&lt;T&gt; root, CriteriaQuery&lt;?&gt; query, CriteriaBuilder criteriaBuilder)</span></span>;
</code></pre>
<p data-nodeid="72848">toPredicate 这个方法是我们用到的时候需要自己去实现的，接下来我们详细介绍一下。</p>
<p data-nodeid="72849">首先我们在刚才的 Demo 里面设置一个断点，看到如下界面。</p>
<p data-nodeid="72850"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw5SAeoXKAAMzN3mF_8I181.png" alt="Drawing 2.png" data-nodeid="72978"></p>
<p data-nodeid="72851">这里可以分别看到 Root 的实现类是 RootImpl，CriteriaQuery 的实现类是 CriteriaQueryImpl，CriteriaBuilder 的实现类是 CriteriaBuilderImpl。</p>
<pre class="lang-java" data-nodeid="72852"><code data-language="java">javax.persistence.criteria.Root
javax.persistence.criteria.CriteriaQuery
javax.persistence.criteria.CriteriaBuilder
</code></pre>
<p data-nodeid="72853">其中，上面三个接口是 Java Persistence API 定义的接口。</p>
<pre class="lang-java" data-nodeid="72854"><code data-language="java">org.hibernate.query.criteria.internal.path.RootImpl
rg.hibernate.query.criteria.internal.CriteriaQueryImpl
org.hibernate.query.criteria.internal.CriteriaBuilderImpl
</code></pre>
<p data-nodeid="72855">而这个三个实现类都是由 Hibernate 进行实现的，也就是说 JpaSpecificationExecutor 封装了原本需要我们直接操作 Hibernate 中 Criteria 的 API 方法。</p>
<p data-nodeid="72856">下面分别解释上述三个参数。</p>
<h4 data-nodeid="72857">Root<code data-backticks="1" data-nodeid="72984">&lt;User&gt;</code> root</h4>
<p data-nodeid="72858">代表了可以查询和操作的实体对象的根，如果将实体对象比喻成表名，那 root 里面就是这张表里面的字段，而这些字段只是 JPQL 的实体字段而已。我们可以通过里面的 Path get（String attributeName），来获得我们想要操作的字段。</p>
<pre class="lang-java" data-nodeid="72859"><code data-language="java">类似于我们上面的：root.get(<span class="hljs-string">"createDate"</span>)等操作
</code></pre>
<h4 data-nodeid="72860">CriteriaQuery&lt;?&gt; query</h4>
<p data-nodeid="72861">代表一个 specific 的顶层查询对象，它包含着查询的各个部分，比如 select 、from、where、group by、order by 等。CriteriaQuery 对象只对实体类型或嵌入式类型的 Criteria 查询起作用。简单理解为，它提供了查询 ROOT 的方法。常用的方法有如下几种：</p>
<p data-nodeid="72862"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw6uAJmdyAALVLoSlnLQ418.png" alt="Drawing 3.png" data-nodeid="72993"></p>
<pre class="lang-java" data-nodeid="72863"><code data-language="java">正如我们上面where的用法：query.where(.....）一样
</code></pre>
<p data-nodeid="72864">这个语法比较简单，我们在其方法后面加上相应的参数即可。下面看一个 group by 的例子。</p>
<p data-nodeid="72865"><img src="https://s0.lgstatic.com/i/image/M00/5E/D2/CgqCHl-Hw7SAYWLnAAEIg0u_SOc872.png" alt="Drawing 4.png" data-nodeid="72997"></p>
<p data-nodeid="72866">如上图所示，我们加入 groupyBy 某个字段，SQL 也会有相应的变化。那么我们再来看第三个参数。</p>
<h4 data-nodeid="72867">CriteriaBuilder cb</h4>
<p data-nodeid="72868">CriteriaBuilder 是用来构建 CritiaQuery 的构建器对象，其实就相当于条件或者条件组合，并以 Predicate 的形式返回。它基本上提供了所有常用的方法，如下所示：</p>
<p data-nodeid="72869"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw7yAZgOYAASXGLn8fS0352.png" alt="Drawing 5.png" data-nodeid="73003"></p>
<p data-nodeid="72870">我们直接通过此类的 Structure 视图就可以看到都有哪些方法。其中，and、any 等用来做查询条件的组合；类似 between、equal、exist、ge、gt、isEmpty、isTrue、in 等用来做查询条件的查询，类似下图的一些方法。</p>
<p data-nodeid="72871"><img src="https://s0.lgstatic.com/i/image/M00/5E/D3/CgqCHl-Hw8KAS41dAAKQ7MNjups722.png" alt="Drawing 6.png" data-nodeid="73007"></p>
<p data-nodeid="72872">而其中 Expression 很简单，都是通过 root.get(...) 某些字段即可返回，正如下面的用法。</p>
<pre class="lang-java" data-nodeid="72873"><code data-language="java">Predicate p1=cb.like(root.get(“name”).as(String.class), “%”+uqm.getName()+“%”);
Predicate p2=cb.equal(root.get(<span class="hljs-string">"uuid"</span>).as(Integer.class), uqm.getUuid());
Predicate p3=cb.gt(root.get(<span class="hljs-string">"age"</span>).as(Integer.class), uqm.getAge());
</code></pre>
<p data-nodeid="72874">我们利用 like、equal、gt 可以生产 Predicate，而 Predicate 可以组合查询。比如我们预定它们之间是 and 或 or 的关系：<code data-backticks="1" data-nodeid="73010">Predicate p = cb.and(p3,cb.or(p1,p2));</code></p>
<p data-nodeid="72875">我们让 p1 和 p2 是 or 的关系，并且得到的 Predicate 和 p3 又构成了 and 的关系。你可以发现它的用法还是比较简单的，正如我们开篇所说的 Junit 中 test 里面一样的写法。</p>
<p data-nodeid="72876">关于 JpaSpecificationExecutor 的语法我们就介绍完了，其实它里面的功能相当强大，只是我们发现 Spring Data JPA 介绍得并不详细，只是一笔带过，可能他们认为写框架的人才能用到，所以介绍得不多。</p>
<p data-nodeid="72877">如果你想了解更多语法的话，可以参考 Hibernate 的文档：<a href="https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#criteria" data-nodeid="73022">https://docs.jboss.org/hibernate/orm/5.2/userguide/html_single/Hibernate_User_Guide.html#criteria</a>。我们再来看看 JpaSpecificationExecutor 的实现原理。</p>
<h3 data-nodeid="72878">JpaSpecificationExecutor 原理分析</h3>
<p data-nodeid="72879">我们先看一下 JpaSpecificationExecutor 的类图。</p>
<p data-nodeid="72880"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw9KAPLUnAAEGAy5X1Y8192.png" alt="Drawing 7.png" data-nodeid="73028"></p>
<p data-nodeid="72881">从图中我们可以看得出来：</p>
<ol data-nodeid="74181">
<li data-nodeid="74182">
<p data-nodeid="74183" class="">JpaSpecificationExecutor 和 JpaRepository 是平级接口，而它们对应的实现类都是 SimpleJpaRepository；</p>
</li>
<li data-nodeid="74184">
<p data-nodeid="74185">Specification 被 ExampleSpecification 和 JpaSpecificationExector 使用，用来创建查询；</p>
</li>
<li data-nodeid="74186">
<p data-nodeid="74187">Predicate 是 JPA 协议里面提供的查询条件的根基；</p>
</li>
<li data-nodeid="74188">
<p data-nodeid="74189">SimpleJpaRepository 利用 EntityManager 和 criteria 来实现由 JpaSpecificationExector 组合的 query。</p>
</li>
</ol>



<p data-nodeid="72891">那么我们再直观地看一下 JpaSpecificationExecutor 接口里面的方法 findAll 对应的 SimpleJpaRepository 里面的实现方法 findAl，我们通过工具可以很容易地看到相应的实现方法，如下所示：</p>
<p data-nodeid="72892"><img src="https://s0.lgstatic.com/i/image/M00/5E/D3/CgqCHl-Hw9uAHAiHAAFZTdcBw2Y564.png" alt="Drawing 8.png" data-nodeid="73037"></p>
<p data-nodeid="72893">你要知道，得到 TypeQuery 就可以直接操作JPA协议里面相应的方法了，那么我们看下 getQuery（spec，pageable）的实现过程。</p>
<p data-nodeid="72894"><img src="https://s0.lgstatic.com/i/image/M00/5E/C7/Ciqc1F-Hw-qAcTArAACqAeHRq3M302.png" alt="Drawing 9.png" data-nodeid="73041"></p>
<p data-nodeid="72895">之后一步一步 debug 就可以了。</p>
<p data-nodeid="72896"><img src="https://s0.lgstatic.com/i/image/M00/5E/D3/CgqCHl-Hw_KAU1wCAAW1BokuNt8728.png" alt="Drawing 10.png" data-nodeid="73045"></p>
<p data-nodeid="72897">到了上图所示这里，就可以看到：</p>
<ol data-nodeid="72898">
<li data-nodeid="72899">
<p data-nodeid="72900">Specification<code data-backticks="1" data-nodeid="73048">&lt;S&gt;</code> spec 是我们测试用例写的 specification 的匿名实现类；</p>
</li>
<li data-nodeid="72901">
<p data-nodeid="72902">由于是方法传递，所以到第 693 行断点的时候，才会执行我们在测试用里面写的 Specification；</p>
</li>
<li data-nodeid="72903">
<p data-nodeid="72904">我们可以看到这个方法最后是调用的 EntityManager，而 EntitytManger 是 JPA 操作实体的核心原理，我在下一课时讲自定义 Repsitory 的时候再详细介绍；</p>
</li>
<li data-nodeid="72905">
<p data-nodeid="72906">从上面的方法实现过程中我们可以看得出来，所谓的JpaSpecificationExecutor原理，用一句话概况，就是利用Java Persistence API定义的接口和Hibernate的实现，做了一个简单的封装，方便我们操作JPA协议中 criteria 的相关方法。</p>
</li>
</ol>
<p data-nodeid="72907">到这里，原理和使用方法，我们基本介绍完了。你可能会有疑问：这个感觉有点重要，但是一般用不到吧？那么接下来我们看看 JpaSpecificationExecutor 的实战应用场景是什么样的。</p>
<h3 data-nodeid="72908">JpaSpecificationExecutor 实战应用场景</h3>
<p data-nodeid="72909">其实JpaSpecificationExecutor 的目的不是让我们做日常的业务查询，而是给我们提供了一种自定义 Query for rest 的架构思路，如果做日常的增删改查，肯定不如我们前面介绍的 Defining Query Methods 和 @Query 方便。</p>
<p data-nodeid="72910">那么来看下，实战过程中如何利用 JpaSpecificationExecutor 写一个框架。</p>
<h4 data-nodeid="72911">MySpecification 自定义</h4>
<p data-nodeid="72912">我们可以自定义一个Specification 的实现类，它可以实现任何实体的动态查询和各种条件的组合。</p>
<pre class="lang-java" data-nodeid="72913"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.spe;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.domain.Specification;
<span class="hljs-keyword">import</span> javax.persistence.criteria.*;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">MySpecification</span>&lt;<span class="hljs-title">Entity</span>&gt; <span class="hljs-keyword">implements</span> <span class="hljs-title">Specification</span>&lt;<span class="hljs-title">Entity</span>&gt; </span>{
   <span class="hljs-keyword">private</span> SearchCriteria criteria;
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">MySpecification</span> <span class="hljs-params">(SearchCriteria criteria)</span> </span>{
      <span class="hljs-keyword">this</span>.criteria = criteria;
   }
   <span class="hljs-comment">/**
    * 实现实体根据不同的字段、不同的Operator组合成不同的Predicate条件
    *
    * <span class="hljs-doctag">@param</span> root            must not be {<span class="hljs-doctag">@literal</span> null}.
    * <span class="hljs-doctag">@param</span> query           must not be {<span class="hljs-doctag">@literal</span> null}.
    * <span class="hljs-doctag">@param</span> builder  must not be {<span class="hljs-doctag">@literal</span> null}.
    * <span class="hljs-doctag">@return</span> a {<span class="hljs-doctag">@link</span> Predicate}, may be {<span class="hljs-doctag">@literal</span> null}.
    */</span>
   <span class="hljs-meta">@Override</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> Predicate <span class="hljs-title">toPredicate</span><span class="hljs-params">(Root&lt;Entity&gt; root, CriteriaQuery&lt;?&gt; query, CriteriaBuilder builder)</span> </span>{
      <span class="hljs-keyword">if</span> (criteria.getOperation().compareTo(Operator.GT)==<span class="hljs-number">0</span>) {
         <span class="hljs-keyword">return</span> builder.greaterThanOrEqualTo(
               root.&lt;String&gt; get(criteria.getKey()), criteria.getValue().toString());
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (criteria.getOperation().compareTo(Operator.LT)==<span class="hljs-number">0</span>) {
         <span class="hljs-keyword">return</span> builder.lessThanOrEqualTo(
               root.&lt;String&gt; get(criteria.getKey()), criteria.getValue().toString());
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (criteria.getOperation().compareTo(Operator.LK)==<span class="hljs-number">0</span>) {
         <span class="hljs-keyword">if</span> (root.get(criteria.getKey()).getJavaType() == String.class) {
            <span class="hljs-keyword">return</span> builder.like(
                  root.&lt;String&gt;get(criteria.getKey()), <span class="hljs-string">"%"</span> + criteria.getValue() + <span class="hljs-string">"%"</span>);
         } <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">return</span> builder.equal(root.get(criteria.getKey()), criteria.getValue());
         }
      }
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   }
}
</code></pre>
<p data-nodeid="72914">我们通过 <code data-backticks="1" data-nodeid="73060">&lt;Entity&gt;</code> 泛型，解决不同实体的动态查询（当然了，我只是举个例子，这个方法可以进行无限扩展）。我们通过 SearchCriteria 可以知道不同的字段是什么、值是什么、如何操作的，看一下代码：</p>
<pre class="lang-java" data-nodeid="72915"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.spe;
<span class="hljs-keyword">import</span> lombok.*;
<span class="hljs-comment">/**
 * <span class="hljs-doctag">@author</span> jack，实现不同的查询条件，不同的操作，针对Value;
 */</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SearchCriteria</span> </span>{
   <span class="hljs-keyword">private</span> String key;
   <span class="hljs-keyword">private</span> Operator operation;
   <span class="hljs-keyword">private</span> Object value;
}
</code></pre>
<p data-nodeid="72916">其中的 Operator 也是我们自定义的。</p>
<pre class="lang-java" data-nodeid="72917"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.spe;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">Operator</span> </span>{
   <span class="hljs-comment">/**
    * 等于
    */</span>
   EQ(<span class="hljs-string">"="</span>),
   <span class="hljs-comment">/**
    * 等于
    */</span>
   LK(<span class="hljs-string">":"</span>),
   <span class="hljs-comment">/**
    * 不等于
    */</span>
   NE(<span class="hljs-string">"!="</span>),
   <span class="hljs-comment">/**
    * 大于
    */</span>
   GT(<span class="hljs-string">"&gt;"</span>),
   <span class="hljs-comment">/**
    * 小于
    */</span>
   LT(<span class="hljs-string">"&lt;"</span>),
   <span class="hljs-comment">/**
    * 大于等于
    */</span>
   GE(<span class="hljs-string">"&gt;="</span>);
   Operator(String operator) {
      <span class="hljs-keyword">this</span>.operator = operator;
   }
   <span class="hljs-keyword">private</span> String operator;
}
</code></pre>
<p data-nodeid="72918">在 Operator 枚举里面定义了逻辑操作符（大于、小于、不等于、等于、大于等于……也可以自己扩展），并在 MySpecification 里面进行实现。那么我们来看看它是怎么用的，写一个测试用例试一下。</p>
<pre class="lang-java" data-nodeid="72919"><code data-language="java"><span class="hljs-comment">/**
 * 测试自定义的Specification语法
 */</span>
<span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">givenLast_whenGettingListOfUsers_thenCorrect</span><span class="hljs-params">()</span> </span>{
    MySpecification&lt;User&gt; name =
        <span class="hljs-keyword">new</span> MySpecification&lt;User&gt;(<span class="hljs-keyword">new</span> SearchCriteria(<span class="hljs-string">"name"</span>, Operator.LK, <span class="hljs-string">"jack"</span>));
MySpecification&lt;User&gt; age =
        <span class="hljs-keyword">new</span> MySpecification&lt;User&gt;(<span class="hljs-keyword">new</span> SearchCriteria(<span class="hljs-string">"age"</span>, Operator.GT, <span class="hljs-number">2</span>));
List&lt;User&gt; results = userRepository.findAll(Specification.where(name).and(age));
    System.out.println(results.get(<span class="hljs-number">0</span>).getName());
}
</code></pre>
<p data-nodeid="72920">你就会发现，我们在调用findAll 组合 Predicate 的时候就会非常简单，省去了各种条件的判断和组合，而省去的这些逻辑可以全部在我们的框架代码 MySpecification 里面实现。</p>
<p data-nodeid="72921">那么如果把这个扩展到 API 接口层面会是什么样的结果呢？我们来看下。</p>
<h4 data-nodeid="72922">利用 Specification 创建 search 为查询条件的 Rest API 接口</h4>
<p data-nodeid="72923">先创建一个 Controller，用来接收 search 这样的查询条件：类似 userssearch=lastName:doe,age&gt;25 的参数。</p>
<pre class="lang-java" data-nodeid="72924"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.web;
<span class="hljs-keyword">import</span> com.example.jpa.example1.User;
<span class="hljs-keyword">import</span> com.example.jpa.example1.UserRepository;
<span class="hljs-keyword">import</span> com.example.jpa.example1.spe.SpecificationsBuilder;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.domain.Specification;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.*;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserController</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository repo;
    <span class="hljs-meta">@RequestMapping(method = RequestMethod.GET, value = "/users")</span>
    <span class="hljs-meta">@ResponseBody</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;User&gt; <span class="hljs-title">search</span><span class="hljs-params">(<span class="hljs-meta">@RequestParam(value = "search")</span> String search)</span> </span>{
        Specification&lt;User&gt; spec = <span class="hljs-keyword">new</span> SpecificationsBuilder&lt;User&gt;().buildSpecification(search);
        <span class="hljs-keyword">return</span> repo.findAll(spec);
    }
}
</code></pre>
<p data-nodeid="72925">Controller 里面非常简单，利用 SpecificationsBuilder 生成我们需要的 Specification 即可。那么我们看看 SpecificationsBuilder 里面是怎么写的。</p>
<pre class="lang-java" data-nodeid="72926"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.spe;
<span class="hljs-keyword">import</span> com.example.jpa.example1.User;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.domain.Specification;
<span class="hljs-keyword">import</span> java.util.ArrayList;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-keyword">import</span> java.util.regex.Matcher;
<span class="hljs-keyword">import</span> java.util.regex.Pattern;
<span class="hljs-keyword">import</span> java.util.stream.Collectors;
<span class="hljs-comment">/**
 * 处理请求参数
 * <span class="hljs-doctag">@param</span> &lt;Entity&gt;
 */</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpecificationsBuilder</span>&lt;<span class="hljs-title">Entity</span>&gt; </span>{
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;SearchCriteria&gt; params;

   <span class="hljs-comment">//初始化params，保证每次实例都是一个新的ArrayList</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">SpecificationsBuilder</span><span class="hljs-params">()</span> </span>{
      params = <span class="hljs-keyword">new</span> ArrayList&lt;SearchCriteria&gt;();
   }

   <span class="hljs-comment">//利用正则表达式取我们search参数里面的值，解析成SearchCriteria对象</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> Specification&lt;Entity&gt; <span class="hljs-title">buildSpecification</span><span class="hljs-params">(String search)</span> </span>{
      Pattern pattern = Pattern.compile(<span class="hljs-string">"(\\w+?)(:|&lt;|&gt;)(\\w+?),"</span>);
      Matcher matcher = pattern.matcher(search + <span class="hljs-string">","</span>);
      <span class="hljs-keyword">while</span> (matcher.find()) {
         <span class="hljs-keyword">this</span>.with(matcher.group(<span class="hljs-number">1</span>), Operator.fromOperator(matcher.group(<span class="hljs-number">2</span>)), matcher.group(<span class="hljs-number">3</span>));
      }
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.build();
   }
   <span class="hljs-comment">//根据参数返回我们刚才创建的SearchCriteria</span>
   <span class="hljs-function"><span class="hljs-keyword">private</span> SpecificationsBuilder <span class="hljs-title">with</span><span class="hljs-params">(String key, Operator operation, Object value)</span> </span>{
      params.add(<span class="hljs-keyword">new</span> SearchCriteria(key, operation, value));
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>;
   }
   <span class="hljs-comment">//根据我们刚才创建的MySpecification返回所需要的Specification</span>
   <span class="hljs-function"><span class="hljs-keyword">private</span> Specification&lt;Entity&gt; <span class="hljs-title">build</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">if</span> (params.size() == <span class="hljs-number">0</span>) {
         <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
      }
      List&lt;Specification&gt; specs = params.stream()
            .map(MySpecification&lt;User&gt;::<span class="hljs-keyword">new</span>)
            .collect(Collectors.toList());
      Specification result = specs.get(<span class="hljs-number">0</span>);
      <span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">1</span>; i &lt; params.size(); i++) {
         result = Specification.where(result)
               .and(specs.get(i));
      }
      <span class="hljs-keyword">return</span> result;
   }
}
</code></pre>
<p data-nodeid="72927">通过上面的代码，我们可以看到通过自定义的 SpecificationsBuilder，来处理请求参数 search 里面的值，然后转化成我们上面写的 SearchCriteria 对象，再调用 MySpecification 生成我们需要的 Specification，从而利用 JpaSpecificationExecutor 实现查询效果。是不是学起来并不困难了，你学会了吗？</p>
<h3 data-nodeid="72928">总结</h3>
<p data-nodeid="72929">本课时，我们通过实例学习了 JpaSpecificationExecutor 的用法，并且通过源码了解了 JpaSpecificationExecutor 的实现原理，最后我举了一个实战场景的例子，使我们可以利用 Spring Data JPA and Specifications 很轻松地创建一个基于 Search 的 Rest API。虽然我介绍的这个例子还有很多可以扩展的地方，但是希望你根据实际情况再进行相应的扩展。</p>
<p data-nodeid="72930">这里顺带留一个思考题：怎么查询 UserAddress？提示你可以利用我上面提到的 SpecificationsBuilder 进行解决。</p>
<p data-nodeid="72931">本课时到这就结束了，欢迎你在下面留言讨论或分享。下节课，我会为你讲解如何自定义 Repository。</p>
<blockquote data-nodeid="72932">
<p data-nodeid="72933" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="73078">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *选：
> 老师，MySpecification中的toPredicate方法如果是不同数值类型比如布尔类型，date类型怎么办？除了写if判断之外有别的比较好的方法吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以利用java的反射机制，switch ；或者利用枚举，做不同的映射。

##### **7519：
> 老师您好啊，请教一下：行转列的需求SpringDataJPA有没有解决方案呢，是不是一定要存储过程？😁

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; HashMap

##### *良：
> 使用JpaSpecificationExecutor">动态方式查询，会比普通的findBy*之类的方法慢吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会慢，可以压测看看

##### July：
> 请问 JpaSpecificationExecutor 如何实现分页查询但不返回总数？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; JpaSpecificationExecutor目前不支持

