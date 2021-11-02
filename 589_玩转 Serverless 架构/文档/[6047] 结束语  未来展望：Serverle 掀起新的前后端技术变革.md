<p data-nodeid="16666">你好，我是老蒋。</p>


<p data-nodeid="16299">从 2020 年 12 月 24 到现在，一晃两个多月的时间过去了，我们的课程也迎来了尾声。在这两个多月的时间里，我们共同学习了 21 讲，从基础概念，到原理，再到开发实践，最后我们还针对几个很经典的场景进行了实战演练和讲解。</p>
<p data-nodeid="16300">要说在这个过程中印象最深的事，就是“熬夜写稿”了。课程上线后，我基本是晚上九、十点到家后写稿，写到凌晨是常有的事。元旦、春节我也几乎是在写稿中度过的。一开始我以为咱们的课写起来很容易，就跟写博客一样，一两个小时就能搞定，后面才发现我想得太简单了，一讲内容从构思、到初稿、到打磨、再到定稿，至少需要一两天。</p>
<p data-nodeid="16301">我记得，为了把概念性的文章讲的通透，有收获感，仅“02 | 概念新知：到底什么是 Serverless?”这一讲就反复优化了 9 次。耗时最长的是 “14 | 系统迁移: 传统应用如何迁移到 Serverless ？”，前后花了近两周时间。不过，“08 | 单元测试：Serverless 应用如何进行单元测试？”写得比较顺利，初稿只花了两三个小时，大概是因为我平时就非常注重单元测试。</p>
<p data-nodeid="16302">虽然磨课的过程比较辛苦，但我觉得这一切都是值得的。</p>
<p data-nodeid="16303">一方面，Serverless 是新兴技术，还不是很成熟，并且正在飞速发展，所以我希望通过这门课，将我在实践 Serverless 过程中总结的经验分享给你，让更多的同学开始了解并使用 Serverless。</p>
<p data-nodeid="16304">另一方面，在这两个月的时间里，我也很多收获，我在评论区看到了很多很经典的问题和思考，也还有一些同学分享了一些新的知识，是你们的积极和努力鼓励着我不断前进，让我明白了学无止境。<strong data-nodeid="16352">在这里，我由衷感谢你的一路陪伴！</strong></p>
<p data-nodeid="16910" class="">今天咱们的课程就全部结束了，相信通过前面的学习，你已经知道了 Serverless 到底是什么、能做什么事、优缺点是什么，也掌握了一些开发 Serverless 的最佳实践。在结束语里，我想再和你聊一聊我经常被问到的问题 <strong data-nodeid="16915">“Serverless 将对前后端开发带来什么影响，未来 Serverless 将会带来哪些机会？”</strong></p>

