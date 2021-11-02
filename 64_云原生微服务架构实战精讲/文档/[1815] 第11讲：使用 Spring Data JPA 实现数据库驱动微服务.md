<p>在确定了微服务的 API 之后，我们就可以开始微服务的具体实现，微服务在实现时，并不限制所使用的编程语言或框架。虽然微服务的功能各有不同，但大部分微服务都是以数据库来驱动的。也就是说，这些微服务有一个后台数据库，可能是关系型数据库或 NoSQL 数据库。很多开发人员应该都有过开发数据库驱动的应用的经验，数据库驱动的微服务实现，与一般的数据库驱动的应用并没有太大的区别。</p>
<p>因为数据库驱动的应用十分流行，市面上相关的参考资料也非常多，本课时不打算对很多实现细节进行介绍，而是侧重于一些与微服务相关的内容，以及需要特别注意的内容。本课时以乘客管理微服务作为示例进行说明，数据库相关的实现使用 Spring Data JPA 访问 PostgreSQL 数据库。完整的实现请参考示例应用的源代码。</p>
<h4>聚合、实体和值对象</h4>
<p>随着对象关系映射（Object-Relational Mapping，ORM）以及 Hibernate 这样的框架的流行，数据库驱动的应用的实现变得简单了很多。<strong>对象关系映射</strong>指的是对象模型和数据库关系模型之间的映射。对象模型由类声明和类之间的引用关系组成，数据库的关系模型指的是数据库中的表和表之间的关系。这两种模型存在阻抗不匹配（Impedance Mismatch）的情况，对象模型可以使用继承和多态，而关系模型则要求对数据进行归一化处理。对象之间的引用方式很简单，而关系模型中则需要定义表的外键。如何在两个模型之间进行映射，这也是 ORM 技术的复杂性所在。</p>
<p>当然，ORM 技术本身并不是很难掌握的技术，Hibernate 这样的框架已经为我们屏蔽了很多底层实现细节。我们需要掌握的只是一些使用模式。比如，对象之间的引用关系，在一对多的映射中，什么时候使用单向关系，什么时候使用双向关系。这些都是有模式可以遵循的。</p>
<p>ORM 技术中最常使用的概念是<strong>实体</strong>（Entity）。在领域驱动设计中，与模型相关的有 3 个概念，分别是聚合、实体和值对象。<strong>聚合</strong>是一个抽象的概念，不需要对应到具体的实体。实体需要映射成 ORM 中的实体。值对象通常不会被映射成单一实体，而是作为其他实体的一部分，实体的标识符被映射成数据库的主键。</p>
<p>在乘客管理微服务中，聚合乘客的根实体是乘客，用户地址实体表示乘客所保存的地址，乘客实体中包含对用户地址实体的引用。区分实体和值对象的关键在于，对象是否有各自独立的生命周期。以用户地址为例，每个用户地址都可以被用户创建、更新和删除，因此它们有各自独立的生命周期，也就是说用户地址属于实体。用户地址实体属于聚合乘客的一部分。</p>
<h4>领域对象</h4>
<p>在创建实体类时，一个需要注意的问题是避免反模式贫血对象，<strong>贫血对象</strong>指的是对象类中只有属性声明以及属性的 getter 和 setter 方法。贫血对象实际上退化成为属性的数据容器，并没有其他的行为。贫血对象不符合我们对领域对象的期望，领域对象的行为应该是完备的。以乘客对象为例，与用户地址相关的管理功能，都应该添加在乘客对象中。这一点对聚合的根实体尤为重要，聚合的根实体需要负责维护业务逻辑中的不变量。与维护不变量相关的代码都应该直接被添加到实体类中。</p>
<p>当用 JPA 实现数据访问时，我们可以用领域对象类来作为 JPA 的实体类，只需要添加 JPA 的相关注解即可。下面代码中的 Passenger 类是领域对象类，同时也是 JPA 的实体类。Passenger 类中字段都添加了 JPA 的相关注解。Lombok 的注解用来生成 getter、setter、构造器和 toString 方法。Passenger 实体和 UserAddress 实体之间存在一对多关系，使用注解 @OneToMany 来表示。管理 UserAddress 对象的方法都添加在 Passenger 类中了，这体现了 Passenger 类是聚合的根。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table</span>(name = <span class="hljs-string">"passengers"</span>)
<span class="hljs-meta">@Getter</span>
<span class="hljs-meta">@Setter</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@ToString</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Passenger</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">BaseEntityWithGeneratedId</span> </span>{

  <span class="hljs-meta">@Column</span>(name = <span class="hljs-string">"name"</span>)
  <span class="hljs-meta">@Size</span>(max = <span class="hljs-number">255</span>)
  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> String name;

  <span class="hljs-meta">@Column</span>(name = <span class="hljs-string">"email"</span>)
  <span class="hljs-meta">@Email</span>
  <span class="hljs-meta">@Size</span>(max = <span class="hljs-number">255</span>)
  <span class="hljs-keyword">private</span> String email;

  <span class="hljs-meta">@Column</span>(name = <span class="hljs-string">"mobile_phone_number"</span>)
  <span class="hljs-meta">@Size</span>(max = <span class="hljs-number">255</span>)
  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> String mobilePhoneNumber;

  <span class="hljs-meta">@OneToMany</span>(cascade = CascadeType.ALL, orphanRemoval = <span class="hljs-keyword">true</span>)
  <span class="hljs-meta">@JoinColumn</span>(name = <span class="hljs-string">"passenger_id"</span>, referencedColumnName = <span class="hljs-string">"id"</span>,
      nullable = <span class="hljs-keyword">false</span>)
  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> List&lt;UserAddress&gt; userAddresses = <span class="hljs-keyword">new</span> ArrayList&lt;&gt;();

  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">addUserAddress</span><span class="hljs-params">(UserAddress userAddress)</span> </span>{
    <span class="hljs-keyword">if</span> (userAddress != <span class="hljs-keyword">null</span>) {
      userAddresses.add(userAddress);
    }
  }

  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeUserAddress</span><span class="hljs-params">(UserAddress userAddress)</span> </span>{
    userAddresses.remove(userAddress);
  }

  <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">removeUserAddress</span><span class="hljs-params">(String addressId)</span> </span>{
    getUserAddress(addressId).ifPresent(<span class="hljs-keyword">this</span>::removeUserAddress);
  }

  <span class="hljs-function"><span class="hljs-keyword">public</span> Optional&lt;UserAddress&gt; <span class="hljs-title">getUserAddress</span><span class="hljs-params">(String addressId)</span> </span>{
    <span class="hljs-keyword">return</span> userAddresses.stream()
        .filter(address -&gt; Objects.equals(address.getId(), addressId))
        .findFirst();
  }
}
</code></pre>
<h4>数据访问</h4>
<p>对于一个聚合来说，只有聚合的根实体可以被外部对象所访问，因此只需要对聚合的根实体创建资源库即可。在乘客管理微服务中，我们只需要为乘客实体创建对应的资源库即可。下面代码中的 PassengerRepository 接口是乘客实体对应的资源库。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Repository</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">PassengerRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">CrudRepository</span>&lt;<span class="hljs-title">Passenger</span>, <span class="hljs-title">String</span>&gt; </span>{

}
</code></pre>
<p>一个需要考虑的问题是数据库表模式的生成。Hibernate 这样的 ORM 框架都支持从实体声明中自动生成和更新数据库表模式。这种做法看起来很简单方便，但是存在很多问题。</p>
<p>第一个问题是数据库表模式的优化问题。为了优化数据库的查询性能，数据库表模式通常需要由专业的人员进行设计。由 Hibernate 这样的框架所生成的数据库表模式只是通用的实现，并没有对特定应用进行优化。</p>
<p>第二个问题是数据库表模式的更新问题。在更新代码时，如果涉及到对数据库表模式的修改，直接使用 Hibernate 提供的更新功能并不是一个好选择。最主要的原因是自动更新的结果并不可控，尤其是需要对已有的数据进行更新时。</p>
<p>更好的做法是手动维护数据库表模式，并使用数据库迁移工具来更新模式。示例应用使用的迁移工具是 Flyway。在乘客管理微服务中，目录 src/main/resources/db/migration 中包含了数据库的迁移脚本。迁移脚本的文件名称类似 V1__init_schema.sql ，其中的前缀 V1 表示的是脚本的版本号。Flyway 会根据当前数据库中记录的版本信息来确定哪些脚本需要运行，并把数据库表模式升级到最新版本。</p>
<h4>领域层</h4>
<p>领域层包括领域对象和服务实现，服务实现直接使用资源库来对实体进行操作。在设计服务实现的接口时，一个常见的做法是使用领域对象作为参数和返回值。在下面的代码中，PassengerService 类中的 getPassenger 方法的返回值是 Optional 类型，直接引用了领域类 Passenger。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-function"><span class="hljs-keyword">public</span> Optional&lt;Passenger&gt; <span class="hljs-title">getPassenger</span><span class="hljs-params">(String passengerId)</span> </span>{}
</code></pre>
<p>这种做法既简单又直接，不过却有两个不足之处。</p>
<p>第一个不足之处在于对外部对象暴露了聚合的实体及其引用的对象。外部对象获取到实体的引用之后，是可以通过该对象来修改状态的，可能会产生意想不到的结果。</p>
<p>第二个不足之处是在 Hibernate 的实现上，乘客实体引用了一个用户地址实体的列表。从性能的角度考虑，对于一个乘客对象来说，它的用户地址列表是延迟获取的。也就是说，只有在第一次获取用户地址列表中的元素时，才会从数据库中读取。而读取数据库需要一个打开的 Hibernate 会话。当在 REST API 的控制器中访问 Passenger 对象中的用户地址列表时，为了操作可以成功，就要求 Hibernate 会话仍然处于打开状态，这带来的结果就是 Hibernate 会话的打开时间过长，影响性能。更合理的做法应该是在服务对象的方法退出时，就关闭会话。</p>
<p>综合上面两个原因，直接使用领域对象作为服务对象方法的返回值，并不是一个好的选择，更好的做法是使用值对象作为返回值。值对象作为领域对象中所包含的数据的复制，去掉了领域对象中包含的业务逻辑，只是单纯的作为数据容器。这使得使用者在获取数据的同时，又无法改变内部实体对象的状态。由于转换成值对象的逻辑发生在服务方法内部，并不会影响 Hibernate 会话的关闭。这种做法同时解决了上述两个问题，应该是值得推荐的做法。Spring Data JPA中的配置属性 spring.jpa.open-in-view 可以控制会话是否在控制器中打开，该属性的默认值为 true。在应用了这种模式之后，该属性的值应该被设置为 false 。</p>
<p>下面代码中的 PassengerVO 类是 Passenger 实体对应的值对象。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Data</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerVO</span> </span>{
  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> String id;

  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> String name;

  <span class="hljs-keyword">private</span> String email;

  <span class="hljs-meta">@NonNull</span>
  <span class="hljs-keyword">private</span> String mobilePhoneNumber;

  <span class="hljs-keyword">private</span> List&lt;UserAddressVO&gt; userAddresses;
}
</code></pre>
<p>下面的代码给出了服务实现 PassengerService 类的部分代码，PassengerVO 类作为返回值的类型。</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@Service</span>
<span class="hljs-meta">@Transactional</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerService</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  PassengerRepository passengerRepository;

  <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;PassengerVO&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> Streams.stream(passengerRepository.findAll())
        .map(PassengerUtils::createPassengerVO)
        .collect(Collectors.toList());
  }
}
</code></pre>
<h4>展示层</h4>
<p>对于微服务来说，其展示层就是它们对外提供的 API，这个 API 可以被其他微服务、Web 界面和移动客户端来使用。示例应用使用 JSON 表示的 REST API。对于使用 Spring Boot 和 Spring 框架的微服务实现来说，暴露 REST API 是非常简单的事情，可以 Spring MVC 或 Spring WebFlux。</p>
<p>下面代码给出了乘客服务的 REST API 控制器的部分代码：</p>
<pre><code data-language="java" class="lang-java"><span class="hljs-meta">@RestController</span>
<span class="hljs-meta">@RequestMapping</span>(<span class="hljs-string">"/api/v1"</span>)
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PassengerController</span> </span>{

  <span class="hljs-meta">@Autowired</span>
  PassengerService passengerService;

  <span class="hljs-meta">@GetMapping</span>
  <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;PassengerVO&gt; <span class="hljs-title">findAll</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">return</span> passengerService.findAll();
  }
}
</code></pre>
<p>在第 10 课时中，我提到了 Swagger 的代码生成工具可以生成服务器存根代码，这其中就包括了对 Spring 框架的支持。在采用了 API 优先的策略之后，我们可以从 OpenAPI 文档中生成 API 服务端代码的骨架，并以此作为实际实现的基础。通过这种方式，可以快速创建一个可工作的 API 服务器。</p>
<p>不过需要注意两个问题。首先是 Swagger 代码生成工具创建的项目是基于 Spring Boot 1.5 的。如果你期望使用 Spring Boot 2，那么需要先自己进行升级；其次是通过工具生成的代码并不一定符合你的团队的开发规范，代码生成过程是单向不可逆的。如果 OpenAPI 文档发生了改变，再次生成会覆盖掉之前所做的手动修改。因此，建议的做法是仅在测试中使用自动生成的服务器实现。实际的产品代码应该手动创建和维护。</p>
<p>在开发中的一个常见需求是发送 HTTP 请求来测试 REST API ，如果使用 Postman 或其他工具，可以直接导入 OpenAPI 文档来生成 HTTP 请求的模板，如下图所示，Postman 自动生成了 POST 请求的内容。</p>
<p><img src="https://s0.lgstatic.com/i/image3/M01/13/79/Ciqah16f2byAMICiAADnG6dPSdw777.png" alt="微服务11.png"></p>
<h4>总结</h4>
<p>数据库驱动的微服务代表了一大类的微服务。通过 Spring Boot 和 Spring 框架，我们可以很容易的创建出暴露 REST API 的数据库驱动的微服务。本课时对数据库驱动的微服务中的重点内容进行了说明，可以帮助掌握重要的知识点。</p>

---

### 精选评论

##### **林：
> 直接导入 OpenAPI 文档来生成 HTTP 请求的模板，那直接写请求的body体不是更快吗？为什么要用OpenAPI呢？不理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 导入OpenAPI文档的好处在于工具可以从OpenAPI文档的请求格式中，自动生成请求的模板，并预先填充一些数据，这比从头开始写请求内容要容易得多。这就意味着在发送请求时，并不需要再去了解请求的具体格式，可以更快速的使用API。OpenAPI的优势就是在API的提供者和使用者之间创建一个契约，这个契约的存在使得使用者可以更方便的使用API，而不需要依赖服务器端的实现。

##### **阔：
> 个人感觉，领域模型设计所带来的是更彻底的面向对象，当然，同时带来的还有设计的复杂性。还是不要在小系统里用了，得不偿失。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 在所要解决的问题很小时，领域驱动设计的复杂度确实很高，它所带来的效益，不足以抵消它的复杂度。在微服务架构的设计中，领域驱动设计确实有它适合使用的地方，重点在于对微服务的划分上。较大的系统推荐使用领域驱动设计的思想。

