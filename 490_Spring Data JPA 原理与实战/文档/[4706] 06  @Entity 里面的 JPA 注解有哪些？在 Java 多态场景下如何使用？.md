<p data-nodeid="444245">前几课时我为你介绍了 Repository 的用法，其中我经常会提到“实体类”（即我们前面的 User 类），它是对我们数据库中表的 Metadata 映射，那么具体如何映射呢？这一课时我们来讲解。</p>


<p data-nodeid="442930">我们先看一下 Java Persistence API 里面都有哪些重要规定；再通过讲解基本注解，重点介绍一下联合主键和实体之间的继承关系，然后你就会知道 JPA 的实体里面常见的注解有哪些。话不多说，看一下 Entity 的相关规定。</p>
<h3 data-nodeid="442931">JPA 协议中关于 Entity 的相关规定</h3>
<p data-nodeid="442932">我们先看一下 JPA 协议里面关于实体做了哪些规定。（这里推荐一个查看 JPA 协议的官方地址：<a href="https://download.oracle.com/otn-pub/jcp/persistence-2_2-mrel-spec/JavaPersistence.pdf" data-nodeid="443106">https://download.oracle.com/otn-pub/jcp/persistence-2_2-mrel-spec/JavaPersistence.pdf</a>）</p>
<ol data-nodeid="442933">
<li data-nodeid="442934">
<p data-nodeid="442935">实体是直接进行数据库持久化操作的领域对象（即一个简单的 POJO，可以按照业务领域划分），必须通过 @Entity 注解进行标示。</p>
</li>
<li data-nodeid="442936">
<p data-nodeid="442937">实体必须有一个 public 或者 protected 的无参数构造方法。</p>
</li>
<li data-nodeid="442938">
<p data-nodeid="442939">持久化映射的注解可以标示在 Entity 的字段 field 上，如下所示：</p>
</li>
</ol>
<pre class="lang-java" data-nodeid="442940"><code data-language="java"><span class="hljs-meta">@Column(length = 20, nullable = false)</span>
<span class="hljs-keyword">private</span> String userName;
</code></pre>
<p data-nodeid="442941">除此之外，也可以将持久化注解运用在 Entity 里面的 get/set 方法上，通常我们是放在 get 方法中，如下所示：</p>
<pre class="lang-java" data-nodeid="442942"><code data-language="java"><span class="hljs-meta">@Column(length = 20, nullable = false)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUserName</span><span class="hljs-params">()</span></span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> userName;
}
</code></pre>
<p data-nodeid="445121">概括起来，就是 Entity 里面的注解生效只有两种方式：将注解写在字段上或者将注解写在方法上（JPA 里面称 Property）。</p>
<p data-nodeid="445122">但是<strong data-nodeid="445129">需要注意的是，在同一个 Entity 里面只能有一种方式生效</strong>，也就是说，注解要么全部写在 field 上面，要么就全部写在 Property 上面，因为我经常会看到有的同事分别在两种方式中加了注解后说：“哎呀，我的注解怎么没有生效呀！”因此这一点需要特别注意。</p>

<ol data-nodeid="442944">
<li data-nodeid="442945">
<p data-nodeid="442946">只要是在 @Entity 的实体里面被注解标注的字段，都会被映射到数据库中，除了使用 @Transient 注解的字段之外。</p>
</li>
<li data-nodeid="442947">
<p data-nodeid="442948">实体里面必须要有一个主键，主键标示的字段可以是单个字段，也可以是复合主键字段。</p>
</li>
</ol>
<p data-nodeid="442949">以上我只挑选了最关键的几条进行了介绍，如果你有兴趣可以读一读 Java Persistence API 协议，这样我们在做 JPA 开发的时候就会顺手很多，可以理解很多 Hibernate 里面实现方法。</p>
<p data-nodeid="442950">这也为你提供了一条解决疑难杂症的思路，也就是当我们遇到解决不了的问题时，就去看协议、阅读官方文档，深入挖掘一下，可能就会找到答案。那么接下来我们看看实例里面常用的注解有哪些。</p>
<h3 data-nodeid="442951">详细的注解都有哪些？</h3>
<p data-nodeid="442952">我们先通过源码看看 JPA 里面支持的注解有哪些。</p>
<p data-nodeid="446870">首先，我们利用 IEDA 工具，打开 @Entity 所在的包，就可以看到 JPA 里面支持的注解有哪些。如下所示：</p>
<p data-nodeid="446871" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/4E/Ciqc1F9rLPSAFNw1AAQfaEA3Bgo587.png" alt="Drawing 0.png" data-nodeid="446875"></p>


<p data-nodeid="442955">我们可以看到，在 jakarta.persistence-api 的包路径下面大概有一百多个注解，你在没事的时候可以到这里面一个一个地看，也可以到 JPA 的协议里面对照查看文档。</p>
<p data-nodeid="442956">我在这里只提及一些最常见的，包括 @Entity、@Table、@Access、@Id、@GeneratedValue、@Enumerated、@Basic、@Column、@Transient、@Lob、@Temporal 等。</p>
<p data-nodeid="447750" class="">1.<strong data-nodeid="447756">@Entity</strong> 用于定义对象将会成为被 JPA 管理的实体，必填，将字段映射到指定的数据库表中，使用起来很简单，直接用在实体类上面即可，通过源码表达的语法如下：</p>

<pre class="lang-java" data-nodeid="442958"><code data-language="java"><span class="hljs-meta">@Target(TYPE)</span> <span class="hljs-comment">//表示此注解只能用在class上面</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Entity {
   <span class="hljs-comment">//可选，默认是实体类的名字，整个应用里面全局唯一。</span>
   <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
}
</code></pre>
<p data-nodeid="448631" class="">2.<strong data-nodeid="448637">@Table</strong> 用于指定数据库的表名，表示此实体对应的数据库里面的表名，非必填，默认表名和 entity 名字一样。</p>

