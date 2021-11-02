<p data-nodeid="32342" class="">最近几年 DataSource 越来越成熟，但当我们做开发的时候对 DataSource 的关心却越来越少，这是因为大多数情况都是利用 application.properties进行简单的数据源配置，项目就可以正常运行了。但是当我们想要解决一些原理性问题的时候，就需要用到 DataSource、连接池等基础知识了。</p>


<p data-nodeid="31018">那么这一讲我将带你揭开 DataSource 的面纱，一起来了解它是什么、如何使用，以及最佳实践是什么呢？</p>
<h3 data-nodeid="31019">数据源是什么？</h3>
<p data-nodeid="34096">当我们用第三方工具去连接数据库（Mysql，Oracle 等）的时候，一般都会让我们选择数据源，如下图所示：</p>
<p data-nodeid="34097" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2D/CgqCHl-o5UKAeKojAAMZoym4vVw887.png" alt="Drawing 0.png" data-nodeid="34101"></p>


<p data-nodeid="35854">我们以 MySQL 为例，当选择 MySQL 的时候就会弹出如下图显示的界面：</p>
<p data-nodeid="35855" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/22/Ciqc1F-o5UuACyC4AAEVdHsmM98446.png" alt="Drawing 1.png" data-nodeid="35859"></p>


<p data-nodeid="31024">其中，我们在选择了 Driver（驱动）和 Host、UserName、Password 等之后，就可以创建一个 Connection，然后连接到数据库里面了。</p>
<p data-nodeid="31025">同样的道理，在 Java 里面我们也需要用到 DataSource 去连接数据库，而 Java 定义了一套 JDBC 的协议标准，其中有一个 javax.sql.DataSource 接口类，通过实现此类就可以进行数据库连接，我们通过源码来分析一下。</p>
<h4 data-nodeid="31026">DataSource 源码分析</h4>
<p data-nodeid="31027">DataSource 接口里面主要的代码如下所示：</p>
<pre class="lang-java" data-nodeid="39383"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">DataSource</span>  <span class="hljs-keyword">extends</span> <span class="hljs-title">CommonDataSource</span>, <span class="hljs-title">Wrapper</span> </span>{
<span class="hljs-function">Connection <span class="hljs-title">getConnection</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException</span>;
<span class="hljs-function">Connection <span class="hljs-title">getConnection</span><span class="hljs-params">(String username, String password)</span>
  <span class="hljs-keyword">throws</span> SQLException</span>;
}
</code></pre>




<p data-nodeid="31029">我们通过源码可以很清楚地看到，DataSource 的主要目的就是获得数据库连接，就像我们前面用工具连接数据库一样，只不过工具是通过界面实现的，而 DataSource 是通过代码实现的。</p>
<p data-nodeid="41136">那么在程序里面如何实现呢？也有很多第三方的实现方式，常见的有C3P0、BBCP、Proxool、Druid、Hikari，而目前 Spring Boot 里面是采用 Hikari 作为默认数据源。Hikari 的优点是：开源，社区活跃，性能高，监控完整。我们通过工具看一下项目里面DataSource 的实现类有哪些，如下图所示：</p>
<p data-nodeid="41137" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2D/CgqCHl-o5VuAHBXrAAF5pvcL8BA769.png" alt="Drawing 2.png" data-nodeid="41141"></p>


<p data-nodeid="31032">其中，当我采用默认数据源的时候，可以看到数据源的实现类有：h2 里面的 JdbcDataSource、MySQL 连接里面的 MysqlDataSource，以及今天要重点介绍的 HikariDataSource（默认数据源，也是 Spring 社区推荐的最佳数据源）。</p>
<p data-nodeid="31033">我们直接打开 HikariDataSource 的源码看一下，它的关键代码如下：</p>
<pre class="lang-java" data-nodeid="44665"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">HikariDataSource</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">HikariConfig</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">DataSource</span>, <span class="hljs-title">Closeable</span></span>{
  <span class="hljs-keyword">private</span> <span class="hljs-keyword">volatile</span> HikariPool pool;
  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">HikariDataSource</span><span class="hljs-params">(HikariConfig configuration)</span>
  </span>{
     configuration.validate();
     configuration.copyStateTo(<span class="hljs-keyword">this</span>);
  
     LOGGER.info(<span class="hljs-string">"{} - Starting..."</span>, configuration.getPoolName());
     pool = fastPathPool = <span class="hljs-keyword">new</span> HikariPool(<span class="hljs-keyword">this</span>);
     LOGGER.info(<span class="hljs-string">"{} - Start completed."</span>, configuration.getPoolName());
  
     <span class="hljs-keyword">this</span>.seal();
  }
  <span class="hljs-comment">//这个是最主要的实现逻辑，即通过连接池获得连接的逻辑</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> Connection <span class="hljs-title">getConnection</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> SQLException</span>{
     <span class="hljs-keyword">if</span> (isClosed()) {
        <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> SQLException(<span class="hljs-string">"HikariDataSource "</span> + <span class="hljs-keyword">this</span> + <span class="hljs-string">" has been closed."</span>);
     }
  
     <span class="hljs-keyword">if</span> (fastPathPool != <span class="hljs-keyword">null</span>) {
        <span class="hljs-keyword">return</span> fastPathPool.getConnection();
     }
  
     <span class="hljs-comment">// See http://en.wikipedia.org/wiki/Double-checked_locking#Usage_in_Java</span>
     HikariPool result = pool;
     <span class="hljs-keyword">if</span> (result == <span class="hljs-keyword">null</span>) {
        <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
           result = pool;
           <span class="hljs-keyword">if</span> (result == <span class="hljs-keyword">null</span>) {
              validate();
              LOGGER.info(<span class="hljs-string">"{} - Starting..."</span>, getPoolName());
              <span class="hljs-keyword">try</span> {
                 pool = result = <span class="hljs-keyword">new</span> HikariPool(<span class="hljs-keyword">this</span>);
                 <span class="hljs-keyword">this</span>.seal();
              }
              <span class="hljs-keyword">catch</span> (PoolInitializationException pie) {
                 <span class="hljs-keyword">if</span> (pie.getCause() <span class="hljs-keyword">instanceof</span> SQLException) {
                    <span class="hljs-keyword">throw</span> (SQLException) pie.getCause();
                 }
                 <span class="hljs-keyword">else</span> {
                    <span class="hljs-keyword">throw</span> pie;
                 }
              }
              LOGGER.info(<span class="hljs-string">"{} - Start completed."</span>, getPoolName());
           }
        }
     }
     <span class="hljs-keyword">return</span> result.getConnection();
  }
