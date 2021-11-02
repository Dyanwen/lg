<p data-nodeid="117375">你好，欢迎学习第 23 讲。通过前几讲对 Session 核心原理的学习，相信你已经可以解决实际工作中的一些疑难杂症了。这一讲我再给你举一个复杂点的例子，继续深度剖析如何利用 Session 原理解决复杂问题。那么，都有哪些问题呢？我们看一个例子。</p>


<h3 data-nodeid="116612">CompletableFuture 的使用实际案例</h3>
<p data-nodeid="116613">在我们的实际开发过程中，难免会用到异步方法，我在这里列举一个异步方法的例子，经典地还原一些在异步方法里面经常会犯的错误。</p>
<p data-nodeid="116614">我们模拟一个 Service 方法，通过异步操作，更新 UserInfo 信息，并且可能一个方法里面有不同的业务逻辑，会多次更新 UserInfo 信息，模拟的代码如下。</p>
<pre class="lang-java" data-nodeid="116615"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoController</span> </span>{
   <span class="hljs-comment">//异步操作必须要建立线程池，这个不多说了，因为不是本讲的重点，有兴趣的话你可以了解一下线程池的原理，我的Demo采用的是Spring异步框架字段的异步线程池</span>
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span>  Executor executor;
   <span class="hljs-comment">/**
    * 模拟一个业务service方法，里面有一些异步操作，一些业务方法里面可能修改了两次用户信息
    * <span class="hljs-doctag">@param</span> name
    * <span class="hljs-doctag">@return</span>
    */</span>
   <span class="hljs-meta">@PostMapping("test/async/user")</span>
   <span class="hljs-meta">@Transactional</span> <span class="hljs-comment">// 模拟一个service方法，期待是一个事务</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">testSaveUser</span><span class="hljs-params">(String name)</span> </span>{
   CompletableFuture&lt;Void&gt; cf = CompletableFuture.runAsync(() -&gt; {
      UserInfo user = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
      <span class="hljs-comment">//..... 此处模拟一些业务操作，第一次改变UserInfo里面的值；</span>
      <span class="hljs-keyword">try</span> {
         Thread.sleep(<span class="hljs-number">200L</span>);<span class="hljs-comment">// 加上复杂业务耗时200毫秒</span>
      } <span class="hljs-keyword">catch</span> (InterruptedException e) {
         e.printStackTrace();
      }
      user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_first"</span>+name); <span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
      userInfoRepository.save(user);
      <span class="hljs-comment">//..... 此处模拟一些业务操作，第二次改变UserInfo里面的值；</span>
      <span class="hljs-keyword">try</span> {
         Thread.sleep(<span class="hljs-number">300L</span>);<span class="hljs-comment">// 加上复杂业务耗时300毫秒</span>
      } <span class="hljs-keyword">catch</span> (InterruptedException e) {
         e.printStackTrace();
      }
      user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_second"</span>+name);<span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
      userInfoRepository.save(user);
   }, executor).exceptionally(throwable -&gt; {
      throwable.printStackTrace();
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   });
   <span class="hljs-comment">//... 实际业务中，可能还有会其他异步方法，我们举这一个例子已经可以说明问题了</span>
   cf.isDone();
   <span class="hljs-keyword">return</span> <span class="hljs-string">"Success"</span>;
}
}
</code></pre>
<p data-nodeid="116616">为了便于测试，我们在 UserInfoController 里面模拟了一个复杂点的 Service 方法，上面的代码很多是为了方便给你演示和做测试，实际工作中可能代码会不一样、会演变，但是你通过实质分析，就会发现解决思路是一样的。</p>
<p data-nodeid="116617">我们在 testSaveUser 方法里面开了一个异步线程，异步线程采用 CompletableFuture 方法，在里面执行了两次 UserInfo 的 Save 操作，实际工作中可能不会有像我的 Demo 那么简单的 Save，因为我把中间的业务计算省去了，这不影响我们分析问题。</p>
<p data-nodeid="116618">那么上面的代码问题的表象是什么呢？</p>
<h4 data-nodeid="116619">表现出来的问题现状是什么样的？</h4>
<p data-nodeid="116620">那么实际工作中，我们如果写出来类似的代码，会发生什么样的问题呢？</p>
<ol data-nodeid="116621">
<li data-nodeid="116622">
<p data-nodeid="116623">整个请求非常正常，永远都是 200；也没有任何报错信息，但是发现数据库里面第二次的 save(user) 永远不生效，永远不会出现 name 包含 "_second" 的记录，这个是必现的；</p>
</li>
<li data-nodeid="116624">
<p data-nodeid="116625">整个请求非常正常，永远都是 200；也没有任何报错信息，有的时候会发现数据库里面没有任何变化，甚至第一次 save(user) 都没有生效，但是这个是偶发的。</p>
</li>
</ol>
<p data-nodeid="116626">实际工作中我们肯定会通过 QA 或者自己多次测试，发现以上现象就会感觉非常奇怪，那么我们来分步拆解一下，看看怎么解决？</p>
<h3 data-nodeid="116627">步骤拆解</h3>
<p data-nodeid="116628">有一定经验的开发者，遇到类似问题，第一步应该想到是不是发生什么异常了？日志信息去哪里了？那么我们需要先看一下 CompletableFuture 的用法，是不是发生异常的时候我们漏掉了什么环节？</p>
<h4 data-nodeid="116629">CompletableFuture 使用最佳实践</h4>
<p data-nodeid="116630">CompletableFuture 主要的功能是实现了 Future 和 CompletionStage 的接口，主要的方法如下述代码所示。</p>
<pre class="lang-java" data-nodeid="116631"><code data-language="java"><span class="hljs-comment">//通过给定的线程池，异步执行 Runnable方法，不带返回结果</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> CompletableFuture&lt;Void&gt; 	<span class="hljs-title">runAsync</span><span class="hljs-params">(Runnable runnable, Executor executor)</span>
<span class="hljs-comment">//通过给定的线程池，异步执行Runnable方法，带返回结果</span>
<span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> &lt;U&gt; CompletableFuture&lt;U&gt; 	<span class="hljs-title">supplyAsync</span><span class="hljs-params">(Supplier&lt;U&gt; supplier, Executor executor)</span>
<span class="hljs-comment">//当上面的异步方法执行完之后需要执行的回调方法</span>
<span class="hljs-keyword">public</span> CompletableFuture&lt;Void&gt; 	<span class="hljs-title">thenAccept</span><span class="hljs-params">(Consumer&lt;? <span class="hljs-keyword">super</span> T&gt; action)</span>
<span class="hljs-comment">//阻塞等待 future执行完结果</span>
<span class="hljs-keyword">boolean</span> <span class="hljs-title">isDone</span><span class="hljs-params">()</span></span>;
<span class="hljs-comment">//阻塞获取结果</span>
<span class="hljs-function">V <span class="hljs-title">get</span><span class="hljs-params">()</span></span>;
<span class="hljs-comment">//当异步操作发生异常的时候执行的方法</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> CompletionStage&lt;T&gt; <span class="hljs-title">exceptionally</span><span class="hljs-params">(Function&lt;Throwable, ? extends T&gt; fn)</span></span>;
</code></pre>
<p data-nodeid="116632">以上我只是列举了一些和我们案例相关的关键方法，而 CompletableFuture 还有更多的方法，其功能也非常强大，所以一般开发过程中用此类的场景还非常多。</p>
<p data-nodeid="116633">其实上面的 Demo 只是利用 runAsync 做了异步操作，并利用 isDone 做了阻塞等待的动作，而没有使用 Exceptionally 处理异常信息。</p>
<p data-nodeid="116634">所以如果我们想打印异常信息，基本上可以利用 Exceptionally。我们改进一下 Demo 代码，把异常信息打印一下，看看是否发生了异常。变动的代码如下所示。</p>
<pre class="lang-java" data-nodeid="116635"><code data-language="java">CompletableFuture&lt;Void&gt; cf = CompletableFuture.runAsync(() -&gt; {
   ......这里的代码不变，我们不做copy了
}, executor).exceptionally(e -&gt; {
   log.error(e);<span class="hljs-comment">//把异常信息打印出来</span>
   <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
});
</code></pre>
<p data-nodeid="116636">那么我们再请求上面的 Controller 方法的时候，发现控制台就会打印出如下所示的 Error 信息。</p>
<pre class="lang-java" data-nodeid="116637"><code data-language="java">java.util.concurrent.CompletionException: org.springframework.orm.ObjectOptimisticLockingFailureException: Object of class [com.example.jpa.demo.db.UserInfo] with identifier [1]: optimistic locking failed; nested exception is org.hibernate.StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect) : [com.example.jpa.demo.db.UserInfo#1]
	at java.base/java.util.concurrent.CompletableFuture.encodeThrowable(CompletableFuture.java:314)
	at java.base/java.util.concurrent.CompletableFuture.completeThrowable(CompletableFuture.java:319)
	at java.base/java.util.concurrent.CompletableFuture$AsyncRun.run$$$capture(CompletableFuture.java:1739)
	at java.base/java.util.concurrent.CompletableFuture$AsyncRun.run(CompletableFuture.java)
	at java.base/java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1167)
	at java.base/java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:641)
	at java.base/java.lang.Thread.run(Thread.java:844)
