<p data-nodeid="1252" class="">下面我们来继续讲解上节课提到的<strong data-nodeid="1405">核心 diff 算法</strong>。</p>
<p data-nodeid="1253">新子节点数组相对于旧子节点数组的变化，无非是通过更新、删除、添加和移动节点来完成，而核心 diff 算法，就是在已知旧子节点的 DOM 结构、vnode 和新子节点的 vnode 情况下，以较低的成本完成子节点的更新为目的，求解生成新子节点 DOM 的系列操作。</p>
<p data-nodeid="1254">为了方便你理解，我先举个例子，假设有这样一个列表：</p>
<pre class="lang-js" data-nodeid="1255"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"c"</span>&gt;</span>c<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1256">然后我们在中间插入一行，得到一个新列表：</p>
<pre class="lang-js" data-nodeid="1257"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"e"</span>&gt;</span>e<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"c"</span>&gt;</span>c<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1258">在插入操作的前后，它们对应渲染生成的 vnode 可以用一张图表示：</p>
<p data-nodeid="1259"><img src="https://s0.lgstatic.com/i/image/M00/33/86/CgqCHl8QHwmAHuQrAAB7807ZTzY864.png" alt="111.png" data-nodeid="1412"></p>
<p data-nodeid="1260">从图中我们可以直观地感受到，差异主要在新子节点中的 b 节点后面多了一个 e 节点。</p>
<p data-nodeid="1261">我们再把这个例子稍微修改一下，多添加一个 e 节点：</p>
<pre class="lang-js" data-nodeid="1262"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"c"</span>&gt;</span>c<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"e"</span>&gt;</span>e<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1263">然后我们删除中间一项，得到一个新列表：</p>
<pre class="lang-js" data-nodeid="1264"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"e"</span>&gt;</span>e<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1265">在删除操作的前后，它们对应渲染生成的 vnode 可以用一张图表示：</p>
<p data-nodeid="1266"><img src="https://s0.lgstatic.com/i/image/M00/32/C3/Ciqc1F8OxNCAbTueAABtqP8l5JI050.png" alt="图片2.png" data-nodeid="1419"></p>
<p data-nodeid="1267">我们可以看到，这时差异主要在新子节点中的 b 节点后面少了一个 c 节点。</p>
<p data-nodeid="1268">综合这两个例子，我们很容易发现新旧 children 拥有相同的头尾节点。对于相同的节点，我们只需要做对比更新即可，所以 diff 算法的第一步<strong data-nodeid="1426">从头部开始同步</strong>。</p>
<h3 data-nodeid="1269">同步头部节点</h3>
<p data-nodeid="1270">我们先来看一下头部节点同步的实现代码：</p>
<pre class="lang-js" data-nodeid="1271"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 3, e2 = 4</span>
  <span class="hljs-comment">// (a b) c d</span>
  <span class="hljs-comment">// (a b) e c d</span>
  <span class="hljs-keyword">while</span> (i &lt;= e1 &amp;&amp; i &lt;= e2) {
    <span class="hljs-keyword">const</span> n1 = c1[i]
    <span class="hljs-keyword">const</span> n2 = c2[i]
    <span class="hljs-keyword">if</span> (isSameVNodeType(n1, n2)) {
      <span class="hljs-comment">// 相同的节点，递归执行 patch 更新节点</span>
      patch(n1, n2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized)
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">break</span>
    }
    i++
  }
}
</code></pre>
<p data-nodeid="1272">在整个 diff 的过程，我们需要维护几个变量：头部的索引 i、旧子节点的尾部索引 e1和新子节点的尾部索引 e2。</p>
<p data-nodeid="1273">同步头部节点就是从头部开始，依次对比新节点和旧节点，如果它们相同的则执行 patch 更新节点；如果不同或者索引 i 大于索引 e1 或者 e2，则同步过程结束。</p>
<p data-nodeid="1274">我们拿第一个例子来说，通过下图看一下同步头部节点后的结果：</p>
<p data-nodeid="1275"><img src="https://s0.lgstatic.com/i/image/M00/32/C3/Ciqc1F8OxN6AMzbfAACPna55Fmk255.png" alt="图片3.png" data-nodeid="1434"></p>
<p data-nodeid="1276">可以看到，完成头部节点同步后：i 是 2，e1 是 3，e2 是 4。</p>
<h3 data-nodeid="1277">同步尾部节点</h3>
<p data-nodeid="1278">接着从尾部开始<strong data-nodeid="1442">同步尾部节点</strong>，实现代码如下：</p>
<pre class="lang-js" data-nodeid="1279"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 3, e2 = 4</span>
  <span class="hljs-comment">// (a b) c d</span>
  <span class="hljs-comment">// (a b) e c d</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 3, e2 = 4</span>
  <span class="hljs-comment">// (a b) (c d)</span>
  <span class="hljs-comment">// (a b) e (c d)</span>
  <span class="hljs-keyword">while</span> (i &lt;= e1 &amp;&amp; i &lt;= e2) {
    <span class="hljs-keyword">const</span> n1 = c1[e1]
    <span class="hljs-keyword">const</span> n2 = c2[e2]
    <span class="hljs-keyword">if</span> (isSameVNodeType(n1, n2)) {
      patch(n1, n2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized)
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-keyword">break</span>
    }
    e1--
    e2--
  }
}
</code></pre>
<p data-nodeid="1280">同步尾部节点就是从尾部开始，依次对比新节点和旧节点，如果相同的则执行 patch 更新节点；如果不同或者索引 i 大于索引 e1 或者 e2，则同步过程结束。</p>
<p data-nodeid="1281">我们来通过下图看一下同步尾部节点后的结果：</p>
<p data-nodeid="1282"><img src="https://s0.lgstatic.com/i/image/M00/32/C3/Ciqc1F8OxO2AffFhAACJ52ATnwQ480.png" alt="图片4.png" data-nodeid="1447"></p>
<p data-nodeid="1283">可以看到，完成尾部节点同步后：i 是 2，e1 是 1，e2 是 2。</p>
<p data-nodeid="1284">接下来只有 3 种情况要处理：</p>
<ul data-nodeid="1285">
<li data-nodeid="1286">
<p data-nodeid="1287">新子节点有剩余要添加的新节点；</p>
</li>
<li data-nodeid="1288">
<p data-nodeid="1289">旧子节点有剩余要删除的多余节点；</p>
</li>
<li data-nodeid="1290">
<p data-nodeid="1291">未知子序列。</p>
</li>
</ul>
<p data-nodeid="1292">我们继续看一下具体是怎样操作的。</p>
<h3 data-nodeid="1293">添加新的节点</h3>
<p data-nodeid="1294">首先要判断新子节点是否有剩余的情况，如果满足则添加新子节点，实现代码如下：</p>
<pre class="lang-js" data-nodeid="1295"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 3, e2 = 4</span>
  <span class="hljs-comment">// (a b) c d</span>
  <span class="hljs-comment">// (a b) e c d</span>
  <span class="hljs-comment">// ...</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 3, e2 = 4</span>
  <span class="hljs-comment">// (a b) (c d)</span>
  <span class="hljs-comment">// (a b) e (c d)</span>
  <span class="hljs-comment">// 3. 挂载剩余的新节点</span>
  <span class="hljs-comment">// i = 2, e1 = 1, e2 = 2</span>
  <span class="hljs-keyword">if</span> (i &gt; e1) {
    <span class="hljs-keyword">if</span> (i &lt;= e2) {
      <span class="hljs-keyword">const</span> nextPos = e2 + <span class="hljs-number">1</span>
      <span class="hljs-keyword">const</span> anchor = nextPos &lt; l2 ? c2[nextPos].el : parentAnchor
      <span class="hljs-keyword">while</span> (i &lt;= e2) {
        <span class="hljs-comment">// 挂载新节点</span>
        patch(<span class="hljs-literal">null</span>, c2[i], container, anchor, parentComponent, parentSuspense, isSVG)
        i++
      }
    }
  }
}
</code></pre>
<p data-nodeid="1296">如果索引 i 大于尾部索引 e1 且 i 小于 e2，那么从索引 i 开始到索引 e2 之间，我们直接挂载新子树这部分的节点。</p>
<p data-nodeid="1297">对我们的例子而言，同步完尾部节点后 i 是 2，e1 是 1，e2 是 2，此时满足条件需要添加新的节点，我们来通过下图看一下添加后的结果：</p>
<p data-nodeid="1298"><img src="https://s0.lgstatic.com/i/image/M00/32/CF/CgqCHl8OxQKAd7fjAACNTHXEkuQ335.png" alt="图片5.png" data-nodeid="1460"></p>
<p data-nodeid="1299">添加完 e 节点后，旧子节点的 DOM 和新子节点对应的 vnode 映射一致，也就完成了更新。</p>
<h3 data-nodeid="1300">删除多余节点</h3>
<p data-nodeid="1301">如果不满足添加新节点的情况，我就要接着判断旧子节点是否有剩余，如果满足则删除旧子节点，实现代码如下：</p>
<pre class="lang-js" data-nodeid="1302"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 4, e2 = 3</span>
  <span class="hljs-comment">// (a b) c d e</span>
  <span class="hljs-comment">// (a b) d e</span>
  <span class="hljs-comment">// ...</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 4, e2 = 3</span>
  <span class="hljs-comment">// (a b) c (d e)</span>
  <span class="hljs-comment">// (a b) (d e)</span>
  <span class="hljs-comment">// 3. 普通序列挂载剩余的新节点</span>
  <span class="hljs-comment">// i = 2, e1 = 2, e2 = 1</span>
  <span class="hljs-comment">// 不满足</span>
  <span class="hljs-keyword">if</span> (i &gt; e1) {
  }
  <span class="hljs-comment">// 4. 普通序列删除多余的旧节点</span>
  <span class="hljs-comment">// i = 2, e1 = 2, e2 = 1</span>
  <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (i &gt; e2) {
    <span class="hljs-keyword">while</span> (i &lt;= e1) {
      <span class="hljs-comment">// 删除节点</span>
      unmount(c1[i], parentComponent, parentSuspense, <span class="hljs-literal">true</span>)
      i++
    }
  }
}
</code></pre>
<p data-nodeid="1303">如果索引 i 大于尾部索引 e2，那么从索引 i 开始到索引 e1 之间，我们直接删除旧子树这部分的节点。</p>
<p data-nodeid="1304">第二个例子是就删除节点的情况，我们从同步头部节点开始，用图的方式演示这一过程。</p>
<p data-nodeid="1305">首先从头部同步节点：</p>
<p data-nodeid="1306"><img src="https://s0.lgstatic.com/i/image/M00/32/C4/Ciqc1F8OxQ-ADmRcAACCSIpni8Y429.png" alt="图片6.png" data-nodeid="1469"></p>
<p data-nodeid="1307">此时的结果：i 是 2，e1 是 4，e2 是 3。</p>
<p data-nodeid="1308">接着从尾部同步节点：</p>
<p data-nodeid="1309"><img src="https://s0.lgstatic.com/i/image/M00/32/C4/Ciqc1F8OxRqANXzyAACGFb9dacI061.png" alt="图片7.png" data-nodeid="1474"></p>
<p data-nodeid="1310">此时的结果：i 是 2，e1 是 2，e2 是 1，满足删除条件，因此删除子节点中的多余节点：</p>
<p data-nodeid="1311"><img src="https://s0.lgstatic.com/i/image/M00/32/CF/CgqCHl8OxSeAMW8gAACCvYcKESo055.png" alt="图片8.png" data-nodeid="1478"></p>
<p data-nodeid="1312">删除完 c 节点后，旧子节点的 DOM 和新子节点对应的 vnode 映射一致，也就完成了更新。</p>
<h3 data-nodeid="1313">处理未知子序列</h3>
<p data-nodeid="1314">单纯的添加和删除节点都是比较理想的情况，操作起来也很容易，但是有些时候并非这么幸运，我们会遇到比较复杂的未知子序列，这时候 diff 算法会怎么做呢？</p>
<p data-nodeid="1315">我们再通过例子来演示存在未知子序列的情况，假设一个按照字母表排列的列表：</p>
<pre class="lang-js" data-nodeid="1316"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"c"</span>&gt;</span>c<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"e"</span>&gt;</span>e<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"f"</span>&gt;</span>f<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"g"</span>&gt;</span>g<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"h"</span>&gt;</span>h<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1317">然后我们打乱之前的顺序得到一个新列表：</p>
<pre class="lang-js" data-nodeid="1318"><code data-language="js">&lt;ul&gt;
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"a"</span>&gt;</span>a<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"b"</span>&gt;</span>b<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"e"</span>&gt;</span>e<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"d"</span>&gt;</span>c<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"c"</span>&gt;</span>d<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"i"</span>&gt;</span>i<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"g"</span>&gt;</span>g<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
  <span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">key</span>=<span class="hljs-string">"h"</span>&gt;</span>h<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
