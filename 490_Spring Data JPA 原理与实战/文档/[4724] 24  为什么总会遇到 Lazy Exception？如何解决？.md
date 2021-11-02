<p data-nodeid="27262">你好，欢迎学习第 24 讲。在我们的实际工作中，经常会遇到 Lazy Exception，所谓的 Lazy Exception 具体一点就是 LazyInitializationException。我经常看到有些同事会遇到这一问题，而他们的处理方式都很复杂，并非最佳实践。那么这一讲，我们就来剖析一下这一概念的原理以及解决方式。</p>


<p data-nodeid="26445">我们先从一个案例入手，看一下什么是 LazyInitializationException。</p>
<h3 data-nodeid="26446">什么是 LazyInitializationException 异常？</h3>
<p data-nodeid="26447">这是一个重现 Lazy 异常的例子，下面我将通过 4 个步骤带你一起看一下什么是 Lazy 异常。</p>
<p data-nodeid="26448">第一步：为了方便我们测试，我们把 spring.jpa.open-in-view 设置成 false，代码如下。</p>
<pre class="lang-java" data-nodeid="26449"><code data-language="java">spring.jpa.open-in-view=<span class="hljs-keyword">false</span>
</code></pre>
<p data-nodeid="26450">第二步：新建一个一对多的关联实体：UserInfo 用户信息，一个用户有多个地址 Address。代码如下所示。</p>
<pre class="lang-java" data-nodeid="26451"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@SuperBuilder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@Table</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-keyword">private</span> String lastName;
   <span class="hljs-keyword">private</span> String emailAddress;
   <span class="hljs-keyword">private</span> String telephone;
   <span class="hljs-comment">//假设一个用户有多个地址，取数据的方式用lazy的模式(默认也是lazy的)；采用CascadeType.PERSIST方便插入演示数据；</span>
   <span class="hljs-meta">@OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.LAZY)</span>
   <span class="hljs-keyword">private</span> List&lt;Address&gt; addressList;
}
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@SuperBuilder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Address</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-keyword">private</span> String city;
   <span class="hljs-comment">//维护关联关系的一方，默认都是lazy模式</span>
   <span class="hljs-meta">@ManyToOne</span>
   <span class="hljs-keyword">private</span> UserInfo userInfo;
}
</code></pre>
<p data-nodeid="26452">第三步：我们再新建一个 Controller，取用户的基本信息，并且查看一下 Address 的地址信息，代码如下。</p>
<pre class="lang-java" data-nodeid="26453"><code data-language="java"><span class="hljs-meta">@GetMapping("/user/info/{id}")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromPath</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> Long id)</span> </span>{
  UserInfo u1 =  userInfoRepository.findById(id).get();
  <span class="hljs-comment">//触发lazy加载，取userInfo里面的地址信息</span>
  System.out.println(u1.getAddressList().get(<span class="hljs-number">0</span>).getCity());
  <span class="hljs-keyword">return</span> u1;
}
</code></pre>
<p data-nodeid="26454">第四步：启动项目，我们直接发起如下请求。</p>
<pre class="lang-java" data-nodeid="26455"><code data-language="java">### get user info的接口
GET /user/info/1 HTTP/1.1
Host: 127.0.0.1:8087
Content-Type: application/json
Cache-Control: no-cache
</code></pre>
<p data-nodeid="26456">然后我们就可以如期得到 Lazy 异常，如下述代码所示。</p>
<pre class="lang-java" data-nodeid="26457"><code data-language="java">org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: com.example.jpa.demo.db.UserInfo.addressList, could not initialize proxy - no Session
   at org.hibernate.collection.internal.AbstractPersistentCollection.throwLazyInitializationException(AbstractPersistentCollection.java:<span class="hljs-number">606</span>) ~[hibernate-core-<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final.jar:<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final]
   at org.hibernate.collection.internal.AbstractPersistentCollection.withTemporarySessionIfNeeded(AbstractPersistentCollection.java:<span class="hljs-number">218</span>) ~[hibernate-core-<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final.jar:<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final]
   at org.hibernate.collection.internal.AbstractPersistentCollection.initialize(AbstractPersistentCollection.java:<span class="hljs-number">585</span>) ~[hibernate-core-<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final.jar:<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final]
   at org.hibernate.collection.internal.AbstractPersistentCollection.read(AbstractPersistentCollection.java:<span class="hljs-number">149</span>) ~[hibernate-core-<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final.jar:<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final]
   at org.hibernate.collection.internal.PersistentBag.get(PersistentBag.java:<span class="hljs-number">561</span>) ~[hibernate-core-<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final.jar:<span class="hljs-number">5.4</span><span class="hljs-number">.20</span>.Final]
   at com.example.jpa.demo.web.UserInfoController.getUserInfoFromPath(UserInfoController.java:<span class="hljs-number">29</span>) ~[main/:na]