Caused by: org.springframework.orm.ObjectOptimisticLockingFailureException: Object of class [com.example.jpa.demo.db.UserInfo] with identifier [1]: optimistic locking failed; nested exception is org.hibernate.StaleObjectStateException: Row was updated or deleted by another transaction (or unsaved-value mapping was incorrect) : [com.example.jpa.demo.db.UserInfo#1]
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.convertHibernateAccessException(HibernateJpaDialect.java:337)
	at org.springframework.orm.jpa.vendor.HibernateJpaDialect.translateExceptionIfPossible(HibernateJpaDialect.java:255)
	at org.springframework.aop.framework.ReflectiveMethodInvocation.proceed(ReflectiveMethodInvocation.java:186)
	at org.springframework.aop.framework.JdkDynamicAopProxy.invoke(JdkDynamicAopProxy.java:212)
	at com.sun.proxy.$Proxy116.save(Unknown Source)
	at com.example.jpa.demo.web.UserInfoController.lambda$testSaveUser$0(UserInfoController.java:57)
	at java.base/java.util.concurrent.CompletableFuture$AsyncRun.run$$$capture(CompletableFuture.java:1736)
	... 4 more
</code></pre>
<p data-nodeid="116638">通过报错信息，可以发现其实就是发生了乐观锁异常，导致上面实例中的第二次 save(user) 必然失败；而第一次 save(user) 的失败，主要是因为在并发的情况下有其他请求线程改变了 UserInfo 的值，也就是改变了 Version。</p>
<p data-nodeid="116639">我们来看一下完整的 UserInfo 对象实体。</p>
<pre class="lang-java" data-nodeid="116640"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@SuperBuilder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString(callSuper = true)</span>
<span class="hljs-meta">@Table</span>
<span class="hljs-meta">@EntityListeners({AuditingEntityListener.class})</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span></span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-meta">@Version</span>
   <span class="hljs-keyword">private</span> Integer version;
   <span class="hljs-meta">@CreatedBy</span>
   <span class="hljs-keyword">private</span> Integer createUserId;
   <span class="hljs-meta">@CreatedDate</span>
   <span class="hljs-keyword">private</span> Instant createTime;
   <span class="hljs-meta">@LastModifiedBy</span>
   <span class="hljs-keyword">private</span> Integer lastModifiedUserId;
   <span class="hljs-meta">@LastModifiedDate</span>
   <span class="hljs-keyword">private</span> Instant lastModifiedTime;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-keyword">private</span> String lastName;
   <span class="hljs-keyword">private</span> String emailAddress;
   <span class="hljs-keyword">private</span> String telephone;
}
</code></pre>
<p data-nodeid="116641">看过前面课时的同学应该知道，我们通过 @Version 乐关锁机制就是防止数据被覆盖；而实际生产过程中其实很难发现类似问题。</p>
<p data-nodeid="116642">所以当我们使用任何的异步线程处理框架的时候，一定要想好异常情况下怎么打印日志，否则就像黑洞一样，完全不知道发生了什么。</p>
<p data-nodeid="116643">那么既然知道发生了乐观锁异常，这里就有个疑问了：我们不是在 UserInfoController 的 testSaveUser 方法上面加了 @Transaction 的注解了吗？为什么事务没有回滚？</p>
<h4 data-nodeid="116644">通过日志查看事务的执行过程</h4>
<p data-nodeid="116645">我们看看异步请求的情况下，事务应该怎么做呢？先打开事务的日志，看看上面方法的事务执行过程是什么样的。</p>
<pre class="lang-java" data-nodeid="116646"><code data-language="java">## 我们在db的连接中开启logger=Slf4JLogger&amp;profileSQL=true看一下每个事务里执行的sql有哪些
spring.datasource.url=jdbc:mysql://localhost:3306/test?logger=Slf4JLogger&amp;profileSQL=true
## 打开下面这些类的日志级别，观察一下事务的开启和关闭时机
logging.level.org.springframework.orm.jpa=DEBUG
logging.level.org.springframework.transaction=DEBUG
logging.level.org.springframework.orm.jpa.JpaTransactionManager=trace
logging.level.org.hibernate.engine.transaction.internal.TransactionImpl=DEBUG
</code></pre>
<p data-nodeid="118381">再请求一下刚才的测试接口：POST<a href="http://127.0.0.1:8087/test/async/user?name=jack" data-nodeid="118386">http://127.0.0.1:8087/test/async/user?name=jack</a>就会产生下图所示的日志。</p>
<p data-nodeid="118382" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/11/Ciqc1F_As0GAa7gmAAQU-sy_Y6s946.png" alt="Drawing 0.png" data-nodeid="118390"></p>