<p data-nodeid="16306">说起前后端，我们再把时光倒回二十多年前。</p>
<p data-nodeid="16307">那时互联网刚刚兴起，程序员还比较稀缺，也没有前后端的区分。当时互联网上的 Web 应用也很简单，主要是基于 PHP 或 JSP 开发的动态网站，一个程序员就能搞定所有事情。2005 AJAX 技术诞生后，我们才可以在前端通过 JavaScript 代码向服务端动态请求数据，再到后来页面的交互越来越复杂，才渐渐有了前后端的分工：前端工程师主要和浏览器打交道，负责前端页面的开发；后端工程师跟服务器打交道，为前端提供数据接口。</p>
<p data-nodeid="16308">直到现在，我们开发一个业务，基本也是这样的前后端分离的研发模式。这种模式虽然让工程师的职责更加清晰，却降低了沟通效率：前后端需要花费大量时间去协调业务和接口的问题。</p>
<p data-nodeid="16309">同时，前后端分离也不利于个人的发展，因为很多业务的后端就是把现有的一些服务、数据组合起来，变成一个新的数据提供给前端。对后端工程师来说，这样的工作既机械化也缺乏挑战，而前端工程师通常又不能简单地去完成这些工作，也没有足够的时间去学习。</p>
<p data-nodeid="16310">而 Serverless 屏蔽了底层资源和复杂的运维工作，简化了编程模型，不仅补足了前端工程师不能开发服务端的能力，还将使行业对前端工程师的定位发生改变。以往大家都认为前端开发比较简单，写写页面就行，其余工作都交给后端完成。但 Serverless 的出现，大家对前端的诉求就不仅仅是完成页面了，而是要能够负责整个应用的开发。前端工程师除了要与设计师打交道，还要理解整个业务，成为业务负责人。所以我认为 Serverless 带来的第一个技术变革是：<strong data-nodeid="16366">前端工程师将再度回到软件工程师的职责</strong>。而对于后端工程师来说，就可以摆脱简单的 CRUD，去做更底层的技术，迎接更大的挑战。</p>
<p data-nodeid="16311">聊完了 Serverless 对开发者的影响，接下来我们再聊聊 Serverless 技术本身。在 2021 年，除了各大 Serverless 平台持续优化函数冷启动时间、提升 Serverless 开发体验外，我认为最大的变化是<strong data-nodeid="16371">Serverless 开发框架将百花齐放。</strong></p>
<p data-nodeid="16312">现在很多企业使用 Serverless 面临的最大问题之一就是缺乏最佳实践。沉淀和推广实践的最好方法，就是将实践经验集成到开发框架中。当我们要使用 Serverless 来编写大型项目，需要管理成百上千个函数时，一定需要 Serverless 开发框架的协助。传统的开发框架主要是面向技术问题提升开发效率，比如 Spring 通过依赖注入解决对象组装问题， Express.js 通过中间件机制简化请求处理。而  Serverless 开发框架，除了解决技术问题，还需要解决应用开发、部署、监控、运维等问题。 传统框架管理的只是单机资源，但当我们基于 Serverless 使用云服务构建自身业务时，框架需要管理的就不再是单机资源，而是云资源了。</p>
<p data-nodeid="16313">Serverless 开发框架主要包含几个部分：</p>
<ul data-nodeid="16314">
<li data-nodeid="16315">
<p data-nodeid="16316">开发工具，比如命令行工具、编辑器插件、WebIDE 等；</p>
</li>
<li data-nodeid="16317">
<p data-nodeid="16318">部署工具，比如与 Git 集成、CI/CD 等；</p>
</li>
<li data-nodeid="16319">
<p data-nodeid="16320">监控运维工具，实现日志、报警等功能。</p>
</li>
</ul>
<p data-nodeid="17160" class="">除此之外，开发框架还可以提供自己的运行时，提供标准编程模型，抹平不同运行平台的差异，使开发者可以专注于业务代码的开发。最近几年，除了云厂商提供的开发框架，还有很多第三方的开发框架也在持续开发新功能，比如<a href="https://www.serverless.com/" data-nodeid="17164"> Serverless Framework</a>、<a href="https://midwayjs.org/" data-nodeid="17168">Midway</a>、<a href="https://github.com/cellbang/malagu" data-nodeid="17172">Malagu</a> 等。相信在 2021 年，这些框架将会更加成熟，并且还会有更多开发框架不断涌现。</p>

