<p data-nodeid="1081" class="">上一讲咱们了解了 APM 系统与 SkyWalking 的配置使用方法。本讲咱们要解决分布式事务这一技术难题，这一讲咱们将介绍三方面内容：</p>
<ul data-nodeid="1082">
<li data-nodeid="1083">
<p data-nodeid="1084">讲解分布式事务的解决方案；</p>
</li>
<li data-nodeid="1085">
<p data-nodeid="1086">介绍 Alibaba Seata 分布式事务中间件；</p>
</li>
<li data-nodeid="1087">
<p data-nodeid="1088">分析 Seata 的 AT 模式实现原理。</p>
</li>
</ul>
<h3 data-nodeid="1089">分布式事务的解决方案</h3>
<p data-nodeid="1090">下面咱们先聊一下为什么会产生分布式事务。咱们举个例子，某线上商城会员在购买商品的同时产生相应的消费积分，消费积分在下一次购物时可以抵用现金。这个业务的逻辑如果放在以前的单点应用是很简单的，如下所示。</p>
<pre class="lang-java" data-nodeid="1091"><code data-language="java">开启数据库事务
创建订单
会员积分增加
商品库存减少
提交数据库事务
</code></pre>
<p data-nodeid="1092">在这个过程中，因为程序操作的是单点数据库，所以在一个数据库事务中便可完成所有操作，利用数据库事务自带的原子性保证了所有数据要么全部处理成功，要么全部回滚撤销。但是放在以微服务为代表的分布式架构下问题就没那么简单了，我们来看一下示意图。</p>
<p data-nodeid="1341" class=""><img src="https://s0.lgstatic.com/i/image6/M00/31/3A/CioPOWBsQPKAN4wTAAEBb9VqWsQ374.png" alt="图片1.png" data-nodeid="1345"></p>
<div data-nodeid="1342"><p style="text-align:center">分布式架构下调用关系图</p></div>


<p data-nodeid="1095">可以看到，商城应用作为业务的发起者分别向订单、会员、库存服务发起了调用，而这些服务又拥有自己独立的数据存储，因为在物理上各个数据库服务器都是独立的，每一个步骤的操作都会创建独立的事务，这就意味着在分布式处理时无法通过单点数据库利用一个事务保证数据的完整性，我们必须引入某种额外的机制来协调多个事务要么全部提交、要么全部回滚，以此保证数据的完整性，这便是“分布式事务”的由来。</p>
<p data-nodeid="1096">在分布式架构中有两种经典的分布式事务解决方案：<strong data-nodeid="1202">二阶段提交（2PC</strong>）与<strong data-nodeid="1203">三阶段提交（3PC）</strong>。</p>
<h4 data-nodeid="1097">二阶段提交</h4>
<p data-nodeid="1098">首先咱们分析下二阶段提交的处理过程，下面是二阶段提交中的<strong data-nodeid="1209">第一个阶段：事务预处理阶段。</strong></p>
<p data-nodeid="1866" class=""><img src="https://s0.lgstatic.com/i/image6/M00/31/3A/CioPOWBsQQCARCHWAAFYFUA6lfU789.png" alt="图片2.png" data-nodeid="1870"></p>
<div data-nodeid="1867"><p style="text-align:center">2PC 阶段一：事务预处理阶段</p></div>


<p data-nodeid="1101">可以看到，相比单点事务，分布式事务中增加了一个新的角色：事务协调者（Coordinator），它的职责就是协调各个分支事务的开启与提交、回滚的处理。以上图为例，当商城应用订单创建后，首先事务协调者会向各服务下达“处理本地事务”的通知，所谓本地事务就是每个服务应该做的事情，如订单服务中负责创建新的订单记录；会员服务负责增加会员的积分；库存服务负责减少库存数量。在这个阶段，被操作的所有数据都处于未提交（uncommit）的状态，会被排它锁锁定。当本地事务都处理完成后，会通知事务协调者“本地事务处理完毕”。当事务协调者陆续收到订单、会员、库存服务的处理完毕通知后，便进入“<strong data-nodeid="1218">阶段二：提交阶段</strong>”。</p>
<p data-nodeid="2391" class=""><img src="https://s0.lgstatic.com/i/image6/M01/31/32/Cgp9HWBsQQ-AO4-SAAEqPzPTZ7w364.png" alt="图片3.png" data-nodeid="2395"></p>
<div data-nodeid="2392"><p style="text-align:center">2PC 阶段二：提交阶段</p></div>


