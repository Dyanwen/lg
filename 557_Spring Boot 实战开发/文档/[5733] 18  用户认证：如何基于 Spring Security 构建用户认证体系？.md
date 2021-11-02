<p data-nodeid="172639" class="">在 17 讲中，我们梳理了 Web 应用程序的安全性需求，并引出了 Spring Security 这款 Spring 家族中专门用于处理安全性需求的开发框架，同时也明确了认证和授权是安全性框架的核心功能。</p>
<p data-nodeid="172640">这一讲我们将先讨论与认证相关的话题，并给出 Spring Security 认证机制及其使用方法。因为 Spring Security 是日常开发过程中的基础组件，所以我们也会对如何实现数据加解密的过程做一些展开。</p>
<p data-nodeid="172641">在 Spring Boot 中整合 Spring Security 框架的方式非常简单，我们只需要在 pom 文件中引入 spring-boot-starter-security 依赖即可，这与以往需要提供很多配置才能与 Spring Security 完成集成的开发过程不同，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="172642"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-security<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<hr data-nodeid="172643">
<p data-nodeid="172644"><strong data-nodeid="172740">请注意，只要我们在代码工程中添加了上述依赖，包含在该工程中的所有 HTTP 端点都将被保护起来。</strong></p>
<p data-nodeid="172645">例如，在 SpringCSS 案例的 account-service 中，我们知道存在一个 AccountController ，且它暴露了一个“accounts/ /{accountId}”端点。现在，我们启动 account-service 服务并访问上述端点，弹出了如下图所示的界面内容：</p>
<p data-nodeid="172646"><img src="https://s0.lgstatic.com/i/image/M00/8D/FA/CgqCHmABN02AZfgTAAAR0uEiE5U744.png" alt="Drawing 0.png" data-nodeid="172744"></p>
<div data-nodeid="172647"><p style="text-align:center">添加 Spring Security 之后自动出现的登录界面</p></div>
<p data-nodeid="172648">同时，在系统的启动控制台日志中，我们发现了如下所示的新的日志信息。</p>
<pre class="lang-xml" data-nodeid="172649"><code data-language="xml">Using generated security password: 17bbf7c4-456a-48f5-a12e-a680066c8f80
</code></pre>
<p data-nodeid="172650">在这里可以看到，Spring Security 为我们自动生成了一个密码，我们可以基于“user”这个账号及上述密码登录这个界面，抽空你也可以尝试下。</p>
<p data-nodeid="172651">如果我们使用了 Postman 可视化 HTTP 请求工具，可以设置授权类型为“Basic Auth”并输入对应的用户名和密码完成对 HTTP 端点的访问，设置界面如下图所示：</p>
<p data-nodeid="172652"><img src="https://s0.lgstatic.com/i/image/M00/8D/FA/CgqCHmABN1uAC5vsAAA3VhGEBJY812.png" alt="Drawing 1.png" data-nodeid="172750"></p>
<div data-nodeid="172653"><p style="text-align:center">使用 Postman 来完成认证信息的设置</p></div>
<p data-nodeid="172654">事实上，在引入 spring-boot-starter-security 依赖之后，Spring Security 会默认创建一个用户名为“user”的账号。很显然，每次启动应用时，通过 Spring Security 自动生成的密码都会有所变化，因此它不适合作为一种正式的应用方法。</p>
<p data-nodeid="172655">如果我们想设置登录账号和密码，最简单的方式是通过配置文件。例如，我们可以在 account-service 的 application.yml 文件中添加如下代码所示的配置项：</p>
<pre class="lang-xml" data-nodeid="172656"><code data-language="xml">spring:
&nbsp; security:
&nbsp;&nbsp;&nbsp; user:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; name: springcss
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; password: springcss_password
</code></pre>
<p data-nodeid="172657">重启 account-service 服务后，我们就可以使用上述用户名和密码完成登录。</p>
<p data-nodeid="172658">虽然基于配置文件的用户信息存储方案简单且直接，但是显然缺乏灵活性，因此 Spring Security 为我们提供了多种存储和管理用户认证信息的方案，我们一起来看一下。</p>
<h3 data-nodeid="172659">配置 Spring Security</h3>
<p data-nodeid="172660">在 SpringSecurity 中，初始化用户信息所依赖的配置类是 WebSecurityConfigurer 接口，该接口实际上是一个空接口，继承了更为基础的 SecurityConfigurer 接口。</p>
<p data-nodeid="172661">在日常开发中，我们往往不需要自己实现这个接口，而是使用 WebSecurityConfigurerAdapter 类简化该配置类的使用方式。比如我们可以通过继承 WebSecurityConfigurerAdapter 类并且覆写其中的 configure(AuthenticationManagerBuilder auth) 的方法完成配置工作。</p>
<p data-nodeid="172662">关于 WebSecurityConfigurer 配置类，首先我们需要明确配置的内容。实际上，初始化所使用的用户信息非常简单，只需要指定用户名（Username）、密码（Password）和角色（Role）这三项数据即可。</p>
<p data-nodeid="172663">在 WebSecurityConfigurer 类中，使用 AuthenticationManagerBuilder 类创建一个 AuthenticationManager 就能够轻松实现基于内存、LADP 和 JDBC 的验证。</p>
<p data-nodeid="172664">接下来，我们就围绕 AuthenticationManagerBuilder 提供的功能实现多种用户信息存储方案。</p>
<h4 data-nodeid="172665">使用基于内存的用户信息存储方案</h4>
<p data-nodeid="172666">我们先来看看如何使用 AuthenticationManagerBuilder 完成基于内存的用户信息存储方案。</p>
<p data-nodeid="172667">实现方法是调用 AuthenticationManagerBuilder 的 inMemoryAuthentication 方法，示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="172668"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder builder)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; builder.inMemoryAuthentication()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"springcss_user"</span>).password(<span class="hljs-string">"password1"</span>)
	.roles(<span class="hljs-string">"USER"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;.and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"springcss_admin"</span>).password(<span class="hljs-string">"password2"</span>)
	.roles(<span class="hljs-string">"USER"</span>, <span class="hljs-string">"ADMIN"</span>);
}
</code></pre>
<p data-nodeid="172669">从上面的代码中，我们看到系统中存在"springcss _user"和"springcss _admin"这两个用户，其密码分别是"password1"和"password2"，分别代表着普通用户 USER 及管理员 ADMIN 这两个角色。</p>
<p data-nodeid="172670">在 AuthenticationManagerBuilder 中，上述 inMemoryAuthentication 的方法的实现过程如下代码所示：</p>
<pre class="lang-java" data-nodeid="172671"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> InMemoryUserDetailsManagerConfigurer&lt;AuthenticationManagerBuilder&gt; <span class="hljs-title">inMemoryAuthentication</span><span class="hljs-params">()</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> apply(<span class="hljs-keyword">new</span> InMemoryUserDetailsManagerConfigurer&lt;&gt;());
}
</code></pre>
<p data-nodeid="172672">这里的 InMemoryUserDetailsManagerConfigurer 内部又使用到了 InMemoryUserDetailsManager 对象，而通过深入该类，我们可以获取 Spring Security 中与用户认证相关的一大批核心对象，它们之间的关系如下图所示：</p>
<p data-nodeid="172858" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image2/M01/05/EE/Cip5yGABYdOAHvU6AADbh8te5-8316.png" alt="图片3 (1).png" data-nodeid="172866"></p>
<div data-nodeid="172859"><p style="text-align:center">Spring Security 中用户认证相关类结构图</p></div>


