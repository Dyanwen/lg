<p data-nodeid="10073" class=""><strong data-nodeid="10142">StatementHandler 接口是 MyBatis 中非常重要的一个接口</strong>，其实现类完成 SQL 语句执行中最核心的一系列操作，这也是后面我们要介绍的 Executor 接口实现的基础。</p>
<p data-nodeid="10074">StatementHandler 接口的定义如下图所示：</p>
<p data-nodeid="10075"><img src="https://s0.lgstatic.com/i/image6/M00/1F/5B/Cgp9HWBRyPSAPnqwAADa0tXnYqQ008.png" alt="Drawing 0.png" data-nodeid="10146"></p>
<div data-nodeid="10076"><p style="text-align:center">StatementHandler 接口中定义的方法</p></div>
<p data-nodeid="10077">可以看到，其中提供了创建 Statement 对象（prepare() 方法）、为 SQL 语句绑定实参（parameterize() 方法）、执行单条 SQL 语句（query() 方法和 update() 方法）、批量执行 SQL 语句（batch() 方法）等多种功能。</p>
<p data-nodeid="10078">下图展示了 MyBatis 中提供的所有 StatementHandler 接口实现类，以及它们的继承关系：</p>
<p data-nodeid="10079"><img src="https://s0.lgstatic.com/i/image6/M00/1F/64/Cgp9HWBR0IaAXG4BAAGLvM_7w1Y255.png" alt="图片3.png" data-nodeid="10151"></p>
<div data-nodeid="10080"><p style="text-align:center">StatementHandler 接口继承关系图</p></div>
<p data-nodeid="10081">今天这一讲我们就来详细分析该继承关系图中每个 StatementHandler 实现的核心逻辑。</p>
<h3 data-nodeid="10082">RoutingStatementHandler</h3>
<p data-nodeid="10083">RoutingStatementHandler 这个 StatementHandler 实现，有点<strong data-nodeid="10159">策略模式</strong>的意味。在 RoutingStatementHandler 的构造方法中，会根据 MappedStatement 中的 statementType 字段值，选择相应的 StatementHandler 实现进行创建，这个新建的 StatementHandler 对象由 RoutingStatementHandler 中的 delegate 字段维护。</p>
<p data-nodeid="10084">RoutingStatementHandler 的构造方法如下：</p>
<pre class="lang-java" data-nodeid="10085"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">RoutingStatementHandler</span><span class="hljs-params">(Executor executor, MappedStatement ms,&nbsp;Object parameter, RowBounds rowBounds, ResultHandler resultHandler, BoundSql boundSql)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 下面就是根据MappedStatement的配置，生成一个相应的StatementHandler对</span>
&nbsp; &nbsp; <span class="hljs-comment">//&nbsp;象，并设置到delegate字段中维护</span>
&nbsp; &nbsp; <span class="hljs-keyword">switch</span> (ms.getStatementType()) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> STATEMENT:
            <span class="hljs-comment">// 创建SimpleStatementHandler对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; delegate = <span class="hljs-keyword">new</span> SimpleStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> PREPARED:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建PreparedStatementHandler对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; delegate = <span class="hljs-keyword">new</span> PreparedStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">case</span> CALLABLE:
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建CallableStatementHandler对象</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; delegate = <span class="hljs-keyword">new</span> CallableStatementHandler(executor, ms, parameter, rowBounds, resultHandler, boundSql);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">break</span>;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">default</span>: <span class="hljs-comment">// 抛出异常</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ExecutorException(<span class="hljs-string">"..."</span>);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="10086">在 RoutingStatementHandler 的其他方法中，<strong data-nodeid="10166">都会委托给底层的 delegate 对象来完成具体的逻辑</strong>。</p>
<h3 data-nodeid="10087">BaseStatementHandler</h3>
<p data-nodeid="10088">作为一个抽象类，BaseStatementHandler 只实现了 StatementHandler 接口的 prepare() 方法，其 prepare() 方法实现为新建的 Statement 对象设置了一些参数，例如，timeout、fetchSize 等。BaseStatementHandler 还新增了一个 instantiateStatement() 抽象方法给子类实现，来完成 Statement 对象的其他初始化操作。不过，BaseStatementHandler 中并没有实现 StatementHandler 接口中的数据库操作等核心方法。</p>
<p data-nodeid="12606" class="te-preview-highlight">了解了 BaseStatementHandler 对 StatementHandler 接口的实现情况之后，我们再来看一下 BaseStatementHandler 的构造方法，其中会<strong data-nodeid="12626">初始化执行 SQL 需要的 Executor 对象</strong>、<strong data-nodeid="12627">为 SQL 绑定实参的 ParameterHandler 对象</strong>以及<strong data-nodeid="12628">生成结果对象的 ResultSetHandler 对象</strong>。这三个核心对象中，ResultSetHandler 对象我们已经在<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6385" data-nodeid="12624">《14 | 探究 MyBatis 结果集映射机制背后的秘密（上）》</a>中介绍过了，ParameterHandler 和 Executor 在后面会展开介绍。</p>





