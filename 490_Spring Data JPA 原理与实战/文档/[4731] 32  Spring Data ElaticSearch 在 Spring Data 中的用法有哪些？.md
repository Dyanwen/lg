<p data-nodeid="4294" class="">这一讲是这门专栏的最后一讲了，恭喜你一直坚持到现在。</p>


<p data-nodeid="3156">相信到这里，你已经对 Spring Data JPA 有一定的认识了，那么这一讲我会为你演示 Spring Data ElasticSearch 如何使用，帮助你打开思路，感受 Spring Data 的抽象封装。</p>
<p data-nodeid="3157">我们还是从一个案例入手。</p>
<h3 data-nodeid="3158">Spring Data ElasticSearch 入门案例</h3>
<p data-nodeid="3159">Spring Data 和 Elasticsearch 结合的时候，唯一需要注意的是版本之间的兼容性问题，Elasticsearch 和 Spring Boot 是同时向前发展的，而 Elasticsearch 的大版本之间还存在一定的 API 兼容性问题，所以我们必须要知道这些版本之间的关系，我整理了一个表格，如下。</p>
<table data-nodeid="5053">
<thead data-nodeid="5054">
<tr data-nodeid="5055">
<th align="center" data-nodeid="5057"><strong data-nodeid="5099">Spring Data Release Train</strong></th>
<th data-nodeid="5058"><strong data-nodeid="5103">Spring Data Elasticsearch</strong></th>
<th align="center" data-nodeid="5059"><strong data-nodeid="5107">Elasticsearch</strong></th>
<th data-nodeid="5060"><strong data-nodeid="5111">Spring Boot</strong></th>
</tr>
</thead>
<tbody data-nodeid="5065">
<tr data-nodeid="5066">
<td align="center" data-nodeid="5067">2020.0.0[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1" data-nodeid="5116">1</a>]</td>
<td data-nodeid="5068">4.1.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1" data-nodeid="5122">1</a>]</td>
<td align="center" data-nodeid="5069">7.9.3</td>
<td data-nodeid="5070">2.4.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_1" data-nodeid="5129">1</a>]</td>
</tr>
<tr data-nodeid="5071">
<td align="center" data-nodeid="5072">Neumann</td>
<td data-nodeid="5073">4.0.x</td>
<td align="center" data-nodeid="5074">7.6.2</td>
<td data-nodeid="5075">2.3.x</td>
</tr>
<tr data-nodeid="5076">
<td align="center" data-nodeid="5077">Moore</td>
<td data-nodeid="5078">3.2.x</td>
<td align="center" data-nodeid="5079">6.8.12</td>
<td data-nodeid="5080">2.2.x</td>
</tr>
<tr data-nodeid="5081">
<td align="center" data-nodeid="5082">Lovelace</td>
<td data-nodeid="5083">3.1.x</td>
<td align="center" data-nodeid="5084">6.2.2</td>
<td data-nodeid="5085">2.1.x</td>
</tr>
<tr data-nodeid="5086">
<td align="center" data-nodeid="5087">Kay[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5147">2</a>]</td>
<td data-nodeid="5088">3.0.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5153">2</a>]</td>
<td align="center" data-nodeid="5089">5.5.0</td>
<td data-nodeid="5090">2.0.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5160">2</a>]</td>
</tr>
<tr data-nodeid="5091">
<td align="center" data-nodeid="5092">Ingalls[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5166">2</a>]</td>
<td data-nodeid="5093">2.1.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5172">2</a>]</td>
<td align="center" data-nodeid="5094">2.4.0</td>
<td data-nodeid="5095">1.5.x[<a href="https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#_footnotedef_2" data-nodeid="5179">2</a>]</td>
</tr>
</tbody>
</table>

<p data-nodeid="3208">现在你对这些版本之间的关联关系有了一定印象，由于版本越新越便利，所以一般情况下我们直接采用最新的版本。</p>
<p data-nodeid="3209">接下来看看这个版本是怎么完成 Demo 演示的。</p>
<p data-nodeid="3210"><strong data-nodeid="3389">第一步：利用 Helm Chart 安装一个 Elasticsearch 集群 7.9.3 版本</strong>，执行命令如下。</p>
<pre class="lang-java" data-nodeid="3211"><code data-language="java"><span class="hljs-number">1</span>. helm2 repo add elastic https:<span class="hljs-comment">//helm.elastic.co</span>
<span class="hljs-number">2</span>. helm2 install --name myelasticsearch elastic/elasticsearch&nbsp; --set imageTag=<span class="hljs-number">7.9</span><span class="hljs-number">.3</span>
</code></pre>
<p data-nodeid="3212">安装完之后，我们就可以看到如下信息。</p>
<p data-nodeid="5929" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/56/CgpVE1_tdLyAY0UfAAH31rOGV0o472.png" alt="Drawing 0.png" data-nodeid="5932"></p>

<p data-nodeid="3214">这代表我们安装成功。</p>
<p data-nodeid="3215">由于 ElasticSearch 是发展变化的，所以它的安装方式你可以参考官方文档：<a href="https://github.com/elastic/helm-charts/tree/master/elasticsearch" data-nodeid="3398">https://github.com/elastic/helm-charts/tree/master/elasticsearch</a></p>
<p data-nodeid="3216">然后我们利用 k8s 集群端口映射到本地，就可以开始测试了。</p>
<pre class="lang-java" data-nodeid="3217"><code data-language="java">~ ❯❯❯ kubectl port-forward svc/elasticsearch-master <span class="hljs-number">9200</span>:<span class="hljs-number">9200</span> -n my-namespace
Forwarding from <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">9200</span> -&gt; <span class="hljs-number">9200</span>
Forwarding from [::<span class="hljs-number">1</span>]:<span class="hljs-number">9200</span> -&gt; <span class="hljs-number">9200</span>
</code></pre>
<p data-nodeid="3218"><strong data-nodeid="3404">第二步：在 gradle.build 里面配置 Spring Data ElasticSearch 依赖的 Jar 包</strong>。</p>
<p data-nodeid="3219">我们依赖 Spring Boot 2.4.1 版本，完整的 gradle.build 文件如下所示。</p>
<pre class="lang-java" data-nodeid="3220"><code data-language="java">plugins {
   id <span class="hljs-string">'org.springframework.boot'</span> version <span class="hljs-string">'2.4.1'</span>
   id <span class="hljs-string">'io.spring.dependency-management'</span> version <span class="hljs-string">'1.0.10.RELEASE'</span>
   id <span class="hljs-string">'java'</span>
}
group = <span class="hljs-string">'com.example.data.es'</span>
version = <span class="hljs-string">'0.0.1-SNAPSHOT'</span>
sourceCompatibility = <span class="hljs-string">'1.8'</span>
configurations {
   compileOnly {
      extendsFrom annotationProcessor
   }
}
repositories {
   mavenCentral()
}
dependencies {
   implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-actuator'</span>
   implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-data-elasticsearch'</span>
   implementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-web'</span>
   compileOnly <span class="hljs-string">'org.projectlombok:lombok'</span>
   developmentOnly <span class="hljs-string">'org.springframework.boot:spring-boot-devtools'</span>
   runtimeOnly <span class="hljs-string">'io.micrometer:micrometer-registry-prometheus'</span>
   annotationProcessor <span class="hljs-string">'org.projectlombok:lombok'</span>
   testImplementation <span class="hljs-string">'org.springframework.boot:spring-boot-starter-test'</span>
}
test {
   useJUnitPlatform()
}
</code></pre>
<p data-nodeid="3221"><strong data-nodeid="3410">第三步：新建一个目录，结构如下图所示，方便我们测试</strong>。</p>
<p data-nodeid="6681" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/55/Cip5yF_tdMaAbP03AAD7ix9soGU430.png" alt="Drawing 1.png" data-nodeid="6684"></p>

<p data-nodeid="3223"><strong data-nodeid="3418">第四步：在 application.properties 里面新增 es 的连接地址，连接本地的 Elasticsearch</strong>。</p>
<pre class="lang-java" data-nodeid="3224"><code data-language="java">spring.data.elasticsearch.client.reactive.endpoints=<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">9200</span>
</code></pre>
<p data-nodeid="3225"><strong data-nodeid="3423">第五步：新增一个 ElasticSearchConfiguration 的配置文件，主要是为了开启扫描的包</strong>。</p>
<pre class="lang-java" data-nodeid="3226"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo.es;
<span class="hljs-keyword">import</span> org.springframework.context.annotation.Configuration;
<span class="hljs-keyword">import</span> org.springframework.data.elasticsearch.repository.config.EnableElasticsearchRepositories;
<span class="hljs-comment">//利用@EnableElasticsearchRepositories注解指定Elasticsearch相关的Repository的包路径在哪里</span>
<span class="hljs-meta">@EnableElasticsearchRepositories(basePackages = "com.example.data.es.demo.es")</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ElasticSearchConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="3227"><strong data-nodeid="3428">第六步：我们新增一个 Topic 的 Document，它类似 JPA 里面的实体，用来保存和读取 Topic 的数据</strong>，代码如下所示。</p>
<pre class="lang-java" data-nodeid="3228"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo.es;
<span class="hljs-keyword">import</span> lombok.Builder;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-keyword">import</span> lombok.ToString;
<span class="hljs-keyword">import</span> org.springframework.data.annotation.Id;
<span class="hljs-keyword">import</span> org.springframework.data.elasticsearch.annotations.Document;
<span class="hljs-keyword">import</span> org.springframework.data.elasticsearch.annotations.Field;
<span class="hljs-keyword">import</span> org.springframework.data.elasticsearch.annotations.FieldType;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@Document(indexName = "topic")</span>
<span class="hljs-meta">@ToString(callSuper = true)</span>
<span class="hljs-comment">//论坛主题信息</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Topic</span> </span>{
    <span class="hljs-meta">@Id</span>
    <span class="hljs-keyword">private</span> Long id;
    <span class="hljs-keyword">private</span> String title;
    <span class="hljs-meta">@Field(type = FieldType.Nested, includeInParent = true)</span>
    <span class="hljs-keyword">private</span> List&lt;Author&gt; authors;
}
<span class="hljs-keyword">package</span> com.example.data.es.demo.es;
<span class="hljs-keyword">import</span> lombok.Builder;
<span class="hljs-keyword">import</span> lombok.Data;
<span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-comment">//作者信息</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Author</span> </span>{
    <span class="hljs-keyword">private</span> String name;
}
</code></pre>
<p data-nodeid="3229"><strong data-nodeid="3433">第七步：新建一个 Elasticsearch 的 Repository，用来对 Elasticsearch 索引的增删改查</strong>，代码如下所示。</p>
<pre class="lang-java" data-nodeid="3230"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo.es;
<span class="hljs-keyword">import</span> org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-comment">//类似JPA一样直接操作Topic类型的索引</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">TopicRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">ElasticsearchRepository</span>&lt;<span class="hljs-title">Topic</span>,<span class="hljs-title">Long</span>&gt; </span>{
    <span class="hljs-function">List&lt;Topic&gt; <span class="hljs-title">findByTitle</span><span class="hljs-params">(String title)</span></span>;
}
</code></pre>
<p data-nodeid="3231"><strong data-nodeid="3437">第八步: 新建一个 Controller，对 Topic 索引进行查询和添加。</strong></p>
<pre class="lang-java" data-nodeid="3232"><code data-language="java"><span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">TopicController</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> TopicRepository topicRepository;
    <span class="hljs-comment">//查询topic的所有索引</span>
    <span class="hljs-meta">@GetMapping("topics")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> List&lt;Topic&gt; <span class="hljs-title">query</span><span class="hljs-params">(<span class="hljs-meta">@Param("title")</span> String title)</span> </span>{
        <span class="hljs-keyword">return</span> topicRepository.findByTitle(title);
    }
    <span class="hljs-comment">//保存 topic索引</span>
    <span class="hljs-meta">@PostMapping("topics")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Topic <span class="hljs-title">create</span><span class="hljs-params">(<span class="hljs-meta">@RequestBody</span> Topic topic)</span> </span>{
        <span class="hljs-keyword">return</span> topicRepository.save(topic);
    }
}
</code></pre>
<p data-nodeid="3233"><strong data-nodeid="3442">第九步：发送一个添加和查询的请求测试一下</strong>。</p>
<p data-nodeid="3234">我们发送三个 POST 请求，添加三条索引，代码如下所示。</p>
<pre class="lang-java" data-nodeid="3235"><code data-language="java">POST /topics HTTP/<span class="hljs-number">1.1</span>
Host: <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8080</span>
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: d9cc1f6c-<span class="hljs-number">24</span>dd-<span class="hljs-number">17f</span>f-f2e8-<span class="hljs-number">3063f</span>a6b86fc
{
    <span class="hljs-string">"title"</span>:<span class="hljs-string">"jack"</span>,
    <span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,
    <span class="hljs-string">"authors"</span>:[{
        <span class="hljs-string">"name"</span>:<span class="hljs-string">"jk1"</span>
        },{
        <span class="hljs-string">"name"</span>:<span class="hljs-string">"jk2"</span>
        }]
}
</code></pre>
<p data-nodeid="3236">然后发送一个 get 请求，获得标题是 jack 的索引，如下面这行代码所示。</p>
<pre class="lang-java" data-nodeid="3237"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8080/topics?title=jack</span>
</code></pre>
<p data-nodeid="3238">得到如下结果。</p>
<pre class="lang-java" data-nodeid="3239"><code data-language="java">GET http:<span class="hljs-comment">//127.0.0.1:8080/topics?title=jack</span>
HTTP/<span class="hljs-number">1.1</span> <span class="hljs-number">200</span>&nbsp;
Content-Type: application/json
Transfer-Encoding: chunked
Date: Wed, <span class="hljs-number">30</span> Dec <span class="hljs-number">2020</span> <span class="hljs-number">15</span>:<span class="hljs-number">12</span>:<span class="hljs-number">16</span> GMT
Keep-Alive: timeout=<span class="hljs-number">60</span>
Connection: keep-alive
[
&nbsp; {
&nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">1</span>,
&nbsp; &nbsp; <span class="hljs-string">"title"</span>: <span class="hljs-string">"jack"</span>,
&nbsp; &nbsp; <span class="hljs-string">"authors"</span>: [
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk1"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk2"</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; ]
&nbsp; },
&nbsp; {
&nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">3</span>,
&nbsp; &nbsp; <span class="hljs-string">"title"</span>: <span class="hljs-string">"jack"</span>,
&nbsp; &nbsp; <span class="hljs-string">"authors"</span>: [
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk1"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk2"</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; ]
&nbsp; },
&nbsp; {
&nbsp; &nbsp; <span class="hljs-string">"id"</span>: <span class="hljs-number">2</span>,
&nbsp; &nbsp; <span class="hljs-string">"title"</span>: <span class="hljs-string">"jack"</span>,
&nbsp; &nbsp; <span class="hljs-string">"authors"</span>: [
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk1"</span>
&nbsp; &nbsp; &nbsp; },
&nbsp; &nbsp; &nbsp; {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-string">"name"</span>: <span class="hljs-string">"jk2"</span>
&nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; ]
&nbsp; }
]
Response code: <span class="hljs-number">200</span>; Time: <span class="hljs-number">348</span>ms; Content length: <span class="hljs-number">199</span> bytes
Cannot preserve cookies, cookie storage file is included in ignored list:
&gt; /Users/jack/Company/git_hub/spring-data-jpa-guide/<span class="hljs-number">2.3</span>/elasticsearch-data/.idea/httpRequests/http-client.cookies
</code></pre>
<p data-nodeid="3240">这时，一个完整的 Spring Data Elasticsearch 的例子就演示完了。其实你会发现，我们使用 Spring Data Elasticsearch 来操作 ES 相关的 API 的话，比我们直接写 Http 的 client 要简单很多，因为这里面帮我们封装了很多基础逻辑，省去了很多重复造轮子的过程。</p>
<p data-nodeid="3241">其实测试用例也是很简单的，我们接着来看一下写法。</p>
<p data-nodeid="3242"><strong data-nodeid="3452">第十步：Elasticsearch Repository 的测试用例写法</strong>，如下面的代码和注释所示。</p>
<pre class="lang-java" data-nodeid="3243"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo;
<span class="hljs-keyword">import</span> com.example.data.es.demo.es.Author;
<span class="hljs-keyword">import</span> com.example.data.es.demo.es.Topic;
<span class="hljs-keyword">import</span> com.example.data.es.demo.es.TopicRepository;
<span class="hljs-keyword">import</span> org.assertj.core.util.Lists;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.BeforeEach;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.context.SpringBootTest;
<span class="hljs-keyword">import</span> org.springframework.test.context.TestPropertySource;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-meta">@SpringBootTest</span>
<span class="hljs-meta">@TestPropertySource(properties = {"logging.level.org.springframework.data.elasticsearch.core=TRACE", "logging.level.org.springframework.data.elasticsearch.client=trace", "logging.level.org.elasticsearch.client=TRACE", "logging.level.org.apache.http=TRACE"})</span><span class="hljs-comment">//新增一些配置， 开启spring data elastic search的http的调用过程，我们可以查看一下日志</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ElasticSearchRepositoryTest</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> TopicRepository topicRepository;
    <span class="hljs-meta">@BeforeEach</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">init</span><span class="hljs-params">()</span> </span>{