<p data-nodeid="116649">先看一下上半部分，通过日志我们可以看到，首先执行这个方法的时候开启了两个事务，分别做如下解释。</p>
<p data-nodeid="116650"><strong data-nodeid="116769">线程 1</strong>：[nio-8087-exec-1] 开启了 UserInfoController.testSaveUser 方法上面的事务，也就是 http 的请求线程，开启了一个 Controller 请求事务。这是因为我们在 testSaveUser 的方法上面加了 @Transaction 的注解，所以开启了一个事务。</p>
<p data-nodeid="116651">而通过日志我们也可以发现，事务 1 里面什么都没有做，随后就进行了 Commit 操作，所以我们可以看得出来，默认不做任何处理的情况下，事务是不能跨线程的。每个线程里面的事务相互隔离、互不影响。</p>
<p data-nodeid="116652" class=""><strong data-nodeid="116781">线程 2</strong>：[&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;task-1]，通过异步线程池开启了 SimpleJpaRepository.findById 方法上面的只读事务。这是因为默认的 SimpleJpaRepository 类上面加了 @Transaction(readOnly=true) 产生的结果。而我们通过 MySQL 的日志也可以看得出来，此次事务里面只做了和我们代码相关的 select user_info 的操作。</p>
<p data-nodeid="119395">我们再看一下后半部分的日志，如图所示。</p>
<p data-nodeid="119396" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/1C/CgqCHl_As2KAPPhLAAVgCANdWbo463.png" alt="Drawing 1.png" data-nodeid="119400"></p>


