<p data-nodeid="229246" class="">在上一节课，我们学习了 Netty 的内存规格分类以及内存管理的核心组件，今天这节课我们继续介绍 Netty 内存分配与回收的实现原理。有了上节课的基础，相信接下来的学习过程会事半功倍。</p>
<p data-nodeid="229247">本节课会侧重于详细分析不同场景下 Netty 内存分配和回收的实现过程，让你对 Netty 内存池的整体设计有一个更加清晰的认识。</p>
<h3 data-nodeid="229248">内存分配实现原理</h3>
<p data-nodeid="229249">Netty 中负责线程分配的组件有两个：<strong data-nodeid="229349">PoolArena</strong>和<strong data-nodeid="229350">PoolThreadCache</strong>。PoolArena 是多个线程共享的，每个线程会固定绑定一个 PoolArena，PoolThreadCache 是每个线程私有的缓存空间，如下图所示。</p>
<p data-nodeid="229250"><img src="https://s0.lgstatic.com/i/image/M00/75/1F/Ciqc1F_HVVWANQ2JAASi2VFvKEg368.png" alt="Drawing 0.png" data-nodeid="229353"></p>
<p data-nodeid="229251">在上节课中，我们介绍了 PoolChunk、PoolSubpage、PoolChunkList，它们都是 PoolArena 中所用到的概念。PoolArena 中管理的内存单位为 PoolChunk，每个 PoolChunk 会被划分为 2048 个 8K 的 Page。在申请的内存大于 8K 时，PoolChunk 会以 Page 为单位进行内存分配。当申请的内存大小小于 8K 时，会由 PoolSubpage 管理更小粒度的内存分配。</p>
<p data-nodeid="229252">PoolArena 分配的内存被释放后，不会立即会还给 PoolChunk，而且会缓存在本地私有缓存 PoolThreadCache 中，在下一次进行内存分配时，会优先从 PoolThreadCache 中查找匹配的内存块。</p>
<p data-nodeid="229253">由此可见，Netty 中不同的内存规格采用的分配策略是不同的，我们主要分为以下三个场景逐一进行分析。</p>
<ul data-nodeid="229254">
<li data-nodeid="229255">
<p data-nodeid="229256">分配内存大于 8K 时，PoolChunk 中采用的 Page 级别的内存分配策略。</p>
</li>
<li data-nodeid="229257">
<p data-nodeid="229258">分配内存小于 8K 时，由 PoolSubpage 负责管理的内存分配策略。</p>
</li>
<li data-nodeid="229259">
<p data-nodeid="229260">分配内存小于 8K 时，为了提高内存分配效率，由 PoolThreadCache 本地线程缓存提供的内存分配。</p>
</li>
</ul>
<h4 data-nodeid="229261">PoolChunk 中 Page 级别的内存分配</h4>
<p data-nodeid="229262">每个 PoolChunk 默认大小为 16M，PoolChunk 是通过伙伴算法管理多个 Page，每个 PoolChunk 被划分为 2048 个 Page，最终通过一颗满二叉树实现，我们再一起回顾下 PoolChunk 的二叉树结构，如下图所示。</p>
<p data-nodeid="229263"><img src="https://s0.lgstatic.com/i/image/M00/75/1F/Ciqc1F_HVV2AAm7jAAkM7nU1E0A130.png" alt="Drawing 1.png" data-nodeid="229364"></p>
<p data-nodeid="229264">假如用户需要依次申请 8K、16K、8K 的内存，通过这里例子我们详细描述下 PoolChunk 如何分配 Page 级别的内存，方便大家理解伙伴算法的原理。</p>
<p data-nodeid="229265">首先看下分配逻辑 allocateRun 的源码，如下所示。PoolChunk 分配 Page 主要分为三步：首先根据分配内存大小计算二叉树所在节点的高度，然后查找对应高度中是否存在可用节点，如果分配成功则减去已分配的内存大小得到剩余可用空间。</p>
<pre class="lang-java" data-nodeid="229266"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> <span class="hljs-title">allocateRun</span><span class="hljs-params">(<span class="hljs-keyword">int</span> normCapacity)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 根据分配内存大小计算二叉树对应的节点高度</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> d = maxOrder - (log2(normCapacity) - pageShifts);
&nbsp; &nbsp; <span class="hljs-comment">// 查找对应高度中是否存在可用节点</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> id = allocateNode(d);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (id &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> id;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 减去已分配的内存大小</span>
&nbsp; &nbsp; freeBytes -= runLength(id);
&nbsp; &nbsp; <span class="hljs-keyword">return</span> id;
}
</code></pre>
<p data-nodeid="229267">结合 PoolChunk 的二叉树结构以及 allocateRun 源码我们开始分析模拟的示例：</p>
<p data-nodeid="229268">第一次分配 8K 大小的内存时，通过 d = maxOrder - (log2(normCapacity) - pageShifts) 计算得到二叉树所在节点高度为 11，其中 maxOrder 为二叉树的最大高度，normCapacity 为 8K，pageShifts 默认值为 13，因为只有当申请内存大小大于 2^13 = 8K 时才会使用 allocateRun 分配内存。然后从第 11 层查找可用的 Page，下标为 2048 的节点可以被用于分配内存，即 Page[0] 被分配使用，此时赋值 memoryMap[2048] = 12，表示该节点已经不可用，然后递归更新父节点的值，父节点的值取两个子节点的最小值，memoryMap[1024] = 11，memoryMap[512] = 10，以此类推直至 memoryMap[1] = 1，更新后的二叉树分配结果如下图所示。</p>
<p data-nodeid="230541" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/47/Ciqc1F_HcU2AUwUeAAS29sFoCrk381.png" alt="图片3.png" data-nodeid="230544"></p>



