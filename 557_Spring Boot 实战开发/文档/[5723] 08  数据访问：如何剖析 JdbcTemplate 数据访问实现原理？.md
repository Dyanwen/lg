<p data-nodeid="20836">07 讲中，我们介绍了使用 JdbcTemplate 模板工具类完成关系型数据库访问的详细实现过程，通过 JdbcTemplate 不仅简化了数据库操作，还避免了使用原生 JDBC 带来的代码复杂度和冗余性问题。</p>
<p data-nodeid="20837">那么，JdbcTemplate 在 JDBC 基础上如何实现封装的呢？今天，我将带领大家从设计思想出发，讨论 JDBC API 到 JdbcTemplate 的演进过程，并剖析 JdbcTemplate 的部分核心源码。</p>
<h3 data-nodeid="20838">从模板方法模式和回调机制说起</h3>
<p data-nodeid="20839">从命名上 JdbcTemplate 显然是一种模板类，这就让我们联想起设计模式中的模板方法模式。为了让你更好地理解 JdbcTemplate 的实现原理，我们先对这一设计模式进行简单说明。</p>
<h4 data-nodeid="20840">模板方法设计模式</h4>
<p data-nodeid="20841">模板方法模式的原理非常简单，它主要是利用了面向对象中类的继承机制，目前应用非常广泛的，且在实现上往往与抽象类一起使用，比如 Spring 框架中也大量应用了模板方法实现基类和子类之间的职责分离和协作。</p>
<p data-nodeid="20842">按照定义，完成一系列步骤时，这些步骤需要遵循统一的工作流程，个别步骤的实现细节除外，这时我们就需要考虑使用模板方法模式处理了。模板方法模式的结构示意图如下所示：</p>
<p data-nodeid="21889"><img src="https://s0.lgstatic.com/i/image/M00/84/15/Ciqc1F_THuiASLZ8AACi3dbo9Ww445.png" alt="图片7.png" data-nodeid="21893"></p>
<div data-nodeid="21890" class=""><p style="text-align:center">模板方法设计模式结构示意图</p></div>




<p data-nodeid="20846">上图中，抽象模板类 AbstractClass 定义了一套工作流程，而具体实现类 ConcreteClassA 和 ConcreteClassB 对工作流程中的某些特定步骤进行了实现。</p>
<h4 data-nodeid="20847">回调机制</h4>
<p data-nodeid="20848">在软件开发过程中，回调（Callback）是一种常见的实现技巧，回调的含义如下图所示：</p>
<p data-nodeid="22473" class=""><img src="https://s0.lgstatic.com/i/image/M00/84/15/Ciqc1F_THviAWcz1AABn8HsEEZA924.png" alt="图片8.png" data-nodeid="22477"></p>
<div data-nodeid="22474"><p style="text-align:center">回调示意图</p></div>




