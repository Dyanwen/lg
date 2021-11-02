<p data-nodeid="1998" class="">在前面的课时中，我们介绍了 Spark 如何抽象图、如何处理图，可能你会觉得有些奇怪，目前学到的这几个图算子离大规模并行挖掘似乎还很遥远，也没什么特别之处。其实，GraphX 真正的精华确实还没有学习到，而这也是本课时我将为你讲到的。</p>


<p data-nodeid="1418">“像顶点一样思考”这句话来源于 Google 在 2010 年发表的一篇论文Pregel，Pregel 是为了纪念大数学家欧拉，著名的欧拉七桥问题提到的那条河就叫 Pregel。<strong data-nodeid="1498">这篇论文提出了一种基于图的大规模并行处理思路</strong>，GraphX 可以认为是 Pregel 的开源实现。</p>
<p data-nodeid="1419">GraphX 的 Pregel API 就是 Pregel 的开源实现，它完全继承了 Pregel 的思想，体现了一种不同的数据处理思路，本课时的内容有：</p>
<ul data-nodeid="1420">
<li data-nodeid="1421">
<p data-nodeid="1422">Pregel 的思想：像顶点一样思考</p>
</li>
<li data-nodeid="1423">
<p data-nodeid="1424">Pregel API</p>
</li>
</ul>
<h3 data-nodeid="1425">像顶点一样思考</h3>
<p data-nodeid="1426">在 Internet 出现后，互联网中的图规模越来越大，如网页之间的链接、社交网络等，这些网络动辄数十亿个顶点、数百亿条边，这对于高效处理这些图提出了新的挑战。Pregel 是一种为此而生的计算模型。在 Pregel 出现之前，要想实现一种处理大规模图数据的算法，需要面临以下几个选项：</p>
<ul data-nodeid="1427">
<li data-nodeid="1428">
<p data-nodeid="1429">构建一个定制化的基础架构需要大量工作，而每一种新算法与图都需要重复这些工作。</p>
</li>
<li data-nodeid="1430">
<p data-nodeid="1431">依赖已有的分布式计算平台，但它们通常不适合处理图，如 MapReduce，这类平台非常擅长处理海量结构化数据，但如果使用这类平台处理图数据，可能会造成性能和易用性方面的问题。这些平台对于 SQL 与聚合场景表现很好，但这些扩展对于更适合消息传递模型的图算法来说通常并不理想。</p>
</li>
<li data-nodeid="1432">
<p data-nodeid="1433">使用单点图算法库，如 NetworkX、JDSL、BGL、LEDA 等，但这类型库对于图规模大小有限制。</p>
</li>
<li data-nodeid="1434">
<p data-nodeid="1435">使用现有的并行图处理系统，如 BGL、CGMgraph，这类型类库虽然能够实现并行图算法，但并没有是实现容错性，或者其他对于超大规模分布式系统非常重要的特性。</p>
</li>
</ul>
<p data-nodeid="1436">Pregel 的目标是构建一个对于表达图算法足够灵活且可扩展与容错的平台，并提供其 API。Pregel <strong data-nodeid="1521">基于整体同步并行模型</strong>（Bulk Synchronous Parallel，BSP），<strong data-nodeid="1522">计算过程包含一系列迭代</strong>，<strong data-nodeid="1523">我们称其为超步</strong>（super step）。在每一个超步中，每个顶点会调用用户自定义函数。</p>
<p data-nodeid="1437">用户自定义函数描述了顶点 V 的行为与单个超步 S。顶点 V 可以读取在超步 S−1 中发送给 V 的消息，并发送消息给其他顶点（这些信息会在超步 S+1 中被读取），然后再修改顶点 V 和它的出边的状态。通常来说，消息是沿着出边的方向发送，也可以通过指定顶点 ID 发送给特定顶点。</p>
<p data-nodeid="1438">BSP 中同步的概念是指当所有顶点计算完成后，才会开始下一轮的迭代。在每个超步中，顶点会并行执行相同的用户自定义函数，这些用户自定义函数描述了整个图算法。<strong data-nodeid="1534">算法停止的条件为每个顶点投票终止</strong>（Vote to halt），在第 0 个超步，所有顶点都是激活状态，所有激活的顶点都会参与到超步的计算中去。顶点通过投票终止来使自己不参与到计算中。这意味着：<strong data-nodeid="1535">如果没有外部触发，该顶点就没有其他工作要做</strong>。直到该顶点收到一条消息，否则 Pregel 框架不会让该顶点参与到接下来的超步计算中去。</p>
<p data-nodeid="2760">如果顶点是通过消息激活的，那么顶点必须显式地使自己进入未激活状态。整个算法停止的条件是所有顶点的状态为未激活，且没有消息在传递。顶点的状态机如下图所示：</p>
<p data-nodeid="2761" class=""><img src="https://s0.lgstatic.com/i/image/M00/31/E3/CgqCHl8NV_CAVC4mAAB_6S8QaE8975.png" alt="3.png" data-nodeid="2765"></p>


<p data-nodeid="3526">Pregel 计算模型对于表达图相关的算法表现力非常强，且更加自然，下图是一个简单的例子，实现的是强连通图的最值传播，可以帮助你加强对 BSP 模型的理解。</p>
<p data-nodeid="3527" class=""><img src="https://s0.lgstatic.com/i/image/M00/31/D8/Ciqc1F8NV_aAHyoCAAENjlh9cGM568.png" alt="4.png" data-nodeid="3531"></p>


