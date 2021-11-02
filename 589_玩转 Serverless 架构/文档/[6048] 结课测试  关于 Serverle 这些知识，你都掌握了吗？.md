<p data-nodeid="11860">现在咱们的课程结束了，恭喜你顺利学完《玩转 Serverless 架构》中所有的内容，不知道你掌握的怎么样呢？我为你准备了一套结课测试题。</p>
<p data-nodeid="11861">它是对你课程学习效果的一个检验，你也可以把它当作对课程的系统性回顾。</p>
<p data-nodeid="11862">我们的测试题目一共包括 20 道题（单选与多选），每题 5 分，满分 100 分。记得在留言区写下你的答案哦，另外我会在结束语中公布正确的答案。</p>
<p data-nodeid="11863">好了，请开始你的测试吧。加油！</p>
<hr data-nodeid="11864">
<h4 data-nodeid="12915" class="">1. 下面关于 Serverless 定义的说法错误的是？</h4>

<p data-nodeid="11868">A Serverless 不需要服务器</p>
<p data-nodeid="11869">B Serverless 是指构建和运行软件时不需要关心服务器的一种架构思想</p>
<p data-nodeid="11870">C 目前 Serverless 的主流实现是 FaaS + BaaS</p>
<p data-nodeid="11871">D Serverless 是云原生的一种实现</p>
<h4 data-nodeid="13331" class="">2. 一个应用如果是 Serverless 架构的，必须要实现自动弹性伸缩和按量付费。</h4>

<p data-nodeid="11875">A 正确</p>
<p data-nodeid="11876">B 错误</p>
<h4 data-nodeid="13743" class="">3. （多选）下面哪些选项是 Serverless 应用的特点？</h4>

<p data-nodeid="11880">A 自动弹性伸缩</p>
<p data-nodeid="11881">B 按量付费</p>
<p data-nodeid="11882">C 事件驱动</p>
<p data-nodeid="11883">D 运维成本高</p>
<h4 data-nodeid="14151" class="">4. Serverless 只能用来开发无状态的应用，不能用来开发有状态的应用。</h4>

<p data-nodeid="11887">A 正确</p>
<p data-nodeid="11888">B 错误</p>
<h4 data-nodeid="14555" class="">5. 下面关于 Serverless 函数启动过程说法错误的是？</h4>

<p data-nodeid="11892">A 函数启动过程分为冷启动和热启动</p>
<p data-nodeid="11893">B 冷启动耗时较长，热启动耗时很短</p>
<p data-nodeid="11894">C 热启动时函数会重复利用上一次的执行上下文</p>
<p data-nodeid="11895">D 函数每次执行都需要经过冷启动</p>
<h4 data-nodeid="14955" class="">6. 函数的启动过程包含下载代码、启动容器、启动运行环境、执行代码四个步骤，前三个步骤为冷启动，最后一个步骤为热启动。</h4>

<p data-nodeid="11899">A 正确</p>
<p data-nodeid="11900">B 错误</p>
<h4 data-nodeid="15351" class="">7. 运行 Serverless 函数代码的 FaaS 平台通常是容器技术实现运行环境隔离的。</h4>

<p data-nodeid="11904">A 正确</p>
<p data-nodeid="11905">B 错误</p>
<h4 data-nodeid="15743" class="">8. 不同 FaaS 平台的触发器和入口函数定义是完全一致的。</h4>

<p data-nodeid="11909">A 正确</p>
<p data-nodeid="11910">B 错误</p>
<h4 data-nodeid="16131" class="">9. Serverless 应用的代码依赖和系统依赖都需要安装在项目中，并和应用代码一起部署到 FaaS 平台。</h4>

<p data-nodeid="11914">A 正确</p>
<p data-nodeid="11915">B 错误</p>
<h4 data-nodeid="16515" class="">10. FaaS 平台的自定义运行时本质上是实现一个自定义的 HTTP 服务。</h4>

<p data-nodeid="11919">A 正确</p>
<p data-nodeid="11920">B 错误</p>
<h4 data-nodeid="16895" class="">11. （多选）下面哪个关于 Serverless 应用单元测试的描述是错误的？</h4>

<p data-nodeid="11924">A Serverless 应用由于其分布式、依赖云服务、事件驱动等特性，导致编写单元测试很困难</p>
<p data-nodeid="11925">B 为了方便编写单元测试，需要将业务逻辑和依赖的云服务分离开来</p>
<p data-nodeid="11926">C 编写单元测试时，需要考虑速度、隔离性、单一职责等因素，避免单元测试成为开发的负担</p>
<p data-nodeid="11927">D 为了代码快速上线，我们可以不编写单元测试</p>
<h4 data-nodeid="17271" class="">12. （多选）下面哪些方案可以提升 Serverless 应用的性能？</h4>