<pre class="lang-java" data-nodeid="442960"><code data-language="java"><span class="hljs-meta">@Target(TYPE)</span> <span class="hljs-comment">//一样只能用在类上面</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Table {
   <span class="hljs-comment">//表的名字，可选。如果不填写，系统认为好实体的名字一样为表名。</span>
   <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//此表所在schema，可选</span>
   <span class="hljs-function">String <span class="hljs-title">schema</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//唯一性约束，在创建表的时候有用，表创建之后后面就不需要了。</span>
   UniqueConstraint[] uniqueConstraints() <span class="hljs-keyword">default</span> { };
   <span class="hljs-comment">//索引，在创建表的时候使用，表创建之后后面就不需要了。</span>
   Index[] indexes() <span class="hljs-keyword">default</span> {};
}
</code></pre>
<p data-nodeid="449512" class="">3.<strong data-nodeid="449653">@Access</strong> 用于指定 entity 里面的注解是写在字段上面，还是 get/set 方法上面生效，非必填。在默认不填写的情况下，当实体里面的第一个注解出现在字段上或者 get/set 方法上面，就以第一次出现的方式为准；也就是说，一个实体里面的注解既有用在 field 上面，又有用在 properties 上面的时候，看下面的代码你就会明白。</p>
<pre class="lang-java" data-nodeid="449513"><code data-language="java"><span class="hljs-meta">@Id</span>
<span class="hljs-keyword">private</span> Long id;
<span class="hljs-meta">@Column(length = 20, nullable = false)</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getUserName</span><span class="hljs-params">()</span></span>{
&nbsp; &nbsp; <span class="hljs-keyword">return</span> userName;
}
</code></pre>
<p data-nodeid="449514">那么由于 @Id 是实体里面第一个出现的注解，并且作用在字段上面，所以所有写在 get/set 方法上面的注解就会失效。而 @Access 可以干预默认值，指定是在 fileds 上面生效还是在 properties 上面生效。我们通过源码看下语法：</p>
<pre class="lang-java" data-nodeid="449515"><code data-language="java"><span class="hljs-meta">@Target( { TYPE, METHOD, FIELD })</span><span class="hljs-comment">//表示此注解可以运用在class上(那么这个时候就可以指定此实体的默认注解生效策略了)，也可以用在方法上或者字段上(表示可以独立设置某一个字段或者方法的生效策略)；</span>
<span class="hljs-meta">@Retention(RUNTIME)</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Access {
<span class="hljs-comment">//指定是字段上面生效还是方法上面生效</span>
    <span class="hljs-function">AccessType <span class="hljs-title">value</span><span class="hljs-params">()</span></span>;
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">AccessType</span> </span>{
    FIELD,
    PROPERTY
}
</code></pre>
<p data-nodeid="451633">4.<strong data-nodeid="451640">@Id</strong> 定义属性为数据库的主键，一个实体里面必须有一个主键，但不一定是这个注解，可以和 @GeneratedValue 配合使用或成对出现。</p>
<p data-nodeid="452521" class="">5.<strong data-nodeid="452527">@GeneratedValue</strong> 主键生成策略，如下所示：</p>



<pre class="lang-java" data-nodeid="449517"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> GeneratedValue {
    <span class="hljs-comment">//Id的生成策略</span>
    <span class="hljs-function">GenerationType <span class="hljs-title">strategy</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> AUTO</span>;
    <span class="hljs-comment">//通过Sequences生成Id,常见的是Orcale数据库ID生成规则，这个时候需要配合@SequenceGenerator使用</span>
    <span class="hljs-function">String <span class="hljs-title">generator</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
}
</code></pre>
<p data-nodeid="449518">其中，GenerationType 一共有以下四个值：</p>
<pre class="lang-java" data-nodeid="449519"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">GenerationType</span> </span>{
    <span class="hljs-comment">//通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植。</span>
    TABLE,
    <span class="hljs-comment">//通过序列产生主键，通过 @SequenceGenerator 注解指定序列名， MySql 不支持这种方式；</span>
    SEQUENCE,
    <span class="hljs-comment">//采用数据库ID自增长， 一般用于mysql数据库</span>
    IDENTITY,
<span class="hljs-comment">//JPA 自动选择合适的策略，是默认选项；</span>
    AUTO
}
</code></pre>
<p data-nodeid="453402" class="">6.<strong data-nodeid="453408">@Enumerated</strong> 这个注解很好用，因为它对 enum 提供了下标和 name 两种方式，用法直接映射在 enum 枚举类型的字段上。请看下面源码。</p>

