<p data-nodeid="701" class="">上一讲我们学习了 Ribbon 与 RestTemplate 两个组件。Ribbon 提供了客户端负载均衡，而 RestTemplate 则封装了 HTTP 的通讯，简化了发送请求的过程。两者相辅相成构建了服务间的高可用通信。</p>
<p data-nodeid="702">不过在使用后，你也应该会发现 RestTemplate，它只是对 HTTP 的简单封装，像 URL、请求参数、请求头、请求体这些细节都需要我们自己处理，如此底层的操作都暴露出来这肯定不利于项目团队间协作，因此就需要一种封装度更高、使用更简单的技术屏蔽通信底层复杂度。好在 Spring Cloud 团队提供了 OpenFeign 技术，大幅简化了服务间高可用通信处理过程。本讲将主要介绍三部分：</p>
<ul data-nodeid="703">
<li data-nodeid="704">
<p data-nodeid="705">介绍 Feign 与 OpenFeign；</p>
</li>
<li data-nodeid="706">
<p data-nodeid="707">讲解 OpenFeign 的使用办法；</p>
</li>
<li data-nodeid="708">
<p data-nodeid="709">讲解生产环境 OpenFeign 的配置优化。</p>
</li>
</ul>
<h3 data-nodeid="710">Feign 与 OpenFeign</h3>
<p data-nodeid="711">Spring Cloud OpenFeign 并不是独立的技术。它底层基于 Netflix Feign，Netflix Feign 是 Netflix 设计的开源的声明式 WebService 客户端，用于简化服务间通信。Netflix Feign 采用“<strong data-nodeid="804">接口+注解</strong>”的方式开发，通过模仿 RPC 的客户端与服务器模式（CS），采用接口方式开发来屏蔽网络通信的细节。OpenFeign 则是在 Netflix Feign 的基础上进行封装，结合原有 Spring MVC 的注解，对 Spring Cloud 微服务通信提供了良好的支持。使用 OpenFeign 开发的方式与开发 Spring MVC Controller 颇为相似。下面我们通过代码说明 OpenFeign 的各种开发技巧。</p>
<h3 data-nodeid="712">OpenFeign 的使用办法</h3>
<p data-nodeid="713">为了便于理解，我们模拟实际案例进行说明。假设某电商平台日常订单业务中，为保证每一笔订单不会超卖，在创建订单前订单服务（order-service）首先去仓储服务（warehouse-service）检查对应商品 skuId（品类编号）的库存数量是否足够，库存充足创建订单，不存不足 App 前端提示“库存不足”。</p>
<p data-nodeid="873" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M00/29/69/CioPOWBhSD6AVJ1jAADZb3zB8iU341.png" alt="图片1.png" data-nodeid="877"></p>
<div data-nodeid="874"><p style="text-align:center">订单与仓储服务处理流程</p></div>


<p data-nodeid="716">在这个业务中，订单服务依赖于仓储服务，那仓储服务就是服务提供者，订单服务是服务消费者。下面我们通过代码还原这个场景。</p>
<p data-nodeid="717">首先，先创建仓储服务（warehouse-service），仓储服务作为提供者就是标准微服务，使用 Spring Boot 开发。</p>
<p data-nodeid="718">第一步，利用 Spring Initializr 向导创建 warehouse-service 工程。确保在 pom.xml 引入以下依赖。</p>
<pre class="lang-xml" data-nodeid="719"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="720">第二步，编辑 application.yml 新增 Nacos 通信配置。</p>
<pre class="lang-yaml" data-nodeid="721"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">warehouse-service</span> <span class="hljs-comment">#应用/微服务名字</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">nacos:</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.102</span><span class="hljs-string">:8848</span> <span class="hljs-comment">#nacos服务器地址</span>
        <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span> <span class="hljs-comment">#用户名密码</span>
        <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