......
}
</code></pre>




<p data-nodeid="31035">从上面的源码可以看到关键的两点问题：</p>
<ol data-nodeid="31036">
<li data-nodeid="31037">
<p data-nodeid="31038">数据源的关键配置属性有哪些？</p>
</li>
<li data-nodeid="31039">
<p data-nodeid="31040">连接怎么获得？连接池的作用如何？</p>
</li>
</ol>
<p data-nodeid="31041">下面我们分别详解一下。</p>
<p data-nodeid="46418">第一个问题，HikariConfig 的配置里面描述了 Hikari 数据源主要的配置属性，我们打开来看一下，如图所示：</p>
<p data-nodeid="46419" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2D/CgqCHl-o5W6AYziKAALhn9RPD2o391.png" alt="Drawing 3.png" data-nodeid="46423"></p>


<p data-nodeid="31044">通过上面的源码我们可以看到数据源的关键配置信息：用户名、密码、连接池的配置、jdbcUrl、驱动的名字，等等，这些字段你可以参考课程开始时我介绍的工具，细心观察的话都可以找到对应关系，也就是创建数据源需要的一些配置项。</p>
<p data-nodeid="31045">上面提到的第 2 个问题，我们通过 getConnection 方法里面的代码可以看到 HikariPool 的用法，也就是说，我们是通过连接池来获得连接的，这个连接用过之后没有断开，而是重新放回到连接池里面（<strong data-nodeid="31250">这个地方你一定要谨记，它也说明了 connection 是可以共享的</strong>）。</p>
<p data-nodeid="31046">而连接池的用途你应该也知道，创建连接是非常昂贵的，所以需要用到连接池技术、共享现有的连接，以增加代码的执行效率。</p>
<p data-nodeid="31047">那么这个时候有一个问题是需要我们搞清楚并且牢记的，就是数据源和 driver（驱动）、数据库连接、连接池是什么关系？</p>
<h4 data-nodeid="31048">数据源与驱动与连接和连接池的关系</h4>
<p data-nodeid="31049">我分为下述四点来说，方便你理解。</p>
<ol data-nodeid="31050">
<li data-nodeid="31051">
<p data-nodeid="31052">数据源的作用是给应用程序提供不同 DB 的连接 connection；</p>
</li>
<li data-nodeid="31053">
<p data-nodeid="31054">连接是通过连接池获取的，这主要是出于连接性能的考虑；</p>
</li>
<li data-nodeid="31055">
<p data-nodeid="31056">创建好连接之后，通过数据库的驱动来进行数据库操作；</p>
</li>
<li data-nodeid="31057">
<p data-nodeid="31058">而不同的 DB（MySQL / h2 / oracle），都有自己的驱动类和相应的驱动 Jar 包。</p>
</li>
</ol>
<p data-nodeid="48168">我们用一个图来表示一下：</p>
<p data-nodeid="48169" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/22/Ciqc1F-o5XmADAK4AAC16V3GEqM205.png" alt="Drawing 4.png" data-nodeid="48173"></p>



<p data-nodeid="31062">而我们常说的 MySQL 驱动，其实就是 com.mysql.cj.jdbc.Driver，而这个类主要存在于 mysql-connection-java:8.0* 的 jar 里面，也就是我们经常说的不同的数据库所代表的驱动 jar 包。</p>
<p data-nodeid="31063">这里我们用的是 spring boot 2.3.3 版本引用的 mysql-connection-java 8.0 版本驱动 jar 包，不同的数据库引用的 jar 包是不一样的。例如，H2 数据源中，我们用的驱动类是 org.h2.Driver，其包含在 com.h2database:h2:1.4.*jar 包里面。</p>
<p data-nodeid="31064">接下来我们通过源码分析 Spring 里面的加载原理，来看下 Hikari 都有哪些配置项。</p>
<h3 data-nodeid="31065">数据源的加载原理和过程是什么样的？</h3>
<p data-nodeid="31066">我们通过 spring.factories 文件可以看到 JDBC 数据源相关的自动加载的类 DataSourceAutoConfiguration，那么我们就从这个类开始分析。</p>
<h4 data-nodeid="31067">DataSourceAutoConfiguration 数据源的加载过程分析</h4>
<p data-nodeid="31068">DataSourceAutoConfiguration 的关键源码如下所示：</p>
<pre class="lang-java" data-nodeid="49046"><code data-language="java"><span class="hljs-comment">//将spring.datasource.**的配置放到DataSourceProperties对象里面；</span>
<span class="hljs-meta">@EnableConfigurationProperties(DataSourceProperties.class)</span>
<span class="hljs-meta">@Import({ DataSourcePoolMetadataProvidersConfiguration.class, DataSourceInitializationConfiguration.class })</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataSourceAutoConfiguration</span> </span>{
   <span class="hljs-comment">//默认集成的数据源，一般指的是H2，方便我们快速启动和上手，一般不在生产环境应用；</span>
   <span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
   <span class="hljs-meta">@Conditional(EmbeddedDatabaseCondition.class)</span>
   <span class="hljs-meta">@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })</span>
   <span class="hljs-meta">@Import(EmbeddedDataSourceConfiguration.class)</span>
   <span class="hljs-keyword">protected</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EmbeddedDatabaseConfiguration</span> </span>{
   }
   <span class="hljs-comment">//加载不同的数据源的配置</span>
   <span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
   <span class="hljs-meta">@Conditional(PooledDataSourceCondition.class)</span>
   <span class="hljs-meta">@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })</span>
   <span class="hljs-meta">@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
         DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
         DataSourceJmxConfiguration.class })</span>
   <span class="hljs-keyword">protected</span> <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PooledDataSourceConfiguration</span> </span>{
   }
   ....
}
</code></pre>