<h4 data-nodeid="10090">1. KeyGenerator</h4>
<p data-nodeid="10091">这里需要关注的是 generateKeys() 方法，其中会<strong data-nodeid="10195">通过 KeyGenerator 接口生成主键</strong>，下面我们就来看看 KeyGenerator 接口的相关内容。</p>
<p data-nodeid="10092">我们知道不同数据库的自增 id 生成策略并不完全一样。例如，我们常见的 Oracle DB 是通过sequence 实现自增 id 的，如果使用自增 id 作为主键，就需要我们先获取到这个自增的 id 值，然后再使用；MySQL 在使用自增 id 作为主键的时候，insert 语句中可以不指定主键，在插入过程中由 MySQL 自动生成 id。KeyGenerator 接口支持 insert 语句执行前后获取自增的 id，分别对应 processBefore() 方法和 processAfter() 方法，下图展示了 MyBatis 提供的两个 KeyGenerator 接口实现：</p>
<p data-nodeid="10093"><img src="https://s0.lgstatic.com/i/image6/M00/1F/64/Cgp9HWBR0HmAAYIQAAE2FEB8sfU102.png" alt="图片4.png" data-nodeid="10199"></p>
<div data-nodeid="10094"><p style="text-align:center">KeyGenerator 接口继承关系图</p></div>
<p data-nodeid="10292" class=""><strong data-nodeid="10297">Jdbc3KeyGenerator 用于获取数据库生成的自增 id（例如 MySQL 那种生成模式）</strong>，其 processBefore() 方法是空实现，processAfter() 方法会将 insert 语句执行后生成的主键保存到用户传递的实参中。我们在使用 MyBatis 执行单行 insert 语句时，一般传入的实参是一个 POJO 对象或是 Map 对象，生成的主键会设置到对应的属性中；执行多条 insert 语句时，一般传入实参是 POJO 对象集合或 Map 对象的数组或集合，集合中每一个元素都对应一次插入操作，生成的多个自增 id 也会设置到每个元素的相应属性中。</p>

