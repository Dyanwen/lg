<p data-nodeid="14138">前面几讲我们系统地介绍了 Spring Security 的认证和授权功能，这是该框架为我们提供的最基础、也是最常用的安全性功能。作为阶段性的总结，今天我们就把前面几讲的内容整合在一起，基于 Spring Security 的认证和授权功能保护 Web 应用程序。</p>
<h3 data-nodeid="14139">案例设计和初始化</h3>
<p data-nodeid="14140">在今天的案例中，我们将构建一个简单但完整的小型 Web 应用程序。当合法用户成功登录系统之后，浏览器会跳转到一个系统主页，并展示一些个人健康档案（HealthRecord）数据。</p>
<h4 data-nodeid="14141">案例设计</h4>
<p data-nodeid="14142">这个 Web 应用程序将采用经典的三层架构，即<strong data-nodeid="14222">Web 层、服务层和数据访问层</strong>，因此我们会存在 HealthRecordController、HealthRecordService 以及 HealthRecordRepository，这是一条独立的代码流程，用来完成<strong data-nodeid="14223">系统业务逻辑处理</strong>。</p>
<p data-nodeid="14143">另一方面，本案例的核心功能是<strong data-nodeid="14229">实现自定义的用户认证流程</strong>，所以我们需要构建独立的 UserDetailsService 以及 AuthenticationProvider，这是另一条独立的代码流程。而在这条代码流程中，势必还需要 User 以及 UserRepository 等组件。</p>
<p data-nodeid="14144">我们可以把这两条代码线整合在一起，得到案例的整体设计蓝图，如下图所示：</p>
<p data-nodeid="15766" class=""><img src="https://s0.lgstatic.com/i/image6/M01/46/4A/CioPOWDJyDOAKosUAACCwzt5aFc611.png" alt="Drawing 0.png" data-nodeid="15770"></p>
<div data-nodeid="15767"><p style="text-align:center">案例中的业务代码流程和用户认证流程</p></div>



