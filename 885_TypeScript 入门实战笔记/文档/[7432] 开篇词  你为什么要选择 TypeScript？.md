<p data-nodeid="1081" class="">你好，我是乾元，在 10 年的前端开发工作中，我曾就职于去哪儿、搜狗等大厂。作为核心成员，我曾负责过多个前端框架、组件库、开源项目核心模块的开发和维护，还为知名 MVVM 框架 Avalon 核心模块贡献过十多个 Commit。</p>
<p data-nodeid="1082">2018 年起，我正式推动 TypeScript 在部门级业务方向的全面应用。至今，我构建了 TypeScript + React、Redux、Nest.js 的全栈技术生态，并积累了丰富的 TypeScript 设计开发经验。同时，我还从 0-1 打造了一支全栈技术架构团队，目前该团队已平稳支撑了公司数百个业务项目。</p>
<p data-nodeid="1083">我是在什么样的机缘巧合下与 TypeScript 相逢的呢？这不得不提起当时的一场面试经历。</p>
<h3 data-nodeid="1084">一次激发我好奇心的面试经历</h3>
<p data-nodeid="1085">2018 年的一场社招面试中，我见到了一个来自微软的 C# 技术栈转前端的候选人。</p>
<p data-nodeid="1086">当时，前端的面试套路中必然包含 JavaScript 隐式类型转换的知识点，如下代码所示：</p>
<pre class="lang-java" data-nodeid="1087"><code data-language="java">[] == <span class="hljs-string">''</span>
</code></pre>
<p data-nodeid="1088">而这个候选人的回答让我很是诧异，一个从国际大厂出来的面试者，似乎并没有掌握 JavaScript 隐式类型转换的规则。</p>
<blockquote data-nodeid="1089">
<p data-nodeid="1090"><strong data-nodeid="1165">注意</strong>：隐式类型转换的规则是当 == 操作符两侧的值不满足恒等时（===），则先将空数组转换为字符串类型，然后再进行恒等比较。</p>
</blockquote>
<p data-nodeid="1091">我好奇地问他：“难道平时你都不关注这些基础知识？”</p>
<p data-nodeid="1092">他回答：“虽然平时使用 TypeScript，但是并不需要关注这些规则。”</p>
<p data-nodeid="1093">虽然这场面试并不算成功，但激发了我的好奇心：<strong data-nodeid="1173">TypeScript 真的能将我们从隐式类型转换等 JavaScript 的各种坑中拯救出来</strong>？</p>
<p data-nodeid="1094">于是，我开始在业务应用中尝试引入 TypeScript。通过使用静态类型约束 React 组件 Props 和 State，我发现它与使用 JavaScript 相比，不仅支持在任何地方直观地获取组件的接口定义，还能对属性、状态中的值是否为空进行自动检测并给出提示（容错处理），甚至还支持对 React JSX 元素接收的各种属性、方法的检测和提示。</p>
<p data-nodeid="1095"><strong data-nodeid="1179">这样看来 TypeScript 实在是太香了，这让我萌生了在接口调用、Redux 代码中全面引入 TypeScript 的想法</strong>。</p>
<p data-nodeid="1096">2018 年中，我开始做 To B 应用。考虑到 To B 应用的业务逻辑及其复杂性，它对代码的稳定性、易读性、可维护性要求极高，而这正高度契合 TypeScript 的优势。于是，<strong data-nodeid="1185">我正式开始推广全栈式 TypeScript 技术方案</strong>。</p>
<p data-nodeid="1097">在接下来的两年多时间里，这套技术方案支撑了数百个应用的 Web 端、Node.js 端开发，接受了近百万行业务代码的实践考验。相对于 JavaScript 应用而言，TypeScript 使得许多低级的 Bug 在开发阶段就能被检测出来并得到快速解决，显著提升了项目的整体质量和稳定性。</p>
<p data-nodeid="1098">在见证业务发展的同时，我还见证了 TypeScript 版本从 3.0 迭代到 4.1，最终成了极其成熟且强大的语言和工具。其中，它有诸多重量级特性发布：</p>
<ul data-nodeid="1099">
<li data-nodeid="1100">
<p data-nodeid="1101">unknown（3.0）</p>
</li>
<li data-nodeid="1102">
<p data-nodeid="1103">stricter generators（3.6）</p>
</li>
<li data-nodeid="1104">
<p data-nodeid="1105">optional chain（3.7）</p>
</li>
<li data-nodeid="1106">
<p data-nodeid="1107">type-only import &amp; export（3.8）</p>
</li>
<li data-nodeid="1108">
<p data-nodeid="1109">template literal（4.1）</p>
</li>
</ul>
<p data-nodeid="1110">现如今，当我再次回味起那位候选人的回答，才明白 TypeScript 可能压根就不允许这么使用。因为当你写下 [] == ' '，立刻会收到一个红色波浪标注的 ts(2367) 错误提示。</p>
<h3 data-nodeid="1111">TypeScript 有这么好用吗？</h3>
<h4 data-nodeid="1112">1. TypeScript 的本质</h4>
<p data-nodeid="1113">TypeScript 与 JavaScript 本质并无区别，你可以将 TypeScipt 理解为是一个<strong data-nodeid="1212">添加了类型注解的 JavaScript</strong>，比如 const num = 1，它同时符合 TypeScript 和 JavaScript 的语法。</p>
<p data-nodeid="1114">此外，TypeScript 是一门中间语言，最终它还需要转译为纯 JavaScript，再交给各种终端解释、执行。不过，<strong data-nodeid="1217">TypeScript 并不会破坏 JavaScript 既有的知识体系，因为它并未创造迥异于 JavaScript 的新语法，依旧是“熟悉的配方”“熟悉的味道”。</strong></p>
<h4 data-nodeid="1115">2. TypeScript 更加可靠</h4>
<p data-nodeid="1116">在业务应用中引入 TypeScript 后，当我们收到 Sentry（一款开源的前端错误监控系统）告警，关于“'undefined' is not a function”“Cannot read property 'xx' of null|undefined” 之类的低级错误统计信息基本没有。<strong data-nodeid="1235">而这正得益于 TypeScript 的静态类型检测，让至少 10% 的 JavaScript 错误（主要是一些低级错误）能在开发阶段就被发现并解决。</strong></p>
<p data-nodeid="1117">我们也可以这么理解，在所有操作符之前，TypeScript 都能检测到接收的类型（在代码运行时，操作符接收的是实际数据；静态检测时，操作符接收的则是类型）是否被当前操作符所支持。</p>
<p data-nodeid="1118">当 TypeScript 类型检测能力覆盖到整个文件、整个项目代码后，任意破坏约定的改动都能被自动检测出来（即便跨越多个文件、很多次传递），并提出类型错误。因此，你可以<strong data-nodeid="1242">放心地修改、重构业务逻辑，而不用过分担忧因为考虑不周而犯下低级错误</strong>。</p>
<p data-nodeid="1119">接手复杂的大型应用时，TypeScript 能让应用易于维护、迭代，且稳定可靠，也会让你更有安全感。</p>
<h4 data-nodeid="1120">3. 面向接口编程</h4>
<p data-nodeid="1121">编写 TypeScript 类型注解，本质就是接口设计。</p>
<p data-nodeid="1122">以下是使用 TypeScript 设计的一个展示用户信息 React 组件示例，从中我们一眼就能了解组件接收数据的结构和类型，并清楚地知道如何在组件内部编写安全稳定的 JSX 代码。</p>
<pre class="lang-typescript" data-nodeid="1123"><code data-language="typescript"><span class="hljs-keyword">interface</span> IUserInfo {
  <span class="hljs-comment">/** 用户 id */</span>
  id: <span class="hljs-built_in">number</span>;
  <span class="hljs-comment">/** 用户名 */</span>
  name: <span class="hljs-built_in">string</span>;
  <span class="hljs-comment">/** 头像 */</span>
  avatar?: <span class="hljs-built_in">string</span>;
}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">UserInfo</span>(<span class="hljs-params">props: IUserInfo</span>) </span>{
  ...
}
</code></pre>
<p data-nodeid="1124"><strong data-nodeid="1253">TypeScript 极大可能改变你的思维方式，从而逐渐养成一个好习惯</strong>。比如，编写具体的逻辑之前，我们需要设计好数据结构、编写类型注解，并按照这接口约定实现业务逻辑。这显然可以减少不必要的代码重构，从而大大提升编码效率。</p>
<p data-nodeid="1125">同时，你会更明白接口约定的重要性，也会约束自己/他人设计接口、编写注解、遵守约定，乐此不疲。</p>
<h4 data-nodeid="1126">4. TypeScript 正成为主流</h4>
<p data-nodeid="1127">相比竞争对手 Facebook 的 Flow 而言，TypeScript 更具备类型编程的优势，而且还有 Microsoft、Google 这两家国际大厂做背书。</p>
<p data-nodeid="1128">另外，<strong data-nodeid="1268">越来越多的主流框架</strong>（例如 React、Vue 3、Angular、Deno、Nest.js 等）<strong data-nodeid="1269">要么选用 TypeScript 编写源码，要么为 TypeScript 提供了完美的支持</strong>。</p>
<p data-nodeid="1129">随着 TypeScript 的普及，TypeScript 在国内（国内滞后国外）成了一个主流的技术方向，国内各大互联网公司和中小型团队都开始尝试使用 TypeScript 开发项目，且越来越多的人正在学习和使用它。</p>
<p data-nodeid="2420" class="te-preview-highlight">而能够熟练掌握 TypeScript 的开发人员，<strong data-nodeid="2426">将能轻松拿下大厂 Offer</strong>。下面我截取了拉勾招聘官网的一些大厂招聘要求，你可以参考。</p>



