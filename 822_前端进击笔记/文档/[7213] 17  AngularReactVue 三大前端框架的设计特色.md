<p data-nodeid="18046">在前面第 10 讲中，我介绍了前端框架中的核心能力——模板引擎。其实，除了模板引擎以外，前端框架还提供了很多其他的能力，比如性能优化相关、状态管理相关等。现如今，虽然各式各样的框架层出不穷，但目前稳定排行在前的基本上是这三大热门框架 Angular/React/Vue（排名不分先后）。</p>
<p data-nodeid="18047">对于不同的前端框架来说，各自的设计原理和解决方案都有所不同，开发者可根据自身需要选择合适的前端框架。</p>
<p data-nodeid="18048">今天，我们就来看一下 Angular/React/Vue 三个框架之间的区别、各自的特点和适用场景等</p>
<h3 data-nodeid="18049">Angular/React/Vue 框架对比</h3>
<p data-nodeid="18050">Angular 是一个<strong data-nodeid="18193">应用设计框架与开发平台</strong>，用于创建高效、复杂、精致的单页面应用，提供了前端项目开发较完整的解决方案。</p>
<p data-nodeid="18051">与此相对，React/Vue 则专注于<strong data-nodeid="18199">构建用户界面</strong>，在一定程度上来说是一个 JavaScript 库，不能称之为框架。</p>
<p data-nodeid="18052">由于 React/Vue 都提供了配套的页面应用解决方案和工具库，因此我们常常将它们作为前端框架与 Angular放在一起比较。</p>
<p data-nodeid="18053">实际上，三个框架的关系可以简单用这样的公式表达。</p>
<pre class="lang-java" data-nodeid="18054"><code data-language="java">Angular = React/Vue + 路由库（react-router/vue-router） + 状态管理（Redux/Flux/Mobx/Vuex） + 脚手架/构建（create-react-app/Vue CLI/Webpack） + ...
</code></pre>
<p data-nodeid="18055">我们先来看看 Angular。</p>
<h3 data-nodeid="18056">Angular</h3>
<p data-nodeid="18057">Angular 最初的设计是针对大型应用进行的，上手成本较高，因为开发者需要掌握一些对前端开发来说较陌生的概念，其中包括依赖注入、模块化、脏检查、AOT 等设计。</p>
<h4 data-nodeid="18058">依赖注入</h4>
<p data-nodeid="18059">依赖注入并不是由 Angular 提出的，它是基于依赖倒置的设计原则设计出来的一套机制。</p>
<p data-nodeid="18060">在项目中，依赖注入体现为：项目提供了这样一个注入机制，其中有人负责提供服务、有人负责消耗服务，通过注入机制搭建了提供服务与消费服务之间的接口。对于消费者来说，无须关心服务是否已经被创建并初始化，依赖注入机制会保证服务的可用性。</p>
<p data-nodeid="18061">在这样的机制下，开发者只需要关注如何使用服务，至于这个服务内部的实现是怎样的、它是什么时候被初始化的、它又依赖了怎样的其他服务，都交给了依赖注入机制来处理，不需要操心。</p>
<p data-nodeid="18062">Angular 提供的便是这样一套依赖注入系统，可以及时创建和交付所依赖的服务。Angular 通过依赖注入来帮你更容易地将应用逻辑分解为服务，并让这些服务可用于各个组件中。这便是 Angular 中的依赖注入设计。</p>
<p data-nodeid="18063">前面提到，Angular 的设计是针对大型应用的，使用依赖注入可以轻松地将各个模块进行解耦，模块与模块之间不会有过多的依赖，可以轻松解决大型应用中模块难以管理的难题。所以在 Angular 中，依赖注入配合模块化组织能达到更好的效果。</p>
<h4 data-nodeid="18064">模块化组织</h4>
<p data-nodeid="18065">Angular 模块把组件、指令和管道打包成内聚的功能块，每个模块聚焦一个特性区域、业务领域、工作流或通用工具。</p>
<p data-nodeid="18066">所以我们可以用Angular 模块来自行聚焦某一个领域的功能模块，也可以使用 Angular 封装好的一些功能模块，像表单模块 FormModule、路由模块 RouterModule、Http 模块，等等。</p>
<p data-nodeid="18067">通过依赖注入的方式，我们可以直接在需要的地方引入这些模块并使用。模块的组织结构为树状结构，不同层级的模块功能组成了完整的应用。通过树状的方式来，依赖注入系统可高效地对服务进行创建和销毁，管理各个模块之间的依赖关系。</p>
<p data-nodeid="18068">其中，脏检查机制也由于模块化组织的设计，被诟病的性能问题得以解决。</p>
<h4 data-nodeid="18069">状态更新：脏检查机制</h4>
<p data-nodeid="18070">什么是脏检查呢？在 Angular 中，触发视图更新的时机来自常见的事件如用户交互（点击、输入等）、定时器、生命周期等，大概的过程如下：</p>
<ol data-nodeid="18071">
<li data-nodeid="18072">
<p data-nodeid="18073">在上述时机被触发后，Angular会计算数据的新值和旧值是否有差异；</p>
</li>
<li data-nodeid="18074">
<p data-nodeid="18075">若有差异，Angular 会更新页面，并触发下一次的脏检查；</p>
</li>
<li data-nodeid="18076">
<p data-nodeid="18077">直到新旧值之间不再有差异，或是脏检查的次数达到设定阈值，才会停下来。</p>
</li>
</ol>
<p data-nodeid="18078">由于并不是直接监听数据的变动，同时每一次更新页面之后，很可能还会引起新的值改变，这导致脏检查的效率很低，还可能会导致死循环。虽然 AngularJS 有阈值控制，但也无法避免脏检查机制所导致的低效甚至性能问题。</p>
<p data-nodeid="18079">脏检查机制在设计上存在的性能问题一直被大家诟病，在 Angular2+ 中引入了模块化组织来解决这个问题。由于应用的组织类是树结构的，脏检查会从根组件开始，自上而下对树上的所有子组件进行检查。相比 AngularJS 中的带有环的结构，这样的单向数据流效率更高，而且容易预测，性能上也有不少的提升。除了模块化组织之外，Angular2+ 同时还引入了 NgZone，提升了脏检查的性能。</p>
<p data-nodeid="18080">在 Angular 中除了对脏检查机制进行了性能优化，还提供了其他的优化手段，AOT 编译便是其中一种。</p>
<h4 data-nodeid="18081">用 AOT 进行编译</h4>
<p data-nodeid="18082">我们先来介绍下 AOT 编译和 JIT 编译。</p>
<ul data-nodeid="18083">
<li data-nodeid="18084">
<p data-nodeid="18085">JIT 编译：在浏览器中运行时编译，视图需要花很长时间才能渲染出来，导致运行期间的性能损耗。</p>
</li>
<li data-nodeid="18086">
<p data-nodeid="18087">AOT 编译（预编译）：在构建时编译，使得页面渲染更快，可提高应用性能。</p>
</li>
</ul>
<p data-nodeid="18088">Angular 提供了预编译（AOT）能力，无须等待应用首次编译，以及通过预编译的方式移除不需要的库代码、减少体积，还可以提早检测模板错误。</p>
<p data-nodeid="18089">到此，我介绍了 Angular 中的依赖注入、模块化组织、脏检查机制、AOT 编译，这些是 Angular框架设计中比较核心的概念和解决方案。</p>
<p data-nodeid="18090">除此之外，Angular提供了完备的结构和规范，新加入的成员能很快地通过复制粘贴完成功能的开发。好的架构设计，能让高级程序员和初入门的程序员写出相似的代码，Angular 通过严格的规范约束，提升了项目的维护体验。</p>
<p data-nodeid="18091">由于 Angular 目标是提供大而全的解决方案，因此相比 Angular，React和 Vue则更专注于用户界面的构建和优化，我们继续来看一下 React。</p>
<h3 data-nodeid="18092">React</h3>
<p data-nodeid="18093">React 和 Vue 都是专注于构建用户界面的 JavaSctipt 库，它们不强制引入很多工程类的功能，也没有过多的强侵入性的概念、语法糖和设计，因此它们相对 Angular 最大的优势是轻量。</p>
<p data-nodeid="18094">而对比 Vue，React 最大的优点是灵活，对原生 JavaScript 的侵入性弱（没有过多的模板语法），不需要掌握太多的API 也可以很好地使用。</p>
<p data-nodeid="18095">React的哲学是：React 是用JavaScript 构建快速响应的大型 Web应用程序的首选方式。</p>
<p data-nodeid="18096">接下来，我们来看看React 中的一些核心设计和特色，首选便是虚拟 DOM的设计。</p>
<h4 data-nodeid="18097">虚拟 DOM</h4>
<p data-nodeid="18098">虚拟 DOM 方案的出现，主要为了解决前端页面频繁渲染过程中的性能问题。该方案最初由 React 提出，如今随着机器性能的提升、框架之间的相互借鉴等，在其他框架（比如 Vue）中也都有使用。</p>
<p data-nodeid="18099">虚拟 DOM的设计，大概可分成 3 个过程，下面我们分别来看看。</p>
<p data-nodeid="23758" class=""><strong data-nodeid="23762">1. 用JavaScript 对象模拟 DOM 树，得到一棵虚拟 DOM 树。</strong></p>


