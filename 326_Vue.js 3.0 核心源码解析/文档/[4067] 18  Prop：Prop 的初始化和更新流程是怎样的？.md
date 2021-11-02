<p data-nodeid="10403">前面我们提到过 Vue.js 的核心思想之一是组件化，页面可以由一个个组件构建而成，组件是一种抽象的概念，它是对页面的部分布局和逻辑的封装。</p>



<p data-nodeid="9668">为了让组件支持各种丰富的功能，Vue.js 设计了 Props 特性，它允许组件的使用者在外部传递 Props，然后组件内部就可以根据这些 Props 去实现各种各样的功能。</p>
<p data-nodeid="9669">为了让你更直观地理解，我们来举个例子，假设有这样一个 BlogPost 组件，它是这样定义的：</p>
<pre class="lang-js" data-nodeid="12829"><code data-language="js">&lt;div <span class="hljs-class"><span class="hljs-keyword">class</span></span>=<span class="hljs-string">"blog-post"</span>&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>{{title}}<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>author: {{author}}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
&lt;/div&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">props</span>: {
      <span class="hljs-attr">title</span>: <span class="hljs-built_in">String</span>,
      <span class="hljs-attr">author</span>: <span class="hljs-built_in">String</span>
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>





<p data-nodeid="9671">然后我们在父组件使用这个 BlogPost 组件的时候，可以给它传递一些 Props 数据：</p>
<pre class="lang-js" data-nodeid="13799"><code data-language="js">&lt;blog-post title=<span class="hljs-string">"Vue3 publish"</span> author=<span class="hljs-string">"yyx"</span>&gt;&lt;/blog-post&gt;
</code></pre>


<p data-nodeid="9673">从最终结果来看，BlogPost 组件会渲染传递的 title 和 author 数据。</p>
<p data-nodeid="9674" class="">我们平时写组件，会经常和 Props 打交道，但你知道 Vue.js 内部是如何初始化以及更新 Props 的呢？Vue.js 3.0 在 props 的 API 设计上和 Vue.js 2.x 保持一致，那它们的底层实现层面有没有不一样的地方呢？带着这些疑问，让我们来一起探索 Props 的相关实现原理吧。</p>
<h3 data-nodeid="15739" class="">Props 的初始化</h3>




<p data-nodeid="9676">首先，我们来了解 Props 的初始化过程。之前在介绍 Setup 组件初始化的章节，我们介绍了在执行 setupComponent 函数的时候，会初始化 Props：</p>
<pre class="lang-java" data-nodeid="16219"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">setupComponent</span> <span class="hljs-params">(instance, isSSR = <span class="hljs-keyword">false</span>)</span> </span>{
  <span class="hljs-keyword">const</span> { props, children, shapeFlag } = instance.vnode
  <span class="hljs-comment">// 判断是否是一个有状态的组件</span>
  <span class="hljs-keyword">const</span> isStateful = shapeFlag &amp; <span class="hljs-number">4</span>
  <span class="hljs-comment">// 初始化 props</span>
  initProps(instance, props, isStateful, isSSR)
  <span class="hljs-comment">// 初始化插槽</span>
  initSlots(instance, children)
  <span class="hljs-comment">// 设置有状态的组件实例</span>
  <span class="hljs-keyword">const</span> setupResult = isStateful
    ? setupStatefulComponent(instance, isSSR)
    : undefined
  <span class="hljs-keyword">return</span> setupResult
}
</code></pre>

<p data-nodeid="9678">所以 Props 初始化，就是通过 initProps 方法来完成的，我们来看一下它的实现：</p>
<pre class="lang-java" data-nodeid="16698"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">initProps</span><span class="hljs-params">(instance, rawProps, isStateful, isSSR = <span class="hljs-keyword">false</span>)</span> </span>{
  <span class="hljs-keyword">const</span> props = {}
  <span class="hljs-keyword">const</span> attrs = {}
  def(attrs, InternalObjectKey, <span class="hljs-number">1</span>)
  <span class="hljs-comment">// 设置 props 的值</span>
  setFullProps(instance, rawProps, props, attrs)
  <span class="hljs-comment">// 验证 props 合法</span>
  <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
    validateProps(props, instance.type)
  }
  <span class="hljs-keyword">if</span> (isStateful) {
    <span class="hljs-comment">// 有状态组件，响应式处理</span>
    instance.props = isSSR ? props : shallowReactive(props)
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 函数式组件处理</span>
    <span class="hljs-keyword">if</span> (!instance.type.props) {
      instance.props = attrs
    }
    <span class="hljs-keyword">else</span> {
      instance.props = props
    }
  }
  <span class="hljs-comment">// 普通属性赋值</span>
  instance.attrs = attrs
}
</code></pre>

<p data-nodeid="9680">这里，初始化 Props 主要做了以下几件事情：<strong data-nodeid="9804">设置 props 的值</strong>，<strong data-nodeid="9805">验证 props 是否合法</strong>，<strong data-nodeid="9806">把 props 变成响应式</strong>，<strong data-nodeid="9807">以及添加到实例 instance.props 上</strong>。</p>
<p data-nodeid="9681">注意，这里我们只分析有状态组件的 Props 初始化过程，所以就默认 isStateful 的值是 true。所谓有状态组件，就是你平时通过对象的方式定义的组件。</p>
<p data-nodeid="9682">接下来，我们来看设置 Props 的流程。</p>
<h4 data-nodeid="17177">设置 Props</h4>


