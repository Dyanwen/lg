<p data-nodeid="1078" class="">课程正式开始了，这里我会以一个案例的形式来和你讲解如何通过 Spring Boot 结合 Spring Data JPA 快速启动一个项目、如何使用 UserRepository 完成对 User 表的操作、如何写测试用例等几个知识点，同时带你体验一下 Spring Data JPA 的优势。通过这个课时，希望你能够对 JPA 建立一个整体的认识。</p>
<blockquote data-nodeid="1079">
<p data-nodeid="1080">提示：在本课程中如果没有特殊说明，JPA 就是指 Spring Data JPA。</p>
</blockquote>
<p data-nodeid="1081">话不多说，我们先来看一个案例。</p>
<h3 data-nodeid="1082">Spring Boot 和 Spring Data JPA 的 Demo演示</h3>
<p data-nodeid="1083">我们利用 JPA + Spring Boot 简单做一个 RESTful API 接口，方便你来了解 Spring Data JPA 是干什么用的，具体步骤如下。</p>
<p data-nodeid="1084"><strong data-nodeid="1248">第一步：利用 IDEA 和 SpringBoot 2.3.3 快速创建一个案例项目。</strong></p>
<p data-nodeid="1085">点击“菜单” | New Project 命令，选择 Spring Initializr 选项，如下图所示。</p>
<p data-nodeid="1086"><img src="https://s0.lgstatic.com/i/image/M00/4E/AE/Ciqc1F9fAriAOBKCAAG7xhrHi_E023.png" alt="Drawing 0.png" data-nodeid="1254"></p>
<p data-nodeid="1087">这里我们利用&nbsp;Spring 官方的 Start 来创建一个项目，接下来<strong data-nodeid="1259">选择 Spring Boot 的依赖：</strong></p>
<ul data-nodeid="1088">
<li data-nodeid="1089">
<p data-nodeid="1090">Lombok：帮我们创建一个简单的 Entity 的 POJO，主要用来省去创建 GET 和 SET 方法；</p>
</li>
<li data-nodeid="1091">
<p data-nodeid="1092">Spring Web：MVC 必备组件；</p>
</li>
<li data-nodeid="1093">
<p data-nodeid="1094">Spring Data JPA：重头戏，这是本课时的重点内容；</p>
</li>
<li data-nodeid="1095">
<p data-nodeid="1096">H2 Database：内存数据库；</p>
</li>
<li data-nodeid="1097">
<p data-nodeid="1098">Spring Boot Actuator：监控我们项目状态使用。</p>
</li>
</ul>
<p data-nodeid="1099">然后通过下图操作界面选择上面提到的这些依赖，如下图所示：</p>
<p data-nodeid="1100"><img src="https://s0.lgstatic.com/i/image/M00/4E/AE/Ciqc1F9fAsKAbYHaAAMUVRZwyEY667.png" alt="Drawing 1.png" data-nodeid="1268"></p>
<p data-nodeid="1101"><strong data-nodeid="1272">第二步：通过 IDEA 的图形化界面，一路单击 Next 按钮，然后单击 Finsh 按钮，得到一个工程，完成后结构如下图所示：</strong></p>
<p data-nodeid="1102"><img src="https://s0.lgstatic.com/i/image/M00/4E/BA/CgqCHl9fAsmAbD28AAR6C6uyIWs848.png" alt="Drawing 2.png" data-nodeid="1275"></p>
<p data-nodeid="1103">现在我们已经可以创建一个 Spring Boot + JPA 的项目了，那么接下来我们看看怎么对 User 表进行增删改查操作。</p>
<p data-nodeid="1104"><strong data-nodeid="1280">第三步：新增 3 个类来完成对 User 的 CURD。</strong></p>
<p data-nodeid="1105">第一个类：新增 User.java，它是一个实体类，用来做 User 表的映射的，如下所示：</p>
<pre class="lang-java" data-nodeid="1106"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-keyword">import</span> javax.persistence.GeneratedValue;
<span class="hljs-keyword">import</span> javax.persistence.GenerationType;
<span class="hljs-keyword">import</span> javax.persistence.Id;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-keyword">private</span> String email;
}
</code></pre>
<p data-nodeid="1107">第二个类：新增 UserRepository，它是我们的 DAO 层，用来操作实体 User 进行增删改成操作，如下所示：</p>
<pre class="lang-java" data-nodeid="1108"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="1109">第三个类：新增 UserController，它是 Controller，用来创建 Rest 的 API 的接口的，如下所示：</p>
<pre class="lang-java" data-nodeid="1110"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Page;
<span class="hljs-keyword">import</span> org.springframework.data.domain.Pageable;
<span class="hljs-keyword">import</span> org.springframework.http.MediaType;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.*;
<span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(path = "/api/v1")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserController</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> UserRepository userRepository;
   <span class="hljs-comment">/**
    * 保存用户
    * <span class="hljs-doctag">@param</span> user
    * <span class="hljs-doctag">@return</span>
    */</span>
   <span class="hljs-meta">@PostMapping(path = "user",consumes = {MediaType.APPLICATION_JSON_VALUE})</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> User <span class="hljs-title">addNewUser</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> User user)</span> </span>{
      <span class="hljs-keyword">return</span> userRepository.save(user);
   }
   <span class="hljs-comment">/**
    * 根据分页信息查询用户
    * <span class="hljs-doctag">@param</span> request
    * <span class="hljs-doctag">@return</span>
    */</span>
   <span class="hljs-meta">@GetMapping(path = "users")</span>
   <span class="hljs-meta">@ResponseBody</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> Page&lt;User&gt; <span class="hljs-title">getAllUsers</span><span class="hljs-params">(Pageable request)</span> </span>{
      <span class="hljs-keyword">return</span> userRepository.findAll(request);
   }
}
</code></pre>
<p data-nodeid="1111">最终，我们的项目结构变成如下图所示的模样：</p>
<p data-nodeid="1112"><img src="https://s0.lgstatic.com/i/image/M00/4E/AE/Ciqc1F9fAt2AKc-oAALnw7dehT4454.png" alt="Drawing 3.png" data-nodeid="1287"></p>
<p data-nodeid="1113">上图中，appliaction.properties 里面的内容是空的，到现在三步搞定，其他什么都不需要配置，直接点击 JpaApplication 这个类，就可启动我们的项目了。</p>
<p data-nodeid="1114"><strong data-nodeid="1292">第四步：调用项目里面的 User 相关的 API 接口测试一下。</strong></p>
<p data-nodeid="1115">我们再新增一个 JpaApplication.http 文件，内容如下：</p>
<pre class="lang-java" data-nodeid="1116"><code data-language="java">POST /api/v1/user HTTP/1.1
Host: 127.0.0.1:8080
Content-Type: application/json
Cache-Control: no-cache
{"name":"jack","email":"123@126.com"}
#######
GET http://127.0.0.1:8080/api/v1/users?size=3&amp;page=0
###
</code></pre>
<p data-nodeid="1117">点击“运行”按钮，效果如下图所示：</p>
<p data-nodeid="1118"><img src="https://s0.lgstatic.com/i/image/M00/4E/AF/Ciqc1F9fAueAaffMAAEEcHlhpSo686.png" alt="Drawing 4.png" data-nodeid="1297"></p>
<p data-nodeid="1119">运行结果如下：</p>
<pre class="lang-java" data-nodeid="1120"><code data-language="java">POST http:<span class="hljs-comment">//127.0.0.1:8080/api/v1/user</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sat, <span class="hljs-number">22</span> Aug <span class="hljs-number">2020</span> <span class="hljs-number">02</span>:<span class="hljs-number">48</span>:<span class="hljs-number">43</span> GMT
Keep-Alive: timeout=<span class="hljs-number">60</span>
Connection: keep-alive
{
  <span class="hljs-string">"id"</span>: <span class="hljs-number">4</span>,
  <span class="hljs-string">"name"</span>: <span class="hljs-string">"jack"</span>,
  <span class="hljs-string">"email"</span>: <span class="hljs-string">"123@126.com"</span>
}
Response code: <span class="hljs-number">200</span>; Time: <span class="hljs-number">30</span>ms; Content length: <span class="hljs-number">44</span> bytes
GET http:<span class="hljs-comment">//127.0.0.1:8080/api/v1/users?size=3&amp;page=0</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span> 
Content-Type: application/json
Transfer-Encoding: chunked
Date: Sat, <span class="hljs-number">22</span> Aug <span class="hljs-number">2020</span> <span class="hljs-number">02</span>:<span class="hljs-number">50</span>:<span class="hljs-number">20</span> GMT
Keep-Alive: timeout=<span class="hljs-number">60</span>
Connection: keep-alive
{
  <span class="hljs-string">"content"</span>: [
    {
      <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
      <span class="hljs-string">"name"</span>: <span class="hljs-string">"jack"</span>,
      <span class="hljs-string">"email"</span>: <span class="hljs-string">"123@126.com"</span>
    },
    {
      <span class="hljs-string">"id"</span>: <span class="hljs-number">2</span>,
      <span class="hljs-string">"name"</span>: <span class="hljs-string">"jack"</span>,
      <span class="hljs-string">"email"</span>: <span class="hljs-string">"123@126.com"</span>
    },
    {
      <span class="hljs-string">"id"</span>: <span class="hljs-number">3</span>,
      <span class="hljs-string">"name"</span>: <span class="hljs-string">"jack"</span>,
      <span class="hljs-string">"email"</span>: <span class="hljs-string">"123@126.com"</span>
    }
  ],
  <span class="hljs-string">"pageable"</span>: {
    <span class="hljs-string">"sort"</span>: {
      <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">false</span>,
      <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">true</span>,
      <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">true</span>
    },
    <span class="hljs-string">"offset"</span>: <span class="hljs-number">0</span>,
    <span class="hljs-string">"pageNumber"</span>: <span class="hljs-number">0</span>,
    <span class="hljs-string">"pageSize"</span>: <span class="hljs-number">3</span>,
    <span class="hljs-string">"unpaged"</span>: <span class="hljs-keyword">false</span>,
    <span class="hljs-string">"paged"</span>: <span class="hljs-keyword">true</span>
  },
  <span class="hljs-string">"totalPages"</span>: <span class="hljs-number">2</span>,
  <span class="hljs-string">"last"</span>: <span class="hljs-keyword">false</span>,
  <span class="hljs-string">"totalElements"</span>: <span class="hljs-number">4</span>,
  <span class="hljs-string">"size"</span>: <span class="hljs-number">3</span>,
  <span class="hljs-string">"number"</span>: <span class="hljs-number">0</span>,
  <span class="hljs-string">"numberOfElements"</span>: <span class="hljs-number">3</span>,
  <span class="hljs-string">"sort"</span>: {
    <span class="hljs-string">"sorted"</span>: <span class="hljs-keyword">false</span>,
    <span class="hljs-string">"unsorted"</span>: <span class="hljs-keyword">true</span>,
    <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">true</span>
  },
  <span class="hljs-string">"first"</span>: <span class="hljs-keyword">true</span>,
  <span class="hljs-string">"empty"</span>: <span class="hljs-keyword">false</span>
}
Response code: <span class="hljs-number">200</span>; Time: <span class="hljs-number">59</span>ms; Content length: <span class="hljs-number">449</span> bytes
</code></pre>
<p data-nodeid="1437" class="te-preview-highlight"><strong data-nodeid="1442">关于 IDEA 运行 Gradle 项目的小技巧：</strong> 在实际工作中，我们启动运行或者是跑测试用例的时候，经常以 Gradle 的方式运行，或者用 Application 的方式运行，两种方式进行随意切换，需要设置的地方如下图所示：</p>