<span class="hljs-comment">//        topicRepository.deleteAll(); //可以直接删除所有索引</span>
        Topic topic = Topic.builder().id(<span class="hljs-number">11L</span>).title(<span class="hljs-string">"jacktest"</span>).authors(Lists.newArrayList(Author.builder().name(<span class="hljs-string">"jk1"</span>).build())).build();
        topicRepository.save(topic);<span class="hljs-comment">//集成测试保存索引</span>
        Topic topic1 = Topic.builder().id(<span class="hljs-number">14L</span>).title(<span class="hljs-string">"jacktest"</span>).authors(Lists.newArrayList(Author.builder().name(<span class="hljs-string">"jk1"</span>).build())).build();
        topicRepository.save(topic1);
        Topic topic2 = Topic.builder().id(<span class="hljs-number">15L</span>).title(<span class="hljs-string">"jacktest"</span>).authors(Lists.newArrayList(Author.builder().name(<span class="hljs-string">"jk1"</span>).build())).build();
        topicRepository.save(topic2);<span class="hljs-comment">//保存索引</span>
    }
    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testTopic</span><span class="hljs-params">()</span> </span>{
        Iterable&lt;Topic&gt; topics = topicRepository.findAll();
        topics.forEach(topic1 -&gt; {
            System.out.println(topic1);
        });
        List&lt;Topic&gt; topicList = topicRepository.findByTitle(<span class="hljs-string">"jacktest"</span>);
        topicList.forEach(t -&gt; {
            System.out.println(t);<span class="hljs-comment">//获得索引的查询结果</span>
        });
        List&lt;Topic&gt; topicList2 = topicRepository.findByTitle(<span class="hljs-string">"xxx"</span>);
        topicList2.forEach(t -&gt; {
            System.out.println(t);<span class="hljs-comment">//我们也可以用上一讲介绍的断言测试</span>
        });
    }
}
</code></pre>
<p data-nodeid="3244">接着我们看一下测试用例的调用日志，从日志可以看出，调用的时候发生的 Http 的 PUT 请求，是用来创建和修改一个索引的文档的。请看下面的图片。</p>
<p data-nodeid="7433" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/56/CgpVE1_tdNiAXq0WAAPx9WYUcvE585.png" alt="Drawing 2.png" data-nodeid="7436"></p>