</code></pre>
<p data-nodeid="26458">通过上面的异常信息基本可以看到，我们的 UserInfo 实体对象加载 Address 的时候，产生了 Lazy 异常，是因为 no session。那么发生异常的根本原因是什么呢？它的加载原理是什么样的？我们接着分析。</p>
<h3 data-nodeid="26459">Lazy 加载机制的原理分析</h3>
<p data-nodeid="26460">我们都知道 JPA 里有 Lazy 的机制，所谓的 Lazy 就是指，当我们使用关联关系的时候，只有用到被关联关系的一方才会请求数据库去加载数据，也就是说关联关系的真实数据不是立马加载的，只有用到的时候才会加载。</p>
<p data-nodeid="26461">而 Hibernate 的实现机制中提供了 PersistentCollection 的机制，利用代理机制改变了关联关系的集合类型，从而实现了懒加载机制，我们详细看一下。</p>
<h4 data-nodeid="26462">PersistentCollection 集合类</h4>
<p data-nodeid="28340">PersistentCollection 是一个集合类的接口，实现类包括如下几种，如下图所示。</p>
<p data-nodeid="28341" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/11/Ciqc1F_HTamAUlWuAADQNwt-UxI634.png" alt="Drawing 0.png" data-nodeid="28345"></p>


<p data-nodeid="26465">也就是说 Hibernate 通过 PersistentCollection 的实现类 AbstractPersistentCollection 的所有子类，对 JDK 里面提供的 List、Map、SortMap、Array、Set、SortedSet 进行了扩展，从而实现了具有懒加载的特性。所以在 Hibernate 里面支持的关联关系的类型只有下面五种。</p>
<ul data-nodeid="26466">
<li data-nodeid="26467">
<p data-nodeid="26468"><code data-backticks="1" data-nodeid="26580">java.util.List</code></p>
</li>
<li data-nodeid="26469">
<p data-nodeid="26470"><code data-backticks="1" data-nodeid="26581">java.util.Set</code></p>
</li>
<li data-nodeid="26471">
<p data-nodeid="26472"><code data-backticks="1" data-nodeid="26582">java.util.Map</code></p>
</li>
<li data-nodeid="26473">
<p data-nodeid="26474"><code data-backticks="1" data-nodeid="26583">java.util.SortedSet</code></p>
</li>
<li data-nodeid="26475">
<p data-nodeid="26476"><code data-backticks="1" data-nodeid="26584">java.util.SortedMap</code></p>
</li>
</ul>
<p data-nodeid="26477">关于这几种类型，Hibernate 官方也提供了扩展 AbstractPersistentCollection 的方法，不是本讲的重点，我就不多介绍了。</p>
<p data-nodeid="26478">下面我们以 PersistentBag 为例，介绍一下 Lazy 原理的关键之处。</p>
<h4 data-nodeid="26479">PersistentBag 为例详解原理</h4>
<p data-nodeid="26480">通过 PersistentBag 的关键源码，我们来看一下集合类 List 是怎么实现的，代码如下所示。</p>
<pre class="lang-java" data-nodeid="26481"><code data-language="java"><span class="hljs-comment">//PersistentBag继承AbstractPersistentCollection，从而继承了PersistenceCollection的一些公共功能、session的持有、lazy的特性、entity的状态转化等功能；同时PersistentBag也实现了java.util.List的所有方法，即对List进行读写的时候包装lazy逻辑</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PersistentBag</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractPersistentCollection</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">List</span> </span>{
    <span class="hljs-comment">//PersistentBag构造方法，当初始化实体对象对集合初始化的时候，把当前的Session保持住</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">PersistentBag</span><span class="hljs-params">(SessionImplementor session)</span> </span>{
       <span class="hljs-keyword">this</span>( (SharedSessionContractImplementor) session );
    }
    <span class="hljs-comment">// 从这个方法可以看出来其对List和ArrayList的支持</span>
    <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">PersistentBag</span><span class="hljs-params">(SharedSessionContractImplementor session, Collection coll)</span> </span>{
       <span class="hljs-keyword">super</span>( session );
       providedCollection = coll;
       <span class="hljs-keyword">if</span> ( coll <span class="hljs-keyword">instanceof</span> List ) {
          bag = (List) coll;
       }
       <span class="hljs-keyword">else</span> {
          bag = <span class="hljs-keyword">new</span> ArrayList( coll );
       }
       setInitialized();
       setDirectlyAccessible( <span class="hljs-keyword">true</span> );
    }
    <span class="hljs-comment">//以下是一些关键的List的实现方法，基本上都是在原有的List功能的基础上增加调用父类AbstractPersistentCollection里面的read()和write()方法</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">remove</span><span class="hljs-params">(<span class="hljs-keyword">int</span> i)</span> </span>{
       write();
       <span class="hljs-keyword">return</span> bag.remove( i );
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">set</span><span class="hljs-params">(<span class="hljs-keyword">int</span> i, Object o)</span> </span>{
       write();
       <span class="hljs-keyword">return</span> bag.set( i, o );
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List <span class="hljs-title">subList</span><span class="hljs-params">(<span class="hljs-keyword">int</span> start, <span class="hljs-keyword">int</span> end)</span> </span>{
       read();
       <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> ListProxy( bag.subList( start, end ) );
    }
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">entryExists</span><span class="hljs-params">(Object entry, <span class="hljs-keyword">int</span> i)</span> </span>{
       <span class="hljs-keyword">return</span> entry != <span class="hljs-keyword">null</span>;
    }
    <span class="hljs-comment">// toString被调用的时候会触发read()</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
       read();
       <span class="hljs-keyword">return</span> bag.toString();
    }
    .....<span class="hljs-comment">//其他方法类似，就不一一举例了</span>
}
</code></pre>
<p data-nodeid="26482">那么我们再看一下 AbstractPersistentCollection 的关键实现，在 AbstractPersistentCollection 中会有大量通过 Session 来初始化关联关系的方法，这些方法基本是利用当前 Session 和当前 Session 中持有的 Connection 来重新操作 DB，从而取到数据库里面的数据。</p>
<p data-nodeid="29422">所以我们发生的 LazyInitializationExcetion 基本都是从这个类里面抛出来的，从源码里面可以看到其严重依赖当前的 Session，关键源码如下图所示。</p>
<p data-nodeid="29423" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/12/Ciqc1F_HTbOAXgh4AAEtSArhDWw293.png" alt="Drawing 1.png" data-nodeid="29427"></p>