<p data-nodeid="1122"><img src="https://s0.lgstatic.com/i/image/M00/4E/AF/Ciqc1F9fAvaAJ7MtAANtS5uq_NY272.png" alt="Drawing 5.png" data-nodeid="1306"></p>
<p data-nodeid="1123">通过以上案例，我们知道了 Spring Data JPA 可以帮我们做数据的 CRUD 操作，掌握到了JPA + Spring Boot 如何启动和集成 JPA，以及对如何创建一个数据库操作也有了一定的了解。那么接下来，我们看下是如何切换默认数据源的，如切换成 MySQL。</p>
<h3 data-nodeid="1124">JPA 如何整合 MySQL 数据库？</h3>
<p data-nodeid="1125">关于 JPA 与 MySQL 的集成我们分为两部分来展开：如何切换 MySQL 数据源、如何写测试用例进行测试。</p>
<h4 data-nodeid="1126">1. 切换 MySQL 数据源</h4>
<p data-nodeid="1127">上面的例子，我们采用的是默认 H2 数据源的方式，这个时候你肯定会问：“H2 重启，数据丢失了怎么办？”那么我们调整一下上面的代码，以 MySQL 作为数据源，看看需要改动哪些。</p>
<p data-nodeid="1128">第一处改动，application.properties 内容如下：</p>
<pre class="lang-java" data-nodeid="1129"><code data-language="java">spring.datasource.url=jdbc:mysql:<span class="hljs-comment">//localhost:3306/test</span>
spring.datasource.username=root
spring.datasource.password=root
spring.jpa.generate-ddl=<span class="hljs-keyword">true</span>
</code></pre>
<p data-nodeid="1130">第二处改动，删除 H2 数据源，新增 MySQL 数据库驱动：</p>
<p data-nodeid="1131"><img src="https://s0.lgstatic.com/i/image/M00/4E/BA/CgqCHl9fAyiAPQf3AADCrCOm25k666.png" alt="Drawing 6.png" data-nodeid="1318"></p>
<p data-nodeid="1132">调整完毕之后，我们重启这个项目，以同样的方式测试上面的两个接口依然 OK。</p>
<p data-nodeid="1133"><img src="https://s0.lgstatic.com/i/image/M00/4E/BA/CgqCHl9fAy2AL0EuAADDLkFVehk043.png" alt="Drawing 7.png" data-nodeid="1322"></p>
<p data-nodeid="1134">其实这个时候可以发现一件事情，那就是我没有手动去创建任何表，而 JPA 自动帮我创建了数据库的 DDL，并新增了 User 表，所以当我们用 JPA 之后创建表的工作就不会那么复杂了，我们只需要把实体写好就可以了。</p>
<p data-nodeid="1135">以上是切换 MySQL 数据源需要进行的操作，接下来看看测试用例怎么写，在修改代码的时候我们就不需要频繁重启项目了，当你掌握 JUnit 之后，可以提升开发效率。</p>
<h4 data-nodeid="1136">2. Spring Data JPA 测试用例的写法</h4>
<p data-nodeid="1137">我们这里只关注 Repository 的测试用例的写法，Controller 和 Service 等更复杂的测试我们在测试课时再详细介绍，这里我们先快速体验一下。</p>
<p data-nodeid="1138">第一步，在 Test 目录里增加 UserRepositoryTest 类：</p>
<pre class="lang-java" data-nodeid="1139"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.junit.Assert;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRepositoryTest</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository userRepository;
    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testSaveUser</span><span class="hljs-params">()</span> </span>{
        User user = userRepository.save(User.builder().name(<span class="hljs-string">"jackxx"</span>).email(<span class="hljs-string">"123456@126.com"</span>).build());
        Assert.assertNotNull(user);
        List&lt;User&gt; users= userRepository.findAll();
        System.out.println(users);
        Assert.assertNotNull(users);
    }
}
</code></pre>
<p data-nodeid="1140">第二步，我们可直接运行测试用例，进行真实的 DB 操作，通过控制台来看下我们的测试用例是否能够跑通，如下图所示：</p>
<p data-nodeid="1141"><img src="https://s0.lgstatic.com/i/image/M00/4E/BA/CgqCHl9fAzqAKH25AADMO537plI734.png" alt="Drawing 8.png" data-nodeid="1333"></p>
<p data-nodeid="1142">通过上图可以看到，测试的时候执行的 SQL 有哪些，那么我们到底是连接的 MySQL 做的测试用例，还是连接的 H2 做的测试呢？在后面的第 30 课时（单元测试和集成测试）的时候我会为你详细揭晓，到时你会发现测试用例写起来也是如此简单。</p>
<h3 data-nodeid="1143">整体认识 JPA</h3>
<p data-nodeid="1144">通过上面的两个例子我们已经快速入门了，知道了 Spring Boot 结合 Spring Data JPA 怎么配置和启动一个项目 ，之后当你熟悉了 JPA 之后，你还会发现 Spring Boot JPA 要比我们配置 MyBatis 简单很多。下面我们来整体认识一下 Java Persistence API 究竟是什么。</p>
<p data-nodeid="1145">介绍 JPA 协议之前，我们先来对比了解下市面上的 ORM 框架有哪些，分别有哪些优缺点，做到心里有数。</p>
<h4 data-nodeid="1146">1. 市场上 ORM 框架比对</h4>
<p data-nodeid="1147">下表是市场上比较流行的 ORM 框架，这里我罗列了 MyBatis、Hibernate、Spring Data JPA 等，并对比了下它们的优缺点。</p>
<p data-nodeid="1148"><img src="https://s0.lgstatic.com/i/image/M00/4E/B1/Ciqc1F9fBfeANrGuAAOa8Y2E5fU233.png" alt="2.png" data-nodeid="1344"></p>
<p data-nodeid="1149">经过对比，你可以看到我们正在学习的 Spring Data JPA 还是比较前卫的，很受欢迎，继承了 Hibernate 的很多优点，上手又比较简单。所以，我非常建议你好好学习一下。</p>
<h4 data-nodeid="1150">2. Java Persistence API 介绍和开源实现</h4>
<p data-nodeid="1151">JPA 是 JDK 5.0 新增的协议，通过相关持久层注解（@Entity 里面的各种注解）来描述对象和关系型数据里面的表映射关系，并将 Java 项目运行期的实体对象，通过一种Session持久化到数据库中。</p>
<p data-nodeid="1152">想象一下，一旦协议有了，大家都遵守了此种协议进行开发，那么周边开源产品就会大量出现，比如我们在后面要介绍的第 29 课时（Spring Data Rest 是什么？和 JPA 是什么关系？）就可以基于这套标准，进而对 Entity 的操作再进行封装，从而可以得到更加全面的 Rest 协议的 API接口。</p>
<p data-nodeid="1153">再比如 JSON API（<a href="https://jsonapi.org/" data-nodeid="1354">https://jsonapi.org/</a>）协议，就是雅虎的大牛基于 JPA 的协议的基础，封装制定的一套 RESTful 风格、JSON 格式的 API 协议，那么一旦 JSON API 协议成了标准，就会有很多周边开源产品出现。比如很多 JSON API 的客户端、现在比较流行的 Ember 前端框架，就是基于 Entity 这套 JPA 服务端的协议，在前端解析 API 协议，从而可以对普通 JSON 和 JSON API 的操作进行再封装。</p>
<p data-nodeid="1154">所以规范是一件很有意思的事情，突然之间世界大变样，很多东西都一样了，我们的思路就需要转换了。</p>
<p data-nodeid="1155"><strong data-nodeid="1360">JPA 的内容分类</strong></p>
<ul data-nodeid="1156">
<li data-nodeid="1157">
<p data-nodeid="1158">一套 API 标准定义了一套接口，在 javax.persistence 的包下面，用来操作实体对象，执行 CRUD 操作，而实现的框架（Hibernate）替代我们完成所有的事情，让开发者从烦琐的 JDBC 和 SQL 代码中解脱出来，更加聚焦自己的业务代码，并且使架构师架构出来的代码更加可控。</p>
</li>
<li data-nodeid="1159">
<p data-nodeid="1160">定义了一套基于对象的 SQL：Java Persistence Query Language（JPQL），像 Hibernate 一样，我们通过写面向对象（JPQL）而非面向数据库的查询语言（SQL）查询数据，避免了程序与数据库 SQL 语句耦合严重，比较适合跨数据源的场景（一会儿 MySQL，一会儿 Oracle 等）。</p>
</li>
<li data-nodeid="1161">
<p data-nodeid="1162">ORM（Object/Relational Metadata）对象注解映射关系，JPA 直接通过注解的方式来表示 Java 的实体对象及元数据对象和数据表之间的映射关系，框架将实体对象与 Session 进行关联，通过操作 Session 中不通实体的状态，从而实现数据库的操作，并实现持久化到数据库表中的操作，与 DB 实现同步。</p>
</li>
</ul>
<p data-nodeid="1163">详细的协议内容你感兴趣的话也可以看一下<a href="https://github.com/eclipse-ee4j/jpa-api" data-nodeid="1367">官方的文档</a>。</p>
<p data-nodeid="1164"><strong data-nodeid="1372">JPA 的开源实现</strong></p>
<p data-nodeid="1165">JPA 的宗旨是为 POJO 提供持久化标准规范，可以集成在 Spring 的全家桶使用，也可以直接写独立 application 使用，任何用到 DB 操作的场景，都可以使用，极大地方便开发和测试，所以 JPA 的理念已经深入人心了。Spring Data JPA、Hibernate 3.2+、TopLink 10.1.3 以及 OpenJPA、QueryDSL 都是实现 JPA 协议的框架，他们之间的关系结构如下图所示：</p>
<p data-nodeid="1166"><img src="https://s0.lgstatic.com/i/image/M00/4E/AF/Ciqc1F9fA1uARUnvAAB2ZNS1UXc485.png" alt="Drawing 9.png" data-nodeid="1376"></p>
<p data-nodeid="1167">俗话说得好：“未来已经来临，只是尚未流行”，大神资深开发用 Spring Data JPA，编程极客者用 JPA；而普通 Java 开发者，不想去挑战的 Java“搬砖者”用 Mybatis。</p>
<p data-nodeid="1168">到这里，相信你已经对 JPA 有了一定的认识，接下来我们了解一下 Spring Data，看看它都有哪些子项目。</p>
<h3 data-nodeid="1169">Spring Data 认识</h3>
<h4 data-nodeid="1170">1. Spring Data 介绍</h4>
<p data-nodeid="1171">Spring Data 项目是从 2010 年开发发展起来的，Spring Data 利用一个大家熟悉的、一致的、基于“注解”的数据访问编程模型，做一些公共操作的封装，它可以轻松地让开发者使用数据库访问技术，包括关系数据库、非关系数据库（NoSQL）。同时又有不同的数据框架的实现，保留了每个底层数据存储结构的特殊特性。</p>
<p data-nodeid="1172">Spring Data Common 是 Spring Data 所有模块的公共部分，该项目提供了基于 Spring 的共享基础设施，它提供了基于 repository 接口以 DB 操作的一些封装，以及一个坚持在 Java 实体类上标注元数据的模型。</p>
<p data-nodeid="1173" class="">Spring Data 不仅对传统的数据库访问技术如 JDBC、Hibernate、JDO、TopLick、JPA、MyBatis 做了很好的支持和扩展、抽象、提供方便的操作方法，还对 MongoDb、KeyValue、Redis、LDAP、Cassandra 等非关系数据的 NoSQL 做了不同的实现版本，方便我们开发者触类旁通。</p>
<p data-nodeid="1174">其实这种接口型的编程模式可以让我们很好地学习 Java 的封装思想，实现对操作的进一步抽象，我们也可以把这种思想运用在自己公司写的 Framework 上面。</p>
<p data-nodeid="1175">下面来看一下 Spring Data 的子项目都有哪些。</p>
<h4 data-nodeid="1176">2. Spring Data 的子项目有哪些</h4>
<p data-nodeid="1177">下图为目前 Spring Data 的框架分类结构图，里面都有哪些模块可以一目了然，也可以知道哪些是我们需要关心的项目。</p>
<p data-nodeid="1178"><img src="https://s0.lgstatic.com/i/image/M00/4E/BA/CgqCHl9fA2iAJZruAAEOKPj_-ZU042.png" alt="Drawing 11.png" data-nodeid="1394"></p>
<p data-nodeid="1179">主要项目（Main Modules）：</p>
<ul data-nodeid="1180">
<li data-nodeid="1181">
<p data-nodeid="1182">Spring Data Commons，相当于定义了一套抽象的接口，下一课时我们会具体介绍</p>
</li>
<li data-nodeid="1183">
<p data-nodeid="1184">Spring Data Gemfire</p>
</li>
<li data-nodeid="1185">
<p data-nodeid="1186">Spring Data JPA，我们关注的重点，对 Spring Data Common 的接口的 JPA 协议的实现</p>
</li>
<li data-nodeid="1187">
<p data-nodeid="1188">Spring Data KeyValue</p>
</li>
<li data-nodeid="1189">
<p data-nodeid="1190">Spring Data LDAP</p>
</li>
<li data-nodeid="1191">
<p data-nodeid="1192">Spring Data MongoDB</p>
</li>
<li data-nodeid="1193">
<p data-nodeid="1194">Spring Data REST</p>
</li>
<li data-nodeid="1195">
<p data-nodeid="1196">Spring Data Redis</p>
</li>
<li data-nodeid="1197">
<p data-nodeid="1198">Spring Data for Apache Cassandra</p>
</li>
<li data-nodeid="1199">
<p data-nodeid="1200">Spring Data for Apache Solr</p>
</li>
</ul>
<p data-nodeid="1201">社区支持的项目（Community Modules）：</p>
<ul data-nodeid="1202">
<li data-nodeid="1203">
<p data-nodeid="1204">Spring Data Aerospike</p>
</li>
<li data-nodeid="1205">
<p data-nodeid="1206">Spring Data Couchbase</p>
</li>
<li data-nodeid="1207">
<p data-nodeid="1208">Spring Data DynamoDB</p>
</li>
<li data-nodeid="1209">
<p data-nodeid="1210">Spring Data Elasticsearch</p>
</li>
<li data-nodeid="1211">
<p data-nodeid="1212">Spring Data Hazelcast</p>
</li>
<li data-nodeid="1213">
<p data-nodeid="1214">Spring Data Jest</p>
</li>
<li data-nodeid="1215">
<p data-nodeid="1216">Spring Data Neo4j</p>
</li>
<li data-nodeid="1217">
<p data-nodeid="1218">Spring Data Vault</p>
</li>
</ul>
<p data-nodeid="1219">其他（Related Modules）：</p>
<ul data-nodeid="1220">
<li data-nodeid="1221">
<p data-nodeid="1222">Spring Data JDBC Extensions</p>
</li>
<li data-nodeid="1223">
<p data-nodeid="1224">Spring for Apache Hadoop</p>
</li>
<li data-nodeid="1225">
<p data-nodeid="1226">Spring Content</p>
</li>
</ul>
<p data-nodeid="1227">关于 Spring Data 的子项目，除了上面这些，还有很多开源社区版本，比如 Spring Data、MyBatis 等，这里就不一一介绍了，感兴趣的同学可以到 Spring 社区，或者 GitHub 上进行查阅。</p>
<h3 data-nodeid="1228">总结</h3>
<p data-nodeid="1229">本课时的主要目的是带领你快速入门，从 H2 数据源和 Mysql 数据源两个方面为你介绍了 Spring Data JPA 数据操作的概念，了解了 Repository 的写法，快速体验一把 Spring Data JPA 的便捷之处。</p>
<p data-nodeid="1230">希望你通过本节课的学习可以对 Spring Data JPA + Spring Boot 有一个整体的认识，为以后的技术进阶打下良好的基础。在掌握了基本知识以后，你会发现 Spring Data JPA 是 ORM 的效率利器，后面课程我会一一揭开 Spring Data JPA 的神秘面纱，带你掌握其实现原理和实战经验，让你在实际开发中游刃有余。</p>
<p data-nodeid="1231">对于本课时所讲的知识点，欢迎你在下方留言区表达自己的学习感悟，大家一起讨论，共同进步。</p>
<blockquote data-nodeid="1232">
<p data-nodeid="1233">补充一个TIPS：课程中的案例是依赖 lombok 插件的，如下图所示：</p>
</blockquote>
<p data-nodeid="1234"><img src="https://s0.lgstatic.com/i/image/M00/4F/DD/CgqCHl9hfQKAEXs0AABb1DeHmt4598.png" alt="image (3).png" data-nodeid="1427"></p>
<blockquote data-nodeid="1235">
<p data-nodeid="1236">并开启 annotation processing。</p>
</blockquote>
<p data-nodeid="1237"><img src="https://s0.lgstatic.com/i/image/M00/4F/D2/Ciqc1F9hfQmAFGFzAACj394zaUc225.png" alt="image (4).png" data-nodeid="1431"></p>
<blockquote data-nodeid="1238">
<p data-nodeid="1239" class="">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="1436">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### TenderLight：
> 学习过程中踩坑如下1、连接的 H2 做的测试用例2、JpaApplication.http 文件POST /api/v1/user HTTP/1.1Host: 127.0.0.1:8080Content-Type: application/jsonCache-Control: no-cache//这个地方加个空行{"name":"jack","email":"123@126.com"}#######GET http://127.0.0.1:8080/api/v1/users?size=3page=0###