<p data-nodeid="18103">不知道你是否仔细研究过 DOM 节点对象，一个真正的DOM 元素非常庞大，拥有很多的属性值。一个 DOM 节点包括特别多的属性、元素和事件对象，但实际上我们会用到的可能只有其中很小一部分，比如节点内容、元素位置、样式、节点的添加删除等方法。</p>
<p data-nodeid="18104">所以，我们通过用 JavaScript 对象来表示 DOM 元素的方式，该对象仅包括常用的这些属性方法和节点关系，这样就可以大大降低对象内存、虚拟 DOM 差异对比的计算量等。</p>
<p data-nodeid="22564" class=""><strong data-nodeid="22568">2. 当页面数据变更时，生成新的虚拟 DOM 树，比较新旧两棵虚拟 DOM 树的差异。</strong></p>






<p data-nodeid="18108">当我们用 JavaScript 对象来模拟 DOM 节点之后，可以构造出虚拟 DOM 树。</p>
<p data-nodeid="18109">当发生状态变更的时候，可以重新构造一棵新的 JavaScript 对象 DOM 树。通过将新的模拟 DOM 树和旧的模拟 DOM 树进行比较，得到两棵树的差异，并记录下来。在比较之后，我们可以获得这样的差异：</p>
<ul data-nodeid="18110">
<li data-nodeid="18111">
<p data-nodeid="18112">需要替换掉原来的节点；</p>
</li>
<li data-nodeid="18113">
<p data-nodeid="18114">移动、删除、新增子节点；</p>
</li>
<li data-nodeid="18115">
<p data-nodeid="18116">修改了节点的属性；</p>
</li>
<li data-nodeid="18117">
<p data-nodeid="18118">对于文本节点的文本内容改变。</p>
</li>
</ul>
<p data-nodeid="24353" class=""><img src="https://s0.lgstatic.com/i/image6/M01/41/CE/Cgp9HWCuGOGAJPh4AABjO4iTOhE480.png" alt="image.png" data-nodeid="24356"></p>