<p data-nodeid="116655">通过后半部分日志，我们可以看到两次 save(user) 方法，也分别开启了各自的事务，这是因为 SimpleJpaRepository.save 方法上面有 @Transaction 注解起了作用，而第二次事务因为 JPA 的实现方法判断了数据库这条数据的 Version 和我们 UserInfo 的对象中的 Version 不一致，从而第二次进行了回滚操作。</p>
<p data-nodeid="116656">两次 save(user) 的操作里面分别有一次 Select 和 Update 语义，正是我们之前所说的 Save 方法的原理。两次事务，分别开启了两个 Session，所以对象对于这两次 Session 来说分别是从游离态（Detached）转成持久态（Persistent）的过程，所以两个独立的事务里面，一次 Select，一次 Update。</p>
<p data-nodeid="116657">通过日志可以看到，上面一个简单的方法中一共发生了四次事务，都是采用的默认隔离级别和传播机制。那么如果我们想让异步方法里面只有一个事务应该怎么办呢？</p>
<h4 data-nodeid="116658">异步事务的正确使用方法</h4>
<p data-nodeid="116659">既然我们知道异步方法里面的事务是独立的，那么直接把异步的代码块用独立的事务包装起来即可，做法有如下几种。</p>
<p data-nodeid="116660">第一种处理方法：把其中的异步代码块，移到一个外部类里面。我们这里放到 UserInfoService 中，同时方法中加上 @Transaction 注解用来开启事务，加上 @Retryable 注解进行乐观锁重试，代码如下。</p>
<pre class="lang-java" data-nodeid="116661"><code data-language="java"><span class="hljs-comment">//加上事务，这样可以做到原子性，解决事务加到异常方法之外没有任何作用的问题</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-comment">//加上重试机制，这样当我们发生乐观锁异常的时候，重新尝试下面的逻辑，减少请求的失败次数</span>
<span class="hljs-meta">@Retryable(value = ObjectOptimisticLockingFailureException.class,backoff = @Backoff(multiplier = 1.5,random = true))</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">businessUserMethod</span><span class="hljs-params">(String name)</span> </span>{
   UserInfo user = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
   <span class="hljs-comment">//..... 此处模拟一些业务操作，第一次改变UserInfo里面的值；</span>
   <span class="hljs-keyword">try</span> {
      Thread.sleep(<span class="hljs-number">200L</span>);<span class="hljs-comment">// 加上复杂业务耗时200毫秒</span>
   } <span class="hljs-keyword">catch</span> (InterruptedException e) {
      e.printStackTrace();
   }
   user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_first"</span>+name); <span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
   userInfoRepository.save(user);
   <span class="hljs-comment">//..... 此处模拟一些业务操作，第二次改变UserInfo里面的值；</span>
   <span class="hljs-keyword">try</span> {
      Thread.sleep(<span class="hljs-number">300L</span>);<span class="hljs-comment">// 加上复杂业务耗时300毫秒</span>
   } <span class="hljs-keyword">catch</span> (InterruptedException e) {
      e.printStackTrace();
   }
   user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_second"</span>+name);<span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
   userInfoRepository.save(user);
}
</code></pre>
<p data-nodeid="116662">那么 Controller 里面只需要变成如下写法即可。</p>
<pre class="lang-java" data-nodeid="116663"><code data-language="java"><span class="hljs-comment">/**
 * 模拟一个业务service方法，里面有一些异步操作，一些业务方法里面可能修改了两次用户信息
 * <span class="hljs-doctag">@param</span> name
 * <span class="hljs-doctag">@return</span>
 */</span>
