<p data-nodeid="4827" class="">通过前面两讲内容的学习，相信你已经掌握了 Spring Security 中的用户认证体系。用户认证的过程通常涉及密码的校验，因此密码的安全性也是我们需要考虑的一个核心问题。Spring Security 作为一款功能完备的安全性框架，一方面提供了<strong data-nodeid="4922">用于完成认证操作的 PasswordEncoder 组件</strong>，另一方面也包含一个独立而完整的<strong data-nodeid="4923">加密模块</strong>，方便在应用程序中单独使用。</p>
<h3 data-nodeid="4828">PasswordEncoder</h3>
<p data-nodeid="4829">我们先来回顾一下整个用户认证流程。在 AuthenticationProvider 中，我们需要使用 PasswordEncoder 组件验证密码的正确性，如下图所示：</p>
<p data-nodeid="4830"><img src="https://s0.lgstatic.com/i/image6/M01/43/9C/Cgp9HWC5_4iAIGeQAABZX7ZLFlk655.png" alt="Drawing 0.png" data-nodeid="4928"></p>
<div data-nodeid="4831"><p style="text-align:center">PasswordEncoder 组件与认证流程之间的关系</p></div>
<p data-nodeid="4832">在“用户认证：如何使用 Spring Security 构建用户认证体系？”一讲中我们也介绍了基于数据库的用户信息存储方案：</p>
<pre class="lang-java" data-nodeid="4833"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">configure</span><span class="hljs-params">(AuthenticationManagerBuilder auth)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; auth.jdbcAuthentication().dataSource(dataSource)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .usersByUsernameQuery(<span class="hljs-string">"select username, password, enabled from Users "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .authoritiesByUsernameQuery(<span class="hljs-string">"select username, authority from UserAuthorities "</span> + <span class="hljs-string">"where username=?"</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .passwordEncoder(<span class="hljs-keyword">new</span> BCryptPasswordEncoder());
}
</code></pre>
<p data-nodeid="4834">请注意，在上述方法中，我们通过 jdbcAuthentication() 方法验证用户信息时一定要<strong data-nodeid="4935">集成加密机制</strong>，也就是使用 passwordEncoder() 方法嵌入一个 PasswordEncoder 接口的实现类。</p>
<h4 data-nodeid="4835">PasswordEncoder 接口</h4>
<p data-nodeid="4836">在 Spring Security 中，PasswordEncoder 接口代表的是一种密码编码器，其核心作用在于<strong data-nodeid="4942">指定密码的具体加密方式</strong>，以及如何将一段给定的加密字符串与明文之间完成匹配校验。PasswordEncoder 接口定义如下：</p>
<pre class="lang-java" data-nodeid="4837"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PasswordEncoder</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//对原始密码进行编码</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function">String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//对提交的原始密码与库中存储的加密密码进行比对</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(CharSequence rawPassword, String encodedPassword)</span></span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//判断加密密码是否需要再次进行加密，默认返回 false</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">default</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">upgradeEncoding</span><span class="hljs-params">(String encodedPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="4838">Spring Security 内置了一大批 PasswordEncoder 接口的实现类，如下所示：</p>
<p data-nodeid="4839"><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_5KAdkT3AAQNHWtae5I850.png" alt="Drawing 1.png" data-nodeid="4946"></p>
<div data-nodeid="4840"><p style="text-align:center">Spring Security 中的 PasswordEncoder 实现类</p></div>
<p data-nodeid="4841">我们对上图中比较常见的几个 PasswordEncoder 接口展开叙述。</p>
<ul data-nodeid="4842">
<li data-nodeid="4843">
<p data-nodeid="4844">NoOpPasswordEncoder：以明文形式保留密码，不对密码进行编码。这种 PasswordEncoder 通常只用于演示，不应该用于生产环境。</p>
</li>
<li data-nodeid="4845">
<p data-nodeid="4846">StandardPasswordEncoder：使用 SHA-256 算法对密码执行哈希操作。</p>
</li>
<li data-nodeid="4847">
<p data-nodeid="4848">BCryptPasswordEncoder：使用 bcrypt 强哈希算法对密码执行哈希操作。</p>
</li>
<li data-nodeid="4849">
<p data-nodeid="4850">Pbkdf2PasswordEncoder：使用 PBKDF2 算法对密码执行哈希操作。</p>
</li>
</ul>
<p data-nodeid="4851">下面我们以 BCryptPasswordEncoder 为例，看一下它的 encode 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="4852"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String salt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (random != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; salt = BCrypt.gensalt(version.getVersion(), strength, random);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">else</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; salt = BCrypt.gensalt(version.getVersion(), strength);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> BCrypt.hashpw(rawPassword.toString(), salt);
}
</code></pre>
<p data-nodeid="4853">可以看到，上述 encode 方法执行了两个步骤，首先使用 Spring Security 提供的 BCrypt 工具类生成盐（Salt），然后根据盐和明文密码生成最终的密文密码。这里有必要对加盐的概念做一些展开：所谓加盐，就是在初始化明文数据时，由系统自动往这个明文里添加一些附加数据，然后散列。引入加盐机制是为了<strong data-nodeid="4958">进一步保证加密数据的安全性</strong>，单向散列加密以及加盐思想也被广泛应用于系统登录过程中的密码生成和校验。</p>
<p data-nodeid="4854">同样，在 Pbkdf2PasswordEncoder 中，也是通过对密码加盐之后进行哈希，然后将结果作为盐再与密码进行哈希，多次重复此过程，生成最终的密文。</p>
<p data-nodeid="4855">介绍完 PasswordEncoder 的基本结构，我们继续来看它的应用方式。如果我们想在应用程序中使用某一个 PasswordEncoder 实现类，通常只需要通过它的构造函数创建一个实例，例如：</p>
<pre class="lang-java" data-nodeid="4856"><code data-language="java">PasswordEncoder p = <span class="hljs-keyword">new</span> StandardPasswordEncoder(); 
PasswordEncoder p = <span class="hljs-keyword">new</span> StandardPasswordEncoder(<span class="hljs-string">"secret"</span>);
&nbsp;
PasswordEncoder p = <span class="hljs-keyword">new</span> SCryptPasswordEncoder(); 
PasswordEncoder p = <span class="hljs-keyword">new</span> SCryptPasswordEncoder(<span class="hljs-number">16384</span>, <span class="hljs-number">8</span>, <span class="hljs-number">1</span>, <span class="hljs-number">32</span>, <span class="hljs-number">64</span>);
</code></pre>
<p data-nodeid="4857">而如果想要使用 NoOpPasswordEncoder，除了构造函数之外，还可以通过它的 getInstance() 方法来获取静态实例，如下所示：</p>
<pre class="lang-java" data-nodeid="4858"><code data-language="java">PasswordEncoder p = NoOpPasswordEncoder.getInstance()
</code></pre>
<h4 data-nodeid="4859">自定义 PasswordEncoder</h4>
<p data-nodeid="4860">尽管 Spring Security 已经为我们提供了丰富的 PasswordEncoder，但你也可以通过实现这个接口来设计满足自身需求的任意一种密码编解码和验证机制。例如，我们可以编写如下所示的一个 PlainTextPasswordEncoder：</p>
<pre class="lang-java" data-nodeid="4861"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PlainTextPasswordEncoder</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">PasswordEncoder</span> </span>{
&nbsp;
   <span class="hljs-meta">@Override</span>
  &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span> </span>{

  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> rawPassword.toString(); 
  &nbsp;}

  &nbsp;<span class="hljs-meta">@Override</span>
  &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(CharSequence rawPassword, String encodedPassword)</span> </span>{

  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> rawPassword.equals(encodedPassword); 
  &nbsp;}
}
</code></pre>
<p data-nodeid="4862">PlainTextPasswordEncoder 的功能与 NoOpPasswordEncoder 类似，没有对明文进行任何处理。如果你想使用某种算法集成 PasswordEncoder，就可以实现类似如下所示的 Sha512PasswordEncoder，这里使用了 SHA-512 作为加解密算法：</p>
<pre class="lang-java" data-nodeid="4863"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Sha512PasswordEncoder</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">PasswordEncoder</span> </span>{
&nbsp;
  &nbsp;<span class="hljs-meta">@Override</span>
  &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span> </span>{
  &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> hashWithSHA512(rawPassword.toString());
  &nbsp;}

  &nbsp;<span class="hljs-meta">@Override</span>
  &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(CharSequence rawPassword, String encodedPassword)</span> </span>{
  &nbsp;&nbsp;&nbsp; String hashedPassword = encode(rawPassword);
  	  <span class="hljs-keyword">return</span> encodedPassword.equals(hashedPassword);
  &nbsp;}

   <span class="hljs-function"><span class="hljs-keyword">private</span> String <span class="hljs-title">hashWithSHA512</span><span class="hljs-params">(String input)</span> </span>{
  &nbsp;  StringBuilder result = <span class="hljs-keyword">new</span> StringBuilder();

    &nbsp;<span class="hljs-keyword">try</span> {
      &nbsp;MessageDigest md = MessageDigest.getInstance(<span class="hljs-string">"SHA-512"</span>);
      &nbsp;<span class="hljs-keyword">byte</span> [] digested = md.digest(input.getBytes());
      &nbsp;<span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; digested.length; i++) {
      &nbsp;result.append(Integer.toHexString(<span class="hljs-number">0xFF</span> &amp; digested[i]));
      &nbsp;}
      &nbsp;} <span class="hljs-keyword">catch</span> (NoSuchAlgorithmException e) {
      &nbsp;<span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> RuntimeException(<span class="hljs-string">"Bad algorithm"</span>);
    &nbsp;}

  &nbsp;  <span class="hljs-keyword">return</span> result.toString();
  }
}
</code></pre>
<p data-nodeid="4864">上述代码中，hashWithSHA512() 方法就使用了前面提到的<strong data-nodeid="4974">单向散列加密算法</strong>来生成消息摘要（Message&nbsp;Digest），其主要特点在于<strong data-nodeid="4975">单向不可逆和密文长度固定</strong>。同时也具备“碰撞”少的优点，即明文的微小差异就会导致所生成密文完全不同。SHA（Secure Hash Algorithm）以及MD5（Message Digest 5）都是常见的单向散列加密算法，在 JDK 自带的 MessageDigest 类中已经包含了默认实现，我们直接调用方法即可。</p>
<h4 data-nodeid="4865">代理式 DelegatingPasswordEncoder</h4>
<p data-nodeid="4866">在前面的讨论中，我们都基于一个假设，即在对密码进行加解密过程中，只会使用到一个 PasswordEncoder，如果这个 PasswordEncoder 不满足我们的需求，那么就需要替换成另一个 PasswordEncoder。这就引出了一个问题，如何优雅地应对这种变化呢？</p>
<p data-nodeid="4867">在普通的业务系统中，由于业务系统也在不断地变化，替换一个组件可能并没有很高的成本。但对于 Spring Security 这种成熟的开发框架而言，在设计和实现上不能经常发生变化。因此，在新/旧 PasswordEncoder 的兼容性，以及框架自身的稳健性和可变性之间需要保持一种平衡。为了实现这种平衡性，Spring Security 提供了 DelegatingPasswordEncoder。</p>