<p data-nodeid="229270">第二次分配 16K 大小内存时，计算得到所需节点的高度为 10。此时 1024 节点已经分配了一个 8K 内存，不再满足条件，继续寻找到 1025 节点。1025 节点并未使用过，满足分配条件，于是将 1025 节点的两个子节点 2050 和 2051 全部分配出去，并赋值 memoryMap[2050] = 12，memoryMap[2051] = 12，再次递归更新父节点的值，更新后的二叉树分配结果如下图所示。</p>
<p data-nodeid="231055" class=""><img src="https://s0.lgstatic.com/i/image/M00/75/53/CgqCHl_HcVyANkl0AASIJd7RNAc086.png" alt="图片4.png" data-nodeid="231058"></p>

<p data-nodeid="229272">第三次再次分配 8K 大小的内存时，依然从二叉树第 11 层开始查找，2048 已经被使用，2049 可以被分配，赋值 memoryMap[2049] = 12，并递归更新父节点值，memoryMap[1024] = 12，memoryMap[512] = 12，以此类推直至 memoryMap[1] = 1，最终的二叉树分配结果如下图所示。</p>
<p data-nodeid="231569" class="te-preview-highlight"><img src="https://s0.lgstatic.com/i/image/M00/75/53/CgqCHl_HcWWAZpDsAASgCUhZxLw922.png" alt="图片5.png" data-nodeid="231572"></p>