<pre class="lang-java" data-nodeid="449521"><code data-language="java"><span class="hljs-meta">@Target({METHOD, FIELD})</span> <span class="hljs-comment">//作用在方法和字段上</span>
<span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Enumerated {
<span class="hljs-comment">//枚举映射的类型，默认是ORDINAL（即枚举字段的下标）。</span>
    <span class="hljs-function">EnumType <span class="hljs-title">value</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ORDINAL</span>;
}
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">EnumType</span> </span>{
    <span class="hljs-comment">//映射枚举字段的下标</span>
    ORDINAL,
    <span class="hljs-comment">//映射枚举的Name</span>
    STRING
}
</code></pre>
<p data-nodeid="449522">再来看一个 User 里面关于性别枚举的例子，你就会知道 @Enumerated 在这里没什么作用了，如下所示：</p>
<pre class="lang-java" data-nodeid="449523"><code data-language="java"><span class="hljs-comment">//有一个枚举类，用户的性别</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">Gender</span> </span>{
    MAIL(<span class="hljs-string">"男性"</span>), FMAIL(<span class="hljs-string">"女性"</span>);
    <span class="hljs-keyword">private</span> String value;
    <span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-title">Gender</span><span class="hljs-params">(String value)</span> </span>{
        <span class="hljs-keyword">this</span>.value = value;
    }
}
<span class="hljs-comment">//实体类@Enumerated的写法如下</span>
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table(name = "tb_user")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
    <span class="hljs-meta">@Enumerated(EnumType.STRING)</span>
    <span class="hljs-meta">@Column(name = "user_gender")</span>
    <span class="hljs-keyword">private</span> Gender gender;
    .......................
}
</code></pre>
<p data-nodeid="455167">这时候插入两条数据，数据库里面的值会变成 MAIL/FMAIL，而不是“男性” / 女性。</p>
<p data-nodeid="455168"><strong data-nodeid="455174">经验分享：</strong> 如果我们用 @Enumerated（EnumType.ORDINAL），这时候数据库里面的值是  0、1。但是实际工作中，不建议用数字下标，因为枚举里面的属性值是会不断新增的，如果新增一个，位置变化了就惨了。并且 0、1、2 这种下标在数据库里面看着非常痛苦，时间长了就会一点也看不懂了。</p>


<p data-nodeid="456051" class="">7.<strong data-nodeid="456057">@Basic</strong> 表示属性是到数据库表的字段的映射。如果实体的字段上没有任何注解，默认即为  @Basic。也就是说默认所有的字段肯定是和数据库进行映射的，并且默认为 Eager 类型。</p>

<pre class="lang-java" data-nodeid="449526"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Basic {
    <span class="hljs-comment">//可选，EAGER（默认）：立即加载；LAZY：延迟加载。（LAZY主要应用在大字段上面）</span>
    <span class="hljs-function">FetchType <span class="hljs-title">fetch</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> EAGER</span>;
    <span class="hljs-comment">//可选。这个字段是否可以为null，默认是true。</span>
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">optional</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
}
</code></pre>
<p data-nodeid="458714">8.<strong data-nodeid="458721">@Transient</strong> 表示该属性并非一个到数据库表的字段的映射，表示非持久化属性。JPA 映射数据库的时候忽略它，与 @Basic 有相反的作用。也就是每个字段上面 @Transient 和 @Basic 必须二选一，而什么都不指定的话，默认是 @Basic。</p>
<p data-nodeid="458715">9.<strong data-nodeid="458727">@Column</strong> 定义该属性对应数据库中的列名。</p>



<pre class="lang-java" data-nodeid="449528"><code data-language="java"><span class="hljs-keyword">public</span> <span class="hljs-meta">@interface</span> Column {
    <span class="hljs-comment">//数据库中的表的列名；可选，如果不填写认为字段名和实体属性名一样。</span>
    <span class="hljs-function">String <span class="hljs-title">name</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
    <span class="hljs-comment">//是否唯一。默认flase，可选。</span>
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">unique</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">false</span></span>;
    <span class="hljs-comment">//数据字段是否允许空。可选，默认true。</span>
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">nullable</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
    <span class="hljs-comment">//执行insert操作的时候是否包含此字段，默认，true，可选。</span>
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">insertable</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
    <span class="hljs-comment">//执行update的时候是否包含此字段，默认，true，可选。</span>
    <span class="hljs-function"><span class="hljs-keyword">boolean</span> <span class="hljs-title">updatable</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">true</span></span>;
    <span class="hljs-comment">//表示该字段在数据库中的实际类型。</span>
    <span class="hljs-function">String <span class="hljs-title">columnDefinition</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> ""</span>;
   <span class="hljs-comment">//数据库字段的长度，可选，默认255</span>
    <span class="hljs-function"><span class="hljs-keyword">int</span> <span class="hljs-title">length</span><span class="hljs-params">()</span> <span class="hljs-keyword">default</span> 255</span>;
}
</code></pre>
<p data-nodeid="459604" class="">10.<strong data-nodeid="459610">@Temporal</strong> 用来设置 Date 类型的属性映射到对应精度的字段，存在以下三种情况：</p>

