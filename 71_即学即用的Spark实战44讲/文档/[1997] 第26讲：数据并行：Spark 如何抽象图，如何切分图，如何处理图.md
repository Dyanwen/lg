<p data-nodeid="125410" class="">GraphX 与 Spark 其他套件相比相对独立，拥有自己的核心数据结构与算子，本课时的主要内容有三个部分：</p>

<ul data-nodeid="22645">
<li data-nodeid="22646">
<p data-nodeid="22647">GraphX 核心数据结构</p>
</li>
<li data-nodeid="22648">
<p data-nodeid="22649">GraphX 图分区</p>
</li>
<li data-nodeid="22650">
<p data-nodeid="22651">GraphX 图算子</p>
</li>
</ul>
<h3 data-nodeid="22652">GraphX 核心数据结构</h3>
<p data-nodeid="29585">GraphX 用属性图的方式表示图，顶点有属性，边有属性。<strong data-nodeid="29592">存储结构采用的是上一课时中介绍的边集数组的形式</strong>，即一个顶点表，一个边表，如下图所示。</p>
<p data-nodeid="29586" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/41/Ciqc1F8GuVqAa32hAAL2nwnnBmA191.png" alt="Drawing 0.png" data-nodeid="29595"></p>



<p data-nodeid="22656">其中，顶点 ID 是非常重要的字段，它不光是顶点的唯一标识符，也是描述边的唯一手段。顶点表与边表实际上就是 RDD，它们分别为 VertexRDD 与 EdgeRDD。在 Spark 的源码中，Graph 类如下：</p>
<pre class="lang-scala" data-nodeid="22657"><code data-language="scala"><span class="hljs-keyword">abstract</span>&nbsp;<span class="hljs-class"><span class="hljs-keyword">class</span><span class="hljs-title">&nbsp;Graph</span>[<span class="hljs-type">VD</span>:&nbsp;<span class="hljs-type">ClassTag</span>,&nbsp;<span class="hljs-type">ED</span>:&nbsp;<span class="hljs-type">ClassTag</span>]<span class="hljs-title">&nbsp;protected&nbsp;</span>(<span class="hljs-params"></span>)<span class="hljs-title">&nbsp;extends&nbsp;Serializable&nbsp;</span></span>{
&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;vertices:&nbsp;<span class="hljs-type">VertexRDD</span>[<span class="hljs-type">VD</span>]
&nbsp;
&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;edges:&nbsp;<span class="hljs-type">EdgeRDD</span>[<span class="hljs-type">ED</span>]
&nbsp;
&nbsp;&nbsp;<span class="hljs-keyword">val</span>&nbsp;triplets:&nbsp;<span class="hljs-type">RDD</span>[<span class="hljs-type">EdgeTriplet</span>[<span class="hljs-type">VD</span>,&nbsp;<span class="hljs-type">ED</span>]]
&nbsp;&nbsp;...
}
</code></pre>
<p data-nodeid="31704">其中 vertices 为顶点表，VD 为顶点属性类型，edges 为边表，ED 为边属性类型，用户可以通过 Graph 的 vertices 与 edges 成员直接得到顶点 RDD 与边 RDD，顶点 RDD 类型为 VerticeRDD，继承自 RDD[(VertexId, VD)]，边 RDD 类型为 EdgeRDD，继承自 RDD[Edge[ED]]，triplets 表示边点三元组，如下图所示（其中圆柱形分别代表顶点属性与边属性）：</p>
<p data-nodeid="31705" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/4C/CgqCHl8GuWiANtC6AACSAc1Gk_Q851.png" alt="Drawing 1.png" data-nodeid="31720"></p>


<p data-nodeid="22660">通过 triplets 成员，用户可以直接获取到起点顶点、起点顶点属性、终点顶点、终点顶点属性、边与边属性信息。triplets 的生成可以由边表与顶点表通过 ScrId 与 DstId 连接而成。GraphX API 的开发语言目前仅支持 Scala。<strong data-nodeid="22848">从上面的内容可以看出，GraphX 的核心数据结构 Graph 就是由 RDD 封装而成。</strong></p>
<h3 data-nodeid="22661">生成 GraphX 的 Graph 对象</h3>
<p data-nodeid="22662">想要生成 GraphX 的 Graph 对象，主要有以下两种方法：</p>
<ul data-nodeid="22663">
<li data-nodeid="22664">
<p data-nodeid="22665">从已有数据中生成</p>
</li>
</ul>
<p data-nodeid="22666">从已有数据中生成 Graph 对象主要分两大类：第一类是用 Graph 类的伴生对象方法，第二类是用 GraphLoader 类的 edgeListFile 方法。下面我将分别对其进行讲解：</p>
<h4 data-nodeid="37556" class="">1. Graph 伴生对象的方法</h4>