##### *兴：
> 各位如果出现了问题does not conform to RFC 7230 and has been ignored请看解决链接：https://blog.csdn.net/Hosea_star/article/details/108589768

##### TenderLight：
> 补充，如果用my sql连接做测试用例：@DataJpaTest@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)public class UserRepositoryTest {

##### **安：
> 加油，一定要学精，学透。

##### **慈：
> 之前还真看过jpa，但是自己研究是真的费劲

##### **民：
> 这个声音好听，读的好，是听过最好的

##### **军：
> 之前用了spring jpa 有多重要，看这个课程，发现还是需要深入理解和重新认识一下.

##### **艺：
> 文章里面贴的代码都是 h2 相关的，没有数据库密码什么的跑的挺好。后面换到 mysql 大多数人应该都是自己有数据库的，需要配置密码什么的这个时候就会遇到问题了。老师最后都贴上了源码，看了一下还是有一些问题没有覆盖到的1. 测试的时候配置的数据库参数会被覆盖掉，这个卡了我一下，需要在测试类上加注解不覆盖配置。@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)2. 使用 mysql 的时候实体类是要加上无参构造全参构造才行的。3. 前面实体类贴的代码没有加上 builder 注解，但是测试类就用上了 builder，这个不熟悉的朋友可能就会疑惑了。实体类加上 lombok 的">@Builder注解就好

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 有道理，不错