<h4 data-nodeid="14147">系统初始化</h4>
<p data-nodeid="14148">要想实现上图中的效果，我们需要先对系统进行初始化。这部分工作涉及<strong data-nodeid="14239">领域对象的定义、数据库初始化脚本的整理以及相关依赖组件的引入</strong>。</p>
<p data-nodeid="14149">针对领域对象，我们重点来看如下所示的 User 类定义：</p>
<pre class="lang-java" data-nodeid="14150"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GeneratedValue(strategy = GenerationType.IDENTITY)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Integer id;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String username;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String password;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Enumerated(EnumType.STRING)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> PasswordEncoderType passwordEncoderType;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@OneToMany(mappedBy = "user", fetch = FetchType.EAGER)</span>
	<span class="hljs-keyword">private</span> List&lt;Authority&gt; authorities;
	…
}
</code></pre>
<p data-nodeid="14151">可以看到，这里除了指定主键 id、用户名 username 和密码 password 之外，还包含了一个<strong data-nodeid="14246">加密算法枚举值 EncryptionAlgorithm</strong>。在案例系统中，我们将提供 BCryptPasswordEncoder 和 SCryptPasswordEncoder 这两种可用的密码解密器，你可以通过该枚举值进行设置。</p>
<p data-nodeid="14152">同时，我们在 User 类中还发现了一个 Authority 列表。显然，这个列表用来指定该 User 所具备的权限信息。Authority 类的定义如下所示：</p>
<pre class="lang-java" data-nodeid="14153"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Authority</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GeneratedValue(strategy = GenerationType.IDENTITY)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Integer id;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String name;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@JoinColumn(name = "user")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@ManyToOne</span>
	<span class="hljs-keyword">private</span> User user;
	…
}
</code></pre>
<p data-nodeid="14154">通过定义不难看出 User 和 Authority 之间是<strong data-nodeid="14257">一对多</strong>的关系，这点和 Spring Security 内置的用户权限模型是一致的。我们注意到这里使用了一系列来自 JPA（Java Persistence API，Java 持久化 API）规范的注解来定义领域对象之间的关联关系。关于这些注解的使用方式你可以参考拉勾教育上的<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=490&amp;sid=20-h5Url-0&amp;lgec_type=website&amp;lgec_sign=86228E00A960E2EB44DCA4027393428B&amp;buyFrom=2&amp;pageId=1pz4#/sale&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14255">《Spring Data JPA 原理与实战》专栏</a>进行学习。</p>
<p data-nodeid="14155">基于 User 和 Authority 领域对象，我们也给出创建数据库表的 SQL 定义，如下所示：</p>
<pre class="lang-xml" data-nodeid="14156"><code data-language="xml">CREATE TABLE IF NOT EXISTS `spring_security`.`user` (
&nbsp; `id` INT NOT NULL AUTO_INCREMENT,
&nbsp; `username` VARCHAR(45) NOT NULL,
&nbsp; `password` TEXT NOT NULL,
&nbsp; `password_encoder_type` VARCHAR(45) NOT NULL,
&nbsp; PRIMARY KEY (`id`));
CREATE TABLE IF NOT EXISTS `spring_security`.`authority` (
&nbsp; `id` INT NOT NULL AUTO_INCREMENT,
&nbsp; `name` VARCHAR(45) NOT NULL,
&nbsp; `user` INT NOT NULL,
&nbsp; PRIMARY KEY (`id`));
</code></pre>
<p data-nodeid="14157">在运行系统之前，我们同样也需要初始化数据，对应脚本如下所示：</p>
<pre class="lang-xml" data-nodeid="14158"><code data-language="xml">INSERT IGNORE INTO `spring_security`.`user` (`id`, `username`, `password`, `password_encoder_type`) VALUES ('1', 'jianxiang', '$2a$10$xn3LI/AjqicFYZFruSwve.681477XaVNaUQbr1gioaWPn4t1KsnmG', 'BCRYPT');
INSERT IGNORE INTO `spring_security`.`authority` (`id`, `name`, `user`) VALUES ('1', 'READ', '1');
INSERT IGNORE INTO `spring_security`.`authority` (`id`, `name`, `user`) VALUES ('2', 'WRITE', '1');
INSERT IGNORE INTO `spring_security`.`health_record` (`id`, `username`, `name`, `value`) VALUES ('1', 'jianxiang', 'weight', '70');
INSERT IGNORE INTO `spring_security`.`health_record` (`id`, `username`, `name`, `value`) VALUES ('2', 'jianxiang', 'height', '177');
INSERT IGNORE INTO `spring_security`.`health_record` (`id`, `username`, `name`, `value`) VALUES ('3', 'jianxiang', 'bloodpressure', '70');
INSERT IGNORE INTO `spring_security`.`health_record` (`id`, `username`, `name`, `value`) VALUES ('4', 'jianxiang', 'pulse', '80');
</code></pre>
<p data-nodeid="14159">请注意，这里初始化了一个用户名为 “jianxiang”的用户，同时指定了它的密码为“12345”，加密算法为“BCRYPT”。</p>
<p data-nodeid="14160">现在，领域对象和数据层面的初始化工作已经完成了，接下来我们需要在代码工程的 pom 文件中添加如下所示的 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="14161"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependencies</span>&gt;</span>	    
        <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-data-jpa<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-security<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-thymeleaf<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>mysql<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>mysql-connector-java<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">scope</span>&gt;</span>runtime<span class="hljs-tag">&lt;/<span class="hljs-name">scope</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.security<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-security-test<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">scope</span>&gt;</span>test<span class="hljs-tag">&lt;/<span class="hljs-name">scope</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependencies</span>&gt;</span>