<p data-nodeid="1104">在提交阶段，事务协调者会向每一个服务下达提交命令，每个服务收到提交命令后在本地事务中对阶段一未提交的数据执行 Commit 提交以完成数据最终的写入，之后服务便向事务协调者上报“提交成功”的通知。当事务协调者收到所有服务“提交成功”的通知后，就意味着一次分布式事务处理已完成。</p>
<p data-nodeid="1105">这便是二阶段提交的正常执行过程，但假设在阶段一有任何一个服务因某种原因向事务协调者上报“事务处理失败”，就意味着整体业务处理出现问题，阶段二的操作就自动改为回滚（Rollback）处理，将所有未提交的数据撤销，使数据还原以保证完整性。</p>
<p data-nodeid="1106">对于二阶段提交来说，它有一个致命问题，当阶段二某个服务因为网络原因无法收到协调者下达的提交命令，则未提交的数据就会被长时间阻塞，可能导致系统崩溃。</p>
<p data-nodeid="2916" class=""><img src="https://s0.lgstatic.com/i/image6/M00/31/3A/CioPOWBsQUSADeWqAAEyyNR7o8E788.png" alt="图片4.png" data-nodeid="2920"></p>
<div data-nodeid="2917"><p style="text-align:center">二阶段提交的缺陷</p></div>


<p data-nodeid="1109">以上图为例，假如在提交阶段，库存服务实例与事务协调者之间断网。提交指令无法下达，这会导致库存中的“飞科剃须刀”商品库存记录会长期处于未提交的状态，因为这条记录被数据库排他锁长期独占，之后再有其他线程要访问“飞科剃须刀”库存数据，该线程就会长期处于阻塞状态，随着阻塞线程的不断增加，库存服务会面临崩溃的风险。</p>
<p data-nodeid="1110">那这个问题要怎么解决呢？其实只要在服务这一侧增加超时机制，过一段时间被锁定的“飞科剃须刀”数据因超时自动执行提交操作，释放锁定资源。尽管这样做会导致数据不一致，但也比线程积压导致服务崩溃要好，出于此目的，三阶段提交（3PC）便应运而生。</p>
<h4 data-nodeid="1111">三阶段提交</h4>
<p data-nodeid="1112">三阶段提交实质是将二阶段中的提交阶段拆分为“<strong data-nodeid="1240">预提交阶段</strong>”与“<strong data-nodeid="1241">提交阶段</strong>”，同时在服务端都引入超时机制，保证数据库资源不会被长时间锁定。下面是三阶段提交的示意流程：</p>
<p data-nodeid="3441" class=""><img src="https://s0.lgstatic.com/i/image6/M01/31/32/Cgp9HWBsQVCAZH3SAAFYFUA6lfU014.png" alt="图片5.png" data-nodeid="3445"></p>
<div data-nodeid="3442"><p style="text-align:center">3PC 阶段一：事务预处理阶段</p></div>


<ul data-nodeid="1115">
<li data-nodeid="1116">
<p data-nodeid="1117"><strong data-nodeid="1248">阶段一：事务预处理阶段。</strong></p>
</li>
</ul>
<p data-nodeid="1118">3PC 的事务预处理阶段与 2PC 是一样的，用于处理本地事务，锁定数据库资源，当所有服务返回成功后，进入阶段二。</p>
<p data-nodeid="3966" class=""><img src="https://s0.lgstatic.com/i/image6/M00/31/3A/CioPOWBsQWqABC-IAAEeBWGqOLQ853.png" alt="图片6.png" data-nodeid="3970"></p>
<div data-nodeid="3967"><p style="text-align:center">3PC 阶段二：预提交阶段</p></div>


