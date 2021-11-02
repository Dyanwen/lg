<p data-nodeid="27132">上一讲中，我们引入了 Spring Security 框架，并梳理了它的各项核心功能。从今天开始，我们就对这些功能一一展开讲解，首先要讨论的就是<strong data-nodeid="27142">用户认证功能</strong>。用户认证涉及用户账户体系的构建，也是<strong data-nodeid="27143">实现授权管理的前提</strong>。在 Spring Security 中，实现用户认证的方式有很多，下面我们就结合框架提供的配置体系进行梳理。</p>


<h3 data-nodeid="26348">Spring Security 配置体系</h3>
<p data-nodeid="26349">在 Spring Security 中，因为认证和授权等功能通常都不止有一种实现方法，所以框架开发了一套完整的配置体系来对这些功能进行灵活设置。开发人员在使用认证和授权等功能时就依赖于如何合理利用和扩展这套配置体系。</p>
<p data-nodeid="26350">例如，针对用户账户存储这个切入点，就可以设计出多种不同的策略。我们可以把用户名和密码保存在内存中，作为一种轻量级的实现方式。更常见的，也可以把这些认证信息存储在关系型数据库中。当然，如果我们使用了 LDAP 协议，那么文件系统也是一种不错的存储媒介。</p>
<p data-nodeid="26351">显然，针对这些可选择的实现方式，需要为开发人员提供一种机制以便他们能够根据自身的需求进行灵活的设置，这就是配置体系的作用。</p>
<p data-nodeid="26352">同时，你应该也注意到了，在上一讲的示例中，我们没有进行任何的配置也能让 Spring Security 发挥作用，这就说明框架内部的功能采用了<strong data-nodeid="26442">特定的默认配置</strong>。就用户认证这一场景而言，Spring Security 内部就初始化了一个默认的用户名“user”并且在应用程序启动时自动生成一个密码。当然，通过这种方式自动生成的密码在每次启动应用时都会发生变化，并不适合面向正式的应用。</p>
<p data-nodeid="26353">我们可以通过翻阅框架的源代码（<a href="https://github.com/spring-projects/spring-security?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="26446">https://github.com/spring-projects/spring-security</a>）来进一步理解 Spring Security 中的一些默认配置。在 Spring Security 中，初始化用户信息依赖的配置类是 WebSecurityConfigurer 接口，该接口实际上是一个空接口，继承了更为基础的 SecurityConfigurer 接口。</p>
<p data-nodeid="26354">在日常开发中，通常不需要我们自己实现这个接口，而是<strong data-nodeid="26453">使用 WebSecurityConfigurerAdapter 类</strong>来简化该配置类的使用方式。而在 WebSecurityConfigurerAdapter 中我们发现了如下所示的 configure 方法：</p>
<pre class="lang-java" data-nodeid="26355"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authorizeRequests()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .anyRequest().authenticated()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .formLogin().and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .httpBasic();
}
</code></pre>
<p data-nodeid="26356">上述代码就是 Spring Security 中作用于用户认证和访问授权的默认实现，这里用到了多个常见的配置方法。再次回想上一讲中我们讲到的，一旦在代码类路径中引入 Spring Security 框架之后，访问任何端点时就会弹出一个登录界面用来完成用户认证。<strong data-nodeid="26459">认证是授权的前置流程</strong>，认证结束之后就可以进入到授权环节。</p>
<p data-nodeid="26357">结合这些配置方法，我们来简单分析一下这种默认效果是如何实现的：</p>
<ul data-nodeid="26358">
<li data-nodeid="26359">
<p data-nodeid="26360">首先，通过 HttpSecurity 类的 authorizeRequests() 方法对所有访问 HTTP 端点的 HttpServletRequest 进行限制；</p>
</li>
<li data-nodeid="26361">
<p data-nodeid="26362">然后，anyRequest().authenticated() 语句指定了对于所有请求都需要执行认证，也就是说没有通过认证的用户就无法访问任何端点；</p>
</li>
<li data-nodeid="26363">
<p data-nodeid="26364">接着，formLogin() 语句用于指定使用表单登录作为认证方式，也就是会弹出一个登录界面；</p>
</li>
<li data-nodeid="26365">
<p data-nodeid="26366">最后，httpBasic() 语句表示可以使用 HTTP 基础认证（Basic Authentication）方法来完成认证。</p>
</li>
</ul>
<p data-nodeid="26367">在日常开发过程中，我们可以继承 WebSecurityConfigurerAdapter 类并且覆写上述的 configure() 方法来完成配置工作。而在 Spring Security 中，存在一批类似于 WebSecurityConfigurerAdapter 的配置类。</p>
<p data-nodeid="26368"><strong data-nodeid="26470">配置体系是开发人员使用 Spring Security 框架的主要手段之一</strong>，关于配置体系的讨论会贯穿我们整个专栏的始终。随着内容深度的演进，Spring Security 所提供的全面而灵活的配置功能也将一一展现在你的面前。</p>
<h3 data-nodeid="26369">实现 HTTP 基础认证和表单登录认证</h3>
<p data-nodeid="26370">在上文中，我们提到了 httpBasic() 和 formLogin() 这两种用于控制用户认证的实现手段，分别代表了<strong data-nodeid="26477">HTTP 基础认证和表单登录认证</strong>。在构建 Web 应用程序时，我们也可以在 Spring Security 提供的认证机制的基础上进行扩展，以满足日常开发需求。</p>
<h4 data-nodeid="26371">HTTP 基础认证</h4>
<p data-nodeid="26372">HTTP 基础认证的原理比较简单，只需<strong data-nodeid="26484">通过 HTTP 协议的消息头携带用户名和密码</strong>进行登录验证。在上一讲中，我们已经通过浏览器简单验证了用户登录操作。今天，我们将引入 Postman 这款可视化的 HTTP 请求工具来对登录的请求和响应过程做进一步分析。</p>
<p data-nodeid="26373">在 Postman 中，我们直接访问<a href="http://localhost:8080/hello?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="26488">http://localhost:8080/hello</a>端点，会得到如下所示的响应：</p>
<pre class="lang-xml" data-nodeid="26374"><code data-language="xml">{
&nbsp;&nbsp;&nbsp;&nbsp;"timestamp":&nbsp;"2021-02-08T03:45:21.512+00:00",
&nbsp;&nbsp;&nbsp;&nbsp;"status":&nbsp;401,
&nbsp;&nbsp;&nbsp;&nbsp;"error":&nbsp;"Unauthorized",
&nbsp;&nbsp;&nbsp;&nbsp;"message":&nbsp;"",
&nbsp;&nbsp;&nbsp;&nbsp;"path":&nbsp;"/hello"
}
</code></pre>
<p data-nodeid="26375">显然，响应码 401 告诉我们没有访问该地址的权限。同时，在响应中出现了一个“WWW-Authenticate”消息头，其值为“Basic realm="Realm"”，这里的 Realm 表示 Web 服务器中受保护资源的安全域。</p>
<p data-nodeid="26376">现在，让我们来执行 HTTP 基础认证，可以通过设置认证类型为“Basic Auth”并输入对应的用户名和密码来完成对 HTTP 端点的访问，设置界面如下所示：</p>
<p data-nodeid="28183" class=""><img src="https://s0.lgstatic.com/i/image6/M01/43/A3/CioPOWC5_duASv7jAACBgx8x8WU605.png" alt="Drawing 0.png" data-nodeid="28187"></p>
<div data-nodeid="28184"><p style="text-align:center">使用 Postman 完成 HTTP 基础认证信息的设置</p></div>



