<p data-nodeid="50897" class="">关于 React Hooks 还有一个高概率命中的题，就是聊设计模式。跟组件一样，如何协调地组合相关的代码是一个强需求，所以设计模式是一个不可避免的问题。</p>
<h3 data-nodeid="50898">审题</h3>
<p data-nodeid="50899">但我个人感觉，这是一个处于只能谈一谈，或者说聊一聊的问题。这是为什么呢？自然有它的原因。</p>
<p data-nodeid="50900">（1）Hooks 整体发展时间不长。</p>
<p data-nodeid="50901">从 18 年推出 Hooks 到现在，整体时间不算长，从成熟度上来讲还不能够与类组件的开发模式相提并论，有些问题还没得到解决。比如在官方文档中给出了获取上一轮 props 或 state 的方案，代码如下所示：</p>
<pre class="lang-java" data-nodeid="50902"><code data-language="java">function Counter() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  useEffect(() =&gt; {
    prevCountRef.current = count;
  });
  const prevCount = prevCountRef.current;
  return &lt;h1&gt;Now: {count}, before: {prevCount}&lt;/h1&gt;;
}
</code></pre>
<p data-nodeid="50903">在这段代码里，先是通过 useRef 函数生成一个 ref，然后将当前的 count 写入 ref.current 作为缓存，以此保证每次能够获取上一轮的 state。</p>
<p data-nodeid="50904">这个设计使用了些许小技巧，虽然能解决问题，但不够直观。虽然官方也提出了，可以将这个过程封装为一个 usePrevious 的 Hooks，社区也提供了这样的 Hooks，但就开发心智而言，确实是个负担。</p>
<p data-nodeid="50905">与此类似的还有在 Hooks 的开发模式下完成 getDerivedStateFromProps 的操作，对心智的挑战也很大。这个时候你会发现有时使用类组件可能更好理解。</p>
<p data-nodeid="50906">（2）Hooks 并不会改变组件本身的设计模式。</p>
<p data-nodeid="50907">比如在第 05 讲“如何设计 React 组件？”提到的展示组件、容器组件、高阶组件等概念，并不会因为使用 Hooks 而改变。因为 Hooks 并不是解决组件如何复用的问题，而是解决内部逻辑抽象复用的问题。以前是通过生命周期的方式思考逻辑如何布局，而现在是以事务的角度归纳合并。所以这个变化对于开发者的心智挑战也很大。</p>
<p data-nodeid="50908">所以在缺乏绝对权威方案的情况下，对于 React Hooks 的设计模式只能是处于谈一谈、聊一聊的开放式状态。对于这类开放式问题，我们依然需要遵循一定的逻辑来陈述自己的观点。最好还能由浅及深，逐步递进地展开自己的想法。</p>
<ul data-nodeid="50909">
<li data-nodeid="50910">
<p data-nodeid="50911">在论述前需要先建立一个认知基础，表明自身对 Hooks 整体的运用理解，这就是“道”；</p>
</li>
<li data-nodeid="50912">
<p data-nodeid="50913">再结合实践谈技巧上的常规操作与个人实践，这就是“术”；</p>
</li>
<li data-nodeid="50914">
<p data-nodeid="50915">最后呢，结合自身经验聊一聊 Hooks 在工程实践中的组合方式，这就是“势”。</p>
</li>
</ul>
<p data-nodeid="50916">当然，这并不是唯一的答案，只是提供了一个思路，要注意，只要是逻辑自洽的答案都是好答案。</p>
<h3 data-nodeid="50917">承题</h3>
<p data-nodeid="50918">我们就可以根据上文分析到的“道、术、势”来答题了：道就是认知基础；术就是常规操作与个人实践；势就是工程实践上的运用。</p>
<p data-nodeid="50919"><img src="https://s0.lgstatic.com/i/image2/M01/08/34/CgpVE2AKhZCAHGhGAABO7J0ONKw763.png" alt="Drawing 1.png" data-nodeid="50987"></p>
<h3 data-nodeid="50920">破题</h3>
<h4 data-nodeid="50921">道</h4>
<p data-nodeid="50922">Dan 在 React Hooks 的介绍中曾经说过：“忘记生命周期，以 effects 的方式开始思考”，这样的思维转换，能帮助我们更好地处理代码。</p>
<p data-nodeid="50923">下面还是用一个聊天组件来举例，代码如下所示：</p>
<pre class="lang-java" data-nodeid="50924"><code data-language="java"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ChatChannel</span> <span class="hljs-keyword">extends</span> <span class="hljs-title">Component</span> </span>{
  state = {
    messages: [];
  }
  componentDidMount() {
    <span class="hljs-keyword">this</span>.subscribeChannel(<span class="hljs-keyword">this</span>.props.channelId);
  }

  componentDidUpdate(prevProps) {
    <span class="hljs-keyword">if</span> (<span class="hljs-keyword">this</span>.props.channelId !== prevProps.channelId) {
      <span class="hljs-keyword">this</span>.unSubscribeChannel(prevProps.channelId);
      <span class="hljs-keyword">this</span>.subscribeChannel(<span class="hljs-keyword">this</span>.props.channelId);
    }
  }
  componentWillUnmount() {
    <span class="hljs-keyword">this</span>.unSubscribeChannel(<span class="hljs-keyword">this</span>.props.channelId);
  }


  subscribeChannel = (channelId) =&gt; {
    ChatAPI.subscribe(
      channelId,
      message =&gt; {
        <span class="hljs-keyword">this</span>.setState(state =&gt; {
          <span class="hljs-keyword">return</span> { messages: [...state.messages, message] };
        });
      }
    );
  }

  unSubscribeChannel = (channelId) =&gt; {
     ChatAPI.unSubscribe(channelId);
  }
  render() {
    <span class="hljs-comment">// 组件样式</span>
    <span class="hljs-comment">// ...</span>
  }
}
</code></pre>
<p data-nodeid="50925">这个组件命名为<strong data-nodeid="51001">ChatChannel</strong>，它的作用是<strong data-nodeid="51002">根据传入的 channelId 订阅相关 channel 的信息</strong>，也就是 messages，并展示内容。因为展示界面 UI 不是要说明的重点，所以这里的代码就暂时省略了。</p>
<p data-nodeid="50926">其中用于订阅 channel 的函数是 subscribeChannel，用于取消订阅 channel 的函数是 unSubscribeChannel，内部都是调用 ChatAPI 实现。</p>
<p data-nodeid="50927">根据生命周期的思路，在 componentDidMount 去订阅 channel，在 componentWillUnmount 取消。那如果外部传入的 channelId 变化了呢？这里通过 componentDidUpdate 去处理，先取消上一次订阅的 channel，再订阅新的 channel。你会发现整个逻辑不只我讲起来拗口，代码看起来也费劲。</p>
<p data-nodeid="50928">如果从生命周期转换到 Hooks 去思考，这段代码将会被大量简化。因为你只需要抓住一个点就够了，那就是 channelId 与 channel 一一对应关联，channelId 切换时，自动取消订阅。写成的代码就像下面这样：</p>
<pre class="lang-java" data-nodeid="50929"><code data-language="java"><span class="hljs-keyword">const</span> ChatChannel = ({ channelId }) =&gt; {
  <span class="hljs-keyword">const</span> [messages, setMessages] = useState([]);
  useEffect(() =&gt; {
   ChatAPI.subscribe(
      channelId,
      message =&gt; {
        <span class="hljs-keyword">this</span>.setState(state =&gt; {
          <span class="hljs-keyword">return</span> { messages: [...state.messages, message] };
        });
      }
    );
    <span class="hljs-keyword">return</span> () =&gt; ChatAPI.unSubscribe(channelId));
  }, [channelId]);

  <span class="hljs-comment">// 组件样式</span>
  <span class="hljs-keyword">return</span> ...
}
</code></pre>
<p data-nodeid="50930">整个逻辑只需要一个 effect 就可以完成了，订阅的操作是 useEffect 的第一个函数，这个函数的返回值是一个取消订阅的函数。useEffect 的第二个参数是 channelId，当 channelId 变化时会优先执行取消订阅的函数，再执行订阅。</p>
<p data-nodeid="50931">经过 Hooks 的改造后，只需要一个 effect 就完整实现了需要多个生命周期函数才能完成的业务逻辑。如果希望进一步抽象与复用，只需要将 effect 中的代码抽为自定义 Hook 就行了。这远比生命周期来得简单。</p>
<h4 data-nodeid="50932">术</h4>
<p data-nodeid="50933">接下来将介绍一些 Hooks 中容易遇到的问题。</p>
<p data-nodeid="50934"><strong data-nodeid="51013">（1）React.memo vs React.useMemo</strong></p>
<p data-nodeid="50935">在第 04 讲<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=566&amp;sid=20-h5Url-0#/detail/pc?id=5794" data-nodeid="51017">“类组件与函数组件有什么区别呢？”</a>中有提到，React.memo 是一个高阶组件，它的效果类似于 React.pureComponent。但在 Hooks 的场景下，更推荐使用 React.useMemo，因为它存在这样一个问题。就像如下的代码一样：</p>
<pre class="lang-java" data-nodeid="50936"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">Banner</span><span class="hljs-params">()</span> </span>{
  let appContext = useContext(AppContext);
  let theme = appContext.theme;
  <span class="hljs-keyword">return</span> &lt;Slider theme={theme} /&gt;
}
export <span class="hljs-keyword">default</span> React.memo(Banner)
</code></pre>
<p data-nodeid="50937">这段代码的意义是这样的，通过 useContext 获取全局的主题信息，然后给 Slider 组件换上主题。但是如果给最外层的 Banner 组件加上 React.memo，那么外部更新 appContext 的值的时候，Slider 就会被触发重渲染。</p>
<p data-nodeid="50938">当然，我们可以通过<strong data-nodeid="51025">分拆组件</strong>的方式阻断重渲染，但使用 React.useMemo 可以实现更精细化的控制。就像下面的代码一样，为 Slider 组件套上 React.useMemo，写上 theme 进行控制。</p>
<pre class="lang-java" data-nodeid="50939"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">Banner</span><span class="hljs-params">()</span> </span>{
  let appContext = useContext(AppContext);
  let theme = appContext.theme;
  <span class="hljs-keyword">return</span> React.useMemo(() =&gt; {
    <span class="hljs-keyword">return</span> &lt;Slider theme={theme} /&gt;;
  }, [theme])
}
export <span class="hljs-keyword">default</span> React.memo(Banner)
</code></pre>
<p data-nodeid="50940">所有考虑到更宽广的使用场景与可维护性，更推荐使用 React.useMemo。</p>
<p data-nodeid="50941"><strong data-nodeid="51030">（2）常量</strong></p>
<p data-nodeid="50942">由于函数组件每次渲染时都会重新执行，所以常量应该放置到函数外部去，避免每次都重新创建。而如果定义的常量是一个函数，且需要使用组件内部的变量做计算，那么一定要使用 useCallback 缓存函数。</p>
<p data-nodeid="50943"><strong data-nodeid="51035">（3）useEffect 第二个参数的判断问题</strong></p>
<p data-nodeid="50944">在设计上它同样是进行浅比较，如果传入的是引用类型，那么很容易会判定不相等，所以尽量不要使用引用类型作为判断条件，很容易出错。</p>
<h4 data-nodeid="50945">势</h4>
<p data-nodeid="50946">你在开发时，想过这样一个问题吗，就是如何将业务逻辑的 Hooks 组合起来？每个团队肯定都有自己的见解和方案，那么下面将举一个出自 The Facade pattern and applying it to React Hooks 的案例来看看如何组合 Hooks。</p>
<p data-nodeid="50947">在这个案例中将 User 的所有操作归到一个自定义 Hook 中去操作，最终返回的值有 users、addUsers 及 deleteUser。其中 users 是通过 useState 获取；addUser 是通过 setUsers 添加 user 完成；deleteUser 通过过滤 userId 完成。代码如下所示：</p>
<pre class="lang-java" data-nodeid="50948"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">useUsersManagement</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;<span class="hljs-keyword">const</span> [users, setUsers] = useState([]);
&nbsp;
&nbsp;&nbsp;<span class="hljs-function">function <span class="hljs-title">addUser</span><span class="hljs-params">(user)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;setUsers([
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;...users,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;user
&nbsp;&nbsp;&nbsp;&nbsp;])
&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;<span class="hljs-function">function <span class="hljs-title">deleteUser</span><span class="hljs-params">(userId)</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span> userIndex = users.findIndex(user =&gt; user.id === userId);
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">if</span> (userIndex &gt; -<span class="hljs-number">1</span>) {
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-keyword">const</span> newUsers = [...users];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;newUsers.splice(userIndex, <span class="hljs-number">1</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;setUsers(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;newUsers
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;<span class="hljs-keyword">return</span> {
&nbsp;&nbsp;&nbsp;&nbsp;users,
&nbsp;&nbsp;&nbsp;&nbsp;addUser,
&nbsp;&nbsp;&nbsp;&nbsp;deleteUser
&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="50949">第二部分是通过 useAddUserModalManagement 这一个自定义 Hook 控制 Modal 的开关。与上面的操作类似。isAddUserModalOpened 表示了当前处于 Modal 开关状态，openAddUserModal 则是打开，closeAddUserModal 则是关闭。如下代码所示：</p>
<pre class="lang-java" data-nodeid="50950"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">useAddUserModalManagement</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;<span class="hljs-keyword">const</span> [isAddUserModalOpened, setAddUserModalVisibility] = useState(<span class="hljs-keyword">false</span>);
&nbsp;
&nbsp;&nbsp;<span class="hljs-function">function <span class="hljs-title">openAddUserModal</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;setAddUserModalVisibility(<span class="hljs-keyword">true</span>);
&nbsp;&nbsp;}
&nbsp;
&nbsp;&nbsp;<span class="hljs-function">function <span class="hljs-title">closeAddUserModal</span><span class="hljs-params">()</span> </span>{
&nbsp;&nbsp;&nbsp;&nbsp;setAddUserModalVisibility(<span class="hljs-keyword">false</span>);
&nbsp;&nbsp;}
&nbsp;&nbsp;<span class="hljs-keyword">return</span> {
&nbsp;&nbsp;&nbsp;&nbsp;isAddUserModalOpened,
&nbsp;&nbsp;&nbsp;&nbsp;openAddUserModal,
&nbsp;&nbsp;&nbsp;&nbsp;closeAddUserModal
&nbsp;&nbsp;}
}
</code></pre>
<p data-nodeid="50951">最后来看看在代码中运用的情况，引入 useUsersManagement 和 useAddUserModalManagement 两个自定义 Hook，然后在组件 UsersTable 与 AddUserModal 直接使用。UsersTable 直接展示 users 相关信息，通过操作 deleteUser 可以控制删减 User。AddUserModal 通过 isAddUserModalOpened 控制显隐，完成 addUser 操作。代码如下所示：</p>
<pre class="lang-js" data-nodeid="50952"><code data-language="js"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-keyword">import</span> AddUserModal <span class="hljs-keyword">from</span> <span class="hljs-string">'./AddUserModal'</span>;
<span class="hljs-keyword">import</span> UsersTable <span class="hljs-keyword">from</span> <span class="hljs-string">'./UsersTable'</span>;
<span class="hljs-keyword">import</span> useUsersManagement <span class="hljs-keyword">from</span> <span class="hljs-string">"./useUsersManagement"</span>;
<span class="hljs-keyword">import</span> useAddUserModalManagement <span class="hljs-keyword">from</span> <span class="hljs-string">"./useAddUserModalManagement"</span>;
&nbsp;
<span class="hljs-keyword">const</span> Users = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
&nbsp;&nbsp;<span class="hljs-keyword">const</span> {
&nbsp;&nbsp;&nbsp;&nbsp;users,
&nbsp;&nbsp;&nbsp;&nbsp;addUser,
&nbsp;&nbsp;&nbsp;&nbsp;deleteUser
&nbsp;&nbsp;} = useUsersManagement();
&nbsp;&nbsp;<span class="hljs-keyword">const</span> {
&nbsp;&nbsp;&nbsp;&nbsp;isAddUserModalOpened,
&nbsp;&nbsp;&nbsp;&nbsp;openAddUserModal,
&nbsp;&nbsp;&nbsp;&nbsp;closeAddUserModal
&nbsp;&nbsp;} = useAddUserModalManagement();
&nbsp;
&nbsp;&nbsp;<span class="hljs-keyword">return</span> (
&nbsp;&nbsp;&nbsp;&nbsp;<span class="xml"><span class="hljs-tag">&lt;&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">button</span> <span class="hljs-attr">onClick</span>=<span class="hljs-string">{openAddUserModal}</span>&gt;</span>Add user<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">UsersTable</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">users</span>=<span class="hljs-string">{users}</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">onDelete</span>=<span class="hljs-string">{deleteUser}</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;<span class="hljs-name">AddUserModal</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">isOpened</span>=<span class="hljs-string">{isAddUserModalOpened}</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">onClose</span>=<span class="hljs-string">{closeAddUserModal}</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-attr">onAddUser</span>=<span class="hljs-string">{addUser}</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span class="hljs-tag">&lt;/&gt;</span></span>
&nbsp;&nbsp;)
};
&nbsp;
<span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> Users;
</code></pre>
<p data-nodeid="50953">在上面的例子中，我们可以看到组件内部的逻辑已经被自定义 Hook 完全抽出去了。外观模式很接近第 05 讲<a href="https://kaiwu.lagou.com/course/courseInfo.htm?courseId=566&amp;sid=20-h5Url-0#/detail/pc?id=5795" data-nodeid="51045">“如何设计 React 组件？”</a>提到的容器组件的概念，即在组件中通过各个自定义 Hook 去操作业务逻辑。每个自定义 Hook 都是一个独立的子模块，有属于自己的领域模型。基于这样的设计就可以避免 Hook 之间逻辑交叉，提升复用性。</p>
<h3 data-nodeid="50954">答题</h3>
<blockquote data-nodeid="50955">
<p data-nodeid="50956">React Hooks 并没有权威的设计模式，很多工作还在建设中，在这里我谈一下自己的一些看法。</p>
<p data-nodeid="50957">首先用 Hooks 开发需要抛弃生命周期的思考模式，以 effects 的角度重新思考。过去类组件的开发模式中，在 componentDidMount 中放置一个监听事件，还需要考虑在 componentWillUnmount 中取消监听，甚至可能由于部分值变化，还需要在其他生命周期函数中对监听事件做特殊处理。在 Hooks 的设计思路中，可以将这一系列监听与取消监听放置在一个 useEffect 中，useEffect 可以不关心组件的生命周期，只需要关心外部依赖的变化即可，对于开发心智而言是极大的减负。这是 Hooks 的设计根本。</p>
<p data-nodeid="50958">在这样一个认知基础上，我总结了一些在团队内部开发实践的心得，做成了开发规范进行推广。</p>
<p data-nodeid="50959">第一点就是 React.useMemo 取代 React.memo，因为 React.memo 并不能控制组件内部共享状态的变化，而 React.useMemo&nbsp;更适合于 Hooks 的场景。</p>
<p data-nodeid="50960">第二点就是常量，在类组件中，我们很习惯将常量写在类中，但在组件函数中，这意味着每次渲染都会重新声明常量，这是完全无意义的操作。其次就是组件内的函数每次会被重新创建，如果这个函数需要使用函数组件内部的变量，那么可以用 useCallback 包裹下这个函数。</p>
<p data-nodeid="50961">第三点就是 useEffect 的第二个参数容易被错误使用。很多同学习惯在第二个参数放置引用类型的变量，通常的情况下，引用类型的变量很容易被篡改，难以判断开发者的真实意图，所以更推荐使用值类型的变量。当然有个小技巧是 JSON 序列化引用类型的变量，也就是通过 JSON.stringify 将引用类型变量转换为字符串来解决。但不推荐这个操作方式，比较消耗性能。</p>
<p data-nodeid="50962">这是开发实践上的一些操作。那么就设计模式而言，还需要顾及 Hooks 的组合问题。在这里，我的实践经验是采用外观模式，将业务逻辑封装到各自的自定义 Hook 中。比如用户信息等操作，就把获取用户、增加用户、删除用户等操作封装到一个 Hook 中。而组件内部是抽空的，不放任何具体的业务逻辑，它只需要去调用单个自定义 Hook 暴露的接口就行了，这样也非常利于测试关键路径下的业务逻辑。</p>
<p data-nodeid="50963">以上就是我在设计上的一些思考。</p>
</blockquote>
<p data-nodeid="50964"><img src="https://s0.lgstatic.com/i/image2/M01/08/32/Cip5yGAKhcCAKZY2AAB3d7s2Ur4216.png" alt="Drawing 3.png" data-nodeid="51058"></p>
<h3 data-nodeid="50965">总结</h3>
<p data-nodeid="50966">React Hook 是未来仍然需要关注的一个技术点，由于与类组件不同的心智模型，也希望你在开发中能更多地去使用它。你在使用 Hooks 时遇到过什么问题？有什么独特的解决方案？不妨在留言区留言，我将和你一起探讨。</p>
<p data-nodeid="51396">从下一讲开始就进入 React 周边生态相关的面试题目，首先是 React Router。 React Router 是一个非常长寿的库，可以说在 React 生态中处于屹立不倒的位置，笑看其他库流行起来又衰落下去。在下一讲中，我将与你一起探索它的秘密。</p>
<hr data-nodeid="51397">
<p data-nodeid="51398"><a href="https://shenceyun.lagou.com/t/mka" data-nodeid="51406"><img src="https://s0.lgstatic.com/i/image/M00/72/94/Ciqc1F_EZ0eANc6tAASyC72ZqWw643.png" alt="Drawing 2.png" data-nodeid="51405"></a></p>
<p data-nodeid="51399">《大前端高薪训练营》</p>
<p data-nodeid="51400" class="te-preview-highlight">对标阿里 P7 技术需求 + 每月大厂内推，6 个月助你斩获名企高薪 Offer。<a href="https://shenceyun.lagou.com/t/mka" data-nodeid="51411">点击链接</a>，快来领取！</p>

---

### 精选评论

##### *曦：
> React.useRef的使用场景能总结下吗？有没有什么好的文章推荐？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以看一下这一篇 《6 Practical Applications for useRef》（https://medium.com/frontend-digest/6-practical-applications-for-useref-2f5414f4ac68）
感觉总结得比较详细了。

##### *…：
> 复杂的组件是不是不适用hooks，感觉要是state多了，写usestate太麻烦了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 从复杂度来讲，目前的社区发展情况来看，不少新组件是通过 hooks 编写的。从生命周期思维转换到新思维确实是不小的挑战。
关于 state 多了的问题，React 官方文档中有提到「你不必使用多个 state 变量。State 变量可以很好地存储对象和数组，因此，你仍然可以将相关数据分为一组。然而，不像 class 中的 this.setState，更新 state 变量总是替换它而不是合并它。」。