<ul data-nodeid="449530">
<li data-nodeid="449531">
<p data-nodeid="449532">@Temporal(TemporalType.DATE)映射为日期 // date （<strong data-nodeid="449712">只有日期</strong>）</p>
</li>
<li data-nodeid="449533">
<p data-nodeid="449534">@Temporal(TemporalType.TIME)映射为日期 // time （<strong data-nodeid="449718">只有时间</strong>）</p>
</li>
<li data-nodeid="449535">
<p data-nodeid="449536">@Temporal(TemporalType.TIMESTAMP)映射为日期 // date time （<strong data-nodeid="449724">日期+时间</strong>）</p>
</li>
</ul>
<p data-nodeid="449537">我们看一个完整的例子，感受一下上面提到的注解的完整用法，如下：</p>
<pre class="lang-java" data-nodeid="449538"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-keyword">import</span> java.util.Date;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table(name = "user_topic")</span>
<span class="hljs-meta">@Access(AccessType.FIELD)</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserTopic</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@Column(name = "id", nullable = false)</span>
   <span class="hljs-meta">@GeneratedValue(strategy = GenerationType.IDENTITY)</span>
   <span class="hljs-keyword">private</span> Integer id;
   <span class="hljs-meta">@Column(name = "title", nullable = true, length = 200)</span>
   <span class="hljs-keyword">private</span> String title;
   <span class="hljs-meta">@Basic</span>
   <span class="hljs-meta">@Column(name = "create_user_id", nullable = true)</span>
   <span class="hljs-keyword">private</span> Integer createUserId;
   <span class="hljs-meta">@Basic(fetch = FetchType.LAZY)</span>
   <span class="hljs-meta">@Column(name = "content", nullable = true, length = -1)</span>
   <span class="hljs-meta">@Lob</span>
   <span class="hljs-keyword">private</span> String content;
   <span class="hljs-meta">@Basic(fetch = FetchType.LAZY)</span>
   <span class="hljs-meta">@Column(name = "image", nullable = true)</span>
   <span class="hljs-meta">@Lob</span>
   <span class="hljs-keyword">private</span> <span class="hljs-keyword">byte</span>[] image;
   <span class="hljs-meta">@Basic</span>
   <span class="hljs-meta">@Column(name = "create_time", nullable = true)</span>
   <span class="hljs-meta">@Temporal(TemporalType.TIMESTAMP)</span>
   <span class="hljs-keyword">private</span> Date createTime;
   <span class="hljs-meta">@Basic</span>
   <span class="hljs-meta">@Column(name = "create_date", nullable = true)</span>
   <span class="hljs-meta">@Temporal(TemporalType.DATE)</span>
   <span class="hljs-keyword">private</span> Date createDate;
   <span class="hljs-meta">@Enumerated(EnumType.STRING)</span>
   <span class="hljs-meta">@Column(name = "topic_type")</span>
   <span class="hljs-keyword">private</span> Type type;
   <span class="hljs-meta">@Transient</span>
   <span class="hljs-keyword">private</span> String transientSimple;
   <span class="hljs-comment">//非数据库映射字段，业务类型的字段</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">getTransientSimple</span><span class="hljs-params">()</span> </span>{
      <span class="hljs-keyword">return</span> title + <span class="hljs-string">"auto:jack"</span> + type;
   }
   <span class="hljs-comment">//有一个枚举类，主题的类型</span>
   <span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">enum</span> <span class="hljs-title">Type</span> </span>{
      EN(<span class="hljs-string">"英文"</span>), CN(<span class="hljs-string">"中文"</span>);
      <span class="hljs-keyword">private</span> <span class="hljs-keyword">final</span> String des;
      Type(String des) {
         <span class="hljs-keyword">this</span>.des = des;
      }
   }
}
</code></pre>
<p data-nodeid="460487">细心的同学就会发现，我们在一开始的 demo 里面没有这么多注解呀，其实这里面的很多注解都可以省略，直接使用默认的就可以。如 @Basic、@Column 名字有一定的映射策略（我们在第 17 课时讲 DataSource 的时候会详细讲解映射策略），所以可以省略。</p>
<p data-nodeid="460488">此外，@Access 也可以省略，我们只要在这些类里面保持一致就可以了。可能你会有疑问了，这么多注解都要手动一个一个配置吗？老师介绍一种简单的做法——利用工具去生成 Entity 类，将会节省很多时间。</p>

<h3 data-nodeid="449540">生成这些注解的小技巧</h3>
<p data-nodeid="449541">有时候老的 Table 非常多，我们一个一个去写 entity 会特别累，因此我们可以利用 IEDA 工具直接帮我们生成 Entity 类。关键步骤如下。</p>
<p data-nodeid="462235">首先，<strong data-nodeid="462246">打开 Persistence 视图，点击 Generate Persistence Mapping&gt;</strong>，接着<strong data-nodeid="462247">点击选中数据源</strong>，如下图所示：</p>
<p data-nodeid="462236" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/5A/CgqCHl9rLS6AK2kJAAGJp8EnMkE910.png" alt="Drawing 1.png" data-nodeid="462250"></p>


<p data-nodeid="463995">然后，<strong data-nodeid="464002">选择表和字段，并点击 OK</strong>。</p>
<p data-nodeid="463996" class=""><img src="https://s0.lgstatic.com/i/image/M00/56/5A/CgqCHl9rLTOAO3rIAAH3r2qRk40730.png" alt="Drawing 2.png" data-nodeid="464005"></p>