<p data-nodeid="20852">上图中，ClassA 的 operation1() 方法调用 ClassB 的 operation2()  方法，ClassB 的 operation2()  方法执行完毕再主动调用 ClassA 的 callback()  方法，这就是回调机制，体现的是一种双向的调用方式。</p>
<p data-nodeid="20853">从上面描述可以看到，回调在任务执行过程中不会造成任何的阻塞，任务结果一旦就绪，回调就会被执行，显然在方法调用上这是一种异步执行的方式。同时，回调还是实现扩展性的一种简单而直接的模式。</p>
<p data-nodeid="20854">在上图中，我们看到执行回调时，代码会从一个类中的某个方法跳到另一个类中的某个方法，这种思想同样可以扩展到组件级别，即代码从一个组件跳转到另一个组件。只要预留回调的契约，原则上我们可以实现运行时根据调用关系动态来实现组件之间的跳转，从而满足扩展性的要求。</p>
<p data-nodeid="20855">事实上，JdbcTemplate 正是基于模板方法模式和回调机制，才真正解决了原生 JDBC 中的复杂性问题。</p>
<p data-nodeid="20856">接下来，我们结合 07 讲中给出的 SpringCSS 案例场景从 JDBC 的原生 API 出发，讨论 JdbcTemplate 的演进过程。</p>
<h3 data-nodeid="20857">JDBC API 到 JdbcTemplate 的演变</h3>
<p data-nodeid="20858">在 06《基础规范：如何理解 JDBC 关系型数据库访问规范？》讲中，我们给出了基于 JDBC 原生 API 访问数据库的开发过程，那如何完成 JDBC 到JdbcTemplate 呢？</p>
<p data-nodeid="20859">在整个过程中，我们不难发现创建 DataSource、获取 Connection、创建 Statement 等步骤实际上都是重复的，只有处理 ResultSet 部分需要我们针对不同的 SQL 语句和结果进行定制化处理，因为每个结果集与业务实体之间的对应关系不同。</p>
<p data-nodeid="20860">这样，我们的思路就来了，首先我们想到的是如何构建一个抽象类实现模板方法。</p>
<h4 data-nodeid="20861">在 JDBC API 中添加模板方法模式</h4>
<p data-nodeid="20862">假设我们将这个抽象类命名为 AbstractJdbcTemplate，那么该类的代码结构应该是这样的：</p>
<pre class="lang-java" data-nodeid="20863"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">AbstractJdbcTemplate</span></span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> DataSource dataSource;
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Object <span class="hljs-title">execute</span><span class="hljs-params">(String sql)</span> </span>{&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection connection = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Statement statement = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet resultSet = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connection = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement = connection.createStatement();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resultSet = statement.executeQuery(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object object = handleResultSet(resultSet);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> object;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (SQLException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.print(e);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (resultSet != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resultSet.close();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (SQLException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (statement != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement.close();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (SQLException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (connection != <span class="hljs-keyword">null</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connection.close();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (SQLException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">abstract</span> Object <span class="hljs-title">handleResultSet</span><span class="hljs-params">(ResultSet rs)</span> <span class="hljs-keyword">throws</span> SQLException</span>;&nbsp; 
}
</code></pre>
<p data-nodeid="20864">AbstractJdbcTemplate 是一个抽象类，而 execute 方法主体代码我们都很熟悉，基本照搬了07 讲所构建的 OrderRawJdbcRepository 类中的代码。<strong data-nodeid="20943">唯一需要注意的是，这里出现了一个模板方法 handleResultSet 用来处理 ResultSet。</strong></p>
<p data-nodeid="20865">下面我们再构建一个 AbstractJdbcTemplate 的实现类 OrderJdbcTemplate，如下代码所示：</p>
<pre class="lang-java" data-nodeid="20866"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderJdbcTemplate</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">AbstractJdbcTemplate</span></span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">protected</span> Object <span class="hljs-title">handleResultSet</span><span class="hljs-params">(ResultSet rs)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Order&gt; orders = <span class="hljs-keyword">new</span> ArrayList&lt;Order&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (rs.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = <span class="hljs-keyword">new</span> Order(rs.getLong(<span class="hljs-string">"id"</span>), rs.getString(<span class="hljs-string">"order_number"</span>), rs.getString(<span class="hljs-string">"delivery_address"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orders.add(order);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orders;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="20867">显然，这里我们获取了 ResultSet 中的结果，并构建了业务对象进行返回。</p>
<p data-nodeid="20868">我们再使用 OrderJdbcTemplate 执行目标 SQL 语句，如下代码所示：</p>
<pre class="lang-java" data-nodeid="20869"><code data-language="java">AbstractJdbcTemplate jdbcTemplate = <span class="hljs-keyword">new</span> OrderJdbcTemplate();&nbsp; 
List&lt;Order&gt; orders = (List&lt;Order&gt;) jdbcTemplate.execute(<span class="hljs-string">"select * from Order"</span>);
</code></pre>
<p data-nodeid="20870">就这样，一个添加了模板方法模式的 JdbcTemplate 就构建完成了，是不是很简单？接下来，我们继续讨论如何对它进行进一步改进。</p>
<h4 data-nodeid="20871">在 JDBC API 中添加回调机制</h4>
<p data-nodeid="20872">试想一下，如果我们需要对各种业务对象实现数据库操作，那势必需要提供各种类似 OrderJdbcTemplate 这样的实现类，这点显然很不方便。一方面我们需要创建和维护一批新类，另一方面如果抽象方法数量很多，子类就需要实现所有抽象方法，尽管有些方法中子类并不会用到，这时该如何解决呢？</p>
<p data-nodeid="20873">实际上，这种问题本质在于我们使用了抽象类。如果我们不想使用抽象类，则可以引入前面介绍的回调机制。使用回调机制的第一步，先定义一个回调接口来剥离业务逻辑，我们将其命名为 StatementCallback，如下代码所示：</p>
<pre class="lang-java" data-nodeid="20874"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">StatementCallback</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function">Object <span class="hljs-title">handleStatement</span><span class="hljs-params">(Statement statement)</span> <span class="hljs-keyword">throws</span> SQLException</span>;&nbsp; 
}
</code></pre>
<p data-nodeid="20875">然后，我们重新创建一个新的 CallbackJdbcTemplate 用来执行数据库访问，如下代码所示:</p>
<pre class="lang-java" data-nodeid="20876"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CallbackJdbcTemplate</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> DataSource dataSource;
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> Object <span class="hljs-title">execute</span><span class="hljs-params">(StatementCallback callback)</span></span>{&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection connection = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Statement statement = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet resultSet = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connection = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement = connection.createStatement();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Object object = callback.handleStatement(statement);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> object;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">catch</span> (SQLException e) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; System.out.print(e);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略异常处理</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="22758" class=""><strong data-nodeid="22763">注意，与 AbstractJdbcTemplate 类相比，CallbackJdbcTemplate 存在两处差异点。</strong> 首先，CallbackJdbcTemplate 不是一个抽象类。其次，execute 方法签名上传入的是一个 StatementCallback 对象，而具体的定制化处理是通过 Statement 传入到 Callback 对象中完成的，我们也可以认为是把原有需要子类抽象方法实现的功能转嫁到了 StatementCallback 对象上。</p>

<p data-nodeid="20878">基于 CallbackJdbcTemplate 和 StatementCallback，我们可以构建具体数据库访问的执行流程，如下代码所示：</p>
<pre class="lang-java" data-nodeid="20879"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">queryOrder</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span>&nbsp; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderStatementCallback</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">StatementCallback</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">handleStatement</span><span class="hljs-params">(Statement statement)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet rs = statement.executeQuery(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Order&gt; orders = <span class="hljs-keyword">new</span> ArrayList&lt;Order&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (rs.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = <span class="hljs-keyword">new</span> Order(rs.getLong(<span class="hljs-string">"id"</span>), rs.getString(<span class="hljs-string">"order_number"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rs.getString(<span class="hljs-string">"delivery_address"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orders.add(order);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orders;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CallbackJdbcTemplate jdbcTemplate = <span class="hljs-keyword">new</span> CallbackJdbcTemplate();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> jdbcTemplate.execute(<span class="hljs-keyword">new</span> OrderStatementCallback());
}
</code></pre>
<p data-nodeid="20880">这里，我们定义了一个 queryOrder 方法并传入 SQL 语句中用来实现对 Order 表的查询。</p>
<p data-nodeid="20881">注意到，在 queryOrder 方法中我们构建了一个 OrderStatementCallback 内部类，该类实现了 StatementCallback 接口并提供了具体操作 SQL 的定制化代码。然后我们创建了一个 CallbackJdbcTemplate 对象并将内部类 OrderStatementCallback 传入该对象的 execute 方法中。</p>
<p data-nodeid="20882">针对这种场景，实际上我们也可以不创建 OrderStatementCallback 内部类，因为该类只适用于这个场景中，此时更为简单的处理方法是使用匿名类，如下代码所示：</p>
<pre class="lang-java" data-nodeid="20883"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">queryOrder</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span>&nbsp; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CallbackJdbcTemplate jdbcTemplate = <span class="hljs-keyword">new</span> CallbackJdbcTemplate();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> jdbcTemplate.execute(<span class="hljs-keyword">new</span> StatementCallback() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">handleStatement</span><span class="hljs-params">(Statement statement)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet rs = statement.executeQuery(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Order&gt; orders = <span class="hljs-keyword">new</span> ArrayList&lt;Order&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (rs.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = <span class="hljs-keyword">new</span> Order(rs.getLong(<span class="hljs-string">"id"</span>), rs.getString(<span class="hljs-string">"order_number"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rs.getString(<span class="hljs-string">"delivery_address"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orders.add(order);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orders;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; });&nbsp;&nbsp;&nbsp;&nbsp; 
}
</code></pre>
<p data-nodeid="20884">匿名类的实现方式比较简洁点，且在日常开发过程中，我们也经常使用这种方式实现回调接口。</p>
<h3 data-nodeid="20885">JdbcTemplate 源码解析</h3>
<p data-nodeid="20886">理解了 JDBC API 到 JdbcTemplate 的演变过程，接下来我们真正进入 Spring Boot 所提供的 JdbcTemplate 模板工具类的源码部分，看看它是否采用了这种设计思路。</p>
<p data-nodeid="20887">我们直接看 JdbcTemplate 的 execute(StatementCallback<t> action)  方法，如下代码所示：</t></p>
<pre class="lang-java" data-nodeid="20888"><code data-language="java"><span class="hljs-keyword">public</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">execute</span><span class="hljs-params">(StatementCallback&lt;T&gt; action)</span> <span class="hljs-keyword">throws</span> DataAccessException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Assert.notNull(action, <span class="hljs-string">"Callback object must not be null"</span>);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection con = DataSourceUtils.getConnection(obtainDataSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Statement stmt = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; stmt = con.createStatement();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; applyStatementSettings(stmt);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; T result = action.doInStatement(stmt);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; handleWarnings(stmt);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">catch</span> (SQLException ex) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; String sql = getSql(action);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JdbcUtils.closeStatement(stmt);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; stmt = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataSourceUtils.releaseConnection(con, getDataSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; con = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">throw</span> translateException(<span class="hljs-string">"StatementCallback"</span>, sql, ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">finally</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; JdbcUtils.closeStatement(stmt);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DataSourceUtils.releaseConnection(con, getDataSource());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="20889">从以上代码中可以看出，execute 方法中同样接收了一个 StatementCallback 回调接口，然后通过传入 Statement 对象完成 SQL 语句的执行，这与前面我们给出的实现方法完全一致。</p>
<p data-nodeid="20890">StatementCallback 回调接口定义代码如下：</p>
<pre class="lang-java" data-nodeid="20891"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">StatementCallback</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">T <span class="hljs-title">doInStatement</span><span class="hljs-params">(Statement stmt)</span> <span class="hljs-keyword">throws</span> SQLException, DataAccessException</span>;
}
</code></pre>
<p data-nodeid="20892">同样，我们发现 StatementCallback 回调接口的定义也很类似。我们来看看上述 execute(StatementCallback<t> action)  方法的具体使用方法。</t></p>
<p data-nodeid="20893">**事实上，在 JdbcTemplate 中，还存在另一个 execute(final String sql) 方法，该方法中恰恰使用了 execute(StatementCallback<t> action)  方法，**如下代码所示：</t></p>
<pre class="lang-java" data-nodeid="20894"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">execute</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> DataAccessException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (logger.isDebugEnabled()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; logger.debug(<span class="hljs-string">"Executing SQL statement ["</span> + sql + <span class="hljs-string">"]"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ExecuteStatementCallback</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">StatementCallback</span>&lt;<span class="hljs-title">Object</span>&gt;, <span class="hljs-title">SqlProvider</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Nullable</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Object <span class="hljs-title">doInStatement</span><span class="hljs-params">(Statement stmt)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; stmt.execute(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getSql</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> sql;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; execute(<span class="hljs-keyword">new</span> ExecuteStatementCallback());
}
</code></pre>
<p data-nodeid="23046" class="te-preview-highlight">这里，我们同样采用了内部类的实现方式创建 StatementCallback 回调接口的实现类 ExecuteStatementCallback，然后通过传入 Statement 对象完成 SQL 语句的执行，最后通过调用 execute(StatementCallback<code data-backticks="1" data-nodeid="23048">&lt;T&gt;</code> action) 方法实现整个执行过程。</p>

<h3 data-nodeid="20896">从源码解析到日常开发</h3>
<p data-nodeid="20897">今天的内容与其说在讲 JdbcTemplate 的源码，不如说在剖析该类背后的设计思想，因此 08 讲中的很多知识点和实现方法都可以应用到日常开发过程中。</p>
<p data-nodeid="20898">无论是模板方法还是回调机制，在技术实现上都没有难度，有难度的是应用的场景以及对问题的抽象。JdbcTemplate 基于 JDBC 的原生 API，把模板方法和回调机制结合在了一起，为我们提供了简洁且高扩展的实现方案，值得我们分析和应用。</p>
<h3 data-nodeid="20899">小结与预告</h3>
<p data-nodeid="20900">JdbcTemplate 是 Spring 中非常具有代表性的一个模板工具类。今天的课程中，我们从现实应用场景出发，系统分析了原始 JDBC 规范到 JdbcTemplate 的演进过程，并给出了模板方法设计模式和回调机制在这一过程中所发挥的作用。我们先提供了 JdbcTemplate 的初步实现方案，然后结合 Spring Boot中 的 JdbcTemplate 源码做了类比。</p>
<p data-nodeid="20901">这里给你留一道思考题：在 JdbcTemplate 的构建过程中，模板方法设计模式和回调机制分别发挥了什么作用？</p>
<p data-nodeid="20902">介绍完了 JdbcTemplate 后，我们将采用另一种技术体系实现数据访问，这就是 Spring 家族所提供的 Spring Data 框架。09 讲我们将分析 Spring Data 如何对数据访问过程进行统一抽象。</p>

---

### 精选评论

##### *旺：
> 可以预留抽象方法在子类中实现，也可以传入回调接口通过接口的不同实现完成

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的