<p data-nodeid="30306" class="">现在查看 HTTP 请求，可以看到 Request Header 中添加了 <a href="https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Authorization?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="30310">Authorization</a> 标头，格式为：Authorization:<code data-backticks="1" data-nodeid="30312"> &lt;type&gt;</code> <code data-backticks="1" data-nodeid="30314">&lt;credentials</code>&gt;。这里的 type 就是“Basic”，而 credentials 则是这样一个字符串：</p>




<pre class="lang-xml" data-nodeid="26380"><code data-language="xml">dXNlcjo5YjE5MWMwNC1lNWMzLTQ0YzctOGE3ZS0yNWNkMjY3MmVmMzk=
</code></pre>
<p data-nodeid="26381">这个字符串就是<strong data-nodeid="26516">将用户名和密码组合在一起，再经过 Base64 编码得到的结果</strong>。而我们知道 Base64 只是一种编码方式，并没有集成加密机制，所以本质上传输的还是<strong data-nodeid="26517">明文形式</strong>。</p>
<p data-nodeid="26382">当然，想要在应用程序中启用 HTTP 基础认证还是比较简单的，只需要在 WebSecurityConfigurerAdapter 的 configure 方法中添加如下配置即可：</p>
<pre class="lang-java" data-nodeid="26383"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.httpBasic();
}
</code></pre>
<p data-nodeid="26384">HTTP 基础认证比较简单，没有定制的登录页面，所以单独使用的场景比较有限。在使用 Spring Security 时，我们<strong data-nodeid="26524">一般会把 HTTP 基础认证和接下来要介绍的表单登录认证结合起来</strong>一起使用。</p>
<h4 data-nodeid="26385">表单登录认证</h4>
<p data-nodeid="26386">在 WebSecurityConfigurerAdapter 的 configure() 方法中，一旦配置了 HttpSecurity 的 formLogin() 方法，就启动了表单登录认证，如下所示：</p>
<pre class="lang-java" data-nodeid="26387"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; http.formLogin();
}
</code></pre>
<p data-nodeid="26388">formLogin() 方法的执行效果就是提供了一个默认的登录界面，如下所示：</p>
<p data-nodeid="31359" class=""><img src="https://s0.lgstatic.com/i/image6/M01/43/A3/CioPOWC5_fmAOso9AAAqvMlklW8869.png" alt="Drawing 1.png" data-nodeid="31363"></p>
<div data-nodeid="31360"><p style="text-align:center">Spring Security 默认的登录界面</p></div>



