<p data-nodeid="2017" class=""><strong data-nodeid="2172">我看到过这样一道面试题：请简述 Linux 权限划分的原则</strong>？</p>
<p data-nodeid="2018">这种类型的面试题也是我比较喜欢的一种题目，因为它考察的不仅是一个具体的指令，还考察了候选人技术层面的认知。</p>
<p data-nodeid="2019">如果你对 Linux 权限有较深的认知和理解，那么完全可以通过查资料去完成具体指令的执行。更重要的是，认知清晰的程序员可以把 Linux 权限管理的知识迁移到其他的系统设计中。而且我认为，能够对某个技术形成认知的人， 同样也会热爱思考，善于总结，这样的程序员是所有团队梦寐以求的。</p>
<p data-nodeid="2020">因此，这次我们就把这道面试题作为引子，开启今天的学习。</p>
<h3 data-nodeid="2021">权限抽象</h3>
<p data-nodeid="2022">一个完整的权限管理体系，要有合理的抽象。这里就包括对用户、进程、文件、内存、系统调用等抽象。下面我将带你一一了解。</p>
<p data-nodeid="2023"><strong data-nodeid="2182">首先，我们先来说说用户和组</strong>。Linux 是一个多用户平台，允许多个用户同时登录系统工作。Linux 将用户抽象成了账户，账户可以登录系统，比如通过输入登录名 + 密码的方式登录；也可以通过证书的方式登录。</p>
<p data-nodeid="2024">但为了方便分配每个用户的权限，Linux 还支持组 <strong data-nodeid="2188">（Group）账户</strong>。组账户是多个账户的集合，组可以为成员们分配某一类权限。每个用户可以在多个组，这样就可以利用组给用户快速分配权限。</p>
<p data-nodeid="2025">组的概念有点像微信群。一个用户可以在多个群中。比如某个组中分配了 10 个目录的权限，那么新建用户的时候可以将这个用户增加到这个组中，这样新增的用户就不必再去一个个目录分配权限。</p>
<p data-nodeid="2026">而每一个微信群都有一个群主，<strong data-nodeid="2195">Root 账户也叫作超级管理员</strong>，就相当于微信群主，它对系统有着完全的掌控。一个超级管理员可以使用系统提供的全部能力。</p>
<p data-nodeid="2027">此外，Linux 还对<strong data-nodeid="2205">文件</strong>进行了权限抽象（<strong data-nodeid="2206">注意目录也是一种文件</strong>）。Linux 中一个文件可以设置下面 3 种权限：</p>
<ol data-nodeid="2028">
<li data-nodeid="2029">
<p data-nodeid="2030">读权限（r）：控制读取文件。</p>
</li>
<li data-nodeid="2031">
<p data-nodeid="2032">写权限（w）：控制写入文件。</p>
</li>
<li data-nodeid="2033">
<p data-nodeid="2034">执行权限（x）：控制将文件执行，比如脚本、应用程序等。</p>
</li>
</ol>
<p data-nodeid="2035"><img src="https://s0.lgstatic.com/i/image/M00/5A/48/Ciqc1F91G6qACantAAC4GIUeips460.png" alt="1.png" data-nodeid="2212"></p>
<p data-nodeid="2036">然后每个文件又可以从 3 个维度去配置上述的 3 种权限：</p>
<ol data-nodeid="2037">
<li data-nodeid="2038">
<p data-nodeid="2039">用户维度。每个文件可以所属 1 个用户，用户维度配置的 rwx 在用户维度生效；</p>
</li>
<li data-nodeid="2040">
<p data-nodeid="2041">组维度。每个文件可以所属 1 个分组，组维度配置的 rwx 在组维度生效；</p>
</li>
<li data-nodeid="2042">
<p data-nodeid="2043">全部用户维度。设置对所有用户的权限。</p>
</li>
</ol>
<p data-nodeid="2044"><img src="https://s0.lgstatic.com/i/image/M00/5A/53/CgqCHl91G9aADTBZAADD7IOpjac809.png" alt="2.png" data-nodeid="2219"></p>
<p data-nodeid="2045">因此 Linux 中文件的权限可以用 9 个字符，3 组<code data-backticks="1" data-nodeid="2221">rwx</code>描述：第一组是用户权限，第二组是组权限，第三组是所有用户的权限。然后用<code data-backticks="1" data-nodeid="2223">-</code>代表没有权限。比如<code data-backticks="1" data-nodeid="2225">rwxrwxrwx</code>代表所有维度可以读写执行。<code data-backticks="1" data-nodeid="2227">rw--wxr-x</code>代表用户维度不可以执行，组维度不可以读取，所有用户维度不可以写入。</p>
<p data-nodeid="2046">通常情况下，如果用<code data-backticks="1" data-nodeid="2230">ls -l</code>查看一个文件的权限，会有 10 个字符，这是因为第一个字符代表的是文件类型。我们在 06 课时讲解“几种常见的文件类型”时提到过，有管道文件、目录文件、链接文件等等。<code data-backticks="1" data-nodeid="2232">-</code>代表普通文件、<code data-backticks="1" data-nodeid="2234">d</code>代表目录、<code data-backticks="1" data-nodeid="2236">p</code>代表管道。</p>
<p data-nodeid="2047"><strong data-nodeid="2242">学习了这套机制之后，请你跟着我的节奏一起思考以下 4 个问题</strong>。</p>
<ol data-nodeid="2048">
<li data-nodeid="2049">
<p data-nodeid="2050">文件被创建后，初始的权限如何设置？</p>
</li>
<li data-nodeid="2051">
<p data-nodeid="2052">需要全部用户都可以执行的指令，比如<code data-backticks="1" data-nodeid="2245">ls</code>，它们的权限如何分配？</p>
</li>
<li data-nodeid="2053">
<p data-nodeid="2054">给一个文本文件分配了可执行权限会怎么样？</p>
</li>
<li data-nodeid="2055">
<p data-nodeid="2056">可不可以多个用户都登录<code data-backticks="1" data-nodeid="2249">root</code>，然后只用<code data-backticks="1" data-nodeid="2251">root</code>账户？</p>
</li>
</ol>
<p data-nodeid="2057">你可以把以上 4 个问题作为本课时的小测验，把你的思考或者答案写在留言区，然后再来看我接下来的分析。</p>
<p data-nodeid="2058"><strong data-nodeid="2257">问题一：初始权限问题</strong></p>
<p data-nodeid="2059">一个文件创建后，文件的所属用户会被设置成创建文件的用户。谁创建谁拥有，这个逻辑很顺理成章。但是文件的组又是如何分配的呢？</p>
<p data-nodeid="2060">这里 Linux 想到了一个很好的办法，就是为每个用户创建一个同名分组。</p>
<p data-nodeid="2061">比如说<code data-backticks="1" data-nodeid="2261">zhang</code>这个账户创建时，会创建一个叫作<code data-backticks="1" data-nodeid="2263">zhang</code>的分组。<code data-backticks="1" data-nodeid="2265">zhang</code>登录之后，工作分组就会默认使用它的同名分组<code data-backticks="1" data-nodeid="2267">zhang</code>。如果<code data-backticks="1" data-nodeid="2269">zhang</code>想要切换工作分组，可以使用<code data-backticks="1" data-nodeid="2271">newgrp</code>指令切换到另一个工作分组。因此，被创建文件所属的分组是当时用户所在的工作分组，如果没有特别设置，那么就属于用户所在的同名分组。</p>
<p data-nodeid="2062">再说下文件的权限如何？文件被创建后的权限通常是：</p>
<pre class="lang-java" data-nodeid="2063"><code data-language="java">rw-rw-r--
</code></pre>
<p data-nodeid="2064">也就是用户、组维度不可以执行，所有用户可读。</p>
<p data-nodeid="2065"><strong data-nodeid="2278">问题二：公共执行文件的权限</strong></p>
<p data-nodeid="2066">前面提到过可以用<code data-backticks="1" data-nodeid="2280">which</code>指令查看<code data-backticks="1" data-nodeid="2282">ls</code>指令所在的目录，我们发现在<code data-backticks="1" data-nodeid="2284">/usr/bin</code>中。然后用<code data-backticks="1" data-nodeid="2286">ls -l</code>查看<code data-backticks="1" data-nodeid="2288">ls</code>的权限，可以看到下图所示：</p>
<p data-nodeid="2067"><img src="https://s0.lgstatic.com/i/image/M00/5A/33/Ciqc1F90SRuAAQCEAADdVOthCFw679.png" alt="Drawing 2.png" data-nodeid="2292"></p>
<ul data-nodeid="48711">
<li data-nodeid="48712">
<p data-nodeid="48713">第一个<code data-backticks="1" data-nodeid="48721">-</code>代表这是一个普通文件，后面的 rwx 代表用户维度可读写和执行；</p>
</li>
<li data-nodeid="48714">
<p data-nodeid="48715" class="">第二个<code data-backticks="1" data-nodeid="48724">r-x</code>代表组维度不可读写；</p>
</li>
<li data-nodeid="48716">
<p data-nodeid="48717">第三个<code data-backticks="1" data-nodeid="48727">r-x</code>代表所有用户可以读和执行；</p>
</li>
<li data-nodeid="48718">
<p data-nodeid="48719">后两个<code data-backticks="1" data-nodeid="48730">root</code>，第一个代表所属用户，第二个代表所属分组。</p>
</li>
</ul>













































