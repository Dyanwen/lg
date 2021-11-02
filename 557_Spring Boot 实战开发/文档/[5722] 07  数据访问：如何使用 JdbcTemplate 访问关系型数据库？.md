<p data-nodeid="2968" class="">06 讲我们详细介绍了 JDBC 规范的相关内容，JDBC 规范是 Java 领域中使用最广泛的数据访问标准，目前市面上主流的数据访问框架都是构建在 JDBC 规范之上。</p>
<p data-nodeid="3122" class="">因为 JDBC 是偏底层的操作规范，所以关于如何使用 JDBC 规范进行关系型数据访问的实现方式有很多（区别在于对 JDBC 规范的封装程度不同），而在 Spring 中，同样提供了 JdbcTemplate 模板工具类实现数据访问，它简化了 JDBC 规范的使用方法，今天我们将围绕这个模板类展开讨论。</p>

<h3 data-nodeid="2970">数据模型和 Repository 层设计</h3>
<p data-nodeid="4672" class="">引入 JdbcTemplate 模板工具类之前，我们回到 SpringCSS 案例，先给出 order-service 中的数据模型为本讲内容的展开做一些铺垫。</p>





<p data-nodeid="2972">我们知道一个订单中往往涉及一个或多个商品，所以在本案例中，我们主要通过一对多的关系来展示数据库设计和实现方面的技巧。而为了使描述更简单，我们把具体的业务字段做了简化。Order 类的定义如下代码所示：</p>
<pre class="lang-java" data-nodeid="2973"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Order</span></span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Long id; <span class="hljs-comment">//订单Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String orderNumber; <span class="hljs-comment">//订单编号</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String deliveryAddress; <span class="hljs-comment">//物流地址</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> List&lt;Goods&gt; goodsList;&nbsp; <span class="hljs-comment">//商品列表</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略了 getter/setter</span>
}
</code></pre>
<p data-nodeid="2974">其中代表商品的 Goods 类定义如下：</p>
<pre class="lang-java" data-nodeid="2975"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Goods</span> </span>{
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Long id; <span class="hljs-comment">//商品Id</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String goodsCode; <span class="hljs-comment">//商品编号</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> String goodsName; <span class="hljs-comment">//商品名称</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> Double price; <span class="hljs-comment">//商品价格</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略了 getter/setter</span>
}
</code></pre>
<p data-nodeid="2976">从以上代码，我们不难看出一个订单可以包含多个商品，因此设计关系型数据库表时，我们首先会构建一个中间表来保存 Order 和 Goods 这层一对多关系。在本课程中，我们使用 MySQL 作为关系型数据库，对应的数据库 Schema 定义如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2977"><code data-language="xml">DROP TABLE IF EXISTS `order`;
DROP TABLE IF EXISTS `goods`;
DROP TABLE IF EXISTS `order_goods`;
&nbsp;
create table `order` (
&nbsp;&nbsp;&nbsp; `id` bigint(20) NOT NULL AUTO_INCREMENT,
&nbsp;&nbsp;&nbsp; `order_number` varchar(50) not null,
&nbsp;&nbsp;&nbsp; `delivery_address` varchar(100) not null,
&nbsp; `create_time` timestamp not null DEFAULT CURRENT_TIMESTAMP,
&nbsp;&nbsp;&nbsp; PRIMARY KEY (`id`)
);
&nbsp;
create table `goods` (
&nbsp; `id` bigint(20) NOT NULL AUTO_INCREMENT,
&nbsp; `goods_code` varchar(50) not null,
&nbsp; `goods_name` varchar(50) not null,
&nbsp; `goods_price` double not null,
&nbsp; `create_time` timestamp not null DEFAULT CURRENT_TIMESTAMP,
&nbsp;&nbsp;&nbsp; PRIMARY KEY (`id`)
);
&nbsp;
create table `order_goods` (
&nbsp;&nbsp;&nbsp; `order_id` bigint(20) not null,
&nbsp;&nbsp;&nbsp; `goods_id` bigint(20) not null,
&nbsp;&nbsp;&nbsp; foreign key(`order_id`) references `order`(`id`),
&nbsp;&nbsp;&nbsp; foreign key(`goods_id`) references `goods`(`id`)
);
</code></pre>
<p data-nodeid="2978">基于以上数据模型，我们将完成 order-server 中的 Repository 层组件的设计和实现。首先，我们需要设计一个 OrderRepository 接口，用来抽象数据库访问的入口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="2979"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">OrderRepository</span> </span>{

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Order <span class="hljs-title">addOrder</span><span class="hljs-params">(Order order)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Order <span class="hljs-title">getOrderById</span><span class="hljs-params">(Long orderId)</span></span>;