<ul data-nodeid="1121">
<li data-nodeid="1122">
<p data-nodeid="1123"><strong data-nodeid="1256">阶段二：预提交阶段。</strong></p>
</li>
</ul>
<p data-nodeid="1124">预提交阶段只是一个询问机制，以确认所有服务都已准备好，同时在此阶段协调者和参与者都设置了超时时间以防止出现长时间资源锁定。当阶段二所有服务返回“可以提交”，进入阶段三“提交阶段”。</p>
<ul data-nodeid="1125">
<li data-nodeid="1126">
<p data-nodeid="1127"><strong data-nodeid="1261">阶段三：提交阶段。</strong></p>
</li>
</ul>
<p data-nodeid="1128">3PC 的提交阶段与 2PC 的提交阶段是一致的，在每一个数据库中执行提交实现数据的资源写入，如果协调者与服务通信中断导致无法提交，在服务端超时后在也会自动执行提交操作来保证资源释放。</p>
<p data-nodeid="1129">通过对比我们发现，三阶段提交是二阶段提交的优化版本，主要通过加入预提交阶段引入了超时机制，让数据库资源不会被长期锁定，但这也会带来一个新问题，数据一致性也很可能因为超时后的强制提交被破坏，对于这个问题各大软件公司都在各显神通，常见的做法有：增加异步的数据补偿任务、日终跑批前的数据补偿、更完善的业务数据完整性的校验代码、引入数据监控及时通知人工补录这些都是不错的补救措施。</p>
<p data-nodeid="1130">讲到这，相比你对 2PC 与 3PC 的分布式事务方案应该有了初步的了解，这里我还是要强调下，无论是 2PC 与 3PC 都是一种方案，是一种宏观的设计。如果要落地就要依托具体的软件产品，在 Java 开源领域能够提供完善的分布式事务解决方案的产品并不多，比较有代表性的产品有 ByteTCC、TX-LCN、EasyTransaction、Alibaba Seata，其中无论从成熟度、厂商背景、更新频度、社区活跃度各维度比较，Alibaba Seata都是数一数二的分布式事务中间件产品，本讲后面的内容将围绕Alibaba Seata的AT模式展开，探讨Alibaba Seata是如何实现自动化的分布式事务处理的。</p>
<h3 data-nodeid="1131">Alibaba Seata 分布式事务中间件</h3>
<p data-nodeid="1132">Alibaba Seata 是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。它的官网是<a href="http://seata.io/?fileGuid=xxQTRXtVcqtHK6j8" data-nodeid="1269">http://seata.io/</a>,截止到目前 Seata 在 GitHub 已有 18564 star，最新版本已迭代到 1.4.0，阿里多年的技术沉淀让 Seata 的内部版本平稳渡过了多次双 11 的考验。2019 年 1 月为了打造更加完善的技术生态和普惠技术成果，Seata 正式宣布对外开源，未来 Seata 将以社区共建的形式帮助其技术更加可靠与完备，按官方的说法Seata目前已具备了在生产环境使用的条件。</p>
<p data-nodeid="1133"><img src="https://s0.lgstatic.com/i/image6/M00/2C/C1/CioPOWBlidqAOQe-AASTbag_bO0476.png" alt="Drawing 6.png" data-nodeid="1273"></p>
<p data-nodeid="1134">Seata 提供了多种分布式事务的解决方案，包含 AT 模式、TCC 模式、SAGA 模式以及 XA 模式。其中 AT 模式提供了最简单易用且无侵入的事务处理机制，通过自动生成反向 SQL 实现事务回滚。从 AT 模式入手使用，使我们理解分布式事务处理机制是非常好的学习办法。</p>
<p data-nodeid="1135"><img src="https://s0.lgstatic.com/i/image6/M01/2C/C1/CioPOWBlieOAReN5AADg3SbfFhE124.png" alt="Drawing 7.png" data-nodeid="1277"></p>
<div data-nodeid="1136"><p style="text-align:center">Seata 的特色功能</p></div>
<p data-nodeid="1137">AT 模式是 Seata 独创的模式，它是基于 2PC 的方案，核心理念是利用数据库 JDBC 加上 Oracle、MySQL 自带的事务方式来对我们分布式事务进行管理。说起来有点晦涩，下边我就结合这张 AT 模式方案图给大家介绍，在 Seata 中关于分布式事务到底需要哪些组件，以及他们都起到了什么样的职能。</p>
<p data-nodeid="1138"><img src="https://s0.lgstatic.com/i/image6/M01/2C/C1/CioPOWBlieyAWtXpAAF71Z7iu4s460.png" alt="Drawing 8.png" data-nodeid="1281"></p>
<div data-nodeid="1139"><p style="text-align:center">Seata 组件图</p></div>
<p data-nodeid="1140">通过Seata组件图我们可以看到三个组成部分：</p>
<ul data-nodeid="1141">
<li data-nodeid="1142">
<p data-nodeid="1143"><strong data-nodeid="1287">第一个是事务协调者（TC）</strong>，它的作用是维护全局和分支事务的状态，驱动全局事务提交或者回滚，这正是前面讲解 2PC 或者 3PC 方案时提到的事务协调者组件的具体实现，TC 由 SEATA 官方提供。</p>
</li>
<li data-nodeid="1144">
<p data-nodeid="1145"><strong data-nodeid="1292">第二个是事务管理器（TM）</strong>，事务管理器用于定义全局事务的范围，开始全局事务提交或者回滚全局事务都是由 TM 来决定。</p>
</li>
<li data-nodeid="1146">
<p data-nodeid="1147"><strong data-nodeid="1297">第三个是资源管理器（RM）</strong>，他用于管理分支事务处理的资源，并且报告分支事务的状态，并驱动分支事务提交或者回滚。</p>
</li>
</ul>
<p data-nodeid="1148">这些概念可能有些晦涩，我们通过前面商城会员采购积分的例子进行讲解。</p>
<h3 data-nodeid="1149">Seata AT 模式执行过程</h3>
<p data-nodeid="4491" class=""><img src="https://s0.lgstatic.com/i/image6/M01/31/32/Cgp9HWBsQXmAElvTAAEBb9VqWsQ307.png" alt="图片1.png" data-nodeid="4495"></p>
<div data-nodeid="4492"><p style="text-align:center">创建订单调用逻辑</p></div>


