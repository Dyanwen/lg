<p data-nodeid="2">在开始今天的课程前，先来看看上节课的思考题：树的最大深度对随机森林模型效果有什么影响呢？答案是树的最大深度可以用来控制模型的过拟合。</p>
<p data-nodeid="3">从这个模块开始，我们将进入到项目实战练习。在那之前，我们先来简要介绍下要实现的系统架构及相关的理论知识，这样在实现的过程中，大家能够更加清楚地领会整个过程，达到先务虚再务实、务虚是为了更好地务实的效果。</p>
<p data-nodeid="4">本课时的主要内容有：</p>
<ul data-nodeid="5">
<li data-nodeid="6">
<p data-nodeid="7">数据仓库</p>
</li>
<li data-nodeid="8">
<p data-nodeid="9">数据立方体与多维分析</p>
</li>
<li data-nodeid="10">
<p data-nodeid="11">商业智能系统</p>
</li>
</ul>
<h3 data-nodeid="12">数据仓库</h3>
<h4 data-nodeid="13">数据仓库的特征</h4>
<p data-nodeid="14" class="">按照数据仓库系统构造方面的领衔设计师 William H.Inmon 的说法，“数据仓库是一个面向主题的、集成的、时变的、非易失的数据集合，支持管理者的决策过程”，它指出了数据仓库的四个主要特征，下面我们分别进行解释。</p>
<ul data-nodeid="15">
<li data-nodeid="16">
<p data-nodeid="17"><strong data-nodeid="185">面向主题的（subject-oriented）</strong>：数据仓库围绕一些重要的主题，如顾客、供应商、产品和销售组织。数据仓库关注决策者的数据建模和分析，而不是单位的日常操作和事物处理。因此数据仓库通常排除对于决策无用的数据，只提供特定主题的简明视图。</p>
</li>
<li data-nodeid="18">
<p data-nodeid="19"><strong data-nodeid="190">集成的（integrated</strong>）：通常构造数据仓库是将多个异构数据源，如关系数据库、一般文件和联机事务处理记录集成在一起。它使用数据清理和数据集成技术，以确保命名约定、编码结构、属性度量等的一致性。</p>
</li>
<li data-nodeid="20">
<p data-nodeid="21"><strong data-nodeid="197">时变的（time-variant）</strong>：数据仓库从历史的角度（例如，过去 5~10 年）提供信息。数据仓库中的关键结构都隐式或显式地包含时间元素。</p>
</li>
<li data-nodeid="22">
<p data-nodeid="23"><strong data-nodeid="202">非易失的（nonvolatile）</strong>：数据仓库总是物理地分离存放数据，这些数据源于操作环境下的应用数据。由于这种分离，数据仓库不需要事务处理、恢复和并发控制机制。数据的易失性在于操作型系统是一次访问和处理一条记录，可以对操作环境中的数据进行更新。但是数据仓库中的数据呈现出非常不同的特性，数据仓库中的数据通常是一次性加载，但在数据仓库环境中并不进行一般意义上的数据更新。通常，它只需要两种数据访问操作：数据的初始化加载和数据访问。</p>
</li>
</ul>
<h4 data-nodeid="24">业务数据库与数据仓库的区别</h4>
<p data-nodeid="25">操作数据库系统的主要任务是执行联机事务和查询处理，这种系统称作<strong data-nodeid="209">联机事务处理（OLTP）系统</strong>。它涵盖了组织的大部分日常操作，如购物、库存、工资等，也被称作业务系统。</p>
<p data-nodeid="26">数据仓库系统在数据分析和决策方面为用户提供服务，这种系统称作<strong data-nodeid="215">联机分析处理（OLAP）系统</strong>。</p>
<p data-nodeid="27">两种系统的主要区别概述如下：</p>
<ul data-nodeid="28">
<li data-nodeid="29">
<p data-nodeid="30"><strong data-nodeid="221">用户和系统的面向性</strong>：OLTP 是面向客户的，用于办事员、客户和信息技术专业人员的事务和查询处理。OLAP 是面向市场的，用于知识工人（包括经理、主管和分析人员）的数据分析。</p>
</li>
<li data-nodeid="31">
<p data-nodeid="32"><strong data-nodeid="226">数据内容</strong>：OLTP 系统管理的是当前数据，通常这种数据太琐碎，很难用于决策。OLAP 系统管理的则是大量的历史数据，提供汇总和聚集机制，并在不同的粒度层上存储和管理信息，这些特点使得数据更容易用于有根据的决策。</p>
</li>
<li data-nodeid="33">
<p data-nodeid="34"><strong data-nodeid="231">视图</strong>：OLTP 系统主要关注一个企业或部门内部的当前数据，而不涉及历史数据或不同组织的数据。相比之下，由于组织的演变，OLAP 系统常常跨越数据库模式的多个版本。OLAP 系统还要处理来自不同组织的信息，以及由多个数据库集成的信息。由于数据量巨大，OLAP 系统的数据也存放在多个存储介质上。</p>
</li>
<li data-nodeid="35">
<p data-nodeid="36"><strong data-nodeid="236">访问模式</strong>：OLTP 系统主要由短的原子事务组成，这种系统需要并发控制和恢复机制。然而，对 OLAP 系统的访问大部分是只读操作（由于大部分数据仓库存放历史数据，而不是最新数据），尽管许多访问可能是复杂的查询。</p>
</li>
</ul>
<p data-nodeid="37">其他区别还包括数据库大小、操作的频繁程度、性能度量等，在这里就不再展开。</p>
<p data-nodeid="38">既然操作数据库存放了大量数据，你可能会奇怪“为什么不直接在这种数据库上进行联机分析处理（OLAP），而是另外花费时间和资源去构造分离的数据仓库？”。分离的主要原因是有助于提高两个系统的性能。操作数据库是为已知的任务和负载设计的，如使用的主键索引，检索特定的记录，优化“定制的”查询。而数据仓库的查询通常是复杂的，涉及大量数据汇总级的计算，可能需要特殊的基于多维视图的数据组织、存取方法和实现方法。在操作数据库上处理 OLAP 查询，可能会大大降低操作任务的性能。</p>
<p data-nodeid="39">此外，操作数据库支持多事务的并发处理，需要并发控制和恢复机制（例如，加锁和记日志），以确保一致性和事务的鲁棒性。通常，OLAP 查询只需要对汇总和聚集数据记录进行只读访问。如果将并发控制和恢复机制用于这种 OLAP 操作，就会危害并行事务的运行，从而大大降低 OLTP 系统的吞吐量。</p>
<p data-nodeid="40">最后，数据仓库与操作数据库分离是由于这两种系统中数据的结构、内容和用法都不相同。决策支持需要历史数据，而操作数据库一般不维护历史数据。在这种情况下，操作数据库中的数据尽管很丰富，但对于决策是远非完整的。决策支持需要整合来自异构源的数据（例如，聚集和汇总），产生高质量的、纯净的和集成的数据。操作数据库只维护详细的原始数据（如事务），这些数据在进行分析之前需要整理。由于两种系统提供大不相同的功能，需要不同类型的数据，因此需要维护分离的数据库。</p>
<h4 data-nodeid="41">数据集市</h4>
<p data-nodeid="42">在数据仓库架构中，还存在一种形态，叫数据集市。数据集市往往服务于一组特定群体的分析需求（如会计部分或者信贷部门）。有些数据集市是独立的，也就是说它可以独立于数据仓库存在，而直接由业务数据库的历史应用创建。但更多的情况是，数据集市往往作为数据仓库之上的一个面向分析应用，换言之，数据集市的用户往往是直接和业务相关的分析应用。</p>
<h3 data-nodeid="43">数据立方体与多维分析</h3>
<p data-nodeid="44">限于篇幅，这里就不和大家详细介绍数据仓库与数据集市建模的方法论，而是介绍一种数据结构和分析方法，在后面的项目中，你可以看到我们是如何将这种结构和方法融入项目中去的。</p>
<p data-nodeid="45">我们先来介绍什么是数据立方体。以下表为例，表中一共有四列，其中 sales_count 被称为度量，而其余三个字段被称为维度，整个这张表就可以看成是一个数据立方体，表格维度就可以看成立方体的维度。</p>
<table data-nodeid="47">
<thead data-nodeid="48">
<tr data-nodeid="49">
<th data-org-content="sales_count" data-nodeid="51">sales_count</th>
<th data-org-content="sales_area" data-nodeid="52">sales_area</th>
<th data-org-content="sales_season" data-nodeid="53">sales_season</th>
<th data-org-content="sales_item" data-nodeid="54">sales_item</th>
</tr>
</thead>
<tbody data-nodeid="59">
<tr data-nodeid="60">
<td data-org-content="1223" data-nodeid="61">1223</td>
<td data-org-content="四川" data-nodeid="62">四川</td>
<td data-org-content="春" data-nodeid="63">春</td>
<td data-org-content="自行车" data-nodeid="64">自行车</td>
</tr>
<tr data-nodeid="65">
<td data-org-content="3244" data-nodeid="66">3244</td>
<td data-org-content="四川" data-nodeid="67">四川</td>
<td data-org-content="夏" data-nodeid="68">夏</td>
<td data-org-content="自行车" data-nodeid="69">自行车</td>
</tr>
<tr data-nodeid="70">
<td data-org-content="3242" data-nodeid="71">3242</td>
<td data-org-content="四川" data-nodeid="72">四川</td>
<td data-org-content="秋" data-nodeid="73">秋</td>
<td data-org-content="自行车" data-nodeid="74">自行车</td>
</tr>
<tr data-nodeid="75">
<td data-org-content="1555" data-nodeid="76">1555</td>
<td data-org-content="四川" data-nodeid="77">四川</td>
<td data-org-content="冬" data-nodeid="78">冬</td>
<td data-org-content="自行车" data-nodeid="79">自行车</td>
</tr>
<tr data-nodeid="80">
<td data-org-content="3333" data-nodeid="81">3333</td>
<td data-org-content="北京" data-nodeid="82">北京</td>
<td data-org-content="春" data-nodeid="83">春</td>
<td data-org-content="自行车" data-nodeid="84">自行车</td>
</tr>
<tr data-nodeid="85">
<td data-org-content="4444" data-nodeid="86">4444</td>
<td data-org-content="北京" data-nodeid="87">北京</td>
<td data-org-content="夏" data-nodeid="88">夏</td>
<td data-org-content="自行车" data-nodeid="89">自行车</td>
</tr>
<tr data-nodeid="90">
<td data-org-content="5555" data-nodeid="91">5555</td>
<td data-org-content="北京" data-nodeid="92">北京</td>
<td data-org-content="秋" data-nodeid="93">秋</td>
<td data-org-content="自行车" data-nodeid="94">自行车</td>
</tr>
<tr data-nodeid="95">
<td data-org-content="3333" data-nodeid="96">3333</td>
<td data-org-content="北京" data-nodeid="97">北京</td>
<td data-org-content="冬" data-nodeid="98">冬</td>
<td data-org-content="自行车" data-nodeid="99">自行车</td>
</tr>
<tr data-nodeid="100">
<td data-org-content="2312" data-nodeid="101">2312</td>
<td data-org-content="四川" data-nodeid="102">四川</td>
<td data-org-content="春" data-nodeid="103">春</td>
<td data-org-content="运动水壶" data-nodeid="104">运动水壶</td>
</tr>
<tr data-nodeid="105">
<td data-org-content="3233" data-nodeid="106">3233</td>
<td data-org-content="四川" data-nodeid="107">四川</td>
<td data-org-content="夏" data-nodeid="108">夏</td>
<td data-org-content="运动水壶" data-nodeid="109">运动水壶</td>
</tr>
<tr data-nodeid="110">
<td data-org-content="3222" data-nodeid="111">3222</td>
<td data-org-content="四川" data-nodeid="112">四川</td>
<td data-org-content="秋" data-nodeid="113">秋</td>
<td data-org-content="运动水壶" data-nodeid="114">运动水壶</td>
</tr>
<tr data-nodeid="115">
<td data-org-content="1110" data-nodeid="116">1110</td>
<td data-org-content="四川" data-nodeid="117">四川</td>
<td data-org-content="冬" data-nodeid="118">冬</td>
<td data-org-content="运动水壶" data-nodeid="119">运动水壶</td>
</tr>
<tr data-nodeid="120">
<td data-org-content="2323" data-nodeid="121">2323</td>
<td data-org-content="北京" data-nodeid="122">北京</td>
<td data-org-content="春" data-nodeid="123">春</td>
<td data-org-content="运动水壶" data-nodeid="124">运动水壶</td>
</tr>
<tr data-nodeid="125">
<td data-org-content="3243" data-nodeid="126">3243</td>
<td data-org-content="北京" data-nodeid="127">北京</td>
<td data-org-content="夏" data-nodeid="128">夏</td>
<td data-org-content="运动水壶" data-nodeid="129">运动水壶</td>
</tr>
<tr data-nodeid="130">
<td data-org-content="1121" data-nodeid="131">1121</td>
<td data-org-content="北京" data-nodeid="132">北京</td>
<td data-org-content="秋" data-nodeid="133">秋</td>
<td data-org-content="运动水壶" data-nodeid="134">运动水壶</td>
</tr>
<tr data-nodeid="135">
<td data-org-content="2343" data-nodeid="136">2343</td>
<td data-org-content="北京" data-nodeid="137">北京</td>
<td data-org-content="冬" data-nodeid="138">冬</td>
<td data-org-content="运动水壶" data-nodeid="139">运动水壶</td>
</tr>
</tbody>
</table>
<p data-nodeid="2883">为了能够可视化，这里特意用了三个维度，数据立方体可以看成下面的坐标系：</p>
<p data-nodeid="2884" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/5D/Ciqc1F8-JA-AbjJsAAAdKtXA4ko197.png" alt="Drawing 0.png" data-nodeid="2888"></p>