<p data-nodeid="3246" class="">从中也可以看得出来，转化成 es 的 api 查询语法之后，发送的 post 请求又变成下图显示的样子。</p>
<p data-nodeid="8185" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/55/Cip5yF_tdN6AQ3l9AAPPn8brHa8263.png" alt="Drawing 3.png" data-nodeid="8188"></p>

<p data-nodeid="3248">日志比较长，你有兴趣的话，可以按照我的 DEMO 和开启日志的方法，自己去分析体会一下。</p>
<p data-nodeid="3249">下面来说说 Spring Data ElasticSearch 中关键的几个类。</p>
<h3 data-nodeid="3250">Spring Data ElasticSearch 关键的类</h3>
<p data-nodeid="3251">通过上面的案例我们可以知道，Spring Data ElasticSearch 的用法其实非常简单，并且我们通过日志也可以看到，底层实现是基于 http 请求，来操作 Elasticsearch 的 server 中的 api 进行的。</p>
<p data-nodeid="3252">那么我们简单看一下这一框架还给我们提供了哪些 ElasticSearch 的操作方法。和分析 Spring Data JPA 一样，看一下 Repository 的所有子类，如下图所示。</p>
<p data-nodeid="8937" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/55/Cip5yF_tdOWAN1p8AAKW4zuYBgc483.png" alt="Drawing 4.png" data-nodeid="8940"></p>