<p data-nodeid="2069">到这里你可能会有一个疑问：如果一个文件设置为不可读，但是可以执行，那么结果会怎样？</p>
<p data-nodeid="2070">答案当然是不可以执行，无法读取文件内容自然不可以执行。</p>
<p data-nodeid="2071"><strong data-nodeid="2307">问题三：执行文件</strong></p>
<p data-nodeid="58984" class="">在 Linux 中，如果一个文件可以被执行，则可以直接通过输入文件路径（相对路径或绝对路径）的方式执行。如果想执行一个不可以执行的文件，Linux 则会报错。</p>










<p data-nodeid="2073">当用户输入一个文件名，如果没有指定完整路径，Linux 就会在一部分目录中查找这个文件。你可以通过<code data-backticks="1" data-nodeid="2315">echo $PATH</code>看到 Linux 会在哪些目录中查找可执行文件，<code data-backticks="1" data-nodeid="2317">PATH</code>是 Linux 的环境变量，关于环境变量，我将在 “12 | 高级技巧之集群部署中”和你详细讨论。</p>
<p data-nodeid="2074"><img src="https://s0.lgstatic.com/i/image/M00/5A/3F/CgqCHl90SSSACa4WAAFIEUypWH4904.png" alt="Drawing 3.png" data-nodeid="2323"></p>
<p data-nodeid="2075"><strong data-nodeid="2327">问题四：可不可以都 root</strong></p>
<p data-nodeid="2076">最后一个问题是，可不可以都<code data-backticks="1" data-nodeid="2329">root</code>？</p>
<p data-nodeid="2077">答案当然是不行！这里先给你留个悬念，具体原因我们会在本课时最后来讨论。</p>
<p data-nodeid="2078"><strong data-nodeid="2338">到这里，用户和组相关权限就介绍完了。接下来说说内核和系统调用权限。</strong> 内核是操作系统连接硬件、提供最核心能力的程序。今天我们先简单了解一下，关于内核的详细知识，会在“14 |用户态和内核态：用户态线程和内核态线程有什么区别？”中介绍。</p>
<p data-nodeid="2079">内核提供操作硬件、磁盘、内存分页、进程等最核心的能力，并拥有直接操作全部内存的权限，因此内核不能把自己的全部能力都提供给用户，而且也不能允许用户通过<code data-backticks="1" data-nodeid="2340">shell</code>指令进行系统调用。Linux 下内核把部分进程需要的系统调用以 C 语言 API 的形式提供出来。部分系统调用会有权限检查，比如说设置系统时间的系统调用。</p>
<p data-nodeid="2080">以上我们看到了 Linux 对系统权限的抽象。接下来我们再说说权限架构的思想。</p>
<h3 data-nodeid="2081">权限架构思想</h3>
<p data-nodeid="2082">优秀的权限架构主要目标是让系统安全、稳定且用户、程序之间相互制约、相互隔离。这要求权限系统中的权限划分足够清晰，分配权限的成本足够低。</p>
<p data-nodeid="61028" class="">因此，优秀的架构，应该遵循最小权限原则（Least Privilege）。权限设计需要保证系统的安全和稳定。比如：每一个成员拥有的权限应该足够的小，每一段特权程序执行的过程应该足够的短。对于安全级别较高的时候，还需要成员权限互相牵制。比如金融领域通常登录线上数据库需要两次登录，也就是需要两个密码，分别掌握在两个角色手中。这样即便一个成员出了问题，也可以保证整个系统安全。</p>


