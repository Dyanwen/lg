<p data-nodeid="22179">18 讲我们学习了如何使用 TypeScript 开发运行 Node.js 端的静态文件服务模块，然而对于大多数的前端人而言，Web 端可能才是主战场。因此，这一讲我们将从 DOM 原生操作和 React 框架这两个方面学习 Web + TypeScript 开发实践。</p>
<blockquote data-nodeid="22180">
<p data-nodeid="22181">学习建议：请按照这一讲中的操作步骤，实践一个完整的开发流程。</p>
</blockquote>
<h3 data-nodeid="22182">DOM 原生操作</h3>
<p data-nodeid="22183">无论我们使用前端框架与否，都免不了需要使用原生操作接口，因此将 TypeScript 与 DOM 原生操作组合起来进行学习很有必要。</p>
<p data-nodeid="22184">接下来，我们通过手写一个简单的待办管理应用来熟悉常见的操作接口。</p>
<h4 data-nodeid="22185">配置项目</h4>
<p data-nodeid="22186">首先，我们可以参照 18 讲中初始化 Node.js 模块的步骤创建一个 todo-web 项目，并安装 TypeScript 依赖。</p>
<p data-nodeid="22187">然后，我们可以按需调整 lib 和 alwaysStrict 参数配置 tsconfig，如下所示：</p>
<pre class="lang-json" data-nodeid="22188"><code data-language="json">{
  "compilerOptions": {
    ...,
    "target": "es5",
    "lib": ["ESNext", "DOM"],                
    "strict": true,                       
    "alwaysStrict": false,
    ...           
  }
}
</code></pre>
<p data-nodeid="22189">在以上配置的第 4 行，我们设置了 tagert 参数是“es5”。在第 5 行，我们设置了 lib 参数为 "ESNext" 和 "DOM"。这样，我们就可以在 TypeScript 中使用最新的语言特性了（比如 Promise.any 等）。</p>
<blockquote data-nodeid="22190">
<p data-nodeid="22191"><strong data-nodeid="22320">注意</strong>：因为设置了 target es5，所以这里我们还需要手动引入 ts-polyfill 为新特性打补丁，以兼容较低版本的浏览器。</p>
</blockquote>
<p data-nodeid="22192">此外，如果我们想在函数中使用 this，则可以把 alwaysStrict 设置为 false，这样生成的代码中就不会有“use strict”（关闭严格模式）了。</p>
<p data-nodeid="22193">配置好项目后，我们开始进行编码实现。</p>
<h4 data-nodeid="22194">编码实现</h4>
<p data-nodeid="22195">首先我们可以创建一个模型 src/model.ts，用来维护待办数据层的增删操作，具体示例如下：</p>
<pre class="lang-typescript" data-nodeid="22196"><code data-language="typescript"><span class="hljs-keyword">class</span> TodoModel {
  <span class="hljs-keyword">private</span> gid: <span class="hljs-built_in">number</span> = <span class="hljs-number">0</span>;
  <span class="hljs-keyword">public</span> add = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-keyword">this</span>.gid++;
  <span class="hljs-keyword">public</span> remove = <span class="hljs-function">(<span class="hljs-params">id: <span class="hljs-built_in">number</span></span>) =&gt;</span> <span class="hljs-built_in">void</span> <span class="hljs-number">0</span>
}
<span class="hljs-keyword">declare</span> <span class="hljs-keyword">var</span> todoModel: TodoModel;
todoModel = <span class="hljs-keyword">new</span> TodoModel;
</code></pre>
<p data-nodeid="22197">在上述示例中，我们定义了模型 TodoModel（示例中仅仅实现了架子，你可以按需丰富这个示例），并在第 7~8 行把模型实例赋值给了全局变量 todoModel。</p>
<p data-nodeid="22198">接下来我们开始实现 src/view.ts，用来维护视图层操作 Dom 逻辑，具体示例如下：</p>
<pre class="lang-typescript" data-nodeid="22199"><code data-language="typescript"><span class="hljs-keyword">const</span> list = <span class="hljs-built_in">document</span>.getElementById(<span class="hljs-string">'todo'</span>) <span class="hljs-keyword">as</span> HTMLUListElement | <span class="hljs-literal">null</span>;
<span class="hljs-keyword">const</span> addButton = <span class="hljs-built_in">document</span>.querySelector&lt;HTMLButtonElement&gt;(<span class="hljs-string">'#add'</span>);
addButton?.addEventListener(<span class="hljs-string">'click'</span>, add);
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">remove</span>(<span class="hljs-params"><span class="hljs-keyword">this</span>: HTMLButtonElement, id: <span class="hljs-built_in">number</span></span>) </span>{
  <span class="hljs-keyword">const</span> todo = <span class="hljs-keyword">this</span>.parentElement;
  todo &amp;&amp; list?.removeChild(todo) &amp;&amp; todoModel.remove(id);
}
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">add</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> id = todoModel.add();
  <span class="hljs-keyword">const</span> todoEle = <span class="hljs-built_in">document</span>.createElement(<span class="hljs-string">'li'</span>);
  todoEle.innerHTML = <span class="hljs-string">`待办 <span class="hljs-subst">${id}</span> &lt;button&gt;删除&lt;/button&gt;`</span>;
  <span class="hljs-keyword">const</span> button = todoEle.getElementsByTagName(<span class="hljs-string">'button'</span>)[<span class="hljs-number">0</span>];
  button.style.color = <span class="hljs-string">'red'</span>;
  <span class="hljs-keyword">if</span> (button) {
    button.onclick = remove.bind(button, id);
  }
  list?.appendChild(todoEle);
}
</code></pre>
<p data-nodeid="22200">上述示例中，我们在 tsconfig 的 lib 参数中添加了 DOM（如果 lib 参数缺省，则默认包含了  DOM；如果显式设置了 lib 参数，那么一定要添加 DOM），TypeScript 便会自动引入内置的 DOM 类型声明（node_modules/typescript/lib/lib.dom.d.ts），这样所有的 DOM 原生操作都将支持静态类型检测。</p>
<p data-nodeid="22201">在第 1 行，我们把通过 id 获取 HTMLElement | null 类型的元素断言为 HTMLUListElement | null，这是因为 HTMLUListElement 是 HTMLElement 的子类型。同样，第 6 行、12 行、14 行的相关元素都也有明确类型。尤其是第 12 行的 createElement、第 14 行的 getElementsByTagName，它们都可以根据标签名返回更确切的元素类型 HTMLLIElement、HTMLButtonElement。</p>
<p data-nodeid="22202">然后，在第 2 行我们通过给 querySelector 指定了明确的类型入参，其获取的元素类型也就变成了更明确的 HTMLButtonElement。</p>
<p data-nodeid="22203">此外，因为 DOM 元素的 style 属性也支持静态类型检测，所以我们在第 15 行可以把字符串 'red' 赋值给 color。但是，如果我们把数字 1 赋值给 color，则会提示一个 ts(2322) 错误。</p>
<p data-nodeid="22204">接下来，我们就可以转译代码，并新建一个 index.html 引入转译后的 lib/model.js、lib/view.js 中，再使用 19 讲中开发的 http-serve CLI 启动服务预览页面。</p>
<p data-nodeid="22205">通过这个简单的例子，我们感受到了 TypeScript 对 DOM 强大的支持，并且官方也根据 JavaScript 的发展十分及时地补齐了新语法特性。因此，即便开发原生应用，TypeScript 也会是一个不错的选择。</p>
<p data-nodeid="22206">接下来，我们将学习 TypeScript 与前端主流框架 React 的搭配使用。</p>
<h3 data-nodeid="22207">React 框架</h3>
<p data-nodeid="22208" class="">React 作为目前非常流行的前端框架，TypeScript 对其支持也是超级完善。在 1.6 版本中，TypeScript 官方专门实现了对 React JSX 语法的静态类型支持，并在 tsconfig 中新增了一个 jsx 参数用来定制 JSX 的转译规则。</p>
<p data-nodeid="22209">而且，React 官方及周边生态对 TypeScript 的支持也越来越完善，比如 create-react-app 支持 TypeScript 模板、babel 支持转译 TypeScript。要知道，在 2018 年我们还需要手动搭建 TypeScript 开发环境，现在通过以下命令即可快速创建 TypeScript 应用，并且还不用过分关心 tsconfig 和开发构建相关的配置，只需把重心放在 React 和 TypeScript 的使用上（坏处则是修改默认配置会比较麻烦）。</p>
<pre class="lang-powershell" data-nodeid="22210"><code data-language="powershell">npm i create<span class="hljs-literal">-react</span><span class="hljs-literal">-app</span> <span class="hljs-literal">-g</span>;
create<span class="hljs-literal">-react</span><span class="hljs-literal">-app</span> my<span class="hljs-literal">-ts</span><span class="hljs-literal">-app</span> -<span class="hljs-literal">-template</span> typescript;
<span class="hljs-built_in">cd</span> my<span class="hljs-literal">-ts</span><span class="hljs-literal">-app</span>;
npm <span class="hljs-built_in">start</span>; // 或者 yarn <span class="hljs-built_in">start</span>
</code></pre>
<p data-nodeid="22211">接下来我们将分别从 Service、Component、状态管理这三个分层介绍 TypeScript 在 React App 开发中的实践。</p>
<h4 data-nodeid="22212">Service 类型化</h4>
<p data-nodeid="22213">首先我们介绍的是 TypeScript 在 Service 层的应用，称之为 Service 类型化，实际就是把 JavaScript 编写的接口调用代码使用 TypeScript 实现。</p>
<p data-nodeid="22214">举个例子， 以下是使用 JavaScript 编写的 getUserById 方法：</p>
<pre class="lang-javascript" data-nodeid="22215"><code data-language="javascript"><span class="hljs-keyword">export</span> <span class="hljs-keyword">const</span> getUserById = <span class="hljs-function"><span class="hljs-params">id</span> =&gt;</span> fetch(<span class="hljs-string">`/api/get/user/by/<span class="hljs-subst">${id}</span>`</span>, { <span class="hljs-attr">method</span>: <span class="hljs-string">'GET'</span> });
</code></pre>
<p data-nodeid="22216">在这个示例中，除了知道参数名 id 以外，我们对该方法接收参数、返回数据的类型和格式一无所知。</p>
<p data-nodeid="22217">以上示例换成 TypeScript 实现后效果如下：</p>
<pre class="lang-typescript" data-nodeid="22218"><code data-language="typescript"><span class="hljs-keyword">export</span> <span class="hljs-keyword">const</span> getUserById = (id: <span class="hljs-built_in">number</span>): <span class="hljs-built_in">Promise</span>&lt;{ id: <span class="hljs-built_in">number</span>; name: <span class="hljs-built_in">string</span> }&gt; =&gt;
  fetch(<span class="hljs-string">`/api/get/user/by/<span class="hljs-subst">${id}</span>`</span>, { method: <span class="hljs-string">'GET'</span> }).then(<span class="hljs-function"><span class="hljs-params">res</span> =&gt;</span> res.json());