<p data-nodeid="22670">Graph 类的伴生对象方法如下：</p>
<p data-nodeid="56195" class="">def&nbsp;apply[VD:&nbsp;ClassTag,&nbsp;ED:&nbsp;ClassTag](<br>
vertices:&nbsp;RDD[(VertexId,&nbsp;VD)],<br>
edges:&nbsp;RDD[Edge[ED]],<br>
defaultVertexAttr:&nbsp;VD&nbsp;=&nbsp;null.asInstanceOf[VD],<br>
edgeStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY,<br>
vertexStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY):&nbsp;Graph[VD,&nbsp;ED]</p>












<p data-nodeid="22678">该方法调用方式如下：</p>
<pre class="lang-scala" data-nodeid="50302"><code data-language="scala"><span class="hljs-keyword">val</span>&nbsp;verteces:&nbsp;<span class="hljs-type">RDD</span>[(<span class="hljs-type">VertexId</span>,&nbsp;<span class="hljs-type">VD</span>)]&nbsp;=&nbsp;……
<span class="hljs-keyword">val</span>&nbsp;edges:&nbsp;<span class="hljs-type">RDD</span>[<span class="hljs-type">Edge</span>[<span class="hljs-type">ED</span>]]&nbsp;=&nbsp;……
<span class="hljs-keyword">val</span>&nbsp;defaultVerteces:&nbsp;<span class="hljs-type">VD</span>&nbsp;=&nbsp;……
<span class="hljs-keyword">val</span>&nbsp;graph:&nbsp;<span class="hljs-type">Graph</span>[<span class="hljs-type">VD</span>,<span class="hljs-type">ED</span>]&nbsp;=&nbsp;<span class="hljs-type">Graph</span>(verteces,&nbsp;edges,defaultVerteces)
</code></pre>




<p data-nodeid="50830">VertexId 是长整型。这是一种相对完善的生成图的方法，有顶点表、边表。此外，顶点与边的属性均可由数据得到。方法如下：</p>
<p data-nodeid="61582">def&nbsp;fromEdgeTuples[VD:&nbsp;ClassTag](<br>
rawEdges:&nbsp;RDD[(VertexId,&nbsp;VertexId)],<br>
defaultValue:&nbsp;VD,<br>
uniqueEdges:&nbsp;Option[PartitionStrategy]&nbsp;=&nbsp;None,<br>
edgeStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY,<br>
vertexStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY):&nbsp;Graph[VD,&nbsp;Int]</p>
<p data-nodeid="61583">从该函数的第一个参数即可看出，该方法针对的是用数字直接表示边与顶点的数据集，如下：</p>






















<pre class="lang-plain" data-nodeid="22687"><code data-language="plain">1&nbsp;&nbsp;7
4&nbsp;&nbsp;7
2&nbsp;&nbsp;1
</code></pre>
<p data-nodeid="63730">边属性值默认为 1，顶点默认属性值可以指定：</p>

<p data-nodeid="67473">def&nbsp;fromEdges[VD:&nbsp;ClassTag,&nbsp;ED:&nbsp;ClassTag](<br>
edges:&nbsp;RDD[Edge[ED]],<br>
defaultValue:&nbsp;VD,<br>
edgeStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY,<br>
vertexStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY):&nbsp;Graph[VD,&nbsp;ED]</p>
<p data-nodeid="67474">该方法从 RDD[Edge[ED]] 直接生成图，边属性由数据集提供，顶点默认属性值可以指定。</p>











<h4 data-nodeid="73803" class="">2. GraphLoader 的 edgeListFile 方法</h4>