<p data-nodeid="2084">同样的，每个程序也应该减少权限，比如说只拥有少量的目录读写权限，只可以进行少量的系统调用。</p>
<h4 data-nodeid="2085">权限划分</h4>
<p data-nodeid="2086">此外，权限架构思想还应遵循一个原则，权限划分边界应该足够清晰，尽量做到相互隔离。Linux 提供了用户和分组。当然 Linux 没有强迫你如何划分权限，这是为了应对更多的场景。通常我们服务器上重要的应用，会由不同的账户执行。比如说 Nginx、Web 服务器、数据库不会执行在一个账户下。现在随着容器化技术的发展，我们甚至希望每个应用独享一个虚拟的空间，就好像运行在一个单独的操作系统中一样，让它们互相不用干扰。</p>
<p data-nodeid="2087"><strong data-nodeid="2353">到这里，你可能会问：为什么不用 root 账户执行程序？</strong> 下面我们就来说说 root 的危害。</p>
<p data-nodeid="79424" class="">举个例子，你有一个 MySQL 进程执行在 root（最大权限）账户上，如果有黑客攻破了你的 MySQL 服务，获得了在 MySQL 上执行 SQL 的权限，那么，你的整个系统就都暴露在黑客眼前了。这会导致非常严重的后果。</p>


