</code></pre>
<p data-nodeid="14162">这些依赖包都是很常见的，相信从包名中你就能明白各依赖包的作用。</p>
<blockquote data-nodeid="14163">
<p data-nodeid="14164">依赖包参考链接：<br>
spring-boot-starter-data-jpa：<a href="https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14268">https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa</a><br>
spring-boot-starter-security：<a href="https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14273">https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security</a><br>
spring-boot-starter-thymeleaf：<a href="https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-thymeleaf?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14278">https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-thymeleaf</a><br>
spring-boot-starter-web：<a href="https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14283">https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web</a><br>
mysql-connector-java：<a href="https://mvnrepository.com/artifact/mysql/mysql-connector-java?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14288">https://mvnrepository.com/artifact/mysql/mysql-connector-java</a><br>
spring-security-test：<a href="https://mvnrepository.com/artifact/org.springframework.security/spring-security-test?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14293">https://mvnrepository.com/artifact/org.springframework.security/spring-security-test</a></p>
</blockquote>
<h3 data-nodeid="14165">实现自定义用户认证</h3>
<p data-nodeid="14166">实现自定义用户认证的过程通常涉及两大部分内容，一方面需要使用 User 和 Authority 对象来完成<strong data-nodeid="14304">定制化的用户管理</strong>，另一方面需要把这个定制化的用户管理<strong data-nodeid="14305">嵌入整个用户认证流程中</strong>。下面我们分别详细分析。</p>
<h4 data-nodeid="14167">实现用户管理</h4>
<p data-nodeid="14168">我们知道在 Spring Security 中，代表用户信息的就是 UserDetails 接口。我们也在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=960#/detail/pc?id=7697&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14310"> 03讲 “认证体系：如何深入理解 Spring Security 的用户认证机制？”</a>中介绍过 UserDetails 接口的具体定义。如果你想实现自定义的用户信息，扩展这个接口即可。实现方式如下所示：</p>
<pre class="lang-java" data-nodeid="14169"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomUserDetails</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetails</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> User user;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">CustomUserDetails</span><span class="hljs-params">(User user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.user = user;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">public</span> Collection&lt;? extends GrantedAuthority&gt; getAuthorities() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user.getAuthorities().stream()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .map(a -&gt; <span class="hljs-keyword">new</span> SimpleGrantedAuthority(a.getName()))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .collect(Collectors.toList());
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getPassword</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user.getPassword();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUsername</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user.getUsername();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonLocked</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isCredentialsNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> User <span class="hljs-title">getUser</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="14170">上述 CustomUserDetails 类实现了 UserDetails 接口中约定的所有需要实现的方法。请注意，这里的 getAuthorities() 方法中，我们将 User 对象中的 Authority 列表转换为了 Spring Security 中代表用户权限的<strong data-nodeid="14317">SimpleGrantedAuthority 列表</strong>。</p>
<p data-nodeid="14171">当然，所有的自定义用户信息和权限信息都是维护在数据库中的，所以为了获取这些信息，我们需要创建数据访问层组件，这个组件就是 UserRepository，定义如下：</p>
<pre class="lang-java" data-nodeid="14172"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>, <span class="hljs-title">Integer</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Optional&lt;User&gt; <span class="hljs-title">findUserByUsername</span><span class="hljs-params">(String username)</span></span>;
}
</code></pre>
<p data-nodeid="14173">这里只是简单扩展了 Spring Data JPA 中的 JpaRepository 接口，并使用<strong data-nodeid="14324">方法名衍生查询机制</strong>定义了根据用户名获取用户信息的 findUserByUsername 方法。</p>
<p data-nodeid="14174">现在，我们已经能够在数据库中维护自定义用户信息，也能够根据这些用户信息获取到 UserDetails 对象，那么接下来要做的事情就是扩展 UserDetailsService。自定义 CustomUserDetailsService 实现如下所示：</p>
<pre class="lang-java" data-nodeid="14175"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomUserDetailsService</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetailsService</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserRepository userRepository;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> CustomUserDetails <span class="hljs-title">loadUserByUsername</span><span class="hljs-params">(String username)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Supplier&lt;UsernameNotFoundException&gt; s =
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;() -&gt; <span class="hljs-keyword">new</span> UsernameNotFoundException(<span class="hljs-string">"Username"</span> + username + <span class="hljs-string">"is invalid!"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; User u = userRepository.findUserByUsername(username).orElseThrow(s);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CustomUserDetails(u);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="14176">这里我们通过 UserRepository 查询数据库来获取 CustomUserDetails 信息，如果传入的用户名没有对应的 CustomUserDetails 则会抛出异常。</p>
<h4 data-nodeid="14177">实现认证流程</h4>
<p data-nodeid="14178">我们再次回顾 AuthenticationProvider 的接口定义，如下所示：</p>
<pre class="lang-java" data-nodeid="14179"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行认证，返回认证结果</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> AuthenticationException</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断是否支持当前的认证对象</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; authentication)</span></span>;
}
</code></pre>
<p data-nodeid="14180">实现自定义认证流程要做的也是实现 AuthenticationProvider 中的这两个方法，而认证过程势必要借助于前面介绍的 CustomUserDetailsService。</p>
<p data-nodeid="14181">我们先来看一下 AuthenticationProvider 接口的实现类 AuthenticationProviderService，如下所示：</p>
<pre class="lang-java" data-nodeid="14182"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AuthenticationProviderService</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> CustomUserDetailsService userDetailsService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> BCryptPasswordEncoder bCryptPasswordEncoder;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> SCryptPasswordEncoder sCryptPasswordEncoder;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span> <span class="hljs-keyword">throws</span> AuthenticationException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = authentication.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = authentication.getCredentials().toString();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据用户名从数据库中获取 CustomUserDetails</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CustomUserDetails user = userDetailsService.loadUserByUsername(username);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据所配置的密码加密算法分别验证用户密码</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">switch</span> (user.getUser().getPasswordEncoderType()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> BCRYPT:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> checkPassword(user, password, bCryptPasswordEncoder);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">case</span> SCRYPT:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> checkPassword(user, password, sCryptPasswordEncoder);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span>&nbsp; BadCredentialsException(<span class="hljs-string">"Bad credentials"</span>);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; aClass)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> UsernamePasswordAuthenticationToken.class.isAssignableFrom(aClass);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> Authentication <span class="hljs-title">checkPassword</span><span class="hljs-params">(CustomUserDetails user, String rawPassword, PasswordEncoder encoder)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (encoder.matches(rawPassword, user.getPassword())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UsernamePasswordAuthenticationToken(user.getUsername(), user.getPassword(), user.getAuthorities());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BadCredentialsException(<span class="hljs-string">"Bad credentials"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="14183">AuthenticationProviderService 类虽然看起来比较长，但代码基本都是自解释的。我们首先通过 CustomUserDetailsService 从数据库中获取用户信息并构造成 CustomUserDetails 对象。然后，根据指定的密码加密器对用户密码进行验证，如果验证通过则构建一个 UsernamePasswordAuthenticationToken 对象并返回，反之直接抛出 BadCredentialsException 异常。而在 supports() 方法中指定的就是这个目标 UsernamePasswordAuthenticationToken 对象。</p>
<h4 data-nodeid="14184">安全配置</h4>
<p data-nodeid="14185">最后，我们要做的就是通过 Spring Security 提供的配置体系将前面介绍的所有内容串联起来，如下所示：</p>
<pre class="lang-java" data-nodeid="14186"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SecurityConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationProviderService authenticationProvider;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> BCryptPasswordEncoder <span class="hljs-title">bCryptPasswordEncoder</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> BCryptPasswordEncoder();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> SCryptPasswordEncoder <span class="hljs-title">sCryptPasswordEncoder</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> SCryptPasswordEncoder();
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.authenticationProvider(authenticationProvider);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.formLogin()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .defaultSuccessUrl(<span class="hljs-string">"/healthrecord"</span>, <span class="hljs-keyword">true</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.authorizeRequests().anyRequest().authenticated();
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="14187">这里注入了已经构建完成的 AuthenticationProviderService，并初始化了两个密码加密器 BCryptPasswordEncoder 和 SCryptPasswordEncoder。最后，我们覆写了 WebSecurityConfigurerAdapter 配置适配器类中的 configure() 方法，并指定用户登录成功后将跳转到"/main"路径所指定的页面。</p>
<p data-nodeid="14188">对应的，我们需要构建如下所示的 MainController 类来指定"/main"路径，并展示业务数据的获取过程，如下所示：</p>
<pre class="lang-java" data-nodeid="14189"><code data-language="java"><span class="hljs-meta">@Controller</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HealthRecordController</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> HealthRecordService healthRecordService;
&nbsp;&nbsp; &nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@GetMapping("/healthrecord")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">main</span><span class="hljs-params">(Authentication a, Model model)</span> </span>{
&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;String userName = a.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; model.addAttribute(<span class="hljs-string">"username"</span>, userName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; model.addAttribute(<span class="hljs-string">"healthRecords"</span>, healthRecordService.getHealthRecordsByUsername(userName));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-string">"health_record.html"</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="14190">我们通过 Authentication 对象获取了认证用户信息，同时通过 HealthRecordService 获取了健康档案信息。关于 HealthRecordService 的实现逻辑不是今天内容的重点，你可以参考案例源码进行学习：<a href="https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSecurityBasicDemo?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14347">https://github.com/lagouEdAnna/SpringSecurity-jianxiang/tree/main/SpringSecurityBasicDemo</a>。</p>
<p data-nodeid="14191">请注意，这里所指定的 health_record.html 位于 resources/templates 目录下，该页面基于 thymeleaf 模板引擎构建，如下所示：</p>
<pre class="lang-xml" data-nodeid="14192"><code data-language="xml"><span class="hljs-meta">&lt;!DOCTYPE <span class="hljs-meta-keyword">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">html</span> <span class="hljs-attr">lang</span>=<span class="hljs-string">"en"</span> <span class="hljs-attr">xmlns:th</span>=<span class="hljs-string">"http://www.thymeleaf.org"</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">meta</span> <span class="hljs-attr">charset</span>=<span class="hljs-string">"UTF-8"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>健康档案<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">h2</span> <span class="hljs-attr">th:text</span>=<span class="hljs-string">"'登录用户：' + ${username}"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"/logout"</span>&gt;</span>退出登录<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>个人健康档案:<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">table</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">thead</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span> 健康指标名称 <span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span> 健康指标值 <span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">thead</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">tbody</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">tr</span> <span class="hljs-attr">th:if</span>=<span class="hljs-string">"${healthRecords.empty}"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">td</span> <span class="hljs-attr">colspan</span>=<span class="hljs-string">"2"</span>&gt;</span> 无健康指标 <span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">tr</span> <span class="hljs-attr">th:each</span>=<span class="hljs-string">"healthRecord : ${healthRecords}"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">th:text</span>=<span class="hljs-string">"${healthRecord.name}"</span>&gt;</span> 健康指标名称 <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">th:text</span>=<span class="hljs-string">"${healthRecord.value}"</span>&gt;</span> 健康指标值 <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">tbody</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">table</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span>
</code></pre>
<p data-nodeid="14193">这里我们从 Model 对象中获取了认证用户信息以及健康档案信息，并渲染在页面上。</p>
<h3 data-nodeid="14194">案例演示</h3>
<p data-nodeid="14195">现在，让我们启动 Spring Boot 应用程序，并访问<a href="http://localhost:8080?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="14357">http://localhost:8080</a>端点。因为访问系统的任何端点都需要认证，所以 Spring Security 会自动跳转到如下所示的登录界面：</p>
<p data-nodeid="16702" class=""><img src="https://s0.lgstatic.com/i/image6/M01/46/4A/CioPOWDJyFCAWC1SAAA5aEhV0-A820.png" alt="Drawing 1.png" data-nodeid="16706"></p>
<div data-nodeid="16703"><p style="text-align:center">用户登录界面</p></div>



<p data-nodeid="14198">我们分别输入用户名“jianxiang”和密码“12345”，系统就会跳转到健康档案主页：</p>
<p data-nodeid="17642" class=""><img src="https://s0.lgstatic.com/i/image6/M01/46/4A/CioPOWDJyFeAUaWEAAB7TB8wdKw208.png" alt="Drawing 2.png" data-nodeid="17646"></p>
<div data-nodeid="17643"><p style="text-align:center">健康档案主页</p></div>



<p data-nodeid="14201">在这个主页中，我们正确获取了登录用户的用户名，并展示了个人健康档案信息。这个结果也证实了自定义用户认证体系的正确性。你可以根据示例代码做一些尝试。</p>
<h3 data-nodeid="14202">小结与预告</h3>
<p data-nodeid="14203">这一讲我们动手实践了“利用 Spring Security 基础功能保护 Web 应用程序”。综合第 2 讲到 6 讲中的核心知识点，我们设计了一个简单而又完整的案例，并通过构建用户管理和认证流程讲解了实现自定义用户认证机制的过程。</p>
<p data-nodeid="14204">本讲内容总结如下：</p>
<p data-nodeid="18117" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/46/41/Cgp9HWDJyF6AAUtVAACuoKgyiho485.png" alt="Drawing 3.png" data-nodeid="18120"></p>

<p data-nodeid="14206">最后给你留一道思考题：在 Spring Security 中，实现一套自定义的用户认证体系需要哪些开发步骤？</p>
<p data-nodeid="14207">介绍完今天的案例之后，从下一讲开始，我们将进入到 Spring Security 高级主题的学习，首先要引出的就是 Spring Security 中应用非常广泛的过滤器机制。</p>

---

### 精选评论

##### iLeGeND：
> 能不能讲点前后端分离的案例，单体项目的

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 只要通过HTTP协议去发起请求就会触发Spring Security的安全机制，这个跟前后端分离本质上没什么关系。前后端分离的时候，统一通过HTTP协议发起请求并获取JSON数据之后，再通过返回的JSON数据来控制前端界面跳转，这时候就跟后台没关系系

##### iLeGeND：
> 现在都是前后端分离的项目，这种案例还有什么意义吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 只要通过HTTP协议去发起请求就会触发Spring Security的安全机制，这个跟前后端分离本质上没什么关系。前后端分离的时候，统一通过HTTP协议发起请求并获取JSON数据之后，再通过返回的JSON数据来控制前端界面跳转，这时候就跟后台没关系系