<p data-nodeid="9685">我们看一下 setFullProps 的实现：</p>
<pre class="lang-java" data-nodeid="17655"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">setFullProps</span><span class="hljs-params">(instance, rawProps, props, attrs)</span> </span>{
  <span class="hljs-comment">// 标准化 props 的配置</span>
  <span class="hljs-keyword">const</span> [options, needCastKeys] = normalizePropsOptions(instance.type)
  <span class="hljs-keyword">if</span> (rawProps) {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key in rawProps) {
      <span class="hljs-keyword">const</span> value = rawProps[key]
      <span class="hljs-comment">// 一些保留的 prop 比如 ref、key 是不会传递的</span>
      <span class="hljs-keyword">if</span> (isReservedProp(key)) {
        <span class="hljs-keyword">continue</span>
      }
      <span class="hljs-comment">// 连字符形式的 props 也转成驼峰形式</span>
      <span class="hljs-function">let camelKey
      <span class="hljs-title">if</span> <span class="hljs-params">(options &amp;&amp; hasOwn(options, (camelKey = camelize(key)</span>))) </span>{
        props[camelKey] = value
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!isEmitListener(instance.type, key)) {
        <span class="hljs-comment">// 非事件派发相关的，且不在 props 中定义的普通属性用 attrs 保留</span>
        attrs[key] = value
      }
    }
  }
  <span class="hljs-keyword">if</span> (needCastKeys) {
    <span class="hljs-comment">// 需要做转换的 props</span>
    <span class="hljs-keyword">const</span> rawCurrentProps = toRaw(props)
    <span class="hljs-keyword">for</span> (let i = <span class="hljs-number">0</span>; i &lt; needCastKeys.length; i++) {
      <span class="hljs-keyword">const</span> key = needCastKeys[i]
      props[key] = resolvePropValue(options, rawCurrentProps, key, rawCurrentProps[key])
    }
  }
}
</code></pre>

<p data-nodeid="18132">我们先注意函数的几个参数的含义：instance 表示组件实例；rawProps 表示原始的 props 值，也就是创建 vnode 过程中传入的 props 数据；props 用于存储解析后的 props 数据；attrs 用于存储解析后的普通属性数据。</p>
<p data-nodeid="18133">设置 Props 的过程也分成几个步骤：标准化 props 的配置，遍历 props 数据求值，以及对需要转换的 props 求值。</p>

<p data-nodeid="9688">接下来，我们来看标准化 props 配置的过程，先看一下 normalizePropsOptions 函数的实现：</p>
<pre class="lang-java" data-nodeid="18612"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">normalizePropsOptions</span><span class="hljs-params">(comp)</span> </span>{
  <span class="hljs-comment">// comp.__props 用于缓存标准化的结果，有缓存，则直接返回</span>
  <span class="hljs-keyword">if</span> (comp.__props) {
    <span class="hljs-keyword">return</span> comp.__props
  }
  <span class="hljs-keyword">const</span> raw = comp.props
  <span class="hljs-keyword">const</span> normalized = {}
  <span class="hljs-keyword">const</span> needCastKeys = []
  <span class="hljs-comment">// 处理 mixins 和 extends 这些 props</span>
  let hasExtends = <span class="hljs-function"><span class="hljs-keyword">false</span>
  <span class="hljs-title">if</span> <span class="hljs-params">(!shared.isFunction(comp)</span>) </span>{
    <span class="hljs-keyword">const</span> extendProps = (raw) =&gt; {
      <span class="hljs-keyword">const</span> [props, keys] = normalizePropsOptions(raw)
      shared.extend(normalized, props)
      <span class="hljs-keyword">if</span> (keys)
        needCastKeys.push(...keys)
    }
    <span class="hljs-keyword">if</span> (comp.extends) {
      hasExtends = <span class="hljs-function"><span class="hljs-keyword">true</span>
      <span class="hljs-title">extendProps</span><span class="hljs-params">(comp.extends)</span>
    }
    <span class="hljs-title">if</span> <span class="hljs-params">(comp.mixins)</span> </span>{
      hasExtends = <span class="hljs-keyword">true</span>
      comp.mixins.forEach(extendProps)
    }
  }
  <span class="hljs-keyword">if</span> (!raw &amp;&amp; !hasExtends) {
    <span class="hljs-keyword">return</span> (comp.__props = shared.EMPTY_ARR)
  }
  <span class="hljs-comment">// 数组形式的 props 定义</span>
  <span class="hljs-keyword">if</span> (shared.isArray(raw)) {
    <span class="hljs-keyword">for</span> (let i = <span class="hljs-number">0</span>; i &lt; raw.length; i++) {
      <span class="hljs-keyword">if</span> (!shared.isString(raw[i])) {
        warn(`props must be strings when using array syntax.`, raw[i])
      }
      <span class="hljs-keyword">const</span> normalizedKey = shared.camelize(raw[i])
      <span class="hljs-keyword">if</span> (validatePropName(normalizedKey)) {
        normalized[normalizedKey] = shared.EMPTY_OBJ
      }
    }
  }
  <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (raw) {
    <span class="hljs-keyword">if</span> (!shared.isObject(raw)) {
      warn(`invalid props options`, raw)
    }
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key in raw) {
      <span class="hljs-keyword">const</span> normalizedKey = shared.camelize(key)
      <span class="hljs-keyword">if</span> (validatePropName(normalizedKey)) {
        <span class="hljs-keyword">const</span> opt = raw[key]
        <span class="hljs-comment">// 标准化 prop 的定义格式</span>
        <span class="hljs-keyword">const</span> prop = (normalized[normalizedKey] =
          shared.isArray(opt) || shared.isFunction(opt) ? { type: opt } : opt)
        <span class="hljs-keyword">if</span> (prop) {
          <span class="hljs-keyword">const</span> booleanIndex = getTypeIndex(Boolean, prop.type)
          <span class="hljs-keyword">const</span> stringIndex = getTypeIndex(String, prop.type)
          prop[<span class="hljs-number">0</span> <span class="hljs-comment">/* shouldCast */</span>] = booleanIndex &gt; -<span class="hljs-number">1</span>
          prop[<span class="hljs-number">1</span> <span class="hljs-comment">/* shouldCastTrue */</span>] =
            stringIndex &lt; <span class="hljs-number">0</span> || booleanIndex &lt; stringIndex
          <span class="hljs-comment">// 布尔类型和有默认值的 prop 都需要转换</span>
          <span class="hljs-keyword">if</span> (booleanIndex &gt; -<span class="hljs-number">1</span> || shared.hasOwn(prop, <span class="hljs-string">'default'</span>)) {
            needCastKeys.push(normalizedKey)
          }
        }
      }
    }
  }
  <span class="hljs-keyword">const</span> normalizedEntry = [normalized, needCastKeys]
  comp.__props = normalizedEntry
  <span class="hljs-keyword">return</span> normalizedEntry
}
</code></pre>