&nbsp;&nbsp;&nbsp; <span class="hljs-function">Order <span class="hljs-title">getOrderDetailByOrderNumber</span><span class="hljs-params">(String orderNumber)</span></span>;
}
</code></pre>
<p data-nodeid="4982" class="">这个接口非常简单，方法都是自解释的。不过请注意，这里的 OrderRepository 并没有继承任何父接口，完全是一个自定义的、独立的 Repository。</p>

<p data-nodeid="2981">针对上述 OrderRepository 中的接口定义，我们将构建一系列的实现类。</p>
<ul data-nodeid="2982">
<li data-nodeid="2983">
<p data-nodeid="2984">OrderRawJdbcRepository：使用原生 JDBC 进行数据库访问</p>
</li>
<li data-nodeid="2985">
<p data-nodeid="2986">OrderJdbcRepository：使用 JdbcTemplate 进行数据库访问</p>
</li>
<li data-nodeid="2987">
<p data-nodeid="2988">OrderJpaRepository：使用 Spring Data JPA 进行数据库访问</p>
</li>
</ul>
<p data-nodeid="2989">上述实现类中的 OrderJpaRepository 我们会放到 10 讲《ORM 集成：如何使用 Spring Data JPA 访问关系型数据库？》中进行展开，而 OrderRawJdbcRepository 最基础，不是本课程的重点，因此 07 讲我们只针对 OrderRepository 中 getOrderById 方法的实现过程重点介绍，也算是对 06 讲的回顾和扩展。</p>
<p data-nodeid="2990">OrderRawJdbcRepository 类中实现方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="2991"><code data-language="java"><span class="hljs-meta">@Repository("orderRawJdbcRepository")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderRawJdbcRepository</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">OrderRepository</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> DataSource dataSource;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">getOrderById</span><span class="hljs-params">(Long orderId)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Connection connection = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PreparedStatement statement = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ResultSet resultSet = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">try</span> {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; connection = dataSource.getConnection();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement = connection.prepareStatement(<span class="hljs-string">"select id, order_number, delivery_address from `order` where id=?"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; statement.setLong(<span class="hljs-number">1</span>, orderId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resultSet = statement.executeQuery();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = <span class="hljs-keyword">null</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (resultSet.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; order = <span class="hljs-keyword">new</span> Order(resultSet.getLong(<span class="hljs-string">"id"</span>), resultSet.getString(<span class="hljs-string">"order_number"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; resultSet.getString(<span class="hljs-string">"delivery_address"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
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
&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//省略其他 OrderRepository 接口方法实现</span>
}
</code></pre>
<p data-nodeid="5292" class="te-preview-highlight">这里，值得注意的是，我们首先需要在类定义上添加 @Repository 注解，标明这是能够被 Spring 容器自动扫描的 Javabean，再在 @Repository 注解中指定这个 Javabean 的名称为"orderRawJdbcRepository"，方便 Service 层中根据该名称注入 OrderRawJdbcRepository 类。</p>