<p data-nodeid="74843">GraphLoader 的 edgeListFile 方法如下：</p>
<p data-nodeid="83218">def&nbsp;edgeListFile(<br>
sc:&nbsp;SparkContext,<br>
path:&nbsp;String,<br>
canonicalOrientation:&nbsp;Boolean&nbsp;=&nbsp;false,<br>
numEdgePartitions:&nbsp;Int&nbsp;=&nbsp;-1,<br>
edgeStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY,<br>
vertexStorageLevel:&nbsp;StorageLevel&nbsp;=&nbsp;StorageLevel.MEMORY_ONLY)<br>
:&nbsp;Graph[Int,&nbsp;Int]</p>
<p data-nodeid="83219">这种方法是由 GraphLoader 类提供的方法，与 fromEdgeTuples 方法类似，但不需要提供顶点默认属性值，顶点属性值与边属性值均默认为 1。</p>




















<ul data-nodeid="22711">
<li data-nodeid="22712">
<p data-nodeid="22713">通过 GraphGenerators API 生成</p>
</li>
</ul>
<p data-nodeid="22714">前面主要介绍的是如何从已有数据生成 Graph 对象，在没有数据时，也可以用 GraphX 自带的 GraphGenerators API 生成符合某种规律的测试图数据。主要有下面几种类型的图：</p>
<p data-nodeid="22715"><strong data-nodeid="22975">基于度的图</strong></p>
<p data-nodeid="84277">下面这种方法生成的图，其顶点的度服从对数正态分布，可以通过参数指定顶点数、边分区数以及顶点的边数均值和标准差，最后一个是随机数种子，可以用当前的时间戳：</p>
<p data-nodeid="91562">def&nbsp;logNormalGraph(<br>
sc:&nbsp;SparkContext,&nbsp;<br>
numVertices:&nbsp;Int,&nbsp;<br>
numEParts:&nbsp;Int&nbsp;=&nbsp;0,&nbsp;<br>
mu:&nbsp;Double&nbsp;=&nbsp;4.0,<br>
sigma:&nbsp;Double&nbsp;=&nbsp;1.3,&nbsp;<br>
seed:&nbsp;Long&nbsp;=&nbsp;-1):&nbsp;Graph[Long,&nbsp;Int]</p>
<p data-nodeid="91563"><strong data-nodeid="91583">R-MAT 图</strong></p>


















<p data-nodeid="22727">R-MAT 代表递归矩阵，通过下面这种方式模拟出来的图与社交网络结构类似：</p>
<p data-nodeid="95725">def&nbsp;rmatGraph(<br>
sc:&nbsp;SparkContext,&nbsp;<br>
requestedNumVertices:&nbsp;Int,&nbsp;<br>
numEdges:&nbsp;Int):&nbsp;Graph[Int,&nbsp;Int]</p>
<p data-nodeid="95726">用上面两种方式生成的图就算参数一样，每次生成的结果也有可能不同。比如下面两种图，只要参数一样，每次生成的图就是一样的。</p>









<p data-nodeid="22734"><strong data-nodeid="23003">网格图</strong></p>
<p data-nodeid="96766">下面的方法会生成一个 rows×cols 的网格。网格中顶点的连接方式是一定的，上面的顶点指向下面的顶点，左边的顶点指向右边的顶点。</p>
<p data-nodeid="100895">def&nbsp;gridGraph(<br>
sc:&nbsp;SparkContext,&nbsp;<br>
rows:&nbsp;Int,&nbsp;<br>
cols:&nbsp;Int):&nbsp;Graph[(Int,&nbsp;Int),&nbsp;Double]</p>
<p data-nodeid="100896"><strong data-nodeid="100910">星形图</strong></p>












<p data-nodeid="101935">下面的方法会生成一个 1×nverts 的星形图（nverts 条边指向一个中心）：</p>
<p data-nodeid="103987" class="">def&nbsp;starGraph(<br>
sc:&nbsp;SparkContext,&nbsp;<br>
nverts:&nbsp;Int):&nbsp;Graph[Int,&nbsp;Int]</p>







<h3 data-nodeid="22748">GraphX 图分区</h3>
<p data-nodeid="109118">分区对于 RDD 来说是一个很重要的概念。它体现了并行的理念，是分布式计算的基础。对于普通 RDD 来说，分区的逻辑很简单，对数据集水平切分即可。但是在 GraphX 中，对 Graph 的分区就没那么简单了。对图进行分区，实际上就是对图进行切割，<strong data-nodeid="109125">通常来说，有两种方式：边切割与顶点切割</strong>。如下图所示，左侧的两块图表示的是边切割，右侧的两块图表示的顶点切割。</p>
<p data-nodeid="109119" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/4D/CgqCHl8GukqAce6qAAHBB0Pagrs805.png" alt="Drawing 2.png" data-nodeid="109128"></p>