<p data-nodeid="1131"><img src="https://s0.lgstatic.com/i/image6/M01/3D/3F/Cgp9HWCToWWAd44HAA6s79WBA5M709.png" alt="Drawing 1.png" data-nodeid="1280"></p>
<h3 data-nodeid="1132">好工具，需要好的学习方式</h3>
<p data-nodeid="1133">TypeScript 在国内成为主流技术方向的时间较晚，大概在 2018 年左右才开始流行。相应地，本土化学习资料也比较匮乏。</p>
<p data-nodeid="1134">2018 年 9 月，在我推广 TypeScript 之际，就连 Babel、create-react-app 官方对 TypeScript 的支持都还不太友好，我主要通过 Medium 等国外资源查阅大量关于 Best Practice of TypeScript、React + TypeScript、Redux + TypeScript 的文章，最终设计并架构了面向部门业务的 TypeScript 技术栈。</p>
<p data-nodeid="1135">伴随着业务的成长，不断有新同学加入，而很多新同学其实并没有任何 TypeScript 开发的经验。那么，如何让更多同学更快速、有效地掌握技术栈，就成了我想解决的关键问题。因此，我又制作了系列培训内容帮助新人快速成长。<strong data-nodeid="1288">本课程正是基于此前的实战经验精炼而成，旨在帮助你快速掌握 TypeScript 技术栈，学会构建高可读性、高稳定性前端应用。</strong></p>
<p data-nodeid="1136">结合<strong data-nodeid="1306">上百个业务应用开发经验</strong>总结，我将课程划分为<strong data-nodeid="1307">入门、进阶、实战</strong>3 个模块，讲解的都是<strong data-nodeid="1308">真实业务场景</strong>中<strong data-nodeid="1309">最实用的知识点，绝非纸上谈兵</strong>。按照知识点的顺序和难易程度，共计 22 讲。</p>
<ul data-nodeid="1137">
<li data-nodeid="1138">
<p data-nodeid="1139"><strong data-nodeid="1313">模块 1：TypeScript 入门</strong></p>
</li>
</ul>
<p data-nodeid="1140">我将介绍 TypeScript 环境搭建，并结合浅显易懂的示例与应用场景讲解 TypeScript 基础类型，也会分享我作为过来人学习 TypeScript 时总结的经验和教训，让你尽量少走弯路，直达重点。<strong data-nodeid="1318">这部分内容是掌握 TypeScript 编程的一块敲门砖，学完之后，你将对 TypeScript 的核心知识和概念有个整体印象。</strong></p>
<ul data-nodeid="1141">
<li data-nodeid="1142">
<p data-nodeid="1143"><strong data-nodeid="1322">模块 2：TypeScript 进阶</strong></p>
</li>
</ul>
<p data-nodeid="1144">主要讲解类型守卫、类型兼容、工具类型等概念，及其在实际业务中的作用和使用技巧，助你快速成长为玩转 TypeScript 高阶开发的“魔法师”。<strong data-nodeid="1327">学完之后，能加深你对进阶知识和工具的理解，并教你掌握造轮子（打造自己的工具类型）进行类型编程的能力。</strong></p>
<ul data-nodeid="1145">
<li data-nodeid="1146">
<p data-nodeid="1147"><strong data-nodeid="1331">模块 3：实战指南</strong></p>
</li>
</ul>
<p data-nodeid="1148">我将结合业务实战经验系统化地讲解 TypeScript Config 配置、TypeScript 常见错误分析定位、浏览器和 Node.js 端开发等知识，以及 JavaScript 项目改造实践。让我们既可以按需定制 TypeScript 类型系统行为和转译产物，还能在碰到官方文档较少提及的各种错误时，快速地对问题进行定位和修复。另外，无论是从零开始的新项目，还是历史遗留的技术债，我们都能有章可循地引入 TypeScript。</p>
<p data-nodeid="1149">并且，<strong data-nodeid="1337">此模块我会穿插分享历经数百个应用开发总结出来的 TypeScript 开发最佳实践经验，助你在业务开发中得心应手地应用 TypeScript，并获得 TypeScript 在 Web 和 Node.js 端最佳开发实践的建议。</strong></p>
<p data-nodeid="1150">TypeScript 发版频繁，特性日新月异，从构思课程到截稿，TypeScript 已经发布了 4.1、4.2 版本（当你读到这段文字，也有可能又发布了 4.3、4.4……），不过核心知识和思想并未过时。<strong data-nodeid="1342">当然，在最后的结束语中，我也会介绍一些新版本比较重要的特性。</strong></p>
<h3 data-nodeid="1151">讲师寄语</h3>
<p data-nodeid="1152">看明白知识点很容易，而难点在于融会贯通。除了关注工程实践，我们更应该关注核心知识点的深入理解和吸收，避免从理论到实践无从着手的无力感，因为<strong data-nodeid="1349">只有吃透其中的原理（生硬的知识点），才能真正打造属于自己的强有力武器</strong>。</p>
<p data-nodeid="1153" class="">本专栏经过精心打磨，并巧妙打磨大量示例，带你轻松学习、有效吸收。你还在犹豫吗？赶紧搭上 TypeScript 学习班车，大厂 Offer 就在前方！</p>

