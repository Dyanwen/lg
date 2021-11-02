<p data-nodeid="1577" class="">在介绍 Dockerfile 最佳实践前，这里再强调一下，<strong data-nodeid="1719">生产实践中一定优先使用 Dockerfile 的方式构建镜像。</strong> 因为使用 Dockerfile 构建镜像可以带来很多好处：</p>
<ul data-nodeid="1578">
<li data-nodeid="1579">
<p data-nodeid="1580">易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；</p>
</li>
<li data-nodeid="1581">
<p data-nodeid="1582">过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；</p>
</li>
<li data-nodeid="1583">
<p data-nodeid="1584">屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。</p>
</li>
</ul>
<p data-nodeid="1585">虽然有这么多好处，但是如果你 Dockerfile 使用不当也会引发很多问题。比如镜像构建时间过长，甚至镜像构建失败；镜像层数过多，导致镜像文件过大。所以，这一课时我就教你如何在生产环境中编写最优的 Dockerfile。</p>
<p data-nodeid="1586">在介绍 Dockerfile 最佳实践前，我们再聊一下我们平时书写 Dockerfile 应该尽量遵循的原则。</p>
<h3 data-nodeid="1587">Dockerfile 书写原则</h3>
<p data-nodeid="1588">遵循以下 Dockerfile 书写原则，不仅可以使得我们的 Dockerfile 简洁明了，让协作者清楚地了解镜像的完整构建流程，还可以帮助我们减少镜像的体积，加快镜像构建的速度和分发速度。</p>
<h4 data-nodeid="1589">（1）单一职责</h4>
<p data-nodeid="1590">由于容器的本质是进程，一个容器代表一个进程，因此不同功能的应用应该尽量拆分为不同的容器，每个容器只负责单一业务进程。</p>
<h4 data-nodeid="1591">（2）提供注释信息</h4>
<p data-nodeid="1592">Dockerfile 也是一种代码，我们应该保持良好的代码编写习惯，晦涩难懂的代码尽量添加注释，让协作者可以一目了然地知道每一行代码的作用，并且方便扩展和使用。</p>
<h4 data-nodeid="1593">（3）保持容器最小化</h4>
<p data-nodeid="1594">应该避免安装无用的软件包，比如在一个 nginx 镜像中，我并不需要安装 vim 、gcc 等开发编译工具。这样不仅可以加快容器构建速度，而且可以避免镜像体积过大。</p>
<h4 data-nodeid="1595">（4）合理选择基础镜像</h4>
<p data-nodeid="1596">容器的核心是应用，因此只要基础镜像能够满足应用的运行环境即可。例如一个<code data-backticks="1" data-nodeid="1735">Java</code>类型的应用运行时只需要<code data-backticks="1" data-nodeid="1737">JRE</code>，并不需要<code data-backticks="1" data-nodeid="1739">JDK</code>，因此我们的基础镜像只需要安装<code data-backticks="1" data-nodeid="1741">JRE</code>环境即可。</p>
<h4 data-nodeid="1597">（5）使用 .dockerignore 文件</h4>
<p data-nodeid="1598">在使用<code data-backticks="1" data-nodeid="1745">git</code>时，我们可以使用<code data-backticks="1" data-nodeid="1747">.gitignore</code>文件忽略一些不需要做版本管理的文件。同理，使用<code data-backticks="1" data-nodeid="1749">.dockerignore</code>文件允许我们在构建时，忽略一些不需要参与构建的文件，从而提升构建效率。<code data-backticks="1" data-nodeid="1751">.dockerignore</code>的定义类似于<code data-backticks="1" data-nodeid="1753">.gitignore</code>。</p>
<p data-nodeid="1599"><code data-backticks="1" data-nodeid="1755">.dockerignore</code>的本质是文本文件，Docker 构建时可以使用换行符来解析文件定义，每一行可以忽略一些文件或者文件夹。具体使用方式如下：</p>
<table data-nodeid="1601">
<thead data-nodeid="1602">
<tr data-nodeid="1603">
<th data-org-content="规则" data-nodeid="1605">规则</th>
<th data-org-content="含义" data-nodeid="1606">含义</th>
</tr>
</thead>
<tbody data-nodeid="1609">
<tr data-nodeid="1610">
<td data-org-content="#" data-nodeid="1611">#</td>
<td data-org-content="\# 开头的表示注释，\# 后面所有内容将会被忽略" data-nodeid="1612"># 开头的表示注释，# 后面所有内容将会被忽略</td>
</tr>
<tr data-nodeid="1613">
<td data-org-content="*/tmp*" data-nodeid="1614"><em data-nodeid="1767">/tmp</em></td>
<td data-org-content="匹配当前目录下任何以 tmp 开头的文件或者文件夹" data-nodeid="1615">匹配当前目录下任何以 tmp 开头的文件或者文件夹</td>
</tr>
<tr data-nodeid="1616">
<td data-org-content="\*.md" data-nodeid="1617">*.md</td>
<td data-org-content="匹配以 .md 为后缀的任意文件" data-nodeid="1618">匹配以 .md 为后缀的任意文件</td>
</tr>
<tr data-nodeid="1619">
<td data-org-content="tem?" data-nodeid="1620">tem?</td>
<td data-org-content="匹配以 tem 开头并且以任意字符结尾的文件，？代表任意一个字符" data-nodeid="1621">匹配以 tem 开头并且以任意字符结尾的文件，？代表任意一个字符</td>
</tr>
<tr data-nodeid="1622">
<td data-org-content="!README.md" data-nodeid="1623">!README.md</td>
<td data-org-content="! 表示排除忽略。<br>例如 .dockerignore 定义如下：<br><br>\*.md<br>!README.md<br><br>表示除了 README.md 文件外所有以 .md 结尾的文件。" data-nodeid="1624">! 表示排除忽略。<br>例如 .dockerignore 定义如下：<br><br>*.md<br>!README.md<br><br>表示除了 README.md 文件外所有以 .md 结尾的文件。</td>
</tr>
</tbody>
</table>
<h4 data-nodeid="1625">（6）尽量使用构建缓存</h4>
<p data-nodeid="1626">Docker 构建过程中，每一条 Dockerfile 指令都会提交为一个镜像层，下一条指令都是基于上一条指令构建的。如果构建时发现要构建的镜像层的父镜像层已经存在，并且下一条命令使用了相同的指令，即可命中构建缓存。</p>
<p data-nodeid="1627">Docker 构建时判断是否需要使用缓存的规则如下：</p>
<ul data-nodeid="1628">
<li data-nodeid="1629">
<p data-nodeid="1630">从当前构建层开始，比较所有的子镜像，检查所有的构建指令是否与当前完全一致，如果不一致，则不使用缓存；</p>
</li>
<li data-nodeid="1631">
<p data-nodeid="1632">一般情况下，只需要比较构建指令即可判断是否需要使用缓存，但是有些指令除外（例如<code data-backticks="1" data-nodeid="1795">ADD</code>和<code data-backticks="1" data-nodeid="1797">COPY</code>）；</p>
</li>
<li data-nodeid="1633">
<p data-nodeid="1634">对于<code data-backticks="1" data-nodeid="1800">ADD</code>和<code data-backticks="1" data-nodeid="1802">COPY</code>指令不仅要校验命令是否一致，还要为即将拷贝到容器的文件计算校验和（根据文件内容计算出的一个数值，如果两个文件计算的数值一致，表示两个文件内容一致 ），命令和校验和完全一致，才认为命中缓存。</p>
</li>
</ul>
<p data-nodeid="1635">因此，基于 Docker 构建时的缓存特性，我们可以把不轻易改变的指令放到 Dockerfile 前面（例如安装软件包），而可能经常发生改变的指令放在 Dockerfile 末尾（例如编译应用程序）。</p>
<p data-nodeid="1636">例如，我们想要定义一些环境变量并且安装一些软件包，可以按照如下顺序编写 Dockerfile：</p>
<pre class="lang-dart" data-nodeid="1637"><code data-language="dart">FROM centos:<span class="hljs-number">7</span>
# 设置环境变量指令放前面
ENV PATH /usr/local/bin:$PATH
# 安装软件指令放前面
RUN yum install -y make
# 把业务软件的配置,版本等经常变动的步骤放最后
...
</code></pre>
<p data-nodeid="1638">按照上面原则编写的 Dockerfile 在构建镜像时，前面步骤命中缓存的概率会增加，可以大大缩短镜像构建时间。</p>
<h4 data-nodeid="1639">（7）正确设置时区</h4>
<p data-nodeid="1640">我们从 Docker Hub 拉取的官方操作系统镜像大多数都是 UTC 时间（世界标准时间）。如果你想要在容器中使用中国区标准时间（东八区），请根据使用的操作系统修改相应的时区信息，下面我介绍几种常用操作系统的修改方式：</p>
<ul data-nodeid="1641">
<li data-nodeid="1642">
<p data-nodeid="1643"><strong data-nodeid="1812">Ubuntu 和Debian 系统</strong></p>
</li>
</ul>
<p data-nodeid="1644">Ubuntu 和Debian 系统可以向 Dockerfile 中添加以下指令：</p>
<pre class="lang-dockerfile" data-nodeid="1645"><code data-language="dockerfile"><span class="hljs-keyword">RUN</span><span class="bash"> ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime</span>
<span class="hljs-keyword">RUN</span><span class="bash"> <span class="hljs-built_in">echo</span> <span class="hljs-string">"Asia/Shanghai"</span> &gt;&gt; /etc/timezone</span>
</code></pre>
<ul data-nodeid="1646">
<li data-nodeid="1647">
<p data-nodeid="1648"><strong data-nodeid="1817">CentOS系统</strong></p>
</li>
</ul>
<p data-nodeid="1649">CentOS 系统则向 Dockerfile 中添加以下指令：</p>
<pre class="lang-dockerfile" data-nodeid="1650"><code data-language="dockerfile"><span class="hljs-keyword">RUN</span><span class="bash"> ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime</span>
</code></pre>
<h4 data-nodeid="1651">（8）使用国内软件源加快镜像构建速度</h4>
<p data-nodeid="1652">由于我们常用的官方操作系统镜像基本都是国外的，软件服务器大部分也在国外，所以我们构建镜像的时候想要安装一些软件包可能会非常慢。</p>
<p data-nodeid="1653">这里我以 CentOS 7 为例，介绍一下如何使用 163 软件源（国内有很多大厂，例如阿里、腾讯、网易等公司都免费提供的软件加速源）加快镜像构建。</p>
<p data-nodeid="1654">首先在容器构建目录创建文件 CentOS7-Base-163.repo，文件内容如下：</p>
<pre class="lang-shell" data-nodeid="1655"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> CentOS-Base.repo</span>
<span class="hljs-meta">#</span>
<span class="hljs-meta">#</span><span class="bash"> The mirror system uses the connecting IP address of the client and the</span>
<span class="hljs-meta">#</span><span class="bash"> update status of each mirror to pick mirrors that are updated to and</span>
<span class="hljs-meta">#</span><span class="bash"> geographically close to the client.  You should use this <span class="hljs-keyword">for</span> CentOS updates</span>
<span class="hljs-meta">#</span><span class="bash"> unless you are manually picking other mirrors.</span>
<span class="hljs-meta">#</span>
<span class="hljs-meta">#</span><span class="bash"> If the mirrorlist= does not work <span class="hljs-keyword">for</span> you, as a fall back you can try the </span>
<span class="hljs-meta">#</span><span class="bash"> remarked out baseurl= line instead.</span>
<span class="hljs-meta">#</span>
<span class="hljs-meta">#</span>
[base]
name=CentOS-$releasever - Base - 163.com
<span class="hljs-meta">#</span><span class="bash">mirrorlist=http://mirrorlist.centos.org/?release=<span class="hljs-variable">$releasever</span>&amp;arch=<span class="hljs-variable">$basearch</span>&amp;repo=os</span>
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
<span class="hljs-meta">#</span><span class="bash">released updates</span>
[updates]
name=CentOS-$releasever - Updates - 163.com
<span class="hljs-meta">#</span><span class="bash">mirrorlist=http://mirrorlist.centos.org/?release=<span class="hljs-variable">$releasever</span>&amp;arch=<span class="hljs-variable">$basearch</span>&amp;repo=updates</span>
baseurl=http://mirrors.163.com/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
<span class="hljs-meta">#</span><span class="bash">additional packages that may be useful</span>
[extras]
name=CentOS-$releasever - Extras - 163.com
<span class="hljs-meta">#</span><span class="bash">mirrorlist=http://mirrorlist.centos.org/?release=<span class="hljs-variable">$releasever</span>&amp;arch=<span class="hljs-variable">$basearch</span>&amp;repo=extras</span>
baseurl=http://mirrors.163.com/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
<span class="hljs-meta">#</span><span class="bash">additional packages that extend functionality of existing packages</span>
[centosplus]
name=CentOS-$releasever - Plus - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/centosplus/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7
</code></pre>
<p data-nodeid="1656">然后在 Dockerfile 中添加如下指令：</p>
<pre class="lang-java" data-nodeid="1657"><code data-language="java">COPY CentOS7-Base-<span class="hljs-number">163.</span>repo /etc/yum.repos.d/CentOS7-Base.repo
</code></pre>
<p data-nodeid="1658">执行完上述步骤后，再使用<code data-backticks="1" data-nodeid="1825">yum install</code>命令安装软件时就会默认从 163 获取软件包，这样可以大大提升构建速度。</p>
<h4 data-nodeid="1659">（9）最小化镜像层数</h4>
<p data-nodeid="1660">在构建镜像时尽可能地减少 Dockerfile 指令行数。例如我们要在 CentOS 系统中安装<code data-backticks="1" data-nodeid="1829">make</code>和<code data-backticks="1" data-nodeid="1831">net-tools</code>两个软件包，应该在 Dockerfile 中使用以下指令：</p>
<pre class="lang-java" data-nodeid="1661"><code data-language="java">RUN yum install -y make net-tools
</code></pre>
<p data-nodeid="1662">而不应该写成这样：</p>
<pre class="lang-java" data-nodeid="1663"><code data-language="java">RUN yum install -y make
RUN yum install -y net-tools
</code></pre>
<p data-nodeid="1664">了解完 Dockerfile 的书写原则后，我们再来具体了解下这些原则落实到具体的 Dockerfile 指令应该如何书写。</p>
<h3 data-nodeid="1665">Dockerfile 指令书写建议</h3>
<p data-nodeid="1666">下面是我们常用的一些指令，这些指令对于刚接触 Docker 的人来说会非常容易出错，下面我对这些指令的书写建议详细讲解一下。</p>
<h4 data-nodeid="1667">（1）RUN</h4>
<p data-nodeid="1668"><code data-backticks="1" data-nodeid="1838">RUN</code>指令在构建时将会生成一个新的镜像层并且执行<code data-backticks="1" data-nodeid="1840">RUN</code>指令后面的内容。</p>
<p data-nodeid="1669">使用<code data-backticks="1" data-nodeid="1843">RUN</code>指令时应该尽量遵循以下原则：</p>
<ul data-nodeid="1670">
<li data-nodeid="1671">
<p data-nodeid="1672">当<code data-backticks="1" data-nodeid="1846">RUN</code>指令后面跟的内容比较复杂时，建议使用反斜杠（\） 结尾并且换行；</p>
</li>
<li data-nodeid="1673">
<p data-nodeid="1674"><code data-backticks="1" data-nodeid="1850">RUN</code>指令后面的内容尽量按照字母顺序排序，提高可读性。</p>
</li>
</ul>
<p data-nodeid="1675">例如，我想在官方的 CentOS 镜像下安装一些软件，一个建议的 Dockerfile 指令如下：</p>
<pre class="lang-dockerfile" data-nodeid="1676"><code data-language="dockerfile"><span class="hljs-keyword">FROM</span> centos:<span class="hljs-number">7</span>
<span class="hljs-keyword">RUN</span><span class="bash"> yum install -y automake \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;curl \
                   python \