<span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="722">第三步，创建 Stock 库存类，用于保存库存数据。</p>
<pre class="lang-java" data-nodeid="723"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.warehouseservice.dto;
<span class="hljs-comment">//库存商品对象</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Stock</span> </span>{
    <span class="hljs-keyword">private</span> Long skuId; <span class="hljs-comment">//商品品类编号</span>
    <span class="hljs-keyword">private</span> String title; <span class="hljs-comment">//商品与品类名称</span>
    <span class="hljs-keyword">private</span> Integer quantity; <span class="hljs-comment">//库存数量</span>
    <span class="hljs-keyword">private</span> String unit; <span class="hljs-comment">//单位</span>
    <span class="hljs-keyword">private</span> String description; <span class="hljs-comment">//描述信息</span>
    <span class="hljs-comment">//带参构造函数</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-title">Stock</span><span class="hljs-params">(Long skuId, String title, Integer quantity, String unit)</span> </span>{
        <span class="hljs-keyword">this</span>.skuId = skuId;
        <span class="hljs-keyword">this</span>.title = title;
        <span class="hljs-keyword">this</span>.quantity = quantity;
        <span class="hljs-keyword">this</span>.unit = unit;
    }
    <span class="hljs-comment">//getter and setter省略...</span>
}
</code></pre>
<p data-nodeid="724">第四步，创建仓储服务控制器 WarehouseController，通过 getStock() 方法传入 skuId 编号则返回具体商品库存数量，代码中模拟 skuId 编号为 1101 的“紫色 128G iPhone 11”库存 32 台，而编号 1102 的“白色 256G iPhone 11”已没有库存。</p>
<pre class="lang-java" data-nodeid="725"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.warehouseservice.controller;
<span class="hljs-comment">//省略 import 部分</span>
<span class="hljs-comment">//仓储服务控制器</span>
<span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">WarehouseController</span> </span>{
    <span class="hljs-comment">/**
     * 查询对应 skuId 的库存状况
     * <span class="hljs-doctag">@param</span> skuId skuId
     * <span class="hljs-doctag">@return</span> Stock 库存对象
     */</span>
    <span class="hljs-meta">@GetMapping("/stock")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Stock <span class="hljs-title">getStock</span><span class="hljs-params">(Long skuId)</span></span>{
        Map result = <span class="hljs-keyword">new</span> HashMap();
        Stock stock = <span class="hljs-keyword">null</span>;
        <span class="hljs-keyword">if</span>(skuId == <span class="hljs-number">1101l</span>){
            <span class="hljs-comment">//模拟有库存商品</span>
            stock = <span class="hljs-keyword">new</span> Stock(<span class="hljs-number">1101l</span>, <span class="hljs-string">"Apple iPhone 11 128GB 紫色"</span>, <span class="hljs-number">32</span>, <span class="hljs-string">"台"</span>);
            stock.setDescription(<span class="hljs-string">"Apple 11 紫色版对应商品描述"</span>);
        }<span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span>(skuId == <span class="hljs-number">1102l</span>){
            <span class="hljs-comment">//模拟无库存商品</span>
            stock = <span class="hljs-keyword">new</span> Stock(<span class="hljs-number">1101l</span>, <span class="hljs-string">"Apple iPhone 11 256GB 白色"</span>, <span class="hljs-number">0</span>, <span class="hljs-string">"台"</span>);
            stock.setDescription(<span class="hljs-string">"Apple 11 白色版对应商品描述"</span>);
        }<span class="hljs-keyword">else</span>{
            <span class="hljs-comment">//演示案例，暂不考虑无对应 skuId 的情况</span>
        }
        <span class="hljs-keyword">return</span> stock;
    }
}
</code></pre>
<p data-nodeid="726">可以看到 WarehouseController 就是普通的 Spring MVC 控制器，对外暴露了 stock 接口，当应用启动后，查看 Nacos 服务列表，已出现 warehouse-service 实例。</p>
<p data-nodeid="727"><img src="https://s0.lgstatic.com/i/image6/M00/1E/2B/Cgp9HWBQbJSAduJQAAI73U_ruyQ393.png" alt="Drawing 3.png" data-nodeid="819"></p>
<div data-nodeid="728"><p style="text-align:center">Nacos 注册成功</p></div>
<p data-nodeid="729">访问下面的 URL 可看到 Stock 对象 JSON 序列化数据。</p>
<pre class="lang-java" data-nodeid="730"><code data-language="java">http:<span class="hljs-comment">//192.168.31.111/stock?skuId=1101</span>
{
  skuId: <span class="hljs-number">1101</span>,
  title: <span class="hljs-string">"Apple iPhone 11 128GB 紫色"</span>,
  quantity : <span class="hljs-number">32</span>,
  unit: <span class="hljs-string">"台"</span>,
  description:<span class="hljs-string">"Apple 11 紫色版对应商品描述"</span>
}
</code></pre>
<p data-nodeid="731">至此，服务提供者 warehouse-service 示例代码已开发完毕。下面我们要开发服务消费者 order-service。</p>
<p data-nodeid="732">第一步，确保利用 Spring Initializr 创建 order-service 工程，确保 pom.xml 引入以下 3 个依赖。</p>
<pre class="lang-xml" data-nodeid="733"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.boot<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-boot-starter-web<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>com.alibaba.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-alibaba-nacos-discovery<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>org.springframework.cloud<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>spring-cloud-starter-openfeign<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>2.2.5.RELEASE<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="734">这里关键在于服务消费者依赖了 spring-cloud-starter-openfeign，在 Spring Boot 工程会自动引入 Spring Cloud OpenFeign 与 Netflix Feign 的 Jar 包。这里有个重要细节，当我们引入 OpenFeign 的时候，在 Maven 依赖中会出现 netflix-ribbon 负载均衡器的身影。</p>
<p data-nodeid="735"><img src="https://s0.lgstatic.com/i/image6/M00/1E/28/CioPOWBQbKGAVNY5AAOIqVeLiUM338.png" alt="Drawing 5.png" data-nodeid="826"></p>
<div data-nodeid="736"><p style="text-align:center">OpenFeign 内置 Ribbon 实现负载均衡</p></div>
<p data-nodeid="737">没错，OpenFeign 为了保证通信高可用，底层也是采用 Ribbon 实现负载均衡，其原理与 Ribbon+RestTemplate 完全相同，只不过相较 RestTemplate，OpenFeign 封装度更高罢了。</p>
<p data-nodeid="738">第二步，启用 OpenFeign 需要在应用入口 OrderServiceApplication 增加 @EnableFeignClients 注解，其含义为通知 Spring 启用 OpenFeign 声明式通信。</p>
<pre class="lang-java" data-nodeid="739"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.orderservice;
<span class="hljs-keyword">import</span> org.springframework.boot.SpringApplication;
<span class="hljs-keyword">import</span> org.springframework.boot.autoconfigure.SpringBootApplication;
<span class="hljs-keyword">import</span> org.springframework.cloud.openfeign.EnableFeignClients;
<span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableFeignClients</span> <span class="hljs-comment">//启用OpenFeign</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderServiceApplication</span> </span>{
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
</code></pre>
<p data-nodeid="740">第三步，默认 OpenFeign 并不需要任何配置，在 application.yml 配置好 Nacos 通信即可。</p>
<pre class="lang-yaml" data-nodeid="741"><code data-language="yaml"><span class="hljs-attr">spring:</span>
  <span class="hljs-attr">application:</span>
    <span class="hljs-attr">name:</span> <span class="hljs-string">order-service</span>
  <span class="hljs-attr">cloud:</span>
    <span class="hljs-attr">nacos:</span>
      <span class="hljs-attr">discovery:</span>
        <span class="hljs-attr">server-addr:</span> <span class="hljs-number">192.168</span><span class="hljs-number">.31</span><span class="hljs-number">.102</span><span class="hljs-string">:8848</span>
        <span class="hljs-attr">username:</span> <span class="hljs-string">nacos</span>
        <span class="hljs-attr">password:</span> <span class="hljs-string">nacos</span>