<span class="hljs-meta">@PostMapping("test/async/user")</span>
<span class="hljs-meta">@Transactional</span> <span class="hljs-comment">// 模拟一个service方法，期待是一个事务</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">testSaveUser</span><span class="hljs-params">(String name)</span> </span>{
   CompletableFuture&lt;Void&gt; cf = CompletableFuture.runAsync(() -&gt; {
      userInfoService.businessUserMethod(name);
   }, executor).exceptionally(e -&gt; {
      log.error(e);<span class="hljs-comment">//把异常信息打印出来</span>
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   });
   <span class="hljs-comment">//... 实际业务中，可能还有会其他异步方法，我们举这个例子已经可以说明问题了</span>
   cf.isDone();
   <span class="hljs-keyword">return</span> <span class="hljs-string">"Success"</span>;
}
</code></pre>
<p data-nodeid="120405">我们再次发起一下请求，看一下日志。</p>
<p data-nodeid="120406" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/1C/CgqCHl_As2uAXKFdAAOga2Jcfio735.png" alt="Drawing 2.png" data-nodeid="120410"></p>


<p data-nodeid="116666">通过上图的日志，我们可以知道两个重要信息：</p>
<ol data-nodeid="116667">
<li data-nodeid="116668">
<p data-nodeid="116669">这个时候只有 UserInfoServiceImpl.businessUserMethod 开启了一个事务，这是因为 findById 和 Save 方法中，事务的传播机制都是“如果存在事务就利用当前事务”的原理，所以就不会像我们上面一样创建四次事务了；</p>
</li>
<li data-nodeid="116670">
<p data-nodeid="116671">而此时两次 save(user) 只产生了一个 update 的 sql 语句，并且也很难出现乐观锁异常了，因为这是 Session 的机制，将两次对 UserInfo 实体的操作进行了合并；所以当我们使用 JPA 的时候某种程度上也会降低 db 的压力，增加代码的执行性能。</p>
</li>
</ol>
<p data-nodeid="116672">而另外一个侧论，就是当事务的生命周期执行越快的时候，发生异常的概率就会越低，因为可以减少并发处理的机会。</p>
<p data-nodeid="116673">第二种处理方法：可以利用“<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=490#/detail/pc?id=4719" data-nodeid="116804">19 | 如何搞清楚事务、连接池的关系？正确配置是怎样的？</a>”讲过的 TransactionTemplate 方法开启事务，这里不再重复讲述了。</p>
<p data-nodeid="116674">第三种处理方法：我们可以建一个自己的 TransanctionHelper，并带上重试机制，代码如下：</p>
<pre class="lang-java" data-nodeid="116675"><code data-language="java"><span class="hljs-comment">/**
 * 利用spring进行管理
 */</span>