<p data-nodeid="18120">如图所示，我们对比了两棵基于<code data-backticks="1" data-nodeid="18258">&lt;div&gt;</code>元素的DOM 树，得到的差异有：</p>
<ul data-nodeid="18121">
<li data-nodeid="18122">
<p data-nodeid="18123">p 元素插入了一个 span 元素子节点；</p>
</li>
<li data-nodeid="18124">
<p data-nodeid="18125">原先的文本节点挪到了 span 元素子节点下面；</p>
</li>
</ul>
<p data-nodeid="18126">经过差异对比之后，我们能获得一组差异记录，接下来我们需要使用它。</p>
<p data-nodeid="18127"><strong data-nodeid="18266">3. 把差异应用到真正的 DOM 树上</strong></p>
<p data-nodeid="18128">要实现最终的页面渲染，需要进行一些 JavaScript 操作，将差异应用到真正的DOM 树上，例如节点的替换、移动、删除，文本内容的改变等。</p>
<p data-nodeid="18129">使用这样的方式来更新页面，可以将页面的DOM 变更范围减到最小，同时通过将多个状态变更合并计算，可以降低页面的更新频率。因此，使用虚拟 DOM，可以有效降低浏览器计算和性能。</p>
<p data-nodeid="18130">虽然虚拟 DOM 解决了页面被频繁更新和渲染带来的性能问题，但传统虚拟 DOM 依然有以下性能瓶颈：</p>
<ul data-nodeid="18131">
<li data-nodeid="18132">
<p data-nodeid="18133">在单个组件内部依然需要遍历该组件的整个虚拟 DOM 树；</p>
</li>
<li data-nodeid="18134">
<p data-nodeid="18135">在一些组件整个模版内只有少量动态节点的情况下，这些遍历都是性能的浪费；</p>
</li>
<li data-nodeid="18136">
<p data-nodeid="18137">递归遍历和更新逻辑容易导致 UI 渲染被阻塞，用户体验下降。</p>
</li>
</ul>
<p data-nodeid="18138">对此，React 框架也有进行相应的优化：<strong data-nodeid="18278">使用任务调度来控制状态更新的计算和渲染</strong>。</p>
<h4 data-nodeid="18139">状态更新：任务调度</h4>
<p data-nodeid="18140">React 中使用协调器（Reconciler）与渲染器（Renderer）来优化页面的渲染性能。</p>
<p data-nodeid="24951" class="">在 React 里，可以使用<code data-backticks="1" data-nodeid="24953">ReactDOM.render</code>/<code data-backticks="1" data-nodeid="24955">this.setState</code>/<code data-backticks="1" data-nodeid="24957">this.forceUpdate</code>/<code data-backticks="1" data-nodeid="24959">useState</code>等方法来触发状态更新，这些方法共用一套状态更新机制，该更新机制主要由两个步骤组成。</p>

