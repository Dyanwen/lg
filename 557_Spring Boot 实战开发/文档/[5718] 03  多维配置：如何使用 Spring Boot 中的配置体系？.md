<p data-nodeid="203028">配置体系是基于 Spring Boot 框架开发应用程序的基础，而自动配置也是该框架的核心功能之一。今天我将带领大家梳理使用 Spring Boot 配置体系的系统方法。我们先从创建和运行第一个 Web 应用程序开始吧。</p>
<h3 data-nodeid="203029">创建第一个 Spring Boot Web 应用程序</h3>
<p data-nodeid="203030">基于 Spring Boot 创建 Web 应用程序的方法有很多，但最简单、最直接的方法是使用 Spring 官方提供的 Spring Initializer 初始化模板。</p>
<p data-nodeid="203914">初始化使用操作：直接访问 Spring Initializer 网站（<a href="http://start.spring.io/" data-nodeid="203919">http://start.spring.io/</a>），选择创建一个 Maven 项目并指定相应的 Group 和 Artifact，然后在添加的依赖中选择 Spring Web，点击生成即可。界面效果下图所示：</p>
<p data-nodeid="203915" class=""><img src="https://s0.lgstatic.com/i/image/M00/71/0D/Ciqc1F-83-KAYTQAAADMrUH44hQ767.png" alt="Drawing 0.png" data-nodeid="203923"></p>


<div data-nodeid="204178" class=""><p style="text-align:center">使用 Spring Initializer 创建 Web 应用程序示意图</p></div>