##### *霸：
> 老师我想问一下您的boot是几版本，为什么我的idea自己创建完项目后，运行.http测试文件，tomcat会报错，7230，百度说boot与内嵌的版本产生了冲突

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; id 'org.springframework.boot' version '2.3.3.RELEASE'，

##### *文：
> 执行测试的时候应该需要把H2数据库的依赖再加回来吧

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不需要，测试用例可以单独配置自己的依赖，看老师的源码吧，gradle里面有设置
https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa

##### Tracy：
> 学习真得边学边动手边总结

##### **栋：
> 老师讲的真好，我要努力了！！！

##### **博：
> 老师 为什么不用maven构建项目？现在maven过时了吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没过时，个人喜好问题，感觉gradle灵活性更高。

##### **街牛牛：
> 不知道稳重转换mysql数据源用的什么版本的包 我用的5.xx的需要指定驱动类spring.datasource.driver-class-name=com.mysql.jdbc.Driver

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当不知道用什么版本的时候，类似这样：runtimeOnly('mysql:mysql-connector-java')，版本号交由spring boot决定

##### **9720：
> 用mybatis就是搬砖的了么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个问题不是绝对的，需要自己悟啊

##### **用户1643：
> 这个模式还是可以的，有视频有音频还有文字合理大大解决了学时间碎片化的问题