&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;vim</span>
</code></pre>
<h4 data-nodeid="1677">（2）CMD 和 ENTRYPOINT</h4>
<p data-nodeid="1678"><code data-backticks="1" data-nodeid="1854">CMD</code>和<code data-backticks="1" data-nodeid="1856">ENTRYPOINT</code>指令都是容器运行的命令入口，这两个指令使用中有很多相似的地方，但是也有一些区别。</p>
<p data-nodeid="1679">这两个指令的相同之处，<code data-backticks="1" data-nodeid="1859">CMD</code>和<code data-backticks="1" data-nodeid="1861">ENTRYPOINT</code>的基本使用格式分为两种。</p>
<ul data-nodeid="5330">
<li data-nodeid="5331">
<p data-nodeid="5332">第一种为<code data-backticks="1" data-nodeid="5339">CMD</code>/<code data-backticks="1" data-nodeid="5341">ENTRYPOINT</code>["command" , "param"]。这种格式是使用 Linux 的<code data-backticks="1" data-nodeid="5352">exec</code>实现的， 一般称为<code data-backticks="1" data-nodeid="5354">exec</code>模式，这种书写格式为<code data-backticks="1" data-nodeid="5356">CMD</code>/<code data-backticks="1" data-nodeid="5358">ENTRYPOINT</code>后面跟 json 数组，也是Docker 推荐的使用格式。</p>
</li>
<li data-nodeid="5333">
<p data-nodeid="5334">另外一种格式为<code data-backticks="1" data-nodeid="5361">CMD</code>/<code data-backticks="1" data-nodeid="5363">ENTRYPOINT</code>command param ，这种格式是基于 shell 实现的， 通常称为<code data-backticks="1" data-nodeid="5365">shell</code>模式。当使用<code data-backticks="1" data-nodeid="5367">shell</code>模式时，Docker 会以 /bin/sh -c command 的方式执行命令。</p>
</li>
</ul>
<blockquote data-nodeid="5335" class="">
<p data-nodeid="5336">使用 exec 模式启动容器时，容器的 1 号进程就是 CMD/ENTRYPOINT 中指定的命令，而使用 shell 模式启动容器时相当于我们把启动命令放在了 shell 进程中执行，等效于执行  /bin/sh -c "task command" 命令。因此 shell 模式启动的进程在容器中实际上并不是 1 号进程。</p>
</blockquote>
<p data-nodeid="5337">这两个指令的区别：</p>