&lt;/ul&gt;
</code></pre>
<p data-nodeid="1319">在操作前，它们对应渲染生成的 vnode 可以用一张图表示：</p>
<p data-nodeid="1320"><img src="https://s0.lgstatic.com/i/image/M00/32/C4/Ciqc1F8OxT6AVycJAAClkNghf-k681.png" alt="图片9.png" data-nodeid="1487"></p>
<p data-nodeid="1321">我们还是从同步头部节点开始，用图的方式演示这一过程。</p>
<p data-nodeid="1322">首先从头部同步节点：</p>
<p data-nodeid="1323"><img src="https://s0.lgstatic.com/i/image/M00/32/CF/CgqCHl8OxUyAaCXvAAC6Lv79hSs090.png" alt="图片10.png" data-nodeid="1492"></p>
<p data-nodeid="1324">同步头部节点后的结果：i 是 2，e1 是 7，e2 是 7。</p>
<p data-nodeid="1325">接着从尾部同步节点：</p>
<p data-nodeid="1326"><img src="https://s0.lgstatic.com/i/image/M00/32/C4/Ciqc1F8OxVeAYV_ZAADCIt6XIHI609.png" alt="图片11.png" data-nodeid="1497"></p>
<p data-nodeid="1327">同步尾部节点后的结果：i 是 2，e1 是 5，e2 是 5。可以看到它既不满足添加新节点的条件，也不满足删除旧节点的条件。那么对于这种情况，我们应该怎么处理呢？</p>
<p data-nodeid="1328">结合上图可以知道，要把旧子节点的 c、d、e、f 转变成新子节点的 e、c、d、i。从直观上看，我们把 e 节点移动到 c 节点前面，删除 f 节点，然后在 d 节点后面添加 i 节点即可。</p>
<p data-nodeid="1329">其实无论多复杂的情况，最终无非都是通过更新、删除、添加、移动这些动作来操作节点，而我们要做的就是找到相对优的解。</p>
<p data-nodeid="1330">当两个节点类型相同时，我们执行更新操作；当新子节点中没有旧子节点中的某些节点时，我们执行删除操作；当新子节点中多了旧子节点中没有的节点时，我们执行添加操作，这些操作我们在前面已经阐述清楚了。相对来说这些操作中最麻烦的就是移动，我们既要判断哪些节点需要移动也要清楚如何移动。</p>
<h4 data-nodeid="1331">移动子节点</h4>
<p data-nodeid="1332">那么什么时候需要移动呢，就是当子节点排列顺序发生变化的时候，举个简单的例子具体看一下：</p>
<pre class="lang-js" data-nodeid="1333"><code data-language="js"><span class="hljs-keyword">var</span> prev = [<span class="hljs-number">1</span>, <span class="hljs-number">2</span>, <span class="hljs-number">3</span>, <span class="hljs-number">4</span>, <span class="hljs-number">5</span>, <span class="hljs-number">6</span>]
<span class="hljs-keyword">var</span> next = [<span class="hljs-number">1</span>, <span class="hljs-number">3</span>, <span class="hljs-number">2</span>, <span class="hljs-number">6</span>, <span class="hljs-number">4</span>, <span class="hljs-number">5</span>]
</code></pre>
<p data-nodeid="1334">可以看到，从 prev 变成 next，数组里的一些元素的顺序发生了变化，我们可以把子节点类比为元素，现在问题就简化为我们如何用最少的移动使元素顺序从 prev 变化为 next 。</p>
<p data-nodeid="1335">一种思路是在 next 中找到一个递增子序列，比如 [1, 3, 6] 、[1, 2, 4, 5]。之后对 next 数组进行倒序遍历，移动所有不在递增序列中的元素即可。</p>
<p data-nodeid="1336">如果选择了 [1, 3, 6] 作为递增子序列，那么在倒序遍历的过程中，遇到 6、3、1 不动，遇到 5、4、2 移动即可，如下图所示：</p>
<p data-nodeid="1337"><img src="https://s0.lgstatic.com/i/image/M00/32/CF/CgqCHl8OxWOAKRnGAAAzjDtkQJI201.png" alt="图片12.png" data-nodeid="1521"></p>
<p data-nodeid="1338">如果选择了 [1, 2, 4, 5] 作为递增子序列，那么在倒序遍历的过程中，遇到 5、4、2、1 不动，遇到 6、3 移动即可，如下图所示：</p>
<p data-nodeid="1339"><img src="https://s0.lgstatic.com/i/image/M00/32/CF/CgqCHl8OxW6APB5gAAAshOjdgMY518.png" alt="图片13.png" data-nodeid="1529"></p>
<p data-nodeid="1340">可以看到第一种移动了三次，而第二种只移动了两次，递增子序列越长，所需要移动元素的次数越少，所以如何移动的问题就回到了求解最长递增子序列的问题。我们稍后会详细讲求解最长递增子序列的算法，所以先回到我们这里的问题，对未知子序列的处理。</p>
<p data-nodeid="1341">我们现在要做的是在新旧子节点序列中找出相同节点并更新，找出多余的节点删除，找出新的节点添加，找出是否有需要移动的节点，如果有该如何移动。</p>
<p data-nodeid="1342">在查找过程中需要对比新旧子序列，那么我们就要遍历某个序列，如果在遍历旧子序列的过程中需要判断某个节点是否在新子序列中存在，这就需要双重循环，而双重循环的复杂度是 O(n<sup>2</sup>) ，为了优化这个复杂度，我们可以用一种空间换时间的思路，建立索引图，把时间复杂度降低到 O(n)。</p>
<h4 data-nodeid="1343">建立索引图</h4>
<p data-nodeid="1344">所以处理未知子序列的第一步，就是建立索引图。</p>
<p data-nodeid="1345">通常我们在开发过程中， 会给 v-for 生成的列表中的每一项分配唯一 key 作为项的唯一 ID，这个 key 在 diff 过程中起到很关键的作用。对于新旧子序列中的节点，我们认为 key 相同的就是同一个节点，直接执行 patch 更新即可。</p>
<p data-nodeid="1346">我们根据 key 建立新子序列的索引图，实现如下：</p>
<pre class="lang-js" data-nodeid="1347"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 7, e2 = 7</span>
  <span class="hljs-comment">// (a b) c d e f g h</span>
  <span class="hljs-comment">// (a b) e c d i g h</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 7, e2 = 7</span>
  <span class="hljs-comment">// (a b) c d e f (g h)</span>
  <span class="hljs-comment">// (a b) e c d i (g h)</span>
  <span class="hljs-comment">// 3. 普通序列挂载剩余的新节点， 不满足</span>
  <span class="hljs-comment">// 4. 普通序列删除多余的旧节点，不满足</span>
  <span class="hljs-comment">// i = 2, e1 = 4, e2 = 5</span>
  <span class="hljs-comment">// 旧子序列开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s1 = i
  <span class="hljs-comment">// 新子序列开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s2 = i <span class="hljs-comment">//</span>
  <span class="hljs-comment">// 5.1 根据 key 建立新子序列的索引图</span>
  <span class="hljs-keyword">const</span> keyToNewIndexMap = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Map</span>()
  <span class="hljs-keyword">for</span> (i = s2; i &lt;= e2; i++) {
    <span class="hljs-keyword">const</span> nextChild = c2[i]
    keyToNewIndexMap.set(nextChild.key, i)
  }
}
</code></pre>
<p data-nodeid="1348">新旧子序列是从 i 开始的，所以我们先用 s1、s2 分别作为新旧子序列的开始索引，接着建立一个 keyToNewIndexMap 的 Map&lt;key, index&gt; 结构，遍历新子序列，把节点的 key 和 index 添加到这个 Map 中，注意我们这里假设所有节点都是有 key 标识的。</p>
<p data-nodeid="1349">keyToNewIndexMap 存储的就是新子序列中每个节点在新子序列中的索引，我们来看一下示例处理后的结果，如下图所示：</p>
<p data-nodeid="1350"><img src="https://s0.lgstatic.com/i/image/M00/32/D0/CgqCHl8OxciAQJ6GAADhf7zD47s944.png" alt="图片14.png" data-nodeid="1547"></p>
<p data-nodeid="1351">我们得到了一个值为 {e:2,c:3,d:4,i:5} 的新子序列索引图。</p>
<h4 data-nodeid="1352">更新和移除旧节点</h4>
<p data-nodeid="1353">接下来，我们就需要遍历旧子序列，有相同的节点就通过 patch 更新，并且移除那些不在新子序列中的节点，同时找出是否有需要移动的节点，我们来看一下这部分逻辑的实现：</p>
<pre class="lang-js" data-nodeid="1354"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 7, e2 = 7</span>
  <span class="hljs-comment">// (a b) c d e f g h</span>
  <span class="hljs-comment">// (a b) e c d i g h</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 7, e2 = 7</span>
  <span class="hljs-comment">// (a b) c d e f (g h)</span>
  <span class="hljs-comment">// (a b) e c d i (g h)</span>
  <span class="hljs-comment">// 3. 普通序列挂载剩余的新节点，不满足</span>
  <span class="hljs-comment">// 4. 普通序列删除多余的旧节点，不满足</span>
  <span class="hljs-comment">// i = 2, e1 = 4, e2 = 5</span>
  <span class="hljs-comment">// 旧子序列开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s1 = i
  <span class="hljs-comment">// 新子序列开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s2 = i
  <span class="hljs-comment">// 5.1 根据 key 建立新子序列的索引图</span>
  <span class="hljs-comment">// 5.2 正序遍历旧子序列，找到匹配的节点更新，删除不在新子序列中的节点，判断是否有移动节点</span>
  <span class="hljs-comment">// 新子序列已更新节点的数量</span>
  <span class="hljs-keyword">let</span> patched = <span class="hljs-number">0</span>
  <span class="hljs-comment">// 新子序列待更新节点的数量，等于新子序列的长度</span>
  <span class="hljs-keyword">const</span> toBePatched = e2 - s2 + <span class="hljs-number">1</span>
  <span class="hljs-comment">// 是否存在要移动的节点</span>
  <span class="hljs-keyword">let</span> moved = <span class="hljs-literal">false</span>
  <span class="hljs-comment">// 用于跟踪判断是否有节点移动</span>
  <span class="hljs-keyword">let</span> maxNewIndexSoFar = <span class="hljs-number">0</span>
  <span class="hljs-comment">// 这个数组存储新子序列中的元素在旧子序列节点的索引，用于确定最长递增子序列</span>
  <span class="hljs-keyword">const</span> newIndexToOldIndexMap = <span class="hljs-keyword">new</span> <span class="hljs-built_in">Array</span>(toBePatched)
  <span class="hljs-comment">// 初始化数组，每个元素的值都是 0</span>
  <span class="hljs-comment">// 0 是一个特殊的值，如果遍历完了仍有元素的值为 0，则说明这个新节点没有对应的旧节点</span>
  <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; toBePatched; i++)
    newIndexToOldIndexMap[i] = <span class="hljs-number">0</span>
  <span class="hljs-comment">// 正序遍历旧子序列</span>
  <span class="hljs-keyword">for</span> (i = s1; i &lt;= e1; i++) {
    <span class="hljs-comment">// 拿到每一个旧子序列节点</span>
    <span class="hljs-keyword">const</span> prevChild = c1[i]
    <span class="hljs-keyword">if</span> (patched &gt;= toBePatched) {
      <span class="hljs-comment">// 所有新的子序列节点都已经更新，剩余的节点删除</span>
      unmount(prevChild, parentComponent, parentSuspense, <span class="hljs-literal">true</span>)
      <span class="hljs-keyword">continue</span>
    }
    <span class="hljs-comment">// 查找旧子序列中的节点在新子序列中的索引</span>
    <span class="hljs-keyword">let</span> newIndex = keyToNewIndexMap.get(prevChild.key)
    <span class="hljs-keyword">if</span> (newIndex === <span class="hljs-literal">undefined</span>) {
      <span class="hljs-comment">// 找不到说明旧子序列已经不存在于新子序列中，则删除该节点</span>
      unmount(prevChild, parentComponent, parentSuspense, <span class="hljs-literal">true</span>)
    }
    <span class="hljs-keyword">else</span> {
      <span class="hljs-comment">// 更新新子序列中的元素在旧子序列中的索引，这里加 1 偏移，是为了避免 i 为 0 的特殊情况，影响对后续最长递增子序列的求解</span>
      newIndexToOldIndexMap[newIndex - s2] = i + <span class="hljs-number">1</span>
      <span class="hljs-comment">// maxNewIndexSoFar 始终存储的是上次求值的 newIndex，如果不是一直递增，则说明有移动</span>
      <span class="hljs-keyword">if</span> (newIndex &gt;= maxNewIndexSoFar) {
        maxNewIndexSoFar = newIndex
      }
      <span class="hljs-keyword">else</span> {
        moved = <span class="hljs-literal">true</span>
      }
      <span class="hljs-comment">// 更新新旧子序列中匹配的节点</span>
      patch(prevChild, c2[newIndex], container, <span class="hljs-literal">null</span>, parentComponent, parentSuspense, isSVG, optimized)
      patched++
    }
  }
}
</code></pre>
<p data-nodeid="1355">我们建立了一个 newIndexToOldIndexMap 的数组，来存储新子序列节点的索引和旧子序列节点的索引之间的映射关系，用于确定最长递增子序列，这个数组的长度为新子序列的长度，每个元素的初始值设为 0， 它是一个特殊的值，如果遍历完了仍有元素的值为 0，则说明遍历旧子序列的过程中没有处理过这个节点，这个节点是新添加的。</p>
<p data-nodeid="1356">下面我们说说具体的操作过程：正序遍历旧子序列，根据前面建立的 keyToNewIndexMap 查找旧子序列中的节点在新子序列中的索引，如果找不到就说明新子序列中没有该节点，就删除它；如果找得到则将它在旧子序列中的索引更新到 newIndexToOldIndexMap 中。</p>
<p data-nodeid="1357">注意这里索引加了长度为 1 的偏移，是为了应对 i 为 0 的特殊情况，如果不这样处理就会影响后续求解最长递增子序列。</p>
<p data-nodeid="2505" class="">遍历过程中，我们用变量 maxNewIndexSoFar 跟踪判断节点是否移动，maxNewIndexSoFar 始终存储的是上次求值的 newIndex，一旦本次求值的 newIndex 小于 maxNewIndexSoFar，这说明顺序遍历旧子序列的节点在新子序列中的索引并不是一直递增的，也就说明存在移动的情况。</p>