##### **鑫：
> jpa可以理解为：自动通过实体和注解，生成表

##### **华：
> 学成以后去装比

##### **彬：
> User.builder()，这个报错，没有这个方法，这是什么问题

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; lombok插件

##### *刚：
> 我grade 用不了，">public void testSaveUser() {    User user = userRepository.save(User.builder().username("jackxx").password("123456@126.com").build());    Assert.assertNotNull(user);    List">userRepository.findAll();    System.out.println(users);    Assert.assertNotNull(users);}user没有build方法，现在报错的了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 看一下gradle的版本，用./gradlew

##### **的太阳：
> 怎样配置日志输出？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; spring.jpa.show-sql=true，后面章节还会详细介绍

##### *霸：
> 删除h2数据源，修改mysql驱动在哪修改

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; build.gradle

##### **东：
> 最终测试打印信息一样但是mysql里边没有数据，根据结果我猜用的是H2而不是mysql数据库，不知道这么说对不对？

##### **宣：
> 不好意思老师，请问User.builder()是啥问题 ： 请求体上面还有一行空行

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; lombok的builder语法方便构建对象

##### **0214：
> 以前也用过hibernate，但很久不用了，希望通过这次学习彻底掌握jpa。

##### *超：
> post请求后，报7230错误，请问怎么解决

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 要看日志的，删除从新创建，或者取老师github上面的例子看

