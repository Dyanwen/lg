<p data-nodeid="68116">相信对有一定基础的前端开发工程师来说，路由并不陌生，它最初源于服务端，在服务端中路由描述的是 URL 与处理函数之间的映射关系。</p>



<p data-nodeid="67684">而在 Web 前端单页应用 SPA 中，路由描述的是 URL 与视图之间的映射关系，这种映射是单向的，即 URL 变化会引起视图的更新。</p>
<p data-nodeid="67685">相比于后端路由，前端路由的好处是无须刷新页面，减轻了服务器的压力，提升了用户体验。目前主流支持单页应用的前端框架，基本都有配套的或第三方的路由系统。相应的，Vue.js 也提供了官方前端路由实现 Vue Router，那么这节课我们就来学习它的实现原理。</p>
<blockquote data-nodeid="67686">
<p data-nodeid="67687">Vue.js 3.0 配套的 Vue Router 源码在<a href="https://github.com/vuejs/vue-router-next" data-nodeid="67761">这里</a>，建议你学习前先把源码 clone 下来。如果你还不会使用路由，建议你先看它的<a href="https://next.router.vuejs.org/" data-nodeid="67765">官网文档</a>，会使用后再来学习本节课。</p>
</blockquote>
<h3 data-nodeid="67688">路由的基本用法</h3>
<p data-nodeid="67689">我们先通过一个简单地示例来看路由的基本用法，希望你也可以使用 Vue cli 脚手架创建一个 Vue.js 3.0 的项目，并安装 4.x 版本的 Vue Router 把项目跑起来。</p>
<p data-nodeid="67690">注意，为了让 Vue.js 可以在线编译模板，你需要在根目录下配置 vue.config.js，并且设置 runtimeCompiler 为 true：</p>
<pre class="lang-java" data-nodeid="67691"><code data-language="java"><span class="hljs-keyword">module</span>.<span class="hljs-keyword">exports</span> = {
  runtimeCompiler: <span class="hljs-keyword">true</span>
}
</code></pre>
<p data-nodeid="67692">然后我们修改页面的 HTML 模板，加上如下代码：</p>
<pre class="lang-js" data-nodeid="68400"><code data-language="js">&lt;div id=<span class="hljs-string">"app"</span>&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Hello App!<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">router-link</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"/"</span>&gt;</span>Go to Home<span class="hljs-tag">&lt;/<span class="hljs-name">router-link</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">router-link</span> <span class="hljs-attr">to</span>=<span class="hljs-string">"/about"</span>&gt;</span>Go to About<span class="hljs-tag">&lt;/<span class="hljs-name">router-link</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">router-view</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">router-view</span>&gt;</span></span>
&lt;/div&gt;
</code></pre>
<p data-nodeid="68401">其中，RouterLink 和 RouterView 是 Vue Router 内置的组件。</p>
<p data-nodeid="68402">RouterLink 表示路由的导航组件，我们可以配置 to 属性来指定它跳转的链接，它最终会在页面上渲染生成 a 标签。</p>
<p data-nodeid="68403">RouterView 表示路由的视图组件，它会渲染路径对应的 Vue 组件，也支持嵌套。</p>
<p data-nodeid="68404">RouterLink 和 RouterView 的具体实现，我们会放到后面去分析。</p>
<p data-nodeid="68405">有了模板之后，我们接下来看如何初始化路由：</p>
<pre class="lang-java" data-nodeid="68406"><code data-language="java"><span class="hljs-keyword">import</span> { createApp } from <span class="hljs-string">'vue'</span>
<span class="hljs-keyword">import</span> { createRouter, createWebHashHistory } from <span class="hljs-string">'vue-router'</span>
<span class="hljs-comment">// 1. 定义路由组件</span>
<span class="hljs-keyword">const</span> Home = { template: <span class="hljs-string">'&lt;div&gt;Home&lt;/div&gt;'</span> }
<span class="hljs-keyword">const</span> About = { template: <span class="hljs-string">'&lt;div&gt;About&lt;/div&gt;'</span> }
<span class="hljs-comment">// 2. 定义路由配置，每个路径映射一个路由视图组件</span>
<span class="hljs-keyword">const</span> routes = [
  { path: <span class="hljs-string">'/'</span>, component: Home },
  { path: <span class="hljs-string">'/about'</span>, component: About },
]
<span class="hljs-comment">// 3. 创建路由实例，可以指定路由模式，传入路由配置对象</span>
<span class="hljs-keyword">const</span> router = createRouter({
  history: createWebHistory(),
  routes
})
<span class="hljs-comment">// 4. 创建 app 实例</span>
<span class="hljs-keyword">const</span> app = createApp({
})
<span class="hljs-comment">// 5. 在挂载页面 之前先安装路由</span>
app.use(router)
<span class="hljs-comment">// 6. 挂载页面</span>
app.mount(<span class="hljs-string">'#app'</span>)
</code></pre>
<p data-nodeid="68407">可以看到，路由的初始化过程很简单，首先需要定义一个路由配置，这个配置主要用于描述路径和组件的映射关系，即什么路径下 RouterView 应该渲染什么路由组件。</p>
<p data-nodeid="68408">接着创建路由对象实例，传入路由配置对象，并且也可以指定路由模式，Vue Router 目前支持三种模式，hash 模式，HTML5 模式和 memory 模式，我们常用的是前两种模式。</p>
<p data-nodeid="68409">最后在挂载页面前，我们需要安装路由，这样我们就可以在各个组件中访问路由对象以及使用路由的内置组件 RouterLink 和 RouterView 了。</p>
<p data-nodeid="68410">知道了 Vue Router 的基本用法后，接下来我们就可以探究它的实现原理了。由于 Vue Router 源码加起来有几千行，限于篇幅，我会把重点放在整体的实现流程上，不会讲实现的细节。</p>
<h3 data-nodeid="68411">路由的实现原理</h3>
<p data-nodeid="68412">我们先从用户使用的角度来分析，先从路由对象的创建过程开始。</p>
<h4 data-nodeid="68413">路由对象的创建</h4>
<p data-nodeid="68414">Vue Router 提供了一个 createRouter API，你可以通过它来创建一个路由对象，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="68415"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">createRouter</span><span class="hljs-params">(options)</span> </span>{
  <span class="hljs-comment">// 定义一些辅助方法和变量 </span>
  
  <span class="hljs-comment">// ...</span>
  
  <span class="hljs-comment">// 创建 router 对象</span>
  <span class="hljs-keyword">const</span> router = {
    <span class="hljs-comment">// 当前路径</span>
    currentRoute,
    addRoute,
    removeRoute,
    hasRoute,
    getRoutes,
    resolve,
    options,
    push,
    replace,
    go,
    back: () =&gt; go(-<span class="hljs-number">1</span>),
    forward: () =&gt; go(<span class="hljs-number">1</span>),
    beforeEach: beforeGuards.add,
    beforeResolve: beforeResolveGuards.add,
    afterEach: afterGuards.add,
    onError: errorHandlers.add,
    isReady,
    install(app) {
      <span class="hljs-comment">// 安装路由函数</span>
    }
  }
  <span class="hljs-keyword">return</span> router
}
</code></pre>
<p data-nodeid="68416">我们省略了大部分代码，只保留了路由对象相关的代码，可以看到路由对象 router 就是一个对象，它维护了当前路径 currentRoute，且拥有很多辅助方法。</p>
<p data-nodeid="68417">目前你只需要了解这么多，创建完路由对象后，我们现在来安装它。</p>
<h4 data-nodeid="68418">路由的安装</h4>
<p data-nodeid="68419">Vue Router 作为 Vue 的插件，当我们执行 app.use(router) 的时候，实际上就是在执行 router 的 install 方法来安装路由，并把 app 作为参数传入，来看它的定义：</p>
<pre class="lang-java" data-nodeid="68420"><code data-language="java"><span class="hljs-keyword">const</span> router = {
  install(app) {
    <span class="hljs-keyword">const</span> router = <span class="hljs-keyword">this</span>
    <span class="hljs-comment">// 注册路由组件</span>
    app.component(<span class="hljs-string">'RouterLink'</span>, RouterLink)
    app.component(<span class="hljs-string">'RouterView'</span>, RouterView)
    <span class="hljs-comment">// 全局配置定义 $router 和 $route</span>
    app.config.globalProperties.$router = router
    Object.defineProperty(app.config.globalProperties, <span class="hljs-string">'$route'</span>, {
      get: () =&gt; unref(currentRoute),
    })
    <span class="hljs-comment">// 在浏览器端初始化导航</span>
    <span class="hljs-keyword">if</span> (isBrowser &amp;&amp;
      !started &amp;&amp;
      currentRoute.value === START_LOCATION_NORMALIZED) {
      <span class="hljs-comment">// see above</span>
      started = <span class="hljs-function"><span class="hljs-keyword">true</span>
      <span class="hljs-title">push</span><span class="hljs-params">(routerHistory.location)</span>.<span class="hljs-title">catch</span><span class="hljs-params">(err =&gt; {
        warn(<span class="hljs-string">'Unexpected error when starting the router:'</span>, err)</span>
      })
    }
    <span class="hljs-comment">// 路径变成响应式</span>
    <span class="hljs-keyword">const</span> reactiveRoute </span>= {}
    <span class="hljs-keyword">for</span> (let key in START_LOCATION_NORMALIZED) {
      reactiveRoute[key] = computed(() =&gt; currentRoute.value[key])
    }
    <span class="hljs-comment">// 全局注入 router 和 reactiveRoute</span>
    app.provide(routerKey, router)
    app.provide(routeLocationKey, reactive(reactiveRoute))
    let unmountApp = app.unmount
    installedApps.add(app)
    <span class="hljs-comment">// 应用卸载的时候，需要做一些路由清理工作</span>
    app.unmount = function () {
      installedApps.delete(app)
      <span class="hljs-keyword">if</span> (installedApps.size &lt; <span class="hljs-number">1</span>) {
        removeHistoryListener()
        currentRoute.value = START_LOCATION_NORMALIZED
        started = <span class="hljs-keyword">false</span>
        ready = <span class="hljs-keyword">false</span>
      }
      unmountApp.call(<span class="hljs-keyword">this</span>, arguments)
    }
  }
}
</code></pre>
<p data-nodeid="68421">路由的安装的过程我们需要记住以下两件事情。</p>
<ol data-nodeid="68422">
<li data-nodeid="68423">
<p data-nodeid="68424">全局注册 RouterView 和 RouterLink 组件——这是你安装了路由后，可以在任何组件中去使用这俩个组件的原因，如果你使用 RouterView 或者 RouterLink 的时候收到提示不能解析  router-link 和 router-view，这说明你压根就没有安装路由。</p>
</li>
<li data-nodeid="68425">
<p data-nodeid="68426">通过 provide 方式全局注入 router 对象和 reactiveRoute 对象，其中 router 表示用户通过 createRouter 创建的路由对象，我们可以通过它去动态操作路由，reactiveRoute 表示响应式的路径对象，它维护着路径的相关信息。</p>
</li>
</ol>
<p data-nodeid="68427">那么至此我们就已经了解了路由对象的创建，以及路由的安装，但是前端路由的实现，还需要解决几个核心问题：路径是如何管理的，路径和路由组件的渲染是如何映射的。</p>
<p data-nodeid="68428">那么接下来，我们就来更细节地来看，依次来解决这两个问题。</p>
<h4 data-nodeid="68429">路径的管理</h4>
<p data-nodeid="68430">路由的基础结构就是一个路径对应一种视图，当我们切换路径的时候对应的视图也会切换，因此一个很重要的方面就是对路径的管理。</p>
<p data-nodeid="68431">首先，我们需要维护当前的路径 currentRoute，可以给它一个初始值 START_LOCATION_NORMALIZED，如下：</p>
<pre class="lang-java" data-nodeid="68432"><code data-language="java"><span class="hljs-keyword">const</span> START_LOCATION_NORMALIZED = {
  path: <span class="hljs-string">'/'</span>,
  name: undefined,
  params: {},
  query: {},
  hash: <span class="hljs-string">''</span>,
  fullPath: <span class="hljs-string">'/'</span>,
  matched: [],
  meta: {},
  redirectedFrom: undefined
}
</code></pre>
<p data-nodeid="68433">可以看到，路径对象包含了非常丰富的路径信息，具体含义我就不在这多说了，你可以参考<a href="https://next.router.vuejs.org/api/#meta-3" data-nodeid="68492">官方文档</a>。</p>
<p data-nodeid="68434">路由想要发生变化，就是通过改变路径完成的，路由对象提供了很多改变路径的方法，比如 router.push、router.replace，它们的底层最终都是通过 pushWithRedirect 完成路径的切换，我们来看一下它的实现：</p>
<pre class="lang-java" data-nodeid="69778"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">pushWithRedirect</span><span class="hljs-params">(to, redirectedFrom)</span> </span>{
  <span class="hljs-keyword">const</span> targetLocation = (pendingLocation = resolve(to))
  <span class="hljs-keyword">const</span> from = currentRoute.value
  <span class="hljs-keyword">const</span> data = to.state
  <span class="hljs-keyword">const</span> force = to.force
  <span class="hljs-keyword">const</span> replace = to.replace === <span class="hljs-keyword">true</span>
  <span class="hljs-keyword">const</span> toLocation = targetLocation
  toLocation.redirectedFrom = <span class="hljs-function">redirectedFrom
  let failure
  <span class="hljs-title">if</span> <span class="hljs-params">(!force &amp;&amp; isSameRouteLocation(stringifyQuery$<span class="hljs-number">1</span>, from, targetLocation)</span>) </span>{
    failure = createRouterError(<span class="hljs-number">16</span> <span class="hljs-comment">/* NAVIGATION_DUPLICATED */</span>, { to: toLocation, from })
    handleScroll(from, from, <span class="hljs-keyword">true</span>, <span class="hljs-keyword">false</span>)
  }
  <span class="hljs-keyword">return</span> (failure ? Promise.resolve(failure) : navigate(toLocation, from))
    .<span class="hljs-keyword">catch</span>((error) =&gt; {
      <span class="hljs-keyword">if</span> (isNavigationFailure(error, <span class="hljs-number">4</span> <span class="hljs-comment">/* NAVIGATION_ABORTED */</span> |
        <span class="hljs-number">8</span> <span class="hljs-comment">/* NAVIGATION_CANCELLED */</span> |
        <span class="hljs-number">2</span> <span class="hljs-comment">/* NAVIGATION_GUARD_REDIRECT */</span>)) {
        <span class="hljs-keyword">return</span> error
      }
      <span class="hljs-keyword">return</span> triggerError(error)
    })
    .then((failure) =&gt; {
      <span class="hljs-keyword">if</span> (failure) {
        <span class="hljs-comment">// 处理错误</span>
      }
      <span class="hljs-keyword">else</span> {
        failure = finalizeNavigation(toLocation, from, <span class="hljs-keyword">true</span>, replace, data)
      }
      triggerAfterEach(toLocation, from, failure)
      <span class="hljs-keyword">return</span> failure
    })
}
</code></pre>
<p data-nodeid="69779">我省略了一部分代码的实现，这里主要来看 pushWithRedirect 的核心思路，首先参数 to 可能有多种情况，可以是一个表示路径的字符串，也可以是一个路径对象，所以要先经过一层 resolve 返回一个新的路径对象，它比前面提到的路径对象多了一个 matched 属性，它的作用我们后续会介绍。</p>
<p data-nodeid="69780">得到新的目标路径后，接下来执行 navigate 方法，它实际上是执行路由切换过程中的一系列导航守卫函数，我们后续会介绍。navigate 成功后，会执行 finalizeNavigation 完成导航，在这里完成真正的路径切换，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="69781"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">finalizeNavigation</span><span class="hljs-params">(toLocation, from, isPush, replace, data)</span> </span>{
  <span class="hljs-keyword">const</span> error = checkCanceledNavigation(toLocation, from)
  <span class="hljs-keyword">if</span> (error)
    <span class="hljs-keyword">return</span> error
  <span class="hljs-keyword">const</span> isFirstNavigation = from === START_LOCATION_NORMALIZED
  <span class="hljs-keyword">const</span> state = !isBrowser ? {} : history.<span class="hljs-function">state
  <span class="hljs-title">if</span> <span class="hljs-params">(isPush)</span> </span>{
    <span class="hljs-keyword">if</span> (replace || isFirstNavigation)
      routerHistory.replace(toLocation.fullPath, assign({
        scroll: isFirstNavigation &amp;&amp; state &amp;&amp; state.scroll,
      }, data))
    <span class="hljs-keyword">else</span>
      routerHistory.push(toLocation.fullPath, data)
  }
  currentRoute.value = <span class="hljs-function">toLocation
  <span class="hljs-title">handleScroll</span><span class="hljs-params">(toLocation, from, isPush, isFirstNavigation)</span>
  <span class="hljs-title">markAsReady</span><span class="hljs-params">()</span>
}
</span></code></pre>
<p data-nodeid="70105">这里的 finalizeNavigation 函数，我们重点关注两个逻辑，一个是更新当前的路径 currentRoute 的值，一个是执行 routerHistory.push 或者是 routerHistory.replace 方法更新浏览器的 URL 的记录。</p>
<p data-nodeid="70106">每当我们切换路由的时候，会发现浏览器的 URL 发生了变化，但是页面却没有刷新，它是怎么做的呢？</p>