<p data-nodeid="26391">我们已经在上一讲中看到过这个登录界面。对于登录操作而言，这个登录界面通常都是定制化的，同时，我们也需要对登录的过程和结果进行细化控制。此时，我们就可以通过如下所示的配置内容来修改系统的默认配置：</p>
<pre class="lang-java" data-nodeid="26392"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(HttpSecurity http)</span> <span class="hljs-keyword">throws</span> Exception </span>{
	&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;http
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .formLogin()&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .loginPage(<span class="hljs-string">"/login.html"</span>)<span class="hljs-comment">//自定义登录页面</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .loginProcessingUrl(<span class="hljs-string">"/action"</span>)<span class="hljs-comment">//登录表单提交时的处理地址</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .defaultSuccessUrl(<span class="hljs-string">"/index"</span>);<span class="hljs-comment">//登录认证成功后的跳转页面&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; </span>
}
</code></pre>
<p data-nodeid="26393">可以看到，这里我们对登录界面、登录请求处理地址以及登录成功后的跳转界面进行了定制化。</p>
<h3 data-nodeid="26394">配置 Spring Security 用户认证体系</h3>
<p data-nodeid="26395">讲完配置体系，现在让我们回到用户认证场景。因为 Spring Security 默认提供的用户名是固定的，而密码会随着每次应用程序的启动而变化，所以很不灵活。在 Spring Boot 中，我们可以通过在 application.yml 配置文件中添加如下所示的配置项来改变这种默认行为：</p>
<pre class="lang-xml" data-nodeid="26396"><code data-language="xml">spring:
&nbsp; security:
&nbsp;&nbsp;&nbsp; user:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; name: spring
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; password: spring_password
</code></pre>
<p data-nodeid="26397">现在让我们重启应用，就可以使用上述用户名和密码完成登录。基于配置文件的用户信息存储方案简单直接，但显然也<strong data-nodeid="26543">缺乏灵活性</strong>，因为我们无法在系统运行时<strong data-nodeid="26544">动态加载对应的用户名和密码</strong>。因此，在现实中，我们主要还是通过使用 WebSecurityConfigurerAdapter 配置类来改变默认的配置行为。</p>
<p data-nodeid="26398">通过前面的内容中，我们已经知道可以通过 WebSecurityConfigurerAdapter 类的 configure(HttpSecurity http) 方法来完成认证。认证过程涉及 Spring Security 中用户信息的交互，我们可以通过继承 WebSecurityConfigurerAdapter 类并且覆写其中的 configure(AuthenticationManagerBuilder auth) 的方法来完成对用户信息的配置工作。请注意<strong data-nodeid="26550">这是两个不同的 configure() 方法</strong>。</p>
<p data-nodeid="26399">针对 WebSecurityConfigurer 配置类，我们首先需要明确配置的内容。实际上，初始化用户信息非常简单，只需要指定用户名（Username）、密码（Password）和角色（Role）这三项数据即可。在 Spring Security 中，基于 AuthenticationManagerBuilder 工具类为开发人员提供了<strong data-nodeid="26556">基于内存、JDBC、LDAP 等多种验证方案</strong>。</p>
<p data-nodeid="26400">接下来，我们就围绕 AuthenticationManagerBuilder 提供的功能来实现多种用户信息存储方案。</p>
<h4 data-nodeid="26401">使用基于内存的用户信息存储方案</h4>
<p data-nodeid="26402">我们先来看如何使用 AuthenticationManagerBuilder 完成基于内存的用户信息存储方案。实现方法就是调用 AuthenticationManagerBuilder 的 inMemoryAuthentication 方法，示例代码如下：</p>
<pre class="lang-java" data-nodeid="26403"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder builder)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;builder.inMemoryAuthentication()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"spring_user"</span>).password(<span class="hljs-string">"password1"</span>).roles(<span class="hljs-string">"USER"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"spring_admin"</span>).password(<span class="hljs-string">"password2"</span>).roles(<span class="hljs-string">"USER"</span>, <span class="hljs-string">"ADMIN"</span>);
}
</code></pre>
<p data-nodeid="26404">从上面的代码中，我们可以看到系统中存在“spring_user”和“spring_admin”这两个用户，其密码分别是"password1"和"password2"，在角色上也分别代表着普通用户 USER 以及管理员 ADMIN。</p>
<p data-nodeid="26405">请注意，这里的 roles() 方法背后使用的还是<strong data-nodeid="26580">authorities() 方法</strong>。通过 roles() 方法，Spring Security 会在每个角色名称前自动添加“ROLE_”前缀，所以我们也可以通过如下所示的代码实现同样的功能：</p>
<pre class="lang-java" data-nodeid="26406"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder builder)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;builder.inMemoryAuthentication()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"spring_user"</span>).password(<span class="hljs-string">"password1"</span>).authorities(<span class="hljs-string">"ROLE_USER"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.and()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.withUser(<span class="hljs-string">"spring_admin"</span>).password(<span class="hljs-string">"password2"</span>).authorities(<span class="hljs-string">"ROLE_USER"</span>, <span class="hljs-string">"ROLE_ADMIN"</span>);
}
</code></pre>
<p data-nodeid="26407">可以看到，基于内存的用户信息存储方案实现也比较简单，但同样缺乏灵活性，因为用户信息是写死在代码里的。所以，我们接下来就要引出另一种更为常见的用户信息存储方案——数据库存储。</p>
<h4 data-nodeid="26408">使用基于数据库的用户信息存储方案</h4>
<p data-nodeid="26409">既然是将用户信息存储在数据库中，势必需要<strong data-nodeid="26588">创建表结构</strong>。我们可以在 Spring Security 的源文件（org/springframework/security/core/userdetails/jdbc/users.ddl）中找到对应的 SQL 语句，如下所示：</p>
<pre class="lang-java" data-nodeid="26410"><code data-language="java"><span class="hljs-function">create table <span class="hljs-title">users</span><span class="hljs-params">(username varchar_ignorecase(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span> primary key,password <span class="hljs-title">varchar_ignorecase</span><span class="hljs-params">(<span class="hljs-number">500</span>)</span> not <span class="hljs-keyword">null</span>,enabled <span class="hljs-keyword">boolean</span> not <span class="hljs-keyword">null</span>)</span>;
 
<span class="hljs-function">create table <span class="hljs-title">authorities</span> <span class="hljs-params">(username varchar_ignorecase(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span>,authority <span class="hljs-title">varchar_ignorecase</span><span class="hljs-params">(<span class="hljs-number">50</span>)</span> not <span class="hljs-keyword">null</span>,constraint fk_authorities_users foreign <span class="hljs-title">key</span><span class="hljs-params">(username)</span> references <span class="hljs-title">users</span><span class="hljs-params">(username)</span>)</span>;
 
<span class="hljs-function">create unique index ix_auth_username on <span class="hljs-title">authorities</span> <span class="hljs-params">(username,authority)</span></span>;
</code></pre>
<p data-nodeid="26411">一旦我们在自己的数据库中创建了这两张表，并添加了相应的数据，就可以直接通过注入一个 DataSource 对象进行用户数据的查询，如下所示：</p>
<pre class="lang-java" data-nodeid="26412"><code data-language="java"><span class="hljs-meta">@Autowired</span>
DataSource dataSource;
&nbsp;
<span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.jdbcAuthentication().dataSource(dataSource)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .usersByUsernameQuery(<span class="hljs-string">"select username, password, enabled from Users "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authoritiesByUsernameQuery(<span class="hljs-string">"select username, authority from UserAuthorities "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .passwordEncoder(<span class="hljs-keyword">new</span> BCryptPasswordEncoder());
}
</code></pre>
<p data-nodeid="26413">这里使用了 AuthenticationManagerBuilder 的 jdbcAuthentication 方法来配置数据库认证方式，内部则使用了 JdbcUserDetailsManager 这个工具类。在该类中，就定义了各种用于数据库查询的 SQL 语句，以及使用 JdbcTemplate 完成数据库访问的具体实现方法。</p>
<p data-nodeid="26414">请你注意，这里我们用到了一个<strong data-nodeid="26600">passwordEncoder() 方法</strong>，这是 Spring Security 中提供的一个<strong data-nodeid="26601">密码加解密器</strong>，我们会在“密码安全：Spring Security 中包含哪些加解密技术？”一讲中进行详细的讨论。</p>
<h3 data-nodeid="26415">小结与预告</h3>
<p data-nodeid="26416">这一讲我们详细介绍了如何使用 Spring Security 构建用户认证体系的系统方法。在 Spring Security 中，认证相关的功能都是可以通过配置体系进行定制化开发和管理的。通过简单的配置方法，我们可以组合使用 HTTP 基础认证和表单登录认证，也可以分别基于内存以及基于数据库方案来存储用户信息，这些功能都是 Spring Security 内置的。</p>
<p data-nodeid="26417">本讲内容总结如下：</p>
<p data-nodeid="31888" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/43/A4/CioPOWC5_g-AP68cAACSqF1r51g526.png" alt="Drawing 2.png" data-nodeid="31891"></p>

<p data-nodeid="26419">最后我想给你留一道思考题：你知道在 Spring Security 中有哪几种存储用户信息的实现方案吗？欢迎在留言区和我分享你的想法。</p>
<p data-nodeid="26420">介绍完用户认证的使用方法之后，下一讲我们将深入框架内部，来剖析 Spring Security 中认证过程的实现原理。</p>

---

### 精选评论

##### *斌：
> Spring-Session+Spring-security

##### **伟：
> 作者您好！现在的项目好多都是前后端分离的，我想问一下：那么在前后端分离的项目中如何指定前段项目的登录页面。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 前后端分离的时候，统一通过HTTP协议发起请求并获取JSON数据之后，再通过返回的JSON数据来控制前端界面跳转，这时候就跟后台没关系了

##### **4076：
> 作者厉害了，这专栏看得收获非常大

##### **清：
> .usersByUsernameQuery("select username, password, enabled from Users " + "where username=?")这行代码，这样写，会不会导致sql注入呢？希望老师能回复下

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用JdbcTemplate这种工具类都能防止SQL注入的

##### **清：
> 1. 能不能把代码也开放出来？2. 能讲下他的整体架构，类和接口设计就好了，会有更整体的认识

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 案例代码Github上有的，可以看一下第一讲中的说明

##### **3178：
> 基本上都是用数据库存储，但是也支持内存存储和文件存储