<p data-nodeid="30504">所以在默认的情况下，如果我们把 Session 关闭了，想利用 Lazy 的机制加载管理关系，就会发生异常了。我们通过实例看一下，在上面例子的 Controller 上加一个 debug 断点，可以看到如下图显示的内容：我们的 Address 指向了 PersistentBag 代理实例类。</p>
<p data-nodeid="30505" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/1D/CgqCHl_HTbeARnVnAAFroRKKQe4214.png" alt="Drawing 2.png" data-nodeid="30509"></p>


<p data-nodeid="26487">同时我们再设置断点的话也可以看到，PersistentBag 被初始化的时候，会传进来 Session 的上下文，即包含 Datasource 和需要执行 Lazy 的 sql。</p>
<p data-nodeid="31586">而需要执行 Lazy 的 sql，我们通过 debug 的栈信息可以看到其中有个 instantiate，有兴趣的同学可以 debug 看一下，关键断点信息如下图所示。</p>
<p data-nodeid="31587" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/1D/CgqCHl_HTb2ARhUkAAQSJOfdHvc744.png" alt="Drawing 3.png" data-nodeid="31591"></p>


<p data-nodeid="32668">再继续 debug 的话，也会看到调用 AbstractPersistentCollection 的初始化 Lazy 的方法，如下所示。</p>
<p data-nodeid="32669" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/12/Ciqc1F_HTcSALyT5AADMzJ35Sgg935.png" alt="Drawing 4.png" data-nodeid="32673"></p>


