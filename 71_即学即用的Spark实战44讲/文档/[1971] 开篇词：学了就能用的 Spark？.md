<p style="line-height: 1.75em; text-align: justify;"><span style="color: rgb(63, 63, 63);"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">你好，很高兴我们在《即学即用的 Spark 实战 44 讲》这个课程中相遇，我是范东来，Spark Contributor 和 Superset Contributor，同样也是《Spark 海量数据处理》与《Hadoop 海量数据处理》两本书的作者。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">谈起大数据技术的学习，我觉得自己很幸运，研究生阶段就通过实验室项目积累了很多实践经验，毕业后在担任技术负责人和架构师的过程中，主导参与过国内很多金融机构的大数据项目与平台实施，搭建过整个公司的大数据架构和平台，积累了很多大数据技术的心得。我平时喜欢总结学习，也喜欢分享探讨，在这里我将这些经验梳理汇总，希望也能助你一臂之力。</span></p> 
<h1><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">掌握利器，跟上时代的步伐</span></p></h1> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">如今，数据的增长速度以及重要意义已经无需多言，互联网企业对于数据的利用效率，在某种程度上决定了其企业竞争力，而数据处理技术很大程度上就决定了数据的利用效率。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">诞生于 2009 年的 Apache Spark，<strong>目前已成为全球范围内最流行、功能最全面、社区最活跃的大数据处理技术</strong><strong>。</strong>从 GitHub 的数据中可以看到，在 Apache 的所有开源项目中，Spark 的关注度排名第 3（前两位分别是 RPC 服务框架 Dubbo 和可视化平台 Superset），在所有大数据处理技术中排名第 1。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="https://s0.lgstatic.com/i/image3/M01/85/B1/Cgq2xl6OriuACWpvAAB3UaNF4FA777.png"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"> &nbsp; &nbsp; &nbsp; &nbsp;<img src="https://s0.lgstatic.com/i/image3/M01/0C/9B/Ciqah16OriuAdsN9AAAVdnUJRWU547.png"> &nbsp; &nbsp; &nbsp;</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">此外，Spark 在资本市场也得到了极高的认可，其背后的商业化公司得到了 62 亿美元的估值。目前，绝大多数公司和组织会基于 Spark 生态搭建自己的大数据平台，构建支持业务的数据管道。可以说，提到大数据处理，Spark 已是一个无法回避的话题。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong style="color: rgb(65, 70, 75);color: #41464b;">Spark</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">之于大数据工程师</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">，</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">就好像</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">Java</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">之于</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">后端工程师</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">：</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">学会了并不能保证</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">你</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">一定能够拿到</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">O</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">ffer，但是不会</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">，</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">拿到</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">O</strong><strong style="color: rgb(65, 70, 75);color: #41464b;">ffer</strong><strong style="color: rgb(65, 70, 75);color: #41464b;"> </strong><strong style="color: rgb(65, 70, 75);color: #41464b;">的可能性很小。</strong></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">另外，Spark 也很适合数据科学家与数据分析师进行中小规模数据处理，多语言接口与 SQL 支持让它赢得了很多分析师用户，而且这部分用户中 Spark 使用者的占比也越来越大，俨然成为了数据工程与数据科学的通用方案。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">既然我们在这个专栏里相遇，你应该也已经跃跃欲试，想要掌握 Spark 了。不过别急，在开始学习 Spark 之前，先来看看下面几个问题，它们能更好地帮你理解这门课程。</span></p> 
<h1><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">Spark 适合谁学？</span></p></h1> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">关于 Spark 的定义，</span><a class="ql-link ql-commented ql-commented-background ql-author-32666240" href="http://spark.apache.org/" target="_blank" rel="noopener noreferrer nofollow" data-comment-guid="comment-XnAy4r03ZS2fYowv" style="text-decoration: underline; font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">S</span></a><a class="ql-link ql-commented ql-commented-background ql-author-28618723" href="http://spark.apache.org/" target="_blank" rel="noopener noreferrer nofollow" data-comment-guid="comment-XnAy4r03ZS2fYowv" style="text-decoration: underline; font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">park官方</span></a><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><span style="font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; font-size: 16px; color: rgb(0, 0, 0);">给出的说法是：一个</span>通用的快速分析引擎。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">这里的“通用”有很多含义，我不展开讲，你可以简单地把它理解为：“<strong>供所有的大数据从业人员使用</strong>”。客观地说，这还是一种愿景，不过随着 Spark 3.0 的预发布，距离实现已经很近了。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">“分析”，表示 Spark 主要面向的是数据处理场景。这里用“分析”这个词，其实是一种委婉的说法，希望那些有分析需求的轻度用户也能注意到 Spark，如果用“分布式计算框架”这类字眼，虽然听起来更为专业，但或许会让一部分人望而生畏。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">&nbsp;</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">从业以来，我在不断自我成长与角色转换的过程中，目睹了公司层面大数据架构的演进以及对人才需求的变化。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">严格意义上说，如果一个组织的大数据架构基础越薄弱，对于<strong>大数据工程师</strong>的需求就会越大，但是当这个组织的大数据架构愈加趋于完善与成熟，对于<strong>数据分析师</strong>的需求就会变大。而这两类职位涵盖了数据工程与数据科学领域，都需要掌握 Spark。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">而行业层面，大数据人才将长期保持供不应求的状态。也因此，大数据工程师与数据分析师的行业薪资相对更高，往往成为应用开发工程师可选的职业发展目标。下面是拉勾平台的大厂招聘信息，你可以看到薪资的差距：</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="https://s0.lgstatic.com/i/image3/M01/85/B1/Cgq2xl6OriuAGO3JAAAe5XhXr_I515.png"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="https://s0.lgstatic.com/i/image3/M01/0C/9B/Ciqah16OriyAYEHcAAAca-MtQb4910.png"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><br></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><img src="https://s0.lgstatic.com/i/image3/M01/0C/9B/Ciqah16OriyAfpiqAAAtSSmtJFE063.png"></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">我见过一位分析师，他平时需要分析样本数据，虽然处理的数据量并不大，但有时仍会选用 Spark，而非 pandas 等这类传统的数据分析软件。我最开始也有些好奇，问其原因，他说是因为 Spark 接口简单还支持 SQL，这对于他处理数据非常方便。虽然是个例，但这也恰好说明 <strong>Spark</strong><strong> </strong><strong>的易用性。这一优点，不仅使之成为</strong><strong>工程师</strong><strong>等人群的必用工具，更得到了数据</strong><strong>分析师群体</strong><strong>的认可。</strong></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">简言之，Spark 为我们常见的批处理、流处理、数据分析、数据探索、机器学习等场景都提供了很好的解决方案，任何有数据处理需求的人，都可以用它来完成自己的研究与日常工作。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>关于</strong><strong> </strong><strong>Spark</strong><strong> </strong><strong>适合谁学，你不妨从</strong><strong> </strong><strong>Spark</strong><strong> </strong><strong>的</strong><strong>一些</strong><strong>使用场景</strong><strong>来对照体会一下</strong><strong>。</strong></span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>如果你是一名数据分析爱好者？</strong></span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">你可能有一些编程基础，对数字敏感，平时喜欢从公开数据中发现一些有趣的故事，你处理的数据量一般都不大，平时用 pandas、NumPy 也能很好地完成数据处理需求，但是因为听说 Spark 对 SQL 支持很好，很方便，现在想试试 Spark。</span></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>如果你是一名分析师?</strong></span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">你是大数据公司的一名量化分析师，对 R、Python 很熟悉，公司一般会有统一的大数据平台，数据和计算资源都对你开放，作为平台的用户，你的工作就是根据业务需求对全量数据进行探索、处理、分析和建模。你们处理的数据量有时会非常大，正好大数据平台支持 Spark，并且 Spark 也提供了 R 和 Python 编程接口，于是想试试 Spark，看能不能提升工作效率。</span></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>如果你是一名大数据工程师？</strong></span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">你原来是 Java 程序员，现在公司已经有了功能齐全的大数据平台，你们需要根据业务需求开发离线计算的批处理应用，还有实时计算的流处理应用，现在最成熟的工具就是 Spark，可以满足所有需求。</span></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>如果你是一名大数据架构师？</strong></span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">你经验丰富，需要为公司搭建一整套大数据平台，设计出支撑业务的数据管道，以满足公司内部分析师和工程师的使用需求，目前 Spark 生态可以很好地满足公司不同层次的数据处理需求，如离线计算、实时处理、数据挖掘等。</span></p> 
<h1><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">都说 Spark 难学，问题究竟出在哪儿？</span></p></h1> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">工作中，我发现很多人对 Spark 有一种天然的“距离感”，总是说“太难了”“更新太快了”，主要原因无外乎：</span></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">看 Spark 的官方文档，有很多新概念很抽象，例如弹性分布式数据集等；此外，Spark 在 2.0 的时候全面更新了一次，与之前的老版本差异很大。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">Spark 是一个分布式系统，与以前熟悉的技术有着本质的不同。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">因为动手能力较差，无法搭建（或者很难搭建）可以运行的 Spark 环境。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">虽然 Spark 图书不少（包括我自己写的《Spark 海量数据处理》），但基本上都是从原理出发，内容事无巨细，大而全，无形中拉升了学习成本。</span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">而且，跟着书中的讲述结构进行学习，还容易出现三个问题：</span></p> 
<ol> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">书上的代码跑不通或者看不懂，无法积累实践经验。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">在不重要且复杂的细节上耗费太多时间，不能针对当下所需高效学习，快速提升业务能力；</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">学习不高效，时间付出不对等，与实践欠缺导致的理解不够深刻，则更加抓不住重点，都容易让你陷入恶性循环。</span></p></li> 
</ol> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">这也是我开设这个专栏的初衷，希望给你一个实践与理论并重，同时又能够帮你抓住关键问题，快速学习提高的 Spark 课程，让你在碎片时间就能有效学习。</span></p> 
<h1><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">我是如何设计这门课程的？</span></p></h1> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">每个人的学习能力是不同的，学习 Spark 也一样，有的人上手很快，有的人却苦苦徘徊于门外。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">在这个专栏里，我特别针对不同背景和需要的用户做了优化，以便为你提供一个平缓的 Spark 学习曲线，这个课程：</span></p> 
<ul> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>简洁、实用：</strong>将复杂而不重要的细节去掉，仅保留必学的主干内容。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>突出实践，人人都能学，学了就能用</strong><strong>。</strong>除了一开始的 2 个模块，后面每个模块后都会有 1 个到 2 个实践案例来结尾。</span></p></li> 
 <li><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);"><strong>一个完整的实战项目</strong>，带你串联和巩固所学知识。专栏最后专门有一个“商业智能系统实战”模块，在这个模块里，带你用 Spark 完整地体验一个商业智能系统的开发流程。</span></p></li> 
