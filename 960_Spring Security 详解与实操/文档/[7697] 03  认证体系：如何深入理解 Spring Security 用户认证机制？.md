<p data-nodeid="33056">上一讲，我们给出了 Spring Security 中实现用户认证的系统方法，可以发现整个实现过程还是比较简单的，开发人员只需要通过一些配置方法就能完成复杂的处理逻辑。这种简单性得益于 Spring Security 对用户认证过程的提炼和抽象。今天我们就围绕这个话题展开讨论，继续探究 Spring Security 中的用户和认证对象，以及如何基于这些对象完成定制化的用户认证方案。</p>


<h3 data-nodeid="32422">Spring Security 中的用户和认证</h3>
<p data-nodeid="32423">Spring Security 中的认证过程由一组核心对象组成，大致可以分成两大类，一类是<strong data-nodeid="32517">用户对象</strong>，一类是<strong data-nodeid="32518">认证对象</strong>，下面我们来具体了解一下。</p>
<h4 data-nodeid="32424">Spring Security 中的用户对象</h4>
<p data-nodeid="32425">Spring Security 中的用户对象用来描述用户并完成对用户信息的管理，涉及<strong data-nodeid="32525">UserDetails、GrantedAuthority、UserDetailsService 和 UserDetailsManager</strong>这四个核心对象。</p>
<ul data-nodeid="32426">
<li data-nodeid="32427">
<p data-nodeid="32428">UserDetails：描述 Spring Security 中的用户。</p>
</li>
<li data-nodeid="32429">
<p data-nodeid="32430">GrantedAuthority：定义用户的操作权限。</p>
</li>
<li data-nodeid="32431">
<p data-nodeid="32432">UserDetailsService：定义了对 UserDetails 的查询操作。</p>
</li>
<li data-nodeid="32433">
<p data-nodeid="32434">UserDetailsManager：扩展 UserDetailsService，添加了创建用户、修改用户密码等功能。</p>
</li>
</ul>
<p data-nodeid="33897">这四个对象之间的关联关系如下图所示，显然，对于由 UserDetails 对象所描述的一个用户而言，它应该具有 1 个或多个能够执行的 GrantedAuthority：</p>
<p data-nodeid="33898" class=""><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_ryAZ3qSAABXNOe10bU172.png" alt="Drawing 0.png" data-nodeid="33903"></p>
<div data-nodeid="33899"><p style="text-align:center">Spring Security 中的四大核心用户对象</p></div>




<p data-nodeid="32438">结合上图，我们先来看承载用户详细信息的 UserDetails 接口，如下所示：</p>
<pre class="lang-java" data-nodeid="32439"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetails</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Serializable</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取该用户的权限信息</span>
&nbsp;&nbsp;&nbsp; Collection&lt;? extends GrantedAuthority&gt; getAuthorities();
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取密码</span>
	<span class="hljs-function">String <span class="hljs-title">getPassword</span><span class="hljs-params">()</span></span>;
	&nbsp;
	<span class="hljs-comment">//获取用户名</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getUsername</span><span class="hljs-params">()</span></span>;
&nbsp;
	<span class="hljs-comment">//判断该账户是否已失效</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonExpired</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该账户是否已被锁定</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonLocked</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该账户的凭证信息是否已失效</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isCredentialsNonExpired</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断该用户是否可用</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="32440">通过 UserDetails，我们可以获取用户相关的基础信息，并判断其当前状态。同时，我们也可以看到 UserDetails 中保存着一组 GrantedAuthority 对象。而 GrantedAuthority 指定了一个方法用来获取权限信息，如下所示：</p>
<pre class="lang-java" data-nodeid="32441"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">GrantedAuthority</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Serializable</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取权限信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">getAuthority</span><span class="hljs-params">()</span></span>;
}
</code></pre>
<p data-nodeid="32442">UserDetails 存在一个子接口 MutableUserDetails，从命名上不难看出后者是一个可变的 UserDetails，而<strong data-nodeid="32540">可变的内容就是密码</strong>。MutableUserDetails 接口的定义如下所示：</p>
<pre class="lang-java" data-nodeid="32443"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">MutableUserDetails</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDetails</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置密码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">setPassword</span><span class="hljs-params">(String password)</span></span>;
}
</code></pre>
<p data-nodeid="32444">如果我们想要在应用程序中创建一个 UserDetails 对象，可以使用如下所示的链式语法：</p>
<pre class="lang-java" data-nodeid="32445"><code data-language="java">UserDetails user = User.withUsername(<span class="hljs-string">"jianxiang"</span>) 
  .password(<span class="hljs-string">"123456"</span>) 
  .authorities(<span class="hljs-string">"read"</span>, <span class="hljs-string">"write"</span>) 
  .accountExpired(<span class="hljs-keyword">false</span>) 
  .disabled(<span class="hljs-keyword">true</span>) 
  .build();
