<p data-nodeid="83224" class="">在上一课时中，我详细介绍了 ShardingSphere 与 JDBC 规范之间的兼容性关系，我们知道 ShardingSphere 对 JDBC 规范进行了重写，并嵌入了分片机制。基于这种兼容性，开发人员使用 ShardingSphere 时就像在使用 JDBC 规范所暴露的各个接口一样。这一课时，我们将讨论如何在业务系统中使用 ShardingSphere 的具体方式。</p>
<h3 data-nodeid="83225">如何抽象开源框架的应用方式？</h3>
<p data-nodeid="83226">当我们自己在设计和实现一款开源框架时，如何规划它的应用方式呢？作为一款与数据库访问相关的开源框架，ShardingSphere 提供了多个维度的应用方式，我们可以对这些应用方式进行抽象，从而提炼出一种模版。这个模版由四个维度组成，分别是<strong data-nodeid="83318">底层工具、基础规范、开发框架和领域框架</strong>，如下图所示：</p>
<p data-nodeid="83227"><img src="https://s0.lgstatic.com/i/image/M00/28/F7/CgqCHl75qv-AFbZvAACz7F_yXRM280.png" alt="2.png" data-nodeid="83321"></p>
<h4 data-nodeid="83228">底层工具</h4>
<p data-nodeid="83229">底层工具指的是这个开源框架所面向的目标工具或所依赖的第三方工具。这种底层工具往往不是框架本身可以控制和管理的，框架的作用只是在它上面添加一个应用层，用于封装对这些底层工具的使用方式。</p>
<p data-nodeid="83230">对于 ShardingSphere 而言，<strong data-nodeid="83329">这里所说的底层工具实际上指的是关系型数据库</strong>。目前，ShardingSphere 支持包括 MySQL、Oracle、SQLServer、PostgreSQL 以及任何遵循 SQL92 标准的数据库。</p>
<h4 data-nodeid="83231">基础规范</h4>
<p data-nodeid="83232">作为一个开源框架，很多时候需要兼容业界已经形成标准的基础性规范。换句话说，想要框架被其他开发人员所认可，就得要考虑开发人员目前在使用的基础规范。例如，如果设计一个与链路跟踪相关的开源框架，一般都需要兼容 OpenTracing 这一开放式分布式追踪规范。</p>
<p data-nodeid="83233">对于 ShardingSphere 而言，所涉及的基础规范很明确，就是我们在上一课时中所详细阐述的 JDBC 规范。</p>
<h4 data-nodeid="83234">开发框架</h4>
<p data-nodeid="83235">开源框架本身也是一个开发框架，但我们通常不会自己设计和实现一个全新的开发框架，而是更倾向于与现有的主流开发框架进行集成。目前，Java 世界中最主流的开发框架就是 Spring 家族系列框架。</p>
<p data-nodeid="83236">ShardingSphere 同时集成了 Spring 和 Spring Boot 这两款 Spring 家族的主流开发框架。<strong data-nodeid="83340">熟悉这两款框架的开发人员在应用 ShardingSphere 进行开发时将不需要任何学习成本</strong>。</p>
<h4 data-nodeid="83237">领域框架</h4>
<p data-nodeid="83238">对于某些开源框架而言，也需要考虑和领域框架进行集成，以便提供更好的用户体验和使用友好性，区别于前面提到的适用于任何场景的开发框架。<strong data-nodeid="83347">所谓领域框架，是指与所设计的开源框架属于同一专业领域的开发框架。</strong> 业务开发人员已经习惯在日常开发过程中使用这些特定于某一领域的开发框架，所以在设计自己的开源框架时，也需要充分考虑与这些框架的整合和集成。</p>
<p data-nodeid="83239">对于 ShardingSphere 而言，领域框架指的是 MyBatis、Hibernate 等常见的 ORM 框架。ShardingSphere 对这领域框架提供了无缝集成的实现方案，熟悉 ORM 框架的开发人员在应用 ShardingSphere 进行开发时同样不需要任何学习成本。</p>
<p data-nodeid="83240">接下来，我们就结合前面抽象的开源框架应用方式来具体分析 ShardingSphere 框架为开发人员提供了哪些开发上的支持。</p>
<h3 data-nodeid="83241">数据库和 JDBC 集成</h3>
<p data-nodeid="83242">由于 ShardingSphere 最终操作的还是关系型数据库，并基于 JDBC 规范做了重写。所以<strong data-nodeid="83355">在具体应用上相对比较简单，我们只要把握 JDBC 驱动和数据库连接池的使用方式即可。</strong></p>
<h4 data-nodeid="83243">JDBC 驱动</h4>
<p data-nodeid="83244">ShardingSphere 支持 MySQL、Oracle 等实现 JDBC 规范的主流关系型数据库。我们在使用这些数据库时，常见的做法就是指定具体数据库对应的 JDBC 驱动类、URL 以及用户名和密码。</p>
<p data-nodeid="83245">这里以 MySQL 为例，展示了在 Spring Boot 应用程序中通过 .yaml 配置文件指定 JDBC 驱动的一般做法：</p>
<pre class="lang-java" data-nodeid="83246"><code data-language="java">driverClassName: com.mysql.jdbc.Driver
url: jdbc:mysql:<span class="hljs-comment">//localhost:3306/test_database</span>
username: root
password: root
</code></pre>
<h4 data-nodeid="83247">数据库连接池</h4>
<p data-nodeid="83248">配置 JDBC 驱动的目的是获取访问数据库所需的 Connection。为了提高性能，主流做法是采用数据库连接池方案，数据库连接池将创建的 Connection 对象存放到连接池中，然后从池中提供 Connection。</p>
<p data-nodeid="83249">ShardingSphere 支持一批主流的第三方数据库连接池，包括 DBCP、C3P0、BoneCP、Druid 和 HikariCP 等。在应用 ShardingSphere 时，我们可以通过创建 DataSource 来使用数据库连接池。例如，在 Spring Boot 中，可以在 .properties 配置文件中使用阿里巴巴提供的 DruidDataSource 类，初始化基于 Druid 数据库连接池的 DataSource：</p>
<pre class="lang-js" data-nodeid="83250"><code data-language="js">spring.shardingsphere.datasource.names= test_datasource
spring.shardingsphere.datasource.test_datasource.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.test_datasource.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span></span>=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.test_datasource.jdbc-url=jdbc:mysql:<span class="hljs-comment">//localhost:3306/test_database</span>
spring.shardingsphere.datasource.test_datasource.username=root
spring.shardingsphere.datasource.test_datasource.password=root
</code></pre>
<p data-nodeid="83251">而对于使用 Spring 框架的开发人员而言，可以直接在 Spring 容器中注入一个 DruidDataSource 的 JavaBean：</p>
<pre class="lang-js" data-nodeid="83252"><code data-language="js">&lt;bean id=<span class="hljs-string">"test_datasource"</span> <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"com.alibaba.druid.pool.DruidDataSource"</span> destroy-method=<span class="hljs-string">"close"</span>&gt;
&nbsp;&nbsp;&nbsp; <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"driverClassName"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"com.mysql.jdbc.Driver"</span>/&gt;</span></span>
&nbsp;&nbsp;&nbsp; <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"url"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"jdbc:mysql://localhost:3306/ test_database"</span>/&gt;</span></span>
&nbsp;&nbsp;&nbsp; <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"username"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"root"</span>/&gt;</span></span>
&nbsp;&nbsp;&nbsp; <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"password"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"root"</span>/&gt;</span></span>
&lt;/bean&gt;
</code></pre>
<h3 data-nodeid="83253">开发框架集成</h3>
<p data-nodeid="83254">从上面所介绍的配置信息中，你实际上已经看到了 ShardingSphere 中集成的两款主流开发框架，即 Spring 和 Spring Boot，它们都对 JDBC 规范做了封装。当然，对于没有使用或无法使用 Spring 家族框架的场景，我们也可以直接在原生 Java 应用程序中使用 ShardingSphere。</p>
<p data-nodeid="83255">在介绍开发框架的具体集成方式之前，我们来设计一个简单的应用场景。假设系统中存在一个用户表 User，这张表的数据量比较大，所以我们将它进行分库分表处理，计划分成两个数据库 ds0 和 ds1，然后每个库中再分成两张表 user0 和 user1：</p>
<p data-nodeid="83256"><img src="https://s0.lgstatic.com/i/image/M00/28/EB/Ciqc1F75qxSADY5yAADgZQ5r488284.png" alt="1.png" data-nodeid="83368"></p>
<p data-nodeid="83257">接下来，让我们来看一下如何基于 Java 原生、Spring 及 Spring Boot 开发框架针对这一场景实现分库分表。</p>
<h4 data-nodeid="83258">Java 原生</h4>
<p data-nodeid="83259">如果使用 Java 原生的开发方式，相当于我们需要全部通过 Java 代码来创建和管理 ShardingSphere 中与分库分表相关的所有类。如果不做特殊说明，本课程将默认使用 Maven 实现包依赖关系的管理。所以，首先需要引入对 sharding-jdbc-core 组件的 Maven 引用：</p>
<pre class="lang-xml" data-nodeid="83260"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.shardingsphere<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>sharding-jdbc-core<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="83261">然后，按照 JDBC 的使用方法，需要创建 DataSource、Connection、Statement 等一系列接口的实现类，并通过这些类来完成具体的数据库访问操作。</p>
<p data-nodeid="83262">我们先来看看创建 DataSource 的 Java 代码，这里构建了一个工具类 DataSourceHelper，基于 Druid 来获取一个 DruidDataSource：</p>
<pre class="lang-java" data-nodeid="83263"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataSourceHelper</span></span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String HOST = <span class="hljs-string">"localhost"</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> PORT = <span class="hljs-number">3306</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String USER_NAME = <span class="hljs-string">"root"</span>;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">final</span> String PASSWORD = <span class="hljs-string">"root"</span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> DataSource <span class="hljs-title">createDataSource</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String dataSourceName)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; DruidDataSource result = <span class="hljs-keyword">new</span> DruidDataSource();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setDriverClassName(com.mysql.jdbc.Driver.class.getName());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setUrl(String.format("jdbc:mysql://%s:%s/%s, HOST, PORT, dataSourceName));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setUsername(USER_NAME);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.setPassword(PASSWORD);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="83264">由于在示例中，我们需要创建两个用户库，所以使用一个 Map 来保存两个数据源对象：</p>
<pre class="lang-java" data-nodeid="83265"><code data-language="java">&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">static</span> Map&lt;String, DataSource&gt; <span class="hljs-title">createDataSourceMap</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, DataSource&gt; result = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(<span class="hljs-string">"ds0"</span>, DataSourceHelper.createDataSource(<span class="hljs-string">"ds0"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.put(<span class="hljs-string">"ds1"</span>, DataSourceHelper.createDataSource(<span class="hljs-string">"ds1"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
</code></pre>
<p data-nodeid="83266">有了包含初始化 DataSource 对象的数据源集合之后，接下来就可以通过设计分库分表规则来获取目标 DataSource：</p>
<pre class="lang-java" data-nodeid="83807"><code data-language="java">    <span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">dataSource</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException </span>{
         <span class="hljs-comment">//创建分片规则配置类</span>
        ShardingRuleConfiguration shardingRuleConfig = <span class="hljs-keyword">new</span> ShardingRuleConfiguration();
        
        <span class="hljs-comment">//创建分表规则配置类</span>
        TableRuleConfiguration tableRuleConfig = <span class="hljs-keyword">new</span> TableRuleConfiguration(<span class="hljs-string">"user"</span>, <span class="hljs-string">"ds${0..1}.user${0..1}"</span>);
        
        <span class="hljs-comment">//创建分布式主键生成配置类</span>
        Properties properties = <span class="hljs-keyword">new</span> Properties();
        properties.setProperty(<span class="hljs-string">"worker.id"</span>, <span class="hljs-string">"33"</span>);
        KeyGeneratorConfiguration keyGeneratorConfig = <span class="hljs-keyword">new</span> KeyGeneratorConfiguration(<span class="hljs-string">"SNOWFLAKE"</span>, <span class="hljs-string">"id"</span>, properties);              
        tableRuleConfig.setKeyGeneratorConfig(keyGeneratorConfig);      
        shardingRuleConfig.getTableRuleConfigs().add(tableRuleConfig);
        
        <span class="hljs-comment">//根据性别分库，一共分为 2 个库</span>
        shardingRuleConfig.setDefaultDatabaseShardingStrategyConfig(<span class="hljs-keyword">new</span> InlineShardingStrategyConfiguration(<span class="hljs-string">"sex"</span>, <span class="hljs-string">"ds${sex % 2}"</span>));
        
        <span class="hljs-comment">//根据用户 ID 分表，一共分为 2 张表</span>
        shardingRuleConfig.setDefaultTableShardingStrategyConfig(<span class="hljs-keyword">new</span> StandardShardingStrategyConfiguration(<span class="hljs-string">"id"</span>, <span class="hljs-string">"user${id % 2}"</span>));
        
        <span class="hljs-comment">//通过工厂类创建具体的 DataSource</span>
        <span class="hljs-keyword">return</span> ShardingDataSourceFactory.createDataSource(createDataSourceMap(), shardingRuleConfig, <span class="hljs-keyword">new</span> Properties());
    }