<p data-nodeid="142">观察数据会发现 sales_area 的值有两个：北京、四川；sales_season 的值有四个：春、夏、秋、冬；sales_item 的值有两个：运动水壶和自行车。我们可以将这些值看成坐标轴上的刻度，则这个大的立方体就可以被分割为  2 * 4 * 2 个子立方体。</p>
<p data-nodeid="143">基于数据立方体的维度，我们就可以进行多维分析，比如分析季节维度下的均值、总量等等。具体的实现则是用 SQL 的 GROUP BY 子句 + 对应的聚合函数完成，这其实也是多维分析中常见的上卷操作。</p>
<p data-nodeid="4529">多维分析中常见的操作包括： 切片，切块，旋转，上卷，下钻。我们 以下面这个数据立方体为例，来看下它们分别指的是什么：</p>
<p data-nodeid="4530" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/69/CgqCHl8-JBmAee0iAACH5lN-R8o463.png" alt="Drawing 1.png" data-nodeid="4534"></p>


<ul data-nodeid="6173">
<li data-nodeid="6174">
<p data-nodeid="6175"><strong data-nodeid="6181">切片（Slice）和切块（Dice）</strong>：如下图所示，切片是在数据立方体的某一维度上选定一个维成员，而切块是对两个或多个维执行进行选择。</p>
</li>
</ul>
<p data-nodeid="6176" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/5D/Ciqc1F8-JCCAbFvHAAIDPyTGbH4730.png" alt="Drawing 2.png" data-nodeid="6184"></p>