<p data-nodeid="11931">A 提前给函数预热</p>
<p data-nodeid="11932">B 减小代码体积、减少不必要的依赖</p>
<p data-nodeid="11933">C 选择 Node.js、Python 等冷启动耗时短的编程语言</p>
<p data-nodeid="11934">D 为函数设置合适的内存和并发</p>
<h4 data-nodeid="17643" class="">13. （多选）下面关于在 Serverless 应用中使用访问控制说法正确的是？</h4>

<p data-nodeid="11938">A 云厂商主要通过主账号、角色、权限策略等方式来实现云上资源的访问控制</p>
<p data-nodeid="11939">B 通过访问控制，我们能实现分权、云服务授权、跨账号授权等云上资源管控需求</p>
<p data-nodeid="11940">C 为了安全，我们需要为函数设置最小的权限</p>
<p data-nodeid="11941">D 为了方便，我们可以给直接给函数设置尽可能大的权限</p>
<h4 data-nodeid="18011" class="">14. （多选）下面关于 Serverless 应用安全的说法正确的是？</h4>

<p data-nodeid="11945">A 在云上运行的应用，云厂商负责计算、网络、存储等底层资源的安全性，应用所有者负责应用本身的安全性</p>
<p data-nodeid="11946">B Serverless 安全性面临的主要挑战是：越来越多的攻击面、越来越复杂的攻击方式、可观测性不足，以及传统安全测试方法和防护方案不适用于 Serverless 架构</p>
<p data-nodeid="11947">C Serverless 安全性的面临的风险有：函数事件数据注入、身份认证无效、用户或角色权限过高、敏感数据泄漏、DDoS 攻击等</p>
<p data-nodeid="11948">D Serverless 应用无须运维，所以 Serverless 应用很安全，不需要我们关心</p>
<h4 data-nodeid="18375" class="">15. （多选）下面哪些方案可以提升 Serverless 应用的安全性？</h4>

<p data-nodeid="11952">A 对于提供 API 服务的 Serverless 应用，使用 API 网关代替 HTTP 触发器</p>
<p data-nodeid="11953">B 在代码中尽可能使用临时访问凭证</p>
<p data-nodeid="11954">C 对存储在云上和需要传输的数据进行加密</p>
<p data-nodeid="11955">D 使用日志服务等产品统一记录函数执行的日志</p>
<h4 data-nodeid="18735" class="">16. （多选）下面哪些方案可以节省 Serverless 应用的成本？</h4>

<p data-nodeid="11959">A 为函数设置超时时间，避免函数因为异常而无限制地运行下去</p>
<p data-nodeid="11960">B 为函数分配合适的内存，在不影响性能的情况下减少资源消耗</p>
<p data-nodeid="11961">C 为函数实例设置合适的并发，使多个请求共用一个实例</p>
<p data-nodeid="11962">D 提升函数的性能</p>
<h4 data-nodeid="19091" class="">17. Serverless 应用的成本包括 FaaS 中函数执行的成本，以及函数所依赖的触发器、数据源和 BaaS 服务的成本。</h4>

<p data-nodeid="11966">A 正确</p>
<p data-nodeid="11967">B 错误</p>
<h4 data-nodeid="19443" class="">18. （多选）下面关于传统应用迁移到 Serverless 架构的说法正确的是？</h4>

<p data-nodeid="11971">A 传统应用迁移到 Serverless，想要考虑内存缓存、身份认证、持久化存储、Web 服务 Serverless 化等改造点</p>
<p data-nodeid="11972">B 如果一个应用本身就是分布式部署的，且在架构上是计算和存储分离的，比较容易迁移到 Serverless</p>
<p data-nodeid="11973">C Web 服务 Serverless 化主要原理是实现一个自定义 HTTP 服务，通过该 HTTP 服务处理事件对象和 Web 请求的差异</p>
<p data-nodeid="11974">D 传统应用不需要改造就可以直接迁移到 Serverless</p>
<h4 data-nodeid="19791" class="">19. Serverless 应用中的函数是无状态的，所以传统的 Cookie-Session、JWT 等身份认证方案都不适用于 Serverless？</h4>

<p data-nodeid="11978">A 正确</p>
<p data-nodeid="11979">B 错误</p>
<h4 data-nodeid="20135" class="te-preview-highlight">20. 将可以执行文件（如 ffmpeg）部署到函数计算时，如果可执行文件在本地权限是 -rwxr-xr-- ，我们可以直接将其上传到函数计算平台，并在代码中使用。</h4>

<p data-nodeid="11983">A 正确</p>
<p data-nodeid="11984">B 错误</p>

---

### 精选评论

##### **煌：
> 1、a。2、b。3、a、b、c。4、b。5、d。6、a。7、a。8、a。9、a。10、a。11、b、d。12、a、b、c、d。13、a、b、c。14、a、b、c。15、a、c、d。16、a、b、c、d。17、a。18、a、b、c。19、b。20、a。

