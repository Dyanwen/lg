<p data-nodeid="35014" class="">如今，虽然越来越多的开发人员开始意识到安全问题，但不幸的是，从应用程序设计和开发之初就充分考虑安全性问题并不是一种常见的做法。这种态度应该改变，每个参与软件系统开发的团队或个人都必须从一开始就学会考虑安全性。</p>
<p data-nodeid="35015">Spring Security 是 Spring 家族的重要组成成员，也是业界领先的一款应用程序开发框架，提供了多项核心功能，帮忙我们构建完整的安全解决方案。本专栏作为针对 Spring Security 的一门系统化课程，在课程的最后，我想和你一起回顾和总结一下 Spring Security 核心功能，分享一些我在写作过程中的一些思考和心得。</p>
<h2 data-nodeid="35271" class="te-preview-highlight">学习 Spring Security 的意义</h2>


<p data-nodeid="35017">在 Java 领域中，Spring Security 是应用非常广泛的一个开发框架，也是 Spring 家族中历史比较悠久的一个框架。Spring Security 同样是 Spring Cloud 等综合性开发框架的底层基础框架之一，功能完备且强大，因此在 Spring 家族的其他框架中应用十分广泛。</p>
<p data-nodeid="35018">另一方面，随着 Spring Boot 的大规模流行，Spring Security 也迎来了全新的发展时期。基于 Spring Boot 的自动配置原理，以往在传统 Spring 应用程序中集成和配置 Spring Security 框架的复杂过程将成为历史。在日常开发过程中，Spring Security 与 Spring Boot 等框架可以无缝集成，为构建完整的安全性解决方案提供保障。</p>
<p data-nodeid="35019">Spring Security 内置了很多强大的功能，针对整个框架，我们可以从以下几个方面对其进行分析和学习：</p>
<ul data-nodeid="35020">
<li data-nodeid="35021">
<p data-nodeid="35022">Spring Security 的体系结构和基本组件，以及如何使用它来保护应用程序；</p>
</li>
<li data-nodeid="35023">
<p data-nodeid="35024">使用 Spring Security 进行身份验证和授权，以及如何将它们应用于面向生产的应用程序；</p>
</li>
<li data-nodeid="35025">
<p data-nodeid="35026">如何在应用程序的不同层中使用 Spring Security；</p>
</li>
<li data-nodeid="35027">
<p data-nodeid="35028">在应用程序中使用不同的安全配置方式和最佳实践；</p>
</li>
<li data-nodeid="35029">
<p data-nodeid="35030">将 Spring Security 用于响应式应用程序；</p>
</li>
<li data-nodeid="35031">
<p data-nodeid="35032">对安全性解决方案进行测试。</p>
</li>
</ul>
<p data-nodeid="35033">但是，基于我自己的学习过程，以及从周围开发人员接收到的信息，我们在学习如何正确地使用 Spring Security 来保护应用程序免受常见漏洞的攻击时，或多或少都会遇到困难。当然，我们可以在网上找到有关 Spring Security 的所有细节。但是如果你希望在使用框架时花费最少的精力，那么就需要将相关知识按正确、合理的顺序放在一起进行学习，这通常需要大量的时间和经验。因此我设计这门课程的初衷，就是为了帮助你节省学习时间，提高学习效率。</p>
<p data-nodeid="35034">此外，不完整的知识可能导致你设计并实现了难以维护的解决方案，甚至可能暴露安全漏洞。很多时候，当我们去 Review 这些问题时，会发现 Spring Security 的使用方式本身可能就是不合理的。而且，在许多情况下，主要原因还是开发人员对“如何使用 Spring Security”缺乏必要的了解。Spring Security 中的一些功能看上去比较简单，但用起来却经常会因为一些细小的配置，导致整个功能无法使用。即使发现了问题，也不太不容易找到原因。</p>
<p data-nodeid="35035">因此，我决定设计一个系统化的专栏，帮助所有使用 Spring 框架的开发人员理解“如何正确使用 Spring Security”。这门课程应该是一个资源，以帮助开发人员逐步了解 Spring Security 框架，希望能为你带来价值，避免在应用程序中引入所有可能的安全漏洞。</p>
<p data-nodeid="35036">下面我们再来回顾一下这门课程具体讲了哪些内容，这些内容又具备哪些特色呢？</p>
<h2 data-nodeid="35037">Spring Security 这门课有什么特色？</h2>
<p data-nodeid="35038">在设计这门课程时，我关注的是将框架提供的各项功能进行合理的组织，并详细介绍它们的应用方式。<strong data-nodeid="35076">课程组织上按照“基础功能篇→高级主题篇→OAuth2 与微服务篇 → 框架扩展篇”的主线来展开内容，呈递进关系。这是本课程的一大特色</strong>。我们将 Spring Security 的各项功能按照基础和高级等不同维度进行划分，由浅入深进行讲解。</p>
<ul data-nodeid="35039">
<li data-nodeid="35040">
<p data-nodeid="35041">基础功能篇中，我们介绍 Spring Security 的一些基础性功能，包括认证、授权和加密；</p>
</li>
<li data-nodeid="35042">
<p data-nodeid="35043">高级主题篇的功能面向特定需求，可以用于构建比较复杂的应用场景，包括过滤器、CSRF 保护、跨域 CORS，以及针对非 Web 应用程序的全局方法安全机制；</p>
</li>
<li data-nodeid="35044">
<p data-nodeid="35045">而 OAuth2 与微服务篇的内容关注微服务开发框架 Spring Cloud 与 Spring Security 之间的整合，我们对 OAuth2 协议和 JWT 进行了全面的展开，并使用这些技术体系构建了安全的微服务系统，以及单点登录系统。</p>
</li>
<li data-nodeid="35046">
<p data-nodeid="35047">最后，在框架扩展篇中，我们对 Spring Security 框架在应用上的一些扩展进行讨论，包括在 Spring Security 中引入全新的响应式编程技术，以及如何对应用程序安全性进行测试的系统方法。</p>
</li>
</ul>
<p data-nodeid="35048"><strong data-nodeid="35085">课程的第二大特色在于案例驱动</strong>。整个专栏分别在基础功能篇、高级主题篇和微服务安全篇中结合本篇内容提供一个完整的案例，分别介绍 Spring Security 的基础认证授权功能、过滤器功能、基于 OAuth2 协议的单点登录和微服务访问授权体系的实战技巧。我们在案例中使用到的很多示例代码都可以直接使用在面向生产的应用程序中。</p>
<p data-nodeid="35049"><strong data-nodeid="35090">第三个特色是技术创新</strong>。随着 Spring 5 的发布涌现出了响应式编程这种新型技术体系，新版本的 Spring Security 中也提供了对响应式编程的全面支持。本课程对响应式 Spring Security 也做了介绍，这部分内容应该在目前 Spring Security 相关资料中还是首创。</p>
<p data-nodeid="35050"><strong data-nodeid="35095">第四个特色是深度和广度并重</strong>。从内容广度上，我们对 Spring Security 框架可以说是面面俱到，相关知识点娓娓道来。而在内容深度上，我们对框架最核心的认证和授权机制进行底层原理的分析，让你知其然更值其所以然。Spring Security 内置了很多可扩展性组件，通过对框架底层实现机制的理解和把握，可以帮助我们更好的实现扩展性。</p>
<h3 data-nodeid="35051">写在最后</h3>
<p data-nodeid="35052">整个课程从平时的积累，到酝酿的启动再到上线经历了小半年的时间，伴随着这个过程，我把 Spring Security 的部分源代码系统地梳理了一遍，并对内部的设计思想和实现原理也做了一些提炼和总结。</p>
<p data-nodeid="35053" class="">总体而言，Spring Security 是一款代码质量非常高的开源框架，其中关于对用户认证和访问授权模型的抽象、内置的过滤器机制、全局方法安全机制、OAuth2 协议，以及响应式编程支持等诸多功能都给我留下了深刻的印象，使我受益良多。相信对于坚持学习到今天的你而言也是一样。</p>

---

### 精选评论

##### **威：
> 谢谢老师！！

##### **林：
> 厉害，比较系统