<p data-nodeid="83002" class="">黑客可以利用 MySQL 的 Copy From Prgram 指令为所欲为，比如先备份你的关键文件，然后再删除他们，并要挟你通过指定账户打款。如果执行最小权限原则，那么黑客即便攻破我们的 MySQL 服务，他也只能获得最小的权限。当然，黑客拿到 MySQL 权限也是非常可怕的，但是相比拿到所有权限，这个损失就小多了。</p>




<h4 data-nodeid="2090" class="">分级保护</h4>
<p data-nodeid="2091">因为内核可以直接操作内存和 CPU，因此非常危险。驱动程序可以直接控制摄像头、显示屏等核心设备，也需要采取安全措施，比如防止恶意应用开启摄像头盗用隐私。通常操作系统都采取一种环状的保护模式。</p>
<p data-nodeid="2092"><img src="https://s0.lgstatic.com/i/image/M00/5A/53/CgqCHl91HB2AdNsAAAEpE6rtlHM754.png" alt="3.png" data-nodeid="2360"></p>
<p data-nodeid="102960" class="te-preview-highlight">如上图所示，内核在最里面，也就是 Ring 0。 应用在最外面也就是 Ring 3。驱动在中间，也就是 Ring 1 和 Ring 2。对于相邻的两个 Ring，内层 Ring 会拥有较高的权限，可以改变外层的 Ring；而外层的 Ring 想要使用内层 Ring 的资源时，会有专门的程序（或者硬件）进行保护。</p>