<p data-nodeid="31070">从源码中我们可以得到以下三点最关键的信息：<br>
第一，通过 @EnableConfigurationProperties(DataSourceProperties.class) 可以看得出来 spring.datasource 的配置项有哪些，那么我们打开 DataSourceProperties 的源码看一下，关键代码如下：</p>
<pre class="lang-java" data-nodeid="49919"><code data-language="java"><span class="hljs-meta">@ConfigurationProperties(prefix = "spring.datasource")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataSourceProperties</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">BeanClassLoaderAware</span>, <span class="hljs-title">InitializingBean</span> </span>{
   <span class="hljs-keyword">private</span> ClassLoader classLoader;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> generateUniqueName = <span class="hljs-keyword">true</span>;
   <span class="hljs-keyword">private</span> Class&lt;? extends DataSource&gt; type;
   <span class="hljs-keyword">private</span> String driverClassName;
   <span class="hljs-keyword">private</span> String url;
   <span class="hljs-keyword">private</span> String username;
   <span class="hljs-keyword">private</span> String password;
   <span class="hljs-comment">//计算确定drivername的值是什么</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">determineDriverClassName</span><span class="hljs-params">()</span> </span>{
   <span class="hljs-keyword">if</span> (StringUtils.hasText(<span class="hljs-keyword">this</span>.driverClassName)) {
      Assert.state(driverClassIsLoadable(), () -&gt; <span class="hljs-string">"Cannot load driver class: "</span> + <span class="hljs-keyword">this</span>.driverClassName);
      <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.driverClassName;
   }
   String driverClassName = <span class="hljs-keyword">null</span>;
   <span class="hljs-comment">//此段逻辑是，当我们没有配置自己的drivername的时候，它会根据我们配置的DB的url自动计算出来drivername的值是什么，所以就会发现我们现在很多datasource里面的配置都省去了driver-name的配置，这是Spring Boot的功劳。</span>
   <span class="hljs-keyword">if</span> (StringUtils.hasText(<span class="hljs-keyword">this</span>.url)) {
      driverClassName = DatabaseDriver.fromJdbcUrl(<span class="hljs-keyword">this</span>.url).getDriverClassName();
   }
   <span class="hljs-keyword">if</span> (!StringUtils.hasText(driverClassName)) {
      driverClassName = <span class="hljs-keyword">this</span>.embeddedDatabaseConnection.getDriverClassName();
   }
   <span class="hljs-keyword">if</span> (!StringUtils.hasText(driverClassName)) {
      <span class="hljs-keyword">throw</span> <span class="hljs-keyword">new</span> DataSourceBeanCreationException(<span class="hljs-string">"Failed to determine a suitable driver class"</span>, <span class="hljs-keyword">this</span>,
            <span class="hljs-keyword">this</span>.embeddedDatabaseConnection);
   }
   <span class="hljs-keyword">return</span> driverClassName;
}
</code></pre>

<p data-nodeid="51656">我们通过 DatabaseDriver 的源码可以看到 MySQL 的默认驱动 Spring Boot 是采用 com.mysql.cj.jdbc.Driver 来实现的。</p>
<p data-nodeid="51657" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/22/Ciqc1F-o5Y6AJJGgAADB_oP_Er8606.png" alt="Drawing 6.png" data-nodeid="51661"></p>


<p data-nodeid="53398">同时，@ConfigurationProperties(prefix = "spring.datasource") 也告诉我们，application.properties 里面的 datasource 相关的公共配置可以以 spring.datasource 为开头，这样当启动的时候，DataSourceProperties 就会将 datasource 的一切配置自动加载进来。正如我们前面在 application.properties 里面的配置的一样，如下图所示：</p>
<p data-nodeid="53399" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2E/CgqCHl-o5ZWAC7oxAAIk-0v0OKA580.png" alt="Drawing 7.png" data-nodeid="53407"></p>


<p data-nodeid="31076">这里有 url、username、password、driver-class-name 等关键配置，不同数据源的公共配置也不多。</p>
<p data-nodeid="31077">第二，我们通过下面这一段代码也可以看得出来不同的数据源的配置是什么样的。</p>
<pre class="lang-java" data-nodeid="54280"><code data-language="java"><span class="hljs-meta">@Import({ DataSourceConfiguration.Hikari.class, DataSourceConfiguration.Tomcat.class,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DataSourceConfiguration.Dbcp2.class, DataSourceConfiguration.Generic.class,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;DataSourceJmxConfiguration.class })</span>
</code></pre>