<p data-nodeid="26492">通过源码分析和实例讲解，你已经基本上知道了 Lazy 的原理，也就是需要 Lazy 的关联关系会初始化成 PersistentCollection，并且依赖持有的 Session。而当操作 List、Map 等集合类的一些基本方法的时候会触发 read()，并利用当前的 Session 进行懒加载。</p>
<p data-nodeid="26493">那么在实际工作中，哪些场景可能会产生 Lazy 异常呢？</p>
<h3 data-nodeid="26494">Lazy 异常的常见场景与解决方法</h3>
<h4 data-nodeid="26495">场景一：跨事务，事务之外的场景</h4>
<p data-nodeid="26496">我们在前面的课时讲过 Session 和事务之间的关系，当 spring.jpa.open-in-view=false 的时候，每个事务就会独立持有 Session；那么当我们在事务之外操作 lazy 的关联关系的时候，就容易发生 Lazy 异常。</p>
<p data-nodeid="26497">正如上面列举的 Demo 一样，一开始我就将 open-in-view 设置成了 false，而 userInfoRepository.findById(id) 又是一个独立事务，方法操作结束之后事务结束，事务结束之后 session close。所以当我们再操作 UserInfo 中 Address 对象的时候，就发生了 Lazy 异常。</p>
<p data-nodeid="26498">实际工作中这种情况比较多见，应该如何解决呢？</p>
<p data-nodeid="26499"><strong data-nodeid="26617">第一种方式：简单粗暴地设置为 spring.jpa.open-in-view=true</strong></p>
<p data-nodeid="26500">通过上面的分析，我们可以知道无非就是 Session 的关闭导致了 Lazy 异常，所以简单粗暴的办法就是加大 Session 的生命周期，将 Session 的生命周期和请求的生命周期设置成一样。但是 open-in-view 可能会带来的副作用你必须要牢记于心，有以下几点。</p>
<ol data-nodeid="26501">
<li data-nodeid="26502">
<p data-nodeid="26503">它对 Connection 的影响是什么？连接池有没有很好地监控？利用率是怎么样的？</p>
</li>
<li data-nodeid="26504">
<p data-nodeid="26505">实体的状态在整个 Session 的生命周期之间的变更都是有效的，数据的更新是不是预期的，你要心里有数。</p>
</li>
<li data-nodeid="26506">
<p data-nodeid="26507">N+1 的 SQL 是不是我们期望的？（这个下一讲我会详细介绍）性能有没有影响？等等。</p>
</li>
</ol>
<p data-nodeid="26508"><strong data-nodeid="26625">第二种方式：也是简单粗暴改成 Eager 模式</strong></p>
<p data-nodeid="26509">我们直接采用 Eager 的模式，这样也不会有 Lazy 异常的问题。如下述代码所示。</p>
<pre class="lang-java" data-nodeid="26510"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-comment">//直接采用eager模式</span>
   <span class="hljs-meta">@OneToMany(mappedBy = "userInfo",cascade = CascadeType.PERSIST,fetch = FetchType.EAGER)</span>
   <span class="hljs-keyword">private</span> List&lt;Address&gt; addressList;