<p data-nodeid="2094">比如说一个 Ring3 的应用需要使用内核，就需要发送一个系统调用给内核。这个系统调用会由内核进行验证，比如验证用户有没有足够的权限，以及这个行为是否安全等等。</p>
<p data-nodeid="2095"><strong data-nodeid="2366">权限包围（Privilege Bracking）</strong></p>
<p data-nodeid="88120" class="">之前我们讨论过，当 MySQL 跑在 root 权限时，如果 MySQLl 被攻破，整个机器就被攻破了。因此我们所有应用都不要跑在 root 上。如果所有应用都跑在普通账户下，那么就会有临时提升权限的场景。比如说安装程序可能需要临时拥有管理员权限，将应用装到<code data-backticks="1" data-nodeid="88122">/usr/bin</code>目录下。</p>





<p data-nodeid="2097">Linux 提供了权限包围的能力。比如一个应用，临时需要高级权限，可以利用交互界面（比如让用户输入 root 账户密码）验证身份，然后执行需要高级权限的操作，然后马上恢复到普通权限工作。这样做可以减少应用在高级权限的时间，并做到专权专用，防止被恶意程序利用。</p>
<h3 data-nodeid="2098">用户分组指令</h3>
<p data-nodeid="2099">上面我们讨论了 Linux 权限的架构，接下来我们学习一些具体的指令。</p>
<h4 data-nodeid="2100">查看</h4>
<p data-nodeid="2101">如果想查看当前用户的分组可以使用<code data-backticks="1" data-nodeid="2375">groups</code>指令。</p>
<p data-nodeid="2102"><img src="https://s0.lgstatic.com/i/image/M00/5A/3F/CgqCHl90SU6AUJrLAADmRyiiAig313.png" alt="Drawing 5.png" data-nodeid="2379"></p>
<p data-nodeid="2103">上面指令列出当前用户的所有分组。第一个是同名的主要分组，后面从<code data-backticks="1" data-nodeid="2381">adm</code>开始是次级分组。</p>
<p data-nodeid="2104">我先给你介绍两个分组，其他分组你可以去查资料：</p>
<ul data-nodeid="2105">
<li data-nodeid="2106">
<p data-nodeid="2107">adm 分组用于系统监控，比如<code data-backticks="1" data-nodeid="2385">/var/log</code>中的部分日志就是 adm 分组。</p>
</li>
<li data-nodeid="2108">
<p data-nodeid="2109">sudo 分组用户可以通过 sudo 指令提升权限。</p>
</li>
</ul>
<p data-nodeid="2110">如果想查看当前用户，可以使用<code data-backticks="1" data-nodeid="2389">id</code>指令，如下所示：</p>
<p data-nodeid="2111"><img src="https://s0.lgstatic.com/i/image/M00/5A/3F/CgqCHl90SVSALssXAAGhSpF-cWY440.png" alt="Drawing 6.png" data-nodeid="2393"></p>
<ul data-nodeid="2112">
<li data-nodeid="2113">
<p data-nodeid="2114">uid 是用户 id；</p>
</li>
<li data-nodeid="2115">
<p data-nodeid="2116">gid 是组 id；</p>
</li>
<li data-nodeid="2117">
<p data-nodeid="2118">groups 后面是每个分组和分组的 id。</p>
</li>
</ul>
<p data-nodeid="2119">如果想查看所有的用户，可以直接看<code data-backticks="1" data-nodeid="2398">/etc/passwd</code>。</p>
<p data-nodeid="2120"><img src="https://s0.lgstatic.com/i/image/M00/5A/3F/CgqCHl90SVqAIja7AAXBj3lebBQ651.png" alt="Drawing 7.png" data-nodeid="2402"></p>
<p data-nodeid="2121"><code data-backticks="1" data-nodeid="2403">/etc/passwd</code>这个文件存储了所有的用户信息，如下图所示：</p>
<p data-nodeid="2122"><img src="https://s0.lgstatic.com/i/image/M00/5A/53/CgqCHl91HIGAWXWVAACI9cgafaM295.png" alt="WechatIMG144.png" data-nodeid="2407"></p>
<h4 data-nodeid="2123">创建用户</h4>
<p data-nodeid="2124">创建用户用<code data-backticks="1" data-nodeid="2410">useradd</code>指令。</p>
<pre class="lang-plain" data-nodeid="2125"><code data-language="plain">sudo useradd foo
</code></pre>
<p data-nodeid="98854" class="">sudo 原意是 superuser do，后来演变成用另一个用户的身份去执行某个指令。如果没有指定需要 sudo 的用户，就可以像上面那样，以超级管理员的身份。因为 useradd 需要管理员身份。这句话执行后，会进行权限提升，并弹出输入管理员密码的输入界面。</p>