<p data-nodeid="9690">normalizePropsOptions 主要目的是标准化 props 的配置，这里需要注意，你要区分 props 的配置和 props 的数据。所谓 props 的配置，就是你在定义组件时编写的 props 配置，它用来描述一个组件的 props 是什么样的；而 props 的数据，是父组件在调用子组件的时候，给子组件传递的数据。</p>
<p data-nodeid="9691">所以这个函数首先会处理 mixins 和 extends 这两个特殊的属性，因为它们的作用都是扩展组件的定义，所以需要对它们定义中的 props 递归执行 normalizePropsOptions。</p>
<p data-nodeid="9692">接着，函数会处理数组形式的 props 定义，例如：</p>
<pre class="lang-java" data-nodeid="19089"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: [<span class="hljs-string">'name'</span>, <span class="hljs-string">'nick-name'</span>]
}
</code></pre>

<p data-nodeid="9694">如果 props 被定义成数组形式，那么数组的每个元素必须是一个字符串，然后把字符串都变成驼峰形式作为 key，并为normalized 的 key 对应的每一个值创建一个空对象。针对上述示例，最终标准化的 props 的定义是这样的：</p>
<pre class="lang-java" data-nodeid="19566"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: {
    name: {},
    nickName: {}
  }
}
</code></pre>

<p data-nodeid="9696">如果 props 定义是一个对象形式，接着就是标准化它的每一个 prop 的定义，把数组或者函数形式的 prop 标准化成对象形式，例如：</p>
<pre class="lang-java" data-nodeid="20043"><code data-language="java">export <span class="hljs-keyword">default</span> {
  title: String,
  author: [String, Boolean]
}
</code></pre>

<p data-nodeid="9698">注意，上述代码中的 String 和 Boolean 都是内置的构造器函数。经过标准化的 props 的定义：</p>
<pre class="lang-java" data-nodeid="20520"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: {
    title: {
      type: String
    },
    author: {
      type: [String, Boolean]
    }
  }
}
</code></pre>

<p data-nodeid="9700">接下来，就是判断一些 prop 是否需要转换，其中，含有布尔类型的 prop 和有默认值的 prop 需要转换，这些 prop 的 key 保存在 needCastKeys 中。注意，这里会给 prop 添加两个特殊的 key，prop[0] 和 prop[1]赋值，它们的作用后续我们会说。</p>
<p data-nodeid="9701">最后，返回标准化结果 normalizedEntry，它包含标准化后的 props 定义 normalized，以及需要转换的 props key needCastKeys，并且用 comp.__props 缓存这个标准化结果，如果对同一个组件重复执行 normalizePropsOptions，直接返回这个标准化结果即可。</p>
<p data-nodeid="9702">标准化 props 配置的目的无非就是支持用户各种的 props 配置写法，标准化统一的对象格式为了后续统一处理。</p>
<p data-nodeid="9703">我们回到 setFullProps 函数，接下来分析遍历 props 数据求值的流程。</p>
<pre class="lang-java" data-nodeid="21951"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">setFullProps</span><span class="hljs-params">(instance, rawProps, props, attrs)</span> </span>{
  <span class="hljs-comment">// 标准化 props 的配置</span>
  
  <span class="hljs-keyword">if</span> (rawProps) {
    <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key in rawProps) {
      <span class="hljs-keyword">const</span> value = rawProps[key]
      <span class="hljs-comment">// 一些保留的 prop 比如 ref、key 是不会传递的</span>
      <span class="hljs-keyword">if</span> (isReservedProp(key)) {
        <span class="hljs-keyword">continue</span>
      }
      <span class="hljs-comment">// 连字符形式的 props 也转成驼峰形式</span>
      <span class="hljs-function">let camelKey
      <span class="hljs-title">if</span> <span class="hljs-params">(options &amp;&amp; hasOwn(options, (camelKey = camelize(key)</span>))) </span>{
        props[camelKey] = value
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (!isEmitListener(instance.type, key)) {
        <span class="hljs-comment">// 非事件派发相关的，且不在 props 中定义的普通属性用 attrs 保留</span>
        attrs[key] = value
      }
    }
  }
  
  <span class="hljs-comment">// 转换需要转换的 props</span>
}
</code></pre>



<p data-nodeid="9705">该过程主要就是遍历 rawProps，拿到每一个 key。由于我们在标准化 props 配置过程中已经把 props 定义的 key 转成了驼峰形式，所以也需要把 rawProps 的 key 转成驼峰形式，然后对比看 prop 是否在配置中定义。</p>
<p data-nodeid="9706">如果 rawProps 中的 prop 在配置中定义了，那么把它的值赋值到 props 对象中，如果不是，那么判断这个 key 是否为非事件派发相关，如果是那么则把它的值赋值到 attrs 对象中。另外，在遍历的过程中，遇到 key、ref 这种 key，则直接跳过。</p>
<p data-nodeid="9707">接下来我们来看 setFullProps 的最后一个流程：对需要转换的 props 求值。</p>
<pre class="lang-java" data-nodeid="22428"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">setFullProps</span><span class="hljs-params">(instance, rawProps, props, attrs)</span> </span>{
  <span class="hljs-comment">// 标准化 props 的配置</span>
  
  <span class="hljs-comment">// 遍历 props 数据求值</span>
  
  <span class="hljs-keyword">if</span> (needCastKeys) {
    <span class="hljs-comment">// 需要做转换的 props</span>
    <span class="hljs-keyword">const</span> rawCurrentProps = toRaw(props)
    <span class="hljs-keyword">for</span> (let i = <span class="hljs-number">0</span>; i &lt; needCastKeys.length; i++) {
      <span class="hljs-keyword">const</span> key = needCastKeys[i]
      props[key] = resolvePropValue(options, rawCurrentProps, key, rawCurrentProps[key])
    }
  }
}
</code></pre>