<p data-nodeid="16322">除了开发框架外，我认为 2021 年 Serverless 将带来的另一个巨大变革是<strong data-nodeid="16395">Serverless Jamstack 将逐渐兴起</strong>。在互联网飞速发展的这二十年中，主流的 Web 开发的技术栈依旧是类似于 LAMP （Linux，Apache、MySQL、PHP）的形式， 我们使用 PHP、Java 等编程语言编写代码，使用 Linux 服务器执行代码生成动态网页，并根据需要使用 MySQL 等数据库来存储数据。</p>
<p data-nodeid="16323">LAMP 架构可以很方便构建动态网站，但随着用户增多流量增大，服务端所需要的资源也就越来越多，动态网页也可能需要更长的时间来构建。<strong data-nodeid="16400">Jamstack 则是为了构建更快、更安全且更易于扩展的 Web 系统而诞生的：</strong></p>
<ul data-nodeid="16324">
<li data-nodeid="16325">
<p data-nodeid="16326">Jamstack 中 “Jam” J 代表 JavaScript，我们使用 JavaScript 实现页面交互、发起请求、获取数据构建动态页面；</p>
</li>
<li data-nodeid="16327">
<p data-nodeid="16328">A 代表 APIs，所有服务端的进程和数据库操作都被抽象为 API，以供页面中的 JavaScript 代码进行访问；</p>
</li>
<li data-nodeid="16329">
<p data-nodeid="16330">M 代表 Markup，即标记语言（如 Markdown、HTML），我们使用标记语言构建页面，并且这些页面需要是在部署前预先构建的。</p>
</li>
</ul>
<p data-nodeid="16331">在 Jamstack 技术栈中，我们会将预先构建好的代码部署到 CDN 上，这样就不需要像传统 LAMP 架构那样依赖服务器生成 HTML 了。有了 Serverless 之后，我们可以将 API 也托管到 Serverless 上，这样整个 Web 系统就完全不依赖服务器，也更易于扩展。在过去的一年中，很多 Jamstack 领域的创业公司，如 Netlify、  Vercel 都获得了巨额融资，Microsoft、Cloudflare 很多大公司也纷纷加入。所以我认为，2021 年 Serverless Jamstack 将逐渐兴起。</p>
<p data-nodeid="16332">你应该还有印象，我在“02 讲”中提到了，Kubernetes 是介于 Serverful 和 Serverless 中间的产物。但 Serverless 的出现也让 Kubernetes 有了一个新的方向：Serverless Kubernetes。目前 Kubernetes 已经成为业界容器编排的事实标准，基于 Kubernetes 的编排和管理功能，我们可以构建跨多个容器的应用，并且长期持续管理这些容器的健康状况。</p>
<p data-nodeid="16333">不过 Kubernetes 给我们带来了好处的同时，也让我们受困于集群容量规划、安全维护、故障诊断等复杂的运维操作之中，这仿佛又回到了没有云的时代。所以我们开始思考在 Kubernetes 上的 Serverless 化，即基于 Serverless 的 Kubernetes。</p>
<p data-nodeid="16334">传统的 Kubernetes 是以节点为中心的架构设计，节点是 Pod 运行的载体，Kubernetes 调度器在节点池中选择合适的节点来运行 Pod，并基于 kubelet 对 Pod 进行生命周期管理和自动化运维，当节点资源不够时，需要对节点进行扩容，然后对 Pod 进行扩容。而 Serverless Kubernetes 最大的改变就是将 Pod 和节点解耦，开发者就不用关注节点的运维，无须进行集群容量规划，只需要创建 Pod 即可。</p>
<p data-nodeid="16335">目前 AWS、阿里云等云厂商都发布了 Serverless Kubernetes 相关的云产品，开源界也有 Knative、OpenFaaS、Fission、Kubeless 等 Serverless Kubernetes 解决方案。相信随着 Serverless 技术的不断发展，在 2021 年<strong data-nodeid="16412">Serverless Kubernetes 也将越发流行。</strong></p>
<p data-nodeid="16336">可以说，Serverless 架构的兴起，真正解放了开发者，让基础设施的管理有了新的方式。如今越来越多的产品正在 Serverless 化，越来越多的企业正在布局和使用 Serverless，这不是对未来的期望，而是正在发生的事实。</p>
<p data-nodeid="16337">随着技术上对去中心化、轻量虚拟化、细粒度计算等需求愈发强烈，Serverless 也必将借势迅速发展，带来一场新的技术变革。这种全面云化的开发模式，也预示着真正的云计算时代正在到来。</p>
<p data-nodeid="16338">最后，我为你准备了一份结课问卷，希望你对本课程的内容进行评价，以便我及时优化课程内容。</p>
<p data-nodeid="17418" class="te-preview-highlight"><a href="https://wj.qq.com/s2/8088736/5489" data-nodeid="17421">点击链接，即可参与课程评价</a>。</p>

<p data-nodeid="16340">感谢你的聆听，衷心祝愿我们都能够快乐幸福地工作，我们江湖再见。</p>

---

### 精选评论

##### **俊：
> 感谢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 感谢支持，互相进步