<p data-nodeid="1152">这里我先给出商城应用中会员采购业务的伪代码。</p>
<pre class="lang-java" data-nodeid="1153"><code data-language="java">会员采购(){
    订单服务.创建订单();
    积分服务.增加积分();
    库存服务.减少库存();
}
</code></pre>
<p data-nodeid="1154">在会员采购方法中，需要分别执行<strong data-nodeid="1309">创建订单、增加积分、减少库存</strong>三个步骤完成业务，对于“会员采购”来说方法执行成功，则代表这个全局分布式事务需要提交，如果中间过程出错，则需要全局回滚，这个业务方法本身就决定了全局提交、回滚的时机以及决定了哪些服务需要参与业务处理，因此商城应用的会员采购方法就充当起事务管理器（TM）的角色。</p>
<p data-nodeid="1155">而与之对应的在订单服务中创建订单、会员服务中增加积分、库存服务减少库存这些实际产生的数据处理的服务模块，则被称为资源管理器（RM)。</p>
<p data-nodeid="1156">最后就是由Seata提供的Seata-Server中间件则提供事务协调者（TC）这个角色，实施全局事务1的提交、回滚命令下发。</p>
<p data-nodeid="1157">为了方便理解，我画了时序图介绍 Seata 的执行过程。</p>
<p data-nodeid="5016" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image6/M01/31/32/Cgp9HWBsQYmAcXTtAAFjr2n0qsc325.png" alt="图片7.png" data-nodeid="5020"></p>
<div data-nodeid="5017"><p style="text-align:center">Seata 时序图</p></div>