<p data-nodeid="9709">在 normalizePropsOptions 的时候，我们拿到了需要转换的 props 的 key，接下来就是遍历 needCastKeys，依次执行 resolvePropValue 方法来求值。我们来看一下它的实现：</p>
<pre class="lang-java" data-nodeid="22905"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">resolvePropValue</span><span class="hljs-params">(options, props, key, value)</span> </span>{
  <span class="hljs-keyword">const</span> opt = options[key]
  <span class="hljs-keyword">if</span> (opt != <span class="hljs-keyword">null</span>) {
    <span class="hljs-keyword">const</span> hasDefault = hasOwn(opt, <span class="hljs-string">'default'</span>)
    <span class="hljs-comment">// 默认值处理</span>
    <span class="hljs-keyword">if</span> (hasDefault &amp;&amp; value === undefined) {
      <span class="hljs-keyword">const</span> defaultValue = opt.<span class="hljs-keyword">default</span>
      value =
        opt.type !== Function &amp;&amp; isFunction(defaultValue)
          ? defaultValue()
          : defaultValue
    }
    <span class="hljs-comment">// 布尔类型转换</span>
    <span class="hljs-keyword">if</span> (opt[<span class="hljs-number">0</span> <span class="hljs-comment">/* shouldCast */</span>]) {
      <span class="hljs-keyword">if</span> (!hasOwn(props, key) &amp;&amp; !hasDefault) {
        value = <span class="hljs-keyword">false</span>
      }
      <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (opt[<span class="hljs-number">1</span> <span class="hljs-comment">/* shouldCastTrue */</span>] &amp;&amp;
        (value === <span class="hljs-string">''</span> || value === hyphenate(key))) {
        value = <span class="hljs-keyword">true</span>
      }
    }
  }
  <span class="hljs-keyword">return</span> value
}
</code></pre>

<p data-nodeid="9711">resolvePropValue 主要就是针对两种情况的转换，第一种是默认值的情况，即我们在 prop 配置中定义了默认值，并且父组件没有传递数据的情况，这里 prop 对应的值就取默认值。</p>
<p data-nodeid="9712">第二种是布尔类型的值，前面我们在 normalizePropsOptions 的时候已经给 prop 的定义添加了两个特殊的 key，所以 opt[0] 为 true 表示这是一个含有 Boolean 类型的 prop，然后判断是否有传对应的值，如果不是且没有默认值的话，就直接转成 false，举个例子：</p>
<pre class="lang-java" data-nodeid="23382"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: {
    author: Boolean
  }
}
</code></pre>

<p data-nodeid="9714">如果父组件调用子组件的时候没有给 author 这个 prop 传值，那么它转换后的值就是 false。</p>
<p data-nodeid="9715">接着看 opt[1] 为 true，并且 props 传值是空字符串或者是 key 字符串的情况，命中这个逻辑表示这是一个含有 Boolean 和 String 类型的 prop，且 Boolean 在 String 前面，例如：</p>
<pre class="lang-java" data-nodeid="23859"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: {
    author: [Boolean, String]
  }
}
</code></pre>

<p data-nodeid="9717">这种时候如果传递的 prop 值是空字符串，或者是 author 字符串，则 prop 的值会被转换成 true。</p>
<p data-nodeid="9718">至此，props 的转换求值结束，整个 setFullProps 函数逻辑也结束了，回顾它的整个流程，我们可以发现<strong data-nodeid="9862">它的主要目的就是对 props 求值</strong>，<strong data-nodeid="9863">然后把求得的值赋值给 props 对象和 attrs 对象中</strong>。</p>
<h4 data-nodeid="9719">验证 Props</h4>
<p data-nodeid="9720">接下来我们再回到 initProps 函数，分析第二个流程：验证 props 是否合法。</p>
<pre class="lang-java" data-nodeid="24336"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">initProps</span><span class="hljs-params">(instance, rawProps, isStateful, isSSR = <span class="hljs-keyword">false</span>)</span> </span>{
  <span class="hljs-keyword">const</span> props = {}
  <span class="hljs-comment">// 设置 props 的值</span>
 
  <span class="hljs-comment">// 验证 props 合法</span>
  <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>)) {
    validateProps(props, instance.type)
  }
}
</code></pre>

<p data-nodeid="9722">验证过程是在非生产环境下执行的，我们来看一下 validateProps 的实现：</p>
<pre class="lang-java" data-nodeid="24813"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">validateProps</span><span class="hljs-params">(props, comp)</span> </span>{
  <span class="hljs-keyword">const</span> rawValues = toRaw(props)
  <span class="hljs-keyword">const</span> options = normalizePropsOptions(comp)[<span class="hljs-number">0</span>]
  <span class="hljs-keyword">for</span> (<span class="hljs-keyword">const</span> key in options) {
    let opt = options[key]
    <span class="hljs-keyword">if</span> (opt == <span class="hljs-keyword">null</span>)
      <span class="hljs-function"><span class="hljs-keyword">continue</span>
    <span class="hljs-title">validateProp</span><span class="hljs-params">(key, rawValues[key], opt, !hasOwn(rawValues, key)</span>)
  }
}
function <span class="hljs-title">validateProp</span><span class="hljs-params">(name, value, prop, isAbsent)</span> </span>{
  <span class="hljs-keyword">const</span> { type, required, validator } = prop
  <span class="hljs-comment">// 检测 required</span>
  <span class="hljs-keyword">if</span> (required &amp;&amp; isAbsent) {
    warn(<span class="hljs-string">'Missing required prop: "'</span> + name + <span class="hljs-string">'"'</span>)
    <span class="hljs-keyword">return</span>
  }
  <span class="hljs-comment">// 虽然没有值但也没有配置 required，直接返回</span>
  <span class="hljs-keyword">if</span> (value == <span class="hljs-keyword">null</span> &amp;&amp; !prop.required) {
    <span class="hljs-keyword">return</span>
  }
  <span class="hljs-comment">// 类型检测</span>
  <span class="hljs-keyword">if</span> (type != <span class="hljs-keyword">null</span> &amp;&amp; type !== <span class="hljs-keyword">true</span>) {
    let isValid = <span class="hljs-keyword">false</span>
    <span class="hljs-keyword">const</span> types = isArray(type) ? type : [type]
    <span class="hljs-keyword">const</span> expectedTypes = []
    <span class="hljs-comment">// 只要指定的类型之一匹配，值就有效</span>
    <span class="hljs-keyword">for</span> (let i = <span class="hljs-number">0</span>; i &lt; types.length &amp;&amp; !isValid; i++) {
      <span class="hljs-keyword">const</span> { valid, expectedType } = assertType(value, types[i])
      expectedTypes.push(expectedType || <span class="hljs-string">''</span>)
      isValid = valid
    }
    <span class="hljs-keyword">if</span> (!isValid) {
      warn(getInvalidTypeMessage(name, value, expectedTypes))
      <span class="hljs-keyword">return</span>
    }
  }
  <span class="hljs-comment">// 自定义校验器</span>
  <span class="hljs-keyword">if</span> (validator &amp;&amp; !validator(value)) {
    warn(<span class="hljs-string">'Invalid prop: custom validator check failed for prop "'</span> + name + <span class="hljs-string">'".'</span>)
  }
}
</code></pre>