</ul> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">基于此，本专栏分为 7 个模块，对于不同的读者类型，侧重点有所不同。对于数据分析爱好者和分析师这类用户来说，模块 2 需要了解，模块 4 可以忽略，其余需要掌握；对于大数据工程师来说，除了模块 6 可以根据需要进行学习，其余都需要掌握；对于大数据架构师来说，所有都必须掌握。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">下面是专栏的目录图，可以看到除了 Spark 基础知识，本课程还涵盖了当下流行的流处理、图挖掘、机器学习等内容，掌握这些必定可以让你在激烈的人才竞争中脱颖而出斩获心仪Offer，最后再以一个实践项目作为结尾，相信你能获得对 Spark 理性和感性的认知。</span></p> 
<p><br></p> 
<p style="line-height: 1.7; margin-bottom: 0pt; margin-top: 0pt; font-size: 11pt; color: rgb(73, 73, 73); text-align: center;"><img src="https://s0.lgstatic.com/i/image3/M01/0C/9B/Ciqah16OrniAUT5WAAZF5tlwc9A846.png"></p> 
<h1><p><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">写在最后</span></p></h1> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">无论大数据工程师，还是数据分析师，你都需要了解技术和业务，不过比重有些不同。工程师对业务理解到了一定程度，要想在技术深度和广度上有所建树，成长为架构师，那么阅读源码并形成自己的思考一定会大有裨益。当然，如果你能向 Spark 贡献代码并被接受，那就更好了，所以我在课程最后增加了篇“如何阅读 Spark 源码并向 Spark 贡献代码”的文章。而对于数据分析师，技术只是工具，结合业务探索数据本身蕴含的价值才是重点。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">点亮了 Spark 的技能，就像魔兽世界的满级，游戏才刚刚开始，希望你永远保持对技术、对数据的好奇心。在我当初开始大数据生涯时，投入了大量精力翻阅和学习各种资料，然后筛选和实践踩坑，那时我就在想，如果有一个能够帮我“先做减法再做加法”的专栏，该多好。很高兴的是，现在我可以来做这件事，在拉勾教育这个平台与你分享。</span></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><br></p> 
<p style="line-height: 1.7;margin-bottom: 0pt;margin-top: 0pt;font-size: 11pt;color: #494949;"><span style="font-size: 16px; font-family: 微软雅黑, &quot;Microsoft YaHei&quot;; color: rgb(63, 63, 63);">准备好了吗？让我们摩拳擦掌，一起开始这个学习与探讨的过程。为了更好地给你提供帮助，我也非常希望听到你的反馈，欢迎你在留言区给我留言，你也可以在这里和不同的用户交流经验，共同成长。</span></p>