<p data-nodeid="3254">从图中可以看得出来，ElasticsearchRepository 是默认的 Repository 的实现类，我们如果继续往下面看源码的话，就可以看到里面进行了很多 ES 的 Http Client 操作。</p>
<p data-nodeid="3255">同时再看一下 Structure 视图，如下所示。</p>
<p data-nodeid="9689" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/57/CgpVE1_tdOyAa_qxAARM3eWQpnQ793.png" alt="Drawing 5.png" data-nodeid="9692"></p>

<p data-nodeid="3257">从这张图可以知道，ElasticsearchRepository 默认给我们提供了 search 和 index 相关的一些操作方法，并且 Spring Data Common 里面的一些公共方法同样适用，这和我们刚才演示的 Defining Method Query 的 JPA 语法同样适用，可以大大减轻操作 ES 的难度，提高了开发的效率，甚至像我们没有演示到的分页、排序、limit 等同样适用。</p>
<p data-nodeid="3258">所以你现在学到了一个“套路”：和 Spring Data JPA 用相同的思路，就可以很快掌握 Spring Data Elasticsearch 的基本用法，及其大概的实现思路。</p>
<p data-nodeid="3259">那么很多时候同一个工程里面既有 JPA 又有 Elasticsearch，又该怎么写呢？</p>
<h3 data-nodeid="3260">ESRepository 和 JPARepository 同时存在</h3>
<p data-nodeid="3261">这个时候应该怎么区分不同的 Repository 用什么呢？</p>
<p data-nodeid="3262">我们假设刚才测试的样例里面，同时有关于 User 信息的 DB 操作，那么看一下我们的项目应该怎么写。</p>
<p data-nodeid="3263"><strong data-nodeid="3484">第一步：我们将对 Elasticsearch 的实体、Repository 和对 JPA 操作的实体、Repository 放到不同的文件里面</strong>，如下图所示。</p>
<p data-nodeid="10441" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/55/Cip5yF_tdPOAVRMHAACTufgK21A436.png" alt="Drawing 6.png" data-nodeid="10444"></p>