<p data-nodeid="22751">在边切割中，边有冗余；在顶点切割中，顶点有冗余。与边切割相比，顶点切割可以减少通信与存储开销。你很容易发现，在 Graph 中的边表其实就是顶点切割的实现。所以在 GraphX 中，采取的是顶点切割的方式对图进行分区。而具体的每个分区包含哪些边，目前 GraphX 提供了 4 种分区策略：</p>
<ul data-nodeid="22752">
<li data-nodeid="22753">
<p data-nodeid="22754">EdgePartition1D；</p>
</li>
<li data-nodeid="22755">
<p data-nodeid="22756">EdgePartition2D；</p>
</li>
<li data-nodeid="22757">
<p data-nodeid="22758">RandomVertexCut；</p>
</li>
<li data-nodeid="22759">
<p data-nodeid="22760">CanonicalRandomVertexCut。</p>
</li>
</ul>
<p data-nodeid="111157">下面我就用一个例子来说明这几个分区策略的不同之处。假设原始图如下：</p>
<p data-nodeid="111158" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/42/Ciqc1F8GuleARe4zAAB-e57qatc654.png" alt="4.png" data-nodeid="111162"></p>


<p data-nodeid="113191">EdgePartition1D 分区策略保证了同一个顶点的出边一定在同一个分区中，如下：</p>
<p data-nodeid="113192" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/4D/CgqCHl8GumOAPZtaAAGGH0naRrk163.png" alt="5.png" data-nodeid="113196"></p>


<p data-nodeid="115225">EdgePartition2D 分区策略将边表看成一个邻接矩阵，其中行标表示边的起点 ID，而列标则表示终点 ID，那么该邻接矩阵如下图所示：</p>
<p data-nodeid="115226" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/4D/CgqCHl8GunCAQ2qqAACRpcfy_aU805.png" alt="Drawing 5.png" data-nodeid="115230"></p>


<p data-nodeid="129001"><strong data-nodeid="129007">其中 0 表示行标和列标对应的起点和终点不存在边，1 表示存在边</strong>。形象地说，分区就是对该矩阵进行分块（虚线所示）。按照上图的分块方式，EdgePartition2D 分区策略的边分布如下图所示：</p>
<p data-nodeid="129002"><img src="https://s0.lgstatic.com/i/image/M00/2F/43/Ciqc1F8Gu0mAJknVAAFz_wYnk1Y036.png" alt="Lark20200709-143727.png" data-nodeid="129010"></p>



<p data-nodeid="126952">因此 EdgePartition2D 的分区数一定是个平方数。RandomVertexCut 的分区策略最为简单，就是对边表进行随机分区，这种方式可以实现边的完全负载均衡，但是对于度分布为幂律分布的图来说，很容易出现数据倾斜的问题，如下图所示：</p>





<p data-nodeid="119298" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/42/Ciqc1F8Guu-AQ4ArAAGEnRZHxUA904.png" alt="2.png" data-nodeid="119302"></p>


<p data-nodeid="121331">CanonicalRandomVertexCut 分区策略与 RandomVertexCut 类似，但它能保证两点之间的边在同一个分区，如下图所示：</p>
<p data-nodeid="121332" class=""><img src="https://s0.lgstatic.com/i/image/M00/2F/42/Ciqc1F8GuvmAVtrtAAF3GgxYi9M571.png" alt="3.png" data-nodeid="121336"></p>


<p data-nodeid="122355">确定好分区策略后，用户可以用如下代码对图分区策略进行设置：</p>
<pre class="lang-scala" data-nodeid="122356"><code data-language="scala">...
<span class="hljs-keyword">val</span>&nbsp;graph&nbsp;=&nbsp;<span class="hljs-type">Graph</span>(nodes,edges)
graph.partitionBy(<span class="hljs-type">PartitionStrategy</span>.<span class="hljs-type">CanonicalRandomVertexCut</span>)
</code></pre>



<h3 data-nodeid="123885">GraphX 图算子</h3>