</code></pre>
<p data-nodeid="26511">但是这种做法我不推荐你使用，因为本来我不想查 Address 信息，这样会白白地触发对 Address 的查询，导致性能有点浪费。<br>
<strong data-nodeid="26632">第三种方式：将可能发生 Lazy 的操作和取数据放在同一个事务里面</strong></p>
<p data-nodeid="26512">这种方式怎么理解呢？我们改造一下上面 Demo 的 Controller 的写法，代码如下所示。</p>
<pre class="lang-java" data-nodeid="26513"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@Log4j2</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoController</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> UserInfoService userInfoService;
   <span class="hljs-meta">@GetMapping("/user/info/{id}")</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromPath</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> Long id)</span> </span>{
     <span class="hljs-comment">//controller里面改调用service方法，这个service明确地返回了UserInfo和Address信息</span>
      UserInfo u1 =  userInfoService.getUserInfoAndAddress(id);
      System.out.println(u1.getAddressList().get(<span class="hljs-number">0</span>).getCity());
      <span class="hljs-keyword">return</span> u1;
   }
</code></pre>
<p data-nodeid="26514">Service 的实现如下所示，我们在里面用事务包装，利用事务，让可能触发 Lazy 的操作提前在事务里面发生。</p>
<pre class="lang-java" data-nodeid="26515"><code data-language="java"><span class="hljs-comment">/**
 * 我们把逻辑封装在service方法里面，方法名字语义要清晰，就是说这个方法会取UserInfo的信息和Address的信息
 * <span class="hljs-doctag">@param</span> id
 */</span>
<span class="hljs-meta">@Override</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoAndAddress</span><span class="hljs-params">(Long id)</span> </span>{
   UserInfo u1 =  userInfoRepository.findById(id).get();
   u1.getAddressList().size();<span class="hljs-comment">//在同一个事务里面触发lazy；不需要查询address的地方就不需要触发了</span>
   <span class="hljs-keyword">return</span> u1;
}
</code></pre>
<p data-nodeid="26516">这个时候就要求我们对方法名的语义和注释比较清晰了，这个方法还有个缺点，就是 Service 返回的依然还是 UserInfo 的实体，如果在关联关系多的情况下，依然有犯错的可能性发生。</p>
<p data-nodeid="26517"><strong data-nodeid="26639">第四种方式：Service 层之外都用 DTO 或者其他 POJO，而不是 Entity</strong></p>
<p data-nodeid="26518">这种是最复杂的，但却是最有效的、不会出问题的方式，我们在 Service 层返回 DTO，改造一下 Service 方法，代码如下所示。</p>
<pre class="lang-java" data-nodeid="26519"><code data-language="java"><span class="hljs-meta">@Transactional</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfoDto <span class="hljs-title">getUserInfoAndAddress</span><span class="hljs-params">(Long id)</span> </span>{
   UserInfo u1 =  userInfoRepository.findById(id).get();
   <span class="hljs-comment">//按照业务要求，需要什么返回什么就可以了，让实体在service层之外是不可见的</span>
   <span class="hljs-keyword">return</span> UserInfoDto.builder().name(u1.getName()).addressList(u1.getAddressList()).build();/
}
</code></pre>
<p data-nodeid="26520">而 UserInfoDto 也就是根据我们的业务需要创建不同的 DTO 即可。例如，我们只需要 name 和 address 的时候，代码如下。</p>
<pre class="lang-java" data-nodeid="26521"><code data-language="java"><span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoDto</span> </span>{
    <span class="hljs-keyword">private</span> String name;
    <span class="hljs-keyword">private</span> List&lt;Address&gt; addressList;
}
</code></pre>
<p data-nodeid="26522">除了 DTO 我们还可以采用任何语义的 POJO，宗旨就是 Entity 对 Service 层之外是不可见的。也可以采用“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=490#/detail/pc?id=4704" data-nodeid="26645">04 | 如何利用 Repository 中的方法返回值解决实际问题？</a>”讲过的 Projection 的方式，返回接口类型的 POJO，这样控制的粒度更细，读写都可以分开。</p>
<p data-nodeid="26523">将 Entity 控制在 Service 层还有个好处就是，有的时候我们会使用各种 RPC 框架进行远程方法调度，可以直接调用 Service 方法通过 TCP 协议，例如 Dubbo，这样也就天然支持了。</p>
<h4 data-nodeid="26524">场景二：异步线程的时候</h4>
<p data-nodeid="26525">既然跨事务容易发生问题，那么异步线程的时候更容易发生 Lazy 异常，你可以先自己想一想该怎么解决。</p>
<p data-nodeid="26526">异步线程的时候，我们再套用上面的四种方式，你会发现，其中的第一种就不适用了，因为异步开启的事务和 DB 操作默认是不受 open-in-view 控制的。所以我们可以明确地知道，开启的异步方法会用到实体参数的哪些关联关系，是否需要按照上面的第三种和第四种方式进行提前处理呢？这些都是需要我们心中有数的，而不是简单地把异步开启就完事了。</p>
<h4 data-nodeid="26527">场景三：Controller 直接返回实体也会产生 Lazy Exception</h4>
<p data-nodeid="26528">工作中我们经常为了省事，直接在 Contoller 里面返回 Entity，这个时候很容易发生 Lazy 异常，例如下面这个场景。</p>
<pre class="lang-java" data-nodeid="26529"><code data-language="java"><span class="hljs-meta">@GetMapping("/user/info/{id}")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> UserInfo <span class="hljs-title">getUserInfoFromPath</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable("id")</span> Long id)</span> </span>{
  <span class="hljs-keyword">return</span> userInfoRepository.findById(id).get();<span class="hljs-comment">//controller层直接将UserInfo返回给view层了；</span>
}
</code></pre>
<p data-nodeid="26530">类似上面的 Contoller，我们直接将 UserInfo 实体对象当成 VO 对象，且直接当成返回结果了，当我们请求上面的 API 的时候也会发生 Lazy 异常，我们看下代码。</p>
<pre class="lang-java" data-nodeid="26531"><code data-language="java">o.hibernate.LazyInitializationException  : failed to lazily initialize a collection of role: com.example.jpa.demo.db.UserInfo.addressList, could not initialize proxy - no Session
   org.hibernate.LazyInitializationException: failed to lazily initialize a collection of role: com.example.jpa.demo.db.UserInfo.addressList, could not initialize proxy - no Session