<p data-nodeid="3265"><strong data-nodeid="3492">第二步：新增 JpaConfiguration，用来指定 Jpa 相关的 Repository 目录</strong>，完整代码如下。</p>
<pre class="lang-java" data-nodeid="3266"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo.jpa;
<span class="hljs-keyword">import</span> org.springframework.context.annotation.Configuration;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.config.EnableJpaRepositories;
<span class="hljs-comment">//利用@EnableJpaRepositories指定JPA的目录是"com.example.data.es.demo.jpa"</span>
<span class="hljs-meta">@EnableJpaRepositories(basePackages = "com.example.data.es.demo.jpa")</span>
<span class="hljs-meta">@Configuration</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">JpaConfiguration</span> </span>{
}
</code></pre>
<p data-nodeid="3267"><strong data-nodeid="3497">第三步：新增 User 实体，用来操作用户基本信息</strong>。</p>
<pre class="lang-java" data-nodeid="3268"><code data-language="java"><span class="hljs-meta">@Data</span>
<span class="hljs-meta">@Builder</span>
<span class="hljs-meta">@Entity</span>
<span class="hljs-meta">@Table</span>
<span class="hljs-meta">@NoArgsConstructor</span>
<span class="hljs-meta">@AllArgsConstructor</span>
<span class="hljs-meta">@ToString</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">User</span> </span>{
    <span class="hljs-meta">@Id</span>
    <span class="hljs-meta">@GeneratedValue(strategy= GenerationType.AUTO)</span>
    <span class="hljs-keyword">private</span> Long id;
    <span class="hljs-keyword">private</span> String name;
    <span class="hljs-keyword">private</span> String email;
}
</code></pre>
<p data-nodeid="3269"><strong data-nodeid="3502">第四步：新增 UserRepository，用来进行 DB 操作</strong>。</p>
<pre class="lang-java" data-nodeid="3270"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo.jpa;
<span class="hljs-keyword">import</span> org.springframework.data.jpa.repository.JpaRepository;
<span class="hljs-comment">//对User的DB操作，我们直接继承JpaRepository</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">UserRepository</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">JpaRepository</span>&lt;<span class="hljs-title">User</span>,<span class="hljs-title">Long</span>&gt; </span>{
}
</code></pre>
<p data-nodeid="3271"><strong data-nodeid="3507">第五步：写测试用例进行测试</strong>。</p>
<pre class="lang-java" data-nodeid="3272"><code data-language="java"><span class="hljs-keyword">package</span> com.example.data.es.demo;
<span class="hljs-keyword">import</span> com.example.data.es.demo.jpa.User;
<span class="hljs-keyword">import</span> com.example.data.es.demo.jpa.UserRepository;
<span class="hljs-keyword">import</span> org.junit.jupiter.api.Test;
<span class="hljs-keyword">import</span> org.springframework.beans.factory.annotation.Autowired;
<span class="hljs-keyword">import</span> org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
<span class="hljs-keyword">import</span> java.util.List;
<span class="hljs-comment">//利用@DataJpaTest完成集成测试</span>
<span class="hljs-meta">@DataJpaTest</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRepositoryTest</span> </span>{
    <span class="hljs-meta">@Autowired</span>
    <span class="hljs-keyword">private</span> UserRepository userRepository;
    <span class="hljs-meta">@Test</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">void</span> <span class="hljs-title">testJpa</span><span class="hljs-params">()</span> </span>{
<span class="hljs-comment">//往数据库里面保存一条数据，并且打印一下</span>
                userRepository.save(User.builder().id(<span class="hljs-number">1L</span>).name(<span class="hljs-string">"jkdb"</span>).email(<span class="hljs-string">"jack@email.com"</span>).build());
        List&lt;User&gt; users = userRepository.findAll();
        users.forEach(user -&gt; {
            System.out.println(user);
        });
    }
}
</code></pre>
<p data-nodeid="11935">这个时候，我们的测试用例就变成了如下图所示的结构。</p>
<p data-nodeid="11936" class=""><img src="https://s0.lgstatic.com/i/image2/M01/04/57/CgpVE1_tdQKAAa7zAABNF77hZ_A879.png" alt="Drawing 7.png" data-nodeid="11940"></p>