<span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TransactionHelper</span> </span>{
    <span class="hljs-comment">/**
     * 利用spring 机制和jdk8的Consumer机制实现只消费的事务
     */</span>
    <span class="hljs-meta">@Transactional(rollbackFor = Exception.class)</span> <span class="hljs-comment">//可以根据实际业务情况，指定明确的回滚异常</span>
    <span class="hljs-meta">@Retryable(value = ObjectOptimisticLockingFailureException.class,backoff = @Backoff(multiplier = 1.5,random = true))</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">transactional</span><span class="hljs-params">(Consumer consumer,Object o)</span> </span>{
        consumer.accept(o);
    }
}
</code></pre>
<p data-nodeid="116676">那么 Controller 里面的写法可以变成如下方式，也可以达到同样效果。</p>
<pre class="lang-java" data-nodeid="116677"><code data-language="java"><span class="hljs-meta">@PostMapping("test/async/user")</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">testSaveUser</span><span class="hljs-params">(String name)</span> </span>{
   CompletableFuture&lt;Void&gt; cf = CompletableFuture.runAsync(() -&gt; {
      transactionHelper.transactional((param)-&gt;{ <span class="hljs-comment">// 通过lambda实现事务管理</span>
         UserInfo user = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
         <span class="hljs-comment">//..... 此处模拟一些业务操作，第一次改变UserInfo里面的值；</span>
         <span class="hljs-keyword">try</span> {
            Thread.sleep(<span class="hljs-number">200L</span>);<span class="hljs-comment">// 加上复杂业务耗时200毫秒</span>
         } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
         }
         user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_first"</span>+name); <span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
         userInfoRepository.save(user);
         <span class="hljs-comment">//..... 此处模拟一些业务操作，第二次改变UserInfo里面的值；</span>
         <span class="hljs-keyword">try</span> {
            Thread.sleep(<span class="hljs-number">300L</span>);<span class="hljs-comment">// 加上复杂业务耗时300毫秒</span>
         } <span class="hljs-keyword">catch</span> (InterruptedException e) {
            e.printStackTrace();
         }
         user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_second"</span>+name);<span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
         userInfoRepository.save(user);
      },name);
   }, executor).exceptionally(e -&gt; {
      log.error(e);<span class="hljs-comment">//把异常信息打印出来</span>
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
   });
   <span class="hljs-comment">//... 实际业务中，可能还有会其他异步方法，我们举一个例子已经可以说明问题了</span>
   cf.isDone();
   <span class="hljs-keyword">return</span> <span class="hljs-string">"Success"</span>;
}
</code></pre>
<p data-nodeid="116678">这种方式主要是通过 Lambda 表达式解决事务问题。</p>
<p data-nodeid="116679">总之，不管是以上哪种方法，都可以解决我们所说的异步事务的问题。所以搞清楚事务的背后实现逻辑，就很容易解决类似问题了。</p>
<p data-nodeid="116680">还有一个问题就是，为什么当异步方法中是同一个事务的时候，第二次 save(user) 就成功了？而异步代码块里面的两个 save(user) 分别在两个事务里面，第二次就不成功了呢？我们利用前两个课时讲过的 Persistence Context 和实体的状态来分析一下。</p>
<h4 data-nodeid="116681">Session 的机制与 Repository.save(entity) 是什么关系？</h4>
<p data-nodeid="116682">我们在学习 Persistence Context 的时候，知道 Entity 有不同的状态。</p>
<p data-nodeid="116683">在一个 Session 里面，如果我们通过 findById(id) 得到一个 Entity，它就会变成 Manager（persist） 持久态。那么同一个 Session 里面，同一个 Entity 多次操作 Hibernate 就会进行 Merge 操作。</p>
<p data-nodeid="116684">所以上面的实例中，当我们在 businessUserMethod 方法上面加 @Transaction 的时候，会造成异步代码的整块逻辑处于同一个事务里面，而按照我们上一讲介绍的 Session 原理，同一个事务就会共享同一个 Session，所以同一个事务里面的 findById、save、save 的多次操作都是同一个实例。</p>
<p data-nodeid="121415">什么意思呢？我们可以通过设置 Debug 断点，查看一下对象的内存对象地址是否一样，就可以看得出来。如下图所示，findById 之后和两次 save 之后都是同一个对象。</p>
<p data-nodeid="121416" class=""><img src="https://s0.lgstatic.com/i/image/M00/72/11/Ciqc1F_As3uAcvehAAG5ahJbL1Y740.png" alt="Drawing 3.png" data-nodeid="121420"></p>