<ul data-nodeid="7823">
<li data-nodeid="7824">
<p data-nodeid="7825"><strong data-nodeid="7831">旋转（Pivot）</strong>：如下图所示，旋转指改变报表或页面的展示方向。对于使用者来说，就是个视图操作，而从 SQL 模拟语句的角度来说，就是改变 SELECT 后面字段的顺序而已。</p>
</li>
</ul>
<p data-nodeid="7826" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/69/CgqCHl8-JCmAY6rGAALSpvUMeew481.png" alt="Drawing 3.png" data-nodeid="7834"></p>


<ul data-nodeid="9475">
<li data-nodeid="9476">
<p data-nodeid="9477"><strong data-nodeid="9487">下钻（Drill-down）和上卷（Roll-up）</strong>：下钻指将某些维度进行细分， 比如将省份维度下钻到城市维度，或是将时间维度从季度的粒度下钻到月份，如下图中将时间维度细分为 4 月、5 月、6 月。上卷可以理解为"无视"某些维度，比如只基于省份和品类进行聚合分析，如下图所示。</p>
</li>
</ul>
<p data-nodeid="9478" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/5D/Ciqc1F8-JDGAOiUwAAHX2wAO64U067.png" alt="Drawing 4.png" data-nodeid="9490"></p>


<p data-nodeid="158">基于数据仓库的多维分析通常广泛运用在商业智能系统中，下面我们来看看商业智能系统。</p>
<h3 data-nodeid="159">商业智能系统</h3>
<p data-nodeid="160">哪里有数据，哪里就有数据挖掘应用，这句话用来形容商业智能再合适不过了。数据仓库解决了存储问题，而 OLAP 技术提供了挖掘手段，企业自然而然会想到将数据利用起来，商业智能就是最好的途径。</p>
<p data-nodeid="161">**商业智能（Business Intelligence，BI）**是一个统称，指的是用于支持制定业务决策的技能、流程、技术、应用和实践。商业智能是将企业中现有的数据转化为知识，以帮助企业做出明智的业务经营决策的工具。具体来说，它通过对商业信息进行搜集、管理和分析，旨在使企业的各级决策者获得知识或洞察力（Insight），从而促使他们做出对企业更有利的决策。从技术层面上讲，商业智能不是什么新技术，它只是数据仓库、OLAP 等技术的综合运用。</p>
<p data-nodeid="162">大多数的数据仓库是为了挖掘某种商业价值而创建的，但是商业智能和数据仓库之间的区别在于，商业智能的定位是生成可向业务用户交付的产品，而数据仓库只对数据进行结构化的存储和组织，所以数据仓库需要 OLAP 技术，才能向商业智能转换。</p>
<p data-nodeid="11131">下图演示了商业智能系统、数据仓库和 OLAP 的关系：</p>
<p data-nodeid="11132" class=""><img src="https://s0.lgstatic.com/i/image/M00/44/5D/Ciqc1F8-JDyAPHFNAADqLJgGMks099.png" alt="1.png" data-nodeid="11136"></p>