<p data-nodeid="1359">除此之外，这个过程中我们也会更新新旧子序列中匹配的节点，另外如果所有新的子序列节点都已经更新，而对旧子序列遍历还未结束，说明剩余的节点就是多余的，删除即可。</p>
<p data-nodeid="1360">至此，我们完成了新旧子序列节点的更新、多余旧节点的删除，并且建立了一个 newIndexToOldIndexMap 存储新子序列节点的索引和旧子序列节点的索引之间的映射关系，并确定是否有移动。</p>
<p data-nodeid="1361">我们来看一下示例处理后的结果，如下图所示：</p>
<p data-nodeid="1362"><img src="https://s0.lgstatic.com/i/image/M00/32/D0/CgqCHl8OxdeAVdPEAAEh9JAOZ_E654.png" alt="图片15.png" data-nodeid="1560"></p>
<p data-nodeid="1363">可以看到， c、d、e 节点被更新，f 节点被删除，newIndexToOldIndexMap 的值为 [5, 3, 4 ,0]，此时 moved 也为 true，也就是存在节点移动的情况。</p>
<h4 data-nodeid="1364">移动和挂载新节点</h4>
<p data-nodeid="1365">接下来，就到了处理未知子序列的最后一个流程，移动和挂载新节点，我们来看一下这部分逻辑的实现：</p>
<pre class="lang-js" data-nodeid="1366"><code data-language="js"><span class="hljs-keyword">const</span> patchKeyedChildren = <span class="hljs-function">(<span class="hljs-params">c1, c2, container, parentAnchor, parentComponent, parentSuspense, isSVG, optimized</span>) =&gt;</span> {
  <span class="hljs-keyword">let</span> i = <span class="hljs-number">0</span>
  <span class="hljs-keyword">const</span> l2 = c2.length
  <span class="hljs-comment">// 旧子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e1 = c1.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 新子节点的尾部索引</span>
  <span class="hljs-keyword">let</span> e2 = l2 - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 1. 从头部开始同步</span>
  <span class="hljs-comment">// i = 0, e1 = 6, e2 = 7</span>
  <span class="hljs-comment">// (a b) c d e f g</span>
  <span class="hljs-comment">// (a b) e c d h f g</span>
  <span class="hljs-comment">// 2. 从尾部开始同步</span>
  <span class="hljs-comment">// i = 2, e1 = 6, e2 = 7</span>
  <span class="hljs-comment">// (a b) c (d e)</span>
  <span class="hljs-comment">// (a b) (d e)</span>
  <span class="hljs-comment">// 3. 普通序列挂载剩余的新节点， 不满足</span>
  <span class="hljs-comment">// 4. 普通序列删除多余的节点，不满足</span>
  <span class="hljs-comment">// i = 2, e1 = 4, e2 = 5</span>
  <span class="hljs-comment">// 旧子节点开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s1 = i
  <span class="hljs-comment">// 新子节点开始索引，从 i 开始记录</span>
  <span class="hljs-keyword">const</span> s2 = i <span class="hljs-comment">//</span>
  <span class="hljs-comment">// 5.1 根据 key 建立新子序列的索引图</span>
  <span class="hljs-comment">// 5.2 正序遍历旧子序列，找到匹配的节点更新，删除不在新子序列中的节点，判断是否有移动节点</span>
  <span class="hljs-comment">// 5.3 移动和挂载新节点</span>
  <span class="hljs-comment">// 仅当节点移动时生成最长递增子序列</span>
  <span class="hljs-keyword">const</span> increasingNewIndexSequence = moved
    ? getSequence(newIndexToOldIndexMap)
    : EMPTY_ARR
  <span class="hljs-keyword">let</span> j = increasingNewIndexSequence.length - <span class="hljs-number">1</span>
  <span class="hljs-comment">// 倒序遍历以便我们可以使用最后更新的节点作为锚点</span>
  <span class="hljs-keyword">for</span> (i = toBePatched - <span class="hljs-number">1</span>; i &gt;= <span class="hljs-number">0</span>; i--) {
    <span class="hljs-keyword">const</span> nextIndex = s2 + i
    <span class="hljs-keyword">const</span> nextChild = c2[nextIndex]
    <span class="hljs-comment">// 锚点指向上一个更新的节点，如果 nextIndex 超过新子节点的长度，则指向 parentAnchor</span>
    <span class="hljs-keyword">const</span> anchor = nextIndex + <span class="hljs-number">1</span> &lt; l2 ? c2[nextIndex + <span class="hljs-number">1</span>].el : parentAnchor
    <span class="hljs-keyword">if</span> (newIndexToOldIndexMap[i] === <span class="hljs-number">0</span>) {
      <span class="hljs-comment">// 挂载新的子节点</span>
      patch(<span class="hljs-literal">null</span>, nextChild, container, anchor, parentComponent, parentSuspense, isSVG)
    }
    <span class="hljs-keyword">else</span> <span class="hljs-keyword">if</span> (moved) {
      <span class="hljs-comment">// 没有最长递增子序列（reverse 的场景）或者当前的节点索引不在最长递增子序列中，需要移动</span>
      <span class="hljs-keyword">if</span> (j &lt; <span class="hljs-number">0</span> || i !== increasingNewIndexSequence[j]) {
        move(nextChild, container, anchor, <span class="hljs-number">2</span>)
      }
      <span class="hljs-keyword">else</span> {
        <span class="hljs-comment">// 倒序递增子序列</span>
        j--
      }
    }
  }
}
</code></pre>
<p data-nodeid="1367">我们前面已经判断了是否移动，如果 moved 为 true 就通过 getSequence(newIndexToOldIndexMap) 计算最长递增子序列，这部分算法我会放在后文详细介绍。</p>
<p data-nodeid="1368">接着我们采用倒序的方式遍历新子序列，因为倒序遍历可以方便我们使用最后更新的节点作为锚点。在倒序的过程中，锚点指向上一个更新的节点，然后判断 newIndexToOldIndexMap[i] 是否为 0，如果是则表示这是新节点，就需要挂载它；接着判断是否存在节点移动的情况，如果存在的话则看节点的索引是不是在最长递增子序列中，如果在则倒序最长递增子序列，否则把它移动到锚点的前面。</p>
<p data-nodeid="1369">为了便于你更直观地理解，我们用前面的例子展示一下这个过程，此时 toBePatched 的值为 4，j 的值为 1，最长递增子序列 increasingNewIndexSequence 的值是 [1, 2]。在倒序新子序列的过程中，首先遇到节点 i，发现它在 newIndexToOldIndexMap 中的值是 0，则说明它是新节点，我们需要挂载它；然后继续遍历遇到节点 d，因为 moved 为 true，且 d 的索引存在于最长递增子序列中，则执行 j-- 倒序最长递增子序列，j 此时为 0；接着继续遍历遇到节点 c，它和 d 一样，索引也存在于最长递增子序列中，则执行 j--，j 此时为 -1；接着继续遍历遇到节点 e，此时 j 是 -1 并且 e 的索引也不在最长递增子序列中，所以做一次移动操作，把 e 节点移到上一个更新的节点，也就是 c 节点的前面。</p>
<p data-nodeid="1370">新子序列倒序完成，即完成了新节点的插入和旧节点的移动操作，也就完成了整个核心 diff 算法对节点的更新。</p>
<p data-nodeid="1371">我们来看一下示例处理后的结果，如下图所示：</p>
<p data-nodeid="1372"><img src="https://s0.lgstatic.com/i/image/M00/32/C5/Ciqc1F8OxeiAIp0WAAFBcsdATCI981.png" alt="图片16.png" data-nodeid="1583"></p>
<p data-nodeid="1373">可以看到新子序列中的新节点 i 被挂载，旧子序列中的节点 e 移动到了 c 节点前面，至此，我们就在已知旧子节点 DOM 结构和 vnode、新子节点 vnode 的情况下，求解出生成新子节点的 DOM 的更新、移动、删除、新增等系列操作，并且以一种较小成本的方式完成 DOM 更新。</p>
<p data-nodeid="1374">我们知道了子节点更新调用的是 patch 方法， Vue.js 正是通过这种递归的方式完成了整个组件树的更新。</p>
<p data-nodeid="1375">核心 diff 算法中最复杂就是求解最长递增子序列，下面我们再来详细学习一下这个算法。</p>
<h4 data-nodeid="1376">最长递增子序列</h4>
<p data-nodeid="1377">求解最长递增子序列是一道经典的算法题，多数解法是使用动态规划的思想，算法的时间复杂度是 O(n<sup>2</sup>)，而 Vue.js 内部使用的是维基百科提供的一套“贪心 + 二分查找”的算法，贪心算法的时间复杂度是 O(n)，二分查找的时间复杂度是 O(logn)，所以它的总时间复杂度是 O(nlogn)。</p>
<p data-nodeid="1378">单纯地看代码并不好理解，我们用示例来看一下这个子序列的求解过程。</p>
<p data-nodeid="1379">假设我们有这个样一个数组 arr：[2, 1, 5, 3, 6, 4, 8, 9, 7]，求解它最长递增子序列的步骤如下：</p>
<p data-nodeid="1380"><img src="https://s0.lgstatic.com/i/image/M00/32/DC/Ciqc1F8O342ATpU7AMfwii64x74028.gif" alt="序列_05.gif" data-nodeid="1603"></p>
<p data-nodeid="1381">最终求得最长递增子序列的值就是 [1, 3, 4, 8, 9]。</p>
<p data-nodeid="1382">通过演示我们可以得到这个算法的主要思路：对数组遍历，依次求解长度为 i 时的最长递增子序列，当 i 元素大于 i - 1 的元素时，添加 i 元素并更新最长子序列；否则往前查找直到找到一个比 i 小的元素，然后插在该元素后面并更新对应的最长递增子序列。</p>
<p data-nodeid="1383">这种做法的主要目的是让递增序列的差尽可能的小，从而可以获得更长的递增子序列，这便是一种贪心算法的思想。</p>
<p data-nodeid="1384">了解了算法的大致思想后，接下来我们看一下源码实现：</p>
<pre class="lang-js" data-nodeid="1385"><code data-language="js"><span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">getSequence</span> (<span class="hljs-params">arr</span>) </span>{
  <span class="hljs-keyword">const</span> p = arr.slice()
  <span class="hljs-keyword">const</span> result = [<span class="hljs-number">0</span>]
  <span class="hljs-keyword">let</span> i, j, u, v, c
  <span class="hljs-keyword">const</span> len = arr.length
  <span class="hljs-keyword">for</span> (i = <span class="hljs-number">0</span>; i &lt; len; i++) {
    <span class="hljs-keyword">const</span> arrI = arr[i]
    <span class="hljs-keyword">if</span> (arrI !== <span class="hljs-number">0</span>) {
      j = result[result.length - <span class="hljs-number">1</span>]
      <span class="hljs-keyword">if</span> (arr[j] &lt; arrI) {
        <span class="hljs-comment">// 存储在 result 更新前的最后一个索引的值</span>
        p[i] = j
        result.push(i)
        <span class="hljs-keyword">continue</span>
      }
      u = <span class="hljs-number">0</span>
      v = result.length - <span class="hljs-number">1</span>
      <span class="hljs-comment">// 二分搜索，查找比 arrI 小的节点，更新 result 的值</span>
      <span class="hljs-keyword">while</span> (u &lt; v) {
        c = ((u + v) / <span class="hljs-number">2</span>) | <span class="hljs-number">0</span>
        <span class="hljs-keyword">if</span> (arr[result[c]] &lt; arrI) {
          u = c + <span class="hljs-number">1</span>
        }
        <span class="hljs-keyword">else</span> {
          v = c
        }
      }
      <span class="hljs-keyword">if</span> (arrI &lt; arr[result[u]]) {
        <span class="hljs-keyword">if</span> (u &gt; <span class="hljs-number">0</span>) {
          p[i] = result[u - <span class="hljs-number">1</span>]
        }
        result[u] = i
      }
    }
  }
  u = result.length
  v = result[u - <span class="hljs-number">1</span>]

  <span class="hljs-comment">// 回溯数组 p，找到最终的索引</span>
  <span class="hljs-keyword">while</span> (u-- &gt; <span class="hljs-number">0</span>) {
    result[u] = v
    v = p[v]
  }
  <span class="hljs-keyword">return</span> result
}
</code></pre>
<p data-nodeid="1386">其中 result 存储的是长度为 i 的递增子序列最小末尾值的索引。比如我们上述例子的第九步，在对数组 p 回溯之前， result 值就是 [1, 3, 4, 7, 9] ，这不是最长递增子序列，它只是存储的对应长度递增子序列的最小末尾。因此在整个遍历过程中会额外用一个数组 p，来存储在每次更新 result 前最后一个索引的值，并且它的 key 是这次要更新的 result 值：</p>
<pre class="lang-java" data-nodeid="1387"><code data-language="java">j = result[result.length - <span class="hljs-number">1</span>]
p[i] = j
result.push(i)
</code></pre>
<p data-nodeid="1388">可以看到，result 添加的新值 i 是作为 p 存储 result 最后一个值 j 的 key。上述例子遍历后 p 的结果如图所示：</p>
<p data-nodeid="1389"><img src="https://s0.lgstatic.com/i/image/M00/32/C5/Ciqc1F8OxgOALDcQAABERFRRNqo370.png" alt="图片17.png" data-nodeid="1620"></p>
<p data-nodeid="1390">从 result 最后一个元素 9 对应的索引 7 开始回溯，可以看到 p[7] = 6，p[6] = 5，p[5] = 3，p[3] = 1，所以通过对 p 的回溯，得到最终的 result 值是 [1, 3 ,5 ,6 ,7]，也就找到最长递增子序列的最终索引了。这里要注意，我们求解的是最长子序列索引值，它的每个元素其实对应的是数组的下标。对于我们的例子而言，[2, 1, 5, 3, 6, 4, 8, 9, 7] 的最长子序列是 [1, 3, 4, 8, 9]，而我们求解的 [1, 3 ,5 ,6 ,7] 就是最长子序列中元素在原数组中的下标所构成的新数组。</p>
<h2 data-nodeid="1391">总结</h2>
<p data-nodeid="1392">这两节课我们主要分析了组件的更新流程，知道了 Vue.js 的更新粒度是组件级别的，并且 Vue.js 在 patch 某个组件的时候，如果遇到组件这类抽象节点，在某些条件下也会触发子组件的更新。</p>
<p data-nodeid="1393">对于普通元素节点的更新，主要是更新一些属性，以及它的子节点。子节点的更新又分为多种情况，其中最复杂的情况为数组到数组的更新，内部又根据不同情况分成几个流程去 diff，遇到需要移动的情况还要去求解子节点的最长递增子序列。</p>
<p data-nodeid="1394">整个更新过程还是利用了树的深度遍历，递归执行 patch 方法，最终完成了整个组件树的更新。</p>
<p data-nodeid="1395">下面，我们通过一张图来更加直观感受组件的更新流程：</p>
<p data-nodeid="1396"><img src="https://s0.lgstatic.com/i/image/M00/32/CB/Ciqc1F8OyzuASuJ7AAHSjr5SVlc999.png" alt="1.png" data-nodeid="1661"></p>
<p data-nodeid="1397">最后，给你留一道思考题目，我们使用 v-for 编写列表的时候 key 能用遍历索引 index 表示吗，为什么？欢迎你在留言区与我分享。</p>
<blockquote data-nodeid="1398">
<p data-nodeid="1399" class=""><strong data-nodeid="1668">本节课的相关代码在源代码中的位置如下：</strong><br>
packages/runtime-core/src/renderer.ts</p>
</blockquote>