<p data-nodeid="116687">而如果我们跨 Session 传递实体对象，那么在一个 Session 里面持久态的对象，对于另外一个 Session 来说就是一个Detached（游离态）的对象。</p>
<p data-nodeid="116688">而根据 Session 里面的 Persistenc Context 的原理，一旦这个游离态的对象进行 db 操作，Session 会 Copy 一个新的实体对象。也就是说，当我们不在异步代码中加事务的时候，即去掉异步代码块businessUserMethod 方法中的@Transaction 注解，findById 之后就会产生一个新的事务、新的 Session，那么返回的就是对象 1；第一次 Save 之后，由于又是一个新的事务、新的 Session，那么返回的实体 u2 就是对象 2。</p>
<p data-nodeid="116689">我们知道这个原理之后，对代码做如下改动。</p>
<pre class="lang-java" data-nodeid="116690"><code data-language="java"><span class="hljs-comment">//  @Transactional 去掉事务</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">businessUserMethod</span><span class="hljs-params">(String name)</span> </span>{
      UserInfo user = userInfoRepository.findById(<span class="hljs-number">1L</span>).get();
      user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_first"</span>+name); <span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
      UserInfo u2 = userInfoRepository.save(user);
      user.setName(RandomUtils.nextInt(<span class="hljs-number">1</span>,<span class="hljs-number">100000</span>)+ <span class="hljs-string">"_second"</span>+name); <span class="hljs-comment">//模拟一些业务操作，改变了UserInfo里面的值</span>
      UserInfo u3 = userInfoRepository.save(u2);<span class="hljs-comment">// 第二次save采用第一次save的返回结果，这样里面带有了最新的version的值，所以也就会保存成功</span>
}
</code></pre>
<p data-nodeid="116691">异步里面调用这个方法也是成功的，因为乐观锁的原理是 Version 变了，我们用最新的对象，也就是最新的 Version 就可以了。</p>
<p data-nodeid="122425">我们设置一个断点看一下 user、u2、u3 在不同的 Session 作用域之后，就变成不同的实例了，如下所示。</p>
<p data-nodeid="122426" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/72/11/Ciqc1F_As4SATk88AAEnxfK1BOo296.png" alt="Drawing 4.png" data-nodeid="122430"></p>


