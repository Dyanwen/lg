<p data-nodeid="1045" class="">上一课时，我们着重讲了方法名和参数的使用方法，这一课时我们来看下Repository 支持的返回结果有哪些，以及 DTO 类型的返回结果如何自定义，及其在实际工作场景中我们如何做。通过本课时的学习，你将了解到 Repository 的几种返回结果，以及如何返回 DTO。我们先看一下返回结果有哪些。</p>
<h3 data-nodeid="1046">Repository 的返回结果有哪些？</h3>
<p data-nodeid="1047">我们之前已经介绍过了 Repository 的接口，那么现在来看一下这些接口支持的返回结果有哪些，如下图所示：</p>
<p data-nodeid="1048"><img src="https://s0.lgstatic.com/i/image/M00/56/1A/Ciqc1F9rCruAArlKAANX2obUD_A764.png" alt="Drawing 0.png" data-nodeid="1202"></p>
<p data-nodeid="1049">打开 SimpleJpaRepository 直接看它的 Structure 就可以知道，它实现的方法，以及父类接口的方法和返回类型包括：Optional、Iterable、List、Page、Long、Boolean、Entity 对象等，而实际上支持的返回类型还要多一些。</p>
<p data-nodeid="1050">由于 Repository 里面支持 Iterable，所以其实 java 标准的 List、Set 都可以作为返回结果，并且也会支持其子类，Spring Data 里面定义了一个特殊的子类 Steamable，Streamable 可以替代 Iterable 或任何集合类型。它还提供了方便的方法来访问 Stream，可以直接在元素上进行 ….filter(…) 和 ….map(…) 操作，并将 Streamable 连接到其他元素。我们看个关于 UserRepository 直接继承 JpaRepository 的例子。</p>
<pre class="lang-java" data-nodeid="1051"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="1052">还用之前的 UserRepository 类，在测试类里面做如下调用：</p>
<pre class="lang-java" data-nodeid="1053"><code data-language="java">User user = userRepository.save(User.builder().name(<span class="hljs-string">"jackxx"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
Assert.assertNotNull(user);
Streamable&lt;User&gt; userStreamable = userRepository.findAll(PageRequest.of(<span class="hljs-number">0</span>,<span class="hljs-number">10</span>)).and(User.builder().name(<span class="hljs-string">"jack222"</span>).build());
userStreamable.forEach(System.out::println);
</code></pre>
<p data-nodeid="1054">然后我们就会得到如下输出：</p>
<pre class="lang-java" data-nodeid="1055"><code data-language="java">User(id=<span class="hljs-number">1</span>, name=jackxx, email=<span class="hljs-number">123456</span>@<span class="hljs-number">126.</span>com, sex=man, address=shanghai)
User(id=<span class="hljs-keyword">null</span>, name=jack222, email=<span class="hljs-keyword">null</span>, sex=<span class="hljs-keyword">null</span>, address=<span class="hljs-keyword">null</span>)
</code></pre>
<p data-nodeid="1056">这个例子 Streamable<code data-backticks="1" data-nodeid="1208">&lt;User&gt;</code> userStreamable，实现了 Streamable 的返回结果，如果想自定义方法，可以进行如下操作。</p>
<h4 data-nodeid="1057">自定义 Streamable</h4>
<p data-nodeid="1058">官方给我们提供了自定义 Streamable 的方法，不过在实际工作中很少出现要自定义保证结果类的情况，在这里我简单介绍一下方法，看如下例子：</p>
<pre class="lang-java" data-nodeid="1059"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Product</span> </span>{ (<span class="hljs-number">1</span>)
  <span class="hljs-function">MonetaryAmount <span class="hljs-title">getPrice</span><span class="hljs-params">()</span> </span>{ … }
}
<span class="hljs-meta">@RequiredArgConstructor(staticName = "of")</span>
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Products</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Streamable</span>&lt;<span class="hljs-title">Product</span>&gt; </span>{ (<span class="hljs-number">2</span>)
  <span class="hljs-keyword">private</span> Streamable&lt;Product&gt; streamable;
  <span class="hljs-function"><span class="hljs-keyword">public</span> MonetaryAmount <span class="hljs-title">getTotal</span><span class="hljs-params">()</span> </span>{ (<span class="hljs-number">3</span>)
    <span class="hljs-keyword">return</span> streamable.stream() <span class="hljs-comment">//</span>
      .map(Priced::getPrice)
      .reduce(Money.of(<span class="hljs-number">0</span>), MonetaryAmount::add);
  }
}
<span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">ProductRepository</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Repository</span>&lt;<span class="hljs-title">Product</span>, <span class="hljs-title">Long</span>&gt; </span>{
  <span class="hljs-function">Products <span class="hljs-title">findAllByDescriptionContaining</span><span class="hljs-params">(String text)</span></span>; (<span class="hljs-number">4</span>)
}
</code></pre>
<p data-nodeid="1060">以上四个步骤介绍了自定义 Streamable 的方法，分别为：</p>
<p data-nodeid="1061">（1）Product 实体，公开 API 以访问产品价格。</p>
<p data-nodeid="1062">（2）Streamable<code data-backticks="1" data-nodeid="1215">&lt;Product&gt;</code> 的包装类型可以通过 Products.of(…) 构造（通过 Lombok 注解创建的工厂方法）。</p>
<p data-nodeid="1063">（3）包装器类型在 Streamable<code data-backticks="1" data-nodeid="1218">&lt;Product&gt;</code> 上公开了计算新值的其他 API。</p>
<p data-nodeid="1064">（4）可以将包装器类型直接用作查询方法返回类型。无须返回 Stremable<code data-backticks="1" data-nodeid="1221">&lt;Product&gt;</code> 并将其手动包装在存储库 Client 端中。</p>
<p data-nodeid="1065">通过以上例子你就可以做到自定义 Streamable，其原理很简单，就是实现Streamable接口，自己定义自己的实现类即可。我们也可以看下源码 QueryExecutionResultHandler 里面是否有 Streamable 子类的判断，来支持自定义 Streamable，关键源码如下：</p>
<p data-nodeid="1066"><img src="https://s0.lgstatic.com/i/image/M00/56/1A/Ciqc1F9rCteAD_ysAADP7mUlfak673.png" alt="Drawing 1.png" data-nodeid="1226"></p>
<p data-nodeid="1067">通过源码你会发现 Streamable 为什么生效，下面来看看常见的集合类的返回实现。</p>
<h4 data-nodeid="1068">返回结果类型 List/Stream/Page/Slice</h4>
<p data-nodeid="1069">在实际开发中，我们如何返回 List/Stream/Page/Slice 呢？</p>
<p data-nodeid="1070">首先，新建我们的 UserRepository：</p>
<pre class="lang-java" data-nodeid="1071"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Pageable;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.Query;
<span class="hljs-keyword">import</span> java.util.stream.Stream;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
   <span class="hljs-comment">//自定义一个查询方法，返回Stream对象，并且有分页属性</span>
    <span class="hljs-meta">@Query("select u from User u")</span>
    <span class="hljs-function">Stream&lt;User&gt; <span class="hljs-title">findAllByCustomQueryAndStream</span><span class="hljs-params">(Pageable pageable)</span></span>;
    <span class="hljs-comment">//测试Slice的返回结果</span>
    <span class="hljs-meta">@Query("select u from User u")</span>
    <span class="hljs-function">Slice&lt;User&gt; <span class="hljs-title">findAllByCustomQueryAndSlice</span><span class="hljs-params">(Pageable pageable)</span></span>;
}
</code></pre>
<p data-nodeid="1072">然后，修改一下我们的测试用例类，如下，验证一下结果：</p>
<pre class="lang-java" data-nodeid="1073"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> com.fasterxml.jackson.core.JsonProcessingException;
<span class="hljs-keyword">import</span> com.fasterxml.jackson.databind.ObjectMapper;
<span class="hljs-keyword">import</span> org.assertj.core.util.Lists;
<span class="hljs-keyword">import</span> org.junit.Assert;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Page;
<span class="hljs-keyword">import</span> org.springframework.data.domain.PageRequest;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Slice;
<span class="hljs-keyword">import</span> org.springframework.data.util.Streamable;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-keyword">import</span> java.util.stream.Stream;
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRepositoryTest</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository userRepository;
    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testSaveUser</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> JsonProcessingException </span>{
        <span class="hljs-comment">//我们新增7条数据方便测试分页结果</span>
        userRepository.save(User.builder().name(<span class="hljs-string">"jack1"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack2"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack3"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack4"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack5"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack6"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        userRepository.save(User.builder().name(<span class="hljs-string">"jack7"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        <span class="hljs-comment">//我们利用ObjectMapper将我们的返回结果Json to String</span>
        ObjectMapper objectMapper = <span class="hljs-keyword">new</span> ObjectMapper();
        <span class="hljs-comment">//返回Stream类型结果（1）</span>
        Stream&lt;User&gt; userStream = userRepository.findAllByCustomQueryAndStream(PageRequest.of(<span class="hljs-number">1</span>,<span class="hljs-number">3</span>));
        userStream.forEach(System.out::println);
        <span class="hljs-comment">//返回分页数据（2）</span>
        Page&lt;User&gt; userPage = userRepository.findAll(PageRequest.of(<span class="hljs-number">0</span>,<span class="hljs-number">3</span>));
        System.out.println(objectMapper.writeValueAsString(userPage));
        <span class="hljs-comment">//返回Slice结果（3）</span>
        Slice&lt;User&gt; userSlice = userRepository.findAllByCustomQueryAndSlice(PageRequest.of(<span class="hljs-number">0</span>,<span class="hljs-number">3</span>));
        System.out.println(objectMapper.writeValueAsString(userSlice));
        <span class="hljs-comment">//返回List结果（4）</span>
        List&lt;User&gt; userList = userRepository.findAllById(Lists.newArrayList(<span class="hljs-number">1L</span>,<span class="hljs-number">2L</span>));
        System.out.println(objectMapper.writeValueAsString(userList));
    }
}
</code></pre>
<p data-nodeid="1074">这个时候我们分别看下四种测试结果：<br>
<strong data-nodeid="1242">第一种：通过</strong>Stream<code data-backticks="1" data-nodeid="1238">&lt;User&gt;</code><strong data-nodeid="1243">取第二页的数据，得到结果如下：</strong></p>
<p data-nodeid="1075">User(id=4, name=jack4, email=123456@126.com, sex=man, address=shanghai)</p>
<p data-nodeid="1076">User(id=5, name=jack5, email=123456@126.com, sex=man, address=shanghai)</p>
<p data-nodeid="1077">User(id=6, name=jack6, email=123456@126.com, sex=man, address=shanghai)</p>
<p data-nodeid="2096" class="">Spring Data 的支持可以通过使用 Java 8 Stream 作为返回类型来逐步处理查询方法的结果。<strong data-nodeid="2102">需要注意的是：流的关闭问题</strong>，try catch 是一种常用的关闭方法，如下所示：</p>


<pre class="lang-java" data-nodeid="1079"><code data-language="java">Stream&lt;User&gt; stream;
<span class="hljs-keyword">try</span> {
   stream = repository.findAllByCustomQueryAndStream()
   stream.forEach(…);
} <span class="hljs-keyword">catch</span> (Exception e) {
   e.printStackTrace();
} <span class="hljs-keyword">finally</span> {
   <span class="hljs-keyword">if</span> (stream!=<span class="hljs-keyword">null</span>){
      stream.close();
   }
}
</code></pre>
<p data-nodeid="1080"><strong data-nodeid="1258">第二种：返回 Page<code data-backticks="1" data-nodeid="1255">&lt;User&gt;</code> 的分页数据结果，如下所示：</strong></p>
<pre class="lang-java" data-nodeid="1081"><code data-language="java">{
&nbsp; &nbsp;<span class="hljs-string">"content"</span>:[
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack1"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack2"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">3</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack3"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp;],
&nbsp; &nbsp;<span class="hljs-string">"pageable"</span>:{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sort"</span>:{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sorted"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"unsorted"</span>:<span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"empty"</span>:<span class="hljs-keyword">true</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"pageNumber"</span>:<span class="hljs-number">0</span>,当前页码
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"pageSize"</span>:<span class="hljs-number">3</span>,页码大小
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"offset"</span>:<span class="hljs-number">0</span>,偏移量
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"paged"</span>:<span class="hljs-keyword">true</span>,是否分页了
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unpaged"</span>:<span class="hljs-keyword">false</span>
&nbsp; &nbsp;},
&nbsp; &nbsp;<span class="hljs-string">"totalPages"</span>:<span class="hljs-number">3</span>,一共有多少页
&nbsp; &nbsp;<span class="hljs-string">"last"</span>:<span class="hljs-keyword">false</span>,是否是到最后
&nbsp; &nbsp;<span class="hljs-string">"totalElements"</span>:<span class="hljs-number">7</span>,一共多少调数
&nbsp; &nbsp;<span class="hljs-string">"numberOfElements"</span>:<span class="hljs-number">3</span>,当前数据下标
&nbsp; &nbsp;<span class="hljs-string">"sort"</span>:{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sorted"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>:<span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"empty"</span>:<span class="hljs-keyword">true</span>
&nbsp; &nbsp;},
&nbsp; &nbsp;<span class="hljs-string">"size"</span>:<span class="hljs-number">3</span>,当前content大小
&nbsp; &nbsp;<span class="hljs-string">"number"</span>:<span class="hljs-number">0</span>,当前页面码的索引
&nbsp; &nbsp;<span class="hljs-string">"first"</span>:<span class="hljs-keyword">true</span>,是否是第一页
&nbsp; &nbsp;<span class="hljs-string">"empty"</span>:<span class="hljs-keyword">false</span>是否有数据
}
</code></pre>
<p data-nodeid="1082">这里我们可以看到 Page<code data-backticks="1" data-nodeid="1260">&lt;User&gt;</code> 返回了第一个页的数据，并且告诉我们一共有三个部分的数据：</p>
<ul data-nodeid="1083">
<li data-nodeid="1084">
<p data-nodeid="1085"><strong data-nodeid="1266">content</strong>：数据的内容，现在指 User 的 List 3 条。</p>
</li>
<li data-nodeid="1086">
<p data-nodeid="1087"><strong data-nodeid="1271">pageable</strong>：分页数据，包括排序字段是什么及其方向、当前是第几页、一共多少页、是否是最后一条等。</p>
</li>
<li data-nodeid="1088">
<p data-nodeid="1089"><strong data-nodeid="1276">当前数据的描述</strong>：“size”：3，当前 content 大小；“number”：0，当前页面码的索引；&nbsp; “first”：true，是否是第一页；“empty”：false，是否没有数据。</p>
</li>
</ul>
<p data-nodeid="1090">通过这三部分数据我们可以知道要查数的分页信息。我们接着看第三种测试结果。</p>
<p data-nodeid="1091"><strong data-nodeid="1283">第三种：返回 Slice<code data-backticks="1" data-nodeid="1280">&lt;User&gt;</code> 结果，如下所示：</strong></p>
<pre class="lang-java" data-nodeid="1092"><code data-language="java">{
&nbsp; &nbsp;<span class="hljs-string">"content"</span>:[
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">4</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack4"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">5</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack5"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"id"</span>:<span class="hljs-number">6</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"name"</span>:<span class="hljs-string">"jack6"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp;],
&nbsp; &nbsp;<span class="hljs-string">"pageable"</span>:{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sort"</span>:{
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"sorted"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"unsorted"</span>:<span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;<span class="hljs-string">"empty"</span>:<span class="hljs-keyword">true</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"pageNumber"</span>:<span class="hljs-number">1</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"pageSize"</span>:<span class="hljs-number">3</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"offset"</span>:<span class="hljs-number">3</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"paged"</span>:<span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unpaged"</span>:<span class="hljs-keyword">false</span>
&nbsp; &nbsp;},
&nbsp; &nbsp;<span class="hljs-string">"numberOfElements"</span>:<span class="hljs-number">3</span>,
&nbsp; &nbsp;<span class="hljs-string">"sort"</span>:{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sorted"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"unsorted"</span>:<span class="hljs-keyword">true</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"empty"</span>:<span class="hljs-keyword">true</span>
&nbsp; &nbsp;},
&nbsp; &nbsp;<span class="hljs-string">"size"</span>:<span class="hljs-number">3</span>,
&nbsp; &nbsp;<span class="hljs-string">"number"</span>:<span class="hljs-number">1</span>,
&nbsp; &nbsp;<span class="hljs-string">"first"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp;<span class="hljs-string">"last"</span>:<span class="hljs-keyword">false</span>,
&nbsp; &nbsp;<span class="hljs-string">"empty"</span>:<span class="hljs-keyword">false</span>
}
</code></pre>
<p data-nodeid="1093">这时我们发现上面的 Page 返回结果少了，那么一共有多少条结果、多少页的数据呢？我们再比较一下第二种和第三种测试结果的执行 SQL：</p>
<p data-nodeid="1094">第二种执行的是普通的分页查询 SQL：</p>
<pre class="lang-java" data-nodeid="1095"><code data-language="java">查询分页数据
Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.email as email3_0_, user0_.name as name4_0_, user0_.sex as sex5_0_ from user user0_ limit ?
计算分页数据
Hibernate: <span class="hljs-function">select <span class="hljs-title">count</span><span class="hljs-params">(user0_.id)</span> as col_0_0_ from user user0_
</span></code></pre>
<p data-nodeid="1096">第三种执行的 SQL 如下：</p>
<pre class="lang-java" data-nodeid="1097"><code data-language="java">Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.email as email3_0_, user0_.name as name4_0_, user0_.sex as sex5_0_ from user user0_ limit ? offset ?
</code></pre>
<p data-nodeid="1098">通过对比可以看出，只查询偏移量，不计算分页数据，这就是 Page 和 Slice 的主要区别。我们接着看第四种测试结果。</p>
<p data-nodeid="1099"><strong data-nodeid="1293">第四种：返回 List<code data-backticks="1" data-nodeid="1290">&lt;User&gt;</code> 结果如下：</strong></p>
<pre class="lang-java" data-nodeid="1100"><code data-language="java">[
&nbsp; &nbsp;{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>:<span class="hljs-string">"jack1"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp;},
&nbsp; &nbsp;{
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>:<span class="hljs-string">"jack2"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"email"</span>:<span class="hljs-string">"123456@126.com"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"sex"</span>:<span class="hljs-string">"man"</span>,
&nbsp; &nbsp; &nbsp; <span class="hljs-string">"address"</span>:<span class="hljs-string">"shanghai"</span>
&nbsp; &nbsp;}
]
</code></pre>
<p data-nodeid="1101">到这里，我们可以很简单地查询出来 ID=1 和 ID=2 的数据，没有分页信息。</p>
<p data-nodeid="1102">上面四种方法介绍了常见的多条数据返回结果的形式，单条的我就不多介绍了，相信你一看就懂，无非就是对 JDK8 的 Optional 的支持。比如支持了 Null 的优雅判断，再一个就是支持直接返回 Entity，或者一些存在 / 不存在的 Boolean 的结果和一些 count 条数的返回结果而已。</p>
<p data-nodeid="1103">我们接下来看下 Repository 的方法是如何对异步进行支持的？</p>
<h4 data-nodeid="1104">Repository 对 Feature/CompletableFuture 异步返回结果的支持：</h4>
<p data-nodeid="1105">我们可以使用 Spring 的异步方法执行Repository查询，这意味着方法将在调用时立即返回，并且实际的查询执行将发生在已提交给 Spring TaskExecutor 的任务中，比较适合定时任务的实际场景。异步使用起来比较简单，直接加@Async 注解即可，如下所示：</p>
<pre class="lang-java" data-nodeid="1106"><code data-language="java"><span class="hljs-meta">@Async</span>
<span class="hljs-function">Future&lt;User&gt; <span class="hljs-title">findByFirstname</span><span class="hljs-params">(String firstname)</span></span>; (<span class="hljs-number">1</span>)
<span class="hljs-meta">@Async</span>
<span class="hljs-function">CompletableFuture&lt;User&gt; <span class="hljs-title">findOneByFirstname</span><span class="hljs-params">(String firstname)</span></span>; (<span class="hljs-number">2</span>)
<span class="hljs-meta">@Async</span>
<span class="hljs-function">ListenableFuture&lt;User&gt; <span class="hljs-title">findOneByLastname</span><span class="hljs-params">(String lastname)</span></span>;(<span class="hljs-number">3</span>)
</code></pre>
<p data-nodeid="1107">上述三个异步方法的返回结果，分别做如下解释：</p>
<ul data-nodeid="1108">
<li data-nodeid="1109">
<p data-nodeid="1110">第一处：使用 java.util.concurrent.Future 的返回类型；</p>
</li>
<li data-nodeid="1111">
<p data-nodeid="1112">第二处：使用 java.util.concurrent.CompletableFuture 作为返回类型；</p>
</li>
<li data-nodeid="1113">
<p data-nodeid="1114">第三处：使用 org.springframework.util.concurrent.ListenableFuture 作为返回类型。</p>
</li>
</ul>
<p data-nodeid="1115">以上是对 @Async 的支持，关于实际使用需要注意以下三点内容：</p>
<ul data-nodeid="1116">
<li data-nodeid="1117">
<p data-nodeid="1118">在实际工作中，直接在 Repository 这一层使用异步方法的场景不多，一般都是把异步注解放在 Service 的方法上面，这样的话，可以有一些额外逻辑，如发短信、发邮件、发消息等配合使用；</p>
</li>
<li data-nodeid="1119">
<p data-nodeid="1120">使用异步的时候一定要配置线程池，这点切记，否则“死”得会很难看；</p>
</li>
<li data-nodeid="1121">
<p data-nodeid="1122">万一失败我们会怎么处理？关于事务是怎么处理的呢？这种需要重点考虑的，我将会在 14 课时（乐观锁机制和重试机制在实战中应该怎么用?）中详细介绍。</p>
</li>
</ul>
<p data-nodeid="1123">接下来看看 Repository 对Reactive 是如何支持的。</p>
<h4 data-nodeid="1124">对 Reactive 支持 flux 与 Mono</h4>
<p data-nodeid="1125">可能有同学会问，看到Spring Data Common里面对React还是有支持的，那为什么在JpaRespository里面没看到有响应的返回结果支持呢？其实Common里面提供的只是接口，而JPA里面没有做相关的Reactive 的实现，但是本身Spring Data Common里面对 Reactive 是支持的。</p>
<p data-nodeid="1126">下面我们在 gradle 里面引用一个Spring Data Common的子模块implementation 'org.springframework.boot:spring-boot-starter-data-mongodb' 来加载依赖，这时候我们打开 Repository 看 Hierarchy 就可以看到，这里多了一个 Mongo 的 Repsitory 的实现，天然地支持着 Reactive 这条线。</p>
<p data-nodeid="1127"><img src="https://s0.lgstatic.com/i/image/M00/56/1C/Ciqc1F9rC-uAWLY5AASQJuG5If0280.png" alt="Drawing 2.png" data-nodeid="1317"></p>
<p data-nodeid="1128">相信到这里你能感受到 Spring Data Common 的强大支持，对 Repository 接口的不同实现也有了一定的认识。对于以上讲述的返回结果，你可以自己测试一下加以理解并运用，那么接下来我们进行一个总结。</p>
<h4 data-nodeid="1129">返回结果支持总结</h4>
<p data-nodeid="1130">下面打开 ResultProcessor 类的源码看一下支持的类型有哪些。</p>
<p data-nodeid="1131"><img src="https://s0.lgstatic.com/i/image/M00/56/27/CgqCHl9rC7uAOiNSAAGT0qXVLyY891.png" alt="Drawing 3.png" data-nodeid="1323"></p>
<p data-nodeid="1132">从上图可以看出 processResult 的时候分别对 PageQuery、Stream、Reactiv 有了各自的判断，我们 debug 到这里的时候来看一下 convert，进入到类里面。</p>
<p data-nodeid="1133"><img src="https://s0.lgstatic.com/i/image/M00/56/1C/Ciqc1F9rC_-AOhtnAALvuoaT4mw230.png" alt="Drawing 4.png" data-nodeid="1327"></p>
<p data-nodeid="1134">可以看到 QueryExecutorConverters 里面对 JDK8、Guava、vavr 也做了各种支持，如果你有兴趣可以课后去仔细看看源码。</p>
<p data-nodeid="1135">这里我们先用表格总结一下返回值，下表列出了 Spring Data JPA Query Method 机制支持的方法的返回值类型：</p>
<p data-nodeid="1136"><img src="https://s0.lgstatic.com/i/image/M00/56/1D/Ciqc1F9rDAiARh9tAAQVFWlht1s532.png" alt="Drawing 5.png" data-nodeid="1332"></p>
<p data-nodeid="1137">以上是对返回的类型做的总结，接下来进入本课时的第二部分，来看看工作中最常见的、同一个 Entity 的不同字段的返回形式有哪些。</p>
<h3 data-nodeid="1138">最常见的 DTO 返回结果的支持方法有哪些？</h3>
<p data-nodeid="1139">上面我们讲解了 Repository 不同的返回类型，下面我们着重说一下除了 Entity，还能返回哪些 POJO 呢？我们先了解一个概念：Projections。</p>
<h4 data-nodeid="1140">Projections 的概念</h4>
<p data-nodeid="1141">Spring JPA 对 Projections 扩展的支持，我个人觉得这是个非常好的东西，从字面意思上理解就是映射，指的是和 DB 的查询结果的字段映射关系。一般情况下，返回的字段和 DB 的查询结果的字段是一一对应的；但有的时候，需要返回一些指定的字段，或者返回一些复合型的字段，而不需要全部返回。</p>
<p data-nodeid="1142">原来我们的做法是自己写各种 entity 到 view 的各种 convert 的转化逻辑，而 Spring Data 正是考虑到了这一点，允许对专用返回类型进行建模，有选择地返回同一个实体的不同视图对象。</p>
<p data-nodeid="1143">下面还以我们的 User 查询对象为例，看看怎么自定义返回 DTO：</p>
<pre class="lang-java" data-nodeid="1144"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> String email;
   <span class="hljs-keyword">private</span> String sex;
   <span class="hljs-keyword">private</span> String address;
}
</code></pre>
<p data-nodeid="1145">看上面的原始 User 实体代码，如果我们只想返回 User 对象里面的 name 和 email，应该怎么做？下面我们介绍三种方法。</p>
<h4 data-nodeid="1146"><strong data-nodeid="1344">第一种方法：新建一张表的不同 Entity</strong></h4>
<p data-nodeid="1147">首先，我们新增一个Entity类：通过 @Table 指向同一张表，这张表和 User 实例里面的表一样都是 user，完整内容如下：</p>
<pre class="lang-java" data-nodeid="1148"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table(name = "user")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserOnlyNameEmailEntity</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> String email;
}
</code></pre>
<p data-nodeid="1149">然后，新增一个 UserOnlyNameEmailEntityRepository，做单独的查询：</p>
<pre class="lang-java" data-nodeid="1150"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserOnlyNameEmailEntityRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">UserOnlyNameEmailEntity</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="1151">最后，我们的测试用例里面的写法如下：</p>
<pre class="lang-java" data-nodeid="1152"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testProjections</span><span class="hljs-params">()</span> </span>{
  userRepository.save(User.builder().id(<span class="hljs-number">1L</span>).name(<span class="hljs-string">"jack12"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
    List&lt;User&gt; users= userRepository.findAll();
    System.out.println(users);
    UserOnlyNameEmailEntity uName = userOnlyNameEmailEntityRepository.getOne(<span class="hljs-number">1L</span>);
    System.out.println(uName);
}
</code></pre>
<p data-nodeid="1153">我们看一下输出结果：</p>
<pre class="lang-java" data-nodeid="1154"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">user</span> <span class="hljs-params">(address, email, name, sex, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?, ?, ?)</span>
Hibernate: select user0_.id as id1_0_, user0_.address as address2_0_, user0_.email as email3_0_, user0_.name as name4_0_, user0_.sex as sex5_0_ from user user0_
[<span class="hljs-title">User</span><span class="hljs-params">(id=<span class="hljs-number">1</span>, name=jack12, email=<span class="hljs-number">123456</span>@<span class="hljs-number">126.</span>com, sex=man, address=shanghai)</span>]
Hibernate: select useronlyna0_.id as id1_0_0_, useronlyna0_.email as email3_0_0_, useronlyna0_.name as name4_0_0_ from user useronlyna0_ where useronlyna0_.id</span>=?
UserOnlyNameEmailEntity(id=<span class="hljs-number">1</span>, name=jack12, email=<span class="hljs-number">123456</span>@<span class="hljs-number">126.</span>com)
</code></pre>
<p data-nodeid="1155">上述结果可以看到，当在 user 表里面插入了一条数据，而 userRepository 和 userOnlyNameEmailEntityRepository 查询的都是同一张表 user，这种方式的好处是简单、方便，很容易可以想到；缺点就是通过两个实体都可以进行 update 操作，如果同一个项目里面这种实体比较多，到时候就容易不知道是谁更新的，从而导致出 bug 不好查询，实体职责划分不明确。我们来看第二种返回 DTO 的做法。</p>
<h4 data-nodeid="1156">第二种方法：直接定义一个 UserOnlyNameEmailDto</h4>
<p data-nodeid="1157">首先，我们新建一个 DTO 类来返回我们想要的字段，它是 UserOnlyNameEmailDto，用来接收 name、email 两个字段的值，具体如下：</p>
<pre class="lang-java" data-nodeid="1158"><code data-language="java"><span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserOnlyNameEmailDto</span> </span>{
    <span class="hljs-keyword">private</span> String name,email;
}
</code></pre>
<p data-nodeid="1159">其次，在 UserRepository 里面做如下用法：</p>
<pre class="lang-java" data-nodeid="1160"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
    <span class="hljs-comment">//测试只返回name和email的DTO</span>
    <span class="hljs-function">UserOnlyNameEmailDto <span class="hljs-title">findByEmail</span><span class="hljs-params">(String email)</span></span>;
}
</code></pre>
<p data-nodeid="1161">然后，测试用例里面写法如下：</p>
<pre class="lang-java" data-nodeid="1162"><code data-language="java">    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testProjections</span><span class="hljs-params">()</span> </span>{
userRepository.save(User.builder().id(<span class="hljs-number">1L</span>).name(<span class="hljs-string">"jack12"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        UserOnlyNameEmailDto userOnlyNameEmailDto =  userRepository.findByEmail(<span class="hljs-string">"123456@126.com"</span>);
        System.out.println(userOnlyNameEmailDto);
    }
</code></pre>
<p data-nodeid="1163">最后，输出结果如下：</p>
<pre class="lang-java" data-nodeid="1164"><code data-language="java">Hibernate: select user0_.name as col_0_0_, user0_.email as col_1_0_ from user user0_ where user0_.email=?
UserOnlyNameEmailDto(name=jack12, email=<span class="hljs-number">123456</span>@<span class="hljs-number">126.</span>com)
</code></pre>
<p data-nodeid="1165">这里需要注意的是，如果我们去看源码的话，看关键的 PreferredConstructorDiscoverer 类时会发现，UserDTO 里面只能有一个全参数构造方法，如下所示：</p>
<p data-nodeid="1166"><img src="https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDDKAPKAIAASfIaP3unE060.png" alt="Drawing 6.png" data-nodeid="1358"></p>
<p data-nodeid="1167">如上图所示，Constructor 选择的时候会帮我们做构造参数的选择，如果 DTO 里面有多个构造方法，就会报转化错误的异常，这一点需要注意，异常是这样的：</p>
<pre class="lang-java" data-nodeid="1168"><code data-language="java">No converter found capable of converting from type [com.example.jpa.example1.User] to type [com.example.jpa.example1.UserOnlyNameEmailDto
</code></pre>
<p data-nodeid="1169">所以这种方式的优点就是返回的结果不需要是个实体对象，对 DB 不能进行除了查询之外的任何操作；缺点就是有 set 方法还可以改变里面的值，构造方法不能更改，必须全参数，这样如果是不熟悉 JPA 的新人操作的时候很容易引发 Bug。</p>
<h4 data-nodeid="1170">第三种方法：返回结果是一个 POJO 的接口</h4>
<p data-nodeid="1171">我们再来学习一种返回不同字段的方式，这种方式与上面两种的区别是只需要定义接口，它的好处是只读，不需要添加构造方法，我们使用起来非常灵活，一般很难产生 Bug，那么它怎么实现呢？</p>
<p data-nodeid="1172">首先，定义一个 UserOnlyName 的接口：</p>
<pre class="lang-java" data-nodeid="1173"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserOnlyName</span> </span>{
    <span class="hljs-function">String <span class="hljs-title">getName</span><span class="hljs-params">()</span></span>;
    <span class="hljs-function">String <span class="hljs-title">getEmail</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="1174">其次，我们的 UserRepository 写法如下：</p>
<pre class="lang-java" data-nodeid="1175"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
    <span class="hljs-comment">/**
     * 接口的方式返回DTO
     * <span class="hljs-doctag">@param</span> address
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-function">UserOnlyName <span class="hljs-title">findByAddress</span><span class="hljs-params">(String address)</span></span>;
}
</code></pre>
<p data-nodeid="1176">然后，测试用例的写法如下：</p>
<pre class="lang-java" data-nodeid="1177"><code data-language="java">    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testProjections</span><span class="hljs-params">()</span> </span>{
userRepository.save(User.builder().name(<span class="hljs-string">"jack12"</span>).email(<span class="hljs-string">"123456@126.com"</span>).sex(<span class="hljs-string">"man"</span>).address(<span class="hljs-string">"shanghai"</span>).build());
        UserOnlyName userOnlyName = userRepository.findByAddress(<span class="hljs-string">"shanghai"</span>);
        System.out.println(userOnlyName);
    }
</code></pre>
<p data-nodeid="1178">最后，我们的运行结果如下：</p>
<pre class="lang-java" data-nodeid="1179"><code data-language="java">Hibernate: select user0_.name as col_0_0_, user0_.email as col_1_0_ from user user0_ where user0_.address=?
org.springframework.data.jpa.repository.query.AbstractJpaQuery$TupleConverter$TupleBackedMap@<span class="hljs-number">1d</span>369521
</code></pre>
<p data-nodeid="1180">这个时候会发现我们的 userOnlyName 接口成了一个代理对象，里面通过 Map 的格式包含了我们的要返回字段的值（如：name、email），我们用的时候直接调用接口里面的方法即可，如 userOnlyName.getName() 即可；这种方式的优点是接口为只读，并且语义更清晰，所以这种是我比较推荐的做法。</p>
<p data-nodeid="1181">其中源码是如何实现的，我来说一个类，你可以通过 debug，看一下最终 DTO 和接口转化执行的 query 有什么不同，看下图中老师 debug 显示的 Query 语句的位置：</p>
<p data-nodeid="1182"><img src="https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDD6AWKs9AAE8kGFOxmo130.png" alt="Drawing 7.png" data-nodeid="1371"></p>
<p data-nodeid="1183">图一：是返回 DTO 接口形式的 query 生成的 JPQL。</p>
<p data-nodeid="1184"><img src="https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDEWAdoxmAARwd1XSUzo704.png" alt="Drawing 8.png" data-nodeid="1375"></p>
<p data-nodeid="1185">图二：是返回 DTO 类的时候 QueryStructure 生成的 JPQL 语句。</p>
<p data-nodeid="1186">两种最大的区别是 DTO 类需要构造方法 new 一个对象出来，这就是我们第二种方法里面需要注意的 DTO 构造函数的问题；而通过图一我们可以看到接口直接通过 as 别名，映射成 hashmap 即可，非常灵活。这里我顺带给你提一个 tips。</p>
<h4 data-nodeid="1187">这里说一个小技巧</h4>
<p data-nodeid="1188">当我们去写userRepositor 的定义方法的时候，IDA 会为我们提供满足 JPA 语法的提示，这也是用 Spring Data JPA 的好处之一，因为这些一旦约定死了（这里是指遵守 JPA 协议），周边的工具会越来越成熟，其中 MyBatis 太灵活了，就会导致周边的工具没办法跟上。创建 defining query method 的时候就会提示，如下图所示：</p>
<p data-nodeid="1189"><img src="https://s0.lgstatic.com/i/image/M00/56/28/CgqCHl9rDE-AYIBqAADX_Pwf0QE948.png" alt="Drawing 9.png" data-nodeid="1382"></p>
<p data-nodeid="1190">以上就是返回 DTO 的几种常见的方法了，你在实际应用时，要不断 debug 和仔细体会。当然除了这些外，还有 @Query 注解也是可以做到，下一节会有介绍。</p>
<h3 data-nodeid="1191">总结</h3>
<p data-nodeid="1192">本课时我为你讲解了返回结果的类型有哪些，也为你重点介绍了返回 DTO 的实战经验和方式，其中返回 DTO 以及第一种方式，我在下一课时“@Query 帮我们解决了什么问题？什么时候应该选择 @Query？”中再详细讲，方便你做实际参考。</p>
<p data-nodeid="1193">实际工作中可能返回结果会比这个更复杂，但是你要掌握学习的“套路”，可以举一反三，学会看源码，就可以轻松应对工作中遇到的任何问题。</p>
<p data-nodeid="1194">你是不是通过老师的课学会了如何利用 Repository 的返回结果解决实际问题了？如果学会了就分享吧，也欢迎你在下方留言，说出自己的观点。</p>
<blockquote data-nodeid="1195">
<p data-nodeid="1196" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="1392">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **坡：
> org.springframework.core.convert.ConverterNotFoundException: No converter found capable of converting from type [com.example.demo.entity.UserBasic] to type [com.example.demo.entity.UserOnlyNameEmailDto]第二种 方法">当前的Repository里面，查询了其他的实体但是为什么老师的可以呢 是不是因为我的配置有问题呢？

##### July：
> Projections功能很棒，用了这么久JPA居然现在才知道这个功能

##### **伟：
> 老师，您好，我在使用第三种方法的同时，使用了@Query注解，导致返回的结果不正确了，有没有解决办法？不使用@Query注解就是正确的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没太理解你的问题，接口返回结果，可以支持@Query，也支持不带@Query注解呀

##### **儿在帝都：
> 平时的生产环境中，我们一般会遇到多表联查，返回的结果集可能是一条，也可能是多条，也可以使用本文中提到的DTO的方式吗？谢谢。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以呀

##### **龙：
> 如果要返回vo前端需要的数据呢？可否直接定义一个对象vo加指定的字段做返回，不需要利用projections机制也可以实现的这种

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不可以的，建议在课程中把projections机制研究透，应该能解决你的问题。vo其实就是DTO，只不过是另外一种POJO而已

##### *良：
> 大佬麻烦问下，使用findAll方法，组装Specification的条件查询，有办法使用DTO座位返回对象吗？因为一般我会使用List">.add("shopStatus", Constant.T).generateSpecification());类似这种的方式进行条件查询，SimpleSpecificationBuilder这个是用来构建Specification查询条件的工具类。这样做的话有没有办法直接返回的是DTO的对象？希望大佬回复下，谢谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 目前没发现有很直接的方法，可以通过06课时讲的多态的方式定义N个DTO实体 去解决。

##### *阳：
> Stream">userInfoRepository.findByUsername("1");返回类型为 Stream的方法为什么必须运行在一个Transaction下？不加@Transaction注解会报错。List">userInfoRepository.findByname("1");而返回List类型的方法则不用包裹在@Transaction下却可以正常运行？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Stream需要依赖事务保证数据的可重复读的特性，而List不需要，List是一个sql查询出来的，利用的DB的autocommit机制

##### *中：
> 用了很久jpa终于知道怎么优雅的接受dto

##### **油：
> 终于知道为什么do转dto报转换异常了

##### **6245：
> 好用,如果返回是一个组合的对象该怎么处理了,两个表组合的信息

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 一样的呀，就是对象接口之间的聚合关系

##### leo：
> 这种流返回相较于直接返回list的优势是什么，是速度更快吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不快，就是stream操作而已

##### leo：
> 这种接口定义的返回值怎么组合嵌套对象

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 直接用对象类型的字段就可以了

##### *晔：
> 想知道jpa有没有类似mybatis的insert Selective或者update Selective

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; @DynamicUpdate可以了解一下，后面章节也会介绍到

##### **刚：
> 用接口作Dto返回实在是让我吃惊。这种方式我觉得是否破坏了面向对象的思想？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 为什么会破坏面向对象的思想？接口也是对象，只是是动态代理