<p data-nodeid="1160">第一步，在商城应用（TM）与三个服务（RM）启动后自动向事务协调者Seata-Server（TC）进行注册，让 TC 知晓各个组件的详细信息。</p>
<p data-nodeid="1161">第二步，当会员购物时会执行 TM 的“会员采购”方法，当进入方法前 Seata 为 TM 提供的客户端会自动生效，向 TC 发出开启全局事务的请求。</p>
<p data-nodeid="1162">第三步，会员采购方法开始执行，会依次执行 3 个服务的新增订单、增加积分、减少库存，在请求送往新的 RM 时，都会向 TC 注册新的分支事务。这些分支事务在处理时不但向业务表写入数据，还会自动向 Seata 强制要求的 UNDO_LOG 回滚日志表写入回滚 SQL 日志。</p>
<p data-nodeid="1163">以新增订单事务为例：新增订单时执行的 SQL 语句如下：</p>
<pre class="lang-sql" data-nodeid="1164"><code data-language="sql"><span class="hljs-keyword">INSERT</span> <span class="hljs-keyword">INTO</span> <span class="hljs-keyword">order</span>(<span class="hljs-keyword">id</span>,...) <span class="hljs-keyword">values</span>(<span class="hljs-number">1001</span>,...)
</code></pre>
<p data-nodeid="1165">与之对应的，Seata 的回滚日志是基于 SQL 反向生成，新增订单创建了 1001 订单，那 Seata会对 SQL 进行解析生成反向的回滚 SQL 日志保存在 UNDO_LOG 表，如下所示：</p>
<pre class="lang-sql" data-nodeid="1166"><code data-language="sql"><span class="hljs-keyword">DELETE</span> <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">order</span> <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">id</span> = <span class="hljs-number">1001</span>
</code></pre>
<p data-nodeid="1167">与之类似会员积分会生成加积分的业务 SQL 以及减积分的回滚 SQL。</p>
<pre class="lang-sql" data-nodeid="1168"><code data-language="sql"><span class="hljs-comment">#加积分</span>
<span class="hljs-keyword">UPDATE</span> <span class="hljs-keyword">FROM</span> points <span class="hljs-keyword">SET</span> point = <span class="hljs-number">180</span> + <span class="hljs-number">20</span> <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">mid</span> = <span class="hljs-number">182</span>
<span class="hljs-comment">#UNDO_LOG表中的减积分SQL</span>
<span class="hljs-keyword">UPDATE</span> <span class="hljs-keyword">FROM</span> points <span class="hljs-keyword">SET</span> point = <span class="hljs-number">200</span> - <span class="hljs-number">20</span> <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">mid</span> = <span class="hljs-number">182</span>
</code></pre>
<p data-nodeid="1169">第四步，当 RM 的分支事务执行成功后，会自动向 TC 上报分支事务处理成功。</p>
<p data-nodeid="1170">第五步，当会员采购方法正确执行，所有 RM 也向 TC 上报分支事务处理成功，在“会员采购”方法退出前，TM 内置的 Seata 客户端会向 TC 自动发起“提交全局事务”请求。TC 收到“提交全局事务”请求，向所有 RM 下达提交分支事务的命令，每一个 RM 在收到提交命令后，会删除之前保存在 UNDO_LOG 表中的回滚日志。</p>
<p data-nodeid="1171">但是事情总会有意外，假设某个 RM 分支事务处理失败，此时 TM 便不再向 TC 发起“提交全局事务”，转而发送“回滚全局事务”，TC 收到后，通知所有之前已处理成功的 RM 执行回滚 SQL 将数据回滚。</p>
<p data-nodeid="1172">比如 1001 订单在第三步“减少库存”时发现库存不足导致库存服务预处理失败，那全局回滚时第一步订单服务会自动执行删除 1001 订单的回滚 SQL。</p>
<pre class="lang-sql" data-nodeid="1173"><code data-language="sql"><span class="hljs-keyword">DELETE</span> <span class="hljs-keyword">FROM</span> <span class="hljs-keyword">order</span> <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">id</span> = <span class="hljs-number">1001</span>
</code></pre>
<p data-nodeid="1174">以及第二步积分服务会自动执行减少积分的回滚 SQL。</p>
<pre class="lang-sql" data-nodeid="1175"><code data-language="sql"><span class="hljs-keyword">UPDATE</span> <span class="hljs-keyword">FROM</span> points <span class="hljs-keyword">SET</span> point = <span class="hljs-number">200</span> - <span class="hljs-number">20</span> <span class="hljs-keyword">WHERE</span> <span class="hljs-keyword">mid</span> = <span class="hljs-number">182</span>
</code></pre>
<p data-nodeid="1176">Seata AT模式就是通过执行反向 SQL 达到数据还原的目的，当反向 SQL 执行后便自动从 UNDO_LOG 表中删除。这便是 Seata AT 模式的大致执行过程，在这个过程中我们发现 Seata AT 模式设计的巧妙之处，Seata 为了能做到无侵入的自动实现全局事务提交与回滚，它在 TM端利用了类似于“Spring 声明式事务”的设计，在进入 TM 方法前通知 TC 开启全局事务，在成功执行后自动提交全局事务，执行失败后进行全局回滚。同时在 RM 端也巧妙的采用了 SQL 解析技术自动生成了反向的回滚 SQL 来实现数据还原。</p>
<p data-nodeid="1177">在这我也思考过，为什么 Seata 要生成反向 SQL，而不是利用数据库自带的排他锁机制处理呢？翻阅资料后理解到它的设计意图，如果采用排它锁机制会导致数据资源被锁死，可能会产生大量的数据资源阻塞，进而存在应用崩溃的风险。而生成反向 SQL 的方案则是在预处理阶段事务便已提交，不会出现长时间数据资源锁定的情况，这样能有效提高并发量。但这样做也有弊端，在研究时发现 Seata 是工作在“读未提交”的隔离级别，高并发环境下容易产生脏读、幻读的情况，这也是需要特别注意的地方。</p>
<h3 data-nodeid="1178">小结与预告</h3>
<p data-nodeid="1179">本讲我们首先针对分布式事务的 2PC 与 3PC 方案进行讲解，了解了分布式事务的执行过程与两者之间的区别；之后咱们认识了 Alibaba 出品的分布式事务中间件 Seata，最后通过电商会员采购的案例讲解了 Seata AT 模式的处理过程，让我们对 Seata 有了初步了解。在后面的实践篇，我们将本节偏理论的内容进行落地实现，看通过代码如何使用 Seata 处理分布式事务。</p>
<p data-nodeid="1180">这里我为你留一道讨论题：既然分布式事务相比单点式事务要复杂得多，在项目中有什么好办法可以规避分布式事务呢？欢迎你把自己的想法写在评论区和大家一起分享。</p>
<p data-nodeid="1181" class="">下一讲我们讲解 Spring Cloud Alibaba 体系下的消息队列中间件 Alibaba RocketMQ，看通过 RocketMQ 如何解决服务间异步通信的问题。</p>