<p data-nodeid="4869">虽然 DelegatingPasswordEncoder 也实现了 PasswordEncoder 接口，但事实上，它更多扮演了一种代理组件的角色，这点从命名上也可以看出来。DelegatingPasswordEncoder 将具体编码的实现根据要求代理给不同的算法，以此实现不同编码算法之间的兼容并协调变化，如下图所示：</p>
<p data-nodeid="4870"><img src="https://s0.lgstatic.com/i/image6/M01/43/9C/Cgp9HWC5_6aACid_AACGDa8mfic454.png" alt="Drawing 2.png" data-nodeid="4982"></p>
<div data-nodeid="4871"><p style="text-align:center">DelegatingPasswordEncoder 的代理作用示意图</p></div>
<p data-nodeid="4872">下面我们来看一下 DelegatingPasswordEncoder 类的构造函数，如下所示：</p>
<pre class="lang-java" data-nodeid="4873"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">DelegatingPasswordEncoder</span><span class="hljs-params">(String idForEncode,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, PasswordEncoder&gt; idToPasswordEncoder)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (idForEncode == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"idForEncode cannot be null"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (!idToPasswordEncoder.containsKey(idForEncode)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"idForEncode "</span> + idForEncode + <span class="hljs-string">"is not found in idToPasswordEncoder "</span> + idToPasswordEncoder);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (String id : idToPasswordEncoder.keySet()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (id == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">continue</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (id.contains(PREFIX)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"id "</span> + id + <span class="hljs-string">" cannot contain "</span> + PREFIX);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (id.contains(SUFFIX)) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> IllegalArgumentException(<span class="hljs-string">"id "</span> + id + <span class="hljs-string">" cannot contain "</span> + SUFFIX);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.idForEncode = idForEncode;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.passwordEncoderForEncode = idToPasswordEncoder.get(idForEncode);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.idToPasswordEncoder = <span class="hljs-keyword">new</span> HashMap&lt;&gt;(idToPasswordEncoder);
}
</code></pre>
<p data-nodeid="4874">该构造函数中的 idForEncode 参数决定 PasswordEncoder 的类型，而 idToPasswordEncoder 参数决定判断匹配时兼容的类型。显然，idToPasswordEncoder 必须包含对应的 idForEncode。</p>
<p data-nodeid="4875">我们再来看这个构造函数的调用入口。在 Spring Security 中，存在一个创建 PasswordEncoder 的工厂类 PasswordEncoderFactories，如下所示：</p>
<pre class="lang-java" data-nodeid="4876"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PasswordEncoderFactories</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@SuppressWarnings("deprecation")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> PasswordEncoder <span class="hljs-title">createDelegatingPasswordEncoder</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String encodingId = <span class="hljs-string">"bcrypt"</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, PasswordEncoder&gt; encoders = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(encodingId, <span class="hljs-keyword">new</span> BCryptPasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"ldap"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.LdapShaPasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"MD4"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.Md4PasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"MD5"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.MessageDigestPasswordEncoder(<span class="hljs-string">"MD5"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"noop"</span>, org.springframework.security.crypto.password.NoOpPasswordEncoder.getInstance());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"pbkdf2"</span>, <span class="hljs-keyword">new</span> Pbkdf2PasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"scrypt"</span>, <span class="hljs-keyword">new</span> SCryptPasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"SHA-1"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.MessageDigestPasswordEncoder(<span class="hljs-string">"SHA-1"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"SHA-256"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.MessageDigestPasswordEncoder(<span class="hljs-string">"SHA-256"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"sha256"</span>, <span class="hljs-keyword">new</span> org.springframework.security.crypto.password.StandardPasswordEncoder());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; encoders.put(<span class="hljs-string">"argon2"</span>, <span class="hljs-keyword">new</span> Argon2PasswordEncoder());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> DelegatingPasswordEncoder(encodingId, encoders);
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">PasswordEncoderFactories</span><span class="hljs-params">()</span> </span>{}
}
</code></pre>
<p data-nodeid="4877">可以看到，在这个工厂类中初始化了一个包含所有 Spring Security 中支持 PasswordEncoder 的 Map。而且，我们也明确了框架默认使用的就是 key 为“bcrypt”的 BCryptPasswordEncoder。</p>
<p data-nodeid="4878">通常，我们可以通过以下方法来使用这个 PasswordEncoderFactories 类：</p>
<pre class="lang-java" data-nodeid="4879"><code data-language="java">PasswordEncoder passwordEncoder =
&nbsp;&nbsp;&nbsp; PasswordEncoderFactories.createDelegatingPasswordEncoder();
</code></pre>
<p data-nodeid="4880">另一方面，PasswordEncoderFactories 的实现方法为我们自定义 DelegatingPasswordEncoder 提供了一种途径，我们也可以根据需要创建符合自己需求的 DelegatingPasswordEncoder，如下所示：</p>
<pre class="lang-java" data-nodeid="4881"><code data-language="java">String idForEncode = <span class="hljs-string">"bcrypt"</span>;
Map encoders = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
encoders.put(idForEncode, <span class="hljs-keyword">new</span> BCryptPasswordEncoder());
encoders.put(<span class="hljs-string">"noop"</span>, NoOpPasswordEncoder.getInstance());
encoders.put(<span class="hljs-string">"pbkdf2"</span>, <span class="hljs-keyword">new</span> Pbkdf2PasswordEncoder());
encoders.put(<span class="hljs-string">"scrypt"</span>, <span class="hljs-keyword">new</span> SCryptPasswordEncoder());
encoders.put(<span class="hljs-string">"sha256"</span>, <span class="hljs-keyword">new</span> StandardPasswordEncoder());
&nbsp;
PasswordEncoder passwordEncoder =
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> DelegatingPasswordEncoder(idForEncode, encoders);
</code></pre>
<p data-nodeid="4882">请注意，在 Spring Security 中，密码的标准存储格式是这样的：</p>
<pre class="lang-java" data-nodeid="4883"><code data-language="java">{id}encodedPassword
</code></pre>
<p data-nodeid="4884">这里的 id 就是 PasswordEncoder 的种类，也就是前面提到的 idForEncode 参数。假设密码原文为“password”，经过 BCryptPasswordEncoder 进行加密之后的密文就变成了这样一个字符串：</p>

<pre class="lang-java" data-nodeid="4886"><code data-language="java">$<span class="hljs-number">2</span>a$<span class="hljs-number">10</span>$dXJ3SW6G7P50lGmMkkmwe<span class="hljs-number">.20</span>cQQubK3.HZWzG3YB1tlRy.fqvM/BG
</code></pre>
<p data-nodeid="5377">最终存储在数据库中的密文应该是这样的：</p>



<pre class="lang-java" data-nodeid="4890"><code data-language="java">{bcrypt}$<span class="hljs-number">2</span>a$<span class="hljs-number">10</span>$dXJ3SW6G7P50lGmMkkmwe<span class="hljs-number">.20</span>cQQubK3.HZWzG3YB1tlRy.fqvM/BG
</code></pre>
<p data-nodeid="4891">以上实现过程可以通过查阅 DelegatingPasswordEncoder 的 encode() 方法得到验证：</p>
<pre class="lang-java" data-nodeid="4892"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">encode</span><span class="hljs-params">(CharSequence rawPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">return</span> PREFIX + <span class="hljs-keyword">this</span>.idForEncode + SUFFIX + <span class="hljs-keyword">this</span>.passwordEncoderForEncode.encode(rawPassword);
}
</code></pre>
<p data-nodeid="4893">我们继续来看 DelegatingPasswordEncoder 的 matcher 方法，如下所示：</p>
<pre class="lang-java" data-nodeid="4894"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">matches</span><span class="hljs-params">(CharSequence rawPassword, String prefixEncodedPassword)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (rawPassword == <span class="hljs-keyword">null</span> &amp;&amp; prefixEncodedPassword == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">true</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//取出 PasswordEncoder 的 id</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String id = extractId(prefixEncodedPassword);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据 PasswordEncoder 的 id 获取对应的 PasswordEncoder</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PasswordEncoder delegate = <span class="hljs-keyword">this</span>.idToPasswordEncoder.get(id);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//如果找不到对应的 PasswordEncoder，则使用默认 PasswordEncoder 进行匹配判断</span>
	<span class="hljs-keyword">if</span> (delegate == <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.defaultPasswordEncoderForMatches
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; .matches(rawPassword, prefixEncodedPassword);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//从存储的密码字符串中抽取密文，去掉 id</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String encodedPassword = extractEncodedPassword(prefixEncodedPassword);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//使用对应 PasswordEncoder 针对密文进行匹配判断</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> delegate.matches(rawPassword, encodedPassword);
}
</code></pre>
<p data-nodeid="4895">上述方法的流程还是很明确的，至此，我们对 DelegatingPasswordEncoder 的实现原理就讲清楚了，也进一步理解了 PasswordEncoder 的使用过程。</p>
<h3 data-nodeid="4896">Spring Security 加密模块</h3>
<p data-nodeid="4897">正如我们在开头介绍的，使用 Spring Security 时，通常涉及用户认证的部分会用到加解密技术。但就应用场景而言，加解密技术是一种通用的基础设施类技术，不仅可以用于用户认证，也可以用于其他任何涉及敏感数据处理的场景。因此，Spring Security 也充分考虑到了这种需求，专门提供了一个加密模式（Spring Security Crypto Module，SSCM）。</p>
<p data-nodeid="4898">请注意，尽管 PasswordEncoder 也属于这个模块的一部分，但这个模块本身是高度独立的，我们可以脱离于用户认证流程来使用这个模块。</p>
<p data-nodeid="4899">Spring Security 加密模块的核心功能有两部分，首先就是加解密器（Encryptors），典型的使用方式如下：</p>
<pre class="lang-java" data-nodeid="4900"><code data-language="java">BytesEncryptor e = Encryptors.standard(password, salt);
</code></pre>
<p data-nodeid="4901">上述方法使用了标准的 256 位 AES 算法对输入的 password 字段进行加密，返回的是一个 BytesEncryptor。同时，我们也看到这里需要输入一个代表盐值的 salt 字段，而这个 salt 值的获取就可以用到 Spring Security 加密模块的另一个功能——键生成器（Key Generators），使用方式如下所示：</p>
<pre class="lang-java" data-nodeid="4902"><code data-language="java">String salt = KeyGenerators.string().generateKey();
</code></pre>
<p data-nodeid="4903">上述键生成器会创建一个 8 字节的密钥，并将其编码为十六进制字符串。</p>
<p data-nodeid="4904">如果将加解密器和键生成器结合起来，我们就可以实现通用的加解密机制，如下所示：</p>
<pre class="lang-java" data-nodeid="4905"><code data-language="java">String salt = KeyGenerators.string().generateKey(); 
String password = <span class="hljs-string">"secret"</span>; 
String valueToEncrypt = <span class="hljs-string">"HELLO"</span>; 
BytesEncryptor e = Encryptors.standard(password, salt); 
<span class="hljs-keyword">byte</span> [] encrypted = e.encrypt(valueToEncrypt.getBytes()); 
<span class="hljs-keyword">byte</span> [] decrypted = e.decrypt(encrypted);
</code></pre>
<p data-nodeid="4906">在日常开发过程中，你可以根据需要调整上述代码并嵌入到我们的系统中。</p>
<h3 data-nodeid="4907">小结与预告</h3>
<p data-nodeid="4908">对于一个 Web 应用程序而言，一旦需要实现用户认证，势必涉及用户密码等敏感信息的加密。为此，Spring Security 专门提供了 PasswordEncoder 组件对密码进行加解密。Spring Security 内置了一批即插即用的 PasswordEncoder，并通过代理机制完成了各个组件的版本兼容和统一管理。这种设计思想也值得我们学习和借鉴。当然，作为一款通用的安全性开发框架，Spring Security 也提供了一个高度独立的加密模块应对日常开发需求。</p>
<p data-nodeid="4909">本讲内容总结如下：</p>
<p data-nodeid="4910"><img src="https://s0.lgstatic.com/i/image6/M00/43/A4/CioPOWC5_7OAC11AAABs6h6B9l8729.png" alt="Drawing 3.png" data-nodeid="5008"></p>
<p data-nodeid="4911">这里给你留一道思考题：你能描述 DelegatingPasswordEncoder 所起到的代理作用吗？欢迎在留言区和我分享你的思考。</p>
<p data-nodeid="4912" class="">介绍完加解密组件的使用方法之后，下一讲我们来到授权流程，讨论如何基于 Spring Security 确保对请求的安全访问。</p>

---

### 精选评论

##### **正：
> DelegatingPasswordEncoder根据密码串的前缀id获取对应的加密算法，然后根据对应的加密算法去检验密码

