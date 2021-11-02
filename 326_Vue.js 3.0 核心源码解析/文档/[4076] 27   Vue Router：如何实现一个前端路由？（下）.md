<p data-nodeid="391">上节课我们学习了 Vue Router 的基本用法，并且开始探究它的实现原理，今天我们继续未完的原理，一起来看路径是如何和路由组件映射的。</p>



<h3 data-nodeid="4">路径和路由组件的渲染的映射</h3>
<p data-nodeid="5">通过前面的示例我们了解到，路由组件就是通过 RouterView 组件渲染的，那么 RouterView 是怎么渲染的呢，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="6"><code data-language="java"><span class="hljs-keyword">const</span> RouterView = defineComponent({
  name: <span class="hljs-string">'RouterView'</span>,
  props: {
    name: {
      type: String,
      <span class="hljs-keyword">default</span>: <span class="hljs-string">'default'</span>,
    },
    route: Object,
  },
  setup(props, { attrs, slots }) {
    warnDeprecatedUsage()
    <span class="hljs-keyword">const</span> injectedRoute = inject(routeLocationKey)
    <span class="hljs-keyword">const</span> depth = inject(viewDepthKey, <span class="hljs-number">0</span>)
    <span class="hljs-keyword">const</span> matchedRouteRef = computed(() =&gt; (props.route || injectedRoute).matched[depth])
    provide(viewDepthKey, depth + <span class="hljs-number">1</span>)
    provide(matchedRouteKey, matchedRouteRef)
    <span class="hljs-keyword">const</span> viewRef = ref()
    watch(() =&gt; [viewRef.value, matchedRouteRef.value, props.name], ([instance, to, name], [oldInstance, from, oldName]) =&gt; {
      <span class="hljs-keyword">if</span> (to) {
        to.instances[name] = <span class="hljs-function">instance
        <span class="hljs-title">if</span> <span class="hljs-params">(from &amp;&amp; instance === oldInstance)</span> </span>{
          to.leaveGuards = from.leaveGuards
          to.updateGuards = from.updateGuards
        }
      }
      <span class="hljs-keyword">if</span> (instance &amp;&amp;
        to &amp;&amp;
        (!from || !isSameRouteRecord(to, from) || !oldInstance)) {
        (to.enterCallbacks[name] || []).forEach(callback =&gt; callback(instance))
      }
    })
    <span class="hljs-keyword">return</span> () =&gt; {
      <span class="hljs-keyword">const</span> route = props.route || injectedRoute
      <span class="hljs-keyword">const</span> matchedRoute = matchedRouteRef.value
      <span class="hljs-keyword">const</span> ViewComponent = matchedRoute &amp;&amp; matchedRoute.components[props.name]
      <span class="hljs-keyword">const</span> currentName = props.<span class="hljs-function">name
      <span class="hljs-title">if</span> <span class="hljs-params">(!ViewComponent)</span> </span>{
        <span class="hljs-keyword">return</span> slots.<span class="hljs-keyword">default</span>
          ? slots.<span class="hljs-keyword">default</span>({ Component: ViewComponent, route })
          : <span class="hljs-keyword">null</span>
      }
      <span class="hljs-keyword">const</span> routePropsOption = matchedRoute.props[props.name]
      <span class="hljs-keyword">const</span> routeProps = routePropsOption
        ? routePropsOption === <span class="hljs-keyword">true</span>
          ? route.params
          : typeof routePropsOption === <span class="hljs-string">'function'</span>
            ? routePropsOption(route)
            : routePropsOption
        : <span class="hljs-keyword">null</span>
      <span class="hljs-keyword">const</span> onVnodeUnmounted = vnode =&gt; {
        <span class="hljs-keyword">if</span> (vnode.component.isUnmounted) {
          matchedRoute.instances[currentName] = <span class="hljs-keyword">null</span>
        }
      }
      <span class="hljs-keyword">const</span> component = h(ViewComponent, assign({}, routeProps, attrs, {
        onVnodeUnmounted,
        ref: viewRef,
      }))
      <span class="hljs-keyword">return</span> (
        slots.<span class="hljs-keyword">default</span>
          ? slots.<span class="hljs-keyword">default</span>({ Component: component, route })
          : component)
    }
  },
})
</code></pre>
<p data-nodeid="7">RouterView 组件也是基于 composition API 实现的，我们重点看它的渲染部分，由于 setup 函数的返回值是一个函数，那这个函数就是它的渲染函数。</p>
<p data-nodeid="8">我们从后往前看，通常不带插槽的情况下，会返回 component 变量，它是根据 ViewComponent 渲染出来的，而ViewComponent 是根据matchedRoute.components[props.name] 求得的，而matchedRoute 是 matchedRouteRef对应的 value。</p>
<p data-nodeid="9">matchedRouteRef 一个计算属性，在不考虑 prop 传入 route 的情况下，它的 getter 是由 injectedRoute.matched[depth] 求得的，而 injectedRoute，就是我们在前面在安装路由时候，注入的响应式 currentRoute 对象，而 depth 就是表示这个 RouterView 的嵌套层级。</p>
<p data-nodeid="10">所以我们可以看到，RouterView 的渲染的路由组件和当前路径 currentRoute 的 matched 对象相关，也和 RouterView 自身的嵌套层级相关。</p>
<p data-nodeid="11">那么接下来，我们就来看路径对象中的 matched 的值是怎么在路径切换的情况下更新的。</p>
<p data-nodeid="12">我们还是通过示例的方式来说明，我们对前面的示例稍做修改，加上嵌套路由的场景：</p>
<pre class="lang-js" data-nodeid="645"><code data-language="js"><span class="hljs-keyword">import</span> { createApp } <span class="hljs-keyword">from</span> <span class="hljs-string">'vue'</span>
<span class="hljs-keyword">import</span> { createRouter, createWebHashHistory } <span class="hljs-keyword">from</span> <span class="hljs-string">'vue-router'</span>
<span class="hljs-keyword">const</span> Home = { <span class="hljs-attr">template</span>: <span class="hljs-string">'&lt;div&gt;Home&lt;/div&gt;'</span> }
<span class="hljs-keyword">const</span> About = {
  <span class="hljs-attr">template</span>: <span class="hljs-string">`&lt;div&gt;About
  &lt;router-link to="/about/user"&gt;Go User&lt;/router-link&gt;
  &lt;router-view&gt;&lt;/router-view&gt;
  &lt;/div&gt;`</span>
}
<span class="hljs-keyword">const</span> User = {
  <span class="hljs-attr">template</span>: <span class="hljs-string">'&lt;div&gt;User&lt;/div&gt;,'</span>
}
<span class="hljs-keyword">const</span> routes = [
  { <span class="hljs-attr">path</span>: <span class="hljs-string">'/'</span>, <span class="hljs-attr">component</span>: Home },
  {
    <span class="hljs-attr">path</span>: <span class="hljs-string">'/about'</span>,
    <span class="hljs-attr">component</span>: About,
    <span class="hljs-attr">children</span>: [
      {
        <span class="hljs-attr">path</span>: <span class="hljs-string">'user'</span>,
        <span class="hljs-attr">component</span>: User
      }
    ]
  }
]
<span class="hljs-keyword">const</span> router = createRouter({
  <span class="hljs-attr">history</span>: createWebHashHistory(),
  routes
})
<span class="hljs-keyword">const</span> app = createApp({})
app.use(router)
app.mount(<span class="hljs-string">'#app'</span>)
</code></pre>
<p data-nodeid="646">它和前面示例的区别在于，我们在 About 路由组件中又嵌套了一个 RouterView 组件，然后对 routes 数组中 path 为 /about 的路径配置扩展了 children 属性，对应的就是 About 组件嵌套路由的配置。</p>
<p data-nodeid="647">当我们执行 createRouter 函数创建路由的时候，内部会执行如下代码来创建一个 matcher 对象：</p>
<pre class="lang-java" data-nodeid="648"><code data-language="java"><span class="hljs-keyword">const</span> matcher = createRouterMatcher(options.routes, options)
</code></pre>
<p data-nodeid="649">执行了createRouterMatcher 函数，并传入 routes 路径配置数组，它的目的就是根据路径配置对象创建一个路由的匹配对象，再来看它的实现：</p>
<pre class="lang-java" data-nodeid="650"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">createRouterMatcher</span><span class="hljs-params">(routes, globalOptions)</span> </span>{
  <span class="hljs-keyword">const</span> matchers = []
  <span class="hljs-keyword">const</span> matcherMap = <span class="hljs-keyword">new</span> Map()
  globalOptions = mergeOptions({ strict: <span class="hljs-keyword">false</span>, end: <span class="hljs-keyword">true</span>, sensitive: <span class="hljs-keyword">false</span> }, globalOptions)
  
  <span class="hljs-function">function <span class="hljs-title">addRoute</span><span class="hljs-params">(record, parent, originalRecord)</span> </span>{
    let isRootAdd = !originalRecord
    let mainNormalizedRecord = normalizeRouteRecord(record)
    mainNormalizedRecord.aliasOf = originalRecord &amp;&amp; originalRecord.record
    <span class="hljs-keyword">const</span> options = mergeOptions(globalOptions, record)
    <span class="hljs-keyword">const</span> normalizedRecords = [
      mainNormalizedRecord,
    ]
    <span class="hljs-function">let matcher
    let originalMatcher
    <span class="hljs-title">for</span> <span class="hljs-params">(<span class="hljs-keyword">const</span> normalizedRecord of normalizedRecords)</span> </span>{
      let { path } = <span class="hljs-function">normalizedRecord
      <span class="hljs-title">if</span> <span class="hljs-params">(parent &amp;&amp; path[<span class="hljs-number">0</span>] !== <span class="hljs-string">'/'</span>)</span> </span>{
        let parentPath = parent.record.path
        let connectingSlash = parentPath[parentPath.length - <span class="hljs-number">1</span>] === <span class="hljs-string">'/'</span> ? <span class="hljs-string">''</span> : <span class="hljs-string">'/'</span>
        normalizedRecord.path =
          parent.record.path + (path &amp;&amp; connectingSlash + path)
      }
      matcher = createRouteRecordMatcher(normalizedRecord, parent, options)
      <span class="hljs-keyword">if</span> ( parent &amp;&amp; path[<span class="hljs-number">0</span>] === <span class="hljs-string">'/'</span>)
        checkMissingParamsInAbsolutePath(matcher, parent)
      <span class="hljs-keyword">if</span> (originalRecord) {
        originalRecord.alias.push(matcher)
        {
          checkSameParams(originalRecord, matcher)
        }
      }
      <span class="hljs-keyword">else</span> {
        originalMatcher = originalMatcher || <span class="hljs-function">matcher
        <span class="hljs-title">if</span> <span class="hljs-params">(originalMatcher !== matcher)</span>
          originalMatcher.alias.<span class="hljs-title">push</span><span class="hljs-params">(matcher)</span>
        <span class="hljs-title">if</span> <span class="hljs-params">(isRootAdd &amp;&amp; record.name &amp;&amp; !isAliasRecord(matcher)</span>)
          <span class="hljs-title">removeRoute</span><span class="hljs-params">(record.name)</span>
      }
      <span class="hljs-title">if</span> <span class="hljs-params">(<span class="hljs-string">'children'</span> in mainNormalizedRecord)</span> </span>{
        let children = mainNormalizedRecord.<span class="hljs-function">children
        <span class="hljs-title">for</span> <span class="hljs-params">(let i = <span class="hljs-number">0</span>; i &lt; children.length; i++)</span> </span>{
          addRoute(children[i], matcher, originalRecord &amp;&amp; originalRecord.children[i])
        }
      }
      originalRecord = originalRecord || <span class="hljs-function">matcher
      <span class="hljs-title">insertMatcher</span><span class="hljs-params">(matcher)</span>
    }
    return originalMatcher
      ? <span class="hljs-params">()</span> </span>=&gt; {
        removeRoute(originalMatcher)
      }
      : noop
  }
  
  <span class="hljs-function">function <span class="hljs-title">insertMatcher</span><span class="hljs-params">(matcher)</span> </span>{
    let i = <span class="hljs-number">0</span>
    <span class="hljs-keyword">while</span> (i &lt; matchers.length &amp;&amp;
    comparePathParserScore(matcher, matchers[i]) &gt;= <span class="hljs-number">0</span>)
      i++
    matchers.splice(i, <span class="hljs-number">0</span>, matcher)
    <span class="hljs-keyword">if</span> (matcher.record.name &amp;&amp; !isAliasRecord(matcher))
      matcherMap.set(matcher.record.name, matcher)
  }
 
  <span class="hljs-comment">// 定义其它一些辅助函数</span>
  
  <span class="hljs-comment">// 添加初始路径</span>
  routes.forEach(route =&gt; addRoute(route))
  <span class="hljs-keyword">return</span> { addRoute, resolve, removeRoute, getRoutes, getRecordMatcher }
}
</code></pre>
<p data-nodeid="651">createRouterMatcher 函数内部定义了一个 matchers 数组和一些辅助函数，我们先重点关注 addRoute 函数的实现，我们只关注核心流程。</p>
<p data-nodeid="652">在 createRouterMatcher 函数的最后，会遍历 routes 路径数组调用 addRoute 方法添加初始路径。</p>
<p data-nodeid="653">在 addRoute 函数内部，首先会把 route 对象标准化成一个 record，其实就是给路径对象添加更丰富的属性。</p>
<p data-nodeid="654">然后再执行createRouteRecordMatcher 函数，传入标准化的 record 对象，我们再来看它的实现：</p>
<pre class="lang-java" data-nodeid="655"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">createRouteRecordMatcher</span><span class="hljs-params">(record, parent, options)</span> </span>{
  <span class="hljs-keyword">const</span> parser = tokensToParser(tokenizePath(record.path), options)
  {
    <span class="hljs-keyword">const</span> existingKeys = <span class="hljs-keyword">new</span> Set()
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key of parser.keys) {
      <span class="hljs-keyword">if</span> (existingKeys.has(key.name))
        warn(`Found duplicated params with name <span class="hljs-string">"${key.name}"</span> <span class="hljs-keyword">for</span> path <span class="hljs-string">"${record.path}"</span>. Only the last one will be available on <span class="hljs-string">"$route.params"</span>.`)
      existingKeys.add(key.name)
    }
  }
  <span class="hljs-keyword">const</span> matcher = assign(parser, {
    record,
    parent,
    children: [],
    alias: []
  })
  <span class="hljs-keyword">if</span> (parent) {
    <span class="hljs-keyword">if</span> (!matcher.record.aliasOf === !parent.record.aliasOf)
      parent.children.push(matcher)
  }
  <span class="hljs-keyword">return</span> matcher
}
</code></pre>
<p data-nodeid="656">其实 createRouteRecordMatcher 创建的 matcher 对象不仅仅拥有 record 属性来存储 record，还扩展了一些其他属性，需要注意，如果存在 parent matcher，那么会把当前 matcher 添加到 parent.children 中去，这样就维护了父子关系，构造了树形结构。</p>
<p data-nodeid="657">那么什么情况下会有 parent matcher 呢？让我们先回到 addRoute 函数，在创建了 matcher 对象后，接着判断 record 中是否有 children 属性，如果有则遍历 children，递归执行 addRoute 方法添加路径，并把创建 matcher 作为第二个参数 parent 传入，这也就是 parent matcher 存在的原因。</p>
<p data-nodeid="658">所有 children 处理完毕后，再执行 insertMatcher 函数，把创建的 matcher 存入到 matchers 数组中。</p>
<p data-nodeid="659">至此，我们就根据用户配置的 routes 路径数组，初始化好了 matchers 数组。</p>
<p data-nodeid="660">那么再回到我们前面的问题，分析路径对象中的 matched 的值是怎么在路径切换的情况下更新的。</p>
<p data-nodeid="661">之前我们提到过，切换路径会执行 pushWithRedirect 方法，内部会执行一段代码：</p>
<pre class="lang-java" data-nodeid="662"><code data-language="java"><span class="hljs-keyword">const</span> targetLocation = (pendingLocation = resolve(to))
</code></pre>
<p data-nodeid="663">这里会执行 resolve 函数解析生成 targetLocation，这个 targetLocation 最后也会在 finalizeNavigation 的时候赋值 currentRoute 更新当前路径。我们来看 resolve 函数的实现：</p>
<pre class="lang-java" data-nodeid="664"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">resolve</span><span class="hljs-params">(location, currentLocation)</span> </span>{
  let matcher
  let params = {}
  <span class="hljs-function">let path
  let name
  <span class="hljs-title">if</span> <span class="hljs-params">(<span class="hljs-string">'name'</span> in location &amp;&amp; location.name)</span> </span>{
    matcher = matcherMap.get(location.name)
    <span class="hljs-keyword">if</span> (!matcher)
      <span class="hljs-keyword">throw</span> createRouterError(<span class="hljs-number">1</span> <span class="hljs-comment">/* MATCHER_NOT_FOUND */</span>, {
        location,
      })
    name = matcher.record.name
    params = assign(
      paramsFromLocation(currentLocation.params,
        matcher.keys.filter(k =&gt; !k.optional).map(k =&gt; k.name)), location.params)
    path = matcher.stringify(params)
  }
  <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (<span class="hljs-string">'path'</span> in location) {
    path = location.<span class="hljs-function">path
    <span class="hljs-title">if</span> <span class="hljs-params">( !path.startsWith(<span class="hljs-string">'/'</span>)</span>) </span>{
      warn(`The Matcher cannot resolve relative paths but received <span class="hljs-string">"${path}"</span>. Unless you directly called \`matcher.resolve(<span class="hljs-string">"${path}"</span>)\`, <span class="hljs-keyword">this</span> is probably a bug in vue-router. Please open an issue at https:<span class="hljs-comment">//new-issue.vuejs.org/?repo=vuejs/vue-router-next.`)</span>
    }
    matcher = matchers.find(m =&gt; m.re.test(path))
  
    <span class="hljs-keyword">if</span> (matcher) {
      params = matcher.parse(path)
      name = matcher.record.name
    }
  }
  <span class="hljs-keyword">else</span> {
    matcher = currentLocation.name
      ? matcherMap.get(currentLocation.name)
      : matchers.find(m =&gt; m.re.test(currentLocation.path))
    <span class="hljs-keyword">if</span> (!matcher)
      <span class="hljs-keyword">throw</span> createRouterError(<span class="hljs-number">1</span> <span class="hljs-comment">/* MATCHER_NOT_FOUND */</span>, {
        location,
        currentLocation,
      })
    name = matcher.record.name
    params = assign({}, currentLocation.params, location.params)
    path = matcher.stringify(params)
  }
  <span class="hljs-keyword">const</span> matched = []
  let parentMatcher = <span class="hljs-function">matcher
  <span class="hljs-title">while</span> <span class="hljs-params">(parentMatcher)</span> </span>{
    matched.unshift(parentMatcher.record)
    parentMatcher = parentMatcher.parent
  }
  <span class="hljs-keyword">return</span> {
    name,
    path,
    params,
    matched,
    meta: mergeMetaFields(matched),
  }
}
</code></pre>
<p data-nodeid="665">resolve 函数主要做的事情就是根据 location 的 name 或者 path 从我们前面创建的 matchers 数组中找到对应的 matcher，然后再顺着 matcher 的 parent 一直找到链路上所有匹配的 matcher，然后获取其中的 record 属性构造成一个 matched 数组，最终返回包含 matched 属性的新的路径对象。</p>
<p data-nodeid="666">这么做的目的就是让 matched 数组完整记录 record 路径，它的顺序和嵌套的 RouterView 组件顺序一致，也就是 matched 数组中的第 n 个元素就代表着 RouterView 嵌套的第 n 层。</p>
<p data-nodeid="667">因此 targetLocation 和 to 相比，其实就是多了一个 matched 对象，这样再回到我们的 RouterView 组件，就可以从<code data-backticks="1" data-nodeid="714">injectedRoute.matched[depth] [props.name]</code>中拿到对应的组件对象定义，去渲染对应的组件了。</p>
<p data-nodeid="668">至此，我们就搞清楚路径和路由组件的渲染是如何映射的了。</p>
<p data-nodeid="669">前面的分析过程中，我们提到过在路径切换过程中，会执行 navigate 方法，它包含了一系列的导航守卫钩子函数的执行，接下来我们就来分析这部分的实现原理。</p>
<h3 data-nodeid="670">导航守卫的实现</h3>
<p data-nodeid="671">导航守卫主要是让用户在路径切换的生命周期中可以注入钩子函数，执行一些自己的逻辑，也可以取消和重定向导航，举个应用的例子：</p>
<pre class="lang-java" data-nodeid="672"><code data-language="java">router.beforeEach((to, from, next) =&gt; {
  <span class="hljs-keyword">if</span> (to.name !== <span class="hljs-string">'Login'</span> &amp;&amp; !isAuthenticated) next({ name: <span class="hljs-string">'Login'</span> }) <span class="hljs-keyword">else</span> {
    next()
  }
})
</code></pre>
<p data-nodeid="673">这里大致含义就是进入路由前检查用户是否登录，如果没有则跳转到登录的视图组件，否则继续。</p>
<p data-nodeid="674">router.beforeEach 传入的参数是一个函数，我们把这类函数就称为导航守卫。</p>
<p data-nodeid="675">那么这些导航守卫是怎么执行的呢？这里我并不打算去详细讲 navigate 实现的完整流程，而是讲清楚它的执行原理，关于导航守卫的执行顺序建议你去对照<a href="https://next.router.vuejs.org/guide/advanced/navigation-guards.html" data-nodeid="725">官网文档</a>，然后再来看实现细节。</p>
<p data-nodeid="676">接下来，我们来看 navigate 函数的实现：</p>
<pre class="lang-java" data-nodeid="677"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">navigate</span><span class="hljs-params">(to, from)</span> </span>{
  let guards
  <span class="hljs-keyword">const</span> [leavingRecords, updatingRecords, enteringRecords,] = extractChangingRecords(to, from)
  guards = extractComponentsGuards(leavingRecords.reverse(), <span class="hljs-string">'beforeRouteLeave'</span>, to, from)
  <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> record of leavingRecords) {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> guard of record.leaveGuards) {
      guards.push(guardToPromiseFn(guard, to, from))
    }
  }
  <span class="hljs-keyword">const</span> canceledNavigationCheck = checkCanceledNavigationAndReject.bind(<span class="hljs-keyword">null</span>, to, from)
  guards.push(canceledNavigationCheck)
  <span class="hljs-keyword">return</span> (runGuardQueue(guards)
    .then(() =&gt; {
      guards = []
      <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> guard of beforeGuards.list()) {
        guards.push(guardToPromiseFn(guard, to, from))
      }
      guards.push(canceledNavigationCheck)
      <span class="hljs-keyword">return</span> runGuardQueue(guards)
    })
    .then(() =&gt; {
      guards = extractComponentsGuards(updatingRecords, <span class="hljs-string">'beforeRouteUpdate'</span>, to, from)
      <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> record of updatingRecords) {
        <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> guard of record.updateGuards) {
          guards.push(guardToPromiseFn(guard, to, from))
        }
      }
      guards.push(canceledNavigationCheck)
      <span class="hljs-keyword">return</span> runGuardQueue(guards)
    })
    .then(() =&gt; {
      guards = []
      <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> record of to.matched) {
        <span class="hljs-keyword">if</span> (record.beforeEnter &amp;&amp; from.matched.indexOf(record) &lt; <span class="hljs-number">0</span>) {
          <span class="hljs-keyword">if</span> (Array.isArray(record.beforeEnter)) {
            <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> beforeEnter of record.beforeEnter)
              guards.push(guardToPromiseFn(beforeEnter, to, from))
          }
          <span class="hljs-keyword">else</span> {
            guards.push(guardToPromiseFn(record.beforeEnter, to, from))
          }
        }
      }
      guards.push(canceledNavigationCheck)
      <span class="hljs-keyword">return</span> runGuardQueue(guards)
    })
    .then(() =&gt; {
      to.matched.forEach(record =&gt; (record.enterCallbacks = {}))
      guards = extractComponentsGuards(enteringRecords, <span class="hljs-string">'beforeRouteEnter'</span>, to, from)
      guards.push(canceledNavigationCheck)
      <span class="hljs-keyword">return</span> runGuardQueue(guards)
    })
    .then(() =&gt; {
      guards = []
      <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> guard of beforeResolveGuards.list()) {
        guards.push(guardToPromiseFn(guard, to, from))
      }
      guards.push(canceledNavigationCheck)
      <span class="hljs-keyword">return</span> runGuardQueue(guards)
    })
    .<span class="hljs-keyword">catch</span>(err =&gt; isNavigationFailure(err, <span class="hljs-number">8</span> <span class="hljs-comment">/* NAVIGATION_CANCELLED */</span>)
      ? err
      : Promise.reject(err)))
}
</code></pre>
<p data-nodeid="678">可以看到 navigate 执行导航守卫的方式是先构造 guards 数组，数组中每个元素都是一个返回 Promise 对象的函数。</p>
<p data-nodeid="679">然后通过 runGuardQueue 去执行这些 guards，来看它的实现：</p>
<pre class="lang-java" data-nodeid="680"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">runGuardQueue</span><span class="hljs-params">(guards)</span> </span>{
  <span class="hljs-keyword">return</span> guards.reduce((promise, guard) =&gt; promise.then(() =&gt; guard()), Promise.resolve())
}
</code></pre>
<p data-nodeid="681">其实就是通过数组的 reduce 方法，链式执行 guard 函数，每个 guard 函数都会返回一个 Promise对象。</p>
<p data-nodeid="682">但是从我们的例子看，我们添加的是一个普通函数，并不是一个返回 Promise对象的函数，那是怎么做的呢？</p>
<p data-nodeid="683">原来在把 guard 添加到 guards 数组前，都会执行 guardToPromiseFn 函数把普通函数 Promise化，来看它的实现：</p>
<pre class="lang-java" data-nodeid="684"><code data-language="java"><span class="hljs-keyword">import</span> { warn as warn$<span class="hljs-number">1</span> } from <span class="hljs-string">"vue/dist/vue"</span>
<span class="hljs-function">function <span class="hljs-title">guardToPromiseFn</span><span class="hljs-params">(guard, to, from, record, name)</span> </span>{
  <span class="hljs-keyword">const</span> enterCallbackArray = record &amp;&amp;
    (record.enterCallbacks[name] = record.enterCallbacks[name] || [])
  <span class="hljs-keyword">return</span> () =&gt; <span class="hljs-keyword">new</span> Promise((resolve, reject) =&gt; {
    <span class="hljs-keyword">const</span> next = (valid) =&gt; {
      <span class="hljs-keyword">if</span> (valid === <span class="hljs-keyword">false</span>)
        reject(createRouterError(<span class="hljs-number">4</span> <span class="hljs-comment">/* NAVIGATION_ABORTED */</span>, {
          from,
          to,
        }))
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (valid <span class="hljs-keyword">instanceof</span> Error) {
        reject(valid)
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (isRouteLocation(valid)) {
        reject(createRouterError(<span class="hljs-number">2</span> <span class="hljs-comment">/* NAVIGATION_GUARD_REDIRECT */</span>, {
          from: to,
          to: valid
        }))
      }
      <span class="hljs-keyword">else</span> {
        <span class="hljs-keyword">if</span> (enterCallbackArray &amp;&amp;
          record.enterCallbacks[name] === enterCallbackArray &amp;&amp;
          typeof valid === <span class="hljs-string">'function'</span>)
          enterCallbackArray.push(valid)
        resolve()
      }
    }
    <span class="hljs-keyword">const</span> guardReturn = guard.call(record &amp;&amp; record.instances[name], to, from, next )
    let guardCall = Promise.resolve(guardReturn)
    <span class="hljs-keyword">if</span> (guard.length &lt; <span class="hljs-number">3</span>)
      guardCall = guardCall.then(next)
    <span class="hljs-keyword">if</span> (guard.length &gt; <span class="hljs-number">2</span>) {
      <span class="hljs-keyword">const</span> message = `The <span class="hljs-string">"next"</span> callback was never called inside of ${guard.name ? <span class="hljs-string">'"'</span> + guard.name + <span class="hljs-string">'"'</span> : <span class="hljs-string">''</span>}:\n${guard.toString()}\n. If you are returning a value instead of calling <span class="hljs-string">"next"</span>, make sure to remove the <span class="hljs-string">"next"</span> parameter from your function.`
      <span class="hljs-keyword">if</span> (typeof guardReturn === <span class="hljs-string">'object'</span> &amp;&amp; <span class="hljs-string">'then'</span> in guardReturn) {
        guardCall = guardCall.then(resolvedValue =&gt; {
          <span class="hljs-comment">// @ts-ignore: _called is added at canOnlyBeCalledOnce</span>
          <span class="hljs-keyword">if</span> (!next._called) {
            warn$<span class="hljs-number">1</span>(message)
            <span class="hljs-keyword">return</span> Promise.reject(<span class="hljs-keyword">new</span> Error(<span class="hljs-string">'Invalid navigation guard'</span>))
          }
          <span class="hljs-keyword">return</span> resolvedValue
        })
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (guardReturn !== undefined) {
        <span class="hljs-keyword">if</span> (!next._called) {
          warn$<span class="hljs-number">1</span>(message)
          reject(<span class="hljs-keyword">new</span> Error(<span class="hljs-string">'Invalid navigation guard'</span>))
          <span class="hljs-keyword">return</span>
        }
      }
    }
    guardCall.<span class="hljs-keyword">catch</span>(err =&gt; reject(err))
  })
}
</code></pre>
<p data-nodeid="685">guardToPromiseFn 函数返回一个新的函数，这个函数内部会执行 guard 函数。</p>
<p data-nodeid="686">这里我们要注意 next 方法的设计，当我们在导航守卫中执行 next 时，实际上就是执行这里定义的 next 函数。</p>
<p data-nodeid="687">在执行 next 函数时，如果不传参数，那么则直接 resolve，执行下一个导航守卫；如果参数是 false，则创建一个导航取消的错误 reject 出去；如果参数是一个 Error 实例，则直接执行 reject，并把错误传递出去；如果参数是一个路径对象，则创建一个导航重定向的错误传递出去。</p>
<p data-nodeid="688">有些时候我们写导航守卫不使用 next 函数，而是直接返回 true 或 false，这种情况则先执行如下代码：</p>
<pre class="lang-js" data-nodeid="996"><code data-language="js">guardCall = <span class="hljs-built_in">Promise</span>.resolve(guardReturn)
</code></pre>
<p data-nodeid="997">把导航守卫的返回值 Promise化，然后再执行 guardCall.then(next)，把导航守卫的返回值传给 next 函数。</p>
<p data-nodeid="998">当然，如果你在导航守卫中定义了第三个参数 next，但是你没有在函数中调用它，这种情况也会报警告。</p>
<p data-nodeid="999">所以，对于导航守卫而言，经过 Promise化后添加到 guards 数组中，然后再通过 runGuards 以及 Promise 的方式链式调用，最终依次顺序执行这些导航守卫。</p>
<h3 data-nodeid="1000">总结</h3>
<p data-nodeid="1001">好的，到这里我们这一节的学习也要结束啦，通过这节课的学习，你应该要了解 Vue Router 的基本实现原理，知道路径是如何管理的，路径和路由组件的渲染是如何映射的，导航守卫是如何执行的。</p>
<p data-nodeid="1002">当然，路由实现的细节是非常多的，我希望你学完之后，可以对照着官网的文档的 feature，自行去分析它们的实现原理。</p>
<p data-nodeid="1003">最后，给你留一道思考题目，如果我们想给路由组件传递数据，有几种方式，分别都怎么做呢？欢迎你在留言区与我分享。</p>

---

### 精选评论