<p data-nodeid="116694">问题分析完了，那么这些内容带给我们哪些思考呢？</p>
<h3 data-nodeid="116695">思考</h3>
<p data-nodeid="116696"><strong data-nodeid="116833">在上面 Demo 中的异步场景下设置 open-in-view 等于 true / false，会对上面的测试结果有影响吗</strong>？</p>
<p data-nodeid="116697">答案是肯定没有影响的，spring.jpa.open-in-view 的本质还是开启 Session，而保持住 Session 的本质还是利用 ThreadLocal，也就是必须为同一个线程的情况下才适用。所以异步场景不受 spring.jpa.open-in-view 控制。</p>
<p data-nodeid="116698"><strong data-nodeid="116838">如果是大量的异步操作 db connection 的持有模式，应该配置成哪一种比较合适？</strong></p>
<p data-nodeid="116699">答案是DELAYED_ACQUISITION_AND_RELEASE_AFTER_TRANSACTION，因为这样可以做到对 db 连接最大的利用率。用的时候就获取，事务提交完就释放，这样就不用关心业务逻辑执行多长时间了。</p>
<h3 data-nodeid="116700">总结</h3>
<p data-nodeid="116701">上面的例子折射出来的是一些 JPA 初学者最容易犯的错误，我们通过前几讲对原理知识的学习，解决了工作中最常见、最容易犯错的，如异步问题和事务问题。其中关键的几个问题你一定要好好思考，尤其是在开发业务代码的时候。</p>
<ol data-nodeid="116702">
<li data-nodeid="116703">
<p data-nodeid="116704">我们的一个请求，开启了几次事务？在什么时机开始的？</p>
</li>
<li data-nodeid="116705">
<p data-nodeid="116706">我们的一个请求，开启了几次 Session？在什么时机开启的？</p>
</li>
<li data-nodeid="116707">
<p data-nodeid="116708">事务和 Session 分别会对实体的状态有什么影响？</p>
</li>
</ol>
<p data-nodeid="116709">上面的几个问题是对一个高级 Java 工程师最基础的要求，如果你想晋级资深开发工程师，还需要知道：</p>
<ol data-nodeid="116710">
<li data-nodeid="116711">
<p data-nodeid="116712">我们的一个请求，对 db 连接池里面的连接持有时间是多久？</p>
</li>
<li data-nodeid="116713">
<p data-nodeid="116714">我们的一个请求，性能指标都有哪些决定因素？</p>
</li>
</ol>
<p data-nodeid="116715">针对以上问题，你可以回过头去文中找答案，并且希望你深入钻研，遇到问题做到心中有数。</p>
<p data-nodeid="116716">本讲内容就到这里了，下一讲我们来聊聊 Lazy 的核心原理和问题。欢迎你对本讲内容提出问题和建议，如果本专栏对你有帮助，就动动手指分享吧。我们下一讲再见。</p>
<blockquote data-nodeid="116717">
<p data-nodeid="116718">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="116864">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **辉：
> 如果没使用乐观锁的话是不是就没这些问题呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 要学会透过现象看本质，乐观锁只是表象。