---

### 精选评论

##### **用户0131：
> 不能把源代码放到git上吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; https://github.com/qiyisoft/sca

##### *风：
> 老师，有项目源码地址吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 大家等待已久的源码已整理，有需要的同学可以登录下载哦
链接：https://pan.baidu.com/s/16avfOynyMNljeCVpjl8IXQ 
提取码：307t

##### *晴：
> 三阶段提交的阶段一和阶段二反了吧，不是一开始先询问吗

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 没有反,肯定是先阶段1先锁定资源,阶段2询问等待所有服务处理完毕.

##### **用户5069：
> 老师，会员采购 sate会把这些资源都锁定呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 默认Seata为了高并发是不锁定的,但是Seata提供了@GlobalLock开启分布式锁可以提供这个功能.

##### **莉：
> 看Seata 时序图 图中服务（RM）是先提交本地事务（本地事务就是分支事务吧？） 再上报Seata（TC）分支事务处理完毕seata 去通知 服务（RM）提交分支事务吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的RM首先要确保本地事务提交完毕再通知TC分支事务的。

##### iretw@163.com：
> 1.利用消息队列的事物机制结合补偿机制。

##### *镇：
> 棒

##### **健：
> Seata有用在生产环境吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 1.3版本以后开始逐渐生产开始试点了。1.3以前版本不能用
以前都是用的阿里商用版GTS

##### **1001：
> 老师你好！请问seata支持所有数据库吗，还是说只支持mysql oracle呢，还有seata有什么缺点吗，比如大批量操作新增，更新或删除

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 目前我了解的MySQL肯定支持的，后期准备支持PSQL，对Oracle的支持不太完整。