<p data-nodeid="9724">顾名思义，validateProps 就是用来检测前面求得的 props 值是否合法，它就是对标准化后的 Props 配置对象进行遍历，拿到每一个配置 opt，然后执行 validateProp 验证。</p>
<p data-nodeid="9725">对于单个 Prop 的配置，我们除了配置它的类型 type，还可以配置 required 表明它的必要性，以及 validator 自定义校验器，举个例子：</p>
<pre class="lang-java" data-nodeid="25290"><code data-language="java">export <span class="hljs-keyword">default</span> {
  props: { 
    value: { 
      type: Number,
      required: <span class="hljs-keyword">true</span>,
      validator(val) {
        <span class="hljs-keyword">return</span> val &gt;= <span class="hljs-number">0</span>
      }
    }
  }
}
</code></pre>

<p data-nodeid="9727">因此 validateProp 首先验证 required 的情况，一旦 prop 配置了 required 为 true，那么必须给它传值，否则会报警告。</p>
<p data-nodeid="9728">接着是验证 prop 值的类型，由于 prop 定义的 type 可以是多个类型的数组，那么只要 prop 的值匹配其中一种类型，就是合法的，否则会报警告。</p>
<p data-nodeid="9729">最后是验证如果配了自定义校验器 validator，那么 prop 的值必须满足自定义校验器的规则，否则会报警告。</p>
<p data-nodeid="9730">相信这些警告你在平时的开发工作中或多或少遇到过，了解了 prop 的验证原理，今后再遇到这些警告，你就能知其然并知其所以然了。</p>
<h4 data-nodeid="9731">响应式处理</h4>
<p data-nodeid="9732">我们再回到 initProps 方法，来看最后一个流程：把 props 变成响应式，添加到实例 instance.props 上。</p>
<pre class="lang-java" data-nodeid="25767"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">initProps</span><span class="hljs-params">(instance, rawProps, isStateful, isSSR = <span class="hljs-keyword">false</span>)</span> </span>{
  <span class="hljs-comment">// 设置 props 的值</span>
  <span class="hljs-comment">// 验证 props 合法</span>
  <span class="hljs-keyword">if</span> (isStateful) {
    <span class="hljs-comment">// 有状态组件，响应式处理</span>
    instance.props = isSSR ? props : shallowReactive(props)
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 函数式组件处理</span>
    <span class="hljs-keyword">if</span> (!instance.type.props) {
      instance.props = attrs
    }
    <span class="hljs-keyword">else</span> {
      instance.props = props
    }
  }
  <span class="hljs-comment">// 普通属性赋值</span>
  instance.attrs = attrs
}
</code></pre>

<p data-nodeid="9734">在前两个流程，我们通过 setFullProps 求值赋值给 props 变量，并对 props 做了检测，接下来，就是把 props 变成响应式，并且赋值到组件的实例上。</p>
<p data-nodeid="9735">至此，Props 的初始化就完成了，相信你可能会有一些疑问，为什么 instance.props 要变成响应式，以及为什么用 shallowReactive API 呢？在接下来的 Props 更新流程的分析中，我来解答这两个问题。</p>
<h3 data-nodeid="9736">Props 的更新</h3>
<p data-nodeid="9737">所谓 Props 的更新主要是指 Props 数据的更新，它最直接的反应是会触发组件的重新渲染，我们可以通过一个简单的示例分析这个过程。例如我们有这样一个子组件 HelloWorld，它是这样定义的：</p>
<pre class="lang-js" data-nodeid="26721"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{{ msg }}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">props</span>: {
      <span class="hljs-attr">msg</span>: <span class="hljs-built_in">String</span>
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>


<p data-nodeid="9739">这里，HelloWorld 组件接受一个 msg prop，然后在模板中渲染这个 msg。</p>
<p data-nodeid="9740">然后我们在 App 父组件中引入这个子组件，它的定义如下：</p>
<pre class="lang-js" data-nodeid="27675"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">hello-world</span> <span class="hljs-attr">:msg</span>=<span class="hljs-string">"msg"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">hello-world</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"toggleMsg"</span>&gt;</span>Toggle Msg<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">import</span> HelloWorld <span class="hljs-keyword">from</span> <span class="hljs-string">'./components/HelloWorld'</span>
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">components</span>: { HelloWorld },
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">msg</span>: <span class="hljs-string">'Hello world'</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      toggleMsg() {
        <span class="hljs-built_in">this</span>.msg = <span class="hljs-built_in">this</span>.msg === <span class="hljs-string">'Hello world'</span> ? <span class="hljs-string">'Hello Vue'</span> : <span class="hljs-string">'Hello world'</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>