<p data-nodeid="69783">在我们创建 router 对象的时候，会创建一个 history 对象，前面提到 Vue Router 支持三种模式，这里我们重点分析 HTML5 的 history 的模式：</p>
<pre class="lang-java" data-nodeid="69784"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">createWebHistory</span><span class="hljs-params">(base)</span> </span>{
  base = normalizeBase(base)
  <span class="hljs-keyword">const</span> historyNavigation = useHistoryStateNavigation(base)
  <span class="hljs-keyword">const</span> historyListeners = useHistoryListeners(base, historyNavigation.state, historyNavigation.location, historyNavigation.replace)
  <span class="hljs-function">function <span class="hljs-title">go</span><span class="hljs-params">(delta, triggerListeners = <span class="hljs-keyword">true</span>)</span> </span>{
    <span class="hljs-keyword">if</span> (!triggerListeners)
      historyListeners.pauseListeners()
    history.go(delta)
  }
  <span class="hljs-keyword">const</span> routerHistory = assign({
    <span class="hljs-comment">// it's overridden right after</span>
    location: <span class="hljs-string">''</span>,
    base,
    go,
    createHref: createHref.bind(<span class="hljs-keyword">null</span>, base),
  }, historyNavigation, historyListeners)
  Object.defineProperty(routerHistory, <span class="hljs-string">'location'</span>, {
    get: () =&gt; historyNavigation.location.value,
  })
  Object.defineProperty(routerHistory, <span class="hljs-string">'state'</span>, {
    get: () =&gt; historyNavigation.state.value,
  })
  <span class="hljs-keyword">return</span> routerHistory
}
</code></pre>
<p data-nodeid="69785">对于 routerHistory 对象而言，它有两个重要的作用，一个是路径的切换，一个是监听路径的变化。</p>
<p data-nodeid="69786">其中，路径切换主要通过 historyNavigation 来完成的，它是 useHistoryStateNavigation 函数的返回值，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="69787"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">useHistoryStateNavigation</span><span class="hljs-params">(base)</span> </span>{
  <span class="hljs-keyword">const</span> { history, location } = window
  let currentLocation = {
    value: createCurrentLocation(base, location),
  }
  let historyState = { value: history.state }
  <span class="hljs-keyword">if</span> (!historyState.value) {
    changeLocation(currentLocation.value, {
      back: <span class="hljs-keyword">null</span>,
      current: currentLocation.value,
      forward: <span class="hljs-keyword">null</span>,
      position: history.length - <span class="hljs-number">1</span>,
      replaced: <span class="hljs-keyword">true</span>,
      scroll: <span class="hljs-keyword">null</span>,
    }, <span class="hljs-keyword">true</span>)
  }
  <span class="hljs-function">function <span class="hljs-title">changeLocation</span><span class="hljs-params">(to, state, replace)</span> </span>{
    <span class="hljs-keyword">const</span> url = createBaseLocation() +
      <span class="hljs-comment">// preserve any existing query when base has a hash</span>
      (base.indexOf(<span class="hljs-string">'#'</span>) &gt; -<span class="hljs-number">1</span> &amp;&amp; location.search
        ? location.pathname + location.search + <span class="hljs-string">'#'</span>
        : base) +
      to
    <span class="hljs-keyword">try</span> {
      history[replace ? <span class="hljs-string">'replaceState'</span> : <span class="hljs-string">'pushState'</span>](state, <span class="hljs-string">''</span>, url)
      historyState.value = state
    }
    <span class="hljs-keyword">catch</span> (err) {
      warn(<span class="hljs-string">'Error with push/replace State'</span>, err)
      location[replace ? <span class="hljs-string">'replace'</span> : <span class="hljs-string">'assign'</span>](url)
    }
  }
  <span class="hljs-function">function <span class="hljs-title">replace</span><span class="hljs-params">(to, data)</span> </span>{
    <span class="hljs-keyword">const</span> state = assign({}, history.state, buildState(historyState.value.back,
      <span class="hljs-comment">// keep back and forward entries but override current position</span>
      to, historyState.value.forward, <span class="hljs-keyword">true</span>), data, { position: historyState.value.position })
    changeLocation(to, state, <span class="hljs-keyword">true</span>)
    currentLocation.value = to
  }
  <span class="hljs-function">function <span class="hljs-title">push</span><span class="hljs-params">(to, data)</span> </span>{
    <span class="hljs-keyword">const</span> currentState = assign({},
      historyState.value, history.state, {
        forward: to,
        scroll: computeScrollPosition(),
      })
    <span class="hljs-keyword">if</span> ( !history.state) {
      warn(`history.state seems to have been manually replaced without preserving the necessary values. Make sure to preserve existing history state <span class="hljs-keyword">if</span> you are manually calling history.replaceState:\n\n` +
        `history.replaceState(history.state, <span class="hljs-string">''</span>, url)\n\n` +
        `You can find more information at https:<span class="hljs-comment">//next.router.vuejs.org/guide/migration/#usage-of-history-state.`)</span>
    }
    changeLocation(currentState.current, currentState, <span class="hljs-keyword">true</span>)
    <span class="hljs-keyword">const</span> state = assign({}, buildState(currentLocation.value, to, <span class="hljs-keyword">null</span>), { position: currentState.position + <span class="hljs-number">1</span> }, data)
    changeLocation(to, state, <span class="hljs-keyword">false</span>)
    currentLocation.value = to
  }
  <span class="hljs-keyword">return</span> {
    location: currentLocation,
    state: historyState,
    push,
    replace
  }
}
</code></pre>
<p data-nodeid="69788">该函数返回的 push 和 replace 函数，会添加给 routerHistory 对象上，因此当我们调用 routerHistory.push 或者是 routerHistory.replace 方法的时候实际上就是在执行这两个函数。</p>
<p data-nodeid="69789">push 和 replace 方法内部都是执行了 changeLocation 方法，该函数内部执行了浏览器底层的 history.pushState 或者 history.replaceState 方法，会向当前浏览器会话的历史堆栈中添加一个状态，这样就在不刷新页面的情况下修改了页面的 URL。</p>
<p data-nodeid="69790">我们使用这种方法修改了路径，这个时候假设我们点击浏览器的回退按钮回到上一个 URL，这需要恢复到上一个路径以及更新路由视图，因此我们还需要监听这种 history 变化的行为，做一些相应的处理。</p>
<p data-nodeid="69791">History 变化的监听主要是通过 historyListeners 来完成的，它是 useHistoryListeners 函数的返回值，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="69792"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">useHistoryListeners</span><span class="hljs-params">(base, historyState, currentLocation, replace)</span> </span>{
  let listeners = []
  let teardowns = []
  let pauseState = <span class="hljs-keyword">null</span>
  <span class="hljs-keyword">const</span> popStateHandler = ({ state, }) =&gt; {
    <span class="hljs-keyword">const</span> to = createCurrentLocation(base, location)
    <span class="hljs-keyword">const</span> from = currentLocation.value
    <span class="hljs-keyword">const</span> fromState = historyState.value
    let delta = <span class="hljs-number">0</span>
    <span class="hljs-keyword">if</span> (state) {
      currentLocation.value = to
      historyState.value = <span class="hljs-function">state
      <span class="hljs-title">if</span> <span class="hljs-params">(pauseState &amp;&amp; pauseState === from)</span> </span>{
        pauseState = <span class="hljs-keyword">null</span>
        <span class="hljs-keyword">return</span>
      }
      delta = fromState ? state.position - fromState.position : <span class="hljs-number">0</span>
    }
    <span class="hljs-keyword">else</span> {
      replace(to)
    }
    listeners.forEach(listener =&gt; {
      listener(currentLocation.value, from, {
        delta,
        type: NavigationType.pop,
        direction: delta
          ? delta &gt; <span class="hljs-number">0</span>
            ? NavigationDirection.forward
            : NavigationDirection.back
          : NavigationDirection.unknown,
      })
    })
  }
  <span class="hljs-function">function <span class="hljs-title">pauseListeners</span><span class="hljs-params">()</span> </span>{
    pauseState = currentLocation.value
  }
  <span class="hljs-function">function <span class="hljs-title">listen</span><span class="hljs-params">(callback)</span> </span>{
    listeners.push(callback)
    <span class="hljs-keyword">const</span> teardown = () =&gt; {
      <span class="hljs-keyword">const</span> index = listeners.indexOf(callback)
      <span class="hljs-keyword">if</span> (index &gt; -<span class="hljs-number">1</span>)
        listeners.splice(index, <span class="hljs-number">1</span>)
    }
    teardowns.push(teardown)
    <span class="hljs-keyword">return</span> teardown
  }
  <span class="hljs-function">function <span class="hljs-title">beforeUnloadListener</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">const</span> { history } = <span class="hljs-function">window
    <span class="hljs-title">if</span> <span class="hljs-params">(!history.state)</span>
      return
    history.<span class="hljs-title">replaceState</span><span class="hljs-params">(assign({}, history.state, { scroll: computeScrollPosition()</span> }), '')
  }
  function <span class="hljs-title">destroy</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> teardown of teardowns)
      teardown()
    teardowns = []
    window.removeEventListener(<span class="hljs-string">'popstate'</span>, popStateHandler)
    window.removeEventListener(<span class="hljs-string">'beforeunload'</span>, beforeUnloadListener)
  }
  window.addEventListener(<span class="hljs-string">'popstate'</span>, popStateHandler)
  window.addEventListener(<span class="hljs-string">'beforeunload'</span>, beforeUnloadListener)
  <span class="hljs-keyword">return</span> {
    pauseListeners,
    listen,
    destroy
  }
}
</code></pre>
<p data-nodeid="69793">该函数返回了 listen 方法，允许你添加一些侦听器，侦听 hstory 的变化，同时这个方法也被挂载到了 routerHistory 对象上，这样外部就可以访问到了。</p>
<p data-nodeid="69794">该函数内部还监听了浏览器底层 Window 的 popstate 事件，当我们点击浏览器的回退按钮或者是执行了 history.back 方法的时候，会触发事件的回调函数 popStateHandler，进而遍历侦听器 listeners，执行每一个侦听器函数。</p>
<p data-nodeid="69795">那么，Vue Router 是如何添加这些侦听器的呢？原来在安装路由的时候，会执行一次初始化导航，执行了 push 方法进而执行了 finalizeNavigation 方法。</p>
<p data-nodeid="69796">在 finalizeNavigation 的最后，会执行 markAsReady 方法，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="69797"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">markAsReady</span><span class="hljs-params">(err)</span> </span>{
  <span class="hljs-keyword">if</span> (ready)
    <span class="hljs-keyword">return</span>
  ready = <span class="hljs-function"><span class="hljs-keyword">true</span>
  <span class="hljs-title">setupListeners</span><span class="hljs-params">()</span>
  readyHandlers
    .<span class="hljs-title">list</span><span class="hljs-params">()</span>
    .<span class="hljs-title">forEach</span><span class="hljs-params">(([resolve, reject])</span> </span>=&gt; (err ? reject(err) : resolve()))
  readyHandlers.reset()
}
</code></pre>
<p data-nodeid="69798">markAsReady 内部会执行 setupListeners 函数初始化侦听器，且保证只初始化一次。我们再接着来看 setupListeners 的实现：</p>
<pre class="lang-java" data-nodeid="69799"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">setupListeners</span><span class="hljs-params">()</span> </span>{
  removeHistoryListener = routerHistory.listen((to, _from, info) =&gt; {
    <span class="hljs-keyword">const</span> toLocation = resolve(to)
    pendingLocation = toLocation
    <span class="hljs-keyword">const</span> from = currentRoute.<span class="hljs-function">value
    <span class="hljs-title">if</span> <span class="hljs-params">(isBrowser)</span> </span>{
      saveScrollPosition(getScrollKey(from.fullPath, info.delta), computeScrollPosition())
    }
    navigate(toLocation, from)
      .<span class="hljs-keyword">catch</span>((error) =&gt; {
        <span class="hljs-keyword">if</span> (isNavigationFailure(error, <span class="hljs-number">4</span> <span class="hljs-comment">/* NAVIGATION_ABORTED */</span> | <span class="hljs-number">8</span> <span class="hljs-comment">/* NAVIGATION_CANCELLED */</span>)) {
          <span class="hljs-keyword">return</span> error
        }
        <span class="hljs-keyword">if</span> (isNavigationFailure(error, <span class="hljs-number">2</span> <span class="hljs-comment">/* NAVIGATION_GUARD_REDIRECT */</span>)) {
          <span class="hljs-keyword">if</span> (info.delta)
            routerHistory.go(-info.delta, <span class="hljs-keyword">false</span>)
          pushWithRedirect(error.to, toLocation
          ).<span class="hljs-keyword">catch</span>(noop)
          <span class="hljs-comment">// avoid the then branch</span>
          <span class="hljs-keyword">return</span> Promise.reject()
        }
        <span class="hljs-keyword">if</span> (info.delta)
          routerHistory.go(-info.delta, <span class="hljs-keyword">false</span>)
        <span class="hljs-keyword">return</span> triggerError(error)
      })
      .then((failure) =&gt; {
        failure =
          failure ||
          finalizeNavigation(
            toLocation, from, <span class="hljs-keyword">false</span>)
        <span class="hljs-keyword">if</span> (failure &amp;&amp; info.delta)
          routerHistory.go(-info.delta, <span class="hljs-keyword">false</span>)
        triggerAfterEach(toLocation, from, failure)
      })
      .<span class="hljs-keyword">catch</span>(noop)
  })
}
</code></pre>
<p data-nodeid="69800">侦听器函数也是执行 navigate 方法，执行路由切换过程中的一系列导航守卫函数，在 navigate 成功后执行 finalizeNavigation 完成导航，完成真正的路径切换。这样就保证了在用户点击浏览器回退按钮后，可以恢复到上一个路径以及更新路由视图。</p>
<p data-nodeid="70391">至此，我们就完成了路径管理，在内存中通过 currentRoute 维护记录当前的路径，通过浏览器底层 API 实现了路径的切换和 history 变化的监听。</p>

---

### 精选评论

##### **章：
> 组件b被别的组件导入后，想在新组件中帮b导入A现在找到的方案就这个最合适

##### **章：
> 我想把UI组件json化，封装过程中发现有些业务要灵活必须允许外面传递组件，允许自定义组件导入并渲染，h函数那个写法又看不下去所以用这个

##### **章：
> 黄老，刚才提了个问不晓得哪里去了，所以重复提一个。我把一个组件componentA，当成参数（如：attr:Object.freeze({tag:componentA})）传递给组件componentB，参数componentB用动态组件component渲染 is直接指向attr.tag,这样有什么弊端不？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 可以传，不过为何要用参数传递呢，直接 import 这个 ComponentA 不可以吗