<ul data-nodeid="1686">
<li data-nodeid="1687">
<p data-nodeid="1688">Dockerfile 中如果使用了<code data-backticks="1" data-nodeid="1896">ENTRYPOINT</code>指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的<code data-backticks="1" data-nodeid="1898">ENTRYPOINT</code>指令 ，而使用<code data-backticks="1" data-nodeid="1900">CMD</code>设置的命令则可以被<code data-backticks="1" data-nodeid="1902">docker run</code>后面的参数直接覆盖。</p>
</li>
<li data-nodeid="1689">
<p data-nodeid="1690"><code data-backticks="1" data-nodeid="1904">ENTRYPOINT</code>指令可以结合<code data-backticks="1" data-nodeid="1906">CMD</code>指令使用，也可以单独使用，而<code data-backticks="1" data-nodeid="1908">CMD</code>指令只能单独使用。</p>
</li>
</ul>
<p data-nodeid="1691">看到这里你也许会问，我什么时候应该使用<code data-backticks="1" data-nodeid="1911">ENTRYPOINT</code>,什么时候使用<code data-backticks="1" data-nodeid="1913">CMD</code>呢？</p>
<p data-nodeid="1692">如果你希望你的镜像足够灵活，推荐使用<code data-backticks="1" data-nodeid="1916">CMD</code>指令。如果你的镜像只执行单一的具体程序，并且不希望用户在执行<code data-backticks="1" data-nodeid="1918">docker run</code>时覆盖默认程序，建议使用<code data-backticks="1" data-nodeid="1920">ENTRYPOINT</code>。</p>
<p data-nodeid="1693">最后再强调一下，无论使用<code data-backticks="1" data-nodeid="1923">CMD</code>还是<code data-backticks="1" data-nodeid="1925">ENTRYPOINT</code>，都尽量使用<code data-backticks="1" data-nodeid="1927">exec</code>模式。</p>
<h4 data-nodeid="1694">（3）ADD 和 COPY</h4>
<p data-nodeid="1695"><code data-backticks="1" data-nodeid="1930">ADD</code>和<code data-backticks="1" data-nodeid="1932">COPY</code>指令功能类似，都是从外部往容器内添加文件。但是<code data-backticks="1" data-nodeid="1934">COPY</code>指令只支持基本的文件和文件夹拷贝功能，<code data-backticks="1" data-nodeid="1936">ADD</code>则支持更多文件来源类型，比如自动提取 tar 包，并且可以支持源文件为 URL 格式。</p>
<p data-nodeid="1696">那么在日常应用中，我们应该使用哪个命令向容器里添加文件呢？你可能在想，既然<code data-backticks="1" data-nodeid="1939">ADD</code>指令支持的功能更多，当然应该使用<code data-backticks="1" data-nodeid="1941">ADD</code>指令了。然而事实恰恰相反，我更推荐你使用<code data-backticks="1" data-nodeid="1943">COPY</code>指令，因为<code data-backticks="1" data-nodeid="1945">COPY</code>指令更加透明，仅支持本地文件向容器拷贝，而且使用<code data-backticks="1" data-nodeid="1947">COPY</code>指令可以更好地利用构建缓存，有效减小镜像体积。</p>
<p data-nodeid="1697">当你想要使用<code data-backticks="1" data-nodeid="1950">ADD</code>向容器中添加 URL 文件时，请尽量考虑使用其他方式替代。例如你想要在容器中安装 memtester（一种内存压测工具），你应该避免使用以下格式：</p>
<pre class="lang-dockerfile" data-nodeid="1698"><code data-language="dockerfile"><span class="hljs-keyword">ADD</span><span class="bash"> http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz /tmp/</span>
<span class="hljs-keyword">RUN</span><span class="bash"> tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp</span>
<span class="hljs-keyword">RUN</span><span class="bash"> make -C /tmp/memtester-4.3.0 &amp;&amp; make -C /tmp/memtester-4.3.0 install</span>
</code></pre>
<p data-nodeid="1699">下面是推荐写法：</p>
<pre class="lang-dockerfile" data-nodeid="1700"><code data-language="dockerfile"><span class="hljs-keyword">RUN</span><span class="bash"> wget -O /tmp/memtester-4.3.0.tar.gz http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz \
&amp;&amp; tar -xvf /tmp/memtester-4.3.0.tar.gz -C /tmp \
&amp;&amp; make -C /tmp/memtester-4.3.0 &amp;&amp; make -C /tmp/memtester-4.3.0 install</span>
</code></pre>
<h4 data-nodeid="1701">（4）WORKDIR</h4>
<p data-nodeid="1702">为了使构建过程更加清晰明了，推荐使用 WORKDIR 来指定容器的工作路径，应该尽量避免使用 RUN cd /work/path &amp;&amp; do some work 这样的指令。</p>
<p data-nodeid="1703">最后给出几个常用软件的官方 Dockerfile 示例链接，希望可以对你有所帮助。</p>
<ul data-nodeid="1704">
<li data-nodeid="1705">
<p data-nodeid="1706"><a href="https://github.com/docker-library/golang/blob/4d68c4dd8b51f83ce4fdce0f62484fdc1315bfa8/1.15/buster/Dockerfile" data-nodeid="1961">Go</a></p>
</li>
<li data-nodeid="1707">
<p data-nodeid="1708"><a href="https://github.com/nginxinc/docker-nginx/blob/9774b522d4661effea57a1fbf64c883e699ac3ec/mainline/buster/Dockerfile" data-nodeid="1964">Nginx</a></p>
</li>
<li data-nodeid="1709">
<p data-nodeid="1710"><a href="https://github.com/hylang/docker-hylang/blob/f9c873b7f71f466e5af5ea666ed0f8f42835c688/dockerfiles-generated/Dockerfile.python3.8-buster" data-nodeid="1967">Hy</a></p>
</li>
</ul>
<h3 data-nodeid="1711">结语</h3>
<p data-nodeid="1712">好了，到此为止，相信你已经对 Dockerfile 的书写原则和一些重要指令有了较深的认识。</p>
<p data-nodeid="1713" class="">当你需要编写编译型语言（例如 Golang、Java）的 Dockerfile 时，如何分离编译环境和运行环境，使得镜像体积尽可能小呢？思考后，可以把你的想法写在留言区。</p>