<p data-nodeid="9742">我们给 HelloWorld 子组件传递的 prop 值是 App 组件中定义的 msg 变量，它的初始值是 Hello world，在子组件的模板中会显示出来。</p>
<p data-nodeid="9743">接着当我们点击按钮修改 msg 的值的时候，就会触发父组件的重新渲染，因为我们在模板中引用了这个 msg 变量。我们会发现这时 HelloWorld 子组件显示的字符串变成了 Hello Vue，那么子组件是如何被触发重新渲染的呢？</p>
<p data-nodeid="9744">在组件更新的章节我们说过，组件的重新渲染会触发 patch 过程，然后遍历子节点递归 patch，那么遇到组件节点，会执行 updateComponent 方法：</p>
<pre class="lang-java" data-nodeid="28152"><code data-language="java"><span class="hljs-keyword">const</span> updateComponent = (n1, n2, parentComponent, optimized) =&gt; {
  <span class="hljs-keyword">const</span> instance = (n2.component = n1.component)
  <span class="hljs-comment">// 根据新旧子组件 vnode 判断是否需要更新子组件</span>
  <span class="hljs-keyword">if</span> (shouldUpdateComponent(n1, n2, parentComponent, optimized)) {
    <span class="hljs-comment">// 新的子组件 vnode 赋值给 instance.next</span>
    instance.next = n2
    <span class="hljs-comment">// 子组件也可能因为数据变化被添加到更新队列里了，移除它们防止对一个子组件重复更新</span>
    invalidateJob(instance.update)
    <span class="hljs-comment">// 执行子组件的副作用渲染函数</span>
    instance.update()
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 不需要更新，只复制属性</span>
    n2.component = n1.component
    n2.el = n1.el
  }
}
</code></pre>

<p data-nodeid="9746">在这个过程中，会执行 shouldUpdateComponent 方法判断是否需要更新子组件，内部会对比 props，由于我们的 prop 数据 msg 由 Hello world 变成了 Hello Vue，值不一样所以 shouldUpdateComponent 会返回 true，这样就把新的子组件 vnode 赋值给 instance.next，然后执行 instance.update 触发子组件的重新渲染。</p>
<p data-nodeid="9747">所以这就是触发子组件重新渲染的原因，但是子组件重新渲染了，子组件实例的 instance.props 的数据需要更新才行，不然还是渲染之前的数据，那么是如何更新 instance.props 的呢，我们接着往下看。</p>
<p data-nodeid="9748">执行 instance.update 函数，实际上是执行 componentEffect 组件副作用渲染函数：</p>
<pre class="lang-java" data-nodeid="28629"><code data-language="java"><span class="hljs-keyword">const</span> setupRenderEffect = (instance, initialVNode, container, anchor, parentSuspense, isSVG, optimized) =&gt; {
  <span class="hljs-comment">// 创建响应式的副作用渲染函数</span>
  instance.update = effect(<span class="hljs-function">function <span class="hljs-title">componentEffect</span><span class="hljs-params">()</span> </span>{
    <span class="hljs-keyword">if</span> (!instance.isMounted) {
      <span class="hljs-comment">// 渲染组件</span>
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 更新组件</span>
      let { next, vnode } = instance
      <span class="hljs-comment">// next 表示新的组件 vnode</span>
      <span class="hljs-keyword">if</span> (next) {
        <span class="hljs-comment">// 更新组件 vnode 节点信息</span>
        updateComponentPreRender(instance, next, optimized)
      }
      <span class="hljs-keyword">else</span> {
        next = vnode
      }
      <span class="hljs-comment">// 渲染新的子树 vnode</span>
      <span class="hljs-keyword">const</span> nextTree = renderComponentRoot(instance)
      <span class="hljs-comment">// 缓存旧的子树 vnode</span>
      <span class="hljs-keyword">const</span> prevTree = instance.subTree
      <span class="hljs-comment">// 更新子树 vnode</span>
      instance.subTree = nextTree
      <span class="hljs-comment">// 组件更新核心逻辑，根据新旧子树 vnode 做 patch</span>
      patch(prevTree, nextTree,
        <span class="hljs-comment">// 如果在 teleport 组件中父节点可能已经改变，所以容器直接找旧树 DOM 元素的父节点</span>
        hostParentNode(prevTree.el),
        <span class="hljs-comment">// 参考节点在 fragment 的情况可能改变，所以直接找旧树 DOM 元素的下一个节点</span>
        getNextHostNode(prevTree),
        instance,
        parentSuspense,
        isSVG)
      <span class="hljs-comment">// 缓存更新后的 DOM 节点</span>
      next.el = nextTree.el
    }
  }, prodEffectOptions)
}
</code></pre>

<p data-nodeid="9750">在更新组件的时候，会判断是否有 instance.next,它代表新的组件 vnode，根据前面的逻辑 next 不为空，所以会执行 updateComponentPreRender 更新组件 vnode 节点信息，我们来看一下它的实现：</p>
<pre class="lang-java" data-nodeid="29106"><code data-language="java"><span class="hljs-keyword">const</span> updateComponentPreRender = (instance, nextVNode, optimized) =&gt; {
  nextVNode.component = instance
  <span class="hljs-keyword">const</span> prevProps = instance.vnode.props
  instance.vnode = nextVNode
  instance.next = <span class="hljs-function"><span class="hljs-keyword">null</span>
  <span class="hljs-title">updateProps</span><span class="hljs-params">(instance, nextVNode.props, prevProps, optimized)</span>
  <span class="hljs-title">updateSlots</span><span class="hljs-params">(instance, nextVNode.children)</span>
}
</span></code></pre>