<p data-nodeid="449546">这样就可以生成我们想要的实体了，多简单。如果是新库、新表，我们也可以先定义好实体，通过实体配置JPA的spring.jpa.generate-ddl=true，反向直接生成 DDL 操作数据库生成表结构。</p>
<p data-nodeid="449547">但是需要注意的是，在生产环境中我们要把外间关联关系关闭，不然会出现意想不到的 ERROR，毕竟生产环境不同开发环境，我们可以通过在开发环境生成的表导出 DDL 到生产执行。我经常会利用生成 DDL 来做测试和写案例， 这样省去了创建表的时间，只需要关注我的代码就行了。</p>
<p data-nodeid="449548">接下来我们再把工作中最常见的联合 ID 字段的场景详细讲解一下。</p>
<h3 data-nodeid="449549">联合主键</h3>
<p data-nodeid="449550">在实际的工作中，我们会经常遇到联合主键的情况。所以在这里我们详细讲解一下，可以通过 javax.persistence.EmbeddedId 和 javax.persistence.IdClass 两个注解实现联合主键的效果。</p>
<h4 data-nodeid="449551">如何通过 @IdClass 做到联合主键？</h4>
<p data-nodeid="449552">我们先看一下怎么通过 @IdClass 做到联合主键。</p>
<p data-nodeid="449553">第一步：新建一个 UserInfoID 类里面是联合主键。</p>
<pre class="lang-java" data-nodeid="449554"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> lombok.AllArgsConstructor;
<span class="hljs-keyword">import</span> lombok.Builder;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.NoArgsConstructor;
<span class="hljs-keyword">import</span> java.io.Serializable;
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoID</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
   <span class="hljs-keyword">private</span> String name,telephone;
}
</code></pre>
<p data-nodeid="449555">第二步：再新建一个 UserInfo 的实体，采用 @IdClass 引用联合主键类。</p>
<pre class="lang-java" data-nodeid="449556"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@IdClass(UserInfoID.class)</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> </span>{
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-meta">@Id</span>
   <span class="hljs-keyword">private</span> String name;
   <span class="hljs-meta">@Id</span>
   <span class="hljs-keyword">private</span> String telephone;
}
</code></pre>
<p data-nodeid="449557">第三步：新增一个 UserInfoReposito 类来做 CRUD 操作。</p>
<pre class="lang-java" data-nodeid="449558"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserInfoRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">UserInfo</span>,<span class="hljs-title">UserInfoID</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="449559">第四步：写一个测试用例，测试一下。</p>
<pre class="lang-java" data-nodeid="449560"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-keyword">import</span> java.util.Optional;
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoRepositoryTest</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> UserInfoRepository userInfoRepository;
   <span class="hljs-meta">@Test</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testIdClass</span><span class="hljs-params">()</span> </span>{
   userInfoRepository.save(UserInfo.builder().ages(<span class="hljs-number">1</span>).name(<span class="hljs-string">"jack"</span>).telephone(<span class="hljs-string">"123456789"</span>).build());
      Optional&lt;UserInfo&gt; userInfo = userInfoRepository.findById(UserInfoID.builder().name(<span class="hljs-string">"jack"</span>).telephone(<span class="hljs-string">"123456789"</span>).build());
      System.out.println(userInfo.get());
   }
}
Hibernate: <span class="hljs-function">create table <span class="hljs-title">user_info</span> <span class="hljs-params">(name varchar(<span class="hljs-number">255</span>)</span> not <span class="hljs-keyword">null</span>, telephone <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span> not <span class="hljs-keyword">null</span>, ages integer, primary <span class="hljs-title">key</span> <span class="hljs-params">(name, telephone)</span>)
Hibernate: select userinfo0_.name as name1_3_0_, userinfo0_.telephone as telephon2_3_0_, userinfo0_.ages as ages3_3_0_ from user_info userinfo0_ where userinfo0_.name</span>=? and userinfo0_.telephone=?
UserInfo(ages=<span class="hljs-number">1</span>, name=jack, telephone=<span class="hljs-number">123456789</span>)
</code></pre>
<p data-nodeid="449561">通过上面的例子我们可以发现，我们的表的主键是 primary key (name, telephone)，而 Entity 里面不再是一个 @Id 字段了。那么我来介绍另外一个注解 @Embeddable，也能做到这一点。</p>
<h4 data-nodeid="467519" class="">@Embeddable 与 @EmbeddedId 注解使用</h4>




<p data-nodeid="449563">第一步：在我们上面例子中的 UserInfoID 里面添加 @Embeddable 注解。</p>
<pre class="lang-java" data-nodeid="449564"><code data-language="java"><span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@Embeddable</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfoID</span> <span class="hljs-keyword">implements</span> <span class="hljs-title">Serializable</span> </span>{
   <span class="hljs-keyword">private</span> String name,telephone;
}
</code></pre>
<p data-nodeid="449565">第二步：改一下我们刚才的 User 对象，删除 @IdClass，添加 @EmbeddedId 注解，如下：</p>
<pre class="lang-java" data-nodeid="449566"><code data-language="java"><span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserInfo</span> </span>{
   <span class="hljs-keyword">private</span> Integer ages;
   <span class="hljs-meta">@EmbeddedId</span>
   <span class="hljs-keyword">private</span> UserInfoID userInfoID;
   <span class="hljs-meta">@Column(unique = true)</span>
   <span class="hljs-keyword">private</span> String uniqueNumber;
}
</code></pre>
<p data-nodeid="449567">第三步：UserInfoRepository 不变，我们直接修改一下测试用例。</p>
<pre class="lang-java" data-nodeid="449568"><code data-language="java"><span class="hljs-meta">@Test</span>
<span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testIdClass</span><span class="hljs-params">()</span> </span>{
  userInfoRepository.save(UserInfo.builder().ages(<span class="hljs-number">1</span>).userInfoID(UserInfoID.builder().name(<span class="hljs-string">"jack"</span>).telephone(<span class="hljs-string">"123456789"</span>).build()).build());
   Optional&lt;UserInfo&gt; userInfo = userInfoRepository.findById(UserInfoID.builder().name(<span class="hljs-string">"jack"</span>).telephone(<span class="hljs-string">"123456789"</span>).build());
   System.out.println(userInfo.get());
}
</code></pre>
<p data-nodeid="449569">运行完之后，你可以得到相同的结果。那么 @IdClass 和 @EmbeddedId 的区别是什么？有以下两个方面：</p>
<ol data-nodeid="449570">
<li data-nodeid="449571">
<p data-nodeid="449572">如上面测试用例，在使用的时候，Embedded 用的是对象，而 IdClass 用的是具体的某一个字段；</p>
</li>
<li data-nodeid="449573">
<p data-nodeid="449574">二者的JPQL 也会不一样：</p>
</li>
</ol>
<p data-nodeid="449575">① 用 @IdClass JPQL 的写法：SELECT u.name FROM UserInfo u</p>
<p data-nodeid="449576">② 用 @EmbeddedId 的 JPQL 的写法：select u.userInfoId.name FROM UserInfo u</p>
<p data-nodeid="449577">联合主键还有需要注意的就是，它与唯一性索引约束的区别是写法不同，如上面所讲，唯一性索引的写法如下：</p>
<pre class="lang-java" data-nodeid="449578"><code data-language="java"><span class="hljs-meta">@Column(unique = true)</span>
<span class="hljs-keyword">private</span> String uniqueNumber;
</code></pre>
<p data-nodeid="468391">到这里，联合主键我们讲完了，那么在遇到联合主键的时候，利用 @IdClass、@EmbeddedId，你就可以应对联合主键了。</p>
<p data-nodeid="468392">此外，Java 是面向对象的，肯定会用到多态的使用场景，那么场景都有哪些？公共父类又该如何写？我们来学习一下。</p>