</code></pre>

<p data-nodeid="83417">这里使用到了大量 ShardingSphere 中的规则配置类，包含分片规则配置、分表规则配置、分布式主键生成配置等。同时，我们在分片规则配置中使用行表达式来设置具体的分片规则。关于行表达式的具体使用方法将在下一课时中进行介绍，这里只简单地根据用户的年龄和 ID 分别来进行分库和分表。同时，我们在方法的最后部分传入前面已经初始化的 DataSource 集合并通过工厂类来创建具体的某一个目标 DataSource。</p>
<p data-nodeid="83418">一旦获取了目标 DataSource 之后，我们就可以使用 JDBC 中的核心接口来执行传入的 SQL 语句：</p>

<pre class="lang-java" data-nodeid="83269"><code data-language="java">&nbsp;&nbsp;&nbsp; <span class="hljs-function">List&lt;User&gt; <span class="hljs-title">getUsers</span><span class="hljs-params">(<span class="hljs-keyword">final</span> String sql)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;User&gt; result = <span class="hljs-keyword">new</span> LinkedList&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> (Connection connection = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PreparedStatement preparedStatement = connection.prepareStatement(sql);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet resultSet = preparedStatement.executeQuery()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (resultSet.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; User user= <span class="hljs-keyword">new</span> User();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略设置User对象的赋值语句</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; result.add(user);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
</code></pre>
<p data-nodeid="83270">可以看到，这里用到了熟悉的 Connection、PreparedStatement 和 ResultSet 等 JDBC 接口来执行查询并获取结果，整个过程就像是在使用普通的 JDBC 一样。但这个时候，这些 JDBC 接口背后的实现类都已经嵌入了分片功能。</p>
<h4 data-nodeid="83271">Spring</h4>
<p data-nodeid="83272">如果使用 Spring 作为我们的开发框架，那么 JDBC 中各个核心对象的创建过程都会交给 Spring 容器进行完成。<strong data-nodeid="83386">ShardingSphere 中基于命名空间（NameSpace）机制完成了与 Spring 框架的无缝集成。要想使用这种机制，需要先引入对应的 Maven 依赖</strong>：</p>
<pre class="lang-xml" data-nodeid="83273"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp; &nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.apache.shardingsphere<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>sharding-jdbc-spring-namespace<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="83274">Spring 中的命名空间机制本质上就是基于 Spring 配置文件的 XML Scheme 添加定制化的配置项并进行解析，所以我们会在 XML 配置文件中看到一系列与分片相关的自定义配置项。例如，DataSource 的初始化过程相当于创建一个 Java Bean 的过程：</p>
<pre class="lang-xml" data-nodeid="83275"><code data-language="xml">&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">bean</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"ds0"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"com.alibaba.druid.pool.DruidDataSource"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"driverClassName"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"com.mysql.jdbc.Driver"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"url"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"jdbc:mysql://localhost:3306/ds0"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"username"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"root"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">property</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"password"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"root"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">bean</span>&gt;</span>
</code></pre>
<p data-nodeid="83276">接下来，我们同样可以通过一系列的配置项来初始化相应的分库规则，并最终完成目标 DataSource 的创建过程：</p>
<pre class="lang-xml" data-nodeid="83277"><code data-language="xml">&nbsp;&nbsp;&nbsp; <span class="hljs-comment">&lt;!-- 创建分库配置 --&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:inline-strategy</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"databaseStrategy"</span> <span class="hljs-attr">sharding-column</span>=<span class="hljs-string">"sex"</span> <span class="hljs-attr">algorithm-expression</span>=<span class="hljs-string">"ds${sex % 2}"</span> /&gt;</span>

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">&lt;!-- 创建分表配置 --&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:inline-strategy</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"tableStrategy"</span> <span class="hljs-attr">sharding-column</span>=<span class="hljs-string">"id"</span> <span class="hljs-attr">algorithm-expression</span>=<span class="hljs-string">"user${id % 2}"</span> /&gt;</span>

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">&lt;!-- 创建分布式主键生成配置 --&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">bean:properties</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"properties"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">prop</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"worker.id"</span>&gt;</span>33<span class="hljs-tag">&lt;/<span class="hljs-name">prop</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">bean:properties</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:key-generator</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"keyGenerator"</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"SNOWFLAKE"</span> <span class="hljs-attr">column</span>=<span class="hljs-string">"id"</span> <span class="hljs-attr">props-ref</span>=<span class="hljs-string">"properties"</span> /&gt;</span>

&nbsp;&nbsp;&nbsp; <span class="hljs-comment">&lt;!-- 创建分片规则配置 --&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:data-source</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"shardingDataSource"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:sharding-rule</span> <span class="hljs-attr">data-source-names</span>=<span class="hljs-string">"ds0, ds1"</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:table-rules</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">sharding:table-rule</span> <span class="hljs-attr">logic-table</span>=<span class="hljs-string">"user"</span> <span class="hljs-attr">actual-data-nodes</span>=<span class="hljs-string">"ds${0..1}.user${0..1}"</span> <span class="hljs-attr">database-strategy-ref</span>=<span class="hljs-string">"databaseStrategy"</span> <span class="hljs-attr">table-strategy-ref</span>=<span class="hljs-string">"tableStrategy"</span> <span class="hljs-attr">key-generator-ref</span>=<span class="hljs-string">"keyGenerator"</span> /&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">sharding:table-rules</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">sharding:sharding-rule</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">sharding:data-source</span>&gt;</span>
</code></pre>
<p data-nodeid="83278">关于这些配置项的内容我们同样放在下一课时中进行详细讨论。</p>
<h4 data-nodeid="83279">Spring Boot</h4>
<p data-nodeid="83280">如果你使用的开发框架是 Spring Boot，那么所需要做的也是编写一些配置项。在 Spring Boot 中，配置项的组织形式有两种，一种是 .yaml 文件，一种是 .properties 文件，这里以 .properties 为例给出 DataSource 的配置：</p>
<pre class="lang-js" data-nodeid="83281"><code data-language="js">spring.shardingsphere.datasource.names=ds0,ds1
&nbsp;
spring.shardingsphere.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds0.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span></span>=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds0.jdbc-url=jdbc:mysql:<span class="hljs-comment">//localhost:3306/ds0</span>
spring.shardingsphere.datasource.ds0.username=root
spring.shardingsphere.datasource.ds0.password=root
&nbsp;
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds1.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span></span>=com.mysql.jdbc.Driver
spring.shardingsphere.datasource.ds1.jdbc-url=jdbc:mysql:<span class="hljs-comment">//localhost:3306/ds1</span>
spring.shardingsphere.datasource.ds1.username=root
spring.shardingsphere.datasource.ds1.password=root
</code></pre>
<p data-nodeid="83282">有了 DataSource 之后，我们同样可以设置对应的分库策略、分表策略及分布式主键生成策略：</p>
<pre class="lang-js" data-nodeid="83283"><code data-language="js">spring.shardingsphere.sharding.default-database-strategy.inline.sharding-column=sex
spring.shardingsphere.sharding.default-database-strategy.inline.algorithm-expression=ds$-&gt;{sex % <span class="hljs-number">2</span>}
&nbsp;
spring.shardingsphere.sharding.tables.user.actual-data-nodes=ds$-&gt;{<span class="hljs-number">0.</span><span class="hljs-number">.1</span>}.user$-&gt;{<span class="hljs-number">0.</span><span class="hljs-number">.1</span>}
spring.shardingsphere.sharding.tables.user.table-strategy.inline.sharding-column=id
spring.shardingsphere.sharding.tables.user.table-strategy.inline.algorithm-expression=user$-&gt;{id % <span class="hljs-number">2</span>}
spring.shardingsphere.sharding.tables.user.key-generator.column=id
spring.shardingsphere.sharding.tables.user.key-generator.type=SNOWFLAKE
spring.shardingsphere.sharding.tables.user.key-generator.props.worker.id=<span class="hljs-number">33</span>
</code></pre>
<p data-nodeid="83284">可以看到，相比 Spring 提供的命名空间机制，基于 Spring Boot 的配置风格相对简洁明了，容易理解。</p>
<p data-nodeid="83285">一旦我们提供了这些配置项，就可以直接在应用程序中注入一个 DataSource 来获取 Connection 等 JDBC 对象。但在日常开发过程中，如果我们使用了 Spring 和 Spring Boot 开发框架，一般都不会直接使用原生的 JDBC 接口来操作数据库，而是通过集成常见的 ORM 框架来实现这一点。让我们来看一下。</p>
<h3 data-nodeid="83286">ORM 框架集成</h3>
<p data-nodeid="83287">在 Java 领域，主流的 ORM 框架可以分成两大类，一类遵循 JPA（Java Persistence API，Java 持久层 API）规范，代表性的框架有 Hibernate、TopLink 等；而另一类则完全采用自定义的方式来实现对象和关系之间的映射，代表性的框架就是 MyBatis。</p>
<p data-nodeid="83288">这里以 Spring Boot 开发框架为例，简要介绍这两种 ORM 框架的集成方式。基于 Spring Boot 提供的强大自动配置机制，我们发现集成这些 ORM 框架的方式非常简单。</p>
<h4 data-nodeid="83289">JPA</h4>
<p data-nodeid="83290">想要在 Spring Boot 中使用 JPA，我们需要在 pom 文件中添加对 spring-boot-starter-data-jpa 的 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="83291"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-data-jpa<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="83292">一旦添加了 Maven 依赖，Spring Boot 就会自动导入 spring-orm、hibernate-entity-manager、spring-data-jpa 等一系列工具包。然后，在 application.properties 配置文件中添加与 JPA 相关的配置项就可以了：</p>
<pre class="lang-sql" data-nodeid="83293"><code data-language="sql">spring.jpa.properties.hibernate.hbm2ddl.auto=<span class="hljs-keyword">create</span>-<span class="hljs-keyword">drop</span>
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
spring.jpa.properties.hibernate.show_sql=<span class="hljs-literal">false</span>
</code></pre>
<p data-nodeid="83294">当然，我们需要在业务代码中完成 JPA 的 Entity 实体类、Repository 仓库类的定义，并在 Spring Boot 的启动类中完成对包含对应包结构的扫描：</p>
<pre class="lang-java" data-nodeid="83295"><code data-language="java"><span class="hljs-meta">@ComponentScan("com.user.jpa")</span>
<span class="hljs-meta">@EntityScan(basePackages = "com.user.jpa.entity")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserApplication</span>
</span></code></pre>
<h4 data-nodeid="83296">MyBatis</h4>
<p data-nodeid="83297">对于 MyBatis 而言，集成的步骤也类似。首先，我们需要添加 Maven 依赖：</p>
<pre class="lang-xml" data-nodeid="83298"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.mybatis.spring.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>mybatis-spring-boot-starter<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="83299">然后，由于 MyBatis 的启动依赖于框架提供的专用配置项，一般我们会把这些配置项组织在一个独立的配置文件中，并在 Spring Boot 的 application.properties 中引用这个配置文件：</p>
<pre class="lang-java" data-nodeid="83300"><code data-language="java">mybatis.config-location=classpath:META-INF/mybatis-config.xml
</code></pre>
<p data-nodeid="83301">在 mybatis-config.xml 配置文件中，至少会包含各种 Mybatis Mapper 文件的定义：</p>
<pre class="lang-xml" data-nodeid="83302"><code data-language="xml"><span class="hljs-meta">&lt;?xml version="1.0" encoding="UTF-8" ?&gt;</span>
<span class="hljs-meta">&lt;!DOCTYPE <span class="hljs-meta-keyword">configuration</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta-keyword">PUBLIC</span> <span class="hljs-meta-string">"-//mybatis.org//DTD Config 3.0//EN"</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta-string">"http://mybatis.org/dtd/mybatis-3-config.dtd"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">configuration</span>&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">mappers</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;<span class="hljs-name">mapper</span> <span class="hljs-attr">resource</span>=<span class="hljs-string">"mappers/UserMapper.xml"</span>/&gt;</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-tag">&lt;/<span class="hljs-name">mappers</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">configuration</span>&gt;</span>
</code></pre>
<p data-nodeid="83303">而在 Mapper 文件中，就包含了运行 MyBatis 所需的实体与数据库模式之间的映射关系，以及各种数据库操作的 SQL 语句定义。</p>
<p data-nodeid="83304">最后，我们同样需要在 Spring Boot 的启动类中添加对包含各种 Entity 和 Repository 定义的包结构的扫描机制：</p>
<pre class="lang-java" data-nodeid="83305"><code data-language="java"><span class="hljs-meta">@ComponentScan("com.user.mybatis")</span>
<span class="hljs-meta">@MapperScan(basePackages = "com.user.mybatis.repository")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserApplication</span>
</span></code></pre>
<p data-nodeid="83306">这样，我们在业务系统中使用 ShardingSphere 的各种方式就介绍完毕了。</p>
<h3 data-nodeid="83307">小结</h3>
<p data-nodeid="83308">作为一个优秀的开源框架，ShardingSphere 提供了多方面的集成方式供广大开发人员在业务系统中使用它来完成分库分表操作。在这一课时中，我们先梳理了作为一个开源框架所应该具备的应用方式，并分析了这些应用方式在 ShardingSphere 中的具体实现机制。可以看到，<strong data-nodeid="83414">从 JDBC 规范，到 Spring、Spring Boot 开发框架，再到 JPA、MyBatis 等主流 ORM 框架，ShardingSphere 都提供了完善的集成方案。</strong></p>
<p data-nodeid="83309">这里给你留一道思考题：为了实现框架的易用性，ShardingSphere 为开发人员提供了哪些工具和规范的集成？</p>
<p data-nodeid="83310" class="">另一方面，在今天的课程中，我们也看到，使用 ShardingSphere 的主要方式事实上就是基于它所提供的配置体系，来完成各种配置项的创建和设置。可以说，配置工作是使用 ShardingSphere 进行开发的主要工作。</p>

---

### 精选评论

##### **0306：
> 不错，有了个概念，大神都是实践出来的

##### **国：
> 是不是能力沒跟上？ 看起來有點吃力

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 加油

##### *征：
> 老师能把文章中的使用到的jar版本都在前面说下吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 框架版本在我的示例代码里面都有的，可以看一下

##### *炎：
> 老师讲得很有条理，比看官方文档容易理解多了。赞赞赞赞赞赞赞赞赞！