<p data-nodeid="9752">其中，会执行 updateProps 更新 props 数据，我们来看它的实现：</p>
<pre class="lang-java" data-nodeid="29583"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">updateProps</span><span class="hljs-params">(instance, rawProps, rawPrevProps, optimized)</span> </span>{
  <span class="hljs-keyword">const</span> { props, attrs, vnode: { patchFlag } } = instance
  <span class="hljs-keyword">const</span> rawCurrentProps = toRaw(props)
  <span class="hljs-keyword">const</span> [options] = normalizePropsOptions(instance.type)
  <span class="hljs-keyword">if</span> ((optimized || patchFlag &gt; <span class="hljs-number">0</span>) &amp;&amp; !(patchFlag &amp; <span class="hljs-number">16</span> <span class="hljs-comment">/* FULL_PROPS */</span>)) {
    <span class="hljs-keyword">if</span> (patchFlag &amp; <span class="hljs-number">8</span> <span class="hljs-comment">/* PROPS */</span>) {
      <span class="hljs-comment">// 只更新动态 props 节点</span>
      <span class="hljs-keyword">const</span> propsToUpdate = instance.vnode.<span class="hljs-function">dynamicProps
      <span class="hljs-title">for</span> <span class="hljs-params">(let i = <span class="hljs-number">0</span>; i &lt; propsToUpdate.length; i++)</span> </span>{
        <span class="hljs-keyword">const</span> key = propsToUpdate[i]
        <span class="hljs-keyword">const</span> value = rawProps[key]
        <span class="hljs-keyword">if</span> (options) {
          <span class="hljs-keyword">if</span> (hasOwn(attrs, key)) {
            attrs[key] = value
          }
          <span class="hljs-keyword">else</span> {
            <span class="hljs-keyword">const</span> camelizedKey = camelize(key)
            props[camelizedKey] = resolvePropValue(options, rawCurrentProps, camelizedKey, value)
          }
        }
        <span class="hljs-keyword">else</span> {
          attrs[key] = value
        }
      }
    }
  }
  <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// 全量 props 更新</span>
    setFullProps(instance, rawProps, props, attrs)
    <span class="hljs-comment">// 因为新的 props 是动态的，把那些不在新的 props 中但存在于旧的 props 中的值设置为 undefined</span>
    <span class="hljs-function">let kebabKey
    <span class="hljs-title">for</span> <span class="hljs-params">(<span class="hljs-keyword">const</span> key in rawCurrentProps)</span> </span>{
      <span class="hljs-keyword">if</span> (!rawProps ||
        (!hasOwn(rawProps, key) &amp;&amp;
          ((kebabKey = hyphenate(key)) === key || !hasOwn(rawProps, kebabKey)))) {
        <span class="hljs-keyword">if</span> (options) {
          <span class="hljs-keyword">if</span> (rawPrevProps &amp;&amp;
            (rawPrevProps[key] !== undefined ||
              rawPrevProps[kebabKey] !== undefined)) {
            props[key] = resolvePropValue(options, rawProps || EMPTY_OBJ, key, undefined)
          }
        }
        <span class="hljs-keyword">else</span> {
          delete props[key]
        }
      }
    }
  }
  <span class="hljs-keyword">if</span> ((process.env.NODE_ENV !== <span class="hljs-string">'production'</span>) &amp;&amp; rawProps) {
    validateProps(props, instance.type)
  }
}
</code></pre>

<p data-nodeid="9754">updateProps 主要的目标就是把父组件渲染时求得的 props 新值，更新到子组件实例的 instance.props 中。</p>
<p data-nodeid="9755">在编译阶段，我们除了捕获一些动态 vnode，也捕获了动态的 props，所以我们可以只去比对动态的 props 数据更新。</p>
<p data-nodeid="9756">当然，如果不满足优化的条件，我们也可以通过 setFullProps 去全量比对更新 props，并且，由于新的 props 可能是动态的，因此会把那些不在新 props 中但存在于旧 props 中的值设置为 undefined。</p>
<p data-nodeid="9757">好了，至此我们搞明白了子组件实例的 props 值是如何更新的，那么我们现在来思考一下前面的一个问题，为什么 instance.props 需要变成响应式呢？其实这是一种需求，因为我们也希望在子组件中可以监听 props 值的变化做一些事情，举个例子：</p>
<pre class="lang-java" data-nodeid="30060"><code data-language="java"><span class="hljs-keyword">import</span> { ref, h, defineComponent, watchEffect } from <span class="hljs-string">'vue'</span>
<span class="hljs-keyword">const</span> count = ref(<span class="hljs-number">0</span>)
let dummy
<span class="hljs-keyword">const</span> Parent = {
  render: () =&gt; h(Child, { count: count.value })
}
<span class="hljs-keyword">const</span> Child = defineComponent({
  props: { count: Number },
  setup(props) {
    watchEffect(() =&gt; {
      dummy = props.count
    })
    <span class="hljs-keyword">return</span> () =&gt; h(<span class="hljs-string">'div'</span>, props.count)
  }
})
count.value++
</code></pre>

<p data-nodeid="30537">这里，我们定义了父组件 Parent 和子组件 Child，子组件 Child 中定义了 prop count，除了在渲染模板中引用了 count，我们在 setup 函数中通过了 watchEffect 注册了一个回调函数，内部依赖了 props.count，当修改 count.value 的时候，我们希望这个回调函数也能执行，所以这个 prop 的值需要是响应式的，由于 setup 函数的第一个参数是props 变量，其实就是组件实例 instance.props，所以也就是要求 instance.props 是响应式的。</p>
<p data-nodeid="30538">我们再来看为什么用 shallowReactive API 呢？shallow 的字面意思是浅的，从实现上来说，就是不会递归执行 reactive，只劫持最外一层对象。</p>