##### **军：
> 之前刚入行半年的时候也是从mybatis转JPA，从一开始非常懵逼到真香。希望能借此学习到更多！

##### 韩：
> 表示这个课程还附带文档了，赞

##### **源：
> 正好公司一个项目用了它，正好学习一下

##### **5382：
> 根本跑不了吧，spring boot 2.3.3内置了9.0tomcat，高版本tomcat添加了http头的验证，跑http文件根本跑不了，json串有非限定的字符

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 看不如试，试一下就知道了

##### **亮：
> 讲的很棒，今天下班时就在想怎么样才能成为一名高手，一直感觉不得其法，希望听完课后可以有一些启发

##### *兴：
> 刚好新入职的公司技术用到了spring data，正好学习一下

##### **新：
> 整合MySQL时需要提前先创建库吗？我整合时没有提前创建库，启动失败了。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; spring.jpa.generate-ddl=true，spring.jpa.hibernate.ddl-auto=create配置这个让JPA自己创建

##### *周：
> 老师讲的很好很详细，好评

##### *稳：
> 请问张老师有什么办法让gradle下载得更快一些编译得更快一些吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 用本地下载好的

##### **祥：
> 请教老师:和mybstis plus比较起来有什么优势之处呢

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 课程有讲哦

##### **一定进：
> 最近刚好在学习spring boot，正好围观、