<span class="hljs-keyword">async</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">test</span>(<span class="hljs-params"></span>) </span>{
  <span class="hljs-keyword">const</span> { id2, name } = <span class="hljs-keyword">await</span> getUserById(<span class="hljs-string">'string'</span>); <span class="hljs-comment">// ts(2339) ts(2345)</span>
} 
</code></pre>
<p data-nodeid="22219">在使用 TypeScript 的示例中，我们可以清楚地知道 getUserById 方法接收了一个不可缺省、number 类型的参数 id，返回的数据是一个异步的包含数字类型属性 id 和字符串类型属性 name 的对象。而且如果我们错误地调用该方法，比如第 5 行解构了一个不存在的属性 id2，就提示了一个 ts(2339) 错误，入参 'string' 类型不匹配也提示了一个 ts(2345) 错误。</p>
<p data-nodeid="22220">通过两个示例的对比，Service 类型化的优势十分明显。</p>
<p data-nodeid="22221">但是，在实际项目中，我们需要调用的接口少则数十个，多则成百上千，如果想通过手写 TypeScript 代码的方式定义清楚参数和返回值的类型结构，肯定不是一件轻松的事情。此时，我们可以借助一些工具，并基于格式化的接口文档自动生成 TypeScript 接口调用代码。</p>
<p data-nodeid="22222">在业务实践中，前后端需要约定统一的接口规范，并使用格式化的 Swagger 或者 YAPI 等方式定义接口格式，然后自动生成 TypeScript 接口调用代码。目前，这块已经有很多成熟、开源的技术方案，例如<a href="https://swagger.io/tools/swagger-codegen/" data-nodeid="22361">Swagger Codegen</a>、<a href="https://github.com/acacode/swagger-typescript-api" data-nodeid="22365">swagger-typescript-api</a>、<a href="https://gogoyqj.github.io/auto-service/" data-nodeid="22369">Autos</a>、<a href="https://github.com/fjc0k/yapi-to-typescript" data-nodeid="22373">yapi-to-typescript</a>。</p>
<p data-nodeid="22223">此外，对于前后端使用 GraphQL 交互的业务场景，我们也可以使用<a href="https://graphql-code-generator.com/" data-nodeid="22378">GraphQL Code Generator</a>等工具生成 TypeScript 接口调用代码。你可以通过官方文档了解这些自动化工具的更多信息，这里就不做深入介绍了。</p>
<p data-nodeid="22224"><strong data-nodeid="22383">以上提到的 Service 类型化其实并未与 React 深度耦合，因此我们也可以在 Vue 或者其他框架中使用 TypeScript 手写或者基于工具生成接口调用代码。</strong></p>
<p data-nodeid="22225">接下来我们将学习 TypeScript 在 React Component 中的应用，将其称之为 Component 类型化。</p>
<h4 data-nodeid="22226">Component 类型化</h4>
<p data-nodeid="22227">Component 类型化的本质在于清晰地表达组件的属性、状态以及 JSX 元素的类型和结构。</p>
<blockquote data-nodeid="22228">
<p data-nodeid="22229">注意：TypeScript 中有专门的 .tsx 文件用来编写 React 组件，并且不能使用与 JSX 语法冲突的尖括号类型断言（“&lt;类型&gt;”）。此外，我们还需要确保安装了 @types/react、@types/react-dom 类型声明，里边定义了 React 和 ReactDOM 模块所有的接口和类型。</p>
</blockquote>
<p data-nodeid="22230">我们首先了解一下最常用的几个接口和类型。</p>
<p data-nodeid="22231"><strong data-nodeid="22394">（1）class 组件</strong></p>
<p data-nodeid="22232">所有的 class 组件都是基于****React.Component 和 React.PureComponent 基类创建的，下面我们看一个具体示例：</p>
<pre class="lang-typescript" data-nodeid="22233"><code data-language="typescript"><span class="hljs-keyword">interface</span> IEProps {
  Cp?: React.ComponentClass&lt;{ id?: <span class="hljs-built_in">number</span> }&gt;;
}
<span class="hljs-keyword">interface</span> IEState { id: <span class="hljs-built_in">number</span>; }
<span class="hljs-keyword">const</span> ClassCp: React.ComponentClass&lt;IEProps, IEState&gt; = <span class="hljs-keyword">class</span> ClassCp <span class="hljs-keyword">extends</span> React.Component&lt;IEProps, IEState&gt; {
  <span class="hljs-keyword">public</span> state: IEState = { id: <span class="hljs-number">1</span> };
  render() {
    <span class="hljs-keyword">const</span> { Cp } = <span class="hljs-keyword">this</span>.props <span class="hljs-keyword">as</span> Required&lt;IEProps&gt;;
    <span class="hljs-keyword">return</span> &lt;Cp id={<span class="hljs-string">`<span class="hljs-subst">${<span class="hljs-keyword">this</span>.state.id}</span>`</span>} /&gt;; <span class="hljs-comment">// ts(2322)</span>
  }
  <span class="hljs-keyword">static</span> defaultProps: Partial&lt;IEProps&gt; = {
    Cp: <span class="hljs-keyword">class</span> <span class="hljs-keyword">extends</span> React.Component { render = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-literal">null</span> }
  }
}
</code></pre>
<p data-nodeid="22234">在示例中的第 5~14 行，因为 React.Component 基类接收了 IEProps 和 IEState 两个类型入参，并且类型化了 class 组件 E 的 props、state 和 defaultProps 属性，所以如果我们错误地调用了组件 props 中 Cp 属性，第 9 行就会提示一个 ts(2322) 错误。</p>
<p data-nodeid="23163" class="">然后我们可以使用接口类型 React.ComponentClass 来指代所有 class 组件的类型。例如在第 5 行，我们可以把 class 组件 ClassCp 赋值给 React.ComponentClass 类型的变量 ClassCp。</p>

<p data-nodeid="22236" class="">但在业务实践中，我们往往只使用 React.ComponentClass 来描述外部组件或者高阶组件属性的类型。比如在示例中的第 2 行，我们使用了 React.ComponentClass 描述 class 组件 E 的 Cp 属性，而不会像第 5 行那样，把定义好的 class 组件赋值给一个 React.ComponentClass 类型的变量。</p>
<p data-nodeid="22237">此外，在定义 class 组件时，使用 public/private 控制属性/方法的可见性，以及使用Readonly 标记 state、props 为只读，都是特别推荐的实践经验。</p>
<p data-nodeid="22238">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="22239"><code data-language="typescript"><span class="hljs-keyword">class</span> ClassCpWithModifier <span class="hljs-keyword">extends</span> React.Component&lt;Readonly&lt;IEProps&gt;, Readonly&lt;IEState&gt;&gt; {
  <span class="hljs-keyword">private</span> gid: <span class="hljs-built_in">number</span> = <span class="hljs-number">1</span>;
  <span class="hljs-keyword">public</span> state: Readonly&lt;IEState&gt; = { id: <span class="hljs-number">1</span> };
  render() { <span class="hljs-keyword">return</span> <span class="hljs-keyword">this</span>.state.id = <span class="hljs-number">2</span>; } <span class="hljs-comment">// ts(2540)</span>
}
</code></pre>
<p data-nodeid="22240">在示例中的第 2 行，如果我们不希望对外暴露 gid 属性，就可以把它标记为 private 私有。</p>
<p data-nodeid="22241">如果我们想禁止直接修改 state、props 属性，则可以在第 1 行中使用 Readonly 包裹 IEProps、IEState。此时，如果我们在第 4 行直接给 state id 属性赋值，就会提示一个 ts(2540) 错误。</p>
<p data-nodeid="22242"><strong data-nodeid="22412">函数组件</strong></p>
<p data-nodeid="22243">我们可以使用类型 React.FunctionComponent（简写为 React.FC）描述函数组件的类型。因为函数组件没有 state 属性，所以我们只需要类型化 props。</p>
<p data-nodeid="22244">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="22245"><code data-language="typescript">interface IEProps { id?: number; }
const ExplicitFC: React.FC&lt;IEProps&gt; = props =&gt; &lt;&gt;{props.id}&lt;/&gt;; // ok
ExplicitFC.defaultProps = { id: 1 } // ok id must be number
const ExplicitFCEle = &lt;ExplicitFC id={1} /&gt;; // ok id must be number
const ExplicitFCWithError: React.FC&lt;IEProps&gt; = props =&gt; &lt;&gt;{props.id2}&lt;/&gt;; // ts(2399)
ExplicitFCWithError.defaultProps = { id2: 1 } // ts(2332)
const thisIsJSX2 = &lt;ExplicitFCWithError id2={2} /&gt;; // ts(2332)
</code></pre>
<p data-nodeid="23817" class="te-preview-highlight">在上述示例中，因为我们定义了类型是 React.FC<code data-backticks="1" data-nodeid="23819">&lt;IEProps&gt;</code> 的组件 ExplicitFC、ExplicitFCWithError，且类型入参 IEProps 可以同时约束 props 参数和 defaultProps 属性的类型，所以第 2~4 行把 number 类型值赋予接口中已定义的 id 属性可以通过静态类型检测。但是，在第 5~7 行，因为操作了未定义的属性 id2，所以提示了 ts(2399)、 ts(2332) 错误。</p>

<blockquote data-nodeid="22247">
<p data-nodeid="22248">注意：函数组件返回值类型必须是 React.Element（稍后会详细介绍） 或者 null，反过来如果函数返回值类型是 React.Element 或者 null，即便未显式声明类型，函数也是合法的函数组件。</p>
</blockquote>
<p data-nodeid="22249">如以下示例中，因为我们定义了未显式声明类型、返回值分别是 null 和 JSX 的函数 ImplicitFCReturnNull、ImplicitFCReturnJSX，所以第 3 行、第 6 行的这两个组件都可以用来创建 JSX。但是，因为第 8 行定义的返回值类型是 number 的函数 NotAFC，所以被用来创建 JSX 时会在第 9 行提示一个 ts(2786) 错误。</p>
<pre class="lang-typescript" data-nodeid="22250"><code data-language="typescript"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ImplicitFCReturnNull</span>(<span class="hljs-params"></span>) </span>{ <span class="hljs-keyword">return</span> <span class="hljs-literal">null</span>; }
ImplicitFCReturnNull.defaultProps = { id: <span class="hljs-number">1</span> }
<span class="hljs-keyword">const</span> ImplicitFCReturnNullEle = &lt;ImplicitFCReturnNull id={<span class="hljs-number">1</span>} /&gt;; <span class="hljs-comment">// ok id must be number</span>
<span class="hljs-keyword">const</span> ImplicitFCReturnJSX = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> &lt;&gt;&lt;/&gt;;
ImplicitFCReturnJSX.defaultProps = { id2: <span class="hljs-number">1</span> }
<span class="hljs-keyword">const</span> ImplicitFCReturnJSXEle = &lt;ImplicitFCReturnJSX id2={<span class="hljs-number">2</span>} /&gt;; <span class="hljs-comment">// ok</span>
<span class="hljs-comment">/** 分界线 **/</span>
<span class="hljs-keyword">const</span> NotAFC = <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-number">1</span>; <span class="hljs-comment">// </span>
<span class="hljs-keyword">const</span> WithError = &lt;NotAFC /&gt;; <span class="hljs-comment">// ts(2786)</span>
</code></pre>
<p data-nodeid="22251">对于编写函数组件而言，显式注解类型是一个好的实践，另外一个好的实践是用 props 解构代替定义 defaultProps 来指定默认属性的值。</p>
<p data-nodeid="22252">此外，组件和泛型 class、函数一样，也是可以定义成接收若干个入参的泛型组件。</p>
<p data-nodeid="22253">以列表组件为例，<strong data-nodeid="22430">我们希望可以根据列表里渲染条目的类型（比如说“User”或“Todo”），分别使用不同的视图组件渲染条目，这个时候就需要使用泛型来约束表示条目类型的入参和视图渲染组件之间的类型关系。</strong></p>
<p data-nodeid="22254">下面看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="22255"><code data-language="typescript"><span class="hljs-keyword">export</span> <span class="hljs-keyword">interface</span> IUserItem {
  username: <span class="hljs-built_in">string</span>;
}
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">RenderUser</span>(<span class="hljs-params">props: IUserItem</span>): <span class="hljs-title">React</span>.<span class="hljs-title">ReactElement</span> </span>{
  <span class="hljs-keyword">return</span> &lt;&gt;{props.username}&lt;/&gt;
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">interface</span> ITodoItem {
  taskName: <span class="hljs-built_in">string</span>;
}
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">RenderTodo</span>(<span class="hljs-params">props: ITodoItem</span>): <span class="hljs-title">React</span>.<span class="hljs-title">ReactElement</span> </span>{
  <span class="hljs-keyword">return</span> &lt;&gt;{props.taskName}&lt;/&gt;
}
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">ListCp</span>&lt;<span class="hljs-title">Item</span> <span class="hljs-title">extends</span> </span>{}&gt;(props: { Cp: React.ComponentType&lt;Item&gt; }): React.ReactElement {
  <span class="hljs-keyword">return</span> &lt;&gt;&lt;/&gt;;
}
<span class="hljs-keyword">const</span> UserList = &lt;ListCp&lt;IUserItem&gt; Cp={RenderUser} /&gt;; <span class="hljs-comment">// ok</span>
<span class="hljs-keyword">const</span> TodoList = &lt;ListCp&lt;ITodoItem&gt; Cp={RenderTodo} /&gt;; <span class="hljs-comment">// ok</span>
<span class="hljs-keyword">const</span> UserListError = &lt;ListCp&lt;ITodoItem&gt; Cp={RenderUser} /&gt;; <span class="hljs-comment">// ts(2322)</span>
<span class="hljs-keyword">const</span> TodoListError = &lt;ListCp&lt;IUserItem&gt; Cp={RenderTodo} /&gt;; <span class="hljs-comment">// ts(2322)</span>
</code></pre>
<p data-nodeid="22256">在示例中的第 13 行，定义的泛型组件 ListCp 通过类型入参 Item 约束接收了 props  的 Cp 属性的具体类型。在第 16 行、第 17 行，因为类型入参 IUserItem、ITodoItem 和 Cp 属性 RenderUser、RenderTodo 类型一一对应，所以可以通过静态类型检测。但是，在第 18 行、第 19 行，因为对应关系不正确，所以提示了一个 ts(2322) 错误。</p>
<p data-nodeid="22257"><strong data-nodeid="22437">class 组件和函数组件类型组成的联合类型被称之为组件类型  React.ComponentType，组件类型一般用来定义高阶组件的属性</strong>，如下代码所示：</p>
<pre class="lang-typescript" data-nodeid="22258"><code data-language="typescript">React.ComponentType&lt;P&gt; = React.ComponentClass&lt;P&gt; | React.FunctionComponent&lt;P&gt;;
</code></pre>
<p data-nodeid="22259">最后介绍几个常用类型：</p>
<ul data-nodeid="22260">
<li data-nodeid="22261">
<p data-nodeid="22262"><strong data-nodeid="22443">元素类型 React.ElementType</strong>：指的是所有可以通过 JSX 语法创建元素的类型组合，包括html 原生标签（比如 div、a 等）和 React.ComponentType，元素类型可以接收一个表示 props 的类型入参；</p>
</li>
<li data-nodeid="22263">
<p data-nodeid="22264"><strong data-nodeid="22448">元素节点类型 React.ReactElement</strong>：指的是元素类型通过 JSX 语法创建的节点类型，它可以接收两个分别表示 props 和元素类型的类型入参；</p>
</li>
<li data-nodeid="22265">
<p data-nodeid="22266"><strong data-nodeid="22453">节点类型 React.ReactNode</strong>：指的是由 string、number、boolean、undefined、null、React.ReactElement 和元素类型是 React.ReactElement 的数组类型组成的联合类型，合法的 class 组件 render 方法返回值类型必须是 React.ReactNode；</p>
</li>
<li data-nodeid="22267">
<p data-nodeid="22268"><strong data-nodeid="22460">JSX 元素类型 JSX.Element</strong>：指的是元素类型通过 JSX 语法创建的节点类型，JSX.Element 等于 React.ReactElement&lt;any, any&gt;。</p>
</li>
</ul>
<p data-nodeid="22269">以上就是 React Component 相关的类型及简单的类型化。</p>
<p data-nodeid="22270">在实际业务中，因为组件接收的 props 数据可能来自路由、Redux，所以我们还需要对类型进行更明确的分解。</p>
<p data-nodeid="22271">下面我们看一个具体的示例：</p>
<pre class="lang-typescript" data-nodeid="22272"><code data-language="typescript"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>; 
<span class="hljs-keyword">import</span> { bindActionCreators, Dispatch } <span class="hljs-keyword">from</span> <span class="hljs-string">"redux"</span>;
<span class="hljs-keyword">import</span> { connect } <span class="hljs-keyword">from</span> <span class="hljs-string">"react-redux"</span>;
<span class="hljs-keyword">import</span> { RouteComponentProps } <span class="hljs-keyword">from</span> <span class="hljs-string">'react-router-dom'</span>;
<span class="hljs-comment">/** 路由 Props */</span>
<span class="hljs-keyword">type</span> RouteProps = RouteComponentProps&lt;{ routeId: <span class="hljs-built_in">string</span> }&gt;;
<span class="hljs-comment">/** Redux Store Props */</span>
<span class="hljs-keyword">type</span> StateProps = ReturnType&lt;<span class="hljs-keyword">typeof</span> mapStateToProps&gt;;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">mapStateToProps</span>(<span class="hljs-params">state: {}</span>) </span>{
  <span class="hljs-keyword">return</span> {
    reduxId: <span class="hljs-number">1</span>
  };
}
<span class="hljs-comment">/** Redux Actions Props */</span>
<span class="hljs-keyword">type</span> DispatchProps = ReturnType&lt;<span class="hljs-keyword">typeof</span> mapDispatchToProps&gt;;
<span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">mapDispatchToProps</span>(<span class="hljs-params">dispatch: Dispatch</span>) </span>{
  <span class="hljs-keyword">return</span> {
    actions: bindActionCreators({
      doSomething: <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> <span class="hljs-built_in">void</span> <span class="hljs-number">0</span>
    }, dispatch),
  };
}
<span class="hljs-comment">/** 组件属性 */</span>
<span class="hljs-keyword">interface</span> IOwnProps {
  ownId: <span class="hljs-built_in">number</span>;
}
<span class="hljs-comment">/** 最终 Props */</span>
<span class="hljs-keyword">type</span> CpProps = IOwnProps &amp; RouteProps &amp; StateProps &amp; DispatchProps;
<span class="hljs-keyword">const</span> OriginalCp = <span class="hljs-function">(<span class="hljs-params">props: CpProps</span>) =&gt;</span> {
  <span class="hljs-keyword">const</span> {
    match: { params: { routeId } }, <span class="hljs-comment">// 路由 Props</span>
    reduxId, <span class="hljs-comment">// Redux Props</span>
    ownId, <span class="hljs-comment">// 组件 Props</span>
    actions: {
      doSomething <span class="hljs-comment">// Action Props</span>
    },
  } = props;
  <span class="hljs-keyword">return</span> <span class="hljs-literal">null</span>;
};
<span class="hljs-keyword">const</span> ConnectedCp = connect&lt;StateProps, DispatchProps, IOwnProps&gt;(mapStateToProps, mapDispatchToProps)(OriginalCp <span class="hljs-keyword">as</span> React.ComponentType&lt;IOwnProps&gt;);
<span class="hljs-keyword">const</span> ConnectedCpJSX = &lt;ConnectedCp ownId={<span class="hljs-number">1</span>} /&gt;; <span class="hljs-comment">// ok</span>
</code></pre>
<p data-nodeid="22273">在第 7 行，我们定义了 RouteProps，描述的是从路由中获取的属性。在第 9 行获取了 mapStateToProps 函数返回值类型 StateProps，描述的是从 Redux Store 中获取的属性。</p>
<p data-nodeid="22274">在第 16 行，我们获取了 mapDispatchToProps 函数返回值类型 DispatchProps，描述的是 Redux Actions 属性。在第 25 行，我们定义的是组件自有的属性，所以最终组件 OriginalCp 的属性类型 CpProps 是 RouteProps、StateProps、DispatchProps 和 IOwnProps 四个类型的交叉类型。在第 31~38 行，我们解构了 props 中不同来源的属性、方法，并且可以通过静态类型检测。</p>
<p data-nodeid="22275"><strong data-nodeid="22471">这里插播一道思考题：以上示例会提示一个缺少 react-redux、react-router-dom 类型声明的错误，应该如何解决呢？</strong></p>
<blockquote data-nodeid="22276">
<p data-nodeid="22277">注意：在示例中的第 41 行，connect 之前，我们把组件 OriginalCp 断言为 React.ComponentType<iownprops> 类型，这样在第 42 行使用组件的时候，就只需要传入 IOwnProps 中定义的属性（因为 RouteProps、StateProps、DispatchProps 属性可以通过路由或者 connect 自动注入）。</iownprops></p>
</blockquote>
<p data-nodeid="22278">这里使用的类型断言是开发 HOC 高阶组件（上边示例中 connect(mapStateToProps, mapDispatchToProps) 返回的是一个高阶组件）的一个惯用技巧，一般我们可以通过划分 HOCProps、IOwnProps 或 Omit 来剔除高阶组件注入的属性，如下示例中的第 4 行、第 5 行。</p>
<pre class="lang-typescript" data-nodeid="22279"><code data-language="typescript"><span class="hljs-keyword">interface</span> IHOCProps { injectId: <span class="hljs-built_in">number</span>; }
<span class="hljs-keyword">interface</span> IOwnProps { ownId: <span class="hljs-built_in">number</span>; }
<span class="hljs-keyword">const</span> hoc = &lt;C <span class="hljs-keyword">extends</span> React.ComponentType&lt;<span class="hljs-built_in">any</span>&gt;&gt;<span class="hljs-function">(<span class="hljs-params">cp: C</span>) =&gt;</span> cp;
<span class="hljs-keyword">const</span> InjectedCp1 = hoc(OriginalCp <span class="hljs-keyword">as</span> React.ComponentType&lt;IOwnProps&gt;);
<span class="hljs-keyword">const</span> InjectedCp2 = hoc(OriginalCp <span class="hljs-keyword">as</span> React.ComponentType&lt;Omit&lt;IHOCProps &amp; IOwnProps, <span class="hljs-string">'injectId'</span>&gt;&gt;); 
</code></pre>
<p data-nodeid="22280">组件类型化还涉及 Hooks 等知识点，限于篇幅，本文就不继续展开了。</p>
<p data-nodeid="22281">接下来我们简单了解一下使用 Redux 进行状态管理技术方案的类型化，将其称之为 Redux 类型化。</p>
<h4 data-nodeid="22282">Redux 类型化</h4>
<p data-nodeid="22283">Redux 类型化涉及 state、action、reducer 三要素类型化，具体示例如下：</p>
<pre class="lang-typescript" data-nodeid="22284"><code data-language="typescript"><span class="hljs-comment">// src/redux/user.ts</span>
<span class="hljs-comment">// state</span>
<span class="hljs-keyword">interface</span> IUserInfoState {
  userid?: <span class="hljs-built_in">number</span>;
  username?: <span class="hljs-built_in">string</span>;
}
<span class="hljs-keyword">export</span> <span class="hljs-keyword">const</span> initialState: IUserInfoState = {};
<span class="hljs-comment">// action</span>
<span class="hljs-keyword">interface</span> LoginAction {
  <span class="hljs-keyword">type</span>: <span class="hljs-string">'userinfo/login'</span>;
  payload: Required&lt;IUserInfoState&gt;;
}
<span class="hljs-keyword">interface</span> LogoutAction {
  <span class="hljs-keyword">type</span>: <span class="hljs-string">'userinfo/logout'</span>;
}
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">doLogin</span>(<span class="hljs-params"></span>): <span class="hljs-title">LoginAction</span> </span>{
  <span class="hljs-keyword">return</span> {
    <span class="hljs-keyword">type</span>: <span class="hljs-string">'userinfo/login'</span>,
    payload: {
      userid: <span class="hljs-number">101</span>,
      username: <span class="hljs-string">'乾元亨利贞'</span>
    }
  };
}
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">doLogout</span>(<span class="hljs-params"></span>): <span class="hljs-title">LogoutAction</span> </span>{
  <span class="hljs-keyword">return</span> {
    <span class="hljs-keyword">type</span>: <span class="hljs-string">'userinfo/logout'</span>
  };
}
<span class="hljs-comment">// reducer</span>
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">applyUserInfo</span>(<span class="hljs-params">state = initialState, action: LoginAction | LogoutAction</span>): <span class="hljs-title">IUserInfoState</span> </span>{
  <span class="hljs-keyword">switch</span> (action.type) {
    <span class="hljs-keyword">case</span> <span class="hljs-string">'userinfo/login'</span>:
      <span class="hljs-keyword">return</span> {
        ...action.payload
      };
    <span class="hljs-keyword">case</span> <span class="hljs-string">'userinfo/logout'</span>:
      <span class="hljs-keyword">return</span> {};
  }
}
</code></pre>
<p data-nodeid="22285">在示例中的第 2~7 行，我们定义了 state 的详细类型，并在第 8~29 行分别定义了表示登入、登出的 action 类型和函数，还在第 30~40 行定义了处理前边定义的 action 的 reducer 函数。</p>
<p data-nodeid="22286">然后，我们就将类型化后的 state、action、reducer 合并到 redux store，再通过 react-redux 关联 React，这样组件在 connect 之后，就能和 Redux 交互了。</p>
<p data-nodeid="22287">不过，因为 state、action、reducer 分别类型化的形式写起来十分复杂，所以我们可以借助 typesafe-actions、redux-actions、rematch、dvajs、@ekit/model 等工具更清晰、高效地组织 Redux 代码。限于篇幅，这里就不做深入介绍了，你可以自行到<a href="https://www.npmjs.com/" data-nodeid="22491">https://www.npmjs.com/</a>上查看更多信息。</p>
<h4 data-nodeid="22288">单元测试</h4>
<p data-nodeid="22289">我们可以选择 Jest + Enzyme + jsdom + ReactTestUtils 作为 React + TypeScript 应用的单元测试技术方案，不过麻烦的地方在于需要手动配置 Jest、Enzyme。因此，我更推荐选择<a href="https://github.com/testing-library/react-testing-library" data-nodeid="22497">react-testing-library</a>这个方案，这也是 create-react-app 默认内置的单元测试方案。</p>
<p data-nodeid="22290">如下示例，我们为前边定义的 RenderUser 组件编写了单元测试。</p>
<pre class="lang-typescript" data-nodeid="22291"><code data-language="typescript"><span class="hljs-keyword">import</span> React <span class="hljs-keyword">from</span> <span class="hljs-string">'react'</span>;
<span class="hljs-keyword">import</span> { render, screen } <span class="hljs-keyword">from</span> <span class="hljs-string">'@testing-library/react'</span>;
<span class="hljs-keyword">import</span> { RenderUser } <span class="hljs-keyword">from</span> <span class="hljs-string">'./Cp'</span>;
test(<span class="hljs-string">'renders learn react link'</span>, <span class="hljs-function"><span class="hljs-params">()</span> =&gt;</span> {
  render(&lt;RenderUser username={<span class="hljs-string">'乾元亨利贞'</span>} /&gt;);
  <span class="hljs-keyword">const</span> linkElement = screen.getByText(<span class="hljs-regexp">/乾元亨利贞/i</span>);
  expect(linkElement).toBeInTheDocument();
});
</code></pre>
<blockquote data-nodeid="22292">
<p data-nodeid="22293">注意：以上介绍的单测执行环境是 Node.js，TypeScript 会被转译成 CommonJS 格式，而在浏览器端运行时，则会被转译成 ES 格式。因此，不同模块之间存在循环依赖时，转译后代码在浏览器端可以正确运行，而在 Node.js 端运行时可能会出现引入的其他模块成员未定义（undefined）的错误。</p>
</blockquote>
<h3 data-nodeid="22294">小结和预告</h3>
<p data-nodeid="22295">以上就是 TypeScript 和 Dom 原生操作及结合 React 框架在 Web 侧开发的实践建议，其核心在于类型化 Dom API 和 React 组件、Redux 和 Service。</p>
<p data-nodeid="22296">插播一道思考题：类型化 React 组件的要义是什么？欢迎你在留言区进行互动、交流。</p>
<p data-nodeid="22297">20 讲我们将学习如何从 JavaScript 迁移到 TypeScript，敬请期待。</p>
<p data-nodeid="22298">另外，如果你觉得本专栏有价值，欢迎分享给更多好友~</p>

---

### 精选评论