<p data-nodeid="172675">首先，我们来看上图中代表用户详细信息的 UserDetails 接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172676"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetails</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Serializable</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取该用户的权限信息</span>
&nbsp;&nbsp;&nbsp; Collection&lt;? extends GrantedAuthority&gt; getAuthorities();
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取密码</span>
	<span class="hljs-function">String <span class="hljs-title">getPassword</span><span class="hljs-params">()</span></span>;
	<span class="hljs-comment">//获取用户名</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getUsername</span><span class="hljs-params">()</span></span>;
	<span class="hljs-comment">//判断该账户是否已失效</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonExpired</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该账户是否已被锁定</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonLocked</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该账户的凭证信息是否已失效</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isCredentialsNonExpired</span><span class="hljs-params">()</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该用户是否可用</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="172677">在上述代码中，我们发现 UserDetails 存在一个子接口 MutableUserDetails，从命名上不难看出，后者是一个可变的 UserDetails，而可变的内容就是密码。</p>
<p data-nodeid="172678">关于 MutableUserDetails 接口的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="172679"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">MutableUserDetails</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDetails</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置密码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">setPassword</span><span class="hljs-params">(String password)</span></span>;
}
</code></pre>
<p data-nodeid="172680">在 Spring Security 中，针对 UserDetails 还存在一个专门的 UserDetailsService，该接口专门用来管理 UserDetails，它的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="172681"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetailsService</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据用户名获取用户信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">UserDetails <span class="hljs-title">loadUserByUsername</span><span class="hljs-params">(String username)</span> <span class="hljs-keyword">throws</span> UsernameNotFoundException</span>;
}
</code></pre>
<p data-nodeid="172682">而 UserDetailsManager 继承了 UserDetailsService，并提供了一批针对 UserDetails 的操作接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172683"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetailsManager</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDetailsService</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">createUser</span><span class="hljs-params">(UserDetails user)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//更新用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">updateUser</span><span class="hljs-params">(UserDetails user)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//删除用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteUser</span><span class="hljs-params">(String username)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//修改密码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">changePassword</span><span class="hljs-params">(String oldPassword, String newPassword)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断指定用户名的用户是否存在</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">userExists</span><span class="hljs-params">(String username)</span></span>;
}
</code></pre>
<p data-nodeid="172684">介绍完 UserDetailsManager 后，我们再回到 InMemoryUserDetailsManager 类，它实现了 UserDetailsManager 接口中的所有方法，这些方法主要用来对用户信息进行维护，从而形成一条代码支线。</p>
<p data-nodeid="172685">为了完成用户信息的配置，还存在另外一条代码支线，即 UserDetailsManagerConfigurer。该类维护了一个 UserDetails 列表，并提供了一组 withUser 方法完成用户信息的初始化，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172686"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;UserDetails&gt; users = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> C <span class="hljs-title">withUser</span><span class="hljs-params">(UserDetails userDetails)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.users.add(userDetails);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (C) <span class="hljs-keyword">this</span>;
}
</code></pre>
<p data-nodeid="172687">从上述代码中，我们看到 withUser 方法返回的是一个 UserDetailsBuilder 对象，通过该对象可以实现类似 .withUser("springcss_user").password("password1").roles("USER") 这样的链式语法，从而完成用户信息的设置。</p>
<p data-nodeid="172688"><strong data-nodeid="172818">请注意，这里的 .roles() 方法实际上是 .authorities() 方法的一种简写，因为 Spring Security 会在每个角色名称前自动添加“ROLE_”前缀</strong>，我们可以通过如下所示的代码实现同样的功能：</p>
<pre class="lang-java" data-nodeid="172689"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder builder)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; builder.inMemoryAuthentication()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"springcss_user"</span>).password(<span class="hljs-string">"password1"</span>)
	.authorities(<span class="hljs-string">"ROLE_USER"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;.and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"springcss_admin"</span>).password(<span class="hljs-string">"password2"</span>)
	.authorities(<span class="hljs-string">"ROLE_USER"</span>, <span class="hljs-string">"ROLE_ADMIN"</span>);
}
</code></pre>
<p data-nodeid="172690">我们可以看到，基于内存的用户信息存储方案也比较简单，但是由于用户信息写死在代码中，因此同样缺乏灵活性。</p>
<p data-nodeid="172691">接下来我们将引出另一种更为常见的用户信息存储方案——数据库存储。</p>
<h4 data-nodeid="172692">使用基于数据库的用户信息存储方案</h4>
<p data-nodeid="172693">既然是将用户信息存储在数据库中，我们势必需要创建表结构。因此，在 Spring Security 的源文件中，我们可以找到对应的 SQL 语句，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172694"><code data-language="java"><span class="hljs-function">create table <span class="hljs-title">users</span><span class="hljs-params">(username varchar_ignorecase(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span> primary key,password <span class="hljs-title">varchar_ignorecase</span><span class="hljs-params">(<span class="hljs-number">500</span>)</span> not <span class="hljs-keyword">null</span>,enabled <span class="hljs-keyword">boolean</span> not <span class="hljs-keyword">null</span>)</span>;
&nbsp;
<span class="hljs-function">create table <span class="hljs-title">authorities</span> <span class="hljs-params">(username varchar_ignorecase(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span>,authority <span class="hljs-title">varchar_ignorecase</span><span class="hljs-params">(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span>,constraint fk_authorities_users foreign <span class="hljs-title">key</span><span class="hljs-params">(username)</span> references <span class="hljs-title">users</span><span class="hljs-params">(username)</span>)</span>;
&nbsp;
<span class="hljs-function">create unique index ix_auth_username on <span class="hljs-title">authorities</span> <span class="hljs-params">(username,authority)</span></span>;
</code></pre>
<p data-nodeid="172695">一旦在自己的数据库中创建了这两张表，且添加了相应数据，我们就可以直接注入一个 DataSource 对象查询用户数据，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172696"><code data-language="java"><span class="hljs-meta">@Autowired</span>
DataSource dataSource;
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.jdbcAuthentication().dataSource(dataSource)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .usersByUsernameQuery(<span class="hljs-string">"select username, password, enabled from Users "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authoritiesByUsernameQuery(<span class="hljs-string">"select username, authority from UserAuthorities "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .passwordEncoder(<span class="hljs-keyword">new</span> BCryptPasswordEncoder());
}
</code></pre>
<p data-nodeid="172697">这里使用了 AuthenticationManagerBuilder 的 jdbcAuthentication 方法配置数据库认证方式，而内部则使用了 JdbcUserDetailsManager 工具类。</p>
<p data-nodeid="172698">围绕 JdbcUserDetailsManager 整条代码链路的类层结构与 InMemoryUserDetailsManager 非常一致，在该类中定义了各种用户数据库查询的 SQL 语句，以及使用 JdbcTemplate 完成数据库访问的具体实现方法。这里我们不再具体展开，你可以对照前面给出的 InMemoryUserDetailsManager 类层结构图进行分析。</p>
<hr data-nodeid="172699">
<p data-nodeid="172700"><strong data-nodeid="172829">注意，在上述方法中，通过 jdbcAuthentication() 方法验证用户信息时，我们必须集成加密机制，即使用 passwordEncoder() 方法嵌入一个 PasswordEncoder 接口的实现类。</strong></p>
<p data-nodeid="172701">在 Spring Security 中，PasswordEncoder 接口代表一种密码编码器，定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="172702"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PasswordEncoder</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//对原始密码进行编码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//对提交的原始密码与库中存储的加密密码进行比对</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(CharSequence rawPassword, String encodedPassword)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断加密密码是否需要再次进行加密，默认返回false</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">upgradeEncoding</span><span class="hljs-params">(String encodedPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="172703">Spring Security 中内置了一大批 PasswordEncoder 接口的实现类，如下图所示：</p>
<p data-nodeid="172704"><img src="https://s0.lgstatic.com/i/image/M00/8D/FB/CgqCHmABN4qACniMAAB2uCwQA6M741.png" alt="Drawing 3.png" data-nodeid="172834"></p>
<div data-nodeid="172705"><p style="text-align:center">Spring Security 中的 PasswordEncoder 实现类</p></div>
<p data-nodeid="172706">上图中，比较常用的算法如 SHA-256 算法的 StandardPasswordEncoder、bcrypt 强哈希算法的 BCryptPasswordEncoder 等。而在实际案例中，我们使用的是 BCryptPasswordEncoder，它的 encode 方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="172707"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String salt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (random != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; salt = BCrypt.gensalt(version.getVersion(), strength, random);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; salt = BCrypt.gensalt(version.getVersion(), strength);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> BCrypt.hashpw(rawPassword.toString(), salt);
}
</code></pre>
<p data-nodeid="172708">可以看到，上述 encode 方法执行了两个步骤，第一步是生成盐，第二步是根据盐和明文密码生成最终的密文密码。</p>
<h3 data-nodeid="172709">实现定制化用户认证方案</h3>
<p data-nodeid="172710">通过前面内容的分析，我们明确了用户信息存储的实现过程实际上是完全可定制化，而 Spring Security 所做的工作只是把常见、符合一般业务场景的实现方式嵌入框架中。如果存在特殊的场景，开发人员完全可以通过自定义用户信息存储方案进行实现。</p>
<p data-nodeid="172711">在前面的内容中，我们介绍了 UserDetails 接口代表用户详细信息，而 UserDetailsService 接口负责对 UserDetails 进行各种操作 。因此，实现定制化用户认证方案的关键是实现 UserDetails 和 UserDetailsService 这两个接口。</p>
<h4 data-nodeid="172712">扩展 UserDetails</h4>
<p data-nodeid="172713">扩展 UserDetails 的方法的实质是直接实现该接口，例如我们可以构建如下所示的 SpringCssUser 类：</p>
<pre class="lang-java" data-nodeid="172714"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringCssUser</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetails</span> </span>{
&nbsp;
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> serialVersionUID = <span class="hljs-number">1L</span>;
&nbsp; <span class="hljs-keyword">private</span> Long id;
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String username;
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String password;
&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String phoneNumber;
&nbsp; <span class="hljs-comment">//省略getter/setter</span>

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUsername</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> username;
&nbsp;&nbsp;&nbsp; }

&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getPassword</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> password;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-keyword">public</span> Collection&lt;? extends GrantedAuthority&gt; getAuthorities() {
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Arrays.asList(<span class="hljs-keyword">new</span> SimpleGrantedAuthority(<span class="hljs-string">"ROLE_USER"</span>));
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonLocked</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isCredentialsNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
}
</code></pre>
<p data-nodeid="172715">显然，这里我们使用了一种更简单的方法满足 UserDetails 中各个接口的实现需求。一旦我们构建了一个 SpringCssUser 类，就可以创建对应的表结构存储类中所定义的字段。同时，我们也可以基于 Spring Data JPA 创建一个自定义的 Repository，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172716"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SpringCssUserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">SpringCssUser</span>, <span class="hljs-title">Long</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-function">SpringCssUser <span class="hljs-title">findByUsername</span><span class="hljs-params">(String username)</span></span>;
}
</code></pre>
<p data-nodeid="172717">SpringCssUserRepository 扩展了 CrudRepository 接口，并提供了一个方法名衍生查询 findByUsername。</p>
<p data-nodeid="172718">关于 Spring Data JPA 的使用方法，你还可以回顾《ORM 集成：如何使用 Spring Data JPA 访问关系型数据库？》。</p>
<h4 data-nodeid="172719">扩展 UserDetailsService</h4>
<p data-nodeid="172720">接着，我们来实现 UserDetailsService 接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="172721"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringCssUserDetailsService</span> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetailsService</span> </span>{
	&nbsp;
	<span class="hljs-meta">@Autowired</span>
&nbsp; <span class="hljs-keyword">private</span> SpringCssUserRepository repository;
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> UserDetails <span class="hljs-title">loadUserByUsername</span><span class="hljs-params">(String username)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> UsernameNotFoundException </span>{
&nbsp;&nbsp;&nbsp; SpringCssUser user = repository.findByUsername(username);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (user != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UsernameNotFoundException(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"SpringCSS User '"</span> + username + <span class="hljs-string">"' not found"</span>);
&nbsp; }
}
</code></pre>
<p data-nodeid="172722">在 UserDetailsService 接口中，我们只需要实现 loadUserByUsername 方法就行。因此，我们可以基于 SpringCssUserRepository 的 findByUsername 方法，再根据用户名从数据库中查询数据。</p>
<h4 data-nodeid="172723">整合定制化配置</h4>
<p data-nodeid="172724">最后，我们再次回到 SpringCssSecurityConfig 类。</p>
<p data-nodeid="172725">这次我们将使用自定义的 SpringCssUserDetailsService 完成用户信息的存储和查询，此时我们只需要对配置策略做一些调整，调整后的完整 SpringCssSecurityConfig 类如下代码所示：</p>
<pre class="lang-java" data-nodeid="172726"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringCssSecurityConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; SpringCssUserDetailsService springCssUserDetailsService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;  &nbsp;auth.userDetailsService(springCssUserDetailsService);
	}
}
</code></pre>
<p data-nodeid="172727">这里我们注入了 SpringCssUserDetailsService，并将其添加到 AuthenticationManagerBuilder 中，这样 AuthenticationManagerBuilder 将基于自定义的 SpringCssUserDetailsService 完成 UserDetails 的创建和管理。</p>
<h3 data-nodeid="172728">小结与预告</h3>
<p data-nodeid="172729">这一讲我们详细介绍了如何使用 Spring Security 构建用户认证体系的系统方法。</p>
<p data-nodeid="172730">一方面，我们可以分别基于内存和数据库方案存储用户信息，这两种方案都是 Spring Security 内置的。另一方面，我们可以通过扩展 UserDetails 接口的方式实现定制化用户的认证方案。同时，为了方便你理解和掌握这部分内容，我们还梳理了与用户认证相关的核心类。</p>
<p data-nodeid="172731">介绍完用户认证信息后，19 讲我们将介绍如何基于 Spring Security 确保 Web 请求的安全访问。</p>
<p data-nodeid="172732">这里给你留一道思考题：你能简要描述 Spring Security 中与用户认证相关的核心类及其它们之间的关联关系吗？欢迎你在留言区进行互动。</p>
<p data-nodeid="172733" class="">另外，如果你觉得本专栏有价值，欢迎分享给更多好友哦～</p>

---

### 精选评论