<p data-nodeid="10096">Jdbc3KeyGenerator 中获取数据库自增 id 的核心代码片段如下：</p>
<pre class="lang-java" data-nodeid="10097"><code data-language="java"><span class="hljs-comment">// 将数据库生成的自增id作为结果集返回</span>
<span class="hljs-keyword">try</span> (ResultSet rs = stmt.getGeneratedKeys()) {&nbsp;
&nbsp; &nbsp; <span class="hljs-keyword">final</span> ResultSetMetaData rsmd = rs.getMetaData();
&nbsp; &nbsp; <span class="hljs-keyword">final</span> Configuration configuration = ms.getConfiguration();
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (rsmd.getColumnCount() &lt; keyProperties.length) {
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 处理rs这个结果集，将生成的id设置到对应的属性中</span>
&nbsp; &nbsp; &nbsp; &nbsp; assignKeys(configuration, rs, rsmd, keyProperties, parameter);
&nbsp; &nbsp; }
} <span class="hljs-keyword">catch</span> (Exception e) {
&nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ExecutorException(<span class="hljs-string">"..."</span>);
}
</code></pre>
<p data-nodeid="10098"><strong data-nodeid="10212">如果使用像 Oracle 这种不支持自动生成主键自增 id 的数据库时，我们可以使用 SelectkeyGenerator 来生成主键 id</strong>。Mapper 映射文件中的<code data-backticks="1" data-nodeid="10210">&lt;selectKey&gt;</code>标签会被解析成 SelectkeyGenerator 对象，其中的 executeBefore 属性（boolean 类型）决定了是在 insert 语句执行之前获取主键，还是在 insert 语句执行之后获取主键 id。</p>
<p data-nodeid="10099">SelectkeyGenerator 中的 processBefore() 方法和 processAfter() 方法都是通过 processGeneratedKeys() 这个私有方法获取主键 id 的，processGeneratedKeys() 方法会执行<code data-backticks="1" data-nodeid="10214">&lt;selectKey&gt;</code>标签中指定的 select 语句，查询主键信息，并记录到用户传入的实参对象的对应属性中，核心代码片段如下所示：<br>
<br></p>
<pre class="lang-java" data-nodeid="10100"><code data-language="java"><span class="hljs-comment">// 创建一个新的Executor对象来执行指定的select语句</span>
Executor keyExecutor = configuration.newExecutor(executor.getTransaction(), ExecutorType.SIMPLE);
<span class="hljs-comment">// 拿到主键信息</span>
List&lt;Object&gt; values = keyExecutor.query(keyStatement, parameter, RowBounds.DEFAULT, Executor.NO_RESULT_HANDLER);
<span class="hljs-keyword">if</span> (values.size() == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ExecutorException(<span class="hljs-string">"SelectKey returned no data."</span>);
} <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (values.size() &gt; <span class="hljs-number">1</span>) {
&nbsp; &nbsp; <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> ExecutorException(<span class="hljs-string">"SelectKey returned more than one value."</span>);
} <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; <span class="hljs-comment">// 创建实参对象的MetaObject对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">final</span> MetaObject metaParam = configuration.newMetaObject(parameter);
&nbsp; &nbsp; MetaObject metaResult = configuration.newMetaObject(values.get(<span class="hljs-number">0</span>));
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (keyProperties.length == <span class="hljs-number">1</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 将主键信息记录到用户传入的实参对象中</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (metaResult.hasGetter(keyProperties[<span class="hljs-number">0</span>])) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; setValue(metaParam, keyProperties[<span class="hljs-number">0</span>], metaResult.getValue(keyProperties[<span class="hljs-number">0</span>]));
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; setValue(metaParam, keyProperties[<span class="hljs-number">0</span>], values.get(<span class="hljs-number">0</span>));
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
        ... <span class="hljs-comment">// 多结果集的处理</span>
&nbsp; &nbsp; }
}
</code></pre>
<h4 data-nodeid="10101">2. ParameterHandler</h4>
<p data-nodeid="10102">介绍完 KeyGenerator 接口之后，我们再来看一下 BaseStatementHandler 中依赖的另一个辅助类—— ParameterHandler。</p>
<p data-nodeid="10103">经过前面<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=612&amp;sid=20-h5Url-0&amp;buyFrom=2&amp;pageId=1pz4#/detail/pc?id=6384&amp;fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="10227">《13 | 深入分析动态 SQL 语句解析全流程（下）》</a>介绍的一系列 SqlNode 的处理之后，我们得到的 SQL 语句（维护在 BoundSql 对象中）可能包含多个“?”占位符，与此同时，用于替换每个“?”占位符的实参都记录在 BoundSql.parameterMappings 集合中。</p>
<p data-nodeid="10104">ParameterHandler 接口中定义了两个方法：一个是 getParameterObject() 方法，用来获取传入的实参对象；另一个是 setParameters() 方法，用来替换“?”占位符，这是 ParameterHandler 的<strong data-nodeid="10234">核心方法</strong>。</p>
<p data-nodeid="10105"><strong data-nodeid="10241">DefaultParameterHandler 是 ParameterHandler 接口的唯一实现，其 setParameters() 方法会遍历 BoundSql.parameterMappings 集合</strong>，根据参数名称查找相应实参，最后会通过 PreparedStatement.set*() 方法与 SQL 语句进行绑定。setParameters() 方法的具体代码如下：</p>
<pre class="lang-java" data-nodeid="10106"><code data-language="java"><span class="hljs-keyword">for</span> (<span class="hljs-keyword">int</span> i = <span class="hljs-number">0</span>; i &lt; parameterMappings.size(); i++) {
&nbsp; &nbsp; ParameterMapping parameterMapping = parameterMappings.get(i);
&nbsp; &nbsp; Object value;
&nbsp; &nbsp; String propertyName = parameterMapping.getProperty();
&nbsp; &nbsp; <span class="hljs-comment">// 获取实参值</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (boundSql.hasAdditionalParameter(propertyName)) {
&nbsp; &nbsp; &nbsp; &nbsp; value = boundSql.getAdditionalParameter(propertyName);
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (parameterObject == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; value = <span class="hljs-keyword">null</span>;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
&nbsp; &nbsp; &nbsp; &nbsp; value = parameterObject;
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; MetaObject metaObject = configuration.newMetaObject(parameterObject);
&nbsp; &nbsp; &nbsp; &nbsp; value = metaObject.getValue(propertyName);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 获取TypeHandler</span>
&nbsp; &nbsp; TypeHandler typeHandler = parameterMapping.getTypeHandler();
&nbsp; &nbsp; JdbcType jdbcType = parameterMapping.getJdbcType();
&nbsp; &nbsp; <span class="hljs-comment">// 底层会调用PreparedStatement.set*()方法完成绑定</span>
&nbsp; &nbsp; typeHandler.setParameter(ps, i + <span class="hljs-number">1</span>, value, jdbcType);
}
</code></pre>
<h3 data-nodeid="10107">SimpleStatementHandler</h3>
<p data-nodeid="10108">SimpleStatementHandler 是 StatementHandler 的具体实现之一，继承了 BaseStatementHandler 抽象类。SimpleStatementHandler 各个方法接收的是 java.sql.Statement 对象，并通过该对象来完成 CRUD 操作，所以在 SimpleStatementHandler 中<strong data-nodeid="10248">维护的 SQL 语句不能存在“?”占位符</strong>，填充占位符的 parameterize() 方法也是空实现。</p>
<p data-nodeid="10109">在 instantiateStatement() 这个初始化方法中，SimpleStatementHandler 会直接通过 JDBC Connection 创建 Statement 对象，这个对象也是后续 SimpleStatementHandler 其他方法的入参。</p>
<p data-nodeid="10110">在 query() 方法实现中，SimpleStatementHandler 会直接通过上面创建的 Statement 对象，执行 SQL 语句，返回的结果集由 ResultSetHandler 完成映射，核心代码如下：</p>
<pre class="lang-java" data-nodeid="10111"><code data-language="java"><span class="hljs-keyword">public</span> &lt;E&gt; <span class="hljs-function">List&lt;E&gt; <span class="hljs-title">query</span><span class="hljs-params">(Statement statement, ResultHandler resultHandler)</span> <span class="hljs-keyword">throws</span> SQLException </span>{、
&nbsp; &nbsp; <span class="hljs-comment">// 获取SQL语句</span>
&nbsp; &nbsp; String sql = boundSql.getSql();
&nbsp; &nbsp; <span class="hljs-comment">// 执行SQL语句</span>
&nbsp; &nbsp; statement.execute(sql);
&nbsp; &nbsp; <span class="hljs-comment">// 处理ResultSet映射，得到结果对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> resultSetHandler.handleResultSets(statement);
}
</code></pre>
<p data-nodeid="10112">queryCursor() 方法与 query() 方法实现类似，这里就不再赘述。</p>
<p data-nodeid="10113">batch() 方法调用的是 Statement.addBatch() 方法添加批量执行的 SQL 语句，但并不是立即执行，而是等待 Statement.executeBatch() 方法执行时才会批量执行，这点你稍微注意一下即可。</p>
<p data-nodeid="10114">至于 update() 方法，首先会通过 Statement.execute() 方法执行 insert、update 或 delete 类型的 SQL 语句，然后执行 KeyGenerator.processAfter() 方法查询主键并填充相应属性（processBefore() 方法已经在 prepare() 方法中执行过了），最后通过 Statement.getUpdateCount() 方法获取 SQL 语句影响的行数并返回。</p>
<h3 data-nodeid="10115">PreparedStatementHandler</h3>
<p data-nodeid="10116">PreparedStatementHandler 是 StatementHandler 的具体实现之一，也是最常用的 StatementHandler 实现，它同样继承了 BaseStatementHandler 抽象类。PreparedStatementHandler 各个方法接收的是 java.sql.PreparedStatement 对象，并通过该对象来完成 CRUD 操作，在其 parameterize() 方法中会通过前面介绍的 ParameterHandler调用 PreparedStatement.set*() 方法为 SQL 语句绑定参数，所以在 PreparedStatementHandler 中<strong data-nodeid="10262">维护的 SQL 语句是可以包含“?”占位符的</strong>。</p>
<p data-nodeid="10117">在 instantiateStatement() 方法中，PreparedStatementHandler 会直接通过 JDBC Connection 的 prepareStatement() 方法创建 PreparedStatement 对象，该对象就是 PreparedStatementHandler 其他方法的入参。</p>
<p data-nodeid="10118">PreparedStatementHandler 的 query() 方法、batch() 方法以及 update() 方法与 SimpleStatementHandler 的实现基本相同，只不过是把 Statement API 换成了 PrepareStatement API 而已。下面我们以 update() 方法为例进行简单介绍：</p>
<pre class="lang-java" data-nodeid="10119"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">int</span> <span class="hljs-title">update</span><span class="hljs-params">(Statement statement)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; PreparedStatement ps = (PreparedStatement) statement;
&nbsp; &nbsp; ps.execute(); <span class="hljs-comment">// 执行SQL语句，修改数据</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> rows = ps.getUpdateCount(); <span class="hljs-comment">// 获取影响行数</span>
&nbsp; &nbsp; <span class="hljs-comment">// 获取实参对象</span>
&nbsp; &nbsp; Object parameterObject = boundSql.getParameterObject();
&nbsp; &nbsp; <span class="hljs-comment">// 执行KeyGenerator</span>
&nbsp; &nbsp; KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
&nbsp; &nbsp; keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> rows; <span class="hljs-comment">// 返回影响行数</span>
}
</code></pre>
<h3 data-nodeid="10120">CallableStatementHandler</h3>
<p data-nodeid="10121"><strong data-nodeid="10270">CallableStatementHandler 是处理存储过程的 StatementHandler 实现</strong>，其 instantiateStatement() 方法会通过 JDBC Connection 的 prepareCall() 方法为指定存储过程创建对应的 java.sql.CallableStatement 对象。在 parameterize() 方法中，CallableStatementHandler 除了会通过 ParameterHandler 完成实参的绑定之外，还会指定输出参数的位置和类型。</p>
<p data-nodeid="10122">在 CallableStatementHandler 的 query()、queryCursor()、update() 方法中，除了处理 SQL 语句本身的结果集（ResultSet 结果集或是影响行数），还会通过 ResultSetHandler 的 handleOutputParameters() 方法处理输出参数，这是与 PreparedStatementHandler 最大的不同。下面我们以 query() 方法为例进行简单分析：</p>
<pre class="lang-java" data-nodeid="10123"><code data-language="java"><span class="hljs-keyword">public</span> &lt;E&gt; <span class="hljs-function">List&lt;E&gt; <span class="hljs-title">query</span><span class="hljs-params">(Statement statement, ResultHandler resultHandler)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp; &nbsp; CallableStatement cs = (CallableStatement) statement;
&nbsp; &nbsp; cs.execute(); <span class="hljs-comment">// 执行存储过程</span>
&nbsp; &nbsp; <span class="hljs-comment">// 处理存储过程返回的结果集</span>
&nbsp; &nbsp; List&lt;E&gt; resultList = resultSetHandler.handleResultSets(cs);
&nbsp; &nbsp; <span class="hljs-comment">// 处理输出参数，可能修改resultList集合</span>
&nbsp; &nbsp; resultSetHandler.handleOutputParameters(cs);
&nbsp; &nbsp; <span class="hljs-comment">// 返回最后的结果对象</span>
&nbsp; &nbsp; <span class="hljs-keyword">return</span> resultList;
}
</code></pre>
<h3 data-nodeid="10124">总结</h3>
<p data-nodeid="10125">这一讲我们重点讲解了 MyBatis 中的 StatementHandler 接口及其核心实现，StatementHandler 接口中定义了执行一条 SQL 语句的核心方法。</p>
<ul data-nodeid="10126">
<li data-nodeid="10127">
<p data-nodeid="10128">首先，分析了 RoutingStatementHandler 实现，它可以帮助我们选择真正的 StatementHandler 实现类。</p>
</li>
<li data-nodeid="10129">
<p data-nodeid="10130">接下来，介绍了 BaseStatementHandler 这个抽象类的实现，同时还详细阐述了其中使用到的 KeyGenerator 和 ParameterHandler。</p>
</li>
<li data-nodeid="10131">
<p data-nodeid="10132">最后，又介绍了 SimpleStatementHandler、PreparedStatementHandler 等实现，它们基于 JDBC API 接口，实现了完整的 StatementHandler 功能。</p>
</li>
</ul>
<p data-nodeid="10133">下一讲，我们将开始讲解 MyBatis 中另一个核心接口—— Executor 接口，记得按时来听课。</p>
<hr data-nodeid="10134">
<p data-nodeid="10135"><a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="10282"><img src="https://s0.lgstatic.com/i/image/M00/6D/3E/CgqCHl-s60-AC0B_AAhXSgFweBY762.png" alt="1.png" data-nodeid="10281"></a></p>
<p data-nodeid="10136"><strong data-nodeid="10286">《Java 工程师高薪训练营》</strong></p>
<p data-nodeid="10137" class="">实战训练+面试模拟+大厂内推，想要提升技术能力，进大厂拿高薪，<a href="https://shenceyun.lagou.com/t/Mka" data-nodeid="10290">点击链接，提升自己</a>！</p>

---

### 精选评论