<p data-nodeid="22778">作为有自己核心数据结构与分区策略的 GraphX，当然也有属于 Graph 独有的算子，不过这些算子平常的使用率并没有那么高。在学习了前面的内容后，相信你能够很快掌握这些算子。</p>
<h4 data-nodeid="22779">属性算子</h4>
<p data-nodeid="22780">属性算子的主要作用是改变边的属性与顶点的属性。</p>
<ul data-nodeid="22781">
<li data-nodeid="22782">
<p data-nodeid="22783">def mapVertices[VD2](map: (VertexId, VD) =&gt; VD2): Graph[VD2, ED]：将 VD 类型的顶点属性值变换为 VD2 类型的顶点属性值。</p>
</li>
<li data-nodeid="22784">
<p data-nodeid="22785">def mapEdges[ED2](map: Edge[ED] =&gt; ED2): Graph[VD, ED2]：将 ED 类型的边属性值变换为 ED2 类型的边属性值。</p>
</li>
<li data-nodeid="22786">
<p data-nodeid="22787">def mapTriplets[ED2](map: EdgeTriplet[VD, ED] =&gt; ED2): Graph[VD, ED2]：将 ED 类型的边点三元组属性值变换为 ED2 类型的边点三元组属性值。</p>
</li>
</ul>
<h4 data-nodeid="22788">结构算子</h4>
<p data-nodeid="22789">该类算子可以改变整个图的结构。</p>
<ul data-nodeid="22790">
<li data-nodeid="22791">
<p data-nodeid="22792">def reverse: Graph[VD, ED]：reverse 算子会将图中所有边的起点与终点进行对调，再返回新生成的图。</p>
</li>
<li data-nodeid="22793">
<p data-nodeid="22794">def subgraph(epred: EdgeTriplet[VD,ED] =&gt; Boolean,vpred: (VertexId, VD) =&gt; Boolean): Graph[VD, ED]：subgraph 算子会根据传入的谓词表达式，分别对顶点与边进行过滤，然后再返回由剩下顶点与边组成的新图。这里，如果移除了点，那么与该点相连的边会断开。</p>
</li>
<li data-nodeid="22795">
<p data-nodeid="22796">def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]：返回和另一个图的公共顶点与边组成的图，类似于两个图求差集。</p>
</li>
<li data-nodeid="22797">
<p data-nodeid="22798">def groupEdges(merge: (ED, ED) =&gt; ED): Graph[VD,ED]：groupEdges 算子会合并相同起点与终点的边，并根据 merge 函数的逻辑生成新的边属性值。</p>
</li>
</ul>
<h4 data-nodeid="22799">连接算子</h4>
<p data-nodeid="22800">连接算子有以下几个。</p>
<ul data-nodeid="22801">
<li data-nodeid="22802">
<p data-nodeid="22803">def joinVertices[U](table: RDD[(VertexId, U)])(map: (VertexId, VD, U) =&gt; VD): Graph[VD, ED]：图本身的顶点表与一个新的顶点表做内连接，用户定义的 map 函数作用于连接上的顶点，生成新的顶点属性值。</p>
</li>
<li data-nodeid="22804">
<p data-nodeid="22805">def outerJoinVertices[U, VD2](table: RDD[(VertexId, U)])(map: (VertexId, VD, Option[U]) =&gt; VD2): Graph[VD2, ED]：图本身的顶点表与一个新的顶点表做外连接，用户定义的 map 函数作用于所有顶点，因此不仅需要处理连接上的顶点属性值合并的问题，还需要考虑如果没有连接上的情况。</p>
</li>
</ul>
<p data-nodeid="22806">图算子与 RDD 的算子类似，主要是对图进行一些简单而常用的变换。</p>
<h3 data-nodeid="22807">小结</h3>
<p data-nodeid="22808">本课时的主要内容与前面模块的内容安排类似，和学习如何生成 RDD、RDD 算子一样，本课时的主要内容是学习如何生成 Graph、图算子等。</p>
<p data-nodeid="124901">值得注意的是，Graph 与 RDD 一样，都比较底层，与 DataFrame 相对应，GraphX 也有基于DataFrame 的套件 GraphFrame，不过现在并没有加入 Spark 官方套件中，有兴趣的话，你可以试一试。</p>

---

### 精选评论