<span class="hljs-attr">server:</span>
  <span class="hljs-attr">port:</span> <span class="hljs-number">80</span>
</code></pre>
<p data-nodeid="742">第四步，最重要的地方来了，创建OpenFeign的通信接口与响应对象，这里先给出完整代码。</p>
<pre class="lang-java" data-nodeid="743"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.orderservice.feignclient;
<span class="hljs-keyword">import</span> com.lagou.orderservice.dto.Stock;
<span class="hljs-keyword">import</span> org.springframework.cloud.openfeign.FeignClient;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.GetMapping;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.RequestParam;
<span class="hljs-meta">@FeignClient("warehouse-service")</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">interface</span> <span class="hljs-title">WarehouseServiceFeignClient</span> </span>{
    <span class="hljs-meta">@GetMapping("/stock")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Stock <span class="hljs-title">getStock</span><span class="hljs-params">(<span class="hljs-meta">@RequestParam("skuId")</span> Long skuId)</span></span>;
}
</code></pre>
<p data-nodeid="744">在 order-service 工程下，创建一个 feignclient 包用于保存通信接口。OpenFeign 通过“接口+注解”形式描述数据传输逻辑，并不需要程序员编写具体实现代码便能实现服务间高可用通信，下面我们来学习这段代码：</p>
<ul data-nodeid="745">
<li data-nodeid="746">
<p data-nodeid="747">@FeignClient 注解说明当前接口为 OpenFeign 通信客户端，参数值 warehouse-service 为服务提供者 ID，这一项必须与 Nacos 注册 ID 保持一致。在 OpenFeign 发送请求前会自动在 Nacos 查询 warehouse-service 所有可用实例信息，再通过内置的 Ribbon 负载均衡选择一个实例发起 RESTful 请求，进而保证通信高可用。</p>
</li>
<li data-nodeid="748">
<p data-nodeid="749">声明的方法结构，接口中定义的方法通常与服务提供者的方法定义保持一致。这里有个非常重要的细节：用于接收数据的 Stock 对象并不强制要求与提供者端 Stock 对象完全相同，消费者端的 Stock 类可以根据业务需要删减属性，但属性必须要与提供者响应的 JSON 属性保持一致。距离说明，我们在代码发现消费者端 Stock 的包名与代码与提供者都不尽相同，而且因为消费者不需要 description 属性便将其删除，其余属性只要保证与服务提供者响应 JSON 保持一致，在 OpenFeign 获取响应后便根据 JSON 属性名自动反序列化到 Stock 对象中。</p>
</li>
</ul>
<pre class="lang-json" data-nodeid="750"><code data-language="json">#服务提供者返回的响应
{
    skuId:&nbsp;1101,
    title:&nbsp;"Apple iPhone 11 128GB 紫色",
    quantity:&nbsp;32,
    unit:&nbsp;"台",
    description:&nbsp;"Apple 11 紫色版对应商品描述"
}
</code></pre>
<pre class="lang-java" data-nodeid="751"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.orderservice.dto;
<span class="hljs-comment">//消费者端接收响应Stock对象</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Stock</span> </span>{
    <span class="hljs-keyword">private</span> Long skuId; <span class="hljs-comment">//商品品类编号</span>
    <span class="hljs-keyword">private</span> String title; <span class="hljs-comment">//商品与品类名称</span>
    <span class="hljs-keyword">private</span> Integer quantity; <span class="hljs-comment">//库存数量</span>
    <span class="hljs-keyword">private</span> String unit; <span class="hljs-comment">//单位</span>
    <span class="hljs-meta">@Override</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> String <span class="hljs-title">toString</span><span class="hljs-params">()</span> </span>{
        <span class="hljs-keyword">return</span> <span class="hljs-string">"Stock{"</span> +
                <span class="hljs-string">"skuId="</span> + skuId +
                <span class="hljs-string">", title='"</span> + title + <span class="hljs-string">'\''</span> +
                <span class="hljs-string">", quantity="</span> + quantity +
                <span class="hljs-string">", unit='"</span> + unit + <span class="hljs-string">'\''</span> +
                <span class="hljs-string">'}'</span>;
    }
    <span class="hljs-comment">//getter与setter省略</span>
}
</code></pre>
<ul data-nodeid="752">
<li data-nodeid="753">
<p data-nodeid="754">@GetMapping/@PostMapping，以前我们在编写 Spring MVC 控制器时经常使用 @GetMapping 或者@ PostMapping 声明映射方法的请求类型。虽然 OpenFeign 也使用了这些注解，但含义完全不同。在消费者端这些注解的含义是：<strong data-nodeid="839">OpenFeign 向服务提供者 warehouse-service 的 stock 接口发起 Get 请求</strong>。简单总结下，如果在服务提供者书写 @GetMapping 是说明 Controller 接收数据的请求类型必须是 Get，而写在消费者端接口中则说明 OpenFeign 采用 Get 请求发送数据，大多数情况下消费者发送的请求类型、URI 与提供者定义要保持一致。</p>
</li>
<li data-nodeid="755">
<p data-nodeid="756">@RequestParam，该注解说明方法参数与请求参数之间的映射关系。举例说明，当调用接口的 getStock() 方法时 skuId 参数值为 1101，那实际通信时 OpenFeign 发送的 Get 请求格式就是：</p>
</li>
</ul>
<pre class="lang-java" data-nodeid="757"><code data-language="java">http:<span class="hljs-comment">//warehouse-service可用实例 ip:端口/stock?skuId=1101</span>
</code></pre>
<p data-nodeid="758">介绍每一个细节后，我用自然语言完整描述处理逻辑：</p>
<p data-nodeid="759">1.在第一次访问 WarehouseServiceFeignClient 接口时，Spring 自动生成接口的实现类并实例化对象。</p>
<p data-nodeid="760">2.当调用 getStock() 方法时，Ribbon 获取 warehouse-service 可用实例信息，根据负载均衡策略选择合适实例。</p>
<p data-nodeid="761">3.OpenFeign 根据方法上注解描述的映射关系生成完整的 URL 并发送 HTTP 请求，如果请求方法是 @PostMapping，则参数会附加在请求体中进行发送。</p>
<pre class="lang-java" data-nodeid="762"><code data-language="java">http:<span class="hljs-comment">//warehouse-service 可用实例 ip:端口/stock?skuId=xxx</span>
</code></pre>
<p data-nodeid="763">4.warehouse-service 处理完毕返回 JSON 数据，消费者端 OpenFeign 接收 JSON 的同时反序列化到 Stock 对象，并将该对象返回。</p>
<p data-nodeid="764">到这里我们花了较大的篇幅介绍 OpenFeign 的执行过程，那该怎么使用呢？这就是第五步的事情了。</p>
<p data-nodeid="765">第五步，在消费者 Controller 中对 FeignClient 接口进行注入，像调用本地方法一样完成业务逻辑。</p>
<pre class="lang-java" data-nodeid="766"><code data-language="java"><span class="hljs-keyword">package</span> com.lagou.orderservice.controller;
<span class="hljs-keyword">import</span> com.lagou.orderservice.dto.Stock;
<span class="hljs-keyword">import</span> com.lagou.orderservice.feignclient.WarehouseServiceFeignClient;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.GetMapping;
<span class="hljs-keyword">import</span> org.springframework.web.bind.annotation.RestController;
<span class="hljs-keyword">import</span> javax.annotation.Resource;
<span class="hljs-keyword">import</span> java.util.LinkedHashMap;
<span class="hljs-keyword">import</span> java.util.Map;
<span class="hljs-meta">@RestController</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderController</span> </span>{
    <span class="hljs-comment">//利用@Resource将IOC容器中自动实例化的实现类对象进行注入</span>
    <span class="hljs-meta">@Resource</span>
    <span class="hljs-keyword">private</span> WarehouseServiceFeignClient warehouseServiceFeignClient;
    <span class="hljs-comment">/**
     * 创建订单业务逻辑
     * <span class="hljs-doctag">@param</span> skuId 商品类别编号
     * <span class="hljs-doctag">@param</span> salesQuantity 销售数量
     * <span class="hljs-doctag">@return</span>
     */</span>
    <span class="hljs-meta">@GetMapping("/create_order")</span>
    <span class="hljs-function"><span class="hljs-keyword">public</span> Map <span class="hljs-title">createOrder</span><span class="hljs-params">(Long skuId , Long salesQuantity)</span></span>{
        Map result = <span class="hljs-keyword">new</span> LinkedHashMap();
        <span class="hljs-comment">//查询商品库存，像调用本地方法一样完成业务逻辑。</span>
        Stock stock = warehouseServiceFeignClient.getStock(skuId);
        System.out.println(stock);
        <span class="hljs-keyword">if</span>(salesQuantity &lt;= stock.getQuantity()){
            <span class="hljs-comment">//创建订单相关代码，此处省略</span>
            <span class="hljs-comment">//CODE=SUCCESS代表订单创建成功</span>
            result.put(<span class="hljs-string">"code"</span> , <span class="hljs-string">"SUCCESS"</span>);
            result.put(<span class="hljs-string">"skuId"</span>, skuId);
            result.put(<span class="hljs-string">"message"</span>, <span class="hljs-string">"订单创建成功"</span>);
        }<span class="hljs-keyword">else</span>{
            <span class="hljs-comment">//code=NOT_ENOUGN_STOCK代表库存不足</span>
            result.put(<span class="hljs-string">"code"</span>, <span class="hljs-string">"NOT_ENOUGH_STOCK"</span>);
            result.put(<span class="hljs-string">"skuId"</span>, skuId);
            result.put(<span class="hljs-string">"message"</span>, <span class="hljs-string">"商品库存数量不足"</span>);
        }
        <span class="hljs-keyword">return</span> result;
    }
}
</code></pre>
<p data-nodeid="767">启动后分别传入不同 skuId 与销售数量。可以看到 1101 商品库存充足订单创建成功，1102 商品因为没有库存导致无法创建订单。</p>
<pre class="lang-javascript" data-nodeid="768"><code data-language="javascript">http:<span class="hljs-comment">//192.168.1.120/create_order?skuId=1101&amp;salesQuantity=1</span>
{
<span class="hljs-attr">code</span>: <span class="hljs-string">"SUCCESS"</span>, 
<span class="hljs-attr">skuId</span>: <span class="hljs-number">1101</span>, 
<span class="hljs-attr">message</span>: <span class="hljs-string">"订单创建成功"</span> 
}
</code></pre>
<p data-nodeid="769">创建订单成功消息</p>
<pre class="lang-javascript" data-nodeid="770"><code data-language="javascript">http:<span class="hljs-comment">//192.168.1.120/create_order?skuId=1102&amp;salesQuantity=1</span>
{
<span class="hljs-attr">code</span>: <span class="hljs-string">"NOT_ENOUGH_STOCK"</span>, 
<span class="hljs-attr">skuId</span>: <span class="hljs-number">1102</span>, 
<span class="hljs-attr">message</span>: <span class="hljs-string">"商品库存数量不足"</span> 
}
</code></pre>
<p data-nodeid="771">库存数量不足错误提示</p>
<p data-nodeid="772">到这里已经基于 OpenFeign 实现了服务间通信。但事情还不算完，OpenFeign 默认的配置并不能满足生产环境的要求，下面咱们来讲解在生产环境下 OpenFeign 还需要哪些必要的优化配置。</p>
<h3 data-nodeid="773">生产环境 OpenFeign 的配置事项</h3>
<h4 data-nodeid="774">如何更改 OpenFeign 默认的负载均衡策略</h4>
<p data-nodeid="775">前面提到，在 OpenFeign 使用时默认引用 Ribbon 实现客户端负载均衡。那如何设置 Ribbon 默认的负载均衡策略呢？在 OpenFeign 环境下，配置方式其实与之前 Ribbon+RestTemplate 方案完全相同，只需在 application.yml 中调整微服务通信时使用的负载均衡类即可。</p>
<pre class="lang-yaml" data-nodeid="776"><code data-language="yaml"><span class="hljs-attr">warehouse-service:</span> <span class="hljs-comment">#服务提供者的微服务ID</span>
  <span class="hljs-attr">ribbon:</span>
    <span class="hljs-comment">#设置对应的负载均衡类</span>
    <span class="hljs-attr">NFLoadBalancerRuleClassName:</span> <span class="hljs-string">com.netflix.loadbalancer.RandomRule</span>