---

### 精选评论

##### **Terry：
> 太牛逼了，国内最权威的TypeScript课程，感谢老师

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油

##### **7980：
> “我发现它与使用 JavaScript 相比，不仅支持在任何地方直观地获取组件的接口定义，还能对属性、状态中的值是否为空进行自动检测并给出提示（容错处理），甚至还支持对 React JSX 元素接收的各种属性、方法的检测和提示”这个是是依赖于编辑器的支持嘛~

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，同时依赖 IDE 的能力；所以笔者推荐 TypeScript + VS Code 组合。真香！

##### 13519022284：
> TS这么重要的课程终于有了

##### **礼理力离：
> 为什么要选择TypeScript，因为只有这个好用。作为javaScript超集，舍我及谁？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 太耿直了

##### **辉：
> 打卡 0

##### *琴：
> 不会ts相当于不会前端

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; TypeScript 现在确实是十分主流的重要的技术方向了！

##### *洋：
> 看到Avalon， 想念它的作者。大家都注意身体健康。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 身体是革命的本钱，大家劳逸结合，加油！

##### console_man：
> 老是，我想问一下，如果现在您再面对开篇中的面试者那样回答，是否会改变你的想法。因为我个人也觉得，隐式转换可以了解，但是不一定非要把全部的规则都记下来。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 现在来看，我会更看重对背后原理的理解，而不是单纯的转化规则，毕竟规则是死的。

##### console_man：
> 用了就是真香！希望大家都积极学习TS，并引入到项目中！

##### *俊：
> ts的类型检测，在编码阶段就能做一些容错检查，这一点非常惊艳；不过添加合适的类型而不是any确实需要大量的学习和积累，老师这个课程很赞

##### selfimpr：
> 老师，react+ts的文章 能推荐一下嘛?

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 哈哈，推荐本课程第 19 讲，会详细涉及到 React + TypeScript，然后我的掘金 https://juejin.cn/user/3913917127735223/posts 或者 tefe 微信公众号上都有讲到 React + TypeScript 的核心的几点内容：组件、Redux、Service 类型化。

##### *浩：
> 一直想系统学习TS，这下机会终于来了。

##### *攀：
> 我用ts的时候遇到类型报错就只会给他一个any类型敷衍😫

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 坏习惯，any 是魔鬼！

##### *振：
> ts 好归好，就是总感觉写起来很麻烦，啥都要先定义，有 java、C# 那种氛围……

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 哈哈TS原本就是作为JS超集出现的，对标是巨人，相对不繁琐。加油