---

### 精选评论

##### **国：
> 在一个列表中，使用index来当key值在进行排序或删除操作的时候，那么根据key建立的索引图会导致重新绑定，使得diff算法重新patch，那么会导致不必要的patch和渲染，有时候还会导致明明删除的指定的元素，但却删除了另一个元素的问题。举个例子：oldVnode列表的key值和元素文本节点关系为0 - 1, 1 - 2, 2 - 3。此时我进行了reverse操作，那么此时的关系为0 - 3, 1 - 2, 2 - 1，由于key值相同的vnode是同一个节点，那么此时这三个newVnode需要进行建立索引图-patch - 重新建立索引图- 依次update -Re render。如果key值不是用index索引做值的话，diff算法在进行patch时，由于key到元素没有变化(索引图没有变化)，那么diff算法只会做必要的操作(删除或移动Vnode)，这样就大大的减少了不必要的patch和渲染。综上，个人认为如果不对dom做操作的话，是可以使用index作为元素的key值的，反之，需要使用唯一的值来作为key值。

##### **杰：
> 老师，vue3的diff算法和vue2的对比有什么优势么

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Vue 3 diff 算法的主要优势是设计了 Block  的概念，在编译阶段对静态模板分析，生成 Block tree，收集动态更新的节点，然后在 patch 阶段就可以只比对 Block tree 中的节点，达到提升 diff 性能的目的，这块内容在后续编译章节会提到。而核心 diff 算法，也就是去头尾的最长递增子序列算法和双端比较算法就性能而言差别并不大。