---

### 精选评论

##### *强：
> 很好很赞，期待更新

##### **飞：
> 很好😀

##### **涵：
> 期待更新。厉害了

##### *刚：
> 很好，几乎涵盖了工作中能用到的所有场景，很不错，内容质量很高！

##### *宽：
> 这个课程会提供完整代码吗？

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 是的，有源码喔。

##### **文：
> 打卡打卡😀

##### **雪梨：
> 不错呀 期待

##### *含：
> 打卡start

 ###### &nbsp;&nbsp;&nbsp; 编辑回复：
> &nbsp;&nbsp;&nbsp; 加油～

##### **萍：
> 打卡 我又来学新工具了 哈哈哈

##### *艺：
> 开篇打卡，希望自己能坚持到最后

##### **辉：
> 学习了。

##### *凯：
> 打卡开篇

##### pawn：
> 感觉捡到宝了😄

##### **0727：
> 打卡开始

##### **超：
> wow，期待学完拿高薪😊

##### **娟：
> 老师，快点更新吧，非常期待了！！！

##### **青：
> 打卡。

##### **3202：
> 不敢相信这样的课程只卖1块钱！1块钱啊！

##### **强：
> 开篇很好，希望能有所收获😀😁

##### **忠：
> 坚持才是胜利，希望自己能脚踏实地，加油！

##### **杰：
> 期待中…

##### **玉：
> 打卡第一节

##### **超：
> 期待

##### **9966：
> 支持范老师😈

##### *平：
> 期待中