</code></pre>
<h4 data-nodeid="777">开启默认的 OpenFeign 数据压缩功能</h4>
<p data-nodeid="778">在 OpenFeign 中，默认并没有开启数据压缩功能。但如果你在服务间单次传递数据超过 1K 字节，强烈推荐开启数据压缩功能。默认 OpenFeign 使用 Gzip 方式压缩数据，对于大文本通常压缩后尺寸只相当于原始数据的 10%~30%，这会极大提高带宽利用率。但有一种情况除外，如果应用属于计算密集型，CPU 负载长期超过 70%，因数据压缩、解压缩都需要 CPU 运算，开启数据压缩功能反而会给 CPU 增加额外负担，导致系统性能降低，这是不可取的。</p>
<pre class="lang-yaml" data-nodeid="779"><code data-language="yaml"><span class="hljs-attr">feign:</span>
  <span class="hljs-attr">compression:</span>
    <span class="hljs-attr">request:</span>
      <span class="hljs-comment"># 开启请求数据的压缩功能</span>
      <span class="hljs-attr">enabled:</span> <span class="hljs-literal">true</span>
      <span class="hljs-comment"># 压缩支持的MIME类型</span>
      <span class="hljs-attr">mime-types:</span> <span class="hljs-string">text/xml,application/xml,</span> <span class="hljs-string">application/json</span>
      <span class="hljs-comment"># 数据压缩下限 1024表示传输数据大于1024 才会进行数据压缩(最小压缩值标准)</span>
      <span class="hljs-attr">min-request-size:</span> <span class="hljs-number">1024</span>
    <span class="hljs-comment"># 开启响应数据的压缩功能</span>
    <span class="hljs-attr">response:</span>
      <span class="hljs-attr">enabled:</span> <span class="hljs-literal">true</span>