<h3 data-nodeid="449580">实体之间的继承关系如何实现？</h3>
<p data-nodeid="449581">在 Java 面向对象的语言环境中，@Entity 之间的关系多种多样，而根据 JPA 的规范，我们大致可以将其分为以下几种：</p>
<ol data-nodeid="449582">
<li data-nodeid="449583">
<p data-nodeid="449584">纯粹的继承，和表没关系，对象之间的字段共享。利用注解 @MappedSuperclass，协议规定父类不能是 @Entity。</p>
</li>
<li data-nodeid="449585">
<p data-nodeid="449586">单表多态问题，同一张 Table，表示了不同的对象，通过一个字段来进行区分。利用<code data-backticks="1" data-nodeid="449786">@Inheritance(strategy = InheritanceType.SINGLE_TABLE)</code>注解完成，只有父类有 @Table。</p>
</li>
<li data-nodeid="449587">
<p data-nodeid="449588">多表多态，每一个子类一张表，父类的表拥有所有公用字段。通过<code data-backticks="1" data-nodeid="449789">@Inheritance(strategy = InheritanceType.JOINED)</code>注解完成，父类和子类都是表，有公用的字段在父表里面。</p>
</li>
<li data-nodeid="449589">
<p data-nodeid="449590">Object 的继承，数据库里面每一张表是分开的，相互独立不受影响。通过<code data-backticks="1" data-nodeid="449792">@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)</code>注解完成，父类（可以是一张表，也可以不是）和子类都是表，相互之间没有关系。</p>
</li>
</ol>
<p data-nodeid="449591">其中，第一种 @MappedSuperclass，我们暂时不多介绍，在第 12 课时讲解“JPA 的审计功能”时，再做详细介绍，我们先看一下第二种<code data-backticks="1" data-nodeid="449795">SINGLE_TABLE</code>。</p>
<h4 data-nodeid="449592">@Inheritance(strategy = InheritanceType.SINGLE_TABLE)</h4>
<p data-nodeid="449593">父类实体对象与各个子实体对象共用一张表，通过一个字段的不同值代表不同的对象，我们看一个例子。</p>
<p data-nodeid="449594">我们抽象一个 Book 对象，如下所示：</p>
<pre class="lang-java" data-nodeid="449595"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-meta">@Entity(name="book")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Inheritance(strategy = InheritanceType.SINGLE_TABLE)</span>
<span class="hljs-meta">@DiscriminatorColumn(name="color", discriminatorType = DiscriminatorType.STRING)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String title;
}
</code></pre>
<p data-nodeid="449596">再新建一个 BlueBook 对象，作为 Book 的子对象。</p>
<pre class="lang-java" data-nodeid="449597"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.EqualsAndHashCode;
<span class="hljs-keyword">import</span> javax.persistence.DiscriminatorValue;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-meta">@DiscriminatorValue("blue")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BlueBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span></span>{
   <span class="hljs-keyword">private</span> String blueMark;
}
</code></pre>
<p data-nodeid="449598">再新建一个 RedBook 对象，作为 Book 的另一子对象。</p>
<pre class="lang-java" data-nodeid="449599"><code data-language="java"><span class="hljs-comment">//红皮书</span>
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@DiscriminatorValue("red")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-keyword">private</span> String redMark;
}
</code></pre>
<p data-nodeid="469265">这时，我们一共新建了三个 Entity 对象，其实都是指 book 这一张表，通过 book 表里面的 color 字段来区分红书还是绿书。我们继续做一下测试看看结果。</p>
<p data-nodeid="469266">我们再新建一个 RedBookRepositor 类，操作一下 RedBook 会看到如下结果：</p>