</code></pre>
<p data-nodeid="32446">Spring Security 还专门提供了一个 UserBuilder 对象来辅助构建 UserDetails，使用方式也类似：</p>
<pre class="lang-java" data-nodeid="32447"><code data-language="java">User.UserBuilder builder = 
	User.withUsername(<span class="hljs-string">"jianxiang"</span>);
&nbsp;
UserDetails user = builder
  .password(<span class="hljs-string">"12345"</span>) 
  .authorities(<span class="hljs-string">"read"</span>, <span class="hljs-string">"write"</span>)
  .accountExpired(<span class="hljs-keyword">false</span>) 
  .disabled(<span class="hljs-keyword">true</span>) 
  .build();
</code></pre>
<p data-nodeid="32448">在 Spring Security 中，针对 UserDetails 专门提供了一个 UserDetailsService，该接口用来管理 UserDetails，定义如下：</p>
<pre class="lang-java" data-nodeid="32449"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetailsService</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据用户名获取用户信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">UserDetails <span class="hljs-title">loadUserByUsername</span><span class="hljs-params">(String username)</span> <span class="hljs-keyword">throws</span> UsernameNotFoundException</span>;
}
</code></pre>
<p data-nodeid="32450">而 UserDetailsManager 继承了 UserDetailsService，并提供了一批针对 UserDetails 的操作接口，如下所示：</p>
<pre class="lang-java" data-nodeid="32451"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserDetailsManager</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">UserDetailsService</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">createUser</span><span class="hljs-params">(UserDetails user)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//更新用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">updateUser</span><span class="hljs-params">(UserDetails user)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//删除用户</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">deleteUser</span><span class="hljs-params">(String username)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//修改密码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">changePassword</span><span class="hljs-params">(String oldPassword, String newPassword)</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断指定用户名的用户是否存在</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">userExists</span><span class="hljs-params">(String username)</span></span>;
}
</code></pre>
<p data-nodeid="32452">这样，几个核心用户对象之间的关联关系就很清楚了，接下来我们需要进一步明确具体的实现过程。</p>
<p data-nodeid="32453">我们来看 UserDetailsManager 的两个实现类，一个是<strong data-nodeid="32555">基于内存存储</strong>的 InMemoryUserDetailsManager，一个是<strong data-nodeid="32556">基于关系型数据库存储</strong>的 JdbcUserDetailsManager。</p>
<p data-nodeid="32454">这里，我们以 JdbcMemoryUserDetailsManager 为例展开分析，它的 createUser 方法如下所示：</p>
<pre class="lang-java" data-nodeid="32455"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">createUser</span><span class="hljs-params">(<span class="hljs-keyword">final</span> UserDetails user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; validateUserDetails(user);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; getJdbcTemplate().update(createUserSql, ps -&gt; {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setString(<span class="hljs-number">1</span>, user.getUsername());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setString(<span class="hljs-number">2</span>, user.getPassword());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setBoolean(<span class="hljs-number">3</span>, user.isEnabled());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">int</span> paramCount = ps.getParameterMetaData().getParameterCount();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (paramCount &gt; <span class="hljs-number">3</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setBoolean(<span class="hljs-number">4</span>, !user.isAccountNonLocked());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setBoolean(<span class="hljs-number">5</span>, !user.isAccountNonExpired());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setBoolean(<span class="hljs-number">6</span>, !user.isCredentialsNonExpired());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (getEnableAuthorities()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; insertUserAuthorities(user);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32456">可以看到，这里直接使用了 Spring 框架中的 JdbcTemplate 模板工具类实现了数据的插入，同时完成了 GrantedAuthority 的存储。</p>
<p data-nodeid="32457">UserDetailsManager 是一条相对独立的代码线，为了完成用户信息的配置，还存在另一条代码支线，即 UserDetailsManagerConfigurer。该类维护了一个 UserDetails 列表，并提供了一组 withUser 方法完成用户信息的初始化，如下所示：</p>
<pre class="lang-java" data-nodeid="32458"><code data-language="java"><span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> List&lt;UserDetails&gt; users = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> C <span class="hljs-title">withUser</span><span class="hljs-params">(UserDetails userDetails)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.users.add(userDetails);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> (C) <span class="hljs-keyword">this</span>;
}
</code></pre>
<p data-nodeid="32459">而 withUser 方法返回的是一个 UserDetailsBuilder 对象，该对象内部使用了前面介绍的 UserBuilder 对象，因此可以实现类似 .withUser("spring_user").password("password1").roles("USER") 这样的链式语法，完成用户信息的设置。这也是上一讲中，我们在介绍基于内存的用户信息存储方案时使用的方法。</p>
<p data-nodeid="32460">作为总结，我们也梳理了 Spring Security 中与用户对象相关的一大批实现类，它们之间的关系如下图所示：</p>
<p data-nodeid="34326" class=""><img src="https://s0.lgstatic.com/i/image6/M00/43/9B/Cgp9HWC5_sqAC7SUAACLftd0SYc321.png" alt="Drawing 1.png" data-nodeid="34329"></p>

<div data-nodeid="34756" class=""><p style="text-align:center">Spring Security 中用户对象相关类结构图</p></div>

<h4 data-nodeid="32463">Spring Security 中的认证对象</h4>
<p data-nodeid="32464">有了用户对象，我们就可以讨论具体的认证过程了，首先来看认证对象 Authentication，如下所示：</p>
<pre class="lang-java" data-nodeid="32465"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">Authentication</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Principal</span>, <span class="hljs-title">Serializable</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//安全主体具有的权限</span>
&nbsp;&nbsp;&nbsp; Collection&lt;? extends GrantedAuthority&gt; getAuthorities();
&nbsp;
	<span class="hljs-comment">//证明主体有效性的凭证</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">getCredentials</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//认证请求的明细信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">getDetails</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//主体的标识信息</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">getPrincipal</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//认证是否通过</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">isAuthenticated</span><span class="hljs-params">()</span></span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//设置认证结果</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">setAuthenticated</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> isAuthenticated)</span> <span class="hljs-keyword">throws</span> IllegalArgumentException</span>;
}
</code></pre>
<p data-nodeid="32466">认证对象代表认证请求本身，并保存该请求访问应用程序过程中涉及的各个实体的详细信息。在安全领域，请求访问该应用程序的用户通常被称为<strong data-nodeid="32585">主体</strong>（Principal），在 JDK 中存在一个同名的接口，而 Authentication 扩展了这个接口。</p>
<p data-nodeid="32467">显然，Authentication 只代表了认证请求本身，而具体执行认证的过程和逻辑需要由专门的组件来负责，这个组件就是 AuthenticationProvider，定义如下：</p>
<pre class="lang-java" data-nodeid="32468"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行认证，返回认证结果</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> AuthenticationException</span>;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断是否支持当前的认证对象</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; authentication)</span></span>;
}
</code></pre>
<p data-nodeid="32469">讲到这里，你可能会认为 Spring Security 是直接使用 AuthenticationProvider 接口完成用户认证的，其实不然。如果你翻阅 Spring Security 的源码，会发现它<strong data-nodeid="32592">使用了 AuthenticationManager 接口来代理 AuthenticationProvider 提供的认证功能</strong>。这里，我们以 InMemoryUserDetailsManager 中的 changePassword 为例，分析用户认证的执行过程（为了展示简洁，部分代码做了裁剪）：</p>
<pre class="lang-java" data-nodeid="32470"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">changePassword</span><span class="hljs-params">(String oldPassword, String newPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Authentication currentUser = SecurityContextHolder.getContext()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .getAuthentication();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (currentUser == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> AccessDeniedException(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"Can't change password as no Authentication object found in context "</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; + <span class="hljs-string">"for current user."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = currentUser.getName();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (authenticationManager != <span class="hljs-keyword">null</span>) {
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; authenticationManager.authenticate(<span class="hljs-keyword">new</span> UsernamePasswordAuthenticationToken(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; username, oldPassword));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; …
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; MutableUserDetails user = users.get(username);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (user == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalStateException(<span class="hljs-string">"Current user doesn't exist in database."</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; user.setPassword(newPassword);
}
</code></pre>
<p data-nodeid="32471">可以看到，这里使用了 AuthenticationManager 而不是 AuthenticationProvider 中的 authenticate() 方法来执行认证。同时，我们也注意到这里出现了  UsernamePasswordAuthenticationToken 类，这就是 Authentication 接口的一个具体实现类，用来<strong data-nodeid="32598">存储用户认证所需的用户名和密码信息</strong>。</p>
<p data-nodeid="35604">同样作为总结，我们也梳理了 Spring Security 中与认证对象相关的一大批核心类，它们之间的关系如下所示：</p>
<p data-nodeid="35605" class=""><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_tmAEUuLAABqhhFbRIw821.png" alt="Drawing 2.png" data-nodeid="35610"></p>
<div data-nodeid="35606"><p style="text-align:center">Spring Security 中认证的对象相关类结构图</p></div>




<h3 data-nodeid="32475">实现定制化用户认证方案</h3>
<p data-nodeid="32476">通过前面的分析，我们明确了用户信息存储的实现过程实际上是可以定制化的。Spring Security 所做的工作只是把常见的、符合一般业务场景的实现方式嵌入到了框架中。如果有特殊的场景，开发人员完全可以实现自定义的用户信息存储方案。</p>
<p data-nodeid="32477">现在，我们已经知道 UserDetails 接口代表着用户详细信息，而负责对 UserDetails 进行各种操作的则是 UserDetailsService 接口。因此，<strong data-nodeid="32609">实现定制化用户认证方案主要就是实现 UserDetails 和 UserDetailsService 这两个接口</strong>。</p>
<h4 data-nodeid="32478">扩展 UserDetails</h4>
<p data-nodeid="32479">扩展 UserDetails 的方法就是直接实现该接口，例如我们可以构建如下所示的 SpringUser 类：</p>
<pre class="lang-java" data-nodeid="32480"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringUser</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetails</span> </span>{
&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">long</span> serialVersionUID = <span class="hljs-number">1L</span>;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> Long id;&nbsp; 
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String username;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String password;
&nbsp; &nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String phoneNumber;
&nbsp; &nbsp; <span class="hljs-comment">//省略 getter/setter</span>
&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUsername</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> username;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getPassword</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> password;
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp; &nbsp; <span class="hljs-meta">@Override</span>
&nbsp; &nbsp; <span class="hljs-keyword">public</span> Collection&lt;? extends GrantedAuthority&gt; getAuthorities() {
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> Arrays.asList(<span class="hljs-keyword">new</span> SimpleGrantedAuthority(<span class="hljs-string">"ROLE_USER"</span>));
&nbsp; &nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isAccountNonLocked</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isCredentialsNonExpired</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">isEnabled</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp; }
}
</code></pre>
<p data-nodeid="32481">显然，这里使用了一种最简单的方法来满足 UserDetails 中各个接口的实现需求。一旦我们构建了这样一个 SpringUser 类，就可以创建对应的表结构存储类中定义的字段。同时，我们也可以基于 Spring Data JPA 来创建一个自定义的 Repository，如下所示：</p>
<pre class="lang-java" data-nodeid="32482"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">SpringUserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">SpringUser</span>, <span class="hljs-title">Long</span>&gt; </span>{
&nbsp; &nbsp; <span class="hljs-function">SpringUser <span class="hljs-title">findByUsername</span><span class="hljs-params">(String username)</span></span>;&nbsp; 
}
</code></pre>
<p data-nodeid="32483">SpringUserRepository 扩展了 Spring Data 中的 CrudRepository 接口，并提供了一个方法名衍生查询 findByUsername。</p>
<h4 data-nodeid="32484">扩展 UserDetailsService</h4>
<p data-nodeid="32485">接着，我们来实现 UserDetailsService 接口，如下所示：</p>
<pre class="lang-java" data-nodeid="32486"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringUserDetailsService</span> 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">implements</span> <span class="hljs-title">UserDetailsService</span> </span>{
	&nbsp;
  <span class="hljs-meta">@Autowired</span>
&nbsp; <span class="hljs-keyword">private</span> SpringUserRepository repository;
&nbsp;
&nbsp; <span class="hljs-meta">@Override</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> UserDetails <span class="hljs-title">loadUserByUsername</span><span class="hljs-params">(String username)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throws</span> UsernameNotFoundException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; SpringUser user = repository.findByUsername(username);
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (user != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> user;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> UsernameNotFoundException(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"SpringUser '"</span> + username + <span class="hljs-string">"' not found"</span>);
&nbsp; }
}
</code></pre>
<p data-nodeid="32487">我们知道 UserDetailsService 接口只有一个 loadUserByUsername 方法需要实现。因此，我们基于 SpringUserRepository 的 findByUsername 方法，根据用户名从数据库中查询数据。</p>
<h4 data-nodeid="32488">扩展 AuthenticationProvider</h4>
<p data-nodeid="32489">扩展 AuthenticationProvider 的过程就是提供一个自定义的 AuthenticationProvider 实现类。这里我们以最常见的用户名密码认证为例，梳理自定义认证过程所需要实现的步骤，如下所示：</p>
<p data-nodeid="36462" class=""><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_uiAEATKAACWyEa75HA969.png" alt="Drawing 3.png" data-nodeid="36466"></p>
<div data-nodeid="36463"><p style="text-align:center">自定义 AuthenticationProvider 的实现流程图</p></div>



<p data-nodeid="32492">上图中的流程并不复杂，首先我们需要通过 UserDetailsService 获取一个 UserDetails 对象，然后根据该对象中的密码与认证请求中的密码进行匹配，如果一致则认证成功，反之抛出一个 BadCredentialsException 异常。示例代码如下所示：</p>
<pre class="lang-java" data-nodeid="32493"><code data-language="java"><span class="hljs-meta">@Component</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringAuthenticationProvider</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">AuthenticationProvider</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserDetailsService userDetailsService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> PasswordEncoder passwordEncoder;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Authentication <span class="hljs-title">authenticate</span><span class="hljs-params">(Authentication authentication)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String username = authentication.getName();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String password = authentication.getCredentials().toString();
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; UserDetails user = userDetailsService.loadUserByUsername(username);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (passwordEncoder.matches(password, user.getPassword())) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> UsernamePasswordAuthenticationToken(username, password, u.getAuthorities());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> BadCredentialsException(<span class="hljs-string">"The username or password is wrong!"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">supports</span><span class="hljs-params">(Class&lt;?&gt; authenticationType)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> authenticationType.equals(UsernamePasswordAuthenticationToken.class);
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="32494">这里我们同样使用了 UsernamePasswordAuthenticationToken 来传递用户名和密码，并使用一个 PasswordEncoder 对象校验密码。</p>
<h4 data-nodeid="32495">整合定制化配置</h4>
<p data-nodeid="32496">最后，我们创建一个 SpringSecurityConfig 类，该类继承了 WebSecurityConfigurerAdapter 配置类。这次，我们将使用自定义的 SpringUserDetailsService 来完成用户信息的存储和查询，需要对原有配置策略做一些调整。调整之后的完整 SpringSecurityConfig 类如下所示：</p>
<pre class="lang-java" data-nodeid="32497"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">SpringSecurityConfig</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">WebSecurityConfigurerAdapter</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> UserDetailsService springUserDetailsService;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> AuthenticationProvider springAuthenticationProvider;
&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;  &nbsp;auth.userDetailsService(springUserDetailsService)
	.authenticationProvider(springAuthenticationProvider);
	}
}
</code></pre>
<p data-nodeid="32498">这里我们注入了 SpringUserDetailsService 和 SpringAuthenticationProvider，并将其添加到 AuthenticationManagerBuilder 中，这样 AuthenticationManagerBuilder 将基于这个自定义的 SpringUserDetailsService 完成 UserDetails 的创建和管理，并基于自定义的 SpringAuthenticationProvider 完成用户认证。</p>
<h3 data-nodeid="32499">小结与预告</h3>
<p data-nodeid="32500">这一讲我们基于 Spring Security 提供的用户认证功能分析了其背后的实现过程。我们的切入点在于分析与用户和认证相关的各个核心类，并梳理它们之间的交互过程。另一方面，我们还通过扩展 UserDetailsService 和 AuthenticationProvider 接口的方式来实现定制化的用户认证方案。</p>
<p data-nodeid="32501">本讲内容总结如下：</p>
<p data-nodeid="36895" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_wKAct0XAACvS5HbogQ525.png" alt="Drawing 4.png" data-nodeid="36898"></p>

<p data-nodeid="32503">最后给你留一道思考题：基于 Spring Security，如何根据用户名和密码实现一套定制化的用户认证解决方案？欢迎在留言区和我分享你的想法。</p>
<p data-nodeid="32504">在上一讲和今天的内容中，我们都看到了用于实现密码加解密的 PasswordEncoder 对象。密码安全也是 Spring Security 提供的一项基础功能，下一讲我们将针对这个主题展开讨论。</p>

---

### 精选评论

##### *强：
> 已经有了AuthenticationManage为啥还要AuthenticationProvider

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这是Spring Security对认证过程中职责的进一步分离，确保架构清晰

##### *西：
> 最简单的方式就是增加一个过滤器，放到usernameautitixxxxfilter前面，然后在过滤器里面把用户名与密码检查，匹配工作都做了。