<p data-nodeid="2993">可以看到，上述代码使用了 JDBC 原生 DataSource、Connection、PreparedStatement、ResultSet 等核心编程对象完成针对“order”表的一次查询。代码流程看起来比较简单，其实也比较烦琐，学到这里，我们可以结合上一课时的内容理解上述代码。</p>
<p data-nodeid="2994">请注意，如果我们想运行这些代码，千万别忘了在 Spring Boot 的配置文件中添加对 DataSource 的定义，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2995"><code data-language="xml">spring:
&nbsp; datasource:
&nbsp;&nbsp;&nbsp; driver-class-name: com.mysql.cj.jdbc.Driver
&nbsp;&nbsp;&nbsp; url: jdbc:mysql://127.0.0.1:3306/appointment
&nbsp;&nbsp;&nbsp; username: root
&nbsp;&nbsp;&nbsp; password: root
</code></pre>
<p data-nodeid="2996">回顾完原生 JDBC 的使用方法，接下来就引出今天的重点，即 JdbcTemplate 模板工具类，我们来看看它如何简化数据访问操作。</p>
<h3 data-nodeid="2997">使用 JdbcTemplate 操作数据库</h3>
<p data-nodeid="2998">要想在应用程序中使用 JdbcTemplate，首先我们需要引入对它的依赖，如下代码所示：</p>
<pre class="lang-xml" data-nodeid="2999"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-jdbc<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="3000">JdbcTemplate 提供了一系列的 query、update、execute 重载方法应对数据的 CRUD 操作。</p>
<h4 data-nodeid="3001">使用 JdbcTemplate 实现查询</h4>
<p data-nodeid="3002">基于 SpringCSS 案例，我们先来讨论一下最简单的查询操作，并对 OrderRawJdbcRepository 中的 getOrderById 方法进行重构。为此，我们构建了一个新的 OrderJdbcRepository 类并同样实现了 OrderRepository 接口，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3003"><code data-language="java"><span class="hljs-meta">@Repository("orderJdbcRepository")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderJdbcRepository</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">OrderRepository</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> JdbcTemplate jdbcTemplate;
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">OrderJdbcRepository</span><span class="hljs-params">(JdbcTemplate jdbcTemplate)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.jdbcTemplate = jdbcTemplate;
	}
}
</code></pre>
<p data-nodeid="3004">可以看到，这里通过构造函数注入了 JdbcTemplate 模板类。</p>
<p data-nodeid="3005">而 OrderJdbcRepository 的 getOrderById 方法实现过程如下代码所示：</p>
<pre class="lang-java" data-nodeid="3006"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">getOrderById</span><span class="hljs-params">(Long orderId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = jdbcTemplate.queryForObject(<span class="hljs-string">"select id, order_number, delivery_address from `order` where id=?"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>::mapRowToOrder, orderId);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
}
</code></pre>
<p data-nodeid="3007">显然，这里使用了 JdbcTemplate 的 queryForObject 方法执行查询操作，该方法传入目标 SQL、参数以及一个 RowMapper 对象。其中 RowMapper 定义如下：</p>
<pre class="lang-java" data-nodeid="3008"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">RowMapper</span>&lt;<span class="hljs-title">T</span>&gt; </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-function">T <span class="hljs-title">mapRow</span><span class="hljs-params">(ResultSet rs, <span class="hljs-keyword">int</span> rowNum)</span> <span class="hljs-keyword">throws</span> SQLException</span>;
}
</code></pre>
<p data-nodeid="3009">从 mapRow 方法定义中，我们不难看出 RowMapper 的作用就是处理来自 ResultSet 中的每一行数据，并将来自数据库中的数据映射成领域对象。例如，使用 getOrderById 中用到的 mapRowToOrder 方法完成对 Order 对象的映射，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3010"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Order <span class="hljs-title">mapRowToOrder</span><span class="hljs-params">(ResultSet rs, <span class="hljs-keyword">int</span> rowNum)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Order(rs.getLong(<span class="hljs-string">"id"</span>), rs.getString(<span class="hljs-string">"order_number"</span>), rs.getString(<span class="hljs-string">"delivery_address"</span>));
}
</code></pre>
<p data-nodeid="3011">讲到这里，你可能注意到 getOrderById 方法实际上只是获取了 Order 对象中的订单部分信息，并不包含商品数据。</p>
<p data-nodeid="3012">接下来，我们再来设计一个 getOrderDetailByOrderNumber 方法，根据订单编号获取订单以及订单中所包含的所有商品信息，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3013"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">getOrderDetailByOrderNumber</span><span class="hljs-params">(String orderNumber)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 Order 基础信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order order = jdbcTemplate.queryForObject(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"select id, order_number, delivery_address from `order` where order_number=?"</span>, <span class="hljs-keyword">this</span>::mapRowToOrder,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orderNumber);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">if</span> (order == <span class="hljs-keyword">null</span>)
&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//获取 Order 与 Goods 之间的关联关系，找到给 Order 中的所有 GoodsId</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Long orderId = order.getId();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Long&gt; goodsIds = jdbcTemplate.query(<span class="hljs-string">"select order_id, goods_id from order_goods where order_id=?"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">new</span> ResultSetExtractor&lt;List&lt;Long&gt;&gt;() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Long&gt; <span class="hljs-title">extractData</span><span class="hljs-params">(ResultSet rs)</span> <span class="hljs-keyword">throws</span> SQLException, DataAccessException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Long&gt; list = <span class="hljs-keyword">new</span> ArrayList&lt;Long&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">while</span> (rs.next()) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; list.add(rs.getLong(<span class="hljs-string">"goods_id"</span>));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> list;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }, orderId);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//根据 GoodsId 分别获取 Goods 信息并填充到 Order 对象中</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Long goodsId : goodsIds) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Goods goods = getGoodsById(goodsId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; order.addGoods(goods);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
}
</code></pre>
<p data-nodeid="3014">上述代码有点复杂，我们分成几个部分来讲解。</p>
<p data-nodeid="3015">首先，我们获取 Order 基础信息，并通过 Order 中的 Id 编号从中间表中获取所有 Goods 的 Id 列表，通过遍历这个 Id 列表再分别获取 Goods 信息，最后将 Goods 信息填充到 Order 中，从而构建一个完整的 Order 对象。</p>
<p data-nodeid="3016">这里通过 Id 获取 Goods 数据的实现方法也与 getOrderById 方法的实现过程一样，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3017"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Goods <span class="hljs-title">getGoodsById</span><span class="hljs-params">(Long goodsId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> jdbcTemplate.queryForObject(<span class="hljs-string">"select id, goods_code, goods_name, price from goods where id=?"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>::mapRowToGoods, goodsId);
}
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">private</span> Goods <span class="hljs-title">mapRowToGoods</span><span class="hljs-params">(ResultSet rs, <span class="hljs-keyword">int</span> rowNum)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> Goods(rs.getLong(<span class="hljs-string">"id"</span>), rs.getString(<span class="hljs-string">"goods_code"</span>), rs.getString(<span class="hljs-string">"goods_name"</span>),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; rs.getDouble(<span class="hljs-string">"price"</span>));
}
</code></pre>
<h4 data-nodeid="3018">使用 JdbcTemplate 实现插入</h4>
<p data-nodeid="3019">在 JdbcTemplate 中，我们可以通过 update 方法实现数据的插入和更新。针对 Order 和 Goods 中的关联关系，插入一个 Order 对象需要同时完成两张表的更新，即 order 表和 order_goods 表，因此插入 Order 的实现过程也分成两个阶段，如下代码所示的 addOrderWithJdbcTemplate 方法展示了这一过程：</p>
<pre class="lang-java" data-nodeid="3020"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Order <span class="hljs-title">addOrderDetailWithJdbcTemplate</span><span class="hljs-params">(Order order)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//插入 Order 基础信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Long orderId = saveOrderWithJdbcTemplate(order);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; order.setId(orderId);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//插入 Order 与 Goods 的对应关系</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Goods&gt; goodsList = order.getGoods();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Goods goods : goodsList) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; saveGoodsToOrderWithJdbcTemplate(goods, orderId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
}
</code></pre>
<p data-nodeid="3021">可以看到，这里同样先是插入 Order 的基础信息，然后再遍历 Order 中的 Goods 列表并逐条进行插入。其中的 saveOrderWithJdbcTemplate 方法如下代码所示：</p>
<pre class="lang-java" data-nodeid="3022"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Long <span class="hljs-title">saveOrderWithJdbcTemplate</span><span class="hljs-params">(Order order)</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PreparedStatementCreator psc = <span class="hljs-keyword">new</span> PreparedStatementCreator() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> PreparedStatement <span class="hljs-title">createPreparedStatement</span><span class="hljs-params">(Connection con)</span> <span class="hljs-keyword">throws</span> SQLException </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; PreparedStatement ps = con.prepareStatement(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-string">"insert into `order` (order_number, delivery_address) values (?, ?)"</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Statement.RETURN_GENERATED_KEYS);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setString(<span class="hljs-number">1</span>, order.getOrderNumber());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ps.setString(<span class="hljs-number">2</span>, order.getDeliveryAddress());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> ps;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; };
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; KeyHolder keyHolder = <span class="hljs-keyword">new</span> GeneratedKeyHolder();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jdbcTemplate.update(psc, keyHolder);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> keyHolder.getKey().longValue();
}
</code></pre>
<p data-nodeid="3023">上述 saveOrderWithJdbcTemplate 的方法比想象中要复杂，主要原因在于我们需要在插入 order 表的同时返回数据库中所生成的自增主键，因此，这里使用了 PreparedStatementCreator 工具类封装 PreparedStatement 对象的构建过程，并在 PreparedStatement 的创建过程中设置了 Statement.RETURN_GENERATED_KEYS 用于返回自增主键。然后我们构建了一个 GeneratedKeyHolder 对象用于保存所返回的自增主键。这是使用 JdbcTemplate 实现带有自增主键数据插入的一种标准做法，你可以参考这一做法并应用到日常开发过程中。</p>
<p data-nodeid="3024">至于用于插入 Order 与 Goods 关联关系的 saveGoodsToOrderWithJdbcTemplate 方法就比较简单了，直接调用 JdbcTemplate 的 update 方法插入数据即可，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3025"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">saveGoodsToOrderWithJdbcTemplate</span><span class="hljs-params">(Goods goods, <span class="hljs-keyword">long</span> orderId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jdbcTemplate.update(<span class="hljs-string">"insert into order_goods (order_id, goods_id) "</span> + <span class="hljs-string">"values (?, ?)"</span>, orderId, goods.getId());
}
</code></pre>
<p data-nodeid="3026">接下来，我们需要实现插入 Order 的整个流程，先实现 Service 类和 Controller 类，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3027"><code data-language="java"><span class="hljs-meta">@Service</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderService</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Autowired</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Qualifier("orderJdbcRepository")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">private</span> OrderRepository orderRepository;