---

### 精选评论

##### Ben.Zhong：
> 把 编译环境 单独打包镜像，只提供编译好的二进制（注意运行 CPU 架构）。运行环境单独做镜像，只考虑基础运行环境的配置。好处还是很明显，程序发布单独版本管理，运行环境也单独组合。还有好处就是，运行环境打补丁升级等等，不用影响程序发布。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 是的，这位同学理解很到位，分离编译环境和运行环境对于生产环境很重要

##### 潘：
> 这个docker学习通俗易懂

##### *庚：
> 为什么推荐exec模式

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; exec 可以保证我们的业务进程就是 1 号进程，这对于需要处理 SIGTERM 信号量实现优雅终止十分重要。

##### **的蜗牛：
> 「尽量使用构建缓存」放在后面命中缓存的概率是一样的，构建镜像的整体时间原则上也一样。所以对这个原则不是很理解？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 简单来说就是如果第n层有改动，则n层以后的缓存都会失效，⼤多数情况下判断有⽆改动的⽅法是判断这层的指令和缓存中的构建 指令是否⼀致，但是对于COPY和ADD命令会计算镜像内的⽂件和构建⽬录⽂件的校验和然后做⽐较来判断本层是否有改动。

##### *朋：
> docker-compose. yml中，service下配置了volumn，在全局中也有volumn的声明，这个声明有什么用

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 全局中声明 volumns，在 service 中使用，这样设计是为了方便多个 service 中的服务共享相同的 volumn