<p data-nodeid="1443">给定一个强连通图，每个顶点包含一个属性值。它将最大值传播到每个顶点。在每个超步中，<strong data-nodeid="1549">任何更新了属性值的顶点都会再把消息发送给邻居顶点</strong>。当超步中没有顶点变化时，算法终止。上图中，虚线表示发送消息，灰色表示顶点投票终止。</p>
<h3 data-nodeid="1444">Pregel API</h3>
<p data-nodeid="4298">GraphX 实现了 Pregel 计算框架，用户可以通过图算子的方式调用 Pregel API，如下：</p>


<p data-nodeid="7251" class="te-preview-highlight">def&nbsp;pregel[A:&nbsp;ClassTag](<br>
initialMsg:&nbsp;A,<br>
maxIterations:&nbsp;Int&nbsp;=&nbsp;Int.MaxValue,<br>
activeDirection:&nbsp;EdgeDirection&nbsp;=&nbsp;EdgeDirection.Either)(<br>
vprog:&nbsp;(VertexId,&nbsp;VD,&nbsp;A)&nbsp;=&gt;&nbsp;VD,<br>
sendMsg:&nbsp;EdgeTriplet[VD,&nbsp;ED]&nbsp;=&gt;&nbsp;Iterator[(VertexId,&nbsp;A)],<br>
mergeMsg:&nbsp;(A,&nbsp;A)&nbsp;=&gt;&nbsp;A)<br>
:&nbsp;Graph[VD,&nbsp;ED]</p>














<p data-nodeid="1455"></p>
<p data-nodeid="1456">下面我来分别为你讲解各个参数的含义：</p>
<ul data-nodeid="1457">
<li data-nodeid="1458">
<p data-nodeid="1459">initialMsg：表示在最开始的超步中发送给所有顶点的初始消息，经常被用作初始化顶点的属性值；</p>
</li>
<li data-nodeid="1460">
<p data-nodeid="1461">maxIterations 为最大迭代次数，防止由于算法设计的原因，使程序陷入死循环；</p>
</li>
<li data-nodeid="1462">
<p data-nodeid="1463">activeDirection 为激活条件；</p>
</li>
<li data-nodeid="1464">
<p data-nodeid="1465">vprog、sendMsg、mergeMsg 都是用户自定义函数，它们会在一个超步中依次执行，用户需要在这 3 个函数中实现自己的图算法逻辑；</p>
</li>
<li data-nodeid="1466">
<p data-nodeid="1467">vprog 自定义函数：该函数是在每个超步中首先执行，从函数的声明也可看出，它的作用是，用接收到的消息与该顶点的属性值根据用户实现的逻辑得到新的顶点属性值；</p>
</li>
<li data-nodeid="1468">
<p data-nodeid="1469">sendMsg 自定义函数：该函数在 vprog 之后执行，返回的是一个消息的迭代子，其中元组的第一个元素为发送目的地顶点 ID；</p>
</li>
<li data-nodeid="1470">
<p data-nodeid="1471">mergeMsg 自定义函数：顶点会接收到多条消息，该函数是为了优化消息传输，对消息进行合并，该函数的输出会成为下一个超步中 vprog 的输入。</p>
</li>
</ul>
<p data-nodeid="1472">当所有消息停止传递且所有顶点投票终止时，Pregel 应用也就停止。activeDirection 参数设置的是消息发送的条件，该参数可以具体看成触发执行 sendMsg 的条件，它的值可以是以下几个：</p>
<ul data-nodeid="1473">
<li data-nodeid="1474">
<p data-nodeid="1475">EdgeDirection.Out：表示当边的起点顶点收到上一个超步的消息时，调用 sendMsg。</p>
</li>
<li data-nodeid="1476">
<p data-nodeid="1477">EdgeDirection.In：表示当边的终点顶点收到上一个超步的消息时，调用 sendMsg。</p>
</li>
<li data-nodeid="1478">
<p data-nodeid="1479">EdgeDirection.Either：表示当边的起点顶点或终点顶点收到上一个超步的消息时，调用sendMsg。</p>
</li>
<li data-nodeid="1480">
<p data-nodeid="1481">EdgeDirection.Both：表示当边的起点顶点和边的终点顶点收到上一个超步的消息时，调用 sendMsg。</p>
</li>
</ul>
<p data-nodeid="1482">经过 Pregel 计算模型地抽象，<del data-nodeid="1597">用户</del>很多图挖掘算法都能很轻易地实现分布式，且 vprog、sendMsg、mergeMsg 对于图算法表现能力极强。用户实现这 3 个自定义函数时，<strong data-nodeid="1598">视野中不再是整个图，更不是顶点表与边表，而是图中的一个个顶点与一条条边</strong>，这也是谷歌公司在 Pregel 论文最后提出的“像顶点一样思考（Think Like A Vertex）”的意义所在。</p>
<h3 data-nodeid="1483">小结</h3>
<p data-nodeid="1484">本课时的内容虽然不是很长，但包含的内容非常丰富，在本课时中，我们提出了一种新思路，并将其实现。这种数据处理思路是很值得玩味的，并且对图的场景非常实用，如果你才刚接触，那么对于你来说确实有些距离感与新奇感。</p>
<p data-nodeid="1485">为了帮助你尽快上手，这里留一个思考题：</p>
<p data-nodeid="1486"><strong data-nodeid="1605">如何求图中包含的三角形结构数？</strong></p>
<p data-nodeid="1487">你可以先试着用普通方法来完成，再试着用 Pregel API 实现。</p>
<p data-nodeid="1488">为了降低难度，我这里给一个思路，可以试着实现：</p>
<p data-nodeid="4491">如果顶点发送的某个消息经过三个超步后，刚好回到原点，那么可以认为是一个三角形结构。好了，就提示到这里。如果你还有问题的话，可以在留言区与我互动。</p>

---

### 精选评论