</code></pre>
<h4 data-nodeid="780">替换默认通信组件</h4>
<p data-nodeid="781">OpenFeign 默认使用 Java 自带的 URLConnection 对象创建 HTTP 请求，但接入生产时，如果能将底层通信组件更换为 Apache HttpClient、OKHttp 这样的专用通信组件，基于这些组件自带的连接池，可以更好地对 HTTP 连接对象进行重用与管理。作为 OpenFeign 目前默认支持 Apache HttpClient 与 OKHttp 两款产品。我以OKHttp配置方式为例，为你展现配置方法。</p>
<p data-nodeid="782">1.引入 feign-okhttp 依赖包。</p>
<pre class="lang-xml" data-nodeid="783"><code data-language="xml"><span class="hljs-tag">&lt;<span class="hljs-name">dependency</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">groupId</span>&gt;</span>io.github.openfeign<span class="hljs-tag">&lt;/<span class="hljs-name">groupId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">artifactId</span>&gt;</span>feign-okhttp<span class="hljs-tag">&lt;/<span class="hljs-name">artifactId</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">version</span>&gt;</span>11.0<span class="hljs-tag">&lt;/<span class="hljs-name">version</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">dependency</span>&gt;</span>
</code></pre>
<p data-nodeid="784">2.在应用入口，利用 Java Config 形式初始化 OkHttpClient 对象。</p>
<pre class="lang-java" data-nodeid="785"><code data-language="java"><span class="hljs-meta">@SpringBootApplication</span>
<span class="hljs-meta">@EnableFeignClients</span>
<span class="hljs-keyword">public</span> <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">OrderServiceApplication</span> </span>{
    <span class="hljs-comment">//Spring IOC容器初始化时构建okHttpClient对象</span>
    <span class="hljs-meta">@Bean</span>
    <span class="hljs-keyword">public</span> okhttp3.<span class="hljs-function">OkHttpClient <span class="hljs-title">okHttpClient</span><span class="hljs-params">()</span></span>{
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">new</span> okhttp3.OkHttpClient.Builder()
                <span class="hljs-comment">//读取超时时间</span>
                .readTimeout(<span class="hljs-number">10</span>, TimeUnit.SECONDS)
                <span class="hljs-comment">//连接超时时间</span>
                .connectTimeout(<span class="hljs-number">10</span>, TimeUnit.SECONDS)
                <span class="hljs-comment">//写超时时间</span>
                .writeTimeout(<span class="hljs-number">10</span>, TimeUnit.SECONDS)
                <span class="hljs-comment">//设置连接池</span>
                .connectionPool(<span class="hljs-keyword">new</span> ConnectionPool())
                .build();
    }
    <span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">static</span> <span class="hljs-keyword">void</span> <span class="hljs-title">main</span><span class="hljs-params">(String[] args)</span> </span>{
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
</code></pre>
<p data-nodeid="786">3.在 application.yml 中启用 OkHttp。</p>
<pre class="lang-yaml" data-nodeid="787"><code data-language="yaml"><span class="hljs-attr">feign:</span>
  <span class="hljs-attr">okhttp:</span>
    <span class="hljs-attr">enabled:</span> <span class="hljs-literal">true</span>