<pre class="lang-java" data-nodeid="449601"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">RedBookRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">RedBook</span>,<span class="hljs-title">Long</span>&gt;</span>{
}
</code></pre>
<p data-nodeid="449602">然后再新建一个测试用例。</p>
<pre class="lang-java" data-nodeid="449603"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1;
<span class="hljs-keyword">import</span> com.example.jpa.example1.book.RedBook;
<span class="hljs-keyword">import</span> com.example.jpa.example1.book.RedBookRepository;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedBookRepositoryTest</span> </span>{
   <span class="hljs-meta">@Autowired</span>
   <span class="hljs-keyword">private</span> RedBookRepository redBookRepository;
   <span class="hljs-meta">@Test</span>
   <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testRedBook</span><span class="hljs-params">()</span> </span>{
      RedBook redBook = <span class="hljs-keyword">new</span> RedBook();
      redBook.setTitle(<span class="hljs-string">"redbook"</span>);
      redBook.setRedMark(<span class="hljs-string">"redmark"</span>);
      redBook.setId(<span class="hljs-number">1L</span>);
      redBookRepository.saveAndFlush(redBook);
      RedBook r = redBookRepository.findById(<span class="hljs-number">1L</span>).get();
    System.out.println(r.getId()+<span class="hljs-string">":"</span>+r.getTitle()+<span class="hljs-string">":"</span>+r.getRedMark());
   }
}
</code></pre>
<p data-nodeid="449604">最后看一下执行结果。</p>
<pre class="lang-java" data-nodeid="449605"><code data-language="java">Hibernate: <span class="hljs-function">create table <span class="hljs-title">book</span> <span class="hljs-params">(color varchar(<span class="hljs-number">31</span>)</span> not <span class="hljs-keyword">null</span>, id bigint not <span class="hljs-keyword">null</span>, title <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, blue_mark <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, red_mark <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)
</span></code></pre>
<p data-nodeid="449606">你会发现，我们只创建了一张表，insert 了一条数据，但是我们发现 color 字段默认给的是 red。</p>
<pre class="lang-java" data-nodeid="449607"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">book</span> <span class="hljs-params">(title, red_mark, color, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, <span class="hljs-string">'red'</span>, ?)</span>
</span></code></pre>
<p data-nodeid="449608">那么再看一下打印结果。</p>
<pre class="lang-java" data-nodeid="449609"><code data-language="java"><span class="hljs-number">1</span>:redbook:redmark
</code></pre>
<p data-nodeid="449610">结果完全和预期的一样，这说明了 RedBook、BlueBook、Book，都是一张表，通过字段 color 的值不一样，来区分不同的实体。<br>
那么接下来我们看一下 InheritanceType.JOINED，它的每个实体都是独立的表。</p>
<h4 data-nodeid="449611">@Inheritance(strategy = InheritanceType.JOINED)</h4>
<p data-nodeid="449612">在这种映射策略里面，继承结构中的每一个实体（entity）类都会映射到数据库里一个单独的表中。也就是说，每个实体（entity）都会被映射到数据库中，一个实体（entity）类对应数据库中的一个表。</p>
<p data-nodeid="449613">其中根实体（root entity）对应的表中定义了主键（primary key），所有的子类对应的数据库表都要共同使用 Book 里面的 @ID 这个主键。</p>
<p data-nodeid="449614">首先，我们改一下上面的三个实体，测试一下InheritanceType.JOINED，改动如下：</p>
<pre class="lang-java" data-nodeid="449615"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-meta">@Entity(name="book")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Inheritance(strategy = InheritanceType.JOINED)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String title;
}
</code></pre>
<p data-nodeid="449616">其次，我们 Book 父类、改变 Inheritance 策略、删除 DiscriminatorColumn，你会看到如下结果。</p>
<pre class="lang-java" data-nodeid="449617"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.EqualsAndHashCode;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-keyword">import</span> javax.persistence.PrimaryKeyJoinColumn;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-meta">@PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BlueBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span></span>{
   <span class="hljs-keyword">private</span> String blueMark;
}
<span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.EqualsAndHashCode;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-keyword">import</span> javax.persistence.PrimaryKeyJoinColumn;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-keyword">private</span> String redMark;
}
</code></pre>
<p data-nodeid="449618">然后，BlueBook和RedBook也删除DiscriminatorColumn，新增@PrimaryKeyJoinColumn(name = "book_id", referencedColumnName = "id")，和 book 父类共用一个主键值，而 RedBookRepository 和测试用例不变，我们执行看一下结果。</p>
<pre class="lang-java" data-nodeid="449619"><code data-language="java">Hibernate: <span class="hljs-function">create table <span class="hljs-title">blue_book</span> <span class="hljs-params">(blue_mark varchar(<span class="hljs-number">255</span>)</span>, book_id bigint not <span class="hljs-keyword">null</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(book_id)</span>)
Hibernate: create table <span class="hljs-title">book</span> <span class="hljs-params">(id bigint not <span class="hljs-keyword">null</span>, title varchar(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)
Hibernate: create table <span class="hljs-title">red_book</span> <span class="hljs-params">(red_mark varchar(<span class="hljs-number">255</span>)</span>, book_id bigint not <span class="hljs-keyword">null</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(book_id)</span>)
Hibernate: alter table blue_book add constraint FK9uuwgq7a924vtnys1rgiyrlk7 foreign <span class="hljs-title">key</span> <span class="hljs-params">(book_id)</span> references book
Hibernate: alter table red_book add constraint FKk8rvl61bjy9lgsr9nhxn5soq5 foreign <span class="hljs-title">key</span> <span class="hljs-params">(book_id)</span> references book
</span></code></pre>
<p data-nodeid="449620">上述代码可以看到，我们一共创建了三张表，并且新增了两个外键约束；而我们 save 的时候也生成了两个 insert 语句，如下：</p>
<pre class="lang-java" data-nodeid="449621"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">book</span> <span class="hljs-params">(title, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?)</span>
Hibernate: insert into <span class="hljs-title">red_book</span> <span class="hljs-params">(red_mark, book_id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?)</span>
</span></code></pre>
<p data-nodeid="449622">而打印结果依然不变。</p>
<pre class="lang-java" data-nodeid="449623"><code data-language="java"><span class="hljs-number">1</span>:redbook:redmark
</code></pre>
<p data-nodeid="449624">这就是 InheritanceType.JOINED 的例子，这个方法和上面的 InheritanceType.SINGLE_TABLE 区别在于表的数量和关系不一样，这是表设计的另一种方式。</p>
<h4 data-nodeid="449625">@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)</h4>
<p data-nodeid="449626">我们在使用 @MappedSuperClass 主键的时候，如果不指定 @Inhertance，默认就是此种TABLE_PER_CLASS模式。当然了，我们也显示指定，要求继承基类的都是一张表，而父类不是表，是 java 对象的抽象类。我们看一个例子。</p>
<p data-nodeid="449627">首先，还是改一下上面的三个实体。</p>
<pre class="lang-java" data-nodeid="449628"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> javax.persistence.*;
<span class="hljs-meta">@Entity(name="book")</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-meta">@Id</span>
   <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
   <span class="hljs-keyword">private</span> Long id;
   <span class="hljs-keyword">private</span> String title;
}
</code></pre>
<p data-nodeid="449629">其次，Book 表采用 TABLE_PER_CLASS 策略，其子实体类都代表各自的表，实体代码如下：</p>
<pre class="lang-java" data-nodeid="449630"><code data-language="java"><span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.EqualsAndHashCode;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">RedBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span> </span>{
   <span class="hljs-keyword">private</span> String redMark;
}
<span class="hljs-keyword">package</span> com.example.jpa.example1.book;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.EqualsAndHashCode;
<span class="hljs-keyword">import</span> javax.persistence.Entity;
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@EqualsAndHashCode(callSuper=false)</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">BlueBook</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Book</span></span>{
   <span class="hljs-keyword">private</span> String blueMark;
}
</code></pre>
<p data-nodeid="449631">这时，从 RedBook 和 BlueBook 里面去掉 PrimaryKeyJoinColumn，而 RedBookRepository 和测试用例不变，我们执行看一下结果。</p>
<pre class="lang-java" data-nodeid="449632"><code data-language="java">Hibernate: <span class="hljs-function">create table <span class="hljs-title">blue_book</span> <span class="hljs-params">(id bigint not <span class="hljs-keyword">null</span>, title varchar(<span class="hljs-number">255</span>)</span>, blue_mark <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)
Hibernate: create table <span class="hljs-title">book</span> <span class="hljs-params">(id bigint not <span class="hljs-keyword">null</span>, title varchar(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)
Hibernate: create table <span class="hljs-title">red_book</span> <span class="hljs-params">(id bigint not <span class="hljs-keyword">null</span>, title varchar(<span class="hljs-number">255</span>)</span>, red_mark <span class="hljs-title">varchar</span><span class="hljs-params">(<span class="hljs-number">255</span>)</span>, primary <span class="hljs-title">key</span> <span class="hljs-params">(id)</span>)
</span></code></pre>
<p data-nodeid="449633">这里可以看到，我们还是创建了三张表，但三张表什么关系也没有。而 insert 语句也只有一条，如下：</p>
<pre class="lang-java" data-nodeid="449634"><code data-language="java">Hibernate: <span class="hljs-function">insert into <span class="hljs-title">red_book</span> <span class="hljs-params">(title, red_mark, id)</span> <span class="hljs-title">values</span> <span class="hljs-params">(?, ?, ?)</span>
</span></code></pre>
<p data-nodeid="449635">打印结果还是不变。</p>
<pre class="lang-java" data-nodeid="449636"><code data-language="java"><span class="hljs-number">1</span>:redbook:redmark
</code></pre>
<p data-nodeid="449637">这个方法与上面两个相比较，语义更加清晰，是比较常用的一种做法。<br>
以上就是实体之间继承关系的实现方法，可以在涉及 java 多态的时候加以应用，不过要注意区分三种方式所表达的表的意思，再加以运用。</p>
<h4 data-nodeid="449638">关于继承关系的经验之谈</h4>
<p data-nodeid="449639">从我的个人经验来看，@Inheritance 的这种使用方式会逐渐被淘汰，因为这样的表的设计很复杂，本应该在业务层面做的事情（多态），而在 datasoure 的表级别做了。所以在 JPA 中使用这个的时候你就会想：“这么复杂的东西，我直接用 Mybatis 算了。”我想告诉你，其实它们是一样的，只是我们使用的思路不对。</p>
<p data-nodeid="449640">那么为什么行业内都不建议使用了，还要介绍这么详细呢？因为，如果你遇到的是老一点的项目，如果不是用 Java 语言写的，不一定有面向对象的思想。这个时候如果让你迁移成 Java 怎么办？如果你可以想到这种用法，就不至于束手无措。</p>
<p data-nodeid="449641">此外，在互联网项目中，一旦有关表的业务对象过多了之后，就可以拆表拆库了，这个时候我们要想到我们的@Table 注解指定表名和 schema。</p>
<p data-nodeid="449642">关于上面提到的方法中，最常用的是第一种 @MappedSuperclass，这个我们将在第 12 课时“JPA 的审计功能解决了哪些问题？”中详细介绍，到时候你可以体验一下它的不同之处。</p>
<h3 data-nodeid="449643">总结</h3>
<p data-nodeid="449644">Entity 里面常用的基本注解我们就介绍到这里，因为注解太多没办法一一介绍，你可以掌握一下学习方法。先通过源码把大致注解看一下，有哪些不熟悉的可以看看源码里面的注释，再阅读 JPA 官方协议，还可以写一个测试用例试，跑一下看看 sql 输出和日志，这样很快就可以知道结果了。</p>
<p data-nodeid="470139" class="">这一课时我们提到的实体与实体之间的关联关系注解，我将在下一课时为你讲解。</p>

<blockquote data-nodeid="449646">
<p data-nodeid="449647">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="449870">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *哥：
> JPS 如何指定主键类型为 varchar？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; @Id 就可以了呀，然后自己新增之前赋值字符串呀，或者利用数据库的GUID

##### **6245：
> JPA怎么指定主键的起始ID 比如指定他从100000开始而不是从1

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; @TableGenerator(name="tab", initialValue=100000)或者@SequenceGenerator(name = "port_gen", sequenceName = "port_gen",  initialValue = 700) 注意两者区别