##### **宇：
> 最长上升子序列的求解还是得结合那个wiki的解释才能完全看懂😂

##### **通：
> 静态数据 v-for 是可以的，因为未涉及到 DOM 的更新。但在动态变化的数据 v-for 时，最好不用索引 index 作为 key 值，因为在 patchKeyedChildren 第一步就是头头比较 isSameVNodeType( n1.type === n2.type n1.key === n2.key ) 这样每次都是相同的，走的直接就是更新(props, children)，未能利用到移动的特性，来提高性能。

##### *超：
> 和vue2的diff算法不一样了吗，vue2好像没有最长递增子序列

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; Vue.js 3 在 diff 算法上和 Vue.js 2.x 已经不一样了，Vue.js 2.x 的 diff 算法是双端比较法

##### **翔：
> 老师能讲一下 getSequence 中 arrI !== 0 这一行意思吗？去看了 vue 源码，这里的 getSequence 是否一种特殊值求解，比如我输入 [-1, 0, 1, 2] ，输出解是 [0, 2, 3]，而不明 [0, 1, 2, 3]。第二，getSequence 中复用了许多变量，表示的并不是一种意义，看起来真的有点混乱，这样的代码可能追求了性能和简化，但阅读起来真的难以理解

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 首先，0 是没有意义的，在 diff 场景中甚至都不会出现小于 0 的数，所以不会处理，其次 getSequence 就是基于一种算法的实现，可能这个算法实现相对固定，之后都不会改，所以从代码阅读性上并没有什么追求，当然这个也是可以优化的点。变量命名简化和性能无关，最终压缩后的代码都差不多。