<p data-nodeid="3274">那么现在我们知道了，JPA 和 Elasticsearch 同时存在，和启动项目是一样的效果，这里就不写 Controller 了。</p>
<p data-nodeid="3275">我们再整体运行一下这三个测试用例，进行完整的测试，就可以看到如下结果。</p>
<p data-nodeid="12689" class="">1.ElasticSearchRepositoryTest 执行的时候，通过日志可以看到这是对 ES 进行的操作，如下图所示。</p>

<p data-nodeid="14177" class=""><img src="https://s0.lgstatic.com/i/image/M00/8C/74/Ciqc1F_tdRWABN3HAASErifQeiw553.png" alt="Drawing 8.png" data-nodeid="14180"></p>

<p data-nodeid="13435" class="">2.UserRepositoryTest 执行的时候，通过日志我们可以看出来这是对 DB 进行的操作，所以谁也不影响谁，如下图所示。</p>

<p data-nodeid="14921" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/8C/74/Ciqc1F_tdRyAe_oeAAMw4yV6H4o471.png" alt="Drawing 9.png" data-nodeid="14924"></p>

<p data-nodeid="3284">通过上面的例子我们可以知道，Spring Data 对 JPA 等 SQL 型的传统数据库的支持是非常好的，同时对 NoSQL 型的非关系类数据库的支持也比较友好，大大降低了操作不同数据源的难度，可以有效提升我们的开发效率。</p>
<h3 data-nodeid="3285">总结</h3>
<p data-nodeid="3286">这一讲内容到这里就结束了，我通过“入门型”的 Spring Data Elasticsearch 样例展示，让你体会了 Spring Data 对数据操作的抽象封装的强大之处。</p>
<p data-nodeid="3287">如果你研究好了这部分内容，其实 Spring Data 中的其他系列也是可以通用的。这里我只是期望起到抛砖引玉的效果，希望你能更好地掌握 Spring Data 的精髓，并且能深入理解 JPA。</p>
<p data-nodeid="3288">至此，我们的专栏也将告一段落，不知道这 32 讲的内容对你是否有帮助，我希望你可以回过头好好回顾，更好地掌握 Spring Data JPA。</p>
<p data-nodeid="3289">如果对此有不懂的地方，也欢迎你在评论区留言，我们一起探讨，一起在这条路上不断精进。再见！</p>
<blockquote data-nodeid="3290">
<p data-nodeid="3291">点击下方链接查看源码（不定时更新）<br>
<a href="https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa" data-nodeid="3533">https://github.com/zhangzhenhuajack/spring-boot-guide/tree/master/spring-data/spring-data-jpa</a></p>
</blockquote>

---

### 精选评论

##### *中：
> 终于追完了，我认为这是拉勾最有价值的课。但是能讲点spring-data-common设计思路那最好了😀