<h4 data-nodeid="2127"><strong data-nodeid="2416">创建分组</strong></h4>
<p data-nodeid="2128">创建分组用<code data-backticks="1" data-nodeid="2418">groupadd</code>指令。下面指令创建一个叫作<code data-backticks="1" data-nodeid="2420">hello</code>的分组。</p>
<pre class="lang-plain" data-nodeid="2129"><code data-language="plain">sudo groupadd hello
</code></pre>
<h4 data-nodeid="2130">为用户增加次级分组</h4>
<p data-nodeid="2131">组分成主要分组（Primary Group）和次级分组（Secondary Group）。主要分组只有 1 个，次级分组可以有多个。如果想为用户添加一个次级分组，可以用<code data-backticks="1" data-nodeid="2424">usermod</code>指令。下面指令将用户<code data-backticks="1" data-nodeid="2426">foo</code>添加到<code data-backticks="1" data-nodeid="2428">sudo</code>分组，从而<code data-backticks="1" data-nodeid="2430">foo</code>拥有了<code data-backticks="1" data-nodeid="2432">sudo</code>的权限。</p>
<pre class="lang-plain" data-nodeid="2132"><code data-language="plain">sudo usermod -a -G sudo foo
</code></pre>
<p data-nodeid="2133"><code data-backticks="1" data-nodeid="2434">-a</code>代表append，<code data-backticks="1" data-nodeid="2436">-G</code>代表一个次级分组的清单， 最后一个<code data-backticks="1" data-nodeid="2438">foo</code>是账户名。</p>
<h4 data-nodeid="2134">修改用户主要分组</h4>
<p data-nodeid="2135">修改主要分组还是使用<code data-backticks="1" data-nodeid="2442">usermod</code>指令。只不过参数是小写的<code data-backticks="1" data-nodeid="2444">-g</code>。</p>
<pre class="lang-plain" data-nodeid="2136"><code data-language="plain">sudo usermod -g somegroup foo
</code></pre>
<h3 data-nodeid="2137">文件权限管理指令</h3>
<p data-nodeid="2138">接下来我们学习文件管理相关的指令。</p>
<h4 data-nodeid="2139">查看</h4>
<p data-nodeid="2140">我们可以用<code data-backticks="1" data-nodeid="2450">ls -l</code>查看文件的权限，相关内容在本课时前面已经介绍过了。</p>
<h4 data-nodeid="2141">修改文件权限</h4>
<p data-nodeid="2142">可以用<code data-backticks="1" data-nodeid="2454">chmod</code>修改文件权限，<code data-backticks="1" data-nodeid="2456">chmod</code>（ change file mode bits），也就是我们之前学习的 rwx，只不过 rwx 在 Linux 中是用三个连在一起的二进制位来表示。</p>
<pre class="lang-shell" data-nodeid="2143"><code data-language="shell"><span class="hljs-meta">#</span><span class="bash"> 设置foo可以执行</span>
chmod +x ./foo
<span class="hljs-meta">#</span><span class="bash"> 不允许foo执行</span>
chmod -x ./foo
<span class="hljs-meta">#</span><span class="bash"> 也可以同时设置多个权限</span>
chmod +rwx ./foo
</code></pre>
<p data-nodeid="99876" class="">因为<code data-backticks="1" data-nodeid="99878">rwx</code>在 Linux 中用相邻的 3 个位来表示。比如说<code data-backticks="1" data-nodeid="99880">111</code>代表<code data-backticks="1" data-nodeid="99882">rwx</code>，<code data-backticks="1" data-nodeid="99884">101</code>代表<code data-backticks="1" data-nodeid="99886">r-x</code>。而<code data-backticks="1" data-nodeid="99888">rwx</code>总共有三组，分别是用户权限、组权限和全部用户权限。也就是可以用<code data-backticks="1" data-nodeid="99890">111111111</code> 9 个 1 代表<code data-backticks="1" data-nodeid="99892">rwxrwxrwx</code>。又因为<code data-backticks="1" data-nodeid="99894">111</code>10 进制是 7，因此当需要一次性设置用户权限、组权限和所有用户权限的时候，我们经常用数字表示。</p>