<p data-nodeid="203034">当然，对于有一定开发经验的同学而言，我们完全可以基于 Maven 本身的功能特性和结构，来生成上图中的代码工程。</p>
<p data-nodeid="203035">接下来，我们参考 02 讲中关于 Controller 的创建基本方法，来为这个代码工程添加一些支持 RESTful 风格的 HTTP 端点，在这里我们同样创建一个 CustomerController 类，如下所示：</p>
<pre class="lang-java" data-nodeid="203036"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping(value="customers")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CustomerController</span> </span>{
&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@RequestMapping(value = "/{id}", method = RequestMethod.GET)</span>
&nbsp;&nbsp;&nbsp; &nbsp;<span class="hljs-function"><span class="hljs-keyword">public</span> CustomerTicket <span class="hljs-title">getCustomerTicketById</span><span class="hljs-params">(<span class="hljs-meta">@PathVariable</span> Long id)</span> </span>{&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; CustomerTicket customerTicket = <span class="hljs-keyword">new</span> CustomerTicket();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setId(<span class="hljs-number">1L</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setAccountId(<span class="hljs-number">100L</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setOrderNumber(<span class="hljs-string">"Order00001"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setDescription(<span class="hljs-string">"DemoOrder"</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; customerTicket.setCreateTime(<span class="hljs-keyword">new</span> Date());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 
&nbsp;&nbsp;&nbsp;  &nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> customerTicket;
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="203037">请注意，这里是为了演示方便，我们才使用了硬编码完成了一个 HTTP GET 请求的响应处理。</p>
<p data-nodeid="203038">现在 RESTful 端点已经开发完成，我们需要对这个应用程序进行打包。基于 Spring Boot 和 Maven，当我们使用 mvn package 命令构建整个应用程序时，将得到一个 customerservice-0.0.1-SNAPSHOT.jar 文件，而这个 jar 文件就是可以直接运行的可执行文件，内置了 Tomcat Web 服务器。也就是说，我们可以通过如下命令直接运行这个 Spring Boot 应用程序：</p>
<pre class="lang-xml" data-nodeid="203039"><code data-language="xml">java –jar customerservice-0.0.1-SNAPSHOT.jar
</code></pre>
<p data-nodeid="203040">那么，如何验证服务是否启动成功，以及 HTTP 请求是否得到正确响应呢？在 03 讲中，我们引入 Postman 来演示如何通过 HTTP 协议暴露的端点进行远程服务访问。</p>
<p data-nodeid="203041">Postman 提供了强大的 Web API 和 HTTP 请求调试功能，界面简洁明晰，操作也比较方便快捷和人性化。Postman 能够发送任何类型的 HTTP 请求（如 GET、HEAD、POST、PUT 等），并能附带任何数量的参数和 HTTP 请求头（Header）。</p>
<p data-nodeid="204681">这时我们通过 Postman 访问“<a href="http://localhost:8083/customers/1" data-nodeid="204686">http://localhost:8083/customers/1</a>”端点，可以得到如下图所示的HTTP响应结果，说明整个服务已经启动成功。</p>
<p data-nodeid="204682" class=""><img src="https://s0.lgstatic.com/i/image/M00/71/0D/Ciqc1F-83_CAMxyvAABNmxMQRXc212.png" alt="Drawing 1.png" data-nodeid="204690"></p>



<p data-nodeid="203044">好了，现在我们已经明白如何构建、打包以及运行一个简单的 Web 应用程序了，这是一切开发工作的起点，后续所有的案例代码我们都将通过这种方式展现在你面前，包括接下来要介绍的 Spring Boot 配置体系也是一样。</p>
<h3 data-nodeid="203045">Spring Boot 中的配置体系</h3>
<p data-nodeid="203046">在 Spring Boot 中，其核心设计理念是对配置信息的管理采用约定优于配置。在这一理念下，则意味着开发人员所需要设置的配置信息数量比使用传统 Spring 框架时还大大减少。当然，今天我们关注的主要是如何理解并使用 Spring Boot 中的配置信息组织方式，这里就需要引出一个核心的概念，即 Profile。</p>
<h4 data-nodeid="203047">配置文件与 Profile</h4>
<p data-nodeid="203048">Profile 本质上代表一种用于组织配置信息的维度，在不同场景下可以代表不同的含义。例如，如果 Profile 代表的是一种状态，我们可以使用 open、halfopen、close 等值来分别代表全开、半开和关闭等。再比如系统需要设置一系列的模板，每个模板中保存着一系列配置项，那么也可以针对这些模板分别创建 Profile。这里的状态或模版的定义完全由开发人员自主设计，我们可以根据需要自定义各种 Profile，这就是 Profile 的基本含义。</p>
<p data-nodeid="203049">另一方面，为了达到集中化管理的目的，Spring Boot 对配置文件的命名也做了一定的约定，分别使用 label 和 profile 概念来指定配置信息的版本以及运行环境，其中 label 表示配置版本控制信息，而 profile 则用来指定该配置文件所对应的环境。在 Spring Boot 中，配置文件同时支持 .properties 和 .yml 两种文件格式，结合 label 和 profile 概念，如下所示的配置文件命名都是常见和合法的：</p>
<pre class="lang-xml" data-nodeid="203050"><code data-language="xml">/{application}.yml
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
</code></pre>
<p data-nodeid="203051">Yaml 的语法和其他高级语言类似，并且可以非常直观地表达各种列表、清单、标量等数据形态，特别适合用来表达或编辑数据结构和各种配置文件。在这里，我们指定了如下所示的数据源配置，这里使用了 . yml 文件，如下所示：</p>
<pre class="lang-xml" data-nodeid="203052"><code data-language="xml">spring: 
&nbsp; datasource:
&nbsp;&nbsp;&nbsp; driver-class-name: com.mysql.cj.jdbc.Driver
&nbsp;&nbsp;&nbsp; url: jdbc:mysql://127.0.0.1:3306/account
&nbsp;&nbsp;&nbsp; username: root
	password: root
</code></pre>
<p data-nodeid="203053">如果采用 .propertie 配置文件，那么上述配置信息将表示为如下的形式：</p>
<pre class="lang-xml" data-nodeid="203054"><code data-language="xml">spring.datasource.driverClassName=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/account
spring.datasource.username=root 
spring.datasource.password=root
</code></pre>
<p data-nodeid="205692">显然，类似这样的数据源通常会根据环境的不同而存在很多套配置。假设我们存在如下所示的配置文件集合：</p>
<p data-nodeid="205693" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/71/19/CgqCHl-83_2AOKfAAAAKdvXQRn8485.png" alt="Drawing 2.png" data-nodeid="205698"></p>
<div data-nodeid="205694"><p style="text-align:center">多配置文件示意图</p></div>







<p data-nodeid="203058">注意，这里有一个全局的 application.yml 配置文件以及多个局部的 profile 配置文件。那么，如何指定当前所使用的那一套配置信息呢？在 Spring Boot 中，我们可以在主 application.properties 中使用如下的配置方式来激活当前所使用的 Profile：</p>
<pre class="lang-xml" data-nodeid="203059"><code data-language="xml">spring.profiles.active = test
</code></pre>
<p data-nodeid="203060">上述配置项意味着系统当前会读取 application-test.yml 配置文件中的配置内容。同样，如果使用 .yml 文件，则可以使用如下所示的配置方法：</p>
<pre class="lang-xml" data-nodeid="203061"><code data-language="xml">spring:
&nbsp; profiles:
&nbsp;&nbsp;&nbsp; active: test
</code></pre>
<p data-nodeid="203062">事实上，我们也可以同时激活几个 Profile，这完全取决于你对系统配置的需求和维度：</p>
<pre class="lang-xml" data-nodeid="203063"><code data-language="xml">spring.profiles.active: prod, myprofile1, myprofile2
</code></pre>
<p data-nodeid="203064">当然，如果你想把所有的 Profile 配置信息只保存在一个文件中而不是分散在多个配置文件中， Spring Boot 也是支持的，需要做的事情只是对这些信息按 Profile 进行组织、分段，如下所示：</p>
<pre class="lang-xml" data-nodeid="203065"><code data-language="xml">spring: 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; profiles: test
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; #test 环境相关配置信息
&nbsp;
spring: 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; profiles: prod
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; #prod 环境相关配置信息
</code></pre>
<p data-nodeid="203066">尽管上述方法是有效的，但在 03 讲中，还是推荐你按多个配置文件的组织方法管理各个 Profile 配置信息，这样才不容易混淆和出错。</p>
<p data-nodeid="203067">最后，如果我们不希望在全局配置文件中指定所需要激活的 Profile，而是想把这个过程延迟到运行这个服务时，那么我们可以直接在 java –jar 命令中添加“--spring.profiles.active”参数，如下所示：</p>
<pre class="lang-xml" data-nodeid="203068"><code data-language="xml">java –jar customerservice-0.0.1-SNAPSHOT.jar --spring.profiles.active=prod
</code></pre>
<p data-nodeid="203069">这种实现方案在通过脚本进行自动化打包和部署的场景下非常有用。</p>
<h4 data-nodeid="203070">代码控制与Profile</h4>
<p data-nodeid="203071">在 Spring Boot 中，Profile 这一概念的应用场景还包括动态控制代码执行流程。为此，我们需要使用 @Profile 注解，先来看一个简单的示例。</p>
<pre class="lang-java" data-nodeid="203072"><code data-language="java"><span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DataSourceConfig</span> </span>{
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Profile("dev")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">devDataSource</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建 dev 环境下的 DataSource </span>
&nbsp;&nbsp;&nbsp; }
&nbsp;
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Bean()</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Profile("prod")</span>
&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> DataSource <span class="hljs-title">prodDataSource</span><span class="hljs-params">()</span></span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//创建 prod 环境下的 DataSource </span>
&nbsp;&nbsp;&nbsp; }
}
</code></pre>
<p data-nodeid="203073">可以看到，我们构建了一个 DataSourceConfig 配置类来专门管理各个环境所需的 DataSource。注意到这里使用 @Profile 注解来指定具体所需要执行的 DataSource 创建代码，通过这种方式，可以达到与使用配置文件相同的效果。</p>
<p data-nodeid="203074">更进一步，能够在代码中控制 JavaBean 的创建过程为我们根据各种条件动态执行代码流程提供了更大的可能性。例如，在日常开发过程中，一个常见的需求是根据不同的运行环境初始化数据，常见的做法是独立执行一段代码或脚本。基于 @Profile 注解，我们就可以将这一过程包含在代码中并做到自动化，如下所示：</p>
<pre class="lang-java" data-nodeid="203075"><code data-language="java"><span class="hljs-meta">@Profile("dev")</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">DevDataInitConfig</span> </span>{
&nbsp;
<span class="hljs-meta">@Bean</span>
&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> CommandLineRunner <span class="hljs-title">dataInit</span><span class="hljs-params">()</span> </span>{ 
&nbsp;&nbsp;&nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> CommandLineRunner() {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-meta">@Override</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">run</span><span class="hljs-params">(String... args)</span> <span class="hljs-keyword">throws</span> Exception </span>{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <span class="hljs-comment">//执行 Dev 环境的数据初始化</span>
&nbsp;&nbsp;&nbsp; };&nbsp; 
}
</code></pre>
<p data-nodeid="203076">这里用到了 Spring Boot 所提供了启动时任务接口 CommandLineRunner，实现了该接口的代码会在 Spring Boot 应用程序启动时自动进行执行，我们会在后续的课程中看到这个接口的具体使用方法。</p>
<p data-nodeid="203077">@Profile 注解的应用范围很广，我们可以将它添加到包含 @Configuration 和 @Component  注解的类及其方法，也就是说可以延伸到继承了 @Component 注解的 @Service、@Controller、@Repository 等各种注解中。</p>
<h4 data-nodeid="203078">常见配置场景和内容</h4>
<p data-nodeid="203079">在今天课程的最后，我们给出几个常见的配置示例来帮助你进一步加深对 Spring Boot 中配置体系的理解。</p>
<p data-nodeid="203080">对于一个 Web 应用程序而言，最常见的配置可能就是指定服务暴露的端口地址，如下所示：</p>
<pre class="lang-xml" data-nodeid="203081"><code data-language="xml">server:
&nbsp;&nbsp;&nbsp; port: 8080
</code></pre>
<p data-nodeid="203082">同时，数据库访问也是 Web 应用程序的基本功能，因此，关于数据源的设置也是常见的一种配置场景，我们在 02 讲中创建第一个 Spring Boot Web 应用程序时给出了一个基本的示例。这里再以 JPA 为例，给出如下所示的一种配置方案：</p>
<pre class="lang-xml" data-nodeid="203083"><code data-language="xml">spring:
&nbsp; jpa:
&nbsp;&nbsp;&nbsp; hibernate:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ddl-auto: create
&nbsp;&nbsp;&nbsp; show-sql: true
</code></pre>
<p data-nodeid="203084">显然，这里使用了 Hibernate 作为 JPA 规范的实现框架，并设置了 show-sql 等相关属性。然后，开发人员一般也需要设置日志级别和对象，如下所示的就是一个典型的配置示例：</p>
<pre class="lang-xml" data-nodeid="203085"><code data-language="xml">logging.level.root=WARN
logging.level.com.springcss.customer=INFO
</code></pre>
<p data-nodeid="203086">我们设置了系统的全局日志级别为 WARN，而针对自定义的 com.springcss.customer 包下的日志则将其级别调整到 INFO。</p>
<p data-nodeid="203087">随时本课程内容的展开，这些常见的配置信息都会在我们的 SpringCSS 案例分析中得到展示。这里需要注意的是，Spring Boot 基于 application.properties 或 application.yml 全局配置文件已经自动内置了很多默认配置。即使我们不设置上述配置内容，Spring Boot 仍然可以基于这些默认配置完成系统的初始化。自动配置是 Spring Boot 中的一个核心概念，我们会在后续内容中给出详细的实现原理分析。</p>
<h3 data-nodeid="203088">小结与预告</h3>
<p data-nodeid="203089">配置体系是学习 Spring Boot 应用程序的基础。在今天的课程中，我们系统梳理了 Spring Boot 中的 Profile 概念，以及如何通过配置文件和代码控制的方式来使用这一核心概念。</p>
<p data-nodeid="203090">这里给你留一道思考题：在使用 Spring Boot 时，使用 Profile 的方式具体有哪些？</p>
<p data-nodeid="203091">尽管 Spring Boot 内置了大量的默认配置信息，但在日常开发过程中，我们还通常会需要设置各种自定义的配置信息。04 讲我们将继续讨论 Spring Boot 中的配置体系，并关注创建和使用自定义配置信息的各种实现方法。</p>

---

### 精选评论

##### *俊：
> 使用越简单，那么源码的学习更显很重要

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 是这样的，技术使用越简单，有时对于开发人员来说反而越难，为了真正使用起来有效率，我们将不得不花些功夫。

##### **7093：
> 根据老师所讲。实际操作了一把。一个全局的 application.yml 配置文件和多个局部的 profile 配置文件的情况下，通过spring:  profiles:    active: test,dev指定加载某一个或多个局部profile后，springboo加载profile顺序为1application.yml，2application-test.yml，3application-dev.yml。全局profile第一个加载，2和3的顺序取决于active配置的值，上面例子逗号前面的test早于dev加载。相同配置项，后加载的profile会覆盖先加载的profile。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 太赞了，学习完立马应用实操，很棒~

##### *奇：
> 比较简单，期待更有难度的

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 有难度的还在后头哦，加油学~

##### **皆可花椒油：
> 老师能否在后序的课程中讲讲spring boot 中的这些注解啊？？往往搞不清这些注解什么时候用，对注解的底层含义也感到模糊，不知道为什么要加注解，以及在哪里加注解，加注解的作用时什么？？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 我们会对一部分核心注解进行展开讨论的~

##### sss：
> 同时激活多个profile，以哪个为准呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Profile可以有很多个，但激活的只能有一个。

##### **伟：
> 多数据源还可以这么做。。

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 是不是感觉发现了新大陆~嘻嘻

##### **伟：
> 验证了下active的profile和配置文件的关系1.如果指定的active的profile不存在，会走默认的application.yaml2.如果默认的application.yaml和指定的profile的配置文件中都有某个配置，那么指定的profile的配置生效。3.如果application.yaml和环境变量中都指定了spring.profiles.active，那么环境变量生效

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 好滴，学习很细致。

##### *星：
> 突然想起来我在开发中，分布式项目，工具放在单独的一个spring-utils中，然后在spring-admin中调用，使用Autowired竟然不能引用过来，怎么办？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 你把工具类所在的库引入进来，应该就能使用Autowired自动注入进来了。

##### **1576：
> 有一个问题一直困扰我，希望老师能够答疑解惑！传统的ssm项目里配置分散但也可以说各司其职方便管理。如果springboot项目很庞大，技术栈比较深，一个application.yml内容比较多时，能不能拆分？怎么拆分？当然，另问我的问题是不是伪需求？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以的，Spring Boot的配置体系里面有个Profile的概念，可以把内容根据不同的类别进行拆分，分Profile进行组织。

##### Zorrrrrro：
> 遇到过奇怪的问题，在全局中设置，在激活的profile 设置，然后有些配置生效的是全局，有些生效的是局部profile。不知道是哪里有 bug ，全局和局部profile 是共存然后有个优先级的吧。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，会进行合并的，关于加载顺序我们课程里有介绍的，你可以再看一下，然后自己多试试~

##### **7540：
> 样例代码没有更新到git

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 因目前课程刚开始，还没到案例阶段，后期随着课程的陆续更新代码会上传至 Github上。