</code></pre>
<p data-nodeid="788">做到这里，我们已将OpenFeign的默认通信对象从URLConnection调整为OKHttp，至于替换为HttpClient组件的配置思路是基本相同的。如果需要了解OpenFeign更详细的配置选项，可以访问Spring Cloud OpenFeign的官方文档进行学习。<br>
<a href="https://docs.spring.io/spring-cloud-openfeign/docs/2.2.6.RELEASE/reference/html/" data-nodeid="868">https://docs.spring.io/spring-cloud-openfeign/docs/2.2.6.RELEASE/reference/html/</a></p>
<h3 data-nodeid="789">小结与预告</h3>
<p data-nodeid="790">本文我们介绍了三方面知识，开始介绍了 Netflix Feign 与 OpenFeign 的关系，之后讲解了如何使用 OpenFeign 实现服务间通信，最后讲解了 3 个在生产环境优化通信的技巧。</p>
<p data-nodeid="791">这里给你留一道思考题：目前跨进程通信主要有两种方式，基于 HTTP 协议的 RESTful 通信与基于二进制通信的 RPC 远程调用，请你梳理两者的特点与适用场景，可以写在评论区中大家一起探讨。</p>
<p data-nodeid="792" class="">下一节课，我们将介绍 Alibaba 自家的 RPC 框架 Dubbo 是如何与微服务生态整合的，敬请期待。</p>