<p data-nodeid="165">可以看出，商业智能系统通过对数据仓库的数据进行数据选择、抽取、加载后，使用数据挖掘方法提取知识，再用 BI 报表将知识呈现给决策者供其参考。</p>
<p data-nodeid="166">一款优秀的商业智能系统应该满足以下四个特性：<strong data-nodeid="402">准确</strong>、<strong data-nodeid="403">及时</strong>、<strong data-nodeid="404">价值高</strong>和<strong data-nodeid="405">可操作</strong>。准确性的意义是数据是可信的，及时性意味着数据可定期获取，价值高表示对商业用户有用，可操作性是指信息可以用于业务决策过程。</p>
<h3 data-nodeid="167">总结</h3>
<p data-nodeid="168">本课时详细讲解了后续实战项目中会用到的概念，但是并没有面面俱到，还有星形模型、雪花模型、数据仓库建模方法论等概念，详细讲解需要太多的理论延伸，对于我们后面的项目来说没有必要，但如果你有兴趣，可以多钻研。</p>
<p data-nodeid="169">不难看出，由于数据集市需要面向业务并且实时分析，数据立方体的结构非常适合它，而商业智能系统的中心正是能够支持 OLAP 分析、报表查询的数据集市。在后面我们会按照这种思路深入实战学习，本课时中最重要的是理解数据立方体和多维分析方法，这也是本节课的<strong data-nodeid="413">课后思考题</strong>。</p>

---

### 精选评论