##### kopelan：
> hi，dockerfile中的ENV是否也是每局一层呢？比如我要配置多个ENV环境变量，我应该怎么写？另外我看教程中ENV 有写作ENV key value，也有写作ENV key=value，这个有什么区别呢？

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 定义多个 ENV 直接多行写就行,基本不会占用额外空间, 一般推荐写成 ENV key=value 的形式

##### **斯：
> 要尽量减小,镜像的体积！ 为什么基础底层镜像不使用alpine呢

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 生产中如果公司镜像统一,底层镜像尽量使用工具较多的镜像, alpine 虽然小,但是功能相对也少,而底层镜像只需要拉取一次.

##### **佳：
> 老师，请教一个Linux的问题。为什么以下两个命令输出的结果不一样呢？我想这也是要尽量用exec的原因是吗？echo HOME=$HOMEsh -c echo">HOME=$HOME

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 第一个命令相当于在容器主进程下执行命令，第二个命令是先创建一个 sh 进程，然后再执行相关的命令

##### **博：
> 不太懂编译环境和运行环境，比如我现在的dockerFile，jar包直接运行了FROM openjdk:8-jre-alpineCOPY . /usr/src/myappWORKDIR /usr/src/myappENTRYPOINT ["java", "-jar", "channeladminservice.jar"]CMD ["--spring.profiles.active=cloud-smoke"]

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 编译环境是指编译运行程序所需要的环境，运行环境即我们的应用程序想要正常运行所依赖的环境。例如我们编译 Java 应用程序需要 JDK 环境，而真正运行我们的 Java 程序则仅需要 JRE 环境即可。

##### **升：
> 真的很赞啊，写得太好了

##### *峰：
> 能否分享下java栈最优dockfile

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 这个需要根据不同的业务场景来制定的，推荐使用多阶段编译的方式，将编译环境和运行环境分开，然后遵循第6讲 Dockerfile 最佳实践中的原则编写 Dockerfile 即可

##### **1156：
> CMD ["nginx", "-g", "daemon off;"]这个-g和daemon off是什么意思

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 就是让 nginx 以前台的方式启动，启动后不退出当前窗口