</code></pre>
<p data-nodeid="26532">此 VO 发生的异常与其他 Lazy 异常不同的时候，我们仔细观察，会发现如下信息。</p>
<pre class="lang-java" data-nodeid="26533"><code data-language="java">&nbsp;Resolved [org.springframework.http.converter.HttpMessageNotWritableException: Could not write JSON:&nbsp;&nbsp;failed to lazily initialize a collection of role: com.example.jpa.demo.db.UserInfo.addressList
</code></pre>
<p data-nodeid="26534">通过日志可以知道，此时发生 Lazy 异常的主要原因是 JSON 系列化的时候会触发 Lazy 加载。这个时候就有了第五种解决 Lazy 异常的方式，就是利用 @JsonIgnoreProperties("addressList") 排除我们不想序列化的属性即可。</p>
<p data-nodeid="26535">但是这种方式的弊端是用这个集合的只能全局配置，没办法有特例配置。因此最佳实践还是采用上面所说的第一种方式和第四种方式。</p>
<h4 data-nodeid="26536">场景四：自定义的拦截器和 filter 中无意的 toString 操作</h4>
<p data-nodeid="26537">第四个 Lazy 异常的场景就是我们打印一些日志，或者无意间触发 toString 的操作也会发生 Lazy 异常，这种处理方法也很简单（第六种处理 Lazy 异常的方式）：toString 里面排除掉不需要 Lazy 加载的关联关系即可。如果我们用 lombok 的话，直接 @ToString(exclude = "addressList") 排除掉就好了，完整例子如下所示。</p>
<pre class="lang-java" data-nodeid="26538"><code data-language="java"><span class="hljs-meta">@ToString(exclude = "addressList")</span>
<span class="hljs-meta">@JsonIgnoreProperties("addressList")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
......}
</code></pre>
<p data-nodeid="26539">以上介绍的四个 Lazy 异常的场景和六种处理方式，你在实际工作中可以灵活运用，其中最主要的是要知道背后的原理和触发 Lazy 产生的性能影响是什么（意外的 SQL 执行）。</p>
<p data-nodeid="26540">Hibernate 官方还提供了第七种处理 Lazy 异常的方式：利用 Hibernate 的配置，我们来了解一下。</p>
<h3 data-nodeid="26541">hibernate.enable_lazy_load_no_trans 配置</h3>
<p data-nodeid="26542">Hibernate 官方提供了 hibernate.enable_lazy_load_no_trans 配置，是否允许在关闭之后依然支持 Lazy 加载，此非 JPA 标准，所以你在用的时候需要关注版本变化。</p>
<p data-nodeid="26543">其使用方法很简单，我们直接在 application.properties 里面增加如下配置即可，请看下面的代码。</p>
<pre class="lang-java" data-nodeid="26544"><code data-language="java">## 运行在session关闭之后，重新lazy操作
spring.jpa.properties.hibernate.enable_lazy_load_no_trans=true
</code></pre>
<p data-nodeid="26545">此时我们不需要做任何其他修改，当在事务之外，甚至是 Session 之外，触发 Lazy 操作的时候也不会报错，也会正常地进行取数据。</p>
<p data-nodeid="26546">但是我建议你不要用这个方法，因为一旦开启了这个对 Lazy 的操作就不可控了，会发生预期之外的 Lazy 异常，然后你只能通过我们上面所说的处理 Lazy 异常的第三种和第四种方式解决成预期之内的，否则的话，还会带来很多预期之外的 SQL 执行。这就会造成一种误解，即使用 Hibernate 或者 JPA 会导致性能变差，其实本质原因是我们不了解原理，没能正确使用。</p>
<p data-nodeid="26547">所以到目前为止，Spring Data JPA 中，hibernate.enable_lazy_load_no_trans 默认是 false，这和 spring.jpa.open-in-view 默认是 true 是相同的道理。所以如果我们都采用 Spring Boot 的默认配置，一般是没有任何问题的；而有的时候为了更优的配置，我们需要知道底层的原理，这样才能判断出来我们业务场景的最佳实践是什么。</p>
<p data-nodeid="26548">以上我重点介绍了 LazyInitializationException，其实 JPA 里面的异常类型还非常多，下面简单介绍一下。</p>
<h3 data-nodeid="26549">Javax.persistence.PersistenceException 异常类型</h3>
<p data-nodeid="33750">我们顺藤摸瓜，可以看到 LazyInitializationException 是 HibernateException 里面的，也可以看到 HibernateException 的父类 Javax.persistence.PersistenceException 下面有很多细分的异常，如下图所示。</p>
<p data-nodeid="33751" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/75/1D/CgqCHl_HTdmADLLsAAJrRYrE3Gk475.png" alt="Drawing 5.png" data-nodeid="33755"></p>