&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">addOrder</span><span class="hljs-params">(Order order)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderRepository.addOrder(order);
&nbsp;&nbsp;&nbsp; } 
}
&nbsp;
<span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value="orders")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderController</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@RequestMapping(value = "", method = RequestMethod.POST)</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> Order <span class="hljs-title">addOrder</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Order order)</span> </span>{

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Order result = orderService.addOrder(order);
&nbsp;&nbsp;&nbsp;  <span class="hljs-keyword">return</span> result;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="3028">这两个类都是直接对 orderJdbcRepository 中的方法进行封装调用，操作非常简单。然后，我们打开 Postman，并在请求消息体中输入如下内容：</p>
<pre class="lang-xml" data-nodeid="3029"><code data-language="xml">{
&nbsp;&nbsp;&nbsp;&nbsp;"orderNumber"&nbsp;:&nbsp;"Order10002",
&nbsp;&nbsp;&nbsp;&nbsp;"deliveryAddress"&nbsp;:&nbsp;"test_address2",
&nbsp;&nbsp;&nbsp;&nbsp;"goods":&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"id":&nbsp;1,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"goodsCode":&nbsp;"GoodsCode1",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"goodsName":&nbsp;"GoodsName1",
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;"price":&nbsp;100.0
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;]
}
</code></pre>
<p data-nodeid="3030">通过 Postman 向<a href="http://localhost:8081/orders" data-nodeid="3101">http://localhost:8081/orders</a>端点发起 Post 请求后，我们发现 order 表和 order_goods 表中的数据都已经正常插入。</p>
<h4 data-nodeid="3031">使用 SimpleJdbcInsert 简化数据插入过程</h4>
<p data-nodeid="3032">虽然通过 JdbcTemplate 的 update 方法可以完成数据的正确插入，我们不禁发现这个实现过程还是比较复杂，尤其是涉及自增主键的处理时，代码显得有点臃肿。那么有没有更加简单的实现方法呢？</p>
<p data-nodeid="3033">答案是肯定的，Spring Boot 针对数据插入场景专门提供了一个 SimpleJdbcInsert 工具类，SimpleJdbcInsert 本质上是在 JdbcTemplate 的基础上添加了一层封装，提供了一组 execute、executeAndReturnKey 以及 executeBatch 重载方法来简化数据插入操作。</p>
<p data-nodeid="3034">通常，我们可以在 Repository 实现类的构造函数中对 SimpleJdbcInsert 进行初始化，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3035"><code data-language="java"><span class="hljs-keyword">private</span> JdbcTemplate jdbcTemplate;
<span class="hljs-keyword">private</span> SimpleJdbcInsert orderInserter;
<span class="hljs-keyword">private</span> SimpleJdbcInsert orderGoodsInserter;
&nbsp;
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">OrderJdbcRepository</span><span class="hljs-params">(JdbcTemplate jdbcTemplate)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.jdbcTemplate = jdbcTemplate;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.orderInserter = <span class="hljs-keyword">new</span> SimpleJdbcInsert(jdbcTemplate).withTableName(<span class="hljs-string">"`order`"</span>).usingGeneratedKeyColumns(<span class="hljs-string">"id"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">this</span>.orderGoodsInserter = <span class="hljs-keyword">new</span> SimpleJdbcInsert(jdbcTemplate).withTableName(<span class="hljs-string">"order_goods"</span>);
}
</code></pre>
<p data-nodeid="3036">可以看到，这里首先注入了一个 JdbcTemplate 对象，然后我们基于 JdbcTemplate 并针对 order 表和 order_goods 表分别初始化了两个 SimpleJdbcInsert 对象 orderInserter 和 orderGoodsInserter。其中 orderInserter 中还使用了 usingGeneratedKeyColumns 方法设置自增主键列。</p>
<p data-nodeid="3037">基于 SimpleJdbcInsert，完成 Order 对象的插入就非常简单了，实现方式如下所示：</p>
<pre class="lang-java" data-nodeid="3038"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Long <span class="hljs-title">saveOrderWithSimpleJdbcInsert</span><span class="hljs-params">(Order order)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; values = <span class="hljs-keyword">new</span> HashMap&lt;String, Object&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; values.put(<span class="hljs-string">"order_number"</span>, order.getOrderNumber());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; values.put(<span class="hljs-string">"delivery_address"</span>, order.getDeliveryAddress());
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Long orderId = orderInserter.executeAndReturnKey(values).longValue();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> orderId;
}
</code></pre>
<p data-nodeid="3039">我们通过构建一个 Map 对象，然后把需要添加的字段设置成一个个键值对。通过SimpleJdbcInsert 的 executeAndReturnKey 方法在插入数据的同时直接返回自增主键。同样，完成 order_goods 表的操作只需要几行代码就可以了，如下代码所示：</p>
<pre class="lang-java" data-nodeid="3040"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">saveGoodsToOrderWithSimpleJdbcInsert</span><span class="hljs-params">(Goods goods, <span class="hljs-keyword">long</span> orderId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Map&lt;String, Object&gt; values = <span class="hljs-keyword">new</span> HashMap&lt;&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; values.put(<span class="hljs-string">"order_id"</span>, orderId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; values.put(<span class="hljs-string">"goods_id"</span>, goods.getId());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; orderGoodsInserter.execute(values);
}
</code></pre>
<p data-nodeid="3041">这里用到了 SimpleJdbcInsert 提供的 execute 方法，我们可以把这些方法组合起来对 addOrderDetailWithJdbcTemplate 方法进行重构，从而得到如下所示的 addOrderDetailWithSimpleJdbcInsert 方法：</p>
<pre class="lang-java" data-nodeid="3042"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> Order <span class="hljs-title">addOrderDetailWithSimpleJdbcInsert</span><span class="hljs-params">(Order order)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//插入 Order 基础信息</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Long orderId = saveOrderWithSimpleJdbcInsert(order);
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; order.setId(orderId);

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//插入 Order 与 Goods 的对应关系</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; List&lt;Goods&gt; goodsList = order.getGoods();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">for</span> (Goods goods : goodsList) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; saveGoodsToOrderWithSimpleJdbcInsert(goods, orderId);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> order;
}
</code></pre>
<p data-nodeid="3043">详细的代码清单可以参考课程的案例代码，你也可以基于 Postman 对重构后的代码进行尝试。</p>
<h3 data-nodeid="3044">小结与预告</h3>
<p data-nodeid="3045">JdbcTemplate 模板工具类是一个基于 JDBC 规范实现数据访问的强大工具，是一个优秀的工具类。它对常见的 CRUD 操作做了封装并提供了一大批简化的 API。今天我们分别针对查询和插入这两大类数据操作给出了基于 JdbcTemplate 的实现方案，特别是针对插入场景，我们还引入了基于 JdbcTemplate 所构建的 SimpleJdbcInsert 简化这一操作。</p>
<p data-nodeid="3046">这里给你留一道思考题：在使用 JdbcTemplate 时，如果想要返回数据库的自增主键值有哪些实现方法？</p>
<p data-nodeid="3047" class="">在 Spring 中存在一组以 -Template 结尾的模板工具类，这些类都是模板方法这一设计模式的典型应用，同时还充分利用了回调机制完成解耦和扩展。在 08 讲中，我们将对 JdbcTemplate 的具体实现机制进行详细剖析。</p>

---

### 精选评论

##### **6086：
> 在同一个业务逻辑方法里有两次insert，应该启用事务，保证数据一致性。

##### **华：
> 表用用关键字order容易踩坑，实际开发中应避免

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，order 尽量少用，应该是 SQL 中的关键词，这里只是做个演示。

##### **仁：
> 本讲的jdbcTemplate算是mybatis的一种替代方案吗？jdbcTemplate 是不是也是对jdbc的一种封装？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 对的，JdbcTemplate 本质上就是对 JDBC 的封装，也可以算是一种半自动化的 ORM 技术，谈不上是对 Mybatis 的替代方案，只是 Spring 提供的一个工具类。