<pre class="lang-plain" data-nodeid="2145"><code data-language="plain"># 设置rwxrwxrwx (111111111 -&gt; 777)
chmod 777 ./foo
# 设置rw-rw-rw-(110110110 -&gt; 666)
chmod 666 ./foo
</code></pre>
<h4 data-nodeid="2146">修改文件所属用户</h4>
<p data-nodeid="2147">有时候我们需要修改文件所属用户，这个时候会使用<code data-backticks="1" data-nodeid="2479">chown</code>指令。 下面指令修改<code data-backticks="1" data-nodeid="2481">foo</code>文件所属的用户为<code data-backticks="1" data-nodeid="2483">bar</code>。</p>
<pre class="lang-plain" data-nodeid="2148"><code data-language="plain">chown bar ./foo
</code></pre>
<p data-nodeid="2149">还有一些情况下，我们需要同时修改文件所属的用户和分组，比如我们想修改<code data-backticks="1" data-nodeid="2486">foo</code>的分组位<code data-backticks="1" data-nodeid="2488">g</code>，用户为<code data-backticks="1" data-nodeid="2490">u</code>，可以使用：</p>
<pre class="lang-plain" data-nodeid="2150"><code data-language="plain">chown g.u ./foo
</code></pre>
<h3 data-nodeid="2151">总结</h3>
<p data-nodeid="101938" class="">这节课我们学习 Linux 的权限管理的抽象和架构思想。Linux 对用户、组、文件、系统调用等都进行了完善的抽象。之后，讨论了最小权限原则。最后我们对用户分组管理和文件权限管理两部分重要的指令进行了系统学习。</p>