<p data-nodeid="26552">当我们遇到异常的时候不要慌张，仔细看日志基本就能知道是什么问题了。</p>
<p data-nodeid="26553">另外需要注意的是，当我们遇到上面这些异常的时候，不同的异常有不同的处理方式，比如 OptimisticLockException 就需要进行重试；而针对 NoSuchBeanException 的异常，就要检查我们的实体配置是否妥当。</p>
<p data-nodeid="26554">你通过上面的异常 Hierarchy 做到心中有数就好了，遇到实际情况再实际分析即可。</p>
<h3 data-nodeid="26555">总结</h3>
<p data-nodeid="26556">所以为什么我们总会遇到 Lazy Exception 呢？当我们知道原理之后是不是应对起来就非常游刃有余了？而 spring.jpa.open-in-view 设置 true/false 是个权衡性的问题，没有绝对的对和错，就看我们的使用场景是什么样的了。</p>
<p data-nodeid="26557">总之遇到问题不要慌，看一下源码，想一下我们讲的原理知识，或许你就能找到答案。下一讲我们来聊聊经典的 N+1 的 SQL 问题。到时见。</p>
<blockquote data-nodeid="26558">
<p data-nodeid="26559">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="26715">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **天：
> 异步的情况我们会重新在异步线程里面findById()一下

##### **宝：
> 在Controller的方法上注入事务 保证2次查询在一个事务里面不就行了吗。。。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 也可以呀

##### **磊：
> 这样的话序列化插件会把所有的懒加载的节点都get一遍， 还不如即时加载。开启open session in view 开销会更大一些

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这个根据实际情况调整。