<p data-nodeid="31079">为了再进一步了解，我们打开 DataSourceConfiguration 的源码，如下所示：</p>
<pre class="lang-java" data-nodeid="55153"><code data-language="java"><span class="hljs-keyword">abstract</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataSourceConfiguration</span> </span>{
   <span class="hljs-meta">@SuppressWarnings("unchecked")</span>
   <span class="hljs-keyword">protected</span> <span class="hljs-keyword">static</span> &lt;T&gt; <span class="hljs-function">T <span class="hljs-title">createDataSource</span><span class="hljs-params">(DataSourceProperties properties, Class&lt;? extends DataSource&gt; type)</span> </span>{
      <span class="hljs-keyword">return</span> (T) properties.initializeDataSourceBuilder().type(type).build();
   }
   <span class="hljs-comment">/**
    * Tomcat连接池数据源的配置，前提条件需要引入tomcat-jdbc*.jar
    */</span>
   <span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
   <span class="hljs-meta">@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnMissingBean(DataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource",
         matchIfMissing = true)</span>
   <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Tomcat</span> </span>{
      <span class="hljs-meta">@Bean</span>
      <span class="hljs-meta">@ConfigurationProperties(prefix = "spring.datasource.tomcat")</span>
      org.apache.tomcat.jdbc.pool.<span class="hljs-function">DataSource <span class="hljs-title">dataSource</span><span class="hljs-params">(DataSourceProperties properties)</span> </span>{
         org.apache.tomcat.jdbc.pool.DataSource dataSource = createDataSource(properties,
               org.apache.tomcat.jdbc.pool.DataSource.class);
         DatabaseDriver databaseDriver = DatabaseDriver.fromJdbcUrl(properties.determineUrl());
         String validationQuery = databaseDriver.getValidationQuery();
         <span class="hljs-keyword">if</span> (validationQuery != <span class="hljs-keyword">null</span>) {
            dataSource.setTestOnBorrow(<span class="hljs-keyword">true</span>);
            dataSource.setValidationQuery(validationQuery);
         }
         <span class="hljs-keyword">return</span> dataSource;
      }
   }
   <span class="hljs-comment">/**
    * Hikari数据源的配置，默认Spring Boot加载的是Hikari数据源
    */</span>
   <span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
   <span class="hljs-meta">@ConditionalOnClass(HikariDataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnMissingBean(DataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "com.zaxxer.hikari.HikariDataSource",
         matchIfMissing = true)</span>
   <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Hikari</span> </span>{
      <span class="hljs-meta">@Bean</span>
      <span class="hljs-meta">@ConfigurationProperties(prefix = "spring.datasource.hikari")</span>
      <span class="hljs-function">HikariDataSource <span class="hljs-title">dataSource</span><span class="hljs-params">(DataSourceProperties properties)</span> </span>{
         HikariDataSource dataSource = createDataSource(properties, HikariDataSource.class);
         <span class="hljs-keyword">if</span> (StringUtils.hasText(properties.getName())) {
            dataSource.setPoolName(properties.getName());
         }
         <span class="hljs-keyword">return</span> dataSource;
      }
   }
   <span class="hljs-comment">/**
    * DBCP数据源的配置，按照Spring Boot的语法，我们必须引入CommonsDbcp**.jar依赖才有用
    */</span>
   <span class="hljs-meta">@Configuration(proxyBeanMethods = false)</span>
   <span class="hljs-meta">@ConditionalOnClass(org.apache.commons.dbcp2.BasicDataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnMissingBean(DataSource.class)</span>
   <span class="hljs-meta">@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.commons.dbcp2.BasicDataSource",
         matchIfMissing = true)</span>
   <span class="hljs-keyword">static</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Dbcp2</span> </span>{
      <span class="hljs-meta">@Bean</span>
      <span class="hljs-meta">@ConfigurationProperties(prefix = "spring.datasource.dbcp2")</span>
      org.apache.commons.dbcp2.<span class="hljs-function">BasicDataSource <span class="hljs-title">dataSource</span><span class="hljs-params">(DataSourceProperties properties)</span> </span>{
         <span class="hljs-keyword">return</span> createDataSource(properties, org.apache.commons.dbcp2.BasicDataSource.class);
      }
   }
</code></pre>

<p data-nodeid="31081">我们通过上述源码可以看到最常见的三种数据源的配置：</p>
<ul data-nodeid="31082">
<li data-nodeid="31083">
<p data-nodeid="31084">HikariDataSource</p>
</li>
<li data-nodeid="31085">
<p data-nodeid="31086">tomcat的JDBC</p>
</li>
<li data-nodeid="31087">
<p data-nodeid="31088">apache的dbcp</p>
</li>
</ul>
<p data-nodeid="31089">而最终用哪个，就看你引用了哪个 datasoure 的 jar 包。不过 Spring Boot 2.0 之后就推荐使用 Hikari 数据源了，你了解一下就好。</p>
<p data-nodeid="31090">第三，我们通过 @ConfigurationProperties(prefix = "spring.datasource.hikari")      HikariDataSource dataSource(DataSourceProperties properties) 可以知道，application.properties 里面 spring.datasource.hikari 开头的配置会被映射到 HikariDataSource 对象中，而开篇我们就提到了，是 HikariDataSource 继承了 HikariConfig。</p>
<p data-nodeid="56890">所以顺理成章地，我们就可以知道 Hikari 数据源的配置有哪些了，如下图所示：</p>
<p data-nodeid="56891" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2E/CgqCHl-o5aqAP6m_AAHuix5hURo100.png" alt="Drawing 8.png" data-nodeid="56895"></p>


<p data-nodeid="31093">Hikari 的配置比较多，你实际工作中想要了解详细配置，可以看一下官方文档：<a href="https://github.com/brettwooldridge/HikariCP" data-nodeid="31312">https://github.com/brettwooldridge/HikariCP</a>，这里我只说一下我们最需要关心的配置，有如下几个：</p>
<pre class="lang-plain" data-nodeid="31094"><code data-language="plain">## 最小空闲链接数量
spring.datasource.hikari.minimum-idle=5
## 空闲链接存活最大时间，默认600000（10分钟）
spring.datasource.hikari.idle-timeout=180000
## 链接池最大链接数，默认是10
spring.datasource.hikari.maximum-pool-size=10
## 此属性控制从池返回的链接的默认自动提交行为,默认值：true
spring.datasource.hikari.auto-commit=true
## 数据源链接池的名称
spring.datasource.hikari.pool-name=MyHikariCP
## 此属性控制池中链接的最长生命周期，值0表示无限生命周期，默认1800000即30分钟
spring.datasource.hikari.max-lifetime=1800000
## 数据库链接超时时间,默认30秒，即30000
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.connection-test-query=SELECT 1mysql
</code></pre>
<p data-nodeid="31095">这里我介绍的主要是针对连接池的配置，研究过线程池和连接池原理的同学都知道，连接池我们不能配置得太大，因为连接池太大的话，会有额外的 CPU 开销，处理连接池的线程切换反而会增加程序的执行时间，减低性能；相应的，连接池也不能配置太小，太小的话可能会增加请求的等待时间，也会降低业务处理的吞吐量。</p>
<p data-nodeid="31096">下面我给你一个推荐一个常见的配置项。</p>
<h4 data-nodeid="31097">Hikari 数据源下的 MySQL 配置最佳实践</h4>
<p data-nodeid="31098">直接通过代码来看看。</p>
<pre class="lang-dart" data-nodeid="65625"><code data-language="dart">##数据源的配置：logger=Slf4JLogger&amp;profileSQL=<span class="hljs-keyword">true</span>是用来debug显示sql的执行日志的
spring.datasource.url=jdbc:mysql:<span class="hljs-comment">//localhost:3306/test?logger=Slf4JLogger&amp;profileSQL=true</span>
spring.datasource.username=root
spring.datasource.password=E6kroWaR9F
##采用默认的
#spring.datasource.hikari.connectionTimeout=<span class="hljs-number">30000</span>
#spring.datasource.hikari.idleTimeout=<span class="hljs-number">300000</span>
##指定一个链接池的名字，方便我们分析线程问题
spring.datasource.hikari.pool-name=jpa-hikari-pool
##最长生命周期<span class="hljs-number">15</span>分钟够了
spring.datasource.hikari.maxLifetime=<span class="hljs-number">900000</span>
spring.datasource.hikari.maximumPoolSize=<span class="hljs-number">8</span>
##最大和最小相对应减少创建线程池的消耗；
spring.datasource.hikari.minimumIdle=<span class="hljs-number">8</span>
spring.datasource.hikari.connectionTestQuery=select <span class="hljs-number">1</span> from dual
##当释放连接到连接池之后，采用默认的自动提交事务
spring.datasource.hikari.autoCommit=<span class="hljs-keyword">true</span>
##用来显示链接测trace日志
logging.level.com.zaxxer.hikari.HikariConfig=DEBUG 
logging.level.com.zaxxer.hikari=TRACE
</code></pre>










<p data-nodeid="31100">通过上面的日志配置，我们在启动的时候可以看到连接池的配置结果和 MySQL 的执行日志：</p>
<p data-nodeid="68240" class="">1.如下日志，显示了Hikari 的 config 配置。</p>

<p data-nodeid="67363" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/23/Ciqc1F-o5ciAb9jeAAR6M63p8e0177.png" alt="Drawing 9.png" data-nodeid="67367"></p>


<p data-nodeid="74287">2.当我们执行一个方法的时候，到底要在一个 MySQL 的 connection 上面执行哪些 SQL 呢？通过如下日志我们可以看得出来。</p>
<p data-nodeid="74288" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2F/CgqCHl-o5dqAOiJCAAUvCxtwstQ913.png" alt="Drawing 10.png" data-nodeid="74292"></p>




<p data-nodeid="71711" class="">3.通过开启 com.zaxxer.hikari.pool.HikariPool 类的 debug 级别，可以实时看到连接池的使用情况：软件日志如下（上图也有体现）：</p>
<pre class="lang-java" data-nodeid="72574"><code data-language="java">com.zaxxer.hikari.pool.HikariPool&nbsp; &nbsp; &nbsp; &nbsp; : jpa-hikari-pool - <span class="hljs-function">Pool <span class="hljs-title">stats</span> <span class="hljs-params">(total=<span class="hljs-number">8</span>, active=<span class="hljs-number">1</span>, idle=<span class="hljs-number">7</span>, waiting=<span class="hljs-number">0</span>)</span>
</span></code></pre>





<p data-nodeid="31113">通过上面的监控日志，你在实际工作中可以根据主机的 CPU 情况和业务处理的耗时情况，再对连接池做适当的调整，但是注意差距不要太大，不要一下将连接池配置几百个，那是错误的配置。</p>
<p data-nodeid="31114">而除了上面的这些日志之外，Hikari 还提供了 Metrics 的监控指标，我们一般配合 Prometheus 使用，甚至可以利用 Granfan 配置一些告警，我们看一下。</p>
<h4 data-nodeid="31115">Hikari 数据通过 Prometheus 的监控指标应用</h4>
<p data-nodeid="31116">就像我们日志里面打印的一样，</p>
<pre class="lang-java" data-nodeid="75153"><code data-language="java">om.zaxxer.hikari.pool.HikariPool        : jpa-hikari-pool - <span class="hljs-function">Pool <span class="hljs-title">stats</span> <span class="hljs-params">(total=<span class="hljs-number">8</span>, active=<span class="hljs-number">0</span>, idle=<span class="hljs-number">8</span>, waiting=<span class="hljs-number">0</span>)</span>
</span></code></pre>

<p data-nodeid="31118">Hikari 的 Metirc 也帮我们提供了 Prometheus 的监控指标，实现方法很简单，代码如下所示：</p>
<pre class="lang-java" data-nodeid="76014"><code data-language="java">1. gradle依赖里面添加
implementation 'io.micrometer:micrometer-registry-prometheus'
2. application.properties里面添加
#Metrics related configurations
management.endpoint.metrics.enabled=true
management.endpoints.web.exposure.include=*
management.endpoint.prometheus.enabled=true
management.metrics.export.prometheus.enabled=true
</code></pre>

<p data-nodeid="77727" class="">然后我们启动项目，通过下图中的地址就可以看到，Prometheus 的 Metrics 里面多了很多 HikariCP 的指标。</p>
<p data-nodeid="77728" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/2F/CgqCHl-o5euAFtXOAAI2DmhXYZQ057.png" alt="Drawing 11.png" data-nodeid="77732"></p>


<p data-nodeid="79445">当看到这些指标之后，我们就可以根据&nbsp;Grafana 社区里面提供的 HikariCP 的监控 Dashboards 的配置文档地址：<a href="https://grafana.com/grafana/dashboards/6083" data-nodeid="79450">https://grafana.com/grafana/dashboards/6083</a>，导入到我们自己的 Grafana 里面，可以通过图表看到如下界面：</p>
<p data-nodeid="79446" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/24/Ciqc1F-o5giAf-jFAAIxax2K82w908.png" alt="Drawing 12.png" data-nodeid="79454"></p>


<p data-nodeid="31124">我们通过这种标准的模板就可以知道 JDBC 的连接情况、Hikari 的连接情况，以及每个连接请求时间、使用时间。这样对我们诊断 DB 性能问题非常有帮助。</p>
<p data-nodeid="31125">下面对其中一些关键指标作一下说明：</p>
<ol data-nodeid="31126">
<li data-nodeid="31127">
<p data-nodeid="31128">totalConnections：总连接数，包括空闲的连接和使用中的连接，即 totalConnections = activeConnection + idleConnections；</p>
</li>
<li data-nodeid="31129">
<p data-nodeid="31130">idleConnections：空闲连接数，也叫可用连接数，也就是连接池里面现成的 DB 连接数；</p>
</li>
<li data-nodeid="31131">
<p data-nodeid="31132">activeConnections：活跃连接数，非业务繁忙期一般都是 0，很快就会释放到连接池里面去；</p>
</li>
<li data-nodeid="31133">
<p data-nodeid="31134">pendingThreads：正在等待连接的线程数量。排查性能问题时，这个指标是一个重要的参考指标，如果正在等待连接的线程在相当长一段时间内数量较多，说明我们的连接没有利用好，是不是占用连接的时间过长了？一旦有 pendingThreads 的数量了可以发个告警，查查原因，或者优化一下连接池；</p>
</li>
<li data-nodeid="31135">
<p data-nodeid="31136">maxConnections：最大连接数，统计指标，统计到目前为止连接的最大数量。</p>
</li>
<li data-nodeid="31137">
<p data-nodeid="31138">minConnections：最小连接数，统计指标，统计到目前为止连接的最小数量。</p>
</li>
<li data-nodeid="31139">
<p data-nodeid="31140">usageTime：每个连接使用的时间，当连接被回收的时候会记录此指标；一般都在 m、s 级别，一旦到 s 级别了可以发个告警；</p>
</li>
<li data-nodeid="31141">
<p data-nodeid="31142">acquireTime：获取每个连接需要等待时间，一个请求获取数据库连接后或者因为超时失败后，会记录此指标。</p>
</li>
<li data-nodeid="31143">
<p data-nodeid="31144">connectionCreateTime：连接创建时间。</p>
</li>
</ol>
<p data-nodeid="31145">在 Granfan 图表或者 Prometheus 里面都可以配置一些邮件或者短信等告警，这样当我们 DB 连接池发生问题的时候就能实时知道。</p>
<p data-nodeid="31146">以上内容涉及了一些运维知识，感兴趣的同学可以研究一下 Prometheus Operator：<a href="https://github.com/prometheus-operator/prometheus-operator" data-nodeid="31360">https://github.com/prometheus-operator/prometheus-operator</a>。我们掌握了 Hikari 的数据源的配置，那么会有同学问数据源 AliDruid 是怎么配置的呢？</p>
<h3 data-nodeid="31147">AliDruidDataSource 的配置与介绍</h3>
<p data-nodeid="31148">在实际工作中，由于 HikariCP 和 Druid 各有千秋，国内的很多开发者都使用 AliDruid 作为数据源，我们看看都是怎么配置的，每一步都很简单。</p>
<p data-nodeid="31149"><strong data-nodeid="31367">第一步：引入 Gradle 依赖。</strong></p>
<pre class="lang-java" data-nodeid="80315"><code data-language="java">implementation <span class="hljs-string">'com.alibaba:druid-spring-boot-starter:1.2.1'</span>
</code></pre>

<p data-nodeid="31151"><strong data-nodeid="31371">第二步：配置数据源。</strong></p>
<pre class="lang-dart" data-nodeid="99257"><code data-language="dart">spring.datasource.druid.url= # 或spring.datasource.url= 
spring.datasource.druid.username= # 或spring.datasource.username=
spring.datasource.druid.password= # 或spring.datasource.password=
spring.datasource.druid.driver-<span class="hljs-class"><span class="hljs-keyword">class</span>-<span class="hljs-title">name</span>= #或 <span class="hljs-title">spring</span>.<span class="hljs-title">datasource</span>.<span class="hljs-title">driver</span>-<span class="hljs-title">class</span>-<span class="hljs-title">name</span>=
</span></code></pre>







<p data-nodeid="31153"><strong data-nodeid="31375">第三步：配置连接池。</strong></p>
<pre class="lang-dart" data-nodeid="94091"><code data-language="dart">spring.datasource.druid.initial-size=
spring.datasource.druid.max-active=
spring.datasource.druid.min-idle=
spring.datasource.druid.max-wait=
spring.datasource.druid.pool-prepared-statements=
spring.datasource.druid.max-pool-prepared-statement-per-connection-size= 
spring.datasource.druid.max-open-prepared-statements= #和上面的等价
spring.datasource.druid.validation-query=
spring.datasource.druid.validation-query-timeout=
spring.datasource.druid.test-<span class="hljs-keyword">on</span>-borrow=
spring.datasource.druid.test-<span class="hljs-keyword">on</span>-<span class="hljs-keyword">return</span>=
spring.datasource.druid.test-<span class="hljs-keyword">while</span>-idle=
spring.datasource.druid.time-between-eviction-runs-millis=
spring.datasource.druid.min-evictable-idle-time-millis=
spring.datasource.druid.max-evictable-idle-time-millis=
spring.datasource.druid.filters= #配置多个英文逗号分隔
....<span class="hljs-comment">//more</span>
</code></pre>















<p data-nodeid="31155">通过以上三步就可以完成 Druid 数据源的配置了，需要注意的是，我们需要把 HikariCP 数据源给排除掉，而其他 Druid 的配置，比如监控，官方的介绍还是挺详细的：<a href="https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter" data-nodeid="31379">https://github.com/alibaba/druid/tree/master/druid-spring-boot-starter</a>，你可以看一下，我就不多说了。</p>
<p data-nodeid="31156">其官方的源码也比较简单，按照我们上面分析 HikariCP 数据源的方法，可以找一下 aliDruid 的源码，其加载的入口类：<a href="https://github.com/alibaba/druid/blob/master/druid-spring-boot-starter/src/main/java/com/alibaba/druid/spring/boot/autoconfigure/DruidDataSourceAutoConfigure.java" data-nodeid="31384">https://github.com/alibaba/druid/blob/master/druid-spring-boot-starter/src/main/java/com/alibaba/druid/spring/boot/autoconfigure/DruidDataSourceAutoConfigure.java</a>，你一步一步去查看即可，我在这里就不重点介绍了。</p>
<p data-nodeid="31157">接下来我们看看数据里面的表的字段，和我们实体里面字段的映射策略都有哪些。</p>
<h3 data-nodeid="31158">Naming 命名策略详解及其实践</h3>
<p data-nodeid="31159">我们在配置 @Entity 时，一定会有同学好奇表名、字段名、外键名、实体字段、@Column 和数据库的字段之间，映射关系是怎么样的？默认规则映射规则又是什么？如果和默认不一样该怎么扩展？</p>
<p data-nodeid="31160">我们下面只介绍 Hibernate 5 的命名策略，因为 H4 已经不推荐使用了，我们直接看最新的即可。Hibernate 5 里面把实体和数据库的字段名和表名的映射分成了两个步骤。</p>
<p data-nodeid="31161"><strong data-nodeid="31399">第一步：通过<strong data-nodeid="31398"><strong data-nodeid="31397">ImplicitNamingStrategy</strong></strong>先找到实例里面定义的逻辑的字段名字。</strong></p>
<p data-nodeid="31162">这是通过ImplicitNamingStrategy 的实现类指定逻辑字段查找策略，也就是当实体里面定义了 @Table、@Column 注解的时候，以注解指定名字返回；而当没有这些注解的时候，返回的是实体里面的字段的名字。</p>
<p data-nodeid="31163">其中，org.hibernate.boot.model.naming.ImplicitNamingStrategy 是一个接口，ImplicitNamingStrategyJpaCompliantImpl 这个实现类兼容 JPA 2.0 的字段映射规范。除此之外，还有如下四个实现类：</p>
<ul data-nodeid="100970">
<li data-nodeid="100971">
<p data-nodeid="100972">ImplicitNamingStrategyLegacyHbmImpl：兼容 Hibernate 老版本中的命名规范；</p>
</li>
<li data-nodeid="100973">
<p data-nodeid="100974">ImplicitNamingStrategyLegacyJpaImpl：兼容 JPA 1.0 规范中的命名规范；</p>
</li>
<li data-nodeid="100975">
<p data-nodeid="100976">ImplicitNamingStrategyComponentPathImpl：@Embedded 等注解标志的组件处理是通过 attributePath 完成的，因此如果我们在使用 @Embedded 注解的时候，如果要指定命名规范，可以直接继承这个类来实现；</p>
</li>
<li data-nodeid="100977">
<p data-nodeid="100978">SpringImplicitNamingStrategy：默认的 spring data 2.2.3 的策略，只是扩展了 ImplicitNamingStrategyJpaCompliantImpl 里面的 JoinTableName 的方法，如下图所示：</p>
</li>
</ul>
<p data-nodeid="100979" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/25/Ciqc1F-o5jmAeh2mAAB0yWE0YWY107.png" alt="Drawing 13.png" data-nodeid="100986"></p>


<p data-nodeid="31174">这里我们只需要关心 SpringImplicitNamingStrategy 就可以了，其他的我们基本上用不到。那么 SpringImplicitNamingStrategy 效果如何呢？我们举个例子看一下 UserInfo 实体，代码如下：</p>
<pre class="lang-java" data-nodeid="101847"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table(name = "userInfo")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntity</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-keyword">private</span> String lastName;
   <span class="hljs-meta">@Column(name = "myAddress")</span>
   <span class="hljs-keyword">private</span> String emailAddress;
}
</code></pre>

<p data-nodeid="31176">通过第一步可以得到如下逻辑字段的映射结果：</p>
<pre class="lang-java" data-nodeid="102708"><code data-language="java">UserInfo -&gt; userInfo
id-&gt;id
ages-&gt;ages
lastName -&gt; lastName
emailAddress -&gt; myAddress
</code></pre>

<p data-nodeid="103569" class=""><strong data-nodeid="103573">第二步：通过 PhysicalNamingStrategy 将逻辑字段转化成数据库的物理字段名字。</strong></p>

<p data-nodeid="105278">它的实现类负责将逻辑字段转化成带下划线，或者统一给字段加上前缀，又或者加上双引号等格式的数据库字段名字，其主要的接口是：org.hibernate.boot.model.naming.PhysicalNamingStrategy，而它的实现类也只有两个，如下图所示：</p>
<p data-nodeid="105279" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/25/Ciqc1F-o5kiAbR_iAAAz6GG2wMk729.png" alt="Drawing 14.png" data-nodeid="105283"></p>


<p data-nodeid="106997" class="">1.PhysicalNamingStrategyStandardImpl：这个类什么都没干，即直接将第一个步骤得到的逻辑字段名字当成数据库的字段名字使用。这个主要的应用场景是，如果某些字段的命名格式不是下划线的格式，我们想通过 @Column 的方式显示声明的话，可以把默认第二步的策略改成 PhysicalNamingStrategyStandardImpl。那么如果再套用第一步的例子，经过这个类的转化会变成如下形式：</p>
<pre class="lang-java" data-nodeid="106998"><code data-language="java">userInfo -&gt; userInfo
id-&gt;id
ages-&gt;ages
lastName -&gt; lastName
     myAddress -&gt; myAddress
</code></pre>



<p data-nodeid="109554">可以看出来逻辑名字到物理名字是保持不变的。</p>
<p data-nodeid="109555">2.SpringPhysicalNamingStrategy：这个类是将第一步得到的逻辑字段名字的大写字母前面加上下划线，并且全部转化成小写，将会标识出是否需要加上双引号。此种是默认策略。我们举个例子，第一步得到的逻辑字段就会变成如下映射：</p>


<pre class="lang-java" data-nodeid="110406"><code data-language="java">userInfo -&gt; user_info
id-&gt;id
ages-&gt;ages
lastName -&gt; last_name
myAddress -&gt; my_address
</code></pre>





<p data-nodeid="31190">我们把刚才的实体执行一下，可以看到生成的表的结构如下：</p>
<pre class="lang-java" data-nodeid="111255"><code data-language="java">Hibernate: <span class="hljs-function">create table <span class="hljs-title">user_info</span> <span class="hljs-params">(id bigint not <span class="hljs-keyword">null</span>, create_time timestamp, create_user_id integer, last_modified_time timestamp, last_modified_user_id integer, version integer, ages integer, my_address varchar(<span class="hljs-number">255</span>)</span>, last_name <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, telephone <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)；
</span></code></pre>

<p data-nodeid="112944">你也可以通过在 SpringPhysicalNamingStrategy 类里面设置断点，来一步一步地验证我们的说法，如下图所示：</p>
<p data-nodeid="112945" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/25/Ciqc1F-o5mSASabwAADWrC44gPw955.png" alt="Drawing 15.png" data-nodeid="112949"></p>


<p data-nodeid="31194">以上就是 Naming 命名策略的详解及其实践，不知道我在这部分开头提到的那几个问题你有没有掌握，如果还是存在疑问，你要多跟着我的步骤实践几次。下面我们了解一下它的加载原理吧。</p>
<h4 data-nodeid="31195">加载原理与自定义方法</h4>
<p data-nodeid="31196">如果我们修改默认策略，只需要在 application.properties 里面修改下面代码所示的两个配置，换成自己的自定义的类即可。</p>
<pre class="lang-dart" data-nodeid="119741"><code data-language="dart">spring.jpa.hibernate.naming.implicit-strategy=org.springframework.boot.orm.jpa.hibernate.SpringImplicitNamingStrategy
spring.jpa.hibernate.naming.physical-strategy=org.springframework.boot.orm.jpa.hibernate.SpringPhysicalNamingStrategy
</code></pre>








<p data-nodeid="121430">如果我们直接搜索：spring.jpa.hibernate 就会发现，其默认配置是在 org.springframework.boot.autoconfigure.orm.jpa.HibernateProerties 这类里面的，如下图所示的方法中进行加载。</p>
<p data-nodeid="121431" class=""><img src="https://s0.lgstatic.com/i/image/M00/6A/25/Ciqc1F-o5nSAUv1hAAJk2XpXe2k189.png" alt="Drawing 16.png" data-nodeid="121435"></p>


<p data-nodeid="31200">其中，IMPLICIT_NAMING_STRATEGY 和 PHYSICAL_NAMING_STRATEGY 的值如下述代码所示，它是 Hibernate 5 的配置变量，用来改变 Hibernate的 naming 的策略。</p>
<pre class="lang-java" data-nodeid="122284"><code data-language="java">String IMPLICIT_NAMING_STRATEGY = <span class="hljs-string">"hibernate.implicit_naming_strategy"</span>;
String PHYSICAL_NAMING_STRATEGY = <span class="hljs-string">"hibernate.physical_naming_strategy"</span>;
</code></pre>

<p data-nodeid="31202">如果我们自定义的话，直接继承 SpringPhysicalNamingStrategy 这个类，然后覆盖需要实现的方法即可。那么它实际的应用场景都有哪些呢？</p>
<h4 data-nodeid="31203">实际应用场景</h4>
<p data-nodeid="31204">有时候我们接触到的系统可能是老系统，表和字段的命名规范不一定是下划线形式，有可能驼峰式的命名法，也有可能不同的业务有不同的表名前缀。不管是哪一种，我们都可以通过修改第二阶段：物理映射的策略，改成 PhysicalNamingStrategyStandardImpl 的形式，请看代码。</p>
<pre class="lang-js" data-nodeid="126529"><code data-language="js">spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
</code></pre>





<p data-nodeid="31206">这样可以使 @Column/@Table 等注解的自定义值生效，或者改成自定义的 MyPhysicalNamingStrategy。不过我不建议你修改&nbsp;implicit-strategy，因为没有必要，你只要在&nbsp;physical-strategy 上做文章就足够了。</p>
<h3 data-nodeid="31207">总结</h3>
<p data-nodeid="31208">本讲的内容到这里就结束了。今天为你介绍了 Datasource 是什么，讲解了数据源和 Connection 的关系，并且通过源码分析，让你知道了不同的数据源应该怎么配置，最常见的数据源 Hikari 的配置和监控是怎样的。此外，我还给你介绍了和数据库相关的字段映射策略。</p>
<p data-nodeid="31209">最后，希望你在学习的同时多去思考，因为不同的版本可能实现的代码是不一样的，但是思考方式是不变的，你可以学着举一反三，学会如何看源码，因为看源码可能要比查看文档资料更靠谱和快捷。</p>
<p data-nodeid="31210">如果你觉得这一讲对你有帮助，就动动手指分享吧。下一讲我会为你介绍多数据源应该怎么配置，它的最佳实践又是什么呢？你可以先思考一下，也欢迎你在留言区发表自己的看法，让我们一起活跃思维，碰撞出不一样的火花！</p>
<blockquote data-nodeid="31211">
<p data-nodeid="31212">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="31457">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### **斌：
> 课程好像越来越深了，还是前面学起来轻松/(ㄒoㄒ)/~~

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 那就再回过头巩固巩固吧，哪里不会记得留言哦