---

### 精选评论

##### *裕：
> 总结一下open feign优化手段1.开启压缩gzip2.替换默认的Java URL Connection Okhttp3.更改默认的负载均衡策略

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的~~~谢谢兄弟的总结

##### **吉：
> 基于http协议通讯属于轻量级通信，适用rest的微服务通讯，效率略低基于二进制rpc通信，二进制rpc一般使用长连接，效率高，客户端维护比较麻烦

##### **1001：
> RPC场景: 大数据或文件传输?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; RPC是长连接，在需要长时间保持连接的场景比如IM即时通信是好的方案，但RPC、RESTful都不适合传递大数据或大文件，对于大尺寸数据应构建数据仓库或者分布式文件存储系统

##### **4019：
> 上面下单接口检查库存充足就成功了，但是没有扣库存啊？是不是会有业务问题

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 示意性的代码，主要是要讲解OpenFeign远程调用的问题，不要在意这些这些小细节。如果我加了几十行业务代码，文章就会变得非常乏味冗长。

##### **启：
> 开启openFeign的压缩功能后，返回的数据会不会乱码？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 不会的,乱码是因为字符集的问题,压缩不改变字符集

##### **用户8064：
> 老师，你之前提供的代码地址打不开https://github.com/qiyisoft/sca，是这个地址吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://github.com/qiyisoft/sca
再试一下？

##### **伟：
> OpenFeign如何传递pojo呢?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 将Pojo JSON序列化/反序列化就可以了。这个过程OpenFeign已自动实现。

##### *华：
> 个人感觉用http传输小包比较浪费性能。