<ol data-nodeid="25555">
<li data-nodeid="25556">
<p data-nodeid="25557"><strong data-nodeid="25571">找出变化的组件，每当有更新发生时，协调器会做如下工作：</strong></p>
</li>
</ol>
<ul data-nodeid="25558">
<li data-nodeid="25559">
<p data-nodeid="25560" class="te-preview-highlight">调用组件<code data-backticks="1" data-nodeid="25573">render</code>方法将 JSX 转化为虚拟 DOM；</p>
</li>
<li data-nodeid="25561">
<p data-nodeid="25562">进行虚拟 DOM Diff 并找出变化的虚拟 DOM；</p>
</li>
<li data-nodeid="25563">
<p data-nodeid="25564">通知渲染器。</p>
</li>
</ul>
<ol start="2" data-nodeid="25565">
<li data-nodeid="25566">
<p data-nodeid="25567"><strong data-nodeid="25580">渲染器接到协调器通知，将变化的组件渲染到页面上。</strong></p>
</li>
</ol>



<p data-nodeid="18155">在 React15 及以前，协调器创建虚拟 DOM 使用的是递归的方式，该过程是无法中断的。这会导致 UI 渲染被阻塞，造成卡顿。</p>
<p data-nodeid="18156">为此，React16 中新增了调度器（Scheduler），调度器能够把可中断的任务切片处理，能够调整优先级，重置并复用任务。调度器会根据任务的优先级去分配各自的过期时间，在过期时间之前按照优先级执行任务，可以在不影响用户体验的情况下去进行计算和更新。</p>
<p data-nodeid="18157">通过这样的方式，React 可在浏览器空闲的时候进行调度并执行任务，篇幅关系这里不再展开。</p>
<p data-nodeid="18158"><strong data-nodeid="18310">虚拟DOM和任务调度的状态更新机制</strong>，是 React 中性能优化的两个重要解决方案。</p>
<p data-nodeid="18159">除了性能优化以外，React 的出现同时带来了特别棒的理念和设计，包括 jsx、函数式编程、Hooks等。其中，函数式编程的无副作用等优势向来被很多程序员所推崇，Hooks 的出现更是将 React 的函数式编程理念推向了更高峰。</p>
<p data-nodeid="18160">相比于 Angular，React 的入门门槛要低很多，但提到简单易学，就不得不说到 Vue了。</p>
<h3 data-nodeid="18161">Vue</h3>
<p data-nodeid="18162">Vue 最大的特点是上手简单，框架的设计和文档对新手极其友好。但这并不代表它只是个简单的框架，当你需要实现一些更加深入的自定义功能时（比如自定义组件、自定义指令、JSX 等），你会发现它也提供了友好的支持能力。</p>
<p data-nodeid="18163">很多人会认为 Vue 只是把 Angular 和 React 的优势结合，但 Vue 也有自身的设计和思考特色。这里，我们同样介绍一下 Vue 的设计特点。</p>
<h4 data-nodeid="18164">虚拟 DOM</h4>
<p data-nodeid="18165">前面我们在介绍 React的虚拟 DOM时，提到传统虚拟 DOM的性能瓶颈，Vue 3.0 同样为此做了些优化。</p>
<p data-nodeid="18166">在 Vue 3.0 中，虚拟 DOM通过动静结合的模式来进行突破：</p>
<ul data-nodeid="18167">
<li data-nodeid="18168">
<p data-nodeid="18169">通过模版静态分析生成更优化的虚拟 DOM 渲染函数，将模版切分为块（<code data-backticks="1" data-nodeid="18320">if</code>/<code data-backticks="1" data-nodeid="18322">for</code>/<code data-backticks="1" data-nodeid="18324">slot</code>)；</p>
</li>
<li data-nodeid="18170">
<p data-nodeid="18171">更新时只需要直接遍历动态节点，虚拟 DOM的更新性能与模版大小解耦，变为与动态节点的数量相关。</p>
</li>
</ul>
<p data-nodeid="18172">可以简单理解为，虚拟 DOM 的更新从以前的整体作用域调整为树状作用域，树状的结构会带来算法的简化以及性能的提升。</p>
<h4 data-nodeid="18173">状态更新： getter/setter、Proxy</h4>
<p data-nodeid="18174">在 Vue 3.0 以前，Vue中状态更新实现主要依赖了<code data-backticks="1" data-nodeid="18330">getter/setter</code>，在数据更新的时候就执行了模板更新、<code data-backticks="1" data-nodeid="18332">watch</code>、<code data-backticks="1" data-nodeid="18334">computed</code>等一些工作。</p>
<p data-nodeid="18175">相比于之前的<code data-backticks="1" data-nodeid="18337">getter/setter</code>监控数据变更，Vue 3.0 将会是基于<code data-backticks="1" data-nodeid="18339">Proxy</code>的变动侦测，通过代理的方式来监控变动，整体性能会得到优化。当我们给某个对象添加了代理之后，就可以改变一些原有的行为，或是通过钩子的方式添加一些处理，用来触发界面更新、其他数据更新等也是可以的。</p>
<p data-nodeid="18176">对比 Angular，Vue 更加轻量、入门容易。对比 React，Vue 则更专注于模板相关，提供了便利和易读的模板语法，开发者熟练掌握这些语法之后，可快速高效地搭建起前端页面。同时，Vue也在不断地进行自我演化，这些我们也能从 Vue 3.0 的响应式设计、模块化架构、更一致的API 设计等设计中观察到。</p>
<h3 data-nodeid="18177">小结</h3>
<p data-nodeid="18178">前端框架很大程度地提升了前端的开发效率，同时提供了各种场景下的解决方案，使得前端开发可以专注于快速拓展功能和业务实现。</p>
<p data-nodeid="18179">今天我主要介绍了如今比较热门的三个前端框架：Angular/React/Vue。其中，Angular 属于适合大型前端项目的“大而全”的框架，而 React/Vue 则是轻量灵活的库，它们各有各的设计特点。</p>
<p data-nodeid="18180">对于框架的升级，React 选择了渐进兼容的技术方案，而 Angular/Vue 都曾经历过“断崖式”的版本升级。</p>
<p data-nodeid="18181">最后，希望你可以思考下：Angular/Vue 在版本升级过程中，为了更彻底地抛弃历史债务而选择了不兼容，这样的做法你们是怎么看待的呢？在我们的业务开发过程中，是否也会存在这样的情况呢？</p>

---

### 精选评论

##### **用户7884：
> 在学校的时候一直用Vue开发项目，年初实习的时候需要学习react。个人觉得Vue相对来说确实要轻量得多，但是react感觉更规范。比如redux和vuex，vue在业务代码中还是可以直接访问到store甚至进行修改的，只是官方不推荐这么做。但是react不行，更新state还是用新的state去覆盖掉旧的state。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 嗯，代码规范其实都有两面性，要求严格的地方灵活性和便利性可能会下降，要求松散的地方如果遇上多人开发的情况，由于编码习惯的不一样，代码可能就会很混乱