<p data-nodeid="2153">那么通过这节课的学习，你现在可以来回答本节关联的面试题目：<strong data-nodeid="2498">请简述 Linux 权限划分的原则？</strong></p>
<p data-nodeid="2154">老规矩，请你先在脑海里构思下给面试官的表述，并把你的思考写在留言区，然后再来看我接下来的分析。</p>
<p data-nodeid="2155"><strong data-nodeid="2504">【解析】</strong> Linux 遵循最小权限原则。</p>
<ol data-nodeid="2156">
<li data-nodeid="2157">
<p data-nodeid="2158">每个用户掌握的权限应该足够小，每个组掌握的权限也足够小。实际生产过程中，最好管理员权限可以拆分，互相牵制防止问题。</p>
</li>
<li data-nodeid="2159">
<p data-nodeid="2160">每个应用应当尽可能小的使用权限。最理想的是每个应用单独占用一个容器（比如 Docker），这样就不存在互相影响的问题。即便应用被攻破，也无法攻破 Docker 的保护层。</p>
</li>
<li data-nodeid="2161">
<p data-nodeid="2162">尽可能少的<code data-backticks="1" data-nodeid="2508">root</code>。如果一个用户需要<code data-backticks="1" data-nodeid="2510">root</code>能力，那么应当进行权限包围——马上提升权限（比如 sudo），处理后马上释放权限。</p>
</li>
<li data-nodeid="2163">
<p data-nodeid="2164">系统层面实现权限分级保护，将系统的权限分成一个个 Ring，外层 Ring 调用内层 Ring 时需要内层 Ring 进行权限校验。</p>
</li>
</ol>
<h3 data-nodeid="2165" class="">思考题</h3>
<p data-nodeid="2166">最后再给你留一道实战问题，希望你在课下自己尝试一下。<strong data-nodeid="2519">如果一个目录是只读权限，那么这个目录下面的文件还可写吗</strong>？</p>
<p data-nodeid="2167" class="">你可以把你的答案、思路或者课后总结写在留言区，这样可以帮助你产生更多的思考，这也是构建知识体系的一部分。经过长期的积累，相信你会得到意想不到的收获。如果你觉得今天的内容对你有所启发，欢迎分享给身边的朋友。期待看到你的思考！</p>

---

### 精选评论

##### **天：
> 这节课总结下来胜过看10篇csdn别人的笔记

##### **盛：
> 如果只是赋予只读权限，那么连目录都无法进入：chmod 444 目录cd 目录提示bash: cd: test-htc/: Permission denied如果赋予目录读和执行的权限，那么进入该目录无法创建新文件cd 目录touch 文件提示touch: cannot touch ‘abc’: Permission denied但是可以编辑旧文件（已存在该目录下的文件）

##### **康：
> 对于非 root 用户只读的目录不能写任何数据，也不可以 cd ls 。root 用户仍然可以读写。

##### **艺：
> 非root用户无法对目录进行读写

##### *程：
> Linux 权限划分的原则：最小权限原则，有点类似于我们编程中函数应该遵循单一职责原则。因为，遵循单一原则有这些好处，首先在维护上是比较容易维护，特别是出现错误的时候，因为函数都是遵循单一原则，所以那里出了错误找那里即可，其次，能够减少函数共享的状态，函数通常需要做很多事情，做很多事情就意味着操作的变量或者计算比较多，那么遵循单一原则，则表示一次只有一件事情，和操作传入的参数变量（当然，参数变量也尽量少）

##### **0013：
> 目录只有读权限的，下面文件即使时候读写权限应该也不能写。

##### **安：
> 如果该目录下的文件具有写权限，依然可写

##### *川：
> 如果是root用户登录的话，读写都没问题，毕竟超级用户。如果是普通用户，则进不了目录。

##### **钻：
> 目录只读权限时，普通用户进不去目录，而且没有权限去写目录下没的文件。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 错了哈， 看下加餐部分解题思路。

##### **钻：
> 1.文件被创建后，初始的权限如何设置？初始文件权限一般=666-umask2.需要全部用户都可以执行的指令，比如ls，它们的权限如何分配？可以给ls赋予chmod a=xxx3.给一个文本文件分配了可执行权限会怎么样？那文本文件就可以用bash执行，类似./file4.可不可以多个用户都登录root，然后只用root账户？可以，但是考虑安全性，一般都不这么做。

 ###### &nbsp;&nbsp;&nbsp; 讲师回复：
> &nbsp;&nbsp;&nbsp; 赞！

##### **盛：
> linux中对文件的权限设置以及作用https://blog.51cto.com/13866901/2148097

##### **0812：
> 讲的不错