<p data-nodeid="229274" class="">至此，PoolChunk 中 Page 级别的内存分配已经介绍完了，可以看出伙伴算法尽可能保证了分配内存地址的连续性，有效地降低了内存碎片。</p>
<h4 data-nodeid="229275">Subpage 级别的内存分配</h4>
<p data-nodeid="229276">为了提高内存分配的利用率，在分配小于 8K 的内存时，PoolChunk 不在分配单独的 Page，而是将 Page 划分为更小的内存块，由 PoolSubpage 进行管理。</p>
<p data-nodeid="229277">首先我们看下 PoolSubpage 的创建过程，由于分配的内存小于 8K，所以走到了 allocateSubpage 源码中：</p>
<pre class="lang-java" data-nodeid="229278"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">long</span> <span class="hljs-title">allocateSubpage</span><span class="hljs-params">(<span class="hljs-keyword">int</span> normCapacity)</span> </span>{
&nbsp; &nbsp; <span class="hljs-comment">// 根据内存大小找到 PoolArena 中 subpage 数组对应的头结点</span>
&nbsp; &nbsp; PoolSubpage&lt;T&gt; head = arena.findSubpagePoolHead(normCapacity);
&nbsp; &nbsp; <span class="hljs-keyword">int</span> d = maxOrder; <span class="hljs-comment">// 因为分配内存小于 8K，所以从满二叉树最底层开始查找</span>
&nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (head) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> id = allocateNode(d); <span class="hljs-comment">// 在满二叉树中找到一个可用的节点</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (id &lt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> id;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> PoolSubpage&lt;T&gt;[] subpages = <span class="hljs-keyword">this</span>.subpages; <span class="hljs-comment">// 记录哪些 Page 被转化为 Subpage</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> pageSize = <span class="hljs-keyword">this</span>.pageSize;&nbsp;
&nbsp; &nbsp; &nbsp; &nbsp; freeBytes -= pageSize;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> subpageIdx = subpageIdx(id); <span class="hljs-comment">// pageId 到 subpageId 的转化，例如 pageId=2048 对应的 subpageId=0</span>
&nbsp; &nbsp; &nbsp; &nbsp; PoolSubpage&lt;T&gt; subpage = subpages[subpageIdx];
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (subpage == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 创建 PoolSubpage，并切分为相同大小的子内存块，然后加入 PoolArena 对应的双向链表中</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; subpage = <span class="hljs-keyword">new</span> PoolSubpage&lt;T&gt;(head, <span class="hljs-keyword">this</span>, id, runOffset(id), pageSize, normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; subpages[subpageIdx] = subpage;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; subpage.init(head, normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> subpage.allocate(); <span class="hljs-comment">// 执行内存分配并返回内存地址</span>
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="229279">假如我们需要分配 20B 大小的内存，一起分析下上述源码的执行过程：</p>
<ol data-nodeid="229280">
<li data-nodeid="229281">
<p data-nodeid="229282">因为 20B 小于 512B，属于 Tiny 场景，按照内存规格的分类 20B 需要向上取整到 32B。</p>
</li>
<li data-nodeid="229283">
<p data-nodeid="229284">根据内存规格的大小找到 PoolArena 中 tinySubpagePools 数组对应的头结点，32B 对应的 tinySubpagePools[1]。</p>
</li>
<li data-nodeid="229285">
<p data-nodeid="229286">在满二叉树中寻找可用的节点用于内存分配，因为我们分配的内存小于 8K，所以直接从二叉树的最底层开始查找。假如 2049 节点是可用的，那么返回的 id = 2049。</p>
</li>
<li data-nodeid="229287">
<p data-nodeid="229288">找到可用节点后，因为 pageIdx 是从叶子节点 2048 开始记录索引，而 subpageIdx 需要从 0 开始的，所以需要将 pageIdx 转化为 subpageIdx，例如 2048 对应的 subpageIdx = 0，2049 对应的 subpageIdx = 1，以此类推。</p>
</li>
<li data-nodeid="229289">
<p data-nodeid="229290">如果 PoolChunk 中 subpages 数组的 subpageIdx 下标对应的 PoolSubpage 不存在，那么将创建一个新的 PoolSubpage，并将 PoolSubpage 切分为相同大小的子内存块，示例对应的子内存块大小为 32B，最后将新创建的 PoolSubpage 节点与 tinySubpagePools[1] 对应的 head 节点连接成双向链表。</p>
</li>
<li data-nodeid="229291">
<p data-nodeid="229292">最后 PoolSubpage 执行内存分配并返回内存地址。</p>
</li>
</ol>
<p data-nodeid="229293">接下来我们跟进一下 subpage.allocate() 源码，看下 PoolSubpage 是如何执行内存分配的，源码如下：</p>
<pre class="lang-java" data-nodeid="229294"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">long</span> <span class="hljs-title">allocate</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (elemSize == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> toHandle(<span class="hljs-number">0</span>);
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (numAvail == <span class="hljs-number">0</span> || !doNotDestroy) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> -<span class="hljs-number">1</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> bitmapIdx = getNextAvail(); <span class="hljs-comment">// 在 bitmap 中找到第一个索引段，然后将该 bit 置为 1</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> q = bitmapIdx &gt;&gt;&gt; <span class="hljs-number">6</span>; <span class="hljs-comment">// 定位到 bitmap 的数组下标</span>
&nbsp; &nbsp; <span class="hljs-keyword">int</span> r = bitmapIdx &amp; <span class="hljs-number">63</span>; <span class="hljs-comment">// 取到节点对应一个 long 类型中的二进制位</span>
&nbsp; &nbsp; <span class="hljs-keyword">assert</span> (bitmap[q] &gt;&gt;&gt; r &amp; <span class="hljs-number">1</span>) == <span class="hljs-number">0</span>;
&nbsp; &nbsp; bitmap[q] |= <span class="hljs-number">1L</span> &lt;&lt; r;
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (-- numAvail == <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; removeFromPool(); <span class="hljs-comment">// 如果 PoolSubpage 没有可分配的内存块，从 PoolArena 双向链表中删除</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> toHandle(bitmapIdx);
}
</code></pre>
<p data-nodeid="229295">PoolSubpage 通过位图 bitmap 记录每个内存块是否已经被使用。在上述的示例中，8K/32B = 256，因为每个 long 有 64 位，所以需要 256/64 = 4 个 long 类型的即可描述全部的内存块分配状态，因此 bitmap 数组的长度为 4，从 bitmap[0] 开始记录，每分配一个内存块，就会移动到 bitmap[0] 中的下一个二进制位，直至 bitmap[0] 的所有二进制位都赋值为 1，然后继续分配 bitmap[1]，以此类推。当我们使用 2049 节点进行内存分配时，bitmap[0] 中的二进制位如下图所示：</p>
<p data-nodeid="229296"><img src="https://s0.lgstatic.com/i/image/M00/75/20/Ciqc1F_HVYCALlpoAAIr-7_6X50667.png" alt="Drawing 5.png" data-nodeid="229467"></p>
<p data-nodeid="229297">当 bitmap 分成成功后，PoolSubpage 会将可用节点的个数 numAvail 减 1，当 numAvail 降为 0 时，表示 PoolSubpage 已经没有可分配的内存块，此时需要从 PoolArena 中 tinySubpagePools[1] 的双向链表中删除。</p>
<p data-nodeid="229298">至此，整个 PoolChunk 中 Subpage 的内存分配过程已经完成了，可见 PoolChunk 的伙伴算法几乎贯穿了整个流程，位图 bitmap 的设计也是非常巧妙的，不仅节省了内存空间，而且加快了定位内存块的速度。</p>
<h4 data-nodeid="229299">PoolThreadCache 的内存分配</h4>
<p data-nodeid="229300">上节课已经介绍了 PoolThreadCache 的基本概念，我们知道 PoolArena 分配的内存被释放时，Netty 并没有将缓存归还给 PoolChunk，而是使用 PoolThreadCache 缓存起来，当下次有同样规格的内存分配时，直接从 PoolThreadCache 取出使用即可。所以下面我们从 PoolArena#allocate() 的源码中看下 PoolThreadCache 是如何使用的。</p>
<pre class="lang-java" data-nodeid="229301"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">void</span> <span class="hljs-title">allocate</span><span class="hljs-params">(PoolThreadCache cache, PooledByteBuf&lt;T&gt; buf, <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> reqCapacity)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">final</span> <span class="hljs-keyword">int</span> normCapacity = normalizeCapacity(reqCapacity);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (isTinyOrSmall(normCapacity)) { <span class="hljs-comment">// capacity &lt; pageSize</span>
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> tableIdx;
&nbsp; &nbsp; &nbsp; &nbsp; PoolSubpage&lt;T&gt;[] table;
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">boolean</span> tiny = isTiny(normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (tiny) { <span class="hljs-comment">// &lt; 512</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cache.allocateTiny(<span class="hljs-keyword">this</span>, buf, reqCapacity, normCapacity)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; tableIdx = tinyIdx(normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; table = tinySubpagePools;
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cache.allocateSmall(<span class="hljs-keyword">this</span>, buf, reqCapacity, normCapacity)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; tableIdx = smallIdx(normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; table = smallSubpagePools;
&nbsp; &nbsp; &nbsp; &nbsp; }

&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// 省略其他代码</span>
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (normCapacity &lt;= chunkSize) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (cache.allocateNormal(<span class="hljs-keyword">this</span>, buf, reqCapacity, normCapacity)) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span>;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">synchronized</span> (<span class="hljs-keyword">this</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; allocateNormal(buf, reqCapacity, normCapacity);
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ++allocationsNormal;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; allocateHuge(buf, reqCapacity);
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="229302">从源码中可以看出在分配 Tiny、Small 和 Normal 类型的内存时，都会尝试先从 PoolThreadCache 中进行分配，源码结构比较清晰，我们整体梳理一遍流程：</p>
<ol data-nodeid="229303">
<li data-nodeid="229304">
<p data-nodeid="229305">对申请的内存大小做向上取整，例如 20B 的内存大小会取整为 32B。</p>
</li>
<li data-nodeid="229306">
<p data-nodeid="229307">当申请的内存大小小于 8K 时，分为 Tiny 和 Small 两种情况，分别都会优先尝试从 PoolThreadCache 分配内存，如果 PoolThreadCache 分配失败，才会走 PoolArena 的分配流程。</p>
</li>
<li data-nodeid="229308">
<p data-nodeid="229309">当申请的内存大小大于 8K，但是小于 Chunk 的默认大小 16M，属于 Normal 的内存分配，也会优先尝试从 PoolThreadCache 分配内存，如果 PoolThreadCache 分配失败，才会走 PoolArena 的分配流程。</p>
</li>
<li data-nodeid="229310">
<p data-nodeid="229311">当申请的内存大小大于 Chunk 的 16M，则不会经过 PoolThreadCache，直接进行分配。</p>
</li>
</ol>
<p data-nodeid="229312">PoolThreadCache 具体分配内存的过程使用到了一个重要的数据结构 MemoryRegionCache，关于 MemoryRegionCache 的概念你可以回顾下上节课的内容，在这里我就不再赘述了。假如我们现在需要分配 32B 大小的堆外内存，会从 MemoryRegionCache 数组 tinySubPageDirectCaches[1] 中取出对应的 MemoryRegionCache 节点，尝试从 MemoryRegionCache 的队列中取出可用的内存块。</p>
<h3 data-nodeid="229313">内存回收实现原理</h3>
<p data-nodeid="229314">通过之前的介绍我们知道，当用户线程释放内存时会将内存块缓存到本地线程的私有缓存 PoolThreadCache 中，这样在下次分配内存时会提高分配效率，但是当内存块被用完一次后，再没有分配需求，那么一直驻留在内存中又会造成浪费。接下来我们就看下 Netty 是如何实现内存释放的呢？直接跟进下 PoolThreadCache 的源码。</p>
<pre class="lang-java" data-nodeid="229315"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">boolean</span> <span class="hljs-title">allocate</span><span class="hljs-params">(MemoryRegionCache&lt;?&gt; cache, PooledByteBuf buf, <span class="hljs-keyword">int</span> reqCapacity)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (cache == <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> <span class="hljs-keyword">false</span>;
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-comment">// 默认每执行 8192 次 allocate()，就会调用一次 trim() 进行内存整理</span>
&nbsp; &nbsp; <span class="hljs-keyword">boolean</span> allocated = cache.allocate(buf, reqCapacity);
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (++ allocations &gt;= freeSweepAllocationThreshold) {
&nbsp; &nbsp; &nbsp; &nbsp; allocations = <span class="hljs-number">0</span>;
&nbsp; &nbsp; &nbsp; &nbsp; trim();
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> allocated;
}
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">trim</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; trim(tinySubPageDirectCaches);
&nbsp; &nbsp; trim(smallSubPageDirectCaches);
&nbsp; &nbsp; trim(normalDirectCaches);
&nbsp; &nbsp; trim(tinySubPageHeapCaches);
&nbsp; &nbsp; trim(smallSubPageHeapCaches);
&nbsp; &nbsp; trim(normalHeapCaches);
}
</code></pre>
<p data-nodeid="229316">从源码中可以看出，Netty 记录了 allocate() 的执行次数，默认每执行 8192 次，就会触发 PoolThreadCache 调用一次 trim() 进行内存整理，会对 PoolThreadCache 中维护的六个 MemoryRegionCache 数组分别进行整理。我们继续跟进 trim 的源码，定位到核心逻辑。</p>
<pre class="lang-java" data-nodeid="229317"><code data-language="java"><span class="hljs-function"><span class="hljs-keyword">public</span> <span class="hljs-keyword">final</span> <span class="hljs-keyword">void</span> <span class="hljs-title">trim</span><span class="hljs-params">()</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> free = size - allocations;
&nbsp; &nbsp; allocations = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-comment">// We not even allocated all the number that are</span>
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (free &gt; <span class="hljs-number">0</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; free(free, <span class="hljs-keyword">false</span>);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">private</span> <span class="hljs-keyword">int</span> <span class="hljs-title">free</span><span class="hljs-params">(<span class="hljs-keyword">int</span> max, <span class="hljs-keyword">boolean</span> finalizer)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">int</span> numFreed = <span class="hljs-number">0</span>;
&nbsp; &nbsp; <span class="hljs-keyword">for</span> (; numFreed &lt; max; numFreed++) {
&nbsp; &nbsp; &nbsp; &nbsp; Entry&lt;T&gt; entry = queue.poll();
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (entry != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; freeEntry(entry, finalizer);
&nbsp; &nbsp; &nbsp; &nbsp; } <span class="hljs-keyword">else</span> {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-comment">// all cleared</span>
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">return</span> numFreed;
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
&nbsp; &nbsp; <span class="hljs-keyword">return</span> numFreed;
}
</code></pre>
<p data-nodeid="229318">通过 size - allocations 衡量内存分配执行的频繁程度，其中 size 为该 MemoryRegionCache 对应的内存规格大小，size 为固定值，例如 Tiny 类型默认为 512。allocations 表示 MemoryRegionCache 距离上一次内存整理已经发生了多少次 allocate 调用，当调用次数小于 size 时，表示 MemoryRegionCache 中缓存的内存块并不常用，从队列中取出内存块依次释放。</p>
<p data-nodeid="229319">此外 Netty 在线程退出的时候还会回收该线程的所有内存，PoolThreadCache 重载了 finalize() 方法，在销毁前执行缓存回收的逻辑，对应源码如下：</p>
<pre class="lang-java" data-nodeid="229320"><code data-language="java"><span class="hljs-meta">@Override</span>
<span class="hljs-function"><span class="hljs-keyword">protected</span> <span class="hljs-keyword">void</span> <span class="hljs-title">finalize</span><span class="hljs-params">()</span> <span class="hljs-keyword">throws</span> Throwable </span>{
&nbsp; &nbsp; <span class="hljs-keyword">try</span> {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">super</span>.finalize();
&nbsp; &nbsp; } <span class="hljs-keyword">finally</span> {
&nbsp; &nbsp; &nbsp; &nbsp; free(<span class="hljs-keyword">true</span>);
&nbsp; &nbsp; }
}
<span class="hljs-function"><span class="hljs-keyword">void</span> <span class="hljs-title">free</span><span class="hljs-params">(<span class="hljs-keyword">boolean</span> finalizer)</span> </span>{
&nbsp; &nbsp; <span class="hljs-keyword">if</span> (freed.compareAndSet(<span class="hljs-keyword">false</span>, <span class="hljs-keyword">true</span>)) {
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">int</span> numFreed = free(tinySubPageDirectCaches, finalizer) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; free(smallSubPageDirectCaches, finalizer) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; free(normalDirectCaches, finalizer) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; free(tinySubPageHeapCaches, finalizer) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; free(smallSubPageHeapCaches, finalizer) +
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; free(normalHeapCaches, finalizer);
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (numFreed &gt; <span class="hljs-number">0</span> &amp;&amp; logger.isDebugEnabled()) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; logger.debug(<span class="hljs-string">"Freed {} thread-local buffer(s) from thread: {}"</span>, numFreed,
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; Thread.currentThread().getName());
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (directArena != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; directArena.numThreadCaches.getAndDecrement();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; &nbsp; &nbsp; <span class="hljs-keyword">if</span> (heapArena != <span class="hljs-keyword">null</span>) {
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; heapArena.numThreadCaches.getAndDecrement();
&nbsp; &nbsp; &nbsp; &nbsp; }
&nbsp; &nbsp; }
}
</code></pre>
<p data-nodeid="229321">线程销毁时 PoolThreadCache 会依次释放所有 MemoryRegionCache 中的内存数据，其中 free 方法的核心逻辑与之前内存整理 trim 中释放内存的过程是一致的，有兴趣的同学可以自行翻阅源码。</p>
<p data-nodeid="229322">到此为止，整个 Netty 内存池的分配和释放原理我们已经分析完了，其中巧妙的设计思路以及源码细节的实现，都是非常值得我们学习的宝贵资源。</p>
<h3 data-nodeid="229323">总结</h3>
<p data-nodeid="229324">最后，我们对 Netty 内存池的设计思想做一个知识点总结：</p>
<ul data-nodeid="229325">
<li data-nodeid="229326">
<p data-nodeid="229327">分四种内存规格管理内存，分别为 Tiny、Samll、Normal、Huge，PoolChunk 负责管理 8K 以上的内存分配，PoolSubpage 用于管理 8K 以下的内存分配。当申请内存大于 16M 时，不会经过内存池，直接分配。</p>
</li>
<li data-nodeid="229328">
<p data-nodeid="229329">设计了本地线程缓存机制 PoolThreadCache，用于提升内存分配时的并发性能。用于申请 Tiny、Samll、Normal 三种类型的内存时，会优先尝试从 PoolThreadCache 中分配。</p>
</li>
<li data-nodeid="229330">
<p data-nodeid="229331">PoolChunk 使用伙伴算法管理 Page，以二叉树的数据结构实现，是整个内存池分配的核心所在。</p>
</li>
<li data-nodeid="229332">
<p data-nodeid="229333">每调用 PoolThreadCache 的 allocate() 方法到一定次数，会触发检查 PoolThreadCache 中缓存的使用频率，使用频率较低的内存块会被释放。</p>
</li>
<li data-nodeid="229334">
<p data-nodeid="229335">线程退出时，Netty 会回收该线程对应的所有内存。</p>
</li>
</ul>
<p data-nodeid="229336" class="">Netty 中引入类似 jemalloc 的内存池管理技术可以说是一大突破，将 Netty 的性能又提升了一个台阶，而这种思想不仅可以用于 Netty，在很对缓存的场景下都可以借鉴学习，希望这些优秀的设计思想能够对你有所帮助，在实际工作中学以致用。</p>

---

### 精选评论

##### *双：
> allocateSubpage流程中，当分配20b的内存时，要是之前有一个page也被拆分成20b且没有使用完，是会复用这个page，还是使用一个全新的page

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是可以复用的，会查找PoolArena 中 tinySubpagePools 中是否存在可用的 PoolSubpage。

##### *敏：
> 自己再debug源码看就比较清晰了

##### **豪：
> 上篇文章不是说不存在Page么，直接用的PoolSubpage来表示Page级别结构有点懵逼了

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个确实是 Netty 设计比较特殊的地方，用一个 PoolSubpage 完成了不同规格子内存块的表示，内存池的源码一定要多读几遍会更有感觉。

##### **3997：
> 老师，如果PoolThreadCache一直缓存的话，那随着使用时间的推移，netty不是会一直占用很多的内存吗？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 当用户线程释放内存时会将内存块缓存到本地线程的私有缓存 PoolThreadCache 中，这样在下次分配内存时会提高分配效率，但是当内存块被用完一次后，再没有分配需求，那么一直驻留在内存中又会造成浪费。默认每执行 8192 次 allocate()，就会调用一次 trim() 进行内存整理。

##### **威：
> 老师，讲得真好😊

##### 无：
> 2020-12-03 打卡