<p data-nodeid="9760">shallowReactive 和普通的 reactive 函数的主要区别是处理器函数不同，我们来回顾 getter 的处理器函数：</p>
<pre class="lang-java" data-nodeid="31017"><code data-language="java"><span class="hljs-function">function <span class="hljs-title">createGetter</span><span class="hljs-params">(isReadonly = <span class="hljs-keyword">false</span>, shallow = <span class="hljs-keyword">false</span>)</span> </span>{
  <span class="hljs-keyword">return</span> <span class="hljs-function">function <span class="hljs-title">get</span><span class="hljs-params">(target, key, receiver)</span> </span>{
    <span class="hljs-keyword">if</span> (key === <span class="hljs-string">"__v_isReactive"</span> <span class="hljs-comment">/* IS_REACTIVE */</span>) {
      <span class="hljs-keyword">return</span> !isReadonly;
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (key === <span class="hljs-string">"__v_isReadonly"</span> <span class="hljs-comment">/* IS_READONLY */</span>) {
      <span class="hljs-keyword">return</span> isReadonly;
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (key === <span class="hljs-string">"__v_raw"</span> <span class="hljs-comment">/* RAW */</span> &amp;&amp;
      receiver ===
      (isReadonly
        ? target[<span class="hljs-string">"__v_readonly"</span> <span class="hljs-comment">/* READONLY */</span>]
        : target[<span class="hljs-string">"__v_reactive"</span> <span class="hljs-comment">/* REACTIVE */</span>])) {
      <span class="hljs-keyword">return</span> target;
    }
    <span class="hljs-keyword">const</span> targetIsArray = isArray(target);
    <span class="hljs-keyword">if</span> (targetIsArray &amp;&amp; hasOwn(arrayInstrumentations, key)) {
      <span class="hljs-keyword">return</span> Reflect.get(arrayInstrumentations, key, receiver);
    }
    <span class="hljs-keyword">const</span> res = Reflect.get(target, key, receiver);
    <span class="hljs-keyword">if</span> (isSymbol(key)
      ? builtInSymbols.has(key)
      : key === `__proto__` || key === `__v_isRef`) {
      <span class="hljs-keyword">return</span> res;
    }
    <span class="hljs-keyword">if</span> (!isReadonly) {
      track(target, <span class="hljs-string">"get"</span> <span class="hljs-comment">/* GET */</span>, key);
    }
    <span class="hljs-keyword">if</span> (shallow) {
      <span class="hljs-keyword">return</span> res;
    }
    <span class="hljs-keyword">if</span> (isRef(res)) {
      <span class="hljs-keyword">return</span> targetIsArray ? res : res.value;
    }
    <span class="hljs-keyword">if</span> (isObject(res)) {
      <span class="hljs-keyword">return</span> isReadonly ? readonly(res) : reactive(res);
    }
    <span class="hljs-keyword">return</span> res;
  };
}
</code></pre>

<p data-nodeid="9762">shallowReactive 创建的 getter 函数，shallow 变量为 true，那么就不会执行后续的递归 reactive 逻辑。也就是说，shallowReactive 只把对象 target 的最外一层属性的访问和修改处理成响应式。</p>
<p data-nodeid="9763">之所以可以这么做，是因为 props 在更新的过程中，只会修改最外层属性，所以用 shallowReactive 就足够了。</p>
<h3 data-nodeid="9764">总结</h3>
<p data-nodeid="9765">好的，到这里我们这一节的学习也要结束啦，通过这节课的学习，你应该要了解 Props 是如何被初始化的，如何被校验的，你需要区分开 Props 配置和 Props 传值这两个概念；你还应该了解 Props 是如何更新的以及实例上的 props 为什么要定义成响应式的。</p>
<p data-nodeid="9766">最后，给你留一道思考题目，我们把前面的示例稍加修改，HelloWorld 子组件如下：</p>
<pre class="lang-js" data-nodeid="31971"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{{ msg }}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{{ info.name }}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>{{ info.age }}<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">props</span>: {
      <span class="hljs-attr">msg</span>: <span class="hljs-built_in">String</span>,
      <span class="hljs-attr">info</span>: <span class="hljs-built_in">Object</span>
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>


<p data-nodeid="9768">我们添加了 info prop，然后在模板中渲染了 info 的子属性数据，然后我们再修改一下父组件：</p>
<pre class="lang-js" data-nodeid="32925"><code data-language="js">&lt;template&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">hello-world</span> <span class="hljs-attr">:msg</span>=<span class="hljs-string">"msg"</span> <span class="hljs-attr">:info</span>=<span class="hljs-string">"info"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">hello-world</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"addAge"</span>&gt;</span>Add age<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">button</span> @<span class="hljs-attr">click</span>=<span class="hljs-string">"toggleMsg"</span>&gt;</span>Toggle Msg<span class="hljs-tag">&lt;/<span class="hljs-name">button</span>&gt;</span></span>
&lt;/template&gt;
<span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">import</span> HelloWorld <span class="hljs-keyword">from</span> <span class="hljs-string">'./components/HelloWorld'</span>
  <span class="hljs-keyword">export</span> <span class="hljs-keyword">default</span> {
    <span class="hljs-attr">components</span>: { HelloWorld },
    data() {
      <span class="hljs-keyword">return</span> {
        <span class="hljs-attr">info</span>: {
          <span class="hljs-attr">name</span>: <span class="hljs-string">'Tom'</span>,
          <span class="hljs-attr">age</span>: <span class="hljs-number">18</span>
        },
        <span class="hljs-attr">msg</span>: <span class="hljs-string">'Hello world'</span>
      }
    },
    <span class="hljs-attr">methods</span>: {
      addAge() {
        <span class="hljs-built_in">this</span>.info.age++
      },
      toggleMsg() {
        <span class="hljs-built_in">this</span>.msg = <span class="hljs-built_in">this</span>.msg === <span class="hljs-string">'Hello world'</span> ? <span class="hljs-string">'Hello Vue'</span> : <span class="hljs-string">'Hello world'</span>
      }
    }
  }
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
</code></pre>


<p data-nodeid="9770">我们在 data 中添加了 info 变量，然后当我们点击 Add age 按钮去修改 this.info.age 的时候，触发了子组件 props 的变化了吗？子组件为什么会重新渲染呢？欢迎你在留言区与我分享。</p>
<blockquote data-nodeid="9771">
<p data-nodeid="9772">本节课的相关代码在源代码中的位置如下：<br>
packages/runtime-core/src/componentProps.ts<br>
packages/reactivity/src/reactive.ts<br>
packages/reactivity/src/baseHandlers.ts</p>
</blockquote>

---

### 精选评论

##### ssh：
> props 没变，但是由于子组件访问了info.name，所以走的是子组件自己的响应式依赖 + 触发渲染的逻辑。

##### **军：
> 我理解,在初始化props时,会获取父组件传递过来的数据,在父组件中该数据是响应式数据,所以获取数据值的过程中,会收集父组件的effect副作用函数,当改变this.info.age, 会使父组件重新执行effect渲染,子组件就会重新渲染, 重新渲染就会更新props的值(因为在instance.update函数中会调用更新props的方法)

##### *策：
> 我的理解：没有触发子组件的prop的变化，指向了同一个prop原因：因为 这个 prop 本身在父组件就已经被 Proxy 代理了，在子组件中被渲染的时候，子组件对应的依赖也被同时收集，所以父组件的 info.age 在修改之后，处理收集的依赖，顺便把子组件也给算进去了不知道对不对

